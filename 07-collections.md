# Java Collections Framework

## Quick Reference

> "A unified architecture of interfaces, implementations, and algorithms used to store, manipulate, and process groups of objects efficiently."

---

## Three Things Often Confused

### 1. Collection API (the entire framework)

The full framework — every interface, every class, every algorithm including `Map`.

```
Collection API (framework)
├── java.util.Collection (interface)
│   ├── java.util.List
│   ├── java.util.Set
│   └── java.util.Queue
└── java.util.Map  ← separate root — does NOT extend Collection
```

### 2. `Collection` Interface

Root interface for List, Set, Queue. **Does NOT include Map.**

```java
Collection<String> data = new ArrayList<>();
data.add("Java");
data.add("Spring");
data.size();     // 2
data.isEmpty();  // false
data.contains("Java"); // true
data.remove("Spring");
```

### 3. `Collections` Class (utility)

Utility class with **static helper methods** for working with collections.

```java
List<Integer> nums = new ArrayList<>(Arrays.asList(5, 2, 8, 1));

Collections.sort(nums);                              // ascending  → [1, 2, 5, 8]
Collections.sort(nums, Comparator.reverseOrder());   // descending → [8, 5, 2, 1]
Collections.reverse(nums);                           // reverse in-place
Collections.shuffle(nums);                           // random order
Collections.max(nums);                               // 8
Collections.min(nums);                               // 1
Collections.frequency(nums, 5);                      // count of 5
Collections.unmodifiableList(nums);                  // read-only wrapper
Collections.synchronizedList(nums);                  // thread-safe wrapper
```

---

## List Interface — Ordered, Allows Duplicates

**Key characteristics:**
- Maintains insertion order
- Allows duplicate elements
- Index-based access (`get(i)`, `set(i, val)`)

---

### ArrayList

1. Backed by a **resizable array** — grows by 50% when capacity is exceeded
2. **Best for random access** — `get(i)` is O(1); backed array allows direct index lookup
3. **Slow for mid-list insert/delete** — shifting elements is O(n)
4. Not thread-safe — use `CopyOnWriteArrayList` or `Collections.synchronizedList()` for concurrent use

```java
List<String> users = new ArrayList<>();
users.add("Alice");
users.add("Bob");
users.add(0, "Admin");           // insert at index — shifts others
System.out.println(users.get(1)); // Bob
users.remove("Alice");
System.out.println(users.size()); // 2
```

**When to use:** API response lists, in-memory DTOs, read-heavy collections, Spring repository return values.

---

### LinkedList

1. Backed by a **doubly linked list** — each node holds value + prev/next pointers
2. **Fast for insert/delete at head or tail** — O(1); no shifting needed
3. **Slow for random access** — `get(i)` is O(n); must traverse from head
4. Implements both `List` and `Deque` — can be used as a queue, stack, or deque

```java
List<String> list = new LinkedList<>();
list.add("A");
list.add("B");
list.add(0, "Z"); // fast — no shifting, just pointer rewire

// Also usable as a Queue/Deque
Deque<String> deque = new LinkedList<>();
deque.addFirst("first");
deque.addLast("last");
deque.pollFirst(); // "first"
```

**When to use:** Task queues, event pipelines, breadth-first search — any scenario with heavy front/back insertions.

---

### Vector (Legacy)

1. **Synchronized `ArrayList`** — every method is `synchronized`; only one thread can access at a time
2. Grows by **100%** (doubles) vs ArrayList's 50% — more memory waste
3. **Slower than ArrayList** in single-threaded code due to lock overhead
4. Legacy class from Java 1.0 — **prefer `CopyOnWriteArrayList` or `synchronizedList()` in modern code**

```java
Vector<Integer> v = new Vector<>();
v.add(10);
v.add(20);
System.out.println(v.size()); // 2
```

**When to use:** Avoid. Exists for backward compatibility.

---

### Stack (extends Vector)

1. **LIFO (Last-In, First-Out)** data structure extending `Vector`
2. Core methods: `push(e)`, `pop()`, `peek()`, `empty()`, `search(o)`
3. Inherits all `Vector` methods — can be used as a list (breaks the stack contract)
4. **Legacy class** — prefer `ArrayDeque` as a stack in modern code (faster, cleaner API)

```java
Stack<Integer> stack = new Stack<>();
stack.push(10);
stack.push(20);
stack.push(30);
System.out.println(stack.peek()); // 30 — view without removing
System.out.println(stack.pop());  // 30 — remove and return
System.out.println(stack.search(10)); // 2 — 1-based position from top
```

**When to use:** Avoid. Use `ArrayDeque` instead.

---

### CopyOnWriteArrayList

1. **Thread-safe List** — every write (add/remove/set) creates a **fresh copy** of the underlying array
2. **Reads are lock-free and very fast** — readers always see a consistent snapshot
3. **Writes are expensive** — full array copy on every mutation; not suitable for write-heavy use
4. Iterators reflect the snapshot at time of creation — never throw `ConcurrentModificationException`

```java
import java.util.concurrent.*;
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
list.add("A");
list.add("B");

// Safe to iterate while another thread modifies
for (String s : list) {
    System.out.println(s); // no ConcurrentModificationException
}
```

**When to use:** Event listener lists, read-heavy config lists, Spring Security filter chains.

---

## Set Interface — No Duplicates

**Key characteristics:**
- No duplicate elements (`equals()` + `hashCode()` determine equality)
- No index-based access
- Ordering depends on implementation

---

### HashSet

1. Backed by a **HashMap internally** — elements are stored as keys with a dummy value
2. **No guaranteed order** — elements are distributed by hash bucket
3. **O(1) average for add/remove/contains** — depends on good `hashCode()` distribution
4. Allows one `null` element

```java
Set<String> roles = new HashSet<>();
roles.add("ADMIN");
roles.add("USER");
roles.add("ADMIN"); // ignored — duplicate
System.out.println(roles.size()); // 2
System.out.println(roles.contains("USER")); // true
```

**When to use:** Deduplication, membership testing, tag/role sets where order doesn't matter.

---

### LinkedHashSet

1. **HashSet + doubly linked list** — maintains insertion order on top of hash performance
2. **O(1) for add/remove/contains** — same as HashSet
3. Slightly more memory than HashSet due to the linked list pointers
4. Allows one `null` element; iterates in insertion order

```java
Set<String> set = new LinkedHashSet<>();
set.add("C");
set.add("A");
set.add("B");
set.add("A"); // ignored
System.out.println(set); // [C, A, B] — insertion order preserved
```

**When to use:** When you need a deduplicating set but insertion order matters (e.g., processing order of unique user IDs).

---

### TreeSet

1. Backed by a **Red-Black Tree** — self-balancing BST
2. **Always sorted** — natural ordering (Comparable) or custom Comparator
3. **O(log n) for add/remove/contains** — tree traversal cost
4. Does NOT allow `null` (throws `NullPointerException` on add)

```java
Set<Integer> sorted = new TreeSet<>();
sorted.add(5);
sorted.add(1);
sorted.add(3);
System.out.println(sorted); // [1, 3, 5] — always sorted

// Extra NavigableSet methods
TreeSet<Integer> ts = new TreeSet<>(sorted);
System.out.println(ts.first());      // 1
System.out.println(ts.last());       // 5
System.out.println(ts.floor(4));     // 3 — largest element <= 4
System.out.println(ts.ceiling(2));   // 3 — smallest element >= 2
```

**When to use:** Sorted unique data — leaderboard scores, sorted category lists, range queries.

---

### EnumSet

1. **Specialized for enum types only** — uses bit vector internally, extremely compact
2. **Fastest Set for enums** — all operations are O(1) and use bit manipulation
3. Null elements not allowed
4. Maintains natural order of enum constants (declaration order)

```java
enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }

EnumSet<Day> weekdays = EnumSet.range(Day.MON, Day.FRI);
EnumSet<Day> weekend  = EnumSet.of(Day.SAT, Day.SUN);
EnumSet<Day> all      = EnumSet.allOf(Day.class);
System.out.println(weekdays); // [MON, TUE, WED, THU, FRI]
```

**When to use:** Flag sets, permission sets (`EnumSet<Permission>`), day/status filters.

---

### CopyOnWriteArraySet

1. Backed by a **CopyOnWriteArrayList** — not a hash table; uses linear scan for membership
2. Thread-safe with lock-free reads — writes copy the entire array
3. **O(n) for add/contains** — slower than HashSet for large sets
4. Iterators are snapshot-based — never throw `ConcurrentModificationException`

```java
import java.util.concurrent.*;
CopyOnWriteArraySet<String> set = new CopyOnWriteArraySet<>();
set.add("listener1");
set.add("listener2");
set.add("listener1"); // ignored — no duplicates
```

**When to use:** Small, rarely-modified thread-safe sets — event listeners, subscription registries.

---

### ConcurrentSkipListSet

1. Thread-safe, **always sorted** — backed by `ConcurrentSkipListMap`
2. **O(log n) expected** for all operations — skip list allows concurrent access at multiple levels
3. Lock-free reads — multiple threads can traverse simultaneously
4. Does NOT allow `null`

```java
import java.util.concurrent.*;
ConcurrentSkipListSet<Integer> set = new ConcurrentSkipListSet<>();
set.add(5);
set.add(1);
set.add(3);
System.out.println(set); // [1, 3, 5] — sorted + thread-safe
```

**When to use:** Concurrent sorted sets — leaderboards, sorted concurrent task queues.

---

## Queue & Deque Interface — FIFO / Priority

**Key characteristics:**
- Queue: FIFO (First-In, First-Out)
- Deque (Double Ended Queue): add/remove from both ends
- `offer()`/`poll()`/`peek()` are preferred over `add()`/`remove()`/`element()` (no exceptions on empty)

---

### PriorityQueue

1. **Min-heap by default** — `poll()` always returns the smallest element (natural order)
2. Custom ordering via `Comparator` (pass at construction)
3. **O(log n)** for insert and remove; **O(1)** for peek
4. Not thread-safe; does NOT allow `null`

```java
Queue<Integer> pq = new PriorityQueue<>();
pq.add(10);
pq.add(5);
pq.add(20);
System.out.println(pq.poll()); // 5 — min first
System.out.println(pq.poll()); // 10
System.out.println(pq.poll()); // 20

// Custom: max-heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.add(10);
maxHeap.add(5);
maxHeap.add(20);
System.out.println(maxHeap.poll()); // 20
```

**When to use:** Task scheduling by priority, Dijkstra's shortest path, top-K problems.

---

### ArrayDeque

1. **Resizable array-backed double-ended queue** — add/remove at both head and tail efficiently
2. **Faster than `LinkedList`** as a stack or queue — no node allocation overhead, better cache locality
3. **Preferred replacement for both `Stack` and `LinkedList`** in modern Java
4. Does NOT allow `null`; not thread-safe

```java
Deque<String> dq = new ArrayDeque<>();
dq.addFirst("first");
dq.addLast("last");
dq.offerFirst("new-first");

System.out.println(dq.peekFirst()); // "new-first"
System.out.println(dq.peekLast());  // "last"
dq.pollFirst(); // removes "new-first"
dq.pollLast();  // removes "last"

// As a stack (LIFO)
dq.push("A");  // = addFirst
dq.push("B");
System.out.println(dq.pop()); // "B" — LIFO
```

**When to use:** BFS/DFS algorithms, undo/redo stacks, sliding window problems, task scheduling.

---

### LinkedList as Queue / Deque

`LinkedList` implements both `List` and `Deque` — can be used as a FIFO queue or double-ended queue.

```java
Queue<String> queue = new LinkedList<>();
queue.offer("first");
queue.offer("second");
System.out.println(queue.poll()); // "first" — FIFO

Deque<String> deque = new LinkedList<>();
deque.addFirst("A");
deque.addLast("B");
```

**When to use:** When you need a combined List + Queue/Deque — prefer `ArrayDeque` if only Deque behaviour is needed.

---

### BlockingQueue Variants (Concurrency)

All `BlockingQueue` implementations block the calling thread when the queue is full (on `put()`) or empty (on `take()`). Designed for producer-consumer patterns.

| Implementation | Capacity | Key Characteristic |
|---|---|---|
| `LinkedBlockingQueue` | Optionally bounded (default: `Integer.MAX_VALUE`) | Linked nodes; separate locks for head and tail — better throughput |
| `ArrayBlockingQueue` | Fixed (must specify at creation) | Array-backed; single lock; bounded, predictable memory |
| `PriorityBlockingQueue` | Unbounded | Priority-ordered blocking queue |
| `DelayQueue` | Unbounded | Elements available only after delay expires |
| `SynchronousQueue` | Zero capacity | Each `put()` must wait for a `take()` — direct handoff |
| `LinkedBlockingDeque` | Optionally bounded | Double-ended blocking deque |

```java
import java.util.concurrent.*;

// Producer-consumer with ArrayBlockingQueue
BlockingQueue<String> queue = new ArrayBlockingQueue<>(100);

// Producer thread
queue.put("task");      // blocks if full

// Consumer thread
String task = queue.take(); // blocks if empty

// Non-blocking variants
queue.offer("task", 1, TimeUnit.SECONDS); // waits up to 1s
queue.poll(1, TimeUnit.SECONDS);          // waits up to 1s
```

**When to use:** `LinkedBlockingQueue` — ThreadPoolExecutor task queue. `ArrayBlockingQueue` — bounded producer-consumer. `SynchronousQueue` — `newCachedThreadPool()` internal queue.

---

## Map Interface — Key-Value Pairs

**Key characteristics:**
- Stores key-value pairs; keys must be unique
- `Map` does NOT extend `Collection` — separate hierarchy
- Common operations: `put(k,v)`, `get(k)`, `remove(k)`, `containsKey(k)`, `entrySet()`, `keySet()`

---

### HashMap

1. Backed by a **hash table** — array of buckets; key's `hashCode()` determines bucket
2. **O(1) average** for get/put/remove — degrades to O(n) if many hash collisions
3. Java 8+: a bucket with >8 entries converts to a **Red-Black Tree** (O(log n) worst case) — **but only if total table capacity ≥ 64** (`MIN_TREEIFY_CAPACITY`); below that the map just resizes instead of treeifying
4. **Allows one `null` key and multiple `null` values**; not thread-safe; no guaranteed order

```java
Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 95);
scores.put("Bob", 87);
scores.put(null, 0);           // null key allowed
scores.get("Alice");           // 95
scores.getOrDefault("Eve", 0); // 0 — safe default
scores.putIfAbsent("Bob", 100); // does nothing — already exists

for (Map.Entry<String, Integer> e : scores.entrySet()) {
    System.out.println(e.getKey() + " → " + e.getValue());
}
```

**When to use:** In-memory caching, response maps, counting frequency, lookup tables in Spring services.

---

### LinkedHashMap

1. **HashMap + doubly linked list** — maintains insertion order (or access order if constructed with `accessOrder=true`)
2. **O(1) for get/put** — same as HashMap
3. Slightly more memory than HashMap due to linked list pointers
4. Access-order mode enables **LRU cache** implementation

```java
Map<String, Integer> map = new LinkedHashMap<>();
map.put("C", 3);
map.put("A", 1);
map.put("B", 2);
System.out.println(map); // {C=3, A=1, B=2} — insertion order preserved

// LRU Cache with access-order
Map<Integer, String> lruCache = new LinkedHashMap<>(16, 0.75f, true) {
    protected boolean removeEldestEntry(Map.Entry<Integer, String> e) {
        return size() > 100; // evict when > 100 entries
    }
};
```

**When to use:** Ordered configuration maps, LRU cache, audit logs where insertion order matters.

---

### TreeMap

1. Backed by a **Red-Black Tree** — self-balancing BST sorted by keys
2. **O(log n)** for get/put/remove — tree traversal
3. Provides `NavigableMap` methods: `firstKey()`, `lastKey()`, `floorKey(k)`, `ceilingKey(k)`, `subMap(from, to)`
4. Does NOT allow `null` keys (throws NPE); allows `null` values

```java
Map<String, Integer> map = new TreeMap<>();
map.put("banana", 2);
map.put("apple", 5);
map.put("cherry", 1);
System.out.println(map); // {apple=5, banana=2, cherry=1} — sorted by key

TreeMap<String, Integer> tm = new TreeMap<>(map);
System.out.println(tm.firstKey());          // "apple"
System.out.println(tm.floorKey("b"));       // "banana"
System.out.println(tm.subMap("a", "c"));    // {apple=5, banana=2}
```

**When to use:** Sorted lookups, range queries, building sorted indexes, leaderboards by name.

---

### Hashtable (Legacy)

1. Legacy map from Java 1.0 — **synchronized** on every method; one thread at a time
2. Does NOT allow `null` keys or `null` values (throws `NullPointerException`)
3. **Slower than HashMap** in single-threaded code; worse than `ConcurrentHashMap` in multi-threaded
4. **Avoid in modern code** — use `ConcurrentHashMap` for thread safety or `HashMap` otherwise

```java
Hashtable<String, Integer> table = new Hashtable<>();
table.put("key", 1);
// table.put(null, 1); // NullPointerException
```

**When to use:** Avoid. Exists for backward compatibility.

---

### ConcurrentHashMap

1. **Thread-safe HashMap** — uses **bucket-level (segment) locking** in Java 8+ (CAS + synchronized on bin)
2. **Multiple threads can read/write different buckets simultaneously** — much higher throughput than `Hashtable`
3. Does NOT allow `null` keys or `null` values
4. Atomic operations: `putIfAbsent()`, `computeIfAbsent()`, `merge()`, `compute()`

```java
import java.util.concurrent.*;
ConcurrentHashMap<String, Integer> cache = new ConcurrentHashMap<>();

// Thread-safe increment
cache.merge("count", 1, Integer::sum);

// Thread-safe compute
cache.computeIfAbsent("user:123", k -> loadFromDB(k));

// putIfAbsent — returns existing value if key exists
String existing = cache.putIfAbsent("token", "abc123");
```

**Spring Boot use cases:**
- In-memory rate limiter: `ConcurrentHashMap<String, AtomicInteger>` keyed by IP
- Request deduplication: store recent request IDs with TTL
- In-process caches before introducing Redis

---

### WeakHashMap

1. Keys are held as **weak references** — key objects are eligible for GC even if they're in the map
2. When the key is garbage collected, the entry is automatically removed
3. Useful for **memory-sensitive caches** — entries disappear when keys have no other strong references
4. Not thread-safe; iteration is unpredictable (entries may disappear)

```java
Map<Object, String> cache = new WeakHashMap<>();
Object key = new Object();
cache.put(key, "data");
System.out.println(cache.size()); // 1

key = null; // remove strong reference
System.gc();
// After GC: entry may be gone
System.out.println(cache.size()); // likely 0
```

**When to use:** Metadata caches keyed by live objects (e.g., class → metadata), memory-sensitive caches.

---

### IdentityHashMap

1. Uses **`==` (reference equality)** instead of `.equals()` for key comparison — two keys are equal only if they are the exact same object
2. Uses `System.identityHashCode()` instead of `hashCode()`
3. Intentionally violates the `Map` contract — niche use cases only
4. Not thread-safe

```java
Map<String, String> map = new IdentityHashMap<>();
String a = new String("key");
String b = new String("key");

map.put(a, "value-a");
map.put(b, "value-b");
System.out.println(map.size()); // 2 — a != b by reference
```

**When to use:** Object graph traversal (e.g., serialization — track visited objects by identity), proxy frameworks.

---

### EnumMap

1. **Specialized Map for enum keys** — backed by an array indexed by enum ordinal
2. **Extremely fast and memory-efficient** — all operations O(1), no hashing
3. Keys must be from a single enum type; null keys not allowed; null values allowed
4. Maintains **natural order** of enum constants (declaration order)

```java
enum Status { PENDING, ACTIVE, CLOSED }
Map<Status, String> labels = new EnumMap<>(Status.class);
labels.put(Status.PENDING, "Awaiting Approval");
labels.put(Status.ACTIVE, "Live");
labels.put(Status.CLOSED, "Archived");
System.out.println(labels); // {PENDING=Awaiting Approval, ACTIVE=Live, CLOSED=Archived}
```

**When to use:** Mapping enum values to labels, configs, handlers — e.g., `EnumMap<OrderStatus, NotificationHandler>`.

---

### ConcurrentSkipListMap

1. **Thread-safe, always sorted** Map — backed by a skip list (probabilistic data structure)
2. **O(log n) expected** for get/put/remove — concurrent without global lock
3. Supports `NavigableMap` methods (`firstKey`, `lastKey`, `subMap`, etc.) concurrently
4. Does NOT allow `null` keys or values

```java
import java.util.concurrent.*;
ConcurrentSkipListMap<Integer, String> map = new ConcurrentSkipListMap<>();
map.put(3, "three");
map.put(1, "one");
map.put(2, "two");
System.out.println(map);              // {1=one, 2=two, 3=three}
System.out.println(map.firstKey());   // 1
System.out.println(map.subMap(1, 3)); // {1=one, 2=two}
```

**When to use:** Concurrent sorted indexes, leaderboards with concurrent updates.

---

### Properties

1. Extends `Hashtable<Object, Object>` — specialized for **string key-value configuration**
2. Supports reading/writing `.properties` files (`load()`, `store()`)
3. Can chain a **default/fallback** `Properties` object
4. Keys and values must be `String` (by convention)

```java
Properties props = new Properties();
props.setProperty("db.url", "jdbc:postgresql://localhost:5432/mydb");
props.setProperty("db.user", "admin");

// Read from file
try (InputStream in = new FileInputStream("config.properties")) {
    props.load(in);
}

String url = props.getProperty("db.url");
String missing = props.getProperty("missing.key", "default-value");
```

**Spring Boot note:** `application.properties` / `application.yml` is read via Spring's `Environment` abstraction — you rarely use `Properties` directly. Use `@Value("${property}")` or `@ConfigurationProperties`.

---

## Iterator & Traversal

### Iterator

- Provides `hasNext()` and `next()` for forward traversal
- `remove()` — safely removes current element during iteration (only safe removal method)
- All `Collection` types provide `iterator()`

```java
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String s = it.next();
    if (s.equals("B")) it.remove(); // safe mid-iteration removal
}
System.out.println(list); // [A, C]
```

### ListIterator

- Bidirectional traversal — `hasPrevious()` / `previous()` as well as forward
- `add()`, `set()` during iteration — can modify the list
- Only available on `List` implementations

```java
List<Integer> nums = new ArrayList<>(Arrays.asList(1, 2, 3));
ListIterator<Integer> it = nums.listIterator(nums.size()); // start from end
while (it.hasPrevious()) {
    System.out.print(it.previous() + " "); // 3 2 1
}
```

### Spliterator

- Introduced in Java 8 for **parallel stream processing** — can split the source into parts
- Used internally by `Stream.parallel()` and `forEach()`
- `trySplit()` — splits for parallel processing; `forEachRemaining()` — processes remainder sequentially

```java
List<Integer> list = List.of(1, 2, 3, 4, 5, 6);
Spliterator<Integer> sp = list.spliterator();
Spliterator<Integer> sp2 = sp.trySplit(); // sp2 gets first half

sp2.forEachRemaining(System.out::println); // 1 2 3
sp.forEachRemaining(System.out::println);  // 4 5 6
```

---

## Java 8+ Streams & flatMap

### Stream Basics

```java
List<String> names = List.of("Alice", "Bob", "Charlie", "Dave");

List<String> result = names.stream()
    .filter(n -> n.length() > 3)           // keep names longer than 3 chars
    .map(String::toUpperCase)              // transform
    .sorted()                              // sort
    .collect(Collectors.toList());         // collect

System.out.println(result); // [ALICE, CHARLIE, DAVE]
```

### map() vs flatMap()

| | `map()` | `flatMap()` |
|---|---|---|
| Input | `Stream<T>` | `Stream<Stream<T>>` (nested) |
| Output | `Stream<R>` — one output per input | `Stream<R>` — flattened, 1:N |
| Use case | Transformation | Flatten nested collections |

```java
// map — 1:1 transformation
List<String> upper = List.of("a", "b").stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList()); // [A, B]

// flatMap — flatten nested lists
List<List<Integer>> nested = List.of(
    List.of(1, 2),
    List.of(3, 4)
);
List<Integer> flat = nested.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList()); // [1, 2, 3, 4]

// flatMap — split sentences into words
List<String> sentences = List.of("hello world", "foo bar");
List<String> words = sentences.stream()
    .flatMap(s -> Arrays.stream(s.split(" ")))
    .collect(Collectors.toList()); // [hello, world, foo, bar]
```

**Spring Boot use:** `flatMap` is common when processing lists of lists from DB queries, combining results from multiple services, or flattening nested JSON structures.

---

## Thread-Safe Collections

```java
// Option 1: synchronized wrapper (Collections utility)
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());
// Note: iteration must be externally synchronized

synchronized (syncList) {
    for (String s : syncList) { /* safe iteration */ }
}

// Option 2: java.util.concurrent classes (preferred)
Map<String, Integer>  concurrentMap  = new ConcurrentHashMap<>();
List<String>          cowList        = new CopyOnWriteArrayList<>();
Queue<String>         blockingQ      = new LinkedBlockingQueue<>();
Deque<String>         concurrentDeq  = new ConcurrentLinkedDeque<>();
Set<String>           concurrentSet  = new ConcurrentSkipListSet<>();
```

---

## Key Interview Comparisons

---

### ArrayList vs LinkedList

| | ArrayList | LinkedList |
|---|---|---|
| Internal structure | Resizable array | Doubly linked list |
| Random access `get(i)` | O(1) | O(n) |
| Insert/delete at middle | O(n) — shifting | O(1) — pointer rewire |
| Insert/delete at head/tail | O(n) for head; O(1) amortized for tail | O(1) both ends |
| Memory | Compact — just data | Extra prev/next pointers per node |
| Thread safety | No | No |
| Use | Read-heavy, random access | Insert/delete heavy, queue/deque use |

---

### HashSet vs LinkedHashSet vs TreeSet

| | HashSet | LinkedHashSet | TreeSet |
|---|---|---|---|
| Order | None (arbitrary) | Insertion order | Sorted (natural/Comparator) |
| Backed by | HashMap | LinkedHashMap | Red-Black Tree |
| get/add/remove | O(1) avg | O(1) avg | O(log n) |
| Null allowed | Yes (one) | Yes (one) | No |
| Use | Fast dedup, membership | Dedup + order | Sorted unique elements |

---

### HashMap vs LinkedHashMap vs TreeMap

| | HashMap | LinkedHashMap | TreeMap |
|---|---|---|---|
| Order | None | Insertion order (or access order) | Sorted by key |
| Backed by | Hash table | Hash table + linked list | Red-Black Tree |
| get/put | O(1) avg | O(1) avg | O(log n) |
| Null key | Yes (one) | Yes (one) | No |
| Use | General lookup | Ordered map, LRU cache | Sorted key-value, range queries |

---

### HashMap vs Hashtable vs ConcurrentHashMap

| | HashMap | Hashtable | ConcurrentHashMap |
|---|---|---|---|
| Thread safety | No | Yes — full lock | Yes — bucket-level lock |
| Null key/value | Yes / Yes | No / No | No / No |
| Performance (multi-thread) | Unsafe | Low (full lock) | High (segment lock) |
| Performance (single-thread) | Fast | Slower (lock overhead) | Fast |
| Legacy | No | Yes (Java 1.0) | No (Java 5+) |
| Use | Single-thread | Avoid | Concurrent apps |

---

### ArrayList vs Vector

| | ArrayList | Vector |
|---|---|---|
| Thread safety | No | Yes — every method synchronized |
| Growth | 50% | 100% (doubles) |
| Performance | Fast | Slower (lock overhead) |
| Legacy | No | Yes (Java 1.0) |
| Use | Prefer | Avoid |

---

### Stack vs ArrayDeque (as Stack)

| | Stack | ArrayDeque |
|---|---|---|
| Extends | Vector (synchronized) | AbstractCollection |
| Thread safety | Yes (inherited) | No |
| Performance | Slower (sync) | Faster |
| Null allowed | Yes | No |
| Extra methods | List methods (breaks stack contract) | Only deque methods |
| Use | Avoid | Prefer |

---

### synchronizedList vs CopyOnWriteArrayList

| | `Collections.synchronizedList()` | `CopyOnWriteArrayList` |
|---|---|---|
| Mechanism | Wraps each method with `synchronized` | Creates full array copy on every write |
| Read performance | Blocks on lock | Lock-free — very fast |
| Write performance | Fast (just lock) | Slow (array copy) |
| Iteration safety | Must externally synchronize | Always safe — snapshot iterator |
| `ConcurrentModificationException` | Possible without sync block | Never |
| Use | Write-balanced workloads | Read-heavy, rarely-modified lists |

---

### synchronizedMap vs ConcurrentHashMap

| | `Collections.synchronizedMap()` | `ConcurrentHashMap` |
|---|---|---|
| Lock granularity | Entire map locked on every operation | Bucket-level (CAS + bin lock) |
| Throughput | Low — serializes all access | High — parallel on different buckets |
| Null key/value | Depends on wrapped map | Not allowed |
| Iteration | Must externally synchronize | Weakly consistent (no external sync needed) |
| Atomic operations | No | Yes — `putIfAbsent`, `computeIfAbsent`, `merge` |
| Use | Rarely | Always prefer for concurrent use |

---

### Iterator vs ListIterator vs Spliterator

| | Iterator | ListIterator | Spliterator |
|---|---|---|---|
| Direction | Forward only | Forward + backward | Forward (split for parallel) |
| Available on | All Collections | List only | All Collections + arrays |
| Modification | `remove()` only | `add()`, `set()`, `remove()` | None |
| Parallel support | No | No | Yes — `trySplit()` |
| Java version | Java 1.2 | Java 1.2 | Java 8 |
| Use | General traversal | Bidirectional List traversal | Parallel streams |

---

### map() vs flatMap() (Streams)

| | `map()` | `flatMap()` |
|---|---|---|
| Mapping ratio | 1:1 | 1:N |
| Output type | `Stream<R>` | `Stream<R>` (flattened) |
| Input type | `T → R` | `T → Stream<R>` |
| Use | Transform each element | Flatten nested collections/optionals |
| Example | `.map(String::toUpperCase)` | `.flatMap(s -> Arrays.stream(s.split(" ")))` |

---

### PriorityQueue vs TreeSet

| | PriorityQueue | TreeSet |
|---|---|---|
| Duplicates | Allowed | Not allowed |
| Access | Only head (`peek`/`poll`) | Any element by value |
| Internal | Min-heap (array) | Red-Black Tree |
| get/add | O(log n) | O(log n) |
| Iteration order | NOT sorted — heap order | Sorted |
| Thread safety | No | No |
| Use | Priority-based processing | Sorted unique collection with range ops |

---

### List vs Set vs Queue vs Map

| | List | Set | Queue | Map |
|---|---|---|---|---|
| Duplicates | Yes | No | Yes | No (keys) |
| Order | Insertion order | Depends | FIFO / priority | Depends |
| Index access | Yes | No | No | By key |
| Extends `Collection` | Yes | Yes | Yes | No |
| Null elements | Yes (most) | One null (most) | No (most) | One null key (HashMap) |
| Use | Ordered items | Unique items | Processing pipeline | Key-value lookup |

---

## Interview Q&A

**Q: What is the Collection API vs `Collection` interface vs `Collections` class?**
> - **Collection API**: the entire framework — all interfaces (List, Set, Queue, Map), all implementations, all algorithms
> - **`Collection` interface**: root interface for List/Set/Queue; does NOT include Map
> - **`Collections` class**: utility class with static methods — `sort()`, `reverse()`, `synchronizedList()`, `unmodifiableList()`, etc.

**Q: Why doesn't `Map` extend `Collection`?**
> `Collection` represents a group of single elements. `Map` stores key-value pairs — it has a fundamentally different contract. Including it under `Collection` would force every `Collection` method to make sense for key-value pairs (e.g., what does `add(element)` mean for a map?). Joshua Bloch (who designed the Collections framework) intentionally kept `Map` as a separate hierarchy.

**Q: How does `HashMap` work internally?**
> An array of "buckets" (linked lists/trees). On `put(k, v)`: compute `hashCode(k)`, apply bit-spread to get bucket index, walk the bucket for an existing key (using `equals()`), insert or update. Java 8+: when a bucket has >8 entries, it converts to a Red-Black Tree (O(log n) worst case) — but only if table capacity is already ≥ 64 (`MIN_TREEIFY_CAPACITY`); if capacity is smaller the map resizes instead of treeifying. Load factor 0.75 by default — map resizes (doubles) when 75% full.

**Q: Why is `ConcurrentHashMap` better than `Hashtable` or `synchronizedMap`?**
> `Hashtable` and `synchronizedMap` lock the **entire map** on every operation — only one thread can read or write at a time. `ConcurrentHashMap` uses bucket-level CAS + synchronized only on individual bins — multiple threads can operate on different buckets simultaneously, giving much higher throughput under concurrent load.

**Q: When would you use `TreeMap` over `HashMap`?**
> When you need keys in sorted order or need range queries (`subMap`, `headMap`, `tailMap`, `floorKey`, `ceilingKey`). Example: time-series data stored by timestamp, sorted leaderboard by player name, ordered config keys.

**Q: What is the difference between `fail-fast` and `fail-safe` iterators?**
> - **Fail-fast** (most `java.util` collections): throws `ConcurrentModificationException` if the collection is modified during iteration (detected via `modCount`). Examples: `ArrayList`, `HashMap`
> - **Fail-safe** (`java.util.concurrent` collections): iterates over a snapshot or uses weakly consistent traversal — never throws `ConcurrentModificationException`. Examples: `CopyOnWriteArrayList`, `ConcurrentHashMap`

**Q: What happens if two keys have the same `hashCode()` in a `HashMap`?**
> They land in the same bucket — a **hash collision**. The bucket stores them as a linked list (Java 7) or tree (Java 8+ when bucket size > 8 **and** table capacity ≥ 64 — otherwise the map resizes instead of treeifying). `equals()` is used to distinguish keys within the bucket. Performance degrades from O(1) to O(n) for lists or O(log n) for trees. Lesson: always implement a good `hashCode()` with low collision probability.

**Q: How do you iterate a `Map` safely?**
> ```java
> // Option 1: entrySet() — most efficient (single iteration)
> for (Map.Entry<String, Integer> e : map.entrySet()) {
>     System.out.println(e.getKey() + ": " + e.getValue());
> }
> // Option 2: forEach (Java 8)
> map.forEach((k, v) -> System.out.println(k + ": " + v));
> // Option 3: keySet() — less efficient (two lookups)
> for (String key : map.keySet()) {
>     System.out.println(key + ": " + map.get(key));
> }
> ```

**Q: Why is `ArrayList` generally preferred over `LinkedList`?**
> Modern CPU cache lines benefit `ArrayList` — its contiguous array layout means multiple elements are loaded into cache together. `LinkedList` nodes are scattered in memory, causing cache misses. For most real-world use cases (reads + occasional adds to tail), `ArrayList` is faster even for inserts because cache effects dominate over pointer chasing.

**Q: What is `computeIfAbsent()` in `ConcurrentHashMap` and why is it useful?**
> Atomically computes and inserts a value for a key if it's absent — in one atomic step. Prevents race conditions where two threads both see `null` and both try to compute+insert.
> ```java
> // Thread-safe lazy initialization in a shared cache
> ConcurrentHashMap<String, List<String>> cache = new ConcurrentHashMap<>();
> cache.computeIfAbsent("user:1", k -> loadPermissionsFromDB(k));
> ```

**Q: How would you implement a simple LRU cache in Java?**
> ```java
> int capacity = 100;
> Map<Integer, String> lru = new LinkedHashMap<>(capacity, 0.75f, true) {
>     protected boolean removeEldestEntry(Map.Entry<Integer, String> e) {
>         return size() > capacity;
>     }
> };
> ```
> `LinkedHashMap` in access-order mode moves recently accessed entries to the tail. `removeEldestEntry()` evicts the head (least recently accessed) when capacity is exceeded. Thread-safety requires wrapping with `Collections.synchronizedMap()` or using Caffeine/Guava cache in production.

**Q: Where do you use Collections in Spring Boot projects?**
> - `List<UserDTO>` — service methods returning lists of entities
> - `Map<String, Object>` — dynamic response bodies, JSON payloads
> - `ConcurrentHashMap` — in-memory rate limiters, request dedup caches
> - `LinkedHashMap` — ordered config maps, response ordering
> - `EnumMap<OrderStatus, Handler>` — dispatching logic by status
> - `CopyOnWriteArrayList` — event listeners, Spring Security filter lists
> - `ArrayDeque` — local processing stacks in algorithms
> - `PriorityQueue` — priority-based task scheduling, retry queues

**Q: What is `Collections.unmodifiableList()` vs `List.of()`?**
> Both return read-only views, but:
> - `Collections.unmodifiableList(list)`: wraps an existing mutable list — if the original list is modified, the unmodifiable view reflects those changes; `UnsupportedOperationException` on direct modification
> - `List.of(...)` (Java 9+): truly immutable — cannot be modified, does not allow `null` elements, not backed by a mutable list
> Prefer `List.of()` / `List.copyOf()` when you want genuine immutability.
