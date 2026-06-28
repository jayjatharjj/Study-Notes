# Week 5 · Day 2 — Tue Jul 28 — DSA Maintenance + News Feed / Notification HLD + STAR #6–#10

> The two highest-frequency HLDs in a senior backend loop are the **News Feed** and the **notification system**, and they share one spine: *decide where the fan-out happens.* News Feed forces you to choose between write amplification and read amplification — and the answer that scores is the **hybrid** (push for the many, pull for the celebrity few). Notification is the same fan-out problem with a reliability twist — every channel behind one interface, dedup so no user gets the same alert twice, retry with a dead-letter queue so nothing is silently lost. On the behavioral side today, the **bar-raiser** stories (#6–#10): the 90-second "tell me about yourself," mentorship, receiving tough feedback, prioritisation under pressure, and customer impact. These are the ones that separate a hire from a strong hire — and a fabricated one collapses under a single follow-up, so each must be a *real* anecdote with a real number.

📌 **Study today:** DSA maintenance — timed mixed set #2 (rotate patterns) · system-design mock: News Feed + notification system (driven end-to-end) · STAR stories #6–#10 (bar-raiser) — write + speak · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA Maintenance: Timed Mixed Set #2

Same OA protocol as yesterday: 45-minute timer, fresh editor, no hints, complexity stated before "submit." **Rotate the patterns you didn't hit Monday** — today lean tree/graph + sliding-window/two-pointer.

### Today's pairing

- **LC 994 Rotting Oranges** (multi-source BFS on a grid) — or **LC 102 Binary Tree Level Order** if you want a gentler tree rep.
- **LC 3 Longest Substring Without Repeating Characters** (sliding window + hashmap) — or **LC 76 Minimum Window Substring** for a harder rep.

### Re-derivation cheats (glance only if stuck)

- **Rotting Oranges (994):** seed the queue with **all** rotten cells at once (multi-source BFS), increment a minute counter per BFS level; at the end, if any fresh orange remains → return -1. O(rows×cols). Key insight: multi-source BFS computes the *simultaneous* spread without nested loops.
- **Longest Substring w/o Repeating (3):** sliding window; a `Map<char,lastIndex>`; when you see a repeat **inside** the window, jump `left` to `max(left, lastIndex+1)`. O(n), O(min(n,charset)) space. The bug everyone hits: not clamping `left` with `max` (a stale index drags it backwards).
- **Min Window Substring (76):** expand right until the window is valid (all target counts met), then contract left while still valid, recording the min. O(n) with a `need`/`have` counter.

### Self-grade

Use yesterday's rubric (finished ≤45 min · complexity unprompted · no logic bugs). Mark anything over time or buggy for **one extra rep** — maintenance means closing small gaps fast, not re-learning.

---

## Block B — System-Design Mock: News Feed + Notification System

### News Feed / Timeline (driven, ~25 min) — the highest-frequency HLD

**1) Requirements + scale.** Functional: post; follow/unfollow; render a follower's home timeline (recent + relevant posts from followees). Non-functional: **read-heavy**, **~100M DAU**, average **~500 follows/user**, ~10:1 read/write. **Open with the defining tension out loud: write amplification vs read amplification.**

**2) Capacity (state it).** 100M DAU, each opening the feed several times/day → **hundreds of thousands of feed reads/sec**. Posts are far rarer than reads, but a single post by a high-follower account can imply *millions* of downstream writes if you fan out naively. That asymmetry is the whole design.

**3) API.**
```
POST /posts              { authorId, content } → 201 { postId }
GET  /feed?cursor=...     → { items:[...], nextCursor }
POST /follow             { followerId, followeeId } → 204
```

**4) The core decision — fan-out-on-write vs fan-out-on-read:**

| | Fan-out-on-write (push) | Fan-out-on-read (pull) |
|---|---|---|
| On post | write `postId` into **every follower's** precomputed feed (Redis sorted set) | store the post once |
| On read | **O(1)** — read the precomputed feed | gather all followees' recent posts + **merge at read time** |
| Cost | a 1M-follower post = **1M writes** (write amplification) | every read is a fan-in merge (read amplification) |
| Best for | normal users (most accounts) | celebrities / very high follower counts |

**5) The hybrid (this is the answer).** **Push for normal users; pull for celebrity/high-follower accounts.** At read time, take the user's precomputed feed and **merge in the recent posts of the few celebrities they follow**. This caps write amplification (you never fan out a 1M-follower post) while keeping reads fast for the 99%. **The follower threshold is a tunable knob** — name it explicitly.

**6) Feed cache.** Per-user **Redis sorted set** (`post_id → score`, score = timestamp or rank), **capped at ~800** entries. **Inactive users' feeds are generated lazily on login** — don't pay to maintain feeds for users who never read.

**7) Cursor pagination — never `OFFSET/LIMIT`.** Use an **opaque cursor = `(score, post_id)`** and query "items with score < cursor." `OFFSET` skips a fixed count and is **unstable under concurrent writes** (a new post shifts everything → duplicates or skips); a cursor anchors to a value, so it's stable.

**8) Ranking — separate the two stages:**
- **Candidate generation** = the fan-out (who *could* appear).
- **Ranking** = a scorer over the candidates (recency, **affinity** to the author, **engagement velocity**, content type).
- Keeping them separate lets you swap a chronological feed for a ranked one without touching fan-out.

**9) Tie to work.** *"Fan-out-on-write is the outbox/Kafka pattern from Deep Fathom — `PostCreated` → Kafka → feed-builder workers update follower caches asynchronously; same decoupling as the notification service."*

**10) Failure modes + trade-offs.** Feed-builder workers lag → feeds are eventually consistent (acceptable; show "you're caught up"). Redis feed lost → regenerate lazily from the post store. Trade-off summary: **hybrid trades a little read-time merge cost for a huge cap on write amplification** — the right call at this read/write ratio.

### Notification System (driven, ~20 min)

**1) Requirements.** Multi-channel: **push, email, SMS, in-app**. Respect **per-user preferences**. **No duplicates** (the #1 user complaint). **At-least-once delivery** with retries. **Delivery-status tracking.**

**2) Architecture (drive it).**
- **Channel abstraction:** four channels behind a common **`Channel` interface** (`send(notification)`); adding WhatsApp later is a new implementation, nothing else changes (Open/Closed).
- **Template Service:** `(templateId, locale, vars)` → rendered content — keeps copy out of code and supports i18n.
- **Preferences consulted *before* dispatch**, at **category granularity** — a user who muted marketing **still gets security alerts**. Check preferences *before* you fan out, not after you've sent.
- **Fan-out + per-user rate limit** via a **Redis sliding window** (don't blast a user with 50 emails in a minute).
- **Dedup via idempotency key** = `event_id + user_id + channel`, enforced with **SETNX** (or a unique constraint) → the second attempt is a no-op.
- **Queue-backed per-channel workers** with **exponential-backoff retry** and a **dead-letter queue (DLQ)** for messages that exhaust retries (so nothing is silently dropped; ops can inspect the DLQ).
- **Delivery-status state machine:** `QUEUED → SENT → DELIVERED → READ` (or `→ FAILED`), updated via **provider webhooks**.

**3) The one-liner that scores.** *"Outbox on the write side guarantees the event is recorded atomically with the business change; at-least-once delivery + idempotency on the consumer side guarantees it's delivered exactly once *effectively*, despite retries."*

**4) Failure modes.** Provider down → retries with backoff, then DLQ; a dependent channel (SMS gateway) outage doesn't block the others (per-channel workers are isolated). Duplicate event from the upstream → idempotency key absorbs it.

---

## Block C — STAR Stories #6–#10 (bar-raiser)

These decide *strong* hire vs hire. Each must be a **real anecdote** — interviewers probe, and a fabricated story collapses on the first "what would you do differently?" Write, then speak each aloud, timed, ending on a number or concrete outcome.

### #6 — Tell me about yourself (the 90-second pitch)

The most-asked question; never recite the CV chronologically. Structure: **current role → key achievements with numbers → what you want → bridge to the company.**

> "I'm a full-stack engineer working mostly in **Java 17 / Spring Boot** with **Azure and Kubernetes**. My impact has been measurable — I cut a flagship dashboard from **60 seconds to 2–3 seconds (96%)**, took our **CI/CD from 23 to 10 minutes (57%)**, built **30+ Azure resources in modular Bicep**, and shipped a **5-provider LLM proxy** for long-running async jobs. I'm now looking to **own a domain end-to-end at scale**, which is exactly what this role offers."

*Target: 90 seconds, impact numbers first, no filler.*

### #7 — Mentorship / helping a teammate

- **S:** A junior was stuck on the multi-tenant flow (and separately, an N+1 debugging session).
- **T:** Unblock them *and* level them up — not just hand over the fix.
- **A:** Taught the **why** (how tenant isolation propagates from the JWT through Hibernate filters), **left a runbook** so the next person wouldn't need me, and **followed up** a week later.
- **R:** Their ramp-up shortened measurably (e.g. they shipped the next multi-tenant feature solo); the runbook became team reference. *Teaching the why, not just the answer.*

### #8 — Receiving tough feedback

- **S:** A senior told me something hard — e.g. **my PRs were too large** to review well (or I'd over-engineered the LLM routing, or didn't surface blockers early enough).
- **T:** Take it without defensiveness and actually change.
- **A:** Listened, asked clarifying questions instead of defending, then made a **concrete, visible change** — split work into small PRs with a one-line intent each — and **closed the loop** by asking the reviewer if it was better.
- **R:** Review turnaround improved and the reviewer noticed unprompted. *Feedback received as a gift, not a threat.*

### #9 — Deadline / prioritisation under pressure

- **S:** A fixed launch date with more scope than time.
- **T:** Hit the date without shipping something broken.
- **A:** Made an explicit **must-have vs nice-to-have** split, cut by **reversibility and user impact** (defer anything reversible and low-impact), and **communicated the cut** to stakeholders *before* it became a surprise.
- **R:** Hit the date with the must-haves solid; the deferred items shipped the next cycle. *Prioritisation is a decision you communicate, not a corner you cut quietly.*

### #10 — Customer / stakeholder impact

- **S:** Users reported the dashboards **timing out** (or LLM jobs *appearing* to hang with no feedback).
- **T:** Lead with the **user outcome**, not the engineering metric.
- **A:** Started from the **user pain**, traced it to the root cause (the N+1 / missing async progress feedback), fixed it, and **validated against the user outcome** — not just the latency graph.
- **R:** Users stopped abandoning the page; satisfaction up, and the engineering number (96% faster) followed *because* the user goal drove it. *User value first, metric second.*

> **Anti-fabrication rule:** for each story, pre-load the answer to *"what would you do differently?"* — if you can't answer that honestly, the story isn't real enough to use.

---

## 💻 Practice coding questions

A lighter maintenance set — two timed for Block A, the rest as warm-ups / reps.

1. **Rotting Oranges (LC 994)** — multi-source BFS; minute = BFS level; -1 if fresh remain.
2. **Longest Substring Without Repeating (LC 3)** — sliding window + last-index map; clamp `left` with `max`.
3. **Minimum Window Substring (LC 76)** — expand-then-contract window with need/have counters.
4. **Binary Tree Level Order (LC 102)** — BFS by level; a clean tree warm-up.
5. **Number of Islands (LC 200)** — grid flood fill; contrast iterative BFS vs recursive DFS stack depth.
6. **Two-Pointer pair-sum (LC 167 Two Sum II)** — sorted-array two pointers; O(n), O(1).
7. **(Stretch) Course Schedule II (LC 210)** — topo order; same engine as 207, collect the sequence.
8. **(Stretch) Sliding Window Maximum (LC 239)** — monotonic deque; O(n) — a harder window rep.

---

## 🎤 Interview questions

1. **Design a News Feed — fan-out on write or read?** → Hybrid: push for normal users, pull for celebrities above a tunable follower threshold; caps write amplification, keeps reads fast for the 99%.
2. **Open with the core tension of a News Feed.** → Write amplification vs read amplification: push makes reads O(1) but a 1M-follower post = 1M writes; pull makes writes cheap but reads a fan-in merge.
3. **Why not `OFFSET/LIMIT` for the feed?** → It's unstable under concurrent writes — a new post shifts the window → duplicates/skips. Use an opaque `(score, post_id)` cursor and "score < cursor."
4. **How do you keep the feed cache bounded?** → Per-user Redis sorted set capped at ~800; inactive users' feeds generated lazily on login rather than maintained continuously.
5. **Candidate generation vs ranking — why separate them?** → Candidate generation is the fan-out (who could appear); ranking is a swappable scorer (recency/affinity/engagement). Separation lets you go chronological → ranked without touching fan-out.
6. **Notification — how do you guarantee no duplicates?** → Idempotency key `event_id+user_id+channel` via SETNX or a unique constraint; the second attempt is a no-op. Duplicate notifications are the #1 user complaint.
7. **A user muted marketing — should they still get a security alert?** → Yes. Preferences are checked before dispatch at *category* granularity; security/transactional categories bypass marketing mutes.
8. **What happens when a notification provider is down?** → Per-channel workers retry with exponential backoff; on retry exhaustion the message goes to a DLQ for inspection — never silently dropped. Other channels are unaffected (isolated workers).
9. **Why "outbox + at-least-once + idempotency" and not just "send it"?** → Outbox records the event atomically with the business change (no lost events); at-least-once + idempotency gives *effectively* exactly-once delivery despite retries.
10. **How does the News Feed reuse a pattern from your work?** → Fan-out-on-write is the outbox/Kafka pattern from Deep Fathom — `PostCreated` → Kafka → feed-builder workers update follower caches async; same decoupling as the notification service.
11. **Tell me about yourself.** → 90-second pitch: current role → numbers (60s→3s, 57% CI/CD, 30+ Bicep, 5-provider LLM proxy) → what I want → bridge to the company. Never chronological.
12. **Tell me about a time you mentored someone.** → Unblocked a junior on the multi-tenant flow by teaching the *why* + leaving a runbook + following up; their ramp shortened and the runbook became team reference.
13. **Tell me about tough feedback you received.** → "PRs too large to review" — listened without defensiveness, split work into small intent-labelled PRs, closed the loop; review turnaround improved.
14. **How do you prioritise under a tight deadline?** → Explicit must-have vs nice-to-have, cut by reversibility + user impact, communicate the cut before it surprises anyone; hit the date with must-haves solid.
15. **Tell me about customer impact you drove.** → Started from user pain (dashboards timing out), traced to the N+1, fixed it, validated against the user outcome — satisfaction up, the 96% metric followed.
16. **(DSA) Rotting Oranges — why multi-source BFS?** → Seeding all rotten cells at once models simultaneous spread; each BFS level = one minute. Single-source would compute the wrong (sequential) spread.
17. **(DSA) Sliding window — the classic bug in Longest Substring?** → Moving `left` to `lastIndex+1` without `max` clamps — a stale index can drag `left` backwards and corrupt the window length.
18. **(Curveball) Celebrity posts and a normal user share one feed — ordering?** → Merge the pushed (precomputed) feed with the pulled celebrity posts by the same score (timestamp/rank) at read time; the cursor still operates on the merged, ordered result.

---

## ✅ Self-check

1. Did you **open the News Feed with the write/read amplification trade-off** and justify the **celebrity-account hybrid** unprompted?
2. Can you explain **cursor pagination** (and *why* `OFFSET/LIMIT` is unsafe) in two sentences?
3. Can you state the notification reliability spine — **preferences-before-dispatch, dedup via idempotency key, retry + DLQ** — without notes?
4. Is the **90-second "tell me about yourself"** fluent, impact-numbers-first, with no filler?
5. Are **all 10 STAR stories** now written *and* spoken at least once, each a real anecdote ending on a number?

---

<div align="center">

← [01-mon-jul-27.md](01-mon-jul-27.md) · [Week 5](README.md) · [03-wed-jul-29.md](03-wed-jul-29.md) →

</div>
