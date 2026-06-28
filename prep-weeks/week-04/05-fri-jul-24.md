# Week 4 · Day 5 — Fri Jul 24 — Heaps / Design Twitter + CAP / PACELC + Consistent Hashing + Cloud/DevOps

> Today the heap goes from a pure algorithm to a *system design primitive*: **Design Twitter (LC 355)** is a merge-K over followees' timelines — the same heap pattern as yesterday's LC 23, now framed as an LLD round, plus two heap+greedy mixers (Task Scheduler, Top K Frequent Words). Block B is the consistency half of the at-scale backbone: **CAP stated precisely** (it's about behaviour *during a partition*, nothing else), **PACELC** (the latency-vs-consistency trade-off even when there's no partition), **consistent hashing** revisited as a consistency/availability lever, and the **four consistency models** — with PostgreSQL, Redis, Cassandra, DynamoDB, and Spanner placed correctly. Block C is your **cloud/DevOps talking points**: Docker multi-stage, K8s rolling-update + probe mechanics, the GitLab DAG/BuildKit pipeline behind the 57% CI/CD cut, and a word on Bicep IaC.

📌 **Study today:** Heaps/mixed + Design Twitter as LLD (LC 355, 621, 692) · CAP / PACELC + consistent hashing + consistency models · cloud/DevOps (Docker multi-stage, K8s probes, CI/CD, Bicep) · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Heaps / Mixed + Design Twitter (LLD)

### Design Twitter (LC 355) — heap merge as a low-level design

This is the bridge problem: a heap algorithm dressed as an **LLD** round. You maintain per-user state and answer `getNewsFeed` by merging the most recent tweets of everyone the user follows — capped at 10. That cap is exactly merge-K-sorted-lists (LC 23) with an early stop.

**Data model:**
- A global monotonic `timestamp` (so we can order tweets across users without clocks).
- `Map<userId, List<int[]>>` — each user's tweets as `[timestamp, tweetId]`, appended in order.
- `Map<userId, Set<Integer>>` — each user's follow set (always include themselves).

```java
class Twitter {
    private int time = 0;
    private final Map<Integer, List<int[]>> tweets = new HashMap<>(); // user → [time, tweetId]
    private final Map<Integer, Set<Integer>> follows = new HashMap<>();

    public void postTweet(int userId, int tweetId) {
        tweets.computeIfAbsent(userId, k -> new ArrayList<>()).add(new int[]{time++, tweetId});
    }

    public List<Integer> getNewsFeed(int userId) {
        // max-heap by timestamp: newest first
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> b[0] - a[0]);
        Set<Integer> followees = new HashSet<>(follows.getOrDefault(userId, Set.of()));
        followees.add(userId);                                  // you see your own tweets
        for (int f : followees) {
            List<int[]> ts = tweets.get(f);
            if (ts != null && !ts.isEmpty())
                pq.offer(ts.get(ts.size() - 1));               // seed with each followee's latest
        }
        // NOTE: to advance within a list, store (time, tweetId, ownerId, idx) and re-push prev.
        List<Integer> feed = new ArrayList<>();
        while (!pq.isEmpty() && feed.size() < 10) {
            feed.add(pq.poll()[1]);
        }
        return feed;
    }

    public void follow(int a, int b)   { follows.computeIfAbsent(a, k -> new HashSet<>()).add(b); }
    public void unfollow(int a, int b) { follows.getOrDefault(a, new HashSet<>()).remove(b); }
}
```

The production-grade version seeds the heap with `(time, tweetId, ownerId, index-of-that-tweet)` and, on each pop, pushes the *previous* tweet of that owner — true merge-K that walks back at most 10 entries per list. **The LLD framing point that wins the round:** "`getNewsFeed` is fan-out-on-read — merge-K at query time. At Twitter scale you'd switch to fan-out-on-write (push each tweet into followers' precomputed timelines) for normal users, and keep fan-out-on-read only for celebrities with millions of followers (the **hybrid timeline** to avoid write amplification)." That sentence connects the LeetCode problem to the real system. **Target AC: 25 min.**

### Task Scheduler (LC 621) — greedy + heap

Identical tasks need `n` cooldown slots between them; minimise total time. The schedule is paced by the **most frequent task** — it forces the idle gaps. Two solutions, know both:

**Math (O(1) after counting):**
```java
public int leastInterval(char[] tasks, int n) {
    int[] freq = new int[26];
    for (char t : tasks) freq[t - 'A']++;
    int maxFreq = Arrays.stream(freq).max().orElse(0);
    int countMax = (int) Arrays.stream(freq).filter(f -> f == maxFreq).count();
    int frame = (maxFreq - 1) * (n + 1) + countMax;     // skeleton built around the rarest gaps
    return Math.max(tasks.length, frame);               // can't be fewer than the total tasks
}
```
The `(maxFreq-1)` full frames each of width `(n+1)`, plus the final partial row of `countMax` max-frequency tasks; but if there are enough *other* tasks to fill every idle slot, the answer is just `tasks.length`. **The max-heap + cooldown-queue simulation** is the alternative to narrate when asked "what if tasks have priorities?" **Target AC: 20 min.**

### Top K Frequent Words (LC 692) — the tie-breaking comparator

The whole exercise is a comparator that mixes two directions: frequency ascending (so the min-heap evicts the *least* frequent) but, on a tie, word lexicographically *descending* (so the "worst" word — later alphabetically — is on top to be evicted).

```java
public List<String> topKFrequent(String[] words, int k) {
    Map<String, Integer> freq = new HashMap<>();
    for (String w : words) freq.merge(w, 1, Integer::sum);
    PriorityQueue<String> pq = new PriorityQueue<>((a, b) ->
        freq.get(a).equals(freq.get(b))
            ? b.compareTo(a)              // tie: keep lexicographically smaller → evict larger
            : freq.get(a) - freq.get(b)); // else: evict least frequent
    for (String w : freq.keySet()) {
        pq.offer(w);
        if (pq.size() > k) pq.poll();
    }
    LinkedList<String> out = new LinkedList<>();
    while (!pq.isEmpty()) out.addFirst(pq.poll()); // reverse: heap pops worst-first
    return out;
}
```

**Talking point:** "The min-heap's comparator is the *inverse* of the desired output order, because the root must be the element I'm most willing to discard. The final reversal restores the intended order." Getting the tie-break direction right under pressure is the signal here. **Target AC: 20 min.**

### Practice set

| # | Problem (LC) | Approach | Key insight | Complexity |
|---|---|---|---|---|
| 1 | Design Twitter (355) | merge-K over followee timelines, cap 10 | fan-out-on-read; mention hybrid at scale | O(F + 10 log F) |
| 2 | Task Scheduler (621) | greedy math OR max-heap + cooldown queue | most frequent task sets the idle frame | O(n) |
| 3 | Top K Frequent Words (692) | size-k min-heap, inverted tie-break comparator | comparator is the inverse of output order | O(n log k) |
| 4 | K Closest Points to Origin (973) | max-heap of size k by squared distance | no sqrt needed; size-k bounds it | O(n log k) |
| 5 | (Stretch) Reorganize String (767) | max-heap by remaining count, place + cooldown | same cooldown idea as 621 | O(n log Σ) |
| 6 | (Stretch) Sliding Window Median (480) | two heaps + lazy deletion | delete-by-marking, clean lazily on top | O(n log n) |

---

## Block B — CAP / PACELC + Consistency Models + Consistent Hashing

This is the consistency half of the backbone. The trap interviewers set is the sloppy "we're a CP system" — they want to hear that you know CAP is a statement about *partition behaviour only*.

### CAP — the precise statement

A distributed system can satisfy at most two of **C**onsistency, **A**vailability, **P**artition-tolerance. But networks *will* partition, so P isn't optional — the real choice is **C vs A, and only while a partition is happening**:

- **CP** — during a partition, refuse requests that can't guarantee the latest value (sacrifice availability to stay correct). The isolated side stops accepting writes / serving reads rather than risk staleness.
- **AP** — during a partition, keep answering (sacrifice consistency) and reconcile later. Every node responds, possibly with stale data.

> **The whole point: when there is no partition you get both C and A.** CAP only governs the *partition* moment. The sharp version of the question is always "**how does your system behave *during* a partition?**"

| System | CAP (during partition) | Why |
|---|---|---|
| PostgreSQL (single-node / sync replica) | **CP** | an isolated replica stops serving rather than return stale data |
| Redis (default async replication) | **AP-leaning** | replicas keep serving possibly-stale reads; no strong cross-node guarantee |
| Cassandra | **AP by default, tunable to C** | per-query `QUORUM`/`ALL` trades availability for consistency |
| DynamoDB | **AP** (eventually consistent reads; opt-in strong reads) | stays available; strong reads cost latency/availability |
| Spanner / CockroachDB | **CP** | TrueTime + consensus chooses consistency, gives up availability on the minority side |

### PACELC — the GCC-favourite extension

CAP only describes the partition case. **PACELC**: **if Partition → choose A or C; Else (normal operation) → choose L(atency) or C(onsistency).** Even with a healthy network, a strongly-consistent read needs cross-node coordination → latency; a low-latency read accepts possible staleness.

| System | PACELC | Reading |
|---|---|---|
| DynamoDB | **PA/EL** | available under partition; low-latency (eventual) normally |
| Cassandra | **PA/EL** | same posture, tunable per query |
| Spanner / CockroachDB | **PC/EC** | consistent under partition; consistent-but-higher-latency normally (the price of TrueTime) |
| PostgreSQL (sync) | **PC/EC** | consistent always, paying coordination latency |

**Why mention PACELC:** "You can't just say 'we're CP.' Even with no partition there's a latency budget for consistency — Spanner pays milliseconds for global consistency; DynamoDB gives single-digit-ms eventual reads. The design question is usually *what's the latency budget for consistency in normal operation?*" That reframing marks you as someone who thought past the textbook.

### Consistency models — the spectrum

| Model | Guarantee | Cost | Example |
|---|---|---|---|
| **Strong (linearizable)** | every read sees the most recent write, globally | coordination on every op (slow) | PostgreSQL `synchronous_commit=on`, Spanner |
| **Read-your-writes** | you always see your *own* writes | route your reads to a fresh-enough node | sufficient for most user-facing features |
| **Causal** | reads after a causally-related write reflect it | track causal dependencies (vector clocks) | MongoDB causal-consistency sessions |
| **Eventual** | replicas converge if writes stop | none — cheapest, weakest | DNS, Redis replication, the S3-URL cache |

The art is choosing the *weakest* model that's still correct for the feature: a bank balance needs strong; a "your profile photo" read needs only read-your-writes; a global trending list is fine eventual.

### Consistent hashing — the consistency/availability lever (deep cut)

Yesterday's ring revisited from the consistency angle: adding a node relocates only the keys on the arc between it and its predecessor — **O(n/k)**, no global rehash — which is *why* Cassandra and Redis Cluster can rebalance **incrementally and stay available** while scaling. Replication overlays the ring: each key is stored on the next **R** nodes clockwise (the replica set). Tunable consistency falls out of this — `W + R > N` (write/read quorums overlapping) gives strong-ish reads; smaller quorums favour availability/latency. That single mechanism — ring + replica set + quorum knobs — is the foundation of Dynamo-style stores.

> **Project tie-in (Smart360):** "PostgreSQL was the **CP** source of truth; Redis was the **AP-leaning** cache. Pre-signed-URL TTLs were tuned to sit safely *inside* the eventual-consistency window so a cached URL was always still valid. For two users editing the same dataset I'd lead with AP + optimistic locking / CRDT merge over a CP distributed lock — availability under partition mattered more than serialised edits."

### Gotchas

- **Saying "we're CP" with no partition qualifier** — CAP is *only* about partition behaviour; without the qualifier you sound like you memorised an acronym.
- **Conflating "consistent" with "available"** — they only trade off *during* a partition; normally you have both.
- **Forgetting PACELC's E branch** — the latency-vs-consistency trade-off exists even on a healthy network.
- **Over-choosing strong consistency** — pick the weakest model that's still correct; strong consistency taxes latency on every read.

---

## Block C — Cloud / DevOps Talking Points

These are *talking points*, not algorithms — but interviewers for senior backend roles probe them hard. Name the **specific mechanisms**, never just "we used Docker / Kubernetes."

### Docker multi-stage build

```dockerfile
# ---- Stage 1: build (heavy: JDK + Maven + source) ----
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
RUN --mount=type=cache,target=/root/.m2 mvn -q dependency:go-offline  # cache deps, don't bake them
COPY src ./src
RUN --mount=type=cache,target=/root/.m2 mvn -q clean package -DskipTests

# ---- Stage 2: runtime (lean: JRE + jar only) ----
FROM eclipse-temurin:21-jre AS runtime
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

- **Why:** the runtime image carries *only* a JRE + the jar — no JDK, no Maven, no source, no `.m2` cache. Result: **~600 MB → ~180 MB**, a smaller attack surface, and faster pulls/cold-starts.
- **BuildKit specifics:** `DOCKER_BUILDKIT=1`; `RUN --mount=type=cache,target=/root/.m2` keeps the Maven cache *out* of the image layer (don't bake it in — it bloats the layer and busts caching); `--cache-from type=registry,ref=...` reuses layers across CI runners.

### Kubernetes zero-downtime rolling update + probes

```text
kubectl set image deploy/api api=...:v2
  → new pod created (within maxSurge)
  → startup probe passes (slow-start grace) → liveness begins
  → readiness probe passes → pod ADDED to Service endpoints (now gets traffic)
  → old pod removed from endpoints → SIGTERM
  → Spring graceful drain (finish in-flight requests) → terminate
  (terminationGracePeriodSeconds must EXCEED the Spring shutdown timeout)
```

The three probes — never confuse them:

| Probe | Question | On failure |
|---|---|---|
| **Liveness** | is the process alive / not deadlocked? | **restart** the container |
| **Readiness** | can it serve traffic right now? | **remove** from Service endpoints (no restart) |
| **Startup** | has a slow-starting app finished booting? | hold off liveness/readiness until it passes |

Spring config that makes the drain real: `server.shutdown=graceful`, `spring.lifecycle.timeout-per-shutdown-phase=30s`, and pod `terminationGracePeriodSeconds: 40` (**must exceed** the Spring timeout, or K8s SIGKILLs mid-drain). `maxSurge`/`maxUnavailable` tune how aggressively the rollout swaps pods.

### GitLab DAG / BuildKit pipeline — the 57% CI/CD cut

```text
BEFORE: stage build → stage test → stage integration → stage deploy   (sequential, ~23 min)
AFTER:  jobs declare `needs:` → unit tests, integration tests, image build run in PARALLEL
        + BuildKit registry layer cache (70%+ layer hits)              (~10 min, −57%)
```

Name the two specific mechanisms: **(1) the DAG** — `needs:` lets a job start the instant its dependencies finish instead of waiting for the whole previous stage; **(2) BuildKit registry cache** — `--cache-from` pulls unchanged layers so most of the image rebuild is cache hits. "Parallelism" alone is the weak answer; the `needs:` DAG + layer-cache hit-rate is the strong one.

### Bicep / IaC (brief)

**Bicep** is Azure's declarative IaC DSL that transpiles to ARM JSON — far more readable, with modules, parameters, and what-if previews. Talking point: "We codified the environment (app service / container app, Redis, Postgres, networking) in Bicep modules so environments were reproducible and reviewed in PRs — `az deployment ... what-if` gives a plan/diff before apply, same idea as Terraform plan." Mention idempotent declarative deploys and drift detection.

---

## 💻 Practice coding questions

1. Design Twitter (LC 355) — merge-K over followees, cap 10; then describe fan-out-on-write vs hybrid.
2. Task Scheduler (LC 621) — the O(1) math formula *and* the max-heap + cooldown-queue simulation.
3. Top K Frequent Words (LC 692) — the inverted tie-break comparator (freq asc, word desc).
4. K Closest Points to Origin (LC 973) — size-k max-heap by squared distance.
5. (Stretch) Reorganize String (LC 767) — max-heap by remaining count with a one-slot cooldown.
6. (Stretch) Sliding Window Median (LC 480) — two heaps with lazy deletion.
7. Write the Docker multi-stage Dockerfile for a Spring Boot jar from memory (build → runtime).
8. Write the K8s readiness/liveness/startup probe stanza + Spring graceful-shutdown config.
9. Sketch the GitLab `needs:` DAG that parallelises unit, integration, and build jobs.
10. From memory, fill the CAP and PACELC tables placing PostgreSQL, Redis, Cassandra, DynamoDB, Spanner.

---

## 🎤 Interview questions

1. **State CAP precisely.** During a network partition a system must choose Consistency (refuse/stale-block) or Availability (answer, maybe stale); with no partition you get both. The only meaningful question is behaviour *during* a partition.
2. **Place PostgreSQL, Redis, Cassandra, DynamoDB, Spanner on CAP.** PostgreSQL CP (isolated replica stops serving), Redis AP-leaning (serves possibly-stale), Cassandra AP tunable to C via quorum, DynamoDB AP (opt-in strong reads), Spanner CP (consensus + TrueTime).
3. **What does PACELC add over CAP?** The Else branch: even with no partition there's a Latency-vs-Consistency trade-off. DynamoDB PA/EL (low-latency eventual), Spanner PC/EC (pays latency for global consistency).
4. **(Curveball) "CAP is outdated, just use PACELC."** PACELC *refines* CAP, doesn't replace it — it adds the normal-operation latency-vs-consistency dimension. The practical question is the latency budget for consistency when the network is healthy.
5. **Name the four consistency models, weakest correct one wins.** Strong (every read = latest write), read-your-writes (see your own), causal (causally-related reads reflect prior writes), eventual (converge if writes stop). Pick the weakest that's still correct: balance → strong, profile photo → RYW, trending list → eventual.
6. **Why does consistent hashing let Cassandra/Redis Cluster rebalance without downtime?** Adding a node moves only the arc between it and its predecessor (O(n/k)) — no global rehash — so rebalancing is incremental and the cluster stays available.
7. **How does quorum (`W+R>N`) give tunable consistency on the ring?** Each key lives on the next R nodes; if write and read quorums overlap (`W+R>N`) a read always sees the latest acknowledged write. Smaller quorums trade consistency for availability/latency.
8. **Walk through a C-vs-A decision from your own work.** Smart360 chose AP for the S3-URL cache (Redis-down → regenerate from S3; TTL inside the URL expiry). Strong consistency (invalidate-on-every-change) would have negated the ~80% S3-call reduction.
9. **Design Twitter `getNewsFeed` — what's the heap doing?** Merge-K-sorted over followees' timelines, capped at 10 — pop the newest, advance that user's list. At scale switch to fan-out-on-write for normal users, keep fan-out-on-read for celebrities (hybrid) to bound write amplification.
10. **Task Scheduler — what sets the minimum time?** The most frequent task: `(maxFreq-1)*(n+1) + countOfMax`, unless there are enough other tasks to fill the idle slots, in which case it's just the task count.
11. **Top K Frequent Words — why is the comparator inverted?** The min-heap root must be the element you're most willing to evict, so the comparator is the *inverse* of the desired output order; you reverse the popped sequence at the end.
12. **Why a Docker multi-stage build?** The runtime stage copies only the jar onto a JRE base — no JDK/Maven/source — cutting ~600 MB → ~180 MB and shrinking the attack surface; BuildKit caches the Maven `.m2` via a cache mount instead of baking it into a layer.
13. **Liveness vs readiness vs startup probe?** Liveness failure restarts the container; readiness failure removes it from Service endpoints (no restart); startup probe holds off the other two until a slow app finishes booting.
14. **(Curveball) A pod passes liveness but fails readiness forever — what happens?** It stays Running, gets no traffic (removed from endpoints), and is *not* restarted — correct for a lost upstream (DB down). Fix the dependency; readiness recovers and traffic resumes automatically.
15. **How did you get a 57% CI/CD reduction?** GitLab `needs:` DAG ran unit tests, integration tests, and image build in parallel instead of sequential stages, plus a BuildKit registry layer cache (70%+ hits) — ~23 min → ~10 min. The mechanism is the DAG + cache hit-rate, not just "parallelism."
16. **Why must `terminationGracePeriodSeconds` exceed the Spring shutdown timeout?** SIGTERM starts the graceful drain; if K8s' grace period elapses first it SIGKILLs mid-drain, dropping in-flight requests. The pod grace must be longer than `spring.lifecycle.timeout-per-shutdown-phase`.
17. **What is Bicep and why use it over raw ARM?** A declarative Azure IaC DSL transpiling to ARM JSON — readable, modular, with `what-if` plan/diff previews and idempotent declarative applies; same plan-before-apply discipline as Terraform.
18. **(Curveball) Redis (your rate limiter / cache) goes down — what happens?** Cache: cache-miss path regenerates from the source of truth (Postgres / S3). Rate limiter: Spring Cloud Gateway defaults to fail-open (allow, lose limiting); choose fail-closed only for billing/abuse-sensitive endpoints. Name the trade-off explicitly.

---

## ✅ Self-check

1. State CAP in one precise sentence (partition behaviour only) and place five systems on it from memory.
2. Explain PACELC's two branches and place DynamoDB and Spanner, justifying each.
3. List the four consistency models and pick the weakest-correct one for: bank balance, profile photo, trending list.
4. Describe a full zero-downtime K8s rolling deploy including the Spring graceful-shutdown config, in 2 minutes.
5. Name the two specific mechanisms behind the 57% CI/CD cut (DAG `needs:` + BuildKit layer cache).

---

*Nav: ← [Day 4 (Thu Jul 23)](04-thu-jul-23.md) · [Week 4](README.md) · [Day 6 (Sat Jul 25)](06-sat-jul-25.md) →*
