# Query Performance Optimization in Databricks

## 1. What Is Query Performance Optimization?

Query performance optimization means improving the speed and efficiency of SQL queries and Spark DataFrame operations.

In Databricks, query performance depends on several factors:

- Amount of data being processed.
- Data storage format and file layout.
- Number of partitions.
- Join strategy.
- Shuffle operations.
- Memory and compute resources.
- Query execution plan.
- Data filtering and aggregation.

The goal is to reduce unnecessary data scanning, computation, network transfer, and execution time.

**Simple way to remember:**

Query optimization = Read less data + Move less data + Process efficiently.

## 2. How Spark Executes a Query

Before optimizing a query, understand how Spark processes it.

Consider this query:

```sql
SELECT
    customer_id,
    SUM(amount) AS total_sales
FROM main.silver.orders
WHERE order_date >= '2026-10-01'
GROUP BY customer_id;
```

Conceptually, Spark performs the following operations:

```text
          SQL QUERY
              |
              v
        PARSE SQL
              |
              v
       LOGICAL PLAN
              |
              v
      CATALYST OPTIMIZER
              |
              v
       OPTIMIZED PLAN
              |
              v
       PHYSICAL PLAN
              |
              v
       EXECUTION ENGINE
              |
              v
          RESULTS
```

### Important Components

| Component | Purpose |
|---|---|
| Parser | Converts SQL into a logical representation |
| Analyzer | Resolves tables, columns, and data types |
| Catalyst Optimizer | Applies query optimization rules |
| Physical Planner | Selects physical execution strategies |
| Execution Engine | Executes the selected plan |

**Important:** Spark can optimize queries automatically, but developers still need to write efficient queries and understand the execution plan.

## 3. Use EXPLAIN to Understand Query Plans

`EXPLAIN` helps inspect how Spark plans to execute a query.

Example:

```sql
EXPLAIN FORMATTED
SELECT *
FROM main.silver.orders
WHERE customer_id = 501;
```

It helps identify:

- Table scans.
- Filters.
- Joins.
- Aggregations.
- Exchanges (shuffle operations).
- Sort operations.
- Physical execution operators.

### Common EXPLAIN Modes

| Mode | Purpose |
|---|---|
| `EXPLAIN` | Displays the query plan |
| `EXPLAIN EXTENDED` | Displays logical and physical plan details |
| `EXPLAIN FORMATTED` | Displays a formatted plan with operator details |
| `EXPLAIN COST` | Displays plan statistics and cost information where available |

Example in PySpark:

```python
df.explain()
```

Formatted plan:

```python
df.explain("formatted")
```

**Interview point:** Use EXPLAIN to understand the planned execution, then use Spark UI to compare it with actual runtime behavior.

## 4. Reduce Data Scanning

One of the easiest ways to improve query performance is to avoid reading unnecessary data.

### 4.1 Avoid SELECT *

Bad:

```sql
SELECT *
FROM main.silver.orders;
```

Better:

```sql
SELECT
    order_id,
    customer_id,
    amount
FROM main.silver.orders;
```

Benefits:

- Reads only required columns when projection pushdown is supported.
- Reduces data processing.
- Can reduce memory and I/O usage.

### 4.2 Apply Filters Early

Bad:

```python
df = spark.read.table("main.silver.orders")

result = df.select(
    "order_id",
    "customer_id",
    "amount"
).filter("amount > 1000")
```

Better:

```python
df = spark.read.table("main.silver.orders")

result = df.filter(
    "amount > 1000"
).select(
    "order_id",
    "customer_id",
    "amount"
)
```

Spark's Catalyst optimizer may reorder operations and push filters down automatically when safe and supported.

However, writing queries with clear filters and selecting only required columns makes the intended processing easier to understand.

### 4.3 Predicate Pushdown

Predicate pushdown allows supported filters to be applied closer to the data source.

Example:

```python
df = spark.read.parquet(path)

filtered_df = df.filter(
    "amount > 1000"
)
```

If supported by the data source, Spark may push the filter into the scan operation.

This can reduce unnecessary data reads.

**Important:** Predicate pushdown is source- and expression-dependent. It is not guaranteed for every query.

## 5. Use Partition Pruning and Data Skipping

### 5.1 Partition Pruning

Partition pruning skips storage partitions that do not match the query filter.

Suppose a Delta table is partitioned by `order_date`.

```sql
SELECT *
FROM main.silver.orders
WHERE order_date = '2026-10-30';
```

Databricks can read only the relevant date partition rather than scanning every partition.

### 5.2 Data Skipping

Delta Lake maintains file-level statistics for eligible columns.

These statistics can include:

- Minimum values.
- Maximum values.
- NULL counts.
- Record counts.

Databricks uses these statistics to skip files that cannot contain matching records.

Example:

```sql
SELECT *
FROM main.silver.orders
WHERE customer_id = 501;
```

If a file's statistics show that its customer IDs fall between 600 and 900, that file can be skipped for this filter.

### Difference

| Feature | Partition Pruning | Data Skipping |
|---|---|---|
| Works at | Partition level | File level |
| Uses | Partition column values | File statistics |
| Main purpose | Skip irrelevant partitions | Skip irrelevant files |
| Example | Filter by order date | Filter by customer ID |

**Remember:** Partition pruning and data skipping both reduce scanning, but they work at different levels.

## 6. Optimize Joins

Joins can become expensive when large datasets need to exchange data across executors.

### 6.1 Broadcast Join

If one table is small enough, Spark can broadcast it to the executors.

Example:

```python
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    "customer_id",
    "inner"
)
```

Benefits:

- Can avoid shuffling the large table.
- Can improve join performance.
- Reduces network transfer for suitable workloads.

### 6.2 Sort-Merge Join

Sort-Merge Join is commonly used for large equi-joins.

Conceptual process:

1. Shuffle records by join key.
2. Sort records within partitions.
3. Merge matching keys.

It can be effective for large datasets but may involve expensive shuffle and sort operations.

### 6.3 Filter Before Joining

Example:

```python
filtered_orders = orders.filter(
    "amount > 1000"
)

result = filtered_orders.join(
    customers,
    "customer_id",
    "inner"
)
```

Reducing the amount of data before a join can reduce processing and shuffle overhead.

Catalyst may also push filters through joins when it is safe to do so.

**Important:** Choose join strategies based on table sizes, statistics, join conditions, and actual execution plans. Do not force broadcast joins for large tables.

## 7. Reduce Shuffle Operations

A shuffle redistributes data across partitions, often between executors.

Common operations that can trigger shuffles include:

- `groupBy()`
- `distinct()`
- `repartition()`
- Many joins.
- Global sorting.
- Some window operations.

Example:

```python
result = df.groupBy(
    "customer_id"
).sum("amount")
```

This commonly requires data to be redistributed by `customer_id`.

### Why Are Shuffles Expensive?

- Network data transfer.
- Disk I/O.
- Serialization and deserialization.
- Sorting and aggregation overhead.
- Possible memory pressure and spilling.

### Optimization Techniques

- Filter data before expensive operations.
- Avoid unnecessary `distinct()`.
- Avoid repeated repartitioning.
- Use broadcast joins where suitable.
- Choose appropriate partition counts.
- Handle data skew.

**Interview point:** Reducing unnecessary shuffle is one of the most important Spark performance optimization techniques.

## 8. Handle Data Skew

Data skew occurs when some partitions contain significantly more records than others.

Example:

| customer_id | Number of Records |
|---|---:|
| 101 | 1,000 |
| 102 | 1,200 |
| 103 | 1,100 |
| 999 | 500,000 |

If a join or aggregation distributes data using `customer_id`, the partition containing customer 999 may become much larger.

Possible consequences:

- One or a few slow tasks.
- Uneven executor workload.
- Memory pressure.
- Longer stage execution time.

### Common Solutions

1. Broadcast a suitable small table.
2. Use Adaptive Query Execution (AQE).
3. Apply salting when appropriate.
4. Pre-aggregate data where logically valid.
5. Reconsider partitioning and join strategy.

**Important:** Repartitioning by the same skewed key does not automatically solve data skew.

## 9. Use Adaptive Query Execution (AQE)

AQE allows Spark to adjust parts of a query plan using runtime statistics.

It can help with:

- Converting certain sort-merge joins to broadcast joins.
- Coalescing shuffle partitions.
- Handling certain skewed joins.

Check the configuration:

```python
spark.conf.get(
    "spark.sql.adaptive.enabled"
)
```

AQE is enabled by default in standard Spark SQL configurations from Spark 3.2, although platform defaults and workspace settings can differ.

### Example

Suppose Spark initially plans a Sort-Merge Join because it estimates both tables to be large.

During execution, Spark discovers that one side is small enough for broadcasting.

AQE may convert the join to a Broadcast Hash Join when supported and beneficial.

**Remember:** AQE improves execution decisions, but it does not guarantee that every inefficient query will become optimal.

## 10. Optimize Delta Table Layout

The physical layout of a Delta table can significantly affect query performance.

### 10.1 OPTIMIZE

Compacts smaller files into a more efficient file layout.

```sql
OPTIMIZE main.silver.orders;
```

### 10.2 Z-Ordering

Can improve data skipping for suitable filter columns.

```sql
OPTIMIZE main.silver.orders
ZORDER BY (customer_id);
```

### 10.3 Liquid Clustering

Organizes data around selected clustering columns.

```sql
CREATE TABLE main.silver.orders (
    order_id BIGINT,
    customer_id BIGINT,
    amount DECIMAL(12,2),
    order_date DATE
)
USING DELTA
CLUSTER BY (customer_id, order_date);
```

For supported tables:

```sql
OPTIMIZE main.silver.orders;
```

### 10.4 Avoid Small Files

Too many small files can increase metadata and file-reading overhead.

Consider:

- Optimized writes.
- Auto compaction.
- Appropriate partition counts.
- OPTIMIZE when beneficial.

**Important:** OPTIMIZE and clustering consume compute. Use them when the expected query benefits justify the maintenance cost.

## 11. Choose the Right Number of Partitions

Spark processes data in partitions.

The number of partitions affects parallelism and task size.

Too few partitions:

- Limited parallelism.
- Large tasks.
- Possible memory pressure.

Too many partitions:

- Excessive task scheduling.
- Small tasks.
- Increased overhead.

### repartition()

Increases or changes the number of partitions and generally causes a shuffle.

```python
df_repartitioned = df.repartition(8)
```

Partition by a column:

```python
df_repartitioned = df.repartition(
    8,
    "customer_id"
)
```

### coalesce()

Reduces the number of partitions, usually without a full shuffle.

```python
df_coalesced = df.coalesce(4)
```

**Important:** `coalesce()` is not a general replacement for repartitioning. It can result in uneven partition sizes and does not guarantee a specific number of output files.

## 12. Cache and Persist Only When Needed

Caching stores intermediate data so it can be reused.

Example:

```python
df.cache()

df.count()

result1 = df.filter("amount > 1000")
result2 = df.filter("amount < 100")
```

Caching may help when the same expensive DataFrame is used multiple times.

However, it can consume executor memory and cause memory pressure.

Release cached data when it is no longer needed:

```python
df.unpersist()
```

**Interview point:** Cache only when the benefit of avoiding repeated computation outweighs the memory and storage overhead.

## 13. Use Spark UI for Performance Debugging

Spark UI helps identify actual execution bottlenecks.

Important areas:

| Spark UI Section | What to Check |
|---|---|
| Jobs | Job duration and status |
| Stages | Slow stages and shuffle boundaries |
| Tasks | Uneven task execution times |
| SQL/DataFrame | Query execution details |
| Storage | Cached data |
| Executors | Memory, task, and resource information |
| Environment | Relevant Spark configurations |

### Important Metrics

- Shuffle Read.
- Shuffle Write.
- Task Duration.
- Input Size.
- Output Size.
- Memory Spill.
- Disk Spill.
- GC Time.

### Example: Diagnosing a Slow Query

Suppose a job has 100 tasks.

- 95 tasks finish in 20 seconds.
- 5 tasks take 5 minutes.

This may indicate data skew or uneven partition sizes.

Next steps:

1. Identify the slow tasks.
2. Inspect their input and shuffle sizes.
3. Check whether they process skewed keys.
4. Review the physical plan.
5. Apply an appropriate optimization.
6. Compare execution metrics after the change.

**Important:** A slow stage can have multiple causes. Use metrics and the execution plan before deciding on a fix.

## 14. Practical Example: Optimizing a Slow Orders Query

Suppose an e-commerce company runs this query:

```sql
SELECT
    o.customer_id,
    c.customer_name,
    SUM(o.amount) AS total_sales
FROM main.silver.orders o
JOIN main.silver.customers c
    ON o.customer_id = c.customer_id
WHERE o.order_date >= '2026-10-01'
GROUP BY
    o.customer_id,
    c.customer_name;
```

Potential performance problems:

- Orders table is very large.
- Customer table is small.
- Many unnecessary columns may be read.
- Date filtering may scan unnecessary files.
- Join may involve an expensive shuffle.
- Data may be skewed.

### Step 1: Filter and Select Required Columns

```python
from pyspark.sql import functions as F
from pyspark.sql.functions import broadcast

filtered_orders = (
    spark.read.table("main.silver.orders")
    .filter(
        F.col("order_date") >= "2026-10-01"
    )
    .select("customer_id", "amount")
)
```

### Step 2: Read Only Required Customer Columns

```python
customers = (
    spark.read.table("main.silver.customers")
    .select("customer_id", "customer_name")
)
```

### Step 3: Use a Suitable Join Strategy

If the customer table is small enough:

```python
joined_df = filtered_orders.join(
    broadcast(customers),
    "customer_id",
    "inner"
)
```

### Step 4: Aggregate

```python
result = (
    joined_df
    .groupBy("customer_id", "customer_name")
    .agg(
        F.sum("amount").alias("total_sales")
    )
)
```

### Step 5: Inspect the Plan

```python
result.explain("formatted")
```

### Step 6: Compare Runtime Metrics

Use Spark UI to compare:

- Data scanned.
- Shuffle read and write.
- Number of tasks.
- Stage duration.
- Spill metrics.
- Total execution time.

**Important:** Broadcasting is appropriate only if the customer table is small enough and the join supports the selected strategy. Always validate the actual plan and runtime metrics.

## 15. Common Performance Optimization Mistakes

| Mistake | Correct Understanding |
|---|---|
| Always use the largest cluster | Right-size compute for the workload |
| Use `SELECT *` everywhere | Read only required columns |
| Repartition before every operation | Repartition only when needed |
| Cache every DataFrame | Cache only when data is reused |
| Broadcast every small-looking table | Check actual size and memory requirements |
| Partition by every filter column | Choose layout based on data and query patterns |
| Run OPTIMIZE after every write | Optimize according to workload |
| Assume AQE fixes all skew | AQE handles certain cases, not every skew problem |
| Increase partitions to fix every slow job | First identify the bottleneck |
| Assume EXPLAIN shows actual runtime | Use Spark UI for actual execution metrics |

## 16. Interview Questions and Answers

**Q1. What is query optimization in Spark?**

It is the process of improving query execution by reducing unnecessary scanning, shuffling, computation, and resource usage.

**Q2. What is Catalyst Optimizer?**

Catalyst is Spark SQL's query optimization framework. It applies optimization rules to logical plans and helps generate efficient execution plans.

**Q3. What is EXPLAIN?**

EXPLAIN displays a query's execution plan. It helps identify scans, filters, joins, aggregations, exchanges, and physical operators.

**Q4. What is predicate pushdown?**

It pushes supported filters closer to the data source so unnecessary data can be avoided during reading.

**Q5. What is partition pruning?**

It skips storage partitions that do not match the query filter.

**Q6. What is data skipping?**

It uses file-level statistics to avoid reading files that cannot contain matching records.

**Q7. How can you reduce shuffle operations?**

Filter early, avoid unnecessary distinct and repartition operations, use suitable broadcast joins, and review the execution plan.

**Q8. What is a broadcast join?**

It distributes a suitable small table to worker nodes so a large table may not need to be shuffled for the join.

**Q9. What is data skew?**

Data skew occurs when some partitions contain significantly more data than others, causing uneven task execution.

**Q10. What is AQE?**

Adaptive Query Execution uses runtime statistics to adjust parts of the query plan, such as join strategies, shuffle partition counts, and certain skewed joins.

**Q11. How do you identify a slow Spark query?**

Use EXPLAIN to inspect the plan and Spark UI to analyze stage duration, task distribution, shuffle metrics, input size, and memory or disk spilling.

**Q12. What is the difference between repartition and coalesce?**

`repartition()` generally causes a shuffle and can increase or decrease partitions. `coalesce()` usually reduces partitions without a full shuffle.

**Q13. How does OPTIMIZE improve query performance?**

It reorganizes Delta data files, commonly compacting small files and potentially improving read efficiency.

**Q14. How would you optimize a slow join between two large tables?**

I would inspect the physical plan, filter unnecessary records, select required columns, check join-key distribution, investigate skew, review shuffle metrics, and choose an appropriate join strategy.

**Q15. Does caching always improve performance?**

No. Caching helps when data is reused, but unnecessary caching consumes memory and can increase resource pressure.

**Q16. What is the difference between EXPLAIN and Spark UI?**

EXPLAIN shows the planned execution strategy. Spark UI provides actual execution information and runtime metrics.

## 17. Quick Revision

| Technique | Main Purpose |
|---|---|
| EXPLAIN | Inspect query execution plan |
| Catalyst | Optimize logical query plans |
| Projection pushdown | Read only required columns |
| Predicate pushdown | Apply supported filters near the source |
| Partition pruning | Skip irrelevant storage partitions |
| Data skipping | Skip irrelevant data files |
| Broadcast join | Reduce shuffle for suitable joins |
| AQE | Adjust plans using runtime statistics |
| Data skew handling | Balance uneven workloads |
| OPTIMIZE | Improve Delta file layout |
| Z-Ordering | Improve data skipping for selected columns |
| Liquid Clustering | Flexible clustering-based data layout |
| Cache | Reuse expensive intermediate results |
| Spark UI | Diagnose actual runtime bottlenecks |

### One-Minute Interview Answer

Query performance optimization in Databricks means reducing unnecessary data scanning, shuffling, and computation.

First, I inspect the query plan using EXPLAIN to understand scans, filters, joins, aggregations, and shuffle operations.

Then I optimize data reading by selecting required columns, applying filters, and using partition pruning or data skipping where applicable.

For joins, I evaluate broadcast joins for suitable small tables and investigate data skew when task execution is uneven. I also consider AQE, appropriate partition counts, and Delta table layout optimization.

Finally, I use Spark UI to analyze actual runtime metrics such as shuffle read, shuffle write, task duration, data scanned, and spill. I compare performance before and after each change.

## 18. Self-Assessment

- [ ] Explain how Spark executes a SQL query.
- [ ] Explain Catalyst Optimizer.
- [ ] Use EXPLAIN FORMATTED.
- [ ] Differentiate logical and physical plans.
- [ ] Explain predicate pushdown.
- [ ] Explain partition pruning and data skipping.
- [ ] Explain broadcast and sort-merge joins.
- [ ] Identify expensive shuffle operations.
- [ ] Explain data skew and its solutions.
- [ ] Explain Adaptive Query Execution.
- [ ] Explain how Delta file layout affects queries.
- [ ] Differentiate repartition and coalesce.
- [ ] Explain when caching is useful.
- [ ] Use Spark UI to identify performance bottlenecks.
- [ ] Design an optimization approach for a slow production query.

**Interview readiness:** You should be able to take a slow SQL or PySpark query, inspect its execution plan, identify the bottleneck, explain the optimization, and validate the improvement using Spark UI metrics.