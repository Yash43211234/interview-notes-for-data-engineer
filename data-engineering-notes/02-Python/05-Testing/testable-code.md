# Testable Code in Python

## 1. What Is Testable Code?

**Testable code** is code that can be tested easily and reliably without requiring complicated setup, real external systems, or manual intervention.

Good testable code is usually:
- **Modular:** Each function has a clear responsibility.
- **Predictable:** The same inputs produce the same outputs when external dependencies are unchanged.
- **Independent:** Functions do not unnecessarily depend on global variables or hidden state.
- **Decoupled:** Business logic is separated from databases, APIs, files, and other external systems.
- **Easy to verify:** Expected behavior can be checked with assertions.

Testable code is especially important in data engineering, where pipelines process files, call APIs, transform data, and write results to databases or data lakes.

---

## 2. Example: Difficult-to-Test Code

Consider a function that reads a CSV file, transforms the records, and prints the result.

```python
import csv

def process_orders():
    with open("orders.csv", newline="") as file:
        rows = list(csv.DictReader(file))

    total = sum(float(row["amount"]) for row in rows)

    print(total)
```

### Problems

- The function depends on a specific file name.
- Testing requires preparing a real file.
- It combines file reading, business logic, and output.
- The calculated result is not returned, so it is harder to verify.
- Testing different inputs requires changing the file.

The function may work correctly, but its design makes testing unnecessarily difficult.

---

## 3. Improve Testability by Separating Responsibilities

Separate the calculation from file access.

```python
def calculate_total(orders):
    return sum(float(order["amount"]) for order in orders)
```

Now the function accepts data directly and returns a result.

```python
orders = [
    {"order_id": 1, "amount": "100"},
    {"order_id": 2, "amount": "200"},
]

result = calculate_total(orders)

assert result == 300.0
```

This function is easier to test because it does not need a file, database, or API.

The file-reading operation can remain separate:

```python
import csv

def read_orders(file_path):
    with open(file_path, newline="") as file:
        return list(csv.DictReader(file))
```

The application can combine both operations:

```python
orders = read_orders("orders.csv")
total = calculate_total(orders)

print(total)
```

**Key principle:** Keep business logic separate from input/output operations whenever practical.

---

## 4. Pure Functions

A **pure function** is a function that:
1. Returns the same output for the same input.
2. Does not produce observable side effects.

Example:

```python
def calculate_discount(price, discount_rate):
    return price * (1 - discount_rate)
```

```python
assert calculate_discount(100, 0.10) == 90
assert calculate_discount(200, 0.20) == 160
```

The function is predictable and easy to test.

### Example of a function with hidden state

```python
tax_rate = 0.18

def calculate_tax(price):
    return price * tax_rate
```

The function depends on a global variable. If another part of the program changes `tax_rate`, the result changes without the function's arguments changing.

A more testable version is:

```python
def calculate_tax(price, tax_rate):
    return price * tax_rate
```

```python
assert calculate_tax(100, 0.18) == 18
assert calculate_tax(100, 0.05) == 5
```

Passing dependencies explicitly makes behavior easier to control in tests.

Note: A pure function can still receive mutable objects, so avoid unexpectedly modifying its input.

---

## 5. Dependency Injection

**Dependency injection** means providing a function or class with the dependencies it needs instead of creating those dependencies internally.

Dependencies might include:
- Database connections
- API clients
- File paths
- Configuration values
- Clocks or random-number generators

### Difficult-to-test approach

```python
import requests

def get_customer(customer_id):
    response = requests.get(
        f"https://api.example.com/customers/{customer_id}"
    )
    response.raise_for_status()
    return response.json()
```

This function directly uses the HTTP library and a fixed API address.

### More testable approach

```python
def get_customer(customer_id, api_client):
    response = api_client.get(
        f"/customers/{customer_id}"
    )
    response.raise_for_status()
    return response.json()
```

The caller supplies `api_client`.

Production code can provide a real client, while tests can provide a fake client.

```python
class FakeResponse:
    def raise_for_status(self):
        pass

    def json(self):
        return {"customer_id": 1, "name": "Amit"}


class FakeAPIClient:
    def get(self, path):
        return FakeResponse()


client = FakeAPIClient()

customer = get_customer(1, client)

assert customer["name"] == "Amit"
```

The test does not require a real API or internet access.

**Important:** Dependency injection does not always require a framework. Passing an object or function as an argument is often enough.

---

## 6. Avoid Side Effects in Business Logic

A side effect is an operation that changes external state or interacts with something outside the function's return value.

Examples include:
- Writing to a file
- Updating a database
- Sending an HTTP request
- Printing to the console
- Changing a global variable

Side effects are not inherently bad. Applications need them. The goal is to keep them separate from logic that can be tested independently.

### Example: Separate transformation from database writing

```python
def transform_order(order):
    return {
        "order_id": order["order_id"],
        "amount": float(order["amount"]),
        "status": order["status"].upper(),
    }
```

```python
def save_orders(orders, repository):
    for order in orders:
        repository.insert(order)
```

The transformation can be tested without a database:

```python
order = {
    "order_id": 101,
    "amount": "250.50",
    "status": "completed",
}

result = transform_order(order)

assert result["amount"] == 250.50
assert result["status"] == "COMPLETED"
```

Database writing can be tested separately using a fake repository or a test database.

---

## 7. Design Functions with Clear Contracts

A function should have a clear definition of:
- What inputs it accepts
- What output it returns
- What errors it may raise
- Whether it modifies its inputs or external state

Example:

```python
def calculate_average(values):
    if not values:
        raise ValueError("values cannot be empty")

    return sum(values) / len(values)
```

Tests can verify both normal and invalid behavior:

```python
assert calculate_average([10, 20, 30]) == 20

try:
    calculate_average([])
except ValueError:
    pass
else:
    raise AssertionError("Expected ValueError")
```

In a real project, `pytest.raises()` is generally a cleaner way to test exceptions.

A clear contract makes it easier to identify whether a failure is caused by incorrect input, a programming bug, or an external system.

---

## 8. Make Time and Randomness Testable

Functions that depend on the current time or random values can produce different results during different test runs.

### Difficult-to-test approach

```python
from datetime import datetime, timezone

def is_expired(expiry_time):
    return datetime.now(timezone.utc) > expiry_time
```

The result depends on when the test runs.

### Better approach

```python
def is_expired(expiry_time, current_time):
    return current_time > expiry_time
```

```python
from datetime import datetime, timezone

expiry = datetime(2026, 12, 31, tzinfo=timezone.utc)
current = datetime(2026, 10, 1, tzinfo=timezone.utc)

assert is_expired(expiry, current) is False
```

The test controls the time explicitly.

The same principle applies to randomness: pass a random-number generator or the generated value into the relevant logic rather than relying on uncontrolled random output.

---

## 9. Testing Data Pipeline Functions

In data engineering, testable code helps isolate data transformations from pipeline infrastructure.

Consider a transformation that filters invalid records.

```python
def filter_valid_orders(orders):
    return [
        order
        for order in orders
        if order["order_id"] is not None
        and float(order["amount"]) >= 0
    ]
```

Example test:

```python
orders = [
    {"order_id": 1, "amount": "100"},
    {"order_id": None, "amount": "200"},
    {"order_id": 3, "amount": "-10"},
]

result = filter_valid_orders(orders)

assert result == [
    {"order_id": 1, "amount": "100"}
]
```

This function can be tested without running an entire ETL pipeline.

However, the example assumes that each record has an `amount` field containing a valid numeric value. If malformed values are possible, define the expected behavior explicitly.

For example, a stricter version could reject malformed amounts:

```python
def normalize_order(order):
    try:
        amount = float(order["amount"])
    except (KeyError, TypeError, ValueError) as exc:
        raise ValueError("Order must contain a valid amount") from exc

    if amount < 0:
        raise ValueError("Amount cannot be negative")

    return {
        **order,
        "amount": amount,
    }
```

This makes the data-quality rules explicit and testable.

### Practical data engineering structure

```text
pipeline/
    extract.py        # Read from APIs, files, or databases
    transform.py      # Business and transformation logic
    load.py           # Write to destination
    config.py         # Configuration
    main.py           # Orchestrate pipeline steps

tests/
    test_transform.py
    test_extract.py
    test_load.py
```

The exact structure depends on the project size. The main idea is to make transformation logic independently testable.

For Spark or PySpark, the same principle applies: isolate DataFrame transformations into functions and test their output using small, controlled input DataFrames.

---

## 10. Unit Tests, Integration Tests, and Mocks

Different tests verify different parts of the application.

| Test type | What it verifies | Example |
|---|---|---|
| Unit test | A small function or component | Testing a discount calculation |
| Integration test | Multiple components working together | Testing a database repository against a test database |
| End-to-end test | A complete workflow | Running an ETL pipeline from input to destination |
| Mock-based test | Behavior involving a replaced dependency | Checking that an API client is called with the correct path |

### When should you mock a dependency?

Mocking is useful when you need to isolate the code under test from an external dependency.

For example, with `pytest` and `unittest.mock`:

```python
from unittest.mock import Mock

def get_customer_name(customer_id, repository):
    customer = repository.get_by_id(customer_id)

    if customer is None:
        return "Unknown"

    return customer["name"]


def test_get_customer_name():
    repository = Mock()
    repository.get_by_id.return_value = {
        "id": 1,
        "name": "Amit",
    }

    result = get_customer_name(1, repository)

    assert result == "Amit"
    repository.get_by_id.assert_called_once_with(1)
```

This test verifies both the returned result and the interaction with the repository.

Do not mock everything. Excessive mocking can make tests verify implementation details rather than meaningful behavior. Use real implementations when they are inexpensive and appropriate, and use integration tests for important interactions with external systems.

---

## 11. Common Design Mistakes

### Mistake 1: One function does everything

A function that reads a file, validates data, transforms records, writes to a database, and sends notifications is difficult to test in isolation.

**Improvement:** Separate these responsibilities into focused functions or components.

### Mistake 2: Excessive global state

Global variables can make tests depend on execution order or on changes made by other tests.

**Improvement:** Pass important values and dependencies explicitly.

### Mistake 3: Hard-coded external dependencies

Hard-coded URLs, file paths, credentials, and configuration make it difficult to control test conditions.

**Improvement:** Use configuration and dependency injection. Never place real secrets in test code.

### Mistake 4: Testing only the happy path

Valid input is only one possible condition.

**Improvement:** Test empty inputs, missing fields, invalid values, boundary conditions, and expected exceptions.

### Mistake 5: Overengineering for testability

Not every function needs an interface, factory, dependency container, or complex abstraction.

**Improvement:** Start with simple functions, clear contracts, and explicit dependencies. Introduce additional abstractions when they solve a real problem.

### Mistake 6: Testing only implementation details

A test that depends on internal variable names or the exact sequence of irrelevant operations may fail after harmless refactoring.

**Improvement:** Test observable behavior and important side effects.

---

## 12. Practical Checklist

Before considering a function easy to test, ask:

- [ ] Does it have one clear responsibility?
- [ ] Are its inputs and outputs explicit?
- [ ] Can its business logic run without a real database or API?
- [ ] Does it avoid unnecessary global state?
- [ ] Are external dependencies supplied or isolated?
- [ ] Can I test normal, invalid, and boundary inputs?
- [ ] Are errors and side effects predictable?
- [ ] Can I run its tests repeatedly without depending on execution order?
- [ ] Are important external integrations covered by suitable integration tests?

Not every function needs to satisfy every item. Use the checklist to identify unnecessary testing difficulties.

---

## 13. Interview Questions and Answers

### Q1. What is testable code?

Testable code is designed so its behavior can be verified easily and reliably through automated tests, with minimal dependence on external systems or hidden state.

### Q2. How do you make Python code more testable?

- Break large functions into smaller functions.
- Separate business logic from input/output operations.
- Pass dependencies explicitly.
- Avoid unnecessary global state.
- Return values instead of relying only on printing or other side effects.
- Define clear input validation and error behavior.

### Q3. Why are pure functions easy to test?

Their output depends only on their inputs, and they do not modify external state. Therefore, tests can provide inputs and compare actual results with expected results without complex setup.

### Q4. What is dependency injection, and why is it useful?

Dependency injection means supplying dependencies to a component rather than creating them inside it. It allows tests to use fake or mocked dependencies instead of real databases, APIs, or other external services.

### Q5. Should all functions be pure?

No. Real applications need side effects to read data, write results, and communicate with external systems. The goal is to keep side effects organized and separate from business logic where practical.

### Q6. How would you make an ETL pipeline testable?

Separate extraction, transformation, and loading responsibilities. Unit-test transformations with controlled input data, test extraction and loading independently, and use integration tests to verify interactions with actual storage or services.

### Q7. What is the difference between a mock and a fake?

A **mock** is commonly used to verify expected interactions, such as whether a method was called with particular arguments. A **fake** is a simplified working implementation, such as an in-memory repository. Both can replace real dependencies in tests.

### Q8. Does testable code always mean more classes and abstractions?

No. Small functions, explicit parameters, and separation of concerns often provide most of the benefit. Additional abstractions should be introduced only when they improve clarity or solve a real design problem.

---

## 14. Quick Revision

- **Testable code:** Code that is easy to verify reliably.
- **Pure function:** Same inputs produce the same outputs, without observable side effects.
- **Dependency injection:** Supply dependencies instead of constructing them inside the code under test.
- **Separation of concerns:** Keep business logic distinct from file, database, and API operations.
- **Unit test:** Verifies a small component in isolation.
- **Integration test:** Verifies that multiple components work together.
- **Mock:** A test double often used to verify interactions.
- **Fake:** A simplified working implementation used in place of a real dependency.

**Remember:** The goal is not to eliminate dependencies or side effects. It is to make them explicit, controlled, and independently testable wherever practical.