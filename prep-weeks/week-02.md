# Week 2 — Foundations (Jun 22–28, 2026)

> Hash the strings, understand the collections internals, and sketch your own architecture on a whiteboard.

---

## 🎯 Week Goal

By Sunday you will have drilled Hashing + Strings patterns cold (8 canonical problems, 2 per day), internalized Java Collections from first principles — not just "what" but "why the data structure was designed that way" — and be able to whiteboard Smart360's 5-microservice topology with a sharp articulation of every architectural decision.

---

## ✅ By Sunday you can...

- [ ] Solve Group Anagrams, Valid Anagram, Top K Frequent Elements, Longest Consecutive Sequence, Valid Palindrome, Encode/Decode Strings, Longest Repeating Character Replacement, Subarray Sum Equals K from memory with correct time/space complexity
- [ ] Explain HashMap's internal resizing threshold (0.75f load factor), treeification trigger (8 entries), and the bit-spread formula `(h ^ (h >>> 16))` and WHY each constant was chosen
- [ ] Distinguish fail-fast vs fail-safe iterators with a live code example and name which collections fall in each camp
- [ ] Compare `ConcurrentHashMap` internal locking (CAS on empty buckets, `synchronized` on head node) vs `Hashtable` (full-map lock) with throughput implications
- [ ] Explain `Comparable` vs `Comparator` and name 3 real scenarios where you'd choose each
- [ ] Whiteboard Smart360 5-service diagram with request flow, data ownership, failure boundaries, and one sentence on why each service is a separate process

---

## 📅 Daily Checklist (Mon–Sun)

---

### Monday, Jun 22 — Hashing Foundations

**DSA Block A (40 min)**

- **Problem 1 — Valid Anagram** (LC 242 · Easy)
  - Pattern: Character frequency map / sorting
  - Method A: `int[26]` array increment for s, decrement for t, check all zeros — O(n) time, O(1) space
  - Method B: sort both strings, compare — O(n log n)
  - Why it matters: foundation for Group Anagrams; interviewers follow up with "Unicode strings?" → use `HashMap<Character, Integer>` instead of `int[26]`
  - Curveball follow-up: "What if strings can contain Unicode characters beyond ASCII?" Answer: replace `int[26]` with `HashMap<Character, Integer>`

- **Problem 2 — Group Anagrams** (LC 49 · Medium)
  - Pattern: HashMap with canonical key
  - Key insight: sorted string as map key — all anagrams share the same sorted form
  - Implementation: `Map<String, List<String>>` — `Arrays.sort(word.toCharArray())` as key
  - Alternative key (no sorting, O(26) per word): `int[26]` array → `Arrays.toString(count)` as key — avoids O(k log k) sort
  - Time: O(n · k log k) where k = max word length; Space: O(n·k)
  - Tie to resume: token/permission deduplication logic in Smart360 Authorization service uses the same "canonical key" pattern for grouping roles

**Core Topic Block B (50 min) — HashMap Internals Deep Dive**

1. **Internal Structure**: `Node<K,V>[] table` — array of linked list heads (or TreeNode heads)
2. **`hashCode()` + bit-spread**: `(h = key.hashCode()) ^ (h >>> 16)` — XORs high bits into low bits to reduce collisions in small tables. Without this, strings differing only in high bits would all land in the same few buckets.
3. **Bucket index formula**: `(n - 1) & hash` where n = table capacity (always a power of 2) — bitwise AND is cheaper than modulo
4. **Load factor 0.75**: empirical sweet spot — below 0.75 the table is spacious enough that chains stay short; above 0.75 collision rate rises disproportionately. Space/time tradeoff.
5. **Treeification at 8 entries**: based on Poisson distribution — with load factor 0.75, probability of a bin having ≥8 entries is ~0.00000006. If it DOES happen, it's a sign of a bad `hashCode()`; the tree prevents O(n) worst case.
6. **Untreeification at 6 entries**: hysteresis band (8 → tree, 6 → list) prevents thrashing when elements are repeatedly added/removed near the threshold
7. **Resize (rehash)**: when `size > capacity * loadFactor`, double the table. In Java 8+, each entry either stays at same index OR moves to `oldIndex + oldCapacity` — split by checking the new bit (`hash & oldCapacity`). Avoids full rehash.
8. **`putIfAbsent`, `computeIfAbsent`, `merge`**: essential for Smart360-style caching patterns

**Self-Check**

- [ ] Can you explain the bit-spread formula without notes?
- [ ] Can you state when treeification triggers and what data structure replaces the linked list?
- [ ] Can you code Valid Anagram two ways in <5 min?

---

### Tuesday, Jun 23 — Top K & Frequency Patterns

**DSA Block A (40 min)**

- **Problem 3 — Top K Frequent Elements** (LC 347 · Medium)
  - Pattern: HashMap + bucket sort OR HashMap + min-heap
  - Method A (bucket sort — O(n)): count frequencies in HashMap; create `List<Integer>[]` of size n+1 indexed by frequency; fill buckets; scan from high to low collecting top k
  - Method B (min-heap — O(n log k)): count frequencies; maintain a `PriorityQueue<Integer>` of size k ordered by frequency ascending; scan map entries, add to heap, poll when size > k
  - Which to use: bucket sort when k is unspecified or k ≈ n; heap when k << n and memory matters
  - Code the bucket sort version — it impresses interviewers with O(n) vs the obvious O(n log n)
  - Tie to Smart360: "In the Analytics service, finding the top-K accessed reports by a user — same bucket sort pattern"

- **Problem 4 — Longest Consecutive Sequence** (LC 128 · Medium)
  - Pattern: HashSet — O(n) time, NOT sorting
  - Key insight: for each number n, only start counting if `n-1` is NOT in the set (n is the start of a sequence)
  - Common mistake: using a sorted structure or sorting — that's O(n log n). Interviewers specifically ask for O(n).
  - Build `HashSet<Integer>` from array. For each num: if `set.contains(num-1)` → skip (not the start). Otherwise count consecutive `num+1, num+2...` while present in set.
  - Time: O(n), Space: O(n)

**Core Topic Block B (50 min) — ArrayList vs LinkedList Internals + When Each Wins**

**ArrayList deep dive:**
- Backing: `Object[] elementData` — contiguous memory
- Initial capacity: 10 (default constructor). `new ArrayList<>(0)` starts at 0, grows lazily
- Growth formula: `int newCapacity = oldCapacity + (oldCapacity >> 1)` → 50% growth
- `add(index, element)`: calls `System.arraycopy()` — O(n) shift but extremely cache-friendly in practice. The JVM/JIT can vectorize arraycopy.
- Memory layout advantage: CPU prefetcher can anticipate next element → L1/L2 cache hits
- `trimToSize()`: shrinks backing array to exact size — useful after bulk-loading a stable list

**LinkedList deep dive:**
- Backing: `Node<E>` — `{E item; Node<E> next; Node<E> prev;}` — doubly linked
- Each node: ~48 bytes (object header 16B + 3 references × 8B each + padding) vs ArrayList element: ~8 bytes (reference in array)
- `add(0, e)`: O(1) pointer rewire — `node.prev = null; head = node`
- `get(i)`: `if i < size/2` iterate from head, else from tail — still O(n)
- Real winner scenario: deque operations (`addFirst`, `removeFirst`, `addLast`, `removeLast`) with LinkedList as `Deque` — but `ArrayDeque` beats even LinkedList here due to cache

**Benchmark reality (know for interviews):**
- For lists of ~1000 elements, ArrayList `add(0, e)` is FASTER than LinkedList due to cache effects
- LinkedList only wins with millions of elements + very frequent mid-insertions + elements are large objects (pointer-chasing overhead shrinks relative to object fetch time)
- **Rule of thumb**: default to `ArrayList`. Switch to `ArrayDeque` for queue/stack. Use `LinkedList` only if you've profiled.

**Self-Check**

- [ ] Can you code Top K Frequent with bucket sort (no heap)?
- [ ] Can you explain why `ArrayList` beats `LinkedList` on small lists even for insertions?
- [ ] What is the ArrayList initial capacity and growth formula from memory?

---

### Wednesday, Jun 24 — Concurrent Collections + Fail-Fast/Fail-Safe

**DSA Block A (40 min)**

- **Problem 5 — Valid Palindrome** (LC 125 · Easy — but has curveballs)
  - Pattern: Two pointers
  - Implementation: two pointers l=0, r=len-1; skip non-alphanumeric with `Character.isLetterOrDigit()`; compare `Character.toLowerCase(c1) == Character.toLowerCase(c2)`; O(n) time, O(1) space
  - Curveball: "What about Unicode? What about emojis?" → `Character.isLetterOrDigit()` handles basic Unicode; supplementary code points (emoji) need `Character.codePointAt()` + `Character.isLetterOrDigit(codePoint)` — know this exists even if you don't need to code it
  - Follow-up: LC 680 "Valid Palindrome II" — one deletion allowed. Use the same two-pointer approach; when mismatch found, try skipping left OR right, check if remainder is palindrome.

- **Problem 6 — Encode and Decode Strings** (LC 271 / LintCode 659 · Medium)
  - Pattern: Length-prefix encoding
  - Encode: for each string, prepend `len(s) + "#" + s` — e.g., `["lint","code"]` → `"4#lint4#code"`
  - Decode: read digits until `#`, parse length, extract exactly that many chars, advance pointer
  - Why `#` delimiter? Not enough — need the length to handle strings that contain `#`. The length makes it unambiguous.
  - Tie to resume: LLM proxy in Deep Fathom used a similar framing protocol to pack multiple prompts into a single API call

**Core Topic Block B (50 min) — ConcurrentHashMap Internals + Fail-Fast vs Fail-Safe**

**ConcurrentHashMap Java 8+ internals (NOT the old segment/striped-lock design from Java 5–7):**
- **Empty bucket → CAS insert**: if the target bucket is null, use `Unsafe.compareAndSwapObject()` to insert atomically — no lock at all
- **Non-empty bucket → `synchronized(bin head)`**: locks only the first node of that bucket's chain. Other buckets are fully accessible concurrently.
- **Read (get)**: fully lock-free — `volatile` reads on nodes
- **Resize**: cooperative "forwarding" — threads helping with resize find `ForwardingNode` at old bucket and cooperatively copy
- **`size()`**: uses `LongAdder`-style striped counters (`counterCells`) — avoids single shared counter becoming a bottleneck
- **`null` keys/values forbidden**: because `null` is used as a sentinel to indicate "not present" — a `null` value in a concurrent context makes `containsKey` vs `get` return values ambiguous

**ConcurrentHashMap Java 5–7 (know for trivia):**
- Used `Segment[]` — 16 segments by default, each a mini `ReentrantLock`-backed hash table
- Maximum concurrency = number of segments

**Fail-Fast vs Fail-Safe — the real mechanism:**
- Fail-fast (`ArrayList`, `HashMap`, `HashSet`, `TreeMap`): maintain `int modCount` field. Every structural modification increments `modCount`. The iterator snapshot the `modCount` at creation time (`expectedModCount`). Every `next()` call checks `if modCount != expectedModCount → throw ConcurrentModificationException`. This is BEST-EFFORT — not a guarantee under concurrent access (only for single-thread misuse detection).
- Fail-safe (`ConcurrentHashMap`, `CopyOnWriteArrayList`): either iterate over a snapshot copy (COWAL) or use weakly consistent traversal (`ConcurrentHashMap` iterator reflects state at or after creation — may or may not show concurrent insertions)
- Interview trick: `for (String s : list) { list.remove(s); }` — always `ConcurrentModificationException`. Fix: use `Iterator.remove()` or `removeIf()` or collect-then-remove.

**Self-Check**

- [ ] Can you explain CAS insert in ConcurrentHashMap without using the word "segment"?
- [ ] Can you show the `modCount` fail-fast mechanism in a code snippet?
- [ ] Valid Palindrome coded with `Character.isLetterOrDigit()` in <4 min?

---

### Thursday, Jun 25 — Comparable vs Comparator + String Patterns

**DSA Block A (40 min)**

- **Problem 7 — Longest Repeating Character Replacement** (LC 424 · Medium)
  - Pattern: Sliding window + character frequency map
  - Key invariant: window is valid if `(windowLength - maxFrequency) <= k`
  - Why: the characters we "replace" = window size minus the count of the most frequent character in the window. If that count ≤ k, we can make the whole window one character with k replacements.
  - Important nuance: you never need to SHRINK `maxFreq` when shrinking the window — you only care about the global max found so far (because you're looking for the LONGEST valid window, not checking every window)
  - Time: O(26·n) → O(n), Space: O(26) → O(1)
  - Tie to resume: sliding window is the same mental model as Smart360's Redis TTL window for rate limiting

- **Problem 8 — Subarray Sum Equals K** (LC 560 · Medium)
  - Pattern: Prefix sum + HashMap
  - Key insight: `sum[i..j] = prefixSum[j] - prefixSum[i-1]`. We want `prefixSum[j] - prefixSum[i-1] = k`, i.e., look for `prefixSum[j] - k` in the map.
  - Implementation: `Map<Integer, Integer> prefixCount = new HashMap<>()` seeded with `{0: 1}` (empty subarray). For each element: update `currentSum`, check `prefixCount.get(currentSum - k)`, then increment `prefixCount[currentSum]`.
  - Common mistake: forgetting to seed `{0: 1}` — causes missed subarrays starting at index 0
  - Time: O(n), Space: O(n)
  - Why `{0: 1}`? The subarray `[0..j]` itself has sum k when `prefixSum[j] - k = 0`. Without seeding, this case is missed.

**Core Topic Block B (50 min) — Comparable vs Comparator, TreeMap/TreeSet, Sorting Deep Dive**

**Comparable (`java.lang`):**
- `int compareTo(T o)` — defines the object's NATURAL ordering
- Implemented ON the class itself — the class "knows" how it should be sorted
- `Arrays.sort()`, `Collections.sort()`, `TreeSet`/`TreeMap` use it automatically
- Return convention: negative → this < o, zero → equal, positive → this > o
- Must be consistent with `equals()` — violating this breaks `TreeSet` behavior (treats unequal objects as equal if `compareTo` returns 0)

```java
// Smart360 example: User sorted by creation date by default
public class User implements Comparable<User> {
    private LocalDateTime createdAt;
    
    @Override
    public int compareTo(User other) {
        return this.createdAt.compareTo(other.createdAt); // natural order: oldest first
    }
}
```

**Comparator (`java.util`):**
- `int compare(T o1, T o2)` — defines EXTERNAL, ad-hoc ordering
- Separate from the class — you can define multiple Comparators for the same class
- Java 8+: `Comparator.comparing()`, `.thenComparing()`, `.reversed()` chain idioms

```java
// Multiple sort orders for the same Report entity
Comparator<Report> byDate    = Comparator.comparing(Report::getCreatedAt).reversed();
Comparator<Report> byOwner   = Comparator.comparing(Report::getOwnerName);
Comparator<Report> byDateThenOwner = byDate.thenComparing(byOwner);

reports.sort(byDateThenOwner);
```

**When Comparable:**
- Core domain objects with ONE obvious natural order (User by ID, Product by price, Report by date)
- Used as TreeMap/TreeSet keys where you don't pass a Comparator

**When Comparator:**
- Multiple sort criteria on the same type
- Sorting third-party classes you can't modify
- Sort order changes at runtime (user picks "sort by name" vs "sort by date")
- Lambda/method reference — one-liner sorts without a full class

**TreeMap/TreeSet gotcha**: if you use them without a Comparator and the key type doesn't implement Comparable → `ClassCastException` at runtime, not compile time.

**Self-Check**

- [ ] Can you code Subarray Sum = K with prefix sums in <7 min including the seed `{0:1}`?
- [ ] Can you chain three Comparators with `.thenComparing()` in 30 seconds?
- [ ] What happens to a TreeSet if you insert objects with `compareTo` returning 0 but `equals` returning false?

---

### Friday, Jun 26 — Review + Weak Spots + Collections Synthesis

**DSA Block A (40 min) — Re-attempt the 2 hardest problems from Mon–Thu without notes**

Rank your confidence from Mon–Thu. Pick your two lowest-confidence problems. Re-solve them clean on paper or in an editor with no notes. Time yourself. Target: each solved in ≤15 min.

Common weak spots to watch:
- Group Anagrams: forgetting the alternative O(n) key (int[26] count array vs sort)
- Longest Consecutive Sequence: accidentally using a sorted set instead of HashSet
- Subarray Sum: forgetting the `{0:1}` seed
- Longest Repeating: forgetting why you don't shrink maxFreq on window contraction

**Core Topic Block B (50 min) — Collections Framework Synthesis + Interview Stress Test**

Synthesis exercise — answer these WITHOUT looking at notes:

1. You have a multi-threaded Spring Boot service tracking request counts per IP address. Which collection? Why not `HashMap`? Why not `Hashtable`? → `ConcurrentHashMap<String, AtomicLong>` or use `merge()`. HashMap: race condition on concurrent puts. Hashtable: full-map lock → serialized access, throughput bottleneck.

2. You need a list of event listeners. Reads happen on every request (100/sec). Writes happen when an admin adds a listener (once per hour). → `CopyOnWriteArrayList` — reads lock-free, writes rare so copy cost is acceptable.

3. A method needs to return both a unique set of tag names AND maintain the order they were first seen. → `LinkedHashSet<String>`.

4. You're building a priority retry queue — failed API calls retried by priority. → `PriorityBlockingQueue<RetryTask>` with Comparable or Comparator on priority field.

5. You have an `EnumMap<Permission, List<User>>` vs a `HashMap<Permission, List<User>>`. Why choose EnumMap? → O(1) all ops backed by array indexed by ordinal, more memory-efficient, iteration in declaration order.

**Self-Check**

- [ ] Can you articulate the resizing flow for a HashMap loaded to 75% capacity, step by step?
- [ ] Can you name the 4 rejection policies of `ThreadPoolExecutor` and when to use `CallerRunsPolicy`?

---

### Saturday, Jun 27 — System Design Primer (4 hr)

**Morning: Foundations (2 hr)**

**Client–Server model:**
- Client initiates request over TCP/HTTP; server responds
- Every Spring Boot microservice is a server; it's also a client when calling downstream services
- HTTP 1.1 vs HTTP/2: multiplexing (multiple streams on one TCP connection) — relevant to API Gateway → downstream calls under load

**Latency vs Throughput:**
- Latency: time for one request (milliseconds) — optimize with caching, fewer hops, faster algorithms
- Throughput: requests per second — optimize with horizontal scaling, async processing, batch ops
- They are in tension: adding redundancy (replication) improves throughput but may add latency (consistency checks)
- Know numbers: L1 cache ~1ns, L2 ~4ns, RAM ~100ns, SSD random read ~100µs, network same DC ~500µs, cross-region ~150ms

**Caching:**
- Where to cache: in-process (Caffeine — sub-ms), distributed (Redis — 1–2ms), CDN (static assets, ~10ms globally)
- Cache-aside (lazy loading): app checks cache, on miss loads from DB and populates cache — most common pattern in Smart360
- Write-through: write to cache AND DB simultaneously — consistent but slower writes
- Write-behind (write-back): write to cache immediately, async flush to DB — fast writes, risk of data loss
- Eviction policies: LRU (most common), LFU (penalizes bursty access), TTL-based
- Cache stampede / thundering herd: on cold start, 1000 concurrent requests all miss cache and hammer DB. Fix: probabilistic early expiry, single-flight (only one request populates, others wait)

**Load Balancing:**
- Layer 4 (TCP): route by IP/port — fast, no content inspection (AWS NLB)
- Layer 7 (HTTP): route by path, header, cookie — smarter, enables A/B testing (AWS ALB, NGINX, Spring Cloud Gateway)
- Algorithms: round robin, weighted round robin, least connections, consistent hashing (for stateful backends like cache sharding)
- Sticky sessions: route same user to same server — needed for in-memory session state (anti-pattern in microservices; avoid with JWT)

**Horizontal vs Vertical Scaling:**
- Vertical (scale-up): bigger machine — limit is the biggest machine AWS offers; single point of failure
- Horizontal (scale-out): more machines — requires stateless services, distributed coordination, load balancer. The default choice in microservices.
- Smart360 on Azure Container Apps: horizontal scaling via KEDA (Kubernetes Event Driven Autoscaler) — scale to zero when idle, scale out on HTTP traffic.

**CAP Theorem (intro):**
- Consistency: every read gets the most recent write
- Availability: every request gets a response (not necessarily latest data)
- Partition tolerance: system works despite network splits
- During a partition, choose C or A. PostgreSQL chooses CP. Cassandra/DynamoDB choose AP.

**Afternoon: Smart360 Architecture Exercise (2 hr)**

**Task: Redraw Smart360's 5-microservice architecture from memory. For each service, answer:**
1. What is its single responsibility?
2. What data does it OWN (its DB tables / schemas)?
3. What does it call synchronously vs asynchronously?
4. What happens if it goes down — what degrades gracefully vs what fails hard?

**The 5 services (reconstruct from resume):**

**1. API Gateway (Spring Cloud Gateway)**
- Responsibility: Single entry point. JWT validation (pre-filter), rate limiting (Redis-backed RequestRateLimiter), routing to downstream services, CORS, request logging
- Data owned: none (stateless) — Redis for rate limit counters
- Sync calls: routes to Authorization, Data, Visualization services
- Failure mode: if down, entire system is unreachable. This is the blast radius concern. Mitigate with multiple replicas + health checks.
- Why separate? Cross-cutting concerns shouldn't pollute business services. Changing rate limiting logic shouldn't require redeploying the Data service.

**2. Authorization Service**
- Responsibility: User authentication, JWT issuance, RBAC — repository/table/permission level enforcement
- Data owned: `users`, `roles`, `permissions`, `role_permissions` tables
- Sync calls: called by API Gateway for JWT validation (or Gateway validates signature locally — know both options)
- Async: publishes `UserCreated`, `UserUpdated` events for downstream consumption
- Why separate? Security surface is isolated. A vulnerability in the Data service doesn't give attacker access to auth logic. Independent deployment of security patches.

**3. Data Service**
- Responsibility: Core business data CRUD, query optimization, the 60s→2s query fix lives here
- Data owned: main business entities — `reports`, `s3_file_metadata`, `organizations`
- Sync calls: S3 (pre-signed URL generation), downstream Visualization service
- Optimization: Redis caching of pre-signed URLs (TTL < S3 URL expiry), JOIN FETCH / EntityGraph to kill N+1
- Failure mode: graceful degradation — return cached data; queue writes for retry via outbox pattern

**4. Visualization Service**
- Responsibility: Rendering/generating visualization data (chart data, aggregations)
- Data owned: pre-aggregated read models (CQRS read side), possibly a separate read DB or materialized views in PostgreSQL
- Sync calls: queries from frontend via API Gateway; reads from Data Service on demand
- Why separate? Aggregation queries are CPU-heavy — scaling this service independently prevents it from choking the OLTP Data service

**5. Notification Service (event-driven)**
- Responsibility: Email/in-app notifications for user events
- Data owned: `notification_log`, `notification_preferences`
- Async: subscribes to events from Authorization service (`UserCreated` → welcome email) and Data service (`ReportReady` → notify owner)
- Why async? Notification failure should NEVER fail a user registration or report generation. Fire-and-forget. Decoupled.
- Why separate? Notification logic (templates, channels, preferences) changes frequently without affecting core auth or data logic.

**Cross-cutting: what ties them together**
- Eureka (dev) or Kubernetes DNS (prod) for service discovery
- Micrometer Tracing with `traceId` propagated via `traceparent` header across all service calls
- Spring Cloud Config for shared configuration
- Azure Container Apps for runtime; Bicep IaC for provisioning
- PostgreSQL row-level security for multi-tenant isolation at DB layer

**Practice saying this out loud in under 3 minutes.** Record yourself and play it back.

**Self-Check**

- [ ] Can you sketch the 5-service diagram on paper from memory with arrows showing sync vs async calls?
- [ ] Can you explain why Notification is async and what would break if it were sync?
- [ ] Can you define CAP theorem and say which side PostgreSQL picks?

---

### Sunday, Jun 28 — Mock Interview Day (4 hr)

**Morning: Mock DSA Session (1.5 hr)**

Simulate a real interview. Use a blank editor (no autocomplete hints). Timer running.

Round 1 (45 min):
- Problem: **Group Anagrams** — code it, analyze complexity, discuss follow-ups
- Curveball: "Now the input has 10 million strings. What changes?" → streaming approach, distributed grouping (MapReduce-style), external sort for memory constraints

Round 2 (45 min):
- Problem: **Subarray Sum Equals K** — code it, then: "What if K can be negative?" (same algorithm works), "What if the array contains only positives?" (sliding window works — know the tradeoff)
- Curveball: "Extend to find the number of subarrays with sum divisible by K" (LC 974 — `prefix % k` as key)

**Afternoon: Mock System Design + Behavioral (2.5 hr)**

**System Design Mock (1 hr):**

Prompt: "Design a URL shortener (like bit.ly) for 100M URLs, 1B reads/day."

Walk through the standard framework but inject your real experience:
1. Requirements: short URL generation, redirect, analytics (optional)
2. Capacity: 1B reads/day = ~11,600 req/sec. Write: 100M URLs total, ~100 new/day = negligible
3. Core algorithm: base62 encode a counter OR md5 hash with collision handling. Discuss tradeoffs (sequential IDs leak volume, hash needs collision detection)
4. DB: write once, read many → cache aggressively. PostgreSQL for storage, Redis for hot URLs (80/20 rule: cache top 20% serving 80% of traffic)
5. Scaling: stateless redirect service behind load balancer; Redis cluster with consistent hashing; read replicas for DB
6. What you'd do differently from Smart360: Smart360 used Redis for S3 URL caching — same pattern as short URL cache

**Behavioral Mock (1 hr):**

Answer these out loud, timed, with the STAR structure:

1. "Walk me through the most complex technical problem you've solved." → 60s→2s query performance story
2. "Tell me about a time you disagreed with a technical decision." → prepare a story; if none, adapt the synchronous→event-driven migration debate
3. "How do you handle a production incident at 2 AM?" → tie to Azure Monitor alerts, runbook, postmortem culture
4. "Why are you leaving your current company?" → frame around growth ceiling, wanting to work at product scale, bigger comp opportunity; NEVER disparage current employer
5. "What's a technical concept you learned recently that changed how you think?" → LLM async job queue pattern? K8s KEDA event-driven scaling? Row-level security in PostgreSQL?

**End-of-Day Review (30 min):**
- Update this checklist with actual completion status
- Write 3 sentences on what you got wrong today and the exact fix
- Add any new curveball questions you encountered to `interview-qa.md`

**Self-Check**

- [ ] URL shortener design explained coherently in <12 minutes?
- [ ] All 5 behavioral questions answered with specific numbers and outcomes?
- [ ] All 8 DSA problems solved at least once without notes this week?

---

## 🧠 Concepts to Master This Week (Depth, Gotchas, Follow-ups)

### HashMap Resizing — the full picture

When `size > capacity * 0.75`, a new table of `2 * capacity` is allocated. Every entry is re-hashed. In Java 8+, the trick: `hash & oldCapacity` is either 0 (entry stays at `oldIndex`) or 1 (entry moves to `oldIndex + oldCapacity`). This elegant bit trick halves the rehash work — you only check ONE new bit per entry.

**Gotcha**: `HashMap` is NOT thread-safe during resize. Two threads resizing simultaneously can create infinite loops in the linked list (Java 7 bug — Java 8 tail insertion fixed this, but concurrent modification is still undefined behavior).

**Follow-up interviewers ask**: "What initial capacity would you use for a HashMap storing exactly 1000 entries?" → `new HashMap<>(2048)` — the next power of 2 above `1000 / 0.75 = 1334`. This avoids a resize.

### ConcurrentHashMap — the `null` question

Interviewers love asking: "Why doesn't ConcurrentHashMap allow null keys?" The real answer: in a concurrent context, `map.get(key)` returning `null` is ambiguous — does the key not exist, or does it map to null? You'd need `containsKey()` to disambiguate. But between `get()` and `containsKey()`, another thread could remove the key. This TOCTOU race makes null keys/values semantically dangerous. HashMap can get away with it because it's single-threaded.

### Fail-Fast Internals — `modCount` details

`ArrayList.modCount` is incremented by: `add()`, `remove()`, `clear()`, `set()` (no, `set()` does NOT modify modCount — it's structural vs content), and `ensureCapacity()`. Interviewers sometimes ask "does `set()` throw ConcurrentModificationException?" — no, because `set()` doesn't change modCount.

### `Comparable` vs `Comparator` — the consistency contract

`compareTo()` MUST be consistent with `equals()`: `(x.compareTo(y) == 0) == (x.equals(y))`. `BigDecimal` famously violates this: `new BigDecimal("1.0").compareTo(new BigDecimal("1.00")) == 0` but `new BigDecimal("1.0").equals(new BigDecimal("1.00")) == false`. This causes `TreeSet<BigDecimal>` to treat them as equal (only one stored) while `HashSet<BigDecimal>` stores both. Know this example.

### Treeification details

The linked list → Red-Black Tree conversion in HashMap requires NOT just > 8 entries in the bucket, but ALSO the total table capacity ≥ 64. If capacity < 64, HashMap prefers to resize the table rather than treeify (spreading entries reduces collisions more efficiently). This matters: with a small initial capacity HashMap, you'll see resizes, not trees.

### Sliding Window — the "don't shrink maxFreq" insight

In Longest Repeating Character Replacement, many solutions incorrectly recompute `maxFreq` when shrinking the window, making it O(26n). The insight: you never need to decrease `maxFreq` because you're searching for the MAXIMUM window size. Once you've found a valid window of length L, you look for L+1 or larger. A smaller maxFreq would only yield a window ≤ L, which you don't care about.

---

## 🎤 Sample Interview Questions (Curveballs Included)

**1. "Walk me through exactly what happens when you call `hashMap.put("name", "Jay")`."**
Pointer: `hashCode("name")` → bit-spread → bucket index → walk bucket chain checking `equals()` → insert new `Node` or update existing value → check `size > threshold` → resize if needed. Mention treeification condition.

**2. "I see you used `ConcurrentHashMap` in Smart360 for rate limiting. Why not a `synchronized HashMap`? Why not `Hashtable`?"**
Pointer: `synchronized HashMap` requires external synchronization — easy to miss. `Hashtable` acquires a single lock on the entire map — one thread blocks all others. `ConcurrentHashMap` allows concurrent reads and parallel writes to different buckets. Under load (hundreds of requests/sec to the API Gateway), the throughput difference is significant.

**3. "What's the difference between `HashMap` and `TreeMap`? When would you use TreeMap in a microservice?"**
Pointer: HashMap O(1) avg, no order; TreeMap O(log n), sorted. In a microservice: use TreeMap for storing time-bucketed metrics (sorted by timestamp), building a sorted index of API endpoint access counts, or anywhere you need `subMap(start, end)` range queries. In Smart360 specifically: storing ordered configuration properties where you need to find all settings with a prefix range.

**4. "Curveball: You have a `List<User>` and you're iterating it with `forEach` to remove users matching a condition. This throws ConcurrentModificationException. Name three ways to fix it."**
Pointer: (1) `list.removeIf(u -> condition)` — Java 8, uses iterator internally correctly; (2) Collect to-remove items then `list.removeAll(toRemove)` after iteration; (3) Use `Iterator<User> it = list.iterator()` explicitly and call `it.remove()`; (4) `list.stream().filter(u -> !condition).collect(Collectors.toList())` — returns new list.

**5. "What is ArrayList's growth strategy and why 50%? Why not double like Vector?"**
Pointer: 50% growth (1.5x) vs Vector's 100% (2x) wastes less memory on average — important for large lists. Doubling can suddenly allocate double the memory even when only one element pushes the boundary. The 1.5x was chosen as a compromise: frequent small resizes are O(n) amortized anyway because each element is moved at most O(log n) times.

**6. "Curveball: BigDecimal in a TreeSet vs HashSet — describe the behavior difference and why."**
Pointer: TreeSet uses `compareTo()` — `new BigDecimal("1.0")` and `new BigDecimal("1.00")` compare as equal (same mathematical value), so TreeSet stores only one. HashSet uses `hashCode()` + `equals()` — `BigDecimal.equals()` considers scale, so they're unequal, and HashSet stores both. This is the `Comparable` inconsistency with `equals()` bite.

**7. "Subarray Sum = K but now the constraint is: only contiguous subarrays of length between `minLen` and `maxLen` are valid." How would you adapt?"**
Pointer: The prefix sum approach still works, but for each `j`, you look in the map for `prefixSum[j] - k` where the stored indices satisfy `j - index >= minLen && j - index <= maxLen`. Use a deque or sliding window on the map to restrict valid indices. This is a harder variant — the point is showing you understand the underlying invariant well enough to adapt it.

**8. "How do you handle the thundering herd / cache stampede problem in Smart360 when Redis is restarted and all caches are cold?"**
Pointer: Short answer used in practice: (1) probabilistic early expiry (Fetch PER algorithm) — recalculate cache before it expires with probability proportional to remaining TTL; (2) single-flight / mutex — first thread populates cache, others wait for it (Caffeine's `get()` with a loading function; Redisson `RLock`); (3) gradual warm-up at startup via `ApplicationReadyEvent`; (4) Redis persistence (`AOF` or `RDB`) so restart reloads data.

**9. "Curveball: You're given a HashMap that's been populated with 100,000 entries. An interviewer says the performance is bad. Without changing the data structure, what would you check first?"**
Pointer: Check `hashCode()` implementation of the key class. A bad `hashCode()` (constant, or collides frequently) causes all entries to land in one bucket → O(n) lookup. Use `HashMap.entrySet()` with a stream to analyze bucket distribution, or instrument with `System.identityHashCode()`. Second: check if keys are mutable objects and their `hashCode()` changed after insertion — this loses entries forever (the key is in the wrong bucket for its current hashCode).

**10. "In your Longest Repeating Character Replacement solution, you said you don't need to shrink `maxFreq`. Prove it."**
Pointer: Mathematical argument — we want the LONGEST valid window. If the current window of size `windowSize` used `maxFreq` as the most frequent count, then any valid window ≤ `windowSize` is uninteresting. When we shrink, we're just maintaining window size parity (looking for something bigger), not trying to find a SHORTER valid window. Formally: the answer can only increase or stay the same, never decrease. So `maxFreq` only needs to track the historical maximum.

**11. "What's the time complexity of `ConcurrentHashMap.size()` in Java 8?"**
Pointer: NOT O(1). It sums `LongAdder`-style `CounterCell[]` arrays — O(number of cells) which is bounded by parallelism level. Contrast with Java 7's `Segment`-based CHM where `size()` held locks on all segments. In practice it's near O(1) but technically O(parallelism). Interviewers appreciate knowing this nuance.

**12. "Curveball: Design the Authorization Service's JWT validation decision: should it validate at the Gateway or let downstream services validate independently?"**
Pointer: Trade-off. Gateway validation: one place to enforce security, downstream services trust the Gateway, simpler (no crypto in each service). Downstream validation: defense in depth, services work if Gateway is bypassed (internal calls), but adds latency (crypto) and key distribution complexity. Smart360 answer: Gateway validates signature and extracts claims, passes claims as signed request headers downstream. Downstream services trust the claims (not re-validating signature) but can check claim contents. Best of both.

**13. "Top K Frequent Elements — what if K changes dynamically per request? Can you still use bucket sort?"**
Pointer: Bucket sort requires knowing the value range (0 to n) in advance, and `n` depends on the input. For dynamic K across different inputs, the bucket sort approach still works — just rebuild per request. For a streaming scenario where the input is a continuous stream and K changes: maintain a `ConcurrentHashMap<T, AtomicLong>` for counts, and use a `TreeMap<Long, List<T>>` as a sorted frequency index with a `ConcurrentSkipListMap` for thread safety. Top-K then walks from the end of the TreeMap.

---

## 🌟 Extraordinary-Candidate Edge

**Go beyond the obvious in collections questions:**

When asked about `HashMap`, every candidate mentions load factor. An extraordinary candidate mentions the bit-spread formula AND explains that `capacity` must be a power of 2 to make the bitwise AND work as modulo. Then says: "In my Smart360 Data Service, we initialized our session cache `HashMap` with a capacity of `(int)(expectedSessions / 0.75) + 1` rounded up to the next power of 2 to avoid the first resize under load — this is a micro-optimization but shows you understand when initial sizing matters at scale."

**On `Comparable` vs `Comparator`, mention the `BigDecimal` gotcha** (question 6 above). Say you discovered this in a code review on a teammates' code that stored prices in a `TreeSet<BigDecimal>`. This shows you read the Java docs critically, not just tutorials.

**On the architecture exercise**, articulate FAILURE MODES unprompted: "The Notification Service failing doesn't fail a user registration because the event is fire-and-forget. The Data Service failing can return stale cached data for read-only operations. The API Gateway is our single point of failure — this is why we run 3 replicas with a rolling update strategy." Showing you think about failure as a first-class concern makes you sound like a senior engineer.

**On DSA**, when you solve a problem, proactively discuss the FOLLOW-UP before the interviewer asks: "I solved this in O(n log k) with a heap. There's also an O(n) bucket sort approach — want me to code that too?" This demonstrates you've thought deeply about the problem, not just reached for the first working solution.

**Connect DSA to production**: after solving Subarray Sum = K, say: "This prefix sum pattern is the same idea behind database cumulative sum window functions — `SUM() OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)`. When I needed running totals in the Smart360 Visualization Service, I used a PostgreSQL window function rather than computing it in Java — same math, pushed to the DB layer."

**Research the target company's tech stack before every interview.** If interviewing at a product company using Kafka + Flink, say: "In Smart360 I used Spring events for async notification — if I were building this at scale I'd use Kafka for durability and Flink for stateful stream processing on the notification analytics side." This shows ambition beyond your current scope.

---

## 📊 End-of-Week Self-Assessment

Rate each item 1–5 (1 = can barely explain, 5 = can explain + code + handle curveballs cold).

### DSA

| Problem | Your Score | Target |
|---|---|---|
| Valid Anagram (Unicode variant included) | __ /5 | 5 |
| Group Anagrams (both key strategies) | __ /5 | 5 |
| Top K Frequent (bucket sort, not just heap) | __ /5 | 5 |
| Longest Consecutive Sequence (O(n) HashSet) | __ /5 | 5 |
| Valid Palindrome (+ 680 follow-up) | __ /5 | 4 |
| Encode/Decode Strings (length-prefix) | __ /5 | 4 |
| Longest Repeating Char Replacement (maxFreq insight) | __ /5 | 5 |
| Subarray Sum = K (seed `{0:1}` explained) | __ /5 | 5 |

### Collections Framework

| Concept | Your Score | Target |
|---|---|---|
| HashMap internal structure (bit-spread, treeification, resize) | __ /5 | 5 |
| ArrayList vs LinkedList internals + CPU cache argument | __ /5 | 5 |
| ConcurrentHashMap CAS vs synchronized + null reason | __ /5 | 5 |
| Fail-fast `modCount` mechanism (including `set()` edge case) | __ /5 | 5 |
| Comparable vs Comparator + BigDecimal gotcha | __ /5 | 4 |
| When to use: `CopyOnWriteArrayList` / `LinkedHashSet` / `EnumMap` / `PriorityBlockingQueue` | __ /5 | 4 |

### System Design

| Concept | Your Score | Target |
|---|---|---|
| Latency numbers (L1 → network) from memory | __ /5 | 3 |
| Caching strategies (cache-aside, write-through, write-behind, stampede fix) | __ /5 | 4 |
| Load balancing L4 vs L7, consistent hashing use case | __ /5 | 4 |
| Smart360 5-service diagram from memory with failure modes | __ /5 | 5 |
| URL shortener design in <12 minutes | __ /5 | 3 |

### Minimum to proceed to Week 3

All DSA problems ≥ 4/5. Collections topics ≥ 4/5. System design concepts ≥ 3/5.

If any DSA problem is ≤ 3, schedule 20 min at the start of Week 3 Day 1 to re-drill it before moving to new material.

---

*Week 2 — generated 2026-06-05, schedule revised 2026-06-16. Next: Week 3 — Advanced Java (Generics, Streams deep dive, Optional, Records) + DSA: Binary Search patterns.*
