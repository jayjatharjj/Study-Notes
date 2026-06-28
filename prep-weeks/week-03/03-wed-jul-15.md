# Week 3 · Day 3 — Wed Jul 15 — Graphs BFS/DFS + Kafka Internals

> Graphs are where most candidates either shine or stall — but the secret is that 70% of "graph" interview problems are a *grid* in disguise, and a grid is just a graph whose edges are implicit (up/down/left/right). Master flood-fill BFS/DFS on a grid, the visited-set discipline, and topological sort, and the medium tier falls. On the systems side, **Kafka** is the distributed-systems topic backend interviewers probe hardest. The answer that scores talks about *partitions, consumer groups, offsets, ISR, and acks* like someone who ran it in prod — and is *honest* that exactly-once-to-a-database is a myth.

📌 **Study today:** Graphs BFS/DFS — islands, max-area, clone graph, rotting oranges (LC 200, 695, 133, 994) · topological sort — Course Schedule I/II (LC 207, 210) · Kafka internals (partitions, consumer groups, offsets, ISR, acks=all, exactly-once) · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Graph Foundations (BFS / DFS) (~2.5 hr)

### Theory deep-dive

A graph is `V` vertices + `E` edges. Two representations, chosen by density:

| Representation | Java type | Space | Best for |
|---|---|---|---|
| **Adjacency list** | `Map<Integer, List<Integer>>` or `List<List<Integer>>` | O(V + E) | **sparse** graphs (the usual case) |
| **Adjacency matrix** | `int[][]` / `boolean[][]` | O(V²) | **dense** graphs, O(1) edge existence checks |

A **grid is an implicit graph**: each cell is a vertex; its 4 (or 8) neighbors are edges. You never build an adjacency list — you compute neighbors on the fly with direction deltas.

**The two traversals, and when each:**

| | BFS | DFS |
|---|---|---|
| Structure | explicit **queue** | recursion (call stack) or explicit **stack** |
| Order | level by level (nearest first) | as deep as possible, then backtrack |
| Killer app | **shortest path on unweighted graphs**, multi-source spread | cycle detection, topological sort, flood-fill, connected components |
| Memory risk | O(width) — wide bottom level | O(depth) — deep/skewed graph can stack-overflow recursively |

Both are **O(V + E)** time and **O(V)** space. The non-negotiable discipline: a **visited set** (or in-place marking). Without it, any cycle loops forever — this is *the* most common graph bug.

```java
// Direction deltas for 4-connected grid neighbors (reused in every grid problem):
private static final int[][] DIRS = {{1,0},{-1,0},{0,1},{0,-1}};
```

### Worked example — LC 200 Number of Islands (both BFS and DFS)

The pattern: scan every cell; when you hit unvisited land, **flood-fill** the whole component (marking it visited), and count one island.

```java
// DFS, in-place marking. O(m*n) time, O(m*n) worst-case recursion depth.
public int numIslands(char[][] grid) {
    int count = 0;
    for (int r = 0; r < grid.length; r++)
        for (int c = 0; c < grid[0].length; c++)
            if (grid[r][c] == '1') { count++; sink(grid, r, c); }
    return count;
}
private void sink(char[][] grid, int r, int c) {
    if (r < 0 || c < 0 || r >= grid.length || c >= grid[0].length || grid[r][c] != '1') return;
    grid[r][c] = '0';                       // mark visited in-place (no separate set)
    for (int[] d : DIRS) sink(grid, r + d[0], c + d[1]);
}
```

For very large grids, prefer **iterative BFS** to avoid recursion stack overflow:

```java
private void bfsSink(char[][] grid, int sr, int sc) {
    Queue<int[]> q = new ArrayDeque<>();
    grid[sr][sc] = '0';
    q.offer(new int[]{sr, sc});
    while (!q.isEmpty()) {
        int[] cell = q.poll();
        for (int[] d : DIRS) {
            int r = cell[0] + d[0], c = cell[1] + d[1];
            if (r >= 0 && c >= 0 && r < grid.length && c < grid[0].length && grid[r][c] == '1') {
                grid[r][c] = '0';           // mark on ENQUEUE, not dequeue (avoids double-add)
                q.offer(new int[]{r, c});
            }
        }
    }
}
```

**The load-bearing detail:** mark a cell visited **when you enqueue it**, not when you dequeue it — otherwise the same cell gets added by multiple neighbors before it's processed, blowing up the queue and re-counting.

**Complexity:** O(m·n) time, O(m·n) space. Target: AC in 15 min.

### Practice set

**1. LC 200 — Number of Islands** (Medium)
- **Approach:** scan; on unvisited land, flood-fill the component and increment the count. DFS in-place mark, or iterative BFS for huge grids.
- **Key insight:** marking in-place (`'1' → '0'`) *is* your visited set — no extra `boolean[][]` needed. Mark on enqueue in BFS.
- **Complexity:** O(m·n) / O(m·n). **Edge cases:** all water, single cell, non-square grid. **Follow-up:** a 100k×100k grid → recursive DFS overflows; use iterative BFS, or partition + union-find across tile boundaries.

**2. LC 695 — Max Area of Island** (Medium)
- **Approach:** same flood-fill, but DFS *returns* the component size: `1 + sum of the four neighbor calls`. Track the max across all starts.
```java
public int maxAreaOfIsland(int[][] grid) {
    int best = 0;
    for (int r = 0; r < grid.length; r++)
        for (int c = 0; c < grid[0].length; c++)
            if (grid[r][c] == 1) best = Math.max(best, area(grid, r, c));
    return best;
}
private int area(int[][] g, int r, int c) {
    if (r < 0 || c < 0 || r >= g.length || c >= g[0].length || g[r][c] != 1) return 0;
    g[r][c] = 0;
    int sum = 1;
    for (int[] d : DIRS) sum += area(g, r + d[0], c + d[1]);
    return sum;
}
```
- **Key insight:** identical skeleton to LC 200; the only change is *returning and summing* sizes instead of just sinking — the post-order "combine children's answers" shape on a grid.
- **Complexity:** O(m·n) / O(m·n).

**3. LC 133 — Clone Graph** (Medium)
- **Approach:** DFS (or BFS) with a `HashMap<original, clone>` to handle cycles. On visiting a node, create its clone, store it in the map *before* recursing into neighbors (so a back-edge finds the in-progress clone), then rewire cloned neighbors.
```java
private Map<Node, Node> seen = new HashMap<>();
public Node cloneGraph(Node node) {
    if (node == null) return null;
    if (seen.containsKey(node)) return seen.get(node);   // already cloned -> break the cycle
    Node copy = new Node(node.val);
    seen.put(node, copy);                                 // register BEFORE recursing
    for (Node nb : node.neighbors) copy.neighbors.add(cloneGraph(nb));
    return copy;
}
```
- **Key insight:** the map serves double duty — it's the visited set *and* the original→clone lookup. Registering the clone before recursing is what stops infinite recursion on cycles.
- **Complexity:** O(V + E) / O(V). Target: AC in 20 min.

**4. LC 994 — Rotting Oranges** (Medium)
- **Approach:** **multi-source BFS**. Seed the queue with *all* initially-rotten cells, then BFS in waves — each wave is one minute. After BFS, if any fresh orange remains, return -1.
```java
public int orangesRotting(int[][] grid) {
    Queue<int[]> q = new ArrayDeque<>();
    int fresh = 0;
    for (int r = 0; r < grid.length; r++)
        for (int c = 0; c < grid[0].length; c++) {
            if (grid[r][c] == 2) q.offer(new int[]{r, c});   // seed ALL rotten
            else if (grid[r][c] == 1) fresh++;
        }
    int minutes = 0;
    while (!q.isEmpty() && fresh > 0) {
        minutes++;
        for (int i = q.size(); i > 0; i--) {                 // process one full wave
            int[] cell = q.poll();
            for (int[] d : DIRS) {
                int r = cell[0] + d[0], c = cell[1] + d[1];
                if (r >= 0 && c >= 0 && r < grid.length && c < grid[0].length && grid[r][c] == 1) {
                    grid[r][c] = 2; fresh--;
                    q.offer(new int[]{r, c});
                }
            }
        }
    }
    return fresh == 0 ? minutes : -1;
}
```
- **Key insight:** seeding *all* sources at once means the BFS frontier expands from every rotten orange simultaneously — the wave count is the answer. Snapshot `q.size()` per wave (the LC 102 trick). Count `fresh` so you can both terminate early and detect the unreachable case.
- **Complexity:** O(m·n) / O(m·n). **Edge cases:** no fresh oranges → 0; fresh but unreachable → -1.

---

## Block B — Kafka Internals (~2 hr)

> Cross-ref: `../../06-concurrency-and-collections.md` (a consumer group is a thread-pool-like work distribution — partitions are the unit of parallelism, exactly as threads are bounded by the pool size). `../../08-modern-java.md` (Spring's `@KafkaListener` and the producer/consumer config map directly to these knobs).

### The mental model

Kafka is a **distributed, partitioned, replicated, append-only commit log**. Producers append; consumers read at their own pace by *position*; nothing is deleted on read. Everything below is a consequence of "it's a log, sharded for parallelism and replicated for durability."

### Topics and partitions

A **topic** is a named stream, split into ordered, immutable **partitions** — the unit of parallelism *and* the unit of ordering. **Ordering is guaranteed only *within* a partition**, never across partitions. The **partition key** decides placement: `hash(key) % numPartitions` → same key always lands in the same partition → **per-key ordering**. A null key → round-robin across partitions (no ordering guarantee). So if you need all events for one `tenant_id` in order, use `tenant_id` as the key.

### Consumer groups — the parallelism model

A **consumer group** is a set of consumers that share the work of a topic. The rule: **each partition is consumed by exactly one consumer in the group**. Consequences:
- You scale consumers *up to the partition count*; extra consumers sit **idle**.
- 2 partitions + 3 consumers → 1 idle. 4 partitions + 3 consumers → one consumer reads 2 partitions.
- **Partition count is the ceiling on parallelism** — pick it deliberately at topic creation; increasing it later breaks key-ordering for in-flight keys.
- Multiple *different* groups each get the *full* stream independently (fan-out to independent subscribers — the thing a DB queue can't do cleanly).

### Offsets — tracking progress

An **offset** is a monotonically increasing position within a partition. The consumer **commits** its offset to record progress. On crash/rebalance, a consumer **resumes from the last committed offset** — which is exactly why the default is **at-least-once**: if you process a message and crash *before* committing, the replacement re-reads and reprocesses it.
- `enable.auto.commit=true` auto-commits periodically (simple, but can commit *before* you finish processing → at-most-once risk). Manual commit *after* processing → at-least-once.
- `auto.offset.reset` (`earliest` vs `latest`) decides where to start **only when there's no committed offset** (first startup of a new group).

### Replication and ISR — durability

Each partition has one **leader** and N **follower** replicas on *different* brokers. The **ISR (In-Sync Replicas)** is the set of replicas caught up to the leader. Durability is governed by producer **acks**:

| `acks` | Producer waits for | Durability | Risk |
|---|---|---|---|
| `0` | nothing (fire and forget) | lowest | message lost if it never lands |
| `1` | leader only | medium | **leader can die before replicating → loss** |
| `all` (`-1`) | all in-sync replicas | highest | higher latency |

The safe durable config is **`acks=all` + `min.insync.replicas=2`** — the write only succeeds if at least 2 replicas have it, so a single broker loss can't lose acknowledged data. `acks=1` is the dangerous middle ground: the leader acks, then dies before any follower copies the message.

### Rebalancing

A **rebalance** redistributes partitions when a consumer joins/leaves, the partition count changes, or a consumer misses its heartbeat (`session.timeout.ms`). The **group coordinator** (a broker) drives it. During a stop-the-world rebalance, consumption pauses briefly — so flapping consumers (frequent rebalances) hurt throughput. Newer protocols (cooperative/incremental rebalancing) reduce the pause.

### Retention and replay

Kafka keeps messages for a configured **retention** (default 7 days, or by size) **regardless of consumption** — so a new consumer group can **replay** history from the beginning, and you can reprocess after a bug fix. Contrast **RabbitMQ**, which deletes a message once it's ACKed. This replay capability is a major reason to choose Kafka over a traditional queue.

### Exactly-once — the honest version

- **At-least-once** is the real-world default (process, then commit; duplicates possible).
- **Kafka EOS (exactly-once semantics)** exists *but is Kafka→Kafka only*: `enable.idempotence=true` (producer dedups retries via a per-partition sequence number) + `transactional.id` (atomic produce-and-commit-offset across topics) + consumer `isolation.level=read_committed`.
- **Exactly-once to an external database is a myth.** The DB write and the Kafka offset commit are two separate systems with no shared transaction — a crash between them double-processes or loses. The real answer: **make the consumer idempotent** (a `processed_messages(message_id)` check, a business-state guard, or an optimistic-lock `UPDATE ... WHERE status='QUEUED'`). Thursday's Outbox block builds on exactly this.

### Project tie-in

- **Smart360 — acks per event class:** *"I'd set `acks=all` + `min.insync.replicas=2` for authorization events (correctness-critical — losing one is a security bug) and `acks=1` for notification events (latency-sensitive, and the consumer is idempotent anyway, so a rare loss is recoverable)."* Naming the *per-topic* trade-off is the senior signal.
- **Smart360 — partition key:** events keyed by `tenant_id` so all of a tenant's events stay ordered within a partition, while different tenants spread across partitions for parallelism.
- **WebX — replay:** Kafka retention let a buggy LLM-result consumer be fixed and *replayed* over the last 24 h of events without re-triggering the upstream LLM calls — impossible with a delete-on-ACK queue.

---

## Block C — DSA: Topological Sort (~1.5 hr)

### Theory

A **topological sort** is a linear ordering of a **DAG**'s vertices such that every edge `u → v` has `u` before `v` — "do prerequisites first." It exists **iff the directed graph has no cycle**. Two algorithms:

| Algorithm | Mechanism | Cycle detection |
|---|---|---|
| **Kahn's (BFS)** | repeatedly remove a 0-in-degree node, decrement neighbors' in-degrees | if processed count `< V` at the end → a cycle absorbed some nodes |
| **DFS (3-color)** | post-order; reverse the finish order | hitting a node currently *in the recursion stack* (gray) → a back edge → cycle |

The 3-color scheme: **white** (0, unvisited), **gray** (1, in the current DFS path), **black** (2, fully done). Reaching a **gray** node means a back edge → cycle.

### Practice set

**1. LC 207 — Course Schedule** (Medium) — *can you finish?* = is the prerequisite graph acyclic?
- **Approach (Kahn's):** build in-degrees; seed a queue with all 0-in-degree nodes; pop, count, and decrement neighbors, enqueuing any that hit 0. If the processed count equals `V`, no cycle.
```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    int[] indeg = new int[numCourses];
    for (int[] p : prerequisites) { adj.get(p[1]).add(p[0]); indeg[p[0]]++; }  // p[1] -> p[0]
    Queue<Integer> q = new ArrayDeque<>();
    for (int i = 0; i < numCourses; i++) if (indeg[i] == 0) q.offer(i);
    int done = 0;
    while (!q.isEmpty()) {
        int u = q.poll(); done++;
        for (int v : adj.get(u)) if (--indeg[v] == 0) q.offer(v);
    }
    return done == numCourses;   // all processed => acyclic
}
```
- **Key insight:** Kahn's *detects the cycle for free* — a cycle's nodes never reach in-degree 0, so they're never processed; `done < V` is the cycle signal. Edge direction matters: `prereq → course`.
- **Complexity:** O(V + E) / O(V). Target: AC in 20 min.

**2. LC 210 — Course Schedule II** (Medium) — produce the actual order.
- **Approach:** Kahn's, but **record each popped node** into an output array; that array *is* a valid topological order (return empty if a cycle leaves it short). Alternatively, reverse a DFS post-order.
```java
public int[] findOrder(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    int[] indeg = new int[numCourses];
    for (int[] p : prerequisites) { adj.get(p[1]).add(p[0]); indeg[p[0]]++; }
    Queue<Integer> q = new ArrayDeque<>();
    for (int i = 0; i < numCourses; i++) if (indeg[i] == 0) q.offer(i);
    int[] order = new int[numCourses];
    int idx = 0;
    while (!q.isEmpty()) {
        int u = q.poll(); order[idx++] = u;
        for (int v : adj.get(u)) if (--indeg[v] == 0) q.offer(v);
    }
    return idx == numCourses ? order : new int[0];   // short array => cycle => no valid order
}
```
- **Key insight:** the *order of removal* in Kahn's is a topo order. The DFS variant gives the order *reversed* (you reverse the post-order finish sequence) — both valid, often different orderings.
- **Complexity:** O(V + E) / O(V). Target: AC in 25 min. **Edge cases:** disconnected components (multiple 0-in-degree seeds), self-loop (`a → a` is an immediate cycle), empty prerequisites (any order valid).

---

## 💻 Practice coding questions

1. **LC 200 Number of Islands** — flood-fill; mark in-place; iterative BFS for huge grids.
2. **LC 695 Max Area of Island** — DFS returns `1 + sum(neighbors)`; track the max.
3. **LC 133 Clone Graph** — `HashMap<original, clone>`; register before recursing.
4. **LC 994 Rotting Oranges** — multi-source BFS; seed all rotten; waves = minutes; -1 if fresh remains.
5. **LC 207 Course Schedule** — Kahn's; `done < V` ⇒ cycle.
6. **LC 210 Course Schedule II** — Kahn's removal order is the answer; short array ⇒ cycle.
7. **LC 286 Walls and Gates** — multi-source BFS from all gates (same shape as Rotting Oranges).
8. **LC 542 01 Matrix** — multi-source BFS from all 0s.
9. **LC 130 Surrounded Regions** — flood-fill *from the borders* to mark survivors, then flip the rest.
10. **LC 417 Pacific Atlantic Water Flow** — two multi-source BFS/DFS from each ocean's edges; intersect.
11. **LC 79 Word Search** — DFS with backtracking (mark/unmark on the grid).
12. **LC 802 Find Eventual Safe States** — reverse-graph Kahn's, or DFS 3-color.
13. **LC 269 Alien Dictionary** — build the precedence graph from adjacent words, then topo sort.

> For each grid problem, write the `DIRS` deltas and the bounds check first; they're identical across all of them. Re-time LC 200 and LC 994 — target ≤ 15 min and ≤ 20 min.

---

## 🎤 Interview questions

1. **BFS vs DFS — trade-offs in 60 seconds.** Both O(V+E). BFS uses a queue, explores level by level → shortest path on *unweighted* graphs and multi-source spread, but O(width) memory. DFS uses the call/explicit stack → cycle detection, topo sort, flood-fill, less memory on wide graphs, but recursion can overflow on deep ones.
2. **Adjacency list vs matrix — when each?** List (O(V+E) space) for sparse graphs — the usual case. Matrix (O(V²)) for dense graphs or when you need O(1) "is there an edge u→v" checks.
3. **Why mark a cell visited on *enqueue*, not dequeue, in grid BFS?** Otherwise multiple neighbors enqueue the same cell before it's processed, bloating the queue and double-counting. Mark the instant you add it.
4. **Number of Islands on a 100,000 × 100,000 grid?** Recursive DFS overflows the stack — switch to iterative BFS/DFS. Distributed: partition into tiles, flood-fill each, then union-find across tile boundaries to merge islands that span seams.
5. **Why does Clone Graph need a map, and why register the clone before recursing?** The map is both the visited set and the original→clone lookup. Registering before recursing means a cycle's back-edge finds the in-progress clone instead of recursing forever.
6. **Why multi-source BFS for Rotting Oranges instead of looping single-source BFS?** Seeding all rotten cells at once makes the frontier expand from every source simultaneously; the wave count is the elapsed minutes directly. Single-source per orange would be far slower and harder to synchronize.
7. **Detect a cycle in a directed vs undirected graph?** Directed: DFS 3-color (reach a gray/in-stack node) or Kahn's (`processed < V`). Undirected: DFS tracking the parent (an edge to a visited non-parent is a cycle), or union-find (union of an already-connected pair).
8. **Why does Kahn's algorithm detect a cycle?** Cycle nodes never reach in-degree 0 (each has an unprocessed predecessor inside the cycle), so they're never enqueued; the processed count ends below `V`.
9. **Kahn's vs DFS for topological sort — outputs?** Kahn's emits the removal order directly; DFS emits the *reverse* of the post-order finish order. Both are valid topo orders and may differ.
10. **Kafka: why is ordering only guaranteed within a partition?** Partitions are independent logs written/read in parallel; there's no global clock across them. To order a set of related events, route them to the same partition via a shared key.
11. **A consumer crashes after receiving but before committing — what happens?** The coordinator detects the missed heartbeat, rebalances, and reassigns the partition; the new owner re-reads from the last committed offset → reprocessing → at-least-once → the consumer must be idempotent.
12. **`acks=1` vs `acks=all` — what's the real risk of `acks=1`?** The leader acknowledges, then dies before any follower replicates the message → silent data loss. `acks=all` + `min.insync.replicas=2` requires the write to land on ≥2 replicas, surviving a single broker loss.
13. **What is the ISR and why does it matter?** The set of replicas caught up to the leader. `acks=all` waits for the ISR; `min.insync.replicas` sets the floor — if too few replicas are in-sync, the producer errors rather than risk an under-replicated (loss-prone) write.
14. **How many consumers can usefully read a 4-partition topic?** Up to 4 in one group (one per partition); a 5th sits idle. Partition count is the parallelism ceiling — choose it at creation time.
15. **Idempotent producer vs idempotent consumer?** Producer (`enable.idempotence=true`): the broker dedups retried writes via a per-partition sequence number (no duplicate from producer retries). Consumer: *your* code dedups effects on the downstream system. Independent guarantees solving different duplicate sources.
16. **Is exactly-once delivery to a database possible?** No — the DB write and the offset commit are two systems with no shared transaction. Kafka EOS is Kafka→Kafka only. For a DB sink, make the consumer idempotent (`processed_messages` table, business-state guard, or optimistic-lock update).
17. **Kafka vs RabbitMQ for an event stream?** Kafka retains messages (default 7 days) regardless of consumption → replay and independent fan-out to multiple consumer groups. RabbitMQ deletes on ACK → no replay, but richer routing and lower-latency work-queue semantics. Choose Kafka for event streaming/replay, RabbitMQ for task queues.
18. **"Just use a bigger database instead of Kafka for our queue."** For low volume a DB-backed queue (`SELECT ... FOR UPDATE SKIP LOCKED`) is simpler and already transactional — that's effectively the Outbox. At scale, DB polling loads the primary, there's no replay, and no clean fan-out to independent consumers. Start simple; introduce Kafka past a throughput/replay threshold.
19. **What triggers a rebalance, and why minimize them?** A consumer joining/leaving, a partition-count change, or a missed heartbeat. During a stop-the-world rebalance consumption pauses, so frequent rebalances (flapping consumers, too-short `session.timeout.ms`) hurt throughput. Cooperative rebalancing reduces the pause.
20. **`auto.offset.reset=earliest` vs `latest` — when does it even apply?** Only when a group has *no committed offset* (first start, or after offsets expired). `earliest` replays from the beginning; `latest` skips to new messages. It does nothing once the group has committed offsets.

---

## ✅ Self-check

1. Write the `DIRS` deltas and the grid bounds check from memory; state why you mark on enqueue in BFS.
2. Explain BFS vs DFS trade-offs in 60 seconds (structure, killer app, memory risk).
3. Why does Kahn's algorithm detect a cycle? (Cycle nodes never hit in-degree 0 → `processed < V`.)
4. State the durable Kafka producer config and the specific failure `acks=1` allows.
5. Explain why a consumer group can't have more useful consumers than partitions.
6. Give the honest answer to "can we get exactly-once into our database?" in two sentences.

---

*Nav: ← [02-tue-jul-14.md](02-tue-jul-14.md) · [Week 3](README.md) · [04-thu-jul-16.md](04-thu-jul-16.md) →*
