# Week 0 — Java Fundamentals Refresh (Sun Jun 28, 2026 · 1-day warm-up)

> One fast-refresh day before full-time prep begins. You wrote modules 01–04; the goal today is not to learn them fresh but to skim each module's Quick Reference, re-fix the five gotchas every interview opens with, and run the highest-value interview Q&A aloud. By tonight the fundamentals are warm so **Mon Jun 29** — the first full-time day — builds on solid ground, not shaky recall.

---

## 🎯 Warm-up Goal

By tonight you can explain, from memory, in a sentence or two each: `==` vs `.equals()`, why `String` is immutable, overloading vs overriding, checked vs unchecked exceptions, and heap vs stack — and you've skimmed all four foundation modules so nothing in Week 1's Java-internals blocks feels cold.

---

## 📅 Daily Checklist

---

### Sunday Jun 28 — Rapid Refresh: Modules 01–04 (~6 hr)

📌 **Study today:** [01](../01-java-language-basics.md) language basics (JVM, primitives, arrays, String/SCP, I/O) · [02](../02-oop-fundamentals.md) OOP fundamentals (classes, pass-by-value, static, heap vs stack) · [03](../03-oop-advanced.md) advanced OOP (inheritance, polymorphism, `final`, wrappers/autoboxing, interfaces) · [04](../04-exception-handling-and-memory.md) exceptions & memory (hierarchy, try-with-resources, GC, final/finally/finalize)

This is a **skim-and-recall pass, not a deep read.** For each of the four modules: read the **Quick Reference blockquote**, scan the section headers, pause on the gotcha below, then close the file and run that module's highest-value interview Q&A aloud (~6–8 questions per module — say the answer first, then check). Budget ~75–90 min per module.

**Block A — Module [01](../01-java-language-basics.md): Language Basics (~75 min)**

- Read the Quick Reference; skim JDK/JRE/JVM, the JVM memory areas, primitives & casting (widening vs narrowing), control flow, the `Arrays` utility.
- **Gotcha to re-fix — String immutability + `==` vs `.equals()`:** every "modification" (`+`, `replace`, `substring`) returns a *new* `String`; the original is untouched. Immutability is why the string constant pool (interning) is safe, why `String` is the ideal `HashMap` key, and why it's shareable across threads without synchronization. `==` compares *references*; `.equals()` compares *content*. `new String("x") == "x"` is `false`; `.equals` is `true`. Build strings in loops with `StringBuilder`, never `+` (O(n) throwaway objects).
- **Fast Q&A:** why is String immutable · `==` vs `.equals()` · StringBuilder vs StringBuffer · Scanner vs BufferedReader · what is the string constant pool · widening vs narrowing casts.

**Block B — Module [02](../02-oop-fundamentals.md): OOP Fundamentals (~75 min)**

- Read the Quick Reference; skim object-creation internals, variable types (instance/local/static), constructors & chaining, the `static` keyword.
- **Gotcha to re-fix — heap vs stack:** the **heap** holds objects, is shared across threads, and is GC-managed. The **stack** holds per-thread method frames, local variables, and references — automatically freed when a method returns. This is the memory mental model behind Week 1's Java-internals deep dive.
- Also re-fix **pass-by-value:** Java is always pass-by-value — for objects, the *reference value* is copied (you can mutate the object, not reseat the caller's variable).
- **Fast Q&A:** is Java pass-by-value or by-reference · static vs instance members · what happens behind the scenes on object creation · can a constructor be private · `this` keyword uses · encapsulation benefits.

**Block C — Module [03](../03-oop-advanced.md): Advanced OOP (~90 min)**

- Read the Quick Reference; skim inheritance & constructor execution order, `this`/`super`, `final`, upcasting vs downcasting, interfaces (default methods, marker/functional/nested), inner classes, composition over inheritance.
- **Gotcha to re-fix — overloading vs overriding:** **overloading** is compile-time, same class, *same name + different parameters* (static dispatch). **Overriding** is runtime, subclass, *same signature* (dynamic dispatch via the actual object type). This one comes up in nearly every interview — nail it.
- Quick autoboxing reminder (it returns Wed of Week 1): `Integer` is cached −128…127, so `Integer a=127,b=127; a==b` is `true` but at 128 it's `false`. Always `.equals()` for wrapper comparison.
- **Fast Q&A:** overloading vs overriding · abstract class vs interface · why composition over inheritance · upcasting vs downcasting safety · what are wrapper classes / autoboxing · method hiding vs overriding.

**Block D — Module [04](../04-exception-handling-and-memory.md): Exceptions & Memory (~75 min)**

- Read the Quick Reference; skim the `Throwable` hierarchy, `try-catch-finally`, `throw` vs `throws`, try-with-resources/`AutoCloseable`, custom exceptions + `@ControllerAdvice`, GC generations & algorithms.
- **Gotcha to re-fix — checked vs unchecked:** **checked** (compile-time enforced, recoverable — `IOException`, `SQLException`) must be declared or caught. **Unchecked** (`RuntimeException` subclasses — `NullPointerException`, `IllegalArgumentException`) signal programming errors and aren't enforced by the compiler. **Errors** (`OutOfMemoryError`) are unrecoverable JVM-level problems.
- **Fast Q&A:** checked vs unchecked vs error · `throw` vs `throws` · `final` vs `finally` vs `finalize()` · try-with-resources & `AutoCloseable` · why write custom exceptions · how garbage collection works (generational hypothesis, young/old gen).

---

## ✅ End-of-Warm-up Check (not a scored gate — just a sanity pass)

Can you explain, from memory, in a sentence or two each:

- [ ] `==` vs `.equals()` (reference identity vs logical/content equality)
- [ ] Why `String` is immutable (security, caching/SCP, thread-safety, hashcode caching)
- [ ] Overloading vs overriding (compile-time, same class, different params — vs runtime, subclass, same signature)
- [ ] Checked vs unchecked exceptions (compile-time enforced & recoverable — vs runtime & programming errors)
- [ ] Heap vs stack (shared object storage, GC-managed — vs per-thread method frames & locals)

If any answer is shaky, re-skim that subsection now. No score, no gate — just don't carry a known gap into Monday.

---

## 🌉 Bridge to Week 1

**Mon Jun 29** opens full-time Week 1 with **Arrays & Two Pointers** plus the **`equals`/`hashCode` contract** — and you've just refreshed exactly what both rest on:

- **Arrays** (Module 01) — internals, iteration, the `Arrays` utility — the substrate for every two-pointer/sliding-window problem.
- **The `Object` class methods** (Modules 02 & 03) — `equals`, `hashCode`, `toString` — the contract Week 1 derives from first principles and breaks live in a `HashSet`.
- **String immutability and `==` vs `.equals()`** (Module 01) — why `String` is the ideal HashMap key and why reference equality bites.
- **Heap vs stack** (Module 02) — the memory mental model behind Week 1's Java-internals (JVM memory model, GC, `volatile`) block.

You're not starting cold on Monday — you're continuing.

---

*Refresh done — next: Week 1 (Mon Jun 29 – Sat Jul 4), full-time.*
