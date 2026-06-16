# Week 4 — Foundations (Jul 6–12, 2026)

> Last foundation week: seal the cracks in DSA (Stacks, Queues, Linked Lists), lock in REST/OAuth2/JWT production knowledge, and prove to yourself you can perform under timed pressure.

---

## 🎯 Week Goal

Exit this week able to: solve any Stack/Queue/LinkedList LeetCode medium cold in ≤25 min; explain the full OAuth2/JWT lifecycle (token issuance → validation → refresh → revocation → RBAC) without notes; design a complete URL shortener system doc from scratch in 45 min. This is the final foundation checkpoint before Phase 2 (system design depth + advanced DSA).

---

## ✅ By Sunday you can...

- [ ] Solve Valid Parentheses, Min Stack, Daily Temperatures, Largest Rectangle in Histogram, Reverse Linked List, Merge Two Sorted Lists, Linked List Cycle, Reorder List, LRU Cache, Remove Nth Node From End — all without hints
- [ ] Explain exactly how Smart360's JWT issuance, validation, refresh, and RBAC (repository/table/permission levels) works in a 5-minute whiteboard answer
- [ ] Describe REST idempotency guarantees for every HTTP method and give a concrete non-obvious example of violating them
- [ ] Write a `@ControllerAdvice` global exception handler with correct HTTP status codes from memory
- [ ] Produce a written system design doc for a URL shortener covering API contract, data model, encoding algorithm, caching strategy, and scaling path
- [ ] Complete a timed 45-min 2-problem DSA test and score yourself honestly
- [ ] List your 3 weakest areas to drill in Phase 2

---

## 📅 Daily Checklist

### Monday Jul 6 — Stacks: Core Problems

**Time budget: ~1.5 hr**

**DSA (55 min)**
- [ ] Read the Stack pattern primer: "Monotonic stack" mental model — what it is, when to use it (next greater element, span problems, histograms)
- [ ] LC #20 — Valid Parentheses (Easy) — solve in ≤10 min, review edge: `"]"` empty stack crash
- [ ] LC #155 — Min Stack (Medium) — implement with two stacks; understand why `O(1)` getMin works; write it clean
- [ ] LC #739 — Daily Temperatures (Medium) — monotonic decreasing stack; trace through `[73,74,75,71,69,72,76,73]` by hand before coding
- [ ] **Monotonic-stack family (next-greater + span)** — the primer above promised "span problems"; here they are. Stack holds indices of a decreasing sequence; you pop and resolve when the current element breaks monotonicity:
  - [ ] LC #496 — Next Greater Element I (Easy) — precompute next-greater for every value in `nums2` with one monotonic-stack pass into a `Map<value, nextGreater>`, then look up each `nums1` query in O(1). The base case for the whole family.
  - [ ] LC #503 — Next Greater Element II (Medium) — **circular** array; iterate `2n` indices using `i % n` so the wrap-around can resolve still-pending elements; only push during the first pass (or just guard with index `< n`). Same decreasing-index stack.
  - [ ] LC #901 — Online Stock Span (Medium) — the canonical **span** problem; stack of `(price, span)` pairs. On each new price, pop while top price ≤ current and accumulate their spans — collapses consecutive smaller days into one entry, giving amortized O(1) per `next()` call. Tie-in: this is the same "collapse a run, keep a running aggregate" shape as a streaming metrics rollup.
- [ ] After each: write one line "What would trip me up on this in an interview?"

**Core (30 min)**
- [ ] REST resource naming rules: nouns not verbs (`/users/{id}` not `/getUser`), plurals for collections, sub-resources for relationships (`/users/{id}/orders`)
- [ ] HTTP status codes to memorize cold: 200, 201, 204, 400, 401, 403, 404, 409, 422, 429, 500, 502, 503
- [ ] Idempotency table: GET ✓, PUT ✓, DELETE ✓, POST ✗, PATCH ✗ (debated) — be ready to explain why POST to create is not idempotent and how to make it safe with idempotency keys

**Self-check (5 min)**
- [ ] Without notes: what is the difference between 401 and 403? Between 409 and 422?
- [ ] Can you explain monotonic stack in one sentence?

---

### Tuesday Jul 7 — Stacks: Hard Problem + REST Versioning & Pagination

**Time budget: ~1.5 hr**

**DSA (55 min)**
- [ ] LC #84 — Largest Rectangle in Histogram (Hard) — this is a Stack flagship; spend 15 min thinking before coding
  - Key insight: for each bar, find the leftmost/rightmost bar that is at least as tall → use stack to track indices
  - Trace through `[2,1,5,6,2,3]` → expected output 10
  - Common mistake: not handling the remaining stack after the loop — add a dummy 0-height bar at the end
- [ ] Re-solve Daily Temperatures without looking at yesterday's solution — time yourself (target ≤18 min)
- [ ] Write time/space complexity for all 4 stack problems this week in your notes

**Core (30 min)**
- [ ] API versioning strategies:
  - URI path: `/v1/users` — most common, most visible, breaks bookmarks on version change
  - Request header: `Accept: application/vnd.app.v2+json` — clean URIs, less discoverable
  - Query param: `/users?version=2` — easiest to test, not RESTful purists' choice
  - Which to use when: URI for public APIs (easy to route at gateway), header for internal services
- [ ] Pagination:
  - Offset: `?page=2&size=20` — simple but slow on large offsets (PostgreSQL scans all skipped rows)
  - Cursor/keyset: `?after=eyJpZCI6MTAwfQ==` (base64 encoded last-seen ID) — O(log n) stable under inserts/deletes; use for feeds/infinite scroll
  - Return: total count, next cursor, `Link` headers (RFC 5988)
- [ ] Relate to Smart360: what pagination did you use in the data service APIs? Would you change it?

**Self-check (5 min)**
- [ ] Explain Largest Rectangle in Histogram approach out loud without looking at code
- [ ] What are the trade-offs of cursor vs. offset pagination for a 10M-row table?

---

### Wednesday Jul 8 — Linked Lists: Core Problems + JWT Lifecycle Deep Dive

**Time budget: ~1.5 hr**

**DSA (45 min)**
- [ ] LC #206 — Reverse Linked List (Easy) — solve iteratively AND recursively; know both cold
  - Iterative: `prev=null, curr=head`; walk forward while swapping pointers
  - Recursive: base case `head.next == null`; trust the recursion to reverse the rest
- [ ] LC #21 — Merge Two Sorted Lists (Easy) — dummy head pattern; always use a dummy to avoid head-special-case bugs
- [ ] LC #141 — Linked List Cycle (Easy) — Floyd's tortoise and hare; slow moves 1, fast moves 2; they meet iff cycle exists; *bonus*: find cycle start (LC #142)
- [ ] After each: write the pointer diagram — draw boxes and arrows for a 3-node example

**Core (40 min) — JWT/OAuth2 End-to-End (Smart360 context)**
- [ ] **Token Issuance flow** (write this out step by step):
  1. User POSTs credentials to `/auth/login`
  2. `AuthService` validates credentials against DB (BCrypt compare)
  3. Builds claims: `sub` (userId), `roles` (e.g., `["ROLE_ADMIN","ROLE_USER"]`), `tenantId`, `iat`, `exp` (e.g., 15 min for access token)
  4. Signs with HMAC-SHA256 (`HS256`) using secret, or RSA private key (`RS256`) for multi-service verification
  5. Returns: `{ accessToken, refreshToken, expiresIn }`
  6. Refresh token stored hashed in DB (or Redis) with `userId`, `tokenFamily` (for rotation)
- [ ] **Validation flow** (every protected request):
  1. Spring Security `JwtAuthenticationFilter` intercepts request (extends `OncePerRequestFilter`)
  2. Extracts Bearer token from `Authorization` header
  3. Parses and validates signature (local, no DB call) → 60% latency win from Smart360
  4. Checks `exp` claim
  5. Checks Redis blacklist (for logged-out tokens — only O(1) lookup)
  6. Sets `SecurityContextHolder` with `UsernamePasswordAuthenticationToken` carrying roles
- [ ] **Refresh flow**:
  1. Client POSTs `refreshToken` to `/auth/refresh`
  2. Server validates refresh token (DB lookup), issues new access token + rotates refresh token (token family rotation — invalidates old refresh token, detects theft if old one used again)
  3. Old refresh token marked invalid
- [ ] **RBAC at repository/table/permission levels (Smart360)**:
  - Permission levels stored in DB: `USER_ROLE` → `[READ_REPORTS, WRITE_REPORTS, MANAGE_USERS]`
  - JWT payload includes role names → decoded in filter
  - `@PreAuthorize("hasAuthority('READ_REPORTS')")` at controller/service level
  - Custom `@PermissionRequired` annotation backed by AOP for fine-grained checks
  - Repository level: `findByIdAndTenantId()` — tenant isolation baked into queries
  - Table level: PostgreSQL RLS as a second line of defense (same pattern as Deep Fathom)

**Self-check (5 min)**
- [ ] Draw the JWT validation filter chain from memory: what happens at each step when the token is expired?
- [ ] Reverse a linked list in your head on a 4-node list: `1→2→3→4→null` → `4→3→2→1→null`

---

### Thursday Jul 9 — Linked Lists: Medium Problems + Global Exception Handling

**Time budget: ~1.5 hr**

**DSA (50 min)**
- [ ] LC #19 — Remove Nth Node From End (Medium) — two-pointer: advance fast pointer n steps, then move both until fast.next == null; handle edge: removing the head (n == length)
- [ ] LC #143 — Reorder List (Medium) — **three-step pattern**:
  1. Find middle (slow/fast pointers)
  2. Reverse second half
  3. Merge two halves by interleaving
  - Most candidates skip step 2 and get stuck — do the dry run first
- [ ] Time-box: 20 min per problem; if stuck at 15 min, look at the hint for the algorithm idea only, then code from there

**Core (35 min) — `@ControllerAdvice` + Bean Validation**
- [ ] Global exception handler — write this from memory:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("NOT_FOUND", ex.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult().getFieldErrors()
            .stream()
            .map(fe -> fe.getField() + ": " + fe.getDefaultMessage())
            .toList();
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
            .body(new ErrorResponse("VALIDATION_ERROR", errors.toString()));
    }

    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleForbidden(AccessDeniedException ex) {
        return ResponseEntity.status(HttpStatus.FORBIDDEN)
            .body(new ErrorResponse("FORBIDDEN", "Insufficient permissions"));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        // log ex fully here — never expose stack trace to client
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred"));
    }
}
```

- [ ] Bean Validation annotations to know: `@NotNull`, `@NotBlank`, `@NotEmpty`, `@Size`, `@Min/@Max`, `@Email`, `@Pattern`, `@Valid` (triggers cascade), `@Validated` (for groups)
- [ ] Key: `@Valid` on the method parameter triggers Spring MVC validation before the method body runs; the `MethodArgumentNotValidException` is thrown automatically
- [ ] Custom validator: `@Constraint(validatedBy = MyValidator.class)` implementing `ConstraintValidator<MyAnnotation, String>`

**Self-check (5 min)**
- [ ] What HTTP status should a validation failure return? (422 Unprocessable Entity OR 400 — know both and have an opinion)
- [ ] Why should the generic `Exception` handler never return the exception message to the client?

---

### Friday Jul 10 — LRU Cache + OAuth2 Flows + Interview Simulation

**Time budget: ~1.5 hr**

**DSA (50 min)**
- [ ] LC #146 — LRU Cache (Medium) — this is a design problem disguised as DSA
  - Data structure: `LinkedHashMap` in Java (access-order mode) — the interview-acceptable O(1) solution
  - **Preferred full implementation**: `HashMap<Integer, Node>` + doubly linked list (DLL)
    - HashMap: key → Node (O(1) lookup)
    - DLL: MRU at head, LRU at tail; on `get` or `put` → move node to head; on capacity → remove tail
    - Sentinel head/tail nodes eliminate null checks
  - Write out the full implementation; this comes up in senior/backend interviews frequently
  - Time yourself: target full solution in ≤30 min

**Core (30 min) — OAuth2 Flows + Security Edge Cases**
- [ ] OAuth2 grant types (know which to use when):
  - **Authorization Code + PKCE**: user-facing web/mobile apps — most secure; code exchanged for token server-side; PKCE prevents code interception
  - **Client Credentials**: service-to-service (no user involved) — your microservice calling another microservice's protected endpoint
  - **Implicit**: deprecated — never recommend
  - **Resource Owner Password**: legacy, avoid — exposes credentials to client app
- [ ] Token storage best practices:
  - Access token: in-memory JS (not localStorage — XSS risk, not cookie — CSRF risk if not configured correctly)
  - Refresh token: HttpOnly, Secure, SameSite=Strict cookie
- [ ] Algorithm confusion attack: always configure `JwtParser` with explicit algorithm: `Jwts.parserBuilder().setSigningKey(key).build()` — never trust the `alg` header from the token itself

**Self-check (5 min)**
- [ ] Which OAuth2 flow does Smart360 use? Could you justify the choice to a security engineer?
- [ ] What is PKCE and why does it matter for mobile apps?

---

### Saturday Jul 11 — Timed DSA Test + System Design Deep Dive

**Time budget: ~4 hr**

#### Block 1 (45 min): Timed DSA Test — Treat This Like a Real Interview

**Rules**: no hints, no solutions, timer running, write clean code as if on a whiteboard

- [ ] **Problem A (Easy-Medium, 20 min)**: LC #141 — Linked List Cycle — cold solve, state your approach before coding
- [ ] **Problem B (Medium, 25 min)**: LC #739 — Daily Temperatures — cold solve

**After the timer:**
- [ ] Score yourself:
  - Did you talk through your approach before coding? (Y/N)
  - Did you handle edge cases before being asked? (Y/N)
  - Did you finish within time? (Y/N)
  - Was your code compilable without fixes? (Y/N)
- [ ] Write what you would do differently

#### Block 2 (30 min): REST API Design Review

- [ ] Review all REST concepts from Mon–Fri
- [ ] Write from memory: what does an ideal REST API response look like for:
  - Successful creation: status code, body, Location header
  - Validation error: status code, body structure with field-level errors
  - Not found: status code, body
  - Paginated list: body structure (data array, pagination metadata)

#### Block 2.5 (15 min): Back-of-Envelope Estimation Drill

Practice the *method* in isolation — the arithmetic candidates fumble live when it's embedded in a bigger design. Treat this as a repeatable warm-up you run before ANY capacity question (the URL shortener below is just one instance). Drill the chain cold, no calculator:

**QPS → storage → bandwidth → memory**

- [ ] **QPS from a daily total**: rule of thumb — **seconds/day ≈ 86,400 ≈ 86.4K** (round to 100K for fast mental math). So `1B reads/day ÷ 86.4K ≈ 11.6K QPS` average. **Peak ≈ 2–3× average** — state your multiplier and design for peak, not average.
- [ ] **Read:write ratio**: ask for it or assume one (10:1 is a common default for read-heavy systems). Writes QPS = reads ÷ ratio. This tells you whether to optimize the write path (sharding, queue) or the read path (cache, replicas).
- [ ] **Bytes → storage/year**: `write_QPS × bytes_per_record × seconds/year`. Rule of thumb — **seconds/year ≈ 31.5M (≈ 3.15 × 10^7)**. Example: 100 writes/s × 500 B × 31.5M s ≈ **1.5 TB/yr**. Always tack on index + replication overhead (×2–3).
- [ ] **Bandwidth**: `QPS × payload_size`. Example: 11.6K reads/s × 500 B ≈ **5.8 MB/s** egress. Separate read vs write bandwidth.
- [ ] **Memory for hot-set caching**: apply the **80/20 rule** — cache the ~20% of data serving ~80% of traffic. Example: 100M URLs × 100 B = 10 GB total; the hot 20% ≈ **2 GB**, comfortably fits one Redis node. This is the number that justifies "Redis fits in memory" instead of hand-waving.

**Powers-of-2 / units cheat sheet to memorize**: 2^10 ≈ 1 K (thousand), 2^20 ≈ 1 M (million), 2^30 ≈ 1 B (billion → 1 GB); char = 1 B, typical metadata row ≈ 100–500 B, UUID = 16 B, timestamp = 8 B.

**Self-check:** Given "500M new posts/day, each post averages 1 KB, read:write 100:1" — produce write QPS, read QPS, storage/yr, and hot-cache size in under 2 minutes, out loud. Then carry these exact numbers into Block 3.

#### Block 3 (2 hr): Full System Design — URL Shortener

Write a complete design document (can be in a scratch file or paper) covering ALL sections:

**1. Requirements clarification (5 min)**
- Functional: shorten long URL → short code; redirect short URL → original; analytics (optional)
- Non-functional: 100M URLs, 10:1 read:write ratio, < 10ms redirect latency, 99.99% availability
- Out of scope for v1: custom aliases, expiry, A/B testing

**2. API contract**
```
POST /api/v1/urls
  Body: { "longUrl": "https://..." }
  Response 201: { "shortCode": "abc123", "shortUrl": "https://short.ly/abc123" }

GET /{shortCode}
  Response 301/302: Location: https://original-long-url.com
  (301 = permanent, cached by browser; 302 = temporary, every request hits server — choose 302 for analytics)
```

**3. Data model**
```sql
urls (
  id          BIGSERIAL PRIMARY KEY,
  short_code  CHAR(7)     UNIQUE NOT NULL,
  long_url    TEXT        NOT NULL,
  created_at  TIMESTAMPTZ DEFAULT now(),
  user_id     BIGINT REFERENCES users(id),
  click_count BIGINT      DEFAULT 0
);
CREATE INDEX idx_short_code ON urls(short_code);
```

**4. Short code generation algorithm**
- Option A: Base62 encode a counter (a-z A-Z 0-9): `counter → base62 → 7 chars = 62^7 = ~3.5T URLs` — predictable, sequential
- Option B: MD5/SHA256 of long URL → take first 7 chars — collision-prone, non-unique for same URL
- Option C: **Recommended**: pre-generate a pool of random 7-char base62 codes in a `code_pool` table; worker populates pool; URL service claims one atomically via `SELECT FOR UPDATE SKIP LOCKED` — no collision, no counter coordination problem across shards
- Alternative for distributed: use Twitter Snowflake-style unique ID → base62 encode

**5. Caching strategy**
- Redis: `shortCode → longUrl` with TTL = 24h (LRU eviction)
- Cache hit rate target: >99% for popular URLs
- Read path: check Redis first; miss → PostgreSQL → populate Redis
- Write path: write to DB first, then invalidate/populate cache
- Cache size estimate: 100M active URLs × avg 100 bytes/URL = 10GB — fits in Redis cluster

**6. Scaling path**
- Read replicas for PostgreSQL (10:1 read ratio)
- Read-heavy → add Redis cluster; horizontal scale app tier (stateless)
- Writes: partition `urls` table by `short_code` hash range for sharding at scale
- CDN: PUT redirect logic at CDN edge for sub-millisecond redirects (304 with cached Location header)
- Rate limiting: token bucket per IP at API gateway (protect POST endpoint from URL flooding)

**7. What you'd do differently at 1B URLs**
- Move to Cassandra for write-heavy workload with known access pattern (short_code lookup)
- Separate analytics service: async click events → Kafka → ClickHouse for aggregation
- Geographic distribution: region-aware routing so short.ly/abc123 resolves at nearest PoP

#### Block 4 (45 min): Weak Spot Identification

- [ ] Review all 4 weeks of notes
- [ ] Write honestly: **Top 3 weak areas** (e.g., "Histogram problem pattern", "OAuth PKCE details", "System design scaling math")
- [ ] For each weak area: write 1 specific action for Phase 2 (e.g., "Do 3 more monotonic stack problems", "Implement PKCE flow in a test project")

---

### Sunday Jul 12 — Review, Consolidation, Phase 2 Preview

**Time budget: ~4 hr**

#### Block 1 (60 min): Linked List + Stack Revision

- [ ] Re-solve without hints (timer on, 15 min each):
  - LC #143 — Reorder List
  - LC #84 — Largest Rectangle in Histogram
- [ ] For any problem you get wrong or slow on: add it to the Phase 2 warm-up list

#### Block 2 (60 min): JWT/REST Verbal Drill

Practice answering these out loud (stand up, talk to yourself or record):

- [ ] "Walk me through exactly how JWT authentication works in Smart360, end to end"
  - Target: 4-5 min answer covering issuance, filter chain, RBAC, refresh
- [ ] "How would you design the auth layer for a new microservice that only other internal services call?"
  - Target: mention Client Credentials, service account JWTs, mTLS as alternative
- [ ] "What's wrong with storing JWTs in localStorage?"
  - Target: XSS attack → token theft → impersonation; mitigate with HttpOnly cookie + CSRF token
- [ ] "Design a rate-limiting API" (this tests REST + system design combined)

#### Block 3 (60 min): Write Your Phase 2 Plan

Review what Week 5+ should cover based on your self-assessment. Key areas to ensure are on the list:
- System design: databases at scale, consistent hashing, message queues deep dive
- DSA: Trees and Graphs (DFS/BFS patterns)
- Spring: reactive programming, async patterns
- Behavioral: STAR stories polished

#### Block 4 (60 min): Interview Simulation — Behavioral Stories

Polish these 3 STAR stories for use in any interview:

1. **Performance story** (Smart360 60s → 2s): practice delivering in exactly 2 min
2. **Technical decision story** (event-driven migration): emphasize trade-off thinking
3. **New story to add**: a time you pushed back on a requirement or caught a bug before production — think of a real example, structure it in STAR format, write it out

---

## 🧠 Concepts to Master This Week

### Data Structures

**Stack**
- Monotonic stack: maintains elements in monotonically increasing/decreasing order; pop when current element breaks the monotonicity; used for "next greater element", "span", "histogram" problems
- Key operations: O(1) push/pop/peek; backed by `Deque<Integer> stack = new ArrayDeque<>()` in Java (never use `Stack<>` class — it's legacy synchronized)
- `ArrayDeque` vs `LinkedList` as stack: `ArrayDeque` is faster (array-backed, no node allocation)

**Queue / Deque**
- Monotonic deque: used in sliding window maximum (LC #239 — good Phase 2 problem)
- `ArrayDeque` as queue: `addLast()` / `pollFirst()`

**Linked List**
- Dummy head node pattern: eliminates head-is-null special case in merge/remove operations
- Two-pointer patterns: slow/fast (cycle detection, midpoint finding), fixed-gap (nth from end)
- Reversal: always know both iterative (3 pointers: prev/curr/next) and recursive
- DLL + HashMap = LRU Cache

### REST API Design
- **Idempotency**: an operation is idempotent if calling it N times has the same effect as calling it once. PUT replaces a resource identically. DELETE of a non-existent resource should return 404 or 204 — both are idempotent (state doesn't change after second call). POST creates a new resource each time — not idempotent. Idempotency keys on POST: `Idempotency-Key: <uuid>` header; server stores key + response; duplicate request returns cached response without re-executing.
- **HATEOAS** (Hypermedia): response includes links to related actions (`_links: { self, next, cancel }`); Spring HATEOAS library; rarely implemented but good to know the concept

### JWT/OAuth2
- JWT structure: `base64url(header).base64url(payload).base64url(signature)` — the signature is the only part that provides security guarantees; payload is readable by anyone
- `RS256` vs `HS256`: RS256 uses private key to sign, public key to verify — allows multiple services to verify without sharing the secret. Prefer RS256 in multi-service environments.
- Token family rotation: each refresh generates a new refresh token and invalidates the family if the old one is replayed — detects refresh token theft
- Spring Security filter chain order: `SecurityContextPersistenceFilter` → `UsernamePasswordAuthenticationFilter` → your custom `JwtAuthenticationFilter` → `ExceptionTranslationFilter` → `FilterSecurityInterceptor`

---

## 🎤 Sample Interview Questions (incl. curveballs)

**DSA**

1. *(Medium)* "Implement a stack that supports push, pop, top, and retrieving the minimum element in O(1) time." — Min Stack; answer: maintain an auxiliary `minStack` that only pushes when new min ≤ current min; pop from both on pop.

2. *(Curveball)* "How would you implement an LRU Cache without using `LinkedHashMap`?" — They want to see you build the DLL + HashMap manually. Walk through: Node class (key, val, prev, next), sentinels, moveToHead(), removeLast().

3. *(Hard, expect in senior loops)* "Given a histogram, find the maximum rectangle. Now what if the histogram is a stream and you need to answer queries as each bar arrives?" — Extend with a running stack; this tests whether you understand the invariant, not just the algorithm.

**REST & API Design**

4. *(Standard)* "What HTTP method and status code would you use to partially update a resource?" — PATCH + 200 (with updated resource) or 204 (no content). Distinguish from PUT (full replacement). PATCH is not guaranteed idempotent — e.g., PATCH `{ increment: 5 }` is not idempotent.

5. *(Curveball)* "A client calls DELETE /orders/123 twice. The first call succeeds. What should the second call return — 404 or 204?" — Both are defensible; the important answer is: idempotency means the server state is the same, but the HTTP spec allows 404 for second call. In practice, return 204 for better client experience (don't force client to handle 404 as an error on a delete).

6. *(Curveball)* "You're designing an API for a bank transfer. POST /transfers is not idempotent — how do you make it safe for clients to retry on network failure?" — Idempotency-Key header; server stores `idempotency_key → result` in Redis with TTL; on replay, return the stored result without executing the transfer again.

**JWT / Security**

7. *(Standard)* "What happens in Smart360 when a user logs out? How do you invalidate their JWT?" — JWT is stateless; we can't un-issue it. On logout: add `jti` (JWT ID) to a Redis blacklist with TTL = remaining token lifetime. Every request checks Redis blacklist (O(1) lookup, minimal latency impact). Refresh token is deleted from DB.

8. *(Curveball)* "An attacker intercepts a user's JWT. What's your defense assuming the token hasn't expired?" — Short expiry (15 min access token) limits damage window. Token binding (binding token to TLS session) is theoretically sound but rarely implemented. Redis blacklist only helps post-logout. The real mitigations are: HTTPS-only, short expiry, HttpOnly cookies (harder to steal than localStorage). Acknowledge the trade-off — this is a known limitation of stateless JWTs.

9. *(Deep dive)* "Walk me through RBAC at three levels: API, service, and database. What breaks if you only implement it at one level?" — API level (`@PreAuthorize`): stops unauthorized calls but a bug bypasses it. Service level: defense-in-depth. Database level (RLS): even a compromised service can't leak cross-tenant data. Smart360 used all three; Deep Fathom used PostgreSQL RLS for 50+ tables — this answer shows production maturity.

**System Design / Architectural**

10. *(Standard)* "How would you add rate limiting to your API Gateway?" — Redis token bucket: `INCR` key `user:{id}:req_count`, set `EXPIRE` on first increment (e.g., 60s window), if count > limit return 429. Spring Cloud Gateway has `RequestRateLimiterFilter` backed by Redis out of the box. Add `Retry-After` header on 429 response.

11. *(Curveball)* "Your URL shortener is getting 10K redirect requests/sec. Redis is your cache. What do you do if Redis goes down?" — Circuit breaker on Redis client; fallback to PostgreSQL directly (slower but functional); set timeout low (50ms) so Redis failure is detected quickly. Pre-warm a read replica that can serve as fallback. For extreme resilience: read-through to DB, alert on cache miss spike.

12. *(Behavioral/Technical)* "You said you reduced sign-in latency by 60% by switching to JWT. What would you have done if the security team had required server-side session validation on every request?" — Acknowledge the trade-off: stateless JWT is a security vs. performance trade-off. If server-side validation was required, would use Redis session store with connection pooling and pipelining; would batch session checks if multiple services need to validate; would accept the latency hit and compensate with better hardware/caching at the Redis layer.

**Curveball / Behavioral**

13. *(Curveball)* "If you had to remove one service from your 5-microservice architecture to simplify it, which one would you remove and why?" — Shows you understand coupling and complexity. Good answer: evaluate by dependency graph — the service with the most inbound dependencies and least unique functionality should be merged. Be specific: "I'd evaluate merging the notification service into the user service boundary if event volume is low, because the async decoupling adds operational overhead without proportionate benefit at current scale."

14. *(Stress test)* "How do you handle a situation where a manager asks you to skip security review to hit a release deadline?" — Calibrate: acknowledge the business pressure, then explain the asymmetric risk of a security incident vs. a one-sprint delay. Offer alternatives: time-box the review, do a targeted review on just the new surfaces, ship with a feature flag to reduce exposure. Never "yes-and-here's-why-it'll-be-fine" a skip.

15. *(Deep Java)* "In your JWT filter, you use `ThreadLocal` via `SecurityContextHolder`. What happens in a reactive/WebFlux context?" — `SecurityContextHolder` is ThreadLocal-based and breaks in reactive pipelines because a single request can span multiple threads. WebFlux uses `ReactiveSecurityContextHolder` backed by Reactor's context (a `Context` object attached to the `Mono`/`Flux` chain, not a thread). If you were to migrate to Spring WebFlux, the JWT filter would need to become a `WebFilter` using `exchange.mutate()` and `ReactiveSecurityContextHolder.withAuthentication()`.

---

## 🌟 Extraordinary-Candidate Edge

**Technical differentiation moves**

- When answering the LRU Cache problem, implement the DLL version, not just `LinkedHashMap`. Then mention: "In production I'd use Redis with `EXPIRE` and `LFU` eviction policy — but understanding the underlying structure helps me reason about Redis's own eviction behavior."

- On JWT: don't just say "we use JWT in Smart360." Say: "We evaluated opaque tokens vs. JWT. We chose JWT specifically because we had 5 services that all needed to verify identity without chatty calls to the auth service. The 60% latency reduction was a direct consequence of that architectural choice." That is a trade-off narrative, not a recitation.

- On REST API versioning: anticipate the "what do you do when breaking changes are needed" follow-up — have an answer ready: deprecation headers (`Deprecation: true`, `Sunset: <date>`), versioned changelog, backward-compatible changes preferred (adding fields is safe; removing fields requires a new version).

- On pagination: mention that you chose cursor pagination for Smart360's data service because the underlying dataset had frequent inserts and offset pagination would return inconsistent pages. This shows you thought about the data access pattern, not just the API pattern.

- On system design: when presenting the URL shortener, proactively say: "I want to flag the read-your-writes consistency issue: if we write the URL and the cache is populated asynchronously, a user who clicks their own shortened link within milliseconds of creation might miss. The fix is to populate the cache synchronously on write, accepting a tiny write-path latency increase." Catching your own design's edge cases before being asked = senior signal.

**Soft skills**

- In every behavioral answer, end with what you'd do differently. "I'd do it the same way" is a growth signal killer.
- When an interviewer says "interesting, can you say more about X" — that means X is the thing they care about. Slow down, give depth, don't rush to close the answer.
- Before answering any system design question: spend 2 minutes asking clarifying questions about scale, SLAs, and constraints. Interviewers mark you up for this even if you then design the wrong thing.

---

## 📊 End-of-Week Self-Assessment

Score yourself after Sunday. Be honest — this is for your benefit, not for anyone else.

| Skill | 1 (shaky) | 2 (can do with hints) | 3 (solid, unaided) | 4 (could teach it) |
|---|---|---|---|---|
| Valid Parentheses / Min Stack | | | | |
| Daily Temperatures (monotonic stack) | | | | |
| Largest Rectangle in Histogram | | | | |
| Reverse / Merge Linked Lists | | | | |
| Linked List Cycle / Reorder List | | | | |
| LRU Cache (full DLL impl) | | | | |
| REST resource naming + status codes | | | | |
| REST idempotency + idempotency keys | | | | |
| API versioning + pagination trade-offs | | | | |
| JWT issuance → validation → refresh chain | | | | |
| OAuth2 grant types (when to use which) | | | | |
| RBAC implementation (Smart360 context) | | | | |
| `@ControllerAdvice` + Bean Validation | | | | |
| URL shortener system design (full doc) | | | | |

**Threshold for Phase 2 readiness**: no more than 2 skills at level 1. If you have ≥3 at level 1, spend one extra day before starting Week 5.

**Write here before closing this file:**
- My 3 weakest areas this week: ______, ______, ______
- Phase 2 first priority: ______
- Confidence level 1–10 going into system design week: ______

---

*Week 4 of the 2026 interview prep plan. Build on: interview-qa.md (JWT/RBAC, Spring, microservices Q&A already covered — don't re-drill what you scored 3–4 on). Phase 2 starts Week 5.*
