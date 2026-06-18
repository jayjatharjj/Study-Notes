# Study Notes — Java + Interview Prep

Master index for a production/interview-focused Java study set and a 13-week interview-prep plan for a full-stack Java engineer (Java 17 / Spring Boot 3 / microservices / AWS + Azure / K8s) targeting GCC and product-company roles in India.

**Three tracks:**
1. **Study Modules** (`01`–`08`) — Java fundamentals → advanced → modern, why-focused with production tie-ins.
2. **Interview references** — a personal Q&A bank and the market-analysis + strategy doc.
3. **13-Week Prep Plan** (`prep-weeks/`) — day-by-day checklists, DSA + system design + behavioral, with weekly self-assessment gates.

---

## 📚 Study Modules

| # | Module | Covers |
|---|--------|--------|
| 01 | [Java Language Basics](01-java-language-basics.md) | JVM, primitives, control flow, Arrays, Strings, I/O |
| 02 | [OOP Fundamentals](02-oop-fundamentals.md) | Classes, constructors, encapsulation, static context, immutability |
| 03 | [Advanced OOP](03-oop-advanced.md) | Inheritance, polymorphism, abstraction, interfaces, inner classes, lambdas, Singleton/Factory |
| 04 | [Exception Handling & Memory](04-exception-handling-and-memory.md) | Exception hierarchy, try-with-resources, GC, `final`/`finally`/`finalize` |
| 05 | [Enums & Annotations](05-enums-and-annotations.md) | Enums with methods/fields, built-in and custom annotations |
| 06 | [Concurrency & Collections](06-concurrency-and-collections.md) | Threads, synchronization, Executor Framework, Future/Callable |
| 07 | [Collections Framework](07-collections.md) | Full Collections API: List/Set/Queue/Map implementations and algorithms |
| 08 | [Modern Java (16–21) & Spring Boot 3](08-modern-java.md) | Records, sealed types, switch/pattern matching, text blocks, virtual threads, Jakarta/Boot 3 |

Each module opens with a **Quick Reference** blockquote and ends with **Interview Q&A**. Code marks anti-patterns `// BAD` and preferred patterns `// GOOD`.

---

## 🎯 Interview References

- **[Interview Q&A bank](interview-qa.md)** — why-focused Q&A across Java, Spring Boot, databases, and architecture.
- **[Career Plan 2026](career-plan-2026.md)** — India job-market analysis, comp-tier strategy, and the 13-week timetable at a glance.

---

## 🗓️ 13-Week Prep Plan

Detailed day-by-day checklists live in **[`prep-weeks/`](prep-weeks/README.md)**. Log daily completion in the **[Progress Tracker](PROGRESS.md)**. A 4-day Java-basics revision (Week 0) precedes Week 1. Plan runs **Thu 18 Jun → Sun 20 Sep 2026** (Week 0 is a 4-day warm-up; Week 1 is a full Mon–Sun week; Weeks 2–13 are clean Mon–Sun).

| Week | Dates | Phase | Focus |
|------|-------|-------|-------|
| [00](prep-weeks/week-00-basics.md) | Jun 18–21 *(4-day)* | Warm-up | Revise Java fundamentals — modules 01–04 (language basics, OOP, advanced OOP, exceptions+memory) |
| [01](prep-weeks/week-01.md) | Jun 22–28 | Foundations | Arrays / Two Pointers / Sliding Window · Java internals (equals/hashCode, HashMap) · resume + deep-dive doc |
| [02](prep-weeks/week-02.md) | Jun 29–Jul 5 | Foundations | Hashing / Strings · bit manipulation · Collections deep-dive · system-design primer |
| [03](prep-weeks/week-03.md) | Jul 6–12 | Foundations | Recursion / Binary Search · Spring core (IoC, beans, @Transactional) · concurrency · LLD primer |
| [04](prep-weeks/week-04.md) | Jul 13–19 | Foundations | Stacks / Queues / LinkedList · REST + OAuth2/JWT · estimation drill · **self-assessment checkpoint** |
| [05](prep-weeks/week-05.md) | Jul 20–26 | Core Build | Trees · caching + indexing + SQL/NoSQL · frontend (Vue 3 / React) · target-company list |
| [06](prep-weeks/week-06.md) | Jul 27–Aug 2 | Core Build | Graphs + Trie + Dijkstra · messaging/async, Outbox, Saga · GitHub polish |
| [07](prep-weeks/week-07.md) | Aug 3–9 | Core Build | DP intro · rate limiting, API gateway, circuit breakers + bulkhead · mock system design |
| [08](prep-weeks/week-08.md) | Aug 10–16 | Core Build | DP + Greedy · cloud/DevOps talking points · **profiles live + warm-up applications** |
| [09](prep-weeks/week-09.md) | Aug 17–23 | Peak | Heaps + design-DS · design at scale (sharding/CAP) · **hard applications to GCCs/product** |
| [10](prep-weeks/week-10.md) | Aug 24–30 | Peak | Company-tagged DSA · **10 STAR stories** · News Feed HLD · more applications + first rounds |
| [11](prep-weeks/week-11.md) | Aug 31–Sep 6 | Peak | DSA maintenance · **interviews cluster** · full-stack consolidation + project rehearsal |
| [12](prep-weeks/week-12.md) | Sep 7–13 | Close | Light DSA · **interview-loop execution** · per-company tailoring + pipeline tracking |
| [13](prep-weeks/week-13.md) | Sep 14–20 | Close | **Negotiation & close** · competing-offer leverage · clean resignation + 60-day notice |

> **After Week 13:** offers realistically cluster late-Sept → late-Oct (loops run ~3–6 weeks + 60-day notice). Accept best offer → 60-day notice → join ~Nov. Target comp ₹14–25 LPA (40–70% jump).

---

## How to use

- **Studying a topic?** Start with the relevant module (`01`–`08`); the numbering is the dependency order.
- **Following the plan?** Each Monday open that week's file, skim the goal + self-assessment gate, tick the daily checklist, and run the Sunday self-assessment before advancing.
- **Searching across everything:** `grep -rn "your-term" .`

*Repository guidance for AI tooling lives in [CLAUDE.md](CLAUDE.md).*
