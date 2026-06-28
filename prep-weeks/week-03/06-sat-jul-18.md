# Week 3 · Day 6 — Sat Jul 18 — Timed DSA (graphs/trees) + GitHub Polish + Consolidation

> Saturday is a **proving + packaging** day, not a new-material day. Block A is a single **timed set under real-interview conditions** — think aloud, handle edge cases, state complexity, no hints — so you find the gap between *can solve* and *can solve at speed while talking*. Block B is the part most engineers neglect and recruiters see first: a **recruiter-ready GitHub** — one polished showcase repo plus a profile that tells the cloud-native + LLM story in five lines. Block C is **consolidation**: compress the week's graph algorithms into one-sentence patterns, write the Kafka delivery-guarantee table from memory, and name your three weakest areas with a concrete drill for each.

📌 **Study today:** Timed DSA set (LC 207, 994 + 2 hardest re-solves) · GitHub polish + showcase repo · consolidation + weak-spot drills · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — Timed Set (graphs/trees, no hints)

**Rules of engagement — simulate the real thing.** Pick the problem, start a timer, narrate as you go (clarify → approach → complexity → code → test), handle edge cases out loud, and state final time/space complexity. **No editorials, no peeking at this week's notes** until the timer ends. The point isn't a green check — it's surfacing where you hesitate or misspeak under pressure.

### Problem A (25 min): Course Schedule (LC 207) — cold

Directed-graph cycle detection: "can all courses finish?" = "is the prereq graph a DAG?" Two correct attacks — drill **both**, lead with Kahn's:

```java
// Kahn's (BFS + in-degree). Cycle iff we process fewer than V nodes.
public boolean canFinish(int n, int[][] prereqs) {
    List<Integer>[] g = new List[n];
    int[] indeg = new int[n];
    for (int i = 0; i < n; i++) g[i] = new ArrayList<>();
    for (int[] p : prereqs) {           // p = [course, prereq]  ⇒ edge prereq → course
        g[p[1]].add(p[0]);
        indeg[p[0]]++;
    }
    Deque<Integer> q = new ArrayDeque<>();
    for (int i = 0; i < n; i++) if (indeg[i] == 0) q.offer(i);
    int processed = 0;
    while (!q.isEmpty()) {
        int u = q.poll();
        processed++;
        for (int v : g[u]) if (--indeg[v] == 0) q.offer(v);
    }
    return processed == n;              // leftover in-degree ⇒ cycle
}
```

The DFS 3-color variant (`0=unvisited, 1=in-current-path, 2=done`; revisiting a `1` node = back edge = cycle) is the alternative — be ready to write it if asked. **Target: AC in 25 min, both approaches discussed.**

### Problem B (25 min): Rotting Oranges (LC 994) — cold

**Multi-source BFS**: seed the queue with **all** rotten cells at once; each BFS level = one minute. After BFS, any remaining fresh orange ⇒ `-1`.

```java
public int orangesRotting(int[][] grid) {
    int m = grid.length, n = grid[0].length, fresh = 0, minutes = 0;
    Deque<int[]> q = new ArrayDeque<>();
    for (int r = 0; r < m; r++)
        for (int c = 0; c < n; c++) {
            if (grid[r][c] == 2) q.offer(new int[]{r, c}); // seed ALL rotten
            else if (grid[r][c] == 1) fresh++;
        }
    if (fresh == 0) return 0;                              // edge case: nothing to rot
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
    while (!q.isEmpty() && fresh > 0) {
        minutes++;
        for (int i = q.size(); i > 0; i--) {              // drain exactly one level
            int[] cell = q.poll();
            for (int[] d : dirs) {
                int nr = cell[0] + d[0], nc = cell[1] + d[1];
                if (nr < 0 || nc < 0 || nr >= m || nc >= n || grid[nr][nc] != 1) continue;
                grid[nr][nc] = 2;
                fresh--;
                q.offer(new int[]{nr, nc});
            }
        }
    }
    return fresh == 0 ? minutes : -1;                      // unreachable fresh ⇒ -1
}
```

Edge cases to *say out loud*: no fresh oranges → `0`; fresh orange walled off from all rot → `-1`; single cell. **Target: AC in 25 min.**

### Then: re-solve your 2 hardest tree/graph problems (15 min each)

Pick the two problems from this week where you hesitated most (likely candidates: Validate BST bounds (LC 98), Binary Tree Max Path Sum (LC 124), Clone Graph (LC 133), Course Schedule II (LC 210), or Path With Minimum Effort (LC 1631)). Re-solve cold, **15 min each**.

### Debrief (write it down)

- Where did you **hesitate** or say something wrong under pressure? (e.g. "blanked on Kahn's cycle condition," "forgot the `fresh==0` early return.")
- Write the **time + space complexity** for **every** problem you touched today — **from memory**. If you can't recall one instantly, that's a flagged weak spot for Block C.

---

## Block B — GitHub Polish + Showcase Repo

A recruiter or hiring manager lands on your GitHub **before** they read your resume in depth. Treat the profile as a landing page and **one** repo as the demo you'll screen-share.

### Pick ONE build (don't sprawl)

- **Option A — `smart360-event-demo`** (build from scratch): a minimal Spring Boot app that demonstrates the **Outbox pattern** you mastered Thursday:
  - `POST /users` → writes the user **and** an outbox event in **one transaction**.
  - `@Scheduled` publisher → reads `status='PENDING'` rows and logs `"would publish to Kafka: <event>"` (no real broker needed for the demo).
  - `GET /events` → returns the outbox table so the pattern is visible.
  - Tiny Vue/React page showing event status (ties in Friday's frontend block).
- **Option B — clean up an existing repo**: a real README, an architecture diagram (Mermaid or ASCII), tech-stack badges, `docker-compose up`, one screenshot; strip every `TODO` and compile warning.

Either way the artifact must demonstrate a pattern you can *talk about* (Outbox, idempotency, caching) — not a tutorial clone.

### README must include

- A one-paragraph **"What problem does this solve?"** written for a recruiter, not a compiler — the dual-write problem in plain English.
- An **architecture diagram** (Mermaid renders inline on GitHub):

  ```mermaid
  flowchart LR
    Client -->|POST /users| API
    API -->|"one tx: user + event"| DB[(Postgres)]
    DB --> Outbox[(outbox_events)]
    Scheduler -->|poll PENDING| Outbox
    Scheduler -->|would publish| Kafka[(Kafka)]
  ```

- **`docker-compose up`** that brings the app up in **< 5 commands**.
- Links from the README back to the relevant **resume bullets** (Smart360 monolith→event-driven, WebX async LLM jobs, Deep Fathom).

### Profile-polish checklist (recruiter lands here first)

- **Pin 3–6 best repos** — quality over quantity; these are the only repos most recruiters open.
- **Add a profile README** at `github.com/<user>/<user>` — 4–5 lines: who you are, the **cloud-native + LLM** narrative, current focus, contact/LinkedIn.
- **Prune/archive** embarrassing repos — old coursework, half-finished experiments, no-README dumps.
- **One showcase repo** aligned to the cloud-native + LLM narrative (Spring Boot + an LLM-proxy or IaC sample) with a clean README + diagram — the repo you screen-share in interviews.
- **Tidy commit messages** on the showcase repo: no `wip` / `fix` / `asdf` — imperative + scoped, e.g. `feat: add provider failover circuit breaker`.

> **Why this is worth a study block:** you can solve every LeetCode graph problem and still get filtered if your GitHub looks abandoned. A single clean, well-documented, runnable repo with a real architecture diagram signals "ships production software" louder than ten unfinished projects.

---

## Block C — Consolidation + Weak Spots

Compress the week into recall-ready summaries. If you can reproduce these from memory, the patterns have stuck.

### Graph algorithms — 3 bullets each (when · pattern · gotcha)

- **BFS** — *when:* shortest path on unweighted graphs, level order, multi-source. *pattern:* queue + visited, drain one level at a time. *gotcha:* mark visited **on enqueue**, not on dequeue (else duplicates flood the queue).
- **DFS** — *when:* flood fill, cycle detection, topo sort, connectivity. *pattern:* recursion or explicit stack + visited. *gotcha:* recursion overflows on huge/deep grids → go iterative.
- **Topological sort** — *when:* ordering with dependencies (Course Schedule). *pattern:* Kahn's (in-degree + BFS) or DFS post-order reversed. *gotcha:* a cycle means **no valid order** — Kahn's detects it when `processed < V`.
- **Union-Find** — *when:* connected components, undirected cycle, MST, dynamic connectivity. *pattern:* `parent[]` + path compression + union by rank, near-O(α(n)). *gotcha:* can't model **directed** cycles or do shortest path.
- **Dijkstra** — *when:* weighted shortest path, **non-negative** weights. *pattern:* min-heap of `(dist, node)`, relax, skip stale pops. *gotcha:* negative edges break the finalization invariant → Bellman-Ford.

### One-sentence pattern per graph problem

| Problem | Pattern |
|---|---|
| Number of Islands (200) | Flood fill (DFS/BFS), mark visited in place |
| Clone Graph (133) | DFS + `HashMap<original, clone>` to handle cycles |
| Course Schedule I/II (207/210) | Topological sort / directed cycle detection |
| Graph Valid Tree / Connected Components (261/323) | Union-find (connected + edge count, or count roots) |
| Rotting Oranges (994) | Multi-source BFS, levels = time |
| Network Delay / weighted paths (743) | Dijkstra (min-heap relaxation) |
| Redundant Connection (684) | Union-find — first edge that closes a cycle |

### Kafka delivery-guarantee table — write it from memory

| Guarantee | Producer config | Consumer design | Trade-off |
|---|---|---|---|
| At-most-once | `acks=0` | auto-commit **before** process | Fast, can lose messages |
| At-least-once | `acks=all` | commit **after** process | Safe, must handle duplicates |
| Exactly-once | idempotent producer + transactions | `isolation.level=read_committed` | Complex, **Kafka→Kafka only** |

The load-bearing footnote: exactly-once is Kafka→Kafka; the moment a side effect hits an external DB/S3/email you're back to **at-least-once + idempotent consumer** (Thursday's Block B).

### Name your top 3 weak areas + one drill each

Fill this in honestly from the Block A debrief and the self-assessment gate below — anything you couldn't recall instantly or solve in target time:

| # | Weak area | Concrete drill |
|---|---|---|
| 1 | _(e.g. Dijkstra under time pressure)_ | _Re-solve LC 743 + 1631 cold, ≤25 min each, Mon of Week 4_ |
| 2 | _(e.g. Saga choreography vs orchestration)_ | _Whiteboard a 5-service payment+audit saga, pick a style, 5 min_ |
| 3 | _(e.g. covering-index explanation)_ | _Explain covering vs partial index to a rubber duck in 2 sentences_ |

---

## 💻 Practice coding questions

1. Course Schedule (LC 207) cold, Kahn's — then write the DFS 3-color variant.
2. Rotting Oranges (LC 994) cold — multi-source BFS, name all three edge cases.
3. Re-solve your two hardest tree/graph problems of the week, 15 min each.
4. From memory, write time + space complexity for every problem you solved this week.
5. Scaffold `smart360-event-demo`: `POST /users` writing user + outbox event in one `@Transactional` method.
6. Write the `@Scheduled` outbox publisher that reads PENDING rows and logs "would publish."
7. Write the repo's Mermaid architecture diagram by hand.
8. Reproduce the Kafka delivery-guarantee table from memory.
9. Write the one-sentence pattern for each of the seven graph problems above.
10. Draft your profile README (4–5 lines: identity, cloud-native + LLM narrative, focus, contact).

---

## 🎤 Interview questions

1. **Why does Kahn's algorithm detect a cycle?** Only zero-in-degree nodes ever enter the queue; a cycle's nodes never reach in-degree 0, so they're never processed → `processed < V` means a cycle exists.
2. **Course Schedule — Kahn's vs DFS 3-color, when each?** Both are `O(V+E)`. Kahn's (BFS + in-degree) also yields the order directly (LC 210) and avoids deep recursion; DFS 3-color is terser and natural if you're already doing post-order work.
3. **Why seed the queue with all rotten oranges in LC 994?** It's multi-source BFS — all sources spread simultaneously, so BFS levels map exactly to elapsed minutes; a single-source BFS would miscount time.
4. **How do you know when a graph problem is union-find vs BFS/DFS?** Union-find for pure connectivity / undirected cycle / online edge additions; BFS/DFS when you need the actual path, distance, or to process level by level.
5. **At-most-once vs at-least-once vs exactly-once — the configs and the catch.** `acks=0` + commit-before-process = at-most-once (can lose); `acks=all` + commit-after = at-least-once (can duplicate); idempotent producer + transactions + `read_committed` = exactly-once, **but only Kafka→Kafka**.
6. **Why is "exactly-once to our database" the wrong claim?** The DB write and the Kafka offset commit are separate systems with no shared transaction; you can't commit both atomically. Real systems do at-least-once + an idempotent consumer.
7. **A recruiter opens your GitHub — what makes them keep reading?** A profile README with a clear narrative, 3–6 pinned quality repos, and one showcase repo with a real README, architecture diagram, and `docker-compose up` — signals "ships software," not "did coursework."
8. **Walk the full async LLM job design in one breath.** Client → API → DB (jobs + outbox in one tx) → Debezium/scheduler → Kafka → worker pool → LLM call → DB write → SSE/poll back to client; at-least-once throughout, so the worker is idempotent on `job_id`.
9. **What's the single most common Outbox bug to call out proactively?** The poller publishes to Kafka then crashes before marking `SENT` → the row republishes → a duplicate. There's no atomicity across Postgres and Kafka; that's the at-least-once boundary, which is why consumers are idempotent from day one.
10. **(Curveball) "Number of Islands on a 100k × 100k grid — does your DFS still work?"** Recursive DFS overflows the stack; switch to iterative BFS/DFS, and for a truly distributed grid, partition it and union-find across the partition boundaries.
11. **Diameter of a binary tree — does the longest path pass through the root?** Not necessarily — it's `max over all nodes of (left height + right height)`, tracked during a height DFS; the root is just one candidate.
12. **(Reflection) Look at your showcase repo as a stranger — would you star it?** If the README doesn't explain the problem in one paragraph, lacks a diagram, or won't run in five commands, the answer is no — fix those three before Week 4.

---

## ✅ Self-check

1. Solved LC 207 and LC 994 cold within target time, narrating approach + complexity?
2. Re-solved your two hardest tree/graph problems and wrote complexity for every problem from memory?
3. Showcase repo pushed with a real README, architecture diagram, and `docker-compose up` — would a stranger click "Star"?
4. Reproduced the Kafka delivery-guarantee table and the seven one-sentence graph patterns from memory?
5. Can you walk the full async LLM job system design (Client → API → DB jobs+outbox → Debezium/scheduler → Kafka → worker pool → LLM → DB → SSE) in 10 minutes without notes?
6. Listed your top 3 weak areas with one concrete drill each, scheduled into Week 4's review slot?

---

*Nav: ← [Day 5 (Fri Jul 17)](05-fri-jul-17.md) · [Week 3](README.md) · [Week 4](../week-04/) →*
