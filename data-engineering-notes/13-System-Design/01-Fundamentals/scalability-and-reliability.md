

# Scalability and Reliability in System Design

## 1. Introduction

Scalability and reliability are two fundamental concepts in system design.

A well-designed system should handle increasing workloads while continuing to operate correctly, even when failures occur.

- **Scalability:** The ability of a system to handle increasing workloads by adding or adjusting resources.
- **Reliability:** The ability of a system to perform its intended functions correctly over time.

### Example

Suppose an e-commerce application initially supports 10,000 users.

Over time, it grows to 10 million users.

The system must:
- Handle more requests.
- Process more orders.
- Store more data.
- Maintain acceptable response times.
- Continue operating when servers fail.

Scalability helps the system handle growth. Reliability helps it continue working correctly.

---

## 2. What Is Scalability?

Scalability is the ability of a system to handle increasing workloads without unacceptable degradation in performance.

A scalable system can support growth in:
- Users.
- API requests.
- Database queries.
- Data volume.
- Transactions.
- Processing workloads.

### Example

Suppose a data platform initially processes 100 GB of data daily.

After business growth, it must process 5 TB daily.

A scalable architecture can increase processing and storage capacity to support this workload.

### Why Is Scalability Important?

1. Supports business growth.
2. Handles traffic spikes.
3. Improves processing capacity.
4. Reduces performance bottlenecks.
5. Supports increasing data volumes.
6. Helps maintain user experience.
7. Enables infrastructure expansion.

---

## 3. Types of Scalability

There are two major approaches:

1. Vertical scaling.
2. Horizontal scaling.

### 3.1 Vertical Scaling (Scale Up)

Vertical scaling means increasing the resources of an existing machine.

Resources may include:
- CPU.
- RAM.
- Storage.
- Network capacity.

### Example

A database server initially has:

```text
CPU: 4 Cores
RAM: 16 GB
Storage: 500 GB
```

After increasing workload:

```text
CPU: 16 Cores
RAM: 64 GB
Storage: 2 TB
```

The same server now has more resources.

<AsyncImage query="vertical scaling scale up single server before after CPU RAM architecture diagram" aspectRatio="16:9" width="100%" />

### Advantages

- Simple to implement.
- Often requires fewer architectural changes.
- Useful for databases and applications with limited horizontal scaling support.
- Easier to manage in some environments.

### Disadvantages

- Hardware has an upper limit.
- Larger machines can be expensive.
- May require downtime depending on the platform.
- A single machine can remain a single point of failure.

### 3.2 Horizontal Scaling (Scale Out)

Horizontal scaling means adding more machines or instances to distribute workload.

### Example

Initially:

```text
          Application
               |
            Server 1
```

After scaling:

```text
              Users
                |
          Load Balancer
          /     |     \
         /      |      \
    Server 1 Server 2 Server 3
```

Requests are distributed across multiple servers.

<AsyncImage query="horizontal scaling scale out multiple application servers behind load balancer architecture diagram" aspectRatio="16:9" width="100%" />

### Advantages

- Supports large workloads.
- Can improve availability with proper redundancy.
- Allows incremental capacity expansion.
- Supports distributed processing.
- Can scale across multiple machines or availability zones.

### Disadvantages

- More architectural complexity.
- Requires workload distribution.
- May introduce network communication overhead.
- Distributed data consistency becomes more challenging.
- Requires monitoring and coordination across instances.

### Vertical vs Horizontal Scaling

| Feature | Vertical Scaling | Horizontal Scaling |
|---|---|---|
| Meaning | Add resources to one machine | Add more machines |
| Method | Scale up | Scale out |
| Infrastructure | Larger instance | Multiple instances |
| Upper limit | Hardware or service limit | Depends on architecture and platform |
| Complexity | Often simpler | Usually more complex |
| Availability | Single machine may remain a failure point | Can improve availability with redundancy |
| Example | Increase RAM | Add application servers |

**Remember:**

- Vertical = Increase machine capacity.
- Horizontal = Increase machine count.

---

## 4. What Is Elasticity?

Elasticity is the ability of a system to automatically or dynamically adjust resources according to workload changes.

### Example

An e-commerce application experiences increased traffic during a festival sale.

Normal traffic:

```text
Application Instances = 3
```

Peak traffic:

```text
Application Instances = 10
```

After the sale:

```text
Application Instances = 3
```

The infrastructure adjusts according to demand.

### Scalability vs Elasticity

| Scalability | Elasticity |
|---|---|
| Ability to handle workload growth | Ability to adjust resources as demand changes |
| Can be manual or automatic | Usually dynamic or automatic |
| Supports long-term growth | Responds to changing workload |
| Example: Add more servers | Automatically add and remove servers |

Elasticity is particularly useful in cloud environments.

---

## 5. What Is Reliability?

Reliability is the ability of a system to perform its intended functions correctly and consistently over time.

A reliable system should:
- Produce correct results.
- Avoid unexpected failures.
- Preserve data.
- Recover from errors.
- Maintain expected behavior.

### Example

Suppose a banking application processes 100,000 transactions daily.

A reliable system must:
- Avoid incorrect balance calculations.
- Prevent duplicate transactions.
- Preserve committed transactions.
- Handle database failures.
- Recover safely after interruptions.

Reliability is not simply about keeping servers online. A system that responds quickly but returns incorrect financial results is not reliable.

---

## 6. Reliability vs Availability

These concepts are related but different.

**Availability** measures whether a system is operational and able to respond.

**Reliability** measures whether the system performs correctly over time.

| Feature | Availability | Reliability |
|---|---|---|
| Main focus | Service accessibility | Correct operation over time |
| Question | Is the system available? | Is the system working correctly? |
| Measurement | Uptime percentage | Failure frequency and correct operation |
| Example failure | Server is unavailable | Server returns incorrect data |

### Example

A data pipeline completes successfully every day but produces incorrect revenue totals.

- Availability: The pipeline is available and runs.
- Reliability: The pipeline is unreliable because its output is incorrect.

Both properties matter in production systems.

---

## 7. Fault Tolerance

Fault tolerance is the ability of a system to continue providing required functionality when one or more components fail.

### Example

An application runs on three servers.

If one server fails, the other two continue handling requests.

```text
             Load Balancer
                  |
          +-------+-------+
          |       |       |
       Server 1 Server 2 Server 3
          |       |       |
        Active  Active  Failed
```

The application continues operating because it has redundant capacity.

### Common Fault-Tolerance Techniques

1. Redundant servers.
2. Data replication.
3. Load balancing.
4. Automatic failover.
5. Retry mechanisms.
6. Health checks.
7. Backups and recovery.
8. Message queues.
9. Multi-zone deployment.

### Important

Fault tolerance does not mean failures never occur.

It means the system is designed to handle certain failures without unacceptable service disruption or data loss.

---

## 8. Redundancy

Redundancy means maintaining additional resources or copies so that the system can continue operating when a component fails.

### Example

Instead of running one database server:

```text
Application
    |
Primary Database
```

We can use a primary database and a replica:

```text
             Application
                  |
             Primary DB
                  |
             Replica DB
```

If the primary fails, the replica may be promoted depending on the database's failover configuration.

### Types of Redundancy

- Hardware redundancy.
- Server redundancy.
- Network redundancy.
- Storage redundancy.
- Database redundancy.
- Geographic redundancy.

### Advantages

- Reduces single points of failure.
- Improves service continuity.
- Supports disaster recovery.
- Helps protect data.

### Challenges

- Additional infrastructure cost.
- Replication lag.
- Failover complexity.
- Data synchronization.

---

## 9. Load Balancing

A load balancer distributes incoming traffic across multiple servers.

### Example

```text
                 Users
                   |
                   v
              Load Balancer
              /     |     \
             /      |      \
        Server 1 Server 2 Server 3
```

Instead of sending every request to one server, the load balancer distributes requests.

### Benefits

- Improves traffic distribution.
- Prevents one server from handling all requests.
- Supports horizontal scaling.
- Can route requests away from unhealthy instances.
- Helps improve availability.

### Common Load Balancing Algorithms

| Algorithm | Description |
|---|---|
| Round Robin | Distributes requests sequentially |
| Weighted Round Robin | Distributes according to assigned weights |
| Least Connections | Sends requests to the server with fewer active connections |
| IP Hash | Uses client IP information to select a server |
| Least Response Time | Uses response-time information where supported |

### Health Checks

A load balancer can periodically check server health.

If a server becomes unhealthy, it can stop routing new traffic to that server.

**Important:** A load balancer alone does not guarantee high availability. The load balancer itself and the backend infrastructure must also be designed for redundancy.

---

## 10. Replication and High Availability

Replication maintains copies of data across multiple servers.

It helps improve availability and can support read scaling.

### Primary-Replica Architecture

```text
             Application
                  |
             Primary DB
              /      \
             /        \
        Replica 1   Replica 2
```

The primary handles writes, while replicas may serve reads.

### Benefits

- Data redundancy.
- Read scaling.
- Recovery options.
- Improved service continuity.

### Challenges

- Replication lag.
- Failover management.
- Consistency requirements.
- Additional storage and network cost.

### Synchronous vs Asynchronous Replication

| Synchronous | Asynchronous |
|---|---|
| Waits for required replica acknowledgments | Does not wait for every replica |
| Often higher write latency | Often lower write latency |
| Can provide stronger durability guarantees | Replicas may temporarily lag |
| May block writes during failures | May risk losing unreplicated acknowledged writes during failover |

The actual guarantees depend on the database and its replication configuration.

---

## 11. Failure Recovery

Failure recovery is the process of restoring a system after an unexpected failure.

Common failure scenarios:
- Server crash.
- Database failure.
- Network interruption.
- Storage failure.
- Application bug.
- Corrupted data.
- Cloud service outage.

### Recovery Techniques

**1. Retry**

Repeat a failed operation when the error is temporary and retrying is safe.

**2. Failover**

Switch operations to a healthy standby resource.

**3. Backup and Restore**

Restore data from a previously created backup.

**4. Checkpointing**

Save processing progress so that a workload can resume from a recoverable point.

**5. Reprocessing**

Run failed or incomplete data-processing operations again.

### Important

Retries must be designed carefully.

For example, retrying a payment request without idempotency protection can result in duplicate charges.

---

## 12. Reliability Metrics

Reliability can be evaluated using several operational metrics.

### 12.1 MTBF (Mean Time Between Failures)

MTBF measures the average operating time between failures for a repairable system.

\[
MTBF = \frac{Total\ Operating\ Time}{Number\ of\ Failures}
\]

Example:

A system operates for 1,000 hours and experiences 5 failures.

\[
MTBF = \frac{1000}{5} = 200\ hours
\]

The average operating time between failures is 200 hours under this measurement method.

### 12.2 MTTR (Mean Time To Repair/Restore)

MTTR measures the average time taken to restore a system after failure.

\[
MTTR = \frac{Total\ Repair\ Time}{Number\ of\ Failures}
\]

Example:

Five failures require a total of 10 hours to restore.

\[
MTTR = \frac{10}{5} = 2\ hours
\]

Average restoration time is 2 hours.

Organizations may define MTTR differently, including repair, recovery, or resolution time. Always clarify the measurement definition.

### 12.3 MTTF (Mean Time To Failure)

MTTF measures the average operating time until a non-repairable component fails.

It is commonly used for components that are replaced rather than repaired.

### 12.4 Relationship Between MTBF and MTTR

For a simple repairable system with alternating operating and repair periods:

\[
Availability =
\frac{MTBF}{MTBF + MTTR}
\]

Example:

- MTBF = 200 hours.
- MTTR = 2 hours.

\[
Availability =
\frac{200}{200+2}
\times100
\]

\[
Availability \approx 99.01\%
\]

This is a simplified inherent-availability calculation. Real service availability can also depend on planned maintenance, dependencies, redundancy, and measurement rules.

---

## 13. High Availability vs Fault Tolerance

| Feature | High Availability | Fault Tolerance |
|---|---|---|
| Main goal | Minimize downtime | Continue operating through specified failures |
| Failure behavior | May involve a short interruption during failover | Designed to avoid interruption for covered failures |
| Architecture | Redundancy and recovery | Redundancy and fault handling |
| Cost | Depends on target | Can be higher for strict requirements |
| Example | Standby database promoted after failure | Redundant components continue without service interruption |

A system can be highly available without being completely fault tolerant.

For example, a database may experience a short interruption while failing over to a replica but still meet its availability target.

---

## 14. Designing for Scalability and Reliability in Data Engineering

Data engineering platforms process large amounts of data and must continue operating despite workload growth and component failures.

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
          Distributed Processing
                   |
                   v
            Curated Storage
                   |
                   v
              BI / Analytics
```

### Improving Scalability

- Use distributed processing with Spark.
- Partition large datasets appropriately.
- Scale compute resources according to workload.
- Optimize joins and shuffle operations.
- Use columnar formats such as Parquet.
- Avoid excessive small files.
- Use autoscaling where appropriate.
- Separate compute from storage when supported.

### Improving Reliability

- Implement retries for transient failures.
- Use idempotent writes.
- Maintain checkpoints for supported streaming workloads.
- Validate data quality.
- Monitor pipeline execution.
- Maintain recoverable storage.
- Use incremental processing carefully.
- Design safe backfills and reruns.

### Example: PySpark Processing

Suppose a pipeline processes 500 GB daily.

Initially, one small cluster takes 8 hours.

After data growth, the same workload reaches 2 TB.

Possible improvements:

1. Increase cluster capacity.
2. Increase parallelism where appropriate.
3. Optimize partition sizes.
4. Reduce unnecessary shuffles.
5. Broadcast suitable small tables.
6. Address data skew.
7. Use efficient file formats.
8. Monitor Spark UI metrics.

However, simply increasing the number of workers does not guarantee better performance. The workload may be limited by skew, I/O, shuffle, or the source system.

---

## 15. Scalability vs Reliability

| Feature | Scalability | Reliability |
|---|---|---|
| Main focus | Workload growth | Correct and dependable operation |
| Main question | Can it handle more workload? | Can it operate correctly over time? |
| Techniques | Scaling, partitioning, load balancing | Testing, redundancy, recovery |
| Common risk | Performance degradation | Failures or incorrect results |
| Example | Processing 10 TB instead of 1 TB | Recovering after a failed job |

### Important

A system can be scalable but unreliable.

Example: A distributed pipeline processes billions of records quickly but frequently generates duplicate output.

A system can also be reliable but not scalable.

Example: A single server processes every request correctly but becomes too slow when traffic increases.

A good production system needs both.

---

## 16. Practical Interview Scenario

**Question: Your data pipeline processes 100 GB daily. Business growth increases the workload to 5 TB daily. The pipeline also fails occasionally. How would you improve scalability and reliability?**

### Answer

I would address scalability and reliability separately.

**For scalability:**

1. Analyze the current processing bottlenecks.
2. Review Spark UI metrics for CPU, shuffle, spill, skew, and task duration.
3. Optimize joins and transformations.
4. Use appropriate partitioning and file formats.
5. Scale cluster resources based on measured workload.
6. Consider incremental processing rather than repeatedly processing all historical data.
7. Monitor processing throughput and completion time.

**For reliability:**

1. Identify common failure causes.
2. Configure retries for transient failures.
3. Use idempotent writes to support safe reruns.
4. Maintain checkpoints for streaming workloads.
5. Validate output data before publishing.
6. Monitor pipeline failures and data freshness.
7. Maintain recovery procedures and backfill capability.

I would also define performance and reliability targets so that improvements can be measured.

---

## 17. Common Mistakes

1. Assuming vertical scaling can continue indefinitely.
2. Assuming horizontal scaling automatically improves performance.
3. Confusing scalability with elasticity.
4. Confusing availability with reliability.
5. Assuming replication automatically guarantees strong consistency.
6. Assuming a load balancer eliminates every single point of failure.
7. Retrying every failure without checking whether it is transient.
8. Ignoring idempotency during recovery.
9. Increasing Spark workers without analyzing bottlenecks.
10. Ignoring monitoring and observability.
11. Treating backups as a complete high-availability solution.
12. Ignoring the cost of redundancy and scaling.

---

## 18. Interview Questions and Answers

### Q1. What is scalability?

Scalability is the ability of a system to handle increasing workloads by increasing or adjusting resources.

### Q2. What are the two types of scaling?

Vertical scaling and horizontal scaling.

### Q3. What is vertical scaling?

Increasing the resources of an existing machine, such as CPU, RAM, and storage.

### Q4. What is horizontal scaling?

Adding more machines or instances to distribute workload.

### Q5. What is elasticity?

The ability to dynamically adjust resources according to workload demand.

### Q6. What is reliability?

The ability of a system to perform its intended functions correctly over time.

### Q7. What is fault tolerance?

The ability of a system to continue providing required functionality when specified components fail.

### Q8. What is redundancy?

Maintaining additional resources or data copies to reduce the impact of component failures.

### Q9. What is load balancing?

Distributing incoming requests across multiple servers or resources.

### Q10. What is the difference between reliability and availability?

Availability measures whether a system can respond. Reliability measures whether it performs correctly and dependably over time.

### Q11. What is MTBF?

Mean Time Between Failures measures the average operating time between failures.

### Q12. What is MTTR?

Mean Time To Repair or Restore measures the average time required to recover from failures, depending on the organization's definition.

### Q13. What is MTTF?

Mean Time To Failure measures the average operating time until a non-repairable component fails.

### Q14. What is high availability?

High availability is the design goal of minimizing service downtime through redundancy, failover, and recovery mechanisms.

### Q15. What is the difference between high availability and fault tolerance?

High availability minimizes downtime and may allow brief failover interruptions. Fault tolerance aims to continue operation through specified failures without interruption.

### Q16. Why is replication important?

Replication maintains additional copies of data, supporting availability, redundancy, and sometimes read scaling.

### Q17. Does horizontal scaling always improve performance?

No. Network overhead, coordination, data skew, and other bottlenecks can limit the benefits.

### Q18. How can you improve the reliability of a data pipeline?

Use idempotent writes, safe retries, checkpoints where applicable, data quality checks, monitoring, and recovery procedures.

### Q19. What is the relationship between MTBF and MTTR?

For a simple repairable system, higher MTBF and lower MTTR generally improve availability.

### Q20. Can a system be scalable but unreliable?

Yes. It can handle increasing workloads while producing incorrect results or experiencing frequent failures.

---

## 19. Quick Revision

| Concept | Key Meaning |
|---|---|
| Scalability | Handle increasing workloads |
| Vertical Scaling | Increase resources of one machine |
| Horizontal Scaling | Add more machines |
| Elasticity | Dynamically adjust resources |
| Reliability | Operate correctly over time |
| Availability | Respond to requests |
| Fault Tolerance | Continue through specified failures |
| Redundancy | Maintain extra resources or copies |
| Load Balancing | Distribute incoming traffic |
| Replication | Maintain multiple data copies |
| MTBF | Average operating time between failures |
| MTTR | Average restoration or repair time |
| MTTF | Average time until a non-repairable component fails |
| High Availability | Minimize downtime |

### Important Formulas

**Availability:**

\[
Availability =
\frac{Uptime}{Total\ Time}\times100
\]

**MTBF:**

\[
MTBF =
\frac{Total\ Operating\ Time}{Number\ of\ Failures}
\]

**MTTR:**

\[
MTTR =
\frac{Total\ Repair\ Time}{Number\ of\ Failures}
\]

**Simplified Availability:**

\[
Availability =
\frac{MTBF}{MTBF+MTTR}
\]

---

## 20. Self-Assessment

Try answering these questions without looking at the notes.

1. What is scalability?
2. Why is scalability important?
3. Explain vertical scaling with an example.
4. Explain horizontal scaling with an example.
5. What is the difference between vertical and horizontal scaling?
6. What is elasticity?
7. Explain scalability vs elasticity.
8. What is reliability?
9. Explain reliability vs availability.
10. What is fault tolerance?
11. What is redundancy?
12. How does a load balancer work?
13. What is the difference between synchronous and asynchronous replication?
14. What is high availability?
15. Explain high availability vs fault tolerance.
16. What are MTBF, MTTR, and MTTF?
17. How do you calculate availability using MTBF and MTTR?
18. How can Spark pipelines be scaled?
19. How can you improve data pipeline reliability?
20. Can a system be scalable but unreliable? Explain with an example.

**Interview readiness:** Be able to explain vertical and horizontal scaling, distinguish scalability from elasticity, understand reliability and availability, calculate MTBF and MTTR, and apply these concepts to distributed applications and data engineering pipelines.

