# Exactly-Once Processing and Delivery Semantics in Spark

## 1. What Are Delivery Semantics?

Delivery semantics describe how a distributed data processing system handles messages or records when failures, retries, or restarts occur.

In data engineering, a record may be:
- Processed once.
- Processed more than once.
- Lost before processing.

Three important delivery semantics are:

1. At-most-once.
2. At-least-once.
3. Exactly-once.

**Key idea:** Delivery semantics determine whether records can be lost, duplicated, or processed without duplicate effects.

---

## 2. At-Most-Once Semantics

At-most-once means a record is processed zero or one time.

If a failure occurs, the record may be lost rather than processed again.

### Example

Imagine a streaming application receives 1,000 records.

During processing:
- 950 records are processed successfully.
- The application fails before processing the remaining 50.
- Those 50 records are not retried.

The result is that some records may be lost.

### Characteristics

- No duplicate processing due to retries.
- Records may be lost.
- Simpler processing in some systems.
- Not suitable when every record must be processed.

**Remember:** At-most-once = No duplicates from retries, but data loss is possible.

---

## 3. At-Least-Once Semantics

At-least-once means a record is processed one or more times.

The system retries processing when it cannot confirm whether an earlier attempt succeeded.

This prevents data loss under the system's recovery guarantees, but duplicate processing can occur.

### Example

Imagine a streaming application processes a payment record.

1. The record is processed.
2. The application fails before confirming successful processing.
3. The record is sent or processed again after recovery.
4. The same payment may be processed twice if the destination does not prevent duplicates.

### Characteristics

- Records are retried when necessary.
- Duplicate processing is possible.
- Requires duplicate handling at the destination when duplicates are unacceptable.

**Remember:** At-least-once = Avoid losing records, but duplicates are possible.

---

## 4. Exactly-Once Semantics

Exactly-once means each record has one effective result in the final system, even if processing is retried internally.

The important distinction is between:
- Processing a record internally.
- Applying its effect to the destination.

A system may process a record multiple times internally but still produce exactly one committed result.

### Example

Imagine a streaming application receives a payment event:

```text
payment_id = 101
amount = 500
```

The application processes the event, but a failure occurs during the write.

After recovery, the event may be processed again.

With suitable transactional or idempotent mechanisms, the destination recognizes that the event has already been committed and avoids applying the same effect twice.

### Characteristics

- Prevents duplicate committed effects.
- Requires appropriate coordination between processing and the destination.
- Depends on source, processing engine, and sink guarantees.
- Usually requires checkpointing, transactional commits, or idempotent writes.

**Remember:** Exactly-once = One effective committed result, even when retries happen.

---

## 5. Comparison of Delivery Semantics

| Feature | At-Most-Once | At-Least-Once | Exactly-Once |
|---|---|---|---|
| Processing attempts | Zero or one | One or more | May be retried |
| Data loss | Possible | Avoided under delivery guarantees | Avoided under end-to-end guarantees |
| Duplicates | No retry duplicates | Possible | No duplicate committed effects |
| Reliability | Lower | Higher | Strongest when end-to-end guarantees are supported |
| Complexity | Usually simpler | Moderate | More complex |
| Common concern | Lost records | Duplicate records | Transactional consistency |

---

## 6. What Is the Difference Between Processing and Delivery?

These terms are related but not identical.

**Delivery semantics** describe how records are delivered from a source or transport system.

**Processing semantics** describe how the processing application handles those records.

For example:

```text
Source
  |
  v
Message Delivery
  |
  v
Spark Processing
  |
  v
Destination
```

Even if a source delivers records reliably, the destination may still receive duplicate writes.

Therefore, exactly-once behavior must be considered across the entire pipeline.

**Important:** Exactly-once processing inside Spark does not automatically guarantee exactly-once results in every external database or API.

---

## 7. How Does Spark Structured Streaming Handle Exactly-Once Processing?

Spark Structured Streaming uses several mechanisms to provide fault tolerance.

Important components include:

1. Checkpointing.
2. Source offsets.
3. Write-ahead progress tracking and commit logs.
4. Replay and recovery.
5. Sink commit or idempotency mechanisms.

### Basic execution flow

```text
Streaming Source
      |
      v
Read Records
      |
      v
Track Source Progress
      |
      v
Process Records
      |
      v
Write Results
      |
      v
Commit Progress
      |
      v
Checkpoint Information
```

Conceptually, Spark records processing progress and uses checkpoint information to recover after failures.

The exact commit protocol depends on the source, sink, and output mode.

### Example: Structured Streaming

```python
stream_df = (
    spark.readStream
         .format("kafka")
         .option("kafka.bootstrap.servers", "localhost:9092")
         .option("subscribe", "orders")
         .load()
)
```

Write the stream:

```python
query = (
    stream_df.writeStream
        .format("parquet")
        .option("path", "/data/output/orders")
        .option("checkpointLocation", "/data/checkpoints/orders")
        .start()
)
```

The checkpoint location stores information needed for recovery, including streaming progress and state information where applicable.

**Important:** A checkpoint alone does not guarantee exactly-once behavior for every sink. The sink must support the required commit or idempotency behavior.

---

## 8. What Is Checkpointing's Role in Exactly-Once Processing?

Checkpointing is important because it helps Spark recover the progress of a streaming query.

It can store:
- Source progress and offsets.
- Batch or epoch completion information.
- Stateful processing information.

When an application fails and restarts using the same valid checkpoint location, Spark can use the recorded progress to recover.

### Example

Suppose Spark processes these batches:

```text
Batch 0 → Completed
Batch 1 → Completed
Batch 2 → Processing
Batch 3 → Not Started
```

The application fails during Batch 2.

After restarting with the same checkpoint location, Spark uses its recorded progress and commit information to recover the query.

The exact replay and commit behavior depends on the source and sink.

**Remember:** Checkpointing supports recovery, but it is not by itself a complete exactly-once guarantee.

---

## 9. What Is Idempotency?

Idempotency means performing the same operation multiple times produces the same final result as performing it once.

It is one of the most important concepts in reliable data pipelines.

### Example: Non-idempotent operation

Suppose an account has ₹1,000.

A payment event says to deduct ₹100.

```sql
UPDATE accounts
SET balance = balance - 100
WHERE account_id = 1;
```

If the same event is processed twice:

```text
Initial balance = ₹1,000
After first execution = ₹900
After second execution = ₹800
```

The duplicate event changes the result.

### Example: Idempotent approach

Instead of applying the same event repeatedly, use a unique transaction ID to ensure the transaction is applied only once.

Conceptually:

```text
Check transaction_id
       |
       v
Already processed?
   /       \
 Yes       No
  |         |
Skip     Apply update
            |
            v
       Record transaction_id
```

The transaction check and update should be performed atomically where possible.

### Why is idempotency important?

- Streaming applications may retry operations.
- Network failures can make commit status uncertain.
- Batch jobs may be rerun.
- API requests may be repeated.
- Data pipelines may process duplicate source records.

**Remember:** Idempotency makes retries safe.

---

## 10. Exactly-Once Processing with Kafka and Spark

Kafka and Spark Structured Streaming can work together to build fault-tolerant pipelines.

A simplified architecture:

```text
Kafka Topic
     |
     v
Spark Structured Streaming
     |
     v
Transformations
     |
     v
Checkpointing
     |
     v
Transactional / Idempotent Sink
```

### Important points

- Kafka stores records in topics and partitions.
- Spark tracks Kafka offsets for streaming progress.
- Spark checkpointing supports recovery.
- Spark can replay records after a failure.
- The destination must handle retries and duplicate writes correctly.

**Important:** Reading from Kafka with Spark Structured Streaming does not automatically make writes to an arbitrary external database exactly-once.

---

## 11. What Is the Role of Output Modes?

Structured Streaming supports different output modes.

| Output mode | Meaning |
|---|---|
| Append | Writes newly added result rows |
| Update | Writes rows updated since the previous trigger |
| Complete | Writes the entire result table |

The supported modes depend on the query and sink.

Output mode is not the same as delivery semantics.

For example, using Append mode does not automatically guarantee exactly-once writes to every destination.

---

## 12. Exactly-Once with Delta Lake

Delta Lake provides transactional writes using its transaction log.

It can support reliable, idempotent streaming writes when used with the appropriate streaming integration and configuration.

### Example

```python
query = (
    stream_df.writeStream
        .format("delta")
        .outputMode("append")
        .option("checkpointLocation", "/data/checkpoints/orders")
        .start("/data/delta/orders")
)
```

For supported Delta streaming write paths, Delta manages transactional commits so that committed streaming batches are not simply appended again after recovery.

### Important considerations

- Use a stable checkpoint location.
- Do not casually delete or replace checkpoint data.
- Use supported source and sink integrations.
- Understand how the destination handles retries.
- Avoid multiple writers that conflict over the same target without a suitable design.

**Remember:** Delta Lake transactions help provide exactly-once results for supported streaming write patterns, but they do not automatically guarantee exactly-once effects in unrelated external systems.

---

## 13. Common Failure Scenarios

| Failure scenario | Possible outcome | Protection |
|---|---|---|
| Application crashes during processing | Batch may need replay | Checkpointing and recovery |
| Network failure during write | Commit status may be uncertain | Transactional or idempotent sink |
| Duplicate source event | Same event may be processed again | Deduplication or unique event IDs |
| Executor failure | Lost task data may need recomputation | Spark fault tolerance and lineage |
| Checkpoint deleted | Previous progress may be unavailable | Protect checkpoint storage |
| External database write retried | Duplicate effects may occur | Idempotent writes or transactions |

---

## 14. Common Mistakes

1. Assuming exactly-once means a record is physically processed only once.
2. Confusing at-least-once with exactly-once.
3. Assuming checkpointing alone guarantees exactly-once output.
4. Ignoring the sink's commit behavior.
5. Assuming Kafka-to-Spark automatically guarantees exactly-once delivery to every destination.
6. Ignoring duplicate source events.
7. Using non-idempotent database updates in retryable pipelines.
8. Deleting checkpoint data without understanding the recovery impact.
9. Confusing output modes with delivery semantics.

---

## 15. Interview Questions

**Q1. What are delivery semantics?**

They describe how a distributed system handles record delivery and processing when failures or retries occur.

**Q2. What is at-most-once?**

A record is processed zero or one time. Data loss is possible.

**Q3. What is at-least-once?**

A record is processed one or more times. Duplicate processing is possible.

**Q4. What is exactly-once processing?**

It ensures each record produces one effective committed result, even if internal processing is retried.

**Q5. What is idempotency?**

Idempotency means repeating an operation produces the same final result as performing it once.

**Q6. Does Spark Structured Streaming support exactly-once processing?**

Structured Streaming provides fault-tolerant processing and recovery mechanisms. End-to-end exactly-once results depend on the source, query, and sink guarantees.

**Q7. What is the role of checkpointing in Structured Streaming?**

It stores progress and state information needed to recover a streaming query after failures.

**Q8. Does checkpointing alone guarantee exactly-once output?**

No. The destination must also support appropriate transactional or idempotent write behavior.

**Q9. How can you prevent duplicate records in a streaming pipeline?**

Use unique event IDs, deduplication, idempotent writes, or transactional sink mechanisms.

**Q10. What happens if a streaming application fails after writing data but before recording progress?**

The batch may be replayed after recovery. A transactional or idempotent sink is needed to prevent duplicate committed effects.

**Q11. What is the difference between processing once and committing once?**

A record may be processed multiple times internally, but transactional mechanisms can ensure its result is committed only once.

**Q12. How does Delta Lake help with exactly-once streaming writes?**

Delta Lake uses a transaction log and transactional commit mechanisms to support reliable, idempotent writes for supported streaming integrations.

---

## 16. Quick Revision

| Concept | Remember |
|---|---|
| At-most-once | Zero or one processing attempt; data loss possible |
| At-least-once | One or more attempts; duplicates possible |
| Exactly-once | One effective committed result |
| Checkpointing | Stores progress and supports recovery |
| Idempotency | Repeated operation produces the same final result |
| Source offsets | Track source progress |
| Transaction | Helps commit changes consistently |
| Sink | Destination where results are written |
| Deduplication | Removes repeated events based on a defined key |

### One-line revision

- At-most-once → Possible data loss.
- At-least-once → Possible duplicates.
- Exactly-once → No duplicate committed effects.
- Checkpointing → Recovery and progress tracking.
- Idempotency → Safe retries.
- Transactional sink → Consistent commits.

---

## 17. Self-Check

- [ ] Can I explain at-most-once delivery?
- [ ] Can I explain at-least-once delivery?
- [ ] Can I explain exactly-once processing?
- [ ] Can I differentiate processing attempts from committed results?
- [ ] Can I explain idempotency with an example?
- [ ] Can I explain checkpointing in Structured Streaming?
- [ ] Can I explain the role of source offsets?
- [ ] Can I explain why the sink matters?
- [ ] Can I describe how Kafka and Spark work together?
- [ ] Can I explain how Delta Lake supports transactional streaming writes?

**Final takeaway:** Exactly-once is an end-to-end reliability goal. Spark checkpointing helps recover processing progress, while transactional or idempotent sink behavior prevents retries from creating duplicate committed effects.