# Week 1 · Day 6 — Saturday Jul 04 — Timed Mixed DSA + SOLID / System-Design Primer + Resume & Deep-Dive Docs

> **Consolidation day.** Lighter on new algorithms — *one* timed mixed set under interview conditions — and heavier on the durable assets you carry into every interview: SOLID grounded in *your own* code, a system-design primer with real latency numbers, and the resume rewrite + one-page project deep-dive docs that turn your shipped work into stories that survive a recruiter screen *and* a staff-engineer drill-down. End the day by scoring the week's self-assessment gate.

📌 **Study today:** Timed mixed DSA set (LC 76 · 152 · 239 · 452) · SOLID + system-design primer · resume rewrite start + project deep-dive docs · (self-study: JVM memory model + concurrency basics) · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Timed Mixed Set (~2.5 h)

**Conditions:** 25 minutes each, no hints, no peeking. After each problem write down (1) the pattern name, (2) time/space complexity, (3) one interviewer follow-up. This is a dress rehearsal — the goal is *pattern recognition under a clock*, not learning new tricks.

### 1. Minimum Window Substring (LC 76) — hardest sliding window

Two frequency maps and a **"have vs need"** counter. Expand `right` until the window covers all required chars, then shrink `left` while still valid, recording the minimum.

```java
// Java 17
public String minWindow(String s, String t) {
    if (s.length() < t.length()) return "";
    int[] need = new int[128];
    for (char c : t.toCharArray()) need[c]++;
    int required = t.length();                 // total chars still needed (counts duplicates)
    int left = 0, bestLen = Integer.MAX_VALUE, bestStart = 0;

    for (int right = 0; right < s.length(); right++) {
        if (need[s.charAt(right)]-- > 0) required--;   // consumed a still-needed char
        while (required == 0) {                          // window valid -> try to shrink
            if (right - left + 1 < bestLen) { bestLen = right - left + 1; bestStart = left; }
            if (need[s.charAt(left)]++ == 0) required++; // about to drop a needed char
            left++;
        }
    }
    return bestLen == Integer.MAX_VALUE ? "" : s.substring(bestStart, bestStart + bestLen);
}   // O(|s| + |t|) time, O(1) space (fixed 128 array)
```
**Why the counter trick works:** `need[c]` goes negative for surplus chars, so `need[c]-- > 0` only decrements `required` when the char was genuinely still needed. Symmetric logic on shrink. Pattern: variable sliding window. Follow-up: "Unicode?" → swap `int[128]` for a `HashMap<Character,Integer>`.

### 2. Maximum Product Subarray (LC 152) — Kadane with a twist

Track **both** the running max and min, because a negative × a previous negative becomes a new max.

```java
public int maxProduct(int[] nums) {
    int max = nums[0], min = nums[0], best = nums[0];
    for (int i = 1; i < nums.length; i++) {
        int n = nums[i];
        if (n < 0) { int t = max; max = min; min = t; }  // negative flips roles
        max = Math.max(n, max * n);
        min = Math.min(n, min * n);
        best = Math.max(best, max);
    }
    return best;
}   // O(n) / O(1)
```
Pattern: Kadane extension. Follow-up: "Why track the min?" → a future negative can turn the smallest (most negative) product into the largest.

### 3. Sliding Window Maximum (LC 239) — monotonic deque

A deque of **indices** kept in decreasing value order; the front is always the current window's max.

```java
public int[] maxSlidingWindow(int[] nums, int k) {
    Deque<Integer> dq = new ArrayDeque<>();   // holds indices, values decreasing front->back
    int[] res = new int[nums.length - k + 1];
    for (int i = 0; i < nums.length; i++) {
        if (!dq.isEmpty() && dq.peekFirst() <= i - k) dq.pollFirst();   // drop out-of-window
        while (!dq.isEmpty() && nums[dq.peekLast()] < nums[i]) dq.pollLast(); // drop smaller
        dq.offerLast(i);
        if (i >= k - 1) res[i - k + 1] = nums[dq.peekFirst()];
    }
    return res;
}   // O(n) amortized (each index pushed/popped once), O(k) space
```
Pattern: monotonic deque. Follow-up: "Why O(n) with the inner while?" → each index enters and leaves the deque at most once.

### 4. Minimum Number of Arrows to Burst Balloons (LC 452) — greedy interval

Sort by **end**, shoot at the first balloon's end, skip everything it bursts.

```java
public int findMinArrowShots(int[][] points) {
    Arrays.sort(points, (a, b) -> Integer.compare(a[1], b[1]));  // by end; never a[1]-b[1] (overflow!)
    int arrows = 1, end = points[0][1];
    for (int[] p : points) {
        if (p[0] > end) { arrows++; end = p[1]; }   // current arrow can't reach -> new arrow
    }
    return arrows;
}   // O(n log n) sort + O(n) scan
```
Pattern: greedy / interval scheduling (preview of next week's interval set). Follow-up: "Sort by start instead?" → works too but the end-sort greedy is cleaner; you'd track the minimum end.

> Note the `Integer.compare` in the sort — a `a[1]-b[1]` lambda overflows for large coordinates and silently misorders (the Friday Comparator gotcha, live).

---

## Block B — SOLID + System-Design Primer (~2 h)

### SOLID — anchored in *your* code (say each with a real example)

- **SRP (Single Responsibility)** — Smart360's `AuthorizationService` originally did token validation *and* role resolution; when it also started handling user-management events it had three reasons to change → the **strangler-fig extraction** story (peel responsibilities into separate services incrementally, route old calls to new components).
- **OCP (Open/Closed)** — WebX's LLM provider router: adding a provider means *implementing* the `LLMProvider` interface, never *editing* the router. Open for extension, closed for modification.
- **LSP (Liskov Substitution)** — `PremiumUser extends User` must work anywhere a `User` works. Classic violation: overriding a method to `throw new UnsupportedOperationException()` — exactly the `java.util.Stack extends Vector` mistake (`stack.add(0, e)` corrupts stack semantics).
- **ISP (Interface Segregation)** — split a fat 20-method `UserService` into `UserReader` / `UserWriter` / `UserAuthenticator` so clients depend only on what they use.
- **DIP (Dependency Inversion)** — Spring DI: depend on the `UserRepository` interface, not `UserRepositoryImpl`. High-level policy and low-level detail both depend on the abstraction.
- **Inheritance vs Composition** — prefer composition (interfaces + delegation); inheritance creates tight coupling and the fragile-base-class problem. Use inheritance *only* for a genuine IS-A.

```java
// OCP in action — new provider = new class, router untouched
interface LLMProvider { Response complete(Prompt p); }
class OpenAIProvider   implements LLMProvider { /* ... */ }
class AnthropicProvider implements LLMProvider { /* ... */ }
// Router depends on the abstraction; adding a provider never edits this class.
class Router {
    private final Map<String, LLMProvider> providers;   // DIP + OCP together
    Router(Map<String, LLMProvider> providers) { this.providers = providers; }
}
```

### System-design primer

**The latency numbers every engineer should know (order of magnitude):**

| Operation | Latency |
|-----------|---------|
| L1 cache reference | ~1 ns |
| L2 cache reference | ~4 ns |
| Main memory (RAM) | ~100 ns |
| SSD random read | ~100 µs |
| Same-datacenter network round trip | ~500 µs |
| Cross-region round trip | ~150 ms |

The takeaway: **a cross-region call is ~1.5 million times slower than an L1 hit** — which is *why* caching, locality, and avoiding chatty cross-service calls dominate design.

**Caching:** cache-aside (lazy — read DB on miss, populate cache; Smart360's default), write-through (write cache + DB synchronously), write-behind (write cache, flush DB async). Eviction: LRU / LFU / TTL. **Cache stampede** (many concurrent misses on one hot key hammer the DB) → fix with probabilistic early expiry or single-flight (one fetch, others wait). **CDN:** edge caching; pull vs push; prefer fingerprinted URLs (`app.a1b2c3.js`) over purge.

**Load balancing:** L4 (TCP) vs L7 (HTTP, can route on path/header); round-robin / least-connections / consistent hashing. **Sticky sessions are an anti-pattern** — use stateless JWT so any node can serve any request. **Horizontal vs vertical scaling** — Smart360 runs on Azure Container Apps + KEDA with scale-to-zero (horizontal, event-driven).

**CAP theorem:** during a network partition you must choose Consistency *or* Availability. PostgreSQL = CP; Cassandra / DynamoDB = AP (tunable). It's a *partition-time* trade-off, not an always-on one.

### Smart360 5-service whiteboard exercise

Redraw from memory. For each service state (1) single responsibility, (2) data owned, (3) sync vs async calls, (4) failure mode. Target: narrate the whole topology in **under 3 minutes.**

1. **API Gateway** (Spring Cloud Gateway) — JWT validation, rate limiting (Redis), routing. Stateless. SPOF → run replicas behind the LB.
2. **Authorization** — authentication, JWT issuance, RBAC. Owns `users / roles / permissions`. Publishes `UserCreated` / `UserUpdated` events.
3. **Data** — core CRUD + the **60s → 2–3s query fix**. Owns `reports / s3_file_metadata / organizations`. Redis pre-signed-URL cache; `@EntityGraph` to kill N+1. Degrades to cached reads if Redis is down.
4. **Visualization** — aggregations / chart data (CQRS read side). Scales independently so heavy aggregations don't choke the OLTP path.
5. **Notification** (event-driven) — subscribes to events; async / fire-and-forget so a notification failure never blocks user registration.

Cross-cutting: Eureka / K8s DNS service discovery, Micrometer distributed tracing, centralized Config, PostgreSQL Row-Level Security.

---

## Block C — Resume Rewrite Start + Project Deep-Dive Docs (~1.5 h)

**Rule for every bullet:** answer "So what? How much? How do I know?" Kill any bullet without a **number** or a **named tech decision.**

### Smart360
- "Cut data-retrieval latency **96% (60s → 2–3s)** by eliminating N+1 Hibernate queries via `@EntityGraph`, composite indexes, and caching S3 pre-signed URLs in Redis (**80% S3 reduction**)."
- "Reduced sign-in latency **60%** by migrating session validation to stateless JWT."
- "Built fine-grained RBAC via Spring Security + JWT claims across **5 microservices**."

### Deep Fathom
- "Implemented PostgreSQL **Row-Level Security across 50+ tables** for tenant isolation enforced at the DB layer."
- "Reduced CI/CD runtime **57% (23 → 10 min)** via BuildKit parallel builds, registry layer caching, and a GitLab DAG restructure."
- "Authored **Bicep IaC across 30+ resources**, producing identical staging and FedRAMP GovCloud environments from one codebase."

### WebX
- "Designed an async job architecture for **2–20 min LLM operations**: `202 + jobId`, SSE push on completion; **5+ providers** behind a unified proxy."
- "Added circuit-breaker graceful degradation that falls back to a cheaper provider on outages."

### One-page deep-dive doc per project (your verbal cheat sheet)

Structure each: **What / Scale / Architecture / Key decisions (with trade-offs) / Hardest problem + fix / Numbers / What I'd do differently.**

> Example — Smart360 "what I'd do differently": *K8s-native service discovery instead of Eureka* (redundant infra once K8s arrived); *Postgres advisory locks instead of Redis* for the token blacklist (one fewer moving part). Naming what you'd change signals senior judgment, not weakness.

### Self-study note — keep JVM memory model + concurrency basics warm (~30 min skim, no separate block)

These return for real in **Week 3's concurrency block** — today just don't let them go cold:
- **Memory regions:** Heap / Stack / Metaspace / Code Cache / Direct Memory.
- **G1GC:** generational hypothesis, `-XX:MaxGCPauseMillis` target.
- **String pool & `intern()`.**
- **`volatile`** = visibility (not atomicity) vs **`AtomicInteger`** (CAS atomicity).
- **`synchronized`** vs **`ReentrantLock`** (`tryLock`, fairness, interruptibility).
- **`ThreadLocal`** — Spring's `SecurityContextHolder`; pool-leak risk if not cleared.
- **`CompletableFuture`** — `thenApply` / `thenCompose` / `allOf` / `exceptionally`; pick the executor explicitly.

See [06-concurrency-and-collections.md](../../06-concurrency-and-collections.md) and [04-exception-handling-and-memory.md](../../04-exception-handling-and-memory.md).

---

## 💻 Practice coding questions

1. **Minimum Window Substring (LC 76)** — two-map / have-vs-need counter (code above).
2. **Maximum Product Subarray (LC 152)** — track max AND min, swap on negative.
3. **Sliding Window Maximum (LC 239)** — monotonic deque of indices.
4. **Minimum Arrows to Burst Balloons (LC 452)** — greedy, sort by end.
5. **Merge Intervals (LC 56)** — sort by start, merge overlapping; another interval warm-up.
6. **Non-overlapping Intervals (LC 435)** — greedy, sort by end, count removals (mirror of LC 452).
7. **Longest Substring Without Repeating Chars (LC 3)** — recap variable window.
8. **Maximum Subarray (LC 53)** — recap Kadane (contrast with LC 152).
9. **Implement an `LLMProvider` interface + 2 impls + a router** — OCP/DIP in code, no `if/else` provider switch.
10. **Refactor a 20-method `UserService` into segregated interfaces** — ISP exercise.
11. **Spot the LSP violation** in a `Square extends Rectangle` (setWidth/setHeight) snippet and fix it.
12. **Design a cache-aside read path** (pseudocode): check cache → on miss read DB → populate cache with TTL → return; add stampede protection.

---

## 🎤 Interview questions

1. **Explain SRP with your own code.** Smart360 `AuthorizationService` doing validation + roles + user events → strangler-fig extraction into focused services.
2. **OCP — how do you add a feature without editing existing code?** Program to an interface; new behavior = new implementation. WebX provider router.
3. **Give a real LSP violation.** `Stack extends Vector` — `add(0,e)` breaks stack invariants; or overriding to throw `UnsupportedOperationException`.
4. **ISP in practice?** Split fat interfaces so clients depend only on methods they call (`UserReader`/`UserWriter`).
5. **DIP vs DI?** DIP is the principle (depend on abstractions); DI (Spring) is one mechanism to achieve it.
6. **Inheritance vs composition — default?** Composition; inheritance only for true IS-A. Avoids fragile base class + tight coupling.
7. **Latency vs throughput?** Latency = time per operation; throughput = operations per second. You can trade one for the other (batching raises throughput, raises per-item latency).
8. **Rough latency: RAM vs SSD vs same-DC vs cross-region?** ~100 ns / ~100 µs / ~500 µs / ~150 ms. Cross-region is ~1.5M× an L1 hit.
9. **Cache-aside vs write-through vs write-behind?** Lazy populate on miss; synchronous cache+DB write; async DB flush. Smart360 uses cache-aside.
10. **What is a cache stampede and how do you prevent it?** Many concurrent misses on a hot key hammer the DB; fix with single-flight or probabilistic early expiry.
11. **L4 vs L7 load balancing?** L4 routes on TCP/IP; L7 can route on HTTP path/header/cookie. Sticky sessions are an anti-pattern — prefer stateless JWT.
12. **CAP — what does PostgreSQL choose?** CP — consistency over availability during a partition. Cassandra/DynamoDB lean AP.
13. **Walk the Smart360 topology in under 3 minutes.** Gateway → Authorization → Data → Visualization → Notification, with sync/async arrows and failure modes.
14. **A technical decision you regret.** Eureka over K8s-native discovery in Smart360 — redundant once K8s arrived; lesson: don't duplicate what the platform provides.
15. **How did you cut Smart360 latency 96%?** Eliminated N+1 via `@EntityGraph`, added composite indexes, cached S3 pre-signed URLs in Redis (60s → 2–3s).
16. **Why PostgreSQL RLS over app-layer tenant filtering?** Enforced by the DB even if the app has a bug; trade-off is setting the tenant context per connection. (Test with Testcontainers — H2 lacks RLS.)
17. **LLM job runs 20 min and the user closes the browser — what happens?** Job is persisted in PostgreSQL, the worker continues, the user re-polls `GET /jobs/{id}`; SSE drops but the result is durable.
18. **How would you push CI/CD from 10 min to 5?** More aggressive Docker/test layer caching, test-impact analysis, more parallel runners, move slow integration tests to nightly.
19. **`@Transactional` self-invocation — why no transaction?** A same-class internal call bypasses the Spring proxy; fix by injecting self or `AopContext.currentProxy()`.
20. **`volatile` vs `AtomicInteger`?** `volatile` gives visibility but not compound-action atomicity; `AtomicInteger` gives CAS-based atomic increments. (Week 3 deep-dive.)

---

## ✅ Self-check — END-OF-WEEK GATE (pass/fail, no partial credit)

Complete this Saturday evening. Honest pass/fail only.

| Gate | Check | Pass criteria |
|------|-------|---------------|
| DSA-1 | Container With Most Water cold | ≤ 15 min, correct O(n) |
| DSA-2 | Minimum Size Subarray Sum cold | ≤ 15 min, correct sliding window |
| DSA-3 | 3Sum cold | ≤ 25 min, handles duplicates |
| DSA-4 | Explain Kadane in 60 sec | States invariant + all-negative case |
| DSA-5 | Top K Frequent with bucket sort (no heap) | ≤ 20 min, O(n) |
| DSA-6 | Longest Consecutive Sequence O(n) HashSet | ≤ 15 min, no sorting |
| DSA-7 | Subarray Sum = K, `{0:1}` seed explained | ≤ 10 min |
| Bits | 4 bit problems (136 / 191 / 338 / 268) | each ≤ 10 min, "why XOR" articulated |
| Java-1 | Derive `equals`/`hashCode` contract | State + code + 1 JPA pitfall |
| Java-2 | Draw HashMap internals from memory | Buckets, list, tree threshold, load factor, resize split |
| Java-3 | Autoboxing 128 cache boundary | Correct + code example |
| Java-4 | ConcurrentHashMap CAS vs synchronized + null reason | Explained without "segment" |
| Java-5 | Fail-fast `modCount` (incl. `set()` edge case) | Code snippet; `set()` doesn't throw |
| Java-6 | Comparable vs Comparator + BigDecimal gotcha | Both + the consistency bite |
| SD | Smart360 5-service diagram from memory | Sync/async arrows + failure modes |
| Resume | All 3 projects have 3+ numbers each | Count them now |

**Score 16/16 → ready for Week 2.**
**Score 12–15 → identify gaps, spend 30 extra min Monday on the failed gates.**
**Score < 12 → repeat the weakest day before moving on — do not skip to Week 2.**

---

*Nav: ← [Fri Jul 03](05-fri-jul-03.md) · [Week 1](README.md) · [Week 2](../week-02/) →*
