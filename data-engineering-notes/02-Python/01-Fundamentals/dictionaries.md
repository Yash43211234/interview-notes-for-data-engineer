# Dictionaries in Python

## 1. Overview

A **dictionary** is a built-in Python data structure that stores data as key-value pairs.

Each key is used to access its corresponding value.

```python
employee = {
    "name": "Yash",
    "age": 26,
    "role": "Data Engineer"
}

print(employee["name"])
```

Output:

```text
Yash
```

Dictionaries are useful when data needs to be accessed using a meaningful key rather than a numeric index.

**Common data engineering uses:**
- Representing records and JSON data.
- Counting the frequency of values.
- Mapping customer IDs to customer details.
- Grouping and transforming data.
- Tracking intermediate results during data processing.

---

## 2. Creating Dictionaries

### 2.1 Using curly braces

```python
person = {
    "name": "Rahul",
    "age": 25,
    "city": "Bengaluru"
}
```

### 2.2 Creating an empty dictionary

```python
data = {}
```

Alternatively:

```python
data = dict()
```

### 2.3 Using the `dict()` constructor

```python
person = dict(name="Rahul", age=25)
```

Output:

```python
{"name": "Rahul", "age": 25}
```

This syntax requires keyword-compatible keys, such as strings that are valid parameter names.

### 2.4 Creating a dictionary from pairs

```python
pairs = [("a", 1), ("b", 2), ("c", 3)]

data = dict(pairs)

print(data)
```

Output:

```python
{"a": 1, "b": 2, "c": 3}
```

### 2.5 Dictionary comprehension

```python
squares = {number: number ** 2 for number in range(1, 5)}

print(squares)
```

Output:

```python
{1: 1, 2: 4, 3: 9, 4: 16}
```

---

## 3. Important Properties

Python dictionaries have the following properties:

- **Mutable:** You can add, update, and delete entries.
- **Ordered:** They preserve insertion order in Python 3.7 and later.
- **Unique keys:** A key can appear only once in the final dictionary.
- **Flexible values:** Values can be integers, strings, lists, other dictionaries, or other objects.
- **Hashable keys:** Keys must be hashable, such as strings, integers, and tuples containing only hashable elements.

Example:

```python
data = {
    "name": "Aman",
    "skills": ["Python", "SQL"],
    "experience": {"years": 2}
}
```

Dictionaries can contain other dictionaries, which is useful for representing nested JSON-like structures.

### Duplicate keys

```python
data = {
    "name": "Aman",
    "age": 24,
    "name": "Riya"
}

print(data)
```

Output:

```python
{"name": "Riya", "age": 24}
```

The last value assigned to a duplicate key wins. The dictionary does not retain two separate entries for that key.

### Valid and invalid keys

```python
valid = {
    "name": "Aman",
    10: "ten",
    (1, 2): "pair"
}
```

Lists and dictionaries cannot be keys because they are mutable and unhashable.

```python
# Invalid:
# data = {[1, 2]: "value"}
# data = {{"a": 1}: "value"}
```

A tuple is valid as a key only if all its elements are hashable.

---

## 4. Accessing Dictionary Values

### 4.1 Using square brackets

```python
employee = {"name": "Aman", "salary": 50000}

print(employee["name"])
```

Output:

```text
Aman
```

If the key does not exist, square-bracket access raises a `KeyError`.

```python
# employee["department"]  # KeyError
```

### 4.2 Using `get()`

```python
employee = {"name": "Aman", "salary": 50000}

print(employee.get("salary"))
print(employee.get("department"))
```

Output:

```text
50000
None
```

You can supply a default value:

```python
print(employee.get("department", "Not assigned"))
```

Output:

```text
Not assigned
```

**Interview distinction:**

| Method | Key exists | Key missing |
|---|---|---|
| `data[key]` | Returns value | Raises `KeyError` |
| `data.get(key)` | Returns value | Returns `None` by default |
| `data.get(key, default)` | Returns value | Returns specified default |

Use square brackets when a missing key should be treated as an error. Use `get()` when a missing key is an expected possibility.

**Important:** `get()` returning `None` does not necessarily prove the key is absent. The key might exist and explicitly contain `None`. Use `key in data` when you need to distinguish those cases.

---

## 5. Adding and Updating Values

### 5.1 Add a new key

```python
employee = {"name": "Aman"}

employee["salary"] = 50000

print(employee)
```

Output:

```python
{"name": "Aman", "salary": 50000}
```

### 5.2 Update an existing key

```python
employee["salary"] = 60000
```

If the key already exists, its value is replaced.

### 5.3 Update multiple entries with `update()`

```python
employee = {"name": "Aman", "salary": 50000}

employee.update({
    "salary": 60000,
    "department": "Data"
})

print(employee)
```

Output:

```python
{"name": "Aman", "salary": 60000, "department": "Data"}
```

`update()` adds new keys and overwrites matching keys.

### 5.4 Merge dictionaries

In Python 3.9 and later, the `|` operator creates a merged dictionary.

```python
first = {"a": 1, "b": 2}
second = {"b": 5, "c": 3}

merged = first | second

print(merged)
```

Output:

```python
{"a": 1, "b": 5, "c": 3}
```

When keys overlap, values from the right-hand dictionary take precedence.

The original dictionaries remain unchanged.

---

## 6. Removing Entries

### 6.1 `pop()`

Removes a key and returns its value.

```python
data = {"a": 1, "b": 2}

value = data.pop("a")

print(value)
print(data)
```

Output:

```text
1
{'b': 2}
```

You can provide a default to avoid a `KeyError`:

```python
data.pop("missing", None)
```

### 6.2 `del`

Removes a specified key.

```python
data = {"a": 1, "b": 2}

del data["a"]
```

A missing key raises `KeyError`.

### 6.3 `popitem()`

Removes and returns the last inserted key-value pair in modern Python.

```python
data = {"a": 1, "b": 2}

item = data.popitem()

print(item)
print(data)
```

Output:

```text
('b', 2)
{'a': 1}
```

### 6.4 `clear()`

Removes all entries.

```python
data = {"a": 1, "b": 2}

data.clear()

print(data)
```

Output:

```python
{}
```

### Comparison

| Method | Behavior |
|---|---|
| `pop(key)` | Removes a key and returns its value |
| `del data[key]` | Removes a key |
| `popitem()` | Removes and returns the last inserted pair |
| `clear()` | Removes all entries |

---

## 7. Checking Keys and Values

Use `in` to check whether a key exists.

```python
employee = {"name": "Aman", "salary": 50000}

print("name" in employee)
print("department" in employee)
```

Output:

```text
True
False
```

By default, membership testing on a dictionary checks **keys**, not values.

```python
print(50000 in employee)           # False
print(50000 in employee.values())  # True
```

To check for a particular key-value pair:

```python
if employee.get("salary") == 50000:
    print("Salary matched")
```

If it is important to distinguish a missing key from a key mapped to `None`, check key membership separately.

---

## 8. Iterating Through Dictionaries

### 8.1 Iterate over keys

```python
employee = {
    "name": "Aman",
    "salary": 50000,
    "department": "Data"
}

for key in employee:
    print(key)
```

Equivalent:

```python
for key in employee.keys():
    print(key)
```

### 8.2 Iterate over values

```python
for value in employee.values():
    print(value)
```

### 8.3 Iterate over keys and values

```python
for key, value in employee.items():
    print(key, value)
```

`items()` yields key-value pairs, allowing both to be accessed in a single loop.

### Performance note

Iterating over all entries takes \(O(n)\) time for a dictionary with \(n\) entries.

Avoid repeatedly searching through all values when a direct key lookup can solve the problem more efficiently.

---

## 9. Dictionary Comprehensions

A dictionary comprehension builds a dictionary from an iterable.

General syntax:

```python
{key_expression: value_expression for item in iterable}
```

### 9.1 Create a mapping

```python
numbers = [1, 2, 3, 4]

squares = {number: number ** 2 for number in numbers}

print(squares)
```

Output:

```python
{1: 1, 2: 4, 3: 9, 4: 16}
```

### 9.2 Apply a condition

```python
numbers = [1, 2, 3, 4, 5, 6]

even_squares = {
    number: number ** 2
    for number in numbers
    if number % 2 == 0
}

print(even_squares)
```

Output:

```python
{2: 4, 4: 16, 6: 36}
```

### 9.3 Transform existing values

```python
prices = {"apple": 100, "banana": 40}

discounted = {
    item: price * 0.9
    for item, price in prices.items()
}
```

This creates a new dictionary with a 10% discount applied to each price.

### 9.4 Swap keys and values

```python
data = {"a": 1, "b": 2, "c": 3}

swapped = {value: key for key, value in data.items()}

print(swapped)
```

Output:

```python
{1: 'a', 2: 'b', 3: 'c'}
```

This works as a complete reversal only when the original values are unique and hashable. Duplicate values cause collisions, with the last value processed winning.

---

## 10. Counting Frequencies with Dictionaries

Frequency counting is one of the most common Python interview patterns.

### 10.1 Using a normal dictionary

```python
numbers = [1, 2, 2, 3, 3, 3]

frequency = {}

for number in numbers:
    if number in frequency:
        frequency[number] += 1
    else:
        frequency[number] = 1

print(frequency)
```

Output:

```python
{1: 1, 2: 2, 3: 3}
```

The dictionary stores each distinct number as a key and its frequency as the value.

Expected time complexity: \(O(n)\), assuming average-case constant-time dictionary operations.

### 10.2 Using `get()`

The same logic can be written more concisely:

```python
numbers = [1, 2, 2, 3, 3, 3]

frequency = {}

for number in numbers:
    frequency[number] = frequency.get(number, 0) + 1

print(frequency)
```

When a number is missing, `get(number, 0)` supplies zero as the starting count.

### 10.3 Using `collections.Counter`

```python
from collections import Counter

numbers = [1, 2, 2, 3, 3, 3]

frequency = Counter(numbers)

print(frequency)
```

Output:

```text
Counter({3: 3, 2: 2, 1: 1})
```

Use a normal dictionary to understand the underlying logic. Use `Counter` when a purpose-built frequency-counting utility is appropriate.

---

## 11. Grouping Data with Dictionaries

Dictionaries can group multiple values under the same key.

Example: Group employees by department.

```python
employees = [
    {"name": "Aman", "department": "Data"},
    {"name": "Riya", "department": "HR"},
    {"name": "Rahul", "department": "Data"}
]

grouped = {}

for employee in employees:
    department = employee["department"]

    if department not in grouped:
        grouped[department] = []

    grouped[department].append(employee["name"])

print(grouped)
```

Output:

```python
{
    "Data": ["Aman", "Rahul"],
    "HR": ["Riya"]
}
```

This pattern is useful for:
- Grouping records by category.
- Aggregating values by customer.
- Organizing events by date.
- Preparing data before writing to a database.

You can also use `collections.defaultdict` to simplify initialization:

```python
from collections import defaultdict

grouped = defaultdict(list)

for employee in employees:
    grouped[employee["department"]].append(employee["name"])
```

Accessing a missing key in a `defaultdict(list)` creates an empty list for that key.

---

## 12. Nested Dictionaries

A dictionary can contain another dictionary as a value.

```python
employees = {
    101: {
        "name": "Aman",
        "salary": 50000
    },
    102: {
        "name": "Riya",
        "salary": 60000
    }
}
```

Access nested values:

```python
print(employees[101]["name"])
print(employees[102]["salary"])
```

Output:

```text
Aman
60000
```

Update a nested value:

```python
employees[101]["salary"] = 55000
```

When accessing deeply nested data from an external source, check whether the required keys exist before accessing them directly.

For example:

```python
employee = employees.get(103)

if employee is not None:
    print(employee.get("name"))
```

This avoids a `KeyError` if employee `103` does not exist.

---

## 13. Useful Dictionary Methods

| Method | Purpose |
|---|---|
| `get(key, default)` | Safely retrieve a value |
| `keys()` | Get a view of keys |
| `values()` | Get a view of values |
| `items()` | Get a view of key-value pairs |
| `update(other)` | Add or overwrite entries |
| `pop(key)` | Remove a key and return its value |
| `popitem()` | Remove and return the last inserted pair |
| `setdefault(key, default)` | Return a value or insert the default if the key is absent |
| `copy()` | Create a shallow copy |
| `clear()` | Remove all entries |

### Understanding `setdefault()`

```python
data = {"a": 1}

value = data.setdefault("b", 0)

print(value)
print(data)
```

Output:

```text
0
{'a': 1, 'b': 0}
```

If the key exists, `setdefault()` returns its current value without replacing it. If the key is missing, it inserts the supplied default.

For simple frequency counting, `get()` is often clearer. For initializing groups, `defaultdict` may be more convenient.

---

## 14. Shallow Copy vs Reference

This is an important interview topic.

### Assignment does not create a new dictionary

```python
original = {"a": 1}
another = original

another["a"] = 100

print(original)
```

Output:

```python
{"a": 100}
```

Both variables refer to the same dictionary.

### Shallow copy

```python
original = {"a": 1, "b": 2}

copied = original.copy()
copied["a"] = 100

print(original)
print(copied)
```

Output:

```python
{'a': 1, 'b': 2}
{'a': 100, 'b': 2}
```

The outer dictionaries are different objects.

However, `.copy()` does not recursively copy nested mutable objects.

```python
original = {"numbers": [1, 2, 3]}
copied = original.copy()

copied["numbers"].append(4)

print(original)
```

Output:

```python
{'numbers': [1, 2, 3, 4]}
```

Both dictionaries still refer to the same nested list.

When a fully independent copy of nested data is required, consider `copy.deepcopy()`.

```python
import copy

copied = copy.deepcopy(original)
```

Deep copying can be more expensive and is not always necessary.

---

## 15. Time Complexity of Dictionaries

Let \(n\) be the number of entries in the dictionary.

| Operation | Average-case time |
|---|---:|
| Access by key | \(O(1)\) |
| Insert or update a key | \(O(1)\) |
| Delete by key | \(O(1)\) |
| Check whether a key exists | \(O(1)\) |
| Iterate over all entries | \(O(n)\) |
| Copy a dictionary | \(O(n)\) |
| Search for a value without knowing its key | \(O(n)\) |

These are average-case complexities. Dictionary operations can take \(O(n)\) in certain worst-case collision scenarios, although Python's implementation is designed to make average-case lookups efficient.

### Why dictionaries are fast

Dictionaries use a hash table. Python computes a hash for a key and uses it to locate the corresponding entry efficiently.

Keys must be hashable, and keys that compare equal must have equal hashes.

**Interview takeaway:** Use a dictionary when frequent lookup by a key is needed. If you need to search through every value, the operation generally takes linear time.

---

## 16. Common Mistakes

### Mistake 1: Accessing a missing key

```python
data = {"name": "Aman"}

# Raises KeyError:
# print(data["salary"])

# Safer when absence is expected:
print(data.get("salary", 0))
```

### Mistake 2: Assuming `in` checks values

```python
data = {"a": 10}

print(10 in data)         # False
print(10 in data.values())  # True
```

### Mistake 3: Using mutable keys

Lists and dictionaries cannot be dictionary keys because they are unhashable.

### Mistake 4: Assuming duplicate keys are retained

Only one entry remains for each key. A later value replaces the earlier value.

### Mistake 5: Modifying dictionary size during iteration

Adding or removing entries while iterating over a dictionary can raise a `RuntimeError`.

If you need to remove selected keys, iterate over a snapshot:

```python
data = {"a": 1, "b": 2, "c": 3}

for key in list(data.keys()):
    if data[key] % 2 == 0:
        del data[key]

print(data)
```

Output:

```python
{'a': 1, 'c': 3}
```

### Mistake 6: Confusing a shallow copy with a deep copy

A shallow copy creates a new outer dictionary but does not recursively copy nested objects.

### Mistake 7: Assuming dictionary values are unique

Different keys can have identical values.

```python
data = {"a": 1, "b": 1}
```

This is valid.

---

## 17. Interview Questions

### Conceptual questions

1. What is a dictionary in Python?
2. Why are dictionary lookups usually \(O(1)\)?
3. Why must dictionary keys be hashable?
4. Can a list be used as a dictionary key? Why?
5. What happens when duplicate keys are used?
6. What is the difference between `data[key]` and `data.get(key)`?
7. What is the difference between `keys()`, `values()`, and `items()`?
8. What is the difference between `update()` and `setdefault()`?
9. What is the difference between a shallow copy and a deep copy?
10. What is the difference between a dictionary and a list?
11. What happens when two keys have the same hash?
12. What is the difference between a normal dictionary and `defaultdict`?
13. Does `in` check dictionary keys or values by default?
14. What is the time complexity of iterating through a dictionary?
15. What happens when a dictionary is modified during iteration?

### Output-based questions

**Question 1**

```python
data = {"a": 1, "b": 2}
data["a"] = 5
data["c"] = 3

print(data)
```

Answer:

```python
{'a': 5, 'b': 2, 'c': 3}
```

**Question 2**

```python
data = {"x": None}

print(data.get("x", 10))
print(data.get("y", 10))
```

Answer:

```text
None
10
```

The default is used only when the key is absent, not when its value is `None`.

**Question 3**

```python
data = {"a": 1, "b": 2, "a": 3}

print(len(data))
print(data["a"])
```

Answer:

```text
2
3
```

Duplicate keys do not create additional entries.

**Question 4**

```python
data = {"a": 1, "b": 2}

for key, value in data.items():
    print(key, value)
```

Answer:

```text
a 1
b 2
```

The pairs are visited in insertion order.

---

## 18. Practice Problems

Try to solve each problem without looking at the solution first.

### Easy

1. Create a dictionary containing your name, age, and skills.
2. Add a new key and update an existing key.
3. Count the frequency of each character in a string.
4. Count the frequency of each number in a list.
5. Check whether a particular key exists.
6. Find the key associated with a given value.
7. Merge two dictionaries.
8. Find the number of unique keys in a dictionary.

### Medium

1. Find the most frequent element in a list.
2. Group a list of names by their first character.
3. Invert a dictionary while handling duplicate values.
4. Merge two dictionaries by summing values for matching keys.
5. Find the first non-repeating character in a string.
6. Group employee records by department.
7. Find common keys between two dictionaries.
8. Sort a dictionary by its values.
9. Build a dictionary from two lists: one of keys and one of values.
10. Remove dictionary entries whose values are below a threshold.

### Data engineering practice

1. Given a list of transaction records, calculate total transaction amounts by customer ID.
2. Group records by date and count the number of records per day.
3. Given customer records, create a mapping from customer ID to customer details.
4. Compare two dictionaries representing old and new records and identify changed values.
5. Given a list of JSON-like records, count how many records belong to each category.
6. Handle missing fields in a record without raising `KeyError`.
7. Merge two configuration dictionaries while allowing the second dictionary to override the first.
8. Identify duplicate IDs in a list of records using a dictionary or set.

For each problem, explain the approach, time complexity, and additional space complexity.

---

## 19. Quick Revision

- A dictionary stores key-value pairs.
- Keys must be hashable; values can be of different types.
- Dictionary keys are unique.
- Dictionaries preserve insertion order in Python 3.7 and later.
- `data[key]` raises `KeyError` if the key is missing.
- `data.get(key, default)` supplies a default for a missing key.
- `key in data` checks keys, not values.
- `data.items()` provides key-value pairs.
- `update()` adds or overwrites entries.
- `pop()` removes a key and returns its value.
- Dictionary comprehensions build dictionaries concisely.
- Frequency counting and grouping are common dictionary patterns.
- Average-case key lookup, insertion, and deletion are \(O(1)\).
- Iterating through a dictionary takes \(O(n)\).
- `.copy()` creates a shallow copy; `deepcopy()` recursively copies nested objects.

**Recommended practice order:** First master dictionary creation, access, and updates. Then practice frequency counting and grouping. Finally, solve the medium DSA problems and record the time and space complexity for each solution.