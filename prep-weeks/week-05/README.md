# Week 5 — Consolidation & Apply-Prep (Jul 27–Aug 1, 2026)

> The blade is sharp. This week you **consolidate** — DSA becomes maintenance, system designs become fluent cold walk-throughs, your 10 STAR stories and three project deep-dives become muscle memory — and you set up the **application machine**: profiles live, tiered target list built, and the first referral connections going out. This is the bridge from pure study to active applications.
>
> **Full-time week — Mon–Sat (6 days), Sun rest. This is the last study week.** Three blocks per day: **Block A** (DSA maintenance / mock, ~2.5 hr), **Block B** (system-design / LLD mocks + profiles, ~2 hr), **Block C** (behavioral / project deep-dives / apply-prep, ~1.5 hr).

> 📨 **Apps & referrals this week:** build your tiered target list (~30–40 cos) + connect with 15–20 engineers at Tier-1/2 targets (connect only). See the [cadence & tracker](../applications-and-referrals.md).

---

## 🎯 Week Goal

Turn everything from Weeks 0–4 into **interview-ready fluency**. Solve 2 mixed problems in 45 min under OA conditions, on demand, any day. Drive five core HLDs cold — URL shortener, file storage, News Feed, notification, chat/typeahead — with requirements → capacity → architecture → deep-dive → failure modes → trade-offs, each tied to your own work. Write and rehearse all 10 STAR stories (6 core + 4 bar-raiser) and the three project talk-tracks (Smart360, Deep Fathom, WebX) until each lands under 2–5 minutes with a number in every answer. Add the senior full-stack rounds — an **LLD mock** (Design Twitter / parking lot with defensible SOLID), the **full-stack LLM async-job feature**, and the **CORS + JWT + error-contract integration story**. Finish the week with profiles live (LinkedIn, Naukri, Instahyre/Cutshort, GitHub) carrying the **immediate-joiner headline**, a tiered ~30–40-company list, and 15–20 referral connections in flight — **ready to apply Aug 3**.

---

## ✅ By Saturday you can...

- Solve 2 LeetCode mediums in 45 min under OA conditions, back-to-back, with correct complexity stated — on any day, cold.
- Whiteboard from memory, in under 10 minutes each: URL shortener, scalable file storage, News Feed/timeline, notification system, and chat/typeahead — covering requirements, capacity, architecture, 2–3 deep-dives, failure modes, trade-offs, and one observability sentence.
- Deliver any of the 10 STAR stories under 2 minutes with a specific number in every answer, plus the 90-second "tell me about yourself" pitch.
- Narrate the Smart360, Deep Fathom, and WebX talk-tracks in under 5 minutes each, no notes, no filler, and field the standard curveballs (LazyInitializationException, Key Vault outage, crashed worker, multi-tenancy).
- Design classes for an LLD prompt (Design Twitter / parking lot) and **defend the SOLID** — SRP per class, OCP+DIP via a `TimelineMerger` Strategy seam — proving a new joiner can add a ranked feed without editing `FeedService`.
- Design a full-stack LLM async-job feature end-to-end (202 Accepted, durable PostgreSQL job state, SSE vs WebSocket, idempotency, watchdog recovery).
- Explain the full-stack integration story — CORS (preflight + credentials, the `*`+credentials trap), the JWT client flow (memory + HttpOnly cookie, 401→refresh→retry with the recursion guard), and the `@RestControllerAdvice` error contract — as one coherent narrative.
- Show recruiter-ready profiles with the "available to join immediately" headline, a tiered ~30–40-company target list with stacks verified, and 15–20 referral connections sent and logged.

---

## 📅 Daily map

| Day | File | Focus | Headline coverage |
|---|---|---|---|
| **Mon Jul 27** | [01-mon-jul-27.md](01-mon-jul-27.md) | DSA Maintenance · URL Shortener / File Storage Mock · STAR #1–#5 | Timed Mixed Set #1; URL shortener + scalable file storage HLDs; performance/conflict/ownership/failure/why-leaving |
| **Tue Jul 28** | [02-tue-jul-28.md](02-tue-jul-28.md) | DSA Maintenance · News Feed / Notification Mock · STAR #6–#10 | Timed Mixed Set #2; fan-out hybrid + cursor pagination; channel abstraction + dedup + DLQ; bar-raiser stories |
| **Wed Jul 29** | [03-wed-jul-29.md](03-wed-jul-29.md) | DSA Maintenance · Chat / Typeahead Mock · Project Deep-Dives | Timed Mixed Set #3 + LRU/LFU; WebSocket routing + sequence ordering; sharded trie; Smart360/Deep Fathom/WebX talk-tracks |
| **Thu Jul 30** | [04-thu-jul-30.md](04-thu-jul-30.md) | DSA Maintenance · LLD Mock + Full-Stack LLM · CORS/Token | Timed Mixed Set #4; Design Twitter classes + SOLID; LLM async-job (202/SSE/idempotency); CORS + JWT + error contract |
| **Fri Jul 31** | [05-fri-jul-31.md](05-fri-jul-31.md) | Full Mock Interview · Profiles Live · Build the Target List | Timed DSA + system-design mock; LinkedIn/Naukri/Instahyre/Cutshort/GitHub + immediate-joiner headline; tiered ~30–40-co list |
| **Sat Aug 1** | [06-sat-aug-01.md](06-sat-aug-01.md) | Final Mock / Closeout · Referral Connections · Readiness Check | Weak-spot closeout (zero red flags); 15–20 Stage-1 connects + ATS routine; readiness gate — ready to apply Aug 3 |
| **Sun Aug 2** | *(rest)* | No study blocks | Let consolidation settle before applications begin Aug 3 |

---

## ✅ Readiness Gate — Ready to Apply Aug 3

You are ready to begin applications when **all** of these are true:

- [ ] **DSA fluent:** solve 2 mixed mediums in 45 min under OA conditions, cold, with complexity stated — demonstrated on at least 3 separate days this week.
- [ ] **Two system-design walk-throughs cold:** any two of URL shortener / file storage / News Feed / notification / chat-typeahead, narrated end-to-end from memory in under 10 minutes each, with failure modes, trade-offs, and a tie to your own work.
- [ ] **10 STAR stories:** all 10 written and spoken, each under 2 minutes with a specific number; the 90-second pitch fluent.
- [ ] **Profiles live + immediate-joiner headline:** LinkedIn, Naukri, Instahyre, Cutshort, GitHub — all updated, keyword-dense, open-to-work set, resume uploaded, "available to join immediately" stated.
- [ ] **Target list built:** ~30–40 companies, three tiers, stacks verified, tracking columns in place.
- [ ] **15–20 referral connections:** Stage-1 connect requests sent and logged (no referral ask yet).

**If any item is red:** carry it into Week 6's daily review slot, prioritize closing it before sending applications — a live loop on an unprepped story or design costs the offer.

---

*Study complete — next: [Week 6](../week-06/) (Mon Aug 3 – Sun Aug 9), applications begin.*

---

### 📎 Companion (offline)
- [Interview answers](interview-answers.md) — written answers to this week's 🎤 prompts.
- [DSA solutions bank](../dsa-solutions/) — full statements + complete Java solutions for the practice sets.
- [Cheat-sheets](../../cheat-sheets/) · [Projects reference](../../projects-reference.md) · [Interview Q&A bank](../../interview-qa.md)
