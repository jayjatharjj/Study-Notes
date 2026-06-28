# DSA Solutions — Graphs, Trie & Dijkstra

> Full worked solutions in Java 17. Each problem gives a self-contained statement, the approach, a complete compilable solution, complexity, and a key insight or follow-up. This file covers grid and adjacency-list traversal (BFS/DFS), topological sort, union-find, the trie, and shortest-path algorithms (Dijkstra, Bellman-Ford).

**Choosing a traversal.** For *reachability* and *connected components*, plain BFS or DFS works — DFS is the natural fit on grids (short recursion, easy in-place marking), BFS when you want the shortest number of *edges* in an **unweighted** graph or need level-by-level expansion (multi-source flood fill). For shortest path with **non-negative edge weights**, use **Dijkstra** (a min-heap-ordered BFS). With *negative* edges or a hard cap on the number of edges/hops, use **Bellman-Ford** (relax all edges V−1 times). For "can this be ordered respecting dependencies," use **topological sort** (Kahn's BFS on in-degrees, or DFS post-order). For "are these two things in the same set / does adding this edge make a cycle," reach for **union-find**.

**Union-find template** (used by LC684, LC547, and a common alternative for LC200/LC130). Path compression + union by rank gives near-O(1) amortized operations (inverse-Ackermann, α(n)).

```java
class DSU {
    private final int[] parent;
    private final int[] rank;
    int components;

    DSU(int n) {
        parent = new int[n];
        rank = new int[n];
        components = n;
        for (int i = 0; i < n; i++) parent[i] = i; // each node is its own root
    }

    int find(int x) {                       // path compression
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }

    boolean union(int a, int b) {           // returns false if already joined
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;         // a cycle would be created
        if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
        parent[rb] = ra;                    // attach smaller tree under larger
        if (rank[ra] == rank[rb]) rank[ra]++;
        components--;
        return true;
    }
}
```

---

### LC200 — Number of Islands *(medium)*
**Problem:** Given an `m x n` 2D grid of `'1'`s (land) and `'0'`s (water), return the number of islands. An island is a maximal group of land cells connected 4-directionally (up/down/left/right). The grid is surrounded by water on all sides. Example: a grid with three separate land clusters returns `3`.

**Approach:** Scan every cell. When an unvisited `'1'` is found, increment the island count and flood-fill from it with DFS, sinking every connected land cell to `'0'` so it is never counted again. Mutating the grid in place avoids a separate `visited` array.
```java
class Solution {
    public int numIslands(char[][] grid) {
        int count = 0;
        for (int r = 0; r < grid.length; r++) {
            for (int c = 0; c < grid[0].length; c++) {
                if (grid[r][c] == '1') {
                    count++;
                    sink(grid, r, c);
                }
            }
        }
        return count;
    }

    private void sink(char[][] grid, int r, int c) {
        if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length
                || grid[r][c] != '1') {
            return;
        }
        grid[r][c] = '0';            // mark visited
        sink(grid, r + 1, c);
        sink(grid, r - 1, c);
        sink(grid, r, c + 1);
        sink(grid, r, c - 1);
    }
}
```
**Complexity:** O(m·n) time — each cell is visited once. O(m·n) worst-case space for the recursion stack (a grid that is one giant island).

**Key insight / follow-up:** Sinking land to water as you visit it doubles as the visited-set, so no extra memory beyond the call stack. For very large grids prone to stack overflow, switch the DFS to an explicit BFS queue, or use union-find (join each land cell with its right/down land neighbor; the answer is the number of components among land cells). The same flood-fill skeleton powers LC695, LC130, and LC417 below.

---

### LC695 — Max Area of Island *(medium)*
**Problem:** Given an `m x n` binary grid of `0`s and `1`s, an island is a maximal 4-directionally connected group of `1`s. Return the area (cell count) of the largest island, or `0` if there is none.

**Approach:** Identical flood-fill to LC200, but instead of just sinking, DFS *returns the size* of the component it sank (1 for the current cell plus the sizes returned by its four neighbors). Track the running maximum.
```java
class Solution {
    public int maxAreaOfIsland(int[][] grid) {
        int max = 0;
        for (int r = 0; r < grid.length; r++) {
            for (int c = 0; c < grid[0].length; c++) {
                if (grid[r][c] == 1) {
                    max = Math.max(max, area(grid, r, c));
                }
            }
        }
        return max;
    }

    private int area(int[][] grid, int r, int c) {
        if (r < 0 || r >= grid.length || c < 0 || c >= grid[0].length
                || grid[r][c] != 1) {
            return 0;
        }
        grid[r][c] = 0;              // sink so it isn't recounted
        return 1
                + area(grid, r + 1, c)
                + area(grid, r - 1, c)
                + area(grid, r, c + 1)
                + area(grid, r, c - 1);
    }
}
```
**Complexity:** O(m·n) time, O(m·n) worst-case recursion stack.

**Key insight / follow-up:** Making the recursive call *return a count* turns the same traversal that LC200 used for counting components into an aggregator — a common upgrade pattern (count → sum → max). Sinking the cell *before* recursing prevents infinite mutual recursion between adjacent cells.

---

### LC994 — Rotting Oranges *(medium)*
**Problem:** In an `m x n` grid each cell is `0` (empty), `1` (fresh orange), or `2` (rotten orange). Every minute, any fresh orange that is 4-directionally adjacent to a rotten one becomes rotten. Return the minimum number of minutes until no fresh orange remains, or `-1` if some fresh orange can never rot.

**Approach:** Multi-source BFS. Seed the queue with *all* rotten oranges at minute 0 and count the fresh ones. Process the queue level by level (one level = one minute), rotting fresh neighbors and enqueuing them. The answer is the number of levels needed; if any fresh orange is left, return `-1`.
```java
import java.util.ArrayDeque;
import java.util.Queue;

class Solution {
    public int orangesRotting(int[][] grid) {
        int rows = grid.length, cols = grid[0].length;
        Queue<int[]> queue = new ArrayDeque<>();
        int fresh = 0;
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == 2) queue.offer(new int[]{r, c});
                else if (grid[r][c] == 1) fresh++;
            }
        }
        if (fresh == 0) return 0;             // nothing to rot

        int[][] dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
        int minutes = 0;
        while (!queue.isEmpty() && fresh > 0) {
            minutes++;
            for (int i = queue.size(); i > 0; i--) {  // drain one full level
                int[] cell = queue.poll();
                for (int[] d : dirs) {
                    int nr = cell[0] + d[0], nc = cell[1] + d[1];
                    if (nr >= 0 && nr < rows && nc >= 0 && nc < cols
                            && grid[nr][nc] == 1) {
                        grid[nr][nc] = 2;     // rot it
                        fresh--;
                        queue.offer(new int[]{nr, nc});
                    }
                }
            }
        }
        return fresh == 0 ? minutes : -1;
    }
}
```
**Complexity:** O(m·n) time, O(m·n) space for the queue.

**Key insight / follow-up:** Multi-source BFS — seeding *every* rotten orange simultaneously — gives the true minimum time, because all sources expand in lockstep. The `for (int i = queue.size(); i > 0; i--)` pattern processes exactly one BFS layer per loop, which is what lets the layer count equal elapsed minutes. Snapshot `queue.size()` *before* the inner loop, since the queue grows as you add new rot.

---

### LC133 — Clone Graph *(medium)*
**Problem:** Given a reference to a node in a connected, undirected graph, return a deep copy (clone). Each node has an `int val` and a `List<Node> neighbors`. The clone must contain entirely new nodes mirroring the original's structure. Return `null` if the input is `null`.

**Approach:** Traverse (DFS or BFS) while maintaining a map from original node → its clone. Before recursing into a neighbor, check the map: if already cloned, reuse it; otherwise create the clone, record it, then recurse to wire up *its* neighbors. The map both deduplicates and breaks cycles.
```java
import java.util.HashMap;
import java.util.Map;
import java.util.ArrayList;
import java.util.List;

class Node {
    public int val;
    public List<Node> neighbors;
    public Node(int val) { this.val = val; this.neighbors = new ArrayList<>(); }
}

class Solution {
    private final Map<Node, Node> clones = new HashMap<>();

    public Node cloneGraph(Node node) {
        if (node == null) return null;
        if (clones.containsKey(node)) return clones.get(node);

        Node copy = new Node(node.val);
        clones.put(node, copy);                 // record BEFORE recursing
        for (Node neighbor : node.neighbors) {
            copy.neighbors.add(cloneGraph(neighbor));
        }
        return copy;
    }
}
```
**Complexity:** O(V + E) time (each node and edge visited once), O(V) space for the map and recursion.

**Key insight / follow-up:** Inserting the clone into the map *before* recursing into neighbors is the critical ordering — it is what stops infinite recursion when the graph has cycles (a neighbor pointing back to the current node finds it already mapped). The same original→copy map technique deep-copies any reference-linked structure (e.g. LC138 Copy List with Random Pointer).

---

### LC207 — Course Schedule *(medium)*
**Problem:** There are `numCourses` courses labeled `0` to `numCourses - 1`. `prerequisites[i] = [a, b]` means you must take course `b` before course `a`. Return `true` if you can finish all courses, i.e. the prerequisite graph has no cycle.

**Approach:** Cycle detection via Kahn's topological sort. Build an adjacency list and an in-degree array. Enqueue every course with in-degree 0, then repeatedly remove a course and decrement its successors' in-degrees, enqueuing any that hit 0. If all courses are processed, there is no cycle.
```java
import java.util.ArrayDeque;
import java.util.ArrayList;
import java.util.List;
import java.util.Queue;

class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        List<List<Integer>> graph = new ArrayList<>();
        for (int i = 0; i < numCourses; i++) graph.add(new ArrayList<>());
        int[] indegree = new int[numCourses];
        for (int[] p : prerequisites) {
            graph.get(p[1]).add(p[0]);   // edge b -> a
            indegree[p[0]]++;
        }

        Queue<Integer> queue = new ArrayDeque<>();
        for (int i = 0; i < numCourses; i++) {
            if (indegree[i] == 0) queue.offer(i);
        }

        int taken = 0;
        while (!queue.isEmpty()) {
            int course = queue.poll();
            taken++;
            for (int next : graph.get(course)) {
                if (--indegree[next] == 0) queue.offer(next);
            }
        }
        return taken == numCourses;
    }
}
```
**Complexity:** O(V + E) time, O(V + E) space.

**Key insight / follow-up:** A directed graph is acyclic iff a topological order exists, and Kahn's algorithm produces one iff every node eventually reaches in-degree 0. If a cycle exists, those nodes never enter the queue, so `taken < numCourses`. The DFS alternative uses three colors (unvisited / in-progress / done) and reports a cycle when it revisits an in-progress node; LC210 below just extends this to emit the order.

---

### LC210 — Course Schedule II *(medium)*
**Problem:** Same setup as LC207: `numCourses` courses and `prerequisites[i] = [a, b]` (take `b` before `a`). Return *any* valid ordering in which to take all courses. If it is impossible (a cycle exists), return an empty array.

**Approach:** Kahn's topological sort, but record the order in which courses are removed from the queue. That dequeue sequence is a valid topological order. If fewer than `numCourses` are produced, a cycle exists and the answer is empty.
```java
import java.util.ArrayDeque;
import java.util.ArrayList;
import java.util.List;
import java.util.Queue;

class Solution {
    public int[] findOrder(int numCourses, int[][] prerequisites) {
        List<List<Integer>> graph = new ArrayList<>();
        for (int i = 0; i < numCourses; i++) graph.add(new ArrayList<>());
        int[] indegree = new int[numCourses];
        for (int[] p : prerequisites) {
            graph.get(p[1]).add(p[0]);
            indegree[p[0]]++;
        }

        Queue<Integer> queue = new ArrayDeque<>();
        for (int i = 0; i < numCourses; i++) {
            if (indegree[i] == 0) queue.offer(i);
        }

        int[] order = new int[numCourses];
        int idx = 0;
        while (!queue.isEmpty()) {
            int course = queue.poll();
            order[idx++] = course;
            for (int next : graph.get(course)) {
                if (--indegree[next] == 0) queue.offer(next);
            }
        }
        return idx == numCourses ? order : new int[0];
    }
}
```
**Complexity:** O(V + E) time, O(V + E) space.

**Key insight / follow-up:** The only change from LC207 is capturing the dequeue order into the result array. There can be many valid orders; Kahn's gives one determined by queue insertion order (a `PriorityQueue` would yield the lexicographically smallest). A DFS approach builds the order by pushing nodes onto a stack in post-order and reversing it.

---

### LC684 — Redundant Connection *(medium)*
**Problem:** A tree on `n` nodes labeled `1..n` had exactly one extra edge added, forming a single cycle; the result is given as `edges`, where `edges[i] = [u, v]`. Return the edge that can be removed so the graph becomes a tree again. If multiple answers exist, return the one that appears last in the input.

**Approach:** Union-find. Process edges in order, unioning each `(u, v)`. The first edge whose two endpoints are *already* in the same set is the one that closes a cycle — return it. Because we scan in input order, this is automatically the last such edge for the given guarantee.
```java
class Solution {
    public int[] findRedundantConnection(int[][] edges) {
        int n = edges.length;                 // nodes are 1..n
        int[] parent = new int[n + 1];
        for (int i = 1; i <= n; i++) parent[i] = i;

        for (int[] e : edges) {
            if (find(parent, e[0]) == find(parent, e[1])) {
                return e;                     // already connected -> this closes the cycle
            }
            parent[find(parent, e[0])] = find(parent, e[1]);   // union
        }
        return new int[0];                    // unreachable per problem guarantee
    }

    private int find(int[] parent, int x) {
        while (parent[x] != x) {
            parent[x] = parent[parent[x]];    // path compression (halving)
            x = parent[x];
        }
        return x;
    }
}
```
**Complexity:** O(n·α(n)) ≈ O(n) time, O(n) space.

**Key insight / follow-up:** Union-find shines for incremental connectivity: each edge either joins two components or, if its endpoints already share a root, must be the cycle-closing edge. Returning *immediately* on the first such edge yields the last redundant edge precisely because the input is scanned front to back. The harder LC685 (directed graph) layers in-degree checks on top of the same union-find idea.

---

### LC547 — Number of Provinces *(medium)*
**Problem:** There are `n` cities. `isConnected[i][j] = 1` means city `i` and city `j` are directly connected (the matrix is symmetric, with `isConnected[i][i] = 1`). A province is a group of directly or indirectly connected cities. Return the number of provinces.

**Approach:** This is counting connected components in an undirected graph given as an adjacency matrix. Union-find: union every pair `(i, j)` with `isConnected[i][j] == 1`; the number of remaining components is the answer. (A DFS over the matrix marking visited cities works equally well.)
```java
class Solution {
    public int findCircleNum(int[][] isConnected) {
        int n = isConnected.length;
        DSU dsu = new DSU(n);
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {   // upper triangle (symmetric matrix)
                if (isConnected[i][j] == 1) dsu.union(i, j);
            }
        }
        return dsu.components;
    }

    private static class DSU {
        int[] parent, rank;
        int components;
        DSU(int n) {
            parent = new int[n];
            rank = new int[n];
            components = n;
            for (int i = 0; i < n; i++) parent[i] = i;
        }
        int find(int x) {
            if (parent[x] != x) parent[x] = find(parent[x]);
            return parent[x];
        }
        void union(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return;
            if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
            parent[rb] = ra;
            if (rank[ra] == rank[rb]) rank[ra]++;
            components--;
        }
    }
}
```
**Complexity:** O(n²·α(n)) time (the matrix has n² entries to scan), O(n) space for the DSU.

**Key insight / follow-up:** Maintaining a live `components` counter that decrements on each successful union gives the answer for free — no second pass to count distinct roots. Iterating only the upper triangle (`j = i + 1`) avoids redundant unions on the symmetric matrix. This is LC200's "count components" recast onto an adjacency matrix instead of a grid.

---

### LC130 — Surrounded Regions *(medium)*
**Problem:** Given an `m x n` board of `'X'` and `'O'`, capture all regions that are 4-directionally surrounded by `'X'` by flipping every `'O'` in such a region to `'X'`. An `'O'` is *not* captured if it is connected to an `'O'` on the border of the board.

**Approach:** Invert the problem: instead of finding surrounded regions, find the *safe* ones. DFS/BFS from every `'O'` on the four borders, marking all reachable `'O'`s with a temporary sentinel (`'S'`). After that, every remaining `'O'` is truly surrounded → flip to `'X'`; restore each `'S'` back to `'O'`.
```java
class Solution {
    public void solve(char[][] board) {
        int rows = board.length, cols = board[0].length;
        // 1. mark border-connected 'O's as safe
        for (int r = 0; r < rows; r++) {
            guard(board, r, 0);
            guard(board, r, cols - 1);
        }
        for (int c = 0; c < cols; c++) {
            guard(board, 0, c);
            guard(board, rows - 1, c);
        }
        // 2. capture the rest, restore the safe ones
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (board[r][c] == 'O') board[r][c] = 'X';
                else if (board[r][c] == 'S') board[r][c] = 'O';
            }
        }
    }

    private void guard(char[][] board, int r, int c) {
        if (r < 0 || r >= board.length || c < 0 || c >= board[0].length
                || board[r][c] != 'O') {
            return;
        }
        board[r][c] = 'S';           // safe sentinel
        guard(board, r + 1, c);
        guard(board, r - 1, c);
        guard(board, r, c + 1);
        guard(board, r, c - 1);
    }
}
```
**Complexity:** O(m·n) time, O(m·n) worst-case recursion stack.

**Key insight / follow-up:** Flipping the question — "which `'O'`s are safe?" instead of "which are surrounded?" — turns an awkward enclosure test into a straightforward border flood-fill. Using a third temporary symbol (`'S'`) cleanly separates "known safe" from "untouched" during the final sweep. Union-find can model this by joining every border `'O'` to a virtual node and capturing only the cells not connected to it.

---

### LC417 — Pacific Atlantic Water Flow *(medium)*
**Problem:** Given an `m x n` matrix `heights` where `heights[r][c]` is the cell's height, the Pacific Ocean touches the top and left edges, the Atlantic touches the bottom and right edges. Water flows from a cell to a 4-directionally adjacent cell of *equal or lower* height. Return all coordinates `[r, c]` from which water can reach **both** oceans.

**Approach:** Reverse the flow. Instead of asking which cells drain to an ocean, ask which cells an ocean can "climb up" to: starting from each ocean's border, DFS to neighbors of *equal or greater* height. Run this from the Pacific borders and from the Atlantic borders, producing two boolean reachability sets. The answer is their intersection.
```java
import java.util.ArrayList;
import java.util.List;

class Solution {
    private int rows, cols;
    private int[][] heights;

    public List<List<Integer>> pacificAtlantic(int[][] heights) {
        this.heights = heights;
        this.rows = heights.length;
        this.cols = heights[0].length;
        boolean[][] pacific = new boolean[rows][cols];
        boolean[][] atlantic = new boolean[rows][cols];

        for (int r = 0; r < rows; r++) {
            dfs(r, 0, pacific);            // left edge -> Pacific
            dfs(r, cols - 1, atlantic);    // right edge -> Atlantic
        }
        for (int c = 0; c < cols; c++) {
            dfs(0, c, pacific);            // top edge -> Pacific
            dfs(rows - 1, c, atlantic);    // bottom edge -> Atlantic
        }

        List<List<Integer>> result = new ArrayList<>();
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (pacific[r][c] && atlantic[r][c]) {
                    result.add(List.of(r, c));
                }
            }
        }
        return result;
    }

    private void dfs(int r, int c, boolean[][] reach) {
        reach[r][c] = true;
        int[][] dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
        for (int[] d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols
                    && !reach[nr][nc]
                    && heights[nr][nc] >= heights[r][c]) {   // climbing up
                dfs(nr, nc, reach);
            }
        }
    }
}
```
**Complexity:** O(m·n) time (each cell visited at most once per ocean), O(m·n) space for the two reachability grids and recursion.

**Key insight / follow-up:** Searching *from* the oceans inward (accepting only equal-or-higher neighbors) is exponentially cheaper than searching from every cell outward — two flood-fills total instead of m·n separate drain checks. The final answer is just the AND of the two reachability matrices. The condition flips from "flows to lower" to "reachable by climbing to higher" precisely because the search direction is reversed.

---

### LC208 — Implement Trie (Prefix Tree) *(medium)*
**Problem:** Implement a trie (prefix tree) supporting: `insert(word)` adds a word; `search(word)` returns `true` if the exact word was inserted; `startsWith(prefix)` returns `true` if any inserted word begins with `prefix`. All inputs consist of lowercase English letters.

**Approach:** Each node holds an array of 26 child pointers (one per lowercase letter) and a boolean `isWord` flag. `insert` walks/creates nodes for each character and flags the final node. `search` and `startsWith` walk the characters; they differ only in whether the final node's `isWord` flag must be set.
```java
class Trie {
    private static class Node {
        Node[] children = new Node[26];
        boolean isWord;
    }

    private final Node root = new Node();

    public void insert(String word) {
        Node cur = root;
        for (char ch : word.toCharArray()) {
            int i = ch - 'a';
            if (cur.children[i] == null) cur.children[i] = new Node();
            cur = cur.children[i];
        }
        cur.isWord = true;
    }

    public boolean search(String word) {
        Node node = walk(word);
        return node != null && node.isWord;
    }

    public boolean startsWith(String prefix) {
        return walk(prefix) != null;
    }

    private Node walk(String s) {                 // follow s, or null if it falls off
        Node cur = root;
        for (char ch : s.toCharArray()) {
            int i = ch - 'a';
            if (cur.children[i] == null) return null;
            cur = cur.children[i];
        }
        return cur;
    }
}
```
**Complexity:** O(L) per operation, where L is the word/prefix length. Space is O(total characters inserted · 26) worst case.

**Key insight / follow-up:** The whole difference between `search` and `startsWith` is one boolean check on the terminal node — both reuse the same `walk` traversal. The `isWord` flag is essential: without it, inserting `"apple"` would make `search("app")` wrongly return true. For sparse alphabets or Unicode, swap the fixed 26-array for a `HashMap<Character, Node>` to save memory at the cost of constant-factor lookup speed.

---

### LC211 — Design Add and Search Words Data Structure *(medium)*
**Problem:** Design `WordDictionary` with `addWord(word)` and `search(word)`. `search` may contain the wildcard `'.'`, which matches **any single letter**. Return `true` if any stored word matches. All other characters are lowercase letters. Example: after adding `"bad"`, `"dad"`, `"mad"`, `search("b..")` is `true` and `search("..d")` is `true`.

**Approach:** A trie for storage, with a recursive `search` that branches on the wildcard. For a normal character, descend into the single matching child. For `'.'`, try *all* non-null children and succeed if any branch matches the rest of the pattern.
```java
class WordDictionary {
    private static class Node {
        Node[] children = new Node[26];
        boolean isWord;
    }

    private final Node root = new Node();

    public void addWord(String word) {
        Node cur = root;
        for (char ch : word.toCharArray()) {
            int i = ch - 'a';
            if (cur.children[i] == null) cur.children[i] = new Node();
            cur = cur.children[i];
        }
        cur.isWord = true;
    }

    public boolean search(String word) {
        return dfs(word, 0, root);
    }

    private boolean dfs(String word, int idx, Node node) {
        if (node == null) return false;
        if (idx == word.length()) return node.isWord;

        char ch = word.charAt(idx);
        if (ch == '.') {
            for (Node child : node.children) {       // try every branch
                if (dfs(word, idx + 1, child)) return true;
            }
            return false;
        }
        return dfs(word, idx + 1, node.children[ch - 'a']);
    }
}
```
**Complexity:** `addWord` is O(L). `search` is O(L) without wildcards, but up to O(26^d · L) when there are d wildcards (each `'.'` fans out to all children). Space O(total characters · 26).

**Key insight / follow-up:** The wildcard forces the search to become a recursive DFS rather than a simple loop, because a `'.'` requires exploring multiple subtrees. Passing the current index `idx` plus the current `node` is the clean way to express "match the suffix from here." Leading wildcards are the expensive case; in practice patterns rarely have many `'.'`, so the exponential bound is loose.

---

### LC1268 — Search Suggestions System *(medium)*
**Problem:** Given a list of `products` and a `searchWord`, after each character typed in `searchWord` return up to 3 lexicographically smallest product names that share the typed prefix. Return a list of lists, one inner list per prefix of `searchWord` (lengths 1..searchWord.length()).

**Approach:** Sort the products once so any contiguous run sharing a prefix is automatically in lexicographic order. For each growing prefix, binary-search (or two-pointer narrow) the range of products that start with it and take the first three. A trie that stores up to 3 suggestions per node is the alternative; the sort-and-search version is simpler to get right.
```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

class Solution {
    public List<List<String>> suggestedProducts(String[] products, String searchWord) {
        Arrays.sort(products);                       // lexicographic order
        List<List<String>> result = new ArrayList<>();
        int lo = 0, hi = products.length - 1;
        StringBuilder prefix = new StringBuilder();

        for (char ch : searchWord.toCharArray()) {
            prefix.append(ch);
            int k = prefix.length() - 1;
            // narrow the window to products still matching the prefix
            while (lo <= hi && (products[lo].length() <= k
                    || products[lo].charAt(k) != ch)) {
                lo++;
            }
            while (lo <= hi && (products[hi].length() <= k
                    || products[hi].charAt(k) != ch)) {
                hi--;
            }
            List<String> suggestions = new ArrayList<>();
            for (int i = lo; i < Math.min(lo + 3, hi + 1); i++) {
                suggestions.add(products[i]);
            }
            result.add(suggestions);
        }
        return result;
    }
}
```
**Complexity:** O(n log n) to sort plus O(L · n) worst case for the two-pointer narrowing across all prefixes (L = searchWord length). Space O(1) beyond the output.

**Key insight / follow-up:** Sorting once means the three lexicographically smallest matches are always the *first three* of the surviving window — no per-prefix re-sort. The `[lo, hi]` window only ever shrinks as the prefix grows, so the pointers never move backward, keeping the narrowing cheap. The trie variant precomputes 3 suggestions at each node and answers each prefix in O(L), preferable when many queries hit the same product set.

---

### LC743 — Network Delay Time *(medium)*
**Problem:** A network of `n` nodes labeled `1..n` is given as `times`, where `times[i] = [u, v, w]` is a directed edge from `u` to `v` with travel time `w`. A signal starts at node `k`. Return the time for *all* nodes to receive the signal (the maximum shortest-path distance from `k`), or `-1` if some node is unreachable.

**Approach:** Single-source shortest paths with **Dijkstra**. Use a min-heap keyed by current distance. Pop the closest unfinalized node, finalize its distance, and relax its outgoing edges. The answer is the largest finalized distance; if any node was never reached, return `-1`.
```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.PriorityQueue;

class Solution {
    public int networkDelayTime(int[][] times, int n, int k) {
        List<int[]>[] graph = new List[n + 1];
        for (int i = 1; i <= n; i++) graph[i] = new ArrayList<>();
        for (int[] t : times) graph[t[0]].add(new int[]{t[1], t[2]});  // (to, weight)

        int[] dist = new int[n + 1];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[k] = 0;

        // heap of (distance, node), ordered by distance
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
        pq.offer(new int[]{0, k});

        while (!pq.isEmpty()) {
            int[] top = pq.poll();
            int d = top[0], node = top[1];
            if (d > dist[node]) continue;             // stale heap entry
            for (int[] edge : graph[node]) {
                int next = edge[0], nd = d + edge[1];
                if (nd < dist[next]) {
                    dist[next] = nd;
                    pq.offer(new int[]{nd, next});
                }
            }
        }

        int max = 0;
        for (int i = 1; i <= n; i++) {
            if (dist[i] == Integer.MAX_VALUE) return -1;   // unreachable
            max = Math.max(max, dist[i]);
        }
        return max;
    }
}
```
**Complexity:** O(E log V) time (each edge can push one heap entry), O(V + E) space.

**Key insight / follow-up:** "Time for all nodes" is the *maximum* of the single-source shortest distances — Dijkstra computes them all from `k` in one run. The `if (d > dist[node]) continue;` guard skips stale entries left in the heap after a shorter path was found, which keeps the lazy-deletion heap correct without a decrease-key operation. Dijkstra requires non-negative weights; with negative edges you would need Bellman-Ford (see LC787).

---

### LC787 — Cheapest Flights Within K Stops *(medium)*
**Problem:** Given `n` cities and `flights[i] = [from, to, price]` (directed), find the cheapest price from `src` to `dst` using **at most `k` stops** (i.e. at most `k + 1` edges). Return `-1` if no such route exists.

**Approach:** Bellman-Ford bounded to `k + 1` relaxation rounds. Each round relaxes every edge *once* using distances from the **previous** round only (a snapshot), so after round `i` you have the cheapest cost reachable in ≤ `i` edges. Snapshotting prevents one round from chaining multiple edges, which would violate the stop limit.
```java
import java.util.Arrays;

class Solution {
    public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
        int[] dist = new int[n];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[src] = 0;

        for (int round = 0; round <= k; round++) {     // k+1 edges allowed
            int[] prev = dist.clone();                 // snapshot of last round
            for (int[] f : flights) {
                int from = f[0], to = f[1], price = f[2];
                if (prev[from] != Integer.MAX_VALUE
                        && prev[from] + price < dist[to]) {
                    dist[to] = prev[from] + price;
                }
            }
        }
        return dist[dst] == Integer.MAX_VALUE ? -1 : dist[dst];
    }
}
```
**Complexity:** O(k · E) time, O(n) space.

**Key insight / follow-up:** Cloning `dist` into `prev` each round is the crux: relaxing against the previous snapshot guarantees each round adds *exactly one* more edge to any path, so `k + 1` rounds = at most `k` stops. Plain Dijkstra fails here because the cheapest path may use more edges than the limit — the cost dimension and the stop dimension must be tracked together. A Dijkstra variant works if you push `(cost, node, stopsRemaining)` and treat stops as a second state dimension.

---

### LC1631 — Path With Minimum Effort *(medium)*
**Problem:** Given an `m x n` grid `heights`, you start at the top-left `(0,0)` and travel to the bottom-right `(m-1, n-1)`, moving 4-directionally. A route's *effort* is the maximum absolute height difference between consecutive cells along it. Return the minimum possible effort.

**Approach:** Dijkstra on the grid where a path's "cost" is the **maximum edge weight** along it (a minimax path) rather than the sum. The min-heap is keyed by the worst step seen so far; relaxing a neighbor uses `max(currentEffort, |height difference|)`. The first time the destination is popped, that effort is optimal.
```java
import java.util.PriorityQueue;

class Solution {
    public int minimumEffortPath(int[][] heights) {
        int rows = heights.length, cols = heights[0].length;
        int[][] effort = new int[rows][cols];
        for (int[] row : effort) java.util.Arrays.fill(row, Integer.MAX_VALUE);
        effort[0][0] = 0;

        // heap of (effortSoFar, row, col)
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
        pq.offer(new int[]{0, 0, 0});
        int[][] dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};

        while (!pq.isEmpty()) {
            int[] top = pq.poll();
            int e = top[0], r = top[1], c = top[2];
            if (r == rows - 1 && c == cols - 1) return e;   // destination finalized
            if (e > effort[r][c]) continue;                 // stale entry
            for (int[] d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
                    int step = Math.abs(heights[nr][nc] - heights[r][c]);
                    int ne = Math.max(e, step);             // minimax: worst step so far
                    if (ne < effort[nr][nc]) {
                        effort[nr][nc] = ne;
                        pq.offer(new int[]{ne, nr, nc});
                    }
                }
            }
        }
        return 0;   // single-cell grid
    }
}
```
**Complexity:** O(m·n·log(m·n)) time, O(m·n) space.

**Key insight / follow-up:** Dijkstra generalizes from "minimize the sum of weights" to "minimize the maximum weight" by swapping `dist + w` for `max(dist, w)` in the relaxation — the greedy heap property still holds because `max` is monotonic. Returning the moment the destination is popped is safe: Dijkstra finalizes a node's optimal cost at pop time. Alternatives are binary search on the answer with a BFS/DFS feasibility check, or union-find adding edges in increasing weight until start and end connect.

---

*[← DSA bank index](README.md)*
