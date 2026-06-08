# Week 1 — Foundations (Jun 9–14, 2026 · 6-day kickoff)

> Lock in the two skills every product-company interview tests first: pattern-recognition on arrays/pointers/windows, and fluent Java internals tied to your own shipped code.
>
> **Kickoff week:** 6 days (Tue Jun 9 – Sun Jun 14) so that Week 2 onward lands on clean Mon–Sun calendar weeks. Friday is a fuller consolidation day that absorbs the original review + behavioral work.

---

## 🎯 Week Goal

By Friday you can solve any Easy and most Medium array/two-pointer/sliding-window problems in ≤20 min with a clean Big-O analysis. By Sunday your resume has hard numbers on every bullet and you can narrate all three projects (Smart360, Deep Fathom, WebX) as architecture stories with explicit trade-off reasoning.

---

## ✅ By Sunday you can...

- Solve Trapping Rain Water and 3Sum from scratch, explain time/space complexity, and articulate the exact *why* behind each pointer move.
- Derive `equals`/`hashCode` contract from first principles and explain what breaks in a `HashSet` when you violate it — with a live code example.
- Explain HashMap's internal structure (array of buckets, linked list → red-black tree at threshold 8, load factor 0.75, rehashing) without looking anything up.
- Walk through Smart360's query optimization (60s → 2–3s) naming exactly which Hibernate anti-patterns you fixed and which indexes you added.
- Recite 3 quantified bullets per project that will survive recruiter screening *and* deep technical drill-down.
- Explain why `autoboxing` `Integer` in a `HashMap` key is safe but `==` comparison is not, and why this matters in production cache code.

---

## 📅 Daily Checklist

---

### Tuesday Jun 9 — Arrays & Two Pointers: Classic Patterns

**DSA (Block A, ~50 min):**

Pattern: **Two Pointers — opposite ends converging**. The invariant is always "what does it mean for this pair/triplet to be valid?" Once you see it, the pointer-move direction becomes mechanical.

Problems (in order):
1. **Two Sum** (LC 1) — warm-up; know both the brute-force O(n²) and HashMap O(n) solutions. Interviewers ask follow-ups: "what if the array is sorted?" → two pointers, no extra space.
2. **Valid Palindrome** (LC 125) — classic two-pointer convergence; handle `Character.isAlphanumeric` edge cases.
3. **Container With Most Water** (LC 11) — key insight: always move the *shorter* pointer inward. If you move the taller one you can only lose area. Prove this to yourself before coding.
4. **Trapping Rain Water** (LC 42) — hardest of the day. Two approaches: (a) prefix/suffix max arrays O(n) space, then (b) two-pointer O(1) space. The O(1) version: `water += min(maxL, maxR) - height[i]`. If `maxL ≤ maxR`, advance left pointer; water at left is constrained by `maxL`. Drill the *why* until it's intuitive.

Time check: if you finish all 4 in <40 min, attempt **LC 167 Two Sum II** (sorted array variant) and **LC 15 3Sum** (save full solution for Wednesday).

**Core Topic (Block B, ~50 min) — Java `equals`/`hashCode` contract:**

- The contract: if `a.equals(b)` is true, then `a.hashCode() == b.hashCode()` MUST be true. The converse is NOT required (hash collisions are fine).
- Violation consequence: store an object in a `HashSet`, then mutate a field used in `hashCode` → the set can no longer find the object even though it's there. This is a real Smart360 bug class — mutable JPA entities used as Map keys.
- Implement `equals`/`hashCode` by hand for a `User(Long id, String email)` class. Use `Objects.equals` and `Objects.hash`. Then show what `Lombok @EqualsAndHashCode` generates and when `@EqualsAndHashCode(onlyExplicitlyIncluded = true)` is necessary for JPA entities (use only the database PK).
- JPA-specific rule: never include mutable collection fields or lazy-loaded associations in `hashCode` — triggers unintended DB queries and breaks hibernate session caching.
- Write a 10-line test that demonstrates the broken `HashSet` lookup when contract is violated. You will be able to live-code this in an interview.

**Self-check:**
1. Given a sorted array, why does two-pointer beat HashMap for Two Sum II? (Answer: O(1) space, O(n) time, no hash overhead — and sorted order is the invariant that guarantees correctness.)
2. What happens in a `HashSet<User>` if you change a user's `id` after inserting it? (Answer: the set now looks in the wrong bucket; `contains` returns false; the element is "ghost-present".)

---

### Wednesday Jun 10 — Sliding Window + HashMap Internals

**DSA (Block A, ~50 min):**

Pattern: **Sliding Window — variable size**. The window expands by moving `right`, shrinks by moving `left` when an invariant is violated. Always ask: "what invariant does my window maintain?"

Problems (in order):
1. **Longest Substring Without Repeating Characters** (LC 3) — use `HashMap<Character, Integer>` storing last-seen index. When duplicate found, jump `left` to `max(left, lastSeen[c] + 1)`. Don't just increment left — you'll re-process characters.
2. **Maximum Average Subarray I** (LC 643) — fixed-size window warm-up. Practice the pattern before variable size.
3. **Minimum Size Subarray Sum** (LC 209) — variable window, shrink when `sum >= target`. Track minimum length. Time O(n) despite two loops because each element enters and exits the window exactly once.
4. **Longest Repeating Character Replacement** (LC 424) — harder. Window invariant: `(windowSize - maxFreq) ≤ k`. You only need to track `maxFreq` — never decrease it (even if stale) because you only care about a *longer* valid window.

**Core Topic (Block B, ~50 min) — HashMap Internals:**

- Internal structure: `Node<K,V>[] table` (array of buckets). Bucket index = `(n-1) & hash(key)`. Default initial capacity 16, load factor 0.75 → resize at 12 entries.
- Collision handling: linked list per bucket. When a bucket's list length hits **8** AND total capacity ≥ 64 → list converts to **red-black tree** (TreeNode). Why? O(n) → O(log n) lookup per bucket under heavy collision.
- `hash(key)` in Java: `key.hashCode() ^ (h >>> 16)`. The XOR with the high bits reduces collisions when capacity is small (low-order bits dominate the bucket index).
- Rehashing: doubles capacity, recomputes all bucket indices. Expensive — O(n). If you know size upfront, use `new HashMap<>(expectedSize / 0.75 + 1)` to avoid rehashing.
- Thread safety: `HashMap` is not thread-safe. `ConcurrentHashMap` uses segment-level locking (Java 7) → CAS + synchronized per-bucket (Java 8+). Never use `Collections.synchronizedMap(new HashMap<>())` in high-throughput code — it's a global lock.
- Tie to Smart360: your Redis caching layer uses string keys. Knowing how `String.hashCode()` works (polynomial rolling hash, cached after first call) explains why `String` is the ideal HashMap key.

**Self-check:**
1. Two HashMap lookups return different results for the same key object — what's the most likely cause? (Answer: `hashCode` is based on mutable fields that changed between inserts; object is in a different bucket now.)
2. At what point does Java convert a HashMap bucket's linked list to a tree, and what does that improve? (Answer: threshold 8, with table capacity ≥ 64; improves worst-case bucket lookup from O(n) to O(log n).)

---

### Thursday Jun 11 — Sliding Window Advanced + Generics & Autoboxing

**DSA (Block A, ~50 min):**

Pattern: **Two Pointers — multi-pointer (3Sum family)**. The trick is reduce-to-two-pointer: sort, fix one element, run two-pointer on the rest. Skip duplicates explicitly.

Problems (in order):
1. **Best Time to Buy and Sell Stock** (LC 121) — sliding window disguised as DP. Track `minPrice` so far, compute `price - minPrice` each day. O(n), O(1). Classic "track a running minimum" pattern.
2. **Maximum Subarray (Kadane's Algorithm)** (LC 53) — `currentSum = max(num, currentSum + num)`. If the running sum goes negative, restart. This is also the foundation for DP — interviewers may ask you to reconstruct the actual subarray (track `start`, `tempStart`, `end` indices).
3. **3Sum** (LC 15) — sort first O(n log n). Fix `nums[i]`, run two pointers on `[i+1, n-1]`. Skip duplicates at both the outer loop and inner pointers. Common bug: forgetting to skip duplicates after finding a valid triplet.
4. **Subarray Sum Equals K** (LC 560) — prefix sum + HashMap. `prefixSum[j] - prefixSum[i] = k` → look for `prefixSum - k` in the map. Important: initialize `map.put(0, 1)` before the loop.

**Core Topic (Block B, ~50 min) — Generics & Autoboxing Pitfalls:**

- **Type erasure**: generics exist at compile time only. At runtime, `List<String>` and `List<Integer>` are both `List`. This means: no `instanceof` with generic types, no `new T[]`, no overloading `method(List<String>)` vs `method(List<Integer>)` — they have the same erasure.
- **Bounded wildcards**: `? extends T` (producer — read-only, covariant), `? super T` (consumer — write-only, contravariant). The PECS rule: **P**roducer **E**xtends, **C**onsumer **S**uper. Example: `Collections.copy(List<? super T> dest, List<? extends T> src)`.
- **Autoboxing pitfalls** (these cause real production bugs):
  - `Integer a = 127; Integer b = 127; a == b` → `true` (cached). `Integer a = 128; Integer b = 128; a == b` → `false` (different objects). Always use `.equals()`.
  - `Long sum = 0L; for (int i : list) sum += i;` → each `i` is autoboxed to `Integer`, then unboxed, then the result is autoboxed to `Long` — 2× object creation per iteration. In tight loops this is measurable GC pressure.
  - Null unboxing: `Integer x = null; int y = x;` → NullPointerException at unboxing. Insidious in JPA entities where nullable DB columns map to `Integer` fields.
  - HashMap with autoboxed keys: `map.get(new Long(id))` misses if keys were inserted as `long` (autoboxed to `Long`). In Smart360's caching layer, always use explicit `Long` boxing and `.equals()` for cache key lookups.
- **Practical rule**: use primitive types in performance-critical paths. Use wrapper types only when null is semantically meaningful (nullable DB columns, Optional internals).

**Self-check:**
1. Why does `List<Dog>` not satisfy `List<Animal>` even though `Dog extends Animal`? (Answer: type erasure + covariance would break type safety — you could insert a `Cat` into a `List<Dog>` reference typed as `List<Animal>`; Java uses wildcards instead.)
2. You have a nullable `Integer` column from the DB. Write the safe null-check before unboxing. (Answer: `if (entity.getCount() != null) { int c = entity.getCount(); }` — or use `Objects.requireNonNullElse`.)

---

### Friday Jun 12 — Mixed Pattern Review + OOP Principles + Consolidation (fuller day)

**DSA (Block A, ~50 min):**

Pattern: **Recognition drill — given a problem, identify the pattern before coding**. Today's goal is speed and accuracy of pattern identification, not just solving.

Problems (in order):
1. **Find All Anagrams in a String** (LC 438) — sliding window + frequency array comparison. Fixed window size = `p.length()`. Compare char counts, not sorted strings (O(26) compare, not O(k log k)).
2. **Longest Subarray of 1s After Deleting One Element** (LC 1493) — sliding window where invariant is "at most one 0 in window." When you remove a `1`, shrink from left until at most one `0` remains.
3. **Product of Array Except Self** (LC 238) — no division allowed. Prefix product array, then suffix product in a second pass. O(n) time, O(1) extra space if output array counts. This is a famous Meta/Google question — know it cold.
4. **Rotate Array** (LC 189) — three-reversal trick. Reverse all, reverse first k, reverse last n-k. O(n) time, O(1) space. Know why this works by visualizing indices.

**Core Topic (Block B, ~50 min) — OOP: SOLID + Inheritance vs Composition:**

- **Single Responsibility**: your `AuthorizationService` in Smart360 had one job — token validation + role resolution. When it started handling user management events, it violated SRP. That's why you extracted it (the strangler-fig migration story).
- **Open/Closed**: your LLM provider router in WebX is a good example — adding a new provider means implementing a `LLMProvider` interface, not modifying the router. Explain this concretely.
- **Liskov Substitution**: if `PremiumUser extends User`, then any method that works correctly with `User` must work with `PremiumUser`. Common violation: overriding a method to throw `UnsupportedOperationException` (e.g., `ReadOnlyList.add()`).
- **Interface Segregation**: in your microservices, a `UserService` interface with 20 methods means every consumer depends on methods it doesn't use. Split into `UserReader`, `UserWriter`, `UserAuthenticator`.
- **Dependency Inversion**: Spring's entire DI container is the practical implementation. Your services depend on interfaces (`UserRepository`), not `UserRepositoryImpl`. Spring injects the concrete class.
- **Inheritance vs Composition**: prefer composition. Inheritance creates tight coupling and fragile base class problems — if the parent changes, all children are affected. Use inheritance only for true IS-A relationships. Example: `PostgreSQLUserRepository implements UserRepository` (composition via interface). NOT `PostgreSQLUserRepository extends BaseRepository extends AbstractRepository`.
- **Abstract class vs Interface**: interface = contract (can implement many); abstract class = partial implementation (single inheritance). Java 8+ default methods blur the line but the semantic distinction remains.

**Self-check:**
1. In Product of Array Except Self, why is the output array not counted as extra space? (Answer: the problem says output array doesn't count; the O(1) claim is about *auxiliary* space beyond input and output.)
2. Name one real violation of LSP in Java's standard library. (Answer: `java.util.Stack extends Vector` — Stack IS NOT a general-purpose Vector, yet it inherits all Vector methods; `Stack.add(0, element)` breaks stack semantics.)

**Stretch DSA — timed mock (this is a fuller consolidation day; if short on time, roll these two and the behavioral block below into Saturday), 25 min each, no hints:**

1. **Minimum Window Substring** (LC 76) — the hardest sliding window. Two pointers + two frequency maps. Expand right until all chars covered, then shrink left while still valid, record minimum window. Practice the "have" vs "need" counter trick: only increment `have` when a char's frequency exactly matches `need[c]`.
2. **Maximum Product Subarray** (LC 152) — extension of Kadane. Track both `maxSoFar` and `minSoFar` because a negative × negative = positive. At each step: `newMax = max(num, maxSoFar*num, minSoFar*num)`.

After each problem: write out the pattern name, time complexity, space complexity, and one follow-up question the interviewer might ask. This is interview muscle memory.

**Block C (~45 min) — Behavioral Stories + STAR Refinement:**

For each story below, recite it aloud (not in your head) against a timer (2 min max per story). Tighten the numbers.

- **Performance story (Smart360)**: Situation → 60s query latency causing timeouts (SLA breach). Task → optimize without schema changes. Action → profiled with `spring.jpa.show-sql=true` + Hibernate statistics, identified N+1 on `User → Roles → Permissions` (3 queries per user × 200 users = 600 queries per request), rewrote with `@EntityGraph(attributePaths = {"roles", "roles.permissions"})` reducing to 1 query, added composite index on `(tenant_id, user_id)`, added Redis cache for S3 pre-signed URLs with TTL=55min (URL expiry=60min). Result → 2–3s latency, 80% S3 reduction.
- **Architecture decision story (Deep Fathom)**: Situation → multi-tenant SaaS, 50+ tables needing isolation, 3-person team. Task → implement tenant isolation without application-layer risk. Action → PostgreSQL RLS policies (`tenant_id = current_setting('app.tenant_id')`) — security enforced at DB layer even if app has a bug; Bicep IaC for reproducible Azure environments. Result → FedRAMP-compatible isolation, reproduced in GovCloud from same Bicep modules.
- **Leadership/initiative story (CI/CD 57% cut)**: Situation → 23-min pipeline blocking developer velocity. Task → reduce without breaking existing tests. Action → analyzed GitLab pipeline DAG, identified sequential bottleneck in Docker build + test stages, enabled BuildKit parallel builds with `--cache-from` registry image, configured Turbo monorepo caching for unchanged packages, restructured GitLab `needs:` to parallelize test matrix. Result → 10 min pipeline, ~13 min saved per run × ~15 runs/day = ~3 hours/day developer time reclaimed.

**Self-check:**
1. In Minimum Window Substring, what is the time complexity and why? (Answer: O(n + m) where n = len(s), m = len(t). Each character enters and exits the window at most once.)
2. If an interviewer says "tell me about a failure," use the CI/CD story in reverse — the initial failure was accepting slow pipelines for months. What did you learn? What would you do differently?

---

### Saturday Jun 13 — Resume Rewrite + Project Deep-Dive Docs (4 hr)

**Block A (90 min) — Resume Rewrite: Impact-First Format**

Every bullet must answer: "So what? How much? How do I know?" Remove all bullets that don't have a number or a named technology decision.

**Smart360 rewrites (before → after):**
- Before: "Developed a microservices architecture with 5 services"
- After: "Designed and shipped a 5-microservice platform (API Gateway, Auth, Data, Visualization, Notification) on Spring Cloud + Eureka; reduced sign-in latency 60% by migrating session validation to stateless JWT, eliminating per-request DB roundtrips"
- Before: "Improved query performance"
- After: "Cut data retrieval latency 96% (60s → 2–3s) by eliminating N+1 Hibernate queries via EntityGraph, adding composite indexes, and caching S3 pre-signed URLs in Redis (80% S3 call reduction)"
- Before: "Implemented RBAC"
- After: "Built fine-grained RBAC at repository/table/permission granularity using Spring Security + JWT claims; enforced across 5 services via shared auth filter in Spring Cloud Gateway"

**Deep Fathom rewrites:**
- "Deployed on Azure Container Apps + AKS with Bicep IaC across 30+ resources; provisioned identical staging and FedRAMP GovCloud environments from a single Bicep codebase"
- "Implemented PostgreSQL Row-Level Security on 50+ tables; tenant isolation enforced at DB layer, eliminating app-layer bypass risk in a multi-tenant SaaS"
- "Reduced CI/CD pipeline runtime 57% (23 min → 10 min) via BuildKit parallel builds, registry layer caching, and GitLab DAG restructuring"

**WebX rewrites:**
- "Designed async job architecture for 2–20 min LLM operations: 202 Accepted + jobId pattern with SSE push on completion; supports 5+ LLM providers via unified proxy interface for cost/capability routing"
- "Integrated LLM-powered features with graceful degradation — circuit breaker pattern ensures provider outages don't degrade core product"

**Block B (90 min) — Project Deep-Dive Doc: One Page Per Project**

Write these in a notebook or separate doc — they are your verbal interview cheat sheet.

**Smart360 Architecture Story:**
```
What: Multi-tenant 360-degree performance review platform
Scale: [insert user count or tenant count if known]
Architecture: API Gateway (Spring Cloud Gateway) → Auth Service → Data Service → Visualization Service → Notification Service
Key decisions:
  - Spring Cloud Gateway for JWT validation at edge (not per-service) — single policy update point
  - Eureka for service discovery over K8s native discovery — team familiarity at the time (trade-off: extra infra component)
  - Redis for two purposes: (1) token blacklist for JWT revocation, (2) S3 URL cache
  - EventDriven notification via messaging — decoupled from Auth so notification failures don't block auth flows
Hardest problem: N+1 query on role-permission graph per authenticated request; 600 queries per single API call
Fix: EntityGraph eager-loads roles.permissions in 1 JOIN query
Numbers: 60s → 2-3s, 80% S3 reduction, 60% sign-in latency cut
What I'd do differently: Start with K8s service discovery instead of Eureka; use Postgres advisory locks instead of Redis for simpler token blacklist
```

**Deep Fathom Architecture Story:**
```
What: Multi-tenant B2B SaaS on Azure (industry: [your domain])
Scale: 50+ DB tables, multiple tenants, FedRAMP GovCloud requirement
Architecture: Azure Container Apps + AKS, PostgreSQL Flexible Server, Redis, Azure Front Door, Key Vault
Key decisions:
  - PostgreSQL RLS over app-layer filtering: security guaranteed even with application bugs; trade-off is DB connection must set tenant context per request (use connection pool advisor)
  - Bicep IaC over Terraform: native Azure, no state file management, first-class VS Code support; trade-off is Azure-only (Terraform is multi-cloud)
  - Azure Container Apps for most services (scales to zero), AKS only for services needing custom K8s operators
Hardest problem: FedRAMP compliance required identical environments — Bicep modules parameterized for GovCloud regions solved this
Numbers: 57% CI/CD reduction, multi-tenant on 50+ tables
What I'd do differently: Introduce Debezium outbox pattern earlier for event publishing instead of polling
```

**WebX Architecture Story:**
```
What: LLM-integrated product (document analysis / [your actual use case])
Architecture: Spring Boot backend, async job queue, 5+ LLM provider proxy, Vue.js / React frontend
Key decisions:
  - 202 Accepted + polling/SSE over synchronous: LLM calls 2-20 min, HTTP timeouts at ~30s for most load balancers
  - Unified provider interface: abstract LLM calls behind a common API; route by cost (GPT-4o expensive, Haiku cheap for classification tasks) or capability
  - Async job persistence: job state in PostgreSQL, not in-memory, so pod restarts don't lose jobs
Hardest problem: Streaming responses from LLM providers have different formats (OpenAI SSE vs Anthropic streaming) — abstracted into a common reactive stream
Numbers: [your actual metrics — response time, job throughput, provider cost reduction]
What I'd do differently: Use a proper job queue (AWS SQS or RabbitMQ) instead of DB polling for better backpressure
```

**Block C (60 min) — Mock Behavioral Questions (speak aloud):**
- "Walk me through your most complex technical project." → Use Deep Fathom RLS story.
- "Tell me about a time you disagreed with a technical decision." → Prepare one — find a real moment.
- "Why are you leaving your current company?" → Frame as growth/scope/compensation, never negative.
- "Where do you see yourself in 3 years?" → Technical leadership track, driving architecture decisions, mentoring.

---

### Sunday Jun 14 — Deep Java Internals + Full Mock Interview (4 hr)

**Block A (90 min) — Java Memory Model + Concurrency Basics:**

These questions trip up 2–3 YOE candidates who have used Spring but haven't thought about what happens at the JVM level.

- **JVM Memory areas**: Heap (objects, GC managed), Stack (method frames, local variables, per-thread), Metaspace (class metadata, replaced PermGen in Java 8), Code Cache (JIT compiled code), Direct Memory (NIO ByteBuffer).
- **Garbage Collection**: G1GC is default from Java 9+. Key concept: generational hypothesis (most objects die young). Young gen (Eden + 2 Survivor spaces) → full objects promoted to Old gen → G1 collects in "regions" rather than full heap scans. In production Spring Boot apps, tune `-XX:MaxGCPauseMillis=200` (G1 target). Know what a GC pause is and why it matters for latency-sensitive services.
- **String Pool**: `String s = "hello"` → interned in the string pool (Metaspace). `String s = new String("hello")` → creates a new heap object every time. `s.intern()` returns the pooled reference. In Smart360, using `String.valueOf(id)` as a cache key is fine; `new String(...)` in a tight loop is wasteful.
- **`volatile` keyword**: guarantees visibility (changes visible to all threads immediately) but NOT atomicity. `volatile int count; count++` is still a race condition (read-modify-write is 3 ops). Use `AtomicInteger` for thread-safe counters.
- **`synchronized` vs `ReentrantLock`**: `synchronized` is JVM-level, auto-releases on exception, no try-lock capability. `ReentrantLock` is explicit, supports `tryLock(timeout)`, `lockInterruptibly()`, and fair/unfair modes. In high-contention scenarios, `ReentrantLock` with fair mode prevents starvation.
- **ThreadLocal**: per-thread storage, no synchronization needed. Spring's `SecurityContextHolder` uses `ThreadLocal` to store the current authentication — this is why the security context is available anywhere in the call stack without passing it explicitly. Warning: in thread pools, `ThreadLocal` values leak across requests unless `SecurityContextHolder.clearContext()` is called.
- **`CompletableFuture`**: used in WebX for orchestrating async LLM calls. Know `thenApply` (sync transform), `thenCompose` (async chain, avoids nested futures), `allOf` (wait for all), `exceptionally` (error handler). Know which methods use the ForkJoinPool by default and when to provide your own executor.

**Block B (60 min) — Additional LeetCode: Week 1 Gaps:**

Identify your 2 weakest problems from Mon–Fri. Redo them without looking at your solutions. If you pass both in <20 min each, attempt:
- **Sliding Window Maximum** (LC 239) — deque-based, hard. Window keeps indices in decreasing order of value. Front of deque is always the max.
- **Minimum Number of Arrows to Burst Balloons** (LC 452) — greedy + interval thinking. Sort by end point, greedily shoot at the end of the first balloon. Introduction to the interval pattern for next week.

**Block C (90 min) — Full Mock Interview (conduct this seriously):**

Use a timer. Treat it as real. No looking at notes during the problem.

Round 1 (45 min):
- 5 min: "Tell me about yourself" — practice your 90-second pitch ending with "which is why I'm excited about this opportunity."
- 20 min: Interviewer asks "Two Sum" → you solve it, then they follow up: "Now the array is sorted" → "Now there can be duplicates and you want all pairs" → "Now k numbers instead of 2."
- 20 min: "Explain how you optimized the 60-second query" → dive deep, expect follow-ups: "What's an EntityGraph?" "Why not just use `@Fetch(FetchType.EAGER)`?" "How did you verify the improvement?"

Round 2 (45 min):
- 25 min: Design question — "Design the async job system in WebX." Draw it: Client → POST /jobs → 202 + jobId → Job persisted in DB → Worker polls DB (or queue) → LLM provider call → Update job status → Client polls GET /jobs/{id} or SSE push. Discuss trade-offs: DB polling vs message queue (RabbitMQ/SQS), at-least-once delivery, idempotency.
- 20 min: Behavioral — "Tell me about your biggest technical mistake." Prepare one honest answer where you own it, explain the fix, and name the learning.

**Self-check (pass/fail for the week):**
1. Can you solve Container With Most Water and explain the pointer-move invariant in 90 seconds? (Target: yes)
2. Can you draw HashMap's internal structure (buckets, linked list, tree) from memory? (Target: yes)
3. Do all three project stories have at least 3 numbers each? (Target: yes)

---

## 🧠 Concepts to Master This Week (with depth — the "why", gotchas, follow-up angles)

### Arrays, Two Pointers, Sliding Window

**The mental model**: arrays are the simplest data structure but the richest source of patterns. Almost every array problem is one of: scan once (prefix/suffix), two ends converging, or a variable window.

**Two-pointer invariant discipline**: Before writing a single line of code, state the invariant. For Container With Most Water: "the water volume is `min(h[l], h[r]) * (r - l)`; we move the shorter pointer inward because keeping the taller pointer can only decrease width while the shorter pointer is already the bottleneck." Write this in an interview — it signals senior-level thinking.

**Sliding window template** (memorize this):
```java
int left = 0, result = 0;
Map<Character, Integer> window = new HashMap<>();
for (int right = 0; right < s.length(); right++) {
    char c = s.charAt(right);
    window.merge(c, 1, Integer::sum); // add right element
    while (/* invariant violated */) {
        char d = s.charAt(left);
        window.merge(d, -1, Integer::sum);
        if (window.get(d) == 0) window.remove(d);
        left++;
    }
    result = Math.max(result, right - left + 1); // or collect result
}
```

**Kadane's gotcha**: when the problem asks for a non-empty subarray, `currentSum = max(num, currentSum + num)` handles it correctly — choosing `num` alone restarts the subarray. If an empty subarray is allowed, initialize differently.

**Prefix sum pattern**: for subarray sum problems, `prefixSum[i] = sum of nums[0..i-1]`. Subarray sum from `i` to `j` = `prefixSum[j+1] - prefixSum[i]`. The HashMap trick (store prefix sums seen so far) enables O(n) solutions for "number of subarrays with sum = k."

---

### Java `equals`/`hashCode` Contract

**Why it matters in your codebase**: JPA entities are often stored in `Set` collections (e.g., `Set<Role>` on a `User`). If you use the default `Object.equals` (identity), two `Role` objects fetched in different Hibernate sessions representing the same DB row will be treated as different — breaking `contains` checks and causing duplicate data in sets.

**JPA-specific best practice**: implement `equals`/`hashCode` on the business key (natural identifier like `email`, `code`) or on the database PK only — and handle the null PK case for transient entities (entities not yet persisted have `id == null`; two new entities should not be considered equal even if all fields match, because they might get different IDs):
```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof User)) return false;
    User user = (User) o;
    return id != null && id.equals(user.id);
}
@Override
public int hashCode() { return getClass().hashCode(); } // constant — safe for JPA
```
The constant `hashCode` means all entities land in the same bucket — O(n) lookup in sets — but it's safe. The alternative (using `id`) breaks if you insert a transient entity before persisting it.

**Follow-up interviewers love**: "What happens in a `HashMap` if two keys have the same hashCode but `equals` returns false?" → They go into the same bucket as a linked list node (collision). The map is still correct, just slower.

---

### HashMap Internals

**The treeification trigger detail**: treeification only occurs when BOTH `bucket.size >= 8` AND `table.length >= 64`. If `table.length < 64`, Java resizes the entire map instead. This prevents premature treeification when the map is small.

**Why load factor 0.75?** It's a space-time trade-off backed by Poisson distribution math. At 0.75 load, the expected number of elements per bucket follows a Poisson(0.5) distribution — the probability of a bucket having 8+ elements (triggering tree) is less than 1 in 10 million. Below 0.75, you waste space; above, collision probability rises sharply.

**Curveball**: "Can two different `String` objects with the same content be the same HashMap key?" → Yes. `HashMap` uses `equals` for key comparison (after hash bucket lookup), and `String.equals` compares content. So `new String("hello")` and `"hello"` are equal as keys. But `==` would return false — never use `==` for String keys.

---

### Generics, Type Erasure, Autoboxing

**Type erasure practical consequence in Spring**: `@Autowired List<UserRepository>` — Spring can inject all beans of type `UserRepository` because the generic parameter is erased at runtime, but Spring's bean factory knows the concrete types at registration time. This is how `@Primary` and `@Qualifier` interact with generic collections.

**Autoboxing and Streams**: `IntStream` exists specifically to avoid boxing. `stream().mapToInt(...).sum()` uses `int` primitives throughout. `stream().map(...).reduce(0, Integer::sum)` boxes every element. For numeric-heavy code in your data visualization service, `IntStream`/`LongStream` is measurably faster.

**The 128 boundary gotcha is interview gold**: ask any developer and half will get it wrong. Memorize: Java caches `Integer` values from -128 to 127. The specific number 128 isn't random — it's configurable via `-XX:AutoBoxCacheMax=<size>` at JVM startup, but defaults to 127.

---

## 🎤 Sample Interview Questions (incl. curveballs) — 8–15 Qs with brief answer pointers

**DSA:**

1. **"Solve Two Sum, then what if the input were a data stream with no fixed size?"** → Two-pointer doesn't work (unsorted stream). Use a `HashSet` — for each number `x`, check if `target - x` is in the set before adding `x`. O(n) time, O(n) space.

2. **"You solved Maximum Subarray with Kadane's. Now find the maximum sum subarray of length exactly k."** → Sliding window (fixed size), not Kadane. `sum of window = prefix[i+k] - prefix[i]`. This tests whether you know WHICH pattern to apply.

3. **[Curveball] "In your sliding window solution, you used a HashMap. What's its worst-case time complexity and when does it degrade?"** → HashMap operations are O(1) average but O(n) worst case if all keys hash to the same bucket. For `char` keys (26 or 128 possible values), use a `int[128]` array instead — truly O(1), no collision.

**Java/Spring:**

4. **"You use `@Transactional` on a service method. Another method in the same class calls it. What happens?"** → The transaction is bypassed. The call is a direct Java call, not through the proxy. Fix: inject `self` via `@Autowired` or use `AopContext.currentProxy()`. Tie to real bug risk in your Smart360 auth service.

5. **"Your Spring Boot app has two beans of type `UserRepository`. How does Spring decide which to inject?"** → Ambiguity error at startup. Resolve with `@Primary` (global preference) or `@Qualifier("beanName")` at injection point. For a list, `@Autowired List<UserRepository>` injects all of them.

6. **"What's the difference between `BeanFactory` and `ApplicationContext`?"** → `ApplicationContext` extends `BeanFactory` and adds: event publishing (`ApplicationEventPublisher`), internationalization (`MessageSource`), AOP support, resource loading. `BeanFactory` is lazy (creates beans on first request); `ApplicationContext` is eager (creates singletons on startup). In practice, always use `ApplicationContext`.

7. **[Curveball] "Your HashMap started with capacity 16. You insert 10 elements. What's the current load factor and will it resize?"** → Load factor = 10/16 = 0.625 < 0.75. No resize yet. Resize triggers at 0.75 × 16 = 12 elements. At the 13th insert, it doubles to capacity 32 and rehashes.

8. **[Curveball] "You used Redis to cache S3 pre-signed URLs in Smart360. What happens if Redis goes down? Walk me through the failure mode."** → Cache miss fallback: every request hits S3 for a new URL. This is acceptable (graceful degradation) IF the code has a proper try-catch around the Redis call and falls through to S3. If the code throws instead, all requests fail. Answer: implement cache-aside pattern with null-check + catch Redis exceptions, never let cache failure propagate to the user.

**Architecture/Design:**

9. **"Why did you use PostgreSQL RLS instead of application-layer tenant filtering in Deep Fathom?"** → Application-layer filtering can be bypassed by a code bug (e.g., a developer forgets to add the `WHERE tenant_id = ?` clause to a new query). RLS is enforced by the database regardless — even raw SQL through pgAdmin respects the policy. The trade-off: you must set the tenant context on every DB connection (use a `DataSourceConnectionInterceptor`), and you can't see cross-tenant data without disabling RLS via a superuser connection.

10. **"How would you test the RLS policies in Deep Fathom?"** → Write integration tests that: (1) insert data for tenant A and tenant B, (2) set `app.tenant_id = A` on the connection, (3) assert that only tenant A's data is visible. Use `@DataJpaTest` with a real PostgreSQL (via Testcontainers) since H2 doesn't support RLS.

11. **"Your LLM job takes 20 minutes. A user closes their browser. What happens to the job?"** → The job is persisted in PostgreSQL, not tied to the HTTP request. The worker continues. When the user reopens the browser, they can poll `GET /jobs/{id}` to get the result. The SSE connection would have dropped, but the job result is durable. This is why you never run long jobs synchronously in the web request thread.

12. **[Curveball] "If you had to add rate limiting per user per LLM provider in WebX, how would you do it?"** → Use Spring Cloud Gateway's `RequestRateLimiter` filter backed by Redis (token bucket algorithm). Key = `user_id:provider_id`. Redis `INCR` + `EXPIRE` for a simple counter, or Resilience4j `RateLimiter` at the service level. The gateway approach is better because it stops requests before they reach your service.

13. **"You mentioned you cut CI/CD from 23 to 10 minutes. What would you do to get it to 5 minutes?"** → Aggressive caching (Docker layer, test results), test impact analysis (run only tests for changed modules — Turbo/Nx supports this), build artifact caching between branches, parallelizing across more runners, and moving slow integration tests to a nightly run instead of every PR.

14. **[Behavioral curveball] "Tell me about a technical decision you regret."** → Prepared answer: choosing Eureka for service discovery in Smart360 early on. The team already had Kubernetes in mind. When we moved to K8s, Eureka became redundant infrastructure to maintain. I'd start with K8s-native service discovery (Spring Cloud Kubernetes) from day one. The learning: don't add infrastructure components that duplicate what your platform already provides.

15. **"Where do microservices fail most often in practice, and have you seen this firsthand?"** → Distributed transactions and data consistency. Every time you need an operation to span two services atomically, you pay the complexity tax (sagas, outbox pattern, eventual consistency). In Smart360, the user creation flow spans Auth and Notification — a failure in Notification after Auth commits means the user exists but never got their welcome email. We addressed it with the outbox pattern, but the debugging took days because the failure was silent.

---

## 🌟 Extraordinary-Candidate Edge (2–4 specific differentiators)

**1. You narrate trade-offs, not just decisions.**
Most candidates describe *what* they built. You describe *why* you chose it over the alternatives and what you'd do differently today. In the RLS story: "I chose DB-layer enforcement over app-layer filtering. Here's why, here's the operational cost (setting tenant context per connection), and here's what I'd architect differently for a global multi-region deployment." This is senior-engineer vocabulary at 2.5 YOE.

**2. You connect algorithms to production code.**
When you solve a sliding window problem, you mention: "This exact pattern appears in our rate limiter — we maintain a window of requests per second per user." When you explain HashMap internals, you connect it to your Redis key design. Interviewers at product companies are evaluating whether you can bridge CS fundamentals to engineering judgment. Almost no candidate does this.

**3. You understand failure modes, not just happy paths.**
Redis goes down → your S3 cache miss degrades gracefully. Kafka consumer falls behind → your LLM job queue backs up but doesn't drop jobs (exactly-once semantics). The circuit breaker trips → your LLM router falls back to a cheaper provider. When you discuss your systems, proactively mention the failure mode and how you handle it. This signals production experience, not just feature development.

**4. You have prepared numbers and they're specific.**
"96% latency reduction (60s → 2–3s)" is remembered. "I improved performance significantly" is forgotten in 30 seconds. Every project story ends with a number. The numbers you give this week in your resume rewrite will be recalled verbatim by the hiring manager in the debrief.

---

## 📊 End-of-Week Self-Assessment (pass/fail gates)

Complete these checks on Sunday evening. Honest pass/fail only — no partial credit.

| Gate | Check | Pass Criteria |
|------|-------|--------------|
| DSA-1 | Solve Container With Most Water cold | ≤ 15 min, correct O(n) solution |
| DSA-2 | Solve Minimum Size Subarray Sum cold | ≤ 15 min, correct sliding window |
| DSA-3 | Solve 3Sum cold | ≤ 25 min, handles duplicates correctly |
| DSA-4 | Explain Kadane's algorithm in 60 sec | Can state invariant, handle all-negative case |
| Java-1 | Derive `equals`/`hashCode` contract | Can state it, code it, name 1 JPA pitfall |
| Java-2 | Draw HashMap internals from memory | Buckets, list, tree threshold, load factor |
| Java-3 | Explain autoboxing 128 cache boundary | Correct, can give code example |
| Java-4 | Name 3 `@Transactional` pitfalls | Self-invocation, checked exceptions, private methods |
| Resume | All 3 projects have 3+ numbers each | Count them right now |
| Stories | Smart360 story under 2 min aloud | Time yourself speaking it |
| Stories | Deep Fathom RLS story under 2 min | Time yourself |
| Stories | Can answer "why are you leaving?" | Prepared, positive, concise |

**Score: 12/12 = ready to proceed to Week 2**
**Score: 9–11 = identify gaps, spend 30 extra min Monday on the failed gates**
**Score: < 9 = repeat the weakest day before moving on — do not skip to Week 2**

---

*Week 1 of 13 — next: Week 2 (Mon Jun 15 – Sun Jun 21) covers Hashing/Strings + the Collections framework deep-dive + a system design primer.*
