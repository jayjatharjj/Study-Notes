# Week 1 — Foundations (Jun 29–Jul 4, 2026)

> The first full-time foundations week (≈36 hr; **Mon–Sat, Sun rest**). Lock in the two skills every product-company interview tests first: pattern-recognition on arrays / pointers / windows / hashing / bits, and fluent Java internals tied to your own shipped code (equals/hashCode, HashMap, generics, the Collections framework). This week is **heads-down study — it does NOT include applications**; the application engine starts later. Assumes the **Sun Jun 28 Week 0 basics refresh** is done and feeds directly into the Java-internals blocks here.

---

## 🎯 Week Goal

By Saturday you can solve any Easy and most Medium array / two-pointer / sliding-window / hashing / string / bit problems in ≤ 20 min with a clean Big-O analysis, derive the `equals`/`hashCode` contract and HashMap internals from memory, reason about the Collections framework from first principles (not just "what" but "why the data structure was designed that way"), and whiteboard Smart360's 5-microservice topology with sharp articulation of every architectural decision. Your resume rewrite and project deep-dive docs are started.

### ✅ By Saturday you can...

- Solve Trapping Rain Water, 3Sum, Minimum Window Substring, Group Anagrams, Top K Frequent, and Longest Consecutive Sequence from scratch with correct time/space complexity, articulating the exact *why* behind each pointer move or hash key.
- Derive the `equals`/`hashCode` contract from first principles and explain what breaks in a `HashSet` when you violate it — with a live code example.
- Explain HashMap's internal structure (buckets, linked list → red-black tree at threshold 8 with capacity ≥ 64, load factor 0.75, the `(h ^ (h >>> 16))` bit-spread, resize split) without looking anything up.
- Distinguish fail-fast vs fail-safe iterators, compare `ConcurrentHashMap` (CAS + per-bucket `synchronized`) vs `Hashtable` (full-map lock), and choose between `ArrayList`/`LinkedList`/`ArrayDeque` with the CPU-cache argument.
- Explain `Comparable` vs `Comparator`, the `BigDecimal`-vs-`equals` consistency gotcha, generics/type-erasure, and the autoboxing 128-cache boundary — and why each matters in production.
- Recite 3 quantified bullets per project (Smart360, Deep Fathom, WebX) that survive recruiter screening *and* deep technical drill-down, and whiteboard the Smart360 5-service diagram with failure boundaries.

---

## 📅 Daily map

Each weekday runs **3 blocks**: **Block A** — DSA (~2.5 h, primary pattern set), **Block B** — core Java/system topic (~2 h), **Block C** — second DSA set or deep-dive (~1.5 h). Saturday is a consolidation day (lighter DSA, heavier on SOLID / system design / resume).

| Day | File | Focus |
|-----|------|-------|
| Mon Jun 29 | [01-mon-jun-29.md](01-mon-jun-29.md) | Two Pointers (opposite ends) + `equals`/`hashCode` contract + Two-Sum / prefix-sum intro |
| Tue Jun 30 | [02-tue-jun-30.md](02-tue-jun-30.md) | Sliding Window (variable size) + HashMap internals + subarray-sum |
| Wed Jul 01 | [03-wed-jul-01.md](03-wed-jul-01.md) | 3Sum / prefix sum + Generics, type erasure & autoboxing + hashing (anagrams) |
| Thu Jul 02 | [04-thu-jul-02.md](04-thu-jul-02.md) | Hashing / Top-K + Collections deep-dive + bit manipulation |
| Fri Jul 03 | [05-fri-jul-03.md](05-fri-jul-03.md) | Strings + Comparable/Comparator + weak-spot review |
| Sat Jul 04 | [06-sat-jul-04.md](06-sat-jul-04.md) | Timed mixed DSA + SOLID / system-design primer + resume & deep-dive docs |
| Sun Jul 05 | — | **Rest.** No study. Optional: glance Saturday's self-assessment so Week 2 Monday starts on target. |

---

## 📊 End-of-Week Self-Assessment (pass/fail gates)

Complete on Saturday evening. Honest pass/fail only — no partial credit. (Full gate also lives at the end of [Saturday](06-sat-jul-04.md).)

| Gate | Check | Pass criteria |
|------|-------|---------------|
| DSA-1 | Container With Most Water cold | ≤ 15 min, correct O(n) |
| DSA-2 | Minimum Size Subarray Sum cold | ≤ 15 min, correct sliding window |
| DSA-3 | 3Sum cold | ≤ 25 min, handles duplicates |
| DSA-4 | Explain Kadane in 60 sec | States invariant + all-negative case |
| DSA-5 | Top K Frequent with bucket sort (no heap) | ≤ 20 min, O(n) |
| DSA-6 | Longest Consecutive Sequence O(n) HashSet | ≤ 15 min, no sorting |
| DSA-7 | Subarray Sum = K, `{0:1}` seed explained | ≤ 10 min |
| Bits | 4 bit problems (136 / 191 / 338 / 268) | each ≤ 10 min, "why XOR" articulated |
| Java-1 | Derive `equals`/`hashCode` contract | State + code + 1 JPA pitfall |
| Java-2 | Draw HashMap internals from memory | Buckets, list, tree threshold, load factor, resize split |
| Java-3 | Autoboxing 128 cache boundary | Correct + code example |
| Java-4 | ConcurrentHashMap CAS vs synchronized + null reason | Explained without "segment" |
| Java-5 | Fail-fast `modCount` (incl. `set()` edge case) | Code snippet; `set()` doesn't throw |
| Java-6 | Comparable vs Comparator + BigDecimal gotcha | Both + the consistency bite |
| SD | Smart360 5-service diagram from memory | Sync/async arrows + failure modes |
| Resume | All 3 projects have 3+ numbers each | Count them now |

**Score 16/16 = ready to proceed to Week 2.**
**Score 12–15 = identify gaps, spend 30 extra min Monday on the failed gates.**
**Score < 12 = repeat the weakest day before moving on — do not skip to Week 2.**

---

*Week 1 of 12 — next: [Week 2](../week-02/) (Mon Jul 6 – Sat Jul 11).*
