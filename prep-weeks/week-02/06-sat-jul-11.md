# Week 2 · Day 6 — Sat Jul 11 — Timed DSA + Concurrency Deep-Dive + LLD Primer

> The week's capstone. Block A is a strict timed set — two cold problems under interview conditions. Block B is the concurrency model behind every Spring app you've shipped (`synchronized` vs locks, `volatile`, `CompletableFuture`, `ThreadLocal`, deadlock) plus your first object-oriented design rep: the parking lot, framed through SOLID. Block C consolidates the week and connects every concept to a real line of Smart360 / WebX / API Gateway code.

📌 **Study today:** Timed DSA set (2 cold problems) · concurrency deep-dive (synchronized/locks/volatile/CompletableFuture/ThreadLocal/deadlock) + LLD primer (parking lot + SOLID) · consolidation · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Timed Set (interview conditions)

Timer running, no hints, clean whiteboard-quality code. State the approach aloud before coding; handle edge cases unprompted.

- **Problem A (20 min):** re-solve **Koko Eating Bananas (LC 875)** or **Search a 2D Matrix (LC 74)** cold.
- **Problem B (25 min):** re-solve **Subsets (LC 78)** or **Daily Temperatures (LC 739)** cold.

Score yourself on four axes: (1) stated approach before coding, (2) handled edge cases unprompted, (3) finished in time, (4) code compiled without fixes. Write one line on what you'd do differently.

### Reference solution — Koko Eating Bananas (LC 875)

Binary-search-on-answer: the answer is an eating speed, the space is `[1, max(piles)]`, and `feasible(speed)` is monotonic (faster speed → fewer hours), so binary-search the speed.

```java
public int minEatingSpeed(int[] piles, int h) {
    int left = 1, right = 0;
    for (int p : piles) right = Math.max(right, p);   // upper bound = biggest pile
    while (left < right) {                            // boundary-finding: left < right
        int mid = left + (right - left) / 2;
        if (canFinish(piles, mid, h)) right = mid;    // feasible → try slower
        else                          left = mid + 1; // too slow → speed up
    }
    return left;                                      // smallest feasible speed
}
private boolean canFinish(int[] piles, int speed, int h) {
    long hours = 0;
    for (int p : piles) hours += (p + speed - 1) / speed; // ceil(p/speed)
    return hours <= h;
}
```
**Complexity:** O(n · log(max pile)). **Edge cases:** single pile; `h == piles.length` (must eat the max pile in one hour each → speed = max). Note `left < right` with `right = mid` (mid stays a candidate).

### Reference solution — Subsets (LC 78)

```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> res = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), res);
    return res;
}
private void backtrack(int[] nums, int start, List<Integer> path, List<List<Integer>> res) {
    res.add(new ArrayList<>(path));            // every node is a valid subset — add a COPY
    for (int i = start; i < nums.length; i++) {
        path.add(nums[i]);                     // choose
        backtrack(nums, i + 1, path, res);     // recurse with i+1 (forward-only)
        path.remove(path.size() - 1);          // un-choose (restore)
    }
}
```
**Complexity:** O(2ⁿ · n) — 2ⁿ subsets, each copied in O(n). The copy on `res.add` is mandatory; adding `path` directly stores a reference that gets mutated.

---

## Block B — Concurrency Deep-Dive + LLD Primer

### `synchronized` vs `ReentrantLock`

`synchronized` is a JVM keyword: it acquires and **automatically releases** the monitor, even when the method throws. `ReentrantLock` is an explicit object — you **must** `unlock()` in a `finally` or you leak the lock forever.

```java
private final ReentrantLock lock = new ReentrantLock();
public void doWork() {
    lock.lock();
    try { /* critical section */ }
    finally { lock.unlock(); } // ALWAYS in finally
}
```

`ReentrantLock` advantages: `tryLock(timeout)` (deadlock avoidance), `lockInterruptibly()`, fairness (`new ReentrantLock(true)`), and **multiple condition variables** (`lock.newCondition()`). `ReentrantReadWriteLock` allows many readers OR one writer — ideal for a read-heavy in-memory map.

> **Project tie-in (WebX):** `tryLock(5, SECONDS)` on LLM-provider slots returns 503 instead of blocking a request thread forever when all providers are busy; `ReentrantReadWriteLock` guards the provider-routing table (read on every request, written rarely on config reload).

### `volatile`

`volatile` guarantees **visibility** (writes are seen by other threads immediately) and **ordering** (no reordering across the access), but **not atomicity**.

```java
private volatile boolean running = true; // VALID — single write, single read
public void shutdown() { running = false; }
public void loop() { while (running) { /* ... */ } }

private volatile int counter; // INVALID for counter++ — read-modify-write race
// use AtomicInteger instead:
private final AtomicInteger safe = new AtomicInteger();
safe.incrementAndGet(); // CAS, atomic
```
JMM rule: a `volatile` write *happens-before* every subsequent `volatile` read of the same variable. Use it for flags, not for compound operations (counters, check-then-act).

### The concurrency toolbox (know the trade-offs)

| Tool | Guarantee | Cost | Use |
|---|---|---|---|
| `volatile` | visibility + ordering | ~free | single-writer flags |
| `synchronized` / `ReentrantLock` | mutual exclusion | blocking | compound critical sections |
| `Atomic*` | atomic CAS | lock-free, spins under contention | counters, refs |
| `ConcurrentHashMap` | lock-free reads, bucket-CAS writes | low | shared maps — never `Collections.synchronizedMap` |

### `CompletableFuture`

```java
// ALWAYS pass an explicit Executor in production — default uses the shared common pool
ExecutorService pool = Executors.newFixedThreadPool(40);
CompletableFuture<LLMResult> job = CompletableFuture
    .supplyAsync(() -> callProvider(req), pool)   // run on dedicated pool
    .thenApply(this::postProcess)                 // transform
    .thenCompose(r -> enrich(r, pool))            // flat-map (avoid nested futures)
    .exceptionally(ex -> LLMResult.fallback());   // recover
```
- `thenApply` transform, `thenCompose` flat-map, `thenCombine` combine two, `allOf`/`anyOf` fan-out/race, `exceptionally`/`handle` recover.
- Default `supplyAsync` uses `ForkJoinPool.commonPool()` — **always pass an explicit `Executor`** for blocking I/O (a slow LLM call starves the shared pool that parallel streams also use).
- `join()` (unchecked `CompletionException`) is preferred over `get()` (checked `ExecutionException`/`InterruptedException`) inside lambda chains.

> **Project tie-in (WebX):** each job is a `CompletableFuture` in `ConcurrentHashMap<jobId, CompletableFuture<LLMResult>>`; poll `isDone()`, `complete(result)` when the provider responds, multi-provider fan-out via `allOf`.

### `ThreadLocal`

Each thread gets an isolated copy. Tomcat = one thread per request, so Spring Security's `SecurityContextHolder` stores the `Authentication` in a `ThreadLocal`.

```java
private static final ThreadLocal<String> TENANT = new ThreadLocal<>();
// in a filter:
try { TENANT.set(tenantId); chain.doFilter(req, res); }
finally { TENANT.remove(); } // CRITICAL — Tomcat reuses threads
```
**The gotcha:** Tomcat pools and **reuses** threads. If you `set` a value and forget `remove()`, the next request on that thread inherits stale state — a cross-tenant data leak *and* a memory leak. Spring Security calls `clearContext()` in its filter; do the same for every custom `ThreadLocal`. `ThreadLocal` does **not** propagate across `@Async` boundaries — pass the value explicitly rather than relying on `InheritableThreadLocal`.

> **War story:** a super-admin's auth once bled into the next request on the same pooled thread because a custom `ThreadLocal` wasn't removed — the fix was a `finally { remove(); }`.

### Deadlock

```java
// Thread 1                    // Thread 2
synchronized (lockA) {         synchronized (lockB) {
    synchronized (lockB) {}        synchronized (lockA) {}
}                              }
// T1 holds A waits B; T2 holds B waits A → deadlock
```
Four **Coffman conditions** (all required): mutual exclusion, hold-and-wait, no preemption, circular wait. The most practical fix: **break circular wait** — always acquire locks in a global order (e.g. sort resources by ID before locking). Other tools: `tryLock(timeout)` to back off, single coarse lock, lock-free structures.

### LLD Primer — "Design a Parking Lot"

The canonical OOD prompt. It is not about a single right answer — it scores **clean responsibilities, interfaces over concretions, and naming which SOLID principle each choice serves.** Keep your `02/03` OOP notes open.

**Entities & responsibilities:**
- `ParkingLot` — orchestration only: `parkVehicle(...)`, `unpark(...)`. No pricing, no spot-finding logic inside it.
- `Level` — owns its `Spot`s, knows availability.
- `Spot` — `{id, SpotType, occupied}`; `canFit(Vehicle)` lives **here**, not in the lot.
- `Vehicle` — abstract base → `Car` / `Truck` / `Motorcycle` / `ElectricCar`.
- `Ticket` — `{id, Spot, Vehicle, entryTime, exitTime}`.

**Interfaces / seams (inject both into `ParkingLot`):**
- `ParkingStrategy` — `Spot findSpot(Level, Vehicle)` → `NearestFirst` / `BestFit`.
- `PricingStrategy` — `BigDecimal price(Ticket)` → `FlatHourly` / `TieredByVehicleType` / `WeekendSurge`.

```java
interface ParkingStrategy { Optional<Spot> findSpot(Level level, Vehicle v); }
interface PricingStrategy { BigDecimal price(Ticket ticket); }

class ParkingLot {
    private final List<Level> levels;
    private final ParkingStrategy parking;   // DIP — depend on the interface
    private final PricingStrategy pricing;   //       (constructor-injected, like Spring DI)
    ParkingLot(List<Level> levels, ParkingStrategy p, PricingStrategy pr) {
        this.levels = levels; this.parking = p; this.pricing = pr;
    }
    Ticket parkVehicle(Vehicle v) { /* delegate spot-finding to parking */ }
    BigDecimal unpark(Ticket t)   { t.markExit(); return pricing.price(t); }
}
```

**Call out the SOLID letter for each decision (this scores points):**
- **SRP** — pricing isn't in `Ticket`; spot-finding isn't in `ParkingLot`.
- **OCP** — a new `SpotType` / `Vehicle` / strategy is a new class, not edits to existing ones.
- **LSP** — any `Vehicle` subtype is substitutable wherever `Vehicle` is expected.
- **ISP** — separate small `ParkingStrategy` / `PricingStrategy` interfaces, not one fat manager.
- **DIP** — `ParkingLot` depends on the strategy *interfaces*, constructor-injected (same pattern as Spring DI).

**Curveballs to rehearse:**
- Add EV charging → new `SpotType` + a `canFit` rule (OCP).
- Weekend pricing → a new `PricingStrategy` (OCP).
- Concurrent gates → a **per-`Level` lock** or a `ConcurrentHashMap` of availability — ties to the concurrency block, **not** one lot-wide `synchronized` that serializes all gates.

**Other LLD sketches (name the entities + the key strategy interface):** Elevator (`SchedulingStrategy`), Vending machine (State pattern), Library (`PricingStrategy` for fines), Rate limiter (`TokenBucket` / `SlidingWindow` — the OO form of the Smart360 gateway Redis logic), Splitwise (`Split` hierarchy: equal/percent/exact).

---

## Block C — Consolidation

1. Write, without notes, the **8-step bean lifecycle** and the three init hooks (`@PostConstruct` → `InitializingBean.afterPropertiesSet()` → `init-method`) in order, then check against your week notes.
2. Connect each concept to one line of real code:
   - **Smart360:** `@Transactional(propagation = REQUIRES_NEW)` for audit-log writes; a custom `ThreadPoolExecutor` for parallel S3 URL caching; N+1 fix via `@EntityGraph` inside a `REQUIRED` tx; `@Transactional(readOnly = true)` to skip dirty checking.
   - **WebX:** `@Async` with a bounded `ThreadPoolTaskExecutor`; `CompletableFuture` multi-provider fan-out; a `volatile boolean` shutdown flag; a `ThreadLocal` tenant ID cleared after each job.
   - **API Gateway:** `@ConditionalOnProperty` to toggle the circuit breaker per environment; a CGLIB proxy on filter beans.
3. Open any Spring Boot app, add `debug=true`, run it, and read the **CONDITIONS EVALUATION REPORT** — note 3 auto-configs that matched and 3 that didn't, and why.

---

## 💻 Practice coding questions

1. Koko Eating Bananas (LC 875) — timed, cold.
2. Search a 2D Matrix (LC 74) — flat-index binary search.
3. Subsets (LC 78) — backtracking with copy-on-add.
4. Daily Temperatures (LC 739) — monotonic decreasing stack.
5. Implement a thread-safe counter with `AtomicInteger` and with a `ReentrantLock`; benchmark under contention.
6. Implement a producer/consumer with a `BlockingQueue` and again with `ReentrantLock` + two `Condition`s.
7. Write a `CompletableFuture` fan-out over 3 providers, returning the first success (`anyOf`) and all results (`allOf`).
8. Reproduce a deadlock with two locks, then fix it by global lock ordering.
9. Write a tenant `ThreadLocal` filter with `set`/`remove` and prove the leak if `remove` is omitted.
10. Sketch the Parking Lot in code: `ParkingLot`, `Level`, `Spot`, `Vehicle` hierarchy, `ParkingStrategy`, `PricingStrategy`.
11. Add an `ElectricCar` + EV `SpotType` to the parking lot without editing existing classes (prove OCP).
12. Implement a `TokenBucket` rate limiter as an OO class with a `tryAcquire()` method.

---

## 🎤 Interview questions

1. **`synchronized` vs `ReentrantLock` — when ReentrantLock?** When you need `tryLock(timeout)`, interruptible locking, fairness, or multiple conditions. WebX used `tryLock(5, SECONDS)` to return 503 instead of blocking.
2. **What does `volatile` guarantee, and what doesn't it?** Visibility + ordering, not atomicity. Good for a shutdown flag, wrong for `counter++`.
3. **Why is `counter++` unsafe even when `counter` is `volatile`?** It's a read-modify-write; two threads can read the same value and both increment to the same result. Use `AtomicInteger`.
4. **`volatile` vs `synchronized` vs `Atomic*` vs `ConcurrentHashMap`?** Visibility-only / mutual exclusion / lock-free CAS / lock-free reads with bucket-CAS writes. Pick the cheapest that meets the guarantee.
5. **Default executor of `CompletableFuture.supplyAsync` — why is that a trap?** `ForkJoinPool.commonPool()` is shared with parallel streams; a slow blocking call starves it. Pass an explicit `Executor`.
6. **`thenApply` vs `thenCompose`?** `thenApply` maps a value; `thenCompose` flat-maps a function that itself returns a `CompletableFuture` (avoids nesting).
7. **`future.get()` vs `future.join()`?** `get()` throws checked `ExecutionException`/`InterruptedException`; `join()` throws unchecked `CompletionException` — cleaner in lambda chains.
8. **How does `ThreadLocal` interact with Tomcat's thread pool?** Threads are reused; an un-removed value leaks to the next request → cross-tenant leak + memory leak. Always `remove()` in `finally`.
9. **Does `ThreadLocal` propagate to `@Async` threads?** No — different thread. Pass the value explicitly; `InheritableThreadLocal` only copies at thread creation, not for pooled threads.
10. **Write a deadlock and prevent it.** Two threads, opposite lock order. Fix: global lock ordering (sort by resource id) or `tryLock` with backoff.
11. **Name the four Coffman conditions.** Mutual exclusion, hold-and-wait, no preemption, circular wait — break any one to prevent deadlock.
12. **Tomcat gets 1000 concurrent requests — what happens at the pool level?** 200 worker threads (default `server.tomcat.threads.max`), then the `accept-count` queue (default 100), then connections refused. Scale replicas before that.
13. **How would you size a thread pool for LLM jobs?** Little's Law: throughput × latency = concurrency. ~5 req/min × 8 min ≈ 40 → `corePoolSize=40, maxPoolSize=60`, bounded queue, plus a saturation metric.
14. **`ForkJoinPool` vs a dedicated `ThreadPoolExecutor`?** ForkJoin (work-stealing) for CPU-bound divide-and-conquer; a dedicated pool for blocking I/O so it doesn't starve the common pool.
15. **Parking Lot — what classes and where does `canFit` live?** `ParkingLot`/`Level`/`Spot`/`Vehicle`/`Ticket`; `canFit(Vehicle)` lives on `Spot`.
16. **Which SOLID principle does injecting `ParkingStrategy`/`PricingStrategy` serve?** DIP (depend on interfaces) and OCP (new strategy = new class, no edits).
17. **Add EV charging to the parking lot without breaking existing code — how?** New `SpotType` + a `canFit` rule and/or a new strategy — OCP, no edits to existing classes.
18. **Concurrent gates parking simultaneously — how do you avoid double-booking a spot without serializing everything?** Per-`Level` lock or a `ConcurrentHashMap` of availability with CAS, not one lot-wide `synchronized`.
19. **Where do AOP/`@Transactional` proxies appear in the bean lifecycle?** Step 6, `postProcessAfterInitialization`, by `AnnotationAwareAspectJAutoProxyCreator`; the injected bean *is* the proxy.
20. **Connect `ThreadLocal` to a security feature you've used.** `SecurityContextHolder` is `ThreadLocal`-backed; the same pattern stored `TenantContext`, always `remove()`d in a `finally`.

---

## ✅ Self-check

1. Tomcat gets 1000 concurrent requests — describe exactly what happens at the thread-pool level (200 max threads, `accept-count` 100 queue, then refused).
2. Difference between `future.get()` and `future.join()` (checked `ExecutionException` vs unchecked `CompletionException`).
3. Write the two-lock deadlock from memory and state the global-ordering fix.
4. Sketch the Parking Lot entities and name one SOLID letter per design decision.
5. From memory, write the 8-step bean lifecycle and the three init hooks in order.

---

*Nav: ← [Day 5 (Fri Jul 10)](05-fri-jul-10.md) · [Week 2](README.md) · [Week 3](../week-03/) →*
