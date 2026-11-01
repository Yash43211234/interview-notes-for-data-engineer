

# Watermarks and Late-Arriving Events

## 1. Introduction

In streaming systems, events do not always arrive in the order in which they occurred.

An event may be delayed because of:
- Network latency.
- Device connectivity problems.
- Kafka buffering.
- Processing backlogs.
- Retries.
- Offline applications.

This creates a challenge when calculating time-based aggregations.

**Watermarks** help stream-processing engines track event-time progress and manage late-arriving events.

This note focuses on watermarks and late-event handling in Apache Spark Structured Streaming, with comparisons to Apache Flink.

---

## 2. What Is a Late-Arriving Event?

A late-arriving event is an event that reaches the processing system later than expected relative to its event timestamp.

### Example

An order is placed at 10:02 AM, but because of network delays, it reaches Spark at 10:15 AM.

| Timestamp | Time |
|---|---|
| Event Time | 10:02 AM |
| Ingestion Time | 10:12 AM |
| Processing Time | 10:15 AM |
| Arrival Delay | 10 minutes |

The order actually occurred at 10:02 AM.

Even though Spark processes it at 10:15 AM, its event timestamp remains 10:02 AM.

### Why does late data matter?

Suppose the business wants to calculate order revenue in five-minute windows.

The order belongs to:

```text
10:00 AM – 10:05 AM
```

But it arrives at 10:15 AM.

The streaming system must decide whether to:
- Include the event in the earlier window.
- Wait longer before finalizing the window.
- Drop it after a configured lateness threshold.
- Process it through a separate correction or reconciliation pipeline.

---

## 3. What Is a Watermark?

A watermark is a mechanism used in event-time stream processing to track event-time progress.

It helps the engine determine when it can finalize certain event-time computations and clean up state that is no longer needed.

In Spark Structured Streaming, `withWatermark()` defines a delay threshold for event-time state management.

### Example

```python
df = df.withWatermark(
    "event_time",
    "10 minutes"
)
```

This configures a 10-minute watermark delay threshold.

### What does a watermark help with?

1. Tracks event-time progress.
2. Handles out-of-order events.
3. Controls late-data handling for supported stateful operations.
4. Allows old aggregation state to be cleaned up.
5. Helps determine when event-time windows can be finalized.

**Important:** A watermark is not a timer that waits exactly ten minutes after each event. It is based on observed event-time progress, not simply the current system clock.

---

## 4. How Does a Watermark Work?

Suppose Spark observes events with the following timestamps:

| Event | Event Time |
|---|---|
| A | 10:00 |
| B | 10:05 |
| C | 10:10 |
| D | 10:15 |
| E | 10:20 |

Assume the configured watermark delay is 10 minutes.

As event-time progress advances, Spark's watermark can advance approximately as follows:

\[
Watermark = MaximumObservedEventTime - Delay
\]

When the maximum observed event time reaches 10:20 AM:

\[
Watermark = 10{:}20 - 10\text{ minutes}
\]

\[
Watermark \approx 10{:}10
\]

This means Spark has advanced its event-time progress to approximately 10:10 AM.

### Visual representation

```text
Maximum observed event time: 10:20
Configured delay:             10 minutes
                              |
                              v
Approximate watermark:        10:10
```

Events earlier than the watermark may be considered too late for supported stateful operations.

### Important technical detail

Spark's watermark is not necessarily updated immediately after every individual record.

It is tracked through streaming execution and micro-batch progress. The exact behavior depends on the query, trigger, input sources, and watermark policy.

---

## 5. What Happens When a Late Event Arrives?

Consider a five-minute aggregation window:

```text
10:00 AM – 10:05 AM
```

Suppose the watermark has already advanced beyond 10:05 AM.

A new event arrives with:

```text
event_time = 10:03 AM
```

This event belongs to the 10:00–10:05 window, but that window may already have been finalized and its state removed.

For supported stateful aggregations, Spark may drop this event rather than update the finalized result.

### Example timeline

```text
10:00       10:05       10:15       10:20
  |-----------|-----------|-----------|
  Window      Event       Watermark   Late event
  starts      window      advances    arrives
              ends
```

The event may be too late for the original aggregation.

**Important:** Whether an event is accepted depends on the operation and watermark progress. A configured delay does not guarantee acceptance of every event within that age.

---

## 6. Watermark and Event-Time Windows

Watermarks are especially useful for stateful event-time aggregations.

Common window types include:

### A. Tumbling Window

A fixed-duration window without overlap.

Example: Five-minute windows.

```text
10:00 ───── 10:05
10:05 ───── 10:10
10:10 ───── 10:15
```

### B. Sliding Window

A fixed-duration window that overlaps with other windows.

Example:
- Window duration: 10 minutes.
- Slide interval: 5 minutes.

```text
Window 1: 10:00 ───── 10:10
Window 2: 10:05 ───── 10:15
Window 3: 10:10 ───── 10:20
```

### C. Session Window

Groups events separated by periods of inactivity.

Example:

A user watches videos. If there is no activity for 30 minutes, the session may end.

### Why are watermarks needed?

Without appropriate state cleanup, a streaming aggregation may need to maintain information about old windows for an unnecessarily long time.

Watermarks help determine when state for completed event-time computations can be removed.

---

## 7. Watermark in PySpark Structured Streaming

Consider an e-commerce application that receives order events.

Each event contains:

```json
{
  "order_id": 101,
  "customer_id": "C01",
  "amount": 500.0,
  "event_time": "2026-10-25T10:02:00"
}
```

The business wants to calculate order count and revenue every five minutes.

### Step 1: Import required functions

```python
from pyspark.sql.functions import (
    col,
    window,
    count,
    sum
)
```

### Step 2: Define the watermark

```python
orders_with_watermark = parsed_orders.withWatermark(
    "event_time",
    "10 minutes"
)
```

This defines a 10-minute event-time delay threshold.

### Step 3: Create event-time windows

```python
result = (
    orders_with_watermark
    .groupBy(
        window(col("event_time"), "5 minutes")
    )
    .agg(
        count("*").alias("total_orders"),
        sum("amount").alias("total_revenue")
    )
)
```

### Step 4: Write the result

```python
query = (
    result.writeStream
    .outputMode("append")
    .format("console")
    .option(
        "checkpointLocation",
        "/tmp/orders-checkpoint"
    )
    .start()
)

query.awaitTermination()
```

### Explanation

| Code | Purpose |
|---|---|
| `withWatermark()` | Configures event-time delay threshold |
| `window()` | Creates time-based windows |
| `groupBy()` | Groups events by window |
| `count()` | Counts orders |
| `sum()` | Calculates revenue |
| `append` | Emits finalized aggregation results |
| `checkpointLocation` | Stores streaming progress and state for recovery |

### Why use Append mode?

For this event-time aggregation, Append mode emits a window after the watermark has passed its end and the window is considered complete.

This avoids repeatedly outputting updates to the same window.

---

## 8. Watermark and Output Modes

Spark Structured Streaming supports three main output modes.

### A. Append Mode

Only newly added rows are written.

For event-time aggregations with watermarks, completed windows can be emitted after the watermark passes their end.

### B. Update Mode

Only rows updated since the previous trigger are written.

This is useful when downstream systems need changing aggregation results.

### C. Complete Mode

The entire result table is written on each trigger.

### Comparison

| Output Mode | Behavior |
|---|---|
| Append | Writes new result rows |
| Update | Writes rows updated since the previous trigger |
| Complete | Writes the entire result table |

**Important:** Output mode is not a delivery guarantee. It determines which result rows are emitted, not whether an event is processed exactly once.

---

## 9. Watermark vs Checkpointing

These concepts solve different problems.

### Watermark

Manages event-time progress and helps control state retention and late-event handling.

### Checkpointing

Stores streaming query progress and state so the application can recover after failure.

| Watermark | Checkpoint |
|---|---|
| Event-time mechanism | Fault-tolerance mechanism |
| Helps manage late data | Helps recover query progress and state |
| Supports state cleanup | Persists recovery information |
| Configured using `withWatermark()` | Configured using `checkpointLocation` |

### Example

```python
df = df.withWatermark(
    "event_time",
    "10 minutes"
)
```

This configures the watermark.

```python
.option(
    "checkpointLocation",
    "/tmp/orders-checkpoint"
)
```

This configures the checkpoint location.

**Remember:** A watermark does not replace checkpointing, and checkpointing does not replace watermarking.

---

## 10. Watermark vs Trigger

A trigger controls when Spark executes streaming processing.

A watermark tracks event-time progress.

### Example trigger

```python
query = (
    result.writeStream
    .format("console")
    .outputMode("append")
    .option(
        "checkpointLocation",
        "/tmp/orders-checkpoint"
    )
    .trigger(processingTime="30 seconds")
    .start()
)
```

This requests micro-batch execution at a 30-second processing-time interval.

It does not guarantee that every batch will finish within 30 seconds.

### Comparison

| Watermark | Trigger |
|---|---|
| Tracks event-time progress | Controls execution scheduling |
| Helps handle late data | Determines when processing runs |
| Supports state cleanup | Controls micro-batch timing |
| Based on event timestamps | Based on execution-time configuration |

---

## 11. What Happens If the Watermark Is Too Small?

Suppose the actual network delay is frequently 15 minutes, but the watermark delay is configured to 5 minutes.

Some events may arrive after the watermark has advanced beyond their windows.

Those events may be dropped from the affected stateful computation.

### Problems

- Missing aggregation records.
- Incorrect revenue totals.
- Incomplete event counts.
- Inaccurate analytics.

### Solution

Analyze actual event-arrival delays and select a suitable watermark threshold.

For example:

| Observed Delay | Possible Watermark Strategy |
|---|---|
| Usually under 2 minutes | A few minutes |
| Usually under 10 minutes | Around 10–15 minutes |
| Frequently 30 minutes | Consider a longer delay or separate late-data handling |

These are illustrative choices, not universal recommendations.

The correct threshold depends on business tolerance, event distribution, state size, and latency requirements.

---

## 12. What Happens If the Watermark Is Too Large?

A large watermark delay allows the engine to retain state for longer.

This can increase memory and state-storage requirements.

### Problems

- Larger state.
- Longer state cleanup.
- Higher checkpoint overhead.
- Increased recovery time.
- More storage usage.

### Example

Suppose the watermark is configured for 24 hours.

The system may need to retain event-time aggregation state much longer than a system using a 10-minute delay.

### Solution

Choose a watermark based on:
- Expected event delay.
- Business tolerance for late data.
- State size.
- Processing latency requirements.
- Infrastructure capacity.

---

## 13. Handling Very Late Events

Not all late events should simply be dropped.

Some business events are too important to lose.

For example:
- Financial transactions.
- Customer payments.
- Orders.
- Insurance claims.
- Critical IoT alerts.

Possible strategies include:

### A. Increase Watermark Delay

Allow more time for delayed events.

Trade-off: More state and potentially delayed final results.

### B. Separate Late-Event Processing

Route or preserve late events for a separate correction pipeline.

### C. Reconciliation

Compare streaming aggregates with source-system totals and correct discrepancies.

### D. Idempotent Upserts

Use a stable business key and an appropriate upsert strategy to correct previously calculated results.

### E. Batch Reprocessing

Reprocess affected time ranges from retained source data.

### Example architecture

```text
               Kafka
                 |
                 v
               Spark
                 |
          Event-Time Processing
                 |
          ┌──────┴──────┐
          v             v
   On-time Results   Late Events
          |             |
          v             v
    Delta Table     Late Data Store
          |             |
          └──────┬──────┘
                 |
                 v
          Reconciliation
                 |
                 v
          Corrected Results
```

The exact implementation depends on whether the late-event path is handled within the streaming query or through a separate ingestion and reconciliation process.

---

## 14. Multiple Watermarks in Spark

A streaming query may combine multiple input streams, each with its own event-time progress.

Spark uses a global watermark policy to determine query-level watermark progress.

By default, Spark uses the `min` policy.

### Minimum policy

The global watermark follows the slowest input stream.

This is safer when slower streams may still deliver relevant events, but it can delay state cleanup.

### Maximum policy

The global watermark follows the fastest input stream.

This can allow faster state cleanup but may cause data from slower streams to be treated as late.

### Configuration

```python
spark.conf.set(
    "spark.sql.streaming.multipleWatermarkPolicy",
    "min"
)
```

The default is generally `min`.

### Comparison

| Policy | Behavior |
|---|---|
| `min` | Uses the slowest input watermark |
| `max` | Uses the fastest input watermark |

**Important:** Choosing `max` is a trade-off. It can cause records from slower input streams to be dropped from stateful processing sooner.

---

## 15. Watermarks in Apache Flink vs Spark

Both frameworks support event-time processing and watermarks, but their implementations differ.

| Feature | Spark Structured Streaming | Apache Flink |
|---|---|---|
| Watermark configuration | `withWatermark()` | `WatermarkStrategy` |
| Event-time processing | Supported | Supported |
| Late-event handling | Depends on operation and watermark | Depends on operator and late-data configuration |
| State cleanup | Watermark-based cleanup for supported operations | Event-time timers and operator-specific state cleanup |
| Execution model | Commonly micro-batch | Native stream-processing runtime |
| Multiple input watermarks | Global watermark policy | Watermark coordination across inputs and operators |

### Spark example

```python
df = df.withWatermark(
    "event_time",
    "10 minutes"
)
```

### Flink example

```java
WatermarkStrategy<Order> strategy =
    WatermarkStrategy
        .<Order>forBoundedOutOfOrderness(
            Duration.ofMinutes(10)
        )
        .withTimestampAssigner(
            (order, timestamp) -> order.eventTime
        );
```

### Important distinction

In Flink, watermarks are part of the stream-processing dataflow and can be generated and propagated by sources and operators.

In Spark Structured Streaming, watermark progress is managed at the query level and used by supported stateful operations.

Do not assume that the two frameworks handle every late event identically.

---

## 16. Practical E-Commerce Scenario

An e-commerce company processes thousands of orders through Kafka and Spark.

The business wants five-minute revenue reports.

### Incoming events

| Order | Event Time | Arrival Time | Amount |
|---|---|---|---:|
| O101 | 10:01 | 10:02 | 500 |
| O102 | 10:03 | 10:04 | 300 |
| O103 | 10:04 | 10:15 | 700 |
| O104 | 10:07 | 10:08 | 400 |

The event O103 arrives much later than its event timestamp.

### Without appropriate late-data handling

The 10:00–10:05 window may be finalized before O103 arrives.

The reported revenue may be:

\[
500 + 300 = 800
\]

But the correct revenue should be:

\[
500 + 300 + 700 = 1500
\]

### With an appropriate watermark strategy

The pipeline can wait for sufficient event-time progress before finalizing the window.

If O103 arrives before the window is finalized, it can be included.

If it arrives after the relevant state has been cleaned up, the pipeline may need separate correction or reconciliation.

### Recommended design

1. Preserve the source event timestamp.
2. Ingest events through Kafka.
3. Parse and validate timestamps.
4. Apply event-time windows.
5. Configure a suitable watermark delay.
6. Monitor late and dropped records.
7. Preserve important source data for replay.
8. Reconcile critical business aggregates.

---

## 17. Monitoring Watermarks and Late Data

A production pipeline should monitor both processing health and data correctness.

Important metrics include:

| Metric | Purpose |
|---|---|
| Maximum observed event time | Understand event-time progress |
| Current watermark | Understand lateness boundary |
| Processing latency | Measure processing delays |
| Input rows per second | Measure ingestion rate |
| Processed rows per second | Measure processing throughput |
| State size | Monitor state growth |
| Checkpoint duration | Monitor recovery overhead |
| Dropped late rows | Identify events excluded by lateness handling |

In Spark, inspect the streaming query's progress information and relevant state-operator metrics.

Example:

```python
progress = query.lastProgress

if progress is not None:
    print(progress)
```

The exact fields available depend on the Spark version and query.

---

## 18. Common Problems and Troubleshooting

| Problem | Possible cause | Solution |
|---|---|---|
| Missing records in window aggregates | Events arrived after the watermark | Review watermark delay and late-data strategy |
| Incorrect revenue totals | Late events excluded from finalized windows | Reconcile with source data |
| State grows continuously | Watermark not advancing or state not eligible for cleanup | Inspect event timestamps and stateful operations |
| Watermark does not advance | Input stream is idle or event-time progress is not advancing | Check source activity and watermark metrics |
| Large checkpoint size | Excessive state retention | Review watermark and aggregation design |
| Different results across streams | Different event-time progress | Review multiple-watermark policy |
| Incorrect window assignment | Timestamp parsing or timezone problem | Validate timestamp format and timezone |
| Late events are not recovered | Checkpoint or replay strategy is incomplete | Design explicit recovery and reprocessing |

---

## 19. Best Practices

1. Preserve event timestamps at the source.
2. Use event time for business-time aggregations.
3. Measure actual event-arrival delays before choosing a watermark.
4. Avoid unnecessarily large watermark thresholds.
5. Avoid thresholds that are too small for normal network delays.
6. Monitor current watermark progress.
7. Monitor late and dropped records.
8. Use checkpoints for recovery.
9. Keep important source data available for replay.
10. Use reconciliation for critical business metrics.
11. Make correction writes idempotent.
12. Test late-event scenarios before production deployment.
13. Understand the behavior of the specific stateful operation.
14. Handle multiple streaming inputs carefully.

---

## 20. Common Interview Questions and Answers

### Q1. What is a late-arriving event?

An event that reaches the processing system later than expected relative to its event timestamp.

### Q2. What is a watermark?

A mechanism that tracks event-time progress and helps manage late events and state cleanup.

### Q3. Why do we need watermarks?

To handle out-of-order events and prevent state from being retained indefinitely in event-time stateful processing.

### Q4. How does Spark calculate a watermark?

Conceptually, Spark uses the maximum observed event time minus the configured delay threshold. Actual progress is managed through streaming execution.

### Q5. Does a watermark use the current system time?

No. It is based on observed event-time progress.

### Q6. What happens when an event arrives after the watermark?

For supported stateful operations, Spark may drop the event from the affected computation because the relevant state may already have been finalized or removed.

### Q7. What happens if the watermark is too small?

Events with normal network delays may arrive too late and be excluded from aggregations.

### Q8. What happens if the watermark is too large?

The engine may retain state longer, increasing storage, memory, and checkpoint overhead.

### Q9. What is the difference between watermark and checkpoint?

Watermark manages event-time progress and state cleanup. Checkpoint stores query progress and state for recovery.

### Q10. What is the difference between watermark and trigger?

Watermark tracks event-time progress. Trigger controls when Spark executes processing.

### Q11. Does watermarking remove duplicates?

No. Watermarking is not general business-key deduplication. Deduplication requires suitable keys and logic.

### Q12. What is the difference between Append and Update mode?

Append emits new result rows, while Update emits rows changed since the previous trigger.

### Q13. What is a global watermark policy?

It determines how Spark combines watermark progress from multiple input streams.

### Q14. What is the default multiple-watermark policy in Spark?

The default is generally `min`, which follows the slowest input watermark.

### Q15. How would you handle financial events arriving very late?

I would preserve the original event timestamp and source data, use an appropriate watermark, monitor late events, and implement replay or reconciliation to correct important aggregates.

### Q16. Are Spark and Flink watermarks identical?

No. Both track event-time progress, but their runtime implementations and late-event handling differ.

---

## 21. Common Mistakes to Avoid

1. **Thinking watermark means waiting exactly ten minutes:** It tracks event-time progress, not a per-event wall-clock timer.
2. **Assuming all late events are accepted:** Events may be dropped by supported stateful operations.
3. **Confusing watermark with checkpoint:** They serve different purposes.
4. **Confusing watermark with trigger:** One manages event time; the other schedules execution.
5. **Assuming watermarking removes duplicates:** Deduplication requires separate logic.
6. **Setting an unnecessarily large watermark:** It can increase state retention and resource use.
7. **Setting an unnecessarily small watermark:** It can exclude normal delayed events.
8. **Ignoring timezone handling:** Incorrect timestamps produce incorrect window assignments.
9. **Assuming Spark and Flink have identical watermark behavior:** Their implementations differ.
10. **Assuming checkpointing can recover every dropped late event:** Checkpoints support query recovery; they do not automatically restore events excluded by watermark logic.

---

## 22. Quick Revision

| Concept | Meaning |
|---|---|
| Late Event | Event arriving after its expected event-time progress |
| Watermark | Tracks event-time progress |
| Watermark Delay | Configured tolerance for out-of-order data |
| Maximum Event Time | Highest observed event timestamp |
| Event-Time Window | Groups records by occurrence time |
| State Cleanup | Removes state no longer needed |
| Checkpoint | Stores recovery progress and state |
| Trigger | Controls processing execution timing |
| Append Mode | Emits newly finalized result rows |
| Update Mode | Emits changed result rows |
| Reconciliation | Corrects results using source data |
| Replay | Reprocesses retained source events |

### Remember these five points

1. Late events arrive after their actual occurrence time.
2. Watermarks track event-time progress.
3. A watermark helps manage late events and state cleanup.
4. Too-small watermarks can exclude valid delayed events; too-large ones can increase state.
5. Critical late events may require replay, correction, or reconciliation.

---

## 23. Self-Assessment

Try answering these questions without looking at the notes.

1. What is a late-arriving event?
2. Why do events arrive out of order?
3. What is a watermark?
4. How does Spark use a watermark?
5. How is watermark progress conceptually calculated?
6. What happens when an event arrives after the watermark?
7. What are the consequences of a watermark that is too small?
8. What are the consequences of a watermark that is too large?
9. Explain the difference between watermark and checkpoint.
10. Explain the difference between watermark and trigger.
11. How does a watermark work with event-time windows?
12. What is the difference between Append and Update output modes?
13. Explain Spark's multiple-watermark policy.
14. How do Spark and Flink differ in watermark implementation?
15. Design a late-event handling strategy for an e-commerce revenue pipeline.

**Interview readiness:** You should be able to explain watermark calculation, late-event behavior, state cleanup, the impact of watermark thresholds, and how to recover important business data that arrives too late.

