# Python Revision Guide

A practical revision guide covering Python fundamentals, data structures, OOP, advanced features, APIs, data science, and AI/ML engineering.

---

# 1. Python Basics

## 1.1 Variables

Python is dynamically typed. You do not need to explicitly declare a variable's type.

```python
name = "Utkarsh"
age = 22
cgpa = 7.3
is_student = True
```

Python variables are references to objects.

```python
x = 10
y = x
```

### Multiple Assignment

```python
a, b, c = 1, 2, 3

x = y = z = 0
```

### Constants

Python does not enforce constants. By convention, uppercase names indicate values that should not be changed.

```python
MAX_SIZE = 100
PI = 3.14159
```

---

## 1.2 Data Types

### Numeric Types

```python
x = 10          # int
y = 3.14        # float
z = 2 + 3j      # complex
```

### Boolean

```python
is_valid = True
is_empty = False
```

### String

```python
name = "Python"
```

### None

```python
result = None
```

### Checking Type

```python
x = 10

print(type(x))          # <class 'int'>
print(isinstance(x, int))  # True
```

### Mutable vs Immutable

**Mutable:**
- `list`
- `dict`
- `set`
- most user-defined objects

**Immutable:**
- `int`
- `float`
- `bool`
- `str`
- `tuple`
- `frozenset`

```python
s = "hello"
# s[0] = "H"  # TypeError
```

---

## 1.3 Type Casting

```python
x = int("10")
y = float("3.14")
z = str(100)
b = bool(1)

print(x, y, z, b)
```

Be careful with invalid conversions:

```python
# int("abc")  # ValueError
```

---

## 1.4 Operators

### Arithmetic Operators

```python
a = 10
b = 3

a + b    # 13
a - b    # 7
a * b    # 30
a / b    # 3.333...
a // b   # 3
a % b    # 1
a ** b   # 1000
```

`/` always produces a floating-point result.

`//` performs floor division.

---

### Comparison Operators

```python
a == b
a != b
a > b
a < b
a >= b
a <= b
```

### Logical Operators

```python
x and y
x or y
not x
```

Example:

```python
age = 22

if age >= 18 and age <= 60:
    print("Adult")
```

### Identity Operators

```python
x is y
x is not y
```

`is` checks object identity, while `==` checks value equality.

```python
a = [1, 2]
b = [1, 2]

print(a == b)  # True
print(a is b)  # False
```

### Membership Operators

```python
"a" in "cat"
3 in [1, 2, 3]
```

---

## 1.5 Bitwise Operators

```python
a & b   # AND
a | b   # OR
a ^ b   # XOR
~a      # NOT
a << 2  # left shift
a >> 2  # right shift
```

Example:

```python
a = 5       # 101
b = 3       # 011

print(a & b)  # 1
print(a | b)  # 7
print(a ^ b)  # 6
```

### Common Bitwise Tricks

```python
# Check odd/even
if n & 1:
    print("Odd")

# Multiply by 2
x << 1

# Divide by 2
x >> 1

# Check/set/clear bits
```

---

## 1.6 Control Flow

### if / elif / else

```python
marks = 75

if marks >= 90:
    grade = "A"
elif marks >= 60:
    grade = "B"
else:
    grade = "C"
```

### for Loop

```python
for i in range(5):
    print(i)
```

Output:

```text
0
1
2
3
4
```

### range()

```python
range(stop)
range(start, stop)
range(start, stop, step)
```

```python
for i in range(2, 10, 2):
    print(i)
```

### while Loop

```python
i = 0

while i < 5:
    print(i)
    i += 1
```

### break

```python
for i in range(10):
    if i == 5:
        break
```

### continue

```python
for i in range(5):
    if i == 2:
        continue
    print(i)
```

### Best Practices / Pitfalls

- Use meaningful variable names.
- Prefer `is None` instead of `== None`.
- Avoid unnecessary type conversions.
- Remember that Python uses indentation instead of braces.
- Be careful with mutable default arguments.

---

# 2. Data Structures

## 2.1 Lists

Lists are ordered and mutable.

```python
nums = [10, 20, 30]
```

### Indexing

```python
nums[0]
nums[-1]
```

### Slicing

```python
nums[start:end:step]
```

```python
nums = [0, 1, 2, 3, 4]

print(nums[1:4])   # [1, 2, 3]
print(nums[::-1])  # reverse
```

### Common Methods

```python
nums.append(40)
nums.extend([50, 60])
nums.insert(1, 15)

nums.remove(20)
x = nums.pop()
nums.clear()
```

### Sorting

```python
nums.sort()
nums.sort(reverse=True)

new_nums = sorted(nums)
```

`sort()` modifies the list.

`sorted()` returns a new sorted object.

---

## 2.2 Tuples

Tuples are ordered and immutable.

```python
point = (10, 20)
```

Useful for fixed collections and unpacking.

```python
x, y = point
```

A tuple can contain mutable objects:

```python
data = ([1, 2], 3)
data[0].append(4)
```

---

## 2.3 Sets

Sets store unique elements.

```python
s = {1, 2, 3, 3}

print(s)  # {1, 2, 3}
```

### Common Operations

```python
a = {1, 2, 3}
b = {3, 4, 5}

a | b   # union
a & b   # intersection
a - b   # difference
a ^ b   # symmetric difference
```

### Methods

```python
s.add(10)
s.remove(10)
s.discard(20)
```

`remove()` raises `KeyError` if the item is absent.

`discard()` does not.

---

## 2.4 Dictionaries

Dictionaries store key-value pairs.

```python
user = {
    "name": "Alice",
    "age": 22
}
```

### Access

```python
user["name"]

user.get("name")
user.get("email", "Not found")
```

### Common Methods

```python
user.keys()
user.values()
user.items()

user.update({"age": 23})
user.pop("age")
```

### Iteration

```python
for key, value in user.items():
    print(key, value)
```

### Dictionary Membership

```python
if "name" in user:
    print("Exists")
```

Dictionary keys must be hashable.

---

## 2.5 List Comprehensions

Instead of:

```python
squares = []

for i in range(10):
    squares.append(i * i)
```

Use:

```python
squares = [i * i for i in range(10)]
```

With condition:

```python
even = [x for x in range(20) if x % 2 == 0]
```

Nested:

```python
pairs = [(x, y) for x in range(3) for y in range(3)]
```

---

## 2.6 Dictionary Comprehensions

```python
squares = {x: x * x for x in range(5)}
```

With condition:

```python
even_squares = {
    x: x * x
    for x in range(10)
    if x % 2 == 0
}
```

### Best Practices / Pitfalls

- Use lists for ordered mutable collections.
- Use tuples for immutable structured data.
- Use sets when uniqueness and fast membership are important.
- Use dictionaries for key-value lookup.
- Avoid modifying a collection while iterating over it.
- Do not use a mutable list/dict as a default function argument.

---

# 3. Functions & OOP

## 3.1 Functions

```python
def add(a, b):
    return a + b

result = add(2, 3)
```

### Default Arguments

```python
def greet(name="Guest"):
    return f"Hello {name}"
```

### Keyword Arguments

```python
def introduce(name, age):
    print(name, age)

introduce(age=22, name="Alice")
```

---

## 3.2 *args

`*args` collects positional arguments into a tuple.

```python
def total(*args):
    return sum(args)

print(total(1, 2, 3, 4))
```

---

## 3.3 **kwargs

`**kwargs` collects keyword arguments into a dictionary.

```python
def show_info(**kwargs):
    for key, value in kwargs.items():
        print(key, value)

show_info(name="Alice", age=22)
```

Both can be used:

```python
def func(*args, **kwargs):
    print(args)
    print(kwargs)
```

---

## 3.4 Lambda Functions

A lambda is a small anonymous function.

```python
square = lambda x: x * x

print(square(5))
```

Commonly used with sorting:

```python
students = [
    ("Alice", 90),
    ("Bob", 80),
    ("Charlie", 95)
]

students.sort(key=lambda x: x[1], reverse=True)
```

---

## 3.5 Decorators

A decorator modifies or extends the behavior of a function.

```python
def logger(func):
    def wrapper(*args, **kwargs):
        print("Function called")
        result = func(*args, **kwargs)
        print("Function finished")
        return result

    return wrapper
```

Use it:

```python
@logger
def add(a, b):
    return a + b
```

Equivalent to:

```python
add = logger(add)
```

Use `functools.wraps` in production decorators:

```python
from functools import wraps

def logger(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("Calling function")
        return func(*args, **kwargs)

    return wrapper
```

---

# 3.6 Classes and Objects

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Hello, I am {self.name}"

p = Person("Alice", 22)
print(p.greet())
```

`self` refers to the current object.

---

## 3.7 Class Variables

```python
class Person:
    species = "Human"

    def __init__(self, name):
        self.name = name
```

`species` is shared by instances.

---

## 3.8 Inheritance

```python
class Animal:
    def speak(self):
        print("Animal sound")


class Dog(Animal):
    def speak(self):
        print("Bark")
```

```python
dog = Dog()
dog.speak()
```

---

## 3.9 Polymorphism

Different objects can provide the same interface.

```python
class Dog:
    def speak(self):
        return "Bark"


class Cat:
    def speak(self):
        return "Meow"


for animal in [Dog(), Cat()]:
    print(animal.speak())
```

Python commonly uses duck typing:

> If an object behaves like the required object, its exact class may not matter.

---

## 3.10 Special Methods

### __init__

Called when an object is initialized.

```python
class User:
    def __init__(self, name):
        self.name = name
```

### __str__

Used for a user-friendly string representation.

```python
class User:
    def __init__(self, name):
        self.name = name

    def __str__(self):
        return self.name
```

### __repr__

Used for a developer-oriented representation.

```python
def __repr__(self):
    return f"User(name={self.name!r})"
```

### Other Useful Dunder Methods

```python
__len__
__getitem__
__setitem__
__eq__
__lt__
__add__
__enter__
__exit__
```

### Best Practices / Pitfalls

- Keep classes focused on one responsibility.
- Prefer composition when inheritance is not naturally appropriate.
- Use `@property` when controlled attribute access is useful.
- Use `super()` to access parent-class behavior.
- Avoid overly deep inheritance hierarchies.

---

# 4. Error Handling

## 4.1 try / except

```python
try:
    x = int(input("Enter number: "))
    print(10 / x)

except ValueError:
    print("Invalid number")

except ZeroDivisionError:
    print("Cannot divide by zero")
```

Catch specific exceptions rather than using a broad `except`.

---

## 4.2 finally

`finally` executes whether an exception occurs or not.

```python
try:
    file = open("data.txt")
except FileNotFoundError:
    print("File not found")
finally:
    print("Cleanup")
```

---

## 4.3 else

`else` runs only when no exception occurs.

```python
try:
    result = 10 / 2
except ZeroDivisionError:
    print("Error")
else:
    print(result)
finally:
    print("Done")
```

---

## 4.4 Raising Exceptions

```python
def withdraw(balance, amount):
    if amount > balance:
        raise ValueError("Insufficient balance")

    return balance - amount
```

---

## 4.5 Custom Exceptions

```python
class InsufficientBalanceError(Exception):
    pass


def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientBalanceError("Insufficient balance")
```

Use custom exceptions when they make error handling clearer.

### Best Practices / Pitfalls

- Catch only exceptions you can meaningfully handle.
- Do not silently ignore errors.
- Avoid `except Exception:` unless there is a deliberate reason.
- Preserve useful context when re-raising exceptions.

---

# 5. Iterators & Generators

## 5.1 Iterables vs Iterators

An iterable can produce an iterator.

```python
nums = [1, 2, 3]

iterator = iter(nums)

print(next(iterator))
print(next(iterator))
print(next(iterator))
```

After exhaustion:

```python
next(iterator)  # StopIteration
```

A `for` loop internally uses the iterator protocol.

---

## 5.2 Custom Iterator

```python
class Counter:
    def __init__(self, limit):
        self.current = 0
        self.limit = limit

    def __iter__(self):
        return self

    def __next__(self):
        if self.current >= self.limit:
            raise StopIteration

        value = self.current
        self.current += 1
        return value
```

---

## 5.3 Generators

Generators use `yield`.

```python
def numbers(n):
    for i in range(n):
        yield i
```

```python
for x in numbers(5):
    print(x)
```

Generators produce values lazily.

### Generator Expression

```python
squares = (x * x for x in range(1000000))
```

This avoids creating a million-element list immediately.

---

## 5.4 itertools

`itertools` provides efficient iterator building blocks.

```python
from itertools import chain, combinations, permutations

a = [1, 2]
b = [3, 4]

print(list(chain(a, b)))
```

### Common Functions

```python
chain()
count()
cycle()
repeat()
islice()
product()
permutations()
combinations()
combinations_with_replacement()
groupby()
```

### Best Practices / Pitfalls

- Use generators for large or streaming datasets.
- Remember that generators are generally consumed once.
- Do not convert huge generators into lists unnecessarily.

---

# 6. File Handling

## 6.1 Reading Text Files

```python
with open("data.txt", "r", encoding="utf-8") as file:
    content = file.read()

print(content)
```

Read line-by-line:

```python
with open("data.txt", "r", encoding="utf-8") as file:
    for line in file:
        print(line.strip())
```

---

## 6.2 Writing Files

```python
with open("output.txt", "w", encoding="utf-8") as file:
    file.write("Hello Python\n")
```

Append:

```python
with open("output.txt", "a", encoding="utf-8") as file:
    file.write("New line\n")
```

### File Modes

| Mode | Meaning |
|---|---|
| `r` | Read |
| `w` | Write/overwrite |
| `a` | Append |
| `x` | Create new file |
| `b` | Binary |
| `+` | Read and write |

---

## 6.3 Binary Files

```python
with open("image.jpg", "rb") as file:
    data = file.read()
```

Writing binary:

```python
with open("copy.jpg", "wb") as file:
    file.write(data)
```

---

## 6.4 Context Managers

Prefer:

```python
with open("data.txt") as file:
    data = file.read()
```

The context manager automatically closes the file.

Custom context managers can implement:

```python
__enter__()
__exit__()
```

### Best Practices / Pitfalls

- Use `with open(...)`.
- Specify encoding for text files when appropriate.
- Avoid loading very large files entirely into memory.
- Use binary mode for images, PDFs, and other binary data.

---

# 7. JSON & Serialization

## 7.1 JSON

Python's `json` module converts between Python objects and JSON.

```python
import json
```

---

## 7.2 json.loads()

JSON string -> Python object.

```python
data = '{"name": "Alice", "age": 22}'

obj = json.loads(data)

print(obj["name"])
```

---

## 7.3 json.dumps()

Python object -> JSON string.

```python
data = {
    "name": "Alice",
    "age": 22
}

text = json.dumps(data)
print(text)
```

---

## 7.4 json.load()

Read JSON from a file.

```python
with open("data.json", "r") as file:
    data = json.load(file)
```

---

## 7.5 json.dump()

Write JSON to a file.

```python
data = {
    "name": "Alice",
    "age": 22
}

with open("data.json", "w") as file:
    json.dump(data, file)
```

---

## 7.6 Pretty Printing

```python
print(json.dumps(data, indent=4))
```

Sort keys:

```python
json.dumps(data, indent=4, sort_keys=True)
```

---

## 7.7 Pickle

`pickle` serializes Python objects into a Python-specific binary format.

```python
import pickle

data = {"name": "Alice", "age": 22}

with open("data.pkl", "wb") as file:
    pickle.dump(data, file)
```

Load:

```python
with open("data.pkl", "rb") as file:
    data = pickle.load(file)
```

### Critical Security Warning

Never unpickle untrusted data. Pickle can execute arbitrary code during deserialization.

### JSON vs Pickle

| JSON | Pickle |
|---|---|
| Language-independent | Python-specific |
| Human-readable | Binary |
| Good for APIs/config | Good for Python objects |
| Safer for untrusted structured data | Never load untrusted pickle |

---

# 8. Working with APIs

## 8.1 requests

Install:

```bash
pip install requests
```

GET request:

```python
import requests

response = requests.get(
    "https://api.example.com/users"
)

print(response.status_code)
print(response.json())
```

---

## 8.2 Query Parameters

```python
params = {
    "page": 1,
    "limit": 10
}

response = requests.get(
    "https://api.example.com/users",
    params=params
)
```

---

## 8.3 Headers

```python
headers = {
    "Authorization": "Bearer TOKEN",
    "Accept": "application/json"
}

response = requests.get(
    "https://api.example.com/users",
    headers=headers
)
```

---

## 8.4 POST Request

JSON body:

```python
payload = {
    "name": "Alice",
    "age": 22
}

response = requests.post(
    "https://api.example.com/users",
    json=payload
)
```

Form data:

```python
requests.post(
    url,
    data={"name": "Alice"}
)
```

---

## 8.5 Authentication

Common approaches include:

```python
headers = {
    "Authorization": "Bearer <token>"
}
```

Basic authentication:

```python
from requests.auth import HTTPBasicAuth

requests.get(
    url,
    auth=HTTPBasicAuth("username", "password")
)
```

Never hardcode secrets in source code.

Use environment variables:

```python
import os

API_KEY = os.getenv("API_KEY")
```

---

## 8.6 Parsing JSON

```python
response = requests.get(url)

if response.ok:
    data = response.json()
    print(data)
```

---

## 8.7 Error Handling

```python
import requests

try:
    response = requests.get(
        url,
        timeout=10
    )

    response.raise_for_status()

    data = response.json()

except requests.exceptions.Timeout:
    print("Request timed out")

except requests.exceptions.HTTPError as e:
    print("HTTP error:", e)

except requests.exceptions.RequestException as e:
    print("Request failed:", e)
```

### Useful status codes

| Code | Meaning |
|---|---|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 429 | Too Many Requests |
| 500 | Server Error |

### Best Practices / Pitfalls

- Always use timeouts for network requests.
- Handle HTTP errors.
- Do not expose API keys.
- Validate external data.
- Implement retries carefully for transient failures.

---

# 9. Data Science Essentials

# 9.1 NumPy

Install:

```bash
pip install numpy
```

Import:

```python
import numpy as np
```

Create an array:

```python
arr = np.array([1, 2, 3, 4])
```

2D array:

```python
matrix = np.array([
    [1, 2],
    [3, 4]
])
```

---

## Array Operations

```python
arr + 10
arr * 2
arr ** 2
```

NumPy performs vectorized operations.

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

print(a + b)
```

---

## Broadcasting

Broadcasting allows operations between compatible shapes.

```python
arr = np.array([
    [1, 2, 3],
    [4, 5, 6]
])

print(arr + 10)
```

The scalar is effectively applied to every element.

Example:

```python
a = np.array([
    [1, 2, 3],
    [4, 5, 6]
])

b = np.array([10, 20, 30])

print(a + b)
```

---

## Useful NumPy Functions

```python
np.zeros((2, 3))
np.ones((2, 3))
np.arange(0, 10, 2)
np.linspace(0, 1, 5)

np.mean(arr)
np.median(arr)
np.std(arr)
np.sum(arr)
np.min(arr)
np.max(arr)
```

### Best Practices / Pitfalls

- Prefer vectorized NumPy operations over Python loops for numerical workloads.
- Understand array shapes.
- Distinguish element-wise multiplication (`*`) from matrix multiplication (`@`).

---

# 9.2 Pandas

Install:

```bash
pip install pandas
```

```python
import pandas as pd
```

Create DataFrame:

```python
df = pd.DataFrame({
    "name": ["Alice", "Bob", "Charlie"],
    "age": [22, 23, 21],
    "score": [90, 80, 95]
})
```

---

## Indexing

```python
df["name"]

df[["name", "score"]]
```

Using `loc`:

```python
df.loc[0, "name"]
```

Using `iloc`:

```python
df.iloc[0, 0]
```

Filtering:

```python
df[df["score"] > 85]
```

---

## Grouping

```python
df.groupby("department")["salary"].mean()
```

Multiple aggregations:

```python
df.groupby("department").agg({
    "salary": "mean",
    "age": "max"
})
```

---

## Merging

```python
result = pd.merge(
    users,
    orders,
    on="user_id",
    how="inner"
)
```

Common join types:

```text
inner
left
right
outer
```

---

## Missing Data

```python
df.isna()
df.dropna()
df.fillna(0)
```

### Best Practices / Pitfalls

- Avoid chained assignment.
- Inspect data types using `df.dtypes`.
- Check missing values before modeling.
- Avoid accidental data leakage during preprocessing.

---

# 9.3 Matplotlib

```python
import matplotlib.pyplot as plt

x = [1, 2, 3, 4]
y = [10, 20, 15, 30]

plt.plot(x, y)
plt.xlabel("X")
plt.ylabel("Y")
plt.title("Example")
plt.show()
```

Common plots:

```python
plt.plot()
plt.scatter()
plt.bar()
plt.hist()
```

---

# 9.4 Seaborn

```python
import seaborn as sns

sns.scatterplot(
    data=df,
    x="age",
    y="score"
)
```

Useful plots:

```python
sns.histplot()
sns.boxplot()
sns.heatmap()
sns.countplot()
sns.scatterplot()
```

### Pitfall

Visualization libraries are for analysis and communication; do not assume correlation implies causation.

---

# 9.5 Scikit-learn

Install:

```bash
pip install scikit-learn
```

Basic workflow:

```python
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score

X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    random_state=42
)

scaler = StandardScaler()

X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)

model = LogisticRegression()

model.fit(X_train, y_train)

predictions = model.predict(X_test)

print(accuracy_score(y_test, predictions))
```

### Important Rule

Fit preprocessing only on training data.

Correct:

```python
scaler.fit(X_train)
X_train = scaler.transform(X_train)
X_test = scaler.transform(X_test)
```

Avoid fitting on the entire dataset because that can cause data leakage.

---

## Common Scikit-learn Models

### Regression

```python
from sklearn.linear_model import LinearRegression
```

### Classification

```python
from sklearn.linear_model import LogisticRegression
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier
```

### Clustering

```python
from sklearn.cluster import KMeans
```

### Dimensionality Reduction

```python
from sklearn.decomposition import PCA
```

---

# 10. Advanced Python

# 10.1 asyncio

`asyncio` is Python's framework for asynchronous programming.

Useful for I/O-bound tasks such as network operations.

```python
import asyncio

async def hello():
    print("Hello")
    await asyncio.sleep(1)
    print("World")

asyncio.run(hello())
```

Concurrent tasks:

```python
async def task(name):
    print(f"Starting {name}")
    await asyncio.sleep(1)
    print(f"Finished {name}")

async def main():
    await asyncio.gather(
        task("A"),
        task("B"),
        task("C")
    )

asyncio.run(main())
```

---

# 10.2 Multithreading

Threads are useful for many I/O-bound workloads.

```python
import threading

def worker():
    print("Working")

thread = threading.Thread(target=worker)

thread.start()
thread.join()
```

### Important

CPython has the Global Interpreter Lock (GIL), which limits simultaneous execution of Python bytecode by multiple threads in the traditional CPython execution model.

Threads are commonly useful for I/O-bound tasks.

---

# 10.3 Multiprocessing

Multiprocessing uses separate processes.

```python
from multiprocessing import Process

def worker():
    print("Working")

p = Process(target=worker)

p.start()
p.join()
```

Useful for CPU-heavy workloads when process-level parallelism is appropriate.

---

## Concurrency Quick Comparison

| Approach | Typical Use |
|---|---|
| `asyncio` | Async I/O |
| Threading | I/O-bound work |
| Multiprocessing | CPU-bound work |

### Pitfalls

- Do not assume async automatically makes CPU-heavy code faster.
- Be careful with shared state.
- Understand synchronization and process/thread communication.

---

# 10.4 Regular Expressions

```python
import re

text = "My phone is 9876543210"

match = re.search(r"\d+", text)

if match:
    print(match.group())
```

### Common Functions

```python
re.search()
re.match()
re.findall()
re.finditer()
re.sub()
re.split()
```

### Common Patterns

```text
\d    digit
\w    word character
\s    whitespace
.     any character except newline
^     start
$     end
+     one or more
*     zero or more
?     zero or one
{n}   exactly n
```

Example:

```python
pattern = r"^\d{10}$"

print(bool(re.match(pattern, "9876543210")))
```

### Best Practices

- Use raw strings for regex patterns: `r"\d+"`.
- Compile frequently reused patterns.

```python
pattern = re.compile(r"\d+")

pattern.findall("123 abc 456")
```

---

# 10.5 Logging

Use logging instead of `print()` for application diagnostics.

```python
import logging

logging.basicConfig(
    level=logging.INFO
)

logging.info("Application started")
logging.warning("Something may be wrong")
logging.error("An error occurred")
```

### Levels

```text
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

### Better Configuration

```python
logger = logging.getLogger(__name__)

logger.info("Processing request")
```

In larger applications, configure handlers, formatters, and log destinations centrally.

### Pitfalls

- Never log passwords, tokens, API keys, or sensitive data.
- Avoid excessive logging in hot paths.
- Include useful context such as request IDs where appropriate.

---

# 10.6 Virtual Environments

Create:

```bash
python -m venv .venv
```

Windows activation:

```bash
.venv\Scripts\activate
```

Linux/macOS:

```bash
source .venv/bin/activate
```

Install packages:

```bash
pip install requests numpy pandas
```

Freeze dependencies:

```bash
pip freeze > requirements.txt
```

Install later:

```bash
pip install -r requirements.txt
```

Deactivate:

```bash
deactivate
```

---

## Conda

Create environment:

```bash
conda create -n ml python=3.12
```

Activate:

```bash
conda activate ml
```

### Best Practices

- Use one environment per project when practical.
- Pin dependencies for reproducible deployments.
- Do not commit `.venv/`.
- Keep secrets out of environment files that are committed to Git.

---

# 11. AI/ML Engineering

# 11.1 PyTorch Basics

Install:

```bash
pip install torch
```

Import:

```python
import torch
```

Tensor:

```python
x = torch.tensor([
    [1.0, 2.0],
    [3.0, 4.0]
])
```

Operations:

```python
y = x * 2
z = x @ x
```

---

## Neural Network

```python
import torch.nn as nn

class Model(nn.Module):
    def __init__(self):
        super().__init__()

        self.network = nn.Sequential(
            nn.Linear(10, 32),
            nn.ReLU(),
            nn.Linear(32, 1)
        )

    def forward(self, x):
        return self.network(x)
```

---

# 11.2 Training Loop

```python
model = Model()

criterion = nn.MSELoss()

optimizer = torch.optim.Adam(
    model.parameters(),
    lr=0.001
)

for epoch in range(100):
    optimizer.zero_grad()

    predictions = model(X)

    loss = criterion(predictions, y)

    loss.backward()

    optimizer.step()

    print(loss.item())
```

### Training Steps

```text
1. Forward pass
2. Calculate loss
3. Clear old gradients
4. Backpropagation
5. Update parameters
6. Repeat
```

---

# 11.3 Optimizers

Common optimizers:

```python
torch.optim.SGD()
torch.optim.Adam()
torch.optim.AdamW()
```

Example:

```python
optimizer = torch.optim.AdamW(
    model.parameters(),
    lr=1e-3
)
```

---

# 11.4 Loss Functions

### Regression

```python
nn.MSELoss()
nn.L1Loss()
```

### Classification

```python
nn.CrossEntropyLoss()
nn.BCEWithLogitsLoss()
```

Important:

`CrossEntropyLoss` generally expects raw logits rather than manually applied softmax probabilities.

---

# 11.5 Evaluation Mode

Training:

```python
model.train()
```

Evaluation:

```python
model.eval()
```

For inference:

```python
model.eval()

with torch.no_grad():
    predictions = model(X_test)
```

---

# 11.6 Saving and Loading PyTorch Models

Recommended approach:

```python
torch.save(
    model.state_dict(),
    "model.pth"
)
```

Load:

```python
model = Model()

model.load_state_dict(
    torch.load("model.pth", weights_only=True)
)

model.eval()
```

Save checkpoints:

```python
torch.save({
    "epoch": epoch,
    "model_state": model.state_dict(),
    "optimizer_state": optimizer.state_dict()
}, "checkpoint.pth")
```

---

# 11.7 TensorFlow / Keras

```python
import tensorflow as tf

model = tf.keras.Sequential([
    tf.keras.layers.Dense(32, activation="relu"),
    tf.keras.layers.Dense(1)
])
```

Compile:

```python
model.compile(
    optimizer="adam",
    loss="mse",
    metrics=["mae"]
)
```

Train:

```python
model.fit(
    X_train,
    y_train,
    epochs=10,
    validation_split=0.2
)
```

Evaluate:

```python
model.evaluate(X_test, y_test)
```

Predict:

```python
predictions = model.predict(X_test)
```

Save:

```python
model.save("model.keras")
```

Load:

```python
model = tf.keras.models.load_model("model.keras")
```

---

# 11.8 ML Model Development Pipeline

A typical ML workflow:

```text
Raw Data
   |
   v
Data Cleaning
   |
   v
Exploratory Data Analysis
   |
   v
Feature Engineering
   |
   v
Train/Validation/Test Split
   |
   v
Preprocessing
   |
   v
Model Training
   |
   v
Evaluation
   |
   v
Hyperparameter Tuning
   |
   v
Model Serialization
   |
   v
Deployment
   |
   v
Monitoring
```

### Important Principle

Do not allow information from the test set to influence training or preprocessing decisions.

---

# 11.9 FastAPI

FastAPI is commonly used to expose ML models as APIs.

Install:

```bash
pip install fastapi uvicorn
```

Example:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def home():
    return {"message": "ML API"}

@app.post("/predict")
def predict(features: dict):
    # model inference
    return {"prediction": 1}
```

Run:

```bash
uvicorn main:app --reload
```

FastAPI automatically provides API documentation.

---

## Request Validation with Pydantic

```python
from pydantic import BaseModel

class PredictionRequest(BaseModel):
    age: float
    income: float


@app.post("/predict")
def predict(request: PredictionRequest):
    return {
        "age": request.age,
        "income": request.income
    }
```

### Production Considerations

- Validate inputs.
- Load the model once during application startup rather than per request.
- Use authentication when required.
- Add logging and monitoring.
- Handle model errors gracefully.
- Containerize the service when appropriate.

---

# 11.10 Flask

Minimal Flask application:

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/predict", methods=["POST"])
def predict():
    data = request.get_json()

    return jsonify({
        "prediction": 1
    })

if __name__ == "__main__":
    app.run()
```

FastAPI is often preferred when automatic API schemas, validation, and modern async support are useful.

---

# Python Interview Quick Revision

## Core Concepts

Know these well:

```text
Mutable vs immutable
List vs tuple
Set vs dictionary
== vs is
Shallow copy vs deep copy
List comprehensions
*args / **kwargs
Lambda
Decorators
Generators
Iterators
Context managers
Exception handling
```

## OOP

Know:

```text
Class
Object
Inheritance
Polymorphism
Encapsulation
Abstraction
Method overriding
super()
self
classmethod
staticmethod
@property
Dunder methods
```

## Advanced Python

Know:

```text
GIL
Threading
Multiprocessing
asyncio
Generators
Iterators
Closures
Decorators
Context managers
Memory management
Garbage collection
```

## Backend Python

Know:

```text
requests
HTTP methods
JSON
REST APIs
Authentication
FastAPI
Flask
Pydantic
Logging
Environment variables
Virtual environments
```

## ML Python

Know:

```text
NumPy
Broadcasting
Pandas
DataFrames
Matplotlib
Scikit-learn
Train/test split
Feature scaling
Pipelines
Data leakage
PyTorch
TensorFlow
Training loops
Optimizers
Loss functions
Model serialization
FastAPI model serving
```

---

# Common Python Pitfalls

## Mutable Default Arguments

Avoid:

```python
def add_item(item, items=[]):
    items.append(item)
    return items
```

Use:

```python
def add_item(item, items=None):
    if items is None:
        items = []

    items.append(item)
    return items
```

---

## Shallow vs Deep Copy

```python
import copy

a = [[1, 2], [3, 4]]

b = copy.copy(a)
c = copy.deepcopy(a)
```

A shallow copy copies the outer object but may share nested objects.

A deep copy recursively copies nested objects.

---

## Late Binding in Closures

Be careful with closures inside loops:

```python
funcs = [
    lambda: i
    for i in range(3)
]
```

The lambdas may all observe the final value of `i`.

One solution:

```python
funcs = [
    lambda i=i: i
    for i in range(3)
]
```

---

## `is` vs `==`

Use:

```python
a == b
```

for value equality.

Use:

```python
a is b
```

for object identity.

Typical singleton check:

```python
if value is None:
    ...
```

---

## List Aliasing

This does not create independent lists:

```python
a = [1, 2, 3]
b = a

b.append(4)

print(a)  # [1, 2, 3, 4]
```

Use:

```python
b = a.copy()
```

for a shallow copy.

---

# Python Performance Tips

## Prefer Built-ins

Instead of manually implementing common operations:

```python
sum(nums)
max(nums)
min(nums)
sorted(nums)
```

Use optimized built-in functions when appropriate.

## Use Sets for Membership

```python
values = {1, 2, 3, 4}

if x in values:
    ...
```

Average-case membership is approximately O(1).

## Use Dictionaries for Lookup

```python
frequency = {}

for x in nums:
    frequency[x] = frequency.get(x, 0) + 1
```

## Use Generators for Large Streams

```python
total = sum(
    x * x
    for x in range(10_000_000)
)
```

Avoid creating an unnecessary giant list.

---

# Useful Standard Library Modules

```python
import collections
import itertools
import functools
import math
import statistics
import pathlib
import os
import sys
import json
import csv
import re
import logging
import datetime
import time
import random
import heapq
import bisect
```

Especially useful for competitive programming:

```python
from collections import Counter, defaultdict, deque
import heapq
import bisect
import itertools
```

Example:

```python
from collections import Counter

freq = Counter([1, 2, 2, 3, 3, 3])

print(freq)
```

---

# Python Cheat Sheet

## Lists

```python
append()
extend()
insert()
remove()
pop()
clear()
sort()
reverse()
index()
count()
```

## Dictionaries

```python
get()
keys()
values()
items()
update()
pop()
setdefault()
```

## Sets

```python
add()
remove()
discard()
union()
intersection()
difference()
```

## Strings

```python
split()
join()
strip()
replace()
startswith()
endswith()
find()
count()
lower()
upper()
```

## Useful Built-ins

```python
len()
sum()
min()
max()
sorted()
reversed()
enumerate()
zip()
map()
filter()
any()
all()
abs()
round()
range()
```

Example:

```python
for i, value in enumerate(["a", "b", "c"]):
    print(i, value)
```

---

# Final Revision Checklist

Before an interview or ML/backend project, make sure you can explain and implement:

- [ ] Python variables and object references
- [ ] Mutable vs immutable objects
- [ ] Lists, tuples, sets, dictionaries
- [ ] Comprehensions
- [ ] Functions and argument passing
- [ ] `*args` and `**kwargs`
- [ ] Lambda functions
- [ ] Decorators
- [ ] Classes and inheritance
- [ ] Polymorphism
- [ ] Dunder methods
- [ ] Exception handling
- [ ] Iterators and generators
- [ ] Context managers
- [ ] File handling
- [ ] JSON serialization
- [ ] Pickle security
- [ ] REST APIs with `requests`
- [ ] NumPy and broadcasting
- [ ] Pandas
- [ ] Matplotlib/Seaborn
- [ ] Scikit-learn workflow
- [ ] Data leakage
- [ ] `asyncio`
- [ ] Threading vs multiprocessing
- [ ] GIL
- [ ] Regular expressions
- [ ] Logging
- [ ] Virtual environments
- [ ] PyTorch tensors and models
- [ ] Training loops
- [ ] Optimizers and loss functions
- [ ] Model saving/loading
- [ ] TensorFlow/Keras
- [ ] FastAPI
- [ ] Flask
- [ ] ML model deployment

---

# One-Page Mental Model

```text
Python
│
├── Fundamentals
│   ├── Variables
│   ├── Types
│   ├── Operators
│   └── Control Flow
│
├── Data Structures
│   ├── List
│   ├── Tuple
│   ├── Set
│   └── Dictionary
│
├── Functions
│   ├── Arguments
│   ├── Lambda
│   ├── Decorators
│   └── Generators
│
├── OOP
│   ├── Classes
│   ├── Inheritance
│   ├── Polymorphism
│   └── Dunder Methods
│
├── Systems / Backend
│   ├── Files
│   ├── JSON
│   ├── Requests
│   ├── Logging
│   ├── Asyncio
│   └── FastAPI / Flask
│
├── Data Science
│   ├── NumPy
│   ├── Pandas
│   ├── Matplotlib
│   └── Scikit-learn
│
└── AI / ML
    ├── PyTorch
    ├── TensorFlow
    ├── Training
    ├── Evaluation
    ├── Serialization
    └── Deployment
```

This guide is intended as a revision reference. For deeper mastery, implement each example yourself and solve small problems after each topic.
