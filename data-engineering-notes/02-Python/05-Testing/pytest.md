# Pytest in Python

## 1. What Is Pytest?

`pytest` is a Python testing framework used to test functions, classes, and applications.

It helps developers verify that their code behaves as expected and catches bugs before code reaches production.

In Data Engineering, pytest is useful for testing:

- Data transformation functions.
- API extraction logic.
- SQL and database integrations.
- Data validation rules.
- ETL/ELT pipelines.
- Error handling and retry logic.
- Data quality checks.

**Main idea:** Write tests that automatically verify whether your code produces the expected results.

## 2. Installing Pytest

Install pytest using pip:

```bash
pip install pytest
```

Check the installation:

```bash
pytest --version
```

Run tests from your project's root directory:

```bash
pytest
```

Pytest automatically discovers test files and functions that follow its naming conventions.

Common conventions:

- Test files: `test_*.py` or `*_test.py`
- Test functions: `test_*`
- Test classes: `Test*` (typically without an `__init__` method)

## 3. Your First Pytest Test

Suppose you have a file named `calculator.py`:

```python
def add(a, b):
    return a + b
```

Create another file named `test_calculator.py`:

```python
from calculator import add


def test_add():
    assert add(2, 3) == 5
```

Run:

```bash
pytest
```

Expected result: one test passes.

### Understanding `assert`

```python
assert add(2, 3) == 5
```

This checks whether the actual result equals the expected result.

- If the condition is true, the test passes.
- If the condition is false, pytest reports a failure with useful details.

You do not need to print the result manually or write a separate test runner for simple tests.

## 4. Testing Data Transformation Functions

Data Engineers frequently transform raw data into a clean structure.

Consider this function:

```python
def calculate_total(price, quantity):
    return price * quantity
```

Test it:

```python
from calculator import calculate_total


def test_calculate_total():
    assert calculate_total(100, 3) == 300


def test_zero_quantity():
    assert calculate_total(100, 0) == 0


def test_decimal_price():
    assert calculate_total(12.5, 2) == 25.0
```

Each test verifies a different case.

A good test suite covers normal inputs, boundary conditions, and expected failure cases.

## 5. Testing Data Cleaning Logic

Suppose an ETL pipeline needs to remove records with missing customer IDs.

```python
def clean_customers(customers):
    return [
        customer
        for customer in customers
        if customer.get("customer_id") is not None
    ]
```

Test the function:

```python
from customer_pipeline import clean_customers


def test_clean_customers():
    customers = [
        {"customer_id": 101, "name": "A"},
        {"customer_id": None, "name": "B"},
        {"customer_id": 103, "name": "C"}
    ]

    result = clean_customers(customers)

    assert result == [
        {"customer_id": 101, "name": "A"},
        {"customer_id": 103, "name": "C"}
    ]
```

This test checks that records with missing IDs are removed while valid records are preserved.

Notice that the expected result is compared directly with the actual result. This is a simple and readable approach for testing data transformations.

## 6. Testing Exceptions with `pytest.raises`

Sometimes a function should reject invalid input.

Example:

```python
def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")

    return a / b
```

Test both success and failure:

```python
import pytest
from calculator import divide


def test_divide():
    assert divide(10, 2) == 5


def test_divide_by_zero():
    with pytest.raises(ValueError):
        divide(10, 0)
```

You can also verify the exception message:

```python
def test_divide_error_message():
    with pytest.raises(
        ValueError,
        match="Cannot divide by zero"
    ):
        divide(10, 0)
```

Use `pytest.raises()` when an exception is the expected behavior.

## 7. Parameterized Tests

Sometimes the same function must be tested with many input combinations.

Instead of writing separate functions for every case, use `pytest.mark.parametrize`.

```python
import pytest
from calculator import add


@pytest.mark.parametrize(
    "a, b, expected",
    [
        (2, 3, 5),
        (0, 0, 0),
        (-2, 5, 3),
        (10, 20, 30)
    ]
)
def test_add(a, b, expected):
    assert add(a, b) == expected
```

Pytest runs the test once for each input combination.

### Why use parameterization?

- Reduces duplicate test code.
- Makes test cases easier to maintain.
- Helps test edge cases systematically.
- Makes it easier to expand coverage.

For Data Engineering, parameterization is useful for testing different data types, null values, invalid records, and transformation rules.

## 8. Fixtures in Pytest

A fixture provides reusable setup data or resources for tests.

Fixtures are commonly used to create:

- Sample datasets.
- Temporary files.
- Database connections.
- API clients.
- Test configurations.

Example:

```python
import pytest


@pytest.fixture
def sample_orders():
    return [
        {"order_id": 1, "amount": 100},
        {"order_id": 2, "amount": 200}
    ]


def test_order_count(sample_orders):
    assert len(sample_orders) == 2
```

Pytest automatically passes the fixture result to the test function because its name matches the fixture name.

### Why use fixtures?

Without fixtures, you may repeat the same setup code in many tests.

Fixtures make that setup reusable and easier to maintain.

## 9. Fixture Scope

Fixtures can have different lifetimes.

```python
@pytest.fixture(scope="function")
def sample_data():
    return [10, 20, 30]
```

Common scopes:

| Scope | Lifetime |
|---|---|
| `function` | Once per test function; default |
| `class` | Once per test class |
| `module` | Once per module |
| `package` | Once per package |
| `session` | Once per test session |

Example:

```python
@pytest.fixture(scope="module")
def config():
    return {"environment": "test"}
```

Choose the narrowest scope that meets your needs unless creating the resource is expensive.

**Important:** Wider scopes can share state between tests. Avoid sharing mutable data or resources in ways that make tests depend on execution order.

## 10. Testing API Calls with Mocking

A Data Engineering pipeline may call an external API to retrieve orders.

You usually do not want every unit test to make a real network request.

Suppose `api_client.py` contains:

```python
import requests


def fetch_orders(url):
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    return response.json()
```

Create `test_api_client.py`:

```python
from unittest.mock import Mock, patch

from api_client import fetch_orders


def test_fetch_orders():
    fake_response = Mock()

    fake_response.json.return_value = [
        {"order_id": 1, "amount": 100},
        {"order_id": 2, "amount": 200}
    ]

    fake_response.raise_for_status.return_value = None

    with patch(
        "api_client.requests.get",
        return_value=fake_response
    ) as mock_get:
        result = fetch_orders(
            "https://api.example.com/orders"
        )

    assert len(result) == 2

    mock_get.assert_called_once_with(
        "https://api.example.com/orders",
        timeout=10
    )
```

This test verifies the function's behavior without calling the actual API.

The patch target is `api_client.requests.get` because that is where the function looks up the request dependency.

## 11. Testing Retry Logic

Suppose an API request fails temporarily and succeeds on the second attempt.

A retry function might look like this:

```python
import requests


def fetch_with_retry(url):
    for attempt in range(3):
        try:
            response = requests.get(url, timeout=10)
            response.raise_for_status()
            return response.json()

        except requests.exceptions.ConnectionError:
            if attempt == 2:
                raise
```

Test it with a mock:

```python
import requests
from unittest.mock import Mock, patch

from api_client import fetch_with_retry


def test_fetch_with_retry():
    successful_response = Mock()
    successful_response.json.return_value = [
        {"order_id": 1}
    ]

    with patch(
        "api_client.requests.get",
        side_effect=[
            requests.exceptions.ConnectionError(
                "Temporary network error"
            ),
            successful_response
        ]
    ) as mock_get:
        result = fetch_with_retry(
            "https://api.example.com/orders"
        )

    assert result == [{"order_id": 1}]
    assert mock_get.call_count == 2
```

This verifies that the function retries after a connection error and returns the successful response.

The example omits backoff delays for simplicity. In production, retry policies should be bounded, distinguish transient from permanent errors, and account for idempotency.

## 12. Testing with Temporary Files

Data pipelines often read CSV or JSON files. Pytest provides the `tmp_path` fixture for temporary filesystem operations.

Suppose your function reads a CSV file:

```python
import pandas as pd


def read_orders(file_path):
    return pd.read_csv(file_path)
```

Test it:

```python
import pandas as pd

from data_reader import read_orders


def test_read_orders(tmp_path):
    file_path = tmp_path / "orders.csv"

    file_path.write_text(
        "order_id,amount\n1,100\n2,200\n",
        encoding="utf-8"
    )

    df = read_orders(file_path)

    assert len(df) == 2
    assert list(df.columns) == ["order_id", "amount"]
    assert df["amount"].tolist() == [100, 200]
```

`tmp_path` provides a temporary directory for the test. Pytest manages its lifecycle, so you do not need to create and clean up a permanent test file manually.

Install Pandas if needed:

```bash
pip install pandas
```

## 13. Testing Database-Dependent Code

Database operations can be tested at different levels.

### Unit tests

Mock database dependencies to verify how your code interacts with them.

```python
from unittest.mock import Mock


def save_orders(database, orders):
    for order in orders:
        database.insert(order)


def test_save_orders():
    database = Mock()

    orders = [
        {"order_id": 1},
        {"order_id": 2}
    ]

    save_orders(database, orders)

    assert database.insert.call_count == 2
```

This confirms that two insert operations were requested.

### Integration tests

Use a real test database or an isolated database instance to verify that queries, transactions, and database drivers work correctly.

Mock-based tests alone cannot prove that SQL syntax, schema constraints, permissions, or real database interactions work as expected.

Never run tests against production data unless you have an explicitly approved, safe testing procedure.

## 14. Running Pytest

Run all discovered tests:

```bash
pytest
```

Run a specific file:

```bash
pytest test_calculator.py
```

Run a specific test function:

```bash
pytest test_calculator.py::test_add
```

Show each test result:

```bash
pytest -v
```

Stop after the first failure:

```bash
pytest -x
```

Run tests matching a keyword:

```bash
pytest -k "customer"
```

Show print output during test execution:

```bash
pytest -s
```

Measure test coverage by installing the coverage plugin:

```bash
pip install pytest-cov
pytest --cov=. --cov-report=term-missing
```

Coverage helps identify code that has not been executed by tests, but high coverage does not automatically mean the tests are effective.

## 15. Recommended Project Structure

For a small data pipeline:

```text
project/
├── api_client.py
├── data_reader.py
├── transformations.py
├── pipeline.py
└── tests/
    ├── test_api_client.py
    ├── test_data_reader.py
    ├── test_transformations.py
    └── test_pipeline.py
```

Separating the application code from test code makes the project easier to maintain.

A typical test workflow is:

1. Write or modify a function.
2. Add tests for expected behavior.
3. Test edge cases and failure conditions.
4. Run pytest locally.
5. Fix failing tests.
6. Run the test suite in CI/CD before deployment.

## 16. Common Mistakes

1. **Incorrect naming:** Test files or functions may not be discovered if they do not follow the configured naming conventions.
2. **Incorrect imports:** The test may import the wrong module or fail to locate the project package.
3. **Missing assertions:** A test that runs without checking the result may not verify useful behavior.
4. **Testing only the happy path:** Nulls, invalid input, exceptions, and empty datasets also matter.
5. **Using real APIs unnecessarily:** Unit tests become slow and unreliable when they depend on external services.
6. **Mocking the wrong namespace:** Patch the dependency where the code under test uses it.
7. **Sharing mutable fixture data:** One test may accidentally affect another.
8. **Confusing coverage with quality:** Executed code is not necessarily correctly tested.
9. **Ignoring integration tests:** Mocks cannot verify every real-world interaction.
10. **Using production systems for testing:** Tests should use safe, isolated resources.

## 17. Interview Questions

### Q1. What is pytest?

A Python testing framework used to write and run automated tests.

### Q2. What is the purpose of `assert`?

It checks whether an expression is true. Pytest reports a test failure when the assertion is false.

### Q3. What is a pytest fixture?

A reusable function that provides setup data or resources to test functions.

### Q4. What is `pytest.mark.parametrize`?

A decorator that runs the same test function with multiple input combinations.

### Q5. How do you test an exception?

Use `pytest.raises()` to verify that the expected exception occurs.

### Q6. How would you test an API without making a real request?

Use `unittest.mock.patch()` to replace the HTTP request function with a controlled mock response.

### Q7. What is the difference between unit testing and integration testing?

Unit testing checks a small component in isolation. Integration testing verifies that multiple real components work together.

### Q8. How would you test an ETL pipeline?

Test extraction behavior, transformation logic, data quality rules, loading interactions, and failure handling. Use mocks for external dependencies where appropriate and integration tests for real system interactions.

### Q9. What is fixture scope?

It determines how long a fixture instance is reused, such as per function, module, or test session.

### Q10. Does 100% test coverage guarantee bug-free code?

No. Coverage measures which code was executed, not whether every behavior, edge case, and expected result was tested correctly.

## 18. Quick Revision

- **Pytest:** Python testing framework.
- **`assert`:** Verify expected results.
- **`pytest.raises()`:** Test expected exceptions.
- **`pytest.mark.parametrize`:** Run tests with multiple input sets.
- **`@pytest.fixture`:** Reuse setup data and resources.
- **`tmp_path`:** Work with temporary files and directories.
- **`unittest.mock`:** Replace dependencies during unit tests.
- **`pytest -v`:** Display detailed test results.
- **`pytest -x`:** Stop after the first failure.
- **Unit tests:** Test individual components in isolation.
- **Integration tests:** Verify real component interactions.
- **Data Engineering principle:** Test transformations, data quality, API failures, and recovery behavior before deploying a pipeline.

**For Data Engineer interviews, focus on:** assertions, fixtures, parameterization, exception testing, mocking APIs, temporary files, unit vs. integration tests, and testing ETL pipelines.