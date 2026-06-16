# Java Concurrency — Threads, Synchronization, Executor Framework & Collections

> **Quick Reference:** A thread is the smallest unit of execution inside a process. Java's multithreading model allows concurrent task execution; synchronization prevents race conditions on shared data; the Executor Framework manages thread lifecycle efficiently; the Collections framework provides data structures for all use cases.

---

## What is a Thread?

A thread is the smallest unit of execution inside a **process**.

- **Process** = running application (e.g., your Spring Boot service)
- **Thread** = individual task running inside that process

Real-world examples:
- Downloading a file while UI remains responsive
- Background autosave while editing a document
- Handling multiple API requests simultaneously (Spring Boot / Tomcat creates a thread per request by default)
- Async email sending while the main request returns immediately

---

## Process vs Thread

| | Process | Thread |
|---|---|---|
| Definition | Running instance of a program | Smallest unit of execution inside a process |
| Memory | Separate memory space (isolated) | **Shared** heap; separate stack per thread |
| Communication | IPC (sockets, pipes, shared memory) | Direct shared memory |
| Overhead | Heavy (OS allocates full memory space) | Light (shares process memory) |
| Creation | Slow | Fast |

---

## Time Sharing

CPU executes multiple threads by rapidly giving each a small **time slice**, creating the illusion of simultaneous execution:

```
Thread A → 10 ms CPU time
Thread B → 10 ms CPU time
Thread C → 10 ms CPU time
→ cycles back to A ...
```

This is managed by the **OS Thread Scheduler**, not Java directly.

---

## Thread Scheduler

- JVM relies on the **OS scheduler** to decide which thread gets CPU time
- Based on: thread priority, thread state (running/waiting/blocked), available CPU cores
- **Java cannot fully control scheduling** — behavior is OS-dependent

---

## Thread Priority

Range: 1 (lowest) to 10 (highest). Default is 5.

```java
Thread t = new Thread();
t.setPriority(Thread.MAX_PRIORITY);   // 10
t.setPriority(Thread.NORM_PRIORITY);  // 5 (default)
t.setPriority(Thread.MIN_PRIORITY);   // 1
```

**Important:** Priority is a **hint**, not a guarantee. The OS scheduler makes the final decision.

---

## Creating Threads — 4 Methods

### Method 1: Extend Thread Class

```java
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread running: " + Thread.currentThread().getName());
    }
}

MyThread t = new MyThread();
t.start();  // creates new OS thread and calls run() inside it
```

**Limitation:** Java has single inheritance — extending `Thread` prevents extending any other class.

### Method 2: Implement Runnable Interface (Preferred)

```java
class MyTask implements Runnable {
    public void run() {
        System.out.println("Runnable task running");
    }
}

Thread t = new Thread(new MyTask());
t.start();
```

**Why preferred:**
- Can still extend another class simultaneously
- Separates the **task** (what to do) from the **thread** (how to run it)
- Cleaner design — `Runnable` is just a job, `Thread` is the execution vehicle
- Better for use with `ExecutorService`

### Method 3: Anonymous Inner Class

```java
Thread t = new Thread(new Runnable() {
    public void run() {
        System.out.println("Anonymous thread");
    }
});
t.start();
```

### Method 4: Lambda Expression (Modern — Java 8+)

```java
Thread t = new Thread(() -> {
    System.out.println("Lambda thread — cleaner syntax");
});
t.start();

// Even shorter for one-liners:
new Thread(() -> System.out.println("Inline lambda thread")).start();
```

**Why lambda:** `Runnable` is a `@FunctionalInterface` — one abstract method `run()`. Lambda directly provides the implementation. Clean, modern, used heavily with `ExecutorService` in production code.

---

## Thread Methods

### `start()`
Creates a new OS thread and calls `run()` inside it — true concurrency.
```java
t.start();  // CORRECT — runs in new thread concurrently
t.run();    // WRONG for multithreading — just a method call in current thread, no new thread
```

### `sleep()`
Pauses the **current thread** for a specified duration. Does **NOT release locks**.
```java
Thread.sleep(1000);  // pause current thread for 1 second
```

### `join()`
The calling thread waits until the **specified thread finishes** before continuing.
```java
t.start();
t.join();  // main thread waits here until t completes
System.out.println("t is done — now continue");
```
Real use case: Wait for a background data-fetching thread before processing its results.

### `yield()`
Hints the scheduler to give CPU time to other threads. Not guaranteed.
```java
Thread.yield();
```

### `wait()` / `notify()` / `notifyAll()`
Used for **inter-thread communication** on a shared object's monitor. Must be called inside a `synchronized` block.
```java
synchronized(sharedObj) {
    while (!condition) {
        sharedObj.wait();       // releases lock and waits until notified
    }
    // condition met — proceed
}

synchronized(sharedObj) {
    condition = true;
    sharedObj.notify();         // wakes ONE waiting thread
    // sharedObj.notifyAll();   // wakes ALL waiting threads
}
```

### `stop()` — Deprecated, Never Use
`t.stop()` is unsafe — can leave objects in inconsistent state. Use a flag or `interrupt()` instead.

```java
// Modern pattern: use a volatile flag for graceful shutdown
volatile boolean running = true;

void run() {
    while (running) {
        // do work
    }
}

void stopThread() {
    running = false;  // thread will exit on next loop check
}
```

---

## `wait()` vs `sleep()`

| | `wait()` | `sleep()` |
|---|---|---|
| Class | `Object` | `Thread` |
| Releases lock | ✅ Yes | ❌ No |
| Wake-up trigger | `notify()` / `notifyAll()` | After specified time elapsed |
| Must be in `synchronized` | ✅ Yes | ❌ No |
| Use case | Inter-thread communication | Pause execution |

---

## Thread Memory Model

**Threads SHARE (from the process):**
- Heap memory (all objects and arrays)
- Static variables
- Instance variables of shared objects

**Each Thread has its OWN:**
- Stack memory (method frames, local variables)
- PC (Program Counter) register
- Local variables (not visible to other threads)

---

## Race Condition

Multiple threads modify shared data simultaneously causing **inconsistent / incorrect results**.

```java
class Counter {
    int count = 0;

    void increment() {
        count++;  // NOT atomic — 3 steps: read count, add 1, write count
    }
}
```

Two threads incrementing simultaneously:
```
Thread A: reads count = 0
Thread B: reads count = 0         (before A writes)
Thread A: writes count = 1
Thread B: writes count = 1        (overwrites A's update — lost update!)
// Expected: count = 2, Actual: count = 1
```

---

## Mutation in Multithreading

**Mutation** = changing shared state. Unsafe shared mutable objects cause race conditions and data corruption.

```java
balance = balance - 100;  // read-modify-write — not atomic
list.add(item);           // ArrayList is not thread-safe
```

In Spring Boot microservices, shared mutable state includes:
- In-memory counters (rate limiters, request counters)
- Cached data (local cache, before writing to Redis)
- DB connection pool state
- Inventory counts or balance fields in concurrent transactions

---

## `synchronized` Keyword

Prevents multiple threads from entering a **critical section** simultaneously. Thread acquires a **monitor lock** on an object, executes, then releases.

### Synchronized Method

```java
class Counter {
    int count = 0;

    synchronized void increment() {
        count++;  // only one thread at a time can execute this
    }
}
```

### Synchronized Block (More Fine-Grained — Better Performance)

```java
class Counter {
    int count = 0;
    Object lock = new Object();

    void increment() {
        synchronized(lock) {    // lock only the critical section
            count++;
        }
        // other code here runs concurrently with other threads
    }
}
```

**Drawbacks of `synchronized`:**
- Blocking — threads queue up waiting for the lock
- Reduces concurrency (only one thread at a time in the section)
- Deadlock risk if locks are acquired in inconsistent orders

**Modern alternatives:**
- `AtomicInteger`, `AtomicLong` — for simple atomic counters (lock-free)
- `ReentrantLock` — for advanced locking (tryLock, fairness, interruptible)
- `ConcurrentHashMap`, `CopyOnWriteArrayList` — built-in thread-safe collections

---

## Deadlock

Two (or more) threads waiting forever for each other's locks — neither can proceed.

```
Thread A holds Lock 1, waiting for Lock 2
Thread B holds Lock 2, waiting for Lock 1
→ Both blocked forever — deadlock
```

**Prevention strategies:**
- Always acquire multiple locks in the **same order** across all threads
- Use `tryLock()` with timeout (`ReentrantLock.tryLock(timeout, unit)`)
- Minimize the scope of `synchronized` blocks — hold locks as briefly as possible
- Prefer immutable objects — shared immutable data needs no locking
- Design to avoid sharing mutable state between threads

---

## Why Modern Thread Management? (Motivation for Executor Framework)

Creating raw threads manually for every task is problematic:
- **Thread creation cost is high** — each platform thread allocates a dedicated OS stack (~512KB–1MB, depending on OS and the `-Xss` setting; ~1MB on a typical 64-bit JVM)
- **Context switching overhead** — OS must save/restore state on every switch
- **Hard to control concurrency** — no easy cap on number of threads
- **Hard to schedule tasks** — no built-in delay/periodic support
- **Difficult lifecycle management** — no clean shutdown mechanism

**Solution: Executor Framework** — introduced in Java 5 (`java.util.concurrent`).

---

## ThreadGroup (Legacy API)

A `ThreadGroup` groups multiple threads under one logical unit — for bulk management.

```java
ThreadGroup group = new ThreadGroup("Workers");

Thread t1 = new Thread(group, () -> System.out.println("Task 1"));
Thread t2 = new Thread(group, () -> System.out.println("Task 2"));

t1.start();
t2.start();
```

**Common Methods:**
```java
group.interrupt();   // interrupt all threads in the group
group.list();        // print info about all threads in group
group.activeCount(); // number of active threads
```

**Used for:**
- Managing related threads together
- Bulk interruption
- Priority control across a group
- Monitoring

**Important:** `ThreadGroup` is a **legacy API**. In modern Java, the **Executor Framework** is strongly preferred — it provides all these capabilities and more with a cleaner API.

---

## Why Thread Pools?

Creating a new platform `Thread` for every task is expensive — each one maps to an OS thread and allocates a dedicated stack (~512KB–1MB, OS/`-Xss`-dependent) plus OS-level context-switching overhead.

> **Version-dependent caveat (Java 21+):** This "threads are expensive → reuse them in a pool" reasoning is true for **platform** threads only. Java 21 **virtual threads** (Project Loom) are extremely cheap (a few hundred bytes of heap, no OS thread per task), which makes thread-per-task practical again and removes the pooling rationale for blocking I/O workloads. See **module 08 — Modern Java** for details.

| Without Thread Pool | With Thread Pool |
|---|---|
| New thread created per task | Fixed set of reusable worker threads |
| High memory overhead | Bounded memory usage |
| OS scheduler overhead per task | Minimal OS scheduling overhead |
| No task queuing | Built-in task queue |
| Hard to limit concurrency | Core/max pool size controls concurrency |
| No lifecycle management | Graceful shutdown built in |

**Thread pool benefits:**
1. Reuses threads — eliminates creation/destruction cost
2. Limits concurrency — prevents thread explosion under load
3. Queues tasks — absorbs bursts gracefully
4. Provides lifecycle management — `shutdown()`, `awaitTermination()`
5. Enables monitoring — pool size, queue depth, completed task count
6. Supports scheduling — delayed and periodic task execution

---

## Executor Framework

The core components of `java.util.concurrent`:

| Type | Purpose |
|---|---|
| `Executor` | Basic contract — execute a `Runnable` |
| `ExecutorService` | Lifecycle management + `submit()` + `Future` support |
| `ScheduledExecutorService` | Delayed and periodic task execution |
| `ThreadPoolExecutor` | The customizable core implementation behind the pools |
| `Executors` | Factory utility class — creates ready-made pools |

---

## Executor Interface

The simplest contract — just execute a task:

```java
public interface Executor {
    void execute(Runnable command);
}

// Custom minimal executor:
Executor executor = command -> new Thread(command).start();
executor.execute(() -> System.out.println("Run"));
```

In practice, you always use higher-level implementations (`ExecutorService`). `Executor` alone has no lifecycle management.

---

## ExecutorService

Extends `Executor`. Adds lifecycle management, `submit()`, `Future` support, and `invokeAll()`.

```java
ExecutorService service = Executors.newFixedThreadPool(3);

service.execute(() -> System.out.println("Fire and forget"));  // no return value
service.submit(() -> System.out.println("Submitted task"));    // returns Future<?>
service.shutdown();
```

**Key Methods:**

| Method | Use |
|---|---|
| `execute(Runnable)` | Run task — no result, no exception tracking |
| `submit(Runnable)` | Run task — returns `Future<?>` |
| `submit(Callable<T>)` | Run task — returns `Future<T>` with result |
| `shutdown()` | Stop accepting new tasks; waits for running tasks to finish |
| `shutdownNow()` | Attempts to stop all running tasks immediately |
| `isShutdown()` | Returns true if shutdown was called |
| `awaitTermination(time, unit)` | Block until all tasks finish or timeout |
| `invokeAll(collection)` | Submit all tasks, wait for all to complete |

---

## `execute()` vs `submit()`

```java
// execute() — fire and forget, void return, uncaught exceptions go to thread's handler
service.execute(() -> riskyOperation());

// submit() — returns Future; exceptions wrapped inside Future
Future<?> f = service.submit(() -> riskyOperation());
try {
    f.get();  // throws ExecutionException if task threw
} catch (ExecutionException e) {
    Throwable cause = e.getCause();  // the actual exception
}
```

| | `execute()` | `submit()` |
|---|---|---|
| Returns | `void` | `Future` |
| Exception handling | Uncaught exception handler / lost | Wrapped in `ExecutionException`, retrievable via `Future.get()` |
| Use when | Fire-and-forget | Need result or exception tracking |

---

## Callable and Future

`Callable<T>` is like `Runnable` but **returns a value** and can **throw checked exceptions**.

```java
Callable<Integer> task = () -> {
    Thread.sleep(1000);
    return 10 + 20;
};

ExecutorService service = Executors.newFixedThreadPool(2);
Future<Integer> future = service.submit(task);

// Do other work here while task runs in background...

Integer result = future.get();          // blocks until result is available
System.out.println("Result: " + result); // 30

// With timeout:
Integer result2 = future.get(2, TimeUnit.SECONDS);  // throws TimeoutException if too slow
```

**Future methods:**
```java
future.get()                     // block and get result (throws checked exceptions)
future.get(timeout, unit)        // block with timeout
future.isDone()                  // check if completed (without blocking)
future.cancel(mayInterruptIfRunning)  // attempt to cancel
future.isCancelled()             // check if cancelled
```

---

## Executors — Factory Class

Creates ready-made thread pools. Each type is backed by `ThreadPoolExecutor` internally.

### 1. `newFixedThreadPool(n)` — Fixed Worker Count

```java
ExecutorService pool = Executors.newFixedThreadPool(5);
```

- Fixed number of threads — always exactly `n` workers
- New tasks queue up in `LinkedBlockingQueue` if all threads are busy
- **Use when:** max concurrency is known (n = CPU cores for CPU-bound; higher for I/O-bound)
- **Real use cases:** REST API request handling, parallel batch jobs, DB query execution

### 2. `newSingleThreadExecutor()` — Serial Execution

```java
ExecutorService pool = Executors.newSingleThreadExecutor();
```

- Only **one** worker thread — tasks execute one at a time, in submission order
- If thread dies unexpectedly, a new one is created to replace it
- **Use when:** tasks must run sequentially (no parallelism)
- **Real use cases:** payment updates, file writes, audit log entries, queue processing

### 3. `newCachedThreadPool()` — Dynamic Pool

```java
ExecutorService pool = Executors.newCachedThreadPool();
```

- Creates new threads as needed; reuses idle threads (idle for 60s are terminated)
- Thread count is unbounded — can grow very large under load
- **Use when:** many short-lived async tasks with unpredictable load
- **Risk:** can create thousands of threads under high load → `OutOfMemoryError`
- **Real use cases:** brief async tasks, lightweight background work

### 4. `newScheduledThreadPool(n)` — Timed / Periodic Tasks

```java
ScheduledExecutorService pool = Executors.newScheduledThreadPool(2);

// Run once after a delay
pool.schedule(
    () -> System.out.println("Runs after 5 seconds"),
    5, TimeUnit.SECONDS
);

// Run repeatedly at fixed rate (starts 0s delay, then every 10s)
pool.scheduleAtFixedRate(
    () -> System.out.println("Periodic task"),
    0, 10, TimeUnit.SECONDS
);

// Run with fixed delay between end of last run and start of next
pool.scheduleWithFixedDelay(
    () -> System.out.println("Fixed delay task"),
    0, 5, TimeUnit.SECONDS
);
```

- **Use when:** retry jobs, health checks, token refresh, cleanup tasks, cron-like polling
- **Real use cases (your profile):** scheduled reports, notification sending, cache invalidation, async job polling

---

## ThreadPoolExecutor — Customizable Core

The real implementation behind all `Executors` factory methods. Use directly when you need full control.

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    2,                          // corePoolSize    — min threads always kept alive
    4,                          // maxPoolSize     — max threads under high load
    60,                         // keepAliveTime   — idle threads above core die after 60s
    TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(100)  // workQueue   — holds tasks when all core threads busy
);
```

### Parameters Explained

| Parameter | Meaning |
|---|---|
| `corePoolSize` | Threads always alive even when idle |
| `maxPoolSize` | Max threads created when queue is full |
| `keepAliveTime` | Time excess threads (above core) wait before dying |
| `workQueue` | Queue that holds waiting tasks |

**Execution flow:**
1. If active threads < `corePoolSize` → create new thread
2. If active threads >= core → add to queue
3. If queue full + active threads < max → create new thread
4. If queue full + max threads reached → apply **rejection policy**

### Rejection Policies

When queue is full AND max thread count reached — what to do with the new task:

| Policy | Behavior |
|---|---|
| `AbortPolicy` (default) | Throw `RejectedExecutionException` |
| `CallerRunsPolicy` | Run the task in the caller's thread (slows down producer) |
| `DiscardPolicy` | Silently discard the new task |
| `DiscardOldestPolicy` | Discard the oldest queued task, then retry submission |

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    2, 4, 60, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(50),
    new ThreadPoolExecutor.CallerRunsPolicy()  // backpressure on caller
);
```

**Production recommendation:** Use `CallerRunsPolicy` — it creates natural backpressure (slows down producers when system is overloaded), rather than silently losing work.

### Why Use `ThreadPoolExecutor` Instead of `Executors`?

`Executors.newFixedThreadPool()` uses an **unbounded** `LinkedBlockingQueue` — under extreme load, millions of tasks can queue up → `OutOfMemoryError`. `ThreadPoolExecutor` lets you bound the queue and define rejection behavior explicitly.

---

## Proper Shutdown

```java
service.shutdown();  // stop accepting new tasks; complete submitted tasks

// Wait up to 5s for graceful shutdown, then force-stop
if (!service.awaitTermination(5, TimeUnit.SECONDS)) {
    service.shutdownNow();  // interrupt running tasks
}
```

**Never just abandon an ExecutorService** — threads stay alive (daemon or not) and prevent JVM shutdown.

---

## Spring Boot — Executor Framework Integration

```java
// @Async — runs method in Spring's async thread pool
@Service
class EmailService {
    @Async
    public CompletableFuture<Void> sendEmail(String to, String subject) {
        // email logic — runs in background thread, doesn't block HTTP response
        return CompletableFuture.completedFuture(null);
    }
}

// @Scheduled — periodic task (needs @EnableScheduling on config class)
@Scheduled(fixedRate = 10000)         // every 10 seconds
public void refreshCache() { ... }

@Scheduled(cron = "0 0 2 * * *")     // every day at 2 AM
public void generateDailyReport() { ... }

// Parallel external API calls with CompletableFuture
CompletableFuture<User>  userFuture  = CompletableFuture.supplyAsync(() -> userService.get(id));
CompletableFuture<Order> orderFuture = CompletableFuture.supplyAsync(() -> orderService.get(id));

CompletableFuture.allOf(userFuture, orderFuture).join();  // wait for both
User user   = userFuture.get();
Order order = orderFuture.get();

// Configure custom thread pool for @Async
@Configuration
@EnableAsync
class AsyncConfig {
    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}
```

---

## Interview Coding Question

**"Write a method that takes tasks and performs them in a synchronized way with proper scheduling."**

This means:
- Accept multiple tasks
- Ensure no race condition
- Run tasks sequentially or in controlled order
- Use a scheduler or queue
- Thread-safe execution
- Clean shutdown

### Implementation — Sequential + Scheduled

```java
import java.util.concurrent.*;

public class TaskScheduler {

    private final ScheduledExecutorService executor =
        Executors.newSingleThreadScheduledExecutor();

    public void submitTask(Runnable task, int delaySec) {
        executor.schedule(() -> {
            synchronized (this) {    // protect any shared state
                task.run();
            }
        }, delaySec, TimeUnit.SECONDS);
    }

    public void shutdown() {
        executor.shutdown();
    }
}

// Usage:
TaskScheduler ts = new TaskScheduler();
ts.submitTask(() -> System.out.println("Task A"), 1);
ts.submitTask(() -> System.out.println("Task B"), 2);
ts.shutdown();
```

**Why this is a strong answer:**
- `newSingleThreadScheduledExecutor` = ordered sequential execution (no race condition between tasks)
- `schedule()` = proper timing control
- `synchronized (this)` = protects any shared state inside the task
- `shutdown()` = clean lifecycle management

### If Tasks Share a Resource (e.g., bank balance)

```java
private int balance = 0;

public synchronized void update(int amount) {
    balance += amount;  // only one thread at a time
}
```

### Enterprise-Level Enhancement

Use `ThreadPoolExecutor` with a bounded queue + `CallerRunsPolicy` + monitoring:

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    1, 1,              // single thread — serial execution
    0L, TimeUnit.MILLISECONDS,
    new LinkedBlockingQueue<>(500),
    new ThreadPoolExecutor.CallerRunsPolicy()  // backpressure if overwhelmed
);
```

---

## What NOT to Say in an Interview

❌ "I will create a `new Thread()` every time a task arrives"
→ Expensive, uncontrolled, no lifecycle management

❌ "Use `synchronized` everywhere"
→ Kills concurrency; use only at critical sections

❌ "Use `Thread.sleep()` for scheduling"
→ Blocking, imprecise; use `ScheduledExecutorService`

❌ "I don't know how to shut down threads"
→ Always mention `shutdown()` + `awaitTermination()`

---

## Strong Interview Answer Script

> "I would use `ScheduledExecutorService` for timing requirements and `newSingleThreadExecutor` if tasks must run sequentially. If tasks share mutable state, I'd synchronize only the critical section — or better, use `AtomicInteger` / `ConcurrentHashMap` for lock-free access. For production systems I'd use `ThreadPoolExecutor` directly with bounded queues and `CallerRunsPolicy` for backpressure. I'd always configure a clean shutdown with `awaitTermination`."

**Real-world project answer:**
> "In our microservices, I used `@Async` with a custom `ThreadPoolTaskExecutor` for notification sending and report generation, keeping the HTTP response fast. For scheduled jobs like cache refresh and cleanup, I used `@Scheduled` with cron expressions. For parallel external API calls I used `CompletableFuture.allOf()` to reduce latency."

---
