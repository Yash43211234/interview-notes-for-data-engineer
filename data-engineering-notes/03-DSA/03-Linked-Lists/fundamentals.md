# Coding Fundamentals

## 1. Overview

Coding fundamentals are the basic concepts you need to understand before solving Data Structures and Algorithms (DSA) problems.

The goal is not just to write code that works. You should understand how the code executes, how data is stored, and how to reason about correctness and efficiency.

This note covers the core Python and problem-solving fundamentals required for coding interviews.

## 2. Variables and Data Types

A variable is a name that refers to an object.

```python
age = 26
price = 99.5
name = "Yash"
is_active = True
```

Common Python data types:

| Type | Example | Purpose |
|---|---|---|
| `int` | `10` | Integer numbers |
| `float` | `10.5` | Decimal numbers |
| `str` | `"hello"` | Text |
| `bool` | `True` | Boolean values |
| `list` | `[1, 2, 3]` | Ordered, mutable collection |
| `tuple` | `(1, 2, 3)` | Ordered, immutable collection |
| `set` | `{1, 2, 3}` | Unique elements |
| `dict` | `{"a": 1}` | Key-value mappings |
| `NoneType` | `None` | Absence of a value |

Check an object's type:

```python
x = 10
print(type(x))  # <class 'int'>
```

**Interview point:** Python is dynamically typed. A variable can refer to objects of different types at different times.

```python
x = 10
x = "hello"
```

This is valid Python because the variable name is rebound to another object.

## 3. Operators

### Arithmetic operators

```python
a = 10
b = 3

print(a + b)   # 13
print(a - b)   # 7
print(a * b)   # 30
print(a / b)   # 3.333...
print(a // b)  # 3
print(a % b)   # 1
print(a ** b)  # 1000
```

- `/` performs true division.
- `//` performs floor division.
- `%` returns the remainder.
- `**` performs exponentiation.

### Comparison operators

```python
print(5 == 5)  # True
print(5 != 3)  # True
print(5 > 3)   # True
print(5 <= 3)  # False
```

### Logical operators

```python
x = 10

print(x > 5 and x < 20)  # True
print(x < 5 or x == 10)  # True
print(not x == 10)       # False
```

Python's `and` and `or` use short-circuit evaluation: the second expression is evaluated only when necessary.

### Identity versus equality

```python
a = [1, 2]
b = [1, 2]

print(a == b)  # True: equal values
print(a is b)  # False: different objects
```

Use `==` to compare values and `is` when object identity matters, such as checking `x is None`.

## 4. Conditions

Conditions let a program choose which code to execute.

```python
score = 75

if score >= 90:
    result = "Excellent"
elif score >= 60:
    result = "Pass"
else:
    result = "Fail"
```

Python executes the first matching branch.

### Truthy and falsy values

The following values are falsy:

```python
False
None
0
0.0
""
[]
{}
set()
```

Most other objects are truthy.

```python
nums = []

if not nums:
    print("The list is empty")
```

**Interview point:** Prefer clear conditions over complicated nested branches. Check edge cases such as empty collections and zero explicitly when they matter.

## 5. Loops

### For loop

Use a `for` loop to iterate over an iterable.

```python
nums = [10, 20, 30]

for num in nums:
    print(num)
```

### Range

```python
for i in range(5):
    print(i)
```

Output: `0, 1, 2, 3, 4`

`range(start, stop, step)` excludes the stop value.

```python
for i in range(2, 10, 2):
    print(i)  # 2, 4, 6, 8
```

### While loop

Use a `while` loop when repetition depends on a condition.

```python
n = 5

while n > 0:
    print(n)
    n -= 1
```

### Break and continue

```python
for num in range(10):
    if num == 3:
        continue

    if num == 7:
        break

    print(num)
```

- `continue` skips the rest of the current iteration.
- `break` exits the loop.

**Common mistake:** Forgetting to update a variable in a `while` loop can create an infinite loop.

## 6. Functions

Functions package reusable logic.

```python
def add(a, b):
    return a + b

result = add(3, 4)
print(result)  # 7
```

- Parameters are names defined in the function.
- Arguments are values passed during the call.
- `return` sends a result back to the caller.
- `print()` displays information but does not replace a return value.

```python
def square(n):
    print(n * n)

result = square(4)
print(result)  # None
```

The function prints `16`, but returns `None`.

### Default parameters

```python
def greet(name, message="Hello"):
    return f"{message}, {name}"

print(greet("Sam"))
print(greet("Sam", "Welcome"))
```

**Interview point:** Avoid mutable default arguments such as `def add_item(item, items=[])`. That list is reused across calls. Use `None` and create a new list inside the function instead.

## 7. Collections and Data Structures

Choose the structure based on the operations required.

| Structure | Common use | Typical operation |
|---|---|---|
| List | Ordered sequence | Index access: `O(1)` |
| Tuple | Fixed collection | Index access: `O(1)` |
| Set | Membership and uniqueness | Membership: expected `O(1)` |
| Dictionary | Key-value lookup | Lookup: expected `O(1)` |
| Stack | Last-in, first-out | Append/pop at end: amortized `O(1)` |
| Queue | First-in, first-out | `deque` append/pop at ends: `O(1)` |

The complexities are typical for Python's built-in implementations, not guarantees for every possible operation.

### List

```python
nums = [10, 20, 30]

nums.append(40)
nums[0] = 5

print(nums)
```

### Set

```python
seen = set()
seen.add(10)

print(10 in seen)  # True
```

### Dictionary

```python
counts = {}

for char in "banana":
    counts[char] = counts.get(char, 0) + 1

print(counts)
```

**Interview point:** Choosing an appropriate data structure often improves a solution more than changing a few lines of code.

## 8. Indexing and Slicing

Python uses zero-based indexing.

```python
nums = [10, 20, 30, 40]

print(nums[0])   # 10
print(nums[-1])  # 40
print(nums[1:3]) # [20, 30]
```

The slice `nums[start:stop]` includes `start` but excludes `stop`.

```python
print(nums[:2])   # [10, 20]
print(nums[2:])   # [30, 40]
print(nums[::-1]) # [40, 30, 20, 10]
```

**Common mistake:** Accessing an invalid index raises `IndexError`, while slicing beyond the sequence's bounds is generally allowed.

## 9. Mutability and References

Mutable objects can be changed after creation. Lists, dictionaries, and sets are mutable. Integers, strings, and tuples are immutable, although a tuple can contain mutable objects.

```python
a = [1, 2]
b = a

b.append(3)

print(a)  # [1, 2, 3]
```

Both variables refer to the same list.

To create a shallow copy:

```python
a = [1, 2]
b = a.copy()

b.append(3)

print(a)  # [1, 2]
print(b)  # [1, 2, 3]
```

A shallow copy does not recursively copy nested mutable objects.

**Interview point:** Understand whether a function changes the original object or creates a new one.

## 10. Recursion

Recursion occurs when a function calls itself.

Every recursive solution needs:

1. A **base case** that stops recursion.
2. A **recursive case** that reduces the problem.

```python
def factorial(n):
    if n < 0:
        raise ValueError("n must be non-negative")

    if n <= 1:
        return 1

    return n * factorial(n - 1)
```

For `factorial(4)`:

`4 × 3 × 2 × 1 = 24`

- Time: `O(n)`
- Call-stack space: `O(n)`

**Common mistakes:**
- Missing a base case.
- Failing to make progress toward the base case.
- Exceeding Python's recursion limit.
- Ignoring the call-stack space required.

## 11. Basic Input and Output

```python
name = input("Enter your name: ")
print(name)
```

`input()` returns a string.

Convert it when numeric input is required:

```python
n = int(input())
price = float(input())
```

Read multiple integers:

```python
nums = list(map(int, input().split()))
```

For coding platforms, follow the required input format. When the platform supplies a function signature, implement the function instead of adding unnecessary input-handling code.

## 12. Errors and Exceptions

Common errors include:

- `SyntaxError`: invalid Python syntax.
- `NameError`: a name is not defined.
- `TypeError`: an operation is incompatible with a value's type.
- `IndexError`: an invalid sequence index.
- `KeyError`: a dictionary key is missing.
- `ValueError`: a value has the right general type but is invalid for the operation.
- `ZeroDivisionError`: division by zero.

```python
try:
    number = int("abc")
except ValueError:
    print("Invalid integer")
```

**Interview point:** Handle expected failure cases deliberately. Do not use broad exception handling to hide programming mistakes.

## 13. Time Complexity Fundamentals

Time complexity describes how an algorithm's running time grows as the input size increases.

| Complexity | Name | Example |
|---|---|---|
| `O(1)` | Constant | Access a list element by index |
| `O(log n)` | Logarithmic | Binary search |
| `O(n)` | Linear | Scan a list once |
| `O(n log n)` | Linearithmic | Typical efficient comparison sorting |
| `O(n²)` | Quadratic | Compare every pair using nested loops |
| `O(2ⁿ)` | Exponential | Some recursive subset-enumeration algorithms |
| `O(n!)` | Factorial | Enumerate every permutation |

Example:

```python
for num in nums:
    print(num)
```

One traversal takes `O(n)` time.

```python
for a in nums:
    for b in nums:
        print(a, b)
```

Two nested traversals over the same input take `O(n²)` time.

**Important:** Complexity describes growth, not an exact execution time. Two `O(n)` solutions can still have different practical performance.

## 14. Space Complexity Fundamentals

Space complexity describes how memory requirements grow with input size.

```python
def find_sum(nums):
    total = 0

    for num in nums:
        total += num

    return total
```

The algorithm uses `O(1)` auxiliary space because it keeps only a constant number of extra variables.

```python
def make_squares(nums):
    result = []

    for num in nums:
        result.append(num * num)

    return result
```

This uses `O(n)` auxiliary space for the output list.

**Important:** Auxiliary space generally means extra memory used by the algorithm, excluding the input. State your convention clearly when analyzing an output-producing function.

## 15. Edge Cases

Before submitting a solution, test cases that may break assumptions.

- Empty input.
- One element.
- Duplicate values.
- Negative values.
- All values equal.
- Already sorted input.
- Reverse-sorted input.
- Missing target.
- Zero or very large values.
- Even and odd lengths.
- Invalid input, when the problem requires validation.

Example:

```python
def find_max(nums):
    if not nums:
        return None

    best = nums[0]

    for num in nums[1:]:
        if num > best:
            best = num

    return best
```

Initializing `best` with the first element ensures that an array containing only negative values is handled correctly.

## 16. Correctness and Debugging

A solution is correct when it produces the required result for every valid input, not just the sample input.

Use this debugging process:

1. Check the problem's assumptions.
2. Trace variables on a small example.
3. Inspect loop boundaries and conditions.
4. Verify what happens at the first and last iteration.
5. Test edge cases.
6. Check that the return value matches the expected output.
7. Analyze time and space complexity.

For tricky algorithms, write down what each variable represents and what must remain true after every iteration. These properties are called **invariants**.

## 17. Common Interview Mistakes

- Coding before understanding the problem.
- Ignoring constraints and input assumptions.
- Using nested loops when hashing or two pointers would be more efficient.
- Confusing `==` with `is`.
- Forgetting that `range()` excludes its stop value.
- Modifying a list while iterating over it without understanding the consequences.
- Returning `None` unintentionally.
- Forgetting empty-input cases.
- Claiming `O(1)` space while allocating an additional list of size `n`.
- Memorizing code without being able to explain why it works.

## 18. Practice Checklist

- [ ] Find the sum and average of a list.
- [ ] Find the minimum and maximum in one traversal.
- [ ] Count the frequency of each element.
- [ ] Reverse a string.
- [ ] Check whether a string is a palindrome.
- [ ] Remove duplicates from a list.
- [ ] Find a missing number.
- [ ] Explain the time complexity of a nested loop.
- [ ] Explain the difference between a shallow copy and an alias.
- [ ] Write a recursive function with a valid base case.
- [ ] Identify time and auxiliary-space complexity.
- [ ] Test a function with empty and boundary inputs.

## 19. Quick Revision

- Variables refer to objects; Python is dynamically typed.
- Lists are mutable; strings are immutable.
- `==` compares values; `is` compares object identity.
- `range()` excludes the stop value.
- Functions use `return` to send results back.
- Sets and dictionaries provide expected constant-time membership or lookup in typical cases.
- Recursion requires a base case and uses call-stack space.
- Time complexity measures growth in running time.
- Space complexity measures memory growth.
- Edge-case testing and clear reasoning are essential for correct interview solutions.

**Final principle:** Build a strong understanding of Python execution, data structures, complexity, and edge cases before moving to advanced DSA patterns.