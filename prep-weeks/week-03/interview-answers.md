# Week 3 — Interview Question Answers

Crisp, self-contained answers to every interview question from Week 3 (trees, graphs, Trie, Dijkstra, DB indexing, caching, SQL/NoSQL, Kafka, Outbox/Saga, frontend, typeahead). For deeper Java context see [interview-qa.md](../../interview-qa.md) and the numbered modules (e.g. `../../07-collections.md`, `../../06-concurrency-and-collections.md`).

---

## Day 1 — Tree Traversals + BST + DB Indexing

1. **Walk through all four traversal orders and one real use of each.** In-order (L→Root→R) → sorted output on a BST, range queries; pre-order (Root→L→R) → serialize/deep-copy, expression-tree prefix; post-order (L→R→Root) → bottom-up work (deletion, subtree size, height/diameter); level-order (BFS) → shortest path on unweighted graphs, by-depth processing. *Gotcha:* the order is defined by *when you visit the node* relative to its children, not by the children's order.

2. **Why does in-order traversal of a BST produce sorted output?** The BST invariant is left < node < right, so visiting L → node → R emits values in ascending order by construction. *Why it matters:* this single fact powers Validate BST and Kth Smallest. *Gotcha:* only holds for a *valid* BST — duplicates or a broken invariant break it.

3. **Validate BST — why isn't `left.val < node.val < right.val` enough?** That only checks immediate parent-child and misses grandparent violations (a deep left-subtree node larger than an ancestor). Pass a shrinking `(min, max)` open interval down instead. *Gotcha:* use `long` bounds (or null sentinels) so a node holding `Integer.MIN/MAX_VALUE` doesn't false-fail.

4. **Recursive vs iterative traversal — space?** Both are O(n) time; recursion uses an O(h) call stack, iteration uses an O(h) explicit stack. *Gotcha:* h is O(log n) balanced but O(n) on a skewed tree. Morris traversal threads in-order-predecessor links for O(1) space — mention it, rarely code it.

5. **LCA where one target is the ancestor of the other?** The recursion returns the matched node when it hits p (or q); the sibling branch returns null; the parent receives the matched node and propagates it up — correct without any special "found both" case. *Why:* the both-sides-non-null check only fires at the true split point.

6. **General-tree LCA vs BST LCA — complexity difference?** General tree: O(n) post-order, since you must search both subtrees. BST: O(h) guided walk with O(1) space, because the ordering tells you which subtree contains both. *Gotcha:* don't reuse the O(n) post-order on a BST — it throws away the ordering.

7. **Why track a global max in Diameter instead of returning it?** The longest path may not pass through the root, so you must consider the left-height + right-height path at *every* node. The function returns height upward while a side-effect global captures the best bending path. *Reuse:* identical shape to Max Path Sum (LC 124).

8. **Kth Smallest follow-up — BST mutates often, kth is hot. Optimize?** Augment each node with `leftCount` (size of its left subtree); each query becomes O(h) — if `leftCount+1 == k` it's the node, if `k <= leftCount` go left, else go right with `k -= leftCount+1`. *Gotcha:* inserts/deletes must maintain the counts (also O(h)).

9. **What is a B-tree and why is it the default DB index?** A self-balancing sorted tree (a B+ tree in practice) giving O(log n) reads; because keys stay sorted with linked leaves, it serves equality, ranges, ordering, and prefix `LIKE 'abc%'`. *Why default:* it covers the widest set of query shapes. *Gotcha:* every write must update every affected index — indexes speed reads, slow writes.

10. **B-tree vs hash index — when each?** B-tree for ranges/ordering/prefix and as the safe default; hash for pure equality with no ordering (rare). *Why:* same trade-off as `TreeMap` (O(log n), ordered) vs `HashMap` (O(1), unordered). *Gotcha:* a hash index can't do `>`, `<`, `BETWEEN`, or `ORDER BY`.

11. **Explain the leftmost-prefix rule with a concrete index.** `(tenant_id, status, created_at)` is sorted by tenant_id, then status, then created_at — so it serves `tenant_id`, `tenant_id+status`, and all three, but **not** `status` alone. *Why:* the tree is sorted by the leading key first, so skipping it leaves nothing to seek on.

12. **How do you order columns in a composite index?** Equality-filter columns first, then the sort column, then the range column last. *Why:* a range predicate "stops" the index — columns after it can't be used for seeking, only filtering. Rule: equality → sort → range.

13. **What is a covering index and why is it the fastest read?** It contains every column the query touches (filter + select), enabling an index-only scan that never visits the heap. *Tool:* use Postgres `INCLUDE` to carry return-only payload columns as non-key data so the tree stays narrow. *Gotcha:* add too many columns and the index bloats and slows writes.

14. **When is a partial index the right tool?** When queries always filter on a constant predicate, e.g. `WHERE status='PENDING'` — the index covers only those rows, staying tiny and hot. *Use:* ideal for the Outbox poll. *Gotcha:* the index only helps queries whose predicate matches (or implies) the partial condition.

15. **Selectivity — why not index a boolean column?** Low cardinality means the index returns a large fraction of rows; random index I/O for ~40% of a table is slower than a sequential scan, so the planner ignores it. *Fix:* index high-cardinality columns (user_id, uuid), or fold the low-cardinality one into a composite/partial index.

16. **Add an index to a 100M-row production table without downtime.** `CREATE INDEX CONCURRENTLY` — takes a weaker lock so reads/writes continue. *Gotcha:* it's slower and can leave an INVALID index on failure (drop and retry); run off-peak, monitor `pg_stat_progress_create_index`, and verify with `EXPLAIN (ANALYZE, BUFFERS)`.

17. **Postgres heap storage vs MySQL InnoDB clustered index — practical difference?** Postgres: the table is an unordered heap and indexes point at row tuples (TIDs) separately. InnoDB: the table *is* a B+ tree keyed by the PK, so secondary indexes store the PK and need a second "bookmark" lookup. *Consequence:* in InnoDB the PK choice affects the size of every secondary index.

18. **You added the right index but the query is still slow — debug it.** Run `EXPLAIN (ANALYZE, BUFFERS)` and check: stale stats (run `ANALYZE`), a predicate not matching the leftmost prefix, a function wrapping the column (`WHERE lower(email)=?` needs a functional index), an implicit type cast disabling the index, or genuinely low selectivity making seq-scan cheaper. *Key:* verify you see `Index Scan`, not `Seq Scan`.

---

## Day 2 — Tree Construction + LCA + Caching / Redis

1. **Which two traversals uniquely rebuild a binary tree, and which pair is ambiguous?** Pre+in and post+in are unique; pre+post is ambiguous for general binary trees (you can't place a single child's side — unique only for *full* trees). *Bonus:* pre-order alone *with null markers* is also lossless (LC 297).

2. **In LC 105, why build the left subtree before the right?** Pre-order lays out `Root, [entire left subtree], [entire right subtree]`, so the moving `preIdx` must consume the whole left subtree before it reaches the right's root. *Gotcha:* swapping the two recursive lines silently produces a wrong tree (no error).

3. **Why is the naive LC 105 O(n²), and how do you fix it?** Scanning the in-order array to locate each root is O(n) per node → O(n²) on a skewed tree. *Fix:* a `HashMap<value, inorderIndex>` makes the split O(1), giving O(n) total. *Requires:* distinct values.

4. **In Max Path Sum, why does the returned value differ from the answer candidate?** The returned value can use only *one* child (so a parent can extend it into a longer path); the candidate uses *both* children (a path that bends at the node, which can't be extended further). *Gotcha:* clamp negative child gains to 0, and init `best` to `Integer.MIN_VALUE` for all-negative trees.

5. **Why do null markers make pre-order alone enough to deserialize?** They encode the tree's *shape* — without them `[1,2]` could be 2-as-left or 2-as-right. *Key:* serialize and deserialize must walk in the same order; a `Deque`/index pointer gives a clean "consume next token" stream.

6. **Name the four cache write strategies and one failure mode each.** Cache-aside (stampede on hot-key expiry), write-through (doubled write latency, caches maybe-never-read data), write-back (data loss if the node dies before async flush), write-around (slow first read). *Default:* cache-aside for application data.

7. **What is a cache stampede and how do you prevent it?** A hot key expires (or cold start) and thousands of concurrent misses hammer the DB at once. *Mitigations:* per-key mutex (request coalescing — first miss recomputes, others wait), probabilistic early expiration, or stale-while-revalidate. *Why it's cache-aside's sharp edge:* the cache fills lazily on miss.

8. **LRU vs LFU — when each?** LRU evicts least-recently-used → good for temporal locality (recently used ⇒ soon used). LFU evicts least-frequently-used → good for a stable hot set where a one-off scan shouldn't evict persistently-hot keys. *Gotcha:* Redis 4+ LFU uses a *decaying* probabilistic counter so last-week-hot items age out.

9. **Implement an O(1) LRU — what data structures?** `HashMap` for O(1) lookup + a doubly-linked list for O(1) recency reordering; sentinel head/tail nodes avoid null checks (head = MRU, tail = LRU). *Shortcut:* `LinkedHashMap(cap, 0.75f, true)` with overridden `removeEldestEntry` — but interviewers usually want the hand-rolled list.

10. **Redis RDB vs AOF — pick for a JWT blacklist vs a URL cache.** RDB = point-in-time snapshots (compact, fast restart, loses data since last snapshot). AOF = append every write (`appendfsync everysec` bounds loss to ~1s). JWT blacklist → AOF (you can't safely un-invalidate a token). URL cache → RDB or none (reconstructible from the source of truth).

11. **Redis Sentinel vs Cluster — what does each solve?** Sentinel = HA/automatic failover for a *single primary* (scales availability — monitors and promotes a replica). Cluster = horizontal sharding across 16384 hash slots (scales capacity). *Key:* they solve different problems — availability vs capacity.

12. **Why set the S3-URL cache TTL *below* the presigned-URL expiry?** A 15-min URL cached for 13 min leaves a 2-min buffer for clock skew and serve latency. *Why:* a cached URL that expires in the client's hand is worse than re-minting slightly sooner.

13. **SQL vs NoSQL — your decision framework?** Start from access patterns: relational optimizes for *unknown future* queries (joins, ACID, ad-hoc); NoSQL optimizes for *known* patterns you model the schema around. *Line:* "Postgres for ~95%; move one slice to NoSQL when it consistently hits relational limits — polyglot persistence, not replacement."

14. **Match each NoSQL family to a use case.** Key-value (Redis/DynamoDB) → cache/sessions/rate-limiter/JWT blacklist; document (MongoDB) → self-contained evolving aggregates with no joins; wide-column (Cassandra) → high-write time-series with known partition-key reads; graph (Neo4j) → multi-hop relationship traversal where SQL self-joins explode at 4+ hops.

15. **Like count — SQL or NoSQL, and write strategy?** Keep the canonical count in Postgres (it's relational, tied to a post/user); serve and increment it in Redis (`INCR`, O(1)); flush to the DB periodically — effectively write-back for the hot counter with the DB as durable backstop. *Why:* losing a few seconds of like increments on a crash is acceptable; for a payment it wouldn't be.

16. **What is write-around and when is it right?** Writes go straight to the DB, bypassing the cache; the cache fills only on reads. *Right for:* write-once/read-rarely data (audit logs) and caches over an authoritative store (the S3-URL cache). *Gotcha:* high first-read latency.

17. **Cache-aside on a cache outage — what happens?** Reads degrade gracefully to the DB — correctness preserved, latency rises. *Why it's the safe default:* contrast write-through, where the cache sits on the critical write path and an outage breaks writes.

18. **How would you cache an expensive Deep Fathom dashboard aggregate?** Cache-aside with a TTL matched to acceptable staleness (e.g. 5 min) and a per-key mutex so only one request recomputes on expiry while the rest wait for the refilled value. *Why the mutex:* prevents a stampede on the expensive aggregate when the TTL lapses at peak load.

---

## Day 3 — Graphs BFS/DFS + Kafka Internals

1. **BFS vs DFS — trade-offs in 60 seconds.** Both O(V+E). BFS uses a queue, explores level by level → shortest path on *unweighted* graphs, multi-source spread, but O(width) memory. DFS uses the call/explicit stack → cycle detection, topo sort, flood-fill, less memory on wide graphs, but recursion can stack-overflow on deep ones.

2. **Adjacency list vs matrix — when each?** List (O(V+E) space) for sparse graphs — the usual case. Matrix (O(V²)) for dense graphs or when you need O(1) "is there an edge u→v?" checks. *Gotcha:* a matrix wastes space on sparse graphs.

3. **Why mark a cell visited on *enqueue*, not dequeue, in grid BFS?** Otherwise multiple neighbors enqueue the same cell before it's processed, bloating the queue and double-counting. *Fix:* mark the instant you add it to the queue.

4. **Number of Islands on a 100,000 × 100,000 grid?** Recursive DFS overflows the stack → switch to iterative BFS/DFS. *Distributed:* partition into tiles, flood-fill each, then union-find across tile boundaries to merge islands spanning seams.

5. **Why does Clone Graph need a map, and why register the clone before recursing?** The `HashMap<original, clone>` is both the visited set and the original→clone lookup. *Why register first:* a cycle's back-edge then finds the in-progress clone instead of recursing forever.

6. **Why multi-source BFS for Rotting Oranges instead of looping single-source BFS?** Seeding all rotten cells at once makes the frontier expand from every source simultaneously, so the wave count *is* the elapsed minutes. *Gotcha:* single-source per orange would be far slower and hard to synchronize into a time count.

7. **Detect a cycle in a directed vs undirected graph?** Directed: DFS 3-color (reach a gray/in-stack node) or Kahn's (`processed < V`). Undirected: DFS tracking the parent (an edge to a visited non-parent is a cycle), or union-find (union of an already-connected pair). *Gotcha:* union-find can't detect *directed* cycles.

8. **Why does Kahn's algorithm detect a cycle?** Cycle nodes never reach in-degree 0 (each has an unprocessed predecessor inside the cycle), so they're never enqueued; the processed count ends below V. *Bonus:* this detection is free — no extra pass.

9. **Kahn's vs DFS for topological sort — outputs?** Kahn's emits the removal order directly. DFS emits the *reverse* of the post-order finish order. Both are valid topo orders and may differ. *Gotcha:* don't forget to reverse the DFS output.

10. **Kafka: why is ordering only guaranteed within a partition?** Partitions are independent logs written/read in parallel — there's no global clock across them. *Fix:* route related events to the same partition via a shared key (`hash(key) % numPartitions`), e.g. key by `tenant_id`. A null key round-robins (no ordering).

11. **A consumer crashes after receiving but before committing — what happens?** The coordinator detects the missed heartbeat, rebalances, and reassigns the partition; the new owner re-reads from the last committed offset → reprocessing → at-least-once. *Consequence:* the consumer must be idempotent.

12. **`acks=1` vs `acks=all` — what's the real risk of `acks=1`?** The leader acknowledges, then dies before any follower replicates → silent data loss. *Safe config:* `acks=all` + `min.insync.replicas=2` requires the write to land on ≥2 replicas, surviving a single broker loss. `acks=0` is fire-and-forget.

13. **What is the ISR and why does it matter?** The In-Sync Replicas set — replicas caught up to the leader. `acks=all` waits for the ISR; `min.insync.replicas` sets the floor, so if too few replicas are in-sync the producer errors rather than risk a loss-prone under-replicated write. *Key:* the ISR shrinks as replicas fall behind.

14. **How many consumers can usefully read a 4-partition topic?** Up to 4 in one group (one consumer per partition); a 5th sits idle. *Key:* partition count is the parallelism ceiling — choose it at creation. *Gotcha:* increasing partitions later breaks key-ordering for in-flight keys.

15. **Idempotent producer vs idempotent consumer?** Producer (`enable.idempotence=true`): the broker dedups *retried writes* via a per-partition sequence number. Consumer: *your* code dedups *effects* on the downstream system. *Key:* independent guarantees solving different duplicate sources.

16. **Is exactly-once delivery to a database possible?** No — the DB write and the offset commit are two systems with no shared transaction, so a crash between them double-processes or loses. Kafka EOS is Kafka→Kafka only. *Real answer:* make the consumer idempotent (`processed_messages` table, business-state guard, or optimistic-lock update).

17. **Kafka vs RabbitMQ for an event stream?** Kafka retains messages (default 7 days) regardless of consumption → replay and independent fan-out to multiple consumer groups. RabbitMQ deletes on ACK → no replay, but richer routing and lower-latency work-queue semantics. *Pick:* Kafka for event streaming/replay, RabbitMQ for task queues.

18. **"Just use a bigger database instead of Kafka for our queue."** For low volume a DB-backed queue (`SELECT ... FOR UPDATE SKIP LOCKED`) is simpler and already transactional — effectively the Outbox. *At scale:* DB polling loads the primary, there's no replay, and no clean fan-out to independent consumers. Start simple; introduce Kafka past a throughput/replay threshold.

19. **What triggers a rebalance, and why minimize them?** A consumer joining/leaving, a partition-count change, or a missed heartbeat (`session.timeout.ms`). During a stop-the-world rebalance consumption pauses, so flapping consumers hurt throughput. *Mitigation:* cooperative/incremental rebalancing reduces the pause.

20. **`auto.offset.reset=earliest` vs `latest` — when does it even apply?** Only when a group has *no committed offset* (first start, or after offsets expired). `earliest` replays from the beginning; `latest` skips to new messages. *Key:* it does nothing once the group has committed offsets.

---

## Day 4 — Union-Find + Outbox / Saga + Trie

1. **Amortized complexity of Union-Find with path compression + union by rank?** ≈ O(1) per op — inverse Ackermann α(n), a small constant (< 5) for any realistic n. *Gotcha:* without *both* optimizations it degrades toward O(n) per op.

2. **Path compression vs union by rank — what does each fix?** Compression flattens the path to the root *during* `find` (`parent[x]=find(parent[x])`); rank keeps trees shallow *during* `union` by attaching the shorter tree under the taller. *Key:* they attack tree height from the two different operations; together they give α(n).

3. **DSU vs BFS/DFS for connected components — when each?** DSU for pure connectivity, online edge additions, and undirected cycle detection. BFS/DFS when you also need the actual path, distance, or to process by level. *Tell:* "components" or "are these two connected?" signals DSU.

4. **Why can't Union-Find detect a cycle in a *directed* graph?** It only models undirected "same set" membership — it has no notion of edge direction. *Use instead:* DFS 3-color or Kahn's for directed cycles.

5. **What is the dual-write problem and how does Outbox solve it?** Saving an entity and publishing an event are two I/O ops to two systems; a crash between them loses one (DB updated, event lost or vice-versa). *Fix:* Outbox writes the event into a table in the *same DB transaction* — one atomic commit — and a separate process delivers it later.

6. **Polling publisher vs Debezium for the outbox?** Polling is a `@Scheduled` query — simple, but adds DB load and poll-interval (seconds) latency. Debezium tails the Postgres WAL (logical replication) → near real-time, no polling load, but more operational complexity (Kafka Connect + connector). *Pick:* polling for low/medium volume, Debezium for high throughput/low latency.

7. **Outbox poller publishes to Kafka, then the DB dies before marking SENT — what happens?** The row stays PENDING → republished next cycle → duplicate. *Why:* no atomicity across Postgres + Kafka. *Real answer:* accept at-least-once and make every consumer idempotent from day one.

8. **Why is exactly-once to an external DB a myth?** The work (DB write) and the offset commit are separate systems with no shared transaction — you can't atomically commit both. Kafka EOS is Kafka→Kafka only. *Real systems:* at-least-once + idempotent effects.

9. **Idempotent producer vs idempotent consumer?** Producer (`enable.idempotence=true`): the broker dedups retried writes via a per-partition sequence number. Consumer: your code dedups *effects* on the downstream DB. *Key:* independent guarantees, different duplicate sources.

10. **Three ways to make a consumer idempotent?** Message-level dedup (`INSERT processed_messages(message_id) ON CONFLICT DO NOTHING`), business-level state machine (`IF status='QUEUED' THEN process`), optimistic-lock claim (`UPDATE ... SET status='PROCESSING' WHERE id=? AND status='QUEUED'` — one writer wins). *Gotcha:* the dedup insert and the business effect must share one transaction.

11. **Choreography vs orchestration Saga — pick one for a 2-service notification flow.** Choreography — loose coupling, no SPOF, and a slightly-late notification doesn't matter. *Switch to orchestration* at 4+ cross-dependent services or when you need central saga-state visibility / compliance audit / targeted retries.

12. **What is a compensating transaction and when do you design it?** A *semantic* undo (issue a refund, not "un-charge" — you can't DB-rollback across services). *When:* design the compensations *before* the forward path, because some steps have no native rollback.

13. **Why is the outbox poll query indexed on `(status, created_at)`?** So it's an index range scan over only PENDING rows in creation order, not a full-table scan that grows with the table. *Bonus:* a partial index `WHERE status='PENDING'` makes it even tinier.

14. **(Curveball) "Just use a bigger DB instead of Kafka for our event queue."** For low volume a DB-backed queue with `SELECT FOR UPDATE SKIP LOCKED` works — that *is* Outbox-as-queue. *At scale:* DB polling loads the primary, gives no replay, and no fan-out to independent consumer groups. Start simple, introduce Kafka past a volume threshold.

15. **Trie vs HashSet for a dictionary?** Both do exact membership in O(L). Only the Trie answers prefix queries ("all words starting with `fac`") without scanning every key, and it shares storage across common prefixes. *Gotcha:* a Trie uses more memory per node (especially the `[26]`-array layout).

16. **Why does the `.` wildcard force DFS in LC 211?** A literal char follows exactly one child; `.` could match any of 26, so you must branch into all non-null children and succeed if *any* path completes — that's a DFS, not a linear walk. *Cost:* worst case O(26^d) for d wildcards.

17. **How would you scale the LC 1268 trie to production typeahead?** Precompute top-k at each prefix node (read, don't DFS, at request time), shard the trie by leading characters, and refresh it offline from query-frequency logs. *Bridge:* this is exactly the typeahead HLD (Day 5).

18. **DSU `union` returns false — what does that tell you in LC 684?** The two endpoints were already in the same set, so a path already existed; this edge closes a cycle and is the redundant one. *Why scan in order:* the first such edge is the answer the problem wants.

---

## Day 5 — Dijkstra / Weighted Graphs + Frontend (Vue/React) + Typeahead HLD

1. **Why does plain BFS fail on a weighted graph, and what replaces it?** BFS expands strictly by hop count (every edge "costs 1"), so the fewest-edge path need not be the cheapest. *Replace with* Dijkstra — a min-heap, greedy by tentative distance, for non-negative weights.

2. **State the Dijkstra relaxation invariant and why it holds.** When a node is popped from the min-heap its recorded distance is final — any alternative route reaches it through a node still in the heap whose distance is ≥ the popped one, so it can't be shorter. *Precondition:* non-negative edge weights.

3. **Why does Dijkstra break on negative edges, and what do you use instead?** A negative edge can make an already-"finalized" node cheaper later, violating the pop-is-final invariant. *Use* Bellman-Ford (O(V·E)) for negatives (no negative cycle), or Floyd-Warshall (O(V³)) for all-pairs.

4. **What is the lazy-deletion trick in your heap, and why use it?** Instead of decrease-key, push the improved `(dist, node)` and skip pops where `d > dist[node]`. *Why:* avoids an indexed heap — same correctness, much simpler code.

5. **Why is plain Dijkstra wrong for Cheapest Flights Within K Stops?** The cheapest path may exceed the stop budget; finalizing a node by price alone discards a pricier-but-fewer-stops route that's actually valid. *Fix:* do `k+1` Bellman-Ford rounds over a distance snapshot.

6. **Why the `snapshot` (clone) in the Bellman-Ford rounds for LC 787?** Relaxing against the live `dist` in the same round lets one path consume multiple new edges, breaking the "≤ k stops" cap. *The snapshot* ensures each round adds at most one edge.

7. **How is Path With Minimum Effort different from normal Dijkstra?** It's a bottleneck/minimax path — the route's cost is the *max* edge along it, so you relax with `max(currentEffort, |Δ|)` instead of summing. *Alternatives:* union-find over edges sorted by difference, or binary-search-the-answer + BFS.

8. **Dijkstra time complexity and where the log comes from.** O(E log V) — each edge can push one heap entry, and each pop/push is `log` of the heap size (≤ E entries). *Space:* O(V + E).

9. **Vue 2 vs Vue 3 reactivity — what changed and why?** Vue 2's `Object.defineProperty` couldn't detect property add/delete or array-index writes (forcing `Vue.set`). Vue 3's ES6 `Proxy` intercepts get/set/deleteProperty/has, tracking new keys, deletes, and index writes natively.

10. **Why does destructuring a `reactive()` object break reactivity, and the fix?** Destructuring extracts the raw values and drops the Proxy, so later changes aren't tracked. *Fix:* `toRefs(state)` turns each field into a connected `ref`. (In Pinia, use `storeToRefs`.)

11. **`ref` vs `reactive` vs `computed` in Vue 3?** `ref(value)` boxes a primitive in `{ value }` (`.value` in JS, auto-unwrapped in templates); `reactive(obj)` deep-proxies an object; `computed` is a lazy, cached derived value that recomputes only when a tracked dep changes.

12. **`useEffect` dependency array — the three cases and the classic bug.** `[]` runs once after mount; `[deps]` runs whenever a dep changes; omitted runs every render (usually a bug). *Classic bug:* missing deps → stale closure over old state; the `exhaustive-deps` lint rule catches it. *Tip:* return a cleanup function to unsubscribe/abort.

13. **React `useEffect(() => fetchData(), [userId])` and `userId` changes mid-flight?** Cleanup runs and the effect re-runs; the earlier fetch may resolve later and overwrite fresh data. *Guard:* an `isCancelled` flag or an `AbortController` aborted in the cleanup function.

14. **Why never use the array index as a React list `key`?** On insert/reorder the index→element mapping shifts, so React reuses the wrong DOM nodes and component state leaks across items. *Fix:* use a stable domain id. *Why keys matter:* they give elements stable identity so React reuses nodes.

15. **Reconciliation and Fiber in one breath.** On a state change React builds a new virtual DOM, diffs it against the old tree, and commits only the minimal real-DOM changes. *Fiber* is the incremental reconciler that splits work into interruptible units, enabling React 18 concurrent features (`useTransition`, `useDeferredValue`).

16. **Vue vs React performance model — the trade-off.** Vue's compiler marks static vs dynamic nodes at build time, so updates are fine-grained automatically. React diffs the whole virtual tree at runtime and you opt into `memo`/`useMemo`/`useCallback`. *Net:* Vue wins update-heavy UIs out of the box; React wins on ecosystem + concurrent rendering at scale.

17. **Typeahead: how do you hold p99 < 100 ms at billions of queries?** Precompute top-k at each trie/FST node so a request is *walk + read* (no request-time DFS), shard the index by prefix, and front it with a hot-prefix cache (cache-aside, TTL minutes). *Key driver:* the < 100 ms budget forces precompute + cache + edge.

18. **Why rank typeahead by frequency, not lexicographically?** Users expect popular completions ("facebook" before "facsimile"); rank by historical query frequency. *Extend:* blend recency (trending), personalization, and geography into the score; keep k small (~5–10) so each node's payload stays tiny.

19. **How is the typeahead index refreshed without serving a half-built tree?** Offline pipeline: query logs → Kafka → Flink/Spark aggregation → per-prefix top-k → rebuild a *versioned blob* → serving shards do an **atomic pointer flip**. *Why:* readers see only the old or the new index, never a partial one. Never mutate the serving index in place.

20. **(Curveball) The product wants instant typo tolerance in suggestions.** Add fuzzy matching: an FST with edit-distance traversal (a Levenshtein automaton, Lucene-style) or precomputed common-typo expansions. *Constraint:* cap the fan-out so the p99 budget still holds.

---

## Day 6 — Timed DSA + GitHub Polish + Consolidation

1. **Why does Kahn's algorithm detect a cycle?** Only zero-in-degree nodes ever enter the queue; a cycle's nodes never reach in-degree 0 (each has an unprocessed predecessor inside the cycle), so they're never processed → `processed < V` signals a cycle. *Bonus:* detection is free.

2. **Course Schedule — Kahn's vs DFS 3-color, when each?** Both O(V+E). Kahn's (BFS + in-degree) also yields the order directly (LC 210) and avoids deep recursion. DFS 3-color is terser and natural if you're already doing post-order work. *Gotcha:* the DFS order must be reversed.

3. **Why seed the queue with all rotten oranges in LC 994?** It's multi-source BFS — all sources spread simultaneously, so BFS levels map exactly to elapsed minutes. *Why not single-source:* per-orange BFS would miscount time. *Edge cases:* no fresh → 0; walled-off fresh → -1.

4. **How do you know when a graph problem is union-find vs BFS/DFS?** Union-find for pure connectivity / undirected cycle / online edge additions; BFS/DFS when you need the actual path, distance, or to process level by level. *Tell:* "components" or "connected?" ⇒ DSU.

5. **At-most-once vs at-least-once vs exactly-once — the configs and the catch.** `acks=0` + commit-before-process = at-most-once (can lose); `acks=all` + commit-after-process = at-least-once (can duplicate); idempotent producer + transactions + `read_committed` = exactly-once — **but only Kafka→Kafka**. *Catch:* any external side effect drops you back to at-least-once + idempotent consumer.

6. **Why is "exactly-once to our database" the wrong claim?** The DB write and the Kafka offset commit are separate systems with no shared transaction; you can't commit both atomically. *Real systems:* at-least-once delivery + an idempotent consumer.

7. **A recruiter opens your GitHub — what makes them keep reading?** A profile README with a clear cloud-native + LLM narrative, 3–6 pinned quality repos, and one showcase repo with a real README, an architecture diagram, and `docker-compose up` in < 5 commands — signals "ships software," not "did coursework."

8. **Walk the full async LLM job design in one breath.** Client → API → DB (jobs + outbox in one tx) → Debezium/scheduler → Kafka → worker pool → LLM call → DB write → SSE/poll back to the client. *Throughout:* at-least-once delivery, so the worker is idempotent on `job_id`.

9. **What's the single most common Outbox bug to call out proactively?** The poller publishes to Kafka then crashes before marking `SENT` → the row republishes → a duplicate. There's no atomicity across Postgres and Kafka — that's the at-least-once boundary, which is why consumers are idempotent from day one.

10. **(Curveball) "Number of Islands on a 100k × 100k grid — does your DFS still work?"** Recursive DFS overflows the stack → switch to iterative BFS/DFS. *Truly distributed:* partition the grid into tiles, flood-fill each, then union-find across tile boundaries to merge islands spanning seams.

11. **Diameter of a binary tree — does the longest path pass through the root?** Not necessarily — it's `max over all nodes of (left height + right height)`, tracked during a height DFS; the root is just one candidate. *Why a global:* you must check the bending path at every node, not only the root.

12. **(Reflection) Look at your showcase repo as a stranger — would you star it?** If the README doesn't explain the problem in one paragraph, lacks a diagram, or won't run in five commands, the answer is no — fix those three before moving on. *Signal:* runnable + documented beats feature-complete-but-opaque.

---

*[← Week 3](README.md)*
