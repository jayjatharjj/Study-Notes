# Week 5 · Day 6 — Sat Aug 1 — Final Mock / Closeout + Referral Connections + Readiness Check

> The last study day. The mode is **close, not learn** — one final mock or weak-spot closeout aimed squarely at whatever rated lowest all week, re-run until there are **zero open red flags** going into applications. Then the application machine starts turning: **15–20 referral connections** go out (Stage-1 *connect only* — the referral ask waits until Week 8), plus the ATS-keyword tailoring routine you'll use on every application. Finally, the **readiness gate** — a hard checklist that decides whether you apply Monday Aug 3 or carry an item into Week 6. By tonight, study is complete and the switch is loaded.

📌 **Study today:** final mock / weak-spot closeout (zero open red flags) · start 15–20 referral connections (Stage-1 connect only) + ATS keyword routine · readiness check — ready to apply Aug 3 · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — Final Mock / Weak-Spot Closeout

This is not a fresh learning session — it's a **targeted closeout**. Look at the whole week's self-assessments (the timed sets, the mocks, the HLD debriefs) and find the **single lowest-rated thing**. Then spend the block driving it to clean.

### How to run the closeout

1. **Identify the weakest item** from the week — a DP/heap/graph pattern that ran over time, the shakiest HLD, or a STAR story that wobbled under a follow-up. Be honest; the gap you skip here is the gap that surfaces in a live loop.
2. **Re-solve or re-narrate until clean:**
   - *DSA gap* → re-solve the problem cold, timed; if it's a *pattern* (not one problem), do two from that pattern back to back.
   - *HLD gap* → re-narrate the full arc (requirements → capacity → architecture → 2–3 deep-dives → failure modes → trade-offs → observability) from memory, recorded, until it lands under 10 minutes with no filler.
   - *STAR gap* → re-speak it timed, making sure the number lands naturally and it survives the obvious follow-up.
3. **The bar: zero open red flags.** You are not aiming for perfection on everything — you're aiming for *no item that would visibly collapse in front of an interviewer*. Anything still shaky after this block becomes a Week-6 carry-forward, but you do **not** start applying until it's addressed.

### Closeout checklist (pick what's red, not all of it)

| Likely weak spot | Re-rep |
|---|---|
| A DP recurrence you can't re-derive cold | Re-derive from state+transition, no notes; then a second problem in the family |
| Two-heap median rebalance | Code it; narrate the 3-step push→move→rebalance aloud |
| A graph traversal (topo/Dijkstra) | Re-solve; state where the complexity comes from |
| The shakiest HLD | Full cold narration, recorded, < 10 min |
| A STAR story that lost the number | Re-speak timed until the metric is automatic |
| The full-stack integration story | Say the 90-second CORS + JWT + error-contract narrative without notes |

**Goal restated:** walk into Monday with nothing on the board you'd be afraid to be asked.

---

## Block B — Start 15–20 Referral Connections + ATS Keyword Routine

The application engine is **referral-first**: a referred application is ~4–5× more likely to get a callback than a cold one (see [Applications & Referrals](../applications-and-referrals.md)). Today you start the connection pipeline — but **Stage-1 only: connect, don't ask**. The referral *request* comes in Week 8, once a warm thread exists.

### The connection round (15–20 today)

- **Find 1–2 engineers per Tier-1/2 target** from yesterday's list: **SDE2 / Senior / Tech Lead** with **"Java" / "Spring Boot" in their profile**, prioritizing **2nd-degree** connections (a shared connection warms the intro).
- **Send connect requests only, with a one-line genuine note** — a *specific* detail about their work, not "I've always wanted to work here." Example: "Saw your talk on migrating to Spring Cloud Gateway — we did the same edge-CORS consolidation; would love to connect." The specificity is what gets the accept.
- **No referral ask yet.** Stage 1 is purely building the thread. Asking for a referral from a cold connect burns the contact; the ask lands in Week 8 once there's rapport.
- **Log every one:** `Company | Name | LinkedIn URL | Date sent | Response`. An unlogged connect is a dropped lead.

### ATS keyword routine (set the habit now)

Most sub-20-LPA applications are **filtered by an ATS before a human reads them** — the tailoring habit is what gets you past the bot:

- Keep **one base resume** + a **60-second tailoring pass** per application.
- **Paste the JD beside the resume**, identify the **top ~5 JD keywords**, and **mirror the JD's exact phrasing** — "Spring Cloud Gateway," not just "API gateway"; "RESTful microservices," not "backend services." The ATS matches strings, not synonyms.
- Don't keyword-stuff or fabricate — mirror phrasing for skills you genuinely have. The goal is *passing the filter on true matches*, not gaming it.

This routine is what makes Week 8's application volume *convert* instead of vanishing into ATS black holes.

---

## Block C — Readiness Check: Ready to Apply Aug 3

Run the readiness gate below. **Anything not green becomes a Week-6 carry-forward** in the daily review slot — but **do not start applying until the gate is met**. A live loop on an unprepped story or design costs the offer; a few extra days closing a gap does not.

### ✅ Readiness Gate — Ready to Apply Aug 3

You are ready to begin applications when **all** of these are true:

- [ ] **DSA fluency:** solve 2 mixed mediums in 45 min under OA conditions, cold, with complexity stated — demonstrated on at least 3 separate days this week.
- [ ] **Two system-design walk-throughs cold:** any two of URL shortener / file storage / News Feed / notification / chat-typeahead, narrated end-to-end from memory in under 10 minutes each, with failure modes, trade-offs, and a tie to your own work.
- [ ] **10 STAR stories:** all 10 written and spoken, each under 2 minutes with a specific number; the 90-second "tell me about yourself" pitch fluent.
- [ ] **Three project talk-tracks:** Smart360, Deep Fathom, WebX — each under 5 minutes, no notes, curveballs handled.
- [ ] **Profiles live + immediate-joiner headline:** LinkedIn, Naukri, Instahyre, Cutshort, GitHub — all updated, keyword-dense, open-to-work set, resume uploaded, "available to join immediately" stated.
- [ ] **Tiered list built:** ~30–40 companies, three tiers, stacks verified, tracking columns in place.
- [ ] **Referral connections in flight:** 15–20 connect requests sent and logged (Stage-1 connect only).
- [ ] **CTC deflection memorised:** the default deflect + researched band + full-CTC-if-forced line, delivered smoothly.

**If any item is red:** carry it into Week 6's daily review slot, prioritize closing it before sending applications, and do not let it block the *others* from going out once it's green.

### Final framing for the weekend

Sunday Aug 2 is **rest** — no study blocks. Let the consolidation settle. Optionally skim the STAR stories and your two weakest HLDs, but **start no new material**. Monday Aug 3, Week 6 begins and **applications start** — the search shifts from building the engine to running it.

---

## 💻 Practice coding questions

1. Re-solve the single weakest DSA problem of the week, cold and timed — prove the gap is closed.
2. If the gap is a *pattern*, do a second problem from the same family back to back.
3. Two-heap median (LC 295) — code it and narrate the 3-step rebalance, if it was shaky.
4. Any HLD deep-dive you fumbled — re-write its data flow + one failure mode from memory.
5. (Process drill) One final timed clarify → narrate → code → complexity loop on a fresh medium, to confirm the rhythm is automatic.

---

## 🎤 Interview questions

1. **What's the single thing you'd most want another rep on before a loop?** Name it honestly and say how you closed it — self-aware gap-closing is itself a strong signal.
2. **Re-narrate your weakest HLD end-to-end.** Requirements → capacity → architecture → 2–3 deep-dives → failure modes → trade-offs → one observability sentence — under 10 minutes, no notes.
3. **Tell me about yourself.** 90-second pitch: current role → quantified achievements (60s→2–3s, 57% CI/CD, 30+ Bicep, 5-provider LLM proxy) → what you want → bridge to the company. Never a chronological CV recital.
4. **When can you join?** Immediately, no notice period — lead with it; it speeds you up in every queue.
5. **(Recruiter) Walk me through your current CTC expectations.** Deflect to role/fit first; researched band (₹18–24 LPA) if pressed; full CTC (base + variable + PF + benefits) only if a form forces a figure.
6. **Why a referral instead of applying cold?** A referred application is ~4–5× more likely to get a callback; a referred candidate is a known quantity with a vouched-for signal, which moves them past the cold-resume pile.
7. **(Networking) Why are you reaching out to me specifically?** Reference a specific detail about their work — a talk, a repo, a shared technical decision — never a generic "I admire the company." Specificity is what earns the accept and, later, the referral.
8. **How do you make sure your application isn't filtered before a human sees it?** Mirror the JD's top ~5 keywords in the JD's exact phrasing on a per-application tailoring pass — ATS matches strings, not synonyms, and most sub-20-LPA roles filter by ATS first.

---

## ✅ Self-check

1. Is the week's weakest item now **clean** — re-solved or re-narrated until there are **zero open red flags**?
2. Are **15–20 connection requests sent and logged** (Company | Name | URL | Date | Response), with **no referral ask** in them?
3. Have you internalised the **ATS 60-second tailoring routine** (base resume + mirror the JD's top-5 exact phrases)?
4. Does **every readiness-gate item pass** — or is each red item written as an explicit Week-6 carry-forward with a plan to close it before applying?
5. Are you set to **rest Sunday** and **start applications Monday Aug 3** with the engine fully loaded?

---

*Nav: ← [Day 5 (Fri Jul 31)](05-fri-jul-31.md) · [Week 5](README.md) · [Week 6](../week-06/) →*
