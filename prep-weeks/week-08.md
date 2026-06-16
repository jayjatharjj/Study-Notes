# Week 8 — Core Build + First Applications (Aug 3–9, 2026)

> Ship code AND ship applications. You've spent seven weeks sharpening the blade; this week you swing it. Every DSA session closes a gap in DP/Greedy. Every Cloud block turns a resume bullet into a whiteboard-ready story. By Friday your LinkedIn is recruiter-optimised and three warm-up interviews are booked.

---

## 🎯 Week Goal

1. Solve all 9 target LeetCode problems cleanly on first attempt with optimal time/space complexity.
2. Be able to whiteboard — without notes — a full deployment pipeline for Deep Fathom (Azure Container Apps + K8s + Bicep + GitLab CI/CD) and field any curveball on it.
3. Publish the polished LinkedIn/Naukri profiles and submit 3–5 warm-up applications with a tracking sheet live.

---

## ✅ By Sunday you can...

- Implement Edit Distance (LC 72), Longest Palindromic Substring (LC 5), Maximal Square (LC 221), and Partition Equal Subset Sum (LC 416) from a blank editor — explaining the recurrence aloud as you type.
- Implement Jump Game (LC 55 & 45), Gas Station (LC 134), Merge Intervals (LC 56), Insert Interval (LC 57), and Non-overlapping Intervals (LC 435) in ≤ 15 min each.
- Narrate the Deep Fathom Azure architecture story for 4–6 minutes (problem → design decisions → metrics) with zero filler.
- Explain Docker multi-stage builds and K8s rolling-update mechanics at whiteboard speed.
- Quote the 57% CI/CD pipeline cut and tie it concretely to the DAG pipeline, parallel builds, and BuildKit caching.
- Walk through Key Vault secret injection in Azure Container Apps without referencing notes.
- Articulate why you chose Bicep over Terraform/Pulumi for this project.
- Recite your "extraordinary candidate" differentiator pitch in under 90 seconds.

---

## 📅 Daily Checklist

### Monday, Aug 3 — 2D DP: Edit Distance & Palindromes

**DSA (45 min)**
- [ ] Read the DP recurrence for Edit Distance intuitively: `dp[i][j]` = min ops to convert `word1[0..i]` to `word2[0..j]`.
  - Insert: `dp[i][j-1] + 1`; Delete: `dp[i-1][j] + 1`; Replace: `dp[i-1][j-1] + (word1[i] != word2[j])`
- [ ] Solve **LC 72 – Edit Distance** (Hard). Target: O(mn) time, O(n) space with rolling array optimisation.
- [ ] Solve **LC 5 – Longest Palindromic Substring** (Medium). Two approaches: expand-around-centre O(n²) and Manacher's O(n). Know both; implement expand-around-centre first. Understand why the DP table (`dp[i][j] = is s[i..j] a palindrome`) also works in O(n²).

**Core Block (30 min) — Azure Container Apps Deep Dive**
- [ ] Narrate the Deep Fathom deployment story aloud (kitchen table, no notes). Hit: multi-service ACA environment, Bicep provisioning, Key Vault references via managed identity, ingress rules, DAPR optional.
- [ ] Be able to answer: "Why Container Apps over AKS here?" (scale-to-zero cost, no node management, adequate for the workload) vs. "When would you flip to AKS?" (custom operators, specific node SKUs, complex network policies).
- [ ] Memorise: Azure Container Apps uses KEDA under the hood for scaling; this is a genuine detail that impresses interviewers.

**Self-Check**
- [ ] Can you explain Edit Distance DP state transition in one breath without hesitation?
- [ ] Can you name three concrete reasons you'd pick ACA over AKS?

---

### Tuesday, Aug 4 — 2D DP: Maximal Square & Subset Sum

**DSA (45 min)**
- [ ] Solve **LC 221 – Maximal Square** (Medium). Key insight: `dp[i][j]` = side length of largest square with bottom-right corner at (i,j). Recurrence: `min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1` if `matrix[i][j] == '1'`. O(mn) time, O(n) space.
- [ ] Solve **LC 416 – Partition Equal Subset Sum** (Medium). Reduction to 0/1 knapsack: target = sum/2; `dp[j]` = can we form sum `j` using items so far. Iterate items outer, sums inner (right to left to avoid reuse). O(n * target) time, O(target) space.
- [ ] Write out both solutions without hints. If stuck >15 min, look at the recurrence only — not the code.

**Core Block (30 min) — Docker Multi-Stage Builds**
- [ ] Study/recall your GitLab CI/CD pipeline. Be ready to explain a 2-stage Dockerfile: `FROM eclipse-temurin:17-jdk AS build` → Maven build → `FROM eclipse-temurin:17-jre` → copy fat jar. Why: final image contains no JDK, Maven cache, or source code — 60–70% smaller image.
- [ ] Know BuildKit flags: `DOCKER_BUILDKIT=1`, `--cache-from registry/image:cache`, `--build-arg BUILDKIT_INLINE_CACHE=1`. Tie to the 57% pipeline drop: unchanged layers cache-hit → no recompile.
- [ ] Practice explaining the GitLab DAG pipeline (`needs:` keyword) vs sequential stages. Draw it: test-unit → [test-integration, build-image] run in parallel → deploy. That parallel fan-out is the architectural reason for the time drop.

**Self-Check**
- [ ] Can you write Maximal Square's recurrence on a whiteboard in 30 seconds?
- [ ] Can you describe the Docker multi-stage Dockerfile from memory, with the exact FROM lines?

---

### Wednesday, Aug 5 — Greedy: Jump Game & Gas Station

**DSA (45 min)**
- [ ] Solve **LC 55 – Jump Game** (Medium). Greedy: track `maxReach`; if current index > `maxReach`, return false. O(n) time, O(1) space. Know why DP is O(n²) and strictly worse here.
- [ ] Solve **LC 45 – Jump Game II** (Medium). Greedy: track current end, furthest reachable, jumps. Increment jumps when you reach `currentEnd`. O(n), O(1).
- [ ] Solve **LC 134 – Gas Station** (Medium). Key insight: if total gas ≥ total cost, a solution exists; start tracking from the point where running sum went negative. O(n), O(1). Prove to yourself why you can discard everything before the reset point.

**Core Block (30 min) — Kubernetes Deep Dive**
- [ ] Be able to whiteboard: Pod → ReplicaSet → Deployment relationship. Why you never create pods directly.
- [ ] Explain rolling update mechanics: `maxSurge` (extra pods allowed), `maxUnavailable` (pods that can be down). Default is 25% each.
- [ ] Know readiness vs liveness probes cold:
  - Liveness: is the process alive? Failure → restart pod. Example: Spring's `/actuator/health/liveness`.
  - Readiness: is the pod ready to receive traffic? Failure → remove from Service endpoints. Example: `/actuator/health/readiness`. This is what prevents traffic to a pod that's still warming up (connection pool, cache hydration).
  - Startup probe: for slow-starting containers — gives extra time before liveness kicks in.
- [ ] Know `spring.lifecycle.timeout-per-shutdown-phase=30s` and `server.shutdown=graceful` — mandatory for zero-downtime. K8s sends SIGTERM → app stops accepting new requests → finishes in-flight → exits.
- [ ] Know `terminationGracePeriodSeconds` on the Pod spec and that it must exceed the Spring shutdown timeout.

**Self-Check**
- [ ] Can you explain why Gas Station's greedy works — not just what it does?
- [ ] Can you describe a full zero-downtime K8s rolling deploy, including Spring config, in 2 minutes?

---

### Thursday, Aug 6 — Greedy: Intervals

**DSA (45 min)**
- [ ] Solve **LC 56 – Merge Intervals** (Medium). Sort by start; merge overlapping by comparing `end` of last merged vs `start` of next. O(n log n) time, O(n) space.
- [ ] Solve **LC 57 – Insert Interval** (Medium). Three phases: collect all before new interval, merge overlapping (extend by max end), collect remaining. O(n), no sort needed since already sorted. This is a common FAANG variant — practice it until it feels mechanical.
- [ ] Solve **LC 435 – Non-overlapping Intervals** (Medium). Reframe as: max non-overlapping intervals = n − (max intervals to keep). Sort by end time; greedy: always keep the interval with earliest end. O(n log n), O(1).
- [ ] Spot the pattern: Interval problems → sort by start OR end (know which and why) → single pass greedy.

**Core Block (30 min) — AWS Talking Points (S3 + EC2)**
- [ ] Rehearse Smart360 AWS story: S3 for storing user-uploaded files + pre-signed URL generation (time-bounded, no public bucket exposure). The performance win: caching pre-signed URLs in Redis with TTL slightly less than S3 expiry → 80% S3 API call reduction.
- [ ] Know S3 access patterns: bucket policies vs IAM roles vs pre-signed URLs. Know when to use each. Pre-signed URL: temporary, no credentials in client. Bucket policy: broad allow for static hosting. IAM role: for EC2/Lambda server-side access.
- [ ] Know EC2 basics for conversation: instance types (t3/m6i/c6i families), auto-scaling groups, application load balancer target groups. Know when you'd reach for EC2 vs Lambda vs Container (ECS/EKS).
- [ ] Tie AWS to Azure: "In Smart360 we used AWS; in Deep Fathom we moved to Azure because the client had an enterprise Azure agreement. Key Vault replaced AWS Secrets Manager; Azure Container Apps replaced ECS; Bicep replaced CloudFormation."

**Self-Check**
- [ ] Can you solve all three interval problems back-to-back without looking anything up?
- [ ] Can you clearly explain the pre-signed URL caching strategy and its tradeoffs?

---

### Friday, Aug 7 — Mock Story Day + Profile Polish Begins

**DSA (30 min — lighter day)**
- [ ] Revisit any one problem from Mon–Thu that felt shaky. Do it again from scratch, no hints, time yourself.
- [ ] Write time/space complexity for all 9 week problems on a single sheet. Review it like flashcards.

**Core Block (30 min) — Bicep IaC & Full Story Rehearsal**
- [ ] Rehearse the Bicep story: "I wrote 30+ Azure resources as modular Bicep — Container Apps environments, PostgreSQL Flexible Server, Redis Cache, Key Vault, Front Door CDN, VNet, private endpoints. Each environment (dev/staging/prod/GovCloud) is one parameter file away. This eliminated environment drift and enabled reproducible FedRAMP-aligned infrastructure."
- [ ] Know Bicep vs Terraform trade-off answer: "Bicep is first-class Azure — tighter integration, no state file to manage, same-day support for new Azure features. Terraform's HCL is cloud-agnostic but lags on new Azure APIs. For a pure-Azure project, Bicep is the pragmatic choice." Know you'd use Terraform if the stack was multi-cloud.
- [ ] Rehearse the full 6-minute "Deep Fathom Architecture" talk-track end-to-end. Cover: problem, tech choices, CI/CD, K8s probes, Key Vault integration, Bicep, outcome metrics.

**Profile Polish (30 min)**
- [ ] Open LinkedIn. Rewrite the headline: `Full Stack Engineer · Java 17 · Spring Boot · Microservices · Azure · Kubernetes · LLM Integration`. Keywords recruiters search.
- [ ] Rewrite the About section (3–4 sentences): quantified impact first (60s→3s query, 57% CI/CD cut, 30+ Bicep resources), tech stack, what you're looking for.
- [ ] Ensure Smart360 and Deep Fathom experience entries have 3–5 bullets each, all starting with an action verb, all with a number. Example: "Reduced P95 query latency from 60 s to 2–3 s by eliminating N+1 patterns with JOIN FETCH and EntityGraph, cutting DB load by ~70%."
- [ ] Add Skills section: Java · Spring Boot · Spring Cloud · Microservices · Azure · AWS · Kubernetes · Docker · PostgreSQL · Redis · GitLab CI/CD · Bicep · Vue.js · React · LLM Integration.

**Self-Check**
- [ ] Is your LinkedIn headline under 220 characters and keyword-dense?
- [ ] Does every experience bullet have a number?

---

### Saturday, Aug 8 — Full Applications Day

**DSA (60 min)**
- [ ] Full timed mock: pick any 2 problems from this week you haven't fully nailed. 30 min each, LeetCode contest mode (no hints, fresh editor).
- [ ] If both pass: pick one Hard from Blind 75 DP list (LC 115 Distinct Subsequences or LC 312 Burst Balloons) for stretch challenge.

**Profile & Applications (3 hr)**

*Naukri Polish (45 min)*
- [ ] Update headline/summary to mirror LinkedIn.
- [ ] Upload latest resume (ensure PDF exports cleanly — one page if possible, two max).
- [ ] Set "Actively Looking" / open-to-work status.
- [ ] Add all skills with proficiency levels. Naukri's keyword algorithm indexes your skills section heavily.
- [ ] Set salary expectation band to target (research: Java + Spring Boot + 2.5 YOE + Bangalore/remote median is ₹14–18 LPA at mid-size product; ₹18–25 LPA at GCC — set 20 LPA as minimum to avoid filtering).

*Application Tracking Sheet (15 min)*
- [ ] Create a Google Sheet with columns: Company | Role | JD URL | Applied Date | Source | Status | Next Step | Contacts | Notes.
- [ ] This sheet is your operating system. Update it after every action.

*Research & Apply — 3–5 Warm-Up Companies (2 hr)*
- [ ] Target criteria: mid-size product companies or established service companies (NOT the GCC dream targets — those come later). Goal is interview reps, not offers.
- [ ] Suggested hunting grounds: Naukri ("Java Spring Boot Microservices" → filter 2–5 YOE, 15–25 LPA, Pune/Bengaluru/remote), LinkedIn Jobs, Instahyre, Cutshort.
- [ ] For each application: tailor the cover note / application message in 3 sentences (what you did, tech match to JD, what you want).
- [ ] Log all 3–5 in the tracking sheet before you close the laptop.

**Self-Check**
- [ ] Is the tracking sheet live with at least 3 rows?
- [ ] Did you verify each company's tech stack before applying (don't apply to .NET shops)?

---

### Sunday, Aug 9 — Weekly Review + Week 9 Prep

**DSA (60 min)**
- [ ] Full DP review pass: write the recurrence (not the code, just the state and transition) for all 4 DP problems this week on paper. If any recurrence is fuzzy, re-solve that problem.
- [ ] Solve **LC 300 – Longest Increasing Subsequence** (if not done previously) as a connective DP problem — reinforces the pattern of "dp[i] depends on all j < i where condition holds."

**Core Block (90 min) — End-of-Week Mock Interview**
- [ ] Do a 45-min self-mock: pick 5 questions from the Sample Questions section below and answer them aloud, timed, as if on a Zoom call. Record yourself on your phone if possible — watch it back.
- [ ] Do a 30-min whiteboard: draw the Deep Fathom deployment architecture on paper (not notes). Check it matches reality.
- [ ] Spend 15 min reviewing what felt weak in the mock → add those topics to Week 9's focus list.

**Planning (30 min)**
- [ ] Review the tracking sheet. Any recruiter responses? Schedule follow-ups.
- [ ] Plan Week 9 DSA focus (Trees: BFS/DFS, LCA, serialization — or Graphs if Trees are solid).
- [ ] Write 2–3 things you learned this week that you didn't know last Sunday.

---

## 🧠 Concepts to Master This Week

### DP — 2D and Subset Patterns

**Edit Distance (LC 72)**
- State: `dp[i][j]` = min edit operations to convert `word1[0..i-1]` to `word2[0..j-1]`.
- Base: `dp[i][0] = i` (delete i chars), `dp[0][j] = j` (insert j chars).
- Transition: if chars equal, `dp[i][j] = dp[i-1][j-1]`; else `1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])`.
- Space opt: only need previous row → O(n) space with a rolling array.

**Longest Palindromic Substring (LC 5)**
- Expand-around-centre: for each index, expand both odd-length and even-length centres. O(n²) time, O(1) space. This is the interview-preferred approach.
- Know that Manacher's is O(n) and what it does conceptually (transforms string, tracks mirror property) — you don't need to implement it under pressure, but saying "Manacher's gives O(n); here's why it works at a high level" is an extraordinary-candidate signal.

**Maximal Square (LC 221)**
- Recurrence: `dp[i][j] = min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1` when `matrix[i][j] == '1'`.
- Intuition: the square's side is bottlenecked by its three neighbours. Prove this with a 2×2 example.
- Answer area = `(max dp[i][j])²`.

**Partition Equal Subset Sum (LC 416)**
- 0/1 Knapsack: `dp[j]` = true if we can form sum `j`. Iterate nums outer, sums from `target` down to `num[i]` inner (prevents reuse). Terminate early if `dp[target]` becomes true.
- Edge cases: odd total sum → false immediately. Num > target → false.

### Greedy Patterns

**Jump Game family**: always track the farthest reachable index. Don't simulate; track the frontier.

**Gas Station**: two key facts to internalise:
1. If total gas ≥ total cost, a solution always exists and is unique.
2. If running sum goes negative at index `k`, the starting station must be after `k` — all stations from 0 to k are disqualified as starts.

**Interval problems**: sort order determines correctness. For merge/insert: sort by start. For non-overlapping (greedy keep): sort by end. Know why.

### Cloud/DevOps Architecture

**Docker Multi-Stage Build** — mental model:
```
Stage 1 (build): JDK + Maven + source → fat jar
Stage 2 (runtime): JRE only + fat jar → final image
```
Key result: no JDK, no source, no Maven cache in prod image. Size goes from ~600 MB → ~180 MB typical. Security surface reduced.

**BuildKit Caching** — key flags to know:
- `DOCKER_BUILDKIT=1` or `docker buildx build`
- `--cache-from type=registry,ref=registry/image:cache` — pull layer cache from registry
- `--cache-to type=registry,ref=registry/image:cache,mode=max` — push updated cache
- `RUN --mount=type=cache,target=/root/.m2` — mount Maven cache across builds without baking it into the layer

**Kubernetes Zero-Downtime Rolling Update** — 6-step mental model:
1. `kubectl set image deployment/app container=new-image:tag` triggers rollout.
2. K8s creates new pod(s) (controlled by `maxSurge`).
3. New pod passes readiness probe → added to Service endpoints.
4. Old pod removed from Service endpoints.
5. Old pod receives SIGTERM → Spring graceful shutdown drains in-flight requests.
6. Old pod terminates. Repeat until all pods replaced.

Spring config required: `server.shutdown=graceful`, `spring.lifecycle.timeout-per-shutdown-phase=30s`. Pod spec: `terminationGracePeriodSeconds: 60` (must exceed Spring timeout).

**Azure Key Vault Integration** — the production pattern:
1. Assign a managed identity to the Container App.
2. Grant the identity `Key Vault Secrets User` role on the Key Vault.
3. In Bicep: reference secret as `secretRef` pointing to Key Vault URI — no credentials in code or env vars.
4. At runtime, ACA's control plane fetches the secret and injects it as an env var. Secret rotation happens without redeployment (only a pod restart is needed).

**Bicep Modular Design** — explain it structurally:
- Root `main.bicep` passes parameters to modules: `network.bicep`, `containerApps.bicep`, `database.bicep`, `security.bicep`.
- Each environment (dev/staging/prod) has its own `parameters.{env}.json`.
- This structure enables: PR-based reviews of infra changes, staged rollouts, and reproducible GovCloud deployments with different parameter values but identical topology.

**GitLab CI/CD DAG Pipeline** — the architecture that drove 57% cut:
```yaml
stages: [test, build, deploy]

unit-test: { stage: test }
integration-test: { stage: test, needs: [] }  # parallel with unit-test
build-image:
  stage: build
  needs: [unit-test, integration-test]         # waits for both, then runs
  script: docker buildx build --cache-from ...
deploy-staging:
  stage: deploy
  needs: [build-image]
```
Before: stages were sequential (23 min). After: DAG parallelism + BuildKit cache (10 min).

---

## 🎤 Sample Interview Questions (incl. Curveballs) — 8–15 Qs + Pointers

### DP/Greedy

**1. "Walk me through Edit Distance. What is the recurrence and why does it work?"**
- Pointer: State the sub-problem clearly first ("minimum ops to convert prefix of length i to prefix of length j"). Then give the recurrence with the three cases. Mention the space optimisation. If they push: discuss how this extends to Longest Common Subsequence (LCS) — set replace cost to ∞.

**2. "Can Partition Equal Subset Sum be solved greedily? Why or why not?"**
- Pointer: No. Greedy fails because a locally "best" choice (take the largest item) doesn't guarantee a valid partition. Counterexample: [3, 3, 3, 4, 1] — greedy takes 4+3=7 ≠ 7 correctly by coincidence, but [3,3,3,4,1], sum=14, target=7: greedy 4+3=7 but leaves [3,3,1]=7 — both work here. Use [1,5,11,5] — target=11, greedy takes 11 first, rest is [1,5,5]=11 — works. Use [1,2,3,5] — target=5 — DP finds [2,3], greedy takes 5, leaves [1,2,3], target 0... actually valid. Construct a real failure case if pushed: [6,6,2,2,2,2], sum=20, odd — actually use [3,3,3,4,4,4] sum=21, odd. Point is: DP is required because subset problems have overlapping subproblems and optimal substructure at each item level, not across items greedily.

**3. "Curveball: How would you detect if an interval scheduling problem requires DP vs Greedy?"**
- Pointer: If the objective is "max number of non-overlapping intervals" OR "min intervals to remove" → Greedy (sort by end, greedy pick). If the objective is "max weight/value of non-overlapping intervals where intervals have weights" → DP (because greedy can't account for value trade-offs). This is a beautiful distinction. Weighted Job Scheduling (LC 1235) is the DP version; Non-overlapping Intervals (LC 435) is the Greedy version.

### Cloud/DevOps

**4. "You said you cut CI/CD time by 57%. Walk me through every specific change you made."**
- Pointer: (1) Identified bottleneck with GitLab pipeline trace — integration tests were blocking the image build. (2) Switched to DAG pipeline with `needs:` so unit tests, integration tests, and build ran in parallel. (3) Added Docker BuildKit with `--cache-from` pointing to registry cache — Maven build layers now hit 70%+ cache. (4) Mounted Maven cache directory with `RUN --mount=type=cache` to avoid downloading dependencies on each build. (5) Result: 23 min → 10 min. Do NOT just say "parallelism" — name the specific mechanisms.

**5. "Curveball: If a Kubernetes pod passes liveness but fails readiness forever, what happens?"**
- Pointer: The pod stays Running but is never added to the Service's Endpoints — it receives no traffic. It won't be restarted (liveness is fine). It just sits there, healthy but idle. This is actually the correct behaviour for a pod that has lost its upstream dependency (e.g., database down) — you don't want to restart it, you want to stop sending traffic to it. The fix is to address the root cause (restore DB connection), at which point readiness passes and traffic resumes automatically.

**6. "Why would you use Azure Key Vault over Kubernetes Secrets for storing credentials?"**
- Pointer: K8s Secrets are base64-encoded (not encrypted) by default — anyone with cluster access can read them. Key Vault provides: hardware-backed encryption (HSM tier), access policies tied to managed identity (no credentials to manage), audit logs of every secret access, automatic rotation capabilities, and centralised management across multiple clusters/environments. In a FedRAMP-aligned environment (Deep Fathom), Key Vault with private endpoints is non-negotiable.

**7. "Curveball: Your new pod starts, passes liveness, but never passes readiness. Traffic never reaches it. Debugging steps?"**
- Pointer: (1) `kubectl describe pod <name>` — look at Events for readiness probe failures and error messages. (2) `kubectl logs <pod>` — is the app throwing exceptions on startup? (3) Exec into pod: `kubectl exec -it <pod> -- curl localhost:8080/actuator/health/readiness` — what does it return? (4) Check if readiness depends on an external resource (DB, downstream service) — is that resource reachable from within the pod? (5) Check if `spring.datasource.url` or other config is misconfigured — app may start but fail health check because it can't connect to DB. This is a systematic debugging answer that demonstrates operational maturity.

**8. "Walk me through how your Bicep code actually provisions the Key Vault and wires it to Container Apps."**
- Pointer: This is a tell-me-the-details question. Be ready with: (1) `keyVault.bicep` module creates Key Vault, sets `enableSoftDelete: true`, adds role assignment `Key Vault Secrets User` to the Container App's managed identity using `roleDefinitionId`. (2) `containerApps.bicep` module references the secret: `secrets: [{ name: 'db-password', keyVaultUrl: kvRef.properties.vaultUri, identity: managedIdentityId }]`. (3) The environment var references the secret: `env: [{ name: 'DB_PASSWORD', secretRef: 'db-password' }]`. If you don't remember the exact Bicep syntax, describe the concept precisely — the intent is to verify you actually wrote it.

### Java/System Design

**9. "Curveball: You have a service that uses @Transactional. You add a @Cacheable on the same method. What happens if the cache returns a stale object after a rollback?"**
- Pointer: Classic AOP ordering problem. Both `@Transactional` and `@Cacheable` use proxies. If cache is populated before the transaction commits, you could serve a value that was later rolled back. Solution: ensure the cache population happens AFTER the transaction commits — use `@TransactionalEventListener(phase = AFTER_COMMIT)` to evict/repopulate the cache, or use `@CacheEvict` on the transactional write method so subsequent reads re-fetch. Also note: the proxy ordering matters (`@Order` on the advice, or `spring.cache.transaction.proxy-target-class`).

**10. "You built an LLM proxy that routes across 5 providers. How did you handle provider-specific rate limits without blocking the main request thread?"**
- Pointer: Async job queue pattern (Spring `@Async` + thread pool, or a proper queue like Redis Streams). Each LLM call submitted as a task; the HTTP thread returns 202 immediately. Rate limit state tracked per-provider in Redis with a sliding window (increment count, set TTL). The router checks available capacity before dispatching; if a provider is at its limit, it routes to the next cheapest available one. For the client: SSE or WebSocket push for real-time streaming; polling for batch jobs. This demonstrates you understand non-blocking architecture and distributed rate limiting.

**11. "Curveball: What's the difference between a Kubernetes Deployment and a StatefulSet? When would Spring Boot services use a StatefulSet?"**
- Pointer: Deployment pods are interchangeable — any pod can be killed and replaced, storage is ephemeral (or shared via PVC). StatefulSet pods have stable network identities (pod-0, pod-1) and stable persistent storage (one PVC per pod that doesn't float). Spring Boot services are almost always Deployments — they're designed to be stateless. StatefulSet is for stateful workloads: databases (PostgreSQL), message brokers (Kafka, Zookeeper), Elasticsearch. The one Spring Boot use case: if your app uses sticky sessions and can't be made truly stateless — but the right answer is to fix the stateful design, not to use a StatefulSet.

**12. "Tell me about a time the infrastructure failed in production and how you debugged it."**
- Pointer: Prepare a STAR story. Even if nothing dramatic has happened, think about: a pod OOMKilled (memory limit hit), a readiness probe that failed due to a DB migration taking too long on startup, a Bicep deployment that failed due to a name collision, a CI/CD pipeline that broke after a Docker Hub pull rate limit hit. Pick whichever is truest for you. The structure interviewers want: how did you detect it (alert? user report?), what was your first hypothesis, what tools did you use (kubectl logs, Azure Monitor, GitLab trace), what was the root cause, what did you change to prevent recurrence.

**13. "Curveball: Your Kubernetes rolling update is stuck — some old pods aren't terminating. What do you check?"**
- Pointer: (1) `kubectl rollout status deployment/app` — look at the stall message. (2) `kubectl get pods` — which pods are in Terminating state? How long? (3) Check if `terminationGracePeriodSeconds` is too low — app hasn't finished draining. (4) Check if a preStop hook is hanging. (5) Check if the pod has a finalizer that isn't being cleared (common with operators). (6) Check if the node is under pressure (DiskPressure, MemoryPressure) — eviction blocked. (7) Nuclear option: `kubectl delete pod <name> --force --grace-period=0` — but explain you'd only do this after ruling out data safety concerns.

**14. "Why Spring Boot and not Quarkus or Micronaut for your projects?"**
- Pointer: Don't be defensive — show you know the landscape. "Spring Boot has the broadest ecosystem maturity (Spring Cloud, Spring Security, Spring Data JPA), the largest community, and the most enterprise support — which mattered for a multi-service production deployment with Azure integration. Quarkus and Micronaut have faster startup and lower memory via native compilation (GraalVM), which matters for functions/serverless. If I were building a high-density Lambda-equivalent, I'd evaluate Quarkus seriously. For a long-running microservice with rich Spring Cloud features, Boot is still the pragmatic choice." This shows architectural awareness, not framework loyalty.

**15. "Curveball: How would you migrate this microservices architecture from Azure to AWS with minimal code changes?"**
- Pointer: Abstract the infrastructure, not the application. (1) The Spring Boot code itself is cloud-agnostic — no Azure SDK calls in business logic (you used managed identity + env var injection, so secrets are just env vars). (2) Replace Azure Container Apps with ECS Fargate or EKS. (3) Replace Key Vault with AWS Secrets Manager or Parameter Store — change the Bicep/CDK only. (4) Replace PostgreSQL Flexible Server with RDS PostgreSQL — connection string change only. (5) Replace Bicep with CDK or Terraform — rewrite infra code, zero app code changes. (6) The architectural discipline to keep cloud-specific concerns at the infra layer (not in application code) is exactly what makes this migration possible. This is the extraordinary candidate answer.

---

## 🌟 Extraordinary-Candidate Edge

**The 90-second differentiator pitch** (practise until it's effortless):

> "I'm unusual in this market because I sit at the intersection of deep backend engineering and infrastructure ownership — most developers in my experience bracket either write application code or manage infra, but rarely both at production depth. In Deep Fathom I wrote the Bicep IaC for 30+ Azure resources, designed the Kubernetes deployment with proper zero-downtime probes and graceful shutdown, cut CI/CD runtime by 57% through pipeline architecture changes, and built the LLM orchestration layer that handles 2–20 minute async jobs across 5 providers. In Smart360 I diagnosed and fixed production performance problems at the query and caching layer, reducing latency by 96%. I have hands-on scars from both sides of the system. That's what makes me different."

**Depth signals to drop in naturally:**

- Mentioning KEDA when discussing Azure Container Apps autoscaling (shows you looked under the hood).
- Mentioning that K8s readiness probe failure removes the pod from Service Endpoints but does NOT trigger a restart (shows precise operational understanding).
- Saying "we used Manacher's algorithm is O(n) vs expand-around-centre at O(n²) — in practice expand-around-centre is fine because of constant factor differences and cache behaviour" (shows you reason about complexity in context, not just asymptotically).
- Noting that `@Transactional` on `private` methods is silently ignored, and that self-invocation bypasses the proxy (shows you've debugged real Spring issues, not just read the docs).
- Explaining that BuildKit's `--mount=type=cache` avoids baking the Maven cache into the image layer (a detail that separates people who wrote Dockerfiles from people who optimised them).

**What extraordinary looks like in each interview format:**

- *Coding*: Clean solution + complexity analysis + space optimisation mentioned + "here's a follow-up variant you might ask" — shows you understand the problem family, not just this instance.
- *System design*: Draw the failure modes first, then design around them. Name anti-patterns you deliberately avoided (e.g., "I didn't use distributed transactions because 2PC couples services and creates availability risk").
- *Behavioural*: Specific numbers in every answer. No "we improved performance" — always "latency dropped from 60 s to 2–3 s, 96% improvement." Metrics are the signal that you owned the outcome.
- *Culture/fit*: Show intellectual curiosity — reference something you learned recently (LLM streaming protocols, Bicep what's new, K8s Gateway API replacing Ingress).

---

## 📊 End-of-Week Self-Assessment

Rate yourself 1–5 on each dimension after Sunday's mock. Be honest — this feeds directly into Week 9 planning.

| Area | Target | Your Rating | Gap / Next Action |
|---|---|---|---|
| Edit Distance — implement from scratch | 5 | | |
| Longest Palindromic Substring — expand approach | 5 | | |
| Maximal Square — recurrence articulation | 5 | | |
| Partition Equal Subset Sum — 0/1 knapsack reduction | 5 | | |
| Jump Game I & II — greedy explanation | 5 | | |
| Gas Station — proof of correctness | 4 | | |
| Merge/Insert/Non-overlapping Intervals | 5 | | |
| Deep Fathom architecture story (6 min) | 5 | | |
| Docker multi-stage build + BuildKit details | 5 | | |
| K8s rolling update + probe mechanics | 5 | | |
| Azure Key Vault managed-identity wiring | 4 | | |
| Bicep modular structure explanation | 4 | | |
| CI/CD 57% cut — specific mechanism explanation | 5 | | |
| Smart360 AWS S3 + pre-signed URL story | 5 | | |
| LinkedIn/Naukri updated + keyword-rich | 5 | | |
| 3–5 applications submitted + tracking sheet live | 5 | | |
| 90-second differentiator pitch — fluent | 5 | | |

**Scoring guide:**
- 5: Can do it cold, under pressure, on a whiteboard.
- 4: Solid but might hesitate on one edge case.
- 3: Know the concept, can't execute under pressure — needs more reps.
- 2: Shaky — must revisit this week's material.
- 1: Not comfortable — block 2 hours in Week 9 specifically for this.

**Week 8 exit criteria (minimum bar before Week 9):**
- All 9 LeetCode problems solved at least once without hints.
- At least 3 applications submitted and logged.
- LinkedIn headline and experience bullets updated with numbers.
- Able to do the 6-minute Deep Fathom story without notes.

**If you missed the bar:** Don't skip the missed items to start Week 9. Carry them as Day 1–2 makeup sessions. Momentum > perfection, but visible gaps compound.

---

*Week 8 complete → Week 9: Trees (BFS/DFS, LCA, Serialize/Deserialize) + System Design deep dives (Design a URL shortener, Design a notification system) + first GCC/product company applications.*
