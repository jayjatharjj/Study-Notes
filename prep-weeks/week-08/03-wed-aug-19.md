# Week 8 · Day 3 — Wed Aug 19 — System-Design Loop + WebX Deep Dive + Full-Stack Story

📌 **Today:** An async/LLM round means lead with WebX. Rehearse the WebX deep-dive and the full-stack integration narrative before the round, then drive any system-design loop end-to-end — requirements through failure modes and observability, not just the happy path.

**DSA maintenance (~30 min):** Light — loops take priority.
- [ ] **LC 146 – LRU Cache** — doubly-linked list + HashMap. Implement once to keep it cold-ready; it's the most-asked design problem.

**Primary activity:**

**Pre-day check (5 min):**
- [ ] Schedule check; **pre-interview checklist** (see Day 1) for any round today/tomorrow.

**Project deep-dive + full-stack rehearsal (50 min):**
- [ ] Rehearse the 4-minute **WebX talk-track** aloud — async job architecture:
  - POST → 202 + jobId; PostgreSQL job record; Redis Streams queue.
  - Worker pool routes across **5 providers** by cost/capability/rate-limit headroom (tracked in Redis `INCR`/`EXPIRE`).
  - SSE vs polling for results; idempotency keys.
- [ ] **Full-stack integration story (say it as one narrative):**
  - Vue/React → API Gateway (JWT validation + CORS) → Spring Boot.
  - Axios interceptors: request attaches Bearer token; response handles 401 → refresh → retry.
  - `@RestControllerAdvice` → consistent JSON error shape; form errors inline, system errors as a toast.
- [ ] Curveball prep:
  - **Crashed worker mid-job** — heartbeat + watchdog `@Scheduled` marks STALE after 2 min → re-queue; idempotent provider calls prevent double-billing.
  - **SSE vs WebSocket** — SSE unidirectional, simpler to proxy — right for read-only streaming.
  - **CORS with credentials** — no wildcard origin; `Access-Control-Allow-Credentials: true` + `withCredentials`.

**System-design loop execution (rest of day):**
- [ ] **Execute any SD loop.** If you get to pick / it leans real-time, **chat / messaging** pairs with your SSE/WebSocket knowledge:
  - WebSocket connection routing (`userId → server` map in Redis).
  - Presence via heartbeat TTLs.
  - Per-conversation sequence ordering (not wall-clock).
  - SENT → DELIVERED → READ receipts.
  - Small-vs-large group fan-out (the same hybrid as a news feed).
  - Wide-column message store partitioned by `conversation_id`.
  - **Drive it; name the trade-offs.**
- [ ] **Decompress between rounds:** 15 min away from the screen before the next round.

**Pipeline:** Loops first. Respond to recruiters within hours; surface immediate availability on timelines. Log every round same day.

**EOD check:**
- [ ] Can you justify SSE over WebSocket and recover a crashed worker without hesitating?
- [ ] Did you drive the SD loop through failure modes + observability, not just the happy path?
- [ ] Every loop today debriefed in the tracker?
- [ ] DSA warm-up done?

---
*Nav: ← [Day 2 (Tue Aug 18)](02-tue-aug-18.md) · [Week 8](README.md) · [Day 4 (Thu Aug 20)](04-thu-aug-20.md) →*
