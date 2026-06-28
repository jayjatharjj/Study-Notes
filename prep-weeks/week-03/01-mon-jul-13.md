# Week 3 · Day 1 — Mon Jul 13 — Tree Traversals + BST + DB Indexing

> The day trees stop being scary. Two-thirds of "binary tree" interview problems are a traversal in disguise — once the three DFS orders and level-order BFS are muscle memory, the rest is bookkeeping. On the systems side, DB indexing is the single highest-leverage backend topic: it's why Smart360's report query went from 60s to 2–3s, and the answer that scores names *which columns* and *why*, not "I added an index."

📌 **Study today:** Tree traversals + BST — inorder/level-order/depth/validate/kth (LC 94, 102, 104, 98, 230) · DB indexing (B-tree, composite, covering, partial) · tree problems — LCA + diameter + right-side-view (LC 236, 235, 543, 199) · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Traversals + BST Core (~2.5 hr)

### Theory deep-dive

A binary tree node is `{ val, left, right }`. Every tree algorithm is one of two shapes: **DFS** (go deep, recursion or an explicit stack) or **BFS** (go wide, a queue). The three DFS orders differ only in *when you visit the node* relative to its children:

| Order | Visit sequence | Produces / used for |
|---|---|---|
| **Pre-order** | Root → L → R | serialize / deep-copy a tree, expression-tree prefix, "process node before children" |
| **In-order** | L → Root → R | **sorted output on a BST**, range queries, the kth-smallest trick |
| **Post-order** | L → R → Root | bottom-up: deletion, subtree size, height/diameter, "need children's answers first" |

**Why in-order sorts a BST:** the BST invariant is *everything in the left subtree < node < everything in the right subtree*. Visiting left-then-node-then-right therefore emits values in ascending order. This single fact powers LC 98 and LC 230.

**Space:** recursive DFS is **O(h)** on the call stack where `h` is the height — `O(log n)` for a balanced tree, **O(n)** for a degenerate/skewed tree (a linked list). BFS is **O(w)** where `w` is the max width — up to `O(n)` for the bottom level of a complete tree. Always state which.

```java
class TreeNode { int val; TreeNode left, right; TreeNode(int v){ val = v; } }
```

### Worked example — LC 94 Binary Tree Inorder Traversal (iterative, from scratch)

The iterative version is the one interviewers push for ("now without recursion"). The mental model: **go left as far as you can, pushing each node; when you can't, pop, record, then pivot right.**

```java
// Iterative in-order. O(n) time, O(h) space (the stack).
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> out = new ArrayList<>();
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode curr = root;
    while (curr != null || !stack.isEmpty()) {
        while (curr != null) {       // dive left, remembering the path
            stack.push(curr);
            curr = curr.left;
        }
        curr = stack.pop();          // leftmost unvisited node
        out.add(curr.val);           // visit
        curr = curr.right;           // then explore its right subtree
    }
    return out;
}
//      1                 in-order -> [1, 3, 2]
//       \                pre-order -> [1, 2, 3]
//        2               post-order -> [3, 2, 1]
//       /
//      3
```

**Drill the other two iteratively too:**
- **Pre-order:** push root; loop: pop, record, push **right then left** (so left comes off first).
- **Post-order:** the slick trick — do a *modified pre-order* (Root → **Right** → Left) and **reverse the result**. That reversed sequence is L → R → Root. Avoids the fiddly two-pointer "last visited" single-stack version.

**Complexity for all three:** O(n) time, O(h) space.

### Practice set

**1. LC 94 — Binary Tree Inorder Traversal** (Easy)
- **Approach:** the iterative go-left/pop/go-right skeleton above; recursion is the warm-up, iteration is the real target.
- **Key insight:** the `while (curr != null)` inner loop *is* the "go left" phase; the pop + `curr = curr.right` is the "visit + pivot" phase.
- **Complexity:** O(n) / O(h). **Target: < 8 min iteratively.**
- **Follow-up:** Morris traversal achieves **O(1) space** by threading temporary links from a node's in-order predecessor; mention it, you rarely need to code it.

**2. LC 102 — Binary Tree Level Order Traversal** (Medium)
- **Approach:** BFS with an `ArrayDeque` queue. The crucial detail: snapshot `queue.size()` *before* the inner loop so you process exactly one level per outer iteration.
```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> res = new ArrayList<>();
    if (root == null) return res;
    Queue<TreeNode> q = new ArrayDeque<>();
    q.offer(root);
    while (!q.isEmpty()) {
        int size = q.size();                 // freeze this level's count
        List<Integer> level = new ArrayList<>(size);
        for (int i = 0; i < size; i++) {
            TreeNode n = q.poll();
            level.add(n.val);
            if (n.left  != null) q.offer(n.left);
            if (n.right != null) q.offer(n.right);
        }
        res.add(level);
    }
    return res;
}
```
- **Key insight:** reading `q.size()` *inside* the loop is the classic bug — children you enqueue would leak into the current level.
- **Complexity:** O(n) / O(w). **Target: < 12 min.**
- **Follow-up:** LC 103 Zigzag — reverse alternate levels (toggle a boolean, `Collections.reverse` or `addFirst`).

**3. LC 104 — Maximum Depth of Binary Tree** (Easy)
- **Approach:** recursive `1 + max(depth(left), depth(right))`, base case `null → 0`. Then redo it as BFS counting levels.
```java
public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```
- **Key insight:** this is post-order in disguise — you need both children's answers before computing the node's.
- **Complexity:** O(n) / O(h). **Follow-up:** LC 111 Minimum Depth — careful, `min` over a `null` child is wrong; a node with one child takes the non-null side's depth.

**4. LC 98 — Validate Binary Search Tree** (Medium)
- **Approach:** DFS carrying a valid `(min, max)` open interval. The left call tightens `max = node.val`; the right call tightens `min = node.val`.
```java
public boolean isValidBST(TreeNode root) {
    return valid(root, Long.MIN_VALUE, Long.MAX_VALUE);
}
private boolean valid(TreeNode n, long min, long max) {
    if (n == null) return true;
    if (n.val <= min || n.val >= max) return false;     // strict bounds
    return valid(n.left, min, n.val) && valid(n.right, n.val, max);
}
```
- **Key insight:** the **wrong** approach checks only `left.val < node.val < right.val` — it misses grandparent violations (a node deep in the left subtree larger than an ancestor). Bounds propagate the constraint all the way down. Use `long` (or `Long`/null sentinels) so a node holding `Integer.MIN_VALUE`/`MAX_VALUE` doesn't false-fail.
- **Complexity:** O(n) / O(h). **Follow-up:** alternative — do an in-order traversal and verify it's strictly increasing (track `prev`).

**5. LC 230 — Kth Smallest Element in a BST** (Medium)
- **Approach:** in-order traversal stops at the kth node. Iterative with early exit avoids walking the whole tree.
```java
public int kthSmallest(TreeNode root, int k) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode curr = root;
    while (curr != null || !stack.isEmpty()) {
        while (curr != null) { stack.push(curr); curr = curr.left; }
        curr = stack.pop();
        if (--k == 0) return curr.val;       // kth in sorted order
        curr = curr.right;
    }
    return -1; // k out of range
}
```
- **Key insight:** in-order on a BST is sorted, so the kth popped node is the answer — no full sort needed, average O(h + k).
- **Complexity:** O(h + k) / O(h). **Follow-up (asked often):** if the BST is modified frequently and kth queries are hot, **augment each node with `leftCount`** (size of its left subtree); then each query is O(h): if `leftCount + 1 == k` it's the node, if `k <= leftCount` go left, else go right with `k -= leftCount + 1`.

---

## Block B — DB Indexing: B-tree, Composite, Covering, Partial (~2 hr)

> Cross-ref: `../../07-collections.md` (a B-tree is the on-disk cousin of `TreeMap`'s red-black tree — both keep keys sorted for O(log n) lookup and range scans). The HashMap-vs-TreeMap trade-off (O(1) no-order vs O(log n) ordered) is the *same* trade-off as hash-index vs B-tree-index.

### The mental model

An index is a **separate, sorted data structure** that maps column values to row locations, so the database avoids scanning every row (a "sequential scan"). The cost: indexes consume disk, and **every write must update every affected index** — so indexes speed reads and slow writes. That trade-off is the whole topic.

### B-tree (the default — know it cold)

- Self-balancing, **O(log n)** reads, the default index type in Postgres and MySQL.
- Keeps keys **sorted**, so it serves equality *and* **range queries** (`>`, `<`, `BETWEEN`, `ORDER BY`, prefix `LIKE 'abc%'`).
- Real databases use a **B+ tree**: internal nodes hold only keys (for routing), all actual values live in the leaves, and **leaves are linked** in a doubly-linked list — that link is what makes range scans fast (find the start, then walk leaves sequentially).
- **Postgres** uses *heap storage*: the table is an unordered heap, and indexes are separate files pointing at row tuples (TIDs). **MySQL InnoDB** uses a **clustered index** — the table *is* a B+ tree keyed by the primary key, so secondary indexes store the PK (a second lookup, "bookmark lookup") to reach the row.

### Other index types (name them, know when)

- **Hash index:** O(1) exact-match, **no range queries, no ordering**. Postgres has it but it's rarely the right call. This is exactly `HashMap` vs `TreeMap`.
- **GIN** (Generalized Inverted Index): for multi-valued columns — `jsonb`, arrays, full-text search.
- **BRIN** (Block Range Index): tiny index for naturally-ordered huge tables (append-only time-series) — stores min/max per block range.

### Composite index `(a, b, c)` — the leftmost-prefix rule

A composite index is sorted by `a`, then `b` within equal `a`, then `c`. So it can serve queries that use a **leftmost prefix** of the columns:

| Query predicate | Uses `(a, b, c)`? |
|---|---|
| `WHERE a = ?` | ✅ |
| `WHERE a = ? AND b = ?` | ✅ |
| `WHERE a = ? AND b = ? AND c = ?` | ✅ |
| `WHERE a = ? AND c = ?` | partial — uses `a`, then filters `c` |
| `WHERE b = ?` or `WHERE c = ?` | ❌ — skips the leading key |

**Column order matters:** put the column you filter by **equality** first, the **range** column last (a range "stops" the index — columns after a range predicate can't be used for seeking). Rule of thumb: **equality columns → sort column → range column.**

### Covering index — the fastest read

A **covering index** contains *every column the query touches* (filter + select), so Postgres answers from the index alone — an **index-only scan** that never visits the heap.

```sql
-- Query:  SELECT tenant_id, status, created_at
--         FROM orders WHERE tenant_id = ? AND status = ?;
CREATE INDEX idx_orders_cover
    ON orders (tenant_id, status, created_at);          -- all 3 cols in the index
-- Postgres also supports INCLUDE for non-key payload columns:
CREATE INDEX idx_orders_cover2
    ON orders (tenant_id, status) INCLUDE (created_at);  -- created_at not used for seeking
```

`INCLUDE` is the precise tool: keep the *search* columns as keys, carry the *returned-only* columns as non-key payload so the tree stays narrow.

### Partial index — tiny and targeted

```sql
CREATE INDEX idx_pending ON orders (created_at) WHERE status = 'PENDING';
```
Only indexes the rows matching the predicate — perfect for the **Outbox poll** (`WHERE status = 'PENDING'`): the index is a fraction of the table size and the poller's query is near-instant. (Thursday connects this back to the Outbox pattern.)

### Selectivity — choose the right column

**Selectivity = distinct values / total rows.** High-cardinality columns (`user_id`, `email`, `uuid`) make great index targets; low-cardinality ones (a 2-value boolean, a 3-value status) don't — the planner may ignore the index and seq-scan because reading 40% of rows via the index (random I/O) is slower than a sequential scan. Composite or partial indexes rescue low-cardinality columns.

### Production technique (the senior signal)

- **`CREATE INDEX CONCURRENTLY`** on a large live table — takes a weaker lock so reads/writes continue (a plain `CREATE INDEX` holds a write lock for the whole build). It's slower and can leave an `INVALID` index if it fails — drop and retry.
- **`EXPLAIN (ANALYZE, BUFFERS)`** to *verify* the index is used (look for `Index Scan` / `Index Only Scan`, not `Seq Scan`) and to see cache hits vs disk reads.

### Project tie-in

- **Smart360 — the 60s → 2–3s win:** the slow report joined ~50 tables filtered by `tenant_id` and `status`, sorted by `created_at`. A composite index `(tenant_id, status, created_at)` turned a seq-scan + sort into an index range scan that was also *covering* for the projection — no heap visits, no sort node. The talking point: "I read the `EXPLAIN`, saw a Seq Scan + Sort, indexed the join/filter/sort columns *in that order*, and confirmed the Index Only Scan."
- **Deep Fathom:** a partial index on `WHERE processed = false` kept the work-queue scan O(pending) instead of O(all-rows-ever).
- **WebX:** high-cardinality `request_id` UUID indexed for idempotency-key lookups; a low-cardinality `provider` column deliberately *not* indexed alone (combined into a composite with `created_at`).

---

## Block C — DSA: Tree Problems — LCA, Diameter, Views (~1.5 hr)

### Theory

These four are the "apply a traversal with a twist" tier. LCA on a general tree is **post-order** (you need both children's answers); LCA on a BST is a **guided walk** (the ordering tells you which way to go); diameter is **post-order returning height while updating a global**; right-side-view is **level-order taking the last element**.

### Practice set

**1. LC 236 — Lowest Common Ancestor of a Binary Tree** (Medium, general tree)
- **Approach:** post-order recursion. Return the node itself if it equals `p` or `q`; otherwise recurse both sides. If both sides return non-null, *this* node is the split point → the LCA. Else bubble up whichever side is non-null.
```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    TreeNode l = lowestCommonAncestor(root.left, p, q);
    TreeNode r = lowestCommonAncestor(root.right, p, q);
    if (l != null && r != null) return root;   // p and q split here
    return l != null ? l : r;                   // both on one side (or none)
}
```
- **Key insight:** the elegant case — if one target *is* the ancestor of the other, the recursion returns that node on the match and `null` from the sibling branch, so the parent propagates it upward correctly *without* a separate "found both" check.
- **Complexity:** O(n) / O(h).

**2. LC 235 — Lowest Common Ancestor of a BST** (Medium)
- **Approach:** exploit ordering — no full search needed.
```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    while (root != null) {
        if (p.val < root.val && q.val < root.val)      root = root.left;
        else if (p.val > root.val && q.val > root.val) root = root.right;
        else return root;   // split point (or one equals root) -> LCA
    }
    return null;
}
```
- **Key insight:** the first node where `p` and `q` fall on opposite sides (or one equals the node) is the LCA. O(h), O(1) iteratively — strictly better than the general-tree O(n).
- **Complexity:** O(h) / O(1). **Common mistake:** treating it as a general tree and ignoring the BST property.

**3. LC 543 — Diameter of Binary Tree** (Easy/Medium)
- **Approach:** the diameter *through* a node = left height + right height (in edges). Run a height DFS and update a global max at every node.
```java
private int best = 0;
public int diameterOfBinaryTree(TreeNode root) {
    height(root);
    return best;
}
private int height(TreeNode n) {
    if (n == null) return 0;
    int l = height(n.left), r = height(n.right);
    best = Math.max(best, l + r);   // path through n, in edges
    return 1 + Math.max(l, r);      // height returned upward
}
```
- **Key insight:** the longest path **need not pass through the root** — that's why you track a global at every node, not just compute root height. Returning height while updating a side-effect global is *the* reusable tree pattern (reused in LC 124 tomorrow).
- **Complexity:** O(n) / O(h).

**4. LC 199 — Binary Tree Right Side View** (Medium)
- **Approach:** level-order BFS; the **last** node dequeued at each level is what you'd see from the right.
```java
public List<Integer> rightSideView(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    if (root == null) return res;
    Queue<TreeNode> q = new ArrayDeque<>();
    q.offer(root);
    while (!q.isEmpty()) {
        int size = q.size();
        for (int i = 0; i < size; i++) {
            TreeNode n = q.poll();
            if (i == size - 1) res.add(n.val);   // rightmost of the level
            if (n.left  != null) q.offer(n.left);
            if (n.right != null) q.offer(n.right);
        }
    }
    return res;
}
```
- **Key insight:** reuse the exact LC 102 skeleton; only the "what to record" changes. **Alternative:** DFS visiting **right before left** and recording the first node seen at each new depth.
- **Complexity:** O(n) / O(w).

---

## 💻 Practice coding questions

1. **LC 94 Inorder Traversal** — iterative go-left/pop/go-right; also code pre-order and the reverse-of-(Root→R→L) post-order.
2. **LC 144 Preorder Traversal** — iterative: push root, pop/record, push right then left.
3. **LC 145 Postorder Traversal** — modified pre-order Root→R→L, then `Collections.reverse`.
4. **LC 102 Level Order** — snapshot `queue.size()` before the inner loop.
5. **LC 103 Zigzag Level Order** — toggle direction per level.
6. **LC 104 Maximum Depth** — recursive one-liner + BFS level count.
7. **LC 111 Minimum Depth** — guard the single-child case (don't `min` over a null child).
8. **LC 98 Validate BST** — `(min, max)` bounds with `long`; or in-order strictly increasing.
9. **LC 230 Kth Smallest** — in-order with early exit; know the augmented-`leftCount` follow-up.
10. **LC 236 LCA (general)** — post-order, both-sides-non-null → current.
11. **LC 235 LCA (BST)** — guided walk, O(h)/O(1).
12. **LC 543 Diameter** — height DFS updating a global.
13. **LC 199 Right Side View** — BFS last-of-level (or right-first DFS).
14. **LC 226 Invert Binary Tree** — swap children recursively (the famous warm-up).
15. **LC 101 Symmetric Tree** — mirror-compare `(left.left, right.right)` and `(left.right, right.left)`.

> For each: say the traversal type aloud first, then code without hints. Re-time LC 102 and LC 98 — target ≤ 12 min each.

---

## 🎤 Interview questions

1. **Walk through all four traversal orders and one real use of each.** In-order → sorted BST output / range queries; pre-order → serialize/deep-copy, expression-tree prefix; post-order → bottom-up deletion, subtree size, height/diameter; level-order → BFS, shortest path on unweighted graphs, "by depth" processing.
2. **Why does in-order traversal of a BST produce sorted output?** The BST invariant is left < node < right; visiting L → node → R emits ascending values by construction.
3. **Validate BST — why isn't `left.val < node.val < right.val` enough?** It only checks immediate parent-child, missing grandparent violations (a deep left-subtree node larger than an ancestor). Pass a shrinking `(min, max)` bound down; use `long` for the int-extreme edges.
4. **Recursive vs iterative traversal — space?** Both O(n) time; recursion uses O(h) call-stack, iteration uses an O(h) explicit stack. Morris traversal threads links for O(1) space.
5. **LCA where one target is the ancestor of the other?** The recursion returns the node on the match; the sibling branch returns null; the parent receives the matched node and propagates it up — correct without special-casing.
6. **General-tree LCA vs BST LCA — complexity difference?** General: O(n) post-order (must search). BST: O(h) guided walk, O(1) space — the ordering tells you which subtree contains both.
7. **Why track a global max in Diameter instead of returning it?** The longest path may not pass through the root; you must consider the "left-height + right-height" path at *every* node, while the function returns height upward.
8. **Kth Smallest follow-up — BST mutates often, kth is hot. Optimize?** Augment nodes with left-subtree counts → O(h) per query (and O(h) maintenance on insert/delete) instead of O(h + k) in-order each time.
9. **What is a B-tree and why is it the default DB index?** A self-balancing sorted tree (B+ tree in practice) giving O(log n) reads and, because keys are sorted with linked leaves, efficient range scans and ordered reads — covering most query shapes.
10. **B-tree vs hash index — when each?** B-tree for ranges/ordering/prefix and as the safe default; hash for pure equality with no ordering (rare). Same trade-off as `TreeMap` vs `HashMap`.
11. **Explain the leftmost-prefix rule with a concrete index.** `(tenant_id, status, created_at)` serves `tenant_id`, `tenant_id+status`, and all three — but **not** `status` alone, because the tree is sorted by the leading key first.
12. **How do you order columns in a composite index?** Equality-filter columns first, then the sort column, then the range column last (a range predicate stops further index seeking).
13. **What is a covering index and why is it the fastest read?** It contains every column the query needs (filter + select), enabling an index-only scan that never touches the heap. Use `INCLUDE` for return-only payload columns.
14. **When is a partial index the right tool?** When queries always filter on a constant predicate (e.g. `WHERE status='PENDING'`) — the index covers only those rows, staying tiny and hot. Ideal for the Outbox poll.
15. **Selectivity — why not index a boolean column?** Low cardinality means the index returns a large fraction of rows; random index I/O for 40% of a table is slower than a sequential scan, so the planner ignores it. Index high-cardinality columns, or combine into a composite/partial index.
16. **Add an index to a 100M-row production table without downtime.** `CREATE INDEX CONCURRENTLY` (weaker lock, allows reads/writes; can leave an INVALID index on failure — drop and retry); run off-peak; monitor `pg_stat_progress_create_index`; verify with `EXPLAIN (ANALYZE, BUFFERS)`.
17. **Postgres heap storage vs MySQL InnoDB clustered index — practical difference?** Postgres indexes point at heap tuples (table separate from index). InnoDB's table *is* a B+ tree by PK, so secondary indexes store the PK and need a second lookup to fetch the row — and the PK choice affects all secondary-index sizes.
18. **You added the right index but the query is still slow — debug it.** Run `EXPLAIN (ANALYZE, BUFFERS)`: stale stats (`ANALYZE`), a predicate not matching the leftmost prefix, a function wrapping the column (`WHERE lower(email)=?` needs a functional index), an implicit type cast disabling the index, or low selectivity making seq-scan genuinely cheaper.

---

## ✅ Self-check

1. Write the iterative in-order skeleton from memory (go-left loop → pop/record → go-right). Then state the post-order trick (modified pre-order + reverse).
2. Explain the difference between a covering index and a regular index in two sentences, no jargon.
3. Given `WHERE a = ? AND c = ?` and an index `(a, b, c)`, which columns does it use and why? (Uses `a` to seek, then filters `c` — `b` is skipped, so `c` can't be a seek key.)
4. Why must Validate BST pass bounds down rather than compare parent-child only? Give the failing shape.
5. State the BST-LCA rule in one line and its complexity. (Both < node → left; both > node → right; else node. O(h)/O(1).)

---

*Nav: ← [Week 2](../week-02/) · [Week 3](README.md) · [02-tue-jul-14.md](02-tue-jul-14.md) →*
