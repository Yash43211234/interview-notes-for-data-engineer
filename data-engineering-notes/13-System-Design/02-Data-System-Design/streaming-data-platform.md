

# Streaming Data Platform

## 1. What Is a Streaming Data Platform?

A streaming data platform continuously ingests, processes, and delivers data as events arrive, enabling applications and analytics systems to work with fresh data.

Unlike traditional batch processing, which processes data in scheduled groups, streaming systems can process data continuously or in small, frequent batches.

### Example

An e-commerce company wants to detect suspicious payments as soon as transactions occur.

```text
Payment Application
        |
        v
Message Broker (Kafka)
        |
        v
Stream Processing (Spark)
        |
        v
Validation and Fraud Rules
        |
        v
Alerts / Storage / Dashboard
```

When a payment event arrives, the platform processes it and updates the relevant output without waiting for the next daily batch.

### Common Use Cases

- Fraud detection.
- Real-time dashboards.
- Website clickstream analytics.
- IoT sensor monitoring.
- Application log processing.
- Stock market event processing.
- Real-time recommendation systems.
- Operational alerts.
- Near-real-time data warehouse updates.

---

## 2. Streaming vs Batch Processing

| Aspect | Batch Processing | Streaming Processing |
|---|---|---|
| Input | Bounded dataset | Continuous or incrementally arriving data |
| Execution | Scheduled or triggered jobs | Continuous or repeated incremental execution |
| Latency | Minutes to hours, depending on schedule | Often milliseconds to seconds, depending on architecture |
| Data volume | Processes a defined collection | Processes arriving events incrementally |
| Complexity | Often simpler to operate | Requires handling state, late data, duplicates, and recovery |
| Examples | Daily sales report | Real-time payment monitoring |

**Important:** Streaming does not always mean processing every event immediately. Some streaming engines use micro-batches, and end-to-end latency depends on the entire pipeline.

### Micro-Batch vs Continuous Processing

**Micro-batch:** The engine processes small groups of newly available records at frequent intervals.

**Continuous processing:** The engine processes records continuously rather than grouping them into ordinary micro-batches. Availability and guarantees depend on the technology.

Spark Structured Streaming commonly uses micro-batch execution.

---

## 3. Core Architecture of a Streaming Platform

A typical streaming platform contains these components.

```text
Event Producers
      |
      v
Ingestion Layer
      |
      v
Message Broker
      |
      v
Stream Processing Engine
      |
      +--------------------+
      |                    |
      v                    v
Real-Time Storage      Alerting System
      |
      v
Analytics / Serving Layer
```

### 3.1 Event Producers

Producers generate events.

Examples:

- Payment applications.
- Web servers.
- Mobile applications.
- IoT devices.
- Database change logs.

Example event:

```json
{
  "event_id": "evt-1001",
  "order_id": "ORD-501",
  "event_type": "order_created",
  "amount": 1499.00,
  "event_time": "2026-10-31T10:30:00Z"
}
```

A production event should contain enough information to identify the event, interpret it, and process it correctly.

### 3.2 Ingestion Layer

The ingestion layer receives events from producers and delivers them to the streaming platform.

Responsibilities include:

- Connecting to sources.
- Validating basic event structure.
- Handling authentication.
- Managing producer failures.
- Preserving source metadata where needed.

### 3.3 Message Broker

A message broker stores and distributes events between producers and consumers.

Examples include:

- Apache Kafka.
- Azure Event Hubs.
- Amazon Kinesis Data Streams.

The broker decouples producers from consumers so that they do not need to communicate directly.

### 3.4 Stream Processing Engine

The processing engine reads events and applies transformations.

Examples:

- Apache Spark Structured Streaming.
- Apache Flink.
- Kafka Streams.

Common operations include filtering, aggregation, joins, enrichment, deduplication, and window calculations.

### 3.5 Storage and Serving

Processed data may be written to:

- Delta tables.
- Data warehouses.
- Operational databases.
- Search indexes.
- Caches.
- Monitoring dashboards.
- Alerting systems.

Different consumers may require different output formats and latency guarantees.

---

## 4. Message Broker Fundamentals

A message broker is an intermediary that receives, retains, and distributes messages.

### Important Kafka Concepts

| Concept | Meaning |
|---|---|
| Topic | Named stream of records |
| Partition | Ordered log segment within a topic |
| Producer | Writes records to a topic |
| Consumer | Reads records from a topic |
| Consumer group | Coordinates consumption among group members |
| Offset | Position of a record within a partition |
| Broker | Server that stores and serves topic partitions |
| Replication | Maintains copies of partitions for resilience |
| Retention | Determines how long records remain available |

### Topic and Partition Example

```text
Topic: orders
  |
  +-- Partition 0: event A -> event B -> event C
  |
  +-- Partition 1: event D -> event E -> event F
  |
  +-- Partition 2: event G -> event H -> event I
```

Records are ordered within each partition, but there is no automatic total ordering across all partitions.

### Why Use Partitions?

Partitions enable:

- Parallel processing.
- Higher throughput.
- Distribution across brokers.
- Independent consumption progress.

However, the partitioning key matters.

If all events use the same key, they may be sent to the same partition, creating a throughput bottleneck.

### Consumer Groups

A consumer group allows multiple consumers to process a topic in parallel.

Within a consumer group, a partition is generally assigned to one consumer at a time.

If a topic has four partitions and the group has six consumers, some consumers may remain idle because there are only four partitions to assign.

Different consumer groups can independently consume the same topic.

### Offset Management

Offsets track consumer progress.

They help a consumer resume from a known position after failure.

Offset management alone does not guarantee exactly-once output. If a consumer writes to a database and fails before committing its offset, the same record may be processed again.

The destination must support safe retries or a coordinated transaction strategy.

---

## 5. Stream Processing Engines

A stream processing engine transforms events as they arrive.

### Common Operations

- Filtering unwanted events.
- Converting data types.
- Validating schemas.
- Deduplicating records.
- Joining streams with reference data.
- Calculating rolling aggregates.
- Detecting patterns.
- Writing results to destinations.

### Spark Structured Streaming Example

The following example reads order events from Kafka and converts the JSON payload into a structured DataFrame.

```python
from pyspark.sql import functions as F
from pyspark.sql.types import (
    StructType,
    StructField,
    StringType,
    DoubleType
)

schema = StructType([
    StructField("event_id", StringType(), True),
    StructField("order_id", StringType(), True),
    StructField("event_type", StringType(), True),
    StructField("amount", DoubleType(), True),
    StructField("event_time", StringType(), True)
])

kafka_df = (
    spark.readStream
    .format("kafka")
    .option("kafka.bootstrap.servers", "<broker:9092>")
    .option("subscribe", "orders")
    .option("startingOffsets", "latest")
    .load()
)

orders_df = (
    kafka_df
    .select(
        F.from_json(
            F.col("value").cast("string"),
            schema
        ).alias("data")
    )
    .select("data.*")
    .withColumn(
        "event_time",
        F.to_timestamp("event_time")
    )
)
```

This produces a streaming DataFrame containing structured order events.

Important points:

- Replace the broker placeholder with the actual Kafka bootstrap server.
- Configure authentication and TLS when required.
- `startingOffsets` applies when establishing a new query without an existing checkpoint; recovery from a checkpoint follows the saved progress.
- An explicit schema defines the expected structure but does not guarantee that every event is valid.
- Invalid JSON or missing fields should be detected and handled explicitly.

---

## 6. Event Time vs Processing Time

Time is a critical concept in streaming systems.

### Event Time

The time when the event actually occurred.

Example: A customer places an order at 10:00 AM.

### Processing Time

The time when the processing engine handles the event.

Example: The order reaches the processing engine at 10:03 AM.

| Event | Event time | Processing time |
|---|---|---|
| Order A | 10:00 | 10:01 |
| Order B | 10:02 | 10:02 |
| Order C | 10:01 | 10:04 |

Order C arrives late even though its event time is earlier than Order B.

### Why Event Time Matters

Suppose a business wants total sales for every five-minute period.

If the system groups records only by processing time, delayed events can be assigned to the wrong period.

Event-time processing allows the system to calculate results according to when the events occurred.

---

## 7. Windowing

Windowing groups streaming events into time-based or other defined groups.

It is commonly used for rolling metrics, monitoring, and real-time analytics.

### 7.1 Tumbling Window

A tumbling window has a fixed duration and does not overlap with the next window.

Example: Five-minute windows.

```text
10:00–10:05
10:05–10:10
10:10–10:15
```

Each event belongs to one window based on its event time.

### 7.2 Sliding Window

A sliding window advances at a shorter interval than its duration, so windows overlap.

Example: A ten-minute window that advances every five minutes.

```text
10:00–10:10
10:05–10:15
10:10–10:20
```

An event can belong to multiple windows.

### 7.3 Session Window

A session window groups events separated by no more than a configured inactivity gap.

Example: Website activity from one user can form a session until the user remains inactive long enough to close it.

### Windowing in PySpark

```python
windowed_sales = (
    orders_df
    .filter(F.col("event_type") == "order_created")
    .withWatermark("event_time", "10 minutes")
    .groupBy(
        F.window("event_time", "5 minutes")
    )
    .agg(
        F.sum("amount").alias("total_sales"),
        F.count("*").alias("order_count")
    )
)
```

This example calculates sales and order counts in five-minute event-time windows.

The ten-minute watermark defines how the engine tracks event-time progress and handles sufficiently late data for supported stateful operations. It is not a guarantee that every event arriving within ten minutes will be accepted under every execution scenario.

---

## 8. Watermarks and Late-Arriving Events

A watermark helps a streaming engine reason about event-time progress and limit the state it must retain for stateful operations.

### Example

Suppose a streaming job uses a ten-minute watermark.

An event with an event time much older than the current watermark may be excluded from a stateful window aggregation.

This allows the system to eventually finalize windows and discard old state.

### Important Points

- A watermark is based on observed event-time progress, not simply the wall clock.
- It does not mean that the system waits exactly ten minutes for every event.
- Late-event handling depends on the operator, output mode, and engine behavior.
- Stateful operations may discard sufficiently late data.
- The watermark policy is a trade-off between completeness, latency, and state size.

For multiple input streams, Spark's global watermark policy commonly uses the minimum of input watermarks by default, though configuration can change the policy.

---

## 9. State Management

Some streaming operations require information from previously processed events.

This information is called state.

### Examples

- Running totals.
- Deduplication across events.
- Stream-stream joins.
- Session windows.
- Tracking the latest event per customer.

### Stateful vs Stateless Operations

| Stateless | Stateful |
|---|---|
| Each record can be processed independently | Processing depends on retained information |
| Simple filtering | Windowed aggregation |
| Basic column transformation | Stream-stream join |
| Many parsing operations | Stateful deduplication |

Stateful operations need memory and durable recovery mechanisms. Large or unbounded state can increase storage use and processing latency.

Watermarks and appropriate keys help control state growth for supported operations.

---

## 10. Delivery Semantics and Exactly-Once Results

Streaming platforms commonly discuss three delivery semantics.

### At-Most-Once

An event is processed zero or one time. Data may be lost if failure occurs at the wrong point.

### At-Least-Once

An event is processed one or more times. Duplicate processing is possible.

### Exactly-Once

The system aims to ensure each event has one effective committed result within the defined processing boundary.

Exactly-once guarantees depend on the source, processing engine, checkpointing, and sink.

A checkpoint alone does not guarantee end-to-end exactly-once results.

### Example of Duplicate Processing

```text
1. Read event E1.
2. Write E1 to the target.
3. Fail before recording source progress.
4. Restart and read E1 again.
5. Write E1 again.
```

The target may now contain duplicates.

### How to Reduce Duplicate Effects

- Use event IDs or business keys.
- Deduplicate source records.
- Use idempotent upserts.
- Use transactional or coordinated source-sink integration where supported.
- Persist checkpoints correctly.
- Design recovery around partial failures.

**Interview point:** Distinguish processing an event once from achieving one effective committed result. Those are not always the same thing.

---

## 11. Fault Tolerance and Recovery

Fault tolerance allows a streaming system to continue operating or recover after failures.

Potential failures include:

- Worker or executor loss.
- Broker failure.
- Network interruption.
- Processing exceptions.
- Sink outage.
- Invalid records.
- Checkpoint access problems.

### Recovery Mechanisms

1. Broker replication.
2. Consumer offset management.
3. Streaming checkpoints.
4. Stateful operator recovery.
5. Retry policies.
6. Idempotent sink writes.
7. Dead-letter or quarantine storage.
8. Monitoring and alerting.

### Spark Structured Streaming Checkpoint

```python
query = (
    orders_df.writeStream
    .format("delta")
    .outputMode("append")
    .option(
        "checkpointLocation",
        "abfss://checkpoints@storage.dfs.core.windows.net/orders/"
    )
    .toTable("bronze.orders")
)
```

The checkpoint location stores progress and, where applicable, state needed for recovery.

Use the same compatible checkpoint when restarting the same query. Do not delete it casually, because doing so can cause the query to lose its prior progress and state history.

The target write semantics still matter: checkpointing does not make every arbitrary sink transactionally exactly-once.

---

## 12. Backpressure and Consumer Lag

### Backpressure

Backpressure occurs when incoming data arrives faster than the processing system can handle it.

For example:

```text
Incoming rate:   50,000 events/second
Processing rate: 30,000 events/second
```

The backlog can grow over time.

### Consumer Lag

Consumer lag represents how far a consumer is behind the latest available records, commonly measured in records or offsets.

High lag can indicate:

- Insufficient processing capacity.
- Slow transformations.
- Slow destination writes.
- Uneven partition distribution.
- Sudden traffic spikes.
- A failed or stalled consumer.

### How to Handle Lag

- Optimize expensive transformations.
- Increase parallelism where useful.
- Add partitions when appropriate.
- Scale processing resources.
- Improve sink throughput.
- Address skewed keys.
- Apply appropriate rate limits.
- Monitor incoming rate, processing rate, and lag together.

Adding workers does not always solve the problem. A single hot partition, slow external database, or serial operation may remain the bottleneck.

---

## 13. Data Quality in Streaming Pipelines

Streaming pipelines need data-quality checks just like batch pipelines.

### Common Checks

- Required fields are present.
- Event IDs are valid.
- Timestamps can be parsed.
- Numeric values are within acceptable ranges.
- Event types are recognized.
- Duplicate events are identified.
- Schema versions are supported.

### Example

```python
valid_orders = orders_df.filter(
    F.col("event_id").isNotNull() &
    F.col("order_id").isNotNull() &
    F.col("event_time").isNotNull() &
    F.col("amount").isNotNull() &
    (F.col("amount") >= 0)
)

invalid_orders = orders_df.filter(
    F.col("event_id").isNull() |
    F.col("order_id").isNull() |
    F.col("event_time").isNull() |
    F.col("amount").isNull() |
    (F.col("amount") < 0)
)
```

The valid and invalid streams should be handled using appropriate output sinks and recovery policies.

For production use, also preserve the original payload and failure reason for rejected records. A validation filter alone does not persist invalid data.

---

## 14. Streaming Platform Performance

Performance depends on the ingestion layer, broker, processing engine, state store, and destination.

### Important Factors

**Partitioning:** Enables parallel processing but can create skew when keys are unevenly distributed.

**Serialization:** Efficient formats and schemas reduce processing overhead.

**State size:** Large state can increase memory and storage pressure.

**Batch or trigger interval:** In micro-batch systems, trigger settings affect latency and workload behavior.

**Sink throughput:** A slow destination can limit the entire pipeline.

**Network:** Network throughput and latency affect ingestion and delivery.

**Resource sizing:** Compute resources must match workload volume and complexity.

### Performance Metrics

| Metric | What it indicates |
|---|---|
| Throughput | Events processed per second |
| End-to-end latency | Time from event creation to usable output |
| Processing latency | Time spent in a processing stage |
| Consumer lag | Distance behind available source records |
| Error rate | Frequency of processing failures |
| State size | Amount of retained state |
| Backlog | Amount of work waiting to be processed |

Do not optimize only for throughput. A system may have high throughput while still delivering data too late for the business requirement.

---

## 15. Security and Governance

Streaming platforms often process sensitive or business-critical information.

Important controls include:

- Authentication for producers and consumers.
- Authorization at topic and resource levels.
- Encryption in transit.
- Encryption at rest.
- Secrets management.
- Data masking or minimization.
- Audit logging.
- Schema and data-contract governance.
- Retention policies.
- Access controls on checkpoints and output tables.

A private network connection does not automatically grant data access or guarantee that the entire system is secure. Network controls, authentication, authorization, and data governance are separate requirements.

---

## 16. Azure Streaming Architecture

A typical Azure-based streaming platform can use the following components:

```text
Applications / IoT Devices
           |
           v
     Azure Event Hubs
           |
           v
  Azure Databricks
  Structured Streaming
           |
           +------------------+
           |                  |
           v                  v
   ADLS Gen2 / Delta      Alerting / Apps
           |
           v
   Gold Tables / BI
```

### Component Responsibilities

| Component | Responsibility |
|---|---|
| Azure Event Hubs | Ingest and retain event streams |
| Azure Databricks | Transform and process streams |
| ADLS Gen2 | Store raw and processed data |
| Delta Lake | Provide transactional table storage |
| Unity Catalog | Manage supported data governance and access controls |
| Power BI | Visualize analytics results |

Exact architecture depends on latency requirements, event volume, source capabilities, and the desired serving layer.

### Bronze, Silver, and Gold

**Bronze:** Retain incoming events with useful source metadata.

**Silver:** Validate, clean, deduplicate, and standardize records.

**Gold:** Publish business-level aggregates and analytics-ready data.

The medallion architecture is a logical organization of data, not a requirement that every pipeline must use exactly three layers.

### Example

```text
Event Hubs
    |
    v
Bronze: Raw Events
    |
    v
Silver: Validated and Deduplicated Events
    |
    v
Gold: Five-Minute Sales Metrics
    |
    v
Dashboard / Business Applications
```

For reliable operations, configure checkpoint storage, managed identity or another supported authentication mechanism, access permissions, monitoring, and appropriate retention.

---

## 17. Streaming vs Change Data Capture (CDC)

Streaming and CDC are related but different concepts.

**Streaming** describes how data is continuously or incrementally processed.

**Change Data Capture** identifies changes made to a source system, such as inserts, updates, and deletes.

CDC events can be delivered through a streaming platform.

Example:

```text
Orders Database
      |
      v
CDC Connector
      |
      v
Kafka / Event Hubs
      |
      v
Stream Processor
      |
      v
Target Delta Table
```

CDC consumers must account for:

- Operation type.
- Primary key.
- Event ordering.
- Source transaction metadata.
- Duplicate events.
- Deletes.
- Schema changes.

A simple append-only pipeline is not sufficient when the target must reflect source updates and deletions. In those cases, the processing logic must apply changes according to the CDC semantics.

---

## 18. Common Mistakes

1. Assuming all streaming systems process every event immediately.
2. Confusing event time with processing time.
3. Assuming records are globally ordered across Kafka partitions.
4. Assuming a checkpoint alone guarantees exactly-once results.
5. Deleting checkpoints without considering recovery.
6. Ignoring late-arriving events.
7. Allowing state to grow without bounds.
8. Scaling compute without investigating the actual bottleneck.
9. Treating every processing failure as transient.
10. Ignoring schema evolution and invalid events.
11. Assuming adding Kafka consumers always increases throughput.
12. Ignoring sink idempotency.
13. Treating streaming and CDC as identical concepts.
14. Monitoring job status but ignoring consumer lag and data freshness.

---

## 19. Interview Questions and Answers

### Q1. What is a streaming data platform?

A system that continuously or incrementally ingests, processes, and delivers events so that downstream systems can use fresh data.

### Q2. What is the difference between batch and streaming?

Batch processes a bounded collection of data at a time. Streaming processes data incrementally as it arrives, often with lower latency.

### Q3. What is the role of Kafka?

Kafka provides a distributed event log that producers write to and consumers read from. It decouples data producers from downstream consumers.

### Q4. What is a Kafka partition?

A partition is an ordered log segment of a topic. Partitions enable parallelism and distribute data across brokers.

### Q5. What is consumer lag?

Consumer lag measures how far a consumer is behind the latest available records. It may indicate insufficient processing capacity or a bottleneck.

### Q6. What is the difference between event time and processing time?

Event time is when the event occurred. Processing time is when the system handled it.

### Q7. What is a watermark?

A watermark tracks event-time progress and helps stateful streaming operations decide when to finalize results and discard sufficiently old state.

### Q8. What is the difference between tumbling and sliding windows?

Tumbling windows do not overlap. Sliding windows overlap when their duration is greater than their slide interval.

### Q9. Why does streaming need state management?

Operations such as aggregations, deduplication, and stream joins need information from previous events to calculate correct results.

### Q10. What is backpressure?

Backpressure occurs when the incoming data rate exceeds the rate at which the system can process or deliver the data.

### Q11. How do you prevent duplicate output after a failure?

Use stable event IDs or business keys, deduplicate records, design idempotent writes, and coordinate source progress with destination commits where supported.

### Q12. Does Spark Structured Streaming use checkpoints?

Yes. Checkpoints retain query progress and state information needed to recover a streaming query.

### Q13. How would you handle late-arriving events?

Choose an appropriate watermark and state-retention policy, understand how the operations handle late data, and use reconciliation or historical correction when necessary.

### Q14. How would you design a real-time sales dashboard?

I would ingest order events through Kafka or Event Hubs, process and validate them using Spark Structured Streaming, calculate event-time window aggregates, store the results in a suitable serving layer, and monitor freshness, lag, failures, and data quality.

### Q15. How do streaming and CDC work together?

CDC captures source database changes and delivers them as events. A streaming processor consumes those events and applies inserts, updates, and deletes to downstream systems.

### Q16. What is the difference between at-least-once and exactly-once?

At-least-once permits repeated processing. Exactly-once aims for one effective committed result within a defined processing boundary, depending on the source, engine, and sink guarantees.

### Q17. How would you troubleshoot high streaming latency?

I would compare ingestion rate, processing throughput, consumer lag, stage duration, state size, and sink latency. Then I would identify the bottleneck before changing resources or configuration.

### Q18. What is the purpose of the Bronze layer in streaming?

It preserves incoming data with useful source metadata so that downstream transformations can be audited, corrected, or reprocessed.

---

## 20. Quick Revision

| Concept | Key point |
|---|---|
| Streaming | Incremental processing of arriving data |
| Message broker | Decouples producers and consumers |
| Topic | Named stream of events |
| Partition | Enables parallelism and per-partition ordering |
| Consumer group | Distributes partition consumption |
| Offset | Tracks a position in a partition |
| Event time | When the event occurred |
| Processing time | When the engine processed it |
| Tumbling window | Fixed, non-overlapping windows |
| Sliding window | Overlapping windows |
| Watermark | Tracks event-time progress |
| State | Information retained across events |
| Backpressure | Input arrives faster than processing capacity |
| Consumer lag | Consumer is behind available records |
| Checkpoint | Supports progress and state recovery |
| CDC | Captures source data changes |
| Idempotency | Makes repeated operations safe |

### Five Key Interview Points

1. Kafka partitions provide parallelism and ordering within each partition.
2. Event time is important when events arrive late or out of order.
3. Watermarks help control state and late-event handling.
4. Checkpointing alone does not guarantee end-to-end exactly-once results.
5. Monitor latency, throughput, lag, data quality, and recovery—not just job status.

---

## 21. Self-Assessment

Try answering these questions without looking at the notes.

1. Design a streaming platform for real-time payment monitoring.
2. Explain how Kafka partitions and consumer groups work.
3. What happens when incoming events arrive faster than the processing engine can handle them?
4. Why are event time and watermarks important?
5. How do tumbling and sliding windows differ?
6. How can a streaming pipeline recover without duplicating target records?
7. Why might consumer lag increase even after adding more workers?
8. How would you handle invalid events without losing them?
9. How would you use Event Hubs, Databricks, and ADLS Gen2 in Azure?
10. Explain how CDC events can update a Delta table correctly.

