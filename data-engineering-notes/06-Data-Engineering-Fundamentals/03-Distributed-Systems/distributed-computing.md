# Distributed Computing

## 1. Introduction

**Distributed computing is a way of solving problems by using multiple computers that communicate and coordinate with one another over a network.**

Instead of relying on one computer to perform all the work, a distributed system divides work across multiple machines.

Distributed computing is widely used in:
- Apache Spark
- Hadoop
- Apache Kafka
- Cloud computing
- Distributed databases
- Large-scale data pipelines
- Real-time data processing

### Simple Example

Suppose a company has 100 GB of sales data to process.

On a single machine, one computer may need to process the entire dataset.

With distributed computing, the workload can be divided among several machines.

```text
             100 GB Dataset
                    |
                    v
              Work Division
                    |
          +---------+---------+
          |         |         |
          v         v         v
       Machine 1 Machine 2 Machine 3
        35 GB      35 GB      30 GB
          |         |         |
          v         v         v
       Process    Process    Process
          |         |         |
          +---------+---------+
                    |
                    v
              Final Results
```

The distribution shown is illustrative. Actual data placement and workload distribution depend on the processing framework and data characteristics.

**Key idea:** Multiple machines work together to process a large workload.

---

## 2. Why Do We Need Distributed Computing?

A single computer has limited CPU, memory, storage, and processing capacity.

As data grows, one machine may become too slow or run out of resources.

Distributed computing helps address these limitations.

### Main Benefits

**1. Scalability**

More machines can be added to increase the system's available resources.

**2. Parallel processing**

Different parts of a workload can be processed simultaneously.

**3. Large-scale data processing**

Datasets that do not fit comfortably on one machine can be processed across multiple machines.

**4. Fault tolerance**

Some systems can recover from machine or task failures by retrying work or using replicated data.

**5. Resource sharing**

Multiple machines contribute CPU, memory, and other resources to a workload.

### Example

An e-commerce company processes billions of transaction records.

A single machine may take too long to calculate monthly revenue.

A distributed processing engine can process different partitions of the data concurrently and combine the partial results.

---

## 3. What Is a Distributed System?

A **distributed system** is a collection of independent computers that communicate over a network and coordinate to provide a service or complete work.

Examples include:
- A Spark cluster.
- A Kafka cluster.
- A distributed database.
- A cloud-based data platform.

### Main Components

A typical distributed data-processing system may include:

| Component | Responsibility |
|---|---|
| Client | Submits a request or job |
| Coordinator | Organizes work and tracks execution |
| Worker nodes | Execute assigned tasks |
| Network | Transfers data and messages |
| Storage | Holds input data, output data, or recovery information |

The exact architecture varies by system. For example, Spark uses a driver and executors, while other frameworks use different coordination models.

### Important Characteristics

- Multiple independent machines.
- Communication over a network.
- Parallel or coordinated execution.
- Potential machine and network failures.
- Need for coordination and recovery.

A distributed system is not simply multiple computers running independently. The computers must coordinate to complete a shared task or provide a common service.

---

## 4. Parallel Computing vs Distributed Computing

These concepts are related but not identical.

### Parallel Computing

Parallel computing means executing multiple parts of a workload simultaneously.

It can happen:
- On multiple CPU cores in one machine.
- Across multiple processors.
- Across multiple machines.

### Distributed Computing

Distributed computing uses multiple networked computers that communicate and coordinate.

It commonly supports parallel execution, but distributed systems may also perform tasks sequentially or coordinate services that do not run in parallel.

### Comparison

| Feature | Parallel Computing | Distributed Computing |
|---|---|---|
| Main idea | Execute work simultaneously | Coordinate work across computers |
| Hardware | One or more machines | Multiple networked computers |
| Communication | Shared memory or communication mechanisms | Usually network communication |
| Failure handling | Depends on the architecture | Must account for independent machine and network failures |
| Example | Multiple CPU cores processing an array | Spark executors processing a dataset across a cluster |

**Interview point:** Parallelism describes how work executes concurrently; distribution describes how computing resources are spread across machines.

---

## 5. Centralized vs Distributed Systems

### Centralized System

A centralized system relies primarily on one central computer or service to perform its main work.

```text
       Client Requests
              |
              v
       Central Computer
              |
              v
           Results
```

Advantages:
- Easier to understand and manage in simple environments.
- Less coordination overhead.
- Data may be easier to manage in one place.

Limitations:
- Limited capacity compared with a scalable cluster.
- The central component can become a bottleneck.
- A failure of the central component may interrupt the service unless redundancy exists.

### Distributed System

```text
        Client Requests
               |
               v
          Coordinator
               |
        +------+------+ 
        |      |      |
        v      v      v
      Node 1 Node 2 Node 3
        |      |      |
        +------+------+ 
               |
               v
            Results
```

Advantages:
- Can scale by adding resources.
- Supports parallel workloads.
- Can provide fault tolerance through suitable architecture.

Limitations:
- More complex coordination.
- Network delays and failures.
- Data consistency challenges.
- Additional monitoring and operational overhead.

Distributed systems do not automatically eliminate single points of failure. Coordination services, storage systems, and network components must also be designed for reliability.

---

## 6. How Distributed Data Processing Works

A typical distributed processing workflow has several stages.

### Step 1: Submit the Job

A user submits a data-processing job.

Example: Calculate total sales by region.

### Step 2: Divide the Work

The framework identifies units of work, often based on input partitions and execution requirements.

### Step 3: Assign Tasks

The framework schedules tasks on available worker resources.

### Step 4: Process Data

Workers execute their assigned tasks in parallel where possible.

### Step 5: Exchange Data if Required

Some operations require data to move between workers. Examples include joins, grouping, and sorting.

### Step 6: Combine Results

Partial results are combined or written to the destination.

```text
Input Data
    |
    v
Divide Work
    |
    v
Schedule Tasks
    |
    v
Execute on Workers
    |
    v
Exchange Data if Needed
    |
    v
Combine or Write Results
```

Not every operation requires a network shuffle or a final result-combination stage. The execution plan determines which stages are needed.

---

## 7. Distributed Computing in Apache Spark

Apache Spark is a distributed data-processing engine designed to process data across a cluster.

It supports:
- Batch processing.
- Structured Streaming.
- SQL processing.
- DataFrame operations.
- Machine learning workloads through supported libraries.

### Main Components

**Driver**

The driver coordinates the Spark application. It creates the execution plan, schedules tasks, and tracks execution.

**Cluster Manager**

The cluster manager allocates resources to applications. Depending on the deployment, this may be Spark's standalone cluster manager, YARN, Kubernetes, or a managed platform's resource system.

**Executors**

Executors run tasks and may cache data or maintain state for the application.

**Tasks**

Tasks are units of work that process data partitions.

**Stages**

Spark divides a job into stages based on execution dependencies, particularly shuffle boundaries.

### Simplified Architecture

```text
              Spark Application
                      |
                      v
                   Driver
                      |
                      v
              Cluster Manager
                      |
            +---------+---------+
            |         |         |
            v         v         v
         Executor  Executor  Executor
            |         |         |
          Tasks     Tasks     Tasks
            |         |         |
            +---------+---------+
                      |
                      v
                 Output Data
```

This is a simplified conceptual diagram. Actual scheduling and resource allocation depend on the deployment environment.

---

## 8. Spark Example: Distributed Data Processing

Suppose a company has an orders dataset with these columns:

| order_id | region | amount |
|---:|---|---:|
| 101 | East | 500 |
| 102 | West | 700 |
| 103 | East | 300 |
| 104 | West | 200 |

The requirement is to calculate total revenue by region.

### PySpark Code

```python
from pyspark.sql import functions as F

orders = spark.read.parquet("/data/orders/")

regional_sales = (
    orders
    .groupBy("region")
    .agg(
        F.sum("amount").alias("total_revenue")
    )
)

regional_sales.show()
```

Expected result for the sample data:

| region | total_revenue |
|---|---:|
| East | 800 |
| West | 900 |

### What Happens Internally?

1. Spark reads the input data.
2. The optimizer plans the query.
3. Input partitions can be processed by different tasks.
4. `groupBy("region")` generally requires data to be grouped by region across the relevant partitions.
5. Spark may shuffle data so that records for the same region reach the appropriate processing partition.
6. The aggregation produces the final regional totals.

The actual execution plan depends on the data source, optimizer, configuration, and available statistics.

---

## 9. Data Partitioning and Distributed Processing

Distributed processing commonly divides data into partitions so that multiple tasks can process different portions of the dataset.

For example:

```text
Dataset
   |
   +--> Partition 1 --> Task 1
   |
   +--> Partition 2 --> Task 2
   |
   +--> Partition 3 --> Task 3
   |
   +--> Partition 4 --> Task 4
```

Each task processes its assigned partition.

### Why Partitions Matter

- They provide units of parallel work.
- They influence task count.
- They affect workload distribution.
- They influence shuffle behavior and execution performance.

### Too Few Partitions

Possible problems:
- Not enough parallelism.
- Some resources remain underused.
- Individual tasks may process too much data.

### Too Many Partitions

Possible problems:
- Excessive task-scheduling overhead.
- Many small tasks.
- Increased overhead from small shuffle blocks or small files, depending on the operation.

### Important Distinction

A **Spark execution partition** is not the same as a **storage partition**.

- Execution partitioning distributes work among tasks.
- Storage partitioning organizes files by selected column values.

The number of execution partitions does not automatically equal the number of storage directories or output files.

---

## 10. Communication and Network Shuffles

Distributed computers must communicate to exchange data and coordinate work.

In Spark, a **shuffle** redistributes data across partitions, often across executors or machines.

### Example

Suppose the input is distributed across three workers:

```text
Worker 1: East, West
Worker 2: East, North
Worker 3: West, North
```

To aggregate by region, Spark must bring records with the same region together for the relevant aggregation stage.

Conceptually:

```text
Records on Multiple Workers
             |
             v
       Shuffle by Region
             |
     +-------+-------+
     |       |       |
     v       v       v
   East    North    West
  Records  Records  Records
     |       |       |
     v       v       v
  Aggregate Aggregate Aggregate
```

This is a simplified illustration; the actual physical plan may use partial aggregation and other optimizations.

### Common Shuffle Operations

- `groupBy()`
- `distinct()`
- Many joins
- `orderBy()`
- `repartition()`

Not every use of these operations results in identical physical behavior. The optimizer may change the plan, and some operations can use partial aggregation or avoid a full shuffle in specific circumstances.

### Why Shuffles Can Be Expensive

- Network data transfer.
- Serialization and deserialization.
- Disk I/O.
- Memory pressure.
- Possible data skew.

**Optimization tip:** Reduce unnecessary shuffles, filter data early when appropriate, select only required columns, and choose join strategies based on data size and distribution.

---

## 11. Fault Tolerance in Distributed Systems

A distributed system must account for failures because individual machines, tasks, and network connections can fail independently.

### Common Failure Types

**1. Worker failure**

A machine or executor becomes unavailable.

**2. Task failure**

A task fails while processing its assigned data.

**3. Network failure**

Machines cannot communicate reliably.

**4. Driver or coordinator failure**

The component coordinating the application fails.

**5. Storage failure**

Input or output data becomes unavailable or a write fails.

### Common Recovery Techniques

- Retry failed tasks.
- Replicate important data.
- Persist source data.
- Use checkpoints for supported recovery workflows.
- Use transactional writes.
- Monitor and restart failed applications where appropriate.

### Spark Fault Tolerance

Spark can retry failed tasks and recompute lost partitions using lineage when the required inputs and dependencies remain available.

For Structured Streaming, checkpoints store progress and state needed for supported recovery.

However, fault tolerance depends on the failure type and configuration. A failed task retry does not automatically make arbitrary external side effects safe.

**Interview point:** Fault tolerance means the system can handle specified failures and recover according to its design; it does not mean failures never happen.

---

## 12. Scalability: Vertical vs Horizontal

Scalability describes the ability to handle increased workloads.

### Vertical Scaling

Vertical scaling means increasing the resources of a single machine.

Examples:
- More CPU cores.
- More memory.
- Faster storage.

Advantages:
- Often simpler to manage.
- May avoid some distributed coordination overhead.

Limitations:
- Hardware has practical limits.
- Larger machines can be expensive.
- A single machine may remain a bottleneck.

### Horizontal Scaling

Horizontal scaling means adding more machines or workers.

Example:

```text
Before:
Machine 1

After:
Machine 1 + Machine 2 + Machine 3 + Machine 4
```

Advantages:
- Can increase aggregate processing capacity.
- Supports distributed execution.
- Can provide more flexibility in resource allocation.

Limitations:
- Requires communication and coordination.
- Not every workload scales linearly.
- Network and storage bottlenecks can limit performance.
- More machines increase operational complexity.

### Comparison

| Feature | Vertical Scaling | Horizontal Scaling |
|---|---|---|
| Method | Increase resources on one machine | Add more machines |
| Complexity | Often lower | Often higher |
| Capacity | Limited by machine size | Can grow with additional machines |
| Communication overhead | Usually lower | Often higher |
| Example | Upgrade server memory | Add Spark worker nodes |

**Important:** Adding twice as many workers does not guarantee twice the performance.

---

## 13. Distributed Computing Challenges

Distributed computing introduces challenges that are less significant in a simple single-machine program.

### 13.1 Network Latency

Communication between machines takes time.

A workload with frequent communication may not scale well even if more workers are added.

### 13.2 Data Skew

Some partitions contain much more data than others.

The workers processing those partitions may take longer than the rest.

### 13.3 Synchronization

Workers sometimes need to coordinate before continuing.

This can create waiting time and reduce parallel efficiency.

### 13.4 Partial Failures

One machine may fail while the rest of the system continues running.

The system must detect and handle the failure appropriately.

### 13.5 Data Consistency

When data is copied or updated across multiple systems, the system must define how readers see changes and how conflicting updates are handled.

### 13.6 Resource Management

The system must manage CPU, memory, storage, network bandwidth, and concurrent workloads.

### 13.7 Duplicate Processing

Retries can cause work to run more than once. Idempotent operations or transactional mechanisms may be needed to prevent duplicate effects.

---

## 14. Distributed Computing vs Distributed Storage

These are related but different concepts.

### Distributed Computing

Uses multiple machines to execute a workload.

Examples:
- Spark processing a dataset.
- Flink processing events.
- Distributed model training.

### Distributed Storage

Stores data across multiple machines or storage nodes.

Examples:
- HDFS.
- Distributed object storage services.
- Distributed databases.

### Comparison

| Feature | Distributed Computing | Distributed Storage |
|---|---|---|
| Main purpose | Process data | Store and retrieve data |
| Main resources | CPU and memory across workers | Storage capacity and data access |
| Example | Apache Spark | HDFS or cloud object storage |
| Typical concern | Scheduling and execution | Durability, availability, and access |

A data platform often uses both.

For example, Spark can read data from cloud object storage, process it across executors, and write the results back to storage.

---

## 15. Distributed Computing vs Cloud Computing

**Cloud computing** provides computing, storage, networking, and managed services through a cloud provider.

**Distributed computing** is a computing approach in which multiple networked machines coordinate to perform work.

Cloud platforms commonly provide the infrastructure used for distributed computing, but the concepts are not identical.

| Feature | Distributed Computing | Cloud Computing |
|---|---|---|
| Meaning | Coordinated computation across machines | On-demand access to computing and related services |
| Scope | Computing architecture and execution | Infrastructure and service delivery model |
| Can exist on-premises | Yes | Not necessarily; cloud services are provider-delivered |
| Example | Spark cluster | Managed cloud Spark service |

A managed Databricks workspace on a cloud platform is an example of a cloud service that can run distributed data-processing workloads.

---

## 16. Practical Data Engineering Scenario

Suppose a company processes 2 TB of sales data each day.

A single machine may not provide sufficient processing capacity within the required time.

### Possible Architecture

```text
Source Systems
      |
      v
Cloud Object Storage
      |
      v
Spark Driver
      |
      v
Distributed Spark Executors
      |
      v
Data Transformation
      |
      v
Shuffle / Aggregation
      |
      v
Delta Tables
      |
      v
BI and Analytics
```

### Design Considerations

1. Store source data durably.
2. Choose appropriate input file sizes and data layout.
3. Allocate sufficient worker resources.
4. Use partitions to provide suitable parallelism.
5. Avoid unnecessary shuffles.
6. Monitor data skew and slow tasks.
7. Use checkpointing for supported streaming recovery.
8. Make destination writes safe to retry.
9. Monitor data quality, freshness, and job failures.
10. Optimize resource usage based on observed performance.

The exact design depends on data volume, processing latency, query patterns, reliability requirements, and budget.

---

## 17. Common Interview Questions and Answers

### Q1. What is distributed computing?

Distributed computing uses multiple networked computers to coordinate and process workloads. It enables parallel execution and can scale processing capacity beyond a single machine.

### Q2. Why is distributed computing important in data engineering?

It allows data engineers to process large datasets using the combined resources of multiple machines. It is used in systems such as Spark, Hadoop, and Flink.

### Q3. What is a distributed system?

A distributed system consists of independent computers that communicate and coordinate to provide a service or complete a shared workload.

### Q4. What is the difference between parallel and distributed computing?

Parallel computing focuses on executing work simultaneously. Distributed computing focuses on coordinating work across multiple networked machines. Distributed computing often uses parallelism.

### Q5. What is a worker node?

A worker node provides resources to execute tasks. In Spark, executors running on worker resources execute tasks assigned by the driver.

### Q6. What is the role of the Spark driver?

The driver coordinates the Spark application, plans execution, schedules tasks, and tracks their progress.

### Q7. What is a Spark executor?

An executor runs tasks and can cache data or maintain state for a Spark application.

### Q8. What is a shuffle?

A shuffle redistributes data across partitions, often across machines. It is commonly required for grouping, many joins, and sorting.

### Q9. Why are shuffles expensive?

They can involve network transfer, serialization, disk I/O, and memory pressure.

### Q10. What is fault tolerance?

Fault tolerance is the ability to handle specified failures and recover without losing required work or corrupting results, according to the system's guarantees.

### Q11. How does Spark handle task failures?

Spark can retry failed tasks and recompute lost partitions from lineage when the required dependencies and inputs are available.

### Q12. What is horizontal scaling?

Horizontal scaling adds more machines or workers to increase available processing resources.

### Q13. Does adding more Spark workers always improve performance?

No. Performance also depends on data volume, partitioning, skew, shuffles, resource utilization, and workload parallelism.

### Q14. What is data skew?

Data skew occurs when data is distributed unevenly across partitions, causing some tasks to process much more data than others.

### Q15. What is the difference between distributed storage and distributed computing?

Distributed storage spreads data storage across systems. Distributed computing uses multiple machines to execute processing tasks.

### Q16. What is the relationship between cloud computing and distributed computing?

Cloud platforms provide infrastructure and managed services that can run distributed systems. Distributed computing is the execution approach, while cloud computing is a way of providing computing resources and services.

### Q17. How does Spark process a large dataset?

Spark reads data, builds an execution plan, divides work into tasks, executes tasks across partitions, performs required shuffles, and writes or returns results.

### Q18. How would you improve a slow distributed data pipeline?

I would inspect execution plans and runtime metrics, identify slow stages, check for skew and excessive shuffles, optimize data layout and file sizes, and adjust partitioning or resource allocation based on evidence.

---

## 18. Common Interview Mistakes

| Mistake | Correct Understanding |
|---|---|
| Distributed computing always means faster processing | Communication and coordination can add overhead |
| Adding workers doubles performance | Scaling depends on workload and bottlenecks |
| Every Spark operation requires a shuffle | Many operations can execute without a shuffle |
| Fault tolerance means no failures happen | Failures occur; the system detects and recovers from supported failures |
| More partitions are always better | Too many partitions create overhead |
| Spark driver processes all data itself | Executors generally process distributed partitions |
| Distributed storage and distributed computing are the same | One stores data; the other processes it |
| Cloud computing and distributed computing are identical | Cloud computing provides services; distributed computing is an architecture and execution approach |
| Retries always prevent data loss and duplicates | Retry safety depends on source, progress tracking, and destination behavior |

---

## 19. Quick Revision

| Concept | Remember |
|---|---|
| Distributed computing | Coordinated computation across multiple machines |
| Distributed system | Independent computers working together |
| Parallel computing | Simultaneous execution of work |
| Worker | Provides resources to execute tasks |
| Spark driver | Coordinates the application |
| Spark executor | Executes tasks |
| Task | Unit of work on a data partition |
| Shuffle | Redistributes data across partitions |
| Fault tolerance | Recovery from supported failures |
| Vertical scaling | Increase resources on one machine |
| Horizontal scaling | Add more machines |
| Data skew | Uneven distribution of data |
| Distributed storage | Stores data across systems |
| Cloud computing | Provides computing and related services |

### One-Minute Interview Answer

Distributed computing is a technique in which multiple networked computers work together to process a workload. It helps data engineering systems handle large datasets by distributing work across machines.

Apache Spark is a common example. The driver coordinates the application, executors run tasks, and data partitions provide units of parallel work. Operations such as grouping and joins may require a shuffle, which redistributes data across partitions.

Distributed systems must also handle machine failures, network delays, data skew, and resource limitations. Spark can retry failed tasks and recompute lost partitions when the required dependencies are available. Good performance depends on appropriate partitioning, efficient data layout, minimizing unnecessary shuffles, and monitoring execution.

## 20. Self-Assessment

- [ ] Define distributed computing.
- [ ] Explain why it is useful in data engineering.
- [ ] Differentiate parallel and distributed computing.
- [ ] Explain centralized vs distributed systems.
- [ ] Explain the Spark driver and executors.
- [ ] Understand tasks, stages, and partitions.
- [ ] Explain why shuffles occur.
- [ ] Describe fault tolerance and task retries.
- [ ] Differentiate vertical and horizontal scaling.
- [ ] Explain data skew and its impact.
- [ ] Differentiate distributed storage and distributed computing.
- [ ] Explain cloud computing vs distributed computing.
- [ ] Describe how Spark processes a large dataset.
- [ ] Identify ways to optimize a slow distributed pipeline.

**Interview readiness:** Be able to explain how Spark distributes a large dataset across executors, why shuffles occur, and how partitioning, fault tolerance, and scaling affect pipeline performance.