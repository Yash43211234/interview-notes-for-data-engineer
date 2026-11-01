# Completeness, Uniqueness, and Validity in Data Quality

## 1. Introduction

Data quality describes how accurately and reliably data meets the requirements of its intended use.

Three important data quality dimensions are:

1. **Completeness:** Are the required values present?
2. **Uniqueness:** Are duplicate records or identifiers present where they should not be?
3. **Validity:** Do values follow the expected format, type, range, and business rules?

These dimensions help data engineers identify problems before data reaches reporting systems, analytics platforms, or machine learning applications.

### Example

Consider an orders table:

| order_id | customer_id | amount | status |
|---|---|---:|---|
| 101 | C01 | 500 | Delivered |
| 102 | NULL | 700 | Shipped |
| 102 | C02 | 700 | Shipped |
| 104 | C04 | -100 | Delivered |

Potential issues:

- **Completeness:** Record 102 has a missing `customer_id`.
- **Uniqueness:** `order_id` 102 appears more than once.
- **Validity:** Order 104 has a negative amount, which may violate the business rule.

**Key idea:** Completeness checks missing data, uniqueness checks duplicates, and validity checks whether data follows defined rules.

---

## 2. Completeness

Completeness measures whether required data is present.

A dataset is incomplete when important fields or expected records are missing.

### Examples

- Customer email is NULL.
- Order ID is missing.
- Employee department is not populated.
- Expected daily source files have not arrived.
- A transaction is missing from the target table.

### 2.1 Column-Level Completeness

Checks whether required columns contain missing values.

Example:

| customer_id | customer_name | email |
|---|---|---|
| C01 | Rahul | rahul@example.com |
| C02 | Priya | NULL |
| C03 | Amit | amit@example.com |

The `email` column has one missing value.

### SQL Example

```sql
SELECT
    COUNT(*) AS total_records,
    COUNT(email) AS populated_emails,
    COUNT(*) - COUNT(email) AS missing_emails
FROM customers;
```

Remember:
- `COUNT(*)` counts all rows.
- `COUNT(column)` counts non-NULL values.

### 2.2 Completeness Percentage

\[
\text{Completeness \%} =
\frac{\text{Non-NULL required values}}
{\text{Total expected values}}
\times 100
\]

Example:

- Total customer records = 1,000
- Records with email = 950

\[
\text{Completeness} =
\frac{950}{1000}\times100=95\%
\]

<badge color="success">95% completeness</badge>

### 2.3 PySpark Example

```python
from pyspark.sql import functions as F

df.select(
    F.count("*").alias("total_records"),
    F.count("email").alias("populated_emails"),
    (
        F.count("*") - F.count("email")
    ).alias("missing_emails")
).show()
```

### 2.4 Required-Field Validation

```python
invalid_df = df.filter(
    F.col("customer_id").isNull()
)

valid_df = df.filter(
    F.col("customer_id").isNotNull()
)
```

### How to Improve Completeness

- Validate required fields during ingestion.
- Use appropriate source-system constraints.
- Monitor missing-value percentages.
- Quarantine records with missing mandatory fields.
- Use default values only when business rules permit.
- Investigate missing source files or incomplete extracts.

### Important Point

NULL does not always mean bad data.

For example, a customer's middle name may legitimately be NULL. Completeness rules should be based on business requirements.

---

## 3. Uniqueness

Uniqueness checks whether records or values that should be unique are duplicated.

Duplicates can cause:
- Incorrect aggregations.
- Inflated sales figures.
- Double counting.
- Incorrect customer counts.
- Duplicate transactions.
- Inconsistent reporting.

### Example

| order_id | customer_id | amount |
|---|---|---:|
| 101 | C01 | 500 |
| 102 | C02 | 700 |
| 102 | C02 | 700 |
| 103 | C03 | 300 |

Here, `order_id` 102 appears twice.

If `order_id` is expected to be unique, this is a data quality issue.

### 3.1 SQL Duplicate Detection

```sql
SELECT
    order_id,
    COUNT(*) AS record_count
FROM orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

This identifies order IDs that occur more than once.

### 3.2 PySpark Duplicate Detection

```python
from pyspark.sql import functions as F

duplicates_df = (
    df.groupBy("order_id")
    .count()
    .filter(F.col("count") > 1)
)

duplicates_df.show()
```

### 3.3 Composite-Key Uniqueness

Sometimes a single column is not enough to identify a record.

For example, a product sales table may require a unique combination of:

- `order_id`
- `product_id`

SQL:

```sql
SELECT
    order_id,
    product_id,
    COUNT(*) AS record_count
FROM order_items
GROUP BY
    order_id,
    product_id
HAVING COUNT(*) > 1;
```

PySpark:

```python
duplicates_df = (
    df.groupBy("order_id", "product_id")
    .count()
    .filter(F.col("count") > 1)
)
```

### 3.4 Remove Duplicates

PySpark provides `dropDuplicates()`.

```python
deduplicated_df = df.dropDuplicates(
    ["order_id"]
)
```

This removes duplicate keys, but it does not guarantee which record is retained.

If records have different values, use a deterministic rule to select the correct record.

### 3.5 Keep the Latest Record

Suppose multiple records exist for the same order, and the latest update should be retained.

```python
from pyspark.sql import Window
from pyspark.sql import functions as F

window_spec = (
    Window
    .partitionBy("order_id")
    .orderBy(
        F.col("updated_at").desc(),
        F.col("source_sequence").desc()
    )
)

latest_df = (
    df
    .withColumn(
        "row_num",
        F.row_number().over(window_spec)
    )
    .filter(F.col("row_num") == 1)
    .drop("row_num")
)
```

The `source_sequence` column acts as a tie-breaker. Use a reliable ordering column so the selected record is deterministic.

### How to Improve Uniqueness

- Define primary and business keys.
- Identify composite keys where needed.
- Deduplicate incoming records.
- Use deterministic latest-record selection.
- Apply suitable database constraints.
- Use idempotent ingestion and write strategies.
- Monitor duplicate rates.

### Important Point

Not every repeated value is a duplicate.

For example, many orders can have the same `customer_id`. Uniqueness must be checked against the correct business key.

---

## 4. Validity

Validity checks whether data follows expected rules, formats, types, ranges, and business constraints.

A value can be present and unique but still be invalid.

### Examples

| Field | Invalid Value | Expected Rule |
|---|---|---|
| Age | -5 | Age cannot be negative |
| Email | `abc@` | Valid email format |
| Amount | -500 | Amount must be non-negative |
| Status | `UnknownStatus` | Must belong to allowed values |
| Order date | Invalid date text | Must be a valid date |

### 4.1 Data-Type Validation

A column expected to contain dates should not contain arbitrary text.

SQL:

```sql
SELECT *
FROM orders
WHERE order_date IS NULL;
```

This checks missing parsed dates, but it only detects conversion failures if the source value was actually parsed into a date column.

PySpark:

```python
from pyspark.sql import functions as F

typed_df = df.withColumn(
    "amount_numeric",
    F.expr("try_cast(amount AS DECIMAL(12,2))")
)

invalid_df = typed_df.filter(
    F.col("amount").isNotNull() &
    F.col("amount_numeric").isNull()
)
```

`try_cast()` returns NULL for failed conversions in supported Spark versions and configurations. Cast behavior can vary with Spark version and ANSI settings.

### 4.2 Range Validation

Example: An order amount must not be negative.

SQL:

```sql
SELECT *
FROM orders
WHERE amount < 0;
```

PySpark:

```python
invalid_df = df.filter(
    F.col("amount") < 0
)
```

### 4.3 Format Validation

Example: Validate a simple email pattern.

SQL:

```sql
SELECT *
FROM customers
WHERE email NOT LIKE '%_@_%._%';
```

This is only a basic pattern check, not complete email validation.

PySpark:

```python
invalid_df = df.filter(
    ~F.col("email").rlike(
        r"^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$"
    )
)
```

This regular expression is a practical screening rule, not a complete implementation of every valid email-address format.

### 4.4 Allowed-Value Validation

Suppose order status can only be:

- Pending
- Shipped
- Delivered
- Cancelled

SQL:

```sql
SELECT *
FROM orders
WHERE status NOT IN (
    'Pending',
    'Shipped',
    'Delivered',
    'Cancelled'
);
```

PySpark:

```python
valid_statuses = [
    "Pending",
    "Shipped",
    "Delivered",
    "Cancelled"
]

invalid_df = df.filter(
    ~F.col("status").isin(valid_statuses)
)
```

If NULL values are possible, validate them separately because SQL and Spark use three-valued NULL logic.

### 4.5 Business-Rule Validation

Some rules depend on multiple columns.

Example:

A delivered order must have a delivery date.

```python
invalid_df = df.filter(
    (F.col("status") == "Delivered") &
    F.col("delivery_date").isNull()
)
```

### How to Improve Validity

- Define expected data types.
- Validate formats and ranges.
- Apply allowed-value checks.
- Validate business rules.
- Reject or quarantine invalid records.
- Monitor invalid-record rates.
- Maintain clear validation rules.

---

## 5. Completeness vs Uniqueness vs Validity

| Dimension | Main Question | Example |
|---|---|---|
| Completeness | Is required data present? | Customer ID is NULL |
| Uniqueness | Is a supposedly unique record duplicated? | Duplicate order ID |
| Validity | Does the value follow the expected rules? | Negative order amount |

### One Example

Consider this record:

| order_id | customer_id | amount |
|---|---|---:|
| 101 | NULL | -500 |

Potential problems:

- Completeness: `customer_id` is missing.
- Validity: `amount` is negative if the business rule prohibits negative orders.
- Uniqueness: Cannot be determined from this single row; the key must be checked across the dataset.

A single record can violate multiple data quality dimensions.

---

## 6. Combining All Three Checks in PySpark

Suppose an orders dataset has the following requirements:

- `order_id` must be present.
- `customer_id` must be present.
- `amount` must be non-negative.
- `status` must be from the allowed list.
- `order_id` must be unique.

```python
from pyspark.sql import functions as F

valid_statuses = [
    "Pending",
    "Shipped",
    "Delivered",
    "Cancelled"
]

# 1. Completeness
complete_df = df.filter(
    F.col("order_id").isNotNull() &
    F.col("customer_id").isNotNull() &
    F.col("amount").isNotNull()
)

# 2. Validity
valid_df = complete_df.filter(
    (F.col("amount") >= 0) &
    F.col("status").isin(valid_statuses)
)

# 3. Identify duplicate order IDs
duplicate_keys = (
    valid_df
    .groupBy("order_id")
    .count()
    .filter(F.col("count") > 1)
    .select("order_id")
)

# 4. Keep only records with unique order IDs
unique_df = valid_df.join(
    duplicate_keys,
    on="order_id",
    how="left_anti"
)
```

This example:
1. Removes records missing mandatory values.
2. Filters records that violate the stated validity rules.
3. Identifies duplicate order IDs.
4. Excludes all records associated with duplicate order IDs.

**Important:** Excluding every record with a duplicate key is one possible business policy. Another pipeline may retain the latest valid record or quarantine duplicates for investigation.

---

## 7. Measuring Data Quality

Data quality should be measured rather than checked only through manual inspection.

### 7.1 Completeness Rate

\[
\text{Completeness \%} =
\frac{\text{Populated required values}}
{\text{Total expected values}}
\times 100
\]

### 7.2 Uniqueness Rate

For a dataset where each row should have a unique key:

\[
\text{Uniqueness \%} =
\frac{\text{Distinct non-NULL keys}}
{\text{Total non-NULL keys}}
\times 100
\]

### 7.3 Validity Rate

\[
\text{Validity \%} =
\frac{\text{Records passing validity rules}}
{\text{Records evaluated}}
\times 100
\]

Example:

| Metric | Total | Passing | Rate |
|---|---:|---:|---:|
| Completeness | 1,000 | 950 | 95% |
| Uniqueness | 1,000 | 980 | 98% |
| Validity | 1,000 | 970 | 97% |

These metrics should use clearly defined denominators and evaluation rules. For example, uniqueness calculations should specify how NULL keys are treated.

---

## 8. Handling Failed Data Quality Checks

When records fail quality checks, the pipeline should follow a defined policy.

Common approaches:

### 1. Reject

Stop the pipeline when critical requirements are violated.

Example: A required business key is missing.

### 2. Quarantine

Move invalid records to a separate location for investigation.

Example: An order contains an invalid status.

### 3. Correct

Apply a safe transformation when the correct value can be derived reliably.

Example: Standardize status values by trimming whitespace and applying consistent capitalization.

### 4. Accept with Warning

Allow records to continue when the issue is non-critical and the business permits it.

Example: An optional description field is missing.

### Example Architecture

```text
Incoming Data
      |
      v
Data Quality Checks
      |
      +----------------+
      |                |
      v                v
Valid Records     Invalid Records
      |                |
      v                v
Target Table       Quarantine
      |                |
      v                v
Analytics       Investigation
```

Do not silently discard invalid records. Track rejected counts and reasons.

---

## 9. Data Quality in a Medallion Architecture

### Bronze Layer

Focus:
- Preserve incoming data.
- Capture source metadata.
- Track ingestion completeness.
- Identify missing files and ingestion failures.

### Silver Layer

Focus:
- Required-field validation.
- Duplicate detection.
- Data-type validation.
- Range and format checks.
- Business-rule validation.
- Standardization and quarantine.

### Gold Layer

Focus:
- Business metric consistency.
- Aggregate validation.
- Referential consistency.
- Reporting accuracy.
- Reconciliation with expected totals.

### Example

```text
Source Data
     |
     v
Bronze
(Raw Data)
     |
     v
Silver
(Completeness + Uniqueness + Validity)
     |
     v
Gold
(Business-Level Validation)
     |
     v
Reports and Analytics
```

The exact placement of checks depends on the pipeline, but important data quality rules should be applied before unreliable data reaches business-facing outputs.

---

## 10. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| NULL always means bad data | Some fields are legitimately optional |
| Duplicate values always mean duplicate records | Check the correct business key |
| `dropDuplicates()` always keeps the latest record | It does not guarantee which record is retained |
| A non-NULL value is always valid | It may violate format, range, or business rules |
| Regex guarantees complete email validation | Regex checks only the pattern it defines |
| Data type conversion guarantees valid data | Conversion and business-rule validation are separate |
| Every invalid record should stop the pipeline | Use severity-based rejection, quarantine, or warning policies |
| High completeness means high overall quality | Completeness is only one quality dimension |
| Successful job execution means clean data | Data quality must be checked separately |

---

## 11. Interview Questions and Answers

### Q1. What is data quality?

Data quality measures whether data is accurate, complete, consistent, valid, and suitable for its intended use.

### Q2. What is completeness?

Completeness checks whether required fields and expected records are present.

### Q3. How do you check completeness in SQL?

I use `COUNT(*)`, `COUNT(column)`, and NULL checks to identify missing values.

### Q4. What is uniqueness?

Uniqueness ensures that records or business keys that should be unique are not duplicated.

### Q5. How do you identify duplicate records in SQL?

I use `GROUP BY` on the required business key and `HAVING COUNT(*) > 1`.

### Q6. How do you handle duplicates in PySpark?

I use `dropDuplicates()` when any retained record is acceptable. If the latest record must be retained, I use a window function with deterministic ordering.

### Q7. What is validity?

Validity checks whether data follows expected data types, formats, ranges, allowed values, and business rules.

### Q8. What is the difference between completeness and validity?

Completeness checks whether a value is present. Validity checks whether the present value is acceptable.

### Q9. Can a record be complete but invalid?

Yes. For example, an order amount of -500 is present but invalid if negative amounts are not allowed.

### Q10. Can a record be valid but incomplete?

A record may contain valid values in its populated fields but still be incomplete because a required field is missing.

### Q11. What is a composite key?

A composite key is a combination of multiple columns used together to identify a record uniquely.

### Q12. How do you handle invalid records in production?

Depending on business requirements, I reject, quarantine, correct, or accept records with warnings. I also monitor the number and reasons for failures.

### Q13. How would you implement data quality checks in a PySpark pipeline?

I would define required fields, check duplicates using business keys, validate data types and business rules, separate valid and invalid records, and monitor quality metrics.

### Q14. Where do you perform data quality checks in a medallion architecture?

Most detailed validation and standardization happens in Silver, while Bronze preserves source information and Gold performs business-level validation.

---

## 12. Quick Revision

| Dimension | Meaning | Typical Check |
|---|---|---|
| Completeness | Required data is present | NULL checks |
| Uniqueness | Required keys are not duplicated | GROUP BY and HAVING |
| Validity | Values follow defined rules | Type, range, format, and business rules |

### Important SQL Patterns

**Find missing values**

```sql
SELECT *
FROM customers
WHERE customer_id IS NULL;
```

**Find duplicate keys**

```sql
SELECT
    order_id,
    COUNT(*) AS total
FROM orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

**Find invalid amounts**

```sql
SELECT *
FROM orders
WHERE amount < 0;
```

### One-Minute Interview Answer

Completeness, uniqueness, and validity are three important dimensions of data quality.

Completeness checks whether required fields are populated. Uniqueness checks whether business keys contain duplicates. Validity checks whether values follow expected data types, formats, ranges, and business rules.

In a PySpark pipeline, I use NULL checks for completeness, grouping or window functions for duplicate detection, and conditional expressions for validity checks. Invalid records can be rejected or moved to a quarantine location based on business requirements. I also monitor quality metrics so that issues are detected before they affect downstream reports.

---

## 13. Self-Assessment

- [ ] Define data quality.
- [ ] Explain completeness with an example.
- [ ] Calculate completeness percentage.
- [ ] Identify missing values using SQL and PySpark.
- [ ] Explain uniqueness and composite keys.
- [ ] Identify duplicate records using SQL and PySpark.
- [ ] Explain deterministic deduplication.
- [ ] Explain validity with examples.
- [ ] Validate data types, formats, and ranges.
- [ ] Combine all three checks in a PySpark pipeline.
- [ ] Explain rejection and quarantine strategies.
- [ ] Describe data quality checks in Bronze, Silver, and Gold.

**Interview readiness:** Be able to explain how you would validate an orders dataset containing missing customer IDs, duplicate order IDs, and negative amounts before loading it into a Silver Delta table.