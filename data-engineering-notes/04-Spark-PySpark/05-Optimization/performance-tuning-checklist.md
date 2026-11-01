# PySpark Performance Tuning Checklist

## 1. What Is Performance Tuning?

Performance tuning is the process of improving Spark job execution by reducing unnecessary computation, data movement, memory usage, and processing time.

The main goal is to make Spark jobs:

- Faster.
- More resource-efficient.
- Scalable for large datasets.
- Reliable under different workloads.

**Key idea:** Before changing Spark configurations, identify the actual performance bottleneck.

---

## 2. Step 1: Understand the Problem

Before optimizing a Spark job, collect basic information.

- [ ] What is the input dataset size?
- [ ] What is the output dataset size?
- [ ] How many input files are being read?
- [ ] How many partitions are involved?
- [ ] Which transformations are expensive?
- [ ] Is the job slow during reading, transformation, shuffle, or writing?
- [ ] Is the problem consistent or does it happen occasionally?

Do not start changing executor memory or partition counts without understanding the problem.

---

## 3. Step 2: Optimize Data Reading

Reading unnecessary data increases processing time and resource usage.

### Checklist

- [ ] Read only required columns.
- [ ] Apply filters as early as possible.
- [ ] Prefer efficient columnar formats such as Parquet or Delta.
- [ ] Avoid reading the same dataset repeatedly.
- [ ] Use partition pruning when storage partition columns are filtered.
- [ ] Check whether the input contains too many small files.
- [ ] Provide an explicit schema when appropriate, instead of repeatedly inferring it.

### Example: Select required columns

Avoid:

```python
df = spark.read.parquet("/data/orders")

df.show()
```

Better:

```python
df = spark.read.parquet("/data/orders") \
    .select("order_id", "customer_id", "amount")
```

### Example: Filter early

```python
df = spark.read.parquet("/data/orders") \
    .filter("order_date >= '2026-01-01'") \
    .select("order_id", "customer_id", "amount")
```

**Why?** Reducing the amount of data early can reduce the work required by later transformations.

Spark SQL's optimizer can push supported filters and column projections closer to the data source.

---

## 4. Step 3: Optimize Transformations

Not all transformations have the same execution cost.

### Checklist

- [ ] Avoid unnecessary transformations.
- [ ] Filter records before expensive operations.
- [ ] Select only required columns.
- [ ] Avoid repeatedly calculating the same expressions.
- [ ] Prefer built-in Spark SQL functions over Python UDFs when possible.
- [ ] Avoid unnecessary `distinct()`, `orderBy()`, and `groupBy()`.
- [ ] Avoid converting large DataFrames to Pandas.
- [ ] Avoid collecting large datasets to the driver.

### Example: Avoid unnecessary columns

```python
from pyspark.sql.functions import col

df = df.select(
    col("customer_id"),
    col("amount")
)
```

### Built-in functions vs Python UDFs

Prefer built-in functions:

```python
from pyspark.sql.functions import upper

df = df.withColumn("name_upper", upper("name"))
```

A Python UDF may introduce additional serialization and execution overhead.

**Interview point:** Built-in Spark SQL functions are generally easier for Spark to optimize than Python UDFs.

---

## 5. Step 4: Optimize Joins

Joins can become expensive when large datasets need to be redistributed.

### Checklist

- [ ] Identify the size of both DataFrames.
- [ ] Filter unnecessary records before joining.
- [ ] Select only required columns before joining.
- [ ] Use broadcast joins when one side is sufficiently small.
- [ ] Check for data skew in join keys.
- [ ] Inspect the physical execution plan.
- [ ] Check shuffle read and shuffle write metrics.
- [ ] Avoid unnecessary joins.

### Example: Broadcast join

```python
from pyspark.sql.functions import broadcast

result = orders.join(
    broadcast(customers),
    on="customer_id",
    how="inner"
)
```

Broadcasting a sufficiently small DataFrame can avoid shuffling the large side solely for the join.

### Join strategies to understand

| Join strategy | Basic idea |
|---|---|
| Broadcast Hash Join | Broadcast a small side to executors |
| Sort Merge Join | Shuffle and sort both sides before merging |
| Shuffle Hash Join | Shuffle both sides and build local hash tables |
| Broadcast Nested Loop Join | Broadcast one side and compare records using nested-loop logic |

**Important:** The best join strategy depends on dataset size, join type, statistics, and available memory.

---

## 6. Step 5: Reduce Shuffle Operations

Shuffle is the redistribution of data across partitions.

It is expensive because it can involve:

- Network communication.
- Disk I/O.
- Serialization and deserialization.
- Memory usage.
- Additional stage boundaries.

### Checklist

- [ ] Avoid unnecessary `repartition()`.
- [ ] Avoid unnecessary `distinct()`.
- [ ] Review expensive `groupBy()` operations.
- [ ] Use broadcast joins where appropriate.
- [ ] Reduce data before shuffle operations.
- [ ] Check shuffle read and write metrics.
- [ ] Avoid repeatedly repartitioning the same DataFrame without a clear reason.

### Common shuffle-related operations

| Operation | Shuffle behavior |
|---|---|
| `groupBy()` | Generally requires shuffle for grouping |
| `distinct()` | Generally requires data redistribution |
| `orderBy()` | Generally requires global ordering and shuffle |
| `repartition()` | Redistributes data through shuffle |
| `coalesce()` | Generally reduces partitions without a full shuffle |
| Broadcast Hash Join | Avoids shuffling the large side solely for the join |

**Remember:** Some operations use partial aggregation or other optimizations to reduce the amount of data shuffled.

---

## 7. Step 6: Handle Data Skew

Data skew occurs when some partitions contain significantly more records than others.

### Checklist

- [ ] Check whether a few keys contain most of the records.
- [ ] Inspect task duration differences in Spark UI.
- [ ] Compare shuffle read sizes across tasks.
- [ ] Check memory and disk spill.
- [ ] Identify hot keys and NULL/default keys.
- [ ] Consider salting for heavily skewed keys.
- [ ] Consider broadcast joins when one side is small.
- [ ] Consider AQE skew join optimization where applicable.
- [ ] Process extremely skewed keys separately when appropriate.

### Example: Identify frequent keys

```python
from pyspark.sql.functions import col

df.groupBy("customer_id") \
    .count() \
    .orderBy(col("count").desc()) \
    .show(20)
```

This helps identify keys with unusually high record counts.

**Important:** Repartitioning by the same skewed key does not necessarily solve data skew.

---

## 8. Step 7: Optimize Partitioning

The number and distribution of partitions affect parallelism and task execution.

### Checklist

- [ ] Check the current number of partitions.
- [ ] Avoid too few partitions.
- [ ] Avoid excessive numbers of tiny partitions.
- [ ] Use `repartition()` when redistribution is necessary.
- [ ] Use `coalesce()` when reducing partitions without a full shuffle is appropriate.
- [ ] Check whether partitions are evenly distributed.
- [ ] Avoid using a highly skewed column as the only partitioning key without considering its distribution.

### Example

```python
print(df.rdd.getNumPartitions())
```

Increase or redistribute partitions:

```python
df = df.repartition(20)
```

Reduce partitions:

```python
df = df.coalesce(5)
```

**Remember:** There is no universal ideal partition count. Choose based on data size, available resources, and observed task performance.

---

## 9. Step 8: Optimize File Sizes

Too many small files can reduce performance.

### Problems caused by small files

- More file listing and metadata operations.
- More file-opening overhead.
- More tasks for small amounts of data.
- Slower query planning and execution in some workloads.

### Checklist

- [ ] Inspect the number of input files.
- [ ] Check average file size.
- [ ] Avoid creating excessive small output files.
- [ ] Choose appropriate execution partitions before writing.
- [ ] Use compaction when supported and appropriate.
- [ ] Avoid unnecessarily high-cardinality storage partition columns.

### Delta Lake example

```sql
OPTIMIZE orders;
```

`OPTIMIZE` can compact small Delta data files.

For suitable workloads, Z-Ordering can improve data skipping:

```sql
OPTIMIZE orders
ZORDER BY (customer_id);
```

**Remember:**

- `OPTIMIZE` is used for file compaction and layout optimization.
- `ZORDER BY` improves data skipping for suitable query patterns.
- They are related but not the same operation.
- Availability and behavior depend on the Databricks/runtime environment.

---

## 10. Step 9: Use Adaptive Query Execution (AQE)

AQE allows Spark SQL to modify parts of the physical execution plan using runtime statistics.

### Checklist

- [ ] Check whether AQE is enabled.
- [ ] Understand the initial physical plan.
- [ ] Inspect the final adaptive plan.
- [ ] Check whether shuffle partition counts are adjusted.
- [ ] Check whether eligible joins are converted to broadcast joins.
- [ ] Check whether skewed partitions are handled.

Check configuration:

```python
spark.conf.get("spark.sql.adaptive.enabled")
```

AQE can help with:

1. Coalescing shuffle partitions.
2. Converting eligible join strategies.
3. Handling skewed partitions in supported shuffled joins.

**Important:** AQE does not automatically solve every performance problem. Its behavior depends on Spark version, configuration, query structure, and available runtime statistics.

---

## 11. Step 10: Optimize Caching and Persistence

Caching can help when the same DataFrame is reused multiple times.

### Checklist

- [ ] Cache only DataFrames that are reused.
- [ ] Avoid caching data used only once.
- [ ] Consider the size of cached data.
- [ ] Choose an appropriate persistence level.
- [ ] Materialize cached data with an action when needed.
- [ ] Release cached data using `unpersist()` when no longer required.
- [ ] Check the Spark UI Storage tab.

### Example

```python
df.cache()

df.count()

# Reuse df in multiple operations

df.unpersist()
```

**Remember:**

- `cache()` is lazy.
- An action triggers computation of the required data.
- Caching consumes storage resources.
- Caching does not automatically make every job faster.

---

## 12. Step 11: Monitor Memory and Spill

Memory pressure can slow down Spark jobs.

### Checklist

- [ ] Check executor memory usage.
- [ ] Check driver memory usage.
- [ ] Inspect memory spill and disk spill metrics.
- [ ] Look for out-of-memory errors.
- [ ] Avoid collecting large datasets to the driver.
- [ ] Avoid unnecessary caching.
- [ ] Investigate large partitions and data skew.
- [ ] Review executor resource configuration when evidence indicates a resource bottleneck.

### Important distinction

| Concept | Meaning |
|---|---|
| Execution memory | Memory used for tasks such as joins and aggregations |
| Storage memory | Memory used for cached and persisted data |
| Memory spill | Intermediate data is moved out of execution memory |
| Disk spill | Intermediate data is written to disk when it cannot remain in memory |

Do not immediately increase executor memory. First identify why the job is consuming excessive memory.

---

## 13. Step 12: Use Spark UI and Explain Plans

Performance tuning requires evidence.

### A. Explain execution plan

```python
df.explain("formatted")
```

Use it to inspect:

- Logical and physical operations.
- Join strategies.
- Exchanges and shuffle boundaries.
- Filters and projections.
- Adaptive execution details when available.

### B. Spark UI

Use Spark UI to inspect:

| Area | What to check |
|---|---|
| Jobs | Job execution time and status |
| Stages | Stage duration and task distribution |
| Tasks | Slow tasks and stragglers |
| SQL | Query execution plan and metrics |
| Storage | Cached DataFrames |
| Executors | Memory, task time, and resource usage |

### Important metrics

- Shuffle read.
- Shuffle write.
- Input size.
- Output size.
- Memory spill.
- Disk spill.
- Task duration.
- Executor utilization.

**Interview point:** Spark UI helps identify the actual bottleneck instead of relying on assumptions.

---

## 14. Practical End-to-End Performance Tuning Example

Imagine an orders pipeline that reads a large dataset, joins customers, aggregates revenue, and writes the result.

### Initial pipeline

```python
orders = spark.read.parquet("/data/orders")
customers = spark.read.parquet("/data/customers")

result = orders.join(
    customers,
    "customer_id",
    "inner"
)

result = result.groupBy("country").sum("amount")

result.write.mode("overwrite").parquet("/output/revenue")
```

### Performance review

1. Read only required columns.
2. Apply filters early if the business requirement allows.
3. Check the sizes of both datasets.
4. Broadcast customers if it is sufficiently small.
5. Check whether the join key is skewed.
6. Inspect the aggregation shuffle.
7. Review partition counts and task distribution.
8. Check the output file sizes.
9. Inspect the physical plan and Spark UI metrics.

### Optimized example

```python
from pyspark.sql.functions import broadcast, col, sum

orders = (
    spark.read.parquet("/data/orders")
    .select("customer_id", "amount", "order_date")
    .filter(col("order_date") >= "2026-01-01")
)

customers = (
    spark.read.parquet("/data/customers")
    .select("customer_id", "country")
)

result = (
    orders.join(
        broadcast(customers),
        on="customer_id",
        how="inner"
    )
    .groupBy("country")
    .agg(sum("amount").alias("total_revenue"))
)

result.write.mode("overwrite").parquet("/output/revenue")
```

**Important:** This example assumes the customers DataFrame is sufficiently small to broadcast. The optimization must be validated using the actual execution plan and runtime metrics.

---

## 15. Common Performance Tuning Mistakes

1. Increasing executor memory without investigating the bottleneck.
2. Increasing partitions without checking task size and overhead.
3. Broadcasting a large DataFrame.
4. Caching every DataFrame.
5. Repartitioning repeatedly.
6. Ignoring data skew.
7. Using Python UDFs when built-in functions are available.
8. Reading unnecessary columns.
9. Collecting large datasets to the driver.
10. Ignoring small files.
11. Assuming AQE fixes every performance problem.
12. Optimizing code without comparing execution metrics.

---

## 16. Interview Questions

**Q1. What is performance tuning in Spark?**

Improving job execution time and resource efficiency by optimizing data reading, transformations, joins, shuffles, partitioning, memory, and file layout.

**Q2. What is the first step in performance tuning?**

Identify the actual bottleneck using the execution plan, Spark UI, and runtime metrics.

**Q3. How can you reduce shuffle?**

Filter and select columns early, avoid unnecessary wide transformations, use suitable broadcast joins, and reduce data before expensive operations.

**Q4. How do you optimize a join?**

Check dataset sizes, filter early, select required columns, broadcast a sufficiently small side, and investigate skew.

**Q5. How do you identify data skew?**

Compare task durations, shuffle read sizes, and spill metrics. A few unusually slow tasks can indicate skew.

**Q6. Why are Python UDFs sometimes slower?**

They can introduce Python execution and serialization overhead and may limit Spark SQL optimizer visibility.

**Q7. When should you cache a DataFrame?**

When it is expensive to compute and reused across multiple operations, provided there are sufficient resources.

**Q8. What is AQE?**

Adaptive Query Execution uses runtime statistics to optimize parts of the physical execution plan.

**Q9. How do you identify a shuffle bottleneck?**

Inspect `Exchange` operations in the physical plan and examine shuffle read/write, task duration, and spill metrics in Spark UI.

**Q10. How do you optimize small files?**

Control output partitions, avoid excessive storage partitioning, and use compaction where supported.

**Q11. Why should you avoid collect()?**

It transfers all records to the driver and can cause driver memory exhaustion.

**Q12. How do you prove that your optimization worked?**

Compare execution time, input/output sizes, shuffle metrics, spill, task distribution, and resource utilization before and after the change.

---

## 17. Quick Performance Tuning Checklist

Use this as a final review before running a production Spark job.

### Data reading
- [ ] Read only required columns.
- [ ] Filter early.
- [ ] Prefer Parquet or Delta where suitable.
- [ ] Check input file sizes.
- [ ] Use partition pruning where possible.

### Transformations
- [ ] Avoid unnecessary operations.
- [ ] Prefer built-in functions.
- [ ] Avoid unnecessary Python UDFs.
- [ ] Avoid large `collect()` operations.
- [ ] Avoid unnecessary sorting and deduplication.

### Joins and shuffle
- [ ] Check join strategies.
- [ ] Broadcast only sufficiently small datasets.
- [ ] Investigate data skew.
- [ ] Reduce data before shuffle.
- [ ] Avoid unnecessary repartitioning.

### Partitioning and files
- [ ] Check partition counts.
- [ ] Check partition distribution.
- [ ] Avoid excessive small files.
- [ ] Use `repartition()` and `coalesce()` appropriately.
- [ ] Consider compaction when appropriate.

### Memory and execution
- [ ] Cache only reused data.
- [ ] Check memory and disk spill.
- [ ] Check executor utilization.
- [ ] Review AQE settings.
- [ ] Avoid unnecessary driver-side processing.

### Monitoring
- [ ] Inspect `explain("formatted")`.
- [ ] Review Spark UI.
- [ ] Compare task durations.
- [ ] Compare shuffle metrics.
- [ ] Measure before and after optimization.

---

## 18. Final Revision: Remember These 10 Points

1. **Filter early:** Reduce data before expensive operations.
2. **Select required columns:** Avoid unnecessary data processing.
3. **Optimize joins:** Use suitable join strategies.
4. **Reduce shuffle:** Avoid unnecessary data redistribution.
5. **Handle skew:** Investigate uneven partition sizes.
6. **Tune partitions:** Balance parallelism and task overhead.
7. **Control file sizes:** Avoid excessive small files.
8. **Use AQE:** Let Spark adapt eligible execution decisions.
9. **Cache wisely:** Cache reused data, not everything.
10. **Monitor first:** Use Spark UI and explain plans before changing configurations.

**Golden rule:** Identify the bottleneck → make one targeted change → measure the result → keep the change only if performance improves.