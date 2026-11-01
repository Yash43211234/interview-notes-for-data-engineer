

# Test Fixtures and Mocks in Data Engineering

## 1. Introduction

When testing data pipelines, we need controlled inputs, predictable dependencies, and repeatable test conditions.

Two important testing concepts help us achieve this:

- **Test Fixtures:** Provide the data, objects, and setup required to run tests.
- **Mocks:** Replace real dependencies with simulated objects so we can test a component without calling external systems.

These techniques are especially useful when testing Python, PySpark, APIs, databases, and data pipelines.

### Simple example

Suppose a PySpark pipeline:
1. Reads customer data from ADLS.
2. Removes invalid records.
3. Calculates customer revenue.
4. Writes the output to a Delta table.

Testing the entire pipeline against real ADLS and production databases can be expensive and slow.

Instead:
- Use a fixture to create a small customer DataFrame.
- Mock the ADLS read operation.
- Test the transformation using predictable input.
- Mock the output-writing operation.

This allows us to test transformation logic without depending on external infrastructure.

---

## 2. What Is a Test Fixture?

A test fixture is a predefined setup used by tests.

It provides the data or resources required before a test executes.

A fixture may create:
- Sample DataFrames.
- Temporary files.
- Database connections.
- Configuration objects.
- Test users.
- Temporary directories.
- Spark sessions.

### Example without a fixture

```python
def test_customer_count(spark):
    data = [
        (1, "Amit"),
        (2, "Rahul"),
        (3, "Priya")
    ]

    df = spark.createDataFrame(
        data,
        ["customer_id", "customer_name"]
    )

    assert df.count() == 3
```

If several tests need the same DataFrame, we would repeatedly write the setup code.

### Example using a fixture

```python
import pytest

@pytest.fixture
def customer_df(spark):
    data = [
        (1, "Amit"),
        (2, "Rahul"),
        (3, "Priya")
    ]

    return spark.createDataFrame(
        data,
        ["customer_id", "customer_name"]
    )
```

Use the fixture in a test:

```python
def test_customer_count(customer_df):
    assert customer_df.count() == 3
```

Pytest automatically provides the fixture when the test requests it as an argument.

### Benefits

1. Reuses test setup.
2. Reduces duplicate code.
3. Makes tests easier to read.
4. Creates consistent test inputs.
5. Simplifies maintenance.

---

## 3. What Is a Mock?

A mock is a simulated object used instead of a real dependency during testing.

It allows us to control how a dependency behaves without actually calling it.

Mocks are useful when testing components that depend on:
- APIs.
- Databases.
- Cloud storage.
- External services.
- File systems.
- Message queues.

### Example

Suppose a Python function retrieves customer information from an external API.

```python
import requests

def get_customer(customer_id):
    response = requests.get(
        f"https://api.example.com/customers/{customer_id}"
    )

    response.raise_for_status()

    return response.json()
```

Calling a real API during every test can be:
- Slow.
- Expensive.
- Unreliable.
- Dependent on network availability.

We can mock the API response instead.

```python
from unittest.mock import patch

def test_get_customer():
    expected = {
        "customer_id": 101,
        "name": "Amit"
    }

    with patch("my_module.requests.get") as mock_get:
        mock_get.return_value.json.return_value = expected
        mock_get.return_value.raise_for_status.return_value = None

        result = get_customer(101)

    assert result == expected
```

Here, `my_module` should be replaced with the actual module containing `get_customer`.

The test does not call the real API. It uses a controlled response.

### Benefits

- Avoids external dependencies.
- Makes tests faster.
- Allows testing failure scenarios.
- Provides predictable results.
- Helps isolate business logic.

---

## 4. Fixture vs Mock

These concepts are related but solve different problems.

| Test Fixture | Mock |
|---|---|
| Provides test setup or resources | Simulates a dependency |
| Often provides sample data | Often replaces external behavior |
| Can create DataFrames and temporary files | Can simulate APIs and database calls |
| Used to prepare a test | Used to isolate a test from dependencies |
| Commonly managed by pytest fixtures | Commonly created using `unittest.mock` |

### Example

Suppose we test a customer transformation.

- Fixture: Creates a sample customer DataFrame.
- Mock: Simulates the function that reads customer data from ADLS.
- Test: Verifies that the transformation removes invalid customers.

A fixture and a mock can be used in the same test.

---

## 5. Types of Test Fixtures in Pytest

Pytest supports different fixture scopes.

The scope controls how often a fixture is created.

| Scope | Lifetime |
|---|---|
| `function` | Once for each test function |
| `class` | Once for each test class |
| `module` | Once for each test module |
| `package` | Once for each package |
| `session` | Once for the entire test session |

### Function Scope

```python
@pytest.fixture(scope="function")
def sample_data():
    return [10, 20, 30]
```

A new fixture instance is created for each test function.

### Module Scope

```python
@pytest.fixture(scope="module")
def configuration():
    return {
        "environment": "test"
    }
```

The fixture is shared by tests in the same module.

### Session Scope

```python
@pytest.fixture(scope="session")
def shared_configuration():
    return {
        "environment": "test",
        "region": "local"
    }
```

The fixture is created once for the test session.

### Which scope should we use?

- Use function scope for mutable test data and isolated resources.
- Use module scope when setup is expensive and safe to share within a module.
- Use session scope for expensive shared resources, such as a Spark session, when appropriate.

**Important:** Shared fixtures can create test interference if one test modifies shared data or state. Use the narrowest practical scope.

---

## 6. Fixture Setup and Teardown

Sometimes tests need resources that must be cleaned up after execution.

Examples:
- Temporary files.
- Database connections.
- Temporary tables.
- Test directories.
- Spark sessions.

Pytest supports setup and teardown using `yield`.

### Example

```python
import pytest

@pytest.fixture
def temporary_file(tmp_path):
    file_path = tmp_path / "customers.csv"

    file_path.write_text(
        "customer_id,name\n1,Amit\n2,Rahul\n"
    )

    yield file_path

    # Cleanup can be performed here if required.
```

Test:

```python
def test_file_exists(temporary_file):
    assert temporary_file.exists()
```

The code before `yield` performs setup.

The code after `yield` performs teardown.

Pytest's built-in `tmp_path` fixture creates a temporary directory for each test invocation and manages its cleanup.

### Why is teardown important?

Without proper cleanup:
- Temporary files may remain.
- Test databases may contain old records.
- Resources may be unnecessarily consumed.
- Tests may affect one another.

---

## 7. Common Mocking Tools in Python

Python provides mocking utilities through `unittest.mock`.

Important components include:

| Tool | Purpose |
|---|---|
| `Mock` | Creates a mock object |
| `MagicMock` | Creates a mock with common Python magic methods |
| `patch` | Temporarily replaces an object |
| `patch.object` | Replaces an attribute of an existing object |
| `side_effect` | Defines custom behavior or exceptions |
| `return_value` | Defines the value returned by a mock |
| `assert_called_once_with()` | Checks that a mock was called once with expected arguments |

### A. Mock

```python
from unittest.mock import Mock

mock_function = Mock(return_value=100)

result = mock_function()

assert result == 100
```

### B. Return Value

```python
mock_api = Mock()

mock_api.get_customer.return_value = {
    "customer_id": 101,
    "name": "Amit"
}

result = mock_api.get_customer(101)
```

### C. Side Effect

`side_effect` can return different values or raise exceptions.

```python
from unittest.mock import Mock

mock_function = Mock(
    side_effect=[100, 200, 300]
)

assert mock_function() == 100
assert mock_function() == 200
assert mock_function() == 300
```

Simulate a failure:

```python
mock_function = Mock(
    side_effect=ConnectionError("Service unavailable")
)

try:
    mock_function()
except ConnectionError:
    print("Connection failed")
```

### D. Verify Function Calls

```python
from unittest.mock import Mock

mock_writer = Mock()

mock_writer("orders")

mock_writer.assert_called_once_with("orders")
```

This checks whether the dependency was called with the expected argument.

---

## 8. Fixtures in PySpark Testing

Fixtures are particularly useful when testing PySpark transformations.

A typical fixture can create a Spark session and sample DataFrames.

### Example: Spark Session Fixture

```python
import pytest

from pyspark.sql import SparkSession

@pytest.fixture(scope="session")
def spark():
    session = (
        SparkSession.builder
        .master("local[2]")
        .appName("pytest-spark-tests")
        .getOrCreate()
    )

    yield session

    session.stop()
```

This creates a local Spark session for the test session.

In a larger test suite, make sure the Spark session is not already owned and managed by another test framework or fixture.

### Customer DataFrame Fixture

```python
@pytest.fixture
def customer_df(spark):
    data = [
        (1, "Amit", "ACTIVE"),
        (2, "Rahul", "INACTIVE"),
        (3, "Priya", "ACTIVE")
    ]

    columns = [
        "customer_id",
        "customer_name",
        "status"
    ]

    return spark.createDataFrame(
        data,
        columns
    )
```

### Transformation

```python
from pyspark.sql.functions import col

def get_active_customers(df):
    return df.filter(
        col("status") == "ACTIVE"
    )
```

### Test

```python
def test_active_customer_filter(customer_df):
    result = get_active_customers(customer_df)

    assert result.count() == 2
```

The fixture provides predictable input, and the test validates the transformation.

---

## 9. Mocking External Dependencies in a Data Pipeline

Consider a pipeline that:
1. Reads orders from cloud storage.
2. Applies transformations.
3. Writes results to a Delta table.

We should test transformation logic separately from cloud infrastructure.

### Example pipeline functions

```python
def load_orders(path):
    return spark.read.parquet(path)


def transform_orders(df):
    return df.filter(
        df["amount"] > 0
    )


def save_orders(df, path):
    df.write.format("delta").mode("append").save(path)
```

For a clean testable design, dependency objects or the Spark session should be passed into these functions rather than relying on an undeclared global variable.

### Better design

```python
def load_orders(spark, path):
    return spark.read.parquet(path)


def transform_orders(df):
    return df.filter(
        df["amount"] > 0
    )


def save_orders(df, path):
    df.write.format("delta").mode("append").save(path)
```

### Mock the storage reader

```python
from unittest.mock import Mock

def test_pipeline_transformation():
    sample_df = Mock()

    mock_spark = Mock()
    mock_spark.read.parquet.return_value = sample_df

    result = load_orders(
        mock_spark,
        "/test/orders"
    )

    assert result is sample_df

    mock_spark.read.parquet.assert_called_once_with(
        "/test/orders"
    )
```

This tests whether the loader calls the expected Spark read method.

It does not test whether Spark can actually read the Parquet file. That requires an integration test using a real Spark session and test data.

---

## 10. Mocking Database Operations

Suppose a pipeline writes processed records to a database.

We do not want every unit test to connect to a real database.

A mock can simulate the database writer.

### Example

```python
from unittest.mock import Mock

def write_customer_data(df, database_client):
    database_client.insert_many(
        df
    )
```

Test:

```python
def test_write_customer_data():
    mock_database = Mock()
    sample_data = [
        {"customer_id": 101, "name": "Amit"}
    ]

    write_customer_data(
        sample_data,
        mock_database
    )

    mock_database.insert_many.assert_called_once_with(
        sample_data
    )
```

This verifies that the database client is called correctly.

It does not verify:
- SQL syntax.
- Database constraints.
- Actual persistence.
- Transaction behavior.

Those require integration testing against a real or dedicated test database.

---

## 11. Mock vs Stub vs Fake

These terms are sometimes used interchangeably, but they describe different testing techniques.

### Stub

A stub provides predefined responses.

Example:

```python
def get_customer():
    return {
        "customer_id": 101,
        "name": "Amit"
    }
```

It provides predictable data without calling a real service.

### Mock

A mock simulates a dependency and can also verify how it was called.

Example:

```python
from unittest.mock import Mock

mock_api = Mock(return_value="SUCCESS")

result = mock_api()

assert result == "SUCCESS"
mock_api.assert_called_once()
```

### Fake

A fake is a simplified working implementation of a dependency.

Example:

```python
class FakeCustomerRepository:
    def __init__(self):
        self.customers = {}

    def save(self, customer):
        self.customers[
            customer["customer_id"]
        ] = customer

    def get(self, customer_id):
        return self.customers.get(customer_id)
```

This fake repository behaves like a small in-memory data store.

### Comparison

| Technique | Behavior |
|---|---|
| Stub | Returns predefined responses |
| Mock | Simulates behavior and verifies interactions |
| Fake | Provides a simplified working implementation |
| Fixture | Prepares test data and resources |

---

## 12. Testing Error Scenarios with Mocks

Mocks are useful for testing failures that are difficult to reproduce with real services.

Examples:
- API timeout.
- Database connection failure.
- Storage permission error.
- Network interruption.
- Service unavailable response.

### Example: API failure

```python
from unittest.mock import patch

import requests

def get_customer_data():
    response = requests.get(
        "https://api.example.com/customers"
    )

    response.raise_for_status()

    return response.json()
```

Test:

```python
def test_api_failure():
    with patch("my_module.requests.get") as mock_get:
        mock_get.side_effect = requests.Timeout(
            "Request timed out"
        )

        try:
            get_customer_data()
            assert False, "Expected timeout"
        except requests.Timeout:
            pass
```

Replace `my_module` with the actual module name.

A cleaner version uses pytest's exception assertion:

```python
import pytest

def test_api_timeout():
    with patch("my_module.requests.get") as mock_get:
        mock_get.side_effect = requests.Timeout(
            "Request timed out"
        )

        with pytest.raises(requests.Timeout):
            get_customer_data()
```

### Why test failures?

A production pipeline must handle more than successful execution.

It should also behave correctly when:
- An external service is unavailable.
- A file is missing.
- A database rejects a write.
- An API returns an error.
- A network request times out.

---

## 13. Fixtures and Mocks in CI/CD

Fixtures and mocks make automated tests faster and more repeatable.

Typical CI workflow:

```text
Developer Pushes Code
         |
         v
    CI Pipeline
         |
         v
   Install Dependencies
         |
         v
   Create Test Fixtures
         |
         v
    Run Unit Tests
         |
         v
    Run Mock Tests
         |
         v
  Run Integration Tests
         |
         v
    Generate Report
         |
         v
   Approve Deployment
```

### Benefits in CI/CD

- Tests do not require production credentials.
- External services do not need to be available for unit tests.
- Tests run with predictable inputs.
- Failures are easier to reproduce.
- Developers receive faster feedback.

**Important:** Mocks should not replace all integration tests. A pipeline can pass mocked unit tests and still fail when interacting with a real database, storage account, or Spark environment.

---

## 14. Best Practices

1. Keep fixtures small and focused.
2. Use meaningful fixture names.
3. Reuse fixtures when setup is genuinely shared.
4. Choose fixture scope carefully.
5. Clean up temporary resources.
6. Use mocks for external dependencies.
7. Mock only the dependency boundary, not every internal function.
8. Avoid excessive mocking of business logic.
9. Verify important interactions with mock assertions.
10. Test both successful and failure scenarios.
11. Use real Spark DataFrames for transformation tests.
12. Use real test infrastructure for important integration tests.
13. Keep test data separate from production data.
14. Avoid shared mutable state between tests.
15. Ensure tests are repeatable and independent.

---

## 15. Common Mistakes to Avoid

1. **Confusing fixtures with mocks:** Fixtures prepare the test; mocks simulate dependencies.
2. **Mocking everything:** Excessive mocking can hide real integration problems.
3. **Using production services in unit tests:** This makes tests slower and less reliable.
4. **Ignoring cleanup:** Temporary resources can affect later tests.
5. **Using shared mutable fixtures carelessly:** One test may change data used by another.
6. **Not verifying mock calls:** A test may pass without confirming the expected interaction.
7. **Mocking the wrong import location:** Patch the name where the code under test looks it up.
8. **Treating mock tests as integration tests:** A mock cannot prove that a real service works.
9. **Using mocked DataFrames for Spark logic:** A generic mock does not reproduce Spark's distributed DataFrame behavior.
10. **Ignoring failure scenarios:** Test timeouts, exceptions, and unavailable dependencies.

---

## 16. Practical Interview Scenario

**Question: Your PySpark pipeline reads data from ADLS, transforms it, and writes to Delta Lake. How would you use fixtures and mocks to test it?**

### Answer

I would separate the pipeline into ingestion, transformation, and output-writing components.

My approach would be:

1. Create a pytest fixture that provides a local Spark session.
2. Create another fixture containing a small sample DataFrame.
3. Test the transformation using the actual Spark DataFrame.
4. Mock the external storage-reading dependency when testing the pipeline's orchestration logic.
5. Mock the output writer to verify that it is called with the expected DataFrame and destination.
6. Test error scenarios such as storage failures and write exceptions.
7. Run integration tests separately using actual test storage and Delta tables.

This keeps unit tests fast while still validating real integrations in dedicated tests.

---

## 17. Common Interview Questions and Answers

### Q1. What is a test fixture?

A fixture provides predefined data or resources required to execute a test.

### Q2. What is a mock?

A mock simulates an external dependency and can verify how the tested code interacts with it.

### Q3. What is the difference between a fixture and a mock?

A fixture prepares the test environment or data. A mock replaces a dependency with controlled simulated behavior.

### Q4. What is pytest?

Pytest is a Python testing framework used to write and execute automated tests.

### Q5. What is `@pytest.fixture`?

It is a decorator that marks a function as a fixture so pytest can provide its return value to test functions.

### Q6. What are fixture scopes?

They determine how often a fixture is created: function, class, module, package, or session.

### Q7. What is the purpose of `yield` in a fixture?

It separates fixture setup from teardown. Code after `yield` executes during fixture cleanup.

### Q8. What is `unittest.mock`?

It is a standard Python library that provides tools such as `Mock`, `MagicMock`, and `patch` for simulating dependencies.

### Q9. What is the difference between `return_value` and `side_effect`?

`return_value` defines the result returned by a mock. `side_effect` can define a sequence of results, execute custom behavior, or raise an exception.

### Q10. What is the difference between Mock and MagicMock?

`MagicMock` extends `Mock` with implementations for common Python magic methods, such as iteration and item access.

### Q11. Why do we mock APIs during testing?

To avoid real network calls, improve test speed, control responses, and test error scenarios.

### Q12. How do you test PySpark transformations?

I create a local Spark session, prepare sample DataFrames through fixtures, execute the transformation, and compare the output with expected results.

### Q13. Can mocks replace integration testing?

No. Mocks test isolated behavior and interactions. Integration tests verify that actual components work together.

### Q14. What is a fake dependency?

A fake is a simplified working implementation of a dependency, such as an in-memory repository.

### Q15. How do you test an API timeout?

I use `patch` to make the API call raise a timeout exception and verify that the application handles or propagates it correctly.

### Q16. Where should you patch a dependency?

Patch the name in the module where the code under test looks it up, rather than automatically patching its original definition.

### Q17. Why should fixtures be isolated?

Isolation prevents one test from modifying shared state and causing unpredictable failures in other tests.

---

## 18. Quick Revision

| Concept | Meaning |
|---|---|
| Fixture | Prepares test data and resources |
| Mock | Simulates a dependency |
| Stub | Returns predefined responses |
| Fake | Simplified working implementation |
| `@pytest.fixture` | Defines a pytest fixture |
| `scope="function"` | Creates fixture for each test |
| `scope="session"` | Shares fixture across test session |
| `yield` | Separates setup and teardown |
| `Mock` | Creates a mock object |
| `MagicMock` | Mock with magic-method support |
| `patch` | Temporarily replaces a dependency |
| `return_value` | Defines mock output |
| `side_effect` | Defines custom behavior or exceptions |
| `assert_called_once_with()` | Verifies a mock call |

### Remember These Five Points

1. Fixtures prepare test data and resources.
2. Mocks simulate external dependencies.
3. Pytest provides reusable fixtures with different scopes.
4. `unittest.mock` provides tools for mocking and patching.
5. Use real Spark DataFrames for transformation tests and real test infrastructure for integration tests.

---

## 19. Self-Assessment

Try answering these questions without looking at the notes.

1. What is a test fixture?
2. What is a mock?
3. What is the difference between fixtures and mocks?
4. What are the different pytest fixture scopes?
5. What is the purpose of `yield` in a fixture?
6. What is the difference between Mock and MagicMock?
7. What is the difference between `return_value` and `side_effect`?
8. How do you mock an API call?
9. How would you test an API timeout?
10. What is the difference between a stub, mock, and fake?
11. How would you create a Spark session fixture?
12. How would you test a PySpark transformation using fixtures?
13. Why should external storage calls be mocked in unit tests?
14. Why are mocks not a replacement for integration tests?
15. Design a fixture-and-mock testing strategy for an ADLS-to-Delta pipeline.

**Interview readiness:** You should be able to explain fixtures, mocks, fixture scopes, setup and teardown, `unittest.mock`, and how to apply these techniques to a PySpark data pipeline.

