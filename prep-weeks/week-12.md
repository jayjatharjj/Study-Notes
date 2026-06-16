# Week 12 — Interview Loops (Aug 31–Sep 6, 2026)

> You are no longer preparing for interviews — you are IN them. The work now is execution, not accumulation. Every round is a performance you've rehearsed. Light DSA maintenance keeps the edge sharp. Everything else is tailored, deliberate, and followed up.

---

## 🎯 Week Goal

1. Execute every live interview round with structured thinking-out-loud, clean narration of trade-offs, and zero freezing on tough questions — even ones you don't fully know the answer to.
2. Keep the pipeline sheet live and current: every company, every stage, every follow-up logged within 24 hours of each interaction.
3. Send a personalised thank-you / follow-up note within 12 hours of every interview round. Debrief each round in writing within the same day.
4. Tailor your preparation the day before every round — re-read company tech blog, re-read your project deep-dive, find one thing about their product to mention.

---

## ✅ By Sunday you can...

- Execute a full 45-minute technical interview — DSA + system design or deep Java/Spring dive — without freezing, narrating your thinking audibly throughout.
- Handle any of these gracefully: "I don't know the answer exactly, but here's how I'd reason through it"; "let me make sure I understand the problem before I start coding"; "here's a simpler solution, and here's the optimised version with the tradeoff."
- Name the tech stack, recent product news, and one meaningful observation about each company currently in your active loop.
- Recite your 90-second differentiator pitch cold, in any sequence, without sounding rehearsed.
- Ask 2–3 genuinely good, tailored questions to every interviewer that are specific to their product, team, or tech choices — not boilerplate.
- Do a written post-interview debrief in 10 minutes: what went well, what to fix, what they seemed to care most about.
- Maintain the pipeline tracker with zero stale rows.

---

## 📅 Daily Checklist (Mon–Sun)

---

### Monday, Aug 31 — Loop Entry: Tailoring + Light DSA

**DSA Maintenance (15–20 min)**
- [ ] Solve one medium problem you haven't touched in 2+ weeks — pick from: LC 79 (Word Search), LC 200 (Number of Islands), or LC 33 (Search in Rotated Sorted Array). Time yourself. The goal is to stay warm, not learn new material.
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

### Tuesday, Sep 1 — Execution Drills: Coding Interview Mechanics

**DSA Maintenance (15–20 min)**
- [ ] Pick one problem from your weak-area list. Solve it with narration — talk through your approach out loud as if the interviewer is listening.
- [ ] After solving: state the time complexity, space complexity, and one alternative approach (even if worse). This is the habit that separates strong candidates from mediocre ones.

**Interview Execution Drill (60 min)**
This is a structured solo mock — full execution practice, not theory.

*Phase 1: Clarifying Questions (10 min)*
- [ ] Take any system design or coding problem (e.g., "Design a rate limiter" or "Find all paths in a grid"). Before you write a single line: practice asking 5 clarifying questions out loud. Examples:
  - "What scale are we designing for — thousands of requests per second or millions?"
  - "Should paths avoid obstacles? Is the grid always rectangular?"
  - "Is this a one-time query or does it need to support updates?"
  - "What are the latency constraints? Is this read-heavy or write-heavy?"
  - "Do I optimise for space or time here?"
- [ ] Goal: make it a habit to always pause and clarify before acting. Interviewers who skip clarification and dive in look impulsive.

*Phase 2: Thinking Out Loud (20 min)*
- [ ] Take a fresh problem (e.g., LC 207 Course Schedule or LC 994 Rotting Oranges). Solve it, but narrate every step audibly:
  - "My first instinct is BFS here because..."
  - "I see a potential issue with cycles — let me think about how to handle that..."
  - "I'm going to start with a simpler O(n²) brute force to validate my understanding, then optimise..."
  - "One edge case I want to make sure I handle is an empty input..."
- [ ] Record yourself on your phone. Play it back. Does it sound like a confident, structured thinker, or like someone muttering?

*Phase 3: Handling "I Don't Know" (15 min)*
- [ ] Pick 3 questions from your interview-qa.md that you feel least confident about. For each, practice the following out-loud recovery script:
  - "I don't have the exact answer at the top of my head, but let me reason through it..."
  - "What I know for certain is [related concept]. From there, I'd expect [logical deduction]..."
  - "I've seen a similar pattern in [project/known concept] — I'd approach it this way..."
  - "If I were to implement this in production, my first step would be to look at [tool/doc] specifically for..."
- [ ] Goal: never freeze. A structured "I don't know" answer is better than a correct answer delivered with hesitation, because it shows how you work under uncertainty — which is always the real condition.

*Phase 4: Recovery from Being Stuck (15 min)*
- [ ] Take a hard problem you haven't solved before (LC 124 Binary Tree Maximum Path Sum or LC 297 Serialize/Deserialize Binary Tree). Get intentionally stuck. Then practice recovery:
  - Step 1: "Let me re-read the problem constraints — maybe I'm overcomplicating this."
  - Step 2: "Let me try a smaller example. If the input were [tiny example], what would the output be?"
  - Step 3: "Let me think about what data structure naturally models this."
  - Step 4: "Could I solve a simpler version of this problem first?"
  - Ask the interviewer: "Could you give me a hint on the data structure you have in mind? I want to make sure I'm moving in the right direction."
- [ ] Goal: a stuck candidate who asks for a hint gracefully is respected more than one who sits in silence for 5 minutes.

**Self-Check**
- [ ] Did you narrate the full solve out loud without stopping to "think silently"?
- [ ] Can you state three different phrasings for "I don't know the exact answer, but..."?

---

### Wednesday, Sep 2 — Behavioural + HM Round Prep

**DSA Maintenance (15–20 min)**
- [ ] Two LC Easy problems from categories you haven't touched this week — use them as warm-up, not study. 10 min max total.

**Behavioural Drill (45 min)**
- [ ] For each of the following STAR stories, write a 4-bullet skeleton (S, T, A, R) — keep it on one page:
  1. Performance improvement (Smart360: 60s → 2–3s query optimisation). Lead: S3 Redis caching + N+1 fix.
  2. Technical decision with trade-offs (event-driven migration, strangler-fig approach). Lead: what you chose NOT to do and why.
  3. A time something broke in production (prepare a real story — OOMKilled pod, DB migration blocking readiness probe, pipeline failure on rate limit). If nothing dramatic: be honest about the scale, but be precise about the debugging.
  4. Influence without authority — a time you convinced a team to change direction technically.
  5. Learning on the fly — a technology you had to pick up fast and what you produced with it.
  6. A project you're most proud of and why (Deep Fathom: full-stack ownership, IaC, LLM async, FedRAMP alignment).

- [ ] For each story: say it out loud, timed. Target 90–120 seconds. If you're under 90s, you're skipping impact. If you're over 2 min, you're narrating process instead of outcome.
- [ ] Number check: every single story must have at least one metric. No exceptions. "We improved performance" is not a STAR answer. "Latency dropped 96% from 60 s to 2–3 s" is.

**HM Round Prep (30 min)**
- [ ] Hiring manager rounds assess: strategic thinking, collaboration, growth trajectory, cultural signal. Prepare crisp answers for:
  - "Where do you want to be in 3 years?" — tie it to technical leadership + product impact, not just title.
  - "What do you look for in a team?" — be honest, be specific. Examples: "A team that does proper code review and takes architecture decisions seriously" > "collaborative and supportive".
  - "What's a technical topic you've been exploring on your own recently?" — have one real answer (e.g., K8s Gateway API replacing Ingress, LLM function calling patterns, Bicep AVM modules).
  - "What would you do in your first 30 days?" — show that you understand onboarding: "understand the existing architecture, shadow the current deployment process, read the incident retrospectives, identify the one thing I could contribute in week 2 without breaking trust."
  - "Why this company over others?" — this MUST be tailored. Generic answers kill offers.

**Self-Check**
- [ ] Every STAR story has a number. Yes/No.
- [ ] You can answer "Why us specifically?" for each active company in your loop without hesitation. Yes/No.

---

### Thursday, Sep 3 — Live Round Execution Day

This is your template for any actual interview day. Use this checklist the morning of.

**2 hours before the round**
- [ ] Re-read your one-page company brief (tech stack, product, recent news, one observation).
- [ ] Re-read the JD — highlight the top 3 skills they're testing for.
- [ ] Re-read your project deep-dive for whichever project is most relevant to this company (Smart360 or Deep Fathom). Know 3 metrics cold.
- [ ] Do NOT cram new material. If you don't know it by now, 2 hours won't fix it. Warm up instead.

**30 min before**
- [ ] Solve one Easy LC problem — not to learn, to warm up your pattern recognition. Just get your brain in "coding mode".
- [ ] Drink water. Check your setup: mic, camera, stable internet, quiet room, browser tabs closed except interview tool and your notes (if allowed).

**During the round**
- [ ] **First 60 seconds**: make an impression. When they say "tell me about yourself", give the 60-second version — quantified. End with: "I'm excited to talk to [company] specifically because [tailored reason]."
- [ ] **Clarify before coding**: even if the problem seems obvious, ask one clarifying question. It buys you thinking time AND shows structured approach.
- [ ] **Narrate your thinking**: the interviewer can't see inside your head. If you're thinking, say what you're thinking. Silence for >30 seconds is a red flag to interviewers.
- [ ] **State complexity proactively**: after every solution, say "This runs in O(n log n) time and O(n) space — the log n comes from the sort." Don't wait to be asked.
- [ ] **End the round with good questions**: see the question bank in the Sample Questions section. Always ask 2–3 real questions. "What does success look like in the first 6 months?" is almost always a good one.

**Within 12 hours after the round**
- [ ] Send a thank-you note to the recruiter (CC the interviewer if you have their email). 3–4 sentences: thank them, reference one specific thing from the conversation, reiterate your fit, express genuine interest.
- [ ] Write your debrief (10 min):
  - What questions came up that you handled well?
  - What questions did you stumble on or handle poorly?
  - What did the interviewer seem to care most about?
  - What would you change for the next round?
  - What should you review before the next round with this company?
- [ ] Update the pipeline sheet row with today's stage, next step, and debrief summary.

**DSA Maintenance (15–20 min)**
- [ ] If you had a live round today, do this after the debrief — not before.
- [ ] Pick one problem that the interview made you think about. Solve it properly. Reinforce the pattern.

**Self-Check**
- [ ] Did the thank-you note go out within 12 hours? Yes/No.
- [ ] Is the pipeline sheet updated? Yes/No.
- [ ] Is the debrief written? Yes/No.

---

### Friday, Sep 4 — System Design + Java Deep Execution Mock

**DSA Maintenance (20 min)**
- [ ] Timed solve: LC 347 (Top K Frequent Elements) or LC 215 (Kth Largest Element). These are high-frequency across all company tiers. Clean solve with heap/quickselect narration in ≤ 20 min.

**System Design Execution Mock (60 min)**
- [ ] Pick one of these: "Design a distributed job queue for long-running LLM tasks" (directly from your Deep Fathom experience) or "Design a multi-tenant authorization service" (from Smart360).
- [ ] Time-box the session hard — 45 min as if live, 15 min debrief.
- [ ] During the 45 min, follow this sequence religiously:
  1. **Clarify (5 min)**: scale, latency requirements, consistency vs availability, client type.
  2. **High-level design (10 min)**: draw boxes and arrows. Name the components. Don't over-engineer yet.
  3. **Deep dive one component (15 min)**: pick the hardest component (e.g., the job dispatcher + LLM router). Go deep: data model, API contract, failure handling.
  4. **Scale and failure (10 min)**: where does this break at 10× load? What's the bottleneck? What would you add (read replicas, sharding, circuit breakers)?
  5. **Trade-offs (5 min)**: what did you NOT do and why? What would you change with more time?

- [ ] After the 45 min: score yourself honestly on each phase. Note what you skimped on.

**Java Deep-Dive Execution (30 min)**
- [ ] Take 5 questions from interview-qa.md — Spring internals, transactions, AOP — and answer them out loud without reading the answer. Then check against your notes.
- [ ] Specifically rehearse the hardest ones: `@Transactional` self-invocation, bean lifecycle BeanPostProcessor timing, circuit breaker state transitions.
- [ ] Pay attention to HOW you deliver the answer, not just accuracy. Does it sound confident? Did you structure it top-down (conclusion first, then detail)? Did you give an example from your own project?

**Self-Check**
- [ ] Could you do the full system design sequence under time pressure without a checklist?
- [ ] Did you tie at least one system design decision back to a concrete experience from Smart360 or Deep Fathom?

---

### Saturday, Sep 5 — Per-Company Tailoring + Questions Bank Refresh

**DSA Maintenance (20 min)**
- [ ] LC 98 (Validate BST) or LC 102 (Level Order Traversal). These appear in final rounds frequently. Clean solve, narrate out loud.

**Per-Company Deep Tailoring (90 min)**
For each active company in your pipeline that has a round coming up in the next 7 days:

- [ ] **Tech stack audit (15 min per company)**: Map their known stack to your experience. For each gap ("they use Kafka, you haven't used it in production"), prepare a bridge answer: "I haven't used Kafka directly, but I've implemented the same async decoupling with event-driven architecture using [approach] — the concepts of consumer groups and offset management are familiar from studying it for [context]. I'm confident I'd be productive quickly."
- [ ] **Product observation (10 min per company)**: Read one recent company blog post, product update, or engineering article. Prepare one genuine observation: "I noticed you recently migrated to X — I'm curious what drove that over Y. In Deep Fathom we faced a similar choice and ended up with..." This is an extraordinary-candidate signal in HM rounds.
- [ ] **Questions you'll ask (10 min per company)**: Write 5 questions specific to this company. You'll use 2–3 in the actual round. Avoid generic questions. Bad: "What does success look like?" Good: "I noticed your blog post on migrating from a monolith to microservices last year — what's been the biggest operational surprise post-migration that you didn't anticipate?"

**Pipeline Review + Follow-Up (30 min)**
- [ ] Review every row in the tracker. Any company you haven't heard from in 5+ business days after a round: send a polite follow-up today.
- [ ] Template: "Hi [Name], I wanted to follow up on my [role] interview on [date]. I remain very interested in the opportunity and am happy to provide any additional information. Looking forward to hearing about next steps."
- [ ] For offers or near-offers: check if you have competing timelines. If a company you prefer is slower than one with an offer, contact the preferred company's recruiter proactively: "I've received an offer from another company with a deadline of [date]. I'm genuinely more interested in [preferred company] — is there any way to accelerate the timeline?"

**Self-Check**
- [ ] Can you bridge your experience to every technology gap in your active company JDs?
- [ ] Does every active company have tailored questions you'll ask in the round?

---

### Sunday, Sep 6 — Full Mock + Pipeline Review + Week 13 Plan

**DSA Maintenance (20 min)**
- [ ] One fresh medium problem, clean solve, full narration. No review material — this is a performance test, not study.

**Full Interview Mock (60 min)**
- [ ] Set up a proper mock: camera on, professional background, external mic if available. Treat it exactly like a real round.
- [ ] Round type: choose the one you have coming up next (coding, system design, or behavioral + HM). Do a full 45-min mock.
- [ ] Score it on: (1) clarification before coding/designing, (2) narration throughout, (3) proactive complexity analysis, (4) trade-off awareness, (5) recovery from any stumbles, (6) quality of questions asked at the end.
- [ ] Write a debrief immediately. Be honest.

**Pipeline Sheet Full Review (30 min)**
- [ ] Go through every row. Current status for each:
  - Applied → Recruiter screen scheduled/done?
  - Recruiter screen done → Technical round scheduled?
  - Technical round done → Follow-up sent? Next round scheduled?
  - Offer received → Deadline noted? Competing timelines flagged?
- [ ] Identify the 2 companies you're most excited about. Confirm they have your full attention and customisation this week.
- [ ] Identify any "zombie" rows (applied > 2 weeks ago, no response) — decide: follow up one more time or move on.

**Week 13 Planning (20 min)**
- [ ] Write down: which rounds are confirmed for next week, what prep each one needs, any new companies to add to the pipeline.
- [ ] Note 2 things you want to do differently in rounds next week based on this week's debriefs.

---

## 🧠 Concepts to Master This Week (Interview Execution Skills)

### 1. Structured Thinking Out Loud

The goal is not to narrate everything — it's to narrate the right things so the interviewer can follow and redirect.

**What to narrate:**
- Your first instinct: "My initial thought is a BFS here because we're exploring level by level..."
- When you're checking a constraint: "I want to make sure this handles the empty input case..."
- When you see a trade-off: "I could use a HashMap here for O(1) lookup, at the cost of O(n) space. Given the problem doesn't constrain memory, that seems worth it."
- When you're about to optimise: "This works in O(n²). I think we can bring it down to O(n log n) by sorting first — let me think through that..."

**What NOT to narrate:**
- Every syntax detail ("and now I'm declaring a variable of type int...")
- Reading the problem back to yourself silently for 3 minutes
- Restating what you just coded

**The pause-narrate rule**: if you've been silent for more than 20 seconds, say something — even "I'm thinking about the base case here..." A thinking interviewer is invisible. A narrating thinker is demonstrating their value.

---

### 2. Clarifying Requirements Before Coding

This is a skill, not a formality. It changes how interviewers perceive your seniority.

**Questions by problem type:**

*Coding problems:*
- "Can the input contain duplicates?"
- "Is the input guaranteed to be sorted?"
- "What should I return if there's no solution — null, empty, or throw?"
- "What's the expected scale — should I optimise for memory or time?"

*System design:*
- "What's the expected request volume at peak?"
- "Are we designing for strong consistency or is eventual consistency acceptable?"
- "What's the read/write ratio?"
- "Is this a global system, or single-region?"
- "What's the acceptable latency for the core operation?"

**The rule**: ask 2–3 questions that ACTUALLY change your design or approach. Don't ask 7 questions to appear thorough — that looks like stalling. Ask the ones whose answers matter.

---

### 3. Handling "I Don't Know" Without Freezing

Interviewers expect you not to know some things. They are testing how you respond, not whether you know everything.

**The recovery framework:**

Step 1 — Acknowledge honestly: "I don't have the exact mechanism for that at the top of my head."

Step 2 — Anchor on what you DO know: "What I'm confident about is [related concept that you genuinely know]."

Step 3 — Reason toward the answer: "If I apply that reasoning here, I'd expect [logical extension]..."

Step 4 — Offer a production substitute: "In practice, when I've encountered this problem, I'd look at [specific approach/doc/tool] first..."

Step 5 — Invite correction or hint: "Does that directional answer align with what you're looking for, or am I off?"

**Example in practice:**
Q: "How does Spring's `@Async` interact with `@Transactional` on the same method?"
Recovery: "I haven't specifically tested that combination, but I know `@Async` runs the method in a different thread, and `@Transactional` binds the transaction to the current thread via ThreadLocal. So intuitively, the transaction context from the caller wouldn't propagate to the async thread — the async method would run outside any transaction unless it starts its own. I'd verify this with the `TransactionSynchronizationManager` source. Is that the kind of answer you're exploring?"

This is a better answer than silence, and it's often accurate.

---

### 4. Recovering from a Stuck Moment

Being stuck is not the problem. Staying silent while stuck is.

**The 4-step recovery sequence:**
1. **Reframe the problem**: "Let me re-read this — I want to make sure I'm not over-constraining myself."
2. **Try a concrete example**: "Let me trace through a small input manually and see what pattern emerges."
3. **Consider data structure first**: "What if I think about this as a graph problem? What nodes and edges would I define?"
4. **Ask for a directional hint gracefully**: "I have a sense that the key insight involves [your partial understanding], but I'm stuck on [specific part]. Would you be willing to point me in a direction?"

Asking for a hint is not failure. Sitting silently for 4 minutes is.

---

### 5. Strong Questions to Ask the Interviewer

The questions you ask signal as much as the answers you give. Mediocre candidates ask: "What does success look like in 90 days?" Strong candidates ask questions that show they've done research and thought about the role specifically.

**Template categories:**

*Technical depth:*
- "What's the hardest technical problem the team has faced in the last 6 months?"
- "How do you currently handle [specific technical challenge relevant to their stack]?"
- "What does your deployment pipeline look like — how often do you ship to prod?"

*Product and architecture:*
- "Your [product/feature] is interesting — what drove the decision to [specific technical or product choice you read about]?"
- "How does the team balance feature velocity with technical debt?"
- "What does the team's on-call rotation look like, and what's the most common class of alert you respond to?"

*Team and growth:*
- "What would make someone exceptional in this role in 12 months — as opposed to just good?"
- "How does the team share technical knowledge — design docs, RFC process, architecture reviews?"
- "What's the thing you wish you'd known before joining this team?"

---

## 🎤 Sample Interview Questions (incl. Curveballs) — Final-Round & HM Style

### Java / Spring Boot — Final Round

**1. "Walk me through a real bug you've debugged in a Spring application. What was the root cause and how did you find it?"**
- Pointer: This is not a theory question. Have a specific story. Best answer for you: the N+1 query bug in Smart360 — how you found it (Hibernate statistics enabled, slow query log), how you diagnosed it (each entity in the list triggered a separate SELECT), how you fixed it (JOIN FETCH + EntityGraph), and what you learned (check Hibernate statistics before writing any complex query). Include the metric: latency dropped from 60 s to 2–3 s.

**2. "If @Transactional is already handling your DB writes, why would you additionally need the Outbox Pattern?"**
- Pointer: `@Transactional` guarantees atomicity within the database — your DB write either succeeds or rolls back. It does not guarantee that your Kafka/RabbitMQ message publish also succeeds atomically. If the app crashes after the DB commit but before the publish, the event is lost — silently, permanently. Outbox writes the event to the same DB transaction as the domain change (in an `outbox` table). A separate relay process publishes from that table. The event is guaranteed to eventually publish because it's durable in the DB. This is the answer that shows you've thought about distributed systems beyond happy-path.

**3. "Curveball: Your Spring Boot app starts fine locally but fails readiness in Kubernetes with an empty response from /actuator/health. What do you check first?"**
- Pointer: (1) Is `management.endpoints.web.exposure.include=health` set? By default in some configs, the endpoint may be excluded. (2) Is the management port the same as the server port, or is `management.server.port` set to a different value? Kubernetes may be probing the wrong port. (3) Are there custom `HealthIndicator` beans that are failing silently and making the aggregated health DOWN? (4) Check `management.health.defaults.enabled=false` — someone may have disabled all health contributors. (5) Check if the app is running on a non-default context path and the probe is hitting `/actuator/health` instead of `/context-path/actuator/health`. This answer demonstrates operational depth.

**4. "You mentioned JWT blacklisting with Redis for logout. What happens if Redis goes down?"**
- Pointer: This is a resilience question. If Redis is unavailable and you're using it to check a blacklist on every request, two failure modes exist: (a) fail-open — skip the blacklist check, allow requests through even if the token was revoked (security risk); (b) fail-closed — reject all requests (availability risk). The right answer: for logout scenarios, tokens have short expiry (15–30 min); during a brief Redis outage, fail-open with enhanced logging/alerting is the pragmatic choice — you accept a small security window. For high-security contexts (financial, healthcare): fail-closed is mandatory. State the trade-off explicitly. Also mention: make Redis highly available (Sentinel or Cluster) to minimise the scenario.

**5. "Curveball: Why does Spring Security's @PreAuthorize not work on a method in a class that @Autowires itself?"**
- Pointer: `@PreAuthorize` works through AOP — Spring wraps your bean in a proxy. When you `@Autowire` a reference to the same bean (injecting self), you get the proxy. So `@PreAuthorize` DOES work on self-injected calls. The failure case is internal method calls (calling `this.method()` directly) — `this` refers to the raw object, not the proxy, so AOP is bypassed entirely. The fix: inject self (`@Autowired private MyService self`) and call `self.method()`. Or restructure to avoid self-calls. This is a subtle Spring internals answer that separates people who've debugged real issues.

---

### System Design — Final Round

**6. "Design the async LLM job queue you built in Deep Fathom. I want to see it handle 10× current load."**
- Pointer: This is your home turf — own it. Walk through: (1) Client submits → API layer returns `202 Accepted` + `jobId`. (2) Job persisted to PostgreSQL with status `PENDING`. (3) Message published to queue (Redis Streams or RabbitMQ — name what you used or what you'd use). (4) Worker pool pulls from queue, calls LLM provider, updates job status to `RUNNING` → `COMPLETE`/`FAILED`. (5) Client polls `/jobs/{id}/status` or receives SSE push. For 10×: horizontal scaling of workers (K8s HPA on queue depth via KEDA), read replicas for job status polling, sharding the job table by tenant if needed, circuit breakers per LLM provider to handle upstream throttling. Mention the 2–20 min job duration and why synchronous HTTP was non-viable.

**7. "You implemented RBAC at repository/table/permission levels. Walk me through the data model and enforcement."**
- Pointer: Entities: `User`, `Role`, `Permission`, `UserRole`, `RolePermission`. Permissions are fine-grained (e.g., `REPO:READ`, `TABLE:WRITE`, `PERMISSION:GRANT`). Roles aggregate permissions. Users have roles. At runtime: JWT contains `roles` claim. Spring Security's `@PreAuthorize("hasAuthority('TABLE:WRITE')")` checks against the JWT claims. For dynamic permission checking (query-time row filtering), use PostgreSQL RLS with `current_setting('app.tenant_id')` set per-connection. Explain why you chose this multi-layered approach: application-layer checks catch most cases fast; DB-layer RLS provides a hard guarantee even if app code has a bug. Defense in depth.

**8. "Curveball: How would you add multi-tenancy to your existing Smart360 data model without rewriting everything?"**
- Pointer: Three approaches with increasing invasiveness: (1) Schema-per-tenant: each tenant gets a separate schema in PostgreSQL. Complete isolation, easy backup/restore per tenant, but schema migrations must run N times. (2) Row-level: add `tenant_id` column to all tables, add a partial index on `(tenant_id, ...)`, enforce via RLS policy. You actually did this in Deep Fathom — reference that. Lower cost, harder to guarantee no cross-tenant leakage without RLS. (3) DB-per-tenant: max isolation, max operational cost. For Smart360 specifically: row-level + RLS is the pragmatic answer — same DB, minimal migration effort, RLS enforces isolation at the DB layer. Tie this to your Deep Fathom experience where you actually implemented it.

---

### Hiring Manager / Culture — Final Round

**9. "You've worked at two companies in 2.5 years. Why are you leaving now, and what's different about what you're looking for?"**
- Pointer: Be honest and forward-looking. Avoid criticising past employers. Good framing: "I've had strong technical growth — full-stack ownership, IaC, LLM integration — but I'm at the point where I want to work on a product that reaches a large user base and where my architectural decisions have visible, measurable impact on the business. Product companies and GCCs offer that. I'm ready for the jump." This shows maturity, self-awareness, and a clear reason they can believe in.

**10. "Tell me about a time you disagreed with a technical decision made by someone senior. What did you do?"**
- Pointer: Prepare a specific story. Structure: describe the decision, why you disagreed (technical reasoning, not personal), how you raised it (async doc/RFC, not a hallway argument), what happened (they agreed, partially agreed, or overruled you with valid reasoning). The extraordinary answer: even when overruled, you committed fully to the chosen approach and made it work — and you note what you learned from why they chose it. Interviewers are testing for: you voice concerns, you do it professionally, and you don't sulk when overruled.

**11. "Curveball: We have a senior engineer on the team who is very protective of certain parts of the codebase and resistant to changes. How would you handle working with them?"**
- Pointer: Don't pretend this never happens. A good answer: "I'd start by understanding their perspective — usually protectiveness comes from having been burned before (brittle code, repeated bugs, past incidents). I'd ask them to walk me through the system before touching anything. When I did propose changes, I'd frame them as RFCs with explicit reasoning, and I'd invite their critique before implementing. I'd earn trust with small, correct changes before touching the core they care about. If protectiveness became a blocker on important work, I'd raise it with the team lead — but I'd try the direct approach first."

**12. "Where do you see yourself in 3 years?"**
- Pointer: Avoid vague answers ("I want to grow"). Be specific to the kind of engineer/leader you want to become. Good answer: "In 3 years I want to be a senior engineer who takes full ownership of a system — not just writing code, but driving architectural decisions, mentoring 1–2 junior engineers, and being the person the team calls when something is on fire. I want my work to have a direct line to product metrics. I'm not in a rush to become a manager — I want to go deep technically first." Adjust if they're explicitly hiring for a management track.

---

## 🌟 Extraordinary-Candidate Edge

**The five behaviours that separate the top 5% in active loops:**

**1. Specificity over generality — always.**
Every interviewer has heard "I improved performance." Almost no one says "I reduced P95 query latency from 60 seconds to 2–3 seconds by diagnosing N+1 patterns with Hibernate statistics enabled, rewriting 4 critical queries with JOIN FETCH and EntityGraph, and caching S3 pre-signed URLs in Redis — total S3 API calls dropped 80%." The second answer is memorable. You close offers with memorable answers.

**2. Name the trade-off you didn't take.**
When you describe a technical decision, always include: "I considered [alternative], but chose [solution] because [concrete reason]. The trade-off was [what you gave up]." This demonstrates architectural thinking. Example: "I considered using Kafka here, but the team's operational overhead for a new broker wasn't justified for this traffic volume — Redis Streams gave us the durability and consumer group semantics we needed without the ops cost."

**3. Ask questions that show you've done your homework.**
The average candidate asks "What does the team culture look like?" The extraordinary candidate asks: "I read your post-mortem on the [specific incident you found] — how has the team's approach to reliability changed since then?" or "Your blog mentioned migrating from X to Y last quarter — what was the hardest part of that migration?" These questions signal that you are already thinking like an insider.

**4. The close.**
At the end of every round, before the interviewer wraps up, say: "I want to make sure I've addressed anything that's uncertain for you — is there anything you'd like me to clarify or expand on?" This is a sales close. It gives the interviewer a chance to raise lingering doubts (which you can address), and it signals confidence. Most candidates don't do this.

**5. The thank-you note that's actually memorable.**
Don't send "Thanks for your time, I enjoyed the conversation." Send: "Thank you for the conversation today. Your point about [specific thing they said] made me think differently about [related concept] — I hadn't considered [their angle]. That's the kind of problem I want to work on. I'm genuinely excited about [specific team/product/challenge they mentioned]." This note makes them remember you when they're comparing you to the other finalist.

---

**Pipeline Tracker — Recommended Layout:**

| Column | What Goes Here |
|---|---|
| Company | Company name + link to careers page |
| Role | Exact job title |
| JD URL | Direct link to the job description (JDs get taken down — screenshot it too) |
| Stage | Applied / Recruiter Screen / Tech Round 1 / Tech Round 2 / HM / Offer / Rejected |
| Last Action | Date + what happened (e.g., "Aug 31 — completed Tech Round 1 with [name]") |
| Next Action | What you need to do next (e.g., "Follow up with recruiter by Sep 5") |
| Next Action Date | Hard deadline for the next action |
| Comp Range | Their stated range OR your estimate from Glassdoor/Levels.fyi |
| Interviewer(s) | Name, title, LinkedIn — for personalised thank-you notes |
| Notes | Debrief summary, what they cared about, gaps they flagged |

**Operating rules for the tracker:**
- Update within 24 hours of every interaction — not "later this week."
- Every row must have a Next Action and Next Action Date — a row with no next action is a dead lead.
- Review the full tracker every Sunday. Act on every "due this week" action on Monday.
- Screenshot every JD as a PDF the moment you apply. JDs disappear within days of closing.

---

## 📊 End-of-Week Self-Assessment

Rate yourself 1–5 after Sunday's mock. Feed gaps directly into Week 13 prep.

| Area | Target | Your Rating | Gap / Next Action |
|---|---|---|---|
| Clarified requirements before coding/designing — every round | 5 | | |
| Narrated thinking audibly throughout — no silent stretches > 30s | 5 | | |
| Handled at least one "I don't know" gracefully with the framework | 5 | | |
| Stated time + space complexity proactively after every solution | 5 | | |
| Recovered from a stuck moment without going silent | 4 | | |
| Asked 2–3 tailored, researched questions in each round | 5 | | |
| Thank-you notes sent within 12 hours — every round | 5 | | |
| Post-round debrief written same day — every round | 5 | | |
| Pipeline tracker updated and zero stale rows | 5 | | |
| Per-company tailoring done the day before every round | 5 | | |
| Smart360 performance story (STAR + metrics) — deliverable in 90s cold | 5 | | |
| Deep Fathom architecture story — 4 min version, no notes | 5 | | |
| LLM async job queue design — whiteboard-ready | 5 | | |
| Behavioural bank (6 stories) — all have ≥ 1 metric, all ≤ 2 min | 5 | | |
| HM round questions answered specifically, not generically | 4 | | |
| "Why us?" answer — tailored and genuine for each active company | 5 | | |
| 90-second differentiator pitch — fluid and specific | 5 | | |
| Offer received OR final round scheduled for at least 1 company | 5 | | |

**Scoring guide:**
- 5: Did it in every round, without prompting, under pressure.
- 4: Did it most of the time — one or two rounds where it slipped.
- 3: Know I should do it but it's not yet a habit under pressure.
- 2: Consistently skipping or stumbling — must drill this before next round.
- 1: Did not do this at all this week — flag for immediate fix in Week 13.

**Week 12 exit criteria (minimum bar):**
- Every live round had a thank-you note and a written debrief.
- Pipeline tracker has zero rows without a Next Action Date.
- At least 2 rounds completed or scheduled for the following week.
- Can execute the "I don't know" recovery framework without reading this doc.

**If you missed the bar:** Don't skip debriefs because "you were tired after the round." The debrief is the compound interest on each interview — without it, you repeat the same mistakes in the next round. Write it even if it's 5 bullets. It will pay for itself.

---

*Week 12 complete → Week 13: Offer evaluation, negotiation strategy, managing competing timelines, and any final-round prep for companies still in loop.*
