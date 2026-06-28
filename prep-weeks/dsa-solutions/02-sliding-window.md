# DSA Solutions — Sliding Window

> Fixed- and variable-size window techniques for subarray/substring problems. Try each before reading the **Approach** and solution. Solutions are Java 17, idiomatic, and compilable.

---

### LC643 — Maximum Average Subarray I *(easy)*
**Problem:** Given an integer array `nums` and an integer `k`, find a contiguous subarray of length exactly `k` that has the maximum average value, and return that average. The answer is accepted if it is within `10^-5` of the true value.
**Approach:** Fixed-size window. Compute the sum of the first `k` elements, then slide: add the entering element and subtract the leaving one. Track the maximum sum, divide by `k` at the end (divide once to avoid floating-point drift).
```java
static double findMaxAverage(int[] nums, int k) {
    long sum = 0;
    for (int i = 0; i < k; i++) sum += nums[i];
    long best = sum;
    for (int i = k; i < nums.length; i++) {
        sum += nums[i] - nums[i - k];
        best = Math.max(best, sum);
    }
    return (double) best / k;
}
```
**Complexity:** O(n) time, O(1) space.
**Key insight / follow-up:** The classic fixed-window slide — each step is O(1) because you only adjust by the two boundary elements rather than re-summing. Use `long` for the sum to avoid overflow when `n * maxValue` exceeds `int` range, and divide a single time at the end to minimize rounding error.

### LC3 — Longest Substring Without Repeating Characters *(medium)*
**Problem:** Given a string `s`, find the length of the longest substring that contains no repeating characters.
**Approach:** Variable-size window with a "last seen index" map. Expand `right` over the string; when the current character was seen at an index `>= left`, jump `left` to one past that previous occurrence. The window `[left, right]` is always duplicate-free, so track its max length.
```java
static int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> lastSeen = new HashMap<>();
    int best = 0, left = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (lastSeen.containsKey(c) && lastSeen.get(c) >= left) {
            left = lastSeen.get(c) + 1;
        }
        lastSeen.put(c, right);
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```
**Complexity:** O(n) time, O(min(n, charset)) space.
**Key insight / follow-up:** The `lastSeen.get(c) >= left` guard is essential — a stale occurrence outside the current window must not pull `left` backward (e.g. `"abba"`). Jumping `left` directly (instead of shrinking one step at a time) keeps the whole scan O(n). For a fixed ASCII alphabet you can swap the map for an `int[128]` of last-seen indices.

### LC209 — Minimum Size Subarray Sum *(medium)*
**Problem:** Given an array of positive integers `nums` and a positive integer `target`, return the minimal length of a contiguous subarray whose sum is `>= target`. If there is no such subarray, return 0.
**Approach:** Variable-size window. Grow the window by adding `nums[right]`; whenever the running sum is `>= target`, record the length and shrink from the left (subtracting `nums[left]`) as far as the constraint still holds. Because all values are positive, shrinking monotonically reduces the sum, so the two-pointer scan finds every minimal window.
```java
static int minSubArrayLen(int target, int[] nums) {
    int best = Integer.MAX_VALUE, sum = 0, left = 0;
    for (int right = 0; right < nums.length; right++) {
        sum += nums[right];
        while (sum >= target) {
            best = Math.min(best, right - left + 1);
            sum -= nums[left++];
        }
    }
    return best == Integer.MAX_VALUE ? 0 : best;
}
```
**Complexity:** O(n) time, O(1) space.
**Key insight / follow-up:** Each index enters and leaves the window at most once, so despite the nested `while` the total work is O(n). The positivity of the elements is what makes shrinking valid — with negatives the window sum is no longer monotonic and you would need a prefix-sum + deque or binary-search approach. A `0` sentinel (`Integer.MAX_VALUE`) cleanly distinguishes "no valid subarray".

### LC424 — Longest Repeating Character Replacement *(medium)*
**Problem:** Given a string `s` of uppercase English letters and an integer `k`, you may replace at most `k` characters with any uppercase letter. Return the length of the longest substring containing a single repeated letter that you can form after at most `k` replacements.
**Approach:** Variable-size window over letter counts. A window is valid if `(windowLength - countOfMostFrequentLetter) <= k`, i.e. the non-majority characters can all be converted within budget. Expand `right`; if the window becomes invalid, slide `left` forward by one (keeping the window's size monotonic). Track the best length seen.
```java
static int characterReplacement(String s, int k) {
    int[] count = new int[26];
    int left = 0, maxFreq = 0, best = 0;
    for (int right = 0; right < s.length(); right++) {
        maxFreq = Math.max(maxFreq, ++count[s.charAt(right) - 'A']);
        if (right - left + 1 - maxFreq > k) {
            count[s.charAt(left++) - 'A']--;
        }
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```
**Complexity:** O(n) time, O(26) = O(1) space.
**Key insight / follow-up:** The subtle part is that `maxFreq` is never decreased when shrinking. That is fine: the window only ever moves forward and never shrinks below its best-so-far size, so a slightly stale `maxFreq` can only keep the window the same length, never report a wrong (too large) answer. Recomputing the true max each step would also work but turns it into O(26·n) for no benefit.

### LC76 — Minimum Window Substring *(hard)*
**Problem:** Given two strings `s` and `t`, return the shortest substring of `s` that contains every character of `t` including duplicates. If no such window exists, return the empty string `""`. The answer is guaranteed unique when it exists.
**Approach:** Variable-size window with a "need" count map and a `missing` counter (total characters still required). Expand `right`, decrementing `need` for each consumed char and decrementing `missing` only when that char was actually still needed. Once `missing == 0` the window is valid: record it if shorter, then shrink from `left`, restoring `need`/`missing` when removing a character the window still owes.
```java
static String minWindow(String s, String t) {
    if (s.length() < t.length() || t.isEmpty()) return "";
    int[] need = new int[128];
    for (char c : t.toCharArray()) need[c]++;
    int missing = t.length();
    int left = 0, bestStart = 0, bestLen = Integer.MAX_VALUE;
    for (int right = 0; right < s.length(); right++) {
        if (need[s.charAt(right)]-- > 0) missing--;
        while (missing == 0) {
            if (right - left + 1 < bestLen) {
                bestLen = right - left + 1;
                bestStart = left;
            }
            if (need[s.charAt(left++)]++ == 0) missing++;
        }
    }
    return bestLen == Integer.MAX_VALUE ? "" : s.substring(bestStart, bestStart + bestLen);
}
```
**Complexity:** O(|s| + |t|) time, O(128) = O(1) space.
**Key insight / follow-up:** `need[c]` is allowed to go negative — it tracks surplus characters in the window. A char counts toward `missing` only while `need[c]` is still positive (`> 0` before decrement on entry, `== 0` after the restoring decrement on exit), which is what lets the window absorb extra copies without over-counting. The `int[128]` indexes directly by char code, avoiding map overhead for the ASCII input.

### LC567 — Permutation in String *(medium)*
**Problem:** Given strings `s1` and `s2`, return `true` if `s2` contains a permutation of `s1` as a contiguous substring — that is, some window of `s2` with length `s1.length()` is an anagram of `s1`.
**Approach:** Fixed-size window of length `s1.length()`. Keep a frequency array for `s1` and a rolling frequency array for the current window of `s2`. Slide one character at a time (add entering, remove leaving) and compare the two arrays; equality means the window is a permutation of `s1`.
```java
static boolean checkInclusion(String s1, String s2) {
    int n = s1.length();
    if (n > s2.length()) return false;
    int[] need = new int[26], window = new int[26];
    for (int i = 0; i < n; i++) {
        need[s1.charAt(i) - 'a']++;
        window[s2.charAt(i) - 'a']++;
    }
    if (Arrays.equals(need, window)) return true;
    for (int i = n; i < s2.length(); i++) {
        window[s2.charAt(i) - 'a']++;
        window[s2.charAt(i - n) - 'a']--;
        if (Arrays.equals(need, window)) return true;
    }
    return false;
}
```
**Complexity:** O(|s2| · 26) time, O(26) = O(1) space.
**Key insight / follow-up:** Anagram == identical character histograms, so the whole problem reduces to comparing two fixed-size count arrays as the window slides. The `Arrays.equals` over 26 buckets is the dominating cost; you can shave it to O(|s2|) by maintaining a single `matches` counter of how many of the 26 buckets currently agree, updating it incrementally on each add/remove.

### LC438 — Find All Anagrams in a String *(medium)*
**Problem:** Given strings `s` and `p`, return a list of the start indices of every substring of `s` that is an anagram of `p`. The indices may be returned in any order.
**Approach:** Identical sliding-window-of-counts technique as LC567, but instead of returning on the first match, record every window start index where the histograms match. Maintain a `need` array for `p` and a rolling `window` array, slide across `s`, and append `i - n + 1` whenever they are equal.
```java
static List<Integer> findAnagrams(String s, String p) {
    List<Integer> result = new ArrayList<>();
    int n = p.length();
    if (n > s.length()) return result;
    int[] need = new int[26], window = new int[26];
    for (int i = 0; i < n; i++) {
        need[p.charAt(i) - 'a']++;
        window[s.charAt(i) - 'a']++;
    }
    if (Arrays.equals(need, window)) result.add(0);
    for (int i = n; i < s.length(); i++) {
        window[s.charAt(i) - 'a']++;
        window[s.charAt(i - n) - 'a']--;
        if (Arrays.equals(need, window)) result.add(i - n + 1);
    }
    return result;
}
```
**Complexity:** O(|s| · 26) time, O(26) extra space (excluding the output list).
**Key insight / follow-up:** This is "find all" to LC567's "find any" — the only change is collecting indices instead of short-circuiting. The recorded index is `i - n + 1`, the left edge of the window that just slid to include `i`. As with LC567, an incremental `matches` counter reduces the per-step comparison to O(1), giving overall O(|s|).

### LC1493 — Longest Subarray of 1's After Deleting One Element *(medium)*
**Problem:** Given a binary array `nums`, you must delete exactly one element. Return the length of the longest contiguous subarray of 1's in the resulting array. (Since one element must be deleted, the answer is the longest window containing at most one zero, minus that deleted slot.)
**Approach:** Variable-size window allowing at most one zero. Expand `right`, counting zeros inside the window; when zeros exceed one, advance `left` past the leftmost zero. The best answer is `windowLength - 1` (the `-1` accounts for the one mandatory deletion), maximized across all valid windows.
```java
static int longestSubarray(int[] nums) {
    int left = 0, zeros = 0, best = 0;
    for (int right = 0; right < nums.length; right++) {
        if (nums[right] == 0) zeros++;
        while (zeros > 1) {
            if (nums[left++] == 0) zeros--;
        }
        best = Math.max(best, right - left + 1 - 1);
    }
    return best;
}
```
**Complexity:** O(n) time, O(1) space.
**Key insight / follow-up:** The constraint "delete exactly one" is encoded as "window with at most one zero, then subtract one from its length" — the deleted slot is either the single zero or, for an all-ones array, one of the ones. Because the deletion is mandatory, an array of all 1's of length `n` correctly yields `n - 1`. Using `right - left + 1 - 1` (rather than counting ones) keeps the slide a clean O(n) single pass.

---

*[← DSA bank index](README.md)*
