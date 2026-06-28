# Week 3 — Foundations + Core (Jul 13–18, 2026)

> Own the two structures that separate candidates — **trees and graphs** — and the data-layer + distributed-systems knowledge product interviewers test next: index internals, caching architecture, Kafka, the Outbox/Saga patterns, and a crisp full-stack story.
>
> **Full-time, heads-down study week — Mon–Sat (6 study days), Sun rest. No applications this week** — pure skill build. Three blocks per day: **Block A** (DSA, ~2.5 hr), **Block B** (core/system knowledge, ~2 hr), **Block C** (a second DSA set or deep-dive, ~1.5 hr).

---

## 🎯 Week Goal

Solve any tree or graph LeetCode medium **cold in ≤25 min** — traversals, BST ops, LCA, construction, BFS/DFS, topological sort, union-find, Trie, and Dijkstra. Reason about DB index internals, the four cache write strategies, and SQL-vs-NoSQL without hand-waving. Design a production async event-driven system end-to-end — Kafka internals, idempotency/exactly-once, Outbox, both Saga flavors — and a typeahead system, tying every answer back to Smart360's monolith→event-driven migration and WebX's async LLM jobs. Finish with a **recruiter-ready GitHub**.

---

## ✅ By Saturday you can...

- Write iterative pre/in/post-order traversals and level-order BFS (`List<List<Integer>>`) cold
- Solve Validate BST (bounds), Kth Smallest, Diameter, both LCA variants, and Tree Construction from pre+inorder
- Implement BFS, DFS, topological sort (Kahn's + DFS), union-find (path compression + rank), and cycle detection from memory
- Solve the target graph problems (Islands, Clone Graph, Course Schedule I/II, Valid Tree, Connected Components, Rotting Oranges) in ≤25 min each
- Implement a Trie (insert/search/startsWith) and Dijkstra with a min-heap, and state the relaxation invariant
- Explain B-tree structure, composite/covering/partial indexes, and the leftmost-prefix rule
- Name the four cache write strategies, their failure modes, LFU vs LRU eviction, Redis persistence, and Cluster vs Sentinel
- Apply a SQL-vs-NoSQL decision framework (access patterns first) and name when each NoSQL family fits
- Explain Kafka partitions, consumer groups, offsets, ISR, and acks like someone who ran it in prod
- Draw the Outbox + CDC flow, compare choreography vs orchestration Saga, and explain why exactly-once-to-a-DB is a myth
- Give a sharp Vue 3 (Proxy reactivity) vs React (reconciliation/Fiber) comparison
- Design a typeahead/autocomplete system end-to-end and present a recruiter-ready GitHub profile

---

## 📅 Daily map

| Day | File | Focus | Headline coverage |
|---|---|---|---|
| **Mon Jul 13** | [01-mon-jul-13.md](01-mon-jul-13.md) | Tree Traversals + BST · DB Indexing · Tree problems | LC 94/102/104/98/230; B-tree, composite/covering/partial indexes; LC 236/235/543/199 |
| **Tue Jul 14** | [02-tue-jul-14.md](02-tue-jul-14.md) | Tree Construction + Hard Trees · Caching · SQL vs NoSQL | LC 105/124/297; 4 write strategies, Redis, LFU/LRU; NoSQL decision framework |
| **Wed Jul 15** | [03-wed-jul-15.md](03-wed-jul-15.md) | Graphs BFS/DFS · Kafka · Topological Sort | LC 200/695/133/994; partitions/consumer-groups/offsets/ISR/acks; LC 207/210 |
| **Thu Jul 16** | [04-thu-jul-16.md](04-thu-jul-16.md) | Union-Find · Outbox / Saga · Trie | LC 684/547; dual-write, idempotency/exactly-once, both Saga flavors; LC 208/211/1268 |
| **Fri Jul 17** | [05-fri-jul-17.md](05-fri-jul-17.md) | Dijkstra / Weighted Graphs · Frontend (Vue/React) · Typeahead HLD | LC 743/787/1631; Vue 3 Proxy reactivity vs React Fiber; sharded trie + offline pipeline |
| **Sat Jul 18** | [06-sat-jul-18.md](06-sat-jul-18.md) | Timed DSA (graphs/trees) · GitHub Polish · Consolidation | LC 207/994 timed + 2 re-solves; showcase repo + profile; weak-spot drills |
| **Sun Jul 19** | *(rest)* | No study blocks | Let trees, graphs, and the distributed patterns consolidate |

---

## 📊 End-of-Week Self-Assessment

Rate yourself 1–5. **Anything below 3 is a mandatory revisit before Week 4.**

| Skill | Target | Your Score | Notes |
|---|---|---|---|
| Iterative tree traversals (all 3) | 5 | | |
| Level-order BFS (clean `List<List<Integer>>`) | 5 | | |
| Validate BST (bounds approach) | 4 | | |
| LCA — BST (iterative) | 5 | | |
| LCA — general tree (recursive) | 4 | | |
| Tree construction from pre+inorder | 4 | | |
| Diameter / Kth Smallest (+ follow-ups) | 4 | | |
| BFS / DFS from scratch | 5 | | |
| Topological sort (Kahn's + DFS) | 4 | | |
| Union-Find (path compression + rank) | 4 | | |
| Cycle detection (directed vs undirected) | 4 | | |
| Trie (insert/search/startsWith + wildcard) | 4 | | |
| Dijkstra with min-heap + relaxation invariant | 4 | | |
| LC 200 / 207 / 994 — AC within target time | 4 | | |
| B-tree + composite/covering index, leftmost prefix | 4 | | |
| Cache strategies (all 4) + LFU vs LRU | 5 | | |
| Redis eviction/persistence + Cluster vs Sentinel | 4 | | |
| SQL vs NoSQL decision framework | 4 | | |
| Kafka: partitions, consumer groups, offsets, ISR, acks | 4 | | |
| Idempotency patterns + exactly-once honesty | 4 | | |
| Outbox pattern (problem + both delivery options) | 4 | | |
| Saga: choreography vs orchestration | 4 | | |
| Full async LLM job system design without notes | 4 | | |
| Typeahead/autocomplete HLD | 4 | | |
| Vue 3 reactivity vs React reconciliation (spoken) | 4 | | |
| GitHub profile/showcase repo recruiter-ready | 4 | | |

**Score interpretation:**
- 4–5 on all DSA items and no system-design item below 3 → ready for Week 4.
- 3 on 1–2 items → revisit in Week 4's daily review slot.
- Below 3 on any DSA item → re-drill it Monday of Week 4 **before** new material — tree/graph patterns compound.

**Bonus check:** Can you tie Smart360's 30% efficiency gain and WebX's async LLM jobs to specific Kafka/Outbox/caching/indexing choices? If yes on 80%+ of rows, you are an extraordinary candidate, not just a prepared one.

---

*Week 3 of 12 — next: [Week 4](../week-04/) (Mon Jul 20 – Sat Jul 25).*
