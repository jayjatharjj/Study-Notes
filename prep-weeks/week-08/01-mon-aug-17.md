# Week 8 · Day 1 — Mon Aug 17 — Loop Execution + Performance-Win Deep Dive

📌 **Today:** The loops are here. Rehearse the Smart360 deep-dive *before* any round, execute scheduled loop(s) leading with impact numbers, and deliver STAR #1 (performance win) under 2 minutes. DSA drops to one warm-up problem — loops are the priority.

**DSA maintenance (~30 min):** Light — loops take priority.
- [ ] **LC 1 – Two Sum** (or any easy you clear in 5 min). Goal this week is to *stay warm*, not learn.

**Primary activity:**

**Pre-day check (5 min):**
- [ ] Open the tracker. Confirm every loop this week is on the calendar with prep time blocked the evening before. Any loop today/tomorrow? Run the **pre-interview checklist** now (below).

**Project deep-dive rehearsal (40 min) — Smart360 (do before any round):**
- [ ] Rehearse the 5-minute Smart360 talk-track aloud:
  - **Perf:** Hibernate stats → N+1 → JOIN FETCH/EntityGraph + composite indexes + Redis + S3-URL caching → **96% cut**.
  - **Security:** stateless JWT at the gateway → **60% sign-in latency cut**, 3-level RBAC.
  - **Architecture:** strangler-fig user-management migration → **30% efficiency gain**.
  - No filler, no "basically."
- [ ] Curveball prep:
  - **LazyInitializationException** — Hibernate proxy outside a transaction → fix with `@Transactional(readOnly=true)` on the serialising method or `@EntityGraph`.
  - **Multi-tenancy** — tenant ID in JWT → Hibernate `@Filter` + PostgreSQL RLS as backstop.

**Loop execution + STAR #1 (rest of day):**
- [ ] **Execute scheduled loop(s).** Lead with impact numbers; ask a researched question; surface immediate availability if start dates come up — "available to join immediately."
- [ ] **STAR #1 — Performance win:** rehearse aloud to <2 min. 60s → 2–3s, the diagnosis, the per-layer fix, "96%" spoken naturally.
- [ ] **Decompress between rounds:** after each loop, 15 min away from the screen before the next — water, walk, reset. A fried third round loses offers.

---

### 📋 Pre-Interview Checklist (use before EVERY loop)

**Evening before (~30 min):**
- [ ] Re-read the JD — which of your 3 projects is most relevant? Which 2 tech areas appear most?
- [ ] Research the company: product, stack (engineering blog), recent news. Write 3 specific things.
- [ ] Prepare 3 questions; at least one references something specific you found.
- [ ] Say the most-relevant project talk-track aloud once.
- [ ] Check Glassdoor/Blind for recent interview reports; note recurring types. Log everything.

**Morning of (15 min):**
- [ ] Virtual round: test camera, mic, screen share, internet; phone backup ready; LeetCode editor open in a tab.
- [ ] Say the differentiator pitch aloud once. Review the 3 questions so they feel natural.
- [ ] Water ready. Tracker open. 5 min before: close all notifications.

**During:**
- [ ] Clarify the problem before coding (repeat it back, confirm constraints, ask about edges).
- [ ] Think aloud. State time/space complexity after every solution, unprompted.
- [ ] Behavioral: end every answer with a number or concrete outcome. Surface immediate availability when start dates come up.

**After (5 min):**
- [ ] Log every question asked while fresh; note what went well and what to change.
- [ ] Send a thank-you note within 4 hours if the process allows.

---

**Pipeline:** Loops come first today. Confirm receipts on any pending application; respond to recruiters within hours. Log every round outcome in the tracker the same day.

**EOD check:**
- [ ] Did you rehearse the relevant project deep-dive *before* the round, not after?
- [ ] Can you tell the performance story in under 2 min with "96%" landing naturally?
- [ ] Every loop today debriefed in the tracker?
- [ ] DSA warm-up done?

---
*Nav: ← [Week 7](../week-07/) · [Week 8](README.md) · [Day 2 (Tue Aug 18)](02-tue-aug-18.md) →*
