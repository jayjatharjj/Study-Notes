# Week 5 — Core Build + Soft Applications (Jul 13–19, 2026)

> Theme: Own the fundamentals that every product-company interviewer will test — Trees, caching architecture, indexing internals, and a crisp frontend story. This week you stop being a "backend person who also does Vue" and become a full-stack engineer who can reason at every layer.

---

## 🎯 Week Goal

Walk into any interview and fluently answer:
- Any tree traversal / BST problem (iterative + recursive, no fumbling)
- Cache strategy trade-offs tied to your own production wins
- DB index internals without hand-waving
- A sharp Vue 3 vs React comparison backed by hands-on knowledge
- LCA, diameter, and tree construction — the "hard" tree questions that separate candidates

---

## ✅ By Sunday you can...

- [ ] Write iterative in-order, pre-order, post-order traversals cold (no hints)
- [ ] Implement BFS level-order with a deque, returning `List<List<Integer>>`
- [ ] Solve Validate BST using the min/max bounds approach (not inorder trick)
- [ ] Explain LRU O(1) get/put using HashMap + doubly-linked list (draw it on a whiteboard)
- [ ] Articulate exactly why your 60s→2-3s fix needed both indexes AND caching — and which came first and why
- [ ] Name the three cache write strategies, their failure modes, and when each fits your projects
- [ ] Explain B-tree structure, why PostgreSQL uses it, and what a covering index is
- [ ] Compare Vue 3 reactivity (Proxy-based) vs React reconciliation (virtual DOM diffing) without hedging
- [ ] Describe `ref` vs `reactive`, Pinia store shape, and React `useEffect` dependency array semantics
- [ ] Solve 8 of the 9 target LeetCode problems before the weekend

---

## 📅 Daily Checklist (Mon–Sun)

---

### Monday, Jul 13 — Tree Traversals: Recursive + Iterative

**Time budget: 1.5 hr**

#### DSA (55 min)
- [ ] **Theory warm-up (10 min)**: Draw a sample BST on paper. Trace in-order, pre-order, post-order manually.
  - In-order (L → Root → R) → sorted output on BST
  - Pre-order (Root → L → R) → tree serialization / copy
  - Post-order (L → R → Root) → deletion, size calculation
- [ ] **LC 144 — Binary Tree Preorder Traversal** (Easy) — implement iterative using explicit Stack
  - Pattern: push root → loop → pop → add to result → push right then left (right first so left is processed first)
- [ ] **LC 94 — Binary Tree Inorder Traversal** (Easy) — implement iterative using the "go-left-then-pop" pattern
  - Pattern: `curr = root`; while curr or stack not empty: go left until null, then pop and record, then go right
- [ ] **LC 145 — Binary Tree Postorder Traversal** (Medium) — iterative two-stack trick OR reverse of modified pre-order
  - Trick: pre-order is Root→L→R; if you do Root→R→L and reverse result → post-order
- [ ] **LC 226 — Invert Binary Tree** (Easy) — both recursive and iterative BFS; explain in terms of your mental model

#### Core System Concept (20 min)
- [ ] Read: B-tree structure primer — understand why it's O(log n) for reads, how pages work, and why PostgreSQL stores all actual data in leaf nodes (clustered vs. heap).
  - Key fact: PostgreSQL uses heap storage (table and index are separate files). MySQL InnoDB uses a clustered index (table IS a B+ tree by primary key).
  - Implication for your work: explain why adding an index on `user_id` in your Smart360 N+1 fix mattered.

#### Self-Check (5 min)
- [ ] Without looking: write the iterative in-order skeleton from memory on paper.
- [ ] Can you explain why recursive DFS has O(h) space complexity and what "h" is for a skewed tree?

---

### Tuesday, Jul 14 — Level Order + Tree Problems II

**Time budget: 1.5 hr**

#### DSA (55 min)
- [ ] **LC 102 — Binary Tree Level Order Traversal** (Medium)
  - Use `ArrayDeque` as queue. For each level: record `queue.size()` before the inner loop, pull exactly that many nodes, add children.
  - Return `List<List<Integer>>` — practice the exact Java generics syntax.
  - Must be clean in < 15 lines. If not, keep iterating.
- [ ] **LC 104 — Maximum Depth of Binary Tree** (Easy) — recursive one-liner, then iterative BFS counting levels.
- [ ] **LC 100 — Same Tree** (Easy) — recursive: base cases first (both null → true, one null → false), then values equal AND subtrees same.
- [ ] **LC 543 — Diameter of Binary Tree** (Medium)
  - Diameter through a node = left height + right height.
  - Classic trick: track `maxDiameter` in a 1-element array or instance variable during height DFS.
  - Common mistake: thinking diameter must pass through root — it doesn't. The LeetCode constraint trips people.

#### Core System Concept (20 min)
- [ ] **Composite Indexes** — your 60s→2-3s win.
  - Understand: a composite index `(tenant_id, created_at)` is useful when you filter by `tenant_id` first. The leftmost prefix rule.
  - Covering index: index contains ALL columns needed by a query → index-only scan, never touches the heap. Example: `CREATE INDEX idx_covering ON orders(tenant_id, status, created_at)` when your query is `SELECT tenant_id, status, created_at FROM orders WHERE tenant_id = ?`
  - Tie to Smart360: explain which columns you indexed and why (join columns, filter columns, sort columns).

#### Self-Check (5 min)
- [ ] Write level order from memory with the inner `size` trick.
- [ ] Explain covering index vs regular index in 2 sentences — no jargon.

---

### Wednesday, Jul 15 — BST Operations + LCA

**Time budget: 1.5 hr**

#### DSA (55 min)
- [ ] **LC 98 — Validate Binary Search Tree** (Medium)
  - Wrong approach: just check left < root < right (misses deeper violations).
  - Right approach: DFS with `(node, min, max)` bounds. Each left subtree call passes `max = node.val`; each right subtree call passes `min = node.val`. Start with `(-∞, +∞)`.
  - In Java: use `long` to handle `Integer.MIN_VALUE` / `Integer.MAX_VALUE` edge cases.
- [ ] **LC 235 — Lowest Common Ancestor of BST** (Easy)
  - Exploit BST property: if both p and q are less than root → go left. If both greater → go right. Otherwise, root IS the LCA.
  - O(h) time, O(1) space iteratively.
- [ ] **LC 230 — Kth Smallest Element in BST** (Medium)
  - In-order traversal gives sorted order → count nodes. Iterative in-order with early exit.
  - Follow-up (very common!): "What if the BST is modified often and findKthSmallest is called frequently?" → augment each node with the count of its left subtree. O(h) per operation.

#### Core System Concept (20 min)
- [ ] **Cache-Aside (Lazy Loading)** — the pattern you used for S3 URLs and JWT blacklist.
  - Flow: app checks cache → miss → app queries DB/S3 → app writes to cache → returns data.
  - Failure mode: cache stampede on cold start / TTL expiry under high load. Mitigation: probabilistic early expiration or a mutex lock per cache key.
  - Your Redis S3 URL cache: TTL set to slightly less than the S3 presigned URL expiry (say, URL expires in 15 min → Redis TTL 13 min). Explain this off the cuff — it's a genuine production insight.

#### Self-Check (5 min)
- [ ] Draw the LCA algorithm on BST in pseudocode — 5 lines max.
- [ ] Explain cache stampede to a non-technical interviewer.

---

### Thursday, Jul 16 — Tree Construction + Indexing Deep Dive

**Time budget: 1.5 hr**

#### DSA (55 min)
- [ ] **LC 105 — Construct Binary Tree from Preorder and Inorder Traversal** (Medium)
  - Key insight: pre-order[0] is always the root. Find that value in in-order → everything to its left is the left subtree, everything right is the right subtree.
  - Optimization: HashMap `{value → inorder_index}` for O(1) lookup instead of O(n) linear scan.
  - Practice the recursive signature: `build(preStart, inStart, inEnd)`.
- [ ] **LC 236 — Lowest Common Ancestor of Binary Tree** (Medium) — NOT a BST this time.
  - Recursive post-order: return the node if it equals p or q. Combine results: if left and right are both non-null → current node is LCA. If only one is non-null → return that one.
  - This is harder than BST LCA — make sure you can explain why the algorithm works.
- [ ] Review any two problems from Mon–Wed that felt shaky. Attempt from scratch, timed (15 min each).

#### Core System Concept (20 min)
- [ ] **Write-Through vs Write-Back vs Write-Around**
  - **Write-Through**: write to cache AND DB synchronously. Cache always consistent. Downside: write latency doubles; cache fills with data that may never be read again.
  - **Write-Back (Write-Behind)**: write to cache only, then asynchronously flush to DB. Fast writes, but risk of data loss if cache node dies before flush. Used in CPU L1/L2 caches.
  - **Write-Around**: skip the cache on write, go directly to DB. Cache is populated only on reads (cache-aside style). Good for write-heavy, rarely-re-read data (logs, audit trails).
  - Tie to your projects: your JWT blacklist uses cache-aside (write directly to Redis on logout — that's actually write-through since DB isn't involved). Your S3 URL cache uses write-around (S3 is the source of truth, you only cache on read).

- [ ] **SQL vs NoSQL — a decision framework, not a religion** (the lens the indexing table doesn't give you)
  - **Start from access patterns, not the data.** The question is never "is my data relational?" — it's "what queries do I run, at what scale, with what consistency?" Relational schema and indexes (above) optimize for *unknown future queries*; NoSQL stores optimize for *known query patterns you design the schema around*.
  - **When relational (PostgreSQL) still wins — and it usually does:** multi-entity **joins**, ACID **transactions** across rows/tables, **ad-hoc / analytical queries** you didn't plan for, and **strong consistency** by default. This is why Smart360 and Deep Fathom are PostgreSQL — tenant data with RLS, joins across 50 tables, transactional correctness on authorization. Your honest line: "I start with Postgres because it handles ~95% of use cases correctly; I switch a *slice* of the system to NoSQL when a specific access pattern consistently hits relational limits — polyglot, not replacement."
  - **The NoSQL families and when each fits:**
    - **Key-value (Redis, DynamoDB)** — O(1) lookup by key, no query flexibility. Fits: cache, sessions, rate-limiter state, JWT blacklist. You already use Redis exactly here.
    - **Document (MongoDB)** — self-contained JSON aggregates, flexible/evolving schema, no joins. Fits: a product whose shape changes weekly, or a denormalized read-model where one document = one screen.
    - **Wide-column (Cassandra, HBase)** — massive **write throughput**, **time-series**, partition-key-driven reads of **known query patterns**. Fits: event/audit logs, metrics, IoT/telemetry. You must model the table per query — no ad-hoc filtering.
    - **Graph (Neo4j)** — cheap **relationship traversal** at many hops. Fits: social graphs, permissions/org hierarchies, recommendation paths. This is the case in the Q9 pointer below: SQL self-joins explode at 4+ hops; a graph DB traverses them natively.
  - **Polyglot persistence:** real systems mix — Postgres for the transactional core, Redis for ephemeral/hot state, a wide-column or document store for one high-volume access pattern. The skill is drawing the *seam*: which slice goes where, and how you keep them consistent (usually via events/outbox — Week 6).

#### Self-Check (5 min)
- [ ] Write the Tree Construction (LC 105) signature and base case without looking.
- [ ] Pick a write strategy for a social media "like count" and justify it.

---

### Friday, Jul 17 — Redis Deep Dive + LFU/LRU + Catch-Up

**Time budget: 1.5 hr**

#### DSA (30 min)
- [ ] **LRU Cache Design** — not a LeetCode "solve it" day, a "explain it cold" day.
  - Data structures: `HashMap<key, Node>` + doubly-linked list. Head = MRU end, tail = LRU end.
  - `get(key)`: O(1) HashMap lookup → move node to head → return value.
  - `put(key, val)`: if exists → update + move to head. If new + at capacity → remove tail node AND remove from HashMap → insert new node at head.
  - Draw this on paper. Be able to code it in 20 min under pressure.
  - **LC 146 — LRU Cache** (Medium) — implement it.

#### Core System Concept (35 min)
- [ ] **Redis in your stack — go deep**
  - Data structures you know: String (JWT blacklist), Hash, ZSet (leaderboards/rate limiting)
  - Eviction policies: `allkeys-lru` (evict any key LRU), `volatile-lru` (evict TTL-set keys LRU), `noeviction` (return error when full). Know which to use for a cache vs a durable store.
  - **LFU vs LRU**: LRU evicts the Least Recently Used (recency). LFU evicts the Least Frequently Used (frequency). LFU is better for non-uniform access patterns (avoids evicting frequently accessed old keys). Redis supports `allkeys-lfu` / `volatile-lfu`.
  - Persistence: RDB (point-in-time snapshots) vs AOF (append-only log, fsync configurable). For your JWT blacklist: AOF with `appendfsync everysec` — you need durability (can't un-invalidate a token), but 1s of data loss on crash is acceptable. For the S3 URL cache: RDB only or no persistence — it's just a cache.
  - Redis Cluster vs Redis Sentinel: Sentinel is HA for a single-primary setup (automatic failover). Cluster is horizontal sharding (16384 hash slots). Know which Azure Cache for Redis tiers map to each.

#### Self-Check (15 min)
- [ ] Code LRU Cache from memory. Time yourself — target under 20 minutes.
- [ ] Explain why you chose a specific Redis eviction policy for one of your two real use cases.

---

### Saturday, Jul 18 — Frontend Deep Dive: Vue 3

**Time budget: 4 hr**

#### Morning Session (2 hr): Vue 3 Internals + Composition API

- [ ] **Reactivity System (45 min)**
  - Vue 2 used `Object.defineProperty` — couldn't detect property addition/deletion, couldn't track array index changes.
  - Vue 3 uses ES6 `Proxy` → intercepts ALL operations (get, set, deleteProperty, has). This is why `reactive()` objects are fully reactive without `Vue.set()`.
  - `ref(value)` wraps primitives in a `{ value }` object (since Proxy needs an object to intercept). Accessing in templates: Vue auto-unwraps refs at the top level, so you don't write `.value` in templates.
  - `reactive(object)` returns a Proxy. Destructuring breaks reactivity (you get plain values, not Proxy). Fix: use `toRefs(state)` to convert each property to a ref.
  - `computed(() => ...)` is lazy + cached. Only re-evaluates when its reactive dependencies change. Read `computed` in the context of your Smart360 dashboard filters.

- [ ] **Composition API vs Options API (20 min)**
  - Options API: `data()`, `methods`, `computed`, `watch`, `mounted`. Logic is split by option type — hard to extract reusable logic.
  - Composition API: `setup()` function (or `<script setup>`). Logic grouped by feature, not option. Composables (`useAuth()`, `useDataTable()`) replace mixins.
  - `<script setup>` is syntactic sugar: top-level variables are automatically exposed to the template.

- [ ] **Lifecycle Hooks in Composition API (15 min)**
  - `onMounted`, `onUpdated`, `onUnmounted` (equivalent to `mounted`, `updated`, `beforeDestroy`)
  - `onBeforeMount` runs after `setup()` but before DOM is created. `onMounted` runs after DOM is inserted.
  - `watchEffect(() => { ... })` runs immediately and re-runs when deps change (auto-tracks). `watch(source, callback)` is explicit, lazy, gives old + new value.

- [ ] **Pinia (30 min)** — your state management
  - Define a store: `defineStore('id', { state: () => ({}), getters: {}, actions: {} })`
  - Getters are like `computed` — cached, reactive. Actions can be async.
  - No mutations (unlike Vuex) — you mutate state directly in actions.
  - `storeToRefs(store)` to destructure state/getters while keeping reactivity (same reason as `toRefs`).
  - Tie to your work: describe a store you've actually built (user session, permissions, or UI state).

- [ ] **Practice drill (10 min)**: Without looking, write a `<script setup>` component that:
  - Uses `ref` for a counter
  - Has a `computed` for doubled value
  - Has a button that increments on click
  - Calls an API `onMounted` and stores result in `reactive` state

#### Afternoon Session (2 hr): Vue Deep Cuts + Interview Prep

- [ ] **Virtual DOM in Vue 3 (30 min)**
  - Vue 3 compiler statically analyzes templates and marks dynamic nodes (no need to diff static nodes). This is "compiler-informed virtual DOM" — far more efficient than React's full-tree diffing.
  - `v-if` vs `v-show`: `v-if` conditionally mounts/unmounts (DOM element created/destroyed). `v-show` always mounts, toggles `display: none`. Use `v-show` for frequent toggles, `v-if` for rare ones.
  - `key` prop: Vue (like React) uses keys to identify list items during diffing. Without keys, Vue reuses DOM nodes in order → can cause subtle state bugs.

- [ ] **Common Vue Interview Questions (45 min)** — write out answers, don't just read:
  - How does `v-model` work? It's `:modelValue` + `@update:modelValue` sugar. On custom components: emit `update:modelValue`.
  - What are slots? Default slot, named slots, scoped slots (passing data from child to parent's slot template).
  - What's a composable? A function starting with `use` that uses Composition API internally. How does it differ from a mixin? No namespace collision, explicit source of logic, type-safe.
  - How do you handle async errors globally in Vue? `app.config.errorHandler` or per-component `onErrorCaptured`.

- [ ] **Vue vs React — the comparison you'll be asked (45 min)**
  - Write a 5-point comparison table: Reactivity model | State management | Template vs JSX | Compilation | Learning curve
  - Reactivity: Vue = fine-grained Proxy-based (only re-renders the exact reactive dep that changed). React = coarse-grained (useState triggers re-render of component + children, mitigated by `memo`, `useMemo`, `useCallback`).
  - Templates: Vue SFC separates HTML/CSS/JS (`.vue` files). React uses JSX (JS + HTML mixed). Vue's template compiler can optimize statically.
  - Community/ecosystem: React has larger ecosystem. Vue has more opinionated defaults (Pinia, Vue Router are first-party).
  - When to choose Vue: team already knows it, Laravel/PHP shop, faster to get productive. React: larger talent pool, React Native needed, company standard.

---

### Sunday, Jul 19 — React + Consolidation + Self-Assessment

**Time budget: 4 hr**

#### Morning Session (2 hr): React Hooks + Reconciliation

- [ ] **Hooks Deep Dive (60 min)**
  - `useState`: returns `[state, setter]`. Setter is async (batched in event handlers). Calling setter with same value → no re-render (Object.is comparison).
  - `useEffect(fn, deps)`: runs after render. Deps array controls when:
    - `[]` → runs once after mount (componentDidMount equivalent)
    - `[a, b]` → runs after mount + whenever a or b change
    - No array → runs after every render (usually a bug)
    - Return a cleanup function → runs before next effect or on unmount
  - Common mistake: missing deps causes stale closure (effect captures old variable value). ESLint `exhaustive-deps` rule catches this.
  - `useCallback(fn, deps)`: memoizes a function reference. Use when passing callbacks to memoized children.
  - `useMemo(fn, deps)`: memoizes a computed value. Use for expensive computations.
  - `useRef`: mutable ref object (`ref.current`). Two uses: (1) DOM node access, (2) mutable value that doesn't trigger re-render (like `setInterval` ID).
  - `useContext(MyContext)`: consume context without nesting. Context changes re-render ALL consumers — use sparingly or split contexts.
  - `useReducer(reducer, initialState)`: like Redux but local. Use when state transitions are complex (multiple sub-values, next state depends on previous).

- [ ] **Reconciliation + Virtual DOM (30 min)**
  - React maintains a VDOM tree. On state change, React creates a new VDOM and diffs it against the previous (diffing algorithm). Changed nodes are applied to the real DOM in a "commit phase."
  - React 18 Concurrent Mode: rendering is interruptible. React can pause rendering to handle higher-priority updates (user input > data fetch).
  - Fiber: React's internal reconciler. Each component is a "fiber" unit of work. React can process fibers incrementally.
  - `key` in lists: without keys, React diffs by position → inserting at start is O(n) mutations. With stable keys → O(1) identification. Never use array index as key if list can be reordered/filtered.
  - `React.memo(Component)`: skips re-render if props haven't changed (shallow equality). Pair with `useCallback` for stable function props.

- [ ] **State Management in React (30 min)**
  - Local: `useState`, `useReducer`
  - Context: built-in, good for infrequent updates (theme, auth). Bad for high-frequency updates (triggers all consumers).
  - Redux Toolkit: `createSlice`, `createAsyncThunk`, `RTK Query`. Still the enterprise standard.
  - Zustand: lightweight, no boilerplate. `const useStore = create(set => ({ count: 0, inc: () => set(s => ({ count: s.count + 1 })) }))`.
  - React Query / TanStack Query: server state (caching, refetching, background sync). Pair with Zustand for client state.

#### Afternoon Session (2 hr): Consolidation + Full Review

- [ ] **Drill: React vs Vue explanation (20 min)**
  - Practice saying the Vue vs React comparison out loud (not reading). Time yourself — 3 minutes max, crisp.
  - Key differentiator you can use: "In Vue, the compiler knows at build time which parts of the template are dynamic. React diffs the whole virtual tree at runtime. Vue 3's approach is faster for update-heavy UIs but React's ecosystem and concurrent features make it the dominant choice at scale."

- [ ] **System Design Review — tie everything together (40 min)**
  - Draw a caching architecture diagram for your Smart360 project: Client → API Gateway (rate limit via Redis) → Spring Service → [Redis cache? hit] → [PostgreSQL? miss + write back to Redis] → S3 for files (URL cached in Redis with TTL).
  - Annotate: which cache strategy at each layer, what eviction policy, what TTL, what happens on a cache miss.
  - Practice narrating this as if answering "Walk me through your system's caching strategy."

- [ ] **DSA Review Session (40 min)**
  - Re-solve two of the hardest problems from the week cold (pick from: LC 105, LC 236, LC 98)
  - Time yourself: target 20 min each
  - Focus on talking while coding — narrate your thought process

- [ ] **Behavioral prep (20 min)** — write fresh STAR answers for:
  - "Tell me about a time you improved frontend performance." (tie to Vue component optimization, lazy loading, or Pinia store design)
  - "How do you decide between SQL and NoSQL?" (tie to your PostgreSQL choice for relational data + Redis for ephemeral/cache data)

---

## 🎯 Build Your Tiered Target-Company List (30 min — do it this week, not in Week 9)

You don't apply until Week 8+, but the *list* needs to exist now. Right now the plan improvises company names ad hoc when applications start. That's backwards — building a curated, living sheet in Week 5 means your referral outreach (LinkedIn warm-up, "alumni at company" pings) can start warming up weeks before you actually apply. Cold-applying in Week 9 with no relationships is the lowest-yield path; a warm referral at a GCC is 3–5× more likely to convert to a first round.

Create a single living sheet (a tab in your application tracker, or a separate Google Sheet) of **~30–40 companies in 3 tiers**:

- **Tier 1 — Target GCCs (your best odds + cloud-profile fit):** the GCCs that hire heavily at 2–4 YOE and value AWS/Azure + microservices + K8s. This is your highest-probability, highest-comp tier — aim for 12–15 names here. Examples in your band: Walmart Global Tech, Barclays, SAP Labs, Bosch, Société Générale, Deutsche Bank, Wells Fargo, Target, Lowe's, JPMC, Mastercard, Visa, ServiceNow, Nvidia/Adobe GCCs in Pune/Bengaluru/Hyderabad.
- **Tier 2 — Product mid-tier / Indian unicorns:** product companies where the comp band and engineering bar are strong but the loop is shorter than top-tier. Aim for 10–12 names — Razorpay, Zerodha, PhonePe, Swiggy, Zeta, Postman, Browserstack, Hasura, CRED-adjacent, mid-stage SaaS.
- **Tier 3 — Warm-up services / mid-size:** the practice-rep companies you hit first in Week 8 to calibrate your interviewing (NOT your target offers). Aim for 8–10 — established services firms and mid-size product shops with a fast cycle.

**Concrete sourcing method (don't guess — mine these):**
- [ ] **Zinnov / NASSCOM GCC reports** — they publish lists of active GCCs by city and sector. This is the authoritative source for "who actually has a GCC in Pune/Bengaluru/Hyderabad."
- [ ] **Levels.fyi company directory** — browse by company; gives you the comp bands so you can pre-filter for your ₹14–25 LPA target before you ever apply.
- [ ] **AmbitionBox** — use the **GCC filter** and the company explorer; cross-check reported salary bands and interview-round structure per company.
- [ ] **LinkedIn** — for each shortlisted company run two searches: **"alumni at <company>"** (people from your college / past employer — warmest referral path) and **"engineers at <company>"** (filter to your stack — Java/Spring). Note 2–3 names per Tier 1 company to reach out to in Week 6–7.
- [ ] **Instahyre / Cutshort listings** — browse open roles to confirm a company is actively hiring at your level *right now*, and to surface companies you hadn't thought of.

**Sheet columns:** Company | Tier | City | Has GCC? (Zinnov-confirmed) | Comp band (Levels/AmbitionBox) | Stack match | Referral contact (LinkedIn) | Open role link | Notes. Keep it living — add and prune as you research.

> Why now: referrals take time to warm. A connection request + a genuine note in Week 6, a follow-up in Week 7, then an application in Week 9 converts far better than a cold apply. Building the list in Week 5 is what makes that sequence possible.

---

## 🧠 Concepts to Master This Week

### Trees
| Concept | Key Insight | Common Mistake |
|---|---|---|
| Iterative In-Order | Use stack: go left until null, pop, record, go right | Forgetting the go-right step after pop |
| Iterative Post-Order | Reverse of (Root→R→L) pre-order variant | Trying to do it with one stack (use two) |
| Level Order | Record `queue.size()` before inner loop for level grouping | Using size inside loop (size changes as you enqueue children) |
| Validate BST | Pass `(min, max)` bounds down recursion | Checking only parent-child relationship (misses grandparent violation) |
| LCA — BST | Exploit BST: both < root → go left; both > root → go right; else → root | Treating it like general tree (overkill) |
| LCA — General Tree | Post-order: if left and right both return non-null → current node | Confusing this with BST LCA |
| Tree Construction | Pre-order[0] = root; find in in-order array; recurse | O(n²) lookup; fix with HashMap |
| Diameter | Track max(left_height + right_height) during height DFS | Assuming diameter must pass through root |
| Kth Smallest in BST | In-order traversal; count k | Not knowing the follow-up (augmented BST) |

### Caching
| Strategy | Write Behavior | Read Behavior | Best For | Risk |
|---|---|---|---|---|
| Cache-Aside | App writes to DB directly; cache populated on read misses | Check cache → miss → read DB → write cache | Read-heavy, tolerate stale | Stampede on miss, cold start |
| Write-Through | Write to cache + DB synchronously | Always reads from cache | Read-heavy, consistency critical | Double write latency |
| Write-Back | Write to cache only; async flush to DB | Always reads from cache | Write-heavy, latency sensitive | Data loss if cache dies before flush |
| Write-Around | Write directly to DB, bypass cache | Cache populated only on reads | Write-once, read-rarely (logs) | High read latency on first access |

### DB Indexing
- **B-tree**: self-balancing tree; O(log n) reads; default PostgreSQL index type; supports range queries (`>`, `<`, `BETWEEN`)
- **Hash index**: O(1) exact match; does NOT support range queries; rarely used
- **Composite index `(a, b, c)`**: effective for queries filtering on `a`, `a+b`, or `a+b+c`. Leftmost prefix rule — `(b, c)` alone doesn't use the index.
- **Covering index**: includes all columns a query needs → index-only scan (no heap access). Fastest possible read.
- **Partial index**: `CREATE INDEX ON orders(status) WHERE status = 'PENDING'` — tiny index, fast for that filter
- **Index selectivity**: high cardinality columns (user_id, email) make better index targets than low-cardinality (boolean, status with 2 values)

### Vue 3 vs React — Side by Side
| Aspect | Vue 3 | React 18 |
|---|---|---|
| Reactivity | Proxy-based, fine-grained (tracks exact deps) | useState triggers full component re-render |
| Optimization | Compiler marks static nodes; no manual memo | `React.memo`, `useMemo`, `useCallback` needed |
| State Mgmt | Pinia (official, Composition API native) | Redux Toolkit / Zustand / React Query |
| Templates | SFC (`.vue`): HTML + JS + CSS separated | JSX: HTML in JS; CSS-in-JS or modules |
| Learning Curve | Lower (options API familiar; SFC clear separation) | Higher (hooks mental model, JSX, ecosystem choices) |
| Ecosystem | Smaller but cohesive | Massive, fragmented |

---

## 🎤 Sample Interview Questions (incl. Curveballs) — 12 Questions

**1. "Walk me through all four tree traversal orders and when you'd use each in a real system."**
- Pointer: In-order → sorted BST output (used for rangequeries). Pre-order → serialize/copy a tree, or evaluate expression trees (prefix notation). Post-order → delete a tree bottom-up (children before parents), size/disk-usage computation. Level-order → BFS graph search, print tree level by level, shortest-path in unweighted graph.

**2. "Your Redis cache for JWT blacklisting — what happens if Redis goes down?"**
- Pointer: This is a security-correctness trade-off. If Redis is unavailable and you fail-open (allow all tokens), invalidated tokens are suddenly valid again. If you fail-closed (reject all requests), the system goes down. Production answer: Redis Sentinel or Cluster for HA; if Redis is truly unavailable, fail-closed on auth (return 503) for the duration. Document this as an accepted trade-off. Also: AOF persistence so Redis can recover quickly with minimal data loss.

**3. "Curveball: You have a write-through cache. A cache node dies after you write to it but before the write reaches the DB. What happens?"**
- Pointer: If the write hit the cache but DB write failed (transactional failure) → cache has stale data, DB doesn't. On next read → cache hit returns stale data. Fix: write-through must be atomic or use write-through only with idempotent operations + TTL as a safety net. This is why write-through alone doesn't solve consistency — you need DB as the source of truth.

**4. "Explain the leftmost prefix rule for composite indexes with a concrete example."**
- Pointer: Index `(tenant_id, status, created_at)`. Queries that can use it: `WHERE tenant_id = ?`, `WHERE tenant_id = ? AND status = ?`, `WHERE tenant_id = ? AND status = ? AND created_at > ?`. Query that CANNOT use it: `WHERE status = ?` alone (skips leftmost column). Why: B-tree is sorted by leftmost key first, then second, etc. Without the first key, you'd have to scan the entire index.

**5. "You mentioned N+1 queries. How would you detect one in production without access to Hibernate statistics?"**
- Pointer: Slow query log in PostgreSQL (`log_min_duration_statement = 100ms`) → you'd see hundreds of nearly identical queries (same structure, different `WHERE id = ?` values) in a short time window. In observability: distributed trace shows many short sequential DB spans for a single request. Also: `p6spy` or `datasource-proxy` in non-prod for logging all queries with stack traces.

**6. "Curveball: In Vue 3, why does destructuring a `reactive()` object break reactivity?"**
- Pointer: `reactive()` returns a JavaScript Proxy. Proxies intercept property access on the proxy OBJECT. When you destructure `const { name } = state`, you extract the raw value at that moment — a primitive or a plain reference. The Proxy is no longer in the chain. Fix: `const { name } = toRefs(state)` — `toRefs` converts each property to a ref (which is itself reactive) before you destructure.

**7. "What's the difference between LRU and LFU? When would you choose each for a cache?"**
- Pointer: LRU evicts the item that hasn't been accessed for the longest time. LFU evicts the item accessed least often over all time. LRU is great for temporal locality (recent data is likely to be accessed again). LFU is better for non-uniform access patterns where some items are always popular regardless of recency. Real Redis configuration: `allkeys-lru` for a general cache; `allkeys-lfu` if you have hot items that should never be evicted even after a quiet period.

**8. "Curveball: If you're solving Lowest Common Ancestor on a general binary tree, how does your algorithm handle the case where one of the target nodes IS the LCA?"**
- Pointer: The recursive algorithm returns the node when it matches p or q. If p is an ancestor of q, then when we recurse down from p, we find p first and return it. The other branch eventually returns null (q is in the subtree below p, so from p's perspective the left and right calls will find q but p is already returned). Wait — actually: if p = ancestor and q = descendant, then when we reach p, we return p (base case). In the sibling subtree we find nothing → return null. So at p's parent: one side returns p, other returns null → returns p. Correct! But only if we don't recurse INTO p's subtree once we've found p. The standard algorithm does NOT stop at p — it recurses through p's subtree too. However, since it returns p as soon as p is found, it doesn't matter what's below — the caller gets p and propagates it up. The result is correct.

**9. "SQL vs NoSQL — when would you NOT use PostgreSQL even though you clearly know it well?"**
- Pointer: (1) Document-heavy, schema-flexible data that evolves rapidly — MongoDB. (2) Time-series data with billions of rows — TimescaleDB or InfluxDB (though Timescale IS PostgreSQL). (3) Graph relationships with many hops — Neo4j (SQL joins get expensive at 4+ hops). (4) Key-value with microsecond latency at extreme scale — Redis or DynamoDB. (5) Event streaming/append-only log — Kafka. Your honest answer: "I start with PostgreSQL because it handles 95% of use cases correctly, but I'd switch when query patterns consistently hit its limits."

**10. "Curveball: In React, you have `useEffect(() => { fetchData(); }, [userId])`. The userId changes while a fetch is still in-flight. What happens?"**
- Pointer: React triggers cleanup and re-runs the effect. The old fetch is still in-flight and will resolve — if you set state from it, you'll overwrite the new fetch's data with stale data. Fix: use the cleanup function to set an `isCancelled` flag or use `AbortController`. `const controller = new AbortController(); fetch(url, { signal: controller.signal }); return () => controller.abort();` The aborted fetch throws an `AbortError` — catch and ignore it.

**11. "Your CI/CD pipeline uses Docker BuildKit cache. How would you ensure cache hits are maximized for a Spring Boot app?"**
- Pointer: Layer ordering is critical. In a Spring Boot Dockerfile: (1) Copy `pom.xml` / `build.gradle` first → run `mvn dependency:go-offline` → this layer is cached as long as deps don't change. (2) Then copy source code → build. The dependency layer is ~200MB and changes infrequently. Source layer is small and changes every commit. Without this ordering, every source change invalidates the dependency layer → full re-download every build.

**12. "Curveball: You add an index to a 100M-row production table in PostgreSQL. What happens during the index creation and how do you avoid downtime?"**
- Pointer: Standard `CREATE INDEX` takes an `AccessShareLock` — blocks writes for the entire duration (could be minutes on 100M rows). Production solution: `CREATE INDEX CONCURRENTLY idx_name ON table(col)` — takes a weaker lock, allows reads and writes during build. Trade-off: takes longer, can fail (then you have an INVALID index you must drop and retry). Always use CONCURRENTLY in production. Also: run during low-traffic period, monitor `pg_stat_progress_create_index`.

---

## 🌟 Extraordinary-Candidate Edge

These are the things average candidates don't say. Drop one or two naturally — don't recite a list.

**On Caching:**
> "When we cached S3 pre-signed URLs in Redis, I set the TTL to 13 minutes against a 15-minute URL expiry — not 14:59. The extra buffer accounts for clock skew between the app server and S3 and the latency of serving the cached URL before the client actually uses it. Getting burned by a 'valid' cached URL that expired by the time the client used it would be worse than a slightly shorter TTL."

**On Indexing:**
> "After adding the composite index, I ran `EXPLAIN (ANALYZE, BUFFERS)` — not just EXPLAIN. The BUFFERS output showed that cache hit ratio on the index pages was already near 100% after warm-up because the index was small enough to fit in PostgreSQL's `shared_buffers`. That told me further caching at the application layer was likely unnecessary for that query path."

**On BST / Data Structures:**
> "The iterative in-order traversal using a stack isn't just a LeetCode trick — it's exactly how a database B-tree cursor works. When PostgreSQL scans a range via an index, it maintains a stack of internal nodes to traverse. This is why understanding tree traversal matters beyond algorithms."

**On Vue vs React:**
> "I've shipped production code in both. The decision I've come to: Vue's SFC model and Pinia make team onboarding faster — juniors write correct, readable code sooner. React's ecosystem and concurrent features make it the right call when you need React Native parity or when you're in an organization where React skill is already ubiquitous. I don't have a strong opinion beyond context — the wrong framework chosen fast beats the right framework chosen late."

**On LLM Integration (if it comes up):**
> "Long-running LLM jobs taught me that the client-visible latency and the actual compute latency are completely different problems. The 202 + jobId pattern solved the HTTP timeout problem, but the harder problem was the user-experience problem: users don't understand 'we're working on it.' What actually improved satisfaction was streaming partial results via SSE — users felt the system was responsive even when the total time hadn't changed."

---

## 📊 End-of-Week Self-Assessment

Rate yourself 1–5 on each. Anything below 4 goes on next week's focus list.

| Area | Target | Your Rating | Notes |
|---|---|---|---|
| Iterative tree traversals (all 3) | 5 | | Write cold without hints |
| Level-order BFS (clean Java) | 5 | | Must return `List<List<Integer>>` correctly |
| Validate BST (bounds approach) | 4 | | Edge cases: `Integer.MIN/MAX_VALUE` |
| LCA — BST (iterative) | 5 | | Should be trivial with BST property |
| LCA — General Tree (recursive) | 4 | | Explain why the base case works |
| Tree Construction from pre+in | 4 | | HashMap optimization |
| Diameter of Binary Tree | 4 | | Instance variable / array trick |
| Kth Smallest + follow-up | 4 | | Know the augmented BST follow-up |
| Cache strategies (all 4) | 5 | | Tie each to a real system |
| Redis eviction policies | 4 | | Name 3+, say which you'd pick and why |
| LRU Cache implementation | 4 | | HashMap + DLL, O(1) get+put |
| B-tree + covering index | 4 | | Leftmost prefix, CONCURRENTLY creation |
| Vue 3 reactivity (Proxy, ref, reactive) | 4 | | Explain destructuring problem |
| Composition API + Pinia | 4 | | Write a composable from memory |
| React hooks (useEffect deps, stale closure) | 4 | | AbortController pattern |
| Vue vs React comparison (spoken) | 4 | | 3 min, confident, no hedging |

**End-of-week threshold**: If 12/16 items are rated 4+, you're on track. If fewer than 10, carve out an extra 2 hours from Weekend Day 2 to revisit gaps before moving to Week 6.

**Red flags to catch now (before Week 6 starts):**
- [ ] Can I write iterative in-order traversal in < 5 minutes without looking? (If no → drill Mon morning)
- [ ] Can I explain LRU cache design without drawing it first? (If no → drill Fri morning)
- [ ] Can I say "Vue vs React" for 2 minutes without saying "um, it depends" and stopping? (If no → record yourself and replay)

---

*Week 5 of interview prep — built on interview-qa.md. Week 6 target: dynamic programming foundations + distributed systems (Kafka, event-driven architecture, idempotency) + behavioral stories refinement.*
