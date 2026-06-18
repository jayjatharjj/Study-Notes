# Week 0 — Java Fundamentals Revision (Jun 18–21, 2026 · 4-day warm-up)

> Before the advanced 13-week plan begins, revise Java from the ground up. Four days, one study module per day, at a relaxed revision pace — the goal is not to learn this material fresh but to refresh it so the harder weekly plan (starting **Mon Jun 22**) builds on solid fundamentals instead of shaky ones. You wrote these modules; now read them back, recall actively, and answer each module's interview questions aloud before peeking at the answers. Sat and Sun add a tiny array warm-up to prime your DSA muscles for Week 1.

---

## 🎯 Warm-up Goal

By Sunday night you can explain, from memory, the core Java mechanics every interview opens with — `==` vs `.equals()`, why `String` is immutable, overloading vs overriding, checked vs unchecked exceptions, and heap vs stack — and you've re-solved two easy array problems to shake off the rust before Week 1's Arrays/Two-Pointers block.

---

## 📅 Daily Checklist

---

### Thursday Jun 18 — Module 01: Java Language Basics

📌 **Study today:** JDK/JRE/JVM · JVM memory areas · primitives & casting · operators · control flow · arrays (internals, `Arrays` utility) · String (constant pool, immutability, `==` vs `.equals()`) · StringBuilder vs StringBuffer · I/O (Scanner vs BufferedReader)
⏱ ~2–3 hr (revision pace)

**Read & active-recall** — [module 01](../01-java-language-basics.md):

- [ ] Key features of Java; JDK vs JRE vs JVM; program execution flow (`.java` → `.class` → bytecode → JIT)
- [ ] JVM memory areas (heap, stack, method area/metaspace, PC register, native stack)
- [ ] Primitive data types (8 total) and primitive vs non-primitive
- [ ] Operators (arithmetic, unary, relational, logical, assignment, bitwise, ternary)
- [ ] Type conversion & casting — implicit widening vs explicit narrowing
- [ ] Control statements — decision making, loops, jump statements
- [ ] Arrays — internal working, accessing/iterating, `Arrays` utility class, taking input
- [ ] String — constant pool (SCP), how it works internally, **why String is immutable**, **`==` vs `.equals()`**, hashcode & identityHashCode
- [ ] StringBuffer vs StringBuilder — mutability, thread-safety, when to use which
- [ ] Input/output — standard streams, BufferedReader/InputStreamReader, Scanner, Scanner vs BufferedReader

**Self-test:** Close the file and answer the module's **Interview Q&A (~23 questions)** aloud — *say the answer first, then scroll down to check it.* Flag any you fumble for a quick re-read.

---

### Friday Jun 19 — Module 02: OOP Fundamentals

📌 **Study today:** classes & objects · object creation internals · variable types (instance/local/static) · methods & pass-by-value · immutable objects · instance/static blocks & methods · access modifiers · composition · `this` keyword · constructors (types, chaining, private) · `static` keyword · heap vs stack deep dive
⏱ ~2–3 hr (revision pace)

**Read & active-recall** — [module 02](../02-oop-fundamentals.md):

- [ ] What is OOP and why use it; Class; Object — what happens behind the scenes on object creation; the `Object` class
- [ ] Types of variables — instance vs local vs static (and the comparison table)
- [ ] Methods — return type, calling methods, **pass by value**
- [ ] Immutable objects — building a fully immutable class with defensive copies; the `record` equivalent
- [ ] Instance block, instance methods, static block, static methods
- [ ] Access modifiers (public/protected/default/private)
- [ ] Composition (HAS-A) and OOP design principles
- [ ] JVM structure — **heap vs stack memory deep dive**, full Java execution flow
- [ ] Encapsulation — definition, benefits
- [ ] `this` keyword — resolve name conflict, constructor chaining, pass current object
- [ ] Constructors — types, overloading, constructor vs method, can a constructor be private
- [ ] `static` keyword — variable, method, block, accessing static members, static vs instance

**Self-test:** Close the file and answer the module's **Interview Q&A (~20 questions)** aloud before reading the answers. Pay special attention to pass-by-value and static vs instance — common stumbling points.

---

### Saturday Jun 20 — Module 03: Advanced OOP

📌 **Study today:** inheritance (constructor execution, `this`/`super`) · polymorphism (overloading vs overriding, dynamic dispatch) · `final` · `Object` class methods · upcasting/downcasting · wrapper classes & autoboxing · abstraction (abstract class) · interfaces (default methods, types) · loose coupling · inner classes · encapsulation vs composition · method hiding
⏱ ~2–3 hr (revision pace)

**Read & active-recall** — [module 03](../03-oop-advanced.md):

- [ ] Inheritance — types, what is NOT inherited, constructor execution order, parent ref can't call child-only methods, static members & inheritance
- [ ] `this` and `super` keywords in inheritance
- [ ] Polymorphism — **compile-time (overloading) vs runtime (overriding)**, dynamic method dispatch, why constructor overriding isn't possible
- [ ] `final` keyword (variable/method/class)
- [ ] `Object` class methods; object typecasting — upcasting (safe) vs downcasting (may fail)
- [ ] Wrapper classes; autoboxing and unboxing
- [ ] Abstraction — abstract class
- [ ] Interfaces — default methods (Java 8+), class↔interface relationships, abstract class vs interface
- [ ] Interface types — normal, functional, marker, nested
- [ ] Loose coupling using interfaces
- [ ] Inner classes — non-static, static, method-local, anonymous
- [ ] Encapsulation vs composition; why composition over inheritance; method hiding vs overriding

**Self-test:** Close the file and answer the module's **Interview Q&A (~23 questions)** aloud before checking. Nail **overloading vs overriding** — it comes up almost every interview.

**☀️ Optional warm-up (DSA primer, ~15 min):** ease back into arrays with **Two Sum (LC 1)**. Know both the brute-force O(n²) and the HashMap O(n) one-pass solution. This is the exact warm-up problem Week 1 opens with on Mon — getting it cold today means Monday starts on momentum.

---

### Sunday Jun 21 — Module 04: Exception Handling & Memory

📌 **Study today:** exception hierarchy · checked vs unchecked vs errors · try-catch-finally · throw vs throws · try-with-resources · custom exceptions · `@ControllerAdvice` global handler · garbage collection (generations, algorithms) · final vs finally vs finalize
⏱ ~2–3 hr (revision pace)

**Read & active-recall** — [module 04](../04-exception-handling-and-memory.md):

- [ ] What is an exception; the exception hierarchy (`Throwable` → `Error`/`Exception` → `RuntimeException`)
- [ ] Types — **checked (compile-time) vs unchecked (runtime) vs errors**
- [ ] `try-catch-finally`
- [ ] **`throw` vs `throws`**
- [ ] try-with-resources (Java 7+) and `AutoCloseable`
- [ ] Custom exceptions — why create them, custom checked vs custom unchecked, using them in a Spring Boot service
- [ ] Global exception handler — `@ControllerAdvice`; real business use cases for custom exceptions
- [ ] Garbage collection — what it is, how it works, GC generations, GC algorithms
- [ ] **`final` vs `finally` vs `finalize()`** (and the summary table)

**Self-test:** Close the file and answer the module's **Interview Q&A (~15 questions)** aloud before reading the answers. Make sure checked vs unchecked and final/finally/finalize are crisp.

**☀️ Optional warm-up (DSA primer, ~15 min):** solve **Best Time to Buy & Sell Stock (LC 121)** — track the running minimum price and the best profit so far in a single O(n), O(1) pass. This "track a running minimum" pattern returns in Week 1's Friday block.

---

### 🌉 Bridge to Week 1

**Mon Jun 22** opens Week 1 with **Arrays & Two Pointers** plus the **`equals`/`hashCode` contract**. You've just revised exactly what both rest on:

- **Arrays** (Module 01) — internals, iteration, the `Arrays` utility — the substrate for every two-pointer/sliding-window problem.
- **The `Object` class methods** (Modules 02 & 03) — `equals`, `hashCode`, `toString` — the contract Week 1 derives from first principles and breaks live in a `HashSet`.
- **String immutability and `==` vs `.equals()`** (Module 01) — why `String` is the ideal HashMap key and why reference equality bites.
- **Heap vs stack** (Module 02) — the memory mental model behind Week 1's Sunday Java-internals deep dive.

You're not starting cold on Monday — you're continuing.

---

## ✅ End-of-Warm-up Check (not a scored gate — just a sanity pass)

Can you explain, from memory, in a sentence or two each:

- [ ] `==` vs `.equals()` (reference identity vs logical/content equality)
- [ ] Why `String` is immutable (security, caching/SCP, thread-safety, hashcode caching)
- [ ] Overloading vs overriding (compile-time, same class, different params — vs runtime, subclass, same signature)
- [ ] Checked vs unchecked exceptions (compile-time enforced & recoverable — vs runtime & programming errors)
- [ ] Heap vs stack (shared object storage, GC-managed — vs per-thread method frames & locals)

If any answer is shaky, re-skim that subsection before Monday. No score, no gate — just don't carry a known gap into Week 1.

---

*Warm-up complete — next: Week 1 (Mon Jun 22 – Sun Jun 28), Arrays/Two-Pointers/Sliding-Window + Java internals.*
