# Spark Structured Streaming

## 1. What Is Structured Streaming?

Spark Structured Streaming is a scalable stream-processing engine built on the Spark SQL engine.

It allows us to process continuously arriving data using familiar DataFrame and SQL operations.

Instead of treating streaming data as individual records, Structured Streaming represents it as an **unbounded table** that continuously receives new rows.

### Example

Consider an e-commerce application receiving orders continuously.

| Order ID | Customer | Amount |
|---|---|---:|
| 101 | A | 500 |
| 102 | B | 700 |
| 103 | A | 300 |

New orders keep arriving. Structured Streaming processes these records incrementally and updates the results.

Typical use cases:
- Real-time sales analytics.
- Transaction monitoring.
- Log processing.
- Fraud detection.
- Real-time data integration.
- Event-driven applications.

**Key idea:** Structured Streaming applies Spark SQL's structured processing model to continuously arriving data.

---

## 2. Core Concept: Streaming as an Unbounded Table

Structured Streaming treats incoming data as an unbounded table.

An unbounded table has no fixed final number of rows because new records can continue arriving.

Imagine a table receiving records over time:

```text
Time 1:
Order 101
Order 102

Time 2:
Order 103
Order 104

Time 3:
Order 105
Order 106
```

Conceptually, the table keeps growing:

```text
Orders Table
----------------
Order 101
Order 102
Order 103
Order 104
Order 105
Order 106
...
```

Spark processes the new records incrementally instead of repeatedly processing the entire input table.

### Why Is This Important?

It allows developers to use familiar operations such as:

- `select()`
- `filter()`
- `groupBy()`
- `join()`
- Aggregations
- Window-based operations

The programming model is similar to batch DataFrame processing, but execution and supported operations have streaming-specific rules.

---

## 3. Structured Streaming Architecture

A typical Structured Streaming application contains the following components:

```text
Streaming Source
      |
      v
ReadStream
      |
      v
Streaming DataFrame
      |
      v
Transformations
      |
      v
Logical Plan
      |
      v
Physical Execution
      |
      v
Output Mode
      |
      v
Streaming Sink
```

### Main Components

| Component | Responsibility |
|---|---|
| Source | Provides incoming data |
| Streaming DataFrame | Represents unbounded input |
| Transformations | Define processing logic |
| Logical plan | Represents the query logically |
| Physical plan | Describes how Spark executes the query |
| State store | Maintains information for stateful operations |
| Checkpoint | Stores progress and recovery information |
| Sink | Receives processed results |

The exact internal execution details depend on the query and Spark version.

---

## 4. Creating a Streaming DataFrame

Use `spark.readStream` to create a streaming DataFrame.

Example using JSON files:

```python
from pyspark.sql.types import (
    StructType,
    StructField,
    IntegerType,
    DoubleType,
    StringType
)

schema = StructType([
    StructField("order_id", IntegerType(), True),
    StructField("customer", StringType(), True),
    StructField("amount", DoubleType(), True)
])

orders_stream = (
    spark.readStream
        .format("json")
        .schema(schema)
        .load("/data/orders")
)
```

### Explanation

- `readStream` creates a streaming read.
- `format("json")` specifies the input format.
- `schema()` defines the expected columns and data types.
- `load()` specifies the input directory.

Check whether the DataFrame is streaming:

```python
orders_stream.isStreaming
```

Expected result:

```text
True
```

**Important:** Creating a streaming DataFrame does not start the streaming application. It only defines the input and query structure.

---

## 5. Transformations on Streaming Data

Once the streaming DataFrame is created, we can apply DataFrame transformations.

Example:

```python
from pyspark.sql.functions import col

filtered_orders = (
    orders_stream
    .filter(col("amount") > 500)
    .select("order_id", "customer", "amount")
)
```

This query:
1. Reads incoming order records.
2. Filters orders with amounts greater than 500.
3. Selects the required columns.

Transformations define what Spark should do. The streaming query starts when we call `writeStream.start()`.

**Important:** Not every batch operation is supported on streaming DataFrames. Operations such as sorting an unbounded stream generally need a supported bounded or stateful design.

---

## 6. Starting a Streaming Query

Use `writeStream` to define the output and `start()` to begin execution.

Example:

```python
query = (
    filtered_orders.writeStream
        .format("console")
        .outputMode("append")
        .start()
)
```

### Explanation

| Method | Purpose |
|---|---|
| `writeStream` | Defines streaming output |
| `format()` | Specifies the sink |
| `outputMode()` | Defines which result rows are written |
| `start()` | Starts the streaming query |

Wait for the query to terminate:

```python
query.awaitTermination()
```

A typical streaming query keeps running until it is stopped, fails, or completes under a finite trigger such as AvailableNow.

---

## 7. Micro-Batch Execution Model

By default, Structured Streaming uses micro-batch execution.

Spark collects available input records into a small batch and processes that batch as a separate execution.

### Example

Suppose records arrive continuously:

```text
Order 1
Order 2
Order 3
Order 4
Order 5
Order 6
```

Spark may process them as:

```text
Micro-batch 1:
Order 1, Order 2

Micro-batch 2:
Order 3, Order 4

Micro-batch 3:
Order 5, Order 6
```

The number of records in each batch depends on data arrival, trigger configuration, and processing capacity.

### Execution Steps

1. Spark identifies available input data.
2. It determines the input range for the next batch.
3. It executes the query plan.
4. It updates state if the query requires stateful processing.
5. It writes the results to the sink.
6. It records progress and recovery information.
7. It proceeds to the next batch.

**Interview point:** Micro-batch execution allows Spark to reuse its distributed batch-processing engine for incremental stream processing.

---

## 8. Continuous Processing Mode

Structured Streaming also provides a continuous processing mode designed for lower-latency workloads.

It differs from micro-batch execution because it aims to process records continuously rather than waiting for micro-batch boundaries.

However, it has significant limitations:

- Restricted operation support.
- Limited source and sink compatibility.
- Different fault-tolerance guarantees from standard micro-batch execution.
- Not suitable for every streaming workload.

**Remember:** Micro-batch is the commonly used execution model. Continuous processing is a specialized option with limitations.

---

## 9. Stateful and Stateless Operations

This is one of the most important Structured Streaming concepts.

### Stateless Operations

Stateless operations process each record independently without maintaining information across batches.

Examples:

- `select()`
- `filter()`
- Simple column transformations.
- Data type conversions.

Example:

```python
result = orders_stream.filter(col("amount") > 1000)
```

Each record is evaluated independently.

### Stateful Operations

Stateful operations maintain information across records or micro-batches.

Examples:

- Aggregations.
- Deduplication.
- Stream-stream joins.
- Event-time window calculations.

Example:

```python
from pyspark.sql.functions import sum

sales_by_customer = (
    orders_stream
    .groupBy("customer")
    .agg(sum("amount").alias("total_sales"))
)
```

Spark maintains the running aggregation state as new orders arrive.

### Comparison

| Feature | Stateless | Stateful |
|---|---|---|
| Maintains state | No | Yes |
| Uses previous processing information | No | Yes |
| Complexity | Generally simpler | Generally more complex |
| Example | Filter | Aggregation |
| State recovery | Not required for the operation itself | Important for recovery |

---

## 10. State Store

A state store maintains intermediate information required by stateful streaming operations.

For example, consider calculating total sales by customer.

Incoming records:

| Batch | Customer | Amount |
|---|---|---:|
| 1 | A | 500 |
| 1 | B | 300 |
| 2 | A | 200 |
| 3 | B | 100 |

The state must retain the current totals.

After Batch 1:

| Customer | Total |
|---|---:|
| A | 500 |
| B | 300 |

After Batch 2:

| Customer | Total |
|---|---:|
| A | 700 |
| B | 300 |

After Batch 3:

| Customer | Total |
|---|---:|
| A | 700 |
| B | 400 |

Spark updates the state as new records arrive.

### Why Is State Store Needed?

Without maintaining state, Spark would not know the previous totals needed to calculate the updated aggregates.

Stateful queries can consume memory and storage resources, so state size and cleanup are important performance considerations.

---

## 11. Event Time and Processing Time

Structured Streaming supports different notions of time.

### Event Time

Event time is the timestamp associated with the actual event.

Example:

An order is placed at 10:05 AM, but it reaches Spark at 10:08 AM.

Its event time is 10:05 AM.

### Processing Time

Processing time is when Spark processes the event.

For the same order, processing time might be 10:08 AM.

### Comparison

| Feature | Event Time | Processing Time |
|---|---|---|
| Based on | Event timestamp | Spark processing clock |
| Represents | When the event happened | When Spark processed it |
| Useful for | Accurate time-based analytics | Processing-time-based operations |

**Interview point:** Event time is important when records may arrive late or out of order.

---

## 12. Watermarks

A watermark helps Spark manage late-arriving event-time data and control the amount of state retained by certain stateful operations.

Example:

```python
from pyspark.sql.functions import window, sum

result = (
    orders_stream
    .withWatermark("event_time", "10 minutes")
    .groupBy(
        window("event_time", "5 minutes"),
        "customer"
    )
    .agg(sum("amount").alias("total_sales"))
)
```

### Explanation

- The aggregation window is 5 minutes.
- The watermark delay threshold is 10 minutes.
- Spark uses the watermark to determine when older event-time results can be finalized and when relevant state can be cleaned up.

### Why Are Watermarks Needed?

Imagine Spark receives an event with an event time of 10:00 AM after already processing events from 10:15 AM.

The event is late relative to the other data.

Watermarks help Spark decide how long to retain information for late events.

**Important:**

- A watermark is not a guarantee that no later data will arrive.
- Records later than the watermark threshold may be dropped from the relevant stateful operation.
- Watermarks do not automatically make every streaming query faster.
- Their behavior depends on the query and stateful operation.

---

## 13. Checkpointing and Fault Tolerance

Checkpointing stores the information required to recover a streaming query after failure.

It may include:

- Source progress and offsets.
- Query recovery metadata.
- Stateful operator information.

Example:

```python
query = (
    orders_stream.writeStream
        .format("delta")
        .outputMode("append")
        .option(
            "checkpointLocation",
            "/checkpoints/orders"
        )
        .start("/output/orders")
)
```

### Why Is Checkpointing Important?

Suppose a streaming application processes data for several hours and then fails.

Checkpointing allows Spark to recover the recorded query progress and restore state where required.

### Important Rules

1. Use a durable checkpoint location.
2. Do not casually share one checkpoint directory between independent queries.
3. Keep the checkpoint location stable when restarting the same query.
4. Changes to query structure or source configuration may make an existing checkpoint incompatible.
5. Checkpointing alone does not guarantee exactly-once results for arbitrary external sinks.

---

## 14. Output Modes

Output modes determine which rows are written to the sink.

Structured Streaming supports three primary output modes:

| Mode | Meaning |
|---|---|
| Append | Writes newly added result rows |
| Update | Writes result rows changed since the previous trigger |
| Complete | Writes the entire result table |

### Example

```python
query = (
    sales_by_customer.writeStream
        .format("console")
        .outputMode("update")
        .start()
)
```

### Remember

- Append → New rows.
- Update → Changed rows.
- Complete → Entire result.

The supported output mode depends on the query and sink.

See `output-modes.md` for detailed examples.

---

## 15. Triggers

A trigger determines when Spark processes available data.

Common trigger types:

| Trigger | Meaning |
|---|---|
| Default | Processes data in successive micro-batches |
| Processing Time | Processes at configured time intervals |
| AvailableNow | Processes available data, potentially across multiple micro-batches, then stops |
| Once | Processes available data in one micro-batch and then stops; deprecated in newer Spark versions |

Example:

```python
query = (
    orders_stream.writeStream
        .format("console")
        .outputMode("append")
        .trigger(processingTime="30 seconds")
        .start()
)
```

Here, Spark attempts to start processing every 30 seconds.

**Difference:**
- Trigger controls when processing happens.
- Output mode controls which results are written.

---

## 16. Monitoring a Streaming Query

Spark provides methods to inspect a running streaming query.

```python
query.isActive
```

Checks whether the query is active.

```python
query.status
```

Returns the current query status.

```python
query.lastProgress
```

Returns the latest progress information.

```python
query.recentProgress
```

Returns recent progress information.

```python
query.explain()
```

Displays the query execution plan.

### Why Is Monitoring Important?

It helps identify:

- Processing delays.
- Slow batches.
- Increasing input rates.
- State growth.
- Query failures.
- Backlogs.

The Spark UI also provides information about jobs, stages, tasks, and streaming execution.

---

## 17. Complete Example: Order Streaming Pipeline

This example demonstrates a basic end-to-end streaming pipeline using JSON files.

### Step 1: Define the Schema

```python
from pyspark.sql.types import (
    StructType,
    StructField,
    IntegerType,
    DoubleType,
    StringType
)

schema = StructType([
    StructField("order_id", IntegerType(), True),
    StructField("customer", StringType(), True),
    StructField("amount", DoubleType(), True)
])
```

### Step 2: Read Streaming Data

```python
orders_stream = (
    spark.readStream
        .format("json")
        .schema(schema)
        .load("/data/orders")
)
```

### Step 3: Transform Data

```python
from pyspark.sql.functions import col

processed_orders = (
    orders_stream
    .filter(col("amount") > 0)
    .select(
        "order_id",
        "customer",
        "amount"
    )
)
```

### Step 4: Write to Delta

```python
query = (
    processed_orders.writeStream
        .format("delta")
        .outputMode("append")
        .option(
            "checkpointLocation",
            "/checkpoints/orders"
        )
        .start("/output/orders")
)
```

### Step 5: Wait for Termination

```python
query.awaitTermination()
```

### Pipeline Flow

```text
JSON Files
    |
    v
readStream
    |
    v
Streaming DataFrame
    |
    v
Filter Invalid Orders
    |
    v
Select Required Columns
    |
    v
writeStream
    |
    v
Delta Table
```

This is a basic streaming ingestion and transformation pipeline. Production implementations should also consider schema evolution, malformed records, monitoring, checkpoint recovery, data quality, and sink behavior.

---

## 18. Common Mistakes

1. Thinking `readStream` starts the query.
2. Forgetting `writeStream.start()`.
3. Treating a streaming DataFrame as a normal bounded DataFrame.
4. Assuming all batch operations work in streaming.
5. Confusing event time with processing time.
6. Ignoring late-arriving data.
7. Using stateful aggregations without understanding state management.
8. Forgetting checkpoint configuration.
9. Assuming checkpointing alone guarantees exactly-once processing.
10. Confusing output modes with triggers.
11. Assuming continuous processing supports every operation.
12. Ignoring streaming query monitoring.

---

## 19. Interview Questions

**Q1. What is Spark Structured Streaming?**

It is Spark's stream-processing engine built on the Spark SQL engine, allowing continuous data processing using DataFrame and SQL operations.

**Q2. How does Structured Streaming represent streaming data?**

As an unbounded table that continuously receives new records.

**Q3. What is the default execution model?**

Micro-batch execution.

**Q4. What happens during a micro-batch?**

Spark identifies available input data, processes it through the query plan, updates state when necessary, writes results, and records progress.

**Q5. What is the difference between stateful and stateless operations?**

Stateless operations process records independently. Stateful operations maintain information across records or batches.

**Q6. What is a state store?**

It maintains intermediate state needed by stateful operations such as aggregations and stream-stream joins.

**Q7. What is event time?**

The timestamp associated with when an event actually occurred.

**Q8. What is processing time?**

The time when Spark processes an event.

**Q9. Why are watermarks used?**

They help manage late-arriving event-time data and control state retention in supported stateful operations.

**Q10. Why is checkpointing important?**

It stores progress and recovery information so Spark can recover a streaming query after failure.

**Q11. What is the difference between output mode and trigger?**

Output mode determines which result rows are written. Trigger determines when Spark processes available data.

**Q12. Can we use normal DataFrame transformations in Structured Streaming?**

Many DataFrame transformations are supported, but some operations have restrictions because the input is unbounded or requires state management.

**Q13. Does Structured Streaming support exactly-once processing?**

The standard micro-batch engine provides fault-tolerance mechanisms, but end-to-end exactly-once results depend on the source, checkpointing, processing logic, and sink.

**Q14. How can you monitor a streaming query?**

Using query methods such as `lastProgress`, `recentProgress`, `status`, and the Spark UI.

**Q15. What is the difference between micro-batch and continuous processing?**

Micro-batch processes available records in small batches. Continuous processing aims for lower latency by processing records continuously but supports fewer operations and has different guarantees.

---

## 20. Quick Revision

| Concept | Remember |
|---|---|
| Structured Streaming | Spark SQL-based streaming engine |
| Unbounded table | Continuously growing logical input |
| `readStream` | Creates streaming DataFrame |
| `writeStream` | Defines streaming output |
| Micro-batch | Default execution model |
| Continuous mode | Specialized low-latency mode |
| Stateful operation | Maintains information across batches |
| Stateless operation | Processes records independently |
| State store | Maintains state for stateful operations |
| Event time | When event actually happened |
| Processing time | When Spark processed the event |
| Watermark | Manages late data and state cleanup |
| Checkpoint | Supports query recovery |
| Output mode | Controls which rows are written |
| Trigger | Controls when processing happens |

### One-Line Revision

- Structured Streaming → Processes continuously arriving data.
- Unbounded table → Represents an ongoing stream.
- Micro-batch → Processes data incrementally.
- State store → Maintains intermediate state.
- Event time → Actual event timestamp.
- Watermark → Handles late data.
- Checkpoint → Supports recovery.
- Trigger → Controls processing schedule.
- Output mode → Controls output rows.

---

## 21. Self-Check

- [ ] Can I explain Structured Streaming architecture?
- [ ] Can I explain the unbounded table concept?
- [ ] Can I create a streaming DataFrame?
- [ ] Can I start and manage a streaming query?
- [ ] Can I explain micro-batch execution?
- [ ] Can I differentiate stateful and stateless operations?
- [ ] Can I explain the state store?
- [ ] Can I differentiate event time and processing time?
- [ ] Can I explain watermarks?
- [ ] Can I explain checkpointing and fault tolerance?
- [ ] Can I differentiate output modes and triggers?
- [ ] Can I build a basic end-to-end streaming pipeline?

**Final takeaway:** Structured Streaming uses Spark's DataFrame and SQL engine to process unbounded data incrementally. For interviews, focus on micro-batch execution, state management, event time, watermarks, checkpointing, output modes, and triggers.