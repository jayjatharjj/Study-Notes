# Week 10 — Peak + Active Applications (Aug 24–30, 2026)

> Interviews are live. Every hour matters. You are not studying anymore — you are performing. DSA is maintenance, not learning. Behavioral stories are your secret weapon. Applications are a pipeline, not a lottery. By Friday you have STAR stories written and rehearsed for every major behavioral category. By Sunday you have done a full end-to-end mock interview and can tell your story as a confident, numbers-backed narrative.

---

## 🎯 Week Goal

1. Complete 10–15 targeted applications to tier-1 product companies and GCCs; follow up on all Week 9 applications with a personalised message (≤ 3 sentences, reference a specific project or team detail).
2. Write and rehearse 6 polished STAR stories (performance win, conflict/disagreement, ownership/leadership, biggest failure, why leaving, tell me about yourself) — each under 2 minutes when spoken aloud.
3. Close DSA weak patterns: plug any gap from Week 9's self-assessment score < 4 — specifically Graphs (BFS/DFS, union-find), Backtracking (combinations/subsets), and any company-tagged problems for your applied companies.
4. Walk out of Sunday's full mock interview feeling unshakeable: technical, behavioral, and system design all coherent.

---

## ✅ By Sunday you can...

- Write and speak your 90-second "tell me about yourself" pitch perfectly — specific, quantified, no filler.
- Deliver any of the 6 STAR stories fluently under 2 minutes, with a specific number in every answer.
- Solve company-tagged Medium/Hard problems for Atlassian, Walmart, Goldman Sachs, Flipkart (or whichever 3 companies you applied to in Weeks 8–9) in ≤ 20 min.
- Explain the difference between BFS-on-a-graph vs BFS-on-a-tree, and why union-find beats repeated BFS for connectivity problems.
- Solve LC 200 (Number of Islands), LC 207 (Course Schedule), LC 695 (Max Area of Island), LC 78 (Subsets), LC 90 (Subsets II) from a blank editor.
- Answer "tell me about a time you were wrong" with a story that shows self-awareness and a concrete lesson, not a humble-brag.
- Answer "how do you handle ambiguity" with a specific example, not a platitude.
- Articulate "why this company" for at least 3 companies on your active list — specific to each company's tech or product.

---

## 📅 Daily Checklist (Mon–Sun, Aug 24–30)

---

### Monday, Aug 24 — DSA Weak Pattern: Graphs (BFS/DFS) + STAR Story #1

📌 **Study today:** Graphs — DFS/BFS flood-fill (LC 200, 695) · STAR story #1 (performance win — 60s→3s, N+1 + Redis)

**DSA (45 min)**
- [ ] **LC 200 – Number of Islands** (Medium). DFS/BFS flood-fill: mark visited cells in-place (set to '0') or with a visited array. Time O(m×n), space O(m×n) recursion stack worst case. Know both DFS and BFS implementations — interviewers often ask for the other one after you pick one.
- [ ] **LC 695 – Max Area of Island** (Medium). Same flood-fill; return count of cells, not just boolean. Adds an accumulator to the DFS. Practice returning the count cleanly — a common mistake is double-counting.
- [ ] After solving: write on paper the 4-directional `int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}}` pattern. You will use this for every 2D grid graph problem — hardcode it in muscle memory.

**Core Block — STAR Story #1: Performance Win (30 min)**
- [ ] This story already has a skeleton in interview-qa.md. Now make it air-tight and rehearse it out loud.
  - **Situation**: Smart360 data retrieval taking 60 seconds — causing HTTP timeout errors for end users attempting to load dashboards. The team had accepted it as a known issue for months.
  - **Task**: You volunteered (or were assigned) to own the fix. No change to the data model was allowed (other services depended on it).
  - **Action** (be specific, step-by-step):
    1. Enabled Hibernate statistics logging (`hibernate.generate_statistics=true`) to count queries per request — found 47 queries firing for a single dashboard load (N+1).
    2. Identified the exact `@OneToMany` relationships triggering the extra queries.
    3. Rewrote JPQL with `JOIN FETCH` and used `@EntityGraph` on repository methods that needed controlled eager loading.
    4. Added composite indexes on the most-filtered columns (`(tenant_id, created_at)`, `(user_id, status)`).
    5. Integrated Redis caching for repeated read queries with 5-minute TTL.
    6. Cached S3 pre-signed URLs in Redis with TTL = (URL expiry − 30s buffer).
  - **Result**: P95 latency dropped from 60 s → 2–3 s (96% reduction). S3 API calls dropped by 80%. Zero DB schema changes.
- [ ] Time yourself speaking this story aloud. Target: 90 seconds, under 2 minutes. Every sentence must have a specific detail — no vague "we improved things."

**Self-Check**
- [ ] Can you solve Number of Islands in both DFS and BFS in under 15 min?
- [ ] Can you tell the performance story without notes, under 2 minutes, with the number "96%" spoken naturally?

---

### Tuesday, Aug 25 — DSA: Graphs (Cycle Detection / Topological Sort) + STAR Story #2

📌 **Study today:** Graphs — cycle detection / topological sort (LC 207, 210) · STAR story #2 (conflict / disagreement) · application follow-ups

**DSA (45 min)**
- [ ] **LC 207 – Course Schedule** (Medium). Directed graph cycle detection via DFS with coloring (white=unvisited, gray=in-stack, black=done) OR Kahn's BFS topological sort (track in-degrees; add nodes with in-degree 0 to queue; if processed count < n, cycle exists). Know BOTH approaches — interviewers ask "now do it without recursion" and Kahn's is the answer.
- [ ] **LC 210 – Course Schedule II** (Medium). Same as 207 but return the actual topological order. With Kahn's: the BFS processing order IS the topological order. With DFS: push to result list on post-order (when node becomes black), then reverse.
- [ ] Know the connection: topological sort only exists on a DAG (directed acyclic graph). If there's a cycle, no valid ordering exists. This directly maps to dependency resolution in build systems, microservice startup ordering, and package management — mention this when describing your solution.

**Core Block — STAR Story #2: Conflict / Disagreement (30 min)**
- [ ] Write this story from scratch. It must be a REAL disagreement, not a manufactured one.
  - Possible scaffold (adapt to what actually happened):
    - **Situation**: During Smart360 or Deep Fathom, there was a design disagreement — perhaps over how to handle authentication (JWT vs session), or whether to use event-driven async vs synchronous REST for the user management migration, or whether to use Bicep vs an existing Terraform module.
    - **Task**: The decision mattered for the project. You had a strong technical opinion backed by data. The other party (senior dev, team lead, or stakeholder) disagreed.
    - **Action**:
      1. You documented both approaches with explicit trade-offs (don't just argue — bring evidence).
      2. You proposed a time-boxed proof-of-concept or a decision matrix.
      3. You listened to their concern and found the kernel of validity in it.
      4. You either reached consensus OR escalated to a third party with both options clearly framed.
    - **Result**: The decision was made without damaging the relationship. Quantify if possible: "we shipped on time," "the chosen approach had X outcome."
  - **Curveball version**: "Tell me about a time you disagreed with your manager." Same structure — key is to show you can push back professionally WITH data, not just comply.
- [ ] Speak it aloud. Time it. Keep it under 2 minutes.

**Application Block (20 min)**
- [ ] Pull up your Week 9 application tracking sheet. For every application sent 5+ days ago with no response:
  - [ ] Find the recruiter or hiring manager on LinkedIn.
  - [ ] Send a 3-sentence follow-up message: (1) reference your application date + role, (2) add one specific line about why that company, (3) offer to connect for a quick chat.
- [ ] Log follow-up date in the tracking sheet.

**Self-Check**
- [ ] Can you explain Kahn's algorithm step-by-step without notes?
- [ ] Is your conflict story written down (even rough draft) in a note or document?

---

### Wednesday, Aug 26 — DSA: Union-Find + STAR Story #3

📌 **Study today:** Union-Find / DSU (LC 684, 323/547) · STAR story #3 (ownership / leadership)

**DSA (45 min)**
- [ ] **LC 684 – Redundant Connection** (Medium). Union-Find (Disjoint Set Union): for each edge, if both endpoints are already in the same set → this edge creates the cycle → return it. Union by rank + path compression → near O(1) amortized per operation.
  - Implement cleanly: `find(x)` with path compression (recursive: `parent[x] = find(parent[x])`), `union(x, y)` with rank. Know this implementation cold — it's reusable across all connectivity problems.
- [ ] **LC 323 – Number of Connected Components in Undirected Graph** (Medium, Premium — use LC 547 Friend Circles instead if no Premium). Union-Find: for each edge, union the two nodes. Count distinct roots at end. Alternative: BFS/DFS flood-fill, count starts. Union-Find is faster in practice and uses less stack space.
- [ ] **Key insight to articulate**: "Union-Find beats repeated BFS for problems that ask ONLY about connectivity (not path), because it processes each edge once in near-constant time. BFS/DFS is better when you need the actual path or per-component information (like max area)."

**Core Block — STAR Story #3: Ownership / Leadership (30 min)**
- [ ] Write this story. Ownership means you took responsibility beyond your explicit role.
  - **Scaffold**:
    - **Situation**: A task or problem existed that nobody owned — or someone owned it but wasn't driving it. Maybe the CI/CD pipeline was slow and everyone complained but nobody fixed it. Maybe the Bicep IaC didn't exist and infrastructure was provisioned manually.
    - **Task**: You decided to own it. You didn't wait to be asked.
    - **Action** (very specific):
      - You assessed the scope: "Pipeline was at 23 min; I profiled it with GitLab pipeline trace and identified 3 bottlenecks."
      - You proposed a plan to your manager/team: "I estimated 2 days to implement parallel stages + BuildKit caching."
      - You executed it: built the DAG pipeline structure, added BuildKit, set up registry-backed cache.
      - You measured the outcome and reported it.
    - **Result**: "Pipeline dropped from 23 to 10 min (57% faster). Developer feedback: faster iteration loops. This change landed before a deadline-critical sprint."
  - The meta-message: you identify problems proactively, quantify them, propose solutions, execute independently, and measure impact. That is the definition of a senior engineer mindset at 2.5 YOE — this is your extraordinary-candidate signal.
- [ ] Speak it aloud. Under 2 minutes.

**Self-Check**
- [ ] Can you implement union-find with path compression + rank from scratch in < 10 min?
- [ ] Is the ownership story specific enough that it sounds like a diary entry, not a job description?

---

### Thursday, Aug 27 — DSA: Backtracking + STAR Story #4

📌 **Study today:** Backtracking — subsets/permutations (LC 78, 90, 46) · STAR story #4 (biggest failure & lesson) · 5 new applications

**DSA (45 min)**
- [ ] **LC 78 – Subsets** (Medium). Backtracking: at each step, choose to include or exclude the current element. Build the recursion tree. Result: 2^n subsets. Time O(n × 2^n), space O(n) call stack. Also know the iterative bit-mask approach: for mask from 0 to 2^n-1, include element i if bit i is set.
- [ ] **LC 90 – Subsets II** (Medium). Same as LC 78 but with duplicates. Sort first, then skip duplicates at the same recursion level: `if (i > start && nums[i] == nums[i-1]) continue;`. This skip-duplicate pattern appears in Combination Sum II (LC 40) and Permutations II (LC 47) — learn the pattern, not just this problem.
- [ ] **LC 46 – Permutations** (Medium) if time allows. Swap-based backtracking: for position i, swap each element from i to n-1 into position i, recurse, swap back. Time O(n × n!). Know this because system-design follow-up often involves "enumerate all orderings" for small inputs.

**Core Block — STAR Story #4: Biggest Failure & Lesson (30 min)**
- [ ] This is the hardest story to tell well. Interviewers distinguish genuine self-awareness from performance.
  - **What NOT to do**: "My biggest failure was working too hard" or "I was TOO detail-oriented." These are transparent non-answers and make you look evasive.
  - **What to do**: Pick something real where YOU made a mistake that had a measurable negative consequence, and where you learned something specific.
  - **Scaffold**:
    - **Situation**: [A real project moment where you misjudged something — scope, technical approach, timeline, communication.]
    - **Task**: What were you trying to achieve?
    - **Action that led to failure**: "I assumed X without validating it. / I didn't communicate Y to the team. / I optimised for Z when the real constraint was W."
    - **Consequence**: Be honest. "This caused a 2-day delay / broke the staging environment / required a hotfix / frustrated a stakeholder."
    - **Lesson**: Specific and actionable. "Since then, I always do X before starting Y. / I added a validation step to my workflow. / I now set up alerts for Z before deploying."
    - **Follow-through**: "I applied this lesson in [subsequent project] — here's what changed."
  - Example failure angle (adapt): "During the LLM proxy routing feature, I assumed provider A would have sufficient uptime SLA and didn't build a retry+fallback until after we had a production incident. I learned to design for failure first, not as an afterthought."
- [ ] Speak it aloud. This one should be UNDER 90 seconds — brevity signals confidence and acceptance of the mistake.

**Application Block (20 min)**
- [ ] Research and apply to 5 new target companies. Focus specifically on:
  - GCCs with Java/Spring Boot stacks: Goldman Sachs (Bengaluru), JPMorgan Chase Tech, Walmart Global Tech, Target, Atlassian, Booking.com India, PhonePe, Razorpay, CRED, Zepto, Meesho.
  - For each: read the JD, identify the 2–3 skills they emphasize most, write a tailored 3-sentence application note referencing those skills and one specific achievement from your resume.
- [ ] Log in tracking sheet. Running total should be approaching 20+ applications.

**Self-Check**
- [ ] Can you write the subset-with-duplicates skip pattern from memory?
- [ ] Is your failure story honest enough to feel slightly uncomfortable? (If it doesn't, it's not genuine enough.)

---

### Friday, Aug 28 — DSA: Company-Tagged Review + STAR Stories #5 & #6

📌 **Study today:** Company-tagged review (LC 56, 33, 76, 105 etc.) · STAR story #5 (why leaving) · STAR story #6 (tell me about a time you were wrong)

**DSA (45 min)**
- [ ] Pull up LeetCode's company tags for the 3 companies you have the most active applications at (check your tracking sheet). Look at their Medium/Hard tagged problems. Prioritize:
  - **Atlassian**: Graph problems, String manipulation, Interval problems — commonly tagged: LC 56 (Merge Intervals), LC 57 (Insert Interval), LC 253 (Meeting Rooms II, Premium → use Greedy sort by start+end), LC 1293 (Shortest Path in Grid with Obstacles).
  - **Goldman Sachs / JPMorgan**: Array/String/Matrix, Binary Search variants — LC 33 (Search in Rotated Array), LC 74 (Search a 2D Matrix), LC 4 (Median of Two Sorted Arrays — know the binary search approach even if you only explain it).
  - **Walmart Global Tech / Target**: Sliding window, Two pointers, Trees — LC 76 (Minimum Window Substring), LC 239 (Sliding Window Maximum), LC 105 (Construct Binary Tree from Preorder+Inorder).
- [ ] Solve 2–3 of the tagged problems most relevant to your scheduled/pending interviews. Time yourself: 20 min max per problem.

**Core Block — STAR Stories #5 & #6 (45 min)**

*Story #5: Why are you leaving your current role?*
- [ ] This is NOT a STAR story technically, but it needs the same preparation rigor. It will be asked in every first-round screen.
  - **The honest answer** (frame it as ambition, not escape):
    - "I've been extremely fortunate to build production experience across the full stack — Java microservices, cloud infra, LLM integration, and DevOps. In two and a half years I went from implementing features to owning the architecture of an entire platform. I'm proud of that.
    - What I'm looking for now is scale — working on systems that serve millions of users, a strong engineering culture with code reviews, architecture discussions, and senior engineers to learn from, and a compensation package that reflects the level I'm operating at.
    - Product companies and GCCs represent that combination. I want to solve harder problems at bigger scale."
  - What you must NEVER say: "my current company is bad / chaotic / underpaying / the manager is difficult." Even if true — it makes you sound like a complainer, not an ambitious engineer.
  - Speak this answer aloud 3× until it sounds natural, not rehearsed.

*Story #6: Behavioral Curveball — "Tell me about a time you were wrong"*
- [ ] This is the conflict story's harder cousin. It requires genuine intellectual humility.
  - **Scaffold**:
    - **Situation**: A technical decision or assumption you made confidently that turned out to be incorrect.
    - **What you said/believed**: Be specific. "I argued that synchronous REST calls would be fine for the user management migration because our current load was under X requests/sec."
    - **How you discovered you were wrong**: "After the migration, we saw cascading failures during peak load — the downstream service's latency spiked and our synchronous calls queued up. I had not accounted for bursty traffic patterns."
    - **What you did**: "I immediately acknowledged the misjudgement to the team. We implemented a circuit breaker + moved to async event-driven communication — the same change I had initially dismissed as over-engineering."
    - **Lesson**: "I now simulate bursty load in load tests before signing off on any synchronous integration. Steady-state benchmarks don't reveal queueing behaviour."
  - Key: the story must show you change your mind based on evidence, not stubbornness. That is a high-signal answer for senior roles.
- [ ] Speak story #6 aloud. Time it: 75–90 seconds.

**Self-Check**
- [ ] Do you have all 6 stories written in a notes doc (or index cards) you can review before interviews?
- [ ] Did you solve at least 2 company-tagged problems for your most-prioritized active application?

---

### Saturday, Aug 29 — Full Mock Interview + Spring/Java Deep-Dive Drill

📌 **Study today:** Timed graph/backtracking mock (LC 127, 130, 417, 79, 131, 51) · Spring/Java rapid-fire drill (auto-config, bean lifecycle, @Transactional, Resilience4j, JWT, RLS) · applications

**DSA (60 min)**
- [ ] Timed mock: open a fresh LeetCode session. Pick problems RANDOMLY from this list (don't cherry-pick):
  - LC 127 – Word Ladder (Hard, BFS)
  - LC 130 – Surrounded Regions (Medium, DFS/Union-Find)
  - LC 417 – Pacific Atlantic Water Flow (Medium, BFS/DFS)
  - LC 79 – Word Search (Medium, Backtracking)
  - LC 131 – Palindrome Partitioning (Medium, Backtracking + DP)
  - LC 51 – N-Queens (Hard, Backtracking)
- [ ] Pick 2, solve them in 30 min each, no hints. After each: analyze time/space complexity aloud, then check your solution against the editorial. Note the gap.

**Spring/Java Deep-Dive Drill (90 min)**

This session revisits the interview-qa.md topics as a rapid-fire drill — you answer each Q aloud in < 90 seconds before checking your notes. Track which ones you hesitate on.

*Group A — Spring Internals (20 min, answer aloud before checking)*
- [ ] "How does Spring Boot auto-configuration work?" (answer: conditional scanning of AutoConfiguration.imports, @ConditionalOnClass, etc.)
- [ ] "What is the bean lifecycle?" (answer: 8-step sequence from instantiation through @PostConstruct to @PreDestroy)
- [ ] "@Transactional self-invocation — what happens and how do you fix it?"
- [ ] "What is the proxy ordering issue with @Transactional + @Cacheable?"

*Group B — Spring Cloud / Microservices (20 min)*
- [ ] "How does circuit breaker work in Resilience4j? Three states, config properties."
- [ ] "Outbox pattern — why, and what are the two delivery mechanisms?"
- [ ] "CQRS — when is it worth the complexity?"
- [ ] "Saga — choreography vs orchestration — give a real use case for each."

*Group C — Security (15 min)*
- [ ] "How did you cut sign-in latency by 60%?" (answer: stateless JWT validation, eliminated DB roundtrip per request)
- [ ] "JWT risks — name 3 and the mitigation for each."
- [ ] "What is the difference between RBAC and ABAC?" (RBAC: roles; ABAC: attribute-based policy — e.g., user.department == resource.department — more granular but harder to audit)

*Group D — Database/Observability (15 min)*
- [ ] "Row-level security in PostgreSQL — how did you use it in Deep Fathom?"
- [ ] "How does distributed tracing work with Micrometer Tracing in Spring Boot 3.x?"
- [ ] "Name the 5 most useful Actuator endpoints in production and why."

*Group E — Testing (10 min)*
- [ ] "@SpringBootTest vs @WebMvcTest vs @DataJpaTest — when to use each?"
- [ ] "How do you test a Feign client without hitting the real service?"

*Group F — Curveball wrap-up (10 min)*
- [ ] "If a pod passes liveness but fails readiness permanently — what exactly happens?"
- [ ] "Deployment vs StatefulSet — when would Spring Boot ever use a StatefulSet?"
- [ ] "Migrate from Azure to AWS — what changes, what doesn't?"

**Applications (30 min)**
- [ ] Submit 3–5 more applications. Running total by end of Saturday: aim for 20–25 total applications since Week 8.
- [ ] For any company that reached out for a screen: confirm the time, research the interviewer on LinkedIn, add prep notes in the tracking sheet (tech stack, product, recent engineering blog posts).

**Self-Check**
- [ ] Which Spring/Java group had the most hesitations? Schedule a 30-min review for that group Sunday morning.
- [ ] Is your STAR stories doc complete with all 6 stories written out?

---

### Sunday, Aug 30 — Full End-to-End Mock Interview + Week 11 Planning

📌 **Study today:** Full end-to-end mock — behavioral + News Feed/Timeline HLD (fan-out write/read/hybrid, cursor pagination, ranking) + coding · Week 11 planning

**Morning: Targeted Weak-Area Review (60 min)**
- [ ] Take the group with the most hesitations from Saturday's drill. Re-read the relevant interview-qa.md section, then answer the same questions aloud again — measure improvement.
- [ ] Solve 1 DSA problem from a pattern you rated < 4 in your Week 9 self-assessment. No hints, timed.

**Full Mock Interview (90 min)**

Simulate a real first-round + technical interview from a GCC (45 min behavioral + system design, 45 min coding). Record yourself on your phone or use a friend.

*Behavioral segment (20 min)*
- [ ] Interviewer: "Tell me about yourself." → Deliver your 90-second pitch.
- [ ] Interviewer: "Tell me about your most impactful technical achievement." → Performance win story.
- [ ] Interviewer: "Tell me about a disagreement with a teammate." → Conflict story.
- [ ] Interviewer: "Tell me about a time you failed." → Failure story.
- [ ] Interviewer: "Why are you looking to leave?" → Why-leaving answer.
- [ ] Interviewer: "Why this company?" → [Research 1 real company on your list and give a specific answer.]

*System Design segment (25 min)*
- [ ] Design a "notification service" for a product like Flipkart that sends email, SMS, and push notifications. Cover:
  - High-level components: API, queue (Kafka/SQS), per-channel workers, retry logic, deduplication.
  - Tie to your experience: "In Deep Fathom I moved to event-driven communication to decouple the notification service from the authorization service — same pattern here."
  - Failure modes first: what if the email provider goes down? (dead-letter queue + fallback provider)
  - Scale numbers: 10 million notifications/day = ~116/sec average, ~1000/sec peak. Single Kafka partition handles ~10k msg/sec — easily covered.
  - Database schema: `notifications(id, user_id, channel, payload, status, created_at, sent_at, retries)`.

*System Design segment — Driven HLD: Design a News Feed / Timeline (45 min)*

This is the single highest-frequency HLD prompt at product companies and GCCs — drive it end-to-end, do not wait for prompts. Whiteboard it on paper, narrate aloud. Prompt: *"Design the home timeline / news feed for a social product — 100M users, a user follows up to a few thousand accounts, feed must load in under 200ms."*

- [ ] **Requirements + scale (5 min):** read-heavy (feed views ≫ posts), ~100M DAU, avg user follows ~500, reads at ~10:1 vs writes. The defining tension is **write amplification vs read amplification** — state it up front, it frames the whole design.
- [ ] **The core decision — fan-out-on-write vs fan-out-on-read:**
  - *Fan-out-on-write (push):* when a user posts, write that post's id into every follower's precomputed feed (a per-user feed cache, e.g. a Redis sorted set keyed by user, scored by timestamp/rank). Reads are O(1) — just slice the cache. Cost: a post by someone with 1M followers triggers 1M writes (write amplification explodes for high-follower accounts).
  - *Fan-out-on-read (pull):* store posts once; at read time, fetch the posts of everyone the user follows and merge-sort them. Writes are cheap; reads are expensive (gather + merge across hundreds of authors on every feed load — read amplification).
- [ ] **The hybrid (this is the answer interviewers want):** push for normal users; for celebrity / high-follower accounts (above a follower threshold), do NOT fan out on write — instead pull their recent posts at read time and merge them into the precomputed feed. This caps write amplification (one Justin-Bieber post no longer does 100M writes) while keeping reads fast for the 99% case. Name the threshold as a tunable knob.
- [ ] **Feed cache:** per-user feed stored as a Redis sorted set (post_id → score). Cap it (e.g. top ~800 entries) — nobody scrolls infinitely; older history is regenerated on demand from the posts store. Cold/inactive users: don't maintain a live feed at all, generate lazily on next login (avoids fanning out to dormant accounts).
- [ ] **Cursor-based pagination:** never `OFFSET/LIMIT` (skips/dupes when new posts arrive mid-scroll). Use an opaque cursor = `(score, post_id)` of the last item seen; the next page is "items with score < cursor." Stable under concurrent writes — this is the correct pattern and a clean signal.
- [ ] **Ranking signals:** chronological is the baseline; ranked feed scores each candidate post by recency, author affinity (interaction history), engagement velocity (likes/comments rate), and content type. Separate **candidate generation** (fan-out gives the candidate set) from **ranking** (a scorer orders them) — naming this two-stage split is the senior signal.
- [ ] **Tie to your work:** "The fan-out-on-write step is exactly the outbox/Kafka pattern I know from Deep Fathom — the post-creation transaction emits a `PostCreated` event to Kafka, and feed-builder workers consume it to update follower feed caches asynchronously. Decoupling the write path from fan-out via a queue is the same decoupling I did for the notification service." Also tie the per-user feed cache to your Smart360 Redis caching work.
- [ ] **Self-critique rubric:** Did I open with the write/read amplification trade-off, explicitly justify the hybrid for celebrity accounts, choose cursor (not offset) pagination, separate candidate-generation from ranking, and name at least one failure mode (e.g. fan-out worker lag → feed staleness, mitigated by also pulling the author's very latest posts at read time)? If any of these was missing or prompted, mark it for a redo.

*Coding segment (45 min)*
- [ ] Pick a random Medium from a company tag. 45 min, simulate no-hints.
- [ ] After coding: explain time and space complexity unprompted. Mention an alternative approach. Ask the interviewer "Is there a follow-up you'd like to explore?" — this is an extraordinary-candidate move.

**Review Mock (15 min)**
- [ ] Watch back the recording (if made) or reflect:
  - Were there filler words ("um", "like", "so basically") — these undermine perceived confidence. Note specific points to tighten.
  - Did you state numbers in every behavioral answer?
  - Did the system design have explicit failure modes and scaling estimates?
  - Did you explain DSA complexity unprompted?

**Week 11 Planning (30 min)**
- [ ] Update tracking sheet: any new interview invitations? Schedule prep sessions.
- [ ] Rate yourself on this week's self-assessment table (below).
- [ ] Identify the top 3 gaps → write them as Day 1–2 tasks for Week 11.
- [ ] Write 2–3 things you learned this week that you couldn't have articulated last Sunday.

---

## 🧠 Concepts to Master This Week

### Graph Patterns — Decision Tree

```
Is the graph a GRID (2D matrix)?
  → DFS/BFS flood-fill. Use int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}}.

Does the problem ask for SHORTEST PATH (unweighted)?
  → BFS (layer-by-layer expansion guarantees shortest path).
  → Dijkstra if weighted edges.

Does the problem ask about CONNECTIVITY ONLY (are X and Y connected)?
  → Union-Find. Faster than BFS for repeated connectivity queries. Near O(1) per query.

Does the problem involve ORDERING with DEPENDENCIES (course prereqs, build order)?
  → Topological Sort. Use Kahn's (BFS with in-degrees) for iterative, DFS post-order for recursive.
  → If cycle detection is the goal: DFS with 3-color marking OR Kahn's count check.

Does the problem involve CYCLE DETECTION in undirected graph?
  → Union-Find: if union(u,v) finds they're already in same set → cycle exists.
```

**Union-Find Template (memorise this exactly):**
```java
int[] parent, rank;

void init(int n) {
    parent = new int[n];
    rank = new int[n];
    for (int i = 0; i < n; i++) parent[i] = i;
}

int find(int x) {
    if (parent[x] != x) parent[x] = find(parent[x]); // path compression
    return parent[x];
}

boolean union(int x, int y) {
    int px = find(x), py = find(y);
    if (px == py) return false; // already connected — cycle if called on edge
    if (rank[px] < rank[py]) { int t = px; px = py; py = t; }
    parent[py] = px;
    if (rank[px] == rank[py]) rank[px]++;
    return true;
}
```

**Backtracking Template — the 3-part structure:**
```java
void backtrack(int start, List<Integer> current, int[] nums, List<List<Integer>> result) {
    result.add(new ArrayList<>(current));        // 1. Record (or check exit condition)
    for (int i = start; i < nums.length; i++) {
        if (i > start && nums[i] == nums[i-1]) continue; // 2. Skip duplicates (if needed, array must be sorted)
        current.add(nums[i]);                    // 3. Choose
        backtrack(i + 1, current, nums, result); // 4. Recurse
        current.remove(current.size() - 1);      // 5. Undo (backtrack)
    }
}
```
Know this pattern cold. It is the skeleton of LC 78, LC 90, LC 39, LC 40, LC 46, LC 131, LC 51.

### Behavioral Prep — The 10 STAR Stories (6 core + 4 bar-raiser)

Rows 1–6 are the core stories drilled Mon–Fri; rows 7–10 are the bar-raiser / values-round scaffolds detailed below (write them as real anecdotes).

| # | Story Type | Must Include | Common Curveball Version |
|---|---|---|---|
| 1 | Performance win | 96% latency reduction numbers, specific technical steps (N+1 diagnosis, JOIN FETCH, EntityGraph, index, Redis) | "Walk me through the query profiling — what tool did you use?" |
| 2 | Conflict / disagreement | A specific disagreement, your counter-evidence, the outcome, relationship intact | "Tell me about a time you disagreed with your manager specifically." |
| 3 | Ownership / leadership | Proactive identification, no one asked you to fix it, metric outcome | "How do you handle a situation where the fix is outside your scope?" |
| 4 | Biggest failure | A REAL failure, specific consequence, specific lesson with follow-through | "Tell me about a time you were wrong about a technical decision." |
| 5 | Why leaving | Ambition framing, no negativity, growth + scale + compensation | "Were there things you tried to change but couldn't?" |
| 6 | Tell me about yourself | 90-second pitch: current role → key achievements (numbers) → what you're looking for | "If I called your current manager right now, what would they say about you?" |
| 7 | Mentorship / helping a teammate | Taught the *why* (didn't just fix it), left a runbook/doc, followed up; quantified ramp/outcome | "Tell me about a time you helped someone grow." |
| 8 | Receiving tough feedback | Someone told you something hard; no defensiveness; a concrete visible change; closed the loop | "What's the most useful piece of feedback you've ever received?" |
| 9 | Deadline / prioritization under pressure | Explicit must-have vs nice-to-have; cut by reversibility & user impact; communicated it; hit the date | "How do you decide what to work on when everything is urgent?" |
| 10 | Customer / stakeholder impact | User/business value first, engineering metric second; started from user pain; validated against the user outcome | "Tell me about a time you went above and beyond for a customer." |

**The 90-second "Tell Me About Yourself" Script (fill in real details, practise until natural):**

> "I'm a full-stack software engineer with two and a half years of experience building Java-based microservices and cloud infrastructure at scale. At [current company], I've led or contributed to two main products: Smart360, where I reduced data retrieval latency from 60 seconds to under 3 seconds by fixing N+1 query patterns and adding a Redis caching layer — and Deep Fathom, an LLM-powered platform where I designed the multi-service architecture on Azure, wrote the Bicep IaC for 30-plus resources, cut CI/CD pipeline time by 57%, and built the async job queue that handles long-running LLM calls across five different AI providers.
>
> My stack is Java 17, Spring Boot, Spring Cloud, PostgreSQL, Redis, Docker, Kubernetes, and Azure — with hands-on AWS experience from Smart360. I also integrated JWT and OAuth2 authentication, reducing sign-in latency by 60% by moving to stateless token validation.
>
> I'm looking to take that foundation into a product company or GCC where I can work at larger scale, contribute to a strong engineering culture, and keep growing as an engineer. That's why I'm excited about the work your team is doing on [specific thing]."

### Behavioral Prep — Bar-Raiser STAR Scaffolds (4 high-frequency stories most candidates skip)

The 6 stories above cover the core. These 4 are the bar-raiser / values rounds — especially at product companies that test "customer obsession," "earns trust," and "develops others." Write each as a real anecdote; the scaffold below is a frame, not a script. **Mark each `[INSERT REAL ANECDOTE]` honestly — a fabricated story collapses under one follow-up question.**

*Story #7: Mentorship / Helping a Teammate*
- [ ] Signal being tested: do you make the people around you better (a senior-track signal at 2.5 YOE)?
  - **Situation**: `[INSERT REAL ANECDOTE — e.g., a junior dev or new joiner onboarded onto Smart360 was blocked for days on the multi-tenant data flow / N+1 debugging / the Bicep IaC setup.]`
  - **Task**: Get them productive without simply taking the work over yourself.
  - **Action**: (1) Sat with them and reproduced the problem together rather than handing a fix. (2) Walked them through the *why* — e.g., how tenant isolation flows through the request, or how to read Hibernate statistics to spot N+1 themselves. (3) Wrote a short runbook / README section so the next joiner wouldn't hit the same wall. (4) Did a follow-up check-in a few days later, not a one-and-done.
  - **Result**: Quantify — "they shipped their first feature within `[X days]` instead of staying blocked; the onboarding doc cut ramp time for the next hire" or "they later handled `[similar task]` independently." Teaching-to-fish beats fixing — say that explicitly.
  - **Curveball version**: "Tell me about a time you helped someone grow." Same structure; emphasise the lasting capability you left behind, not the single unblock.

*Story #8: Receiving Tough / Critical Feedback*
- [ ] Signal being tested: are you coachable? (Distinct from "a time you were wrong" — there you caught your own mistake; here *someone told you* something hard and you acted on it.)
  - **Situation**: `[INSERT REAL ANECDOTE — e.g., in a code review or 1:1, a senior engineer or lead told you your PRs were too large to review safely / you over-engineered the LLM routing abstraction / you weren't communicating blockers early enough.]`
  - **Task**: Take the feedback seriously without getting defensive, and actually change.
  - **Action**: (1) Listened fully and asked a clarifying question to make sure you understood the *specific* behaviour, not a vague impression. (2) Acknowledged it openly rather than justifying. (3) Made a concrete, visible change — "I started splitting work into PRs under ~400 lines and posting a daily blocker note in standup." (4) Closed the loop: checked back with the person to confirm the change landed.
  - **Result**: "Review turnaround on my PRs dropped from `[X]` to `[Y]`" or "the lead noted the improvement in my next 1:1." The lesson must show you treat feedback as data, not criticism.
  - **Curveball version**: "What's the most useful piece of feedback you've ever received?" — pick the same moment, lead with the change it produced.

*Story #9: Deadline / Prioritization Under Pressure*
- [ ] Signal being tested: judgement under constraints — can you cut scope deliberately rather than blindly grinding or shipping everything half-done?
  - **Situation**: `[INSERT REAL ANECDOTE — e.g., a Deep Fathom release / demo with a hard date and two competing must-dos: the async LLM job queue had to be stable AND the CI/CD migration was mid-flight, with not enough time for both at full polish.]`
  - **Task**: Hit the date without shipping something broken.
  - **Action**: (1) Made the trade-off explicit instead of silently choosing — listed what was must-have for the date vs nice-to-have. (2) Negotiated with `[lead/stakeholder]` on what could slip: "I proposed shipping the job queue with a single-provider fallback now and deferring the multi-provider routing polish by one sprint." (3) Cut the deferred work cleanly (feature-flagged / documented as a known follow-up) rather than leaving it half-wired. (4) Communicated the decision and the reasoning to everyone affected.
  - **Result**: "Shipped on the date with the critical path stable; the deferred item landed the following sprint with zero customer impact." Name the *principle*: cut by reversibility and user impact, not by what's easiest to drop.
  - **Curveball version**: "How do you decide what to work on when everything is urgent?" — describe the must-have/nice-to-have + reversibility framing as your default, then ground it in this story.

*Story #10: Customer / Stakeholder Impact*
- [ ] Signal being tested: customer obsession — do you connect engineering work to end-user or business value, not just internal metrics? (This is the highest-weight signal at product companies.)
  - **Situation**: `[INSERT REAL ANECDOTE — e.g., Smart360 end users (reviewers/tenants) were abandoning dashboards because they timed out at 60s, OR a Deep Fathom B2B customer flagged that long-running LLM jobs appeared to "hang" with no feedback.]`
  - **Task**: Solve it framed around the *user's* experience, not just the technical symptom.
  - **Action**: (1) Started from the user's pain, not the stack trace — "I sat with the support tickets / talked to `[stakeholder]` to understand what 'slow' actually cost them." (2) Translated it to the technical fix (the N+1 + Redis caching work, or surfacing async job status so the UI showed progress instead of a spinner). (3) Validated the fix against the user-facing outcome, not just the server metric.
  - **Result**: Lead with the *user/business* number — "dashboards that were timing out now load in under 3 seconds, and `[the abandonment / support-ticket volume / customer escalation]` dropped" — then back it with the engineering metric (96% latency reduction, 80% fewer S3 calls). The order matters: customer impact first, engineering detail second.
  - **Curveball version**: "Tell me about a time you went above and beyond for a customer." — emphasise that you chose to dig into the user's actual workflow rather than closing the ticket at the minimum bar.

### Spring/Java — Curveball Answers Synthesised From interview-qa.md

**"@Transactional + @Cacheable — what can go wrong?"**
Both are AOP proxies. Default proxy order: `@Cacheable` wraps `@Transactional`. If cache is populated inside the transaction, a rolled-back transaction can leave stale cache entries. Fix: use `@TransactionalEventListener(phase = AFTER_COMMIT)` to populate or evict cache only after successful commit.

**"Why can't you put @Transactional on a private method?"**
Spring AOP uses a proxy (JDK dynamic proxy or CGLIB subclass). The proxy intercepts calls at the class boundary. Private methods are not part of the proxy's overridable surface — calls go directly to the real object. Spring silently ignores `@Transactional` on private methods. Same reason self-invocation bypasses the proxy.

**"What is the difference between RBAC and ABAC?"**
RBAC: user has roles; roles have permissions. Simple, auditable, good for most enterprise apps. ABAC: access policy is a function of user attributes + resource attributes + environment (time, IP, etc.). More granular — e.g., "a user can edit a document only if user.department == document.department AND time is business hours." More powerful but harder to audit. In Spring, RBAC via `@PreAuthorize("hasRole('ADMIN')")`. ABAC via SpEL expressions in `@PreAuthorize` or a custom `PermissionEvaluator`.

**"How does @Async work and what's the common mistake?"**
`@Async` wraps the method call in a proxy that submits the task to a `TaskExecutor` (thread pool). Common mistakes: (1) calling an `@Async` method from the same class — proxy bypassed, runs synchronously. (2) Not configuring a custom executor — defaults to `SimpleAsyncTaskExecutor` which creates a new thread per call (no pooling). (3) Not handling `Future`/`CompletableFuture` return properly — exceptions swallowed if not explicitly handled.

---

## 🎤 Sample Interview Questions (incl. Curveballs) — 17 Qs + Pointers

### Behavioral

**1. "Tell me about yourself."**
- Pointer: Use the 90-second script above. Do not recite your CV chronologically — that's what weak candidates do. Lead with the biggest outcomes (60s→3s, 57% CI/CD cut), then your stack, then what you're looking for. End with a bridge to this company specifically. Never exceed 2 minutes.

**2. "Tell me about a time you significantly improved system performance."**
- Pointer: Story #1. Lead with the outcome: "I reduced a 60-second data retrieval down to under 3 seconds." Then walk through the diagnosis (Hibernate statistics → N+1 found → specific fix per layer). Mention each layer: query rewrite, index, Redis caching, S3 URL caching. Use "I" not "we" for actions you personally took.

**3. "Tell me about a time you disagreed with someone on your team."**
- Pointer: Story #2. Key: show you disagree with data, not opinion. "I prepared a comparison doc with two approaches, load test numbers, and maintenance implications." Show you can be wrong gracefully too — "They raised a valid point about X that I hadn't considered, which changed my view on Y."

**4. Curveball: "Tell me about a time you had to make a decision without enough information."**
- Pointer: Frame it as structured ambiguity resolution. "I broke the unknown into what I could learn quickly vs what I'd have to assume, documented my assumptions explicitly, chose the most reversible option, and set a checkpoint to revisit after we had more data." Tie to the LLM proxy routing project — routing logic decisions under uncertainty about provider uptime SLAs.

**5. Curveball: "How do you handle a situation where you're asked to do something you don't agree with technically?"**
- Pointer: Two-part answer. Part 1: "I raise my concern once, clearly, with a specific technical reason and evidence." Part 2: "If the decision is still made to proceed, I commit fully and execute the plan — but I document my concern and propose a checkpoint to evaluate the outcome." This shows both backbone and professionalism. Never say "I just do what I'm told."

**6. "Why do you want to work at [Company]?"**
- Pointer: This is the most underprepared question by most candidates. Research BEFORE the interview: the company's engineering blog, recent tech stack decisions (from job posts or GitHub), product direction. Then make it specific: "I noticed your team recently open-sourced X / wrote about migrating to Y / is building in the Z space. That aligns with [what I built] and I'd contribute by [specific thing]." Generic answers ("great culture, innovative products") are filtered out. Specificity is the signal.

**6a. "What is your greatest strength, and what is your greatest weakness?"**
- Pointer: For strength, pick ONE that maps to a story you can prove with a number — "I'm strongest at turning a vague performance problem into a measured fix" → bridge straight into the 60s→3s story. Don't list five strengths; depth over breadth. For weakness, the trap is the fake weakness ("I work too hard", "I'm a perfectionist") — interviewers hear it as evasion. Give a REAL, low-blast-radius weakness with active remediation: e.g., "I used to delay asking for help because I wanted to solve things myself — which once turned a half-day blocker into two days. I now timebox to 45 minutes before pulling in a teammate, and I post blockers in standup the same morning." The structure is: real weakness → the concrete cost it once had → the specific system you put in place so it doesn't recur. A weakness you've actively engineered around is a strength signal.

**6b. Curveball: "Why two companies / so many projects in just 2.5 years?"**
- Pointer: This can read as a stability concern — reframe it as breadth and ownership, never as restlessness. The honest framing: "Within `[current company]` I worked across three products — Smart360, a multi-tenant review platform; Deep Fathom, a B2B SaaS on Azure; and WebX, an LLM-integrated product. That wasn't job-hopping — it was the company trusting me with new problems as I delivered. It's why I have hands-on range across the full stack, cloud infra, and LLM integration that a single long-running project wouldn't have given me." Land the close on intent: "What I want next is to go deeper at scale on one product, which is exactly the move I'm making now." If the two companies are literally two employers, frame the first as foundational and the second as the deliberate step up in scope — a trajectory, not churn. Never sound defensive; breadth at 2.5 YOE is an asset when you frame it as range plus the judgement of someone who's seen more than one system end-to-end.

### Java / Spring / System Design

**7. "Your circuit breaker opened. Half-Open let one request through and it failed. What happens?"**
- Pointer: The circuit stays Open (or resets the wait timer depending on config). In Resilience4j, `waitDurationInOpenState` resets and you wait again before the next Half-Open probe. The circuit only closes when a configurable number of probe calls in Half-Open state succeed (`permittedNumberOfCallsInHalfOpenState`). Important: Resilience4j allows you to configure the minimum number of Half-Open calls that must succeed before closing — know this config property (`minimumNumberOfCalls` for half-open). The fallback method executes for ALL calls while Open and for failed Half-Open calls.

**8. "Design a rate limiter for an API gateway. What are the options and their trade-offs?"**
- Pointer: Three algorithms: (1) Fixed window counter — simple, suffers from boundary burst (100 requests at 11:59 + 100 at 12:01 = 200 in 2 sec despite "100/min" limit). (2) Sliding window log — accurate but memory-intensive (store every request timestamp). (3) Token bucket — tokens accumulate at a rate; request consumes a token; allows controlled bursting. (4) Leaky bucket — queue-based, smooth output rate. In Spring Cloud Gateway: `RequestRateLimiter` filter backed by Redis uses token bucket. Tie to your Deep Fathom API Gateway — "we used RequestRateLimiter with Redis to protect the LLM proxy from token-abuse."

**9. Curveball: "You have a @Cacheable method that caches a DB result. Another method updates the DB. How do you keep the cache consistent?"**
- Pointer: Use `@CacheEvict(value = "users", key = "#user.id")` on the update method — evicts the entry on write. Or `@CachePut` to update the cache entry in place (fetches the new value and puts it). Key consideration: evict is simpler but causes a cache miss on next read (brief DB hit); `@CachePut` keeps cache warm but couples the write method to cache management. For multi-instance deployments: cache eviction on one instance doesn't propagate to others — use Redis (distributed cache) not in-memory (Caffeine) for shared-state caching. This is a real production issue.

**10. Curveball: "Explain eventual consistency. Give me an example where it's acceptable and one where it's not."**
- Pointer: Eventual consistency: after a write, replicas converge to the same value eventually, but may return stale data in the interim. Acceptable: social media like count (a few seconds stale is fine), product catalog prices (can tolerate brief inconsistency), notification delivery (delay acceptable). NOT acceptable: financial transactions (double-spend must be impossible), inventory deduction in a flash sale (must be strongly consistent to prevent overselling), authentication token validation after logout (must invalidate immediately). Your Redis token blacklist in Smart360 is an example of adding strong consistency to an otherwise stateless JWT system — mention it.

**11. "How does Kubernetes handle a pod that's consuming too much memory?"**
- Pointer: If a container exceeds its `resources.limits.memory`, the kernel OOM killer terminates the container process. Kubernetes sees the container exit and restarts it (if `restartPolicy: Always`, the default). The pod stays Running but its restart count increments. If it keeps OOMKilling rapidly, it enters `CrashLoopBackOff` with exponential backoff. Detection: `kubectl describe pod <name>` shows `OOMKilled` in the last State section. Fix: (1) increase memory limit if the workload legitimately needs it, (2) find the memory leak in the app (heap dumps, JVM flags `-Xmx`), (3) enable JVM native memory tracking. For Spring Boot: set `JAVA_OPTS="-Xmx512m -Xms256m"` and configure `resources.limits.memory` slightly above the JVM max heap + non-heap overhead (~50–100 MB overhead).

**12. Curveball: "What is a CompletableFuture and how is it different from a Future in Java?"**
- Pointer: `Future` is a handle to an async computation — you can check `isDone()` or call `get()` (which blocks). No composition, no callbacks. `CompletableFuture` extends `Future` and adds: `thenApply` (transform result), `thenCompose` (chain async operations), `thenCombine` (merge two futures), `exceptionally` (handle errors), `allOf`/`anyOf` (wait for multiple). Non-blocking composition is the key difference. In the LLM proxy: used `CompletableFuture.supplyAsync(() -> callProvider(req), executor)` to submit LLM calls to a thread pool and chain `.thenApply(response -> processResult(response)).exceptionally(e -> fallbackResult(e))`.

### Graph / Backtracking

**13. "Given a list of prerequisites, find a valid course ordering (topological sort). What if there's a cycle?"**
- Pointer: Kahn's algorithm. Build in-degree array. Add all 0-in-degree nodes to queue. Process queue: for each node, decrement in-degree of its neighbors; if neighbor's in-degree becomes 0, add to queue. If processed count < n → cycle exists, no valid ordering. Output the processing order. Time O(V+E). Connect to real usage: "Kubernetes pod startup order with init containers is essentially topological sort — init containers must complete before main container starts."

**14. Curveball: "You're solving LC 79 – Word Search (backtracking on a grid). It's too slow. What are the options to speed it up?"**
- Pointer: Word Search is O(m × n × 4^L) where L = word length. Optimizations: (1) Early termination — if remaining characters in word > remaining unvisited cells, return false. (2) Character frequency pruning — if the grid doesn't contain enough of a required character (e.g., word needs 3 'Z's but grid has only 1), return false immediately. (3) If searching for MULTIPLE words (LC 212 – Word Search II), use a Trie to prune paths that can't match any word — reduces redundant traversal dramatically. This shows you understand problem variations and optimization strategies, not just base solutions.

**15. Curveball: "You've built multiple microservices. What would you do differently if you were starting over?"**
- Pointer: This tests intellectual honesty and maturity. Don't say "nothing." Good candidates say: "I'd invest earlier in contract testing between services (Spring Cloud Contract or Pact) — we discovered integration mismatches in staging that could have been caught at CI time. I'd also define the observability stack (tracing, structured logs, metrics dashboards) before the first service goes to production, not as a retrofit. And I'd be more conservative about the number of services initially — start with 2–3 bounded contexts, not 5+, and split further only when a specific scaling or deployment need justifies it. Splitting too early creates coordination overhead before you understand the domain boundaries well enough."

---

## 🌟 Extraordinary-Candidate Edge

**Behavioral signals that separate top candidates:**

1. **Every answer has a number.** Not "we improved performance significantly" — "latency dropped from 60 seconds to 2–3 seconds, a 96% reduction, and S3 API calls dropped by 80%." Metrics prove ownership. Vague statements prove you watched someone else do it.

2. **Failures are specific and honest.** The candidate who says "my biggest failure was not documenting my code well enough" gets an internal eye-roll. The candidate who says "I shipped a synchronous integration without load-testing it under bursty conditions, it caused cascading failures in staging, and I now treat bursty-load simulation as a pre-release checklist item" gets genuine respect.

3. **"Why this company" is researched.** Looking up the company's engineering blog or recent job posts and saying "I noticed you're migrating to X / building in Y / recently wrote about Z" signals a candidate who does homework. It differentiates immediately from the candidates who say "I love your culture and innovation."

4. **Technical answers connect back to YOUR experience.** When asked about topological sort, don't just explain the algorithm — say "this is the same pattern as Spring's bean initialization order, which is resolved by the BeanDefinitionRegistry using a dependency graph." When asked about rate limiting, reference your API Gateway work in Deep Fathom. This shows you have application experience, not just textbook knowledge.

5. **Handling curveballs:** When you don't know the answer, say "I haven't hit that exact scenario — let me reason through it." Then reason out loud, systematically. Interviewers care more about HOW you think under uncertainty than whether you have the answer memorised. The worst thing: going silent, or bluffing.

**Depth signals to drop naturally in Week 10 interviews:**

- When discussing Graphs: mention that Union-Find's near-O(1) amortized complexity comes from path compression + union by rank together — either alone is insufficient.
- When discussing Kubernetes: mention that `terminationGracePeriodSeconds` on the Pod spec must exceed `spring.lifecycle.timeout-per-shutdown-phase` — if it doesn't, K8s kills the process before Spring finishes draining.
- When discussing caching: mention that `@Cacheable` in a multi-instance deployment requires a distributed cache (Redis, not Caffeine/in-memory) — otherwise each instance has its own stale cache.
- When discussing `@Transactional`: the proxy is CGLIB by default in Spring Boot (subclass proxy) because the class may not implement an interface — but if it does implement an interface, Spring can use a JDK dynamic proxy. This is a real subtlety that most developers don't articulate.
- When asked about LLM integration: distinguish between latency-sensitive tasks (streaming SSE, need low TTFB) vs. throughput-sensitive tasks (batch job, optimize for total cost). The routing logic in your LLM proxy should account for this — route to the fastest provider for interactive queries, cheapest for batch.

**What extraordinary looks like in each round:**

| Round | Ordinary | Extraordinary |
|---|---|---|
| Phone Screen (HR) | Recites resume chronologically | Leads with 2 impact numbers immediately, asks 1 sharp question about the team's tech stack at the end |
| Technical Phone | Solves the problem | Solves it, analyses complexity unprompted, suggests a follow-up variant, mentions an edge case they'd add in production |
| System Design | Draws happy-path architecture | Opens with failure modes and constraints, explicitly names patterns avoided (e.g., "I'm not using 2PC here because..."), quantifies scale estimates |
| Behavioral | Tells a vague story about teamwork | Specific number in every answer, shows self-awareness in the failure story, connects "lesson learned" to a subsequent decision |
| Culture Fit | "I love working in teams" | "I noticed your team contributes to [open source project / tech blog / specific tooling] — I spent time exploring that this week and have a question about your approach to X" |

---

## 📊 End-of-Week Self-Assessment

Rate yourself 1–5 after Sunday's mock. Feed gaps directly into Week 11 Day 1–2 makeup blocks.

| Area | Target | Your Rating | Gap / Next Action |
|---|---|---|---|
| LC 200 – Number of Islands (DFS + BFS both) | 5 | | |
| LC 207 – Course Schedule (Kahn's + DFS) | 5 | | |
| LC 684 – Redundant Connection (Union-Find) | 5 | | |
| LC 78/90 – Subsets (with/without duplicates) | 5 | | |
| LC 46 – Permutations (swap backtracking) | 4 | | |
| Company-tagged problems for top 3 applied companies | 4 | | |
| STAR Story #1 – Performance win (fluent, under 2 min) | 5 | | |
| STAR Story #2 – Conflict / disagreement | 5 | | |
| STAR Story #3 – Ownership / leadership | 5 | | |
| STAR Story #4 – Biggest failure (honest, specific) | 5 | | |
| STAR Story #5 – Why leaving (positive framing) | 5 | | |
| STAR Story #6 – "Tell me about yourself" 90s pitch | 5 | | |
| STAR Story #7 – Mentorship / helping a teammate | 4 | | |
| STAR Story #8 – Receiving tough feedback (coachability) | 4 | | |
| STAR Story #9 – Prioritization under pressure | 4 | | |
| STAR Story #10 – Customer / stakeholder impact (user value first) | 4 | | |
| Curveball: "time you were wrong" | 5 | | |
| Curveball: "how you handle ambiguity" | 4 | | |
| "Why this company?" for 3 specific companies | 5 | | |
| Rate limiter design (3 algorithms + trade-offs) | 4 | | |
| @Cacheable + @Transactional interaction pitfall | 5 | | |
| CompletableFuture vs Future — composition API | 4 | | |
| OOMKilled pod — diagnosis and fix | 4 | | |
| Topological sort — Kahn's algorithm from scratch | 5 | | |
| News Feed / Timeline HLD — fan-out (write/read/hybrid), cursor pagination, ranking | 5 | | |
| 20+ applications submitted + tracking sheet current | 5 | | |
| Full mock interview completed (recorded) | 5 | | |

**Scoring guide:**
- 5: Cold, whiteboard-pressure, no hesitation.
- 4: Solid — minor pause on one edge case.
- 3: Know the concept but stumble under pressure — needs more reps.
- 2: Shaky — revisit source material.
- 1: Not ready — block 2 focused hours in Week 11.

**Week 10 exit criteria (minimum bar before Week 11):**
- All 6 core STAR stories written AND spoken aloud at least 3× each, plus the 4 bar-raiser scaffolds (#7–#10) drafted with real anecdotes.
- Running application total ≥ 20 with tracking sheet current.
- At least 1 full mock interview completed (self-recorded or with a peer).
- DSA: Union-Find template memorised; backtracking template memorised; company-tagged problems for top 3 targets attempted.
- If an interview is already scheduled for next week: research that company's tech stack, read their engineering blog, prepare a "why this company" answer specific to them.

**If you missed the bar:** Do NOT skip into Week 11 with open gaps on STAR stories — behavioral prep compounds. A missing story costs you the offer, not the DSA problem you didn't solve. Prioritise the stories.

---

*Week 10 complete → Week 11 (Aug 31–Sep 6): Live interview execution + advanced system design (design a search system, design a payment service) + Heap/Priority Queue problems + follow-up pipeline management.*
