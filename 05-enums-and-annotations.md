# Enums & Annotations

> **Quick Reference:** Enums define a fixed, type-safe set of named constants; annotations provide metadata to compiler, JVM, or frameworks without changing business logic. Both are heavily used in Spring Boot for clean, maintainable APIs.

---

## Enums

### What is an Enum?

A special data type used to define a **fixed set of named constants**. Type-safe and readable — prevents invalid input at compile time.

```java
enum Status {
    SUCCESS,
    FAILED,
    PENDING
}

Status s = Status.SUCCESS;
System.out.println(s);  // "SUCCESS"
```

---

### Why Use Enum Instead of String Constants?

```java
// BAD — typo-prone, no compile-time safety
String status = "Actve";   // typo, no compile error, discovered only at runtime

// GOOD — compile-time safety
Status status = Status.ACTIVE;  // typo = compile error immediately
```

**Benefits:**
- **Type safety** — only valid enum values can be assigned
- **Compile-time check** — invalid values caught before running
- **Readability** — self-documenting constants
- **`switch` support** — works natively in switch statements
- **Methods and fields** — enums can carry data and behavior (unlike plain constants)

---

### Enums with Fields, Constructors, and Methods

Enums can have private fields, constructors, and methods — making them powerful, self-contained data carriers.

```java
enum OrderStatus {
    CREATED(100),
    SHIPPED(200),
    DELIVERED(300);

    private final int code;

    OrderStatus(int code) {  // constructor — private by default in enums
        this.code = code;
    }

    public int getCode() {
        return code;
    }
}

System.out.println(OrderStatus.SHIPPED.getCode());  // 200
```

---

### Common Enum Methods

```java
Status s = Status.SUCCESS;

s.name();                     // "SUCCESS" — string name of the constant
s.ordinal();                  // 0, 1, 2... — position in declaration order
Status.values();              // Status[] — array of all constants
Status.valueOf("FAILED");     // Status.FAILED — parse from string
```

**Important:** Avoid `ordinal()` in business logic — if you insert a new constant in the middle, all ordinals after it shift, breaking stored values (e.g., in a database column).

---

### Enums in `switch`
```java
OrderStatus status = OrderStatus.SHIPPED;

switch (status) {
    case CREATED:   System.out.println("Processing order"); break;
    case SHIPPED:   System.out.println("In transit"); break;
    case DELIVERED: System.out.println("Order complete"); break;
}
```

---

### Enums in REST APIs / DTOs

Highly recommended for request/response fields that have a fixed set of valid values.

```java
public class OrderResponse {
    private String orderId;
    private OrderStatus status;  // enum field in DTO
}
```

JSON response (Jackson serializes enum by `name()` automatically):
```json
{
  "orderId": "ORD101",
  "status": "DELIVERED"
}
```

**Jackson deserialization:** Spring Boot's Jackson automatically maps `"DELIVERED"` in JSON → `OrderStatus.DELIVERED` — invalid values throw a `400 Bad Request`.

**Benefits in REST APIs:**
- Standardized, consistent responses
- Prevent invalid state values (frontend can't send `"DLVRD"` by accident)
- Easier frontend integration — known set of valid values
- Cleaner Swagger/OpenAPI documentation — shows valid values explicitly

---

### Real-World Enum Use Cases (Backend Systems)

```java
enum Role              { ADMIN, USER, MODERATOR }
enum JobStatus         { PENDING, RUNNING, COMPLETED, FAILED }
enum Environment       { DEV, STAGING, PROD }
enum PaymentStatus     { INITIATED, SUCCESS, FAILED, REFUNDED }
enum NotificationChannel { EMAIL, SMS, PUSH }
```

**With associated data:**
```java
enum HttpStatusCode {
    OK(200, "Success"),
    NOT_FOUND(404, "Resource not found"),
    INTERNAL_ERROR(500, "Server error");

    private final int code;
    private final String description;

    HttpStatusCode(int code, String description) {
        this.code = code;
        this.description = description;
    }

    public int getCode() { return code; }
    public String getDescription() { return description; }
}
```

---

## Annotations

### What is an Annotation?

Metadata added to code elements (class, method, field, parameter) that gives instructions to the **compiler**, **JVM**, or **frameworks**. Does NOT directly change business logic — provides context and behavior.

```java
@Override
public String toString() {
    return "Demo";
}
```

`@Override` tells the compiler: "verify this method actually overrides a parent method." Compile error if it doesn't — catches typos in method names.

---

### Giving Context to Compiler

```java
@Deprecated
void oldMethod() {}
// Any caller of oldMethod() gets a compiler warning
```

Annotations are processed at:
- **Compile time** — `@Override`, `@Deprecated`, `@SuppressWarnings`
- **Runtime** — `@Transactional`, `@Autowired` (via reflection)
- **Both** — `@FunctionalInterface`

---

### Common Built-in Java Annotations

| Annotation | Purpose |
|---|---|
| `@Override` | Validates method overrides a parent method; compile error if it doesn't |
| `@Deprecated` | Marks API as old; triggers compiler warning when used |
| `@SuppressWarnings("unchecked")` | Suppresses specific compiler warnings |
| `@FunctionalInterface` | Ensures interface has exactly one abstract method; compile error if not |
| `@SafeVarargs` | Suppresses unchecked warnings for varargs with generics |

```java
@FunctionalInterface
interface Calculator {
    int add(int a, int b);
    // Adding a second abstract method here = compile error
}
```

---

### Spring Boot Annotations — Very Important

Spring uses annotations to configure beans, wire dependencies, map HTTP endpoints, and manage transactions — all without XML configuration.

**Bean / Component Annotations:**
| Annotation | Role |
|---|---|
| `@Component` | Generic Spring-managed bean |
| `@Service` | Service layer bean (business logic) |
| `@Repository` | Data access layer bean; enables exception translation |
| `@RestController` | REST controller (combines `@Controller` + `@ResponseBody`) |
| `@Controller` | MVC controller (returns view names) |
| `@Configuration` | Configuration class; defines `@Bean` methods |

**Dependency Injection:**
| Annotation | Role |
|---|---|
| `@Autowired` | Inject dependency (prefer constructor injection) |
| `@Qualifier("name")` | Specify which bean to inject when multiple exist |
| `@Value("${prop.key}")` | Inject value from `application.properties` |

**Web / REST Annotations:**
| Annotation | Role |
|---|---|
| `@RequestMapping("/path")` | Map HTTP requests to class or method |
| `@GetMapping` / `@PostMapping` / `@PutMapping` / `@DeleteMapping` | HTTP method-specific mappings |
| `@PathVariable` | Extract value from URI path `/{id}` |
| `@RequestParam` | Extract query param `?name=Jay` |
| `@RequestBody` | Bind request body JSON to object |
| `@ResponseBody` | Write return value directly to response |
| `@ResponseStatus(HttpStatus.CREATED)` | Set HTTP response status code |

**Data / JPA Annotations:**
| Annotation | Role |
|---|---|
| `@Entity` | Marks class as JPA entity (maps to DB table) |
| `@Table(name="users")` | Specify table name |
| `@Id` | Primary key field |
| `@GeneratedValue` | Auto-generate ID (IDENTITY, SEQUENCE, etc.) |
| `@Column(name="email")` | Map field to DB column |
| `@Transactional` | Wraps method in a DB transaction |

**Other Common Annotations:**
| Annotation | Role |
|---|---|
| `@Async` | Run method in background thread (needs `@EnableAsync`) |
| `@Scheduled(cron="...")` | Schedule method execution |
| `@CacheEvict` / `@Cacheable` | Cache control |
| `@EventListener` | Listen for application events |
| `@ControllerAdvice` | Global exception handler |
| `@ExceptionHandler` | Handle specific exception in a controller or advice |

### Full Spring Boot Controller Example

```java
@RestController
@RequestMapping("/api/users")
class UserController {

    private final UserService userService;

    // Constructor injection — preferred over @Autowired on field
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(userService.getUser(id));
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public UserResponse createUser(@RequestBody CreateUserRequest request) {
        return userService.createUser(request);
    }

    @GetMapping
    public List<UserResponse> getAll(@RequestParam(required = false) String role) {
        return userService.getAll(role);
    }
}
```

---

## Interview Q&A

**Q: What is an enum? Why use over String?**
Enum is a type-safe fixed set of constants. Unlike Strings, enums prevent typos at compile time, support `switch`, and can carry fields/methods. `Status.ACTIVE` is safer than `"ACTIVE"` — invalid values fail at compile time, not runtime.

**Q: Can an enum have a constructor?**
Yes — private by default. Used to attach data to each constant. Example: `OrderStatus.SHIPPED.getCode()` returns `200`. Enum constructors are called when the class loads, once per constant.

**Q: What is `ordinal()` and why avoid it in business logic?**
`ordinal()` returns position (0, 1, 2…) in declaration order. If you insert a new constant in the middle, all subsequent ordinals shift — breaks stored values (DB columns, JSON). Use named constants or explicit field values instead.

**Q: Enum in REST API — how does Jackson handle it?**
Jackson serializes enum by `name()` (e.g., `"DELIVERED"`). Deserialization maps the string back to the enum constant. Invalid values throw `400 Bad Request`. You can customize with `@JsonValue` and `@JsonCreator` if needed.

**Q: What is an annotation?**
Metadata attached to code elements — doesn't change logic, provides instructions. Processed by compiler (`@Override`), JVM (via reflection at runtime), or frameworks (`@Service`, `@Transactional`).

**Q: Does annotation change business logic?**
Not directly — it gives context/instructions. The framework/JVM acts on those instructions. `@Transactional` doesn't change your method's code — Spring wraps it in a transaction proxy at runtime.

**Q: What are the most important Spring Boot annotations for you?**
- `@RestController` + `@RequestMapping` — define REST endpoints
- `@Service`, `@Repository` — layer identification + Spring DI
- `@Autowired` / constructor injection — wire dependencies
- `@Transactional` — DB transaction boundaries
- `@Entity`, `@Id`, `@Column` — JPA entity mapping
- `@ControllerAdvice` + `@ExceptionHandler` — global error handling
- `@Async`, `@Scheduled` — async and scheduled execution

**Q: `@Service` vs `@Component` vs `@Repository`?**
All three are `@Component` specializations (Spring treats them as beans).
- `@Component`: generic bean
- `@Service`: semantic marker for service layer (business logic)
- `@Repository`: semantic marker for DAO layer + enables Spring's persistence exception translation (wraps DB-specific exceptions into Spring's `DataAccessException`)

**Q: Why is constructor injection preferred over `@Autowired` on field?**
- Field can be `final` (immutable after injection)
- Explicit — dependencies visible in constructor signature
- Testable — just pass a mock in tests, no Spring context needed
- Detects circular dependencies at startup, not at runtime
- Field injection uses reflection (bypasses visibility rules) — constructor injection is cleaner

**Q: How do you use enums in your projects?**
`Role.ADMIN/USER` for authorization checks, `OrderStatus.PENDING/SHIPPED/DELIVERED` in order DTOs, `JobStatus.RUNNING/COMPLETED/FAILED` for async job tracking, `PaymentStatus.SUCCESS/FAILED` in payment responses. Enums ensure valid states — no invalid string values can sneak through at compile time.

**Q: Show an enum with a field and use it in an API response DTO.**
> ```java
> public enum OrderStatus {
>     PENDING("Pending Payment"),
>     SUCCESS("Payment Completed"),
>     FAILED("Payment Failed");
>
>     private final String message;
>     OrderStatus(String message) { this.message = message; }
>     public String getMessage() { return message; }
> }
>
> // DTO
> public class UserResponse {
>     private String username;
>     private OrderStatus status;
>     // Jackson serializes enum as its name by default: "PENDING", "SUCCESS", "FAILED"
>     // Use @JsonValue on getMessage() to serialize the display message instead
> }
> ```

**Q: Why use enums in APIs instead of plain Strings?**
> - **Type safety**: `OrderStatus.PENDING` cannot be misspelled; `"PENDIG"` can
> - **Invalid state prevention**: Frontend and backend share the same enum namespace — impossible to pass an unrecognised value
> - **IDE support**: Autocomplete and refactoring work on enums; raw strings are invisible to the compiler
> - **Switch exhaustiveness**: With `switch` on an enum, the compiler warns if a case is missing
> - **Serialization consistency**: Jackson maps enum names to/from JSON automatically; no manual mapping
