# Week 4 — Interview Question Answers

Crisp, self-contained answers to every interview question across Week 4's six days — DP (1D/2D, knapsack), greedy + intervals, heaps, design-a-DS, and the distributed-systems backbone (rate limiting, Resilience4j, API gateway/gRPC, sharding/replication, CAP/PACELC, consistent hashing, cloud/DevOps). For deeper code, see the day files; for cross-week concepts see [interview-qa.md](../../interview-qa.md) and the numbered modules.

---

## Day 1 — DP 1D + Rate-Limiting Algorithms

1. **What two properties must a problem have for DP to apply?** Optimal substructure (the answer is composed from answers to smaller subproblems) and overlapping subproblems (the same subproblem recurs). Without overlap you have divide-and-conquer, not DP — caching buys nothing.

2. **Walk through your DP framework.** State (write it as one English sentence), transition (how `dp[i]` depends on smaller states), base case (smallest directly-answerable), direction (top-down memo vs bottom-up tabulation), then space-optimise if only the last few states are read. Gotcha: if you can't say the state sentence, you can't write the code.

3. **Memoization vs tabulation — when each?** Memo (top-down recursion + cache) is easier to write and computes only needed states but risks stack overflow on deep inputs. Tabulation (bottom-up array fill) is cache-friendly, stack-safe, and easiest to space-optimise — prefer it for production. Use memo when the recurrence is awkward to flatten into loops.

4. **Climbing Stairs is Fibonacci — is Fibonacci always DP?** Fibonacci is one instance of DP, not the definition. DP is the general state+transition method; many DP problems are 2D or min/max (Edit Distance, Burst Balloons). Gotcha: don't let the additive Fibonacci intuition make you expect every DP to be a simple sum.

5. **House Robber — justify the recurrence, don't recite it.** At house `i` you either rob it (add `nums[i]` to the best ending at `i-2`, since `i-1` is now forbidden) or skip it (carry `dp[i-1]`); take the max. The "no adjacent" rule *is* the `dp[i-2]` term. Why: robbing forces you back past the neighbour you skipped.

6. **House Robber II — why does splitting into two passes work?** In a circle house 0 and house n-1 are adjacent, so they can never both be robbed. Every valid selection excludes the first or the last; running House Robber I on `[0..n-2]` and `[1..n-1]` covers both cases. Gotcha: special-case `n==1`.

7. **Coin Change — why doesn't greedy work?** coins=[1,3,4], amount=6: greedy picks 4+1+1=3 coins; DP finds 3+3=2. A locally-large coin can force more small coins later. Why DP wins: it evaluates every sub-amount, so a locally-greedy pick can't fool it.

8. **Coin Change I vs II — what changes?** I minimises coin *count* (`min`, value DP); II *counts ways* (`+=`, sum DP). Gotcha: in II loop order matters — outer=coins, inner=amount → combinations; swapping counts permutations.

9. **LIS in O(n log n) — explain the `tails` array.** `tails[k]` is the smallest possible tail of any increasing subsequence of length `k+1`. Binary-search each number's lower bound; extend if it's largest, else replace to keep tails minimal. Gotcha: `tails` is *not* the LIS itself — only its *length* is the answer.

10. **Word Break — why is the DP equivalent to DFS with memo?** `dp[j]` answers "can I segment `s[0..j)`?" — exactly the memoized "is this start position reachable?" The bottom-up table just fills those same states iteratively. Gotcha: put the dictionary in a `HashSet` so each substring check is O(L).

11. **When is DP the *wrong* tool in production?** When it's correct but too slow for the latency budget — e.g. real-time LLM-provider cost routing, where a heuristic that's 95% as good in 1 ms beats an optimal DP in 50 ms. Knowing when to abandon optimality is senior judgment.

12. **Token bucket vs leaky bucket — when each?** Token bucket allows controlled bursts (idle clients accumulate tokens) — best for spiky human traffic. Leaky bucket smooths output to a constant rate — best when the downstream can't tolerate spikes (a payment processor with a strict TPS cap). Gotcha: leaky bucket adds queueing latency.

13. **Fixed window's boundary-burst flaw and the fix?** A client can send the full limit in the last second of one window and again in the first second of the next → 2× the limit across the boundary. Fix: sliding window counter weights the previous window's count into the current estimate, catching the straddling burst.

14. **Why a Lua script for Redis token bucket?** The refill is a read-compute-write (read tokens+timestamp, add by elapsed time, clamp, decrement, write back). As separate commands, two pods could both see "1 token left" and both succeed. Redis runs a Lua script atomically (single-threaded), making it one indivisible op — no distributed lock needed.

15. **Sliding window log vs counter — the trade-off?** Log (ZSET of timestamps) is perfectly accurate but O(requests-in-window) memory — bad at high throughput. Counter is O(1) and within ~0.003% accuracy via interpolation — the practical default (what Cloudflare-style limiters use).

16. **Your rate limiter's Redis goes down — what happens?** Spring Cloud Gateway defaults to fail-open (allow, temporarily lose limiting) — right for internal platforms where availability beats limiting. For billing/abuse endpoints switch to fail-closed (reject when unverifiable). Gotcha: state it as a deliberate trade-off, not a forgotten default.

17. **How do you decide *who* gets limited?** A `KeyResolver` keys the limit by user ID, API key, or IP. Per-user is fairest; per-IP risks punishing NAT'd corporate clients sharing one address; per-API-key suits B2B quotas.

18. **Rate-limit a multi-instance gateway correctly.** Keep the counter in shared Redis (not in-process) so all gateway pods share one view, and use the atomic Lua refill so concurrent pods can't double-spend tokens. Gotcha: an in-process counter means N pods enforce N× the intended limit.

---

## Day 2 — DP 2D + Resilience4j

1. **How do you set up the indices for a two-sequence DP?** Use a `(m+1)×(n+1)` table so row/col 0 is the empty-prefix base case, and compare `a[i-1]` to `b[j-1]` at `dp[i][j]`. Why: this makes the base cases natural and avoids off-by-one errors.

2. **Edit Distance — explain the three transitions.** Insert → `dp[i][j-1]+1`; delete → `dp[i-1][j]+1`; replace → `dp[i-1][j-1]+1` (free, i.e. `dp[i-1][j-1]`, on a match). Base rows `dp[i][0]=i` / `dp[0][j]=j` encode converting to/from the empty string.

3. **How is Edit Distance related to LCS?** It's LCS with three operations: the diagonal is the match/replace axis, the verticals/horizontals are insert/delete. Recognising the connection is the expert signal.

4. **Optimise a 2D string DP to O(n) space.** `dp[i][j]` only reads the previous row and the current row to its left, so keep two 1D arrays — or one array plus a `prev` scalar holding the `dp[i-1][j-1]` diagonal. Always offer this after the O(mn) version.

5. **Maximal Square — why `min` of three neighbours plus one?** A square of side `k+1` ending at `(i,j)` needs side-`k` squares above, left, and top-left; the smallest of those three caps the new side. Gotcha: a single missing `1` among the three limits the min, so the side can't grow past it.

6. **Longest Palindromic Substring — why expand-around-centre over the DP table?** Same O(n²) time but O(1) space instead of O(n²). There are `2n−1` centres (n odd, n−1 even). Mention Manacher's O(n) exists (mirror symmetry) but don't code it under pressure.

7. **0/1 knapsack 1D — why iterate weights right-to-left?** Going high→low, `dp[w-wt]` still holds the *previous* item's value (the `i-1` row) when read, so each item is used at most once. Left-to-right would read an already-updated value → the item gets reused (that's unbounded behaviour).

8. **Coin Change II — why does loop order change the answer?** Outer=coins, inner=amounts counts each unordered coin set once (combinations: {1,2}={2,1}); swapping to outer=amounts counts ordered sequences (permutations). The problem wants combinations, so coins go on the outside.

9. **Draw the Resilience4j circuit-breaker state machine.** CLOSED → OPEN when failure-rate *or* slow-call-rate crosses threshold (only after `minimumNumberOfCalls`); OPEN → HALF_OPEN after `waitDurationInOpenState`; HALF_OPEN → CLOSED if all probe calls succeed, → OPEN on any single failure.

10. **Bulkhead vs circuit breaker — the difference in one sentence each.** Circuit breaker *cuts off* a dependency that has already failed (trips on a failure threshold). Bulkhead *limits* the resources one dependency can consume, so a slow-but-not-yet-failing one can't starve the rest. They're complementary.

11. **Semaphore bulkhead vs thread-pool bulkhead?** Semaphore caps concurrent calls on the caller's own thread — cheap, no timeout isolation. Thread-pool runs calls in a dedicated bounded pool and frees the caller's thread (returns a `CompletableFuture`) — fully isolates a slow dependency, but costs more.

12. **What's the wrap order for the resilience decorators and why?** `TimeLimiter(Retry(CircuitBreaker(call)))`: TimeLimiter bounds total time including retries, Retry re-invokes the breaker-guarded call, CircuitBreaker sits closest so each attempt is recorded in its window.

13. **A timeout fires — does it count as a circuit-breaker failure?** Yes — the TimeLimiter cancels the future and the timeout is recorded as a failure, feeding the breaker's failure-rate window. That composition is exactly why you stack TimeLimiter and CircuitBreaker.

14. **20 pods, each with its own circuit state — is that a problem?** Usually not: under round-robin, failures spread and all pods converge to OPEN. Global state needs a Redis-backed adapter, adding latency and a shared failure point — only worth it if you truly need a single shared decision.

15. **You enabled Retry — what could go wrong?** Retrying non-idempotent operations double-executes (a retried payment POST charges twice). Restrict retries to idempotent calls, or use idempotency keys so the server deduplicates the retried request.

16. **Half-open thundering herd — what is it and how do you fix it?** Multiple pods probe a half-recovered downstream simultaneously, all fail, all re-open (flapping). Fix: `permittedNumberOfCallsInHalfOpenState=1` plus a controlled health-check poller so only one probe re-tests recovery.

17. **COUNT_BASED vs TIME_BASED sliding window — when each?** COUNT_BASED (last N calls) suits steady traffic; TIME_BASED (last N seconds) suits bursty/variable traffic where call counts swing — it reflects the recent *rate* rather than a fixed sample size.

18. **Why did Resilience4j replace Hystrix?** Hystrix is in maintenance mode (no active development). Resilience4j is lightweight, Java-8-functional, modular (pull only the decorators you need), and integrates cleanly with Spring Boot 3 and Micrometer metrics.

---

## Day 3 — Greedy + Intervals + API Gateway / gRPC vs REST vs Messaging

1. **When is greedy correct, and how do you prove it?** When the problem has the greedy-choice property and optimal substructure. Prove with an exchange argument: any optimal solution can be transformed into the greedy one without getting worse. Gotcha: greedy is seductive and often subtly wrong (Coin Change) — always justify, don't assume.

2. **Greedy vs DP — how do you decide on an interval problem?** The objective decides: "max non-overlapping" / "min removals" → greedy (sort by end); "max *weight* of non-overlapping" (weighted job scheduling, LC 1235) → DP, since greedy can't weigh a high-value long interval against several small ones.

3. **Jump Game — why is greedy O(n) and DP O(n²)?** Greedy tracks only the furthest reachable index (`maxReach`); you never need *how* you reached a cell, only the frontier. DP recomputes reachability over all prior jumps per cell.

4. **Gas Station — prove the discard step.** If the tank goes negative at `i` starting from `start`, every station in `[start..i]` also fails as a start — each had even less accumulated gas reaching `i` (it skipped the positive prefix `start` enjoyed). So jump to `i+1`. Total gas ≥ total cost guarantees the survivor completes the loop.

5. **Merge Intervals — why sort by start?** Sorting by start makes any interval overlapping the current one appear immediately after it, so a single pass merges correctly. Gotcha: use `max` for the end so a fully-contained interval doesn't shrink the merge.

6. **Non-overlapping Intervals — why sort by *end*?** Keeping the earliest-ending compatible interval leaves the most room for subsequent ones — the activity-selection exchange argument. Sorting by start would greedily keep a long early interval that blocks several short ones.

7. **Meeting Rooms II — what does the min-heap represent?** The end times of currently-active meetings; its peek is the soonest-freeing room. If that end ≤ the next meeting's start, reuse the room (poll); else allocate one. Heap size = rooms needed. The sweep-line (sort +1/−1 events, track max) is the equivalent alternative.

8. **REST vs gRPC vs messaging — name the axis each wins on.** REST: tooling/debuggability/browser-native. gRPC: latency + binary size + streaming + enforced schema for internal calls. Messaging: temporal decoupling, fan-out, replay, backpressure.

9. **Why did you choose REST over gRPC for the edge?** The browser front end can't speak gRPC without gRPC-Web + a proxy; KB-sized payloads don't justify binary; team tooling and familiarity. gRPC was reserved for internal high-frequency service-to-service streams where sub-ms latency and streaming actually matter.

10. **How does gRPC handle schema evolution safely?** The wire contract is the **field number**, not the name. Add fields with new numbers (old clients ignore them); never reuse or renumber a field. That's stronger than REST/JSON, which breaks silently on a rename.

11. **What does async messaging buy you, and what does it cost?** Buys temporal decoupling (consumer can be down), fan-out, backpressure, and replay. Costs eventual consistency, harder debugging (no synchronous stack trace), and ordering only per partition/queue. Failures land in a dead-letter queue rather than failing the caller.

12. **Kafka vs RabbitMQ?** Kafka is a distributed, replayable log (high throughput, partition-ordered, consumer groups) — best for event streaming and fan-out. RabbitMQ is a smart broker (flexible routing, per-message ack, priorities, queue-ordered) — best for task queues and complex routing.

13. **API Gateway vs BFF?** The gateway is a single shared edge (auth, routing, rate limiting, observability) for all clients. A BFF is a per-client backend that aggregates and tailors responses for one client type (web vs mobile). Gateway = shared; BFF = client-specific.

14. **What runs in a Spring Cloud Gateway route?** A predicate + filter pipeline: predicates match (`Path`, `Host`, `Method`); filters transform (`StripPrefix`, `AddRequestHeader`, `RequestRateLimiter`, a custom JWT-validation `GatewayFilter`). Auth and rate limiting happen at the edge so downstream services can trust the gateway.

15. **Where should authentication live — gateway or each service?** Validate the JWT at the gateway (reject unauthenticated traffic at the edge), then pass identity claims downstream so services don't each re-validate against the auth provider. Gotcha: services still enforce *authorization* on their own resources.

16. **(Curveball) Your message consumer is slow and the queue backs up — what do you do?** Scale consumers (more partitions / consumer-group members for Kafka), apply backpressure, move poison messages to a dead-letter queue so one bad message doesn't block the queue, and make consumers idempotent so redelivery is safe.

17. **(Curveball) How do you keep ordering with Kafka when you scale consumers?** Ordering is per-partition. Key messages by the entity that needs ordering (e.g. `userId`) so all of that entity's events land on one partition and one consumer processes them in order; parallelism comes from many keys spread across partitions.

18. **(Curveball) A REST call between two internal services times out under load — first move?** Add a timeout + circuit breaker + bulkhead (Resilience4j), then evaluate switching that hot path to gRPC (lower latency, multiplexing) or to async messaging if the caller doesn't actually need a synchronous response.

---

## Day 4 — Heaps + System Design: Sharding & Replication + Design-DS

1. **To find the k largest, why a min-heap of size k and not a max-heap?** The size-k min-heap keeps the k best seen; its root (the smallest of those) is the k-th largest, and a new element only matters if it exceeds the root. It's O(n log k), O(k) space, and works on a stream — a max-heap of all n is O(n) build + O(k log n) and needs all n in memory.

2. **Heap vs QuickSelect for Kth Largest?** Heap: O(n log k), streaming-friendly, no array mutation. QuickSelect: expected O(n) with a random pivot, worst O(n²), in-place, needs the full array. Heap for streams/external data; QuickSelect for in-memory expected-linear.

3. **Why is building a heap O(n), not O(n log n)?** Bottom-up heapify: most nodes are near the leaves with tiny sift-down depth, and the sum ∑(height·count) converges to O(n). Gotcha: inserting one-by-one *is* O(n log n) — the distinction is the building method.

4. **Explain the two-heap median invariant.** A max-heap holds the low half, a min-heap the high half; sizes differ by ≤1 and every low element ≤ every high element. Median is the larger heap's top (odd total) or the average of both tops (even). Classic bug: forgetting to rebalance after each insert.

5. **Merge K Sorted Lists — where does `log k` come from?** The heap holds at most k nodes (one frontier per list); each of the n nodes is pushed/popped once at O(log k) → O(n log k), space O(k). It's log k, not log n.

6. **Top K Frequent — when do you abandon the heap for bucket sort?** When k approaches n or the value domain is small: index buckets by frequency (≤ n), walk high→low → O(n), beating O(n log k). The heap wins when k ≪ n or it's a stream.

7. **Range vs hash vs consistent-hashing sharding — one-line failure mode each.** Range → hotspots when data/traffic isn't uniform ("newest users" on the last shard). Hash-mod-N → adding a node rehashes nearly all keys. Consistent hashing → uneven load without vnodes (and a dead node dumps onto one neighbour).

8. **Why virtual nodes in consistent hashing?** With few physical nodes the ring arcs are uneven (skewed load) and a node death dumps its whole arc onto one neighbour. ~100–200 vnodes per node smooth the distribution and spread a failed node's load across many neighbours — at the cost of more ring metadata.

9. **How do you choose a shard key?** Shard on the field your hottest query filters by (co-locate that query's data on one shard), confirm it spreads load uniformly (no hotspots), and factor in how often nodes join/leave (frequent → consistent hashing). Example: metadata on `owner_id` for single-shard "list my files."

10. **What is replication lag and when does it bite?** The window between a leader commit and a follower applying it (PostgreSQL streaming ~10 ms–2 s under load). A follower read inside that window returns stale data — the "I uploaded a file but it's not in my list" bug.

11. **Three replication-lag mitigations — which is most pragmatic?** (1) Read critical queries from the leader; (2) read-your-writes via LSN comparison; (3) accept staleness + optimistic UI (+ monotonic reads). For file-list-after-upload, (3) is most pragmatic — show the POST response while the DB catches up.

12. **Async vs sync replication trade-off?** Async: leader acks before followers apply — fast, but stale reads and possible data loss on failover. Sync: leader waits for a follower ack — no loss on failover, but every write pays the slowest follower; real systems use semi-sync (wait for one of several).

13. **Reshard a 10M-row table to more nodes without downtime.** Pick a shard key (e.g. `owner_id` hash), dual-write to old + new, background-migrate in batches, verify checksums/row counts, switch reads to new, stop dual-writes, decommission old. Never cut over without verification.

14. **LFU: what does the min-freq pointer buy you?** O(1) eviction — it points straight at the lowest non-empty frequency bucket so you never scan. It advances when a `touch` empties the min bucket and resets to 1 on every new insert.

15. **LFU eviction with a frequency tie — who goes?** The least *recently used* key at the minimum frequency. The `LinkedHashSet` per bucket preserves recency order, so its iterator's first element is the victim.

16. **GetRandom O(1) — the one step candidates miss?** After swapping the victim with the last array element, you must update the *moved* element's index in the value→index map before popping the tail; otherwise the map points at a stale slot and the next remove corrupts.

17. **(Curveball) Median when 90% of values are in [0,100].** Drop the two heaps for a counting array over [0,100] plus overflow buckets for the rest — O(1) amortised insert and median lookup. Knowing when the general O(log n) structure is overkill for a constrained domain is the answer.

18. **(Curveball) Your shard key turned out to hotspot — fix it live.** Identify the hot key/range, add a salted/randomised suffix to spread it (or split the hot shard), dual-write during migration, verify, cut over. If it's time-series, range-shard but bucket the current window across K sub-shards.

---

## Day 5 — Heaps / Design Twitter + CAP / PACELC + Consistent Hashing + Cloud/DevOps

1. **State CAP precisely.** During a network partition a system must choose Consistency (refuse / block on stale) or Availability (answer, maybe stale); with no partition you get both. The only meaningful question is behaviour *during* a partition. Gotcha: P isn't optional — networks will partition.

2. **Place PostgreSQL, Redis, Cassandra, DynamoDB, Spanner on CAP.** PostgreSQL CP (isolated replica stops serving), Redis AP-leaning (serves possibly-stale reads), Cassandra AP tunable to C via quorum, DynamoDB AP (opt-in strong reads), Spanner CP (consensus + TrueTime).

3. **What does PACELC add over CAP?** The Else branch: even with no partition there's a Latency-vs-Consistency trade-off. DynamoDB is PA/EL (low-latency eventual normally), Spanner PC/EC (pays latency for global consistency even when healthy).

4. **(Curveball) "CAP is outdated, just use PACELC."** PACELC *refines* CAP, it doesn't replace it — it adds the normal-operation latency-vs-consistency dimension on top of the partition case. The practical question is the latency budget for consistency when the network is healthy.

5. **Name the four consistency models, weakest correct one wins.** Strong/linearizable (every read = latest write), read-your-writes (you see your own writes), causal (causally-related reads reflect prior writes), eventual (converge if writes stop). Pick the weakest still-correct: balance → strong, profile photo → RYW, trending list → eventual.

6. **Why does consistent hashing let Cassandra/Redis Cluster rebalance without downtime?** Adding a node moves only the arc between it and its predecessor (O(n/k)) — no global rehash — so rebalancing is incremental and the cluster stays available throughout.

7. **How does quorum (`W+R>N`) give tunable consistency on the ring?** Each key lives on the next R nodes (replica set); if write and read quorums overlap (`W+R>N`) a read always intersects the latest acknowledged write. Smaller quorums trade consistency for availability/latency.

8. **Walk through a C-vs-A decision from your own work.** Smart360 chose AP for the S3-pre-signed-URL cache (Redis down → regenerate from S3; TTL tuned inside the URL's expiry window). Strong consistency (invalidate-on-every-change) would have negated the ~80% S3-call reduction. PostgreSQL stayed the CP source of truth.

9. **Design Twitter `getNewsFeed` — what's the heap doing?** Merge-K-sorted over followees' timelines, capped at 10 — pop the newest, advance that user's list. At scale switch to fan-out-on-write for normal users and keep fan-out-on-read for celebrities (the hybrid timeline) to bound write amplification.

10. **Task Scheduler — what sets the minimum time?** The most frequent task: `(maxFreq-1)*(n+1) + countOfMax`, unless there are enough *other* tasks to fill the idle slots, in which case it's just `tasks.length`. The rarest gaps form the skeleton the rest packs into.

11. **Top K Frequent Words — why is the comparator inverted?** The min-heap root must be the element you're most willing to evict, so the comparator is the *inverse* of the desired output order (freq ascending, word lexicographically descending on a tie); you reverse the popped sequence at the end.

12. **Why a Docker multi-stage build?** The runtime stage copies only the jar onto a JRE base — no JDK/Maven/source/`.m2` — cutting ~600 MB → ~180 MB and shrinking the attack surface. BuildKit caches the Maven `.m2` via a `--mount=type=cache` instead of baking it into a layer.

13. **Liveness vs readiness vs startup probe?** Liveness failure restarts the container; readiness failure removes it from Service endpoints (no restart); startup probe holds off the other two until a slow-booting app finishes starting.

14. **(Curveball) A pod passes liveness but fails readiness forever — what happens?** It stays Running, gets no traffic (removed from endpoints), and is *not* restarted — correct behaviour for a lost upstream (DB down). Fix the dependency; readiness recovers and traffic resumes automatically.

15. **How did you get a 57% CI/CD reduction?** GitLab `needs:` DAG ran unit tests, integration tests, and image build in parallel instead of sequential stages, plus a BuildKit registry layer cache (70%+ hits) — ~23 min → ~10 min. The mechanism is the DAG + cache hit-rate, not just "parallelism."

16. **Why must `terminationGracePeriodSeconds` exceed the Spring shutdown timeout?** SIGTERM starts the graceful drain; if K8s' grace period elapses first it SIGKILLs mid-drain, dropping in-flight requests. The pod grace must be longer than `spring.lifecycle.timeout-per-shutdown-phase`.

17. **What is Bicep and why use it over raw ARM?** A declarative Azure IaC DSL that transpiles to ARM JSON — readable, modular (parameters/modules), with `what-if` plan/diff previews and idempotent declarative applies; same plan-before-apply discipline as Terraform, plus drift detection.

18. **(Curveball) Redis (your rate limiter / cache) goes down — what happens?** Cache: the cache-miss path regenerates from the source of truth (Postgres / S3). Rate limiter: Spring Cloud Gateway defaults to fail-open (allow, lose limiting); choose fail-closed only for billing/abuse-sensitive endpoints. Name the trade-off explicitly.

---

## Day 6 — Timed DP/Heap Set + System-Design-at-Scale Consolidation

1. **Can you derive every DP recurrence from this week on paper in ≤2 min each?** That's the bar — Climbing Stairs, House Robber, Coin Change (I and II), LIS, LCS, Edit Distance, Maximal Square, 0/1 knapsack, Partition Subset, Word Break. If any one stalls, that's the exact pattern to re-drill — DP/heap patterns compound, so a gap now costs you in every later mock.

2. **0/1 knapsack 1D — why right-to-left?** Right-to-left reads `dp[w-wt]` while it still holds the `i-1`-row value, so each item is used once; left-to-right would read a value already updated for item `i`, which is unbounded (item-reused) behaviour. One-liner: 0/1 = right-to-left, unbounded = left-to-right.

3. **Coin Change — why does greedy fail?** coins=[1,3,4], amount=6: greedy 4+1+1=3 but DP 3+3=2. Greedy commits to the largest coin without exploring the global optimum; DP examines every sub-amount so it can't be fooled.

4. **Merge K Sorted Lists — where does `log k` come from?** The heap holds ≤ k frontiers, so each of n nodes is pushed/popped once at O(log k) → O(n log k), space O(k). It's log k, not log n.

5. **Place PostgreSQL, Redis, Cassandra, DynamoDB, Spanner on CAP and PACELC.** PostgreSQL CP / PC-EC, Redis AP-leaning / PA-EL, Cassandra AP-tunable / PA-EL, DynamoDB AP / PA-EL, Spanner CP / PC-EC — each with its one-line justification (TrueTime for Spanner, async replication for Redis, per-query quorum for Cassandra).

6. **(Curveball) "Two pods, different circuit-breaker states — is that a bug?"** No — Resilience4j state is in-process; under round-robin each pod's window converges as failures propagate, which is usually fine. Global state needs a Redis-backed adapter, adding latency and a shared failure point. Answer without hesitating.

7. **Three replication-lag mitigations, and which for "file list right after upload"?** Read-from-leader / read-your-writes via LSN / accept-staleness + optimistic UI. The third — render the POST response while the DB catches up — is the most pragmatic for a non-critical read.

8. **Reshard a 10M-row table to more nodes without downtime.** Pick a shard key, dual-write old+new, background-migrate in batches, verify checksums/row counts, switch reads, stop dual-writes, decommission. Verification before cut-over is non-negotiable.

9. **Which rate-limiting algorithm for bursty analytics traffic, and why?** Token bucket — it permits a burst up to bucket size while enforcing the sustained `replenishRate`; a sliding window log would reject the burst even when the system is otherwise idle.

10. **(Curveball) Pick the weakest correct consistency model for: bank balance, profile photo, trending list.** Strong (balance must be exact), read-your-writes (you must see your own new photo; others can lag), eventual (trending can converge). Choosing the weakest-correct model is the senior signal.

11. **State the Resilience4j wrap order and what each layer does.** `TimeLimiter → Retry → CircuitBreaker`: TimeLimiter caps a single call (a timeout counts as a failure), Retry re-attempts transient failures with backoff, CircuitBreaker trips after the failure-rate threshold. Bulkhead bounds concurrency around them.

12. **(Curveball) Find Median when 90% of values are in [0,100].** Drop the two heaps for a counting array over [0,100] plus overflow buckets — O(1) amortised insert and lookup. Recognising when the general structure is overkill is itself the answer.

---

*[← Week 4](README.md)*
