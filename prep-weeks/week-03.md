# Week 3 — Foundations + Core (Jul 13–18, 2026)

> Own the two structures that separate candidates — trees and graphs — and the data-layer + distributed-systems knowledge product interviewers test next: indexing internals, caching architecture, Kafka, the Outbox/Saga patterns, and a crisp full-stack story.
>
> **Full-time, heads-down study week — Mon–Sat (6 study days), Sun rest.** Three blocks per day: **Block A** (DSA, ~2.5 hr), **Block B** (core/system knowledge, ~2 hr), **Block C** (a second DSA set or deep-dive, ~1.5 hr). No applications this week — pure skill build.

---

## 🎯 Week Goal

Solve any tree or graph LeetCode medium cold in ≤25 min — traversals, BST ops, LCA, construction, BFS/DFS, topological sort, union-find, Trie, and Dijkstra. Reason about DB index internals, the four cache write strategies, and SQL-vs-NoSQL without hand-waving. Design a production async event-driven system end-to-end — Kafka internals, idempotency/exactly-once, Outbox, and both Saga flavors — and a typeahead system, tying every answer back to Smart360's monolith→event-driven migration and WebX's async LLM jobs. Have a recruiter-ready GitHub.

---

## ✅ By Saturday you can...

- Write iterative pre/in/post-order traversals and level-order BFS (`List<List<Integer>>`) cold
- Solve Validate BST (bounds approach), Kth Smallest, Diameter, both LCA variants, and Tree Construction from pre+inorder
- Implement BFS, DFS, topological sort (Kahn's + DFS), union-find (path compression + rank), and cycle detection from memory
- Solve all the target graph problems (Islands, Clone Graph, Course Schedule I/II, Valid Tree, Connected Components, Rotting Oranges) in ≤25 min each
- Implement a Trie (insert/search/startsWith) and Dijkstra with a min-heap, and state the relaxation invariant
- Explain B-tree structure, composite/covering/partial indexes, and the leftmost-prefix rule
- Name the four cache write strategies, their failure modes, and LFU vs LRU eviction; explain Redis persistence and Cluster vs Sentinel
- Apply a SQL-vs-NoSQL decision framework (access patterns first) and name when each NoSQL family fits
- Explain Kafka partitions, consumer groups, offsets, ISR, and acks like someone who ran it in prod
- Draw the Outbox + CDC flow, compare choreography vs orchestration Saga, and explain why exactly-once-to-a-DB is a myth
- Give a sharp Vue 3 (Proxy reactivity) vs React (reconciliation/Fiber) comparison
- Design a typeahead/autocomplete system end-to-end and present a recruiter-ready GitHub profile

---

## 📅 Daily Checklist

---

### Monday Jul 13 — Tree Traversals + BST

📌 **Study today:** Tree traversals + BST (LC 94, 102, 104, 98, 230) · DB indexing (B-tree, composite, covering) · tree problems (LC 236, 235, 543, 199)

**DSA (Block A, ~2.5 hr) — Traversals + BST Core:**

Theory warm-up: draw a BST, trace all three DFS orders. In-order (L→Root→R) → sorted output on a BST; pre-order (Root→L→R) → serialize/copy; post-order (L→R→Root) → deletion/size. Recursive DFS is O(h) space (h = height; O(n) for a skewed tree).

Problems (in order):
1. **Binary Tree Inorder Traversal** (LC 94) — iterative "go-left-then-pop": `curr=root`; while curr or stack nonempty: go left until null, pop and record, go right. (Also drill iterative pre-order and the two-stack/reverse trick for post-order.)
2. **Binary Tree Level Order Traversal** (LC 102) — `ArrayDeque` queue; record `queue.size()` *before* the inner loop, pull exactly that many, enqueue children. Return `List<List<Integer>>` — clean in <15 lines.
3. **Maximum Depth of Binary Tree** (LC 104) — recursive one-liner, then iterative BFS counting levels.
4. **Validate BST** (LC 98) — DFS with `(node, min, max)` bounds (left call passes `max=node.val`, right passes `min=node.val`); start `(-∞, +∞)`. Use `long` for the `Integer.MIN/MAX_VALUE` edges. Wrong approach: checking only parent-child (misses grandparent violations).
5. **Kth Smallest Element in a BST** (LC 230) — iterative in-order with early exit at k. Follow-up: if the BST mutates often, augment each node with its left-subtree count → O(h) per query.

**Core Topic (Block B, ~2 hr) — DB Indexing:**

- **B-tree**: self-balancing, O(log n) reads, default Postgres index type, supports range queries (`>`, `<`, `BETWEEN`). Postgres uses heap storage (table and index are separate files); MySQL InnoDB uses a clustered index (the table IS a B+ tree by PK).
- **Hash index**: O(1) exact match, no range queries; rarely used.
- **Composite index `(a, b, c)`**: usable for `a`, `a+b`, `a+b+c` (leftmost-prefix rule). `(b, c)` alone won't use it — the B-tree is sorted by the leftmost key first.
- **Covering index**: contains all columns a query needs → index-only scan, never touches the heap. Fastest read. Example: `CREATE INDEX idx ON orders(tenant_id, status, created_at)` for `SELECT tenant_id, status, created_at WHERE tenant_id = ?`.
- **Partial index**: `CREATE INDEX ON orders(status) WHERE status = 'PENDING'` — tiny, fast for that filter.
- **Selectivity**: high-cardinality columns (user_id, email) make better targets than low (boolean, 2-value status).
- Tie to Smart360's 60s→2-3s win: explain which columns you indexed (join, filter, sort) and why.

**DSA (Block C, ~1.5 hr) — Tree Problems:**

1. **Lowest Common Ancestor of a Binary Tree** (LC 236, general tree) — recursive post-order: return the node if it equals p or q; if both left and right return non-null → current is LCA; else return whichever is non-null.
2. **Lowest Common Ancestor of a BST** (LC 235) — exploit the BST property: both < root → go left; both > root → go right; else root is the LCA. O(h), O(1) iteratively.
3. **Diameter of Binary Tree** (LC 543) — diameter through a node = left height + right height; track `maxDiameter` in an instance var / 1-element array during height DFS. Note: it need not pass through the root.
4. **Binary Tree Right Side View** (LC 199) — level-order BFS, take the last node of each level.

**Self-check:**
1. Write the iterative in-order skeleton from memory.
2. Explain covering index vs regular index in 2 sentences — no jargon.

---

### Tuesday Jul 14 — Tree Construction + LCA + Caching

📌 **Study today:** Tree construction + LCA + hard trees (LC 105, 124, 297) · caching strategies + Redis + LFU/LRU · SQL vs NoSQL framework

**DSA (Block A, ~2.5 hr) — Construction + Hard Trees:**

1. **Construct Binary Tree from Preorder and Inorder Traversal** (LC 105) — pre-order[0] is the root; find it in in-order → left part = left subtree, right part = right subtree. HashMap `{value → inorder_index}` for O(1) lookup (else O(n²)). Practice the signature `build(preStart, inStart, inEnd)`.
2. **Binary Tree Maximum Path Sum** (LC 124, Hard) — post-order; at each node the "gain" returned upward is `node.val + max(0, leftGain, rightGain limited to one side)`, but the answer candidate is `node.val + leftGain + rightGain`. Track a global max. Clamp negative gains to 0.
3. **Serialize and Deserialize Binary Tree** (LC 297) — pre-order with explicit null markers; deserialize by consuming the token stream recursively. The canonical "design + recursion" tree problem.

**Core Topic (Block B, ~2 hr) — Caching Strategies + Redis + Eviction:**

- **Cache-Aside (lazy loading)**: app checks cache → miss → reads DB/S3 → writes cache → returns. Failure mode: cache stampede on cold start / TTL expiry under load (mitigate with probabilistic early expiration or a per-key mutex). Your Redis S3-URL cache uses this — TTL set *below* the presigned-URL expiry (URL 15 min → Redis 13 min) to absorb clock skew.
- **Write-Through**: write cache + DB synchronously; always consistent, but write latency doubles and the cache fills with maybe-never-read data.
- **Write-Back (write-behind)**: write cache only, async flush to DB; fast writes, risk of loss if the node dies before flush (CPU L1/L2 caches use this).
- **Write-Around**: write straight to DB, populate cache only on reads; good for write-once/read-rarely data (logs, audit). Your S3-URL cache is write-around (S3 is source of truth).
- **Eviction**: `allkeys-lru` / `volatile-lru` / `allkeys-lfu` / `volatile-lfu` / `noeviction`. **LRU** evicts least-recently-used (recency); **LFU** evicts least-frequently-used (frequency) — better for non-uniform access where some keys stay hot regardless of recency.
- **Persistence**: RDB (point-in-time snapshots) vs AOF (append-only, `appendfsync everysec`). JWT blacklist → AOF (can't un-invalidate a token; 1s loss tolerable). S3-URL cache → RDB or none (it's just a cache).
- **Redis Cluster vs Sentinel**: Sentinel = HA for single-primary (auto failover); Cluster = horizontal sharding (16384 hash slots).

**DSA (Block C, ~1.5 hr) — SQL vs NoSQL Framework:**

A decision framework, not a religion:
- **Start from access patterns, not the data.** Relational schema + indexes optimize for *unknown future queries*; NoSQL optimizes for *known query patterns you design the schema around*.
- **When relational (Postgres) wins (usually):** multi-entity joins, ACID transactions across rows/tables, ad-hoc/analytical queries, strong consistency by default. Smart360 and Deep Fathom are Postgres for exactly this (RLS, 50-table joins, transactional authorization). Line to use: "I start with Postgres for ~95% of cases; I move a *slice* to NoSQL when one access pattern consistently hits relational limits — polyglot, not replacement."
- **NoSQL families:** Key-value (Redis, DynamoDB) — O(1) by key; cache, sessions, rate-limiter state, JWT blacklist. Document (MongoDB) — self-contained JSON aggregates, evolving schema, no joins. Wide-column (Cassandra, HBase) — massive write throughput, time-series, partition-key reads of known patterns (event/audit logs, metrics, IoT). Graph (Neo4j) — cheap multi-hop relationship traversal (social graphs, permission hierarchies) where SQL self-joins explode at 4+ hops.
- **Polyglot persistence:** Postgres for the transactional core, Redis for hot/ephemeral state, a wide-column/document store for one high-volume pattern; keep the seam consistent via events/outbox (Thursday).

**Self-check:**
1. Write the Tree Construction (LC 105) signature and base case without looking.
2. Pick a write strategy for a social-media "like count" and justify it.

---

### Wednesday Jul 15 — Graphs BFS/DFS + Kafka

📌 **Study today:** Graphs BFS/DFS (LC 200, 695, 133, 994) · Kafka internals (partitions/consumer groups/offsets/ISR) · topological sort (LC 207, 210)

**DSA (Block A, ~2.5 hr) — Graph Foundations:**

Implement adjacency list (`Map<Integer, List<Integer>>` for sparse) vs matrix (`int[][]` for dense); iterative BFS (queue), recursive + iterative DFS (stack). BFS = shortest path on unweighted graphs / level order; DFS = cycle detection, topo sort, less memory on deep graphs.

Problems (in order):
1. **Number of Islands** (LC 200) — grid = implicit graph; 4-directional neighbors; mark visited in-place (`'1'→'0'`) or with `boolean[][]`. O(m×n). Target AC in 15 min. Edge cases: all water, single cell, non-square.
2. **Max Area of Island** (LC 695) — same flood-fill, return the max component size; DFS returns `1 + sum of neighbor areas`.
3. **Clone Graph** (LC 133) — DFS + `HashMap<original, clone>` to handle cycles; rewire adjacency, don't just copy values. Target AC in 20 min.
4. **Rotting Oranges** (LC 994) — multi-source BFS: seed the queue with ALL rotten cells; BFS levels = minutes. After BFS, if any fresh remains → -1. Edge cases: no fresh → 0.

**Core Topic (Block B, ~2 hr) — Kafka Internals:**

- **Topic / partition**: a topic is split into ordered, immutable partitions. The **partition key** decides placement — same key → same partition → per-key ordering; null key → round-robin.
- **Consumer group**: each partition → exactly one consumer in the group; scale consumers up to partition count (extra consumers idle). 2 partitions + 3 consumers → 1 idle; 4 partitions + 3 → one reads 2.
- **Offset**: monotonic position in a partition; the consumer commits offset to track progress; crash recovery = re-read from last committed offset (→ at-least-once).
- **Replication & ISR**: each partition has a leader + follower replicas on different brokers; ISR = replicas caught up to the leader. `acks=all` + `min.insync.replicas=2` prevents data loss; `acks=1` is dangerous (leader can die before replication).
- **Rebalance**: triggered by a consumer joining/leaving, partition-count change, or session timeout; the `group.coordinator` reassigns partitions.
- **`auto.offset.reset`**: `earliest` vs `latest` — matters on first startup / when no committed offset exists.
- **Retention**: Kafka keeps messages for a configured time (default 7 days) regardless of consumption → replay; RabbitMQ deletes on ACK.
- Tie to Smart360: "I'd set `acks=all` for authorization events (correctness-critical) and `acks=1` for notification events (latency-sensitive, idempotent consumer anyway)."

**DSA (Block C, ~1.5 hr) — Topological Sort:**

1. **Course Schedule** (LC 207) — can finish? = no cycle in a directed graph. Kahn's (BFS + in-degree array; cycle iff processed < V) OR DFS 3-color (0=unvisited, 1=in-path→cycle, 2=done). Target AC in 20 min.
2. **Course Schedule II** (LC 210) — produce the actual order: Kahn's output array, or reverse of DFS post-order. Target AC in 25 min. Edge cases: disconnected components, self-loops, empty prerequisites.

**Self-check:**
1. BFS vs DFS trade-offs in 60 seconds.
2. Why does Kahn's detect a cycle? (Processed count < V → cycle.)

---

### Thursday Jul 16 — Union-Find + Outbox/Saga + Trie

📌 **Study today:** Union-Find + advanced graphs (LC 684, 547) · Outbox + Saga + idempotency/exactly-once · Trie (LC 208, 211, 1268)

**DSA (Block A, ~2.5 hr) — Union-Find:**

Implement with **path compression** + **union by rank** until it's muscle memory:
```java
int[] parent, rank;
int find(int x) { return parent[x] == x ? x : (parent[x] = find(parent[x])); }
void union(int x, int y) {
    int px = find(x), py = find(y);
    if (px == py) return;
    if (rank[px] < rank[py]) parent[px] = py;
    else if (rank[px] > rank[py]) parent[py] = px;
    else { parent[py] = px; rank[px]++; }
}
```
Amortized near-O(1) per op (inverse Ackermann α(n)).
1. **Redundant Connection** (LC 684) — find the edge that creates a cycle: if `union(u,v)` finds them already connected → that edge is the answer. Target AC in 20 min.
2. **Number of Provinces** (LC 547) — count connected components: count roots (`find(i)==i`) after all unions, or DFS-count entry points. Target AC in 15 min.

**Core Topic (Block B, ~2 hr) — Outbox + Saga + Idempotency/Exactly-Once:**

- **Idempotency / exactly-once vs at-least-once**: an idempotent consumer produces the same result processing a message N times as once. Implement: unique `messageId` in payload → check `processed_messages(message_id)` before processing → skip if seen. Kafka **EOS** (`enable.idempotence=true` + `transactional.id` + consumer `isolation.level=read_committed`) is Kafka→Kafka only. Exactly-once *to an external DB is a myth* — the DB write and offset commit are two systems; make the consumer idempotent. **At-least-once is the real-world default.** Patterns: message-level (`processed_messages` table), business-level (`IF status='QUEUED' THEN process`), optimistic lock (`UPDATE jobs SET status='PROCESSING' WHERE id=? AND status='QUEUED'` — only one consumer wins).
- **Outbox pattern**: solves the dual-write problem (save entity + publish event = two I/O ops; crash between = lost event). Schema: `outbox_events(id UUID, aggregate_type, aggregate_id, event_type, payload JSONB, status, created_at)` — written in the *same transaction* as the domain change. Delivery: (1) polling publisher (`@Scheduled` reads `status='PENDING'`, publishes, marks `SENT` — simple, adds DB load/latency); (2) Debezium CDC (tails the Postgres WAL → Kafka — near real-time, no polling, more ops). At-least-once → consumer must be idempotent. Index `(status, created_at)` for the poll query. Smart360 fixed a real dual-write bug (lost authorization events on pod restart) this way.
- **Saga**: **Choreography** — no coordinator; each service publishes events others react to. Pros: loose coupling, no SPOF; cons: hard to visualize, distributed debugging. Use for ≤3 stable services you own. **Orchestration** — a coordinator (Temporal, Step Functions, custom state machine) sends commands and awaits replies. Pros: full visibility, central rollback; cons: orchestrator is a bottleneck/SPOF. Use for ≥4 services, external systems (payment), or audit/compliance needs. **Compensating transactions**: each step needs a defined undo (you can't un-capture a payment — you issue a refund) — design them before the forward path. Smart360 used choreography for notification extraction (2 services, no complex rollback).

**DSA (Block C, ~1.5 hr) — Trie:**

A Trie is a graph specialized for prefix queries: each node has children keyed by character + an `isEnd` flag. Your search-product home turf (autocomplete/typeahead).
1. **Implement Trie (Prefix Tree)** (LC 208) — node = `TrieNode[26] children` + `boolean isEnd`; insert/search/startsWith walk char by char; `search` needs `isEnd` at the terminal, `startsWith` doesn't. Target AC in 15 min.
2. **Design Add and Search Words Data Structure** (LC 211) — trie + DFS for the `.` wildcard: `.` recurses into ALL children; any other char follows the single match. Target AC in 20 min.
3. **Search Suggestions System** (LC 1268) — at each prefix node, DFS to collect ≤3 lexicographically smallest completions (or sort products + binary-search the prefix lower bound). Target AC in 25 min. Talking point: "the in-memory core of autocomplete; in prod I'd back it with a trie/FST (Lucene-style) and bound the fan-out."

**Self-check:**
1. Amortized complexity of Union-Find with path compression + rank? (≈O(1), α(n).)
2. What is Debezium and why is it better than polling for high-throughput outbox?

---

### Friday Jul 17 — Dijkstra/Weighted Graphs + Frontend + Typeahead

📌 **Study today:** Dijkstra / weighted graphs (LC 743, 787, 1631) · frontend refresh (Vue 3 reactivity, React hooks/reconciliation) · typeahead/autocomplete system design

**DSA (Block A, ~2.5 hr) — Dijkstra / Weighted Graphs:**

Plain BFS breaks with edge weights — use Dijkstra. **Relaxation invariant:** once a node is popped from a min-heap of `(dist, node)`, its shortest distance is final — any other path goes through a not-yet-finalized node with strictly larger tentative distance. The min-heap guarantees you always finalize the closest unsettled node next. Caveat: non-negative weights only (negatives → Bellman-Ford).
1. **Network Delay Time** (LC 743) — canonical Dijkstra: `PriorityQueue<int[]{dist,node}>`, pop smallest, skip if finalized, relax neighbors; answer = max finalized distance (or -1 if any node unreachable). Target AC in 25 min.
2. **Cheapest Flights Within K Stops** (LC 787) — plain Dijkstra is wrong (cheapest path may exceed allowed stops); do **k+1 rounds of Bellman-Ford** over a distance snapshot, or level-capped BFS tracking `(node, stops)`. Target AC in 25 min.
3. **Path With Minimum Effort** (LC 1631, Hard) — Dijkstra on a grid where path cost = MAX abs height-diff along it (bottleneck path); relax with `effort = max(currentEffort, |h[next]-h[cur]|)`. Mention union-find-with-sorted-edges and binary-search-on-answer + BFS as alternatives. Target AC in 30 min.

**Core Topic (Block B, ~2 hr) — Frontend Refresh (Vue 3 + React):**

- **Vue 3 reactivity**: Vue 2 used `Object.defineProperty` (couldn't detect property add/delete or array index changes). Vue 3 uses ES6 `Proxy` → intercepts get/set/deleteProperty/has. `ref(value)` wraps primitives in `{ value }` (auto-unwrapped in templates); `reactive(object)` returns a Proxy — destructuring breaks reactivity (use `toRefs`). `computed(() => ...)` is lazy + cached. Composition API (`<script setup>`) groups logic by feature; composables (`useAuth()`) replace mixins. Lifecycle: `onMounted`/`onUpdated`/`onUnmounted`; `watchEffect` (auto-tracks, runs immediately) vs `watch` (explicit, lazy, old+new). **Pinia**: `defineStore('id', { state, getters, actions })`; getters are cached like `computed`; mutate state directly in actions; `storeToRefs` to destructure with reactivity.
- **React**: `useState` (async/batched setter; same value → no re-render). `useEffect(fn, deps)`: `[]` once after mount; `[a,b]` on change; no array = every render (usually a bug); return a cleanup. Missing deps → stale closure (ESLint `exhaustive-deps`). `useCallback`/`useMemo` memoize a function/value; `useRef` = mutable non-rendering ref; `useContext` re-renders all consumers (split contexts); `useReducer` for complex transitions. **Reconciliation**: React builds a new VDOM, diffs against the previous, commits changes. **Fiber** = the incremental reconciler; React 18 Concurrent Mode makes rendering interruptible. `key` in lists: stable keys → O(1) identity (never array index if reorderable). `React.memo` + `useCallback` skips re-render on unchanged props.
- **Vue vs React (the comparison you'll be asked)**: Vue = fine-grained Proxy reactivity + compiler marks static nodes (no manual memo); React = `useState` triggers component+children re-render, mitigated by `memo`/`useMemo`/`useCallback`. Vue SFC separates HTML/CSS/JS; React uses JSX. Pinia/Vue Router are first-party; React's ecosystem is larger but fragmented. "The compiler knows at build time which template parts are dynamic; React diffs the whole virtual tree at runtime — Vue is faster for update-heavy UIs but React's ecosystem and concurrent features dominate at scale."

**DSA (Block C, ~1.5 hr) — Typeahead / Autocomplete System Design:**

Turn the Trie (LC 208/211/1268) into a real HLD. The interviewer wants the system around the trie.
- **Scale + latency first**: read-heavy, billions of queries; suggestions must return in **< 100 ms p99** (renders on every keystroke).
- **Serving path (online)**: client (debounced) → API/edge → **sharded suggestion index** in memory. Trie/FST too big for one box → **shard by prefix** (first 1–2 chars, or consistent-hash). Each node stores **precomputed top-k** so a lookup is walk-to-prefix-node-then-read-top-k, not a request-time DFS.
- **Top-k ranking**: rank by **historical query frequency** (popularity), not lexicographic — "fac" surfaces "facebook" before "facsimile". Optionally blend recency/personalization later.
- **Offline pipeline (candidates forget this)**: query logs → stream/batch aggregation (Kafka → Spark/Flink or nightly batch) → per-prefix frequency counts → build/refresh the trie/FST → ship to serving shards via a versioned blob swap (atomic pointer flip, never mutate in place). Refresh hourly/daily.
- **Cache hot prefixes**: short prefixes ("a", "fa") dominate → Redis/edge cache keyed by prefix → top-k, TTL minutes (the cache-aside + hot-key reasoning from the caching block).

**Self-check:**
1. Why does Dijkstra finalize a node's distance the moment it's popped?
2. Why does destructuring a `reactive()` object break reactivity, and how do you fix it? (Proxy is dropped on extract; use `toRefs`.)

---

### Saturday Jul 18 — Timed DSA + GitHub Polish + Consolidation

📌 **Study today:** Timed DSA set (graphs/trees) · GitHub polish + showcase repo · consolidation + weak spots

**DSA (Block A, ~2.5 hr) — Timed Set (graphs/trees, no hints):**

Real-interview conditions — think aloud, handle edge cases, state complexity.
- **Problem A** (25 min): **Course Schedule** (LC 207) cold.
- **Problem B** (25 min): **Rotting Oranges** (LC 994) cold.
- Then re-solve the 2 tree/graph problems you found hardest this week (15 min each). Debrief: where did you hesitate or say something wrong under pressure? Write down time/space complexity for every problem from memory.

**Core Topic (Block B, ~2 hr) — GitHub Polish + Showcase Repo:**

Pick ONE build:
- **Option A — `smart360-event-demo`**: minimal Spring Boot app showing the Outbox pattern — POST `/users` writes user + outbox event in one transaction; `@Scheduled` publisher reads outbox and logs "would publish to Kafka"; GET `/events` shows the table; small Vue/React frontend for status.
- **Option B — clean up an existing repo**: proper README, architecture diagram (Mermaid/ASCII), tech-stack badges, `docker-compose up`, one screenshot; remove TODOs and compile warnings.

README must include: one-paragraph "What problem does this solve?" (recruiter-targeted), an architecture diagram, `docker-compose up` runnable in <5 commands, and links to relevant resume bullets (Smart360/WebX/Deep Fathom).

Profile-polish checklist (a recruiter lands here first):
- Pin 3–6 best repos (quality over quantity — the only repos most recruiters open).
- Add a profile README (`github.com/<user>/<user>`) — 4–5 lines: who you are, the cloud-native + LLM narrative, current focus, contact/LinkedIn.
- Prune/archive embarrassing repos (old coursework, half-finished experiments, no-README repos).
- One showcase repo aligned to the cloud-native + LLM narrative (Spring Boot + an LLM-proxy or IaC sample) with a clean README + diagram — the repo you screen-share in interviews.
- Tidy commit messages on the showcase repo (no `wip`/`fix`/`asdf` — imperative, scoped: `feat: add provider failover circuit breaker`).

**DSA (Block C, ~1.5 hr) — Consolidation + Weak Spots:**

- Write 3 bullets per graph algorithm: when to use, the pattern, one gotcha.
- State the pattern for each graph problem in one sentence (Islands → flood fill; Clone Graph → DFS + node map; Course Schedule → topo/cycle; Valid Tree / Components → union-find; Rotting Oranges → multi-source BFS; weighted → Dijkstra).
- Write the Kafka delivery-guarantee table from memory:

| Guarantee | Producer Config | Consumer Design | Trade-off |
|---|---|---|---|
| At-most-once | acks=0 | auto-commit before process | Fast, can lose messages |
| At-least-once | acks=all | commit after process | Safe, must handle duplicates |
| Exactly-once | idempotent producer + transactions | read_committed | Complex, Kafka→Kafka only |

- Write your top 3 weak areas from the week and one concrete drill for each.

**Self-check:**
1. Look at your pushed showcase repo as a stranger — would you click "Star"?
2. Can you walk the full async LLM job system design (Client → API → DB jobs+outbox → Debezium/scheduler → Kafka → worker pool → LLM → DB → SSE) in 10 minutes without notes?

---

### Sunday Jul 19 — Rest

No study blocks. Let trees, graphs, and the distributed-systems patterns consolidate. Optionally skim the self-check answers you got wrong — but do not start new material.

---

## 🧠 Concepts to Master This Week

### Trees
| Concept | Key Insight | Common Mistake |
|---|---|---|
| Iterative In-Order | Stack: go left until null, pop, record, go right | Forgetting the go-right step after pop |
| Iterative Post-Order | Reverse of (Root→R→L) pre-order | Trying it with one stack (use two) |
| Level Order | Record `queue.size()` before the inner loop | Reading size inside the loop |
| Validate BST | Pass `(min, max)` bounds down | Checking only parent-child |
| LCA — BST | both<root→left; both>root→right; else root | Treating it as a general tree |
| LCA — General Tree | Post-order: both sides non-null → current | Confusing it with BST LCA |
| Tree Construction | pre-order[0]=root; split in-order; recurse | O(n²) lookup (fix with HashMap) |
| Diameter | max(left height + right height) during height DFS | Assuming it passes through root |
| Kth Smallest | In-order, count k | Not knowing the augmented-BST follow-up |

### Caching
| Strategy | Write | Read | Best For | Risk |
|---|---|---|---|---|
| Cache-Aside | DB direct; cache on read miss | check cache → miss → DB → fill | Read-heavy, tolerate stale | Stampede on miss |
| Write-Through | cache + DB sync | always cache | Consistency-critical | Double write latency |
| Write-Back | cache only; async flush | always cache | Write-heavy, latency-sensitive | Loss if cache dies |
| Write-Around | DB direct, bypass cache | cache on reads only | Write-once/read-rarely | High first-read latency |

### DB Indexing
- B-tree (default, O(log n), range queries) vs hash (O(1) exact, no ranges).
- Composite `(a,b,c)`: leftmost-prefix rule. Covering: index-only scan. Partial: filtered, tiny. Selectivity: prefer high-cardinality columns.
- Production: `CREATE INDEX CONCURRENTLY` on large tables (weaker lock, allows reads/writes); `EXPLAIN (ANALYZE, BUFFERS)` to verify.

### Graph Algorithms
| Algorithm | Pattern | Time | Space | Use Case |
|---|---|---|---|---|
| BFS | Queue, visited, level-by-level | O(V+E) | O(V) | Shortest path unweighted, multi-source |
| DFS | Call/explicit stack, visited | O(V+E) | O(V) | Flood fill, cycle, topo |
| Kahn's Topo | In-degree + BFS | O(V+E) | O(V) | Course Schedule (cycle if processed<V) |
| DFS Topo | Post-order + reverse | O(V+E) | O(V) | Course Schedule II |
| Union-Find | parent/rank, path compress | O(α(n)) | O(V) | Components, undirected cycle, MST |
| Multi-source BFS | Seed multiple starts | O(V+E) | O(V) | Rotting Oranges |
| Dijkstra | Min-heap, relaxation | O(E log V) | O(V) | Weighted shortest path (non-negative) |

### Messaging & Async
- **Kafka**: topic→partitions; partition key → per-key ordering; consumer group (one consumer per partition); offset commit; ISR + `acks=all`+`min.insync.replicas=2`; retention enables replay (RabbitMQ deletes on ACK).
- **Outbox**: domain table + `outbox_events` in one tx; polling publisher or Debezium CDC; at-least-once → idempotent consumer; index `(status, created_at)`.
- **Idempotency**: message-level (`processed_messages`), business-level state machine, optimistic lock.
- **Saga**: choreography (≤3 services, you own all consumers) vs orchestration (≥4 services, external systems, audit). Design compensating transactions before the forward path.

### Vue 3 vs React
| Aspect | Vue 3 | React 18 |
|---|---|---|
| Reactivity | Proxy-based, fine-grained | useState → full component re-render |
| Optimization | Compiler marks static nodes | `memo`/`useMemo`/`useCallback` |
| State | Pinia (first-party) | Redux Toolkit / Zustand / React Query |
| Templates | SFC (`.vue`) | JSX |
| Ecosystem | Smaller, cohesive | Massive, fragmented |

---

## 🎤 Sample Interview Questions (incl. curveballs)

1. **Walk through all four traversal orders and a real use of each.** → In-order: sorted BST output/range queries; pre-order: serialize/copy, expression trees; post-order: bottom-up deletion, size/disk-usage; level-order: BFS, shortest path unweighted.
2. **Validate BST — why isn't checking left<root<right enough?** → Misses grandparent violations; pass `(min,max)` bounds down; use `long` for the int edges.
3. **LCA on a general tree where one target IS the LCA?** → The recursion returns the node on match; the sibling branch returns null; the parent gets that node and propagates it up — correct without stopping at the match.
4. **Leftmost-prefix rule with a concrete example.** → Index `(tenant_id, status, created_at)` serves `WHERE tenant_id=?`, `tenant_id+status`, all three — but not `status` alone (skips the leading key).
5. **Add an index to a 100M-row prod table without downtime.** → `CREATE INDEX CONCURRENTLY` (weaker lock, allows reads/writes; can leave an INVALID index on failure — drop and retry); off-peak; monitor `pg_stat_progress_create_index`.
6. **LRU vs LFU — when each?** → LRU for temporal locality; LFU for non-uniform access where some keys stay hot regardless of recency (`allkeys-lru` general; `allkeys-lfu` for persistent hot items).
7. **Kafka consumer crashes mid-processing (received, not committed) — what happens?** → Coordinator detects the missed heartbeat, rebalances, reassigns the partition; the new consumer re-reads from the last committed offset → reprocessing → at-least-once → consumer must be idempotent.
8. **Detect a cycle in directed vs undirected graphs?** → Directed: DFS 3-color or Kahn's (processed<V); undirected: DFS tracking parent, or union-find.
9. **(Curveball) Number of Islands on a 100,000 × 100,000 grid?** → Recursive DFS overflows; switch to iterative BFS/DFS; distributed → partition + union-find across boundaries.
10. **(Curveball) Outbox poller publishes to Kafka, then the DB dies before marking SENT?** → Row stays PENDING → republished next cycle → duplicate → idempotent consumer. No atomicity across DB+Kafka without a distributed transaction; accept at-least-once.
11. **Idempotent producer vs idempotent consumer?** → Producer (`enable.idempotence=true`): broker dedups retried writes via a per-partition sequence number. Consumer: your code dedups effects on the downstream DB. Independent guarantees.
12. **(Curveball) "Just use a bigger DB instead of Kafka for our event queue."** → For low volume a DB-backed queue (`SKIP LOCKED`) works (that's Outbox); at scale, DB polling loads the primary, no replay, no fan-out to independent consumers. Start simple, introduce Kafka past a threshold.
13. **Choreography → orchestration — what forces the switch?** → Growing to 4+ services with cross-dependencies (compliance audit ← CRM sync ← analytics) makes pure-event debugging a tracing nightmare; orchestration gives saga-state visibility and targeted retries.
14. **(Curveball) Vue 3 — why does destructuring `reactive()` break reactivity?** → It returns a Proxy; destructuring extracts the raw value and drops the Proxy; fix with `toRefs`.
15. **(Curveball) React `useEffect(() => fetchData(), [userId])` and userId changes mid-flight?** → Cleanup runs and the effect re-runs; the stale fetch may overwrite the new data — guard with an `isCancelled` flag or `AbortController` in cleanup.

---

## 🌟 Extraordinary-Candidate Edge

1. **Name trade-offs, not features.** "At our WebX volume, Kafka's ops overhead wasn't justified — a Postgres `SELECT FOR UPDATE SKIP LOCKED` queue was simpler and already transactional with the job table. I'd introduce Kafka past ~50k events/day or when we needed replay/multiple consumer groups."
2. **Lead with resume numbers in design.** Not "the system was slow" — "query latency was 60s, here's why" and "30% efficiency gain, P95 850ms → 590ms after removing the synchronous Notification call."
3. **Proactively name failure modes.** Outbox: "if the poller crashes after the Kafka write but before marking SENT, you get a duplicate — the at-least-once boundary, which is why every consumer is idempotent from day one."
4. **Indexing depth.** "I ran `EXPLAIN (ANALYZE, BUFFERS)` — the index pages were near 100% cache hit after warm-up because the index fit in `shared_buffers`, so app-layer caching for that path was unnecessary."
5. **Caching specifics.** "I set the S3-URL TTL to 13 min against a 15-min expiry — the buffer absorbs clock skew and serve latency; a 'valid' cached URL that expired before the client used it is worse than a shorter TTL."
6. **Saga beyond the name.** "Choreography was right for the notification flow because Notification was genuinely decoupled — being 500ms late didn't matter. I'd switch to orchestration only if the notification had to be confirmed before the API returned."

---

## 📊 End-of-Week Self-Assessment

Rate yourself 1–5. Anything below 3 is a mandatory revisit before Week 4.

| Skill | Target | Your Score | Notes |
|---|---|---|---|
| Iterative tree traversals (all 3) | 5 | | |
| Level-order BFS (clean `List<List<Integer>>`) | 5 | | |
| Validate BST (bounds approach) | 4 | | |
| LCA — BST (iterative) | 5 | | |
| LCA — general tree (recursive) | 4 | | |
| Tree construction from pre+inorder | 4 | | |
| Diameter / Kth Smallest (+ follow-ups) | 4 | | |
| BFS / DFS from scratch | 5 | | |
| Topological sort (Kahn's + DFS) | 4 | | |
| Union-Find (path compression + rank) | 4 | | |
| Cycle detection (directed vs undirected) | 4 | | |
| Trie (insert/search/startsWith + wildcard) | 4 | | |
| Dijkstra with min-heap + relaxation invariant | 4 | | |
| LC 200 / 207 / 994 — AC within target time | 4 | | |
| B-tree + composite/covering index, leftmost prefix | 4 | | |
| Cache strategies (all 4) + LFU vs LRU | 5 | | |
| Redis eviction/persistence + Cluster vs Sentinel | 4 | | |
| SQL vs NoSQL decision framework | 4 | | |
| Kafka: partitions, consumer groups, offsets, ISR, acks | 4 | | |
| Idempotency patterns + exactly-once honesty | 4 | | |
| Outbox pattern (problem + both delivery options) | 4 | | |
| Saga: choreography vs orchestration | 4 | | |
| Full async LLM job system design without notes | 4 | | |
| Typeahead/autocomplete HLD | 4 | | |
| Vue 3 reactivity vs React reconciliation (spoken) | 4 | | |
| GitHub profile/showcase repo recruiter-ready | 4 | | |

**Score interpretation:**
- 4–5 on all DSA items and no system-design item below 3: ready for Week 4.
- 3 on 1–2 items: revisit in Week 4's daily review slot.
- Below 3 on any DSA item: re-drill it Monday of Week 4 before new material — tree/graph patterns compound.

**Bonus check:** Can you tie Smart360's 30% efficiency gain and WebX's async LLM jobs to specific Kafka/Outbox/caching/indexing choices? If yes on 80%+ of rows, you are an extraordinary candidate, not just a prepared one.

---

*Week 3 of 12 — next: Week 4 (Mon Jul 20 – Sat Jul 25).*
