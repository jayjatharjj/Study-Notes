# Java Concurrency — Threads, Synchronization & Collections

> **Quick Reference:** A thread is the smallest unit of execution inside a process. Java's multithreading model allows concurrent task execution; synchronization prevents race conditions on shared data; the Collections framework provides data structures for all use cases.

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

## ExecutorService (Modern Thread Management)

Instead of creating raw threads, use `ExecutorService` to manage a **thread pool** — reuses threads, limits concurrency, handles lifecycle.

```java
import java.util.concurrent.*;

ExecutorService executor = Executors.newFixedThreadPool(5);  // 5 threads

executor.submit(() -> {
    System.out.println("Task running in thread pool");
});

executor.submit(() -> {
    return "Result from callable";  // Callable — returns a value
});

executor.shutdown();      // no new tasks; waits for submitted tasks to finish
executor.shutdownNow();   // tries to stop running tasks immediately
```

**Types of thread pools:**
| Factory | When to Use |
|---|---|
| `newFixedThreadPool(n)` | Known max concurrency (e.g., n = CPU cores) |
| `newCachedThreadPool()` | Short-lived bursts of tasks; reuses idle threads |
| `newSingleThreadExecutor()` | Sequential execution in a dedicated thread |
| `newScheduledThreadPool(n)` | Delayed or periodic tasks |

**In Spring Boot:**
- `@Async` annotation runs method in separate thread (uses Spring's `TaskExecutor`)
- `CompletableFuture` for async operations with composition
- `@Scheduled` for periodic tasks

```java
@Service
class EmailService {
    @Async                                   // runs in Spring's async executor thread pool
    public CompletableFuture<Void> sendEmail(String to, String subject) {
        // email logic here — doesn't block caller
        return CompletableFuture.completedFuture(null);
    }
}

// Parallel API calls with CompletableFuture
CompletableFuture<User> userFuture    = CompletableFuture.supplyAsync(() -> userService.get(id));
CompletableFuture<Order> orderFuture  = CompletableFuture.supplyAsync(() -> orderService.get(id));

CompletableFuture.allOf(userFuture, orderFuture).join();  // wait for both
User user   = userFuture.get();
Order order = orderFuture.get();
```

---

## Collections Framework

### Three Distinct Things Often Confused

### 1. Collection API (the entire framework)

The full framework of data structures in Java — `List`, `Set`, `Queue`, `Map`, algorithms, and utilities.

```
Collection API (framework)
├── java.util.List
├── java.util.Set
├── java.util.Queue
└── java.util.Map  (separate root — does NOT extend Collection interface)
```

### 2. `Collection` Interface

Root interface for List, Set, Queue hierarchies. Does NOT include `Map`.

```java
Collection<String> data = new ArrayList<>();
data.add("Java");
data.add("Spring");
data.size();     // 2
data.isEmpty();  // false
```

### 3. `Collections` Class

Utility class with **static helper methods** for working with collections.

```java
List<Integer> nums = new ArrayList<>(Arrays.asList(5, 2, 8, 1));

Collections.sort(nums);                              // sort ascending → [1, 2, 5, 8]
Collections.sort(nums, Comparator.reverseOrder());   // sort descending → [8, 5, 2, 1]
Collections.reverse(nums);                           // reverse in-place
Collections.shuffle(nums);                           // random order
Collections.max(nums);                               // 8
Collections.min(nums);                               // 1
Collections.frequency(nums, 5);                      // count occurrences of 5
Collections.unmodifiableList(nums);                  // read-only wrapper
Collections.synchronizedList(nums);                  // thread-safe wrapper
```

---

### Common Collection Types

| Interface | Common Implementations | Key Characteristic |
|---|---|---|
| `List` | `ArrayList`, `LinkedList`, `Vector` | Ordered, allows duplicates, index-based |
| `Set` | `HashSet`, `LinkedHashSet`, `TreeSet` | No duplicates |
| `Queue` | `LinkedList`, `PriorityQueue`, `ArrayDeque` | FIFO / priority order |
| `Map` | `HashMap`, `LinkedHashMap`, `TreeMap`, `Hashtable` | Key-value pairs |

**When to choose:**
- `ArrayList` → random access, reads >> writes
- `LinkedList` → frequent insertions/deletions in middle
- `HashSet` → fast lookup, no order needed
- `LinkedHashSet` → fast lookup + insertion order preserved
- `TreeSet` → sorted order needed
- `HashMap` → fast key-value lookup
- `LinkedHashMap` → key-value + insertion order
- `TreeMap` → key-value + sorted by key

---

### Thread-Safe Collections

```java
// Option 1: synchronized wrapper (Collections utility class)
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());
// Note: iteration still needs external synchronization

// Option 2: java.util.concurrent classes (preferred — better performance)
Map<String, Integer> concurrentMap  = new ConcurrentHashMap<>();
List<String>         cowList        = new CopyOnWriteArrayList<>();
Queue<String>        blockingQueue  = new LinkedBlockingQueue<>();
Deque<String>        concurrentDeque = new ConcurrentLinkedDeque<>();
```

**`ConcurrentHashMap` vs `Collections.synchronizedMap`:**
- `synchronizedMap` — wraps entire map; locks the **whole map** on every operation → one thread at a time
- `ConcurrentHashMap` — uses **segment/bucket-level locking** → multiple threads can read/write different buckets simultaneously → much higher throughput
- Use `ConcurrentHashMap` in Spring Boot for in-memory caches, rate limiters, shared counters

**`CopyOnWriteArrayList` — when to use:**
- Thread-safe list where reads vastly outnumber writes
- Reads: lock-free (very fast). Writes: copy the entire array (expensive)
- Good for event listener lists, configuration lists that rarely change

---

## Interview Q&A

**Q: What is a thread?**
The smallest unit of execution inside a process. A process (Spring Boot app) can have many threads running concurrently — sharing heap memory but each with its own stack. Tomcat creates a thread per HTTP request from its thread pool.

**Q: Why are threads useful?**
Parallel tasks, better CPU utilization, faster response handling, async processing. In microservices: parallel external API calls, background jobs, async email, scheduled tasks, long-running LLM operations.

**Q: `start()` vs `run()`?**
`start()` creates a new OS thread and calls `run()` inside it — true concurrency.
`run()` called directly is just a regular method call in the current thread — no new thread created, no concurrency.

**Q: Why prefer Runnable over extending Thread?**
Java has single inheritance — extending `Thread` blocks extending any other class. `Runnable` separates the task from the thread mechanism. Better design (single responsibility). Easier to use with `ExecutorService` and lambdas.

**Q: What is a race condition?**
When multiple threads read-modify-write shared data without synchronization, causing lost updates or inconsistent state. Example: two threads both read `count = 0`, both add 1, both write `count = 1` — expected 2, got 1.

**Q: What is `synchronized`?**
Acquires a monitor lock on an object — only one thread enters the critical section at a time. Prevents race conditions but adds serialization overhead (threads wait). Use at method or block level.

**Q: `sleep()` vs `wait()`?**
`sleep()` — pauses current thread; holds any locks; `Thread` class; wakes after time.
`wait()` — releases the monitor lock; waits until `notify()`; `Object` class; must be inside `synchronized`.

**Q: What is deadlock?**
Two threads each holding a lock the other needs, waiting forever. Prevention: always acquire locks in same order, use `tryLock()` with timeout, minimize lock scope, prefer immutable objects.

**Q: `Collection` vs `Collections` vs Collection API?**
- **Collection (interface):** root of List/Set/Queue hierarchy — not Map
- **Collections (class):** utility class with static methods: `sort()`, `reverse()`, `synchronizedList()`, etc.
- **Collection API (framework):** the whole thing — all interfaces, implementations, algorithms, including Map

**Q: Where have you used multithreading in backend?**
- `@Async` for email/SMS/push notifications — main request returns immediately
- Background report generation — heavy processing offloaded to thread pool
- `CompletableFuture.allOf()` for parallel API calls to external services
- `@Scheduled` for periodic cleanup, job processing, cache refresh
- Long-running LLM inference tasks offloaded to dedicated thread pool

**Q: How do you handle concurrency in production?**
- `ExecutorService` / `@Async` instead of raw threads
- Immutable DTOs passed between layers — no shared mutable state
- `ConcurrentHashMap` for shared in-memory state (caches, counters)
- `synchronized` only for small, critical sections
- DB-level optimistic locking (`@Version` + `OptimisticLockException`) for inventory/balance
- Stateless services where possible — easier to scale horizontally

**Q: Why is `ConcurrentHashMap` better than `synchronizedMap`?**
`synchronizedMap` acquires a lock on the **entire map** for every operation — one thread at a time for any key.
`ConcurrentHashMap` uses bucket-level locking — threads working on different keys don't block each other. Much higher throughput under concurrent access — critical for shared caches in high-traffic microservices.

**Q: What is `volatile`?**
Ensures that reads and writes to a variable go directly to main memory — prevents threads from caching stale values in CPU registers. Use for flags (`volatile boolean running`) that are read/written by multiple threads. Does NOT make compound operations (like `count++`) atomic — use `AtomicInteger` for that.
