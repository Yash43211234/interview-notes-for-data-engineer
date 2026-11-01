# Triggers in Spark Structured Streaming

## 1. What Is a Trigger?

A trigger determines **when Spark executes the next processing cycle** in a Structured Streaming query.

In simple terms, a trigger controls how frequently Spark checks for and processes available streaming data.

For example:
- Process data as quickly as possible.
- Process data every 10 seconds.
- Process currently available data and then stop.

**Key idea:** A trigger controls when processing happens, not which result rows are written.

---

## 2. Types of Triggers

Spark Structured Streaming supports these commonly used trigger types:

1. Default Trigger
2. Processing-Time Trigger
3. Available-Now Trigger
4. Once Trigger (deprecated in newer Spark versions)
5. Continuous Trigger (specialized execution mode)

---

## 3. Default Trigger

The default trigger processes available data in successive micro-batches as quickly as possible.

Spark starts the next micro-batch after the previous one finishes.

### Example

```python
query = (
    stream_df.writeStream
        .format("console")
        .outputMode("append")
        .start()
)
```

No explicit trigger is configured, so Spark uses the default micro-batch behavior.

### Execution Flow

```text
Data Arrives
     |
     v
Micro-batch 1
     |
     v
Processing Completes
     |
     v
Micro-batch 2
     |
     v
Processing Completes
     |
     v
Micro-batch 3
     |
     v
Continue
```

### When to Use

- Continuous data ingestion.
- Applications that should process available data without a fixed interval.
- General-purpose streaming pipelines.

**Remember:** Default trigger does not mean that every record is processed instantly. Spark still uses micro-batches.

---

## 4. Processing-Time Trigger

A processing-time trigger starts micro-batch processing at a configured time interval.

For example, Spark can attempt to start a processing cycle every 10 seconds.

### PySpark Example

```python
query = (
    stream_df.writeStream
        .format("console")
        .outputMode("append")
        .trigger(processingTime="10 seconds")
        .start()
)
```

### How It Works

Suppose the configured interval is 10 seconds.

```text
Time 0s  → Micro-batch 1
Time 10s → Micro-batch 2
Time 20s → Micro-batch 3
Time 30s → Micro-batch 4
```

Spark attempts to follow the configured interval.

If one batch takes longer than the interval, Spark does not run overlapping micro-batches for the same query. The next batch starts after the previous batch finishes, and scheduling may be delayed.

### When to Use

- Periodic data processing.
- Pipelines that do not need immediate processing.
- Workloads where a predictable processing interval is useful.

**Important:** The trigger interval is not a guarantee that every batch will finish within that interval.

---

## 5. Available-Now Trigger

Available-Now processes data that is available when the query starts, potentially across multiple micro-batches, and then stops.

It is useful when we want to process the current available data without keeping the streaming query running continuously.

### PySpark Example

```python
query = (
    stream_df.writeStream
        .format("delta")
        .outputMode("append")
        .option(
            "checkpointLocation",
            "/checkpoints/orders"
        )
        .trigger(availableNow=True)
        .start("/output/orders")
)
```

### How It Works

Suppose 10,000 records are available when the query starts.

```text
Available Data: 10,000 Records
          |
          v
Start Query
          |
          v
Micro-batch 1
          |
          v
Micro-batch 2
          |
          v
Micro-batch 3
          |
          v
Available Data Processed
          |
          v
Query Stops
```

Spark may divide the available data into multiple micro-batches depending on source and configuration.

### When to Use

- Scheduled incremental ingestion.
- Periodic file processing.
- Processing available data and then stopping.
- Batch-style workloads that benefit from streaming source progress and checkpointing.

**Important:** Available-Now processes data available at the start of the run. It does not wait indefinitely for future records.

---

## 6. Once Trigger

The Once trigger processes available data in one micro-batch and then stops.

### PySpark Example

```python
query = (
    stream_df.writeStream
        .format("delta")
        .outputMode("append")
        .option(
            "checkpointLocation",
            "/checkpoints/orders"
        )
        .trigger(once=True)
        .start("/output/orders")
)
```

### How It Works

```text
Available Data
      |
      v
Start Query
      |
      v
One Micro-batch
      |
      v
Process Available Data
      |
      v
Query Stops
```

### Important

- Once processes available data in a single micro-batch.
- It can be inefficient when a large amount of data must be processed in one batch.
- It is deprecated in newer Spark versions in favor of Available-Now.

**Interview point:** Available-Now can use multiple micro-batches, while Once uses one micro-batch.

---

## 7. Continuous Trigger

Continuous processing is a specialized low-latency execution mode.

Unlike normal micro-batch execution, it aims to process records continuously rather than waiting for micro-batch boundaries.

### PySpark Example

```python
query = (
    stream_df.writeStream
        .format("console")
        .outputMode("append")
        .trigger(continuous="1 second")
        .start()
)
```

### Important Characteristics

- Designed for lower-latency processing.
- Uses continuous execution rather than the normal micro-batch model.
- Supports a restricted set of operations and connectors.
- Does not provide the same exactly-once guarantees as standard micro-batch execution.

### When to Use

Only when the workload requires very low latency and is compatible with continuous processing limitations.

**Remember:** Most Structured Streaming applications use micro-batch execution rather than continuous processing.

---

## 8. Comparison of Trigger Types

| Trigger | Processing behavior | Query behavior |
|---|---|---|
| Default | Processes available data as quickly as possible | Keeps running |
| Processing Time | Processes at configured intervals | Keeps running |
| Available-Now | Processes currently available data in one or more batches | Stops after processing |
| Once | Processes available data in one batch | Stops after processing |
| Continuous | Uses continuous execution | Keeps running until stopped or failed |

### Easy Way to Remember

- Default → Process as quickly as possible.
- Processing Time → Process at fixed intervals.
- Available-Now → Process available data in multiple batches if needed, then stop.
- Once → Process available data in one batch, then stop.
- Continuous → Specialized low-latency execution.

---

## 9. Trigger vs Output Mode

These two concepts are frequently confused in interviews.

### Trigger

Controls when Spark processes available data.

Examples:
- Every 10 seconds.
- As quickly as possible.
- Process available data and stop.

### Output Mode

Controls which result rows are written to the sink.

Examples:
- Append.
- Update.
- Complete.

### Example

```python
query = (
    stream_df.writeStream
        .format("console")
        .outputMode("append")
        .trigger(processingTime="30 seconds")
        .start()
)
```

Here:

- `outputMode("append")` writes newly added result rows.
- `trigger(processingTime="30 seconds")` configures a 30-second processing interval.

**Interview answer:** Trigger controls when processing happens, while output mode controls what results are written.

---

## 10. Trigger vs Watermark

A trigger and a watermark solve different problems.

| Feature | Trigger | Watermark |
|---|---|---|
| Purpose | Controls processing schedule | Manages late event-time data and state retention |
| Based on | Processing schedule | Event-time progress |
| Controls | When processing runs | How long certain state is retained |
| Example | Every 10 seconds | 10-minute delay threshold |

### Example

```python
from pyspark.sql.functions import window, sum

result = (
    stream_df
    .withWatermark("event_time", "10 minutes")
    .groupBy(
        window("event_time", "5 minutes")
    )
    .agg(sum("amount").alias("total_amount"))
)

query = (
    result.writeStream
        .format("console")
        .outputMode("append")
        .trigger(processingTime="30 seconds")
        .start()
)
```

In this example:

- The watermark handles late event-time data.
- The 5-minute window groups events.
- The trigger configures the processing interval.
- Append mode writes finalized aggregation results when supported watermark conditions are met.

---

## 11. How to Choose the Right Trigger

| Requirement | Suitable trigger |
|---|---|
| Process data as quickly as possible | Default |
| Process data periodically | Processing Time |
| Process available data and stop | Available-Now |
| Process available data in one micro-batch | Once (deprecated in newer versions) |
| Specialized low-latency execution | Continuous |

### Practical Scenarios

**Scenario 1: Real-time order monitoring**

New orders should be processed as quickly as possible.

Suitable choice: Default trigger.

**Scenario 2: Periodic analytics**

Process incoming data every 30 seconds.

Suitable choice: Processing-Time trigger.

**Scenario 3: Scheduled file ingestion**

A scheduled job should process files currently available and then stop.

Suitable choice: Available-Now trigger.

**Scenario 4: Low-latency processing**

The application requires continuous execution and supports its limitations.

Suitable choice: Continuous trigger.

---

## 12. Common Mistakes

1. Confusing trigger with output mode.
2. Thinking the default trigger processes every record immediately.
3. Assuming a processing-time interval guarantees batch completion within that interval.
4. Confusing Available-Now with Once.
5. Using Once without knowing it is deprecated in newer Spark versions.
6. Assuming Available-Now waits for future data indefinitely.
7. Assuming continuous processing supports every DataFrame operation.
8. Assuming all triggers use the same execution model.
9. Confusing trigger intervals with watermark thresholds.

---

## 13. Interview Questions

**Q1. What is a trigger in Structured Streaming?**

A trigger determines when Spark executes the next processing cycle for a streaming query.

**Q2. What is the default trigger?**

It processes available data in successive micro-batches as quickly as possible.

**Q3. What is a processing-time trigger?**

It schedules micro-batch processing at a configured time interval.

**Q4. What happens if a micro-batch takes longer than the configured interval?**

The next micro-batch waits until the current one finishes. The query does not run overlapping micro-batches.

**Q5. What is Available-Now?**

It processes data available when the query starts, potentially using multiple micro-batches, and then stops.

**Q6. What is Once trigger?**

It processes available data in one micro-batch and then stops. It is deprecated in newer Spark versions.

**Q7. What is the difference between Available-Now and Once?**

Available-Now can process data using multiple micro-batches, while Once processes it in a single micro-batch.

**Q8. What is continuous processing?**

It is a specialized low-latency execution mode that processes records continuously rather than using normal micro-batch execution.

**Q9. What is the difference between trigger and output mode?**

Trigger controls when processing happens. Output mode controls which result rows are written.

**Q10. What is the difference between trigger and watermark?**

Trigger controls the processing schedule. Watermark manages late event-time data and state retention.

**Q11. Which trigger is suitable for scheduled incremental ingestion?**

Available-Now is commonly suitable when the requirement is to process available data and then stop.

**Q12. Does a processing-time trigger guarantee real-time processing?**

No. It configures a processing interval, but actual execution can be delayed by processing time and resource availability.

---

## 14. Quick Revision

| Concept | Remember |
|---|---|
| Trigger | Controls when processing happens |
| Default | Processes available data as quickly as possible |
| Processing Time | Processes at configured intervals |
| Available-Now | Processes available data, then stops |
| Once | One micro-batch, then stops; deprecated in newer versions |
| Continuous | Specialized low-latency execution |
| Output Mode | Controls which result rows are written |
| Watermark | Manages late event-time data and state retention |

### One-Line Revision

- Default → As fast as possible.
- Processing Time → Fixed interval.
- Available-Now → Multiple batches if needed, then stop.
- Once → One batch, then stop.
- Continuous → Low-latency continuous execution.

---

## 15. Self-Check

- [ ] Can I explain what a trigger is?
- [ ] Can I explain the default trigger?
- [ ] Can I configure a processing-time trigger?
- [ ] Can I explain Available-Now?
- [ ] Can I differentiate Available-Now and Once?
- [ ] Can I explain continuous processing?
- [ ] Can I differentiate trigger and output mode?
- [ ] Can I differentiate trigger and watermark?
- [ ] Can I select a trigger for a real-world scenario?

**Final takeaway:** A trigger controls when Spark processes streaming data. Understand the difference between Default, Processing Time, Available-Now, Once, and Continuous triggers, especially the difference between Available-Now and Once for interviews.