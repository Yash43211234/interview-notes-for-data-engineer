# Conditions and Loops in Python

## 1. Overview

Conditions and loops control the flow of a Python program.

- **Conditions** help a program make decisions.
- **Loops** allow a program to repeat instructions.
- **`break`** stops a loop.
- **`continue`** skips the current iteration.
- **`pass`** acts as a placeholder when a statement is required but no action is needed yet.

These concepts are essential for solving DSA problems, processing data, and writing Python scripts for data engineering.

---

## 2. Conditional Statements

### 2.1 `if`

Executes a block only when a condition is `True`.

```python
age = 22

if age >= 18:
    print("Eligible")
```

Output:

```text
Eligible
```

Python uses indentation to define the block.

### 2.2 `if-else`

Executes one block when the condition is true and another when it is false.

```python
number = 7

if number % 2 == 0:
    print("Even")
else:
    print("Odd")
```

Output:

```text
Odd
```

### 2.3 `if-elif-else`

Use `elif` when multiple conditions must be checked in order.

```python
marks = 75

if marks >= 90:
    grade = "A"
elif marks >= 70:
    grade = "B"
elif marks >= 50:
    grade = "C"
else:
    grade = "Fail"

print(grade)
```

Output:

```text
B
```

**Important:** Python checks the conditions from top to bottom and executes the first matching branch. The remaining branches are skipped.

Put more specific or higher-priority conditions first when the order matters.

### 2.4 Nested conditions

A condition can contain another condition.

```python
age = 25
has_id = True

if age >= 18:
    if has_id:
        print("Entry allowed")
    else:
        print("ID required")
else:
    print("Underage")
```

Nested conditions are useful when one decision depends on another. However, too much nesting can make code difficult to read.

### 2.5 Conditional expression (ternary expression)

Use this for a simple choice between two values.

```python
number = 10

result = "Even" if number % 2 == 0 else "Odd"
print(result)
```

Output:

```text
Even
```

General syntax:

```python
value_if_true if condition else value_if_false
```

Prefer a normal `if-else` statement when the logic is complicated.

---

## 3. Truthy and Falsy Values

Python conditions do not always need explicit comparisons such as `x == True`.

The following values are falsy:

- `False`
- `None`
- Numeric zero, such as `0` and `0.0`
- Empty strings, such as `""`
- Empty collections, such as `[]`, `{}`, `()`, and `set()`

Most other values are truthy, including non-empty collections and non-zero numbers.

```python
items = []

if items:
    print("Data available")
else:
    print("No data")
```

Output:

```text
No data
```

This is a common way to check whether a collection contains elements.

### Check for `None` correctly

```python
value = None

if value is None:
    print("No value")
```

Use `is None` rather than `== None` for an explicit `None` check.

**Common mistake:**

```python
items = []

if items is None:
    print("Missing")
```

This does not print anything because an empty list is not the same as `None`.

- `None` means no value is present.
- `[]` means a list exists but contains no elements.

---

## 4. Logical Operators

Logical operators combine conditions.

| Operator | Meaning | Example |
|---|---|---|
| `and` | Both conditions must be true | `age >= 18 and has_id` |
| `or` | At least one condition must be true | `is_admin or is_owner` |
| `not` | Reverses truth value | `not is_active` |

Example:

```python
age = 25
has_id = True

if age >= 18 and has_id:
    print("Allowed")
```

### Short-circuit evaluation

Python may stop evaluating a logical expression as soon as its result is known.

With `and`, if the first operand is falsy, Python does not evaluate the second operand.

With `or`, if the first operand is truthy, Python does not evaluate the second operand.

```python
items = []

if items and items[0] > 10:
    print("Large first item")
```

This is safe because `items[0]` is evaluated only if `items` is non-empty.

**Interview point:** `and` and `or` return one of their operands, not necessarily a Boolean value.

```python
print(0 or 5)       # 5
print("hello" and 7)  # 7
```

---

## 5. Comparison Operators

| Operator | Meaning |
|---|---|
| `==` | Equal to |
| `!=` | Not equal to |
| `>` | Greater than |
| `<` | Less than |
| `>=` | Greater than or equal to |
| `<=` | Less than or equal to |
| `is` | Same object identity |
| `in` | Membership test |

Python supports chained comparisons:

```python
age = 25

if 18 <= age <= 60:
    print("Within range")
```

This is equivalent to:

```python
if age >= 18 and age <= 60:
    print("Within range")
```

Use `==` for value equality and `is` for object identity. Do not use `is` to compare ordinary numbers or strings.

---

## 6. The `for` Loop

A `for` loop iterates over the elements of an iterable, such as a list, tuple, string, set, dictionary, or range.

### 6.1 Loop through a list

```python
numbers = [10, 20, 30]

for number in numbers:
    print(number)
```

Output:

```text
10
20
30
```

Python assigns each element to `number` in turn.

### 6.2 Loop through a string

```python
for char in "Python":
    print(char)
```

Each iteration processes one character.

### 6.3 Using `range()`

`range()` generates a sequence of integers.

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

Common forms:

```python
range(stop)
range(start, stop)
range(start, stop, step)
```

Examples:

```python
list(range(5))        # [0, 1, 2, 3, 4]
list(range(2, 6))     # [2, 3, 4, 5]
list(range(1, 10, 2)) # [1, 3, 5, 7, 9]
list(range(5, 0, -1)) # [5, 4, 3, 2, 1]
```

**Important:** The stop value is excluded.

In Python 3, `range()` produces a range object rather than creating a list of all the numbers immediately.

### 6.4 Using `enumerate()`

Use `enumerate()` when you need both the index and the value.

```python
names = ["Aman", "Riya", "Rahul"]

for index, name in enumerate(names):
    print(index, name)
```

Output:

```text
0 Aman
1 Riya
2 Rahul
```

Start indexing from 1 when required:

```python
for index, name in enumerate(names, start=1):
    print(index, name)
```

Prefer `enumerate()` over manually maintaining a counter.

### 6.5 Using `zip()`

Use `zip()` to iterate over multiple iterables together.

```python
names = ["Aman", "Riya"]
scores = [85, 92]

for name, score in zip(names, scores):
    print(name, score)
```

Output:

```text
Aman 85
Riya 92
```

By default, `zip()` stops when the shortest iterable is exhausted.

---

## 7. The `while` Loop

A `while` loop repeats as long as its condition remains true.

```python
count = 1

while count <= 5:
    print(count)
    count += 1
```

Output:

```text
1
2
3
4
5
```

### How it works

1. Check the condition.
2. If true, execute the loop body.
3. Update the relevant variables.
4. Check the condition again.
5. Stop when the condition becomes false.

### Infinite loops

An infinite loop occurs when the stopping condition never becomes false.

```python
count = 1

while count <= 5:
    print(count)
    # Missing count += 1
```

This loop continues printing `1` indefinitely.

To avoid this, make sure the loop has a valid stopping condition and that the state can change to reach it.

### When to use `while` versus `for`

- Use `for` when iterating over a collection or a known sequence.
- Use `while` when repetition depends on a condition that changes during execution.

For example, processing each row in a list is usually a `for` loop. Repeating an operation until a target is reached may be a `while` loop.

---

## 8. `break`, `continue`, and `pass`

### 8.1 `break`

Immediately terminates the nearest enclosing loop.

```python
for number in range(1, 6):
    if number == 3:
        break
    print(number)
```

Output:

```text
1
2
```

The loop stops when `number` becomes `3`.

### 8.2 `continue`

Skips the rest of the current iteration and moves to the next iteration.

```python
for number in range(1, 6):
    if number == 3:
        continue
    print(number)
```

Output:

```text
1
2
4
5
```

### 8.3 `pass`

Does nothing. It is useful when Python syntax requires a statement but the implementation is not ready.

```python
for number in range(5):
    if number == 2:
        pass
    print(number)
```

Output:

```text
0
1
2
3
4
```

Unlike `break` and `continue`, `pass` does not change the loop's control flow.

### Quick comparison

| Statement | Effect |
|---|---|
| `break` | Exits the nearest loop |
| `continue` | Skips to the next iteration |
| `pass` | Does nothing |

---

## 9. Loop `else`

Python allows an `else` block with both `for` and `while` loops.

The `else` block runs when the loop finishes normally, including when a `for` loop exhausts its iterable or a `while` condition becomes false. It does **not** run if the loop exits through `break`.

Example: Search for a value.

```python
numbers = [10, 20, 30, 40]
target = 25

for number in numbers:
    if number == target:
        print("Found")
        break
else:
    print("Not found")
```

Output:

```text
Not found
```

Because no `break` occurred, the `else` block executes.

If `target = 30`, the output is `Found`, and the `else` block is skipped.

This feature is useful for search operations, prime-number checks, and validation logic, although a flag or a separate function may be clearer in some situations.

---

## 10. Nested Loops

A nested loop is a loop inside another loop.

```python
for i in range(1, 4):
    for j in range(1, 3):
        print(i, j)
```

Output:

```text
1 1
1 2
2 1
2 2
3 1
3 2
```

For each iteration of the outer loop, the inner loop completes all its iterations.

### Time complexity

If the outer loop runs \(n\) times and the inner loop runs \(m\) times per outer iteration, the total number of inner-body executions is \(n \times m\).

Therefore, the time complexity is:

\[
O(nm)
\]

If both loops run \(n\) times, the complexity is \(O(n^2)\).

Nested loops are common in matrix operations, pair comparisons, and brute-force algorithms.

---

## 11. Looping Through Dictionaries

Dictionaries contain key-value pairs.

```python
employee = {
    "name": "Aman",
    "salary": 50000,
    "department": "Data"
}
```

Loop through keys:

```python
for key in employee:
    print(key)
```

Loop through values:

```python
for value in employee.values():
    print(value)
```

Loop through both keys and values:

```python
for key, value in employee.items():
    print(key, value)
```

Use `.items()` when both the key and value are needed.

---

## 12. Modifying Collections During a Loop

Modifying a collection while iterating over it can cause unexpected behavior, particularly when deleting elements from a list.

**Problematic approach:**

```python
numbers = [1, 2, 3, 4, 5]

for number in numbers:
    if number % 2 == 0:
        numbers.remove(number)
```

Removing elements shifts subsequent elements and can cause some elements to be skipped.

**Better approach: create a new list.**

```python
numbers = [1, 2, 3, 4, 5]

numbers = [
    number for number in numbers
    if number % 2 != 0
]

print(numbers)
```

Output:

```text
[1, 3, 5]
```

Alternatively, iterate over a copy when appropriate:

```python
for number in numbers[:]:
    if number % 2 == 0:
        numbers.remove(number)
```

Creating a new list is often easier to reason about and avoids changing the collection being traversed.

---

## 13. Common Interview Patterns

### 13.1 Sum of numbers

```python
numbers = [10, 20, 30, 40]
total = 0

for number in numbers:
    total += number

print(total)
```

Output: `100`

Time complexity: \(O(n)\).

### 13.2 Count even numbers

```python
numbers = [1, 2, 3, 4, 5, 6]
count = 0

for number in numbers:
    if number % 2 == 0:
        count += 1

print(count)
```

Output: `3`

Time complexity: \(O(n)\).

### 13.3 Find the maximum without `max()`

```python
numbers = [12, 5, 30, 8, 19]

largest = numbers[0]

for number in numbers[1:]:
    if number > largest:
        largest = number

print(largest)
```

Output: `30`

This assumes the list is non-empty. Handle empty lists separately when necessary.

Time complexity: \(O(n)\).

### 13.4 Reverse a string using a loop

```python
text = "python"
reversed_text = ""

for char in text:
    reversed_text = char + reversed_text

print(reversed_text)
```

Output: `nohtyp`

This is easy to understand but can take \(O(n^2)\) time because strings are immutable and each concatenation may create a new string.

A more efficient loop-based approach is:

```python
text = "python"
result = []

for char in text:
    result.append(char)

result.reverse()

print("".join(result))
```

This takes \(O(n)\) time and \(O(n)\) additional space.

### 13.5 Check whether a number is prime

```python
number = 29

if number < 2:
    is_prime = False
else:
    is_prime = True

    for divisor in range(2, int(number ** 0.5) + 1):
        if number % divisor == 0:
            is_prime = False
            break

print(is_prime)
```

Output: `True`

Only divisors up to the square root need to be checked because if a number has a factor greater than its square root, it must also have a corresponding factor smaller than its square root.

Time complexity: \(O(\sqrt{n})\).

### 13.6 Find the first matching value

```python
numbers = [4, 8, 15, 16, 23, 42]
target = 16

for number in numbers:
    if number == target:
        print("Found")
        break
else:
    print("Not found")
```

This is a linear search with \(O(n)\) worst-case time complexity.

---

## 14. Common Mistakes

### Mistake 1: Using `=` instead of `==`

```python
# Incorrect
# if number = 5:

# Correct
if number == 5:
    print("Five")
```

`=` assigns a value; `==` compares values.

### Mistake 2: Forgetting to update a `while` loop

A missing update can cause an infinite loop.

### Mistake 3: Incorrect indentation

Python uses indentation to identify blocks. Inconsistent indentation can cause errors or change program behavior.

### Mistake 4: Off-by-one errors with `range()`

```python
range(1, 5)
```

Produces `1, 2, 3, 4`, not `1, 2, 3, 4, 5`.

### Mistake 5: Using `break` when you mean `continue`

- `break` stops the loop.
- `continue` skips only the current iteration.

### Mistake 6: Confusing `is` and `==`

Use `==` for value comparison and `is` for object identity.

### Mistake 7: Assuming `for` loops always use indexes

Python's `for` loop iterates over values by default. Use `enumerate()` when an index is needed.

### Mistake 8: Assuming loop `else` runs only when there is no error

Loop `else` is about normal loop completion versus exiting through `break`. It is not a general exception handler.

---

## 15. Time Complexity of Loops

| Pattern | Typical time complexity |
|---|---|
| One loop over \(n\) elements | \(O(n)\) |
| Two sequential loops over \(n\) elements | \(O(n)\) |
| Nested loops, each running \(n\) times | \(O(n^2)\) |
| Three nested loops, each running \(n\) times | \(O(n^3)\) |
| Loop that halves the input each iteration | \(O(\log n)\) |
| Outer loop \(n\) times and inner loop \(m\) times | \(O(nm)\) |

**Important:** Sequential loops are added, not automatically multiplied.

```python
for i in range(n):
    print(i)

for j in range(n):
    print(j)
```

The total work is \(n+n=2n\), so the time complexity is \(O(n)\).

By contrast:

```python
for i in range(n):
    for j in range(n):
        print(i, j)
```

The total work is \(n^2\), so the time complexity is \(O(n^2)\).

Always analyze how many times the loop body executes rather than judging complexity only by the number of loops.

---

## 16. Interview Questions

### Conceptual questions

1. What is the difference between `for` and `while`?
2. What is the difference between `break`, `continue`, and `pass`?
3. What does `range(1, 10, 2)` produce?
4. What are truthy and falsy values in Python?
5. How does short-circuit evaluation work?
6. What is the purpose of `enumerate()`?
7. What happens when a loop terminates through `break`?
8. When does a loop's `else` block execute?
9. What is an infinite loop, and how can it be avoided?
10. Why can modifying a list during iteration be problematic?
11. What is the time complexity of two nested loops?
12. Why is a loop over a list usually \(O(n)\)?

### Output-based questions

**Question 1**

```python
for i in range(2, 7, 2):
    print(i, end=" ")
```

Answer:

```text
2 4 6
```

**Question 2**

```python
for i in range(5):
    if i == 2:
        continue
    print(i, end=" ")
```

Answer:

```text
0 1 3 4
```

**Question 3**

```python
count = 0

while count < 3:
    print(count, end=" ")
    count += 1
else:
    print("Done")
```

Answer:

```text
0 1 2 Done
```

The loop finishes normally, so its `else` block runs.

**Question 4**

```python
for i in range(3):
    for j in range(2):
        print("*", end="")
```

Answer:

```text
******
```

The inner statement executes \(3 \times 2 = 6\) times.

---

## 17. Practice Problems

Try solving these without looking at the solutions first.

### Easy

1. Print numbers from 1 to 100.
2. Print all even numbers between 1 and 50.
3. Calculate the sum of the first \(n\) positive integers.
4. Count the number of vowels in a string.
5. Count positive, negative, and zero values in a list.
6. Find the largest element in a non-empty list.
7. Reverse a string using a loop.
8. Print the multiplication table of a given number.

### Medium

1. Check whether a number is prime.
2. Print all prime numbers between 1 and \(n\).
3. Calculate the factorial of a number.
4. Check whether a number is a palindrome.
5. Find the sum of the digits of an integer.
6. Find the second-largest distinct element in a list.
7. Print the Fibonacci sequence for \(n\) terms.
8. Find all pairs of elements whose sum equals a target using nested loops.

### DSA-focused

1. Implement linear search.
2. Find duplicate elements in a list.
3. Move all zeros to the end of a list.
4. Find the maximum and minimum in a single traversal.
5. Print a right-angled triangle using nested loops.
6. Find the first repeated element in a list.
7. Determine whether a list is sorted.
8. Count the frequency of each element using a dictionary.

For every solution, explain the approach, time complexity, and additional space complexity.

---

## 18. Quick Revision

- `if` executes a block when a condition is true.
- `elif` checks another condition if previous branches did not match.
- `else` handles the remaining case.
- `for` iterates over an iterable.
- `while` repeats while its condition is true.
- `range(stop)` excludes the stop value.
- `enumerate()` provides both index and value.
- `zip()` iterates over multiple iterables together.
- `break` exits the nearest loop.
- `continue` skips the current iteration.
- `pass` does nothing.
- A loop's `else` executes if the loop finishes without `break`.
- One full traversal of \(n\) elements is typically \(O(n)\).
- Two nested loops of size \(n\) are typically \(O(n^2)\).
- Avoid changing a collection while iterating over it unless the behavior is carefully controlled.

**Recommended learning order:** Understand `if-else`, practice `for` loops and `range()`, learn `while`, then master `break` and `continue`. Finish by solving the easy practice problems before moving to nested loops and DSA patterns.