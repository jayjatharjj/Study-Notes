# Week 5 · Day 5 — Fri Jul 31 — Full Mock Interview + Profiles Live + Build the Target List

> Today is the dress rehearsal and the launch-pad combined. **Block A is a full timed mock** — run it like a real loop: a DSA round and a system-design round back to back, clarify-narrate-state-complexity, with a written self-debrief at the end. **Block B takes every profile live** — LinkedIn, Naukri, Instahyre/Cutshort, GitHub — all carrying the **"available to join immediately"** headline that speeds you up in every recruiter queue. **Block C builds the tiered target-company list** (~30–40 companies across three tiers) — the single artifact that drives every application and referral from Aug 3 onward. By tonight the interview engine is rehearsed and the application engine is loaded; only the trigger remains.

📌 **Study today:** full mock interview (DSA round + system-design round, timed) · finalize resume + LinkedIn + Naukri + Instahyre/Cutshort + GitHub with the immediate-joiner headline · build the tiered target-company list (~30–40, 3 tiers) — see [Applications & Referrals](../applications-and-referrals.md) · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — Full Mock Interview (timed)

Treat this as a **real loop**, not a practice set. The point is to feel the time pressure, the context-switch between rounds, and the "talk while you think" load all at once — the things that trip people in a live loop but never show up in untimed solo practice. Use a partner (**Pramp / interviewing.io**) if you can; a structured self-mock with a recorder works if not.

### Round 1 — DSA (45 min)

Two problems, OA conditions. Whether a partner picks them or you draw blind, the **process** is the graded thing:

1. **Clarify before coding** — restate the problem, ask about input size, duplicates, empty/null, expected return on no-solution. (This is ~30 seconds and it's a senior signal — juniors start typing immediately.)
2. **Narrate the approach before writing** — state the pattern, the data structure, and the complexity you *expect* before a line of code. If two approaches exist, name both and say why you're picking one.
3. **Code while talking** — keep a running commentary; silence reads as "stuck."
4. **State final complexity before submitting** — time and space, best and worst if they differ, and a sentence on how you'd test it.

If a partner isn't available, set a 45-min timer on two blind picks from the maintenance rotation (one DP/greedy, one heap/graph) and **record yourself narrating** — playback exposes filler words and silent stretches.

### Round 2 — System Design (45 min)

Pick **one** of the HLDs you've drilled — **URL shortener / file storage / notification** are the cleanest 45-min fits — and drive it through the full arc without prompting:

1. **Requirements** (functional + non-functional; read/write ratio; consistency needs).
2. **Capacity estimation** (DAU → QPS, storage growth, the one number that sizes the design).
3. **High-level architecture** (the boxes and the data flow).
4. **2–3 deep-dives** (the parts that actually matter — e.g. for file storage: chunked pre-signed upload, `owner_id` sharding, the read-path CDN + cache).
5. **Failure modes** (what breaks, how you detect and recover).
6. **Trade-offs** (what you chose *against* and why — naming the rejected option is the senior move).
7. **One sentence on observability** to close: golden signals (latency / traffic / errors / saturation), trace propagation across services, and SLO burn-rate alerting. Most candidates forget this; volunteering it lands the round.

**Tie every choice to your own work** as you go — "this is the same pre-signed-URL pattern from Smart360, extended to per-chunk," "Front Door via Bicep in Deep Fathom did exactly this CDN offload." A design grounded in shipped work outranks a textbook one.

### Self-debrief (mandatory)

Write **3 things you did well + 3 to improve**. Be specific: not "system design was OK" but "forgot capacity estimation until prompted" or "buried the fan-out trade-off instead of leading with it." These three improvement items feed straight into tomorrow's weak-spot closeout. A mock you don't debrief is half a mock.

---

## Block B — Profiles Live (LinkedIn · Naukri · Instahyre/Cutshort · GitHub)

Today every profile goes live and recruiter-ready, all carrying the same spine: **quantified impact first, keyword-dense, and the "available to join immediately" flag** that gives a recruiter a concrete reason to prioritize you. (See the [Applications & Referrals](../applications-and-referrals.md) note on surfacing the immediate-joiner advantage everywhere.)

### LinkedIn

- **Headline (keyword-dense + the flag):**
  `Full Stack Engineer · Java 17 · Spring Boot · Microservices · Azure · Kubernetes · LLM Integration · Available to join immediately`
  Recruiters search by keyword — every term in your headline is a query you now match.
- **About (3–4 sentences):** lead with **quantified impact** (60s→2–3s / 96%, 57% CI/CD cut, 30+ Bicep resources, 5-provider LLM proxy), then the stack, then what you want. Numbers in the first sentence — not the third.
- **Experience bullets:** every bullet starts with an **action verb** and carries a **number**. "Reduced dashboard load 60s→2–3s by eliminating N+1 queries (`JOIN FETCH` + composite indexes + Redis cache)" — not "worked on performance."
- **Skills section:** keyword-dense and ordered by relevance — it feeds both recruiter search and LinkedIn's own matching.
- **Open to work:** set it (recruiters-only visibility if you're discreet about your current employer).

### Naukri

- **Mirror the LinkedIn headline/summary** so a recruiter cross-checking sees one consistent story.
- Upload a **clean 1–2 page PDF resume** (header carries "Available to join immediately").
- Set status **"Actively Looking"** + **immediate joiner**; list skills with proficiency.
- Set an **expected-CTC band with a sensible floor** (a **20 LPA floor** avoids being filtered out by under-banded roles) — but keep the deflection line ready for the human conversation (below).

### Instahyre + Cutshort

- Create both profiles, set **open-to-work**, fill skills + expected CTC, upload the resume. These are **high-signal inbound channels** for your band — they fill the pipeline *passively* while you spend active effort on referrals. Low cost, asymmetric upside.

### GitHub

- **Pin 3–6 best repos.** Write a **profile README** narrating the cloud-native + LLM story.
- **Prune** anything embarrassing or abandoned — a recruiter who clicks through should see signal, not coursework.
- Make **one showcase repo** genuinely clean: a clear README, an **architecture diagram**, and a one-command `docker-compose up` so a reviewer can run it in 30 seconds.
- Use **imperative, scoped commit messages** going forward ("Add JWT refresh interceptor," not "stuff").

### First-screen CTC deflection (memorise now — you'll use it within days)

- **Default deflect:** "I'd rather focus on the role and the total opportunity first, then talk numbers once we both see a fit."
- **If pressed:** give a **researched band** (₹18–24 LPA), never your current number.
- **If a form forces a figure:** enter your **full CTC** (base + variable + employer PF + benefits), not just base — anchor high and accurate.

---

## Block C — Build the Tiered Target-Company List (~30–40, 3 tiers)

This is the **input artifact for everything downstream** — Week 6's referral-contact mapping and Aug 3's first applications both read from this list. Build it once, build it well. (It pairs with the cadence in [Applications & Referrals](../applications-and-referrals.md), which runs a target track *and* a safety-net track in parallel.)

### The structure — three tiers, ~30–40 companies

| Tier | What it is | Examples (verify the stack on each) |
|---|---|---|
| **Tier-1 — dream (GCC / top product)** | the roles worth a tailored, referred application | Goldman Sachs · Morgan Stanley · JPMC · Walmart Global Tech · Atlassian · Salesforce · Adobe · Visa · Mastercard · Razorpay · PhonePe · CRED |
| **Tier-2 — strong product mid-tier** | high-quality, faster-moving product companies | Groww · Juspay · Navi · Slice · Postman · BrowserStack · Druva · Meesho · Zepto · Setu · Darwinbox |
| **Tier-3 — calibration / warm-ups** | established service + mid-size product cos for interview reps and the safety-net track | (established services + regional product companies; lower stakes, real practice loops) |

**Calibration for your profile** across all tiers: **Java / Spring Boot in the JD**, **2–5 YOE**, **₹18–30 LPA**, **Pune / Bengaluru / remote**.

### The tracking columns

Build it as a table (spreadsheet or markdown) with exactly these columns — anything less and a lead goes cold:

| Company | Tier | Role | JD URL | Tech-stack check | Referral contact(s) | Status |
|---|---|---|---|---|---|---|

- **Tech-stack check** is non-negotiable: **verify each JD is actually Java/Spring** — do not waste an application (or a referral ask) on a **.NET / Node / Go-primary shop**. One column, one minute per company, saves dozens of dead applications.
- **Referral contact(s)** starts empty — Saturday's connect round and Week 6's mapping fill it.
- **Status** starts as `Researched`; it'll move through `Connected → Applied → Screen → ...` as the search runs.

### Why tiered (and not one flat list)

The tiers encode **strategy, not snobbery**: Tier-1 gets your most tailored, referred, patient effort; Tier-3 gives you **live interview reps** to calibrate against *and* protects cash flow as a safety net while the Tier-1 loops mature. Running all three **in parallel from day one** — never Tier-1 first and Tier-3 "if needed" — is the deliberate floor that makes *being employed* the near-certain outcome. The list is where that two-track strategy becomes concrete.

---

## 💻 Practice coding questions

1. The two mock DSA problems — re-solve any that ran over time, narrating aloud this time.
2. Re-derive the complexity of both mock problems cold (best/worst), as if asked the follow-up.
3. For the system-design round you ran, write the **capacity estimation** from scratch (DAU → QPS → storage) in under 5 min.
4. List the 2–3 deep-dives you chose and one you *didn't* — and why you'd reach for it under a follow-up.
5. Write the one-sentence observability close (golden signals + trace propagation + SLO burn-rate) from memory.
6. (Process drill) Time yourself doing the full clarify → narrate → code → complexity loop on one fresh medium — under 25 min, narration included.

---

## 🎤 Interview questions

1. **Walk me through your approach before you code.** Restate the problem, confirm constraints (size, duplicates, null/empty, no-solution return), name the pattern and expected complexity, then code — clarifying first is the senior signal.
2. **Design a URL shortener.** Capacity → base-62 key length; counter+base-62 vs hash+collision-check; redirect via Redis cache-aside; analytics as an async event; 301 (cacheable, loses tracking) vs 302 (every hit reaches you).
3. **File storage — start with the upload flow.** Chunked pre-signed S3 URLs; data never flows through your servers; client → Upload Service → direct-to-S3 → complete → metadata `READY`. "Smart360's single-file pre-signed pattern, extended to per-chunk."
4. **Notification — how do you guarantee no duplicates?** Idempotency key (event_id + user_id + channel) via `SETNX` or a unique constraint; duplicate notifications are the #1 user complaint.
5. **What's your one observability sentence for any design?** Golden signals (latency/traffic/errors/saturation), trace context propagated across services, and SLO burn-rate alerting so you page on user-facing degradation, not raw CPU.
6. **Why should we believe your impact numbers?** Each ties to a mechanism: 60s→2–3s came from eliminating 47 queries/load (N+1) via `JOIN FETCH` + composite indexes + Redis; 57% CI/CD from a GitLab DAG + BuildKit. The mechanism is the proof.
7. **(Recruiter) What's your current CTC / expectation?** Deflect to role-and-fit first; if pressed, a researched band (₹18–24 LPA), never the current number; if a form forces it, full CTC including variable + PF + benefits.
8. **(Recruiter) When can you join?** Immediately — no notice period. Lead with it; it's a concrete reason to move you up the queue.
9. **Why these target companies?** Java/Spring stack match (verified per JD), 2–5 YOE band, comp aligned to the level, and a tiered strategy — dream GCC/product, strong mid-tier, and calibration loops run in parallel.
10. **(Self-debrief framing) What would you do differently in that round?** Name a specific, fixable thing — "I buried the fan-out trade-off; next time I lead with it" — not a vague "be better." Self-awareness with a concrete fix is the signal.

---

## ✅ Self-check

1. Did the full mock run **timed**, both rounds, with a written 3-up / 3-to-improve debrief?
2. Is your **LinkedIn headline keyword-dense with the immediate-joiner flag**, and does **every experience bullet carry a number**?
3. Are LinkedIn, Naukri, Instahyre, Cutshort, and GitHub **all live** with the resume uploaded and open-to-work set?
4. Is the **tiered target list ~30–40 companies**, three tiers, with **stacks verified** (no .NET/Node-primary shops) and the tracking columns in place?
5. Can you deliver the **CTC-deflection line** smoothly, and state your **immediate-joiner** availability without hedging?

---

*Nav: ← [Day 4 (Thu Jul 30)](04-thu-jul-30.md) · [Week 5](README.md) · [Day 6 (Sat Aug 1)](06-sat-aug-01.md) →*
