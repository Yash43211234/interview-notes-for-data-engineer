

# Pipeline Recovery and Reprocessing

## 1. What Is Pipeline Recovery?

Pipeline recovery is the process of restoring a data pipeline after a failure so that processing can continue without losing data or creating incorrect results.

A data pipeline can fail because of:

- Source system unavailability.
- Network interruptions.
- Compute resource failures.
- Invalid or corrupted input data.
- Schema changes.
- Storage failures.
- Transformation errors.
- Database or destination unavailability.
- Incorrect configuration or deployment.

The goal of recovery is to bring the pipeline back to a correct and consistent state.

### Example

Consider an e-commerce pipeline:

```text
Orders Database
      |
      v
Data Ingestion
      |
      v
ADLS Gen2 (Raw)
      |
      v
Databricks / PySpark
      |
      v
Delta Tables (Silver)
      |
      v
Data Warehouse (Gold)
```

Suppose the Silver transformation fails after processing 70% of the input data.

A recovery mechanism should ensure that:

1. Successfully committed data is not unnecessarily duplicated.
2. Unprocessed data is processed again.
3. Invalid records are identified and handled.
4. The pipeline eventually reaches a correct state.

---

## 2. Recovery vs Reprocessing vs Retry vs Restart

These terms are related but have different meanings.

| Term | Meaning |
|---|---|
| Retry | Execute a failed operation again. |
| Restart | Start a failed job or pipeline again. |
| Recovery | Restore the pipeline to a correct operational state. |
| Reprocessing | Process previously received or processed data again. |
| Replay | Read and process previously retained source events again. |
| Backfill | Process historical data for a missing or requested time period. |

### Example

A daily pipeline processes data for October 20.

- **Retry:** Try the failed database write again.
- **Restart:** Start the October 20 pipeline run again.
- **Recovery:** Resume from a safe point and restore the correct output.
- **Reprocessing:** Process October 20 data again.
- **Replay:** Read retained Kafka events again.
- **Backfill:** Process data for October 1–10 because those dates were missing.

**Interview point:** Restarting a pipeline does not automatically guarantee correct recovery. The pipeline must be designed to handle partially completed work.

---

## 3. Common Pipeline Failure Scenarios

| Failure | Possible impact | Recovery approach |
|---|---|---|
| Source unavailable | No new input | Retry with backoff |
| Network failure | Interrupted transfer | Retry or resume |
| Compute failure | Processing stops | Restart from a safe checkpoint |
| Invalid records | Transformation errors | Quarantine bad records |
| Schema change | Parsing or transformation failure | Validate and handle schema evolution |
| Target unavailable | Output cannot be committed | Retry an idempotent write |
| Duplicate input | Duplicate output | Deduplication and idempotent processing |
| Incorrect business logic | Wrong output | Correct code and reprocess affected data |
| Partial write | Inconsistent output | Atomic commit or safe overwrite |
| Lost checkpoint | Progress information unavailable | Controlled replay or reconstruction |

Not every failure should be handled in the same way. Recovery depends on where the failure occurred and what data was already committed.

---

## 4. Checkpoints and Recovery Points

A checkpoint stores progress information so that a pipeline can recover without unnecessarily starting from the beginning.

The exact checkpoint contents depend on the processing framework.

### 4.1 What Can a Checkpoint Store?

Examples include:

- Processed source offsets.
- Batch or micro-batch progress.
- Streaming state.
- Query metadata.
- Information required for recovery.

### 4.2 Checkpoint in Spark Structured Streaming

Structured Streaming uses a checkpoint location to store query progress and state information.

```python
query = (
    source_df.writeStream
    .format("delta")
    .outputMode("append")
    .option(
        "checkpointLocation",
        "abfss://checkpoints@storage.dfs.core.windows.net/orders/"
    )
    .toTable("silver.orders")
)
```

If the query fails, restarting it with the same compatible checkpoint location allows Spark to recover its progress and state.

Important points:

- Use a separate checkpoint location for each independent streaming query.
- Do not casually delete checkpoint data.
- Keep the checkpoint location durable and accessible.
- Changing query logic or stateful operations may require a carefully planned checkpoint migration or a new checkpoint.
- A checkpoint does not automatically make an arbitrary external sink exactly-once.

### 4.3 Batch Pipeline Checkpoints

Batch pipelines can maintain their own progress information.

For example, a pipeline may store the last successfully processed date:

```text
Pipeline: orders_daily
Last successful date: 2026-10-20
Status: SUCCESS
```

The next run can use this information to determine which date needs processing.

However, progress should only be advanced after the corresponding target data has been successfully committed.

---

## 5. Idempotency: The Foundation of Safe Recovery

Idempotency means that executing the same operation multiple times produces the same final effect as executing it once, under the defined conditions.

It is especially important because pipeline failures often lead to retries and reprocessing.

### Example: Non-Idempotent Write

Suppose a pipeline appends the same order records every time it runs.

```python
orders_df.write \
    .format("delta") \
    .mode("append") \
    .save("/mnt/delta/orders")
```

If the job fails after writing data but before recording successful completion, a retry may append the same records again.

This creates duplicates.

### Example: Idempotent Processing with MERGE

Suppose `order_id` is the business key.

```sql
MERGE INTO silver.orders AS target
USING staging.orders AS source
ON target.order_id = source.order_id

WHEN MATCHED THEN
    UPDATE SET
        target.customer_id = source.customer_id,
        target.amount = source.amount,
        target.order_date = source.order_date

WHEN NOT MATCHED THEN
    INSERT (
        order_id,
        customer_id,
        amount,
        order_date
    )
    VALUES (
        source.order_id,
        source.customer_id,
        source.amount,
        source.order_date
    );
```

This avoids inserting another target row for an already existing business key.

However, MERGE alone does not guarantee idempotency in every situation.

You must also consider:

- Whether the source contains duplicate keys.
- Whether the merge condition is correct.
- Whether updates are deterministic.
- Whether concurrent writes can conflict.
- Whether the source represents the same logical batch.

### Important: Deduplicate the Source

A MERGE operation can fail or behave ambiguously when multiple source rows match the same target row.

Example:

```python
from pyspark.sql import Window
from pyspark.sql.functions import (
    col,
    row_number,
    desc
)

window_spec = (
    Window
    .partitionBy("order_id")
    .orderBy(desc("updated_at"))
)

deduplicated_df = (
    source_df
    .withColumn(
        "row_num",
        row_number().over(window_spec)
    )
    .filter(col("row_num") == 1)
    .drop("row_num")
)
```

For deterministic results, include a reliable tie-breaker when multiple records have the same `updated_at`.

### Common Idempotency Techniques

1. Use business keys to identify records.
2. Use MERGE or upsert operations.
3. Overwrite a well-defined partition when appropriate.
4. Track source batch identifiers.
5. Deduplicate source records.
6. Make progress updates consistent with successful target commits.

---

## 6. Partial Failure and Atomicity

A pipeline can fail after completing some steps but before completing the entire workflow.

Consider:

```text
Step 1: Read source data
Step 2: Write raw data
Step 3: Transform raw data
Step 4: Write silver data
Step 5: Update pipeline watermark
```

Suppose Step 4 succeeds but Step 5 fails.

On retry, the pipeline may process the same source data again.

If the Silver write is not idempotent, duplicate records may appear.

### Recommended Approach

Design the workflow around safe commit boundaries.

```text
Read Source
    |
    v
Validate Input
    |
    v
Transform Data
    |
    v
Commit Target Data
    |
    v
Record Successful Progress
```

The progress marker should not advance before the target write succeeds.

However, a target write and a separate progress-table update are usually not one atomic transaction.

Therefore, the design must handle the failure between these two operations.

For example:

- If the target commit succeeds but progress update fails, retry the same logical batch safely.
- If the target commit fails, do not mark the batch as successful.
- Use batch identifiers and idempotent target writes to make retries safe.

**Important:** Atomicity of an individual Delta table transaction does not automatically make multiple tables and external systems one atomic transaction.

---

## 7. Retry Strategy and Exponential Backoff

Retries help recover from temporary failures.

Examples:

- Temporary network interruption.
- Service throttling.
- Database connection timeout.
- Temporary storage unavailability.

However, retrying every error is not useful.

### 7.1 Types of Errors

| Error type | Example | Recommended action |
|---|---|---|
| Transient | Temporary network error | Retry |
| Throttling | HTTP 429 | Retry with backoff |
| Authentication | Expired or invalid credentials | Fix authentication |
| Data quality | Invalid amount | Quarantine or reject |
| Schema | Unexpected column type | Investigate schema |
| Logic | Incorrect transformation | Fix code |
| Configuration | Wrong storage path | Fix configuration |

### 7.2 Exponential Backoff

Exponential backoff increases the waiting time between retry attempts.

A basic formula is:

\[
Delay_n = \min(MaxDelay,\ BaseDelay \times 2^n)
\]

For example, with a base delay of 2 seconds:

| Retry | Delay |
|---|---:|
| 1 | 2 seconds |
| 2 | 4 seconds |
| 3 | 8 seconds |
| 4 | 16 seconds |

In production systems, random jitter is often added to avoid many clients retrying simultaneously.

### 7.3 Python Retry Example

```python
import time
import random

def execute_with_retry(
    operation,
    max_retries=4,
    base_delay=2,
    max_delay=30
):
    for attempt in range(max_retries + 1):
        try:
            return operation()

        except TemporaryServiceError:
            if attempt == max_retries:
                raise

            delay = min(
                max_delay,
                base_delay * (2 ** attempt)
            )

            delay += random.uniform(0, 1)

            time.sleep(delay)
```

`TemporaryServiceError` is a placeholder for an application-specific exception. In production, catch only known retryable exceptions.

### Important Rules

- Set a maximum retry count.
- Use timeouts.
- Retry only transient errors.
- Use backoff and jitter.
- Make the operation safe to retry.
- Log each retry attempt.
- Escalate after retries are exhausted.

Retries should not hide permanent failures.

---

## 8. Reprocessing Historical Data

Reprocessing means processing data again when the original result is missing, incorrect, or needs to be recalculated.

Common reasons:

1. Transformation logic was incorrect.
2. A pipeline failed for a historical date.
3. Late-arriving records were received.
4. A source system corrected historical data.
5. Business rules changed.
6. A target table was accidentally deleted or corrupted.
7. A new column or attribute must be populated historically.

### 8.1 Full Reprocessing

Process the entire dataset again.

```text
All Historical Data
        |
        v
Reprocessing Pipeline
        |
        v
Rebuilt Target
```

Advantages:

- Simple conceptual approach.
- Can rebuild a target from trusted source data.
- Useful when many historical records are affected.

Disadvantages:

- Expensive for large datasets.
- Takes longer.
- Requires sufficient compute capacity.
- Can affect downstream systems.

### 8.2 Incremental Reprocessing

Process only the affected data.

Example: Reprocess records for October 15–20.

```python
from pyspark.sql.functions import col

historical_df = (
    spark.read
    .format("delta")
    .load("/mnt/delta/raw/orders")
    .filter(
        (col("order_date") >= "2026-10-15") &
        (col("order_date") < "2026-10-21")
    )
)
```

Then overwrite only the affected target partition when the table layout and write operation support that approach.

For example:

```python
(
    corrected_df.write
    .format("delta")
    .mode("overwrite")
    .option(
        "replaceWhere",
        "order_date >= '2026-10-15' AND order_date < '2026-10-21'"
    )
    .save("/mnt/delta/silver/orders")
)
```

The replacement condition should match the intended data scope. Validate the affected dates and write behavior before executing a historical overwrite.

### Full vs Incremental Reprocessing

| Aspect | Full | Incremental |
|---|---|---|
| Data volume | Entire dataset | Selected data |
| Compute cost | Usually higher | Usually lower |
| Execution time | Usually longer | Usually shorter |
| Complexity | Often simpler | Requires accurate scope |
| Risk | Broad impact | Risk of missing affected records |
| Best use | Large-scale rebuild | Limited historical correction |

---

## 9. Backfill and Late-Arriving Data

### 9.1 What Is Backfill?

Backfill is the process of loading or processing data for a historical period that was missed or needs to be populated.

Example:

A daily pipeline did not run from October 10 to October 12.

A backfill processes those dates.

```text
October 9  -> SUCCESS
October 10 -> MISSING
October 11 -> MISSING
October 12 -> MISSING
October 13 -> SUCCESS
```

The pipeline can be executed for the missing dates.

### 9.2 What Is Late-Arriving Data?

Late-arriving data is data that reaches the pipeline after its expected processing window.

Example:

An order was created on October 10 but reached the data platform on October 12.

If the pipeline only reads new records using a strict timestamp boundary, this record might be missed.

### Handling Late-Arriving Data

Common approaches:

- Use a lookback window.
- Use CDC where available.
- Reprocess affected partitions.
- Maintain source extraction metadata.
- Run periodic reconciliation.

Example of a lookback window:

```text
Current processing date: October 12
Lookback period: 3 days

Read:
October 9
October 10
October 11
October 12
```

The target must support safe repeated writes, such as MERGE or controlled partition replacement.

A lookback window reduces the chance of missing late updates, but it cannot guarantee completeness if records can arrive later than the configured window.

---

## 10. Pipeline Watermarks and Progress Tracking

A watermark in batch ingestion often represents the last successfully processed source position, such as:

- Last processed timestamp.
- Last processed ID.
- Last successful date.
- Last CDC sequence number.

It helps identify which source data should be read in the next run.

### Example

Suppose the last successful watermark is:

```text
2026-10-20 23:59:59
```

The next run reads records after that timestamp.

```sql
SELECT *
FROM source_orders
WHERE updated_at > '2026-10-20 23:59:59';
```

### Safe Watermark Update Process

```text
Read Current Watermark
        |
        v
Extract New Data
        |
        v
Transform and Validate
        |
        v
Write Target Successfully
        |
        v
Update Watermark
```

### Important Failure Scenario

Suppose the target write succeeds but the watermark update fails.

The next run may read the same source records again.

This is why the target write should be idempotent.

Another important issue is that timestamp-based extraction can miss updates when timestamps are unreliable, records arrive late, or multiple records share a timestamp. Use a reliable composite cursor, CDC sequence, or overlapping lookback strategy where necessary.

---

## 11. Dead-Letter Queues and Quarantine

Not every record can be processed successfully.

A pipeline should separate invalid records from valid records instead of silently losing them.

### Dead-Letter Queue (DLQ)

A dead-letter queue stores messages or records that could not be processed successfully.

### Quarantine Storage

Quarantine storage is a location where invalid records are retained for investigation and correction.

Example:

```text
Incoming Data
      |
      v
Validation
      |
      +------------------+
      |                  |
      v                  v
Valid Records       Invalid Records
      |                  |
      v                  v
Silver Table        Quarantine
```

### Example in PySpark

```python
from pyspark.sql.functions import col

valid_df = source_df.filter(
    col("order_id").isNotNull() &
    col("amount").isNotNull() &
    (col("amount") >= 0)
)

invalid_df = source_df.filter(
    col("order_id").isNull() |
    col("amount").isNull() |
    (col("amount") < 0)
)
```

Write invalid records to a quarantine location with useful metadata:

- Source file or topic.
- Ingestion timestamp.
- Error reason.
- Pipeline run ID.
- Original record.
- Schema or validation version, where useful.

### DLQ vs Quarantine

| DLQ | Quarantine |
|---|---|
| Common in messaging systems | Common in data pipelines |
| Holds messages that failed processing | Holds invalid or rejected data |
| Often supports later replay | Often supports investigation and correction |
| May preserve message metadata | Should preserve source and validation context |

The terms can overlap in practice.

---

## 12. Replay and Reprocessing in Streaming Pipelines

Streaming systems need special recovery mechanisms because they process continuous data.

### Kafka Example

Kafka retains messages according to its retention configuration.

A consumer can read messages again by resetting its offsets, provided the required messages are still retained.

```text
Kafka Topic
    |
    v
Consumer
    |
    v
Stream Processing
    |
    v
Target Table
```

If a processing bug is discovered, historical events may be replayed into a corrected processing pipeline.

Important considerations:

- Required messages must still exist in Kafka.
- Offset management must be controlled.
- Replayed output must not create duplicates.
- Event ordering may matter.
- Stateful processing requires appropriate recovery design.
- Reprocessing can affect downstream consumers.

### Spark Structured Streaming

Spark stores source progress and state information in its checkpoint location.

When a query restarts with its compatible checkpoint, Spark uses that information to recover processing.

Do not delete the checkpoint simply to force replay. A fresh checkpoint creates a new query history and can cause previously processed data to be read again.

### Exactly-Once Consideration

Exactly-once processing is not guaranteed merely because a system uses checkpoints.

End-to-end correctness depends on:

- Source replay behavior.
- Progress tracking.
- Processing logic.
- State recovery.
- Sink commit behavior.
- Idempotency or transactional integration.

The practical goal is often exactly-once effective results, even when an operation may be attempted more than once.

---

## 13. Delta Lake Recovery Features

Delta Lake provides features that help recover and correct data.

### 13.1 Time Travel

Time Travel allows users to query an earlier committed version of a Delta table.

```sql
SELECT *
FROM silver.orders VERSION AS OF 10;
```

Or:

```sql
SELECT *
FROM silver.orders
TIMESTAMP AS OF '2026-10-20 10:00:00';
```

Useful when:

- A table was accidentally overwritten.
- A previous version needs investigation.
- A comparison between table versions is required.

Time Travel depends on the availability of transaction log information and underlying data files.

### 13.2 RESTORE

RESTORE creates a new table version that reflects the data and metadata of an earlier version.

```sql
RESTORE TABLE silver.orders
TO VERSION AS OF 10;
```

Important:

- RESTORE does not erase intervening history.
- It creates a new committed version.
- It does not replace a proper backup and disaster-recovery strategy.
- Retention and file availability matter.

### 13.3 VACUUM

VACUUM removes eligible unreferenced data files.

```sql
VACUUM silver.orders DRY RUN;
```

Then, when appropriate:

```sql
VACUUM silver.orders;
```

Important:

- VACUUM is a cleanup operation, not a recovery operation.
- It can make older table versions unavailable if their data files are deleted.
- The commonly used default deleted-file retention is seven days, but configuration and platform behavior can vary.
- Do not casually disable retention safety checks.

### Recovery Feature Summary

| Feature | Purpose |
|---|---|
| Time Travel | Read an earlier table version |
| RESTORE | Create a new version reflecting an earlier version |
| VACUUM | Remove eligible unreferenced data files |
| Transaction Log | Track committed table changes |
| Checkpoint files | Summarize transaction log state |

---

## 14. Orchestration-Level Recovery

An orchestrator manages pipeline execution and dependencies.

Examples:

- Azure Data Factory.
- Databricks Workflows.
- Apache Airflow.
- Azure Synapse Pipelines.

Orchestration-level recovery may include:

- Retry policies.
- Timeout configuration.
- Failure handling.
- Dependency management.
- Alerts.
- Pipeline restart.
- Parameterized backfills.
- Rerunning failed activities.

### Example: Azure Data Factory

Consider:

```text
Copy Activity
      |
      v
Databricks Transformation
      |
      v
Load Gold Table
      |
      v
Send Completion Notification
```

If the Databricks activity fails:

1. Check the failure details.
2. Determine whether the error is transient or permanent.
3. Retry if the failure is temporary and the operation is safe.
4. Correct the root cause if necessary.
5. Rerun from an appropriate activity or pipeline boundary.
6. Verify the target data before marking the run successful.

**Important:** Rerunning an activity is not automatically safe if the activity partially wrote data. Its target operation must support safe retry or have a recovery procedure.

---

## 15. Monitoring and Recovery Observability

A pipeline should record enough information to identify where and why a failure occurred.

### Useful Monitoring Fields

| Field | Purpose |
|---|---|
| Pipeline name | Identify the pipeline |
| Run ID | Identify a particular execution |
| Batch ID | Identify a logical processing batch |
| Start time | Track execution start |
| End time | Track execution completion |
| Status | SUCCESS, FAILED, RUNNING |
| Source count | Number of input records |
| Valid count | Number of accepted records |
| Rejected count | Number of rejected records |
| Target count | Number of written records |
| Watermark | Track source progress |
| Error message | Understand failure |
| Retry count | Track repeated attempts |

### Important Metrics

- Pipeline success rate.
- Failure rate.
- Retry count.
- Processing duration.
- Data freshness.
- Input and output record counts.
- Rejected record count.
- Duplicate record count.
- Backlog size.
- Recovery time.

### Data Reconciliation

Reconciliation compares source and target data to identify missing, duplicated, or inconsistent records.

Example:

```sql
SELECT COUNT(*) AS source_count
FROM source_orders
WHERE order_date = '2026-10-20';
```

```sql
SELECT COUNT(*) AS target_count
FROM silver.orders
WHERE order_date = '2026-10-20';
```

Matching counts alone do not prove correctness. Also compare business keys, amounts, aggregates, and relevant checksums where appropriate.

---

## 16. Designing a Reliable Recovery Strategy

A production pipeline should have a defined recovery plan.

### Recommended Design

```text
                Source System
                     |
                     v
                Data Ingestion
                     |
                     v
                  Raw Zone
                     |
                     v
             Schema and Quality Checks
                     |
              +------+------+
              |             |
              v             v
          Valid Data    Invalid Data
              |             |
              v             v
        Transformation  Quarantine
              |
              v
        Idempotent Target Write
              |
              v
       Successful Commit Check
              |
              v
        Update Progress State
              |
              v
          Monitoring
```

### Recovery Design Principles

1. Preserve raw source data whenever practical.
2. Use stable batch IDs or source offsets.
3. Make target writes safe to retry.
4. Separate transient failures from permanent failures.
5. Maintain reliable checkpoints and watermarks.
6. Quarantine invalid records.
7. Support historical reprocessing and backfills.
8. Record pipeline execution metadata.
9. Validate data after recovery.
10. Define who can trigger reprocessing and how its impact is controlled.

---

## 17. Practical Scenario: Recovering a Failed Daily Orders Pipeline

### Scenario

An e-commerce company processes daily orders.

The pipeline runs every day at midnight.

On October 20:

- 100,000 records were extracted.
- 100,000 records were written to Raw.
- The Silver transformation failed.
- The pipeline status became FAILED.

### Step-by-Step Recovery

**Step 1: Identify the failure**

Check the orchestrator run history, Spark logs, and error message.

Determine whether the problem is caused by invalid data, code, compute, or infrastructure.

**Step 2: Confirm source availability**

Verify that the original October 20 data is still available in Raw storage.

**Step 3: Check target state**

Determine whether any Silver records were committed before the failure.

Do not assume that a failed job wrote nothing.

**Step 4: Correct the issue**

If the failure was caused by a transformation bug, fix and test the code.

If invalid records caused the issue, route them to quarantine according to business rules.

**Step 5: Reprocess the affected batch**

Read the October 20 raw data again.

Use a stable batch identifier and a safe target write strategy, such as a properly scoped partition replacement or deduplicated MERGE.

**Step 6: Validate the result**

Check:

- Expected source and target counts.
- Duplicate business keys.
- Rejected records.
- Important business aggregates.
- Data completeness.

**Step 7: Update pipeline progress**

Mark October 20 as successfully processed only after the target commit and required validations succeed.

**Step 8: Resume normal processing**

Allow the next scheduled run to continue without skipping or duplicating data.

---

## 18. Common Mistakes

1. Restarting a pipeline without checking partial writes.
2. Assuming retries automatically prevent duplicate records.
3. Deleting streaming checkpoints without understanding the impact.
4. Updating a watermark before the target write succeeds.
5. Reprocessing all historical data when only a few partitions are affected.
6. Reprocessing only a small date range when the affected scope is larger.
7. Retrying permanent data-quality or authentication failures.
8. Using MERGE without deduplicating source keys.
9. Assuming matching record counts prove data correctness.
10. Treating Time Travel as a permanent backup.
11. Running VACUUM without considering retention and recovery requirements.
12. Assuming a pipeline failure means no output was written.
13. Replaying streaming events without considering sink idempotency.
14. Failing to record the reason for a recovery or reprocessing run.

---

## 19. Interview Questions and Answers

### Q1. What is pipeline recovery?

Pipeline recovery is the process of restoring a failed data pipeline to a correct state while preventing data loss, duplication, and inconsistency.

### Q2. What is the difference between retry and reprocessing?

Retry executes a failed operation again, usually to recover from a temporary error. Reprocessing processes previously received or processed data again, often to correct missing or incorrect results.

### Q3. Why is idempotency important in data pipelines?

Because a pipeline may execute the same batch multiple times after failures. Idempotency ensures repeated execution does not create unintended duplicate or inconsistent results.

### Q4. What is a checkpoint?

A checkpoint stores processing progress and, for stateful streaming queries, state information needed for recovery.

### Q5. Does a checkpoint guarantee exactly-once processing?

No. Exactly-once effective results depend on source progress tracking, processing behavior, and sink commit semantics. An arbitrary external sink may still receive duplicate writes.

### Q6. What is a watermark in batch ingestion?

It is a progress marker, such as the last successfully processed timestamp or ID, used to identify which source records should be extracted next.

### Q7. When should a watermark be updated?

After the corresponding target data has been successfully committed. The design must also handle a failure between the target commit and the watermark update.

### Q8. What is backfilling?

Backfilling is processing historical data for a missing or requested period.

### Q9. What is late-arriving data?

Data that reaches the pipeline after its expected processing window. It can be handled using lookback windows, CDC, reconciliation, or targeted reprocessing.

### Q10. What is a dead-letter queue?

A dead-letter queue stores messages that could not be processed successfully so they can be investigated and potentially replayed later.

### Q11. How do you recover from a partial write?

First inspect the target state. Then use a safe recovery strategy, such as an idempotent MERGE or controlled partition replacement, and validate the result before marking the batch successful.

### Q12. What is the difference between full and incremental reprocessing?

Full reprocessing rebuilds the output from all relevant historical data. Incremental reprocessing processes only the affected records or partitions.

### Q13. How does Delta Lake support recovery?

Delta Lake provides transaction logs, table versioning, Time Travel, and RESTORE. These features help inspect or restore earlier committed table states, subject to retention and file availability.

### Q14. What is exponential backoff?

It is a retry strategy that increases the waiting time between attempts, reducing pressure on temporarily unavailable services.

### Q15. How do you prevent duplicates during reprocessing?

Use stable business keys, deduplicate the source, and apply idempotent writes such as MERGE or controlled partition replacement. Validate the final target state.

### Q16. What is the difference between replay and backfill?

Replay reads previously retained events or messages again. Backfill processes data for a historical time period that was missing or needs recalculation. They can overlap but are not identical.

### Q17. How would you recover a failed Databricks pipeline?

I would inspect the job run and Spark logs, identify the failure stage, check whether any target data was committed, fix the root cause, rerun from a safe boundary, and validate the output. I would also verify checkpoint and watermark handling where applicable.

### Q18. What is the biggest challenge in pipeline recovery?

Maintaining data correctness after partial failures, especially when a target write succeeds but the pipeline fails before recording successful progress.

---

## 20. Quick Revision

| Concept | Remember |
|---|---|
| Recovery | Restore correct pipeline operation |
| Retry | Execute a failed operation again |
| Restart | Start a failed job again |
| Reprocessing | Process existing data again |
| Replay | Read retained events again |
| Backfill | Process missing historical periods |
| Checkpoint | Preserve processing progress and state |
| Watermark | Track source extraction progress |
| Idempotency | Safe repeated execution |
| Exponential backoff | Increase retry delay |
| DLQ | Store failed messages |
| Quarantine | Store invalid records |
| Time Travel | Read an earlier Delta version |
| RESTORE | Create a new version reflecting an earlier state |
| VACUUM | Remove eligible unreferenced files |

### Five Rules to Remember

1. Never assume a failed pipeline wrote nothing.
2. Make target operations safe to retry.
3. Advance progress only after successful target commits.
4. Preserve checkpoints and raw data for recovery.
5. Validate data after every recovery or reprocessing operation.

---

## 21. Self-Assessment

Try answering these without looking at the notes.

1. Why is restarting a failed pipeline not always sufficient?
2. How can you prevent duplicates when reprocessing a batch?
3. What happens if a target write succeeds but the watermark update fails?
4. What is the difference between a checkpoint and a watermark?
5. When would you choose full reprocessing instead of incremental reprocessing?
6. How would you handle invalid records without stopping the entire pipeline?
7. Why is deleting a Structured Streaming checkpoint risky?
8. How does Delta Lake Time Travel help with recovery?
9. Why should retries use exponential backoff?
10. How would you verify that a recovered pipeline produced correct data?

