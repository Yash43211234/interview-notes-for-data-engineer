# Decorators in Python

## 1. Overview

A decorator is a function that takes another function or callable, adds or changes its behavior, and returns a callable.

Decorators let you add functionality without modifying the original function's code.

Common use cases include:
- Logging function calls.
- Measuring execution time.
- Checking permissions.
- Caching results.
- Retrying failed operations.
- Validating inputs.

For Data Engineering, decorators can help log pipeline operations, measure transformation execution time, and implement retry or monitoring behavior.

## 2. Understanding the Basics

Before learning decorators, understand three Python concepts.

### Functions are objects

A function can be assigned to a variable.

```python
def greet():
    print("Hello!")

say_hello = greet

say_hello()
# Hello!
```

Here, `say_hello` refers to the same function as `greet`. The parentheses in `greet()` would call the function; without parentheses, `greet` refers to the function object.

### Functions can be passed as arguments

```python
def greet():
    print("Hello!")

def execute(func):
    func()

execute(greet)
# Hello!
```

The `execute()` function receives another function as an argument and calls it.

### Functions can return functions

```python
def outer():
    def inner():
        print("Inside inner")

    return inner

result = outer()
result()
# Inside inner
```

The `outer()` function returns the `inner` function without calling it. These three concepts form the foundation of decorators.

## 3. Creating Your First Decorator

Suppose you want to print a message before and after a function executes.

Without a decorator, you might repeat the same logic in multiple functions.

```python
def process_data():
    print("Processing data")

def run_pipeline():
    print("Starting pipeline")
    process_data()
    print("Pipeline completed")
```

A decorator lets you reuse that additional behavior.

```python
def my_decorator(func):
    def wrapper():
        print("Before function")
        func()
        print("After function")

    return wrapper

def greet():
    print("Hello!")

greet = my_decorator(greet)

greet()
```

Output:

```text
Before function
Hello!
After function
```

The decorator replaces the name `greet` with the returned `wrapper` function. Calling `greet()` now executes the wrapper, which calls the original function between the two messages.

## 4. The `@decorator` Syntax

Python provides convenient syntax for applying decorators.

Instead of writing:

```python
def greet():
    print("Hello!")

greet = my_decorator(greet)
```

You can write:

```python
def my_decorator(func):
    def wrapper():
        print("Before function")
        func()
        print("After function")

    return wrapper

@my_decorator
def greet():
    print("Hello!")

greet()
```

Output:

```text
Before function
Hello!
After function
```

The statement:

```python
@my_decorator
def greet():
    print("Hello!")
```

is essentially equivalent to:

```python
def greet():
    print("Hello!")

greet = my_decorator(greet)
```

**Interview takeaway:** `@decorator` is syntactic sugar for passing a function to a decorator and assigning the returned callable to the original function's name.

## 5. Supporting Function Arguments

The previous decorator works only for functions that take no arguments.

Consider:

```python
def greet(name):
    print(f"Hello, {name}!")
```

If you apply the earlier decorator, `greet("Amit")` fails because `wrapper()` does not accept arguments.

The standard solution is to use `*args` and `**kwargs`.

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before function")

        result = func(*args, **kwargs)

        print("After function")
        return result

    return wrapper

@my_decorator
def greet(name):
    print(f"Hello, {name}!")

greet("Amit")
```

Output:

```text
Before function
Hello, Amit!
After function
```

Here:

- `*args` collects positional arguments into a tuple.
- `**kwargs` collects keyword arguments into a dictionary.
- `func(*args, **kwargs)` forwards both kinds of arguments.
- `return result` preserves the wrapped function's return value.

### Why is `return result` important?

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return result

    return wrapper

@my_decorator
def add(a, b):
    return a + b

print(add(10, 20))
# 30
```

Without `return result`, the decorated function would return `None`, even though the original `add()` function returns a number.

## 6. Preserving Function Metadata with `functools.wraps`

A decorator can change the visible name and documentation of a function.

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper

@my_decorator
def add(a, b):
    """Add two numbers."""
    return a + b

print(add.__name__)
# wrapper
```

This can make debugging and introspection confusing.

Use `functools.wraps` to preserve the original function's metadata.

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper

@my_decorator
def add(a, b):
    """Add two numbers."""
    return a + b

print(add.__name__)  # add
print(add.__doc__)   # Add two numbers.
```

`@wraps(func)` copies important metadata from the original function to the wrapper, including its name and documentation.

**Best practice:** Use `@wraps(func)` in custom decorators that define a wrapper function.

## 7. Practical Decorator Examples

### Example 1: Logging function execution

Logging can help you understand which functions are running in a pipeline.

```python
from functools import wraps

def log_execution(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Starting: {func.__name__}")

        result = func(*args, **kwargs)

        print(f"Completed: {func.__name__}")
        return result

    return wrapper

@log_execution
def transform_data():
    print("Transforming records")

transform_data()
```

Output:

```text
Starting: transform_data
Transforming records
Completed: transform_data
```

For production systems, use Python's `logging` module rather than relying on `print()` statements.

### Example 2: Measuring execution time

```python
import time
from functools import wraps

def measure_time(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()

        try:
            return func(*args, **kwargs)
        finally:
            elapsed = time.perf_counter() - start
            print(f"{func.__name__} took {elapsed:.4f} seconds")

    return wrapper

@measure_time
def process_data():
    time.sleep(1)
    return "Completed"

result = process_data()
print(result)
```

The `finally` block runs whether the wrapped function succeeds or raises an exception, so execution time is still reported when an error occurs.

`time.perf_counter()` is suitable for measuring elapsed time. The decorator does not make the function faster; it measures how long it takes to run.

### Example 3: Caching results

A cache avoids repeating expensive calculations for the same arguments.

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def square(n):
    print("Calculating...")
    return n * n

print(square(5))
print(square(5))
```

Output:

```text
Calculating...
25
25
```

The second call reuses the cached result for the same argument.

Important points:
- `lru_cache` is a built-in decorator from `functools`.
- Arguments must be hashable.
- Cached results can become stale if the underlying data changes.
- Caching is most suitable when repeated calls with the same arguments should return the same result.

### Example 4: Retrying a failed operation

Retry decorators can be useful for transient failures in external API calls or database connections.

```python
import time
from functools import wraps

def retry(max_attempts=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception:
                    if attempt == max_attempts - 1:
                        raise

                    time.sleep(delay)

        return wrapper
    return decorator

@retry(max_attempts=3, delay=1)
def fetch_data():
    print("Attempting request...")
    # Replace with an actual operation.
    return "Data received"

print(fetch_data())
```

This example demonstrates the pattern, but the function succeeds on its first call. A real retry is useful only when the operation can fail transiently.

Production retry logic should:
- Catch specific recoverable exceptions rather than every `Exception`.
- Use a sensible maximum attempt count.
- Consider exponential backoff and jitter.
- Avoid retrying non-idempotent operations unless duplicate effects are handled.
- Log failed attempts.

## 8. Decorators That Accept Arguments

So far, decorators have accepted a function directly:

```python
@my_decorator
def greet():
    ...
```

What if you want to configure the decorator?

```python
@repeat(times=3)
def greet():
    print("Hello")
```

This requires an extra function layer.

```python
from functools import wraps

def repeat(times):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                func(*args, **kwargs)

        return wrapper
    return decorator

@repeat(times=3)
def greet():
    print("Hello")

greet()
```

Output:

```text
Hello
Hello
Hello
```

Understand the three layers:

1. `repeat(times)` receives the decorator configuration.
2. `decorator(func)` receives the function being decorated.
3. `wrapper(*args, **kwargs)` runs whenever the decorated function is called.

A decorator with arguments is therefore a function that returns a decorator.

### Preserving return values in a configurable decorator

The earlier `repeat` decorator intentionally ignores return values because it calls the function multiple times. If a decorator is meant to return a result, decide what that result should represent.

For example, a configurable decorator could execute a function once and add behavior around its result:

```python
from functools import wraps

def add_prefix(prefix):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            result = func(*args, **kwargs)
            return f"{prefix}{result}"

        return wrapper
    return decorator

@add_prefix("Result: ")
def get_value():
    return 100

print(get_value())
# Result: 100
```

## 9. Stacking Multiple Decorators

You can apply more than one decorator to a function.

```python
def first(func):
    def wrapper():
        print("First before")
        func()
        print("First after")
    return wrapper

def second(func):
    def wrapper():
        print("Second before")
        func()
        print("Second after")
    return wrapper

@first
@second
def greet():
    print("Hello")

greet()
```

Output:

```text
First before
Second before
Hello
Second after
First after
```

Decorators are applied from the bottom upward:

```python
@first
@second
def greet():
    ...
```

is equivalent to:

```python
greet = first(second(greet))
```

The outer decorator runs first when the function is called, while the inner decorator surrounds the original function.

## 10. Built-in Python Decorators

Python includes several useful decorators.

### `@staticmethod`

Defines a method that does not automatically receive an instance or class reference.

```python
class MathUtils:
    @staticmethod
    def add(a, b):
        return a + b

print(MathUtils.add(10, 20))
# 30
```

Use it when a method logically belongs to a class but does not need instance or class state.

### `@classmethod`

Receives the class as the first argument, conventionally named `cls`.

```python
class Employee:
    company = "TCS"

    @classmethod
    def get_company(cls):
        return cls.company

print(Employee.get_company())
# TCS
```

A common use case is an alternative constructor.

### `@property`

Lets you access a method like an attribute.

```python
class Employee:
    def __init__(self, salary):
        self._salary = salary

    @property
    def salary(self):
        return self._salary

employee = Employee(90000)

print(employee.salary)
# 90000
```

Instead of calling `employee.salary()`, you access `employee.salary`.

These decorators serve different purposes:
- `staticmethod`: no automatic `self` or `cls`.
- `classmethod`: receives `cls`.
- `property`: exposes a method through attribute access.

## 11. Error Handling Inside Decorators

A decorator should usually preserve the original function's exception behavior unless changing it is an explicit requirement.

```python
from functools import wraps

def log_errors(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except Exception:
            print(f"Error in {func.__name__}")
            raise

    return wrapper

@log_errors
def divide(a, b):
    return a / b

# divide(10, 0)  # Logs the error and re-raises ZeroDivisionError
```

The bare `raise` re-raises the active exception without replacing it.

Avoid silently swallowing exceptions:

```python
def bad_decorator(func):
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except Exception:
            return None  # Can hide important failures
    return wrapper
```

Returning `None` for every failure may make a pipeline appear successful when it actually failed.

## 12. Decorators in Data Engineering

Decorators are useful for adding cross-cutting behavior to data-processing functions.

Example: a simple decorator that logs the start and end of a transformation.

```python
import logging
from functools import wraps

logging.basicConfig(level=logging.INFO)

def log_transform(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        logging.info("Starting %s", func.__name__)

        result = func(*args, **kwargs)

        logging.info("Finished %s", func.__name__)
        return result

    return wrapper

@log_transform
def transform_orders(orders):
    return [
        order for order in orders
        if order["amount"] > 0
    ]

orders = [
    {"order_id": 1, "amount": 100},
    {"order_id": 2, "amount": -10}
]

print(transform_orders(orders))
```

The decorator adds logging without changing the transformation logic.

Possible applications:
- Track the start and completion of ETL functions.
- Measure execution duration.
- Apply consistent validation.
- Cache reusable calculations.
- Implement retries for safe, transient operations.

**Important:** A decorator around a Python function does not automatically optimize distributed PySpark work. Spark execution is lazy, and the actual computation may happen later when an action runs. Measure the correct execution boundary, such as the relevant Spark action, rather than assuming the function's runtime captures all distributed processing.

## 13. Common Mistakes

### Mistake 1: Forgetting to return the wrapper

Incorrect:

```python
def my_decorator(func):
    def wrapper():
        func()
    # Missing: return wrapper
```

The decorator returns `None` implicitly, so the decorated function name becomes bound to `None`.

### Mistake 2: Forgetting to return the original function's result

Incorrect:

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        func(*args, **kwargs)
        # Result is discarded
    return wrapper
```

Correct:

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

### Mistake 3: Not supporting arguments

A wrapper defined as `wrapper()` cannot accept arguments passed to the decorated function. Use `*args` and `**kwargs` when the decorator should support arbitrary call signatures.

### Mistake 4: Forgetting `@wraps`

Without `@wraps(func)`, function metadata such as `__name__` and `__doc__` usually reflects the wrapper instead of the original function.

### Mistake 5: Confusing decorator application with function execution

```python
@my_decorator
def greet():
    print("Hello")
```

The decorator is applied when the function definition is executed, typically during module import. The wrapper's body runs when the decorated function is called.

### Mistake 6: Retrying every exception

Not all failures are temporary. Retrying invalid input, permission errors, or non-idempotent writes can waste resources or create duplicate side effects.

## 14. Interview Practice

### Easy

**Q1. What is a decorator in Python?**

**Q2. What does the `@` syntax do?**

**Q3. What is the purpose of `*args` and `**kwargs` in a wrapper?**

**Q4. Why should a decorator use `functools.wraps`?**

### Medium

**Q5. Predict the output.**

```python
def decorator(func):
    def wrapper():
        print("Before")
        func()
        print("After")
    return wrapper

@decorator
def greet():
    print("Hello")

greet()
```

**Q6. What is wrong with this decorator?**

```python
def decorator(func):
    def wrapper(*args, **kwargs):
        func(*args, **kwargs)
    return wrapper

@decorator
def add(a, b):
    return a + b

print(add(2, 3))
```

**Q7. Explain the order of execution.**

```python
@first
@second
def task():
    print("Running")
```

**Q8. How would you write a decorator that measures function execution time?**

**Q9. What is the difference between a decorator and a decorator that accepts arguments?**

**Q10. Why can retries be dangerous for data pipelines?**

### Answers

1. A callable that wraps another callable to add or change behavior.
2. It applies a decorator to a function or class definition.
3. They let the wrapper accept and forward arbitrary positional and keyword arguments.
4. It preserves metadata such as the original function's name and documentation.
5. `Before`, `Hello`, `After`, each on its own line.
6. The wrapper discards the return value, so `add(2, 3)` returns `None`. Use `return func(*args, **kwargs)`.
7. Decorators are applied bottom-up: `task = first(second(task))`.
8. Record a start time, call the function, calculate elapsed time, and return its result. Use `try/finally` if timing must be reported even when an exception occurs.
9. A basic decorator receives a function; a configurable decorator first receives configuration and returns a decorator.
10. Retrying a non-idempotent operation may repeat writes or other side effects and create duplicate records.

## 15. Quick Revision

- A decorator wraps a function or callable to add behavior.
- Functions are objects and can be passed around and returned.
- `@decorator` is equivalent to `function = decorator(function)`.
- Use `*args` and `**kwargs` to forward arguments.
- Return the wrapped function's result to preserve behavior.
- Use `functools.wraps` to preserve function metadata.
- Decorators with arguments require an additional function layer.
- Stacked decorators are applied from bottom to top.
- Built-in decorators include `@staticmethod`, `@classmethod`, and `@property`.
- Use decorators for reusable logging, timing, caching, validation, and retry logic.
- In production pipelines, handle exceptions carefully and retry only operations where doing so is safe.

**Interview takeaway:** Be able to implement a decorator from scratch, explain how `@` works, preserve arguments and return values, use `@wraps`, and describe a practical Data Engineering use case.