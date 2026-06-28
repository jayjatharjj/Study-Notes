# Week 3 · Day 2 — Tue Jul 14 — Tree Construction + LCA + Caching / Redis

> Yesterday's traversals were the alphabet; today you write sentences. **Tree construction** (build a tree *from* its traversals) and the two **hard** tree problems — Max Path Sum and Serialize/Deserialize — are where interviewers separate "knows the API" from "thinks in recursion." All three reuse one idea: a traversal *carries* enough information to reconstruct or summarize a tree. On the systems side, **caching** is the highest-leverage latency lever in any read-heavy service. The answer that scores names the *write strategy*, its *failure mode*, and *why your TTL is the number it is* — not "I put it in Redis."

📌 **Study today:** Tree construction + hard trees — build from pre+inorder, max path sum, serialize/deserialize (LC 105, 124, 297) · LCA recap (LC 236, 235) · caching strategies + Redis + LFU/LRU + cache-aside/write-through · SQL-vs-NoSQL framework · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Construction + Hard Trees (~2.5 hr)

### Theory deep-dive

A traversal is a *lossy* serialization — but the right *pair* of traversals is lossless. The key facts:

| You have | Can you rebuild the tree? | Why |
|---|---|---|
| **Pre-order + In-order** | ✅ unique | pre-order gives the root; in-order splits left/right |
| **Post-order + In-order** | ✅ unique | post-order's *last* element is the root; in-order splits |
| **Pre-order + Post-order** | ⚠️ ambiguous for general binary trees | can't tell a single child's side (unique only for *full* trees) |
| **Pre-order alone (with null markers)** | ✅ unique | the nulls restore the shape — this is LC 297 |

**The construction engine (LC 105):** the *first* element of pre-order is always the root. Find that value in the in-order array — everything to its **left** is the left subtree's in-order, everything to its **right** is the right subtree's in-order. The *sizes* of those slices tell you how to split the pre-order array too. Recurse. The only performance trap: finding the root in in-order is O(n) if you scan, making the whole build O(n²) on a skewed tree — fix it with a `HashMap<value, inorderIndex>` for O(1) lookup, giving O(n) total.

**The "return-a-value-while-updating-a-global" pattern (LC 124):** identical machinery to yesterday's Diameter. A post-order DFS returns the best *downward* path through a node (it can only go down *one* side to remain a path you can extend), while a side-effect global captures the best path that *bends* at the node (uses *both* sides). Clamp negative child gains to 0 — a negative branch is never worth including.

```java
class TreeNode { int val; TreeNode left, right; TreeNode(int v){ val = v; } }
```

### Worked example — LC 105 Construct Binary Tree from Preorder and Inorder

The clean version uses a moving pre-order pointer plus in-order index bounds — no array copying.

```java
// O(n) time, O(n) space (the map + recursion stack).
private int preIdx = 0;
private int[] preorder;
private Map<Integer, Integer> inPos = new HashMap<>();

public TreeNode buildTree(int[] preorder, int[] inorder) {
    this.preorder = preorder;
    for (int i = 0; i < inorder.length; i++) inPos.put(inorder[i], i);  // value -> in-order index
    return build(0, inorder.length - 1);
}

private TreeNode build(int inLeft, int inRight) {
    if (inLeft > inRight) return null;              // empty slice -> no node
    int rootVal = preorder[preIdx++];               // next root in pre-order
    TreeNode root = new TreeNode(rootVal);
    int mid = inPos.get(rootVal);                   // O(1) split point in in-order
    root.left  = build(inLeft, mid - 1);            // MUST build left first:
    root.right = build(mid + 1, inRight);           // pre-order is Root, Left..., Right...
    return root;
}
//   pre = [3,9,20,15,7]   in = [9,3,15,20,7]   ->        3
//                                                       / \
//                                                      9   20
//                                                         /  \
//                                                        15   7
```

**The load-bearing detail:** you build the **left subtree before the right**, because pre-order lays out `Root, [entire left subtree], [entire right subtree]` — the moving `preIdx` must consume all of the left before it reaches the right's root. Swap the two lines and the tree is silently wrong.

**Complexity:** O(n) time, O(n) space. Target: ≤ 15 min.

### Practice set

**1. LC 105 — Construct Binary Tree from Preorder and Inorder** (Medium)
- **Approach:** the moving-`preIdx` + in-order `HashMap` skeleton above; build left before right.
- **Key insight:** pre-order[0] is the root; its position in in-order partitions the rest into left/right subtrees of known size.
- **Complexity:** O(n) / O(n). **Follow-up:** LC 106 (post-order + in-order) — post-order's *last* element is the root, and you build **right before left** (post-order is `Left, Right, Root`, so consume from the back).

**2. LC 124 — Binary Tree Maximum Path Sum** (Hard)
- **Approach:** post-order DFS. Each call returns the best path *starting at this node and going down one branch*; a global captures the best path that bends here (node + both branch gains).
```java
private int best = Integer.MIN_VALUE;
public int maxPathSum(TreeNode root) {
    gain(root);
    return best;
}
private int gain(TreeNode n) {
    if (n == null) return 0;
    int left  = Math.max(0, gain(n.left));   // clamp negative branches to 0
    int right = Math.max(0, gain(n.right));
    best = Math.max(best, n.val + left + right);  // path that bends at n
    return n.val + Math.max(left, right);         // extendable path: one side only
}
```
- **Key insight:** the *returned* value can use only one child (so a parent can extend it into a path); the *candidate* answer uses both children (a path that turns at `n`). Initialize `best` to `Integer.MIN_VALUE` — an all-negative tree's answer is its largest single node.
- **Complexity:** O(n) / O(h). **Follow-up:** return the actual path nodes, not just the sum (track the bending node and reconstruct).

**3. LC 297 — Serialize and Deserialize Binary Tree** (Hard)
- **Approach:** pre-order with explicit null markers. Serialize writes `val,` for a node and `#,` for null; deserialize consumes the token stream recursively in the same order.
```java
public String serialize(TreeNode root) {
    StringBuilder sb = new StringBuilder();
    ser(root, sb);
    return sb.toString();
}
private void ser(TreeNode n, StringBuilder sb) {
    if (n == null) { sb.append("#,"); return; }
    sb.append(n.val).append(',');
    ser(n.left, sb);
    ser(n.right, sb);
}
public TreeNode deserialize(String data) {
    Deque<String> tokens = new ArrayDeque<>(Arrays.asList(data.split(",")));
    return de(tokens);
}
private TreeNode de(Deque<String> tokens) {
    String t = tokens.poll();
    if (t.equals("#")) return null;
    TreeNode n = new TreeNode(Integer.parseInt(t));
    n.left  = de(tokens);   // same pre-order order as serialize
    n.right = de(tokens);
    return n;
}
```
- **Key insight:** the null markers are what make pre-order *alone* lossless — they encode the tree's shape. Serialize and deserialize must walk in the **same order** (both pre-order here). A `Deque` (or an index pointer) gives the "consume next token" stream cleanly.
- **Complexity:** O(n) / O(n). **Follow-up:** LC 449 — serialize a *BST* more compactly (you can drop null markers and reconstruct from bounds, since the BST property restores the shape).

**4. LC 236 — Lowest Common Ancestor (general tree)** (Medium, recap)
- **Approach:** post-order; return the node if it equals `p` or `q`; if both sides return non-null, *this* node is the LCA; else bubble up the non-null side.
```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    TreeNode l = lowestCommonAncestor(root.left, p, q);
    TreeNode r = lowestCommonAncestor(root.right, p, q);
    if (l != null && r != null) return root;   // p and q split here
    return l != null ? l : r;
}
```
- **Key insight:** if one target is the ancestor of the other, the match returns that node and the sibling branch returns null — the parent propagates it correctly without a "found both" flag.
- **Complexity:** O(n) / O(h).

**5. LC 235 — Lowest Common Ancestor (BST)** (Medium, recap)
- **Approach:** exploit ordering — walk down; the first node where `p` and `q` fall on opposite sides (or one equals the node) is the LCA.
```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    while (root != null) {
        if (p.val < root.val && q.val < root.val)      root = root.left;
        else if (p.val > root.val && q.val > root.val) root = root.right;
        else return root;
    }
    return null;
}
```
- **Key insight:** the BST property turns an O(n) search into an O(h) guided walk, O(1) space iteratively.
- **Complexity:** O(h) / O(1). **Common mistake:** ignoring the BST property and reusing the LC 236 post-order.

---

## Block B — Caching Strategies + Redis + Eviction (~2 hr)

> Cross-ref: `../../07-collections.md` (an LRU cache is a `LinkedHashMap` with `accessOrder=true` and an overridden `removeEldestEntry`; Redis is the distributed version of that idea). `../../06-concurrency-and-collections.md` (cache stampede is a thundering-herd concurrency problem — a per-key lock is the same `synchronized`/`Lock` reasoning).

### The mental model

A cache is a fast, *small*, *lossy* copy of data that lives closer to the reader. Every caching decision is two questions: **(1) how does a write reach the cache and the source of truth** (the four write strategies), and **(2) what do you throw away when full** (eviction). Get those two right and you've answered 80% of caching interviews.

### The four write strategies

| Strategy | Write path | Read path | Best for | Failure mode |
|---|---|---|---|---|
| **Cache-Aside** (lazy) | app writes DB directly; cache filled on read miss | check cache → miss → read DB → fill cache → return | read-heavy, tolerate slight staleness | **stampede** on cold start / TTL expiry under load |
| **Write-Through** | write cache + DB **synchronously** | always served from cache | consistency-critical reads | doubled write latency; caches maybe-never-read data |
| **Write-Back** (write-behind) | write cache only; **async** flush to DB | always served from cache | write-heavy, latency-sensitive | data loss if the node dies before flush (CPU L1/L2 work this way) |
| **Write-Around** | write straight to DB, bypass cache | cache populated on reads only | write-once / read-rarely (logs, audit) | high *first-read* latency |

**Cache-Aside is the default** for application data — it's simple, the cache only holds read-demanded data, and a cache outage degrades gracefully to DB reads. Its one sharp edge is the **stampede / thundering herd**: when a hot key expires (or on cold start), thousands of concurrent requests all miss and hammer the DB at once. Mitigations:
- **Per-key mutex (request coalescing):** the first miss takes a lock and recomputes; others wait and read the freshly-cached value.
- **Probabilistic early expiration:** each reader rolls a dice that gets likelier as the TTL approaches, so *one* reader refreshes the key slightly early while it's still serving the cached value.
- **Stale-while-revalidate:** serve the stale value and refresh in the background.

### Eviction: LRU vs LFU (and the Redis policies)

When `maxmemory` is hit, Redis evicts per its `maxmemory-policy`:

| Policy | Evicts | Use when |
|---|---|---|
| `noeviction` | nothing — writes error | you'd rather fail than lose data (a queue) |
| `allkeys-lru` / `volatile-lru` | least-**recently**-used (any key / only keys with a TTL) | general-purpose; temporal locality |
| `allkeys-lfu` / `volatile-lfu` | least-**frequently**-used | non-uniform access — some keys stay hot regardless of recency |
| `allkeys-random` / `volatile-ttl` | random / soonest-to-expire | niche |

**LRU vs LFU is the headline trade-off.** LRU tracks *recency* (when last touched) — great for "working set" access where recently-used implies soon-to-be-used. LFU tracks *frequency* — better when a stable set of keys is persistently hot and a one-off scan of cold keys would wrongly evict them under LRU (the classic "sequential scan pollutes the cache" problem). Redis 4+ implements LFU with a *probabilistic counter that decays over time*, so an item that was hot last week but cold now ages out.

### Implementing an O(1) LRU yourself (the LC 146 you should know cold)

```java
// HashMap for O(1) lookup + doubly-linked list for O(1) recency reordering.
class LRUCache {
    private final int cap;
    private final Map<Integer, Node> map = new HashMap<>();
    private final Node head, tail;                 // sentinels: head=MRU side, tail=LRU side
    private static class Node { int k, v; Node prev, next; Node(int k,int v){this.k=k;this.v=v;} }

    LRUCache(int capacity) {
        cap = capacity;
        head = new Node(0,0); tail = new Node(0,0);
        head.next = tail; tail.prev = head;
    }
    public int get(int key) {
        Node n = map.get(key);
        if (n == null) return -1;
        moveToFront(n);                            // touched -> most recent
        return n.v;
    }
    public void put(int key, int value) {
        Node n = map.get(key);
        if (n != null) { n.v = value; moveToFront(n); return; }
        if (map.size() == cap) {                   // evict LRU (just before tail)
            Node lru = tail.prev;
            remove(lru); map.remove(lru.k);
        }
        Node fresh = new Node(key, value);
        map.put(key, fresh); addFront(fresh);
    }
    private void remove(Node n){ n.prev.next = n.next; n.next.prev = n.prev; }
    private void addFront(Node n){ n.next = head.next; n.prev = head; head.next.prev = n; head.next = n; }
    private void moveToFront(Node n){ remove(n); addFront(n); }
}
```
Both `get` and `put` are O(1). In an interview, mention that `LinkedHashMap(capacity, 0.75f, true)` with an overridden `removeEldestEntry` gives the same thing in ~10 lines — but interviewers usually want the hand-rolled version to test the doubly-linked-list reasoning.

### Redis persistence and topology

- **Persistence — RDB vs AOF:** **RDB** takes point-in-time snapshots (compact, fast restart, but you lose everything since the last snapshot on a crash). **AOF** appends every write to a log; `appendfsync everysec` (the common setting) bounds loss to ~1 second. Many run *both*. **Pick by data value:** a JWT blacklist → **AOF** (you can't safely un-invalidate a token; ~1s loss tolerable). A pure S3-URL cache → **RDB or none** (it's reconstructible from the source of truth).
- **Topology — Sentinel vs Cluster:** **Sentinel** gives HA for a *single primary* — it monitors and performs automatic failover (promotes a replica). **Cluster** gives *horizontal sharding* across 16384 hash slots — for when one node can't hold the dataset or serve the throughput. Sentinel scales *availability*; Cluster scales *capacity*. They solve different problems.

### Project tie-in

- **WebX — the S3-URL cache:** presigned S3 download URLs are expensive to mint and short-lived. The cache is **cache-aside + write-around** (S3 is the source of truth) with a deliberate TTL: the presigned URL lives 15 min, so the Redis entry is set to **13 min**. The 2-minute buffer absorbs clock skew and serve latency — a "valid" cached URL that expires in the client's hand is worse than a slightly shorter TTL. Persistence: **none/RDB** — it's a pure cache.
- **Smart360 — JWT blacklist on logout:** stored in Redis with **AOF** persistence and a TTL equal to the token's remaining lifetime, so revoked tokens auto-clean. Losing this on a crash would re-validate logged-out tokens — hence AOF, not RDB.
- **Deep Fathom — hot dashboard aggregates:** cache-aside with a per-key mutex on the expensive aggregate query to prevent stampede when the 5-minute TTL lapses during peak load.

---

## Block C — SQL vs NoSQL Decision Framework (~1.5 hr)

### Theory

This is a **framework, not a religion** — the senior signal is reasoning from *access patterns*, not reciting "NoSQL scales." The core distinction: a **relational schema + indexes optimize for unknown future queries** (you can ad-hoc join and filter anything later); **NoSQL optimizes for known query patterns you design the schema around** (fast for those, awkward for anything else).

### When relational (Postgres) wins — which is *usually*

- **Multi-entity joins** — the data is genuinely relational (orders ↔ users ↔ products).
- **ACID transactions across rows/tables** — transfer money, transactional authorization.
- **Ad-hoc / analytical queries** — you don't know tomorrow's question.
- **Strong consistency by default** — read-your-writes without ceremony.

Smart360 and Deep Fathom are Postgres for exactly these reasons (row-level security, ~50-table joins, transactional authorization). **The line to use:** *"I start with Postgres for ~95% of cases; I move a slice to NoSQL when one access pattern consistently hits relational limits — polyglot persistence, not replacement."*

### The NoSQL families (name the right one for the pattern)

| Family | Examples | Sweet spot | Don't use for |
|---|---|---|---|
| **Key-value** | Redis, DynamoDB | O(1) lookup by key — cache, sessions, rate-limiter state, JWT blacklist | anything needing queries other than by key |
| **Document** | MongoDB | self-contained JSON aggregates, evolving schema, no joins | data with many cross-entity relationships |
| **Wide-column** | Cassandra, HBase | massive write throughput, time-series, partition-key reads of *known* patterns (event/audit logs, metrics, IoT) | ad-hoc queries / joins |
| **Graph** | Neo4j | cheap multi-hop traversal (social graphs, permission hierarchies) where SQL self-joins explode at 4+ hops | simple tabular data |

### Polyglot persistence

The mature answer: **Postgres for the transactional core, Redis for hot/ephemeral state, and a wide-column or document store for one high-volume access pattern** — keeping the seam consistent via events/Outbox (Thursday's topic). You don't replace your database; you add a specialized store for the *one* pattern that's outgrowing it.

### A worked decision

> *"A social-media 'like count' — SQL or NoSQL, and what write strategy?"* — The count is read-massively, written-frequently, and a few seconds of staleness is fine. I'd keep the *canonical* count in Postgres (it's relational — tied to a post and user) but serve and increment it in **Redis** (`INCR`, O(1)), flushing to Postgres periodically — effectively **write-back** for the hot counter with the DB as the durable backstop. Losing a few seconds of increments on a Redis crash is acceptable for a like count; it would not be for a payment.

---

## 💻 Practice coding questions

1. **LC 105 Construct from Preorder + Inorder** — moving `preIdx` + in-order `HashMap`; build left before right.
2. **LC 106 Construct from Inorder + Postorder** — post-order's last element is the root; build right before left.
3. **LC 889 Construct from Preorder + Postorder** — note the ambiguity; any valid tree accepted.
4. **LC 124 Max Path Sum** — clamp negative gains to 0; global bend-candidate vs one-sided return.
5. **LC 543 Diameter** (recap) — same return-height-update-global shape as 124.
6. **LC 297 Serialize/Deserialize** — pre-order with `#` null markers; consume tokens via `Deque`.
7. **LC 449 Serialize BST** — drop null markers; reconstruct from BST bounds.
8. **LC 236 LCA (general)** — post-order, both-sides-non-null → current.
9. **LC 235 LCA (BST)** — guided walk, O(h)/O(1).
10. **LC 146 LRU Cache** — `HashMap` + doubly-linked list, O(1) get/put.
11. **LC 460 LFU Cache** — frequency buckets, each a recency list; evict min-freq's LRU.
12. **LC 1382 Balance a BST** — in-order to sorted array, then build balanced (construction practice).
13. **LC 109 Sorted List to BST** — in-order construction with a moving list pointer.

> For each tree problem, say the traversal type aloud first. Re-time LC 105 and LC 146 — target ≤ 15 min each.

---

## 🎤 Interview questions

1. **Which two traversals uniquely rebuild a binary tree, and which pair is ambiguous?** Pre+in and post+in are unique. Pre+post is ambiguous for general binary trees (can't place a single child's side) — unique only for *full* trees. Pre-order alone *with null markers* is also unique (LC 297).
2. **In LC 105, why build the left subtree before the right?** Pre-order is `Root, [left subtree], [right subtree]`; the moving `preIdx` must consume the entire left subtree before reaching the right's root. Swapping the order silently produces a wrong tree.
3. **Why is the naive LC 105 O(n²), and how do you fix it?** Scanning in-order to find each root is O(n) per node. A `HashMap<value, inorderIndex>` makes the split O(1), giving O(n) total.
4. **In Max Path Sum, why does the returned value differ from the answer candidate?** The returned value can use only *one* child so a parent can extend it into a longer path; the candidate uses *both* children (a path that bends at the node and can't be extended further). Clamp negative child gains to 0.
5. **Why do null markers make pre-order alone enough to deserialize?** They encode the tree's *shape* — without them, `[1,2]` could be 2-as-left or 2-as-right. The markers disambiguate every absent child.
6. **Name the four cache write strategies and one failure mode each.** Cache-aside (stampede on expiry), write-through (doubled write latency), write-back (loss on crash before flush), write-around (slow first read).
7. **What is a cache stampede and how do you prevent it?** A hot key expires and thousands of concurrent misses hammer the DB at once. Mitigate with a per-key mutex (request coalescing), probabilistic early expiration, or stale-while-revalidate.
8. **LRU vs LFU — when each?** LRU for temporal locality (recently used ⇒ soon used); LFU for a stable hot set where a one-off scan shouldn't evict persistently-hot keys. Redis LFU uses a decaying probabilistic counter so stale-hot items age out.
9. **Implement an O(1) LRU — what data structures?** `HashMap` for O(1) lookup + a doubly-linked list for O(1) recency reordering; sentinels avoid null checks. Or `LinkedHashMap(cap, 0.75f, true)` + `removeEldestEntry`.
10. **Redis RDB vs AOF — pick for a JWT blacklist vs a URL cache.** JWT blacklist → AOF (`appendfsync everysec`, ~1s loss tolerable; you can't un-invalidate a token). URL cache → RDB or none (reconstructible from the source of truth).
11. **Redis Sentinel vs Cluster — what does each solve?** Sentinel = HA/automatic failover for a single primary (scales availability). Cluster = horizontal sharding across 16384 hash slots (scales capacity). Different problems.
12. **Why set the S3-URL cache TTL *below* the presigned-URL expiry?** A 15-min URL cached for 13 min leaves a 2-min buffer for clock skew and serve latency. A cached URL that expires in the client's hand is worse than re-minting slightly sooner.
13. **SQL vs NoSQL — your decision framework?** Start from access patterns. Relational optimizes for unknown future queries (joins, ACID, ad-hoc); NoSQL optimizes for known patterns you model the schema around. Postgres for ~95%; move one slice to NoSQL when it consistently hits relational limits — polyglot, not replacement.
14. **Match each NoSQL family to a use case.** Key-value → cache/sessions/rate-limiter; document → self-contained evolving aggregates; wide-column → high-write time-series with known partition-key reads; graph → multi-hop relationship traversal where SQL self-joins explode.
15. **Like count — SQL or NoSQL, and write strategy?** Canonical count in Postgres; serve/increment in Redis (`INCR`, O(1)); periodic flush to the DB — write-back for the hot counter, DB as durable backstop. A few seconds' loss on crash is acceptable for likes, not for payments.
16. **What is write-around and when is it right?** Writes go straight to the DB, bypassing the cache; the cache fills only on reads. Right for write-once/read-rarely data (audit logs) and for caches over an authoritative store (the S3-URL cache).
17. **Cache-aside on a cache outage — what happens?** Reads degrade gracefully to the DB (correctness preserved, latency rises) — a key reason cache-aside is the safe default versus write-through where the cache is on the critical write path.
18. **How would you cache an expensive Deep Fathom dashboard aggregate?** Cache-aside with a TTL matched to acceptable staleness (e.g. 5 min) and a per-key mutex so only one request recomputes on expiry while the rest wait for the refilled value.

---

## ✅ Self-check

1. Write the LC 105 signature and base case from memory (`build(inLeft, inRight)`, `if (inLeft > inRight) return null`), and state why left builds before right.
2. In Max Path Sum, give the two distinct expressions: the value returned upward (`n.val + max(left, right)`) vs the answer candidate (`n.val + left + right`).
3. Name the four cache write strategies and one failure mode of each, no notes.
4. Explain LRU vs LFU in two sentences and name the Redis policy for each.
5. Pick a write strategy for a social-media "like count" and justify it in one breath.
6. State the SQL-first framework line and the one condition that moves a slice to NoSQL.

---

*Nav: ← [01-mon-jul-13.md](01-mon-jul-13.md) · [Week 3](README.md) · [03-wed-jul-15.md](03-wed-jul-15.md) →*
