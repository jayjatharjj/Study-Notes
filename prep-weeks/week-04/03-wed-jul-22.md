# Week 4 · Day 3 — Wed Jul 22 — Greedy + Intervals + API Gateway / gRPC vs REST vs Messaging

> Greedy is the inverse of DP: instead of exploring every choice, you make the **locally best** move and *prove* it's globally optimal. The hard part isn't the code — it's the one-sentence justification interviewers demand ("why does this greedy choice never need undoing?"). Intervals are the highest-frequency greedy family: the entire genre reduces to **sort by the right key, then a single pass**. On the systems side, microservice communication — REST vs gRPC vs async messaging, and the API Gateway in front of it all — is the "draw the boxes and arrows" question that opens most system-design rounds. Score it by naming the trade-off axis, not the buzzword.

📌 **Study today:** greedy + intervals (LC 55, 45, 134, 56, 57, 435, 253) · API Gateway + gRPC vs REST vs async messaging (five dimensions) · interval re-solve drills · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Greedy + Intervals (~2.5 hr)

### Theory deep-dive — when is greedy correct?

Greedy makes the locally optimal choice at each step and never reconsiders. It's correct only when the problem has the **greedy-choice property** (a globally optimal solution can be reached by greedy local choices) and **optimal substructure**. The danger: greedy is *seductive* — it often looks right and is subtly wrong (Coin Change yesterday). So the interview ritual is: state the greedy choice, then **prove** it with an exchange argument ("any optimal solution can be transformed into the greedy one without getting worse").

**Greedy vs DP — the deciding question:** does a locally optimal choice force a globally optimal one?
- **Jump Game / intervals "max count" / "min removals"** → greedy works (the local frontier/earliest-end choice is provably safe).
- **"Max *weight* of non-overlapping intervals"** (weighted job scheduling, LC 1235) → **DP**, because greedy can't weigh a high-value long interval against several small ones.

### Worked example — LC 55 Jump Game (frontier, don't simulate)

Track the **furthest index reachable so far** (`maxReach`). Walk left to right; if you ever stand on an index beyond `maxReach`, you're stuck. Otherwise extend the frontier.

```java
public boolean canJump(int[] nums) {
    int maxReach = 0;
    for (int i = 0; i < nums.length; i++) {
        if (i > maxReach) return false;            // can't even get here
        maxReach = Math.max(maxReach, i + nums[i]); // extend the frontier
    }
    return true;
}
//  [2,3,1,1,4] -> reachable; [3,2,1,0,4] -> stuck at index 3
```

**Why greedy beats DP here:** the DP solution (`dp[i]` = "is `i` reachable?", checking all prior jumps) is O(n²). The frontier insight makes it O(n)/O(1) — you never need *how* you got somewhere, only the furthest you can go.

### Worked example — LC 134 Gas Station (the discard proof)

If `total gas >= total cost`, a unique starting station exists. Track a running tank; whenever it goes negative at station `i`, **no station in `[start..i]` can be the answer** — reset `start = i+1` and zero the tank.

```java
public int canCompleteCircuit(int[] gas, int[] cost) {
    int total = 0, tank = 0, start = 0;
    for (int i = 0; i < gas.length; i++) {
        int diff = gas[i] - cost[i];
        total += diff;
        tank  += diff;
        if (tank < 0) { start = i + 1; tank = 0; } // discard [start..i]
    }
    return total >= 0 ? start : -1;
}
```

**The proof you must say aloud:** if the tank goes negative at `i` starting from `start`, then *every* station between `start` and `i` also fails as a start — because each of those starts had even less accumulated gas when it reached `i` (it skipped the positive prefix `start` enjoyed). So you can safely jump past all of them to `i+1`. Total ≥ 0 guarantees the surviving candidate completes the loop.

### Theory — the interval playbook

Almost every interval problem is **"sort, then single pass."** The only decision is the **sort key**:

| Goal | Sort by | Pass logic |
|---|---|---|
| **Merge** overlapping (56) | **start** | extend last merged `end`, else append |
| **Insert** one interval (57) | already sorted | before / merge-overlap / after |
| **Max non-overlapping kept** (435) | **end** | greedily keep the earliest-ending |
| **Min rooms / max overlap** (253) | **start** (+ min-heap of ends) | reuse a room when its end ≤ new start |

### Practice set

**1. LC 55 — Jump Game** (Medium)
- **Approach:** the `maxReach` frontier above.
- **Key insight:** never simulate individual jumps — track only the furthest reachable index. O(n)/O(1) vs the O(n²) DP.
- **Complexity:** O(n) / O(1).

**2. LC 45 — Jump Game II** (Medium)
- **Approach:** greedy BFS-by-levels. Track `currentEnd` (the farthest reachable with the current jump count) and `farthest` (the best you can reach overall); when `i` hits `currentEnd`, you must take a jump — increment and set `currentEnd = farthest`.
```java
public int jump(int[] nums) {
    int jumps = 0, currentEnd = 0, farthest = 0;
    for (int i = 0; i < nums.length - 1; i++) {   // stop before the last index
        farthest = Math.max(farthest, i + nums[i]);
        if (i == currentEnd) { jumps++; currentEnd = farthest; }
    }
    return jumps;
}
```
- **Key insight:** it's implicit BFS — each "level" is the set of indices reachable in `k` jumps; `currentEnd` is the level boundary. Stopping at `n-1` avoids an extra phantom jump.
- **Complexity:** O(n) / O(1).

**3. LC 134 — Gas Station** (Medium)
- **Approach:** total feasibility check + the discard-on-negative reset above.
- **Key insight:** the discard proof — stations before a negative-tank reset can never start a successful loop.
- **Complexity:** O(n) / O(1).

**4. LC 56 — Merge Intervals** (Medium)
- **Approach:** sort by start; if the next interval's start ≤ the last merged end, extend the end (`max`), else append.
```java
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
    List<int[]> out = new ArrayList<>();
    int[] cur = intervals[0];
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] <= cur[1])                       // overlap
            cur[1] = Math.max(cur[1], intervals[i][1]);
        else { out.add(cur); cur = intervals[i]; }            // disjoint
    }
    out.add(cur);
    return out.toArray(new int[0][]);
}
```
- **Key insight:** sorting by start guarantees that any interval overlapping `cur` appears immediately after it. Use `max` for the end — a fully-contained interval mustn't shrink the merge.
- **Complexity:** O(n log n) / O(n).

**5. LC 57 — Insert Interval** (Medium)
- **Approach:** the input is already sorted → three phases: copy all intervals ending before the new one; merge all that overlap (extend by `max` start/end); copy the rest. No sort needed → O(n).
```java
public int[][] insert(int[][] intervals, int[] ni) {
    List<int[]> out = new ArrayList<>();
    int i = 0, n = intervals.length;
    while (i < n && intervals[i][1] < ni[0]) out.add(intervals[i++]);      // before
    while (i < n && intervals[i][0] <= ni[1]) {                            // overlap
        ni[0] = Math.min(ni[0], intervals[i][0]);
        ni[1] = Math.max(ni[1], intervals[i][1]);
        i++;
    }
    out.add(ni);
    while (i < n) out.add(intervals[i++]);                                 // after
    return out.toArray(new int[0][]);
}
```
- **Key insight:** the three-phase structure is the whole trick — it's a common FAANG variant; drill it until mechanical. O(n) because the list is pre-sorted.
- **Complexity:** O(n) / O(n).

**6. LC 435 — Non-overlapping Intervals** (Medium)
- **Approach:** minimum removals = `n − (max non-overlapping kept)`. Sort by **end**; greedily keep an interval if its start ≥ the last kept end; otherwise it overlaps → count a removal.
```java
public int eraseOverlapIntervals(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[1], b[1])); // sort by END
    int kept = 0, lastEnd = Integer.MIN_VALUE;
    for (int[] iv : intervals)
        if (iv[0] >= lastEnd) { kept++; lastEnd = iv[1]; }         // keep
    return intervals.length - kept;
}
```
- **Key insight:** **sort by end**, not start. Keeping the earliest-ending compatible interval leaves the most room for the rest — the classic activity-selection exchange argument.
- **Complexity:** O(n log n) / O(1).

**7. LC 253 — Meeting Rooms II** (Medium)
- **Approach:** minimum rooms = maximum number of meetings overlapping at any instant. Sort by start; a **min-heap of end times** holds active meetings; for each meeting, if the earliest end ≤ its start, reuse that room (poll), then offer its end. Heap size = rooms needed.
```java
public int minMeetingRooms(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0])); // by start
    PriorityQueue<Integer> ends = new PriorityQueue<>();           // min-heap of ends
    for (int[] m : intervals) {
        if (!ends.isEmpty() && ends.peek() <= m[0]) ends.poll();   // room frees up
        ends.offer(m[1]);
    }
    return ends.size();
}
```
- **Key insight:** the heap's peek is the soonest-freeing room; if it's free by the new meeting's start, reuse it instead of allocating. The alternative **sweep line** (sort all start/end events, +1/−1, track max) is equally valid and worth mentioning.
- **Complexity:** O(n log n) / O(n). Prerequisite pattern for many scheduling problems.

---

## Block B — API Gateway + gRPC vs REST vs Messaging (~2 hr)

> Cross-ref: `../../08-modern-java.md` — Spring Boot 3's `WebClient`/`RestClient` for REST, the gRPC Spring Boot starter for service-to-service, and `KafkaTemplate`/`@KafkaListener` for async messaging. This block is about *choosing* among them by trade-off.

### The mental model

Microservices have to talk. The choice among **REST**, **gRPC**, and **async messaging** is a trade-off across five dimensions — coupling, latency, schema evolution, observability, and failure mode. There is no universally best answer; the senior signal is naming *which axis* drove your decision for a *specific* context.

| Dimension | REST (HTTP/1.1 + JSON) | gRPC (HTTP/2 + Protobuf) | Async messaging (Kafka/RabbitMQ) |
|---|---|---|---|
| **Coupling** | Loose (URL + JSON contract) | Tight (shared `.proto` schema) | Loosest (producer doesn't know consumers) |
| **Latency** | ~1ms+ (JSON parse, text) | ~0.3ms (binary, multiplexed) | ms → seconds (async by design) |
| **Schema evolution** | Risky (no enforcement) | Controlled (field *numbers* are the contract) | Risky unless a schema registry |
| **Observability** | Easy (plain HTTP logs/tools) | Needs gRPC interceptors | Needs correlation IDs in the message |
| **Failure mode** | Synchronous — caller waits/times out | Synchronous — caller waits/times out | Asynchronous — retry + dead-letter queue |

### REST (HTTP/1.1 + JSON) — the default

Universal tooling (curl, browsers, Postman), human-readable, stateless, leverages HTTP caching and status codes. Downsides: chatty for many small fields, no built-in streaming, and **silent breaking changes** (rename a field and clients break at runtime, not compile time). **Smart360 used REST gateway → services**: human-debuggable, the Vue front end consumes it directly, and the team knew it cold. Payloads were KB-sized, so binary efficiency wasn't worth the cost.

### gRPC (HTTP/2 + Protobuf) — internal high-throughput

3–10× smaller payloads (binary Protobuf), HTTP/2 multiplexing and **bidirectional streaming**, a strong `.proto` contract with generated client/server stubs in every language. The schema-evolution story is its superpower: **field numbers**, not names, are the wire contract — add a field with a new number and old clients ignore it; never reuse or renumber a field.

```proto
// user.proto — the contract is the field NUMBER, not the name
syntax = "proto3";
service UserService {
  rpc GetUser (GetUserRequest) returns (User);          // unary
  rpc StreamUsers (stream GetUserRequest) returns (stream User); // bidi stream
}
message User {
  int64 id = 1;          // never reuse number 1
  string email = 2;
  string display_name = 3;   // safe to add as a new number
}
```

Downsides: **not browser-native** (needs gRPC-Web + an Envoy/proxy translation layer), binary payloads are harder to eyeball-debug, and schema evolution needs discipline. Use it for **internal, high-frequency service-to-service** calls where the sub-5ms overhead and streaming matter.

### Async messaging (Kafka / RabbitMQ) — temporal decoupling

The producer fires an event and moves on; consumers process when ready. Buys you **temporal decoupling** (consumer can be down), **backpressure**, **fan-out** (many consumers per event), and **replay** (Kafka retains the log). Costs: **eventual consistency**, harder debugging (no synchronous stack trace), and **ordering** is only guaranteed per Kafka *partition* or per RabbitMQ *queue*. Failures land in a **dead-letter queue** for inspection/redrive rather than failing the caller. **Smart360's notification service** went event-driven after the user-management service was extracted — a `UserRegistered` event fanned out to email, audit, and analytics consumers without the registration path waiting on any of them.

- **Kafka vs RabbitMQ in one line:** Kafka is a *distributed log* (high throughput, replay, partition-ordered, consumer groups) — best for event streaming and fan-out; RabbitMQ is a *smart broker* (flexible routing, per-message ack, priorities, queue-ordered) — best for task queues and complex routing.

### API Gateway vs BFF

- **API Gateway** — a single shared edge in front of all services. Spring Cloud Gateway is a **predicate + filter pipeline**: predicates match a request (`Path`, `Host`, `Method`), filters transform it. Concrete filters from Smart360: `StripPrefix` (drop the `/api` segment before forwarding), `AddRequestHeader` (inject a trace header), `RequestRateLimiter` (yesterday's token bucket), and a **custom `GatewayFilter` for JWT validation** (reject unauthenticated requests at the edge so services trust the gateway). It centralises auth, rate limiting, routing, and observability.
- **BFF (Backend-for-Frontend)** — a *per-client* backend (one for web, one for mobile) that tailors and aggregates responses for that client's exact needs, hiding the fan-out to many services. The gateway is *shared*; a BFF is *client-specific*. Use a BFF when web and mobile need very different response shapes; use a plain gateway when one shape serves all clients.

### Project tie-in

- **Smart360:** REST at the edge (Vue consumes it, human-debuggable) behind Spring Cloud Gateway (`StripPrefix` + `RequestRateLimiter` + custom JWT filter); async Kafka for the notification fan-out after the user-management extraction.
- **"Why not gRPC everywhere?"** — the Vue front end can't speak gRPC without gRPC-Web + a proxy; payloads were KB-sized so binary gave no real win; the team's REST tooling and familiarity outweighed a sub-millisecond latency gain. gRPC was the right call *only* for the internal high-frequency stream, not the browser-facing edge.

---

## Block C — DSA: Interval Re-Solve Drills (~1.5 hr)

### Theory

Fluency block — no new problems, just speed and the sort-key reflex. In an interview, the moment you hear "intervals" you should instantly state the sort key and the pass logic before writing a line.

### Drill set

**Re-solve LC 56, 57, 435 back-to-back, no hints, ≤ 15 min each.** Before each, write **one sentence** naming the sort key and why:

- **Merge (56):** sort by **start** — overlapping intervals become adjacent, so a single pass merges them.
- **Insert (57):** already sorted — three phases (before / merge-overlap / after), no sort, O(n).
- **Non-overlapping (435):** sort by **end** — keeping the earliest-ending interval leaves maximum room for the rest (activity selection).

**State the greedy-vs-DP interval rule aloud:** the *objective* decides it. "Max non-overlapping" / "min removals" → greedy (sort by end). "Max **weight** of non-overlapping" → DP (LC 1235 Weighted Job Scheduling, binary-search the last compatible job + take/skip).

**Spot-the-sort-key flash drill** — for each, snap-answer the key:
- Merge Intervals → start
- Meeting Rooms II → start (+ min-heap of ends)
- Non-overlapping Intervals → end
- Minimum Number of Arrows to Burst Balloons (LC 452) → end (same shape as 435)
- Interval List Intersections (LC 986) → two-pointer over two pre-sorted lists

**Optional extensions if time:**
- **LC 452 Minimum Arrows** — sort by end, shoot at each new non-overlapping end (sibling of 435).
- **LC 986 Interval List Intersections** — two pointers, `[max(starts), min(ends)]`, advance the one ending first.
- **LC 1235 Weighted Job Scheduling** — the DP counterpart: sort by end, `dp[i] = max(skip, value[i] + dp[lastCompatible])`.

---

## 💻 Practice coding questions

1. **LC 55 Jump Game** — `maxReach` frontier; O(n)/O(1).
2. **LC 45 Jump Game II** — level-BFS greedy; `currentEnd`/`farthest`; stop before last index.
3. **LC 134 Gas Station** — total check + discard-on-negative reset; say the proof.
4. **LC 56 Merge Intervals** — sort by start; `max` the end.
5. **LC 57 Insert Interval** — three phases, no sort, O(n).
6. **LC 435 Non-overlapping Intervals** — sort by end; keep earliest end.
7. **LC 253 Meeting Rooms II** — min-heap of ends, or sweep line.
8. **LC 252 Meeting Rooms** — can one person attend all? sort + adjacency check.
9. **LC 452 Minimum Arrows to Burst Balloons** — sort by end, shoot greedily.
10. **LC 986 Interval List Intersections** — two pointers over sorted lists.
11. **LC 763 Partition Labels** — greedy by last-seen index of each char.
12. **LC 1235 Maximum Profit in Job Scheduling** — the DP interval counterpart.
13. **LC 621 Task Scheduler** — greedy with cooldown (heap or the math formula).

> Snap-answer the sort key before coding each interval problem. Re-time LC 57 — target ≤ 12 min mechanical.

---

## 🎤 Interview questions

1. **When is greedy correct, and how do you prove it?** When the problem has the greedy-choice property and optimal substructure. Prove with an exchange argument: any optimal solution can be transformed into the greedy one without getting worse.
2. **Greedy vs DP — how do you decide on an interval problem?** The objective decides: "max non-overlapping" / "min removals" → greedy (sort by end); "max *weight* of non-overlapping" → DP, since greedy can't weigh value trade-offs.
3. **Jump Game — why is greedy O(n) and DP O(n²)?** Greedy tracks only the furthest reachable index (`maxReach`); you never need *how* you reached a cell. DP recomputes reachability over all prior jumps per cell.
4. **Gas Station — prove the discard step.** If the tank goes negative at `i` starting from `start`, every station in `[start..i]` also fails as a start (each had less accumulated gas reaching `i`). So jump to `i+1`. Total gas ≥ total cost guarantees the survivor completes the loop.
5. **Merge Intervals — why sort by start?** Sorting by start makes any interval overlapping the current one appear immediately after it, so a single pass merges correctly. Use `max` for the end so a contained interval doesn't shrink the merge.
6. **Non-overlapping Intervals — why sort by *end*?** Keeping the earliest-ending compatible interval leaves the most room for subsequent ones — the activity-selection exchange argument. Sorting by start would greedily keep a long early interval that blocks several short ones.
7. **Meeting Rooms II — what does the min-heap represent?** The end times of currently-active meetings; its peek is the soonest-freeing room. If that end ≤ the next meeting's start, reuse the room; else allocate one. Heap size = rooms needed. Sweep line is the equivalent alternative.
8. **REST vs gRPC vs messaging — name the axis each wins on.** REST: tooling/debuggability/browser-native. gRPC: latency + binary size + streaming + enforced schema for internal calls. Messaging: temporal decoupling, fan-out, replay, backpressure.
9. **Why did you choose REST over gRPC for the edge?** The browser front end can't speak gRPC without gRPC-Web + a proxy; KB payloads don't justify binary; team tooling and familiarity. gRPC was reserved for internal high-frequency service-to-service streams.
10. **How does gRPC handle schema evolution safely?** The wire contract is the **field number**, not the name. Add fields with new numbers (old clients ignore them); never reuse or renumber a field. That's stronger than REST/JSON, which breaks silently on a rename.
11. **What does async messaging buy you, and what does it cost?** Buys temporal decoupling, fan-out, backpressure, replay. Costs eventual consistency, harder debugging (no synchronous trace), and ordering only per partition/queue. Failures go to a dead-letter queue.
12. **Kafka vs RabbitMQ?** Kafka is a distributed, replayable log (high throughput, partition-ordered, consumer groups) — event streaming and fan-out. RabbitMQ is a smart broker (flexible routing, per-message ack, priorities, queue-ordered) — task queues and complex routing.
13. **API Gateway vs BFF?** The gateway is a single shared edge (auth, routing, rate limiting, observability) for all clients. A BFF is a per-client backend that aggregates and tailors responses for one client type. Gateway = shared; BFF = client-specific.
14. **What runs in a Spring Cloud Gateway route?** A predicate + filter pipeline: predicates match (`Path`, `Host`, `Method`); filters transform (`StripPrefix`, `AddRequestHeader`, `RequestRateLimiter`, a custom JWT-validation `GatewayFilter`). Auth and rate limiting happen at the edge so services can trust the gateway.
15. **Where should authentication live — gateway or each service?** Validate the JWT at the gateway (reject unauthenticated traffic at the edge), then pass identity claims downstream so services don't each re-validate against the auth provider. Services still enforce *authorization* on their own resources.
16. **(Curveball) Your message consumer is slow and the queue backs up — what do you do?** Scale consumers (more partitions/consumer-group members for Kafka), apply backpressure, move poison messages to a dead-letter queue so one bad message doesn't block the queue, and ensure consumers are idempotent so redelivery is safe.
17. **(Curveball) How do you keep ordering with Kafka when you scale consumers?** Ordering is per-partition. Key messages by the entity that needs ordering (e.g. `userId`) so all of that entity's events land on one partition and one consumer processes them in order; parallelism comes from many keys across partitions.
18. **(Curveball) A REST call between two internal services times out under load — first move?** Add a timeout + circuit breaker + bulkhead (yesterday's Resilience4j), then evaluate switching that hot path to gRPC (lower latency, multiplexing) or to async messaging if the caller doesn't actually need a synchronous response.

---

## ✅ Self-check

1. Explain *why* Gas Station's greedy discard step is safe — the proof, not just the steps.
2. For each of Merge / Insert / Non-overlapping / Meeting Rooms II, snap-answer the sort key and the one-line pass logic.
3. A recruiter asks "why didn't you just use gRPC everywhere?" — answer in 90 seconds referencing specific Smart360 constraints (browser, payload size, tooling).
4. State the greedy-vs-DP interval rule in one sentence, with the example problem for each side.
5. Name the five comparison dimensions for REST vs gRPC vs messaging and which option wins each.

---

<div align="center">

← [02-tue-jul-21.md](02-tue-jul-21.md) · [Week 4](README.md) · [04-thu-jul-23.md](04-thu-jul-23.md) →

</div>
