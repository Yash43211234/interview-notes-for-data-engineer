# Cost Optimization in Databricks

## 1. What Is Cost Optimization?

Cost optimization in Databricks means reducing the cost of data processing, storage, and compute resources while maintaining acceptable performance and reliability.

Databricks workloads consume compute resources to perform operations such as:

- Data ingestion.
- Data transformation.
- Batch processing.
- Streaming.
- SQL analytics.
- Machine learning.

The goal is not simply to use the cheapest compute. It is to achieve the required performance at a reasonable overall cost.

**Simple way to remember:**

Cost optimization = Efficient compute + Efficient storage + Efficient queries + Controlled usage.

## 2. Understand Databricks Cost Components

Databricks costs can include multiple components.

| Component | Meaning |
|---|---|
| Databricks DBUs | Databricks usage units consumed by workloads |
| Cloud infrastructure | Underlying virtual machines and related cloud resources |
| Storage | Cloud storage used for data and files |
| Network | Data transfer and network-related charges |
| Other services | Additional cloud or platform services used by the workload |

### What Is a DBU?

A **DBU (Databricks Unit)** is a normalized unit of processing capability used to measure Databricks service usage.

The number of DBUs consumed depends on factors such as:

- Compute type.
- Workload.
- Runtime configuration.
- Duration of usage.
- Databricks pricing model.

**Important:** DBU charges and underlying cloud VM charges are different. Total cost may include both, depending on the deployment and pricing arrangement.

Exact pricing varies by cloud provider, region, compute type, Databricks plan, and contract.

## 3. Major Cost Optimization Techniques

### 3.1 Choose the Right Compute

Selecting the correct compute configuration is one of the most important cost optimization decisions.

Avoid selecting unnecessarily large clusters for small workloads.

Consider:

- Number of workers.
- Worker instance type.
- Driver instance type.
- Memory requirements.
- CPU requirements.
- Workload duration.
- Parallel processing requirements.

**Example:**

Suppose a daily transformation processes a relatively small dataset.

Using a very large cluster may increase compute cost without providing a meaningful performance improvement.

A smaller cluster may be sufficient.

**Interview point:** Always choose compute based on workload requirements, not simply the largest available configuration.

### 3.2 Use Autoscaling

Autoscaling automatically adjusts the number of workers based on workload demand, within configured limits.

For example:

```text
Minimum Workers = 2
Maximum Workers = 8

Low Workload:
2 Workers

High Workload:
6 Workers

Workload Completed:
Workers can scale down
```

Benefits:

- Reduces unnecessary worker usage.
- Handles changing workloads.
- Avoids maintaining maximum capacity continuously.

**Important:** Autoscaling does not guarantee lower cost. Scaling behavior, startup time, workload characteristics, and minimum worker settings affect actual savings.

### 3.3 Enable Auto Termination

Auto termination shuts down eligible interactive compute after a configured period of inactivity.

Example:

```text
Auto Termination = 15 Minutes

No Activity
     |
     v
Idle Timeout Reached
     |
     v
Compute Terminates
```

Benefits:

- Prevents forgotten clusters from running.
- Reduces unnecessary compute usage.
- Helps control development costs.

**Interview point:** Auto termination is especially useful for interactive development compute.

Job compute generally has a different lifecycle because it is managed around job execution.

### 3.4 Use Job Compute for Scheduled Workloads

Databricks supports different compute options, including:

- All-purpose compute.
- Job compute.
- SQL warehouses.
- Serverless compute, where available.

For scheduled ETL workloads, job-oriented compute can be more suitable than keeping an interactive cluster running.

Example:

```text
Daily ETL Job
      |
      v
Start Job Compute
      |
      v
Process Data
      |
      v
Write Delta Tables
      |
      v
Job Completes
      |
      v
Compute Terminates
```

Benefits:

- Compute runs for the workload duration.
- Reduces idle interactive compute usage.
- Separates development and production workloads.

Actual cost depends on the compute configuration, runtime, and pricing model.

### 3.5 Use Photon Where Beneficial

Photon is Databricks' native vectorized query engine.

It accelerates supported workloads by improving execution efficiency.

Potential benefits:

- Faster SQL queries.
- Faster supported DataFrame operations.
- Better CPU utilization for supported operations.
- Reduced processing time for suitable workloads.

**Important:** Photon is not guaranteed to accelerate every operation. Its availability and pricing depend on the compute type and Databricks environment.

Evaluate performance and total cost rather than enabling it without measurement.

## 4. Optimize Data Processing

Reducing unnecessary processing can lower both execution time and compute consumption.

### 4.1 Avoid Unnecessary Data Reads

Read only the columns and records required for the transformation.

Bad approach:

```python
df = spark.read.table("main.silver.orders")

df.show()
```

Better approach:

```python
df = (
    spark.read.table("main.silver.orders")
    .select("order_id", "customer_id", "amount")
    .filter("amount > 100")
)
```

Benefits:

- Reduces unnecessary data processing.
- Can reduce data scanned.
- Allows Spark to use projection and predicate pushdown where supported.

### 4.2 Use Predicate Pushdown

Predicate pushdown allows supported filters to be applied closer to the data source.

Example:

```python
df = spark.read.parquet(path)

filtered_df = df.filter(
    "amount > 1000"
)
```

When supported by the source and query plan, Spark can reduce the amount of data read.

**Important:** Predicate pushdown is not guaranteed for every source or expression. Check the execution plan when performance matters.

### 4.3 Use Partition Pruning

Partition pruning avoids scanning partitions that do not match the query filter.

Suppose a Delta table is partitioned by `order_date`.

Query:

```sql
SELECT *
FROM orders
WHERE order_date = '2026-10-30';
```

Spark may read only the relevant date partition instead of scanning all partitions.

Benefits:

- Reduces unnecessary data scanning.
- Improves query performance.
- Can reduce compute consumption.

**Important:** Partitioning should be based on query patterns and data volume. High-cardinality partition columns can create too many small directories and files.

### 4.4 Avoid Unnecessary Shuffles

Shuffle operations move data between partitions and can be expensive.

Common shuffle operations:

- `groupBy()`
- `distinct()`
- `repartition()`
- Many joins
- Global sorting

Example:

```python
# Can trigger a shuffle
result = df.groupBy("customer_id").count()
```

Optimization techniques:

- Filter data before joins and aggregations.
- Avoid unnecessary `distinct()`.
- Use broadcast joins for suitable small tables.
- Avoid repeated repartitioning.
- Choose appropriate partition counts.

### 4.5 Use Broadcast Joins

If one DataFrame is small enough, broadcasting it can avoid shuffling the large DataFrame for a suitable join.

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

- Can reduce shuffle overhead.
- Can improve join performance.
- Can reduce execution time and compute consumption.

**Important:** Broadcasting a table that is too large can create memory pressure or failures. Check table size, statistics, and the actual execution plan.

### 4.6 Avoid Unnecessary Caching

Caching stores intermediate data so it can be reused without recomputing it.

Example:

```python
df.cache()

df.count()
```

Caching can help when the same DataFrame is reused multiple times.

However, unnecessary caching can:

- Consume executor memory.
- Increase memory pressure.
- Cause disk spilling.
- Reduce resources available to other operations.

Use caching only when repeated computation justifies its cost.

Release cached data when it is no longer needed:

```python
df.unpersist()
```

**Remember:** Caching is an optimization technique, not a default requirement for every DataFrame.

## 5. Optimize Delta Lake Storage

Storage layout affects query performance and the amount of compute required.

### 5.1 Use OPTIMIZE

Delta tables may contain many small files due to frequent writes.

Small files increase file-management overhead and can slow down queries.

`OPTIMIZE` compacts smaller files into larger files.

Example:

```sql
OPTIMIZE main.silver.orders;
```

Benefits:

- Reduces small-file overhead.
- Improves read performance for suitable workloads.
- Can reduce query execution time.

**Important:** OPTIMIZE itself consumes compute resources. Run it when the expected future query benefits justify its cost.

### 5.2 Use Z-Ordering Where Appropriate

Z-Ordering reorganizes data to improve data skipping for selected columns.

Example:

```sql
OPTIMIZE main.silver.orders
ZORDER BY (customer_id);
```

Useful when queries frequently filter on the selected columns.

Benefits:

- Can improve data skipping.
- Can reduce the amount of data scanned.
- Can improve query performance.

**Important:** Z-Ordering is not the same as partitioning. Its usefulness depends on data distribution and query patterns. It is not required for every table.

### 5.3 Choose Appropriate Partition Columns

Partitioning organizes data into separate storage partitions based on column values.

Example:

```python
df.write \
    .format("delta") \
    .partitionBy("order_date") \
    .mode("append") \
    .saveAsTable("main.silver.orders")
```

Good partitioning can improve data skipping through partition pruning.

Avoid:

- High-cardinality partition columns.
- Too many tiny partitions.
- Partitioning small tables without a clear benefit.

**Important:** Partitioning is a storage-layout decision. It is not the same as Spark execution repartitioning.

### 5.4 Use VACUUM Carefully

`VACUUM` removes eligible unreferenced data files from a Delta table after the retention period.

Example:

```sql
VACUUM main.silver.orders;
```

Potential benefits:

- Removes obsolete data files.
- Reduces unnecessary storage consumption.

**Important:**

- VACUUM does not directly optimize query execution.
- It permanently removes eligible files.
- It can affect the ability to access older table versions through Time Travel.
- Retention settings and safety checks must be understood before running it.

Do not reduce retention periods or disable safety checks just to save storage without understanding the recovery requirements.

## 6. Optimize File Size and Write Operations

Too many small files can increase read overhead and metadata operations.

Common causes:

- Frequent small batch writes.
- Excessive Spark partitions.
- Over-partitioning by high-cardinality columns.
- Streaming workloads producing frequent small output files.

Optimization techniques:

- Use appropriate write parallelism.
- Avoid unnecessary repartitioning.
- Use optimized writes where supported.
- Use auto compaction where supported.
- Run OPTIMIZE when appropriate.

Example:

```python
df.write \
    .format("delta") \
    .mode("append") \
    .saveAsTable("main.silver.orders")
```

Databricks may provide optimized write and auto-compaction features depending on the table, runtime, and configuration.

There is no universal ideal file size. Choose a target based on workload, file format, storage, and platform behavior.

## 7. Optimize SQL Warehouses

SQL warehouses provide compute for SQL queries and BI workloads.

Cost optimization techniques include:

1. Select an appropriate warehouse size.
2. Use auto stop to avoid unnecessary idle usage.
3. Avoid running multiple unnecessary warehouses.
4. Monitor query execution time.
5. Optimize frequently executed SQL queries.
6. Use serverless options when available and economically suitable.

Example query:

```sql
SELECT
    order_date,
    SUM(amount) AS total_sales
FROM main.gold.orders
WHERE order_date >= '2026-10-01'
GROUP BY order_date;
```

Avoid selecting unnecessary columns or scanning the entire table when filters can reduce the data processed.

**Important:** SQL warehouse sizing, scaling, auto stop, and serverless pricing vary by configuration and cloud environment.

## 8. Monitor Databricks Usage and Costs

Cost optimization requires monitoring actual resource usage.

Useful areas to monitor:

| Area | What to Check |
|---|---|
| Compute | Cluster and warehouse usage |
| DBUs | Databricks usage consumption |
| Job runs | Execution duration and frequency |
| Spark UI | Stages, tasks, shuffles, spills |
| Query history | Slow and expensive SQL queries |
| Storage | Table and file growth |
| Cloud billing | Infrastructure and network charges |

### Important Monitoring Tools

**1. Databricks Compute**

Check:

- Running clusters.
- Cluster size.
- Worker count.
- Idle time.
- Runtime duration.

**2. Spark UI**

Check:

- Slow stages.
- Shuffle read and write.
- Task duration.
- Data skew.
- Memory usage.
- Spill metrics.

**3. Query History**

Identify:

- Frequently executed queries.
- Long-running queries.
- Expensive scans.
- Inefficient joins.

**4. System Tables and Billing Data**

Where available and enabled, Databricks system tables can help analyze usage and billing information.

For example, usage records can be queried to understand consumption over time. Availability and exact schemas depend on the workspace and platform configuration.

## 9. Practical End-to-End Cost Optimization Example

Suppose a company processes 500 GB of order data every day.

Initial problems:

- Large all-purpose cluster runs continuously.
- Unnecessary columns are read.
- Many small Delta files are created.
- Large joins cause expensive shuffles.
- Old data files consume storage.

### Before Optimization

```text
Large Interactive Cluster
          |
          v
Read Entire Dataset
          |
          v
Large Shuffle Operations
          |
          v
Many Small Files
          |
          v
Long Execution Time
          |
          v
Higher Compute Cost
```

### After Optimization

```text
Scheduled Job Compute
          |
          v
Incremental Data Ingestion
          |
          v
Read Required Columns
          |
          v
Filter Early
          |
          v
Optimize Join Strategy
          |
          v
Efficient Delta File Layout
          |
          v
Monitor Usage and Performance
```

### Improvements Applied

| Problem | Optimization |
|---|---|
| Compute runs continuously | Use scheduled job compute |
| Unnecessary idle usage | Configure auto termination where applicable |
| Excessive compute size | Right-size workers |
| Unnecessary data scanning | Select required columns and filter early |
| Expensive joins | Use suitable broadcast joins |
| Too many small files | Optimize file layout |
| Slow repeated queries | Review partition pruning and data skipping |
| Unnecessary storage usage | Apply suitable retention and VACUUM policies |

**Expected result:** Lower resource consumption and potentially faster processing. Actual savings must be measured because some optimization operations also consume compute.

## 10. Cost Optimization Checklist

### Compute

- [ ] Choose the right compute type.
- [ ] Right-size driver and worker instances.
- [ ] Use autoscaling where appropriate.
- [ ] Configure auto termination for eligible compute.
- [ ] Prefer job-oriented compute for suitable scheduled workloads.
- [ ] Evaluate Photon for supported workloads.

### Data Processing

- [ ] Read only required columns.
- [ ] Filter data early.
- [ ] Avoid unnecessary shuffles.
- [ ] Use broadcast joins when appropriate.
- [ ] Avoid unnecessary caching.
- [ ] Avoid unnecessary repartitioning.
- [ ] Use incremental processing where appropriate.

### Storage

- [ ] Monitor small files.
- [ ] Use OPTIMIZE when beneficial.
- [ ] Evaluate Z-Ordering for suitable query patterns.
- [ ] Choose appropriate partition columns.
- [ ] Apply VACUUM safely.
- [ ] Monitor storage growth.

### Monitoring

- [ ] Review compute usage.
- [ ] Monitor DBU consumption.
- [ ] Check job execution time.
- [ ] Analyze Spark UI.
- [ ] Review SQL query history.
- [ ] Compare performance before and after optimization.

## 11. Common Interview Questions

**Q1. What is cost optimization in Databricks?**

It is the process of reducing compute, storage, and processing costs while maintaining required performance and reliability.

**Q2. What is a DBU?**

A DBU is a Databricks Unit, used to measure Databricks service usage. It is separate from the underlying cloud infrastructure cost.

**Q3. How can you reduce Databricks compute costs?**

Choose the right compute size, use suitable job compute, configure autoscaling and auto termination, and avoid unnecessary processing.

**Q4. What is the benefit of autoscaling?**

It adjusts worker capacity based on workload demand, helping avoid unnecessary provisioned capacity.

**Q5. What is auto termination?**

It automatically terminates eligible compute after a configured period of inactivity.

**Q6. Why should we avoid unnecessary caching?**

Caching consumes memory and can create memory pressure. It should be used when repeated DataFrame computation justifies it.

**Q7. How does partition pruning reduce cost?**

It avoids reading irrelevant partitions, potentially reducing data scanning and query execution time.

**Q8. How does OPTIMIZE help?**

It compacts small Delta files, reducing file-management overhead and potentially improving query performance.

**Q9. What is the difference between OPTIMIZE and VACUUM?**

OPTIMIZE reorganizes data files to improve layout. VACUUM removes eligible unreferenced files to reclaim storage.

**Q10. How does a broadcast join improve performance?**

It distributes a suitable small table to worker nodes, potentially avoiding a shuffle of the large table.

**Q11. Does Photon always reduce cost?**

No. Photon can accelerate supported workloads, but actual cost depends on workload performance, compute configuration, and pricing.

**Q12. How do you identify an expensive Spark job?**

Use Spark UI to inspect stage duration, shuffle metrics, task distribution, memory usage, and spill. Also review job history and compute usage.

**Q13. How would you optimize a slow and expensive ETL pipeline?**

First identify the bottleneck using Spark UI and query plans. Then review compute sizing, data scanning, joins, shuffles, partitioning, file layout, and unnecessary transformations. Measure the result after each change.

**Q14. Is the largest cluster always the fastest and cheapest?**

No. A larger cluster may finish sooner but can cost more. The correct configuration balances execution time and total cost.

## 12. Quick Revision

| Technique | Main Benefit |
|---|---|
| Right-sizing | Avoids oversized compute |
| Autoscaling | Adjusts worker capacity |
| Auto termination | Reduces idle compute usage |
| Job compute | Suitable lifecycle for scheduled workloads |
| Photon | Accelerates supported workloads |
| Predicate pushdown | Reduces unnecessary reads |
| Partition pruning | Skips irrelevant partitions |
| Broadcast join | Can reduce shuffle overhead |
| Caching | Avoids repeated computation when useful |
| OPTIMIZE | Compacts small Delta files |
| Z-Ordering | Improves data skipping for suitable queries |
| VACUUM | Removes eligible obsolete data files |
| Monitoring | Identifies cost and performance bottlenecks |

### One-Minute Interview Answer

Cost optimization in Databricks means reducing compute and storage expenses while maintaining performance and reliability.

I would start by choosing the right compute configuration, using autoscaling where appropriate, enabling auto termination for eligible compute, and using job compute for suitable scheduled workloads.

At the processing level, I would reduce unnecessary data reads, filter early, avoid unnecessary shuffles, use broadcast joins for suitable small tables, and cache only when data is reused.

For Delta Lake, I would monitor small files, use OPTIMIZE when beneficial, evaluate Z-Ordering for query patterns, and apply VACUUM carefully.

Finally, I would use Spark UI, query history, job metrics, and billing information to identify bottlenecks and measure whether the optimizations actually reduce total cost.

## 13. Self-Assessment

- [ ] Explain what a DBU is.
- [ ] Differentiate DBU charges and cloud infrastructure charges.
- [ ] Explain autoscaling and auto termination.
- [ ] Differentiate all-purpose and job compute.
- [ ] Explain how Photon can improve performance.
- [ ] Explain predicate pushdown and partition pruning.
- [ ] Explain how broadcast joins reduce shuffle overhead.
- [ ] Explain why unnecessary caching is harmful.
- [ ] Differentiate OPTIMIZE, Z-Ordering, and VACUUM.
- [ ] Explain how small files affect performance.
- [ ] Identify expensive operations using Spark UI.
- [ ] Explain how to optimize an expensive ETL pipeline.

**Interview readiness:** You should be able to explain at least five compute and five processing/storage optimization techniques, and describe how you would measure their impact on cost.