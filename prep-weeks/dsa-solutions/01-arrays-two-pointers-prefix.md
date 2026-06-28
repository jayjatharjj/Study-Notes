# DSA Solutions — Arrays, Two Pointers & Prefix Sum

> Self-contained solutions reference for the classic array patterns: hash-map lookups, two pointers, prefix/suffix products, prefix-sum + hash map, and Kadane's algorithm. Every solution is Java 17, compilable, and idiomatic. Read the **Problem** first, attempt it, then check the **Approach** and code.

---

### LC1 — Two Sum *(easy)*
**Problem:** Given an int array `nums` and an int `target`, return the indices of the two numbers that add up to `target`. Exactly one solution exists, and you may not use the same element twice.
**Approach:** One-pass hash map from value → index. For each element check whether its complement `target - nums[i]` has already been seen.
```java
import java.util.HashMap;
import java.util.Map;

class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> seen = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            int need = target - nums[i];
            if (seen.containsKey(need)) {
                return new int[]{seen.get(need), i};
            }
            seen.put(nums[i], i);
        }
        return new int[]{-1, -1}; // unreachable per problem guarantee
    }
}
```
**Complexity:** Time O(n), Space O(n).
**Key insight / follow-up:** The map trades space for time, collapsing the O(n²) brute force into one pass. If the array were sorted, two pointers give O(1) space — see LC167.

---

### LC167 — Two Sum II (Input Array Is Sorted) *(medium)*
**Problem:** Given a 1-indexed sorted array `numbers` and a `target`, return the 1-based indices of the two numbers that sum to `target`. Exactly one solution exists; use O(1) extra space.
**Approach:** Two pointers at both ends. If the sum is too big move the right pointer in; if too small move the left pointer out. Sortedness guarantees this never skips the answer.
```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int lo = 0, hi = numbers.length - 1;
        while (lo < hi) {
            int sum = numbers[lo] + numbers[hi];
            if (sum == target) {
                return new int[]{lo + 1, hi + 1}; // 1-indexed
            } else if (sum < target) {
                lo++;
            } else {
                hi--;
            }
        }
        return new int[]{-1, -1}; // unreachable
    }
}
```
**Complexity:** Time O(n), Space O(1).
**Key insight / follow-up:** Sorted input unlocks the converging two-pointer pattern with no extra memory. Each step provably eliminates one row/column of the implicit pair matrix.

---

### LC125 — Valid Palindrome *(easy)*
**Problem:** Given a string `s`, return true if it reads the same forward and backward after lowercasing and removing all non-alphanumeric characters.
**Approach:** Two pointers from both ends; skip non-alphanumeric chars, compare lowercased letters.
```java
class Solution {
    public boolean isPalindrome(String s) {
        int i = 0, j = s.length() - 1;
        while (i < j) {
            while (i < j && !Character.isLetterOrDigit(s.charAt(i))) i++;
            while (i < j && !Character.isLetterOrDigit(s.charAt(j))) j--;
            if (Character.toLowerCase(s.charAt(i)) != Character.toLowerCase(s.charAt(j))) {
                return false;
            }
            i++;
            j--;
        }
        return true;
    }
}
```
**Complexity:** Time O(n), Space O(1).
**Key insight / follow-up:** Filtering in place with two pointers avoids building a cleaned copy. `Character.isLetterOrDigit` / `toLowerCase` handle the normalization without regex.

---

### LC11 — Container With Most Water *(medium)*
**Problem:** Given `height[]` where each value is a vertical line at that index, pick two lines that with the x-axis form a container holding the most water. Return the maximum area. Area = `min(height[i], height[j]) * (j - i)`.
**Approach:** Two pointers at the ends. Area is bounded by the shorter wall, so always move the shorter pointer inward — moving the taller one can never increase area.
```java
class Solution {
    public int maxArea(int[] height) {
        int lo = 0, hi = height.length - 1, best = 0;
        while (lo < hi) {
            int area = Math.min(height[lo], height[hi]) * (hi - lo);
            best = Math.max(best, area);
            if (height[lo] < height[hi]) {
                lo++;
            } else {
                hi--;
            }
        }
        return best;
    }
}
```
**Complexity:** Time O(n), Space O(1).
**Key insight / follow-up:** Width only shrinks as pointers converge, so to ever beat the current area you need a taller limiting wall — that can only come from advancing the shorter side. Greedy two pointers replaces the O(n²) scan.

---

### LC42 — Trapping Rain Water *(hard)*
**Problem:** Given non-negative `height[]` representing an elevation map (bar width 1), compute how many units of water are trapped after raining.
**Approach:** Two pointers tracking `leftMax` and `rightMax`. Water above a bar is bounded by the smaller of the tallest walls on either side; process from the side whose running max is smaller, where that side's max is the true bound.
```java
class Solution {
    public int trap(int[] height) {
        int lo = 0, hi = height.length - 1;
        int leftMax = 0, rightMax = 0, water = 0;
        while (lo < hi) {
            if (height[lo] < height[hi]) {
                leftMax = Math.max(leftMax, height[lo]);
                water += leftMax - height[lo];
                lo++;
            } else {
                rightMax = Math.max(rightMax, height[hi]);
                water += rightMax - height[hi];
                hi--;
            }
        }
        return water;
    }
}
```
**Complexity:** Time O(n), Space O(1).
**Key insight / follow-up:** When `height[lo] < height[hi]`, the left side is the binding constraint, so `leftMax` is guaranteed correct regardless of what lies between. This beats the O(n)-space prefix/suffix-max-arrays approach.

---

### LC238 — Product of Array Except Self *(medium)*
**Problem:** Given `nums[]`, return `answer[]` where `answer[i]` is the product of all elements except `nums[i]`. Solve without division and in O(n).
**Approach:** Two passes. First fill `answer[i]` with the prefix product of everything to the left, then multiply in the running suffix product from the right.
```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int n = nums.length;
        int[] answer = new int[n];
        answer[0] = 1;
        for (int i = 1; i < n; i++) {
            answer[i] = answer[i - 1] * nums[i - 1]; // product of all to the left
        }
        int suffix = 1;
        for (int i = n - 1; i >= 0; i--) {
            answer[i] *= suffix;                     // fold in product to the right
            suffix *= nums[i];
        }
        return answer;
    }
}
```
**Complexity:** Time O(n), Space O(1) extra (output array excluded).
**Key insight / follow-up:** Avoiding division also dodges the zero-element trap. The suffix product is carried in a single scalar instead of a second array, keeping it O(1) auxiliary space.

---

### LC189 — Rotate Array *(medium)*
**Problem:** Rotate `nums[]` to the right by `k` steps in place. For `[1,2,3,4,5,6,7]`, `k=3` gives `[5,6,7,1,2,3,4]`.
**Approach:** Reverse the whole array, then reverse the first `k` and the remaining `n-k`. The double reversal lands every element in its rotated slot.
```java
class Solution {
    public void rotate(int[] nums, int k) {
        int n = nums.length;
        k %= n;                  // k may exceed n
        reverse(nums, 0, n - 1);
        reverse(nums, 0, k - 1);
        reverse(nums, k, n - 1);
    }

    private void reverse(int[] a, int i, int j) {
        while (i < j) {
            int tmp = a[i];
            a[i++] = a[j];
            a[j--] = tmp;
        }
    }
}
```
**Complexity:** Time O(n), Space O(1).
**Key insight / follow-up:** `k %= n` prevents redundant full rotations. The reverse trick is the classic O(1)-space rotation; a temp array is simpler but uses O(n) memory.

---

### LC560 — Subarray Sum Equals K *(medium)*
**Problem:** Given `nums[]` (may include negatives) and an integer `k`, return the total number of contiguous subarrays whose sum equals `k`.
**Approach:** Prefix sum + hash map of `prefixSum → count`. A subarray ending at `i` sums to `k` whenever `prefixSum - k` was seen before; add its frequency.
```java
import java.util.HashMap;
import java.util.Map;

class Solution {
    public int subarraySum(int[] nums, int k) {
        Map<Integer, Integer> counts = new HashMap<>();
        counts.put(0, 1);            // empty prefix
        int prefix = 0, result = 0;
        for (int x : nums) {
            prefix += x;
            result += counts.getOrDefault(prefix - k, 0);
            counts.merge(prefix, 1, Integer::sum);
        }
        return result;
    }
}
```
**Complexity:** Time O(n), Space O(n).
**Key insight / follow-up:** Seeding the map with `{0:1}` counts subarrays that start at index 0. Because negatives are allowed, a sliding window does not work — the prefix-sum map is the standard tool.

---

### LC53 — Maximum Subarray (Kadane) *(medium)*
**Problem:** Given an int array `nums`, find the contiguous subarray with the largest sum and return that sum. The subarray must contain at least one element.
**Approach:** Kadane's algorithm. At each element decide whether to extend the previous running sum or restart from the current element; track the best seen.
```java
class Solution {
    public int maxSubArray(int[] nums) {
        int current = nums[0], best = nums[0];
        for (int i = 1; i < nums.length; i++) {
            current = Math.max(nums[i], current + nums[i]);
            best = Math.max(best, current);
        }
        return best;
    }
}
```
**Complexity:** Time O(n), Space O(1).
**Key insight / follow-up:** `current` resets whenever the running sum turns into a liability (goes negative relative to starting fresh). Initializing from `nums[0]` (not 0) handles all-negative arrays correctly.

---

### LC152 — Maximum Product Subarray *(medium)*
**Problem:** Given an int array `nums`, find the contiguous subarray with the largest product and return that product.
**Approach:** Track both the running max and min products, because a negative number flips them. On each element swap when it is negative, then update both candidates against the element alone.
```java
class Solution {
    public int maxProduct(int[] nums) {
        int maxSoFar = nums[0], minSoFar = nums[0], best = nums[0];
        for (int i = 1; i < nums.length; i++) {
            int x = nums[i];
            if (x < 0) {                 // negative flips max and min roles
                int t = maxSoFar;
                maxSoFar = minSoFar;
                minSoFar = t;
            }
            maxSoFar = Math.max(x, maxSoFar * x);
            minSoFar = Math.min(x, minSoFar * x);
            best = Math.max(best, maxSoFar);
        }
        return best;
    }
}
```
**Complexity:** Time O(n), Space O(1).
**Key insight / follow-up:** Unlike sums, a large negative product can become the maximum after multiplying by another negative — so the minimum must be carried too. Zeros naturally reset both via the `max(x, ...)` / `min(x, ...)` restart.

---

### LC121 — Best Time to Buy and Sell Stock *(easy)*
**Problem:** Given `prices[]` where `prices[i]` is the stock price on day `i`, choose one day to buy and a later day to sell to maximize profit. Return the max profit, or 0 if none is possible.
**Approach:** Single pass tracking the minimum price seen so far; at each day the best profit is the current price minus that running minimum.
```java
class Solution {
    public int maxProfit(int[] prices) {
        int minPrice = Integer.MAX_VALUE, best = 0;
        for (int p : prices) {
            minPrice = Math.min(minPrice, p);
            best = Math.max(best, p - minPrice);
        }
        return best;
    }
}
```
**Complexity:** Time O(n), Space O(1).
**Key insight / follow-up:** This is Kadane in disguise — maximizing the largest "drop-to-rise" is equivalent to the max subarray of daily price deltas. Buying must precede selling, which the running-min ordering enforces.

---

*[← DSA bank index](README.md)*
