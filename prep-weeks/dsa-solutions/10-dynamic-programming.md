# DSA Solutions — Dynamic Programming

> Dynamic programming solves problems that decompose into **overlapping subproblems** with **optimal substructure**: the answer to a problem is built from answers to smaller instances of the same problem, and those smaller answers get reused. The whole job is four decisions. **(1) Define the state** — what does `dp[i]` (or `dp[i][j]`) *mean*? Be precise; a fuzzy state definition is why most DP attempts fail. **(2) Write the recurrence** — express `dp[i]` in terms of strictly smaller states. **(3) Pin the base case(s).** **(4) Choose an evaluation order** so every state is computed before it is needed. Try each problem yourself before reading the **Approach** and solution. Solutions are Java 17, compilable.

## Top-down vs bottom-up

**Top-down (memoization)** writes the recurrence as a recursive function and caches results in an array/map. It computes only the states actually reached, and the code reads exactly like the recurrence — easiest to get correct. Cost: recursion stack and (sometimes) hashing overhead.

**Bottom-up (tabulation)** fills a table in dependency order with explicit loops. No recursion stack, usually a small constant factor faster, and it exposes the structure you need for **space optimization**: if `dp[i]` only depends on `dp[i-1]` (and maybe `dp[i-2]`), you can drop the full array and keep one or two rolling variables — O(1) or O(n) space instead of O(n) or O(n·m). Many 2D DPs (`dp[i][j]` depending only on the previous row) collapse to a single 1D row the same way.

The two are equivalent in complexity; pick top-down to derive and verify a recurrence quickly, then convert to bottom-up when you want the constant-factor win or the space reduction. Throughout this file the solutions favor bottom-up with space optimization noted in the follow-ups.

---

### LC70 — Climbing Stairs *(easy)*
**Problem:** You are climbing a staircase that takes `n` steps to reach the top. Each time you can climb either **1 or 2** steps. Return the number of distinct ways to reach the top. `1 <= n <= 45`.
**Approach:** Let `dp[i]` = number of ways to reach step `i`. The last move was either a 1-step (from `i-1`) or a 2-step (from `i-2`), so `dp[i] = dp[i-1] + dp[i-2]` — the Fibonacci recurrence. Base: `dp[0] = dp[1] = 1`. Only the last two values matter, so keep two rolling variables.
```java
class Solution {
    public int climbStairs(int n) {
        int prev2 = 1, prev1 = 1;           // ways to reach step 0 and step 1
        for (int i = 2; i <= n; i++) {
            int cur = prev1 + prev2;
            prev2 = prev1;
            prev1 = cur;
        }
        return prev1;
    }
}
```
**Complexity:** O(n) time, O(1) space.
**Key insight / follow-up:** This is Fibonacci in disguise — recognizing that "the answer depends only on the previous two" is the canonical first DP. If the allowed step sizes were `{1, 2, ..., k}`, the recurrence becomes a sum over the last `k` states (a sliding window). For astronomically large `n`, matrix exponentiation gives O(log n).

### LC198 — House Robber *(medium)*
**Problem:** You are a robber on a street of houses; `nums[i]` is the money in house `i`. You cannot rob **two adjacent** houses (the alarm triggers). Return the maximum money you can rob.
**Approach:** Let `dp[i]` = max money robbing among the first `i+1` houses. At house `i` you either skip it (`dp[i-1]`) or rob it and add `nums[i]` to the best up to `i-2` (`dp[i-2] + nums[i]`). So `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`. Two rolling variables suffice.
```java
class Solution {
    public int rob(int[] nums) {
        int prev2 = 0, prev1 = 0;           // best up to i-2 and i-1
        for (int x : nums) {
            int cur = Math.max(prev1, prev2 + x);
            prev2 = prev1;
            prev1 = cur;
        }
        return prev1;
    }
}
```
**Complexity:** O(n) time, O(1) space.
**Key insight / follow-up:** The "rob or skip" choice is the prototypical pick/don't-pick DP. Starting both rolling vars at 0 cleanly handles the empty and single-house cases. The circular variant (LC213) reuses this exact routine twice.

### LC213 — House Robber II *(medium)*
**Problem:** Same as LC198, but the houses are arranged in a **circle** — the first and last houses are adjacent, so you cannot rob both. Return the maximum money.
**Approach:** The circle adds one constraint: house `0` and house `n-1` can't both be robbed. Split into two linear subproblems — rob houses `[0 .. n-2]` (exclude the last) or `[1 .. n-1]` (exclude the first) — run the LC198 helper on each, and take the max. The single-house case is handled separately.
```java
class Solution {
    public int rob(int[] nums) {
        int n = nums.length;
        if (n == 1) return nums[0];
        return Math.max(robLine(nums, 0, n - 2), robLine(nums, 1, n - 1));
    }

    private int robLine(int[] nums, int lo, int hi) {
        int prev2 = 0, prev1 = 0;
        for (int i = lo; i <= hi; i++) {
            int cur = Math.max(prev1, prev2 + nums[i]);
            prev2 = prev1;
            prev1 = cur;
        }
        return prev1;
    }
}
```
**Complexity:** O(n) time, O(1) space.
**Key insight / follow-up:** The trick is reducing a circular constraint to two linear runs by fixing the mutually-exclusive endpoints. This "break the cycle by casing on a boundary element" pattern recurs in circular-array problems (e.g. maximum circular subarray sum).

### LC139 — Word Break *(medium)*
**Problem:** Given a string `s` and a dictionary `wordDict` of words, return `true` if `s` can be segmented into a space-separated sequence of one or more dictionary words. Words may be reused. `1 <= s.length <= 300`.
**Approach:** Let `dp[i]` = `true` if the prefix `s[0..i)` (length `i`) can be segmented. `dp[i]` is true if there is a split point `j < i` where `dp[j]` is true **and** `s[j..i)` is a dictionary word. Base: `dp[0] = true` (empty prefix). Use a `HashSet` for O(1) word lookups.
```java
import java.util.*;

class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        Set<String> words = new HashSet<>(wordDict);
        int n = s.length();
        boolean[] dp = new boolean[n + 1];
        dp[0] = true;                                   // empty prefix is segmentable
        for (int i = 1; i <= n; i++) {
            for (int j = 0; j < i; j++) {
                if (dp[j] && words.contains(s.substring(j, i))) {
                    dp[i] = true;
                    break;                              // one valid split is enough
                }
            }
        }
        return dp[n];
    }
}
```
**Complexity:** O(n²) substrings × O(n) substring/hash cost ⇒ O(n³) worst case time, O(n) space (plus the dictionary set).
**Key insight / follow-up:** State = "is this prefix reachable"; the second loop tries every last-word boundary. Capping the inner loop by the longest dictionary word, or using a Trie, prunes the work. To return the actual segmentation (LC140) you store back-pointers or recurse with memoization, watching out for exponential blow-up on inputs like `"aaaa...b"`.

### LC300 — Longest Increasing Subsequence *(medium)*
**Problem:** Given an integer array `nums`, return the length of the longest **strictly increasing subsequence** (elements need not be contiguous, but order is preserved). `1 <= nums.length <= 2500`.
**Approach:** Two solutions. **(A) O(n²) DP:** `dp[i]` = length of the longest increasing subsequence **ending at** index `i`; `dp[i] = 1 + max(dp[j])` over all `j < i` with `nums[j] < nums[i]`. **(B) O(n log n) patience sorting:** maintain `tails`, where `tails[k]` is the smallest possible tail of an increasing subsequence of length `k+1`. For each value, binary-search the first tail `>= value` and replace it (or append if larger than all). The length of `tails` is the answer.
```java
import java.util.*;

class Solution {
    // O(n log n) — patience sorting
    public int lengthOfLIS(int[] nums) {
        List<Integer> tails = new ArrayList<>();
        for (int x : nums) {
            int pos = lowerBound(tails, x);             // first tail >= x
            if (pos == tails.size()) tails.add(x);      // x extends the longest run
            else tails.set(pos, x);                     // tighten an existing length
        }
        return tails.size();
    }

    private int lowerBound(List<Integer> a, int target) {
        int lo = 0, hi = a.size();
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (a.get(mid) < target) lo = mid + 1;
            else hi = mid;
        }
        return lo;
    }

    // O(n^2) DP alternative
    public int lengthOfLIS_dp(int[] nums) {
        int n = nums.length, best = 1;
        int[] dp = new int[n];
        Arrays.fill(dp, 1);
        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[j] < nums[i]) dp[i] = Math.max(dp[i], dp[j] + 1);
            }
            best = Math.max(best, dp[i]);
        }
        return best;
    }
}
```
**Complexity:** O(n log n) time / O(n) space for the patience method; O(n²) time / O(n) space for the DP.
**Key insight / follow-up:** `tails` is **not** an actual subsequence — it only tracks the best tail per length, which is what makes the binary search valid (`tails` is always sorted). Use `lowerBound` (first element `>= x`) for *strictly* increasing; switch to `upperBound` (first `> x`) to allow equal elements (non-strict / longest non-decreasing). To reconstruct the actual LIS, record predecessor indices alongside the DP version.

### LC322 — Coin Change *(medium)*
**Problem:** Given coin denominations `coins` (unlimited supply of each) and a target `amount`, return the **fewest** number of coins that sum to `amount`, or `-1` if it cannot be made. `0 <= amount <= 10^4`.
**Approach:** Unbounded knapsack on minimization. `dp[a]` = fewest coins to make amount `a`. For each amount `a`, try every coin `c <= a`: `dp[a] = min(dp[a], dp[a - c] + 1)`. Initialize `dp` to a sentinel "infinity" (`amount + 1` is safely larger than any real answer), with `dp[0] = 0`.
```java
import java.util.*;

class Solution {
    public int coinChange(int[] coins, int amount) {
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, amount + 1);                    // sentinel ≈ infinity
        dp[0] = 0;
        for (int a = 1; a <= amount; a++) {
            for (int c : coins) {
                if (c <= a) dp[a] = Math.min(dp[a], dp[a - c] + 1);
            }
        }
        return dp[amount] > amount ? -1 : dp[amount];
    }
}
```
**Complexity:** O(amount × #coins) time, O(amount) space.
**Key insight / follow-up:** Because each coin has unlimited supply, the amount loop runs **outer** and coins inner, and we read `dp[a - c]` from the *current* row — that's what permits reusing a coin. Greedy (always take the largest coin) fails for denominations like `{1, 3, 4}` paying 6 (greedy gives 4+1+1=3 coins; optimal is 3+3=2), which is exactly why DP is required. LC518 counts the *number of ways* instead of the minimum.

### LC62 — Unique Paths *(medium)*
**Problem:** A robot sits at the top-left of an `m x n` grid and wants to reach the bottom-right. It can only move **right** or **down**. Return the number of distinct paths. `1 <= m, n <= 100`.
**Approach:** `dp[i][j]` = number of paths to cell `(i, j)`. You arrive from the left or from above: `dp[i][j] = dp[i-1][j] + dp[i][j-1]`. The first row and first column have exactly one path each. Since each row only needs the row above, collapse to a single 1D array updated in place left-to-right.
```java
class Solution {
    public int uniquePaths(int m, int n) {
        int[] row = new int[n];
        java.util.Arrays.fill(row, 1);                  // top row: one path to each cell
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                row[j] += row[j - 1];                   // above (old row[j]) + left (new row[j-1])
            }
        }
        return row[n - 1];
    }
}
```
**Complexity:** O(m·n) time, O(n) space (1D rolling row).
**Key insight / follow-up:** The in-place `row[j] += row[j-1]` works because, at the moment of the update, `row[j]` still holds the value from the row above and `row[j-1]` already holds the new (left) value. There is also a pure combinatorics answer: choose which `m-1` of the `m+n-2` moves are "down", i.e. `C(m+n-2, m-1)`.

### LC63 — Unique Paths II *(medium)*
**Problem:** Same grid and movement as LC62, but some cells contain obstacles, marked `1` in `obstacleGrid` (free cells are `0`). A path cannot pass through an obstacle. Return the number of distinct paths from top-left to bottom-right.
**Approach:** Same recurrence as LC62, with one rule: an obstacle cell has `0` paths through it. Use a 1D rolling row; when `grid[i][j] == 1`, set `row[j] = 0`, otherwise add the left neighbor. Initialize the start: if the top-left cell is an obstacle, there are zero paths.
```java
class Solution {
    public int uniquePathsWithObstacles(int[][] grid) {
        int n = grid[0].length;
        int[] row = new int[n];
        row[0] = grid[0][0] == 1 ? 0 : 1;               // seed the start cell
        for (int j = 1; j < n; j++)                      // first row
            row[j] = grid[0][j] == 1 ? 0 : row[j - 1];
        for (int i = 1; i < grid.length; i++) {
            if (grid[i][0] == 1) row[0] = 0;            // blocked first column propagates
            for (int j = 1; j < n; j++) {
                row[j] = grid[i][j] == 1 ? 0 : row[j] + row[j - 1];
            }
        }
        return row[n - 1];
    }
}
```
**Complexity:** O(m·n) time, O(n) space.
**Key insight / follow-up:** The only change from LC62 is forcing obstacle cells to contribute zero, which automatically zeroes out every downstream path that would have used them. Watch the first row/column: once an obstacle appears there, every later cell in that line is unreachable (its single feeder is `0`).

### LC64 — Minimum Path Sum *(medium)*
**Problem:** Given an `m x n` grid of **non-negative** integers, find a path from top-left to bottom-right that minimizes the sum of numbers along the path. You may only move **right** or **down**. Return that minimum sum.
**Approach:** `dp[i][j]` = minimum cost to reach `(i, j)`. You arrive from the cheaper of above or left, then pay the current cell: `dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])`. First row/column accumulate along their single direction. Collapse to a 1D rolling row.
```java
class Solution {
    public int minPathSum(int[][] grid) {
        int n = grid[0].length;
        int[] row = new int[n];
        row[0] = grid[0][0];
        for (int j = 1; j < n; j++) row[j] = row[j - 1] + grid[0][j];   // first row
        for (int i = 1; i < grid.length; i++) {
            row[0] += grid[i][0];                       // first column accumulates downward
            for (int j = 1; j < n; j++) {
                row[j] = grid[i][j] + Math.min(row[j], row[j - 1]); // min(above, left)
            }
        }
        return row[n - 1];
    }
}
```
**Complexity:** O(m·n) time, O(n) space.
**Key insight / follow-up:** This is LC62 with `+` replaced by `min` and a per-cell cost added — the same grid-DP skeleton (`above` op `left`) reused for an optimization instead of a count. As in Unique Paths, the rolling-row update relies on `row[j]` (above) and `row[j-1]` (left) both being valid at the moment of the update. If diagonal moves were allowed you'd add a third predecessor term.

### LC72 — Edit Distance *(hard)*
**Problem:** Given two strings `word1` and `word2`, return the minimum number of operations to convert `word1` into `word2`. Permitted operations: **insert** a character, **delete** a character, or **replace** a character. `0 <= length <= 500`.
**Approach:** `dp[i][j]` = edit distance between the first `i` characters of `word1` and the first `j` of `word2`. If the current characters match, no cost: `dp[i][j] = dp[i-1][j-1]`. Otherwise take `1 + min` of the three edits: replace (`dp[i-1][j-1]`), delete from `word1` (`dp[i-1][j]`), insert into `word1` (`dp[i][j-1]`). Base: converting to/from the empty string costs `i` (deletes) or `j` (inserts). Collapse to two rolling rows.
```java
class Solution {
    public int minDistance(String word1, String word2) {
        int m = word1.length(), n = word2.length();
        int[] prev = new int[n + 1];
        for (int j = 0; j <= n; j++) prev[j] = j;       // word1="" → j inserts
        for (int i = 1; i <= m; i++) {
            int[] cur = new int[n + 1];
            cur[0] = i;                                 // word2="" → i deletes
            for (int j = 1; j <= n; j++) {
                if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                    cur[j] = prev[j - 1];               // chars match, no op
                } else {
                    cur[j] = 1 + Math.min(prev[j - 1],  // replace
                                 Math.min(prev[j],      // delete from word1
                                          cur[j - 1])); // insert into word1
                }
            }
            prev = cur;
        }
        return prev[n];
    }
}
```
**Complexity:** O(m·n) time, O(n) space (two rolling rows).
**Key insight / follow-up:** The three transitions map exactly to the three operations — diagonal = replace, up = delete, left = insert. The matched-character case carries the diagonal straight through at zero cost, which is what aligns common substrings. This is the Levenshtein distance; restricting to insert+delete only (no replace) reduces it to "shortest path via LCS" and changes the recurrence to drop the diagonal-with-replace term.

### LC1143 — Longest Common Subsequence *(medium)*
**Problem:** Given two strings `text1` and `text2`, return the length of their longest **common subsequence** (a subsequence keeps relative order but need not be contiguous), or `0` if there is none. `1 <= length <= 1000`.
**Approach:** `dp[i][j]` = LCS length of the first `i` chars of `text1` and first `j` of `text2`. If the current characters match, extend the diagonal: `dp[i][j] = dp[i-1][j-1] + 1`. Otherwise drop one character from either string and keep the better: `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`. Base: any LCS with an empty prefix is 0. Two rolling rows suffice.
```java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int m = text1.length(), n = text2.length();
        int[] prev = new int[n + 1];                    // all zeros: empty prefix
        for (int i = 1; i <= m; i++) {
            int[] cur = new int[n + 1];
            for (int j = 1; j <= n; j++) {
                if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                    cur[j] = prev[j - 1] + 1;           // match extends the diagonal
                } else {
                    cur[j] = Math.max(prev[j], cur[j - 1]); // skip one char
                }
            }
            prev = cur;
        }
        return prev[n];
    }
}
```
**Complexity:** O(m·n) time, O(n) space (two rolling rows).
**Key insight / follow-up:** LCS is the structural twin of edit distance — match → diagonal+1, mismatch → max of the two skips. The **longest common substring** (contiguous) is a different DP: reset to 0 on a mismatch and track the running maximum. LCS also solves shortest-common-supersequence and "minimum deletions to make two strings equal" (`m + n − 2·LCS`).

### LC221 — Maximal Square *(medium)*
**Problem:** Given an `m x n` binary matrix filled with `0`s and `1`s, find the largest **square** containing only `1`s and return its **area**.
**Approach:** `dp[i][j]` = side length of the largest all-ones square whose **bottom-right corner** is `(i, j)`. If `matrix[i][j] == 1`, that corner can extend a square only as far as its three neighbors allow: `dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])`. If the cell is `0`, `dp[i][j] = 0`. Track the maximum side; area is side². Collapse to a 1D row carrying the diagonal in a temp variable.
```java
class Solution {
    public int maximalSquare(char[][] matrix) {
        int m = matrix.length, n = matrix[0].length, best = 0;
        int[] dp = new int[n + 1];                      // 1-indexed cols; dp[0] padding
        for (int i = 1; i <= m; i++) {
            int diag = 0;                               // dp[i-1][j-1] before overwrite
            for (int j = 1; j <= n; j++) {
                int temp = dp[j];                       // save current dp[j] (= dp[i-1][j])
                if (matrix[i - 1][j - 1] == '1') {
                    dp[j] = 1 + Math.min(diag, Math.min(dp[j], dp[j - 1]));
                    best = Math.max(best, dp[j]);
                } else {
                    dp[j] = 0;
                }
                diag = temp;                            // next iteration's diagonal
            }
        }
        return best * best;
    }
}
```
**Complexity:** O(m·n) time, O(n) space.
**Key insight / follow-up:** The `min` of three neighbors is the heart of it — a square corner is limited by its **weakest** adjacent square, so the bottleneck dictates the side. The `diag`/`temp` shuffle is the standard idiom for preserving the diagonal `dp[i-1][j-1]` while rolling a 1D array. For the largest *rectangle* of ones (LC85), the square recurrence no longer applies — you fall back to a per-row largest-rectangle-in-histogram pass.

### LC416 — Partition Equal Subset Sum *(medium)*
**Problem:** Given a non-empty array `nums` of positive integers, determine whether it can be partitioned into **two subsets with equal sum**. `1 <= nums.length <= 200`, `1 <= nums[i] <= 100`.
**Approach:** Two equal halves mean each subset sums to `total / 2`; if `total` is odd, it's immediately impossible. The question reduces to a **0/1 subset-sum**: can some subset reach `target = total / 2`? `dp[s]` = `true` if sum `s` is achievable. For each number, update `dp` **right-to-left** (`s` from `target` down to `num`) so each number is used at most once.
```java
class Solution {
    public boolean canPartition(int[] nums) {
        int total = 0;
        for (int x : nums) total += x;
        if ((total & 1) == 1) return false;             // odd total can't split evenly
        int target = total / 2;
        boolean[] dp = new boolean[target + 1];
        dp[0] = true;                                   // sum 0 always achievable
        for (int x : nums) {
            for (int s = target; s >= x; s--) {         // reverse → 0/1 (use x once)
                if (dp[s - x]) dp[s] = true;
            }
            if (dp[target]) return true;                // early exit
        }
        return dp[target];
    }
}
```
**Complexity:** O(n · target) time, O(target) space.
**Key insight / follow-up:** The **reverse** inner loop is the defining feature of 0/1 knapsack with a 1D array: iterating downward guarantees `dp[s - x]` still reflects the state *before* `x` was considered, so each item is counted once. (Iterating forward, as in unbounded knapsack / Coin Change II, would let an item be reused.) This is subset-sum; the optimization variant (LC494 Target Sum) counts sign assignments via the same transform.

### LC518 — Coin Change II *(medium)*
**Problem:** Given an `amount` and coin denominations `coins` (unlimited supply of each), return the **number of distinct combinations** that sum to `amount`. Order does not matter, so `1+2` and `2+1` count once. `0 <= amount <= 5000`.
**Approach:** Counting variant of unbounded knapsack. `dp[a]` = number of ways to make amount `a`. Loop **coins on the outside, amount on the inside** — this fixes the order in which coins are considered, so each combination is counted exactly once (combinations, not permutations). `dp[a] += dp[a - coin]`. Base: `dp[0] = 1` (one way to make 0: take nothing).
```java
class Solution {
    public int change(int amount, int[] coins) {
        int[] dp = new int[amount + 1];
        dp[0] = 1;                                      // empty selection makes amount 0
        for (int coin : coins) {                        // coin outer → combinations
            for (int a = coin; a <= amount; a++) {      // forward → reuse coin (unbounded)
                dp[a] += dp[a - coin];
            }
        }
        return dp[amount];
    }
}
```
**Complexity:** O(amount × #coins) time, O(amount) space.
**Key insight / follow-up:** Loop order is the whole problem. **Coins outer** counts combinations (this problem); **amount outer** (as in LC322) would count *permutations* — i.e. ordered sequences, which is what LC377 Combination Sum IV actually wants. The **forward** inner loop allows reusing a coin (unbounded); compare LC416's reverse loop for 0/1 (use-once). Memorize this 2×2: {coin vs amount outer} × {forward vs reverse}.

### LC97 — Interleaving String *(medium)*
**Problem:** Given strings `s1`, `s2`, and `s3`, return `true` if `s3` is formed by an **interleaving** of `s1` and `s2` — i.e. `s3` can be split into pieces that, read in order, alternate from `s1` and `s2` while preserving each string's internal order. A necessary condition is `s3.length == s1.length + s2.length`.
**Approach:** `dp[i][j]` = `true` if `s3`'s first `i+j` characters are an interleaving of `s1`'s first `i` and `s2`'s first `j`. The `(i+j)`-th char of `s3` came from either `s1[i-1]` (needs `dp[i-1][j]` and a match) or `s2[j-1]` (needs `dp[i][j-1]` and a match). Base `dp[0][0] = true`. Collapse to a 1D row over `j`.
```java
class Solution {
    public boolean isInterleave(String s1, String s2, String s3) {
        int m = s1.length(), n = s2.length();
        if (m + n != s3.length()) return false;         // length mismatch → impossible
        boolean[] dp = new boolean[n + 1];
        for (int i = 0; i <= m; i++) {
            for (int j = 0; j <= n; j++) {
                if (i == 0 && j == 0) {
                    dp[j] = true;
                } else if (i == 0) {                    // only s2 contributes
                    dp[j] = dp[j - 1] && s2.charAt(j - 1) == s3.charAt(j - 1);
                } else if (j == 0) {                    // only s1 contributes
                    dp[j] = dp[j] && s1.charAt(i - 1) == s3.charAt(i - 1);
                } else {
                    boolean fromS1 = dp[j] && s1.charAt(i - 1) == s3.charAt(i + j - 1);
                    boolean fromS2 = dp[j - 1] && s2.charAt(j - 1) == s3.charAt(i + j - 1);
                    dp[j] = fromS1 || fromS2;
                }
            }
        }
        return dp[n];
    }
}
```
**Complexity:** O(m·n) time, O(n) space.
**Key insight / follow-up:** In the 1D rolling form, `dp[j]` (before overwrite) is the *old* row = `dp[i-1][j]` (the "took a char from s1" branch), and `dp[j-1]` is the *current* row = `dp[i][j-1]` (the "took from s2" branch). The index into `s3` is `i + j - 1` because that many characters have been placed. The early length check is essential — without it the `s3.charAt` calls can read past the end. A naive recursive interleave check without memoization is exponential.

---

*[← DSA bank index](README.md)*
