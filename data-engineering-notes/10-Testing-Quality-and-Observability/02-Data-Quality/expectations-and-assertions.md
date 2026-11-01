

# Data Quality Expectations and Assertions

## 1. Introduction

Data quality expectations and assertions are rules used to verify whether data meets predefined requirements.

They help ensure that data is accurate, complete, consistent, valid, and reliable before it is used for reporting, analytics, or downstream processing.

For example, consider an e-commerce orders dataset:

| order_id | customer_id | amount | order_date |
|---:|---:|---:|---|
| 101 | 501 | 500.00 | 2026-10-01 |
| 102 | 502 | 300.00 | 2026-10-02 |
| 103 | NULL | 250.00 | 2026-10-03 |
| 104 | 504 | -100.00 | 2026-10-04 |

Possible data quality rules:

- `order_id` must not be NULL.
- `customer_id` must not be NULL.
- `amount` must be greater than or equal to zero.
- `order_id` must be unique.
- `order_date` must contain a valid date.

These rules define what acceptable data looks like.

## 2. What Are Data Quality Expectations?

A data quality expectation is a predefined condition that data must satisfy.

Expectations describe the required quality of a dataset or a particular column.

### Examples

| Column | Expectation | Reason |
|---|---|---|
| order_id | Not NULL | Every order needs an identifier |
| customer_id | Not NULL | Every order must belong to a customer |
| amount | Greater than or equal to 0 | Negative order amounts are invalid |
| email | Valid email format | Prevent invalid contact information |
| order_id | Unique | Prevent duplicate order identifiers |
| order_date | Valid date | Ensure correct time-based analysis |

### Why Are Expectations Important?

1. Detect invalid records early.
2. Prevent incorrect data from reaching reporting tables.
3. Maintain consistent business rules.
4. Improve pipeline reliability.
5. Identify upstream source problems.
6. Make data quality measurable and repeatable.

## 3. What Are Assertions?

An assertion is a programmatic check that verifies whether a condition is true.

If the condition is true, the assertion passes. If it is false, the test fails or raises an error.

### Python Example

```python
amount = 500

assert amount >= 0
```

The assertion passes because the amount is non-negative.

Invalid example:

```python
amount = -100

assert amount >= 0, "Amount cannot be negative"
```

This raises an `AssertionError`.

### Difference Between Expectations and Assertions

| Feature | Expectation | Assertion |
|---|---|---|
| Meaning | Defines a data quality rule | Checks whether a condition is true |
| Purpose | Specifies acceptable data | Enforces or verifies the rule |
| Example | Amount must be non-negative | `assert amount >= 0` |
| Usage | Data quality framework or business rule | Python tests, pipeline validation, SQL checks |

**Remember:** An expectation defines what should be true. An assertion checks whether it is true.

In practice, the terms may overlap depending on the testing framework.

## 4. Common Data Quality Expectations

### 4.1 Completeness

Checks whether required fields contain values.

Example:

`customer_id` must not be NULL.

SQL:

```sql
SELECT *
FROM orders
WHERE customer_id IS NULL;
```

Expected result: zero invalid records, if every order must have a customer.

PySpark:

```python
from pyspark.sql.functions import col

invalid_records = df.filter(
    col("customer_id").isNull()
)

assert invalid_records.count() == 0
```

### 4.2 Uniqueness

Checks whether a key contains duplicate values.

SQL:

```sql
SELECT
    order_id,
    COUNT(*) AS record_count
FROM orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

PySpark:

```python
duplicate_count = (
    df.groupBy("order_id")
      .count()
      .filter(col("count") > 1)
      .count()
)

assert duplicate_count == 0
```

For composite keys, check uniqueness across all columns that together identify a record.

### 4.3 Validity

Checks whether values follow acceptable business rules or formats.

Example: order amount must be non-negative.

SQL:

```sql
SELECT *
FROM orders
WHERE amount < 0;
```

PySpark:

```python
invalid_amounts = df.filter(
    col("amount") < 0
)

assert invalid_amounts.count() == 0
```

### 4.4 Consistency

Checks whether related data follows the same business rules across tables or systems.

Example:

Every order's `customer_id` should exist in the customer table.

SQL:

```sql
SELECT o.*
FROM orders o
LEFT ANTI JOIN customers c
    ON o.customer_id = c.customer_id;
```

Expected result: no unmatched orders, if orphan orders are not allowed.

### 4.5 Accuracy

Checks whether data correctly represents the real-world value or trusted source.

Example:

The total amount in the reporting table should match the expected total from the source after applying the same business rules.

SQL:

```sql
SELECT SUM(amount) AS total_amount
FROM orders;
```

Accuracy often requires comparison with a trusted source, reference dataset, or independently calculated result. A value being within a valid range does not prove that it is accurate.

### 4.6 Referential Integrity

Checks whether foreign-key values refer to existing records in a related table.

Example:

Every `customer_id` in orders must exist in customers.

```sql
SELECT COUNT(*) AS invalid_orders
FROM orders o
LEFT JOIN customers c
    ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;
```

This assumes `customer_id` is a non-NULL key in the customer table.

### 4.7 Freshness

Checks whether data has been updated within the expected time.

Example:

A daily orders table should contain data up to the expected processing date.

```sql
SELECT MAX(order_date) AS latest_order_date
FROM orders;
```

A freshness rule may compare the latest available timestamp with an expected threshold.

Freshness is different from completeness: a dataset can contain all expected records but still be outdated.

## 5. Implementing Assertions in PySpark

Suppose we have an orders DataFrame.

```python
from pyspark.sql.functions import col

orders_data = [
    (101, 501, 500.0),
    (102, 502, 300.0),
    (103, 503, 250.0)
]

df = spark.createDataFrame(
    orders_data,
    ["order_id", "customer_id", "amount"]
)
```

### Check 1: Required Columns

```python
required_columns = {
    "order_id",
    "customer_id",
    "amount"
}

assert required_columns.issubset(
    set(df.columns)
)
```

This verifies that all required columns exist.

### Check 2: NULL Validation

```python
null_count = df.filter(
    col("customer_id").isNull()
).count()

assert null_count == 0
```

### Check 3: Duplicate Validation

```python
duplicate_count = (
    df.groupBy("order_id")
      .count()
      .filter(col("count") > 1)
      .count()
)

assert duplicate_count == 0
```

### Check 4: Amount Validation

```python
invalid_amount_count = df.filter(
    col("amount").isNull() |
    (col("amount") < 0)
).count()

assert invalid_amount_count == 0
```

### Check 5: Record Count Validation

```python
assert df.count() > 0
```

This verifies that the DataFrame is not empty.

However, a non-empty DataFrame does not necessarily contain all expected records.

## 6. Building Reusable Data Quality Checks

Instead of writing the same validation logic repeatedly, create reusable functions.

```python
from pyspark.sql.functions import col

def assert_not_null(df, column):
    invalid_count = df.filter(
        col(column).isNull()
    ).count()

    assert invalid_count == 0, (
        f"{column} contains NULL values"
    )


def assert_non_negative(df, column):
    invalid_count = df.filter(
        col(column).isNull() |
        (col(column) < 0)
    ).count()

    assert invalid_count == 0, (
        f"{column} contains NULL or negative values"
    )


def assert_unique(df, column):
    duplicate_count = (
        df.groupBy(column)
          .count()
          .filter(col("count") > 1)
          .count()
    )

    assert duplicate_count == 0, (
        f"{column} contains duplicate values"
    )
```

Usage:

```python
assert_not_null(df, "order_id")
assert_not_null(df, "customer_id")
assert_non_negative(df, "amount")
assert_unique(df, "order_id")
```

These functions make validation easier to reuse across multiple pipelines.

For large datasets, repeated `count()` operations can be expensive because they trigger Spark jobs. Combine checks where practical, use efficient aggregations, and avoid unnecessary full scans.

## 7. Data Quality Expectations Using SQL

Data quality rules can also be implemented directly in SQL.

### Example: Invalid Record Count

```sql
SELECT COUNT(*) AS invalid_count
FROM orders
WHERE
    order_id IS NULL
    OR customer_id IS NULL
    OR amount IS NULL
    OR amount < 0;
```

### Example: Duplicate Key Check

```sql
SELECT COUNT(*) AS duplicate_groups
FROM (
    SELECT order_id
    FROM orders
    GROUP BY order_id
    HAVING COUNT(*) > 1
) duplicates;
```

### Example: Referential Integrity Check

```sql
SELECT COUNT(*) AS orphan_orders
FROM orders o
LEFT ANTI JOIN customers c
    ON o.customer_id = c.customer_id;
```

### Example: Data Reconciliation

Compare source and target totals using the same business rules.

```sql
SELECT
    SUM(amount) AS total_amount,
    COUNT(*) AS total_records
FROM orders;
```

Run the equivalent aggregation against the source and compare the results.

A reconciliation check should account for expected filtering, deduplication, late-arriving records, and transformation rules.

## 8. Handling Failed Expectations

When a data quality check fails, the pipeline needs a defined response.

There are three common approaches.

### 8.1 Fail the Pipeline

Stop processing when critical rules fail.

Example:
- Primary key is NULL.
- Required column is missing.
- Financial amount is invalid.

```python
assert df.filter(
    col("order_id").isNull()
).count() == 0, "Invalid order IDs found"
```

Use this when invalid data could corrupt downstream results.

### 8.2 Quarantine Invalid Records

Separate invalid records from valid records.

```python
valid_df = df.filter(
    col("order_id").isNotNull() &
    col("customer_id").isNotNull() &
    col("amount").isNotNull() &
    (col("amount") >= 0)
)

invalid_df = df.filter(
    col("order_id").isNull() |
    col("customer_id").isNull() |
    col("amount").isNull() |
    (col("amount") < 0)
)
```

The invalid records can be written to a quarantine table for investigation.

### 8.3 Log and Continue

Some quality rules may be warnings rather than blockers.

Example:
- A small increase in NULL values in an optional field.
- A non-critical descriptive field has an unexpected format.

The pipeline can record the issue and continue if the business has explicitly approved that behavior.

**Important:** Do not silently ignore failed expectations. Record the rule, failure count, severity, and affected dataset.

## 9. Expectations in Data Quality Frameworks

Data quality frameworks allow teams to define and execute reusable rules.

### Great Expectations

Great Expectations is a data quality framework that supports defining expectations for datasets.

Examples of expectation types include:

- Column values must not be NULL.
- Column values must be unique.
- Column values must belong to a set.
- Column values must be between specified bounds.
- Table row count must satisfy a condition.

Illustrative example using the Great Expectations Python API:

```python
import great_expectations as gx

context = gx.get_context()
```

The exact setup for connecting a data source, creating a validation definition, and running a checkpoint depends on the installed Great Expectations version and project configuration.

### Other Tools

| Tool | Purpose |
|---|---|
| Great Expectations | Define and validate data expectations |
| Soda | Data quality checks and monitoring |
| Deequ | Data quality verification for Spark-based data |
| Pytest | Automated Python and PySpark tests |
| Databricks expectations | Enforce quality rules in supported pipeline features |
| SQL assertions | Validate data directly in database or warehouse queries |

A framework helps organize and report checks, but it does not automatically guarantee that the business rules are correct.

## 10. Expectations in Databricks and Delta Live Tables

Databricks supports data quality expectations in Lakeflow Spark Declarative Pipelines, previously known as Delta Live Tables.

Expectations can define conditions on incoming records.

Common actions include:

| Action | Behavior |
|---|---|
| `expect` | Records the expectation result and allows records to continue |
| `expect_or_drop` | Drops records that violate the expectation |
| `expect_or_fail` | Fails the pipeline update when the expectation is violated |

Illustrative PySpark pipeline code:

```python
from pyspark import pipelines as dp
from pyspark.sql.functions import col

@dp.table
@dp.expect(
    "valid_order_amount",
    "amount >= 0"
)
@dp.expect_or_drop(
    "valid_order_id",
    "order_id IS NOT NULL"
)
def cleaned_orders():
    return spark.readStream.table(
        "raw_orders"
    )
```

The example assumes a supported Lakeflow Spark Declarative Pipelines environment and an existing `raw_orders` table.

### Important Difference

- `expect`: tracks violations without dropping those records.
- `expect_or_drop`: removes violating records from the resulting dataset.
- `expect_or_fail`: stops the pipeline update when the expectation fails.

Choose the behavior based on business criticality and data handling requirements.

## 11. Data Quality in a Medallion Architecture

Expectations can be applied at different layers of a data pipeline.

```text
              Source Data
                   |
                   v
             Bronze Layer
          Basic ingestion checks
                   |
                   v
             Silver Layer
        Validation and cleansing
                   |
                   v
              Gold Layer
        Business-rule validation
                   |
                   v
          Reports and Analytics
```

### Bronze Layer

Focus:
- File availability.
- Schema presence.
- Ingestion timestamp.
- Basic metadata.
- Source record count.

### Silver Layer

Focus:
- NULL checks.
- Duplicate checks.
- Data type and format validation.
- Business-rule validation.
- Referential integrity.

### Gold Layer

Focus:
- Aggregation correctness.
- KPI validation.
- Reconciliation.
- Business constraints.
- Reporting freshness.

Not every rule belongs in only one layer. Critical checks may need to be repeated at different boundaries.

## 12. Data Quality Metrics

Data quality can be measured using metrics.

| Metric | Example |
|---|---|
| Completeness | Percentage of non-NULL customer IDs |
| Uniqueness | Percentage of unique order IDs |
| Validity | Percentage of records satisfying business rules |
| Accuracy | Difference from a trusted source |
| Freshness | Time since the latest successful update |
| Consistency | Percentage of records matching related datasets |

### Example: Completeness Percentage

\[
Completeness =
\frac{NonNullRecords}{TotalRecords}
\times 100
\]

Suppose 950 out of 1,000 records contain a customer ID.

\[
Completeness = \frac{950}{1000}\times100=95\%
\]

### Example: Invalid Record Percentage

\[
Invalid\ Percentage =
\frac{InvalidRecords}{TotalRecords}
\times 100
\]

If 20 out of 1,000 records are invalid:

\[
Invalid\ Percentage = 2\%
\]

A metric needs an agreed threshold and business interpretation. For example, 95% completeness may be acceptable for an optional field but unacceptable for a mandatory customer identifier.

## 13. Best Practices

1. Define business rules before implementing checks.
2. Validate required columns and schemas early.
3. Apply critical checks before writing trusted data.
4. Use reusable validation functions.
5. Separate critical failures from warnings.
6. Quarantine invalid records when appropriate.
7. Track failed records and rule-level metrics.
8. Make checks repeatable across pipeline runs.
9. Avoid unnecessary Spark actions and repeated full scans.
10. Monitor data quality trends over time.
11. Use representative test datasets.
12. Document the reason behind each expectation.
13. Validate both individual columns and relationships between datasets.
14. Ensure that validation does not silently remove important business records.

## 14. Common Mistakes

- Treating a non-empty dataset as a valid dataset.
- Checking only NULL values and ignoring business rules.
- Assuming a successful pipeline means data is correct.
- Checking uniqueness on the wrong column.
- Ignoring composite keys.
- Comparing source and target counts without considering transformations.
- Dropping invalid records without recording them.
- Treating every failed check as equally critical.
- Using assertions for all production monitoring without structured logging.
- Assuming schema validation guarantees valid business values.

**Production note:** Python `assert` statements are useful in tests and simple validation examples, but production data quality systems should use explicit validation logic, structured error handling, and monitoring. Python assertions can be disabled when Python runs with optimization enabled.

## 15. Practical Interview Scenario

**Question: You are processing orders from ADLS Gen2 into a Delta Lake Silver table. How would you implement data quality expectations?**

### Answer

I would define quality rules based on the business requirements.

**Step 1: Schema validation**

Verify that required columns such as `order_id`, `customer_id`, `amount`, and `order_date` exist with expected data types.

**Step 2: Completeness checks**

Verify that mandatory columns do not contain NULL values.

**Step 3: Uniqueness checks**

Verify that `order_id` is unique, or use the appropriate composite key if the business grain requires it.

**Step 4: Validity checks**

Verify that amounts are non-negative and dates are valid.

**Step 5: Referential integrity**

Check whether each customer ID exists in the customer dimension, where that relationship is required.

**Step 6: Handle invalid records**

Separate invalid records into a quarantine location and retain information about the failed rules.

**Step 7: Monitor quality**

Record total records, valid records, invalid records, and rule-level failure counts.

**Step 8: Load trusted data**

Write validated records into the Silver Delta table.

This approach helps prevent invalid data from reaching downstream Gold tables and reports.

## 16. Common Interview Questions

### Q1. What are data quality expectations?

They are predefined rules that describe the acceptable quality of data.

### Q2. What is an assertion?

An assertion checks whether a condition is true. It raises an error when the condition fails.

### Q3. What is the difference between an expectation and an assertion?

An expectation defines a data quality requirement, while an assertion programmatically verifies a condition.

### Q4. What are the main dimensions of data quality?

Completeness, uniqueness, validity, accuracy, consistency, and freshness are common dimensions.

### Q5. How do you check duplicate records in PySpark?

Group records by the key, count them, and identify groups with more than one record.

### Q6. How do you handle invalid records?

Depending on business requirements, fail the pipeline, quarantine invalid records, or log a warning and continue.

### Q7. What is the difference between completeness and accuracy?

Completeness checks whether required data is present. Accuracy checks whether the data correctly represents the real-world value.

### Q8. What is referential integrity?

It ensures that foreign-key values refer to valid records in the related dataset.

### Q9. What are Great Expectations and Soda?

They are data quality tools that help define, execute, and monitor validation rules.

### Q10. What are Databricks expectations?

They are data quality rules supported in Lakeflow Spark Declarative Pipelines to monitor, drop, or fail on records that violate conditions.

### Q11. What is a quarantine table?

A separate destination used to store invalid records for investigation and possible correction.

### Q12. How do you monitor data quality?

Track metrics such as NULL counts, duplicate counts, invalid records, freshness, and reconciliation differences.

### Q13. Does schema validation guarantee data quality?

No. A dataset can have the correct schema but still contain invalid, incomplete, or inconsistent values.

### Q14. Why should we not rely only on row counts?

Two datasets can have the same number of rows but contain different records or incorrect values.

### Q15. Where should data quality checks be applied in a medallion architecture?

Basic ingestion checks can be applied in Bronze, cleansing and business validation in Silver, and aggregation and KPI checks in Gold.

## 17. Quick Revision

| Concept | Meaning |
|---|---|
| Expectation | Defines an acceptable data condition |
| Assertion | Checks whether a condition is true |
| Completeness | Required values are present |
| Uniqueness | Key values are not duplicated |
| Validity | Values follow defined rules |
| Accuracy | Values represent reality correctly |
| Consistency | Related data follows compatible rules |
| Referential Integrity | Relationships between datasets are valid |
| Freshness | Data is updated within the expected time |
| Quarantine | Stores invalid records separately |
| Great Expectations | Data quality validation framework |
| Databricks Expectations | Pipeline-level data quality rules |

### Remember These Five Points

1. Expectations define what good data should look like.
2. Assertions verify whether data satisfies those expectations.
3. Data quality includes more than NULL and duplicate checks.
4. Invalid data should be handled according to business criticality.
5. Quality checks should be monitored and automated throughout the pipeline.

## 18. Self-Assessment

Try answering these questions without looking at the notes.

1. What are data quality expectations?
2. What is an assertion?
3. Explain the difference between expectations and assertions.
4. What are the six common dimensions of data quality?
5. How do you check NULL values in PySpark?
6. How do you identify duplicate records?
7. What is referential integrity?
8. How is data freshness measured?
9. What is the difference between completeness and accuracy?
10. How do you handle invalid records in a pipeline?
11. What is a quarantine table?
12. What is Great Expectations?
13. What are Databricks expectations?
14. Explain `expect`, `expect_or_drop`, and `expect_or_fail`.
15. How would you implement quality checks in a Bronze-Silver-Gold pipeline?

**Interview readiness:** Be able to define data quality expectations, write basic SQL and PySpark checks, explain how failed records are handled, and describe how you would enforce quality in a real data pipeline.

