

# Apache Flink Fundamentals

## 1. Introduction

**Apache Flink** is an open-source distributed processing framework designed for stateful computations over bounded and unbounded data streams.

It is widely used for:
- Real-time data processing.
- Event-driven applications.
- Stateful stream processing.
- Real-time analytics.
- Fraud detection.
- Monitoring and alerting.
- Continuous data pipelines.

Flink can process both streaming and batch workloads. Its streaming runtime is the foundation of its execution model.

### Example

An e-commerce platform receives thousands of order events every second.

Flink can continuously:
1. Read events from Kafka.
2. Validate incoming records.
3. Calculate real-time revenue.
4. Detect unusual transactions.
5. Write results to databases or data lakes.

```text
E-commerce Application
          |
          v
      Kafka Topic
          |
          v
      Apache Flink
          |
    ┌─────┴─────┐
    v           v
Real-time    Fraud Detection
Analytics         |
    |             |
    v             v
Dashboard     Alert System
```

---

## 2. What Is Apache Flink?

Apache Flink is a distributed data processing engine that supports stateful stream processing with event-time semantics and fault-tolerant execution.

Its important capabilities include:

- Continuous stream processing.
- Stateful computations.
- Event-time processing.
- Watermarks.
- Checkpointing.
- Distributed execution.
- Exactly-once state consistency under supported configurations.
- Integration with Kafka, databases, and storage systems.

### Important point

Flink is a processing engine, not a message broker.

Kafka stores and distributes events. Flink processes those events.

---

## 3. Why Do We Need Flink?

Traditional batch processing waits for a collection of data before processing it.

However, many applications need results continuously.

### Batch Processing

```text
Data Collection
       |
       v
Wait for Batch
       |
       v
Process Data
       |
       v
Generate Result
```

### Stream Processing

```text
Event 1 ──┐
Event 2 ──┼──> Continuous Processing
Event 3 ──┤
Event 4 ──┘
                  |
                  v
             Updated Results
```

### Example

Suppose a payment company wants to identify suspicious transactions immediately.

A batch system may identify suspicious activity only after a scheduled processing run.

Flink can process transactions continuously and evaluate fraud-detection rules as events arrive.

### When Flink is useful

- Low-latency event processing.
- Real-time aggregations.
- Stateful event processing.
- Complex event detection.
- Event-time window calculations.
- Continuous enrichment and transformation.

---

## 4. Flink Architecture

Flink uses a distributed architecture with different components responsible for job coordination and execution.

The main components are:

1. JobManager
2. TaskManager
3. Task Slots
4. Job
5. Tasks
6. Operators

### Architecture diagram

```text
                Flink Client
                     |
                     v
               Job Submission
                     |
                     v
              JobManager
          ┌──────────┴──────────┐
          |                     |
          v                     v
    Job Scheduling        Checkpoint
                          Coordination
          |
          v
    TaskManager 1              TaskManager 2
    ┌────────────┐             ┌────────────┐
    | Task Slot  |             | Task Slot  |
    |            |             |            |
    | Source     |             | Transform  |
    | Operator   |             | Operator   |
    └────────────┘             └────────────┘
          |                          |
          └──────── Data Flow ───────┘
```

### A. JobManager

JobManager coordinates the execution of Flink applications.

Responsibilities include:
- Receiving job submissions.
- Scheduling tasks.
- Coordinating distributed execution.
- Coordinating checkpoints.
- Managing job execution and recovery.

In Flink's architecture, the JobManager includes important internal components such as:
- Dispatcher.
- ResourceManager.
- JobMaster.

Their exact responsibilities and deployment details depend on the Flink version and cluster setup.

### B. TaskManager

TaskManagers are worker processes that execute the tasks assigned to them.

Responsibilities include:
- Executing operators.
- Processing records.
- Managing task slots.
- Exchanging data with other tasks.
- Managing task-related state.

### C. Task Slots

A TaskManager provides one or more task slots.

A task slot is a unit of resource allocation for task execution.

For example:

```text
TaskManager 1
    |
    ├── Slot 1
    ├── Slot 2
    └── Slot 3
```

Multiple tasks can execute in the same slot through operator chaining and slot-sharing arrangements.

A task slot is not necessarily one CPU core.

### D. Job

A Job represents a Flink application submitted for execution.

For example, a Kafka-to-Delta streaming application can be submitted as a Flink job.

### E. Task

A Task is an execution unit created from the application's operators and execution graph.

### F. Operator

An operator performs a processing operation, such as:
- Reading records.
- Filtering records.
- Transforming records.
- Aggregating records.
- Writing results.

---

## 5. Flink Data Processing Flow

A typical Flink application follows this structure:

```text
Source
   |
   v
Transformation
   |
   v
Keying / Partitioning
   |
   v
Stateful Processing
   |
   v
Sink
```

### Source

A Source reads data from an external system.

Examples:
- Kafka.
- Files.
- Databases through supported connectors.
- Custom sources.

### Transformation

Transformations modify or process records.

Examples:
- Map.
- Filter.
- FlatMap.
- KeyBy.
- Process functions.

### Sink

A Sink writes the processed data to an external destination.

Examples:
- Kafka.
- Databases.
- Filesystems.
- Data lakes.

### Example

```text
Kafka Source
     |
     v
Parse JSON
     |
     v
Filter Invalid Events
     |
     v
Group by Customer
     |
     v
Calculate Spending
     |
     v
Database / Kafka Sink
```

---

## 6. Flink APIs

Flink provides different APIs for processing data.

### A. DataStream API

The DataStream API is used for stream processing.

It provides operations such as:
- `map()`
- `filter()`
- `flatMap()`
- `keyBy()`
- `process()`
- `window()`

Example using Java:

```java
DataStream<String> events = env.fromSource(
    source,
    WatermarkStrategy.noWatermarks(),
    "Kafka Source"
);

DataStream<String> validEvents = events
    .filter(event -> event != null)
    .map(event -> event.trim());
```

### B. Table API

The Table API provides a relational abstraction for processing data.

It supports:
- Table operations.
- Filtering.
- Aggregations.
- Joins.
- SQL-like transformations.

### C. Flink SQL

Flink SQL allows developers to process streaming data using SQL syntax.

Example:

```sql
SELECT
    customer_id,
    COUNT(*) AS total_orders,
    SUM(amount) AS total_amount
FROM orders
GROUP BY customer_id;
```

The exact behavior of a streaming query depends on its source, event-time configuration, and whether the query produces updating or append-only results.

### API comparison

| API | Purpose |
|---|---|
| DataStream API | Programmatic stream processing |
| Table API | Relational data processing |
| Flink SQL | SQL-based stream and batch processing |

---

## 7. Stateful vs Stateless Processing

One of Flink's major strengths is stateful stream processing.

### A. Stateless Processing

A stateless operation processes each record independently.

It does not need to remember previous records.

Example:

```text
Input Event
     |
     v
Convert Amount to INR
     |
     v
Output Event
```

Other examples:
- Filtering.
- Simple mapping.
- Data format conversion.

### B. Stateful Processing

A stateful operation maintains information across multiple records.

Example:

A customer makes multiple purchases.

Flink maintains a running spending total for each customer.

```text
Customer C01
   |
   ├── Order 100
   ├── Order 200
   └── Order 300
          |
          v
   Running Total = 600
```

### Common stateful use cases

- Running totals.
- Session tracking.
- Fraud detection.
- Event deduplication.
- Window aggregations.
- Pattern detection.

### Comparison

| Stateless | Stateful |
|---|---|
| Processes records independently | Maintains information across records |
| No application state required | Uses managed state |
| Simpler recovery | Requires state recovery |
| Example: filter | Example: running customer total |

---

## 8. Flink State Management

Flink provides managed state so that applications can maintain information across events.

Common state types include:

### A. Keyed State

State associated with a key.

Example:

```text
Customer C01 → Spending = 500
Customer C02 → Spending = 800
Customer C03 → Spending = 300
```

Each key maintains its own state.

Keyed state is used after partitioning the stream by a key.

### B. Operator State

State associated with a parallel operator instance rather than an individual key.

It is useful when an operator maintains its own information, such as source-reader progress or partition assignments.

### C. State Backends

State backends determine how Flink stores and accesses state.

Common choices include:
- HashMapStateBackend.
- EmbeddedRocksDBStateBackend.

The HashMap state backend keeps working state in JVM heap memory.

The Embedded RocksDB state backend stores working state using RocksDB, with storage and memory behavior depending on configuration.

### Common state types

| State | Purpose |
|---|---|
| ValueState | Stores one value |
| ListState | Stores a list of values |
| MapState | Stores key-value entries |
| ReducingState | Stores an aggregated value |
| AggregatingState | Stores an aggregate using an accumulator |

Exact APIs and state-backend configuration can vary by Flink version.

---

## 9. KeyBy and Parallel Processing

Flink uses `keyBy()` to partition records based on a key.

Example:

```java
DataStream<Order> keyedOrders = orders
    .keyBy(order -> order.customerId);
```

Records with the same key are routed to the same keyed stream partition.

### Example

Input:

| Customer | Amount |
|---|---:|
| C01 | 100 |
| C02 | 200 |
| C01 | 300 |
| C03 | 150 |
| C02 | 100 |

After `keyBy(customer_id)`:

```text
C01 → 100, 300
C02 → 200, 100
C03 → 150
```

This allows each key's stateful computation to be processed consistently.

### Why is keyBy important?

- Enables keyed state.
- Supports per-key aggregation.
- Supports per-key windows.
- Enables parallel processing by key.

**Important:** A heavily skewed key can create a hot partition and reduce processing performance.

---

## 10. Event Time and Watermarks in Flink

Flink supports event-time processing.

### Event Time

The timestamp when an event actually occurred.

### Processing Time

The time when Flink processes the event.

### Ingestion Time

The time when an event enters a processing system, depending on where it is recorded.

Example:

```text
Order Created:       10:00 AM
Kafka Receives:      10:02 AM
Flink Processes:     10:05 AM
```

| Timestamp | Value |
|---|---|
| Event Time | 10:00 AM |
| Ingestion Time | 10:02 AM |
| Processing Time | 10:05 AM |

### What is a Watermark?

A watermark is a mechanism that indicates progress in event time.

It helps Flink determine when it can make progress with event-time windows and how to handle late-arriving events.

Example:

```text
Watermark = 10:10 AM
```

This indicates that the system considers event-time progress to have reached approximately 10:10 AM under the configured watermark strategy.

Events with timestamps earlier than the watermark may be considered late.

The exact handling depends on the operator and application configuration.

### Example: Watermark Strategy

```java
WatermarkStrategy<Order> strategy =
    WatermarkStrategy
        .<Order>forBoundedOutOfOrderness(
            Duration.ofMinutes(5)
        )
        .withTimestampAssigner(
            (order, timestamp) -> order.eventTime
        );
```

This configures a bounded-out-of-orderness watermark strategy with a five-minute delay.

### Why are watermarks important?

- Handle out-of-order events.
- Support event-time windows.
- Enable event-time progress.
- Help manage late events.
- Support cleanup of completed window state.

---

## 11. Windows in Flink

Windows group events into finite collections for calculations.

Common window types include:

### A. Tumbling Windows

Fixed-size, non-overlapping windows.

Example: Five-minute windows.

```text
10:00 ───── 10:05
10:05 ───── 10:10
10:10 ───── 10:15
```

### B. Sliding Windows

Fixed-size windows that overlap.

Example:
- Window size: 10 minutes.
- Slide interval: 5 minutes.

```text
Window 1: 10:00 ───── 10:10
Window 2: 10:05 ───── 10:15
Window 3: 10:10 ───── 10:20
```

### C. Session Windows

Group events based on periods of activity separated by inactivity gaps.

Example:

A user watches videos. If no activity occurs for 30 minutes, the session ends.

### Window comparison

| Window | Behavior |
|---|---|
| Tumbling | Fixed and non-overlapping |
| Sliding | Fixed and overlapping |
| Session | Groups activity separated by inactivity |

Flink supports event-time and processing-time windowing.

---

## 12. Checkpointing in Flink

Checkpointing is a fault-tolerance mechanism that periodically captures the state of a Flink application.

It allows the application to recover from failures using a consistent checkpoint.

### Why is checkpointing required?

Consider a running application that maintains customer spending totals.

If a TaskManager fails, the application needs to recover its state.

Without a recoverable checkpoint, previously accumulated state may be lost.

### Checkpointing flow

```text
Flink Application
       |
       v
Checkpoint Triggered
       |
       v
Capture Operator State
       |
       v
Capture Source Progress
       |
       v
Persist Checkpoint
       |
       v
Failure Occurs
       |
       v
Restore from Checkpoint
       |
       v
Resume Processing
```

### Example configuration

```java
env.enableCheckpointing(60000);
```

This enables checkpointing every 60 seconds.

### Checkpointing stores

Depending on the source, operators, and configuration, a checkpoint can contain:
- Operator state.
- Keyed state.
- Source positions or progress.
- Other recovery-related metadata.

### Important points

- Checkpoints support recovery.
- Checkpoint storage should be durable and accessible after worker failure.
- Checkpoint intervals affect recovery point and overhead.
- Checkpointing does not automatically make every external sink exactly-once.

---

## 13. Savepoints vs Checkpoints

Both savepoints and checkpoints capture application state, but they serve different operational purposes.

| Checkpoint | Savepoint |
|---|---|
| Primarily for automatic fault recovery | Primarily for controlled operational actions |
| Usually triggered automatically | Usually triggered manually or through an operational workflow |
| Managed by Flink's checkpoint mechanism | Used for upgrades, migration, and planned restarts |
| Lifecycle often managed by Flink | Usually retained according to user policy |

### Checkpoint use cases

- Automatic recovery after failures.
- Restoring application state during execution.

### Savepoint use cases

- Upgrading a Flink application.
- Changing application parallelism.
- Migrating a job to another cluster.
- Planned application shutdown and restart.

**Important:** Savepoints and checkpoints both contain recovery state, but their lifecycle and operational guarantees differ.

---

## 14. Fault Tolerance and Exactly-Once Processing

Flink supports fault-tolerant state recovery through checkpointing.

It can provide exactly-once state consistency when the source, checkpointing configuration, and processing topology support it.

However, end-to-end exactly-once behavior also depends on the sink.

### Example

```text
Kafka Source
     |
     v
Flink Processing
     |
     v
Checkpointing
     |
     v
Transactional Kafka Sink
```

For end-to-end exactly-once behavior, the source and sink must support the necessary coordination and recovery semantics.

### Delivery guarantees

| Guarantee | Meaning |
|---|---|
| At-most-once | Records may be lost, but are not processed more than once within the defined guarantee |
| At-least-once | Records may be processed more than once |
| Exactly-once | Each committed effect occurs once within the supported guarantee boundary |

### Important distinction

Exactly-once does not necessarily mean that a record is physically executed only once.

A record may be processed again after recovery, but the supported transactional or checkpoint-coordinated mechanism prevents duplicate committed effects within its guarantee boundary.

---

## 15. Flink vs Apache Spark Structured Streaming

Both Flink and Spark Structured Streaming are distributed data-processing technologies.

They support streaming workloads, stateful processing, and event-time operations.

### Comparison

| Feature | Apache Flink | Spark Structured Streaming |
|---|---|---|
| Primary execution model | Native stream-processing runtime | Structured processing with micro-batch as the common execution mode |
| Processing | Continuous streaming capabilities | Micro-batch by default |
| Event-time processing | Supported | Supported |
| Watermarks | Supported | Supported |
| Stateful processing | Supported | Supported |
| Checkpointing | Supported | Supported |
| SQL support | Flink SQL | Spark SQL |
| Batch processing | Supported | Supported |
| Ecosystem | Flink connectors and APIs | Spark, Databricks, Delta Lake, and related ecosystem |

### When to choose Flink

- Continuous, low-latency stream processing.
- Complex event processing.
- Long-running stateful applications.
- Applications requiring fine-grained event-time control.

### When to choose Spark Structured Streaming

- Unified batch and streaming workloads.
- Existing Spark or Databricks environments.
- Large-scale ETL pipelines.
- Strong integration with Delta Lake and lakehouse architectures.

**Important:** Neither framework is universally better. Selection depends on latency, state, ecosystem, operational complexity, and business requirements.

---

## 16. Flink vs Kafka

Kafka and Flink solve different problems.

| Kafka | Flink |
|---|---|
| Distributed event-streaming platform | Distributed processing engine |
| Stores and distributes events | Processes events |
| Uses topics and partitions | Uses operators and tasks |
| Supports event retention and replay | Supports transformations and stateful computations |
| Handles producers and consumers | Runs stream-processing applications |

### How they work together

```text
Application
    |
    v
Kafka Topic
    |
    v
Flink Source
    |
    v
Transformations
    |
    v
Flink Sink
    |
    v
Kafka / Database / Data Lake
```

Kafka provides the event stream, while Flink processes it.

---

## 17. Practical E-Commerce Example

Consider an e-commerce company that wants to calculate real-time customer spending and detect suspicious orders.

### Architecture

```text
E-commerce Application
          |
          v
      Kafka Topic
          |
          v
      Flink Source
          |
          v
    Parse Order Events
          |
          v
    Validate Records
          |
          v
    Key By Customer ID
          |
          v
    Stateful Processing
          |
      ┌───┴────┐
      v        v
  Revenue   Fraud Detection
  Metrics       |
      |         v
      |      Alerts
      v
  Kafka / Database
```

### Processing steps

1. Applications publish order events to Kafka.
2. Flink reads events from Kafka.
3. It parses and validates records.
4. It groups events by customer ID.
5. It maintains keyed state for customer activity.
6. It calculates spending and detects unusual patterns.
7. It writes results to downstream systems.
8. Checkpoints support recovery after failures.

### Example event

```json
{
  "order_id": 101,
  "customer_id": "C01",
  "amount": 500.0,
  "event_time": "2026-10-25T10:00:00Z"
}
```

### Business benefits

- Near-real-time analytics.
- Faster fraud detection.
- Continuous customer activity tracking.
- Reduced delay in business decisions.

---

## 18. Flink Performance and Scalability

Flink supports parallel execution and distributed processing.

Important performance concepts include:

### A. Parallelism

Parallelism determines how many parallel instances execute an operator.

For example:

```text
Source Parallelism = 4
Filter Parallelism = 4
Sink Parallelism = 2
```

Different operators can have different parallelism.

### B. Backpressure

Backpressure occurs when downstream operators cannot process data as quickly as upstream operators produce it.

Example:

```text
Source:  10,000 records/sec
Transform: 8,000 records/sec
Sink:     5,000 records/sec
```

The sink becomes a bottleneck, and backpressure can propagate upstream.

### C. Checkpoint Overhead

Frequent checkpoints can add processing overhead and increase storage activity.

Checkpoint configuration should balance:
- Recovery requirements.
- Checkpoint duration.
- Processing throughput.
- Storage cost.

### D. State Size

Large state can increase:
- Checkpoint duration.
- Recovery time.
- Storage requirements.
- Memory and disk usage.

### E. Key Skew

If a few keys receive most records, some subtasks may become overloaded.

Possible approaches include:
- Reviewing key distribution.
- Adjusting parallelism.
- Using a suitable keying strategy.
- Applying carefully designed two-stage aggregation when semantics permit.

---

## 19. Common Flink Problems and Troubleshooting

| Problem | Possible cause | Approach |
|---|---|---|
| Job fails | Application or runtime exception | Inspect JobManager and TaskManager logs |
| High processing latency | Backpressure or slow operators | Identify the bottleneck |
| Checkpoint fails | Storage, timeout, or state issue | Review checkpoint logs and storage |
| State grows continuously | Unbounded state retention | Review state cleanup and TTL where appropriate |
| Uneven task load | Key skew | Analyze key distribution |
| Events arrive late | Network or source delays | Review event-time and watermark strategy |
| Slow recovery | Large state or slow checkpoint restore | Review state backend and checkpoint storage |
| Sink writes fail | Connectivity or destination limits | Check sink configuration and retry behavior |

---

## 20. Best Practices

1. Choose Flink when its continuous streaming and stateful-processing capabilities fit the workload.
2. Use event time when business logic depends on when events actually occurred.
3. Configure watermark strategies based on expected out-of-order delays.
4. Enable checkpointing for fault-tolerant applications.
5. Store checkpoints in durable, accessible storage.
6. Use keyed state carefully and monitor its size.
7. Select parallelism based on workload and available resources.
8. Monitor backpressure, latency, throughput, and checkpoint duration.
9. Make external sink writes idempotent or transactional where required.
10. Test recovery behavior before production deployment.
11. Handle late events according to business requirements.
12. Avoid unnecessarily large or unbounded state.

---

## 21. Common Interview Questions and Answers

### Q1. What is Apache Flink?

Apache Flink is a distributed processing framework designed for stateful computations over bounded and unbounded data streams.

### Q2. Is Flink a message broker?

No. Flink is a processing engine. Kafka is commonly used as a message broker and event-streaming platform.

### Q3. Explain Flink architecture.

Flink uses JobManagers to coordinate execution and TaskManagers to execute tasks. TaskManagers provide task slots for distributed execution.

### Q4. What is a JobManager?

It coordinates job scheduling, execution, checkpointing, and recovery-related operations.

### Q5. What is a TaskManager?

It is a worker process that executes assigned tasks and manages their runtime resources.

### Q6. What is a task slot?

A task slot is a unit of resource allocation within a TaskManager. It is not necessarily equivalent to one CPU core.

### Q7. What is the difference between a Job and a Task?

A Job represents the submitted Flink application. Tasks are execution units created from the application's operators.

### Q8. What is stateful processing?

Processing that maintains information across multiple records, such as a customer's running spending total.

### Q9. What is keyed state?

State associated with a key, such as maintaining separate spending totals for each customer.

### Q10. What is operator state?

State associated with an operator instance rather than an individual key.

### Q11. What is KeyBy?

`keyBy()` partitions a stream by key so that records with the same key are processed together for keyed operations.

### Q12. What is a watermark?

A watermark indicates event-time progress and helps Flink manage event-time windows and late-arriving events.

### Q13. What is checkpointing?

Checkpointing periodically captures consistent application state so Flink can recover after failures.

### Q14. What is the difference between checkpoint and savepoint?

Checkpoints primarily support automatic fault recovery. Savepoints are generally used for controlled operations such as upgrades and migrations.

### Q15. Does checkpointing guarantee exactly-once processing?

Not by itself for every external system. End-to-end exactly-once behavior depends on the source, checkpointing, and sink guarantees.

### Q16. What is backpressure?

Backpressure occurs when downstream operators cannot keep up with upstream data production.

### Q17. What is parallelism?

Parallelism determines how many parallel instances execute an operator.

### Q18. What is the difference between Flink and Spark Structured Streaming?

Flink has a native stream-processing runtime with continuous streaming capabilities. Spark Structured Streaming commonly uses micro-batch execution and integrates closely with the Spark ecosystem.

### Q19. Can Flink process batch data?

Yes. Flink supports bounded as well as unbounded data processing.

### Q20. How would you design a real-time fraud detection pipeline?

I would ingest transaction events from Kafka, validate and key them by customer or account, maintain state for recent activity, evaluate fraud rules, and publish suspicious transactions to an alerting system. I would configure event-time handling, checkpointing, and reliable sink writes.

---

## 22. Common Mistakes to Avoid

1. **Confusing Flink with Kafka:** Kafka stores and distributes events; Flink processes them.
2. **Confusing JobManager and TaskManager:** JobManager coordinates; TaskManager executes.
3. **Assuming one task slot equals one CPU core:** Slots are resource-allocation units.
4. **Confusing stateful and stateless processing:** Stateful operations retain information across records.
5. **Assuming checkpointing automatically guarantees end-to-end exactly-once:** Sink behavior also matters.
6. **Confusing checkpoint and savepoint:** Their operational purposes and lifecycle differ.
7. **Assuming watermarks use wall-clock time:** Watermarks track event-time progress.
8. **Ignoring backpressure:** A slow sink can affect the whole pipeline.
9. **Assuming increasing parallelism always improves performance:** Skew, resources, and overhead can limit gains.
10. **Assuming all late events are automatically accepted:** Late-event handling depends on watermark and operator configuration.

---

## 23. Quick Revision

| Concept | Meaning |
|---|---|
| Apache Flink | Distributed stream-processing framework |
| JobManager | Coordinates job execution |
| TaskManager | Executes tasks |
| Task Slot | Resource allocation unit |
| Job | Submitted Flink application |
| Task | Execution unit |
| Operator | Performs processing logic |
| DataStream API | Programmatic stream-processing API |
| Flink SQL | SQL-based processing |
| Stateful Processing | Maintains information across events |
| Keyed State | State associated with keys |
| Operator State | State associated with operator instances |
| KeyBy | Partitions records by key |
| Watermark | Tracks event-time progress |
| Window | Groups events over a time range |
| Checkpoint | Captures state for recovery |
| Savepoint | Supports controlled operational actions |
| Backpressure | Downstream processing bottleneck |
| Parallelism | Number of parallel operator instances |

### Remember these five points

1. Flink is a distributed stream-processing framework.
2. JobManager coordinates; TaskManagers execute.
3. Stateful processing maintains information across events.
4. Watermarks support event-time processing and late-data handling.
5. Checkpointing supports fault recovery, while end-to-end exactly-once depends on the full pipeline.

---

## 24. Self-Assessment

Try answering these questions without looking at the notes.

1. What is Apache Flink, and why is it used?
2. Explain Flink architecture.
3. What is the difference between JobManager and TaskManager?
4. What is a task slot?
5. Explain the Source → Transformation → Sink flow.
6. What is the difference between stateful and stateless processing?
7. Explain keyed state and operator state.
8. What is the purpose of `keyBy()`?
9. What are watermarks?
10. Explain tumbling, sliding, and session windows.
11. What is checkpointing?
12. What is the difference between checkpoints and savepoints?
13. Does checkpointing alone guarantee exactly-once delivery?
14. What is backpressure?
15. How does parallelism improve processing?
16. Compare Flink with Spark Structured Streaming.
17. Compare Flink with Kafka.
18. Design a real-time e-commerce order-processing pipeline using Kafka and Flink.

**Interview readiness:** You should be able to explain Flink architecture, state management, event-time processing, checkpointing, and how Flink works with Kafka in a real-time data engineering pipeline.

