# Exception Handling in Python

## 1. Overview

An **exception** is an event that occurs during program execution and interrupts the normal flow of instructions.

Exception handling allows a program to detect errors and respond to them instead of terminating unexpectedly.

```python
number = 10
divisor = 0

result = number / divisor
print(result)
```

Output:

```text
ZeroDivisionError: division by zero
```

Python raises a `ZeroDivisionError` because division by zero is not allowed.

We can handle this exception using `try` and `except`.

```python
number = 10
divisor = 0

try:
    result = number / divisor
    print(result)
except ZeroDivisionError:
    print("Cannot divide by zero")
```

Output:

```text
Cannot divide by zero
```

**Why exception handling matters in data engineering:**

- Handling missing or malformed input data.
- Managing file-reading failures.
- Handling API and database errors.
- Logging failed data-processing operations.
- Ensuring resources are released correctly.
- Preventing one recoverable failure from crashing an entire workflow.

---

## 2. Errors vs Exceptions

Errors can occur for different reasons. Two important categories are syntax errors and runtime exceptions.

### 2.1 Syntax errors

A syntax error occurs when Python code violates the language's syntax rules.

```python
# Invalid syntax:
# if 10 > 5
#     print("Valid")
```

The missing colon causes a `SyntaxError`.

Python generally detects syntax errors before executing the affected code.

### 2.2 Runtime exceptions

Runtime exceptions occur while code is executing.

```python
numbers = [10, 20, 30]

print(numbers[5])
```

Output:

```text
IndexError: list index out of range
```

The code is syntactically valid, but the requested index does not exist.

### Common exceptions

| Exception | Typical cause |
|---|---|
| `ZeroDivisionError` | Dividing a number by zero |
| `ValueError` | Correct type, but an invalid value |
| `TypeError` | An operation is used with an inappropriate type |
| `IndexError` | Accessing an invalid sequence index |
| `KeyError` | Accessing a missing dictionary key |
| `NameError` | Using an undefined variable name |
| `AttributeError` | Accessing an attribute that an object does not have |
| `FileNotFoundError` | Opening a file that does not exist |
| `ImportError` | An import cannot be completed |
| `ModuleNotFoundError` | A module cannot be found |
| `ZeroDivisionError` | Performing division by zero |
| `OverflowError` | A numeric operation exceeds a supported range |
| `OSError` | An operating-system operation fails |
| `RuntimeError` | A runtime problem that does not fit a more specific exception |
| `StopIteration` | An iterator has no more items; commonly handled by iteration machinery |

---

## 3. The `try` and `except` Blocks

Use `try` for code that might raise an exception and `except` for handling a specific failure.

```python
try:
    number = int("abc")
except ValueError:
    print("Invalid integer")
```

Output:

```text
Invalid integer
```

The conversion fails because `"abc"` cannot be converted into an integer.

### Execution flow

1. Python executes the `try` block.
2. If no exception occurs, the `except` block is skipped.
3. If a matching exception occurs, the remaining statements in the `try` block are skipped.
4. Python executes the matching `except` block.
5. Execution continues after the exception-handling structure.

Example without an exception:

```python
try:
    number = int("123")
    print(number)
except ValueError:
    print("Invalid input")
```

Output:

```text
123
```

### Important principle

Keep the `try` block as small as practical.

Avoid placing unrelated operations inside it because an exception from any of those operations could be mistaken for the failure you intended to handle.

---

## 4. Handling Multiple Exceptions

A program may encounter different types of exceptions.

### 4.1 Separate `except` blocks

```python
try:
    number = int(input("Enter a number: "))
    result = 100 / number
    print(result)

except ValueError:
    print("Enter a valid integer")

except ZeroDivisionError:
    print("Number cannot be zero")
```

Each exception type has its own handler.

### 4.2 Handling multiple exception types together

If multiple exceptions require the same response, use a tuple.

```python
try:
    result = 100 / int("0")
except (ValueError, ZeroDivisionError):
    print("Invalid calculation")
```

### 4.3 Exception order matters

Catch specific exceptions before their broader parent classes.

```python
try:
    number = int("abc")

except ValueError:
    print("Invalid value")

except Exception:
    print("Unexpected error")
```

`ValueError` is handled by the first block.

If `Exception` appeared first, it would catch the `ValueError` as well, making the more specific handler unreachable.

**Interview point:** `Exception` is a broad class for most ordinary application exceptions. Catch it only when a broad fallback is genuinely needed.

---

## 5. The `else` Block

The `else` block runs only when the `try` block completes without raising an exception.

```python
try:
    number = int("25")

except ValueError:
    print("Invalid input")

else:
    print("Conversion successful:", number)
```

Output:

```text
Conversion successful: 25
```

If the conversion raises `ValueError`, the `except` block executes and the `else` block is skipped.

### Why use `else`?

It separates successful operations from exception handling.

```python
try:
    file = open("data.txt", "r")

except FileNotFoundError:
    print("File not found")

else:
    print("File opened successfully")
    file.close()
```

For real file handling, a context manager is generally safer, as discussed later.

**Remember:** The `else` block is not a replacement for `except`. It is for code that should execute only after the `try` block succeeds.

---

## 6. The `finally` Block

The `finally` block runs when execution leaves the `try` structure, whether an exception occurs or not, including when an exception is propagated.

```python
try:
    number = 10 / 0

except ZeroDivisionError:
    print("Division failed")

finally:
    print("Execution completed")
```

Output:

```text
Division failed
Execution completed
```

Without an exception:

```python
try:
    print("Processing data")

finally:
    print("Cleanup")
```

Output:

```text
Processing data
Cleanup
```

### Common uses

- Closing files.
- Releasing database connections.
- Releasing locks.
- Cleaning up temporary resources.

However, `finally` is not an absolute guarantee under events such as forced process termination or certain interpreter-level failures.

### `try`, `except`, `else`, and `finally`

```python
try:
    number = int("10")
    result = 100 / number

except ValueError:
    print("Invalid input")

except ZeroDivisionError:
    print("Cannot divide by zero")

else:
    print("Result:", result)

finally:
    print("Finished")
```

Output:

```text
Result: 10.0
Finished
```

| Block | When it runs |
|---|---|
| `try` | First; contains the operation being attempted |
| `except` | When a matching exception occurs |
| `else` | When `try` finishes without an exception |
| `finally` | When leaving the structure, including exceptional paths |

---

## 7. Raising Exceptions with `raise`

Use `raise` to signal that an operation cannot continue under the current conditions.

```python
age = -5

if age < 0:
    raise ValueError("Age cannot be negative")
```

Output:

```text
ValueError: Age cannot be negative
```

Raising an exception is useful for validating input and enforcing business rules.

### 7.1 Validate function arguments

```python
def calculate_discount(price, discount):
    if price < 0:
        raise ValueError("Price cannot be negative")

    if not 0 <= discount <= 100:
        raise ValueError("Discount must be between 0 and 100")

    return price * (1 - discount / 100)
```

Example:

```python
print(calculate_discount(1000, 10))
```

Output:

```text
900.0
```

Invalid inputs raise a clear exception rather than silently producing an incorrect result.

### 7.2 Re-raise an exception

Inside an `except` block, bare `raise` re-raises the current exception.

```python
try:
    number = int("abc")

except ValueError:
    print("Logging conversion failure")
    raise
```

The message is printed, and the original `ValueError` continues propagating.

Use this when you need to log or perform a small action before allowing the caller to handle the failure.

---

## 8. Custom Exceptions

You can define your own exception class when an application needs to represent a specific kind of failure.

Custom exceptions generally inherit from `Exception`.

```python
class InvalidAgeError(Exception):
    pass
```

Use it:

```python
def register_user(age):
    if age < 18:
        raise InvalidAgeError("User must be at least 18")

    return "Registration successful"
```

Handle it:

```python
try:
    print(register_user(16))

except InvalidAgeError as error:
    print(error)
```

Output:

```text
User must be at least 18
```

### Why create custom exceptions?

They make failures easier to identify and handle.

Examples in a data pipeline might include:

- `InvalidRecordError`
- `SchemaValidationError`
- `DataQualityError`
- `PipelineConfigurationError`

Use custom exceptions when they clarify the application. Avoid creating a separate exception class for every minor variation of the same error.

---

## 9. Accessing Exception Details with `as`

Use `as` to bind an exception object to a variable.

```python
try:
    number = int("hello")

except ValueError as error:
    print("Error:", error)
```

Output:

```text
Error: invalid literal for int() with base 10: 'hello'
```

This is useful for logging diagnostic details.

Example:

```python
try:
    result = 10 / 0

except ZeroDivisionError as error:
    print(type(error).__name__)
    print(error)
```

Output:

```text
ZeroDivisionError
division by zero
```

Avoid exposing sensitive internal details to end users. Diagnostic information should be directed to appropriate logs when needed.

---

## 10. Exception Hierarchy

Python exceptions are organized in an inheritance hierarchy.

A simplified view:

```text
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
└── Exception
    ├── ArithmeticError
    │   ├── ZeroDivisionError
    │   ├── OverflowError
    │   └── FloatingPointError
    ├── LookupError
    │   ├── IndexError
    │   └── KeyError
    ├── ValueError
    ├── TypeError
    ├── OSError
    │   ├── FileNotFoundError
    │   └── PermissionError
    └── RuntimeError
```

This is a simplified representation, not the complete hierarchy.

### Why does the hierarchy matter?

When an exception class inherits from another class, a handler for the parent can also catch instances of the child class.

For example:

```python
try:
    numbers = [1, 2, 3]
    print(numbers[10])

except LookupError:
    print("Lookup failed")
```

Output:

```text
Lookup failed
```

`IndexError` inherits from `LookupError`, so the handler catches it.

### Why avoid catching `BaseException`?

`BaseException` also includes control-flow exceptions such as `KeyboardInterrupt` and `SystemExit`.

Catching it broadly can prevent normal interruption or shutdown behavior. Most application code should catch specific exceptions or, when justified, `Exception`.

---

## 11. Exception Chaining

Sometimes an operation raises a low-level exception, but the application needs to report a more meaningful high-level exception.

Use `raise ... from ...` to preserve the relationship between the two exceptions.

```python
def load_age(value):
    try:
        return int(value)

    except ValueError as error:
        raise ValueError("Age must be a valid integer") from error
```

Calling `load_age("abc")` raises a new `ValueError` with the original conversion error recorded as its cause.

### Suppressing exception context

You can use `from None` to suppress the display of the automatically chained context.

```python
try:
    int("abc")

except ValueError:
    raise RuntimeError("Unable to process input") from None
```

Use this only when suppressing the displayed context is intentional. Preserving the original cause is generally helpful for debugging.

---

## 12. File Handling and Exceptions

File operations can fail because of missing files, permission restrictions, invalid paths, or other operating-system problems.

### 12.1 Basic exception handling

```python
try:
    file = open("data.txt", "r")
    content = file.read()
    print(content)

except FileNotFoundError:
    print("The file does not exist")

finally:
    if "file" in locals():
        file.close()
```

This demonstrates cleanup, but manual resource management can become cumbersome.

### 12.2 Preferred approach: context managers

Use `with open(...)` to ensure that the file is closed automatically when the block exits.

```python
try:
    with open("data.txt", "r") as file:
        content = file.read()
        print(content)

except FileNotFoundError:
    print("The file does not exist")

except PermissionError:
    print("Permission denied")
```

The context manager closes the file even if an exception occurs while reading it.

### Data engineering example

When reading input files, handle expected file-related exceptions and log enough information to investigate failures.

```python
import logging

logging.basicConfig(level=logging.INFO)

try:
    with open("orders.csv", "r") as file:
        content = file.read()

except FileNotFoundError:
    logging.error("Required orders file was not found")
    raise

except PermissionError:
    logging.error("Insufficient permission to read orders file")
    raise
```

Re-raising the exception allows an orchestrator or calling function to recognize that the operation failed.

**Important:** Do not catch an exception, log it, and then silently continue if the pipeline cannot produce correct output.

---

## 13. Exception Handling in Data Engineering

Data pipelines interact with files, databases, APIs, and distributed processing systems. Exceptions should be handled according to whether the failure is recoverable.

### Example: Validate a record

```python
def process_record(record):
    try:
        customer_id = int(record["customer_id"])
        amount = float(record["amount"])

    except KeyError as error:
        raise ValueError(
            f"Required field is missing: {error}"
        ) from error

    except (TypeError, ValueError) as error:
        raise ValueError("Record contains invalid data") from error

    if amount < 0:
        raise ValueError("Amount cannot be negative")

    return {
        "customer_id": customer_id,
        "amount": amount
    }
```

This example demonstrates:
- Detecting missing fields.
- Converting input values.
- Validating business rules.
- Raising clear exceptions for invalid records.

In a production pipeline, you might route invalid records to a quarantine table or dead-letter location instead of failing the entire batch. The correct choice depends on the data quality requirements and the pipeline's failure policy.

### Example: Retry transient failures

Some errors are temporary, such as a network timeout or rate limit. A retry may help, but invalid data and permission errors generally require a different response.

A simple illustrative pattern:

```python
import time

def process_with_retry(operation, attempts=3):
    for attempt in range(attempts):
        try:
            return operation()

        except TimeoutError:
            if attempt == attempts - 1:
                raise

            time.sleep(2 ** attempt)
```

This retries a `TimeoutError` up to three times, waiting 1 second and then 2 seconds before subsequent attempts.

**Production considerations:**
- Retry only errors that are likely to be transient.
- Set sensible retry limits and timeouts.
- Use exponential backoff where appropriate.
- Consider jitter to avoid many clients retrying simultaneously.
- Ensure the operation is safe to retry, especially if it writes data or triggers side effects.
- Prefer built-in retry mechanisms from the relevant database, API client, or orchestration framework when available.

---

## 14. Logging Exceptions

Logging records information about what happened during program execution.

```python
import logging

logging.basicConfig(level=logging.INFO)

try:
    result = 10 / 0

except ZeroDivisionError:
    logging.exception("Calculation failed")
    raise
```

`logging.exception()` records a message along with exception traceback information. It is intended to be called inside an exception handler.

### Common logging levels

| Level | Typical purpose |
|---|---|
| `DEBUG` | Detailed diagnostic information |
| `INFO` | Normal progress and milestones |
| `WARNING` | Unexpected situation that may be recoverable |
| `ERROR` | An operation failed |
| `CRITICAL` | A serious failure that may affect the application significantly |

### Best practices

- Include useful context, such as the job name or record identifier.
- Avoid logging passwords, access tokens, or sensitive personal information.
- Include traceback information when it helps diagnose the failure.
- Do not log the same exception repeatedly at every layer without a clear reason.
- Preserve failure signals so monitoring systems can detect unsuccessful jobs.

---

## 15. Common Exception Handling Mistakes

### Mistake 1: Catching every exception without a reason

```python
try:
    result = 10 / 0

except Exception:
    print("Something went wrong")
```

This catches the exception but hides its details and may make debugging difficult.

Prefer handling known exceptions explicitly. A broad handler is appropriate only when the program has a meaningful fallback or logging strategy.

### Mistake 2: Using `except` without an exception type

```python
try:
    result = 10 / 0

except:
    print("Failed")
```

A bare `except` can catch exceptions that application code usually should not intercept, including `KeyboardInterrupt` and `SystemExit`.

Prefer `except Exception as error` for a deliberate broad fallback, or catch a specific exception.

### Mistake 3: Silencing an exception

```python
try:
    save_data()

except Exception:
    pass
```

This hides the failure. The program may appear successful even though data was not saved.

Only suppress exceptions when the failure is genuinely safe to ignore and the behavior is documented.

### Mistake 4: Putting too much code inside `try`

If many unrelated operations are in the same block, an exception from the wrong operation may be caught and misinterpreted.

Keep the protected section focused on the operation that may fail.

### Mistake 5: Raising a generic exception for everything

Use meaningful exception types. For invalid input, `ValueError` may be appropriate; for incorrect argument types, `TypeError` may be better.

Custom exceptions are useful when application-specific failures need to be distinguished.

### Mistake 6: Returning success after a failure

Do not catch a critical exception and return a success-shaped result unless that fallback is intentional and clearly represented.

### Mistake 7: Using exceptions for normal control flow

Do not use exceptions as a substitute for ordinary conditions when a simple check is clearer.

For example, use `if key in data` when checking dictionary membership is routine and expected.

---

## 16. Time and Performance Considerations

Exception handling has little impact when the protected operation succeeds and no exception is raised, although there is some overhead.

Actually raising and handling an exception is relatively expensive compared with ordinary control flow.

Avoid using exceptions repeatedly for routine operations when a direct condition is clear.

Example:

```python
data = {"name": "Aman"}

if "salary" in data:
    salary = data["salary"]
else:
    salary = 0
```

This may be clearer than relying on `KeyError` for expected missing-key behavior.

However, do not duplicate complex checks that can introduce race conditions or inconsistencies. Choose the simplest correct approach for the situation.

---

## 17. Interview Questions

### Conceptual questions

1. What is an exception in Python?
2. What is the difference between a syntax error and a runtime exception?
3. What is the purpose of `try` and `except`?
4. What is the difference between `else` and `finally`?
5. What happens when an exception is not handled?
6. What is the purpose of `raise`?
7. How do you re-raise an exception?
8. Why should specific exceptions be caught before general ones?
9. What is the difference between `Exception` and `BaseException`?
10. How do you define a custom exception?
11. What is exception chaining?
12. Why should broad exceptions not be silently ignored?
13. Why are context managers useful for file handling?
14. How should a data pipeline handle invalid records?
15. When should a pipeline retry a failed operation?
16. What does `logging.exception()` do?
17. What is the difference between `ValueError` and `TypeError`?
18. What happens if an exception occurs inside a `finally` block?

### Output-based questions

**Question 1**

```python
try:
    print(10 / 2)
except ZeroDivisionError:
    print("Error")
else:
    print("Success")
finally:
    print("Done")
```

Answer:

```text
5.0
Success
Done
```

The operation succeeds, so `else` and `finally` execute.

**Question 2**

```python
try:
    int("abc")
except ValueError:
    print("Invalid")
finally:
    print("Finished")
```

Answer:

```text
Invalid
Finished
```

**Question 3**

```python
try:
    print("Start")
    10 / 0
    print("End")
except ZeroDivisionError:
    print("Error")
```

Answer:

```text
Start
Error
```

The exception prevents `print("End")` from executing.

**Question 4**

```python
try:
    raise ValueError("Invalid value")
except Exception:
    print("Handled")
```

Answer:

```text
Handled
```

`ValueError` inherits from `Exception`.

**Question 5**

```python
def check():
    try:
        return "try"
    finally:
        print("finally")

print(check())
```

Answer:

```text
finally
try
```

The `finally` block runs before the function returns its value.

Avoid returning a different value from `finally`, because it can override a pending return or suppress an exception and make the behavior confusing.

---

## 18. Practice Problems

Try to solve these before looking at examples or solutions.

### Easy

1. Convert a string to an integer and handle invalid input.
2. Divide two numbers while handling division by zero.
3. Access a dictionary key safely.
4. Open a file and handle a missing-file exception.
5. Handle invalid list indexing.
6. Create a custom exception for a negative amount.
7. Use `finally` to print a cleanup message.
8. Log an exception with its traceback.

### Medium

1. Write a function that validates a user's age and raises a suitable exception.
2. Process a list of numeric strings while handling invalid entries.
3. Create a custom exception for invalid transaction records.
4. Validate required fields in a dictionary.
5. Write a retry function for a simulated timeout.
6. Wrap a low-level exception in a meaningful application exception.
7. Process multiple records and collect invalid records separately.
8. Read a file using a context manager and handle expected failures.

### Data engineering practice

1. Validate that each transaction has a customer ID and amount.
2. Distinguish missing fields from invalid numeric values.
3. Handle a failed API request without retrying permanent errors.
4. Design a function that logs an error and re-raises it.
5. Create a simple batch processor that records successful and failed rows.
6. Decide which errors should fail an entire ETL job versus quarantine individual records.
7. Write a database-operation wrapper that preserves the original exception when adding context.
8. Explain how you would avoid duplicate writes when retrying a failed data pipeline step.

For each solution, explain what exception is raised, where it is handled, and whether processing should continue or fail.

---

## 19. Quick Revision

- An exception interrupts the normal execution flow.
- `try` contains code that may fail.
- `except` handles matching exceptions.
- `else` runs when the `try` block succeeds without an exception.
- `finally` runs when leaving the structure, including exceptional paths.
- `raise` explicitly raises an exception.
- Bare `raise` re-raises the current exception from an exception handler.
- Custom exceptions should normally inherit from `Exception`.
- Catch specific exceptions before broader parent classes.
- Avoid silently swallowing errors.
- Use context managers such as `with open(...)` for safe resource management.
- Use logging to record useful diagnostic details.
- Retry transient failures only when retrying is safe.
- In data engineering, decide deliberately whether a failure should stop a pipeline, be retried, or be quarantined.

**Recommended learning order:** First master `try-except`, then `else` and `finally`. Next practice `raise` and custom exceptions. Finally, apply exception handling to files, validation, logging, and data-pipeline scenarios.