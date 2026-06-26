# Week 7 — Apply Wave 2 + First Rounds (Aug 10–16, 2026)

> The pipeline you built last week starts firing back. First OA invites and phone screens arrive while a second wave of applications goes out — so this week runs two tracks at once: keep the volume up *and* execute the first live rounds cleanly. DSA stays in maintenance (1–2 problems daily, now leaning toward company-tagged patterns). The new work is per-company tailoring, system-design-under-time pressure, behavioral warm-ups, and disciplined follow-up on every Week 6 application that's gone quiet. A first round you walk into researched and calm is worth more than ten cold applications you fired off in a hurry.

> 📨 **Apps & referrals this week:** 15–20 more applications + follow up all Week 6 apps + ~10 referral asks. See the [cadence & tracker](applications-and-referrals.md).

---

## 🎯 Week Goal

1. Submit 15–20 more applications (Wave 2), every one ATS-tailored and per-company researched, and send ~10 more referral asks.
2. Follow up — once — on every Week 6 application now past the 5-day quiet mark, with a 3-sentence personalised nudge.
3. Execute your first OA / phone screens researched, calm, and with sharp questions ready — and surface immediate availability when timelines come up.
4. Run one system-design-under-time mock (45 min, prompt cold) so the first real SD round doesn't catch you flat-footed.
5. Warm up the core behavioral answers — "tell me about yourself," the performance win, "why are you leaving," "why this company" — so a recruiter screen lands clean.

---

## ✅ By Sunday you can...

- Walk into a phone screen having researched the company's product, stack, and one recent engineering detail — with 3 questions ready.
- Deliver a 90-second "tell me about yourself" and a 2-minute performance-win story without notes.
- Solve a company-tagged Medium in ≤ 20 min in a shared editor under observation.
- Drive a system-design prompt end-to-end in 45 minutes: requirements → capacity → high-level → 2 deep dives → failure modes → trade-offs.
- Send a clean 3-sentence follow-up that references the role, adds one specific reason, and offers a quick chat.
- State "available to join immediately" as a closing advantage in any screen, naturally.
- Read your funnel after ~35 applications and know whether the callback rate is healthy (≥ 10%).

---

## 📅 Daily Checklist (Mon–Sun)

---

### Monday Aug 10 — Wave 2 Batch + Follow-Up Discipline

📌 **Study today:** DSA maintenance (LC 3, 167) · 4–5 applications (Wave 2) · follow up all quiet Week 6 apps

**Pre-day check (5 min):**
- [ ] Open the tracker. Any OA/screen invites land over the weekend? Schedule them immediately and block prep time the evening before each.

**DSA Maintenance (30 min):**
- [ ] **LC 3 – Longest Substring Without Repeating Characters** (Medium) — sliding window + HashSet. Under 15 min. Appears in nearly every product-company warm-up.
- [ ] **LC 167 – Two Sum II (Sorted)** (Medium) — two-pointer. Under 10 min. State why two-pointer beats HashMap when the array is sorted.

**Applications Block (60 min) — Wave 2 batch:**
- [ ] **Apply to 4–5** through the ATS routine (paste JD → mirror top ~5 keywords → 3-sentence note ending in immediate availability → log). Lean toward GCC/product this week — the safety net is seeded; now chase the real targets.

**Follow-Up Block (45 min) — chase the quiet Week 6 apps:**
- [ ] Pull every Week 6 application now **5+ days quiet.** Follow up *once* each:
  > *"Hi [Name], I applied for [Role] on [Date] and wanted to confirm receipt and reiterate my interest — my background in cloud-native Java/Spring microservices (with a 96% latency win and LLM integration) maps closely to the role, and I'm available to join immediately. Happy to answer any initial questions."*
- [ ] Log each follow-up date. The rule is *once* — set it, don't nag.

**Self-Check**
- [ ] Did every quiet Week 6 application get exactly one follow-up?
- [ ] Are 4–5 new Wave 2 applications logged?

---

### Tuesday Aug 11 — First OA / Phone Screen + Per-Company Tailoring

📌 **Study today:** company-tagged DSA (LC 209, 33) · per-company prep routine · first OA/screen execution + 3–4 applications

**Pre-day check (5 min):**
- [ ] Any OA or screen today/tomorrow? Run the per-company prep routine below *now*, before anything else.

**DSA Maintenance (30 min) — company-tagged:**
- [ ] **LC 209 – Minimum Size Subarray Sum** (Medium) — sliding window, two pointers. Microsoft/Atlassian screens. Under 15 min.
- [ ] **LC 33 – Search in Rotated Sorted Array** (Medium) — modified binary search. Goldman/JPMC favourite. Under 20 min.

**Core Block (40 min) — Per-Company Tailoring Routine:**

For each company with a live round or strong-signal recruiter call:
- [ ] Re-read the JD. Highlight 3 tech keywords; map each to a specific resume story (Smart360 perf, Deep Fathom infra, WebX async/LLM).
- [ ] Research: product, recent news (blog/press/LinkedIn), tech stack (engineering blog + JD clues).
- [ ] Write 3 questions to ask — at least one specific ("I read your 2024 post on migrating to microservices — what were the biggest operational surprises?").
- [ ] Check Glassdoor/Blind for recent interview reports; note recurring question types.
- [ ] Log all of it in the tracker — don't rely on memory.

**OA / Screen Execution + Applications (75 min):**
- [ ] **Execute any OA/screen** with the per-company prep done. In a screen, lead with 2 impact numbers, ask 1 sharp question at the end, and **mention immediate availability when timelines come up** — it's a real lever for teams with an urgent backfill.
- [ ] **Apply to 3–4 more** (Wave 2 continues). Running total across Weeks 6–7 should be climbing past 25.

**Self-Check**
- [ ] Did you complete the full per-company prep before the round (not after)?
- [ ] Did you surface immediate availability and ask a researched question?

---

### Wednesday Aug 12 — System-Design-Under-Time Mock + Referral Asks

📌 **Study today:** DSA maintenance (LC 1046, 215) · 45-min SD mock (file storage or notification system) · 5 referral asks

**DSA Maintenance (30 min):**
- [ ] **LC 1046 – Last Stone Weight** (Easy) — max-heap warm-up. Under 10 min.
- [ ] **LC 215 – Kth Largest Element** (Medium) — min-heap of size k (O(n log k)); mention QuickSelect as the "can you do better?" follow-up. Under 20 min.

**Core Block (45 min) — System Design Under Time (45-min mock):**

Pick a prompt cold, set a 45-min timer, whiteboard on paper. Suggested: **"Design a scalable file storage service"** or **"Design a notification system for 10M notifications/day."** Drive it end-to-end — do not wait to be prompted:
- [ ] **Requirements + capacity (8 min):** clarify file sizes / volume, read:write ratio, consistency needs, global vs regional. Give a rough capacity estimate (e.g., 10M users × 5 GB = 50 TB; 100K uploads/day × 10 MB = 1 TB/day).
- [ ] **High-level design (10 min):** for file storage — chunked upload via **pre-signed URLs** (data never flows through your servers), metadata service on PostgreSQL, S3/Blob for content, CDN on the read path. Name the pre-signed-URL caching pattern from Smart360.
- [ ] **Two deep dives (12 min):** metadata sharding (hash on `owner_id`), leader-follower replication + lag handling, or the notification channel abstraction + dedup/idempotency + retry/DLQ.
- [ ] **Failure modes (10 min):** orphaned chunks (S3 lifecycle cleanup), DB leader failover (CP trade-off), CDN stale-after-delete (invalidation), provider 5xx → exponential-backoff retry → DLQ.
- [ ] **Trade-offs + observability (5 min):** name CAP/PACELC where relevant, and add one sentence on how you'd *operate* it (golden signals, trace propagation). Tie at least one decision to your real work.

**Referral Outreach (30 min):**
- [ ] Send 5 referral asks (toward the ~10 weekly target), prioritising companies where you've applied. Same template, personalised, logged.

**Self-Check**
- [ ] Did the SD mock cover failure modes and a capacity estimate, not just the happy path?
- [ ] Are 5 referral asks logged?

---

### Thursday Aug 13 — Behavioral Warm-Ups + Application Batch

📌 **Study today:** DSA maintenance (LC 102, 199) · behavioral warm-ups (TMAY, perf win, why leaving, why company) · 4–5 applications + 5 referral asks

**DSA Maintenance (30 min):**
- [ ] **LC 102 – Binary Tree Level Order Traversal** (Medium) — BFS queue. Under 10 min — a confidence-builder often asked as a warm-up.
- [ ] **LC 199 – Binary Tree Right Side View** (Medium) — BFS, last node per level. Under 12 min.

**Core Block (45 min) — Behavioral Warm-Ups (say each aloud, timed):**
- [ ] **"Tell me about yourself" (90s):** current role → biggest outcomes (60s→2–3s, 57% CI/CD cut, 5-provider LLM proxy) → stack → what you want → bridge to this company. Lead with outcomes, not a chronological CV.
- [ ] **Performance-win story (2 min):** Smart360 60s→2–3s. Diagnosis (Hibernate statistics → 47 queries → N+1) → fix per layer (JOIN FETCH, EntityGraph, composite indexes, Redis cache, S3-URL caching) → result (96% latency cut, 80% fewer S3 calls). Use "I" for what you personally did.
- [ ] **"Why are you leaving?" (60–75s):** ambition framing — scale, engineering culture, growth, comp. Never negative about the current employer. **Naturally include availability:** "...and I'm in a position to move immediately."
- [ ] **"Why this company?" (60s):** research-specific, one real detail per company. Generic answers ("great culture") get filtered.

**Applications + Referrals Block (90 min):**
- [ ] **Apply to 4–5 more** — push toward the **15–20 Wave 2 target.**
- [ ] **Send 5 more referral asks** — close out the ~10 weekly target.
- [ ] Triage inbound across all channels; respond to recruiters within hours.

**Self-Check**
- [ ] Can you deliver TMAY and the performance-win story without notes, under time?
- [ ] Is the Wave 2 application count at 12–15?

---

### Friday Aug 14 — First-Round Debrief + Company-Tagged DSA

📌 **Study today:** company-tagged DSA (LC 56, 76) · debrief any rounds done so far · 3–4 applications + inbound triage

**DSA Maintenance (30 min) — company-tagged:**
- [ ] **LC 56 – Merge Intervals** (Medium) — sort by start, merge. Atlassian-tagged. Under 12 min.
- [ ] **LC 76 – Minimum Window Substring** (Hard) — sliding window with need-count + formed counter. Walmart/Microsoft-tagged. Under 25 min. The canonical hard sliding window.

**Core Block (40 min) — Debrief Any Rounds Done:**
- [ ] For every OA/screen this week: write down every question asked (while fresh), what went well, what you'd change. Log in the tracker under a "round notes" column.
- [ ] If a coding round felt slow on a pattern, add it to next week's maintenance list. If a behavioral answer rambled, mark which sentence to cut.
- [ ] Send a thank-you note within 4 hours of any round that allows it.

**Applications Block (60 min):**
- [ ] **Apply to 3–4 more** — finish pushing toward the 15–20 Wave 2 target. Prioritise GCC/product roles still untouched.
- [ ] **Inbound triage:** check all channels, respond fast, bump Naukri "last active."

**Self-Check**
- [ ] Are this week's round questions logged while still fresh?
- [ ] Are you at 15+ Wave 2 applications heading into the weekend?

---

### Saturday Aug 15 — Full Mock Day (DSA + System Design) + Quota Close

📌 **Study today:** timed DSA mock (2 problems) · full SD mock with debrief · close the weekly application + referral quota

**DSA Mock (60 min):**
- [ ] Contest mode: 2 problems you haven't done this week, 30 min each, no hints. Recommended from company tags: **LC 207 – Course Schedule** (topo/cycle) + **LC 146 – LRU Cache** (design — doubly-linked list + HashMap, asked verbatim more than any other design problem).
- [ ] Score yourself: clean finish? Clear narration? Complexity stated unprompted?

**System Design Mock (60 min):**
- [ ] Pick a second prompt (rotate from Wednesday) — e.g., "Design a URL shortener" or "Design a real-time notification system." Drive it in 45 min with the same structure, then spend 15 min self-critiquing: requirements clarified? capacity estimated? failure modes named? one decision tied to your real work? observability mentioned?

**Applications + Referrals (2 hr) — close the quota:**
- [ ] **Close out 15–20 Wave 2 applications** total for the week. Tally and finish if short.
- [ ] **Close out ~10 referral asks** and confirm all Week 6 follow-ups are sent.
- [ ] Tracker quality pass: every active row has a next action and a current stage.
- [ ] Research 2–3 companies for next week — interviews are about to cluster, so quality of prep matters more than raw new volume now.

**Self-Check**
- [ ] Is the weekly application count at 15–20 and referrals at ~10?
- [ ] Did you complete both a DSA and a system-design mock?

---

### Sunday Aug 16 — Quota Check + Funnel Read + Week 8 Plan (lighter day)

📌 **Study today:** light DSA review · weekly quota check + funnel read (≥10% callback?) · Week 8 plan

**Light DSA (30 min):**
- [ ] Re-solve the week's weakest problem from scratch, timed. Update your complexity review card.

**Weekly Quota Check + Funnel Read (45 min):**
- [ ] Fill the **Tracker 1 weekly quota check.** Tally cumulative applications (Weeks 6–7 should be ~30–40), referral asks, and now — **callbacks.**
- [ ] **Read the funnel for real this week:** after ~35 applications, is your callback rate **≥ 10%**? If not, the problem is upstream — resume/ATS keywords, positioning, or too few referrals. Fix that *before* sending more volume. Referrals convert ~4–5× cold; if your referred-vs-cold ratio is low, that's the lever.
- [ ] Confirm every round this week is debriefed and every quiet application has had its one follow-up.

**Week 8 Plan (30 min):**
- [ ] Week 8 is **interview loops cluster** — scheduled rounds take priority over new volume. Map every confirmed loop onto the calendar and block project-deep-dive rehearsal time before each.
- [ ] Identify the top 2 gaps from this week's rounds → Day 1–2 makeup in Week 8.
- [ ] Write 2–3 things you can do now that you couldn't last Sunday (drove an SD prompt under time; ran a live screen; followed up with discipline).

---

## 🧠 This Week's Operating Principles

### Two tracks at once: volume *and* execution
Week 7 is the overlap week. You're still building the pipeline (Wave 2 + follow-ups + referrals) while the first loops fire. The discipline: scheduled rounds get *prep priority*, but the application engine doesn't stop — a thin pipeline now means a dry Week 9. Block the calendar so neither starves the other.

### Per-company tailoring beats spray-and-pray
At this stage a researched application + a referral beats five cold applies. For every target-tier company: JD keywords mapped to stories, one specific company detail, 3 questions ready. The candidate who references the company's actual engineering blog is instantly different from the 20 who said "I love your culture."

### Follow-up is a system, not a vibe
Every application 5+ days quiet gets exactly one 3-sentence nudge — role + one specific reason + immediate availability + offer to chat. Logged. Once. An application with no logged next action is a dropped lead.

### Surface immediate availability at every timeline moment
Zero notice is a real lever now that timelines are being discussed. Drop it in screens when scheduling comes up, in follow-ups, and in the "why leaving" answer. For a team with an urgent backfill it can be the tiebreaker.

### System design under time — the structure that never changes
Requirements + capacity → high-level → 2 deep dives → failure modes → trade-offs + observability. Same skeleton every time; only the prompt changes. Drive it; don't wait to be pulled through it.

---

## 📊 End-of-Week Self-Assessment

Fill Sunday. Anything below target is Week 8 Day 1.

| Area | Target | Your Score | Notes |
|---|---|---|---|
| Wave 2 applications submitted (15–20) | 5 | | |
| All quiet Week 6 apps followed up (once) | 5 | | |
| Referral asks sent (~10) | 4 | | |
| First OA/screens executed with per-company prep | 5 | | |
| System-design-under-time mock completed | 4 | | |
| Behavioral warm-ups (TMAY, perf win, why leaving, why co) | 4 | | |
| Company-tagged DSA in ≤ 20 min | 4 | | |
| DSA maintenance streak (1–2/day) | 4 | | |
| Immediate availability surfaced in rounds/follow-ups | 5 | | |
| Every round debriefed + logged | 5 | | |
| Funnel read done (callback rate ≥ 10%?) | 4 | | |

**Week 7 exit criteria (minimum bar before Week 8):**
- 15–20 Wave 2 applications submitted and logged; cumulative ~30–40.
- Every Week 6 application past 5 days quiet followed up once.
- At least one system-design-under-time mock completed with debrief.
- Core behavioral answers (TMAY + performance win) deliverable without notes.
- Every live round this week debriefed in the tracker.

**If you missed the bar:** carry it into Week 8 Day 1 — but remember Week 8 prioritises *scheduled loops over new volume*. A missed follow-up or an unprepped round costs more than a delayed application.

---

*Week 7 of the execution phase — next: Week 8 (Mon Aug 17 – Sun Aug 23, 2026): Interview Loops Cluster.*
