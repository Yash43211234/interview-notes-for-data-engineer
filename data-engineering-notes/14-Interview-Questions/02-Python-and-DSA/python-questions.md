# Python Interview Questions for Data Engineers

## 1. Python Fundamentals

### 1.1 What is Python?

Python is a high-level, general-purpose programming language known for its readable syntax and extensive libraries.

**Why Data Engineers use Python:**
- Automating data-processing tasks.
- Building ETL and ELT pipelines.
- Working with APIs and files.
- Transforming and validating data.
- Writing PySpark applications.
- Scheduling and monitoring workflows.

### 1.2 Is Python interpreted or compiled?

Python is commonly described as an interpreted language.

In CPython, source code is first compiled into bytecode. The Python virtual machine then executes that bytecode.

### 1.3 What are Python's main data types?

| Type | Example | Mutable? |
|---|---|---|
| int | `10` | No |
| float | `10.5` | No |
| str | `"Python"` | No |
| bool | `True` | No |
| list | `[1, 2, 3]` | Yes |
| tuple | `(1, 2, 3)` | No |
| set | `{1, 2, 3}` | Yes |
| dict | `{"name": "Alex"}` | Yes |
| NoneType | `None` | No |

### 1.4 What is the difference between mutable and immutable objects?

- **Mutable:** The object can be modified after creation.
- **Immutable:** The object cannot be modified after creation.

```python
# Mutable
numbers = [1, 2, 3]
numbers.append(4)

# Immutable
name = "Python"
name = name + " Programming"
```

The second example creates a new string and assigns it to `name`; it does not modify the original string.

### 1.5 What is the difference between `==` and `is`?

- `==` compares values.
- `is` checks whether two references point to the same object.

```python
a = [1, 2, 3]
b = [1, 2, 3]

print(a == b)  # True
print(a is b)  # False
```

Use `is None` when checking whether a value is `None`.

### 1.6 What is type casting?

Type casting converts a value from one data type to another.

```python
number = "100"

converted = int(number)

print(converted)       # 100
print(type(converted)) # <class 'int'>
```

Conversion can fail when the input cannot be represented in the target type.

### 1.7 What is the difference between `break`, `continue`, and `pass`?

| Keyword | Meaning |
|---|---|
| `break` | Exits the nearest loop |
| `continue` | Skips the current iteration |
| `pass` | Does nothing; acts as a placeholder |

```python
for number in range(5):
    if number == 2:
        continue

    if number == 4:
        break

    print(number)
```

Output:

```text
0
1
3
```

---

## 2. Lists, Tuples, Sets, and Dictionaries

### 2.1 List vs Tuple

| Feature | List | Tuple |
|---|---|---|
| Syntax | `[]` | `()` |
| Mutable | Yes | No |
| Add/remove items | Yes | No |
| Typical use | Changing collections | Fixed collections |

```python
numbers = [10, 20, 30]
numbers.append(40)

coordinates = (10, 20)
```

A tuple is immutable, but it can contain mutable objects. Those contained objects may still be modified.

### 2.2 What is a Set?

A set is an unordered collection of unique, hashable elements.

```python
numbers = {1, 2, 2, 3, 3}

print(numbers)  # {1, 2, 3}
```

Common operations:

```python
a = {1, 2, 3}
b = {3, 4, 5}

print(a | b)  # Union
print(a & b)  # Intersection
print(a - b)  # Difference
```

Sets are useful for membership testing and removing duplicate values.

### 2.3 What is a Dictionary?

A dictionary stores data as key-value pairs.

```python
employee = {
    "id": 101,
    "name": "Rahul",
    "salary": 50000
}

print(employee["name"])
```

Important operations:

```python
employee["department"] = "IT"

employee.get("salary")
employee.keys()
employee.values()
employee.items()
```

Dictionary keys must be hashable. Values can be of any type.

### 2.4 What is the difference between `append()` and `extend()`?

- `append()` adds one object to the list.
- `extend()` adds each element from an iterable.

```python
a = [1, 2]
a.append([3, 4])

print(a)
# [1, 2, [3, 4]]

b = [1, 2]
b.extend([3, 4])

print(b)
# [1, 2, 3, 4]
```

### 2.5 What is list slicing?

Slicing extracts a portion of a sequence.

Syntax:

```python
sequence[start:stop:step]
```

Example:

```python
numbers = [10, 20, 30, 40, 50]

print(numbers[1:4])   # [20, 30, 40]
print(numbers[::-1])  # Reversed list
print(numbers[::2])   # [10, 30, 50]
```

The `stop` index is excluded.

### 2.6 What is list comprehension?

List comprehension provides a concise way to create a list.

```python
numbers = [1, 2, 3, 4, 5]

squares = [x * x for x in numbers]

print(squares)
# [1, 4, 9, 16, 25]
```

With a condition:

```python
even_numbers = [x for x in numbers if x % 2 == 0]
```

### 2.7 How do you remove duplicates from a list?

Using a set:

```python
numbers = [1, 2, 2, 3, 3, 4]

unique_numbers = list(set(numbers))
```

However, set iteration order should not be used when preserving the original order matters.

Order-preserving approach:

```python
numbers = [1, 2, 2, 3, 3, 4]

unique_numbers = list(dict.fromkeys(numbers))

print(unique_numbers)
# [1, 2, 3, 4]
```

This requires hashable elements.

---

## 3. Functions and Scope

### 3.1 What is a function?

A function is a reusable block of code designed to perform a particular task.

```python
def add(a, b):
    return a + b

result = add(10, 20)
print(result)
```

### 3.2 What is the difference between `return` and `print()`?

- `return` sends a value back to the caller.
- `print()` displays a value on the output stream.

```python
def add(a, b):
    return a + b

result = add(10, 20)
print(result)
```

A function without an explicit return statement returns `None`.

### 3.3 What are `*args` and `**kwargs`?

- `*args` collects extra positional arguments into a tuple.
- `**kwargs` collects extra keyword arguments into a dictionary.

```python
def calculate_sum(*args):
    return sum(args)

print(calculate_sum(10, 20, 30))
```

```python
def display_details(**kwargs):
    for key, value in kwargs.items():
        print(key, value)

display_details(name="Rahul", age=25)
```

### 3.4 What is variable scope?

Scope determines where a variable can be accessed.

Common scopes follow the LEGB rule:

1. Local
2. Enclosing
3. Global
4. Built-in

```python
x = 100

def example():
    x = 50
    print(x)

example()  # 50
print(x)   # 100
```

### 3.5 What are `global` and `nonlocal`?

- `global` refers to a variable in the module-level scope.
- `nonlocal` refers to a variable in an enclosing function scope.

```python
count = 0

def update():
    global count
    count += 1
```

```python
def outer():
    count = 0

    def inner():
        nonlocal count
        count += 1

    inner()
    return count
```

### 3.6 What is a lambda function?

A lambda is a small anonymous function containing a single expression.

```python
square = lambda x: x * x

print(square(5))  # 25
```

It is commonly used with `sorted()`, `map()`, and `filter()`.

### 3.7 What are `map()`, `filter()`, and `reduce()`?

**map():** Applies a function to each item.

```python
numbers = [1, 2, 3]

result = list(map(lambda x: x * 2, numbers))
```

**filter():** Keeps items that satisfy a condition.

```python
result = list(filter(lambda x: x % 2 == 0, numbers))
```

**reduce():** Combines elements into one accumulated result.

```python
from functools import reduce

result = reduce(lambda a, b: a + b, numbers)
```

---

## 4. Object-Oriented Programming

### 4.1 What is OOP?

Object-oriented programming organizes code around objects that combine data and behavior.

The four commonly discussed principles are:

1. Encapsulation
2. Inheritance
3. Polymorphism
4. Abstraction

### 4.2 What is a Class and an Object?

- **Class:** A blueprint defining attributes and methods.
- **Object:** An instance of a class.

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary

    def display(self):
        print(self.name, self.salary)


employee = Employee("Rahul", 50000)
employee.display()
```

### 4.3 What is `self`?

`self` refers to the current instance of a class.

It is used to access instance attributes and methods.

It is a naming convention, not a reserved keyword.

### 4.4 What is `__init__()`?

`__init__()` initializes an object's attributes after the object has been created.

```python
class Employee:
    def __init__(self, name):
        self.name = name
```

### 4.5 What is inheritance?

Inheritance allows a class to reuse or extend the behavior of another class.

```python
class Person:
    def display(self):
        print("Person details")


class Employee(Person):
    def work(self):
        print("Employee is working")


employee = Employee()
employee.display()
employee.work()
```

Python supports single, multiple, multilevel, hierarchical, and hybrid inheritance patterns.

### 4.6 What is polymorphism?

Polymorphism allows different objects to respond to the same method or interface in different ways.

```python
class Dog:
    def sound(self):
        print("Bark")


class Cat:
    def sound(self):
        print("Meow")


for animal in [Dog(), Cat()]:
    animal.sound()
```

### 4.7 What is encapsulation?

Encapsulation groups data and methods together and controls how internal state is accessed.

Python uses naming conventions rather than strict access modifiers:

- `name`: Public.
- `_name`: Intended for internal use.
- `__name`: Name-mangled in a class.

Double underscores do not provide absolute privacy.

### 4.8 What is abstraction?

Abstraction exposes essential behavior while hiding implementation details.

Python supports abstract classes through the `abc` module.

```python
from abc import ABC, abstractmethod

class DataProcessor(ABC):

    @abstractmethod
    def process(self, data):
        pass


class CSVProcessor(DataProcessor):

    def process(self, data):
        return data
```

### 4.9 Instance Method vs Class Method vs Static Method

| Method | First parameter | Purpose |
|---|---|---|
| Instance method | `self` | Works with instance state |
| Class method | `cls` | Works with class state |
| Static method | None automatically | Utility related to the class |

```python
class Employee:
    company = "ABC"

    def instance_method(self):
        return self.company

    @classmethod
    def class_method(cls):
        return cls.company

    @staticmethod
    def is_valid_salary(salary):
        return salary > 0
```

---

## 5. Exception Handling and File Handling

### 5.1 What is Exception Handling?

Exception handling allows a program to respond to runtime errors without terminating unexpectedly.

```python
try:
    number = int("abc")

except ValueError:
    print("Invalid number")

finally:
    print("Execution completed")
```

### 5.2 Difference between `try`, `except`, `else`, and `finally`

| Block | Purpose |
|---|---|
| `try` | Contains code that may raise an exception |
| `except` | Handles matching exceptions |
| `else` | Runs if no exception occurred in `try` |
| `finally` | Runs during normal or exceptional control flow |

### 5.3 How do you raise a custom exception?

```python
class InvalidAgeError(Exception):
    pass


def validate_age(age):
    if age < 18:
        raise InvalidAgeError("Age must be at least 18")
```

### 5.4 How do you read a file in Python?

```python
with open("data.txt", "r", encoding="utf-8") as file:
    content = file.read()
```

The `with` statement manages the file resource and closes it when the block exits.

### 5.5 What are common file modes?

| Mode | Meaning |
|---|---|
| `r` | Read |
| `w` | Write; truncates an existing file |
| `a` | Append |
| `x` | Create; fails if file exists |
| `rb` | Read binary |
| `wb` | Write binary |

### 5.6 How do you handle JSON data?

```python
import json

data = {
    "name": "Rahul",
    "age": 25
}

json_string = json.dumps(data)

parsed_data = json.loads(json_string)
```

- `json.dumps()` converts a Python object into a JSON string.
- `json.loads()` parses a JSON string into a Python object.
- `json.dump()` writes JSON to a file.
- `json.load()` reads JSON from a file.

---

## 6. Iterators, Generators, and Decorators

### 6.1 What is an Iterable?

An iterable is an object that can return an iterator.

Examples include lists, tuples, strings, sets, and dictionaries.

### 6.2 What is an Iterator?

An iterator produces values one at a time using `next()`.

```python
numbers = [10, 20, 30]

iterator = iter(numbers)

print(next(iterator))  # 10
print(next(iterator))  # 20
```

When no more elements are available, `next()` raises `StopIteration`.

### 6.3 What is a Generator?

A generator is an iterator commonly created using a function with `yield`.

```python
def generate_numbers():
    yield 1
    yield 2
    yield 3


for number in generate_numbers():
    print(number)
```

Generators produce values lazily instead of building the entire result in memory.

### 6.4 Generator vs List

| Feature | List | Generator |
|---|---|---|
| Evaluation | Eager | Lazy |
| Memory | Stores all elements | Produces elements as needed |
| Reuse | Can iterate repeatedly | Usually one-pass |
| Access | Supports indexing | No direct indexing |

**Data Engineering use case:** Generators can process large files or streams incrementally without loading all records into memory.

### 6.5 What is a Decorator?

A decorator modifies or extends a function's behavior without changing its original source code.

```python
from functools import wraps

def log_execution(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("Function started")
        result = func(*args, **kwargs)
        print("Function completed")
        return result

    return wrapper


@log_execution
def process_data():
    print("Processing data")


process_data()
```

Decorators are useful for logging, timing, authentication, and retry logic.

---

## 7. Python Memory and Performance

### 7.1 What is shallow copy vs deep copy?

A shallow copy creates a new outer object but keeps references to nested objects.

A deep copy recursively copies nested objects.

```python
import copy

original = [[1, 2], [3, 4]]

shallow = copy.copy(original)
deep = copy.deepcopy(original)

original[0][0] = 100

print(shallow[0][0])  # 100
print(deep[0][0])     # 1
```

### 7.2 What is the difference between `sort()` and `sorted()`?

- `list.sort()` modifies the original list and returns `None`.
- `sorted()` returns a new sorted list from an iterable.

```python
numbers = [3, 1, 2]

numbers.sort()

result = sorted(numbers, reverse=True)
```

### 7.3 What is the GIL?

The Global Interpreter Lock (GIL) in standard CPython builds allows only one thread at a time to execute Python bytecode within a process.

Implications:
- Threads can still help with I/O-bound work.
- CPU-bound Python code may not scale across threads in traditional GIL-enabled CPython.
- Multiprocessing can use multiple processes and CPU cores.
- Some native extensions release the GIL while performing work.

Free-threaded CPython builds are also available in newer Python versions, so the GIL statement should be qualified by implementation and build.

### 7.4 Multithreading vs Multiprocessing

| Feature | Multithreading | Multiprocessing |
|---|---|---|
| Execution | Multiple threads | Multiple processes |
| Memory | Shared process memory | Separate process memory |
| Best suited for | I/O-bound work | CPU-bound work |
| Communication | Shared objects, synchronization | IPC, queues, pipes, shared memory |
| Overhead | Usually lower | Usually higher |

### 7.5 What is Time Complexity?

Time complexity describes how an algorithm's running time grows as input size increases.

| Complexity | Example |
|---|---|
| O(1) | List indexing |
| O(log n) | Binary search |
| O(n) | Linear search |
| O(n log n) | Efficient comparison sorting |
| O(n²) | Nested loops over the same input |

These are typical algorithmic complexities; actual performance depends on the operation and implementation.

### 7.6 Common Python Collection Complexities

Typical average-case complexities:

| Operation | List | Set | Dictionary |
|---|---:|---:|---:|
| Access by index/key | O(1) for list index | Not applicable | O(1) average |
| Search/membership | O(n) | O(1) average | O(1) average for keys |
| Append/insert at end | O(1) amortized | N/A | N/A |
| Add item | N/A | O(1) average | O(1) average |
| Delete by value/key | O(n) for list value search | O(1) average | O(1) average |

Worst-case behavior can differ, especially for hash-based collections.

---

# 8. Python Coding Questions

These problems are useful for general Python interviews and Data Engineer coding rounds.

## 8.1 Reverse a String

```python
text = "python"

result = text[::-1]

print(result)
# nohtyp
```

**Complexity:** O(n) time and O(n) additional space.

Alternative:

```python
def reverse_string(text):
    result = ""

    for char in text:
        result = char + result

    return result
```

The alternative can take O(n²) time because strings are immutable and repeated concatenation creates new strings.

## 8.2 Check Whether a String Is a Palindrome

```python
def is_palindrome(text):
    return text == text[::-1]


print(is_palindrome("madam"))
# True
```

For case-insensitive alphanumeric comparison:

```python
def is_palindrome(text):
    cleaned = "".join(
        char.lower()
        for char in text
        if char.isalnum()
    )

    return cleaned == cleaned[::-1]
```

## 8.3 Find the Largest Number in a List

```python
numbers = [10, 50, 20, 80, 30]

largest = max(numbers)

print(largest)
# 80
```

Without using `max()`:

```python
def find_largest(numbers):
    if not numbers:
        raise ValueError("List cannot be empty")

    largest = numbers[0]

    for number in numbers:
        if number > largest:
            largest = number

    return largest
```

**Complexity:** O(n) time and O(1) extra space.

## 8.4 Find the Second-Largest Distinct Number

```python
def second_largest(numbers):
    unique_numbers = set(numbers)

    if len(unique_numbers) < 2:
        return None

    largest = second = float("-inf")

    for number in unique_numbers:
        if number > largest:
            second = largest
            largest = number
        elif number > second:
            second = number

    return second


print(second_largest([10, 20, 30, 30, 40]))
# 30
```

**Complexity:** O(n) average time and O(n) extra space.

## 8.5 Count Frequency of Each Element

```python
numbers = [1, 2, 2, 3, 3, 3]

frequency = {}

for number in numbers:
    frequency[number] = frequency.get(number, 0) + 1

print(frequency)
# {1: 1, 2: 2, 3: 3}
```

Using `Counter`:

```python
from collections import Counter

frequency = Counter(numbers)
```

## 8.6 Find Duplicate Elements

```python
numbers = [1, 2, 3, 2, 4, 3, 5]

seen = set()
duplicates = set()

for number in numbers:
    if number in seen:
        duplicates.add(number)
    else:
        seen.add(number)

print(duplicates)
# {2, 3}
```

**Complexity:** O(n) average time and O(n) space.

## 8.7 Find Missing Number from 1 to N

Assumption: The list contains distinct integers from 1 to N, with exactly one number missing.

```python
numbers = [1, 2, 4, 5]

n = 5

expected_sum = n * (n + 1) // 2
actual_sum = sum(numbers)

missing = expected_sum - actual_sum

print(missing)
# 3
```

**Complexity:** O(n) time and O(1) extra space.

## 8.8 Find Common Elements Between Two Lists

```python
a = [1, 2, 3, 4]
b = [3, 4, 5, 6]

result = list(set(a) & set(b))

print(result)
# [3, 4] (order is not guaranteed)
```

Order-preserving approach:

```python
b_set = set(b)

result = list(dict.fromkeys(x for x in a if x in b_set))
```

## 8.9 Check Whether Two Strings Are Anagrams

```python
from collections import Counter

def are_anagrams(a, b):
    return Counter(a) == Counter(b)


print(are_anagrams("listen", "silent"))
# True
```

If case, spaces, or punctuation should be ignored, normalize both strings first.

## 8.10 Find the First Non-Repeating Character

```python
from collections import Counter

def first_non_repeating(text):
    frequency = Counter(text)

    for char in text:
        if frequency[char] == 1:
            return char

    return None


print(first_non_repeating("swiss"))
# w
```

**Complexity:** O(n) time and O(k) space, where k is the number of distinct characters.

## 8.11 Flatten a Nested List (One Level)

```python
nested = [[1, 2], [3, 4], [5, 6]]

result = [
    item
    for sublist in nested
    for item in sublist
]

print(result)
# [1, 2, 3, 4, 5, 6]
```

This handles one nesting level, not arbitrarily deep nested lists.

## 8.12 Find the Most Frequent Element

```python
from collections import Counter

numbers = [1, 2, 2, 3, 3, 3, 4]

frequency = Counter(numbers)

most_common = frequency.most_common(1)

print(most_common)
# [(3, 3)]
```

If multiple values tie, `Counter.most_common()` preserves first-encounter order among equal counts in current Python behavior.

## 8.13 Move All Zeros to the End

```python
def move_zeros(numbers):
    non_zeros = [x for x in numbers if x != 0]
    zeros = [x for x in numbers if x == 0]

    return non_zeros + zeros


print(move_zeros([0, 1, 0, 3, 12]))
# [1, 3, 12, 0, 0]
```

**Complexity:** O(n) time and O(n) extra space.

## 8.14 Find Two Numbers That Add Up to a Target

```python
def two_sum(numbers, target):
    seen = {}

    for index, number in enumerate(numbers):
        required = target - number

        if required in seen:
            return [seen[required], index]

        seen[number] = index

    return None


print(two_sum([2, 7, 11, 15], 9))
# [0, 1]
```

**Complexity:** O(n) average time and O(n) space.

## 8.15 Merge Two Sorted Lists

```python
def merge_sorted(a, b):
    i = j = 0
    result = []

    while i < len(a) and j < len(b):
        if a[i] <= b[j]:
            result.append(a[i])
            i += 1
        else:
            result.append(b[j])
            j += 1

    result.extend(a[i:])
    result.extend(b[j:])

    return result


print(merge_sorted([1, 3, 5], [2, 4, 6]))
# [1, 2, 3, 4, 5, 6]
```

**Complexity:** O(n + m) time and O(n + m) output space.

## 8.16 Find the Maximum Subarray Sum

Kadane's algorithm finds the maximum sum of a non-empty contiguous subarray.

```python
def max_subarray_sum(numbers):
    if not numbers:
        raise ValueError("Input cannot be empty")

    current_sum = numbers[0]
    maximum_sum = numbers[0]

    for number in numbers[1:]:
        current_sum = max(number, current_sum + number)
        maximum_sum = max(maximum_sum, current_sum)

    return maximum_sum


print(max_subarray_sum([-2, 1, -3, 4, -1, 2, 1, -5, 4]))
# 6
```

The maximum subarray is `[4, -1, 2, 1]`.

**Complexity:** O(n) time and O(1) extra space.

## 8.17 Binary Search

Binary search works on a sorted collection.

```python
def binary_search(numbers, target):
    left = 0
    right = len(numbers) - 1

    while left <= right:
        mid = left + (right - left) // 2

        if numbers[mid] == target:
            return mid

        elif numbers[mid] < target:
            left = mid + 1

        else:
            right = mid - 1

    return -1


print(binary_search([10, 20, 30, 40, 50], 30))
# 2
```

**Complexity:** O(log n) time and O(1) extra space.

## 8.18 Check Balanced Parentheses

```python
def is_balanced(expression):
    stack = []

    pairs = {
        ")": "(",
        "}": "{",
        "]": "["
    }

    for char in expression:
        if char in "([{":
            stack.append(char)

        elif char in ")]}":
            if not stack or stack.pop() != pairs[char]:
                return False

    return len(stack) == 0


print(is_balanced("{[()]}"))
# True
```

**Complexity:** O(n) time and O(n) space.

---

# 9. Python Questions for Data Engineers

These questions connect core Python concepts to real data pipeline work.

### 9.1 How do you process a large CSV file without loading it completely into memory?

Use chunk-based reading with Pandas or stream the file line by line.

```python
import pandas as pd

for chunk in pd.read_csv("large_file.csv", chunksize=100_000):
    # Validate and process this chunk
    print(len(chunk))
```

The chunk size should be chosen based on memory, row width, and processing requirements.

For simpler line-based processing:

```python
import csv

with open("large_file.csv", "r", newline="", encoding="utf-8") as file:
    reader = csv.DictReader(file)

    for row in reader:
        process_row(row)
```

### 9.2 How do you handle missing values in Python?

```python
import pandas as pd

df = pd.DataFrame({
    "name": ["A", "B", "C"],
    "salary": [50000, None, 70000]
})

df["salary"] = df["salary"].fillna(0)
```

The correct strategy depends on business rules. Missing values should not automatically be replaced with zero.

### 9.3 How do you validate data before loading it?

Typical validation checks include:

- Required columns exist.
- Data types are correct.
- Required fields are not NULL.
- Business keys are unique where expected.
- Values fall within valid ranges.
- Dates follow expected rules.
- Records satisfy business constraints.

```python
def validate_record(record):
    required_fields = ["customer_id", "amount"]

    for field in required_fields:
        if record.get(field) is None:
            return False

    if record["amount"] < 0:
        return False

    return True
```

### 9.4 How do you call an API using Python?

```python
import requests

response = requests.get(
    "https://api.example.com/customers",
    timeout=30
)

response.raise_for_status()

data = response.json()
```

Production API pipelines should also consider authentication, pagination, rate limits, retries, timeouts, and response validation.

### 9.5 How do you handle API pagination?

A common approach is to repeatedly request pages until the API indicates that no more data is available.

```python
import requests

url = "https://api.example.com/customers"
page = 1
records = []

while True:
    response = requests.get(
        url,
        params={"page": page},
        timeout=30
    )

    response.raise_for_status()

    data = response.json()
    page_records = data["results"]

    if not page_records:
        break

    records.extend(page_records)
    page += 1
```

The exact pagination logic depends on the API. Cursor-based pagination is often preferable when supported.

### 9.6 How do you implement retry logic?

```python
import time
import requests

def fetch_data(url, retries=3):
    for attempt in range(retries):
        try:
            response = requests.get(url, timeout=30)
            response.raise_for_status()
            return response.json()

        except requests.RequestException:
            if attempt == retries - 1:
                raise

            time.sleep(2 ** attempt)
```

Production systems should retry only appropriate failures, use bounded backoff, consider jitter, and respect `Retry-After` when supplied.

### 9.7 What is idempotency in a data pipeline?

An operation is idempotent when repeating it has the same final effect as performing it once, under the defined conditions.

Example:
- Loading the same source file repeatedly should not create duplicate target records.
- A `MERGE` based on a stable business key can support idempotent upserts.
- A pipeline should safely handle retries after partial failures.

Idempotency depends on the complete write process, not simply on using Python.

### 9.8 How do you log pipeline execution?

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(message)s"
)

logging.info("Pipeline started")
logging.warning("Some records were rejected")
logging.error("Pipeline failed")
```

Production logging should include useful context such as pipeline name, run ID, source, target, record counts, and error details. Avoid logging credentials or sensitive data.

### 9.9 What is the difference between Pandas and PySpark?

| Feature | Pandas | PySpark |
|---|---|---|
| Processing | Primarily in-memory, single-machine | Distributed across a cluster |
| Typical use | Small and medium datasets | Large distributed datasets |
| Execution | Eager for typical DataFrame operations | Lazy for DataFrame transformations |
| Scaling | Limited by machine resources | Can scale across cluster resources |
| API | Python DataFrame API | Spark DataFrame API |

Pandas is useful for local analysis and smaller workloads. PySpark is useful when distributed processing is required.

### 9.10 Why is Python used with PySpark?

Python provides a familiar interface for defining Spark transformations and actions.

PySpark translates DataFrame operations into Spark execution plans. The actual distributed processing is performed by Spark's execution engine, not by ordinary Python loops over every distributed row.

Avoid using Python UDFs when built-in Spark SQL functions can perform the same operation efficiently.

---

# 10. Common Python Interview Traps

### Trap 1: Mutable Default Arguments

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

### Trap 2: List Assignment vs Copy

```python
a = [1, 2, 3]
b = a

b.append(4)

print(a)
# [1, 2, 3, 4]
```

Both variables refer to the same list.

To make a shallow copy:

```python
b = a.copy()
```

### Trap 3: Dictionary Key Overwriting

```python
data = {
    "name": "A",
    "name": "B"
}

print(data)
# {'name': 'B'}
```

Dictionary keys must be unique; a later value replaces an earlier value for the same key.

### Trap 4: Modifying a List While Iterating

Modifying a list during iteration can cause elements to be skipped or produce confusing behavior.

Safer approach:

```python
numbers = [1, 2, 3, 4, 5]

result = [x for x in numbers if x % 2 == 0]
```

### Trap 5: Confusing `None` and Empty Collections

```python
a = None
b = []
c = {}

print(a is None)  # True
print(bool(b))    # False
print(bool(c))    # False
```

`None`, an empty list, and an empty dictionary are different values.

### Trap 6: Using `and` / `or` with Pandas or PySpark Columns

For vectorized or column expressions, use the relevant library's operators and parentheses rather than Python's scalar `and` / `or`.

PySpark example:

```python
from pyspark.sql.functions import col

filtered = df.filter(
    (col("age") > 18) & (col("salary") > 30000)
)
```

---

# 11. Interview Preparation Checklist

## Python Fundamentals

- [ ] Mutable vs immutable
- [ ] List vs tuple
- [ ] Set vs dictionary
- [ ] `==` vs `is`
- [ ] List comprehension
- [ ] Slicing
- [ ] `append()` vs `extend()`
- [ ] `break`, `continue`, `pass`

## Functions and OOP

- [ ] `*args` and `**kwargs`
- [ ] Lambda functions
- [ ] `map()`, `filter()`, `reduce()`
- [ ] LEGB scope
- [ ] Class and object
- [ ] Inheritance and polymorphism
- [ ] Encapsulation and abstraction
- [ ] Instance, class, and static methods

## Advanced Python

- [ ] Exception handling
- [ ] File handling
- [ ] Iterators and generators
- [ ] Decorators
- [ ] Shallow vs deep copy
- [ ] GIL
- [ ] Multithreading vs multiprocessing
- [ ] Time and space complexity

## Coding Problems

- [ ] Reverse a string
- [ ] Palindrome
- [ ] Largest and second-largest element
- [ ] Frequency counting
- [ ] Duplicate detection
- [ ] Missing number
- [ ] Anagram
- [ ] First non-repeating character
- [ ] Two Sum
- [ ] Merge sorted lists
- [ ] Maximum subarray sum
- [ ] Binary search
- [ ] Balanced parentheses

## Data Engineering Python

- [ ] Process large files in chunks
- [ ] Read and write CSV/JSON
- [ ] API integration and pagination
- [ ] Retry and exception handling
- [ ] Logging
- [ ] Data validation
- [ ] Idempotent pipeline design
- [ ] Pandas vs PySpark

---

# 12. Quick Revision

| Topic | Remember |
|---|---|
| List | Mutable, ordered |
| Tuple | Immutable, ordered |
| Set | Unique hashable elements |
| Dictionary | Key-value mapping |
| `==` | Value equality |
| `is` | Object identity |
| `*args` | Extra positional arguments |
| `**kwargs` | Extra keyword arguments |
| `yield` | Produces generator values |
| Decorator | Extends function behavior |
| `self` | Current instance |
| `cls` | Current class |
| `finally` | Cleanup/control-flow block |
| Shallow copy | New outer object, nested references shared |
| Deep copy | Recursively copies nested objects |
| GIL | Limits simultaneous Python bytecode execution in standard GIL-enabled CPython |
| Generator | Lazy value production |
| Idempotency | Repeated execution has the same defined final effect |
| Logging | Records pipeline execution details |

## Final Interview Advice

For a Data Engineer interview, prioritize these areas:

1. Python collections and their time complexities.
2. Functions, comprehensions, and dictionary operations.
3. Iterators and generators for memory-efficient processing.
4. Exception handling, file handling, and API integration.
5. OOP fundamentals and clean code.
6. Coding problems involving strings, arrays, hashing, and searching.
7. Python's role in ETL pipelines and its relationship with PySpark.

Do not focus only on memorizing definitions. Be prepared to write code, explain its time and space complexity, discuss edge cases, and describe how you would use the concept in a real data pipeline.