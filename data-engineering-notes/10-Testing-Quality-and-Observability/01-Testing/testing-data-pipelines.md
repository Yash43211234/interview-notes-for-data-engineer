

# Testing Data Pipelines

## 1. Introduction

Data pipeline testing is the process of verifying that data is ingested, transformed, validated, and delivered correctly from source systems to target systems.

A data pipeline may contain several components:

```text
Source Systems
      |
      v
Data Ingestion
      |
      v
Raw Layer
      |
      v
Data Transformation
      |
      v
Data Quality Checks
      |
      v
Curated Layer
      |
      v
Data Warehouse
      |
      v
Reports and Analytics
```

Testing ensures that every stage works correctly and that the final output meets business requirements.

### Example

Suppose an e-commerce company has a pipeline that processes orders.

The pipeline:
1. Reads orders from a database.
2. Ingests them into ADLS Gen2.
3. Transforms them using PySpark.
4. Stores the results in Delta Lake.
5. Makes the data available for reporting.

Testing should verify:
- All expected records are ingested.
- Invalid records are handled correctly.
- Transformations produce correct values.
- Duplicate orders are handled.
- The final schema is correct.
- Revenue calculations match business rules.
- The pipeline can recover from failures.

**Key idea:** Pipeline testing verifies both the correctness of individual transformations and the reliability of the complete data flow.

---

## 2. Why Is Data Pipeline Testing Important?

Without proper testing, a pipeline may execute successfully but still produce incorrect data.

For example:
- A join may accidentally remove valid customers.
- A filter may exclude important records.
- An aggregation may double-count revenue.
- A schema change may break downstream queries.
- A retry may insert duplicate records.

These problems can affect business decisions and reports.

### Main benefits

1. Improves data accuracy.
2. Detects transformation errors.
3. Prevents duplicate and missing records.
4. Protects downstream systems.
5. Reduces production failures.
6. Supports safe deployments.
7. Improves pipeline reliability.
8. Builds confidence in business metrics.

---

## 3. Different Levels of Data Pipeline Testing

Testing should happen at multiple levels rather than only after the complete pipeline runs.

### A. Unit Testing

Tests an individual function or transformation independently.

Example: Test a function that calculates order revenue.

```python
from pyspark.sql.functions import col

def calculate_final_amount(df):
    return df.withColumn(
        "final_amount",
        col("amount") - col("discount")
    )
```

The test verifies whether the calculation is correct for known input records.

### B. Integration Testing

Tests whether multiple components work together correctly.

Example:
- Read data from a test file.
- Transform it using Spark.
- Write the result to a test Delta table.
- Read the table and verify the output.

### C. End-to-End Testing

Tests the complete pipeline from source to final destination.

Example:

```text
Source Database
      |
      v
ADF Pipeline
      |
      v
ADLS Gen2
      |
      v
Databricks
      |
      v
Delta Lake
      |
      v
Data Warehouse
```

The goal is to verify that the complete workflow produces the expected business output.

### D. Regression Testing

Checks whether a new code or configuration change has broken previously working behavior.

Example:
A developer modifies a PySpark join. Regression testing verifies that existing customer and order results remain correct.

### E. Smoke Testing

Performs a quick set of checks to confirm that critical functionality works.

Example:
- Pipeline starts successfully.
- Source is accessible.
- Output table exists.
- A basic query returns results.

### Comparison

| Testing Type | Main Purpose |
|---|---|
| Unit | Test individual functions |
| Integration | Test component interactions |
| End-to-End | Test the complete pipeline |
| Regression | Protect existing behavior after changes |
| Smoke | Quickly verify essential functionality |

These testing types complement one another.

---

## 4. What Should We Test in a Data Pipeline?

A data pipeline should be tested across several dimensions.

| Testing Area | What to Verify |
|---|---|
| Schema | Column names, types, and structure |
| Completeness | Expected records are present |
| Accuracy | Values and calculations are correct |
| Uniqueness | Business keys meet uniqueness rules |
| Validity | Values follow business rules |
| Referential Integrity | Related records have valid references |
| Transformation | Filters, joins, and aggregations work |
| Incremental Loading | Only intended changes are processed |
| Idempotency | Reruns do not create unintended effects |
| Performance | Processing meets expected limits |
| Recovery | Pipeline handles failures correctly |
| Compatibility | Downstream consumers continue working |

---

## 5. Schema Testing

Schema testing verifies that the structure of a dataset matches the expected structure.

Important checks include:
- Column names.
- Data types.
- Nullability requirements.
- Required columns.
- Unexpected columns.
- Schema compatibility.

### Example

Expected schema:

| Column | Data Type |
|---|---|
| order_id | Integer |
| customer_id | Integer |
| amount | Double |
| order_date | Timestamp |

PySpark validation:

```python
expected_columns = [
    "order_id",
    "customer_id",
    "amount",
    "order_date"
]

assert df.columns == expected_columns
```

For complete schema comparison:

```python
assert df.schema == expected_schema
```

**Important:** Schema nullability metadata does not automatically validate that incoming records contain no NULL values. Check actual data separately.

---

## 6. Data Completeness Testing

Completeness testing verifies whether expected records and values are available.

### Example

Suppose the source system contains 10,000 orders, but the target table contains only 9,800.

There may be 200 missing records.

### Row count comparison

```python
source_count = source_df.count()
target_count = target_df.count()

assert source_count == target_count
```

This is useful when the source and target are expected to contain exactly the same records.

However, row-count equality alone does not prove that the records match.

### Missing-record comparison

```python
missing_records = source_df.join(
    target_df,
    on="order_id",
    how="left_anti"
)
```

The `left_anti` join returns source records that do not have a matching target record.

Check:

```python
assert missing_records.count() == 0
```

This assumes `order_id` uniquely identifies records and both datasets represent the same expected scope.

---

## 7. Data Accuracy Testing

Accuracy testing checks whether values are correct.

Example business rule:

\[
FinalAmount = Amount - Discount
\]

Input:

| Order | Amount | Discount |
|---|---:|---:|
| 101 | 1000 | 100 |
| 102 | 500 | 50 |

Expected output:

| Order | Final Amount |
|---|---:|
| 101 | 900 |
| 102 | 450 |

PySpark:

```python
from pyspark.sql.functions import col

result = df.withColumn(
    "final_amount",
    col("amount") - col("discount")
)
```

A test should compare the actual output against the expected result.

For monetary calculations, use an appropriate decimal data type and precision rather than relying on floating-point equality.

---

## 8. Data Uniqueness Testing

Uniqueness testing verifies whether records meet the expected duplicate rules.

Example:

Every order should have a unique `order_id`.

### PySpark

```python
duplicate_orders = (
    df.groupBy("order_id")
    .count()
    .filter("count > 1")
)

assert duplicate_orders.count() == 0
```

### SQL

```sql
SELECT
    order_id,
    COUNT(*) AS record_count
FROM orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

If the query returns records, duplicate order IDs exist.

**Important:** Not every dataset requires a unique individual column. Some datasets use composite keys, such as `(order_id, order_line_id)`.

---

## 9. Data Validity Testing

Validity testing checks whether values satisfy defined rules.

Examples:

| Column | Validation Rule |
|---|---|
| order_id | Must not be NULL |
| amount | Must be greater than or equal to zero |
| email | Must follow an accepted format |
| status | Must be an approved status |
| order_date | Must be a valid timestamp |
| quantity | Must be greater than zero |

### PySpark example

```python
from pyspark.sql.functions import col

invalid_records = df.filter(
    col("order_id").isNull() |
    (col("amount") < 0) |
    (col("quantity") <= 0)
)
```

Check:

```python
assert invalid_records.count() == 0
```

For production pipelines, it is often better to separate invalid records into a quarantine or error dataset instead of simply terminating every pipeline.

The correct behavior depends on business requirements.

---

## 10. Referential Integrity Testing

Referential integrity verifies that relationships between datasets are valid.

Example:

- Customers table contains customer records.
- Orders table contains customer IDs.
- Every order should reference an existing customer.

### PySpark

```python
invalid_orders = orders.join(
    customers.select("customer_id"),
    on="customer_id",
    how="left_anti"
)
```

Check:

```python
assert invalid_orders.count() == 0
```

This identifies orders whose customer IDs are missing from the customer dataset.

### Why is this important?

Incorrect relationships can cause:
- Missing customer information.
- Incorrect reports.
- Inaccurate customer counts.
- Incorrect revenue attribution.

---

## 11. Testing Transformations

Transformation testing verifies the logic applied to source data.

Common transformations include:
- Filtering.
- Joins.
- Aggregations.
- Deduplication.
- Date conversions.
- NULL handling.
- Window functions.
- Business calculations.

### Example: Active Customer Filtering

```python
from pyspark.sql.functions import col

def get_active_customers(df):
    return df.filter(
        col("status") == "ACTIVE"
    )
```

Test:

```python
def test_active_customers(customer_df):
    result = get_active_customers(customer_df)

    invalid_count = (
        result
        .filter(col("status") != "ACTIVE")
        .count()
    )

    assert invalid_count == 0
```

Good transformation tests should include:
- Normal records.
- NULL values.
- Duplicate records.
- Empty datasets.
- Boundary values.
- Unexpected input values.

---

## 12. Testing Incremental Data Loads

Incremental loading processes only new or changed records instead of reprocessing the complete dataset.

A pipeline might use:
- Timestamp watermarks.
- Increasing IDs.
- CDC.
- Change Tracking.
- Source-system change logs.

### What should be tested?

1. New records are inserted.
2. Updated records are handled.
3. Unchanged records are not unnecessarily duplicated.
4. The watermark advances correctly.
5. Failed runs can be safely retried.
6. Late-arriving changes are handled.
7. Deleted records follow the defined deletion strategy.

### Example

Initial target:

| order_id | amount |
|---|---:|
| 101 | 500 |
| 102 | 300 |

New source data:

| order_id | amount |
|---|---:|
| 102 | 350 |
| 103 | 700 |

Expected target after incremental processing:

| order_id | amount |
|---|---:|
| 101 | 500 |
| 102 | 350 |
| 103 | 700 |

The test should verify:
- Order 101 remains unchanged.
- Order 102 is updated.
- Order 103 is inserted.
- No duplicate order IDs are created.

---

## 13. Testing Idempotency

Idempotency means that repeating an operation produces the same intended final effect under the defined conditions.

This is important because data pipelines can be retried after failures.

### Example

Suppose a pipeline inserts 1,000 orders into a target table.

If the same batch runs again, it should not create another 1,000 duplicate orders.

### Testing approach

1. Run the pipeline once.
2. Record the target output.
3. Run the same batch again.
4. Compare the final target state.
5. Verify that no unintended duplicates or changes were introduced.

### Delta MERGE example

```sql
MERGE INTO target_orders AS target
USING source_orders AS source
ON target.order_id = source.order_id

WHEN MATCHED THEN
    UPDATE SET
        target.amount = source.amount

WHEN NOT MATCHED THEN
    INSERT (
        order_id,
        amount
    )
    VALUES (
        source.order_id,
        source.amount
    );
```

The source should be deduplicated so that each target key has an unambiguous source row.

**Important:** MERGE does not automatically make every pipeline idempotent. The complete design must account for source duplicates, deletion behavior, batch boundaries, and concurrent writes.

---

## 14. Testing Failure Recovery

A reliable pipeline should handle failures without corrupting data.

Possible failure scenarios:

- Source file is missing.
- API is unavailable.
- Database connection fails.
- Spark job fails.
- Storage write is interrupted.
- Pipeline is stopped midway.
- Network connection is lost.

### What should we test?

| Failure | Expected Behavior |
|---|---|
| Missing input file | Fail clearly or follow the defined waiting policy |
| API timeout | Retry or report a controlled failure |
| Invalid records | Reject, quarantine, or handle according to rules |
| Database unavailable | Retry safely or fail with useful diagnostics |
| Spark job failure | Recover from checkpoint or rerun safely |
| Partial write | Avoid exposing incomplete or inconsistent output |

### Important distinction

A successful retry does not automatically guarantee exactly-once processing.

Recovery depends on:
- Source replay capability.
- Checkpoint design.
- Sink transaction behavior.
- Idempotent writes.
- Correct batch boundaries.

---

## 15. Testing Performance and Scalability

A pipeline may produce correct results but still perform poorly.

Performance testing checks whether the pipeline meets expected execution requirements.

Important metrics include:
- Execution time.
- Input rows per second.
- Processed rows per second.
- Shuffle volume.
- Memory usage.
- Disk spill.
- Number and size of output files.
- Executor failures.

### Example

Suppose a pipeline normally processes 10 million records in 20 minutes.

After a code change, it takes 50 minutes.

The output may be correct, but the change has introduced a performance regression.

### What should be tested?

1. Run representative data volumes.
2. Compare execution times.
3. Inspect Spark execution plans.
4. Review shuffle and spill metrics.
5. Check data skew.
6. Verify output file sizes.
7. Test concurrency where required.

**Important:** Performance thresholds should be based on workload requirements and a controlled test environment. A single execution time is not enough to establish a reliable baseline.

---

## 16. Test Fixtures and Mocks

Fixtures and mocks help isolate pipeline components.

### Fixtures

Provide reusable test data and resources.

Examples:
- Sample DataFrames.
- Temporary CSV files.
- Local Spark sessions.
- Test configurations.

### Mocks

Simulate external dependencies.

Examples:
- ADLS read operations.
- API calls.
- Database writers.
- Notification services.

### Example

```python
import pytest

@pytest.fixture
def orders_df(spark):
    data = [
        (101, 500.0),
        (102, 300.0),
        (103, 700.0)
    ]

    return spark.createDataFrame(
        data,
        ["order_id", "amount"]
    )
```

Test:

```python
def test_total_order_amount(orders_df):
    from pyspark.sql.functions import sum

    total = orders_df.agg(
        sum("amount")
    ).first()[0]

    assert total == 1500.0
```

Use fixtures for predictable input and mocks for external service interactions.

---

## 17. Testing Data Pipelines in Azure

A typical Azure data engineering architecture may use:

```text
Source Database / API
          |
          v
Azure Data Factory
          |
          v
ADLS Gen2 - Raw
          |
          v
Azure Databricks
          |
          v
ADLS Gen2 - Curated
          |
          v
Synapse / SQL
          |
          v
Power BI
```

Testing should cover each component.

### A. Azure Data Factory

Test:
- Pipeline parameters.
- Activity dependencies.
- Copy Activity record counts.
- Source and sink mappings.
- Retry behavior.
- Failure handling.
- Incremental load logic.

### B. ADLS Gen2

Test:
- File availability.
- File format.
- File schema.
- Permissions.
- Expected file counts.
- Partition directory structure.

### C. Azure Databricks

Test:
- PySpark transformations.
- Schema validation.
- Business rules.
- Data quality.
- Delta writes.
- MERGE behavior.
- Recovery and checkpointing.

### D. Synapse or SQL

Test:
- SQL transformations.
- Joins and aggregations.
- Data completeness.
- Query correctness.
- Downstream compatibility.

### E. End-to-End

Verify that the complete workflow produces the expected final business result.

---

## 18. Automating Pipeline Tests in CI/CD

Testing should be integrated into the development and deployment process.

### Typical workflow

```text
Code Change
    |
    v
Git Commit
    |
    v
Pull Request
    |
    v
Code Validation
    |
    v
Unit Tests
    |
    v
Data Validation Tests
    |
    v
Regression Tests
    |
    v
Integration Tests
    |
    v
Approval
    |
    v
Deployment
    |
    v
Smoke Tests
```

### Common tools

| Tool | Purpose |
|---|---|
| pytest | Python testing framework |
| unittest.mock | Mocking dependencies |
| chispa | PySpark DataFrame comparison |
| Great Expectations | Data quality validation |
| Soda | Data quality and monitoring |
| dbt tests | Tests for transformed data models |
| Azure DevOps | CI/CD automation |
| GitHub Actions | CI/CD automation |
| Databricks Workflows | Job orchestration |

Tool selection depends on the architecture and testing requirements.

---

## 19. Practical End-to-End Testing Scenario

**Scenario:** An e-commerce company processes orders using ADF, ADLS Gen2, and Databricks.

### Pipeline

```text
Orders Database
       |
       v
ADF Copy Activity
       |
       v
ADLS Raw Layer
       |
       v
Databricks PySpark
       |
       v
Delta Curated Table
       |
       v
Reporting
```

### Testing plan

| Step | Test | Expected Result |
|---|---|---|
| 1 | Source count validation | Expected source records available |
| 2 | File validation | Correct files arrive in raw storage |
| 3 | Schema validation | Expected columns and types |
| 4 | NULL validation | Required fields are populated |
| 5 | Duplicate validation | Business keys meet uniqueness rules |
| 6 | Transformation validation | Business calculations are correct |
| 7 | Referential integrity | Every order references a valid customer |
| 8 | Incremental load validation | New and changed records are processed |
| 9 | Idempotency test | Rerun does not create unintended duplicates |
| 10 | Failure recovery test | Failed execution can recover safely |
| 11 | Performance test | Processing meets agreed requirements |
| 12 | End-to-end validation | Final reporting data is correct |

### Example business checks

- Every order must have an order ID.
- Amount must not be negative.
- Every customer ID must exist.
- Completed-order revenue must match the business calculation.
- Reprocessing the same batch must not duplicate orders.

---

## 20. Best Practices

1. Test every important transformation independently.
2. Use small, representative test datasets.
3. Validate schema at ingestion boundaries.
4. Test completeness, accuracy, uniqueness, and validity.
5. Test joins and aggregations carefully.
6. Include NULL, duplicate, and boundary cases.
7. Test incremental loading and deletion behavior.
8. Verify idempotency before enabling automatic retries.
9. Test failure and recovery scenarios.
10. Compare old and new outputs after code changes.
11. Automate tests through CI/CD.
12. Use mocks for external dependencies and real infrastructure for integration tests.
13. Monitor both pipeline execution and data quality.
14. Protect sensitive data in test environments.
15. Keep test results and failure details available for investigation.

---

## 21. Common Mistakes to Avoid

1. **Testing only whether the pipeline succeeds:** Successful execution does not guarantee correct data.
2. **Checking only row counts:** Different datasets can have identical counts.
3. **Ignoring business rules:** Technical validation alone cannot verify business correctness.
4. **Testing only individual functions:** Integration and end-to-end tests are also required.
5. **Ignoring incremental loads:** New and updated records may be processed incorrectly.
6. **Ignoring duplicate records:** Retries and source duplication can create duplicate output.
7. **Assuming retries guarantee correctness:** Safe retries require appropriate recovery and idempotency design.
8. **Mocking every dependency:** This can hide real integration problems.
9. **Ignoring downstream consumers:** A change may break reporting even when the transformation itself works.
10. **Ignoring performance:** Correct output can still arrive too late for business requirements.

---

## 22. Common Interview Questions and Answers

### Q1. What is data pipeline testing?

It verifies that data is correctly ingested, transformed, validated, and delivered to target systems.

### Q2. What are the different levels of pipeline testing?

Unit testing, integration testing, end-to-end testing, regression testing, and smoke testing.

### Q3. How do you validate data completeness?

I compare expected source and target records, validate record counts, and identify missing records using appropriate keys.

### Q4. How do you validate data accuracy?

I verify transformation logic, business calculations, and important values against expected results.

### Q5. How do you identify duplicate records?

I group records by their business key and check whether any key appears more times than allowed.

### Q6. What is referential integrity testing?

It verifies that relationships between datasets are valid, such as ensuring every order references an existing customer.

### Q7. How do you test an incremental pipeline?

I test inserts, updates, unchanged records, late-arriving changes, deletion behavior, watermark advancement, and safe retries.

### Q8. How do you test idempotency?

I run the same input batch more than once and verify that the final target state contains no unintended duplicate or incorrect changes.

### Q9. How do you test a PySpark transformation?

I create sample DataFrames, execute the transformation, and compare the actual output with expected results. I also test NULLs, duplicates, and boundary conditions.

### Q10. What is the difference between unit and integration testing?

Unit testing checks an individual component. Integration testing verifies that multiple components work together correctly.

### Q11. Why are end-to-end tests important?

They verify the complete pipeline and detect problems that may not appear when individual components are tested separately.

### Q12. How do you test a pipeline failure?

I simulate failures such as missing files, API timeouts, database errors, and interrupted writes. Then I verify error handling, recovery, and safe reruns.

### Q13. How do you test an ADF pipeline?

I validate parameters, activity dependencies, source-to-sink mappings, record counts, retry behavior, failure paths, and final output.

### Q14. How do you test a Delta Lake pipeline?

I validate schemas, table contents, business rules, MERGE behavior, duplicate handling, and safe recovery after failures.

### Q15. What is the role of CI/CD in pipeline testing?

CI/CD automates tests whenever code changes and helps prevent incorrect pipeline changes from reaching production.

### Q16. Can a pipeline pass all unit tests and still fail in production?

Yes. Unit tests do not fully validate infrastructure, permissions, real data characteristics, external integrations, or production-scale performance.

### Q17. What is the difference between data quality testing and pipeline testing?

Data quality testing focuses on whether data satisfies defined rules. Pipeline testing covers the broader correctness and reliability of the complete data flow.

### Q18. How would you test a pipeline after a Spark version upgrade?

I would run regression tests against representative inputs, compare schemas and outputs, validate important business metrics, check library compatibility, and measure performance.

---

## 23. Quick Revision

| Concept | Meaning |
|---|---|
| Unit Testing | Tests an individual function |
| Integration Testing | Tests interactions between components |
| End-to-End Testing | Tests the complete pipeline |
| Regression Testing | Protects existing behavior after changes |
| Smoke Testing | Quickly verifies essential functionality |
| Schema Testing | Validates structure and data types |
| Completeness | Checks expected records are present |
| Accuracy | Checks values and calculations |
| Uniqueness | Checks duplicate rules |
| Validity | Checks business constraints |
| Referential Integrity | Checks relationships between datasets |
| Idempotency | Repeated execution has the same intended final effect |
| Failure Recovery | Handles failures and safe reruns |
| Performance Testing | Measures speed and resource usage |
| CI/CD Testing | Automates validation during delivery |

### Remember These Five Points

1. Test data correctness, not just pipeline execution.
2. Validate schema, completeness, accuracy, uniqueness, and business rules.
3. Test individual transformations as well as the complete pipeline.
4. Test incremental loads, retries, idempotency, and recovery.
5. Automate testing through CI/CD and monitor production data quality.

---

## 24. Self-Assessment

Try answering these questions without looking at the notes.

1. What is data pipeline testing?
2. Why is testing important in data engineering?
3. Explain unit, integration, and end-to-end testing.
4. What is regression testing?
5. How do you validate schema and data completeness?
6. Why is row-count validation insufficient?
7. How do you test data accuracy?
8. How do you identify duplicate records in PySpark?
9. What is referential integrity?
10. How do you test incremental data loading?
11. How do you verify pipeline idempotency?
12. How do you test failure recovery?
13. How do you test an ADF-to-Databricks pipeline?
14. How would you integrate testing into CI/CD?
15. Design a complete testing strategy for an e-commerce data pipeline.

**Interview readiness:** You should be able to explain the testing levels, demonstrate data validation using SQL and PySpark, and design a practical testing strategy for an ADF, Databricks, and Delta Lake pipeline.

