

# Kafka Delivery Guarantees

## 1. Introduction

Kafka delivery guarantees describe how records are delivered and processed when failures occur.

The three commonly discussed guarantees are:

1. **At-most-once:** A record may be lost, but it is not intentionally processed again.
2. **At-least-once:** A record is not skipped under the intended recovery design, but it may be processed more than once.
3. **Exactly-once:** The committed result of processing is reflected once, within the supported transactional boundary.

These guarantees are important when designing reliable data pipelines.

---

## 2. Understanding the Processing Flow

A typical Kafka data pipeline looks like this:

```text
Producer
   |
   v
Kafka Topic
   |
   v
Consumer
   |
   v
Processing
   |
   v
Destination
   |
   v
Offset Commit
```

Failures can occur at different stages:

- Producer sends a record but does not receive an acknowledgement.
- Broker receives a record but fails before the producer receives confirmation.
- Consumer reads a record but crashes before processing.
- Consumer processes a record but crashes before committing its offset.
- Destination write succeeds, but the offset commit fails.

Delivery guarantees define how these situations are handled.

---

## 3. At-Most-Once Delivery

### What is at-most-once delivery?

At-most-once means a record is processed zero or one time from the application's perspective.

Records are acknowledged or offsets are committed before processing is completed.

### Processing flow

```text
Read record
    |
    v
Commit offset
    |
    v
Process record
    |
    v
Write destination
```

### Failure scenario

Suppose a consumer reads offset 10.

1. The consumer commits offset 11.
2. It starts processing offset 10.
3. The application crashes before writing the output.
4. After restarting, it resumes from offset 11.

Offset 10 is skipped by the application.

### Advantages

- Avoids duplicate processing in this failure scenario.
- Can be useful when occasional data loss is acceptable.
- Can reduce recovery work.

### Disadvantages

- Records can be lost from the application's processing perspective.
- Unsuitable for critical financial or business transactions where every event must be accounted for.

**Interview point:** At-most-once prioritizes avoiding duplicate processing over ensuring every record is processed.

---

## 4. At-Least-Once Delivery

### What is at-least-once delivery?

At-least-once means records are processed before their offsets are committed.

If processing succeeds but the consumer crashes before committing its offset, Kafka can deliver the same records again.

### Processing flow

```text
Read record
    |
    v
Process record
    |
    v
Write destination
    |
    v
Commit offset
```

### Failure scenario

Suppose a consumer reads offset 10.

1. The consumer processes offset 10.
2. It successfully writes the output.
3. The consumer crashes before committing offset 11.
4. After restarting, it resumes from offset 10.
5. The same record is processed again.

This can create duplicate output if the destination write is not idempotent.

### Advantages

- Reduces the risk of skipping records.
- Useful for reliable data pipelines.
- Supports recovery after consumer failures.

### Disadvantages

- Duplicate processing is possible.
- Destination systems must handle retries safely.

**Interview point:** At-least-once is commonly used in data engineering because duplicate processing can often be handled through deduplication and idempotent writes.

---

## 5. Exactly-Once Processing

### What is exactly-once processing?

Exactly-once processing aims to ensure that each input record's effect appears once in the committed output, even when retries occur.

It requires more than simply committing Kafka offsets.

The application must coordinate processing, output writes, and progress tracking.

### Basic idea

```text
Read records
     |
     v
Process records
     |
     v
Write output
     |
     v
Commit progress and output atomically
```

The goal is to prevent:
- Lost output.
- Duplicate committed output.
- Inconsistent progress after failures.

### Example

Suppose an order event is processed.

```json
{
  "order_id": 101,
  "amount": 500,
  "status": "PLACED"
}
```

The goal is for the destination to reflect this order event once, even if the processing application retries.

### Important distinction

Exactly-once processing does not necessarily mean the code physically executes only once.

A record may be read or processed multiple times internally. The important requirement is that the committed result is reflected once within the supported transaction boundary.

---

## 6. Producer Delivery Guarantees

Kafka producers also have reliability settings.

Important producer configurations include:

- `acks`
- `enable.idempotence`
- `retries`
- `max.in.flight.requests.per.connection`

### A. `acks`

The `acks` configuration controls how many broker acknowledgements the producer waits for.

| Setting | Meaning |
|---|---|
| `acks=0` | Producer does not wait for broker acknowledgement |
| `acks=1` | Producer waits for acknowledgement from the partition leader |
| `acks=all` | Producer waits for acknowledgement from all current in-sync replicas |

#### `acks=0`

The producer sends records without waiting for a broker acknowledgement.

This provides low acknowledgement overhead but offers weaker delivery assurance.

#### `acks=1`

The leader acknowledges the write after accepting it locally, without waiting for follower replicas to acknowledge it.

A leader failure before replication can result in data loss.

#### `acks=all`

The leader waits for the current ISR to acknowledge the write according to Kafka's replication protocol.

This provides stronger durability when combined with suitable replication and `min.insync.replicas` settings.

**Important:** `acks=all` does not mean every configured replica must acknowledge the record. It refers to the current ISR.

---

## 7. Producer Idempotence

### What is producer idempotence?

Producer idempotence prevents duplicate records from being written to Kafka due to certain producer retries.

For example:

1. Producer sends a record.
2. Broker writes the record.
3. Acknowledgement is lost.
4. Producer retries the request.
5. Kafka recognizes the retry and avoids appending a duplicate within the idempotent producer protocol.

### Configuration

```properties
enable.idempotence=true
acks=all
```

In modern Kafka clients, idempotence is enabled by default under compatible configurations. Explicit configuration can make the intended behavior clear.

### How does it work?

Kafka uses producer identity and sequence numbers to identify duplicate retry requests.

### Important points

- Producer idempotence handles duplicate writes caused by supported producer retries.
- It does not automatically deduplicate business events sent multiple times as separate application records.
- It does not guarantee exactly-once processing in an external database.
- Its guarantees are scoped to the producer protocol and partition behavior.

---

## 8. Kafka Transactions

Kafka supports transactions for workflows that need atomic writes across multiple Kafka partitions and topics, and for coordinating consumed offsets with produced Kafka records.

### Why are transactions useful?

Consider a stream-processing application:

```text
Input Topic
     |
     v
Consumer
     |
     v
Transformation
     |
     v
Output Topic
```

The application needs to:
1. Read input records.
2. Transform them.
3. Write output records.
4. Commit the input offsets.

If output records are committed but input offsets are not, the application may process the same input again.

If input offsets are committed but output records are not, output can be lost.

Kafka transactions allow the output records and consumed offsets to be committed atomically within Kafka.

### Transactional producer configuration

```properties
transactional.id=order-transformer-1
enable.idempotence=true
acks=all
```

The transactional ID must be stable and unique among concurrently active producer instances.

### Transactional workflow

Conceptually:

```text
Begin Transaction
       |
       v
Read Input Records
       |
       v
Transform Records
       |
       v
Write Output Records
       |
       v
Add Consumed Offsets to Transaction
       |
       v
Commit Transaction
```

If the transaction aborts, its output records are not visible to `read_committed` consumers.

### Important APIs

Common Kafka producer transaction methods include:

```java
producer.initTransactions();

producer.beginTransaction();

producer.send(record);

producer.sendOffsetsToTransaction(offsets, consumer.groupMetadata());

producer.commitTransaction();
```

For an aborted or failed transaction, the application can use:

```java
producer.abortTransaction();
```

Exact API signatures depend on the Kafka client version.

---

## 9. `read_committed` vs `read_uncommitted`

Consumers can control which transactional records they read.

### `read_uncommitted`

```properties
isolation.level=read_uncommitted
```

The consumer can read records from transactions that have not yet been committed, including records from transactions that are later aborted.

This is the default isolation level for standard Kafka consumers.

### `read_committed`

```properties
isolation.level=read_committed
```

The consumer reads only committed transactional records and non-transactional records.

Records from aborted transactions are not returned.

### Comparison

| `read_uncommitted` | `read_committed` |
|---|---|
| Can read uncommitted transactional records | Reads only committed transactional records |
| May expose records from aborted transactions | Hides records from aborted transactions |
| Default consumer isolation level | Used for transactional visibility |

**Important:** `read_committed` does not make an external database write transactional with Kafka. It controls visibility of records in Kafka.

---

## 10. Consumer Offset Commit and Delivery Guarantees

Offset commit timing is one of the most important factors in consumer delivery behavior.

### At-most-once

```text
Read → Commit Offset → Process
```

Possible result: record processing is skipped after a failure.

### At-least-once

```text
Read → Process → Write Output → Commit Offset
```

Possible result: record is processed again if the output succeeds but the commit fails.

### Exactly-once within Kafka

```text
Read Input
    |
    v
Process
    |
    v
Write Kafka Output
    |
    v
Commit Output + Input Offsets
       Atomically
```

Kafka transactions coordinate the output records and consumed offsets.

### Important interview distinction

A Kafka consumer's ordinary manual offset commit does not atomically commit an external database transaction.

For an external sink, additional design is required.

---

## 11. Exactly-Once with External Systems

Kafka transactions can coordinate Kafka input offsets and Kafka output records. They do not automatically include external databases, object storage, or Delta Lake in the same transaction.

Consider this architecture:

```text
Kafka
  |
  v
Consumer
  |
  v
Transformation
  |
  v
External Database
  |
  v
Commit Kafka Offset
```

### Failure scenario

1. Consumer reads an event.
2. Database write succeeds.
3. Consumer crashes before committing the Kafka offset.
4. Event is consumed again.
5. Database may receive a duplicate write.

### Common solutions

**1. Idempotent writes**

Use a stable event ID or business key to prevent duplicate effects.

**2. Deduplication**

Remove duplicate events using a stable identifier and appropriate business rules.

**3. Upsert or MERGE**

Use a deterministic key to insert new records or update existing records.

**4. Transactional coordination**

Where supported, coordinate progress and destination changes using an appropriate transaction or outbox/inbox pattern.

**5. Reconciliation**

Compare source and destination counts or business totals to identify missing or duplicated results.

### Example: Delta Lake MERGE

```sql
MERGE INTO target_orders AS target
USING incoming_orders AS source
ON target.order_id = source.order_id

WHEN MATCHED THEN
  UPDATE SET
    target.amount = source.amount,
    target.status = source.status

WHEN NOT MATCHED THEN
  INSERT (order_id, amount, status)
  VALUES (source.order_id, source.amount, source.status);
```

The incoming source should be deduplicated by `order_id` before the merge.

Also ensure that update logic is deterministic. A merge that increments a value on every retry may still produce duplicate effects.

**Interview point:** For external systems, exactly-once effective results are often achieved through idempotency, deduplication, transactional coordination, and reliable progress tracking.

---

## 12. Kafka Delivery Guarantees vs Spark Structured Streaming

Spark Structured Streaming can consume Kafka records and process them using micro-batches or supported continuous processing modes.

Its recovery mechanism differs from a standard Kafka consumer application.

### Standard Kafka consumer

- Uses consumer group membership and partition assignment.
- Commonly commits offsets to `__consumer_offsets`.
- Uses committed offsets to resume consumption.

### Spark Structured Streaming

- Tracks Kafka source offsets in the Spark checkpoint.
- Uses checkpointed progress to recover the query.
- Does not rely on ordinary Kafka consumer-group committed offsets for its source recovery.
- Can use transactional or idempotent sink designs depending on the destination.

### Example

```python
df = (
    spark.readStream
        .format("kafka")
        .option("kafka.bootstrap.servers", "broker1:9092")
        .option("subscribe", "orders")
        .option("startingOffsets", "earliest")
        .load()
)
```

The checkpoint is configured when writing the streaming output:

```python
query = (
    transformed_df.writeStream
        .format("delta")
        .option("checkpointLocation", "/checkpoints/orders")
        .outputMode("append")
        .start("/data/orders")
)
```

### Important points

- `startingOffsets` applies when starting a new query without an existing checkpoint.
- On recovery, the checkpoint determines the query's source progress.
- Deleting or replacing the checkpoint can cause the query to start with different progress behavior.
- Checkpointing alone does not make every arbitrary external sink exactly once.
- The sink's commit and retry behavior also matters.

---

## 13. Common Failure Scenarios

| Failure | Possible outcome | Recommended approach |
|---|---|---|
| Producer acknowledgement lost | Producer may retry | Enable idempotence |
| Broker fails after accepting a write | Data may be at risk depending on replication and acknowledgement settings | Replication, `acks=all`, suitable minimum ISR |
| Consumer crashes before processing | Record can be read again | Reliable offset recovery |
| Consumer crashes after output write but before offset commit | Duplicate output possible | Idempotent sink |
| Consumer commits before output write | Processing may be skipped | Commit after successful processing |
| Transaction aborts | Transactional output is not visible to `read_committed` consumers | Retry or recover transaction safely |
| Spark query restarts | Source progress is recovered from checkpoint | Preserve checkpoint and use a safe sink |

---

## 14. Best Practices

### Producer

- Use idempotence for reliable retry behavior.
- Use `acks=all` when stronger durability is required.
- Configure replication factor appropriately.
- Set `min.insync.replicas` according to availability and durability requirements.
- Use stable transactional IDs for transactional producers.
- Handle producer errors and retries.

### Consumer

- Commit offsets only at the correct processing boundary.
- Use manual commits when precise control is needed.
- Handle rebalances correctly.
- Monitor consumer lag.
- Make processing safe for retries.
- Avoid blocking poll loops for too long.

### Data pipeline

- Use stable event IDs.
- Deduplicate incoming events.
- Design idempotent writes.
- Preserve streaming checkpoints.
- Monitor failed batches and data quality.
- Reconcile source and destination data.
- Test failure and recovery scenarios.

---

## 15. Common Interview Questions and Answers

### Q1. What are Kafka delivery guarantees?

At-most-once, at-least-once, and exactly-once processing.

### Q2. What is at-most-once delivery?

A record is processed zero or one time. Committing progress before processing can cause records to be skipped after a failure.

### Q3. What is at-least-once delivery?

A record is processed before its offset is committed. A failure can cause the same record to be processed again.

### Q4. What is exactly-once processing?

It ensures the committed processing result is reflected once within a supported transactional boundary.

### Q5. How can producer retries create duplicates?

If a broker accepts a record but its acknowledgement is lost, the producer may retry. Without idempotence, the retry can append another copy.

### Q6. What is producer idempotence?

A Kafka producer feature that prevents duplicate writes caused by supported retries, using producer identity and sequence numbers.

### Q7. What does `acks=all` mean?

The leader waits for acknowledgement from the current in-sync replicas according to Kafka's replication protocol.

### Q8. What is `min.insync.replicas`?

The minimum ISR size required for a write when the producer uses `acks=all`.

### Q9. What is a Kafka transaction?

A mechanism that atomically commits or aborts a group of Kafka writes and can include consumed offsets.

### Q10. What is the difference between `read_committed` and `read_uncommitted`?

`read_committed` hides aborted and uncommitted transactional records. `read_uncommitted` can expose them.

### Q11. Does Kafka guarantee exactly-once writes to an external database?

No. External sinks need appropriate transaction coordination or idempotent processing.

### Q12. How do you avoid duplicate records in Delta Lake?

Use stable event or business keys, deduplicate incoming records, and use deterministic idempotent writes such as MERGE.

### Q13. Why is committing offsets before processing risky?

If the consumer crashes after committing but before completing the write, the record may not be processed again.

### Q14. Why is committing offsets after processing risky?

If the output write succeeds but the offset commit fails, the record can be processed again.

### Q15. Does Spark Structured Streaming use Kafka consumer offsets for recovery?

It tracks Kafka source progress in its own checkpoint rather than relying on ordinary consumer-group committed offsets.

### Q16. Does a Spark checkpoint guarantee exactly-once writes to every sink?

No. The sink's transaction and retry behavior must also support the required guarantee.

---

## 16. Common Mistakes to Avoid

1. **Confusing delivery with execution:** Exactly-once does not mean application code physically runs only once.
2. **Assuming producer idempotence solves every duplicate:** It handles supported producer retries, not separate duplicate business events.
3. **Assuming `acks=all` means all configured replicas:** It refers to the current ISR.
4. **Assuming Kafka transactions cover external databases:** Kafka transactions have a Kafka-scoped transactional boundary.
5. **Committing offsets before processing:** This can cause lost processing.
6. **Committing offsets after processing without idempotency:** This can cause duplicate effects.
7. **Assuming checkpoints alone guarantee exactly-once:** The sink must also support safe commits and retries.
8. **Ignoring aborted transactions:** Consumers that use `read_uncommitted` can see records from aborted transactions.
9. **Assuming MERGE is always idempotent:** The source and update logic must be deterministic and properly deduplicated.
10. **Treating at-least-once as exactly-once:** At-least-once permits duplicate processing.

---

## 17. Quick Revision

| Concept | Remember |
|---|---|
| At-most-once | Possible loss; avoids repeat processing in the described commit pattern |
| At-least-once | Possible duplicates |
| Exactly-once | One committed effect within the supported boundary |
| `acks=0` | No broker acknowledgement |
| `acks=1` | Leader acknowledgement |
| `acks=all` | Current ISR acknowledgement |
| Producer idempotence | Prevents duplicates from supported producer retries |
| Kafka transactions | Atomic Kafka writes and consumed-offset commits |
| `read_uncommitted` | Can read uncommitted transactional records |
| `read_committed` | Reads only committed transactional records |
| Manual commit | Application controls offset progress |
| Idempotency | Makes repeated writes safe under defined conditions |
| Spark checkpoint | Tracks streaming source progress and state |

### Remember these five points

1. At-most-once can lose processing; at-least-once can duplicate processing.
2. Producer idempotence protects against duplicates caused by supported producer retries.
3. Kafka transactions coordinate Kafka output and input offsets atomically.
4. External systems need their own transaction or idempotency strategy.
5. Spark Structured Streaming uses checkpoints for source progress and recovery.

---

## 18. Self-Assessment

Try answering these without looking at the notes.

1. Explain at-most-once, at-least-once, and exactly-once.
2. What happens if a consumer commits an offset before writing the output?
3. What happens if output succeeds but the offset commit fails?
4. How does producer idempotence work?
5. What is the difference between `acks=1` and `acks=all`?
6. How does `min.insync.replicas` affect writes?
7. What problem do Kafka transactions solve?
8. What is the difference between `read_committed` and `read_uncommitted`?
9. Can Kafka transactions guarantee exactly-once writes to an external database?
10. How would you prevent duplicate records in a Delta Lake pipeline?
11. How does Spark Structured Streaming recover Kafka source progress?
12. Design a pipeline that safely handles consumer crashes and retries.

**Interview readiness:** You should be able to explain a failure where the destination write succeeds but the Kafka offset commit fails, and show how idempotency or transactional coordination prevents duplicate committed results.

