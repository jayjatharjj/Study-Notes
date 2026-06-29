# Week 2 — Foundations (Jul 6–11, 2026)

> Master the bedrock every Java interview tests first: binary search and its on-answer variant, recursion/backtracking, the linear structures (stacks, queues, linked lists), and the Spring internals + concurrency model behind your own shipped code.
>
> **Full-time, heads-down study week — Mon–Sat (6 study days), Sun rest. No applications this week — pure skill build.** Three blocks per day: **Block A** (DSA, ~2.5 hr), **Block B** (core/system knowledge, ~2 hr), **Block C** (a second DSA set or deep-dive, ~1.5 hr).

---

## 🎯 Week Goal

Walk into any interview and answer cold questions on Spring IoC/DI, `@Transactional`, bean lifecycle, auto-configuration, the full request lifecycle, and Java concurrency without hesitation. Solve any binary-search, backtracking, stack/monotonic-stack, or linked-list medium cold in ≤25 min. Implement an LRU cache and binary-search-on-answer under time pressure. Explain REST/JWT/OAuth2 production knowledge and articulate the concurrency and transaction decisions you made in WebX and Smart360 at senior-engineer depth.

## ✅ By Saturday you can...

- Implement iterative and recursive binary search and explain the off-by-one logic (`left <= right` vs `left < right`)
- Solve Search in Rotated Sorted Array, Koko Eating Bananas, and Search a 2D Matrix correctly on first attempt
- Identify binary-search-on-answer and apply it (Koko, Ship Packages, Split Array Largest Sum)
- Generate Subsets, Permutations, and Combination Sum via backtracking with correct state-restore and de-duplication
- Solve Valid Parentheses, Min Stack, Daily Temperatures, Next Greater Element (I/II), Online Stock Span, and Largest Rectangle in Histogram with the monotonic-stack model
- Solve Reverse/Merge Linked List, Floyd's cycle detection, Reorder List, Remove Nth from End; implement an LRU cache (HashMap + DLL) cold
- Whiteboard the full Spring request lifecycle and the 8-step bean lifecycle (incl. where AOP proxies appear)
- Explain `@Transactional` proxy mechanics, propagations, and the self-invocation pitfall with a fix
- Explain auto-configuration: `AutoConfiguration.imports`, `@ConditionalOnClass`, `@ConditionalOnMissingBean`, and how to debug what fired
- Explain REST versioning, pagination (offset vs cursor), and idempotency keys; write a `@ControllerAdvice` from memory
- Describe JWT issuance → validation → refresh → RBAC and the OAuth2 grant types end-to-end
- Compare `synchronized` vs `ReentrantLock`, explain `volatile`, write and prevent a deadlock, connect `CompletableFuture`/`ThreadLocal` to WebX, and sketch a parking-lot LLD with a SOLID letter per decision

---

## 📅 Daily map

| Day | File | Focus |
|---|---|---|
| Mon Jul 06 | [01-mon-jul-06.md](01-mon-jul-06.md) | Binary search (standard · rotated · 2D) + Spring bean lifecycle / IoC-DI + binary-search-on-answer |
| Tue Jul 07 | [02-tue-jul-07.md](02-tue-jul-07.md) | Recursion / backtracking (choose-recurse-unchoose) + `@Transactional` internals + backtracking variations |
| Wed Jul 08 | [03-wed-jul-08.md](03-wed-jul-08.md) | Stacks + monotonic stack + Spring Boot auto-config / full request lifecycle + queues |
| Thu Jul 09 | [04-thu-jul-09.md](04-thu-jul-09.md) | Linked lists (reversal · Floyd's · two-pointer) + REST versioning / pagination / idempotency + LRU cache |
| Fri Jul 10 | [05-fri-jul-10.md](05-fri-jul-10.md) | Mixed DSA review (timed) + JWT / OAuth2 + global exception handling (`@ControllerAdvice`) |
| Sat Jul 11 | [06-sat-jul-11.md](06-sat-jul-11.md) | Timed DSA set + concurrency deep-dive (locks/volatile/CompletableFuture/ThreadLocal/deadlock) + LLD primer (parking lot + SOLID) |

*Sun Jul 12 — rest. No new material; optionally skim the self-check answers you missed.*

---

## 📊 End-of-Week Self-Assessment

Rate yourself 1–5. Anything below 3 is a mandatory revisit before Week 3.

| Skill | Target | Your Score | Notes |
|---|---|---|---|
| Binary search — standard, from scratch | 5 | | |
| Search in Rotated Sorted Array (LC 33), first attempt | 4 | | |
| Koko / binary-search-on-answer — identify independently | 4 | | |
| Subsets / Permutations / Combination Sum — state-restore + dedup | 4 | | |
| Valid Parentheses / Min Stack | 4 | | |
| Daily Temperatures / Next Greater (monotonic stack) | 4 | | |
| Largest Rectangle in Histogram | 4 | | |
| Reverse / Merge / Cycle (Floyd's) linked lists | 4 | | |
| Reorder List / Remove Nth from End | 4 | | |
| LRU Cache (full HashMap + DLL) | 4 | | |
| Spring bean lifecycle — all 8 steps without notes | 5 | | |
| @Transactional: proxy type, self-invocation, propagations | 5 | | |
| ApplicationContext vs BeanFactory — 3+ real differences | 4 | | |
| Auto-configuration: conditionals + how to debug | 4 | | |
| Full request lifecycle whiteboard (DispatcherServlet → repo) | 4 | | |
| REST versioning + pagination + idempotency keys | 4 | | |
| JWT issuance → validation → refresh → RBAC | 4 | | |
| OAuth2 grant types (when to use which) | 4 | | |
| @ControllerAdvice + Bean Validation from memory | 4 | | |
| synchronized vs ReentrantLock — when to use each | 4 | | |
| volatile — guarantees, limits, correct use | 4 | | |
| CompletableFuture — chaining, executor choice, WebX link | 4 | | |
| ThreadLocal — Tomcat pool-reuse gotcha, Security usage | 4 | | |
| Deadlock — scenario, 4 Coffman conditions, prevention | 4 | | |
| LLD: Parking Lot classes + a SOLID letter per decision | 4 | | |

**Score interpretation:**
- 4–5 on all: ready for any Spring/concurrency/foundation-DSA question.
- 3 on 1–2 items: revisit in Week 3's daily review slot.
- Below 3 on any item: schedule a focused 45-min re-drill before the topic compounds.

**Bonus check:** Can you connect every concept above to at least one specific line of code or decision in Smart360, WebX, or your API Gateway? 80%+ → you are an extraordinary candidate, not just a prepared one.

---

*Week 2 of 12 — next: [Week 3](../week-03/) (Mon Jul 13 – Sat Jul 18).*

---

### 📎 Companion (offline)
- [Interview answers](interview-answers.md) — written answers to this week's 🎤 prompts.
- [DSA solutions bank](../dsa-solutions/) — full statements + complete Java solutions for the practice sets.
- [Cheat-sheets](../../cheat-sheets/) · [Projects reference](../../projects-reference.md) · [Interview Q&A bank](../../interview-qa.md)
