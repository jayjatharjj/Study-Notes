# Week 4 · Day 4 — Thu Jul 23 — Heaps + System Design: Sharding & Replication + Design-DS

> The heap (priority queue) is the answer to a whole family of "k-th / top-k / streaming-order" questions that look like they need a full sort but don't — you only need *partial* order, and a size-bounded heap gives it in O(n log k). Today's four patterns — **top-K via a size-k min-heap**, the **two-heap streaming median**, **merge-K**, and the **custom-comparator lazy-expansion** heap — cover the vast majority of heap interviews. Block B is the at-scale backbone every senior backend interview eventually reaches: **sharding** (range / hash / consistent-hashing) and **leader-follower replication** with its lag mitigations. Block C is **design-a-data-structure** — composing array + hashmap (+ heap) under hard O(1)/O(log n) constraints (GetRandom, LFU).

📌 **Study today:** Heaps — top-K, two-heap median, merge-K, custom comparator (LC 215, 347, 295, 23, 373) · system design: sharding & replication · design-DS (LC 380, 460) · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Heaps / Priority Queues

### Theory

A **binary heap** is a complete binary tree stored in an array, maintaining the **heap property**: in a *min-heap* every parent ≤ its children (root = global minimum); in a *max-heap* every parent ≥ its children. Stored at index `i`: children at `2i+1`, `2i+2`, parent at `(i-1)/2`. Both `offer` (sift-up) and `poll` (sift-down) are **O(log n)**; `peek` is O(1). Building a heap from `n` elements is O(n) via bottom-up heapify (not O(n log n) — the classic surprise).

Java's `PriorityQueue` is a min-heap by default. For a max-heap pass `Collections.reverseOrder()` or a comparator. The single most important reflex: **`PriorityQueue<T>(Comparator)` lets you order by anything** — that's how every "top-k by frequency / by sum / by custom tie-break" problem is solved.

**When to reach for a heap:** "k largest / smallest / most frequent", "running median of a stream", "merge k sorted sequences", "schedule by priority / next-available", Dijkstra/Prim. **When NOT to:** if you need the *full* sorted order anyway (just sort — O(n log n) once beats n heap ops), or random access by rank (use an order-statistics tree / QuickSelect).

The four patterns to own cold:

| Pattern | Heap | Why | Complexity |
|---|---|---|---|
| **Top-K largest** | *min*-heap of size k | root is the smallest of the k best; evict it when a bigger one arrives | O(n log k) |
| **Streaming median** | max-heap (low half) + min-heap (high half) | medians sit at the two tops | O(log n) insert, O(1) query |
| **Merge-K sorted** | min-heap of size ≤ k (one per list) | always pop the global-min frontier | O(n log k) |
| **K-smallest pairs / Dijkstra** | min-heap with custom comparator, lazy expansion | only materialise candidates as needed | O(k log k) |

### Worked example — Kth Largest Element (LC 215)

The counter-intuitive trick: to find the k *largest*, keep a *min*-heap of size k. After processing all n elements, the heap holds exactly the k largest seen, and its root (the minimum of those) is the k-th largest.

```java
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>(); // min-heap
    for (int x : nums) {
        minHeap.offer(x);
        if (minHeap.size() > k) minHeap.poll(); // evict the smallest → keep k largest
    }
    return minHeap.peek(); // smallest of the k largest = k-th largest
}
```

Why size-k min-heap and not a max-heap of all n? A max-heap of all n then k polls is O(n + k log n); the size-k min-heap is O(n log k) and, crucially, **works on a stream** — you never need all n in memory at once. State both; lead with the streaming-friendly one. **Target AC: 15 min.**

**The "can you do better?" follow-up — QuickSelect:** partition around a pivot (Lomuto/Hoare); recurse only into the side containing rank k. Average **O(n)**, worst O(n²) (mitigate with a random pivot → expected O(n)). It mutates the array and is *not* streaming. Say: "Heap for streaming / external data; QuickSelect when the whole array is in memory and I want expected-linear."

### Worked example — Find Median from Data Stream (LC 295, Hard)

Maintain two heaps splitting the data at the median:

- **`low`** — a *max*-heap holding the smaller half (its top is the largest of the small half).
- **`high`** — a *min*-heap holding the larger half (its top is the smallest of the large half).

Invariant: `low.size() == high.size()` or `low.size() == high.size() + 1`. The median is `low.top()` (odd total) or `(low.top() + high.top()) / 2` (even).

```java
class MedianFinder {
    private final PriorityQueue<Integer> low  = new PriorityQueue<>(Collections.reverseOrder()); // max-heap
    private final PriorityQueue<Integer> high = new PriorityQueue<>();                            // min-heap

    public void addNum(int num) {
        low.offer(num);              // 1) always push to low
        high.offer(low.poll());      // 2) move low's max into high (keeps order across the split)
        if (high.size() > low.size())// 3) rebalance so low is never smaller than high
            low.offer(high.poll());
    }

    public double findMedian() {
        return low.size() > high.size()
            ? low.peek()
            : (low.peek() + high.peek()) / 2.0;
    }
}
```

The push-then-shuffle-then-rebalance dance guarantees both the size invariant *and* that every element in `low` ≤ every element in `high`. **The classic bug is skipping the rebalance** — narrate all three steps aloud. O(log n) insert, O(1) median. **Target AC: 25 min.**

### Worked example — Merge K Sorted Lists (LC 23, Hard)

```java
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);
    for (ListNode head : lists) if (head != null) pq.offer(head); // seed: one head per list
    ListNode dummy = new ListNode(0), tail = dummy;
    while (!pq.isEmpty()) {
        ListNode node = pq.poll();        // global minimum among the k frontiers
        tail.next = node; tail = node;
        if (node.next != null) pq.offer(node.next); // advance that list
    }
    return dummy.next;
}
```

**Where does `log k` come from?** The heap never holds more than k nodes (one frontier per list), so each of the n total nodes is pushed/popped once at O(log k) → **O(n log k)**, space O(k). Compare: naive pairwise merge is O(nk); divide-and-conquer pairwise merging is also O(n log k) but without a heap. **Target AC: 20 min.**

### Practice set

| # | Problem (LC) | Approach | Key insight | Complexity |
|---|---|---|---|---|
| 1 | Kth Largest Element (215) | size-k min-heap (or QuickSelect) | min-heap top = k-th largest | O(n log k) |
| 2 | Top K Frequent Elements (347) | freq map → size-k min-heap | order by frequency; bucket-sort O(n) follow-up | O(n log k) |
| 3 | Find Median from Data Stream (295) | two heaps, rebalanced | median sits at the two tops | O(log n) ins |
| 4 | Merge K Sorted Lists (23) | min-heap of frontiers | heap bounded by k → log k | O(n log k) |
| 5 | Find K Pairs w/ Smallest Sums (373) | min-heap, lazy pair expansion | push neighbour candidates only as popped | O(k log k) |
| 6 | (Stretch) Kth Smallest in Sorted Matrix (378) | heap over rows OR binary-search on value | value-range binary search is the elegant one | O(k log k) |

**Top K Frequent (LC 347)** — the comparator carries the logic:

```java
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int x : nums) freq.merge(x, 1, Integer::sum);
    PriorityQueue<Integer> pq = new PriorityQueue<>((a, b) -> freq.get(a) - freq.get(b)); // min by freq
    for (int key : freq.keySet()) {
        pq.offer(key);
        if (pq.size() > k) pq.poll(); // drop the least frequent of the running top-k
    }
    int[] out = new int[k];
    for (int i = k - 1; i >= 0; i--) out[i] = pq.poll();
    return out;
}
```

**Talking point:** "If `k` is close to `n`, or the value domain is small, I'd switch to **bucket sort** — index buckets by frequency (max n), walk high→low — that's O(n) and beats the heap. The heap wins when k ≪ n or the stream doesn't fit in memory." **Target AC: 347 in 15 min.**

**Find K Pairs (LC 373)** — lazy expansion keeps the heap small: seed with `(nums1[i], nums2[0])` for each `i`; when you pop `(i, j)`, push `(i, j+1)`. You only ever materialise the candidates adjacent to what you've already taken — never the full n×m grid.

---

## Block B — System Design: Sharding & Replication

This is the horizontal-scale half of the at-scale backbone (CAP/PACELC follows tomorrow). Interviewers probe it because it separates "I added a read replica once" from "I can reason about *where data lives* and *what happens when a node dies*."

### Sharding (horizontal partitioning) — three strategies, cold

**Sharding** splits one logical dataset across N physical nodes so no single node holds (or serves) all of it. The **shard key** decides which node a row lands on — choosing it is the whole game.

| Strategy | Rule | Pro | Failure mode |
|---|---|---|---|
| **Range** | key ranges → shards (IDs 0–1M → shard 1, 1M–2M → shard 2) | trivial range scans (`WHERE id BETWEEN`) | **hotspots** if data/traffic isn't uniform — "newest users" all on the last shard |
| **Hash (mod N)** | `shard = hash(key) % N` | even distribution | adding a node changes N → **rehashes ~all keys** (massive data movement) |
| **Consistent hashing** | nodes + keys on a ring; key → first node clockwise | adding/removing a node moves only **~1/N of keys** | uneven load without vnodes (a node owns a big arc) |

**Consistent hashing — the GCC-level answer.** Place nodes on a 0..2³² ring (hash of node id); a key hashes to a point and is owned by the next node clockwise. Add a node and only the keys between it and its predecessor move — **O(n/N)**, no global rehash. The catch: with few nodes the arcs are uneven, so each physical node is given **~100–200 virtual nodes (vnodes)** scattered around the ring → load smooths out, and a dead node's load spreads across many neighbours instead of dumping entirely on one. More vnodes = better balance, more ring-metadata/coordination overhead. Used by **Cassandra, DynamoDB, Redis Cluster** (16,384 fixed hash slots — a discretised variant).

```text
        node A (vnodes a1,a2,a3 ...)
   key k ─────────────► first vnode clockwise owns k
        node B (vnodes b1,b2 ...)     add node C → only keys on the arc
        node C (vnodes c1,c2 ...)     just before each c-vnode relocate
```

**The shard-key decision framework** (say this out loud in a design round):
1. **Most common query pattern → shard key.** Co-locate the data a query needs on one shard. Metadata service → shard on `owner_id` so "list my files" hits one shard, not a scatter-gather.
2. **Will it hotspot?** The key must spread load uniformly. Time-series sharded by raw timestamp → the *current* shard is always hot; mitigate with a randomised bucket suffix (`yyyymmdd-{0..K}`).
3. **How often do nodes join/leave?** Frequent → consistent hashing (cheap rebalance). Rare/fixed → hash-mod-N is simpler.

The pain point of any scheme: **cross-shard queries / joins** (scatter-gather, slow) and **distributed transactions** (need Saga / 2PC). A good shard key makes the hot queries single-shard and pushes the rare ones to a fan-out path.

### Leader-follower (primary-replica) replication

One **leader** takes all writes; **followers** replicate the change log and serve reads — scaling reads and giving you failover.

- **Async replication (common):** leader commits and acks the client *before* followers apply. Fast, but a follower read can be stale, and a leader crash can lose the last few un-replicated writes.
- **Sync replication (rare, expensive):** leader waits for ≥1 follower to ack before acking the client. No data loss on failover, but every write pays the slowest follower's latency, and a stuck follower stalls writes (so real systems use *semi-sync* — wait for one of several).

**Replication lag** = the window between a leader commit and a follower applying it (PostgreSQL streaming replication: ~10 ms–2 s under load). A follower read inside that window returns **stale data** — the source of "I uploaded a file but it's not in the list" bugs.

**Three mitigations (know all three, and which is most pragmatic):**
1. **Read from the leader for critical reads** — correct but defeats the replica's purpose for those queries; reserve for must-be-fresh reads (a balance check right after a transfer).
2. **Read-your-writes via LSN** — record the write's WAL **LSN**; on the next read, if the chosen replica's applied LSN is behind that, route to the leader (or wait). Gives each user their own writes back without forcing *all* reads to the leader.
3. **Accept and handle staleness** — for non-critical reads, render optimistic UI from the POST response while the DB catches up (most pragmatic for file-list-after-upload). Pair with **monotonic reads** — hash a user to a consistent replica so they never see time *go backwards* (read a new value, then an older one).

> **Project tie-in (Smart360):** caching S3 pre-signed URLs in Redis with a TTL is a deliberate **eventual-consistency** choice — a slightly stale but still-valid URL is fine, and strong consistency (invalidate-on-every-change) would negate the ~80% S3-call reduction. PostgreSQL stayed the CP source of truth; Redis was the AP-leaning read cache. The TTL was tuned to sit safely *inside* the pre-signed URL's own expiry window.

### Gotchas

- **Picking a shard key by what's convenient, not by query pattern** — guarantees scatter-gather on your hottest query.
- **Hash-mod-N in a system that scales often** — every node addition is a near-total reshuffle; use consistent hashing.
- **Treating a read replica as strongly consistent** — it isn't; name the lag window and pick a mitigation.
- **No vnodes with consistent hashing** — load skews and a node death dumps onto a single neighbour.
- **Resharding without verification** — always dual-write → verify checksums/row counts → cut over reads → stop dual-write → decommission.

---

## Block C — Design a Data Structure

Design-DS rounds test composition: can you stitch array + hashmap (+ heap/linked structure) so every operation hits its required O(1) or O(log n) bound? **Narrate the complexity of each operation as you write it** — that narration *is* the signal.

### Worked example — Insert Delete GetRandom O(1) (LC 380)

The trick: an `ArrayList` gives O(1) random access (for `getRandom`) and O(1) append; a `HashMap<value, index>` gives O(1) lookup. The only hard part is O(1) delete from the middle of an array — solved by **swap-with-last-then-pop**.

```java
class RandomizedSet {
    private final List<Integer> vals = new ArrayList<>();
    private final Map<Integer, Integer> idx = new HashMap<>(); // value → index in vals
    private final Random rnd = new Random();

    public boolean insert(int val) {
        if (idx.containsKey(val)) return false;
        idx.put(val, vals.size());
        vals.add(val);
        return true;
    }

    public boolean remove(int val) {
        Integer i = idx.get(val);
        if (i == null) return false;
        int last = vals.size() - 1;
        int lastVal = vals.get(last);
        vals.set(i, lastVal);          // move last element into the hole
        idx.put(lastVal, i);           // ← the step everyone forgets: fix its index
        vals.remove(last);             // O(1) pop from the tail
        idx.remove(val);
        return true;
    }

    public int getRandom() { return vals.get(rnd.nextInt(vals.size())); }
}
```

**The classic bug:** removing the target but forgetting to update the *moved* element's index in the map → a dangling index that corrupts the next remove. Say "I swap the victim with the last element, patch the moved element's index, then pop" — that sentence proves you've got it. **Target AC: 20 min.**

### Worked example — LFU Cache (LC 460, Hard)

The step up from LRU (recency) to **frequency** (with recency as the tie-breaker). All ops O(1). Three structures plus one pointer:

1. `Map<key, Node>` — `Node` holds key, value, and current frequency.
2. `Map<freq, LinkedHashSet<key>>` — frequency buckets; `LinkedHashSet` preserves *insertion/recency order within a tie*, so the oldest at a given frequency is evicted first.
3. **`minFreq`** pointer — the smallest frequency currently present, so eviction is O(1) (no scanning buckets).

```java
class LFUCache {
    private final int capacity;
    private int minFreq = 0;
    private final Map<Integer, int[]> kv = new HashMap<>();              // key → [value, freq]
    private final Map<Integer, LinkedHashSet<Integer>> buckets = new HashMap<>(); // freq → keys (recency order)

    LFUCache(int capacity) { this.capacity = capacity; }

    public int get(int key) {
        if (!kv.containsKey(key)) return -1;
        touch(key);                       // bump frequency
        return kv.get(key)[0];
    }

    public void put(int key, int value) {
        if (capacity == 0) return;
        if (kv.containsKey(key)) { kv.get(key)[0] = value; touch(key); return; }
        if (kv.size() >= capacity) evict();
        kv.put(key, new int[]{value, 1});
        buckets.computeIfAbsent(1, f -> new LinkedHashSet<>()).add(key);
        minFreq = 1;                      // a brand-new key always has freq 1
    }

    private void touch(int key) {         // move key from freq f to f+1
        int f = kv.get(key)[1]++;
        LinkedHashSet<Integer> oldB = buckets.get(f);
        oldB.remove(key);
        if (oldB.isEmpty() && f == minFreq) minFreq++; // its bucket emptied at the min → advance
        buckets.computeIfAbsent(f + 1, x -> new LinkedHashSet<>()).add(key);
    }

    private void evict() {                // drop LRU key in the minFreq bucket
        LinkedHashSet<Integer> b = buckets.get(minFreq);
        int victim = b.iterator().next(); // oldest at the lowest frequency
        b.remove(victim);
        kv.remove(victim);
    }
}
```

**The min-freq pointer is the whole trick** — without it, eviction would scan all buckets for the lowest non-empty frequency, breaking O(1). On a new insert `minFreq` resets to 1 (the new key); on a `touch` that empties the min bucket it advances by one. State both transitions aloud. **Target AC: 30 min.**

---

## 💻 Practice coding questions

1. Kth Largest Element (LC 215) — size-k min-heap, then state the QuickSelect follow-up.
2. Top K Frequent Elements (LC 347) — heap version, then the O(n) bucket-sort version.
3. Find Median from Data Stream (LC 295) — two heaps; narrate the 3-step rebalance.
4. Merge K Sorted Lists (LC 23) — min-heap of frontiers; explain where `log k` comes from.
5. Find K Pairs with Smallest Sums (LC 373) — lazy candidate expansion.
6. Insert Delete GetRandom O(1) (LC 380) — array + index map, swap-with-last delete.
7. LFU Cache (LC 460) — kv map + frequency buckets + min-freq pointer.
8. Implement a max-heap top-K (k smallest) by flipping the comparator — prove you can do both directions.
9. (Stretch) Kth Smallest Element in a Sorted Matrix (LC 378) — heap, then value-range binary search.
10. (Stretch) LRU Cache (LC 146) — `LinkedHashMap` or hashmap + doubly-linked list; contrast with LFU.
11. Write the consistent-hashing key lookup (ring of vnode hashes; binary-search clockwise to the owner).
12. Write a read-your-writes router: given the write's LSN and each replica's applied LSN, pick leader or replica.

---

## 🎤 Interview questions

1. **To find the k largest, why a min-heap of size k and not a max-heap?** The size-k min-heap keeps the k best seen; its root (the smallest of those) is the k-th largest, and a new element only matters if it exceeds the root. It's O(n log k), O(k) space, and works on a stream — a max-heap of all n is O(n) build + O(k log n) and needs all n in memory.
2. **Heap vs QuickSelect for Kth Largest?** Heap: O(n log k), streaming-friendly, no array mutation. QuickSelect: expected O(n) (random pivot), worst O(n²), in-place, needs the full array. Heap for streams/external data; QuickSelect for in-memory expected-linear.
3. **Why is building a heap O(n), not O(n log n)?** Bottom-up heapify: most nodes are near the leaves with tiny sift-down depth; the sum ∑ height·count converges to O(n). Inserting one-by-one *is* O(n log n) — the distinction is the building method.
4. **Explain the two-heap median invariant.** A max-heap holds the low half, a min-heap the high half; sizes differ by ≤1 and every low element ≤ every high element. Median is the larger heap's top (odd) or the average of both tops (even). Forgetting to rebalance after each insert is the classic bug.
5. **Merge K Sorted Lists — where does `log k` come from?** The heap holds at most k nodes (one frontier per list); each of the n nodes is pushed/popped once at O(log k) → O(n log k), space O(k). Not log n.
6. **Top K Frequent — when do you abandon the heap for bucket sort?** When k approaches n or the value domain is small: index buckets by frequency (≤ n), walk high→low → O(n), beating O(n log k). Heap wins when k ≪ n or it's a stream.
7. **Range vs hash vs consistent-hashing sharding — one-line failure mode each.** Range → hotspots when data/traffic isn't uniform. Hash-mod-N → adding a node rehashes nearly all keys. Consistent hashing → uneven load without vnodes (and a dead node dumps onto one neighbour).
8. **Why virtual nodes in consistent hashing?** With few physical nodes the ring arcs are uneven (skewed load) and a node death dumps its whole arc onto one neighbour. ~100–200 vnodes per node smooth the distribution and spread a failed node's load across many — at the cost of more ring metadata.
9. **How do you choose a shard key?** Shard on the field your hottest query filters by (co-locate that query's data on one shard), confirm it spreads load uniformly (no hotspots), and factor in how often nodes join/leave (frequent → consistent hashing). Example: metadata on `owner_id` for single-shard "list my files."
10. **What is replication lag and when does it bite?** The window between a leader commit and a follower applying it (PostgreSQL streaming ~10 ms–2 s). A follower read inside that window returns stale data — "I uploaded a file but it's not in my list."
11. **Three replication-lag mitigations — which is most pragmatic?** (1) Read critical queries from the leader; (2) read-your-writes via LSN comparison; (3) accept staleness + optimistic UI (+ monotonic reads). For file-list-after-upload, (3) is the most pragmatic — show the POST response while the DB catches up.
12. **Async vs sync replication trade-off?** Async: leader acks before followers apply — fast, but stale reads and possible data loss on failover. Sync: leader waits for a follower ack — no loss on failover, but every write pays the slowest follower; real systems use semi-sync (wait for one of several).
13. **Reshard a 10M-row table to more nodes without downtime.** Pick a shard key (e.g. `owner_id` hash), dual-write to old + new, background-migrate in batches, verify checksums/row counts, switch reads to new, stop dual-writes, decommission old. Never cut over without verification.
14. **LFU: what does the min-freq pointer buy you?** O(1) eviction — it points straight at the lowest non-empty frequency bucket so you never scan. It advances when a `touch` empties the min bucket and resets to 1 on every new insert.
15. **LFU eviction with a frequency tie — who goes?** The least *recently used* key at the minimum frequency. The `LinkedHashSet` per bucket preserves recency order, so its iterator's first element is the victim.
16. **GetRandom O(1) — the one step candidates miss?** After swapping the victim with the last array element, you must update the *moved* element's index in the value→index map before popping the tail; otherwise the map points at a stale slot.
17. **(Curveball) Median when 90% of values are in [0,100].** Drop the two heaps for a counting array over [0,100] plus overflow buckets for the rest — O(1) amortised insert and median lookup. Know when the general O(log n) structure is overkill for a constrained domain.
18. **(Curveball) Your shard key turned out to hotspot — fix it live.** Identify the hot key/range, add a salted/randomised suffix to spread it (or split the hot shard), dual-write during migration, verify, cut over. If it's time-series, range-shard but bucket the current window across K sub-shards.

---

## ✅ Self-check

1. Implement the size-k min-heap top-K and explain why a *min*-heap finds the *largest*, in ≤10 min, no notes.
2. Code the two-heap median and narrate the three-step rebalance (push-low, move-to-high, rebalance) aloud.
3. Name the failure mode of range vs hash vs consistent-hashing sharding in one sentence each.
4. Give the three replication-lag mitigations and say which you'd pick for "file list right after upload" and why.
5. Implement LFU's `touch` and `evict` cold; explain what the min-freq pointer saves you.

---

*Nav: ← [Day 3 (Wed Jul 22)](03-wed-jul-22.md) · [Week 4](README.md) · [Day 5 (Fri Jul 24)](05-fri-jul-24.md) →*
