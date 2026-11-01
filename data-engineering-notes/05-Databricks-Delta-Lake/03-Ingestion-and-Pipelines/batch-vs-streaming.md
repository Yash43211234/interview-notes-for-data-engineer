# Batch Processing vs Streaming Processing

## 1. What Is Data Processing?

Data processing is the process of collecting, transforming, and delivering data so it can be used for analytics, reporting, and business operations.

In data engineering, two common processing approaches are:

1. **Batch Processing:** Processes data in groups at scheduled intervals.
2. **Stream Processing:** Processes incoming data continuously or in small increments as it arrives.

The main difference is how frequently data is processed and how quickly results become available.

## 2. Batch Processing

Batch processing collects data over a period of time and processes it together as a group.

Instead of processing every record immediately, the system waits until a scheduled time or until a batch is ready.

**Example:**

An e-commerce company receives 100,000 orders throughout the day.

Instead of processing each order immediately, it processes all orders at midnight to generate daily sales reports.

### Batch Processing Flow

```text
Source Systems
      |
      v
Data Collection
      |
      v
Storage (ADLS / S3)
      |
      v
Scheduled Batch Job
      |
      v
Data Transformation
      |
      v
Data Warehouse / Delta Tables
      |
      v
Reports and Analytics
```

### Example Using PySpark

Suppose we have an orders CSV file.

```python
from pyspark.sql import functions as F

df = spark.read.csv(
    "/data/orders/",
    header=True,
    inferSchema=True
)

daily_sales = (
    df.groupBy("order_date")
      .agg(
          F.sum("amount").alias("total_sales"),
          F.count("*").alias("total_orders")
      )
)

daily_sales.write \
    .format("delta") \
    .mode("overwrite") \
    .save("/data/gold/daily_sales")
```

This example reads a batch of files, calculates daily sales, and writes the result to a Delta location.

### Characteristics of Batch Processing

- Processes a collection of records together.
- Usually runs on a schedule or when a batch is available.
- Suitable for large-volume historical processing.
- Results are available after the batch completes.
- Often easier to implement and debug than continuously running streaming pipelines.

### Common Use Cases

- Daily sales reports.
- Monthly financial reports.
- Payroll processing.
- Historical data migration.
- Daily ETL and ELT pipelines.
- Data warehouse loading.

## 3. Streaming Processing

Streaming processing handles data as it arrives, usually in small increments.

Instead of waiting for a large batch, the system processes incoming data continuously or through frequent micro-batches.

**Example:**

An online payment platform receives transactions throughout the day.

The system processes incoming transactions to detect suspicious activity and update transaction metrics with low latency.

### Streaming Processing Flow

```text
Live Data Sources
      |
      v
Kafka / Event Hubs
      |
      v
Streaming Engine
      |
      v
Real-Time Transformations
      |
      v
Delta Lake / Other Sink
      |
      v
Dashboards / Applications
```

### Example Using PySpark Structured Streaming

Suppose payment events arrive in a Kafka-like or file-based streaming source.

```python
from pyspark.sql import functions as F

df = (
    spark.readStream
    .format("json")
    .schema(payment_schema)
    .load("/data/incoming/payments/")
)

processed_df = (
    df.filter(F.col("amount") > 0)
      .withColumn("processed_time", F.current_timestamp())
)

query = (
    processed_df.writeStream
    .format("delta")
    .outputMode("append")
    .option("checkpointLocation", "/data/checkpoints/payments/")
    .start("/data/bronze/payments/")
)
```

This example continuously discovers and processes incoming JSON files using Structured Streaming.

**Important:** The source shown here is a file stream, not Kafka. Structured Streaming supports multiple source types, including files and Kafka.

### Characteristics of Streaming Processing

- Processes incoming data continuously or in small increments.
- Provides lower-latency results than traditional scheduled batch jobs.
- Requires checkpointing and recovery considerations.
- Must handle late-arriving and out-of-order events where relevant.
- Often requires state management for aggregations and deduplication.

### Common Use Cases

- Fraud detection.
- Real-time payment monitoring.
- IoT sensor monitoring.
- Application log processing.
- Live dashboards.
- Clickstream analysis.
- Real-time notifications.

## 4. Batch vs Streaming: Key Differences

| Feature | Batch Processing | Streaming Processing |
|---|---|---|
| Processing | Processes data in groups | Processes incoming data continuously or in increments |
| Latency | Usually minutes to hours or longer | Usually seconds or minutes, depending on design |
| Data | Bounded datasets | Unbounded data over time |
| Execution | Scheduled or triggered jobs | Long-running or repeatedly triggered queries |
| Complexity | Generally simpler | Generally more complex |
| Checkpointing | Depends on the pipeline design | Important for streaming recovery |
| Late data | Handled during batch processing or corrections | Requires event-time and late-data handling where relevant |
| Use case | Daily reports | Live transaction monitoring |
| Cost | Can be efficient for scheduled workloads | Depends on throughput, latency, and runtime requirements |

**Remember:** Streaming does not always mean millisecond processing. Latency depends on the source, trigger, processing logic, compute resources, and sink.

## 5. Micro-Batch Processing

Micro-batch processing divides incoming data into small batches and processes each batch separately.

Apache Spark Structured Streaming commonly uses micro-batch execution.

**Example:**

Instead of processing one million records together every hour, the system processes newly available records in smaller batches every few seconds.

```text
Incoming Events
      |
      v
Micro-Batch 1
      |
      v
Micro-Batch 2
      |
      v
Micro-Batch 3
      |
      v
Output Sink
```

### Example

```python
query = (
    df.writeStream
    .format("delta")
    .option("checkpointLocation", checkpoint_path)
    .trigger(processingTime="30 seconds")
    .start(target_path)
)
```

This requests processing at a 30-second interval.

**Important:** A 30-second trigger is not a guarantee that every batch finishes within 30 seconds. If processing takes longer, the next batch may be delayed.

### Micro-Batch vs Continuous Processing

| Micro-Batch | Continuous Processing |
|---|---|
| Processes data in small batches | Processes records with a lower-latency continuous execution model |
| Common Structured Streaming execution model | Specialized mode with limitations |
| Supports a broad range of streaming operations | Supports a more limited set of operations and sources/sinks |
| Commonly used in production pipelines | Used only when supported and suitable |

For most Databricks Structured Streaming pipelines, micro-batch processing is the common execution model.

## 6. Event Time vs Processing Time

This is an important streaming concept.

### Event Time

Event time is the time when an event actually occurred at the source.

Example:

A customer makes a payment at 10:00 AM.

```text
event_time = 10:00 AM
```

### Processing Time

Processing time is the time when the streaming system processes the event.

Example:

The payment reaches the processing system at 10:03 AM.

```text
processing_time = 10:03 AM
```

### Difference

| Event Time | Processing Time |
|---|---|
| When the event happened | When the system processed it |
| Comes from the event or source data | Comes from the processing system |
| Useful for business-time analysis | Useful for measuring processing behavior |

**Example:** A mobile application may send an event late because of poor network connectivity. The event occurred at 10:00 AM but reached the processing system at 10:05 AM.

For accurate business-time aggregations, event time is often more meaningful than processing time.

## 7. Late-Arriving Data

Late-arriving data is data that reaches the processing system after its actual event time.

Example:

| Transaction | Event Time | Arrival Time |
|---|---|---|
| T1 | 10:00 AM | 10:01 AM |
| T2 | 10:02 AM | 10:03 AM |
| T3 | 10:01 AM | 10:10 AM |

Transaction T3 arrived late even though it occurred before T2.

This can affect streaming aggregations and window calculations.

### How Do We Handle Late Data?

Structured Streaming provides **watermarks** to manage state for event-time operations.

Example:

```python
from pyspark.sql import functions as F

result = (
    df.withWatermark("event_time", "10 minutes")
      .groupBy(
          F.window("event_time", "5 minutes")
      )
      .agg(
          F.count("*").alias("total_events")
      )
)
```

Here:

- `event_time` is the event-time column.
- `5 minutes` defines the aggregation window.
- `10 minutes` defines the watermark delay threshold.

A watermark helps Spark determine when it can finalize event-time windows and remove old aggregation state.

**Important:** A watermark is not a guarantee that every event arriving within exactly 10 minutes will always be accepted, nor does it mean all older events are automatically dropped in every streaming operation. Its behavior depends on the stateful operation and watermark progress.

## 8. Windowing in Streaming

Windowing groups streaming events based on time.

It is useful when calculating metrics over fixed or moving time periods.

### A. Tumbling Window

A tumbling window has a fixed duration, and windows do not overlap.

Example: 5-minute windows.

```text
10:00 - 10:05
10:05 - 10:10
10:10 - 10:15
```

### B. Sliding Window

A sliding window moves at a defined interval and can overlap.

Example: 10-minute window sliding every 5 minutes.

```text
10:00 - 10:10
10:05 - 10:15
10:10 - 10:20
```

### C. Session Window

A session window groups events based on periods of activity separated by a defined inactivity gap.

Example:

A user performs several actions. If there is no activity for 30 minutes, the session may be considered complete.

| Window Type | Meaning |
|---|---|
| Tumbling | Fixed duration, no overlap |
| Sliding | Fixed duration, overlapping windows |
| Session | Groups activity separated by inactivity gaps |

## 9. Batch and Streaming in Databricks

Databricks supports both batch and streaming workloads using Apache Spark.

### Batch Processing in Databricks

Common tools:

- Databricks notebooks.
- Databricks Jobs.
- PySpark DataFrames.
- SQL.
- Delta Lake.

Example:

```python
df = spark.read.format("delta").load(source_path)

transformed_df = df.filter("amount > 0")

transformed_df.write \
    .format("delta") \
    .mode("append") \
    .save(target_path)
```

### Streaming Processing in Databricks

Common tools:

- Structured Streaming.
- Auto Loader.
- Kafka.
- Delta Lake.
- Databricks Jobs.

Example:

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
    .toTable("bronze.events")
)
```

This uses Auto Loader to incrementally ingest files. With `AvailableNow`, it processes the data available when the run starts and then stops.

## 10. Auto Loader: Batch or Streaming?

Auto Loader is built on Structured Streaming, but it can be used for both continuously running and scheduled ingestion.

### Continuous Ingestion

```python
query = (
    df.writeStream
    .format("delta")
    .option("checkpointLocation", checkpoint_path)
    .start(target_path)
)
```

The query remains active and processes new files as they are discovered.

### Scheduled Incremental Ingestion

```python
query = (
    df.writeStream
    .format("delta")
    .option("checkpointLocation", checkpoint_path)
    .trigger(availableNow=True)
    .start(target_path)
)
```

The query processes available data and stops. A Databricks Job can run it again on a schedule.

**Interview point:** Auto Loader does not have to run continuously. It can support scheduled incremental ingestion through triggers such as `AvailableNow`.

## 11. Exactly-Once Processing and Checkpointing

Streaming pipelines need to recover safely after failures.

Structured Streaming uses checkpointing to store information such as:

- Query progress.
- Source offsets or file-processing progress.
- State information for stateful operations.

Example:

```python
query = (
    df.writeStream
    .format("delta")
    .option("checkpointLocation", checkpoint_path)
    .start(target_path)
)
```

If the query stops unexpectedly, it can recover from its checkpoint when restarted with the same checkpoint location and compatible query configuration.

### Exactly-Once Semantics

Exactly-once processing means the final committed results are not duplicated or lost within the guarantees provided by the source, processing engine, and sink.

It does not necessarily mean that every physical computation happens only once.

**Important:**

- Checkpointing supports recovery.
- A checkpoint alone does not make every external sink exactly-once.
- The sink must support appropriate transactional or idempotent behavior.
- Delta Lake provides transactional writes for supported Structured Streaming integrations.

## 12. When Should We Use Batch vs Streaming?

Choose based on business requirements.

| Requirement | Suitable Approach |
|---|---|
| Daily sales reports | Batch |
| Monthly financial summaries | Batch |
| Historical data migration | Batch |
| Live payment monitoring | Streaming |
| Real-time fraud detection | Streaming |
| IoT sensor monitoring | Streaming |
| Scheduled incremental file ingestion | Batch or streaming with `AvailableNow` |
| Low-latency event processing | Streaming |

### Practical Decision

Ask these questions:

1. How quickly must the data become available?
2. Does the business need continuous updates?
3. How frequently does new data arrive?
4. What is the expected data volume?
5. Can the business accept scheduled processing?
6. What are the operational and infrastructure costs?

Do not choose streaming just because it sounds more advanced. If daily processing meets the business requirement, batch processing may be simpler and more cost-effective.

## 13. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| Batch processing means small data | Batch can process very large datasets |
| Streaming always means real-time milliseconds | Streaming latency depends on architecture and execution |
| Every streaming system processes one record at a time | Spark Structured Streaming commonly uses micro-batches |
| Event time and processing time are identical | They represent different timestamps |
| Watermark removes all late data immediately | Watermark manages event-time state according to the operation |
| Checkpointing guarantees exactly-once for every sink | End-to-end guarantees depend on the source and sink |
| Auto Loader must run continuously | It can also run incrementally with scheduled `AvailableNow` jobs |
| Streaming is always better than batch | Choose based on latency and business requirements |

## 14. Interview Questions and Answers

**Q1. What is batch processing?**

Batch processing collects data and processes it together at scheduled intervals or when a batch is available.

**Q2. What is streaming processing?**

Streaming processing handles continuously arriving data or data in small increments, allowing results to become available with lower latency.

**Q3. What is the main difference between batch and streaming?**

Batch processes bounded collections of data, while streaming processes data as it arrives over time.

**Q4. What is micro-batch processing?**

Micro-batch processing divides incoming data into small batches and processes them separately. It is the common execution model for Spark Structured Streaming.

**Q5. What is event time?**

Event time is the timestamp when an event actually occurred.

**Q6. What is processing time?**

Processing time is the timestamp when the processing system handles an event.

**Q7. What is late-arriving data?**

Data that reaches the processing system later than its actual event time.

**Q8. What is a watermark?**

A watermark helps Structured Streaming manage event-time state and determine when it can finalize certain operations and clean up old state.

**Q9. What is the difference between tumbling and sliding windows?**

Tumbling windows have fixed, non-overlapping intervals. Sliding windows overlap because they advance at intervals shorter than their duration.

**Q10. Can Auto Loader be used for batch-style ingestion?**

Yes. Auto Loader supports scheduled incremental ingestion using triggers such as `AvailableNow`.

**Q11. Why is checkpointing important?**

It stores streaming progress and state information required for recovery after failures.

**Q12. Is streaming always better than batch?**

No. Batch is often better when the business can accept scheduled results. Streaming is useful when lower latency is required.

**Q13. Can a streaming pipeline write into Delta Lake?**

Yes. Delta Lake supports Structured Streaming reads and writes.

**Q14. What is the difference between continuous processing and micro-batch processing in Spark?**

Micro-batch processes data in small batches and supports a broad range of Structured Streaming operations. Continuous processing is a specialized lower-latency mode with limitations.

## 15. Quick Revision

| Concept | Key Point |
|---|---|
| Batch | Processes data in groups |
| Streaming | Processes incoming data continuously or incrementally |
| Micro-batch | Processes small batches |
| Event time | When the event occurred |
| Processing time | When the system processed the event |
| Late data | Data arriving after its event time |
| Watermark | Manages event-time state |
| Tumbling window | Fixed, non-overlapping |
| Sliding window | Overlapping |
| Session window | Groups activity separated by inactivity |
| Checkpoint | Supports streaming recovery |
| Auto Loader | Incremental cloud-file ingestion |
| AvailableNow | Processes available data and then stops |

### One-Minute Interview Answer

Batch processing processes collected data in groups, usually on a schedule. It is suitable for daily reports, historical processing, and large ETL workloads.

Streaming processing handles incoming data continuously or in small increments. It is useful for payment monitoring, fraud detection, and live dashboards.

In Databricks, both approaches can be implemented using Spark. Structured Streaming commonly uses micro-batch execution. Concepts such as event time, watermarks, windows, and checkpointing are important for building reliable streaming pipelines.

## 16. Self-Assessment

- [ ] Explain batch processing with a real-world example.
- [ ] Explain streaming processing with a real-world example.
- [ ] Compare batch and streaming.
- [ ] Explain micro-batch processing.
- [ ] Differentiate event time and processing time.
- [ ] Explain late-arriving data.
- [ ] Explain watermarks.
- [ ] Differentiate tumbling, sliding, and session windows.
- [ ] Explain checkpointing and recovery.
- [ ] Explain how Auto Loader supports scheduled ingestion.
- [ ] Decide when to use batch versus streaming for a business requirement.

**Interview readiness:** You should be able to explain batch vs streaming in 2 minutes, write a basic PySpark batch job and Structured Streaming query, and explain event time, watermarks, and checkpointing.