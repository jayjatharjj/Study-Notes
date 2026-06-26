# Week 2 · Day 5 — Fri Jul 10 — Mixed DSA Review + JWT / OAuth2 + Global Exception Handling

> Friday is a pressure test. Block A re-attempts the week's hardest problems cold and timed, the way a real interview feels — no hints, talking aloud. Block B is the security backbone every backend role probes: how a JWT is issued, validated, refreshed, and revoked, where OAuth2 grants fit, and how a single `@RestControllerAdvice` turns raw exceptions into clean, consistent API errors. Block C drills the exact points you got stuck on this morning.

📌 **Study today:** Mixed DSA review (re-attempt the week's hardest cold) · JWT lifecycle + OAuth2 + global exception handling (`@ControllerAdvice`) · weak-spot drills · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Mixed Review (timed, no hints)

### How to run this block

Treat each problem as a live interview. Set a timer, narrate your approach *before* you type, state complexity unprompted, and handle edge cases without being asked. For any problem you get wrong or solve slowly, write down **exactly** where you stalled — that line becomes a Block C drill.

### Review set (20–25 min each, talking aloud)

1. **Search in Rotated Sorted Array (LC 33)** — target ≤10 min.
2. **Koko Eating Bananas (LC 875)** — binary-search-on-answer; target ≤10 min.
3. **Largest Rectangle in Histogram (LC 84, Hard)** — re-derive the monotonic-stack invariant.
4. **Reorder List (LC 143)** — the three-step pattern (midpoint → reverse → merge).

### Worked re-derivation — Search in Rotated Sorted Array (LC 33)

The trick: at any `mid`, **one half is always sorted**. Compare `nums[mid]` to `nums[left]` to find which half is sorted, then check whether the target lies within that sorted half's range — an O(1) decision per step.

```java
public int search(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;   // overflow-safe midpoint
        if (nums[mid] == target) return mid;
        if (nums[left] <= nums[mid]) {         // LEFT half is sorted
            if (nums[left] <= target && target < nums[mid]) right = mid - 1;
            else left = mid + 1;
        } else {                               // RIGHT half is sorted
            if (nums[mid] < target && target <= nums[right]) left = mid + 1;
            else right = mid - 1;
        }
    }
    return -1;
}
```
**Complexity:** O(log n) / O(1). **Edge cases:** single element; target absent; rotation point at index 0 (fully sorted array). **Follow-up (LC 81, duplicates):** when `nums[left]==nums[mid]==nums[right]` you can't tell which half is sorted — do `left++; right--`, degrading worst-case to O(n).

### Worked re-derivation — Largest Rectangle in Histogram (LC 84)

Maintain an **increasing** monotonic stack of indices. When the current bar is shorter than the stack top, that top bar can't extend further right, so pop it and compute its maximal rectangle. A trailing 0-height sentinel drains the stack at the end.

```java
public int largestRectangleArea(int[] heights) {
    Deque<Integer> stack = new ArrayDeque<>(); // indices, heights increasing
    int n = heights.length, max = 0;
    for (int i = 0; i <= n; i++) {
        int h = (i == n) ? 0 : heights[i];     // sentinel drains the stack
        while (!stack.isEmpty() && heights[stack.peek()] >= h) {
            int height = heights[stack.pop()];
            int width  = stack.isEmpty() ? i : i - stack.peek() - 1;
            max = Math.max(max, height * width);
        }
        stack.push(i);
    }
    return max;
}
```
**Complexity:** O(n) / O(n) — each index pushed and popped once. Trace `[2,1,5,6,2,3]` → 10. **Common mistake:** forgetting the sentinel so bars left on the stack are never measured.

### Scoring rubric

For each problem, score yourself: Did I state the approach before coding? Handle edge cases unprompted? Finish in time? Was the code compilable without fixes? Write one line on what to do differently.

---

## Block B — JWT + OAuth2 + Global Exception Handling

### JWT structure

A JWT is `header.payload.signature`, each part base64url-encoded. **Only the signature is secure** — the payload is *readable by anyone* (base64, not encryption). Never put secrets in a JWT. The signature is what proves the token wasn't tampered with.

- **HS256** — HMAC with a shared secret. Fast, but every verifier needs the secret (fine for a monolith).
- **RS256** — RSA: sign with a private key, verify with the public key. Use for **multi-service** systems so verifiers never hold the signing key.

### Token issuance

```java
public TokenPair issueTokens(User user) {
    Instant now = Instant.now();
    String access = Jwts.builder()
        .subject(user.getId().toString())
        .claim("roles", user.getRoles())
        .claim("tenantId", user.getTenantId())
        .issuedAt(Date.from(now))
        .expiration(Date.from(now.plus(Duration.ofMinutes(15)))) // short-lived
        .signWith(key, Jwts.SIG.HS256)
        .compact();
    String refresh = secureRandomToken();           // opaque, not a JWT
    refreshStore.save(hash(refresh), user.getId(),   // store HASHED, with a token family
                      tokenFamilyId, Duration.ofDays(7));
    return new TokenPair(access, refresh, 900);
}
```
Flow: POST credentials → `AuthService` validates with **BCrypt** → build claims (`sub`, `roles`, `tenantId`, `iat`, `exp` ≈ 15 min) → sign → return `{accessToken, refreshToken, expiresIn}`. The refresh token is stored **hashed** in DB/Redis with a `tokenFamily` so a replayed old token signals theft.

### Validation (every protected request)

```java
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res,
                                    FilterChain chain) throws ServletException, IOException {
        String header = req.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);
            try {
                Claims claims = Jwts.parser()
                    .verifyWith(key)            // pin the algorithm — never trust token's alg
                    .build().parseSignedClaims(token).getPayload();
                if (!blacklist.contains(claims.getId())) { // jti check, O(1) Redis
                    var auth = toAuthentication(claims);
                    SecurityContextHolder.getContext().setAuthentication(auth);
                }
            } catch (JwtException e) { /* leave context empty → 401 downstream */ }
        }
        chain.doFilter(req, res);
    }
}
```
Validation is **local** — verify signature + `exp` with no DB call. That stateless verification is the ~60% latency win in Smart360 vs. a session lookup per request. The only optional lookup is an O(1) Redis blacklist check for revoked `jti`s.

### Refresh & rotation

POST the refresh token → validate it (DB/Redis lookup) → issue a new access token **and rotate the refresh token** (issue a new one, invalidate the old). **Token-family rotation** detects theft: if an already-used (rotated-out) refresh token is replayed, the whole family is revoked — both the attacker and the legitimate user are logged out, which is the safe outcome.

### RBAC at three levels (defense-in-depth)

| Level | Mechanism | What it protects |
|---|---|---|
| API | `@PreAuthorize("hasAuthority('READ_REPORTS')")` | The endpoint |
| Service | Explicit role check in the service method | Catches internal callers that bypass the controller |
| Database | Postgres Row-Level Security / `findByIdAndTenantId` | Cross-tenant leaks even from a compromised service |

If you only had API-level checks, a routing bug or a new internal caller bypasses them. Service-level is defense-in-depth. **DB RLS** means even a fully compromised service cannot read another tenant's rows.

### OAuth2 grant types

| Grant | Use | Note |
|---|---|---|
| Authorization Code + **PKCE** | User-facing web/mobile | Most secure; PKCE prevents auth-code interception. **Default choice.** |
| Client Credentials | Service-to-service (no user) | Machine identity |
| Implicit | — | **Deprecated** — never recommend (tokens in URL fragment) |
| Resource Owner Password | Legacy first-party only | Avoid — app handles the raw password |

### Token storage & attacks

- **Access token** in JS memory (a variable), **not** `localStorage` (XSS reads it).
- **Refresh token** in an `HttpOnly; Secure; SameSite=Strict` cookie (JS can't read it; CSRF mitigated).
- **Algorithm-confusion attack:** an attacker sets the token header to `alg: none` or swaps RS256→HS256 (signing with the *public* key as an HMAC secret). **Always pin the expected algorithm in the parser** (`verifyWith(key)` above) — never trust the token's own `alg` header.

### Logout on stateless JWT

You can't un-issue a signed token. Add its `jti` to a **Redis blacklist** with `TTL = remaining lifetime`, and delete the refresh token. Each request checks the blacklist (O(1)). Spring Security filter-chain order: `SecurityContextPersistenceFilter` → `UsernamePasswordAuthenticationFilter` → custom `JwtAuthenticationFilter` → `ExceptionTranslationFilter` → `AuthorizationFilter` (the modern replacement for `FilterSecurityInterceptor`).

> **WebFlux note:** `SecurityContextHolder` is `ThreadLocal`-backed and breaks in reactive code (work hops threads). Use `ReactiveSecurityContextHolder`, which stores auth in the Reactor `Context` carried along the `Mono`/`Flux` chain.

### Global exception handling — write from memory

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
        List<String> errors = ex.getBindingResult().getFieldErrors().stream()
            .map(fe -> fe.getField() + ": " + fe.getDefaultMessage()).toList();
        return ResponseEntity.badRequest()
            .body(new ErrorResponse("VALIDATION_ERROR", errors.toString()));
    }

    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleForbidden(AccessDeniedException ex) {
        return ResponseEntity.status(HttpStatus.FORBIDDEN)
            .body(new ErrorResponse("FORBIDDEN", "Insufficient permissions"));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        log.error("Unhandled exception", ex);  // log FULLY server-side
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred"));
    }
}

public record ErrorResponse(String code, String message) {}
```

**Where it intercepts:** `@ControllerAdvice`/`@ExceptionHandler` is **not** a filter and not a servlet — `DispatcherServlet` catches the exception the handler threw and delegates to `HandlerExceptionResolver` → `ExceptionHandlerExceptionResolver` → your `@ExceptionHandler`. It runs *after* the controller throws, *inside* the dispatch. (Exceptions thrown earlier in the `SecurityFilterChain`, e.g. a bad JWT, are handled by `ExceptionTranslationFilter`, not by `@ControllerAdvice`.)

**Why the generic handler never returns `ex.getMessage()`:** stack traces and internal messages leak implementation details (class names, SQL, file paths) that help an attacker. Log fully server-side; return a generic, stable message + an error code to the client. Modern alternative: extend `ResponseEntityExceptionHandler` and return RFC 7807 `ProblemDetail`.

### Bean Validation

`@NotNull`, `@NotBlank` (non-null + non-whitespace), `@NotEmpty` (collection/string size > 0), `@Size`, `@Min`/`@Max`, `@Email`, `@Pattern`. `@Valid` on a controller parameter **triggers** validation before the method body runs; a failure raises `MethodArgumentNotValidException` automatically (handled above). `@Validated` (Spring) adds validation **groups**. Custom validator: `@Constraint(validatedBy = MyValidator.class)` on the annotation, implementing `ConstraintValidator<MyAnnotation, T>`.

```java
public record CreateUserRequest(
    @NotBlank String name,
    @Email String email,
    @Size(min = 8) String password) {}

@PostMapping("/users")
public ResponseEntity<?> create(@Valid @RequestBody CreateUserRequest req) { /* ... */ }
```

---

## Block C — Weak-Spot Drills

1. Re-solve, from a blank file, each specific stuck point you captured in Block A. Don't peek at the morning's solution.
2. If those are clean, attempt **Spiral Matrix (LC 54)** — boundary-shrinking simulation, a common Amazon/Google warm-up.

```java
public List<Integer> spiralOrder(int[][] m) {
    List<Integer> out = new ArrayList<>();
    int top = 0, bottom = m.length - 1, left = 0, right = m[0].length - 1;
    while (top <= bottom && left <= right) {
        for (int c = left; c <= right; c++) out.add(m[top][c]);
        top++;
        for (int r = top; r <= bottom; r++) out.add(m[r][right]);
        right--;
        if (top <= bottom) { for (int c = right; c >= left; c--) out.add(m[bottom][c]); bottom--; }
        if (left <= right) { for (int r = bottom; r >= top; r--) out.add(m[r][left]); left++; }
    }
    return out;
}
```
The two `if` guards after the horizontal/vertical sweeps are the bug magnet — they prevent re-reading a row/column in a non-square matrix.

---

## 💻 Practice coding questions

1. Search in Rotated Sorted Array (LC 33) — cold, ≤10 min.
2. Search in Rotated Sorted Array II with duplicates (LC 81) — handle the `left==mid==right` case.
3. Koko Eating Bananas (LC 875) — binary-search-on-answer, ≤10 min.
4. Largest Rectangle in Histogram (LC 84) — re-derive the stack invariant.
5. Reorder List (LC 143) — three-step pattern.
6. Spiral Matrix (LC 54) — boundary shrinking.
7. Write a `JwtAuthenticationFilter extends OncePerRequestFilter` that validates a token and sets `SecurityContextHolder`.
8. Write a `@RestControllerAdvice` with handlers for not-found, validation, forbidden, and generic.
9. Implement an idempotent token-refresh endpoint with family rotation.
10. Write a custom `ConstraintValidator` (e.g. `@StrongPassword`).
11. Implement a Redis-backed `jti` blacklist check used by the filter.
12. Build an `ErrorResponse`/RFC 7807 `ProblemDetail` mapper for `MethodArgumentNotValidException`.

---

## 🎤 Interview questions

1. **What are the three parts of a JWT and which is secure?** `header.payload.signature`; only the signature. The payload is base64, not encrypted — readable by anyone.
2. **HS256 vs RS256 — when each?** HS256 (shared secret) for a monolith; RS256 (private sign / public verify) for multi-service so verifiers never hold the signing key.
3. **Walk me through token validation on a protected request.** Extract Bearer token → verify signature + `exp` locally (no DB) → optional O(1) blacklist check → set `SecurityContextHolder`.
4. **Why is local validation a latency win?** No per-request DB/session lookup; signature verification is CPU-only. ~60% latency reduction in Smart360.
5. **How do you handle logout with a stateless JWT?** Add `jti` to a Redis blacklist with TTL = remaining lifetime; delete the refresh token; check the blacklist per request.
6. **What is refresh-token rotation and why?** Issue a new refresh token on each use and invalidate the old; a replayed old token (a token family) signals theft → revoke the whole family.
7. **Where should the access vs refresh token live in a browser?** Access in JS memory (XSS-resistant vs `localStorage`); refresh in an `HttpOnly; Secure; SameSite=Strict` cookie.
8. **Explain the algorithm-confusion attack and the fix.** Attacker forces `alg: none` or RS256→HS256; fix is to pin the expected algorithm in the parser and never trust the token's `alg`.
9. **OAuth2 grant types — which for a mobile app, which for service-to-service?** Authorization Code + PKCE for mobile; Client Credentials for service-to-service. Avoid Implicit (deprecated) and Password (legacy).
10. **What does PKCE protect against?** Authorization-code interception on public clients (no client secret).
11. **RBAC at API/service/DB — what breaks with only one?** API-only is bypassed by a bug/new caller; service adds defense-in-depth; DB RLS protects against a compromised service leaking cross-tenant data.
12. **Where does `@ControllerAdvice` intercept in the request flow?** After the controller throws, inside `DispatcherServlet` via `HandlerExceptionResolver` → `ExceptionHandlerExceptionResolver`. Not a filter.
13. **A bad JWT throws before the controller — does `@ControllerAdvice` catch it?** No — it's in the `SecurityFilterChain`; `ExceptionTranslationFilter` handles it and returns 401/403.
14. **Why must the generic exception handler never return `ex.getMessage()`?** It leaks internals (SQL, class names, paths) useful to an attacker; log server-side, return a generic message + code.
15. **What status for a validation failure — 400 or 422?** Defensible either way; 400 if you treat the body as a malformed request, 422 if syntactically valid but semantically wrong. Pick one and be consistent.
16. **What triggers `MethodArgumentNotValidException`?** `@Valid` on a `@RequestBody` parameter when a Bean Validation constraint fails.
17. **`@Valid` vs `@Validated`?** `@Valid` is the JSR-380 trigger (incl. cascading); `@Validated` is Spring's variant that adds validation groups and works on method-level validation.
18. **How do you write a custom validation rule?** A `@Constraint(validatedBy=...)` annotation + a `ConstraintValidator<A, T>` implementation.
19. **Why does Spring Security break in WebFlux and how is it fixed?** `SecurityContextHolder` uses `ThreadLocal`; reactive chains hop threads. Use `ReactiveSecurityContextHolder` (Reactor `Context`).
20. **What's RFC 7807 and why use it?** A standard `ProblemDetail` error format (`type`, `title`, `status`, `detail`, `instance`); consistent machine-readable errors across services.

---

## ✅ Self-check

1. What status should a validation failure return — 422 or 400? Have an opinion and defend it.
2. Why must the generic `Exception` handler never return the exception message to the client?
3. Draw the filter-chain order from request to controller and name where JWT validation and exception translation occur.
4. From memory, write the `@RestControllerAdvice` with four handlers, compilable on the first try.

---

*Nav: ← [Day 4 (Thu Jul 09)](04-thu-jul-09.md) · [Week 2](README.md) · [Day 6 (Sat Jul 11)](06-sat-jul-11.md) →*
