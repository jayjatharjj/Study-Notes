# Week 4 — Core Build (Jul 20–25, 2026)

> Tame **dynamic programming and greedy**, master the **resilience + at-scale** layer interviewers reach for next — rate limiting, circuit breakers / bulkheads, API gateways, heaps, and the distributed-systems backbone (sharding, replication, CAP / PACELC, consistent hashing) — and tie every answer back to a real decision from Smart360, Deep Fathom, or WebX.
>
> **Full-time, heads-down study week — Mon–Sat (6 study days), Sun rest. No applications this week** — pure skill build. Three blocks per day: **Block A** (DSA, ~2.5 hr), **Block B** (core/system knowledge, ~2 hr), **Block C** (a second DSA set or deep-dive, ~1.5 hr).

---

## 🎯 Week Goal

Derive any classic 1D/2D DP recurrence (Climbing Stairs, House Robber I/II, Coin Change I/II, LIS, LCS, 0/1 knapsack, Word Break, Edit Distance, Maximal Square) from **state + transition** — no memorising. Solve greedy and interval problems on sight. Implement all four heap patterns and the design-DS primitives cold. Explain rate-limiting algorithms, the Resilience4j circuit-breaker state machine and bulkhead, gRPC vs REST vs messaging, and the at-scale layer (sharding, leader-follower replication, CAP, PACELC, consistent hashing) at depth — and field every curveball from first principles.

---

## ✅ By Saturday you can...

- Derive and code Climbing Stairs, House Robber I/II, Coin Change, LIS, and Word Break from scratch, space-optimising where `dp[i]` depends only on a couple of prior states.
- Solve 2D DP — Unique Paths, LCS, Edit Distance, Maximal Square, Longest Palindromic Substring — and the 0/1 knapsack family (Partition Equal Subset Sum, Coin Change II), explaining why the 0/1 inner loop runs right-to-left and unbounded runs left-to-right.
- Solve the greedy + interval set (Jump Game I/II, Gas Station, Merge/Insert/Non-overlapping Intervals, Meeting Rooms II) in ≤15 min each, and state which sort order each needs and why.
- Implement the four heap patterns (top-K min-heap, two-heap streaming median, merge-K, custom comparator) and design-DS (Insert/Delete/GetRandom O(1), LFU cache) from a blank editor.
- Explain token bucket vs leaky bucket vs sliding window (log + counter): complexity, burst handling, and the Redis structure each uses.
- Draw the Resilience4j circuit-breaker state machine with exact config edges, distinguish bulkhead (semaphore vs thread-pool) from circuit breaker, and name the wrap order with Retry + TimeLimiter.
- Compare REST, gRPC, and async messaging across coupling, latency, schema evolution, observability, and failure mode, and contrast API Gateway vs BFF.
- Explain range/hash/consistent-hashing sharding and their failure modes, leader-follower replication lag with three mitigations, CAP precisely (behaviour *during* a partition), PACELC, and the four consistency models — placing PostgreSQL, Redis, Cassandra, and DynamoDB correctly.
- Whiteboard Docker multi-stage builds, K8s rolling-update + probe mechanics, and the GitLab DAG / BuildKit pipeline that drove the 57% CI/CD cut.

---

## 📅 Daily map

| Day | File | Focus | Headline coverage |
|---|---|---|---|
| **Mon Jul 20** | [01-mon-jul-20.md](01-mon-jul-20.md) | 1D DP · Rate-Limiting Algorithms · DP practice | LC 70/198/213/139/300/322; token/leaky/sliding-window + Redis structures |
| **Tue Jul 21** | [02-tue-jul-21.md](02-tue-jul-21.md) | 2D DP · Resilience4j · Knapsack/Subset DP | LC 62/72/1143/221/5; circuit-breaker state machine + bulkhead + wrap order; LC 416/518 |
| **Wed Jul 22** | [03-wed-jul-22.md](03-wed-jul-22.md) | Greedy + Intervals · API Gateway / gRPC · Interval drills | LC 55/45/134/56/57/435/253; REST vs gRPC vs messaging, Gateway vs BFF |
| **Thu Jul 23** | [04-thu-jul-23.md](04-thu-jul-23.md) | Heaps · Sharding & Replication · Design-DS | LC 215/347/295/23/373; range/hash/consistent-hashing + leader-follower lag; LC 380/460 |
| **Fri Jul 24** | [05-fri-jul-24.md](05-fri-jul-24.md) | Heaps / Design Twitter · CAP / PACELC · Cloud/DevOps | LC 355/621/692; CAP/PACELC + consistency models + consistent hashing; Docker/K8s/CI-CD/Bicep |
| **Sat Jul 25** | [06-sat-jul-25.md](06-sat-jul-25.md) | Timed DP/Heap Set · System-Design-at-Scale Consolidation | LC 70/322/300/1143/416 timed (+215/295); backbone reproduced from memory; weak-spot drills |
| **Sun Jul 26** | *(rest)* | No study blocks | Let DP, greedy, heaps, and the at-scale patterns consolidate |

---

## 📊 End-of-Week Self-Assessment

Rate yourself 1–5. **Anything below 3 is a mandatory revisit before Week 5.**

| Skill | Target | Your Score | Notes |
|---|---|---|---|
| 1D DP framework (Climbing Stairs, House Robber I/II) | 5 | | |
| Coin Change I/II + Word Break | 4 | | |
| LIS (both O(n²) and O(n log n)) | 4 | | |
| 2D DP: Unique Paths, LCS, Edit Distance | 4 | | |
| Maximal Square + Longest Palindromic Substring | 4 | | |
| 0/1 knapsack 1D right-to-left (LC 416, 518) | 4 | | |
| Greedy: Jump Game I/II + Gas Station | 5 | | |
| Intervals: Merge/Insert/Non-overlapping + Meeting Rooms II | 5 | | |
| Heap top-K + two-heap median (LC 215, 295) | 5 | | |
| Merge-K + custom comparator (LC 23, 373, 692) | 4 | | |
| Design-DS: GetRandom O(1) + LFU (LC 380, 460) | 4 | | |
| Rate-limiting algorithms + Redis structures | 4 | | |
| Resilience4j state machine + bulkhead + wrap order | 4 | | |
| REST vs gRPC vs messaging (five dimensions) | 4 | | |
| API Gateway vs BFF + Spring Cloud Gateway filters | 4 | | |
| Sharding (range/hash/consistent) + failure modes | 5 | | |
| Leader-follower lag + 3 mitigations | 4 | | |
| CAP (precise) + place 4 systems | 5 | | |
| PACELC + place DynamoDB/Spanner | 4 | | |
| Consistency models (strong/RYW/eventual/causal) | 4 | | |
| Docker multi-stage + BuildKit details | 4 | | |
| K8s rolling update + probe mechanics | 4 | | |
| GitLab DAG + 57% CI/CD cut (specific mechanisms) | 4 | | |

**Score interpretation:**
- 4–5 on all DSA items and no system-design item below 3 → ready for Week 5.
- 3 on 1–2 items → revisit in Week 5's daily review slot.
- Below 3 on any DSA item → re-drill it Monday of Week 5 **before** mocks — DP/heap patterns compound.

**Bonus check:** Can you tie the rate-limiting, circuit-breaker, sharding, and CAP choices to specific decisions in Smart360 / Deep Fathom / WebX? If yes on 80%+ of rows, you are an extraordinary candidate, not just a prepared one.

---

*Week 4 of 12 — next: [Week 5](../week-05/) (Mon Jul 27 – Sat Aug 1).*
