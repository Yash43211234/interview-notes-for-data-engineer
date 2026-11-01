# Spark Jobs, Stages, and Tasks

## 1. Introduction

Apache Spark processes data in a distributed manner. When you execute a Spark application, Spark breaks the computation into smaller units so that multiple Executors can process data in parallel.

There are three important execution concepts:

1. **Job:** A computation triggered by an action.
2. **Stage:** A group of tasks that can execute without crossing a shuffle dependency.
3. **Task:** The smallest unit of Spark execution, generally processing one data partition.

The relationship is:

`Spark Application → Job → Stages → Tasks`

Understanding these concepts is essential for learning Spark execution, debugging performance issues, and preparing for Data Engineer interviews.

---

## 2. What Is a Spark Job?

A Spark Job is a computation triggered when an action is executed on a DataFrame, RDD, or Dataset.

Transformations generally build an execution plan, while actions trigger execution.

### Transformations

Transformations describe operations to perform on data.

Examples:

- `filter()`
- `select()`
- `map()`
- `groupBy()`
- `join()`
- `distinct()`

### Actions

Actions request a result or trigger execution.

Examples:

- `show()`
- `count()`
- `collect()`
- `take()`
- `write()`

### Example

```python
df = spark.read.parquet("sales/")

filtered_df = df.filter(
    df.amount > 1000
)

selected_df = filtered_df.select(
    "customer_id",
    "amount"
)

selected_df.show()
```

Execution behavior:

1. Spark reads the DataFrame definition.
2. `filter()` defines a transformation.
3. `select()` defines another transformation.
4. `show()` triggers a Spark job.
5. Spark creates an execution plan and processes the required data.

**Important:** Transformations are generally lazy. They do not immediately execute the complete computation.

Also, an action does not necessarily create exactly one job in every situation. Some operations can trigger additional jobs internally.

---

## 3. What Is a Spark Stage?

A Stage is a group of tasks that can execute together without crossing a shuffle dependency.

Spark divides a job into stages based on dependencies between transformations.

The most important concept here is the difference between narrow and wide transformations.

### Narrow Transformation

A narrow transformation is one where each output partition depends on a limited number of input partitions, typically one.

Examples:

- `filter()`
- `select()`
- `map()`
- `withColumn()` for ordinary column expressions

These operations can often be pipelined together in the same stage.

### Wide Transformation

A wide transformation requires data to be redistributed across partitions.

Examples:

- `groupBy()`
- `distinct()`
- `repartition()`
- Many joins
- `orderBy()`

These operations commonly involve a shuffle and introduce stage boundaries.

**Important:** Not every join or operation necessarily causes a shuffle. Spark's physical execution plan and chosen strategy determine whether data redistribution is required. For example, a broadcast join can avoid a shuffle on the streamed side.

---

## 4. What Is a Shuffle?

A shuffle is the process of redistributing data across partitions, usually across Executors, so that the next operation can process the required records together.

### Example: Group By

Suppose the dataset contains:

| Customer | City | Amount |
|---|---|---:|
| A | Delhi | 100 |
| B | Mumbai | 200 |
| C | Delhi | 150 |
| D | Mumbai | 300 |

You execute:

```python
result = df.groupBy("city").sum("amount")
```

Spark needs to bring records with the same city together for aggregation.

Conceptually:

- Delhi records must be grouped together.
- Mumbai records must be grouped together.
- Data may need to move between partitions and Executors.

This redistribution is called a shuffle.

<AsyncImage query="Apache Spark shuffle operation map stage reduce stage data redistribution diagram" aspectRatio="16:9" width="100%" fit="contain"/>

### Why is Shuffle Expensive?

Shuffle can involve:

- Network communication.
- Disk writes and reads.
- Serialization and deserialization.
- Data redistribution.
- Additional memory usage.

Large or inefficient shuffles can significantly increase Spark job execution time.

---

## 5. What Is a Spark Task?

A Task is the smallest unit of work scheduled by Spark.

A task generally processes one partition of data for a particular stage.

For example, suppose a stage has four input partitions.

Spark can create four tasks:

| Task | Partition |
|---|---|
| Task 1 | Partition 0 |
| Task 2 | Partition 1 |
| Task 3 | Partition 2 |
| Task 4 | Partition 3 |

These tasks can execute in parallel if sufficient Executor resources are available.

### Important Points

- Tasks belong to a stage.
- Stages belong to a job.
- Tasks are scheduled on Executors.
- A task processes a partition for its stage.
- A failed task may be retried.

**Remember:** The number of tasks in a stage is generally equal to the number of partitions processed by that stage.

The number of tasks is not necessarily equal to the number of Executors.

---

## 6. Relationship Between Jobs, Stages, and Tasks

<AsyncImage query="Apache Spark jobs stages tasks hierarchy diagram job stage tasks partitions" aspectRatio="16:9" width="100%" fit="contain"/>

Consider a simple example:

```python
df = spark.read.parquet("sales/")

filtered = df.filter("amount > 1000")

grouped = filtered.groupBy("city").sum("amount")

result = grouped.orderBy("sum(amount)")

result.show()
```

Conceptually, Spark may create a job containing multiple stages.

| Level | Responsibility |
|---|---|
| Job | Executes the computation triggered by the action |
| Stage 1 | Reads and filters data, then prepares shuffle data |
| Stage 2 | Performs aggregation and sorting as required by the physical plan |
| Tasks | Process individual partitions within each stage |

The exact number of stages depends on the physical execution plan, shuffle boundaries, and optimizations.

---

## 7. Step-by-Step Execution Flow

Let's understand how Spark processes a job.

### Step 1: Application submission

A Spark application is submitted to the cluster.

### Step 2: Driver initialization

The Driver creates the SparkSession and initializes the execution environment.

### Step 3: Transformations

The application defines transformations such as filtering, selecting, and grouping.

Spark builds a logical plan.

### Step 4: Action

An action such as `count()`, `show()`, or `write()` triggers execution.

### Step 5: Job creation

Spark creates a job for the requested computation.

### Step 6: Stage creation

The DAGScheduler examines dependencies and divides the job into stages.

Shuffle dependencies generally create stage boundaries.

### Step 7: Task creation

Each stage is divided into tasks based on the required partitions.

### Step 8: Task scheduling

The TaskScheduler schedules tasks on available Executors.

### Step 9: Execution

Executors process their assigned partitions.

### Step 10: Completion

Task status and required results are reported. The Driver coordinates the completion of the job.

---

## 8. Narrow vs Wide Transformations

This distinction is particularly important in Spark interviews.

| Feature | Narrow Transformation | Wide Transformation |
|---|---|---|
| Data dependency | Limited input partition dependencies | Can depend on multiple input partitions |
| Shuffle | Usually no shuffle | Commonly requires shuffle |
| Stage boundary | Usually no new stage | Usually creates a stage boundary |
| Performance | Often cheaper | Often more expensive |
| Examples | `filter`, `select`, `map` | `groupBy`, `distinct`, `repartition` |

### Example: Narrow Transformation

```python
filtered = df.filter("age > 25")
selected = filtered.select("name", "age")
```

These operations can often be executed together in one stage.

### Example: Wide Transformation

```python
grouped = df.groupBy("department").count()
```

Spark commonly redistributes records based on the grouping key.

This requires a shuffle and generally introduces a stage boundary.

---

## 9. How Partitions Affect Tasks

Partitions are logical divisions of data that allow Spark to process data in parallel.

The number of partitions influences the number of tasks.

Suppose a DataFrame has eight partitions and a stage processes all eight.

Spark generally creates eight tasks for that stage.

### Example

```python
df = spark.read.parquet("sales/")

print(df.rdd.getNumPartitions())
```

This returns the number of partitions exposed by the DataFrame's underlying RDD.

You can change partitioning:

```python
df = df.repartition(8)
```

Or reduce partitions:

```python
df = df.coalesce(4)
```

### Repartition vs Coalesce

| Feature | Repartition | Coalesce |
|---|---|---|
| Purpose | Increase or decrease partitions | Usually reduce partitions |
| Shuffle | Generally causes shuffle | Usually avoids a full shuffle |
| Data distribution | Can redistribute data | May result in uneven partitions |
| Common use | Redistribute data for parallelism | Reduce small partitions |

**Important:** More partitions do not always mean better performance. Too many small partitions introduce scheduling overhead, while too few large partitions can reduce parallelism.

---

## 10. Parallelism and Executor Cores

Executors execute tasks using their available CPU resources.

For example:

- 2 Executors.
- 4 cores per Executor.
- 8 total Executor cores.

Under suitable conditions, the application can run approximately eight tasks concurrently.

<AsyncImage query="Apache Spark executor cores parallel tasks multiple executors worker nodes diagram" aspectRatio="16:9" width="100%" fit="contain"/>

Suppose a stage has 20 tasks.

With eight available task slots:

- First wave: 8 tasks.
- Second wave: 8 tasks.
- Third wave: 4 tasks.

The actual scheduling depends on available resources, task locality, resource configuration, and other workloads.

**Important:** Having 20 tasks does not mean all 20 execute simultaneously.

---

## 11. Multiple Actions and Multiple Jobs

Each action can trigger a separate Spark job.

Example:

```python
df = spark.read.parquet("sales/")

filtered = df.filter("amount > 1000")

filtered.count()

filtered.show()

filtered.write.mode("overwrite").parquet("output/")
```

These actions can trigger separate jobs:

1. `count()`
2. `show()`
3. `write()`

Spark may reuse cached data if it has been persisted, but without persistence it may need to recompute the upstream transformations.

### Avoid Unnecessary Repeated Computation

```python
filtered = df.filter("amount > 1000").cache()

filtered.count()
filtered.show()
filtered.write.parquet("output/")
```

Caching can help when the same result is reused multiple times.

However, caching is not always beneficial. It consumes storage resources and should be used when repeated computation justifies the cost.

---

## 12. How to View Jobs, Stages, and Tasks

Spark provides a Web UI to inspect application execution.

The Spark UI commonly runs on port `4040` for a local application, although the port can vary.

Important tabs include:

| Spark UI Tab | Information |
|---|---|
| Jobs | Job status, duration, and associated stages |
| Stages | Stage details, task execution, shuffle metrics |
| Storage | Cached and persisted datasets |
| Executors | Executor memory, CPU, and task information |
| SQL/DataFrame | SQL queries and physical execution plans |
| Environment | Spark configuration properties |

### What to Check During Performance Debugging

- Number of jobs and stages.
- Number of tasks in each stage.
- Task duration.
- Shuffle read and shuffle write.
- Failed and retried tasks.
- Executor memory usage.
- Skew between task execution times.

The Spark UI helps identify where the application spends most of its execution time.

---

## 13. Common Performance Problems

### 1. Too Many Small Tasks

When a dataset is divided into too many small partitions, Spark spends more time scheduling and managing tasks.

**Possible solution:** Review partition count and use suitable repartitioning or coalescing.

### 2. Too Few Partitions

A small number of partitions can limit parallelism and leave Executor resources unused.

**Possible solution:** Increase partitions when the workload and cluster resources justify it.

### 3. Expensive Shuffle

Operations such as large joins, aggregations, and sorting can create expensive shuffles.

**Possible solution:**

- Filter data early.
- Select only required columns.
- Use suitable join strategies.
- Review partitioning.
- Avoid unnecessary repartitioning.

### 4. Data Skew

Some partitions may contain much more data than others.

This can cause a few tasks to take significantly longer than the rest.

**Possible solution:**

- Enable Adaptive Query Execution where appropriate.
- Use salting for suitable skewed joins.
- Review partitioning and join keys.
- Check task duration differences in the Spark UI.

### 5. Repeated Computation

Multiple actions on the same uncached DataFrame may recompute transformations.

**Possible solution:** Consider caching or persisting reused intermediate results.

### 6. Executor Memory Problems

Large partitions, joins, aggregations, and caching can increase memory pressure.

**Possible solution:** Optimize the workload, manage partition sizes, and adjust resources when justified.

---

## 14. Common Interview Questions

**Q1. What is a Spark Job?**

A Spark Job is a computation triggered by an action such as `count()`, `show()`, or `write()`.

**Q2. What is a Stage?**

A Stage is a group of tasks that can execute without crossing a shuffle dependency.

**Q3. What is a Task?**

A Task is the smallest unit of Spark execution and generally processes one partition of data.

**Q4. What is the relationship between Job, Stage, and Task?**

A job contains one or more stages, and each stage contains one or more tasks.

**Q5. What creates a new stage in Spark?**

A shuffle dependency generally creates a stage boundary.

**Q6. What is the difference between narrow and wide transformations?**

Narrow transformations have limited partition dependencies and usually avoid shuffling. Wide transformations redistribute data across partitions and commonly require shuffling.

**Q7. Does every transformation create a new job?**

No. Transformations are generally lazy. An action triggers execution.

**Q8. Does every action create exactly one job?**

Not necessarily. An action commonly triggers a job, but some operations can trigger additional internal jobs.

**Q9. Is the number of tasks equal to the number of Executors?**

No. The number of tasks is generally determined by the number of partitions processed by a stage. Executors can execute multiple tasks over time, subject to available cores.

**Q10. Can multiple tasks run on one Executor?**

Yes. An Executor can run multiple tasks concurrently if it has sufficient available cores and resources.

**Q11. Why is shuffle expensive?**

Shuffle involves data redistribution, network transfer, serialization, and often disk I/O.

**Q12. How can you identify a slow stage?**

Use the Spark UI's Stages tab to inspect execution duration, task duration, shuffle metrics, and failures.

**Q13. What happens if a task fails?**

Spark can retry the failed task according to its retry configuration. If failures persist beyond allowed limits, the stage or application can fail.

**Q14. Can Spark combine multiple transformations into one stage?**

Yes. Narrow transformations can often be pipelined together within a stage.

**Q15. What is the difference between a job and an application?**

A Spark application is the complete running program. It can contain multiple jobs, and each job can contain multiple stages.

---

## 15. Quick Revision

| Concept | Remember |
|---|---|
| Application | Complete Spark program |
| Job | Triggered by an action |
| Stage | Group of tasks separated by shuffle dependencies |
| Task | Processes a partition |
| Partition | Logical division of data |
| Shuffle | Redistributes data across partitions |
| Narrow transformation | Usually no shuffle |
| Wide transformation | Commonly causes shuffle |
| DAGScheduler | Divides jobs into stages |
| TaskScheduler | Schedules tasks on Executors |
| Spark UI | Monitors jobs, stages, tasks, and performance |

### Execution Flow

`Action → Job → Stages → Tasks → Executors`

### Most Important Interview Points

1. An action triggers Spark execution.
2. A job is divided into stages.
3. Shuffle dependencies generally create stage boundaries.
4. Each stage is divided into tasks based on partitions.
5. Executors execute tasks in parallel.
6. The Spark UI helps identify slow stages, expensive shuffles, and task failures.

**Remember:** Job represents the computation, Stage represents a group of tasks with compatible dependencies, and Task represents the actual unit of work performed on a partition.