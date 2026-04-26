# Java Language Basics

> **Quick Reference:** Java is a high-level, object-oriented, platform-independent programming language (Sun Microsystems, now Oracle) that follows "Write Once, Run Anywhere (WORA)" — the same bytecode runs on any OS with a JVM.

---

## Key Features of Java

- **Platform Independent** — Write Once, Run Anywhere (WORA)
- **Object-Oriented** — Everything is modelled as objects and classes
- **Secure and Robust** — Strong type-checking, exception handling, no pointers
- **Multithreaded** — Built-in support for concurrent execution
- **Distributed** — Supports networking and distributed computing natively

---

## Java Architecture: JDK, JRE, JVM

### JDK (Java Development Kit)
Full package for **developing** Java applications.
- Contains: JRE + Compiler (`javac`) + Debugger (`jdb`) + Packaging tool (`jar`)
- Use when: writing and compiling Java code

### JRE (Java Runtime Environment)
Required to **run** Java programs.
- Contains: JVM + Core libraries + Runtime components
- Use when: deploying/running an app (no development tools needed)

### JVM (Java Virtual Machine)
The engine that **executes** Java bytecode.
- Converts bytecode → machine-specific code
- Manages memory (Heap, Stack, Method Area)
- Handles Garbage Collection
- Provides security sandbox

```
JDK  ⊃  JRE  ⊃  JVM
```

| Component | Role |
|---|---|
| JDK | Develop Java programs |
| JRE | Run Java programs |
| JVM | Execute bytecode |

---

## Java Program Execution Flow

```
1. Write source code      →  HelloWorld.java
2. Compile with javac     →  HelloWorld.class  (bytecode)
3. JVM loads .class file
4. Interpreter / JIT compiles bytecode → machine code
5. OS executes machine code
```

**Bytecode** — Intermediate, platform-independent code stored in `.class` files. The JVM on any OS can execute the same bytecode.

**JIT Compiler (Just-In-Time)** — Converts frequently-used bytecode to native machine code at runtime, dramatically improving performance. This is why Java warms up over time — important in Spring Boot microservices where the first requests are slower than subsequent ones.

---

## Hello World

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

- `public` — access modifier; class is accessible everywhere
- `class` — keyword to define a class
- `main()` — entry point of every Java program
- `System.out.println()` — prints to standard output with newline

---

## JVM Memory Areas

| Area | What's Stored | GC Managed |
|---|---|---|
| **Heap** | Objects, arrays | Yes |
| **Stack** | Local variables, primitives, method frames | No (auto-cleaned) |
| **Method Area / Metaspace** | Class metadata, static variables | Partial |
| **PC Register** | Current instruction address per thread | No |

**Heap vs Stack:**
- **Heap:** Shared across all threads; objects live here; GC reclaims unreachable objects
- **Stack:** Thread-specific; holds local variables and method call frames; auto-cleaned when method returns

**JVM Tuning (Spring Boot microservices):**
```bash
-Xms512m   # initial heap size
-Xmx2g     # max heap size
-XX:+UseG1GC  # G1 GC for balanced throughput + latency
```

---

## Primitive Data Types (8 Total)

| Type | Size | Range | Example |
|---|---|---|---|
| `byte` | 1 byte | -128 to 127 | `byte b = 100;` |
| `short` | 2 bytes | -32,768 to 32,767 | `short s = 20000;` |
| `int` | 4 bytes | -2³¹ to 2³¹-1 | `int num = 50000;` |
| `long` | 8 bytes | -2⁶³ to 2⁶³-1 | `long big = 9999999999L;` |
| `float` | 4 bytes | ~7 decimal digits | `float price = 99.99f;` |
| `double` | 8 bytes | ~15 decimal digits | `double pi = 3.14159;` |
| `char` | 2 bytes | 0 to 65,535 (Unicode) | `char grade = 'A';` |
| `boolean` | JVM-dep. | true / false | `boolean flag = true;` |

**Key Rules:**
- `int` is the default integer type
- `double` is the default floating-point type
- `char` uses Unicode (2 bytes, unlike C/C++'s 1 byte)
- `boolean` size is JVM-dependent (not fixed in spec)
- All primitives are stored in **stack memory**

---

## Primitive vs Non-Primitive

| Feature | Primitive | Non-Primitive |
|---|---|---|
| Stores | Actual value | Reference (address) |
| Memory | Stack | Heap (object), Stack (reference) |
| Default value | `0`, `false`, etc. | `null` |
| Examples | `int`, `double`, `char` | `String`, `int[]`, `ArrayList` |

---

## Operators

### Arithmetic Operators
```java
int a = 10, b = 3;
System.out.println(a + b);  // 13
System.out.println(a - b);  // 7
System.out.println(a * b);  // 30
System.out.println(a / b);  // 3  (integer division — truncates)
System.out.println(a % b);  // 1  (modulus/remainder)
```

### Unary Operators
```java
int x = 5;
x++;   // post-increment → x becomes 6
++x;   // pre-increment  → increments before use
x--;   // post-decrement
--x;   // pre-decrement
```

### Relational (Comparison) Operators
```java
int a = 10, b = 20;
System.out.println(a == b);  // false
System.out.println(a != b);  // true
System.out.println(a > b);   // false
System.out.println(a < b);   // true
System.out.println(a >= b);  // false
System.out.println(a <= b);  // true
```

### Logical Operators
```java
boolean result  = (10 > 5) && (5 < 3);  // false  — AND
boolean result2 = (10 > 5) || (5 < 3);  // true   — OR
boolean result3 = !(10 > 5);            // false  — NOT
```
`&&` and `||` are **short-circuit** operators — second operand is not evaluated if result is already determined by the first.

### Assignment Operators
```java
int a = 10;
a += 5;   // a = 15
a -= 3;   // a = 12
a *= 2;   // a = 24
a /= 4;   // a = 6
a %= 4;   // a = 2
```

### Bitwise Operators
```java
int a = 5;  // 0101
int b = 3;  // 0011
System.out.println(a & b);   // 1   (AND)
System.out.println(a | b);   // 7   (OR)
System.out.println(a ^ b);   // 6   (XOR)
System.out.println(~a);      // -6  (NOT)
System.out.println(a << 1);  // 10  (left shift — multiply by 2)
System.out.println(a >> 1);  // 2   (right shift — divide by 2)
```

### Ternary Operator
```java
int a = 10, b = 20;
int max = (a > b) ? a : b;  // 20 — shorthand for if-else; returns a value
```

---

## Type Conversion & Casting

### Implicit Casting (Widening) — Automatic
Smaller type → Larger type. No data loss. No cast syntax needed.

**Hierarchy:** `byte → short → int → long → float → double`

```java
int a = 10;
double d = a;  // int → double, automatic
```

### Explicit Casting (Narrowing) — Manual
Larger type → Smaller type. **Data loss possible.** Cast syntax required.

```java
double d = 10.9;
int a = (int) d;  // 10 — decimal truncated, NOT rounded
```

```java
// Full example
int x = 100;
long y = x;         // widening — automatic
double z = 10.9;
int w = (int) z;    // narrowing — explicit cast, w = 10
```

**Key Points:**
- Widening is safe and automatic
- Narrowing requires explicit cast `(dataType)`
- Narrowing **truncates**, does NOT round

---

## Control Statements

### Decision Making

**if:**
```java
int age = 18;
if (age >= 18) {
    System.out.println("Eligible to vote");
}
```

**if-else:**
```java
int num = 5;
if (num % 2 == 0) {
    System.out.println("Even");
} else {
    System.out.println("Odd");
}
```

**else-if ladder:**
```java
int marks = 85;
if (marks >= 90) {
    System.out.println("A");
} else if (marks >= 75) {
    System.out.println("B");
} else {
    System.out.println("C");
}
```

**switch:**
```java
int day = 2;
switch (day) {
    case 1: System.out.println("Monday"); break;
    case 2: System.out.println("Tuesday"); break;
    default: System.out.println("Invalid");
}
```
- Works with: `int`, `char`, `String`, `enum`
- `break` prevents fall-through to next case
- `default` is optional but recommended

### Looping Statements

**for** — when iteration count is known:
```java
for (int i = 0; i < 5; i++) {
    System.out.println(i);
}
```

**while** — condition-based, may not execute:
```java
int i = 0;
while (i < 5) {
    System.out.println(i);
    i++;
}
```

**do-while** — executes **at least once**, then checks condition:
```java
int i = 0;
do {
    System.out.println(i);
    i++;
} while (i < 5);
```

### Jump Statements

**break** — exits the loop/switch immediately:
```java
for (int i = 0; i < 5; i++) {
    if (i == 3) break;
    System.out.println(i);  // prints 0, 1, 2
}
```

**continue** — skips current iteration, moves to next:
```java
for (int i = 0; i < 5; i++) {
    if (i == 2) continue;
    System.out.println(i);  // prints 0, 1, 3, 4
}
```

**return** — exits method execution; returns value if non-void.

---

## Arrays

### What is an Array?
A **fixed-size** collection of elements of the **same data type** stored in contiguous memory.

```java
int[] arr = new int[5];          // declare + allocate (defaults to 0)
int[] nums = {1, 2, 3, 4};      // declare + initialize with values
```

**Key Rules:**
- Array is a **dynamic object** — gets memory on heap
- If dimensions given → don't provide values; if values given → don't provide dimensions
- Index starts from **0**
- Fixed size — cannot grow/shrink after creation
- Reference stored in stack; array object stored in heap

### Accessing & Iterating
```java
int[] arr = {10, 20, 30, 40, 50};
System.out.println(arr[0]);      // 10
System.out.println(arr.length);  // 5

// for loop
for (int i = 0; i < arr.length; i++) {
    System.out.println(arr[i]);
}

// for-each loop (read-only)
for (int num : arr) {
    System.out.println(num);
}
```

### Arrays Utility Class
```java
import java.util.Arrays;

int[] arr = {5, 2, 8, 1};
Arrays.sort(arr);                          // sort in-place → [1, 2, 5, 8]
System.out.println(Arrays.toString(arr));  // "[1, 2, 5, 8]"
Arrays.fill(arr, 0);                       // fill all elements with 0
boolean equal = Arrays.equals(arr1, arr2); // element-by-element comparison
int idx = Arrays.binarySearch(arr, 5);     // requires sorted array
```

### Taking Array Input
```java
Scanner sc = new Scanner(System.in);
int[] arr = new int[3];
for (int i = 0; i < 3; i++) {
    arr[i] = sc.nextInt();
}
```

### Internal Working
Arrays use contiguous memory; index access is O(1):
```
address = base_address + (index × element_size)
```
This makes arrays the fastest data structure for random access — no traversal needed.

---

## String

### What is a String?
A **sequence of characters** stored as an immutable object.

```java
String str = "Java";             // literal — stored in String Constant Pool (SCP)
String obj = new String("Java"); // object  — stored in heap (outside SCP)
```

### String Constant Pool (SCP)
A special memory area **inside the heap** that stores unique string literals. If `"hello"` already exists, a new literal `"hello"` reuses the same object — memory optimization.

```java
String s1 = "Java";
String s2 = "Java";
// s1 and s2 point to the SAME object in SCP
```

```java
String s1 = "Java";
String s3 = new String("Java");  // forces new heap object, bypasses SCP
System.out.println(s1 == s3);     // false — different addresses
System.out.println(s1.equals(s3)); // true — same value
```

### Why String is Immutable
Once created, a String object's content **cannot be changed**. Any modification creates a new String object.

```java
String s = "Hello";
s.concat(" World");   // creates a NEW string; s still points to "Hello"
System.out.println(s); // "Hello"
```

**Why immutable?**
1. **Security** — safe for JWT tokens, URLs, DB connection strings (can't be tampered with)
2. **Thread safety** — shared across threads without synchronization
3. **Memory optimization** — SCP safely reuses the same object
4. **Hashcode caching** — efficient HashMap/HashSet key (hashcode computed once)

### `==` vs `.equals()`
- `==` → compares **references** (memory addresses)
- `.equals()` → compares **values** (actual character content)

```java
String s1 = "java";
String s2 = new String("java");
System.out.println(s1 == s2);      // false — different objects
System.out.println(s1.equals(s2)); // true  — same value
```

**Always use `.equals()` for String value comparison.**

### Common String Methods
```java
String str = "Hello World";

str.length();                        // 11
str.charAt(0);                       // 'H'
str.substring(6);                    // "World"
str.substring(0, 5);                 // "Hello"
str.toUpperCase();                   // "HELLO WORLD"
str.toLowerCase();                   // "hello world"
str.trim();                          // removes leading/trailing spaces
str.replace('o', 'u');               // "Hellu Wurld"
str.contains("World");               // true
str.startsWith("Hello");             // true
str.endsWith("World");               // true
str.equals("Hello World");           // true
str.equalsIgnoreCase("hello world"); // true
str.split(" ");                      // ["Hello", "World"]
str.toCharArray();                   // char[]
str.concat(" Java");                 // "Hello World Java"
str.intern();                        // moves to SCP if not already there

str1.compareTo(str2);          // 0 if equal, negative if str1 < str2, positive if str1 > str2
str1.compareToIgnoreCase(str2);
```

### Hashcode & IdentityHashCode
```java
str.hashCode();                    // unique int based on content (for HashMap keys)
System.identityHashCode(str);      // unique int based on object identity
```
Note: These are NOT raw memory addresses. Java never exposes raw addresses.

### Classic Interview Example
```java
String s1 = "Java";
String s2 = "Java";
String s3 = new String("Java");

System.out.println(s1 == s2);       // true  — same SCP object
System.out.println(s1 == s3);       // false — s3 is a heap object
System.out.println(s1.equals(s3));  // true  — same value
// How many objects created? 2: one in SCP, one in heap
```

---

## StringBuffer vs StringBuilder

### Why Needed?
`String` is immutable — every modification creates a new object. Inefficient for frequent string operations (building large JSON, log strings in loops, etc.).

### StringBuffer — Mutable + Thread-Safe
```java
StringBuffer sb = new StringBuffer("Hello");
sb.append(" World");    // modifies same object — no new object created
sb.reverse();
sb.length();
sb.capacity();          // default: 16 + initial string length
sb.ensureCapacity(35);
sb.trimToSize();
```
- **Synchronized** (thread-safe) — methods are `synchronized`
- **Slower** than StringBuilder due to synchronization overhead

### StringBuilder — Mutable + NOT Thread-Safe
```java
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");
sb.insert(5, ",");
sb.delete(0, 5);
sb.reverse();
```
- **Not synchronized** — faster
- Preferred for single-threaded contexts (most service methods)

### When to Use Which

| Scenario | Use |
|---|---|
| Constant/small string | `String` |
| High-performance, single-threaded | `StringBuilder` |
| Multi-threaded string building | `StringBuffer` |

**In microservices:** Use `StringBuilder` for building large JSON payloads, log messages, or SQL query strings inside a single-threaded service method.

```java
// BAD — creates ~1000 intermediate String objects → GC pressure
String s = "";
for (int i = 0; i < 1000; i++) {
    s += i;
}

// GOOD — single object modified in-place
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

---

## Input and Output Handling

### Standard Streams
- `System.out` → standard output (monitor) — type `PrintStream`
- `System.in` → standard input (keyboard) — type `InputStream`
- `System.err` → standard error output (monitor, for error messages)

```java
// System is a predefined class in java.lang
// out is a static variable of type PrintStream
// println() is a method on PrintStream
System.out.println("Output here");
```

### Using BufferedReader / InputStreamReader
```java
import java.io.*;

InputStreamReader isr = new InputStreamReader(System.in);
BufferedReader br = new BufferedReader(isr);

String str = br.readLine();                    // reads a full line
int num = Integer.parseInt(br.readLine());     // parse line as int
float avg = Float.parseFloat(br.readLine());   // parse line as float
char a = (char) br.read();                     // reads a single character
br.skip(1);                                    // skip 1 character from buffer
```

### Using Scanner (Most Common)
```java
import java.util.Scanner;

Scanner sc = new Scanner(System.in);

int num      = sc.nextInt();     // reads integer
double d     = sc.nextDouble();  // reads double
String word  = sc.next();        // reads single word (stops at whitespace)
String line  = sc.nextLine();    // reads full line including spaces

sc.close();  // prevent resource leak
```

**Common Scanner Methods:**
| Method | Reads |
|---|---|
| `nextInt()` | int |
| `nextDouble()` | double |
| `nextFloat()` | float |
| `nextLong()` | long |
| `next()` | single word (whitespace delimiter) |
| `nextLine()` | entire line including spaces |

**Gotcha:** `nextLine()` after `nextInt()` reads the leftover newline character. Fix: add a dummy `sc.nextLine()` to consume it.
```java
int age = sc.nextInt();
sc.nextLine();          // consume leftover newline
String name = sc.nextLine();
```

### Scanner vs BufferedReader
- `Scanner` — convenient, parses types directly, slightly slower
- `BufferedReader` — reads raw lines, faster, needs manual `Integer.parseInt()` etc.

---

## Interview Q&A

**Q: What is Java?**
A high-level, object-oriented, platform-independent programming language following WORA — the same `.class` bytecode file runs on any OS that has a JVM.

**Q: Why is Java platform-independent?**
Java compiles to bytecode (`.class`), not OS-specific machine code. The JVM on each OS translates bytecode to native code at runtime. Same bytecode, different JVMs per OS → platform independence.

**Q: JVM vs JRE vs JDK?**
- JVM: Executes bytecode, manages memory, handles GC
- JRE: JVM + core libraries — sufficient to run Java apps (no dev tools)
- JDK: JRE + compiler (`javac`) + debugger (`jdb`) + packaging tool (`jar`) — needed to develop

**Q: What is bytecode?**
Intermediate, platform-independent instruction set in `.class` files, generated by `javac`. Not OS-readable directly — JVM interprets or JIT-compiles it to native code.

**Q: What is the JIT compiler?**
Just-In-Time compiler converts hot (frequently used) bytecode to native machine code at runtime. Compiled paths are cached → faster execution over time. This is why Spring Boot apps warm up — early requests are slower than later ones under load.

**Q: Where is data stored in memory?**
- Stack: local variables, primitives
- Heap: objects, arrays
- Method Area/Metaspace: class metadata, static variables

**Q: What are the OOP principles?**
1. **Encapsulation** — private fields + public getters/setters
2. **Inheritance** — `extends` for code reuse
3. **Polymorphism** — overloading (compile-time), overriding (runtime)
4. **Abstraction** — abstract classes, interfaces

**Q: What is garbage collection?**
JVM's automatic memory management. Objects unreachable from any live reference become eligible for GC. JVM reclaims heap space — prevents memory leaks without manual `free()`/`delete` like C/C++.

**Q: Why is Java used in microservices?**
Strong ecosystem (Spring Boot, Quarkus), JVM performance with JIT, mature concurrency model, vast library support, strong typing for large codebases, excellent tooling (IntelliJ, Maven, Gradle).

**Q: What is an array? How is it fast?**
Fixed-size, same-type collection in contiguous memory. Fast O(1) random access because address arithmetic: `address = base + index × size`. No traversal needed — direct jump to position.

**Q: What is String? What is SCP?**
String is an immutable sequence of characters. SCP (String Constant Pool) is a special heap area caching unique string literals — `"hello"` used 100 times creates only 1 SCP object, saving memory.

**Q: Why is String immutable?**
Security (safe for tokens/URLs), thread safety (no sync needed for shared strings), memory optimization (SCP reuse), efficient HashMap keys (hashcode cached after first computation).

**Q: `==` vs `.equals()` for String?**
`==` compares memory addresses. `.equals()` compares actual character content. Always use `.equals()` for value comparison.

**Q: StringBuilder vs StringBuffer?**
`StringBuilder`: mutable, not thread-safe, faster — use in single-threaded service methods.
`StringBuffer`: mutable, thread-safe (synchronized), slower — use when multiple threads build the same string.
`String`: immutable — use for small/constant strings.

**Q: Why avoid `String +=` in loops?**
Each `+=` creates a new String object. 1000 iterations → ~1000 temporary objects → GC pressure. Use `StringBuilder.append()` — modifies one object in-place.

**Q: Where do you use these in real projects?**
- Arrays → batch data processing, fixed-size buffers
- String → API request/response payloads, JWT token parsing, Redis keys
- StringBuilder → constructing JSON logs, SQL query building dynamically
- SCP behavior → memory optimization awareness in high-load services
