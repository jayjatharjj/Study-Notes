# Week 3 · Day 4 — Thu Jul 16 — Union-Find + Outbox / Saga + Trie

> Union-Find is the structure that turns "are these connected?" from an O(V+E) graph traversal into a near-O(1) lookup — and it's the cleanest tool for undirected-cycle and connected-component problems. The core block is the distributed-systems trio every senior backend interview probes: the **Outbox** pattern (the fix for the dual-write problem), the two **Saga** flavors (choreography vs orchestration), and the brutal honesty around **idempotency / exactly-once**. Block C is the **Trie** — your search-product home turf, and the in-memory core of tomorrow's typeahead design.

📌 **Study today:** Union-Find + advanced graphs (LC 684, 547) · Outbox + Saga + idempotency / exactly-once · Trie (LC 208, 211, 1268) · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Union-Find (path compression · union by rank)

### Theory

Union-Find (Disjoint Set Union, DSU) maintains a partition of `n` elements into disjoint sets and answers two questions fast:

- `find(x)` — which set is `x` in? (returns the set's representative / root)
- `union(x, y)` — merge the sets containing `x` and `y`.

The mental model is a **forest**: each set is a tree, the root is the set's identity. `find` walks to the root; `union` points one root at the other. Two optimizations turn a naive O(n)-per-op forest into near-constant time:

1. **Path compression** — during `find`, re-point every node on the path directly to the root, flattening the tree. One line: `parent[x] = find(parent[x])`.
2. **Union by rank (or size)** — always attach the *shorter* tree under the *taller* one, so trees stay shallow. `rank` is an upper bound on height.

With **both**, each operation is amortized **α(n)** (inverse Ackermann) — effectively a small constant (< 5) for any `n` you'll ever see. With neither it degrades to O(n).

**When to reach for Union-Find:** connected components, undirected cycle detection, "are these two in the same group?", Kruskal's MST, account/email merging, percolation, dynamic connectivity. **When NOT to:** directed-graph cycles (use DFS 3-color or Kahn's — DSU can't express edge direction), shortest path (use BFS/Dijkstra), anything needing *un*-union (DSU has no efficient delete).

### Worked template — write this cold

```java
class UnionFind {
    private final int[] parent;
    private final int[] rank;   // upper bound on tree height
    private int count;          // number of disjoint sets

    UnionFind(int n) {
        parent = new int[n];
        rank   = new int[n];
        count  = n;
        for (int i = 0; i < n; i++) parent[i] = i; // each node is its own root
    }

    // path compression: flatten the path to the root on the way up
    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }

    // union by rank; returns false if already in the same set
    boolean union(int x, int y) {
        int px = find(x), py = find(y);
        if (px == py) return false;            // already connected → would form a cycle
        if (rank[px] < rank[py])      parent[px] = py;
        else if (rank[px] > rank[py]) parent[py] = px;
        else { parent[py] = px; rank[px]++; }  // equal ranks → pick one, bump its rank
        count--;                                // two sets became one
        return true;
    }

    int count() { return count; }
    boolean connected(int x, int y) { return find(x) == find(y); }
}
```

Note the iterative-vs-recursive `find` tradeoff: the recursive form above compresses fully but uses O(path) stack; for adversarial inputs an iterative two-pass `find` (find root, then re-point) avoids deep recursion. Either is fine in interviews — state that you know.

### Worked example — Redundant Connection (LC 684)

A tree on `n` nodes has exactly `n−1` edges; the input has `n` edges, so exactly one creates a cycle. Process edges in order; the **first** edge whose endpoints are already connected is the redundant one.

```java
public int[] findRedundantConnection(int[][] edges) {
    int n = edges.length;
    UnionFind uf = new UnionFind(n + 1); // nodes are 1-indexed
    for (int[] e : edges) {
        if (!uf.union(e[0], e[1])) return e; // endpoints already connected → this edge closes a cycle
    }
    return new int[0]; // unreachable per constraints
}
```

Why it works: `union` returns `false` exactly when `find(u) == find(v)` *before* the merge — meaning a path already exists between `u` and `v`, so adding this edge forms a cycle. Returning the edge satisfies the "return the last edge that can be removed" requirement because we scan in input order. **Target AC: 20 min.**

### Practice set

| # | Problem (LC) | Approach | Key insight | Complexity |
|---|---|---|---|---|
| 1 | Redundant Connection (684) | DSU, return first edge already connected | `union` false ⇒ cycle edge | O(n·α(n)) |
| 2 | Number of Provinces (547) | DSU on the adjacency matrix; count roots | Province = connected component | O(n²·α(n)) |
| 3 | Graph Valid Tree (261) | DSU: tree iff `edges == n−1` AND no `union` fails | Connected + acyclic = tree | O(n·α(n)) |
| 4 | Number of Connected Components (323) | DSU, answer is `count()` | Start at `n`, decrement per real union | O((V+E)·α(n)) |
| 5 | Accounts Merge (721) | DSU over emails, group by root | String→id map, then bucket by `find` | O(NK·α) |
| 6 | (Stretch) Number of Islands II (305) | DSU, add land incrementally, union with 4 neighbors | Online connectivity; track running count | O(k·α) |

**Number of Provinces (LC 547)** — the canonical "count components" rep:

```java
public int findCircleNum(int[][] isConnected) {
    int n = isConnected.length;
    UnionFind uf = new UnionFind(n);
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++)
            if (isConnected[i][j] == 1) uf.union(i, j);
    return uf.count(); // each remaining set is one province
}
```

**Talking point:** "Components and undirected-cycle questions are a DSU tell. I'd reach for BFS/DFS only if I also needed the actual paths or distances — DSU answers *connectivity*, not *route*." **Target AC: 547 in 15 min.**

---

## Block B — Outbox + Saga + Idempotency / Exactly-Once

This block is the heart of the monolith→event-driven story. Interviewers test it because it separates people who *used* a queue from people who *reasoned about delivery guarantees*.

### Idempotency & the exactly-once myth

An **idempotent consumer** produces the same end state whether it processes a message once or N times. This matters because **at-least-once is the real-world default** — any system that commits the offset *after* processing will, on a crash between processing and commit, reprocess.

> **Exactly-once *to an external database* is a myth.** The DB write and the Kafka offset commit are two independent systems with no shared transaction. You cannot atomically "do the work AND commit the offset" across them. Kafka's EOS (`enable.idempotence=true` + `transactional.id` + consumer `isolation.level=read_committed`) is **Kafka→Kafka only** — it makes a read-process-write loop *within Kafka* atomic. The moment your side effect is an external DB, S3, or an email, you are back to at-least-once + idempotent design.

Three idempotency patterns, cheapest to most general:

| Pattern | How | Use when |
|---|---|---|
| **Message-level dedup** | unique `messageId` in payload → `INSERT` into `processed_messages(message_id PK)`; on conflict, skip | Generic, any consumer |
| **Business-level state machine** | `IF status='QUEUED' THEN process ELSE skip` | Natural state column exists |
| **Optimistic lock / claim** | `UPDATE jobs SET status='PROCESSING' WHERE id=? AND status='QUEUED'` — only one consumer's update affects a row | Competing consumers, exactly-one-wins |

```java
// Message-level dedup — the workhorse
@Transactional
public void handle(Event ev) {
    int inserted = jdbc.update(
        "INSERT INTO processed_messages(message_id) VALUES (?) ON CONFLICT DO NOTHING",
        ev.messageId());
    if (inserted == 0) return;        // already processed — idempotent skip
    applyBusinessEffect(ev);          // same tx → effect + dedup record commit together
}
```

The dedup `INSERT` and the business effect must be in the **same transaction**, or a crash between them leaves you either double-applying (effect committed, dedup not) or losing work.

### The Outbox pattern — solving the dual-write problem

The **dual-write problem**: a handler that does `save(entity)` then `publish(event)` performs two I/O operations against two systems. A crash *between* them leaves the DB updated but the event lost (or vice-versa). There is no cross-system transaction.

**Outbox fix:** write the event into an `outbox_events` table **in the same transaction** as the domain change. Now it's a single atomic DB commit — the event can't be lost. A separate process delivers it later.

```sql
CREATE TABLE outbox_events (
    id             UUID PRIMARY KEY,
    aggregate_type VARCHAR(64)  NOT NULL,   -- 'User', 'Order'
    aggregate_id   VARCHAR(64)  NOT NULL,
    event_type     VARCHAR(64)  NOT NULL,   -- 'UserCreated'
    payload        JSONB        NOT NULL,
    status         VARCHAR(16)  NOT NULL DEFAULT 'PENDING',
    created_at     TIMESTAMPTZ  NOT NULL DEFAULT now()
);
CREATE INDEX idx_outbox_poll ON outbox_events(status, created_at) WHERE status = 'PENDING';
```

```java
@Transactional
public User createUser(CreateUserCmd cmd) {
    User user = userRepo.save(new User(cmd));            // domain change
    outboxRepo.save(new OutboxEvent(                     // event — SAME transaction
        UUID.randomUUID(), "User", user.id(),
        "UserCreated", toJson(user), "PENDING"));
    return user; // one commit → user + event are atomic; nothing can be half-written
}
```

**Two delivery mechanisms:**

| | Polling publisher | Debezium (CDC) |
|---|---|---|
| How | `@Scheduled` reads `status='PENDING'`, publishes to Kafka, marks `SENT` | Tails the Postgres **WAL** (logical replication) → Kafka, no app polling |
| Latency | Poll interval (seconds) | Near real-time |
| DB load | Extra query each cycle | None on the table (reads the log) |
| Ops | Trivial — just a scheduled bean | More moving parts (Kafka Connect, Debezium connector) |
| Use when | Low/medium volume, simplicity wins | High throughput, low latency, no polling load |

```java
@Scheduled(fixedDelay = 1000)
@Transactional
public void publishOutbox() {
    List<OutboxEvent> batch = outboxRepo.findTop100ByStatusOrderByCreatedAt("PENDING");
    for (OutboxEvent e : batch) {
        kafka.send(e.aggregateType(), e.aggregateId(), e.payload()); // key = aggregateId → per-aggregate ordering
        e.markSent();                                                 // dirty-checked update
    }
}
```

**The at-least-once boundary you must name:** if the poller publishes to Kafka and the pod dies *before* `markSent()`, the row stays `PENDING` and is republished next cycle → **duplicate**. There is no atomicity across Postgres and Kafka. The fix isn't "make it exactly-once" — it's "every consumer is idempotent from day one." Index `(status, created_at)` so the poll query is an index range scan, not a full-table sequential scan.

> **Project tie-in (Smart360):** the monolith had a real dual-write bug — authorization events were lost on pod restart because the handler published to the message broker *after* the DB commit but the pod was killed in between. Outbox made the event part of the same Postgres commit; a Debezium connector tailed the WAL. The 30% efficiency gain came partly from removing the synchronous Notification call (P95 850ms → 590ms) and moving it behind the outbox.

### Saga — distributed transactions without 2PC

A **Saga** is a sequence of local transactions across services; if a step fails, earlier steps are undone by **compensating transactions** (semantic undo, not a DB rollback — you can't un-capture a payment, you issue a refund). Design the compensations *before* the forward path. Two coordination styles:

| | **Choreography** | **Orchestration** |
|---|---|---|
| Coordinator | None — each service reacts to events | Central orchestrator (Temporal, Step Functions, custom state machine) sends commands, awaits replies |
| Coupling | Loose; no SPOF | Orchestrator is a bottleneck / SPOF |
| Visibility | Hard — flow is implicit across event logs | Full — saga state lives in one place |
| Debugging | Distributed-tracing nightmare at scale | Central, with targeted retries |
| Use when | ≤ 3 stable services you own | ≥ 4 services, external systems (payment), audit/compliance |

> **Project tie-in:** Smart360 used **choreography** for notification extraction (2 services, no complex rollback — Notification being 500ms late didn't matter). The switch to orchestration is forced by growing to 4+ cross-dependent services (compliance audit ← CRM sync ← analytics), where pure-event debugging becomes untraceable and you need saga-state visibility + targeted retries.

### Gotchas

- **Forgetting same-transaction outbox write** — defeats the entire pattern; you've just moved the dual write.
- **Claiming "exactly-once to the DB"** — instant credibility loss; say "at-least-once + idempotent consumer."
- **No compensation design** — a Saga without defined undos is a half-applied distributed transaction waiting to happen.
- **Unbounded outbox growth** — archive/delete `SENT` rows (partition by month, or a retention job) or the table and its index bloat.

---

## Block C — Trie (Prefix Tree)

### Theory

A **Trie** is a tree specialized for *prefix* queries. Each node holds children keyed by character plus an `isEnd` flag marking a complete word. A word of length `L` is a root-to-node path of length `L`. Insert / search / `startsWith` are all **O(L)** — independent of how many words are stored — which is why it beats a `HashSet<String>` for prefix work (a hash set can do exact membership in O(L) too, but cannot answer "which words start with `fac`" without scanning everything).

Two node layouts: a fixed `TrieNode[26]` array (fast, lowercase-only, ~constant per node) or a `Map<Character, TrieNode>` (memory-lean for sparse / Unicode alphabets). Use the array in interviews unless told otherwise.

This is your search-product home turf — autocomplete, spell-check, IP routing tables, and the in-memory core of tomorrow's typeahead HLD.

### Worked example — Implement Trie (LC 208)

```java
class Trie {
    private static class Node {
        Node[] children = new Node[26];
        boolean isEnd;
    }
    private final Node root = new Node();

    public void insert(String word) {
        Node cur = root;
        for (char c : word.toCharArray()) {
            int i = c - 'a';
            if (cur.children[i] == null) cur.children[i] = new Node();
            cur = cur.children[i];
        }
        cur.isEnd = true;
    }

    public boolean search(String word)      { Node n = walk(word); return n != null && n.isEnd; }
    public boolean startsWith(String prefix) { return walk(prefix) != null; } // no isEnd check

    private Node walk(String s) {
        Node cur = root;
        for (char c : s.toCharArray()) {
            int i = c - 'a';
            if (cur.children[i] == null) return null;
            cur = cur.children[i];
        }
        return cur;
    }
}
```

The only difference between `search` and `startsWith` is the terminal `isEnd` check — say that out loud; it's the whole point of the flag. **Target AC: 15 min.**

### Practice set

| # | Problem (LC) | Approach | Key insight | Complexity |
|---|---|---|---|---|
| 1 | Implement Trie (208) | array children + `isEnd` | search checks `isEnd`, startsWith doesn't | O(L) per op |
| 2 | Add and Search Word — `.` wildcard (211) | Trie + DFS | `.` recurses into *all* children; letter follows one | O(26^d · ...) worst |
| 3 | Search Suggestions System (1268) | Trie or sorted+binary-search; top-3 per prefix | DFS collects ≤3 lexicographically smallest | O(Σ|word|) build |
| 4 | (Stretch) Word Search II (212) | Trie of words + DFS over the board | Prune dead branches with the trie | O(cells · 4^L) |
| 5 | (Stretch) Replace Words (648) | Trie of roots, stop at first `isEnd` | Shortest matching root wins | O(text length) |

**Add and Search (LC 211)** — the `.` wildcard forces DFS instead of a linear walk:

```java
class WordDictionary {
    private static class Node { Node[] ch = new Node[26]; boolean isEnd; }
    private final Node root = new Node();

    public void addWord(String w) {
        Node cur = root;
        for (char c : w.toCharArray()) {
            int i = c - 'a';
            if (cur.ch[i] == null) cur.ch[i] = new Node();
            cur = cur.ch[i];
        }
        cur.isEnd = true;
    }

    public boolean search(String w) { return dfs(w, 0, root); }

    private boolean dfs(String w, int idx, Node node) {
        if (node == null) return false;
        if (idx == w.length()) return node.isEnd;
        char c = w.charAt(idx);
        if (c == '.') {                                  // wildcard → try every child
            for (Node nxt : node.ch)
                if (dfs(w, idx + 1, nxt)) return true;
            return false;
        }
        return dfs(w, idx + 1, node.ch[c - 'a']);        // normal char → one path
    }
}
```

**Talking point (1268):** "This is the in-memory core of autocomplete. In production I'd back it with a trie/FST (Lucene-style), store **precomputed top-k** at each prefix node so a lookup is walk-to-node-then-read, not a request-time DFS, and bound the fan-out." That sentence bridges directly into tomorrow's typeahead design. **Target AC: 211 in 20 min, 1268 in 25 min.**

---

## 💻 Practice coding questions

1. Implement Union-Find with path compression + union by rank from a blank file (≤10 min).
2. Redundant Connection (LC 684) — return the cycle edge.
3. Number of Provinces (LC 547) — count components via DSU.
4. Graph Valid Tree (LC 261) — connected + acyclic check (DSU and BFS variants).
5. Number of Connected Components in an Undirected Graph (LC 323).
6. Accounts Merge (LC 721) — DSU over emails, then bucket by root.
7. Implement Trie (LC 208) — insert / search / startsWith.
8. Add and Search Word with `.` wildcard (LC 211).
9. Search Suggestions System (LC 1268) — top-3 completions per prefix.
10. Write a same-transaction Outbox `createUser` method (entity + outbox event).
11. Write an idempotent consumer using a `processed_messages` table with `ON CONFLICT DO NOTHING`.
12. Write the optimistic-lock claim: `UPDATE jobs SET status='PROCESSING' WHERE id=? AND status='QUEUED'`.
13. (Stretch) Number of Islands II (LC 305) — incremental DSU with a running component count.
14. (Stretch) Word Search II (LC 212) — Trie-pruned board DFS.

---

## 🎤 Interview questions

1. **Amortized complexity of Union-Find with path compression + union by rank?** ≈ O(1) per op — inverse Ackermann α(n), a small constant for any realistic `n`. Without both optimizations it degrades toward O(n).
2. **Path compression vs union by rank — what does each fix?** Compression flattens the path *during* `find`; rank keeps trees shallow *during* `union`. They attack tree height from the two operations; together they give α(n).
3. **DSU vs BFS/DFS for connected components — when each?** DSU for pure connectivity / online edge additions / cycle detection on undirected graphs; BFS/DFS when you also need the actual path, distance, or to process by level.
4. **Why can't Union-Find detect a cycle in a *directed* graph?** It only models undirected "same set" membership — it has no notion of edge direction. Use DFS 3-color or Kahn's for directed cycles.
5. **What is the dual-write problem and how does Outbox solve it?** Saving an entity and publishing an event are two I/O ops to two systems; a crash between them loses one. Outbox writes the event into a table in the *same DB transaction*, making it a single atomic commit; a separate process delivers it.
6. **Polling publisher vs Debezium for the outbox?** Polling is a `@Scheduled` query — simple, but adds DB load and poll-interval latency. Debezium tails the WAL — near real-time, no polling load, but more operational complexity (Kafka Connect + connector).
7. **Outbox poller publishes to Kafka, then the DB dies before marking SENT — what happens?** Row stays PENDING → republished next cycle → duplicate. No atomicity across Postgres + Kafka, so you accept at-least-once and make consumers idempotent.
8. **Why is exactly-once to an external DB a myth?** The work (DB write) and the offset commit are separate systems with no shared transaction; you can't atomically commit both. Kafka EOS is Kafka→Kafka only. Real systems do at-least-once + idempotent effects.
9. **Idempotent producer vs idempotent consumer?** Producer (`enable.idempotence=true`): the broker dedups retried writes via a per-partition sequence number. Consumer: your code dedups *effects* on the downstream DB. Independent guarantees.
10. **Three ways to make a consumer idempotent?** Message-level (`processed_messages(message_id)` with `ON CONFLICT DO NOTHING`), business-level state machine (`IF status='QUEUED'`), optimistic-lock claim (`UPDATE ... WHERE status='QUEUED'` — one writer wins).
11. **Choreography vs orchestration Saga — pick one for a 2-service notification flow.** Choreography — loose coupling, no SPOF, and being slightly late doesn't matter. Switch to orchestration at 4+ cross-dependent services or when you need central saga-state visibility / compliance audit.
12. **What is a compensating transaction and when do you design it?** A semantic undo (refund, not "un-charge"); design it *before* the forward path, because some steps have no native rollback.
13. **Why is the outbox poll query indexed on `(status, created_at)`?** So it's an index range scan over only PENDING rows in creation order, not a full-table scan that grows with the table.
14. **(Curveball) "Just use a bigger DB instead of Kafka for our event queue."** For low volume a DB-backed queue with `SELECT FOR UPDATE SKIP LOCKED` works — that *is* Outbox-as-queue. At scale, DB polling loads the primary, gives no replay, and no fan-out to independent consumer groups. Start simple, introduce Kafka past a volume threshold.
15. **Trie vs HashSet for a dictionary?** Both do exact membership in O(L). Only the Trie answers prefix queries ("all words starting with `fac`") without scanning every key — and shares storage across common prefixes.
16. **Why does the `.` wildcard force DFS in LC 211?** A literal char follows exactly one child; `.` could match any of 26, so you must branch into all non-null children and succeed if any path completes — that's a DFS, not a linear walk.
17. **How would you scale the LC 1268 trie to production typeahead?** Precompute top-k at each prefix node (read, don't DFS, at request time), shard the trie by leading characters, and refresh it offline from query-frequency logs — exactly tomorrow's HLD.
18. **DSU `union` returns false — what does that tell you in LC 684?** The two endpoints were already in the same set, so a path already existed; this edge closes a cycle and is the redundant one.

---

## ✅ Self-check

1. Write Union-Find (path compression + union by rank) from a blank file in ≤10 min; state the amortized complexity (α(n)).
2. Draw the Outbox + CDC flow end-to-end and name the exact point where a duplicate can occur (publish succeeds, `markSent` doesn't).
3. Explain in one sentence why exactly-once *to a database* is impossible, and what you do instead.
4. Implement Trie insert/search/startsWith cold, and state the one-line difference between `search` and `startsWith`.
5. Pick choreography vs orchestration for a 5-service payment+audit flow and justify in 30 seconds.

---

*Nav: ← [Day 3 (Wed Jul 15)](03-wed-jul-15.md) · [Week 3](README.md) · [Day 5 (Fri Jul 17)](05-fri-jul-17.md) →*
