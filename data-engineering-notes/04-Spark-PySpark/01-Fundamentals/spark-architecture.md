# Apache Spark Architecture

## 1. What Is Apache Spark Architecture?

Apache Spark uses a distributed computing architecture to process large datasets across multiple machines.

Instead of processing an entire dataset on one machine, Spark distributes the workload across multiple worker nodes.

The main components are:

1. Driver
2. Executors
3. Cluster Manager
4. Worker Nodes
5. Spark Application

**In simple terms:**

- Driver coordinates the application.
- Cluster Manager allocates resources.
- Executors perform data-processing tasks.
- Worker Nodes provide the machines where executors run.

---

## 2. High-Level Spark Architecture

```text
                 Spark Application
                        |
                        v
                 +-------------+
                 |   Driver    |
                 |             |
                 | SparkSession|
                 | SparkContext|
                 | DAG Scheduler
                 | Task Scheduler
                 +-------------+
                        |
                        |
                 Cluster Manager
                        |
             Allocates Resources
                        |
          +-------------+-------------+
          |                           |
          v                           v
   +--------------+            +--------------+
   | Worker Node 1|            | Worker Node 2|
   |              |            |              |
   |  Executor 1  |            |  Executor 2  |
   |  Tasks       |            |  Tasks       |
   |  Cache       |            |  Cache       |
   +--------------+            +--------------+
          |                           |
          v                           v
      Process Data                Process Data
```

This is a simplified representation. A Spark application can run with multiple executors, and their placement depends on the cluster manager and deployment configuration.

---

## 3. Driver

The Driver is the central coordinator of a Spark application.

It manages the execution of the application and coordinates work across executors.

### Main Responsibilities

- Creates the Spark application.
- Maintains the application execution context.
- Builds execution plans.
- Converts jobs into stages and tasks.
- Schedules tasks on executors.
- Tracks task execution and results.
- Coordinates with the cluster manager for resources.

### Important Components Inside the Driver

**1. SparkSession**

The main entry point for working with DataFrames and Spark SQL.

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("DataEngineeringApp")
    .getOrCreate()
)
```

**2. SparkContext**

Connects the Spark application to the cluster and coordinates distributed execution.

In modern PySpark applications, SparkSession is the preferred entry point. Its underlying SparkContext can be accessed using:

```python
sc = spark.sparkContext
```

**3. DAG Scheduler**

- Builds a Directed Acyclic Graph (DAG) of operations.
- Divides jobs into stages.
- Identifies shuffle dependencies.

**4. Task Scheduler**

- Schedules individual tasks.
- Sends tasks to available executors.
- Coordinates task execution.

### Important

The Driver does not normally process the entire distributed dataset itself. Executors perform the distributed data-processing work.

However, operations such as `collect()` can bring data back to the Driver.

---

## 4. Executors

Executors are processes launched for a Spark application on worker nodes.

They perform the actual distributed data processing.

### Main Responsibilities

- Execute tasks assigned by the Driver.
- Process partitions of data.
- Store intermediate data in memory or on disk.
- Cache and persist data.
- Send task results and status information to the Driver.

### Example

Suppose a dataset contains 1 million records distributed across four partitions.

```text
Partition 1 -> 250,000 records
Partition 2 -> 250,000 records
Partition 3 -> 250,000 records
Partition 4 -> 250,000 records
```

Spark may distribute these partitions across executors:

```text
Executor 1:
    Task 1 -> Partition 1
    Task 2 -> Partition 2

Executor 2:
    Task 3 -> Partition 3
    Task 4 -> Partition 4
```

This is an illustrative distribution. Actual task scheduling depends on available cores, resources, partition count, and execution requirements.

### Executor Components

**1. Task Execution**

Executors run tasks assigned by the Driver.

**2. Memory Management**

Executors use memory for execution, caching, and intermediate processing.

**3. Storage**

Executors can store cached or persisted partitions.

**4. Shuffle Processing**

Executors write and read intermediate shuffle data during wide transformations.

### Important

Executors generally remain active for the lifetime of the application, unless dynamic allocation or other resource-management behavior changes their number.

---

## 5. Cluster Manager

The Cluster Manager is responsible for allocating resources to Spark applications.

It helps Spark obtain CPU and memory resources from the cluster.

### Common Cluster Managers

| Cluster Manager | Description |
|---|---|
| Standalone | Spark's built-in cluster manager |
| YARN | Resource management framework in Hadoop |
| Kubernetes | Container orchestration platform |
| Mesos | Older cluster management option; no longer a typical choice for new deployments |

### Responsibilities

- Allocate CPU and memory resources.
- Launch executors.
- Manage resource availability.
- Coordinate resource allocation across applications.

### Important Difference

The Cluster Manager allocates resources, while the Driver schedules tasks on the executors.

Do not confuse resource allocation with task scheduling.

---

## 6. Worker Nodes

Worker Nodes are machines in the cluster that provide resources for executing Spark applications.

They typically provide:

- CPU
- Memory
- Storage
- Network resources

Executors run on worker nodes.

Example:

```text
Cluster
  |
  +---- Worker Node 1
  |          |
  |          +---- Executor 1
  |
  +---- Worker Node 2
             |
             +---- Executor 2
```

A worker node can host multiple executors, depending on the cluster configuration and available resources.

---

## 7. How Spark Architecture Works Step by Step

Consider a PySpark application:

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("OrdersPipeline")
    .getOrCreate()
)

df = spark.read.csv(
    "orders.csv",
    header=True,
    inferSchema=True
)

filtered_df = df.filter(df.amount > 1000)

result = filtered_df.groupBy("category").sum("amount")

result.show()
```

Let's understand what happens internally.

### Step 1: Application Starts

The Spark application starts, and the Driver process is initialized.

SparkSession and SparkContext are created or obtained.

### Step 2: Resource Allocation

The Driver communicates with the Cluster Manager.

The Cluster Manager allocates resources and launches executors on worker nodes.

### Step 3: Transformations Are Defined

The application defines operations such as:

- Read data.
- Filter records.
- Group records.
- Calculate aggregations.

Spark generally records these transformations lazily instead of executing them immediately.

### Step 4: Action Is Called

```python
result.show()
```

The action triggers the computation required to produce the result.

### Step 5: Execution Plan Is Created

For DataFrame operations, Spark analyzes and optimizes the query plan.

The execution is organized into jobs, stages, and tasks.

### Step 6: Tasks Are Scheduled

The DAG Scheduler divides the job into stages based on dependencies.

The Task Scheduler sends tasks to available executors.

### Step 7: Executors Process Data

Executors process the required data partitions.

During `groupBy()`, Spark commonly performs partial aggregation and shuffles data by grouping key before final aggregation.

### Step 8: Results Are Returned

The required result is brought back to the Driver for display by `show()`.

---

## 8. Job, Stage, and Task Relationship

These are important concepts in Spark architecture.

### Job

A Job is created when an action triggers computation.

Examples:

```python
df.show()
df.count()
df.collect()
```

### Stage

A Stage is a group of tasks that can execute without crossing a shuffle dependency.

Wide transformations commonly introduce shuffle boundaries between stages.

### Task

A Task is the smallest unit of execution scheduled by Spark.

A task generally processes one partition for a particular stage.

### Relationship

```text
Spark Application
       |
       v
      Job
       |
       v
     Stages
       |
       v
     Tasks
       |
       v
   Executors
```

Example:

```text
Job 1
 |
 +---- Stage 1
 |       |
 |       +---- Task 1
 |       +---- Task 2
 |       +---- Task 3
 |
 +---- Shuffle
         |
         +---- Stage 2
                 |
                 +---- Task 1
                 +---- Task 2
```

**Remember:**

- One application can contain multiple jobs.
- One job can contain multiple stages.
- One stage can contain multiple tasks.
- Executors execute tasks.

---

## 9. Spark Deployment Modes

Deployment mode determines where the Driver process runs.

The two commonly discussed modes are:

### 9.1 Client Mode

In client mode, the Driver runs on the machine from which the application is submitted.

```text
Submission Machine
       |
       +---- Driver
                 |
                 v
          Cluster Manager
                 |
          +------+------+
          |             |
          v             v
      Executor 1    Executor 2
```

**Characteristics:**

- Driver runs outside the cluster's worker nodes.
- Useful for interactive development and debugging.
- The submission machine must remain available.
- Network connectivity between the Driver and cluster is important.

### 9.2 Cluster Mode

In cluster mode, the Driver runs inside the cluster, typically on a worker node.

```text
Submission Machine
       |
       v
Cluster Manager
       |
       v
Worker Node 1
       |
       +---- Driver
       |
       +---- Executor 1
       |
       v
Worker Node 2
       |
       +---- Executor 2
```

**Characteristics:**

- Driver runs within the cluster.
- Suitable for production batch applications.
- The application does not depend on the submission machine remaining connected after submission.
- The Driver is managed as part of the cluster deployment.

**Interview point:** Client mode and cluster mode describe Driver placement, not where the executors run. Executors run on cluster resources in both modes.

---

## 10. Spark Architecture and Data Engineering

Consider a real-world Data Engineering pipeline:

```text
Source Systems
     |
     v
Data Ingestion
     |
     v
ADLS / S3 / HDFS
     |
     v
Spark Driver
     |
     v
Execution Planning
     |
     v
Cluster Manager
     |
     v
Executors
     |
     v
Data Transformation
     |
     v
Delta Lake / Data Warehouse
```

Example transformations:

- Read raw files.
- Remove invalid records.
- Handle null values.
- Remove duplicates.
- Join datasets.
- Perform aggregations.
- Write transformed data.

In a production pipeline, Spark distributes these operations across executors to process large datasets efficiently.

---

## 11. Common Interview Questions

### Q1. What are the main components of Spark architecture?

The main components are Driver, Executors, Cluster Manager, and Worker Nodes.

### Q2. What is the role of the Driver?

The Driver coordinates the Spark application, builds execution plans, creates jobs, divides them into stages and tasks, and schedules tasks on executors.

### Q3. What is the role of an Executor?

An Executor runs tasks, processes data partitions, manages cached data, and handles intermediate data such as shuffle output.

### Q4. What is the difference between Driver and Executor?

| Driver | Executor |
|---|---|
| Coordinates the application | Executes tasks |
| Creates jobs and stages | Processes data partitions |
| Schedules tasks | Runs scheduled tasks |
| Tracks execution | Reports task status |
| Usually one per application | Usually multiple per application |

### Q5. What is the role of the Cluster Manager?

It allocates cluster resources and launches executors. It does not normally schedule individual Spark tasks.

### Q6. Can one worker node have multiple executors?

Yes. Multiple executors can run on a worker node, depending on the cluster configuration and available resources.

### Q7. What happens if the Driver fails?

The Spark application generally fails because the Driver coordinates execution and task scheduling. Recovery behavior depends on the deployment platform and orchestration setup.

### Q8. What happens if an Executor fails?

Tasks running on that executor may be retried on other available resources. Spark can recompute lost intermediate data using lineage when necessary.

### Q9. What is the difference between client mode and cluster mode?

In client mode, the Driver runs on the submission machine. In cluster mode, the Driver runs inside the cluster.

### Q10. Do Executors process the entire dataset?

Executors process distributed partitions of the dataset. Multiple executors can process different partitions in parallel.

### Q11. What is the relationship between Driver, DAG Scheduler, and Task Scheduler?

The Driver contains the scheduling components. The DAG Scheduler divides jobs into stages, while the Task Scheduler schedules individual tasks on executors.

### Q12. What is the difference between a Worker Node and an Executor?

A Worker Node is a machine that provides resources. An Executor is a process running on a worker node that executes Spark tasks.

---

## 12. Common Mistakes

1. **Confusing Driver with Cluster Manager:** The Driver coordinates application execution; the Cluster Manager allocates resources.

2. **Confusing Worker Node with Executor:** A Worker Node is a machine; an Executor is a process running on that machine.

3. **Thinking Executors create jobs:** The Driver coordinates job creation and scheduling.

4. **Thinking one Executor runs only one task:** An executor can run multiple tasks concurrently, depending on its available cores and resources.

5. **Thinking all data is processed by the Driver:** Distributed processing happens on executors.

6. **Confusing deployment modes with execution modes:** Client and cluster modes describe where the Driver runs.

---

## 13. Quick Revision

| Component | Main Responsibility |
|---|---|
| Driver | Coordinates Spark application |
| SparkSession | Main entry point for DataFrame and SQL operations |
| SparkContext | Connects application to Spark cluster |
| DAG Scheduler | Divides jobs into stages |
| Task Scheduler | Schedules individual tasks |
| Cluster Manager | Allocates resources |
| Worker Node | Provides CPU, memory, and other resources |
| Executor | Executes tasks and processes partitions |
| Job | Computation triggered by an action |
| Stage | Group of tasks separated by shuffle dependencies |
| Task | Processes a partition for a stage |

### Remember This Execution Flow

```text
Application Submission
        |
        v
       Driver
        |
        v
  Cluster Manager
        |
        v
 Resource Allocation
        |
        v
     Executors
        |
        v
   Task Execution
        |
        v
  Result Processing
```

**One-line interview answer:**

Apache Spark follows a distributed architecture in which the Driver coordinates application execution, the Cluster Manager allocates resources, Worker Nodes host Executors, and Executors process data partitions by executing tasks in parallel.