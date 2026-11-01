# Spark Driver, Executors, and Cluster Manager

## 1. Introduction

Apache Spark is a distributed data-processing engine. It processes large datasets by distributing work across multiple machines or multiple processes.

To understand how Spark executes a job, you must understand three important components:

1. **Driver:** Coordinates the Spark application and manages execution.
2. **Executors:** Execute tasks and store data during computation.
3. **Cluster Manager:** Allocates computing resources to Spark applications.

These components work together to execute distributed operations efficiently.

---

## 2. Spark Application Architecture

A Spark application generally consists of one Driver and one or more Executors.

<AsyncImage query="Apache Spark architecture diagram driver executors cluster manager worker nodes labeled" aspectRatio="16:9" width="100%" fit="contain"/>

The basic execution flow is:

1. The user submits a Spark application.
2. The Driver starts and creates a SparkSession and SparkContext.
3. The Driver requests resources from the Cluster Manager.
4. The Cluster Manager allocates resources and starts Executors.
5. The Driver creates an execution plan and divides work into tasks.
6. Executors execute the tasks on data partitions.
7. Executors send task status and results back to the Driver.
8. The Driver coordinates the application until completion.

**Important:** The Driver coordinates execution; Executors perform the distributed data processing.

---

## 3. Driver

The Driver is the central process responsible for managing a Spark application.

It runs the application's main program and coordinates the execution of Spark jobs.

### Main responsibilities

- Creates the SparkSession and SparkContext.
- Maintains information about the Spark application.
- Converts user operations into execution plans.
- Creates jobs, stages, and tasks.
- Schedules tasks on available Executors.
- Tracks task execution and failures.
- Collects results returned by Executors.
- Coordinates the overall application lifecycle.

### Example

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("CustomerProcessing")
    .getOrCreate()
)

df = spark.read.csv(
    "customers.csv",
    header=True,
    inferSchema=True
)

result = df.groupBy("city").count()

result.show()
```

In this example:

- The Python application runs through the Driver.
- `SparkSession` provides the interface to Spark.
- Spark builds an execution plan for the transformations.
- When `show()` is called, Spark executes the required computation.
- Executors process the data partitions.
- The Driver coordinates execution and displays the result.

### Important Driver components

**SparkSession**

The entry point for working with Spark SQL, DataFrames, and Datasets.

**SparkContext**

The connection between the Driver and the Spark execution environment. It coordinates access to the cluster resources and execution services.

**DAGScheduler**

Converts a job into stages based on dependencies between operations.

**TaskScheduler**

Schedules tasks on available Executors and coordinates task execution.

**Note:** SparkSession is the standard entry point in modern PySpark applications. SparkContext remains an important underlying execution component.

---

## 4. Executors

Executors are processes launched for a Spark application on worker nodes.

They perform the actual distributed computation assigned by the Driver.

### Main responsibilities

- Execute tasks assigned by the Driver.
- Process data partitions.
- Store cached or persisted data.
- Perform shuffle-related operations.
- Return task results and status information.
- Report execution metrics to the Driver.

### Example

Suppose you have a dataset containing 1 million customer records.

Spark divides the dataset into multiple partitions.

<AsyncImage query="Apache Spark distributed data partitions executors worker nodes architecture diagram" aspectRatio="16:9" width="100%" fit="contain"/>

If the dataset has four partitions and four tasks can run simultaneously:

| Executor | Task | Data partition |
|---|---|---|
| Executor 1 | Task 1 | Partition 0 |
| Executor 2 | Task 2 | Partition 1 |
| Executor 3 | Task 3 | Partition 2 |
| Executor 4 | Task 4 | Partition 3 |

Each Executor processes its assigned partition independently.

The Driver coordinates the tasks and combines the results when necessary.

### Executor resources

An Executor generally uses:

- **Executor cores:** CPU resources available for running tasks.
- **Executor memory:** Memory available for execution and storage.
- **Storage memory:** Memory used for cached or persisted data.
- **Execution memory:** Memory used for joins, aggregations, sorting, and shuffles.

Executors can also use local disk when data cannot fit in memory or when shuffle operations require disk storage.

### Important point

Executors are generally created for a particular Spark application. Different Spark applications normally have their own Driver and Executor processes.

---

## 5. Cluster Manager

A Cluster Manager is responsible for allocating resources to Spark applications.

It decides where application processes can run and helps provide the requested CPU and memory resources.

Spark supports different cluster-management options.

### Common Cluster Managers

| Cluster Manager | Description |
|---|---|
| Standalone | Spark's built-in cluster manager |
| YARN | Resource manager commonly used in Hadoop environments |
| Kubernetes | Manages Spark workloads using Kubernetes resources |
| Local mode | Runs Spark locally without a separate cluster manager |

### 1. Standalone

Spark Standalone is Spark's built-in cluster manager.

It manages worker nodes and allocates resources to Spark applications.

### 2. YARN

YARN stands for Yet Another Resource Negotiator.

It manages cluster resources in Hadoop environments.

It allows Spark applications to share cluster resources with other workloads.

### 3. Kubernetes

Kubernetes manages containerized applications.

Spark can run its Driver and Executors inside Kubernetes pods.

### 4. Local Mode

In local mode, Spark runs on the same machine.

It is useful for development, learning, and testing.

Example:

```python
spark = (
    SparkSession.builder
    .master("local[*]")
    .appName("LocalSparkApp")
    .getOrCreate()
)
```

Here, `local[*]` tells Spark to use the available logical CPU cores on the local machine.

**Important:** Local mode is not a distributed multi-machine cluster.

---

## 6. Worker Node vs Executor

These terms are related but do not mean the same thing.

| Worker Node | Executor |
|---|---|
| A machine or computing node in the cluster | A process running on a worker node |
| Provides CPU, memory, and other resources | Uses allocated resources to execute tasks |
| Can host multiple Executors | Belongs to a particular Spark application |
| Managed by the cluster environment | Managed as part of a Spark application |

A worker node provides the computing environment. An Executor is the process that uses allocated resources to run Spark tasks.

**Remember:** Worker Node = Machine or computing node. Executor = Application process.

---

## 7. How Driver, Executors, and Cluster Manager Work Together

Consider a Spark application that reads customer data, filters records, and calculates total sales by city.

```python
df = spark.read.parquet("sales/")

filtered_df = df.filter(
    df.amount > 1000
)

result = filtered_df.groupBy("city").sum("amount")

result.show()
```

The execution process is:

<AsyncImage query="Apache Spark job execution workflow driver DAG scheduler task scheduler cluster manager executors stages tasks diagram" aspectRatio="16:9" width="100%" fit="contain"/>

### Step-by-step execution

**Step 1: Application submission**

The Spark application is submitted to the cluster.

**Step 2: Driver initialization**

The Driver starts, creates the SparkSession, and initializes the Spark execution context.

**Step 3: Resource allocation**

The Driver requests resources. The Cluster Manager allocates resources and launches Executors.

**Step 4: Logical plan**

Spark builds a logical plan representing the DataFrame operations.

**Step 5: Physical plan**

Spark optimizes the logical plan and creates a physical execution plan.

**Step 6: Job creation**

When an action such as `show()` is executed, Spark triggers a job.

**Step 7: Stage creation**

The DAGScheduler divides the job into stages based on dependencies, particularly shuffle boundaries.

**Step 8: Task creation**

Each stage is divided into tasks. Typically, one task processes one partition in that stage.

**Step 9: Task execution**

The Driver's scheduling components assign tasks to available Executors.

**Step 10: Result collection**

Executors return task status and required results. The Driver coordinates completion and presents the result.

---

## 8. Relationship Between Jobs, Stages, and Tasks

These three terms are essential for understanding Spark execution.

| Term | Meaning |
|---|---|
| Job | A computation triggered by an action |
| Stage | A group of tasks that can execute without crossing a shuffle dependency |
| Task | The smallest unit of Spark execution, generally processing one partition |

### Example

```python
df = spark.read.parquet("sales/")

filtered = df.filter("amount > 1000")

grouped = filtered.groupBy("city").sum("amount")

grouped.show()
```

The `show()` action triggers execution.

Spark may create multiple stages depending on the physical plan and shuffle requirements.

A shuffle often creates a stage boundary because data must be redistributed between partitions.

**Important:** Not every transformation creates a new stage. Narrow transformations can often be pipelined together within a stage.

---

## 9. Driver vs Executor vs Cluster Manager

| Feature | Driver | Executor | Cluster Manager |
|---|---|---|---|
| Main purpose | Coordinate application execution | Execute distributed tasks | Allocate resources |
| Runs application logic | Yes | Executes scheduled tasks | No |
| Creates jobs and stages | Yes | No | No |
| Executes tasks | No, not as the normal distributed task worker | Yes | No |
| Manages resources | Requests and coordinates resources | Uses allocated resources | Allocates resources |
| Stores cached data | Not its primary responsibility | Yes | No |
| Tracks task execution | Coordinates tracking | Reports status | Manages cluster resources |
| Application relationship | One per application | One or more per application | Shared across applications |

---

## 10. Deployment Modes

Spark applications can run in different deployment modes.

The deployment mode determines where the Driver runs.

### Client Mode

The Driver runs on the machine from which the application is submitted.

For example, you submit a Spark application from your local machine, and the Driver stays there while Executors run on the cluster.

### Cluster Mode

The Driver runs inside the cluster, generally on a worker node or in a cluster-managed container.

The client submits the application and does not need to host the Driver process.

### Comparison

| Feature | Client Mode | Cluster Mode |
|---|---|---|
| Driver location | Client machine | Inside the cluster |
| Executor location | Cluster | Cluster |
| Client dependency | Client remains important for Driver availability | Client can disconnect after submission |
| Common use | Interactive development and notebooks | Production batch applications |

**Important:** Deployment mode is different from cluster-manager type.

For example, Spark can use YARN or Kubernetes with different deployment configurations.

---

## 11. Common Performance and Failure Scenarios

### 1. Driver Out of Memory

The Driver may run out of memory when too much data or too many objects are collected to the Driver.

Example:

```python
data = df.collect()
```

`collect()` transfers all rows to the Driver.

For large datasets, this can cause memory problems.

Better approach:

```python
df.show(20)
```

Or perform aggregations and transformations in Spark rather than transferring the entire dataset to the Driver.

### 2. Executor Out of Memory

An Executor may run out of memory when it processes large partitions, performs expensive aggregations, or caches too much data.

Possible improvements:

- Increase available Executor memory when appropriate.
- Reduce partition size.
- Avoid unnecessary caching.
- Optimize joins and aggregations.
- Handle data skew.

### 3. Too Few Executors

A small number of Executors may limit parallel processing and increase execution time.

The appropriate number depends on the workload, available resources, and partition count.

### 4. Too Many Small Tasks

Too many tiny partitions can introduce scheduling overhead.

Adjust partitioning when necessary.

### 5. Data Skew

Data skew occurs when some partitions contain significantly more data than others.

As a result, some Executors finish quickly while others take much longer.

Possible solutions:

- Repartition data appropriately.
- Use salting for suitable skewed joins.
- Enable Adaptive Query Execution (AQE) where appropriate.
- Review join strategies and partitioning.

### 6. Executor Failure

An Executor may fail due to memory errors, hardware issues, or process termination.

Spark can retry failed tasks, subject to its configuration and failure limits. Persistent failures can cause the application to fail.

---

## 12. Important Spark Configurations

These configurations help control the resources available to a Spark application.

| Configuration | Purpose |
|---|---|
| `spark.driver.memory` | Memory allocated to the Driver |
| `spark.executor.memory` | Memory allocated to each Executor |
| `spark.executor.cores` | CPU cores allocated to each Executor |
| `spark.executor.instances` | Number of Executors requested in applicable configurations |
| `spark.driver.cores` | CPU cores allocated to the Driver in supported cluster configurations |
| `spark.sql.shuffle.partitions` | Default number of partitions for SQL shuffle operations |
| `spark.dynamicAllocation.enabled` | Enables dynamic adjustment of Executor count when supported |

Example:

```bash
spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --driver-memory 2g \
  --executor-memory 4g \
  --executor-cores 2 \
  --num-executors 3 \
  application.py
```

This example requests:

- YARN as the cluster manager.
- Cluster deployment mode.
- 2 GB Driver memory.
- 4 GB memory per Executor.
- 2 cores per Executor.
- 3 Executors.

Actual resource allocation depends on cluster availability, configuration, and resource-manager policies.

---

## 13. Spark Architecture in Databricks

In Databricks, you generally work with a Spark cluster or compute resource managed through the Databricks platform.

The underlying Spark architecture still includes:

- Driver.
- Executors.
- Worker nodes.
- Spark execution components.

A simplified view:

| Component | Responsibility |
|---|---|
| Databricks control plane | Manages the Databricks service and workspace operations |
| Compute plane | Provides the environment where workloads execute |
| Driver | Coordinates the Spark application |
| Worker nodes | Provide compute resources |
| Executors | Execute distributed Spark tasks |

The exact infrastructure and management details depend on the Databricks deployment and compute type.

**Important:** Databricks is a managed data and AI platform, not simply another name for Apache Spark.

---

## 14. Common Interview Questions

**Q1. What is the Driver in Spark?**

The Driver is the central process that runs the main application, creates the execution plan, schedules tasks, and coordinates execution.

**Q2. What is an Executor?**

An Executor is a process launched for a Spark application that executes tasks and can store cached data.

**Q3. What is a Cluster Manager?**

A Cluster Manager allocates resources to Spark applications and helps launch the required application processes.

**Q4. What is the difference between a Worker Node and an Executor?**

A Worker Node provides computing resources. An Executor is a process running on that node that executes Spark tasks.

**Q5. Can multiple Executors run on one Worker Node?**

Yes. A worker node can host multiple Executor processes if sufficient resources are available.

**Q6. Can multiple Spark applications share the same Cluster Manager?**

Yes. A cluster manager can allocate resources to multiple Spark applications.

**Q7. Can multiple Spark applications share the same Executor?**

Normally, no. Executors are associated with a specific Spark application.

**Q8. What happens when the Driver fails?**

The Spark application generally fails because the Driver coordinates scheduling and execution. Recovery behavior depends on the deployment environment and application configuration.

**Q9. What happens when an Executor fails?**

Tasks running on the failed Executor may be rescheduled on available Executors. Cached data on that Executor may need to be recomputed.

**Q10. What is the difference between Client and Cluster deployment modes?**

In Client Mode, the Driver runs on the submitting machine. In Cluster Mode, the Driver runs inside the cluster.

**Q11. Who creates jobs, stages, and tasks?**

The Driver coordinates their creation through Spark's scheduling components. The DAGScheduler creates stages, and the TaskScheduler schedules tasks.

**Q12. Does the Cluster Manager execute Spark tasks?**

No. It allocates resources and launches application processes. Executors execute Spark tasks.

**Q13. What is the relationship between partitions and tasks?**

A task generally processes one partition in a particular stage. Therefore, the number of tasks in a stage is generally equal to the number of partitions processed by that stage.

**Q14. Why should we avoid `collect()` on large datasets?**

Because it transfers all records to the Driver, potentially exhausting Driver memory.

---

## 15. Quick Revision

- **Driver:** Coordinates the Spark application.
- **Executor:** Executes tasks and stores cached data.
- **Cluster Manager:** Allocates computing resources.
- **Worker Node:** Machine or computing node providing resources.
- **SparkSession:** Main entry point for modern Spark applications.
- **SparkContext:** Connects the Driver to Spark execution services.
- **Job:** Computation triggered by an action.
- **Stage:** Group of tasks separated by shuffle dependencies.
- **Task:** Unit of work that generally processes one partition.
- **Client Mode:** Driver runs on the submitting machine.
- **Cluster Mode:** Driver runs inside the cluster.

### Remember This Execution Flow

`Application → Driver → Cluster Manager → Executors → Tasks → Results`

**Interview takeaway:** The Driver plans and coordinates, the Cluster Manager allocates resources, and the Executors perform distributed data processing.