# Spark Structured Streaming Fundamentals

## 1. What Is Stream Processing?

Stream processing is the process of continuously processing data as it arrives, instead of waiting for all data to be collected first.

Data is processed in small increments or continuously, depending on the processing engine and execution model.

### Example

Imagine an e-commerce application receiving thousands of orders every minute.

Each order contains:
- Order ID
- Customer ID
- Order amount
- Order timestamp

A streaming application can process incoming orders to:
- Calculate total sales.
- Track orders in real time.
- Identify unusual transactions.
- Monitor sales by location.

**Key idea:** Stream processing handles data continuously as new events arrive.

---

## 2. Batch Processing vs Stream Processing

| Feature | Batch Processing | Stream Processing |
|---|---|---|
| Data | Bounded dataset | Continuously arriving data |
| Processing | Processes collected data | Processes incoming data incrementally |
| Execution | Scheduled or manually triggered | Continuously running query |
| Latency | Usually higher | Usually lower |
| Example | Daily sales report | Live sales monitoring |
| Use case | Historical analysis | Real-time analytics |

### Batch Processing Example

```text
Collect today's orders
        |
        v
Process all orders
        |
        v
Calculate total sales
        |
        v
Generate daily report
```

### Stream Processing Example

```text
New order arrives
        |
        v
Process the order
        |
        v
Update sales result
        |
        v
Continue processing new orders
```

---

## 3. What Is Spark Structured Streaming?

Spark Structured Streaming is a scalable stream-processing engine built on the Spark SQL engine.

It allows developers to process streaming data using familiar DataFrame and SQL operations.

You can use operations such as:

- `select()`
- `filter()`
- `groupBy()`
- `join()`
- Aggregations
- Window functions

The main advantage is that developers can use a similar programming model for both batch and streaming data.

### Example

```python
stream_df = (
    spark.readStream
        .format("json")
        .schema(schema)
        .load("/input/orders")
)
```

Here:
- `readStream` creates a streaming DataFrame.
- `format("json")` specifies the input format.
- `schema` defines the expected structure.
- `load()` specifies the input location.

**Important:** A streaming DataFrame represents an unbounded stream of data. It does not immediately contain all incoming records.

---

## 4. How Structured Streaming Works

Structured Streaming processes incoming data as an unbounded table.

New records are continuously added to the logical input table.

Spark executes incremental computations to produce updated results.

### Basic Architecture

```text
Streaming Source
      |
      v
Read Incoming Data
      |
      v
Streaming DataFrame
      |
      v
Transformations
      |
      v
Aggregations / Joins
      |
      v
Output Mode
      |
      v
Streaming Sink
```

### Main Components

| Component | Meaning |
|---|---|
| Source | Where streaming data comes from |
| Streaming DataFrame | Represents incoming data |
| Transformations | Process and modify records |
| State | Stores information needed across triggers |
| Output Mode | Determines which results are written |
| Sink | Where processed data is written |
| Trigger | Determines when Spark processes available data |
| Checkpoint | Stores progress and recovery information |

---

## 5. Sources in Structured Streaming

A source is the input from which Spark reads streaming data.

Common sources include:

### 1. File Source

Reads files arriving in a directory.

Supported formats include:
- JSON
- CSV
- Parquet
- ORC
- Text

Example:

```python
stream_df = (
    spark.readStream
        .format("json")
        .schema(schema)
        .load("/data/orders")
)
```

### 2. Kafka Source

Reads streaming records from Kafka topics.

Example:

```python
stream_df = (
    spark.readStream
        .format("kafka")
        .option("kafka.bootstrap.servers", "localhost:9092")
        .option("subscribe", "orders")
        .load()
)
```

Kafka records contain fields such as:
- `key`
- `value`
- `topic`
- `partition`
- `offset`
- `timestamp`

The `value` field is commonly converted from binary data into a readable format.

### 3. Rate Source

Generates rows at a specified rate.

It is useful for testing streaming applications without connecting to an external source.

Example:

```python
stream_df = (
    spark.readStream
        .format("rate")
        .option("rowsPerSecond", 10)
        .load()
)
```

This generates approximately 10 rows per second.

**Note:** Available streaming sources depend on the Spark version and installed connectors.

---

## 6. Sinks in Structured Streaming

A sink is the destination where Spark writes processed streaming results.

Common sinks include:

| Sink | Purpose |
|---|---|
| File | Writes output to files |
| Kafka | Sends records to Kafka topics |
| Console | Displays output for testing |
| Memory | Makes results available as an in-memory table |
| Delta | Writes streaming results to Delta tables |

### Example: Console Sink

```python
query = (
    stream_df.writeStream
        .format("console")
        .outputMode("append")
        .start()
)
```

### Example: Delta Sink

```python
query = (
    stream_df.writeStream
        .format("delta")
        .outputMode("append")
        .option("checkpointLocation", "/checkpoints/orders")
        .start("/output/orders")
)
```

**Important:** Sink capabilities and supported output modes depend on the specific sink and query.

---

## 7. Micro-Batch Processing

By default, Spark Structured Streaming uses a micro-batch execution model.

Instead of processing every individual record as a separate execution, Spark groups available records into small batches.

Each batch is processed as a separate execution.

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

Spark may process them like this:

```text
Micro-batch 1: Order 1, Order 2
Micro-batch 2: Order 3, Order 4
Micro-batch 3: Order 5, Order 6
```

The actual number of records per batch depends on data arrival, trigger configuration, and available processing capacity.

### How Micro-Batch Processing Works

1. Spark checks for available input data.
2. It identifies the data to process.
3. It executes the required transformations.
4. It writes the results to the sink.
5. It records progress using checkpoint information.
6. It continues with the next batch.

**Key idea:** Micro-batch processing provides incremental processing using Spark's batch execution engine.

---

## 8. Continuous Processing

Structured Streaming also introduced a continuous processing mode designed for lower-latency processing.

Unlike micro-batch execution, continuous processing aims to process records continuously rather than waiting for micro-batch boundaries.

However, it has important limitations:

- It supports a restricted set of operations.
- It does not support every source and sink.
- It does not provide the same exactly-once guarantees as the standard micro-batch engine.
- It is not suitable for every streaming application.

**Interview point:** Micro-batch is the commonly used execution model in Structured Streaming. Continuous processing is a specialized low-latency option with limitations.

---

## 9. Creating a Streaming DataFrame

A streaming DataFrame is created using `readStream`.

Example:

```python
from pyspark.sql.types import (
    StructType,
    StructField,
    IntegerType,
    DoubleType,
    TimestampType
)

schema = StructType([
    StructField("order_id", IntegerType(), True),
    StructField("amount", DoubleType(), True),
    StructField("event_time", TimestampType(), True)
])

stream_df = (
    spark.readStream
        .format("json")
        .schema(schema)
        .load("/input/orders")
)
```

### Important Points

- Streaming sources generally require a schema to be defined.
- `readStream` creates a streaming DataFrame.
- The DataFrame is not processed until a streaming query is started.
- Transformations are defined before execution.
- Spark processes data incrementally when the query runs.

---

## 10. Transformations in Streaming

Structured Streaming supports many familiar DataFrame transformations.

Example:

```python
from pyspark.sql.functions import col

filtered_df = (
    stream_df
    .filter(col("amount") > 1000)
    .select("order_id", "amount")
)
```

This query:
1. Reads incoming orders.
2. Filters orders with amounts greater than 1000.
3. Selects the required columns.

However, not every batch DataFrame operation is supported on streaming DataFrames.

Some operations require special handling because streaming data is unbounded or because they need to maintain state.

For example:
- Streaming aggregations may require state management.
- Event-time aggregations often use watermarks.
- Some streaming joins have specific restrictions.
- Sorting an unbounded stream is not generally supported without an appropriate bounded or stateful query design.

---

## 11. Starting and Managing a Streaming Query

Transformations alone do not start streaming execution.

A streaming query must be started using `writeStream.start()`.

### Example

```python
query = (
    filtered_df.writeStream
        .format("console")
        .outputMode("append")
        .start()
)
```

### Important Streaming Query Methods

| Method | Purpose |
|---|---|
| `start()` | Starts the streaming query |
| `stop()` | Stops the query |
| `awaitTermination()` | Waits until the query terminates |
| `isActive` | Checks whether the query is active |
| `status` | Returns the current status |
| `lastProgress` | Returns the latest progress information |
| `recentProgress` | Returns recent progress information |

Example:

```python
query = (
    filtered_df.writeStream
        .format("console")
        .outputMode("append")
        .start()
)

query.awaitTermination()
```

**Important:** `awaitTermination()` does not start the query. The query must first be started using `start()`.

---

## 12. Stateful vs Stateless Processing

This is an important concept in streaming interviews.

### Stateless Processing

Stateless processing handles each record independently without maintaining information across records or micro-batches.

Examples:
- Selecting columns.
- Filtering records.
- Converting data types.
- Simple record-level transformations.

Example:

```python
result = stream_df.filter(col("amount") > 1000)
```

### Stateful Processing

Stateful processing maintains information across records or micro-batches to calculate results over time.

Examples:
- Running totals.
- Aggregations.
- Deduplication.
- Stream-stream joins.
- Event-time window calculations.

Example:

```python
from pyspark.sql.functions import sum

result = (
    stream_df
    .groupBy("country")
    .agg(sum("amount").alias("total_sales"))
)
```

Spark must maintain aggregation state as new records arrive.

### Comparison

| Feature | Stateless | Stateful |
|---|---|---|
| Maintains state | No | Yes |
| Depends on previous data | No | Yes |
| Complexity | Generally simpler | Generally more complex |
| Examples | Filter, select | Aggregation, deduplication |
| Checkpoint importance | Progress recovery | Progress and state recovery |

---

## 13. Checkpointing in Streaming

Checkpointing stores information required to recover a streaming query after failure.

It can include:
- Source progress and offsets.
- State information for stateful operations.
- Query recovery metadata.

Example:

```python
query = (
    stream_df.writeStream
        .format("delta")
        .outputMode("append")
        .option("checkpointLocation", "/checkpoints/orders")
        .start("/output/orders")
)
```

### Why Is Checkpointing Important?

Imagine a streaming application processes data for several hours and then fails.

Without the required recovery information, Spark may not know where to resume or how to restore state.

Checkpointing helps Spark recover the query and continue from its recorded progress.

**Important:** Checkpointing alone does not guarantee exactly-once results for every external sink. End-to-end guarantees depend on the source, processing logic, and sink behavior.

---

## 14. Triggers in Structured Streaming

A trigger determines when Spark starts processing available data.

Common trigger types include:

| Trigger | Meaning |
|---|---|
| Default trigger | Processes available data in successive micro-batches |
| Processing-time trigger | Starts processing at configured time intervals |
| Available-now trigger | Processes data available when the query runs, potentially across multiple micro-batches, then stops |
| Once trigger | Processes available data in one micro-batch and then stops; deprecated in newer Spark versions |

Example:

```python
query = (
    stream_df.writeStream
        .format("console")
        .trigger(processingTime="30 seconds")
        .start()
)
```

Here, Spark attempts to start a processing cycle every 30 seconds.

**Remember:** Trigger controls when data is processed, while output mode controls which results are written.

---

## 15. Common Mistakes

1. Confusing batch processing with stream processing.
2. Thinking `readStream` immediately starts execution.
3. Forgetting to call `writeStream.start()`.
4. Assuming all batch DataFrame operations work on streaming DataFrames.
5. Confusing a source with a sink.
6. Assuming Structured Streaming always processes individual records immediately.
7. Confusing micro-batch execution with continuous processing.
8. Assuming every streaming operation is stateless.
9. Ignoring checkpoint configuration.
10. Assuming checkpointing alone guarantees exactly-once delivery.
11. Confusing output modes with triggers.
12. Assuming every sink supports all output modes.

---

## 16. Interview Questions

**Q1. What is stream processing?**

Stream processing continuously processes data as it arrives instead of waiting for a complete dataset.

**Q2. What is Spark Structured Streaming?**

It is Spark's scalable stream-processing engine that uses the DataFrame and SQL programming model.

**Q3. What is the difference between batch and stream processing?**

Batch processing handles bounded collected data, while stream processing handles continuously arriving data.

**Q4. What is a streaming DataFrame?**

A streaming DataFrame represents an unbounded dataset that receives new records over time.

**Q5. What is a source?**

A source is the input from which Spark reads streaming data, such as Kafka or files.

**Q6. What is a sink?**

A sink is the destination where Spark writes processed streaming results.

**Q7. What is micro-batch processing?**

Spark groups available records into small batches and processes each batch incrementally.

**Q8. Does Structured Streaming support continuous processing?**

Yes, but continuous processing has limited operation and connector support compared with the standard micro-batch engine.

**Q9. What is the difference between stateless and stateful processing?**

Stateless processing handles records independently. Stateful processing maintains information across records or batches.

**Q10. Why is checkpointing important?**

It stores progress and recovery information so a streaming query can recover after failure.

**Q11. What is a trigger?**

A trigger determines when Spark processes available streaming data.

**Q12. How do you start a streaming query?**

Using `writeStream.start()`.

**Q13. What is the difference between `read` and `readStream`?**

`read` creates a batch DataFrame from bounded input, while `readStream` creates a streaming DataFrame from a streaming source.

**Q14. Can we use `groupBy()` in Structured Streaming?**

Yes. Streaming aggregations are supported, but they generally require state management and may need watermarks depending on the query.

**Q15. Is Structured Streaming exactly-once?**

The standard micro-batch engine provides fault-tolerance mechanisms, but end-to-end exactly-once results depend on the source, checkpointing, processing logic, and sink.

---

## 17. Quick Revision

| Concept | Remember |
|---|---|
| Stream processing | Processes incoming data continuously |
| Structured Streaming | Spark's stream-processing engine |
| `readStream` | Reads streaming data |
| Source | Input system |
| Sink | Output destination |
| Micro-batch | Processes data in small batches |
| Continuous processing | Specialized low-latency mode |
| Streaming DataFrame | Represents unbounded data |
| Stateless | Does not maintain state across records |
| Stateful | Maintains information across processing |
| Checkpoint | Stores recovery information |
| Trigger | Controls processing schedule |
| Output mode | Controls which results are written |
| `writeStream.start()` | Starts streaming execution |

### One-line revision

- Source → Where data comes from.
- Sink → Where data goes.
- `readStream` → Reads streaming data.
- `writeStream` → Defines streaming output.
- Micro-batch → Processes small batches incrementally.
- Stateful → Maintains information across batches.
- Checkpoint → Supports recovery.
- Trigger → Controls when processing happens.
- Output mode → Controls what gets written.

---

## 18. Self-Check

- [ ] Can I explain batch vs stream processing?
- [ ] Can I explain Spark Structured Streaming?
- [ ] Can I describe the streaming architecture?
- [ ] Can I differentiate sources and sinks?
- [ ] Can I explain micro-batch processing?
- [ ] Can I explain continuous processing?
- [ ] Can I create a streaming DataFrame?
- [ ] Can I explain stateless vs stateful processing?
- [ ] Can I explain checkpointing?
- [ ] Can I differentiate triggers and output modes?
- [ ] Can I start and manage a streaming query?

**Final takeaway:** Spark Structured Streaming processes continuously arriving data using familiar DataFrame operations. Understand its sources, sinks, micro-batch execution, state management, checkpointing, triggers, and output modes before moving to advanced streaming topics.