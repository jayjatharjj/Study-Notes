# Week 6 — Core Build (Jul 13–19, 2026)

> **Theme: Graphs + Async Messaging Architecture — Own the design room and the whiteboard.**

---

## 🎯 Week Goal

Exit the week able to solve any LeetCode medium graph problem cold in ≤25 min, and design a production-grade async event-driven system end-to-end — including Kafka internals, idempotency guarantees, the Outbox pattern, and both Saga flavors — while tying every answer back to **Smart360's monolith→event-driven migration** and **WebX's async LLM jobs**.

---

## ✅ By Sunday you can...

- Implement BFS, DFS, topological sort, union-find, and cycle detection from memory in Java with correct edge-case handling
- Solve all 8 target LeetCode graph problems in under 25 min each without hints
- Explain Kafka partitions, consumer groups, and offset commits like a senior engineer who has used them in prod
- Draw the Outbox + CDC flow on a whiteboard and defend every component choice
- Compare choreography vs orchestration Saga with a real trade-off argument, citing your Smart360 strangler-fig migration
- Articulate why WebX uses async job queues (`202 Accepted` + polling/SSE) and how Kafka would upgrade that design
- Answer "why not just use a bigger database?" and "where does exactly-once actually break down?" without hesitation
- Demo or walk through a clean GitHub project (Spring Boot + Vue/React) with a README that a recruiter can parse in 90 seconds

---

## 📅 Daily Checklist (Mon–Sun)

---

### Monday Jul 13 — Graph Foundations: BFS & DFS

**Total: ~1.5 hr**

**DSA (50 min)**
- [ ] Implement adjacency list and adjacency matrix graph in Java — `Map<Integer, List<Integer>>` for sparse, `int[][]` for dense; know when to choose which
- [ ] Implement iterative BFS (queue) and recursive + iterative DFS (stack) from scratch; trace through a 5-node directed graph
- [ ] Solve **LC 200 — Number of Islands** (BFS or DFS with visited set); target: AC in 15 min
  - Key insight: grid = implicit graph; 4-directional neighbors; mark visited in-place (`'1'→'0'`) vs separate `boolean[][]`
  - Edge cases: all water, single cell, non-square grid
- [ ] Solve **LC 133 — Clone Graph** (DFS + HashMap for node-clone mapping); target: AC in 20 min
  - Key insight: HashMap maps original→clone to handle cycles; can't just copy node values, must rewire adjacency

**Core Concept (25 min)**
- [ ] Read: Kafka official docs — "Introduction" + "Design" sections (producer, broker, consumer roles; partitions as ordered log)
- [ ] Write in your own words (3 sentences each): What is a partition? What does a consumer group do? What is an offset?

**Self-Check**
- [ ] Can you explain BFS vs DFS trade-offs in 60 seconds? (BFS: shortest path unweighted, level-order; DFS: cycle detection, topological, less memory for deep graphs)
- [ ] Can you state the time/space complexity for Number of Islands? (O(m×n) time, O(m×n) space worst case for recursion stack / visited)

---

### Tuesday Jul 14 — Topological Sort + Kafka Internals Deep Dive

**Total: ~1.5 hr**

**DSA (50 min)**
- [ ] Implement **Kahn's algorithm** (BFS-based topo sort with in-degree array) from memory — cleaner for cycle detection
- [ ] Implement **DFS-based topo sort** (postorder + reverse) — understand both; interviewers ask for both
- [ ] Solve **LC 207 — Course Schedule I** (can finish? → cycle detection in directed graph); target: AC in 20 min
  - Track 3 states per node: 0=unvisited, 1=in-current-path (cycle!), 2=fully processed; or use Kahn's: if processed nodes < total → cycle
- [ ] Solve **LC 210 — Course Schedule II** (actual topo order); target: AC in 25 min
  - Extension of 207: collect postorder DFS into a stack or use Kahn's output array
  - Edge cases: disconnected components, self-loops, empty prerequisites list

**Core Concept (25 min)**
- [ ] **Kafka Internals — go deep:**
  - Partition leader election and ISR (in-sync replicas) — what happens if leader dies mid-write?
  - `acks=all` + `min.insync.replicas=2` — the combination that prevents data loss; know why `acks=1` is dangerous
  - Consumer group rebalance: when does it happen? (new consumer joins/leaves, partition count changes, session timeout) What is the `group.coordinator`?
  - `auto.offset.reset=earliest` vs `latest` — when each matters; what happens on first consumer startup?
- [ ] Tie to Smart360: "If I had instrumented the monolith→event migration with Kafka instead of ad-hoc async, I'd have set `acks=all` for the authorization events (correctness critical) and `acks=1` for notification events (latency-sensitive, idempotent consumer anyway)"

**Self-Check**
- [ ] What is the difference between Kafka and RabbitMQ in one paragraph? (Kafka: durable log, replay, high throughput, consumer pulls; RabbitMQ: message broker, push-based, traditional queue semantics, routing via exchanges, messages deleted on ACK)
- [ ] What makes Kahn's algorithm detect a cycle? (If final processed count < V, there's a cycle)

---

### Wednesday Jul 15 — Union-Find + Idempotency & Exactly-Once

**Total: ~1.5 hr**

**DSA (50 min)**
- [ ] Implement Union-Find with **path compression** + **union by rank** from memory — must be muscle memory
  ```java
  int[] parent, rank;
  int find(int x) { return parent[x] == x ? x : (parent[x] = find(parent[x])); } // path compression
  void union(int x, int y) { int px = find(x), py = find(y); if(px==py) return;
      if(rank[px]<rank[py]) { parent[px]=py; } else if(rank[px]>rank[py]) { parent[py]=px; }
      else { parent[py]=px; rank[px]++; } }
  ```
- [ ] Solve **LC 684 — Graph Valid Tree** (n nodes, n-1 edges, connected, no cycle → valid tree); target: AC in 20 min
  - Use Union-Find: if `union(u,v)` returns false (same component already) → cycle found; also check `components == 1`
- [ ] Solve **LC 323 — Number of Connected Components** (Union-Find or DFS count); target: AC in 15 min
  - Simple: count how many times `find(i) == i` after all unions, or count DFS entry points with visited array

**Core Concept (30 min)**
- [ ] **Idempotency + Exactly-Once vs At-Least-Once:**
  - Define idempotent consumer: processing the same message N times produces the same result as processing it once
  - How to implement: unique `messageId` in payload → check in DB/Redis before processing → skip if already seen
  - Kafka **exactly-once semantics (EOS)**: requires producer `enable.idempotence=true` + `transactional.id` + consumer `isolation.level=read_committed`; stream-processing only — crosses broker boundary, not external DB
  - Why "exactly-once to an external DB" is a myth: the DB write and the Kafka offset commit are two separate systems; you must make the consumer idempotent anyway
  - **At-least-once** is the real-world default; design consumers to handle duplicates
- [ ] Tie to WebX: "LLM job queue uses at-least-once delivery. If a worker picks up a job, calls the LLM, crashes before ACK, another worker retries. Without idempotency check on `jobId`, the LLM is called twice and the user sees duplicate output. The fix: check `job_status != 'COMPLETED'` in the same DB transaction as updating status."

**Self-Check**
- [ ] What is the amortized time complexity of Union-Find with path compression + union by rank? (near O(1) per operation — inverse Ackermann α(n))
- [ ] Can you define "exactly-once" without using the word "exactly" or "once"? Force yourself to explain the delivery semantics precisely.

---

### Thursday Jul 16 — Advanced Graphs + Outbox Pattern

**Total: ~1.5 hr**

**DSA (50 min)**
- [ ] Solve **LC 417 — Pacific Atlantic Water Flow** (BFS/DFS from both oceans inward); target: AC in 25 min
  - Key insight: reverse the direction — instead of "can water flow from cell to ocean", do BFS outward from all ocean-border cells
  - Two visited sets (pacific, atlantic); answer = intersection; use BFS with deque or DFS
  - Common mistake: doing BFS from every cell (O(m²n²)) — always think about reversing direction in flow problems
- [ ] Solve **LC 994 — Rotting Oranges** (multi-source BFS from all rotten oranges simultaneously); target: AC in 20 min
  - Multi-source BFS: seed queue with ALL initially rotten positions; BFS levels = minutes
  - After BFS, check if any fresh remains — if yes, return -1
  - Edge cases: no fresh oranges (return 0), isolated fresh oranges (return -1)

**Core Concept (30 min)**
- [ ] **Outbox Pattern — production-grade understanding:**
  - Problem it solves: dual-write problem (save entity + publish event are two I/O ops; crash between them = lost event)
  - Schema: `outbox_events(id UUID, aggregate_type, aggregate_id, event_type, payload JSONB, created_at, status)` — write in same transaction as domain entity change
  - Delivery options:
    1. **Polling publisher**: scheduled job (`@Scheduled`) reads `status='PENDING'` rows, publishes to Kafka, updates to `status='SENT'` — simple but adds DB load and latency
    2. **CDC with Debezium**: tails Postgres WAL (write-ahead log), captures row inserts, streams to Kafka — near real-time, no polling, but adds Debezium operational complexity
  - At-least-once: outbox guarantees delivery but not exactly-once — consumer must be idempotent
  - Tie to Smart360: "When I migrated user management to event-driven, we had the dual-write bug in the first iteration — authorization events were occasionally lost on pod restart. The fix was the Outbox pattern: the `UserUpdated` event was written to `outbox_events` in the same Hibernate transaction as the user entity update. A scheduler published and cleaned up."

**Self-Check**
- [ ] Why is Pacific Atlantic Water Flow's BFS solution O(m×n) not O(m²n²)?
- [ ] What is Debezium and why is it better than polling for high-throughput outbox processing?

---

### Friday Jul 17 — Word Ladder + Saga Pattern

**Total: ~1.5 hr**

**DSA (50 min)**
- [ ] Solve **LC 127 — Word Ladder** (BFS shortest path, treat words as graph nodes with edge if 1-char diff); target: AC in 30 min
  - Build adjacency lazily: for each word, try replacing each char with a–z; if result in wordSet, it's a neighbor
  - BFS gives shortest path; use a `visited` set to avoid revisiting
  - Key optimization: bidirectional BFS — expand from both start and end, meet in the middle; reduces O(b^d) to O(b^(d/2))
  - Common mistake: not removing words from the `wordSet` as you visit them (TLE)
  - Edge cases: begin==end, endWord not in wordList
- [ ] Review all 8 graph problems: can you state the pattern for each in one sentence?
  - Islands → flood fill DFS/BFS
  - Clone Graph → DFS + node map
  - Course Schedule → topo sort / cycle detection
  - Valid Tree → Union-Find
  - Connected Components → Union-Find or DFS count
  - Pacific Atlantic → reverse-direction multi-source BFS
  - Rotting Oranges → multi-source BFS with time levels
  - Word Ladder → BFS shortest path on implicit graph

**Core Concept (30 min)**
- [ ] **Saga Pattern — deep comparison:**
  - **Choreography**:
    - No central coordinator; each service publishes domain events and others react
    - Pros: loose coupling, services fully autonomous, no SPOF
    - Cons: hard to visualize full flow, distributed debugging nightmare, risk of cycles if not careful
    - Use when: simple 2–3 step flows, services are mature and stable, you own all consumers
  - **Orchestration**:
    - Saga orchestrator (a dedicated service or state machine — e.g., Temporal, AWS Step Functions, or custom) explicitly sends commands and awaits replies
    - Pros: full visibility, easy rollback/compensation logic in one place, traceable
    - Cons: orchestrator is a bottleneck and SPOF; tighter coupling between orchestrator and participants
    - Use when: complex multi-step flows (5+ services), flows involve external systems (payment gateway), compliance/audit requirements
  - **Compensating transactions**: each step must have a defined undo operation — design these BEFORE you write the forward path
  - Tie to Smart360: "For the notification extraction, I used choreography — the Authorization service published `UserRoleChanged` and the Notification service consumed it. Simple, 2 services, no complex rollback needed. For a hypothetical order-payment-inventory flow, I'd use orchestration — too many failure modes to debug with pure events."

**Self-Check**
- [ ] Can you describe a saga where the compensation transaction is non-trivial? (e.g., payment was captured but inventory failed — you can't "undo" a bank transfer, you issue a refund; this is a compensating transaction, not a rollback)
- [ ] In Word Ladder, why does BFS guarantee the shortest transformation sequence? (BFS explores level by level; first time you reach endWord is via the fewest edges)

---

### Saturday Jul 18 — GitHub Project Polish + System Design Deep Session

**Total: ~4 hr**

**DSA Review (45 min)**
- [ ] Re-solve any 2 of the 8 graph problems you found hardest this week — no looking at previous solution
- [ ] Practice verbal explanation: pick one problem, solve it while talking out loud as if in a live interview. Record yourself on phone.
- [ ] Review time/space complexity for all 8 problems; write them down from memory

**System Design Deep Session (90 min)**

Design: "Design the async LLM job processing system for WebX at 10× current scale"

Work through these layers:
- [ ] **Client interface**: POST `/jobs` → `202 Accepted` with `{jobId}`. Why not `200`? Why not `201`? (202 = accepted but not yet processed — correct HTTP semantics for async)
- [ ] **Job submission**: Spring Boot controller → write job to `jobs` table (status=`QUEUED`) + write event to `outbox_events` in same transaction
- [ ] **Event delivery**: Debezium or scheduler reads outbox → publishes to Kafka topic `llm-jobs` with `jobId` as partition key (ensures ordering per job)
- [ ] **Worker/consumer**: Spring Kafka `@KafkaListener` consumer group `llm-workers`; fetch job details from DB by `jobId`, call LLM provider, update job status, handle idempotency
- [ ] **Status polling vs push**: GET `/jobs/{id}/status` for polling; SSE endpoint for push; which to choose when?
- [ ] **Failure handling**: dead letter topic (`llm-jobs-dlq`) after N retries; alert on DLQ messages; manual reprocessing workflow
- [ ] **Scaling**: partition count = max desired parallelism; consumer group scales horizontally up to partition count; beyond that → increase partitions (can only increase, never decrease — plan ahead)
- [ ] **Exactly-once guarantee**: can't achieve with external LLM (HTTP call to OpenAI is not transactional); make worker idempotent — check `job_status != 'COMPLETED'` before calling LLM
- [ ] **Draw the full architecture diagram** (paper or whiteboard): Client → API → DB (jobs + outbox) → Debezium/Scheduler → Kafka → Worker pool → LLM Provider → DB update → SSE push

**GitHub Project Work (75 min)**

Pick ONE of:
- [ ] Option A: Create `smart360-event-demo` — minimal Spring Boot app showing the Outbox pattern in action: POST `/users` writes user + outbox event in same transaction; `@Scheduled` publisher reads outbox and logs "would publish to Kafka"; GET `/events` shows outbox table. Vue.js frontend showing status. README explains the pattern, the dual-write problem it solves, and links to your Smart360 experience.
- [ ] Option B: Clean up an existing repo — add a proper README with: architecture diagram (Mermaid or ASCII), tech stack badges, how to run locally with Docker Compose, and one screenshot. Remove all TODO comments, fix all compile warnings.

**README must include:**
- [ ] One-paragraph "What problem does this solve?" — recruiter-targeted
- [ ] Architecture diagram (even ASCII is fine)
- [ ] `docker-compose up` runnable in < 5 commands
- [ ] Link to relevant resume bullet points (Smart360 / WebX / Deep Fathom)

**Self-Check**
- [ ] Push the repo. Look at it as a stranger. Would you click "Star"?
- [ ] Can you walk through the full LLM job system design in 10 minutes without notes?

---

### Sunday Jul 19 — Mock Interview Day + Week Consolidation

**Total: ~4 hr**

**Mock Interview Session 1: DSA (60 min)**
- [ ] Set a 25-min timer. Solve **LC 207 (Course Schedule)** as if in a live interview: think aloud, handle edge cases, state complexity
- [ ] Set a 25-min timer. Solve **LC 994 (Rotting Oranges)** same conditions
- [ ] Debrief 10 min: what did you say under pressure that was wrong? Where did you hesitate?

**Mock Interview Session 2: System Design (60 min)**
- [ ] Design "Event-Driven Order Processing System" (no hints): client → order service → Kafka → payment service → inventory service → notification service
  - Must cover: Outbox pattern, idempotency, Saga (choose orchestration or choreography and defend), failure recovery, exactly-once discussion, scaling
  - Time yourself: 5 min clarify requirements, 10 min high-level, 20 min deep dive, 10 min scale + failure modes, 15 min questions/defense

**Mock Interview Session 3: Behavioral (30 min)**
- [ ] Answer all 5 behavioral questions in the section below as STAR responses, timed 2 min each
- [ ] Record yourself answering one — play it back, check: did you say "um" more than 3 times? Did you give specific numbers?

**Consolidation (30 min)**
- [ ] Write 3 bullet points for EACH graph algorithm: when to use it, what's the pattern, one gotcha
- [ ] Write the Kafka delivery guarantee comparison table from memory:

| Guarantee | Producer Config | Consumer Design | Trade-off |
|---|---|---|---|
| At-most-once | acks=0 | auto-commit before process | Fast, can lose messages |
| At-least-once | acks=all | commit after process | Safe, must handle duplicates |
| Exactly-once | idempotent producer + transactions | read_committed | Complex, Kafka→Kafka only |

- [ ] Update `interview-qa.md` with any new Q&A formulations you refined this week
- [ ] Review the Week 7 theme — is there anything from this week you want to carry forward?

---

## 🧠 Concepts to Master This Week

### Graph Algorithms

| Algorithm | Pattern | Time | Space | Jay's Go-To Use Case |
|---|---|---|---|---|
| BFS | Queue, visited set, level-by-level | O(V+E) | O(V) | Shortest path unweighted, multi-source spread (Rotting Oranges) |
| DFS recursive | Call stack, visited set | O(V+E) | O(V) stack | Flood fill, cycle detection, topo sort |
| DFS iterative | Explicit stack, visited | O(V+E) | O(V) | DFS when recursion depth risks StackOverflow |
| Kahn's Topo Sort | In-degree array + BFS | O(V+E) | O(V) | Course Schedule — detects cycle if processed < V |
| DFS Topo Sort | Postorder + reverse | O(V+E) | O(V) | Course Schedule II — natural ordering |
| Union-Find | parent/rank arrays, path compress | O(α(n)) | O(V) | Connected components, cycle in undirected, MST |
| Multi-source BFS | Seed queue with multiple starts | O(V+E) | O(V) | Rotting Oranges, Pacific Atlantic |
| Bidirectional BFS | Two frontiers, meet in middle | O(b^(d/2)) | O(b^(d/2)) | Word Ladder optimization |

### Messaging & Async Architecture

**Kafka Fundamentals (must know cold)**
- **Topic**: logical channel; split into ordered, immutable **partitions**
- **Partition key**: determines which partition a message lands in; same key → same partition → ordering guarantee per key; null key → round-robin
- **Consumer group**: all consumers in a group share partition assignment (each partition → exactly one consumer in the group); scale consumers up to partition count
- **Offset**: monotonically increasing message position in a partition; consumer commits offset to track progress; crash recovery = re-read from last committed offset
- **Replication**: each partition has leader + follower replicas on different brokers; ISR = replicas caught up to leader
- **Retention**: Kafka retains messages for a configured time (default 7 days) regardless of consumption — can replay events; RabbitMQ deletes on ACK

**RabbitMQ vs Kafka Decision Matrix**
- Choose Kafka when: high throughput, event replay needed, multiple independent consumers of same event, audit log, stream processing
- Choose RabbitMQ when: complex routing (topic/fanout/direct exchanges), task queues with work stealing, lower throughput, traditional message queue semantics, easier ops for smaller teams

**Outbox Pattern — Implementation Checklist**
1. Domain entity table + `outbox_events` table in same DB
2. Write both in same `@Transactional` method
3. Outbox schema: `id, event_type, aggregate_id, payload JSONB, status ENUM(PENDING/SENT/FAILED), created_at, processed_at`
4. Publisher options: polling scheduler (simple) or Debezium CDC (low-latency, WAL-based)
5. After publish: update `status='SENT'` — at-least-once; consumer must be idempotent
6. Add index on `(status, created_at)` for polling query performance

**Idempotency Implementation Patterns**
- **Message-level**: include `messageId` in payload; consumer checks `processed_messages(message_id)` table before processing; insert on first seen
- **Business-level**: `IF job_status = 'QUEUED' THEN process` — state machine ensures same job not processed twice
- **Optimistic locking**: `UPDATE jobs SET status='PROCESSING' WHERE id=? AND status='QUEUED'`; only one consumer wins the race

**Saga — When to Use Each**

Choreography checklist (use if ALL true):
- ≤3 services involved
- Each step is independently reversible or irreversible but acceptable
- You own all consumers
- No strict ordering requirement across steps

Orchestration checklist (use if ANY true):
- ≥4 services involved
- Need audit trail of saga execution
- Complex compensating transactions with retry logic
- External service integration (payment gateway, shipping API)
- Regulatory/compliance requirement to trace every step

---

## 🎤 Sample Interview Questions (incl. curveballs) — 15 Qs + Pointers

**Graph / DSA**

1. **"Walk me through how you'd detect a cycle in a directed vs undirected graph."**
   - Directed: DFS with 3-color marking (white/gray/black) or Kahn's (cycle if processed < V). Undirected: DFS tracking parent — if you visit a neighbor that's not your parent, it's a cycle; or Union-Find.
   - Don't mix up: Union-Find works for undirected; DFS 3-color for directed.

2. **"What's the difference between topological sort and a regular sort?"**
   - Topological sort is defined only on DAGs; produces a linear ordering where for every edge u→v, u comes before v. Not unique — can have multiple valid orderings. Regular sort (comparison) has a total order. Topo sort uses graph structure, not value comparison.

3. **"Can you do topological sort on a graph with a cycle? What happens?"**
   - No — topological sort is undefined for cyclic graphs. Kahn's algorithm detects this: processed node count < V means a cycle exists. DFS-based approach gets stuck in the gray set. This is exactly the Course Schedule problem.

4. **"Curveball: In Number of Islands, what if the grid is 100,000 × 100,000? How does your approach change?"**
   - DFS would cause a StackOverflow (recursion depth up to 10^10). Switch to iterative BFS/DFS with explicit queue/stack. For distributed scenarios: partition the grid into chunks, process each with MapReduce/Spark, use Union-Find across partition boundaries.

**Messaging / Async**

5. **"What happens when a Kafka consumer crashes mid-processing — message has been received but not yet committed?"**
   - If auto-commit was not yet fired, the consumer group coordinator detects the missed heartbeat, triggers a rebalance, and reassigns the partition to another consumer. That consumer re-reads from the last committed offset — so the message is reprocessed. This is at-least-once delivery. Your consumer must be idempotent to handle this correctly.

6. **"Smart360: you said async messaging gave 30% efficiency boost. What exactly was inefficient before?"**
   - Before: synchronous REST call from Authorization service to Notification service on every user role change. If Notification service was slow or down, it blocked Authorization service and caused timeout errors, degrading the user management flow. After: fire-and-forget event publish; Notification service consumes asynchronously. The Authorization service returns immediately; the 30% improvement came from eliminating blocking I/O in the hot path.

7. **"What is the difference between a Kafka consumer group with 3 consumers and a topic with 2 partitions vs 4 partitions?"**
   - 2 partitions + 3 consumers: one consumer is idle (no partition to read from). Partition count caps parallelism.
   - 4 partitions + 3 consumers: one consumer reads 2 partitions. All consumers are busy. Throughput scales better.
   - Rule: partition count = max desired parallelism. Plan partition count at topic creation — increasing is possible, but it breaks key-based ordering for existing messages.

8. **"Curveball: Your Outbox poller runs every 5 seconds. What if the DB goes down right after you publish to Kafka but before you mark the outbox row as SENT?"**
   - The row stays `PENDING`. Next poll cycle, the event is published again → duplicate in Kafka → consumer receives it twice. This is why your consumer MUST be idempotent. There is no way to make this atomic across DB + Kafka without a distributed transaction, which you're explicitly avoiding. Accept at-least-once, design idempotent consumers. This is the correct, honest answer — interviewers respect engineers who acknowledge distributed systems trade-offs.

9. **"What is the difference between idempotent producer in Kafka and idempotent consumer?"**
   - Idempotent producer (`enable.idempotence=true`): Kafka broker deduplicates retried producer requests using a sequence number per partition — prevents duplicate writes to the Kafka log from the same producer. Broker-side guarantee.
   - Idempotent consumer: your application code deduplicates messages it receives — prevents duplicate effects on your downstream DB/service. Application-side guarantee. The two are independent.

10. **"Curveball: A junior engineer says 'let's just use a bigger, faster DB instead of Kafka for our event queue.' How do you respond?"**
    - Acknowledge: for low volume, a DB-backed queue (like using PostgreSQL `SKIP LOCKED`) actually works well and is simpler. Outbox pattern is essentially this.
    - Explain limits: at scale (millions of events/sec), DB polling adds load to your primary DB, doesn't give you replay, and doesn't fan out to multiple independent consumers without polling each.
    - Kafka advantages: horizontal scale, consumer group semantics (each consumer group gets all messages independently), message retention for replay, stream processing integration.
    - Conclusion: "For our current volume, I'd start with the Outbox + DB poller. If we exceed X events/sec or need replay/multiple consumers, then introduce Kafka."

11. **"Choreography vs orchestration — Smart360 used choreography. What's the failure scenario that would have forced you to switch to orchestration?"**
    - If the flow had grown to: user role changes → notify user → update analytics → sync to external CRM → trigger compliance audit. With choreography across 4 services, debugging a failed compliance audit that depends on the CRM sync that depends on analytics becomes a distributed event tracing nightmare. No single place to see the saga state. Orchestration with a state machine (Temporal or custom) would give visibility and allow targeted retries of individual steps.

**Behavioral / Project-Tied**

12. **"Tell me about a production bug related to messaging or async processing."**
    - Use the WebX LLM job queue: "We had workers that would pick up a job, call the LLM, get a response, then fail to write the result back to DB due to a transient connection error. The job status stayed QUEUED. The next polling cycle picked it up again and called the LLM a second time — wasted API cost and produced a duplicate response visible to the user. Fix: pessimistic update — `UPDATE jobs SET status='PROCESSING' WHERE id=? AND status='QUEUED'` returns 0 rows if already processing, preventing the duplicate call. Then wrapped the LLM call + DB update in a retry with the same guard."

13. **"Curveball: Can you implement Kafka's at-exactly-once guarantee for writing to PostgreSQL?"**
    - Strictly, no — Kafka's exactly-once transactions are Kafka→Kafka (within the broker's transaction log). For Kafka→PostgreSQL, you can approximate it: use the idempotent consumer pattern with a `deduplication_log(message_id, processed_at)` table and write to it + your target table in the same DB transaction. If the consumer commits offset after the DB transaction succeeds, you get effectively-once semantics as long as the consumer re-reads and finds the dedup record on retry. The key word is "effectively-once" not "exactly-once" — be precise.

14. **"Your resume says you integrated 5+ LLM providers in WebX. How would you design the routing so that if one provider is down, traffic automatically fails over?"**
    - Circuit breaker per provider (Resilience4j `@CircuitBreaker`). Router service checks provider health state before routing. Priority list with fallback chain: OpenAI → Anthropic → Azure OpenAI → Cohere. For async jobs: if the routed provider's circuit is open, the Kafka consumer publishes a new message to `llm-jobs` with the next provider in the fallback chain (or re-routes in-process). Track cost/latency/availability metrics per provider in Redis for intelligent routing.

15. **"Curveball: We process Kafka messages, and our consumer is very slow (takes 30s per message). What problems does this cause and how do you fix them?"**
    - Problems: (1) Heartbeat timeout — if processing takes longer than `max.poll.interval.ms` (default 5 min, but lower in practice), Kafka thinks the consumer died and triggers rebalance, your message gets reassigned and processed again. (2) Throughput bottleneck — one slow consumer blocks partition progress.
    - Fixes: increase `max.poll.interval.ms` to accommodate processing time; decrease `max.poll.records` to 1 so the consumer finishes the record before the timeout; move heavy work to a thread pool and commit offset manually after async completion; or redesign to decouple consuming from processing (consumer writes to a local queue, separate thread pool processes, commits offset after completion).

---

## 🌟 Extraordinary-Candidate Edge

**What separates good from extraordinary on this week's material:**

1. **Name trade-offs, not just features.** Weak: "Kafka is good for async." Strong: "At our WebX LLM job volume, the operational overhead of Kafka (ZooKeeper/KRaft, consumer group management, partition planning) wasn't justified. A PostgreSQL-backed queue with `SELECT FOR UPDATE SKIP LOCKED` was simpler, sufficient, and already transactional with our job table. I'd introduce Kafka if we exceeded ~50k events/day or needed replay/multiple consumer groups." Interviewers remember engineers who said "we didn't need Kafka yet."

2. **Lead with numbers from your resume in system design.** Don't say "the system was slow." Say "query latency was 60s — I'll tell you why." Don't say "we improved efficiency." Say "30% efficiency gain, measured by P95 latency on the authorization endpoint from 850ms to 590ms after removing the synchronous Notification call." Make them ask follow-up questions.

3. **Proactively mention failure modes.** When describing the Outbox pattern: "One gotcha: if the poller crashes mid-publish after writing to Kafka but before marking SENT, you get a duplicate. This is the at-least-once boundary. I document this at the architecture level so every consumer is written idempotently from day one."

4. **Show your GitHub during behavioral conversations.** "I actually implemented a minimal version of this — can I pull it up?" Even a toy project with a clean README signals you learn by building, not just reading.

5. **Graph problems: always state the pattern type first.** Before writing code: "This is a multi-source BFS problem — seed the queue with all rotten cells simultaneously. The answer is the number of BFS levels. Let me write the skeleton." This shows algorithmic thinking, not just coding.

6. **On Saga, go beyond the pattern name.** Most candidates say "choreography or orchestration." Extraordinary candidates say: "For Smart360's notification extraction, choreography was right because the Notification service was genuinely decoupled — it didn't matter if it was 500ms late processing the event. I'd reconsider if we needed to guarantee the notification was delivered before the API returned a response to the client — that's where orchestration with a wait-for-completion step would be required."

---

## 📊 End-of-Week Self-Assessment

Rate yourself 1–5 on each item after Sunday's mock session:

**DSA — Graphs**
- [ ] BFS from scratch (no reference): ___/5
- [ ] DFS from scratch (no reference): ___/5
- [ ] Topological sort (both Kahn's and DFS): ___/5
- [ ] Union-Find with path compression + rank: ___/5
- [ ] Cycle detection (directed vs undirected): ___/5
- [ ] LC 200 Number of Islands — AC in ≤15 min: ___/5
- [ ] LC 133 Clone Graph — AC in ≤20 min: ___/5
- [ ] LC 207 Course Schedule I — AC in ≤20 min: ___/5
- [ ] LC 210 Course Schedule II — AC in ≤25 min: ___/5
- [ ] LC 684 Graph Valid Tree — AC in ≤20 min: ___/5
- [ ] LC 323 Connected Components — AC in ≤15 min: ___/5
- [ ] LC 417 Pacific Atlantic — AC in ≤25 min: ___/5
- [ ] LC 994 Rotting Oranges — AC in ≤20 min: ___/5
- [ ] LC 127 Word Ladder — AC in ≤30 min: ___/5

**System Design — Messaging**
- [ ] Kafka: partitions, consumer groups, offset commit, rebalance: ___/5
- [ ] Idempotency implementation patterns: ___/5
- [ ] Outbox pattern (problem + both delivery options): ___/5
- [ ] Exactly-once vs at-least-once — honest trade-off discussion: ___/5
- [ ] Choreography Saga — when and why: ___/5
- [ ] Orchestration Saga — when and why: ___/5
- [ ] Full LLM async job system design without notes: ___/5

**Career Positioning**
- [ ] GitHub repo/project is public, README is recruiter-ready: ___/5
- [ ] Can tie Smart360 30% efficiency gain to specific technical choices: ___/5
- [ ] Can tie WebX async LLM jobs to Kafka/messaging concepts: ___/5

**Minimum to proceed to Week 7:** All DSA items ≥ 3/5, no system design item below 3/5.

**If any DSA item is 1–2/5:** Revisit that algorithm Monday morning of Week 7 before moving to new material. Graph patterns repeat in Week 7 (Dijkstra, A*, advanced DP on graphs) — gaps here compound.

---

*Week 6 of Jay's 2026 interview prep — builds on interview-qa.md. Do not duplicate Q&A already covered there.*
