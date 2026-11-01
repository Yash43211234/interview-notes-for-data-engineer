# Spark UI Debugging in PySpark

## 1. What Is Spark UI?

Spark UI is a web-based monitoring tool that helps developers understand how Spark applications execute.

It provides information about:

- Jobs and stages.
- Tasks and execution time.
- Executors and resource usage.
- Memory and disk spill.
- Shuffle read and write.
- SQL query execution plans.
- Cached and persisted DataFrames.

**Key idea:** Spark UI helps identify where a Spark job is slow and why it is slow.

Instead of randomly increasing memory or changing partitions, use Spark UI to find the actual bottleneck.

---

## 2. How to Access Spark UI

When a Spark application runs, Spark creates a web interface.

The default Spark UI port is:

```text
http://localhost:4040
```

If multiple Spark applications are running, Spark may use another available port.

### In Databricks

You can access the Spark UI from the cluster or compute interface, depending on the workspace and runtime version.

### In local Spark

Start your Spark application and open the Spark UI in your browser.

**Important:** The Spark UI is available while the application is running. For completed applications, you need event logging and a configured Spark History Server to review historical execution details.

---

## 3. Main Tabs in Spark UI

The main Spark UI contains several useful sections.

| Tab | Purpose |
|---|---|
| Jobs | Shows job execution and status |
| Stages | Shows stage-level execution details |
| Storage | Shows cached and persisted data |
| Environment | Shows Spark properties and configuration |
| Executors | Shows executor activity and resource usage |
| SQL / DataFrame | Shows SQL queries and execution plans |

The exact tabs and metrics may vary by Spark version and platform.

---

## 4. Jobs Tab

The Jobs tab provides an overview of Spark jobs.

A job is created when an action triggers execution of a DataFrame or RDD computation.

Examples of actions:

```python
df.count()
df.collect()
df.show()
df.write.parquet("/output/orders")
```

### What to check

- Job ID.
- Job description.
- Job duration.
- Job status.
- Number of stages.
- Completed and failed jobs.

### Example

Suppose your application has three jobs:

| Job | Duration | Status |
|---|---:|---|
| Job 0 | 10 seconds | Completed |
| Job 1 | 3 minutes | Completed |
| Job 2 | 5 seconds | Completed |

Job 1 takes significantly longer.

You should investigate its stages and execution plan to understand the reason.

**Interview point:** The Jobs tab helps identify which Spark jobs consume the most execution time.

---

## 5. Stages Tab

A Spark job is divided into stages.

A stage contains tasks that can execute in parallel.

Shuffle operations generally introduce stage boundaries.

Examples of operations that commonly involve shuffles:

- `groupBy()`
- `distinct()`
- `orderBy()`
- Many joins.
- `repartition()`.

### What to check

- Stage duration.
- Number of tasks.
- Completed and failed tasks.
- Shuffle read.
- Shuffle write.
- Input and output size.
- Memory spill.
- Disk spill.

### Example

| Stage | Tasks | Duration |
|---|---:|---:|
| Stage 0 | 20 | 8 seconds |
| Stage 1 | 20 | 15 seconds |
| Stage 2 | 20 | 2 minutes |

Stage 2 is taking much longer.

Possible reasons:

- Data skew.
- Large shuffle.
- Expensive aggregation.
- Memory pressure.
- Disk spill.

Do not assume the reason from duration alone. Inspect task-level metrics.

---

## 6. Tasks Tab: Identifying Slow Tasks

A task generally processes one partition for a particular stage.

Spark UI provides task-level execution information.

### What to check

- Task duration.
- Input size.
- Shuffle read.
- Shuffle write.
- Memory spill.
- Disk spill.
- Executor assigned to the task.

### Example: Data skew

Imagine a stage containing 10 tasks.

| Task | Duration |
|---|---:|
| Task 1 | 10 seconds |
| Task 2 | 12 seconds |
| Task 3 | 11 seconds |
| Task 4 | 9 seconds |
| Task 5 | 13 seconds |
| Task 6 | 10 seconds |
| Task 7 | 12 seconds |
| Task 8 | 11 seconds |
| Task 9 | 14 seconds |
| Task 10 | 4 minutes |

Task 10 is much slower than the others.

This may indicate:

- One partition contains significantly more data.
- A hot key is causing data skew.
- The task is experiencing heavy spill.
- The executor is facing resource pressure.

### How to investigate

1. Open the slow stage.
2. Compare task durations.
3. Compare shuffle read sizes.
4. Check memory and disk spill.
5. Identify whether one or a few tasks are significantly slower.

**Important:** A slow task does not automatically mean data skew. Executor problems, garbage collection, and other resource issues can also cause slow tasks.

---

## 7. Shuffle Read and Shuffle Write

Shuffle occurs when Spark redistributes data across partitions.

It commonly happens during joins, aggregations, sorting, and repartitioning.

### Shuffle write

Shuffle write represents data written by upstream tasks for later retrieval by downstream tasks.

### Shuffle read

Shuffle read represents data fetched by downstream tasks from shuffle output.

### What to check

- Total shuffle write.
- Total shuffle read.
- Shuffle read distribution across tasks.
- Remote versus local shuffle reads, where available.
- Fetch wait time.
- Spill metrics.

### Why is excessive shuffle a problem?

- Network communication increases.
- Disk I/O increases.
- Serialization overhead increases.
- Memory pressure can increase.
- Execution time can increase.

### Possible solutions

1. Filter data before joins and aggregations.
2. Select only required columns.
3. Use broadcast joins when appropriate.
4. Avoid unnecessary repartitioning.
5. Investigate skewed keys.
6. Review the physical execution plan.

**Remember:** Shuffle is not always bad. It is often necessary for distributed operations. The goal is to avoid unnecessary or excessive shuffle.

---

## 8. Memory Spill and Disk Spill

Spark uses memory for operations such as joins, aggregations, and sorting.

When sufficient execution memory is unavailable, intermediate data may spill to disk.

### What to check

- Memory spill.
- Disk spill.
- Executor memory usage.
- Task execution time.
- Data size per task.

### Example

A task processes a large partition and requires more execution memory than is available.

Spark may spill intermediate data to disk.

This can slow down execution because disk operations are generally more expensive than memory operations.

### Possible solutions

- Reduce the amount of data processed.
- Filter early.
- Handle data skew.
- Avoid unnecessarily large partitions.
- Review join strategies.
- Avoid unnecessary caching.
- Review executor resources when evidence supports it.

**Important:** Memory spill and disk spill are related but distinct metrics. Do not assume every memory-related performance issue can be fixed by increasing executor memory.

---

## 9. Executors Tab

Executors perform the actual tasks assigned by Spark.

The Executors tab provides information about executor activity and resources.

### What to check

| Metric | Meaning |
|---|---|
| Active tasks | Tasks currently running |
| Completed tasks | Tasks completed successfully |
| Failed tasks | Tasks that failed |
| Executor memory | Memory used by the executor |
| Storage memory | Memory used for cached or persisted data |
| Task time | Time spent executing tasks |
| Input | Data read by the executor |
| Shuffle read/write | Shuffle data handled by the executor |
| Failed tasks | Can help identify recurring failures |

### Common problems

**Problem 1: One executor is much slower**

Possible reasons:
- Uneven task distribution.
- Resource contention.
- Data skew.
- Executor-specific problems.

**Problem 2: Executor lost**

Possible reasons:
- Out-of-memory errors.
- Container or process termination.
- Node failure.
- Resource allocation problems.

**Problem 3: High memory usage**

Possible reasons:
- Large partitions.
- Expensive joins or aggregations.
- Excessive caching.
- Data skew.

Investigate executor logs and task metrics before changing resource configurations.

---

## 10. SQL / DataFrame Tab

The SQL tab is especially useful for DataFrame and Spark SQL performance debugging.

It displays query execution details and the associated execution plan.

### What to check

- Query duration.
- Execution plan.
- Join strategy.
- Scan operations.
- Exchange operations.
- Filter and projection operations.
- Runtime metrics.

### Example

```python
from pyspark.sql.functions import col

result = (
    orders.filter(col("amount") > 1000)
          .join(customers, "customer_id")
          .groupBy("country")
          .count()
)

result.explain("formatted")
```

You can also inspect the query in Spark UI after an action executes it.

### Important plan operators

| Operator | Meaning |
|---|---|
| FileScan | Reads data from files |
| Filter | Applies filtering conditions |
| Project | Selects or calculates columns |
| Exchange | Often indicates data redistribution |
| Sort | Sorts data |
| HashAggregate | Performs hash-based aggregation |
| SortMergeJoin | Performs a sort-merge join |
| BroadcastHashJoin | Performs a broadcast hash join |

**Important:** Plan operators and displayed metrics vary by Spark version and execution mode. An `Exchange` often indicates a shuffle or other data redistribution, but it should be interpreted in context.

---

## 11. Environment Tab

The Environment tab displays Spark configuration and environment details.

It is useful when investigating configuration-related performance problems.

### Examples of important configurations

| Configuration | Purpose |
|---|---|
| `spark.sql.shuffle.partitions` | Default number of shuffle partitions for Spark SQL |
| `spark.sql.adaptive.enabled` | Enables Adaptive Query Execution |
| `spark.sql.autoBroadcastJoinThreshold` | Static broadcast join threshold |
| `spark.executor.memory` | Executor memory allocation |
| `spark.executor.cores` | Executor CPU cores |
| `spark.driver.memory` | Driver memory allocation |

Example:

```python
print(spark.conf.get("spark.sql.shuffle.partitions"))

print(spark.conf.get("spark.sql.adaptive.enabled"))
```

**Important:** Defaults and effective values can differ across Spark versions and managed platforms. Check the actual configuration rather than assuming a particular value.

---

## 12. Common Performance Problems and How to Debug Them

| Problem | What to check in Spark UI | Possible solution |
|---|---|---|
| Slow job | Jobs and Stages tabs | Identify the slow stage |
| Data skew | Uneven task duration and shuffle read | Salting, AQE, or separate hot keys |
| Excessive shuffle | Shuffle read/write | Reduce data movement |
| Memory pressure | Spill and executor metrics | Reduce data size or investigate resources |
| Too many small tasks | Number of tasks and task duration | Review partition count |
| Executor failure | Executors and logs | Investigate memory, node, or container failures |
| Slow join | SQL plan and shuffle metrics | Review join strategy |
| Slow file reading | Scan metrics and input files | Use column pruning, partition pruning, and suitable formats |
| Small files | Input file count and scan behavior | Compact files where appropriate |

---

## 13. Practical Debugging Scenario

Imagine a PySpark job that reads orders, joins customers, aggregates revenue, and writes the result.

The job normally takes 2 minutes but now takes 15 minutes.

### Step 1: Check Jobs tab

Identify which job is taking the most time.

### Step 2: Check Stages tab

Find the stage with the longest duration.

### Step 3: Check Tasks

Compare task durations.

If one task takes several minutes while the others finish quickly, investigate skew or executor-level issues.

### Step 4: Check Shuffle metrics

If shuffle read and write are unusually high, investigate the join or aggregation.

### Step 5: Check spill

If tasks show significant disk spill, investigate memory pressure, large partitions, and skew.

### Step 6: Check SQL execution plan

Inspect:
- Join strategy.
- Exchange operators.
- Scan operations.
- Filters.
- Adaptive execution details.

### Step 7: Make targeted changes

Possible changes:
- Filter earlier.
- Select fewer columns.
- Broadcast a sufficiently small table.
- Handle skew.
- Adjust partitions based on task metrics.
- Compact small files.

### Step 8: Compare results

Run the job again and compare:

- Total execution time.
- Stage duration.
- Shuffle read/write.
- Spill.
- Task distribution.
- Resource utilization.

**Golden rule:** Do not make multiple random changes at once. Make targeted changes and measure their impact.

---

## 14. Common Mistakes During Spark UI Debugging

1. Looking only at total job duration.
2. Ignoring slow individual tasks.
3. Assuming every slow task is caused by data skew.
4. Increasing executor memory without checking spill and logs.
5. Ignoring shuffle read and write.
6. Treating every `Exchange` operator as unnecessary.
7. Changing partition counts without examining task sizes.
8. Assuming more executors always improve performance.
9. Ignoring the SQL execution plan.
10. Making multiple changes without measuring individual impact.

---

## 15. Interview Questions

**Q1. What is Spark UI?**

Spark UI is a web-based monitoring interface used to inspect Spark jobs, stages, tasks, executors, SQL queries, and resource usage.

**Q2. What is the difference between a job, stage, and task?**

- Job: Created when an action triggers execution.
- Stage: A group of tasks separated by execution dependencies, commonly at shuffle boundaries.
- Task: The unit of work that processes a partition within a stage.

**Q3. How do you identify data skew in Spark UI?**

Look for a few tasks with significantly higher execution time or shuffle read compared with other tasks in the same stage.

**Q4. How do you identify excessive shuffle?**

Inspect shuffle read and write metrics in the Stages and SQL tabs, and examine the physical execution plan.

**Q5. What is memory spill?**

It occurs when intermediate execution data cannot remain entirely in execution memory and some of it is moved out of memory.

**Q6. What is disk spill?**

It is intermediate data written to disk when Spark cannot keep the required data in memory.

**Q7. How do you debug a slow join?**

Inspect the physical join strategy, input sizes, shuffle metrics, task distribution, and possible data skew.

**Q8. How do you identify executor problems?**

Check failed or lost executors, task failures, resource usage, executor logs, and memory-related errors.

**Q9. Why is the SQL tab useful?**

It provides query execution plans and runtime information that help identify expensive scans, joins, aggregations, and exchanges.

**Q10. What is the purpose of the Environment tab?**

It displays Spark configuration and environment information that can help diagnose configuration-related issues.

**Q11. How do you prove that a performance optimization worked?**

Compare execution time, task distribution, shuffle metrics, spill, input/output size, and resource utilization before and after optimization.

**Q12. Can you inspect a completed Spark application's UI?**

Yes, if Spark event logging and a Spark History Server are configured. The History Server provides access to historical application details.

---

## 16. Quick Revision

| Spark UI section | Main purpose |
|---|---|
| Jobs | Identify slow jobs |
| Stages | Identify expensive stages |
| Tasks | Find slow tasks and skew |
| Storage | Inspect cached data |
| Executors | Inspect resource usage and failures |
| Environment | Inspect Spark configurations |
| SQL / DataFrame | Inspect query plans and execution metrics |

### Important metrics

| Metric | What it helps identify |
|---|---|
| Task duration | Slow tasks and stragglers |
| Shuffle read | Data fetched after shuffle |
| Shuffle write | Data written for shuffle |
| Memory spill | Execution memory pressure |
| Disk spill | Intermediate disk writes |
| Input size | Data read from sources |
| Executor memory | Resource usage |
| Failed tasks | Execution failures |

### Debugging sequence

1. Jobs → Find the slow job.
2. Stages → Find the slow stage.
3. Tasks → Find the slow tasks.
4. Shuffle → Check data movement.
5. Spill → Check memory pressure.
6. Executors → Check resource problems.
7. SQL → Inspect the execution plan.
8. Optimize → Make a targeted change.
9. Compare → Measure the improvement.

---

## 17. Self-Check

- [ ] Can I explain what Spark UI is?
- [ ] Can I differentiate jobs, stages, and tasks?
- [ ] Can I identify a slow stage?
- [ ] Can I detect possible data skew?
- [ ] Can I interpret shuffle read and write?
- [ ] Can I understand memory and disk spill?
- [ ] Can I inspect executor problems?
- [ ] Can I read important physical plan operators?
- [ ] Can I use Spark UI to investigate a slow join?
- [ ] Can I compare performance before and after optimization?

**Final takeaway:** Spark UI is one of the most important tools for debugging Spark performance. First identify the slow job, then the expensive stage, investigate task-level metrics, inspect the execution plan, and make a targeted optimization.