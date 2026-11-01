# Spark Execution Flow — Interview Notes

## 1. What Is Spark Execution Flow?

Spark execution flow describes how Apache Spark processes a program, from the moment an action is triggered until the final result is produced.

When you write PySpark code, Spark does not necessarily execute every operation immediately. It builds an execution plan, optimizes that plan, divides the work into stages and tasks, and executes them across the available cluster resources.

**Basic flow:**

1. Write Spark transformations.
2. Trigger an action.
3. Build a logical execution plan.
4. Optimize the logical plan.
5. Generate a physical execution plan.
6. Create stages and tasks.
7. Schedule tasks on executors.
8. Execute tasks and return or store the result.

---

## 2. Important Components Involved

Before understanding the execution flow, understand these components.

| Component | Responsibility |
|---|---|
| Driver | Coordinates the Spark application and manages execution |
| SparkSession | Entry point for working with Spark DataFrames and SQL |
| SparkContext | Connects the driver to the cluster and coordinates execution |
| Cluster Manager | Allocates resources for the application |
| Executor | Runs tasks and stores cached data |
| Job | Work triggered by an action |
| Stage | A group of tasks that can execute without crossing a shuffle boundary |
| Task | Smallest unit of execution, usually processing one partition |

### Driver

The driver is the main process responsible for coordinating a Spark application.

It:
- Creates the Spark execution context.
- Builds and optimizes execution plans.
- Requests resources through the cluster manager.
- Creates jobs, stages, and tasks.
- Coordinates task execution and collects results when requested.

### Executor

Executors are processes launched for a Spark application.

They:
- Execute tasks assigned by the driver.
- Process data partitions.
- Store cached or persisted data.
- Send task status and results back to the driver.

**Important:** The driver coordinates the work; executors perform the distributed computation.

---

## 3. Step-by-Step Spark Execution Flow

Let's understand the complete flow using a simple PySpark example.

```python
orders_df = spark.read.csv(
    "/data/orders.csv",
    header=True,
    inferSchema=True
)

filtered_df = orders_df.filter(
    orders_df.amount > 1000
)

result_df = filtered_df.groupBy(
    "customer_id"
).sum("amount")

result_df.show()
```

### Step 1: Create a SparkSession

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("OrderAnalysis")
    .getOrCreate()
)
```

SparkSession provides the entry point for DataFrame and SQL operations.

The Spark application runs with a driver process.

### Step 2: Read the Data

```python
orders_df = spark.read.csv(
    "/data/orders.csv",
    header=True,
    inferSchema=True
)
```

Spark creates a DataFrame representing the source data.

DataFrame transformations are generally lazy. Defining a DataFrame does not mean all the source records have already been processed.

### Step 3: Apply Transformations

```python
filtered_df = orders_df.filter(
    orders_df.amount > 1000
)

result_df = filtered_df.groupBy(
    "customer_id"
).sum("amount")
```

Here:
- `filter()` selects records where the amount is greater than 1000.
- `groupBy()` groups records by customer.
- `sum()` calculates the total amount for each customer.

These transformations build up the execution plan.

Spark does not need to execute the complete computation immediately.

### Step 4: Trigger an Action

```python
result_df.show()
```

`show()` is an action.

It triggers execution of the required computation to produce the requested output.

Other common actions include:

```python
df.count()
df.collect()
df.first()
df.take(10)
df.write.parquet("/output/orders")
```

**Remember:** Transformations define what to do. Actions trigger execution.

### Step 5: Build the Logical Plan

Spark creates a logical representation of the requested operations.

Conceptually, our example looks like this:

```text
Read CSV
   |
Filter amount > 1000
   |
Group By customer_id
   |
Sum(amount)
```

The logical plan describes *what* operations need to happen, not necessarily the exact physical execution strategy.

### Step 6: Optimize the Logical Plan

Spark SQL's Catalyst Optimizer improves the logical plan.

Common optimization techniques include:

- Predicate pushdown.
- Column pruning.
- Constant folding.
- Combining compatible operations.
- Simplifying expressions.

Example:

```python
df = (
    spark.read.parquet("/data/orders")
    .filter("amount > 1000")
    .select("customer_id", "amount")
)
```

Spark may push the filter closer to the data source and read only the required columns when the source supports those optimizations.

This reduces unnecessary data processing.

### Step 7: Generate the Physical Plan

After optimization, Spark selects physical operators to execute the computation.

For example, a grouping operation may use a hash aggregation strategy.

A physical plan can include operators such as:
- File scan.
- Filter.
- Project.
- Hash aggregate.
- Exchange.
- Sort.

You can inspect the plan using:

```python
result_df.explain(True)
```

`explain(True)` displays logical and physical plan information, including optimization details.

### Step 8: Create Jobs, Stages, and Tasks

When an action is triggered, Spark creates a job for the required computation.

The scheduler divides the job into stages.

A stage contains tasks that can execute together without crossing a shuffle dependency.

A task generally processes one partition.

For example, if a stage has 4 partitions, it normally has 4 tasks.

```text
Spark Application
       |
       v
      Job
       |
       v
     Stage 1
       |
   4 Tasks
       |
       v
     Stage 2
       |
   4 Tasks
       |
       v
     Result
```

The exact number of stages and tasks depends on the execution plan, shuffle boundaries, and partitioning.

### Step 9: Schedule Tasks on Executors

The driver sends tasks to executors through Spark's scheduling mechanisms.

Executors run tasks using their available resources.

For example:

```text
             Driver
               |
       Cluster Manager
               |
       Resource Allocation
               |
       -------------------
       |                 |
   Executor 1        Executor 2
       |                 |
    Task 1             Task 3
    Task 2             Task 4
```

This is a simplified illustration. Real execution can involve multiple executors, task slots, and stages.

### Step 10: Produce the Result

After successful execution:

- Results may be returned to the driver for actions such as `collect()`.
- Results may be displayed by `show()`.
- Results may be written to external storage by write operations.

For large datasets, writing the result to distributed storage is generally more appropriate than collecting all records to the driver.

---

## 4. Understand the Relationship Between Job, Stage, and Task

These three terms are frequently asked in Spark interviews.

### Job

A job is created when an action triggers execution.

Example:

```python
df.count()
```

This action triggers a Spark job.

### Stage

A job is divided into stages based on dependencies, particularly shuffle boundaries.

Two important types of dependencies are:

- Narrow dependency.
- Wide dependency.

A shuffle generally separates stages.

### Task

A task is the smallest execution unit scheduled by Spark.

Each task normally processes one partition of data for a particular stage.

### Simple comparison

| Concept | Meaning | Example |
|---|---|---|
| Job | Complete computation triggered by an action | `df.count()` |
| Stage | Group of tasks separated by shuffle boundaries | Stage before aggregation shuffle |
| Task | Processes a partition | Processing partition 0 |

**Easy way to remember:**

- Job = What computation needs to finish?
- Stage = Which part of the computation can run together?
- Task = What work does one execution unit perform?

---

## 5. Example: How a Shuffle Creates Multiple Stages

Consider:

```python
result_df = (
    orders_df
    .filter("amount > 1000")
    .groupBy("customer_id")
    .sum("amount")
)

result_df.show()
```

The `groupBy()` operation may require records with the same `customer_id` to be brought together.

This requires a shuffle.

Conceptual execution:

```text
             Job
              |
              v
          Stage 1
              |
       Filter records
              |
       Partial aggregation
              |
           Shuffle
              |
              v
          Stage 2
              |
       Final aggregation
              |
              v
           Result
```

### Why does Spark shuffle data?

Suppose records for the same customer are spread across multiple partitions.

Before Spark can calculate the complete sum for each customer, it must bring the relevant records together according to the grouping key.

This data redistribution is called a shuffle.

**Important:** Not every transformation creates a shuffle. Operations such as `filter()` and `select()` usually have narrow dependencies.

---

## 6. How to Inspect Spark Execution

Two useful tools for understanding execution are `explain()` and the Spark UI.

### A. DataFrame explain

```python
df.explain()
```

Displays the physical execution plan by default.

```python
df.explain(True)
```

Displays extended plan information.

Use it to understand:
- Which operations Spark performs.
- Whether a shuffle is required.
- Whether filters and projections are pushed down.
- Which physical operators are selected.

### B. Spark UI

The Spark UI helps you inspect actual application execution.

Common sections:

| Section | What it shows |
|---|---|
| Jobs | Jobs triggered by actions |
| Stages | Stage execution details |
| Tasks | Individual task information |
| Executors | Executor resources and activity |
| SQL / DataFrame | SQL queries and execution plans |
| Storage | Cached and persisted datasets |

Useful metrics include:
- Task duration.
- Shuffle read and write.
- Input and output size.
- Executor memory.
- Failed tasks.
- Data skew indicators.

**Interview point:** `explain()` shows the execution plan, while the Spark UI helps analyze actual execution behavior and performance.

---

## 7. Important Execution Concepts

### Lazy Evaluation

Spark delays executing transformations until an action is called.

```python
df1 = df.filter("amount > 1000")
df2 = df1.select("customer_id", "amount")
```

These transformations build the computation plan.

Execution is triggered when an action is called:

```python
df2.show()
```

### DAG (Directed Acyclic Graph)

Spark represents the computation as a DAG.

- Directed: Operations follow a defined direction.
- Acyclic: The graph does not contain a cycle.
- Graph: Operations and their dependencies are represented as connected nodes.

The DAG Scheduler uses dependencies to divide jobs into stages.

### Shuffle

Shuffle redistributes data across partitions, usually across executors.

It is common in:
- `groupBy()`.
- `distinct()`.
- `join()`.
- `repartition()`.
- Some window operations.

### Partition

A partition is a logical division of a dataset.

Spark processes partitions in parallel through tasks.

More partitions can provide more parallelism, but too many small partitions can create scheduling and processing overhead.

---

## 8. Common Interview Questions

**Q1. Explain the complete Spark execution flow.**

When an action is triggered, Spark builds and optimizes the logical plan, generates a physical plan, and creates a job. The DAG Scheduler divides the job into stages at shuffle boundaries. The Task Scheduler creates and schedules tasks on executors. Executors process partitions and return results or write output.

**Q2. What triggers a Spark job?**

An action such as `count()`, `show()`, `collect()`, or a write operation triggers execution of the required computation.

**Q3. What is the difference between a job, stage, and task?**

A job represents the computation triggered by an action. A stage is a group of tasks separated by shuffle dependencies. A task processes one partition for a stage.

**Q4. What is the role of the driver?**

The driver coordinates the Spark application, creates execution plans, requests resources, schedules work, and tracks execution.

**Q5. What is the role of an executor?**

Executors run tasks, process partitions, cache data, and report execution status to the driver.

**Q6. What is the Catalyst Optimizer?**

It is Spark SQL's query optimizer. It improves logical plans and helps Spark choose an efficient physical execution plan.

**Q7. What is the DAG Scheduler?**

It divides a job into stages based on dependencies, particularly shuffle boundaries, and submits stages for execution.

**Q8. What is the Task Scheduler?**

It schedules individual tasks on available executor resources.

**Q9. Does every transformation create a new job?**

No. Transformations are generally lazy and build the execution plan. An action triggers the required job.

**Q10. Does every action create exactly one job?**

Not necessarily. An action can trigger one or more jobs depending on the operation and execution details. Avoid assuming a one-to-one relationship.

**Q11. What happens when an executor fails?**

Spark can retry failed tasks and, when possible, recompute lost partitions using lineage. Repeated failures or insufficient resources can still cause the application to fail.

**Q12. How do you investigate a slow Spark job?**

Start with the Spark UI and execution plan. Check shuffle volume, task duration, data skew, partition sizes, executor memory, and input/output performance.

---

## 9. Common Mistakes

1. **Confusing driver and executor:** The driver coordinates; executors execute tasks.
2. **Confusing job and stage:** A job can contain multiple stages.
3. **Assuming every transformation runs immediately:** Spark uses lazy evaluation for most DataFrame transformations.
4. **Assuming every transformation causes a shuffle:** Many operations have narrow dependencies.
5. **Assuming one task processes the entire DataFrame:** Tasks generally process individual partitions.
6. **Assuming more partitions always improve performance:** Excessive small partitions create overhead.
7. **Confusing execution plan with actual execution:** `explain()` shows the plan; the Spark UI shows runtime behavior.
8. **Assuming every action produces exactly one job:** Some actions can involve multiple jobs.

---

## 10. Quick Revision

| Term | One-line meaning |
|---|---|
| SparkSession | Entry point for DataFrame and SQL operations |
| Driver | Coordinates application execution |
| Cluster Manager | Allocates cluster resources |
| Executor | Executes tasks and stores cached data |
| Transformation | Defines a data operation |
| Action | Triggers execution |
| Logical Plan | Describes what computation is required |
| Catalyst Optimizer | Optimizes the logical plan |
| Physical Plan | Describes how Spark will execute the computation |
| DAG | Graph representing computation dependencies |
| Job | Computation triggered by an action |
| Stage | Group of tasks separated by shuffle dependencies |
| Task | Processes a partition |
| Partition | Logical division of data |
| Shuffle | Redistributes data between partitions |

### Complete flow to memorize

```text
Write PySpark Code
        |
        v
Apply Transformations
        |
        v
Build Logical Plan
        |
        v
Trigger Action
        |
        v
Catalyst Optimization
        |
        v
Physical Execution Plan
        |
        v
DAG Scheduler
        |
        v
Create Stages
        |
        v
Create Tasks
        |
        v
Task Scheduler
        |
        v
Executors Execute Tasks
        |
        v
Return or Write Results
```

---

## 11. Self-Check

Before considering this topic complete, make sure you can:

- [ ] Explain the Spark execution flow without looking at notes.
- [ ] Explain the responsibilities of the driver and executor.
- [ ] Differentiate job, stage, and task.
- [ ] Explain lazy evaluation.
- [ ] Describe the role of the Catalyst Optimizer.
- [ ] Explain how the DAG Scheduler creates stages.
- [ ] Explain why shuffle creates stage boundaries.
- [ ] Inspect an execution plan using `explain(True)`.
- [ ] Use the Spark UI to investigate slow jobs.

**Interview takeaway:** Understand the complete journey: transformations build the plan, an action triggers execution, Spark optimizes the plan, the DAG Scheduler creates stages, the Task Scheduler assigns tasks, and executors process the data.