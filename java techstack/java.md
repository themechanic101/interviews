# Java Interview Preparation — SDE1 / SDE2

A practical interview-oriented guide to Java from fundamentals to advanced topics, with code examples, common traps, and SDE1/SDE2 priorities.

---

# 1. Java Fundamentals

## What is Java?

Java is a high-level, class-based, object-oriented language. Java source is compiled into bytecode, which is executed by the JVM.

```text
.java source
    ↓ javac
.class bytecode
    ↓
   JVM
    ↓
 Operating System
```

### Why is Java platform independent?

The compiler generates bytecode rather than OS-specific machine code. A JVM implementation for each platform executes that bytecode.

---

## JDK vs JRE vs JVM

- **JVM** — executes Java bytecode.
- **JRE** — JVM plus runtime libraries.
- **JDK** — development tools plus the runtime needed to develop/run Java applications.

```text
JDK
├── Development tools
└── Runtime
    ├── Libraries
    └── JVM
```

For modern Java development, you normally install a JDK.

---

## Primitive vs Reference Types

```java
int age = 22;
double salary = 50000.0;
boolean active = true;
char grade = 'A';
```

Reference types:

```java
String name = "Alice";
int[] nums = {1, 2, 3};
User user = new User();
```

---

## `static`

A static member belongs to the class rather than an individual object.

```java
class Counter {
    static int count = 0;

    Counter() {
        count++;
    }
}

public class Main {
    public static void main(String[] args) {
        new Counter();
        new Counter();

        System.out.println(Counter.count); // 2
    }
}
```

---

## Why is `main()` static?

The JVM needs to invoke the entry point without first creating an instance of the class.

```java
public static void main(String[] args)
```

---

## `final`, `finally`, `finalize`

### `final`

```java
final int x = 10;
// x = 20; // error
```

A final method cannot be overridden:

```java
class Parent {
    final void show() {}
}
```

A final class cannot be extended:

```java
final class Parent {}
```

### `finally`

Used for cleanup around exception handling:

```java
try {
    // work
} finally {
    System.out.println("cleanup");
}
```

### `finalize()`

A legacy/deprecated mechanism associated with garbage collection. Do not use it for resource cleanup in modern Java.

---

# 2. OOP — Extremely Important

The four major OOP concepts are:

1. Encapsulation
2. Abstraction
3. Inheritance
4. Polymorphism

## Encapsulation

Hide internal state and control access through methods.

```java
class BankAccount {
    private double balance;

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    public double getBalance() {
        return balance;
    }
}
```

Benefits:

- protects invariants
- reduces coupling
- controls modification
- improves maintainability

---

## Abstraction

Expose what an object does while hiding implementation details.

```java
interface Payment {
    void pay(double amount);
}

class CardPayment implements Payment {
    public void pay(double amount) {
        System.out.println("Paid by card");
    }
}
```

---

## Inheritance

```java
class Animal {
    void eat() {
        System.out.println("Eating");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("Barking");
    }
}
```

```java
Dog d = new Dog();
d.eat();
d.bark();
```

Java supports single, multilevel, and hierarchical class inheritance, but not multiple inheritance of classes.

---

## Polymorphism

Runtime polymorphism:

```java
class Animal {
    void sound() {
        System.out.println("Animal sound");
    }
}

class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Bark");
    }
}

Animal a = new Dog();
a.sound(); // Bark
```

The actual overridden method is selected at runtime.

Compile-time polymorphism is commonly achieved through overloading:

```java
class Calculator {
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }
}
```

---

# 3. Overloading vs Overriding

| Feature | Overloading | Overriding |
|---|---|---|
| Relationship | Usually same class | Parent-child |
| Parameters | Must differ | Same signature |
| Binding | Compile time | Runtime |
| Return type | Cannot differ only by return type | Covariant return allowed |
| Static methods | Can overload | Static methods are hidden, not overridden |

---

# 4. Abstract Class vs Interface

Abstract class:

```java
abstract class Vehicle {
    abstract void start();

    void stop() {
        System.out.println("Stopping");
    }
}
```

Interface:

```java
interface Flyable {
    void fly();
}
```

A class can implement multiple interfaces:

```java
class Bird implements Flyable {
    public void fly() {
        System.out.println("Flying");
    }
}
```

### Interview rule

Use an abstract class when you need shared state/implementation in a related hierarchy.

Use an interface when you want a contract/capability that multiple, potentially unrelated classes can implement.

Modern interfaces can also have `default`, `static`, and private methods.

---

# 5. `this` vs `super`

```java
class User {
    String name;

    User(String name) {
        this.name = name;
    }
}
```

`this` refers to the current object.

```java
class Parent {
    int x = 10;
}

class Child extends Parent {
    int x = 20;

    void print() {
        System.out.println(this.x);  // 20
        System.out.println(super.x); // 10
    }
}
```

`super` accesses the parent-class portion of the current object.

---

# 6. Constructors

Constructors initialize objects.

```java
class User {
    String name;

    User(String name) {
        this.name = name;
    }
}
```

Constructors:

- have the class name
- have no return type
- are not inherited
- can be overloaded

If no constructor is declared, Java can provide a default no-argument constructor.

---

# 7. Access Modifiers

| Modifier | Same class | Same package | Subclass | Everywhere |
|---|---:|---:|---:|---:|
| private | Yes | No | No* | No |
| package-private | Yes | Yes | Package-dependent | No |
| protected | Yes | Yes | Yes | No |
| public | Yes | Yes | Yes | Yes |

`protected` across packages has additional rules involving inheritance.

---

# 8. `Object` Class

Every Java class ultimately derives from `Object`.

Important methods:

```java
toString()
equals(Object)
hashCode()
getClass()
clone()
wait()
notify()
notifyAll()
```

---

# 9. `==` vs `equals()`

For objects, `==` compares references.

`equals()` checks logical equality according to the class implementation.

```java
String a = new String("hello");
String b = new String("hello");

System.out.println(a == b);      // false
System.out.println(a.equals(b)); // true
```

---

# 10. `equals()` and `hashCode()`

Important contract:

> If two objects are equal according to `equals()`, they must have the same hash code.

Example:

```java
class User {
    private final int id;

    User(int id) {
        this.id = id;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User)) return false;

        User other = (User) o;
        return id == other.id;
    }

    @Override
    public int hashCode() {
        return Integer.hashCode(id);
    }
}
```

This is critical for `HashMap` and `HashSet`.

---

# 11. String

## Why is String immutable?

```java
String s = "hello";
s.concat(" world");

System.out.println(s); // hello
```

String operations create new strings.

Benefits:

- safe sharing
- string pool
- reliable hashing
- useful security properties
- thread-safety benefits

---

## String Pool

```java
String a = "hello";
String b = "hello";

System.out.println(a == b); // true
```

But:

```java
String a = new String("hello");
String b = new String("hello");

System.out.println(a == b); // false
```

Use `equals()` for content comparison.

---

# 12. String vs StringBuilder vs StringBuffer

| Type | Mutable | Thread-safe | Typical use |
|---|---|---|---|
| String | No | Immutable | Fixed text |
| StringBuilder | Yes | No | Fast string construction |
| StringBuffer | Yes | Yes | Legacy synchronized use |

```java
StringBuilder sb = new StringBuilder();

sb.append("Hello");
sb.append(" ");
sb.append("Java");

System.out.println(sb);
```

---

# 13. Wrapper Classes

```text
int     → Integer
long    → Long
double  → Double
char    → Character
boolean → Boolean
```

Autoboxing/unboxing:

```java
Integer x = 10; // autoboxing
int y = x;      // unboxing
```

Do not rely on `==` for wrapper logical equality:

```java
Integer a = 1000;
Integer b = 1000;

System.out.println(a == b);      // generally false
System.out.println(a.equals(b)); // true
```

---

# 14. Exceptions

## Checked exceptions

Compiler requires handling/declaring them.

Examples:

```text
IOException
SQLException
```

## Unchecked exceptions

Subclasses of `RuntimeException`.

Examples:

```text
NullPointerException
IllegalArgumentException
IndexOutOfBoundsException
```

---

## `throw` vs `throws`

`throw` actually throws:

```java
if (age < 18) {
    throw new IllegalArgumentException("Underage");
}
```

`throws` declares propagation:

```java
void readFile() throws IOException {
    // ...
}
```

---

## try-catch-finally

```java
try {
    int x = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("Invalid division");
} finally {
    System.out.println("Cleanup");
}
```

---

## Try-with-resources

Preferred for `AutoCloseable` resources:

```java
try (BufferedReader br =
         new BufferedReader(new FileReader("data.txt"))) {

    System.out.println(br.readLine());

} catch (IOException e) {
    e.printStackTrace();
}
```

---

# 15. Collections Framework

Know these extremely well:

```text
List
 ├── ArrayList
 └── LinkedList

Set
 ├── HashSet
 ├── LinkedHashSet
 └── TreeSet

Map
 ├── HashMap
 ├── LinkedHashMap
 ├── TreeMap
 └── ConcurrentHashMap

Queue / Deque
 ├── PriorityQueue
 └── ArrayDeque
```

---

# 16. ArrayList vs LinkedList

### ArrayList

Dynamic array.

```java
List<Integer> list = new ArrayList<>();
```

Index access is O(1).

Appending is amortized O(1).

Insertion/removal in the middle is typically O(n) because elements may need shifting.

### LinkedList

Doubly linked node structure.

Finding an arbitrary index is O(n). Insertion/removal can be O(1) once the target node/position is already known.

### Interview answer

`ArrayList` is the usual default. `LinkedList` has more specialized use cases.

---

# 17. HashMap

```java
Map<String, Integer> map = new HashMap<>();

map.put("Alice", 90);
map.put("Bob", 80);

System.out.println(map.get("Alice"));
```

Typical average complexity:

```text
put    O(1)
get    O(1)
remove O(1)
```

Actual performance depends on hashing and implementation details.

---

## How HashMap works

Conceptually:

```text
key
 ↓
hashCode()
 ↓
bucket index
 ↓
bucket
 ↓
equals() when comparing keys
```

Modern Java can treeify heavily collided buckets under certain conditions.

---

# 18. HashMap Collision

Two keys can map to the same bucket.

```text
Key A ──┐
        ├── Bucket
Key B ──┘
```

The implementation stores colliding entries within the bucket and can use a tree structure for sufficiently large collision chains under appropriate conditions.

---

# 19. HashMap vs Hashtable

| HashMap | Hashtable |
|---|---|
| Not synchronized | Synchronized |
| Allows null key/value | Does not allow null |
| Modern choice | Legacy |
| `ConcurrentHashMap` for concurrency | Generally not preferred |

---

# 20. HashMap vs ConcurrentHashMap

`HashMap` is not thread-safe.

`ConcurrentHashMap` is designed for concurrent access and supports thread-safe operations without simply putting one giant lock around the entire map.

```java
ConcurrentHashMap<String, Integer> map =
    new ConcurrentHashMap<>();

map.put("A", 1);
```

---

# 21. HashSet

Stores unique elements.

```java
Set<Integer> set = new HashSet<>();

set.add(10);
set.add(10);

System.out.println(set.size()); // 1
```

Hash-based uniqueness depends on `equals()` and `hashCode()`.

---

# 22. TreeSet

Maintains sorted order.

```java
Set<Integer> set = new TreeSet<>();

set.add(30);
set.add(10);
set.add(20);

System.out.println(set); // [10, 20, 30]
```

Typical operations are O(log n).

---

# 23. LinkedHashMap / LinkedHashSet

Provide predictable iteration order while retaining hashing-based lookup behavior.

Useful when you need insertion/access-order semantics depending on configuration.

---

# 24. Comparable vs Comparator

Comparable defines natural ordering:

```java
class Student implements Comparable<Student> {
    int marks;

    public int compareTo(Student other) {
        return Integer.compare(this.marks, other.marks);
    }
}
```

Comparator defines external/custom ordering:

```java
Comparator<Student> byMarks =
    (a, b) -> Integer.compare(a.marks, b.marks);
```

Use Comparator when you need multiple sorting strategies.

---

# 25. PriorityQueue

Heap-based priority queue.

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();

pq.add(30);
pq.add(10);
pq.add(20);

System.out.println(pq.poll()); // 10
```

Default is a min-heap.

Typical:

```text
peek → O(1)
offer → O(log n)
poll → O(log n)
```

---

# 26. Stack

Prefer `Deque` rather than the legacy `Stack` class:

```java
Deque<Integer> stack = new ArrayDeque<>();

stack.push(10);
stack.push(20);

System.out.println(stack.pop()); // 20
```

---

# 27. Generics

Generics provide compile-time type safety.

```java
List<String> names = new ArrayList<>();

names.add("Alice");
// names.add(10); // compile-time error
```

---

# 28. Wildcards and PECS

### `? extends T`

Producer:

```java
List<? extends Number> nums;
```

Safe for reading as `Number`, but you generally cannot add arbitrary `Number` values.

### `? super T`

Consumer:

```java
List<? super Integer> nums;
```

You can add `Integer`.

### PECS

**Producer Extends, Consumer Super.**

Very common interview question.

---

# 29. Type Erasure

Java generics are primarily implemented through type erasure.

```java
List<String>
List<Integer>
```

do not become separate runtime generic classes.

This explains several runtime limitations of Java generics.

---

# 30. Functional Interfaces

An interface with exactly one abstract method.

Examples:

```text
Runnable
Callable
Comparator
Consumer
Function
Predicate
Supplier
```

Example:

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}
```

---

# 31. Lambda Expressions

```java
Calculator add = (a, b) -> a + b;

System.out.println(add.calculate(2, 3));
```

---

# 32. Method References

```java
List<String> names = List.of("A", "B", "C");

names.forEach(System.out::println);
```

---

# 33. Stream API

```java
List<Integer> nums = List.of(1, 2, 3, 4, 5);

List<Integer> result = nums.stream()
        .filter(n -> n % 2 == 0)
        .map(n -> n * n)
        .toList();

System.out.println(result); // [4, 16]
```

Streams are useful for declarative data processing.

---

# 34. Intermediate vs Terminal Stream Operations

Intermediate operations:

```text
filter
map
sorted
distinct
limit
```

Terminal operations:

```text
toList
collect
forEach
reduce
count
anyMatch
```

Intermediate operations are generally lazy.

---

# 35. `map()` vs `flatMap()`

`map`: one input → one transformed output.

```java
List<Integer> lengths =
    List.of("a", "bb")
        .stream()
        .map(String::length)
        .toList();
```

`flatMap`: transforms and flattens nested streams.

```java
List<Integer> flat =
    List.of(List.of(1, 2), List.of(3, 4))
        .stream()
        .flatMap(List::stream)
        .toList();
```

---

# 36. reduce()

Combines elements into one result.

```java
int sum = List.of(1, 2, 3, 4)
        .stream()
        .reduce(0, Integer::sum);
```

---

# 37. Optional

Represents a value that may be absent.

```java
Optional<String> name =
    Optional.ofNullable(getName());

String result = name.orElse("Unknown");
```

Avoid careless use of `Optional.get()`.

---

# 38. JVM Architecture

```text
                 JVM
 ┌──────────────────────────────┐
 │ Class Loader Subsystem       │
 │                              │
 │ Runtime Data Areas            │
 │  ├── Heap                    │
 │  ├── Thread Stacks           │
 │  ├── Metaspace               │
 │  ├── PC Registers            │
 │  └── Native Method Stacks    │
 │                              │
 │ Execution Engine             │
 │  ├── Interpreter             │
 │  └── JIT Compiler            │
 └──────────────────────────────┘
```

---

# 39. Heap vs Stack

Heap stores objects and arrays.

```java
User u = new User();
```

The object is allocated in heap memory.

Each thread has its own stack containing frames for method invocations and local execution state.

```java
void foo() {
    int x = 10;
}
```

`x` is associated with the method's stack frame.

---

# 40. Is Java Pass-by-Reference?

**No. Java is always pass-by-value.**

When an object is passed, the value being copied is the reference.

```java
void change(User u) {
    u.name = "Bob";
}
```

The method can modify the object because the copied reference points to the same object.

But:

```java
void reassign(User u) {
    u = new User();
}
```

does not replace the caller's reference.

---

# 41. Garbage Collection

Java automatically reclaims objects that are no longer reachable.

```text
Object created
     ↓
References disappear
     ↓
Object unreachable
     ↓
Eligible for GC
```

Eligible does not mean immediately collected.

---

# 42. Can you force GC?

```java
System.gc();
```

This is only a request. The JVM is not required to collect immediately.

---

# 43. Java Memory Leaks

Java can have memory leaks when objects remain reachable unintentionally.

```java
static List<byte[]> cache = new ArrayList<>();

void add() {
    cache.add(new byte[1024 * 1024]);
}
```

Common causes:

- unbounded caches
- static collections
- listeners not removed
- ThreadLocal misuse
- long-lived references

---

# 44. JIT Compiler

Frequently executed bytecode can be compiled into optimized native machine code.

```text
Bytecode
   ↓
Interpreter
   ↓
Hot code
   ↓
JIT
   ↓
Optimized native code
```

---

# 45. Class Loading

Important class-loader concepts:

```text
Application ClassLoader
        ↓
Platform ClassLoader
        ↓
Bootstrap ClassLoader
```

The delegation model helps prevent application classes from replacing core platform classes accidentally.

---

# 46. Multithreading

A thread is an execution path within a process.

```java
class Task implements Runnable {
    public void run() {
        System.out.println("Running");
    }
}

Thread t = new Thread(new Task());
t.start();
```

Use `start()`, not `run()`, to start a new thread.

---

# 47. Thread States

Java's formal states:

```text
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

---

# 48. `start()` vs `run()`

```java
thread.start();
```

starts a new thread.

```java
thread.run();
```

is simply a normal method call on the current thread.

---

# 49. Race Condition

A race condition occurs when correctness depends on timing/interleaving between threads.

```java
count++;
```

This is not an atomic operation.

Two threads can read the same value and overwrite each other's updates.

---

# 50. synchronized

Provides mutual exclusion and memory-visibility guarantees around synchronized regions.

```java
class Counter {
    private int count;

    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}
```

Only one thread at a time can execute the synchronized instance method for the same object's monitor.

---

# 51. synchronized Block

Use a smaller critical section when appropriate:

```java
synchronized (lock) {
    count++;
}
```

---

# 52. volatile

`volatile` provides visibility and ordering guarantees for a variable.

```java
private volatile boolean running = true;
```

It does **not** make compound operations atomic:

```java
volatile int count;
count++; // still not atomic
```

---

# 53. AtomicInteger

```java
AtomicInteger counter = new AtomicInteger();

counter.incrementAndGet();
```

Useful for atomic read-modify-write operations.

---

# 54. volatile vs Atomic

```text
volatile:
visibility + ordering

AtomicInteger:
atomic operations + visibility
```

Choose based on the required guarantee.

---

# 55. ExecutorService

Prefer thread pools for many independent tasks.

```java
ExecutorService executor =
        Executors.newFixedThreadPool(4);

executor.submit(() ->
    System.out.println("Task")
);

executor.shutdown();
```

Benefits:

- thread reuse
- bounded concurrency
- simpler task management
- lifecycle control

---

# 56. Runnable vs Callable

Runnable:

```java
Runnable r = () -> System.out.println("work");
```

Callable returns a result:

```java
Callable<Integer> c = () -> 42;
```

Callable can also throw checked exceptions.

---

# 57. Future

```java
Future<Integer> future =
    executor.submit(() -> 42);

Integer result = future.get();
```

`get()` may block.

---

# 58. CompletableFuture

Useful for asynchronous composition.

```java
CompletableFuture
    .supplyAsync(() -> fetchUser())
    .thenApply(user -> user.getName())
    .thenAccept(System.out::println);
```

Know chaining, exception handling, combining futures, and executor selection for SDE2 interviews.

---

# 59. Deadlock

Two or more threads wait forever for locks held by one another.

```text
Thread A:
Lock 1 → waiting for Lock 2

Thread B:
Lock 2 → waiting for Lock 1
```

Prevent with:

- consistent lock ordering
- smaller critical sections
- avoiding unnecessary nested locks
- timed acquisition where appropriate

---

# 60. wait() vs sleep()

```java
Thread.sleep(1000);
```

Pauses the current thread. It does not release a monitor merely because the thread sleeps.

```java
synchronized (lock) {
    lock.wait();
}
```

`wait()` releases the object's monitor and waits.

---

# 61. notify() vs notifyAll()

```java
lock.notify();
```

wakes one waiting thread.

```java
lock.notifyAll();
```

wakes all waiting threads.

Correct code should always coordinate around an actual condition and re-check that condition after waking.

---

# 62. Concurrent Collections

Know:

```text
ConcurrentHashMap
CopyOnWriteArrayList
BlockingQueue
ConcurrentLinkedQueue
```

Use the collection whose concurrency semantics fit the workload.

---

# 63. Immutability

Immutable objects cannot change after construction.

```java
final class User {
    private final String name;

    User(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

For mutable fields, use defensive copies.

Benefits:

- easier reasoning
- safer sharing
- thread-safety benefits
- simpler caching

---

# 64. Records

Modern Java supports records for concise data carriers.

```java
record User(int id, String name) {}
```

The compiler provides accessors and value-oriented `equals`, `hashCode`, and `toString` implementations based on the components.

---

# 65. Enum

Use enums for a fixed set of constants.

```java
enum Status {
    PENDING,
    SUCCESS,
    FAILED
}
```

Enums can also contain fields and methods.

---

# 66. Sealed Classes

Restrict permitted subclasses/implementations.

```java
sealed interface Shape
        permits Circle, Rectangle {}

final class Circle implements Shape {}
final class Rectangle implements Shape {}
```

Useful when a closed hierarchy is desirable.

---

# 67. Dependency Injection

Instead of creating dependencies internally:

```java
class OrderService {
    private PaymentService payment =
        new PaymentService();
}
```

Inject them:

```java
class OrderService {
    private final PaymentService payment;

    OrderService(PaymentService payment) {
        this.payment = payment;
    }
}
```

Benefits:

- loose coupling
- testability
- easier configuration
- easier replacement

---

# 68. SOLID

### S — Single Responsibility
A class should have one primary responsibility.

### O — Open/Closed
Open for extension, closed for modification.

### L — Liskov Substitution
Subtypes should be usable wherever the base type is expected.

### I — Interface Segregation
Prefer focused interfaces over large ones.

### D — Dependency Inversion
High-level code should depend on abstractions rather than concrete implementations.

---

# 69. Singleton

A thread-safe lazy implementation:

```java
final class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

A simple modern alternative is an enum singleton:

```java
enum AppConfig {
    INSTANCE
}
```

Know singleton's trade-offs: global state, testing difficulty, and hidden dependencies.

---

# 70. Factory Pattern

Centralizes object creation.

```java
interface Payment {
    void pay();
}

class CardPayment implements Payment {
    public void pay() {
        System.out.println("Card");
    }
}

class PaymentFactory {
    static Payment create(String type) {
        if ("card".equals(type)) {
            return new CardPayment();
        }
        throw new IllegalArgumentException("Unknown type");
    }
}
```

---

# 71. Builder Pattern

Useful for objects with many optional parameters.

```java
class User {
    private final String name;
    private final int age;

    private User(Builder b) {
        this.name = b.name;
        this.age = b.age;
    }

    static class Builder {
        private String name;
        private int age;

        Builder name(String name) {
            this.name = name;
            return this;
        }

        Builder age(int age) {
            this.age = age;
            return this;
        }

        User build() {
            return new User(this);
        }
    }
}
```

Usage:

```java
User u = new User.Builder()
        .name("Alice")
        .age(25)
        .build();
```

---

# 72. Java I/O

Important classes:

```text
InputStream
OutputStream
Reader
Writer
BufferedReader
BufferedWriter
Path
Files
```

Modern Java commonly uses `Path` and `Files`:

```java
Path path = Path.of("data.txt");

String content = Files.readString(path);
```

---

# 73. Serialization

Java historically supports object serialization using `Serializable`:

```java
class User implements Serializable {
    private static final long serialVersionUID = 1L;
}
```

For network APIs, explicit formats such as JSON are commonly preferred. Native Java serialization has security and maintenance concerns.

---

# 74. JDBC

Typical flow:

```text
Get connection
      ↓
Prepare statement
      ↓
Execute query
      ↓
Read ResultSet
      ↓
Close resources
```

Example:

```java
String sql =
    "SELECT id, name FROM users WHERE id = ?";

try (Connection con = dataSource.getConnection();
     PreparedStatement ps = con.prepareStatement(sql)) {

    ps.setInt(1, 10);

    try (ResultSet rs = ps.executeQuery()) {
        while (rs.next()) {
            System.out.println(rs.getString("name"));
        }
    }
}
```

---

# 75. Why PreparedStatement?

It helps prevent SQL injection and handles parameters separately from SQL syntax.

Bad:

```java
"SELECT * FROM users WHERE name = '" + name + "'"
```

Good:

```java
"SELECT * FROM users WHERE name = ?"
```

---

# 76. Connection Pooling

Creating DB connections is expensive. A pool maintains reusable connections.

```text
Application
     ↓
Connection Pool
 ┌───┬───┬───┬───┐
 C1  C2  C3  C4
 └───┴───┴───┴───┘
     ↓
  Database
```

Important production considerations:

- pool size
- connection timeout
- idle timeout
- max lifetime
- database capacity

---

# 77. Java Memory Model — SDE2

The Java Memory Model defines visibility, ordering, and synchronization behavior between threads.

Important concepts:

- happens-before
- visibility
- atomicity
- ordering
- synchronized
- volatile

---

# 78. Happens-Before

If action A happens-before action B, effects of A are guaranteed to be visible to B under the Java Memory Model.

Important examples:

- unlock happens-before a subsequent lock on the same monitor
- volatile write happens-before a subsequent read of that variable
- `Thread.start()` happens-before actions in the started thread
- actions in a thread happen-before another thread successfully returns from `join()`

---

# 79. `synchronized` vs `Lock`

```java
Lock lock = new ReentrantLock();

lock.lock();

try {
    // critical section
} finally {
    lock.unlock();
}
```

`Lock` can provide:

- `tryLock()`
- interruptible acquisition
- multiple conditions
- explicit lock control

---

# 80. ReentrantLock

A thread holding a `ReentrantLock` can acquire it again.

```java
lock.lock();

try {
    lock.lock();

    try {
        // nested critical section
    } finally {
        lock.unlock();
    }

} finally {
    lock.unlock();
}
```

---

# 81. ReadWriteLock

Useful when reads greatly outnumber writes.

```java
ReadWriteLock lock =
    new ReentrantReadWriteLock();

lock.readLock().lock();

try {
    // read
} finally {
    lock.readLock().unlock();
}
```

Multiple readers can proceed concurrently while writers require exclusive access.

---

# 82. Semaphore

Controls access using permits.

```java
Semaphore semaphore = new Semaphore(10);

semaphore.acquire();

try {
    // use limited resource
} finally {
    semaphore.release();
}
```

Useful for limiting concurrency to external resources.

---

# 83. CountDownLatch

Wait until a count reaches zero.

```java
CountDownLatch latch =
    new CountDownLatch(3);

latch.countDown();
latch.countDown();
latch.countDown();

latch.await();
```

A latch is generally one-shot.

---

# 84. CyclicBarrier

Allows a group of threads to wait at a common barrier.

```java
CyclicBarrier barrier =
    new CyclicBarrier(3);

barrier.await();
```

A cyclic barrier can be reused.

---

# 85. Garbage Collector Concepts

Know:

- young generation
- old generation
- GC roots
- stop-the-world pauses
- allocation rate
- heap sizing
- throughput vs latency
- collector selection

Do not memorize every collector's implementation unless interviewing for JVM/performance-focused roles.

---

# 86. GC Roots

Objects reachable from GC roots are considered live.

Examples include:

- active thread stacks
- static references
- JNI references
- other JVM-managed roots

An object becomes collectible when no GC root can reach it.

---

# 87. OutOfMemoryError vs StackOverflowError

### OutOfMemoryError

The JVM cannot satisfy an allocation/resource requirement.

### StackOverflowError

Usually caused by excessive recursion:

```java
void recurse() {
    recurse();
}
```

---

# 88. ClassNotFoundException vs NoClassDefFoundError

### ClassNotFoundException

Often occurs when code explicitly attempts to load a class and cannot find it.

### NoClassDefFoundError

The JVM cannot successfully load a class that was available/expected during earlier compilation or resolution.

They are different problems.

---

# 89. Reflection

Reflection allows runtime inspection of classes and members.

```java
Class<?> clazz = User.class;

System.out.println(clazz.getName());
```

Used by frameworks, dependency injection, serializers, and testing tools.

Trade-offs:

- reduced compile-time safety
- complexity
- potential performance overhead
- encapsulation concerns

---

# 90. Annotations

Annotations provide metadata.

```java
@Override
public String toString() {
    return "User";
}
```

Custom annotation:

```java
@interface Auditable {}
```

Frameworks use annotations heavily.

---

# 91. Java Modules

The Java Platform Module System provides explicit modules.

```java
module com.example.app {
    requires java.sql;
    exports com.example.api;
}
```

It provides explicit dependencies and stronger encapsulation.

---

# 92. Date and Time API

Prefer `java.time` over legacy `Date`/`Calendar`.

```java
LocalDate date = LocalDate.now();

LocalDate nextWeek =
    date.plusWeeks(1);
```

Know:

```text
LocalDate
LocalTime
LocalDateTime
Instant
ZonedDateTime
Duration
Period
DateTimeFormatter
```

---

# 93. Instant vs LocalDateTime

`LocalDateTime` has no timezone:

```java
LocalDateTime.now();
```

`Instant` represents a point on the UTC timeline:

```java
Instant.now();
```

For distributed systems, timestamps are commonly stored as instants/UTC and converted to local time for presentation.

---

# 94. Parallel Streams

```java
list.parallelStream()
    .map(...)
    .toList();
```

Do not assume parallel streams are always faster.

They can hurt performance for:

- small workloads
- blocking I/O
- shared mutable state
- ordering-sensitive tasks
- poorly partitionable work

---

# 95. Stream Thread-Safety Trap

Avoid shared mutation:

```java
List<Integer> result = new ArrayList<>();

numbers.parallelStream()
       .forEach(result::add);
```

Prefer collection through the stream pipeline:

```java
List<Integer> result =
    numbers.parallelStream()
           .filter(...)
           .toList();
```

---

# 96. groupingBy

```java
Map<String, List<Employee>> grouped =
    employees.stream()
        .collect(
            Collectors.groupingBy(Employee::department)
        );
```

Very common in Java interviews.

---

# 97. toMap

```java
Map<Integer, String> map =
    users.stream()
         .collect(Collectors.toMap(
             User::id,
             User::name
         ));
```

If duplicate keys are possible, provide a merge function.

---

# 98. Fail-Fast Iteration

Some collections detect structural modification during iteration and may throw `ConcurrentModificationException`.

Bad:

```java
for (Integer x : list) {
    if (x == 2) {
        list.remove(x);
    }
}
```

Use an iterator's removal operation where appropriate:

```java
Iterator<Integer> it = list.iterator();

while (it.hasNext()) {
    if (it.next() == 2) {
        it.remove();
    }
}
```

Concurrent collections have different iteration semantics.

---

# 99. ArrayList Capacity

If approximate size is known:

```java
List<Integer> list =
    new ArrayList<>(1000);
```

This can reduce internal resizing/copying.

---

# 100. Time Complexity Cheat Sheet

| Structure | Access | Search | Insert | Delete |
|---|---:|---:|---:|---:|
| ArrayList | O(1) index | O(n) | O(n)* | O(n)* |
| LinkedList | O(n) index | O(n) | O(1)** | O(1)** |
| HashMap | — | O(1) avg | O(1) avg | O(1) avg |
| TreeMap | — | O(log n) | O(log n) | O(log n) |
| HashSet | — | O(1) avg | O(1) avg | O(1) avg |
| TreeSet | — | O(log n) | O(log n) | O(log n) |
| PriorityQueue | — | — | O(log n) | O(log n) |

`*` depends on position; appending is amortized O(1).

`**` assumes the target node/position is already known.

---

# 101. Common Java Coding Questions

## Reverse String

```java
String s = "hello";

String reversed =
    new StringBuilder(s).reverse().toString();
```

## Palindrome

```java
boolean isPalindrome(String s) {
    int l = 0;
    int r = s.length() - 1;

    while (l < r) {
        if (s.charAt(l++) != s.charAt(r--)) {
            return false;
        }
    }

    return true;
}
```

## Character Frequency

```java
Map<Character, Integer> freq = new HashMap<>();

for (char c : s.toCharArray()) {
    freq.merge(c, 1, Integer::sum);
}
```

## Two Sum

```java
int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();

    for (int i = 0; i < nums.length; i++) {
        int need = target - nums[i];

        if (map.containsKey(need)) {
            return new int[]{map.get(need), i};
        }

        map.put(nums[i], i);
    }

    return new int[]{};
}
```

Time: O(n), Space: O(n).

## First Non-Repeating Character

```java
Character firstUnique(String s) {
    Map<Character, Integer> freq =
        new LinkedHashMap<>();

    for (char c : s.toCharArray()) {
        freq.merge(c, 1, Integer::sum);
    }

    for (var entry : freq.entrySet()) {
        if (entry.getValue() == 1) {
            return entry.getKey();
        }
    }

    return null;
}
```

---

# 102. Spring Boot Topics for Java Backend Interviews

For SDE1/SDE2 backend roles, Java language knowledge is only one layer.

Know:

- IoC
- Dependency Injection
- Beans
- component scanning
- `@Component`
- `@Service`
- `@Repository`
- `@RestController`
- constructor injection
- configuration
- profiles
- validation
- exception handling
- transactions

Example:

```java
@Service
class UserService {

    private final UserRepository repository;

    UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

Constructor injection makes dependencies explicit and testable.

---

# 103. REST Controller

```java
@RestController
@RequestMapping("/users")
class UserController {

    private final UserService service;

    UserController(UserService service) {
        this.service = service;
    }

    @GetMapping("/{id}")
    User getUser(@PathVariable Long id) {
        return service.getUser(id);
    }
}
```

---

# 104. Transactions

Spring applications commonly use transaction boundaries:

```java
@Transactional
public void transferMoney(
        Long from,
        Long to,
        BigDecimal amount) {

    // debit
    // credit
}
```

Operations that must succeed/fail together should be inside the appropriate transaction boundary.

---

# 105. ACID

### Atomicity
All-or-nothing transaction behavior.

### Consistency
Transactions preserve defined database invariants.

### Isolation
Concurrent transactions follow the database's chosen isolation guarantees.

### Durability
Committed changes survive failures according to the database's durability guarantees.

---

# 106. SQL Injection

Bad:

```java
String sql =
    "SELECT * FROM users WHERE name = '" + name + "'";
```

Good:

```java
PreparedStatement ps =
    connection.prepareStatement(
        "SELECT * FROM users WHERE name = ?");
```

---

# 107. Caching

Typical backend flow:

```text
Client
  ↓
API
  ↓
Cache ── hit ──→ Response
  │
 miss
  ↓
Database
```

Know:

- cache-aside
- TTL
- invalidation
- cache stampede
- hot keys
- distributed caching
- Redis basics

---

# 108. Thread Pool Sizing

A rough starting point:

```text
CPU-bound:
threads ≈ CPU cores
```

For I/O-heavy workloads, more concurrency may help because threads spend time waiting.

Production sizing should consider:

- CPU
- blocking ratio
- latency target
- queue size
- downstream capacity
- load testing

---

# 109. Virtual Threads

Modern Java provides virtual threads for lightweight concurrency.

```java
Thread.startVirtualThread(() -> {
    System.out.println("Hello");
});
```

They are useful for large numbers of mostly-blocking tasks.

They do not make CPU-bound computation faster automatically.

Also remember that downstream limits still matter:

```text
100,000 virtual threads
        ↓
Database pool of 20 connections
        ↓
Only 20 DB operations can proceed concurrently
```

---

# 110. SDE1 Priority Checklist

## Java Core

- Java/JVM/JDK
- primitive/reference types
- static
- final
- constructors
- access modifiers
- this/super
- OOP
- overloading/overriding
- abstract class/interface
- String immutability
- String pool
- == vs equals
- equals/hashCode

## Collections

- ArrayList
- LinkedList
- HashMap
- HashSet
- TreeMap
- TreeSet
- PriorityQueue
- Comparable/Comparator
- HashMap internals
- Generics

## Exceptions

- checked/unchecked
- throw/throws
- try/catch/finally
- try-with-resources

## Java 8+

- lambda
- functional interfaces
- streams
- Optional
- method references

## Concurrency

- Thread
- Runnable
- synchronized
- volatile
- ExecutorService
- Future
- race conditions
- deadlock

---

# 111. SDE2 Priority Checklist

Add:

## JVM

- heap/stack
- GC
- GC roots
- JIT
- class loading
- memory leaks
- profiling concepts

## Concurrency

- Java Memory Model
- happens-before
- Atomic classes
- ConcurrentHashMap
- locks
- ReentrantLock
- ReadWriteLock
- CompletableFuture
- ForkJoinPool
- virtual threads
- deadlock
- starvation
- livelock

## Backend

- Spring Boot
- dependency injection
- REST
- transactions
- connection pooling
- caching
- queues
- idempotency
- retries/timeouts
- observability
- distributed systems basics

## Design

- SOLID
- Factory
- Builder
- Strategy
- Observer
- Adapter
- Decorator
- Singleton
- LLD
- HLD

---

# 112. Rapid-Fire Questions

### Why is String immutable?

It provides safe sharing, pooling benefits, reliable hashing, and useful security/thread-safety properties.

### Why override hashCode with equals?

Equal objects must have equal hash codes for hash-based collections to behave correctly.

### Is Java pass-by-reference?

No. Java is always pass-by-value. For objects, the copied value is the reference.

### ArrayList or LinkedList?

Usually ArrayList. LinkedList has specialized use cases and does not provide O(1) arbitrary insertion by index.

### HashMap complexity?

Average O(1) for basic operations, subject to hashing and implementation details.

### HashMap vs ConcurrentHashMap?

HashMap is not thread-safe. ConcurrentHashMap is designed for concurrent access.

### volatile vs synchronized?

Volatile provides visibility/order guarantees for a variable. Synchronized provides mutual exclusion plus memory-visibility guarantees.

### == vs equals?

For objects, `==` compares references; `equals()` defines logical equality.

### What is a race condition?

A concurrency bug caused by timing/interleaving of operations.

### What is deadlock?

Threads permanently wait for resources/locks held by one another.

### Can Java have memory leaks?

Yes. Objects can remain reachable unintentionally and therefore cannot be garbage collected.

### Why dependency injection?

It reduces coupling and improves testability/configurability.

---

# 113. Common Interview Traps

## Trap 1 — Wrapper comparison

```java
Integer a = 1000;
Integer b = 1000;

System.out.println(a == b);
```

Do not use `==` for wrapper logical equality.

---

## Trap 2 — String immutability

```java
String s = "hello";
s.concat("world");
```

`s` remains `"hello"`.

---

## Trap 3 — Starting threads

```java
thread.run();
```

does not create a new thread.

Use:

```java
thread.start();
```

---

## Trap 4 — volatile

```java
volatile int count;
count++;
```

`volatile` does not make increment atomic.

---

## Trap 5 — ArrayList complexity

Do not say every ArrayList insertion is O(n).

Appending is amortized O(1). Inserting/removing in the middle is typically O(n).

---

# 114. How to Answer Java Interview Questions

Use:

```text
1. Definition
2. Why it exists
3. How it works
4. Small example
5. Real-world use
6. Trade-off / limitation
```

Example:

**Q: Why use ConcurrentHashMap?**

A strong answer:

> ConcurrentHashMap is a thread-safe map designed for concurrent access. It allows useful concurrency rather than requiring every access to be protected by one application-wide lock. It is useful for shared state such as caches, counters, and registries. Its atomic operations such as `computeIfAbsent` are useful for compound map operations, but application-level correctness still depends on the surrounding logic.

---

# 115. Recommended Fresher Preparation Order

For SDE1:

```text
Java Core
   ↓
OOP
   ↓
Collections
   ↓
Exceptions
   ↓
Streams + Lambdas
   ↓
Basic Concurrency
   ↓
DSA
   ↓
SQL
   ↓
Spring Boot
   ↓
Projects
```

For SDE2:

```text
Everything above
      ↓
JVM internals
      ↓
Advanced concurrency
      ↓
Spring Boot
      ↓
Database + Transactions
      ↓
Caching + Messaging
      ↓
LLD
      ↓
HLD/System Design
      ↓
Production trade-offs
```

The goal is not to memorize syntax. Be able to explain **why** a feature/data structure is used, what happens internally, its complexity, and its trade-offs.
