# Python Comprehensions — Interview Notes

**File:** `comprehensions.md`

## 1. What Is a Comprehension?

A comprehension is a concise way to create a collection from an iterable, such as a list, dictionary, set, or range.

Instead of writing a loop and manually adding each result to a collection, you can express the transformation in a single statement.

Python supports four common patterns:

- List comprehension
- Set comprehension
- Dictionary comprehension
- Generator expression

## 2. List Comprehension

A list comprehension creates a new list.

**Syntax:**

```python
[expression for item in iterable]
```

### Example 1: Create a list of squares

Using a normal loop:

```python
squares = []

for n in range(1, 6):
    squares.append(n ** 2)

print(squares)
# [1, 4, 9, 16, 25]
```

Using a list comprehension:

```python
squares = [n ** 2 for n in range(1, 6)]

print(squares)
# [1, 4, 9, 16, 25]
```

**How it works:**

1. `range(1, 6)` produces numbers from 1 to 5.
2. `n` receives each number.
3. `n ** 2` calculates its square.
4. The results are collected into a new list.

### Example 2: Convert strings to uppercase

```python
names = ["amit", "priya", "rahul"]

upper_names = [name.upper() for name in names]

print(upper_names)
# ['AMIT', 'PRIYA', 'RAHUL']
```

### Example 3: Extract string lengths

```python
words = ["python", "sql", "spark"]

lengths = [len(word) for word in words]

print(lengths)
# [6, 3, 5]
```

## 3. List Comprehension with Conditions

You can filter elements using an `if` condition.

**Syntax:**

```python
[expression for item in iterable if condition]
```

### Example 1: Select even numbers

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8]

evens = [n for n in numbers if n % 2 == 0]

print(evens)
# [2, 4, 6, 8]
```

The condition decides whether an element is included in the output.

### Example 2: Filter strings by length

```python
words = ["data", "engineering", "sql", "python"]

long_words = [word for word in words if len(word) > 4]

print(long_words)
# ['engineering', 'python']
```

### Example 3: Remove negative numbers

```python
numbers = [-5, 3, -2, 8, 0, 4]

positive_numbers = [n for n in numbers if n > 0]

print(positive_numbers)
# [3, 8, 4]
```

Notice that zero is not included because the condition is `n > 0`.

## 4. List Comprehension with `if-else`

Use `if-else` when you want to transform every element differently rather than filter elements out.

**Syntax:**

```python
[expression_if_true if condition else expression_if_false
 for item in iterable]
```

### Example 1: Label numbers

```python
numbers = [1, 2, 3, 4, 5, 6]

labels = [
    "Even" if n % 2 == 0 else "Odd"
    for n in numbers
]

print(labels)
# ['Odd', 'Even', 'Odd', 'Even', 'Odd', 'Even']
```

### Example 2: Replace negative values with zero

```python
numbers = [-3, 5, -1, 8, 0]

cleaned = [n if n >= 0 else 0 for n in numbers]

print(cleaned)
# [0, 5, 0, 8, 0]
```

### Important difference

Filtering with `if`:

```python
[n for n in numbers if n > 0]
```

This excludes nonpositive numbers.

Transforming with `if-else`:

```python
[n if n > 0 else 0 for n in numbers]
```

This keeps the same number of elements but replaces nonpositive values with zero.

## 5. Set Comprehension

A set comprehension creates a set containing unique values.

**Syntax:**

```python
{expression for item in iterable}
```

### Example 1: Get unique squares

```python
numbers = [1, 2, 2, 3, 3, 4]

squares = {n ** 2 for n in numbers}

print(squares)
# {1, 4, 9, 16}
```

Duplicate results are automatically removed.

### Example 2: Extract unique initials

```python
names = ["Amit", "Ankit", "Priya", "Rahul"]

initials = {name[0] for name in names}

print(initials)
# A set containing A, P, and R
```

**Important:** Sets are unordered collections. Do not depend on their display order.

## 6. Dictionary Comprehension

A dictionary comprehension creates key-value pairs.

**Syntax:**

```python
{key_expression: value_expression for item in iterable}
```

### Example 1: Create a number-to-square mapping

```python
numbers = [1, 2, 3, 4, 5]

squares = {n: n ** 2 for n in numbers}

print(squares)
# {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

### Example 2: Convert a list into a frequency dictionary

```python
names = ["Amit", "Priya", "Amit", "Rahul", "Priya", "Amit"]

frequency = {
    name: names.count(name)
    for name in set(names)
}

print(frequency)
# Contains Amit: 3, Priya: 2, Rahul: 1
```

This is useful for learning, but `names.count()` repeatedly scans the list, making the approach inefficient for large inputs. A loop or `collections.Counter` is preferable for frequency counting.

### Example 3: Filter dictionary entries

```python
scores = {
    "Amit": 85,
    "Priya": 92,
    "Rahul": 65,
    "Sneha": 78
}

passed = {
    name: score
    for name, score in scores.items()
    if score >= 80
}

print(passed)
# {'Amit': 85, 'Priya': 92}
```

## 7. Generator Expressions

A generator expression produces values lazily instead of building the entire collection immediately.

**Syntax:**

```python
(expression for item in iterable)
```

### Example 1: Generate squares lazily

```python
squares = (n ** 2 for n in range(1, 6))

print(next(squares))  # 1
print(next(squares))  # 4
print(next(squares))  # 9
```

Each call to `next()` retrieves the next value.

### Example 2: Sum squares

```python
total = sum(n ** 2 for n in range(1, 1001))

print(total)
```

A generator expression avoids creating a separate list containing all 1,000 squares.

### List comprehension vs. generator expression

| Feature | List comprehension | Generator expression |
|---|---|---|
| Syntax | `[x * 2 for x in data]` | `(x * 2 for x in data)` |
| Evaluation | Builds the list immediately | Produces values as requested |
| Memory | Stores all output elements | Usually stores only generator state and current values |
| Reusable output | List can be iterated repeatedly | A consumed generator does not restart automatically |
| Best suited for | When you need the complete list | Streaming or processing values one at a time |

A generator is especially useful when processing large datasets, but it does not automatically make every operation faster.

## 8. Nested List Comprehensions

A nested comprehension can process elements from multiple loops.

**Syntax:**

```python
[expression for outer_item in outer_iterable
            for inner_item in inner_iterable]
```

### Example 1: Flatten a nested list

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

flattened = [n for row in matrix for n in row]

print(flattened)
# [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

The order is equivalent to:

```python
flattened = []

for row in matrix:
    for n in row:
        flattened.append(n)
```

### Example 2: Generate coordinate pairs

```python
pairs = [
    (x, y)
    for x in range(2)
    for y in range(3)
]

print(pairs)
# [(0, 0), (0, 1), (0, 2),
#  (1, 0), (1, 1), (1, 2)]
```

The outer loop iterates over `x`; the inner loop iterates over `y` for each value of `x`.

### Example 3: Flatten and filter

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

evens = [
    n
    for row in matrix
    for n in row
    if n % 2 == 0
]

print(evens)
# [2, 4, 6, 8]
```

## 9. Multiple Conditions

A comprehension can contain multiple filtering conditions.

```python
numbers = range(1, 21)

result = [
    n for n in numbers
    if n % 2 == 0
    if n % 3 == 0
]

print(result)
# [6, 12, 18]
```

This is equivalent to checking both conditions with `and`:

```python
result = [
    n for n in numbers
    if n % 2 == 0 and n % 3 == 0
]
```

Use the form that makes the logic easiest to read.

## 10. Common Interview Patterns

### Pattern 1: Remove duplicates while preserving order

```python
numbers = [3, 1, 3, 2, 1, 4]

unique_numbers = list(dict.fromkeys(numbers))

print(unique_numbers)
# [3, 1, 2, 4]
```

A set comprehension alone cannot guarantee preserving the original order.

### Pattern 2: Extract values from dictionaries

```python
employees = [
    {"name": "Amit", "salary": 70000},
    {"name": "Priya", "salary": 50000},
    {"name": "Rahul", "salary": 85000}
]

high_earners = [
    employee["name"]
    for employee in employees
    if employee["salary"] > 60000
]

print(high_earners)
# ['Amit', 'Rahul']
```

### Pattern 3: Normalize strings

```python
names = [" Amit ", "PRIYA", " Rahul "]

cleaned = [name.strip().lower() for name in names]

print(cleaned)
# ['amit', 'priya', 'rahul']
```

### Pattern 4: Build a mapping from records

```python
employees = [
    {"id": 1, "name": "Amit"},
    {"id": 2, "name": "Priya"},
    {"id": 3, "name": "Rahul"}
]

employee_map = {
    employee["id"]: employee["name"]
    for employee in employees
}

print(employee_map)
# {1: 'Amit', 2: 'Priya', 3: 'Rahul'}
```

If keys repeat, later values overwrite earlier values in the resulting dictionary.

## 11. Time and Space Complexity

Let \(n\) be the number of input elements.

| Operation | Typical time complexity | Additional space |
|---|---|---|
| Simple list comprehension | \(O(n)\), assuming constant-time work per element | \(O(n)\) |
| Set comprehension | Expected \(O(n)\) for hashable values | \(O(n)\) |
| Dictionary comprehension | Expected \(O(n)\) for hashable keys | \(O(n)\) |
| Generator expression creation | \(O(1)\) to create the generator | \(O(1)\) generator state, excluding referenced input |
| Consume generator over \(n\) elements | Usually \(O(n)\) total | Usually small additional state |

These are typical costs. The actual complexity depends on the expression, hashing, nested loops, and any function called inside the comprehension.

**Example of a costly comprehension:**

```python
numbers = [1, 2, 3, 4, 5]

result = [n for n in numbers if n in numbers]
```

The membership test against a list can take \(O(n)\) per element, making the overall operation \(O(n^2)\).

If the goal is membership checking against many values, a set may be more appropriate:

```python
valid_numbers = set(numbers)

result = [n for n in numbers if n in valid_numbers]
```

## 12. Common Mistakes

### Mistake 1: Confusing filtering with transformation

```python
[n for n in numbers if n > 5]
```

Filters values.

```python
[n if n > 5 else 0 for n in numbers]
```

Transforms every value.

### Mistake 2: Using too many nested loops

```python
[
    (x, y, z)
    for x in range(10)
    for y in range(10)
    for z in range(10)
]
```

This creates 1,000 tuples. Nested comprehensions can grow rapidly and may consume substantial memory.

### Mistake 3: Writing unreadable comprehensions

If the expression requires multiple levels of conditions or complex calculations, a regular loop may be clearer.

### Mistake 4: Assuming generators can be reused

```python
numbers = (n for n in range(3))

print(list(numbers))  # [0, 1, 2]
print(list(numbers))  # []
```

Once consumed, a generator is exhausted.

### Mistake 5: Using comprehensions for side effects

Avoid using a comprehension only to call `print()`, update external state, or perform other side effects. Use a normal loop for such operations.

## 13. Interview Questions

1. What is a list comprehension?
2. What is the difference between list and set comprehensions?
3. What is the difference between a comprehension with `if` and one with `if-else`?
4. How do you flatten a two-dimensional list?
5. How do generator expressions reduce memory usage?
6. Can a dictionary comprehension produce duplicate keys?
7. What is the time complexity of a simple list comprehension?
8. When is a regular loop preferable to a comprehension?
9. How can you remove duplicates while preserving order?
10. How would you filter a list of dictionaries by a field?
11. What is the difference between `[expression for ...]` and `(expression for ...)`?
12. How would you avoid repeated linear membership checks inside a comprehension?

## 14. Practice Problems

Try these without looking at the solutions.

### Easy

1. Create a list of squares from 1 to 20.
2. Extract all odd numbers from a list.
3. Convert a list of names to lowercase.
4. Create a list containing the lengths of all words.
5. Extract numbers greater than 10.

### Medium

6. Flatten a nested list.
7. Create a dictionary mapping each number to its cube.
8. Extract names of employees earning more than ₹60,000 from a list of dictionaries.
9. Remove empty strings from a list.
10. Create a dictionary containing only values greater than 50.
11. Find all pairs `(x, y)` where `x + y == 10`.
12. Create a frequency dictionary for a list of values.

### Hard

13. Flatten a nested list and retain only unique even values in first-seen order.
14. Transform a list of dictionaries into a dictionary keyed by employee ID.
15. Given a matrix, extract all elements greater than the average of the matrix.
16. Use a generator expression to calculate the sum of squares without creating an intermediate list.
17. Given a list of strings, create a dictionary mapping each string to its vowel count.
18. Explain the time complexity of a nested comprehension that checks membership in a list.

## 15. Quick Revision

- **List comprehension:** `[expression for item in iterable]`
- **Filtering:** `[expression for item in iterable if condition]`
- **Conditional transformation:** `[a if condition else b for item in iterable]`
- **Set comprehension:** `{expression for item in iterable}`
- **Dictionary comprehension:** `{key: value for item in iterable}`
- **Generator expression:** `(expression for item in iterable)`
- **Nested loops:** Put `for` clauses in the same order as the equivalent regular loops.
- **Performance:** Consider the complexity of the expression and functions inside the comprehension.
- **Readability:** Prefer a regular loop when the comprehension becomes difficult to understand.

For Data Engineer interviews, focus on filtering and transforming lists, processing lists of dictionaries, flattening nested data, understanding generators, and recognizing when an apparently concise expression has poor time complexity.