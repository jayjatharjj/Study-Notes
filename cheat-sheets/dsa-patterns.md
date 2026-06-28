# Cheat-Sheet — DSA Patterns, Templates & Big-O

> Last-minute revision. Read the **signal** in the problem statement, jump to the **pattern**, paste the **template**. One page, offline, no fluff.

---

## How to use this sheet

1. Read the problem once. Note the **input size `n`** (sets the complexity budget — see the n-table at the end).
2. Scan the **Pattern recognition table** for the phrase that matches what the problem *says*.
3. Grab the matching **Java 17 template**, adapt state/transition.
4. Sanity-check complexity against the budget before coding.

---

## 1. Pattern recognition — signal → pattern

| Signal in the problem (what it says) | Pattern |
|---|---|
| Sorted array, find a **pair/triplet** with a sum/product/area relationship; reverse/partition in place | **Two pointers — opposite ends** |
| Detect a **cycle** in a linked list; find the **middle**; "happens to meet" | **Two pointers — fast/slow (Floyd)** |
| "Contiguous **subarray/substring** of a **fixed length `k`**"; max/avg of every window of size k | **Sliding window — fixed** |
| "Longest/shortest/smallest **substring/subarray** satisfying a constraint" (≤ K distinct, sum ≥ target, no repeats) | **Sliding window — variable** |
| "**Range sum** between i..j", "sum equals K", many subarray-sum queries | **Prefix sum** (`prefix[j+1]-prefix[i]`); pair with HashMap for "count subarrays = K" |
| "Have I **seen** this before?", complement lookup, count frequencies, group anagrams, dedupe | **Hashing / frequency map** |
| Array is **sorted** (or rotated-sorted); "find target / insertion point / first-or-last occurrence" | **Binary search (on a sorted array)** |
| "**Minimize the max** / maximize the min", "smallest feasible X", answer is monotonic ("if X works, X+1 works") | **Binary search on the answer** |
| "Generate **all** permutations/combinations/subsets", "all valid placements", N-Queens, partitions | **Recursion / backtracking** |
| **Grid** flood-fill, islands, regions, shortest path on **unweighted** cells | **DFS (regions) / BFS (shortest steps)** |
| **Graph** connectivity / reachability / shortest hops on unweighted edges | **DFS / BFS on adjacency** |
| "**Order tasks** given dependencies", "course schedule", detect cycle in a DAG | **Topological sort (Kahn / DFS)** |
| "Are these in the **same group**?", connected components, "redundant connection", dynamic merging | **Union-Find (DSU)** |
| Shortest path with **non-negative weighted** edges, "cheapest/fastest route" | **Dijkstra (PriorityQueue)** |
| "Word/prefix lookup", autocomplete, "starts with", many strings sharing prefixes | **Trie** |
| "**Next greater/smaller** element", "largest rectangle", "daily temperatures", span problems | **Monotonic stack** |
| "**Top K** / K-th largest/smallest/closest/frequent", "merge K sorted lists", running median | **Heap (PriorityQueue) / Top-K** |
| Input is a list of **intervals**; merge/insert/count overlaps, meeting rooms, min removals | **Intervals — sort by start (merge) or by end (greedy/scheduling)** |
| "Ways to reach step `i`", "max/min over choices building on smaller `i`" (Fibonacci-shaped) | **DP — 1D** |
| Two sequences / a grid; "edit distance", "paths in grid", "LCS" | **DP — 2D** |
| "Pick items under a **capacity/budget** to max value", subset-sum, coin change | **DP — knapsack (0/1 or unbounded)** |
| "Longest **increasing** subsequence", "longest chain" | **DP — LIS** (O(n²) or O(n log n)) |
| "Longest **common** subsequence/substring", sequence alignment | **DP — LCS** |
| "Burst balloons", "matrix chain", "merge stones", optimal over a **range `[i..j]`** | **DP — interval** |
| "Max activities / min resources", "always take the locally best and it provably wins", sort-then-sweep | **Greedy** |
| "Without `+`/`-`", subsets via masks, "single number", count bits, XOR tricks | **Bit manipulation** |
| Numbers are **`1..n` (or `0..n`)**, "find the missing/duplicate in place, O(1) space" | **Cyclic sort** |

### Quick disambiguators

- **Two pointers vs sliding window:** both walk indices, but a window tracks an *aggregate over a contiguous range*; two-pointers-opposite-ends *eliminate one side per step*. Window = one direction, both pointers move forward.
- **DFS vs BFS on a grid/graph:** need *shortest* number of steps on **unweighted** edges → **BFS**. Need to *explore/count regions* or you have a recursion-friendly structure → **DFS**.
- **BFS vs Dijkstra:** all edges weight 1 → BFS. Non-negative arbitrary weights → Dijkstra. Negative weights → Bellman-Ford (not covered here).
- **Greedy vs DP:** if a locally-optimal choice can be *proven* globally optimal (exchange argument) → greedy. If a locally-large choice can force a worse global result (e.g. Coin Change `[1,3,4]`, amount 6) → DP.
- **Binary search on array vs on answer:** the *array* is sorted → search the array. The *answer space* is monotonic but the input isn't sorted → search the answer.

---

## 2. Java 17 templates

### Sliding window (variable — longest window satisfying a constraint)

```java
// Longest substring with at most K distinct chars. Shrink while invalid.
int longestWindow(String s, int k) {
    int[] freq = new int[128];
    int distinct = 0, left = 0, best = 0;
    for (int right = 0; right < s.length(); right++) {
        if (freq[s.charAt(right)]++ == 0) distinct++;       // new char entered
        while (distinct > k) {                              // shrink until valid
            if (--freq[s.charAt(left)] == 0) distinct--;
            left++;
        }
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

### Binary search (on a sorted array — lower bound)

```java
// First index with a[i] >= target  (insertion point). Returns a.length if none.
int lowerBound(int[] a, int target) {
    int lo = 0, hi = a.length;            // hi is EXCLUSIVE
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;     // overflow-safe midpoint
        if (a[mid] < target) lo = mid + 1;
        else hi = mid;                    // mid could be the answer — keep it
    }
    return lo;
}
```

### Binary search on the answer (minimize a feasible value)

```java
// Smallest capacity that lets us finish within `limit` (feasible() is monotonic).
int minFeasible(int lo, int hi) {        // [lo, hi] is the candidate answer range
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (feasible(mid)) hi = mid;     // mid works → maybe smaller works too
        else lo = mid + 1;               // mid too small → go bigger
    }
    return lo;                           // first value that is feasible
}
// boolean feasible(int x) { ... simulate with candidate x, return true if it satisfies ... }
```

### Backtracking (subsets / permutations / combinations skeleton)

```java
// All subsets. For permutations: track a `used[]`; for combinations: pass a start index.
void backtrack(int[] nums, int start, List<Integer> path, List<List<Integer>> out) {
    out.add(new ArrayList<>(path));               // record (every node is a subset)
    for (int i = start; i < nums.length; i++) {
        // if (i > start && nums[i] == nums[i-1]) continue;  // skip dups (needs sorted input)
        path.add(nums[i]);                        // choose
        backtrack(nums, i + 1, path, out);        // explore (i+1 = no reuse; i = reuse)
        path.remove(path.size() - 1);             // un-choose
    }
}
```

### BFS on a grid (shortest steps, 4-directional)

```java
int bfsGrid(int[][] grid, int sr, int sc) {
    int R = grid.length, C = grid[0].length;
    int[][] DIR = {{1,0},{-1,0},{0,1},{0,-1}};
    boolean[][] seen = new boolean[R][C];
    Deque<int[]> q = new ArrayDeque<>();
    q.add(new int[]{sr, sc}); seen[sr][sc] = true;
    int steps = 0;
    while (!q.isEmpty()) {
        for (int sz = q.size(); sz > 0; sz--) {       // drain one level per step
            int[] cur = q.poll();
            // if (isTarget(cur)) return steps;
            for (int[] d : DIR) {
                int nr = cur[0] + d[0], nc = cur[1] + d[1];
                if (nr < 0 || nr >= R || nc < 0 || nc >= C) continue;
                if (seen[nr][nc] || grid[nr][nc] == 1) continue;   // 1 = wall, adapt
                seen[nr][nc] = true;
                q.add(new int[]{nr, nc});
            }
        }
        steps++;
    }
    return -1;
}
```

### DFS (graph, recursive)

```java
void dfs(int node, List<List<Integer>> adj, boolean[] seen) {
    seen[node] = true;
    // pre-order work here
    for (int next : adj.get(node)) {
        if (!seen[next]) dfs(next, adj, seen);
    }
    // post-order work here (e.g. push to a stack for topo sort)
}
```

### Union-Find (DSU with path compression + union by rank)

```java
class DSU {
    int[] parent, rank;
    DSU(int n) {
        parent = new int[n];
        rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    int find(int x) {                                 // path compression
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }
    boolean union(int a, int b) {                     // false if already joined
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;
        if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
        parent[rb] = ra;
        if (rank[ra] == rank[rb]) rank[ra]++;
        return true;
    }
}
// find/union are effectively O(1) — inverse-Ackermann α(n).
```

### Dijkstra (PriorityQueue, non-negative weights)

```java
int[] dijkstra(List<int[]>[] adj, int src, int n) {  // adj[u] = list of {v, weight}
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;
    PriorityQueue<int[]> pq = new PriorityQueue<>((x, y) -> x[1] - y[1]); // {node, dist}
    pq.add(new int[]{src, 0});
    while (!pq.isEmpty()) {
        int[] cur = pq.poll();
        int u = cur[0], d = cur[1];
        if (d > dist[u]) continue;                    // stale entry — skip
        for (int[] e : adj[u]) {
            int v = e[0], w = e[1];
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.add(new int[]{v, dist[v]});
            }
        }
    }
    return dist;                                       // O(E log V)
}
```

### Trie node

```java
class Trie {
    private final Trie[] next = new Trie[26];
    private boolean end;
    void insert(String word) {
        Trie cur = this;
        for (char c : word.toCharArray()) {
            int i = c - 'a';
            if (cur.next[i] == null) cur.next[i] = new Trie();
            cur = cur.next[i];
        }
        cur.end = true;
    }
    boolean search(String word)   { Trie n = walk(word); return n != null && n.end; }
    boolean startsWith(String pre) { return walk(pre) != null; }
    private Trie walk(String s) {
        Trie cur = this;
        for (char c : s.toCharArray()) {
            cur = cur.next[c - 'a'];
            if (cur == null) return null;
        }
        return cur;
    }
}
```

### Monotonic stack (next greater element)

```java
// For each i, index of the next element greater than a[i]; -1 if none.
int[] nextGreater(int[] a) {
    int n = a.length;
    int[] res = new int[n];
    Arrays.fill(res, -1);
    Deque<Integer> stack = new ArrayDeque<>();        // holds indices, values DECREASING
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && a[stack.peek()] < a[i]) {
            res[stack.pop()] = i;                     // a[i] is the next-greater for popped
        }
        stack.push(i);
    }
    return res;
}
```

### Top-K (min-heap of size K)

```java
// K largest elements. Min-heap keeps the K biggest; root is the smallest of them.
int[] topK(int[] nums, int k) {
    PriorityQueue<Integer> heap = new PriorityQueue<>();  // natural order = min-heap
    for (int x : nums) {
        heap.add(x);
        if (heap.size() > k) heap.poll();             // evict smallest
    }
    int[] res = new int[k];
    for (int i = 0; i < k; i++) res[i] = heap.poll();
    return res;                                       // O(n log k)
}
```

### 0/1 Knapsack (1D rolling array)

```java
// Max value with total weight <= cap. Each item used at most once.
int knapsack(int[] w, int[] val, int cap) {
    int[] dp = new int[cap + 1];                      // dp[c] = best value at capacity c
    for (int i = 0; i < w.length; i++)
        for (int c = cap; c >= w[i]; c--)             // REVERSE → each item used once
            dp[c] = Math.max(dp[c], dp[c - w[i]] + val[i]);
    return dp[cap];
}
// Unbounded knapsack (reuse items): loop c ASCENDING (for c = w[i]; c <= cap; c++).
```

### LIS in O(n log n) (patience sorting)

```java
// Length of the longest strictly increasing subsequence.
int lengthOfLIS(int[] nums) {
    List<Integer> tails = new ArrayList<>();          // tails[k] = smallest tail of len k+1
    for (int x : nums) {
        int lo = 0, hi = tails.size();
        while (lo < hi) {                             // lower bound of x
            int mid = (lo + hi) >>> 1;
            if (tails.get(mid) < x) lo = mid + 1; else hi = mid;
        }
        if (lo == tails.size()) tails.add(x);         // extend
        else tails.set(lo, x);                        // replace to keep tails minimal
    }
    return tails.size();                              // length is the answer, not contents
}
```

---

## 3. Master Big-O

### Data structures (average / worst)

| Structure | Access | Search | Insert | Delete | Notes |
|---|---|---|---|---|---|
| Array (`int[]`) | O(1) | O(n) | O(n) | O(n) | fixed size; insert/delete shifts |
| `ArrayList` | O(1) | O(n) | O(1)* / O(n) | O(n) | *amortized append; mid-insert shifts |
| `LinkedList` | O(n) | O(n) | O(1)† | O(1)† | †at a known node/end; index access is O(n) |
| `HashMap` / `HashSet` | — | O(1) / O(n) | O(1) / O(n) | O(1) / O(n) | worst = all collide; O(log n) after treeify (Java 8+) |
| `TreeMap` / `TreeSet` | — | O(log n) | O(log n) | O(log n) | sorted; red-black tree; range/ceiling/floor queries |
| `LinkedHashMap` | — | O(1) avg | O(1) avg | O(1) avg | HashMap + insertion/access order |
| Heap (`PriorityQueue`) | O(1) peek | O(n) | O(log n) | O(log n) pop | no O(log n) arbitrary search/remove |
| `ArrayDeque` | O(1) ends | O(n) | O(1) ends | O(1) ends | best stack & queue; faster than `Stack`/`LinkedList` |

### Sorting algorithms

| Algorithm | Best | Average | Worst | Space | Stable | Notes |
|---|---|---|---|---|---|---|
| Quicksort | O(n log n) | O(n log n) | O(n²) | O(log n) | No | `Arrays.sort(int[])` (dual-pivot) |
| Mergesort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes | `Arrays.sort(Object[])` / `Collections.sort` |
| Heapsort | O(n log n) | O(n log n) | O(n log n) | O(1) | No | in-place, not cache-friendly |
| Insertion sort | O(n) | O(n²) | O(n²) | O(1) | Yes | great for tiny / nearly-sorted |
| Counting / Radix | O(n + k) | O(n + k) | O(n + k) | O(n + k) | Yes | only for bounded integer keys |

### n vs acceptable complexity (per ~1 second budget)

| Input size `n` | Acceptable complexity | Typical pattern |
|---|---|---|
| n ≤ 12 | O(n!) | brute permutations |
| n ≤ 20 | O(2ⁿ) | subset/bitmask DP, backtracking |
| n ≤ 100 | O(n⁴) | heavy multi-loop DP |
| n ≤ 500 | O(n³) | interval / matrix-chain DP (Floyd-Warshall) |
| n ≤ 5,000 | O(n²) | classic 2D DP, O(n²) LIS |
| n ≤ 10⁵ | O(n log n) | sort, heap, binary search, O(n log n) LIS |
| n ≤ 10⁶–10⁷ | O(n) | single pass, two pointers, prefix sum, counting sort |
| n ≥ 10⁸ | O(log n) / O(1) | direct math / binary search only |

> Rule of thumb: a modern judge does ~10⁸ simple operations/second. Match `n^(complexity)` against ~10⁸ to pick the budget.

---

## 4. Complexity gotchas

- **Recursion = hidden stack space.** Depth-`d` recursion costs **O(d) stack memory** even if it does no allocation. DFS on a degenerate (linked-list-shaped) tree/graph of n nodes is **O(n) stack** → can `StackOverflowError`. Convert to an explicit stack, or use BFS, when depth can approach 10⁴–10⁵.
- **Amortized ≠ per-operation.** `ArrayList.add` is **O(1) amortized** but a single add that triggers a resize copies all n elements → **O(n) that one time**. Over n adds the total is O(n), so each averages O(1). Same logic for `HashMap` rehash. Don't claim worst-case O(1).
- **Why `HashMap` is O(1) average but O(n) worst.** Lookup = hash to a bucket (O(1)), then walk that bucket's chain calling `equals`. With a good hash, buckets hold ~1 entry → O(1). If every key collides into one bucket (bad/constant `hashCode`, or an adversarial set), the chain holds all n entries → **O(n)** walk. Java 8+ treeifies a bucket past 8 entries, capping the worst case at **O(log n)** — but only if keys are `Comparable`.
- **`hashCode`/`equals` contract is load-bearing.** Equal objects **must** share a hash code, or hash collections silently lose them (land in different buckets). Mutating a field used in `hashCode` while the object is in a `HashSet`/`HashMap` makes it "ghost-present": still stored, but `contains` returns false (it now hashes to a different bucket).
- **`String` keys are cheap on repeat.** `String` is immutable and caches its `hashCode` after first computation — ideal as a map key; the hash never drifts and isn't recomputed.
- **PriorityQueue has no cheap arbitrary search.** `contains`/`remove(Object)` are **O(n)** (linear scan), not O(log n). For "decrease-key" in Dijkstra, push a fresh entry and skip stale ones (the `if (d > dist[u]) continue;` guard) instead of removing.
- **`LinkedList` index access is O(n).** `list.get(i)` in a loop is an accidental O(n²). Use an iterator or `ArrayList`.
- **`substring` / building strings in a loop.** Repeated `+` on `String` is O(n²); use `StringBuilder`. `String.substring` is O(k) (copies) since Java 7.
- **`Arrays.sort(int[])` can be O(n²).** Primitive sort is dual-pivot quicksort — adversarial inputs hit the quadratic worst case. Object sort (Timsort/mergesort) is guaranteed O(n log n); box to `Integer[]` if worst-case matters.
- **2D DP space.** An `n×m` table is O(n·m) memory — at n,m=10⁴ that's 10⁸ ints ≈ 400 MB → MLE. Roll to one or two rows when the recurrence only reads the previous row.

---

*Cross-refs: two-pointers + prefix sum → `prep-weeks/week-01/`; 1D DP + knapsack/LIS → `prep-weeks/week-04/`.*
