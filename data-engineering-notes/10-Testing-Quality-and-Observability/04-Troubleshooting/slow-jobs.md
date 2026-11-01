

# Slow Data Jobs: Troubleshooting and Performance Optimization

## 1. Introduction

A slow data job is a job that takes longer than its expected or acceptable execution time to complete.

In data engineering, slow jobs can affect:
- Pipeline completion time.
- Data freshness.
- SLA and SLO targets.
- Downstream pipeline execution.
- Compute costs.
- Business reporting.

A job may be slow because of inefficient transformations, data skew, excessive shuffling, poor partitioning, resource limitations, slow storage, or source-system bottlenecks.

### Example

Suppose a daily orders transformation normally takes 15 minutes but suddenly takes 90 minutes.

The data engineer must investigate:
- Whether the input volume increased.
- Whether Spark is performing expensive shuffles.
- Whether data is skewed.
- Whether the cluster has sufficient resources.
- Whether a source or storage operation is slow.

The objective is not simply to increase compute resources. It is to identify and resolve the actual bottleneck.

---

## 2. Job vs Stage vs Task

Understanding Spark execution is important when troubleshooting slow jobs.

| Component | Meaning |
|---|---|
| Job | A computation triggered by an action |
| Stage | A group of tasks that can execute without crossing a shuffle dependency |
| Task | The unit of work executed on one partition |

### Example

```python
result = (
    orders_df
    .groupBy("customer_id")
    .sum("amount")
)

result.show()
```

When `show()` triggers execution, Spark creates a job.

The aggregation may require a shuffle, resulting in multiple stages.

Each stage contains tasks that process partitions of data.

### Important

- A slow job may contain one or several slow stages.
- A slow stage may contain one or several slow tasks.
- A small number of unusually slow tasks often indicates skew or uneven partition sizes.

---

## 3. Common Causes of Slow Data Jobs

| Cause | Explanation |
|---|---|
| Data skew | Some partitions contain much more data than others |
| Excessive shuffle | Large amounts of data move across executors |
| Too few partitions | Insufficient parallelism |
| Too many small partitions | Scheduling and task overhead |
| Inefficient joins | Expensive join strategies or large data movement |
| Large data volume | More data requires more processing |
| Memory pressure | Garbage collection and spilling |
| Disk I/O | Heavy reads, writes, or spills |
| Poor file layout | Too many small files or ineffective partition pruning |
| Unnecessary caching | Memory pressure and cache overhead |
| Slow source system | Data extraction is delayed |
| Resource limitations | Insufficient CPU, memory, or compute capacity |

---

## 4. Step-by-Step Troubleshooting Process

Follow a structured approach when a production job becomes slow.

### Step 1: Confirm the Performance Regression

Compare the current run with previous successful runs.

Check:
- Total execution duration.
- Input record count.
- Input data size.
- Number of tasks.
- Number of stages.
- Compute configuration.

Example:

```text
Pipeline: orders_pipeline

Previous Duration: 15 minutes
Current Duration:  90 minutes

Input Data:
Previous: 10 GB
Current:  11 GB

Observation:
Execution time increased significantly
without a similar increase in input size.
```

This suggests that a new bottleneck may exist.

### Step 2: Identify the Slow Stage

Open Spark UI and inspect the completed or active jobs.

Check:
- Stage duration.
- Number of tasks.
- Task duration distribution.
- Shuffle read and write.
- Memory and disk spill.
- Failed and retried tasks.

### Step 3: Inspect Slow Tasks

Compare task durations within the same stage.

If most tasks complete quickly but a few take much longer, investigate:
- Data skew.
- Uneven partition sizes.
- Executor problems.
- Long garbage collection.
- Slow external operations.

### Step 4: Inspect the Execution Plan

Use:

```python
df.explain("formatted")
```

Check for:
- Unexpected exchanges or shuffles.
- Sort operations.
- Join strategy.
- Repeated scans.
- Missing partition pruning.
- Unnecessary transformations.

The plan explains how Spark intends to execute the query. Runtime metrics are needed to confirm where time is actually spent.

### Step 5: Check Resource Utilization

Inspect:
- Executor CPU utilization.
- Executor memory usage.
- Garbage collection time.
- Disk spill.
- Network activity.
- Executor failures.

### Step 6: Identify the Root Cause

Classify the bottleneck:

- CPU-bound.
- Memory-bound.
- Disk I/O-bound.
- Network/shuffle-bound.
- Source-bound.
- Sink-bound.
- Scheduling-bound.

### Step 7: Apply an Appropriate Optimization

Choose an optimization based on the actual bottleneck.

### Step 8: Compare Before and After

Measure:
- Execution duration.
- Data volume processed.
- Shuffle size.
- Spill.
- Task distribution.
- Resource utilization.

Optimization should be validated through measurement rather than assumptions.

---

## 5. Troubleshooting Data Skew

Data skew occurs when data is distributed unevenly across partitions.

Some partitions contain significantly more records than others.

### Example

Suppose an orders table contains 10 million records.

| Partition | Records |
|---|---:|
| Partition 1 | 1,000,000 |
| Partition 2 | 1,100,000 |
| Partition 3 | 950,000 |
| Partition 4 | 6,500,000 |
| Partition 5 | 450,000 |

Partition 4 contains much more data than the others.

Its task may take significantly longer, delaying the entire stage.

### How to Identify Skew

In Spark UI:
1. Open the slow stage.
2. Compare task durations.
3. Inspect shuffle read sizes.
4. Compare input records per task.
5. Look for a few unusually large tasks.

### Common Solutions

**1. Broadcast Join**

If one side of a join is small enough, broadcast it to avoid shuffling the large side for the join.

```python
from pyspark.sql.functions import broadcast

result = orders_df.join(
    broadcast(customers_df),
    on="customer_id",
    how="inner"
)
```

Broadcast joins are suitable for supported join types and when the broadcast side safely fits in executor memory.

**2. Salting**

Salting distributes a heavily skewed key across multiple artificial keys.

Conceptual example:

```python
from pyspark.sql import functions as F

salted_orders = orders_df.withColumn(
    "salt",
    (F.rand() * 10).cast("int")
)
```

A complete salted join requires compatible changes to both sides of the join and often a second aggregation or cleanup step.

Random salting alone does not solve a skewed join.

**3. Adaptive Query Execution**

AQE can use runtime statistics to improve certain execution decisions, including handling supported skewed joins.

```python
spark.conf.set(
    "spark.sql.adaptive.enabled",
    "true"
)
```

AQE does not eliminate every type of data skew.

---

## 6. Troubleshooting Excessive Shuffle

A shuffle occurs when Spark redistributes data across partitions, often involving network transfer, serialization, sorting, and disk I/O.

### Common Shuffle-Causing Operations

- `groupBy()`
- `distinct()`
- `dropDuplicates()`
- `repartition()`
- Many joins.
- Window operations requiring redistribution or sorting.

### Example

```python
result = orders_df.groupBy(
    "customer_id"
).sum("amount")
```

Spark may need to redistribute records so that records for the same customer can be aggregated together.

### How to Reduce Shuffle Cost

1. Filter unnecessary records early.
2. Select only required columns.
3. Avoid unnecessary `distinct()`.
4. Avoid repeated repartitioning.
5. Use suitable join strategies.
6. Pre-aggregate data where business logic permits.
7. Avoid unnecessary wide transformations.

### Example: Filter and Project Early

Less efficient:

```python
result = (
    orders_df
    .join(customers_df, "customer_id")
    .filter("amount > 1000")
    .select("customer_id", "amount")
)
```

More explicit:

```python
filtered_orders = (
    orders_df
    .filter(F.col("amount") > 1000)
    .select("customer_id", "amount")
)

selected_customers = customers_df.select(
    "customer_id",
    "customer_name"
)

result = filtered_orders.join(
    selected_customers,
    on="customer_id",
    how="inner"
)
```

The second version reduces the data and columns presented to the join.

The Catalyst optimizer may already push down equivalent filters and projections, so verify the physical plan and runtime metrics before assuming the rewrite improves performance.

---

## 7. Troubleshooting Slow Joins

Joins can become expensive when large datasets must be redistributed or sorted.

### Common Join Strategies

| Join Strategy | Description |
|---|---|
| Broadcast Hash Join | Broadcasts a small relation to executors |
| Sort-Merge Join | Sorts and merges both sides using join keys |
| Shuffle Hash Join | Shuffles data and builds hash tables for partitions |
| Broadcast Nested Loop Join | Uses nested-loop matching with a broadcast side for supported cases |

The selected strategy depends on join conditions, data size estimates, configuration, and supported join types.

### Broadcast Join Example

```python
from pyspark.sql.functions import broadcast

result = large_orders_df.join(
    broadcast(small_customers_df),
    "customer_id"
)
```

### When to Use Broadcast Join

- One side is relatively small.
- The broadcast relation can fit safely in executor memory.
- The join type and condition support the strategy.

### Common Mistakes

- Broadcasting a large table.
- Ignoring inaccurate table statistics.
- Assuming every join can use Broadcast Hash Join.
- Joining data at incompatible grains.
- Ignoring duplicate join keys.

A broadcast hint influences planning but does not guarantee that a particular strategy will be used.

---

## 8. Troubleshooting Partitioning Problems

Partitioning affects parallelism, task overhead, and data distribution.

### 8.1 Too Few Partitions

Symptoms:
- Few tasks run.
- Available executors remain underutilized.
- Each task processes a large amount of data.

Possible solution:

```python
df_repartitioned = df.repartition(200)
```

This increases the number of partitions and redistributes data.

### 8.2 Too Many Partitions

Symptoms:
- Many very small tasks.
- Scheduling overhead increases.
- Excessive small output files.

Possible solution:

```python
df_reduced = df.coalesce(50)
```

`coalesce()` reduces the number of partitions, generally without a full shuffle.

It may produce uneven partition sizes and does not guarantee an exact number of output files.

### 8.3 Repartition by a Column

```python
df_repartitioned = df.repartition(
    200,
    "customer_id"
)
```

This redistributes records by the specified column.

**Important:** Repartitioning by a highly skewed key can preserve or create uneven partition sizes. Increasing the partition count alone does not solve skew.

---

## 9. Troubleshooting Memory and Disk Spill

Spark uses memory for execution and caching.

When memory is insufficient for certain operations, Spark may spill intermediate data to disk.

### Symptoms

- High garbage collection time.
- Frequent memory-related warnings.
- High disk spill.
- Slow sort, aggregation, or join stages.
- Executor failures.

### Common Causes

- Large joins.
- Excessive caching.
- Large aggregations.
- Data skew.
- Oversized broadcast relations.
- Too much data collected to the driver.

### Solutions

1. Avoid unnecessary `collect()`.
2. Remove unused cached DataFrames.
3. Review join strategies.
4. Reduce unnecessary columns.
5. Handle skew.
6. Review executor memory and configuration.
7. Check whether more suitable partitioning can reduce per-task memory requirements.

### Example: Avoid Collecting Large Data

Risky:

```python
all_rows = df.collect()
```

`collect()` transfers all records to the driver and can cause driver out-of-memory errors.

Better:

```python
df.show(20)
```

Or use distributed writes and aggregations when processing large datasets.

### Important

Memory spill and disk spill metrics can vary across Spark UI versions and operations. Inspect the relevant stage metrics rather than treating every spill metric as identical.

---

## 10. Troubleshooting Small Files and Storage Performance

A large number of small files can increase:
- File listing overhead.
- Query planning overhead.
- Metadata operations.
- Task scheduling overhead.
- Storage read overhead.

### Example

A dataset contains 10 GB of data.

Instead of a manageable number of files, it contains hundreds of thousands of tiny files.

Reading the dataset may become inefficient.

### Solutions

**1. Compact Files**

For a Delta table:

```sql
OPTIMIZE orders_delta;
```

This compacts eligible small files into larger files.

**2. Use Suitable Partitioning**

Partition by columns commonly used in selective filters, with appropriate cardinality and data volume.

Avoid high-cardinality partition columns.

**3. Avoid Excessive Output Partitions**

Control the number of output partitions based on workload and file-size observations.

### Important

There is no universal ideal file size. The appropriate size depends on workload, storage system, compute, and query patterns.

`OPTIMIZE` consumes compute resources and should be evaluated against its performance and cost benefits.

---

## 11. Troubleshooting Slow Data Sources and Sinks

Not every slow job is caused by Spark transformations.

The source or target system may be the bottleneck.

### Source-Side Problems

- Slow database queries.
- API rate limits.
- Network latency.
- Large unfiltered extracts.
- Source-system workload limits.
- Inefficient pagination.

### Sink-Side Problems

- Slow target database writes.
- Storage throttling.
- Excessive small-file creation.
- Concurrent write conflicts.
- Expensive target indexing or constraints.

### Investigation

Check:
1. Source query duration.
2. Data extraction throughput.
3. Network transfer time.
4. Transformation duration.
5. Target write duration.
6. Target-system resource usage.

Separate extraction, transformation, and loading durations where possible.

### Optimization

- Push down selective filters where supported.
- Extract incrementally.
- Select only required columns.
- Tune source query performance.
- Use suitable write parallelism.
- Avoid excessive concurrent writes.
- Optimize target file layout.

---

## 12. Using Spark UI to Troubleshoot Slow Jobs

Spark UI is one of the most useful tools for diagnosing Spark performance issues.

### Important Tabs

| Tab | What to Inspect |
|---|---|
| Jobs | Job status and duration |
| Stages | Stage duration, task metrics, shuffle |
| Storage | Cached datasets and memory use |
| Executors | Executor activity, memory, failures |
| SQL/DataFrame | Query execution details and plans |
| Environment | Spark configuration |

Exact tabs and displayed metrics can vary by Spark version and platform.

### Important Metrics

| Metric | What It Indicates |
|---|---|
| Input Size / Records | Data processed by tasks |
| Shuffle Read | Data read from shuffle output |
| Shuffle Write | Data written for downstream stages |
| Executor Run Time | Time spent executing tasks |
| GC Time | Time spent in garbage collection |
| Memory Spill | Intermediate data spilled from memory |
| Disk Spill | Intermediate data written to disk |
| Task Duration | Time taken by individual tasks |

### How to Investigate

1. Open the failed or slow job.
2. Identify its slowest stage.
3. Compare task duration distribution.
4. Inspect shuffle and spill metrics.
5. Check executor health.
6. Compare the execution plan.
7. Determine the bottleneck.
8. Apply one targeted optimization.
9. Compare the next run.

---

## 13. Adaptive Query Execution (AQE)

Adaptive Query Execution allows Spark SQL to adjust parts of the physical execution plan using runtime statistics.

It can help with:
- Coalescing small shuffle partitions.
- Supported skewed join handling.
- Changing certain join strategies.

Example configuration:

```python
spark.conf.set(
    "spark.sql.adaptive.enabled",
    "true"
)

spark.conf.set(
    "spark.sql.adaptive.coalescePartitions.enabled",
    "true"
)
```

### Important

- AQE is enabled by default in standard Spark SQL configurations from Spark 3.2, but platform settings may differ.
- AQE uses runtime statistics.
- It cannot fix every inefficient query.
- It does not eliminate the need for good partitioning and data modeling.
- Verify the final adaptive plan and actual runtime metrics.

---

## 14. Troubleshooting Slow Jobs in Databricks

In Azure Databricks, investigate both Spark execution and platform-level resources.

### What to Check

- Job run duration.
- Task-level execution time.
- Cluster or serverless compute configuration.
- Driver and executor logs.
- Spark UI.
- Input/output data size.
- Shuffle and spill metrics.
- Photon usage where applicable.
- Storage access and network delays.

### Common Databricks Performance Problems

| Problem | Investigation |
|---|---|
| Slow notebook task | Inspect execution and query plans |
| Long-running Spark stage | Check skew and shuffle |
| Driver failure | Inspect driver memory and logs |
| Executor loss | Inspect resource pressure and infrastructure events |
| Slow Delta reads | Check pruning, file layout, and statistics |
| Slow Delta writes | Check file count and concurrent writers |
| High compute cost | Review workload duration and compute selection |

### Delta Table Optimization

```sql
OPTIMIZE orders_delta;
```

This compacts files.

For supported workloads, data skipping can improve query performance when file statistics allow irrelevant files to be skipped.

Z-Ordering can improve data skipping for suitable access patterns:

```sql
OPTIMIZE orders_delta
ZORDER BY (customer_id);
```

Z-Ordering and OPTIMIZE are related but different:
- OPTIMIZE compacts files.
- Z-ORDER reorganizes data to improve locality for data skipping.

Liquid clustering is another table-layout capability available in supported Databricks environments. Its suitability and availability depend on the runtime and table configuration.

---

## 15. Performance Troubleshooting Checklist

Before changing code or increasing compute, review this checklist.

### Data

- [ ] Has input volume changed?
- [ ] Are there unusually large partitions?
- [ ] Is data skew present?
- [ ] Are unnecessary columns being processed?
- [ ] Can filters be applied earlier?
- [ ] Are source files too small or numerous?

### Spark Execution

- [ ] Which stage is slow?
- [ ] Are a few tasks much slower than others?
- [ ] Is shuffle volume unusually high?
- [ ] Is disk spill occurring?
- [ ] Is garbage collection excessive?
- [ ] Is the join strategy appropriate?
- [ ] Is AQE enabled where suitable?

### Infrastructure

- [ ] Is CPU utilization high or low?
- [ ] Is memory sufficient?
- [ ] Are executors failing?
- [ ] Is storage throttling occurring?
- [ ] Is the source or target system slow?

### Recovery and Validation

- [ ] Has the root cause been identified?
- [ ] Is the optimization safe?
- [ ] Is the output still correct?
- [ ] Has the execution time improved?
- [ ] Have SLA/SLO targets been restored?

---

## 16. Practical Interview Scenario

**Question: A PySpark job that normally takes 20 minutes now takes 90 minutes. How would you troubleshoot it?**

### Answer

I would first compare the current execution with previous successful runs.

I would check input volume, job duration, number of stages, and compute configuration.

Then I would open Spark UI and identify the slowest stage.

I would inspect task duration distribution, shuffle read and write, memory spill, disk spill, and executor health.

If a few tasks are significantly slower than the others, I would investigate data skew.

If shuffle volume is high, I would review joins, aggregations, and repartitioning.

If memory pressure is high, I would inspect caching, large joins, and driver-side collection.

I would also check whether the source extraction or target write is causing the delay.

After identifying the bottleneck, I would apply a targeted optimization and compare the new execution time and resource metrics.

Finally, I would validate the output to ensure that performance improvements did not change the business results.

---

## 17. Common Interview Questions and Answers

### Q1. What is a slow data job?

A job that takes longer than its expected or acceptable execution time.

### Q2. What are the common causes of slow Spark jobs?

Data skew, excessive shuffling, inefficient joins, poor partitioning, memory pressure, small files, and insufficient resources.

### Q3. How do you identify a slow stage?

Use Spark UI to inspect stage duration, task duration, shuffle metrics, and spill.

### Q4. What is data skew?

Uneven distribution of data across partitions, causing some tasks to process much more data than others.

### Q5. How do you identify data skew?

Compare task duration, input size, and shuffle read across tasks in the same stage.

### Q6. How do you handle data skew?

Use suitable strategies such as broadcast joins, salting, and AQE where applicable.

### Q7. What is shuffle?

Shuffle is the redistribution of data across partitions, generally involving network and possibly disk I/O.

### Q8. Which operations commonly cause shuffle?

Joins, `groupBy`, `distinct`, `repartition`, and certain window operations.

### Q9. What is the difference between `repartition()` and `coalesce()`?

`repartition()` redistributes data and generally performs a shuffle. `coalesce()` reduces partitions with less data movement in typical use.

### Q10. What is a broadcast join?

A join strategy that distributes a small relation to executors so the large relation does not need to be shuffled for that join.

### Q11. What is memory spill?

Intermediate execution data that cannot remain in memory is moved to disk or another spill location.

### Q12. What is the purpose of AQE?

AQE uses runtime statistics to adapt parts of Spark SQL execution, including supported shuffle partition coalescing and skewed join handling.

### Q13. How can small files affect performance?

They increase file listing, metadata, planning, and task scheduling overhead.

### Q14. What is the difference between OPTIMIZE and Z-ORDER?

OPTIMIZE compacts files. Z-ORDER reorganizes data to improve data skipping for suitable queries.

### Q15. Should you always increase cluster size when a job is slow?

No. First identify the bottleneck. Increasing resources may not solve skew, inefficient joins, source delays, or unnecessary shuffling.

### Q16. What is the difference between CPU-bound and I/O-bound workloads?

CPU-bound workloads spend most of their time performing computation. I/O-bound workloads spend significant time waiting for storage, network, or external systems.

### Q17. Why can a job be slow even when CPU utilization is low?

It may be waiting for I/O, network transfer, source systems, synchronization, or uneven task completion.

### Q18. How do you verify that an optimization worked?

Compare execution duration, input volume, shuffle, spill, task distribution, resource usage, and output correctness.

---

## 18. Quick Revision

| Concept | Key Meaning |
|---|---|
| Slow Job | Job takes longer than expected |
| Job | Computation triggered by an action |
| Stage | Tasks separated by shuffle dependencies |
| Task | Work performed on one partition |
| Data Skew | Uneven partition distribution |
| Shuffle | Data redistribution across partitions |
| Broadcast Join | Distributes a small relation to executors |
| Salting | Splits a skewed key into multiple artificial keys |
| AQE | Runtime adaptation of query execution |
| Memory Spill | Intermediate data moved out of memory |
| Small Files | Excessive small files increase overhead |
| OPTIMIZE | Compacts Delta files |
| Z-ORDER | Reorganizes data for data skipping |

### Remember These Five Points

1. Identify the slow stage before changing the code.
2. Compare task durations to detect skew.
3. Investigate shuffle, spill, and join strategies.
4. Check source, sink, and infrastructure bottlenecks.
5. Measure performance and validate output after optimization.

---

## 19. Self-Assessment

Try answering these questions without looking at the notes.

1. What is a slow data job?
2. What is the difference between a job, stage, and task?
3. What are the common causes of slow Spark jobs?
4. How do you investigate a slow job using Spark UI?
5. What is data skew, and how do you identify it?
6. How does a broadcast join improve performance?
7. What is salting?
8. Which Spark operations commonly cause shuffle?
9. How do you reduce unnecessary shuffle?
10. What is the difference between `repartition()` and `coalesce()`?
11. What causes memory and disk spill?
12. How can small files affect query performance?
13. What is Adaptive Query Execution?
14. What is the difference between OPTIMIZE and Z-ORDER?
15. Why is increasing cluster size not always the right solution?
16. How do you identify source-side and sink-side bottlenecks?
17. What metrics would you compare before and after optimization?
18. Design a troubleshooting approach for a PySpark job that suddenly becomes five times slower.

**Interview readiness:** Be able to explain how you use Spark UI to locate a bottleneck, diagnose skew and shuffle, optimize joins and partitioning, and prove that the optimized job produces correct results.

