# Exception Handling, Custom Exceptions & Memory Management

> **Quick Reference:** Exception handling ensures programs degrade gracefully on runtime failures; custom exceptions represent domain-specific failures clearly; GC manages heap memory automatically; `final`, `finally`, and `finalize()` are three distinct but commonly confused constructs.

---

## What is an Exception?

An **unexpected event** that disrupts the normal flow of a program at runtime.

---

## Exception Hierarchy

```
java.lang.Throwable
├── Error  (serious JVM/system — do NOT catch)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── NoClassDefFoundError
└── Exception
    ├── Checked Exception  (compile-time — must handle or declare)
    │   ├── IOException
    │   ├── SQLException
    │   └── FileNotFoundException
    └── Unchecked Exception / RuntimeException  (runtime — no mandatory handling)
        ├── NullPointerException
        ├── ArithmeticException
        ├── ClassCastException
        └── ArrayIndexOutOfBoundsException
```

---

## Types of Exceptions

### Checked Exceptions (Compile-Time)
- Compiler forces you to either `try-catch` them or declare `throws`
- Represent **recoverable external failures** (file not found, DB connection lost)
- Examples: `IOException`, `SQLException`, `FileNotFoundException`

```java
FileReader fr = new FileReader("a.txt");  // must handle FileNotFoundException
```

### Unchecked Exceptions (Runtime)
- No mandatory handling — compiler doesn't enforce it
- Represent **programming errors** or violated business rules
- Examples: `NullPointerException`, `ArithmeticException`, `ArrayIndexOutOfBoundsException`

```java
int a = 10 / 0;  // ArithmeticException at runtime
```

### Errors
- Serious JVM/system-level failures — **do NOT catch** in application code
- Examples: `OutOfMemoryError` (heap exhausted), `StackOverflowError` (infinite recursion)

---

## try-catch-finally

```java
try {
    int a = 10 / 0;                          // risky code
} catch (ArithmeticException e) {
    System.out.println("Caught: " + e.getMessage());  // specific first
} catch (Exception e) {
    System.out.println("General exception");           // general last
} finally {
    System.out.println("Always executes");             // cleanup
}
```

**Execution Flow:**
- `try` → execute risky code
- `catch` → handle exception — **order matters: specific before general**
- `finally` → **always executes**, even after `return`, `break`, or `continue`
- Does NOT run on JVM crash or `System.exit()` call

---

## throw vs throws

```java
// throw — explicitly throw an exception object (in method body)
throw new IllegalArgumentException("Age must be >= 18");

// throws — declare in method signature that this method may propagate an exception
void readFile() throws IOException {
    FileReader fr = new FileReader("file.txt");
}
```

| | `throw` | `throws` |
|---|---|---|
| Location | Method body | Method signature |
| Purpose | Actually throws exception | Declares possible exception |
| Usage | `throw new ExceptionType(msg)` | `void method() throws ExType` |

---

## try-with-resources (Java 7+)

Automatically closes resources implementing `AutoCloseable` — cleaner than `finally` for resource cleanup.

```java
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    System.out.println(br.readLine());
}  // br.close() called automatically, even if exception occurs
```

Multiple resources:
```java
try (
    Connection conn = dataSource.getConnection();
    PreparedStatement ps = conn.prepareStatement(sql)
) {
    // use conn and ps
}  // both closed automatically in reverse order
```

**Why better than `finally`:**
- Cleaner, less boilerplate
- No resource leak — closed even if exception thrown in `try`
- Resources closed **before** any `finally` block runs
- Works with anything implementing `AutoCloseable`: file streams, DB connections, `Scanner`, HTTP clients

**In Spring Boot / microservices:** DB connections from HikariCP pool, file uploads, external HTTP clients — always close properly. Spring's `@Transactional` and JPA manage DB connections automatically, but manual JDBC code needs `try-with-resources`.

---

## Custom Exceptions

### Why Create Custom Exceptions?

To represent **domain-specific failures** with clear, meaningful names — not generic exceptions.

```java
// Generic — vague, hard to catch specifically
throw new RuntimeException("user not found");

// Custom — clear name, catchable by type, carries domain context
throw new UserNotFoundException("User with id 42 not found");
```

### Custom Checked Exception (Recoverable — Caller Must Handle)

```java
class InvalidAgeException extends Exception {
    public InvalidAgeException(String message) {
        super(message);
    }
}

void register(int age) throws InvalidAgeException {
    if (age < 18) {
        throw new InvalidAgeException("Age must be 18 or above. Got: " + age);
    }
}
```

### Custom Unchecked Exception (Runtime — No Mandatory Handling)

```java
class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }

    // second constructor to wrap a cause (for exception chaining)
    public UserNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

**When to use which:**
- **Checked** → caller can meaningfully recover (e.g., `FileNotFoundException` → retry with different path)
- **Unchecked** → programming/business rule violation (e.g., `UserNotFoundException` → return 404, no recovery needed in calling code)
- **In REST APIs:** prefer unchecked + `@ControllerAdvice` for clean separation of concerns

### Using Custom Exceptions in Spring Boot Service

```java
@Service
class UserService {
    @Autowired
    private UserRepository repo;

    public User getUser(Long id) {
        return repo.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User not found with id: " + id));
    }
}
```

---

## Global Exception Handler — `@ControllerAdvice`

Centralizes exception handling across all controllers — consistent API error responses.

```java
@ControllerAdvice
class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("USER_NOT_FOUND", ex.getMessage()));
    }

    @ExceptionHandler(InsufficientBalanceException.class)
    public ResponseEntity<ErrorResponse> handleInsufficientBalance(InsufficientBalanceException ex) {
        return ResponseEntity
            .status(HttpStatus.PAYMENT_REQUIRED)
            .body(new ErrorResponse("INSUFFICIENT_BALANCE", ex.getMessage()));
    }

    @ExceptionHandler(Exception.class)  // fallback for any unhandled exception
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ErrorResponse("INTERNAL_ERROR", "An unexpected error occurred"));
    }
}
```

---

## Real Business Use Cases for Custom Exceptions

**Banking:**
```java
if (balance < amount) {
    throw new InsufficientBalanceException(
        "Required: " + amount + ", Available: " + balance);
}
```

**E-commerce:**
```java
if (stock == 0) {
    throw new ProductOutOfStockException(
        "Product " + productId + " is currently unavailable");
}
```

**User Module:**
```java
if (user == null) {
    throw new UserNotFoundException(
        "No user found with email: " + email);
}
```

**Why critical in microservices:**
- Clean error contracts between services
- Meaningful HTTP status codes in REST responses (`404`, `402`, `422`)
- Easier debugging — exception name reveals root cause instantly
- Consistent error response format across all endpoints via `@ControllerAdvice`

---

## Garbage Collection (GC)

### What is GC?

Automatic memory management process that removes unused (unreachable) objects from heap memory. Java has no `free()` or `delete` — GC handles it.

```java
Test t1 = new Test();
t1 = null;  // object now has no live reference — eligible for GC
```

### How GC Works

1. JVM tracks object **reachability** via GC roots (stack references, static variables, active threads)
2. Objects with no path from any GC root = **unreachable**
3. GC reclaims memory from unreachable objects in heap

**Cannot force GC — only request:**
```java
System.gc();  // hint to JVM; JVM may or may not comply immediately
```

### GC Generations

- **Young Generation (Eden + Survivor spaces):** New objects allocated here; **Minor GC** runs frequently, fast
- **Old Generation (Tenured heap):** Long-lived objects promoted here; **Major GC / Full GC** runs less frequently, slower
- **Metaspace (Java 8+):** Class metadata — replaces old PermGen

**GC Pause Impact:** Long GC pauses affect response times (stop-the-world pauses). In latency-sensitive microservices, the GC algorithm and heap sizing matter.

### GC Algorithms

| Algorithm | Best For | JVM Flag |
|---|---|---|
| G1GC (default, Java 9+) | Balanced throughput + low latency | `-XX:+UseG1GC` |
| ZGC (Java 15+) | Very low pause times (sub-millisecond) | `-XX:+UseZGC` |
| Shenandoah | Ultra-low pause (Red Hat) | `-XX:+UseShenandoahGC` |
| Parallel GC | Max throughput (batch jobs) | `-XX:+UseParallelGC` |

**JVM flags for Spring Boot services:**
```bash
-XX:+UseG1GC         # G1 GC — good default for microservices
-Xms512m             # initial heap size
-Xmx2g               # max heap size
-XX:MaxGCPauseMillis=200  # target max GC pause (G1 tries to meet this)
```

---

## final vs finally vs finalize()

Three distinct and commonly confused constructs.

### `final` — Language Keyword

Restricts modification at the language level.

```java
final int x = 10;           // variable — constant, cannot be reassigned after init

final void method() {}      // method — cannot be overridden in any subclass

final class String {}       // class — cannot be extended (that's why String is immutable)
```

### `finally` — Exception Handling Block

Cleanup block that **always runs** after try-catch, regardless of what happened.

```java
try {
    // risky code
} catch (Exception e) {
    // handle
} finally {
    connection.close();  // always runs — resource cleanup
    System.out.println("Cleanup done");
}
```

### `finalize()` — Object Method (Deprecated — Avoid)

Called by the GC **before destroying an object** — but execution timing is completely unpredictable, and it may never run at all.

```java
@Override
protected void finalize() throws Throwable {
    System.out.println("Object about to be destroyed");
    super.finalize();
}
```

**Why `finalize()` is bad:**
- Timing is unpredictable — GC decides when (or if) it runs
- Object resurrection is possible (bad for GC)
- Can delay object collection and increase memory pressure
- Deprecated since Java 9, scheduled for removal in a future JDK
- Use `try-with-resources` or explicit `close()` via `AutoCloseable` instead

### Summary Table

| | `final` | `finally` | `finalize()` |
|---|---|---|---|
| Type | Keyword | Block | Method |
| Purpose | Restrict modification | Cleanup after try-catch | GC callback before destruction |
| When runs | Compile/runtime (restriction enforced) | After try/catch, always | Before GC — unpredictable |
| Modern status | Widely used ✅ | Widely used ✅ | Deprecated — avoid ❌ |

---

## Interview Q&A

**Q: What is an exception?**
An event that disrupts normal program flow at runtime. Java represents exceptions as objects inheriting from `Throwable`.

**Q: Checked vs unchecked exception?**
- Checked: compile-time, must handle (`try-catch`) or declare (`throws`), represents recoverable external failures (`IOException`)
- Unchecked: runtime, no mandatory handling, represents programming errors or business rule violations (`NullPointerException`, `UserNotFoundException`)
- Errors: serious JVM issues — never catch (`OutOfMemoryError`)

**Q: `throw` vs `throws`?**
`throw` (verb) actually throws an exception object in method body: `throw new Exception()`.
`throws` (declaration) in method signature warns callers: `void method() throws IOException`.

**Q: What is try-with-resources?**
Java 7+ syntax that auto-closes resources implementing `AutoCloseable`. Resources close in reverse order after `try` block exits, before any `finally`. Cleaner than `finally` for file/DB/network resources.

**Q: When to create custom exceptions?**
When you need to represent a **domain-specific failure** clearly (e.g., `UserNotFoundException`, `InsufficientBalanceException`). Use unchecked (extends `RuntimeException`) for business rule violations in REST services — pair with `@ControllerAdvice` for consistent API error responses.

**Q: How do you handle exceptions in microservices (Spring Boot)?**
Custom exceptions + `@ControllerAdvice` for global handling. Each exception maps to a meaningful HTTP status and error response. This ensures consistent error contracts across all endpoints — easier frontend integration and debugging.

**Q: What is GC? Can you force it?**
GC automatically reclaims heap memory from unreachable objects. Cannot force it — only request via `System.gc()`. JVM decides timing. Modern GCs (G1GC, ZGC) minimize stop-the-world pauses.

**Q: What causes GC pauses? How to reduce them?**
Stop-the-world pauses occur when GC needs to scan live objects. Reduce by: sizing heap appropriately (`-Xmx`), using G1GC or ZGC for low-latency, reducing object allocation rate (reuse objects, use pooling), and tuning GC target pause time (`-XX:MaxGCPauseMillis`).

**Q: GC generations — why do they exist?**
Most objects are short-lived (created and discarded quickly). Generational GC optimizes this — Minor GC frequently cleans the young generation (fast, small scope). Long-lived objects are promoted to old generation where Full GC runs infrequently but takes longer.

**Q: `final` vs `finally` vs `finalize()`?**
- `final`: language restriction (variable/method/class can't change)
- `finally`: cleanup block, always runs after try-catch
- `finalize()`: deprecated Object method, called by GC before destruction — avoid; use `try-with-resources` instead

**Q: Why is `finalize()` bad?**
Execution timing is unpredictable — may never run. Can delay object collection (GC must do extra work). Deprecated since Java 9 for removal. For resource cleanup: use `AutoCloseable` + `try-with-resources` or explicit `close()`.

**Q: `finally` — does it always run?**
Almost always — runs after try/catch even with `return`, `break`, `continue`. Does NOT run on JVM crash or `System.exit()` call.

**Q: How does `@ControllerAdvice` help?**
Centralizes exception-to-response mapping. Instead of try-catch in every controller method, you define exception handlers once. Spring catches unhandled exceptions thrown from any controller and routes them to the matching `@ExceptionHandler` — consistent HTTP status codes and error body format across all endpoints.
