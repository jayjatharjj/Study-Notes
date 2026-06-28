# Week 8 · Day 5 — Fri Aug 21 — Loop Execution + Curveball Drill + Debrief

📌 **Today:** Rapid-fire the Spring/Java curveballs until each answer is <90s clean, execute scheduled loop(s), and debrief every round the *same day* while it's fresh. Track which curveball group makes you hesitate — that's Sunday's re-drill.

**DSA maintenance (~30 min):** Light — loops take priority.
- [ ] **LC 3 – Longest Substring Without Repeating Characters** — sliding window, under 12 min.

**Primary activity:**

**Pre-day check (5 min):**
- [ ] Schedule check; **pre-interview checklist** (see Day 1) for any round today/tomorrow.

**Core block (40 min) — Spring/Java Curveball Rapid-Fire (answer aloud in <90s each):**
- [ ] `@Transactional` self-invocation — why it's ignored and how to fix (proxy bypass).
- [ ] `@Transactional` + `@Cacheable` proxy ordering — stale cache after rollback; fix with `@TransactionalEventListener(AFTER_COMMIT)`.
- [ ] Circuit breaker (Resilience4j) — three states + half-open probe behaviour.
- [ ] OOMKilled pod — diagnosis (`kubectl describe` → OOMKilled) and fix (heap vs limit sizing).
- [ ] Readiness fails forever but liveness passes — pod stays Running, removed from Endpoints, no restart.
- [ ] CompletableFuture vs Future — non-blocking composition (`thenApply` / `thenCompose` / `exceptionally`).
- [ ] **Track which ones you hesitate on; re-drill Sunday.**

**Loop execution + debrief (rest of day):**
- [ ] **Execute scheduled loop(s).** Lead with numbers; surface immediate availability on timelines.
- [ ] **Same-day debrief** after every round: log every question asked (while fresh), what went well, what to change. Send a thank-you note within 4 hours where allowed.
- [ ] **Decompress between rounds:** 15 min away from the screen before the next round.

**Pipeline:** Loops first. Heavy follow-up on quiet applications; respond to recruiters within hours. Every round logged same day.

**EOD check:**
- [ ] Which curveball group had the most hesitations? (Re-drill Sunday.)
- [ ] Is every round this week debriefed the same day?
- [ ] DSA warm-up done?

---
*Nav: ← [Day 4 (Thu Aug 20)](04-thu-aug-20.md) · [Week 8](README.md) · [Day 6 (Sat Aug 22)](06-sat-aug-22.md) →*
