# Week 1 — Foundations (Jun 29–Jul 4, 2026)

> The first full-time foundations week (≈36 hr; Mon–Sat, Sun rest). Lock in the two skills every product-company interview tests first: pattern-recognition on arrays / pointers / windows / hashing / bits, and fluent Java internals tied to your own shipped code (equals/hashCode, HashMap, generics, the Collections framework). This week is **heads-down study — it does NOT include applications**; the application engine starts later. Assumes the **Sun Jun 28 Week 0 basics refresh** is done and feeds directly into the Java-internals blocks here.

---

## 🎯 Week Goal

By Saturday you can solve any Easy and most Medium array / two-pointer / sliding-window / hashing / string / bit problems in ≤20 min with a clean Big-O analysis, derive the `equals`/`hashCode` contract and HashMap internals from memory, reason about the Collections framework from first principles (not just "what" but "why the data structure was designed that way"), and whiteboard Smart360's 5-microservice topology with sharp articulation of every architectural decision. Your resume rewrite and project deep-dive docs are started.

---

## ✅ By Saturday you can...

- Solve Trapping Rain Water, 3Sum, Minimum Window Substring, Group Anagrams, Top K Frequent, and Longest Consecutive Sequence from scratch with correct time/space complexity, articulating the exact *why* behind each pointer move or hash key.
- Derive the `equals`/`hashCode` contract from first principles and explain what breaks in a `HashSet` when you violate it — with a live code example.
- Explain HashMap's internal structure (buckets, linked list → red-black tree at threshold 8 with capacity ≥ 64, load factor 0.75, the `(h ^ (h >>> 16))` bit-spread, resize split) without looking anything up.
- Distinguish fail-fast vs fail-safe iterators, compare `ConcurrentHashMap` (CAS + per-bucket `synchronized`) vs `Hashtable` (full-map lock), and choose between `ArrayList`/`LinkedList`/`ArrayDeque` with the CPU-cache argument.
- Explain `Comparable` vs `Comparator`, the `BigDecimal`-vs-`equals` consistency gotcha, generics/type-erasure, and the autoboxing 128-cache boundary — and why each matters in production.
- Recite 3 quantified bullets per project (Smart360, Deep Fathom, WebX) that survive recruiter screening *and* deep technical drill-down, and whiteboard the Smart360 5-service diagram with failure boundaries.

---

## 📅 Daily Checklist (Mon–Sat; Sun rest)

Each weekday runs **3 blocks**: **Block A** — DSA (~2.5 h, primary pattern set), **Block B** — core Java/system topic (~2 h), **Block C** — second DSA set or deep-dive (~1.5 h).

---

### Monday Jun 29 — Two Pointers + equals/hashCode

📌 **Study today:** Two Pointers — opposite ends converging (LC 1 · 125 · 11 · 42 · 167) · Java `equals`/`hashCode` contract · Two-Sum variants + prefix-sum intro

**Block A (~2.5 h) — Two Pointers, opposite ends converging:**

The invariant is always "what does it mean for this pair/triplet to be valid?" Once you see it, the pointer-move direction becomes mechanical.

1. **Two Sum** (LC 1) — warm-up; know both brute-force O(n²) and HashMap O(n). Follow-up: "what if sorted?" → two pointers, no extra space.
2. **Valid Palindrome** (LC 125) — classic convergence; handle `Character.isLetterOrDigit` edge cases. Curveball: Unicode/emoji → `Character.codePointAt()` + `Character.isLetterOrDigit(codePoint)`. Follow-up LC 680 (one deletion allowed).
3. **Container With Most Water** (LC 11) — key insight: always move the *shorter* pointer inward; moving the taller one can only lose area. Prove it before coding.
4. **Trapping Rain Water** (LC 42) — hardest of the day. (a) prefix/suffix max arrays O(n) space, then (b) two-pointer O(1) space: `water += min(maxL, maxR) - height[i]`; if `maxL ≤ maxR` advance left. Drill the *why*.
5. **Two Sum II — sorted** (LC 167) — the sorted variant that proves two-pointer beats HashMap here (O(1) space).

**Block B (~2 h) — Java `equals`/`hashCode` contract:**

- The contract: if `a.equals(b)` then `a.hashCode() == b.hashCode()` MUST hold. The converse is NOT required (collisions are fine).
- Violation: store an object in a `HashSet`, mutate a field used in `hashCode` → the set can no longer find it though it's there. This is a real Smart360 bug class — mutable JPA entities as Map keys.
- Implement `equals`/`hashCode` by hand for `User(Long id, String email)` with `Objects.equals`/`Objects.hash`. Show what Lombok `@EqualsAndHashCode` generates and when `@EqualsAndHashCode(onlyExplicitlyIncluded = true)` is needed for JPA (use only the PK).
- JPA rule: never include mutable collections or lazy associations in `hashCode` — triggers unintended queries and breaks session caching. For entities, use a constant `hashCode` (`getClass().hashCode()`) + `id`-based `equals` with the transient (null-id) case handled.
- Write a 10-line test demonstrating the broken `HashSet` lookup when the contract is violated — you'll live-code this in an interview.

**Block C (~1.5 h) — Two-Sum variants + prefix-sum intro:**

- Re-solve Two Sum as a *stream* (no fixed size): for each `x`, check `target - x` in a `HashSet` before adding `x` — O(n)/O(n).
- **Prefix sum intro:** `prefixSum[i] = sum of nums[0..i-1]`; subarray `i..j = prefixSum[j+1] - prefixSum[i]`. This sets up Wednesday's Subarray-Sum-Equals-K. Code a "running prefix sum array" helper and a "sum of range [i,j]" query.

**Self-check:**
1. Given a sorted array, why does two-pointer beat HashMap for Two Sum II? (O(1) space, O(n) time, sorted order is the correctness invariant.)
2. What happens in a `HashSet<User>` if you change a user's `id` after inserting it? (Wrong bucket; `contains` returns false; element is "ghost-present".)

---

### Tuesday Jun 30 — Sliding Window + HashMap Internals

📌 **Study today:** Sliding Window — variable size (LC 3 · 209 · 424 · 643) · HashMap internals · subarray-sum + advanced sliding window

**Block A (~2.5 h) — Sliding Window, variable size:**

The window expands by moving `right`, shrinks by moving `left` when an invariant is violated. Always ask: "what invariant does my window maintain?"

1. **Maximum Average Subarray I** (LC 643) — fixed-size window warm-up; practice the template before variable size.
2. **Longest Substring Without Repeating Characters** (LC 3) — `HashMap<Character,Integer>` of last-seen index; on duplicate jump `left = max(left, lastSeen[c] + 1)`. Don't just increment left — you'll re-process characters.
3. **Minimum Size Subarray Sum** (LC 209) — variable window, shrink when `sum >= target`, track min length. O(n) despite two loops (each element enters/exits once).
4. **Longest Repeating Character Replacement** (LC 424) — invariant: `(windowSize - maxFreq) ≤ k`. Track `maxFreq` but never shrink it (you only care about a *longer* valid window).

**Block B (~2 h) — HashMap internals:**

- Structure: `Node<K,V>[] table` (array of buckets). Bucket index = `(n-1) & hash(key)`; default capacity 16, load factor 0.75 → resize at 12.
- `hash(key) = key.hashCode() ^ (h >>> 16)` — the bit-spread XORs high bits into low so they influence the index in small tables.
- Collisions: linked list per bucket → converts to **red-black tree** when bucket length hits **8 AND table capacity ≥ 64** (else it resizes instead). O(n) → O(log n) per bucket under heavy collision. Untreeify at 6 (hysteresis band).
- Resize: double capacity; Java 8+ split trick — `hash & oldCapacity` is 0 (stay) or 1 (move to `oldIndex + oldCapacity`), avoiding a full rehash. Pre-size with `new HashMap<>(expected/0.75 + 1)` rounded to next power of 2 to dodge resizes.
- Thread safety: `HashMap` is not safe; `ConcurrentHashMap` uses CAS + per-bucket `synchronized` (Java 8+). Avoid `Collections.synchronizedMap(...)` in hot paths — global lock.
- Tie to Smart360: Redis cache uses string keys; `String.hashCode()` (polynomial rolling hash, cached after first call) is why `String` is the ideal HashMap key.

**Block C (~1.5 h) — subarray-sum + advanced sliding window:**

- **Subarray Sum Equals K** (LC 560) — prefix sum + HashMap: look for `prefixSum - k` in the map; seed `map.put(0, 1)` before the loop (else you miss subarrays starting at index 0). This bridges Monday's prefix-sum intro into Wednesday's set.
- Stretch: **Minimum Window Substring** (LC 76) preview — two frequency maps + the "have vs need" counter trick; expand right until covered, shrink left while still valid, record min window. (Full timed attempt Saturday.)

**Self-check:**
1. Two HashMap lookups return different results for the same key object — most likely cause? (`hashCode` based on mutable fields that changed; object now in a different bucket.)
2. When does Java treeify a bucket and what does it improve? (Length 8 with capacity ≥ 64; worst-case bucket lookup O(n) → O(log n).)

---

### Wednesday Jul 1 — 3Sum / Prefix Sum + Generics & Autoboxing

📌 **Study today:** Multi-pointer 3Sum family + prefix sum (LC 15 · 560 · 238 · 189) · Generics, type erasure & autoboxing · hashing (LC 242 · 49)

**Block A (~2.5 h) — Multi-pointer (3Sum family) + prefix/array tricks:**

The 3Sum trick is reduce-to-two-pointer: sort, fix one element, run two-pointer on the rest, skip duplicates explicitly.

1. **3Sum** (LC 15) — sort O(n log n); fix `nums[i]`, two pointers on `[i+1, n-1]`; skip duplicates at the outer loop AND both inner pointers (common bug: forgetting to skip after a valid triplet).
2. **Subarray Sum Equals K** (LC 560) — re-solidify the prefix-sum + HashMap pattern; explain the `{0:1}` seed. Curveball: count subarrays with sum *divisible* by K (LC 974 — `prefix % k` as key).
3. **Product of Array Except Self** (LC 238) — no division; prefix-product then suffix-product pass. O(n) time, O(1) extra (output array excluded). Famous Meta/Google question — know it cold.
4. **Rotate Array** (LC 189) — three-reversal trick: reverse all, reverse first k, reverse last n−k. O(n)/O(1). Visualize indices to see why it works.

**Block B (~2 h) — Generics, type erasure & autoboxing:**

- **Type erasure**: generics are compile-time only; at runtime `List<String>` and `List<Integer>` are both `List`. So: no `instanceof` with generic types, no `new T[]`, no overload on erasure-equal signatures.
- **Bounded wildcards / PECS**: `? extends T` (producer, read-only, covariant), `? super T` (consumer, write-only, contravariant). `Collections.copy(List<? super T> dest, List<? extends T> src)`.
- **Autoboxing pitfalls (real production bugs):**
  - `Integer` cached −128…127 → `a == b` true at 127, false at 128. Always `.equals()`.
  - `Long sum = 0L; for (int i : list) sum += i;` → box/unbox per iteration = GC pressure in tight loops. Prefer primitives / `IntStream.sum()`.
  - Null unboxing: `Integer x = null; int y = x;` → NPE. Insidious with nullable DB columns mapped to `Integer`.
  - HashMap key boxing: `map.get(new Long(id))` misses if keys were inserted as autoboxed `long`. Use explicit `Long` + `.equals()` in cache lookups.
- Practical rule: primitives in hot paths; wrappers only when null is semantically meaningful.

**Block C (~1.5 h) — Hashing (frequency maps & canonical keys):**

1. **Valid Anagram** (LC 242) — `int[26]` increment/decrement → all-zero check, O(n)/O(1). Curveball: Unicode → `HashMap<Character,Integer>`.
2. **Group Anagrams** (LC 49) — HashMap with canonical key: sorted string, or `int[26]` count → `Arrays.toString(count)` (O(26) per word, avoids the O(k log k) sort). `Map<String, List<String>>`. Tie to Smart360: role/permission dedup uses the same canonical-key pattern.

**Self-check:**
1. Why does `List<Dog>` not satisfy `List<Animal>` though `Dog extends Animal`? (Covariance would let you insert a `Cat`; Java uses wildcards instead.)
2. You have a nullable `Integer` DB column — write the safe null-check before unboxing. (`if (e.getCount() != null) { int c = e.getCount(); }` or `Objects.requireNonNullElse`.)

---

### Thursday Jul 2 — Hashing/Top-K + Collections Deep-Dive + Bits

📌 **Study today:** Hashing / Top-K (LC 347 · 128) · Collections deep-dive (ArrayList/LinkedList, ConcurrentHashMap, fail-fast) · bit manipulation (LC 136 · 191 · 338 · 268)

**Block A (~2.5 h) — Top-K & frequency patterns:**

1. **Top K Frequent Elements** (LC 347) — (A) bucket sort O(n): count in HashMap, `List<Integer>[]` indexed by frequency, scan high→low; (B) min-heap O(n log k): `PriorityQueue` of size k. Bucket sort when k ≈ n; heap when k ≪ n. Code the bucket-sort version — it impresses (O(n) vs the obvious O(n log n)). Tie to Smart360 Analytics top-K accessed reports.
2. **Longest Consecutive Sequence** (LC 128) — HashSet, O(n) NOT sorting. For each `num`, only start counting if `num-1` ∉ set (it's a sequence start), then count `num+1, num+2…`. Interviewers specifically ask for O(n).

**Block B (~2 h) — Collections deep-dive:**

- **ArrayList**: `Object[] elementData`, default capacity 10, growth `oldCap + (oldCap >> 1)` (50%, vs Vector's 100% — less wasted memory). `add(index,e)` = `System.arraycopy` (O(n) but cache-friendly/vectorizable). `trimToSize()` after bulk load.
- **LinkedList**: doubly-linked `Node{item,next,prev}`; ~48 B/node vs ~8 B array slot. `add(0,e)` O(1) rewire; `get(i)` O(n) (iterates from nearer end). For lists of ~1000, ArrayList `add(0,e)` beats LinkedList due to cache effects. **Rule: default ArrayList; ArrayDeque for queue/stack; LinkedList only if profiled.**
- **ConcurrentHashMap (Java 8+)**: empty bucket → CAS insert (lock-free); non-empty → `synchronized` on the bin head only; reads lock-free (`volatile`); cooperative resize via `ForwardingNode`; `size()` uses `LongAdder`-style striped counters; null keys/values forbidden (sentinel ambiguity + TOCTOU). Java 5–7 trivia: `Segment[]` striped locks. vs `Hashtable` = single full-map lock (throughput bottleneck).
- **Fail-fast vs fail-safe**: fail-fast (`ArrayList`/`HashMap`/`TreeMap`) snapshot `modCount` at iterator creation; `next()` throws `ConcurrentModificationException` if it changed (best-effort, single-thread misuse detection). Note `set()` does NOT bump `modCount`. Fail-safe (`CopyOnWriteArrayList` snapshot, `ConcurrentHashMap` weakly-consistent). Trick: `for (s : list) list.remove(s)` always throws — fix with `Iterator.remove()` / `removeIf()`.

**Block C (~1.5 h) — Bit manipulation (4 canonical Easy):**

The week's hash internals lean on bit tricks (`h ^ (h >>> 16)`, `(n-1) & hash`, `hash & oldCapacity`) — same muscle. Drill these cold (~10 min each):

1. **Single Number** (LC 136) — XOR cancels pairs (`a^a=0`, `a^0=a`); fold with `result ^= num`. O(n)/O(1).
2. **Number of 1 Bits** (LC 191) — `n &= (n-1)` drops the lowest set bit; loops once per set bit. (`Integer.bitCount()` is the intrinsic.)
3. **Counting Bits** (LC 338) — DP on bits: `bits[i] = bits[i>>1] + (i&1)`. Whole 0..n table in O(n).
4. **Missing Number** (LC 268) — XOR all indices with values then `^= n`; the missing index survives. (Gauss `n(n+1)/2 - sum` alternative, but XOR avoids overflow.)

**Self-check:**
1. Why does ArrayList beat LinkedList on small lists even for insertions? (Contiguous memory → CPU prefetch/cache hits dominate pointer-chasing.)
2. Show the `modCount` fail-fast mechanism in a snippet; does `set()` throw? (No — `set()` is content, not structural, change.)

---

### Friday Jul 3 — Strings + Comparable/Comparator + Weak-Spot Review

📌 **Study today:** Strings (LC 5 · 271 · 680) · Comparable vs Comparator (TreeMap/TreeSet sorting) · mixed-pattern review / weak spots

> Quick String-immutability recap (bridging from Sun Jun 28 Week 0 refresh): every "modification" returns a new object; interning works only because of immutability; build in loops with `StringBuilder`, not `+`.

**Block A (~2.5 h) — String patterns:**

1. **Longest Palindromic Substring** (LC 5) — expand-around-center: for each index try odd (one center) and even (two centers) expansion; track best. O(n²)/O(1). (Mention Manacher's O(n) exists.)
2. **Encode and Decode Strings** (LC 271) — length-prefix encoding: each string → `len + "#" + s` (e.g. `"4#lint4#code"`). Decode by reading digits to `#`, parse length, extract exactly that many chars. The length (not just `#`) makes it unambiguous when strings contain `#`. Tie to Deep Fathom's LLM-proxy framing protocol.
3. **Valid Palindrome II** (LC 680) — one deletion allowed; two pointers, on mismatch try skipping left OR right and check remainder.

**Block B (~2 h) — Comparable vs Comparator + TreeMap/TreeSet:**

- **Comparable** (`java.lang`, `int compareTo(T)`): natural ordering, implemented on the class; used automatically by `Arrays.sort`, `Collections.sort`, `TreeSet`/`TreeMap`. Must be consistent with `equals()`.
- **Comparator** (`java.util`, `int compare(T,T)`): external/ad-hoc ordering; define many per class. Java 8 `Comparator.comparing(...).thenComparing(...).reversed()`. Use for multiple/ runtime-chosen sort criteria, third-party classes, one-liner lambda sorts.
- **TreeMap/TreeSet gotcha**: no Comparator + key not `Comparable` → `ClassCastException` at runtime. And the **`BigDecimal` consistency bite**: `new BigDecimal("1.0").compareTo("1.00") == 0` but `.equals()` is false → `TreeSet` keeps one, `HashSet` keeps both.

**Block C (~1.5 h) — Mixed-pattern recognition / weak spots:**

Rank confidence Mon–Thu; re-solve your two lowest-confidence problems clean, no notes, ≤15 min each. Plus a recognition drill — identify the pattern *before* coding:

- **Find All Anagrams in a String** (LC 438) — sliding window + `int[26]` frequency compare (not sorted strings).
- **Maximum Subarray / Kadane** (LC 53) — `currentSum = max(num, currentSum + num)`; foundation for "max sum of length exactly k" (that's fixed-window, not Kadane — know which to apply).

**Self-check:**
1. Chain three Comparators with `.thenComparing()` in 30 seconds.
2. What happens to a TreeSet if you insert objects where `compareTo` returns 0 but `equals` returns false? (Treated as duplicates — only one stored.)

---

### Saturday Jul 4 — Timed Mixed DSA + SOLID/System-Design + Resume & Deep-Dive Docs

📌 **Study today:** Timed mixed DSA set (LC 76 · 152 · 239 · 452) · SOLID + system-design primer · resume rewrite start + project deep-dive docs · (self-study: JVM memory model + concurrency basics)

**Block A (~2.5 h) — Timed mixed DSA (25 min each, no hints):**

1. **Minimum Window Substring** (LC 76) — hardest sliding window: two frequency maps, "have vs need" counter; expand right until covered, shrink left while valid, record min window. O(n + m).
2. **Maximum Product Subarray** (LC 152) — Kadane extension: track `maxSoFar` AND `minSoFar` (neg × neg = pos); `newMax = max(num, maxSoFar*num, minSoFar*num)`.
3. **Sliding Window Maximum** (LC 239) — monotonic deque holding indices in decreasing value order; front is always the window max.
4. **Minimum Number of Arrows to Burst Balloons** (LC 452) — greedy/interval: sort by end, shoot at the first balloon's end. Intro to next week's interval pattern.

After each: write pattern name, time/space complexity, and one interviewer follow-up.

**Block B (~2 h) — SOLID + system-design primer:**

**SOLID (with your own code):**
- **SRP** — Smart360 `AuthorizationService` did token validation + role resolution; when it began handling user-management events it broke SRP → the strangler-fig extraction story.
- **OCP** — WebX LLM provider router: a new provider means implementing `LLMProvider`, not editing the router.
- **LSP** — `PremiumUser extends User` must work anywhere `User` does. Violation: overriding to throw `UnsupportedOperationException` (e.g. `java.util.Stack extends Vector` — `Stack.add(0,e)` breaks stack semantics).
- **ISP** — split a 20-method `UserService` into `UserReader`/`UserWriter`/`UserAuthenticator`.
- **DIP** — Spring DI: depend on `UserRepository`, not `UserRepositoryImpl`.
- **Inheritance vs Composition**: prefer composition (interface) — inheritance creates tight coupling / fragile base class. Inheritance only for true IS-A.

**System-design primer:**
- **Latency vs throughput**, and the numbers: L1 ~1ns, L2 ~4ns, RAM ~100ns, SSD random read ~100µs, same-DC network ~500µs, cross-region ~150ms.
- **Caching**: cache-aside (lazy, most common in Smart360), write-through, write-behind; LRU/LFU/TTL eviction; **cache stampede** fix (probabilistic early expiry, single-flight). **CDN**: edge cache, pull vs push, fingerprinted URLs over purge.
- **Load balancing**: L4 (TCP) vs L7 (HTTP); round-robin / least-connections / consistent hashing; sticky sessions are an anti-pattern (use JWT). **Horizontal vs vertical scaling** (Smart360 on Azure Container Apps + KEDA, scale-to-zero).
- **CAP**: during a partition choose C or A. PostgreSQL = CP; Cassandra/DynamoDB = AP.
- **Smart360 5-service whiteboard exercise** — redraw from memory; for each service state (1) single responsibility, (2) data owned, (3) sync vs async calls, (4) failure mode:
  1. **API Gateway** (Spring Cloud Gateway) — JWT validation, rate limiting (Redis), routing; stateless; SPOF → run replicas.
  2. **Authorization** — auth, JWT issuance, RBAC; owns `users/roles/permissions`; publishes `UserCreated/Updated`.
  3. **Data** — core CRUD + the 60s→2s query fix; owns `reports/s3_file_metadata/organizations`; Redis pre-signed-URL cache, EntityGraph to kill N+1; degrades to cached reads.
  4. **Visualization** — aggregations/chart data (CQRS read side); scales independently so heavy aggregations don't choke OLTP.
  5. **Notification** (event-driven) — subscribes to events; async/fire-and-forget so failures never block registration. Cross-cutting: Eureka/K8s DNS discovery, Micrometer tracing, Config, PostgreSQL RLS. Practice saying it in <3 min.

**Block C (~1.5 h) — Resume rewrite start + project deep-dive docs:**

Every bullet answers "So what? How much? How do I know?" — kill any bullet without a number or a named tech decision.

- **Smart360**: "Cut data retrieval latency 96% (60s → 2–3s) by eliminating N+1 Hibernate queries via EntityGraph, composite indexes, and caching S3 pre-signed URLs in Redis (80% S3 reduction)"; "Reduced sign-in latency 60% migrating session validation to stateless JWT"; "Built fine-grained RBAC via Spring Security + JWT claims across 5 services."
- **Deep Fathom**: "PostgreSQL Row-Level Security on 50+ tables — tenant isolation at the DB layer"; "Reduced CI/CD runtime 57% (23 → 10 min) via BuildKit parallel builds, registry layer caching, GitLab DAG restructuring"; "Bicep IaC across 30+ resources, identical staging + FedRAMP GovCloud from one codebase."
- **WebX**: "Async job architecture for 2–20 min LLM ops: 202 + jobId + SSE on completion; 5+ providers via unified proxy"; "Circuit-breaker graceful degradation on provider outages."
- **One-page deep-dive doc per project** (your verbal cheat sheet): What / Scale / Architecture / Key decisions (with trade-offs) / Hardest problem + fix / Numbers / What I'd do differently. E.g. Smart360 "would-do-differently": K8s-native discovery instead of Eureka; Postgres advisory locks instead of Redis for the token blacklist.

**Self-study note (woven from the old W1 full-mock day — read/skim ~30 min, no separate block):** keep the **JVM memory model + concurrency basics** warm — Heap/Stack/Metaspace/Code Cache/Direct Memory; G1GC generational hypothesis + `-XX:MaxGCPauseMillis`; string pool & `intern()`; `volatile` (visibility, not atomicity) vs `AtomicInteger`; `synchronized` vs `ReentrantLock` (`tryLock`, fairness); `ThreadLocal` (Spring `SecurityContextHolder`, pool-leak risk); `CompletableFuture` (`thenApply`/`thenCompose`/`allOf`/`exceptionally`, executor choice). These return for real in Week 3's concurrency block — today just don't let them go cold.

**Self-check (pass/fail for the week):**
1. Solve Container With Most Water and explain the pointer-move invariant in 90 seconds? (Target: yes)
2. Draw HashMap's internal structure (buckets, list, tree) from memory? (Target: yes)
3. Do all three project stories have at least 3 numbers each? (Target: yes)

---

### Sunday Jul 5 — Rest

No study. Rest, recover, and let the week consolidate. Optional: glance Saturday's self-assessment so Monday of Week 2 starts on target.

---

## 🧠 Concepts to Master This Week (depth — the "why", gotchas, follow-up angles)

### Arrays, Two Pointers, Sliding Window, Prefix Sum

Almost every array problem is one of: scan once (prefix/suffix), two ends converging, or a variable window. **State the invariant before coding** — for Container With Most Water: "volume is `min(h[l],h[r])*(r-l)`; move the shorter pointer because it's the bottleneck and keeping the taller one only shrinks width." Memorize the sliding-window template (expand `right`, shrink `left` while invariant violated, collect result). **Kadane gotcha**: `currentSum = max(num, currentSum+num)` handles the non-empty-subarray case by restarting. **Prefix-sum HashMap trick** enables O(n) "number of subarrays with sum = k" (seed `{0:1}`).

### `equals`/`hashCode` Contract

JPA entities stored in `Set<Role>` break with default identity `equals` when fetched in different Hibernate sessions. Best practice: `equals` on the business key or PK with the transient null-id case handled; a constant `hashCode` is safe (all in one bucket, O(n) set lookup, but never lost). Follow-up: "two keys, same hashCode, `equals` false?" → same bucket as a collision chain node; correct, just slower.

### HashMap Internals

Treeify needs BOTH bucket ≥ 8 AND capacity ≥ 64 (else resize). Load factor 0.75 is a Poisson-backed space-time sweet spot (P(bucket ≥ 8) < 1e-7). Resize split uses one bit (`hash & oldCapacity`). Curveball: "capacity 16, 10 entries — resize?" → no, threshold is 12; 13th insert doubles to 32. "Initial capacity for 1000 entries?" → 2048 (next power of 2 above 1000/0.75).

### Generics, Type Erasure, Autoboxing

Erasure lets Spring inject `@Autowired List<UserRepository>` (param erased; factory knows concrete types). `IntStream`/`LongStream` avoid boxing — measurably faster in numeric-heavy code. The 128 cache boundary is interview gold (configurable via `-XX:AutoBoxCacheMax`, defaults 127).

### Collections Framework

ArrayList 1.5× growth (vs Vector 2×) wastes less memory; arraycopy is cache-friendly. ConcurrentHashMap forbids null because in a concurrent context `get()==null` is ambiguous (key absent vs mapped-to-null) and `containsKey` disambiguation is a TOCTOU race. `modCount` is bumped by structural changes only (`set()` doesn't). `BigDecimal` in `TreeSet` vs `HashSet` differs because `compareTo` ignores scale but `equals` doesn't. `Comparable` must be consistent with `equals`.

### Sliding Window — "don't shrink maxFreq"

In LC 424, recomputing `maxFreq` on shrink makes it O(26n). You search for the LONGEST window, so the answer only ever grows; a smaller `maxFreq` would only yield a shorter window you don't care about. Track the historical max only.

---

## 🎤 Sample Interview Questions (incl. curveballs)

**DSA:**
1. "Solve Two Sum, now as an unbounded data stream." → HashSet: check `target - x` before adding `x`. O(n)/O(n).
2. "Max sum subarray of length exactly k." → fixed-size sliding window (NOT Kadane); `prefix[i+k] - prefix[i]`. Tests pattern selection.
3. [Curveball] "Your sliding-window HashMap — worst-case complexity?" → O(n) if all keys collide; for `char` keys use `int[128]` for true O(1).
4. "Walk through `hashMap.put("name","Jay")`." → hashCode → bit-spread → bucket → walk chain via `equals` → insert/update → resize check (+ treeify condition).
5. [Curveball] "`for (s : list) list.remove(s)` throws CME — three fixes." → `removeIf`, `Iterator.remove()`, collect-then-`removeAll`, or stream-filter to a new list.
6. "BigDecimal in TreeSet vs HashSet?" → TreeSet stores one (`compareTo` equal), HashSet stores both (`equals` considers scale).

**Java/Spring:**
7. "`@Transactional` method called from another method in the same class?" → self-invocation bypasses the proxy; no transaction. Fix: inject self or `AopContext.currentProxy()`.
8. "Two `UserRepository` beans — which is injected?" → ambiguity error; resolve with `@Primary` or `@Qualifier`; `List<UserRepository>` injects all.
9. "Why no null keys in ConcurrentHashMap?" → sentinel ambiguity + TOCTOU between `get` and `containsKey`.
10. "ArrayList growth — why 1.5× not 2×?" → less average wasted memory; resizes are amortized O(1) anyway.

**Architecture/Design:**
11. "Why PostgreSQL RLS over app-layer tenant filtering?" → enforced by the DB even with an app bug; trade-off is setting tenant context per connection. Test with Testcontainers (H2 lacks RLS).
12. "LLM job runs 20 min, user closes browser?" → job persisted in PostgreSQL, worker continues, user re-polls `GET /jobs/{id}`; SSE drops but result is durable.
13. "Cut CI/CD 23→10 min — how to reach 5?" → aggressive Docker/test caching, test-impact analysis, more parallel runners, move slow integration tests to nightly.
14. [Behavioral] "A technical decision you regret." → Eureka over K8s-native discovery in Smart360 — redundant infra once K8s arrived; learning: don't duplicate what the platform provides.

---

## 🌟 Extraordinary-Candidate Edge

1. **Narrate trade-offs, not just decisions** — "I chose DB-layer RLS over app filtering; here's the operational cost and what I'd change for multi-region." Senior vocabulary at 2.5 YOE.
2. **Connect algorithms to production** — sliding window → your rate limiter; HashMap internals → your Redis key design; prefix sum → PostgreSQL window functions (`SUM() OVER (...)`) pushed to the DB.
3. **Lead with failure modes** — Redis down → S3 cache-miss degrades gracefully; circuit breaker → cheaper-provider fallback. Signals production experience.
4. **Specific, prepared numbers** — "96% latency reduction (60s → 2–3s)" is remembered; "improved performance" is forgotten in 30 seconds.
5. **Go beyond the obvious in collections** — mention the bit-spread AND that capacity must be a power of 2 for `(n-1) & hash` to act as modulo; size caches to the next power of 2 above `expected/0.75`.

---

## 📊 End-of-Week Self-Assessment (pass/fail gates)

Complete on Saturday evening. Honest pass/fail only — no partial credit.

| Gate | Check | Pass Criteria |
|------|-------|--------------|
| DSA-1 | Container With Most Water cold | ≤ 15 min, correct O(n) |
| DSA-2 | Minimum Size Subarray Sum cold | ≤ 15 min, correct sliding window |
| DSA-3 | 3Sum cold | ≤ 25 min, handles duplicates |
| DSA-4 | Explain Kadane in 60 sec | States invariant, all-negative case |
| DSA-5 | Top K Frequent with bucket sort (no heap) | ≤ 20 min, O(n) |
| DSA-6 | Longest Consecutive Sequence O(n) HashSet | ≤ 15 min, no sorting |
| DSA-7 | Subarray Sum = K, `{0:1}` seed explained | ≤ 10 min |
| Bits | 4 bit problems (136/191/338/268) | each ≤ 10 min, "why XOR" articulated |
| Java-1 | Derive `equals`/`hashCode` contract | State + code + 1 JPA pitfall |
| Java-2 | Draw HashMap internals from memory | Buckets, list, tree threshold, load factor, resize split |
| Java-3 | Autoboxing 128 cache boundary | Correct + code example |
| Java-4 | ConcurrentHashMap CAS vs synchronized + null reason | Explained without "segment" |
| Java-5 | Fail-fast `modCount` (incl. `set()` edge case) | Code snippet, `set()` doesn't throw |
| Java-6 | Comparable vs Comparator + BigDecimal gotcha | Both + the consistency bite |
| SD | Smart360 5-service diagram from memory | Sync/async arrows + failure modes |
| Resume | All 3 projects have 3+ numbers each | Count them now |

**Score: 16/16 = ready to proceed to Week 2**
**Score: 12–15 = identify gaps, spend 30 extra min Monday on the failed gates**
**Score: < 12 = repeat the weakest day before moving on — do not skip to Week 2**

---

*Week 1 of 12 — next: Week 2 (Mon Jul 6 – Sat Jul 11).*
