# DSA Solutions — Trees

> Full worked solutions in Java 17. Each problem gives a self-contained statement, the approach, a complete compilable solution, complexity, and a key insight or follow-up. Recurring techniques: **DFS recursion** (the call stack *is* the traversal), **BFS with a queue** (level-by-level work), **the BST ordering invariant** (left < node < right), and **post-order accumulation** (compute a child answer, return one value to the parent while updating a global).

All solutions assume the standard binary-tree node defined once below.

```java
// Shared definition used by every problem in this file.
class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode() {}
    TreeNode(int val) { this.val = val; }
    TreeNode(int val, TreeNode left, TreeNode right) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}
```

---

### LC94 — Binary Tree Inorder Traversal *(easy)*
**Problem:** Given the `root` of a binary tree, return the inorder traversal of its nodes' values (left subtree, node, right subtree). Example: for the tree `1` with right child `2` whose left child is `3`, the result is `[1, 3, 2]`.

**Approach:** Two equivalent solutions. The **recursive** version mirrors the definition directly. The **iterative** version simulates the call stack with an explicit `Deque`: push the entire left spine, then pop a node (visit it), and descend into its right subtree.
```java
import java.util.ArrayList;
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.List;

class Solution {
    // Recursive
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        inorder(root, result);
        return result;
    }

    private void inorder(TreeNode node, List<Integer> out) {
        if (node == null) return;
        inorder(node.left, out);
        out.add(node.val);
        inorder(node.right, out);
    }

    // Iterative (explicit stack)
    public List<Integer> inorderTraversalIterative(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        Deque<TreeNode> stack = new ArrayDeque<>();
        TreeNode cur = root;
        while (cur != null || !stack.isEmpty()) {
            while (cur != null) {       // push the entire left spine
                stack.push(cur);
                cur = cur.left;
            }
            cur = stack.pop();          // leftmost unvisited node
            result.add(cur.val);        // visit
            cur = cur.right;            // then explore the right subtree
        }
        return result;
    }
}
```
**Complexity:** O(n) time. Space O(h) where h is the tree height (recursion stack or explicit stack): O(log n) for a balanced tree, O(n) for a degenerate one.

**Key insight / follow-up:** Inorder traversal of a **BST yields sorted order** — that single fact powers LC98 and LC230 below. The iterative form is what interviewers probe; the key invariant is "before visiting a node, every node to its left has already been pushed." A truly O(1)-space variant is **Morris traversal**, which threads temporary links from each predecessor and unthreads them after visiting.

---

### LC102 — Binary Tree Level Order Traversal *(medium)*
**Problem:** Given the `root` of a binary tree, return its node values grouped by level, top to bottom, left to right. Example: tree `3 / (9, 20) / 20 → (15, 7)` returns `[[3], [9, 20], [15, 7]]`.

**Approach:** Breadth-first search with a queue. The trick is to capture `queue.size()` at the start of each level so you process exactly the current level's nodes before their children, building one sublist per level.
```java
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;
import java.util.Queue;

class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        if (root == null) return result;

        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while (!queue.isEmpty()) {
            int levelSize = queue.size();        // freeze this level's count
            List<Integer> level = new ArrayList<>(levelSize);
            for (int i = 0; i < levelSize; i++) {
                TreeNode node = queue.poll();
                level.add(node.val);
                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
            }
            result.add(level);
        }
        return result;
    }
}
```
**Complexity:** O(n) time, O(n) space (the queue holds up to one full level, which can be ~n/2 nodes).

**Key insight / follow-up:** Snapshotting `levelSize` before the inner loop is the whole technique — without it you'd mix children into the current level. This BFS skeleton is the basis for LC199 (right-side view), zigzag traversal (reverse alternate levels), and level averages. A DFS variant also works: recurse with a `depth` argument and append to `result.get(depth)`, creating the sublist on first visit to a level.

---

### LC104 — Maximum Depth of Binary Tree *(easy)*
**Problem:** Given the `root` of a binary tree, return its maximum depth — the number of nodes along the longest path from the root down to the farthest leaf. An empty tree has depth 0. Example: tree `3 → (9, 20 → (15, 7))` has depth 3.

**Approach:** Post-order recursion. A node's depth is 1 plus the larger of its two subtree depths; the base case (`null`) contributes 0.
```java
class Solution {
    public int maxDepth(TreeNode root) {
        if (root == null) return 0;
        int left = maxDepth(root.left);
        int right = maxDepth(root.right);
        return 1 + Math.max(left, right);
    }
}
```
**Complexity:** O(n) time, O(h) space for the recursion stack (O(n) worst case for a skewed tree).

**Key insight / follow-up:** This is the canonical "return one value up from each subtree" pattern that generalizes to diameter (LC543) and max path sum (LC124) — the difference is only what you compute from `left` and `right`. For very deep, unbalanced trees that risk a `StackOverflowError`, switch to the iterative BFS level-count from LC102 (number of levels = depth).

---

### LC226 — Invert Binary Tree *(easy)*
**Problem:** Given the `root` of a binary tree, invert it (mirror it left-to-right) and return the root. Example: `4 → (2 → (1, 3), 7 → (6, 9))` becomes `4 → (7 → (9, 6), 2 → (3, 1))`.

**Approach:** Recursively swap each node's left and right children. The order (swap before or after recursing) doesn't matter as long as every node is visited.
```java
class Solution {
    public TreeNode invertTree(TreeNode root) {
        if (root == null) return null;
        TreeNode left = invertTree(root.left);   // invert subtrees first
        TreeNode right = invertTree(root.right);
        root.left = right;                        // then swap
        root.right = left;
        return root;
    }
}
```
**Complexity:** O(n) time, O(h) space for the recursion stack.

**Key insight / follow-up:** Inversion is a pure structural mutation — swap children at every node and you're done. An iterative version uses a queue or stack: poll a node, swap its children, enqueue both. This is the famous "homebrew author who couldn't whiteboard it" interview problem; the recursion is three lines once you see that "invert the tree" = "swap children, recurse."

---

### LC101 — Symmetric Tree *(easy)*
**Problem:** Given the `root` of a binary tree, return `true` if it is a mirror image of itself around its center. Example: `1 → (2 → (3, 4), 2 → (4, 3))` is symmetric; `1 → (2 → (_, 3), 2 → (_, 3))` is not.

**Approach:** Compare two subtrees for mirror symmetry. Two trees mirror each other when their roots are equal and the left child of one mirrors the right child of the other (and vice versa). Recurse with a pair of pointers walking inward from opposite sides.
```java
class Solution {
    public boolean isSymmetric(TreeNode root) {
        if (root == null) return true;
        return isMirror(root.left, root.right);
    }

    private boolean isMirror(TreeNode a, TreeNode b) {
        if (a == null && b == null) return true;     // both empty -> mirror
        if (a == null || b == null) return false;    // exactly one empty
        return a.val == b.val
            && isMirror(a.left, b.right)             // outer pair
            && isMirror(a.right, b.left);            // inner pair
    }
}
```
**Complexity:** O(n) time, O(h) space for the recursion stack.

**Key insight / follow-up:** The cross-comparison `(a.left, b.right)` and `(a.right, b.left)` is what encodes "mirror" rather than "identical" — the same-side comparison would instead solve LC100 (Same Tree). An iterative version pushes pairs onto a queue and checks each pair's symmetry, processing `(a.left, b.right)` and `(a.right, b.left)` together.

---

### LC112 — Path Sum *(easy)*
**Problem:** Given the `root` of a binary tree and an integer `targetSum`, return `true` if the tree has a **root-to-leaf** path such that adding up all the values along the path equals `targetSum`. A leaf is a node with no children. Example: in `5 → (4 → (11 → (7, 2)), 8 → (13, 4 → (_, 1)))` with `targetSum = 22`, the path `5 → 4 → 11 → 2` sums to 22, so return `true`.

**Approach:** DFS, subtracting each node's value from the remaining target as you descend. At a **leaf**, check whether the remaining target equals the leaf's value (equivalently, whether the running remainder hits zero exactly at a leaf).
```java
class Solution {
    public boolean hasPathSum(TreeNode root, int targetSum) {
        if (root == null) return false;
        // leaf: succeed iff this node finishes the sum
        if (root.left == null && root.right == null) {
            return targetSum == root.val;
        }
        int remaining = targetSum - root.val;
        return hasPathSum(root.left, remaining)
            || hasPathSum(root.right, remaining);
    }
}
```
**Complexity:** O(n) time, O(h) space for the recursion stack.

**Key insight / follow-up:** The leaf check is essential and easy to get wrong — you must reach an actual leaf, not just any node where the sum hits zero (a `null` child of a one-armed node would falsely match). Returning `false` for `null` lets a one-armed node fall through to its real child. Follow-ups: **LC113** returns all such paths (backtrack with a list), and **LC437** counts paths that may start/end anywhere (prefix-sum hash map).

---

### LC98 — Validate Binary Search Tree *(medium)*
**Problem:** Given the `root` of a binary tree, determine if it is a valid binary search tree. A valid BST requires: every node in the left subtree is **strictly less** than the node, every node in the right subtree is **strictly greater**, and both subtrees are themselves valid BSTs. Example: `2 → (1, 3)` is valid; `5 → (1, 4 → (3, 6))` is not (3 and 4 sit in the right subtree of 5 but are < 5).

**Approach:** Recurse with an open `(low, high)` bound for each subtree. The root is unbounded; descending left tightens the upper bound to the current value, descending right tightens the lower bound. A node must lie strictly inside its inherited bounds. `Long` bounds avoid overflow when a node holds `Integer.MIN_VALUE` or `MAX_VALUE`.
```java
class Solution {
    public boolean isValidBST(TreeNode root) {
        return validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
    }

    private boolean validate(TreeNode node, long low, long high) {
        if (node == null) return true;
        if (node.val <= low || node.val >= high) return false;
        return validate(node.left, low, node.val)       // upper bound tightens
            && validate(node.right, node.val, high);    // lower bound tightens
    }
}
```
**Complexity:** O(n) time, O(h) space for the recursion stack.

**Key insight / follow-up:** The classic bug is checking only `node.left.val < node.val < node.right.val` locally — that misses violations deeper in a subtree (a value can be larger than its parent yet still break an ancestor's bound). Passing bounds down fixes this. An alternative is an **inorder traversal**: a valid BST produces a strictly increasing sequence, so track the previous value and fail if the current one isn't strictly greater.

---

### LC230 — Kth Smallest Element in a BST *(medium)*
**Problem:** Given the `root` of a binary search tree and an integer `k` (1-indexed), return the k-th smallest value among all nodes. Example: in the BST `3 → (1 → (_, 2), 4)` with `k = 1`, the answer is `1`.

**Approach:** An inorder traversal of a BST visits values in ascending order, so the k-th node visited is the answer. Use an iterative inorder so you can stop the instant the count reaches `k`, rather than traversing the whole tree.
```java
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public int kthSmallest(TreeNode root, int k) {
        Deque<TreeNode> stack = new ArrayDeque<>();
        TreeNode cur = root;
        while (cur != null || !stack.isEmpty()) {
            while (cur != null) {        // descend the left spine
                stack.push(cur);
                cur = cur.left;
            }
            cur = stack.pop();
            if (--k == 0) return cur.val; // k-th node in sorted order
            cur = cur.right;
        }
        return -1; // unreachable for valid 1 <= k <= n
    }
}
```
**Complexity:** O(h + k) time (descend to the smallest, then pop k times), O(h) space.

**Key insight / follow-up:** "BST + k-th smallest" should immediately suggest **inorder traversal**, because it linearizes the tree into sorted order. The iterative early-exit beats a full recursive traversal when k is small. **Follow-up (a common one):** if the tree is modified often and you need many k-th queries, augment each node with its left-subtree size — then each query is O(h) by navigating directly, no traversal needed.

---

### LC235 — Lowest Common Ancestor of a BST *(medium)*
**Problem:** Given a binary **search** tree and two nodes `p` and `q` present in it, return their lowest common ancestor (the deepest node that has both `p` and `q` as descendants; a node can be a descendant of itself). Example: in the BST rooted at `6`, the LCA of `2` and `8` is `6`; the LCA of `2` and `4` is `2`.

**Approach:** Exploit the BST ordering. Starting at the root: if both `p` and `q` are greater than the current node, the LCA is in the right subtree; if both are smaller, go left; otherwise the paths to `p` and `q` diverge here (or one *is* the current node), so the current node is the LCA.
```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        TreeNode cur = root;
        while (cur != null) {
            if (p.val > cur.val && q.val > cur.val) {
                cur = cur.right;          // both larger -> go right
            } else if (p.val < cur.val && q.val < cur.val) {
                cur = cur.left;           // both smaller -> go left
            } else {
                return cur;               // split point (or cur == p/q)
            }
        }
        return null;
    }
}
```
**Complexity:** O(h) time, O(1) space (no recursion or extra structures).

**Key insight / follow-up:** The "split point" — the first node where `p` and `q` fall on opposite sides (or one equals the node) — is by definition the lowest common ancestor in a BST. This is strictly easier than the general-tree LC236 below because the ordering tells you which way to go without exploring both subtrees. The iterative loop uses O(1) space; the recursive version is equally clean but uses O(h) stack.

---

### LC236 — Lowest Common Ancestor of a Binary Tree *(medium)*
**Problem:** Given a binary tree (no ordering guarantee) and two nodes `p` and `q` guaranteed to exist in it, return their lowest common ancestor. Example: in `3 → (5 → (6, 2 → (7, 4)), 1 → (0, 8))`, the LCA of `5` and `1` is `3`, and the LCA of `5` and `4` is `5`.

**Approach:** Post-order DFS. Recurse into both subtrees searching for `p` or `q`. If a node *is* `p` or `q`, return it. A node whose left and right recursions both return non-null is the LCA (one target found in each subtree). Otherwise propagate up whichever side found a target.
```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null || root == p || root == q) {
            return root;                 // found a target (or hit a dead end)
        }
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);

        if (left != null && right != null) return root; // p, q in different subtrees
        return (left != null) ? left : right;           // both on one side, or neither
    }
}
```
**Complexity:** O(n) time (each node visited once), O(h) space for the recursion stack.

**Key insight / follow-up:** The "both sides non-null → this is the LCA" rule works because the recursion bubbles each target up to its ancestors; the first node that sees a target arriving from *both* children is the deepest common ancestor. The early `root == p || root == q` return correctly handles the case where one node is an ancestor of the other (it returns that node, and the other subtree returns null). This assumes both nodes exist; if not guaranteed, you'd track found-counts separately.

---

### LC543 — Diameter of Binary Tree *(easy)*
**Problem:** Given the `root` of a binary tree, return the length of its diameter — the number of **edges** on the longest path between any two nodes. The path need not pass through the root. Example: `1 → (2 → (4, 5), 3)` has diameter 3 (the path `4 → 2 → 1 → 3` spans 3 edges).

**Approach:** Post-order recursion that returns each subtree's **height** (in edges) while updating a global maximum. At each node, the longest path *through* it is `leftHeight + rightHeight` edges; the value returned to the parent is `1 + max(leftHeight, rightHeight)`.
```java
class Solution {
    private int diameter = 0;

    public int diameterOfBinaryTree(TreeNode root) {
        height(root);
        return diameter;
    }

    private int height(TreeNode node) {
        if (node == null) return 0;          // height in edges
        int left = height(node.left);
        int right = height(node.right);
        diameter = Math.max(diameter, left + right);  // path through this node
        return 1 + Math.max(left, right);             // height to the parent
    }
}
```
**Complexity:** O(n) time, O(h) space for the recursion stack.

**Key insight / follow-up:** The crux is the dual role of the recursion: it *returns* a height to its caller but *updates a global* with the through-node path — you can't return both, so the longest path is captured as a side effect. Defining height in edges (null = 0) makes `left + right` directly the edge count, no off-by-one. This same "return one thing, accumulate another" structure recurs in LC124 below, where the global is a sum instead of a length.

---

### LC199 — Binary Tree Right Side View *(medium)*
**Problem:** Given the `root` of a binary tree, imagine standing to the right of it; return the values of the nodes visible from top to bottom — i.e., the **rightmost node at each level**. Example: `1 → (2 → (_, 5), 3 → (_, 4))` returns `[1, 3, 4]`.

**Approach:** Level-order BFS (as in LC102), but record only the last node dequeued at each level — that node is the rightmost. Capturing `queue.size()` per level lets you identify the final node of each level.
```java
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;
import java.util.Queue;

class Solution {
    public List<Integer> rightSideView(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        if (root == null) return result;

        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while (!queue.isEmpty()) {
            int levelSize = queue.size();
            for (int i = 0; i < levelSize; i++) {
                TreeNode node = queue.poll();
                if (i == levelSize - 1) {        // rightmost node of this level
                    result.add(node.val);
                }
                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
            }
        }
        return result;
    }
}
```
**Complexity:** O(n) time, O(n) space for the queue.

**Key insight / follow-up:** The right-side view is just "the last node of each BFS level," so the LC102 skeleton solves it with a one-line change. A DFS alternative recurses **right child first**, adding a node only when its depth first exceeds the result size — the first node seen at each depth is then the rightmost. The left-side view is the symmetric problem: take the first node of each level instead.

---

### LC105 — Construct Binary Tree from Preorder and Inorder Traversal *(medium)*
**Problem:** Given two integer arrays `preorder` and `inorder` representing the preorder and inorder traversals of a binary tree (all values distinct), construct and return the tree. Example: `preorder = [3, 9, 20, 15, 7]`, `inorder = [9, 3, 15, 20, 7]` builds `3 → (9, 20 → (15, 7))`.

**Approach:** Preorder's first element is always the current root. Find that value in inorder: everything to its left is the left subtree, everything to its right is the right subtree. Recurse, consuming preorder elements left-to-right via a moving index. A hash map from value → inorder index makes the lookup O(1).
```java
import java.util.HashMap;
import java.util.Map;

class Solution {
    private int preIndex = 0;
    private Map<Integer, Integer> inorderIndex = new HashMap<>();

    public TreeNode buildTree(int[] preorder, int[] inorder) {
        for (int i = 0; i < inorder.length; i++) {
            inorderIndex.put(inorder[i], i);    // value -> position in inorder
        }
        return build(preorder, 0, inorder.length - 1);
    }

    private TreeNode build(int[] preorder, int inLeft, int inRight) {
        if (inLeft > inRight) return null;       // empty range -> no node

        int rootVal = preorder[preIndex++];      // next preorder value is the root
        TreeNode root = new TreeNode(rootVal);
        int mid = inorderIndex.get(rootVal);     // split point in inorder

        root.left = build(preorder, inLeft, mid - 1);   // build left first (preorder!)
        root.right = build(preorder, mid + 1, inRight);
        return root;
    }
}
```
**Complexity:** O(n) time (each node created once, O(1) index lookups), O(n) space for the map plus O(h) recursion.

**Key insight / follow-up:** The order of the two recursive calls matters critically: preorder visits the **entire left subtree before the right**, so you must consume the left subtree (advancing `preIndex`) before building the right. The inorder index map turns a repeated O(n) linear scan into O(1), dropping the total from O(n²) to O(n). The sibling problem **LC106** builds from *post*order + inorder — there you consume postorder from the **end** and build the **right subtree first**.

---

### LC124 — Binary Tree Maximum Path Sum *(hard)*
**Problem:** A path is any sequence of nodes connected by edges, where each node appears at most once; the path need not pass through the root. The path sum is the sum of the node values on it. Given the `root` of a binary tree, return the maximum path sum of any non-empty path. Node values may be negative. Example: `-10 → (9, 20 → (15, 7))` has max path sum 42 (`15 → 20 → 7`).

**Approach:** Post-order recursion returning the best **downward** path gain from each node (a single arm: the node plus the better of its two children, but never less than just the node). At each node, the best path that *bends* through it is `node.val + leftGain + rightGain`; update a global max with that. Crucially, clamp each child's gain to `>= 0` so negative subtrees are dropped rather than dragging the sum down.
```java
class Solution {
    private int maxSum = Integer.MIN_VALUE;

    public int maxPathSum(TreeNode root) {
        gain(root);
        return maxSum;
    }

    private int gain(TreeNode node) {
        if (node == null) return 0;
        // drop negative contributions: a child arm is taken only if it helps
        int leftGain = Math.max(gain(node.left), 0);
        int rightGain = Math.max(gain(node.right), 0);

        // best path that peaks at this node (may use both arms)
        maxSum = Math.max(maxSum, node.val + leftGain + rightGain);

        // to the parent, return a straight path: node + at most one arm
        return node.val + Math.max(leftGain, rightGain);
    }
}
```
**Complexity:** O(n) time, O(h) space for the recursion stack.

**Key insight / follow-up:** Two distinct quantities live at each node: the **best bending path through it** (uses both arms — updates the global, but can't be returned because a parent can only extend a straight line) and the **best straight path down from it** (one arm — returned to the parent). Clamping arm gains at 0 is what handles negatives correctly: a subtree that only subtracts is simply not taken. Initializing `maxSum` to `Integer.MIN_VALUE` (not 0) is required so an all-negative tree returns its least-negative single node.

---

### LC297 — Serialize and Deserialize Binary Tree *(hard)*
**Problem:** Design an algorithm to serialize a binary tree to a string and deserialize that string back into the identical tree. There is no restriction on the format, as long as `deserialize(serialize(root))` reproduces the original tree. Values may be any integers; the tree may be empty. Example: `1 → (2, 3 → (4, 5))` round-trips to an equal tree.

**Approach:** Preorder DFS with explicit null markers. Serialization writes each node's value (and `#` for null) separated by commas, capturing the structure unambiguously. Deserialization reads the tokens in the same preorder and rebuilds: consume one token as the root, then recursively build left, then right.
```java
import java.util.Arrays;
import java.util.Iterator;
import java.util.LinkedList;
import java.util.Queue;

public class Codec {
    private static final String NULL = "#";
    private static final String SEP = ",";

    public String serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        serializeHelper(root, sb);
        return sb.toString();
    }

    private void serializeHelper(TreeNode node, StringBuilder sb) {
        if (node == null) {
            sb.append(NULL).append(SEP);
            return;
        }
        sb.append(node.val).append(SEP);
        serializeHelper(node.left, sb);    // preorder: root, left, right
        serializeHelper(node.right, sb);
    }

    public TreeNode deserialize(String data) {
        Queue<String> tokens = new LinkedList<>(Arrays.asList(data.split(SEP)));
        return deserializeHelper(tokens);
    }

    private TreeNode deserializeHelper(Queue<String> tokens) {
        String token = tokens.poll();
        if (NULL.equals(token)) return null;
        TreeNode node = new TreeNode(Integer.parseInt(token));
        node.left = deserializeHelper(tokens);   // same preorder consumption
        node.right = deserializeHelper(tokens);
        return node;
    }
}
```
**Complexity:** O(n) time for both directions, O(n) space for the string plus O(h) recursion.

**Key insight / follow-up:** Null markers are non-negotiable — without them the structure is ambiguous (preorder alone can't distinguish many trees). Using the **same preorder order** for both serialize and deserialize is what makes the round-trip work: the deserializer consumes tokens in exactly the order the serializer produced them, so each recursive call naturally claims its subtree's tokens. A **BFS/level-order** format (the style LeetCode displays) is equally valid. For a *BST* specifically, you can skip null markers entirely and reconstruct from preorder alone using value bounds, yielding a more compact encoding.

---

*[← DSA bank index](README.md)*
