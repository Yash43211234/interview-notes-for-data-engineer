# Spark Optimization – Interview Questions and Answers

## 1. Introduction to Spark Optimization

### 1.1 What is Spark Optimization?

Spark optimization is the process of improving the performance, resource utilization, and cost efficiency of Spark applications.

The main goals are to:
- Reduce execution time.
- Reduce unnecessary data movement.
- Improve parallel processing.
- Avoid memory and disk pressure.
- Handle data skew.
- Reduce unnecessary file operations.
- Use cluster resources efficiently.

### 1.2 Why is Spark Optimization Important?

A Spark job can be slow even when it has enough CPU and memory.

Common reasons include:
- Large shuffle operations.
- Data skew.
- Too many small files.
- Poor join strategies.
- Unnecessary columns and rows.
- Excessive partitions.
- Inefficient transformations.
- Repeated computation.
- Driver memory pressure.

**Important:** Do not start optimization by randomly increasing executor memory or the number of workers. First identify the actual bottleneck.

---

## 2. Understand Spark Execution Before Optimization

### 2.1 What Happens When a Spark Job Runs?

A simplified execution flow:

1. The application creates transformations.
2. An action triggers execution.
3. Spark builds and optimizes the query plan.
4. The physical plan determines execution strategies.
5. Spark divides work into stages.
6. Tasks execute on partitions.
7. Results are returned or written to storage.

### 2.2 Job, Stage, and Task

| Concept | Meaning |
|---|---|
| Job | Work initiated by an action |
| Stage | Group of tasks separated by shuffle dependencies |
| Task | Work performed on a partition |

Understanding these concepts helps identify where a job is spending time.

### 2.3 What is a Shuffle?

A shuffle redistributes data across partitions.

Common operations that can cause shuffles:
- `groupBy()`
- `distinct()`
- `repartition()`
- `orderBy()`
- Many joins.
- Some window operations.

Shuffle may involve:
- Network transfer.
- Serialization.
- Sorting or hashing.
- Disk I/O.
- Memory usage.

Shuffles are not always bad. Many analytical operations require them. The goal is to avoid unnecessary shuffle and make required shuffle efficient.

---

## 3. Use the Spark UI for Performance Analysis

### 3.1 What is Spark UI?

Spark UI is the monitoring interface used to inspect Spark application execution.

It helps analyze:
- Jobs.
- Stages.
- Tasks.
- Executors.
- Storage.
- SQL queries.
- Environment configuration.

### 3.2 Important Spark UI Sections

| Section | What to Check |
|---|---|
| Jobs | Job duration and status |
| Stages | Shuffle, task duration, failures |
| Tasks | Slow tasks and uneven execution |
| Executors | CPU, memory, task counts, failures |
| Storage | Cached DataFrames and memory usage |
| SQL | Query plans and execution details |
| Environment | Spark configuration |

### 3.3 How Do You Identify a Slow Spark Job?

Follow this approach:

1. Find the slow job in the Jobs tab.
2. Open the slow stage.
3. Compare task durations.
4. Check whether a few tasks are much slower than others.
5. Inspect shuffle read and write.
6. Check memory and disk spill metrics.
7. Review executor failures and garbage collection.
8. Inspect the SQL execution plan.

### 3.4 What is Data Spill?

Data spill occurs when intermediate data cannot be kept entirely in memory and Spark writes some of it to disk during execution.

It can happen during operations such as:
- Sorting.
- Aggregations.
- Joins.
- Shuffle processing.

Spill can increase execution time because disk I/O is generally slower than memory access.

### 3.5 What is the Difference Between Memory Spill and Disk Spill?

Spark UI exposes different memory and disk spill metrics.

- Memory spill generally refers to the amount of data spilled from in-memory structures during execution.
- Disk spill refers to data written to disk.

Interpret these metrics in the context of the operation and Spark version. They are not interchangeable measurements.

---

## 4. Reduce Unnecessary Data Processing

### 4.1 Filter Data Early

Apply filters as early as practical to reduce the number of rows processed downstream.

```python
from pyspark.sql.functions import col

filtered_df = df.filter(
    col("status") == "ACTIVE"
)
```

Benefits:
- Less data processed.
- Smaller shuffle volume.
- Lower memory requirements.
- Faster downstream operations.

Spark may push filters toward the data source when supported.

### 4.2 Select Only Required Columns

Avoid reading or carrying unnecessary columns.

```python
selected_df = df.select(
    "customer_id",
    "customer_name",
    "amount"
)
```

This can reduce:
- I/O.
- Memory usage.
- Network transfer.
- Shuffle volume.

### 4.3 What is Projection Pruning?

Projection pruning means reading or processing only the columns required by the query.

It is especially useful with columnar formats such as Parquet.

### 4.4 What is Predicate Pushdown?

Predicate pushdown means applying a filter close to the data source when the source supports it.

For example:

```python
df.filter(col("country") == "India")
```

The filter may be pushed into the source scan, reducing the amount of data read.

Predicate pushdown is not guaranteed for every data source or expression.

### 4.5 Projection Pruning vs Predicate Pushdown

| Projection Pruning | Predicate Pushdown |
|---|---|
| Reduces columns read | Reduces rows read |
| Focuses on required columns | Focuses on filter conditions |
| Example: select three columns | Example: filter one country |

### 4.6 Why Should We Avoid `SELECT *`?

`SELECT *` can read unnecessary columns and increase I/O and memory usage.

Prefer selecting only the columns required by the workload.

---

## 5. Partition Optimization

### 5.1 What is a Partition?

A partition is a logical division of a distributed dataset.

Spark processes partitions in parallel through tasks.

Partition count affects:
- Parallelism.
- Task scheduling.
- Data distribution.
- Shuffle performance.
- Output file count.

### 5.2 What Happens When There Are Too Few Partitions?

Too few partitions can cause:
- Low parallelism.
- Underutilized executors.
- Large tasks.
- Memory pressure.
- Long-running tasks.

### 5.3 What Happens When There Are Too Many Partitions?

Too many partitions can cause:
- Excessive task scheduling overhead.
- Many small tasks.
- Too many small output files.
- Increased metadata overhead.

### 5.4 What is `repartition()`?

`repartition()` changes the number of partitions and can redistribute data across them.

```python
df_repartitioned = df.repartition(20)
```

Repartition by a column:

```python
df_repartitioned = df.repartition(
    20,
    "customer_id"
)
```

It generally causes a shuffle.

Use it when redistribution or additional parallelism is needed.

### 5.5 What is `coalesce()`?

`coalesce()` generally reduces the number of partitions without performing a full shuffle.

```python
df_coalesced = df.coalesce(5)
```

It is useful when reducing partitions before writing output.

However, it may produce uneven partition sizes.

### 5.6 Repartition vs Coalesce

| Repartition | Coalesce |
|---|---|
| Can increase or decrease partitions | Normally reduces partitions |
| Generally performs a shuffle | Usually avoids full shuffle |
| Can redistribute data | May preserve uneven distribution |
| Useful for balancing data | Useful for reducing partition count |

Neither operation guarantees an exact number of output files in every write scenario.

### 5.7 What is Partition Pruning?

Partition pruning skips storage partitions that cannot match a query filter.

Suppose a table is stored by year:

```text
transactions/
|
|-- year=2024/
|-- year=2025/
|-- year=2026/
```

Query:

```python
df.filter(col("year") == 2026)
```

Spark may read only the relevant year partition.

### 5.8 What is the Difference Between Spark Partitions and Storage Partitions?

**Spark partitions:**
- Logical divisions of distributed data.
- Determine task-level processing parallelism.

**Storage partitions:**
- Directory-level organization of files by partition column values.
- Can support partition pruning.

They are related but not the same thing.

---

## 6. Shuffle Optimization

### 6.1 Why is Shuffle Expensive?

Shuffle can require:
- Network communication.
- Serialization and deserialization.
- Disk reads and writes.
- Sorting or hashing.
- Additional memory.

It can become a major bottleneck in large Spark jobs.

### 6.2 How Can You Reduce Shuffle?

1. Filter unnecessary rows early.
2. Select only required columns.
3. Avoid unnecessary `distinct()`.
4. Avoid repeated repartitioning.
5. Use suitable join strategies.
6. Broadcast small tables when appropriate.
7. Aggregate data at the correct grain.
8. Use appropriate partitioning.
9. Inspect the physical execution plan.

### 6.3 What is Partial Aggregation?

Partial aggregation performs some aggregation locally before data is shuffled for final aggregation.

For example:

```python
result = (
    df.groupBy("department")
      .sum("salary")
)
```

Spark may perform partial aggregation before the shuffle and then combine intermediate results.

This can reduce the volume of data transferred.

The exact physical strategy depends on the query plan and data source.

### 6.4 Why Should We Avoid Unnecessary `distinct()`?

`distinct()` removes duplicate rows and commonly requires data redistribution.

If the business requirement is to select the latest record per customer, use a deterministic window-based deduplication strategy instead of blindly applying `distinct()`.

---

## 7. Join Optimization

### 7.1 Why Can Joins Be Expensive?

Joins may require data redistribution so matching keys can be processed together.

Large joins can cause:
- Shuffle.
- Network traffic.
- Memory pressure.
- Data skew.
- Long-running tasks.

### 7.2 What is a Broadcast Join?

A broadcast join sends a sufficiently small relation to executors so that the larger relation can be joined locally on each executor.

Example:

```python
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    on="customer_id",
    how="inner"
)
```

Benefits:
- Can avoid shuffling the large side.
- Can improve performance for suitable joins.

Risks:
- Broadcasting a large table can cause executor memory pressure.
- Not suitable for every join type or condition.
- A hint influences planning but does not guarantee a particular physical strategy.

### 7.3 When Should You Use Broadcast Join?

Use it when:
- One side is genuinely small.
- The broadcast data can fit safely in executor memory.
- The join type and condition support the strategy.
- Statistics or testing support the choice.

Do not broadcast a table simply because it has fewer rows. Row width and actual data size matter.

### 7.4 What is Sort-Merge Join?

Sort-Merge Join is a distributed join strategy commonly used for large equi-joins.

At a high level:
1. Data is partitioned by join key.
2. Data is sorted within partitions.
3. Matching keys are merged.

It can involve expensive shuffle and sorting operations.

### 7.5 Broadcast Join vs Sort-Merge Join

| Broadcast Join | Sort-Merge Join |
|---|---|
| Broadcasts a small relation | Redistributes and sorts join data |
| Can avoid shuffle on the large side | Commonly involves shuffle on both sides |
| Useful when one side is small | Useful for large equi-joins |
| Can create memory pressure if broadcast is too large | Can be expensive due to shuffle and sorting |

### 7.6 What is a Shuffle Hash Join?

Shuffle Hash Join redistributes data by join key and builds hash tables for joining within partitions.

It can be useful when partition-level build-side data is sufficiently small.

The selected strategy depends on Spark's planner, statistics, configuration, and supported join conditions.

### 7.7 What is a Join Hint?

A join hint suggests a preferred join strategy.

Example:

```python
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    "customer_id"
)
```

Hints influence the optimizer but should be validated using the physical plan and runtime metrics.

---

## 8. Handling Data Skew

### 8.1 What is Data Skew?

Data skew occurs when data is distributed unevenly across partitions.

Example:

A transaction table contains records for many customers, but one customer has 50% of all transactions.

If partitioning is based on `customer_id`, one partition may receive much more data than others.

### 8.2 Why is Data Skew a Problem?

- Some tasks take much longer.
- Other executors may become idle.
- Memory pressure increases on overloaded tasks.
- Shuffle stages take longer.
- Jobs may fail due to resource exhaustion.

### 8.3 How Do You Identify Data Skew?

Check:
- Uneven task durations.
- Large differences in task input sizes.
- Uneven shuffle read.
- Long-running tasks in one stage.
- Key-frequency distribution.

Spark UI is useful for confirming skew.

### 8.4 What is Key Salting?

Key salting distributes records with a heavily skewed key across multiple artificial keys.

Example:

```python
from pyspark.sql.functions import (
    col,
    concat,
    lit,
    pmod,
    hash
)

salted_df = df.withColumn(
    "salted_key",
    concat(
        col("customer_id"),
        lit("_"),
        pmod(hash(col("transaction_id")), lit(10))
    )
)
```

This creates multiple join keys for the same original customer.

For a join, the other side must be expanded or otherwise transformed consistently so matching records are not lost.

After processing, aggregate or remove the artificial key as required.

### 8.5 What is Adaptive Query Execution?

Adaptive Query Execution (AQE) allows Spark SQL to adjust parts of the physical plan using runtime statistics.

It can:
- Coalesce small shuffle partitions.
- Change certain sort-merge joins to broadcast joins.
- Handle certain skewed join partitions.

AQE is enabled by default in standard Spark SQL configurations since Spark 3.2, but platform distributions and application settings can override defaults.

### 8.6 How Does AQE Help With Data Skew?

AQE can identify skewed shuffle partitions and split them into smaller partitions for supported operations.

This helps reduce the effect of a few unusually large partitions.

AQE is not a universal solution for every skew pattern.

---

## 9. Caching and Persistence

### 9.1 What is Caching in Spark?

Caching stores a computed dataset so that subsequent operations can reuse it rather than recomputing it.

Example:

```python
df_filtered = df.filter(
    col("status") == "ACTIVE"
).cache()

df_filtered.count()

# Reuse df_filtered

df_filtered.unpersist()
```

The action helps materialize the cached data.

### 9.2 When Should You Cache a DataFrame?

Cache when:
- An expensive DataFrame is reused multiple times.
- Recomputing the same transformation is costly.
- The data fits reasonably within available storage resources.
- Measurements show that caching improves performance.

### 9.3 When Should You Avoid Caching?

Avoid caching when:
- Data is used only once.
- The DataFrame is inexpensive to recompute.
- Cached data creates memory pressure.
- The dataset is too large for the available resources.
- Cache management overhead exceeds the benefit.

### 9.4 What is Persistence?

Persistence stores a dataset using a selected storage level.

```python
from pyspark import StorageLevel

df.persist(StorageLevel.MEMORY_AND_DISK)
```

`cache()` is a convenient default persistence method. Exact defaults depend on Spark version and API.

### 9.5 Cache vs Persist

| Cache | Persist |
|---|---|
| Convenient default storage level | Allows explicit storage level |
| Simple reuse | More control over storage behavior |
| Useful for repeated computation | Useful when memory and disk strategy matters |

---

## 10. File Optimization

### 10.1 What is the Small Files Problem?

The small files problem occurs when a dataset contains a very large number of small files.

It increases:
- File listing overhead.
- Metadata operations.
- Query planning overhead.
- Task scheduling overhead.
- Storage-management complexity.

### 10.2 How Can You Handle Small Files?

- Compact files.
- Avoid excessive output partitions.
- Use suitable write partitioning.
- Avoid unnecessary partition columns.
- Use platform-supported optimized write features when available.

### 10.3 What is File Compaction?

File compaction combines smaller files into larger files to reduce overhead.

In Delta Lake on supported Databricks environments:

```sql
OPTIMIZE customers;
```

### 10.4 What is the Difference Between `OPTIMIZE` and `ZORDER`?

| OPTIMIZE | ZORDER |
|---|---|
| Compacts small files | Reorganizes data locality |
| Reduces file overhead | Can improve data skipping |
| Can be used alone | Commonly used with OPTIMIZE |
| Does not inherently create an index | Does not create a traditional index |

Example:

```sql
OPTIMIZE customers
ZORDER BY (customer_id);
```

### 10.5 What is Data Skipping?

Data skipping uses file statistics to avoid reading files that cannot match a query filter.

Its effectiveness depends on:
- Data layout.
- Statistics availability.
- Filter columns.
- Data distribution.

---

## 11. Memory Optimization

### 11.1 Why is Memory Important in Spark?

Spark uses memory for:
- Execution.
- Cached data.
- Shuffle processing.
- Sorting.
- Aggregation.
- Join operations.

Insufficient memory can lead to spilling, garbage collection overhead, or failures.

### 11.2 What is Executor Memory?

Executor memory is the memory available to an executor process.

It supports execution and storage workloads, along with other runtime overheads.

The exact memory layout depends on Spark configuration and deployment mode.

### 11.3 What is Unified Memory Management?

Spark's unified memory management allows execution and storage to share a configurable region of executor JVM heap.

- Execution memory is used for operations such as joins, sorts, and aggregations.
- Storage memory is used for cached or persisted data.

Execution can evict storage memory beyond a protected region. Storage cannot evict execution memory.

### 11.4 What Causes Out-of-Memory Errors?

Possible causes:
- Large partitions.
- Data skew.
- Excessive caching.
- Oversized broadcast tables.
- Large driver-side collections.
- Inefficient joins.
- Insufficient memory for workload size.

### 11.5 How Can You Reduce Memory Pressure?

- Filter unnecessary data.
- Select only required columns.
- Avoid large `collect()` operations.
- Handle data skew.
- Use appropriate partition sizes.
- Avoid unnecessary caching.
- Use suitable join strategies.
- Inspect spill and garbage collection metrics.

### 11.6 Why Should You Avoid `collect()`?

`collect()` transfers all rows from executors to the driver.

If the dataset is large, the driver can run out of memory.

Use distributed operations and aggregations. For small inspections, use `show()` or `take()`.

---

## 12. Adaptive Query Execution (AQE)

### 12.1 What is AQE?

AQE is a Spark SQL optimization feature that uses runtime statistics to revise parts of a physical query plan.

### 12.2 What are the Main AQE Features?

1. Coalescing shuffle partitions.
2. Converting certain sort-merge joins to broadcast joins.
3. Handling certain skewed joins.

### 12.3 How Do You Enable AQE?

```python
spark.conf.set(
    "spark.sql.adaptive.enabled",
    "true"
)
```

Check the setting:

```python
spark.conf.get(
    "spark.sql.adaptive.enabled"
)
```

### 12.4 What is Coalescing Shuffle Partitions?

AQE can combine small shuffle partitions after observing runtime statistics.

This reduces the number of tiny tasks.

### 12.5 What is the Difference Between Static and Adaptive Optimization?

| Static Optimization | Adaptive Optimization |
|---|---|
| Uses information available during planning | Uses runtime statistics |
| Plans before execution | Can revise parts of the plan during execution |
| Cannot use all actual runtime information | Can react to observed data sizes and distributions |

AQE supplements normal query optimization rather than replacing it.

---

## 13. Spark Configuration Tuning

### 13.1 Important Spark Configurations

| Configuration | Purpose |
|---|---|
| `spark.executor.memory` | Executor JVM heap memory |
| `spark.executor.cores` | Executor CPU cores |
| `spark.executor.instances` | Number of executors in supported deployment modes |
| `spark.driver.memory` | Driver JVM heap memory |
| `spark.sql.shuffle.partitions` | Default number of shuffle partitions |
| `spark.sql.adaptive.enabled` | Enables or disables AQE |
| `spark.sql.autoBroadcastJoinThreshold` | Threshold used for automatic broadcast planning |
| `spark.default.parallelism` | Default parallelism for certain RDD operations |

Configuration availability and behavior can depend on the deployment platform.

### 13.2 What is `spark.sql.shuffle.partitions`?

It controls the default number of partitions used by many Spark SQL shuffle operations.

Example:

```python
spark.conf.set(
    "spark.sql.shuffle.partitions",
    "200"
)
```

The correct value depends on:
- Data size.
- Cluster resources.
- Partition size.
- Workload.
- AQE behavior.

There is no universally correct value.

### 13.3 What is `spark.sql.autoBroadcastJoinThreshold`?

It controls the estimated relation-size threshold used by Spark SQL for automatic broadcast join planning.

The planner relies on statistics, and actual memory requirements can differ from estimated sizes.

Do not increase this threshold without understanding executor memory and query behavior.

### 13.4 Should We Always Increase Executor Memory?

No.

More memory does not fix every bottleneck.

First inspect:
- Data skew.
- Shuffle volume.
- Partition sizes.
- Driver memory usage.
- Cache pressure.
- Join strategy.
- Garbage collection.
- Disk spill.

Tune resources based on evidence.

---

## 14. Python and PySpark Optimization

### 14.1 Why Should You Avoid Python UDFs When Possible?

Python UDFs can introduce serialization and Python execution overhead.

Prefer built-in PySpark SQL functions when they can express the required transformation.

Example:

```python
from pyspark.sql.functions import upper

df2 = df.withColumn(
    "customer_name_upper",
    upper("customer_name")
)
```

### 14.2 What is a Python UDF?

A User Defined Function allows custom Python logic to be applied to DataFrame columns.

Example:

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

def normalize_name(value):
    if value is None:
        return None

    return value.strip().title()

normalize_name_udf = udf(
    normalize_name,
    StringType()
)

df2 = df.withColumn(
    "normalized_name",
    normalize_name_udf("customer_name")
)
```

Use a UDF only when built-in functions cannot reasonably express the required logic.

### 14.3 What is a Pandas UDF?

A Pandas UDF uses Apache Arrow to transfer data between Spark and Python in batches.

It can improve performance compared with conventional row-based Python UDFs for suitable operations.

However, performance depends on:
- Function logic.
- Batch size.
- Data volume.
- Memory.
- Serialization.
- Spark and Arrow versions.

### 14.4 Built-in Functions vs Python UDF

| Built-in Functions | Python UDF |
|---|---|
| Spark understands the expression | Custom Python logic |
| Can benefit from SQL optimizations | Can introduce Python serialization overhead |
| Usually preferred | Use when required |
| Often more efficient | Performance depends on implementation |

### 14.5 What is Apache Arrow?

Apache Arrow is a columnar in-memory data format used for efficient data exchange between systems.

In PySpark, Arrow can improve data transfer efficiency for supported Pandas UDF and Pandas conversion operations.

---

## 15. Databricks-Specific Optimization

### 15.1 What is Photon?

Photon is Databricks' vectorized query engine designed to accelerate supported workloads.

It can improve performance for supported SQL and DataFrame operations.

It does not accelerate every operation equally.

### 15.2 What is Databricks Optimized Autoscaling?

Databricks autoscaling adjusts compute resources based on workload demand in supported compute configurations.

Benefits can include:
- Better resource utilization.
- Reduced idle compute.
- Improved workload handling.

Autoscaling behavior and capabilities depend on compute type and platform configuration.

### 15.3 What are Optimized Writes?

Optimized writes help produce more appropriately sized files during supported write operations.

They can reduce small-file problems.

Availability and behavior depend on Databricks Runtime, table type, and configuration.

### 15.4 What is Auto Compaction?

Auto compaction combines small files after supported write operations.

It helps reduce small-file accumulation.

It is not a replacement for understanding table layout and workload-specific maintenance.

### 15.5 What is Liquid Clustering?

Liquid clustering is a Databricks-supported data layout feature for organizing data around selected clustering columns.

It can reduce the need for manually managing traditional partitions and ZORDER in supported workloads.

Availability and behavior depend on the platform and runtime.

---

## 16. Practical Optimization Scenario

**Question:** A PySpark job processes 500 GB of transaction data and takes three hours. How would you reduce its execution time?

### Step 1: Understand the Job

Collect:
- Input size.
- Output size.
- Number of files.
- Number of partitions.
- Cluster configuration.
- Execution duration.

### Step 2: Inspect Spark UI

Check:
- Slow stages.
- Long-running tasks.
- Shuffle read and write.
- Spill.
- Executor failures.
- Uneven task duration.

### Step 3: Reduce Data Read

```python
transactions = (
    spark.read.parquet("/data/transactions/")
    .select(
        "customer_id",
        "transaction_date",
        "amount"
    )
    .filter(
        col("transaction_date") >= "2026-01-01"
    )
)
```

Use filters and column selection early. Confirm whether partition pruning and predicate pushdown are effective.

### Step 4: Optimize Joins

If one relation is small enough:

```python
from pyspark.sql.functions import broadcast

result = transactions.join(
    broadcast(customers),
    "customer_id",
    "inner"
)
```

Otherwise, inspect the join plan and data distribution.

### Step 5: Handle Skew

Check whether a few keys contain most of the records.

Use:
- AQE for supported skew handling.
- Key salting where appropriate.
- Separate processing for unusually heavy keys.

### Step 6: Review Partitions

Check whether the job has:
- Too few large partitions.
- Too many tiny partitions.
- Uneven partition sizes.

Tune repartitioning and shuffle partition settings based on measurements.

### Step 7: Optimize Output

If writing to Delta:
- Avoid excessive output partitions.
- Use optimized writes where supported.
- Compact small files when beneficial.
- Consider partitioning based on query patterns.

### Step 8: Compare Results

Measure:
- Total execution time.
- Shuffle volume.
- Spill.
- Task distribution.
- Output file count and size.
- Compute cost.

**Interview explanation:** I would first use Spark UI and the physical execution plan to identify the bottleneck. Then I would reduce unnecessary data reads, optimize joins, handle skew, tune partitions, and improve file layout. I would measure each change rather than applying every optimization blindly.

---

## 17. Common Spark Optimization Mistakes

1. Increasing executor memory without checking the bottleneck.
2. Broadcasting a table that is too large.
3. Using `repartition()` repeatedly.
4. Caching data that is used only once.
5. Calling `collect()` on large datasets.
6. Using Python UDFs when built-in functions are available.
7. Ignoring data skew.
8. Creating too many storage partitions.
9. Using too many shuffle partitions without measuring.
10. Assuming AQE fixes every performance problem.
11. Running OPTIMIZE without considering its compute cost.
12. Assuming a lower task count always means better performance.
13. Optimizing one stage while ignoring the actual end-to-end bottleneck.
14. Treating all disk spill as proof that executor memory must be increased.

---

## 18. Important Spark Optimization Interview Questions

### Q1. How do you optimize a slow Spark job?

Inspect Spark UI and the execution plan, identify the bottleneck, then optimize data reading, joins, shuffle, partitions, skew, and file layout.

### Q2. What is the first thing you check when a Spark job is slow?

I check the Spark UI to identify slow jobs, stages, and tasks. Then I inspect shuffle, spill, task distribution, and executor metrics.

### Q3. What is the difference between repartition and coalesce?

`repartition()` generally shuffles data and can increase or decrease partitions. `coalesce()` generally reduces partitions without a full shuffle.

### Q4. How can you reduce shuffle?

Filter early, select required columns, avoid unnecessary wide operations, use suitable join strategies, and avoid repeated repartitioning.

### Q5. How do you optimize a join between a large and small DataFrame?

Consider a broadcast join if the smaller DataFrame fits safely in executor memory and the join supports that strategy.

### Q6. What is data skew?

Data skew is an uneven distribution of data across partitions, causing some tasks to process much more work than others.

### Q7. How do you handle data skew?

Use Spark UI to confirm skew, then consider AQE, key salting, separate processing for heavy keys, or a better distribution strategy.

### Q8. What is AQE?

AQE uses runtime statistics to revise parts of the Spark SQL physical plan.

### Q9. What is caching?

Caching stores computed data for reuse and avoids recomputing expensive transformations.

### Q10. When should you avoid caching?

Avoid it when data is used only once, is inexpensive to recompute, or creates memory pressure.

### Q11. What is predicate pushdown?

It applies filters close to the data source when supported, potentially reducing data read.

### Q12. What is projection pruning?

It reads only the columns required by a query.

### Q13. What is the small files problem?

Too many small files increase metadata, file listing, planning, and task scheduling overhead.

### Q14. How do you optimize Delta Lake tables?

Use suitable partitioning or clustering, compact small files with OPTIMIZE, consider ZORDER or liquid clustering where appropriate, and avoid unnecessary data rewrites.

### Q15. Why should Python UDFs be avoided when possible?

They can introduce serialization and Python execution overhead. Built-in Spark SQL functions are generally easier for Spark to optimize.

### Q16. Does increasing the number of executors always improve performance?

No. Performance depends on parallelism, data distribution, shuffle, memory, CPU, and the workload's bottleneck.

### Q17. What is the difference between memory spill and disk spill?

They are distinct Spark execution metrics related to spilling intermediate data. Interpret them using the relevant stage and task metrics.

### Q18. What is the difference between partition pruning and predicate pushdown?

Partition pruning skips irrelevant storage partitions. Predicate pushdown applies filters near the data source when supported.

---

## 19. Quick Revision

| Concept | Key Point |
|---|---|
| Spark Optimization | Improve performance and resource usage |
| Spark UI | Identify execution bottlenecks |
| Shuffle | Data redistribution across partitions |
| Data Spill | Intermediate data written to disk |
| Filter Early | Reduce unnecessary rows |
| Projection Pruning | Read required columns |
| Predicate Pushdown | Apply filters near source |
| Repartition | Redistribute partitions |
| Coalesce | Usually reduce partitions without full shuffle |
| Partition Pruning | Skip irrelevant storage partitions |
| Broadcast Join | Send small relation to executors |
| Sort-Merge Join | Distributed sort and merge |
| Data Skew | Uneven partition workload |
| Key Salting | Distribute heavy keys |
| AQE | Optimize using runtime statistics |
| Cache | Reuse computed data |
| Persist | Choose a storage level |
| OPTIMIZE | Compact Delta files |
| ZORDER | Improve data locality |
| Data Skipping | Avoid reading irrelevant files |
| UDF | Custom user-defined function |
| Pandas UDF | Vectorized Python processing using Arrow |
| Photon | Databricks vectorized execution engine |
| Small Files | Too many small files increase overhead |

---

## 20. Self-Assessment

Before a Data Engineer interview, make sure you can:

- [ ] Explain how Spark executes a job.
- [ ] Read the Spark UI and identify slow stages.
- [ ] Explain shuffle and data spill.
- [ ] Explain narrow and wide transformations.
- [ ] Optimize data reading with filters and column selection.
- [ ] Explain predicate pushdown and projection pruning.
- [ ] Explain repartition and coalesce.
- [ ] Differentiate Spark partitions and storage partitions.
- [ ] Explain partition pruning.
- [ ] Choose an appropriate join strategy.
- [ ] Explain broadcast and sort-merge joins.
- [ ] Identify and handle data skew.
- [ ] Explain key salting.
- [ ] Explain AQE.
- [ ] Know when to cache and persist.
- [ ] Explain the small files problem.
- [ ] Explain memory and spill metrics.
- [ ] Compare built-in functions, Python UDFs, and Pandas UDFs.
- [ ] Explain Delta OPTIMIZE and ZORDER.
- [ ] Describe a systematic approach to optimizing a slow Spark job.

**Preparation priority:** First master Spark UI, shuffle, partitions, joins, data skew, and AQE. Then focus on caching, memory, file optimization, and Databricks-specific performance features.