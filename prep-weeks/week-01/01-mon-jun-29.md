# Week 1 · Day 1 — Mon Jun 29 — Two Pointers + equals/hashCode

> The opposite-ends two-pointer pattern collapses many O(n²) array scans to O(n) with O(1) space — and `equals`/`hashCode` is the contract that decides whether your objects can even live in a `HashSet`/`HashMap`.

📌 **Study today:** Two Pointers — opposite ends converging (LC 1 · 125 · 11 · 42 · 167) · Java `equals`/`hashCode` contract · Two-Sum variants + prefix-sum intro · ⏱ ~6 hr (Block A ~2.5h · Block B ~2h · Block C ~1.5h)

---

## Block A — DSA: Two Pointers (opposite ends converging)

### Theory — why the pattern works

A **two-pointer** scan places one index at each end of a (usually sorted) array and walks them toward each other. The whole technique rests on a single idea: **at every step, one of the two pointers can be eliminated from further consideration without losing the optimal answer.** If you can prove that, the inner loop disappears and an O(n²) double loop becomes a single O(n) pass.

The mental model has three parts:

1. **The invariant** — a sentence that is true at the top of every loop iteration. For Two Sum II it is: *"the answer, if it exists, lies within `[left, right]`."* For Container With Most Water it is: *"the best area using any line outside `[left, right]` has already been considered."*
2. **The decision rule** — given the invariant, which pointer do I move, and why does moving it never discard the optimum? This is the part interviewers probe. "Why the shorter line?" is the whole question.
3. **The termination** — pointers cross (`left >= right`) or meet a found condition.

**When to reach for it:** sorted (or sortable) array; you are looking for a *pair/triplet* with a target relationship (sum, product, max area); or you are partitioning/reversing in place. If the array is unsorted and order doesn't matter, a hash set is often the better tool (see Block C).

**Complexity:** time O(n) for the scan (O(n log n) if you must sort first); space O(1) — this is the headline advantage over hashing.

**Common pitfalls:**
- Moving the wrong pointer (e.g., the taller line in Container With Most Water) — silently returns a wrong answer, not a crash.
- Off-by-one at the crossing (`while (left < right)` vs `<=`) — for *pairs* you want `<` (a pair needs two distinct indices); for some scans `<=`.
- Forgetting to skip duplicates (matters Wednesday for 3Sum, but the habit starts here).
- Integer overflow in `left + right` for midpoint-style problems — use `left + (right - left) / 2`.

### Worked example 1 — Two Sum II, sorted (LC 167)

Sorted array, return the 1-based indices of the two numbers that add to `target`.

```java
// Two pointers on a SORTED array — O(n) time, O(1) space.
int[] twoSumSorted(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) {
            return new int[]{left + 1, right + 1}; // 1-based per LC 167
        } else if (sum < target) {
            left++;   // need a bigger sum → smallest element is too small
        } else {
            right--;  // need a smaller sum → largest element is too big
        }
    }
    return new int[]{-1, -1}; // problem guarantees a solution; defensive return
}
```

**Why moving `left` on `sum < target` is safe:** `nums[left]` is the smallest value still in play. Paired with the largest (`nums[right]`) it's still too small, so it can *never* reach `target` with anything ≤ `nums[right]`. Every pair involving `nums[left]` is dead — eliminate it by `left++`. Symmetric argument for `right--`. **Each step removes one element from the candidate set, so we make at most `n` moves → O(n).**

### Worked example 2 — Trapping Rain Water (LC 42)

The hardest of the day. Given bar heights, how much water is trapped after rain?

**Key fact:** water above bar `i` = `min(maxLeft[i], maxRight[i]) - height[i]` (clamped at 0). A bar holds water up to the *shorter* of the tallest wall to its left and the tallest wall to its right.

**(a) Prefix/suffix max — O(n) time, O(n) space (build intuition first):**

```java
int trapPrefix(int[] h) {
    int n = h.length;
    if (n == 0) return 0;
    int[] maxL = new int[n], maxR = new int[n];
    maxL[0] = h[0];
    for (int i = 1; i < n; i++) maxL[i] = Math.max(maxL[i - 1], h[i]);
    maxR[n - 1] = h[n - 1];
    for (int i = n - 2; i >= 0; i--) maxR[i] = Math.max(maxR[i + 1], h[i]);
    int water = 0;
    for (int i = 0; i < n; i++) water += Math.min(maxL[i], maxR[i]) - h[i];
    return water;
}
```

**(b) Two-pointer — O(n) time, O(1) space (the version to know cold):**

```java
int trap(int[] h) {
    int left = 0, right = h.length - 1;
    int maxL = 0, maxR = 0, water = 0;
    while (left < right) {
        if (h[left] <= h[right]) {
            // maxL is the true left-bound for `left` because the right wall
            // (h[right]) is known to be at least as tall.
            maxL = Math.max(maxL, h[left]);
            water += maxL - h[left];
            left++;
        } else {
            maxR = Math.max(maxR, h[right]);
            water += maxR - h[right];
            right--;
        }
    }
    return water;
}
```

**The crux:** when `h[left] <= h[right]`, the limiting wall for index `left` is `maxL`, *not* `maxR` — because we already know there exists a right wall (`h[right]`) at least as tall as `h[left]`, so `min(maxL, maxR)` is governed by `maxL`. That lets us commit water for `left` without ever computing the full right-max array. We advance the pointer on the **shorter side**, the same "shorter wins" instinct as Container With Most Water.

**Complexity walk-through:** each iteration advances exactly one pointer; they cross after `n - 1` moves → O(n). Two scalars (`maxL`, `maxR`) → O(1).

### Practice set

- **Two Sum (LC 1)** *(Easy)* · brute force O(n²); HashMap O(n) one-pass — store `value → index`, check `target - x` before inserting. · **Key insight:** the hash version doesn't need sorting; the two-pointer version does — pick based on whether indices or space matter. · O(n)/O(n). · *Follow-up:* "What if the array is already sorted and you can't use extra space?" → two pointers, O(1) space (that's LC 167).

- **Valid Palindrome (LC 125)** *(Easy)* · two pointers from both ends, skip non-alphanumerics with `Character.isLetterOrDigit`, compare lowercased. · **Key insight:** convergence works on *characters*, not just numbers. · O(n)/O(1). · *Follow-up:* "Unicode / emoji?" → iterate by code point: `s.codePointAt(i)` + `Character.isLetterOrDigit(cp)` + advance by `Character.charCount(cp)`; a surrogate pair is two `char`s but one code point.

- **Container With Most Water (LC 11)** *(Medium)* · two pointers at the ends; area = `min(h[l], h[r]) * (r - l)`; always move the **shorter** pointer inward. · **Key insight:** width only shrinks as pointers converge, so the only way to beat the current area is a taller bottleneck — and moving the taller line can never help (height still capped by the shorter). · O(n)/O(1). · *Follow-up:* "Prove moving the shorter pointer is correct." → moving the taller line keeps the shorter line as the height cap while reducing width ⇒ strictly worse; so the taller line's pairing with the current shorter line is already maximal — discard it by moving the shorter one.

- **Trapping Rain Water (LC 42)** *(Hard)* · two-pointer O(1)-space as above; advance the shorter side and bank `maxSide - h[i]`. · **Key insight:** `min(maxL, maxR)` is fixed by the shorter wall, so you only ever need one running max at a time. · O(n)/O(1). · *Follow-up:* "Now in 2D (LC 407, Trapping Rain Water II)?" → BFS from the border inward with a min-heap of boundary cells; water level rises from the lowest boundary.

- **Two Sum II — sorted (LC 167)** *(Medium)* · worked above; the case where two pointers strictly beat HashMap (O(1) space). · **Key insight:** sortedness *is* the correctness invariant — drop it and the elimination argument collapses. · O(n)/O(1). · *Follow-up:* "Return *all* pairs that sum to target (with duplicates)." → after a hit, move both pointers and skip equal neighbors on each side.

- **Valid Palindrome II (LC 680)** *(Easy)* · two pointers; on first mismatch, try skipping the left char OR the right char and verify the remaining substring is a palindrome. · **Key insight:** one allowed deletion = one allowed branch; you don't need to try deletions everywhere, only at the first mismatch. · O(n)/O(1). · *Follow-up:* "Allow up to `k` deletions." → this becomes a DP (longest palindromic subsequence) — the greedy single-branch trick no longer suffices.

---

## Block B — Core Java: the `equals` / `hashCode` contract

### Theory — the contract and why it exists

Hash-based collections (`HashMap`, `HashSet`, `LinkedHashMap`, `ConcurrentHashMap`) find an object in two steps: compute `hashCode()` to pick a **bucket**, then walk that bucket calling `equals()` to find the exact match. If two equal objects produce different hash codes, they land in different buckets and the collection can **never** match them. Hence the contract:

1. **Consistency with equals:** if `a.equals(b)` is `true`, then `a.hashCode() == b.hashCode()` **must** hold.
2. **The converse is NOT required:** equal hash codes with `equals == false` is fine — that's a *collision*, and the bucket's `equals` walk resolves it correctly (just slightly slower).
3. **`equals` must be** reflexive, symmetric, transitive, consistent, and `x.equals(null) == false`.
4. **`hashCode` must be stable** for the lifetime an object sits in a hash structure (return the same value across calls as long as the fields used in `equals` don't change).

A constant `hashCode` (e.g. `return 1;`) is *legal* — it never violates the contract — but pathological: every element lands in one bucket, so lookups degrade to O(n) (O(log n) after treeification; see Tuesday). Correctness preserved, performance destroyed.

### Code — implement it by hand

```java
import java.util.Objects;

public final class User {
    private final Long id;
    private final String email;

    public User(Long id, String email) {
        this.id = id;
        this.email = email;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;                     // fast identity path
        if (o == null || getClass() != o.getClass()) return false; // null + exact-type
        User other = (User) o;
        return Objects.equals(id, other.id)
            && Objects.equals(email, other.email);      // null-safe field compare
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, email); // same fields as equals — non-negotiable
    }
}
```

`Objects.equals(a, b)` handles nulls (`a == null ? b == null : a.equals(b)`). `Objects.hash(...)` builds a combined hash via `31 * result + c` folding — the same polynomial idea as `String.hashCode()`.

> **`getClass()` vs `instanceof`:** `getClass()` makes `equals` *not* symmetric across subclasses but keeps the contract airtight (a `User` is never equal to a `PremiumUser`). `instanceof` allows subclass-to-superclass equality but you must keep the relation symmetric — most teams use `getClass()` for value classes.

### Gotchas

- **Mutating a field used in `hashCode` while the object is in a set** is the headline bug. The object moves logically (its hash changed) but physically stays in the old bucket → `set.contains(obj)` returns `false` though the object is *right there*. It is "ghost-present": you can iterate to it but never look it up.
- **Override one, override both.** Overriding `equals` without `hashCode` (or vice versa) breaks hash collections silently.
- **Don't include lazy/mutable collections** in `hashCode` for JPA entities — accessing a lazy association from `hashCode` can trigger a database query (or a `LazyInitializationException`) at surprising times.

### Production tie-in — Smart360 JPA entities

In Smart360, `Role` and `Permission` are JPA entities stored in `Set<Role>` on `User`. A real bug class here:

- Using **default identity `equals`** breaks when the same logical role is fetched in two Hibernate sessions — two distinct object instances, `Set` treats them as different, you get duplicates.
- Using **all fields in `equals`/`hashCode`** breaks because a freshly-`new`ed transient entity has `id == null`, then Hibernate assigns the PK on flush → its `hashCode` *changes while it's in a `Set`* → ghost-present bug.

**The JPA-safe recipe:**

```java
@Entity
public class Role {
    @Id @GeneratedValue private Long id;
    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Role)) return false;     // proxy-friendly: instanceof, not getClass
        Role role = (Role) o;
        return id != null && id.equals(role.id);    // null id ⇒ only equal to itself (this==o)
    }

    @Override
    public int hashCode() {
        return getClass().hashCode();               // CONSTANT — never changes when id is assigned
    }
}
```

A **constant `hashCode`** is the standard JPA trick: it never changes when the PK is assigned (no ghost-present bug), and `equals` discriminates by `id`. With Lombok this is `@EqualsAndHashCode(onlyExplicitlyIncluded = true)` plus `@EqualsAndHashCode.Include` on the PK only — by default Lombok uses *all* fields, which is wrong for entities.

> Note `instanceof` (not `getClass()`) here because Hibernate hands you **proxy subclasses** whose `getClass()` is `Role$HibernateProxy$xyz`, not `Role` — `getClass()` equality would fail proxy-vs-entity comparisons.

### Live-code test (you will write this in an interview)

```java
import java.util.HashSet;
import java.util.Set;

class BrokenKey {
    int id;
    BrokenKey(int id) { this.id = id; }
    @Override public boolean equals(Object o) {
        return o instanceof BrokenKey && ((BrokenKey) o).id == id;
    }
    @Override public int hashCode() { return id; } // depends on a MUTABLE field
}

public class GhostDemo {
    public static void main(String[] args) {
        Set<BrokenKey> set = new HashSet<>();
        BrokenKey k = new BrokenKey(1);
        set.add(k);
        System.out.println(set.contains(k)); // true — found in bucket 1
        k.id = 2;                            // mutate the hashCode field while in the set
        System.out.println(set.contains(k)); // false — now hashes to bucket 2, but lives in bucket 1
        System.out.println(set.iterator().next() == k); // true — it's STILL in the set!
    }
}
```

This three-line demonstration — `true`, then `false`, then proof it's still there — is the canonical whiteboard answer to "what breaks if you mutate a hashCode field?"

---

## Block C — Two-Sum variants + prefix-sum intro

### Two Sum as a stream

When the input isn't a fixed array but an unbounded stream (you can't sort, can't index), the hash approach is the only one that works:

```java
// For a stream of ints, detect a pair summing to target as elements arrive.
boolean seenPair(Iterable<Integer> stream, int target) {
    Set<Integer> seen = new HashSet<>();
    for (int x : stream) {
        if (seen.contains(target - x)) return true; // complement arrived earlier
        seen.add(x);                                 // add AFTER the check (avoids x+x false-positive)
    }
    return false;
}
```

**Why check before add:** if you add first, `target = 2*x` would falsely match `x` against itself. O(n) time, O(n) space — the space is the price of giving up sortedness.

### Prefix sum — the foundation for Tuesday & Wednesday

Define `prefix[i] = nums[0] + nums[1] + ... + nums[i-1]`, with `prefix[0] = 0`. Then the sum of any subarray `nums[i..j]` (inclusive) is:

```
sum(i..j) = prefix[j + 1] - prefix[i]
```

This turns any range-sum query into **O(1)** after an **O(n)** preprocessing pass — the array equivalent of an integral.

```java
// Build a prefix-sum array; prefix has length n+1, prefix[0] = 0.
int[] buildPrefix(int[] nums) {
    int[] prefix = new int[nums.length + 1];
    for (int i = 0; i < nums.length; i++) {
        prefix[i + 1] = prefix[i] + nums[i];
    }
    return prefix;
}

// O(1) range sum for nums[i..j] inclusive, using the prefix array above.
int rangeSum(int[] prefix, int i, int j) {
    return prefix[j + 1] - prefix[i];
}
```

This sets up **Subarray Sum Equals K (LC 560)** on Tuesday/Wednesday: instead of an array we keep a `HashMap<prefixSum, count>` and look for `prefix - k` — converting an O(n²) subarray scan into O(n). Today, just internalize the index arithmetic: the `+1` offset (so an empty prefix exists) is what makes the formula clean and what later justifies the `map.put(0, 1)` seed.

**Production tie-in:** prefix sums are the algorithmic cousin of SQL window functions — `SUM(amount) OVER (ORDER BY ts)` is a server-side prefix sum. In Smart360 analytics, pushing a running total to PostgreSQL's window functions beats pulling rows and summing in Java.

---

## 💻 Practice coding questions

1. **Two Sum (LC 1)** — HashMap one-pass: `value → index`; check complement before inserting. O(n)/O(n).
2. **Two Sum II, sorted (LC 167)** — converge from both ends; `left++` if sum too small, `right--` if too big. O(n)/O(1).
3. **Valid Palindrome (LC 125)** — skip non-alphanumerics, compare lowercased ends. O(n)/O(1).
4. **Valid Palindrome II (LC 680)** — on first mismatch branch into "skip left" or "skip right" and verify. O(n)/O(1).
5. **Container With Most Water (LC 11)** — area `min(h[l],h[r])*(r-l)`; move the shorter pointer. O(n)/O(1).
6. **Trapping Rain Water (LC 42)** — two pointers, advance the shorter side, bank `maxSide - h[i]`. O(n)/O(1).
7. **Move Zeroes (LC 283)** — slow/fast pointer: `slow` marks the next non-zero slot, `fast` scans; swap. O(n)/O(1). (A *same-direction* two-pointer — contrast with converging.)
8. **Remove Duplicates from Sorted Array (LC 26)** — slow/fast; write unique values at `slow`. O(n)/O(1).
9. **Reverse String (LC 344)** — swap `s[left]` and `s[right]`, converge. O(n)/O(1).
10. **Squares of a Sorted Array (LC 977)** — array can be negative; fill the result *from the back* by comparing `abs(left)` vs `abs(right)`. O(n)/O(n).
11. **3Sum (LC 15) — preview** — sort, fix one, two-pointer the rest, skip dups. (Full treatment Wednesday.) O(n²)/O(1).
12. **Sort Colors / Dutch National Flag (LC 75)** — three pointers `low/mid/high`; one pass partitions 0/1/2. O(n)/O(1).
13. **Two Sum stream** — `HashSet`, check `target - x` before add. O(n)/O(n).
14. **Range sum queries (build prefix array)** — `prefix[j+1]-prefix[i]` in O(1) after O(n) prep.

---

## 🎤 Interview questions

1. **"When do two pointers beat a HashMap for Two Sum?"** — When the array is sorted (or you can sort), because two pointers use **O(1) space** vs the hash map's O(n); the trade-off is the O(n log n) sort cost if it wasn't already sorted.
2. **"Prove that moving the shorter pointer is correct in Container With Most Water."** — Width strictly decreases as pointers converge; the height is capped by the shorter line; moving the taller line keeps that same cap but shrinks width ⇒ strictly worse, so the taller line is already maximal with the current shorter one — eliminate the shorter.
3. **"In Trapping Rain Water, why do you only need one running max at a time?"** — Water at `i` is `min(maxL, maxR) - h[i]`; when `h[left] <= h[right]` you *know* the right wall is at least as tall, so the `min` is governed by `maxL`, letting you commit water using only the left running max.
4. **"State the equals/hashCode contract."** — `a.equals(b) ⇒ a.hashCode() == b.hashCode()`. The converse need not hold; unequal objects may share a hash (collision). `equals` must be reflexive, symmetric, transitive, consistent, and false against null.
5. **"What breaks in a HashSet if you violate the contract?"** — If equal objects hash differently they land in different buckets and the set treats them as distinct (duplicates allowed in, lookups miss). If you mutate a hashCode field after insertion, the object becomes ghost-present: still in the set, but `contains` returns false.
6. **"Two objects, same hashCode, equals false — what happens in a HashMap?"** — Legal collision: both go to the same bucket as separate chain nodes; the bucket's `equals` walk keeps them distinct. Correct, just an O(n)/O(log n) bucket walk instead of O(1).
7. **"Is a constant hashCode (`return 1`) ever legal?"** — Yes, it satisfies the contract (equal objects trivially share it). But every element collides into one bucket → O(n) (O(log n) after treeify) lookups. Legal, never advisable.
8. **"Why use a constant hashCode for JPA entities specifically?"** — A generated PK is `null` while transient and assigned on flush. If `hashCode` depended on `id`, it would change *while the entity sits in a `Set`*, causing the ghost-present bug. A constant hash + id-based `equals` is stable across the transient→persistent transition.
9. **"Why `instanceof` instead of `getClass()` in a JPA entity's equals?"** — Hibernate returns proxy subclasses (`Role$HibernateProxy$...`); `getClass()` equality fails proxy-vs-real comparisons, `instanceof` tolerates them.
10. **"What does Lombok `@EqualsAndHashCode` do by default, and why is that wrong for entities?"** — By default it uses *all non-static fields*, including mutable ones and associations. For entities use `@EqualsAndHashCode(onlyExplicitlyIncluded = true)` and include only the PK.
11. **"Valid Palindrome with emoji/Unicode — how does the code change?"** — Iterate by code point (`codePointAt` + `Character.charCount`) rather than `char`, because a surrogate pair is two `char`s but one logical character.
12. **"Move Zeroes / Remove Duplicates use two pointers but they don't converge — what's the variant?"** — Same-direction (slow/fast) pointers: `fast` scans, `slow` marks the write position. Different invariant ("everything before `slow` is finalized") than converging pointers.
13. **"How is `Objects.hash(a, b)` computed?"** — A folding loop `result = 31 * result + Objects.hashCode(x)` over the args — the same polynomial used by `String.hashCode`. Note `Objects.hash(x)` *boxes into a varargs array*; for a single hot field prefer `x.hashCode()` directly.
14. **"Why does `String` make an ideal HashMap key?"** — It's immutable (hash can't drift), and its `hashCode` is cached after first computation, so repeated lookups don't recompute.
15. **"Squares of a Sorted Array — why fill from the back?"** — The largest square is at one of the two ends (most-negative or most-positive); filling the result from its highest index lets a single converging pass stay O(n) without a final sort.
16. **"What's the difference between reflexive/symmetric/transitive in the equals contract — give a symmetry violation?"** — A `getClass()`-vs-`instanceof` mismatch across a subclass: if `Point.equals` uses `instanceof` and `ColorPoint.equals` adds a color check, `point.equals(colorPoint)` can be true while `colorPoint.equals(point)` is false — asymmetric.

---

## ✅ Self-check

1. Given a sorted array, explain in one sentence why two pointers beat a HashMap for Two Sum. *(O(1) space vs O(n); sortedness is the correctness invariant enabling pointer elimination.)*
2. Mutate a `User`'s `id` after putting it in a `HashSet<User>` — what does `contains` return and why? *(false — the object now hashes to a different bucket than the one it physically occupies: ghost-present.)*
3. Write the two-pointer move rule for Container With Most Water and justify it in under 60 seconds. *(Move the shorter line; the taller line is already maximal against the current shorter one because width only shrinks.)*
4. Without looking, write the prefix-sum range formula and explain the `+1` offset. *(`sum(i..j) = prefix[j+1] - prefix[i]`; `prefix[0]=0` lets the formula include subarrays starting at index 0.)*
5. Give the JPA-safe `equals`/`hashCode` recipe in two lines. *(`equals`: `id != null && id.equals(other.id)`; `hashCode`: constant `getClass().hashCode()`.)*

---
*Nav: ← [Week 0](../week-00-basics/README.md) · [Week 1](README.md) · [next →](02-tue-jun-30.md)*
