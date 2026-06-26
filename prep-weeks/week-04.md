# Week 4 — Core Build (Jul 20–25, 2026)

> Tame dynamic programming and greedy, master the resilience + at-scale layer interviewers reach for next — rate limiting, circuit breakers/bulkheads, API gateways, heaps, and the distributed-systems backbone (sharding, replication, CAP/PACELC, consistent hashing) — and tie every answer back to a real decision from Smart360, Deep Fathom, or WebX.
>
> **Full-time, heads-down study week — Mon–Sat (6 study days), Sun rest.** Three blocks per day: **Block A** (DSA, ~2.5 hr), **Block B** (core/system knowledge, ~2 hr), **Block C** (a second DSA set or deep-dive, ~1.5 hr). No applications this week — pure skill build.

---

## 🎯 Week Goal

Derive any classic 1D/2D DP recurrence (Climbing Stairs, House Robber I/II, Coin Change I/II, LIS, LCS, 0/1 knapsack, Word Break, Edit Distance, Maximal Square) from state + transition — no memorising. Solve greedy and interval problems on sight. Implement all four heap patterns and design-DS primitives cold. Explain rate-limiting algorithms, the Resilience4j circuit-breaker state machine and bulkhead, gRPC vs REST vs messaging, and the at-scale layer (sharding, leader-follower replication, CAP, PACELC, consistent hashing) at depth — and field every curveball from first principles.

---

## ✅ By Saturday you can...

- Derive and code Climbing Stairs, House Robber I/II, Coin Change, LIS, and Word Break from scratch, optimising space where `dp[i]` only depends on a couple of prior states.
- Solve 2D DP — Unique Paths, LCS, Edit Distance, Maximal Square, Longest Palindromic Substring — and the 0/1 knapsack family (Partition Equal Subset Sum, Coin Change II), explaining why the 0/1 inner loop runs right-to-left and unbounded runs left-to-right.
- Solve the greedy + interval set (Jump Game I/II, Gas Station, Merge/Insert/Non-overlapping Intervals, Meeting Rooms II) in ≤15 min each, and state which sort order each needs and why.
- Implement the four heap patterns (top-K min-heap, two-heap streaming median, merge-K, custom comparator) and design-DS (Insert/Delete/GetRandom O(1), LFU cache) from a blank editor.
- Explain token bucket vs leaky bucket vs sliding window (log + counter): complexity, burst handling, and the Redis structure each uses.
- Draw the Resilience4j circuit-breaker state machine with exact config edges, distinguish bulkhead (semaphore vs thread-pool) from circuit breaker, and name the wrap order with Retry + TimeLimiter.
- Compare REST, gRPC, and async messaging across coupling, latency, schema evolution, observability, and failure mode, and contrast API Gateway vs BFF.
- Explain range/hash/consistent-hashing sharding and their failure modes, leader-follower replication lag with three mitigations, CAP precisely (behaviour *during* a partition), PACELC, and the four consistency models — placing PostgreSQL, Redis, Cassandra, and DynamoDB correctly.
- Whiteboard Docker multi-stage builds, K8s rolling-update + probe mechanics, and the GitLab DAG/BuildKit pipeline that drove the 57% CI/CD cut.

---

## 📅 Daily Checklist

---

### Monday Jul 20 — DP 1D + Rate-Limiting Algorithms

📌 **Study today:** 1D DP — state + transition framework (LC 70, 198, 213, 139, 300, 322) · rate-limiting algorithms (token/leaky/sliding) · DP practice

**DSA (Block A, ~2.5 hr) — 1D DP Foundations:**

Read the "identify state + transition" framework before coding anything:
- *State*: what do you need to know to solve a subproblem? Usually `dp[i]` = answer for first `i` elements / target `i`.
- *Transition*: how does `dp[i]` depend on `dp[i-1]`, `dp[i-2]`, etc.? This is the hard step — draw examples.
- *Base case*: the smallest subproblem you can answer directly.
- *Direction*: top-down (memoization, recursion + HashMap/array) vs bottom-up (tabulation, loops).

Problems (in order):
1. **Climbing Stairs** (LC 70) — `dp[i] = dp[i-1] + dp[i-2]`. Code both memoization and tabulation, then reduce to O(1) space with two variables.
2. **House Robber** (LC 198) — `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`. Justify *why* you can't rob adjacent houses as a recurrence, not a rule you memorise. Target AC in 8 min.
3. **House Robber II** (LC 213) — houses in a circle. Run House Robber I on `nums[0..n-2]` and `nums[1..n-1]`, take max. *Why does splitting work?* House 0 and house n-1 can never both be chosen.
4. **Word Break** (LC 139) — `dp[i]` = can the first `i` chars be segmented? For each `j < i`, if `dp[j]` is true AND `s[j..i]` ∈ `wordDict`, then `dp[i] = true`. O(n² × avg_word_len). Work out why DFS-with-memo is equivalent.
5. **Longest Increasing Subsequence** (LC 300) — O(n²): `dp[i] = max(dp[j]+1)` for `j<i` where `nums[j]<nums[i]`. Then the O(n log n) patience-sort + binary-search solution — interviewers push for the optimal. LIS ≈ longest chain of compatible microservice versions.
6. **Coin Change** (LC 322) — `dp[amount] = min(dp[amount-coin] + 1)` per coin (unbounded knapsack). Init `dp[0]=0`, rest `INF`. Greedy fails here (coins=[1,3,4], amount=6: greedy 4+1+1=3 vs DP 3+3=2); resembles Smart360's LLM cost-routing — DP would be optimal but is too slow for real-time, which is why production used heuristic scoring.

**Core Topic (Block B, ~2 hr) — Rate-Limiting Algorithms:**

Master all four (plus the two sliding-window variants):

| Algorithm | How it works | Burst allowed? | Memory | Redis structure |
|---|---|---|---|---|
| **Token bucket** | Tokens added at a fixed rate, consumed per request | Yes (up to bucket size) | O(1) | HASH (tokens + timestamp) |
| **Leaky bucket** | Requests queue, drain at a fixed output rate | No (smoothed) | O(queue size) | LIST as queue |
| **Fixed window counter** | Count resets every window | Yes (boundary burst) | O(1) | STRING with TTL |
| **Sliding window log** | Store the timestamp of every request | No | O(requests in window) | ZSET (score = timestamp) |
| **Sliding window counter** | Interpolate between two fixed windows | Approximately no | O(1) | Two STRINGs |

- **Token bucket** (Spring Cloud Gateway `RequestRateLimiter`): bucket holds ≤ `burstCapacity` tokens; tokens added at `replenishRate`/sec; each request consumes one; empty → 429. Redis: a Lua script atomically reads `{tokens, last_refill_time}`, computes tokens to add, clamps to bucket size, decrements by cost, writes back. **Why Lua?** Redis is single-threaded per command, so the script runs as one atomic op — no race condition without locks.
- **Sliding window log** (most accurate): ZSET of timestamps; on each request remove entries older than `windowSize`, count remaining, allow if under limit. Memory-proportional to volume — not for high-throughput endpoints.
- **Sliding window counter** (practical balance): two fixed-window counters; estimate = `prev_count × (1 - elapsed_fraction) + curr_count`. O(1), within ~0.003% of true sliding window in practice.
- Recall the Smart360 API Gateway config: `redis-rate-limiter.replenishRate` and `burstCapacity` — write plausible values with justification (e.g. analytics users fire 10 queries then idle → token bucket allows the burst while enforcing sustained rate; sliding window log would reject them when idle).

**DSA (Block C, ~1.5 hr) — DP Practice:**

- Re-derive each of today's recurrences on paper in ≤2 min, no peeking. If you stall on any, that's the one to re-solve tonight.
- Compare memoization vs tabulation: memoization is easier to write and only computes needed subproblems but carries recursion-stack overhead; tabulation has no stack-overflow risk, is cache-friendly, and easier to space-optimise — prefer it for production code.
- Teach-back: explain `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` to a rubber duck — *why*, not recite.

**Self-check:**
1. Code House Robber from scratch in 8 min without notes — time yourself.
2. A client sends 10 requests in the last 0.5s of window 1 and 10 in the first 0.5s of window 2. Fixed window allows all 20 — how does sliding window counter catch it? (Weighted interpolation of the previous window's count.)

---

### Tuesday Jul 21 — DP 2D + Resilience4j

📌 **Study today:** 2D DP — Unique Paths, Edit Distance, LCS, Maximal Square, palindromes (LC 62, 72, 1143, 221, 5) · Resilience4j (circuit-breaker state machine + bulkhead + retry/time-limiter) · knapsack/subset DP (LC 416, 518)

**DSA (Block A, ~2.5 hr) — 2D DP:**

1. **Unique Paths** (LC 62) — `dp[i][j] = dp[i-1][j] + dp[i][j-1]`. Reduce to a 1D array. Also combinatorics: `C(m+n-2, m-1)` — know both.
2. **Edit Distance** (LC 72, Hard) — `dp[i][j]` = min ops to convert `word1[0..i]` to `word2[0..j]`. Insert `dp[i][j-1]+1`; delete `dp[i-1][j]+1`; replace `dp[i-1][j-1] + (chars differ)`. Base: `dp[i][0]=i`, `dp[0][j]=j`. O(mn) time, O(n) space with a rolling array. This is LCS in disguise — seeing the connection is the expert move.
3. **Longest Common Subsequence** (LC 1143) — `dp[i][j]`: if `text1[i-1]==text2[j-1]` then `dp[i-1][j-1]+1`, else `max(dp[i-1][j], dp[i][j-1])`. Reconstruct the actual subsequence by backtracking — interviewers sometimes ask.
4. **Maximal Square** (LC 221) — `dp[i][j]` = side of the largest square with bottom-right corner at (i,j) = `min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1` when `matrix[i][j]=='1'`. The side is bottlenecked by its three neighbours — prove with a 2×2. Answer area = `(max dp)²`.
5. **Longest Palindromic Substring** (LC 5) — expand-around-centre O(n²) (interview-preferred) and the `dp[i][j] = is s[i..j] a palindrome` table O(n²). Know Manacher's is O(n) conceptually (transform string, mirror property) — saying so is an extraordinary-candidate signal; don't implement it under pressure.

**Core Topic (Block B, ~2 hr) — Resilience4j (Circuit Breaker + Bulkhead + Retry/TimeLimiter):**

Circuit-breaker state machine with exact conditions:
```
CLOSED ──[failure_rate >= threshold OR slow_call_rate >= threshold]──► OPEN
OPEN ──[after waitDurationInOpenState]──► HALF_OPEN
HALF_OPEN ──[permittedNumberOfCallsInHalfOpenState calls all succeed]──► CLOSED
HALF_OPEN ──[any failure]──► OPEN
```
- `slidingWindowType`: COUNT_BASED (last N calls) or TIME_BASED (last N seconds). `minimumNumberOfCalls`: won't open until this many calls (prevents premature opening on startup). `permittedNumberOfCallsInHalfOpenState`: probe calls before deciding. Fallback method needs the same signature + a `Throwable` parameter. Annotate `slidingWindowSize`, `failureRateThreshold`, `slowCallRateThreshold`, `waitDurationInOpenState` against the state-machine edges.
- **Multi-instance concern:** Resilience4j state is in-process — two pods can have different circuit states; usually fine (eventual convergence). Global state needs a Redis-backed adapter; know the trade-off.

**Bulkhead vs circuit breaker** (never bluff this): a circuit breaker *stops* calls to a failing dependency (trips after a failure threshold, short-circuits); a bulkhead *limits* how much of your resources one dependency can consume so a slow-but-not-yet-failing dependency can't starve the whole service. Named after a ship's watertight compartments. Two flavors:
- **`Bulkhead` (semaphore)** — a counting semaphore caps *concurrent* calls; the caller's own thread runs the call. Cheap, no timeout isolation. Good default for fast synchronous dependencies.
- **`ThreadPoolBulkhead` (thread-pool)** — the call runs in a dedicated bounded pool with its own queue; the caller's thread is freed (returns a `CompletableFuture`). Costlier, but fully contains a slow dependency — isolate the slow LLM-provider calls so they can never starve threads serving fast data queries.

They're complementary: bulkhead *contains* the blast radius while a dependency degrades, the circuit breaker *cuts off* a dead one. Wire both — bulkhead first to bound concurrency, then circuit breaker — alongside **Retry** (`maxAttempts`, `waitDuration`, `exponentialBackoff`, `retryExceptions`; retrying non-idempotent POSTs is dangerous → idempotency keys) and **TimeLimiter** (wraps a `CompletableFuture`; fires first, counts the timeout as a circuit failure). Wrap order: **TimeLimiter wraps Retry wraps CircuitBreaker.** The five modules: CircuitBreaker, Retry, RateLimiter, TimeLimiter, Bulkhead.

**DSA (Block C, ~1.5 hr) — Knapsack / Subset DP:**

- **0/1 Knapsack** (classic): `dp[i][w] = max(dp[i-1][w], dp[i-1][w-wt[i]] + val[i])` if `wt[i] <= w`. Space-optimise to 1D by iterating weights *right to left* — this preserves the `i-1` row (left-to-right would read a value already updated for item `i`, i.e. unbounded behaviour).
1. **Partition Equal Subset Sum** (LC 416) — 0/1 knapsack: can we fill a subset summing to `totalSum/2`? `dp[j] |= dp[j-nums[i]]`, inner loop right-to-left. Odd total → false immediately.
2. **Coin Change II** (LC 518) — count the number of ways (unbounded knapsack). Loop order matters: outer = coins, inner = amounts → combinations; swapping → permutations. Know why.

**Self-check:**
1. Code 0/1 Knapsack bottom-up in 12 min, convert to 1D, and explain why the inner loop direction differs for 0/1 vs unbounded.
2. Draw the Resilience4j state machine from memory with the exact config parameters on each edge.

---

### Wednesday Jul 22 — Greedy + Intervals + API Gateway / gRPC

📌 **Study today:** Greedy + intervals (LC 55, 45, 134, 56, 57, 435, 253) · API gateway + gRPC vs REST vs messaging · interval problems

**DSA (Block A, ~2.5 hr) — Greedy + Intervals:**

1. **Jump Game** (LC 55) — track `maxReach`; if an index > `maxReach`, return false. O(n), O(1). Know why DP is O(n²) and strictly worse.
2. **Jump Game II** (LC 45) — track current end, furthest reachable, jumps; increment jumps when you reach `currentEnd`. O(n), O(1).
3. **Gas Station** (LC 134) — if total gas ≥ total cost, a solution exists and is unique; start tracking from the station after the point where the running sum went negative. O(n), O(1). Prove *why* you can discard everything before the reset.
4. **Merge Intervals** (LC 56) — sort by start; merge by comparing the last merged `end` vs next `start`. O(n log n).
5. **Insert Interval** (LC 57) — three phases: before, merge-overlapping (extend by max end), after. O(n), no sort (already sorted). Common FAANG variant — drill until mechanical.
6. **Non-overlapping Intervals** (LC 435) — max non-overlapping = n − (max to keep); sort by end, greedily keep the earliest end. O(n log n).
7. **Meeting Rooms II** (LC 253) — min-heap of end times / sweep line; prerequisite for many scheduling problems.

Pattern: intervals → sort by start (merge/insert) OR end (non-overlapping keep) → single pass. **DP vs greedy on intervals:** "max non-overlapping" / "min to remove" → greedy; "max *weight* of non-overlapping" (weighted job scheduling, LC 1235) → DP, because greedy can't account for value trade-offs.

**Core Topic (Block B, ~2 hr) — API Gateway + gRPC vs REST vs Messaging:**

Microservice communication across five dimensions:

| Dimension | REST | gRPC | Async Messaging |
|---|---|---|---|
| **Coupling** | Loose (URL contract) | Tight (`.proto` schema) | Loosest (message schema) |
| **Latency** | ~1ms+ (JSON parse) | ~0.3ms (binary) | Milliseconds to seconds (async) |
| **Schema evolution** | Risky (no enforcement) | Controlled (field numbers) | Risky unless schema registry |
| **Observability** | Easy (HTTP logs) | Needs gRPC interceptors | Needs correlation IDs in message |
| **Failure mode** | Synchronous → caller waits | Synchronous → caller waits | Asynchronous → dead-letter queue |

- **REST (HTTP/1.1 + JSON)**: universal tooling, human-readable, stateless, HTTP caching; chatty for many small fields, no built-in streaming, silent breaking changes. Smart360 used it gateway→services: human-debuggable, Vue consumes it directly, team familiarity.
- **gRPC (HTTP/2 + Protobuf)**: 3–10× smaller payloads, multiplexed/bidirectional streams, strong `.proto` contract + codegen; not browser-native (needs gRPC-Web + proxy), binary is harder to debug, schema evolution needs care. Use for internal high-throughput service-to-service (sub-5ms overhead).
- **Async messaging (Kafka/RabbitMQ)**: temporal decoupling, backpressure, fan-out, replay; eventual consistency, harder debugging, complex ordering (Kafka per-partition, Rabbit per-queue). Smart360's event-driven notification service after the user-management extraction.
- **API Gateway vs BFF**: Spring Cloud Gateway's predicate + filter pipeline. Concrete filters used: `AddRequestHeader`, `StripPrefix`, `RequestRateLimiter`, a custom JWT-validation `GatewayFilter`. BFF tailors a backend per client; the gateway is a shared edge.

**DSA (Block C, ~1.5 hr) — Interval Problems:**

- Re-solve the three interval problems (LC 56, 57, 435) back-to-back, no hints, timing each ≤15 min.
- Spot the sort-order decision instantly: write one sentence per problem on which key you sort by and why.
- State the greedy-vs-DP interval distinction aloud (objective decides it).

**Self-check:**
1. Explain *why* Gas Station's greedy works — not just what it does.
2. A recruiter asks "why didn't you just use gRPC everywhere?" — answer in 90 seconds referencing specific Smart360 constraints.

---

### Thursday Jul 23 — Heaps + Sharding & Replication + Design-DS

📌 **Study today:** Heaps — top-K, two-heap median, merge-K (LC 215, 347, 295, 23, 373) · system design: sharding & replication · design-DS (LC 380, 460)

**DSA (Block A, ~2.5 hr) — Heaps:**

1. **Kth Largest Element** (LC 215) — *min-heap of size k*: push each element, pop if size > k; answer = heap top. O(n log k), O(k) — interview-preferred for streaming. Also *QuickSelect*: avg O(n), worst O(n²) — the "can you do better?" follow-up. Implement heap first; mention QuickSelect when pushed.
2. **Top K Frequent Elements** (LC 347) — frequency map → min-heap of size k. O(n log k). Bucket-sort O(n) follow-up: `buckets[freq]` lists, iterate high→low. Implement heap version first.
3. **Find Median from Data Stream** (LC 295, Hard) — max-heap (lower half) + min-heap (upper half); invariant `|lower| - |upper| <= 1`. Median = top of max-heap (odd) or average of both tops (even). O(log n) insert, O(1) query. Common mistake: forgetting to rebalance after every insert — narrate the rebalancing aloud.
4. **Merge K Sorted Lists** (LC 23, Hard) — min-heap of `(value, node)` seeded with each list head; pop min, push that node's next. O(n log k) (heap bounded by k, not n — that's where log k comes from). Know the naive O(nk) and the divide-and-conquer alternative.
5. **Find K Pairs with Smallest Sums** (LC 373) — min-heap with a custom comparator; lazy expansion of candidate pairs.

Why a min-heap finds the k *largest*: the heap of size k keeps the k largest seen; its minimum sits on top, so a new element larger than the top deserves to be in the top-k → pop, push. Make this reasoning explicit.

**Core Topic (Block B, ~2 hr) — Sharding & Replication:**

Three sharding strategies, cold:
- *Range sharding*: partition by key range (IDs 0–1M → shard 1, etc.). Simple; hotspots if data isn't uniform (most active users in one range).
- *Hash sharding*: `shard = hash(key) % N`. Even distribution; resharding on node add requires rehashing all keys.
- *Consistent hashing*: nodes + keys on a ring; key → nearest node clockwise. Adding/removing a node redistributes only 1/N of keys. Used by Cassandra, DynamoDB, Redis Cluster (16384 hash slots). Virtual nodes (vnodes, ~100–200/node) even out distribution — more vnodes = better balance, higher coordination overhead. This is the GCC-level answer.

Leader-follower replication:
- Leader takes all writes; followers replicate async (common) or sync (rare, expensive). **Replication lag** = time between leader commit and follower propagation (PostgreSQL streaming: ~10ms–2s); a follower read during lag returns stale data.
- Three application strategies: (1) **read from leader for critical reads** (defeats replica purpose for those queries); (2) **read-your-writes** — track the write's WAL LSN, route the next read to the leader if the replica's LSN is behind; (3) **accept and handle staleness** — for non-critical reads (file list after upload), show optimistic UI from the POST response while the DB catches up (most pragmatic).
- *Monotonic reads*: hash a user to a consistent replica so they never read an older value after a newer one.
- Tie to work: caching S3 pre-signed URLs in Redis with a TTL is an eventual-consistency choice — a slightly stale but still-valid URL is fine; strong consistency would negate the 80% S3 reduction.

**DSA (Block C, ~1.5 hr) — Design a Data Structure:**

Design-DS rounds test whether you can compose primitives (array + hashmap + heap) under O(1)/O(log n) constraints. Narrate operation complexity as you go.
1. **Insert Delete GetRandom O(1)** (LC 380) — `ArrayList` for values + `HashMap<value, index>`. `remove`: swap target with the last element, update the moved element's map index, pop the last slot. All O(1). Common mistake: forgetting to update the swapped element's index.
2. **LFU Cache** (LC 460, Hard) — the step up from LRU (recency) to frequency: (1) `HashMap<key, Node>`; (2) `HashMap<freq, LinkedHashSet<key>>` frequency buckets (recency order within a tie); (3) a **min-freq pointer**. On hit: move the key to the `freq+1` bucket; if its old bucket was `minFreq` and now empty, increment `minFreq`. On eviction: drop the LRU key from the `minFreq` bucket, reset `minFreq=1` on the new insert. The min-freq pointer avoids scanning buckets — state that aloud.

**Self-check:**
1. Implement the min-heap top-K solution and explain the two-heap median invariant out loud, no notes.
2. Name the failure mode of range vs hash vs consistent hashing in one sentence each.

---

### Friday Jul 24 — Heaps/Mixed + CAP/PACELC + Cloud/DevOps

📌 **Study today:** Heaps/mixed + Design Twitter (LC 355, 621, 692) · CAP/PACELC + consistent hashing + consistency models · cloud/DevOps talking points (Docker multi-stage, K8s probes, CI/CD)

**DSA (Block A, ~2.5 hr) — Heaps / Mixed:**

1. **Design Twitter** (LC 355) — heap merge over follow lists. `getNewsFeed` = merge-K-sorted over the followees' tweet lists, capped at 10 — the same heap pattern as LC 23. Maintain per-user tweet lists + a follow set.
2. **Task Scheduler** (LC 621) — greedy + heap: idle time is set by the most frequent task. Math formula `max(n_tasks, (max_freq-1)*(n+1) + count_of_max_freq)`; also the max-heap + cooldown-queue simulation. Know both.
3. **Top K Frequent Words** (LC 692) — min-heap with a custom comparator `(frequency ASC, word DESC)` so the root is the "worst" of the top-k; pop when size > k. O(n log k). Clean exercise in a tie-breaking comparator under pressure.

**Core Topic (Block B, ~2 hr) — CAP / PACELC + Consistency Models:**

- **CAP — the precise statement:** during a network **partition** (P), a distributed system must choose **Consistency** (every read returns the most recent write) or **Availability** (every request gets a non-error response, possibly stale). Without a partition you can have both. The real question: *how does the system behave during a partition?* PostgreSQL (single-node/sync replication) = **CP** (isolated replica stops accepting writes rather than serve stale). Redis (default) = **AP-leaning**. Cassandra = AP by default, tunable to C via `QUORUM`/`ALL`.
- **Consistency models spectrum:** *Strong* (every read = most recent write; needs coordination — PostgreSQL `synchronous_commit=on`). *Read-your-writes* (always see your own writes; sufficient for most user-facing features). *Eventual* (replicas converge given no new writes; DNS, Redis replication, the S3 URL cache). *Causal* (reads following a causally-related write reflect it; MongoDB sessions).
- **PACELC** (the GCC-favourite extension): even with no partition (E), there's a Latency vs Consistency trade-off. DynamoDB = **PA/EL** (available during partition, low-latency eventual normally). Spanner/CockroachDB = **PC/EC** (consistent during partition, consistent-but-higher-latency normally). Mentioning PACELC distinguishes you as someone who thought beyond CAP — "you can't just say 'we're CP'; what's the latency budget for consistency in normal operation?"
- **Consistent hashing deep cut:** adding a node moves only the keys between it and its predecessor on the ring — O(n/k), no global rehash. This is why Redis Cluster and Cassandra rebalance incrementally.
- Tie to work: "Smart360 used PostgreSQL as the CP source of truth and Redis as the AP-leaning cache; pre-signed-URL TTLs were tuned to be safe inside the eventual-consistency window."

**DSA (Block C, ~1.5 hr) — Cloud/DevOps Talking Points:**

- **Docker multi-stage build:** Stage 1 (build) JDK + Maven + source → fat jar; Stage 2 (runtime) JRE only + jar. No JDK/source/Maven cache in prod → ~600 MB → ~180 MB, smaller security surface. BuildKit: `DOCKER_BUILDKIT=1`, `--cache-from type=registry,ref=...`, `RUN --mount=type=cache,target=/root/.m2` (don't bake the Maven cache into the layer).
- **K8s zero-downtime rolling update:** `kubectl set image` → new pod created (`maxSurge`) → passes **readiness** → added to Service endpoints → old pod removed → SIGTERM → Spring graceful drain → terminate. Liveness (process alive? failure → restart) vs readiness (ready for traffic? failure → removed from endpoints, no restart) vs startup probe (slow-start grace). Config: `server.shutdown=graceful`, `spring.lifecycle.timeout-per-shutdown-phase=30s`, pod `terminationGracePeriodSeconds` must *exceed* the Spring timeout.
- **GitLab DAG / BuildKit pipeline (the 57% cut):** sequential stages (23 min) → DAG with `needs:` so unit tests, integration tests, and image build run in parallel + BuildKit registry cache (70%+ layer hits) → 10 min. Name the specific mechanisms, not just "parallelism."

**Self-check:**
1. Place PostgreSQL, Redis, Cassandra, and DynamoDB on the CAP spectrum and justify each.
2. Describe a full zero-downtime K8s rolling deploy, including the Spring config, in 2 minutes.

---

### Saturday Jul 25 — Timed DP/Heap Set + At-Scale Consolidation

📌 **Study today:** Timed DP/heap set (LC 70, 322, 300, 1143, 416, 215, 295) · system-design-at-scale consolidation · weak-spot drills

**DSA (Block A, ~2.5 hr) — Timed Set (no hints):**

Real-interview conditions — think aloud, handle edge cases, state complexity. 75-min timer, fresh editor:
- LC 70 Climbing Stairs (10 min)
- LC 322 Coin Change (15 min)
- LC 300 LIS (15 min — O(n²) first, then O(n log n) if time)
- LC 1143 LCS (15 min)
- LC 416 Partition Equal Subset Sum (15 min)
- Stretch if time: LC 215 Kth Largest or LC 295 Find Median.

After the timer: review only the slow/wrong ones. Identify *why* — wrong state definition? wrong base case? off-by-one? missed rebalance? Write time/space complexity for every problem from memory.

**Core Topic (Block B, ~2 hr) — System-Design-at-Scale Consolidation:**

Consolidate the distributed-systems backbone you'll lean on for the rest of the plan. Without notes, write:
- **Sharding decision framework:** (1) most common query pattern → shard key (keep query data co-located); (2) will the key cause hotspots? (uniform distribution is a must); (3) how often do nodes join/leave? (frequent → consistent hashing; rare → hash mod N). Metadata example: shard on `owner_id` → all of a user's files on one shard → efficient "list my files." Time-series: range-shard on timestamp → efficient range scans, hot current-time shard mitigated by a randomised bucket suffix.
- **CAP/PACELC table** from memory (CP vs AP examples, PA/EL vs PC/EC).
- **Replication-lag mitigations** (read-from-leader, read-your-writes via LSN, accept-and-handle staleness).
- **Resilience4j delivery table:** circuit-breaker states + config, bulkhead (semaphore vs thread-pool), wrap order TimeLimiter→Retry→CircuitBreaker.
- **Rate-limiting table** (four algorithms + Redis structure each).

**DSA (Block C, ~1.5 hr) — Weak-Spot Drills:**

- Write 3 bullets per DP/greedy/heap pattern: when to use, the pattern, one gotcha.
- State the one-line pattern per problem: Coin Change → unbounded knapsack; LCS/Edit Distance → 2D string DP; 0/1 knapsack → right-to-left 1D; intervals → sort + single pass; top-K → min-heap of size k; median → two heaps; merge-K → heap bounded by k.
- Re-solve your two hardest problems of the week (15 min each).
- Write your top 3 weak areas and one concrete drill for each.

**Self-check:**
1. Can you derive every DP recurrence from this week on paper in ≤2 min each?
2. Can you place DynamoDB and Spanner on PACELC and justify, and answer "two pods, different circuit states" without hesitating?

---

### Sunday Jul 26 — Rest

No study blocks. Let DP, greedy, heaps, and the at-scale patterns consolidate. Optionally skim the self-check answers you got wrong — but do not start new material.

---

## 🧠 Concepts to Master This Week

### Dynamic Programming — The Universal Framework

1. **Define state**: write it as a sentence ("`dp[i]` = min coins to make amount `i`").
2. **Write the recurrence**: how `dp[i]` depends on smaller subproblems (the hard step — draw examples).
3. **Identify base cases**: `dp[0]`, `dp[1]`, empty-string/array.
4. **Choose implementation**: memoization if the state space is sparse; tabulation if you need all states.
5. **Optimise space**: if `dp[i]` only depends on `dp[i-1]` (+ maybe `dp[i-2]`), drop the full array.

| Problem | Pattern | Key insight |
|---|---|---|
| Climbing Stairs | 1D, 2 prev states | Fibonacci — each step a small decision |
| House Robber I/II | 1D / 1D × 2 passes | Rob-or-skip; circle → two linear subproblems |
| Coin Change / II | 1D unbounded | Min coins; II counts ways (outer=coins → combinations) |
| LIS | 1D O(n²) or O(n log n) | Patience sort for optimal |
| LCS / Edit Distance | 2D, two strings | Match→extend; Edit = LCS with three ops |
| 0/1 Knapsack / Partition | 2D → 1D right-to-left | Each item used at most once |
| Unique Paths | 2D grid | Sum of paths from adjacent cells |
| Maximal Square | 2D | Side = `min(3 neighbours)+1` |
| Word Break | 1D, check all splits | Is this prefix "reachable"? |

### Rate Limiting
Token bucket (Spring Cloud Gateway, Lua-atomic in Redis, allows controlled burst) · leaky bucket (smoothed) · fixed window (boundary burst) · sliding window log (ZSET, accurate, memory-heavy) · sliding window counter (O(1), ~exact via interpolation).

### Resilience4j
Circuit breaker (CLOSED→OPEN→HALF_OPEN, in-process state) · bulkhead (semaphore vs thread-pool isolation) · Retry · TimeLimiter · RateLimiter. Wrap order TimeLimiter→Retry→CircuitBreaker. Bulkhead contains the blast radius; circuit breaker cuts off a dead dependency.

### Greedy + Intervals
Jump Game → track the frontier, don't simulate. Gas Station → total ≥ cost ⇒ unique solution after the last negative-reset. Intervals → sort by start (merge/insert) or end (non-overlapping keep). Max non-overlapping → greedy; max-*weight* non-overlapping → DP.

### Heap / Priority Queue
Top-K with min-heap of size k (O(n log k)) · two-heap streaming median (O(log n) insert) · merge-K (heap bounded by k → O(n log k)) · lazy deletion / custom comparator.

### Sharding, Replication & At-Scale
Range/hash/consistent-hashing sharding (+ vnodes) · leader-follower lag with three mitigations · CAP (behaviour *during* partition) · PACELC (latency-vs-consistency without a partition) · four consistency models. Place PostgreSQL (CP), Redis (AP-leaning), Cassandra (AP-tunable), DynamoDB (PA/EL), Spanner (PC/EC).

### Cloud / DevOps
Docker multi-stage (build → runtime, ~70% smaller) + BuildKit caching · K8s rolling update + liveness/readiness/startup probes + graceful shutdown · GitLab DAG (`needs:`) + BuildKit cache = the 57% CI/CD cut.

---

## 🎤 Sample Interview Questions (incl. curveballs)

1. **Coin Change — why doesn't greedy work?** → coins=[1,3,4], amount=6: greedy 4+1+1=3, DP 3+3=2. DP explores all sub-amounts; show a small `dp` trace.
2. **Time/space of LCS and how to optimise space?** → O(m×n)/O(m×n) naive; `dp[i][j]` only uses row `i-1` → two 1D arrays, then one with a `prev` variable for `dp[i-1][j-1]`.
3. **0/1 knapsack 1D — why right-to-left?** → Right-to-left reads `dp[w-wt]` before it's overwritten for item `i` (still the `i-1` row). Left-to-right = unbounded (item reused).
4. **Redis (rate limiter) goes down — what happens?** → Spring Cloud Gateway default fail-open (requests allowed, rate limiting temporarily lost). Trade-off: internal platform → fail-open; billing/abuse-sensitive → fail-closed.
5. **Three circuit-breaker states; now 20 pods with independent state?** → Each tracks its own window; under round-robin, convergence to Open as failures propagate is usually fine. Global state needs Redis (adds latency + a failure point).
6. **Why REST over gRPC for Smart360?** → Vue consumes REST directly (gRPC-Web adds proxy), team tooling, ~KB payloads don't justify binary; gRPC for an internal high-frequency stream.
7. **Climbing Stairs = Fibonacci — is Fibonacci always DP?** → Fibonacci is one DP instance; DP is general (state + transition). Many DP problems are 2D/min-max (Edit Distance, Burst Balloons) — don't let the Fibonacci intuition mislead.
8. **Interval scheduling: DP vs greedy?** → "max non-overlapping" / "min to remove" → greedy (sort by end); "max weight non-overlapping" → DP.
9. **(Curveball) Find Median when 90% of numbers are in [0,100]?** → Counting/bucket sort with a count array + negative/large buckets → O(1) amortised vs O(log n) heap. Know when to abandon the general solution.
10. **Merge K Sorted Lists — where does log k come from?** → Heap holds ≤ k elements (one per list), each node pushed/popped once → O(n log k), not log n. Space O(k).
11. **Walk through CAP with a C-vs-A decision from your own work.** → Smart360 chose availability for the URL cache (TTL < S3 expiry; Redis-down → regenerate from S3). Strong consistency (immediate invalidation on delete) would use cache-aside + explicit invalidation events.
12. **(Curveball) "CAP is outdated — PACELC is better."** → PACELC refines, not replaces. Even without a partition there's a latency-vs-consistency trade-off; Spanner (PC/EC) pays latency for global consistency, DynamoDB (PA/EL) gives low-latency eventual. The relevant design question is often the latency budget for consistency in normal operation.
13. **Shard a 10M-user metadata DB without downtime.** → Choose shard key (`owner_id` hash); dual-write to old + new; background-migrate in batches; verify checksums/row counts; switch reads, then stop dual-writes; decommission. Dual-write → verify → cut over, never without verification.
14. **(Curveball) Pod passes liveness but fails readiness forever?** → Stays Running, removed from Service endpoints, no traffic, not restarted — correct behaviour for a lost upstream dependency (e.g. DB down). Fix the root cause; readiness passes and traffic resumes automatically.
15. **(Curveball) Implement a circuit breaker without Resilience4j?** → `AtomicReference<State>` (CLOSED/OPEN/HALF_OPEN), an atomic failure counter, an open-timestamp, and a wrapper that checks state before calling, increments on exception, and flips on threshold.

---

## 🌟 Extraordinary-Candidate Edge

1. **Tie DP to real engineering.** "Coin Change is the same class as our LLM provider routing — greedy fails on non-linear cost-latency trade-offs, but full DP would be too slow for real-time, so we used heuristic scoring. Knowing when DP is *overkill* matters as much as knowing when to use it."
2. **Rate limiting beyond the happy path.** "We chose token bucket over sliding window log because traffic was bursty — analytics users fire 10 queries then idle. Token bucket allows the burst while enforcing sustained rate; the log would reject queries even when the system was idle."
3. **Resilience4j failure modes.** "We had a Half-Open thundering-herd — two pods probed a partially-recovered downstream simultaneously, both failed, both reopened. Fix: `permittedNumberOfCallsInHalfOpenState=1` + a health-check poller."
4. **Lead with CAP in design.** "Before drawing — for two users editing the same dataset I'm choosing AP with optimistic locking + CRDT merge over distributed locks, which would be CP but hurt availability under partition."
5. **Sharding precision.** "Consistent hashing over hash-mod-N so adding a node moves 1/k of keys, not a full rehash — Redis Cluster does this with 16,384 hash slots."
6. **Replication-lag honesty.** "We tracked the write's WAL LSN and routed the subsequent read to the leader if the replica hadn't caught up — read-your-writes without forcing every read to the leader."

---

## 📊 End-of-Week Self-Assessment

Rate yourself 1–5. Anything below 3 is a mandatory revisit before Week 5.

| Skill | Target | Your Score | Notes |
|---|---|---|---|
| 1D DP framework (Climbing Stairs, House Robber I/II) | 5 | | |
| Coin Change I/II + Word Break | 4 | | |
| LIS (both O(n²) and O(n log n)) | 4 | | |
| 2D DP: Unique Paths, LCS, Edit Distance | 4 | | |
| Maximal Square + Longest Palindromic Substring | 4 | | |
| 0/1 knapsack 1D right-to-left (LC 416, 518) | 4 | | |
| Greedy: Jump Game I/II + Gas Station | 5 | | |
| Intervals: Merge/Insert/Non-overlapping + Meeting Rooms II | 5 | | |
| Heap top-K + two-heap median (LC 215, 295) | 5 | | |
| Merge-K + custom comparator (LC 23, 373, 692) | 4 | | |
| Design-DS: GetRandom O(1) + LFU (LC 380, 460) | 4 | | |
| Rate-limiting algorithms + Redis structures | 4 | | |
| Resilience4j state machine + bulkhead + wrap order | 4 | | |
| REST vs gRPC vs messaging (five dimensions) | 4 | | |
| API Gateway vs BFF + Spring Cloud Gateway filters | 4 | | |
| Sharding (range/hash/consistent) + failure modes | 5 | | |
| Leader-follower lag + 3 mitigations | 4 | | |
| CAP (precise) + place 4 systems | 5 | | |
| PACELC + place DynamoDB/Spanner | 4 | | |
| Consistency models (strong/RYW/eventual/causal) | 4 | | |
| Docker multi-stage + BuildKit details | 4 | | |
| K8s rolling update + probe mechanics | 4 | | |
| GitLab DAG + 57% CI/CD cut (specific mechanisms) | 4 | | |

**Score interpretation:**
- 4–5 on all DSA items and no system-design item below 3: ready for Week 5.
- 3 on 1–2 items: revisit in Week 5's daily review slot.
- Below 3 on any DSA item: re-drill it Monday of Week 5 before mocks — DP/heap patterns compound.

**Bonus check:** Can you tie the rate-limiting, circuit-breaker, sharding, and CAP choices to specific decisions in Smart360 / Deep Fathom / WebX? If yes on 80%+ of rows, you are an extraordinary candidate, not just a prepared one.

---

*Week 4 of 12 — next: Week 5 (Mon Jul 27 – Sat Aug 1).*
