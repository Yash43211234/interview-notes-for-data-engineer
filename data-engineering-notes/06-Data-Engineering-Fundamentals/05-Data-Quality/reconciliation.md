# Data Reconciliation in Data Engineering

## 1. Introduction

**Data reconciliation is the process of comparing data between two or more systems, datasets, or pipeline stages to verify that the information is consistent and correctly transferred.**

It helps ensure that data has not been lost, duplicated, incorrectly transformed, or changed unexpectedly during processing.

For example, an e-commerce application sends 10,000 orders to a data warehouse.

After ingestion, the data engineering team finds only 9,850 records in the target table.

The difference is 150 records.

Reconciliation helps identify whether those records were:
- Lost during ingestion.
- Rejected during validation.
- Duplicated or removed.
- Excluded by transformation logic.
- Delayed or missing from the source extract.

**Key idea:** Reconciliation verifies that data in the source and target systems agrees according to defined comparison rules.

---

## 2. Why Is Data Reconciliation Important?

Data moves through multiple systems and pipeline stages.

```text
Source System
     |
     v
Data Ingestion
     |
     v
Bronze Layer
     |
     v
Silver Layer
     |
     v
Gold Layer
     |
     v
Reporting System
```

During these stages, data may be filtered, transformed, aggregated, or rejected.

Without reconciliation, it can be difficult to know whether the target contains the expected data.

### Common Problems Reconciliation Detects

1. Missing records.
2. Duplicate records.
3. Incorrect amounts or totals.
4. Data loss during ingestion.
5. Unexpected transformation results.
6. Source-target mismatches.
7. Incorrect filtering conditions.
8. Delayed or incomplete data loads.

### Benefits

- Improves data accuracy.
- Detects missing or duplicated records.
- Validates pipeline transformations.
- Builds trust in reports and analytics.
- Helps investigate production issues.
- Supports financial and operational controls.
- Improves data pipeline reliability.

---

## 3. Types of Data Reconciliation

Reconciliation can be performed at different levels depending on the dataset and business requirement.

### 3.1 Record Count Reconciliation

Compares the number of records between source and target.

Example:

| System | Record Count |
|---|---:|
| Source | 10,000 |
| Target | 9,850 |
| Difference | 150 |

SQL:

```sql
SELECT COUNT(*) AS total_records
FROM source_orders;
```

```sql
SELECT COUNT(*) AS total_records
FROM target_orders;
```

Difference:

\[
\text{Difference} =
\text{Source Count} - \text{Target Count}
\]

A non-zero difference indicates that further investigation may be required.

However, different counts do not always mean an error. The target may intentionally exclude invalid or irrelevant records.

### 3.2 Aggregate Reconciliation

Compares totals or summary metrics between systems.

Common examples:
- Total sales amount.
- Total transaction value.
- Total quantity.
- Total tax.
- Total number of customers.

Example:

| Metric | Source | Target |
|---|---:|---:|
| Order Count | 1,000 | 1,000 |
| Total Amount | ₹500,000 | ₹498,000 |
| Difference | — | ₹2,000 |

The record counts match, but the total amount differs.

This can indicate incorrect transformations, missing values, or incorrect calculations.

SQL:

```sql
SELECT
    COUNT(*) AS total_orders,
    SUM(amount) AS total_amount
FROM source_orders;
```

```sql
SELECT
    COUNT(*) AS total_orders,
    SUM(amount) AS total_amount
FROM target_orders;
```

**Important:** Matching record counts does not prove that the actual records or values match.

### 3.3 Key-Level Reconciliation

Compares business keys between source and target.

Example:

Source:

| order_id |
|---|
| 101 |
| 102 |
| 103 |
| 104 |

Target:

| order_id |
|---|
| 101 |
| 102 |
| 104 |

Order 103 is missing from the target.

SQL:

```sql
SELECT s.order_id
FROM source_orders s
LEFT JOIN target_orders t
    ON s.order_id = t.order_id
WHERE t.order_id IS NULL;
```

This identifies source records that do not have a matching target record.

The reverse comparison identifies records that exist only in the target.

### 3.4 Column-Level Reconciliation

Compares individual field values for matching records.

Example:

| order_id | Source Amount | Target Amount |
|---|---:|---:|
| 101 | 500 | 500 |
| 102 | 700 | 700 |
| 103 | 300 | 250 |

The amount for order 103 does not match.

SQL:

```sql
SELECT
    s.order_id,
    s.amount AS source_amount,
    t.amount AS target_amount
FROM source_orders s
JOIN target_orders t
    ON s.order_id = t.order_id
WHERE s.amount <> t.amount;
```

For nullable columns, use NULL-safe comparison supported by the SQL engine. In Spark SQL, the NULL-safe equality operator is `<=>`.

### 3.5 Transformation Reconciliation

Checks whether data transformations produce the expected results.

Example:

Source:

| order_id | quantity | unit_price |
|---|---:|---:|
| 101 | 2 | 100 |
| 102 | 3 | 200 |

Expected transformation:

\[
\text{Total Amount} = \text{Quantity} \times \text{Unit Price}
\]

Expected output:

| order_id | total_amount |
|---|---:|
| 101 | 200 |
| 102 | 600 |

Reconciliation verifies that the target calculation matches the expected business logic.

---

## 4. Record Count Reconciliation in SQL

Suppose we have two tables:

- `source_orders`
- `target_orders`

### Step 1: Compare Record Counts

```sql
SELECT
    (SELECT COUNT(*) FROM source_orders) AS source_count,
    (SELECT COUNT(*) FROM target_orders) AS target_count;
```

### Step 2: Calculate the Difference

```sql
SELECT
    (SELECT COUNT(*) FROM source_orders) -
    (SELECT COUNT(*) FROM target_orders) AS count_difference;
```

### Step 3: Identify Missing Records

```sql
SELECT s.order_id
FROM source_orders s
LEFT JOIN target_orders t
    ON s.order_id = t.order_id
WHERE t.order_id IS NULL;
```

### Step 4: Identify Extra Target Records

```sql
SELECT t.order_id
FROM target_orders t
LEFT JOIN source_orders s
    ON t.order_id = s.order_id
WHERE s.order_id IS NULL;
```

These comparisons assume `order_id` is a suitable key and is non-NULL.

---

## 5. Data Reconciliation Using PySpark

Suppose we have two DataFrames:

- `source_df`
- `target_df`

Both contain an `order_id` and `amount` column.

### 5.1 Compare Record Counts

```python
source_count = source_df.count()
target_count = target_df.count()

print("Source Count:", source_count)
print("Target Count:", target_count)

print("Difference:", source_count - target_count)
```

### 5.2 Find Missing Target Records

```python
missing_df = source_df.join(
    target_df.select("order_id"),
    on="order_id",
    how="left_anti"
)

missing_df.show()
```

A left anti join returns source rows without a matching target key.

### 5.3 Find Extra Target Records

```python
extra_df = target_df.join(
    source_df.select("order_id"),
    on="order_id",
    how="left_anti"
)

extra_df.show()
```

### 5.4 Compare Column Values

```python
from pyspark.sql import functions as F

mismatch_df = (
    source_df.alias("s")
    .join(
        target_df.alias("t"),
        F.col("s.order_id") == F.col("t.order_id"),
        "inner"
    )
    .filter(
        ~F.col("s.amount").eqNullSafe(
            F.col("t.amount")
        )
    )
    .select(
        F.col("s.order_id"),
        F.col("s.amount").alias("source_amount"),
        F.col("t.amount").alias("target_amount")
    )
)

mismatch_df.show()
```

`eqNullSafe()` handles NULL comparisons explicitly.

### 5.5 Compare Aggregate Amounts

```python
source_total = source_df.agg(
    F.sum("amount").alias("source_total")
)

target_total = target_df.agg(
    F.sum("amount").alias("target_total")
)

source_total.show()
target_total.show()
```

For a production reconciliation, aggregate both sides using the same filters, grouping keys, and NULL-handling rules.

---

## 6. Reconciliation at Different Pipeline Stages

Reconciliation should be performed at important boundaries in a data pipeline.

### Source to Bronze

Check:
- Number of source records.
- Number of ingested records.
- Number of files received.
- Missing or failed files.
- Duplicate ingestion.

### Bronze to Silver

Check:
- Input and output record counts.
- Rejected record counts.
- Duplicate removal.
- Validated records.
- Transformation results.

### Silver to Gold

Check:
- Aggregated totals.
- Business calculations.
- Group-level counts.
- Metric consistency.
- Expected output values.

### Gold to Reporting

Check:
- Dashboard totals.
- Report freshness.
- Business metric consistency.
- Reconciliation with approved source totals.

### Example

```text
Source: 10,000 records
          |
          v
Bronze: 10,000 records
          |
          v
Silver: 9,850 valid records
        150 quarantined records
          |
          v
Gold: Aggregated business data
          |
          v
Dashboard
```

The Bronze-to-Silver count difference is expected if all 150 rejected records are accounted for.

Reconciliation should explain differences, not simply require every layer to have identical record counts.

---

## 7. Reconciliation Using Control Totals

A control total is a summary value calculated to help verify that data has been transferred or processed correctly.

Common control totals include:

- Record count.
- Sum of transaction amounts.
- Total quantity.
- Minimum and maximum transaction dates.
- Number of distinct business keys.

### Example

A source system sends 5,000 transactions.

The source control totals are:

```text
Record Count: 5,000
Total Amount:  ₹2,500,000
Distinct IDs: 5,000
```

The target system produces:

```text
Record Count: 5,000
Total Amount:  ₹2,500,000
Distinct IDs: 5,000
```

The matching totals provide evidence that the transfer is consistent for those checks.

However, control totals cannot prove every individual record is correct. Different errors can cancel each other out.

For stronger validation, combine control totals with key-level and column-level comparisons.

---

## 8. Partition-Level Reconciliation

For large datasets, comparing the entire dataset at once may be expensive.

A common approach is to reconcile data by date, region, batch ID, or another partitioning key.

Example:

| Order Date | Source Count | Target Count | Difference |
|---|---:|---:|---:|
| 2026-10-01 | 1,000 | 1,000 | 0 |
| 2026-10-02 | 1,200 | 1,150 | 50 |
| 2026-10-03 | 900 | 900 | 0 |

This helps identify the specific date or partition where records are missing.

SQL:

```sql
SELECT
    order_date,
    COUNT(*) AS source_count
FROM source_orders
GROUP BY order_date;
```

Compare with:

```sql
SELECT
    order_date,
    COUNT(*) AS target_count
FROM target_orders
GROUP BY order_date;
```

A production implementation can join these grouped results by date and calculate differences.

### Benefits

- Easier identification of affected data.
- Reduced comparison scope.
- Better support for incremental pipelines.
- More efficient investigation and reprocessing.

---

## 9. Reconciliation in Incremental Pipelines

Incremental pipelines process only new or changed data instead of reloading the complete dataset.

Reconciliation is important because incremental loads can miss updates, late-arriving records, or deletes.

### Example

The pipeline processes orders updated after the last successful watermark.

```text
Last Watermark: 10:00 AM
Current Run:    11:00 AM

Expected Input:
Records updated between 10:00 and 11:00
```

The pipeline should track:
- Source records selected.
- Records successfully written.
- Records rejected.
- Duplicate keys.
- Target changes committed.

### Important Practices

1. Track the extraction range or batch ID.
2. Compare source and target records for that batch.
3. Account for rejected records.
4. Use idempotent writes so retries do not create duplicates.
5. Handle late-arriving updates.
6. Reconcile deletes when the source supports them.
7. Advance the watermark only after successful target processing and validation.

**Important:** A matching record count does not prove an incremental load captured every expected update. Key-level or change-level reconciliation may also be required.

---

## 10. Handling Reconciliation Failures

When reconciliation detects a mismatch, the team should investigate before assuming the target is incorrect.

### Common Causes

- Missing source files.
- Failed ingestion.
- Duplicate source records.
- Incorrect filters.
- Transformation bugs.
- Data type conversion issues.
- Late-arriving data.
- Incorrect watermark handling.
- Target write failures.
- Expected records rejected by validation.

### Recovery Process

```text
Reconciliation Failure
         |
         v
Identify Mismatched Metric
         |
         v
Locate Affected Records
         |
         v
Find Root Cause
         |
         v
Correct Source or Pipeline
         |
         v
Reprocess Affected Data
         |
         v
Run Reconciliation Again
         |
         v
Confirm Expected Results
```

### Example

Suppose the source contains 10,000 records and the target contains 9,850.

Investigation shows:
- 100 records failed validation.
- 50 records were missed because of an incorrect extraction filter.

The correct response is not simply to insert 150 records without investigation.

Instead:
1. Verify the 100 rejected records and their handling.
2. Correct the extraction filter.
3. Reprocess the missing 50 records safely.
4. Reconcile the target again.
5. Confirm that all expected records are accounted for.

---

## 11. Reconciliation vs Data Quality Monitoring

These concepts are related but different.

### Data Quality Monitoring

Continuously measures whether data meets quality standards.

Examples:
- Missing-value percentage.
- Duplicate count.
- Invalid-record percentage.
- Data freshness.

### Data Reconciliation

Compares data between systems or pipeline stages to verify consistency.

Examples:
- Source and target record counts.
- Source and target financial totals.
- Missing business keys.
- Differences in transformed values.

### Comparison

| Data Quality Monitoring | Data Reconciliation |
|---|---|
| Tracks data quality metrics | Compares datasets or systems |
| Can monitor a single dataset | Usually involves two or more datasets or stages |
| Detects NULLs, duplicates, and invalid values | Detects missing, extra, or mismatched data |
| Tracks quality over time | Verifies consistency between defined boundaries |
| Example: NULL percentage is 2% | Example: 50 source records are missing in target |

Both are important for reliable data pipelines.

---

## 12. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| Matching record counts prove data is correct | Individual records or values can still differ |
| Every source-target count difference is an error | Some records may be intentionally filtered or rejected |
| Aggregate totals prove complete consistency | Different record-level errors can cancel each other out |
| Reconciliation only means comparing counts | It also includes keys, values, aggregates, and transformations |
| Every mismatch should be fixed by inserting records | Investigate the root cause first |
| Full-table reconciliation is always required | Partition or batch-level reconciliation may be more practical |
| NULL values can be compared with ordinary equality | Use explicit NULL-safe comparison |
| Reconciliation is only needed after a pipeline fails | It should be part of normal pipeline validation |

---

## 13. Interview Questions and Answers

### Q1. What is data reconciliation?

Data reconciliation is the process of comparing data between source and target systems or pipeline stages to verify that records and values are consistent.

### Q2. Why is reconciliation important in data engineering?

It helps identify missing records, duplicate records, incorrect totals, and transformation issues before unreliable data reaches downstream systems.

### Q3. What are the different types of reconciliation?

Common types include record count, aggregate, key-level, column-level, and transformation reconciliation.

### Q4. How do you compare source and target record counts?

I use `COUNT(*)` on both datasets and calculate the difference. If the counts differ, I investigate whether the difference is expected or caused by missing records.

### Q5. How do you find records missing from the target using PySpark?

I use a left anti join between the source and target on the business key.

### Q6. What is a control total?

A control total is a summary metric, such as record count or total transaction amount, used to verify data transfer or processing.

### Q7. Is matching record count sufficient for reconciliation?

No. Matching counts do not guarantee that the same records or values exist. I also compare business keys, important columns, and aggregate values.

### Q8. How do you reconcile large datasets?

I use partition-level or batch-level reconciliation, compare control totals, and perform detailed comparisons on affected partitions or business keys.

### Q9. How do you handle reconciliation failures?

I identify the mismatched metric, locate affected records, investigate the root cause, correct the issue, reprocess data safely, and run reconciliation again.

### Q10. What is the difference between data reconciliation and data quality monitoring?

Data quality monitoring tracks metrics such as NULLs, duplicates, and freshness. Reconciliation compares data between systems or stages to verify consistency.

### Q11. How do you reconcile an incremental pipeline?

I track each batch or extraction range, compare expected source records with target results, account for rejected records, and ensure retries are safe and idempotent.

### Q12. Can aggregate reconciliation detect every data issue?

No. Two datasets can have matching totals even when individual records differ. Aggregate checks should be combined with key-level and column-level comparisons.

---

## 14. Quick Revision

| Reconciliation Type | Purpose |
|---|---|
| Record count | Compare number of records |
| Aggregate | Compare sums and summary metrics |
| Key-level | Identify missing or extra business keys |
| Column-level | Compare individual field values |
| Transformation | Verify expected transformation results |
| Partition-level | Compare data by date, region, or batch |

### Important SQL Patterns

**Compare counts**

```sql
SELECT COUNT(*) FROM source_orders;
SELECT COUNT(*) FROM target_orders;
```

**Find missing target records**

```sql
SELECT s.order_id
FROM source_orders s
LEFT JOIN target_orders t
    ON s.order_id = t.order_id
WHERE t.order_id IS NULL;
```

**Compare amounts**

```sql
SELECT
    s.order_id,
    s.amount AS source_amount,
    t.amount AS target_amount
FROM source_orders s
JOIN target_orders t
    ON s.order_id = t.order_id
WHERE s.amount <> t.amount;
```

### One-Minute Interview Answer

Data reconciliation is the process of comparing data between source and target systems or different stages of a data pipeline to verify consistency.

I commonly use record count reconciliation, aggregate reconciliation, key-level reconciliation, and column-level reconciliation.

For example, if a source contains 10,000 orders but the target contains only 9,850, I compare the business keys to identify missing records. I also compare important totals, such as the total order amount, to detect transformation issues.

For large datasets, I prefer partition-level or batch-level reconciliation. If a mismatch occurs, I investigate the root cause, correct the issue, reprocess the affected data safely, and run reconciliation again.

---

## 15. Self-Assessment

- [ ] Define data reconciliation.
- [ ] Explain why reconciliation is important.
- [ ] Compare source and target record counts.
- [ ] Perform aggregate reconciliation.
- [ ] Identify missing and extra records using SQL.
- [ ] Identify missing records using PySpark left anti joins.
- [ ] Compare individual column values.
- [ ] Explain control totals.
- [ ] Perform partition-level reconciliation.
- [ ] Explain reconciliation in incremental pipelines.
- [ ] Handle reconciliation failures.
- [ ] Differentiate reconciliation from data quality monitoring.
- [ ] Explain why matching counts do not prove data correctness.

**Interview readiness:** Be able to explain how you would reconcile an orders pipeline from source to Bronze and Silver, identify missing order IDs, compare total amounts, account for rejected records, and safely reprocess mismatches.