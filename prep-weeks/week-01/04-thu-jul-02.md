# Week 1 · Day 4 — Thursday Jul 02 — Hashing / Top-K + Collections Deep-Dive + Bit Manipulation

> The "data-structure internals" day. Two pointer/window patterns are warm from Mon–Wed; today you graduate to **frequency aggregation** (Top-K, consecutive sequences) and then go *under the hood* of the Collections framework — the single richest vein of senior Java interview questions. The bit-manipulation set at the end isn't filler: every HashMap internal you'll recite today (`h ^ (h >>> 16)`, `(n-1) & hash`, `hash & oldCapacity`) is the same muscle.

📌 **Study today:** Hashing / Top-K (LC 347 · 128) · Collections deep-dive (ArrayList/LinkedList, ConcurrentHashMap, fail-fast) · bit manipulation (LC 136 · 191 · 338 · 268) · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Top-K & Frequency Patterns (~2.5 h)

### Theory deep-dive

Frequency problems share one shape: **count occurrences into a `HashMap`, then do something order-sensitive with the counts.** The "something" decides your complexity:

- Need *all* items sorted by frequency → sort the entries: `O(n log n)`.
- Need only the *top k* → a **bounded min-heap** of size k: `O(n log k)`.
- Need top k but counts are bounded by n → **bucket sort by frequency**: `O(n)`.

The senior move is recognizing that frequency is itself an integer in `[0, n]`, so you can **index an array by frequency** and skip comparison sorting entirely. That O(n) bucket-sort answer is what separates a strong candidate from one who reaches for `PriorityQueue` reflexively.

A second family — **set-membership for O(n) ordering** (Longest Consecutive Sequence) — uses a `HashSet` to turn what looks like a sort (`O(n log n)`) into a single linear scan by only *starting* a count at sequence boundaries.

### Worked example — Top K Frequent Elements (LC 347), bucket sort

```java
// Java 17
import java.util.*;

class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        // 1) frequency map: O(n)
        Map<Integer, Integer> freq = new HashMap<>();
        for (int x : nums) freq.merge(x, 1, Integer::sum);

        // 2) buckets[f] = list of values seen exactly f times.
        //    Max possible frequency is nums.length, so index 1..n.
        List<Integer>[] buckets = new List[nums.length + 1];
        for (var e : freq.entrySet()) {
            int f = e.getValue();
            if (buckets[f] == null) buckets[f] = new ArrayList<>();
            buckets[f].add(e.getKey());
        }

        // 3) scan high frequency -> low, collect until we have k: O(n)
        int[] result = new int[k];
        int idx = 0;
        for (int f = buckets.length - 1; f >= 1 && idx < k; f--) {
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

**Complexity:** time `O(n)` — every step is linear in the input; space `O(n)` for the map + buckets. Compare the heap version: `O(n log k)` time, `O(k)` space. **Use bucket sort when k ≈ n; use the heap when k ≪ n** (a size-3 heap over 10M elements is far cheaper than allocating a 10M-slot bucket array).

`freq.merge(x, 1, Integer::sum)` is the idiomatic Java-8+ counting one-liner: insert 1 if absent, else apply the remapping function. Cleaner than `getOrDefault` + `put`.

### Practice set

| # | Problem | Approach | Key insight | Complexity | Follow-up |
|---|---------|----------|-------------|------------|-----------|
| LC 347 | Top K Frequent Elements | Bucket sort OR min-heap | Frequency ∈ [0,n] → index an array by it | `O(n)` / `O(n)` | "Now stream the data, can't sort." → min-heap of size k |
| LC 128 | Longest Consecutive Sequence | HashSet, start only at boundaries | Only count from `num` if `num-1 ∉ set` | `O(n)` / `O(n)` | "Prove it's O(n) not O(n²)." → each number visited ≤ twice |

#### LC 128 — Longest Consecutive Sequence, the O(n) trick

```java
public int longestConsecutive(int[] nums) {
    Set<Integer> set = new HashSet<>();
    for (int n : nums) set.add(n);          // dedup + O(1) membership

    int best = 0;
    for (int n : set) {
        if (set.contains(n - 1)) continue;  // n is NOT a sequence start; skip
        int len = 1, cur = n;
        while (set.contains(cur + 1)) { cur++; len++; }  // walk the run
        best = Math.max(best, len);
    }
    return best;
}
```

The `continue` guard is the whole trick. Without it you'd re-walk every run from every member → `O(n²)`. With it, the inner `while` only ever runs for the *start* of each run, so across the whole loop every element is touched at most twice (once as a non-start skip, once inside one run). That's the **amortized O(n)** argument interviewers want you to articulate, not just assert.

---

## Block B — Collections Framework Deep-Dive (~2 h)

This is the densest interview block of Week 1. Goal: explain *why each structure is built the way it is*, not just its Big-O. Cross-reference [07-collections.md](../../07-collections.md) and [06-concurrency-and-collections.md](../../06-concurrency-and-collections.md).

### ArrayList — the contiguous default

- Backed by `Object[] elementData`; **default capacity 10**, but allocation is lazy — an empty `new ArrayList<>()` starts with a shared empty array and grows to 10 on first `add`.
- Growth: `newCap = oldCap + (oldCap >> 1)` → **1.5×**, vs `Vector`'s 2×. The 1.5× factor wastes less memory on average while keeping `add` amortized `O(1)`.
- `add(index, e)` does `System.arraycopy` to shift the tail — `O(n)` but cache-friendly and JIT-vectorizable, so in practice it beats `LinkedList` for lists up to ~thousands of elements.
- `trimToSize()` after a bulk load reclaims the slack from over-allocation.

### LinkedList — almost never the right default

- Doubly-linked `Node{item, prev, next}`; roughly **48 bytes/node** (object header + 3 references + padding) vs ~8 bytes per array slot. Heavy memory + pointer-chasing kills CPU cache locality.
- `add(0, e)` / `addFirst` is `O(1)` rewire, but `get(i)` is `O(n)` (it walks from whichever end is nearer).
- **Rule: default to `ArrayList`; use `ArrayDeque` for stack/queue; reach for `LinkedList` only when a profiler tells you to.** "I'll use LinkedList for frequent inserts" is a junior tell — measure first; cache effects usually win for ArrayList.

```java
// ArrayDeque as a stack (preferred over java.util.Stack which extends Vector — synchronized + LSP violation)
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1); stack.push(2);   // 2,1
int top = stack.pop();          // 2
```

### ConcurrentHashMap (Java 8+) — lock striping done right

- Empty bucket → **CAS** insert (lock-free fast path); non-empty bucket → `synchronized` on the **bin head only**, so different buckets never contend.
- **Reads are lock-free** — nodes' `val`/`next` are `volatile`, so a reader sees a consistent snapshot without locking.
- Resize is **cooperative**: a thread that finds a bucket already being moved (a `ForwardingNode`) helps migrate instead of blocking.
- `size()` uses a striped counter (`baseCount` + `CounterCell[]`, LongAdder-style) to avoid a single hot contended counter.
- **No null keys or values.** In a concurrent map, `get(k) == null` is ambiguous (key absent vs mapped-to-null), and you can't disambiguate with `containsKey` without a TOCTOU race. So nulls are simply banned.
- Java 5–7 trivia: the old design used a fixed `Segment[]` of striped locks; Java 8 dropped segments for per-bin locking + trees.
- vs `Hashtable`: a single full-map `synchronized` lock → throughput bottleneck. vs `Collections.synchronizedMap`: also a global lock, plus iteration needs manual external synchronization.

### Fail-fast vs fail-safe iterators

- **Fail-fast** (`ArrayList`, `HashMap`, `TreeMap`): the iterator snapshots `modCount` at creation; each `next()` checks `modCount == expectedModCount` and throws `ConcurrentModificationException` if a *structural* change happened. It's **best-effort** misuse detection, not a guarantee.
- **`set()` does NOT bump `modCount`** — replacing an element's value is a content change, not a structural one, so it never triggers CME.
- **Fail-safe / weakly-consistent**: `CopyOnWriteArrayList` (iterates a snapshot), `ConcurrentHashMap` (weakly consistent — reflects some but not necessarily all concurrent updates, never throws CME).

```java
List<String> list = new ArrayList<>(List.of("a","b","c"));
// for (String s : list) list.remove(s);   // ALWAYS throws ConcurrentModificationException
list.removeIf("b"::equals);                 // correct
// or: Iterator<String> it = list.iterator(); while(it.hasNext()){ it.next(); it.remove(); }
```

### Smart360 / Deep Fathom / WebX tie-in

- **Smart360** Redis cache uses `String` keys; `String.hashCode()` is a polynomial rolling hash *cached after first call* — which is exactly why `String` is the ideal HashMap/cache key (immutable + memoized hash).
- **Smart360 Notification** service is multi-threaded over a shared lookup → `ConcurrentHashMap` (CAS path), never `Hashtable`.
- **WebX** provider registry is read-mostly under concurrent request load → `ConcurrentHashMap`'s lock-free reads matter.

---

## Block C — Bit Manipulation: 4 Canonical Easy (~1.5 h)

Drill each cold, ~10 min. The XOR identities to internalize: `a ^ a = 0`, `a ^ 0 = a`, XOR is commutative & associative (order-independent).

### 1. Single Number (LC 136)

```java
public int singleNumber(int[] nums) {
    int result = 0;
    for (int n : nums) result ^= n;   // pairs cancel to 0, the lone number survives
    return result;
}   // O(n) time, O(1) space
```
**Why XOR:** every duplicate cancels (`x ^ x = 0`); XOR-ing them in any order leaves only the unpaired value. No extra space, unlike a HashSet.

### 2. Number of 1 Bits (LC 191)

```java
public int hammingWeight(int n) {
    int count = 0;
    while (n != 0) { n &= (n - 1); count++; }  // clears the lowest set bit each pass
    return count;
}   // loops once per set bit; Integer.bitCount(n) is the JDK intrinsic
```
**Why `n &= n-1`:** subtracting 1 flips the lowest set bit to 0 and all bits below it to 1; AND-ing clears exactly that lowest set bit. Loop runs k times for k set bits — faster than checking all 32 bits.

### 3. Counting Bits (LC 338)

```java
public int[] countBits(int n) {
    int[] bits = new int[n + 1];
    for (int i = 1; i <= n; i++)
        bits[i] = bits[i >> 1] + (i & 1);   // popcount(i) = popcount(i/2) + lowbit(i)
    return bits;
}   // O(n) for the whole 0..n table, O(1) work per entry
```
**Why it's DP:** `i >> 1` drops the last bit; its popcount is already computed; add back the bit you dropped (`i & 1`). Each answer reuses a smaller, already-solved subproblem.

### 4. Missing Number (LC 268)

```java
public int missingNumber(int[] nums) {
    int xor = nums.length;               // start with n (the top index)
    for (int i = 0; i < nums.length; i++)
        xor ^= i ^ nums[i];              // XOR all indices 0..n-1 and all values
    return xor;                          // every present number cancels its index; missing survives
}   // O(n) time, O(1) space
```
**Why XOR over Gauss:** `n(n+1)/2 - sum` works but can **overflow** for large n; XOR never overflows and needs no `long`.

---

## 💻 Practice coding questions

1. **Top K Frequent Elements (LC 347)** — bucket sort, code above. *Follow-up:* return them in ascending frequency order → reverse the scan.
2. **Top K Frequent — heap version** — `PriorityQueue<int[]>` ordered by count, capped at size k, poll the smallest when it overflows. `O(n log k)`.
3. **Longest Consecutive Sequence (LC 128)** — HashSet boundary trick, code above.
4. **K Closest Points to Origin (LC 973)** — max-heap of size k on squared distance (no `sqrt` needed). `O(n log k)`.
5. **Single Number (LC 136)** — XOR fold.
6. **Single Number II (LC 137)** — every element appears 3× except one; bit-by-bit `count % 3`, or the `ones/twos` two-mask trick.
7. **Number of 1 Bits (LC 191)** — `n &= n-1`.
8. **Counting Bits (LC 338)** — DP `bits[i>>1] + (i&1)`.
9. **Missing Number (LC 268)** — XOR indices and values.
10. **Sort Characters By Frequency (LC 451)** — frequency map → bucket sort by count → rebuild string. Same bucket pattern as LC 347.
11. **Top K Frequent Words (LC 692)** — heap with a tie-break comparator (count desc, then lexicographic asc) — tests custom `Comparator` (Friday's topic).
12. **Reverse Bits (LC 190)** — shift result left, OR in `n & 1`, shift n right, ×32.
13. **Power of Two (LC 231)** — `n > 0 && (n & (n-1)) == 0` (exactly one set bit).
14. **Fix-the-bug:** `for (String s : list) list.remove(s);` throws CME — give three fixes (`removeIf`, `Iterator.remove()`, collect-then-`removeAll`).

---

## 🎤 Interview questions

1. **Top K Frequent in O(n) — how?** Count into a HashMap, then bucket sort: an array indexed by frequency, scan high→low. Avoids the `O(n log n)` of comparison sorting.
2. **When heap over bucket sort for Top-K?** When `k ≪ n` and/or you can't materialize an n-sized bucket array (e.g. streaming) — a size-k min-heap is `O(n log k)` time, `O(k)` space.
3. **Longest Consecutive Sequence in O(n) — why isn't it O(n²)?** Inner walk only starts at run boundaries (`num-1 ∉ set`); each element is touched at most twice across the whole loop → amortized O(n).
4. **Why is `String` the ideal HashMap key?** Immutable (hash can't drift) and its `hashCode` is **cached after first computation**.
5. **ArrayList growth — why 1.5× not 2×?** Less average wasted memory; amortized `O(1)` `add` is preserved either way. Vector's 2× over-allocates.
6. **ArrayList vs LinkedList for frequent `add(0,e)`?** Surprisingly ArrayList often wins below ~thousands of elements — contiguous memory → cache prefetch beats LinkedList's pointer chasing, despite the `O(n)` arraycopy.
7. **Why `ArrayDeque` over `java.util.Stack`?** `Stack extends Vector` (synchronized, slow) and violates LSP (`add(0,e)` breaks stack semantics). `ArrayDeque` is the modern stack/queue.
8. **ConcurrentHashMap put internals?** Empty bucket → CAS (lock-free); non-empty → `synchronized` on the bin head only; reads lock-free via `volatile`; resize is cooperative via `ForwardingNode`.
9. **Why no null keys/values in ConcurrentHashMap?** `get()==null` is ambiguous (absent vs mapped-to-null) and you can't disambiguate without a TOCTOU race against `containsKey`.
10. **CHM vs Hashtable vs synchronizedMap?** CHM = fine-grained per-bin locking; Hashtable = single full-map lock; synchronizedMap = global lock + manual sync for iteration. CHM wins on throughput.
11. **What is fail-fast and is it a guarantee?** Iterator snapshots `modCount`, throws CME on structural change during iteration. It's **best-effort** misuse detection, not a thread-safety guarantee.
12. **Does `list.set(i, x)` during iteration throw CME?** No — `set` is a content change; `modCount` only tracks structural changes.
13. **`for (s : list) list.remove(s)` — three fixes.** `removeIf`, `Iterator.remove()`, or collect-then-`removeAll` / filter to a new list.
14. **Weakly-consistent iterator meaning?** (ConcurrentHashMap) reflects the state at/after creation, may or may not show concurrent updates, never throws CME.
15. **Why XOR for Single Number?** `x ^ x = 0`, `x ^ 0 = x`, order-independent → all pairs cancel, the lone element remains. `O(1)` space vs a HashSet.
16. **What does `n & (n-1)` do?** Clears the lowest set bit. Used to count set bits in O(popcount) and to test power-of-two (`(n & (n-1)) == 0`).
17. **Counting Bits without `Integer.bitCount`?** DP: `bits[i] = bits[i>>1] + (i&1)`.
18. **Missing Number — XOR vs Gauss sum?** XOR avoids the integer overflow that `n(n+1)/2 - sum` risks for large n.
19. **Where do these bit tricks show up in the JDK?** HashMap: `h ^ (h >>> 16)` spreads high bits; `(n-1) & hash` is modulo-by-power-of-two; `hash & oldCapacity` is the resize split bit.
20. **`merge` vs `getOrDefault` for counting?** `map.merge(k, 1, Integer::sum)` is one atomic-ish call (in CHM it *is* atomic) and clearer than read-modify-write with `getOrDefault`.

---

## ✅ Self-check

1. Code Top K Frequent two ways (bucket O(n), heap O(n log k)) and state when each wins.
2. Why does ArrayList beat LinkedList on small lists even for `add(0,e)`? *(Contiguous memory → CPU cache prefetch dominates pointer-chasing.)*
3. Show the `modCount` fail-fast mechanism in a snippet; does `set()` throw? *(No — it's a content change, not structural.)*
4. Explain why ConcurrentHashMap forbids nulls in one sentence. *(`get()==null` is ambiguous and `containsKey` disambiguation is a TOCTOU race.)*
5. Solve all 4 bit problems cold in ≤ 10 min each and say *why XOR / why `n & (n-1)`* aloud.
6. Prove Longest Consecutive Sequence is O(n), not O(n²).

---

*Nav: ← [Wed Jul 01](03-wed-jul-01.md) · [Week 1](README.md) · [Fri Jul 03](05-fri-jul-03.md) →*
