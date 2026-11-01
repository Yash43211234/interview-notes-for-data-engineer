# Partitioning and Parallelism in Distributed Systems

## 1. Introduction

**Partitioning** divides a large dataset into smaller pieces so they can be stored or processed independently.

**Parallelism** means performing multiple tasks at the same time to complete work faster.

In data engineering, partitioning and parallelism help distributed processing engines such as Apache Spark process large datasets across multiple CPU cores and machines.

### Simple Example

Suppose a company needs to process 100 GB of sales data.

Processing the entire dataset sequentially on one machine may be slow. Instead, Spark can divide the data into partitions and process multiple partitions concurrently.

```text
             100 GB Sales Data
                     |
                     v
                 Partitioning
                     |
          +----------+----------+
          |          |          |
          v          v          v
       Part 1     Part 2     Part 3
          |          |          |
          v          v          v
       Task 1     Task 2     Task 3
          |          |          |
          +----------+----------+
                     |
                     v
               Combined Result
```

This is a simplified illustration. A real dataset may have many more partitions, and the number of concurrently running tasks depends on available resources and execution constraints.

**Key idea:** Partitioning divides the work; parallelism allows parts of that work to run concurrently.

---

## 2. What Is Partitioning?

Partitioning divides data into smaller logical or physical units.

In Spark, a DataFrame is distributed across partitions. Each partition contains a subset of the data and can generally be processed by a task.

For example, a sales dataset may contain 10 million records.

Instead of processing all records as one unit, Spark may distribute them across 100 partitions.

```text
Sales DataFrame
       |
       v
+------+------+------+------+------+
| P1   | P2   | P3   | ...  | P100 |
+------+------+------+------+------+
```

Each partition can be processed independently for operations that do not require coordination across partitions.

Some operations, such as aggregations and joins, may require data to be redistributed across partitions.

### Why Partitioning Is Useful

- Distributes data across available resources.
- Enables parallel processing.
- Helps scale workloads.
- Can reduce processing time.
- Can improve data locality for suitable operations.
- Supports efficient distributed storage and querying when the layout matches query patterns.

Partitioning does not automatically make every operation faster. Poor partitioning can introduce overhead, data skew, or excessive small tasks.

---

## 3. What Is Parallelism?

Parallelism is the ability to execute multiple units of work simultaneously.

In Spark, different tasks can run concurrently on available executor cores.

### Example

Suppose a Spark application has four available task slots and eight independent tasks ready to run.

- Four tasks may run in the first wave.
- The remaining four may run after resources become available.

```text
Available Task Slots: 4

Wave 1:  [Task 1] [Task 2] [Task 3] [Task 4]
             Run concurrently

Wave 2:  [Task 5] [Task 6] [Task 7] [Task 8]
             Run concurrently
```

This assumes all four slots remain available and the tasks are eligible to run.

### Parallelism Depends On

- Number of available CPU cores.
- Number of partitions and runnable tasks.
- Executor resources.
- Memory availability.
- Data skew.
- Shuffle and network overhead.
- Dependencies between stages.
- Resource scheduling and cluster configuration.

Having more partitions does not guarantee proportionally more useful parallelism. The cluster must have enough resources to execute the tasks efficiently.

---

## 4. Partitioning vs Parallelism

| Feature | Partitioning | Parallelism |
|---|---|---|
| Meaning | Divides data into smaller units | Executes work concurrently |
| Main purpose | Organize and distribute data | Improve processing throughput |
| Example | Divide a DataFrame into 100 partitions | Run 8 tasks at the same time |
| Depends on | Data layout and partitioning strategy | Available resources and runnable tasks |
| Main risk | Skew or too many small partitions | Resource contention or scheduling overhead |

### Interview Point

Partitioning creates opportunities for parallel execution, but actual parallelism depends on available resources and task dependencies.

A dataset can have 100 partitions while only eight tasks run concurrently if the application has eight available task slots.

---

## 5. Types of Partitioning

The term *partitioning* can refer to several different concepts. In interviews, clarify which one you mean.

### 5.1 Spark Execution Partitioning

This determines how data is divided across Spark partitions during computation.

Example:

```python
df = spark.read.parquet("/data/sales")

print(df.rdd.getNumPartitions())
```

This returns the DataFrame's current number of execution partitions through its underlying RDD interface.

The number of partitions may change during processing.

For example, `repartition()`, `coalesce()`, joins, and aggregations can affect partitioning.

### 5.2 Storage Partitioning

Storage partitioning organizes files into directories based on column values.

For example:

```text
sales/
  year=2025/
    month=01/
    month=02/
  year=2026/
    month=01/
    month=02/
```

A query filtering for `year = 2026` may be able to skip files in the 2025 directories.

This is called **partition pruning**.

Storage partitioning can improve query performance when the chosen columns match common filters. It can also create many small directories and files if the design uses too many distinct values.

### 5.3 Hash Partitioning

Hash partitioning uses a hash function on one or more columns to distribute records across partitions.

Conceptually:

```text
Partition = hash(customer_id) % number_of_partitions
```

Spark's actual implementation and partitioner behavior can vary by API and operation; the formula above is illustrative.

Hash partitioning is useful for distributing records by keys for operations such as joins and aggregations.

However, if one key appears much more frequently than others, hash partitioning alone does not eliminate the resulting data skew.

### 5.4 Range Partitioning

Range partitioning distributes records based on ranges of ordered values.

Example:

```text
Partition 1: IDs 1–1000
Partition 2: IDs 1001–2000
Partition 3: IDs 2001–3000
```

These ranges are illustrative, not a guarantee of Spark's exact boundaries.

Range partitioning can help with ordered processing and range-based workloads, but uneven value distributions can still create skew.

---

## 6. How Spark Uses Partitions

Spark executes distributed work through stages and tasks.

A simplified execution model is:

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

- **Application:** The Spark program and its execution environment.
- **Job:** Work triggered by an action.
- **Stage:** A group of tasks that can execute without crossing a required shuffle boundary within that stage.
- **Task:** A unit of work that typically processes one partition for a stage.
- **Executor:** A process that runs tasks and can store cached data.

For a particular stage, Spark commonly creates one task per partition of that stage's input.

The number of tasks can therefore change between stages when operations redistribute data.

### Example

```python
df = spark.read.parquet("/data/orders")

filtered_df = df.filter(df.amount > 100)

filtered_df.count()
```

The filter can often execute independently on each input partition. The `count()` action triggers execution.

The exact number of jobs, stages, and tasks depends on the execution plan and runtime behavior.

---

## 7. Controlling the Number of Partitions

### 7.1 `repartition()`

`repartition()` creates a DataFrame with a specified number of partitions or repartitions data by specified expressions.

Example:

```python
df_repartitioned = df.repartition(8)
```

This requests eight execution partitions.

You can also repartition by a column:

```python
df_by_customer = df.repartition(8, "customer_id")
```

This redistributes records using the specified partitioning expression.

**Important:** `repartition()` generally involves a shuffle, which can require substantial network and disk I/O.

Use it when the benefits of redistribution justify the cost.

### 7.2 `coalesce()`

`coalesce()` is commonly used to reduce the number of partitions without a full shuffle.

Example:

```python
df_reduced = df.coalesce(4)
```

This requests four partitions, assuming the input has enough partitions to reduce.

It is useful when reducing task or output-file counts after processing.

However:
- It does not guarantee perfectly balanced partition sizes.
- It can reduce available parallelism.
- It is not generally the right choice for redistributing severely skewed data.

### 7.3 `repartition()` vs `coalesce()`

| Feature | `repartition()` | `coalesce()` |
|---|---|---|
| Typical purpose | Increase or decrease partitions | Usually reduce partitions |
| Full shuffle | Generally yes | Generally no |
| Data redistribution | Redistributes data | Usually combines existing partitions |
| Balancing | Can improve balance, but not guarantee it | May preserve uneven partition sizes |
| Cost | Often higher because of shuffle | Often lower for simple reduction |

Remember: `pyspark.sql.functions.coalesce()` is a different function. It returns the first non-NULL value among its arguments.

---

## 8. Choosing the Right Number of Partitions

There is no universal ideal number of partitions.

Too few partitions:
- Limit available parallelism.
- May create very large tasks.
- Can increase memory pressure.
- May lead to long-running tasks.

Too many partitions:
- Increase scheduling overhead.
- May create many small output files.
- Can waste resources on tiny tasks.
- May increase metadata and file-management overhead.

### Practical Approach

1. Understand the dataset size.
2. Check the current number and sizes of partitions.
3. Inspect available executor cores and resources.
4. Run the workload and examine the Spark UI.
5. Look for long-running tasks, skew, spills, and excessive small tasks.
6. Adjust partitioning and measure the result.

Avoid choosing partition counts solely by copying a fixed number from another project.

---

## 9. Data Skew and Partitioning

**Data skew** occurs when data is distributed unevenly across partitions.

Suppose records are grouped by `country`, but one country accounts for 80% of all records.

A hash-based distribution by country may place most of the records for that country in one partition.

```text
Partition 1: 1,000,000 rows
Partition 2:   100,000 rows
Partition 3:    90,000 rows
Partition 4:    80,000 rows
```

One task may take much longer than the others.

This creates a bottleneck because a stage often cannot finish until its required tasks finish.

### Ways to Handle Skew

- Inspect partition sizes and task-duration differences.
- Use Adaptive Query Execution (AQE) where supported.
- Use skew-join optimization for applicable joins.
- Reconsider partitioning keys.
- Use salting for suitable key-based operations.
- Filter or process exceptional keys separately when appropriate.
- Avoid increasing the partition count without understanding the cause.

Increasing the number of partitions alone does not necessarily split a single hot key evenly.

---

## 10. Partitioning and Shuffle

A **shuffle** redistributes data across partitions, often across the network.

Operations that may require a shuffle include:
- `groupBy()`
- `distinct()`
- Many joins
- `orderBy()`
- `repartition()`

Example:

```python
result = (
    df.groupBy("department")
      .sum("salary")
)
```

To aggregate records by department across the dataset, Spark may need to bring records with the same grouping key together.

This redistribution can introduce:
- Network transfer.
- Disk I/O.
- Serialization overhead.
- Additional memory pressure.
- Possible disk spill.

Not every operation triggers a shuffle. Filters and many simple column transformations can execute independently within existing partitions.

### Important Distinction

Partitioning determines how data is divided or distributed. A shuffle is one mechanism Spark uses to redistribute data between partitions.

---

## 11. Partitioning and Output Files

When Spark writes a DataFrame, it commonly creates multiple part files. The number and size of files depend on the execution partitions, writer behavior, data volume, and output options.

Example:

```python
(
    df.coalesce(4)
      .write
      .mode("overwrite")
      .parquet("/output/orders")
)
```

This reduces the DataFrame to four partitions before writing, but it does not guarantee exactly four nonempty output files in every situation.

### Too Many Small Files

Small files can cause:
- More file-open operations.
- More metadata overhead.
- Slower query planning and file listing.
- Increased maintenance work.

### Too Few Large Files

Very large files or too few output partitions can reduce read parallelism and make individual tasks expensive.

Choose output layout based on data size, query patterns, and the storage format. There is no single file size that is ideal for every workload.

For Delta Lake, file compaction with `OPTIMIZE` can help improve file layout, but it consumes compute resources.

---

## 12. Storage Partitioning vs Execution Partitioning

This distinction is especially important for Data Engineer interviews.

| Feature | Storage Partitioning | Execution Partitioning |
|---|---|---|
| Where it applies | Data stored on disk or object storage | Data processed by Spark |
| Main purpose | Organize stored data for efficient reads | Distribute computation |
| Example | Directories grouped by year and month | DataFrame has 100 Spark partitions |
| Typical benefit | Partition pruning | Parallel task execution |
| Common concern | Too many storage partitions or small files | Skew, scheduling overhead, uneven tasks |

A table stored in `year` directories does not necessarily have one Spark execution partition per year.

Spark can read files from multiple storage partitions and create a different execution layout for processing.

---

## 13. Practical PySpark Example

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col

spark = (
    SparkSession.builder
    .appName("PartitioningExample")
    .getOrCreate()
)

df = spark.read.parquet("/data/sales")

# Inspect current execution partition count
print("Original:", df.rdd.getNumPartitions())

# Filter records
filtered_df = df.filter(col("amount") > 100)

# Request eight execution partitions
repartitioned_df = filtered_df.repartition(8)

print(
    "After repartition:",
    repartitioned_df.rdd.getNumPartitions()
)

# Reduce partition count before writing
output_df = repartitioned_df.coalesce(4)

output_df.write.mode("overwrite").parquet(
    "/output/sales"
)
```

### What This Code Does

1. Reads sales data from Parquet.
2. Displays the current execution partition count.
3. Filters records with amounts greater than 100.
4. Requests eight partitions using `repartition()`.
5. Reduces the partition count to four using `coalesce()`.
6. Writes the output as Parquet.

### Important Considerations

- `repartition(8)` generally triggers a shuffle.
- `coalesce(4)` usually reduces partitions without a full shuffle.
- The code demonstrates API behavior; it does not prove that eight or four partitions are optimal.
- Overwrite mode can replace existing output data, so use it only when that behavior is intended.
- For large datasets, inspect the execution plan and Spark UI instead of relying only on partition counts.

---

## 14. Common Interview Mistakes

| Incorrect Statement | Correct Understanding |
|---|---|
| More partitions always mean faster execution | Excessive partitions can increase scheduling overhead |
| One partition always equals one executor | A partition is a data unit; a task processes it, and executors can run multiple tasks |
| `repartition()` is free | It generally introduces a shuffle |
| `coalesce()` always balances data | It may preserve uneven partition sizes |
| Storage partitions equal Spark partitions | Storage layout and execution layout are different |
| Hash partitioning removes skew | A frequent key can still dominate a partition |
| All transformations shuffle data | Many narrow transformations do not require a shuffle |
| `repartition(8)` means eight tasks run simultaneously | It requests eight partitions; concurrency depends on available resources |
| Four partitions always produce exactly four output files | Writer behavior and empty partitions can affect output-file count |
| Increasing partitions fixes every slow task | The root cause may be skew, I/O, memory pressure, or expensive computation |

---

## 15. Interview Questions and Answers

### Q1. What is partitioning in Spark?

Partitioning divides a distributed dataset into smaller units that can be processed by Spark tasks.

### Q2. What is parallelism?

Parallelism is the execution of multiple tasks at the same time using available resources.

### Q3. What is the relationship between partitioning and parallelism?

Partitions provide units of work for tasks. More runnable partitions can create more opportunities for parallelism, but actual concurrency depends on cluster resources and task dependencies.

### Q4. What is the difference between `repartition()` and `coalesce()`?

`repartition()` generally shuffles data and can increase or decrease partitions. `coalesce()` usually reduces partitions with less data movement, but it may not balance the data evenly.

### Q5. What is data skew?

Data skew occurs when data is unevenly distributed across partitions, causing some tasks to process much more data or take much longer than others.

### Q6. What is a shuffle?

A shuffle redistributes data across partitions, often across the network. It is common in grouping, distinct operations, many joins, sorting, and repartitioning.

### Q7. Does each Spark partition run on a separate executor?

No. A task processes a partition, and an executor can run multiple tasks concurrently if it has sufficient task slots and resources.

### Q8. What is the difference between storage and execution partitioning?

Storage partitioning organizes files on disk or object storage. Execution partitioning divides data for computation inside Spark.

### Q9. How would you investigate a slow Spark job caused by partitioning?

I would inspect the Spark UI for task-duration differences, input sizes, shuffle reads and writes, and spill. Then I would check for skew, excessive small tasks, and unsuitable partition counts before changing the layout.

### Q10. How do you choose a partition count?

I would consider data volume, available cores, task duration, partition sizes, shuffle cost, and output-file sizes. I would test and measure rather than use a universal fixed number.

### Q11. Can increasing partitions solve data skew?

Not always. If a single key dominates the workload, repartitioning by the same key may leave the hot key concentrated in one partition. Skew-aware techniques may be needed.

### Q12. Does partitioning data by date always improve performance?

No. It helps when queries commonly filter by date and the layout is appropriate. Excessive date partitions or many small files can make performance worse.

---

## 16. Quick Revision

- **Partitioning:** Divides data into smaller units.
- **Parallelism:** Runs multiple tasks concurrently.
- **Spark task:** Typically processes one partition for a stage.
- **`repartition()`:** Redistributes data and generally triggers a shuffle.
- **`coalesce()`:** Usually reduces partitions with less data movement.
- **Storage partitioning:** Organizes stored files by column values.
- **Hash partitioning:** Distributes data using hash-based partition expressions.
- **Range partitioning:** Distributes data using value ranges.
- **Shuffle:** Redistributes data across partitions.
- **Data skew:** Uneven distribution that can create task bottlenecks.
- **Partition pruning:** Skips storage partitions irrelevant to a query.
- **Output files:** Commonly depend on execution partitions and writer behavior.
- **Best practice:** Inspect metrics, understand the bottleneck, and measure changes.

### One-Minute Interview Answer

Partitioning divides a large dataset into smaller units, while parallelism means processing multiple units at the same time. In Spark, tasks process partitions, and the number of tasks that can run concurrently depends on available executor resources.

I use `repartition()` when data needs redistribution or a different partition count, keeping in mind that it generally causes a shuffle. I use `coalesce()` when reducing partitions with less data movement is appropriate. I also distinguish execution partitioning from storage partitioning, which organizes files and can enable partition pruning.

When tuning a job, I inspect the Spark UI for task duration, partition sizes, shuffle overhead, and data skew rather than assuming that more partitions will always improve performance.

---

## 17. Self-Assessment

- [ ] Define partitioning and parallelism.
- [ ] Explain how Spark partitions become tasks.
- [ ] Differentiate storage and execution partitioning.
- [ ] Explain hash and range partitioning.
- [ ] Compare `repartition()` and `coalesce()`.
- [ ] Explain what causes a shuffle.
- [ ] Describe data skew and ways to investigate it.
- [ ] Explain how partitioning affects output files.
- [ ] Describe how to select a suitable partition count.
- [ ] Explain why more partitions do not always improve performance.

**Interview readiness:** Be able to explain how Spark uses partitions to create tasks, why `repartition()` can be expensive, and how you would diagnose uneven task execution using the Spark UI.