# Join Optimization in Spark

## 1. What Is Join Optimization?

Join optimization is the process of improving how Spark combines two or more datasets.

A poorly optimized join can cause:
- Excessive data shuffling.
- High memory consumption.
- Long-running tasks.
- Data skew.
- Disk spilling.
- Increased execution time.

The goal is to choose an efficient join strategy and reduce unnecessary data movement.

## 2. Why Are Joins Expensive in Spark?

Consider two DataFrames:

**Customers**

| customer_id | customer_name |
|---|---|
| 1 | Amit |
| 2 | Rahul |
| 3 | Priya |

**Orders**

| order_id | customer_id | amount |
|---|---|---:|
| 101 | 1 | 500 |
| 102 | 2 | 700 |
| 103 | 1 | 300 |

Join them using:

```python
result = customers.join(
    orders,
    on="customer_id",
    how="inner"
)
```

Spark needs to match records using `customer_id`.

If the matching records are distributed across different partitions, Spark may need to move data between executors. This movement is called **shuffling**.

Shuffling can involve:
- Network communication.
- Serialization and deserialization.
- Disk I/O.
- Sorting and memory usage.

However, not every join requires shuffling both datasets. The physical join strategy determines how Spark executes the operation.

## 3. Understand the Main Join Strategies

Spark can use different physical join strategies depending on data size, statistics, configuration, join type, and query conditions.

### A. Broadcast Hash Join (BHJ)

A Broadcast Hash Join is useful when one dataset is small enough to be broadcast to the executors.

Example:

```python
from pyspark.sql.functions import broadcast

result = orders.join(
    broadcast(customers),
    on="customer_id",
    how="inner"
)
```

**How it works:**

1. Spark collects the small dataset for broadcasting.
2. It distributes a copy to the executors.
3. Each executor builds a hash table from the broadcast dataset.
4. The larger dataset is joined locally against that hash table.

**Advantages:**
- Avoids shuffling the large side for the join.
- Often faster when one side is small.
- Reduces network movement compared with shuffling both sides.

**Limitations:**
- The broadcast dataset must fit comfortably in executor memory.
- Broadcasting a large dataset can cause memory pressure or failures.
- Broadcast preparation and distribution still have a cost.

### B. Sort Merge Join (SMJ)

Sort Merge Join is commonly used for large datasets.

**How it works:**

1. Spark redistributes both datasets by their join keys when required.
2. It sorts records within the resulting partitions.
3. It merges the sorted records to find matching keys.

Example:

```python
result = orders.join(
    customers,
    on="customer_id",
    how="inner"
)
```

Spark may select Sort Merge Join automatically when appropriate.

**Advantages:**
- Works well for large datasets.
- Does not require building one entire side as an in-memory hash table.
- Suitable for many large equi-joins.

**Limitations:**
- Shuffling can be expensive.
- Sorting adds processing overhead.
- Data skew can cause uneven task execution.

### C. Shuffle Hash Join

In a Shuffle Hash Join:

1. Both datasets are shuffled by the join key.
2. Spark builds a hash table from one side within each partition.
3. It probes that hash table using records from the other side.

It can be useful when the build side is small enough within each partition.

**Limitation:** Building hash tables can create memory pressure if the build side is too large.

### Quick Comparison

| Strategy | Main idea | Typical use |
|---|---|---|
| Broadcast Hash Join | Broadcast small side | Small dimension + large fact |
| Sort Merge Join | Shuffle, sort, merge | Large datasets |
| Shuffle Hash Join | Shuffle, build local hash table | Suitable smaller build side per partition |
| Broadcast Nested Loop Join | Broadcast one side and compare records | Some non-equi joins |

The actual strategy depends on the query and Spark's planning decisions.

## 4. Important Join Optimization Techniques

### A. Use Broadcast Join for Small Tables

If one table is small, broadcasting it can avoid shuffling the larger table for the join.

```python
from pyspark.sql.functions import broadcast

result = fact_sales.join(
    broadcast(dim_customer),
    on="customer_id",
    how="inner"
)
```

Typical example:

- Fact table: 100 GB
- Dimension table: 5 MB

A broadcast join may be appropriate, assuming the dimension table fits comfortably in memory after being prepared for broadcast.

**Important:** Do not broadcast a table just because its stored file size looks small. Consider its in-memory representation and the memory available to executors.

### B. Filter Data Before Joining

Avoid joining unnecessary records.

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

Filtering early can reduce the amount of data processed by the join.

Spark's Catalyst optimizer may push filters down automatically when it is safe and possible. Writing clear queries still helps make the intended logic obvious.

### C. Select Only Required Columns

Avoid carrying unnecessary columns through a join.

Less efficient:

```python
result = orders.join(
    customers,
    on="customer_id",
    how="inner"
)
```

Better:

```python
from pyspark.sql.functions import broadcast

small_customers = customers.select(
    "customer_id",
    "customer_name"
)

selected_orders = orders.select(
    "order_id",
    "customer_id",
    "amount"
)

result = selected_orders.join(
    broadcast(small_customers),
    on="customer_id",
    how="inner"
)
```

This reduces the amount of data that needs to be processed and can make broadcasting more effective.

### D. Handle Data Skew

Data skew occurs when some join keys contain significantly more records than others.

Example:

| customer_id | Number of records |
|---|---:|
| 1 | 10,000 |
| 2 | 12,000 |
| 3 | 15,000 |
| 999 | 5,000,000 |

If one key has millions of records, the corresponding join partition may become much larger than the others.

Consequences:
- One or more tasks run much longer.
- Some executors finish early while others remain busy.
- Memory usage and disk spilling may increase.

Possible solutions:

1. Use Adaptive Query Execution (AQE) skew join optimization.
2. Use salting for suitable skewed keys.
3. Broadcast a small side when appropriate.
4. Separate hot keys and process them independently.
5. Pre-aggregate data when the business logic allows it.

Repartitioning by the same skewed key alone does not solve the problem.

### E. Use Appropriate Join Keys

Join on the correct columns and avoid unnecessary expressions.

Prefer:

```python
result = orders.join(
    customers,
    on="customer_id",
    how="inner"
)
```

Be careful with transformations on join keys:

```python
from pyspark.sql.functions import lower

result = orders.join(
    customers,
    lower(orders.email) == lower(customers.email),
    "inner"
)
```

Expressions can affect optimization opportunities. If normalized join keys are used frequently, consider creating and maintaining normalized columns in the data pipeline.

### F. Avoid Unnecessary Joins

Before joining, ask:

- Do I need columns from both datasets?
- Can I filter or aggregate first?
- Can I use a semi join or anti join instead?
- Is the join required for the final output?

For example, if you only need orders with matching customers, an inner join may be unnecessary if you only need to check whether a customer exists.

Use a left semi join:

```python
result = orders.join(
    customers.select("customer_id"),
    on="customer_id",
    how="left_semi"
)
```

A left semi join returns matching rows from the left DataFrame without adding columns from the right DataFrame.

### G. Use Accurate Table Statistics

Spark uses statistics to estimate data sizes and choose physical execution plans.

Accurate statistics can help Spark select an appropriate join strategy.

For supported catalog tables, statistics can be collected using SQL commands such as:

```sql
ANALYZE TABLE sales COMPUTE STATISTICS;
```

For column statistics:

```sql
ANALYZE TABLE sales
COMPUTE STATISTICS FOR COLUMNS customer_id;
```

Availability and behavior depend on the catalog, table type, and Spark environment.

## 5. Adaptive Query Execution (AQE)

AQE allows Spark to adjust parts of a query plan using runtime statistics.

It can improve joins by:

1. Converting an eligible sort merge join into a broadcast hash join.
2. Handling skewed partitions in supported shuffled joins.
3. Coalescing small shuffle partitions.

Example configuration:

```python
spark.conf.set(
    "spark.sql.adaptive.enabled",
    "true"
)
```

For skew join optimization:

```python
spark.conf.set(
    "spark.sql.adaptive.skewJoin.enabled",
    "true"
)
```

AQE is enabled by default in standard Spark SQL configurations from Spark 3.2 onward, but platform settings may differ.

**Important:** AQE does not guarantee that every inefficient join will become efficient. It works within the operations and conditions it supports.

## 6. Join Hints

Join hints can influence Spark's physical join strategy.

### Broadcast Hint

```python
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    on="id",
    how="inner"
)
```

### SQL Broadcast Hint

```sql
SELECT /*+ BROADCAST(c) */
    o.order_id,
    c.customer_name
FROM orders o
JOIN customers c
    ON o.customer_id = c.customer_id;
```

Other SQL hints include:

- `MERGE`
- `SHUFFLE_HASH`
- `SHUFFLE_REPLICATE_NL`

**Important:** Hints influence the optimizer but do not guarantee that a particular strategy will be used in every situation. Spark may ignore an unsupported or inapplicable hint.

Use hints only after inspecting the execution plan and understanding the data.

## 7. How to Inspect Join Execution Plans

Use:

```python
result.explain("formatted")
```

Look for physical operators such as:

| Physical operator | Meaning |
|---|---|
| BroadcastHashJoin | Broadcast hash join |
| SortMergeJoin | Sort merge join |
| ShuffledHashJoin | Shuffle hash join |
| BroadcastNestedLoopJoin | Broadcast nested loop join |
| Exchange | Often indicates data redistribution or another exchange operation |
| Sort | Sorting operation |

Example:

```text
== Physical Plan ==
AdaptiveSparkPlan
+- BroadcastHashJoin
   :- Scan ExistingRDD
   +- BroadcastExchange
      +- Scan ExistingRDD
```

This indicates that Spark is using a broadcast hash join.

An `Exchange` often indicates a shuffle, but interpret it in context rather than assuming every exchange is necessarily an expensive shuffle.

## 8. Use Spark UI to Find Join Bottlenecks

The Spark UI helps identify why a join is slow.

Check:

**1. Stages**
- Which stage takes the longest?
- Are there unusually long-running tasks?

**2. Shuffle Read and Write**
- Is a large amount of data being shuffled?
- Is shuffle read uneven across tasks?

**3. Spill**
- Are tasks spilling data to disk?
- Could memory pressure be affecting execution?

**4. Task Duration**
- Are most tasks finishing quickly while a few take much longer?
- Could this indicate skew?

**5. SQL Tab**
- Which join strategy was selected?
- What does the physical plan show?
- Did AQE change the plan at runtime?

Do not assume that a slow join is always caused by shuffling. Data skew, file scanning, poor statistics, and memory pressure can also contribute.

## 9. Practical Interview Scenario

**Scenario:**

You have two datasets:

- Sales fact table: 100 GB
- Customer dimension table: 20 MB

Your join is taking too long.

How would you optimize it?

**Step-by-step approach:**

1. Inspect the physical plan using `explain("formatted")`.
2. Check whether Spark is using Sort Merge Join or Broadcast Hash Join.
3. If the customer table is genuinely small enough, try broadcasting it.
4. Select only the required customer columns.
5. Filter unnecessary sales records before joining.
6. Check whether the join key has data skew.
7. Review shuffle metrics, task durations, and spill in Spark UI.
8. Check whether AQE is enabled.
9. Compare execution time and resource usage after changes.

Example:

```python
from pyspark.sql.functions import broadcast

filtered_sales = sales.filter(
    "sale_status = 'COMPLETED'"
).select(
    "sale_id",
    "customer_id",
    "amount"
)

selected_customers = customers.select(
    "customer_id",
    "customer_name"
)

result = filtered_sales.join(
    broadcast(selected_customers),
    on="customer_id",
    how="inner"
)
```

The expected improvement comes from reducing unnecessary data and potentially avoiding a shuffle of the large fact table.

## 10. Common Mistakes

1. Broadcasting a large dataset without checking memory requirements.
2. Assuming every join causes a shuffle of both datasets.
3. Repartitioning without understanding the join key distribution.
4. Ignoring data skew.
5. Selecting unnecessary columns before a join.
6. Assuming AQE fixes every join performance issue.
7. Using join hints without inspecting the physical plan.
8. Assuming a smaller compressed file size always means a safe broadcast size.
9. Confusing left semi joins with inner joins.
10. Optimizing execution time without checking correctness of the join result.

## 11. Interview Questions

**Q1. Why are joins expensive in Spark?**

Joins can require data movement across executors, sorting, hashing, network communication, and memory usage.

**Q2. What is a Broadcast Hash Join?**

Spark broadcasts a small dataset to executors and joins it locally with the larger dataset using a hash table.

**Q3. When would you use a Broadcast Join?**

When one side is sufficiently small to fit comfortably in executor memory and broadcasting is supported for the join.

**Q4. What is a Sort Merge Join?**

Spark redistributes data by join keys when required, sorts records within partitions, and merges the sorted records to find matches.

**Q5. What is the difference between Broadcast Hash Join and Sort Merge Join?**

Broadcast Hash Join distributes a small side to executors and avoids shuffling the large side for the join. Sort Merge Join generally shuffles both sides when they are not already appropriately distributed and sorts them before merging.

**Q6. What is data skew in joins?**

Data skew occurs when some join keys contain much more data than others, causing uneven partition sizes and long-running tasks.

**Q7. How do you handle data skew?**

Use AQE skew join optimization, salting, broadcasting a suitable small side, separating hot keys, or pre-aggregating where appropriate.

**Q8. What is AQE?**

Adaptive Query Execution uses runtime statistics to adjust query execution plans, including certain join strategy changes and skew handling.

**Q9. How do you check which join strategy Spark selected?**

Use `explain("formatted")` and inspect the Spark SQL tab in the Spark UI.

**Q10. What is a left semi join?**

It returns rows from the left DataFrame that have matching keys in the right DataFrame, without returning right-side columns.

**Q11. Does repartitioning always improve join performance?**

No. Repartitioning causes a shuffle and can add overhead. It should be used when the resulting distribution benefits the workload.

**Q12. What is the role of table statistics in join optimization?**

Statistics help Spark estimate data sizes and choose suitable physical join strategies.

## 12. Practice Questions

1. You have a 200 GB fact table and a 10 MB dimension table. Which join strategy would you investigate first, and why?
2. Your join uses Sort Merge Join, but the dimension table is small. How would you investigate whether broadcasting is appropriate?
3. One join task takes 20 minutes while the remaining tasks finish in 2 minutes. What could be the reason?
4. How would you identify a shuffle in the physical execution plan?
5. What is the difference between a broadcast join and a shuffle hash join?
6. Why can selecting fewer columns improve join performance?
7. When would you use a left semi join instead of an inner join?
8. Can AQE change a join strategy after execution begins?
9. Why is repartitioning by a skewed join key not always effective?
10. How would you investigate a join that is slow even after broadcasting a small table?

## 13. Quick Revision

- **Join optimization:** Reduce unnecessary data movement and processing.
- **Broadcast Hash Join:** Broadcast small side; avoid shuffling the large side for the join.
- **Sort Merge Join:** Shuffle when required, sort, and merge.
- **Shuffle Hash Join:** Shuffle and build a local hash table per partition.
- **Data skew:** Uneven data distribution across join keys.
- **AQE:** Adjusts eligible query plans using runtime statistics.
- **Filter early:** Reduce records before joining.
- **Select required columns:** Reduce data volume.
- **Statistics:** Help the optimizer choose a suitable strategy.
- **Join hints:** Influence the physical strategy.
- **explain("formatted"):** Inspect the physical plan.
- **Spark UI:** Investigate shuffle, spill, and task duration.
- **Left semi join:** Return matching rows from the left side only.

## 14. Self-Check

- [ ] Explain why joins can cause shuffling.
- [ ] Differentiate Broadcast Hash Join, Sort Merge Join, and Shuffle Hash Join.
- [ ] Explain when broadcasting is appropriate.
- [ ] Understand data skew and its impact on joins.
- [ ] Explain AQE join optimization.
- [ ] Know how to use broadcast hints.
- [ ] Understand filter and column pruning before joins.
- [ ] Identify join operators using `explain("formatted")`.
- [ ] Use Spark UI to investigate slow joins.
- [ ] Explain a real-world join optimization scenario.

**Interview goal:** Given two datasets and a slow join, explain how you would inspect the execution plan, choose an appropriate join strategy, reduce data movement, handle skew, and verify the improvement.