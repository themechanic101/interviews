# StringBuffer in Java

**StringBuffer** is a *mutable, thread-safe* sequence of characters—unlike `String`, which is immutable.  
It’s mainly used when you need to frequently modify strings (append, insert, delete, replace) in a multi-threaded environment.

---

## 🔹 Key Features
- **Mutable**: Content can be changed without creating new objects.
- **Thread-safe**: All methods are synchronized, making it safe for use in multi-threaded programs.
- **Capacity management**: Default capacity is 16 characters; expands automatically when exceeded.
- **Implements**: `Serializable`, `Appendable`, `CharSequence`, `Comparable<StringBuffer>`.

---

## 🔹 Constructors
1. `StringBuffer()` → Creates empty buffer with default capacity (16).
2. `StringBuffer(int capacity)` → Creates buffer with specified capacity.
3. `StringBuffer(String str)` → Initializes buffer with given string + 16 extra capacity.

---

## 🔹 Common Methods

| Method | Description | Example |
|--------|-------------|---------|
| `append()` | Adds text at the end | `sb.append("World");` |
| `insert(int index, String str)` | Inserts text at given index | `sb.insert(5, "Java");` |
| `delete(int start, int end)` | Removes characters in range | `sb.delete(0, 5);` |
| `replace(int start, int end, String str)` | Replaces characters in range | `sb.replace(0, 5, "Hi");` |
| `reverse()` | Reverses sequence | `sb.reverse();` |
| `capacity()` | Returns current capacity | `sb.capacity();` |
| `ensureCapacity(int min)` | Ensures minimum capacity | `sb.ensureCapacity(50);` |

---

## 🔹 Example Code

```java
public class Demo {
    public static void main(String[] args) {
        // Create StringBuffer
        StringBuffer sb = new StringBuffer("Hello");

        // Append
        sb.append(" World");
        System.out.println(sb); // Output: Hello World

        // Insert
        sb.insert(6, "Java ");
        System.out.println(sb); // Output: Hello Java World

        // Delete
        sb.delete(0, 6);
        System.out.println(sb); // Output: Java World

        // Reverse
        sb.reverse();
        System.out.println(sb); // Output: dlroW avaJ
    }
}
```

# 🔹 Static in Java

The `static` keyword in Java means **belonging to the class rather than an instance**.  
Static members are shared across all objects of the class.
---




## 1️⃣ Static Variable
- Declared with the `static` keyword inside a class.
- **Shared** among all objects (only one copy exists).
- Useful for constants or counters.

### Example:
```java
class Counter {
    static int count = 0; // static variable

    Counter() {
        count++;
    }
}

public class Demo {
    public static void main(String[] args) {
        Counter c1 = new Counter();
        Counter c2 = new Counter();
        System.out.println(Counter.count); // Output: 2
    }
}

```
# 🔹 Public vs Static in Java

In Java, both `public` and `static` are modifiers, but they serve **different purposes**:

---

## 1️⃣ Public
- **Access modifier** → defines visibility of a class, method, or variable.
- If something is `public`, it can be accessed **from anywhere** (any class, package).
- Belongs to the **object (instance)** if applied to variables.
- Each object gets its **own copy** of a public instance variable.

### Example:
```java
class Student {
    public String name; // public instance variable
}

public class Demo {
    public static void main(String[] args) {
        Student s1 = new Student();
        s1.name = "Alice";

        Student s2 = new Student();
        s2.name = "Bob";

        System.out.println(s1.name); // Alice
        System.out.println(s2.name); // Bob
    }
}
```


# 📌 Annotations in Java

## 🔹 What Are Annotations?
- **Definition**: Metadata attached to program elements (classes, methods, variables, etc.).
- **Purpose**: Guide compilers, tools, and frameworks.
- **Syntax**: Always start with `@`.

Example:
```java
@Override
public String toString() {
    return "Hello";
}
```


# 🔹 Categories of Annotations in Java

Java supports several categories of annotations, each serving a different purpose:

| Category | Description | Example |
|----------|-------------|---------|
| **Marker Annotations** | No elements; presence alone conveys meaning. | `@Deprecated` |
| **Single-Value Annotations** | One element; shorthand notation allowed. | `@SuppressWarnings("unchecked")` |
| **Full Annotations** | Multiple key-value pairs. | `@Entity(name="Student")` |
| **Type Annotations** | Applied to types for stronger checks (Java 8+). | `List<@NonNull String>` |
| **Repeating Annotations** | Same annotation applied multiple times. | `@Schedule(day="Mon") @Schedule(day="Tue")` |

---

## 📌 Examples

### 1. Marker Annotation
```java
@Deprecated
public void oldMethod() { }
```

# 📌 Enums in Java

## 🔹 What Are Enums?
- **Definition**: Enums (short for *enumerations*) are special data types that define a fixed set of constants.
- **Purpose**: Represent a group of related values (like days of the week, directions, states).
- **Syntax**: Declared using the `enum` keyword.

Example:
```java
enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
```


# 📌 `this` Keyword in Java

## 🔹 What is `this`?
- `this` is a reference to the **current object** in Java.
- It is used inside methods or constructors to refer to the object that is currently being acted upon.

---

## 🔹 Uses of `this` Keyword

### 1. Refer to Current Object’s Instance Variables
```java
class Student {
    String name;
    int age;

    Student(String name, int age) {
        this.name = name;  // refers to instance variable
        this.age = age;
    }
}
```



