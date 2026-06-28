# DSA Solutions — Heaps & Design Data Structures

> Full worked solutions in Java 17. Each problem gives a self-contained statement, the approach, a complete compilable solution, complexity, and a key insight or follow-up. Two recurring engines drive this file: the **binary heap** (`PriorityQueue` in Java) for streaming top-K / order-statistics work, and **composite data structures** (hash map + linked list / array) that trade extra memory for O(1) operations.

**Heap selection cheat-sheet — read this once:**

- **"K largest" → use a MIN-heap of size K.** Keep the heap at size K; whenever it grows past K, poll the smallest. The root is then the *threshold* — the K-th largest seen so far — and any new element smaller than the root is rejected in O(log K). Symmetrically, **"K smallest" → MAX-heap of size K**. The counterintuitive inversion (min-heap for *largest*) is the single most common heap mistake.
- **Why not just sort?** Sorting is O(n log n); a size-K heap is O(n log K), and it works on *streams* where you never hold all n elements at once (LC703).
- **Streaming median → two heaps.** A MAX-heap holds the smaller half, a MIN-heap holds the larger half. Keep their sizes balanced (differ by ≤ 1). The median is either the larger heap's root or the average of both roots. Every insert is O(log n); every median query is O(1).
- **Merging K sorted sequences → MIN-heap of K cursors.** Hold one "front" element per sequence; repeatedly poll the global minimum and push its successor. O(N log K) total.
- In Java, `PriorityQueue` is a min-heap by default. For a max-heap use `new PriorityQueue<>(Collections.reverseOrder())` or `(a, b) -> b - a` (mind overflow — prefer `Integer.compare(b, a)`).

---

### LC215 — Kth Largest Element in an Array *(medium)*
**Problem:** Given an integer array `nums` and an integer `k`, return the k-th largest element in the array. Note that it is the k-th largest in sorted order, not the k-th distinct element. Example: `nums = [3,2,1,5,6,4]`, `k = 2` → `5`; `nums = [3,2,3,1,2,4,5,5,6]`, `k = 4` → `4`.

**Approach:** Maintain a **min-heap of size k**. Scan all elements, offering each one; whenever the heap exceeds size k, poll the smallest. After the scan the heap holds the k largest elements and its root is the smallest of those — exactly the k-th largest.
```java
import java.util.PriorityQueue;

class Solution {
    public int findKthLargest(int[] nums, int k) {
        PriorityQueue<Integer> minHeap = new PriorityQueue<>(); // min-heap
        for (int n : nums) {
            minHeap.offer(n);
            if (minHeap.size() > k) {
                minHeap.poll(); // drop the smallest, keep the top k
            }
        }
        return minHeap.peek(); // root = k-th largest
    }
}
```
**Complexity:** O(n log k) time, O(k) space.

**Key insight / follow-up:** A min-heap of size k beats sorting (O(n log n)) when k ≪ n, and it streams. The asymptotically optimal answer is **Quickselect** — average O(n), worst-case O(n²) — which partitions around a pivot (like quicksort) but recurses into only one side. Interviewers often want you to mention Quickselect; the heap is the safe, easy-to-code choice with no worst-case blowup.

---

### LC295 — Find Median from Data Stream *(hard)*
**Problem:** The median is the middle value in an ordered list; with an even count it is the average of the two middle values. Design a structure that supports `void addNum(int num)` to add an integer from a data stream and `double findMedian()` to return the median of all elements added so far. Example: `addNum(1); addNum(2); findMedian() → 1.5; addNum(3); findMedian() → 2.0`.

**Approach:** Keep two heaps. A **max-heap `lo`** holds the smaller half; a **min-heap `hi`** holds the larger half. Invariant: every element of `lo` ≤ every element of `hi`, and `0 ≤ lo.size() - hi.size() ≤ 1`. On insert, push to `lo`, move `lo`'s max into `hi` (to keep the partition correct), then rebalance if `hi` got larger. The median is `lo`'s root (odd total) or the average of both roots (even total).
```java
import java.util.Collections;
import java.util.PriorityQueue;

class MedianFinder {
    private final PriorityQueue<Integer> lo = new PriorityQueue<>(Collections.reverseOrder()); // smaller half, max-heap
    private final PriorityQueue<Integer> hi = new PriorityQueue<>();                            // larger half, min-heap

    public MedianFinder() { }

    public void addNum(int num) {
        lo.offer(num);              // 1. add to smaller half
        hi.offer(lo.poll());        // 2. balance order: largest of lo moves to hi
        if (hi.size() > lo.size()) { // 3. keep lo the same size or one larger
            lo.offer(hi.poll());
        }
    }

    public double findMedian() {
        if (lo.size() > hi.size()) {
            return lo.peek();        // odd count: middle sits in lo
        }
        return (lo.peek() + hi.peek()) / 2.0; // even count: average the two middles
    }
}
```
**Complexity:** `addNum` O(log n), `findMedian` O(1); O(n) space.

**Key insight / follow-up:** The push-then-shuffle pattern (`lo.offer(num); hi.offer(lo.poll())`) automatically keeps both heaps correctly partitioned without any explicit comparison of `num` against the roots — that is the elegant trick. Follow-ups: if numbers are bounded (e.g. 0–100), a counting/bucket array gives O(1) inserts; if 99% of values fall in a range, keep a count array for the common range and heaps for the tails.

---

### LC23 — Merge k Sorted Lists *(hard)*
**Problem:** You are given an array of `k` linked lists, each sorted in ascending order. Merge all the lists into one sorted linked list and return its head. Example: `[[1,4,5],[1,3,4],[2,6]]` → `1→1→2→3→4→4→5→6`. The list array may be empty or contain null lists.

**Approach:** Put the head node of each non-empty list into a **min-heap keyed by `val`**. Repeatedly poll the smallest node, append it to the result, and push that node's successor (if any). The heap always holds at most k nodes — one "front" per list — so each poll yields the global minimum.
```java
import java.util.PriorityQueue;

// Definition: class ListNode { int val; ListNode next; ListNode(int v){val=v;} }
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        PriorityQueue<ListNode> heap =
            new PriorityQueue<>((a, b) -> Integer.compare(a.val, b.val));
        for (ListNode node : lists) {
            if (node != null) heap.offer(node);
        }
        ListNode dummy = new ListNode(0);
        ListNode tail = dummy;
        while (!heap.isEmpty()) {
            ListNode smallest = heap.poll();
            tail.next = smallest;
            tail = tail.next;
            if (smallest.next != null) {
                heap.offer(smallest.next); // push the next node from that list
            }
        }
        return dummy.next;
    }
}
```
**Complexity:** O(N log k) time where N is the total number of nodes; O(k) heap space.

**Key insight / follow-up:** The heap size stays at k, so each of the N nodes costs O(log k) — far better than naive pairwise merging from the front (O(Nk)). An alternative with the same O(N log k) bound and O(1) extra space is **divide-and-conquer pairwise merge** (merge lists in pairs, halving the count each round, log k rounds × O(N) per round). Reusing existing nodes (no copying) keeps space minimal.

---

### LC373 — Find K Pairs with Smallest Sums *(medium)*
**Problem:** Given two integer arrays `nums1` and `nums2` sorted in ascending order, and an integer `k`, define a pair `(u, v)` with `u` from `nums1` and `v` from `nums2`. Return the `k` pairs `(u1,v1),(u2,v2),…` with the smallest sums. Example: `nums1 = [1,7,11]`, `nums2 = [2,4,6]`, `k = 3` → `[[1,2],[1,4],[1,6]]`.

**Approach:** This is LC23 in disguise — think of `nums1.length` virtual sorted lists, where list `i` is `(nums1[i]+nums2[0]), (nums1[i]+nums2[1]), …`. Seed a **min-heap by pair sum** with the first column: each `(i, 0)`. Poll the smallest pair `(i, j)`, record it, and push its right neighbour `(i, j+1)`. This explores the grid of sums in increasing order without generating all m×n pairs.
```java
import java.util.ArrayList;
import java.util.List;
import java.util.PriorityQueue;

class Solution {
    public List<List<Integer>> kSmallestPairs(int[] nums1, int[] nums2, int k) {
        List<List<Integer>> result = new ArrayList<>();
        if (nums1.length == 0 || nums2.length == 0 || k == 0) return result;

        // heap holds index pairs {i, j}, ordered by nums1[i] + nums2[j]
        PriorityQueue<int[]> heap = new PriorityQueue<>(
            (a, b) -> Integer.compare(nums1[a[0]] + nums2[a[1]], nums1[b[0]] + nums2[b[1]]));

        // seed with the first column: each row paired with nums2[0]
        for (int i = 0; i < nums1.length && i < k; i++) {
            heap.offer(new int[]{i, 0});
        }
        while (k-- > 0 && !heap.isEmpty()) {
            int[] cur = heap.poll();
            int i = cur[0], j = cur[1];
            result.add(List.of(nums1[i], nums2[j]));
            if (j + 1 < nums2.length) {
                heap.offer(new int[]{i, j + 1}); // next pair in the same row
            }
        }
        return result;
    }
}
```
**Complexity:** O(k log k) time, O(k) space (the heap never exceeds ~k entries since we seed at most k rows).

**Key insight / follow-up:** Storing **indices, not values**, in the heap is what lets you lazily advance one column at a time and avoid materializing the full m×n sum grid. Seeding only the first column (then expanding rightward) guarantees every reachable pair is generated exactly once. Capping the seed at `min(k, nums1.length)` rows avoids needless work when k is small.

---

### LC692 — Top K Frequent Words *(medium)*
**Problem:** Given an array of strings `words` and an integer `k`, return the `k` most frequent words. The answer must be sorted by frequency from highest to lowest; words with the same frequency are ordered **lexicographically (alphabetically)**. Example: `words = ["i","love","leetcode","i","love","coding"]`, `k = 2` → `["i","love"]` (both appear twice, "i" before "love").

**Approach:** Count frequencies in a hash map. Then maintain a **min-heap of size k** whose ordering is the *inverse* of the desired final order: less frequent first, and for ties, lexicographically larger first. That way the "worst" candidate sits at the root and gets evicted when the heap overflows. Finally drain the heap and reverse, since the heap pops worst-to-best.
```java
import java.util.*;

class Solution {
    public List<String> topKFrequent(String[] words, int k) {
        Map<String, Integer> count = new HashMap<>();
        for (String w : words) {
            count.merge(w, 1, Integer::sum);
        }
        // min-heap: root is the least desirable entry to keep.
        // lower frequency is "worse"; on a frequency tie, larger word is "worse".
        PriorityQueue<String> heap = new PriorityQueue<>((a, b) -> {
            int fa = count.get(a), fb = count.get(b);
            if (fa != fb) return Integer.compare(fa, fb); // lower freq -> root
            return b.compareTo(a);                        // larger word -> root
        });
        for (String word : count.keySet()) {
            heap.offer(word);
            if (heap.size() > k) heap.poll(); // evict the worst
        }
        LinkedList<String> result = new LinkedList<>();
        while (!heap.isEmpty()) {
            result.addFirst(heap.poll()); // pop worst-first, so prepend
        }
        return result;
    }
}
```
**Complexity:** O(n + m log k) time (n = total words, m = distinct words), O(m + k) space.

**Key insight / follow-up:** The tricky part is the **comparator inversion**: to keep the *best* k with a size-k heap you must order the heap so the *worst* surviving candidate is at the root. The tie-break `b.compareTo(a)` (reverse-lex at the root) is deliberately opposite the final requirement (forward-lex), and `addFirst` undoes the worst-first pop order. A simpler O(m log m) alternative just sorts all distinct words with the final comparator and takes the first k.

---

### LC621 — Task Scheduler *(medium)*
**Problem:** Given a char array `tasks` representing CPU tasks (each a capital letter A–Z) and an integer `n`, each task takes one unit of time. Between two **identical** tasks there must be at least `n` units of gap (during which the CPU runs a different task or stays idle). Return the minimum number of time units the CPU needs to finish all tasks. Example: `tasks = ["A","A","A","B","B","B"]`, `n = 2` → `8` (`A B idle A B idle A B`).

**Approach:** The schedule length is governed by the **most frequent task**. Let `maxFreq` be the highest task count and `maxCount` the number of tasks sharing that count. The frequent tasks lay out a frame of `(maxFreq - 1)` full blocks of length `(n + 1)`, plus a final partial block of `maxCount` tasks. Idle slots are whatever is left. The answer is the max of this framed length and `tasks.length` (when there are so many distinct tasks that no idling is ever needed).
```java
class Solution {
    public int leastInterval(char[] tasks, int n) {
        int[] freq = new int[26];
        for (char t : tasks) freq[t - 'A']++;

        int maxFreq = 0;
        for (int f : freq) maxFreq = Math.max(maxFreq, f);

        int maxCount = 0; // how many tasks hit that max frequency
        for (int f : freq) if (f == maxFreq) maxCount++;

        // (maxFreq - 1) frames of size (n + 1), plus the last row of maxCount tasks
        int framed = (maxFreq - 1) * (n + 1) + maxCount;
        return Math.max(framed, tasks.length);
    }
}
```
**Complexity:** O(t) time (t = number of tasks; the 26-slot scan is O(1)), O(1) space.

**Key insight / follow-up:** Although this is filed under heaps, the **greedy math formula is O(t) and beats any heap simulation**. Take `max(framed, tasks.length)`: when distinct tasks are plentiful they fill every gap, so no idling occurs and the answer is simply `tasks.length`. The heap-based simulation (always run the most-frequent available task each tick, using a cooldown queue) is what you reach for when the follow-up asks for the **actual ordering** rather than just the count.

---

### LC703 — Kth Largest Element in a Stream *(easy)*
**Problem:** Design a class to find the k-th largest element in a stream (the k-th largest in sorted order, not the k-th distinct). Implement `KthLargest(int k, int[] nums)` to initialise with an initial set, and `int add(int val)` to append `val` to the stream and return the k-th largest element so far. Example: `KthLargest(3, [4,5,8,2])`; `add(3) → 4; add(5) → 5; add(10) → 5; add(9) → 8; add(4) → 8`.

**Approach:** Keep a **min-heap capped at size k** holding the k largest elements seen so far. Its root is always the k-th largest. On `add`, offer the new value and, if the heap grew past k, poll the smallest. Then return the root.
```java
import java.util.PriorityQueue;

class KthLargest {
    private final int k;
    private final PriorityQueue<Integer> minHeap = new PriorityQueue<>();

    public KthLargest(int k, int[] nums) {
        this.k = k;
        for (int n : nums) {
            add(n); // reuse the same capped-heap logic
        }
    }

    public int add(int val) {
        minHeap.offer(val);
        if (minHeap.size() > k) {
            minHeap.poll(); // keep only the k largest
        }
        return minHeap.peek(); // k-th largest so far
    }
}
```
**Complexity:** `add` O(log k); constructor O(m log k) for m initial elements; O(k) space.

**Key insight / follow-up:** This is the streaming sibling of LC215 — the cap-at-k min-heap is the same idea, now persisted across calls. Reusing `add` inside the constructor keeps the invariant in one place. Holding only k elements (not the whole stream) is the win: memory is O(k) regardless of how long the stream runs, and each update is O(log k).

---

### LC1046 — Last Stone Weight *(easy)*
**Problem:** You have an array `stones` of positive integers, each the weight of a stone. Each turn, smash the two heaviest stones `x ≤ y` together: if `x == y` both are destroyed; if `x != y`, the stone of weight `x` is destroyed and the heavier becomes `y - x`. Continue until at most one stone remains. Return the weight of the last remaining stone, or `0` if none remain. Example: `stones = [2,7,4,1,8,1]` → `1`.

**Approach:** Repeatedly needing the two largest elements screams **max-heap**. Build a max-heap of all stones; each turn poll the top two, and if they differ push the difference back. When one or zero stones remain, return the top (or 0).
```java
import java.util.Collections;
import java.util.PriorityQueue;

class Solution {
    public int lastStoneWeight(int[] stones) {
        PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
        for (int s : stones) maxHeap.offer(s);

        while (maxHeap.size() > 1) {
            int y = maxHeap.poll(); // heaviest
            int x = maxHeap.poll(); // second heaviest
            if (y != x) {
                maxHeap.offer(y - x); // survivor goes back in
            }
        }
        return maxHeap.isEmpty() ? 0 : maxHeap.peek();
    }
}
```
**Complexity:** O(n log n) time (up to n smashes, each O(log n)), O(n) space.

**Key insight / follow-up:** "Repeatedly extract the extreme elements and reinsert a derived value" is the textbook signal for a heap. `Collections.reverseOrder()` flips Java's default min-heap into a max-heap cleanly (no overflow-prone subtraction comparator). A follow-up variant ("Last Stone Weight II") changes the rules entirely into a partition / subset-sum DP problem — a useful reminder that a small wording change can swap the whole technique.

---

### LC378 — Kth Smallest Element in a Sorted Matrix *(medium)*
**Problem:** Given an `n × n` matrix where each row and each column is sorted in ascending order, return the k-th smallest element in the matrix (k-th in sorted order, counting duplicates). Example: `matrix = [[1,5,9],[10,11,13],[12,13,15]]`, `k = 8` → `13`.

**Approach:** Binary-search on the **value range** `[matrix[0][0], matrix[n-1][n-1]]`. For a candidate `mid`, count how many elements are ≤ `mid` by walking from the bottom-left corner: move up when the current value exceeds `mid`, move right otherwise — each step is monotonic, giving an O(n) count. Shrink the value range until `lo == hi`, which lands on a value actually present in the matrix.
```java
class Solution {
    public int kthSmallest(int[][] matrix, int k) {
        int n = matrix.length;
        int lo = matrix[0][0], hi = matrix[n - 1][n - 1];
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (countLessEqual(matrix, mid) < k) {
                lo = mid + 1;   // too few <= mid: target is larger
            } else {
                hi = mid;       // enough <= mid: target is mid or smaller
            }
        }
        return lo; // converges on a value present in the matrix
    }

    // count elements <= target, walking from bottom-left in O(n)
    private int countLessEqual(int[][] matrix, int target) {
        int n = matrix.length;
        int row = n - 1, col = 0, count = 0;
        while (row >= 0 && col < n) {
            if (matrix[row][col] <= target) {
                count += row + 1; // whole column up to this row qualifies
                col++;            // move right
            } else {
                row--;            // move up
            }
        }
        return count;
    }
}
```
**Complexity:** O(n · log(max − min)) time, O(1) space.

**Key insight / follow-up:** Binary-searching the *value space* (not indices) beats the heap approach when n is large. The heap alternative — seed a min-heap with the first row/column and pop k times, pushing neighbours (like LC373) — is O(k log n) and easier to reason about, but uses O(n) space and degrades when k is near n². The bottom-left walk for `countLessEqual` exploits the row+column sorting to count in linear time, the crux of the binary-search solution.

---

### LC380 — Insert Delete GetRandom O(1) *(medium)*
**Problem:** Implement `RandomizedSet` supporting three operations, each in **average O(1)**: `boolean insert(int val)` (add `val`, return `false` if already present), `boolean remove(int val)` (remove `val`, return `false` if absent), and `int getRandom()` (return a uniformly random current element). At least one element exists when `getRandom` is called.

**Approach:** Pair a **dynamic array** (for O(1) random indexing) with a **hash map** `val → index` (for O(1) lookup). Insert appends to the array and records its index. The trick for O(1) remove: **swap the target with the last array element**, update that moved element's index in the map, then pop the last slot — avoiding the O(n) shift a normal middle-removal would cost.
```java
import java.util.*;

class RandomizedSet {
    private final List<Integer> values = new ArrayList<>();
    private final Map<Integer, Integer> indexOf = new HashMap<>(); // val -> index in values
    private final Random rand = new Random();

    public RandomizedSet() { }

    public boolean insert(int val) {
        if (indexOf.containsKey(val)) return false;
        indexOf.put(val, values.size());
        values.add(val);
        return true;
    }

    public boolean remove(int val) {
        Integer idx = indexOf.get(val);
        if (idx == null) return false;
        int lastIdx = values.size() - 1;
        int lastVal = values.get(lastIdx);
        // move the last element into the slot being vacated
        values.set(idx, lastVal);
        indexOf.put(lastVal, idx);
        // pop the now-duplicated tail
        values.remove(lastIdx);
        indexOf.remove(val);
        return true;
    }

    public int getRandom() {
        return values.get(rand.nextInt(values.size()));
    }
}
```
**Complexity:** O(1) average for all three operations; O(n) space.

**Key insight / follow-up:** `getRandom` in O(1) demands contiguous storage you can index by a random integer — a plain hash set can't do that. The **swap-with-last** removal is the linchpin: it keeps the array gap-free so indices stay valid for `getRandom`. Order matters when the element being removed *is* the last one (or when the set has one element) — overwriting then popping still works because the final `values.remove(lastIdx)` discards the stale copy. Follow-up LC381 allows duplicates: store a *set of indices* per value.

---

### LC460 — LFU Cache *(hard)*
**Problem:** Design a Least Frequently Used (LFU) cache with capacity `capacity`. `int get(int key)` returns the value or `-1` and increments the key's use-count. `void put(int key, int value)` inserts/updates and increments the count; when at capacity it evicts the key with the **smallest use-count**, breaking ties by **least recently used** among those. Both operations must run in average O(1). Example: with capacity 2, `put(1,1); put(2,2); get(1)→1; put(3,3)` evicts key 2 (count 1, vs key 1's count 2).

**Approach:** Three maps. `values: key → value`. `counts: key → frequency`. `lists: freq → LinkedHashSet<key>`, a per-frequency bucket that preserves insertion order so the *oldest* key at a frequency is found in O(1). Track `minFreq`, the lowest frequency currently present. **Touching** a key moves it from bucket `f` to bucket `f+1`; if bucket `minFreq` empties, `minFreq++`. Eviction removes the first (oldest) key in bucket `minFreq`.
```java
import java.util.*;

class LFUCache {
    private final int capacity;
    private int minFreq;
    private final Map<Integer, Integer> values = new HashMap<>();          // key -> value
    private final Map<Integer, Integer> counts = new HashMap<>();          // key -> frequency
    private final Map<Integer, LinkedHashSet<Integer>> lists = new HashMap<>(); // freq -> keys in LRU order

    public LFUCache(int capacity) {
        this.capacity = capacity;
        this.minFreq = 0;
    }

    public int get(int key) {
        if (!values.containsKey(key)) return -1;
        touch(key);                  // bump frequency
        return values.get(key);
    }

    public void put(int key, int value) {
        if (capacity == 0) return;
        if (values.containsKey(key)) {
            values.put(key, value);  // update + bump frequency
            touch(key);
            return;
        }
        if (values.size() >= capacity) {
            // evict the oldest key in the minFreq bucket
            LinkedHashSet<Integer> minList = lists.get(minFreq);
            int evict = minList.iterator().next();
            minList.remove(evict);
            values.remove(evict);
            counts.remove(evict);
        }
        values.put(key, value);
        counts.put(key, 1);
        lists.computeIfAbsent(1, f -> new LinkedHashSet<>()).add(key);
        minFreq = 1;                 // a fresh key always has frequency 1
    }

    // move key from its current frequency bucket to the next one
    private void touch(int key) {
        int freq = counts.get(key);
        counts.put(key, freq + 1);
        LinkedHashSet<Integer> oldList = lists.get(freq);
        oldList.remove(key);
        if (oldList.isEmpty()) {
            lists.remove(freq);
            if (minFreq == freq) minFreq++; // bucket drained: advance the floor
        }
        lists.computeIfAbsent(freq + 1, f -> new LinkedHashSet<>()).add(key);
    }
}
```
**Complexity:** O(1) average for `get` and `put`; O(capacity) space.

**Key insight / follow-up:** The combination "bucket per frequency + insertion-ordered set within each bucket" resolves the two-dimensional eviction rule — **frequency first, recency second** — in O(1). `LinkedHashSet` is doing double duty: O(1) membership *and* preserved order so the LRU tie-break is `iterator().next()`. Maintaining `minFreq` correctly is the subtle part: it resets to 1 on every fresh insert and only ever increments inside `touch` when the current minimum bucket empties.

---

### LC355 — Design Twitter *(medium)*
**Problem:** Design a simplified Twitter. `void postTweet(int userId, int tweetId)` composes a new tweet. `List<Integer> getNewsFeed(int userId)` returns the **10 most recent** tweet IDs in the user's news feed — tweets posted by the user themselves or by users they follow, most recent first. `void follow(int followerId, int followeeId)` and `void unfollow(int followerId, int followeeId)` manage the follow graph. A user implicitly follows themselves for the feed.

**Approach:** Store a global, monotonically increasing **timestamp** so tweets are totally orderable across users. Each user keeps a list of `(time, tweetId)`; the follow graph is a `userId → set of followees`. For the news feed, gather the candidate authors (self + followees), and **merge their tweet lists by recency using a max-heap on timestamp** — exactly LC23's k-way merge — stopping after 10 tweets.
```java
import java.util.*;

class Twitter {
    private int timestamp = 0;
    private final Map<Integer, List<int[]>> tweets = new HashMap<>();   // userId -> list of {time, tweetId}
    private final Map<Integer, Set<Integer>> following = new HashMap<>(); // userId -> followees

    public Twitter() { }

    public void postTweet(int userId, int tweetId) {
        tweets.computeIfAbsent(userId, u -> new ArrayList<>())
              .add(new int[]{timestamp++, tweetId});
    }

    public List<Integer> getNewsFeed(int userId) {
        // max-heap by timestamp (index 0 of each {time, tweetId})
        PriorityQueue<int[]> heap = new PriorityQueue<>((a, b) -> Integer.compare(b[0], a[0]));

        Set<Integer> authors = new HashSet<>(following.getOrDefault(userId, Set.of()));
        authors.add(userId); // a user sees their own tweets

        for (int author : authors) {
            List<int[]> userTweets = tweets.get(author);
            if (userTweets != null && !userTweets.isEmpty()) {
                // push only the latest tweet of each author; expand lazily
                heap.offer(new int[]{userTweets.size() - 1, author});
            }
        }

        List<Integer> feed = new ArrayList<>();
        while (!heap.isEmpty() && feed.size() < 10) {
            int[] cur = heap.poll();
            int idx = cur[0], author = cur[1];
            int[] tweet = tweets.get(author).get(idx);
            feed.add(tweet[1]);          // tweetId
            if (idx > 0) {
                heap.offer(new int[]{idx - 1, author}); // next-older tweet from same author
            }
        }
        return feed;
    }

    public void follow(int followerId, int followeeId) {
        following.computeIfAbsent(followerId, u -> new HashSet<>()).add(followeeId);
    }

    public void unfollow(int followerId, int followeeId) {
        Set<Integer> followees = following.get(followerId);
        if (followees != null) followees.remove(followeeId);
    }
}
```

> The heap above stores `{tweetIndex, author}` and dereferences `tweets.get(author).get(idx)` to read the timestamp, but for clarity the comparator should compare *timestamps*. The compact, unambiguous version pushes `{time, tweetId, author, idx}` arrays directly. Either works; the index-based form keeps the heap small (one entry per author until expanded).

**Complexity:** `postTweet`, `follow`, `unfollow` O(1); `getNewsFeed` O(f + 10 log f) where f is the number of followees (heap holds ≤ f entries, popped at most ~10 times plus re-pushes); space O(total tweets + follow edges).

**Key insight / follow-up:** The feed is a **k-way merge of per-user timelines by timestamp** — the same heap pattern as Merge k Sorted Lists, capped at 10 outputs. A global timestamp gives a clean cross-user ordering without comparing wall-clock times. The lazy expansion (push one tweet per author, then pull the next-older only when needed) keeps the heap at ≤ f entries instead of loading every tweet. Real systems precompute ("fan-out on write") feeds into each follower's inbox so reads are O(1), trading write cost for read speed.

---

*[← DSA bank index](README.md)*
