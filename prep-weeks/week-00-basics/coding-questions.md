# Java Practice Coding Bank — Fundamentals

> Try each problem yourself before reading the **Approach** and solution. Solutions are Java 17, concise, and compilable. Mixed types: **implement X**, **predict the output**, and **fix the bug**.

---

## Syntax & Control Flow

### P1. FizzBuzz *(easy)*
**Problem:** Print 1..n. For multiples of 3 print "Fizz", of 5 "Buzz", of both "FizzBuzz".
**Approach:** Check 15 first (or build a string), else 3, else 5, else the number.
```java
static void fizzBuzz(int n) {
    for (int i = 1; i <= n; i++) {
        String out = "";
        if (i % 3 == 0) out += "Fizz";
        if (i % 5 == 0) out += "Buzz";
        System.out.println(out.isEmpty() ? Integer.toString(i) : out);
    }
}
```
**Key point:** Building the string avoids the bug of testing `i % 15` order-dependence; `isEmpty()` cleanly handles the "neither" case.

### P2. Count digits of an integer *(easy)*
**Problem:** Count the number of digits in a non-negative int (treat 0 as 1 digit).
**Approach:** Divide by 10 in a loop; handle 0 as a special case.
```java
static int countDigits(int n) {
    if (n == 0) return 1;
    int count = 0;
    while (n != 0) { n /= 10; count++; }
    return count;
}
```
**Key point:** Integer division truncates toward zero, so the loop terminates. Forgetting the `n == 0` guard returns 0 for input 0.

### P3. Sum of digits *(easy)*
**Problem:** Return the sum of digits of a non-negative int.
**Approach:** Repeatedly take `n % 10` and `n /= 10`.
```java
static int sumDigits(int n) {
    int sum = 0;
    while (n > 0) { sum += n % 10; n /= 10; }
    return sum;
}
```
**Key point:** `%` gives the last digit; `/` drops it. Standard digit-extraction idiom.

### P4. Predict the output — integer division *(easy)*
**Problem:** What is printed?
```java
System.out.println(7 / 2);
System.out.println(7.0 / 2);
System.out.println(7 / 2.0);
System.out.println((double) 7 / 2);
```
**Approach:** If both operands are int, result is int; one double operand promotes the whole expression.
```java
// Output:
// 3
// 3.5
// 3.5
// 3.5
```
**Key point:** `7 / 2` is integer division → truncates to 3. Any `double` operand forces floating-point math. The cast in the last line applies to `7` only, but that's enough to promote the division.

### P5. Predict the output — char arithmetic *(easy)*
**Problem:** What is printed?
```java
char c = 'A';
System.out.println(c + 1);
System.out.println((char) (c + 1));
```
**Approach:** `char` is promoted to `int` in arithmetic.
```java
// Output:
// 66
// B
```
**Key point:** `'A'` is 65. `c + 1` promotes to int → 66. Cast back to `char` to get the letter.

### P6. Predict the output — pre vs post increment *(medium)*
**Problem:** What is printed?
```java
int i = 5;
int a = i++;
int b = ++i;
System.out.println(a + " " + b + " " + i);
```
**Approach:** Post-increment returns the old value then increments; pre-increment increments then returns.
```java
// Output:
// 5 7 7
```
**Key point:** `i++` assigns 5 to `a` (i becomes 6). `++i` makes i 7 and assigns 7 to `b`.

### P7. Predict the output — switch fall-through *(medium)*
**Problem:** What is printed for `day = 2`?
```java
int day = 2;
switch (day) {
    case 1: System.out.println("Mon");
    case 2: System.out.println("Tue");
    case 3: System.out.println("Wed"); break;
    default: System.out.println("Other");
}
```
**Approach:** Without `break`, execution falls through to subsequent cases.
```java
// Output:
// Tue
// Wed
```
**Key point:** Matching `case 2` runs Tue, then falls through to Wed, where `break` stops it. Classic missing-break trap.

### P8. Check if a year is a leap year *(easy)*
**Problem:** Return true if `year` is a leap year.
**Approach:** Divisible by 4 and (not by 100 unless also by 400).
```java
static boolean isLeap(int year) {
    return (year % 4 == 0 && year % 100 != 0) || (year % 400 == 0);
}
```
**Key point:** The century rule (`% 100` exclusion, `% 400` inclusion) is what most naive solutions miss.

### P9. Predict the output — short-circuit evaluation *(medium)*
**Problem:** What is printed? Does `sideEffect()` run?
```java
static boolean sideEffect() { System.out.println("called"); return true; }

public static void main(String[] args) {
    if (false && sideEffect()) {}
    if (true  || sideEffect()) {}
    System.out.println("done");
}
```
**Approach:** `&&` skips the right side if left is false; `||` skips it if left is true.
```java
// Output:
// done
```
**Key point:** `sideEffect()` never runs — short-circuiting skips the right operand in both cases. Use `&` / `|` to force evaluation of both.

### P10. Print a multiplication table *(easy)*
**Problem:** Print the multiplication table for `n` from 1 to 10.
**Approach:** Single loop, format each line.
```java
static void table(int n) {
    for (int i = 1; i <= 10; i++) {
        System.out.println(n + " x " + i + " = " + (n * i));
    }
}
```
**Key point:** Note the parentheses around `n * i` — string concatenation is left-to-right, so `"= " + n * i` would still multiply first (`*` binds tighter than `+`), but parentheses make intent explicit and avoid the common `"a" + 1 + 2` → `"a12"` mistake.

---

## Strings

### P11. Reverse a string *(easy)*
**Problem:** Return the reverse of a string.
**Approach:** Use `StringBuilder.reverse()` (or loop from the end).
```java
static String reverse(String s) {
    return new StringBuilder(s).reverse().toString();
}
```
**Key point:** `String` is immutable; `StringBuilder` mutates in place and is the idiomatic reverse. A manual loop with `s.charAt(i)` works too.

### P12. Check palindrome *(easy)*
**Problem:** Return true if a string reads the same forward and backward (case-sensitive).
**Approach:** Two pointers from both ends.
```java
static boolean isPalindrome(String s) {
    int i = 0, j = s.length() - 1;
    while (i < j) {
        if (s.charAt(i++) != s.charAt(j--)) return false;
    }
    return true;
}
```
**Key point:** Two-pointer compare is O(n) time, O(1) space — better than reversing and comparing.

### P13. Check anagram *(medium)*
**Problem:** Return true if two strings are anagrams (same letters, any order).
**Approach:** Sort both char arrays and compare (or count frequencies).
```java
static boolean isAnagram(String a, String b) {
    if (a.length() != b.length()) return false;
    char[] x = a.toCharArray(), y = b.toCharArray();
    Arrays.sort(x);
    Arrays.sort(y);
    return Arrays.equals(x, y);
}
```
**Key point:** Length check first short-circuits unequal inputs. A frequency-count array (`int[26]`) is O(n) vs sort's O(n log n).

### P14. Count character frequency *(easy)*
**Problem:** Return a map of each character to its count.
**Approach:** Iterate chars, use `Map.merge` to increment.
```java
static Map<Character, Integer> charCount(String s) {
    Map<Character, Integer> counts = new HashMap<>();
    for (char c : s.toCharArray()) {
        counts.merge(c, 1, Integer::sum);
    }
    return counts;
}
```
**Key point:** `merge(key, 1, Integer::sum)` is the clean way to count — inserts 1 if absent, else adds 1 to the existing value.

### P15. First non-repeating character *(medium)*
**Problem:** Return the first character that appears exactly once, or `'\0'` if none.
**Approach:** Count frequencies in one pass, then find the first char with count 1. Use a LinkedHashMap to preserve insertion order.
```java
static char firstUnique(String s) {
    Map<Character, Integer> counts = new LinkedHashMap<>();
    for (char c : s.toCharArray()) counts.merge(c, 1, Integer::sum);
    for (var e : counts.entrySet()) {
        if (e.getValue() == 1) return e.getKey();
    }
    return '\0';
}
```
**Key point:** `LinkedHashMap` keeps first-seen order, so the second loop returns the genuinely first unique char. A plain `HashMap` would lose ordering.

### P16. Count vowels and consonants *(easy)*
**Problem:** Count vowels and consonants in a string (letters only).
**Approach:** Lowercase, check membership in "aeiou".
```java
static int[] countVowelsConsonants(String s) {
    int vowels = 0, consonants = 0;
    for (char c : s.toLowerCase().toCharArray()) {
        if (Character.isLetter(c)) {
            if ("aeiou".indexOf(c) >= 0) vowels++;
            else consonants++;
        }
    }
    return new int[]{vowels, consonants};
}
```
**Key point:** `"aeiou".indexOf(c) >= 0` is a compact membership test. `Character.isLetter` filters out digits/spaces.

### P17. Predict the output — string pool vs new *(medium)*
**Problem:** What is printed?
```java
String a = "java";
String b = "java";
String c = new String("java");
System.out.println(a == b);
System.out.println(a == c);
System.out.println(a.equals(c));
System.out.println(a == c.intern());
```
**Approach:** Literals share one pooled object; `new String` forces a distinct heap object; `intern()` returns the pooled reference.
```java
// Output:
// true
// false
// true
// true
```
**Key point:** `==` compares references. `a` and `b` are the same pooled object. `c` is a separate heap object, so `a == c` is false but `.equals()` is true. `c.intern()` returns the pooled `"java"`, equal by reference to `a`.

### P18. Predict the output — string concatenation in a loop *(medium)*
**Problem:** How many String objects (roughly) does this create, and what does it print?
```java
String s = "";
for (int i = 0; i < 3; i++) {
    s += i;
}
System.out.println(s);
```
**Approach:** Each `+=` builds a new String via an intermediate StringBuilder.
```java
// Output:
// 012
```
**Key point:** Prints `012`, but each iteration creates a new String (and StringBuilder) — O(n²) work and GC pressure. Use a single `StringBuilder` for loops.

### P19. Count word occurrences in a sentence *(medium)*
**Problem:** Given a sentence, count how many times each word appears.
**Approach:** Split on whitespace, count with a map.
```java
static Map<String, Integer> wordCount(String sentence) {
    Map<String, Integer> counts = new HashMap<>();
    for (String w : sentence.trim().split("\\s+")) {
        if (!w.isEmpty()) counts.merge(w, 1, Integer::sum);
    }
    return counts;
}
```
**Key point:** `split("\\s+")` collapses runs of whitespace; `trim()` first avoids a leading empty token.

### P20. Fix the bug — comparing strings with == *(easy)*
**Problem:** This method should detect the password "secret" but always returns false even when the input is correct. Fix it.
```java
// BUG
static boolean checkPassword(String input) {
    return input == "secret";
}
```
**Approach:** Compare string *content* with `.equals()`, not references with `==`.
```java
static boolean checkPassword(String input) {
    return "secret".equals(input);
}
```
**Key point:** `==` compares references; a String built at runtime (e.g., from input) is a different object than the literal. Calling `equals` on the literal also avoids a NullPointerException if `input` is null.

### P21. Capitalize first letter of each word *(medium)*
**Problem:** Title-case a sentence: "hello world" → "Hello World".
**Approach:** Split, uppercase first char of each token, join.
```java
static String titleCase(String s) {
    String[] words = s.trim().split("\\s+");
    StringBuilder sb = new StringBuilder();
    for (String w : words) {
        if (w.isEmpty()) continue;
        sb.append(Character.toUpperCase(w.charAt(0)))
          .append(w.substring(1).toLowerCase())
          .append(' ');
    }
    return sb.toString().trim();
}
```
**Key point:** `substring(1)` is safe on single-char words (returns ""). Build with `StringBuilder`, trim the trailing space at the end.

### P22. Remove duplicate characters preserving order *(medium)*
**Problem:** "programming" → "progamin".
**Approach:** Track seen chars in a LinkedHashSet (order preserved, dedup free).
```java
static String dedupChars(String s) {
    Set<Character> seen = new LinkedHashSet<>();
    for (char c : s.toCharArray()) seen.add(c);
    StringBuilder sb = new StringBuilder();
    seen.forEach(sb::append);
    return sb.toString();
}
```
**Key point:** `LinkedHashSet` both deduplicates and preserves insertion order — exactly the semantics this problem needs.

---

## Arrays

### P23. Find max and min in an array *(easy)*
**Problem:** Return the max and min of an int array.
**Approach:** Single pass tracking both.
```java
static int[] maxMin(int[] arr) {
    int max = arr[0], min = arr[0];
    for (int x : arr) {
        if (x > max) max = x;
        if (x < min) min = x;
    }
    return new int[]{max, min};
}
```
**Key point:** Initialize from `arr[0]`, not `0` or `Integer.MAX_VALUE` mistakes — initializing min to 0 breaks for all-positive arrays.

### P24. Reverse an array in place *(easy)*
**Problem:** Reverse an int array without allocating a new one.
**Approach:** Swap symmetric pairs with two pointers.
```java
static void reverse(int[] arr) {
    int i = 0, j = arr.length - 1;
    while (i < j) {
        int tmp = arr[i];
        arr[i++] = arr[j];
        arr[j--] = tmp;
    }
}
```
**Key point:** Loop while `i < j` (not `<=`) — the middle element of an odd-length array stays put.

### P25. Rotate array left by k *(medium)*
**Problem:** Rotate `[1,2,3,4,5]` left by k=2 → `[3,4,5,1,2]`.
**Approach:** Reverse-based trick: reverse first k, reverse rest, reverse whole. Or use modulo with a new array.
```java
static int[] rotateLeft(int[] arr, int k) {
    int n = arr.length;
    k %= n;
    int[] res = new int[n];
    for (int i = 0; i < n; i++) {
        res[i] = arr[(i + k) % n];
    }
    return res;
}
```
**Key point:** `k %= n` handles k larger than the array length. The modulo index `(i + k) % n` wraps around cleanly.

### P26. Remove duplicates from a sorted array *(medium)*
**Problem:** Given a sorted array, remove duplicates in place; return the new length.
**Approach:** Slow/fast pointers — write position advances only on a new value.
```java
static int dedup(int[] arr) {
    if (arr.length == 0) return 0;
    int write = 1;
    for (int read = 1; read < arr.length; read++) {
        if (arr[read] != arr[write - 1]) {
            arr[write++] = arr[read];
        }
    }
    return write;
}
```
**Key point:** Two-pointer in-place dedup is O(n)/O(1). Relies on the array being sorted so duplicates are adjacent.

### P27. Predict the output — array default values *(easy)*
**Problem:** What is printed?
```java
int[] nums = new int[3];
boolean[] flags = new boolean[2];
String[] names = new String[2];
System.out.println(nums[0] + " " + flags[0] + " " + names[0]);
```
**Approach:** Arrays are objects on the heap; elements get type default values.
```java
// Output:
// 0 false null
```
**Key point:** `new int[3]` zero-fills, `boolean[]` defaults to `false`, object arrays default to `null`. (Local *variables* have no defaults, but array elements do.)

### P28. Predict the output — array reference vs copy *(medium)*
**Problem:** What is printed?
```java
int[] a = {1, 2, 3};
int[] b = a;
b[0] = 99;
int[] c = a.clone();
c[1] = 77;
System.out.println(a[0] + " " + a[1]);
```
**Approach:** `b = a` copies the reference (same array); `clone()` makes a shallow copy.
```java
// Output:
// 99 2
```
**Key point:** `b` aliases `a`, so `b[0]=99` changes `a[0]`. `c` is a separate array, so `c[1]=77` does not touch `a[1]`.

### P29. Find the second largest element *(medium)*
**Problem:** Return the second largest distinct value, or throw if fewer than 2 distinct values exist.
**Approach:** Track largest and second-largest in one pass.
```java
static int secondLargest(int[] arr) {
    int first = Integer.MIN_VALUE, second = Integer.MIN_VALUE;
    for (int x : arr) {
        if (x > first) { second = first; first = x; }
        else if (x > second && x != first) { second = x; }
    }
    if (second == Integer.MIN_VALUE) throw new IllegalArgumentException("No second largest");
    return second;
}
```
**Key point:** The `x != first` guard skips duplicates of the max so the second-largest is distinct.

### P30. Move all zeros to the end *(medium)*
**Problem:** `[0,1,0,3,12]` → `[1,3,12,0,0]`, preserving non-zero order, in place.
**Approach:** Write non-zeros to the front, then fill the rest with zeros.
```java
static void moveZeros(int[] arr) {
    int write = 0;
    for (int x : arr) {
        if (x != 0) arr[write++] = x;
    }
    while (write < arr.length) arr[write++] = 0;
}
```
**Key point:** Stable compaction: first pass keeps relative order of non-zeros, second pass zero-fills the tail.

### P31. Fix the bug — ArrayIndexOutOfBounds *(easy)*
**Problem:** This sum loop throws `ArrayIndexOutOfBoundsException`. Fix it.
```java
// BUG
static int sum(int[] arr) {
    int total = 0;
    for (int i = 0; i <= arr.length; i++) {
        total += arr[i];
    }
    return total;
}
```
**Approach:** Valid indices are `0` to `length - 1`; the loop condition must be `<`, not `<=`.
```java
static int sum(int[] arr) {
    int total = 0;
    for (int i = 0; i < arr.length; i++) {
        total += arr[i];
    }
    return total;
}
```
**Key point:** Off-by-one: `arr[arr.length]` is out of bounds. A for-each loop (`for (int x : arr)`) avoids index errors entirely.

---

## OOP

### P32. Design a class hierarchy with polymorphism *(medium)*
**Problem:** Model `Shape` with subclasses `Circle` and `Rectangle`, each computing `area()`. Call polymorphically.
**Approach:** Abstract base method overridden in subclasses; dispatch via base reference.
```java
abstract class Shape {
    abstract double area();
}
class Circle extends Shape {
    private final double r;
    Circle(double r) { this.r = r; }
    @Override double area() { return Math.PI * r * r; }
}
class Rectangle extends Shape {
    private final double w, h;
    Rectangle(double w, double h) { this.w = w; this.h = h; }
    @Override double area() { return w * h; }
}
// Shape s = new Circle(2); s.area() → ~12.566 via dynamic dispatch
```
**Key point:** A `Shape` reference calls the actual object's `area()` — runtime polymorphism (dynamic method dispatch). `@Override` catches signature mistakes at compile time.

### P33. Implement an immutable class *(medium)*
**Problem:** Build an immutable `Point` with x and y.
**Approach:** `final` class, `private final` fields, no setters, init via constructor.
```java
public final class Point {
    private final int x, y;
    public Point(int x, int y) { this.x = x; this.y = y; }
    public int getX() { return x; }
    public int getY() { return y; }
    public Point withX(int newX) { return new Point(newX, y); } // returns a new object
}
```
**Key point:** Immutability = `final` class + `final` fields + no mutators. "Changes" return new instances. For mutable fields (Date, List) you'd also need defensive copies in/out.

### P34. Implement the Builder pattern *(medium)*
**Problem:** Build a `User` with required name and optional email/age using a fluent builder.
**Approach:** Static nested Builder accumulates fields, `build()` constructs the object.
```java
public class User {
    private final String name;
    private final String email;
    private final int age;

    private User(Builder b) { this.name = b.name; this.email = b.email; this.age = b.age; }

    public static class Builder {
        private final String name;     // required
        private String email;
        private int age;
        public Builder(String name) { this.name = name; }
        public Builder email(String email) { this.email = email; return this; }
        public Builder age(int age) { this.age = age; return this; }
        public User build() { return new User(this); }
    }
}
// new User.Builder("Jay").email("j@x.io").age(25).build();
```
**Key point:** Each setter returns `this` to enable chaining. Private constructor forces construction through the builder. Great for objects with many optional params.

### P35. Implement a thread-safe Singleton *(medium)*
**Problem:** Provide a single shared instance of a `Config` class.
**Approach:** Eager initialization (simplest, thread-safe) or the holder idiom for lazy loading.
```java
public class Config {
    private static final Config INSTANCE = new Config(); // eager, thread-safe
    private Config() {}                                   // prevent external new
    public static Config getInstance() { return INSTANCE; }
}
```
**Key point:** A `private` constructor blocks `new Config()`. Eager `static final` init is thread-safe because class loading is synchronized by the JVM. For lazy init, use the static-holder idiom or `enum`.

### P36. Override equals and hashCode *(medium)*
**Problem:** Make two `Money` objects equal when amount and currency match, so they work as HashMap keys.
**Approach:** Override both consistently using `Objects.equals` / `Objects.hash`.
```java
public final class Money {
    private final int amount;
    private final String currency;
    public Money(int amount, String currency) { this.amount = amount; this.currency = currency; }

    @Override public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Money m)) return false;
        return amount == m.amount && Objects.equals(currency, m.currency);
    }
    @Override public int hashCode() { return Objects.hash(amount, currency); }
}
```
**Key point:** If you override `equals`, you MUST override `hashCode` — unequal hashes for "equal" objects break HashMap/HashSet lookups. The `instanceof` pattern (`o instanceof Money m`) also handles null.

### P37. Predict the output — constructor chaining order *(medium)*
**Problem:** What is printed by `new Child();`?
```java
class Parent {
    Parent() { System.out.println("Parent ctor"); }
    { System.out.println("Parent instance block"); }
}
class Child extends Parent {
    { System.out.println("Child instance block"); }
    Child() { System.out.println("Child ctor"); }
}
```
**Approach:** Parent fully initializes first; instance blocks run before each constructor body.
```java
// Output:
// Parent instance block
// Parent ctor
// Child instance block
// Child ctor
```
**Key point:** Order: super() runs first → parent's instance block → parent ctor body → child's instance block → child ctor body. Instance blocks run *after* super() but *before* the constructor body.

### P38. Predict the output — overriding vs hiding *(medium)*
**Problem:** What is printed?
```java
class A {
    void inst()  { System.out.println("A.inst"); }
    static void stat() { System.out.println("A.stat"); }
}
class B extends A {
    @Override void inst() { System.out.println("B.inst"); }
    static void stat() { System.out.println("B.stat"); }
}

public static void main(String[] args) {
    A ref = new B();
    ref.inst();
    ref.stat();
}
```
**Approach:** Instance methods use the object type (override → runtime dispatch); static methods use the reference type (hiding → compile-time).
```java
// Output:
// B.inst
// A.stat
```
**Key point:** `inst()` is overridden → resolved by actual object (B). `stat()` is *hidden*, not overridden → resolved by reference type (A). Static methods don't participate in polymorphism.

### P39. Predict the output — field access is not polymorphic *(medium)*
**Problem:** What is printed?
```java
class Parent { String name = "parent"; }
class Child extends Parent { String name = "child"; }

public static void main(String[] args) {
    Parent p = new Child();
    System.out.println(p.name);
    System.out.println(((Child) p).name);
}
```
**Approach:** Fields are resolved by the reference (compile-time) type, not the object type.
```java
// Output:
// parent
// child
```
**Key point:** Field access is *not* polymorphic — `p.name` uses the declared type `Parent`. Only instance *methods* dispatch on the object's runtime type. Fields are "shadowed", not overridden.

### P40. Implement an interface with multiple implementations *(medium)*
**Problem:** Define a `PaymentMethod` interface and two implementations; select one at runtime.
**Approach:** Program to the interface; pass the concrete impl in.
```java
interface PaymentMethod {
    String pay(double amount);
}
class CreditCard implements PaymentMethod {
    public String pay(double amount) { return "Paid " + amount + " by card"; }
}
class Paypal implements PaymentMethod {
    public String pay(double amount) { return "Paid " + amount + " via PayPal"; }
}
// PaymentMethod m = new Paypal(); m.pay(50); — swap impl without changing callers
```
**Key point:** Coding against the interface (loose coupling) lets you swap `CreditCard`/`Paypal` without touching consumer code — the core of dependency injection.

### P41. Fix the bug — calling overridable method from constructor *(medium)*
**Problem:** `getName()` prints `null` instead of "Bob". Why, and how to fix?
```java
// BUG
class Base {
    Base() { init(); }
    void init() {}
}
class Sub extends Base {
    private String name = "Bob";
    @Override void init() { System.out.println(name); } // prints null!
}
```
**Approach:** The parent constructor runs `init()` before the subclass's field initializers run. Don't call overridable methods from constructors.
```java
class Sub extends Base {
    private String name = "Bob";
    Sub() { init(); }          // call after field init completes
    @Override void init() { System.out.println(name); }
}
```
**Key point:** During `Base()`, `Sub`'s `name` field is still `null` (initialized only after super() returns). Avoid calling overridable methods from constructors — a classic, subtle initialization-order bug.

### P42. Fix the bug — missing super constructor call *(easy)*
**Problem:** This does not compile. Why?
```java
// BUG
class Animal {
    Animal(String name) { /* ... */ }
}
class Dog extends Animal {
    Dog() { /* compile error: no default super() */ }
}
```
**Approach:** `Animal` has no no-arg constructor, so the implicit `super()` fails. Call `super(...)` explicitly.
```java
class Dog extends Animal {
    Dog() { super("dog"); }   // explicitly call the parameterized parent ctor
}
```
**Key point:** Once a class defines any constructor, the compiler stops adding a no-arg default. Subclasses must explicitly call an existing `super(...)`.

---

## Collections

### P43. Group words by length *(medium)*
**Problem:** Group a list of words by their length.
**Approach:** Use `computeIfAbsent` to build lists per key (or `Collectors.groupingBy`).
```java
static Map<Integer, List<String>> groupByLength(List<String> words) {
    Map<Integer, List<String>> map = new HashMap<>();
    for (String w : words) {
        map.computeIfAbsent(w.length(), k -> new ArrayList<>()).add(w);
    }
    return map;
}
```
**Key point:** `computeIfAbsent(key, k -> new ArrayList<>())` creates the bucket only when missing, then returns it for appending — the idiomatic multimap pattern.

### P44. Sort a list of objects with Comparator *(medium)*
**Problem:** Sort a list of `Person(name, age)` by age, then name.
**Approach:** `Comparator.comparingInt(...).thenComparing(...)`.
```java
record Person(String name, int age) {}

static void sortPeople(List<Person> people) {
    people.sort(
        Comparator.comparingInt(Person::age)
                  .thenComparing(Person::name)
    );
}
```
**Key point:** `comparingInt` avoids autoboxing; `thenComparing` chains tiebreakers. Use `.reversed()` for descending.

### P45. Find the most frequent element *(medium)*
**Problem:** Return the element that appears most often in a list.
**Approach:** Count with a map, then take the max-by-value entry.
```java
static <T> T mostFrequent(List<T> list) {
    Map<T, Integer> counts = new HashMap<>();
    for (T x : list) counts.merge(x, 1, Integer::sum);
    return counts.entrySet().stream()
                 .max(Map.Entry.comparingByValue())
                 .map(Map.Entry::getKey)
                 .orElseThrow();
}
```
**Key point:** `Map.Entry.comparingByValue()` plus `stream().max(...)` cleanly finds the top entry. `merge` does the counting.

### P46. Deduplicate a list preserving order *(easy)*
**Problem:** Remove duplicates from a list while keeping first-seen order.
**Approach:** Wrap in a LinkedHashSet, back to a list.
```java
static <T> List<T> dedup(List<T> list) {
    return new ArrayList<>(new LinkedHashSet<>(list));
}
```
**Key point:** `LinkedHashSet` removes duplicates and preserves insertion order in one step. A plain `HashSet` would lose order.

### P47. Find the intersection of two lists *(medium)*
**Problem:** Return elements present in both lists (as a set).
**Approach:** Build a set from one, `retainAll` with the other.
```java
static <T> Set<T> intersection(List<T> a, List<T> b) {
    Set<T> result = new HashSet<>(a);
    result.retainAll(b);
    return result;
}
```
**Key point:** `retainAll` keeps only elements also in `b` — O(n) with a HashSet vs O(n²) nested loops.

### P48. Predict the output — HashMap getOrDefault / merge *(easy)*
**Problem:** What is printed?
```java
Map<String, Integer> m = new HashMap<>();
m.merge("a", 1, Integer::sum);
m.merge("a", 1, Integer::sum);
System.out.println(m.get("a"));
System.out.println(m.getOrDefault("b", 0));
```
**Approach:** First `merge` inserts 1; second adds 1; missing key falls back to default.
```java
// Output:
// 2
// 0
```
**Key point:** `merge` is insert-or-accumulate. `getOrDefault` returns the fallback without inserting it (the map still has no "b").

### P49. Predict the output — Integer cache and == *(medium)*
**Problem:** What is printed?
```java
Integer a = 100, b = 100;
Integer c = 200, d = 200;
System.out.println(a == b);
System.out.println(c == d);
System.out.println(c.equals(d));
```
**Approach:** Autoboxing caches Integers in the range -128..127; outside it, each box is a new object.
```java
// Output:
// true
// false
// true
```
**Key point:** The Integer cache makes `100 == 100` true (same cached object), but `200 == 200` false (two distinct objects). Always use `.equals()` for wrapper comparison.

### P50. Predict the output — List vs Set iteration *(easy)*
**Problem:** What does this print?
```java
List<Integer> list = new ArrayList<>(List.of(3, 1, 2, 3, 1));
Set<Integer> set = new HashSet<>(list);
System.out.println(list.size());
System.out.println(set.size());
```
**Approach:** A List keeps duplicates; a HashSet stores distinct values only.
```java
// Output:
// 5
// 3
```
**Key point:** Converting to a `Set` drops duplicates (3 distinct values: 1, 2, 3). `HashSet` iteration order is unspecified — don't rely on it.

### P51. Fix the bug — ConcurrentModificationException *(medium)*
**Problem:** This throws `ConcurrentModificationException`. Fix it.
```java
// BUG
List<Integer> nums = new ArrayList<>(List.of(1, 2, 3, 4));
for (Integer n : nums) {
    if (n % 2 == 0) nums.remove(n);
}
```
**Approach:** You can't structurally modify a collection during a for-each. Use an Iterator's `remove()` or `removeIf`.
```java
List<Integer> nums = new ArrayList<>(List.of(1, 2, 3, 4));
nums.removeIf(n -> n % 2 == 0);
```
**Key point:** For-each uses an iterator that fails fast on structural modification. `removeIf` (or `Iterator.remove()`) is the safe way to delete while traversing.

### P52. Fix the bug — using a mutable object as a HashMap key *(medium)*
**Problem:** After mutating the key, `map.get(key)` returns null. Why?
```java
// BUG: assume Tag overrides equals/hashCode on its 'value' field
Map<Tag, String> map = new HashMap<>();
Tag key = new Tag("a");
map.put(key, "found");
key.setValue("b");                    // mutate after insertion
System.out.println(map.get(key));     // prints null!
```
**Approach:** The map bucketed the entry by the *original* hashCode. Mutating the key changes its hashCode, so lookup probes the wrong bucket. Use immutable keys.
```java
// FIX: make Tag immutable (final fields, no setters) so its hashCode never changes
final class Tag {
    private final String value;
    Tag(String value) { this.value = value; }
    // equals/hashCode based on value; no setter
}
```
**Key point:** HashMap keys must have a stable hashCode. Mutating a key after insertion corrupts lookups. Prefer immutable types (String, records, Integer) as keys.

### P53. Count frequency and sort entries by count descending *(medium)*
**Problem:** Given words, return entries sorted by frequency (highest first).
**Approach:** Count into a map, stream entries sorted by value reversed.
```java
static List<Map.Entry<String, Integer>> sortedByFrequency(List<String> words) {
    Map<String, Integer> counts = new HashMap<>();
    for (String w : words) counts.merge(w, 1, Integer::sum);
    return counts.entrySet().stream()
                 .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
                 .collect(Collectors.toList());
}
```
**Key point:** `Map.Entry.comparingByValue().reversed()` sorts descending by count. The explicit `<String, Integer>` type witness helps the compiler infer the comparator.

---

## Exceptions

### P54. Implement and use a custom exception *(medium)*
**Problem:** Throw an `InsufficientBalanceException` when a withdrawal exceeds the balance.
**Approach:** Extend `RuntimeException` (unchecked, business rule), throw on violation.
```java
class InsufficientBalanceException extends RuntimeException {
    public InsufficientBalanceException(String msg) { super(msg); }
}
class Account {
    private double balance;
    Account(double balance) { this.balance = balance; }
    void withdraw(double amount) {
        if (amount > balance) {
            throw new InsufficientBalanceException(
                "Required: " + amount + ", Available: " + balance);
        }
        balance -= amount;
    }
}
```
**Key point:** A custom exception names the failure clearly and is catchable by type. Unchecked (extends RuntimeException) suits business-rule violations; checked suits recoverable external failures.

### P55. try-with-resources for automatic cleanup *(medium)*
**Problem:** Read the first line of a file, ensuring the reader is always closed.
**Approach:** Declare the resource in `try(...)`; it auto-closes.
```java
static String firstLine(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } // br.close() called automatically, even on exception
}
```
**Key point:** Any `AutoCloseable` declared in the try header is closed automatically (in reverse declaration order) — no `finally` needed, no leak even if `readLine()` throws.

### P56. Predict the output — finally overrides return *(medium)*
**Problem:** What does `test()` return?
```java
static int test() {
    try {
        return 1;
    } finally {
        return 2;
    }
}
```
**Approach:** `finally` always runs; a `return` in `finally` replaces the `try`'s return.
```java
// test() returns 2
```
**Key point:** A `return` inside `finally` overrides any earlier return or thrown exception — a notorious anti-pattern. Never `return` from `finally`.

### P57. Predict the output — finally runs even after return *(easy)*
**Problem:** What is printed and what is returned?
```java
static int compute() {
    try {
        System.out.println("try");
        return 10;
    } finally {
        System.out.println("finally");
    }
}
// caller: System.out.println(compute());
```
**Approach:** The return value is computed, then `finally` runs, then control returns.
```java
// Output:
// try
// finally
// 10
```
**Key point:** `finally` runs after the `try`'s return value is evaluated but before control actually leaves the method. Here it doesn't change the return (no return in finally), so 10 is returned.

### P58. Predict the output — catch order specific before general *(medium)*
**Problem:** What is printed?
```java
try {
    Object o = "text";
    Integer i = (Integer) o;        // ClassCastException
} catch (ClassCastException e) {
    System.out.println("cast");
} catch (RuntimeException e) {
    System.out.println("runtime");
}
```
**Approach:** The first matching catch wins; the specific subtype is listed first.
```java
// Output:
// cast
```
**Key point:** `ClassCastException` is a `RuntimeException`, so the more specific catch must come first. (If you reversed them, it wouldn't even compile — unreachable catch.)

### P59. Fix the bug — swallowed exception *(easy)*
**Problem:** This parses an int but silently hides failures, returning a misleading 0. Improve it.
```java
// BUG
static int parse(String s) {
    try {
        return Integer.parseInt(s);
    } catch (NumberFormatException e) {
        return 0; // silently swallows the error — caller can't tell 0 from failure
    }
}
```
**Approach:** Don't silently swallow; either rethrow with context or return an explicit signal (e.g., Optional).
```java
static OptionalInt parse(String s) {
    try {
        return OptionalInt.of(Integer.parseInt(s));
    } catch (NumberFormatException e) {
        return OptionalInt.empty();
    }
}
```
**Key point:** Swallowing exceptions hides bugs. Either propagate with context or make failure explicit (`OptionalInt`, a thrown exception). Returning a magic value like 0 is ambiguous.

### P60. Fix the bug — wrong exception type for recoverable failure *(medium)*
**Problem:** `validateAge` uses a checked exception for a programming/validation error, forcing every caller into try-catch. When is unchecked better, and how?
```java
// Heavy-handed: checked exception for an argument-validation error
class InvalidAgeException extends Exception {
    InvalidAgeException(String m) { super(m); }
}
void register(int age) throws InvalidAgeException {
    if (age < 0) throw new InvalidAgeException("age < 0");
}
```
**Approach:** Argument validation is a programming error → use an unchecked exception (e.g., `IllegalArgumentException`), so callers aren't forced to handle it.
```java
void register(int age) {
    if (age < 0) throw new IllegalArgumentException("age < 0");
}
```
**Key point:** Use *checked* exceptions for recoverable external failures (IO, network); use *unchecked* for programming errors and business-rule violations. `IllegalArgumentException` is the standard for bad arguments.

---

## Generics & Wrappers

### P61. Implement a generic swap / pair *(easy)*
**Problem:** Write a generic method that returns the two arguments as a swapped array-like pair.
**Approach:** Use a type parameter `<T>`; return a record holding both.
```java
record Pair<T>(T first, T second) {}

static <T> Pair<T> swap(T a, T b) {
    return new Pair<>(b, a);
}
// swap("x", "y") → Pair[first=y, second=x]
```
**Key point:** `<T>` before the return type declares the method's type parameter. Generics give compile-time type safety without casts.

### P62. Implement a generic max using Comparable *(medium)*
**Problem:** Return the max of a list of `Comparable` elements.
**Approach:** Bound the type parameter with `extends Comparable<T>`.
```java
static <T extends Comparable<T>> T max(List<T> list) {
    T best = list.get(0);
    for (T x : list) {
        if (x.compareTo(best) > 0) best = x;
    }
    return best;
}
```
**Key point:** The bound `<T extends Comparable<T>>` guarantees every element has `compareTo`, so the method works for Integer, String, etc., with full type safety.

### P63. Predict the output — autoboxing in a ternary *(medium)*
**Problem:** What happens here?
```java
Object obj = true ? Integer.valueOf(1) : Double.valueOf(2.0);
System.out.println(obj);
```
**Approach:** A conditional expression with `Integer` and `Double` operands undergoes binary numeric promotion to `double`.
```java
// Output:
// 1.0
```
**Key point:** The ternary's result type is the *promoted* common type (`double`), so the `Integer` 1 is unboxed and widened to `1.0` — even though the `true` branch was taken. A surprising autoboxing/promotion gotcha.

### P64. Predict the output — unboxing a null Integer *(medium)*
**Problem:** What happens at runtime?
```java
Map<String, Integer> m = new HashMap<>();
int x = m.get("missing");   // unboxing
System.out.println(x);
```
**Approach:** `get` returns `null` for a missing key; unboxing `null` to `int` throws NPE.
```java
// Throws NullPointerException at the unboxing of null
```
**Key point:** Assigning an `Integer` that is `null` to an `int` unboxes via `.intValue()` → NPE. Use `getOrDefault("missing", 0)` or keep it as `Integer` and null-check.

### P65. Predict the output — wrapper parsing and valueOf *(easy)*
**Problem:** What is printed?
```java
System.out.println(Integer.parseInt("42") + 1);
System.out.println(Integer.valueOf("42").getClass().getSimpleName());
System.out.println(Integer.MAX_VALUE + 1);
```
**Approach:** `parseInt` returns `int`; `valueOf` returns `Integer`; int overflow wraps around.
```java
// Output:
// 43
// Integer
// -2147483648
```
**Key point:** `parseInt` → primitive, `valueOf` → wrapper object. `Integer.MAX_VALUE + 1` silently overflows to `Integer.MIN_VALUE` — Java int arithmetic wraps, it does not throw.

### P66. Fix the bug — raw type causing ClassCastException *(medium)*
**Problem:** This compiles with a warning but throws `ClassCastException` at runtime. Fix it.
```java
// BUG
List list = new ArrayList();   // raw type
list.add("hello");
list.add(42);
for (Object o : list) {
    String s = (String) o;     // throws on the Integer
    System.out.println(s.length());
}
```
**Approach:** Use a parameterized type so the compiler enforces element type.
```java
List<String> list = new ArrayList<>();
list.add("hello");
// list.add(42);   // now a compile error — caught early
for (String s : list) {
    System.out.println(s.length());
}
```
**Key point:** Raw types bypass generic type checking, deferring errors to runtime. Always parameterize collections so type mismatches fail at compile time.

---

## Warm-up DSA (easy)

### P67. Two Sum *(easy)*
**Problem:** Given an int array and a target, return indices of two numbers that sum to the target.
**Approach:** One-pass hash map: for each value, check if `target - value` was seen.
```java
static int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int need = target - nums[i];
        if (seen.containsKey(need)) return new int[]{seen.get(need), i};
        seen.put(nums[i], i);
    }
    return new int[]{-1, -1};
}
```
**Key point:** The map turns the O(n²) brute force into O(n) by remembering complements as you go.

### P68. Factorial — recursive and iterative *(easy)*
**Problem:** Compute n! both ways.
**Approach:** Recursion: `n * fact(n-1)`, base case 1. Iteration: accumulate a product.
```java
static long factRec(int n) {
    return n <= 1 ? 1 : n * factRec(n - 1);
}
static long factIter(int n) {
    long result = 1;
    for (int i = 2; i <= n; i++) result *= i;
    return result;
}
```
**Key point:** Use `long` — `int` overflows at 13!. Iterative avoids stack growth; recursion is cleaner but risks StackOverflow for large n.

### P69. Fibonacci — recursive and iterative *(easy)*
**Problem:** Return the nth Fibonacci number (F0=0, F1=1).
**Approach:** Iterative with two rolling variables (O(n)); recursion shown for contrast (O(2^n)).
```java
static long fibIter(int n) {
    long a = 0, b = 1;
    for (int i = 0; i < n; i++) {
        long next = a + b;
        a = b;
        b = next;
    }
    return a;
}
static long fibRec(int n) {              // exponential — illustrative only
    return n < 2 ? n : fibRec(n - 1) + fibRec(n - 2);
}
```
**Key point:** Naive recursion recomputes subproblems (exponential). The iterative two-variable version is O(n) time, O(1) space — always prefer it.

### P70. Check if a number is prime *(easy)*
**Problem:** Return true if n is prime.
**Approach:** Test divisors up to √n only.
```java
static boolean isPrime(int n) {
    if (n < 2) return false;
    for (int i = 2; (long) i * i <= n; i++) {
        if (n % i == 0) return false;
    }
    return true;
}
```
**Key point:** Checking up to √n suffices — a composite has a factor ≤ √n. The `(long) i * i` cast avoids int overflow for large n.

### P71. GCD using Euclid's algorithm *(easy)*
**Problem:** Compute the greatest common divisor of a and b.
**Approach:** Repeatedly replace (a, b) with (b, a % b) until b is 0.
```java
static int gcd(int a, int b) {
    while (b != 0) {
        int tmp = b;
        b = a % b;
        a = tmp;
    }
    return a;
}
```
**Key point:** Euclid's algorithm is O(log min(a,b)). When b hits 0, a holds the GCD.

### P72. Reverse an integer *(easy)*
**Problem:** Reverse the digits of an int (e.g., 123 → 321), returning 0 on overflow.
**Approach:** Pop last digit with `% 10`, push onto result; check overflow against `long`.
```java
static int reverseInt(int n) {
    long rev = 0;
    while (n != 0) {
        rev = rev * 10 + n % 10;
        n /= 10;
    }
    return (rev < Integer.MIN_VALUE || rev > Integer.MAX_VALUE) ? 0 : (int) rev;
}
```
**Key point:** Accumulate in a `long` to detect overflow before narrowing. `n % 10` works for negatives too in Java (sign follows the dividend).

### P73. Binary search on a sorted array *(easy)*
**Problem:** Return the index of `target` in a sorted array, or -1.
**Approach:** Maintain lo/hi bounds; compare the midpoint.
```java
static int binarySearch(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```
**Key point:** `lo + (hi - lo) / 2` avoids the integer-overflow bug of `(lo + hi) / 2`. Loop condition `lo <= hi` is essential to check the final single element.

### P74. Check if two strings are rotations *(medium)*
**Problem:** Is "erbottlewat" a rotation of "waterbottle"? Return true.
**Approach:** A rotation of `s` is always a substring of `s + s` (same length).
```java
static boolean isRotation(String a, String b) {
    return a.length() == b.length() && (a + a).contains(b);
}
```
**Key point:** The `s + s` trick elegantly captures all rotations in one string. Always check equal length first to reject trivially.

### P75. Find the missing number in 0..n *(easy)*
**Problem:** An array contains n distinct numbers from 0..n with one missing. Find it.
**Approach:** Expected sum `n*(n+1)/2` minus actual sum.
```java
static int missingNumber(int[] nums) {
    int n = nums.length;
    int expected = n * (n + 1) / 2;
    int actual = 0;
    for (int x : nums) actual += x;
    return expected - actual;
}
```
**Key point:** Sum formula gives an O(n) time, O(1) space answer. (XOR of indices and values is an overflow-proof alternative.)

### P76. Count set bits in an integer *(easy)*
**Problem:** Return the number of 1-bits in the binary representation of n.
**Approach:** Brian Kernighan's trick: `n &= (n - 1)` clears the lowest set bit each iteration.
```java
static int countSetBits(int n) {
    int count = 0;
    while (n != 0) {
        n &= (n - 1);
        count++;
    }
    return count;
}
```
**Key point:** `n & (n - 1)` clears the lowest set bit, so the loop runs once per set bit — faster than checking all 32 bits. (`Integer.bitCount(n)` is the built-in.)
```
