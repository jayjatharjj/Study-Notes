# Week 1 · Day 2 — Tue Jun 30 — Sliding Window + HashMap Internals

> A sliding window turns "examine every subarray" (O(n²)) into "grow and shrink one window" (O(n)) — and HashMap internals are the single most-asked Java-fundamentals topic in product-company interviews.

📌 **Study today:** Sliding Window — variable size (LC 3 · 209 · 424 · 643) · HashMap internals (buckets, treeify, resize, the bit-spread) · subarray-sum + advanced sliding window · ⏱ ~6 hr (Block A ~2.5h · Block B ~2h · Block C ~1.5h)

---

## Block A — DSA: Sliding Window (variable size)

### Theory — why the pattern works

A **sliding window** maintains a contiguous range `[left, right]` and processes the array by **expanding `right`** one step at a time and **shrinking `left`** whenever the window violates an invariant. Each element enters the window once (when `right` passes it) and leaves at most once (when `left` passes it), so the total work is **2n pointer moves → O(n)** even though there are two nested-looking loops.

The whole pattern is governed by one question: **"What invariant must my window maintain?"**

- **Fixed-size window** (LC 643): the window is always exactly `k` wide. Add the entering element, remove the leaving element, record the metric. No conditional shrink.
- **Variable-size, longest** (LC 3, 424): expand greedily; when the invariant breaks, shrink from the left *just enough* to restore it; record the max length seen.
- **Variable-size, shortest** (LC 209, 76): expand until the invariant is *satisfied*, then shrink from the left while it *stays* satisfied, recording the min length.

**The canonical template:**

```java
int left = 0;
for (int right = 0; right < n; right++) {
    // 1. include nums[right] in window state
    while (/* window invariant violated */) {
        // 2. remove nums[left] from window state
        left++;
    }
    // 3. window [left, right] is now valid — record result
}
```

**When to reach for it:** contiguous subarray/substring problems asking for a longest/shortest/optimal range under a constraint (sum, distinct chars, frequency). If order doesn't matter or the subarray needn't be contiguous, it's *not* a window problem.

**Complexity:** time O(n) (each index enters/exits once); space O(1) to O(k) depending on what window state you track (a frequency map of an alphabet is O(1) if the alphabet is bounded, e.g. `int[26]`).

**Common pitfalls:**
- Shrinking with `if` when you need `while` (or vice versa) — "shortest" problems shrink with `while`; "longest with one violation budget" may shrink with `if`.
- In LC 3, doing `left++` on a duplicate instead of `left = max(left, lastSeen + 1)` — incrementing re-processes characters and is wrong when the duplicate is *behind* `left`.
- Forgetting to update the result at the *right* point (inside vs after the shrink loop).
- Recomputing an expensive window metric on every shrink (LC 424 `maxFreq` — see Block C reasoning).

### Worked example 1 — Longest Substring Without Repeating Characters (LC 3)

```java
// Longest substring with all-distinct chars. O(n) time, O(min(n, charset)) space.
int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> lastSeen = new HashMap<>(); // char -> last index it appeared
    int left = 0, best = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (lastSeen.containsKey(c) && lastSeen.get(c) >= left) {
            // duplicate is INSIDE the window: jump left past it
            left = lastSeen.get(c) + 1;
        }
        lastSeen.put(c, right);
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

**Why `left = lastSeen.get(c) + 1`, not `left++`:** if the same char appeared at index 2 and we're at index 9 with `left = 0`, we must jump `left` to 3 in one move. Incrementing would leave duplicates in the window. The guard `lastSeen.get(c) >= left` matters: a stale earlier occurrence that's already *behind* `left` must be ignored (don't move `left` backward).

**Complexity:** `right` advances n times; `left` only ever increases → O(n). Map holds at most the charset size → O(min(n, |Σ|)).

### Worked example 2 — Minimum Size Subarray Sum (LC 209)

```java
// Smallest length of a contiguous subarray with sum >= target. O(n)/O(1).
int minSubArrayLen(int target, int[] nums) {
    int left = 0, sum = 0, best = Integer.MAX_VALUE;
    for (int right = 0; right < nums.length; right++) {
        sum += nums[right];                 // expand
        while (sum >= target) {             // shrink while STILL valid → finds minimum
            best = Math.min(best, right - left + 1);
            sum -= nums[left];
            left++;
        }
    }
    return best == Integer.MAX_VALUE ? 0 : best;
}
```

**Why O(n) despite the nested `while`:** `left` is incremented at most n times total across the entire run (it never resets), so the inner loop's *aggregate* work is bounded by n. Amortized, every element is added once and subtracted once. **This "two loops but O(n)" argument is a classic interview probe** — be ready to say "each element enters and leaves the window at most once."

### Practice set

- **Maximum Average Subarray I (LC 643)** *(Easy)* · fixed-size window of width `k`; slide by adding `nums[right]` and subtracting `nums[right-k]`; track max sum. · **Key insight:** fixed windows need no conditional shrink — just add-one/remove-one. · O(n)/O(1). · *Follow-up:* "Return the subarray, not the average." → also track the window's start index when the max updates.

- **Longest Substring Without Repeating Characters (LC 3)** *(Medium)* · last-seen map; jump `left = max(left, lastSeen[c] + 1)`. · **Key insight:** jump, don't crawl, on a duplicate. · O(n)/O(|Σ|). · *Follow-up:* "At most `k` distinct characters (LC 340)?" → shrink while `map.size() > k`.

- **Minimum Size Subarray Sum (LC 209)** *(Medium)* · expand; shrink while `sum >= target`, tracking min length. · **Key insight:** "shortest" ⇒ shrink with `while` while the invariant still holds. · O(n)/O(1). · *Follow-up:* "Negative numbers allowed?" → window monotonicity breaks; use prefix sum + a monotonic deque or binary search instead.

- **Longest Repeating Character Replacement (LC 424)** *(Medium)* · invariant `(windowSize - maxFreq) <= k`; track `maxFreq` but never shrink it. · **Key insight:** you seek the *longest* valid window, so a stale (too-high) `maxFreq` only ever blocks *shorter* windows you don't care about — never decreasing it stays correct and keeps it O(n) instead of O(26n). · O(n)/O(1). · *Follow-up:* "Why is not recomputing maxFreq still correct?" → see the dedicated explanation below.

- **Find All Anagrams in a String (LC 438)** *(Medium)* · fixed window of `p.length()`; compare an `int[26]` window-count to `p`'s count. · **Key insight:** anagram = identical frequency vector; comparing two 26-arrays is O(26)=O(1). · O(n)/O(1). · *Follow-up:* "Avoid re-comparing arrays each slide." → maintain a `matches` counter that increments/decrements only on the two changing buckets.

- **Permutation in String (LC 567)** *(Medium)* · same fixed-window frequency match as LC 438, return boolean. · **Key insight:** identical machinery; the only change is the return condition. · O(n)/O(1). · *Follow-up:* "Case-insensitive / Unicode?" → switch from `int[26]` to a `HashMap<Character,Integer>`.

---

## Block B — Core Java: HashMap internals

### Theory — the structure

A `HashMap<K,V>` is an **array of buckets**: `Node<K,V>[] table`. Each `Node` holds `hash`, `key`, `value`, and `next` (a singly-linked chain). Defaults:

- **Initial capacity 16** (table length; always a power of 2).
- **Load factor 0.75** → **resize threshold = capacity × 0.75** = 12 for the default. Insert that makes `size > threshold` triggers a resize.

**Finding the bucket** for a key is two steps:

```java
// 1. spread the hash so high bits influence the low-order index
static int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
// 2. bucket index = (table.length - 1) & hash   // == hash % length because length is 2^n
```

**Why the bit-spread `h ^ (h >>> 16)`:** the index is computed as `(n-1) & hash`, which for a small table (say n=16, mask `0b1111`) uses only the **low 4 bits** of the hash. Many real hash codes differ mainly in their *high* bits (e.g. `Integer.hashCode()` is the value itself; small object addresses). XORing the top 16 bits down into the low 16 mixes high-bit entropy into the bits that actually select the bucket — cheaply spreading keys and cutting collisions.

**Why capacity must be a power of 2:** `(n-1) & hash` equals `hash % n` *only* when `n` is a power of 2 (the mask `n-1` is all-ones). That's why even `new HashMap<>(1000)` rounds *up* to 1024.

### Collisions: chaining → treeification

Keys whose spread hashes land in the same bucket form a chain. As the chain grows:

- It **converts to a red-black tree** when the bucket reaches **8 nodes AND the table capacity is ≥ 64**. If capacity is below 64, the map **resizes instead** (cheaper than treeifying a small table, and resizing usually disperses the chain).
- This turns worst-case bucket lookup from **O(n) → O(log n)** under heavy collision (e.g. an adversary feeding colliding keys — a DoS vector that treeification mitigates).
- It **un-treeifies back to a list at 6 nodes** (after a resize/removal). The 8/6 gap is a **hysteresis band** preventing thrashing between list and tree around the threshold.

> Why 8? The Poisson distribution: with a good hash and load factor 0.75, the probability that a given bucket reaches 8 entries is < 1e-7 — treeification is a rare safety net, not the normal path. The normal path is short chains (0, 1, or 2 nodes).

### Resize: double + the Java 8 split trick

When `size > threshold`, capacity **doubles** (16 → 32 → ...). Pre-Java-8 this rehashed every key (`hash % newCap`). Java 8 uses a smarter split:

- For each entry in old bucket `i`, look at one bit: **`hash & oldCapacity`**.
- If it's **0**, the entry stays at index `i` in the new table.
- If it's **1**, it moves to index `i + oldCapacity`.

Because the new mask is just the old mask with one extra high bit, every entry either stays or shifts by exactly `oldCapacity` — no full hash recomputation, and each old bucket splits cleanly into two new buckets (the "lo" and "hi" lists). This makes resize O(n) with minimal work per node.

**Pre-sizing to avoid resizes:** if you know you'll insert ~`N` entries, construct with `new HashMap<>((int)(N / 0.75f) + 1)` so it doesn't grow under you. The constructor rounds the requested capacity up to the next power of 2.

### Thread safety

- `HashMap` is **not thread-safe**. Concurrent writes can corrupt chains; in Java 7 a concurrent resize could create a cycle and spin a CPU at 100% (the infamous infinite-loop bug). Java 8 fixed the loop but concurrent use is still unsafe (lost updates, lost data).
- `ConcurrentHashMap` (Java 8+): empty bucket → lock-free **CAS** insert; non-empty bucket → `synchronized` on **just that bucket's head node**; reads are lock-free via `volatile`. Far higher throughput than the alternatives. (Full treatment Thursday.)
- `Collections.synchronizedMap(...)` and `Hashtable` use a **single global lock** over the whole map — avoid in hot paths; they serialize every operation.

### Code — observing the internals

```java
Map<String, Integer> m = new HashMap<>();
m.put("name", 1);   // hash("name") -> spread -> (16-1) & spread -> bucket -> walk chain -> insert
m.put("name", 2);   // same key: equals() match in the chain -> value REPLACED, size unchanged

// Pre-size for 10,000 entries to avoid all resizes:
Map<String, byte[]> cache = new HashMap<>((int)(10_000 / 0.75f) + 1); // -> capacity 16384
```

### Gotchas

- A **bad `hashCode`** (e.g. constant) collapses everything into one bucket → O(n)/O(log n) — the map "works" but is slow; profile under load.
- **Mutating a key's hashCode field after insertion** (yesterday's ghost-present bug) applies to map keys too.
- **Capacity ≠ size.** Capacity is the table length (power of 2); size is the entry count; threshold is `capacity * loadFactor`.

### Production tie-in — Smart360 Redis keys

Smart360's Redis cache uses `String` keys (e.g. `report:{orgId}:{reportId}`). `String.hashCode()` is a **polynomial rolling hash** (`s[0]*31^(n-1) + ... + s[n-1]`), **cached after first computation** (stored in the `String`'s `hash` field), and the `String` is immutable — so the hash never drifts. That immutability + caching is exactly why `String` is the canonical HashMap (and cache) key. The same reasoning applies to the in-process Caffeine/Guava caches sitting in front of Redis.

---

## Block C — subarray-sum + advanced sliding window

### Subarray Sum Equals K (LC 560) — prefix sum meets HashMap

This is the bridge from Monday's prefix-sum intro into Wednesday's set. We want the *count* of contiguous subarrays summing to exactly `k`.

```java
// Count subarrays with sum == k. O(n)/O(n).
int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> prefixCount = new HashMap<>();
    prefixCount.put(0, 1);          // SEED: a prefix sum of 0 occurs once (the empty prefix)
    int prefix = 0, count = 0;
    for (int x : nums) {
        prefix += x;
        // a subarray ending here sums to k iff some earlier prefix == prefix - k
        count += prefixCount.getOrDefault(prefix - k, 0);
        prefixCount.merge(prefix, 1, Integer::sum);
    }
    return count;
}
```

**Why seed `map.put(0, 1)`:** a subarray that starts at index 0 has no "earlier prefix" to subtract — its complement is the empty prefix whose sum is 0. Without the seed you'd miss every subarray beginning at index 0. **This single line is the most-asked detail of LC 560** — be able to explain it cold.

**Why this isn't a sliding window:** `nums` may contain negatives, so growing the window doesn't monotonically grow the sum — you can't shrink-when-too-big. Prefix-sum + hashing handles negatives; sliding windows generally require non-negative values.

### Why you must NOT shrink `maxFreq` in LC 424

Longest Repeating Character Replacement maintains `(windowSize - maxFreq) <= k` (replace the non-majority chars). The trick: when you shrink `left`, you do **not** recompute `maxFreq` downward. Reasoning:

- You're searching for the **longest** valid window. The answer (`best`) only ever increases.
- `maxFreq` only matters when it lets the window *grow*. A stale, too-high `maxFreq` would at worst permit a window that's actually invalid — but that window is **no longer than `best`**, so it can't change the answer.
- Recomputing `maxFreq` on every shrink would cost O(26) per step → O(26n). Keeping the historical max keeps it O(n).

```java
int characterReplacement(String s, int k) {
    int[] freq = new int[26];
    int left = 0, maxFreq = 0, best = 0;
    for (int right = 0; right < s.length(); right++) {
        maxFreq = Math.max(maxFreq, ++freq[s.charAt(right) - 'A']);
        if ((right - left + 1) - maxFreq > k) {   // window invalid: slide (no shrink of maxFreq)
            freq[s.charAt(left) - 'A']--;
            left++;
        }
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

Note the `if` (not `while`): the window grows by one each step and shrinks by at most one, so its size never decreases — it just stops growing once invalid. The window "slides" rather than truly shrinking.

### Stretch — Minimum Window Substring (LC 76) preview

The hardest sliding window (full timed attempt Saturday). Two frequency maps and a **`have` vs `need`** counter:

- `need` = number of *distinct* required chars; `have` = how many of them are currently satisfied in the window.
- Expand `right`; when a char's window-count reaches its required count, `have++`.
- When `have == need`, the window is valid → shrink `left` while still valid, recording the minimum window.

This `have/need` counter (rather than comparing whole maps each step) is what keeps it O(n + m). Internalize the shape today; code it timed on Saturday.

---

## 💻 Practice coding questions

1. **Maximum Average Subarray I (LC 643)** — fixed window `k`; slide-add/slide-subtract; track max sum. O(n)/O(1).
2. **Longest Substring Without Repeating Characters (LC 3)** — last-seen map, jump `left`. O(n)/O(|Σ|).
3. **Minimum Size Subarray Sum (LC 209)** — expand, shrink while `sum >= target`, min length. O(n)/O(1).
4. **Longest Repeating Character Replacement (LC 424)** — `(size - maxFreq) <= k`, don't shrink `maxFreq`. O(n)/O(1).
5. **Find All Anagrams in a String (LC 438)** — fixed window, `int[26]` compare. O(n)/O(1).
6. **Permutation in String (LC 567)** — fixed window frequency match → boolean. O(n)/O(1).
7. **Subarray Sum Equals K (LC 560)** — prefix sum + HashMap, seed `{0:1}`. O(n)/O(n).
8. **Longest Substring with At Most K Distinct Characters (LC 340)** — shrink while `map.size() > k`. O(n)/O(k).
9. **Fruit Into Baskets (LC 904)** — at-most-2-distinct window (LC 340 with k=2). O(n)/O(1).
10. **Max Consecutive Ones III (LC 1004)** — longest window with ≤ k zeros (424-flavored). O(n)/O(1).
11. **Minimum Window Substring (LC 76)** — two maps + `have/need`; expand then shrink. O(n+m)/O(|Σ|).
12. **Sliding Window Maximum (LC 239) — preview** — monotonic deque of indices in decreasing value. O(n)/O(k). (Full on Saturday.)
13. **Count Number of Nice Subarrays (LC 1248)** — prefix-count of odd numbers; same `{0:1}`-seed trick as LC 560. O(n)/O(n).
14. **Binary Subarrays With Sum (LC 930)** — prefix-sum count, or "at most" window difference. O(n)/O(n).

---

## 🎤 Interview questions

1. **"Why is the sliding window O(n) when it has two nested loops?"** — `left` and `right` each advance at most `n` times across the whole run and never reset; total pointer moves ≤ 2n. Each element enters and leaves the window at most once.
2. **"In LC 3, why `left = max(left, lastSeen + 1)` instead of `left++`?"** — A duplicate may be far behind `right`; you must jump `left` past the prior occurrence in one move. The `max` guards against moving `left` backward when the duplicate is already outside the window.
3. **"Walk me through `hashMap.put(\"name\", \"Jay\")`."** — Compute `key.hashCode()`, spread with `h ^ (h >>> 16)`, index = `(n-1) & spread`, walk the bucket chain calling `equals`; if a match, replace the value; else append (treeify if chain ≥ 8 and capacity ≥ 64); finally if `size > threshold`, resize.
4. **"Why the `h ^ (h >>> 16)` bit-spread?"** — The index uses only the low bits (`(n-1) & hash`); many hash codes vary mostly in high bits. XORing high bits down mixes their entropy into the index bits, reducing collisions cheaply.
5. **"Why must capacity be a power of 2?"** — So `(n-1) & hash` equals `hash % n` — the mask `n-1` is all-ones only when `n = 2^k`. This makes bucket selection a single AND instead of a modulo.
6. **"When does a bucket treeify, and what does it fix?"** — At chain length 8 *and* table capacity ≥ 64; below capacity 64 it resizes instead. It bounds worst-case bucket lookup at O(log n) (vs O(n)), mitigating hash-collision DoS. Un-treeifies at 6 (hysteresis).
7. **"Capacity 16, 10 entries — does the 11th insert resize?"** — No. Threshold is `16 * 0.75 = 12`; resize happens when size *exceeds* 12, i.e. on the 13th insert (→ capacity 32).
8. **"Initial capacity to hold 1000 entries without resizing?"** — `1000 / 0.75 ≈ 1334` → next power of 2 = **2048**. (Or construct with `new HashMap<>(1334)`; it rounds up.)
9. **"How does Java 8 resize without full rehashing?"** — Each entry's `hash & oldCapacity` bit decides: 0 → stays at index `i`; 1 → moves to `i + oldCapacity`. Old buckets split into a "lo" and "hi" list with no hash recomputation.
10. **"Two lookups for the same key return different results — most likely cause?"** — The key's `hashCode` depends on a mutable field that changed between insert and lookup, so it now hashes to a different bucket (ghost-present).
11. **"Why is `HashMap` unsafe under concurrency, and what's the Java 7 horror story?"** — Concurrent writes corrupt chains; in Java 7 a concurrent resize could form a cycle in a bucket and spin a thread at 100% CPU forever. Java 8 changed resize to preserve order and avoid the loop, but concurrent use is still unsafe.
12. **"`ConcurrentHashMap` vs `Hashtable` vs `synchronizedMap`?"** — CHM: per-bucket `synchronized` + CAS, lock-free reads, high throughput. Hashtable / synchronizedMap: single global lock serializing all ops. Prefer CHM in hot paths.
13. **"Why is `String` the ideal HashMap key?"** — Immutable (hash can't drift) and its `hashCode` is cached after first computation.
14. **"What's the difference between capacity, size, and threshold?"** — Capacity = table length (power of 2); size = number of entries; threshold = `capacity × loadFactor`, the size at which the table doubles.
15. **"Why is LC 560 not solvable with a sliding window?"** — With negative numbers, expanding the window doesn't monotonically increase the sum, so the shrink-when-too-big invariant fails. Prefix sum + hashing handles negatives.
16. **"Explain the `{0:1}` seed in LC 560."** — A subarray starting at index 0 has the empty prefix (sum 0) as its complement; without seeding `prefixCount.put(0,1)` you'd miss every subarray that begins at index 0.
17. **"Why never shrink `maxFreq` in LC 424?"** — You want the longest window; a stale high `maxFreq` only ever permits windows no longer than the current best, so it can't corrupt the answer, and not recomputing keeps it O(n) instead of O(26n).
18. **"What's the load factor and why 0.75?"** — The fraction of capacity filled before resizing. 0.75 is the empirical space/time sweet spot: low enough that buckets stay short (Poisson keeps P(len ≥ 8) tiny), high enough to not waste much memory.

---

## ✅ Self-check

1. Write the sliding-window template from memory and label expand/shrink/record. *(expand `right`; `while` invariant violated, remove `left` and `left++`; record after shrink.)*
2. Draw HashMap internals from memory: buckets, chain, tree, plus the treeify threshold and load factor. *(array of `Node`, chain per bucket, RB-tree at len 8 & cap ≥ 64, load factor 0.75.)*
3. Two map lookups for the same key disagree — give the cause in one sentence. *(mutated `hashCode` field → different bucket.)*
4. Explain why Java 8 resize needs only one bit per entry. *(`hash & oldCapacity`: 0 stays, 1 moves to `i + oldCapacity`.)*
5. State the `{0:1}` seed reason for LC 560 in under 30 seconds. *(complement of an index-0 subarray is the empty prefix, sum 0.)*

---
*Nav: ← [prev](01-mon-jun-29.md) · [Week 1](README.md) · [next →](03-wed-jul-01.md)*
