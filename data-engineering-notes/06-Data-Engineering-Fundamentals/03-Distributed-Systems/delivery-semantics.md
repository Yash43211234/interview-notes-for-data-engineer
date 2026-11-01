# Delivery Semantics in Distributed Systems

## 1. Introduction

In data engineering, **delivery semantics describe how many times a message or event may be delivered or processed when a system handles failures, retries, and recovery.**

They are important in:
- Apache Kafka
- Apache Spark Structured Streaming
- Apache Flink
- Event-driven architectures
- Real-time data pipelines
- Distributed message processing
- Database and data lake ingestion

Imagine an e-commerce application generates an order event. A processing system reads the event, calculates revenue, and writes the result to a table.

What happens if the system crashes after writing the result but before acknowledging the event?

When the system restarts, it may process the same event again. Depending on the delivery and processing design, the event may be duplicated or its result may be safely committed.

Delivery semantics help us understand and manage these situations.

## 2. The Three Main Delivery Semantics

There are three commonly discussed delivery guarantees:

1. At-most-once
2. At-least-once
3. Exactly-once

### 2.1 At-Most-Once

**An event is processed zero or one time. It is not retried after certain failures, so events may be lost.**

If a failure occurs before the event is successfully processed, the system may move on without processing it again.

Example:

```text
Event received
      |
      v
Position acknowledged
      |
      v
Processing fails
      |
      v
Event is not retried
      |
      v
Event may be lost
```

Advantages:
- Low processing overhead in some designs.
- Avoids duplicate processing caused by retries.

Disadvantages:
- Events can be lost.
- Unsuitable when every transaction must be preserved.

Use cases may include applications where occasional data loss is acceptable and low overhead is more important than completeness.

### 2.2 At-Least-Once

**An event is retried until successful processing or acknowledgment is achieved, according to the system's retry policy. An event may be processed more than once.**

Example:

```text
Event received
      |
      v
Process event
      |
      v
Write result successfully
      |
      v
Acknowledgment fails
      |
      v
Event is delivered again
      |
      v
Duplicate processing is possible
```

Advantages:
- Helps prevent data loss caused by transient failures.
- Supports retry-based recovery.

Disadvantages:
- Duplicate processing is possible.
- The destination must handle retries safely when duplicate effects are unacceptable.

Common use cases:
- Data ingestion.
- Message processing.
- Retryable ETL pipelines.
- Systems that prioritize not losing events.

### 2.3 Exactly-Once

**Exactly-once semantics aim to ensure that each logical event has one effective committed result, even when failures and retries occur.**

A system may internally execute an operation more than once, but duplicate effects must be prevented or transactionally controlled.

Example:

```text
Event received
      |
      v
Process event
      |
      v
Commit result safely
      |
      v
Failure or retry occurs
      |
      v
Duplicate effect is prevented
      |
      v
One effective committed result
```

Advantages:
- Prevents duplicate committed effects within the supported guarantee.
- Useful for financial calculations, order processing, and reliable analytical pipelines.

Challenges:
- Requires appropriate coordination between source progress and destination writes.
- May require transactions, deduplication, idempotent operations, or supported connector guarantees.
- End-to-end exactly-once behavior depends on the source, processing engine, and sink.

**Important:** Exactly-once does not necessarily mean the code executes physically only once. It means duplicate execution does not create duplicate committed effects within the guarantee's scope.

---

## 3. Comparison of Delivery Semantics

| Feature | At-Most-Once | At-Least-Once | Exactly-Once |
|---|---|---|---|
| Processing attempts | Zero or one | One or more | May execute more than once |
| Data loss | Possible | Generally avoided through retries, subject to system guarantees | Prevented within the supported guarantee |
| Duplicate effects | Avoided through no retry in the relevant failure case | Possible | Prevented within the guarantee |
| Retry behavior | Limited or absent | Retries are allowed | Retries may occur with duplicate-effect protection |
| Complexity | Often lower | Moderate | Often higher |
| Typical concern | Missing events | Duplicate events | Correct transactional or deduplication behavior |

The actual guarantees depend on system configuration, acknowledgment timing, persistence, and failure handling.

## 4. Delivery vs Processing vs Effect

These terms are related, but they are not identical.

- **Delivery:** How an event reaches a consumer.
- **Processing:** How the consumer handles the event.
- **Effect:** The committed change made by processing, such as inserting a row or increasing a total.

An event can be delivered more than once but still produce only one effective database change if the destination uses a suitable idempotency key or transaction.

For example, an order event might be delivered twice, but the destination can use `order_id` to prevent the same order from being counted twice.

This distinction is important when discussing exactly-once guarantees.

## 5. What Is Idempotency?

**An operation is idempotent when repeating it under the defined conditions produces the same final state as performing it once.**

Consider a customer order with `order_id = 501`.

A pipeline receives the same event twice.

### Non-Idempotent Operation

```sql
UPDATE daily_revenue
SET revenue = revenue + 500
WHERE business_date = '2026-01-10';
```

If the same order is processed twice, revenue may increase by 1,000 instead of 500.

### Idempotent Approach

Instead of blindly adding the amount, store each order once using a stable business key, then calculate revenue from the unique order records.

Conceptually:

```text
Order 501 arrives
      |
      v
Check destination by order_id
      |
      v
Order does not exist
      |
      v
Insert order 501

Same event arrives again
      |
      v
Order 501 already exists
      |
      v
Do not insert a duplicate
```

The exact implementation may use a unique constraint, transactional insert, deduplication, or an appropriate `MERGE` operation.

### Idempotency vs Exactly-Once

| Idempotency | Exactly-Once |
|---|---|
| A property of an operation or workflow | A delivery or processing guarantee |
| Makes safe retries possible | Aims for one effective committed result |
| Can be implemented with keys and deterministic writes | Often relies on transactions, idempotency, or coordinated progress tracking |
| Does not guarantee an event is delivered | Does not automatically make arbitrary external side effects safe |

Idempotency is a key technique for building reliable pipelines, but it does not by itself guarantee that every event will be delivered.

## 6. Retries and Acknowledgments

A distributed system often uses acknowledgments to indicate that a message or processing step has succeeded.

A typical message workflow:

```text
Producer
   |
   v
Message Broker
   |
   v
Consumer reads message
   |
   v
Consumer processes message
   |
   v
Consumer writes result
   |
   v
Consumer acknowledges message
```

The order of writing and acknowledgment matters.

### Scenario A: Acknowledge Before Writing

```text
Read event
   |
   v
Acknowledge event
   |
   v
Write result fails
```

If the system does not redeliver the acknowledged event, the result may be lost.

This can lead to at-most-once behavior for that failure scenario.

### Scenario B: Write Before Acknowledging

```text
Read event
   |
   v
Write result successfully
   |
   v
Acknowledgment fails
   |
   v
Event is redelivered
```

The event may be processed again, creating duplicate effects unless the destination handles retries safely.

This is a common source of at-least-once behavior.

### Key Lesson

There is often a gap between committing a destination write and recording source progress. Reliable designs must handle failures in that gap.

Transactions or coordinated source-and-sink commits can help, where supported. Otherwise, idempotent writes and deduplication are common approaches.

## 7. Delivery Semantics in Apache Kafka

Kafka provides mechanisms for reliable message production and consumption, but its guarantees depend on configuration and processing design.

### 7.1 Producer Delivery

A Kafka producer sends records to a topic.

Important settings include:

- `acks`
- `enable.idempotence`
- Retry configuration

The producer's acknowledgment and retry behavior affect durability and duplicate handling.

Kafka producer idempotence can prevent duplicate log entries caused by producer retries within its supported scope. It does not automatically make arbitrary downstream database operations exactly once.

### 7.2 Consumer Processing

A Kafka consumer reads records and tracks its position using offsets.

An offset indicates progress through a partition.

For example:

```text
Topic partition:

Offset 100 → Order A
Offset 101 → Order B
Offset 102 → Order C
```

If the consumer has committed offset 101, it generally resumes from the next offset according to the consumer's offset-management behavior.

### 7.3 Offset Commit Timing

**Commit progress before processing completes:**

- A crash can cause records to be skipped after restart.
- This can lead to lost processing results.

**Commit progress after processing completes:**

- A crash between the destination write and offset commit can cause records to be processed again.
- Duplicate destination effects are possible.

Kafka supports transactions and idempotent producers for certain workflows, including Kafka-to-Kafka processing with transactional offset handling. Exactly-once behavior for external databases or APIs requires additional coordination or idempotent sink design.

## 8. Delivery Semantics in Spark Structured Streaming

Spark Structured Streaming supports fault-tolerant stream processing using checkpointing and source/sink-specific mechanisms.

### 8.1 Checkpointing

A checkpoint stores information needed to recover a streaming query, such as source progress and state for supported operations.

Example:

```python
query = (
    events.writeStream
    .format("delta")
    .option(
        "checkpointLocation",
        "/checkpoints/orders"
    )
    .outputMode("append")
    .toTable("sales.processed_orders")
)
```

The checkpoint location must be stable and accessible to the query after restart.

For a given query, preserve its checkpoint across restarts when recovery is intended. Do not casually delete or reuse the same checkpoint location for an unrelated query.

### 8.2 Does Checkpointing Guarantee Exactly-Once?

**No, checkpointing alone does not guarantee exactly-once effects in every destination.**

It helps Spark recover source progress and state, but the effective delivery guarantee also depends on:
- Source replay and recovery support.
- Sink commit behavior.
- Output mode and query design.
- External side effects.
- Whether writes are transactional or idempotent.

For example, a custom `foreachBatch` operation that sends an email or makes a non-idempotent API call can repeat the side effect after a failure, even if the streaming query uses checkpoints.

### 8.3 Delta Lake Sink

Delta Lake supports transactional table writes and integrates with Structured Streaming for reliable table updates.

For supported streaming writes, Spark and Delta coordinate progress and commits to help prevent duplicate committed table results during recovery.

However, do not generalize this to arbitrary side effects performed alongside the Delta write. Each external destination needs its own reliability strategy.

### 8.4 `foreachBatch`

`foreachBatch` runs a function for each micro-batch.

Example:

```python
def process_batch(batch_df, batch_id):
    batch_df.write \
        .format("delta") \
        .mode("append") \
        .saveAsTable("sales.orders")

query = (
    events.writeStream
    .foreachBatch(process_batch)
    .option(
        "checkpointLocation",
        "/checkpoints/orders"
    )
    .start()
)
```

This example illustrates the structure of `foreachBatch`, but an append-only write should not automatically be assumed to be idempotent across retries.

If a batch is retried, duplicate rows may be written unless the sink operation is designed to handle it safely. A robust implementation may use a deterministic batch identifier, transactional deduplication, or a suitable `MERGE` strategy.

## 9. Exactly-Once in End-to-End Pipelines

An end-to-end pipeline may contain several components:

```text
Source
  |
  v
Message Broker
  |
  v
Processing Engine
  |
  v
Destination Table
  |
  v
External API
```

Even if one component supports exactly-once behavior, the entire pipeline does not automatically inherit that guarantee.

For example:
- Kafka can transactionally write records to Kafka topics.
- Spark can recover streaming progress from checkpoints.
- Delta Lake can commit table changes transactionally.
- An external REST API may not support idempotency or distributed transactions.

Therefore, end-to-end guarantees depend on the combined behavior of all components.

### Common Reliability Techniques

1. Use durable source storage.
2. Enable suitable acknowledgment and retry settings.
3. Store progress only after the appropriate work succeeds.
4. Use stable event IDs or business keys.
5. Make writes idempotent where possible.
6. Use transactional sinks where supported.
7. Handle duplicate and out-of-order events.
8. Monitor failed batches and recovery behavior.
9. Test failures between source progress and destination commits.

## 10. Deduplication in Data Pipelines

Deduplication removes repeated records based on a defined key or business rule.

Example input:

| event_id | order_id | amount |
|---|---:|---:|
| E101 | 501 | 500 |
| E101 | 501 | 500 |
| E102 | 502 | 700 |

If `event_id` uniquely identifies an event, the repeated `E101` may be a duplicate.

A simple PySpark example:

```python
deduplicated_df = df.dropDuplicates(["event_id"])
```

This removes duplicates based on `event_id`, but it does not guarantee which row is retained if duplicate keys contain different values.

If conflicting versions are possible, define a deterministic rule, such as selecting the record with the latest event timestamp and a stable tie-breaker.

### Important Difference

- **Transport-level duplicate prevention** deals with repeated messages in the delivery system.
- **Business-level deduplication** determines whether two records represent the same logical event or entity.

A system may require both.

## 11. Common Failure Scenarios

| Failure Scenario | Risk | Possible Solution |
|---|---|---|
| Consumer crashes before processing | Event may need replay | Durable source and retry |
| Destination write fails | Missing result | Retry safely |
| Destination succeeds but acknowledgment fails | Duplicate processing | Idempotent destination write |
| Source offset commits too early | Lost processing result | Commit progress after successful work |
| Streaming query restarts | Reprocessing of work may occur | Checkpoint recovery and safe sink writes |
| External API request times out after success | Client may retry an already completed request | API idempotency key, if supported |
| Two workers process the same event | Duplicate effects or race conditions | Unique constraints, transactions, or concurrency control |
| Late event arrives | Incorrect aggregate or missing update | Event-time handling and reconciliation |

No single technique solves every failure. Choose the solution based on the source, destination, and required guarantee.

## 12. Practical Example: Reliable Order Processing

Suppose an e-commerce company receives order events from Kafka and writes them to a Delta table.

### Requirements

- Do not lose valid order events.
- Avoid counting the same order twice.
- Recover after a processing failure.
- Support safe retries.

### Design

1. Use durable Kafka topics with suitable retention and producer settings.
2. Consume events using a fault-tolerant streaming query.
3. Configure a persistent checkpoint location.
4. Validate the order records.
5. Use a stable order or event ID.
6. Write through a transactional, idempotent strategy.
7. Deduplicate repeated business events where necessary.
8. Monitor processing progress, failures, and data quality.

### Example Flow

```text
Kafka Order Event
       |
       v
Spark Structured Streaming
       |
       v
Validation
       |
       v
Deduplication by Event ID
       |
       v
Transactional / Idempotent Write
       |
       v
Delta Table
```

### What If Spark Crashes?

Spark can use its checkpoint to recover progress and reprocess work as required by the source and query.

If the same event is encountered again, the destination's transactional or idempotent design should prevent a duplicate business effect.

**Interview point:** Checkpointing supports recovery; idempotent or transactional writes help control duplicate effects. Reliable processing often needs both.

## 13. Common Interview Questions and Answers

### Q1. What are delivery semantics?

Delivery semantics describe the guarantees a system provides about whether events may be lost, redelivered, or produce duplicate processing effects.

### Q2. What is at-most-once delivery?

An event is processed zero or one time. It avoids retry-driven duplicate processing in the relevant failure case, but events can be lost.

### Q3. What is at-least-once delivery?

The system can retry until processing or acknowledgment succeeds, according to its guarantees. An event may be processed more than once.

### Q4. What is exactly-once processing?

Exactly-once processing aims to ensure one effective committed result for each logical event within a defined scope, even if internal retries occur.

### Q5. Why can at-least-once processing produce duplicates?

A destination write may succeed while the acknowledgment or source-progress commit fails. The event can then be delivered again.

### Q6. Does exactly-once mean the code executes only once?

No. The code may run multiple times. Exactly-once concerns the effective committed result, not necessarily the number of physical execution attempts.

### Q7. What is idempotency?

An operation is idempotent if repeating it under defined conditions produces the same final state as executing it once.

### Q8. Is idempotency the same as exactly-once?

No. Idempotency makes retries safer, while exactly-once is a guarantee about the effective processing result. Idempotency alone does not guarantee delivery.

### Q9. Does Kafka support exactly-once semantics?

Kafka supports idempotent producers and transactions for defined workflows. End-to-end guarantees depend on how consumers, offsets, and destination systems are integrated.

### Q10. Does Spark checkpointing guarantee exactly-once?

Not for every pipeline. Checkpointing supports recovery of progress and state, but sink behavior and external side effects determine whether duplicate effects can occur.

### Q11. How can duplicate records be prevented in a data pipeline?

Use stable event IDs or business keys, deterministic deduplication, idempotent writes, unique constraints, or transactional operations where supported.

### Q12. What happens if a database write succeeds but the acknowledgment fails?

The event may be retried. If the database operation is not idempotent or transactionally protected, the retry can create a duplicate effect.

### Q13. What is the difference between delivery and processing semantics?

Delivery semantics concern how messages reach a consumer. Processing semantics concern what happens when the consumer handles them. A system may redeliver a message but still produce one effective result.

### Q14. How would you design an exactly-once-style order pipeline?

I would use a durable source, fault-tolerant progress tracking, stable event IDs, and transactional or idempotent destination writes. I would test failures between the destination commit and source-progress update.

## 14. Quick Revision

| Concept | Remember |
|---|---|
| At-most-once | No retry in the relevant case; events may be lost |
| At-least-once | Retries may cause duplicate processing |
| Exactly-once | One effective committed result within the guarantee |
| Idempotency | Repeated operation has the same final effect |
| Acknowledgment | Confirms receipt or successful processing, depending on the system |
| Offset | Tracks consumer progress in a partitioned log |
| Checkpoint | Stores progress and state needed for recovery |
| Deduplication | Identifies and removes repeated logical records |
| Transaction | Helps commit a group of changes atomically |
| End-to-end guarantee | Depends on all pipeline components |

### One-Minute Interview Answer

Delivery semantics describe how a distributed system handles events when failures and retries occur.

At-most-once means an event may be lost but is not retried in the relevant failure case. At-least-once allows retries, which reduce the risk of lost events but can create duplicates. Exactly-once aims to provide one effective committed result for each logical event within a defined scope.

In data engineering, idempotency is important because it allows an operation to be retried without creating duplicate effects. Kafka offers producer idempotence and transactional features for supported workflows. Spark Structured Streaming uses checkpoints to recover progress and state, but checkpoints alone do not guarantee exactly-once effects in arbitrary external systems.

For a reliable pipeline, I would combine durable sources, correct progress tracking, stable event IDs, and transactional or idempotent sink writes.

## 15. Self-Assessment

- [ ] Explain at-most-once delivery.
- [ ] Explain at-least-once delivery.
- [ ] Explain exactly-once semantics.
- [ ] Differentiate delivery, processing, and committed effects.
- [ ] Explain idempotency.
- [ ] Explain acknowledgment timing.
- [ ] Explain Kafka offsets and retry risks.
- [ ] Explain Spark Structured Streaming checkpoints.
- [ ] Explain why checkpoints alone are insufficient for arbitrary sinks.
- [ ] Explain how duplicate processing occurs.
- [ ] Explain how to design a retry-safe Delta pipeline.
- [ ] Describe the limitations of end-to-end exactly-once guarantees.

**Interview readiness:** Be able to explain what happens when a destination write succeeds but the source acknowledgment fails, and describe how idempotency or transactions prevent duplicate committed effects.