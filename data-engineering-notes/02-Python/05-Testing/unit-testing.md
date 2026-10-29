# Unit Testing in Python

## 1. What Is Unit Testing?

**Unit testing** is the process of testing a small, isolated part of a program, such as a function or method, to verify that it behaves as expected.

A unit test typically:
1. Provides known input.
2. Executes the code being tested.
3. Compares the actual result with the expected result.
4. Reports whether the test passed or failed.

Example:

```python
def add(a, b):
    return a + b
```

Unit test:

```python
def test_add():
    assert add(2, 3) == 5
```

The assertion checks whether the function returns the expected result.

If the result is `5`, the test passes. If it is different, the test fails.

## 2. Why Is Unit Testing Important?

Unit testing helps developers:

- Detect bugs early.
- Verify that business logic works correctly.
- Prevent previously fixed bugs from returning.
- Refactor code with greater confidence.
- Document expected behavior through executable examples.
- Test edge cases that may not appear during normal execution.

### Data engineering example

Suppose a pipeline calculates the total sales amount for each customer. A unit test can verify the aggregation logic without reading files, connecting to a database, or running the entire pipeline.

This makes it faster to identify whether a problem is in the transformation logic or in an external system.

---

## 3. Python Testing Frameworks

Python has several ways to write tests.

| Tool | Purpose |
|---|---|
| `assert` | Checks whether a condition is true |
| `unittest` | Built-in testing framework |
| `pytest` | Popular third-party testing framework |
| `unittest.mock` | Replaces dependencies and verifies interactions |
| `coverage.py` | Measures which code is executed by tests |

For most modern Python projects, **pytest** is a convenient starting point because its syntax is simple and it supports fixtures, parameterization, and integration with other tools.

Install it using:

```bash
python -m pip install pytest
```

---

## 4. Writing Your First Unit Test with pytest

Suppose you have this file:

`calculator.py`

```python
def multiply(a, b):
    return a * b
```

Create another file named:

`test_calculator.py`

```python
from calculator import multiply


def test_multiply():
    assert multiply(3, 4) == 12
```

Run the test from the project directory:

```bash
python -m pytest
```

Expected result: one test passes.

### Important naming conventions

By default, pytest discovers:

- Files named `test_*.py` or `*_test.py`.
- Functions whose names begin with `test_`.
- Test methods beginning with `test_` inside appropriately named test classes.

Following these conventions makes test discovery straightforward.

---

## 5. Understanding Assertions

Assertions compare actual behavior with expected behavior.

```python
def is_even(number):
    return number % 2 == 0
```

```python
def test_is_even():
    assert is_even(4) is True
    assert is_even(7) is False
```

Common assertions include:

```python
assert actual == expected
assert actual != unexpected
assert value is None
assert value is not None
assert condition
assert not condition
```

For collections:

```python
def test_get_ids():
    result = [101, 102, 103]

    assert result == [101, 102, 103]
    assert 102 in result
    assert len(result) == 3
```

When an assertion fails, pytest shows the comparison and helps identify the mismatch.

**Best practice:** Test meaningful behavior rather than writing assertions merely to increase the number of tests.

---

## 6. Testing Edge Cases

Testing only normal inputs is insufficient. A good test suite also covers boundary conditions and invalid inputs.

Consider this function:

```python
def calculate_average(values):
    if not values:
        raise ValueError("values cannot be empty")

    return sum(values) / len(values)
```

Tests should cover:

- Normal input.
- A single element.
- An empty list.
- Negative numbers, if allowed.
- Invalid input types, if they are part of the function's contract.

Example:

```python
import pytest

from calculator import calculate_average


def test_average():
    assert calculate_average([10, 20, 30]) == 20


def test_average_single_value():
    assert calculate_average([15]) == 15


def test_average_empty_list():
    with pytest.raises(ValueError):
        calculate_average([])
```

The last test passes only when `calculate_average([])` raises `ValueError`.

### Why edge cases matter

A function may work for ordinary data but fail on empty input, duplicate values, missing fields, or unexpected boundaries. Edge-case tests help uncover these failures before production.

---

## 7. Testing Exceptions

Use `pytest.raises()` when an exception is expected.

```python
import pytest


def divide(a, b):
    if b == 0:
        raise ValueError("Denominator cannot be zero")

    return a / b
```

Test:

```python
def test_divide_by_zero():
    with pytest.raises(
        ValueError,
        match="Denominator cannot be zero",
    ):
        divide(10, 0)
```

This verifies both the exception type and the message.

Do not catch every exception and treat it as a passing test. The test should verify the specific failure that the function is expected to produce.

---

## 8. Parameterized Tests

When the same logic needs to be tested against multiple inputs, use `pytest.mark.parametrize`.

Without parameterization, you might repeat similar test functions.

With parameterization:

```python
import pytest


@pytest.mark.parametrize(
    "a, b, expected",
    [
        (2, 3, 5),
        (0, 5, 5),
        (-2, 2, 0),
        (-3, -4, -7),
    ],
)
def test_add(a, b, expected):
    assert a + b == expected
```

Each row represents a separate test case.

Benefits:
- Less duplicated code.
- Clearer test coverage across different inputs.
- Easier maintenance when adding new cases.

Use parameterization when multiple inputs exercise the same behavior. Use separate test functions when the scenarios have meaningfully different setup or expectations.

---

## 9. Fixtures in pytest

A **fixture** provides reusable setup or data for tests.

For example:

```python
import pytest


@pytest.fixture
def sample_orders():
    return [
        {"order_id": 1, "amount": 100},
        {"order_id": 2, "amount": 200},
    ]


def calculate_total(orders):
    return sum(order["amount"] for order in orders)


def test_calculate_total(sample_orders):
    assert calculate_total(sample_orders) == 300
```

Pytest recognizes the `sample_orders` parameter and supplies the fixture automatically.

Fixtures are useful for:
- Reusable test data.
- Creating temporary files.
- Initializing test database connections.
- Setting up mock objects.
- Cleaning up resources after tests.

### Fixture scope

Fixtures can run at different scopes:

- `function`: once per test function; the default.
- `class`: once per test class.
- `module`: once per test module.
- `session`: once per test session.

Choose a scope that matches the lifetime of the resource. Wider scopes can reduce setup cost, but shared mutable state can make tests interfere with each other.

---

## 10. Mocking External Dependencies

Unit tests should usually isolate the function being tested from external systems.

For example, a function that calls a real API may fail because of network problems even when its own logic is correct.

Mocking allows you to replace that external dependency during the test.

```python
from unittest.mock import Mock


def get_customer_name(customer_id, repository):
    customer = repository.get_by_id(customer_id)

    if customer is None:
        return "Unknown"

    return customer["name"]
```

Test:

```python
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

This test verifies:
- The correct name is returned.
- The repository method is called once.
- The expected customer ID is passed.

For a missing customer:

```python
def test_unknown_customer():
    repository = Mock()
    repository.get_by_id.return_value = None

    result = get_customer_name(999, repository)

    assert result == "Unknown"
```

**Important:** A mock does not prove that a real database or API works. Integration tests are still needed to verify actual interactions with those systems.

---

## 11. Unit Testing Data Engineering Code

Data engineering pipelines commonly include extraction, transformation, and loading.

A practical testing strategy is to test each component at the appropriate level.

### Example: Testing a data transformation

Production code in `transform.py`:

```python
def transform_orders(orders):
    result = []

    for order in orders:
        amount = float(order["amount"])

        if amount < 0:
            continue

        result.append({
            "order_id": order["order_id"],
            "amount": amount,
            "status": order["status"].upper(),
        })

    return result
```

Test code in `test_transform.py`:

```python
from transform import transform_orders


def test_transform_orders():
    orders = [
        {
            "order_id": 1,
            "amount": "100.50",
            "status": "completed",
        },
        {
            "order_id": 2,
            "amount": "-10",
            "status": "cancelled",
        },
    ]

    result = transform_orders(orders)

    assert result == [
        {
            "order_id": 1,
            "amount": 100.50,
            "status": "COMPLETED",
        }
    ]
```

This test checks that:
- Numeric strings are converted to numbers.
- Negative amounts are excluded.
- Status values are standardized.
- The resulting records have the expected structure.

The function currently assumes that each order contains valid `amount`, `order_id`, and `status` fields. If missing fields or malformed amounts are possible in production, define their expected handling and add tests for it.

### What should be tested in an ETL pipeline?

| Component | Useful tests |
|---|---|
| Extraction | API response parsing, pagination handling, missing input files |
| Transformation | Filtering, type conversion, null handling, deduplication, aggregation |
| Loading | Correct records sent to the destination, duplicate handling, error behavior |
| Orchestration | Correct stages run in the expected order |
| Data quality | Schema, null constraints, uniqueness, valid ranges |

For PySpark transformations, use small input DataFrames and compare the output schema and rows with expected results. For important database, storage, or Spark integration behavior, use integration tests in addition to unit tests.

---

## 12. Test Coverage

**Test coverage** measures how much of the code is executed while tests run.

For example, install Coverage.py:

```bash
python -m pip install coverage
```

Run tests with coverage:

```bash
python -m coverage run -m pytest
python -m coverage report -m
```

Generate an HTML report:

```bash
python -m coverage html
```

Coverage can identify code that tests never execute.

However, **100% code coverage does not guarantee correct software**. A line can execute without its output or behavior being properly verified.

Focus on meaningful assertions, important business rules, and high-risk failure scenarios rather than coverage percentage alone.

---

## 13. Common Unit Testing Mistakes

### Mistake 1: Testing only one normal scenario

**Problem:** Bugs remain in edge cases.

**Improvement:** Test valid inputs, invalid inputs, empty data, and important boundaries.

### Mistake 2: Making tests dependent on each other

**Problem:** One test passes only because another test ran first.

**Improvement:** Give each test independent setup and predictable cleanup.

### Mistake 3: Using production data or real services unnecessarily

**Problem:** Tests become slow, expensive, unreliable, or unsafe.

**Improvement:** Use controlled test data, fakes, mocks, or isolated test environments.

### Mistake 4: Mocking everything

**Problem:** Tests may pass even when the real components do not work together.

**Improvement:** Use unit tests for isolated logic and integration tests for real component interactions.

### Mistake 5: Testing implementation details

**Problem:** Harmless refactoring breaks tests even when external behavior remains correct.

**Improvement:** Verify outputs, errors, and meaningful interactions.

### Mistake 6: Ignoring test isolation

**Problem:** Shared state, leftover files, or database changes cause inconsistent results.

**Improvement:** Use fixtures, temporary resources, and appropriate cleanup.

---

## 14. Recommended Project Structure

A simple Python project might look like this:

```text
data_pipeline/
├── transform.py
├── extract.py
├── load.py
├── main.py
├── requirements.txt
└── tests/
    ├── test_transform.py
    ├── test_extract.py
    └── test_load.py
```

Run all discovered tests:

```bash
python -m pytest
```

Run a particular test file:

```bash
python -m pytest tests/test_transform.py
```

Run one test function:

```bash
python -m pytest tests/test_transform.py::test_transform_orders
```

Show more detailed test output:

```bash
python -m pytest -v
```

These commands assume pytest is installed and you run them from the project root.

---

## 15. Unit Testing Best Practices

1. Write small, focused tests.
2. Give tests descriptive names.
3. Use deterministic inputs and expected outputs.
4. Test error conditions as well as successful behavior.
5. Avoid unnecessary dependencies on external systems.
6. Keep tests independent and repeatable.
7. Use fixtures for reusable setup.
8. Use parameterization for related input cases.
9. Mock external dependencies when isolation is useful.
10. Add integration tests for important real-world interactions.
11. Run tests automatically in continuous integration when practical.
12. Update tests when business requirements change.

A useful naming pattern is:

`test_<function>_<scenario>_<expected_behavior>`

For example:

```python
def test_calculate_average_empty_list_raises_value_error():
    ...
```

This name communicates what the test is checking without needing to inspect its entire implementation.

---

## 16. Interview Questions and Answers

### Q1. What is unit testing?

Unit testing verifies a small, isolated component of software to confirm that it behaves according to its expected requirements.

### Q2. What is the difference between unit testing and integration testing?

Unit testing focuses on a small component in isolation. Integration testing verifies that multiple components work correctly together, such as application code interacting with a real database.

### Q3. What is pytest?

Pytest is a Python testing framework that supports automatic test discovery, assertions, fixtures, parameterized tests, and plugins.

### Q4. What is a fixture?

A fixture provides reusable setup, test data, or resources to test functions. Pytest creates and supplies a fixture when a test requests it by name.

### Q5. What is mocking?

Mocking replaces a dependency with a controllable test object. It is useful for simulating API responses, controlling error conditions, and verifying that expected methods are called.

### Q6. What is parameterization?

Parameterization runs the same test logic against multiple sets of inputs and expected outputs, reducing duplicated test code.

### Q7. Does 100% test coverage mean an application has no bugs?

No. Coverage measures code execution, not whether every behavior or requirement has been verified. Tests can execute every line while still missing incorrect results or important scenarios.

### Q8. How would you unit-test an ETL pipeline?

I would isolate transformation logic and test it using controlled input data. I would separately test extraction and loading behavior, mock external dependencies when appropriate, and add integration tests for real storage and service interactions.

### Q9. How do you test a function that raises an exception?

With pytest, use `pytest.raises()` to verify that the expected exception type is raised. If useful, also verify the exception message.

### Q10. Why should unit tests be independent?

Independent tests can run in any order and produce consistent results. This prevents hidden dependencies and reduces failures caused by shared state or test execution order.

---

## 17. Quick Revision

- **Unit test:** Tests a small component in isolation.
- **`assert`:** Verifies that a condition or result matches expectations.
- **pytest:** Python framework for discovering and running tests.
- **`pytest.raises()`:** Verifies expected exceptions.
- **`pytest.mark.parametrize`:** Tests multiple input combinations.
- **Fixture:** Provides reusable test setup or resources.
- **Mock:** Replaces a dependency and can verify interactions.
- **Coverage:** Measures how much code tests execute.
- **Integration test:** Checks that multiple components work together.
- **Best practice:** Test observable behavior using controlled, independent, repeatable tests.

**Remember:** Unit testing is not just about proving that code works for one input. It is about verifying expected behavior across realistic scenarios and catching regressions before they reach production.