# Python — Quick Revision Cheat Sheet

> **Purpose:** Revise Python fundamentals, built-in data structures, functions, object-oriented programming, exceptions, iterators, and coding interview essentials for Data Engineer and product-company interviews.

## 1. Python Fundamentals

### Variables and Data Types

```python
name = "Yash"
age = 26
salary = 90000.0
is_active = True
value = None
```

| Type | Example | Mutable? |
|---|---|---|
| `int` | `10` | No |
| `float` | `10.5` | No |
| `bool` | `True` | No |
| `str` | `"Python"` | No |
| `list` | `[1, 2, 3]` | Yes |
| `tuple` | `(1, 2, 3)` | No |
| `set` | `{1, 2, 3}` | Yes |
| `dict` | `{"id": 1}` | Yes |

Python is dynamically typed. A variable name refers to an object; it does not permanently declare a fixed type.

### Type Conversion

```python
int("42")
float("3.14")
str(100)
list((1, 2, 3))
set([1, 1, 2])  # {1, 2}
```

Conversions can fail when the input is invalid, such as `int("hello")`.

### Operators

| Category | Operators |
|---|---|
| Arithmetic | `+`, `-`, `*`, `/`, `//`, `%`, `**` |
| Comparison | `==`, `!=`, `<`, `>`, `<=`, `>=` |
| Logical | `and`, `or`, `not` |
| Membership | `in`, `not in` |
| Identity | `is`, `is not` |
| Bitwise | `&`, `|`, `^`, `~`, `<<`, `>>` |

Important distinctions:

- `/` performs true division.
- `//` performs floor division.
- `%` returns the remainder under Python's modulo rules.
- `==` compares values.
- `is` checks object identity.
- Use `value is None`, not `value == None`.

---

## 2. Strings

Strings are immutable sequences of Unicode characters.

```python
text = "Data Engineering"

text.lower()
text.upper()
text.strip()
text.replace("Data", "Cloud")
text.split()
"-".join(["data", "engineering"])
text.startswith("Data")
text.find("Engineering")
text.count("e")
```

### Indexing and Slicing

```python
s = "Python"

s[0]       # 'P'
s[-1]      # 'n'
s[1:4]     # 'yth'
s[::-1]    # 'nohtyP'
s[:3]      # 'Pyt'
s[3:]      # 'hon'
```

Slicing uses a half-open range: the start is included and the end is excluded.

### String Formatting

```python
name = "Yash"
score = 95

message = f"{name} scored {score}"
```

### Common String Mistakes

- Strings cannot be modified in place.
- Repeated concatenation in a large loop can be inefficient; consider collecting pieces and using `"".join(parts)`.
- `split()` without a separator handles runs of whitespace.
- String indexes are positions, not character values.

---

## 3. Lists

Lists are mutable, ordered sequences.

```python
nums = [10, 20, 30]

nums.append(40)
nums.extend([50, 60])
nums.insert(1, 15)
nums.remove(30)
last = nums.pop()
nums.sort()
nums.reverse()
```

### Common Operations

| Operation | Typical Complexity |
|---|---:|
| Index access | \(O(1)\) |
| Append at end | Amortized \(O(1)\) |
| Pop from end | \(O(1)\) |
| Insert at front | \(O(n)\) |
| Remove from front | \(O(n)\) |
| Search by value | \(O(n)\) |
| Sort | \(O(n\log n)\) |

These are typical CPython complexity characteristics; exact behavior can depend on the implementation and operation.

### `append()` vs `extend()`

```python
a = [1, 2]
a.append([3, 4])
# [1, 2, [3, 4]]

b = [1, 2]
b.extend([3, 4])
# [1, 2, 3, 4]
```

### `sort()` vs `sorted()`

```python
nums = [3, 1, 2]

nums.sort()       # Modifies nums; returns None
result = sorted(nums)  # Returns a new sorted list
```

### List Comprehension

```python
squares = [x * x for x in range(5)]
evens = [x for x in range(10) if x % 2 == 0]
```

Use comprehensions for clear transformations, not excessively complex logic.

---

## 4. Tuples, Sets, and Dictionaries

### Tuple

A tuple is an immutable sequence.

```python
point = (10, 20)
x, y = point
```

A tuple can contain mutable objects; immutability does not recursively freeze everything inside it.

A one-element tuple requires a comma:

```python
single = (5,)
```

### Set

A set stores unique hashable elements.

```python
nums = {1, 2, 3}

nums.add(4)
nums.discard(2)
3 in nums
```

Typical average-case membership: \(O(1)\).

Use sets for:
- Duplicate detection.
- Membership checks.
- Set intersection and union.

```python
a = {1, 2, 3}
b = {3, 4, 5}

a & b  # Intersection
a | b  # Union
a - b  # Difference
```

### Dictionary

A dictionary maps hashable keys to values.

```python
person = {"name": "Yash", "age": 26}

person["name"]
person.get("city", "Unknown")
person["city"] = "Kolkata"
person.pop("age")
```

Common iteration patterns:

```python
for key in person:
    print(key)

for key, value in person.items():
    print(key, value)
```

Frequency counting:

```python
from collections import Counter

counts = Counter(["a", "b", "a"])
# Counter({'a': 2, 'b': 1})
```

**Remember:** Dictionary and set lookups are average-case \(O(1)\), not an unconditional worst-case guarantee. Keys must be hashable.

---

## 5. Copying and Mutability

Assignment creates another reference to the same object; it does not automatically copy the object.

```python
a = [1, 2, 3]
b = a

b[0] = 99
# a is also [99, 2, 3]
```

### Shallow Copy

```python
a = [[1, 2], [3, 4]]
b = a.copy()
```

The outer list is copied, but the nested lists are shared.

### Deep Copy

```python
import copy

b = copy.deepcopy(a)
```

A deep copy recursively copies objects according to Python's copying rules.

### Common Mistake: Mutable Default Arguments

Incorrect:

```python
def add_item(item, items=[]):
    items.append(item)
    return items
```

The same default list is reused across calls.

Correct:

```python
def add_item(item, items=None):
    if items is None:
        items = []

    items.append(item)
    return items
```

---

## 6. Conditions and Loops

```python
if score >= 90:
    grade = "A"
elif score >= 75:
    grade = "B"
else:
    grade = "C"
```

### Loops

```python
for i in range(5):
    print(i)

for index, value in enumerate(["a", "b"]):
    print(index, value)

for a, b in zip([1, 2], ["x", "y"]):
    print(a, b)
```

- `range(stop)` excludes `stop`.
- `enumerate()` provides an index and value.
- `zip()` combines iterables and stops at the shortest one by default.

### Loop Control

- `break`: Exit the nearest loop.
- `continue`: Skip to the next iteration.
- `pass`: Do nothing; used as a placeholder.

---

## 7. Functions

```python
def add(a, b):
    return a + b
```

### Parameters

```python
def greet(name, message="Hello"):
    return f"{message}, {name}"

greet("Yash")
greet("Yash", message="Welcome")
```

### `*args` and `**kwargs`

```python
def example(*args, **kwargs):
    print(args)
    print(kwargs)

example(1, 2, name="Yash")
```

- `*args` collects extra positional arguments into a tuple.
- `**kwargs` collects extra keyword arguments into a dictionary.

### Lambda

```python
square = lambda x: x * x
```

A lambda is a small anonymous function containing a single expression.

### Scope: LEGB

Python resolves ordinary names through:

1. Local
2. Enclosing
3. Global
4. Built-in

Use `nonlocal` for rebinding a name in an enclosing function scope and `global` for rebinding a module-level name.

---

## 8. Iterables, Iterators, and Generators

### Iterable vs Iterator

- **Iterable:** An object from which an iterator can be obtained.
- **Iterator:** An object that produces values through `next()` and retains iteration state.

```python
nums = [10, 20, 30]
iterator = iter(nums)

next(iterator)  # 10
next(iterator)  # 20
```

### Generator

A generator produces values lazily.

```python
def count_up_to(n):
    for i in range(1, n + 1):
        yield i
```

```python
for number in count_up_to(3):
    print(number)
```

### Generator Expression

```python
squares = (x * x for x in range(1_000_000))
```

Advantages:
- Avoids creating the entire result list at once.
- Useful for large or streaming data.
- Values are computed as they are requested.

**Important:** A generator is generally consumed as it is iterated. It is not automatically replayable.

---

## 9. Exception Handling

```python
try:
    value = int("123")
except ValueError:
    value = 0
else:
    print("Conversion succeeded")
finally:
    print("Cleanup runs")
```

- `try`: Code that may raise an exception.
- `except`: Handles selected exceptions.
- `else`: Runs if the `try` block completes without an exception.
- `finally`: Runs during normal or exceptional control flow, subject to process termination and other exceptional circumstances.

### Raise an Exception

```python
def divide(a, b):
    if b == 0:
        raise ValueError("b must not be zero")

    return a / b
```

**Best practice:** Catch specific exceptions. Avoid silently swallowing unexpected failures with a broad `except Exception` unless you have a clear recovery or logging strategy.

---

## 10. File Handling

Use a context manager to close files reliably.

```python
with open("data.txt", "r", encoding="utf-8") as file:
    content = file.read()
```

### Common Modes

| Mode | Purpose |
|---|---|
| `"r"` | Read |
| `"w"` | Write; truncates an existing file |
| `"a"` | Append |
| `"x"` | Create a new file, failing if it exists |
| `"b"` | Binary mode modifier |
| `"+"` | Read and write modifier |

### Read Line by Line

```python
with open("data.txt", encoding="utf-8") as file:
    for line in file:
        print(line.rstrip("\n"))
```

Line-by-line reading can reduce memory usage compared with reading a large file entirely into memory.

---

## 11. Object-Oriented Programming

### Class and Object

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

    def describe(self):
        return f"{self.name}: {self.salary}"
```

```python
employee = Employee("Yash", 90000)
print(employee.describe())
```

- Class: Defines behavior and structure.
- Object: An instance of a class.
- `self`: Conventionally refers to the current instance.
- `__init__`: Initializes an instance after it is created.

### Four Common OOP Concepts

| Concept | Meaning |
|---|---|
| Encapsulation | Bundle state and behavior; control access through an interface |
| Abstraction | Expose essential behavior while hiding implementation details |
| Inheritance | Derive a class from another class |
| Polymorphism | Use a common interface with different implementations |

### Inheritance

```python
class Developer(Employee):
    def __init__(self, name, salary, language):
        super().__init__(name, salary)
        self.language = language
```

`super()` follows Python's method resolution order (MRO); it is not simply a direct call to the textual parent class.

### Class vs Instance Attributes

```python
class Employee:
    company = "Example"

    def __init__(self, name):
        self.name = name
```

- `company` is a class attribute.
- `name` is an instance attribute.

Avoid mutable class attributes when each instance needs independent mutable state.

---

## 12. Useful Built-in Functions

| Function | Purpose |
|---|---|
| `len(x)` | Number of items |
| `sum(x)` | Sum values |
| `min(x)` / `max(x)` | Minimum / maximum |
| `sorted(x)` | Return sorted list |
| `reversed(x)` | Reverse iterator where supported |
| `enumerate(x)` | Index-value pairs |
| `zip(a, b)` | Combine iterables |
| `any(x)` | True if any element is truthy |
| `all(x)` | True if all elements are truthy |
| `map(f, x)` | Apply a function lazily |
| `filter(f, x)` | Filter lazily |
| `isinstance(x, T)` | Check type or subclass relationship |
| `type(x)` | Return the object's type |

### Sorting with a Key

```python
employees = [
    {"name": "A", "salary": 50000},
    {"name": "B", "salary": 70000},
]

employees.sort(key=lambda item: item["salary"], reverse=True)
```

Sorting dictionaries by a selected field is common in data transformation and interview problems.

---

## 13. Useful Standard Library Tools

### `collections`

```python
from collections import Counter, defaultdict, deque
```

- `Counter`: Count hashable values.
- `defaultdict`: Supply a default value for missing keys.
- `deque`: Efficient append and removal at either end.

### `heapq`

```python
import heapq

nums = [5, 1, 8, 2]
heapq.heapify(nums)
smallest = heapq.heappop(nums)
```

Python's `heapq` implements a min-heap.

### `itertools`

```python
from itertools import combinations, permutations, product
```

- `combinations`: Choose items without regard to order.
- `permutations`: Arrange items in order.
- `product`: Cartesian product.

These can generate large numbers of results; estimate output size before using them.

### `functools`

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)
```

Memoization can avoid repeated computation. Cached arguments must be hashable, and unbounded caching can consume significant memory.

---

## 14. Time and Space Complexity

| Operation | Typical Complexity |
|---|---:|
| List index access | \(O(1)\) |
| List membership | \(O(n)\) |
| List append | Amortized \(O(1)\) |
| Insert/delete near front of list | \(O(n)\) |
| Dictionary lookup | Average \(O(1)\) |
| Set membership | Average \(O(1)\) |
| Sorting | \(O(n\log n)\) |
| Binary search on sorted sequence | \(O(\log n)\) |
| Heap push/pop | \(O(\log n)\) |
| Deque append/pop at either end | \(O(1)\) |

### Complexity Reminders

- Nested loops are often \(O(n^2)\), but not always; inspect how many times each loop actually runs.
- Two independent sequential \(O(n)\) loops remain \(O(n)\).
- Sorting followed by a linear scan is usually \(O(n\log n)\).
- Recursive algorithms may use stack space.
- Hash maps and sets require memory proportional to stored elements.

---

## 15. Python for Data Engineering

### Read a CSV with the Standard Library

```python
import csv

with open("orders.csv", newline="", encoding="utf-8") as file:
    reader = csv.DictReader(file)

    for row in reader:
        order_id = row["order_id"]
        amount = float(row["amount"])
```

Production code should validate missing fields, malformed records, numeric ranges, and expected schemas.

### JSON

```python
import json

record = {"order_id": 101, "amount": 500.0}
serialized = json.dumps(record)

restored = json.loads(serialized)
```

### Environment Variables

```python
import os

environment = os.getenv("APP_ENV", "development")
```

Use environment variables or a secret-management system for configuration. Do not hardcode credentials in source code.

### Logging

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

logger.info("Pipeline started")
logger.warning("Unexpected record count")
```

Prefer structured, meaningful logs over scattered debugging prints in production applications.

---

## 16. Python Concurrency Basics

| Approach | Typical Use |
|---|---|
| Threading | I/O-bound tasks |
| Multiprocessing | CPU-bound parallel work, with process overhead |
| `asyncio` | Concurrent asynchronous I/O |
| Sequential execution | Simple workflows or small workloads |

In conventional CPython builds, the Global Interpreter Lock (GIL) limits simultaneous execution of Python bytecode by threads. This does not mean threads cannot overlap I/O, and free-threaded CPython builds are also available in newer Python versions.

Choose concurrency based on workload, libraries, resource constraints, and coordination costs.

---

## 17. Common Python Interview Mistakes

1. Using `is` instead of `==` for value comparison.
2. Using mutable default arguments.
3. Assuming assignment creates a copy.
4. Confusing shallow and deep copies.
5. Forgetting that `list.sort()` returns `None`.
6. Confusing `append()` with `extend()`.
7. Using `list.pop(0)` for a high-volume queue.
8. Modifying a list while iterating over it.
9. Catching all exceptions without a recovery strategy.
10. Forgetting to close files or use a context manager.
11. Assuming dictionary lookup is worst-case \(O(1)\).
12. Forgetting recursion depth and memory constraints.
13. Assuming a generator can be replayed indefinitely.
14. Using mutable class attributes unintentionally.
15. Ignoring input validation and edge cases.
16. Using a list when a set or dictionary better fits the lookup pattern.
17. Assuming threads always accelerate CPU-bound Python code.
18. Forgetting that floating-point values can have representation and rounding errors.

---

## 18. Last-Minute Revision — 25 Key Facts

1. Python is dynamically typed.
2. Strings and tuples are immutable.
3. Lists, sets, and dictionaries are mutable.
4. `==` checks value equality; `is` checks identity.
5. Use `is None` for `None` checks.
6. Dictionary keys and set elements must be hashable.
7. List indexing is typically \(O(1)\).
8. List search is \(O(n)\).
9. List append is amortized \(O(1)\).
10. `append()` adds one object.
11. `extend()` adds items from an iterable.
12. `sort()` modifies the list.
13. `sorted()` returns a new sorted list.
14. Assignment does not automatically copy objects.
15. Shallow copy preserves references to nested objects.
16. Mutable defaults persist across calls.
17. `*args` collects extra positional arguments.
18. `**kwargs` collects extra keyword arguments.
19. Generators produce values lazily.
20. Iterators maintain iteration state.
21. `with` helps manage resources safely.
22. `deque` is suitable for queue operations.
23. `heapq` provides a min-heap.
24. Conventional CPython's GIL affects CPU-bound threading.
25. Complexity analysis must include relevant time and space costs.

---

## 19. Interview Readiness Checklist

- [ ] Explain mutable vs immutable objects.
- [ ] Explain list, tuple, set, and dictionary trade-offs.
- [ ] Write clean functions and use arguments correctly.
- [ ] Explain iterables, iterators, and generators.
- [ ] Explain shallow vs deep copying.
- [ ] Handle exceptions and files safely.
- [ ] Explain classes, inheritance, and `super()`.
- [ ] Use `Counter`, `defaultdict`, `deque`, and `heapq`.
- [ ] Analyze time and auxiliary-space complexity.
- [ ] Solve common array, string, and hashing problems.
- [ ] Explain Python concurrency choices.
- [ ] Apply Python to CSV, JSON, logging, and data processing.

**Final reminder:** Write simple, readable Python first. Then explain why it is correct, test edge cases, and state its time and space complexity.