# Week 7 — Core Build (Aug 3–9, 2026)

> Theme: **Tame DP, master rate limiting + resilience, and simulate a real design interview — then own every curveball from first principles.**

---

## 🎯 Week Goal

By Friday you can walk an interviewer through any classic 1D/2D DP problem using a clean state-transition framework, explain rate-limiting algorithms and trade-offs at depth, describe circuit-breaker internals (not just "it has three states"), and tie every answer back to a specific decision you made in Smart360 or Deep Fathom. Saturday you run a timed mock system-design, record it, and score yourself against a rubric.

---

## ✅ By Sunday you can...

- Derive the DP recurrence for Climbing Stairs, House Robber I/II, Coin Change, LIS, LCS, 0/1 Knapsack, Word Break, and Unique Paths from scratch — no memorising, just state + transition.
- Explain token bucket vs leaky bucket vs sliding window counter vs sliding window log: time/space complexity, burst handling, Redis data structure used for each.
- Walk through Resilience4j circuit breaker state machine (Closed → Open → Half-Open → Closed), bulkhead, retry, and time-limiter — and say *exactly* how you wired them in Smart360's API Gateway.
- Compare REST, gRPC, and async messaging across five dimensions (coupling, latency, schema evolution, observability, failure modes) and justify which you used and why.
- Time-box a "design a data-collaboration platform" mock to 45 min, hit every rubric section, and articulate at least three non-obvious trade-offs.

---

## 📅 Daily Checklist

### Monday Aug 3 — DP Foundations: 1D Problems
📌 **Study today:** 1D DP — state + transition framework (LC 70, 198, 213, 322) · Resilience4j circuit breaker state machine

**DSA (60 min)**
- [ ] Read the "identify state + transition" framework before coding anything:
  - *State*: what do you need to know to solve a subproblem? Usually `dp[i]` = answer for first `i` elements / target `i`.
  - *Transition*: how does `dp[i]` depend on `dp[i-1]`, `dp[i-2]`, etc.?
  - *Base case*: smallest subproblem you can answer directly.
  - *Direction*: top-down (memoization, recursion + HashMap/array) vs bottom-up (tabulation, loops).
- [ ] **LC 70 — Climbing Stairs** (Easy): `dp[i] = dp[i-1] + dp[i-2]`. Code both memoization and tabulation. Reduce to O(1) space with two variables.
- [ ] **LC 198 — House Robber** (Medium): `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`. Justify why you can't rob adjacent houses as a recurrence — not a rule you memorise.
- [ ] **LC 213 — House Robber II** (Medium): Houses in a circle. Key insight: run House Robber I on `nums[0..n-2]` and `nums[1..n-1]`, take max. *Why does splitting work?* Because house 0 and house n-1 can never both be chosen.
- [ ] **LC 322 — Coin Change** (Medium): `dp[amount] = min(dp[amount - coin] + 1)` for each coin. This is *unbounded knapsack*. Initialise `dp[0]=0`, everything else `INF`. Note which part of Smart360's LLM cost routing this resembles — greedy fails here, DP is mandatory.

**Core (25 min)**
- [ ] Read Resilience4j docs section on CircuitBreaker states. Draw the state machine on paper. Annotate *exactly* what `slidingWindowSize`, `failureRateThreshold`, `slowCallRateThreshold`, and `waitDurationInOpenState` do to the state machine edges.
- [ ] Find your actual `application.yml` or recall the Resilience4j config you used in Smart360. Write it out from memory, then verify.

**Self-check**
- [ ] Can you code House Robber from scratch in 8 min without looking at notes? Time yourself.
- [ ] Can you explain to a rubber duck *why* `dp[i] = max(dp[i-1], dp[i-2] + nums[i])` — not just recite it?

---

### Tuesday Aug 4 — DP 1D continued + Resilience4j deep dive
📌 **Study today:** 1D DP — Word Break & LIS, memo vs tabulation (LC 139, 300) · Resilience4j: bulkhead, retry, time-limiter chain

**DSA (60 min)**
- [ ] **LC 139 — Word Break** (Medium): `dp[i]` = can the first `i` characters of `s` be segmented? Transition: for each `j < i`, if `dp[j]` is true AND `s[j..i]` is in `wordDict`, then `dp[i] = true`. Time: O(n² × avg_word_len). Work out why BFS/DFS-with-memo is equivalent.
- [ ] **LC 300 — Longest Increasing Subsequence** (Medium): Classic O(n²) DP: `dp[i] = max(dp[j]+1)` for all `j<i` where `nums[j]<nums[i]`. Then the O(n log n) patience-sort solution with binary search — know both, interviewers often push for the optimal. Link to real world: LIS ≈ longest chain of compatible microservice versions.
- [ ] Spend 10 min: compare memoization vs tabulation. Know when each wins:
  - Memoization: easier to write, only computes needed subproblems, but recursion stack overhead.
  - Tabulation: no stack overflow risk, cache-friendly, easier to optimise space. Prefer for production code.

**Core (30 min)**
- [ ] Resilience4j — go beyond CircuitBreaker:
  - **Bulkhead** (semaphore-based vs thread-pool-based): semaphore limits concurrent calls, thread-pool isolates threads per dependency. When would you use thread-pool bulkhead in Smart360? (Isolate slow LLM provider calls from fast data queries.)
  - **Retry**: `maxAttempts`, `waitDuration`, `exponentialBackoff`, `retryExceptions`. Critical: retrying non-idempotent POSTs is dangerous — add idempotency keys.
  - **TimeLimiter**: wraps a `CompletableFuture`-based call. Works with `@CircuitBreaker` in a chain — `TimeLimiter` fires first, then counts the timeout as a failure toward the circuit.
- [ ] Write the full chain annotation from memory: `@CircuitBreaker` + `@Retry` + `@TimeLimiter` on a Feign client method. Know the order of wrapping: TimeLimiter wraps Retry wraps CircuitBreaker.

**Self-check**
- [ ] Whiteboard LC 139 Word Break transition diagram — draw the dp array, trace through `s="leetcode"`, `wordDict=["leet","code"]`.
- [ ] Write a 2-sentence answer to: "In Smart360, how did you prevent one slow downstream service from cascading into a full outage?" (Combine circuit breaker + bulkhead in one answer.)

---

### Wednesday Aug 5 — 2D DP + Rate Limiting algorithms
📌 **Study today:** 2D DP — Unique Paths, LCS, 0/1 knapsack (LC 62, 1143, 416) · rate limiting: token/leaky bucket, sliding window

**DSA (65 min)**
- [ ] **LC 62 — Unique Paths** (Medium): `dp[i][j] = dp[i-1][j] + dp[i][j-1]`. Reduce to 1D array. This is also combinatorics: `C(m+n-2, m-1)` — know both approaches.
- [ ] **LC 1143 — Longest Common Subsequence** (Medium): Classic 2D. `dp[i][j]` = LCS of first `i` chars of `text1` and first `j` chars of `text2`. Transition: if `text1[i-1]==text2[j-1]` then `dp[i-1][j-1]+1`, else `max(dp[i-1][j], dp[i][j-1])`. Reconstruct the actual subsequence by backtracking through the dp table — interviewers sometimes ask for this.
- [ ] **0/1 Knapsack** (classic, not on LC directly but appears as LC 416, 494, 1049):
  - `dp[i][w]` = max value using first `i` items with weight budget `w`.
  - Transition: `dp[i][w] = max(dp[i-1][w], dp[i-1][w-wt[i]] + val[i])` if `wt[i] <= w`.
  - Space optimise to 1D by iterating weights *right to left*.
  - Practice LC 416 (Partition Equal Subset Sum) — it's 0/1 knapsack in disguise.

**Core (25 min)**
- [ ] Rate limiting algorithms — master all four:

  | Algorithm | How it works | Burst allowed? | Memory | Redis structure |
  |---|---|---|---|---|
  | **Token bucket** | Tokens added at fixed rate, consumed per request | Yes (up to bucket size) | O(1) | HASH (tokens + timestamp) |
  | **Leaky bucket** | Requests queue at fixed output rate | No (smoothed) | O(queue size) | LIST as queue |
  | **Fixed window counter** | Count resets every window | Yes (at window boundary) | O(1) | STRING with TTL |
  | **Sliding window log** | Store timestamp of each request | No | O(requests in window) | ZSET (score = timestamp) |
  | **Sliding window counter** | Interpolate between two fixed windows | Approximately no | O(1) | Two STRINGs |

- [ ] Know the Spring Cloud Gateway `RequestRateLimiter` filter uses the **token bucket** algorithm backed by Redis. Find your Smart360 API Gateway config and recall how you set `redis-rate-limiter.replenishRate` and `burstCapacity`. If you don't have exact values, write plausible ones with justification.

**Self-check**
- [ ] Code 0/1 Knapsack bottom-up in 12 min, then convert to 1D. Can you explain *why* the inner loop must go right-to-left for 0/1 but left-to-right for unbounded?
- [ ] Pop quiz: A client sends 10 requests in the last 0.5 s of window 1 and 10 requests in the first 0.5 s of window 2. Fixed window counter allows all 20. How does sliding window counter catch this? (Weighted interpolation of previous window's count.)

---

### Thursday Aug 6 — gRPC vs REST vs Messaging + API Gateway deep dive
📌 **Study today:** DP recurrence review + subset sum / palindrome (LC 416, 5) · REST vs gRPC vs async messaging; API Gateway vs BFF

**DSA (55 min)**
- [ ] Review all six problems from Mon–Wed: re-derive each recurrence on paper in 2 min max per problem. No peeking. If you stall on any, that's the one to re-solve tonight.
- [ ] **LC 416 — Partition Equal Subset Sum** (Medium): Reduce to 0/1 knapsack: can we fill a subset summing to `totalSum/2`? `dp[j] |= dp[j - nums[i]]`, inner loop right to left.
- [ ] **Stretch if time: LC 5 — Longest Palindromic Substring** (Medium): 2D DP or expand-around-center O(n²). Know both; Manacher's is O(n) but rarely required.

**Core (35 min)**
- [ ] Microservice communication comparison — go deep:

  **REST (HTTP/1.1 + JSON)**
  - Pros: universal tooling, human-readable, stateless, easy load balancing, caching with HTTP semantics.
  - Cons: chatty for many small fields, no streaming built-in (need SSE/WebSocket), schema looseness (breaking changes silent).
  - In Smart360: API Gateway → Data/Visualization services. Why REST? Human-debuggable, Vue.js front end consumes it directly, team familiarity.

  **gRPC (HTTP/2 + Protobuf)**
  - Pros: binary (3–10× smaller payloads), multiplexed streams, strong schema contract via `.proto`, built-in code generation, bidirectional streaming.
  - Cons: not browser-native (need gRPC-Web + proxy), harder to debug (binary), schema evolution requires care (field numbers).
  - Use when: internal high-throughput service-to-service calls (e.g., if your LLM routing proxy needed sub-5ms overhead between services).

  **Async messaging (Kafka/RabbitMQ)**
  - Pros: temporal decoupling, backpressure, fan-out, replay.
  - Cons: eventual consistency, harder debugging, ordering guarantees complex (Kafka: per-partition; Rabbit: per-queue).
  - In Smart360: event-driven notification service after user management extraction — publisher doesn't wait for consumer.

- [ ] API Gateway pattern vs BFF — revisit from `interview-qa.md` and add: how Spring Cloud Gateway's predicate + filter pipeline maps to Smart360 concerns. Concrete filters you used: `AddRequestHeader`, `StripPrefix`, `RequestRateLimiter`, JWT validation filter (custom `GatewayFilter`).

**Self-check**
- [ ] A recruiter asks: "Why didn't you just use gRPC everywhere?" Answer in 90 seconds, referencing specific Smart360 constraints.
- [ ] Write the token bucket rate-limiter config for Spring Cloud Gateway from memory.

---

### Friday Aug 7 — Integration + Curveball practice
📌 **Study today:** Timed DP set (LC 70, 322, 300, 1143, 416) · curveball follow-ups on circuit breaker, distributed txns, CQRS, JWT

**DSA (60 min)**
- [ ] Full timed set — 75 min timer, do not look up anything:
  - LC 70 Climbing Stairs (10 min)
  - LC 322 Coin Change (15 min)
  - LC 300 LIS (15 min — O(n²) first, then O(n log n) if time)
  - LC 1143 LCS (15 min)
  - LC 416 Partition Equal Subset Sum (15 min)
- [ ] After timer: review only the ones you got wrong or slow. Identify *why* (wrong state definition? wrong base case? off-by-one?).

**Core (20 min)**
- [ ] Re-read your own `interview-qa.md` answers on circuit breaker, distributed transactions, CQRS, and JWT. For each, add one "curveball follow-up" — the question an expert interviewer asks *after* your answer. Write your answer to that follow-up now. Examples:
  - After circuit breaker: "What if two pods have different circuit breaker states?" → Discuss stateless vs centralised (Redis-backed) circuit breaker state; Resilience4j is in-process, so each pod has independent state — this is usually fine, eventual convergence.
  - After JWT: "How do you handle token refresh race conditions in a distributed system?" → Single-use refresh tokens + refresh token rotation + Redis lock on the refresh operation.

**Self-check**
- [ ] Teach-back: explain DP memoization vs tabulation to an imaginary junior dev in 3 min. Record audio on your phone. Play it back — did you use "um" excessively? Was the explanation logically ordered?
- [ ] Can you name the five Resilience4j modules and what each solves? (CircuitBreaker, Retry, RateLimiter, TimeLimiter, Bulkhead.)

---

### Saturday Aug 8 — Mock System Design: Data-Collaboration Platform
📌 **Study today:** Timed mock HLD — data-collaboration platform (CRDT/OT, RLS, CQRS, scale) self-scored on rubric · LIS O(n log n) review

**Block 1 (45 min) — Timed mock, record yourself**

Set a 45-min timer. Open a blank doc. Record yourself speaking your design aloud (phone or laptop mic). Follow this structure:

1. **Requirements (5 min)**
   - Clarify: How many users? (Say: 10K concurrent, 1M registered.) Real-time collaboration or async? (Real-time, like Google Sheets.) Data types? (Tabular + unstructured docs.) Permissions model? (RBAC + row-level.)
   - Functional: CRUD on datasets, real-time co-editing, version history, sharing/permissions, LLM-powered analysis (ties to Smart360).
   - Non-functional: 99.9% uptime, <200ms API latency p99, data durability, GDPR compliance.

2. **API Design (5 min)**
   - `POST /datasets` → `{datasetId}`
   - `GET /datasets/{id}/records?filter=...&page=...`
   - `PATCH /datasets/{id}/records/{recordId}` (optimistic locking with `version` field)
   - `WebSocket /collaborate/{datasetId}` → real-time ops (CRDT or OT)
   - `POST /datasets/{id}/analysis` → `{jobId}` (async, 202 Accepted — same pattern as your Smart360 LLM jobs)

3. **Data Model (8 min)**
   - PostgreSQL: `datasets`, `records` (JSONB for flexible schema), `versions` (append-only for history), `permissions` (RBAC — subject, object, action).
   - Row-Level Security on `records` — same pattern as Deep Fathom's 50-table tenant isolation.
   - Redis: presence map (who is editing), pub/sub for real-time ops broadcast, rate-limiter state.
   - S3/Blob: binary attachments, dataset exports.

4. **High-Level Architecture (10 min)**
   - Client → CDN → API Gateway (rate limiting, JWT auth — mirrors Smart360 API Gateway microservice)
   - Gateway routes to: Collaboration Service (WebSocket, handles CRDTs), Data Service (CRUD + queries), Analysis Service (async LLM jobs), Notification Service (event-driven).
   - Kafka: Collaboration Service publishes `record.changed` events → Notification Service, Audit Service.
   - Separate read replica for analytics queries (CQRS read side).

5. **Scale + Bottlenecks (10 min)**
   - Real-time: WebSocket sticky sessions via consistent hashing or shared Redis pub/sub (fan-out). At 10K concurrent users, a single WebSocket server handles ~50K connections — need horizontal scale.
   - Write hot spot: if one dataset has 1K concurrent editors, all writes hit the same partition. Mitigate with CRDT merge (conflict-free) + optimistic locking at DB.
   - Read scale: read replicas + caching with cache-aside (Redis TTL 60s for dataset metadata).
   - LLM analysis: queue-backed workers (same architecture as Smart360's 2–20 min jobs), autoscale workers on queue depth.

6. **Trade-offs (7 min)**
   - CRDT vs OT for collaboration: CRDTs (like Yjs) are peer-to-peer, no server required for merge — simpler ops, but higher memory per document. OT requires server to serialize operations — simpler mental model for tabular data.
   - PostgreSQL JSONB vs dedicated schema: JSONB enables flexible columns without migrations, but loses type safety and some index performance. Accept the trade-off for MVP, migrate to typed columns for frequently queried fields.
   - Separate collaboration service vs monolithic: Separate allows independent scaling of WebSocket tier (RAM-intensive) from CRUD tier (CPU/DB-intensive). Adds network hop.

**Block 2 (45 min) — Self-critique and rebuild**
- [ ] Play back your recording. Score yourself on the rubric below.
- [ ] For each rubric item you scored < 3, write a 1-paragraph improved answer in your notes.
- [ ] Re-draw the architecture diagram from scratch without the recording. Is it the same? What did you forget?

**Rubric (score each 1–5)**
| Area | What 5 looks like |
|---|---|
| Requirements elicitation | Clarified scale, CAP choice, real-time vs async before drawing anything |
| API design | RESTful, versioned, async pattern for long ops, clear request/response |
| Data model | Tables + schema + indexes named; RLS or tenant isolation addressed |
| Architecture clarity | Named services, named message queues, named DBs — no hand-waving |
| Scale reasoning | Quantified: "X req/s needs Y replicas because Z" |
| Trade-offs | At least 3 genuine trade-offs with *why you'd choose each side* |
| Tie to experience | Referenced Smart360 / Deep Fathom at least twice naturally |

**DSA (30 min)**
- [ ] Revisit any problem from the week where you still feel shaky. Re-solve on paper.
- [ ] Read editorial for LC 300 LIS O(n log n) solution if you haven't fully internalized the patience-sort intuition.

---

### Sunday Aug 9 — Consolidation + Weak-spot drills
📌 **Study today:** Edit Distance & Coin Change II (LC 72, 518, 97) · interview-ready rate-limiting walkthrough; preview Week 8 (Kafka, Saga, K8s)

**DSA (60 min)**
- [ ] **New problem: LC 72 — Edit Distance** (Hard): `dp[i][j]` = min edits to convert `word1[0..i]` to `word2[0..j]`. Three operations → three transitions. This is LCS in disguise — both are 2D string DP. Seeing the connection is the expert move.
- [ ] **New problem: LC 518 — Coin Change II** (Medium): Count number of ways, not min coins. Unbounded knapsack variant. Note: order of loops (outer = coins, inner = amounts) gives combinations; swapping gives permutations. Know why.
- [ ] Optional stretch: LC 97 — Interleaving String (Medium) — pure 2D DP, solidifies the pattern.

**Core (60 min)**
- [ ] Write a complete, interview-ready answer to: "Walk me through how you designed the rate-limiting in Smart360's API Gateway." Cover: why rate limiting (protect downstream, fair usage), which algorithm (token bucket), Redis backing (single atomic Lua script for atomicity), config (`replenishRate`, `burstCapacity`), how to handle Redis failure gracefully (fail-open vs fail-closed — know the trade-off), and how you'd extend to per-user limits vs per-IP limits.
- [ ] Review your mock design recording one more time. Update your architecture notes with anything you'd say differently.
- [ ] Preview Week 8 topics (spend 10 min only): Kafka internals, Saga pattern implementation, and advanced Kubernetes (HPA, resource limits, pod disruption budgets) — so Monday isn't a cold start.

**Self-check**
- [ ] Mock oral: pick any three questions from the Sample Interview Questions below. Answer each aloud, full STAR or technical walkthrough, without notes. Time yourself — aim for 90–120 sec per technical Q, 2 min per behavioral.

---

## 🧠 Concepts to Master This Week

### Dynamic Programming

**The universal framework — apply this before writing a single line of code:**
1. **Define state**: what information uniquely identifies a subproblem? Write it as a sentence: "`dp[i]` = the minimum number of coins needed to make amount `i`."
2. **Write the recurrence**: how does `dp[i]` depend on smaller subproblems? This is the hard step — draw examples.
3. **Identify base cases**: `dp[0]`, `dp[1]`, or empty-string/empty-array cases.
4. **Choose implementation**: memoization (top-down) if the state space is sparse; tabulation (bottom-up) if you need all states anyway.
5. **Optimise space**: if `dp[i]` only depends on `dp[i-1]` (and maybe `dp[i-2]`), you don't need the full array.

**Problem → Pattern mapping:**
| Problem | Pattern | Key insight |
|---|---|---|
| Climbing Stairs | 1D, 2 prev states | Fibonacci — each step is a small decision |
| House Robber I | 1D, 2 prev states | At each house: rob (skip prev) or skip |
| House Robber II | 1D × 2 passes | Circle → two linear subproblems |
| Coin Change | 1D, unbounded | Inner loop: try every coin denomination |
| Coin Change II | 1D, unbounded | Outer = coins (avoid permutation counting) |
| LIS | 1D, O(n²) or O(n log n) | Patience sort for optimal |
| LCS | 2D, two strings | Match → extend; mismatch → take better of skip either |
| 0/1 Knapsack | 2D → 1D, right-to-left | Each item used at most once |
| Unique Paths | 2D grid | Only right + down → sum of paths from adjacent cells |
| Word Break | 1D, check all splits | Like knapsack: is this prefix "reachable"? |
| Edit Distance | 2D, three ops | Insert/delete/replace → three transitions |

### Rate Limiting

**Token Bucket** (used in Spring Cloud Gateway):
- A bucket holds at most `burstCapacity` tokens.
- Tokens are added at `replenishRate` per second.
- Each request consumes one token (or more for expensive endpoints).
- If bucket empty: request rejected (429 Too Many Requests).
- Redis implementation: a Lua script atomically reads `{tokens, last_refill_time}`, computes tokens to add, clamps to bucket size, decrements by request cost, writes back.
- **Why Lua?** Redis is single-threaded per command; a Lua script runs as a single atomic operation — no race condition without locks.

**Sliding Window Log** (most accurate):
- Store timestamp of every request in a Redis ZSET (score = timestamp).
- On each request: remove timestamps older than `windowSize`, count remaining, if under limit add current timestamp and allow.
- Accurate but memory-proportional to request volume — not suitable for high-throughput endpoints.

**Sliding Window Counter** (practical balance):
- Keep two fixed-window counters (current window + previous).
- Estimate count = `prev_count × (1 - elapsed_fraction) + curr_count`.
- O(1) memory, approximate (within ~0.003% of true sliding window in practice).

### Circuit Breaker Internals (Resilience4j)

State machine with exact conditions:

```
CLOSED ──[failure_rate >= threshold OR slow_call_rate >= threshold]──► OPEN
OPEN ──[after waitDurationInOpenState]──► HALF_OPEN
HALF_OPEN ──[permittedNumberOfCallsInHalfOpenState calls all succeed]──► CLOSED
HALF_OPEN ──[any failure]──► OPEN
```

- `slidingWindowType`: COUNT_BASED (last N calls) or TIME_BASED (last N seconds).
- `minimumNumberOfCalls`: circuit won't open until at least this many calls have been made (prevents premature opening on startup).
- `permittedNumberOfCallsInHalfOpenState`: probe calls before deciding.
- Fallback method must have the same signature + a `Throwable` parameter.

**Multi-instance concern:** Resilience4j state is in-process. Two pods can have different circuit states. This is usually acceptable — eventually both will open/close based on real traffic. If you need global state, use a Redis-backed solution (e.g., Sentinel or a custom adapter). Know this trade-off.

### Bulkhead (Resilience4j)

The plan keeps pairing "bulkhead" with circuit breaker — here's the difference, so you never bluff it. **A circuit breaker stops calls to a failing dependency** (it trips after a failure threshold and short-circuits). **A bulkhead limits how much of your resources any one dependency can consume** — so a slow-but-not-yet-failing dependency can't tie up every thread and take the whole service down with it. Named after a ship's watertight compartments: one floods, the rest stay dry. The classic failure they prevent is *resource exhaustion / cascading failure* — one sluggish downstream (say a 5s LLM provider) accumulates in-flight calls until the shared thread pool is starved and *unrelated* fast endpoints (a quick Postgres read) start timing out too.

Resilience4j gives two flavors:
- **`Bulkhead` (semaphore isolation)** — a counting semaphore caps the number of *concurrent* calls; the caller's own thread executes the call. Cheap (no extra threads, no context switch), but no timeout isolation — the calling thread blocks until the call returns. Good default for fast, mostly-synchronous dependencies.
- **`ThreadPoolBulkhead` (thread-pool isolation)** — the call runs in a *dedicated, bounded thread pool* with its own queue. The caller's thread is freed immediately (returns a `CompletableFuture`). Costlier, but it fully contains a slow dependency to its own pool and lets you set a queue/timeout per dependency. Use it for the genuinely slow/risky calls — e.g., isolate the slow LLM-provider calls into their own pool so they can never starve the threads serving fast data queries.

**Bulkhead vs / with circuit breaker:** they're complementary, not either/or. Bulkhead *contains* the blast radius while the dependency is degrading (caps concurrency / isolates threads); the circuit breaker *stops* hammering it once it's clearly failing (trips open). Best practice is to wire both — bulkhead first to bound concurrency, circuit breaker to cut off a dead dependency — alongside `Retry` and `TimeLimiter`. This is the precise, senior version of the Tuesday self-check answer ("how did you stop one slow downstream from cascading into a full outage?"): *bulkhead isolates it, circuit breaker cuts it off.*

### gRPC vs REST vs Async Messaging (five dimensions)

| Dimension | REST | gRPC | Async Messaging |
|---|---|---|---|
| **Coupling** | Loose (URL contract) | Tight (`.proto` schema) | Loosest (only message schema) |
| **Latency** | ~1ms+ (JSON parse) | ~0.3ms (binary) | Milliseconds to seconds (async) |
| **Schema evolution** | Risky (no enforcement) | Controlled (field numbers) | Risky unless schema registry |
| **Observability** | Easy (HTTP logs) | Needs gRPC interceptors | Needs correlation IDs in message |
| **Failure mode** | Synchronous → caller waits | Synchronous → caller waits | Asynchronous → dead-letter queue |

---

## 🎤 Sample Interview Questions (incl. curveballs) — 8–15 Qs + pointers

**1. "Walk me through the DP solution for Coin Change. Why doesn't greedy work?"**
> Pointer: Greedy (always pick largest coin) fails when a smaller coin combination sums to less (e.g., coins=[1,3,4], amount=6: greedy picks 4+1+1=3 coins; DP finds 3+3=2 coins). DP guarantees optimal by exploring all sub-amounts. Show `dp` array trace for a small example.

**2. "What's the time and space complexity of LCS, and how would you optimise space?"**
> Pointer: O(m×n) time, O(m×n) space naively. Observe that `dp[i][j]` only uses row `i-1` → reduce to two 1D arrays of size `n`. Only need one array if you process carefully — `prev` variable tracks `dp[i-1][j-1]`.

**3. "In Smart360, you used Spring Cloud Gateway for rate limiting. A Redis instance goes down. What happens?"**
> Pointer: Spring Cloud Gateway's `RequestRateLimiter` filter has a `denyEmptyKey` setting and Redis failure behaviour. Default: if Redis is unavailable, requests are allowed (fail-open). This prevents a Redis outage from taking down your entire API, but you temporarily lose rate limiting. Explicitly reason about the trade-off: for an internal platform, fail-open is safer than dropping all traffic; for a billing/abuse-sensitive API, fail-closed might be necessary.

**4. "Explain the three Resilience4j states. Now: what if you have 20 pods each with independent circuit breaker state?"**
> Pointer: Each pod tracks its own failure window. Pod A might trip its circuit while Pod B hasn't. Under round-robin load balancing, roughly half of requests still go to the protected service through Pod B. This is usually fine — eventually all pods converge to Open as failures propagate. True global circuit state requires a centralised store (Redis). Trade-off: centralised adds latency and a new failure point.

**5. "Why did you choose REST over gRPC for Smart360's inter-service communication?"**
> Pointer: Browser/Vue.js clients consume the API directly, gRPC-Web adds proxy complexity. Team was proficient in REST tooling. JSON payloads are human-debuggable during development. The payload sizes (~KB per response) didn't justify binary encoding. If Smart360 had an internal high-frequency data-streaming service (e.g., real-time sensor ingestion), gRPC would be the right call.

**6. "Curveball: Your rate limiter uses Redis. How do you rate-limit without Redis?"**
> Pointer: Options: (1) local in-memory token bucket per pod (no cross-pod coordination — different pods have separate counters, so a user can get N × numPods requests through; acceptable for low-security cases). (2) Sticky load balancing so one pod always handles a given client. (3) A lightweight coordination layer (e.g., a shared Hazelcast distributed map). Know the trade-offs of each: local is fast/simple but inaccurate at scale, sticky breaks stateless design, Hazelcast adds a dependency.

**7. "Curveball: How does LCS relate to Edit Distance?"**
> Pointer: They share the same 2D DP structure over two strings. In LCS, matching characters `extend` the sequence. In Edit Distance, mismatching characters choose the cheapest of three operations. A deletions-only edit distance equals `m + n - 2 × LCS(s1, s2)`. This shows they solve related problems from different angles — demonstrates conceptual depth.

**8. "Design the rate limiter for Smart360's API Gateway at 1M requests/day from 10K users."**
> Pointer: 1M/day ≈ 11.5 req/s average, but peak might be 10× = 115 req/s. Per-user limit: 10 req/s burst, 5 req/s sustained (token bucket: `replenishRate=5, burstCapacity=10`). Redis single-node handles >100K ops/s — fine. Key by `userId` extracted from JWT claim (not IP, since users may share IPs behind corporate NAT). Handle Redis failure: fail-open with a circuit breaker around the rate limiter call itself. Mention Spring Cloud Gateway's built-in `redis-rate-limiter` configuration.

**9. "Curveball: Your circuit breaker is in Open state, but the downstream service has already recovered. How does your system find out?"**
> Pointer: `waitDurationInOpenState` expires → circuit transitions to Half-Open → sends `permittedNumberOfCallsInHalfOpenState` probe requests. If probes succeed, closes. This is the only way Resilience4j "probes" recovery — it doesn't actively check health. You could add a health-check-based mechanism: a background thread pings `/actuator/health` of the downstream and calls `circuitBreaker.transitionToHalfOpenState()` programmatically if healthy. Know the API: `CircuitBreaker#transitionToHalfOpenState()`.

**10. "Walk me through the 0/1 Knapsack space optimisation. Why must the inner loop go right to left?"**
> Pointer: In 2D DP, `dp[i][w] = max(dp[i-1][w], dp[i-1][w-wt])`. When you collapse to 1D, `dp[w]` represents `dp[i-1][w]` before the update. Processing right-to-left ensures you read `dp[w-wt]` before it's overwritten for item `i` — effectively still reading from the `i-1` row. Left-to-right would mean `dp[w-wt]` already incorporates item `i` (unbounded knapsack behaviour — each item usable multiple times).

**11. "In your mock design, how would you handle a user editing a cell that another user deleted simultaneously?"**
> Pointer: CRDT (Conflict-free Replicated Data Type) resolves this without server coordination. For tabular data, a common approach: tombstone the deleted cell and apply the edit; last-writer-wins at the field level with a vector clock. Alternative: OT (Operational Transform) — server serializes concurrent ops into a consistent order. CRDT is simpler to scale (no central sequencer), but state grows with history (needs GC). This shows familiarity with real-time collaboration systems beyond the surface level.

**12. "How would you add observability to your rate limiter to catch abuse patterns?"**
> Pointer: Micrometer metrics from Spring Cloud Gateway emit `spring.cloud.gateway.requests` with `outcome=FORWARD` vs `outcome=CLIENT_ERROR` (429). Add a custom metric tag for `userId`. Push to Prometheus + Grafana. Alert on: any single user exceeding 80% of their rate limit consistently (potential bot). Export to ELK for forensic analysis. Mention the Actuator `/actuator/metrics` endpoint for spot-checking.

**13. "Curveball: Climbing Stairs has the same recurrence as Fibonacci. Is it always safe to assume Fibonacci = DP?"**
> Pointer: Fibonacci is a specific DP problem, but not all DP is Fibonacci. The DP framework is general — state + transition. Fibonacci happens to be 1D with a 2-back recurrence. Many DP problems are 2D, have non-trivial state definitions, or require min/max rather than sum. The Fibonacci intuition is a useful warmup but can mislead you into thinking DP problems are always simple 1D recurrences — they're not (e.g., Edit Distance, Burst Balloons, Matrix Chain Multiplication are much more complex).

**14. "You mentioned async messaging for the notification service. How do you ensure exactly-once delivery?"**
> Pointer: Exactly-once is expensive. Kafka provides at-least-once by default (idempotent producer gives exactly-once on the producer side with `enable.idempotence=true`). For consumer exactly-once, you need transactional consumption (`isolation.level=read_committed`) AND idempotent consumers (check if event was already processed using a `processed_event_ids` table). In practice, design consumers to be idempotent — handle duplicate events gracefully — rather than fighting for exactly-once at the infrastructure level.

**15. "Curveball: If you had to implement a circuit breaker without Resilience4j, how would you do it?"**
> Pointer: A minimal circuit breaker needs: (1) an atomic counter of recent failures (ConcurrentHashMap or AtomicInteger), (2) a timestamp of when the circuit opened, (3) state enum (CLOSED, OPEN, HALF_OPEN), (4) a wrapper method that checks state before calling the real method, increments failure count on exception, and flips state based on thresholds. In multi-threaded Java, use `AtomicReference<State>` for lock-free state transitions. This demonstrates you understand the pattern, not just the library.

---

## 🌟 Extraordinary-Candidate Edge

Most candidates know the *what*. You'll stand out with the *why* and the *I built this*.

**1. Tie DP to real engineering decisions.** When asked about Coin Change, mention: "This is the same class of problem as our LLM provider routing in Smart360 — greedy (always pick cheapest provider) fails when provider combinations have non-linear cost-latency trade-offs. DP-style exhaustive evaluation would be prohibitively slow for real-time routing, which is why we used heuristic scoring instead. Knowing when DP is overkill is as important as knowing when to use it."

**2. Rate limiting: go beyond the happy path.** Mention: "In Smart360, we specifically chose token bucket over sliding window log because our traffic was bursty — analytics users fire 10 queries in 5 seconds then idle. Token bucket allows that burst while still enforcing sustained rate. Sliding window log would reject those queries even when the system was idle."

**3. Resilience4j: show you've read the failure modes.** Most candidates parrot the three states. Distinguish yourself: "We had an incident where our circuit was in Half-Open and two pods simultaneously sent probe requests to the downstream. Both probes failed (downstream was partially recovered), so both pods went back to Open. The fix was `permittedNumberOfCallsInHalfOpenState=1` combined with a health check poller to avoid thundering-herd on the downstream during recovery."

**4. System design: lead with CAP, not features.** Most candidates jump to boxes and arrows. You start with: "Before I draw anything — for a collaboration platform where two users are editing the same dataset simultaneously, I'm choosing AP (availability + partition tolerance) over strong consistency. I'll use optimistic locking with CRDT merge for conflict resolution rather than distributed locks, which would make the system CP but hurt availability under network partitions." This one move signals senior-level thinking.

**5. DP: know when NOT to use it.** "DP requires overlapping subproblems. If subproblems don't overlap, memoization just adds overhead — use plain recursion or greedy. I check for overlapping subproblems first by asking: 'Can I reach the same state from multiple paths?' If yes, DP. If no, consider greedy or divide-and-conquer."

**6. Observe your own system design mock objectively.** Most candidates never record themselves. You did. You'll catch that you said "and then we just..." (vague), "it would be fast" (unquantified), or skipped the trade-offs section entirely (the most-weighted section in senior interviews). Correcting these in a week of deliberate practice is a genuine edge over candidates who've been "preparing" for months by reading articles.

---

## 📊 End-of-Week Self-Assessment

Fill this out on Sunday evening before closing your laptop.

**DSA (honest scores 1–5)**
| Problem | Can derive recurrence from scratch | Coded correctly first try | Can explain the transition intuitively |
|---|---|---|---|
| LC 70 Climbing Stairs | | | |
| LC 198/213 House Robber I/II | | | |
| LC 322 Coin Change | | | |
| LC 300 LIS (both solutions) | | | |
| LC 1143 LCS | | | |
| LC 62 Unique Paths | | | |
| LC 139 Word Break | | | |
| LC 416 Partition Equal Subset Sum | | | |
| LC 72 Edit Distance | | | |
| LC 518 Coin Change II | | | |

**System Design**
| Criterion | Score (1–5) | What I'd improve |
|---|---|---|
| Mock design: requirements elicitation | | |
| Mock design: API design | | |
| Mock design: data model | | |
| Mock design: architecture clarity | | |
| Mock design: scale reasoning | | |
| Mock design: trade-offs articulation | | |
| Mock design: tied to own experience | | |

**Core Concepts**
- [ ] I can explain the four rate-limiting algorithms and name the Redis data structure each uses.
- [ ] I can draw the Resilience4j state machine from memory with the exact config parameters.
- [ ] I can compare REST vs gRPC vs messaging across coupling, latency, schema evolution, observability, and failure mode.
- [ ] I can answer the "two pods, different circuit states" curveball without hesitating.
- [ ] I can answer the "Redis goes down, what happens to rate limiting" curveball with a concrete trade-off.

**What to carry into Week 8**
Write 1–3 sentences here on Sunday:
> _(Your notes here — e.g., "LCS reconstruction still slow — practice once more. Rate limiting answers are solid. Mock design: I skipped CAP analysis entirely, must lead with that next time.")_

---

*Week 7 of the 2026 interview prep plan. Preceding material in `/home/jay/jj/Study-Notes/interview-qa.md`.*
