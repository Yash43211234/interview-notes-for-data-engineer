

# Kafka Consumer Groups and Offsets

## 1. Introduction

Kafka consumers read records from topics. Consumer groups allow multiple consumers to work together to process records in parallel.

Two important concepts are:

- **Consumer group:** A group of consumers that coordinate to consume topic partitions.
- **Offset:** The position of a record within a Kafka partition.

Together, they help Kafka distribute processing work and resume consumption after failures.

---

## 2. What Is a Kafka Consumer?

A consumer is an application that reads records from Kafka topics.

Example:

An e-commerce application publishes order events to the `orders` topic.

A consumer reads these events and sends them to a data warehouse or data lake.

```text
Producer
   |
   v
Kafka Topic: orders
   |
   v
Consumer
   |
   v
Data Processing
   |
   v
ADLS / Delta Lake
```

A consumer can:
- Subscribe to one or more topics.
- Read records from assigned partitions.
- Track its reading position using offsets.
- Commit offsets to record processing progress.
- Resume consumption after restarting.

---

## 3. What Is a Consumer Group?

A consumer group is a collection of consumers that cooperate to consume records from one or more topics.

Every consumer group has a unique `group.id`.

Example:

```text
Topic: orders
Partitions: 3

Consumer Group: order-processors
│
├── Consumer 1 → Partition 0
├── Consumer 2 → Partition 1
└── Consumer 3 → Partition 2
```

Each consumer processes its assigned partitions.

### Why use consumer groups?

Consumer groups provide:

1. Parallel processing.
2. Workload distribution.
3. Independent consumption by different applications.
4. Recovery after consumer failures.
5. Horizontal scaling.

### Example: Multiple consumer groups

Suppose the same `orders` topic is consumed by two applications.

```text
                    Topic: orders
                          |
              ┌───────────┴───────────┐
              |                       |
              v                       v
       Consumer Group A        Consumer Group B
       Order Processing        Analytics Pipeline
              |                       |
              v                       v
       Order Database           Data Warehouse
```

Both groups independently consume the same topic.

Each group maintains its own offsets.

**Important:** Consumer groups do not delete or exclusively own topic records. Multiple groups can read the same records independently.

---

## 4. Consumer-to-Partition Assignment

Kafka assigns partitions to consumers within a consumer group.

### Example 1: Equal number of consumers and partitions

```text
Partitions: 3
Consumers: 3

Consumer 1 → Partition 0
Consumer 2 → Partition 1
Consumer 3 → Partition 2
```

Each consumer gets one partition.

### Example 2: More consumers than partitions

```text
Partitions: 3
Consumers: 5

Consumer 1 → Partition 0
Consumer 2 → Partition 1
Consumer 3 → Partition 2
Consumer 4 → Idle
Consumer 5 → Idle
```

Two consumers remain idle because there are only three partitions.

### Example 3: Fewer consumers than partitions

```text
Partitions: 6
Consumers: 2

Consumer 1 → Partition 0, 1, 2
Consumer 2 → Partition 3, 4, 5
```

A consumer can process multiple partitions.

### Important rules

- A partition is assigned to at most one consumer within a consumer group at a time.
- A consumer can process multiple partitions.
- Different consumer groups can independently read the same partition.
- The number of partitions limits the maximum number of active consumers for that topic within a group.

**Interview point:** Increasing consumers beyond the number of partitions does not increase parallelism for that topic within the same group.

---

## 5. What Is a Kafka Offset?

An offset is a sequential position assigned to a record within a Kafka partition.

Example:

```text
Topic: orders
Partition: 0

Offset 0 → Order A
Offset 1 → Order B
Offset 2 → Order C
Offset 3 → Order D
Offset 4 → Order E
```

Offsets help consumers track which records they have processed or where they should resume reading.

### Important characteristics

- Every partition maintains its own offset sequence.
- Offsets are not globally unique across a topic.
- Offsets are not necessarily continuous among records currently visible to a consumer.
- An offset is not the same as a consumer group's committed offset.

A record can be identified using:

```text
Topic: orders
Partition: 0
Offset: 2
```

---

## 6. Current Position vs Committed Offset

These are important concepts for interviews.

### Current consumer position

The current position represents the next offset the consumer will read in a partition.

### Committed offset

The committed offset is the position saved by the consumer group so it can resume after a restart.

Example:

```text
Partition 0

Offset 0 → Processed
Offset 1 → Processed
Offset 2 → Processed
Offset 3 → Not processed
```

If the consumer has successfully processed offsets 0, 1, and 2, its next position is 3.

The committed offset is therefore typically:

```text
Committed offset = 3
```

Kafka stores the next offset to read, not the last successfully processed record's offset.

### Why is this important?

Suppose a consumer processes offset 2 but crashes before committing offset 3.

After restarting, it may read offset 2 again.

This can cause duplicate processing.

---

## 7. How Does Kafka Store Consumer Offsets?

Kafka stores committed consumer offsets in an internal topic:

```text
__consumer_offsets
```

This topic stores information such as:
- Consumer group identity.
- Topic and partition.
- Committed offset.
- Related offset metadata.

Kafka uses this internal topic to maintain consumer progress.

**Important:** Consumer offsets are not stored inside the consumer's application memory alone. Committed offsets are stored by Kafka.

---

## 8. Automatic vs Manual Offset Commit

Kafka supports automatic and manual offset commits.

### A. Automatic commit

With automatic commits enabled, the consumer periodically commits offsets.

Example Java consumer configuration:

```properties
group.id=order-processors
enable.auto.commit=true
auto.commit.interval.ms=5000
```

This configures periodic automatic commits, with a default interval of five seconds in this example.

Advantages:
- Simple configuration.
- Less application-level offset management.

Disadvantages:
- The committed position may get ahead of successfully completed business processing.
- A crash can cause records to be skipped by the application if offsets were committed too early.
- Reprocessing can occur if processing finishes but the commit does not happen.

### B. Manual commit

With manual commits, the application decides when to commit offsets.

Example:

```properties
group.id=order-processors
enable.auto.commit=false
```

The application commits offsets after processing records.

Conceptually:

```text
Read records
     |
     v
Process records
     |
     v
Write output successfully
     |
     v
Commit offsets
```

Advantages:
- Better control over processing progress.
- Can commit after successful output writes.
- Useful for pipelines that need careful failure handling.

Disadvantages:
- More application logic.
- Incorrect commit timing can cause duplicates or data loss.

### Interview point

Manual commits are useful when the application needs to coordinate offset commits with successful processing. They do not automatically guarantee exactly-once processing.

---

## 9. Synchronous vs Asynchronous Commit

Kafka consumers can commit offsets synchronously or asynchronously.

### Synchronous commit

A synchronous commit waits for the broker's response.

Conceptual Java example:

```java
consumer.commitSync();
```

Characteristics:
- Waits for the commit result.
- Makes it easier to handle commit failures.
- Can add latency.

### Asynchronous commit

An asynchronous commit sends the request without waiting for the result.

Conceptual Java example:

```java
consumer.commitAsync();
```

Characteristics:
- Does not block waiting for the response.
- Can improve throughput.
- Requires careful handling of failures and commit ordering.

### Comparison

| Synchronous commit | Asynchronous commit |
|---|---|
| Waits for response | Does not wait for response |
| Easier to handle result immediately | Requires callback or later error handling |
| Can add latency | Usually lower waiting overhead |
| Useful for important commit boundaries | Useful for frequent commits |

A common strategy is to use asynchronous commits during normal processing and a synchronous commit during controlled shutdown or other critical boundaries, with careful coordination.

---

## 10. What Is Consumer Rebalancing?

Rebalancing is the process of redistributing partitions among consumers in a consumer group.

It can happen when:
- A new consumer joins the group.
- A consumer leaves the group.
- A consumer crashes or becomes unavailable.
- Topic partition count changes.
- Subscription or assignment changes.

### Example

Before rebalancing:

```text
Partitions: 3

Consumer 1 → Partition 0
Consumer 2 → Partition 1, 2
```

Consumer 3 joins.

After rebalancing:

```text
Consumer 1 → Partition 0
Consumer 2 → Partition 1
Consumer 3 → Partition 2
```

Kafka redistributes partition ownership.

### Why is rebalancing important?

It allows Kafka to adjust processing assignments as the group changes.

However, rebalancing can temporarily interrupt processing.

### Eager vs cooperative rebalancing

**Eager rebalancing**
- Revokes existing partition assignments before assigning new ones.
- Can temporarily stop processing across the group.

**Cooperative rebalancing**
- Allows partitions that do not need reassignment to remain assigned.
- Can reduce unnecessary processing interruptions.

The actual behavior depends on the configured partition assignment strategy and Kafka client version.

---

## 11. Consumer Group Coordinator

Kafka uses a group coordinator to manage consumer group membership and offset commits.

The coordinator is a broker responsible for a particular consumer group.

Its responsibilities include:
- Managing group membership.
- Coordinating rebalances.
- Managing group generation and membership information.
- Handling offset commit requests.

A consumer discovers its group coordinator and communicates with it for group-related operations.

### Important distinction

| Component | Responsibility |
|---|---|
| Broker | Stores and serves Kafka partitions |
| Group coordinator | Manages a consumer group |
| Consumer | Reads records |
| Partitioner | Selects a destination partition for producer records |
| Partition assignor | Determines consumer-to-partition assignments |

---

## 12. Consumer Failure and Recovery

Suppose a group has three consumers.

```text
Consumer 1 → Partition 0
Consumer 2 → Partition 1
Consumer 3 → Partition 2
```

Consumer 2 crashes.

Kafka detects that the consumer is no longer an active group member and initiates a rebalance.

After reassignment:

```text
Consumer 1 → Partition 0, 1
Consumer 3 → Partition 2
```

Consumer 1 may now process Partition 1.

It resumes from the committed offset for that partition.

### What happens to uncommitted records?

If records were processed but their offsets were not committed, they may be read again after reassignment.

This is one reason consumers should be designed to handle duplicate processing safely.

---

## 13. Consumer Lag

### What is consumer lag?

Consumer lag is the difference between the latest available offset position in a partition and the consumer group's committed position.

A simplified conceptual formula is:

```text
Consumer Lag = Latest Log End Offset - Committed Offset
```

Example:

```text
Latest Log End Offset = 1,000
Committed Offset      =   900

Consumer Lag          =   100
```

The group is approximately 100 offsets behind in this example.

### Why does consumer lag increase?

- Consumers process records slowly.
- Producer throughput increases.
- Consumer count is insufficient.
- Partitions are unevenly distributed.
- A downstream database or API is slow.
- Consumer processing is blocked by expensive transformations.
- A consumer repeatedly fails or rebalances.

### How to reduce consumer lag

1. Optimize consumer processing logic.
2. Increase consumer instances, if partitions allow.
3. Increase partitions when appropriate.
4. Improve downstream sink performance.
5. Investigate hot partitions.
6. Avoid unnecessary blocking operations.
7. Monitor processing and commit rates.

**Important:** Lag measured in offsets is not the same as lag measured in seconds. A difference of 100 offsets may represent a few milliseconds or several hours depending on event volume and traffic.

---

## 14. Offset Reset Strategy

What happens when a consumer group has no valid committed offset for a partition?

Kafka uses the configured offset reset strategy.

Common values:

### `earliest`

Start from the earliest available offset.

```properties
auto.offset.reset=earliest
```

Useful when the consumer needs to process available historical records.

### `latest`

Start from the latest end position.

```properties
auto.offset.reset=latest
```

Useful when the consumer should process newly arriving records rather than historical records.

### `none`

Throw an exception when no valid committed offset exists.

```properties
auto.offset.reset=none
```

Useful when the application requires an explicit offset decision.

### Important point

`auto.offset.reset` applies when there is no valid committed offset, such as when a group has no committed offset or its committed offset is no longer available.

It does not normally reset a consumer that already has a valid committed offset.

---

## 15. At-Most-Once, At-Least-Once, and Exactly-Once Processing

These concepts describe delivery or processing guarantees.

### At-most-once

Offsets are committed before processing.

```text
Read records
     |
     v
Commit offsets
     |
     v
Process records
```

If the application crashes after committing but before completing processing, records can be skipped.

**Result:** Records may be lost from the application's processing perspective, but duplicates are avoided in this failure scenario.

### At-least-once

Records are processed before offsets are committed.

```text
Read records
     |
     v
Process records
     |
     v
Commit offsets
```

If processing succeeds but the consumer crashes before committing, the records may be processed again.

**Result:** No records should be skipped under the intended recovery design, but duplicates are possible.

### Exactly-once

Exactly-once processing aims to ensure that each input record's effect is reflected once in the committed output.

Kafka supports transactional processing for suitable Kafka-to-Kafka workflows.

For external databases or storage systems, end-to-end exactly-once behavior requires coordination or idempotent writes at the destination.

### Comparison

| Guarantee | Offset timing | Main risk |
|---|---|---|
| At-most-once | Before processing | Lost processing |
| At-least-once | After processing | Duplicate processing |
| Exactly-once | Coordinated processing and commit | Requires suitable transactional or idempotent design |

**Interview point:** Kafka's exactly-once features do not automatically make every external database or data lake write exactly once.

---

## 16. Idempotency in Consumer Applications

Idempotency means that processing the same event more than once produces the same final result as processing it once, under the defined business rules.

It is important because Kafka consumers may process records again after failures.

### Example

Suppose an order event is written to a Delta table.

A retry should not create duplicate order records.

One approach is to use a stable business key such as `order_id` and perform an idempotent upsert.

Conceptual Delta Lake example:

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

The source must be deduplicated so that it does not contain multiple conflicting rows for the same merge key.

### Practical considerations

- Use stable event IDs or business keys.
- Deduplicate incoming records.
- Make writes safe to retry.
- Commit offsets only after the output write succeeds.
- Handle failures between writing output and committing offsets.

**Important:** A `MERGE` is not automatically idempotent for every business operation. For example, incrementing a counter on every retry can still produce duplicate effects.

---

## 17. Kafka Consumer Configuration

Common consumer configuration properties:

| Configuration | Purpose |
|---|---|
| `group.id` | Identifies the consumer group |
| `enable.auto.commit` | Enables or disables automatic commits |
| `auto.commit.interval.ms` | Automatic commit interval |
| `auto.offset.reset` | Starting behavior when no valid committed offset exists |
| `max.poll.records` | Maximum records returned by a single poll |
| `max.poll.interval.ms` | Maximum interval between consumer poll calls before the consumer can be considered failed |
| `session.timeout.ms` | Timeout for detecting an unresponsive consumer, subject to group protocol |
| `heartbeat.interval.ms` | Heartbeat interval in applicable consumer group protocols |

Example:

```properties
group.id=order-processors
enable.auto.commit=false
auto.offset.reset=earliest
max.poll.records=500
max.poll.interval.ms=300000
```

These are illustrative settings, not universal production defaults.

### `max.poll.records`

Controls how many records a consumer can receive in one `poll()` call.

It does not directly determine the total amount of data stored in Kafka.

### `max.poll.interval.ms`

If the application takes too long between poll calls, the consumer may be considered unable to make progress and removed from the group.

### `session.timeout.ms`

Controls how long the coordinator waits for heartbeats before considering a consumer unavailable under the applicable group protocol.

Exact heartbeat and timeout behavior depends on the Kafka client and group protocol.

---

## 18. Kafka Consumer vs Consumer Group

| Consumer | Consumer Group |
|---|---|
| Individual application instance | Collection of coordinated consumers |
| Reads assigned partitions | Distributes partitions among members |
| Has a local reading position | Maintains committed offsets for the group |
| Can fail independently | Can rebalance after membership changes |

---

## 19. Practical Data Engineering Scenario

Consider an e-commerce company processing order events.

### Architecture

```text
E-commerce Application
          |
          v
       Producer
          |
          v
    Kafka Topic: orders
          |
     ┌────┼────┐
     v    v    v
    P0   P1   P2
     |    |    |
     └────┼────┘
          |
          v
   Consumer Group:
     order-processors
          |
          v
  Spark Structured Streaming
          |
          v
       Delta Lake
```

### Processing workflow

1. The producer publishes order events to Kafka.
2. Kafka distributes records across partitions.
3. Consumers in the group receive partition assignments.
4. Consumers read records and track offsets.
5. The processing application validates and transforms records.
6. Results are written to Delta Lake.
7. The application manages progress and recovery using checkpoints or Kafka offset commits, depending on the integration.

### Failure scenario

Suppose a consumer writes records successfully to the target but crashes before its offset is committed.

After recovery, the same records may be processed again.

To avoid duplicate output:
- Use event IDs or business keys.
- Deduplicate incoming events.
- Make target writes idempotent.
- Coordinate progress and output commits where supported.

---

## 20. Kafka Consumer Groups vs Spark Structured Streaming

Both can consume Kafka data, but their offset management differs.

| Kafka consumer application | Spark Structured Streaming |
|---|---|
| Uses Kafka consumer group coordination | Uses Spark's streaming execution and checkpoint mechanism |
| Committed offsets commonly stored in `__consumer_offsets` | Kafka source offsets are tracked in the Spark checkpoint |
| Uses Kafka consumer APIs | Uses Spark's Kafka source connector |
| Consumer instances receive partition assignments | Spark schedules tasks to read Kafka partitions |
| Application manages commits | Spark manages source progress through its checkpoint |

**Important:** Spark Structured Streaming's Kafka source does not rely on ordinary Kafka consumer-group committed offsets in the same way as a standard Kafka consumer application. Spark tracks source offsets in its checkpoint.

For recovery, preserve the checkpoint location. Deleting it can cause the query to lose its previous progress information.

---

## 21. Common Interview Questions and Answers

### Q1. What is a Kafka consumer group?

A group of consumers that coordinate to consume topic partitions and process records in parallel.

### Q2. Why do we use consumer groups?

To distribute processing work, scale consumers, and allow independent applications to consume the same topic.

### Q3. Can two consumers in the same group read the same partition simultaneously?

Not under normal group assignment. A partition is assigned to at most one consumer in a group at a time.

### Q4. Can two different consumer groups read the same partition?

Yes. Each group independently consumes the topic and maintains its own offsets.

### Q5. What is an offset?

A sequential position of a record within a Kafka partition.

### Q6. Where are committed offsets stored?

In Kafka's internal `__consumer_offsets` topic.

### Q7. What is the difference between current position and committed offset?

Current position is the next record position the consumer will read. Committed offset is the saved position used for recovery.

### Q8. What is consumer lag?

The difference between the latest available offset position and the consumer group's committed position.

### Q9. What causes consumer rebalancing?

Consumer joins, leaves, failures, subscription changes, and partition-count changes can trigger rebalancing.

### Q10. What happens if a consumer crashes?

Kafka detects the failed group member, rebalances its partitions, and assigns them to other consumers. They resume from committed offsets.

### Q11. What is the difference between automatic and manual commit?

Automatic commit periodically commits offsets. Manual commit lets the application decide when to save progress.

### Q12. What is `auto.offset.reset`?

It determines where consumption starts when no valid committed offset exists.

### Q13. What is the difference between `earliest` and `latest`?

`earliest` starts from the earliest available offset; `latest` starts from the current end position when offset reset is required.

### Q14. What is at-least-once processing?

Records are processed before offsets are committed, so failures can cause records to be processed again.

### Q15. How can duplicate processing be handled?

Use idempotent writes, stable event IDs, deduplication, and appropriate offset commit timing.

### Q16. Does Kafka guarantee exactly-once processing for an external database?

No. External sinks require suitable transaction coordination or idempotent processing to achieve exactly-once effective results.

### Q17. What happens if there are more consumers than partitions?

Some consumers remain idle because there are not enough partitions to assign.

### Q18. What is the role of the group coordinator?

It manages consumer group membership, partition assignment coordination, and offset commit requests.

### Q19. Does Spark Structured Streaming use Kafka committed offsets?

It tracks Kafka source offsets in its own checkpoint rather than relying on ordinary consumer-group committed offsets for recovery.

### Q20. Why is idempotency important?

It makes repeated processing safe when a failure causes previously processed records to be read again.

---

## 22. Common Mistakes to Avoid

1. **Confusing offsets with record IDs:** Offsets identify positions within partitions; they are not business identifiers.
2. **Assuming offsets are globally unique:** They are partition-specific.
3. **Assuming a consumer group deletes records:** Kafka retention is independent of consumer progress.
4. **Assuming more consumers always increase throughput:** Parallelism is limited by partition assignments.
5. **Committing before successful processing:** This can cause records to be skipped after failures.
6. **Committing only after processing without idempotency:** This can cause duplicate output.
7. **Confusing `earliest` with replay every time:** It applies when there is no valid committed offset.
8. **Ignoring rebalancing:** Rebalances can temporarily interrupt processing.
9. **Treating consumer lag as a time measurement:** Offset lag measures record positions, not elapsed time.
10. **Assuming Spark and standard Kafka consumers manage offsets identically:** Their recovery mechanisms differ.

---

## 23. Quick Revision

| Concept | Remember |
|---|---|
| Consumer | Reads records from Kafka |
| Consumer group | Coordinates multiple consumers |
| `group.id` | Identifies a consumer group |
| Offset | Record position within a partition |
| Committed offset | Saved next position for recovery |
| `__consumer_offsets` | Internal topic storing committed offsets |
| Rebalancing | Redistributes partitions among consumers |
| Group coordinator | Manages group membership and commits |
| Consumer lag | Difference between latest position and committed position |
| `earliest` | Start at earliest available offset when reset is needed |
| `latest` | Start at the end position when reset is needed |
| Auto commit | Kafka client periodically commits offsets |
| Manual commit | Application controls offset commits |
| At-most-once | Possible loss of processing |
| At-least-once | Possible duplicate processing |
| Idempotency | Makes retries safe under defined conditions |

### Remember these five points

1. A consumer group distributes partition processing among its members.
2. Different consumer groups maintain independent offsets.
3. Kafka stores committed offsets in `__consumer_offsets`.
4. Committing too early can cause skipped processing; committing too late can cause duplicates.
5. Idempotent output writes are essential for reliable retry handling.

---

## 24. Self-Assessment

Try answering these without looking at the notes.

1. What is a consumer group?
2. How does Kafka assign partitions to consumers?
3. What happens when the number of consumers exceeds the number of partitions?
4. What is the difference between current position and committed offset?
5. Where does Kafka store committed offsets?
6. What happens when a consumer crashes?
7. What causes consumer rebalancing?
8. What is the difference between automatic and manual offset commits?
9. Explain `earliest`, `latest`, and `none`.
10. What is consumer lag, and how can you reduce it?
11. Explain at-most-once and at-least-once processing.
12. Why is idempotency important in Kafka data pipelines?
13. How does Spark Structured Streaming track Kafka source offsets?
14. How would you prevent duplicate records when writing Kafka events to Delta Lake?

**Interview readiness:** You should be able to explain how three consumers process three partitions, what happens when one consumer crashes, how offsets help recovery, and why idempotent writes are important.

