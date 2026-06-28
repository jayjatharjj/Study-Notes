# Week 8 · Day 2 — Tue Aug 18 — Loop Execution + Deep Fathom Deep Dive + STAR #2/#3

📌 **Today:** Match the project to the company — an infra/DevOps round means lead with Deep Fathom. Rehearse the deep-dive before the round, execute scheduled loop(s), and land STAR #2 (conflict) and #3 (ownership), each under 2 minutes with a number.

**DSA maintenance (~30 min):** Light — loops take priority.
- [ ] **LC 102 – Binary Tree Level Order Traversal** — BFS, under 10 min. Warm-up only.

**Primary activity:**

**Pre-day check (5 min):**
- [ ] Check the schedule; run the **pre-interview checklist** (see Day 1) for any round today/tomorrow.

**Project deep-dive rehearsal (40 min) — Deep Fathom:**
- [ ] Rehearse the 5-minute Deep Fathom talk-track aloud:
  - 30+ Azure resources in modular **Bicep** (Container Apps, PostgreSQL Flexible Server, Redis, Key Vault, Front Door, private endpoints).
  - **CI/CD redesign:** sequential → DAG with `needs:`, BuildKit `--cache-from` → **23 min to 10 min, 57% cut**.
  - K8s probes + graceful shutdown; Key Vault via managed identity; PostgreSQL RLS across 50+ tables; FedRAMP posture.
- [ ] Curveball prep:
  - **Key Vault outage at startup** — deploy fails safe; runtime secrets cached in-memory, running pods unaffected.
  - **Bicep vs Terraform** — Bicep = Azure-native, no state file; Terraform if multi-cloud.

**Loop execution + STAR #2/#3 (rest of day):**
- [ ] **Execute scheduled loop(s).** Lead with impact numbers; ask a researched question; surface "available to join immediately" when start dates come up.
- [ ] **STAR #2 — Conflict/disagreement:** a real disagreement, your counter-evidence (a comparison doc / time-boxed POC), outcome, relationship intact. <2 min.
- [ ] **STAR #3 — Ownership/leadership:** you fixed something nobody owned (e.g., the slow CI/CD pipeline) — assessed scope, proposed a plan, executed, measured (**23→10 min**). <2 min.
- [ ] **Decompress between rounds:** 15 min away from the screen before the next round.

**Pipeline:** Loops first. Respond to recruiters within hours; confirm receipts. Log every round outcome same day. Hold new applications until prep is clear.

**EOD check:**
- [ ] Does the Deep Fathom story land in 4–5 min with the **57% cut** tied to DAG + BuildKit specifically?
- [ ] Are the conflict and ownership stories backed by a specific number/outcome?
- [ ] Every loop today debriefed in the tracker?
- [ ] DSA warm-up done?

---
*Nav: ← [Day 1 (Mon Aug 17)](01-mon-aug-17.md) · [Week 8](README.md) · [Day 3 (Wed Aug 19)](03-wed-aug-19.md) →*
