# DSA Solutions — Binary Search

> Binary search is the canonical "halve the search space each step" technique — O(log n). It shows up in two flavors: searching a sorted (or rotated) array directly, and **searching on the answer** when the answer lives in a numeric range and a `feasible(mid)` test is monotonic. Try each problem yourself before reading the **Approach** and solution. Solutions are Java 17, compilable, with overflow-safe midpoints.

## Two templates

**1. Standard binary search (`left <= right`)** — used when you search an index in a concrete array and the loop should terminate when the range is empty.

```java
int lo = 0, hi = n - 1;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;     // overflow-safe; never (lo + hi) / 2
    if (arr[mid] == target) return mid;
    else if (arr[mid] < target) lo = mid + 1;
    else hi = mid - 1;
}
return -1;
```

Always compute `mid` as `lo + (hi - lo) / 2`. The naive `(lo + hi) / 2` overflows when `lo + hi > Integer.MAX_VALUE`.

**2. Binary search on the answer (`feasible(mid)` over a monotonic range)** — used when the answer is a value in `[lo, hi]` and there is a predicate `feasible(x)` that is **false, false, ..., false, true, true, ..., true** (or the mirror image). You binary-search for the boundary. The array is never indexed; you evaluate a feasibility function.

```java
int lo = minPossibleAnswer, hi = maxPossibleAnswer;
while (lo < hi) {                     // converge to a single value
    int mid = lo + (hi - lo) / 2;
    if (feasible(mid)) hi = mid;      // mid works → try smaller (search left half, keep mid)
    else lo = mid + 1;               // mid fails → must go bigger
}
return lo;                            // lo == hi == smallest feasible answer
```

The key skill is recognizing monotonicity: "if capacity X works, every capacity > X also works." Then `feasible` partitions the range and you find the boundary.

---

### LC704 — Binary Search *(easy)*
**Problem:** Given a sorted (ascending) array of distinct integers `nums` and an integer `target`, return the index of `target` if it is in `nums`, otherwise return `-1`. You must write an algorithm with O(log n) runtime.
**Approach:** Textbook standard template. Maintain `[lo, hi]` inclusive bounds; compare the midpoint and discard the half that cannot contain the target.
```java
class Solution {
    public int search(int[] nums, int target) {
        int lo = 0, hi = nums.length - 1;
        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;
            if (nums[mid] == target) return mid;
            else if (nums[mid] < target) lo = mid + 1;
            else hi = mid - 1;
        }
        return -1;
    }
}
```
**Complexity:** O(log n) time, O(1) space.
**Key insight / follow-up:** The `lo <= hi` condition is what lets the search inspect the final single-element range. With `lo < hi` you must restructure the loop or you miss matches. `lo + (hi - lo) / 2` is the overflow-safe midpoint.

### LC33 — Search in Rotated Sorted Array *(medium)*
**Problem:** An ascending sorted array of **distinct** integers is rotated at an unknown pivot (e.g. `[0,1,2,4,5,6,7]` → `[4,5,6,7,0,1,2]`). Given the rotated array `nums` and a `target`, return its index or `-1`, in O(log n).
**Approach:** At any `mid`, at least one of the two halves `[lo..mid]` and `[mid..hi]` is sorted. Identify the sorted half, check whether the target falls inside its range, and move accordingly.
```java
class Solution {
    public int search(int[] nums, int target) {
        int lo = 0, hi = nums.length - 1;
        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;
            if (nums[mid] == target) return mid;
            if (nums[lo] <= nums[mid]) {                 // left half sorted
                if (nums[lo] <= target && target < nums[mid]) hi = mid - 1;
                else lo = mid + 1;
            } else {                                     // right half sorted
                if (nums[mid] < target && target <= nums[hi]) lo = mid + 1;
                else hi = mid - 1;
            }
        }
        return -1;
    }
}
```
**Complexity:** O(log n) time, O(1) space.
**Key insight / follow-up:** Use `nums[lo] <= nums[mid]` (not `<`) so a two-element window where `lo == mid` is treated as a sorted left half. The trick is that rotation leaves one half fully sorted, which restores the binary-search invariant.

### LC81 — Search in Rotated Sorted Array II *(medium)*
**Problem:** Same as LC33, but the array may contain **duplicates**. Return `true` if `target` exists, else `false`.
**Approach:** Duplicates break the "one half is sorted" test when `nums[lo] == nums[mid] == nums[hi]` — you can't tell which side is sorted. Handle that ambiguity by shrinking both ends by one (`lo++; hi--`), then fall back to the LC33 logic.
```java
class Solution {
    public boolean search(int[] nums, int target) {
        int lo = 0, hi = nums.length - 1;
        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;
            if (nums[mid] == target) return true;
            if (nums[lo] == nums[mid] && nums[mid] == nums[hi]) {
                lo++; hi--;                              // can't decide; trim ambiguity
            } else if (nums[lo] <= nums[mid]) {          // left half sorted
                if (nums[lo] <= target && target < nums[mid]) hi = mid - 1;
                else lo = mid + 1;
            } else {                                     // right half sorted
                if (nums[mid] < target && target <= nums[hi]) lo = mid + 1;
                else hi = mid - 1;
            }
        }
        return false;
    }
}
```
**Complexity:** O(log n) average, **O(n) worst case** (e.g. all equal values force the `lo++; hi--` branch repeatedly), O(1) space.
**Key insight / follow-up:** The duplicate case is why this variant cannot guarantee O(log n). The `lo++; hi--` trim removes only the ambiguity; correctness is preserved because we already checked `nums[mid] != target`.

### LC153 — Find Minimum in Rotated Sorted Array *(medium)*
**Problem:** A sorted ascending array of **distinct** integers is rotated at an unknown pivot. Return the minimum element, in O(log n).
**Approach:** The minimum is the single "inflection" point. Compare `nums[mid]` to `nums[hi]`: if `nums[mid] > nums[hi]`, the minimum is strictly to the right of `mid`; otherwise it is at `mid` or to its left. Converge with `lo < hi`.
```java
class Solution {
    public int findMin(int[] nums) {
        int lo = 0, hi = nums.length - 1;
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (nums[mid] > nums[hi]) lo = mid + 1;     // min is in (mid, hi]
            else hi = mid;                              // min is in [lo, mid]
        }
        return nums[lo];                                // lo == hi at the minimum
    }
}
```
**Complexity:** O(log n) time, O(1) space.
**Key insight / follow-up:** Compare against `nums[hi]`, **not** `nums[lo]` — comparing to `lo` is ambiguous on an already-sorted (un-rotated) array. The `lo < hi` loop with `hi = mid` (keeping `mid`) is the standard "find the boundary" pattern and never overruns. With duplicates (LC154), add the `else if (nums[mid] == nums[hi]) hi--;` fallback.

### LC74 — Search a 2D Matrix *(medium)*
**Problem:** An `m x n` matrix where each row is sorted ascending and the first integer of each row is greater than the last integer of the previous row. Given a `target`, return `true` if it is present, in O(log(m·n)).
**Approach:** The constraints make the matrix a single sorted sequence read row-major. Binary-search the virtual index `[0, m*n - 1]`, mapping `idx` to `(idx / n, idx % n)`.
```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int m = matrix.length, n = matrix[0].length;
        int lo = 0, hi = m * n - 1;
        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;
            int val = matrix[mid / n][mid % n];
            if (val == target) return true;
            else if (val < target) lo = mid + 1;
            else hi = mid - 1;
        }
        return false;
    }
}
```
**Complexity:** O(log(m·n)) time, O(1) space.
**Key insight / follow-up:** The index mapping `row = mid / n`, `col = mid % n` flattens the 2D grid into one sorted array without copying. If only each row and each column were sorted (LC240, weaker guarantee), you'd instead walk from the top-right corner in O(m + n).

### LC162 — Find Peak Element *(medium)*
**Problem:** A peak element is strictly greater than its neighbors. Given `nums` where `nums[i] != nums[i+1]` and conceptual `nums[-1] = nums[n] = -∞`, return the index of **any** peak, in O(log n).
**Approach:** Binary search on the slope. If `nums[mid] < nums[mid + 1]`, an ascending slope guarantees a peak to the right; otherwise a peak is at `mid` or to its left. Converge with `lo < hi`.
```java
class Solution {
    public int findPeakElement(int[] nums) {
        int lo = 0, hi = nums.length - 1;
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (nums[mid] < nums[mid + 1]) lo = mid + 1; // climbing → peak on the right
            else hi = mid;                               // descending → peak at mid or left
        }
        return lo;
    }
}
```
**Complexity:** O(log n) time, O(1) space.
**Key insight / follow-up:** This works even though the array isn't sorted — the `-∞` boundaries guarantee that following the uphill direction always leads to a peak. `mid + 1` is always in bounds because `lo < hi` means `mid < hi`. This is a great example of binary search on a monotonic *property* (slope direction) rather than on values.

### LC875 — Koko Eating Bananas *(medium)*
**Problem:** There are `n` piles of bananas, `piles[i]` in pile `i`. Koko eats at speed `k` bananas/hour: each hour she picks one pile and eats `k` from it (if the pile has fewer than `k`, she finishes it and stops for that hour). Given `h` hours (with `h >= n`), return the **minimum** integer speed `k` so she finishes all bananas within `h` hours.
**Approach:** Binary search on the answer `k` in `[1, max(piles)]`. Feasibility is monotonic: a higher speed never needs more hours. Hours at speed `k` is `Σ ceil(pile / k)`. Find the smallest `k` whose total hours `<= h`.
```java
class Solution {
    public int minEatingSpeed(int[] piles, int h) {
        int lo = 1, hi = 0;
        for (int p : piles) hi = Math.max(hi, p);
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (hoursNeeded(piles, mid) <= h) hi = mid;  // feasible → try slower
            else lo = mid + 1;                           // too slow → speed up
        }
        return lo;
    }

    private long hoursNeeded(int[] piles, int speed) {
        long hours = 0;
        for (int p : piles) hours += (p + speed - 1) / speed;  // ceil(p / speed)
        return hours;
    }
}
```
**Complexity:** O(n · log(max pile)) time, O(1) space.
**Key insight / follow-up:** `(p + speed - 1) / speed` is integer ceiling division without floating point. Accumulate hours in a `long` — with up to 10^4 piles of 10^9 bananas at speed 1, the sum overflows `int`. The monotonic predicate ("speed k feasible ⇒ all faster speeds feasible") is what makes binary-search-on-answer valid.

### LC1011 — Capacity to Ship Packages Within D Days *(medium)*
**Problem:** Packages on a conveyor must ship within `days` days, in the **given order**. Each day you load the ship with packages (in order) without exceeding its weight capacity. Return the **least** ship capacity that delivers all packages within `days` days.
**Approach:** Binary search on capacity in `[max(weights), sum(weights)]`. The lower bound must hold the heaviest single package; the upper bound ships everything in one day. `feasible(cap)` greedily packs in order, counting days; monotonic because a bigger ship never needs more days.
```java
class Solution {
    public int shipWithinDays(int[] weights, int days) {
        int lo = 0, hi = 0;
        for (int w : weights) { lo = Math.max(lo, w); hi += w; }
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (daysNeeded(weights, mid) <= days) hi = mid; // fits → try smaller ship
            else lo = mid + 1;                              // too small → enlarge
        }
        return lo;
    }

    private int daysNeeded(int[] weights, int cap) {
        int days = 1, load = 0;
        for (int w : weights) {
            if (load + w > cap) { days++; load = 0; }       // start a new day
            load += w;
        }
        return days;
    }
}
```
**Complexity:** O(n · log(sum − max)) time, O(1) space.
**Key insight / follow-up:** The search range is bounded by `max(weights)` (a day must carry the biggest item) and `sum(weights)` (everything in one day). This is structurally identical to LC410 (split-array) and LC875 (Koko) — recognize the family: "minimize the maximum chunk subject to a fixed number of partitions."

### LC410 — Split Array Largest Sum *(hard)*
**Problem:** Given an array `nums` of non-negative integers and an integer `k`, split `nums` into `k` non-empty **contiguous** subarrays so that the **largest** subarray sum is minimized. Return that minimized largest sum.
**Approach:** Binary search on the answer (the largest allowed subarray sum) in `[max(nums), sum(nums)]`. `feasible(limit)` greedily forms subarrays, starting a new one whenever adding the next element would exceed `limit`, and checks the count `<= k`. Monotonic: a larger limit needs fewer or equal pieces.
```java
class Solution {
    public int splitArray(int[] nums, int k) {
        int lo = 0; long hi = 0;
        for (int x : nums) { lo = Math.max(lo, x); hi += x; }
        long left = lo, right = hi;
        while (left < right) {
            long mid = left + (right - left) / 2;
            if (piecesNeeded(nums, mid) <= k) right = mid;  // fits in k → try smaller cap
            else left = mid + 1;                            // needs too many → raise cap
        }
        return (int) left;
    }

    private int piecesNeeded(int[] nums, long limit) {
        int pieces = 1; long sum = 0;
        for (int x : nums) {
            if (sum + x > limit) { pieces++; sum = 0; }     // close current piece
            sum += x;
        }
        return pieces;
    }
}
```
**Complexity:** O(n · log(sum)) time, O(1) space.
**Key insight / follow-up:** Use `long` for the bounds and `limit` — the sum of up to 1000 elements each as large as 10^6 fits in `int`, but using `long` is the safe habit and avoids overflow if constraints grow. This is the canonical "minimize the maximum" problem; LC1011 is the same algorithm with `k = days`. The alternative is O(n²·k) DP — binary search on the answer is dramatically simpler and faster.

### LC4 — Median of Two Sorted Arrays *(hard)*
**Problem:** Given two sorted arrays `nums1` and `nums2` of sizes `m` and `n`, return the median of the combined sorted array, in **O(log(m + n))** time.
**Approach:** Binary search a **partition** on the smaller array. Choose `i` elements from `nums1` and `j = half - i` from `nums2` so the left side has exactly half the total. The partition is correct when `maxLeft1 <= minRight2` and `maxLeft2 <= minRight1`. Use `±∞` sentinels for empty sides.
```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        if (nums1.length > nums2.length) return findMedianSortedArrays(nums2, nums1);
        int m = nums1.length, n = nums2.length;
        int half = (m + n + 1) / 2;                         // left-side size
        int lo = 0, hi = m;
        while (lo <= hi) {
            int i = lo + (hi - lo) / 2;                     // take i from nums1
            int j = half - i;                               // take j from nums2
            int left1  = (i == 0) ? Integer.MIN_VALUE : nums1[i - 1];
            int right1 = (i == m) ? Integer.MAX_VALUE : nums1[i];
            int left2  = (j == 0) ? Integer.MIN_VALUE : nums2[j - 1];
            int right2 = (j == n) ? Integer.MAX_VALUE : nums2[j];
            if (left1 <= right2 && left2 <= right1) {        // correct partition
                int maxLeft = Math.max(left1, left2);
                if (((m + n) & 1) == 1) return maxLeft;      // odd total → middle element
                int minRight = Math.min(right1, right2);
                return (maxLeft + minRight) / 2.0;           // even total → average of middles
            } else if (left1 > right2) {
                hi = i - 1;                                  // took too many from nums1
            } else {
                lo = i + 1;                                  // took too few from nums1
            }
        }
        throw new IllegalArgumentException("Input arrays are not sorted");
    }
}
```
**Complexity:** O(log(min(m, n))) time, O(1) space.
**Key insight / follow-up:** Always binary-search the **smaller** array (the recursive swap guarantees this) so `hi = m` stays small and `j = half - i` never goes negative. The `MIN_VALUE`/`MAX_VALUE` sentinels elegantly handle partitions that fall at an array's edge, removing pages of edge-case branching. `half = (m + n + 1) / 2` makes the formula work uniformly for odd and even totals.

---

*[← DSA bank index](README.md)*
