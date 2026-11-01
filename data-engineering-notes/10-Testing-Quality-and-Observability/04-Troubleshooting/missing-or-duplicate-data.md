

# Missing or Duplicate Data: Troubleshooting in Data Pipelines

## 1. Introduction

Missing data and duplicate data are two common problems in data engineering.

- **Missing data** occurs when expected records are absent from a dataset or target table.
- **Duplicate data** occurs when the same logical business record appears more than once when it should appear only once.

Both problems can lead to incorrect reports, inaccurate business metrics, and unreliable downstream systems.

### Example

Suppose an e-commerce source contains 10,000 orders.

After processing:

- The target table contains only 9,800 orders.
- Some orders are missing.
- Other orders appear more than once.

The data engineer must identify where the records were lost or duplicated, correct the pipeline, and validate the final result.

---

## 2. Understanding Missing Data

Missing data means expected records or values are not available at the required location or time.

There are two important types.

### 2.1 Missing Records

Entire records are absent.

Example:

**Source table**

| order_id | amount |
|---|---:|
| 101 | 500 |
| 102 | 700 |
| 103 | 900 |
| 104 | 300 |

**Target table**

| order_id | amount |
|---|---:|
| 101 | 500 |
| 102 | 700 |
| 104 | 300 |

Order `103` is missing from the target.

### 2.2 Missing Values

A record exists, but one or more columns contain NULL or blank values.

Example:

| order_id | customer_id | amount |
|---|---|---:|
| 101 | C101 | 500 |
| 102 | NULL | 700 |
| 103 | C103 | NULL |

Here, records exist, but some values are missing.

**Important:** Missing records and missing column values are different problems and require different checks.

---

## 3. Common Causes of Missing Data

| Cause | Explanation |
|---|---|
| Incorrect filter | Records are unintentionally excluded |
| Failed ingestion | Source records never reach the raw layer |
| Incremental load issue | Watermark skips records |
| Late-arriving data | Records arrive after the pipeline runs |
| Schema mismatch | Records fail during parsing or conversion |
| NULL handling | Invalid filtering or transformation logic |
| Join issue | Records are removed by an inner join |
| Failed write | Target write does not complete |
| Incorrect partition filter | Required date partitions are skipped |
| Source deletion | Records are deleted before extraction |
| API pagination issue | Some pages are not retrieved |

### Example: Incorrect Filter

```sql
SELECT *
FROM source_orders
WHERE order_date >= '2026-10-01'
  AND order_date < '2026-10-25';
```

If the required reporting period includes October 25, this filter excludes that date.

Use an appropriate end boundary:

```sql
SELECT *
FROM source_orders
WHERE order_date >= '2026-10-01'
  AND order_date < '2026-10-26';
```

The half-open interval includes all timestamps on October 25 without relying on a particular time precision.

---

## 4. Understanding Duplicate Data

Duplicate data occurs when the same logical business record appears multiple times.

### Example

| order_id | customer_id | amount |
|---|---|---:|
| 101 | C101 | 500 |
| 102 | C102 | 700 |
| 102 | C102 | 700 |
| 103 | C103 | 900 |

Order `102` appears twice.

### Types of Duplicates

**1. Exact Duplicates**

All relevant column values are identical.

**2. Business-Key Duplicates**

The same business key appears multiple times, even if other column values differ.

Example:

| order_id | amount | updated_at |
|---|---:|---|
| 102 | 700 | 10:00 |
| 102 | 750 | 11:00 |

These are two versions of the same order, not necessarily exact duplicates. The correct treatment depends on whether the target stores current state or historical versions.

**3. Ingestion Duplicates**

The same source record is ingested more than once.

**4. Join-Induced Duplicates**

A join creates multiple output rows because the join key matches multiple rows on the other side.

---

## 5. Common Causes of Duplicate Data

| Cause | Explanation |
|---|---|
| Pipeline retry | Previously written data is inserted again |
| Pipeline rerun | The same batch is appended again |
| Missing deduplication | Source contains repeated business keys |
| Join multiplication | Multiple matching rows produce extra output |
| Concurrent execution | Overlapping pipeline runs process the same data |
| Incorrect MERGE logic | Matching key is incomplete or incorrect |
| Source replay | Historical events are consumed again |
| Watermark issue | Previously processed records are read again |
| API pagination | Pages overlap or are requested repeatedly |
| At-least-once delivery | Records may be delivered more than once |

---

## 6. Step-by-Step Troubleshooting Process

Use a structured approach instead of immediately deleting records or rerunning the pipeline.

### Step 1: Identify the Affected Dataset

Record:
- Source table or file.
- Target table.
- Business key.
- Processing date or batch ID.
- Pipeline run ID.
- Expected record count.
- Actual record count.

### Step 2: Compare Source and Target Counts

Check whether the target has fewer or more records than expected.

```sql
SELECT COUNT(*) AS source_count
FROM source_orders;

SELECT COUNT(*) AS target_count
FROM target_orders;
```

Count differences are useful signals, but they do not identify which records are missing or duplicated.

### Step 3: Check Duplicate Business Keys

```sql
SELECT
    order_id,
    COUNT(*) AS record_count
FROM target_orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

This identifies repeated `order_id` values.

### Step 4: Find Missing Records

For a unique, non-NULL business key:

```sql
SELECT s.order_id
FROM source_orders s
LEFT JOIN target_orders t
    ON s.order_id = t.order_id
WHERE t.order_id IS NULL;
```

This identifies source keys that are absent from the target.

### Step 5: Inspect Pipeline Logs

Check:
- Failed activities or tasks.
- Retry attempts.
- Source extraction details.
- Transformation filters.
- Join conditions.
- Target write operations.
- Watermark values.
- Checkpoint information.

### Step 6: Identify the Root Cause

Determine whether the issue originated in:
- Source extraction.
- Data ingestion.
- Transformation.
- Deduplication.
- Target loading.
- Pipeline recovery.
- Downstream aggregation.

### Step 7: Fix and Recover

Apply the appropriate correction.

Examples:
- Reprocess missing intervals.
- Correct filtering conditions.
- Fix join logic.
- Deduplicate source records.
- Use idempotent target writes.
- Repair watermark handling.

### Step 8: Validate the Result

Check:
- Missing business keys.
- Duplicate business keys.
- Record counts.
- Important column values.
- Source-to-target reconciliation.
- Downstream aggregates.

---

## 7. Finding Missing Records Using SQL

### 7.1 Using LEFT JOIN

```sql
SELECT
    s.order_id,
    s.customer_id,
    s.amount
FROM source_orders s
LEFT JOIN target_orders t
    ON s.order_id = t.order_id
WHERE t.order_id IS NULL;
```

**Explanation:**

- Keep every source record.
- Match records using `order_id`.
- Return records without a target match.

This assumes `order_id` is a unique, non-NULL business key.

### 7.2 Using NOT EXISTS

```sql
SELECT
    s.order_id,
    s.customer_id,
    s.amount
FROM source_orders s
WHERE NOT EXISTS (
    SELECT 1
    FROM target_orders t
    WHERE t.order_id = s.order_id
);
```

`NOT EXISTS` is often a clear way to find source records without a matching target record.

### 7.3 Using EXCEPT

```sql
SELECT order_id
FROM source_orders

EXCEPT

SELECT order_id
FROM target_orders;
```

This returns source keys not present in the target.

**Important:** `EXCEPT` commonly uses set semantics and removes duplicates. Use grouped counts when duplicate multiplicity matters.

### 7.4 Compare Missing and Extra Records

Find records missing from the target:

```sql
SELECT order_id
FROM source_orders

EXCEPT

SELECT order_id
FROM target_orders;
```

Find records present in the target but absent from the source:

```sql
SELECT order_id
FROM target_orders

EXCEPT

SELECT order_id
FROM source_orders;
```

These comparisons help identify one-way discrepancies.

---

## 8. Finding Duplicate Records Using SQL

### 8.1 Identify Duplicate Business Keys

```sql
SELECT
    order_id,
    COUNT(*) AS duplicate_count
FROM orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

### 8.2 Identify Exact Duplicate Rows

```sql
SELECT
    order_id,
    customer_id,
    amount,
    order_date,
    COUNT(*) AS duplicate_count
FROM orders
GROUP BY
    order_id,
    customer_id,
    amount,
    order_date
HAVING COUNT(*) > 1;
```

This checks duplicates across the selected columns. It does not prove that every column in the physical table is identical.

### 8.3 Keep the Latest Record

Suppose the same order can have multiple versions and the target should retain only the latest version.

```sql
WITH ranked_orders AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY order_id
            ORDER BY updated_at DESC, ingestion_id DESC
        ) AS rn
    FROM orders
)
SELECT
    order_id,
    customer_id,
    amount,
    updated_at
FROM ranked_orders
WHERE rn = 1;
```

**Explanation:**

- `PARTITION BY order_id` groups versions of the same order.
- `ORDER BY updated_at DESC` puts the latest version first.
- `ingestion_id` provides a deterministic tie-breaker.
- `ROW_NUMBER()` assigns a sequence.
- `rn = 1` selects one record per order.

Use a tie-breaker that is unique and meaningful. Without deterministic ordering, the selected record may vary.

---

## 9. Troubleshooting Missing Data in PySpark

### 9.1 Find NULL Values

```python
from pyspark.sql import functions as F

df.filter(
    F.col("customer_id").isNull()
).show()
```

### 9.2 Find Missing Business Keys

```python
missing_orders = source_df.join(
    target_df.select("order_id"),
    on="order_id",
    how="left_anti"
)

missing_orders.show()
```

A left anti join returns source rows that do not have a matching target key.

### 9.3 Compare Record Counts

```python
source_count = source_df.count()
target_count = target_df.count()

print("Source:", source_count)
print("Target:", target_count)
print("Difference:", source_count - target_count)
```

Counts alone are not sufficient to establish correctness.

### 9.4 Find Invalid Records

```python
invalid_orders = source_df.filter(
    F.col("order_id").isNull() |
    F.col("amount").isNull() |
    (F.col("amount") < 0)
)
```

The rules should reflect actual business requirements.

---

## 10. Troubleshooting Duplicate Data in PySpark

### 10.1 Find Duplicate Business Keys

```python
duplicate_orders = (
    df.groupBy("order_id")
      .count()
      .filter(F.col("count") > 1)
)

duplicate_orders.show()
```

### 10.2 Remove Exact Duplicate Rows

```python
deduplicated_df = df.dropDuplicates()
```

This removes rows that are identical across all columns.

### 10.3 Deduplicate Using a Business Key

```python
deduplicated_df = df.dropDuplicates(
    ["order_id"]
)
```

This retains one row for each `order_id`, but does not guarantee which version is retained.

### 10.4 Keep the Latest Record

```python
from pyspark.sql import Window
from pyspark.sql import functions as F

window_spec = (
    Window
    .partitionBy("order_id")
    .orderBy(
        F.col("updated_at").desc(),
        F.col("ingestion_id").desc()
    )
)

deduplicated_df = (
    df.withColumn(
        "rn",
        F.row_number().over(window_spec)
    )
    .filter(F.col("rn") == 1)
    .drop("rn")
)
```

This keeps the latest record using a deterministic ordering, assuming the ordering columns resolve ties.

---

## 11. Incremental Load Issues

Incremental loading is a common source of missing and duplicate records.

An incremental pipeline processes only new or changed records instead of reloading the entire dataset.

### Example

Suppose the last successful watermark is:

```text
2026-10-24 23:59:59
```

The pipeline extracts records using:

```sql
SELECT *
FROM source_orders
WHERE updated_at > '2026-10-24 23:59:59';
```

This can create problems if:
- Multiple records share the same timestamp.
- A record arrives late with an older timestamp.
- The watermark is updated before the target write succeeds.
- Timestamp precision differs between source and target.
- A source record is updated without changing its timestamp.

### Better Approach: Composite Watermark

Where supported, use a combination of timestamp and unique key.

Conceptual SQL:

```sql
SELECT *
FROM source_orders
WHERE
    updated_at > :last_timestamp
    OR (
        updated_at = :last_timestamp
        AND order_id > :last_order_id
    )
ORDER BY updated_at, order_id;
```

This can handle multiple records sharing the same timestamp when the ordering key is stable and the source query is consistent.

### Important Recovery Rules

1. Store the last successfully processed position.
2. Write target data successfully before advancing the watermark.
3. Make target writes safe to repeat.
4. Handle late-arriving updates.
5. Use CDC where change history and deletes must be captured.
6. Reconcile source and target periodically.

Updating the watermark after a failed or incomplete target write can cause permanent data loss.

---

## 12. Join-Induced Missing or Duplicate Data

Joins are another common source of data discrepancies.

### 12.1 Inner Join Can Remove Records

Suppose the orders table contains orders for customers who are not present in the customer table.

```sql
SELECT
    o.order_id,
    c.customer_name
FROM orders o
INNER JOIN customers c
    ON o.customer_id = c.customer_id;
```

An inner join removes orders without a matching customer.

If all orders must be retained, consider a left join:

```sql
SELECT
    o.order_id,
    c.customer_name
FROM orders o
LEFT JOIN customers c
    ON o.customer_id = c.customer_id;
```

### 12.2 One-to-Many Join Can Create Duplicates

Suppose one customer has multiple address records.

```sql
SELECT
    o.order_id,
    c.customer_id,
    c.address
FROM orders o
JOIN customer_addresses c
    ON o.customer_id = c.customer_id;
```

An order may appear multiple times because several addresses match.

### Troubleshooting Steps

- Check uniqueness of join keys.
- Identify one-to-many relationships.
- Confirm the intended grain of the output.
- Deduplicate or aggregate the right-side dataset when appropriate.
- Avoid joining fact tables at incompatible grains without aggregation.

A repeated order ID after a join does not automatically mean the source contained duplicate orders.

---

## 13. Late-Arriving Data and Missing Records

Late-arriving data occurs when a record becomes available after the expected processing time.

Example:

An order is created at 10:00 AM but reaches the data platform at 11:30 AM.

A pipeline that processes data only once at 11:00 AM may miss it.

### Common Solutions

**1. Lookback Window**

Re-read a recent time range during each run.

Example:

```sql
SELECT *
FROM source_orders
WHERE updated_at >= :last_watermark_minus_2_days;
```

This is a conceptual example; use the appropriate date arithmetic for the SQL engine.

**2. Change Data Capture (CDC)**

Capture inserts, updates, and deletes from the source.

**3. Periodic Reconciliation**

Compare source and target records to identify missing data.

**4. Event-Time Watermarks**

For streaming pipelines, use event-time watermarks to manage late events and state retention.

**Important:** A streaming event-time watermark and a batch extraction watermark are different concepts.

---

## 14. Safe Recovery of Missing Data

When records are missing, reprocessing is often required.

### Recovery Process

```text
Identify Missing Records
          |
          v
Find Affected Date / Batch
          |
          v
Identify Root Cause
          |
          v
Correct Pipeline Logic
          |
          v
Reprocess Missing Data
          |
          v
Validate Source and Target
          |
          v
Confirm Downstream Results
```

### Recovery Options

| Method | When to Use |
|---|---|
| Rerun failed task | Task failed before producing valid output |
| Reprocess date partition | A particular date or partition is incomplete |
| Backfill | Multiple historical intervals need processing |
| CDC replay | Changes need to be recovered from a change stream |
| Source reload | Existing target data is incomplete or corrupted |
| Reconciliation-based repair | Differences are identified by comparing source and target |

### Important

Before reprocessing:
- Confirm which records are missing.
- Check whether the target already contains some of them.
- Use idempotent writes.
- Avoid overwriting unrelated data.
- Validate the affected interval after recovery.

---

## 15. Safe Recovery of Duplicate Data

Duplicate removal must follow business rules.

### Recovery Process

1. Identify the affected business key.
2. Determine why duplicates were created.
3. Define which record should be retained.
4. Check downstream dependencies.
5. Back up or preserve affected data according to policy.
6. Apply the approved correction.
7. Fix the ingestion or transformation logic.
8. Reconcile the final dataset.

### Example: Deduplicate Before MERGE

```sql
MERGE INTO target_orders AS target
USING (
    SELECT
        order_id,
        customer_id,
        amount,
        updated_at
    FROM (
        SELECT
            *,
            ROW_NUMBER() OVER (
                PARTITION BY order_id
                ORDER BY updated_at DESC, ingestion_id DESC
            ) AS rn
        FROM source_orders
    ) ranked
    WHERE rn = 1
) AS source

ON target.order_id = source.order_id

WHEN MATCHED THEN UPDATE SET
    target.customer_id = source.customer_id,
    target.amount = source.amount,
    target.updated_at = source.updated_at

WHEN NOT MATCHED THEN INSERT (
    order_id,
    customer_id,
    amount,
    updated_at
)
VALUES (
    source.order_id,
    source.customer_id,
    source.amount,
    source.updated_at
);
```

This example assumes that the latest source record should represent the current order state.

The source must be deduplicated because multiple source rows matching the same target row can make a MERGE ambiguous or fail, depending on the engine and operation.

---

## 16. Data Reconciliation

Reconciliation checks whether source and target data agree according to defined rules.

### Common Reconciliation Checks

| Check | Purpose |
|---|---|
| Record count | Compare total records |
| Key comparison | Identify missing or extra business keys |
| Duplicate check | Find repeated business keys |
| Aggregate comparison | Compare totals and amounts |
| Column comparison | Compare important field values |
| Partition comparison | Validate individual date or batch partitions |
| Hash comparison | Detect differences across normalized records |

### Example: Count Reconciliation

```sql
SELECT
    'Source' AS dataset,
    COUNT(*) AS total_records
FROM source_orders

UNION ALL

SELECT
    'Target' AS dataset,
    COUNT(*) AS total_records
FROM target_orders;
```

### Example: Aggregate Reconciliation

```sql
SELECT
    SUM(amount) AS total_amount
FROM source_orders;

SELECT
    SUM(amount) AS total_amount
FROM target_orders;
```

Matching counts and totals do not prove that every record matches. Combine aggregate checks with key-level and column-level comparisons.

For financial data, use suitable decimal types rather than floating-point types.

---

## 17. Prevention Best Practices

### Prevent Missing Data

1. Use reliable incremental extraction logic.
2. Update watermarks only after successful target writes.
3. Handle late-arriving data.
4. Validate source and target record counts.
5. Use appropriate join types.
6. Validate schema and required fields.
7. Monitor pipeline freshness.
8. Use CDC when updates and deletes must be captured.
9. Reconcile important datasets periodically.

### Prevent Duplicate Data

1. Define business keys clearly.
2. Deduplicate source records using deterministic rules.
3. Use idempotent target writes.
4. Avoid uncontrolled append operations during retries.
5. Prevent overlapping pipeline runs where necessary.
6. Validate join-key uniqueness.
7. Use correct MERGE conditions.
8. Track processed batches or source files.
9. Monitor duplicate business keys.

### General Rule

Do not use `DISTINCT` as a universal fix.

It can remove exact duplicate rows, but it cannot determine which business record is correct when rows differ.

---

## 18. Practical Interview Scenario

**Question: Your source table contains 100,000 orders, but the target table contains only 98,000 records. How would you investigate?**

### Answer

First, I would compare the source and target counts for the same processing period and filters.

Then I would identify the missing business keys using a left anti join or `NOT EXISTS`.

I would investigate:

1. Whether the ingestion task completed successfully.
2. Whether incremental watermark logic skipped records.
3. Whether records arrived late.
4. Whether transformation filters removed valid records.
5. Whether an inner join removed unmatched records.
6. Whether schema conversion rejected records.
7. Whether the target write completed successfully.

After identifying the root cause, I would reprocess the missing records using an idempotent approach.

Finally, I would compare source and target keys, validate record counts and aggregates, and add monitoring to detect similar discrepancies.

---

## 19. Common Interview Questions and Answers

### Q1. What is missing data?

Expected records or values that are absent from a dataset or target system.

### Q2. What is duplicate data?

Multiple occurrences of the same logical business record when only one occurrence is expected.

### Q3. What are common causes of missing records?

Incorrect filters, failed ingestion, watermark issues, late-arriving data, schema mismatch, and inner joins.

### Q4. What are common causes of duplicate records?

Retries, reruns, repeated source events, missing deduplication, incorrect MERGE conditions, and one-to-many joins.

### Q5. How do you find missing records using SQL?

Use a LEFT JOIN with a NULL check, `NOT EXISTS`, or `EXCEPT` to identify source keys absent from the target.

### Q6. How do you identify duplicate records?

Use `GROUP BY` with `COUNT(*)` and filter using `HAVING COUNT(*) > 1`.

### Q7. What is the difference between exact duplicates and business-key duplicates?

Exact duplicates match across all selected columns. Business-key duplicates share the same business key but may have different values in other columns.

### Q8. How do you keep the latest record?

Use `ROW_NUMBER()` partitioned by the business key and ordered by update timestamp and a deterministic tie-breaker.

### Q9. Why can an inner join cause missing data?

It returns only matching records, so unmatched rows from either side are excluded.

### Q10. Why can a join create duplicates?

A join can produce multiple output rows when a key matches multiple rows on the other side.

### Q11. How can incremental loading cause missing records?

Incorrect watermarks, timestamp precision, late updates, and premature watermark advancement can skip records.

### Q12. How do you prevent duplicates during retries?

Use idempotent writes, deterministic deduplication, correct business keys, and safe MERGE or upsert logic.

### Q13. Is `dropDuplicates()` enough to solve duplicate data?

Not always. It does not guarantee which record is retained when rows share a key but contain different values.

### Q14. Is matching source and target record count enough?

No. Different records can produce identical counts. Key-level and value-level reconciliation is also needed.

### Q15. How do you recover missing historical records?

Identify the affected interval, fix the root cause, backfill or reprocess the missing data, and validate the target.

### Q16. How do you handle late-arriving data?

Use lookback windows, CDC, periodic reconciliation, or appropriate streaming event-time handling.

### Q17. What is the difference between a batch watermark and a streaming watermark?

A batch watermark tracks extraction progress, often using a timestamp or key. A streaming event-time watermark tracks event-time progress for stateful processing and late-event handling.

### Q18. Why should you avoid using DISTINCT everywhere?

It may hide data problems, remove legitimate repeated values, and does not define which version of a business record should be retained.

---

## 20. Quick Revision

| Concept | Key Meaning |
|---|---|
| Missing Records | Expected rows are absent |
| Missing Values | Existing rows contain NULL or blank values |
| Duplicate Records | Same logical record appears more than once |
| Business Key | Key identifying a business entity or event |
| LEFT ANTI JOIN | Returns rows without a match on the other side |
| NOT EXISTS | Checks whether a matching record does not exist |
| ROW_NUMBER | Assigns sequential row numbers within a window |
| Deduplication | Removes or selects records according to defined rules |
| Watermark | Tracks incremental extraction progress |
| CDC | Captures source inserts, updates, and deletes |
| Reconciliation | Compares source and target data |
| Idempotency | Makes repeated processing safe under defined conditions |
| Backfill | Processes historical data intervals |

### Remember These Five Points

1. Record-count comparison is a starting point, not complete reconciliation.
2. Use business keys to identify missing and duplicate records.
3. Check joins and incremental watermarks carefully.
4. Use deterministic deduplication and idempotent writes.
5. Validate the final data after every recovery operation.

---

## 21. Self-Assessment

Try answering these questions without looking at the notes.

1. What is the difference between missing records and missing values?
2. What are the common causes of missing data?
3. What are the common causes of duplicate data?
4. How do you identify missing records using a LEFT JOIN?
5. How do you use `NOT EXISTS` to find missing records?
6. How do you identify duplicate business keys?
7. How do you keep the latest record using `ROW_NUMBER()`?
8. How can an inner join remove records?
9. How can a one-to-many join create duplicate output rows?
10. How can incorrect watermark logic cause missing records?
11. How can retries create duplicate records?
12. What is the difference between exact and business-key duplicates?
13. How do you deduplicate records in PySpark?
14. Why is `dropDuplicates()` not always sufficient?
15. How do you safely recover missing records?
16. How do you prevent duplicates during pipeline reruns?
17. What is the difference between batch and streaming watermarks?
18. How do you reconcile source and target datasets?
19. Why are record counts alone insufficient?
20. Design a troubleshooting strategy for a pipeline that produces both missing and duplicate orders.

**Interview readiness:** Be able to write SQL for missing records, duplicate detection, and latest-record selection. Also be prepared to explain how joins, retries, incremental watermarks, and late-arriving data create data discrepancies.

