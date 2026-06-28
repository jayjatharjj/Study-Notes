# Week 4 · Day 6 — Sat Jul 25 — Timed DP/Heap Set + System-Design-at-Scale Consolidation

> A lighter, consolidation day under real-interview pressure. Block A is a **75-minute timed set** across the week's DP and heap patterns — fresh editor, no hints, think aloud, state complexity, handle edge cases. Block B is **system-design-at-scale consolidation**: reproduce, from memory, the sharding decision framework, the CAP/PACELC table, the replication-lag mitigations, the Resilience4j delivery table, and the rate-limiting table — the distributed-systems backbone you'll lean on for the rest of the plan. Block C is **weak-spot drills**: one-line patterns per problem, re-solve your two hardest, and name your top three weak areas with a concrete drill for each.

📌 **Study today:** Timed DP/heap set (LC 70, 322, 300, 1143, 416, 215, 295) · system-design-at-scale consolidation · weak-spot drills · ⏱ ~5 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Timed Set (no hints, interview conditions)

Set a **75-minute timer**, open a blank editor, and treat each problem like a live round: restate the problem, think aloud, name the state/transition before coding, handle edge cases, and state time/space complexity at the end. Targets below are the *budget*, not a stretch.

| Order | Problem (LC) | Budget | One-line pattern | What to narrate |
|---|---|---|---|---|
| 1 | Climbing Stairs (70) | 10 min | 1D DP, two prior states (Fibonacci) | reduce to O(1) space with two variables |
| 2 | Coin Change (322) | 15 min | 1D **unbounded** knapsack, min | `dp[a]=min(dp[a-coin]+1)`; why greedy fails |
| 3 | Longest Increasing Subsequence (300) | 15 min | 1D O(n²), then O(n log n) | patience sort + binary search for optimal |
| 4 | Longest Common Subsequence (1143) | 15 min | 2D string DP | match → `dp[i-1][j-1]+1` else max of two |
| 5 | Partition Equal Subset Sum (416) | 15 min | 0/1 knapsack, 1D **right-to-left** | odd total → false; why the inner loop reverses |
| — | Stretch if time: Kth Largest (215) or Find Median (295) | — | size-k min-heap / two heaps | only if the timer allows |

**After the timer — review only the slow or wrong ones.** For each, diagnose the *root cause*, not just the fix:
- Wrong **state definition**? (the most common and most expensive error)
- Wrong **base case** or off-by-one on initialisation?
- Wrong **loop direction** (0/1 right-to-left vs unbounded left-to-right)?
- Missed **rebalance** (median) or missed **eviction-after-size-check** (top-K)?

Then **write the time and space complexity for every problem from memory** — including the space-optimised variant. If you stalled on a recurrence, that's tonight's re-derive-on-paper drill.

### The one-line recurrence sheet (recall before you start)

```text
Climbing Stairs   dp[i] = dp[i-1] + dp[i-2]                      base dp[0]=dp[1]=1
House Robber      dp[i] = max(dp[i-1], dp[i-2] + nums[i])
Coin Change       dp[a] = min(dp[a-coin] + 1) over coins          base dp[0]=0, rest INF
Coin Change II    dp[a] += dp[a-coin]  (outer=coins → combos)
LIS (n²)          dp[i] = max(dp[j]+1) for j<i, nums[j]<nums[i]
LCS               match: dp[i-1][j-1]+1   else: max(dp[i-1][j], dp[i][j-1])
Edit Distance     min(insert dp[i][j-1], delete dp[i-1][j], replace dp[i-1][j-1]) (+1 unless equal)
Maximal Square    dp[i][j] = min(up, left, up-left) + 1   when cell=='1'
0/1 Knapsack 1D   for w from W down to wt[i]: dp[w]=max(dp[w], dp[w-wt[i]]+val[i])
Partition Subset  dp[s] |= dp[s-nums[i]]   (inner loop right-to-left)
Word Break        dp[i] = OR over j<i of (dp[j] && s[j..i] in dict)
```

```text
Top-K largest     min-heap of size k → root is k-th largest          O(n log k)
Streaming median  max-heap(low) + min-heap(high), |low|-|high| ∈ {0,1} O(log n)
Merge-K           min-heap of frontiers (one per list)               O(n log k)
```

---

## Block B — System-Design-at-Scale Consolidation

Close the notes. Reproduce each of the following **from memory** on paper or a whiteboard — this is the distributed-systems backbone for the rest of the 12-week plan, and the part interviewers test most at the senior level.

### 1. Sharding decision framework

1. **Most common query pattern → shard key.** Co-locate the data a query needs on one shard (metadata → `owner_id` → single-shard "list my files").
2. **Will the key hotspot?** Distribution must be uniform. Time-series on raw timestamp → hot current shard → add a randomised bucket suffix.
3. **How often do nodes join/leave?** Frequent → consistent hashing (move ~1/N of keys); rare/fixed → hash-mod-N (simpler).
4. **Resharding without downtime:** dual-write old+new → background-migrate in batches → verify checksums/row counts → switch reads → stop dual-write → decommission. **Never cut over without verification.**

Failure modes one-liner: **range → hotspots; hash-mod-N → full rehash on resize; consistent hashing → skew without vnodes.**

### 2. CAP / PACELC table (from memory)

| System | CAP (during partition) | PACELC (normal op) |
|---|---|---|
| PostgreSQL (sync) | CP | PC/EC |
| Redis (async) | AP-leaning | PA/EL |
| Cassandra | AP (tunable to C via quorum) | PA/EL |
| DynamoDB | AP (opt-in strong reads) | PA/EL |
| Spanner / CockroachDB | CP | PC/EC |

CAP one-liner: *"Behaviour **during a partition** — choose C or A; with no partition you get both."* PACELC adds: *"Else (no partition) → Latency vs Consistency."*

### 3. Replication-lag mitigations

1. **Read from leader** for must-be-fresh reads (defeats the replica for those).
2. **Read-your-writes via LSN** — compare the write's WAL LSN to the replica's applied LSN; route to leader if behind.
3. **Accept staleness + optimistic UI** (+ monotonic reads: pin a user to one replica) — most pragmatic for "file list right after upload."

### 4. Resilience4j delivery table

```text
CLOSED ──[failure_rate ≥ threshold OR slow_call_rate ≥ threshold]──► OPEN
OPEN ──[after waitDurationInOpenState]──► HALF_OPEN
HALF_OPEN ──[permitted probe calls all succeed]──► CLOSED
HALF_OPEN ──[any failure]──► OPEN
```
- **Bulkhead**: *semaphore* (caps concurrent calls, caller's own thread, cheap) vs *thread-pool* (dedicated bounded pool, frees the caller, contains a slow dependency fully).
- **Wrap order: TimeLimiter → Retry → CircuitBreaker.** Bulkhead *contains* the blast radius while a dependency degrades; the circuit breaker *cuts off* a dead one.
- In-process state: 20 pods each track their own window — convergence under round-robin is usually fine; global state needs a Redis adapter (adds latency + a failure point).

### 5. Rate-limiting table

| Algorithm | Burst? | Memory | Redis structure |
|---|---|---|---|
| Token bucket | yes (to bucket size) | O(1) | HASH (tokens + ts), Lua-atomic |
| Leaky bucket | no (smoothed) | O(queue) | LIST as queue |
| Fixed window counter | yes (boundary burst) | O(1) | STRING + TTL |
| Sliding window log | no | O(reqs in window) | ZSET (score = ts) |
| Sliding window counter | ~no | O(1) | two STRINGs (interpolated) |

Token bucket allows bursts (analytics users fire 10 then idle); sliding window log is most accurate but memory-heavy; sliding window counter is the practical O(1) balance.

---

## Block C — Weak-Spot Drills

### 1. One-line pattern per problem class

State each aloud until it's reflexive:

- **Coin Change** → unbounded knapsack (left-to-right inner loop).
- **0/1 Knapsack / Partition** → 1D right-to-left (each item once).
- **Coin Change II** → unbounded *count*; outer=coins → combinations, swap → permutations.
- **LCS / Edit Distance** → 2D string DP (Edit = LCS with three ops).
- **LIS** → O(n²) DP or O(n log n) patience sort.
- **Maximal Square** → side = min(3 neighbours) + 1.
- **Intervals** → sort by start (merge/insert) or end (non-overlapping keep) → single pass.
- **Greedy frontier** (Jump Game) → track maxReach, don't simulate.
- **Top-K** → min-heap of size k. **Median** → two heaps. **Merge-K** → heap bounded by k.
- **GetRandom O(1)** → array + value→index map, swap-with-last delete. **LFU** → kv map + freq buckets + min-freq pointer.

### 2. Three bullets per pattern (when / pattern / gotcha)

For each of DP, greedy, and heap, write:
- **When to reach for it** (the tell in the problem statement).
- **The mechanical pattern** (state+transition / sort+sweep / size-bounded heap).
- **One gotcha** (loop direction / sort key / rebalance or eviction step).

### 3. Re-solve the two hardest problems of the week

Pick your two slowest/wrongest from the timed set (likely LIS O(n log n), Edit Distance, or the two-heap median). **15 minutes each, fresh editor.** The second attempt should feel mechanical — if it doesn't, that pattern goes on next week's daily-review slot.

### 4. Name your top three weak areas + one drill each

Be honest and specific. Examples of the format:
- *"0/1 vs unbounded loop direction"* → drill: code Partition Equal Subset Sum and Coin Change II back-to-back, narrate why the inner loops differ.
- *"CAP placement under pressure"* → drill: from a blank page, place 5 systems on CAP **and** PACELC in under 3 minutes.
- *"Two-heap median rebalance"* → drill: implement LC 295 cold and explain all three steps aloud.

### 5. Whiteboard mock prompts (pick two, 20 min each, spoken)

Treat these like a real system-design round — clarify, sketch, name trade-offs, lead with the C-vs-A / shard-key decision *before* drawing boxes:

1. **"Design a URL-shortener that serves 100k reads/sec."** Shard the mapping table on `hash(short_code)` (uniform, single-shard lookups); read replicas + Redis cache for the hot read path (AP-leaning, eventual is fine — a short code is immutable once created); rate-limit creation with token bucket; consistent hashing on the cache tier so adding nodes moves ~1/N of keys.
2. **"Design a metadata service for a file-storage product."** Shard on `owner_id` so "list my files" is single-shard; PostgreSQL CP source of truth, Redis AP cache for pre-signed URLs (TTL inside the URL's expiry); read-your-writes via LSN so a user sees a just-uploaded file; outbox for the "file-uploaded" event.
3. **"Design a rate limiter for a multi-tenant API gateway."** Token bucket per tenant in Redis (Lua-atomic), `replenishRate`/`burstCapacity` tuned per tier; fail-open for internal traffic, fail-closed for billing-sensitive endpoints; circuit-breaker + bulkhead around the downstream so one slow tenant can't starve the rest.
4. **"Design a notification fan-out (like Design Twitter at scale)."** Fan-out-on-write into precomputed timelines for normal users, fan-out-on-read (merge-K) for celebrities — the hybrid timeline to bound write amplification; Kafka for the fan-out events, idempotent consumers.

For each, end with: *what breaks during a network partition, and what's the latency budget for consistency in normal operation?* (CAP + PACELC, spoken).

---

## 💻 Practice coding questions

1. The full 75-min timed set: LC 70, 322, 300, 1143, 416 (then 215 / 295 if time).
2. Re-derive every recurrence on the one-line sheet from memory, on paper, in ≤2 min each.
3. Re-solve your two hardest problems of the week, 15 min each, fresh editor.
4. From a blank page, reproduce the CAP/PACELC table for all five systems.
5. From a blank page, reproduce the rate-limiting table (algorithm → burst → memory → Redis structure).
6. From a blank page, draw the Resilience4j state machine with the exact config edges + the wrap order.
7. Write the sharding decision framework (3 questions + resharding sequence) in your own words.

---

## 🎤 Interview questions

1. **Can you derive every DP recurrence from this week on paper in ≤2 min each?** If any one stalls, that's the exact pattern to re-drill before Week 5 — DP/heap patterns compound, so a gap now costs you in every later mock.
2. **0/1 knapsack 1D — why right-to-left?** Right-to-left reads `dp[w-wt]` while it still holds the `i-1` value; left-to-right would read a value already updated for item `i`, which is unbounded (item reused) behaviour.
3. **Coin Change — why does greedy fail?** coins=[1,3,4], amount=6: greedy 4+1+1=3 but DP 3+3=2. Greedy commits to the largest coin without exploring the global optimum; DP examines every sub-amount.
4. **Merge K Sorted Lists — where does `log k` come from?** The heap holds ≤ k frontiers, so each of n nodes is pushed/popped once at O(log k) → O(n log k), space O(k). Not log n.
5. **Place PostgreSQL, Redis, Cassandra, DynamoDB, Spanner on CAP and PACELC.** PostgreSQL CP/PC-EC, Redis AP-leaning/PA-EL, Cassandra AP-tunable/PA-EL, DynamoDB AP/PA-EL, Spanner CP/PC-EC — with the one-line justification for each.
6. **(Curveball) "Two pods, different circuit-breaker states — is that a bug?"** No — Resilience4j state is in-process; under round-robin each pod's window converges as failures propagate, which is usually fine. Global state needs a Redis-backed adapter, adding latency and a shared failure point. Answer without hesitating.
7. **Three replication-lag mitigations, and which for "file list right after upload"?** Read-from-leader / read-your-writes via LSN / accept-staleness + optimistic UI. The third — render the POST response while the DB catches up — is the most pragmatic for a non-critical read.
8. **Reshard a 10M-row table to more nodes without downtime.** Pick a shard key, dual-write old+new, background-migrate in batches, verify checksums/row counts, switch reads, stop dual-writes, decommission. Verification before cut-over is non-negotiable.
9. **Which rate-limiting algorithm for bursty analytics traffic, and why?** Token bucket — it permits a burst up to bucket size while enforcing the sustained `replenishRate`; a sliding window log would reject the burst even when the system is otherwise idle.
10. **(Curveball) Pick the weakest correct consistency model for: bank balance, profile photo, trending list.** Strong (balance must be exact), read-your-writes (you must see your own new photo; others can lag), eventual (trending can converge). Choosing the weakest-correct model is the senior signal.
11. **State the Resilience4j wrap order and what each layer does.** TimeLimiter → Retry → CircuitBreaker: TimeLimiter caps a single call (timeout counts as a failure), Retry re-attempts transient failures with backoff, CircuitBreaker trips after the failure-rate threshold. Bulkhead bounds concurrency around them.
12. **(Curveball) Find Median when 90% of values are in [0,100].** Drop the two heaps for a counting array over [0,100] plus overflow buckets — O(1) amortised insert and lookup. Recognising when the general structure is overkill is itself the answer.

---

## ✅ Self-check

1. Did you finish the 75-min timed set within budget, stating complexity for each? List the ones you'd re-drill.
2. Can you derive every recurrence on the one-line sheet on paper in ≤2 min each?
3. Can you reproduce the CAP/PACELC table for all five systems and justify each placement?
4. Can you answer "two pods, different circuit states" and "pod fails readiness forever" without hesitating?
5. Have you written your top three weak areas with one concrete drill each for Week 5's review slot?

---

*Nav: ← [Day 5 (Fri Jul 24)](05-fri-jul-24.md) · [Week 4](README.md) · [Week 5](../week-05/) →*
