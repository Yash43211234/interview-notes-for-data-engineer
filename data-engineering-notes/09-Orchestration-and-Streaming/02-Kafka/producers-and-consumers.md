

# Kafka Producers and Consumers

## 1. Introduction

Kafka producers and consumers are the two main components responsible for writing records to Kafka and reading records from Kafka.

- **Producer:** Publishes records to Kafka topics.
- **Consumer:** Reads records from Kafka topics.
- **Broker:** Stores and serves records.
- **Topic:** A named stream of records.
- **Partition:** An ordered log within a topic.

### Basic architecture

```text
Producer
   |
   v
Kafka Cluster
   |
   ├── Broker 1
   ├── Broker 2
   └── Broker 3
         |
         v
    Topic: orders
         |
    ┌────┼────┐
    v    v    v
   P0   P1   P2
    |    |    |
    └────┼────┘
         |
         v
      Consumer
         |
         v
   Data Processing
         |
         v
    Data Lake / DB
```

---

## 2. Kafka Producer

### What is a producer?

A Kafka producer is an application that publishes records to Kafka topics.

For example, an e-commerce application generates order events and sends them to the `orders` topic.

Sample event:

```json
{
  "order_id": 101,
  "customer_id": "C01",
  "amount": 500,
  "status": "PLACED"
}
```

The producer sends this event to Kafka, where it is stored in a topic partition.

### Producer responsibilities

- Create and publish records.
- Select the destination topic.
- Determine the destination partition.
- Serialize keys and values.
- Handle acknowledgements and retries.
- Batch records for efficient transmission.

---

## 3. Producer Record Structure

A Kafka record commonly contains:

| Field | Description |
|---|---|
| Topic | Destination topic |
| Partition | Destination partition, if explicitly selected or determined |
| Key | Optional record key |
| Value | Actual event data |
| Timestamp | Record timestamp |
| Headers | Optional key-value metadata |

Example:

```text
Topic: orders
Key: C01
Value: {"order_id": 101, "amount": 500}
Partition: 0
Timestamp: 2026-10-31T10:00:00
```

### Key vs Value

- **Key:** Helps determine partition assignment and can identify the entity associated with the event.
- **Value:** Contains the actual event payload.

For example:

```text
Key   = customer_id
Value = order details
```

Using the same key generally routes records to the same partition when the partition count and partitioning strategy remain unchanged.

This helps preserve ordering for records belonging to the same key.

---

## 4. How Does a Producer Send Records?

A simplified producer workflow:

```text
Application Creates Record
          |
          v
      Serializer
          |
          v
      Partitioner
          |
          v
    Producer Buffer
          |
          v
       Batching
          |
          v
      Kafka Broker
          |
          v
    Broker Acknowledgement
```

### Step 1: Create a record

The application creates a Kafka record containing a topic, key, and value.

### Step 2: Serialization

The producer converts the key and value into bytes.

Common serialization formats include:
- String
- JSON
- Avro
- Protobuf
- JSON Schema

Kafka stores records as bytes. The producer and consumer must use compatible serialization formats.

### Step 3: Partition selection

The producer determines which partition should receive the record.

Partition selection can depend on:
- Record key.
- Explicit partition.
- Configured partitioner.
- Number of available partitions.

### Step 4: Batching

The producer can collect multiple records and send them together.

Batching reduces network overhead and improves throughput.

### Step 5: Sending and acknowledgement

The producer sends batches to the partition leader and handles acknowledgements according to the configured `acks` setting.

---

## 5. Producer Partitioner

### What is a partitioner?

A partitioner determines which partition receives a record when the producer has not explicitly specified a partition.

### Key-based partitioning

Suppose the producer uses `customer_id` as the key.

```text
Key: C01 → Partition 0
Key: C02 → Partition 1
Key: C01 → Partition 0
Key: C03 → Partition 2
```

Records with the same key generally go to the same partition when the partition count and partitioning strategy remain unchanged.

### Explicit partition

An application can specify the partition directly.

Conceptual record:

```text
Topic: orders
Partition: 1
Key: C01
Value: Order event
```

The explicit partition takes precedence over normal partition selection.

### No-key records

When no key is provided, the producer's partitioner distributes records across partitions according to its configured behavior.

The exact behavior depends on the Kafka client version and partitioner implementation.

### Why is partition selection important?

- It affects ordering.
- It affects load distribution.
- It influences consumer parallelism.
- Poor key selection can create hot partitions.

---

## 6. Producer Batching

### What is batching?

Batching means collecting multiple records and sending them together rather than sending every record separately.

Example:

```text
Without batching:

Record 1 → Broker
Record 2 → Broker
Record 3 → Broker

With batching:

Record 1 ┐
Record 2 ├── Batch → Broker
Record 3 ┘
```

### Important producer configurations

| Configuration | Purpose |
|---|---|
| `batch.size` | Maximum batch size in bytes for a partition |
| `linger.ms` | Time the producer may wait for more records before sending a non-full batch |
| `buffer.memory` | Approximate memory available to buffer records waiting to be sent |
| `compression.type` | Compression codec used for producer batches |

Example:

```properties
batch.size=32768
linger.ms=10
buffer.memory=33554432
compression.type=snappy
```

These are illustrative values, not universal production settings.

### Batch size

A larger batch can improve throughput by reducing request overhead.

However, excessively large batches can increase memory use and latency.

### Linger time

`linger.ms` allows the producer to wait briefly for additional records to fill a batch.

Increasing it may improve batching efficiency but can increase record delivery latency.

### Compression

Kafka supports codecs such as:
- `none`
- `gzip`
- `snappy`
- `lz4`
- `zstd`

Compression can reduce network usage and storage requirements, but it also consumes CPU.

---

## 7. Producer Acknowledgements

The `acks` configuration controls when the producer considers a write acknowledged.

| Setting | Behavior |
|---|---|
| `acks=0` | Does not wait for broker acknowledgement |
| `acks=1` | Waits for the partition leader's acknowledgement |
| `acks=all` | Waits for acknowledgement from the current in-sync replicas |

### `acks=0`

- Lowest acknowledgement overhead.
- Producer does not know whether the broker successfully stored the record.
- Weaker delivery assurance.

### `acks=1`

- The leader acknowledges the write.
- Followers do not need to acknowledge before the response.
- Leader failure before replication may cause data loss.

### `acks=all`

- The leader waits for the current ISR according to Kafka's replication protocol.
- Provides stronger durability when combined with suitable replication settings.

**Important:** `acks=all` does not mean every configured replica must acknowledge. It refers to the current in-sync replicas.

### `min.insync.replicas`

This setting defines the minimum number of in-sync replicas required for a write when using `acks=all`.

Example:

```properties
replication.factor=3
min.insync.replicas=2
```

With a producer using `acks=all`, Kafka requires at least two in-sync replicas for the write to succeed.

---

## 8. Producer Retries and Idempotence

### What are retries?

Retries allow the producer to resend records when certain temporary failures occur.

Possible causes:
- Network problems.
- Temporary broker unavailability.
- Leader changes.
- Request timeouts.

Example:

```text
Producer sends record
        |
        v
Temporary failure
        |
        v
Producer retries
        |
        v
Broker acknowledges
```

### What is producer idempotence?

Producer idempotence prevents duplicate records caused by supported producer retries.

Example:

1. Producer sends a record.
2. Broker stores the record.
3. The acknowledgement is lost.
4. Producer retries.
5. Kafka identifies the retry and avoids appending another copy within the idempotent producer protocol.

Configuration:

```properties
enable.idempotence=true
acks=all
```

Modern Kafka clients enable idempotence by default when compatible configurations are used.

### Important limitations

Producer idempotence:
- Protects against duplicates caused by supported retries.
- Uses producer IDs and sequence numbers.
- Does not deduplicate separate application-level duplicate events.
- Does not guarantee exactly-once writes to external databases.

---

## 9. Kafka Consumer

### What is a consumer?

A Kafka consumer is an application that reads records from Kafka topics.

Example:

An order-processing service consumes events from the `orders` topic and updates an order database.

### Consumer responsibilities

- Subscribe to topics.
- Receive partition assignments.
- Fetch records from brokers.
- Deserialize records.
- Process records.
- Manage offsets.
- Handle failures and rebalancing.

### Consumer workflow

```text
Kafka Topic
     |
     v
Partition Assignment
     |
     v
Consumer Fetch
     |
     v
Deserialization
     |
     v
Processing
     |
     v
Destination Write
     |
     v
Offset Commit
```

---

## 10. Consumer Subscription and Assignment

Consumers can subscribe to topics.

Example:

```text
Consumer Group: order-processors

Topic: orders
Partitions: 3

Consumer 1 → Partition 0
Consumer 2 → Partition 1
Consumer 3 → Partition 2
```

A consumer can also receive multiple partitions.

### Important rules

- A partition is assigned to at most one consumer in a group at a time.
- Multiple consumer groups can read the same topic independently.
- A consumer can process multiple partitions.
- The number of partitions limits the maximum number of active consumers for a topic within a group.

### Consumer group example

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

Both groups maintain independent offsets.

---

## 11. Consumer Polling

Kafka consumers fetch records by calling `poll()`.

Conceptual Java example:

```java
while (true) {
    ConsumerRecords<String, String> records =
        consumer.poll(Duration.ofMillis(100));

    for (ConsumerRecord<String, String> record : records) {
        process(record);
    }
}
```

### What does `poll()` do?

- Fetches available records from assigned partitions.
- Participates in consumer group coordination.
- Allows the consumer to maintain group membership.
- Returns records for application processing.

### `max.poll.records`

Controls the maximum number of records returned by a single `poll()` call.

Example:

```properties
max.poll.records=500
```

### `max.poll.interval.ms`

Controls the maximum allowed delay between poll calls before the consumer may be considered unable to make progress.

If processing takes too long, the consumer can lose its group membership and trigger a rebalance.

**Best practice:** Keep processing time within the configured polling interval or move long-running work into a carefully designed processing model.

---

## 12. Consumer Deserialization

Kafka stores record keys and values as bytes.

The consumer converts these bytes into usable application objects.

Common formats:
- String
- JSON
- Avro
- Protobuf
- JSON Schema

Producer and consumer serialization formats must be compatible.

Example:

```text
Producer
   |
   v
JSON Serializer
   |
   v
Kafka
   |
   v
JSON Deserializer
   |
   v
Consumer Application
```

### Serialization vs Deserialization

| Serialization | Deserialization |
|---|---|
| Converts objects into bytes | Converts bytes into objects |
| Used by producers | Used by consumers |
| Happens before sending | Happens after fetching |

---

## 13. Consumer Offsets

### What is an offset?

An offset is a record's sequential position within a Kafka partition.

Example:

```text
Partition 0

Offset 0 → Order A
Offset 1 → Order B
Offset 2 → Order C
Offset 3 → Order D
```

Consumers use offsets to track their progress.

### Current position vs committed offset

- **Current position:** The next offset the consumer will read.
- **Committed offset:** The saved next position used for recovery.

Example:

If records through offset 5 have been processed, the next position is 6.

```text
Committed offset = 6
```

Kafka stores committed consumer offsets in the internal topic:

```text
__consumer_offsets
```

### Automatic commit

```properties
enable.auto.commit=true
auto.commit.interval.ms=5000
```

The consumer periodically commits offsets.

### Manual commit

```properties
enable.auto.commit=false
```

The application controls when offsets are committed.

Manual commits are useful when offsets should be committed only after successful processing.

---

## 14. Consumer Rebalancing

### What is rebalancing?

Rebalancing is the process of redistributing partitions among consumers in a group.

It can occur when:
- A consumer joins.
- A consumer leaves.
- A consumer crashes.
- Topic partitions change.
- Subscription or assignment changes.

Example:

Before:

```text
Consumer 1 → Partition 0
Consumer 2 → Partition 1, 2
```

After Consumer 3 joins:

```text
Consumer 1 → Partition 0
Consumer 2 → Partition 1
Consumer 3 → Partition 2
```

### Why does rebalancing matter?

Rebalancing helps maintain workload distribution, but can temporarily interrupt processing.

Kafka supports eager and cooperative rebalancing strategies. Cooperative rebalancing can reduce unnecessary partition revocation.

---

## 15. Producer vs Consumer

| Producer | Consumer |
|---|---|
| Writes records | Reads records |
| Publishes to topics | Subscribes to topics |
| Serializes records | Deserializes records |
| Selects destination partition | Receives partition assignment |
| Uses acknowledgements | Uses offsets |
| Handles retries for publishing | Handles retries for processing |
| Can use idempotence | Can use manual offset commits |
| Can participate in transactions | Can coordinate consumed offsets with Kafka transactions |

---

## 16. End-to-End Data Engineering Example

Consider an e-commerce platform.

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
   Consumer Group
    order-processors
          |
          v
   Spark Structured Streaming
          |
          v
       Validation
          |
          v
     Transformation
          |
          v
       Delta Lake
          |
          v
    Analytics / Reports
```

### Producer responsibilities

1. Receive order events from the application.
2. Validate the event structure.
3. Serialize the event.
4. Select a partition using an appropriate key.
5. Publish the event.
6. Handle acknowledgements and retries.

### Consumer responsibilities

1. Read events from Kafka.
2. Deserialize the event.
3. Validate required fields.
4. Transform the data.
5. Handle duplicates and late events.
6. Write the result to Delta Lake.
7. Track processing progress safely.

### Example event

```json
{
  "order_id": 101,
  "customer_id": "C01",
  "amount": 500,
  "status": "PLACED"
}
```

### Design considerations

| Requirement | Design choice |
|---|---|
| Preserve customer event ordering | Use `customer_id` as key |
| Improve producer reliability | Idempotence and suitable acknowledgements |
| Distribute processing | Multiple partitions |
| Scale consumers | Multiple consumers in a group |
| Recover consumer progress | Offset management or streaming checkpoint |
| Prevent duplicate output | Idempotent writes and deduplication |
| Improve data quality | Validate schema and business rules |

---

## 17. Common Producer and Consumer Problems

| Problem | Possible cause | Approach |
|---|---|---|
| Producer cannot publish | Broker unavailable or authentication failure | Check connectivity, credentials, and broker health |
| Producer timeout | Network delay or broker overload | Investigate latency and retry configuration |
| Duplicate records | Producer retries or duplicate application events | Enable idempotence and deduplicate business events |
| Uneven partition traffic | Poor partition key | Review key distribution |
| Consumer lag | Slow processing or insufficient capacity | Optimize processing and review partition parallelism |
| Consumer repeatedly rebalances | Long processing between polls or unstable consumers | Review polling, heartbeat, and timeout settings |
| Deserialization failure | Incorrect schema or incompatible format | Validate schema and serializer configuration |
| Duplicate destination records | Failure after output write but before offset commit | Use idempotent writes |
| Records appear missing | Offset committed too early or wrong reset behavior | Inspect committed offsets and processing flow |

---

## 18. Best Practices

### Producer best practices

- Use meaningful topic names.
- Choose partition keys based on ordering and distribution requirements.
- Enable idempotence for reliable retries.
- Configure acknowledgements and replication appropriately.
- Tune batching and compression based on workload.
- Handle serialization errors.
- Monitor producer throughput and request latency.

### Consumer best practices

- Use meaningful consumer group IDs.
- Commit offsets at the correct processing boundary.
- Handle rebalancing correctly.
- Monitor consumer lag.
- Keep poll intervals within configured limits.
- Use stable event IDs for deduplication.
- Make destination writes safe to retry.

### Data engineering best practices

- Validate incoming schemas.
- Handle malformed records separately.
- Use idempotent writes.
- Monitor data freshness and completeness.
- Preserve checkpoints for streaming applications.
- Test failure and recovery behavior.
- Avoid assuming Kafka guarantees exactly-once writes to every external sink.

---

## 19. Common Interview Questions and Answers

### Q1. What is a Kafka producer?

An application that publishes records to Kafka topics.

### Q2. What is a Kafka consumer?

An application that reads records from Kafka topics.

### Q3. What is the role of a producer partitioner?

It selects the destination partition for a record when the partition is not explicitly specified.

### Q4. Why do producers serialize records?

Kafka stores record keys and values as bytes, so objects must be converted into bytes before transmission.

### Q5. What is batching?

Collecting multiple records and sending them together to reduce request overhead.

### Q6. What is the purpose of `linger.ms`?

It allows the producer to wait briefly for additional records to improve batch efficiency.

### Q7. What is the difference between `batch.size` and `linger.ms`?

`batch.size` controls the target batch size, while `linger.ms` controls how long the producer may wait for more records.

### Q8. What does `acks=all` mean?

The leader waits for acknowledgement from the current in-sync replicas according to Kafka's replication protocol.

### Q9. What is producer idempotence?

It prevents duplicate writes caused by supported producer retries.

### Q10. What is consumer polling?

The consumer calls `poll()` to fetch records and participate in group coordination.

### Q11. What is the purpose of `max.poll.records`?

It limits the number of records returned in a single poll call.

### Q12. What happens if a consumer takes too long to poll?

It may exceed `max.poll.interval.ms`, lose group membership, and trigger rebalancing.

### Q13. What is consumer rebalancing?

Redistributing topic partitions among active consumers in a group.

### Q14. What is the difference between automatic and manual offset commits?

Automatic commits periodically save progress. Manual commits let the application control when progress is saved.

### Q15. How do you prevent duplicate records in a destination?

Use stable event IDs, deduplication, idempotent writes, and appropriate offset commit timing.

### Q16. Can multiple consumer groups consume the same topic?

Yes. Each group independently consumes the topic and maintains its own offsets.

### Q17. Can a consumer read multiple partitions?

Yes. A consumer can be assigned multiple partitions within its group.

### Q18. What is the difference between a producer and a consumer?

A producer publishes records to Kafka, while a consumer reads and processes those records.

### Q19. Does Kafka guarantee ordering across all partitions?

No. Ordering is guaranteed within an individual partition.

### Q20. Does producer idempotence guarantee exactly-once processing in a database?

No. External database writes require their own transaction or idempotency strategy.

---

## 20. Common Mistakes to Avoid

1. **Confusing producer and consumer:** Producers publish; consumers read.
2. **Ignoring serialization:** Kafka records are stored as bytes.
3. **Choosing a poor partition key:** This can cause uneven distribution and hot partitions.
4. **Assuming batching always reduces latency:** Larger batches or longer linger times can increase waiting time.
5. **Confusing `acks=all` with all configured replicas:** It refers to the current ISR.
6. **Assuming producer idempotence removes all duplicates:** It handles supported retries, not separate duplicate business events.
7. **Ignoring consumer polling:** Long processing between polls can trigger group membership problems.
8. **Committing offsets too early:** This can cause skipped processing.
9. **Committing offsets too late without idempotency:** This can cause duplicate output.
10. **Assuming Kafka automatically makes external writes exactly once:** External sinks need suitable transaction or idempotency handling.

---

## 21. Quick Revision

| Concept | Remember |
|---|---|
| Producer | Publishes records |
| Consumer | Reads records |
| Serializer | Converts objects into bytes |
| Deserializer | Converts bytes into objects |
| Partitioner | Selects destination partition |
| Batching | Sends multiple records together |
| `batch.size` | Target batch size |
| `linger.ms` | Wait time for additional records |
| `acks` | Producer acknowledgement setting |
| Idempotence | Prevents duplicates from supported producer retries |
| `poll()` | Fetches records and supports group coordination |
| `max.poll.records` | Maximum records returned per poll |
| `max.poll.interval.ms` | Maximum delay between poll calls |
| Consumer group | Coordinates consumers |
| Offset | Record position within a partition |
| Rebalancing | Redistributes partition assignments |

### Remember these five points

1. Producers serialize and publish records.
2. Partitioners determine where records are stored.
3. Batching and compression can improve producer efficiency.
4. Consumers poll, deserialize, process, and track offsets.
5. Idempotent processing is essential for safe retries in data pipelines.

---

## 22. Self-Assessment

Try answering these without looking at the notes.

1. Explain the complete producer-to-consumer workflow.
2. What information does a Kafka record contain?
3. What is the role of serialization?
4. How does a producer select a partition?
5. Why is partition-key selection important?
6. Explain `batch.size` and `linger.ms`.
7. What is the difference between `acks=1` and `acks=all`?
8. How does producer idempotence work?
9. What does a consumer do when it calls `poll()`?
10. Explain `max.poll.records` and `max.poll.interval.ms`.
11. What happens when a consumer joins or leaves a group?
12. How are offsets committed?
13. How would you prevent duplicate records when writing Kafka events to Delta Lake?
14. Design a producer-consumer architecture for an e-commerce order pipeline.

**Interview readiness:** You should be able to explain how an event moves from a producer through Kafka partitions to a consumer, including serialization, partition selection, batching, acknowledgements, polling, offset management, and failure recovery.

