# Week 1 · Day 3 — Wed Jul 01 — 3Sum / Prefix Sum + Generics & Autoboxing

> The 3Sum family is "sort, fix one, two-pointer the rest" — and generics/type-erasure/autoboxing is where confident-looking Java code hides real production NPEs and GC pressure.

📌 **Study today:** Multi-pointer 3Sum family + prefix/array tricks (LC 15 · 560 · 238 · 189) · Generics, type erasure & autoboxing · hashing with canonical keys (LC 242 · 49) · ⏱ ~6 hr (Block A ~2.5h · Block B ~2h · Block C ~1.5h)

---

## Block A — DSA: Multi-pointer (3Sum family) + prefix/array tricks

### Theory — reduce-to-two-pointer

The 3Sum insight is **dimensional reduction**: a triplet problem becomes a *pair* problem once you fix one element. Sort the array, fix `nums[i]`, then run yesterday's converging two-pointer over `[i+1, n-1]` looking for `-nums[i]`. Sorting is what unlocks both the pointer-elimination logic *and* clean duplicate-skipping (equal values are adjacent).

**The invariant per fixed `i`:** "the remaining pair, if any, summing to `target = -nums[i]` lies within `[left, right]`." Identical to Two Sum II — you've just wrapped it in an outer loop.

**Why sorting helps three ways:** (1) enables two-pointer elimination on the inner range; (2) makes duplicates adjacent so you can skip them in O(1); (3) allows early termination (`if nums[i] > 0` for 3Sum-to-zero, no positive triplet can sum to zero).

**Complexity:** sort O(n log n); outer loop n × inner two-pointer O(n) → **O(n²)** total, **O(1)** extra space (ignoring the sort's recursion and the output list). You cannot beat O(n²) for 3Sum in the general case (it's 3SUM-hard).

**Common pitfalls:**
- **Forgetting to skip duplicates in three places**: the outer `i`, and *both* inner pointers *after* recording a valid triplet. This is the #1 bug.
- Skipping the outer duplicate with the wrong guard (`nums[i] == nums[i-1]`, and only for `i > start`).
- Moving only one inner pointer after a hit (must move both — a valid pair is consumed).

### Worked example 1 — 3Sum (LC 15)

```java
// All unique triplets summing to 0. O(n^2) time, O(1) extra (output excluded).
List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);                         // O(n log n) — enables two-pointer + dup skipping
    List<List<Integer>> res = new ArrayList<>();
    for (int i = 0; i < nums.length - 2; i++) {
        if (nums[i] > 0) break;                // sorted: no positive can start a zero-sum triplet
        if (i > 0 && nums[i] == nums[i - 1]) continue; // skip duplicate FIXED element
        int left = i + 1, right = nums.length - 1, target = -nums[i];
        while (left < right) {
            int sum = nums[left] + nums[right];
            if (sum == target) {
                res.add(List.of(nums[i], nums[left], nums[right]));
                left++; right--;               // BOTH move — the pair is consumed
                while (left < right && nums[left] == nums[left - 1]) left++;   // skip dup left
                while (left < right && nums[right] == nums[right + 1]) right--; // skip dup right
            } else if (sum < target) {
                left++;                         // need bigger
            } else {
                right--;                        // need smaller
            }
        }
    }
    return res;
}
```

**Complexity walk-through:** the sort is O(n log n). The outer loop runs n times; for each, the inner two-pointer scans the remaining range once → O(n) per outer iteration → O(n²) total, which dominates the sort. Output aside, only a handful of scalars → O(1) extra.

### Worked example 2 — Product of Array Except Self (LC 238)

No division allowed; `out[i]` = product of everything *except* `nums[i]`. Famous Meta/Google question.

```java
// out[i] = product of all elements except nums[i]. O(n) time, O(1) extra (output excluded).
int[] productExceptSelf(int[] nums) {
    int n = nums.length;
    int[] out = new int[n];
    out[0] = 1;
    for (int i = 1; i < n; i++) out[i] = out[i - 1] * nums[i - 1]; // prefix product (left of i)
    int suffix = 1;
    for (int i = n - 1; i >= 0; i--) {
        out[i] *= suffix;          // multiply in product of everything to the RIGHT of i
        suffix *= nums[i];
    }
    return out;
}
```

**Why no division:** division fails when any element is 0 (you'd divide by zero) and is also banned to force the prefix/suffix insight. First pass fills `out[i]` with the product of everything to the *left*; second pass multiplies in the running product from the *right*. Two passes, O(n), and the output array doesn't count as extra space.

### Practice set

- **3Sum (LC 15)** *(Medium)* · sort, fix `i`, two-pointer the rest, skip dups in 3 places. · **Key insight:** fixing one element reduces 3Sum to Two Sum II. · O(n²)/O(1). · *Follow-up:* "3Sum Closest (LC 16)?" → same scaffold but track the triplet whose sum is nearest target instead of exactly zero.

- **Subarray Sum Equals K (LC 560)** *(Medium)* · prefix sum + `HashMap<prefix,count>`, seed `{0:1}`, add `count[prefix - k]`. · **Key insight:** the `{0:1}` seed captures subarrays starting at index 0. · O(n)/O(n). · *Follow-up:* "Subarrays with sum *divisible* by K (LC 974)?" → key on `((prefix % k) + k) % k` (normalize negatives), count pairs with equal remainder.

- **Product of Array Except Self (LC 238)** *(Medium)* · prefix product then suffix product in place. · **Key insight:** left-product pass then right-product pass avoids division and stays O(1) extra. · O(n)/O(1). · *Follow-up:* "What if division *were* allowed?" → total product / nums[i], but must special-case one or more zeros.

- **Rotate Array (LC 189)** *(Medium)* · three-reversal: reverse all, reverse first `k`, reverse last `n−k`. · **Key insight:** reversing twice realigns the rotated halves without extra space. · O(n)/O(1). · *Follow-up:* "Rotate left by k?" → equivalent to rotate right by `n - k`; or reverse first k, reverse rest, reverse all.

- **3Sum Closest (LC 16)** *(Medium)* · sort + fix + two-pointer; track `bestSum` minimizing `abs(sum - target)`. · **Key insight:** same elimination, different objective (closest vs exact). · O(n²)/O(1). · *Follow-up:* "Early exit?" → if `sum == target`, return immediately; it can't be beaten.

- **4Sum (LC 18)** *(Medium)* · two nested fixes + two-pointer; skip dups at all four levels; guard overflow with `long`. · **Key insight:** the reduction generalizes — kSum recurses down to 2Sum-two-pointer. · O(n³)/O(1). · *Follow-up:* "General kSum?" → recursive: fix-and-recurse until k=2, base case is two-pointer.

---

## Block B — Core Java: Generics, type erasure & autoboxing

### Theory — type erasure

Java generics are a **compile-time** feature. After the compiler type-checks, it **erases** type parameters: `List<String>` and `List<Integer>` both become plain `List` at runtime (their bytecode is identical). Erasure exists for **backward compatibility** — pre-generics `.class` files and code keep working. Consequences you must know:

- **No runtime generic type info:** `list instanceof List<String>` is illegal; you can only check `instanceof List`.
- **No `new T[]`** and no `new T()` — the type isn't known at runtime. Workaround: pass a `Class<T>` token or `T[]` and use `Array.newInstance`.
- **No overload on erasure-equal signatures:** `void f(List<String>)` and `void f(List<Integer>)` collide — both erase to `f(List)`.
- **Bridge methods** are synthesized to preserve polymorphism after erasure.
- **Generic exceptions:** you can't `catch (T e)` or have a generic class extend `Throwable`.

### Bounded wildcards — PECS

**Producer Extends, Consumer Super.** When a structure *produces* values you read, use `? extends T` (covariant, read-only). When it *consumes* values you write, use `? super T` (contravariant, write-only).

```java
// PECS in the JDK signature:
// dest CONSUMES T -> super; src PRODUCES T -> extends
static <T> void copy(List<? super T> dest, List<? extends T> src) { ... }

// Read from a producer (you get out a T or supertype):
double sum(List<? extends Number> nums) {        // can pass List<Integer>, List<Double>
    double s = 0;
    for (Number n : nums) s += n.doubleValue();   // read OK
    // nums.add(1); // COMPILE ERROR — can't write to a ? extends list
    return s;
}

// Write to a consumer:
void fill(List<? super Integer> dst) {            // can pass List<Integer>, List<Number>, List<Object>
    dst.add(42);                                   // write OK
    // Integer x = dst.get(0); // only Object comes out — read is limited
}
```

**Why `List<Dog>` is not a `List<Animal>`:** if it were, you could assign `List<Animal> a = dogs; a.add(new Cat());` and corrupt the dog list. Java's generics are **invariant** for exactly this safety; wildcards give back the flexibility *safely*.

### Autoboxing pitfalls — real production bugs

- **The Integer cache (−128…127):** `Integer.valueOf` returns cached instances for −128…127, so `==` (reference equality) is `true` in that range and `false` outside it. **Always use `.equals()` on boxed numbers.**
  ```java
  Integer a = 127, b = 127;  System.out.println(a == b); // true  (cached)
  Integer c = 128, d = 128;  System.out.println(c == d); // false (new objects)
  System.out.println(c.equals(d));                        // true  (always correct)
  ```
  The upper bound is tunable via `-XX:AutoBoxCacheMax`, lower bound fixed at −128.

- **Box/unbox in tight loops = GC pressure:**
  ```java
  Long sum = 0L;                       // BAD: boxes/unboxes EVERY iteration
  for (long x : data) sum += x;        // each += creates a new Long object
  // GOOD:
  long total = 0L;                     // primitive accumulator — zero allocations
  for (long x : data) total += x;
  // or:  long s = LongStream.of(data).sum();
  ```

- **Null unboxing → NPE:** the silent killer with nullable DB columns.
  ```java
  Integer count = row.getInteger("count"); // null if column is NULL
  int c = count;                            // NPE on auto-unboxing null!
  // GOOD:
  int safe = Objects.requireNonNullElse(count, 0);
  ```

- **Map-key boxing mismatch:** `map.get(longPrimitive)` autoboxes to `Long`, while `map.get(intPrimitive)` boxes to `Integer` — a `Map<Long,V>` lookup with an `int` key *silently misses* because `Integer.equals(Long)` is false. Be explicit: `map.get(Long.valueOf(id))`.

**Practical rule:** primitives in hot paths and numeric accumulators; wrappers only where `null` is semantically meaningful (optional/absent values).

### Production tie-in — Smart360 / Deep Fathom / WebX

- **Erasure enables Spring DI:** `@Autowired List<LLMProvider> providers` works because the parameter's generic type is erased but Spring's container resolves the concrete bean types from its metadata and injects all `LLMProvider` beans — WebX's provider router relies on exactly this to discover every provider implementation.
- **Null-unboxing in Deep Fathom:** nullable analytic columns mapped to `Integer`/`Long` are the classic source of intermittent NPEs in report aggregation — guarded with `requireNonNullElse` / `Optional`.
- **Autoboxing GC in hot paths:** Smart360's per-request metric accumulation uses `long`/`LongAdder`, not `Long`, to avoid per-event allocations under load.

---

## Block C — Hashing: frequency maps & canonical keys

### Valid Anagram (LC 242)

Two strings are anagrams iff they have identical character frequencies.

```java
// O(n) time, O(1) space (fixed 26-letter alphabet).
boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] count = new int[26];
    for (int i = 0; i < s.length(); i++) {
        count[s.charAt(i) - 'a']++;
        count[t.charAt(i) - 'a']--;   // increment for s, decrement for t in one pass
    }
    for (int c : count) if (c != 0) return false; // all-zero ⇒ identical frequencies
    return true;
}
```

**Curveball — Unicode:** `int[26]` assumes lowercase ASCII. For arbitrary Unicode, use a `HashMap<Integer, Integer>` keyed by **code point** (iterate via `codePointAt` / `charCount`), since a single logical character may be a surrogate pair.

### Group Anagrams (LC 49) — canonical keys

Group words that are anagrams of each other. The trick is a **canonical key**: a representation identical for all anagrams of the same multiset of letters.

```java
// Canonical key via frequency signature — O(n * k) where k = avg word length, O(26) per key.
List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();
    for (String w : strs) {
        int[] count = new int[26];
        for (char ch : w.toCharArray()) count[ch - 'a']++;
        String key = Arrays.toString(count);   // e.g. "[1, 0, 2, ...]" — same for all anagrams
        groups.computeIfAbsent(key, k -> new ArrayList<>()).add(w);
    }
    return new ArrayList<>(groups.values());
}
```

**Two canonical-key options:** (1) **sort each word** → `O(k log k)` per word; simple, works for any alphabet. (2) **frequency signature** (`Arrays.toString(int[26])`) → `O(k + 26)` per word; avoids the sort and is faster for long words over a fixed alphabet. Mention both; pick the frequency one when the alphabet is small and known.

**Production tie-in:** Smart360's role/permission deduplication uses the **same canonical-key pattern** — a permission set is normalized (sorted/hashed into a signature) so that two roles with the same effective permissions map to one cache entry, avoiding duplicate authorization computations.

---

## 💻 Practice coding questions

1. **3Sum (LC 15)** — sort + fix + two-pointer; skip dups in 3 places. O(n²)/O(1).
2. **3Sum Closest (LC 16)** — track sum nearest target. O(n²)/O(1).
3. **4Sum (LC 18)** — two fixes + two-pointer; `long` to avoid overflow. O(n³)/O(1).
4. **Subarray Sum Equals K (LC 560)** — prefix sum + map, `{0:1}` seed. O(n)/O(n).
5. **Subarray Sums Divisible by K (LC 974)** — key on normalized `prefix % k`. O(n)/O(k).
6. **Product of Array Except Self (LC 238)** — prefix/suffix product, no division. O(n)/O(1).
7. **Rotate Array (LC 189)** — three-reversal trick. O(n)/O(1).
8. **Valid Anagram (LC 242)** — `int[26]` increment/decrement → all-zero. O(n)/O(1).
9. **Group Anagrams (LC 49)** — canonical key (sorted string or `int[26]` signature). O(n·k)/O(n·k).
10. **Find All Anagrams in a String (LC 438)** — fixed window + `int[26]` compare. O(n)/O(1).
11. **Contains Duplicate (LC 217)** — `HashSet` membership; canonical "is hashing the tool" warm-up. O(n)/O(n).
12. **Top K Frequent Elements (LC 347) — preview** — frequency map + bucket sort. O(n)/O(n). (Full Thursday.)
13. **Continuous Subarray Sum (LC 523)** — prefix `% k` remainder map; subarray length ≥ 2. O(n)/O(k).
14. **First Unique Character in a String (LC 387)** — frequency map then scan for count==1. O(n)/O(1).

---

## 🎤 Interview questions

1. **"Why does `List<Dog>` not satisfy `List<Animal>` though `Dog extends Animal`?"** — Generics are invariant for safety: if it did, you could `add(new Cat())` to a list typed as `List<Animal>` that's really a `List<Dog>`, corrupting it. Wildcards (`? extends`/`? super`) restore flexibility safely.
2. **"Explain type erasure and three things it forbids."** — Generic types exist only at compile time; at runtime `List<String>` is just `List`. Forbidden: `instanceof` on a parameterized type, `new T[]`, and overloads that erase to the same signature.
3. **"What is PECS?"** — Producer Extends, Consumer Super. Read from a producer with `? extends T` (covariant, read-only); write to a consumer with `? super T` (contravariant, write-only). See `Collections.copy(List<? super T> dest, List<? extends T> src)`.
4. **"`Integer a = 128, b = 128; a == b`?"** — `false`. The Integer cache only covers −128…127; outside it `valueOf` makes new objects, so `==` (reference equality) fails. `a.equals(b)` is `true`. Lower bound fixed; upper tunable via `-XX:AutoBoxCacheMax`.
5. **"Write the safe unboxing of a nullable `Integer` DB column."** — `int c = Objects.requireNonNullElse(count, 0);` or guard `if (count != null) { int c = count; }`. Direct `int c = count;` NPEs when null.
6. **"Why does `Long sum = 0L; for (int x: list) sum += x;` hurt performance?"** — Every `+=` unboxes and re-boxes `sum` into a new `Long`, allocating per iteration → GC pressure in tight loops. Use a `long` primitive or `IntStream.sum()`.
7. **"How does Spring inject `@Autowired List<LLMProvider>` despite erasure?"** — The list's generic type is erased in bytecode, but Spring resolves bean types from container metadata (and reads generic info via reflection where available), injecting all matching beans.
8. **"`map.get(id)` where `id` is an `int` and the map is `Map<Long,V>` — why a miss?"** — The `int` autoboxes to `Integer`, and `Integer.equals(Long)` is `false`, so the keys never match. Use `map.get(Long.valueOf(id))`.
9. **"Why can't you create a generic array `new T[n]`?"** — Arrays are reified (carry runtime type), generics are erased — there's no runtime `T` to store, so it's disallowed. Use `(T[]) new Object[n]` with an unchecked cast or `Array.newInstance(clazz, n)`.
10. **"3Sum — where do duplicates get skipped and why three places?"** — The outer fixed element (`nums[i] == nums[i-1]`), and both inner pointers *after* recording a valid triplet, to avoid emitting the same triplet from equal adjacent values.
11. **"Product of Array Except Self without division — approach?"** — Left-to-right prefix products into the output, then a right-to-left pass multiplying in a running suffix product. O(n), O(1) extra. Division is banned because zeros break it.
12. **"Rotate Array by k in O(1) space?"** — Three reversals: reverse the whole array, reverse the first `k`, reverse the last `n−k`. Remember `k %= n`.
13. **"Group Anagrams — sorted-string key vs frequency-array key?"** — Sorted key is O(k log k) per word, alphabet-agnostic; frequency signature (`Arrays.toString(int[26])`) is O(k + 26), faster for a small fixed alphabet. Both produce an identical key for anagrams.
14. **"Subarray Sums Divisible by K — what's the key?"** — The normalized remainder `((prefix % k) + k) % k`; count pairs of prefixes sharing a remainder (their difference is divisible by k). Seed remainder 0 once.
15. **"When should you prefer wrappers over primitives?"** — Only when `null`/absence is meaningful (optional fields, nullable columns, generic collections which can't hold primitives). Otherwise primitives — no boxing, no NPE, no GC churn.
16. **"What is a bridge method?"** — A compiler-synthesized method that preserves polymorphism after erasure — e.g. a generic `compareTo(T)` gets a bridge `compareTo(Object)` so overriding still works through the erased supertype signature.
17. **"Can you overload `f(List<String>)` and `f(List<Integer>)`?"** — No; both erase to `f(List)`, a duplicate-method compile error.

---

## ✅ Self-check

1. Explain why generics are invariant using the `List<Dog>`/`add(Cat)` argument. *(allowing it would let a Cat into a Dog list — type safety lost.)*
2. State the Integer cache boundary and the rule it implies. *(−128…127 cached; always use `.equals()` for boxed comparison.)*
3. Write the safe null-check before unboxing a nullable `Integer`. *(`Objects.requireNonNullElse(x, 0)` or `if (x != null)`.)*
4. List the three places 3Sum skips duplicates. *(outer fixed element; left pointer and right pointer after a valid triplet.)*
5. Give the two canonical-key strategies for Group Anagrams and when to use each. *(sorted string — any alphabet; `int[26]` signature — small fixed alphabet, avoids the sort.)*

---
*Nav: ← [prev](02-tue-jun-30.md) · [Week 1](README.md) · [next →](04-thu-jul-02.md)*
