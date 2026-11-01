

# Capacity Estimation in System Design

## 1. Introduction

Capacity estimation is the process of calculating the resources required to support an application's expected workload.

Before designing a system, we need to estimate:

- How many users will use the application?
- How many requests will arrive per second?
- How much data will be generated?
- How much storage will be required?
- How much network bandwidth will be needed?
- How many servers or processing resources will be required?

Capacity estimation helps engineers design systems that can handle expected traffic without unnecessary infrastructure costs.

### Example

Suppose we are designing an e-commerce application.

We need to estimate:
- Daily active users.
- Orders per day.
- API requests per second.
- Database storage.
- Image and product data storage.
- Network bandwidth.
- Processing capacity.

These estimates help us choose an appropriate architecture.

---

## 2. Why Is Capacity Estimation Important?

Capacity estimation is important because an application must support its expected workload.

Without proper estimation, two major problems can occur.

### Under-provisioning

The system has fewer resources than required.

Possible consequences:
- Slow API responses.
- High CPU and memory usage.
- Database overload.
- Request timeouts.
- Failed pipelines.
- Service downtime.

### Over-provisioning

The system has more resources than required.

Possible consequences:
- Higher infrastructure costs.
- Underutilized servers.
- Unnecessary cloud spending.
- Poor resource efficiency.

### Main Objectives

1. Estimate expected workload.
2. Calculate storage requirements.
3. Estimate compute resources.
4. Estimate network requirements.
5. Identify bottlenecks.
6. Plan for future growth.
7. Balance performance and cost.

---

## 3. Important Terms

Before performing calculations, understand these terms.

| Term | Meaning |
|---|---|
| DAU | Daily Active Users |
| MAU | Monthly Active Users |
| QPS | Queries Per Second |
| RPS | Requests Per Second |
| TPS | Transactions Per Second |
| Peak Traffic | Highest expected traffic during a period |
| Throughput | Amount of work completed per unit of time |
| Latency | Time taken to complete a request |
| Payload Size | Amount of data transferred in a request or response |
| Storage Capacity | Amount of data the system must store |
| Bandwidth | Data transfer capacity per unit of time |
| Replication Factor | Number of copies maintained for data |

### QPS vs RPS vs TPS

**QPS:** Number of queries processed per second.

**RPS:** Number of requests received per second.

**TPS:** Number of transactions completed per second.

These values are not necessarily equal.

For example, one API request may execute three database queries and one transaction.

---

## 4. Important Units and Conversions

Capacity estimation frequently uses bytes, seconds, and time conversions.

### Time Conversions

| Unit | Value |
|---|---:|
| 1 minute | 60 seconds |
| 1 hour | 3,600 seconds |
| 1 day | 86,400 seconds |
| 1 week | 604,800 seconds |
| 1 month (30 days) | 2,592,000 seconds |
| 1 year (365 days) | 31,536,000 seconds |

### Data Size Conversions

| Unit | Value |
|---|---:|
| 1 KB | 1,024 bytes |
| 1 MB | 1,024 KB |
| 1 GB | 1,024 MB |
| 1 TB | 1,024 GB |
| 1 PB | 1,024 TB |

**Important:** Cloud providers and network tools may use decimal units:

- 1 KB = 1,000 bytes
- 1 MB = 1,000 KB
- 1 GB = 1,000 MB

Always state whether you are using decimal or binary units. Do not mix them in one calculation.

For quick interview estimation, engineers often use rounded values.

---

## 5. Estimating Daily Active Users

Daily Active Users represent the number of unique users who use an application during a day.

### Example

Suppose an application has:

- Total registered users: 10 million.
- Daily active user percentage: 20%.

Then:

\[
DAU = 10,000,000 \times 20\%
\]

\[
DAU = 2,000,000
\]

The application has approximately 2 million daily active users.

### Why DAU Matters

DAU helps estimate:
- Daily API requests.
- Daily transactions.
- Data generation.
- Storage growth.
- Application workload.

**Important:** Registered users and active users are different. Do not assume every registered user accesses the application daily.

---

## 6. Estimating Requests Per Second

Requests per second help estimate application traffic.

### Formula

\[
Average\ RPS =
\frac{Total\ Daily\ Requests}{86,400}
\]

### Example

Suppose:
- Daily active users = 2 million.
- Average requests per user per day = 50.

Total daily requests:

\[
2,000,000 \times 50
= 100,000,000
\]

Average RPS:

\[
\frac{100,000,000}{86,400}
\approx 1,157
\]

The application receives approximately 1,157 requests per second on average.

### Important

Average RPS does not represent peak traffic.

Applications may receive much higher traffic during:
- Sales events.
- Product launches.
- Festivals.
- Promotional campaigns.
- Breaking news.
- Scheduled business events.

---

## 7. Estimating Peak Traffic

Peak traffic is the highest expected request rate during a particular period.

A simple approximation is:

\[
Peak\ RPS = Average\ RPS \times Peak\ Factor
\]

### Example

Suppose:

- Average RPS = 1,157.
- Peak factor = 3.

Then:

\[
Peak\ RPS = 1,157 \times 3
\]

\[
Peak\ RPS = 3,471
\]

The system should be designed to handle approximately 3,471 requests per second under this assumption.

### What Is a Peak Factor?

A peak factor represents how much higher peak traffic is than average traffic.

Example assumptions:

| Application Pattern | Illustrative Peak Factor |
|---|---:|
| Relatively stable traffic | 1.5–2 |
| Typical consumer application | 2–3 |
| Event-driven traffic | 3–5 |
| Major promotional event | 5 or more |

These are planning assumptions, not universal industry standards. Real peak factors should come from traffic measurements or workload forecasts.

---

## 8. Read and Write Traffic Estimation

Most applications perform both read and write operations.

Examples:

**Read operations:**
- Viewing products.
- Reading user profiles.
- Checking order status.
- Loading dashboards.

**Write operations:**
- Creating orders.
- Updating profiles.
- Making payments.
- Posting comments.

### Example

Suppose peak traffic is 3,471 requests per second.

Assume:
- 80% reads.
- 20% writes.

Read requests:

\[
3,471 \times 0.80 = 2,776.8
\]

Write requests:

\[
3,471 \times 0.20 = 694.2
\]

Approximately:

- 2,777 read requests/second.
- 694 write requests/second.

### Why Does This Matter?

Read and write workloads have different resource requirements.

Read-heavy systems may benefit from:
- Caching.
- Read replicas.
- CDN.
- Query optimization.

Write-heavy systems may require:
- Efficient transaction handling.
- Write scaling.
- Partitioning.
- Message queues.
- Batch processing.

---

## 9. Estimating Storage Requirements

Storage estimation calculates how much data an application generates and retains.

### Basic Formula

\[
Daily\ Storage =
Records\ Per\ Day \times Average\ Record\ Size
\]

### Example

Suppose an e-commerce application generates:

- 5 million orders per day.
- Average order record size = 2 KB.

Daily storage:

\[
5,000,000 \times 2\ KB
= 10,000,000\ KB
\]

Using decimal units:

\[
10,000,000\ KB = 10\ GB
\]

Therefore, the application generates approximately 10 GB of order data daily.

### Monthly Storage

\[
Monthly\ Storage =
Daily\ Storage \times 30
\]

\[
10 \times 30 = 300\ GB
\]

### Yearly Storage

\[
Yearly\ Storage =
Daily\ Storage \times 365
\]

\[
10 \times 365 = 3,650\ GB
\]

Approximately 3.65 TB of raw order records per year.

### Important

This estimate represents only the assumed order-record payload.

Actual storage can be higher because of:
- Database indexes.
- Transaction logs.
- Metadata.
- Additional columns.
- Replication.
- Backups.
- Temporary files.
- Data format and compression differences.

---

## 10. Estimating Storage with Replication

Distributed systems often maintain multiple copies of data for availability and durability.

### Formula

\[
Total\ Replicated\ Storage =
Raw\ Storage \times Replication\ Factor
\]

### Example

Suppose:
- Raw storage = 3.65 TB.
- Replication factor = 3.

Then:

\[
3.65 \times 3 = 10.95\ TB
\]

The system may require approximately 10.95 TB for three full copies, excluding metadata and other overhead.

### Important

Do not automatically multiply cloud object storage requirements by three.

Some cloud storage redundancy options manage replication internally. Their billing and capacity accounting differ from manually maintaining three copies.

---

## 11. Estimating Media Storage

Applications may store images, videos, and documents.

Media files can consume much more storage than ordinary database records.

### Example

Suppose a social media application has:

- 100,000 image uploads per day.
- Average image size = 500 KB.

Daily image storage:

\[
100,000 \times 500\ KB
= 50,000,000\ KB
\]

Using decimal units:

\[
50,000,000\ KB = 50\ GB
\]

Monthly storage:

\[
50 \times 30 = 1,500\ GB
\]

Approximately 1.5 TB per month.

### Storage Design

A common architecture is:

```text
              Application
                   |
                   v
              API Server
                   |
                   v
             Object Storage
                   |
                   v
                  CDN
                   |
                   v
                 Users
```

Examples of object storage:
- Amazon S3.
- Azure Blob Storage.
- Google Cloud Storage.

Images and videos are commonly stored in object storage rather than directly inside relational database rows.

The database stores metadata and object references.

---

## 12. Estimating Bandwidth

Bandwidth estimation calculates the network capacity required to transfer data.

### Basic Formula

\[
Bandwidth =
Requests\ Per\ Second \times Average\ Response\ Size
\]

The result is bytes per second if response size is measured in bytes.

To convert bytes per second into bits per second:

\[
Bandwidth\ (bps) =
Bytes\ Per\ Second \times 8
\]

### Example

Suppose:
- Peak RPS = 3,000.
- Average response size = 100 KB.

Data transferred per second:

\[
3,000 \times 100\ KB
= 300,000\ KB/s
\]

Using decimal units:

\[
300\ MB/s
\]

Convert to bits:

\[
300 \times 8 = 2,400\ Mbps
\]

The response traffic is approximately 2.4 Gbps.

### Important

This is a simplified estimate. Actual network usage also depends on:
- Request payloads.
- Protocol overhead.
- Headers.
- TLS overhead.
- Retries.
- Compression.
- Caching.
- Concurrent connections.

Also, incoming and outgoing traffic should be estimated separately.

---

## 13. Estimating Compute Capacity

Compute capacity is the processing power required to handle application workload.

Important resources include:

- CPU.
- Memory.
- Network.
- Disk I/O.
- Database connections.
- Worker concurrency.

### Simple Throughput Estimation

Suppose:
- Peak workload = 3,000 requests/second.
- One server can handle 500 requests/second under the required latency and error-rate target.

Then:

\[
Required\ Servers =
\frac{3,000}{500}
\]

\[
Required\ Servers = 6
\]

Six servers are the theoretical minimum under these assumptions.

If we want 25% spare capacity:

\[
Capacity\ Target =
\frac{3,000}{0.75}
= 4,000\ RPS
\]

\[
Servers =
\frac{4,000}{500}
= 8
\]

Approximately eight servers would be needed under this simplified model.

### Important

A server's throughput must be measured using representative workload and target latency.

A server handling 500 lightweight requests per second may not handle 500 database-heavy requests per second.

Load testing is required to validate estimates.

---

## 14. Estimating Concurrency

Concurrency represents how many requests or operations are active at the same time.

A useful approximation is Little's Law:

\[
L = \lambda \times W
\]

Where:

- L = Average number of requests in the system.
- \(\lambda\) = Arrival rate (requests per second).
- W = Average time spent in the system (seconds).

### Example

Suppose:
- Request rate = 1,000 requests/second.
- Average response time = 0.2 seconds.

Then:

\[
L = 1,000 \times 0.2
\]

\[
L = 200
\]

Approximately 200 requests are in the system on average.

### Why Does This Matter?

Concurrency affects:
- Worker threads.
- Database connections.
- Memory usage.
- Queue length.
- Response latency.

Little's Law is an average relationship for a stable system. It does not directly tell us the maximum concurrency needed during bursts.

---

## 15. Estimating Database Capacity

Database capacity depends on both workload and data characteristics.

Important factors:

1. Read queries per second.
2. Write queries per second.
3. Query complexity.
4. Index usage.
5. Transaction size.
6. Connection count.
7. Data size.
8. Concurrent users.
9. Replication and consistency requirements.

### Example

Suppose an application generates:

- 2,400 read queries/second.
- 600 write queries/second.

Total query rate:

\[
2,400 + 600 = 3,000\ QPS
\]

Assume a database instance can handle 1,000 QPS under the required workload and latency.

The theoretical number of instances is:

\[
\frac{3,000}{1,000} = 3
\]

However, this does not mean we can simply deploy three independent database servers and split requests equally.

We must consider:
- Whether the database supports read replicas.
- Whether writes can be distributed.
- Transaction coordination.
- Data partitioning.
- Consistency requirements.
- Connection pooling.
- Replication lag.

A realistic database design requires workload testing.

---

## 16. Estimating Data Engineering Workloads

Capacity estimation is also important for data engineering platforms.

Suppose an e-commerce pipeline processes:

- 100 million records daily.
- Average raw record size = 1 KB.
- Processing window = 4 hours.

### Step 1: Calculate Daily Data Volume

\[
100,000,000 \times 1\ KB
= 100\ GB
\]

Using decimal units, the pipeline ingests approximately 100 GB daily.

### Step 2: Calculate Average Processing Rate

Four hours:

\[
4 \times 60 \times 60 = 14,400\ seconds
\]

Records per second:

\[
\frac{100,000,000}{14,400}
\approx 6,944
\]

The pipeline must process approximately 6,944 records per second on average during the processing window.

### Step 3: Estimate Processing Capacity

Suppose a worker can process 1,000 records per second under the required transformation workload.

\[
Workers =
\frac{6,944}{1,000}
\approx 6.94
\]

At least seven workers would be the theoretical minimum.

Actual cluster sizing must also account for:
- Data skew.
- Shuffle operations.
- Join complexity.
- Memory requirements.
- Storage I/O.
- Executor overhead.
- Failure recovery.
- Peak ingestion.
- Output file sizes.

### Data Engineering Capacity Checklist

| Component | What to Estimate |
|---|---|
| Ingestion | Records and bytes per second |
| Storage | Raw, processed, and curated data |
| Processing | Records per second and job duration |
| Compute | Driver and worker resources |
| Shuffle | Data exchanged between executors |
| Output | Number and size of files |
| Streaming | Input rate, processing rate, state size |
| Retention | Number of days or years of data |
| Recovery | Backfill and reprocessing capacity |

---

## 17. Estimating Data Growth

Data growth estimation helps predict future infrastructure requirements.

### Formula

\[
Future\ Storage =
Current\ Storage \times (1+Growth\ Rate)^n
\]

Where:
- Growth Rate = Annual growth rate.
- n = Number of years.

### Example

Suppose:
- Current storage = 10 TB.
- Annual growth = 20%.
- Planning period = 3 years.

\[
Future\ Storage =
10 \times (1.20)^3
\]

\[
Future\ Storage = 17.28\ TB
\]

The estimated storage requirement after three years is 17.28 TB.

### Important

This calculation assumes a constant annual compound growth rate.

Real workloads may grow unevenly due to:
- New customers.
- New product features.
- Seasonal traffic.
- New data sources.
- Retention policy changes.

---

## 18. Estimating Cache Capacity

Caching stores frequently accessed data in faster storage to reduce database load and latency.

### Example

Suppose:
- Total active products = 10 million.
- Frequently accessed products = 1 million.
- Average cached product size = 2 KB.

Cache storage:

\[
1,000,000 \times 2\ KB
= 2\ GB
\]

The basic payload estimate is 2 GB.

Actual cache memory must also account for:
- Keys.
- Object metadata.
- Expiration information.
- Internal data structures.
- Replication.
- Memory fragmentation.

### Cache Capacity Depends On

- Number of cached entries.
- Average entry size.
- Cache hit ratio.
- TTL (Time to Live).
- Eviction policy.
- Replication factor.

### Important

Caching reduces repeated reads from the primary data source, but does not automatically solve every database bottleneck.

Cache invalidation and stale data must also be handled.

---

## 19. Headroom and Safety Margin

Headroom is the unused capacity kept available for unexpected workload increases.

A system should not normally operate continuously at 100% resource utilization.

### Example

Suppose a service needs to support 4,000 requests per second.

The design target is 70% utilization.

Required capacity:

\[
Required\ Capacity =
\frac{4,000}{0.70}
\]

\[
Required\ Capacity \approx 5,714\ RPS
\]

The system should have capacity for approximately 5,714 requests per second under the same workload assumptions.

### Why Headroom Matters

- Unexpected traffic spikes.
- Node failures.
- Deployment activity.
- Background processing.
- Traffic imbalance.
- Future growth.

A safety margin is a planning assumption, not a substitute for load testing and autoscaling.

---

## 20. Complete Capacity Estimation Example: E-Commerce Application

Suppose we are designing an e-commerce application.

### Given Requirements

| Parameter | Value |
|---|---:|
| Registered users | 10 million |
| Daily active users | 2 million |
| Requests per user per day | 50 |
| Peak factor | 3 |
| Read percentage | 80% |
| Write percentage | 20% |
| Orders per day | 500,000 |
| Average order record size | 2 KB |
| Planning period | 1 year |

### Step 1: Daily Requests

\[
2,000,000 \times 50
= 100,000,000
\]

Daily requests = 100 million.

### Step 2: Average RPS

\[
\frac{100,000,000}{86,400}
\approx 1,157
\]

Average RPS = 1,157.

### Step 3: Peak RPS

\[
1,157 \times 3
= 3,471
\]

Peak RPS = 3,471.

### Step 4: Read and Write Requests

Reads:

\[
3,471 \times 80\%
\approx 2,777
\]

Writes:

\[
3,471 \times 20\%
\approx 694
\]

### Step 5: Daily Order Storage

\[
500,000 \times 2\ KB
= 1,000,000\ KB
\]

Daily order storage = 1 GB using decimal units.

### Step 6: Annual Order Storage

\[
1\ GB \times 365 = 365\ GB
\]

Annual raw order storage = 365 GB.

### Step 7: Replication Estimate

If three full copies are maintained:

\[
365 \times 3 = 1,095\ GB
\]

Approximately 1.095 TB.

### Step 8: Server Estimate

Assume one application server handles 500 requests per second under the target workload.

Using a 75% utilization target:

\[
Required\ Capacity =
\frac{3,471}{0.75}
= 4,628\ RPS
\]

\[
Servers =
\frac{4,628}{500}
\approx 9.26
\]

Approximately 10 servers are required under these simplified assumptions.

### Final Estimated Requirements

| Component | Estimate |
|---|---:|
| Daily active users | 2 million |
| Daily requests | 100 million |
| Average RPS | 1,157 |
| Peak RPS | 3,471 |
| Peak read RPS | 2,777 |
| Peak write RPS | 694 |
| Daily order storage | 1 GB |
| Annual raw order storage | 365 GB |
| Three-copy storage | 1.095 TB |
| Application servers | 10 |

These are initial estimates, not production sizing decisions. Actual sizing must be validated with realistic load tests and workload measurements.

---

## 21. Common Mistakes in Capacity Estimation

### Mistake 1: Using Registered Users as Daily Active Users

Not every registered user accesses an application every day.

**Correct approach:** Estimate DAU separately.

### Mistake 2: Using Average RPS as Peak RPS

Traffic is not evenly distributed throughout the day.

**Correct approach:** Estimate peak traffic using historical data or a justified peak factor.

### Mistake 3: Ignoring Read and Write Ratios

Read and write workloads place different demands on databases.

**Correct approach:** Estimate read and write traffic separately.

### Mistake 4: Ignoring Metadata and Indexes

Raw record size does not represent total database storage.

**Correct approach:** Account for indexes, logs, metadata, and storage overhead.

### Mistake 5: Assuming Every Server Has the Same Throughput

Different workloads consume different resources.

**Correct approach:** Benchmark representative workloads.

### Mistake 6: Ignoring Replication and Backups

Data protection increases storage and infrastructure requirements.

**Correct approach:** Estimate replication, backup, and retention separately.

### Mistake 7: Ignoring Data Growth

A system designed only for today's traffic may become insufficient.

**Correct approach:** Include future growth assumptions.

### Mistake 8: Confusing Bandwidth with Storage

Storage measures retained data. Bandwidth measures data transferred per unit of time.

**Correct approach:** Calculate them independently.

### Mistake 9: Treating Estimates as Exact Numbers

Early system design rarely has complete workload measurements.

**Correct approach:** State assumptions, use ranges where appropriate, and validate with testing.

---

## 22. Interview Questions and Answers

### Q1. What is capacity estimation?

Capacity estimation calculates the compute, storage, network, and processing resources required to handle an expected workload.

### Q2. Why is capacity estimation important?

It prevents under-provisioning, reduces unnecessary infrastructure costs, and helps maintain system performance.

### Q3. What is DAU?

Daily Active Users are unique users who interact with an application during a day.

### Q4. What is RPS?

Requests Per Second measures how many requests an application receives or processes per second.

### Q5. How do you calculate average RPS?

\[
Average\ RPS =
\frac{Daily\ Requests}{86,400}
\]

### Q6. What is peak RPS?

Peak RPS is the highest expected request rate during a particular period.

### Q7. What is the difference between QPS and RPS?

RPS counts application requests. QPS counts queries executed by a database or query system.

### Q8. How do you estimate storage?

Multiply the number of records by their average stored size, then account for retention, indexes, metadata, replication, and other overhead.

### Q9. How do you estimate bandwidth?

Multiply the data transferred per second by eight to convert bytes per second into bits per second.

### Q10. What is concurrency?

Concurrency represents the number of operations active at the same time.

### Q11. What is Little's Law?

\[
L = \lambda W
\]

It relates average items in a system to arrival rate and average time spent in the system.

### Q12. What is headroom?

Headroom is unused capacity reserved for traffic spikes, failures, and future growth.

### Q13. How do you estimate the number of application servers?

Divide the required throughput by the measured throughput per server, adjusting for the desired utilization and redundancy.

### Q14. Why is read/write ratio important?

Read and write workloads have different database and infrastructure requirements.

### Q15. How do you estimate data engineering processing capacity?

Estimate total records or bytes, processing window, processing rate, and workload complexity. Then validate the required compute resources through benchmarks.

### Q16. What is the difference between throughput and latency?

Throughput measures completed work per unit of time. Latency measures the time taken by an individual operation.

### Q17. What is the difference between horizontal and vertical scaling?

Horizontal scaling adds more machines or instances. Vertical scaling increases the resources of an existing machine.

### Q18. Is capacity estimation enough to finalize production infrastructure?

No. It provides an initial design. Load testing, monitoring, and production workload measurements are required to validate it.

---

## 23. Quick Revision

| Formula | Purpose |
|---|---|
| `DAU = Registered Users × Active Percentage` | Estimate active users |
| `Daily Requests = DAU × Requests per User` | Estimate daily workload |
| `Average RPS = Daily Requests / 86,400` | Estimate average traffic |
| `Peak RPS = Average RPS × Peak Factor` | Estimate peak traffic |
| `Daily Storage = Records × Record Size` | Estimate daily data |
| `Total Storage = Daily Storage × Retention Days` | Estimate retained data |
| `Replicated Storage = Raw Storage × Replication Factor` | Estimate full-copy storage |
| `Bandwidth (bps) = Bytes/sec × 8` | Estimate network bandwidth |
| `L = λ × W` | Estimate average concurrency |
| `Future Storage = Current × (1 + Growth Rate)^n` | Estimate future storage |
| `Required Capacity = Workload / Target Utilization` | Include headroom |
| `Servers = Required Capacity / Per-Server Capacity` | Estimate server count |

### Remember These Points

1. Start with users and workload assumptions.
2. Calculate average and peak traffic separately.
3. Separate reads from writes.
4. Estimate storage, compute, and bandwidth independently.
5. Include replication, retention, and growth.
6. Use Little's Law for average concurrency.
7. Validate assumptions through load testing.
8. Monitor actual production behavior and adjust capacity.

---

## 24. Self-Assessment

Try solving these without looking at the answers.

1. What is capacity estimation?
2. Why is it required in system design?
3. What is the difference between DAU and MAU?
4. How do you calculate average RPS?
5. What is the difference between average RPS and peak RPS?
6. How do you calculate read and write traffic?
7. How do you estimate daily storage?
8. How does replication affect storage?
9. How do you estimate bandwidth?
10. What is Little's Law?
11. How do you estimate the number of application servers?
12. What is headroom?
13. What is the difference between throughput and latency?
14. How do you estimate data engineering processing capacity?
15. Why should capacity estimates be validated through load testing?

**Interview readiness:** Be able to estimate daily traffic, peak RPS, read/write workload, storage growth, bandwidth, and server count using simple assumptions. Explain every assumption clearly rather than presenting calculations as exact production requirements.

