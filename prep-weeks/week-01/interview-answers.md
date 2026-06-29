# Week 1 — Interview Question Answers

Answers to the 🎤 interview prompts in each Week-1 day file — attempt aloud first, then check.

---

## Day 1 — Mon Jun 29 — Two Pointers + equals/hashCode

1. **When do two pointers beat a HashMap for Two Sum?** When the array is sorted (or cheaply sortable) — two pointers use **O(1)** extra space versus the hash map's O(n). *Why:* sortedness lets you eliminate one candidate per step. *Gotcha:* if you must sort first, you pay O(n log n), so the hash map can win on an unsorted array where you also need original indices.

2. **Prove that moving the shorter pointer is correct in Container With Most Water.** Area = `min(h[l], h[r]) * (r - l)`; width strictly shrinks as pointers converge, and height is capped by the shorter line. Moving the *taller* line keeps the same height cap but reduces width ⇒ strictly worse, so the taller line is already maximal against the current shorter one — discard the shorter. *Gotcha:* move the taller pointer and you can silently skip the optimum (wrong answer, no crash).

3. **In Trapping Rain Water, why only one running max at a time?** Water at `i` is `min(maxL, maxR) - h[i]`. When `h[left] <= h[right]` you *know* the right wall is at least as tall, so the `min` is governed by `maxL` — commit water using only the left running max. *Gotcha:* you must advance the **shorter** side; advancing the taller side breaks the "min is on my side" guarantee.

4. **State the equals/hashCode contract.** If `a.equals(b)` then `a.hashCode() == b.hashCode()` (mandatory). The converse need not hold — unequal objects may collide. `equals` must be reflexive, symmetric, transitive, consistent, and `x.equals(null)` is false. *Gotcha:* override one, override both, or hash collections break silently.

5. **What breaks in a HashSet if you violate the contract?** If equal objects hash differently they land in different buckets → the set stores duplicates and lookups miss. If you mutate a field used in `hashCode` after insertion, the object becomes **ghost-present**: still in the set (you can iterate to it) but `contains` returns false because it now hashes to a different bucket than it physically occupies.

6. **Two objects, same hashCode, equals false — what happens in a HashMap?** A legal collision: both go to the same bucket as separate chain nodes; the bucket's `equals` walk keeps them distinct and correct. *Gotcha:* only performance suffers — that bucket degrades to O(n), or O(log n) after treeification.

7. **Is a constant hashCode (`return 1`) ever legal?** Yes — equal objects trivially share it, so the contract holds. *Why it's bad:* every element collides into one bucket → O(n) (O(log n) after treeify) lookups. Legal, never advisable for general keys (but see Q8 for the JPA exception).

8. **Why a constant hashCode for JPA entities specifically?** A `@GeneratedValue` PK is `null` while transient and assigned on flush. If `hashCode` depended on `id`, it would *change while the entity sits in a `Set`* → ghost-present bug. A constant hash + id-based `equals` (`id != null && id.equals(other.id)`) stays stable across the transient→persistent transition. *Gotcha:* don't put mutable business fields in entity equals either.

9. **Why `instanceof` instead of `getClass()` in a JPA entity's equals?** Hibernate hands you **proxy subclasses** (`Role$HibernateProxy$xyz`); `getClass()` equality fails proxy-vs-real comparisons, while `instanceof` tolerates them. *Gotcha:* for plain value classes the opposite advice holds — `getClass()` is safer there to keep symmetry across subclasses.

10. **What does Lombok `@EqualsAndHashCode` do by default, and why is that wrong for entities?** By default it uses **all non-static fields**, including mutable ones and lazy associations (which can trigger queries). For entities use `@EqualsAndHashCode(onlyExplicitlyIncluded = true)` with `@EqualsAndHashCode.Include` on the PK only.

11. **Valid Palindrome with emoji/Unicode — how does the code change?** Iterate by **code point** (`s.codePointAt(i)` + `Character.isLetterOrDigit(cp)`, advance by `Character.charCount(cp)`) instead of `char`. *Why:* a surrogate pair is two `char`s but one logical character, so per-`char` comparison would split it.

12. **Move Zeroes / Remove Duplicates use two pointers but don't converge — what's the variant?** **Same-direction (slow/fast) pointers:** `fast` scans, `slow` marks the write position. Different invariant ("everything before `slow` is finalized") than the converging-from-both-ends variant. *Gotcha:* don't confuse the two — the move rule and termination differ.

13. **How is `Objects.hash(a, b)` computed?** A folding loop `result = 31 * result + Objects.hashCode(x)` over the args — the same polynomial as `String.hashCode`. *Gotcha:* `Objects.hash(x)` boxes args into a varargs array; for a single hot field prefer `x.hashCode()` directly to avoid the allocation.

14. **Why does `String` make an ideal HashMap key?** It's immutable (the hash can't drift after insertion) and its `hashCode` is **cached** after first computation, so repeated lookups don't recompute. *Gotcha:* the cache is only valid *because* it's immutable — a mutable key with a cached hash would lie.

15. **Squares of a Sorted Array — why fill from the back?** The largest square is at one of the two ends (most-negative or most-positive), so a converging pass comparing `abs(left)` vs `abs(right)` and writing into the **highest** free index stays O(n). *Gotcha:* filling front-to-back forces a final sort, losing the linear bound.

16. **Difference between reflexive/symmetric/transitive — give a symmetry violation.** Reflexive: `x.equals(x)`; symmetric: `a.equals(b) ⇔ b.equals(a)`; transitive: `a=b, b=c ⇒ a=c`. **Symmetry violation:** `Point.equals` uses `instanceof` and `ColorPoint.equals` adds a color check — `point.equals(colorPoint)` can be true while `colorPoint.equals(point)` is false. *Gotcha:* this is exactly why value classes often use `getClass()` instead of `instanceof`.

---

## Day 2 — Tue Jun 30 — Sliding Window + HashMap Internals

1. **Why is the sliding window O(n) when it has two nested loops?** `left` and `right` each advance at most `n` times across the whole run and never reset, so total pointer moves ≤ 2n. *Why:* each element enters the window once and leaves at most once — amortized O(1) per element. *Gotcha:* this only holds if `left` never moves backward.

2. **In LC 3, why `left = max(left, lastSeen + 1)` instead of `left++`?** A duplicate may be far behind `right`, so you must jump `left` past the prior occurrence in one move. The `max` guards against moving `left` *backward* when the duplicate sits outside the current window (a stale earlier occurrence). *Gotcha:* plain `left++` leaves duplicates inside the window — wrong answer.

3. **Walk me through `hashMap.put("name", "Jay")`.** Compute `key.hashCode()` → spread with `h ^ (h >>> 16)` → index `(n-1) & spread` → walk the bucket chain calling `equals`. On a match, replace the value (size unchanged); else append the node (treeify if chain ≥ 8 **and** capacity ≥ 64); finally if `size > threshold`, resize. *Gotcha:* same-key put replaces, it doesn't add.

4. **Why the `h ^ (h >>> 16)` bit-spread?** The index uses only the low bits (`(n-1) & hash`), but many hash codes vary mostly in their **high** bits. XORing the top 16 bits down mixes that entropy into the index bits, cutting collisions cheaply. *Gotcha:* without it, e.g. `Integer` keys differing only in high bits would all collide in a small table.

5. **Why must capacity be a power of 2?** So `(n-1) & hash` equals `hash % n` — the mask `n-1` is all-ones only when `n = 2^k`, making bucket selection a single AND instead of a modulo. *Gotcha:* `new HashMap<>(1000)` rounds *up* to 1024 for this reason.

6. **When does a bucket treeify, and what does it fix?** At chain length **8 AND** table capacity **≥ 64**; below capacity 64 it resizes instead (cheaper, usually disperses the chain). It bounds worst-case bucket lookup at O(log n) vs O(n), mitigating hash-collision DoS. Un-treeifies back to a list at **6** (hysteresis to prevent thrashing). *Why 8:* with a good hash and load factor 0.75, P(bucket ≥ 8) < 1e-7 (Poisson) — it's a rare safety net.

7. **Capacity 16, 10 entries — does the 11th insert resize?** No. Threshold is `16 * 0.75 = 12`; resize triggers when size *exceeds* 12, i.e. on the 13th insert (→ capacity 32). *Gotcha:* the comparison is `size > threshold`, not `>=`.

8. **Initial capacity to hold 1000 entries without resizing?** `1000 / 0.75 ≈ 1334` → next power of 2 = **2048**. Or construct `new HashMap<>(1334)` and let the constructor round up. *Gotcha:* passing `1000` directly gives capacity 1024, which *will* resize before 1000 entries.

9. **How does Java 8 resize without full rehashing?** For each entry, one bit decides: `hash & oldCapacity` is 0 → stays at index `i`; is 1 → moves to `i + oldCapacity`. Each old bucket splits into a "lo" and "hi" list, no hash recomputation. *Why:* the new mask is just the old mask plus one higher bit.

10. **Two lookups for the same key return different results — most likely cause?** The key's `hashCode` depends on a mutable field that changed between insert and lookup, so it now hashes to a different bucket (ghost-present). *Gotcha:* the entry is still physically in the map — iteration finds it, `get` doesn't.

11. **Why is `HashMap` unsafe under concurrency, and what's the Java 7 horror story?** Concurrent writes corrupt chains; in **Java 7** a concurrent resize could form a cycle in a bucket and spin a thread at 100% CPU forever. Java 8 changed resize to preserve order and avoid the loop, but concurrent use is *still* unsafe (lost updates). *Gotcha:* "Java 8 fixed it" is wrong — it only fixed the infinite loop.

12. **`ConcurrentHashMap` vs `Hashtable` vs `synchronizedMap`?** CHM: per-bucket `synchronized` + CAS fast path, lock-free reads via `volatile`, high throughput. Hashtable / `synchronizedMap`: a single global lock serializing every operation. Prefer CHM in hot paths. *Gotcha:* iterating a `synchronizedMap` still needs manual external synchronization.

13. **Why is `String` the ideal HashMap key?** Immutable (hash can't drift) and its `hashCode` is cached after first computation. *Gotcha:* the caching is safe only because of immutability.

14. **Difference between capacity, size, and threshold?** Capacity = table length (a power of 2); size = number of entries; threshold = `capacity × loadFactor`, the size at which the table doubles. *Gotcha:* capacity ≠ size — an empty map can have capacity 16.

15. **Why is LC 560 not solvable with a sliding window?** With negative numbers, expanding the window doesn't monotonically increase the sum, so the "shrink when too big" invariant fails. Prefix sum + a `HashMap<prefix,count>` handles negatives. *Gotcha:* sliding windows generally require non-negative values for the monotonicity argument.

16. **Explain the `{0:1}` seed in LC 560.** A subarray starting at index 0 has the *empty prefix* (sum 0) as its complement; seeding `prefixCount.put(0, 1)` lets `prefix - k == 0` match it. Without it you miss every subarray that begins at index 0. *Gotcha:* this single line is the most-asked LC 560 detail.

17. **Why never shrink `maxFreq` in LC 424?** You want the **longest** valid window; a stale, too-high `maxFreq` only ever permits windows no longer than the current best, so it can't corrupt the answer. Not recomputing it keeps the algorithm O(n) instead of O(26n). *Gotcha:* use `if` (not `while`) to slide — the window never actually shrinks, it just stops growing.

18. **What's the load factor and why 0.75?** The fraction of capacity filled before resizing. 0.75 is the empirical space/time sweet spot: low enough that buckets stay short (Poisson keeps P(len ≥ 8) tiny), high enough not to waste much memory. *Gotcha:* raising it saves memory but lengthens chains and lookups.

---

## Day 3 — Wed Jul 01 — 3Sum / Prefix Sum + Generics & Autoboxing

1. **Why does `List<Dog>` not satisfy `List<Animal>` though `Dog extends Animal`?** Generics are **invariant** for safety: if it were allowed, you could do `List<Animal> a = dogs; a.add(new Cat());` and corrupt the dog list. *Why:* wildcards (`? extends` / `? super`) restore the flexibility *safely*. *Gotcha:* arrays *are* covariant (and throw `ArrayStoreException` at runtime to compensate) — generics chose compile-time safety instead.

2. **Explain type erasure and three things it forbids.** Generic types exist only at compile time; at runtime `List<String>` is just `List`. Forbidden: `instanceof` on a parameterized type, `new T[]` / `new T()`, and overloads that erase to the same signature (`f(List<String>)` vs `f(List<Integer>)`). *Why:* erasure preserves backward compatibility with pre-generics bytecode.

3. **What is PECS?** **Producer Extends, Consumer Super.** Read from a producer with `? extends T` (covariant, read-only); write to a consumer with `? super T` (contravariant, write-only). See `Collections.copy(List<? super T> dest, List<? extends T> src)`. *Gotcha:* you can't `add` to a `? extends` list (except null) nor read a specific type from a `? super` list (only `Object`).

4. **`Integer a = 128, b = 128; a == b`?** `false`. The `Integer` cache only covers **−128…127**; outside it `valueOf` makes new objects, so `==` (reference equality) fails. `a.equals(b)` is always `true`. *Gotcha:* lower bound is fixed; the upper bound is tunable via `-XX:AutoBoxCacheMax`. Always use `.equals()` on boxed numbers.

5. **Write the safe unboxing of a nullable `Integer` DB column.** `int c = Objects.requireNonNullElse(count, 0);` or guard `if (count != null) { int c = count; }`. *Why:* direct `int c = count;` throws NPE on auto-unboxing a null. *Gotcha:* the NPE site is the assignment, not the DB read — easy to misdiagnose.

6. **Why does `Long sum = 0L; for (int x: list) sum += x;` hurt performance?** Every `+=` unboxes `sum`, adds, and re-boxes into a **new** `Long`, allocating per iteration → GC pressure in tight loops. Use a `long` primitive accumulator or `IntStream.sum()`. *Gotcha:* it's correct, just slow — only shows up under load/profiling.

7. **How does Spring inject `@Autowired List<LLMProvider>` despite erasure?** The list's generic type is erased in bytecode, but Spring resolves bean types from its **container metadata** (and reads generic info via reflection where the JVM retains it, e.g. field/parameter signatures), injecting all matching beans. *Gotcha:* erasure removes runtime *instances'* type args, not the *declaration* signatures reflection can still read.

8. **`map.get(id)` where `id` is `int` and map is `Map<Long,V>` — why a miss?** The `int` autoboxes to `Integer`, and `Integer.equals(Long)` is `false`, so keys never match. Use `map.get(Long.valueOf(id))`. *Gotcha:* it compiles cleanly and silently misses — no exception.

9. **Why can't you create a generic array `new T[n]`?** Arrays are **reified** (carry runtime component type for `ArrayStoreException` checks); generics are erased, so there's no runtime `T` to store. *Workaround:* `(T[]) new Object[n]` with an unchecked cast, or `Array.newInstance(clazz, n)` with a `Class<T>` token.

10. **3Sum — where do duplicates get skipped and why three places?** The outer fixed element (`nums[i] == nums[i-1]` for `i > start`), and **both** inner pointers *after* recording a valid triplet. *Why:* equal adjacent values would otherwise emit the same triplet repeatedly. *Gotcha:* moving only one inner pointer after a hit is wrong — a valid pair is consumed, both must move.

11. **Product of Array Except Self without division — approach?** Left-to-right pass fills `out[i]` with the product of everything to the left; right-to-left pass multiplies in a running suffix product. O(n) time, O(1) extra (output excluded). *Why no division:* a single 0 breaks it, and division is banned to force the prefix/suffix insight.

12. **Rotate Array by k in O(1) space?** Three reversals: reverse the whole array, reverse the first `k`, reverse the last `n−k`. *Gotcha:* `k %= n` first, or `k > n` over-rotates.

13. **Group Anagrams — sorted-string key vs frequency-array key?** Sorted-string key is O(k log k) per word and alphabet-agnostic; frequency signature (`Arrays.toString(int[26])`) is O(k + 26), faster for a small fixed alphabet. Both produce an identical key for anagrams. *Gotcha:* the frequency key assumes a bounded alphabet (lowercase ASCII here).

14. **Subarray Sums Divisible by K — what's the key?** The **normalized** remainder `((prefix % k) + k) % k`; count pairs of prefixes sharing a remainder (their difference is divisible by k). Seed remainder 0 once. *Gotcha:* Java's `%` can return negative, so you must normalize or remainders won't match.

15. **When should you prefer wrappers over primitives?** Only when `null`/absence is meaningful (optional fields, nullable columns) or when a generic collection requires it (collections can't hold primitives). Otherwise primitives — no boxing, no NPE, no GC churn.

16. **What is a bridge method?** A compiler-synthesized method that preserves polymorphism after erasure — e.g. a generic `compareTo(T)` gets a bridge `compareTo(Object)` so overriding still works through the erased supertype signature. *Gotcha:* it can show up in stack traces and reflection as a `synthetic`/`bridge` method.

17. **Can you overload `f(List<String>)` and `f(List<Integer>)`?** No — both erase to `f(List)`, a duplicate-method compile error. *Gotcha:* you'd need different method names or a non-generic distinguishing parameter.

---

## Day 4 — Thu Jul 02 — Hashing / Top-K + Collections Deep-Dive + Bit Manipulation

1. **Top K Frequent in O(n) — how?** Count into a HashMap, then **bucket sort**: an array indexed by frequency (frequency ∈ [0, n]), scanned high→low until you have k. *Why:* avoids the O(n log n) of comparison sorting since frequency is itself a bounded integer. *Gotcha:* the bucket array is size n+1 — wasteful when k ≪ n (see Q2).

2. **When heap over bucket sort for Top-K?** When `k ≪ n` and/or you can't materialize an n-sized bucket array (e.g. streaming). A size-k **min-heap** is O(n log k) time, O(k) space — far cheaper than an n-slot array for a size-3 heap over 10M elements.

3. **Longest Consecutive Sequence in O(n) — why isn't it O(n²)?** The inner walk only *starts* at run boundaries (`num - 1 ∉ set`); each element is touched at most twice across the whole loop (once as a skipped non-start, once inside its run) → amortized O(n). *Gotcha:* drop the boundary `continue` guard and you re-walk every run → O(n²).

4. **Why is `String` the ideal HashMap key?** Immutable (hash can't drift) and its `hashCode` is cached after first computation. *Gotcha:* caching is sound only because of immutability.

5. **ArrayList growth — why 1.5× not 2×?** `newCap = oldCap + (oldCap >> 1)` wastes less average memory than `Vector`'s 2×, while still preserving amortized O(1) `add`. *Gotcha:* both factors give amortized O(1); 1.5× just trades a bit more copying for less slack.

6. **ArrayList vs LinkedList for frequent `add(0, e)`?** ArrayList often **wins** below ~thousands of elements: contiguous memory → CPU cache prefetch beats LinkedList's pointer-chasing, despite the O(n) `arraycopy`. *Gotcha:* "LinkedList for frequent inserts" is a junior tell — measure first.

7. **Why `ArrayDeque` over `java.util.Stack`?** `Stack extends Vector` (synchronized → slow) and violates LSP (`add(0, e)` breaks stack semantics). `ArrayDeque` is the modern, unsynchronized stack/queue. *Gotcha:* `ArrayDeque` forbids null elements.

8. **ConcurrentHashMap put internals?** Empty bucket → **CAS** insert (lock-free fast path); non-empty bucket → `synchronized` on the **bin head node only**, so different buckets never contend; reads are lock-free via `volatile`; resize is cooperative (a thread hitting a `ForwardingNode` helps migrate). *Gotcha:* the lock is per-bin, not per-map — that's the throughput win.

9. **Why no null keys/values in ConcurrentHashMap?** `get(k) == null` is ambiguous (key absent vs mapped-to-null), and you can't disambiguate with `containsKey` without a **TOCTOU race** in a concurrent map. So nulls are simply banned. *Gotcha:* plain `HashMap` allows them because single-threaded `containsKey` disambiguation is safe.

10. **CHM vs Hashtable vs synchronizedMap?** CHM = fine-grained per-bin locking + lock-free reads; Hashtable = single full-map lock; `synchronizedMap` = global lock + manual sync for iteration. CHM wins on throughput. *Gotcha:* Hashtable and `synchronizedMap` serialize *all* operations.

11. **What is fail-fast and is it a guarantee?** An iterator snapshots `modCount` at creation; each `next()` checks it and throws `ConcurrentModificationException` on a **structural** change during iteration. It's **best-effort** misuse detection, *not* a thread-safety guarantee. *Gotcha:* never rely on CME being thrown — it may not fire under racing threads.

12. **Does `list.set(i, x)` during iteration throw CME?** No — `set` is a **content** change, not structural, so it doesn't bump `modCount`. *Gotcha:* `add`/`remove` are structural and *do* throw; the distinction trips people up.

13. **`for (s : list) list.remove(s)` — three fixes.** `list.removeIf(predicate)`; `Iterator.remove()` via an explicit iterator; or collect-then-`removeAll` / filter into a new list. *Why:* all avoid mutating the list structurally while the for-each iterator is live.

14. **Weakly-consistent iterator meaning?** (ConcurrentHashMap) It reflects the map state at/after creation, may or may not show concurrent updates, and **never throws CME**. *Gotcha:* it's not a point-in-time snapshot (that's `CopyOnWriteArrayList`) — it can see *some* later writes but isn't guaranteed to see all.

15. **Why XOR for Single Number?** `x ^ x = 0`, `x ^ 0 = x`, and XOR is order-independent → all paired duplicates cancel, leaving the lone element. O(1) space vs a HashSet. *Gotcha:* requires *exactly* pairs except one — if elements repeat 3× it fails (that's LC 137).

16. **What does `n & (n-1)` do?** Clears the **lowest set bit**. Used to count set bits in O(popcount) (`while (n != 0) { n &= n-1; count++; }`) and to test power-of-two: `n > 0 && (n & (n-1)) == 0` (exactly one set bit). *Gotcha:* the `n > 0` guard matters — 0 and negatives would falsely pass.

17. **Counting Bits without `Integer.bitCount`?** DP: `bits[i] = bits[i >> 1] + (i & 1)`. *Why:* `i >> 1` drops the last bit (its popcount already computed); add back the dropped bit. O(n) for the whole 0..n table.

18. **Missing Number — XOR vs Gauss sum?** XOR all indices 0..n and all values; every present number cancels its index, the missing one survives. *Why prefer XOR:* `n(n+1)/2 - sum` can **overflow** for large n; XOR never overflows and needs no `long`.

19. **Where do these bit tricks show up in the JDK?** HashMap: `h ^ (h >>> 16)` spreads high bits into the index; `(n-1) & hash` is modulo-by-power-of-two; `hash & oldCapacity` is the resize split bit. *Gotcha:* all three rely on capacity being a power of 2.

20. **`merge` vs `getOrDefault` for counting?** `map.merge(k, 1, Integer::sum)` is one call and clearer than a `getOrDefault` read-modify-write — and in `ConcurrentHashMap` it's **atomic**, whereas get-then-put is a race. *Gotcha:* the remapping function must handle the "absent" case (merge inserts the value directly when the key is missing).

---

## Day 5 — Fri Jul 03 — Strings + Comparable/Comparator + Weak-Spot Review

1. **Comparable vs Comparator — one line each.** `Comparable` = the class's single **natural** order (`compareTo`, in `java.lang`, implemented on the class). `Comparator` = an **external**, many-possible order (`compare`, in `java.util`, defined outside the class). *Gotcha:* a class can have one `compareTo` but unlimited comparators.

2. **When do you reach for Comparator?** Multiple or runtime-chosen orderings, sorting a class you don't own (can't edit its source), or a quick lambda sort. *Gotcha:* if there's one obvious order intrinsic to the type, prefer `Comparable`.

3. **Chain three comparators with a reversed tie-break.** `Comparator.comparing(A).thenComparing(B).thenComparing(C, Comparator.reverseOrder())`. *Gotcha:* reverse *that key's* comparator inline — don't call `.reversed()` on the whole chain (see Q4).

4. **What does `.reversed()` reverse?** The **entire** composed comparator, not just the last key. To reverse one field, reverse that field's comparator before chaining: `Comparator.comparing(User::age, Comparator.reverseOrder()).thenComparing(User::name)`. *Gotcha:* `.reversed()` at the end silently flips every key.

5. **Why never use subtraction in a comparator?** `a.getX() - b.getX()` **overflows** for large/negative ints and silently produces wrong order. Use `Integer.compare` / `Long.compare`. *Gotcha:* it usually works on small test data, then misorders in production.

6. **TreeSet with a non-Comparable key and no comparator?** `ClassCastException` at **runtime** (not compile time) — the cast happens on the first cross-element comparison, i.e. the second insert. *Gotcha:* a single-element TreeSet won't reveal it.

7. **BigDecimal in TreeSet vs HashSet — sizes?** TreeSet → **1** (`compareTo` is scale-insensitive: `1.0` vs `1.00` compare equal → duplicate). HashSet → **2** (`equals` is scale-sensitive → distinct). The canonical "ordering inconsistent with equals" trap. *Gotcha:* a real money-handling landmine when values arrive at different scales.

8. **Why must `compareTo` be consistent with `equals`?** Sorted collections (TreeSet/TreeMap) use `compareTo == 0` for equality, not `equals`; inconsistency makes them silently merge or "lose" elements (the BigDecimal case). *Gotcha:* `compareTo`-inconsistent classes break `SortedSet`/`SortedMap` contracts but work fine in `HashSet`.

9. **Why is `String` immutable, and what does it buy?** Thread-safety, safe hash caching, string-pool interning, and secure use as map keys / in security contexts (a password `String` can't be mutated mid-use). *Gotcha:* immutability is also *why* `+` in a loop is O(n²) — each concat makes a new object.

10. **Cost of `s = s + x` in a loop?** Each `+` allocates a new backing char array → O(n²) total work and garbage. Use `StringBuilder` for O(n). *Gotcha:* a *single* `"a" + "b" + "c"` expression is fine — the compiler folds/optimizes it; the trap is concatenation *inside a loop*.

11. **`==` vs `.equals()` for strings, and the pool?** `==` compares references; string literals are interned (pooled), so `"a" == "a"` is true, but `new String("a") == "a"` is false. Always use `.equals()`. *Gotcha:* `==` "working" on literals lulls you into bugs the moment a `String` comes from input/IO.

12. **Does `substring` share the backing char array?** Not since **Java 7u6** — it copies, so a small substring of a huge string no longer pins the whole backing array (the old memory-leak surprise is gone). *Gotcha:* old interview material claims it shares — that's pre-7u6.

13. **Expand-around-center vs DP for longest palindrome?** Both O(n²) time, but expand-around-center is **O(1) space** vs DP's O(n²). Manacher's gives true O(n) if pushed. *Gotcha:* expand-around-center needs *two* calls per index — odd `(i, i)` and even `(i, i+1)` centers; forgetting the even case is the classic bug.

14. **Why length-prefix encoding over a pure delimiter?** A delimiter can appear inside the payload and desync the parser; a length prefix tells you exactly how many bytes/chars to consume, so the payload can contain any byte. *Why it matters:* same idea as TCP framing, Redis RESP, gRPC length-prefixed messages.

15. **Kadane invariant + the all-negative case.** `cur = max(num, cur + num)` = best subarray sum *ending at i* (extend vs restart); `best = max(best, cur)`. The `max(num, …)` restart correctly returns the **largest single element** when all values are negative. *Gotcha:* initializing `best = 0` wrongly returns 0 for an all-negative array — init to `nums[0]`.

16. **Find All Anagrams — why `int[26]` not sorting each window?** `Arrays.equals` on two fixed-size count arrays is O(26) = O(1) per window, vs O(k log k) to sort each window. *Gotcha:* you can do even better by tracking a `matches` counter that updates only the two changing buckets per slide.

17. **Difference between fixed-size and variable-size sliding window?** Fixed = window length constant (slide one in, one out, no conditional shrink); variable = expand/shrink based on an invariant (longest/shortest under a constraint). *Gotcha:* using a fixed window for a variable-length problem (or vice versa) is a pattern-recognition miss.

18. **`Comparator.naturalOrder()` vs implementing `Comparable`?** `naturalOrder()` delegates to the type's `compareTo` — handy when you want the natural order but need to wrap it, e.g. `Comparator.nullsLast(Comparator.naturalOrder())`. *Gotcha:* it requires the type to *be* `Comparable`; it doesn't create an order, it reuses one.

19. **How does `TreeMap` find a ceiling key in O(log n)?** Red-black tree traversal: go right when `key > node`, else record the node as a candidate and go left; the last recorded candidate is the ceiling. `floorKey` mirrors it. *Gotcha:* exploits BST ordering — only works because the tree is sorted.

20. **Where does a custom Comparator show up in your projects?** Smart360 sorts report lists by (org, then created-date desc, then name) in one chained expression; WebX ranks providers by (health, then latency, then cost) per request — a **runtime-chosen** comparator because the order depends on request context.

---

## Day 6 — Sat Jul 04 — Timed Mixed DSA + SOLID / System-Design Primer

1. **Explain SRP with your own code.** Smart360's `AuthorizationService` once did token validation *and* role resolution *and* user-management events — three reasons to change → **strangler-fig extraction** (peel responsibilities into focused services incrementally). *Why:* a class with multiple reasons to change is fragile and hard to test. *Gotcha:* SRP is about *reasons to change*, not literal line count.

2. **OCP — how do you add a feature without editing existing code?** Program to an interface; new behavior = a new implementation. WebX's router takes `LLMProvider` impls — adding a provider never edits the router. *Gotcha:* OCP is achieved via abstraction/polymorphism, not by literally never touching files.

3. **Give a real LSP violation.** `Stack extends Vector` — `stack.add(0, e)` corrupts stack semantics; or overriding a method to `throw new UnsupportedOperationException()`. *Why:* a subtype must work anywhere the supertype does. *Gotcha:* `Square extends Rectangle` with independent setters is the textbook one.

4. **ISP in practice?** Split a fat 20-method `UserService` into `UserReader` / `UserWriter` / `UserAuthenticator` so clients depend only on the methods they call. *Gotcha:* a client forced to depend on (and stub) methods it never uses signals an ISP violation.

5. **DIP vs DI?** DIP is the **principle** (high- and low-level modules both depend on abstractions, not concretes). DI (Spring) is one **mechanism** to achieve it — injecting a `UserRepository` interface rather than `new UserRepositoryImpl()`. *Gotcha:* you can do DI without DIP (injecting a concrete class) and DIP without a DI framework.

6. **Inheritance vs composition — default?** Composition (interfaces + delegation); inheritance only for a genuine IS-A. *Why:* inheritance creates tight coupling and the fragile-base-class problem. *Gotcha:* "code reuse" alone is not a reason to inherit — compose instead.

7. **Latency vs throughput?** Latency = time per single operation; throughput = operations per second. They trade off — batching raises throughput but raises per-item latency. *Gotcha:* optimizing one can hurt the other; know which the system actually needs.

8. **Rough latency: RAM vs SSD vs same-DC vs cross-region?** ~100 ns / ~100 µs / ~500 µs / ~150 ms. A cross-region round trip is ~1.5 million× an L1 hit (~1 ns). *Why it matters:* this is *why* caching, locality, and avoiding chatty cross-service calls dominate design.

9. **Cache-aside vs write-through vs write-behind?** Cache-aside = lazy populate on miss (Smart360's default); write-through = write cache + DB synchronously (consistent, slower writes); write-behind = write cache, flush DB async (fast, risk of loss on crash). *Gotcha:* cache-aside can serve stale data until TTL/invalidation.

10. **What is a cache stampede and how do you prevent it?** Many concurrent misses on one hot key all hit the DB at once. Fix with **single-flight** (one fetch, others wait) or **probabilistic early expiry** (refresh before TTL with a random jitter). *Gotcha:* a plain TTL expiry on a hot key is exactly the trigger.

11. **L4 vs L7 load balancing?** L4 routes on TCP/IP (fast, opaque); L7 routes on HTTP path/header/cookie (smarter, more overhead). **Sticky sessions are an anti-pattern** — prefer stateless JWT so any node serves any request. *Gotcha:* stickiness breaks horizontal scaling and clean failover.

12. **CAP — what does PostgreSQL choose?** **CP** — consistency over availability during a network partition. Cassandra / DynamoDB lean **AP** (tunable). *Gotcha:* CAP is a *partition-time* trade-off, not an always-on property — with no partition you get both.

13. **Walk the Smart360 topology in under 3 minutes.** Gateway (JWT + rate limit + routing, stateless) → Authorization (auth/JWT/RBAC, owns users/roles/permissions, publishes events) → Data (CRUD + the 60s→2-3s fix, `@EntityGraph` for N+1, Redis pre-signed-URL cache) → Visualization (CQRS read side, scales independently) → Notification (event-driven, fire-and-forget). *Gotcha:* call out sync vs async arrows and each service's failure mode.

14. **A technical decision you regret.** Choosing **Eureka over K8s-native service discovery** in Smart360 — redundant once K8s arrived. *Lesson:* don't duplicate what the platform already provides. *Gotcha:* naming a real regret + the lesson signals senior judgment, not weakness.

15. **How did you cut Smart360 latency 96%?** Eliminated N+1 Hibernate queries via `@EntityGraph`, added composite indexes, and cached S3 pre-signed URLs in Redis → 60s down to 2–3s (and ~80% fewer S3 calls). *Gotcha:* lead with the *number* and the *named decisions*, not generic "optimized queries."

16. **Why PostgreSQL RLS over app-layer tenant filtering?** RLS is enforced by the DB even if the app has a bug — defense in depth for multi-tenant isolation. *Trade-off:* you must set the tenant context per connection. *Gotcha:* test with Testcontainers — H2 lacks RLS, so unit tests against H2 won't catch RLS issues.

17. **LLM job runs 20 min and the user closes the browser — what happens?** The job is persisted in PostgreSQL, the worker keeps running, and the user re-polls `GET /jobs/{id}`. The SSE stream drops but the **result is durable**. *Why:* the `202 + jobId` async pattern decouples the request lifecycle from the work.

18. **How would you push CI/CD from 10 min to 5?** More aggressive Docker/test layer caching, test-impact analysis (run only affected tests), more parallel runners, and move slow integration tests to a nightly run. *Gotcha:* parallelism has diminishing returns once the critical path is a single long job.

19. **`@Transactional` self-invocation — why no transaction?** A same-class internal method call bypasses the Spring **proxy**, so the transactional advice never runs. Fix by injecting self, calling through the proxy (`AopContext.currentProxy()`), or restructuring into another bean. *Gotcha:* the method *looks* transactional but silently isn't.

20. **`volatile` vs `AtomicInteger`?** `volatile` gives **visibility** (reads/writes seen across threads) but not compound-action atomicity, so `count++` (read-modify-write) still races. `AtomicInteger` gives **CAS-based atomic** increments. *Gotcha:* `volatile` is enough for a one-writer flag, not for counters.

---

*[← Week 1](README.md)*
