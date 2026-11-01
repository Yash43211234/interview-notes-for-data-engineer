

# Kafka Brokers, Topics, and Partitions

## 1. Introduction

Apache Kafka is a distributed event-streaming platform used to publish, store, and consume streams of records.

Kafka organizes data using three important concepts:

- **Broker:** A Kafka server that stores and serves data.
- **Topic:** A named category or logical stream of records.
- **Partition:** A division of a topic that enables parallel processing and distributed storage.

Understanding these concepts is essential for designing scalable and fault-tolerant data pipelines.

---

## 2. Kafka Broker

### What is a broker?

A Kafka broker is a server that receives, stores, and serves messages.

A Kafka cluster consists of one or more brokers working together.

Each broker:
- Stores topic partitions on disk.
- Accepts messages from producers.
- Serves messages to consumers.
- Handles partition leadership and replication.
- Participates in cluster coordination.

### Example

Imagine a Kafka cluster containing three brokers:

```text
Kafka Cluster
│
├── Broker 1
├── Broker 2
└── Broker 3
```

Producers send records to topics, and brokers store those records in topic partitions.

### Broker ID

Every broker has a unique ID within its Kafka cluster.

Example:

```properties
broker.id=1
```

In newer Kafka configurations, `node.id` is used in KRaft mode.

### Broker vs Kafka Cluster

| Broker | Kafka Cluster |
|---|---|
| One Kafka server | Collection of Kafka brokers |
| Stores assigned partitions | Distributes partitions across brokers |
| Handles requests | Coordinates distributed data storage |
| Can operate alone | Usually contains multiple brokers for resilience |

**Interview point:** A broker is an individual Kafka server, whereas a Kafka cluster is a group of brokers working together.

---

## 3. Kafka Topic

### What is a topic?

A topic is a logical category or named stream where Kafka records are published and consumed.

Producers write records to topics, and consumers read records from topics.

Example:

An e-commerce application might have:

```text
Kafka Cluster
│
├── orders
├── payments
├── customers
└── deliveries
```

Each topic stores a particular type of event.

### Example records

The `orders` topic might contain:

| order_id | customer_id | amount |
|---|---|---:|
| 101 | C01 | 500 |
| 102 | C02 | 750 |
| 103 | C03 | 900 |

A producer publishes these records, and one or more consumers can process them.

### Topic characteristics

1. A topic has a unique name within a Kafka cluster.
2. A topic is divided into partitions.
3. Records are stored durably according to configured retention policies.
4. Multiple consumers can independently read the same topic.
5. Records are not normally deleted immediately after consumption.

### Topic retention

Kafka retains records based on configured retention policies, not simply until a consumer reads them.

Common retention configurations:

```properties
log.retention.hours=168
log.retention.bytes=-1
```

These are illustrative broker-level settings:
- `log.retention.hours=168`: Seven days of time-based retention.
- `log.retention.bytes=-1`: No size-based limit at the broker configuration level.

Topic-level configurations can override broker defaults.

Kafka can also use log compaction.

### Retention vs Log Compaction

| Retention | Log Compaction |
|---|---|
| Removes old log segments based on time or size | Retains the latest value for each key, subject to compaction |
| Useful for event history within a retention window | Useful for maintaining the latest state by key |
| Does not require a record key | Requires records to have keys for key-based compaction |

Example:

A topic receives:

```text
Key: C01, Value: Kolkata
Key: C02, Value: Delhi
Key: C01, Value: Mumbai
```

After compaction, the latest value for `C01` can remain:

```text
Key: C01, Value: Mumbai
Key: C02, Value: Delhi
```

Compaction is performed asynchronously. Older records may remain temporarily, and tombstones are used to represent deletions.

---

## 4. Kafka Partition

### What is a partition?

A partition is a smaller, ordered log inside a Kafka topic.

A topic can contain multiple partitions, and each partition stores records independently.

Partitions allow Kafka to:
- Distribute data across brokers.
- Process records in parallel.
- Scale producer and consumer workloads.
- Replicate data for fault tolerance.

### Example

Suppose the `orders` topic has three partitions:

```text
Topic: orders
│
├── Partition 0
│   ├── Record 1
│   ├── Record 2
│   └── Record 3
│
├── Partition 1
│   ├── Record 4
│   ├── Record 5
│   └── Record 6
│
└── Partition 2
    ├── Record 7
    ├── Record 8
    └── Record 9
```

Each partition is an independent append-only log.

### Record ordering

Kafka guarantees ordering within a partition.

For example:

```text
Partition 0:
Offset 0 → Order A
Offset 1 → Order B
Offset 2 → Order C
```

Records are read in partition order.

However, Kafka does not guarantee a single global ordering across all partitions of a topic.

**Interview point:** If strict ordering is required for a particular entity, route its records to the same partition using a consistent key.

---

## 5. Partition Key

### What is a partition key?

A partition key is a record attribute used by Kafka's partitioner to determine which partition receives the record.

Example:

```text
order_id = 101
customer_id = C01
amount = 500
```

If `customer_id` is selected as the key, records for the same customer are generally routed to the same partition when the partition count and partitioning strategy remain unchanged.

### Example

```text
Topic: customer-events

Key: C01 → Partition 0
Key: C02 → Partition 1
Key: C01 → Partition 0
Key: C03 → Partition 2
```

This helps preserve ordering for records belonging to the same customer.

### How does Kafka choose a partition?

Common behavior:

**1. Key-based partitioning**

When a key is provided, Kafka's default keyed partitioning generally uses a hash of the serialized key to select a partition.

Conceptually:

```text
partition = hash(key) % number_of_partitions
```

This is an illustration of the concept, not an exact specification of every Kafka partitioner implementation.

**2. No-key partitioning**

When records have no key, the producer's partitioner distributes them across partitions. Exact behavior depends on the configured partitioner and Kafka client version.

### Choosing a good partition key

A good partition key should:
- Distribute records reasonably evenly.
- Avoid concentrating most records in one partition.
- Preserve ordering where required.
- Match the business entity being processed.

Example:

For an order-processing pipeline, `customer_id` or `order_id` may be appropriate depending on the ordering requirement.

### What is a hot partition?

A hot partition receives significantly more traffic than other partitions.

Example:

```text
Partition 0 → 80% of records
Partition 1 → 10% of records
Partition 2 → 10% of records
```

Possible causes:
- A highly frequent key.
- Poor partition-key selection.
- Uneven traffic distribution.
- A large number of events belonging to one entity.

Solutions:
- Choose a better partition key.
- Increase partition count where appropriate.
- Use a composite or salted key if the business requirement permits.
- Review consumer processing capacity.

**Important:** Increasing partitions does not automatically fix every hot-key problem. Records with the same key may still go to the same partition.

---

## 6. Partition Offset

### What is an offset?

An offset is a sequential position assigned to a record within a Kafka partition.

Offsets identify records inside a particular partition.

Example:

```text
Partition 0

Offset 0 → Order A
Offset 1 → Order B
Offset 2 → Order C
Offset 3 → Order D
```

Offsets are not globally unique across a topic.

A record is identified by its topic, partition, and offset.

```text
orders, partition 0, offset 2
```

### Important points

- Offsets are maintained separately for each partition.
- Consumers use offsets to track their reading progress.
- Offsets do not necessarily start at zero for the currently retained records.
- Deleting or compacting records can create gaps in the offsets visible to consumers.
- An offset is not the same as a consumer's committed offset.

### Example

Suppose a consumer has processed records through offset 5.

Its committed position is typically the next offset to read:

```text
Committed offset = 6
```

This means the consumer can resume from offset 6 after restarting, subject to its offset-reset and retention configuration.

---

## 7. Replication and Replica

### What is replication?

Replication means maintaining copies of a partition across multiple brokers.

It protects data availability when a broker fails.

Example:

```text
Topic: orders
Replication Factor: 3

Broker 1 → Partition 0 (Leader)
Broker 2 → Partition 0 (Follower)
Broker 3 → Partition 0 (Follower)
```

One replica acts as the leader, while the others are followers.

### Leader replica

The leader:
- Handles normal produce requests for that partition.
- Handles normal fetch requests for that partition.
- Determines the ordered log for that partition.
- Coordinates replication with follower replicas.

### Follower replica

Followers:
- Fetch records from the leader.
- Maintain replicated copies.
- Can become leaders if the current leader becomes unavailable and Kafka selects an eligible replacement.

### Replication factor

Replication factor specifies the number of replicas for each partition.

Example:

```text
Replication Factor = 3
```

This means each partition has three replicas, including one leader and two followers.

### Replication factor vs partition count

| Partition count | Replication factor |
|---|---|
| Number of partitions in a topic | Number of copies of each partition |
| Supports parallelism and distribution | Supports fault tolerance |
| More partitions can increase overhead | More replicas consume additional storage and network resources |

### Important note

A replication factor of 3 does not mean three brokers are always required for every topic operation, but the replicas must be distributed across distinct brokers.

A topic with replication factor 3 generally requires at least three brokers to place all replicas on separate brokers.

---

## 8. ISR — In-Sync Replicas

### What is ISR?

ISR stands for In-Sync Replicas.

It is the set of replicas that are sufficiently caught up with the partition leader according to Kafka's replication rules.

Example:

```text
Partition 0

Leader: Broker 1
Followers:
  Broker 2 → In Sync
  Broker 3 → In Sync

ISR = [Broker 1, Broker 2, Broker 3]
```

If Broker 3 falls too far behind, it may be removed from the ISR.

```text
ISR = [Broker 1, Broker 2]
```

### Why is ISR important?

ISR helps Kafka determine which replicas are eligible for certain leadership and acknowledgement decisions.

It supports data durability and availability.

### `min.insync.replicas`

This setting controls the minimum number of in-sync replicas required for a write when the producer uses `acks=all`.

Example:

```properties
min.insync.replicas=2
```

With:
- Replication factor = 3
- `min.insync.replicas = 2`
- Producer `acks = all`

Kafka requires at least two in-sync replicas for the write to be acknowledged successfully.

If fewer than two replicas are in sync, the write fails rather than accepting a write below the configured minimum.

**Important:** `min.insync.replicas` does not itself guarantee that every replica has acknowledged a write. `acks=all` and the ISR determine the acknowledgement behavior.

---

## 9. Leader Election

### What is leader election?

Leader election is the process of selecting a new leader replica when the current partition leader becomes unavailable.

Example:

```text
Before failure:

Broker 1 → Leader
Broker 2 → Follower
Broker 3 → Follower

Broker 1 fails.

After election:

Broker 2 → New Leader
Broker 3 → Follower
```

Kafka selects an eligible replica based on its replication and election rules.

### Preferred leader

Kafka can have a preferred replica for a partition.

The preferred replica is generally the first replica in the partition's configured replica assignment.

Preferred leader election can help restore the intended leadership distribution across brokers.

### Unclean leader election

Unclean leader election allows a replica outside the ISR to become leader when no ISR replica is available, if configured.

This can improve availability but may result in data loss because the selected replica may not contain all previously acknowledged records.

**Interview point:** Clean leader election prioritizes replicas in the ISR. Unclean leader election may improve availability at the cost of possible data loss.

---

## 10. How Topics and Partitions Are Distributed

Suppose we have:

- 3 brokers
- 1 topic named `orders`
- 3 partitions
- Replication factor = 2

One possible distribution:

```text
Kafka Cluster
│
├── Broker 1
│   ├── Partition 0 (Leader)
│   └── Partition 2 (Follower)
│
├── Broker 2
│   ├── Partition 1 (Leader)
│   └── Partition 0 (Follower)
│
└── Broker 3
    ├── Partition 2 (Leader)
    └── Partition 1 (Follower)
```

Each partition has:
- One leader.
- One follower.

The replicas are distributed across different brokers.

This distribution provides:
- Parallel processing across partitions.
- Load distribution across brokers.
- Replication for fault tolerance.

The exact assignment is determined by Kafka's replica assignment process.

---

## 11. How Producers and Consumers Use Partitions

### Producer flow

```text
Producer
   |
   v
Topic: orders
   |
   v
Partitioner
   |
   ├── Partition 0
   ├── Partition 1
   └── Partition 2
```

The producer sends records to a topic. The partitioner determines the destination partition.

For a keyed record, the key influences partition selection.

### Consumer flow

Consumers read records from topic partitions.

A consumer group allows multiple consumers to share the processing workload.

Example:

```text
Topic: orders
Partitions: 3

Consumer Group: order-processors

Consumer 1 → Partition 0
Consumer 2 → Partition 1
Consumer 3 → Partition 2
```

Within a consumer group, a partition is assigned to at most one consumer at a time during normal assignment.

### What if there are more consumers than partitions?

Example:

```text
Partitions: 3
Consumers: 5

Consumer 1 → Partition 0
Consumer 2 → Partition 1
Consumer 3 → Partition 2
Consumer 4 → Idle
Consumer 5 → Idle
```

Two consumers remain unassigned because there are only three partitions.

### Important points

- A partition can be assigned to only one consumer in a consumer group at a time.
- Different consumer groups can independently consume the same topic.
- A consumer can read multiple partitions.
- Adding consumers beyond the partition count does not increase parallelism for that topic within the same group.
- Partition reassignment can occur when consumers join, leave, or fail.

---

## 12. Increasing Partition Count

Kafka allows a topic's partition count to be increased.

Example:

```text
Before:
orders → 3 partitions

After:
orders → 6 partitions
```

### Why increase partitions?

- Higher potential consumer parallelism.
- More distribution across brokers.
- Increased capacity for concurrent processing.

### Important risks

Increasing partitions can affect key-based ordering.

For example, if a key previously mapped to Partition 1, it may map to another partition after the partition count changes.

This can cause records for the same key to be split across partitions over time.

Other considerations:
- More partition metadata.
- More files and open file handles.
- More leader and replica management overhead.
- Potentially more consumer coordination work.

**Best practice:** Choose a suitable partition count based on throughput, expected consumer parallelism, broker capacity, and future growth.

---

## 13. Practical Kafka Example for a Data Engineer

Consider an e-commerce platform generating order events.

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
    ┌─────┼─────┐
    v     v     v
   P0    P1    P2
    |     |     |
    └─────┼─────┘
          |
          v
   Consumer Group
          |
          v
  Spark Structured Streaming
          |
          v
     ADLS Gen2
          |
          v
   Databricks / Delta Lake
```

### Sample event

```json
{
  "order_id": 101,
  "customer_id": "C01",
  "amount": 500,
  "status": "PLACED"
}
```

### Design decisions

| Requirement | Design choice |
|---|---|
| Separate order events | Create an `orders` topic |
| Scale ingestion | Use multiple partitions |
| Preserve customer event order | Use `customer_id` as the key |
| Improve fault tolerance | Configure replication |
| Process events in parallel | Use multiple consumers in a group |
| Recover after consumer restart | Use committed offsets |
| Store analytical data | Consume into ADLS Gen2 or Delta Lake |

### Important design consideration

Choosing `customer_id` as the key preserves ordering for a customer, but it can create hot partitions if a few customers generate a disproportionate amount of traffic.

Choose the key according to the business ordering requirement and traffic distribution.

---

## 14. Common Interview Questions and Answers

### Q1. What is a Kafka broker?

A broker is a Kafka server responsible for storing topic partitions and handling producer and consumer requests.

### Q2. What is a Kafka topic?

A topic is a named logical stream where producers publish records and consumers read them.

### Q3. What is a partition?

A partition is an ordered log within a topic. It allows Kafka to distribute data and support parallel processing.

### Q4. Why does Kafka use partitions?

Partitions enable horizontal scaling, distributed storage, and parallel consumption.

### Q5. Does Kafka guarantee ordering?

Kafka guarantees ordering within a partition, not across all partitions of a topic.

### Q6. What is a partition key?

A partition key is a record attribute used by the producer's partitioner to select a partition.

### Q7. What is an offset?

An offset is a record's sequential position within a partition.

### Q8. What is replication factor?

It specifies how many replicas of each partition Kafka maintains.

### Q9. What is a leader replica?

The leader handles normal reads and writes for a partition.

### Q10. What is a follower replica?

A follower replicates data from the leader and can become a leader if eligible.

### Q11. What is ISR?

ISR is the set of replicas that are sufficiently caught up with the partition leader.

### Q12. What happens if a broker fails?

Kafka can elect an eligible follower replica as the new leader for affected partitions, subject to the cluster's election and replication state.

### Q13. What is `min.insync.replicas`?

It specifies the minimum number of in-sync replicas required for a write when the producer uses `acks=all`.

### Q14. Can multiple consumers read the same partition?

Different consumer groups can independently read the same partition. Within one consumer group, a partition is assigned to at most one consumer at a time.

### Q15. What happens if consumers exceed partitions?

Some consumers remain idle because each partition can be assigned to only one consumer in a group at a time.

### Q16. What is a hot partition?

A hot partition receives disproportionately high traffic, which can create processing bottlenecks.

### Q17. Can we increase the partition count?

Yes. However, increasing partitions can change key-to-partition mapping and affect ordering across records produced before and after the change.

### Q18. What is the difference between retention and compaction?

Retention removes old log segments based on configured policies. Compaction retains the latest value for each key, subject to compaction behavior.

### Q19. Is a Kafka offset globally unique?

No. An offset is unique only within its partition. A record is identified using topic, partition, and offset.

### Q20. Does Kafka delete a message after a consumer reads it?

No. Kafka retains records according to topic retention or compaction policies, independently of whether a consumer has read them.

---

## 15. Common Mistakes to Avoid

1. **Confusing brokers and partitions:** A broker is a server; a partition is a topic's ordered log.
2. **Assuming global ordering:** Ordering is guaranteed within a partition, not across a topic's partitions.
3. **Confusing partition count with replication factor:** Partition count supports parallelism; replication factor supports fault tolerance.
4. **Assuming consumers delete records:** Kafka retention is independent of consumer reads.
5. **Assuming more consumers always improve performance:** Consumer parallelism within a group is limited by the number of assigned partitions.
6. **Choosing a poor partition key:** Skewed keys can create hot partitions.
7. **Assuming replication means every replica is always synchronized:** Followers can fall behind and leave the ISR.
8. **Assuming increasing partitions is risk-free:** It can change key-based partition mapping.
9. **Treating offsets as globally unique:** Offsets are partition-specific.
10. **Confusing `acks=all` with every configured replica being available:** It means acknowledgement from the current ISR, subject to the minimum ISR requirement.

---

## 16. Quick Revision

| Concept | Remember |
|---|---|
| Broker | Kafka server |
| Cluster | Group of brokers |
| Topic | Named stream of records |
| Partition | Ordered log within a topic |
| Partition key | Helps determine partition assignment |
| Offset | Record position within a partition |
| Leader | Handles normal partition reads and writes |
| Follower | Replicates the leader |
| Replication factor | Number of partition replicas |
| ISR | In-sync replicas |
| `min.insync.replicas` | Minimum ISR size for `acks=all` writes |
| Hot partition | Partition receiving disproportionate traffic |
| Retention | Removes old data based on configured policies |
| Compaction | Retains latest values per key, subject to compaction |

### Remember these five points

1. A Kafka cluster contains brokers.
2. A topic is divided into partitions.
3. Ordering is guaranteed within a partition.
4. Replication protects against broker failures.
5. Consumer parallelism within a group is limited by available partitions.

---

## 17. Self-Assessment

Try answering these without looking at the notes.

1. What is the difference between a broker and a topic?
2. Why does Kafka divide topics into partitions?
3. How does Kafka select a partition for a keyed record?
4. What is the difference between a partition and an offset?
5. Why is ordering not guaranteed across multiple partitions?
6. What is the difference between a leader and a follower?
7. What happens when a leader broker fails?
8. What is ISR, and why is it important?
9. How do `acks=all` and `min.insync.replicas` work together?
10. Why can adding more consumers fail to improve processing parallelism?
11. What is a hot partition, and how can you reduce it?
12. What risks should you consider before increasing partition count?

**Interview readiness:** You should be able to explain a three-broker Kafka cluster with three partitions and replication factor two, including how records are distributed, replicated, consumed, and recovered after a broker failure.

