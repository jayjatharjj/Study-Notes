# OOP Fundamentals — Classes, Objects, Encapsulation, Constructors & Static

> **Quick Reference:** OOP is a programming paradigm where software is modeled as objects combining state (variables) and behavior (methods). Java achieves encapsulation, polymorphism, inheritance, and abstraction through classes, access modifiers, constructors, and the `static` keyword.

---

## What is OOP? Why Use It?

- **Code Reusability** — write once, reuse via inheritance/composition
- **Maintainability** — changes in one class don't ripple everywhere
- **Scalability** — add features without touching existing code
- **Real-world modeling** — maps naturally to business domains (User, Order, Payment)

**Real-World Analogy:**
- Class → Blueprint (Car design document)
- Object → Actual Car (instance built from that design)
- Methods → Car's functions (`drive()`, `brake()`)
- Variables → Car's properties (`color`, `speed`)

**In Spring Boot apps:** `Controller`, `Service`, `Repository`, `DTO` — every layer is a class applying OOP: encapsulation (private fields), single responsibility, composition (service uses repository).

---

## Class

A **class** is a blueprint/template for creating objects.

```java
class Car {
    String color;   // instance variable
    int speed;

    void drive() {
        System.out.println("Car is driving");
    }
}
```

**Access Rules for Top-Level Classes:**
- `public` — accessible from anywhere
- `default` (no modifier) — accessible within the same package only
- Inner classes additionally support `private` and `static`

**Top-level classes cannot be `private` or `static`:**
```java
private class Car { }  // Error: modifier private not allowed here
static class Car { }   // Error: modifier static not allowed here
```

---

## Object

An **object** is an instance of a class, created using the `new` keyword.

```java
Car myCar = new Car();
myCar.color = "Red";
myCar.drive();
```

### What Happens Behind the Scenes When You Create an Object?

1. `new` keyword allocates memory in **heap**
2. Constructor is invoked to initialize the object
3. Reference (e.g., `myCar`) is stored in **stack**
4. Object itself lives in **heap**

### Object Class

Every class in Java implicitly extends `java.lang.Object`.

Important methods inherited from `Object`:
- `hashCode()` — unique integer for the object
- `equals(Object o)` — check value equality
- `toString()` — string representation; override for logging
- `finalize()` — called by GC before destruction (deprecated in modern Java)
- `getClass()` — returns runtime class info
- `clone()` — creates a field-for-field copy (requires implementing `Cloneable`)
- `intern()` — (String-specific) returns the canonical string from the string pool

---

## Types of Variables

### Instance Variables
- Declared inside class, outside any method
- Each object gets its **own copy**
- Stored in **heap** (inside the object)
- Gets a default value (`0` for int, `null` for objects, etc.)

```java
class User {
    String name;  // instance variable — unique per object
    int age;
}
```

### Local Variables
- Declared inside a method/block
- Stored in **stack**
- **No default value** — must initialize explicitly before use

```java
void greet() {
    String message = "Hello";  // local — must initialize
    System.out.println(message);
}
```

### Static Variables
- Declared with `static` keyword inside class
- **Shared across ALL objects** — one copy in memory
- Stored in **Method Area / Metaspace**

```java
class Counter {
    static int count = 0;  // shared across all Counter instances
}
```

### Variable Comparison Table

| | Instance Variable | Local Variable | Static Variable |
|---|---|---|---|
| Location | Class body | Method/block | Class body (static) |
| Memory | Heap | Stack | Method Area / Metaspace |
| Default value | Yes (`0`, `null`) | No (must init) | Yes (`0`, `null`) |
| Belongs to | Object | Method | Class |
| Lifetime | As long as object | Until method returns | As long as class is loaded |

---

## Methods

A **method** is a block of code that performs a specific task.

```java
// Syntax: returnType methodName(parameters) { body }
int add(int a, int b) {
    return a + b;
}
```

### Return Type
- Specifies what the method returns
- `void` means the method returns nothing

### Calling Methods
```java
User u = new User();
u.display();              // instance method call via object
int result = add(5, 3);   // method with return value
```

### Pass by Value
Java **always** passes by value — copies the value (or copies the reference for objects, not the object itself).

```java
void changeValue(int x) {
    x = 100;  // does NOT affect caller's variable
}

void modifyObject(User u) {
    u.name = "Jay";    // DOES affect — modifying object via copied reference
    u = new User();    // does NOT affect caller's reference
}
```

---

## Immutable Objects

An **immutable object** is one whose state **cannot be changed** after creation.

- `String` is the most common example — once created, its value never changes; operations like `concat()` return a new object
- To create an immutable class: make all fields `private final`, no setters, initialize via constructor only
- Benefits: thread-safe (no synchronization needed), safe to share across methods and caches

```java
// String is immutable — every "modification" creates a new object
String s = "Hello";
s = s.concat(" World");  // does NOT change original; s now points to a new object
```

### Building a Fully Immutable Class (with defensive copies)

`private final` fields alone are **not enough** when a field is itself mutable (a `Date`, a collection, an array). A caller who keeps a reference to the object you stored — or who gets a reference to your internal field via a getter — can mutate your "immutable" state from the outside. Fix: **defensively copy** mutable inputs in the constructor and mutable outputs in getters.

```java
// GOOD — defensive copies guard mutable fields and collections
public final class Trip {
    private final String destination;
    private final Date departure;            // Date is mutable!
    private final List<String> travellers;   // List is mutable!

    public Trip(String destination, Date departure, List<String> travellers) {
        this.destination = destination;
        this.departure   = new Date(departure.getTime());        // copy IN
        this.travellers  = List.copyOf(travellers);              // unmodifiable copy IN
    }

    public String getDestination() { return destination; }
    public Date getDeparture()     { return new Date(departure.getTime()); } // copy OUT
    public List<String> getTravellers() { return travellers; }   // already unmodifiable
}

// BAD — no copies: caller mutates internal state after construction
// this.departure = departure;          // caller's Date and ours are the SAME object
// public Date getDeparture() { return departure; } // hands out the live reference
```

### The `record` equivalent (modern idiom, Java 16+)

A `record` auto-generates the constructor, `final` fields, accessors, `equals()`, `hashCode()`, and `toString()` — but it does **not** add defensive copies for you. For mutable components, do the copying in a **compact constructor**:

```java
// GOOD — record with a compact constructor for defensive copying
public record Trip(String destination, Date departure, List<String> travellers) {
    public Trip {                                  // compact constructor
        departure  = new Date(departure.getTime());
        travellers = List.copyOf(travellers);
    }
}
```

See **module 08 — Modern Java** for the full treatment of records, including when NOT to use them (e.g., JPA `@Entity`).

---

## Instance Block
- Runs **before constructor** every time an object is created
- Used to initialize instance variables shared across constructors

```java
class Demo {
    {
        System.out.println("Instance block — runs before constructor");
    }

    Demo() {
        System.out.println("Constructor");
    }
}
```

---

## Instance Methods
- Belong to objects, not the class
- Can access both static and instance variables/methods
- Require an object to be called

```java
class User {
    String name;

    void display() {
        System.out.println("Name: " + name);  // accesses instance variable
    }
}
```

---

## Static Block
- Executes **once** when the class is loaded into JVM, before `main()`
- Used to initialize static variables
- Common use: one-time setup — DB connection pool, loading config properties

```java
class Config {
    static String dbUrl;

    static {
        dbUrl = "jdbc:mysql://localhost/mydb";
        System.out.println("Class loaded — static block executed");
    }
}
```

---

## Static Methods
- Belong to the class, not any object
- Use stack memory when called
- Can only access **static** variables and methods directly (no `this`, no instance members)
- Called using class name: `ClassName.method()`

```java
class MathUtils {
    static int square(int n) {
        return n * n;
    }
}
MathUtils.square(5);  // 25 — no object needed
```

---

## Access Modifiers

| Modifier | Same Class | Same Package | Subclass | Any Class |
|---|---|---|---|---|
| `private` | ✅ | ❌ | ❌ | ❌ |
| `default` (none) | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

```java
public class Demo {
    public int a;       // accessible everywhere
    private int b;      // same class only
    int c;              // default — same package only
    protected int d;    // package + subclasses
}
```

**Key Points:**
- `private` is most restrictive — use for all instance variables (encapsulation)
- `default` = "package-private" — no keyword needed, no modifier written
- `public` is least restrictive — use for API methods

**Real-world usage in Spring Boot:**
- `private` → entity fields, internal helpers, service implementations
- `public` → controller endpoints (`@GetMapping`), service interface methods
- `protected` → base class methods in inheritance hierarchies
- `default` → package-internal utilities

---

## Composition (HAS-A Relationship)

One class **contains** an object of another class as a member.

```java
class Engine {
    void start() {
        System.out.println("Engine started");
    }
}

class Car {
    Engine engine = new Engine();  // Car HAS-A Engine

    void drive() {
        engine.start();
        System.out.println("Car moving");
    }
}
```

**In Spring Boot:** `UserService` HAS-A `UserRepository`, `OrderService` HAS-A `PaymentGateway` and `InventoryService`.

---

## OOP Design Principles

- **Modularity** — each class has one responsibility (Single Responsibility)
- **Reusability** — inheritance, composition
- **Low coupling** — objects depend on interfaces, not concrete classes
- **High cohesion** — related behavior grouped in one class
- **SOLID** principles
- **DRY** — Don't Repeat Yourself

---

## JVM Structure

| Component | Role |
|---|---|
| Class Loader | Loads `.class` files into JVM |
| Method Area | Stores class metadata, static variables |
| Heap Memory | Stores objects and arrays |
| Stack Memory | Stores method frames, local variables, references |
| Execution Engine | Interpreter + JIT compiler |

### Heap vs Stack — Memory Deep Dive

```java
class Demo {
    int x = 10;
}

Demo d = new Demo();
```

Memory breakdown:
- Reference `d` → **Stack**
- `Demo` object → **Heap**
- Variable `x = 10` → inside heap (part of the object)

**Thread safety:**
- **Heap** — shared across all threads (objects and static variables accessible by all threads)
- **Stack** — thread-specific; each thread has its own stack (local variables are never shared)

**What happens when a method is called:**
1. JVM pushes a new **stack frame** for the method
2. Local variables, parameters, and the return address go into this frame
3. When method returns, frame is **popped** and memory freed automatically

### Full Java Execution Flow

```
MyClass.java  →  (javac compiler)  →  MyClass.class (bytecode)
                                              ↓
                                     JVM loads class
                                              ↓
                                  Stack frame for main() created
                                              ↓
                              Objects allocated on heap via `new`
                                              ↓
                              References stored in stack frames
                                              ↓
                        Method calls → new stack frames pushed/popped
```

1. `.java` source file compiled by `javac` into `.class` bytecode
2. JVM's Class Loader loads the `.class` file into Method Area
3. Static block executes once (class initialization)
4. JVM creates a stack frame for `main()`
5. `new` allocates objects on heap; references stored on stack
6. Method calls push/pop frames; program runs via Execution Engine (Interpreter + JIT)

---

## Encapsulation

**Definition:** Binding data (variables) and behavior (methods) into a single unit, and restricting direct access to internal data using access modifiers.

**Achieved with:**
- `private` variables (hide internal state)
- `public` getter/setter methods (controlled access)

```java
class Student {
    private int id;
    private String name;

    public void setId(int id) {
        this.id = id;
    }
    public int getId() {
        return id;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
}
```

### Benefits
- **Data hiding** — internal state cannot be accessed/modified directly
- **Security** — add validation logic inside setters
- **Controlled access** — read-only or write-only fields possible
- **Maintainability** — internal implementation can change without affecting callers

**In Spring Boot:**
- Entity fields are `private` — accessed via getters/setters (Lombok `@Getter`/`@Setter`)
- DTOs expose only the fields needed for the API contract (e.g., never expose `passwordHash`)
- Services expose business methods, hide implementation details

```java
@Entity
class User {
    @Id
    private Long id;
    private String email;
    private String passwordHash;  // never exposed in response DTO

    public String getEmail() { return email; }
    // no getPasswordHash() in response DTO
}
```

---

## `this` Keyword

`this` refers to the **current object instance**.

### Use 1: Resolve Variable Name Conflict
```java
class Test {
    int x;

    void setX(int x) {
        this.x = x;  // this.x = instance variable; x = parameter
    }
}
```

### Use 2: Call Another Constructor (Constructor Chaining)
```java
class Test {
    int a, b;

    Test() {
        this(10, 20);  // calls parameterized constructor — must be first line
    }

    Test(int a, int b) {
        this.a = a;
        this.b = b;
    }
}
```

### Use 3: Pass Current Object as Argument
```java
class Test {
    void show() {
        printObject(this);  // passing current object to another method
    }
}
```

**Key Points:**
- `this` is implicit in all instance methods
- `this` does NOT exist in static methods (no object context)
- Used in Spring Boot constructor injection to avoid field-name conflicts

---

## Constructors

A **constructor** is a special method used to **initialize objects**.

**Rules:**
- Same name as the class
- No return type (not even `void`)
- Called automatically when `new` is used
- First line is implicitly `super()` (calls parent constructor)
- Constructor's stack frame is pushed to stack during execution

```java
class Demo {
    Demo() {
        // super() is implicitly here — calls Object constructor
        System.out.println("Constructor called");
    }
}
```

### Types of Constructors

#### 1. Default Constructor (Compiler-Provided)
```java
class Demo {}
// Compiler silently adds: Demo() { super(); }
```
Added automatically **only if no constructor** is defined in the class.

#### 2. No-Argument Constructor (User-Defined)
```java
class Demo {
    Demo() {
        System.out.println("No-arg constructor");
    }
}
```
Once you define any constructor, the compiler **no longer** provides a default.

#### 3. Parameterized Constructor
```java
class User {
    String name;
    int age;

    User(String name, int age) {
        this.name = name;   // parameters are local to this constructor
        this.age = age;
    }
}
User u = new User("Jay", 25);
```

#### 4. Copy Constructor (Custom)
```java
class Demo {
    int x;

    Demo(Demo obj) {
        this.x = obj.x;  // copy state from another object
    }
}
```
Java does NOT provide a copy constructor automatically (unlike C++) — write it manually when needed.

### Constructor Overloading
```java
class Demo {
    Demo() { }
    Demo(int x) { }
    Demo(int x, String name) { }
}
```

### Constructor vs Method

| | Constructor | Method |
|---|---|---|
| Name | Same as class | Any valid name |
| Return type | None (not even void) | Must have one |
| Called | Automatically at `new` | Explicitly by name |
| Purpose | Initialize object | Define behavior |

### Can a Constructor be Private?
Yes — used in the **Singleton design pattern** to prevent external instantiation.

```java
class Singleton {
    private static Singleton instance;

    private Singleton() {}  // prevents new Singleton()

    public static Singleton getInstance() {
        if (instance == null) instance = new Singleton();
        return instance;
    }
}
```

**In Spring Boot:** Constructor injection is the recommended DI style.

```java
@Service
class UserService {
    private final UserRepository repo;

    public UserService(UserRepository repo) {
        this.repo = repo;  // constructor injection — best practice
    }
}
```
Preferred over `@Autowired` on fields because: the field is `final` (immutable after injection), easy to test (just pass a mock), and dependencies are explicit.

---

## `static` Keyword

`static` means the member **belongs to the class**, not to any specific object.

### Static Variable
```java
class Counter {
    static int count = 0;  // shared by ALL Counter objects

    Counter() {
        count++;  // every new object increments the shared count
    }
}
```
- Stored in **Method Area / Metaspace**
- Initialized once when class loads
- Accessed as `Counter.count` (preferred over `obj.count`)

### Static Method
```java
class MathUtils {
    static int square(int n) {
        return n * n;
    }
}
MathUtils.square(5);  // called without creating object
```
- Can only access **static** variables/methods (no `this`, no instance members)
- Cannot be overridden — only hidden (see Method Hiding in OOP Advanced)

### Static Block
```java
class Config {
    static String dbUrl;

    static {
        dbUrl = "jdbc:mysql://localhost/mydb";
        System.out.println("Class loaded — static block runs once");
    }
}
```
- Runs **once** when the class is loaded into JVM, before `main()`
- Used for: DB connections, loading external config, class-level initialization

### Accessing Static Members
```java
Counter.count;   // via class name (preferred — makes static nature clear)
obj.count;       // via object reference (works but misleading — avoid)
```

### Static vs Instance

| | Static | Instance |
|---|---|---|
| Belongs to | Class | Object |
| Memory | Method Area | Heap |
| Access | Without object | Needs object |
| Can access instance vars | ❌ | ✅ |
| Overridable | ❌ (hidden only) | ✅ |

**Can static access non-static?**
No directly — static context has no `this`. Would need to create an object: `new Demo().instanceVar`.

**In Spring Boot:**
- Static → utility classes (`StringUtils`, `DateUtils`), constants (`AppConstants.MAX_RETRY`), logging helpers
- `@Bean` factory methods in `@Configuration` classes are often static
- Avoid static state in services — makes testing harder and causes issues in multi-instance deployments

---

## Interview Q&A

**Q: What is OOP?**
A paradigm where software is modeled as objects combining state (variables) and behavior (methods). Enables reusability (inheritance), modularity (encapsulation), and flexibility (polymorphism). In Spring Boot: controllers, services, repositories, and DTOs are all OOP applied to backend.

**Q: Class vs Object?**
Class is a blueprint/design; object is an instance built from it. `User` class defines what a user is; `new User()` creates an actual user in memory (on heap).

**Q: What happens when you create an object?**
1. JVM loads the class (if not already loaded)
2. `new` allocates memory on heap
3. Constructor executes to initialize instance variables
4. Reference stored on stack
5. Object ready for use

**Q: Types of variables?**
- **Local** — inside methods, on stack, no default value, must initialize
- **Instance** — inside class, unique per object, on heap, has default value
- **Static** — shared across all instances, in Method Area, has default value

**Q: What is pass-by-value in Java?**
Java always copies. For primitives: copies the value. For objects: copies the reference (address) — not the object. Modifying the object's state via the copied reference works; reassigning the reference does not affect the caller.

**Q: What is encapsulation?**
Binding data + methods together + restricting access via `private` fields + `public` getters/setters. Ensures callers interact through a controlled API, not raw fields. In Spring Boot: entity fields are `private` to prevent arbitrary mutation.

**Q: What is `this`?**
Reference to the current object. Resolves naming conflict between instance variable and parameter, enables constructor chaining (`this()`), and can pass the current object as an argument.

**Q: Can `this` be used in a static method?**
No. Static methods have no object context — `this` would have nothing to refer to.

**Q: What is a constructor? Types?**
Special method (same name as class, no return type) invoked automatically at `new`.
Types: default (compiler-provided), no-arg, parameterized, copy (all user-defined).

**Q: Can a constructor be overloaded?**
Yes — multiple constructors with different parameter lists.

**Q: Can a constructor be private?**
Yes — Singleton pattern uses a private constructor to prevent external `new` calls.

**Q: What is `static`?**
Makes a member belong to the class, not individual objects. Shared state (static variables), utility methods (static methods), one-time initialization (static blocks).

**Q: Can static access non-static?**
No directly. Static has no `this`. Non-static members require an object reference.

**Q: What is a static block?**
Runs once when the class loads — before `main()`. Used for class-level initialization (DB connection pool setup, config loading).

**Q: How is constructor injection used in Spring Boot?**
```java
@Service
class OrderService {
    private final PaymentGateway paymentGateway;

    public OrderService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }
}
```
Spring injects the dependency via constructor. Advantages: `final` field (immutable), easy to test (pass mock), explicit dependencies visible in constructor signature.

**Q: Why use composition over inheritance?**
Inheritance creates tight coupling (child bound to parent's internals). Composition allows behavior swapping (different implementations at runtime), easier testing (mock dependencies), and avoids the fragile base class problem. In microservices: services composed of repositories + clients are easier to evolve independently.

**Q: What is an immutable object?**
An object whose state cannot be changed after creation. `String` is the canonical example — every "modification" produces a new object. Benefits: inherently thread-safe, safe to cache and share. Implemented by making all fields `private final` and providing no setters.

**Q: Give an OOP example from your Spring Boot project.**
`UserService` (class/bean) has `private final UserRepository userRepository` (encapsulation + composition). `UserRepository` extends `JpaRepository` (inheritance). Controller depends on the `UserService` interface (polymorphism + abstraction). The DI container wires the concrete `UserServiceImpl` at runtime — callers never know the implementation.

**Q: How do you design a service using OOP?**
- `Controller` → receives HTTP request, delegates to service (single responsibility)
- `Service` → business logic; depends on repository via interface (abstraction, loose coupling)
- `Repository` → DB access; Spring Data provides implementation (polymorphism)
- `DTO` → data transfer; only exposes fields needed for the API contract (encapsulation)
Each layer is a class with one clear responsibility — classic OOP applied to layered architecture.

**Q: What is middleware?**
Software layer between components. In Spring Boot: `Filter`, `Interceptor`, `API Gateway` (Spring Cloud), Auth service (JWT/OAuth2).
