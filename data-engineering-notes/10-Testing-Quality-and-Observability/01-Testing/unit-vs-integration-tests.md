

# Unit Testing vs Integration Testing

## 1. Introduction

Unit testing and integration testing are two important levels of software and data pipeline testing.

Both help identify defects, but they focus on different parts of an application.

- **Unit Testing:** Tests an individual function, method, or component in isolation.
- **Integration Testing:** Tests whether multiple components work correctly together.

### Simple Data Engineering Example

Suppose an e-commerce pipeline processes customer orders.

```text
Source Database
       |
       v
Data Ingestion
       |
       v
PySpark Transformation
       |
       v
Delta Lake
       |
       v
Data Warehouse
```

We can test this pipeline at different levels:

**Unit testing:**
- Does the discount calculation work?
- Does the filtering function remove invalid records?
- Does the aggregation function calculate revenue correctly?

**Integration testing:**
- Can Spark read the input file?
- Does the transformation work with the actual input schema?
- Can the pipeline write the result to a Delta table?
- Can the written table be read correctly?

---

## 2. What Is Unit Testing?

Unit testing verifies an individual piece of code independently.

A unit can be:
- A Python function.
- A class method.
- A SQL transformation.
- A PySpark transformation function.
- A validation rule.

The objective is to confirm that the unit produces the expected result for a given input.

### Example

Suppose we have an order amount and discount.

Business rule:

\[
FinalAmount = Amount - Discount
\]

Python function:

```python
def calculate_final_amount(amount, discount):
    return amount - discount
```

Unit tests:

```python
def test_calculate_final_amount():
    result = calculate_final_amount(
        1000,
        100
    )

    assert result == 900
```

Another test:

```python
def test_zero_discount():
    result = calculate_final_amount(
        500,
        0
    )

    assert result == 500
```

These tests verify the function without requiring a database, Spark cluster, or cloud storage.

### Characteristics of Unit Testing

1. Tests a small piece of code.
2. Runs independently.
3. Usually executes quickly.
4. Uses controlled input data.
5. Often uses fixtures, stubs, or mocks.
6. Helps identify the exact location of a defect.

---

## 3. What Is Integration Testing?

Integration testing verifies that multiple components work correctly together.

Instead of testing only one function, it tests the interaction between components.

In data engineering, components may include:
- File storage.
- Spark.
- Database connections.
- APIs.
- Delta Lake.
- Kafka.
- Azure Data Factory.
- External services.

### Example

Suppose a PySpark application reads a CSV file, transforms the records, and writes the output to Delta Lake.

```text
CSV File
   |
   v
Spark Read
   |
   v
Transformation
   |
   v
Delta Write
   |
   v
Delta Read
```

An integration test verifies that these components work together.

### Example Code

```python
def test_csv_to_delta_pipeline(spark, tmp_path):
    import os

    input_path = str(tmp_path / "orders.csv")
    output_path = str(tmp_path / "delta_output")

    input_data = [
        (101, 500.0),
        (102, 300.0)
    ]

    input_df = spark.createDataFrame(
        input_data,
        ["order_id", "amount"]
    )

    input_df.write.mode("overwrite").option(
        "header", True
    ).csv(input_path)

    loaded_df = (
        spark.read
        .option("header", True)
        .option("inferSchema", True)
        .csv(input_path)
    )

    loaded_df.write.format("delta").mode(
        "overwrite"
    ).save(output_path)

    result_df = (
        spark.read
        .format("delta")
        .load(output_path)
    )

    assert result_df.count() == 2
    assert set(result_df.columns) == {
        "order_id",
        "amount"
    }
```

This test verifies:
- Spark can write the input data.
- Spark can read the CSV.
- Spark can write the Delta output.
- Spark can read the Delta output.

It requires a working Spark environment with Delta support. The `tmp_path` fixture provides a temporary local directory.

---

## 4. Unit Testing vs Integration Testing

| Comparison | Unit Testing | Integration Testing |
|---|---|---|
| Purpose | Test individual components | Test component interactions |
| Scope | Small | Larger |
| Dependencies | Usually isolated | Uses actual integrated components |
| Execution speed | Usually fast | Usually slower |
| Test data | Small controlled inputs | Representative test inputs |
| Mocking | Common | Used selectively |
| Failure identification | Usually precise | May involve multiple components |
| Infrastructure | Often minimal | May require Spark, database, or storage |
| Frequency | Run frequently | Run at integration checkpoints |
| Main objective | Verify component logic | Verify integration behavior |

### Simple Difference

- Unit testing asks: **Does this function work correctly?**
- Integration testing asks: **Do these components work correctly together?**

---

## 5. Unit Testing in PySpark

PySpark transformations can be tested using a local Spark session and small DataFrames.

### Step 1: Create a Transformation

```python
from pyspark.sql.functions import col

def get_valid_orders(df):
    return df.filter(
        col("amount") > 0
    )
```

### Step 2: Create a Test Fixture

```python
import pytest

@pytest.fixture
def orders_df(spark):
    data = [
        (101, 500.0),
        (102, -100.0),
        (103, 300.0)
    ]

    return spark.createDataFrame(
        data,
        ["order_id", "amount"]
    )
```

### Step 3: Write a Unit Test

```python
def test_get_valid_orders(orders_df):
    result = get_valid_orders(orders_df)

    actual_ids = {
        row["order_id"]
        for row in result.collect()
    }

    assert actual_ids == {101, 103}
```

### What Does This Test Verify?

- Positive amounts are retained.
- Negative amounts are removed.
- The expected order IDs are returned.

This tests the transformation logic using a real Spark DataFrame, without requiring external storage or a production database.

---

## 6. Integration Testing in PySpark

Integration testing verifies interactions between actual components.

For example:
1. Read a test CSV file.
2. Apply a PySpark transformation.
3. Write the result to a Delta table.
4. Read the Delta table.
5. Validate the final result.

### Example

```python
def transform_orders(df):
    from pyspark.sql.functions import col

    return df.filter(
        col("amount") > 0
    )
```

Integration test:

```python
def test_orders_to_delta(spark, tmp_path):
    input_path = str(tmp_path / "input")
    output_path = str(tmp_path / "output")

    source_data = [
        (101, 500.0),
        (102, -100.0),
        (103, 300.0)
    ]

    source_df = spark.createDataFrame(
        source_data,
        ["order_id", "amount"]
    )

    source_df.write.mode("overwrite").parquet(
        input_path
    )

    loaded_df = spark.read.parquet(input_path)

    transformed_df = transform_orders(
        loaded_df
    )

    transformed_df.write.format("delta").mode(
        "overwrite"
    ).save(output_path)

    final_df = (
        spark.read
        .format("delta")
        .load(output_path)
    )

    actual_ids = {
        row["order_id"]
        for row in final_df.collect()
    }

    assert actual_ids == {101, 103}
```

This test verifies that file reading, transformation, Delta writing, and Delta reading work together.

---

## 7. Role of Mocks in Unit and Integration Testing

Mocks are simulated objects that replace real dependencies.

They are commonly used in unit testing to isolate the component being tested.

### Unit Testing Example

Suppose a function retrieves data from an API.

```python
import requests

def get_customer(customer_id):
    response = requests.get(
        f"https://api.example.com/customers/{customer_id}"
    )

    response.raise_for_status()

    return response.json()
```

We can mock the API call:

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

Replace `my_module` with the actual module containing the function.

This test verifies the function's behavior without making a real network request.

### What About Integration Testing?

For integration testing, use a real test API, database, or storage environment when the integration itself is what needs validation.

Mocks can still be used for unrelated external dependencies, but mocking the very component being tested defeats the purpose of that integration test.

---

## 8. Unit Testing vs Integration Testing in Data Engineering

Consider an ADLS-to-Delta pipeline.

```text
ADLS Gen2
    |
    v
Spark Read
    |
    v
Data Cleaning
    |
    v
Business Transformation
    |
    v
Delta Write
```

### Unit Testing

Test each transformation separately.

| Component | Test |
|---|---|
| Data Cleaning | Removes invalid records |
| Deduplication | Removes duplicates according to the defined key |
| Aggregation | Calculates correct revenue |
| Date Transformation | Converts timestamps correctly |
| Validation | Identifies missing required values |

### Integration Testing

Test interactions between components.

| Integration | Test |
|---|---|
| ADLS + Spark | Spark can read the expected files |
| Spark + Transformation | Input schema works with the transformation |
| Spark + Delta | Output can be written and read |
| Database + Pipeline | Database operations work correctly |
| Kafka + Spark | Streaming source can be consumed in the test environment |

---

## 9. Unit Testing vs Integration Testing vs End-to-End Testing

There is another important testing level: end-to-end testing.

End-to-end testing verifies the complete workflow from the source to the final business output.

| Feature | Unit | Integration | End-to-End |
|---|---|---|---|
| Scope | Individual component | Connected components | Complete workflow |
| Dependencies | Mostly isolated | Real integrated dependencies | Full application environment |
| Speed | Usually fastest | Medium | Usually slowest |
| Main focus | Logic correctness | Interaction correctness | Business workflow correctness |
| Example | Test revenue function | Test Spark-to-Delta write | Test complete ADLS-to-report pipeline |

### Example

For an e-commerce pipeline:

- Unit: Verify the discount calculation.
- Integration: Verify Spark reads Parquet and writes Delta correctly.
- End-to-end: Verify source orders eventually appear correctly in the reporting table.

---

## 10. When Should We Use Each Type?

### Use Unit Testing When:

- Developing a new transformation.
- Fixing a business logic defect.
- Testing filtering or aggregation.
- Validating NULL handling.
- Testing Python functions.
- Checking edge cases.

### Use Integration Testing When:

- Connecting Spark to storage.
- Integrating with databases.
- Reading and writing Delta tables.
- Connecting Kafka with Spark.
- Validating API interactions.
- Testing authentication and permissions.

### Use End-to-End Testing When:

- Validating a complete production-like workflow.
- Testing orchestration across multiple services.
- Verifying final business output.
- Checking important downstream dependencies.

---

## 11. Common Tools

| Tool | Usage |
|---|---|
| pytest | Python unit and integration test framework |
| unittest | Python standard testing framework |
| unittest.mock | Mocking dependencies |
| chispa | PySpark DataFrame comparison |
| Spark local mode | Testing Spark transformations locally |
| Delta Lake | Testing Delta read/write operations |
| Testcontainers | Running disposable test services in containers |
| Great Expectations | Data quality validation |
| Soda | Data quality testing and monitoring |
| Azure DevOps | Automated test execution in CI/CD |
| GitHub Actions | Automated test execution in CI/CD |

The appropriate tools depend on the pipeline architecture and available infrastructure.

---

## 12. Testing Strategy for a Data Engineering Project

A practical testing strategy combines different testing levels.

### Recommended sequence

```text
Developer Writes Code
         |
         v
    Unit Testing
         |
         v
  Integration Testing
         |
         v
  Regression Testing
         |
         v
  End-to-End Testing
         |
         v
    Deployment
         |
         v
     Smoke Test
```

### Step-by-step approach

**Step 1: Unit Testing**

Test Python and PySpark functions with controlled input.

**Step 2: Integration Testing**

Verify that storage, Spark, databases, and other required components work together.

**Step 3: Regression Testing**

Verify that new changes have not broken existing functionality.

**Step 4: End-to-End Testing**

Execute the complete workflow using a test environment.

**Step 5: Smoke Testing**

After deployment, verify that essential production functionality works.

Not every small code change needs a complete end-to-end test run. Test selection should consider the affected components and business risk.

---

## 13. Common Mistakes to Avoid

1. **Testing only individual functions:** This can miss integration problems.
2. **Testing only the complete pipeline:** It makes defect isolation difficult.
3. **Mocking the component being integrated:** This does not verify the real integration.
4. **Using production data for every test:** This can introduce security and cost risks.
5. **Ignoring test environment differences:** Local success does not guarantee cloud success.
6. **Checking only execution status:** A successful pipeline can still produce incorrect data.
7. **Ignoring failure scenarios:** Test timeouts, permission errors, and unavailable services.
8. **Making integration tests depend on each other:** Tests should be repeatable and independently manageable.
9. **Ignoring data volume:** Small test data may not reveal production-scale performance problems.
10. **Confusing integration and end-to-end testing:** Integration tests focus on component interactions; end-to-end tests cover the complete workflow.

---

## 14. Practical Interview Scenario

**Question: You have developed a PySpark pipeline that reads orders from ADLS, transforms the data, and writes it into Delta Lake. How would you perform unit and integration testing?**

### Answer

I would divide testing into two levels.

**Unit testing:**

1. Test each transformation function independently.
2. Create small DataFrames with known expected results.
3. Validate filtering, joins, aggregations, and NULL handling.
4. Test duplicate and boundary cases.
5. Use pytest for automated execution.

**Integration testing:**

1. Use a test storage location.
2. Read actual test files using Spark.
3. Execute the transformations.
4. Write the output to a test Delta table.
5. Read the Delta table again.
6. Verify the output schema, record counts, and business values.

I would also include regression and end-to-end tests before deploying significant pipeline changes.

---

## 15. Common Interview Questions and Answers

### Q1. What is unit testing?

Unit testing verifies an individual function or component independently using controlled inputs.

### Q2. What is integration testing?

Integration testing verifies that multiple components work correctly together.

### Q3. What is the main difference between unit and integration testing?

Unit testing focuses on individual component logic, while integration testing focuses on interactions between components.

### Q4. Why is unit testing faster?

It usually tests small components without requiring external infrastructure or network communication.

### Q5. Can we use mocks in unit testing?

Yes. Mocks isolate the component by simulating external dependencies.

### Q6. Should integration tests use mocks?

They can use mocks for unrelated dependencies, but the actual components whose integration is being tested should be real.

### Q7. How do you perform unit testing in PySpark?

I create a local Spark session, prepare sample DataFrames, execute transformation functions, and compare actual results with expected results.

### Q8. How do you perform integration testing in PySpark?

I use actual Spark operations to read test data, execute transformations, write to a test destination, and validate the resulting data.

### Q9. What is the difference between integration and end-to-end testing?

Integration testing checks whether connected components work together. End-to-end testing checks the complete workflow from source to final output.

### Q10. Can a unit test pass while integration testing fails?

Yes. A transformation may work correctly with sample data but fail when integrated with actual storage, schemas, permissions, or external services.

### Q11. Which testing framework is commonly used in Python?

Pytest and Python's built-in unittest framework are commonly used.

### Q12. What is the purpose of fixtures?

Fixtures provide reusable test data and resources, such as Spark sessions and sample DataFrames.

### Q13. How do you test a database integration?

Use a dedicated test database, execute actual database operations, and verify the stored results. Avoid relying entirely on mocked database calls.

### Q14. What is the testing pyramid?

The testing pyramid is a strategy that emphasizes many fast unit tests, fewer integration tests, and a smaller number of end-to-end tests.

### Q15. Why are all three testing levels important in data engineering?

Unit tests validate transformation logic, integration tests validate component interactions, and end-to-end tests validate complete business workflows.

---

## 16. Quick Revision

| Concept | Meaning |
|---|---|
| Unit Testing | Tests individual functions or components |
| Integration Testing | Tests interactions between components |
| End-to-End Testing | Tests the complete workflow |
| Mock | Simulates a dependency |
| Fixture | Provides test data and setup |
| pytest | Python testing framework |
| Spark Local Mode | Runs Spark tests locally |
| Regression Testing | Checks existing behavior after changes |
| Smoke Testing | Checks essential functionality quickly |
| Testing Pyramid | More unit tests, fewer integration and end-to-end tests |

### Remember These Five Points

1. Unit testing verifies individual components.
2. Integration testing verifies interactions between components.
3. Mocks are useful for isolating unit tests.
4. Real dependencies should be used when testing their actual integration.
5. End-to-end testing verifies the complete business workflow.

---

## 17. Self-Assessment

Try answering these questions without looking at the notes.

1. What is unit testing?
2. What is integration testing?
3. Explain the difference between unit and integration testing.
4. Why are unit tests generally faster?
5. What is the role of mocks?
6. What is the role of test fixtures?
7. How do you test a PySpark transformation?
8. How do you test Spark reading from a file?
9. How would you test a Spark-to-Delta integration?
10. What is the difference between integration and end-to-end testing?
11. Can unit tests pass while integration tests fail?
12. What is the testing pyramid?
13. Which tools are commonly used for Python and PySpark testing?
14. How would you test an ADLS-to-Delta pipeline?
15. Design a testing strategy for an e-commerce data pipeline.

**Interview readiness:** You should be able to clearly explain unit, integration, and end-to-end testing, demonstrate a PySpark example, and describe how you would test an ADLS-to-Delta pipeline.

