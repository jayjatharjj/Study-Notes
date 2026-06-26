# Week 0 — Java Fundamentals Refresh (Sun Jun 28, 2026 · 1-day warm-up)

> One fast-refresh day before full-time prep begins. You wrote modules 01–04; today isn't about learning them fresh — it's skimming each module's Quick Reference, re-fixing the five gotchas every interview opens with, and drilling the question banks in this folder. By tonight the fundamentals are warm so **Mon Jun 29** (the first full-time day) builds on solid ground.

**This folder:**
- **[interview-questions.md](interview-questions.md)** — 130 Q&A across modules 01–04 (answer aloud first, then check).
- **[coding-questions.md](coding-questions.md)** — 76 Java practice problems (implement / predict-the-output / fix-the-bug) with solutions.

---

## 🎯 Warm-up Goal

By tonight you can explain, from memory: `==` vs `.equals()`, why `String` is immutable, overloading vs overriding, checked vs unchecked exceptions, and heap vs stack — you've skimmed all four foundation modules, and you've run a healthy slice of both question banks so nothing in Week 1's Java-internals blocks feels cold.

---

## 📅 Sunday Jun 28 — Rapid Refresh: Modules 01–04 (~6 hr)

📌 **Study today:** [01](../../01-java-language-basics.md) language basics · [02](../../02-oop-fundamentals.md) OOP fundamentals · [03](../../03-oop-advanced.md) advanced OOP · [04](../../04-exception-handling-and-memory.md) exceptions & memory — then drill the banks.

A **skim-and-recall pass, not a deep read.** For each module: read the **Quick Reference**, scan headers, pause on the gotcha, then close the file and answer that module's questions from [interview-questions.md](interview-questions.md) aloud. Budget ~60–75 min per module + ~90 min on the banks.

**Block A — Module [01](../../01-java-language-basics.md): Language Basics**
- Skim JDK/JRE/JVM, JVM memory areas, primitives & casting (widening vs narrowing), control flow, the `Arrays` utility, String/SCP, I/O.
- **Gotcha — String immutability + `==` vs `.equals()`:** every "modification" returns a *new* `String`; immutability is why the string pool is safe, why `String` is the ideal `HashMap` key, and why it's thread-shareable. `==` compares references; `.equals()` compares content. Build strings in loops with `StringBuilder`.
- Drill: interview-questions §Language Basics/Primitives/Strings; coding §Syntax & Control Flow + §Strings.

**Block B — Module [02](../../02-oop-fundamentals.md): OOP Fundamentals**
- Skim object-creation internals, variable types (instance/local/static), constructors & chaining, `static`.
- **Gotcha — heap vs stack:** heap holds objects (shared, GC-managed); stack holds per-thread frames/locals/references (freed on return). Plus **pass-by-value**: Java copies the reference *value* (you can mutate the object, not reseat the caller's variable).
- Drill: interview-questions §OOP Fundamentals/Encapsulation; coding §OOP.

**Block C — Module [03](../../03-oop-advanced.md): Advanced OOP**
- Skim inheritance & constructor order, `this`/`super`, `final`, up/down-casting, interfaces (default/marker/functional/nested), inner classes, composition over inheritance.
- **Gotcha — overloading vs overriding:** overloading = compile-time, same class, different params (static dispatch); overriding = runtime, subclass, same signature (dynamic dispatch). Plus the autoboxing `Integer` cache (−128…127 → `==` true, 128 → false; always `.equals()`).
- Drill: interview-questions §Inheritance/Abstraction/Wrappers; coding §OOP + §Generics & Wrappers + §Predict-the-Output.

**Block D — Module [04](../../04-exception-handling-and-memory.md): Exceptions & Memory**
- Skim the `Throwable` hierarchy, `try-catch-finally`, `throw` vs `throws`, try-with-resources/`AutoCloseable`, custom exceptions + `@ControllerAdvice`, GC generations & algorithms.
- **Gotcha — checked vs unchecked:** checked (compile-enforced, recoverable) must be declared/caught; unchecked (`RuntimeException`) signal programming errors; `Error` is unrecoverable.
- Drill: interview-questions §Exceptions/Memory & GC; coding §Exceptions + §Fix-the-Bug.

---

## ✅ End-of-Warm-up Check (sanity pass, not a scored gate)

Explain from memory, one or two sentences each:
- [ ] `==` vs `.equals()` · [ ] Why `String` is immutable · [ ] Overloading vs overriding · [ ] Checked vs unchecked exceptions · [ ] Heap vs stack
- [ ] You completed at least one full topic group from each bank.

If any answer is shaky, re-skim that subsection now — don't carry a known gap into Monday.

---

## 🌉 Bridge to Week 1

**Mon Jun 29** opens full-time Week 1 with **Arrays & Two Pointers** + the **`equals`/`hashCode` contract** — exactly what today refreshed: arrays (Module 01), the `Object` methods (02 & 03), String immutability/`==` (01), heap vs stack (02). You're not starting cold Monday — you're continuing.

---

*Refresh done — next: [Week 1](../week-01/) (Mon Jun 29 – Sat Jul 4), full-time.*
