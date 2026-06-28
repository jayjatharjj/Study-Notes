# Week 9 · Day 5 — Fri Aug 28 — System Design + Java Deep Execution Mock

📌 **Today:** Two execution mocks under time pressure. A 45-minute system-design mock following the sequence religiously, then a Java deep-dive answered out loud — Spring internals, transactions, AOP. Watch *how* you deliver, not just accuracy.

**DSA maintenance (~30 min):**
- [ ] Timed solve: **LC 347 (Top K Frequent)** or **LC 215 (Kth Largest)** — high-frequency across all tiers. Clean solve with heap/quickselect narration in ≤ 20 min.

**Primary activity:**

**System design execution mock (60 min):**
- [ ] Pick one: "Design a distributed job queue for long-running LLM tasks" (Deep Fathom) or "Design a multi-tenant authorization service" (Smart360).
- [ ] Time-box hard — 45 min live, 15 min debrief. Follow the sequence:
  1. **Clarify (5 min):** scale, latency, consistency vs availability, client type.
  2. **High-level design (10 min):** boxes and arrows; name components; don't over-engineer yet.
  3. **Deep-dive one component (15 min):** the hardest (job dispatcher + LLM router) — data model, API contract, failure handling.
  4. **Scale & failure (10 min):** where does it break at 10×? Bottleneck? What do you add (read replicas, sharding, circuit breakers)?
  5. **Trade-offs (5 min):** what you did NOT do and why; what you'd change with more time.
- [ ] Score yourself honestly per phase. Note what you skimped.

**Java deep-dive execution (30 min):**
- [ ] Take 5 questions from interview-qa.md — Spring internals, transactions, AOP — answer **out loud without reading**, then check against notes.
- [ ] Rehearse the hardest: `@Transactional` self-invocation, bean lifecycle / BeanPostProcessor timing, circuit-breaker state transitions.
- [ ] Watch delivery: confident? top-down (conclusion first, then detail)? did you give a project example?

**Pipeline:** Mid-loop check — any company silent 5+ business days after a round gets a polite nudge today. If a near-offer is live, start compression nudges to preferred-but-slower companies.

**EOD check:**
- [ ] Could you run the full system-design sequence under time pressure without a checklist?
- [ ] Did you tie at least one design decision back to Smart360 or Deep Fathom?
- [ ] DSA heap/quickselect solve done?

---
*Nav: ← [Day 4 (Thu Aug 27)](04-thu-aug-27.md) · [Week 9](README.md) · [Day 6 (Sat Aug 29)](06-sat-aug-29.md) →*
