# Week 9 — Loops + Early Offers (Aug 24–30, 2026)

> You are no longer preparing for interviews — you are IN them. The work now is execution, not accumulation. Every round is a performance you've rehearsed. Light DSA maintenance keeps the edge sharp; everything else is tailored, deliberate, and followed up within the same day. First offers may start landing this week — and an offer in hand changes every conversation after it.

> 📨 **Apps & referrals this week:** Taper new applications — focus live loops + pipeline compression nudges. See the [cadence & tracker](applications-and-referrals.md).

---

## 🎯 Week Goal

1. Execute every live interview round with structured thinking-out-loud, clean narration of trade-offs, and zero freezing — even on questions you don't fully know the answer to.
2. Keep the pipeline sheet live and current: every company, every stage, every follow-up logged within 24 hours of each interaction.
3. Send a personalised thank-you / follow-up note within 12 hours of every round, and a written debrief the same day.
4. Tailor your prep the day before every round — re-read the company tech blog, re-read your project deep-dive, find one specific thing about their product to mention.
5. If a first offer lands: do NOT accept on the spot. Buy 3–5 days, and use it to compress every other live loop. (Full negotiation playbook is Week 10–11.)

---

## ✅ By Sunday you can...

- Execute a full 45-minute technical round — DSA + system design or a deep Java/Spring dive — without freezing, narrating your thinking audibly throughout.
- Handle any of these gracefully: "I don't know exactly, but here's how I'd reason through it"; "let me make sure I understand the problem before I code"; "here's a simpler solution, and here's the optimised version with the trade-off."
- Name the tech stack, recent product news, and one meaningful observation about each company currently in your active loop.
- Recite your 90-second differentiator pitch cold, in any sequence, without sounding rehearsed.
- Ask 2–3 genuinely good, tailored questions of every interviewer — specific to their product, team, or tech choices, not boilerplate.
- Do a written post-interview debrief in 10 minutes: what went well, what to fix, what they cared most about.
- Respond to a first offer correctly — buy time, don't accept on the call — and use it to nudge the rest of the pipeline.

---

## 📅 Daily Checklist (Mon–Sun)

---

### Monday, Aug 24 — Loop Entry: Tailoring + Light DSA

📌 **Study today:** Light DSA warm-up (LC 79, 200, or 33) · per-company tailoring (JD → project story mapping) · pipeline sheet hygiene

**DSA Maintenance (15–20 min)**
- [ ] Solve one medium you haven't touched in 2+ weeks — pick from LC 79 (Word Search), LC 200 (Number of Islands), or LC 33 (Search in Rotated Sorted Array). Time yourself. The goal is to stay warm, not learn new material.
- [ ] If you solved it cleanly in ≤ 20 min: done. If not, identify the single gap (tree traversal? binary search edge case?) and note it — do not rabbit-hole.

**Company Tailoring Block (45 min per active company)**
- [ ] Pull up the JD for each active loop. For each role:
  - [ ] Identify the 3 most-emphasised technical areas in the JD (e.g., "microservices", "real-time data pipelines", "distributed systems").
  - [ ] Map each to a specific project story you can tell: Smart360 (performance, security, AWS) vs Deep Fathom (Azure IaC, K8s, LLM async, CI/CD).
  - [ ] Note which project story to LEAD with for this company — don't lead the same story every time.
  - [ ] Google: "[Company] tech blog", "[Company] engineering stack", "[Company] recent product launch". Read one article. Extract one real observation.
  - [ ] Check LinkedIn for your interviewers (if known). Note their background — shared tech = shared language.

**Pipeline Sheet Hygiene (15 min)**
- [ ] Open your pipeline sheet. Every row must have: Company | Role | JD URL | Stage | Last Action Date | Next Action | Next Action Date | Comp Range | Interviewer Names | Notes.
- [ ] Update any stale rows. If you're waiting on a company > 5 business days: send a polite follow-up today ("Following up on my [round] on [date] — happy to provide any additional context").

**Self-Check**
- [ ] For each active company: can you name their primary product, their likely tech stack, and one specific thing you'd bring to their team that you haven't mentioned before?

---

### Tuesday, Aug 25 — Execution Drills: Coding Interview Mechanics

📌 **Study today:** Narrated solve (LC 207 or 994) + recovery drills (LC 124, 297) · interview execution mechanics — clarifying, thinking out loud, graceful "I don't know"

**DSA Maintenance (15–20 min)**
- [ ] Pick one problem from your weak-area list. Solve it with narration — talk through your approach out loud as if the interviewer is listening.
- [ ] After solving: state time complexity, space complexity, and one alternative approach (even if worse). This is the habit that separates strong candidates from mediocre ones.

**Interview Execution Drill (60 min)**
This is a structured solo mock — full execution practice, not theory.

*Phase 1: Clarifying Questions (10 min)*
- [ ] Take any system design or coding problem (e.g., "Design a rate limiter" or "Find all paths in a grid"). Before you write a single line: practice asking 5 clarifying questions out loud. Examples:
  - "What scale — thousands of requests per second or millions?"
  - "Should paths avoid obstacles? Is the grid always rectangular?"
  - "Is this a one-time query or does it need to support updates?"
  - "What are the latency constraints? Read-heavy or write-heavy?"
  - "Do I optimise for space or time here?"
- [ ] Goal: make it a habit to pause and clarify before acting. Interviewers who skip clarification and dive in look impulsive.

*Phase 2: Thinking Out Loud (20 min)*
- [ ] Take a fresh problem (LC 207 Course Schedule or LC 994 Rotting Oranges). Solve it, narrating every step audibly:
  - "My first instinct is BFS here because..."
  - "I see a potential issue with cycles — let me think about how to handle that..."
  - "I'll start with a simpler O(n²) brute force to validate my understanding, then optimise..."
  - "One edge case I want to handle is an empty input..."
- [ ] Record yourself. Play it back. Does it sound like a confident, structured thinker, or like someone muttering?

*Phase 3: Handling "I Don't Know" (15 min)*
- [ ] Pick 3 questions from your interview-qa.md you feel least confident about. For each, practice this recovery script out loud:
  - "I don't have the exact answer at the top of my head, but let me reason through it..."
  - "What I know for certain is [related concept]. From there, I'd expect [logical deduction]..."
  - "I've seen a similar pattern in [project/known concept] — I'd approach it this way..."
  - "If I were to implement this in production, my first step would be to look at [tool/doc] specifically for..."
- [ ] Goal: never freeze. A structured "I don't know" is better than a correct answer delivered with hesitation — it shows how you work under uncertainty.

*Phase 4: Recovery from Being Stuck (15 min)*
- [ ] Take a hard problem you haven't solved (LC 124 Binary Tree Maximum Path Sum or LC 297 Serialize/Deserialize Binary Tree). Get intentionally stuck. Then practice recovery:
  - "Let me re-read the constraints — maybe I'm overcomplicating this."
  - "Let me try a smaller example. If the input were [tiny example], what would the output be?"
  - "Let me think about what data structure naturally models this."
  - "Could I solve a simpler version first?"
  - Ask: "Could you give me a hint on the data structure you have in mind? I want to make sure I'm moving in the right direction."
- [ ] Goal: a stuck candidate who asks for a hint gracefully is respected more than one who sits in silence for 5 minutes.

**Self-Check**
- [ ] Did you narrate the full solve out loud without stopping to "think silently"?
- [ ] Can you state three different phrasings for "I don't know the exact answer, but..."?

---

### Wednesday, Aug 26 — Behavioural + HM Round Prep

📌 **Study today:** Two LC Easy warm-ups · behavioural STAR skeletons (6 stories, metric in each) · HM round prep (3-year vision, first 30 days, why-this-company)

**DSA Maintenance (15–20 min)**
- [ ] Two LC Easy problems from categories you haven't touched this week — warm-up, not study. 10 min max total.

**Behavioural Drill (45 min)**
- [ ] For each of these STAR stories, write a 4-bullet skeleton (S, T, A, R) — keep it on one page:
  1. Performance improvement (Smart360: 60s → 2–3s query optimisation). Lead: S3 Redis caching + N+1 fix.
  2. Technical decision with trade-offs (event-driven migration, strangler-fig). Lead: what you chose NOT to do and why.
  3. A time something broke in production (OOMKilled pod, DB migration blocking readiness probe, pipeline failure on rate limit). If nothing dramatic: be honest about scale, precise about debugging.
  4. Influence without authority — a time you convinced a team to change technical direction.
  5. Learning on the fly — a technology you picked up fast and what you produced with it.
  6. A project you're most proud of and why (Deep Fathom: full-stack ownership, IaC, LLM async, FedRAMP alignment).
- [ ] For each: say it out loud, timed. Target 90–120 seconds. Under 90s = you're skipping impact. Over 2 min = you're narrating process instead of outcome.
- [ ] Number check: every story must have at least one metric. "We improved performance" is not a STAR answer. "Latency dropped 96% from 60s to 2–3s" is.

**HM Round Prep (30 min)**
- [ ] Hiring-manager rounds assess strategic thinking, collaboration, growth trajectory, cultural signal. Prepare crisp answers for:
  - "Where do you want to be in 3 years?" — tie it to technical leadership + product impact, not just title.
  - "What do you look for in a team?" — be specific: "A team that does proper code review and takes architecture decisions seriously" > "collaborative and supportive".
  - "What's a technical topic you've been exploring on your own?" — have one real answer (K8s Gateway API replacing Ingress, LLM function calling patterns, Bicep AVM modules).
  - "What would you do in your first 30 days?" — show you understand onboarding: "understand the architecture, shadow the deployment process, read the incident retrospectives, identify the one thing I could contribute in week 2 without breaking trust."
  - "Why this company over others?" — MUST be tailored. Generic answers kill offers.

**Self-Check**
- [ ] Every STAR story has a number. Yes/No.
- [ ] You can answer "Why us specifically?" for each active company without hesitation. Yes/No.

---

### Thursday, Aug 27 — Live Round Execution Day

📌 **Study today:** Live round execution — company brief + JD top-3 + project metrics · Easy LC warm-up · thank-you note + written debrief within 12h

This is your template for any actual interview day. Use this checklist the morning of.

**2 hours before the round**
- [ ] Re-read your one-page company brief (tech stack, product, recent news, one observation).
- [ ] Re-read the JD — highlight the top 3 skills they're testing for.
- [ ] Re-read your project deep-dive for whichever project is most relevant (Smart360 or Deep Fathom). Know 3 metrics cold.
- [ ] Do NOT cram new material. If you don't know it by now, 2 hours won't fix it. Warm up instead.

**30 min before**
- [ ] Solve one Easy LC — not to learn, to warm up pattern recognition. Get your brain in "coding mode".
- [ ] Drink water. Check setup: mic, camera, stable internet, quiet room, browser tabs closed except interview tool and notes (if allowed).

**During the round**
- [ ] **First 60 seconds**: when they say "tell me about yourself," give the 60-second quantified version. End with: "I'm excited to talk to [company] specifically because [tailored reason]."
- [ ] **Clarify before coding**: even if the problem seems obvious, ask one clarifying question. It buys thinking time AND shows structure.
- [ ] **Narrate your thinking**: the interviewer can't see inside your head. Silence for >30 seconds is a red flag.
- [ ] **State complexity proactively**: after every solution, "This runs in O(n log n) time and O(n) space — the log n comes from the sort." Don't wait to be asked.
- [ ] **End with good questions**: always ask 2–3 real ones (see Sample Questions). "What does success look like in the first 6 months?" is almost always good.

**Within 12 hours after the round**
- [ ] Send a thank-you note to the recruiter (CC the interviewer if you have their email). 3–4 sentences: thank them, reference one specific thing from the conversation, reiterate fit, express genuine interest.
- [ ] Write your debrief (10 min): what you handled well; what you stumbled on; what the interviewer cared most about; what to change next round; what to review before the next round with this company.
- [ ] Update the pipeline sheet row with today's stage, next step, and debrief summary.

**DSA Maintenance (15–20 min)**
- [ ] If you had a live round today, do this after the debrief — not before.
- [ ] Pick one problem the interview made you think about. Solve it properly. Reinforce the pattern.

**Self-Check**
- [ ] Did the thank-you note go out within 12 hours? Yes/No.
- [ ] Is the pipeline sheet updated? Yes/No.
- [ ] Is the debrief written? Yes/No.

---

### Friday, Aug 28 — System Design + Java Deep Execution Mock

📌 **Study today:** Heap / quickselect (LC 347, 215) · system design execution mock (distributed LLM job queue or multi-tenant auth) · Java deep-dive (@Transactional self-invocation, bean lifecycle, circuit breaker states)

**DSA Maintenance (20 min)**
- [ ] Timed solve: LC 347 (Top K Frequent Elements) or LC 215 (Kth Largest Element). High-frequency across all tiers. Clean solve with heap/quickselect narration in ≤ 20 min.

**System Design Execution Mock (60 min)**
- [ ] Pick one: "Design a distributed job queue for long-running LLM tasks" (from Deep Fathom) or "Design a multi-tenant authorization service" (from Smart360).
- [ ] Time-box hard — 45 min as if live, 15 min debrief. Follow this sequence religiously:
  1. **Clarify (5 min)**: scale, latency, consistency vs availability, client type.
  2. **High-level design (10 min)**: boxes and arrows; name the components; don't over-engineer yet.
  3. **Deep dive one component (15 min)**: pick the hardest (job dispatcher + LLM router). Data model, API contract, failure handling.
  4. **Scale and failure (10 min)**: where does this break at 10× load? What's the bottleneck? What would you add (read replicas, sharding, circuit breakers)?
  5. **Trade-offs (5 min)**: what did you NOT do and why? What would you change with more time?
- [ ] After: score yourself honestly on each phase. Note what you skimped on.

**Java Deep-Dive Execution (30 min)**
- [ ] Take 5 questions from interview-qa.md — Spring internals, transactions, AOP — and answer them out loud without reading. Then check against your notes.
- [ ] Rehearse the hardest: `@Transactional` self-invocation, bean lifecycle BeanPostProcessor timing, circuit breaker state transitions.
- [ ] Watch HOW you deliver, not just accuracy. Confident? Top-down (conclusion first, then detail)? Did you give a project example?

**Self-Check**
- [ ] Could you do the full system design sequence under time pressure without a checklist?
- [ ] Did you tie at least one design decision back to Smart360 or Deep Fathom?

---

### Saturday, Aug 29 — Per-Company Tailoring + First-Offer Readiness

📌 **Study today:** Trees (LC 98, 102) · per-company deep tailoring (tech-stack bridge answers, product observations, tailored questions) · first-offer response protocol + pipeline compression

**DSA Maintenance (20 min)**
- [ ] LC 98 (Validate BST) or LC 102 (Level Order Traversal). These appear in final rounds frequently. Clean solve, narrate out loud.

**Per-Company Deep Tailoring (90 min)**
For each active company with a round in the next 7 days:
- [ ] **Tech stack audit (15 min/company)**: Map their known stack to your experience. For each gap ("they use Kafka, you haven't used it in production"), prepare a bridge answer: "I haven't used Kafka directly, but I've implemented the same async decoupling with event-driven architecture using [approach] — consumer groups and offset management are familiar from studying it. I'm confident I'd be productive quickly."
- [ ] **Product observation (10 min/company)**: Read one recent blog post / product update / engineering article. Prepare one genuine observation: "I noticed you recently migrated to X — I'm curious what drove that over Y. In Deep Fathom we faced a similar choice and ended up with..."
- [ ] **Questions you'll ask (10 min/company)**: Write 5 questions specific to this company; you'll use 2–3 live. Avoid generic. Bad: "What does success look like?" Good: "I noticed your blog post on migrating monolith → microservices — what's the biggest operational surprise post-migration you didn't anticipate?"

**First-Offer Response Protocol (30 min)**
First offers can land this week. Be ready BEFORE the call so you don't accept on impulse.
- [ ] When an offer arrives — even a great one — never accept on the spot. Use this exact line: *"Thank you so much — I'm genuinely excited. I'd like to review everything carefully. Can I get back to you by [specific date, 3–5 business days out]?"* Every legitimate company says yes.
- [ ] The first offer is leverage. The moment one lands, every other live loop gets a compression nudge: *"I've received an offer I'm evaluating with a decision window of about a week. [Company] is genuinely a top choice for me — is there any way to accelerate the remaining steps?"*
- [ ] Quick first-pass evaluation (full framework is Week 10): is fixed base at/above floor? Variable/bonus structure? Joining bonus possible? ESOP/LTI vesting? Growth/band? Location/hybrid? BGV document requirements?
- [ ] Do NOT judge an offer on prestige alone — a ₹22 LPA offer at an unknown product company beats an ₹18 LPA GCC offer for your *next* jump's base.

**Pipeline Compression + Follow-Up (30 min)**
- [ ] Review every row. Any company not heard from in 5+ business days after a round: send a polite follow-up today.
- [ ] Template: "Hi [Name], following up on my [role] interview on [date]. I remain very interested and happy to provide any additional information. Looking forward to next steps."
- [ ] If you already have an offer or near-offer: contact every preferred-but-slower company proactively — *"I've received an offer from another company with a deadline of [date]. I'm genuinely more interested in [preferred company] — is there any way to accelerate the timeline?"*

**Self-Check**
- [ ] Can you bridge your experience to every technology gap in your active JDs?
- [ ] Does every active company have tailored questions you'll ask?
- [ ] Can you deliver the first-offer "buy time" line without hesitation?

---

### Sunday, Aug 30 — Full Mock + Pipeline Review + Week 10 Plan

📌 **Study today:** Fresh medium DSA (full narration) · full 45-min interview mock (scored) · pipeline sheet full review · Week 10 plan

**DSA Maintenance (20 min)**
- [ ] One fresh medium, clean solve, full narration. No review material — this is a performance test, not study.

**Full Interview Mock (60 min)**
- [ ] Set up a proper mock: camera on, professional background, external mic if available. Treat it exactly like a real round.
- [ ] Round type: choose what you have coming up next (coding, system design, or behavioural + HM). Full 45-min mock.
- [ ] Score on: (1) clarification before coding/designing, (2) narration throughout, (3) proactive complexity analysis, (4) trade-off awareness, (5) recovery from stumbles, (6) quality of questions asked.
- [ ] Write a debrief immediately. Be honest.

**Pipeline Sheet Full Review (30 min)**
- [ ] Go through every row. Current status for each:
  - Applied → Recruiter screen scheduled/done?
  - Recruiter screen done → Technical round scheduled?
  - Technical round done → Follow-up sent? Next round scheduled?
  - Offer received → Deadline noted? Competing timelines flagged?
- [ ] Identify the 2 companies you're most excited about. Confirm they have your full attention and customisation this week.
- [ ] Identify any "zombie" rows (applied > 2 weeks ago, no response) — decide: follow up once more or move on.

**Week 10 Planning (20 min)**
- [ ] Write down: which rounds are confirmed next week, what prep each needs, any offer decisions due.
- [ ] Note 2 things to do differently in rounds next week based on this week's debriefs.

---

## 🧠 Concepts to Master This Week (Interview Execution Skills)

### 1. Structured Thinking Out Loud

The goal is not to narrate everything — it's to narrate the right things so the interviewer can follow and redirect.

**What to narrate:**
- Your first instinct: "My initial thought is a BFS here because we're exploring level by level..."
- When you check a constraint: "I want to make sure this handles the empty input case..."
- When you see a trade-off: "I could use a HashMap for O(1) lookup at the cost of O(n) space. The problem doesn't constrain memory, so that's worth it."
- When you're about to optimise: "This works in O(n²). I think we can get to O(n log n) by sorting first — let me think through that..."

**What NOT to narrate:**
- Every syntax detail ("and now I'm declaring a variable of type int...")
- Reading the problem back to yourself silently for 3 minutes
- Restating what you just coded

**The pause-narrate rule**: if you've been silent for more than 20 seconds, say something — even "I'm thinking about the base case here..." A thinking interviewer is invisible. A narrating thinker is demonstrating their value.

---

### 2. Clarifying Requirements Before Coding

This is a skill, not a formality. It changes how interviewers perceive your seniority.

*Coding problems:* "Can the input contain duplicates?" · "Is the input guaranteed sorted?" · "What do I return if there's no solution — null, empty, or throw?" · "What's the expected scale — optimise for memory or time?"

*System design:* "What's the expected request volume at peak?" · "Strong consistency or is eventual acceptable?" · "What's the read/write ratio?" · "Global or single-region?" · "Acceptable latency for the core operation?"

**The rule**: ask 2–3 questions that ACTUALLY change your design or approach. Don't ask 7 to appear thorough — that looks like stalling.

---

### 3. Handling "I Don't Know" Without Freezing

Interviewers expect you not to know some things. They're testing how you respond, not whether you know everything.

**The recovery framework:**
1. Acknowledge honestly: "I don't have the exact mechanism at the top of my head."
2. Anchor on what you DO know: "What I'm confident about is [related concept]."
3. Reason toward the answer: "Applying that reasoning here, I'd expect [logical extension]..."
4. Offer a production substitute: "In practice I'd look at [specific approach/doc/tool] first..."
5. Invite correction: "Does that directional answer align with what you're looking for, or am I off?"

**Example:** Q: "How does `@Async` interact with `@Transactional` on the same method?" Recovery: "I haven't specifically tested that, but `@Async` runs in a different thread and `@Transactional` binds the transaction to the current thread via ThreadLocal. So the caller's transaction context wouldn't propagate to the async thread — the async method runs outside any transaction unless it starts its own. I'd verify via `TransactionSynchronizationManager`. Is that the kind of answer you're exploring?"

---

### 4. Recovering from a Stuck Moment

Being stuck is not the problem. Staying silent while stuck is.

1. **Reframe**: "Let me re-read this — I want to make sure I'm not over-constraining myself."
2. **Concrete example**: "Let me trace a small input manually and see what pattern emerges."
3. **Data structure first**: "What if this is a graph problem? What are the nodes and edges?"
4. **Ask for a directional hint gracefully**: "I sense the key insight involves [partial understanding], but I'm stuck on [specific part]. Could you point me in a direction?"

Asking for a hint is not failure. Sitting silently for 4 minutes is.

---

### 5. Strong Questions to Ask the Interviewer

The questions you ask signal as much as the answers you give.

*Technical depth:* "What's the hardest technical problem the team has faced in the last 6 months?" · "How do you currently handle [specific challenge relevant to their stack]?" · "What does your deployment pipeline look like — how often do you ship to prod?"

*Product/architecture:* "Your [feature] is interesting — what drove the decision to [specific choice you read about]?" · "How does the team balance feature velocity with technical debt?" · "What's the most common class of alert on your on-call rotation?"

*Team/growth:* "What would make someone exceptional in this role in 12 months — not just good?" · "How does the team share technical knowledge — design docs, RFCs, architecture reviews?" · "What's the thing you wish you'd known before joining?"

---

## 🎤 Sample Interview Questions (incl. Curveballs) — Final-Round & HM Style

### Java / Spring Boot — Final Round

**1. "Walk me through a real bug you debugged in a Spring app. Root cause and how you found it?"**
- Pointer: Not theory — have a specific story. Best for you: the N+1 query bug in Smart360 — how you found it (Hibernate statistics, slow query log), diagnosed it (each entity triggered a separate SELECT), fixed it (JOIN FETCH + EntityGraph), and learned (check Hibernate statistics before any complex query). Include the metric: 60s → 2–3s.

**2. "If `@Transactional` already handles DB writes, why also need the Outbox Pattern?"**
- Pointer: `@Transactional` guarantees DB atomicity only. It does NOT make your Kafka/RabbitMQ publish atomic with the DB write. If the app crashes after commit but before publish, the event is lost silently. Outbox writes the event to the same DB transaction as the domain change; a relay publishes from that table — guaranteed eventual publish because it's durable in the DB.

**3. "Curveball: App starts fine locally but fails readiness in K8s with an empty response from /actuator/health. What first?"**
- Pointer: (1) `management.endpoints.web.exposure.include=health` set? (2) Is `management.server.port` different from server port — probing wrong port? (3) Custom `HealthIndicator` beans failing silently → aggregated health DOWN? (4) `management.health.defaults.enabled=false` disabled contributors? (5) Non-default context path — probe hitting wrong URL? Demonstrates operational depth.

**4. "You mentioned JWT blacklisting with Redis for logout. What if Redis goes down?"**
- Pointer: Resilience question. Two modes: fail-open (skip check, security risk) vs fail-closed (reject all, availability risk). Right answer: tokens have short expiry (15–30 min); during a brief outage, fail-open with enhanced logging/alerting is pragmatic — small security window. High-security (financial/healthcare): fail-closed is mandatory. State the trade-off. Also: make Redis HA (Sentinel/Cluster).

**5. "Curveball: Why does `@PreAuthorize` not work on a method in a class that `@Autowires` itself?"**
- Pointer: `@PreAuthorize` works through AOP — Spring wraps the bean in a proxy. Self-injection gives you the proxy, so `@PreAuthorize` DOES work on self-injected calls. The failure case is internal `this.method()` calls — `this` is the raw object, not the proxy, so AOP is bypassed. Fix: inject self and call `self.method()`, or restructure to avoid self-calls.

### System Design — Final Round

**6. "Design the async LLM job queue you built in Deep Fathom. Handle 10× current load."**
- Pointer: Home turf. (1) Client submits → `202 Accepted` + `jobId`. (2) Job persisted to PostgreSQL with status `PENDING`. (3) Message published to queue (Redis Streams or RabbitMQ). (4) Worker pool pulls, calls provider, updates `RUNNING` → `COMPLETE`/`FAILED`. (5) Client polls `/jobs/{id}/status` or SSE push. 10×: horizontal worker scaling (K8s HPA on queue depth via KEDA), read replicas for status polling, shard job table by tenant, circuit breakers per provider. Mention 2–20 min job duration and why synchronous HTTP was non-viable.

**7. "You implemented RBAC at repository/table/permission levels. Data model and enforcement?"**
- Pointer: Entities `User`, `Role`, `Permission`, `UserRole`, `RolePermission`. Fine-grained permissions (`REPO:READ`, `TABLE:WRITE`). JWT carries `roles`; `@PreAuthorize("hasAuthority('TABLE:WRITE')")` checks claims. For query-time row filtering, PostgreSQL RLS with `current_setting('app.tenant_id')` per-connection. Defense in depth: app-layer checks are fast, DB-layer RLS is a hard guarantee even if app code has a bug.

**8. "Curveball: Add multi-tenancy to Smart360 without rewriting everything?"**
- Pointer: Three approaches, increasing invasiveness: (1) schema-per-tenant — full isolation, but migrations run N times; (2) row-level `tenant_id` + RLS policy — lower cost, you did this in Deep Fathom, reference it; (3) DB-per-tenant — max isolation, max ops cost. For Smart360: row-level + RLS is pragmatic — same DB, minimal migration, RLS enforces isolation at the DB layer.

### Hiring Manager / Culture — Final Round

**9. "Two companies in 2.5 years. Why leave now, and what's different about what you're looking for?"**
- Pointer: Honest and forward-looking; don't criticise past employers. "I've had strong technical growth — full-stack ownership, IaC, LLM integration — but I want to work on a product reaching a large user base where my architectural decisions have visible, measurable business impact. Product companies and GCCs offer that. I'm ready for the jump."

**10. "Tell me about a time you disagreed with a senior's technical decision. What did you do?"**
- Pointer: Specific story. Describe the decision, why you disagreed (technical, not personal), how you raised it (async doc/RFC, not a hallway argument), the outcome. Extraordinary: even when overruled, you committed fully and made it work, and noted what you learned from their reasoning.

**11. "Curveball: A senior engineer is protective of part of the codebase and resistant to change. How do you work with them?"**
- Pointer: "I'd understand their perspective first — protectiveness usually comes from being burned before. I'd ask them to walk me through the system before touching anything. I'd propose changes as RFCs with explicit reasoning and invite their critique. I'd earn trust with small, correct changes before the core they care about. If it became a blocker on important work, I'd raise it with the lead — but try the direct approach first."

**12. "Where do you see yourself in 3 years?"**
- Pointer: Specific, not vague. "A senior engineer who owns a system — driving architectural decisions, mentoring 1–2 juniors, the person the team calls when something's on fire, with a direct line to product metrics. I'm not rushing to management — I want to go deep technically first." Adjust if they hire for a management track.

---

## 🌟 Extraordinary-Candidate Edge

**The five behaviours that separate the top 5% in active loops:**

**1. Specificity over generality — always.** Every interviewer has heard "I improved performance." Almost no one says "I reduced P95 query latency from 60s to 2–3s by diagnosing N+1 patterns with Hibernate statistics, rewriting 4 queries with JOIN FETCH and EntityGraph, and caching S3 pre-signed URLs in Redis — S3 API calls dropped 80%." The second is memorable. You close offers with memorable answers.

**2. Name the trade-off you didn't take.** Always include: "I considered [alternative] but chose [solution] because [concrete reason]. The trade-off was [what I gave up]." Example: "I considered Kafka, but the ops overhead for a new broker wasn't justified for this traffic — Redis Streams gave the durability and consumer-group semantics without the ops cost."

**3. Ask questions that show you've done your homework.** Average: "What's the team culture like?" Extraordinary: "I read your post-mortem on [specific incident] — how has the team's approach to reliability changed since?" Signals you already think like an insider.

**4. The close.** At the end of every round: "I want to make sure I've addressed anything uncertain for you — is there anything you'd like me to clarify or expand on?" This is a sales close. It surfaces lingering doubts you can address, and signals confidence. Most candidates don't do it.

**5. The thank-you note that's actually memorable.** Not "Thanks for your time." Instead: "Your point about [specific thing they said] made me think differently about [related concept] — I hadn't considered [their angle]. That's the kind of problem I want to work on." This makes them remember you against the other finalist.

---

**Pipeline Tracker — Recommended Layout:**

| Column | What Goes Here |
|---|---|
| Company | Company name + link to careers page |
| Role | Exact job title |
| JD URL | Direct link to the JD (screenshot it too — JDs disappear) |
| Stage | Applied / Recruiter Screen / Tech Round 1 / Tech Round 2 / HM / Offer / Rejected |
| Last Action | Date + what happened |
| Next Action | What you need to do next |
| Next Action Date | Hard deadline for the next action |
| Comp Range | Their stated range OR your Glassdoor/Levels.fyi estimate |
| Interviewer(s) | Name, title, LinkedIn — for personalised thank-you notes |
| Notes | Debrief summary, what they cared about, gaps they flagged |

**Operating rules:** Update within 24 hours of every interaction. Every row needs a Next Action + Date — a row with no next action is a dead lead. Review the full tracker every Sunday; act on every "due this week" item on Monday. Screenshot every JD as a PDF the moment you apply.

---

## 📊 End-of-Week Self-Assessment

Rate yourself 1–5 after Sunday's mock. Feed gaps directly into Week 10.

| Area | Target | Your Rating | Gap / Next Action |
|---|---|---|---|
| Clarified requirements before coding/designing — every round | 5 | | |
| Narrated thinking audibly — no silent stretches > 30s | 5 | | |
| Handled at least one "I don't know" gracefully with the framework | 5 | | |
| Stated time + space complexity proactively after every solution | 5 | | |
| Recovered from a stuck moment without going silent | 4 | | |
| Asked 2–3 tailored, researched questions in each round | 5 | | |
| Thank-you notes sent within 12 hours — every round | 5 | | |
| Post-round debrief written same day — every round | 5 | | |
| Pipeline tracker updated, zero stale rows | 5 | | |
| Per-company tailoring done the day before every round | 5 | | |
| Smart360 performance story (STAR + metrics) — deliverable in 90s cold | 5 | | |
| Deep Fathom architecture story — 4 min version, no notes | 5 | | |
| LLM async job queue design — whiteboard-ready | 5 | | |
| First-offer "buy time" response — deliverable without hesitation | 5 | | |
| "Why us?" answer — tailored and genuine for each active company | 5 | | |
| 90-second differentiator pitch — fluid and specific | 5 | | |
| Offer received OR final round scheduled for at least 1 company | 5 | | |

**Scoring guide:**
- 5: Did it in every round, without prompting, under pressure.
- 4: Did it most of the time — one or two rounds where it slipped.
- 3: Know I should do it, not yet a habit under pressure.
- 2: Consistently skipping or stumbling — drill before next round.
- 1: Did not do this at all — flag for immediate fix in Week 10.

**Week 9 exit criteria (minimum bar):**
- Every live round had a thank-you note and a written debrief.
- Pipeline tracker has zero rows without a Next Action Date.
- At least 2 rounds completed or scheduled for the following week.
- Can execute the "I don't know" recovery framework without reading this doc.
- If a first offer landed: bought time correctly and sent compression nudges to the rest of the pipeline.

**If you missed the bar:** Don't skip debriefs because "you were tired after the round." The debrief is the compound interest on each interview — without it, you repeat the same mistakes next round. Write it even if it's 5 bullets.

---

*Week 9 complete → Week 10 (Aug 31–Sep 6): Offers + Negotiation — offer evaluation (in-hand vs CTC), the current/expected-CTC deflection, the competing-offer leverage call, and asking for a joining bonus / ESOP.*
