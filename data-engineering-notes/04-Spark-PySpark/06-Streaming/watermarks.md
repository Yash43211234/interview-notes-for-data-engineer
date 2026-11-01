# Watermarks in Spark Structured Streaming

## 1. What Is a Watermark?

A watermark is a mechanism in Spark Structured Streaming used to handle late-arriving data in event-time-based streaming queries.

It helps Spark determine how long to wait for delayed events and when it can clean up old state maintained by stateful operations.

**Key idea:** A watermark helps manage late data and control state growth in streaming applications.

### Why Do We Need Watermarks?

Imagine an e-commerce application receiving orders.

| Order ID | Event Time | Arrival Time |
|---|---|---|
| 101 | 10:00 AM | 10:01 AM |
| 102 | 10:02 AM | 10:03 AM |
| 103 | 10:01 AM | 10:08 AM |

Order 103 happened at 10:01 AM but arrived at 10:08 AM.

This is called **late-arriving data**.

Without a watermark, Spark may need to retain aggregation state for a long time to handle delayed records.

Watermarks help Spark decide when older event-time data is too late for a particular stateful operation.

---

## 2. Event Time vs Processing Time

To understand watermarks, first understand two types of time.

### Event Time

Event time is the timestamp when an event actually occurred.

Example:

An order was placed at 10:00 AM.

Its event time is 10:00 AM.

### Processing Time

Processing time is when Spark processes the event.

If the order reaches Spark at 10:05 AM, its processing time may be 10:05 AM.

### Comparison

| Feature | Event Time | Processing Time |
|---|---|---|
| Meaning | When the event happened | When Spark processed it |
| Comes from | Event timestamp | Processing system clock |
| Used for | Time-based business calculations | Processing schedules |
| Example | Order created at 10:00 | Spark processes it at 10:05 |

**Interview point:** Watermarks are based on event-time progress, not simply the wall-clock time when Spark processes a batch.

---

## 3. How Does a Watermark Work?

A watermark tracks event-time progress and establishes a threshold for handling late-arriving data.

Suppose we configure:

```python
.withWatermark("event_time", "10 minutes")
```

This means Spark uses a 10-minute delay threshold when calculating the watermark.

Conceptually:

\[
Watermark = Maximum\ Observed\ Event\ Time - Delay\ Threshold
\]

For example:

- Maximum observed event time = 10:30 AM.
- Configured delay threshold = 10 minutes.
- Watermark = 10:20 AM.

```text
Maximum Event Time: 10:30 AM
          |
          v
Delay Threshold: 10 minutes
          |
          v
Watermark: 10:20 AM
```

Events older than the watermark may be considered too late by the relevant stateful operation.

**Important:** This formula is a simplified conceptual explanation. Spark's actual watermark progression and state cleanup depend on batch execution, query structure, and the event-time information available to Spark.

---

## 4. Creating a Watermark in PySpark

Use the `withWatermark()` method.

### Syntax

```python
df.withWatermark("event_time", "10 minutes")
```

### Parameters

| Parameter | Meaning |
|---|---|
| `event_time` | Timestamp column used for event-time processing |
| `10 minutes` | Maximum configured delay threshold |

### Example

```python
from pyspark.sql.functions import window, sum

result = (
    stream_df
    .withWatermark("event_time", "10 minutes")
    .groupBy(
        window("event_time", "5 minutes"),
        "country"
    )
    .agg(
        sum("amount").alias("total_sales")
    )
)
```

### Explanation

- `withWatermark()` defines the event-time delay threshold.
- `window()` creates 5-minute event-time windows.
- `groupBy()` groups records by time window and country.
- `sum()` calculates total sales.

The watermark helps Spark manage the state required for the windowed aggregation.

---

## 5. Watermark with Window Aggregation

Window aggregation is one of the most common uses of watermarks.

Suppose an application receives sales events continuously and needs to calculate sales every 5 minutes.

### Input Data

| Event Time | Country | Amount |
|---|---|---:|
| 10:01 | India | 500 |
| 10:03 | India | 300 |
| 10:06 | USA | 400 |
| 10:08 | USA | 200 |

### PySpark Example

```python
from pyspark.sql.functions import window, sum

result = (
    stream_df
    .withWatermark("event_time", "10 minutes")
    .groupBy(
        window("event_time", "5 minutes"),
        "country"
    )
    .agg(
        sum("amount").alias("total_sales")
    )
)
```

### Window Results

| Window | Country | Total Sales |
|---|---|---:|
| 10:00–10:05 | India | 800 |
| 10:05–10:10 | USA | 600 |

Spark maintains the aggregation state for windows that have not yet been finalized or cleaned up.

With Append mode, a window's final result can be emitted once the watermark has advanced sufficiently beyond the window's end time.

---

## 6. What Happens to Late-Arriving Data?

Suppose the configured watermark delay is 10 minutes.

Assume the watermark has advanced to 10:20 AM.

Now a new record arrives with event time 10:15 AM.

It is older than the watermark.

For a stateful window aggregation, Spark may drop this record because the relevant state may already have been cleaned up.

### Example

| Event Time | Watermark | Expected Behavior |
|---|---|---|
| 10:25 AM | 10:20 AM | Not older than watermark |
| 10:21 AM | 10:20 AM | Not older than watermark |
| 10:18 AM | 10:20 AM | May be considered too late |
| 10:10 AM | 10:20 AM | May be considered too late |

**Important:** The exact behavior depends on the stateful operator and query. A watermark is not a universal filter that automatically removes every old record from every streaming DataFrame.

---

## 7. Watermark and Output Modes

Watermarks are particularly useful with stateful aggregations.

Consider a 5-minute event-time window.

### Append Mode

Append mode writes a result only after Spark considers the window final under the watermark policy.

Example:

```python
query = (
    result.writeStream
        .format("console")
        .outputMode("append")
        .start()
)
```

This avoids repeatedly writing intermediate aggregate results for the same window.

### Update Mode

Update mode writes aggregate rows that changed during the latest trigger.

```python
query = (
    result.writeStream
        .format("console")
        .outputMode("update")
        .start()
)
```

This is useful when consumers need updated results before the window is finalized.

### Complete Mode

Complete mode writes the entire current aggregation result table after each trigger.

```python
query = (
    result.writeStream
        .format("console")
        .outputMode("complete")
        .start()
)
```

Complete mode has different state-retention and cleanup considerations. Do not assume that adding a watermark automatically makes every aggregation state eligible for cleanup in Complete mode.

---

## 8. Why Are Watermarks Important?

Watermarks solve two important problems.

### 1. Handling Late Data

Real-world streaming systems do not always receive events in chronological order.

Watermarks allow supported stateful operations to accept delayed events within a configured threshold.

### 2. Controlling State Growth

Stateful operations maintain information across micro-batches.

Without appropriate state cleanup, state can continue growing.

Watermarks help Spark determine when older state can be removed for supported operations.

### Benefits

- Handles delayed event-time records.
- Supports finalization of windowed aggregations.
- Helps control state size.
- Improves the manageability of long-running streaming queries.

---

## 9. Watermark vs Trigger vs Event Time

These concepts are frequently confused in interviews.

| Concept | Purpose |
|---|---|
| Event Time | When an event actually happened |
| Watermark | Tracks event-time progress and helps manage late data and state |
| Trigger | Determines when Spark executes processing |

### Example

```python
from pyspark.sql.functions import window, sum

result = (
    stream_df
    .withWatermark("event_time", "10 minutes")
    .groupBy(window("event_time", "5 minutes"))
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

- Event time comes from `event_time`.
- Watermark uses a 10-minute delay threshold.
- Window groups events into 5-minute windows.
- Trigger schedules processing every 30 seconds.
- Append mode writes finalized aggregation results.

---

## 10. Watermark with Multiple Streaming Inputs

When a query has multiple streaming inputs, each input can have its own watermark.

Spark can use a global watermark to manage stateful operations involving multiple streams.

By default, Spark generally uses the minimum watermark across the input streams. This allows the slower stream to continue contributing late data, but can delay state cleanup.

The following configuration changes the global watermark policy to use the maximum watermark:

```python
spark.conf.set(
    "spark.sql.streaming.multipleWatermarkPolicy",
    "max"
)
```

### Difference

| Policy | Behavior |
|---|---|
| `min` | Uses the slowest watermark; safer for late data from slower streams |
| `max` | Uses the fastest watermark; can clean state sooner but may drop more late data from slower streams |

**Important:** Choosing `max` can cause records from slower streams to be considered too late. Use it only when that trade-off is acceptable.

---

## 11. Common Mistakes

1. Confusing event time with processing time.
2. Thinking a watermark is the same as a trigger.
3. Assuming a watermark guarantees that all late data will be accepted within the configured duration.
4. Assuming every record older than the watermark is automatically filtered from every query.
5. Forgetting that watermarks are especially important for stateful operations.
6. Assuming a watermark automatically removes all state in every output mode.
7. Assuming a 10-minute watermark means Spark waits exactly 10 minutes before processing a window.
8. Ignoring the effect of multiple input streams on watermark progression.
9. Assuming watermarking alone guarantees exactly-once processing.

---

## 12. Interview Questions

**Q1. What is a watermark in Spark Structured Streaming?**

A watermark is a mechanism that tracks event-time progress and helps manage late-arriving data and state cleanup in supported stateful operations.

**Q2. Why do we need watermarks?**

To handle delayed event-time records and prevent state from growing indefinitely in supported stateful queries.

**Q3. What is the difference between event time and processing time?**

Event time represents when the event occurred, while processing time represents when Spark processed it.

**Q4. How do you define a watermark in PySpark?**

Using:

```python
df.withWatermark("event_time", "10 minutes")
```

**Q5. What does a 10-minute watermark mean?**

It defines a 10-minute event-time delay threshold used by Spark to manage late data and state retention. It does not mean Spark always waits exactly 10 minutes.

**Q6. What happens when data arrives later than the watermark?**

For relevant stateful operations, Spark may drop the record if the required state has already been cleaned up.

**Q7. Can we use watermarks with window aggregations?**

Yes. Watermarks are commonly used with event-time window aggregations to manage late records and state cleanup.

**Q8. How does a watermark help control state growth?**

It allows Spark to identify older event-time state that is no longer needed and clean it up when supported by the operation.

**Q9. What is the difference between watermark and trigger?**

Watermark manages event-time progress and late data. Trigger controls when Spark processes available data.

**Q10. Does a watermark guarantee that no late records will be lost?**

No. Records arriving beyond the supported delay threshold may be dropped from the relevant stateful operation.

**Q11. What happens when a query has multiple streaming inputs?**

Spark tracks watermarks for the inputs and uses a global watermark policy to coordinate state management.

**Q12. What is the default multiple-watermark policy?**

The default policy is generally `min`, which uses the slowest input watermark.

---

## 13. Quick Revision

| Concept | Remember |
|---|---|
| Watermark | Manages late event-time data |
| Event Time | Actual event timestamp |
| Processing Time | Time Spark processes the event |
| Delay Threshold | Configured lateness threshold |
| `withWatermark()` | Defines the watermark |
| Window Aggregation | Common use case |
| Stateful Operation | Maintains information across batches |
| State Cleanup | Removes older state when eligible |
| Multiple Watermarks | Coordinates multiple streaming inputs |
| Default Policy | `min` |

### One-Line Revision

- Watermark → Manages late data.
- Event time → When the event happened.
- Processing time → When Spark processed it.
- Delay threshold → Allowed event-time delay for the query.
- State cleanup → Removes old state when eligible.
- `min` policy → Uses the slowest input watermark.

---

## 14. Self-Check

- [ ] Can I explain what a watermark is?
- [ ] Can I differentiate event time and processing time?
- [ ] Can I explain how a watermark works?
- [ ] Can I use `withWatermark()` in PySpark?
- [ ] Can I explain watermark with window aggregation?
- [ ] Can I explain what happens to late-arriving data?
- [ ] Can I explain how watermarks control state growth?
- [ ] Can I differentiate watermark and trigger?
- [ ] Can I explain multiple-input watermark policies?
- [ ] Can I explain the limitations of watermarks?

**Final takeaway:** Watermarks help Spark Structured Streaming manage late-arriving event-time data and clean up old state in supported stateful operations. Understand event time, delay thresholds, window aggregations, and state cleanup for interviews.