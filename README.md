# Study Notes — Java + Interview Prep

Master index for a production/interview-focused Java study set and a full-time interview-prep sprint for a full-stack Java engineer (Java 17 / Spring Boot 3 / microservices / AWS + Azure / K8s) targeting GCC and product-company roles in India.

**Three tracks:**
1. **Study Modules** (`01`–`08`) — Java fundamentals → advanced → modern, why-focused with production tie-ins.
2. **Interview references** — a personal Q&A bank and the market-analysis + strategy doc.
3. **Full-Time Sprint Plan** (`prep-weeks/`) — day-by-day checklists, DSA + system design + behavioral, with weekly self-assessment gates.

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

## 🗓️ Full-Time Sprint — study by Aug 1, apply in August, land by September

Detailed day-by-day checklists live in **[`prep-weeks/`](prep-weeks/README.md)**. Log daily completion in the **[Progress Tracker](PROGRESS.md)**. A 1-day Java-fundamentals refresh (Week 0, Sun Jun 28) precedes the **full-time intensive**: 5 study weeks (Mon–Sat, ~36 hr/wk) finishing **Sat Aug 1**, then 7 execution weeks of applications + interviews. Candidate is **available to join immediately** (no notice). Plan runs **Sun Jun 28 → Sun Sep 20 2026**. Track applications + referrals — the lever that decides offers — in **[Applications & Referrals](prep-weeks/applications-and-referrals.md)** (includes the **Floor / Safety-Net ladder** for "employed no matter what"). Each study week is a folder of deep per-day files.

| Week | Dates | Phase | Focus |
|------|-------|-------|-------|
| [00](prep-weeks/week-00-basics/) | Jun 28 *(1-day)* | Refresh | Fast Java-fundamentals pass — modules 01–04 · **130-Q [interview bank](prep-weeks/week-00-basics/interview-questions.md) + 76 [coding problems](prep-weeks/week-00-basics/coding-questions.md)** |
| [01](prep-weeks/week-01/) | Jun 29–Jul 4 | Study | Arrays / Two Pointers / Sliding Window · hashing/strings/bit · Java internals (equals/hashCode, HashMap, generics, Collections) |
| [02](prep-weeks/week-02/) | Jul 6–11 | Study | Binary search · recursion/backtracking · stacks/queues/LL/LRU · Spring core · concurrency · REST/OAuth2/JWT · exceptions |
| [03](prep-weeks/week-03/) | Jul 13–18 | Study | Trees · Graphs+Trie+Dijkstra · caching/indexing/SQL-NoSQL · Kafka/Outbox/Saga · frontend · LLD primer |
| [04](prep-weeks/week-04/) | Jul 20–25 | Study | DP · Greedy · Intervals · Heaps · design-DS · rate-limit/gateway/CB/bulkhead · sys-design fundamentals · cloud stories |
| [05](prep-weeks/week-05/) | Jul 27–Aug 1 | Study | sys-design mocks · full mocks · **10 STAR stories** · finalize resume/profiles/GitHub · **build target list + referral connections** |
| [06](prep-weeks/week-06/) | Aug 3–9 | Apply | **Apply wave 1** (target + safety net) · referral asks · DSA maintenance · OA drills |
| [07](prep-weeks/week-07/) | Aug 10–16 | Apply | Apply wave 2 · first rounds · system design under pressure · per-company tailoring |
| [08](prep-weeks/week-08/) | Aug 17–23 | Interview | **Interview loops cluster** · project deep-dives · behavioral execution |
| [09](prep-weeks/week-09/) | Aug 24–30 | Interview | Loops · coding-mechanics execution · pipeline mgmt · early offers |
| [10](prep-weeks/week-10/) | Aug 31–Sep 6 | Close | **Offers + negotiation** · CTC deflection · competing-offer leverage |
| [11](prep-weeks/week-11/) | Sep 7–13 | Close | Convert + accept · counter-offers · join-date negotiation |
| [12](prep-weeks/week-12/) | Sep 14–20 | Close | Close / buffer · BGV prep · fallback if pipeline thin |

> **After Week 12:** offers realistically cluster late-Sept → late-Oct (loops run ~3–6 weeks). With **immediate availability** you can accept and start fast — use it as leverage. Target comp ₹14–25 LPA (40–70% jump).

---

## How to use

- **Studying a topic?** Start with the relevant module (`01`–`08`); the numbering is the dependency order.
- **Following the plan?** Each Monday open that week's file, skim the goal + self-assessment gate, tick the daily checklist, and run the Sunday self-assessment before advancing.
- **Searching across everything:** `grep -rn "your-term" .`

*Repository guidance for AI tooling lives in [CLAUDE.md](CLAUDE.md).*
