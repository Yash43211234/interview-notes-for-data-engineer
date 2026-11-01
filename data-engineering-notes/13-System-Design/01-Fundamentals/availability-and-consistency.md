

# Availability and Consistency in Distributed Systems

## 1. Introduction

Availability and consistency are two important properties of distributed systems.

When data is stored across multiple servers, databases, or geographic locations, the system must handle:
- Server failures.
- Network interruptions.
- Concurrent updates.
- Replication delays.
- Data synchronization.
- Conflicting operations.

Two important questions arise:

1. **Availability:** Can the system continue responding to requests?
2. **Consistency:** Do users see data that follows the system's consistency guarantees?

These properties influence how we design databases, data platforms, distributed applications, and large-scale data pipelines.

---

## 2. What Is Availability?

Availability is the ability of a system to remain operational and respond to requests when users or applications need it.

A highly available system continues providing service even when certain components fail.

### Example

Suppose an e-commerce application runs on three servers.

```text
             Application
                  |
             Load Balancer
             /     |     \
            /      |      \
         Server 1 Server 2 Server 3
```

If Server 2 fails, the load balancer can route requests to Server 1 and Server 3.

The application can continue operating.

### Availability Formula

\[
Availability =
\frac{Uptime}{Uptime + Downtime}
\times 100
\]

Where:
- Uptime = Time the system is operational.
- Downtime = Time the system is unavailable.

### Example

Suppose a service is expected to operate for 30 days.

It experiences 30 minutes of downtime.

\[
Availability =
\frac{43,200-30}{43,200}\times100
\]

\[
Availability \approx 99.93\%
\]

### Common Availability Levels

| Availability | Approximate Downtime per 30-Day Month |
|---|---:|
| 99% | 7 hours 12 minutes |
| 99.9% | 43 minutes 12 seconds |
| 99.95% | 21 minutes 36 seconds |
| 99.99% | 4 minutes 19 seconds |
| 99.999% | 26 seconds |

These are mathematical estimates. Actual SLA calculations may use different measurement windows and exclusions.

---

## 3. What Is Consistency?

Consistency is the guarantee that data operations behave according to the system's defined rules.

In distributed databases, consistency often refers to whether clients observe updates in a predictable way across replicas.

### Example

Suppose an e-commerce application stores a customer's account balance.

Initial balance:

```text
Customer Balance = ₹10,000
```

The customer withdraws ₹2,000.

The updated balance becomes:

```text
Customer Balance = ₹8,000
```

If the system provides strong consistency, a subsequent successful read is guaranteed to observe the committed update according to the consistency model.

If the system provides eventual consistency, some replicas may temporarily return the older balance of ₹10,000 before synchronization completes.

### Important

Consistency has different meanings in different contexts.

- Distributed systems: consistency models describe the visibility and ordering of updates.
- Relational databases: consistency can refer to preserving constraints and valid states.
- CAP theorem: consistency generally means a strong, linearizable consistency guarantee.

Do not treat all these meanings as interchangeable.

---

## 4. Availability vs Consistency

| Feature | Availability | Consistency |
|---|---|---|
| Main concern | System responsiveness | Correct visibility of data updates |
| Focus | Serving requests | Data correctness across operations |
| Failure example | Server unavailable | Stale replica returns old data |
| Design techniques | Replication, failover, redundancy | Coordination, ordering, synchronization |
| Main question | Can the system respond? | What data will the response reflect? |

### Simple Example

Imagine a banking application with two replicas.

```text
             Application
                  |
          Load Balancer
             /       \
            /         \
       Replica A   Replica B
```

A customer deposits ₹5,000 into Replica A.

Replica A now contains the updated balance, but Replica B has not received the update yet.

Two possible behaviors:

**Strong consistency**

The system ensures that a read reflects the committed update. It may wait for synchronization or reject a request if the required guarantee cannot be met.

**Eventual consistency**

Replica B may temporarily return the old balance but should converge after successful synchronization, assuming updates stop and communication recovers.

---

## 5. What Is the CAP Theorem?

The CAP theorem is a fundamental concept in distributed systems.

It states that when a network partition occurs, a distributed data system must choose between:

- **Consistency (C):** Every successful read reflects the latest committed write according to the consistency guarantee.
- **Availability (A):** Every request to a non-failing node receives a successful response, subject to the CAP definition.
- **Partition Tolerance (P):** The system continues operating despite network communication failures between parts of the system.

### The Three Properties

**1. Consistency**

All clients observe data that respects a single, strongly ordered view of committed operations.

**2. Availability**

Every request received by a non-failing node receives a successful response.

**3. Partition Tolerance**

The system can continue operating when communication between nodes is interrupted.

### CAP Diagram

<box border radius="lg" padding={3} gap={3}>
  <title size="lg" textAlign="center">CAP Theorem</title>
  <svg viewBox="0 0 320 230" width="100%" xmlns="http://www.w3.org/2000/svg">
    <path d="M160 25 L45 195 L275 195 Z" fill="#60A5FA" fillOpacity="0.08" stroke="#60A5FA" strokeWidth="2"/>
    <circle cx="160" cy="25" r="22" fill="#2563EB"/>
    <circle cx="45" cy="195" r="22" fill="#059669"/>
    <circle cx="275" cy="195" r="22" fill="#D97706"/>
    <g fill="white" fontSize="13" fontWeight="bold" textAnchor="middle">
      <text x="160" y="30">C</text>
      <text x="45" y="200">A</text>
      <text x="275" y="200">P</text>
    </g>
    <g fill="currentColor" fontSize="11" textAnchor="middle">
      <text x="160" y="64">Consistency</text>
      <text x="45" y="165">Availability</text>
      <text x="275" y="165">Partition</text>
      <text x="275" y="178">Tolerance</text>
    </g>
  </svg>
  <caption textAlign="center" color="secondary">C, A, and P are the three properties considered by the theorem.</caption>
</box>

### The Most Important CAP Rule

The CAP theorem is specifically about behavior during a network partition.

When a partition occurs, a system cannot guarantee both:
- Strong consistency.
- Availability as defined by CAP.

Partition tolerance is generally necessary in distributed systems because network failures cannot be completely eliminated.

Therefore, the practical trade-off during a partition is often described as CP vs AP.

---

## 6. CP Systems: Consistency and Partition Tolerance

A CP system prioritizes consistency when a network partition occurs.

It may reject, delay, or block some requests rather than return potentially stale data.

### Example

Suppose a distributed banking database has two replicas.

```text
             Banking System
                    |
              Network Split
                 /     \
                /       \
          Replica A   Replica B
```

A network partition prevents the replicas from communicating.

If Replica A accepts a balance update but Replica B cannot learn about it, allowing both sides to independently accept conflicting updates may violate strong consistency.

A CP-oriented system may make one side unavailable for certain operations until safe coordination is possible.

### Characteristics

- Prioritizes strong consistency.
- May reject requests during a partition.
- Uses coordination or quorum-based mechanisms.
- Suitable when stale or conflicting data is unacceptable.

### Typical Use Cases

- Banking transactions.
- Inventory reservation.
- Financial account updates.
- Systems requiring strict coordination.

**Important:** A CP system is not always unavailable. It can provide both consistency and availability when there is no relevant partition.

---

## 7. AP Systems: Availability and Partition Tolerance

An AP system prioritizes availability during a network partition.

It continues responding to requests even if some replicas cannot communicate.

However, different replicas may temporarily contain different values.

### Example

Suppose a social media application stores user likes across multiple regions.

```text
             Social Media App
                    |
              Network Split
                 /     \
                /       \
          Region A    Region B
```

A user likes a post in Region A.

Region A records the update, but Region B cannot receive it immediately.

Region B may temporarily display the old like count.

When communication is restored, the replicas synchronize according to the system's conflict-resolution and replication rules.

### Characteristics

- Prioritizes request availability.
- May return stale data.
- Supports asynchronous replication in common designs.
- Requires conflict-resolution strategies where concurrent writes are possible.

### Typical Use Cases

- Social media feeds.
- Product recommendations.
- Distributed caching.
- Some shopping-cart designs.
- Analytics and read-heavy applications.

**Important:** AP does not mean data can remain inconsistent forever. Many AP-oriented systems aim for eventual convergence.

---

## 8. CP vs AP Comparison

| Feature | CP | AP |
|---|---|---|
| Priority during partition | Consistency | Availability |
| Response behavior | May reject or delay requests | Continues responding where possible |
| Stale reads | Avoided according to consistency guarantee | May occur |
| Conflict handling | Coordination and serialization | Conflict resolution may be required |
| Example use case | Banking balance updates | Social media counters |
| Main trade-off | Some requests may become unavailable | Some responses may be stale |

### Easy Way to Remember

- CP: Do not return data that violates the required consistency guarantee.
- AP: Keep responding, even if some responses may temporarily contain stale data.

---

## 9. What Is Eventual Consistency?

Eventual consistency is a consistency model in which replicas are expected to converge to the same value if no new updates occur and synchronization eventually succeeds.

### Example

A user updates their profile name.

```text
Old Name: Yash
New Name: Yashwant
```

The update reaches one replica first.

| Time | Replica A | Replica B |
|---|---|---|
| T1 | Yash | Yash |
| T2 | Yashwant | Yash |
| T3 | Yashwant | Yashwant |

At T2, the replicas temporarily disagree.

At T3, replication completes and both replicas contain the updated value.

### Characteristics

- Temporary inconsistency is allowed.
- Updates propagate asynchronously in many implementations.
- Replicas can converge.
- Read results may depend on which replica serves the request.

### Advantages

- High availability in suitable architectures.
- Reduced coordination overhead.
- Better support for geographically distributed systems.

### Disadvantages

- Applications may temporarily read stale data.
- Concurrent updates require conflict handling.
- Developers must account for delayed propagation.

---

## 10. Strong Consistency vs Eventual Consistency

| Feature | Strong Consistency | Eventual Consistency |
|---|---|---|
| Read after committed write | Reflects latest committed value according to the model | May return an older value |
| Replication | Requires coordination or equivalent guarantees | Often asynchronous |
| Latency | May be higher | Often lower |
| Availability during partition | May be reduced | Can be prioritized |
| Conflict handling | Strong ordering/coordination | May require conflict resolution |
| Suitable for | Financial balances, reservations | Feeds, counters, some caches |

### Important Distinction

Strong consistency does not necessarily mean every replica physically stores the update at exactly the same instant.

It means the system provides the specified logical consistency guarantee to clients.

---

## 11. Other Consistency Models

Distributed systems can provide several consistency guarantees.

### 11.1 Linearizability

Every operation appears to occur atomically at some point between its invocation and completion.

It also respects real-time ordering: if one operation completes before another begins, the second must observe an order consistent with the first.

Example:

A customer updates a balance and receives confirmation. A later read must reflect that update.

### 11.2 Sequential Consistency

Operations appear in a single sequential order that preserves the program order of each individual process.

Unlike linearizability, it does not necessarily preserve real-time ordering between different processes.

### 11.3 Causal Consistency

Operations that are causally related must be observed in the same order by all clients.

Independent concurrent operations may be observed in different orders.

Example:

A user creates a post, then another user comments on it. The comment should not be observed before the post in a way that violates the causal relationship.

### 11.4 Read-Your-Writes Consistency

A client always sees its own completed writes in subsequent reads.

Example:

After changing a profile name, the same user should see the updated name.

### 11.5 Monotonic Reads

Once a client has observed a particular version of data, subsequent reads should not return an older version.

Example:

A user sees a balance of ₹8,000. A later read should not return an older balance of ₹10,000.

### 11.6 Consistent Prefix Reads

A client sees writes in an order that preserves a consistent prefix of the system's update history.

This is useful when updates have dependencies and should not appear in a causally inconsistent order.

---

## 12. Quorum-Based Consistency

Quorum-based systems use read and write acknowledgments from a subset of replicas.

Suppose a distributed database has three replicas:

```text
Replica A
Replica B
Replica C
```

Let:

- N = Total replicas.
- W = Number of replicas that must acknowledge a write.
- R = Number of replicas consulted for a read.

A commonly used quorum condition is:

\[
R + W > N
\]

This ensures that the read and write replica sets overlap, assuming the system's replica and version semantics support that guarantee.

### Example

Suppose:

\[
N = 3,\quad W = 2,\quad R = 2
\]

Then:

\[
R + W = 4 > 3
\]

The read and write quorums must overlap in at least one replica.

### Important

Quorum overlap alone does not guarantee linearizability in every distributed database.

The actual consistency guarantee also depends on:
- Replica version handling.
- Conflict resolution.
- Read repair.
- Leader or consensus protocols.
- Write acknowledgment semantics.

---

## 13. Replication and Consistency

Replication means maintaining copies of data across multiple nodes.

It improves availability and can support scalability, but introduces synchronization challenges.

### Synchronous Replication

A write waits for acknowledgments from the required replicas before being considered successful.

Advantages:
- Stronger coordination between replicas.
- Can support strong consistency guarantees.

Disadvantages:
- Higher write latency.
- Network failures may block writes.
- Availability can decrease during partitions.

### Asynchronous Replication

The primary system acknowledges a write before all replicas have received it.

Advantages:
- Lower write latency in many architectures.
- Better write availability in some designs.

Disadvantages:
- Replicas may temporarily return stale data.
- Failover may expose data loss if acknowledged writes were not replicated.
- Conflict handling may be required.

### Comparison

| Feature | Synchronous | Asynchronous |
|---|---|---|
| Acknowledgment | Waits for required replica acknowledgments | Does not wait for all replicas |
| Latency | Often higher | Often lower |
| Replica lag | Reduced according to configured acknowledgment policy | Possible |
| Partition behavior | May block writes | May allow writes on some nodes |
| Data loss risk on failover | Depends on acknowledgment and durability guarantees | Can be higher |

Synchronous replication does not automatically guarantee consistency across every replica or every failure scenario.

---

## 14. Availability in Data Engineering

Availability is important in data platforms because business applications depend on reliable access to datasets and processing services.

### Example Architecture

```text
              Data Sources
                   |
                   v
              Ingestion Layer
                   |
                   v
              Raw Storage
                   |
                   v
           Processing Platform
                   |
                   v
             Curated Tables
                   |
                   v
            BI / Applications
```

If the processing platform becomes unavailable:
- Data ingestion may be delayed.
- Curated tables may become stale.
- Reports may not refresh.
- Downstream applications may receive old data.

### Improving Availability

1. Deploy services across availability zones where supported.
2. Use redundant infrastructure.
3. Configure health checks and automatic failover.
4. Use durable storage.
5. Monitor service availability.
6. Implement retry and recovery mechanisms.
7. Maintain backups and disaster recovery plans.
8. Define RPO and RTO requirements.

### High Availability vs Disaster Recovery

| Concept | Purpose |
|---|---|
| High Availability | Minimize service interruption during failures |
| Disaster Recovery | Restore service after major incidents |
| Backup | Preserve recoverable copies of data |
| Failover | Switch operations to another resource |

High availability and disaster recovery are related but not interchangeable.

---

## 15. Consistency in Data Engineering

Consistency matters when data is distributed across storage systems, databases, and processing engines.

### Example

Suppose an orders pipeline writes data into three layers:

```text
Source Database
      |
      v
Bronze Table
      |
      v
Silver Table
      |
      v
Gold Table
```

A new order is successfully written to Bronze but the Silver transformation fails.

The datasets are temporarily at different processing states.

The system must track progress and ensure that downstream consumers do not incorrectly assume every layer contains the latest data.

### Ways to Maintain Data Consistency

- Use transactions where supported.
- Use idempotent writes.
- Apply deterministic transformation logic.
- Maintain reliable checkpoints.
- Use correct watermark management.
- Validate source-to-target reconciliation.
- Avoid uncontrolled concurrent writes.
- Use schema and data quality checks.
- Publish data only after required processing succeeds.

### Important

A successful write to Bronze does not mean the entire pipeline has completed successfully.

Track processing status and data freshness separately for each layer.

---

## 16. CAP Theorem vs ACID Consistency

These concepts are often confused.

### CAP Consistency

In CAP, consistency refers to a strong distributed consistency guarantee, commonly explained through linearizability.

### ACID Consistency

In ACID transactions, consistency means a transaction preserves database constraints and valid states.

### Comparison

| Feature | CAP Consistency | ACID Consistency |
|---|---|---|
| Context | Distributed systems | Database transactions |
| Focus | Visibility and ordering of updates | Valid database state |
| Example | Reads reflect the latest committed write | Foreign key and check constraints remain valid |
| Main concern | Distributed replicas and partitions | Transaction correctness |

A database can provide ACID transactions while also offering different distributed consistency or replication guarantees.

---

## 17. Practical Interview Scenario

**Question: You are designing a distributed inventory system. Two users attempt to purchase the last available item at the same time. How would you handle consistency and availability?**

### Answer

I would first identify the business requirement.

For inventory reservation, overselling should generally be prevented. Therefore, correctness is more important than accepting every request during a network partition.

I would use a strongly coordinated inventory update, such as an atomic conditional update or transaction.

For example:

```sql
UPDATE inventory
SET available_quantity = available_quantity - 1
WHERE product_id = 101
  AND available_quantity > 0;
```

The application should check the affected row count:

- One affected row means the reservation succeeded.
- Zero affected rows means the item was unavailable or the condition was not met.

I would also use transaction boundaries and idempotency keys to prevent duplicate purchase requests from reserving multiple items.

During a network partition, the system may reject or delay some reservation requests rather than risk overselling.

For less critical features, such as product recommendations or view counters, eventual consistency may be acceptable.

This design uses stronger consistency for critical inventory operations while allowing more availability-oriented behavior for less critical features.

---

## 18. Common Interview Questions and Answers

### Q1. What is availability?

Availability is the ability of a system to respond to requests and remain operational.

### Q2. What is consistency in distributed systems?

Consistency defines how reads and writes behave across distributed nodes and what updates clients are guaranteed to observe.

### Q3. What is the CAP theorem?

The CAP theorem states that during a network partition, a distributed system cannot guarantee both strong consistency and availability as defined by CAP.

### Q4. What are the three CAP properties?

Consistency, Availability, and Partition Tolerance.

### Q5. What is a network partition?

A network partition occurs when some nodes cannot communicate with other nodes in a distributed system.

### Q6. What is a CP system?

A system that prioritizes consistency during a network partition, potentially rejecting or delaying some requests.

### Q7. What is an AP system?

A system that prioritizes availability during a network partition and may temporarily return stale or conflicting data.

### Q8. What is eventual consistency?

A model in which replicas are expected to converge if updates stop and synchronization eventually succeeds.

### Q9. What is strong consistency?

A guarantee that reads reflect the latest committed update according to the system's strong consistency model.

### Q10. What is linearizability?

A consistency guarantee in which operations appear atomic and respect real-time ordering.

### Q11. What is quorum?

A required number of replica acknowledgments or responses used to determine successful reads or writes.

### Q12. Explain the quorum formula.

A common quorum condition is `R + W > N`, where R is the read quorum, W is the write quorum, and N is the number of replicas.

### Q13. What is synchronous replication?

A replication method in which a write waits for acknowledgments from the required replicas before being acknowledged.

### Q14. What is asynchronous replication?

A method in which a write can be acknowledged before all replicas have received the update.

### Q15. What is the difference between high availability and disaster recovery?

High availability minimizes service interruption. Disaster recovery restores service after a major failure.

### Q16. How does consistency affect data pipelines?

It determines how updates and processing results become visible across storage layers, databases, and downstream consumers.

### Q17. Is eventual consistency always bad?

No. It is suitable when temporary stale reads are acceptable and high availability or lower coordination overhead is more important.

### Q18. Does CAP mean a system can only choose two of three properties all the time?

No. The trade-off applies when a network partition occurs. Without a partition, a system can provide both strong consistency and availability.

---

## 19. Quick Revision

| Concept | Key Meaning |
|---|---|
| Availability | Ability to respond to requests |
| Consistency | Defined correctness and visibility of data updates |
| CAP | Consistency, Availability, Partition Tolerance |
| Network Partition | Nodes cannot communicate reliably |
| CP | Prioritizes consistency during partition |
| AP | Prioritizes availability during partition |
| Eventual Consistency | Replicas eventually converge |
| Linearizability | Atomic operations respecting real-time order |
| Quorum | Required number of replica responses |
| Synchronous Replication | Waits for required replica acknowledgments |
| Asynchronous Replication | Replication continues after acknowledgment |
| High Availability | Minimizes service downtime |
| Disaster Recovery | Restores service after major failures |

### Remember These Five Points

1. Availability means the system can respond to requests.
2. Consistency describes the guarantees provided for reads and writes.
3. CAP trade-offs matter specifically during network partitions.
4. CP may reject requests; AP may return stale data.
5. Choose consistency guarantees based on business requirements.

---

## 20. Self-Assessment

Try answering these questions without looking at the notes.

1. What is availability?
2. How do you calculate availability?
3. What is consistency in distributed systems?
4. Explain availability vs consistency.
5. What is the CAP theorem?
6. What is a network partition?
7. Why can't a distributed system guarantee both CAP consistency and availability during a partition?
8. What is a CP system?
9. What is an AP system?
10. Explain eventual consistency with an example.
11. What is the difference between strong and eventual consistency?
12. What is linearizability?
13. What is causal consistency?
14. Explain quorum-based consistency.
15. What does `R + W > N` mean?
16. What is the difference between synchronous and asynchronous replication?
17. What is the difference between CAP consistency and ACID consistency?
18. How would you design a consistent inventory reservation system?
19. How can availability be improved in a data platform?
20. How do you maintain consistency across Bronze, Silver, and Gold layers?

**Interview readiness:** Be able to explain CAP clearly, distinguish CP from AP, describe eventual consistency and quorum reads/writes, and apply these concepts to practical systems such as banking, inventory, social media, and data pipelines.

