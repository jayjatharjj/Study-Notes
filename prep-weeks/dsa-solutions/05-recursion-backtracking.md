# DSA Solutions — Recursion & Backtracking

> Backtracking is brute-force search made systematic: you build a candidate one decision at a time, recurse, and **undo** the decision before trying the next. Every problem below is an instance of the same skeleton — what changes is the *choices* available at each step and the *constraint* that prunes dead ends.

**The choose → recurse → un-choose template.** Almost every backtracking solution looks like this:

```java
void backtrack(State state, List<Result> results) {
    if (isComplete(state)) {        // base case: record a finished candidate
        results.add(snapshot(state));
        return;
    }
    for (Choice c : choicesFrom(state)) {
        if (!isValid(c, state)) continue;  // prune
        apply(c, state);            // CHOOSE
        backtrack(state, results);  // RECURSE
        undo(c, state);             // UN-CHOOSE (restore state for the next sibling)
    }
}
```

The un-choose step is what makes it *backtracking* rather than plain recursion: the mutable `state` (a path list, a board, a visited grid) is shared across the whole call tree, so every branch must leave it exactly as it found it.

**The dedupe-on-sorted-input trick.** When the input has duplicates and you must not emit duplicate results, **sort first**, then at each decision level skip a value identical to the previous one *that you already considered at this level*:

```java
Arrays.sort(nums);
// inside the for-loop over candidates starting at index `start`:
for (int i = start; i < nums.length; i++) {
    if (i > start && nums[i] == nums[i - 1]) continue; // skip duplicate siblings
    ...
}
```

The guard is `i > start` (not `i > 0`): we skip a duplicate only when it would start a *sibling* branch at the same tree level — reusing the same value deeper in the recursion (a child) is still allowed. This single line turns "Subsets" into "Subsets II", "Combination Sum" into "Combination Sum II", and "Permutations" into "Permutations II".

---

### LC78 — Subsets *(medium)*
**Problem:** Given an integer array `nums` of **unique** elements, return all possible subsets (the power set). The solution set must not contain duplicate subsets; the order of subsets and of elements within them does not matter. Example: `nums = [1,2,3]` → `[[],[1],[2],[3],[1,2],[1,3],[2,3],[1,2,3]]` (8 subsets).

**Approach:** At each index you make a binary choice — include `nums[i]` or not — and recurse on the rest. Track a `start` index so each element is considered once; every node of the recursion tree (not just the leaves) is a valid subset, so record the current path on entry to every call.

```java
import java.util.*;

class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        backtrack(nums, 0, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int[] nums, int start, List<Integer> path,
                           List<List<Integer>> result) {
        result.add(new ArrayList<>(path));      // every node is a subset
        for (int i = start; i < nums.length; i++) {
            path.add(nums[i]);                  // choose
            backtrack(nums, i + 1, path, result);// recurse on the rest
            path.remove(path.size() - 1);       // un-choose
        }
    }
}
```

**Complexity:** O(n · 2ⁿ) time — there are 2ⁿ subsets and copying each costs up to O(n). O(n) extra space for the recursion stack and path (excluding the output).

**Key insight / follow-up:** Recording the path at *every* node (not only at leaves) is what generates all subset sizes in one traversal. The `start` index enforces a fixed order so `[1,2]` and `[2,1]` aren't both produced. An iterative alternative: start with `[[]]` and for each number append it to copies of every existing subset.

### LC90 — Subsets II *(medium)*
**Problem:** Given an integer array `nums` that **may contain duplicates**, return all possible subsets (the power set). The solution set must not contain duplicate subsets. Example: `nums = [1,2,2]` → `[[],[1],[1,2],[1,2,2],[2],[2,2]]`.

**Approach:** Same recursion as LC78, but **sort first** so equal values are adjacent, then apply the dedupe-on-sorted-input guard: within one recursion level, skip a value identical to the previous sibling already tried. This prevents two branches at the same depth from starting with the same value and producing identical subsets.

```java
import java.util.*;

class Solution {
    public List<List<Integer>> subsetsWithDup(int[] nums) {
        Arrays.sort(nums);                      // bring duplicates together
        List<List<Integer>> result = new ArrayList<>();
        backtrack(nums, 0, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int[] nums, int start, List<Integer> path,
                           List<List<Integer>> result) {
        result.add(new ArrayList<>(path));
        for (int i = start; i < nums.length; i++) {
            if (i > start && nums[i] == nums[i - 1]) continue; // skip duplicate siblings
            path.add(nums[i]);
            backtrack(nums, i + 1, path, result);
            path.remove(path.size() - 1);
        }
    }
}
```

**Complexity:** O(n · 2ⁿ) worst case (all distinct); fewer subsets when duplicates exist. O(n) extra space plus O(log n) for the sort.

**Key insight / follow-up:** The guard is `i > start`, not `i > 0` — a duplicate is allowed to *extend* a path (the child branch) but not to *restart* an identical sibling branch at the same level. Sorting is the enabler: it makes "is this a duplicate of a sibling I already used" a simple adjacent-element check.

### LC46 — Permutations *(medium)*
**Problem:** Given an array `nums` of **distinct** integers, return all the possible permutations in any order. Example: `nums = [1,2,3]` → all 6 orderings `[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]`.

**Approach:** Unlike subsets, order matters and every element must appear, so there's no `start` index — at each position you may pick any not-yet-used element. Track usage with a `boolean[] used` array; record a permutation when the path length reaches `n`.

```java
import java.util.*;

class Solution {
    public List<List<Integer>> permute(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        backtrack(nums, new boolean[nums.length], new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int[] nums, boolean[] used, List<Integer> path,
                           List<List<Integer>> result) {
        if (path.size() == nums.length) {
            result.add(new ArrayList<>(path));  // a full permutation
            return;
        }
        for (int i = 0; i < nums.length; i++) {
            if (used[i]) continue;              // each element once per permutation
            used[i] = true;                     // choose
            path.add(nums[i]);
            backtrack(nums, used, path, result);// recurse
            path.remove(path.size() - 1);       // un-choose
            used[i] = false;
        }
    }
}
```

**Complexity:** O(n · n!) time — n! permutations, O(n) to copy each. O(n) extra space for the recursion depth, path, and `used` array.

**Key insight / follow-up:** The absence of a `start` index (we loop from 0 every call) is precisely what lets the same set of values appear in different orders. An in-place swap variant avoids the `used` array by swapping `nums[start]` with each `nums[i]`, but the `used`-array version is clearer and generalizes to Permutations II.

### LC47 — Permutations II *(medium)*
**Problem:** Given a collection `nums` that **might contain duplicates**, return all possible **unique** permutations in any order. Example: `nums = [1,1,2]` → `[[1,1,2],[1,2,1],[2,1,1]]` (only 3, not 6).

**Approach:** Sort, then combine the `used` array with the dedupe trick. The subtle rule: when the current value equals the previous one, only use it if the previous identical value **has already been used in this path** (`used[i-1]` is true). This forces duplicates to be placed in a fixed left-to-right order, so a given multiset of values is permuted exactly once.

```java
import java.util.*;

class Solution {
    public List<List<Integer>> permuteUnique(int[] nums) {
        Arrays.sort(nums);                      // group duplicates
        List<List<Integer>> result = new ArrayList<>();
        backtrack(nums, new boolean[nums.length], new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int[] nums, boolean[] used, List<Integer> path,
                           List<List<Integer>> result) {
        if (path.size() == nums.length) {
            result.add(new ArrayList<>(path));
            return;
        }
        for (int i = 0; i < nums.length; i++) {
            if (used[i]) continue;
            // skip a duplicate unless its identical predecessor is already in the path
            if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;
            used[i] = true;
            path.add(nums[i]);
            backtrack(nums, used, path, result);
            path.remove(path.size() - 1);
            used[i] = false;
        }
    }
}
```

**Complexity:** O(n · n!) worst case (all distinct); far fewer when duplicates collapse branches. O(n) extra space plus O(log n) for the sort.

**Key insight / follow-up:** The `!used[i-1]` condition is the crux. After `used[i-1]` is set false on backtrack, an equal `nums[i]` would start a sibling branch that duplicates work already done — so we skip it. (Using `used[i-1]` true as the *allow* condition equivalently enforces "duplicates are consumed in order.") Compare with LC90 where the parallel guard is `i > start`.

### LC39 — Combination Sum *(medium)*
**Problem:** Given an array of **distinct** integers `candidates` and a target integer `target`, return all unique combinations of candidates that sum to `target`. The **same number may be chosen unlimited times**. Two combinations are unique if the frequency of at least one number differs. `1 <= candidates[i]`. Example: `candidates = [2,3,6,7], target = 7` → `[[2,2,3],[7]]`.

**Approach:** Backtrack over candidates with a `start` index to avoid permuted duplicates of the same combination. Because reuse is unlimited, recurse with `i` (not `i + 1`) so the current candidate can be picked again. Subtract from a remaining `target`; record when it hits 0, prune when it goes negative.

```java
import java.util.*;

class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        List<List<Integer>> result = new ArrayList<>();
        Arrays.sort(candidates);                // enables the early-break prune
        backtrack(candidates, target, 0, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int[] candidates, int remaining, int start,
                           List<Integer> path, List<List<Integer>> result) {
        if (remaining == 0) {
            result.add(new ArrayList<>(path));  // exact hit
            return;
        }
        for (int i = start; i < candidates.length; i++) {
            if (candidates[i] > remaining) break; // sorted: no later one fits either
            path.add(candidates[i]);
            backtrack(candidates, remaining - candidates[i], i, path, result); // reuse i
            path.remove(path.size() - 1);
        }
    }
}
```

**Complexity:** O(n^(target/min)) in the worst case — the tree branches up to n ways and is at most `target/min(candidates)` deep; copying each result adds the path length. Space O(target/min) for the recursion depth.

**Key insight / follow-up:** Passing `i` instead of `i + 1` to the recursive call is the single change that permits unlimited reuse; the `start` index still blocks reordering, so `[2,3,2]` is never produced separately from `[2,2,3]`. Sorting plus `break` (rather than `continue`) prunes whole tails once a candidate overshoots.

### LC40 — Combination Sum II *(medium)*
**Problem:** Given a collection of candidate numbers `candidates` (**which may contain duplicates**) and a target, find all unique combinations that sum to `target`. **Each number may be used at most once** in a combination. The solution set must not contain duplicate combinations. Example: `candidates = [10,1,2,7,6,1,5], target = 8` → `[[1,1,6],[1,2,5],[1,7],[2,6]]`.

**Approach:** Sort to group duplicates, then combine two ideas: recurse with `i + 1` (each element used at most once) **and** apply the dedupe-on-sorted-input sibling-skip so two equal values don't start identical branches at the same level.

```java
import java.util.*;

class Solution {
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        Arrays.sort(candidates);                // group duplicates + enable break
        List<List<Integer>> result = new ArrayList<>();
        backtrack(candidates, target, 0, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int[] candidates, int remaining, int start,
                           List<Integer> path, List<List<Integer>> result) {
        if (remaining == 0) {
            result.add(new ArrayList<>(path));
            return;
        }
        for (int i = start; i < candidates.length; i++) {
            if (i > start && candidates[i] == candidates[i - 1]) continue; // dedupe siblings
            if (candidates[i] > remaining) break;                          // sorted prune
            path.add(candidates[i]);
            backtrack(candidates, remaining - candidates[i], i + 1, path, result); // no reuse
            path.remove(path.size() - 1);
        }
    }
}
```

**Complexity:** O(2ⁿ) in the worst case (each element in or out), times O(n) to copy each combination. Space O(n) for recursion depth plus O(log n) for the sort.

**Key insight / follow-up:** This problem is the meeting point of both tricks: `i + 1` for "use once" (like LC78) and `i > start` skip for "no duplicate combinations" (like LC90). Drop the skip and you'd get repeats such as two `[1,7]` (from the two distinct `1`s); use `i` instead of `i + 1` and you'd allow illegal reuse.

### LC79 — Word Search *(medium)*
**Problem:** Given an `m x n` grid of characters `board` and a string `word`, return `true` if `word` exists in the grid. The word is constructed from letters of **sequentially adjacent** cells (horizontally or vertically neighboring); **the same cell may not be used more than once**. Example: `board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"` → `true`.

**Approach:** DFS from every cell whose letter matches `word[0]`. At each step, mark the current cell visited (mutate it in place to a sentinel), recurse into the four neighbors for the next character, then restore the cell on the way back — the classic choose/un-choose, here applied to the grid itself.

```java
class Solution {
    public boolean exist(char[][] board, String word) {
        int rows = board.length, cols = board[0].length;
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (dfs(board, word, 0, r, c)) return true;
            }
        }
        return false;
    }

    private boolean dfs(char[][] board, String word, int idx, int r, int c) {
        if (idx == word.length()) return true;             // matched all chars
        if (r < 0 || c < 0 || r >= board.length || c >= board[0].length
                || board[r][c] != word.charAt(idx)) {
            return false;                                  // out of bounds or mismatch
        }
        char saved = board[r][c];
        board[r][c] = '#';                                 // choose: mark visited
        boolean found =
                dfs(board, word, idx + 1, r + 1, c) ||
                dfs(board, word, idx + 1, r - 1, c) ||
                dfs(board, word, idx + 1, r, c + 1) ||
                dfs(board, word, idx + 1, r, c - 1);
        board[r][c] = saved;                               // un-choose: restore
        return found;
    }
}
```

**Complexity:** O(m · n · 4^L) time, where L = `word.length()` — each of m·n starts explores up to 4 directions per character. O(L) extra space for the recursion stack (the in-place marking avoids a separate visited array).

**Key insight / follow-up:** Mutating `board[r][c]` to a sentinel and restoring it is an in-place `used` set — it prevents revisiting a cell within the current path while costing no extra memory. Short-circuit `||` stops at the first successful direction. Early follow-up: prune by checking the board has enough of each letter before searching.

### LC51 — N-Queens *(hard)*
**Problem:** Place `n` queens on an `n x n` chessboard so that no two queens attack each other (no two share a row, column, or diagonal). Return **all distinct solutions**, each as a list of strings where `'Q'` marks a queen and `'.'` an empty square. Example: `n = 4` → 2 solutions.

**Approach:** Place exactly one queen per row, recursing row by row. For each row try every column, checking against three sets of already-occupied lines: columns, the `↘` diagonals (constant `row - col`, offset to stay non-negative), and the `↙` anti-diagonals (constant `row + col`). Boolean arrays make each validity check O(1).

```java
import java.util.*;

class Solution {
    public List<List<String>> solveNQueens(int n) {
        List<List<String>> result = new ArrayList<>();
        boolean[] cols = new boolean[n];
        boolean[] diag = new boolean[2 * n - 1];     // row - col + (n-1)
        boolean[] anti = new boolean[2 * n - 1];     // row + col
        backtrack(0, n, new int[n], cols, diag, anti, result);
        return result;
    }

    private void backtrack(int row, int n, int[] queenCol, boolean[] cols,
                           boolean[] diag, boolean[] anti,
                           List<List<String>> result) {
        if (row == n) {
            result.add(build(queenCol, n));          // all rows filled
            return;
        }
        for (int col = 0; col < n; col++) {
            int d = row - col + n - 1, a = row + col;
            if (cols[col] || diag[d] || anti[a]) continue;   // under attack
            cols[col] = diag[d] = anti[a] = true;            // choose
            queenCol[row] = col;
            backtrack(row + 1, n, queenCol, cols, diag, anti, result);
            cols[col] = diag[d] = anti[a] = false;           // un-choose
        }
    }

    private List<String> build(int[] queenCol, int n) {
        List<String> board = new ArrayList<>();
        for (int r = 0; r < n; r++) {
            char[] line = new char[n];
            Arrays.fill(line, '.');
            line[queenCol[r]] = 'Q';
            board.add(new String(line));
        }
        return board;
    }
}
```

**Complexity:** O(n!) time — n choices in row 0, ~n−1 viable in row 1, and so on, with O(n²) to render each solution. O(n) extra space for the three marker arrays and the `queenCol` path.

**Key insight / follow-up:** Encoding diagonals as `row - col` (shifted by `n-1` to index from 0) and `row + col` turns the attack test into three O(1) array lookups — the key optimization over scanning placed queens. LC52 (N-Queens II) asks only for the *count*; drop the board-building and increment a counter at the base case.

### LC131 — Palindrome Partitioning *(medium)*
**Problem:** Given a string `s`, partition it such that **every substring of the partition is a palindrome**. Return all possible palindrome partitionings of `s`. Example: `s = "aab"` → `[["a","a","b"],["aa","b"]]`.

**Approach:** At each `start` index, try every prefix `s[start..end]`; if that prefix is a palindrome, add it to the path and recurse from `end + 1`. When `start` reaches the end of the string, the path is a complete valid partition. A two-pointer check tests each candidate prefix.

```java
import java.util.*;

class Solution {
    public List<List<String>> partition(String s) {
        List<List<String>> result = new ArrayList<>();
        backtrack(s, 0, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(String s, int start, List<String> path,
                           List<List<String>> result) {
        if (start == s.length()) {
            result.add(new ArrayList<>(path));      // whole string partitioned
            return;
        }
        for (int end = start; end < s.length(); end++) {
            if (!isPalindrome(s, start, end)) continue;   // prune non-palindromic prefix
            path.add(s.substring(start, end + 1));        // choose this prefix
            backtrack(s, end + 1, path, result);          // partition the rest
            path.remove(path.size() - 1);                 // un-choose
        }
    }

    private boolean isPalindrome(String s, int lo, int hi) {
        while (lo < hi) {
            if (s.charAt(lo++) != s.charAt(hi--)) return false;
        }
        return true;
    }
}
```

**Complexity:** O(n · 2ⁿ) time — up to 2^(n-1) ways to cut the string, each costing O(n) to validate and copy. O(n) extra space for the recursion depth and path.

**Key insight / follow-up:** The "choice" at each level is *where to make the next cut*; the palindrome test prunes invalid cuts before recursing. Precomputing an `isPalindrome[i][j]` DP table (O(n²) once) removes the repeated O(n) checks, dropping the per-node cost to O(1). LC132 (min cuts) is the optimization version, solved with DP rather than enumeration.

### LC22 — Generate Parentheses *(medium)*
**Problem:** Given `n` pairs of parentheses, generate all combinations of **well-formed** (valid) parentheses. Example: `n = 3` → `["((()))","(()())","(())()","()(())","()()()"]` (5 strings, the Catalan number C₃).

**Approach:** Build the string character by character, tracking how many `(` and `)` have been used. The two constraints that keep every prefix valid: you may add `(` while `open < n`, and you may add `)` only while `close < open` (never close more than you've opened). Record when the string reaches length `2n`.

```java
import java.util.*;

class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> result = new ArrayList<>();
        backtrack(new StringBuilder(), 0, 0, n, result);
        return result;
    }

    private void backtrack(StringBuilder sb, int open, int close, int n,
                           List<String> result) {
        if (sb.length() == 2 * n) {
            result.add(sb.toString());          // a complete valid string
            return;
        }
        if (open < n) {                         // can still open
            sb.append('(');
            backtrack(sb, open + 1, close, n, result);
            sb.deleteCharAt(sb.length() - 1);   // un-choose
        }
        if (close < open) {                     // can close only what's open
            sb.append(')');
            backtrack(sb, open, close + 1, n, result);
            sb.deleteCharAt(sb.length() - 1);   // un-choose
        }
    }
}
```

**Complexity:** O(4ⁿ / √n) time and space (excluding output) — bounded by the nth Catalan number, the count of valid strings; each is built in O(n). Recursion depth is O(n).

**Key insight / follow-up:** Enforcing `close < open` prunes the search to *only* valid prefixes, so unlike a naive "generate all 2^(2n) strings then filter" it never builds an invalid candidate. The `StringBuilder` with append/deleteCharAt is the choose/un-choose applied to a string buffer. The number of outputs is the Catalan number Cₙ.

---

*[← DSA bank index](README.md)*
