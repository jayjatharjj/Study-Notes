# Advanced OOP — Inheritance, Polymorphism, Abstraction, Interfaces, Inner Classes & Design

> **Quick Reference:** Advanced OOP covers how classes relate through inheritance hierarchies, how behavior is shaped via polymorphism and abstraction, how interfaces enable loose coupling and multiple inheritance, and how composition vs encapsulation differ as design strategies.

---

## Inheritance

**Definition:** Allows a child class to acquire the properties and behaviors (non-private fields and methods) of a parent class.

```java
class Animal {
    String name;

    void eat() {
        System.out.println("Animal is eating");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("Dog is barking");
    }
}

Dog d = new Dog();
d.eat();   // inherited from Animal
d.bark();  // own method
```

**Purpose:**
- Code reusability — don't rewrite existing behavior
- IS-A relationship — `Dog IS-A Animal`
- Establish class hierarchies

### Types of Inheritance

| Type | Description | Supported in Java |
|---|---|---|
| Single | One parent → one child | ✅ |
| Multilevel | A → B → C (chain) | ✅ |
| Hierarchical | One parent → many children | ✅ |
| Multiple (class) | Two parents → one child | ❌ via class (diamond problem) |
| Multiple (interface) | Two interfaces → one class | ✅ via interfaces |

### What is NOT Inherited?
- `private` variables and methods
- Constructors (but `super()` calls the parent constructor)

### Constructor Execution in Inheritance
When a child object is created, the **parent constructor runs first**.

```java
class Parent {
    Parent() { System.out.println("Parent constructor"); }
}

class Child extends Parent {
    Child() {
        super();  // implicitly added by compiler as first line
        System.out.println("Child constructor");
    }
}

new Child();
// Output:
// Parent constructor
// Child constructor
```

Only **one** child object is created. Parent instance variables are initialized via `super()`.

### Static Members and Inheritance
- Parent static variables/methods are **callable** via child's class name
- Parent instance variables/methods are callable via child's object
- Static methods in child **hide** the parent's static method (not override — see Method Hiding below)

---

## `this` and `super` Keywords in Inheritance

### `this` — Current Object
- Refers to the current object instance
- Implicit in all instance methods
- Does NOT exist in static methods

### `super` — Parent Class Reference
- Refers to the parent class's version of a variable or method
- `super()` — calls the parent class constructor (must be first line in constructor)
- `super.method()` — calls parent's overridden method
- `super.variable` — accesses parent's variable (if hidden by child's variable)

```java
class Vehicle {
    String type = "Vehicle";

    void display() { System.out.println("Vehicle"); }
}

class Car extends Vehicle {
    String type = "Car";  // hides parent's 'type' field

    void display() {
        System.out.println(super.type);   // "Vehicle" — parent field
        System.out.println(this.type);    // "Car"     — own field
        super.display();                  // calls Vehicle's display()
    }
}
```

**`this()` vs `super()`:**
- `this()` — calls another constructor in the **same** class
- `super()` — calls the **parent** class constructor
- Both must be the **first line** in a constructor — they cannot both appear

---

## Polymorphism

**Definition:** The ability of an object to take multiple forms.

### Compile-Time Polymorphism — Method Overloading

Same method name, different parameter list (type, count, or order). Resolved by compiler.

```java
class Calculator {
    int add(int a, int b)             { return a + b; }
    double add(double a, double b)    { return a + b; }
    int add(int a, int b, int c)      { return a + b + c; }
}
```

**Rules:**
- Same method name, different parameter list
- Return type **alone** is insufficient for overloading
- Resolved at **compile time** based on method signature

### Runtime Polymorphism — Method Overriding

Child class redefines a method from the parent class. Resolved at runtime.

```java
class Shape {
    void draw() { System.out.println("Drawing a shape"); }
}

class Circle extends Shape {
    @Override
    void draw() { System.out.println("Drawing a circle"); }
}

Shape s = new Circle();  // reference = Shape, object = Circle
s.draw();                // "Drawing a circle" — resolved at runtime
```

**Overriding Rules:**
- Same method name AND same parameter list
- Child's method must have same or **wider** access modifier
- Cannot override `static`, `final`, or `private` methods
- `@Override` annotation — optional but catches errors at compile time

### Dynamic Method Dispatch

Method call resolved at **runtime** based on the **actual object type**, not the reference type.

```java
Animal a = new Dog();  // reference = Animal, object = Dog
a.sound();             // Dog's sound() is called — dynamic dispatch
```

This enables writing flexible, polymorphic code — swap implementations without changing calling code. Core to Spring Boot's `@Service` beans and interface-based DI.

---

## `final` Keyword

| Usage | Meaning |
|---|---|
| `final int x = 10;` | Variable — constant, cannot be reassigned |
| `final void method() {}` | Method — cannot be overridden in subclass |
| `final class MyClass {}` | Class — cannot be subclassed |

```java
final class ImmutableClass {}
// class Sub extends ImmutableClass {}  // compile error

class Parent {
    final void display() {}
}
class Child extends Parent {
    // void display() {}  // compile error — cannot override final method
}
```

`String` is a `final` class — that's why strings are immutable and can't be subclassed.

---

## Object Class

Every class in Java implicitly extends `java.lang.Object`.

```java
class MyClass {}
// equivalent to: class MyClass extends Object {}
```

**Commonly Used Methods from Object:**
```java
obj.toString()     // string representation — override for meaningful logging
obj.equals(other)  // value equality — override along with hashCode()
obj.hashCode()     // integer hash — must be consistent with equals()
obj.getClass()     // returns runtime Class object
```

**Best practice:** When you override `equals()`, always override `hashCode()` too — required for correct behavior in `HashMap`, `HashSet`.

---

## Object Typecasting

### Upcasting (Child → Parent Reference) — Automatic, Always Safe
```java
Dog d = new Dog();
Animal a = d;  // implicit — always safe; Dog IS-A Animal
```

### Downcasting (Parent → Child Reference) — Manual, May Fail
```java
Animal a = new Dog();
Dog d = (Dog) a;  // explicit cast — safe only if actual object IS a Dog

// Check before casting to avoid ClassCastException:
if (a instanceof Dog) {
    Dog safeD = (Dog) a;
}
```

`ClassCastException` is thrown at runtime if the actual object is not of the target type.

---

## Wrapper Classes

Convert **primitives** to **objects** — needed for generics, collections, Optional, etc.

| Primitive | Wrapper |
|---|---|
| `int` | `Integer` |
| `double` | `Double` |
| `float` | `Float` |
| `long` | `Long` |
| `char` | `Character` |
| `boolean` | `Boolean` |

### Autoboxing and Unboxing
```java
Integer obj = 10;      // autoboxing — compiler converts int → Integer
int x = obj;           // unboxing  — compiler converts Integer → int

List<Integer> list = new ArrayList<>();
list.add(5);           // autoboxing happens automatically
int val = list.get(0); // unboxing happens automatically
```

Relevant when working with `Optional<Integer>`, `List<Long>`, Spring Data return types, etc.

---

## Abstraction

**Definition:** Hiding implementation details and showing only the essential functionality.

**Achieved via:**
1. Abstract classes (0–100% abstraction)
2. Interfaces (100% abstraction by design)

### Abstract Class

```java
abstract class Shape {
    String color;

    abstract void draw();           // no body — subclass MUST implement

    void setColor(String c) {       // concrete method — can have implementation
        this.color = c;
    }
}

class Circle extends Shape {
    @Override
    void draw() {
        System.out.println("Drawing circle");
    }
}
```

**Rules:**
- Declared with `abstract` keyword
- Can have **abstract** (no body) and **concrete** (with body) methods
- **Cannot be instantiated** directly (`new Shape()` → compile error)
- Subclass must implement all abstract methods (or itself be abstract)
- Can have constructor — called via `super()` from child class

---

## Interfaces

**Definition:** A contract (blueprint) that defines **what** a class must do, not **how**.

```java
interface Flyable {
    void fly();                 // public abstract by default
    int MAX_ALTITUDE = 10000;   // public static final by default
}

class Airplane implements Flyable {
    public void fly() {
        System.out.println("Airplane flying");
    }
}
```

**Rules:**
- All methods are `public abstract` by default (pre-Java 8)
- All variables are `public static final` by default
- No constructor
- A class can implement **multiple** interfaces (unlike class extends)
- An interface can extend multiple interfaces
- Java 8+: `default` and `static` methods (with body) allowed
- Java 9+: `private` methods allowed in interface

### Class → Interface Relationships
```
Class    extends     Class        (single only)
Class    implements  Interface    (multiple allowed)
Interface extends   Interface    (multiple allowed)
```

### Default Method (Java 8+)
```java
interface Printer {
    void print();

    default void log() {
        System.out.println("Default logging behavior");
    }
}
```

### Abstract Class vs Interface

| | Abstract Class | Interface |
|---|---|---|
| Instantiation | ❌ | ❌ |
| Constructor | ✅ | ❌ |
| Concrete methods | ✅ | ✅ (default/static, Java 8+) |
| Variables | Any type | `public static final` only |
| Multiple inheritance | ❌ (single class extend) | ✅ |
| When to use | Shared base implementation + state | Pure contract / capability |

---

## Interface Types — Complete Reference

### 1. Normal Interface

Defines a contract for classes to implement. Used for abstraction, loose coupling, multiple implementations.

```java
interface PaymentService {
    void pay(double amount);
    boolean refund(String transactionId);
}

class StripePaymentService implements PaymentService {
    public void pay(double amount) { /* Stripe API call */ }
    public boolean refund(String transactionId) { return true; }
}

class PayPalService implements PaymentService {
    public void pay(double amount) { /* PayPal API call */ }
    public boolean refund(String transactionId) { return false; }
}
```

### 2. Functional Interface (Java 8+)

Exactly **one abstract method** — designed for lambda expressions.

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
    // Adding a second abstract method here = compile error
}

Calculator add      = (a, b) -> a + b;
Calculator multiply = (a, b) -> a * b;

System.out.println(add.calculate(5, 3));       // 8
System.out.println(multiply.calculate(5, 3));  // 15
```

**Built-in Functional Interfaces (`java.util.function`):**
| Interface | Method | Use |
|---|---|---|
| `Predicate<T>` | `test(T t)` | boolean condition — used in `filter()` |
| `Function<T,R>` | `apply(T t)` | transform T to R — used in `map()` |
| `Consumer<T>` | `accept(T t)` | consume T, no return — used in `forEach()` |
| `Supplier<T>` | `get()` | produce T, no input — used in `orElseGet()` |
| `BiFunction<T,U,R>` | `apply(T,U)` | two inputs, one output |

### 3. Marker Interface

Empty interface — tags a class for special JVM or framework behavior.

```java
interface Marker {}   // no methods
class Test implements Marker {}
```

The JVM or framework checks `instanceof MarkerInterface` at runtime and provides special behavior.

**Built-in Examples:**
| Interface | Behavior Granted |
|---|---|
| `Serializable` | Object can be serialized to byte stream (files, network, Redis) |
| `Cloneable` | `Object.clone()` performs field-for-field copy |
| `RandomAccess` | Signals O(1) indexed access; used by `Collections` algorithms to pick optimal strategy |

```java
class User implements Serializable {
    private String name;
    private int age;
}
// User objects can now be serialized — stored in Redis, sent over network, written to file
```

**Key Points:**
- No methods to implement — just a tag
- JVM/framework checks via `instanceof` at runtime
- Modern alternative: annotations (`@Entity`, `@Transactional`) — more flexible, but marker interfaces still exist widely in Java APIs

### 4. Nested Interface

Interface defined inside a class or another interface.

```java
class PaymentProcessor {
    interface Validator {
        boolean validate(double amount);
    }
}

class CardValidator implements PaymentProcessor.Validator {
    public boolean validate(double amount) {
        return amount > 0;
    }
}
```

Used for grouping related contracts and builder patterns.

---

## Loose Coupling Using Interfaces

```java
// Define contract
interface UserService {
    User getUser(Long id);
    User createUser(CreateUserRequest req);
}

// Provide implementation
@Service
class UserServiceImpl implements UserService {
    @Autowired
    private UserRepository repo;

    public User getUser(Long id) {
        return repo.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User not found: " + id));
    }
    // ...
}

// Controller depends on interface, NOT implementation
@RestController
class UserController {
    private final UserService userService;

    UserController(UserService userService) {
        this.userService = userService;
    }
}
```

**Benefits:**
- **Loose coupling** — `UserController` depends on `UserService` contract, not `UserServiceImpl`
- **Easy testing** — mock the interface: `Mockito.mock(UserService.class)`
- **Multiple implementations** — `CachedUserService`, `ReadOnlyUserService` — swap without changing callers

---

## Inner Classes

A class defined **inside another class**.

### 1. Normal (Non-Static) Inner Class
```java
class Outer {
    int x = 10;

    class Inner {
        // static members NOT allowed in regular inner class
        void show() {
            System.out.println(x);  // can access outer class member directly
        }
    }
}

Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();  // requires outer object
inner.show();
```
- Has two `this` pointers: `this` (own), `Outer.this` (outer class)
- `this$0` internally stores the address of the outer object

### 2. Static Inner Class
```java
class Outer {
    static class StaticInner {
        void show() { System.out.println("Static inner"); }
    }
}

Outer.StaticInner si = new Outer.StaticInner();  // NO outer object needed
```

### 3. Method Local Inner Class
```java
void method() {
    class LocalClass {
        void show() { System.out.println("Local"); }
    }
    LocalClass lc = new LocalClass();
    lc.show();
}
```
Object can only be created inside the method where it's defined.

### 4. Anonymous Inner Class
Created on-the-fly, on an object of a class or interface. The parent is the class/interface on whose object it's created.

```java
Flyable f = new Flyable() {  // anonymous class implementing Flyable
    public void fly() {
        System.out.println("Anonymous flying");
    }
};
f.fly();
```

Commonly replaced by **lambda expressions** (Java 8+) for functional interfaces (single-method interfaces).

---

## Encapsulation vs Composition — Design Comparison

These are often confused because both involve class structure, but they solve different problems.

### Encapsulation — Data Hiding (Within One Class)

```java
class Student {
    private int id;          // hide state

    public void setId(int id) { this.id = id; }    // controlled write
    public int getId() { return id; }               // controlled read
}
```

- Focuses on **protecting internal state** from direct access
- Controls what callers can read/write
- Answers: "How is this class's data protected?"

### Composition — HAS-A Relationship (Between Classes)

```java
class Engine {
    void start() { System.out.println("Engine started"); }
}

class Car {
    Engine engine = new Engine();  // Car HAS-A Engine — composition

    void drive() {
        engine.start();
        System.out.println("Car driving");
    }
}
```

- Focuses on **building behavior** by combining objects
- One class delegates responsibility to another
- Answers: "How are objects assembled?"

### Summary Table

| | Encapsulation | Composition |
|---|---|---|
| Purpose | Data hiding / access control | Code reuse / delegation |
| Relationship | Within a single class | Between classes (HAS-A) |
| Implemented by | `private` fields + getters/setters | Reference field to another class |
| Answers | "How is data protected?" | "How are objects assembled?" |

**In Spring Boot:**
- Encapsulation → `User` entity has `private String passwordHash` — no getter in response DTO
- Composition → `OrderService` has `PaymentGateway`, `InventoryService` — complex behavior assembled from smaller services

### Why Composition Over Inheritance?
- **Flexible** — swap behavior at runtime (different `PaymentGateway` impl for test vs prod)
- **Loose coupling** — `Car` doesn't know `Engine` internals
- **Easier to test** — mock the dependency (`new Car(mockEngine)`)
- **Avoids fragile base class problem** — parent changes don't unexpectedly break children
- In microservices: services composed of repos and HTTP clients are easier to evolve independently

---

## Method Hiding

Occurs when a **static method** in a child class has the **same signature** as a static method in the parent class. The child's method *hides* (does not override) the parent's.

```java
class Parent {
    static void show() {
        System.out.println("Parent static method");
    }
}

class Child extends Parent {
    static void show() {  // hides, does NOT override
        System.out.println("Child static method");
    }
}

Parent obj = new Child();
obj.show();  // "Parent static method" — resolved by REFERENCE type, not object type
```

### Method Hiding vs Method Overriding

| | Method Overriding | Method Hiding |
|---|---|---|
| Applies to | Instance methods | Static methods |
| Resolution | Runtime (actual object type) | Compile-time (reference type) |
| Polymorphism | ✅ Yes | ❌ No |
| `@Override` usable | ✅ Yes | ❌ No |

**Key Rule:** Static methods don't participate in runtime polymorphism. The "method" called depends on the **reference type**, not the actual object. This is why calling static methods via object references (`obj.method()`) is misleading — always use class name (`ClassName.method()`).

---

## Interview Q&A

**Q: Types of inheritance in Java?**
Single (A → B), Multilevel (A → B → C), Hierarchical (A → B, A → C) — all supported. Multiple via class not supported (diamond problem). Multiple via interfaces is supported — conflicts in `default` methods must be explicitly resolved.

**Q: Why is multiple inheritance via class not supported?**
Diamond problem — if B and C both extend A and both override `method()`, and D extends B and C, the compiler can't determine which `method()` to use. Java avoids this for classes but allows it for interfaces (with forced resolution).

**Q: What is dynamic method dispatch?**
JVM resolves method calls at runtime based on the actual object type, not the reference type. `Animal a = new Dog(); a.sound()` calls Dog's `sound()`. Enables polymorphism — the calling code doesn't need to know the concrete type.

**Q: `final` keyword uses?**
Variable → constant (can't reassign). Method → can't override in subclass. Class → can't extend. `String` is `final` — core reason Java Strings are immutable.

**Q: Abstract class vs interface?**
Abstract class: can have state, constructor, concrete + abstract methods — use for shared base implementation.
Interface: pure contract, no state/constructor, 100% abstract (Java 8+ allows `default` methods) — use for capabilities and multiple inheritance.

**Q: What are the 4 types of interfaces?**
1. Normal — contract with abstract methods
2. Functional — exactly one abstract method; used with lambdas (`@FunctionalInterface`)
3. Marker — empty, tags class for JVM/framework behavior (`Serializable`)
4. Nested — defined inside class/interface for grouping related contracts

**Q: What is a marker interface? How does it work?**
Empty interface used to tag a class. JVM/framework checks `instanceof MarkerInterface` at runtime and enables special behavior. `Serializable` → serialization allowed. `Cloneable` → `clone()` works. Modern Java uses annotations instead, but marker interfaces are still prevalent.

**Q: What is upcasting vs downcasting?**
Upcasting (child → parent ref): implicit, always safe.
Downcasting (parent ref → child): explicit cast, may throw `ClassCastException`. Always check with `instanceof` before downcasting.

**Q: What are wrapper classes? Autoboxing?**
Wrapper classes box primitives as objects — needed for generics, collections, `Optional`. Autoboxing = compiler auto-converts `int ↔ Integer`. Relevant when adding ints to `List<Integer>`, using `Optional<Long>`, Spring Data numeric return types.

**Q: How do interfaces enable loose coupling in Spring Boot?**
Define `UserService` interface → inject it everywhere → Spring wires `UserServiceImpl` as the bean. Swap impl for tests: `Mockito.mock(UserService.class)`. Controllers never know about `UserServiceImpl` — only the interface contract. This is the standard Spring service pattern.

**Q: Encapsulation vs composition?**
Encapsulation = data hiding in one class. Composition = assembling behavior from multiple classes (HAS-A). Encapsulation answers "how is data protected?"; composition answers "how are objects assembled?".

**Q: Why is composition preferred over inheritance?**
Flexible (swap behavior at runtime), loosely coupled (no tight parent-child binding), easier to test (mock dependencies), avoids fragile base class problem. In service-layer code: `OrderService` composed of `PaymentGateway` and `InventoryService` can swap either dependency in tests.

**Q: What is method hiding? How is it different from overriding?**
Method hiding applies to static methods — child redefines a static method with the same signature, hiding the parent's version. Resolution is compile-time based on reference type. Overriding applies to instance methods — resolution is runtime based on actual object. Static methods have no polymorphism.

**Q: `this()` vs `super()`?**
`this()` calls another constructor in the same class (chaining). `super()` calls the parent class constructor. Both must be first line in constructor — they can't coexist in the same constructor.
