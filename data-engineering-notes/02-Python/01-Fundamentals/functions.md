# Functions in Python

## 1. Overview

A **function** is a reusable block of code designed to perform a specific task. It can accept inputs, process them, and return a result.

Functions help you:
- Avoid repeating code.
- Break complex problems into smaller parts.
- Make programs easier to test and maintain.
- Reuse logic across scripts and data pipelines.

```python
def add(a, b):
    return a + b

result = add(10, 20)
print(result)
```

Output:

```text
30
```

Here:
- `def` defines a function.
- `add` is the function name.
- `a` and `b` are parameters.
- `10` and `20` are arguments.
- `return` sends the result back to the caller.

---

## 2. Defining and Calling Functions

### 2.1 Basic function

```python
def greet():
    print("Hello, Python!")

greet()
```

Output:

```text
Hello, Python!
```

Defining a function does not normally execute its body. The body runs when the function is called.

### 2.2 Function with parameters

```python
def greet(name):
    print(f"Hello, {name}!")

greet("Yash")
```

Output:

```text
Hello, Yash!
```

`name` is a parameter, and `"Yash"` is the argument passed to it.

### 2.3 Function with a return value

```python
def square(number):
    return number * number

result = square(5)
print(result)
```

Output:

```text
25
```

A returned value can be stored, passed to another function, or used in an expression.

### `print()` vs `return`

```python
def add_print(a, b):
    print(a + b)

def add_return(a, b):
    return a + b

x = add_print(2, 3)
y = add_return(2, 3)

print(x)
print(y)
```

Output:

```text
5
None
5
```

The first function prints the result but does not return it. Python therefore returns `None` implicitly.

The second function returns the result, making it available for further computation.

**Interview takeaway:** `print()` displays information; `return` sends a value back to the caller.

---

## 3. Parameters and Arguments

A parameter is a variable listed in the function definition. An argument is the actual value supplied when the function is called.

```python
def multiply(a, b):  # a and b are parameters
    return a * b

multiply(4, 5)       # 4 and 5 are arguments
```

### 3.1 Positional arguments

Values are assigned based on their position.

```python
def describe(name, age):
    return f"{name} is {age} years old"

print(describe("Riya", 25))
```

Output:

```text
Riya is 25 years old
```

### 3.2 Keyword arguments

Values are assigned using parameter names.

```python
print(describe(age=25, name="Riya"))
```

Output:

```text
Riya is 25 years old
```

Keyword arguments make calls easier to read and allow arguments to be supplied in a different order.

### 3.3 Mixing positional and keyword arguments

```python
def calculate(a, b, c):
    return a + b + c

print(calculate(10, c=30, b=20))
```

Output:

```text
60
```

Positional arguments must come before keyword arguments in a function call.

---

## 4. Default Parameters

A default parameter has a value that is used when the caller does not provide an argument.

```python
def greet(name, message="Welcome"):
    return f"{message}, {name}!"

print(greet("Aman"))
print(greet("Riya", "Good morning"))
```

Output:

```text
Welcome, Aman!
Good morning, Riya!
```

Parameters with defaults generally appear after parameters without defaults.

```python
def calculate(price, discount=0):
    return price * (1 - discount / 100)
```

Both calls are valid:

```python
calculate(1000)
calculate(1000, 10)
```

### Common mistake: Mutable default arguments

Avoid using a mutable object such as a list or dictionary as a default when you intend to create a fresh object for each call.

**Problematic example:**

```python
def add_item(item, items=[]):
    items.append(item)
    return items

print(add_item("A"))
print(add_item("B"))
```

Output:

```text
['A']
['A', 'B']
```

The same default list is reused across calls because default parameter values are evaluated once when the function is defined.

**Better approach:**

```python
def add_item(item, items=None):
    if items is None:
        items = []

    items.append(item)
    return items

print(add_item("A"))
print(add_item("B"))
```

Output:

```text
['A']
['B']
```

Use `None` when you want to create a fresh mutable object for each call.

---

## 5. Return Statements

The `return` statement ends the current function call and optionally provides a value.

### 5.1 Returning a single value

```python
def cube(number):
    return number ** 3

print(cube(3))
```

Output:

```text
27
```

### 5.2 Returning multiple values

Python functions can return multiple values, which are packaged into a tuple.

```python
def calculate(a, b):
    return a + b, a - b

addition, subtraction = calculate(10, 4)

print(addition)
print(subtraction)
```

Output:

```text
14
6
```

The return statement effectively returns a tuple containing the two results.

### 5.3 Returning early

```python
def check_number(number):
    if number < 0:
        return "Negative"

    if number == 0:
        return "Zero"

    return "Positive"

print(check_number(-5))
```

Output:

```text
Negative
```

An early return can reduce unnecessary nesting and make a function easier to understand.

### 5.4 Returning `None`

A function returns `None` when it reaches the end without executing a return statement, or when it executes `return` without a value.

```python
def show_message():
    print("Processing")

result = show_message()

print(result)
```

Output:

```text
Processing
None
```

---

## 6. Variable Scope: Local and Global

Scope determines where a variable can be accessed.

### 6.1 Local variables

A variable created inside a function is normally local to that function.

```python
def calculate():
    result = 10 + 20
    return result

print(calculate())
```

The local variable `result` is available inside the function but is not automatically available outside it.

### 6.2 Global variables

A variable defined at the module level can normally be read inside a function.

```python
tax_rate = 0.1

def calculate_tax(amount):
    return amount * tax_rate

print(calculate_tax(1000))
```

Output:

```text
100.0
```

### 6.3 The `global` keyword

Use `global` when a function needs to reassign a module-level variable.

```python
count = 0

def increment():
    global count
    count += 1

increment()
print(count)
```

Output:

```text
1
```

Global state can make code harder to test and maintain. Prefer returning a new value or passing the relevant state as an argument when practical.

### 6.4 The `nonlocal` keyword

`nonlocal` allows a nested function to reassign a variable defined in an enclosing function.

```python
def outer():
    count = 0

    def inner():
        nonlocal count
        count += 1

    inner()
    inner()

    return count

print(outer())
```

Output:

```text
2
```

Use `nonlocal` when nested functions need to maintain and update enclosing-function state.

### Scope summary

| Scope | Meaning |
|---|---|
| Local | Defined inside the current function |
| Enclosing | Defined in an enclosing function |
| Global | Defined at the module level |
| Built-in | Names provided by Python, such as `len()` |

Python commonly resolves names using the **LEGB** rule: Local, Enclosing, Global, Built-in.

---

## 7. `*args` and `**kwargs`

These features allow functions to accept a variable number of arguments.

### 7.1 `*args`

`*args` collects extra positional arguments into a tuple.

```python
def total(*args):
    return sum(args)

print(total(10, 20, 30))
print(total(5, 15))
```

Output:

```text
60
20
```

The name `args` is conventional, not mandatory. The `*` is what enables this behavior.

### 7.2 `**kwargs`

`**kwargs` collects extra keyword arguments into a dictionary.

```python
def display_details(**kwargs):
    for key, value in kwargs.items():
        print(key, value)

display_details(name="Aman", age=25, role="Data Engineer")
```

Output:

```text
name Aman
age 25
role Data Engineer
```

### 7.3 Using both

```python
def show_data(*args, **kwargs):
    print(args)
    print(kwargs)

show_data(10, 20, name="Riya", city="Delhi")
```

Output:

```text
(10, 20)
{'name': 'Riya', 'city': 'Delhi'}
```

When defining a function with both, `*args` comes before `**kwargs`, with any ordinary parameters placed appropriately before them.

### 7.4 Unpacking arguments

The same operators can unpack an iterable or mapping into a function call.

```python
def add(a, b, c):
    return a + b + c

numbers = [10, 20, 30]

print(add(*numbers))
```

Output:

```text
60
```

For keyword arguments:

```python
def greet(name, message):
    return f"{message}, {name}"

details = {"name": "Aman", "message": "Hello"}

print(greet(**details))
```

Output:

```text
Hello, Aman
```

---

## 8. Lambda Functions

A lambda function is a small anonymous function consisting of a single expression.

```python
square = lambda number: number ** 2

print(square(5))
```

Output:

```text
25
```

Equivalent regular function:

```python
def square(number):
    return number ** 2
```

### Common use: `sorted()`

```python
employees = [
    {"name": "Aman", "salary": 50000},
    {"name": "Riya", "salary": 70000},
    {"name": "Rahul", "salary": 60000}
]

sorted_employees = sorted(
    employees,
    key=lambda employee: employee["salary"]
)

print(sorted_employees)
```

The `key` function determines the value used for sorting.

### Lambda limitations

A lambda contains one expression, not a sequence of ordinary statements.

Use a regular `def` function when the logic needs multiple steps, validation, documentation, or a meaningful reusable name.

---

## 9. Higher-Order Functions

A higher-order function accepts another function as an argument, returns a function, or does both.

### 9.1 Passing a function as an argument

```python
def apply_operation(a, b, operation):
    return operation(a, b)

def add(a, b):
    return a + b

def multiply(a, b):
    return a * b

print(apply_operation(3, 4, add))
print(apply_operation(3, 4, multiply))
```

Output:

```text
7
12
```

Notice that `add` and `multiply` are passed without parentheses. Passing `add` supplies the function itself; `add(3, 4)` would call it immediately.

### 9.2 Returning a function

```python
def make_multiplier(factor):
    def multiply(number):
        return number * factor

    return multiply

double = make_multiplier(2)

print(double(5))
```

Output:

```text
10
```

This demonstrates a closure: the inner function retains access to the enclosing variable `factor`.

Higher-order functions are useful for reusable transformations, callbacks, decorators, and data-processing utilities.

---

## 10. Recursive Functions

A recursive function calls itself to solve a smaller version of the same problem.

A recursive function typically needs:

1. A **base case** that stops recursion.
2. A **recursive case** that moves toward the base case.

### Example: Factorial

The factorial of a non-negative integer \(n\) is:

\[
n! = n \times (n-1) \times \cdots \times 1
\]

And \(0! = 1\).

```python
def factorial(n):
    if n < 0:
        raise ValueError("n must be non-negative")

    if n == 0:
        return 1

    return n * factorial(n - 1)

print(factorial(5))
```

Output:

```text
120
```

Each recursive call reduces `n` until the base case is reached.

Time complexity: \(O(n)\).

Auxiliary call-stack space: \(O(n)\).

### Common recursion mistake

```python
def countdown(n):
    print(n)
    return countdown(n - 1)
```

This function has no base case and eventually raises `RecursionError`.

Always ensure recursive calls move toward a valid stopping condition.

### When to use recursion

Recursion can be natural for trees, divide-and-conquer algorithms, and nested structures. For simple repetition, a loop may be easier to understand and may use less call-stack memory.

---

## 11. Docstrings

A docstring explains what a function does, its parameters, and its return value.

```python
def calculate_total(price, quantity):
    """Return the total cost for the given price and quantity."""
    return price * quantity
```

You can access the docstring through:

```python
print(calculate_total.__doc__)
```

Good docstrings are especially useful for reusable functions in shared codebases and data pipelines.

A docstring should explain purpose and relevant behavior, not merely repeat the function's name.

---

## 12. Type Hints

Type hints document the expected types of function arguments and return values.

```python
def add(a: int, b: int) -> int:
    return a + b
```

For a function that returns a string:

```python
def format_name(name: str) -> str:
    return name.strip().title()
```

Type hints improve readability and support static analysis tools.

**Important:** Python generally does not enforce type hints at runtime by itself. Passing a value of an unexpected type may still execute, fail, or produce a different result depending on the function's implementation.

### Optional values

In modern Python, a parameter that can be either a string or `None` can be annotated as:

```python
def greet(name: str | None = None) -> str:
    if name is None:
        return "Hello!"
    return f"Hello, {name}!"
```

The `str | None` syntax requires Python 3.10 or later.

---

## 13. Functions and Mutable Objects

Python passes object references to functions. This is often described as **object sharing** or **call by sharing**.

If a function mutates a mutable object passed to it, the caller can observe that mutation.

```python
def add_item(items):
    items.append(4)

numbers = [1, 2, 3]

add_item(numbers)

print(numbers)
```

Output:

```text
[1, 2, 3, 4]
```

The function mutates the same list object.

However, rebinding a parameter to a different object does not reassign the caller's variable.

```python
def replace_list(items):
    items = [100, 200]

numbers = [1, 2, 3]

replace_list(numbers)

print(numbers)
```

Output:

```text
[1, 2, 3]
```

The assignment changes only the local parameter binding.

**Interview takeaway:** Mutating an object and rebinding a variable are different operations.

---

## 14. Functions as Building Blocks for Data Engineering

Functions make ETL and data-transformation code easier to test and reuse.

### Example: Transform a record

```python
def transform_order(order):
    return {
        "order_id": int(order["order_id"]),
        "customer_id": int(order["customer_id"]),
        "amount": float(order["amount"])
    }

raw_order = {
    "order_id": "101",
    "customer_id": "501",
    "amount": "250.75"
}

clean_order = transform_order(raw_order)

print(clean_order)
```

Output:

```text
{'order_id': 101, 'customer_id': 501, 'amount': 250.75}
```

The function converts raw string values into the required types.

### Example: Separate pipeline steps

```python
def extract(records):
    return records

def transform(records):
    return [
        {
            "id": int(record["id"]),
            "amount": float(record["amount"])
        }
        for record in records
    ]

def load(records):
    # Replace with a real database or storage write.
    return len(records)

raw_records = [
    {"id": "1", "amount": "100.5"},
    {"id": "2", "amount": "250.0"}
]

records = extract(raw_records)
clean_records = transform(records)
loaded_count = load(clean_records)

print(loaded_count)
```

Output:

```text
2
```

This is a simplified example; the `load()` function only counts records.

Separating extraction, transformation, and loading makes it easier to test each stage and identify where a failure occurred.

---

## 15. Time and Space Complexity of Functions

A function does not automatically have a specific time complexity. Its complexity depends on the operations it performs.

| Function operation | Typical time complexity |
|---|---:|
| Add two numbers | \(O(1)\) |
| Access a list element by index | \(O(1)\) |
| Traverse a list of \(n\) elements | \(O(n)\) |
| Sort \(n\) elements using Python's built-in sort | \(O(n \log n)\) |
| Recursive factorial | \(O(n)\) |
| Copy a list of \(n\) elements | \(O(n)\) |

Example:

```python
def find_max(numbers):
    largest = numbers[0]

    for number in numbers[1:]:
        if number > largest:
            largest = number

    return largest
```

For a non-empty list, the function runs in \(O(n)\) time.

The slice `numbers[1:]` creates a new list, requiring \(O(n)\) additional space. A version using indexes or iteration without slicing can use \(O(1)\) auxiliary space.

Always consider both the function's algorithm and the cost of operations performed inside it.

---

## 16. Common Mistakes

### Mistake 1: Forgetting `return`

```python
def add(a, b):
    print(a + b)
```

This prints the result but returns `None`.

### Mistake 2: Calling a function incorrectly

```python
def greet(name):
    return f"Hello, {name}"

# Missing required argument:
# greet()
```

A required parameter must be supplied unless it has a default.

### Mistake 3: Using a mutable default argument

```python
# Avoid this pattern when a fresh list is intended:
def collect(value, result=[]):
    result.append(value)
    return result
```

Use `None` and create the list inside the function instead.

### Mistake 4: Confusing function and function call

```python
def square(n):
    return n * n

operation = square      # Stores a reference to the function
result = square(4)      # Calls the function
```

### Mistake 5: Reassigning a global variable without `global`

```python
count = 0

def increment():
    # Without global, this assignment makes count local,
    # and count += 1 raises UnboundLocalError.
    global count
    count += 1
```

Prefer passing state explicitly when possible.

### Mistake 6: Forgetting the recursion base case

Without a stopping condition, recursive calls may continue until `RecursionError`.

### Mistake 7: Making one function responsible for too much

Functions are easier to maintain when each has a clear responsibility.

### Mistake 8: Returning inconsistent types

A function that sometimes returns a number and sometimes returns a string may be harder to use safely. Define and document a consistent return contract where practical.

---

## 17. Interview Questions

### Conceptual questions

1. What is a function, and why is it useful?
2. What is the difference between a parameter and an argument?
3. What is the difference between `print()` and `return`?
4. What happens if a function has no return statement?
5. What are positional and keyword arguments?
6. What are default parameters?
7. Why are mutable default arguments dangerous?
8. What are `*args` and `**kwargs`?
9. What is the difference between local and global scope?
10. What is the LEGB rule?
11. What is the difference between `global` and `nonlocal`?
12. What is a lambda function?
13. What is a higher-order function?
14. What is a closure?
15. What is recursion, and why does it need a base case?
16. What is a docstring?
17. What are type hints, and are they enforced at runtime?
18. What is the difference between mutating a list and rebinding a parameter?
19. How do you return multiple values from a function?
20. How can functions improve ETL pipeline design?

### Output-based questions

**Question 1**

```python
def test():
    print("Hello")

result = test()
print(result)
```

Answer:

```text
Hello
None
```

**Question 2**

```python
def add(a, b=5):
    return a + b

print(add(10))
print(add(10, 20))
```

Answer:

```text
15
30
```

**Question 3**

```python
def outer():
    x = 10

    def inner():
        return x + 5

    return inner()

print(outer())
```

Answer:

```text
15
```

The inner function can access the enclosing function's variable.

**Question 4**

```python
def calculate(*args):
    return sum(args)

print(calculate(1, 2, 3, 4))
```

Answer:

```text
10
```

**Question 5**

```python
def update(items):
    items.append(4)

numbers = [1, 2, 3]
update(numbers)

print(numbers)
```

Answer:

```text
[1, 2, 3, 4]
```

The function mutates the list object shared with the caller.

---

## 18. Practice Problems

Try solving these without looking at solutions first.

### Easy

1. Write a function to add two numbers.
2. Write a function that checks whether a number is even.
3. Write a function that returns the maximum of three numbers.
4. Write a function to calculate the factorial of a number.
5. Write a function that counts vowels in a string.
6. Write a function that returns the sum of a list.
7. Write a function that checks whether a string is a palindrome.
8. Write a function that converts Celsius to Fahrenheit.

### Medium

1. Write a function that returns both the minimum and maximum of a list.
2. Write a function that removes duplicates while preserving order.
3. Write a function that counts the frequency of each element.
4. Write a recursive function to calculate the sum of the first \(n\) positive integers.
5. Write a function that accepts any number of numeric arguments.
6. Write a function that groups records by a chosen key.
7. Write a function that validates a transaction record and raises appropriate exceptions.
8. Write a function that transforms a list of raw dictionaries into cleaned records.

### Data engineering practice

1. Create a function to validate required fields in a record.
2. Create a function that converts raw CSV-style values into appropriate data types.
3. Write separate functions for extracting, transforming, and loading data.
4. Write a reusable function that filters invalid records.
5. Write a function that aggregates transaction amounts by customer ID.
6. Write a function that retries a transient operation with a maximum attempt count.
7. Add type hints and docstrings to a data transformation function.
8. Write unit tests for a function that handles missing and malformed fields.

For each solution, explain the input, output, edge cases, time complexity, and space complexity.

---

## 19. Quick Revision

- `def` defines a function.
- Parameters appear in the function definition; arguments are passed during a call.
- `return` sends a result back and ends the current function call.
- A function without a return value returns `None`.
- Default parameters provide values when arguments are omitted.
- Use `None` for mutable defaults when each call should create a fresh object.
- `*args` collects extra positional arguments into a tuple.
- `**kwargs` collects extra keyword arguments into a dictionary.
- Local, enclosing, global, and built-in scopes follow the LEGB lookup rule.
- `global` reassigns a module-level variable; `nonlocal` reassigns an enclosing-function variable.
- Lambda functions contain a single expression.
- Higher-order functions accept or return functions.
- Recursive functions need a base case.
- Type hints document expected types but generally do not enforce them at runtime.
- Mutating a mutable argument can affect the caller's object.
- Well-designed functions make data pipelines easier to test, debug, and maintain.

**Recommended learning order:** First master parameters, arguments, and `return`. Then learn scope, default parameters, `*args`, and `**kwargs`. Finally, practise recursion, higher-order functions, and reusable ETL transformations.