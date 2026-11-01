# Data Quality Checks in Spark Structured Streaming

## 1. What Is Data Quality?

Data quality means ensuring that data is accurate, complete, consistent, valid, and reliable before it is used by downstream systems.

In a streaming pipeline, data arrives continuously. Therefore, quality checks must run as new records are processed.

### Example

An e-commerce application receives order records.

| Order ID | Customer | Amount | Status |
|---|---|---:|---|
| 101 | Rahul | 500 | SUCCESS |
| 102 | Priya | -200 | SUCCESS |
| 103 | NULL | 300 | SUCCESS |
| 104 | Amit | 700 | INVALID |

Some records contain problems:
- Negative order amount.
- Missing customer.
- Invalid order status.

A data quality process identifies these records and decides whether to accept, reject, correct, or quarantine them.

**Key idea:** Data quality checks ensure that only valid and reliable data reaches downstream processing.

---

## 2. Why Are Data Quality Checks Important?

Without proper validation, bad data can affect reports, dashboards, machine learning models, and business decisions.

Common problems include:

- Missing values.
- Duplicate records.
- Invalid data types.
- Incorrect business values.
- Invalid timestamps.
- Unexpected schema changes.
- Inconsistent records.

### Benefits

1. Improves data reliability.
2. Prevents incorrect business calculations.
3. Identifies bad records early.
4. Helps maintain consistent datasets.
5. Makes pipelines easier to monitor and debug.

---

## 3. Common Data Quality Checks

### 1. Null Checks

Identify missing values in important columns.

Example:

```python
from pyspark.sql.functions import col

valid_df = df.filter(
    col("customer_id").isNotNull()
)
```

To identify records with missing customer IDs:

```python
invalid_df = df.filter(
    col("customer_id").isNull()
)
```

### 2. Duplicate Checks

Identify duplicate records using a business key.

```python
duplicate_df = (
    df.groupBy("order_id")
      .count()
      .filter(col("count") > 1)
)
```

To remove duplicate order IDs:

```python
deduplicated_df = df.dropDuplicates(["order_id"])
```

**Important:** `dropDuplicates()` does not guarantee which version of a duplicate record is retained. If the latest record must be preserved, use a deterministic ordering strategy.

### 3. Range Checks

Check whether numeric values fall within an acceptable range.

Example: Order amount must be greater than zero.

```python
valid_df = df.filter(
    col("amount") > 0
)
```

Invalid records:

```python
invalid_df = df.filter(
    col("amount") <= 0
)
```

### 4. Format Checks

Validate whether values follow an expected format.

Example: Email addresses should contain an `@` symbol.

```python
valid_df = df.filter(
    col("email").rlike(r"^[^@]+@[^@]+\.[^@]+$")
)
```

This is a basic format check, not a complete guarantee that an email address exists or is deliverable.

### 5. Referential Integrity Checks

Check whether a foreign key exists in the corresponding reference dataset.

Example:

- Orders contain `customer_id`.
- Customers contain valid `customer_id` values.

A left anti join identifies orders without a matching customer.

```python
invalid_orders = orders.join(
    customers.select("customer_id"),
    on="customer_id",
    how="left_anti"
)
```

### 6. Business Rule Checks

Validate rules defined by the business.

Example:

An order status must be one of:

- CREATED
- SUCCESS
- CANCELLED

```python
valid_statuses = [
    "CREATED",
    "SUCCESS",
    "CANCELLED"
]

valid_df = df.filter(
    col("status").isin(valid_statuses)
)
```

---

## 4. Data Quality Using PySpark

Consider an incoming orders dataset.

### Sample Schema

```python
from pyspark.sql.types import (
    StructType,
    StructField,
    IntegerType,
    StringType,
    DoubleType
)

schema = StructType([
    StructField("order_id", IntegerType(), True),
    StructField("customer_id", IntegerType(), True),
    StructField("amount", DoubleType(), True),
    StructField("status", StringType(), True)
])
```

### Step 1: Read the Data

```python
df = (
    spark.read
        .format("csv")
        .option("header", True)
        .schema(schema)
        .load("/data/orders")
)
```

### Step 2: Validate Required Columns

```python
from pyspark.sql.functions import col

valid_df = df.filter(
    col("order_id").isNotNull() &
    col("customer_id").isNotNull() &
    col("amount").isNotNull()
)
```

### Step 3: Validate Amount

```python
valid_df = valid_df.filter(
    col("amount") > 0
)
```

### Step 4: Validate Status

```python
valid_statuses = [
    "CREATED",
    "SUCCESS",
    "CANCELLED"
]

valid_df = valid_df.filter(
    col("status").isin(valid_statuses)
)
```

### Step 5: Remove Duplicates

```python
valid_df = valid_df.dropDuplicates(
    ["order_id"]
)
```

### Step 6: Write Valid Records

```python
valid_df.write \
    .format("parquet") \
    .mode("overwrite") \
    .save("/output/valid_orders")
```

This example demonstrates basic batch validation. In a production pipeline, invalid records should generally be captured separately rather than silently discarded.

---

## 5. Handling Invalid Records

A good data pipeline should not simply remove bad records without tracking them.

A common approach is to separate valid and invalid records.

### Pipeline Design

```text
Incoming Data
      |
      v
Data Quality Checks
      |
      v
   Decision
    /   \
   /     \
Valid   Invalid
  |        |
  v        v
Clean    Quarantine
Data     / Reject Table
  |
  v
Silver Layer
```

### Example

```python
from pyspark.sql.functions import col

valid_condition = (
    col("order_id").isNotNull() &
    col("customer_id").isNotNull() &
    col("amount").isNotNull() &
    (col("amount") > 0)
)

valid_df = df.filter(valid_condition)

invalid_df = df.filter(~valid_condition)
```

### Write Valid Records

```python
valid_df.write \
    .mode("append") \
    .parquet("/data/valid_orders")
```

### Write Invalid Records

```python
invalid_df.write \
    .mode("append") \
    .parquet("/data/quarantine_orders")
```

**Important:** A production pipeline should also capture rejection reasons, source information, and processing timestamps. In a streaming application, both outputs must be designed carefully to handle retries and avoid duplicate writes.

---

## 6. Data Quality Checks in Structured Streaming

The same validation concepts can be applied to streaming DataFrames.

Example:

```python
from pyspark.sql.functions import col

stream_df = (
    spark.readStream
        .format("json")
        .schema(schema)
        .load("/input/orders")
)

valid_stream = stream_df.filter(
    col("order_id").isNotNull() &
    col("customer_id").isNotNull() &
    col("amount").isNotNull() &
    (col("amount") > 0)
)
```

### Writing Valid Records

```python
query = (
    valid_stream.writeStream
        .format("parquet")
        .outputMode("append")
        .option(
            "checkpointLocation",
            "/checkpoints/valid_orders"
        )
        .start("/output/valid_orders")
)
```

This continuously filters invalid records from the streaming output.

**Important:** A filter alone does not preserve rejected records. To capture invalid records, define a separate output path or implement a suitable `foreachBatch` workflow.

---

## 7. Using foreachBatch for Data Quality

`foreachBatch()` allows us to apply batch DataFrame logic to each micro-batch of a streaming query.

It is useful when quality checks require more complex processing or multiple output destinations.

### Example

```python
from pyspark.sql.functions import col

def process_batch(batch_df, batch_id):

    valid_condition = (
        col("order_id").isNotNull() &
        col("customer_id").isNotNull() &
        col("amount").isNotNull() &
        (col("amount") > 0)
    )

    valid_df = batch_df.filter(valid_condition)

    invalid_df = batch_df.filter(~valid_condition)

    valid_df.write \
        .format("delta") \
        .mode("append") \
        .save("/data/valid_orders")

    invalid_df.write \
        .format("delta") \
        .mode("append") \
        .save("/data/invalid_orders")


query = (
    stream_df.writeStream
        .foreachBatch(process_batch)
        .option(
            "checkpointLocation",
            "/checkpoints/orders"
        )
        .start()
)
```

### How It Works

1. Spark reads streaming data.
2. Spark creates a micro-batch.
3. `process_batch()` receives that batch as a normal DataFrame.
4. The function separates valid and invalid records.
5. Valid records are written to one destination.
6. Invalid records are written to another destination.
7. Spark continues with the next micro-batch.

### Important Points

- `batch_df` is a batch DataFrame.
- `batch_id` identifies the micro-batch within the query.
- You can use normal batch DataFrame operations inside the function.
- The function runs once per micro-batch execution, but failures and retries can cause the same batch to be processed again.

**Interview point:** `foreachBatch()` is useful for applying custom batch processing logic to streaming data.

---

## 8. Data Quality Rules and Expectations

A data quality rule is a condition that a record must satisfy.

Examples:

| Rule | Condition |
|---|---|
| Order ID required | `order_id IS NOT NULL` |
| Customer ID required | `customer_id IS NOT NULL` |
| Amount positive | `amount > 0` |
| Valid status | `status IN ('CREATED', 'SUCCESS', 'CANCELLED')` |
| Unique order | One record per `order_id` |

In Databricks, Delta Live Tables (now part of Lakeflow Spark Declarative Pipelines) provides data quality expectations.

Expectations allow pipelines to define rules and choose how to handle records that violate them.

Common actions include:

- `EXPECT`: Track quality without dropping invalid records.
- `EXPECT ... ON VIOLATION DROP ROW`: Drop records that violate the rule.
- `EXPECT ... ON VIOLATION FAIL UPDATE`: Fail the pipeline update when a rule is violated.

Example using the Lakeflow Spark Declarative Pipelines Python API:

```python
from pyspark import pipelines as dp
from pyspark.sql.functions import col

@dp.expect("valid_order_amount", "amount > 0")
@dp.table
def cleaned_orders():
    return spark.readStream.table("orders")
```

**Important:** These pipeline expectations are Databricks-specific functionality. They are not ordinary PySpark DataFrame methods.

---

## 9. Data Quality Metrics

A production pipeline should measure data quality, not just filter invalid records.

Common metrics include:

| Metric | Meaning |
|---|---|
| Total Records | Total records received |
| Valid Records | Records that passed validation |
| Invalid Records | Records that failed validation |
| Duplicate Count | Number of duplicate records |
| Null Count | Number of missing values |
| Rejection Rate | Percentage of invalid records |

### Rejection Rate

\[
Rejection\ Rate =
\frac{Invalid\ Records}{Total\ Records}
\times 100
\]

Example:

- Total records = 10,000.
- Invalid records = 200.

\[
Rejection\ Rate =
\frac{200}{10000}\times100=2\%
\]

This means 2% of incoming records failed the validation rules.

### Why Track Metrics?

- Detect data source problems.
- Identify sudden quality drops.
- Monitor pipeline health.
- Support operational reporting.
- Alert teams when rejection rates exceed acceptable limits.

---

## 10. Data Quality in Medallion Architecture

Data quality checks are commonly implemented across the Bronze, Silver, and Gold layers.

| Layer | Main Responsibility | Quality Checks |
|---|---|---|
| Bronze | Store raw incoming data | Basic schema and ingestion checks |
| Silver | Clean and standardize data | Null, duplicate, format, and business-rule checks |
| Gold | Prepare business-ready datasets | Aggregation and business consistency checks |

### Typical Flow

```text
Source Data
     |
     v
Bronze Layer
(Raw Data)
     |
     v
Silver Layer
(Cleaned and Validated Data)
     |
     v
Gold Layer
(Business-Ready Data)
```

### Example

**Bronze:**
- Preserve source records.
- Capture ingestion metadata.
- Identify malformed input.

**Silver:**
- Remove or handle duplicates.
- Validate required columns.
- Standardize formats.
- Apply business rules.
- Quarantine invalid records.

**Gold:**
- Validate business totals.
- Check aggregation consistency.
- Prepare reporting datasets.

**Important:** The exact placement of checks depends on business requirements. Some basic checks should happen before Bronze data is accepted, while raw data is often preserved for auditing and reprocessing.

---

## 11. Common Mistakes

1. Dropping invalid records without storing them.
2. Checking only NULL values and ignoring business rules.
3. Assuming `dropDuplicates()` always retains the latest record.
4. Ignoring schema validation.
5. Not monitoring rejection rates.
6. Treating a watermark as a data quality check.
7. Assuming filtering invalid records automatically creates a quarantine dataset.
8. Forgetting that `foreachBatch()` can process the same batch again after retries.
9. Writing to multiple destinations without considering idempotency.
10. Confusing Databricks pipeline expectations with standard PySpark methods.
11. Assuming a successful Spark job means the data is correct.

---

## 12. Interview Questions

**Q1. What is data quality?**

Data quality ensures that data is accurate, complete, consistent, valid, and reliable.

**Q2. What are common data quality checks?**

Null checks, duplicate checks, range checks, format checks, referential integrity checks, and business-rule validation.

**Q3. How do you identify duplicate records in PySpark?**

Using `groupBy()` with `count()` to identify duplicate keys, or `dropDuplicates()` to remove duplicates.

**Q4. How do you handle invalid records?**

Separate valid and invalid records, write invalid records to a quarantine or reject destination, and track rejection reasons.

**Q5. How do you implement data quality checks in Structured Streaming?**

Use supported DataFrame transformations and filters. For more complex processing, use `foreachBatch()` to apply batch validation logic to each micro-batch.

**Q6. What is `foreachBatch()`?**

It allows us to apply normal batch DataFrame operations to each micro-batch of a streaming query.

**Q7. Why is `foreachBatch()` useful?**

It supports custom processing logic, multiple output destinations, and operations that are easier to implement using batch DataFrames.

**Q8. What is a quarantine table?**

A destination where invalid or rejected records are stored for investigation, correction, or reprocessing.

**Q9. What is a data quality expectation in Databricks?**

It is a rule that validates records in a Lakeflow Spark Declarative Pipeline and can track, drop, or fail on violating records depending on the configured action.

**Q10. What is the difference between validation and cleansing?**

Validation identifies whether data meets defined rules. Cleansing corrects, standardizes, or removes problematic data.

**Q11. How do you monitor data quality?**

Track metrics such as total records, valid records, invalid records, duplicates, null counts, and rejection rates.

**Q12. Where do you implement data quality checks in a medallion architecture?**

Basic ingestion checks can happen in Bronze, detailed cleansing and validation usually happen in Silver, and business-level consistency checks can happen in Gold.

---

## 13. Quick Revision

| Concept | Remember |
|---|---|
| Data Quality | Accuracy, completeness, consistency, validity |
| Null Check | Identifies missing values |
| Duplicate Check | Identifies repeated records |
| Range Check | Validates numeric boundaries |
| Format Check | Validates expected patterns |
| Referential Integrity | Checks relationships between datasets |
| Business Rules | Validates business conditions |
| Quarantine | Stores invalid records |
| `foreachBatch()` | Applies batch logic to micro-batches |
| Expectations | Databricks pipeline quality rules |
| Rejection Rate | Invalid records / Total records × 100 |

### One-Line Revision

- Null check → Missing values.
- Duplicate check → Repeated records.
- Range check → Valid numeric values.
- Format check → Expected patterns.
- Business rule → Business-specific validation.
- Quarantine → Store rejected records.
- `foreachBatch()` → Batch processing inside streaming.
- Expectations → Databricks pipeline quality rules.

---

## 14. Self-Check

- [ ] Can I explain data quality?
- [ ] Can I perform null and duplicate checks?
- [ ] Can I perform range and format checks?
- [ ] Can I validate business rules?
- [ ] Can I separate valid and invalid records?
- [ ] Can I explain quarantine tables?
- [ ] Can I use `foreachBatch()`?
- [ ] Can I explain data quality expectations?
- [ ] Can I calculate rejection rates?
- [ ] Can I explain quality checks in Bronze, Silver, and Gold?

**Final takeaway:** Data quality is not just about removing invalid records. A reliable data pipeline validates incoming data, separates rejected records, tracks quality metrics, and ensures that downstream systems receive trustworthy data.