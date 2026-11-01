# Debugging Common Errors in Spark Structured Streaming

## 1. What Is Debugging?

Debugging is the process of identifying, understanding, and fixing errors that occur during application execution.

In Spark Structured Streaming, errors can occur while:

- Reading streaming data.
- Validating schemas.
- Processing micro-batches.
- Managing state.
- Writing output.
- Recovering from failures.
- Managing checkpoints.

**Key idea:** Debugging a streaming pipeline requires identifying where the failure occurs and checking the relevant logs, query status, Spark UI, and checkpoint information.

---

## 2. Common Categories of Errors

| Category | Common Problem |
|---|---|
| Schema errors | Unexpected or incompatible schema |
| Source errors | Missing files or unavailable source |
| Transformation errors | Invalid expressions or unsupported operations |
| State errors | State store or state management problems |
| Checkpoint errors | Invalid or incompatible checkpoint |
| Sink errors | Permission or write failures |
| Memory errors | Executor or driver out of memory |
| Performance issues | Slow batches and increasing processing delay |

---

## 3. Schema-Related Errors

### Problem

Spark Structured Streaming generally requires a schema for file-based streaming sources such as JSON and CSV.

Example:

```python
stream_df = (
    spark.readStream
        .format("json")
        .load("/input/orders")
)
```

This can fail because Spark cannot automatically infer the schema for a streaming file source.

### Solution

Define the schema explicitly.

```python
from pyspark.sql.types import (
    StructType,
    StructField,
    IntegerType,
    DoubleType
)

schema = StructType([
    StructField("order_id", IntegerType(), True),
    StructField("amount", DoubleType(), True)
])

stream_df = (
    spark.readStream
        .format("json")
        .schema(schema)
        .load("/input/orders")
)
```

**Remember:** Schema inference behavior depends on the source. For file-based streaming sources, provide an explicit schema.

---

## 4. Checkpoint-Related Errors

### What Is a Checkpoint?

A checkpoint stores information required to recover a streaming query after failure.

It can contain:
- Query progress and committed batch information.
- Source offset information.
- Stateful operator information.

### Common Problems

1. Checkpoint directory is missing or inaccessible.
2. Incorrect permissions.
3. Reusing a checkpoint with an incompatible query.
4. Deleting checkpoint files accidentally.
5. Starting multiple queries that incorrectly share the same checkpoint location.

### Example

```python
query = (
    stream_df.writeStream
        .format("parquet")
        .option(
            "checkpointLocation",
            "/checkpoints/orders"
        )
        .start("/output/orders")
)
```

### Solution

- Use a reliable storage location.
- Ensure Spark has read and write permissions.
- Use a separate checkpoint location for each independent query.
- Preserve the checkpoint when recovering the same query.
- Avoid manually deleting checkpoint files while a query is active.

**Important:** Changing a query's source, stateful operations, or other important characteristics may make an existing checkpoint incompatible. Recovery compatibility depends on the changes and Spark version.

---

## 5. Output Mode Errors

Structured Streaming supports three main output modes:

| Mode | Behavior |
|---|---|
| Append | Writes newly added result rows |
| Update | Writes rows that changed |
| Complete | Writes the entire result table |

### Common Problem

A query may fail because the selected output mode is unsupported for its operations or sink.

Example:

```python
query = (
    stream_df.writeStream
        .outputMode("complete")
        .format("parquet")
        .option(
            "checkpointLocation",
            "/checkpoints/orders"
        )
        .start("/output/orders")
)
```

The combination of output mode, query type, and sink must be supported.

### Solution

Choose a compatible output mode.

For example, a simple streaming filter commonly uses Append mode:

```python
query = (
    stream_df.filter("amount > 0")
        .writeStream
        .format("parquet")
        .outputMode("append")
        .option(
            "checkpointLocation",
            "/checkpoints/valid_orders"
        )
        .start("/output/valid_orders")
)
```

**Interview point:** Output mode compatibility depends on the streaming query and the sink.

---

## 6. Watermark and State-Related Errors

### What Is State?

State is information that Spark maintains across micro-batches for stateful operations.

Examples:
- Window aggregations.
- Streaming deduplication.
- Stream-stream joins.

### Common Problems

- State grows continuously.
- Late events affect expected results.
- State store failures.
- Large state causes memory or disk pressure.
- Incorrect watermark configuration.

### Example

```python
from pyspark.sql.functions import window

result = (
    stream_df
        .withWatermark("event_time", "10 minutes")
        .groupBy(
            window("event_time", "5 minutes")
        )
        .count()
)
```

### Solution

1. Check whether the event-time column is valid.
2. Confirm that timestamps are parsed correctly.
3. Choose an appropriate watermark delay.
4. Check state store metrics.
5. Monitor the number of state rows and memory usage.

**Important:** A watermark helps manage late data and state cleanup in supported stateful operations. It does not guarantee that every record arriving within the configured delay will be processed.

---

## 7. Sink-Related Errors

A sink is the destination where streaming output is written.

Examples:
- Files.
- Delta tables.
- Kafka.
- Console.

### Common Problems

- Insufficient write permissions.
- Invalid output path.
- Destination unavailable.
- Schema mismatch.
- Network or connectivity failure.
- Duplicate writes after retries with custom processing.

### Example

```python
query = (
    stream_df.writeStream
        .format("delta")
        .option(
            "checkpointLocation",
            "/checkpoints/orders"
        )
        .start("/data/delta/orders")
)
```

### Solution

- Verify the destination path.
- Check permissions.
- Validate the output schema.
- Check destination connectivity.
- Inspect sink-specific error messages.
- Design writes to handle retries safely.

**Important:** Exactly-once behavior depends on the source, sink, checkpointing, and write implementation. A checkpoint alone does not guarantee exactly-once results for every external sink.

---

## 8. Memory and Out-of-Memory Errors

### Common Errors

- Driver out of memory.
- Executor out of memory.
- Excessive state growth.
- Large shuffle operations.
- Too much data collected on the driver.

### Example of a Risky Operation

```python
df.collect()
```

`collect()` transfers all rows to the driver. If the dataset is large, it can cause driver memory problems.

### Better Approach

Use distributed operations where possible.

```python
df.groupBy("status").count().show()
```

### Solution

1. Avoid unnecessary `collect()`.
2. Filter data early.
3. Avoid unnecessary shuffles.
4. Tune partition counts appropriately.
5. Monitor executor memory.
6. Check whether state is growing without bounds.
7. Review Spark UI for memory and spill metrics.

**Remember:** Increasing memory is not always the best solution. First identify why the application is consuming excessive memory.

---

## 9. Slow Micro-Batches and Processing Delays

### Problem

A streaming query may process data slower than it arrives.

For example:

- New data arrives every 10 seconds.
- A micro-batch takes 30 seconds to finish.

The query may develop processing delays and backlog.

### Common Causes

- Large data volume.
- Data skew.
- Expensive transformations.
- Shuffle-heavy joins.
- Small or inefficient partitions.
- Slow source or sink.
- Insufficient cluster resources.

### Solution

1. Check batch processing time.
2. Check input rows per batch.
3. Inspect Spark UI stages.
4. Identify expensive transformations.
5. Review shuffle read and write metrics.
6. Check for data skew.
7. Optimize joins and partitions.
8. Verify source and sink performance.

**Interview point:** Compare the time between batches with their processing duration and inspect input-rate and processing-rate metrics.

---

## 10. Unsupported Operations in Streaming

Not every batch DataFrame operation is supported in every streaming query.

Some operations require special handling or have restrictions depending on the source, sink, output mode, and Spark version.

### Common Examples

- Certain global sorting operations.
- Some aggregation and deduplication combinations.
- Operations that require an unsupported streaming output mode.
- Batch-only actions applied directly to streaming DataFrames.

### Example of an Invalid Pattern

```python
stream_df.collect()
```

A streaming DataFrame cannot be collected like a normal batch DataFrame.

### Better Approach

Use a streaming output operation:

```python
query = (
    stream_df.writeStream
        .format("console")
        .outputMode("append")
        .start()
)
```

For custom batch processing, use `foreachBatch()`.

```python
def process_batch(batch_df, batch_id):
    batch_df.show()

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

Inside `foreachBatch()`, `batch_df` is a regular batch DataFrame, so batch operations can be applied to it.

---

## 11. How to Debug a Streaming Query

Spark provides several useful methods and interfaces.

### 1. Check Query Status

```python
query.status
```

Returns information about the current status of the streaming query.

### 2. Check Query Progress

```python
query.lastProgress
```

Returns progress information for the most recently completed trigger, when available.

### 3. Check Recent Progress

```python
query.recentProgress
```

Returns recent progress information for the query.

### 4. Check Whether the Query Is Active

```python
query.isActive
```

Returns whether the query is currently active.

### 5. Check the Exception

```python
query.exception()
```

Returns the exception that terminated the query, if one is available.

### 6. Wait for Query Termination

```python
query.awaitTermination()
```

Waits for the streaming query to terminate.

**Important:** `awaitTermination()` does not start the query. The query starts when `.start()` is called.

---

## 12. Spark UI for Debugging

Spark UI is one of the most useful tools for understanding execution and performance problems.

Important sections:

| Section | What to Check |
|---|---|
| Jobs | Job execution and status |
| Stages | Stage duration and failures |
| Tasks | Failed or slow tasks |
| SQL | Query execution plans |
| Executors | Memory, CPU, and task metrics |
| Structured Streaming | Streaming query and batch progress, where supported |

### Useful Metrics

- Input rows.
- Input rate.
- Processing rate.
- Batch duration.
- Scheduling delay.
- State operator metrics.
- Shuffle read and write.
- Memory spill and disk spill.
- Failed tasks.

### Example

If one task takes much longer than the others, investigate:
- Data skew.
- Uneven partition sizes.
- Expensive transformations.
- Resource constraints.

**Remember:** Spark UI helps identify where time and resources are being consumed.

---

## 13. A Practical Debugging Workflow

Use this sequence when a streaming pipeline fails or becomes slow.

```text
Streaming Query Problem
          |
          v
Check Query Status
          |
          v
Check Exception and Logs
          |
          v
Identify Failure Area
          |
          v
Check Spark UI and Metrics
          |
          v
Check Source / Transformation / Sink
          |
          v
Fix the Root Cause
          |
          v
Restart or Recover Safely
          |
          v
Verify Output
```

### Step-by-Step Checklist

1. Check whether the query is active.
2. Read the exception and driver/executor logs.
3. Identify whether the failure is related to the source, transformation, state, checkpoint, or sink.
4. Check the last completed micro-batch.
5. Inspect Spark UI and query progress.
6. Fix the underlying issue.
7. Restart using the appropriate checkpoint strategy.
8. Verify that output is correct and that retries have not caused duplicate results.

---

## 14. Common Errors: Quick Reference

| Problem | Possible Cause | What to Check |
|---|---|---|
| Schema inference failure | Missing schema for file stream | Define explicit schema |
| Checkpoint failure | Invalid path or permissions | Check checkpoint storage |
| Output mode error | Unsupported mode | Check query and sink compatibility |
| State store failure | State management issue | Check state metrics and logs |
| Out of memory | Excessive memory usage | Check driver and executor metrics |
| Slow batches | Expensive processing | Inspect Spark UI |
| Duplicate output | Retry or non-idempotent writes | Review sink write logic |
| Missing records | Filtering, parsing, or late-data handling | Inspect validation and watermark logic |
| Query stopped | Runtime exception | Check exception and logs |
| Permission error | Insufficient access | Verify storage permissions |

---

## 15. Interview Questions

**Q1. How do you debug a failed Structured Streaming query?**

I first check the query status and exception. Then I inspect driver and executor logs, Spark UI, and the last completed micro-batch to identify whether the issue is related to the source, transformation, state, checkpoint, or sink.

**Q2. What is a checkpoint error?**

It occurs when Spark cannot access or use the checkpoint information required for query recovery. Common causes include permissions, missing files, or incompatible query changes.

**Q3. How do you debug slow streaming applications?**

I check batch duration, input rate, processing rate, scheduling delay, shuffle metrics, and task execution time in Spark UI. Then I investigate skew, expensive transformations, and source or sink bottlenecks.

**Q4. What causes state store problems?**

Large or continuously growing state, incorrect state management, storage issues, and resource constraints can cause state store problems.

**Q5. How do you handle out-of-memory errors?**

I identify whether the problem is on the driver or executors, inspect memory and spill metrics, avoid unnecessary data collection, optimize transformations, and review partitioning and state size.

**Q6. What is the difference between `lastProgress` and `recentProgress`?**

`lastProgress` returns the most recent completed trigger's progress information, while `recentProgress` returns a collection of recent trigger progress records.

**Q7. Why should we not delete checkpoint files during recovery?**

Checkpoint files contain important recovery information, including progress and state. Deleting them can cause the query to lose its previous recovery position or state.

**Q8. What happens if a streaming micro-batch fails?**

Spark can retry or recover processing depending on the failure, source, sink, and checkpoint behavior. Safe recovery also depends on the sink handling repeated writes correctly.

**Q9. How do you identify data skew?**

I inspect task duration, partition sizes, shuffle metrics, and differences between the fastest and slowest tasks in Spark UI.

**Q10. Does checkpointing guarantee exactly-once processing?**

No. Checkpointing supports recovery and tracks progress, but end-to-end exactly-once results depend on source and sink guarantees and the implementation of writes.

---

## 16. Quick Revision

| Topic | Remember |
|---|---|
| Schema Error | Define and validate the schema |
| Checkpoint Error | Check path, permissions, compatibility |
| Output Mode Error | Check query and sink support |
| State Error | Inspect state store and watermark |
| Sink Error | Check permissions and connectivity |
| OOM Error | Investigate driver/executor memory |
| Slow Batch | Check processing time and Spark UI |
| Query Status | `query.status` |
| Latest Progress | `query.lastProgress` |
| Recent Progress | `query.recentProgress` |
| Query Exception | `query.exception()` |
| Query Active | `query.isActive` |
| Wait for Termination | `query.awaitTermination()` |

### One-Line Revision

- Schema error → Check schema.
- Checkpoint error → Check recovery files and permissions.
- Output mode error → Check compatibility.
- State error → Check state growth and watermark.
- Sink error → Check destination.
- OOM error → Check memory and data volume.
- Slow batch → Check Spark UI.
- Query failure → Check exception and logs.

---

## 17. Self-Check

- [ ] Can I explain common Structured Streaming errors?
- [ ] Can I debug schema-related errors?
- [ ] Can I explain checkpoint failures?
- [ ] Can I identify output mode compatibility issues?
- [ ] Can I explain state store and watermark problems?
- [ ] Can I troubleshoot sink failures?
- [ ] Can I identify driver and executor memory problems?
- [ ] Can I investigate slow micro-batches?
- [ ] Can I use `query.status` and `query.lastProgress`?
- [ ] Can I use Spark UI to identify performance bottlenecks?
- [ ] Can I explain how to recover a failed streaming query?

**Final takeaway:** Effective Spark Streaming debugging is not just about fixing an error message. It is about identifying the root cause, checking query progress and execution metrics, recovering safely, and verifying that the output remains correct.