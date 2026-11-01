# Incremental Loads in Data Engineering

## 1. Introduction

**Incremental loading is a data ingestion technique that processes only new or changed records instead of repeatedly processing the entire source dataset.**

It is commonly used in production data pipelines to reduce processing time, compute usage, and unnecessary data movement.

### Example

Suppose an orders table contains 10 million records.

Every day, only 50,000 new or updated records need to be processed.

Instead of reading all 10 million records, an incremental pipeline identifies and processes those 50,000 changes.

```text
Source Table
10 Million Records
       |
       v
Identify New or Changed Records
       |
       v
50,000 Records
       |
       v
Transform and Validate
       |
       v
Write to Target
```

**Key idea:** Incremental loading avoids unnecessary full-table processing by tracking changes.

---

## 2. Why Do We Need Incremental Loading?

As datasets grow, full loading becomes increasingly expensive.

Incremental loading helps to:

- Reduce repeated source reads.
- Reduce processing time.
- Lower compute and storage I/O costs.
- Support frequent data refreshes.
- Handle large transactional datasets efficiently.
- Improve pipeline scalability.

### Example

| Execution | Total Source Records | Records Processed |
|---|---:|---:|
| Initial load | 10,000,000 | 10,000,000 |
| Day 1 | 10,050,000 | 50,000 |
| Day 2 | 10,090,000 | 40,000 |
| Day 3 | 10,130,000 | 40,000 |

The initial execution loads the historical data. Subsequent executions process only identified changes.

---

## 3. Types of Incremental Loading

The extraction method depends on how the source system identifies changes.

### 3.1 Timestamp-Based Incremental Loading

Uses a timestamp column to identify newly inserted or updated records.

Common columns:
- `created_at`
- `updated_at`
- `modified_date`

Example:

```sql
SELECT *
FROM orders
WHERE updated_at > '2026-10-24 00:00:00';
```

If the last successful watermark is October 24, the pipeline selects records modified after that timestamp.

**Advantages:**
- Easy to understand and implement.
- Suitable for sources with reliable modification timestamps.
- Can capture both inserts and updates.

**Limitations:**
- Depends on accurate timestamps.
- Can miss late-arriving updates.
- Does not normally detect physical deletes.
- Requires careful watermark management.

### 3.2 ID-Based Incremental Loading

Uses an increasing identifier to identify new records.

Example:

```sql
SELECT *
FROM orders
WHERE order_id > 50000;
```

If the last successfully processed order ID is 50,000, the pipeline selects higher IDs.

**Advantages:**
- Simple to implement.
- Efficient when IDs increase reliably.
- Suitable for append-only datasets.

**Limitations:**
- Does not detect updates to existing records.
- May miss records if ID assignment and transaction commit order are not reliable.
- Does not detect physical deletes.

### 3.3 Change Data Capture (CDC)

CDC captures changes made to a source database.

Common change types:

- INSERT
- UPDATE
- DELETE

CDC can capture changes using database transaction logs or other source-supported mechanisms.

```text
Source Database
       |
       v
CDC Mechanism
       |
       v
Change Events
       |
       v
Data Pipeline
       |
       v
Target Table
```

**Advantages:**
- Captures inserts, updates, and deletes when supported.
- Avoids repeatedly scanning the complete source table.
- Useful for frequently changing transactional systems.

**Limitations:**
- Requires CDC support and configuration.
- Needs offset or position tracking.
- Requires handling duplicate and out-of-order events where applicable.
- Source log retention and connector behavior matter.

### 3.4 File-Based Incremental Loading

Processes newly arrived files rather than repeatedly reading all historical files.

Example:

```text
orders_2026_10_24.csv
orders_2026_10_25.csv
orders_2026_10_26.csv
```

A pipeline processes newly discovered files.

In Databricks, Auto Loader can discover and ingest new files using Structured Streaming.

**Important:** File discovery is not the same as business-level change detection. Auto Loader does not automatically determine whether two records represent the same order or whether an existing order needs an update.

---

## 4. Watermark Management

A watermark is a stored progress value that indicates how far a batch incremental pipeline has successfully processed source data.

It helps determine which records should be extracted in the next execution.

### Example

Suppose the last successful watermark is:

```text
2026-10-24 00:00:00
```

The next extraction uses:

```sql
SELECT *
FROM orders
WHERE updated_at > '2026-10-24 00:00:00';
```

After the target write and required validation succeed, the pipeline advances the watermark.

### Watermark Workflow

```text
Read Previous Watermark
          |
          v
Extract Changed Records
          |
          v
Transform and Validate
          |
          v
Write to Target
          |
          v
Verify Target Write
          |
          v
Update Watermark
```

### Rules for Watermark Management

1. Read the last successfully committed watermark.
2. Extract the required changes.
3. Transform and validate the records.
4. Write the target successfully.
5. Perform required validation.
6. Advance the watermark only after successful processing.
7. Make target writes safe to repeat.

### What Happens If the Pipeline Fails?

Suppose the target write succeeds, but updating the watermark fails.

The next execution may read the same records again.

If the target uses an unsafe append operation, duplicate records can appear.

Use idempotent writes or suitable transaction-aware progress handling.

**Important:** A control table stores progress, but it does not automatically make the pipeline retry-safe.

---

## 5. Handling Inserts, Updates, and Deletes

A reliable incremental pipeline must understand the type of source changes it needs to process.

Consider this source record:

| order_id | customer_id | amount |
|---|---|---:|
| 101 | C01 | 500 |

Later, the order amount changes:

| order_id | customer_id | amount |
|---|---|---:|
| 101 | C01 | 700 |

If the pipeline uses append mode, it may create two records for the same order.

For a current-state target table, use an upsert strategy.

### Delta MERGE Example

```sql
MERGE INTO silver.orders AS target
USING staging.orders AS source
ON target.order_id = source.order_id

WHEN MATCHED THEN
    UPDATE SET
        target.customer_id = source.customer_id,
        target.amount = source.amount

WHEN NOT MATCHED THEN
    INSERT (
        order_id,
        customer_id,
        amount
    )
    VALUES (
        source.order_id,
        source.customer_id,
        source.amount
    );
```

This example handles inserts and updates.

Before MERGE, ensure that the source has at most one applicable record per target key. Deduplicate source records using a deterministic rule, such as the latest update timestamp and a tie-breaker.

### Handling Deletes

Timestamp-based extraction usually cannot identify physical deletes because deleted records are no longer present in the source table.

Common solutions:

- CDC delete events.
- Soft-delete flags.
- Source audit tables.
- Periodic source-to-target reconciliation.

For example, if CDC reports that `order_id = 101` was deleted, the pipeline can apply the corresponding delete to the target.

---

## 6. Late-Arriving Data

Late-arriving data is data that becomes available after the pipeline has already processed the corresponding time period or watermark.

### Example

The pipeline has processed records through October 24.

On October 26, an update arrives with an `updated_at` value of October 23.

A filter such as:

```sql
WHERE updated_at > '2026-10-24'
```

will miss that record.

### Solution: Lookback Window

A lookback window re-reads a recent period before the previous watermark.

Example:

If the previous watermark is October 24 and the lookback period is two days, the pipeline reads from October 22.

```python
from pyspark.sql import functions as F

last_watermark = "2026-10-24 00:00:00"

lookback_start = F.expr(
    "timestamp('2026-10-24 00:00:00') - INTERVAL 2 DAYS"
)

incremental_df = source_df.filter(
    F.col("updated_at") >= lookback_start
)
```

The target must safely handle repeated records using deduplication and an idempotent write strategy.

A lookback window reduces the risk of missing late changes but cannot guarantee detection of arbitrarily late records.

Other solutions include CDC and periodic reconciliation.

---

## 7. Incremental Loading with PySpark

### Scenario

Suppose a source orders dataset contains:
- `order_id`
- `customer_id`
- `amount`
- `updated_at`

We want to select records modified after the previous successful watermark.

```python
from pyspark.sql import functions as F

# Previous successful watermark
last_watermark = "2026-10-24 00:00:00"

# Read source data
source_df = (
    spark.read
    .format("parquet")
    .load("/data/source/orders")
)

# Select changed records
incremental_df = source_df.filter(
    F.col("updated_at") >
    F.to_timestamp(F.lit(last_watermark))
)

# Basic validation
valid_df = incremental_df.filter(
    F.col("order_id").isNotNull() &
    F.col("amount").isNotNull()
)

# Remove duplicate business keys using a deterministic rule
from pyspark.sql.window import Window

window_spec = (
    Window
    .partitionBy("order_id")
    .orderBy(
        F.col("updated_at").desc()
    )
)

deduplicated_df = (
    valid_df
    .withColumn(
        "row_num",
        F.row_number().over(window_spec)
    )
    .filter(F.col("row_num") == 1)
    .drop("row_num")
)
```

This example selects changed records, validates required fields, and keeps the latest record per order.

If multiple records have the same `updated_at`, add a reliable tie-breaker to the window ordering to make the selection deterministic.

The example stops before writing to the target. A production pipeline should:
1. Apply a retry-safe target write.
2. Validate the committed output.
3. Update the watermark only after successful processing.

---

## 8. Incremental Loading with Delta Lake

Delta Lake supports `MERGE`, which is commonly used to apply incremental inserts and updates to a target table.

### Example

```sql
MERGE INTO silver.orders AS target
USING staging.orders AS source
ON target.order_id = source.order_id

WHEN MATCHED THEN
    UPDATE SET *

WHEN NOT MATCHED THEN
    INSERT *;
```

This concise example assumes the source and target have compatible schemas and the source is deduplicated by `order_id`.

For production pipelines, explicitly specify columns when schemas may differ or when only selected columns should be updated.

### Why MERGE Is Useful

- Inserts new records.
- Updates existing records.
- Maintains a current-state target table.
- Supports retry-safe designs when source keys and write conditions are handled correctly.

**Important:** MERGE is not automatically a complete exactly-once pipeline solution. Source deduplication, concurrency handling, and progress tracking are still important.

---

## 9. Incremental Loading with Databricks Auto Loader

Auto Loader is used to incrementally ingest newly arriving files from cloud storage.

Example:

```python
from pyspark.sql import functions as F

source_df = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option(
        "cloudFiles.schemaLocation",
        "/checkpoints/orders/schema"
    )
    .load("/data/incoming/orders")
)

query = (
    source_df.writeStream
    .format("delta")
    .option(
        "checkpointLocation",
        "/checkpoints/orders/stream"
    )
    .outputMode("append")
    .toTable("bronze.orders")
)
```

### Important Options

| Option | Purpose |
|---|---|
| `cloudFiles` | Enables Auto Loader |
| `cloudFiles.format` | Specifies the input file format |
| `cloudFiles.schemaLocation` | Stores schema-tracking information |
| `checkpointLocation` | Stores streaming query progress and state |
| `toTable()` | Writes the streaming output to a table |

### Important Points

- Auto Loader discovers new files.
- Schema evolution behavior depends on the configured mode and runtime.
- Schema location and checkpoint location serve different purposes.
- A separate checkpoint should be used for each independent query.
- Auto Loader does not automatically deduplicate business records.
- Do not delete checkpoints casually.

---

## 10. Incremental Loading Best Practices

### 1. Use Reliable Change Detection

Choose timestamps, CDC, IDs, or file discovery based on source capabilities.

### 2. Maintain Watermark Carefully

Advance progress only after successful target processing and required validation.

### 3. Make Writes Idempotent

Repeated processing should not create unintended duplicate results.

### 4. Deduplicate Source Records

Use stable business keys and deterministic ordering rules.

### 5. Handle Deletes Explicitly

Use CDC, soft deletes, audit records, or reconciliation.

### 6. Handle Late-Arriving Data

Use lookback windows, CDC, or historical reconciliation as appropriate.

### 7. Validate Data

Check record counts, nulls, duplicate keys, and business rules.

### 8. Monitor Pipeline Execution

Track:
- Records extracted.
- Records inserted.
- Records updated.
- Records rejected.
- Watermark values.
- Processing duration.
- Failed batches.

### 9. Keep Recovery in Mind

A failed pipeline should be able to retry or reprocess data without corrupting the target.

### 10. Test Edge Cases

Test:
- Empty incremental batches.
- Duplicate source records.
- Late-arriving updates.
- Physical deletes.
- Failure after target write.
- Failure before watermark update.
- Schema changes.

---

## 11. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| Incremental loading processes only inserts | It can process updates and deletes when supported by the change-detection mechanism |
| Increasing IDs detect every change | They generally detect new records, not updates to existing records |
| Timestamp filters capture all changes | Late or unreliable timestamps can cause missed records |
| Append is always suitable | Updates can create duplicate logical records |
| MERGE automatically deduplicates the source | Source deduplication is still important |
| Watermark can be updated before the target write | This can cause data loss if the target write fails |
| A control table guarantees exactly-once processing | It tracks progress but does not coordinate all writes automatically |
| Auto Loader detects business-level updates | It discovers files, not business changes inside records |
| Lookback windows capture every late record | They only cover the configured overlapping period |
| Incremental loading automatically detects deletes | Delete handling needs an explicit mechanism |

---

## 12. Interview Questions and Answers

### Q1. What is incremental loading?

Incremental loading processes only new or changed records since the previous successful execution instead of reading the complete source dataset.

### Q2. What are the common incremental loading techniques?

Timestamp-based loading, ID-based loading, CDC, and file-based incremental ingestion.

### Q3. What is a watermark?

A batch watermark is a stored progress value used to identify which source records should be extracted in the next execution.

### Q4. How do you handle updates in incremental loading?

I use a reliable change-detection mechanism and apply an upsert operation, such as Delta MERGE, when maintaining a current-state target table.

### Q5. How do you handle deletes?

I use CDC delete events, soft-delete flags, audit tables, or periodic source-to-target reconciliation.

### Q6. What happens if a pipeline fails after writing data but before updating the watermark?

The same records may be extracted again. I make the target write idempotent or transaction-aware so that retrying does not create duplicate logical results.

### Q7. How do you handle late-arriving data?

I can use a lookback window, CDC, or periodic reconciliation. The target must safely handle records that are processed more than once.

### Q8. What is the difference between a watermark and a checkpoint?

A batch watermark commonly tracks extraction progress. A Structured Streaming checkpoint stores query progress, metadata, and state needed for recovery.

### Q9. What is the role of Delta MERGE in incremental loading?

MERGE applies inserts and updates to a target table based on matching conditions. It is useful for maintaining current-state tables.

### Q10. How does Auto Loader support incremental ingestion?

Auto Loader discovers and processes newly arriving files from cloud storage. It uses schema tracking and streaming checkpoints to support ongoing ingestion and recovery.

### Q11. Can incremental loading miss records?

Yes. Unreliable timestamps, late-arriving data, physical deletes, incorrect progress updates, and source change-tracking limitations can cause missed records.

### Q12. How would you design a reliable incremental pipeline?

I would use a reliable change-detection mechanism, maintain successful progress, validate and deduplicate incoming records, write using an idempotent strategy, handle deletes and late data, and update progress only after successful target processing.

---

## 13. Quick Revision

| Concept | Remember |
|---|---|
| Incremental load | Process only identified changes |
| Timestamp-based | Uses modification timestamps |
| ID-based | Uses increasing identifiers |
| CDC | Captures source changes |
| File-based ingestion | Processes newly arriving files |
| Watermark | Tracks batch extraction progress |
| Checkpoint | Stores streaming progress and state |
| Lookback window | Re-reads a recent historical period |
| MERGE | Applies inserts and updates |
| Deduplication | Removes repeated business keys using a defined rule |
| Idempotency | Makes repeated operations safe |
| Reconciliation | Compares source and target data |

### One-Minute Interview Answer

Incremental loading is a technique in which a pipeline processes only new or changed records instead of reading the entire source dataset every time.

Common approaches include timestamp-based extraction, increasing IDs, CDC, and file-based ingestion. I would maintain a watermark to track successful progress, validate and deduplicate incoming records, and use an idempotent write strategy such as Delta MERGE when appropriate.

I would also handle late-arriving data and physical deletes separately. Finally, I would update the watermark only after successful target processing and validation so that pipeline failures can be recovered safely.

---

## 14. Self-Assessment

- [ ] Define incremental loading.
- [ ] Explain why it is useful for large datasets.
- [ ] Compare timestamp-based and ID-based loading.
- [ ] Explain CDC.
- [ ] Explain file-based incremental ingestion.
- [ ] Describe watermark management.
- [ ] Differentiate batch watermarks and streaming checkpoints.
- [ ] Handle inserts, updates, and deletes.
- [ ] Explain late-arriving data and lookback windows.
- [ ] Write a PySpark incremental filter.
- [ ] Explain Delta MERGE.
- [ ] Explain how Auto Loader discovers new files.
- [ ] Design a retry-safe incremental pipeline.

**Interview readiness:** Be able to explain how you would incrementally load a large orders table every day, capture updates, handle deletes, and recover safely if the pipeline fails after writing to the target.