# Week 1 · Day 5 — Friday Jul 03 — Strings + Comparable/Comparator + Weak-Spot Review

> The "polish and consolidate" day before Saturday's timed gauntlet. Strings get their own block because interviewers love them (immutability, encoding, palindromes) and they're a rich source of subtle bugs. `Comparable` vs `Comparator` is the ordering vocabulary that powers `TreeMap`/`TreeSet`/`sort` — and hides the infamous `BigDecimal` consistency trap. The final block is deliberately self-directed: find your two weakest problems from Mon–Thu and kill them clean.

📌 **Study today:** Strings (LC 5 · 271 · 680) · Comparable vs Comparator (TreeMap/TreeSet sorting) · mixed-pattern review / weak spots · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

> **String-immutability recap (bridging from the Week 0 refresh):** every "modification" returns a *new* `String`; interning works *only because* strings are immutable; build in loops with `StringBuilder`, never `+` (each `+` in a loop allocates a new char array → `O(n²)` garbage). See [01-java-language-basics.md](../../01-java-language-basics.md).

---

## Block A — DSA: String Patterns (~2.5 h)

### Theory deep-dive

Three string sub-patterns dominate interviews:

1. **Expand-around-center** — for palindromes; treat each position (and each gap) as a potential center and grow outward. `O(n²)` time, `O(1)` space — beats the obvious `O(n³)` brute force, and you should *mention* that Manacher's gives true `O(n)` even if you don't code it.
2. **Two-pointer with one allowance** — palindrome-with-one-deletion; converge from both ends, and on the first mismatch branch into two sub-checks.
3. **Length-prefix framing** — for serialization (encode/decode); the length, not a delimiter alone, is what makes decoding unambiguous when the payload can contain the delimiter. This is a *real protocol design* idea (TCP framing, Redis RESP, gRPC length-prefixed messages).

`String` immutability underpins all of this: you never mutate in place, you slice (`substring`) or build (`StringBuilder`). `substring` in modern JDKs copies (since Java 7u6 it no longer shares the backing array — no more memory-leak surprise).

### Worked example — Longest Palindromic Substring (LC 5), expand-around-center

```java
// Java 17
class Solution {
    public String longestPalindrome(String s) {
        if (s == null || s.length() < 1) return "";
        int start = 0, end = 0;
        for (int i = 0; i < s.length(); i++) {
            int odd  = expand(s, i, i);      // odd-length center (single char)
            int even = expand(s, i, i + 1);  // even-length center (between two chars)
            int len = Math.max(odd, even);
            if (len > end - start) {
                start = i - (len - 1) / 2;   // recover the left boundary from the length
                end   = i + len / 2;
            }
        }
        return s.substring(start, end + 1);
    }

    private int expand(String s, int l, int r) {
        while (l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)) { l--; r++; }
        return r - l - 1;   // window grew one step too far on each side
    }
}
```

**Complexity:** `O(n²)` time (n centers × up to n expansion), `O(1)` extra space. The two `expand` calls per index handle odd- and even-length palindromes uniformly — the classic bug is forgetting the even case (`i, i+1`).

### Practice set

| # | Problem | Approach | Key insight | Complexity | Follow-up |
|---|---------|----------|-------------|------------|-----------|
| LC 5 | Longest Palindromic Substring | Expand around center | Try odd (i,i) AND even (i,i+1) centers | `O(n²)` / `O(1)` | "Can you do O(n)?" → Manacher's algorithm |
| LC 271 | Encode and Decode Strings | Length-prefix framing | `len + "#" + s`; read digits up to `#` then take exactly `len` chars | `O(total)` / `O(1)` | "What if you used only a delimiter?" → breaks when payload contains it |
| LC 680 | Valid Palindrome II | Two pointers, one deletion | On mismatch, try skipping left OR right, check remainder | `O(n)` / `O(1)` | "k deletions?" → generalize / DP |

#### LC 271 — Encode and Decode Strings (length-prefix)

```java
public String encode(List<String> strs) {
    StringBuilder sb = new StringBuilder();
    for (String s : strs) sb.append(s.length()).append('#').append(s);
    return sb.toString();                       // e.g. ["lint","code"] -> "4#lint4#code"
}

public List<String> decode(String s) {
    List<String> res = new ArrayList<>();
    int i = 0;
    while (i < s.length()) {
        int j = i;
        while (s.charAt(j) != '#') j++;         // read the length digits
        int len = Integer.parseInt(s.substring(i, j));
        String word = s.substring(j + 1, j + 1 + len);  // take EXACTLY len chars
        res.add(word);
        i = j + 1 + len;
    }
    return res;
}
```

The length prefix makes the scheme unambiguous even when a string *contains* `#` — you never search for the next delimiter inside the payload; you jump exactly `len` characters. **Tie to Deep Fathom's LLM-proxy framing protocol:** the same length-prefix idea frames variable-length messages over the wire so a `#` (or any byte) inside content can't desync the parser.

#### LC 680 — Valid Palindrome II

```java
public boolean validPalindrome(String s) {
    int l = 0, r = s.length() - 1;
    while (l < r) {
        if (s.charAt(l) != s.charAt(r))
            return isPalin(s, l + 1, r) || isPalin(s, l, r - 1);  // skip left OR right
        l++; r--;
    }
    return true;
}
private boolean isPalin(String s, int l, int r) {
    while (l < r) { if (s.charAt(l++) != s.charAt(r--)) return false; }
    return true;
}
```

---

## Block B — Comparable vs Comparator + TreeMap/TreeSet (~2 h)

Cross-reference [07-collections.md](../../07-collections.md) (TreeMap/TreeSet) and [03-oop-advanced.md](../../03-oop-advanced.md) (functional interfaces/lambdas).

### Comparable — the natural ordering (intrinsic)

- `java.lang.Comparable<T>`, single method `int compareTo(T o)`. Implemented *on the class itself* — there's one natural order.
- Used automatically by `Arrays.sort`, `Collections.sort`, `TreeSet`, `TreeMap` (no comparator supplied), `PriorityQueue`.
- Contract: returns negative / 0 / positive for less / equal / greater; must be a *total order* (transitive, antisymmetric).
- **Should be consistent with `equals`**: `a.compareTo(b) == 0` *iff* `a.equals(b)`. When it isn't, sorted collections behave "surprisingly" (see BigDecimal below).

```java
record Money(long cents) implements Comparable<Money> {
    public int compareTo(Money o) { return Long.compare(this.cents, o.cents); } // never use subtraction (overflow!)
}
```

### Comparator — external / ad-hoc ordering

- `java.util.Comparator<T>`, `int compare(T a, T b)`. Defined *outside* the class — you can have **many** orderings, choose one at call time, and order classes you don't own.
- Java 8 fluent builders: `Comparator.comparing(...)`, `.thenComparing(...)`, `.reversed()`, `.nullsFirst(...)`.

```java
List<User> users = ...;
users.sort(
    Comparator.comparing(User::lastName)
              .thenComparing(User::firstName)
              .thenComparingInt(User::age)
              .reversed()                 // careful: reversed() flips the WHOLE chain
);
// Natural order with nulls last:
list.sort(Comparator.nullsLast(Comparator.naturalOrder()));
```

**Gotcha:** `.reversed()` reverses the *entire* composed comparator, not just the last key. To reverse one field, reverse that comparator before chaining: `Comparator.comparing(User::age, Comparator.reverseOrder()).thenComparing(User::name)`.

**Never use subtraction** (`a.getX() - b.getX()`) as a comparator — it overflows for large/negative ints and silently produces a wrong order. Use `Integer.compare` / `Long.compare`.

### TreeMap / TreeSet gotchas

- Red-black tree → keys kept sorted; operations `O(log n)`; `firstKey`, `ceilingKey`, `floorKey`, `subMap`, `headMap`, `tailMap` are the payoff.
- **No comparator + key not `Comparable` → `ClassCastException` at runtime** (not compile time — the cast happens on the first cross-element comparison).
- **The `BigDecimal` consistency bite:** `new BigDecimal("1.0").compareTo(new BigDecimal("1.00")) == 0` (scale-insensitive) but `.equals()` returns `false` (scale-sensitive). Consequence:
  - `TreeSet` uses `compareTo` → treats `1.0` and `1.00` as **equal** → keeps **one**.
  - `HashSet` uses `equals` + `hashCode` → treats them as **distinct** → keeps **both**.
  This is the canonical "ordering inconsistent with equals" failure — and a real production landmine when money values arrive at different scales.

```java
var ts = new TreeSet<BigDecimal>();
ts.add(new BigDecimal("1.0")); ts.add(new BigDecimal("1.00"));
System.out.println(ts.size());   // 1  (compareTo == 0 => duplicate)

var hs = new HashSet<BigDecimal>();
hs.add(new BigDecimal("1.0")); hs.add(new BigDecimal("1.00"));
System.out.println(hs.size());   // 2  (equals false)
```

**Smart360/WebX tie-in:** chained comparators sort report lists by (org, then created-date desc, then name) in one expression; WebX provider selection sorts candidates by (health, then latency, then cost) — a runtime-chosen `Comparator` is exactly the right tool because the order changes by request context.

---

## Block C — Mixed-Pattern Recognition / Weak Spots (~1.5 h)

**The drill:** rank your confidence on every problem Mon–Thu (1–5). Re-solve your **two lowest** clean, from scratch, no notes, ≤ 15 min each. Then run the recognition exercise below — *name the pattern before writing any code.* Pattern recognition speed is what wins timed rounds.

### Recognition drill

| Problem | First-glance signal | Pattern | Why not the other one |
|---------|--------------------|---------|-----------------------|
| Find All Anagrams (LC 438) | fixed-length substrings, char counts | sliding window + `int[26]` compare | not sorted-string keys — too slow per window |
| Maximum Subarray (LC 53) | contiguous, max sum, any length | Kadane | not fixed-window — length isn't fixed |
| Max sum of length *exactly* k | contiguous, **fixed** length | fixed-size sliding window | NOT Kadane — Kadane is variable-length |

#### LC 438 — Find All Anagrams in a String

```java
public List<Integer> findAnagrams(String s, String p) {
    List<Integer> res = new ArrayList<>();
    if (s.length() < p.length()) return res;
    int[] need = new int[26], win = new int[26];
    for (char c : p.toCharArray()) need[c - 'a']++;
    for (int i = 0; i < s.length(); i++) {
        win[s.charAt(i) - 'a']++;                       // expand right
        if (i >= p.length()) win[s.charAt(i - p.length()) - 'a']--;  // shrink left (fixed size)
        if (i >= p.length() - 1 && Arrays.equals(win, need)) res.add(i - p.length() + 1);
    }
    return res;
}   // O(n * 26) = O(n) time, O(1) space
```

#### LC 53 — Maximum Subarray (Kadane)

```java
public int maxSubArray(int[] nums) {
    int cur = nums[0], best = nums[0];
    for (int i = 1; i < nums.length; i++) {
        cur = Math.max(nums[i], cur + nums[i]);   // restart vs extend
        best = Math.max(best, cur);
    }
    return best;
}   // O(n) / O(1)
```
**Invariant:** `cur` = max subarray sum *ending at i*. Either extend the previous run or restart at `nums[i]`. The `Math.max(nums[i], …)` restart is what correctly handles the **all-negative** array (returns the largest single element, not 0).

---

## 💻 Practice coding questions

1. **Longest Palindromic Substring (LC 5)** — expand around center (odd + even). *Follow-up:* mention Manacher's `O(n)`.
2. **Palindromic Substrings (LC 647)** — same expand-around-center, but *count* every palindrome instead of tracking the longest.
3. **Encode and Decode Strings (LC 271)** — length-prefix framing.
4. **Valid Palindrome II (LC 680)** — two pointers, one deletion.
5. **Valid Palindrome (LC 125)** — recap: skip non-alphanumerics, case-fold, converge.
6. **Find All Anagrams in a String (LC 438)** — fixed-window `int[26]` compare.
7. **Group Anagrams (LC 49)** — recap: canonical key (`int[26]` → `Arrays.toString`).
8. **Maximum Subarray / Kadane (LC 53)** — restart-vs-extend invariant.
9. **Sort objects by 3 keys** — `Comparator.comparing().thenComparing().thenComparing()`; print the chain.
10. **BigDecimal in TreeSet vs HashSet** — predict and explain the sizes (1 vs 2).
11. **Custom Comparator for Top K Frequent Words (LC 692)** — count desc, then lexicographic asc.
12. **Reverse Words in a String (LC 151)** — trim, split on `\\s+`, reverse, join; or in-place reverse-all then reverse-each-word.
13. **First Unique Character (LC 387)** — `int[26]` frequency, second pass for the first count==1. Tie-in to frequency maps.
14. **Predict-the-output:** `"a" + "b" + "c"` vs concatenation in a 100k loop — explain the `O(n²)` trap and `StringBuilder` fix.

---

## 🎤 Interview questions

1. **Comparable vs Comparator — one line each.** Comparable = the class's single natural order (`compareTo`, in `java.lang`); Comparator = external, many possible orders (`compare`, in `java.util`).
2. **When do you reach for Comparator?** Multiple/ runtime-chosen orderings, sorting a class you don't own, or a quick lambda sort.
3. **Chain three comparators with a reversed tie-break.** `Comparator.comparing(A).thenComparing(B).thenComparing(C, reverseOrder())`.
4. **What does `.reversed()` reverse?** The *entire* composed comparator — to reverse one key, reverse that key's comparator before chaining.
5. **Why never use subtraction in a comparator?** Integer overflow for large/negative values silently breaks the order; use `Integer.compare`.
6. **TreeSet with a non-Comparable key and no comparator?** `ClassCastException` at runtime on the first comparison.
7. **BigDecimal in TreeSet vs HashSet — sizes?** TreeSet 1 (`compareTo` scale-insensitive → duplicate), HashSet 2 (`equals` scale-sensitive → distinct). The "ordering inconsistent with equals" trap.
8. **Why must `compareTo` be consistent with `equals`?** Sorted collections (TreeSet/TreeMap) use `compareTo` for equality; inconsistency makes them "lose" or merge elements unexpectedly.
9. **Why is `String` immutable, and what does it buy?** Thread-safety, safe hashing/caching, string-pool interning, secure use as map keys / in security contexts (a `String` password can't be mutated mid-use).
10. **Cost of `s = s + x` in a loop?** Each `+` allocates a new backing array → `O(n²)`. Use `StringBuilder` for `O(n)`.
11. **`==` vs `.equals()` for strings, and the pool?** `==` compares references; literals are interned (pooled) so `"a"=="a"` is true, but `new String("a") == "a"` is false. Always `.equals()`.
12. **Does `substring` share the backing char array?** Not since Java 7u6 — it copies, so a small substring of a huge string no longer pins the whole array.
13. **Expand-around-center vs DP for longest palindrome?** Both `O(n²)`, but expand-around-center is `O(1)` space vs DP's `O(n²)`; Manacher's is `O(n)` if pushed.
14. **Why length-prefix encoding over a pure delimiter?** A delimiter can appear inside the payload and desync the parser; a length tells you exactly how many bytes to consume.
15. **Kadane invariant + the all-negative case.** `cur = max(num, cur+num)` = best sum ending here; the `max(num, …)` restart returns the largest single element when all are negative.
16. **Find All Anagrams — why `int[26]` not sorting each window?** `Arrays.equals` on fixed-size count arrays is `O(26)=O(1)` per window vs `O(k log k)` to sort each.
17. **Difference between fixed-size and variable-size sliding window?** Fixed = window length constant (slide one in, one out); variable = expand/shrink based on an invariant (Kadane-like or "min window").
18. **`Comparator.naturalOrder()` vs implementing `Comparable`?** `naturalOrder()` delegates to `compareTo` — useful with `nullsFirst/nullsLast` wrappers when you still want the class's natural order.
19. **How does `TreeMap` find a ceiling key in O(log n)?** Red-black tree traversal — go right when key > node, else record node and go left; `ceilingKey`/`floorKey` exploit BST ordering.
20. **Where does a custom Comparator show up in your projects?** Sorting report lists by (org, date desc, name) in Smart360; ranking WebX providers by (health, latency, cost) per request.

---

## ✅ Self-check

1. Chain three Comparators with `.thenComparing()` in 30 seconds, and reverse just the middle key.
2. What happens to a `TreeSet` if you insert objects where `compareTo` returns 0 but `equals` returns false? *(Treated as duplicates — only one stored.)*
3. Code Longest Palindromic Substring (expand-around-center) cold; handle both odd and even centers.
4. Explain length-prefix encoding and why a pure delimiter is unsafe.
5. State Kadane's invariant and the all-negative handling in one breath.
6. Re-solve your two weakest Mon–Thu problems, no notes, ≤ 15 min each.

---

*Nav: ← [Thu Jul 02](04-thu-jul-02.md) · [Week 1](README.md) · [Sat Jul 04](06-sat-jul-04.md) →*
