# Incremental Data Ingestion

## 1. What Is Incremental Ingestion?

**Incremental ingestion** is the process of loading only new or changed data from a source system instead of loading the entire dataset every time.

It helps reduce:

- Processing time.
- Compute costs.
- Data transfer.
- Unnecessary reads and writes.
- Overall pipeline execution time.

**Example:**

Suppose an e-commerce application has 10 million customer records.

Every day, only 5,000 new or updated records are generated.

Instead of loading all 10 million records every day, an incremental pipeline loads only those 5,000 changed records.

### Full Load vs Incremental Load

| Full Load | Incremental Load |
|---|---|
| Loads the complete dataset | Loads only new or changed data |
| Processes historical data repeatedly | Processes only the required changes |
| Can consume more compute | Usually more efficient for recurring loads |
| Simpler in some scenarios | Requires tracking changes or ingestion progress |
| Useful for initial loading | Useful for regular production pipelines |

**Remember:** Incremental ingestion is commonly used after the initial full load.

## 2. Why Do We Need Incremental Ingestion?

Consider a customer table containing 10 million records.

Every day:

- 2,000 new customers are added.
- 3,000 existing customers are updated.
- The remaining records do not change.

If we perform a full load every day, we unnecessarily process millions of unchanged records.

With incremental ingestion, we process only the new and changed records.

### Benefits

1. Faster pipeline execution.
2. Reduced compute consumption.
3. Less unnecessary data movement.
4. Better scalability.
5. More frequent data availability.
6. Efficient handling of large datasets.

## 3. Full Load vs Incremental Load: Example

Suppose the source table initially contains:

| customer_id | name | city | updated_at |
|---|---|---|---|
| 101 | Rahul | Kolkata | 2026-10-29 |
| 102 | Amit | Delhi | 2026-10-29 |
| 103 | Priya | Mumbai | 2026-10-29 |

On the next day, the source receives these changes:

| customer_id | name | city | updated_at |
|---|---|---|---|
| 102 | Amit | Ranchi | 2026-10-30 |
| 104 | Neha | Pune | 2026-10-30 |

The changes are:

- Customer 102 has an updated city.
- Customer 104 is a new customer.
- Customers 101 and 103 remain unchanged.

An incremental pipeline processes only customers 102 and 104.

The final target table becomes:

| customer_id | name | city | updated_at |
|---|---|---|---|
| 101 | Rahul | Kolkata | 2026-10-29 |
| 102 | Amit | Ranchi | 2026-10-30 |
| 103 | Priya | Mumbai | 2026-10-29 |
| 104 | Neha | Pune | 2026-10-30 |

The existing customer is updated, and the new customer is inserted.

## 4. Common Methods of Incremental Ingestion

There are several ways to identify new or changed records.

### A. Timestamp-Based Incremental Loading

This method uses a timestamp column such as:

- `updated_at`
- `modified_date`
- `last_updated_time`

The pipeline stores the timestamp of the last successful ingestion and reads records updated after that point.

Example:

```sql
SELECT *
FROM source_customers
WHERE updated_at > '2026-10-29 23:59:59';
```

If the previous successful load ended on October 29, the pipeline selects records updated afterward.

**Advantages:**

- Simple to implement.
- Useful when the source maintains reliable update timestamps.
- Supports both new and updated records.

**Limitations:**

- Depends on accurate timestamps.
- Late-arriving updates can be missed if the watermark is managed incorrectly.
- Timestamp precision and timezone differences can create problems.

### B. ID-Based Incremental Loading

This method uses an increasing identifier.

Example:

| order_id | amount |
|---|---:|
| 1001 | 500 |
| 1002 | 700 |
| 1003 | 300 |

If the last successfully processed ID is 1002, the next load can retrieve:

```sql
SELECT *
FROM source_orders
WHERE order_id > 1002;
```

This retrieves order 1003.

**Limitation:** ID-based loading is generally suitable for append-only data. It does not detect updates to existing records unless another change-tracking mechanism is used.

### C. Change Data Capture (CDC)

CDC identifies changes made to a source database.

It can capture:

- INSERT operations.
- UPDATE operations.
- DELETE operations.

Example:

Suppose a customer changes their city from Kolkata to Delhi.

CDC can capture the update rather than requiring the pipeline to scan the entire customer table.

CDC is useful when the source provides a reliable change stream or transaction-log-based change feed.

**Important:** CDC captures changes; the destination pipeline must still apply them correctly, including handling update order and delete events.

### D. File-Based Incremental Ingestion

In cloud storage, new files may arrive periodically.

The pipeline processes newly discovered files rather than reading every historical file again.

Databricks Auto Loader is commonly used for this purpose.

Example:

```text
orders/
    orders_01.json
    orders_02.json
    orders_03.json
```

Initially, the pipeline processes the first two files.

Later, `orders_03.json` arrives.

Auto Loader discovers and processes the new file while maintaining ingestion progress through checkpointing.

**Important:** File-level incremental discovery and record-level deduplication are different. A new file can still contain duplicate records.

## 5. Watermark in Incremental Ingestion

A watermark is a saved progress value used to determine how far the pipeline has successfully processed.

In batch incremental ingestion, the watermark is often a timestamp or increasing ID.

Example:

```text
Last successful watermark:
2026-10-29 23:59:59

Next load:
Records with updated_at greater than the watermark
```

After successfully processing the new records, the pipeline advances the watermark.

### Watermark Flow

```text
Read Previous Watermark
         |
         v
Extract New or Changed Records
         |
         v
Transform Data
         |
         v
Write to Target
         |
         v
Verify Successful Commit
         |
         v
Save New Watermark
```

**Important:** This batch-ingestion watermark is different from the event-time watermark used in Spark Structured Streaming.

- Batch watermark: Tracks ingestion progress, often using a timestamp or ID.
- Streaming watermark: Manages event-time state and late-arriving data in stateful streaming operations.

Do not confuse them in interviews.

## 6. Implementing Timestamp-Based Incremental Ingestion

Suppose a source database contains customer records with an `updated_at` column.

We want to load only records modified after the previous successful run.

### Step 1: Read the Previous Watermark

```python
last_watermark = "2026-10-29 23:59:59"
```

In production, this value should be read from a persistent control or audit table.

### Step 2: Extract Incremental Data

```python
incremental_df = (
    spark.read
    .format("jdbc")
    .option("url", jdbc_url)
    .option("dbtable", "customers")
    .option("user", username)
    .option("password", password)
    .load()
    .filter(
        F.col("updated_at") > F.lit(last_watermark)
    )
)
```

This is an illustrative example. For large source tables, push the filter into the source SQL query so the source database does not need to transfer the entire table.

### Step 3: Write to a Staging Table

```python
incremental_df.write \
    .format("delta") \
    .mode("overwrite") \
    .save(staging_path)
```

The staging table or path holds the current incremental batch.

### Step 4: Merge Into the Target Table

```sql
MERGE INTO target_customers AS target
USING staging_customers AS source
ON target.customer_id = source.customer_id

WHEN MATCHED THEN
    UPDATE SET *

WHEN NOT MATCHED THEN
    INSERT *;
```

This updates existing customers and inserts new customers.

### Step 5: Update the Watermark

Only after the target write has succeeded and the batch has been verified should the pipeline advance its saved watermark.

For example:

```text
Previous watermark:
2026-10-29 23:59:59

New watermark:
2026-10-30 23:59:59
```

**Production consideration:** The watermark update and target-table write are separate operations unless explicitly coordinated. Design the pipeline so a failure between them can be safely retried.

## 7. Why Do We Need a Control Table?

A control table stores metadata about ingestion pipelines.

It can track:

- Pipeline name.
- Source table.
- Last successful watermark.
- Last execution time.
- Batch ID.
- Processing status.
- Number of records processed.

### Example Control Table

| pipeline_name | last_watermark | status |
|---|---|---|
| customers_pipeline | 2026-10-29 23:59:59 | SUCCESS |
| orders_pipeline | 2026-10-30 10:00:00 | SUCCESS |
| payments_pipeline | 2026-10-30 11:00:00 | FAILED |

### Example SQL

```sql
CREATE TABLE pipeline_control (
    pipeline_name STRING,
    last_watermark TIMESTAMP,
    status STRING,
    updated_at TIMESTAMP
)
USING DELTA;
```

The pipeline reads its previous watermark before extraction and updates the control record after successful processing.

**Remember:** A control table is metadata management. It does not automatically make the pipeline retry-safe. The data write and watermark update must be designed to handle failures correctly.

## 8. Incremental Ingestion Using Auto Loader

Auto Loader is a Databricks feature for incrementally discovering and ingesting files from cloud storage.

It is useful when files arrive continuously or periodically.

### Example

```python
df = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.schemaLocation", schema_path)
    .load(source_path)
)

query = (
    df.writeStream
    .format("delta")
    .option("checkpointLocation", checkpoint_path)
    .trigger(availableNow=True)
    .toTable("bronze.orders")
)
```

This pipeline:

1. Reads incoming JSON files.
2. Uses Auto Loader to discover new files.
3. Maintains schema information.
4. Tracks processing progress through checkpointing.
5. Writes data into a Bronze Delta table.
6. Processes available data and stops.

The job can run again on a schedule to ingest additional files.

**Important:** Auto Loader handles file discovery and ingestion. It does not automatically perform business-key deduplication or CDC merge logic.

## 9. Incremental Ingestion vs CDC

These concepts are related but not identical.

| Incremental Ingestion | Change Data Capture |
|---|---|
| Loads only new or changed data | Captures changes made in a source system |
| Can use timestamps, IDs, files, or CDC | Uses a change feed or source change mechanism |
| May not capture deletes | Can capture deletes when supported |
| Can be implemented without transaction-log CDC | Often uses database logs or change-tracking features |
| General ingestion strategy | Specific change-capture approach |

**Example:**

A timestamp-based pipeline may detect a customer update, but it may not detect a deleted customer if the source simply removes the row.

CDC can capture the delete operation when the source provides delete events.

## 10. Incremental Ingestion vs Full Load

Consider a source table with 10 million records and 10,000 daily changes.

| Factor | Full Load | Incremental Load |
|---|---|---|
| Records processed | 10 million | Approximately 10,000 changes |
| Processing time | Usually higher | Usually lower |
| Data movement | Larger | Smaller |
| Implementation | Often simpler | Requires progress tracking |
| Change detection | Not required for complete snapshot | Required |
| Historical correction | Included if present in source snapshot | Depends on change capture and backfill strategy |

**Interview point:** Incremental ingestion is not always the best option. Full loads can be suitable for small tables, complete snapshots, or situations where source change tracking is unreliable.

## 11. Handling Late-Arriving and Out-of-Order Data

One problem with timestamp-based incremental ingestion is that data may arrive late.

Example:

| customer_id | updated_at | Arrival Time |
|---|---|---|
| 101 | 10:00 AM | 10:01 AM |
| 102 | 10:05 AM | 10:06 AM |
| 103 | 10:03 AM | 10:15 AM |

Customer 103 has an earlier update timestamp but arrives later.

If the pipeline has already advanced its watermark beyond 10:03 AM, a strict `updated_at > watermark` condition may miss this record.

### Common Solutions

1. Use a lookback window.
2. Use source CDC when available.
3. Use reliable source sequence numbers.
4. Use overlapping extraction windows.
5. Deduplicate records in the destination.
6. Support periodic reconciliation and backfills.

### Example: Lookback Window

Instead of extracting only records after the exact last watermark, read a small period before it.

```python
from pyspark.sql import functions as F

lookback_start = F.expr(
    "timestamp'2026-10-30 09:00:00'"
)

incremental_df = source_df.filter(
    F.col("updated_at") >= lookback_start
)
```

The extracted records may overlap with previously processed data, so the destination should use idempotent writes such as MERGE.

## 12. Handling Deletes in Incremental Pipelines

A normal timestamp-based query cannot detect a row that has been physically deleted from the source table.

For example:

| customer_id | name |
|---|---|
| 101 | Rahul |
| 102 | Amit |
| 103 | Priya |

If customer 102 is deleted from the source, a query selecting existing rows cannot identify that deletion by itself.

### Common Solutions

- CDC delete events.
- Soft-delete flags such as `is_deleted`.
- Source change-tracking tables.
- Periodic source-to-target reconciliation.

If CDC provides a delete event, the destination can apply it using MERGE:

```sql
MERGE INTO target_customers AS target
USING customer_changes AS source
ON target.customer_id = source.customer_id

WHEN MATCHED AND source.operation = 'DELETE'
    THEN DELETE

WHEN MATCHED AND source.operation IN ('INSERT', 'UPDATE')
    THEN UPDATE SET *

WHEN NOT MATCHED AND source.operation = 'INSERT'
    THEN INSERT *;
```

This is an illustrative CDC pattern. The source must provide a reliable operation indicator, and the merge source should be deduplicated and ordered appropriately.

## 13. Idempotency in Incremental Ingestion

Incremental pipelines must be safe to retry.

Suppose a batch contains:

| order_id | amount |
|---|---:|
| 101 | 500 |
| 102 | 700 |

The target write succeeds, but the pipeline fails before updating its control table.

When the job runs again, the same batch may be extracted.

If the pipeline uses append mode, duplicate records may be created.

A safer approach is to use MERGE with a stable business key.

```sql
MERGE INTO target_orders AS target
USING staging_orders AS source
ON target.order_id = source.order_id

WHEN MATCHED THEN
    UPDATE SET *

WHEN NOT MATCHED THEN
    INSERT *;
```

This helps make repeated processing of the same deterministic batch safe.

**Remember:** Incremental ingestion and idempotency work together:

- Incremental ingestion identifies what needs to be processed.
- Idempotency ensures that processing it again does not create unintended results.

## 14. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| Loading the entire table every day unnecessarily | Use incremental ingestion when reliable change tracking is available |
| Using only increasing IDs for mutable records | IDs do not detect updates to existing records |
| Updating the watermark before writing target data | Advance progress only after the intended data commit succeeds |
| Ignoring late-arriving records | Use lookback windows, CDC, or reconciliation |
| Assuming timestamp-based ingestion detects deletes | Deletes need CDC, soft-delete flags, or reconciliation |
| Using append mode for every retry | Use idempotent writes to prevent duplicate records |
| Confusing batch watermark with streaming watermark | They solve different problems |
| Assuming Auto Loader deduplicates business records | It tracks file ingestion, not business-key uniqueness |
| Assuming incremental ingestion is always better | Full load may be simpler for small or complete-snapshot datasets |

## 15. Interview Questions and Answers

**Q1. What is incremental ingestion?**

Incremental ingestion loads only new or changed data instead of processing the complete source dataset every time.

**Q2. What is the difference between full load and incremental load?**

Full load processes the complete dataset, while incremental load processes only the changes since the previous successful load.

**Q3. How do you identify new or updated records?**

Common methods include timestamp columns, increasing IDs, CDC, and file-based discovery.

**Q4. What is a watermark in incremental batch ingestion?**

It is a saved progress value, such as a timestamp or ID, that identifies how far the pipeline has successfully processed.

**Q5. What is the difference between a batch watermark and a streaming watermark?**

A batch watermark tracks ingestion progress. A streaming watermark manages event-time state and late data in Structured Streaming.

**Q6. Can timestamp-based incremental ingestion detect deletes?**

Not when records are physically deleted without leaving a change record. CDC, soft-delete flags, or reconciliation are needed.

**Q7. Why do we need a control table?**

It stores pipeline metadata such as the last successful watermark, execution status, and batch information.

**Q8. How do you handle late-arriving records?**

Use lookback windows, CDC, overlapping extraction, deduplication, or reconciliation.

**Q9. How do you make an incremental pipeline idempotent?**

Use stable business keys, deduplicate the source, and apply MERGE or another retry-safe write strategy.

**Q10. What is CDC?**

Change Data Capture identifies source database changes such as inserts, updates, and deletes.

**Q11. How does Auto Loader support incremental ingestion?**

It incrementally discovers new files in cloud storage and tracks ingestion progress through Structured Streaming checkpointing.

**Q12. What happens if the target write succeeds but the watermark update fails?**

The same data may be extracted again. The pipeline should safely handle this retry using idempotent writes and a carefully designed control-table update process.

**Q13. When would you choose full load instead of incremental load?**

For small datasets, complete snapshots, initial ingestion, or when reliable change tracking is unavailable.

**Q14. How would you design an incremental customer ingestion pipeline?**

I would identify a reliable change-tracking column or CDC source, read the previous watermark, extract changed records, validate and deduplicate them, MERGE into the Delta target, verify the commit, and then advance the control-table watermark.

## 16. Quick Revision

| Concept | Key Point |
|---|---|
| Incremental ingestion | Loads only new or changed data |
| Full load | Loads the complete dataset |
| Timestamp-based load | Uses `updated_at` or similar columns |
| ID-based load | Uses increasing identifiers |
| CDC | Captures inserts, updates, and deletes |
| File-based ingestion | Processes newly discovered files |
| Batch watermark | Tracks ingestion progress |
| Control table | Stores pipeline execution metadata |
| Lookback window | Helps capture late-arriving changes |
| MERGE | Supports insert and update operations |
| Idempotency | Makes retries safe |
| Auto Loader | Incrementally discovers cloud files |

### One-Minute Interview Answer

Incremental ingestion means loading only new or changed records rather than processing the entire dataset repeatedly.

I can implement it using timestamps, increasing IDs, CDC, or file-based ingestion. In Databricks, Auto Loader is useful for incrementally ingesting cloud files, while Delta MERGE can apply inserts and updates to the target table.

I would maintain a control table to store the last successful watermark, handle late-arriving records using a lookback window or CDC, and make the target write idempotent so that failed batches can be retried safely.

## 17. Self-Assessment

- [ ] Explain incremental ingestion with an example.
- [ ] Differentiate full load and incremental load.
- [ ] Explain timestamp-based ingestion.
- [ ] Explain ID-based ingestion and its limitations.
- [ ] Explain CDC and its benefits.
- [ ] What is a watermark in batch ingestion?
- [ ] Differentiate batch and streaming watermarks.
- [ ] Explain the purpose of a control table.
- [ ] Handle late-arriving records.
- [ ] Explain how deletes are captured.
- [ ] Explain how Auto Loader supports incremental ingestion.
- [ ] Explain why MERGE is useful for retry-safe incremental pipelines.
- [ ] Design an end-to-end incremental ingestion pipeline.

**Interview readiness:** You should be able to explain incremental ingestion in 2 minutes, compare its common methods, and design a retry-safe Databricks pipeline using a watermark, control table, and Delta MERGE.