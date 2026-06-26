# Week 8 — Interview Loops Cluster (Aug 17–23, 2026)

> The loops are here. This is the week the work pays off — full interview rounds cluster, and scheduled loops now override new-application volume. You are not studying anymore; you are performing, then resetting, then performing again. The rhythm changes: rehearse the relevant project deep-dive before each round, deliver behavioral STAR stories with a number in every answer, drive system-design loops end-to-end, and decompress deliberately between rounds so you arrive at the next one sharp. DSA drops to light maintenance — one quick problem to stay warm. Applications taper to a trickle (8–12 new) with heavy follow-up, because a live loop you nail is worth more than ten cold applies you rushed.

> 📨 **Apps & referrals this week:** 8–12 new applications + heavy follow-up — scheduled interviews take priority over new volume. See the [cadence & tracker](applications-and-referrals.md).

---

## 🎯 Week Goal

1. Execute every scheduled loop researched, calm, and rehearsed — arriving with the right project deep-dive fresh and 3 sharp questions ready.
2. Deliver all 10 STAR stories (6 core + 4 bar-raiser) fluently, each under 2 minutes, each with a specific number.
3. Drive system-design loops end-to-end: requirements → capacity → design → deep dives → failure modes → observability, tying decisions to your real work.
4. Rehearse the Smart360 / Deep Fathom / WebX project deep-dives so any "tell me about a project" lands in 4–5 minutes with no filler.
5. Keep the pipeline alive with 8–12 new applications + heavy follow-up — without ever sacrificing prep for a live loop.

---

## ✅ By Sunday you can...

- Walk into any loop having rehearsed the most relevant project deep-dive that morning, with a company-specific angle in your intro.
- Deliver any of the 10 STAR stories under 2 minutes, with a number in every answer.
- Drive a system-design loop (chat/messaging, notification, file storage) end-to-end without being pulled through it.
- Explain the full-stack integration story — CORS, JWT token flow, API error contract — as one coherent narrative.
- Field the high-frequency curveballs (LazyInitializationException, @Transactional self-invocation, crashed worker recovery, Key Vault outage) without pausing.
- Decompress and reset between back-to-back rounds so the third loop is as sharp as the first.
- State "available to join immediately" naturally when a recruiter discusses start dates — and know it shortens their decision.

---

## 📅 Daily Checklist (Mon–Sun)

---

### Monday Aug 17 — Loop Execution + Performance-Win Deep Dive

📌 **Study today:** light DSA (LC 1) · Smart360 deep-dive rehearsal · execute scheduled loop(s) + STAR #1 (performance win)

**Pre-day check (5 min):**
- [ ] Open the tracker. Confirm every loop this week is on the calendar with prep time blocked the evening before. Any loop today/tomorrow? Run the **pre-interview checklist** (bottom of this file) now.

**Light DSA Maintenance (15–20 min):**
- [ ] **LC 1 – Two Sum** or any easy you can clear in 5 min. The goal this week is to *stay warm*, not learn — loops are the priority.

**Project Deep-Dive Rehearsal (40 min) — Smart360 (do before any round):**
- [ ] Rehearse the 5-minute Smart360 talk-track aloud: multi-tenant SaaS; perf (Hibernate stats → N+1 → JOIN FETCH/EntityGraph + composite indexes + Redis + S3-URL caching → 96% cut); security (stateless JWT at the gateway → 60% sign-in latency cut, 3-level RBAC); architecture (strangler-fig user-management migration → 30% efficiency gain). No filler, no "basically."
- [ ] Curveball prep: **LazyInitializationException** (Hibernate proxy outside a transaction → fix with `@Transactional(readOnly=true)` on the serialising method or `@EntityGraph`); multi-tenancy (tenant ID in JWT → Hibernate `@Filter` + PostgreSQL RLS as backstop).

**Loop Execution + STAR #1 (rest of day):**
- [ ] **Execute scheduled loop(s).** Lead with impact numbers; ask a researched question; surface immediate availability if start dates come up.
- [ ] **STAR #1 — Performance win:** rehearse aloud to <2 min. 60s→2–3s, the diagnosis, the per-layer fix, "96%" spoken naturally.
- [ ] **Decompress between rounds:** after each loop, 15 min away from the screen before the next — water, walk, reset. A fried third round loses offers.

**Self-Check**
- [ ] Did you rehearse the relevant project deep-dive *before* the round, not after?
- [ ] Can you tell the performance story in under 2 min with "96%" landing naturally?

---

### Tuesday Aug 18 — Loop Execution + Deep Fathom Deep Dive + STAR #2/#3

📌 **Study today:** light DSA (LC 102) · Deep Fathom deep-dive rehearsal · loop execution + STAR #2 (conflict) & #3 (ownership)

**Pre-day check (5 min):**
- [ ] Check the schedule; run the pre-interview checklist for any round today/tomorrow.

**Light DSA Maintenance (15–20 min):**
- [ ] **LC 102 – Binary Tree Level Order Traversal** — BFS, under 10 min. Warm-up only.

**Project Deep-Dive Rehearsal (40 min) — Deep Fathom:**
- [ ] Rehearse the 5-minute Deep Fathom talk-track: 30+ Azure resources in modular Bicep (Container Apps, PostgreSQL Flexible Server, Redis, Key Vault, Front Door, private endpoints); CI/CD redesign (sequential → DAG with `needs:`, BuildKit `--cache-from` → 23 min to 10 min, 57% cut); K8s probes + graceful shutdown; Key Vault via managed identity; PostgreSQL RLS across 50+ tables; FedRAMP posture.
- [ ] Curveball prep: **Key Vault outage at startup** (deploy fails safe; runtime secrets cached in-memory, running pods unaffected); **Bicep vs Terraform** (Bicep = Azure-native, no state file; Terraform if multi-cloud).

**Loop Execution + STAR #2/#3 (rest of day):**
- [ ] **Execute scheduled loop(s).**
- [ ] **STAR #2 — Conflict/disagreement:** a real disagreement, your counter-evidence (a comparison doc / time-boxed POC), outcome, relationship intact. <2 min.
- [ ] **STAR #3 — Ownership/leadership:** you fixed something nobody owned (e.g., the slow CI/CD pipeline) — assessed scope, proposed a plan, executed, measured (23→10 min). <2 min.
- [ ] Decompress between rounds.

**Self-Check**
- [ ] Does the Deep Fathom story land in 4–5 min with the 57% cut tied to DAG + BuildKit specifically?
- [ ] Are the conflict and ownership stories backed by a specific number/outcome?

---

### Wednesday Aug 19 — System-Design Loop + WebX Deep Dive + Full-Stack Story

📌 **Study today:** light DSA (LC 146) · WebX deep-dive + full-stack integration (CORS/JWT/error contract) · system-design loop execution

**Pre-day check (5 min):**
- [ ] Schedule check; pre-interview checklist for any round today/tomorrow.

**Light DSA Maintenance (20 min):**
- [ ] **LC 146 – LRU Cache** — doubly-linked list + HashMap. Implement once to keep it cold-ready; it's the most-asked design problem.

**Project Deep-Dive + Full-Stack Rehearsal (50 min):**
- [ ] Rehearse the 4-minute WebX talk-track: async job architecture (POST → 202 + jobId; PostgreSQL job record; Redis Streams queue; worker pool routes across 5 providers by cost/capability/rate-limit headroom tracked in Redis INCR/EXPIRE; SSE vs polling for results; idempotency keys).
- [ ] **Full-stack integration story (say it as one narrative):** Vue/React → API Gateway (JWT validation + CORS) → Spring Boot. Axios interceptors (request attaches Bearer token; response handles 401 → refresh → retry). `@RestControllerAdvice` → consistent JSON error shape; form errors inline, system errors as a toast.
- [ ] Curveball prep: **crashed worker mid-job** (heartbeat + watchdog `@Scheduled` marks STALE after 2 min → re-queue; idempotent provider calls prevent double-billing); **SSE vs WebSocket** (SSE unidirectional, simpler to proxy — right for read-only streaming); **CORS with credentials** (no wildcard origin; `Access-Control-Allow-Credentials: true` + `withCredentials`).

**System-Design Loop Execution (rest of day):**
- [ ] **Execute any SD loop.** If you get to pick / it leans real-time, **chat / messaging** pairs with your SSE/WebSocket knowledge: WebSocket connection routing (`userId → server` map in Redis), presence via heartbeat TTLs, per-conversation sequence ordering (not wall-clock), SENT→DELIVERED→READ receipts, small-vs-large group fan-out (the same hybrid as a news feed), wide-column message store partitioned by `conversation_id`. Drive it; name the trade-offs.
- [ ] Decompress between rounds.

**Self-Check**
- [ ] Can you justify SSE over WebSocket and recover a crashed worker without hesitating?
- [ ] Did you drive the SD loop through failure modes + observability, not just the happy path?

---

### Thursday Aug 20 — Loop Execution + Bar-Raiser STAR Stories + Application Trickle

📌 **Study today:** light DSA (LC 207) · bar-raiser STAR #7–#10 · loop execution + 3–4 applications + follow-ups

**Pre-day check (5 min):**
- [ ] Schedule check; pre-interview checklist for any round today/tomorrow.

**Light DSA Maintenance (20 min):**
- [ ] **LC 207 – Course Schedule** — cycle detection / topo sort (Kahn's or DFS). Under 15 min.

**Core Block (45 min) — Bar-Raiser STAR Stories (#7–#10):**

The 6 core stories cover most rounds; these 4 are the values / bar-raiser rounds at product companies. Rehearse each as a real anecdote, <2 min, a number in each:
- [ ] **#7 Mentorship:** unblocked a teammate by teaching the *why* + leaving a runbook; quantify their ramp.
- [ ] **#8 Receiving tough feedback:** someone told you something hard (PRs too large / over-engineered abstraction); you made a concrete visible change; closed the loop.
- [ ] **#9 Prioritization under pressure:** explicit must-have vs nice-to-have, cut by reversibility + user impact, hit the date.
- [ ] **#10 Customer/stakeholder impact:** lead with the *user/business* number, then the engineering metric (customer impact first, engineering detail second).

**Loop Execution + Pipeline (rest of day):**
- [ ] **Execute scheduled loop(s).**
- [ ] **Pipeline (45 min) — but loops come first:** apply to 3–4 new roles only if loop prep is done. Heavy follow-up: nudge stalled referrals once, chase any application 5+ days quiet, confirm receipts. Respond to recruiters within hours — and surface immediate availability when they discuss start dates.
- [ ] Decompress between rounds.

**Self-Check**
- [ ] Are all 4 bar-raiser stories anchored in a real anecdote with a number?
- [ ] Did loop prep stay ahead of the application trickle?

---

### Friday Aug 21 — Loop Execution + Curveball Drill + Debrief

📌 **Study today:** light DSA (LC 3) · Spring/Java curveball rapid-fire · loop execution + same-day debriefs

**Pre-day check (5 min):**
- [ ] Schedule check; pre-interview checklist for any round today/tomorrow.

**Light DSA Maintenance (15 min):**
- [ ] **LC 3 – Longest Substring Without Repeating Characters** — sliding window, under 12 min.

**Core Block (40 min) — Spring/Java Curveball Rapid-Fire (answer aloud in <90s each):**
- [ ] `@Transactional` self-invocation — why it's ignored and how to fix (proxy bypass).
- [ ] `@Transactional` + `@Cacheable` proxy ordering — stale cache after rollback; fix with `@TransactionalEventListener(AFTER_COMMIT)`.
- [ ] Circuit breaker (Resilience4j) — three states + half-open probe behaviour.
- [ ] OOMKilled pod — diagnosis (`kubectl describe` → OOMKilled) and fix (heap vs limit sizing).
- [ ] Readiness fails forever but liveness passes — pod stays Running, removed from Endpoints, no restart.
- [ ] CompletableFuture vs Future — non-blocking composition (`thenApply`/`thenCompose`/`exceptionally`).
- [ ] Track which ones you hesitate on; re-drill Sunday.

**Loop Execution + Debrief (rest of day):**
- [ ] **Execute scheduled loop(s).**
- [ ] **Same-day debrief** after every round: log every question asked (while fresh), what went well, what to change. Send a thank-you note within 4 hours where allowed.
- [ ] Decompress between rounds.

**Self-Check**
- [ ] Which curveball group had the most hesitations? (Re-drill Sunday.)
- [ ] Is every round this week debriefed the same day?

---

### Saturday Aug 22 — Full Mock Day + Pipeline Maintenance

📌 **Study today:** timed DSA mock (LC 21, 15/560) · full behavioral + system-design mock · pipeline maintenance

**DSA Mock (45 min):**
- [ ] Contest mode: **LC 21 – Merge Two Sorted Lists** (10-min warm-up) + **LC 15 – 3Sum** or **LC 560 – Subarray Sum Equals K** (25 min). State complexity unprompted.

**Full Mock (90 min) — behavioral + system design:**
- [ ] **Behavioral (30 min):** run TMAY → performance win → conflict → failure → why leaving → why company, back to back, recorded. Every answer ends with a number/outcome.
- [ ] **System design (45 min):** drive one prompt (notification system / file storage / news feed) end-to-end; self-critique against the rubric (requirements, capacity, deep dives, failure modes, observability, one tie to your work).
- [ ] **Review (15 min):** filler words? numbers in every behavioral answer? failure modes in the SD? complexity stated unprompted?

**Pipeline Maintenance (60 min) — loops still come first:**
- [ ] Apply to 3–4 new roles to keep the 8–12 weekly trickle alive.
- [ ] Heavy follow-up: every quiet application + every stalled referral gets its one nudge. Tracker quality pass — every active row has a next action and current stage.
- [ ] For any new loop invite: confirm time, research the interviewer, add prep notes.

**Self-Check**
- [ ] Did the full mock surface a specific gap to fix Sunday?
- [ ] Is the weekly application trickle at 8–12 with heavy follow-up done?

---

### Sunday Aug 23 — Reset + Weak-Area Review + Week 9 Plan (lighter day)

📌 **Study today:** weak-area re-drill (the shakiest curveball group + one DSA gap) · cold project-story run-through · Week 9 plan

**Weak-Area Review (60 min):**
- [ ] Re-drill the curveball group with the most Friday hesitations until each answer is <90s clean.
- [ ] Solve one DSA problem from a pattern that felt slow in a loop this week, timed, no hints.

**Cold Project-Story Run-Through (45 min):**
- [ ] Smart360, Deep Fathom, WebX — say each talk-track cold, no warm-up, timed (≤5 min each). Then the 60-second career-arc linker. If any runs long, cut the bloated sentence.

**Week 9 Plan (30 min):**
- [ ] Update the tracker: status for every application, every recruiter interaction, every loop outcome. Note any verbal interest or next-stage invite.
- [ ] If loops keep clustering, hold this rhythm (light DSA + deep-dive rehearsal + decompress). If the pipeline is thinning, line up 2 new applications for Monday.
- [ ] Read the funnel: with cumulative applications past 40, are loops converting to next rounds? If a specific round type keeps stalling (SD, behavioral, a curveball area), that's next week's targeted fix.
- [ ] Write 2–3 things you can do now that you couldn't last Sunday (ran back-to-back loops; recovered a botched answer; decompressed and reset cleanly).

---

## Pre-Interview Checklist (use before EVERY loop)

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

## 🧠 This Week's Operating Principles

### Scheduled loops override new volume
The cadence is explicit now: a live loop you nail beats ten cold applies you rushed. Applications drop to an 8–12 trickle with heavy follow-up; prep for the next round always wins the time conflict. Don't sabotage a loop to hit a volume number.

### Rehearse the right deep-dive *before* each round
Match the project to the company: a perf-heavy role → Smart360; an infra/DevOps role → Deep Fathom; an async/LLM role → WebX. Say the talk-track aloud the morning of, so it's fresh, not recalled.

### Decompress and reset between rounds
Back-to-back loops are an endurance test. A 15-minute reset between rounds (away from the screen) is the difference between a sharp third loop and a fried one. Treat recovery as part of the performance.

### Numbers in every behavioral answer
"We improved performance" proves you watched; "60s → 2–3s, a 96% reduction, 80% fewer S3 calls" proves you owned it. Every STAR story lands a specific number — that's the ownership signal.

### Immediate availability shortens the close
When a recruiter or HM discusses start dates, "available to join immediately" is a concrete advantage — it shortens their decision and can tip a close in a clustered, competitive window. Surface it whenever timelines come up.

---

## 📊 End-of-Week Self-Assessment

Fill Sunday. Anything below target is Week 9 Day 1.

| Area | Target | Your Score | Notes |
|---|---|---|---|
| Every scheduled loop executed with prep done | 5 | | |
| Project deep-dive rehearsed before each round | 5 | | |
| 10 STAR stories deliverable <2 min, number in each | 5 | | |
| System-design loop driven end-to-end | 5 | | |
| Full-stack integration story (CORS/JWT/error contract) | 4 | | |
| Curveball rapid-fire (Spring/Java) <90s each | 4 | | |
| Decompress/reset between back-to-back rounds | 4 | | |
| Every round debriefed same day | 5 | | |
| Light DSA maintenance streak | 3 | | |
| 8–12 new applications + heavy follow-up | 4 | | |
| Immediate availability surfaced when timelines arose | 5 | | |

**Week 8 exit criteria (minimum bar before Week 9):**
- Every scheduled loop executed with the pre-interview checklist completed.
- All 10 STAR stories spoken aloud at least once this week, each under 2 min with a number.
- At least one system-design loop or mock driven end-to-end with failure modes.
- Every live round debriefed in the tracker the same day.
- Pipeline kept alive (8–12 new) with heavy follow-up — without sacrificing loop prep.

**If you missed the bar:** prioritise the loop-execution and STAR items — a missing story or an unprepped round costs you the offer, not the application you didn't send. Carry application shortfall, never prep shortfall.

---

*Week 8 of the execution phase — next: Week 9 (Mon Aug 24 – Sun Aug 30, 2026).*
