# Spark Performance Optimization

## 1. What Is Spark Optimization?

Spark optimization is the process of improving the performance and resource efficiency of Spark applications.

The main goals are to:

- Reduce execution time.
- Minimize unnecessary data movement.
- Reduce memory consumption.
- Avoid excessive disk I/O.
- Improve parallel processing.
- Reduce unnecessary task scheduling.
- Use cluster resources efficiently.

A well-optimized Spark job processes the required data with less unnecessary computation and resource usage.

## 2. Why Do Spark Jobs Become Slow?

Common reasons include:

1. Excessive shuffling.
2. Data skew.
3. Too many small files.
4. Poor join strategies.
5. Unnecessary columns and records.
6. Too many or too few partitions.
7. Excessive caching.
8. Memory spill.
9. Inefficient transformations.
10. Poor data layout and partitioning.

Before optimizing, identify the actual bottleneck. Do not randomly change Spark configurations.

## 3. Understand Spark's Optimization Layers

Spark performance optimization can be understood through four main areas.

### A. Query Optimization

Spark's Catalyst Optimizer improves logical query plans.

Examples:
- Predicate pushdown.
- Column pruning.
- Constant folding.
- Simplifying expressions.

### B. Execution Optimization

Spark chooses physical execution strategies.

Examples:
- Broadcast Hash Join.
- Sort Merge Join.
- Shuffle Hash Join.
- Whole-stage code generation.

### C. Data Optimization

Improve how data is stored and distributed.

Examples:
- Appropriate partitioning.
- File compaction.
- Columnar formats such as Parquet.
- Delta Lake optimization.

### D. Resource Optimization

Use cluster resources efficiently.

Examples:
- Appropriate executor memory.
- Suitable partition counts.
- Avoiding unnecessary caching.
- Managing parallelism.

## 4. Use the Catalyst Optimizer

Catalyst is Spark SQL's query optimization framework.

It transforms a query into an optimized execution plan.

Example:

```python
result = df.filter(
    "amount > 1000"
).select(
    "customer_id",
    "amount"
)
```

Catalyst can optimize this query by pushing filters closer to the data source when supported and reading only the required columns when possible.

### Important Catalyst Optimizations

| Optimization | Purpose |
|---|---|
| Predicate Pushdown | Apply filters closer to the data source |
| Column Pruning | Read only required columns |
| Constant Folding | Evaluate constant expressions during optimization |
| Constant Propagation | Use known constant values to simplify expressions |
| Boolean Simplification | Simplify logical conditions |
| Join Reordering | Reorder eligible joins when beneficial |

**Important:** These optimizations depend on the query, data source, available statistics, and supported operations.

## 5. Filter Early

Avoid processing records that are not required.

Less efficient:

```python
joined = orders.join(
    customers,
    on="customer_id",
    how="inner"
)

result = joined.filter(
    "order_status = 'COMPLETED'"
)
```

Better:

```python
filtered_orders = orders.filter(
    "order_status = 'COMPLETED'"
)

result = filtered_orders.join(
    customers,
    on="customer_id",
    how="inner"
)
```

Filtering early can reduce:
- Records processed.
- Shuffle volume.
- Join input size.
- Memory requirements.

Spark may automatically push filters down when it is safe and supported, but writing clear transformations is still good practice.

## 6. Select Only Required Columns

Avoid carrying unnecessary columns through transformations.

Less efficient:

```python
result = df.select("*")
```

Better:

```python
result = df.select(
    "customer_id",
    "order_id",
    "amount"
)
```

Benefits:
- Less data read when column pruning is supported.
- Less memory consumption.
- Less data shuffled.
- Smaller intermediate DataFrames.

This technique is commonly called **column pruning**.

## 7. Optimize Joins

Joins can become expensive when Spark needs to shuffle large datasets.

### A. Use Broadcast Joins

If one dataset is small enough, broadcast it.

```python
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    on="customer_id",
    how="inner"
)
```

Benefits:
- Avoids shuffling the large side for the join.
- Can reduce network communication.
- Often improves performance for small dimension tables.

Do not broadcast a dataset without checking its size and executor memory requirements.

### B. Handle Data Skew

Data skew occurs when some join keys contain significantly more records than others.

Possible solutions:
- AQE skew join optimization.
- Salting.
- Broadcasting a suitable small dataset.
- Separating hot keys.
- Pre-aggregation.

### C. Inspect the Join Strategy

```python
result.explain("formatted")
```

Look for:
- BroadcastHashJoin.
- SortMergeJoin.
- ShuffledHashJoin.
- Exchange.
- Sort.

Choose the optimization based on the actual execution plan.

## 8. Reduce Unnecessary Shuffling

Shuffling redistributes data across partitions.

It commonly occurs in:
- `groupBy()`.
- `distinct()`.
- `repartition()`.
- Many join operations.
- Global sorting.

Shuffling can involve network communication, serialization, disk I/O, and memory usage.

### Example: Repartitioning

```python
df = df.repartition(20)
```

This redistributes data through a shuffle.

Use it when you need to change the distribution or number of partitions.

### Example: Coalesce

```python
df = df.coalesce(10)
```

This generally reduces partitions without a full shuffle.

Use it when reducing the number of partitions, such as before writing output files.

**Important:** Avoid unnecessary repartitioning. Every additional shuffle can increase execution cost.

## 9. Choose Appropriate Partitioning

Partitioning affects how Spark distributes work.

### A. Too Few Partitions

Possible problems:
- Large tasks.
- Poor parallelism.
- High memory requirements per task.

### B. Too Many Partitions

Possible problems:
- Too many small tasks.
- Scheduling overhead.
- Many small output files.

### C. Repartition by a Column

```python
df = df.repartition(
    20,
    "customer_id"
)
```

This distributes records using the specified key.

However, if a key is highly skewed, partitioning by that key may still create uneven partitions.

### D. Storage Partitioning

```python
df.write.partitionBy(
    "sale_date"
).parquet("/output/sales")
```

This organizes files into directories based on `sale_date`.

Choose storage partition columns based on common query filters and cardinality. Avoid unnecessarily high-cardinality partition columns.

## 10. Optimize File Sizes

Too many small files can slow down Spark queries.

Common causes:
- Frequent incremental writes.
- Too many output partitions.
- High-cardinality storage partitioning.

Solutions:
- Use an appropriate number of output partitions.
- Use `coalesce()` or `repartition()` where appropriate.
- Compact existing files.
- Use optimized writes where supported.

For Delta Lake:

```sql
OPTIMIZE sales;
```

This compacts smaller files into fewer, larger files.

A common starting target for analytical files is around 128 MB to 1 GB, but the best size depends on the workload and platform.

## 11. Use Efficient File Formats

Different file formats have different performance characteristics.

| Format | Characteristics |
|---|---|
| CSV | Text-based; commonly requires parsing |
| JSON | Flexible structure; parsing overhead |
| Parquet | Columnar format; supports column pruning and compression |
| ORC | Columnar format; optimized for analytical processing |
| Delta | Adds transaction and table-management capabilities over Parquet-based storage |

For analytical workloads, Parquet and Delta are common choices.

### Example: Write Parquet

```python
df.write.mode("overwrite").parquet(
    "/output/sales"
)
```

### Example: Write Delta

```python
df.write.format("delta").mode(
    "overwrite"
).save("/output/sales_delta")
```

Columnar formats can reduce the amount of data read when only selected columns are required.

## 12. Use Adaptive Query Execution (AQE)

AQE allows Spark to adjust eligible parts of a query plan using runtime statistics.

Important capabilities:

1. Convert eligible Sort Merge Joins into Broadcast Hash Joins.
2. Handle skewed partitions in supported shuffled joins.
3. Coalesce small shuffle partitions.

Enable it using:

```python
spark.conf.set(
    "spark.sql.adaptive.enabled",
    "true"
)
```

AQE is enabled by default in standard Spark SQL configurations from Spark 3.2 onward, but platform settings may differ.

AQE does not automatically solve every performance problem.

## 13. Optimize Caching and Persistence

Caching is useful when the same DataFrame is reused multiple times.

Example:

```python
df.cache()

df.count()

# Reuse df in multiple operations

df.unpersist()
```

Benefits:
- Avoids recomputing reusable data.
- Can improve repeated operations.

However:
- Caching consumes storage memory.
- Large cached datasets can create memory pressure.
- Caching a DataFrame used only once may not provide a benefit.

Use caching only when reuse justifies its cost.

## 14. Avoid Expensive Operations When Unnecessary

Some operations can require substantial data movement or computation.

| Operation | Why It Can Be Expensive |
|---|---|
| `collect()` | Transfers all records to the driver |
| `distinct()` | Usually requires data redistribution |
| `orderBy()` | Global sorting can require a shuffle |
| `groupBy()` | Usually requires data redistribution |
| `repartition()` | Performs a shuffle |
| Repeated actions | May recompute uncached transformations |

Use the appropriate alternative when possible.

For example:

```python
df.show(10)
```

is safer for inspecting a small sample than collecting an entire large DataFrame.

For existence checks, consider whether a limited operation can meet the requirement instead of computing a full result.

## 15. Tune Memory and Executor Resources

Resource tuning can improve performance when the job is genuinely constrained by available resources.

Important settings include:

### Executor Memory

```bash
--executor-memory 4g
```

Controls executor JVM heap allocation.

### Driver Memory

```bash
--driver-memory 2g
```

Controls driver JVM heap allocation.

### Executor Cores

```bash
--executor-cores 4
```

Controls the number of task slots available to an executor in common cluster configurations.

Actual resource behavior depends on the cluster manager and deployment mode.

**Important:** Increasing memory or cores does not automatically make every job faster. First identify whether the bottleneck is CPU, memory, disk, network, skew, or scheduling.

## 16. Understand Memory Spill

Memory spill occurs when Spark cannot keep all intermediate processing data in memory and writes some of it to disk.

Common causes:
- Large joins.
- Large aggregations.
- Sorting.
- Data skew.
- Insufficient working memory.

Check Spark UI for:
- Memory Bytes Spilled.
- Disk Bytes Spilled.
- Shuffle Read.
- Shuffle Write.
- Task Duration.

To reduce spill:
- Filter unnecessary records.
- Select fewer columns.
- Handle skew.
- Use suitable join strategies.
- Avoid unnecessary caching.
- Choose appropriate partition counts.

Do not immediately increase executor memory without investigating the root cause.

## 17. Use Spark UI for Performance Investigation

Spark UI is one of the most important tools for diagnosing performance issues.

### A. Jobs Tab

Check:
- Total job execution time.
- Number of stages.
- Failed jobs.

### B. Stages Tab

Check:
- Stage duration.
- Number of tasks.
- Shuffle read and write.
- Memory and disk spill.

### C. Tasks

Compare:
- Task execution time.
- Input size.
- Shuffle read.
- Spill metrics.

If one task takes much longer than the others, investigate possible data skew or uneven partition sizes.

### D. SQL Tab

Check:
- Physical execution plan.
- Join strategies.
- Adaptive execution changes.
- Query execution metrics.

### E. Storage Tab

Check:
- Cached DataFrames.
- Memory usage.
- Storage level.
- Cached partitions.

## 18. Use explain() to Inspect Query Plans

Example:

```python
result.explain("formatted")
```

This helps inspect the physical execution plan.

Important operators:

| Operator | Meaning |
|---|---|
| BroadcastHashJoin | Broadcast hash join |
| SortMergeJoin | Sort merge join |
| ShuffledHashJoin | Shuffle hash join |
| Exchange | Often indicates data redistribution |
| Sort | Sorting operation |
| FileScan | Reading files |
| Filter | Filtering records |
| HashAggregate | Hash-based aggregation |

Compare the plan before and after optimization.

Remember that an `Exchange` node is not automatically a problem. Some operations require data redistribution.

## 19. End-to-End Performance Optimization Example

**Scenario:**

A daily sales pipeline processes 100 GB of data and takes 90 minutes.

You are asked to improve its performance.

### Step 1: Understand the Workload

Identify:
- Input data size.
- Number of files.
- Number of partitions.
- Join operations.
- Aggregations.
- Output size.

### Step 2: Inspect Spark UI

Check:
- Slow stages.
- Long-running tasks.
- Shuffle read and write.
- Spill.
- Uneven task execution.

### Step 3: Inspect the Physical Plan

```python
result.explain("formatted")
```

Identify:
- Expensive joins.
- Unnecessary exchanges.
- Sorting operations.
- File scans.

### Step 4: Reduce Data Volume

- Filter records early.
- Select required columns.
- Avoid unnecessary transformations.

### Step 5: Optimize Joins

- Broadcast small dimension tables when appropriate.
- Handle data skew.
- Check join keys and statistics.

### Step 6: Optimize Partitions and Files

- Choose appropriate execution partitions.
- Avoid unnecessary repartitioning.
- Compact small files when needed.

### Step 7: Review Caching and Memory

- Cache only reused DataFrames.
- Check spill.
- Investigate executor memory pressure.

### Step 8: Measure Again

Compare:
- Execution time.
- Shuffle volume.
- Spill.
- Task distribution.
- Resource usage.

Do not claim an improvement based only on a changed configuration. Measure the actual result.

## 20. Common Optimization Mistakes

1. Increasing executor memory without identifying the bottleneck.
2. Broadcasting large datasets.
3. Repartitioning repeatedly.
4. Caching every DataFrame.
5. Ignoring data skew.
6. Using too many storage partition columns.
7. Assuming AQE fixes every performance problem.
8. Using `collect()` on large datasets.
9. Ignoring small files.
10. Changing multiple configurations without measuring their impact.
11. Assuming every shuffle is unnecessary.
12. Optimizing execution time without checking output correctness.

## 21. Interview Questions

**Q1. What is Spark performance optimization?**

Improving execution time and resource efficiency by reducing unnecessary computation, data movement, memory usage, and scheduling overhead.

**Q2. What is Catalyst Optimizer?**

Spark SQL's query optimization framework that transforms logical plans into optimized plans.

**Q3. What is predicate pushdown?**

Applying filters closer to the data source to reduce the amount of data read when supported.

**Q4. What is column pruning?**

Reading only the columns required by the query.

**Q5. How do you optimize joins in Spark?**

Use suitable join strategies, broadcast small datasets when appropriate, filter early, select required columns, and handle skew.

**Q6. What is data skew?**

Uneven data distribution where some partitions contain significantly more records than others.

**Q7. How do you reduce shuffling?**

Avoid unnecessary repartitioning, filter and select early, use suitable join strategies, and avoid unnecessary wide transformations.

**Q8. What is AQE?**

Adaptive Query Execution uses runtime statistics to adjust eligible execution plans.

**Q9. How do you investigate a slow Spark job?**

Use Spark UI, inspect slow stages and tasks, check shuffle and spill metrics, and examine the physical execution plan.

**Q10. What is the small files problem?**

Too many small files increase metadata, file listing, scheduling, and read overhead.

**Q11. When should you cache a DataFrame?**

When it is reused multiple times and the cost of caching is justified by avoiding recomputation.

**Q12. What is the difference between coalesce() and repartition()?**

`coalesce()` generally reduces partitions without a full shuffle. `repartition()` redistributes data through a shuffle and can increase or decrease partitions.

**Q13. Does increasing executor memory always improve performance?**

No. It helps only when memory availability is a real bottleneck. Other causes such as skew, shuffle, and inefficient queries must also be investigated.

**Q14. How do you verify that an optimization worked?**

Compare execution time, shuffle volume, spill, task distribution, resource usage, and output correctness before and after the change.

## 22. Practice Questions

1. Your Spark job takes 90 minutes. How would you identify the bottleneck?
2. A join creates a large shuffle. What would you investigate?
3. One task takes much longer than all other tasks. What could be the reason?
4. How would you optimize a DataFrame containing unnecessary columns?
5. Why can too many small files affect query performance?
6. When should you use broadcast joins?
7. How does AQE improve performance?
8. Why is caching not always beneficial?
9. How would you identify memory spill?
10. What is the difference between query optimization and execution optimization?
11. Why can repartitioning increase execution time?
12. How would you optimize a pipeline that reads, joins, aggregates, and writes large datasets?

## 23. Quick Revision

- **Catalyst:** Optimizes Spark SQL query plans.
- **Predicate Pushdown:** Filters data closer to the source.
- **Column Pruning:** Reads only required columns.
- **Broadcast Join:** Avoids shuffling the large side for the join.
- **Sort Merge Join:** Shuffles when required, sorts, and merges.
- **Data Skew:** Uneven data distribution.
- **AQE:** Runtime query plan optimization.
- **Shuffle:** Redistributes data across partitions.
- **Repartition:** Redistributes data through a shuffle.
- **Coalesce:** Generally reduces partitions without a full shuffle.
- **Caching:** Stores reusable data.
- **Spill:** Writes intermediate execution data to disk.
- **Small Files:** Too many small physical files.
- **Spark UI:** Identifies execution bottlenecks.
- **explain("formatted"):** Inspects the physical plan.
- **Optimization process:** Inspect → Identify → Optimize → Measure.

## 24. Self-Check

- [ ] Explain Spark's optimization layers.
- [ ] Understand Catalyst Optimizer.
- [ ] Explain predicate pushdown and column pruning.
- [ ] Optimize joins using suitable strategies.
- [ ] Understand data skew and shuffle.
- [ ] Explain AQE.
- [ ] Choose appropriate partitions.
- [ ] Understand file compaction.
- [ ] Use caching appropriately.
- [ ] Identify memory spill.
- [ ] Read Spark UI metrics.
- [ ] Inspect physical execution plans.
- [ ] Explain an end-to-end optimization scenario.
- [ ] Measure performance improvements.

**Interview goal:** When asked how you optimize a Spark pipeline, give a structured answer: inspect the Spark UI and execution plan, identify the bottleneck, apply the relevant optimization, and measure the improvement while preserving output correctness.