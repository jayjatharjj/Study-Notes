# Week 2 · Day 4 — Thu Jul 09 — Linked Lists + LRU Cache + REST Versioning / Idempotency

> Linked lists are the structure every interviewer reaches for to test pointer discipline, and the LRU cache is where DSA and system design meet — a `HashMap` + doubly-linked list that you must be able to write cold. On the backend side, today is about the contracts that keep a public API stable and safe under retries: versioning, pagination, and idempotency keys.

📌 **Study today:** Linked lists — reversal, dummy head, Floyd's, two-pointer (LC 206, 21, 141, 142, 19, 143) · REST versioning + pagination + idempotency · LRU cache (LC 146) · ⏱ ~6 hr (A ~2.5h · B ~2h · C ~1.5h)

---

## Block A — DSA: Linked Lists (dummy head · two-pointer · reversal)

### Theory

A singly linked list node is `{ val, next }`. There is no random access — every traversal is O(n) and you only ever hold pointers, never indices. Three patterns cover ~90% of linked-list interview questions:

1. **Dummy head (sentinel)** — allocate a fake node `dummy` whose `next` is the real head. It removes the "what if I modify/delete the head?" special case, because the head is now just another `node.next`. Return `dummy.next` at the end. Use it for every problem that builds or mutates a list from the front.
2. **Two-pointer** — two pointers moving at different speeds or with a fixed gap:
   - *slow/fast (Floyd's)* — slow +1, fast +2; used for cycle detection and finding the midpoint.
   - *fixed-gap* — advance `fast` `n` steps first, then move both together; used for "nth from end".
3. **Reversal (3 pointers)** — `prev`, `curr`, `next`. Save `next`, point `curr.next` back to `prev`, advance both. At the end `prev` is the new head.

**Golden rule:** draw boxes-and-arrows for a 3-node example *before* writing a single line. Most linked-list bugs are lost-pointer bugs (you overwrote `node.next` before saving it) that a 30-second diagram catches.

```java
class ListNode {
    int val;
    ListNode next;
    ListNode() {}
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}
```

### Worked example — Reverse Linked List (LC 206)

Iterative reversal is the single most important linked-list primitive: it appears as a *sub-step* in Reorder List, Palindrome List, and Reverse Nodes in k-Group.

```java
// Iterative — O(n) time, O(1) space
public ListNode reverseList(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode next = curr.next; // 1. SAVE next (or you lose the rest of the list)
        curr.next = prev;          // 2. REVERSE this link
        prev = curr;               // 3. advance prev
        curr = next;               // 4. advance curr
    }
    return prev; // prev is the new head; curr is now null
}

// Recursive — O(n) time, O(n) stack space
public ListNode reverseRecursive(ListNode head) {
    if (head == null || head.next == null) return head; // base: empty or last node
    ListNode newHead = reverseRecursive(head.next);     // reverse the rest
    head.next.next = head;  // the node after me should point back to me
    head.next = null;       // I become the new tail
    return newHead;         // newHead bubbles up unchanged
}
```

Trace `1→2→3→null` iteratively:
- start: prev=null, curr=1
- iter1: next=2, 1.next=null, prev=1, curr=2 → list so far `1→null`
- iter2: next=3, 2.next=1, prev=2, curr=3 → `2→1→null`
- iter3: next=null, 3.next=2, prev=3, curr=null → `3→2→1→null`
- return prev=3. Done.

**Complexity:** O(n) time, O(1) space iterative; O(n) stack recursive. State both; the interviewer often wants the O(1)-space version explicitly.

### Practice set

| # | Problem (LC) | Approach | Key insight | Complexity | Follow-up |
|---|---|---|---|---|---|
| 1 | Reverse Linked List (206) | 3-pointer iterative + recursive | Save `next` before rewiring | O(n) / O(1) | Reverse only nodes `[m,n]` (LC 92) |
| 2 | Merge Two Sorted Lists (21) | Dummy head, splice smaller each step | Dummy removes head special-case; attach the remaining list at the end (don't loop it) | O(n+m) / O(1) | Merge k lists with a min-heap (LC 23) |
| 3 | Linked List Cycle (141) | Floyd's slow/fast | They meet iff a cycle exists; no cycle → fast hits null | O(n) / O(1) | Why does fast +2 guarantee a meeting? (gap shrinks by 1 each step) |
| 4 | Linked List Cycle II (142) | Floyd's, then reset one to head | After meeting, move both +1 from head & meet point → they meet at cycle entry | O(n) / O(1) | Prove it: 2(F+a)=F+a+nC ⇒ F=nC−a |
| 5 | Remove Nth From End (19) | Fixed-gap two-pointer + dummy | Advance `fast` n steps, then both until `fast.next==null` | O(n) / O(1) | Single pass required — no length precompute |
| 6 | Reorder List (143) | midpoint → reverse 2nd half → interleave | Three sub-steps; most candidates skip the reverse | O(n) / O(1) | Same skeleton as Palindrome List (LC 234) |

**Merge Two Sorted Lists** — the dummy-head template you reuse everywhere:

```java
public ListNode mergeTwoLists(ListNode a, ListNode b) {
    ListNode dummy = new ListNode(0), tail = dummy;
    while (a != null && b != null) {
        if (a.val <= b.val) { tail.next = a; a = a.next; }
        else                { tail.next = b; b = b.next; }
        tail = tail.next;
    }
    tail.next = (a != null) ? a : b; // attach the non-empty remainder in O(1)
    return dummy.next;
}
```

**Cycle II (LC 142)** — know the proof, interviewers ask "why does this work?":

```java
public ListNode detectCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {            // cycle confirmed
            ListNode p = head;
            while (p != slow) { p = p.next; slow = slow.next; }
            return p;                  // cycle entry
        }
    }
    return null;
}
```
Math: let `F` = distance head→entry, `a` = entry→meet, `C` = cycle length. Fast travels twice slow: `2(F+a) = F+a+nC` ⇒ `F = nC − a`. So walking `F` from head and `F` from the meet point lands both at the entry.

---

## Block B — REST Versioning, Pagination, Idempotency

### Resource naming

REST resources are **nouns, not verbs**: `GET /users/{id}`, never `/getUser`. Collections are plural (`/users`), sub-resources nest (`/users/{id}/orders`). The HTTP verb carries the action; the URI names the thing. Filtering, sorting, and pagination are query params, not path segments: `GET /users?status=active&sort=-createdAt&page=2`.

### Status codes (memorize cold)

| Code | Meaning | When |
|---|---|---|
| 200 | OK | Successful GET/PUT/PATCH with body |
| 201 | Created | POST created a resource (return `Location` header) |
| 204 | No Content | Successful DELETE / PUT with no body |
| 400 | Bad Request | Malformed syntax / unparseable body |
| 401 | Unauthorized | Missing/invalid auth — *not authenticated* |
| 403 | Forbidden | Authenticated but *not allowed* (RBAC denial) |
| 404 | Not Found | Resource absent (or hidden for security) |
| 409 | Conflict | State conflict — version mismatch, duplicate unique key |
| 422 | Unprocessable Entity | Syntax OK but semantically invalid (validation) |
| 429 | Too Many Requests | Rate limited (send `Retry-After`) |
| 500 / 502 / 503 | Server error / bad gateway / unavailable | Internal failure / upstream down / overloaded |

**401 vs 403:** 401 = "I don't know who you are" (bad/absent token); 403 = "I know who you are, you can't do this." **409 vs 422:** 409 = conflict with current *state* (optimistic-lock version stale, duplicate email); 422 = the *payload* is well-formed JSON but breaks a business rule (negative price).

### Versioning strategies

| Strategy | Example | Pro | Con | Use for |
|---|---|---|---|---|
| URI path | `/v1/users` | Most visible, trivial gateway routing | Breaks bookmarks; "v1 forever" | Public APIs |
| Header | `Accept: application/vnd.app.v2+json` | Clean URIs, true REST | Hard to test in a browser, less discoverable | Internal services |
| Query param | `/users?version=2` | Easy to test/curl | Not purist; easy to forget | Quick internal toggles |

Pick **URI versioning for public APIs** (gateways route on path; consumers see the version), **header versioning for internal service-to-service**. Breaking-change protocol: ship the new version alongside the old, send `Deprecation: true` and `Sunset: <RFC1123 date>` headers on the old one, and prefer **additive** changes (new optional fields) so you never need a new version at all.

```java
@RestController
@RequestMapping("/v1/users")        // URI versioning — most common
public class UserV1Controller { /* ... */ }

// Header-based variant — same path, version selected by Accept
@GetMapping(value = "/users", produces = "application/vnd.app.v2+json")
public List<UserV2Dto> listV2() { /* ... */ }
```

### Pagination — offset vs cursor

```java
// Offset (page/size) — simple, but Postgres scans+discards the skipped rows
// SELECT * FROM users ORDER BY id LIMIT 20 OFFSET 100000;  ← slow at depth
public record PageResponse<T>(List<T> data, long totalCount, int page, int size) {}

// Cursor / keyset — O(log n) via index seek, stable under inserts/deletes
// SELECT * FROM users WHERE id > :lastSeenId ORDER BY id LIMIT 20;
public record CursorResponse<T>(List<T> data, String nextCursor) {} // base64(lastSeenId)
```

- **Offset** (`?page=2&size=20`): simple, supports "jump to page N", but degrades on large offsets (the DB still walks the skipped rows) and is **inconsistent under concurrent inserts** — an insert shifts every later row, so page 2 can repeat or drop items.
- **Cursor/keyset** (`?after=<base64 of last id>`): O(log n) index seek, stable under inserts/deletes, the right choice for feeds and infinite scroll. Trade-off: no random page access, cursor must encode a stable sort key.

> **Project tie-in (Smart360):** the data service moved from offset to cursor pagination because frequent inserts made offset pages inconsistent — users saw duplicate rows when scrolling a live-updating list. Return `nextCursor` plus `Link` headers (RFC 5988) so clients don't construct URLs by hand.

### Idempotency

An operation is **idempotent** if N identical calls leave the server in the same state as one call.

| Method | Idempotent? | Note |
|---|---|---|
| GET | ✓ | Safe + idempotent (no side effects) |
| PUT | ✓ | Full replace → same result every time |
| DELETE | ✓ | 2nd call returns 404 or 204 — state is identical either way |
| POST | ✗ | Creates a new resource each call |
| PATCH | ✗ (debated) | `{increment: 5}` is not idempotent; `{set: 5}` is |

Make **POST safe to retry** with an `Idempotency-Key: <uuid>` header. The server stores `key → response` in Redis with a TTL; a duplicate request returns the cached result **without re-executing**. This is the bank-transfer retry pattern: a client whose connection dropped after the charge but before the response can safely retry without double-charging.

```java
@PostMapping("/transfers")
public ResponseEntity<TransferResult> transfer(
        @RequestHeader("Idempotency-Key") String key,
        @Valid @RequestBody TransferRequest req) {
    TransferResult cached = idempotencyStore.get(key); // Redis GET
    if (cached != null) return ResponseEntity.ok(cached); // replay, no re-charge
    TransferResult result = transferService.execute(req);
    idempotencyStore.put(key, result, Duration.ofHours(24)); // SET key val EX 86400
    return ResponseEntity.status(HttpStatus.CREATED).body(result);
}
```

**Gotchas:** store the key *before* executing (or use `SET NX` to atomically claim it) to avoid a race where two concurrent retries both execute; key the cache per-endpoint+user so keys can't collide across operations; on a still-in-flight duplicate, return 409 or block until the first completes.

---

## Block C — LRU Cache (LC 146)

### Theory

An LRU (Least Recently Used) cache evicts the entry untouched for the longest time when it hits capacity. The requirement is **O(1) `get` and `put`**, which forces a two-structure design:

- **`HashMap<Key, Node>`** → O(1) lookup by key.
- **Doubly-linked list** → O(1) move-to-front and O(1) tail eviction. Most-recently-used at the head, least-recently-used at the tail.

Every `get` and `put` (on an existing key) **moves the node to the head**. On `put` over capacity, **evict the tail**. The DLL gives O(1) removal because each node knows its `prev` (a singly-linked list can't remove in O(1) — you'd need to scan for the predecessor).

**Sentinel head/tail nodes** (dummy nodes that always exist) eliminate every null check: the real first node is `head.next`, the real last is `tail.prev`, and you never special-case an empty list.

### Quick form (interview-acceptable)

```java
// LinkedHashMap in access-order mode — say this first, then offer the full version
class LRUCache extends LinkedHashMap<Integer, Integer> {
    private final int capacity;
    LRUCache(int capacity) {
        super(capacity, 0.75f, true); // accessOrder=true → get() moves to end
        this.capacity = capacity;
    }
    @Override protected boolean removeEldestEntry(Map.Entry<Integer,Integer> e) {
        return size() > capacity; // auto-evict eldest on overflow
    }
    public int get(int key)            { return super.getOrDefault(key, -1); }
    public void put(int key, int val)  { super.put(key, val); }
}
```

### Preferred full implementation — HashMap + DLL (write this cold, target ≤30 min)

```java
class LRUCache {
    private static class Node {
        int key, val;
        Node prev, next;
        Node(int key, int val) { this.key = key; this.val = val; }
    }

    private final int capacity;
    private final Map<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0); // sentinel MRU side
    private final Node tail = new Node(0, 0); // sentinel LRU side

    public LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        Node node = map.get(key);
        if (node == null) return -1;
        moveToHead(node);     // mark as most-recently-used
        return node.val;
    }

    public void put(int key, int value) {
        Node node = map.get(key);
        if (node != null) {           // update existing
            node.val = value;
            moveToHead(node);
            return;
        }
        Node fresh = new Node(key, value);
        map.put(key, fresh);
        addToHead(fresh);
        if (map.size() > capacity) {  // evict LRU
            Node lru = tail.prev;
            remove(lru);
            map.remove(lru.key);      // must remove from BOTH structures
        }
    }

    private void addToHead(Node n) {
        n.prev = head;
        n.next = head.next;
        head.next.prev = n;
        head.next = n;
    }
    private void remove(Node n) {
        n.prev.next = n.next;
        n.next.prev = n.prev;
    }
    private void moveToHead(Node n) { remove(n); addToHead(n); }
}
```

**Complexity:** O(1) for both `get` and `put`. O(capacity) space.

**Common bugs:** forgetting to remove the evicted node from the *HashMap* (memory leak — the node lingers); using a singly-linked list (can't remove in O(1)); not handling capacity 0; mutating `node.val` but forgetting `moveToHead`.

> **Production analogue:** Redis with `maxmemory-policy allkeys-lru` (or `allkeys-lfu`) does exactly this internally — understanding the DLL+HashMap structure lets you reason about Redis's own eviction and about why `EXPIRE` + LRU is the standard cache layer in Smart360.

### Practice extension

- **LFU Cache (LC 460, Hard)** — evict *least frequently* used (tie-break by recency). Two HashMaps + a frequency-bucketed DLL. Mention it as the harder cousin; don't necessarily code it today.

---

## 💻 Practice coding questions

1. Reverse a linked list iteratively and recursively (LC 206).
2. Reverse a sublist between positions m and n in one pass (LC 92).
3. Merge two sorted lists with a dummy head (LC 21).
4. Merge k sorted lists using a `PriorityQueue<ListNode>` (LC 23).
5. Detect a cycle with Floyd's (LC 141) and find its entry point (LC 142).
6. Remove the nth node from the end in a single pass (LC 19).
7. Reorder list: `L0→Ln→L1→Ln-1→…` (LC 143).
8. Check if a linked list is a palindrome in O(1) space (LC 234).
9. Find the intersection node of two lists (LC 160) — two-pointer length equalization.
10. Implement LRU Cache with `LinkedHashMap` (LC 146).
11. Implement LRU Cache with HashMap + DLL, no `LinkedHashMap` (LC 146).
12. Add two numbers represented as reversed linked lists (LC 2) — carry handling.
13. Remove duplicates from a sorted list (LC 83) and from an unsorted one (extra-space variant).
14. Implement an `Idempotency-Key` middleware that caches POST responses in a map with TTL.
15. (Stretch) LFU Cache (LC 460).

---

## 🎤 Interview questions

1. **Why a dummy head?** Removes head-is-null/head-mutation special cases; the real head becomes just `dummy.next`. Return `dummy.next`.
2. **Reverse a list iteratively — walk me through the pointers.** prev/curr/next; save next, point curr.next→prev, advance. O(n)/O(1).
3. **Recursive reversal — what's the base case and the rewiring line?** Base: `head==null || head.next==null`. Rewire: `head.next.next = head; head.next = null;`.
4. **How does Floyd's cycle detection work and why must they meet?** Slow +1, fast +2; the gap shrinks by 1 per step, so inside a cycle they must coincide. No cycle → fast hits null.
5. **After detecting a cycle, how do you find its start?** Reset one pointer to head; advance both +1; they meet at the entry. Proof: `F = nC − a`.
6. **Remove nth from end in one pass — how?** Fixed-gap two-pointer: advance fast n steps, then both until `fast.next==null`; dummy head handles removing the head.
7. **Why a doubly-linked list for LRU and not singly?** O(1) removal needs the predecessor pointer; a singly-linked list would require an O(n) scan to find it.
8. **Why both a HashMap and a DLL?** HashMap = O(1) lookup by key; DLL = O(1) reorder + tail eviction. Neither alone gives O(1) for both operations.
9. **What do sentinel nodes buy you?** They eliminate all null/empty-list checks in add/remove; the list always has a head and tail.
10. **`LinkedHashMap` access-order — what does the constructor flag do?** `accessOrder=true` moves an entry to the end on `get`; override `removeEldestEntry` to auto-evict.
11. **Make LRU thread-safe — what changes?** Wrap operations in a lock (the get-then-move is a compound op, so `ConcurrentHashMap` alone isn't enough); or use a striped lock / `Caffeine`.
12. **REST: difference between 401 and 403?** 401 = not authenticated (bad/missing token); 403 = authenticated but not authorized.
13. **409 vs 422 — when each?** 409 = state conflict (stale version, duplicate key); 422 = well-formed but semantically invalid payload.
14. **Offset vs cursor pagination on a 10M-row table?** Offset is simple but O(offset) and inconsistent under inserts; cursor is O(log n) and stable — use cursor for feeds.
15. **DELETE /orders/123 called twice — 404 or 204?** Both defensible (idempotency = same final state); 204 gives a cleaner client experience.
16. **Make POST /transfers safe to retry.** `Idempotency-Key` header; store key→result in Redis with TTL; replay returns the cached result without re-charging. Claim the key atomically (`SET NX`) to avoid double-execution on concurrent retries.
17. **Which HTTP methods are idempotent and why does it matter?** GET/PUT/DELETE; matters because clients, proxies, and load balancers may safely retry them.
18. **How do you version an API without breaking clients?** Prefer additive changes; if breaking, run versions side-by-side and signal end-of-life with `Deprecation`/`Sunset` headers.
19. **Why is PATCH's idempotency debated?** Depends on semantics: `{set: x}` is idempotent, `{increment: x}` is not.
20. **Production: how does Redis implement LRU and how does it relate to your hand-rolled cache?** `maxmemory-policy allkeys-lru` approximates LRU via sampling; the DLL+HashMap model is the exact form Redis approximates for performance.

---

## ✅ Self-check

1. Reverse `1→2→3→4→null` in your head → `4→3→2→1→null`. Draw the prev/curr/next diagram at each step without looking.
2. State the trade-offs of cursor vs offset pagination on a 10M-row table (depth cost + consistency under inserts).
3. Implement the full HashMap+DLL LRU cache from a blank file in ≤30 min, compilable on the first try.
4. Explain in one sentence why an `Idempotency-Key` must be claimed before the operation runs.

---

*Nav: ← [Day 3 (Wed Jul 08)](03-wed-jul-08.md) · [Week 2](README.md) · [Day 5 (Fri Jul 10)](05-fri-jul-10.md) →*
