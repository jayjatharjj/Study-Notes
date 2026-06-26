# Week 5 — Consolidation & Apply-Prep (Jul 27–Aug 1, 2026)

> The blade is sharp. This week you consolidate — DSA becomes maintenance, system designs become fluent walk-throughs, your 10 STAR stories and three project deep-dives become muscle memory — and you set up the application machine: profiles live, tiered target list built, and the first referral connections going out. This is the bridge from pure study to active applications.
>
> **Full-time week — Mon–Sat (6 days), Sun rest.** Three blocks per day: **Block A** (DSA maintenance / mock, ~2.5 hr), **Block B** (system-design mocks + profiles, ~2 hr), **Block C** (behavioral / project deep-dives / apply-prep, ~1.5 hr).

> 📨 **Apps & referrals this week:** build your tiered target-company list (~30–40 cos) + connect with 15–20 engineers at Tier-1/2 targets (connect only). See the [cadence & tracker](applications-and-referrals.md).

---

## 🎯 Week Goal

Turn everything from Weeks 0–4 into interview-ready fluency. Solve 2 mixed problems in 45 min under OA conditions, on demand, any day. Drive five core HLDs cold — URL shortener, file storage, News Feed, notification, chat/typeahead — with requirements → capacity → architecture → deep-dive → failure modes → trade-offs, each tied to your own work. Write and rehearse all 10 STAR stories (6 core + 4 bar-raiser) and the three project talk-tracks (Smart360, Deep Fathom, WebX) until each lands under 2–5 minutes with a number in every answer. Finish the week with profiles live (LinkedIn, Naukri, Instahyre/Cutshort, GitHub), a tiered ~30–40-company list, and 15–20 referral connections in flight — ready to apply Aug 3.

---

## ✅ By Saturday you can...

- Solve 2 LeetCode problems in 45 min under OA conditions, back-to-back, with correct complexity stated — on any day, cold.
- Whiteboard from memory, in under 10 minutes each: URL shortener, scalable file storage, News Feed/timeline, notification system, and chat/typeahead — covering requirements, capacity estimation, architecture, 2–3 deep-dives, failure modes, and trade-offs.
- Deliver any of the 10 STAR stories under 2 minutes with a specific number in every answer, plus the 90-second "tell me about yourself" pitch.
- Narrate the Smart360, Deep Fathom, and WebX talk-tracks in under 5 minutes each, no notes, no filler, and field the standard curveballs (LazyInitializationException, Key Vault outage, crashed worker, multi-tenancy).
- Explain the full-stack integration story — CORS (preflight + credentials), JWT client flow with 401/refresh handling, the API error contract — as one coherent narrative.
- Design a full-stack LLM async-job feature end-to-end (202 Accepted, SSE vs WebSocket, idempotency, durable job state).
- Show recruiter-ready profiles (headline: "available to join immediately") and a tiered target-company list with referral contacts mapped.

---

## 📅 Daily Checklist

---

### Monday Jul 27 — DSA Maintenance + URL Shortener / File Storage Mock + STAR #1–#5

📌 **Study today:** DSA maintenance — timed mixed set · system-design mock: URL shortener + file storage · STAR stories #1–#5 (write + rehearse)

**DSA (Block A, ~2.5 hr) — Timed Mixed Set #1 (OA simulation):**

Set the timer, no hints, fresh editor — treat it as a real OA. Pick 2 across patterns (don't pre-announce the type):
- One DP or greedy from Week 4 (e.g. LC 322 Coin Change or LC 56 Merge Intervals).
- One heap or graph (e.g. LC 215 Kth Largest or LC 207 Course Schedule).
After the timer: write time/space (best and worst if they differ), the one-line key insight, and one follow-up the interviewer might ask, for each.

**Core Topic (Block B, ~2 hr) — System-Design Mock: URL Shortener + File Storage:**

*URL shortener (20 min):* requirements (read-heavy, ~100:1 read/write, <100ms redirect), capacity (write rate × retention → key length via base-62), short-key generation (counter + base-62, or hash + collision check), the redirect path (cache-aside in Redis → DB), and analytics as an async event. Trade-off: 301 (cacheable, loses click tracking) vs 302 (every hit reaches you).

*Scalable file storage (25 min) — drive it end-to-end:*
- **Requirements + capacity:** files up to 5 GB (chunked upload), read-heavy (~1:10 write/read), eventual consistency for content / strong for metadata, global → CDN. 10M users × 5 GB = 50 TB; 100K uploads/day × 10 MB = 1 TB/day.
- **Upload pipeline:** client → Upload Service (gets `fileId` + per-chunk pre-signed S3 URLs) → client uploads chunks *directly to S3* (no data through your servers) → notify → async assembly (`CompleteMultipartUpload`) → Metadata Service records `status=READY` → publish `FileUploaded`. "Same pre-signed-URL pattern as Smart360, extended to per-chunk for large files."
- **Metadata Service:** PostgreSQL (strong consistency), shard on `owner_id` (efficient "list my files"), read replicas for the read-heavy queries.
- **Read path + CDN:** Download Service validates permission → pre-signed URL (TTL 15 min), cache it in Redis (TTL 10 min, under expiry); CloudFront/Front Door in front of S3 for shared files. "Front Door via Bicep in Deep Fathom for exactly this CDN offload."
- **Replication + failure modes:** S3 cross-AZ by default (+ cross-region replication); PostgreSQL leader-follower (replication-lag implication on "list my files" → route post-upload reads to leader, or optimistic UI). Failure modes: orphaned chunks → S3 lifecycle deletes incomplete uploads after 24h; metadata leader dies → promote replica (CP); CDN stale after delete → invalidation on delete event (batch the calls). Common gaps: virus scanning, quota enforcement, audit logging.

**Behavioral (Block C, ~1.5 hr) — STAR Stories #1–#5 (write + rehearse):**

Write and speak each aloud, timed; every sentence carries a specific detail.
1. **Performance win** — Smart360 60s→2–3s (96%): Hibernate statistics found 47 queries/load (N+1) → `JOIN FETCH` + `@EntityGraph` → composite indexes `(tenant_id, created_at)`, `(user_id, status)` → Redis read cache (5-min TTL) → S3 URL cache (TTL = expiry − 30s buffer, 80% fewer S3 calls). Zero schema changes. Target 90s spoken.
2. **Conflict / disagreement** — a real design disagreement (JWT vs session / async event-driven vs sync REST / Bicep vs existing Terraform): documented both with trade-offs, proposed a time-boxed PoC or decision matrix, found the kernel of validity in their concern, reached consensus or escalated cleanly. Relationship intact, quantified outcome.
3. **Ownership / leadership** — owned the slow CI/CD nobody fixed: profiled with GitLab trace → 3 bottlenecks → DAG + BuildKit → 23→10 min (57%). Proactive identification, no one asked, measured impact.
4. **Biggest failure** — a real mistake with a measurable consequence (e.g. shipped the LLM-proxy routing without retry+fallback, assumed provider uptime → production incident). Honest consequence, specific lesson ("design for failure first"), follow-through. Under 90s.
5. **Why leaving** — ambition framing (scale, engineering culture, compensation that reflects the level), never negativity. Speak 3× until natural.

**Self-check:**
1. Did both timed problems finish within 45 min, with complexity stated?
2. Can you tell the performance story under 2 minutes with "96%" spoken naturally?

---

### Tuesday Jul 28 — DSA Maintenance + News Feed / Notification Mock + STAR #6–#10

📌 **Study today:** DSA maintenance · system-design mock: News Feed + Notification · STAR stories #6–#10 (bar-raiser)

**DSA (Block A, ~2.5 hr) — Timed Mixed Set #2:**

Two more mixed problems under OA conditions. Rotate patterns you didn't hit Monday (a tree/graph + a sliding-window/two-pointer, e.g. LC 994 Rotting Oranges + LC 3 Longest Substring Without Repeating). Grade yourself; mark anything over time for an extra rep.

**Core Topic (Block B, ~2 hr) — System-Design Mock: News Feed + Notification:**

*News Feed / Timeline (driven, 25 min)* — the highest-frequency HLD:
- **Requirements + scale:** read-heavy, ~100M DAU, avg ~500 follows, ~10:1 read/write. Open with the defining tension: **write amplification vs read amplification.**
- **Core decision — fan-out-on-write vs fan-out-on-read:** push (write the post id into every follower's precomputed Redis sorted set; O(1) reads, but a 1M-follower post = 1M writes) vs pull (store once, gather + merge at read time; cheap writes, expensive reads).
- **The hybrid (the answer):** push for normal users; for celebrity/high-follower accounts, pull their recent posts at read time and merge into the precomputed feed — caps write amplification while keeping reads fast for the 99%. Threshold is a tunable knob.
- **Feed cache:** per-user Redis sorted set (post_id → score), capped (~800); inactive users generated lazily on login.
- **Cursor pagination:** never `OFFSET/LIMIT`; opaque cursor = `(score, post_id)`, "items with score < cursor" — stable under concurrent writes.
- **Ranking:** separate **candidate generation** (fan-out) from **ranking** (scorer: recency, affinity, engagement velocity, content type).
- **Tie to work:** "fan-out-on-write is the outbox/Kafka pattern from Deep Fathom — `PostCreated` → Kafka → feed-builder workers update follower caches async; same decoupling as the notification service."

*Notification system (20 min):* four channels behind a common `Channel` interface (push/email/SMS/in-app); Template Service `(templateId, locale, vars)` → content; per-user preferences consulted before dispatch (category granularity — a muted-marketing user still gets security alerts); fan-out + per-user rate limit (Redis sliding window) + dedup via idempotency key (event_id+user_id+channel, SETNX/unique constraint); queue-backed per-channel workers with exponential-backoff retry + DLQ; delivery-status tracking (QUEUED→SENT→DELIVERED→READ/FAILED via provider webhooks). "Outbox on the write side, at-least-once + idempotency on the consumer side."

**Behavioral (Block C, ~1.5 hr) — STAR Stories #6–#10 (bar-raiser):**

6. **Tell me about yourself** — the 90-second pitch: current role → key achievements with numbers (60s→3s, 57% CI/CD, 30+ Bicep resources, 5-provider LLM proxy) → what you want → bridge to the company.
7. **Mentorship / helping a teammate** — unblocked a junior on the multi-tenant flow / N+1 debugging by teaching the *why* + leaving a runbook + following up. Quantify ramp.
8. **Receiving tough feedback** — someone told you something hard (PRs too large / over-engineered the LLM routing / didn't surface blockers early); listened without defensiveness, made a concrete visible change, closed the loop.
9. **Deadline / prioritization under pressure** — explicit must-have vs nice-to-have, cut by reversibility + user impact, communicated it, hit the date.
10. **Customer / stakeholder impact** — user value first, engineering metric second; started from user pain (dashboards timing out / LLM jobs appearing to hang), validated against the user outcome.

Mark each bar-raiser story with a real anecdote — a fabricated one collapses under one follow-up.

**Self-check:**
1. Did you open the News Feed with the write/read amplification trade-off and justify the celebrity-account hybrid unprompted?
2. Are all 10 STAR stories written and spoken at least once?

---

### Wednesday Jul 29 — DSA Maintenance + Chat / Typeahead Mock + Project Deep-Dives

📌 **Study today:** DSA maintenance · system-design mock: chat + typeahead · project deep-dive rehearsals (Smart360, Deep Fathom, WebX)

**DSA (Block A, ~2.5 hr) — Timed Mixed Set #3 + LRU/LFU:**

Two mixed problems, OA conditions. Include one design-DS — **LRU Cache (LC 146)** or **LFU (LC 460)** — the most-asked "design a data structure." Know LC 146 cold (doubly-linked list + HashMap).

**Core Topic (Block B, ~2 hr) — System-Design Mock: Chat + Typeahead:**

*Chat / messaging (driven, 25 min):*
- **WebSocket connection management:** persistent bidirectional socket per client (contrast SSE — chat needs client→server too). Millions of long-lived connections → a gateway layer mapping `userId → (server, connectionId)` in Redis so any backend can find a recipient; reconnects + heartbeats/ping-pong detect dead sockets.
- **Presence:** online/offline/last-seen via heartbeat TTL keys (`SETEX presence:userId 30s`); fan-out to contacts on change; eventual consistency is fine.
- **Message ordering:** per-conversation monotonic sequence number (not wall-clock) — clients order by sequence, immune to clock skew.
- **Delivery / read receipts:** SENT → DELIVERED → READ, flowing back as lightweight messages.
- **Group fan-out:** small groups → fan-out-on-write to each inbox; large channels → fan-out-on-read — same hybrid as News Feed.
- **Storage:** wide-column (Cassandra/HBase) partitioned by `conversation_id`, clustered by `message_seq` → efficient "fetch last N / paginate older" at write-heavy scale; TTL/archival tiering for old messages.

*Typeahead / autocomplete (20 min):* read-heavy, <100ms p99. Serving: client (debounced) → edge → sharded suggestion index (trie/FST sharded by prefix), each node stores **precomputed top-k** (walk-to-prefix-then-read, not request-time DFS). Rank by historical query frequency, not lexicographic. Offline pipeline: query logs → stream/batch aggregation → per-prefix counts → build/refresh trie → ship to shards via versioned blob swap (atomic pointer flip). Cache hot prefixes in Redis.

**Project Deep-Dives (Block C, ~1.5 hr) — Smart360 / Deep Fathom / WebX:**

Rehearse each talk-track aloud, recorded, until tight.
- **Smart360 (5 min):** multi-tenant SaaS, Java 17 / Spring Boot / Vue. Three contributions — performance (N+1 → JOIN FETCH/EntityGraph + indexes + Redis + S3 URL cache → 60s→2–3s, 96%); security (session→stateless JWT at the gateway, 60% sign-in latency cut; 3-level RBAC); architecture (strangler-fig user-management migration, notification boundary first → 30% efficiency gain). Curveballs: JWT filter testing (WireMock + `@WebMvcTest`/`MockMvc`); hardest bug (`LazyInitializationException` outside a transaction → `@Transactional(readOnly=true)` / `@EntityGraph`); multi-tenancy (tenant in JWT + Hibernate `@Filter` + Postgres RLS backstop).
- **Deep Fathom (5 min):** FedRAMP-aligned; 30+ Azure resources in modular Bicep (Container Apps, PostgreSQL Flexible, Redis, Key Vault, Front Door, private endpoints; one parameter file per env); DAG CI/CD + BuildKit → 23→10 min (57%); K8s probes + graceful shutdown; Key Vault via managed identity; RLS across 50+ tables. Curveball: Key Vault outage at startup (deploy fails safe; runtime secrets cached in memory survive an outage).
- **WebX (4 min):** LLM orchestration for 2–20 min jobs across 5 providers; POST → 202 + jobId; PostgreSQL job state + Redis Streams queue; worker pool routes by cost/capability/rate-limit headroom (Redis atomic INCR/EXPIRE); SSE/polling for results; idempotency keys. Curveball: worker pod killed mid-PROCESSING (heartbeat + `@Scheduled` watchdog marks STALE after 2 min, re-queues; idempotent calls prevent double-billing).

**Self-check:**
1. Did the chat design cover connection routing, sequence ordering, the receipt state machine, the group fan-out split, and the wide-column partition/cluster key — unprompted?
2. Do all three talk-tracks land in time with no "basically"?

---

### Thursday Jul 30 — DSA Maintenance + LLD Mock + Full-Stack LLM + CORS/Token

📌 **Study today:** DSA maintenance · LLD mock (parking lot / Design Twitter) + full-stack LLM feature design · CORS/token handling + full-stack integration story

**DSA (Block A, ~2.5 hr) — Timed Mixed Set #4:**

Two mixed problems, OA conditions — favour patterns rated weakest in the Week 4 self-assessment. State complexity unprompted; mention one alternative approach per problem.

**Core Topic (Block B, ~2 hr) — LLD Mock + Full-Stack LLM Feature:**

*LLD mock (45 min) — pick parking lot OR Design Twitter classes:*
- **Design Twitter as LLD:** clean responsibilities + defensible SOLID. `User` (id/profile, no feed logic), `Tweet` (immutable value object), `FollowGraph` (`follow`/`unfollow`/`followeesOf`, encapsulates the social graph), `FeedService` (`postTweet`/`getTimeline`, depends on `FollowGraph` + `TimelineMerger` via interfaces), `TimelineMerger` interface (heap-merge is one impl; chronological/ranked drop in). Defend: SRP (each has one reason to change), Open/Closed + DIP (swap chronological→ranked without editing `FeedService`), Strategy pattern (name it). Bridge to scale: "the in-memory `FollowGraph` is the LLD; at scale this becomes fan-out-on-write vs fan-out-on-read — interface boundaries stay identical." Test: could a new joiner add a "muted accounts" filter without modifying `FeedService`?

*Full-stack LLM async-job feature (15 min):* UI → POST /jobs → API Gateway → LLM Orchestration (validate, write `PENDING` to PostgreSQL, publish to Redis Streams/RabbitMQ, return 202 + jobId) → worker pool (route to provider by cost/capability/rate-limit, call, write result, `COMPLETED`, publish completion) → Status/Result Service (`GET /status` polling + SSE `/stream` for partial tokens) → UI (poll every 3s or subscribe to SSE; progress bar; FAILED → error + retry). Justify each: 202+async (LLM can take 20 min, sync HTTP times out); Redis for rate-limit tracking (atomic INCR/EXPIRE, distributed); SSE over WebSocket (unidirectional, HTTP/1.1, easy to proxy); PostgreSQL for job state (durable — crash leaves PENDING for recovery); idempotency (duplicate click returns the existing jobId via client key or content hash).

**Full-Stack Integration (Block C, ~1.5 hr) — CORS / Token Handling:**

- **CORS:** browser same-origin policy — the browser sends the request and blocks the *response* without `Access-Control-Allow-Origin`. With credentials: server `Access-Control-Allow-Credentials: true` + client `withCredentials`/`credentials:'include'`; wildcard `*` is illegal with credentials. Preflight OPTIONS for non-simple requests (PUT/DELETE, `Authorization` header) → 200 with allowed methods/headers; `maxAge` caches the preflight. Configure once at Spring Cloud Gateway so downstream services don't each need it.
- **JWT client flow:** access token in memory (NOT localStorage — XSS), refresh token in HttpOnly Secure SameSite cookie. Axios request interceptor attaches `Bearer`; response interceptor on 401 → call refresh (cookie sent automatically) → new token → retry; refresh also 401 → force logout. **Guard the recursion:** if the failing URL *is* `/auth/refresh`, don't retry — redirect to login (avoids an infinite loop). 403 → "access denied," do NOT retry. 5xx/network → friendly toast, never raw exception messages.
- **API error contract:** `@RestControllerAdvice` + `@ExceptionHandler` maps domain exceptions to consistent JSON (`timestamp/status/error/message/path`); `@Valid` → 400 field-level list; business rule → 409; downstream failure → clean 502/503. Frontend: form errors inline, system errors to toast, loading reset in `finally`.

Say the integration story aloud (under 90s): "Vue → Spring Boot through the gateway (JWT validation, CORS, routing); a global Axios instance with request + response interceptors; `@RestControllerAdvice` gives every error a consistent shape the frontend consumes uniformly."

**Self-check:**
1. Could a new joiner add a ranked feed to your Twitter LLD without modifying `FeedService`?
2. Can you explain why localStorage is an XSS risk and what happens step-by-step on a 401 mid-session?

---

### Friday Jul 31 — Full Mock + Profiles Live + Build the Target List

📌 **Study today:** full mock interview (DSA + system design, timed) · finalize resume + LinkedIn + Naukri + GitHub (headline: "available to join immediately") · build the tiered target-company list

**DSA + System Design (Block A, ~2.5 hr) — Full Mock (timed):**

Treat it as a real loop. Round 1 — DSA (45 min): a partner (Pramp/interviewing.io) or self-mock gives 2 problems; clarify before coding, narrate, state complexity before submitting. Round 2 — System Design (45 min): one of URL shortener / file storage / notification — requirements → capacity → high-level → 2–3 deep-dives → failure modes → trade-offs → one sentence on observability (golden signals, trace propagation, SLO burn-rate alerting). Write 3 things you did well + 3 to improve.

**Profiles (Block B, ~2 hr) — Finalize Resume + LinkedIn + Naukri + GitHub:**

- **LinkedIn:** headline `Full Stack Engineer · Java 17 · Spring Boot · Microservices · Azure · Kubernetes · LLM Integration · Available to join immediately`. About (3–4 sentences): quantified impact first (60s→3s, 57% CI/CD, 30+ Bicep), stack, what you want. Every experience bullet starts with an action verb and carries a number. Skills section keyword-dense.
- **Naukri:** mirror headline/summary; upload a clean PDF (1–2 pages); "Actively Looking" + immediate-joiner; skills with proficiency; expected-CTC band (set 20 LPA floor to avoid filtering).
- **Instahyre + Cutshort:** create profiles, set open-to-work, fill skills/expected-CTC, upload resume — high-signal inbound channels for your band; fills the pipeline passively.
- **GitHub:** pin 3–6 best repos, profile README (cloud-native + LLM narrative), prune embarrassing repos, one showcase repo with a clean README + architecture diagram + `docker-compose up`. Imperative, scoped commit messages.
- **First-screen CTC deflection (memorise now):** default deflect ("I'd rather focus on the role and total opportunity first…"); if pressed, give a researched band (₹18–24 LPA), never your current number; if a form forces it, enter full CTC (base + variable + employer PF + benefits).

**Apply-Prep (Block C, ~1.5 hr) — Build the Tiered Target-Company List:**

Build a ~30–40-company list, tiered:
- **Tier-1 (dream — GCC/product):** Goldman Sachs, Morgan Stanley, JPMC, Walmart Global Tech, Atlassian, Salesforce, Adobe, Visa, Mastercard, Razorpay, PhonePe, CRED, etc. — Java/Spring Boot in JD, 2–5 YOE, ₹18–30 LPA, Pune/Bengaluru/remote.
- **Tier-2 (strong product mid-tier):** Groww, Juspay, Navi, Slice, Postman, BrowserStack, Druva, Meesho, Zepto, Setu, Darwinbox.
- **Tier-3 (calibration / warm-ups):** established service + mid-size product companies for interview reps.
Columns: Company | Tier | Role | JD URL | Tech-stack check | Referral contact(s) | Status. Verify each stack (don't apply to .NET shops). This is the input for Week 6's referral-contact mapping and Week 8's first applications.

**Self-check:**
1. Is your LinkedIn headline keyword-dense with the immediate-joiner flag, and does every bullet have a number?
2. Is the tiered list ~30–40 companies with stacks verified?

---

### Saturday Aug 1 — Final Mock / Closeout + Referral Connections + Readiness Check

📌 **Study today:** final mock / weak-spot closeout · start 15–20 referral connections + ATS keyword routine · readiness check — ready to apply Aug 3

**DSA + Design (Block A, ~2.5 hr) — Final Mock / Weak-Spot Closeout:**

One last timed set targeting whatever rated lowest all week (a DP/heap/graph gap or the shakiest HLD). Re-solve or re-narrate until clean. Goal: zero open red flags going into applications.

**Referrals + ATS (Block B, ~2 hr) — Start 15–20 Connections:**

- For Tier-1/2 targets, find 1–2 engineers each (SDE2/Senior/Tech Lead, "Java"/"Spring Boot" in profile, 2nd-degree prioritized). Send **connect requests only** with a one-line genuine note (a specific detail about their work) — **no referral ask yet**; that comes in Week 8. Log each: Company | Name | LinkedIn URL | Date | Response.
- **ATS keyword routine:** keep a base resume + a 60-second tailoring habit — paste a JD beside the resume, mirror the top ~5 JD keywords using the JD's exact phrasing (e.g. "Spring Cloud Gateway," not just "API gateway"). Most sub-20-LPA applications are ATS-filtered before a human reads them.

**Readiness Check (Block C, ~1.5 hr) — Ready to Apply Aug 3:**

Run the readiness gate below. Anything not green becomes a Week 6 carry-forward — but do not start applying until the gate is met.

**Self-check:**
1. Are 15–20 connection requests sent and logged?
2. Does every readiness-gate item pass?

---

### Sunday Aug 2 — Rest

No study blocks. Let the consolidation settle before applications begin Aug 3. Optionally skim your STAR stories and the two weakest HLDs — but do not start new material.

---

## ✅ Readiness Gate — Ready to Apply Aug 3

You are ready to begin applications when ALL of these are true:

- [ ] **DSA fluency:** solve 2 mixed mediums in 45 min under OA conditions, cold, with complexity stated — demonstrated on at least 3 separate days this week.
- [ ] **Two system-design walk-throughs cold:** any two of URL shortener / file storage / News Feed / notification / chat-typeahead, narrated end-to-end from memory in under 10 minutes each, with failure modes, trade-offs, and a tie to your own work.
- [ ] **10 STAR stories:** all 10 written and spoken, each under 2 minutes with a specific number; the 90-second pitch fluent.
- [ ] **Three project talk-tracks:** Smart360, Deep Fathom, WebX — each under 5 minutes, no notes, curveballs handled.
- [ ] **Profiles live:** LinkedIn, Naukri, Instahyre, Cutshort, GitHub — all updated, keyword-dense, open-to-work set, resume uploaded.
- [ ] **Tiered list built:** ~30–40 companies, tiered, stacks verified, referral contacts mapped.
- [ ] **Immediate-joiner ready:** headline + profiles state "available to join immediately"; first-screen CTC-deflection line memorised.
- [ ] **Referral connections in flight:** 15–20 connect requests sent and logged.

**If any item is red:** carry it into Week 6's daily review slot, but prioritize closing it before sending applications — a live loop on an unprepped story or design costs the offer.

---

## 🧠 Concepts to Master This Week (consolidation, not new material)

### System Design — Five HLDs to Drive Cold
URL shortener (base-62 keys, 301 vs 302) · file storage (chunked pre-signed upload, owner_id sharding, CDN, replication-lag handling) · News Feed (fan-out write/read/hybrid, cursor pagination, candidate-gen vs ranking) · notification (channel abstraction, preferences-before-dispatch, dedup, retry + DLQ) · chat (connection routing, sequence ordering, receipts, wide-column storage) + typeahead (sharded trie, precomputed top-k, offline refresh). Always: requirements → capacity → architecture → 2–3 deep-dives → failure modes → trade-offs → observability sentence.

### Behavioral — The 10 STAR Stories
Core (1–6): performance win, conflict, ownership, biggest failure, why-leaving, tell-me-about-yourself. Bar-raiser (7–10): mentorship, receiving tough feedback, prioritization under pressure, customer impact. Every answer ends with a number or concrete outcome.

### Full-Stack Integration
CORS (preflight, credentials, gateway-level config, the `*`+credentials trap) · JWT client flow (memory + HttpOnly cookie, 401→refresh→retry, recursion guard, 403 handling) · API error contract (`@RestControllerAdvice`, consistent JSON, inline vs toast). LLM async job: 202, SSE vs WebSocket, durable job state, idempotency.

### Apply-Prep
Tiered list (Tier-1 GCC/product, Tier-2 mid-tier, Tier-3 calibration) · profiles live with immediate-joiner flag · referral-first (connect now, ask Week 8) · ATS keyword tailoring · first-screen CTC deflection.

---

## 🎤 Sample Interview Questions (incl. curveballs)

1. **Design a URL shortener.** → Capacity → base-62 key length; counter+base-62 vs hash+collision; redirect via Redis cache-aside; analytics async; 301 (cacheable) vs 302 (trackable).
2. **File storage — start with the upload flow.** → Chunked pre-signed URLs; data never flows through your servers; client → Upload Service → direct-to-S3 → complete. Smart360 single-file pattern extended to chunks.
3. **News Feed — fan-out on write or read?** → Hybrid: push for normal users, pull for celebrities above a follower threshold; caps write amplification, keeps reads fast for the 99%.
4. **(Curveball) Two users upload a file with the same name simultaneously.** → Name isn't the PK; `file_id` UUID is. For "unique per folder," a `(owner_id, folder_id, name)` constraint → 409 on the loser, enforced transactionally within a shard.
5. **Notification — ensure no duplicates.** → Idempotency key (event_id+user_id+channel) via SETNX or a unique constraint; duplicate notifications are the #1 user complaint.
6. **Chat — why order by sequence, not timestamp?** → Per-conversation monotonic sequence is immune to sender clock skew; wall-clock can reorder messages.
7. **Tell me about yourself.** → 90-second pitch: impact numbers first, stack, what you want, bridge to the company. Never recite the CV chronologically.
8. **Tell me about a time you were wrong.** → A confident technical assumption that proved wrong (sync REST fine under current load → cascading failures on bursty traffic); acknowledged it, moved to async + circuit breaker, now load-test bursty traffic before sign-off.
9. **(Curveball) Frontend on app.example.com, API on api.example.com — browser blocks it. Debug.** → DevTools Network → failed OPTIONS / missing header; check `Access-Control-Allow-Origin` exact match, credentials flags both sides, gateway path pattern, no trailing slash, no `*`+credentials.
10. **(Curveball) After 15 min idle, actions fail silently with no redirect.** → Access + refresh both expired; the 401 from `/auth/refresh` is being caught recursively. Fix: detect the refresh URL in the interceptor and redirect to login instead of retrying.
11. **Why SSE over WebSocket for the LLM stream?** → Unidirectional server→client, HTTP/1.1, simpler to proxy through the gateway; WebSocket's bidirectionality is overkill for a read-only token stream (chat is where you'd need WebSocket).
12. **(Curveball) WebX worker pod is killed mid-job.** → Heartbeat every 30s; a `@Scheduled` watchdog marks jobs STALE after 2 min and re-queues; idempotent provider calls (request ID) prevent double-billing.

---

## 🌟 Extraordinary-Candidate Edge

1. **Every behavioral answer has a number.** "Latency dropped from 60s to 2–3s, 96%, and S3 calls fell 80%" — metrics prove ownership; vague statements prove you watched.
2. **Design for failure first.** Open HLDs with failure modes and constraints, name patterns you deliberately avoided ("no 2PC — it couples services and risks availability"), quantify scale.
3. **Lead with the answer, not the journey.** "JWT is validated at the gateway on every request — no downstream service touches auth." Then explain. Weak candidates bury the answer.
4. **Anticipate the follow-up.** After CORS: "this lives at the gateway, not each service, so security policy is consistent without every team configuring it."
5. **Weave the three projects into one arc.** "Smart360 taught me performance engineering; Deep Fathom gave me infra ownership; WebX pushed me into async + LLM integration — now I want to own a domain end-to-end at scale."
6. **Referral-first, researched.** Connect with a specific detail about their work, not "I've always wanted to work here"; a referred application is ~4–5× more likely to get a callback.

---

## 📊 End-of-Week Self-Assessment

Rate yourself 1–5. Anything below 3 blocks the readiness gate.

| Area | Target | Your Score | Notes |
|---|---|---|---|
| Timed mixed OA set (2 in 45 min) — repeatable | 5 | | |
| LRU/LFU cache from scratch (LC 146/460) | 5 | | |
| URL shortener HLD (cold, <10 min) | 4 | | |
| File storage HLD (cold, <10 min) | 5 | | |
| News Feed HLD (fan-out/cursor/ranking) | 5 | | |
| Notification HLD (channel/dedup/retry/DLQ) | 4 | | |
| Chat + typeahead HLD | 4 | | |
| LLD: Design Twitter classes (SOLID defended) | 4 | | |
| Full-stack LLM async-job feature | 4 | | |
| CORS + JWT client flow + error contract | 5 | | |
| STAR #1–#6 (core, under 2 min, numbers) | 5 | | |
| STAR #7–#10 (bar-raiser, real anecdotes) | 4 | | |
| 90-second "tell me about yourself" | 5 | | |
| Smart360 talk-track (5 min, no notes) | 5 | | |
| Deep Fathom talk-track (5 min, no notes) | 5 | | |
| WebX talk-track (4 min, no notes) | 5 | | |
| Full mock completed (DSA + design) | 5 | | |
| Profiles live (LinkedIn/Naukri/Instahyre/Cutshort/GitHub) | 5 | | |
| Tiered target list (~30–40, stacks verified) | 5 | | |
| 15–20 referral connections sent + logged | 5 | | |
| First-screen CTC deflection memorised | 4 | | |
| Immediate-joiner flag set everywhere | 5 | | |

**Score interpretation:**
- All readiness-gate items green → begin applications Aug 3.
- Any HLD or STAR item below 3 → close it before applying; a live loop on an unprepped story costs the offer.
- Any DSA item below 3 → re-drill Week 6 Day 1.

**Bonus check:** Can you tie every system-design decision and every STAR story to a specific Smart360 / Deep Fathom / WebX number? If yes on 80%+ of rows, you are ready not just to apply, but to convert.

---

*Study complete — next: Week 6 (Mon Aug 3 – Sun Aug 9), applications begin.*
