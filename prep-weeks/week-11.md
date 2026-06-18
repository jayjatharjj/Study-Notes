# Week 11 — Peak + Interview Clustering (Aug 31–Sep 6, 2026)

> Interviews are arriving. Stop building; start performing. Every hour this week is either a rep in front of an interviewer or a simulation of one. DSA sharpness is maintained, not grown. Frontend-backend integration becomes a fluent story. By Sunday your project narratives are so clean you could answer at 6 a.m. with no warm-up.

---

## 🎯 Week Goal

1. Execute confidently in every scheduled interview — arriving researched, calm, and with 3 sharp questions ready for each company.
2. Narrate Smart360, Deep Fathom, and WebX architecture stories in exactly 5 minutes each, with no filler, no notes, and quantified outcomes every time.
3. Fluently explain the frontend-to-backend integration layer: CORS policy, JWT/OAuth2 token flow on the client, API error handling surfaced in the UI, and how you tested it — as a single coherent story.
4. Maintain DSA sharpness with 1–2 targeted problems per day; no new patterns, only reinforcement of company-relevant ones.

---

## ✅ By Sunday you can...

- Walk into any interview, pull out a company-specific angle in your intro, and pivot to your most relevant project within 60 seconds.
- Solve a medium-difficulty sliding window, two-pointer, or tree problem in under 20 minutes, clean, in a shared editor under observation.
- Explain how your Vue.js / React frontend authenticates with your Spring Boot API: token storage, attaching to requests, handling 401/403 responses, and CORS configuration — without referencing notes.
- Describe the WebX LLM proxy architecture end-to-end: async job submission, SSE/polling for status, multi-provider routing, rate limit handling — in 4 minutes.
- State exactly what `Access-Control-Allow-Origin`, `Access-Control-Allow-Credentials`, and preflight OPTIONS requests mean in the context of your API Gateway configuration.
- Answer "tell me about a time the system behaved unexpectedly in production" with a STAR answer that includes the specific diagnostic tools used (`kubectl describe`, `kubectl logs`, Azure Monitor, Hibernate statistics).
- Use the pre-interview checklist from memory before every round this week.

---

## 📅 Daily Checklist (Mon–Sun)

---

### Monday, Aug 31 — DSA Maintenance + CORS & Token Handling

📌 **Study today:** Sliding window (LC 3, 567) · CORS (preflight, credentials) + JWT token handling on the client

**Pre-day check (5 min)**
- [ ] Open the tracking sheet. Any new interview invites since Friday? Schedule them immediately and block calendar time the evening before each for company research.
- [ ] Any interview scheduled today or tomorrow? If yes: complete the Pre-Interview Checklist (bottom of this section) NOW before the DSA block.

**DSA Maintenance (30 min)**
- [ ] Solve **LC 3 – Longest Substring Without Repeating Characters** (Medium). Sliding window with a HashSet or char frequency array. O(n) time, O(min(m,n)) space. Target: clean solution in under 15 min.
- [ ] Solve **LC 567 – Permutation in String** (Medium). Fixed-size sliding window with two frequency arrays or a single diff counter. Target: under 15 min. This pattern appears at Amazon, Google, and most product companies in warm-up rounds.
- [ ] After solving: write the time/space complexity for both without looking it up.

**Core Block (45 min) — CORS + JWT Token Handling on the Client**

*Why this matters for full-stack roles:* Interviewers at product companies and GCCs frequently probe whether you actually understand the browser security model, not just the backend annotation. Being able to explain it end-to-end — including what breaks and why — is an instant signal.

- [ ] Review and be able to explain from memory:

  **CORS (Cross-Origin Resource Sharing)**
  - The browser enforces the same-origin policy. A request from `https://app.smart360.io` to `https://api.smart360.io` is cross-origin (different subdomain). The browser will block the response unless the server includes `Access-Control-Allow-Origin: https://app.smart360.io`.
  - For requests with credentials (cookies or Authorization headers), the server must set `Access-Control-Allow-Credentials: true` AND the client must set `withCredentials: true` (Axios) or `credentials: 'include'` (fetch). Wildcard `*` is not allowed with credentials.
  - Preflight: for non-simple requests (PUT/DELETE, or custom headers like `Authorization`), the browser first sends an OPTIONS request. The server must respond 200 with `Access-Control-Allow-Methods` and `Access-Control-Allow-Headers`.
  - Spring Boot configuration:
    ```java
    @Configuration
    public class CorsConfig implements WebMvcConfigurer {
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/api/**")
                .allowedOrigins("https://app.smart360.io")
                .allowedMethods("GET","POST","PUT","DELETE","OPTIONS")
                .allowedHeaders("Authorization","Content-Type")
                .allowCredentials(true)
                .maxAge(3600); // preflight cache duration
        }
    }
    ```
  - In Spring Cloud Gateway: configure CORS at the gateway level so downstream services don't need individual CORS config.

  **JWT Token Flow on the Client (Vue.js / React)**
  - On login: backend returns `{ accessToken, refreshToken }`. Store access token in memory (JavaScript variable or Vuex/Redux store), NOT in `localStorage` (XSS risk). Store refresh token in an `HttpOnly` Secure cookie (cannot be read by JS — protects against XSS).
  - On every API request: Axios interceptor attaches the token: `config.headers.Authorization = 'Bearer ' + store.state.accessToken`.
  - On 401 response: Axios response interceptor catches it, calls the refresh endpoint (cookie is sent automatically), gets a new access token, stores it in memory, retries the original request. If the refresh also 401s → force logout.
  - On 403 response (authenticated but not authorized): show a "you don't have permission" UI state — do NOT retry; do NOT redirect to login.
  - On network error / 5xx: show a user-friendly error toast; log the error; do NOT expose raw API error messages in the UI.

  **Smart360 specifics you used:**
  - Spring Security JWT filter validates the token on every request at the gateway.
  - The frontend (Vue.js) had a global Axios interceptor for token attachment and refresh logic.
  - CORS was configured at the API Gateway (Spring Cloud Gateway) to allow the Vue frontend origin.

- [ ] Say aloud: "Explain the full JWT flow from login button click to an authenticated API response and back to the UI." Time yourself. Aim for 2.5 minutes, clean.

**Self-Check**
- [ ] Can you explain why `localStorage` is an XSS risk for token storage in one sentence?
- [ ] Can you describe what happens step-by-step when an Axios request gets a 401 mid-session?

---

### Tuesday, Sep 1 — DSA Maintenance + API Error Handling UX & Full-Stack Integration Story

📌 **Study today:** Sliding window (LC 209, 424) · API error contract (@RestControllerAdvice) + full-stack integration story

**Pre-day check (5 min)**
- [ ] Check interview schedule. Apply Pre-Interview Checklist for any round today/tomorrow.

**DSA Maintenance (30 min)**
- [ ] Solve **LC 209 – Minimum Size Subarray Sum** (Medium). Sliding window with two pointers. O(n) time. This appears in Microsoft and Atlassian screens.
- [ ] Revisit **LC 424 – Longest Repeating Character Replacement** (Medium). Sliding window with a character frequency map. Key insight: `windowSize - maxFreq > k` → shrink. If this felt shaky in Week 8/9, redo it from scratch (no hints).
- [ ] Write the complexity for both. State the invariant of the sliding window in one sentence for each problem.

**Core Block (45 min) — API Error Handling UX + Integration Story**

*API Error Contract (what you enforce on the backend):*
- All errors return a consistent JSON body:
  ```json
  {
    "timestamp": "2026-08-30T10:00:00Z",
    "status": 400,
    "error": "Bad Request",
    "message": "email must not be blank",
    "path": "/api/users"
  }
  ```
- Spring Boot: `@RestControllerAdvice` + `@ExceptionHandler` to map domain exceptions to HTTP status codes. Never let unhandled exceptions return stack traces — that's a security leak and a terrible UX.
- Validation errors: `@Valid` on request body + `MethodArgumentNotValidException` handler → 400 with field-level error list.
- Business rule violations: custom exception (e.g., `UserAlreadyExistsException`) → 409 Conflict.
- Downstream service failures: wrap in a clean 502 or 503 with a message the frontend can show, not a raw Feign exception.

*Frontend error handling UX (Vue.js / React):*
- Axios interceptor for global 5xx: show a toast "Something went wrong, please try again."
- 4xx errors that are form-specific: return error detail to the calling component, render inline under the field.
- Loading states: every async call should set `isLoading = true` before the request and `false` in the `finally` block — guaranteed reset even on error.
- Skeleton screens over spinners for content blocks: better perceived performance.

*The integration story (practice saying this aloud):*
> "In Smart360, the Vue.js frontend communicates with the Spring Boot API through an API Gateway. The gateway handles JWT validation, CORS headers, and routing. All API calls go through a global Axios instance with two interceptors: a request interceptor that attaches the Bearer token, and a response interceptor that handles 401s by transparently refreshing the token and retrying the request. On the backend, a `@RestControllerAdvice` ensures every exception — validation failures, business rule violations, downstream errors — maps to a consistent JSON error shape. The frontend consumes this shape uniformly: form-level errors go inline, system errors go to a toast notification. This meant the frontend team never had to guess what shape an error would take."

- [ ] Say this story aloud twice. Second time should be under 90 seconds and flow naturally.
- [ ] Be ready to answer: "What happens if the token refresh endpoint itself fails?" (Answer: catch the 401 from the refresh call, clear the token from memory, redirect to `/login`, show "Session expired" message.)

**Self-Check**
- [ ] Can you describe the `@RestControllerAdvice` pattern and name three exception types it would handle in Smart360?
- [ ] Can you explain the loading state pattern and why `finally` is required (not `then`)?

---

### Wednesday, Sep 2 — DSA Maintenance + System Design: Full-Stack LLM Feature

📌 **Study today:** Trees / BFS (LC 199, 102, 236) · System design — full-stack LLM async job feature (202 Accepted, SSE vs WebSocket, idempotency)

**Pre-day check (5 min)**
- [ ] Check interview schedule. Apply Pre-Interview Checklist for any round today/tomorrow.

**DSA Maintenance (30 min)**
- [ ] Solve **LC 199 – Binary Tree Right Side View** (Medium). BFS level-order traversal, take the last node per level. O(n) time/space. Tree problems appear frequently in first rounds at product companies.
- [ ] Solve **LC 102 – Binary Tree Level Order Traversal** (Medium). Standard BFS with a queue. Ensure you can implement it in under 10 minutes — this is a confidence-builder and sometimes asked as a warm-up.
- [ ] Bonus if time: **LC 236 – LCA of Binary Tree** (Medium). Recursive post-order DFS. Know the O(n) solution cold — it's a favourite curveball after a BFS question.

**Core Block (60 min) — System Design: Full-Stack LLM Feature (End-to-End)**

This is a rehearsal for a system design question framed as: "Design the WebX feature that lets users submit a long document analysis job to an LLM and see results in the UI."

*Architecture to be able to whiteboard:*

```
[Vue/React UI]
   |
   | POST /jobs  (submit request, get jobId back instantly)
   |
[API Gateway — Spring Cloud Gateway]
   |
   | Routes to LLM Orchestration Service
   |
[LLM Orchestration Service]
   | — validates request
   | — writes job record to PostgreSQL (status: PENDING)
   | — publishes to job queue (Redis Streams or RabbitMQ)
   | — returns 202 Accepted { jobId }
   |
[Worker Threads / @Async pool]
   | — picks up job from queue
   | — selects provider (OpenAI / Anthropic / Gemini) based on
   |     cost/capability/rate-limit headroom (tracked in Redis)
   | — calls provider API (can take 2–20 min)
   | — writes result to PostgreSQL, updates status: COMPLETED
   | — publishes completion event
   |
[Status/Result Service]
   | — GET /jobs/{id}/status → polling endpoint
   | — SSE endpoint: GET /jobs/{id}/stream → pushes partial tokens
   |
[Vue/React UI]
   — polls /status every 3s OR subscribes to SSE
   — renders progress bar, then final result
   — handles FAILED status: shows error + retry button
```

*Design decisions to justify:*
- Why 202 Accepted and async? LLM calls can take 20 min — synchronous HTTP times out. Client-server contract: server owns the work, client observes.
- Why Redis for rate-limit tracking? Atomic `INCR` + `EXPIRE` per provider key — distributed, fast, consistent across multiple worker instances.
- Why SSE over WebSocket for streaming? SSE is unidirectional (server→client), HTTP/1.1 compatible, simpler to implement and proxy through the API Gateway. WebSocket is bidirectional — overkill for a read-only stream.
- Why PostgreSQL for job state? Durability. If the worker crashes mid-job, the PENDING record remains and a recovery process can requeue it. Redis alone is volatile (unless AOF is on).
- Idempotency: if the client submits the same request twice (duplicate click), the second POST should return the existing jobId, not create a new job. Enforce with a client-generated idempotency key or a content hash.

- [ ] Draw this architecture on paper without looking at notes. Time: 10 minutes.
- [ ] Say aloud: justify each major design decision in one sentence each.
- [ ] Curveball prep: "What if the LLM provider returns a partial result before rate-limiting you?" — Answer: store partial tokens to PostgreSQL as they stream in, mark job as PARTIAL, client can display what arrived. Resume from last token if the provider supports it; otherwise re-submit with context.

**Self-Check**
- [ ] Can you draw the LLM async job architecture from scratch in under 10 minutes?
- [ ] Can you justify SSE over WebSocket without hesitating?

---

### Thursday, Sep 3 — DSA Maintenance + Smart360 Project Deep Dive Rehearsal

📌 **Study today:** Hard sliding window (LC 76, 438) · Smart360 5-min talk-track + LazyInitializationException / multi-tenancy curveballs

**Pre-day check (5 min)**
- [ ] Check interview schedule. Apply Pre-Interview Checklist for any round today/tomorrow.

**DSA Maintenance (30 min)**
- [ ] Solve **LC 76 – Minimum Window Substring** (Hard). Sliding window with a character need-count map and a formed counter. O(n+m) time. This is the canonical hard sliding window — if you can do this cleanly, all sliding window variants are handled.
- [ ] If LC 76 felt rough, do **LC 438 – Find All Anagrams in a String** (Medium) instead. Same pattern, easier constraints — build up to 76 on Friday.
- [ ] Write the sliding window template on paper: expand right pointer, shrink left pointer when condition met, track the best window. No code — just the English description. This is your mental model.

**Core Block (50 min) — Smart360 Deep Dive**

*5-minute talk-track (rehearse aloud until it takes exactly 5 minutes):*

> "Smart360 is a multi-tenant SaaS analytics platform built for enterprise clients. I joined a team building a microservices backend in Java 17 and Spring Boot, with a Vue.js frontend. My core contributions were in three areas.

> First, performance: when I joined, the primary data retrieval endpoint took 60 seconds and frequently timed out. I profiled it using Hibernate statistics and found N+1 query patterns — Hibernate was issuing dozens of SELECT statements per request. I rewrote the repository layer using JOIN FETCH and EntityGraph to collapse those into single queries, added composite indexes on the most-filtered columns, and integrated Redis caching for repeated queries. I also cached AWS S3 pre-signed URLs in Redis with a TTL just under the URL expiry — this alone cut S3 API calls by 80%. The result: latency dropped to 2–3 seconds, a 96% reduction.

> Second, security: I owned the authentication and authorization layer. We replaced session-based auth with stateless JWT tokens using Spring Security. On the backend, every request is validated at the API Gateway via a JWT filter — no DB roundtrip, which gave us a 60% reduction in sign-in latency. I implemented RBAC at three granularity levels — repository, table, and permission — using Spring Security's method-level annotations and a custom access evaluator.

> Third, architecture: I helped migrate the user management domain from a monolithic service to an event-driven microservices model. We used a strangler-fig approach — extracted the notification boundary first, validated the pattern, then expanded. This yielded a 30% efficiency gain in user lifecycle operations.

> The stack: Java 17, Spring Boot, Spring Cloud Gateway, Spring Security, Hibernate/JPA, PostgreSQL, Redis, AWS S3, Docker, deployed to AWS."

*Curveball prep for Smart360:*
- "How did you test the JWT filter?" → WireMock for downstream calls, `@WebMvcTest` with `MockMvc` and custom JWT tokens generated in test setup. Integration test: `@SpringBootTest` with `TestRestTemplate` and a test user.
- "What was the hardest bug you fixed in Smart360?" → Prepare a specific story: a bug where a Hibernate proxy caused a `LazyInitializationException` outside a transaction (the classic "session is closed" error). Fix: `@Transactional(readOnly = true)` on the service method that serialised the response, or `@EntityGraph` to eager-load the needed association.
- "How did you handle multi-tenancy?" → Tenant ID encoded in the JWT. Every repository query implicitly filtered by tenant ID using a Hibernate filter (`@Filter` annotation) or a query parameter. Row-Level Security in PostgreSQL as a belt-and-suspenders backstop.

- [ ] Say the 5-minute talk-track aloud. Record yourself. Listen back: is every sentence tight? No filler ("basically", "like", "um")?
- [ ] Prepare and say aloud the answer to the `LazyInitializationException` bug curveball.

**Self-Check**
- [ ] Does your Smart360 story land in exactly 4–5 minutes?
- [ ] Can you answer "what was the hardest bug?" with a specific technical story that shows diagnostic process?

---

### Friday, Sep 4 — DSA Maintenance + Deep Fathom + WebX Deep Dive Rehearsal

📌 **Study today:** Re-solve weakest problem of the week + complexity review card · Deep Fathom + WebX talk-tracks (Bicep IaC, CI/CD 57%, async job + SSE) + Key Vault / crashed-worker curveballs

**Pre-day check (5 min)**
- [ ] Check interview schedule. Apply Pre-Interview Checklist for any round today/tomorrow.

**DSA Maintenance (20 min — lighter day)**
- [ ] Revisit whichever problem this week felt least solid. Re-solve from scratch, no hints, 20-minute limit.
- [ ] Write the time/space complexity and one-line invariant for all 5 problems solved Mon–Thu on a single sheet. This is your pre-weekend review card.

**Core Block (60 min) — Deep Fathom + WebX Talk-Tracks**

*Deep Fathom — 5-minute talk-track:*

> "Deep Fathom is a government-adjacent data intelligence platform requiring FedRAMP-aligned security posture. I built and owned the full infrastructure and backend services.

> On the infrastructure side: I provisioned 30+ Azure resources using modular Bicep IaC — Azure Container Apps for services, PostgreSQL Flexible Server for the database, Redis Cache for caching and rate limiting, Key Vault for secrets, Azure Front Door as the CDN/WAF layer, and private endpoints throughout to eliminate public exposure. Each environment — dev, staging, prod, GovCloud — is one parameter file. No environment drift.

> On the CI/CD side: I redesigned the GitLab pipeline from sequential stages to a DAG-based pipeline using the `needs:` keyword. Unit tests, integration tests, and Docker image builds ran in parallel. BuildKit caching with `--cache-from` pointed at the registry — unchanged layers hit cache at 70%+ rate. Result: pipeline went from 23 minutes to 10 minutes, a 57% reduction.

> On the backend: Spring Cloud microservices deployed to Container Apps — API Gateway, Authorization Service, Data Service, Visualization Service. Kubernetes-compatible probes: liveness at `/actuator/health/liveness`, readiness at `/actuator/health/readiness`. Graceful shutdown configured so rolling updates drop zero requests.

> On security: secrets injected via Key Vault references bound to a managed identity — no credentials in code, no credentials in environment variables, audit log of every secret access. PostgreSQL Row-Level Security for tenant isolation across 50+ tables.

> The Azure-to-microservices story, the Bicep infra ownership, and the FedRAMP posture are what make this project distinctive."

*WebX — 4-minute talk-track:*

> "WebX is an LLM orchestration platform I built to address a specific pain point: enterprise clients needed to run long-form document analysis jobs against multiple LLM providers, but synchronous HTTP couldn't handle jobs that take 2–20 minutes.

> The solution: async job architecture. Client submits a POST → gets a jobId and 202 Accepted immediately. The backend writes the job to PostgreSQL and publishes it to a Redis Streams queue. Worker threads, managed by Spring's async executor pool, pick up jobs and route them to one of five LLM providers — OpenAI, Anthropic, Gemini, Cohere, Azure OpenAI — based on cost, capability, and current rate-limit headroom tracked in Redis with atomic INCR/EXPIRE counters.

> For result delivery: clients either poll `GET /jobs/{id}/status` or subscribe to a Server-Sent Events stream at `GET /jobs/{id}/stream` that pushes partial tokens as they arrive from the LLM.

> The routing intelligence meant we could shift load away from a rate-limited provider transparently — no client change required. We also implemented idempotency keys to prevent duplicate job submission on client retry.

> This project is where I got my deepest experience with async job patterns, SSE streaming in Spring Boot, and the operational challenges of calling third-party APIs with unpredictable latency."

- [ ] Say both talk-tracks aloud, back to back, with a 30-second break between. Total should be under 10 minutes.
- [ ] Curveball for Deep Fathom: "How would you handle a Key Vault outage at startup?" → Answer: Azure Key Vault has 99.99% SLA with multi-region redundancy. For startup resilience: configure the Container App to use a Key Vault reference — if Key Vault is unavailable at deploy time, the deployment fails (safe-fail). For runtime: secrets are injected at pod start and cached in memory — a Key Vault outage after startup doesn't affect running pods.
- [ ] Curveball for WebX: "What if a job is in PROCESSING state and the worker pod gets killed?" → Answer: Heartbeat mechanism. Worker updates a `last_heartbeat` timestamp every 30 seconds. A watchdog job (Spring `@Scheduled`) marks jobs as STALE if heartbeat is older than 2 minutes, then re-queues them. Idempotent LLM calls via request ID (if the provider supports it) prevent double-billing.

**Self-Check**
- [ ] Can you give the Deep Fathom talk without using the word "basically" or "stuff"?
- [ ] Can you field the Key Vault outage curveball and the crashed worker curveball without pausing?

---

### Saturday, Sep 5 — Full Mock Day: Project Deep Dives + System Design + Company-Specific Prep

📌 **Study today:** Timed DSA mock (LC 21, 15/560, 146 LRU Cache) · Company-specific prep · System design mock (chat/messaging or rate limiter/notification) · behavioral rehearsal

**Time: ~4 hours**

**Block 1 (60 min) — Timed DSA Mock**
- [ ] LeetCode contest mode (no hints, fresh editor, time visible):
  - Problem 1 (Easy/warm-up, 10 min): **LC 21 – Merge Two Sorted Lists**
  - Problem 2 (Medium, 25 min): **LC 15 – 3Sum** or **LC 560 – Subarray Sum Equals K** — pick whichever you've done less recently.
  - Problem 3 (Medium, 25 min): **LC 146 – LRU Cache** — implement with a doubly-linked list + HashMap. This is the most common "design a data structure" question. Know it cold.
- [ ] After mock: score yourself. Did you finish cleanly? Were explanations clear? Write the issues down — don't just move on.

**Block 2 (60 min) — Company-Specific Prep for Pipeline Companies**
- [ ] For each company you have a scheduled round with (or a strong-signal recruiter call):
  - [ ] Re-read the Job Description. Highlight 3 tech keywords that appear. Map each to a specific resume story.
  - [ ] Research the company: product, recent news (blog, press release, LinkedIn), tech stack (engineering blog, job postings for clues).
  - [ ] Write 3 questions to ask the interviewer. At least one should be specific to the company (e.g., "I noticed you moved to a microservices model in your 2024 engineering blog — what were the biggest operational challenges in that migration?").
  - [ ] Review their Glassdoor/Blind interview reports if available. Note recurring question types.
- [ ] Log all of this in the tracking sheet — do not rely on memory.

**Block 3 (60 min) — System Design Mock: Tailored to Pipeline Companies**
- [ ] Pick one system design prompt relevant to a company you're interviewing with. Rotate across rounds so you cover breadth — if unsure, pick one of: "Design a rate limiter", "Design a notification service", or **"Design a chat / messaging system"** (pairs directly with the WebSocket/SSE knowledge from the Week 7 real-time collaboration mock and the Week 6 concurrency work — a strong rotation pick if a round leans real-time).

  *Chat / messaging concept note — drive these points if you pick this prompt:*
  - **WebSocket connection management:** persistent bidirectional WebSocket per client (vs SSE which is unidirectional — contrast this with the WebX SSE choice: chat needs client→server too). Millions of long-lived connections need a connection/gateway layer that maps `userId → (server, connectionId)` in a shared store (Redis), so any backend can find where a recipient is connected. Handle reconnects + heartbeats/ping-pong to detect dead sockets.
  - **Presence:** online/offline/last-seen via heartbeat TTL keys in Redis (`SETEX presence:userId 30s`); presence fan-out to a user's contacts on change. Accept eventual consistency here — a few seconds of stale "online" is fine.
  - **Message ordering:** per-conversation monotonic sequence number (or logical/Lamport-style clock), not wall-clock time — clients order by the conversation sequence so messages never appear out of order even with clock skew across senders.
  - **Delivery / read receipts:** state machine SENT → DELIVERED (recipient's device ACKed) → READ; receipts flow back as their own lightweight messages over the socket and update the sender's UI.
  - **Group fan-out:** small groups → fan-out-on-write to each member's inbox; large groups/channels → fan-out-on-read (pull) to cap write amplification — the same hybrid trade-off as the Week 10 News Feed HLD; name the connection.
  - **Message storage:** **wide-column store (Cassandra/HBase)** partitioned by `conversation_id`, clustered by `message_seq` — gives efficient "fetch last N / paginate older" per conversation at write-heavy scale, where a relational DB would struggle. Mention TTL/archival tiering for old messages.
  - Self-critique: did I cover connection routing (userId→server map), ordering by sequence (not timestamp), the receipt state machine, the small-vs-large group fan-out split, and the wide-column partition/cluster key choice — without prompting?
- [ ] Whiteboard it on paper, end to end, in 40 minutes.
  - Start with requirements (functional + non-functional). Ask clarifying questions aloud.
  - High-level components → data model → API contract → deep dive on 1 component → failure modes → scaling.
  - Time breakdown: 5 min requirements, 10 min high-level, 10 min deep dive, 10 min failure modes + scaling, 5 min wrap-up.
- [ ] In the remaining 20 min: walk through it aloud as if presenting to a panel. Anticipate and answer your own follow-ups.

**Block 4 (60 min) — Behavioral Rehearsal**
- [ ] Prepare and say aloud (timed, recorded if possible) each of the following STAR answers. Each answer must be 2–3 minutes. Every answer must include at least one specific number.
  - [ ] "Tell me about a time you disagreed with a technical decision and how you handled it."
  - [ ] "Tell me about your most impactful contribution to a project."
  - [ ] "Tell me about a time you had to learn something new quickly."
  - [ ] "Tell me about a production incident and how you handled it."
- [ ] Check: does every answer end with a concrete outcome (not just "it went well")?
- [ ] Prepare your answer to "Why are you leaving your current role?" — honest, forward-looking, no bitterness. Frame around growth: "I want to move into a product company environment where I can own a feature end-to-end, see user impact directly, and grow into a senior engineer role faster."

---

### Sunday, Sep 6 — Light Rehearsal + Week Review + Week 12 Plan

📌 **Study today:** Light DSA (LC 994, 207) · cold run all three project talk-tracks + 60-second career arc · pre-interview checklist rehearsal · Week 12 plan

**Time: ~4 hours**

**Block 1 (60 min) — Light DSA**
- [ ] Solve **LC 994 – Rotting Oranges** (Medium). Multi-source BFS. O(n) time/space. Reinforces BFS pattern from Week 9 trees.
- [ ] Solve **LC 207 – Course Schedule** (Medium). Cycle detection in a directed graph via DFS or Kahn's BFS topological sort. Graph problems are coming up in later-stage interviews.
- [ ] Review your Week 11 DSA complexity card (the sheet from Friday). For any problem where the complexity felt unclear, re-derive it on paper.

**Block 2 (60 min) — 5-Minute Project Story Run-Through (All Three)**
- [ ] Smart360: say the talk-track aloud, cold, no warm-up. Time it.
- [ ] Deep Fathom: say the talk-track aloud, cold. Time it.
- [ ] WebX: say the talk-track aloud, cold. Time it.
- [ ] Goal: all three under 5 minutes each, no filler, no "basically." If any runs long, identify which sentence is bloated and cut it.
- [ ] Bonus: link the three projects into a 60-second "career arc" answer for "walk me through your background":
  > "I've spent 2.5 years building full-stack Java microservices systems at increasing scale and complexity. Smart360 taught me performance engineering at the query and caching layer. Deep Fathom gave me infrastructure ownership — I built the entire Azure deployment, IaC, and CI/CD pipeline. WebX pushed me into async architecture and LLM integration. I'm now looking to bring all of that into a product company where I can own a domain end-to-end."

**Block 3 (45 min) — Pre-Interview Checklist Rehearsal**
- [ ] For the most important upcoming interview: run through the full checklist below, treating it as real. This builds the habit so it's automatic before every round.

**Block 4 (45 min) — End-of-Week Review + Week 12 Plan**
- [ ] Complete the Self-Assessment table at the bottom of this file.
- [ ] Review the tracking sheet: update status for every application, log every recruiter interaction from the week.
- [ ] Write 3 things you're more confident about than last Sunday.
- [ ] Write 2 things that still feel shaky — these go to the top of Week 12's focus list.
- [ ] Plan Week 12: are interviews continuing to cluster? If yes, maintain the same rhythm (maintenance DSA + rehearsal focus). If pipeline is thinning, identify 2 new companies to apply to on Monday.

---

## Pre-Interview Checklist (use before EVERY round)

Complete this the evening before and morning of each interview. It is not optional.

**Evening before (~30 min):**
- [ ] Re-read the Job Description. Identify: which of your 3 projects is most relevant? Which 2 tech areas appear most prominently?
- [ ] Research the company: product, tech stack (check their engineering blog), recent news. Write 3 specific things you learned.
- [ ] Prepare 3 questions to ask. At least one should reference something specific you found (blog post, product feature, company milestone).
- [ ] Review your talk-track for the most relevant project. Say it aloud once.
- [ ] Check Glassdoor/Blind for recent interview reports. Note recurring question types.
- [ ] Log everything in the tracking sheet.

**Morning of (15 min):**
- [ ] Virtual round: test your setup — camera, mic, screen share, stable internet. Have a phone backup ready. Open the LeetCode editor in a browser tab.
- [ ] Re-read your "extraordinary candidate" differentiator pitch. Say it aloud once.
- [ ] Review your 3 questions to ask. They should feel natural, not memorised.
- [ ] Have water. Have the tracking sheet open. 5 minutes before: close all notifications.

**During the interview:**
- [ ] Clarify the problem before coding — repeat the problem back, confirm constraints, ask about edge cases. This is not wasting time; it's signal.
- [ ] Think aloud while coding. The interviewer wants to see your process, not just your answer.
- [ ] State time and space complexity after every solution, unprompted.
- [ ] For behavioural: every answer ends with a specific number or concrete outcome.

**After the interview (5 min):**
- [ ] Write down every question asked in the tracking sheet, while fresh.
- [ ] Note what went well and what you'd change.
- [ ] Send a follow-up thank-you note within 4 hours if the process allows.

---

## 🧠 Concepts to Master This Week (consolidation, not new material)

### Frontend-Backend Integration (Full-Stack Cohesion)

**CORS — the exact mental model:**
- Same-origin policy is a browser rule. The browser sends the request; the browser blocks the response if CORS headers are missing.
- A server that lacks CORS config is not "secure" — it's just breaking the browser. A direct curl call bypasses CORS entirely.
- The `maxAge` on preflight responses matters for performance: browser caches the preflight result for `maxAge` seconds, avoiding a round-trip on every non-simple request.
- In Spring Cloud Gateway, configure CORS once globally:
  ```yaml
  spring.cloud.gateway.globalcors:
    corsConfigurations:
      '[/**]':
        allowedOrigins: "https://app.yourdomain.io"
        allowedMethods: ["GET","POST","PUT","DELETE","OPTIONS"]
        allowedHeaders: ["Authorization","Content-Type"]
        allowCredentials: true
        maxAge: 3600
  ```

**JWT Token Lifecycle on the Client — the exact flow:**

```
1. User submits login form
2. POST /auth/login → { accessToken (short-lived, ~15 min), refreshToken (long-lived, ~7 days) }
3. accessToken stored in memory (React: useState/Context; Vue: Vuex/Pinia store)
4. refreshToken stored in HttpOnly Secure SameSite=Strict cookie
5. Axios request interceptor: attaches Authorization: Bearer <accessToken> to every request
6. API call succeeds → response used normally
7. API call returns 401 → response interceptor fires:
   a. Call POST /auth/refresh (refresh token sent via cookie automatically)
   b. If 200: store new accessToken in memory, retry the original request with the new token
   c. If 401 (refresh expired/revoked): clear memory token, redirect to /login, show "Session expired"
8. API call returns 403 → show "Access denied" message, do NOT retry
9. On logout: POST /auth/logout (server invalidates refresh token in Redis blacklist), clear memory token
```

**API Error Contract — what you enforce:**
- Every error from the backend has: `status`, `message` (safe for display), `timestamp`, `path`.
- Frontend handles by status class: 4xx → user-fixable problem (show inline or toast); 5xx → system problem (show generic toast, log to error tracking); network error → "check your connection."
- Never surface raw exception messages to the user. `ConstraintViolationException` message is an internal detail; the frontend should see "Validation failed: email must not be blank."

### DSA Maintenance — Patterns by Company Relevance

**Sliding Window** (Amazon, Google, Atlassian, most product companies):
- Pattern: two pointers `left` and `right`, expand right, shrink left when a condition is violated.
- Know: LC 3, LC 76, LC 424, LC 567 cold. These 4 cover all sliding window variants.

**BFS/Level Order** (product companies, GCCs):
- Pattern: queue, process all nodes at current level before advancing.
- Know: LC 102 (level order), LC 199 (right side view), LC 994 (multi-source BFS — rotting oranges).

**Graph / Topological Sort** (later rounds, harder companies):
- Know: LC 207 (course schedule — cycle detection via DFS), LC 210 (course schedule II — topological sort output).

**Design a Data Structure** (appears as a standalone problem):
- Know: LC 146 (LRU Cache — doubly-linked list + HashMap) cold. It is asked verbatim more often than any other design problem.

---

## 🎤 Sample Interview Questions (incl. Curveballs) — 8–15 Qs + Pointers

### Full-Stack Integration

**1. "Walk me through how your Vue.js frontend authenticates with your Spring Boot API. What happens on a 401?"**
- Pointer: Give the full flow: Axios interceptors, token in memory, HttpOnly cookie for refresh, retry on 401. Then describe the fallback: if refresh also 401s, force logout. Mention that the gateway validates JWT so individual microservices don't need to — separation of concerns.

**2. "Curveball: Your frontend is on `https://app.example.com` and your API is on `https://api.example.com`. The browser is blocking the request. Walk me through your debugging steps."**
- Pointer: (1) Open DevTools Network tab — look for the failed OPTIONS preflight or a blocked response. (2) Check the response headers — is `Access-Control-Allow-Origin` present? Does it match the exact origin? (3) If using credentials, is `Access-Control-Allow-Credentials: true`? Is `withCredentials: true` on the Axios instance? (4) Check the Spring Cloud Gateway CORS configuration — is the path pattern `/**`? Is the origin exactly `https://app.example.com` (no trailing slash)? (5) Common mistake: returning `*` as origin with credentials — browser rejects this.

**3. "Curveball: A user reports that after 15 minutes of inactivity, their actions fail silently and the page doesn't redirect to login. What's happening?"**
- Pointer: The access token expired. The Axios response interceptor for 401 triggered, called the refresh endpoint, and got back a new access token — BUT the refresh token was also expired (user has been inactive for 7 days, or the HttpOnly cookie has a shorter-than-expected lifetime). The silent failure means either: the interceptor isn't rethrowing the error correctly after a failed refresh, OR the redirect to `/login` isn't happening because the 401 from the refresh endpoint is being caught by the interceptor recursively (infinite loop risk). Fix: in the response interceptor, check if the failing URL is the `/auth/refresh` endpoint itself — if yes, do NOT retry, directly redirect to login.

**4. "How did you handle form validation — both client-side and server-side — in Smart360?"**
- Pointer: Client side: Vuelidate or Yup schema validation on the form model — instant feedback before the API call. Server side: `@Valid` on the `@RequestBody` + `@NotBlank`, `@Email`, `@Size` annotations on the DTO + `MethodArgumentNotValidException` handler that returns a 400 with field-level error map. The frontend maps the error map back to individual form fields. This dual-layer approach prevents invalid data from ever reaching the DB, and the server-side validation is the authoritative source of truth (client validation can be bypassed).

### Microservices + Backend

**5. "Curveball: You have a Spring Boot service that calls two downstream services synchronously. Both calls take ~300ms. Total latency is ~650ms. How do you fix this?"**
- Pointer: Parallelize the calls. If the two calls are independent (no data dependency between them), use `CompletableFuture`:
  ```java
  CompletableFuture<ResultA> futureA = CompletableFuture.supplyAsync(() -> serviceA.call(), executor);
  CompletableFuture<ResultB> futureB = CompletableFuture.supplyAsync(() -> serviceB.call(), executor);
  CompletableFuture.allOf(futureA, futureB).join();
  ResultA a = futureA.get();
  ResultB b = futureB.get();
  ```
  Latency drops from ~650ms to ~320ms (max of the two calls + overhead). Alternatively, use reactive WebClient with `Mono.zip()` if the service is reactive. Point out: this works only if the calls are independent. If B depends on A's result, they must be sequential.

**6. "Walk me through what actually happens when a Spring @Transactional method calls another @Transactional method in the same class."**
- Pointer: This is the self-invocation problem. Both methods are in the same bean. The outer method is called via the Spring proxy. Inside the outer method, the inner method is called directly on `this` — not on the proxy. So the inner method's `@Transactional` annotation is completely ignored by Spring. The inner method runs in the same transaction as the outer (or no transaction if the outer has none). Fix: inject the bean into itself (`@Autowired` on the same class, Boot 3 supports this), or use `REQUIRES_NEW` via a separate bean, or restructure to avoid the pattern.

**7. "Curveball: You're getting OutOfMemoryError in production but memory usage looks fine on the JVM heap metrics. What do you check?"**
- Pointer: If heap metrics look fine, the OOM might be in off-heap memory — specifically: (1) **Metaspace**: class metadata, grows with dynamic class loading (common in Spring with proxies/reflection). Check `jvm.memory.used{area=nonheap}` in Actuator/Prometheus. (2) **Direct memory**: NIO, Netty (used by Spring WebFlux/Gateway). Check `jvm.nio.buffer.memory.used`. (3) **Thread stack memory**: too many threads, each with a large stack. (4) **Native memory** from JNI calls. In production: add `-XX:NativeMemoryTracking=summary` JVM flag and use `jcmd <pid> VM.native_memory` to get a breakdown. Also check for memory leaks in thread-local variables or event listener registrations that aren't cleaned up.

**8. "How did your 5-microservice system handle a scenario where the Database was temporarily unavailable?"**
- Pointer: Two layers of protection: (1) Resilience4j Circuit Breaker on the service that calls the DB-dependent service — after a threshold of failures, the circuit opens and calls fail fast (return cached data or a degraded response) rather than piling up. (2) Spring Boot readiness probe failing when DB is unreachable — Kubernetes stops routing traffic to the pod, which prevents new requests from failing. The pod stays Running (liveness is fine) but takes no traffic. The circuit breaker handles the runtime case; the readiness probe handles the startup case. Mention: Resilience4j's `@Bulkhead` limits concurrent calls so a slow DB doesn't exhaust the thread pool.

### Behavioral + Curveballs

**9. "Tell me about a time you had to push back on a deadline."**
- Pointer: Frame it positively: you advocated for quality. Use STAR. Key structure: you didn't just say "no" — you presented data (estimated effort vs. available time, specific risks of rushing), proposed an alternative (phased delivery, reduced scope for the deadline, identifying what could ship vs. what couldn't), and got alignment. The outcome should show that your pushback led to a better result. If you've never experienced this directly, use the closest approximation: scoping a feature down to hit a deadline. Avoid stories that make the manager the villain.

**10. "Curveball: You're asked to estimate how long a new microservice will take. How do you approach that?"**
- Pointer: Decompose before estimating. Don't give a number without decomposition — that's how estimates are always wrong. Steps: (1) List the major components: API contract definition, data model, business logic, integration with downstream services, security (auth), testing (unit + integration), CI/CD pipeline setup, deployment config. (2) For each component, estimate in hours with confidence levels. (3) Sum with a contingency buffer (typically 20–30% for unknowns). (4) State assumptions explicitly: "This assumes I have the API contract from the dependent service on Day 1. If that slips, add 3 days." (5) Commit to a range, not a point estimate: "6–9 days, assuming the DB schema is stable." This shows engineering maturity, not guessing.

**11. "Curveball: Describe the difference between a memory leak in Java and a memory problem in a Spring Boot app that isn't technically a leak."**
- Pointer: A true memory leak: objects that are referenced (so GC can't collect them) but are no longer needed. Classic Java examples: static collections that grow indefinitely, unclosed resources (streams, connections), inner classes holding references to outer class instances. Spring-specific non-leak memory problems: (1) Too many beans cached in request/session scopes. (2) `@Cacheable` with no eviction policy — the cache grows unboundedly. (3) ThreadLocal variables not removed after request processing (common with `TransactionSynchronizationManager` misuse). (4) Tomcat connection pool exhaustion — not a memory leak but presents as one. In Smart360: we had a Redis connection pool leak because a `Jedis` connection wasn't closed in a try-with-resources — fixed by switching to `LettuceConnectionFactory` (connection-pool managed automatically) via Spring Data Redis.

**12. "You've been working with LLMs. What do you see as the biggest engineering challenge in building reliable LLM-based features?"**
- Pointer: This is an opinion question — show genuine thinking, not a textbook answer. Strong answer: non-determinism is the hardest engineering challenge. The same input can yield different outputs, which breaks standard testing approaches. You can't assert `assertEquals(expected, llmOutput())`. Mitigation: evaluation-based testing (LLM-as-judge or human evals on a golden dataset), structured output enforcement (JSON schema + `response_format`), fallback logic for malformed responses, and observability (log every prompt + response with a trace ID). Also mention: latency is unpredictable (2–20 seconds vs 100ms for a DB call), which forces async patterns. Rate limits from providers add another layer of complexity. Cost is a metric you have to engineer around — prompt caching, model routing by task type, token budgeting.

**13. "Curveball: Explain what happens if you put @Async on a method and call it from within the same class."**
- Pointer: Same problem as `@Transactional` — `@Async` is implemented via a Spring proxy. Self-invocation bypasses the proxy. The method executes synchronously on the calling thread. No async behavior. No `CompletableFuture` returned. The fix is the same: inject the bean into itself or extract the async method to a separate Spring bean. This is a subtle question that tests whether you understand Spring's AOP proxy model deeply, not just the annotations.

**14. "How do you decide when to use synchronous vs asynchronous communication between microservices?"**
- Pointer: Use synchronous (REST/gRPC) when: you need an immediate response to complete the current operation (e.g., checking if a user has permission before proceeding), the caller must handle failure from the callee, or the interaction is simple and latency is acceptable. Use asynchronous (events/queues) when: the caller doesn't need an immediate result, the operation is long-running, you want loose coupling (the caller doesn't need to know about the callee), you need retry/durability guarantees, or the consumer might be temporarily unavailable (queue buffers). In Smart360: synchronous REST for permission checks (need immediate yes/no), async events for notification delivery (fire and forget, durable). The key principle: async trades simplicity for resilience and decoupling.

**15. "Curveball: Your API returns a 200 OK but the response body is empty. The service is definitely returning data. Walk me through debugging."**
- Pointer: Systematic: (1) Is the serialization failing silently? Check if the `@JsonIgnore` annotation is on too many fields, or a Jackson mixin is filtering the response. (2) Is the HTTP response body being consumed before it's written? Check for filters/interceptors that read the response stream (e.g., a logging filter that reads `response.getOutputStream()` without a wrapper — this is a common mistake). Use `ContentCachingResponseWrapper` for logging. (3) Check if the Controller is returning `void` or `ResponseEntity<Void>` by mistake. (4) Is the `@ResponseBody` annotation (or `@RestController`) missing? In a `@Controller` without `@ResponseBody`, returning an object tries to resolve a view. (5) Check GZIP compression: some proxy configs incorrectly double-compress, resulting in unreadable body. (6) Check Content-Length header: if set to 0 by an interceptor, some clients will ignore the body.

---

## 🌟 Extraordinary-Candidate Edge

**This week's edge is about presence, not content.** By Week 11 you know the material. What separates good from extraordinary in a live interview is:

**1. Leading with the answer, not the journey.**
Interviewers are pattern-matching. When asked "how does JWT validation work in your system?", start with the most important sentence: "The JWT is validated at the API Gateway on every request — no downstream service needs to touch auth." Then explain how. Weak candidates bury the answer at the end of a long explanation.

**2. Anticipating the follow-up and getting there first.**
After explaining CORS, add: "The reason this matters at the gateway and not individual services is that we can enforce a consistent security policy without every team needing to configure it." You answered the question they were about to ask. This signals architectural thinking, not just knowledge.

**3. Connecting the dots between projects.**
"In Smart360 I first encountered the N+1 problem. In Deep Fathom I built the infra to prevent it from happening in the first place — private endpoints, read replicas, and connection pooling at the Container Apps level." Interviewers remember candidates who weave a coherent engineering narrative, not a list of bullet points.

**4. Owning the numbers.**
Every metric you cite, you own. Be ready for: "How did you measure the 96% latency reduction?" Answer: "We had Prometheus metrics on the endpoint P95 latency before and after. We ran the change in staging against production-like data volume and then monitored the first week in prod — P95 stabilised at 2.1 seconds compared to 58 seconds before."

**5. Showing intellectual honesty about trade-offs.**
When asked why you chose technology X, name a real trade-off: "Bicep over Terraform — the trade-off is that Bicep is Azure-only. If we ever needed multi-cloud, we'd need to rewrite the infra layer. For a client locked into Azure enterprise agreement with no multi-cloud requirement, that's an acceptable constraint." Interviewers trust candidates who show nuanced judgment over candidates who say "it was the best choice."

**6. Asking a question that reveals you did homework.**
Before every interview, find one specific engineering blog post, changelog, or product feature from that company. Turn it into a question: "I read your post about migrating from Kafka to Pulsar last year — what were the biggest operational surprises in that migration?" This single question signals that you are serious, curious, and different from the 20 other candidates they interviewed that week.

---

## 📊 End-of-Week Self-Assessment

Rate yourself 1–5 after Sunday's rehearsal. Be honest — this feeds directly into Week 12 planning.

| Area | Target | Your Rating | Gap / Next Action |
|---|---|---|---|
| Smart360 talk-track — 5 min, no notes, no filler | 5 | | |
| Deep Fathom talk-track — 5 min, no notes, no filler | 5 | | |
| WebX talk-track — 4 min, no notes, no filler | 5 | | |
| 60-second career arc answer | 5 | | |
| CORS — full explanation incl. preflight and credentials | 5 | | |
| JWT client flow — full flow including 401 handling | 5 | | |
| API error contract — backend + frontend handling | 5 | | |
| Full-stack integration story (coherent end-to-end) | 5 | | |
| LLM async job architecture — whiteboard in 10 min | 5 | | |
| Sliding window DSA — LC 3, LC 76, LC 424, LC 567 | 5 | | |
| BFS/Tree DSA — LC 102, LC 199, LC 994 | 5 | | |
| LRU Cache — implement from scratch (LC 146) | 5 | | |
| Pre-interview checklist — completed for all rounds | 5 | | |
| Company research — 3 questions ready for each | 5 | | |
| Behavioral answers — specific numbers in every answer | 5 | | |
| "Why are you leaving?" — polished, positive framing | 5 | | |
| Extraordinary-candidate pitch — under 90 seconds, fluent | 5 | | |
| Tracking sheet — fully updated with all pipeline activity | 5 | | |

**Scoring guide:**
- 5: Can do it cold, under pressure, with an interviewer watching.
- 4: Solid but hesitates on one edge case or loses a specific number.
- 3: Know the concept, stumble in delivery — needs 2 more reps minimum.
- 2: Shaky understanding — must revisit the material before the next interview.
- 1: Not ready — this is a risk to an upcoming interview. Address immediately.

**Week 11 exit criteria (minimum bar before Week 12):**
- All three project talk-tracks deliverable in under 5 minutes each, no notes.
- Pre-interview checklist completed for every scheduled round.
- CORS + JWT token flow explainable end-to-end without hesitation.
- LRU Cache (LC 146) implementable from scratch in under 25 minutes.
- Tracking sheet current and complete.

**If you missed the bar:** Do not skip missed items. They become Day 1–2 of Week 12. An unprepped round costs you far more than a delayed application.

---

*Week 11 complete → Week 12 (Sep 7–13): Debrief scheduled interviews, identify gaps, targeted reinforcement. If offers arrive: negotiation strategy and counteroffer prep. If pipeline thins: resume second-wave outreach with updated talking points. Advanced topics if bandwidth permits: virtual threads (Java 21), Spring AI, system design at larger scale (Cassandra, event sourcing).*
