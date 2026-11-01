# PySpark Unit Testing

## 1. What Is Unit Testing?

Unit testing is the process of testing small, individual parts of an application to verify that they work as expected.

In PySpark, we commonly test:

- DataFrame transformations.
- Filtering conditions.
- Aggregations.
- Joins.
- Null handling.
- Data validation rules.
- Custom functions.

**Example:**

Suppose we have a function that filters orders with an amount greater than zero.

We write a unit test to verify that:
- Valid orders are returned.
- Invalid orders are excluded.
- The output contains the expected records.

**Key idea:** Unit testing helps identify errors early and ensures that changes to code do not break existing functionality.

---

## 2. Why Is Unit Testing Important in Data Engineering?

Data pipelines process large amounts of data. A small transformation error can affect thousands or millions of records.

Unit testing helps to:

1. Identify bugs before production.
2. Validate transformation logic.
3. Prevent unexpected changes.
4. Verify business rules.
5. Improve code reliability.
6. Support safe refactoring.
7. Reduce production failures.

### Example

A pipeline calculates total sales for each customer.

If the aggregation logic is incorrect, business reports may show incorrect revenue.

A unit test can verify the expected totals using a small test dataset.

---

## 3. Common Testing Frameworks

Two common tools for PySpark testing are:

| Framework | Purpose |
|---|---|
| Pytest | Python testing framework |
| unittest | Python's built-in testing framework |

For DataFrame comparison, additional libraries such as `chispa` can make testing easier.

### Pytest

Pytest is widely used because it provides:
- Simple test functions.
- Fixtures.
- Parameterized tests.
- Clear assertion failures.

Install:

```bash
pip install pytest pyspark
```

Optional DataFrame comparison library:

```bash
pip install chispa
```

---

## 4. Creating a SparkSession for Testing

PySpark tests need a SparkSession to create and process DataFrames.

Example:

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
        .master("local[2]")
        .appName("PySparkUnitTesting")
        .getOrCreate()
)
```

### Understanding the Configuration

- `master("local[2]")`: Runs Spark locally using two worker threads.
- `appName()`: Sets the application name.
- `getOrCreate()`: Returns an existing SparkSession or creates one.

For unit tests, local execution is usually sufficient.

---

## 5. Testing with Pytest Fixtures

A fixture provides reusable setup logic for tests.

Instead of creating a SparkSession separately in every test, we can define one fixture.

### Example: conftest.py

```python
import pytest

from pyspark.sql import SparkSession


@pytest.fixture(scope="session")
def spark():

    session = (
        SparkSession.builder
            .master("local[2]")
            .appName("PySparkTests")
            .getOrCreate()
    )

    yield session

    session.stop()
```

### Understanding the Fixture

- `@pytest.fixture`: Marks a function as a fixture.
- `scope="session"`: Reuses the SparkSession throughout the test session.
- `yield`: Provides the SparkSession to tests.
- `session.stop()`: Stops Spark after testing finishes.

**Remember:** Fixtures help avoid repeating setup code.

---

## 6. Creating a Sample DataFrame for Testing

We should use small, controlled datasets instead of depending on production data.

Example:

```python
from pyspark.sql.types import (
    StructType,
    StructField,
    IntegerType,
    DoubleType,
    StringType
)

schema = StructType([
    StructField("order_id", IntegerType(), False),
    StructField("customer_id", IntegerType(), False),
    StructField("amount", DoubleType(), True),
    StructField("status", StringType(), True)
])

data = [
    (1, 101, 500.0, "SUCCESS"),
    (2, 102, 300.0, "SUCCESS"),
    (3, 101, 200.0, "CANCELLED"),
    (4, 103, 100.0, "SUCCESS")
]

df = spark.createDataFrame(data, schema)
```

This DataFrame can be reused to test different transformations.

**Important:** Explicit schemas make test data more predictable than relying on schema inference.

---

## 7. Testing a DataFrame Transformation

Consider a function that filters successful orders.

### Production Code: transformations.py

```python
from pyspark.sql.functions import col


def filter_successful_orders(df):

    return df.filter(
        col("status") == "SUCCESS"
    )
```

### Test Code: test_transformations.py

```python
from transformations import filter_successful_orders


def test_filter_successful_orders(spark):

    data = [
        (1, 101, 500.0, "SUCCESS"),
        (2, 102, 300.0, "CANCELLED"),
        (3, 103, 200.0, "SUCCESS")
    ]

    columns = [
        "order_id",
        "customer_id",
        "amount",
        "status"
    ]

    df = spark.createDataFrame(data, columns)

    result = filter_successful_orders(df)

    actual_ids = {
        row["order_id"]
        for row in result.collect()
    }

    expected_ids = {1, 3}

    assert actual_ids == expected_ids
```

### What Happens?

1. Create a small input DataFrame.
2. Pass it to the transformation function.
3. Collect the small test result.
4. Compare actual and expected order IDs.
5. The test passes if both sets match.

**Important:** `collect()` is acceptable for small, controlled test datasets. Avoid collecting large production datasets to the driver.

---

## 8. Testing Aggregations

Suppose we need to calculate total sales for each customer.

### Production Code

```python
from pyspark.sql.functions import sum


def calculate_customer_sales(df):

    return (
        df.groupBy("customer_id")
          .agg(
              sum("amount").alias("total_sales")
          )
    )
```

### Test Code

```python
def test_calculate_customer_sales(spark):

    data = [
        (1, 101, 500.0),
        (2, 101, 200.0),
        (3, 102, 300.0)
    ]

    columns = [
        "order_id",
        "customer_id",
        "amount"
    ]

    df = spark.createDataFrame(data, columns)

    result = calculate_customer_sales(df)

    actual = {
        row["customer_id"]: row["total_sales"]
        for row in result.collect()
    }

    expected = {
        101: 700.0,
        102: 300.0
    }

    assert actual == expected
```

This verifies that the aggregation produces the expected totals.

---

## 9. Testing DataFrame Equality

Sometimes comparing individual columns or collected rows becomes lengthy.

We can use `chispa` to compare DataFrames.

Install:

```bash
pip install chispa
```

Example:

```python
from chispa import assert_df_equality


def test_filter_orders(spark):

    input_data = [
        (1, "SUCCESS"),
        (2, "CANCELLED")
    ]

    input_df = spark.createDataFrame(
        input_data,
        ["order_id", "status"]
    )

    actual_df = input_df.filter(
        "status = 'SUCCESS'"
    )

    expected_df = spark.createDataFrame(
        [(1, "SUCCESS")],
        ["order_id", "status"]
    )

    assert_df_equality(
        actual_df,
        expected_df,
        ignore_nullable=True
    )
```

### What Does It Check?

It compares the actual and expected DataFrames.

By default, DataFrame equality checks can be sensitive to:
- Column names.
- Schema.
- Data types.
- Row contents.
- Row ordering, depending on the comparison method.

`ignore_nullable=True` ignores differences in nullable metadata. It does not ignore differences in actual NULL values.

**Important:** DataFrame rows are not inherently ordered. If order matters, explicitly sort both DataFrames before comparing them.

---

## 10. Testing Null Handling

Null handling is important in data engineering because source data often contains missing values.

### Production Code

```python
from pyspark.sql.functions import col


def remove_null_customers(df):

    return df.filter(
        col("customer_id").isNotNull()
    )
```

### Test Code

```python
def test_remove_null_customers(spark):

    data = [
        (1, 101),
        (2, None),
        (3, 103)
    ]

    df = spark.createDataFrame(
        data,
        ["order_id", "customer_id"]
    )

    result = remove_null_customers(df)

    actual_ids = {
        row["order_id"]
        for row in result.collect()
    }

    assert actual_ids == {1, 3}
```

This verifies that records with NULL customer IDs are removed.

---

## 11. Testing Joins

Suppose we join orders with customer details.

### Production Code

```python
def join_orders_customers(orders, customers):

    return orders.join(
        customers,
        on="customer_id",
        how="inner"
    )
```

### Test Code

```python
def test_join_orders_customers(spark):

    orders = spark.createDataFrame(
        [
            (1, 101, 500.0),
            (2, 102, 300.0)
        ],
        ["order_id", "customer_id", "amount"]
    )

    customers = spark.createDataFrame(
        [
            (101, "Rahul"),
            (102, "Priya")
        ],
        ["customer_id", "customer_name"]
    )

    result = join_orders_customers(
        orders,
        customers
    )

    assert result.count() == 2
```

A stronger test should also verify the actual customer names and order values, not just the row count.

---

## 12. Testing Exceptions

Sometimes a function should reject invalid input.

Pytest provides `pytest.raises()` to verify that an expected exception occurs.

Example:

```python
import pytest


def validate_amount(amount):

    if amount <= 0:
        raise ValueError(
            "Amount must be positive"
        )

    return amount
```

Test:

```python
def test_invalid_amount():

    with pytest.raises(ValueError):
        validate_amount(-100)
```

This test passes when `validate_amount(-100)` raises a `ValueError`.

---

## 13. Parameterized Testing

Parameterized tests allow us to run the same test with multiple input values.

Example:

```python
import pytest


@pytest.mark.parametrize(
    "amount, expected",
    [
        (100, True),
        (500, True),
        (-100, False),
        (0, False)
    ]
)
def test_amount_validation(amount, expected):

    result = amount > 0

    assert result == expected
```

### Why Use It?

- Reduces repeated test code.
- Tests multiple scenarios.
- Makes edge cases easier to cover.
- Improves test readability.

---

## 14. What Should We Test in a Data Pipeline?

A Data Engineer should test more than whether a function executes successfully.

| Test Area | What to Validate |
|---|---|
| Schema | Column names and data types |
| Null handling | Missing values |
| Duplicates | Duplicate business keys |
| Filtering | Correct records retained |
| Aggregation | Correct calculated values |
| Joins | Correct matching records |
| Data types | Correct casting and conversion |
| Business rules | Valid business conditions |
| Empty DataFrame | Correct behavior with no records |
| Edge cases | Boundary and unusual inputs |

### Example Edge Cases

For an order-processing pipeline, test:

1. Empty input DataFrame.
2. NULL customer ID.
3. Negative amount.
4. Zero amount.
5. Duplicate order ID.
6. Invalid status.
7. Very large amount.
8. Unexpected date value.

---

## 15. Unit Testing vs Integration Testing

These are different testing levels.

| Unit Testing | Integration Testing |
|---|---|
| Tests a small function or component | Tests multiple components together |
| Usually uses small controlled data | May use files, databases, or external systems |
| Faster to execute | Usually takes longer |
| Identifies logic errors | Identifies interaction and connectivity issues |
| Often uses mocked dependencies | Uses real or test versions of dependencies |

### Example

**Unit Test:**

Test whether a PySpark function correctly filters invalid orders.

**Integration Test:**

Test whether the complete pipeline can:
1. Read input data from storage.
2. Apply transformations.
3. Write the output to a Delta table.
4. Produce the expected records.

**Interview point:** Unit testing validates individual components, while integration testing validates how components work together.

---

## 16. Running PySpark Tests

A common project structure:

```text
project/
│
├── src/
│   └── transformations.py
│
├── tests/
│   ├── conftest.py
│   └── test_transformations.py
│
└── requirements.txt
```

Run all tests:

```bash
pytest
```

Run a specific test file:

```bash
pytest tests/test_transformations.py
```

Run with detailed output:

```bash
pytest -v
```

Run a specific test function:

```bash
pytest tests/test_transformations.py::test_filter_successful_orders
```

### Understanding the Results

- `PASSED`: Test completed successfully.
- `FAILED`: Test ran but an assertion failed.
- `ERROR`: An error occurred while setting up or executing the test.
- `SKIPPED`: Test was intentionally skipped.

---

## 17. Best Practices

1. Keep transformation logic in reusable functions.
2. Use small and controlled test datasets.
3. Define explicit schemas.
4. Test both valid and invalid records.
5. Include NULL and empty DataFrame scenarios.
6. Verify actual output values, not only row counts.
7. Use deterministic test data.
8. Avoid unnecessary `collect()` on large datasets.
9. Use fixtures to manage SparkSession setup.
10. Separate unit tests from integration tests.
11. Test important business rules.
12. Run tests whenever transformation logic changes.

---

## 18. Common Mistakes

1. Testing only whether the code executes without checking the output.
2. Using large datasets for unit tests.
3. Depending on production data.
4. Ignoring NULL values and edge cases.
5. Comparing DataFrames without considering row ordering.
6. Assuming `count()` alone proves the output is correct.
7. Creating a new SparkSession for every test unnecessarily.
8. Forgetting to stop the SparkSession.
9. Testing only valid inputs.
10. Ignoring schema and data type differences.

---

## 19. Interview Questions

**Q1. What is unit testing in PySpark?**

Unit testing verifies individual PySpark functions and transformations using controlled test data to ensure they produce the expected output.

**Q2. Which testing frameworks are commonly used with PySpark?**

Pytest and Python's built-in unittest framework. Chispa is also useful for comparing PySpark DataFrames.

**Q3. How do you test a PySpark transformation?**

I create a small input DataFrame, pass it to the transformation function, create the expected output, and compare the actual and expected results.

**Q4. Why do we use fixtures in Pytest?**

Fixtures provide reusable setup and cleanup logic. For PySpark, they are commonly used to create and stop a SparkSession.

**Q5. How do you test NULL handling?**

I create test records containing NULL values, execute the transformation, and verify that the output follows the expected null-handling logic.

**Q6. What is the difference between unit and integration testing?**

Unit testing checks individual components, whereas integration testing checks whether multiple components work correctly together.

**Q7. What is Chispa?**

Chispa is a Python library that provides assertion utilities for comparing PySpark DataFrames.

**Q8. Why should we use explicit schemas in unit tests?**

Explicit schemas make data types and nullability predictable and prevent unexpected schema inference behavior.

**Q9. How do you test duplicate handling?**

I create a DataFrame containing duplicate business keys, apply the deduplication logic, and verify that the resulting records meet the expected rules.

**Q10. How do you test exception handling?**

In Pytest, I use `pytest.raises()` to verify that a function raises the expected exception for invalid input.

**Q11. Why is testing only the row count insufficient?**

Two DataFrames can have the same number of rows but contain different values. We should also verify the actual records, columns, and schema.

**Q12. How do you run PySpark unit tests?**

I use commands such as `pytest`, `pytest -v`, or `pytest tests/test_transformations.py`.

---

## 20. Quick Revision

| Concept | Remember |
|---|---|
| Unit Testing | Tests individual components |
| Pytest | Python testing framework |
| unittest | Built-in Python testing framework |
| Fixture | Reusable test setup |
| SparkSession | Required to create PySpark DataFrames |
| Chispa | DataFrame comparison library |
| Assertion | Compares actual and expected results |
| `pytest.raises()` | Tests expected exceptions |
| Parameterization | Runs tests with multiple inputs |
| Integration Testing | Tests multiple components together |

### Important Commands

```bash
pip install pytest pyspark
pip install chispa

pytest
pytest -v
pytest tests/test_transformations.py
```

### One-Line Revision

- Unit test → Test one component.
- Pytest → Run Python tests.
- Fixture → Reusable setup.
- Assertion → Verify expected output.
- Chispa → Compare DataFrames.
- Parameterization → Test multiple inputs.
- Integration test → Test connected components.

---

## 21. Self-Check

- [ ] Can I explain unit testing?
- [ ] Can I create a SparkSession for tests?
- [ ] Can I use Pytest fixtures?
- [ ] Can I create test DataFrames?
- [ ] Can I test filtering and aggregation?
- [ ] Can I test joins and NULL handling?
- [ ] Can I compare DataFrames?
- [ ] Can I test expected exceptions?
- [ ] Can I write parameterized tests?
- [ ] Can I explain unit testing vs integration testing?
- [ ] Can I run tests using Pytest?

**Final takeaway:** PySpark unit testing verifies that individual transformations produce correct and predictable results. In data engineering, it helps protect data quality, business logic, and pipeline reliability whenever code changes.