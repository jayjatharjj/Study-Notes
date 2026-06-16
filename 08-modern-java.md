# Modern Java (16–21) & the Spring Boot 3 Ecosystem

## Quick Reference

> "The language and platform that interviews assume in 2024+: **records** for data carriers, **sealed types** for closed domain models, **switch expressions + pattern matching** for exhaustive, type-safe branching, **text blocks** for embedded JSON/SQL, and **virtual threads** that make thread-per-request scale again — all running on the **JDK 17 baseline** with the **`javax.*` → `jakarta.*`** namespace that Spring Boot 3 / Spring 6 require."

This module assumes the fundamentals from modules 02–07. Several entries here are the modern idiom for patterns described earlier — immutability (module 02), exhaustive dispatch over an inheritance tree (module 03), and the "threads are expensive → use pools" reasoning (module 06).

---

## Records (Java 16)

A `record` is a transparent, **immutable** data carrier. You declare the *components* once in the header and the compiler generates everything else.

```java
public record Point(int x, int y) {}
```

The compiler auto-generates, from that one line:
- `private final` fields `x` and `y`
- a **canonical constructor** `Point(int x, int y)`
- accessors `x()` and `y()` (note: no `get` prefix)
- value-based `equals()` and `hashCode()` (compare all components)
- a `toString()` → `Point[x=1, y=2]`

```java
Point a = new Point(1, 2);
Point b = new Point(1, 2);
System.out.println(a.equals(b)); // true  — value equality, no boilerplate
System.out.println(a);           // Point[x=1, y=2]
System.out.println(a.x());       // 1     — accessor, not getX()
```

### Canonical vs compact constructor

The **canonical constructor** takes every component. You rarely write it in full — instead use the **compact constructor** to validate or normalize (defensive copy), and let the compiler assign the fields for you afterwards.

```java
// GOOD — compact constructor: validate + defensively copy, no explicit field assignment
public record Money(long amountCents, String currency, List<String> tags) {
    public Money {                                   // no parameter list, no body assignment
        if (amountCents < 0) throw new IllegalArgumentException("negative amount");
        currency = currency.toUpperCase();           // normalize
        tags = List.copyOf(tags);                    // defensive copy — records do NOT copy for you
    }
}

// BAD — writing the full canonical constructor just to assign fields (boilerplate the record removed)
public record Money(long amountCents, String currency, List<String> tags) {
    public Money(long amountCents, String currency, List<String> tags) {
        this.amountCents = amountCents;
        this.currency = currency;
        this.tags = tags; // and you forgot to copy — the list is now shared & mutable
    }
}
```

> **Records are shallowly immutable, not deeply immutable.** Final fields stop *reassignment*, but a mutable component (a `List`, `Date`, array) can still be mutated through a leaked reference. Defensive-copy mutable components in the compact constructor — exactly as shown in module 02's immutability section.

### Where to use records

- **DTOs / response objects** — `record UserResponse(Long id, String email, Instant createdAt) {}` is the cleanest way to shape a JSON response. Jackson serializes records out of the box.
- **JPA / Spring Data projections** — interface or record projections let a query return only the columns you need: `record UserView(Long id, String email) {}` used as the query's return type.
- **Value objects in the domain** — `record Money(...)`, `record Coordinate(...)`.
- **Multiple return values** — return a `record` instead of a `Pair`/`Object[]`.
- **`switch` pattern matching targets** — see record patterns below.

### When NOT to use records (the interview trap)

**Do not use a `record` as a JPA `@Entity`.** JPA requires:
- a **no-arg constructor** (records only have the canonical, all-args constructor),
- **mutable fields** (Hibernate sets the generated `@Id` and dirty-checks fields after load — records are final),
- often **proxying / lazy loading**, which needs a non-final class.

So `@Entity` classes stay as ordinary mutable classes; records are for the **DTO/projection/value-object** layer *around* the entity. Also avoid records when you need inheritance (records are implicitly `final` and cannot extend another class) or genuinely mutable state.

---

## Sealed Classes & Interfaces (Java 17)

A **sealed** type restricts *which* classes may extend or implement it — a **closed hierarchy**. This lets the compiler reason about every possible subtype (powering exhaustive `switch`) and documents the domain model precisely.

```java
public sealed interface Shape permits Circle, Rectangle, Triangle {}

public record Circle(double radius)              implements Shape {}
public record Rectangle(double w, double h)      implements Shape {}
public record Triangle(double base, double height) implements Shape {}
```

Every permitted subtype must itself declare its openness: `final` (closed), `sealed` (further-restricted), or `non-sealed` (re-opened for extension). Records are implicitly `final`, so they satisfy this automatically — which is why **sealed interface + record subtypes** is the canonical algebraic-data-type pattern.

### Exhaustive switch over a sealed hierarchy

Because the compiler knows the *complete* set of subtypes, a `switch` covering all of them needs **no `default`** — and if you later add a permitted subtype, the switch **fails to compile** until you handle it. This is compile-time safety you cannot get from an open class hierarchy.

```java
// GOOD — exhaustive, no default needed; adding a 4th Shape breaks compilation here
static double area(Shape s) {
    return switch (s) {
        case Circle c     -> Math.PI * c.radius() * c.radius();
        case Rectangle r  -> r.w() * r.h();
        case Triangle t   -> 0.5 * t.base() * t.height();
    };
}

// BAD — open hierarchy forces a default that silently swallows new subtypes
static double areaOpen(Shape s) {
    if (s instanceof Circle c)    return Math.PI * c.radius() * c.radius();
    if (s instanceof Rectangle r) return r.w() * r.h();
    return 0; // a new subtype lands here unnoticed — a runtime bug, not a compile error
}
```

### Domain-modeling use

Sealed types shine for modeling **a fixed set of states or events**: payment results (`Approved | Declined | Pending`), parser tokens, API command types, the result of an operation (`Success<T> | Failure`). They give you Java's version of sum types / discriminated unions, with the compiler enforcing that you handle every case.

---

## Switch Expressions + Pattern Matching (Java 17 / 21)

### Switch expressions (arrow syntax, `yield`)

A `switch` can now be an **expression that returns a value**, with arrow labels that **don't fall through** (no `break` needed). Use `yield` to return from a multi-statement block.

```java
// GOOD — switch expression: returns a value, no fall-through, no break
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    case THURSDAY, SATURDAY     -> 8;
    case WEDNESDAY              -> 9;
};

// yield for a block body
int score = switch (grade) {
    case "A" -> 4;
    case "B" -> 3;
    default  -> {
        log.warn("unknown grade {}", grade);
        yield 0;
    }
};

// BAD — classic statement switch: fall-through bugs, mutable result variable
int n;
switch (day) {
    case MONDAY: n = 6; break;     // forget a break → fall-through bug
    case TUESDAY: n = 7; break;
    default: n = 0;
}
```

Switch expressions over an `enum` or a `sealed` type are checked for **exhaustiveness** — cover every constant/subtype and you can drop `default`.

### `instanceof` pattern matching (Java 16)

The classic "test then cast" collapses into one step that also binds a typed variable, scoped to where the test holds.

```java
// GOOD — pattern variable u is in scope only when the instanceof is true
if (o instanceof User u) {
    System.out.println(u.email());   // no explicit cast
}

// the binding even flows into && and early returns ("flow scoping")
if (!(o instanceof User u)) return;
process(u);                          // u in scope here

// BAD — the pre-16 ceremony
if (o instanceof User) {
    User u = (User) o;               // redundant cast, easy to get wrong
    System.out.println(u.email());
}
```

### Type & record patterns in `switch` (Java 21)

Java 21 finalized **pattern matching for switch**, letting `switch` branch on *type* and **destructure records** inline.

```java
// Type patterns
String describe(Object o) {
    return switch (o) {
        case Integer i  -> "int " + i;
        case String s   -> "string of length " + s.length();
        case null       -> "null";        // null can be a dedicated label
        default         -> "unknown";
    };
}

// Record (deconstruction) patterns — bind components directly, even nested
sealed interface Shape permits Circle, Rectangle {}
record Point(int x, int y) {}
record Circle(Point center, double r) implements Shape {}
record Rectangle(Point topLeft, Point bottomRight) implements Shape {}

String render(Shape s) {
    return switch (s) {
        case Circle(Point(var x, var y), var r) -> "circle @ (" + x + "," + y + ") r=" + r;
        case Rectangle(Point tl, Point br)      -> "rect " + tl + " → " + br;
    };
}
```

### Guarded patterns (`when`)

Add a boolean guard to a pattern label with `when`:

```java
String classify(Shape s) {
    return switch (s) {
        case Circle c when c.r() > 100 -> "huge circle";
        case Circle c                  -> "circle";
        case Rectangle r               -> "rectangle";
    };
}
```

Order matters — more specific (guarded) labels must come before the unguarded one for the same type, or the code won't compile (dominated label).

---

## Text Blocks (Java 15)

A multi-line string literal delimited by `"""`. Eliminates `\n`, `\"` escaping, and `+` concatenation for embedded JSON, SQL, HTML.

```java
// GOOD — readable embedded SQL/JSON
String sql = """
    SELECT id, email, created_at
    FROM users
    WHERE tenant_id = ?
      AND active = true
    ORDER BY created_at DESC
    """;

String json = """
    {
      "id": %d,
      "email": "%s"
    }
    """.formatted(id, email);   // String.formatted() pairs nicely with text blocks

// BAD — pre-15 escaping and concatenation noise
String sqlOld = "SELECT id, email, created_at\n"
    + "FROM users\n"
    + "WHERE tenant_id = ?\n"
    + "  AND active = true\n";
```

Incidental leading whitespace is stripped relative to the closing `"""`, and the line break *before* the closing delimiter is part of the string only if it's on its own line — use `\` at line end to suppress a newline.

---

## Virtual Threads / Project Loom (Java 21)

Virtual threads are lightweight threads **managed by the JVM**, not the OS. Many thousands (millions) of them are multiplexed onto a small pool of OS **carrier** threads. They make the simple, blocking, thread-per-request style scale again.

### Platform vs virtual threads

| | Platform thread | Virtual thread |
|---|---|---|
| Backed by | One dedicated OS thread | JVM-scheduled; shares a few carrier OS threads |
| Stack | Fixed OS stack (~512KB–1MB) | Small, resizable heap stack (~hundreds of bytes initially) |
| Creation cost | High — OS syscall | Cheap — just an object |
| How many | Thousands before memory/scheduler pressure | Millions |
| Blocking I/O | Blocks the OS thread (wasteful) | **Unmounts** from the carrier so it can run others |
| Use | CPU-bound work, pooled | I/O-bound, thread-per-task |

### Creating them

```java
// One-off virtual thread
Thread vt = Thread.ofVirtual().start(() -> handle(request));

// Thread-per-task executor — a NEW virtual thread per submitted task (no pooling!)
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (Request r : requests) {
        executor.submit(() -> handle(r));     // 100k tasks ≈ 100k virtual threads — fine
    }
} // close() awaits all tasks
```

### Why thread-per-request scales again

Classic reasoning (module 06): platform threads are expensive, so cap them in a pool and the pool size limits concurrency — under load, requests queue behind the pool. With virtual threads, **when a request blocks on I/O (DB, downstream HTTP, LLM call), the virtual thread unmounts and the carrier thread runs another request**. You can have one virtual thread per in-flight request and still serve enormous concurrency with a handful of OS threads, written in plain blocking style — no reactive/callback complexity.

> **This explicitly changes module 06's "threads are expensive → use pools" rule.** That rule was about *platform* threads. Do **not** pool virtual threads (pooling defeats their purpose) — create one per task. Pools still make sense for *CPU-bound* work and for rate-limiting access to a scarce resource (use a `Semaphore`, not a pool, for the latter).

### Pinning — the gotcha to mention

A virtual thread **cannot unmount** (it "pins" its carrier OS thread) while it is inside a `synchronized` block/method or executing a native/JNI frame. If a pinned virtual thread then blocks, it ties up the carrier — and enough pinning can starve the carrier pool, killing scalability.

```java
// BAD — synchronized pins the virtual thread to its carrier across the blocking call
synchronized (lock) {
    var result = jdbcTemplate.query(...); // blocking I/O while pinned — carrier stuck
}

// GOOD — ReentrantLock does NOT pin; the virtual thread can unmount while waiting/blocking
private final ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    var result = jdbcTemplate.query(...);
} finally {
    lock.unlock();
}
```

Rule of thumb in the Loom era: **prefer `ReentrantLock` over `synchronized`** around blocking operations. (Recent JDKs are progressively removing pinning for `synchronized`, but `ReentrantLock` is the safe answer in interviews.) Diagnose pinning with `-Djdk.tracePinnedThreads=full`.

### In Spring Boot

Spring Boot 3.2+ wires virtual threads with a single property:

```properties
spring.threads.virtual.enabled=true
```

This makes Tomcat serve each request on a virtual thread (instead of a bounded worker pool — see module 06's Tomcat note) and routes `@Async` / task executors through virtual threads. Requires Java 21+.

---

## Jakarta Namespace / Spring Boot 3 / Spring 6

> **Near-guaranteed interview question for any Spring Boot 3 resume.** "What changes when you go from Boot 2 to Boot 3?" Have the three baselines below ready.

### 1. `javax.*` → `jakarta.*`

When Java EE moved from Oracle to the Eclipse Foundation as **Jakarta EE**, trademark rules forced a package rename. Every Enterprise-API import changes:

| Old (Java EE, Boot 2) | New (Jakarta EE, Boot 3) | Used for |
|---|---|---|
| `javax.persistence.*` | `jakarta.persistence.*` | JPA — `@Entity`, `@Id`, `@Column` |
| `javax.validation.*` | `jakarta.validation.*` | Bean Validation — `@NotNull`, `@Valid` |
| `javax.servlet.*` | `jakarta.servlet.*` | Servlets, filters, `HttpServletRequest` |
| `javax.annotation.*` | `jakarta.annotation.*` | `@PostConstruct`, `@PreDestroy` |

```java
// GOOD — Spring Boot 3
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.validation.constraints.NotNull;

// BAD — Boot 2 imports; will not compile against Boot 3, and any dependency
// still on javax.* is incompatible until upgraded
import javax.persistence.Entity;
import javax.validation.constraints.NotNull;
```

**What breaks:** the code won't compile until imports are migrated, and **any third-party library still on `javax.*` must be upgraded** to a Jakarta-aware version. Use the OpenRewrite recipe (`org.openrewrite.java.migrate.jakarta.JavaxMigrationToJakarta`) rather than hand-editing.

### 2. JDK 17 baseline

Spring Framework 6 / Boot 3 require **Java 17 minimum** (Boot 2.x ran on Java 8). Upgrade the JDK first; this can surface reflection / strong-encapsulation issues in older libraries.

### 3. Micrometer Tracing replaces Spring Cloud Sleuth

Sleuth is **removed** in Boot 3. Tracing now goes through **Micrometer Tracing** (atop the unified Micrometer Observation API). Swap `spring-cloud-sleuth` for:

```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>
```

(See `interview-qa.md` → Observability for the tracing-propagation details.)

**Practical upgrade order:** bump JDK to 17 → migrate `javax`→`jakarta` (OpenRewrite) → upgrade all dependencies to Jakarta versions → replace Sleuth with Micrometer Tracing → run the test suite. Also expect Hibernate 6 (stricter SQL/dialect behaviour) along for the ride.

---

## Interview Q&A

**Q: What is a record and what does the compiler generate for you?**
> An immutable, transparent data carrier. From `record Point(int x, int y) {}` the compiler generates the private final fields, a canonical (all-args) constructor, accessors (`x()`, `y()` — no `get`), value-based `equals()`/`hashCode()`, and `toString()`. It removes the boilerplate of a value object.

**Q: Why can't a JPA `@Entity` be a record?**
> JPA/Hibernate needs a no-arg constructor, mutable fields (to set the generated `@Id` and dirty-check after load), and often non-final classes for proxying/lazy loading. Records have only the all-args constructor, are final, and have final fields. So entities stay as plain mutable classes; records are for DTOs, projections, and value objects around the entity.

**Q: Records have `final` fields — does that make them deeply immutable?**
> No, only shallowly. A mutable component (a `List`, `Date`, array) can still be mutated through a leaked reference. Defensively copy mutable components in the compact constructor (`tags = List.copyOf(tags)`), and the record won't copy for you.

**Q: What problem do sealed types solve over a normal interface?**
> They define a *closed* hierarchy — `permits` lists every allowed subtype. The compiler can then verify a `switch` is exhaustive (no `default` needed), and adding a new subtype causes every non-exhaustive switch to fail compilation. That turns "did I handle the new case?" from a runtime bug into a compile error. They're Java's discriminated unions / sum types.

**Q: Switch expression vs switch statement?**
> The expression form (arrow labels) returns a value, has no fall-through (no `break`), uses `yield` for block bodies, and is checked for exhaustiveness over enums and sealed types. The old statement form falls through by default (a classic bug source) and can't return a value directly.

**Q: What is `instanceof` pattern matching and flow scoping?**
> `if (o instanceof User u)` tests and binds a typed variable in one step, with no explicit cast. The binding `u` is in scope wherever the compiler can prove the test held — including after `if (!(o instanceof User u)) return;`, where `u` is usable below. This is called flow scoping.

**Q: What are record patterns?**
> Java 21 lets you deconstruct a record inside `instanceof`/`switch`: `case Circle(Point(var x, var y), var r) ->` binds the nested components directly. Combined with sealed types, it gives concise, exhaustive, type-safe branching on a domain model.

**Q: What are virtual threads and how do they differ from platform threads?**
> Virtual threads are JVM-scheduled lightweight threads multiplexed over a few OS carrier threads. They're cheap (hundreds of bytes vs a ~1MB OS stack), and when they block on I/O they *unmount* from the carrier so it can run another task. Platform threads are 1:1 with OS threads, expensive, and block the OS thread when they wait.

**Q: Do virtual threads make thread pools obsolete?**
> For I/O-bound, thread-per-request workloads, yes — create one virtual thread per task (`Executors.newVirtualThreadPerTaskExecutor()`), don't pool them. Pools still matter for CPU-bound work and as a concurrency limiter (though a `Semaphore` is the better limiter). This reverses the classic "threads are expensive, always pool" advice (module 06), which applied to platform threads.

**Q: What is pinning, and how do you avoid it?**
> A virtual thread can't unmount while inside a `synchronized` block or a native frame — it "pins" its carrier OS thread. If it then blocks, the carrier is stuck, and enough pinning starves the carrier pool. Avoid it by using `ReentrantLock` instead of `synchronized` around blocking operations. Trace with `-Djdk.tracePinnedThreads=full`.

**Q: How do you enable virtual threads in Spring Boot?**
> Boot 3.2+ with Java 21: set `spring.threads.virtual.enabled=true`. Tomcat then serves each request on a virtual thread and `@Async`/task executors use virtual threads.

**Q: What changes when migrating from Spring Boot 2 to Spring Boot 3?**
> Three baselines: (1) `javax.*` → `jakarta.*` package rename across JPA/validation/servlet/annotations — code won't compile and `javax`-only dependencies must be upgraded; (2) JDK 17 minimum; (3) Spring Cloud Sleuth removed in favour of Micrometer Tracing. Plus Hibernate 6. Migrate the namespace with OpenRewrite, not by hand.

**Q: When should you use a text block?**
> For multi-line embedded content — SQL, JSON, HTML — where `\n`/`\"` escaping and `+` concatenation hurt readability. Pair with `.formatted(...)` for interpolation.
