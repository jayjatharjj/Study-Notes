# Week 3 · Day 5 — Fri Jul 17 — Dijkstra / Weighted Graphs + Frontend (Vue/React) + Typeahead HLD

> Plain BFS finds the fewest *edges*; the moment edges carry **weights** you need **Dijkstra** — a greedy min-heap walk whose whole correctness rests on one invariant: *the node you pop is done*. Block B is the full-stack story interviewers probe to see if a "backend engineer" can actually reason about the client — **Vue 3's Proxy reactivity** vs **React's reconciliation/Fiber**, said like someone who shipped both. Block C turns yesterday's Trie into a real **typeahead** design: sharded index, precomputed top-k, an offline log→trie pipeline, and a hot-prefix cache — the bridge from a LeetCode data structure to a system that answers in under 100 ms p99.

📌 **Study today:** Dijkstra / weighted graphs (LC 743, 787, 1631) · Vue 3 reactivity + React hooks/reconciliation · typeahead / autocomplete HLD · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Dijkstra / Weighted Graphs

### Theory

BFS gives the shortest path **only on unweighted graphs**, because it expands strictly by hop count — every edge "costs 1." Add weights and the fewest-hop path may not be the cheapest. **Dijkstra** fixes this with a greedy strategy driven by a min-heap of `(dist, node)`:

> **Relaxation invariant:** the instant a node is *popped* from the min-heap, its recorded distance is **final** (the true shortest distance from the source). Any alternative route to it must pass through some node still in the heap whose tentative distance is **≥** the popped node's — so it can't be shorter.

That invariant is the entire proof, and it has a hard precondition: **non-negative edge weights**. A negative edge could make a "finalized" node reachable more cheaply later, breaking the guarantee — for negatives you need **Bellman-Ford** (or SPFA), and for all-pairs, Floyd-Warshall.

**The mechanics — "relax":** to *relax* an edge `(u → v, w)` is to check whether going through `u` improves `v`: `if dist[u] + w < dist[v] → dist[v] = dist[u] + w` and push `(dist[v], v)`. We use a **lazy** heap: we never decrease-key in place; we push the improved entry and **skip stale pops** (`if d > dist[node] continue`). This is simpler than a fancy indexed heap and just as correct.

**Complexity:** `O(E log V)` with a binary heap (each edge can push once; each pop is `log` of heap size). Space `O(V + E)`.

**When to reach for what:**

| Situation | Algorithm |
|---|---|
| Unweighted shortest path | BFS — `O(V+E)` |
| Weights, all non-negative | Dijkstra — `O(E log V)` |
| Negative edges (no negative cycle) | Bellman-Ford — `O(V·E)` |
| All-pairs shortest path | Floyd-Warshall — `O(V³)` |
| 0/1 edge weights | 0-1 BFS (deque) — `O(V+E)` |
| Shortest path + a *constraint* (≤ k stops) | Bellman-Ford rounds / state-augmented BFS |

### Worked template — write this cold

```java
// Dijkstra from `src` over an adjacency list: graph.get(u) = list of [v, weight]
int[] dijkstra(int n, List<int[]>[] graph, int src) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;

    // min-heap ordered by distance: {dist, node}
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
    pq.offer(new int[]{0, src});

    while (!pq.isEmpty()) {
        int[] top = pq.poll();
        int d = top[0], u = top[1];
        if (d > dist[u]) continue;            // stale entry — already finalized cheaper
        for (int[] edge : graph[u]) {
            int v = edge[0], w = edge[1];
            if (dist[u] + w < dist[v]) {      // relax
                dist[v] = dist[u] + w;
                pq.offer(new int[]{dist[v], v});
            }
        }
    }
    return dist;
}
```

The `if (d > dist[u]) continue` line *is* the lazy-deletion trick — say it out loud in the interview: "I don't decrease-key; I push duplicates and skip stale pops."

### Worked example — Network Delay Time (LC 743)

Send a signal from node `k`; return the time for **all** nodes to receive it = the **maximum** of all finalized shortest distances (or `-1` if any node is unreachable). Canonical single-source Dijkstra.

```java
public int networkDelayTime(int[][] times, int n, int k) {
    List<int[]>[] graph = new List[n + 1];          // 1-indexed
    for (int i = 1; i <= n; i++) graph[i] = new ArrayList<>();
    for (int[] t : times) graph[t[0]].add(new int[]{t[1], t[2]}); // u -> [v, w]

    int[] dist = new int[n + 1];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[k] = 0;
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
    pq.offer(new int[]{0, k});

    while (!pq.isEmpty()) {
        int[] top = pq.poll();
        int d = top[0], u = top[1];
        if (d > dist[u]) continue;
        for (int[] e : graph[u]) {
            if (d + e[1] < dist[e[0]]) {
                dist[e[0]] = d + e[1];
                pq.offer(new int[]{dist[e[0]], e[0]});
            }
        }
    }
    int ans = 0;
    for (int i = 1; i <= n; i++) {
        if (dist[i] == Integer.MAX_VALUE) return -1; // unreachable node
        ans = Math.max(ans, dist[i]);
    }
    return ans;
}
```

**Target AC: 25 min.**

### Practice set

| # | Problem (LC) | Approach | Key insight | Complexity |
|---|---|---|---|---|
| 1 | Network Delay Time (743) | Dijkstra, answer = max dist | All-receive ⇒ max of shortest dists | O(E log V) |
| 2 | Cheapest Flights Within K Stops (787) | **Bellman-Ford k+1 rounds** (or BFS on `(node, stops)`) | Plain Dijkstra is *wrong* — cost can beat stop budget | O(K·E) |
| 3 | Path With Minimum Effort (1631, Hard) | Dijkstra on grid, cost = MAX diff (bottleneck) | Relax with `max(eff, |Δh|)`, not sum | O(mn log(mn)) |
| 4 | Swim in Rising Water (778, Hard) | Same bottleneck-Dijkstra (min of max elevation) | Minimax path = Dijkstra with `max` relax | O(mn log(mn)) |
| 5 | (Stretch) Path with Maximum Probability (1514) | Dijkstra with `max` heap, multiply probs | Maximize product ⇒ max-heap, `>` relax | O(E log V) |
| 6 | (Stretch) Minimum Cost to Reach Destination in Time (1928) | DP / Dijkstra over `(node, time)` state | Constraint dimension → state augmentation | O(...) |

**Cheapest Flights Within K Stops (LC 787)** — the trap is that the **cheapest** path may use **too many stops**, so finalizing a node by price alone is wrong. Do `k+1` rounds of Bellman-Ford over a snapshot of distances so each round adds at most one more edge:

```java
public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;
    for (int i = 0; i <= k; i++) {            // at most k stops ⇒ k+1 edges
        int[] snapshot = dist.clone();         // relax from the PREVIOUS round only
        for (int[] f : flights) {
            int u = f[0], v = f[1], w = f[2];
            if (snapshot[u] != Integer.MAX_VALUE && snapshot[u] + w < dist[v])
                dist[v] = snapshot[u] + w;
        }
    }
    return dist[dst] == Integer.MAX_VALUE ? -1 : dist[dst];
}
```

The `snapshot` is essential: relaxing against the live `dist` in the same round would let a path use more than one new edge per round, blowing the stop limit. **Target AC: 25 min.**

**Path With Minimum Effort (LC 1631)** — a *bottleneck* path: the cost of a route is the **maximum** absolute height-difference along it, and we want the route minimizing that maximum. Same Dijkstra skeleton, but relax with `max` instead of `+`:

```java
public int minimumEffortPath(int[][] h) {
    int m = h.length, n = h[0].length;
    int[][] effort = new int[m][n];
    for (int[] row : effort) Arrays.fill(row, Integer.MAX_VALUE);
    effort[0][0] = 0;
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]); // {effort, r, c}
    pq.offer(new int[]{0, 0, 0});
    while (!pq.isEmpty()) {
        int[] t = pq.poll();
        int e = t[0], r = t[1], c = t[2];
        if (r == m - 1 && c == n - 1) return e;   // first pop of target = answer
        if (e > effort[r][c]) continue;
        for (int[] d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr < 0 || nc < 0 || nr >= m || nc >= n) continue;
            int ne = Math.max(e, Math.abs(h[nr][nc] - h[r][c])); // bottleneck relax
            if (ne < effort[nr][nc]) {
                effort[nr][nc] = ne;
                pq.offer(new int[]{ne, nr, nc});
            }
        }
    }
    return 0;
}
```

**Talking point:** "1631 is Dijkstra with a `max` relaxation — a minimax/bottleneck path. Two solid alternatives: **union-find over edges sorted by difference** (stop when start and end connect), or **binary-search the answer + BFS** to test reachability under a threshold." **Target AC: 30 min.**

---

## Block B — Frontend Refresh (Vue 3 + React)

This is the "can a backend person hold a full-stack conversation" block. You don't need to out-React a React specialist — you need a *crisp mental model* of each framework's reactivity and the trade-offs between them. (Cross-ref: lambdas/functional patterns in `../../03-oop-advanced.md`; immutability reasoning in `../../01-java-language-basics.md`.)

### Vue 3 reactivity

- **Vue 2 → Vue 3 reactivity shift.** Vue 2 used `Object.defineProperty` getter/setter hijacking — it **could not detect** property *additions/deletions* or direct array-index writes (`arr[0] = x`), forcing `Vue.set`/`Vue.delete` hacks. Vue 3 uses the ES6 **`Proxy`**, which intercepts `get`, `set`, `deleteProperty`, `has`, and more — so new keys, deletes, and array index writes are all tracked natively.
- **`ref` vs `reactive`.** `ref(value)` wraps a primitive in a `{ value }` box (you read `.value` in JS; templates auto-unwrap it). `reactive(obj)` returns a deep **Proxy** of an object. **Destructuring a `reactive()` breaks reactivity** — you extract the raw value and drop the Proxy; fix with **`toRefs(state)`** (each field becomes a `ref` that stays connected).
- **`computed`** is **lazy + cached** — it recomputes only when a tracked dependency changes, otherwise it returns the memoized value (like a getter with a cache).
- **Composition API (`<script setup>`)** groups code *by feature* instead of by option type; **composables** (`useAuth()`, `useFetch()`) are the reuse mechanism that replaced Vue 2 mixins (no implicit name clashes, explicit imports).
- **`watch` vs `watchEffect`.** `watch(source, cb)` is explicit and lazy and gives you old + new values; `watchEffect(fn)` auto-tracks whatever it reads and runs **immediately** then on change. Lifecycle hooks: `onMounted` / `onUpdated` / `onUnmounted`.
- **Pinia** (the official store): `defineStore('id', { state, getters, actions })`. Getters are cached like `computed`; you mutate state **directly** inside actions (no Vuex-style mutations); use **`storeToRefs(store)`** to destructure state/getters while keeping reactivity.

```vue
<script setup>
import { ref, reactive, computed, watch, toRefs } from 'vue'

const count = ref(0)                              // primitive → .value in JS
const state = reactive({ first: 'Ada', last: 'L' })
const fullName = computed(() => `${state.first} ${state.last}`) // lazy + cached

const { first } = toRefs(state)                  // keeps reactivity (raw destructure would break it)
watch(count, (n, old) => console.log(old, '→', n))   // explicit, lazy, old+new
</script>
```

### React (hooks + reconciliation)

- **`useState`** returns `[value, setter]`; the setter is **async/batched**, and setting the *same* value (by `Object.is`) **skips the re-render**. Use the functional form `setX(prev => prev + 1)` when the next state depends on the previous.
- **`useEffect(fn, deps)`** runs after commit: `[]` → once after mount; `[a, b]` → whenever `a`/`b` change; **no array** → every render (usually a bug). Return a **cleanup** function (unsubscribes, abort). **Missing deps → stale closure** — the effect closes over old values; the ESLint `exhaustive-deps` rule catches it.
- **Memoization hooks.** `useCallback(fn, deps)` memoizes a *function*, `useMemo(fn, deps)` a *computed value*; `useRef` is a mutable box that **does not** trigger re-render; `useContext` re-renders **all** consumers when the context value changes (split contexts to limit blast radius); `useReducer` for complex state transitions.
- **Reconciliation.** On a state change React builds a **new virtual DOM**, **diffs** it against the previous tree, and **commits** only the minimal real-DOM changes. **Fiber** is the incremental reconciler (work split into units that can pause/resume); React 18 **Concurrent** features make rendering **interruptible** (e.g. `useTransition`, `useDeferredValue`).
- **`key` in lists** gives elements stable identity → React reuses DOM nodes instead of re-rendering. **Never use the array index as a key** for reorderable/insertable lists (it remaps identities and corrupts state). `React.memo` + `useCallback` skips re-render when props are referentially unchanged.

```jsx
function Search({ userId }) {
  const [q, setQ] = useState('');
  useEffect(() => {
    const ctrl = new AbortController();
    fetch(`/api/u/${userId}`, { signal: ctrl.signal }).then(/* ... */);
    return () => ctrl.abort();          // cleanup: cancel stale in-flight request
  }, [userId]);                          // re-run only when userId changes
  const onChange = useCallback(e => setQ(e.target.value), []); // stable identity
  return <input value={q} onChange={onChange} />;
}
```

### Vue vs React — the comparison you'll be asked

| Aspect | Vue 3 | React 18 |
|---|---|---|
| Reactivity | **Proxy**-based, fine-grained dependency tracking | `useState` triggers component + children re-render |
| Optimization | **Compiler** marks static nodes at build time (auto) | Manual: `memo` / `useMemo` / `useCallback` |
| State mgmt | Pinia (first-party) | Redux Toolkit / Zustand / React Query (3rd-party) |
| Templates | SFC `.vue` (HTML/CSS/JS separated) | JSX (logic + markup together) |
| Ecosystem | Smaller, cohesive, first-party router/store | Massive, fragmented |

> **The one-liner:** "Vue's **compiler knows at build time** which parts of a template are dynamic, so updates touch only those nodes; React **diffs the whole virtual tree at runtime** and you opt into memoization manually. Vue is faster for update-heavy UIs out of the box, but React's ecosystem and concurrent-rendering story dominate at scale." Mention you've shipped both (WebX frontend) so it's experience, not trivia.

---

## Block C — Typeahead / Autocomplete System Design

Yesterday's Trie (LC 208/211/1268) is the *data structure*; today is the **system around it**. The interviewer wants scale, latency, the serving path, the offline pipeline, and the cache — not another `insert()`.

### 1. Clarify scope + numbers first

- **Read-heavy, enormous QPS** — a suggestion request fires on **every keystroke**, so it's the highest-volume endpoint in the product.
- **Latency budget: < 100 ms p99** end-to-end, because the dropdown must feel instant. This single number drives every later decision (precompute, cache, edge).
- **Eventual freshness is fine** — suggestions can lag real query trends by minutes/hours; we trade freshness for speed.

### 2. Serving path (online, hot path)

```
client (debounced 150ms, abortable) → API/edge → hot-prefix cache (Redis/edge)
                                                       │ miss
                                                       ▼
                                        sharded in-memory suggestion index
                                        (Trie/FST, top-k precomputed per node)
```

- **Client**: debounce keystrokes (~150 ms) and **abort the previous in-flight request** (the React `AbortController` cleanup pattern above) so a slow earlier response can't overwrite a newer one.
- **Sharded index**: the full Trie/FST is too big for one box → **shard by prefix** (first 1–2 characters, or consistent-hash the prefix). A request routes to the shard owning its prefix.
- **Precomputed top-k at each node**: a lookup is **walk-to-prefix-node → read the stored top-k list** — *not* a request-time DFS. This is what buys the < 100 ms p99. (Exactly the "store precomputed top-k at each prefix node" optimization from yesterday's Trie talking point.)

### 3. Ranking

Rank completions by **historical query frequency (popularity)**, not lexicographically — "fac" must surface **"facebook"** before "facsimile." You can later **blend** recency (trending terms), personalization (user history), and geography into the score. Keep top-k small (k ≈ 5–10) so each node's payload stays tiny.

### 4. Offline pipeline (the part candidates forget)

```
query logs → Kafka → stream/batch aggregation (Flink/Spark or nightly batch)
           → per-prefix frequency counts → BUILD trie/FST with top-k baked in
           → ship to serving shards via a VERSIONED blob swap (atomic pointer flip)
```

- Aggregate raw query logs into **per-term frequency**, fold into **per-prefix top-k**, and **rebuild** the index offline (hourly or daily).
- **Never mutate the serving index in place.** Build a new versioned blob and do an **atomic pointer flip** — readers see either the old or the new index, never a half-built one. This reuses the immutability/versioned-swap discipline from the caching block (Tue) and the event pipeline (Wed/Thu: Kafka → stream processor).

### 5. Cache hot prefixes

Short prefixes ("a", "fa", "the") dominate traffic by a huge margin → put a **prefix → top-k** cache (Redis or CDN edge) in front of the index, **TTL of minutes**. This is **cache-aside + hot-key** reasoning straight from Tuesday's caching block: absorb the dominant short-prefix load, fall through to the sharded index on a miss.

### Talking points that land

- "The trie is the *easy* part; the **interesting** engineering is the **offline log → index pipeline** and the **atomic versioned swap** so the hot path never reads a partial index."
- "I keep request-time work to **walk + read precomputed top-k** — no DFS on the hot path — that's how you hold **< 100 ms p99** at billions of queries."
- "This ties straight into the rest of the week: **Kafka** to ingest query logs, **stream processing** to aggregate, **cache-aside** for hot prefixes, and the **immutable swap** mirrors blue/green index deploys."

---

## 💻 Practice coding questions

1. Write the Dijkstra template (lazy heap, stale-pop skip) from a blank file in ≤10 min; state the relaxation invariant.
2. Network Delay Time (LC 743) — answer = max finalized distance, `-1` if any unreachable.
3. Cheapest Flights Within K Stops (LC 787) — `k+1` Bellman-Ford rounds over a snapshot.
4. Path With Minimum Effort (LC 1631) — Dijkstra with `max` relaxation (bottleneck path).
5. Swim in Rising Water (LC 778) — minimax Dijkstra; compare to union-find-by-elevation.
6. (Stretch) Path with Maximum Probability (LC 1514) — max-heap Dijkstra, multiply probabilities.
7. Implement Vue `computed` semantics in plain JS: lazy + cached, recompute only on dependency change.
8. Write a React search box with debounce + `AbortController` cleanup that can't be clobbered by a stale response.
9. Sketch the typeahead serving path and label the exact step that keeps p99 < 100 ms (read precomputed top-k, no DFS).
10. Draw the offline log → trie pipeline and name where the **atomic versioned swap** happens and why.

---

## 🎤 Interview questions

1. **Why does plain BFS fail on a weighted graph, and what replaces it?** BFS expands by hop count, so the fewest-edge path need not be the cheapest. Dijkstra (min-heap, greedy by tentative distance) handles non-negative weights.
2. **State the Dijkstra relaxation invariant and why it holds.** When a node is popped from the min-heap its distance is final — any other route reaches it through a node still in the heap with distance ≥ the popped one, so it can't be shorter. Requires non-negative weights.
3. **Why does Dijkstra break on negative edges, and what do you use instead?** A negative edge can make an already-"finalized" node cheaper later, violating the invariant. Use Bellman-Ford (`O(V·E)`) for negatives, or Floyd-Warshall for all-pairs.
4. **What is the lazy-deletion trick in your heap, and why use it?** Instead of decrease-key, push the improved `(dist, node)` and skip pops where `d > dist[node]`. Avoids an indexed heap; same correctness, simpler code.
5. **Why is plain Dijkstra wrong for Cheapest Flights Within K Stops?** The cheapest path may exceed the stop budget; finalizing by price alone discards a pricier-but-fewer-stops route that's actually valid. Do `k+1` Bellman-Ford rounds over a distance snapshot.
6. **Why the `snapshot` (clone) in the Bellman-Ford rounds for LC 787?** Relaxing against live `dist` in the same round lets one path consume multiple new edges, breaking the "≤ k stops" cap. The snapshot ensures each round adds at most one edge.
7. **How is Path With Minimum Effort different from normal Dijkstra?** It's a bottleneck/minimax path — cost is the *max* edge along the route, so you relax with `max(currentEffort, |Δ|)` instead of summing. Alternatives: union-find over sorted edges, or binary-search-on-answer + BFS.
8. **Dijkstra time complexity and where the log comes from.** `O(E log V)` — each edge can push one heap entry, and each pop/push is `log` of the heap size (≤ E entries).
9. **Vue 2 vs Vue 3 reactivity — what changed and why?** Vue 2's `Object.defineProperty` couldn't detect property add/delete or array-index writes; Vue 3's `Proxy` intercepts get/set/deleteProperty/has, tracking all of them natively.
10. **Why does destructuring a `reactive()` object break reactivity, and the fix?** Destructuring extracts raw values and drops the Proxy, so further changes aren't tracked. Use `toRefs()` to turn each field into a connected `ref`.
11. **`ref` vs `reactive` vs `computed` in Vue 3?** `ref` boxes a primitive (`.value`); `reactive` deep-proxies an object; `computed` is a lazy, cached derived value that recomputes only when a tracked dep changes.
12. **`useEffect` dependency array — the three cases and the classic bug.** `[]` runs once; `[deps]` runs on change; omitted runs every render. The classic bug is missing deps → stale closure over old state; `exhaustive-deps` lint catches it.
13. **React `useEffect(() => fetchData(), [userId])` and `userId` changes mid-flight?** Cleanup runs and the effect re-runs; the earlier fetch may resolve later and overwrite fresh data — guard with an `isCancelled` flag or `AbortController` in cleanup.
14. **Why never use the array index as a React list `key`?** On insert/reorder the index→element mapping shifts, so React reuses the wrong DOM nodes and component state leaks across items. Use a stable domain id.
15. **Reconciliation and Fiber in one breath.** React builds a new VDOM, diffs it against the old, and commits the minimal real-DOM changes; Fiber is the incremental reconciler that splits work into interruptible units (enabling React 18 concurrent rendering).
16. **Vue vs React performance model — the trade-off.** Vue's compiler marks static vs dynamic nodes at build time so updates are fine-grained automatically; React diffs the whole tree at runtime and you opt into `memo`/`useMemo`/`useCallback`. Vue wins update-heavy UIs out of the box; React wins on ecosystem + concurrency.
17. **Typeahead: how do you hold p99 < 100 ms at billions of queries?** Precompute top-k at each trie/FST node so a request is walk + read (no DFS), shard the index by prefix, and front it with a hot-prefix cache (cache-aside, TTL minutes).
18. **Why rank typeahead by frequency, not lexicographically?** Users expect popular completions ("facebook" before "facsimile"); rank by historical query frequency, optionally blending recency/personalization.
19. **How is the typeahead index refreshed without serving a half-built tree?** Offline pipeline (query logs → Kafka → Flink/Spark → per-prefix top-k → rebuild) ships a *versioned blob*; serving shards do an atomic pointer flip — readers see only the old or new index.
20. **(Curveball) The product wants instant typo tolerance in suggestions.** Add fuzzy matching: an FST with edit-distance traversal (Levenshtein automaton, Lucene-style) or precomputed common-typo expansions; cap the fan-out so the p99 budget holds.

---

## ✅ Self-check

1. Write Dijkstra (lazy heap + stale-pop skip) from a blank file in ≤10 min and state the relaxation invariant and its non-negative-weight precondition.
2. Explain in one sentence why plain Dijkstra is wrong for "≤ k stops" and what you do instead.
3. Explain why destructuring a `reactive()` object breaks reactivity and how `toRefs` fixes it.
4. Describe the React stale-response bug and the `AbortController` cleanup that prevents it.
5. Draw the typeahead system end-to-end (serving path + offline pipeline + hot-prefix cache) and name the exact step that keeps p99 < 100 ms.

---

*Nav: ← [Day 4 (Thu Jul 16)](04-thu-jul-16.md) · [Week 3](README.md) · [Day 6 (Sat Jul 18)](06-sat-jul-18.md) →*
