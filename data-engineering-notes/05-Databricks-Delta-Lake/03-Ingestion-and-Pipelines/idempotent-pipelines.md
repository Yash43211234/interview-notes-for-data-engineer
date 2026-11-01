# Idempotent Pipelines in Data Engineering

## 1. What Is Idempotency?

**Idempotency** means that executing the same operation multiple times produces the same final result as executing it once.

In data engineering, an idempotent pipeline can be safely rerun without creating unintended duplicate records or changing the final result incorrectly.

This is important because pipelines can fail due to:

- Cluster failures.
- Network interruptions.
- Job timeouts.
- Source system issues.
- Incorrect data.
- Streaming query restarts.
- Manual reruns.

**Simple example:**

Suppose a pipeline loads 1,000 customer records into a Delta table.

If the job fails after writing the data but before reporting success, an operator may rerun it.

- Non-idempotent pipeline: The same 1,000 records may be inserted again.
- Idempotent pipeline: The final table still contains the correct 1,000 records.

**Remember:** Idempotency does not mean the pipeline executes only once. It means repeated execution produces the intended final state.

## 2. Why Is Idempotency Important?

In production, data pipelines are not guaranteed to run successfully every time.

A job may complete its data write but fail before the orchestration system receives confirmation.

When the job is retried, the same input may be processed again.

Without idempotency, this can cause:

- Duplicate records.
- Incorrect aggregations.
- Inflated revenue calculations.
- Repeated updates.
- Inconsistent reporting.
- Incorrect downstream data.

### Example: Duplicate Sales

Suppose a sales pipeline processes this data:

| order_id | amount |
|---|---:|
| 101 | 500 |
| 102 | 700 |
| 103 | 300 |

Expected total sales:

\[
500 + 700 + 300 = 1500
\]

If the pipeline inserts the same records again:

| order_id | amount |
|---|---:|
| 101 | 500 |
| 102 | 700 |
| 103 | 300 |
| 101 | 500 |
| 102 | 700 |
| 103 | 300 |

Incorrect total:

\[
1500 + 1500 = 3000
\]

An idempotent design prevents this unintended duplication.

## 3. Idempotent vs Non-Idempotent Pipelines

| Feature | Idempotent Pipeline | Non-Idempotent Pipeline |
|---|---|---|
| Rerunning the job | Safe when inputs and logic are unchanged | May create duplicate or incorrect results |
| Duplicate handling | Uses keys, merge logic, overwrite, or other controls | May repeatedly insert records |
| Failure recovery | Easier to retry safely | Requires manual correction more often |
| Data consistency | Designed to preserve the intended final state | Can produce inconsistent results |
| Production reliability | More reliable under retries | More vulnerable to repeated execution |

## 4. Common Techniques to Build Idempotent Pipelines

There are several ways to make a data pipeline idempotent.

### A. Use MERGE Instead of Blind INSERT

One common approach is to use a unique business key and perform an upsert.

An upsert means:

- Update the record if it already exists.
- Insert the record if it does not exist.

Example source data:

| customer_id | name | city |
|---|---|---|
| 101 | Rahul | Kolkata |
| 102 | Amit | Delhi |

Target Delta table:

| customer_id | name | city |
|---|---|---|
| 101 | Rahul | Ranchi |
| 103 | Priya | Mumbai |

Use MERGE:

```sql
MERGE INTO target_customers AS target
USING source_customers AS source
ON target.customer_id = source.customer_id

WHEN MATCHED THEN
    UPDATE SET
        target.name = source.name,
        target.city = source.city

WHEN NOT MATCHED THEN
    INSERT (
        customer_id,
        name,
        city
    )
    VALUES (
        source.customer_id,
        source.name,
        source.city
    );
```

After execution:

| customer_id | name | city |
|---|---|---|
| 101 | Rahul | Kolkata |
| 102 | Amit | Delhi |
| 103 | Priya | Mumbai |

Running the same MERGE again with the same source produces the same final target state.

**Important:** The source should be deduplicated by the merge key. Multiple source rows matching the same target row can cause ambiguous or failed MERGE operations.

### B. Use a Unique Business Key

A business key uniquely identifies a record within the business domain.

Examples:

| Dataset | Possible Business Key |
|---|---|
| Customers | customer_id |
| Orders | order_id |
| Payments | transaction_id |
| Employees | employee_id |
| Products | product_id |

Using a business key helps identify whether a record already exists.

**Important:** A key is only useful if its uniqueness assumptions are valid. If the source contains duplicate keys, handle them before writing.

### C. Use Deduplication

Deduplication removes repeated records based on selected columns.

PySpark example:

```python
deduplicated_df = df.dropDuplicates(
    ["order_id"]
)
```

This removes duplicate rows based on `order_id`.

However, `dropDuplicates()` does not guarantee which record is retained when records with the same key contain different values.

For example:

| order_id | status | updated_at |
|---|---|---|
| 101 | Pending | 10:00 |
| 101 | Completed | 11:00 |

If the requirement is to keep the latest record, use a window function.

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

window_spec = (
    Window
    .partitionBy("order_id")
    .orderBy(
        F.col("updated_at").desc(),
        F.col("event_id").desc()
    )
)

latest_df = (
    df.withColumn(
        "row_num",
        F.row_number().over(window_spec)
    )
    .filter(F.col("row_num") == 1)
    .drop("row_num")
)
```

Here, `event_id` is an additional deterministic tie-breaker.

### D. Use Overwrite for Complete Rebuilds

If a pipeline recalculates an entire target dataset from a complete source snapshot, overwriting the target can be an idempotent strategy.

Example:

```python
df.write \
    .format("delta") \
    .mode("overwrite") \
    .save(target_path)
```

Running the same transformation again with the same complete input produces the same logical result.

**Important considerations:**

- Overwrite replaces existing target data according to the write configuration.
- It is not suitable for every incremental pipeline.
- Concurrent writers and partial input can cause problems.
- Consider table replacement, partition overwrite, or `replaceWhere` when appropriate.

Do not use full overwrite when the source only contains a small incremental batch.

### E. Use a Processed-Batch Tracking Table

For batch ingestion, a pipeline can maintain a tracking table containing processed batch identifiers.

Example:

| batch_id | status | processed_at |
|---|---|---|
| batch_001 | SUCCESS | 2026-10-30 10:00 |
| batch_002 | SUCCESS | 2026-10-30 11:00 |
| batch_003 | FAILED | NULL |

Before processing a batch:

1. Check whether the batch has already completed.
2. If it has completed, skip it.
3. Otherwise, process the batch.
4. Mark it successful only after the intended data write has committed.

**Important:** Tracking and writing should be designed to handle failures between these steps. If the data write succeeds but the tracking update fails, a retry must still be safe. Use transactional or idempotent write patterns where possible.

## 5. Idempotency in Databricks and Delta Lake

Delta Lake provides useful features for building reliable, retry-safe pipelines.

Important techniques include:

- `MERGE INTO` for upserts.
- Transactional Delta writes.
- `DELETE` and `UPDATE` for controlled modifications.
- `dropDuplicates()` and window functions for deduplication.
- Structured Streaming checkpoints for recovery.
- Transaction identifiers for supported idempotent write patterns.

### Example: Idempotent Delta MERGE

Suppose incoming order data is stored in a staging DataFrame.

```python
from delta.tables import DeltaTable

target = DeltaTable.forPath(
    spark,
    target_path
)

(
    target.alias("target")
    .merge(
        source_df.alias("source"),
        "target.order_id = source.order_id"
    )
    .whenMatchedUpdateAll()
    .whenNotMatchedInsertAll()
    .execute()
)
```

If the source has one deterministic row per `order_id`, rerunning the same MERGE with unchanged source data produces the same final target state.

### Important Delta Lake Concepts

| Feature | Role |
|---|---|
| MERGE | Inserts or updates records based on matching conditions |
| ACID transactions | Ensures atomic and consistent table commits |
| Checkpointing | Supports streaming recovery |
| Deduplication | Removes repeated records |
| Transaction identifiers | Help identify repeated writes in supported patterns |

**Remember:** ACID transactions and idempotency are related but different concepts. ACID protects transaction consistency; idempotency ensures repeated operations produce the intended result.

## 6. Idempotency in Structured Streaming

Structured Streaming uses checkpointing to track progress and recover after failures.

Example:

```python
query = (
    df.writeStream
    .format("delta")
    .outputMode("append")
    .option("checkpointLocation", checkpoint_path)
    .start(target_path)
)
```

When the query restarts with its existing checkpoint, Spark can recover its progress.

Delta Lake supports transactional streaming writes that help provide exactly-once results for supported integrations.

However, checkpointing alone does not make every custom operation or external side effect idempotent.

For example, if a streaming application calls an external payment API for each record, retrying a micro-batch could repeat the API call unless the API operation is designed to be idempotent.

### foreachBatch and Idempotency

`foreachBatch()` allows custom processing for each micro-batch.

Example:

```python
def process_batch(batch_df, batch_id):

    batch_df.createOrReplaceTempView(
        "incoming_orders"
    )

    batch_df.sparkSession.sql("""
        MERGE INTO bronze.orders AS target
        USING incoming_orders AS source
        ON target.order_id = source.order_id

        WHEN MATCHED THEN UPDATE SET *

        WHEN NOT MATCHED THEN INSERT *
    """)

query = (
    df.writeStream
    .foreachBatch(process_batch)
    .option("checkpointLocation", checkpoint_path)
    .start()
)
```

Here:

- `batch_df` contains the current micro-batch.
- `batch_id` identifies the micro-batch within the query.
- MERGE applies the batch to the target table.

Using a stable business key and deterministic source data makes this processing safer to retry.

**Important:** `foreachBatch()` provides at-least-once execution by default. The function may run again for the same batch after a failure. The custom write logic must therefore be idempotent or use an appropriate transaction/deduplication strategy.

## 7. Transaction Identifiers for Idempotent Delta Writes

Delta Lake supports transaction identifiers for certain write patterns.

Two useful options are:

- `txnAppId`: Identifies the application or writer.
- `txnVersion`: Identifies the transaction version for that application.

These can help Delta recognize repeated writes from the same application.

Example:

```python
(
    batch_df.write
    .format("delta")
    .mode("append")
    .option("txnAppId", "orders_ingestion")
    .option("txnVersion", batch_id)
    .save(target_path)
)
```

If the same application and transaction version are submitted again, Delta can recognize the duplicate transaction and avoid applying the same write again under the supported transaction-ID semantics.

**Important considerations:**

1. Use a stable application ID for the logical writer.
2. Use a deterministic transaction version.
3. Do not reuse the same transaction version for different data.
4. Manage transaction versions carefully across restarts.
5. If a new checkpoint creates a new batch-ID sequence, review the transaction-ID strategy before restarting.

This is a specialized technique. For many pipelines, a well-designed MERGE is easier to understand and maintain.

## 8. Idempotency vs Exactly-Once Processing

These concepts are closely related, but they are not identical.

### Idempotency

Repeated execution of the same operation produces the same final result.

### Exactly-Once Processing

The pipeline produces each intended committed result once from the perspective of the destination, within the guarantees of the source, processing engine, and sink.

| Idempotency | Exactly-Once Processing |
|---|---|
| Focuses on safe repeated execution | Focuses on final processing and commit guarantees |
| Repeated operations do not change the intended result | Prevents unintended duplicate or missing committed effects |
| Can be achieved using MERGE, overwrite, or deduplication | Requires coordination between source, processing, and sink |
| Useful for retries and reruns | Useful for reliable end-to-end processing |

**Example:**

A pipeline may physically execute the same batch twice after a failure, but an idempotent MERGE can ensure that the final Delta table contains only the intended records.

Thus, repeated computation does not necessarily mean duplicate committed results.

## 9. Practical End-to-End Example

Suppose an e-commerce company receives daily order files.

Requirements:

- New orders must be inserted.
- Existing orders must be updated.
- Re-running the same batch must not duplicate orders.
- The latest order status must be retained.

### Step 1: Read Source Data

```python
source_df = spark.read.format("json").load(
    source_path
)
```

### Step 2: Deduplicate Source Records

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

window_spec = (
    Window
    .partitionBy("order_id")
    .orderBy(
        F.col("updated_at").desc(),
        F.col("event_id").desc()
    )
)

deduplicated_df = (
    source_df
    .withColumn(
        "row_num",
        F.row_number().over(window_spec)
    )
    .filter(F.col("row_num") == 1)
    .drop("row_num")
)
```

### Step 3: Merge Into Delta Table

```python
from delta.tables import DeltaTable

target = DeltaTable.forPath(
    spark,
    target_path
)

(
    target.alias("target")
    .merge(
        deduplicated_df.alias("source"),
        "target.order_id = source.order_id"
    )
    .whenMatchedUpdateAll()
    .whenNotMatchedInsertAll()
    .execute()
)
```

### Step 4: Verify the Result

```python
result_df = spark.read.format("delta").load(
    target_path
)

result_df.groupBy("order_id") \
    .count() \
    .filter("count > 1") \
    .show()
```

If the target is expected to contain one row per order, this check should return no rows.

### Result

- Duplicate source records are reduced to one deterministic record per order.
- Existing orders are updated.
- New orders are inserted.
- Rerunning the same batch produces the same intended final state.

**Production note:** This example assumes `order_id` is the correct business key and that the source contains a valid latest-record ordering. Real pipelines should also validate null keys, malformed records, and source consistency.

## 10. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| Using append mode for every retry | Repeated appends can create duplicate records |
| Assuming checkpointing alone makes custom writes idempotent | Custom processing and external side effects need their own safeguards |
| Using MERGE without deduplicating source keys | Multiple source matches can make the operation ambiguous or fail |
| Assuming `dropDuplicates()` always keeps the latest record | It does not guarantee which duplicate is retained |
| Using overwrite for an incremental source batch | It may remove valid historical data |
| Assuming ACID automatically means idempotent | ACID and idempotency solve different problems |
| Reusing transaction IDs for different data | Can cause legitimate writes to be skipped |
| Assuming a successful write means the entire job completed successfully | The orchestration system may fail after the data commit |
| Ignoring null business keys | Records may not match correctly during MERGE |

## 11. Interview Questions and Answers

**Q1. What is idempotency in data engineering?**

Idempotency means executing the same operation multiple times produces the same intended final result as executing it once.

**Q2. Why is idempotency important in ETL pipelines?**

It allows pipelines to be safely retried after failures without creating duplicate records or incorrect results.

**Q3. How can you make a Delta Lake pipeline idempotent?**

Use business keys, deduplicate incoming records, and apply MERGE operations instead of blindly appending the same data.

**Q4. What is the difference between append and MERGE?**

Append adds incoming records without matching them against existing records. MERGE can insert new records and update existing records based on a matching condition.

**Q5. Does checkpointing guarantee idempotency?**

No. Checkpointing stores streaming progress and supports recovery. Custom processing and external side effects may still need idempotent logic.

**Q6. Is MERGE always idempotent?**

Not automatically. It is generally safe to rerun with the same deterministic source and matching logic, but source duplicates, nondeterministic expressions, and changing inputs can affect the result.

**Q7. What is the difference between idempotency and exactly-once processing?**

Idempotency makes repeated execution safe. Exactly-once processing concerns the end-to-end guarantees that intended results are committed without unintended duplicates or omissions.

**Q8. How do you handle duplicate records in PySpark?**

Use `dropDuplicates()` for simple duplicate removal. Use window functions with a deterministic ordering when a specific record, such as the latest one, must be retained.

**Q9. What is `foreachBatch()`?**

It allows custom batch-level processing of each micro-batch in a Structured Streaming query. Its function may be retried, so the custom logic should be idempotent.

**Q10. What are `txnAppId` and `txnVersion`?**

They are transaction identifiers used by supported Delta write patterns to recognize duplicate transaction submissions.

**Q11. Give a real-world example of an idempotent pipeline.**

An order ingestion pipeline uses `order_id` as a business key and MERGE to update existing orders or insert new ones. Rerunning the same batch does not create duplicate orders.

**Q12. What would you check if rerunning a pipeline creates duplicates?**

I would check the write mode, business key, source duplicates, MERGE conditions, checkpoint configuration, and whether the pipeline has retry-safe logic.

## 12. Quick Revision

| Concept | Key Point |
|---|---|
| Idempotency | Repeated execution produces the same intended final result |
| Main purpose | Safe retries and reruns |
| Business key | Identifies a record |
| MERGE | Insert or update based on a matching condition |
| Deduplication | Removes repeated records |
| Overwrite | Useful for complete deterministic rebuilds |
| Checkpointing | Supports streaming recovery |
| `foreachBatch()` | Custom processing for each micro-batch |
| `txnAppId` | Identifies the logical writer |
| `txnVersion` | Identifies a transaction version |
| Exactly-once | End-to-end committed-result guarantee |
| ACID | Transaction consistency and reliability |

### One-Minute Interview Answer

Idempotency means that running the same data pipeline multiple times produces the same intended final result as running it once.

It is important because production jobs can fail after partially or fully committing data, and orchestration systems may retry them.

In Databricks, I can build idempotent pipelines using business keys, source deduplication, Delta Lake MERGE operations, and appropriate transaction identifiers. For Structured Streaming, checkpointing supports recovery, but custom operations such as `foreachBatch()` must also be designed to handle retries safely.

## 13. Self-Assessment

- [ ] Define idempotency in simple terms.
- [ ] Explain why production pipelines need idempotency.
- [ ] Differentiate idempotent and non-idempotent pipelines.
- [ ] Explain how MERGE prevents duplicate inserts.
- [ ] Explain the importance of business keys.
- [ ] Differentiate `dropDuplicates()` and window-based deduplication.
- [ ] Explain when overwrite is appropriate.
- [ ] Explain checkpointing versus idempotency.
- [ ] Explain `foreachBatch()` retry behavior.
- [ ] Explain `txnAppId` and `txnVersion`.
- [ ] Differentiate idempotency, exactly-once processing, and ACID transactions.
- [ ] Design a retry-safe order ingestion pipeline.

**Interview readiness:** You should be able to explain idempotency in 2 minutes, write a Delta MERGE statement, and describe how you would prevent duplicates when a production pipeline is retried.