# Apache Spark Cheat Sheet

## 1. What Is Apache Spark?

Apache Spark is a distributed data-processing engine used to process large datasets across multiple machines.

It supports:
- Batch processing
- Streaming and near-real-time processing
- SQL queries
- DataFrame transformations
- Machine learning and graph processing through supported libraries

**Why Spark?** It distributes computation across a cluster instead of requiring all data to fit on one machine.

### Spark vs PySpark

| Term | Meaning |
|---|---|
| Apache Spark | Distributed processing engine |
| PySpark | Python API for Spark |
| Spark SQL | Structured-data processing and SQL engine |
| Spark Streaming | Legacy DStream-based streaming API |
| Structured Streaming | Modern streaming API built on Spark SQL |
| Databricks | A platform that provides managed Spark and additional data tools |

**Interview point:** PySpark is not a separate processing engine. It is the Python interface to Apache Spark.

---

## 2. Core Architecture

A typical Spark application contains:

- **Driver:** Runs the application logic, builds execution plans, and coordinates work.
- **Executors:** Run tasks and can cache data and maintain task-related state.
- **Cluster manager:** Allocates resources to Spark applications.
- **Worker nodes:** Machines that provide resources for executors.

### Important components

| Component | Responsibility |
|---|---|
| SparkSession | Main entry point for DataFrame and Spark SQL operations |
| SparkContext | Connects an application to Spark's execution environment |
| Driver | Coordinates execution |
| Executor | Executes tasks on worker resources |
| Cluster manager | Allocates resources |
| Task | Smallest scheduled unit of Spark computation |
| Stage | Group of tasks separated by execution dependencies |
| Job | Execution triggered by an action or another execution request |

### Execution flow

1. Write PySpark transformations.
2. Spark builds a logical plan.
3. Spark optimizes the plan.
4. Spark creates a physical execution plan.
5. An action triggers execution when needed.
6. Spark divides the work into stages and tasks.
7. Executors process data partitions.
8. Results are returned, written, or otherwise consumed.

**Important:** The exact execution details depend on the operation, execution plan, and Spark version.

---

## 3. SparkSession

`SparkSession` is the standard entry point for structured Spark processing.

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("DataEngineering")
    .getOrCreate()
)
```

Useful operations:

```python
df = spark.read.parquet("/data/orders")

df.show()
df.printSchema()
df.count()

spark.sql("SELECT 1").show()
```

- `getOrCreate()` reuses an existing session when possible.
- `spark.read` loads data.
- `spark.sql()` executes Spark SQL.
- `spark.stop()` stops the Spark session's application context when appropriate.

---

## 4. RDD vs DataFrame vs Dataset

| Feature | RDD | DataFrame | Dataset |
|---|---|---|---|
| Abstraction | Distributed objects | Structured rows and columns | Typed structured objects |
| Schema | Not required | Yes | Yes |
| Optimization | Less automatic SQL optimization | Catalyst and physical planning | Structured optimizations |
| API | Low-level | High-level | High-level, typed |
| Python support | Yes | Yes | No strongly typed Dataset API in PySpark |
| Typical use | Specialized low-level processing | Most data engineering workloads | Common in typed Scala/Java Spark applications |

**Recommendation:** Prefer DataFrames for most PySpark data engineering tasks because Spark can optimize structured operations.

RDDs remain useful for certain low-level tasks, but they are not the default choice for ordinary ETL.

---

## 5. Transformations vs Actions

### Transformations

Transformations define new datasets or operations.

Examples:

```python
df.filter(df.amount > 100)
df.select("order_id", "amount")
df.withColumn("amount_with_tax", df.amount * 1.18)
df.groupBy("customer_id").sum("amount")
df.join(customers, "customer_id")
```

Most transformations are lazy: Spark records the computation instead of immediately processing all the data.

### Actions

Actions request results or trigger execution.

```python
df.show()
df.count()
df.collect()
df.first()
df.take(10)
df.write.mode("overwrite").parquet("/output/orders")
```

### Quick comparison

| Transformation | Action |
|---|---|
| `filter()` | `count()` |
| `select()` | `show()` |
| `withColumn()` | `collect()` |
| `groupBy().agg()` | `write` |
| `join()` | `take()` |

**Interview question: Why is Spark lazy?**

Lazy evaluation lets Spark optimize operations and combine compatible work before execution. It can also avoid executing transformations whose results are never requested.

**Caution:** `collect()` transfers all rows to the driver and can cause driver memory problems on large datasets.

---

## 6. Narrow vs Wide Transformations

### Narrow transformation

Each output partition depends on a limited set of input partitions, without requiring a general shuffle across the cluster.

Examples commonly include:
- `filter()`
- `select()`
- `withColumn()`
- `map()` on an RDD

### Wide transformation

Output partitions may depend on data from many input partitions. This commonly requires a shuffle.

Examples:
- `groupBy()`
- `distinct()`
- `repartition()`
- Many joins
- Sorting across partitions

| Narrow | Wide |
|---|---|
| Usually avoids a shuffle | Usually requires a shuffle |
| Often less expensive | Often more expensive |
| Can execute within a stage | Commonly introduces a stage boundary |

**Interview point:** A shuffle redistributes data between partitions, often across executors. It can involve network transfer, serialization, disk I/O, and additional memory use.

Not every join has the same execution behavior. For example, a broadcast join can avoid shuffling the large side for the join.

---

## 7. Partitions

A partition is a logical chunk of distributed data processed by Spark tasks.

- A task generally processes one partition for a given stage.
- More partitions can increase parallelism.
- Too few partitions can leave resources underused.
- Too many tiny partitions can increase scheduling overhead.
- Uneven partition sizes can create data skew.

### `repartition()` vs `coalesce()`

```python
df_repartitioned = df.repartition(20)
df_reduced = df.coalesce(5)
```

| `repartition()` | `coalesce()` |
|---|---|
| Generally performs a shuffle | Generally avoids a full shuffle when reducing partitions |
| Can increase or decrease partitions | Commonly used to reduce partitions |
| Can redistribute data more evenly | May leave uneven partition sizes |

**Remember:** Partition count should be chosen based on data size, cluster resources, workload, and output requirements—not by a universal rule.

### `partitionBy()` when writing

```python
df.write.partitionBy("order_date").parquet("/output/orders")
```

This creates a directory layout organized by the partition column.

Use partition columns that support common query filters and have reasonable cardinality. High-cardinality partitioning can produce many small files.

---

## 8. Catalyst Optimizer and Physical Planning

Spark SQL uses the Catalyst optimizer to improve structured queries.

Typical stages include:

1. Parse the query.
2. Analyze columns, tables, and types.
3. Optimize the logical plan.
4. Select a physical execution plan.
5. Execute the plan.

Common optimization techniques include:
- Predicate pushdown when supported
- Projection pruning
- Constant folding
- Join optimization
- Partition pruning
- Runtime adaptations through Adaptive Query Execution (AQE), when enabled

### Predicate pushdown

Pushes supported filters closer to the data source so fewer records may need to be read.

### Projection pruning

Reads only the columns required by the query when the source and plan support it.

### Partition pruning

Skips storage partitions that cannot match a filter.

**Important:** These optimizations depend on the source, query, table layout, and runtime. They are not guaranteed for every operation.

---

## 9. Joins

Common join types:

| Join | Result |
|---|---|
| Inner | Matching rows from both sides |
| Left | All left rows and matching right rows |
| Right | All right rows and matching left rows |
| Full outer | All rows from both sides, matched where possible |
| Left semi | Left rows that have a match on the right |
| Left anti | Left rows that do not have a match on the right |

### Join example

```python
orders_with_customers = orders.join(
    customers,
    on="customer_id",
    how="inner"
)
```

### Broadcast join

Useful when one side is small enough to distribute to all relevant executors.

```python
from pyspark.sql.functions import broadcast

result = orders.join(
    broadcast(customers),
    on="customer_id",
    how="inner"
)
```

Broadcasting can avoid shuffling the large side for the join, but it consumes executor memory. Confirm the size and physical plan before using it.

### Skewed joins

Data skew occurs when some keys have much more data than others.

Symptoms:
- A few tasks run much longer than the rest.
- Some tasks process substantially more records.
- Stages stall while a small number of tasks remain.

Possible solutions:
- Broadcast a genuinely small table.
- Enable or tune AQE skew handling when supported.
- Filter unnecessary data before joining.
- Pre-aggregate when business semantics permit.
- Salt heavily skewed keys when appropriate.

**Caution:** Salting adds complexity and should not be the first response before inspecting the execution plan and task metrics.

---

## 10. Aggregations and Window Functions

### Aggregation

```python
from pyspark.sql import functions as F

customer_totals = (
    orders
    .groupBy("customer_id")
    .agg(
        F.sum("amount").alias("total_amount"),
        F.count("*").alias("order_count")
    )
)
```

A grouped aggregation often requires a shuffle.

### Window function

A window function calculates values across related rows without collapsing them into one row per group.

```python
from pyspark.sql import Window
from pyspark.sql import functions as F

window_spec = (
    Window
    .partitionBy("customer_id")
    .orderBy(F.col("order_date").desc(), F.col("order_id").desc())
)

latest_orders = (
    orders
    .withColumn("rn", F.row_number().over(window_spec))
    .filter(F.col("rn") == 1)
    .drop("rn")
)
```

The `order_id` tie-breaker helps make the ordering deterministic if `order_date` values are equal.

Common window functions:
- `row_number()`
- `rank()`
- `dense_rank()`
- `lag()`
- `lead()`
- Running totals using windowed aggregations

**Caution:** Window operations can be expensive because Spark may need to redistribute and sort data within each window partition.

---

## 11. DataFrame Operations to Remember

```python
from pyspark.sql import functions as F

# Select columns
df.select("order_id", "amount")

# Filter rows
df.filter(F.col("amount") > 100)

# Add or replace a column
df.withColumn("tax", F.col("amount") * 0.18)

# Rename a column
df.withColumnRenamed("amount", "order_amount")

# Remove columns
df.drop("temporary_column")

# Remove rows with nulls
df.dropna(subset=["customer_id"])

# Replace null values
df.fillna({"amount": 0})

# Distinct rows
df.distinct()

# Deduplicate based on selected columns
df.dropDuplicates(["order_id"])

# Sort rows
df.orderBy(F.col("amount").desc())

# Count rows
df.count()

# Union by column name
combined = df1.unionByName(df2)
```

### Important differences

- DataFrames are immutable: transformations return a new DataFrame.
- `union()` combines columns by position and retains duplicate rows.
- `unionByName()` aligns columns by name.
- `unionByName(..., allowMissingColumns=True)` fills missing columns with `NULL` where supported.
- `dropDuplicates()` does not guarantee which conflicting row is retained.
- `distinct()` removes duplicate complete rows, not duplicates by a chosen business key.
- `dropna()` removes rows; it does not repair the underlying data.
- `fillna()` is only correct when the replacement value matches the business meaning.

---

## 12. Caching and Persistence

Caching can avoid recomputing an expensive DataFrame when it is reused.

```python
df.cache()

# An action can materialize cached partitions
df.count()

# Release cached data when no longer needed
df.unpersist()
```

Persistence offers more control over storage levels.

Use caching when:
- The same expensive result is reused.
- The cached data fits the available storage resources.
- Recalculation costs more than keeping the data.

Avoid caching every intermediate DataFrame. It can compete for memory and cause unnecessary storage pressure.

**Remember:** Caching is not a substitute for a good execution plan or an efficient data layout.

---

## 13. Explain Plans and Spark UI

### Inspect the plan

```python
df.explain()
df.explain("formatted")
```

The exact output varies by Spark version and runtime.

Look for:
- Unexpected scans
- Large exchanges or shuffles
- Sort operations
- Join strategy
- Partition filters
- Repeated computation
- Broadcast decisions

### Spark UI

Use the Spark UI to investigate:
- Job and stage durations
- Task duration distribution
- Shuffle read and write
- Spill to memory or disk
- Executor memory and failures
- Input and output records
- Skewed tasks

**Interview approach:** Identify the slow stage, inspect task-level metrics, examine the physical plan, form a hypothesis, and test one change at a time.

Do not optimize only by looking at total runtime.

---

## 14. Common Performance Problems

| Problem | Possible response |
|---|---|
| Excessive shuffle | Reduce unnecessary wide transformations; filter earlier |
| Data skew | Inspect key distribution; consider AQE or skew-specific strategies |
| Small files | Compact files when beneficial |
| Driver out of memory | Avoid large `collect()` calls |
| Executor out of memory | Inspect data size, skew, caching, and join strategy |
| Slow joins | Check join keys, statistics, broadcast eligibility, and skew |
| Too few tasks | Review input partitioning and available parallelism |
| Too many tiny tasks | Review partition count and file layout |
| Repeated computation | Reuse or persist expensive results when appropriate |
| Slow source reads | Check file format, column pruning, predicate pushdown, and storage performance |

### Execution tuning principles

1. Read only required columns.
2. Filter early when it reduces data safely.
3. Use efficient data formats such as Parquet for analytical workloads.
4. Inspect join and shuffle behavior.
5. Handle skew based on evidence.
6. Avoid unnecessary caching.
7. Choose partitions and output file sizes based on workload measurements.
8. Validate that optimization preserves the required results.

---

## 15. Memory and Fault Tolerance

### Memory pressure

Spark can experience memory pressure from:
- Large joins
- Aggregations and sorting
- Skewed partitions
- Excessive caching
- Large objects or expensive Python UDFs
- Too much data collected to the driver

**Spill** occurs when intermediate data cannot remain entirely in memory and Spark uses disk as needed. Spill can reduce performance but is not automatically a failure.

### Fault tolerance

Spark can recompute lost partitions using lineage for supported computations. Structured Streaming uses checkpoints and state storage to recover supported query progress and state.

However, recovery behavior depends on the source, sink, operation, and configuration.

**Important:** A checkpoint alone does not guarantee exactly-once results in an arbitrary external system. Sink writes must also be designed for safe retries and duplicate handling.

---

## 16. Structured Streaming Essentials

Structured Streaming processes streaming data using the structured Spark SQL engine. Many queries use micro-batches.

```python
stream_df = (
    spark.readStream
    .format("json")
    .schema(schema)
    .load("/input/events")
)

query = (
    stream_df.writeStream
    .format("parquet")
    .option("path", "/output/events")
    .option("checkpointLocation", "/checkpoints/events")
    .outputMode("append")
    .start()
)
```

Key terms:

| Term | Meaning |
|---|---|
| Source | Where streaming records come from |
| Sink | Where results are written |
| Micro-batch | A batch of records processed during a streaming trigger |
| Checkpoint | Stores progress and, where required, state for recovery |
| Event time | Time an event occurred according to the event data |
| Processing time | Time Spark processes the event |
| Watermark | Event-time progress used to bound state and determine late-data handling |
| Output mode | Determines how query results are emitted |

### Important cautions

- `start()` starts the streaming query.
- `awaitTermination()` waits for the query to terminate; it does not start the query.
- Keep separate checkpoint locations for independent queries.
- Do not casually delete or reuse a checkpoint for an incompatible query.
- Watermarks are based on observed event-time progress, not simply the wall clock.
- Late data handling depends on the query and stateful operation.
- Output mode and delivery guarantees are different concepts.
- Retry safety depends on the sink and write design.

---

## 17. Spark vs SQL vs Pandas

| Feature | Spark DataFrame | SQL engine | Pandas |
|---|---|---|---|
| Typical execution | Distributed | Depends on the database or engine | Usually single-machine memory |
| Best fit | Large-scale distributed processing | Database queries and analytics | Local data analysis |
| Execution optimization | Spark planning and runtime optimizations | Engine-specific optimizer | Local execution |
| Scaling | Across a cluster | Depends on database architecture | Limited by local resources unless used with additional systems |

Choose the tool based on data size, workload, latency, operational complexity, and available infrastructure—not on data size alone.

---

## 18. Common Interview Questions

### Q1. What is Apache Spark?

A distributed data-processing engine that supports large-scale batch and streaming workloads.

### Q2. What is PySpark?

The Python API for working with Apache Spark.

### Q3. Why is Spark lazy?

It allows Spark to optimize a computation plan before executing the requested work.

### Q4. What is a shuffle?

Redistribution of data between partitions, often involving network transfer and disk I/O.

### Q5. What is the difference between a job, stage, and task?

A job is an execution request, a stage is a group of tasks separated by dependencies, and a task processes a partition for a stage.

### Q6. What causes data skew?

Uneven distribution of records or processing work, often because some keys contain much more data than others.

### Q7. How do you optimize a slow Spark job?

Inspect the Spark UI and execution plan, identify the bottleneck, then investigate shuffles, skew, joins, partitioning, file layout, and memory pressure.

### Q8. What is AQE?

Adaptive Query Execution can revise parts of a physical plan using runtime statistics when supported and enabled.

### Q9. What is the difference between `repartition()` and `coalesce()`?

`repartition()` generally shuffles data to redistribute partitions. `coalesce()` commonly reduces partitions with less data movement.

### Q10. Why avoid `collect()` on large DataFrames?

It transfers all rows to the driver, potentially exhausting driver memory.

### Q11. Does Spark always process data in memory?

No. Spark can read and write disk data and spill intermediate results to disk.

### Q12. Does caching always improve performance?

No. Caching helps when reused computations justify the storage cost; otherwise it can consume resources unnecessarily.

### Q13. Does Spark guarantee exactly-once processing?

Not universally across every source and sink. Effective guarantees depend on the source, query, checkpointing, and sink commit or idempotency behavior.

### Q14. What is the difference between a transformation and an action?

A transformation defines a new computation; an action requests a result or triggers execution.

### Q15. How do you identify a broadcast join?

Inspect the physical plan and runtime metrics. A broadcast hash join commonly includes a broadcast exchange for the smaller side.

---

## 19. Common Mistakes

- Assuming every transformation executes immediately.
- Treating a DataFrame as a local Python collection.
- Calling `collect()` on large datasets.
- Broadcasting a table without checking its size and memory implications.
- Assuming all joins require the same shuffle behavior.
- Using `dropDuplicates()` when a deterministic latest-record rule is required.
- Increasing partition count without understanding the bottleneck.
- Caching data that is used only once.
- Assuming more executors will fix every performance problem.
- Assuming AQE eliminates every skew problem.
- Confusing storage partitioning with execution partitions.
- Assuming a successful pipeline run proves data correctness.
- Assuming checkpointing alone guarantees exactly-once sink writes.
- Ignoring data grain, nulls, duplicates, and business rules while tuning performance.

---

## 20. Quick Revision

- **Spark:** Distributed processing engine.
- **PySpark:** Python API for Spark.
- **Driver:** Coordinates application execution.
- **Executor:** Runs tasks and may cache data.
- **Job:** Execution request.
- **Stage:** Group of tasks separated by dependencies.
- **Task:** Processes a partition for a stage.
- **Transformation:** Defines a computation.
- **Action:** Requests a result or triggers execution.
- **Lazy evaluation:** Defers execution until needed.
- **Shuffle:** Redistributes data between partitions.
- **Narrow transformation:** Usually avoids a general shuffle.
- **Wide transformation:** Commonly requires a shuffle.
- **Catalyst:** Spark SQL optimizer.
- **AQE:** Runtime adaptive query optimization.
- **Broadcast join:** Distributes a sufficiently small side to executors.
- **Data skew:** Uneven work distribution.
- **Cache:** Reuses computed data when beneficial.
- **Checkpoint:** Supports recovery of supported progress or state.
- **Structured Streaming:** Structured processing of unbounded data.
- **Watermark:** Bounds event-time state and influences late-data handling.

---

## 21. Self-Assessment Checklist

Before an interview, make sure you can explain these without reading:

- [ ] Spark architecture and execution flow
- [ ] Driver, executor, job, stage, and task
- [ ] RDD vs DataFrame vs Dataset
- [ ] Transformations, actions, and lazy evaluation
- [ ] Narrow vs wide transformations
- [ ] Partitions, `repartition()`, and `coalesce()`
- [ ] Catalyst and AQE
- [ ] Join strategies and data skew
- [ ] Aggregations and window functions
- [ ] Cache, persistence, and checkpointing
- [ ] Spark UI and execution plans
- [ ] Memory pressure, spill, and driver OOM
- [ ] Structured Streaming, checkpoints, and watermarks
- [ ] Practical methods for diagnosing slow jobs

**Final interview rule:** Explain what the operation does, why it matters, what can go wrong, and how you would verify the result or performance improvement.