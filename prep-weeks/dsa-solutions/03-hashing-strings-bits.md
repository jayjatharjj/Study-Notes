# DSA Solutions — Hashing, Strings & Bit Manipulation

> Curated LeetCode patterns for hash maps, string manipulation, and bit tricks. Each problem has a self-contained statement, an approach, a compilable Java 17 solution, complexity, and a key insight or follow-up. Try each yourself before reading the solution.

---

### LC242 — Valid Anagram *(easy)*
**Problem:** Given two strings `s` and `t`, return `true` if `t` is an anagram of `s` — i.e., `t` uses exactly the same characters as `s` with the same frequencies, just possibly reordered. Assume lowercase English letters.
**Approach:** Lengths must match. Tally each character of `s` in an `int[26]`, then decrement for each character of `t`. If every count returns to zero, they are anagrams. A single counts array suffices.
```java
class Solution {
    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) return false;
        int[] counts = new int[26];
        for (int i = 0; i < s.length(); i++) {
            counts[s.charAt(i) - 'a']++;
            counts[t.charAt(i) - 'a']--;
        }
        for (int c : counts) {
            if (c != 0) return false;
        }
        return true;
    }
}
```
**Complexity:** O(n) time, O(1) space (fixed 26-element array).
**Key insight / follow-up:** A frequency array beats sorting (O(n log n)). Follow-up: for Unicode input, use a `HashMap<Character, Integer>` (or `Map<Integer, Integer>` over code points) instead of the fixed array.

### LC49 — Group Anagrams *(medium)*
**Problem:** Given an array of strings `strs`, group the strings that are anagrams of each other. Return the groups in any order (each group is a list of the original strings).
**Approach:** Two anagrams share a canonical form. Use the sorted characters of each word as a map key, and accumulate words into the matching bucket with `computeIfAbsent`.
```java
import java.util.*;

class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> groups = new HashMap<>();
        for (String s : strs) {
            char[] chars = s.toCharArray();
            Arrays.sort(chars);
            String key = new String(chars);
            groups.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
        }
        return new ArrayList<>(groups.values());
    }
}
```
**Complexity:** O(n · k log k) time, O(n · k) space, where n is the number of words and k the max word length.
**Key insight / follow-up:** The sorted string is a canonical anagram signature. Follow-up: with only lowercase letters, build the key from a 26-count signature (e.g., `"#1#0#2..."`), giving O(n · k) time with no sort.

### LC347 — Top K Frequent Elements *(medium)*
**Problem:** Given an integer array `nums` and an integer `k`, return the `k` most frequent elements. The answer is guaranteed to be unique; return it in any order.
**Approach:** Count frequencies in a map, then use bucket sort: index `i` of a buckets array holds all values that appear exactly `i` times. Walk buckets from highest frequency down, collecting until `k` elements are gathered.
```java
import java.util.*;

class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> freq = new HashMap<>();
        for (int n : nums) freq.merge(n, 1, Integer::sum);

        // bucket[i] = list of numbers appearing exactly i times
        List<Integer>[] buckets = new List[nums.length + 1];
        for (var e : freq.entrySet()) {
            int f = e.getValue();
            if (buckets[f] == null) buckets[f] = new ArrayList<>();
            buckets[f].add(e.getKey());
        }

        int[] result = new int[k];
        int idx = 0;
        for (int f = buckets.length - 1; f >= 0 && idx < k; f--) {
            if (buckets[f] == null) continue;
            for (int val : buckets[f]) {
                result[idx++] = val;
                if (idx == k) break;
            }
        }
        return result;
    }
}
```
**Complexity:** O(n) time, O(n) space.
**Key insight / follow-up:** Frequencies are bounded by `n`, so bucket sort gives linear time — better than a heap's O(n log k). Follow-up: a `PriorityQueue` of size `k` ordered by frequency is the standard heap alternative when memory for buckets is a concern.

### LC128 — Longest Consecutive Sequence *(medium)*
**Problem:** Given an unsorted integer array `nums`, return the length of the longest run of consecutive integers (e.g., `[100,4,200,1,3,2]` → 4 for `1,2,3,4`). Must run in O(n).
**Approach:** Put all numbers in a hash set. A number starts a sequence only if `num - 1` is absent. From each such start, walk upward (`num+1`, `num+2`, ...) counting the run length.
```java
import java.util.*;

class Solution {
    public int longestConsecutive(int[] nums) {
        Set<Integer> set = new HashSet<>();
        for (int n : nums) set.add(n);

        int longest = 0;
        for (int n : set) {
            if (set.contains(n - 1)) continue; // not a sequence start
            int length = 1;
            int current = n;
            while (set.contains(current + 1)) {
                current++;
                length++;
            }
            longest = Math.max(longest, length);
        }
        return longest;
    }
}
```
**Complexity:** O(n) time, O(n) space.
**Key insight / follow-up:** Skipping non-starts means the inner `while` runs at most once per element overall, keeping it O(n) despite the nested loop. Sorting would be O(n log n) — the set trick is what hits the required linear bound.

### LC5 — Longest Palindromic Substring *(medium)*
**Problem:** Given a string `s`, return the longest contiguous substring of `s` that is a palindrome. If multiple exist, return any one.
**Approach:** Expand around centers. Every palindrome has a center: either one character (odd length) or between two characters (even length). For each of the `2n-1` centers, expand outward while characters match, tracking the longest span found.
```java
class Solution {
    public String longestPalindrome(String s) {
        if (s == null || s.length() < 1) return "";
        int start = 0, end = 0;
        for (int i = 0; i < s.length(); i++) {
            int odd = expand(s, i, i);       // odd-length, center at i
            int even = expand(s, i, i + 1);  // even-length, center between i and i+1
            int len = Math.max(odd, even);
            if (len > end - start + 1) {
                start = i - (len - 1) / 2;
                end = i + len / 2;
            }
        }
        return s.substring(start, end + 1);
    }

    private int expand(String s, int left, int right) {
        while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
            left--;
            right++;
        }
        return right - left - 1; // length after the loop overshoots by one on each side
    }
}
```
**Complexity:** O(n²) time, O(1) space.
**Key insight / follow-up:** Handling odd and even centers separately covers every palindrome. Follow-up: Manacher's algorithm solves it in O(n) by reusing mirror information, but expand-around-center is the expected interview answer.

### LC271 — Encode and Decode Strings *(medium)*
**Problem:** Design an algorithm to encode a list of strings into a single string, then decode that single string back into the original list. The strings may contain any characters (including delimiters and digits). Implement `encode(List<String>)` and `decode(String)`.
**Approach:** Length-prefix each string: write its length, a non-digit delimiter (`#`), then the raw chars. On decode, read digits up to the `#` to learn the length, then slice exactly that many characters — so the content can contain anything safely.
```java
import java.util.*;

public class Codec {
    // Encodes a list of strings to a single string.
    public String encode(List<String> strs) {
        StringBuilder sb = new StringBuilder();
        for (String s : strs) {
            sb.append(s.length()).append('#').append(s);
        }
        return sb.toString();
    }

    // Decodes a single string back to a list of strings.
    public List<String> decode(String s) {
        List<String> result = new ArrayList<>();
        int i = 0;
        while (i < s.length()) {
            int j = i;
            while (s.charAt(j) != '#') j++;      // read length digits
            int len = Integer.parseInt(s.substring(i, j));
            String word = s.substring(j + 1, j + 1 + len);
            result.add(word);
            i = j + 1 + len;
        }
        return result;
    }
}
```
**Complexity:** O(N) encode and decode, where N is the total number of characters; O(N) space for the output.
**Key insight / follow-up:** Length-prefixing (the chunked-transfer idea) sidesteps escaping entirely — the content is never scanned for delimiters. A naive `join("#")` breaks whenever a string itself contains `#`.

### LC680 — Valid Palindrome II *(easy)*
**Problem:** Given a string `s`, return `true` if it can be made a palindrome by deleting at most one character (deleting zero is allowed). Assume lowercase letters.
**Approach:** Two pointers from both ends. On the first mismatch, you may delete exactly one of the two offending characters — so check whether the substring with the left char skipped, or with the right char skipped, is a palindrome.
```java
class Solution {
    public boolean validPalindrome(String s) {
        int i = 0, j = s.length() - 1;
        while (i < j) {
            if (s.charAt(i) != s.charAt(j)) {
                return isPalindrome(s, i + 1, j) || isPalindrome(s, i, j - 1);
            }
            i++;
            j--;
        }
        return true;
    }

    private boolean isPalindrome(String s, int i, int j) {
        while (i < j) {
            if (s.charAt(i++) != s.charAt(j--)) return false;
        }
        return true;
    }
}
```
**Complexity:** O(n) time, O(1) space.
**Key insight / follow-up:** At the first mismatch there are only two repair options, each verifiable in linear time — so the overall cost stays linear. Follow-up: allowing up to `k` deletions turns this into a DP / edit-distance problem.

### LC136 — Single Number *(easy)*
**Problem:** Given a non-empty array `nums` where every element appears exactly twice except for one element that appears once, return that single element. Solve in linear time using constant extra space.
**Approach:** XOR all elements. `x ^ x == 0` and `x ^ 0 == x`, so all paired values cancel out and the lone value remains.
```java
class Solution {
    public int singleNumber(int[] nums) {
        int result = 0;
        for (int n : nums) {
            result ^= n;
        }
        return result;
    }
}
```
**Complexity:** O(n) time, O(1) space.
**Key insight / follow-up:** XOR is associative and commutative, so pairing order does not matter — duplicates annihilate. Follow-up (LC137): if every element appears three times except one, count bits per position mod 3.

### LC191 — Number of 1 Bits *(easy)*
**Problem:** Write a function that takes an unsigned integer (given as an `int`) and returns the number of `1` bits in its binary representation (the Hamming weight).
**Approach:** Brian Kernighan's trick: `n & (n - 1)` clears the lowest set bit. Repeat until `n` is zero; the iteration count equals the number of set bits. Use `n != 0` (not `> 0`) so the high "sign" bit is handled correctly.
```java
class Solution {
    public int hammingWeight(int n) {
        int count = 0;
        while (n != 0) {
            n &= (n - 1); // clear the lowest set bit
            count++;
        }
        return count;
    }
}
```
**Complexity:** O(k) time where k is the number of set bits (≤ 32), O(1) space.
**Key insight / follow-up:** Looping once per set bit beats scanning all 32 bit positions. Built-in equivalent: `Integer.bitCount(n)`. The `n != 0` guard matters because in Java `int` is signed — `> 0` would skip negatives.

### LC338 — Counting Bits *(easy)*
**Problem:** Given an integer `n`, return an array `ans` of length `n + 1` where `ans[i]` is the number of `1` bits in the binary representation of `i`, for every `i` from 0 to `n`.
**Approach:** DP using the relation `bits[i] = bits[i >> 1] + (i & 1)`. Dropping the lowest bit (`i >> 1`) gives an already-computed smaller value; add 1 back if the dropped bit was set.
```java
class Solution {
    public int[] countBits(int n) {
        int[] bits = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            bits[i] = bits[i >> 1] + (i & 1);
        }
        return bits;
    }
}
```
**Complexity:** O(n) time, O(n) space (for the output array).
**Key insight / follow-up:** Reusing `bits[i >> 1]` makes each entry O(1), giving overall linear time — far better than calling `bitCount` (O(log i)) for each of the n+1 values. An equivalent recurrence is `bits[i] = bits[i & (i - 1)] + 1`.

### LC268 — Missing Number *(easy)*
**Problem:** Given an array `nums` containing `n` distinct numbers drawn from the range `[0, n]`, exactly one number in that range is missing. Return it.
**Approach:** XOR every index `0..n` together with every value in the array. Each present number cancels with its matching index; the leftover is the missing number. This avoids the overflow risk of the sum formula.
```java
class Solution {
    public int missingNumber(int[] nums) {
        int n = nums.length;
        int result = n; // start with the index n, which has no array slot
        for (int i = 0; i < n; i++) {
            result ^= i ^ nums[i];
        }
        return result;
    }
}
```
**Complexity:** O(n) time, O(1) space.
**Key insight / follow-up:** XOR-ing indices against values cancels every matched pair, leaving the unmatched (missing) index. Alternative: expected sum `n*(n+1)/2` minus the actual sum — simpler to explain but can overflow for large `n` (use `long`).

---

*[← DSA bank index](README.md)*
