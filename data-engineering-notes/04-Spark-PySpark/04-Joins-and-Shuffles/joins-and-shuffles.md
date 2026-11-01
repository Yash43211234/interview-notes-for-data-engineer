# Joins and Shuffles in Spark

## 1. What Is a Join?

A join combines rows from two DataFrames using a related column or condition.

Example:

**Customers**

| customer_id | name |
|---|---|
| 1 | Rahul |
| 2 | Amit |
| 3 | Priya |

**Orders**

| order_id | customer_id | amount |
|---|---|---|
| 101 | 1 | 500 |
| 102 | 2 | 800 |
| 103 | 1 | 300 |

```python
result = customers.join(
    orders,
    on="customer_id",
    how="inner"
)
```

The result contains customer details along with matching orders.

## 2. Why Can Joins Be Expensive?

Spark processes data across multiple partitions and executors.

Suppose:
- Customers DataFrame has 10 partitions.
- Orders DataFrame has 20 partitions.
- Matching customer records are spread across different partitions.

Spark may need to move data across executors so matching keys can be processed together.

This data movement is called a **shuffle**.

A shuffle can involve:
1. Reading data from existing partitions.
2. Redistributing records based on join keys.
3. Transferring data between executors.
4. Writing and reading intermediate shuffle data.
5. Performing the join.

This network and disk activity can make joins expensive.

## 3. What Is a Shuffle?

A shuffle is the redistribution of data across partitions, usually across executors, so that Spark can perform an operation.

Example:

Before shuffle:

| Partition 1 | Partition 2 |
|---|---|
| customer_id = 1 | customer_id = 2 |
| customer_id = 3 | customer_id = 1 |

After repartitioning by `customer_id`, records with the same key are directed to the same target partition.

Conceptually:

| Target partition | Records |
|---|---|
| Partition A | customer_id = 1, customer_id = 1 |
| Partition B | customer_id = 2 |
| Partition C | customer_id = 3 |

This allows matching records to be processed together.

**Important:** A shuffle does not necessarily mean every record moves to another executor. It means Spark redistributes data between partitions; some records may remain where they are.

## 4. Why Do Joins Cause Shuffles?

Consider two large DataFrames:

```python
result = customers.join(
    orders,
    on="customer_id",
    how="inner"
)
```

If Spark chooses a Sort Merge Join, it generally needs to:
- Shuffle both DataFrames by the join key.
- Sort the records within partitions.
- Merge matching keys.

This is why large joins can generate significant shuffle read and write.

However, not every join requires a shuffle. The join strategy determines how data is processed.

## 5. Common Join Strategies

### A. Sort Merge Join (SMJ)

A common strategy for joining large DataFrames.

**How it works:**
1. Shuffle both DataFrames by the join key, when required.
2. Sort records within each partition.
3. Merge records with matching keys.

Example:

```python
result = large_df1.join(
    large_df2,
    on="customer_id",
    how="inner"
)
```

**Advantages**
- Suitable for large datasets.
- Can handle datasets that do not fit entirely in memory.
- Commonly used for large equi-joins.

**Disadvantages**
- Shuffle can be expensive.
- Sorting requires additional processing.
- Data skew can create slow tasks.

Physical plan commonly contains:

```text
SortMergeJoin
Exchange
Sort
```

### B. Broadcast Hash Join (BHJ)

Used when one DataFrame is small enough to broadcast.

Spark sends a copy of the small DataFrame to executors. Each executor can then join its local large-data partitions with the broadcasted data.

Example:

```python
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    on="customer_id",
    how="inner"
)
```

**How it works:**
1. Broadcast the small DataFrame to executors.
2. Build a hash table from the broadcasted data.
3. Process each partition of the large DataFrame locally.
4. Find matching keys using the hash table.

**Advantages**
- Usually avoids shuffling the large DataFrame for the join.
- Reduces network data movement compared with shuffling both sides.
- Often faster for large-to-small joins.

**Disadvantages**
- Broadcasting a large dataset can cause executor memory pressure.
- Broadcast data is sent across the cluster.
- The small DataFrame must be suitable for broadcasting.

Physical plan commonly contains:

```text
BroadcastHashJoin
BroadcastExchange
```

### C. Broadcast Nested Loop Join

Used in certain joins where a hash-based equi-join is not suitable, such as some non-equality join conditions.

It broadcasts one side and evaluates matching conditions against the other side.

It can be expensive for large datasets because it may require many comparisons.

### D. Shuffle Hash Join

Spark shuffles data by join keys and builds hash tables within partitions.

It can be useful when the build side of each partition is sufficiently small, but it can consume substantial memory.

It is different from Broadcast Hash Join because the data is shuffled rather than broadcasting one complete small DataFrame to all executors.

## 6. Sort Merge Join vs Broadcast Hash Join

| Feature | Sort Merge Join | Broadcast Hash Join |
|---|---|---|
| Best suited for | Large datasets | Large + small dataset |
| Broadcast required | No | Yes |
| Shuffle | Usually both sides | Usually avoids shuffling the large side |
| Sorting | Yes | No sort required for the hash join |
| Memory requirement | Sorting and execution memory | Broadcast data and hash table memory |
| Common use | Large-to-large joins | Large-to-small joins |

**Interview takeaway:** If one DataFrame is small enough, a Broadcast Hash Join can avoid the expensive shuffle of a large DataFrame.

## 7. How Does Spark Choose a Join Strategy?

Spark's optimizer considers factors such as:
- Join condition.
- DataFrame size estimates.
- Available statistics.
- Configuration settings.
- Join hints.
- Adaptive Query Execution (AQE), when enabled.

Important configuration:

```python
spark.conf.get("spark.sql.autoBroadcastJoinThreshold")
```

The standard Spark SQL default is commonly 10 MB. Platform configurations can differ.

You can provide a broadcast hint:

```python
from pyspark.sql.functions import broadcast

result = fact_df.join(
    broadcast(dim_df),
    "product_id",
    "inner"
)
```

A hint influences the optimizer's choice, but it does not guarantee that every requested strategy will be used in every situation.

## 8. How to Identify Shuffles?

Use:

```python
result.explain("formatted")
```

Look for physical-plan operators such as:

```text
Exchange
Sort
SortMergeJoin
BroadcastExchange
BroadcastHashJoin
```

- `Exchange`: indicates data redistribution; often associated with a shuffle.
- `Sort`: sorts data within partitions.
- `SortMergeJoin`: performs a sort-merge join.
- `BroadcastExchange`: prepares data for broadcasting.
- `BroadcastHashJoin`: performs a broadcast hash join.

Also inspect the Spark UI:
- Stages and task duration.
- Shuffle Read.
- Shuffle Write.
- Memory and disk spill.
- Uneven task execution times.

**Important:** An `Exchange` is a useful clue, but not every exchange represents the same kind of data movement. Read the physical plan in context.

## 9. Ways to Reduce Shuffle Cost

### 1. Use Broadcast Join

If one DataFrame is genuinely small, broadcast it.

```python
result = large_df.join(
    broadcast(small_df),
    "id"
)
```

Do not broadcast a DataFrame blindly. Check its size and executor memory.

### 2. Filter Before Joining

Reduce the amount of data before the join.

```python
filtered_orders = orders.filter(
    orders.status == "COMPLETED"
)

result = customers.join(
    filtered_orders,
    "customer_id"
)
```

This can reduce the amount of data processed and shuffled.

### 3. Select Only Required Columns

Avoid carrying unnecessary columns through a join.

```python
customers_small = customers.select(
    "customer_id",
    "name"
)

orders_small = orders.select(
    "customer_id",
    "amount"
)

result = customers_small.join(
    orders_small,
    "customer_id"
)
```

### 4. Handle Data Skew

If one join key has a very large number of records, one or a few partitions may become much larger than others.

Possible solutions:
- Broadcast a suitable small DataFrame.
- Use AQE skew-join optimization where applicable.
- Apply salting for hot keys when appropriate.
- Revisit the join logic and data distribution.

### 5. Reuse Partitioning When Appropriate

If multiple operations use the same join key, suitable existing partitioning may reduce repeated redistribution in some situations.

However, Spark does not guarantee that a previous repartition will eliminate every later shuffle. The optimizer and physical plan determine whether partitioning can be reused.

## 10. repartition() vs coalesce()

These methods change the number of DataFrame partitions.

### repartition()

```python
df_repartitioned = df.repartition(10)
```

- Can increase or decrease the number of partitions.
- Performs a shuffle in the general case.
- Can repartition using specific columns.

```python
df_repartitioned = df.repartition(
    10,
    "customer_id"
)
```

### coalesce()

```python
df_coalesced = df.coalesce(5)
```

- Primarily used to reduce partitions.
- Usually avoids a full shuffle.
- May produce uneven partition sizes.
- Cannot increase the number of partitions.

**Important:** Repartitioning by a highly skewed key does not automatically solve data skew. Records with the same key can still end up concentrated in one partition.

## 11. AQE and Join Optimization

Adaptive Query Execution (AQE) can use runtime statistics to improve execution plans.

For joins, AQE can:
- Convert a sort-merge join to a broadcast hash join when runtime statistics make broadcasting suitable.
- Handle certain skewed shuffle partitions in supported joins.
- Optimize some shuffle partition decisions.

Common configuration:

```python
spark.conf.get(
    "spark.sql.adaptive.enabled"
)
```

AQE is enabled by default in standard Spark SQL configurations from Spark 3.2 onward, but platform settings may differ.

**Remember:** AQE is not a guarantee that every shuffle or skew problem will be eliminated.

## 12. Common Mistakes

1. Assuming every join causes a shuffle.
2. Assuming every `Exchange` means the same operation.
3. Broadcasting a large DataFrame without checking memory requirements.
4. Repartitioning without understanding the join key and data distribution.
5. Filtering after a join when the filter could safely be applied before it.
6. Carrying unnecessary columns through joins.
7. Assuming AQE fixes every performance problem.
8. Ignoring data skew and uneven task execution times.
9. Assuming a broadcast hint guarantees a Broadcast Hash Join.

## 13. Interview Questions

**Q1. What is a shuffle in Spark?**

A shuffle is the redistribution of records across partitions so that Spark can perform operations such as joins and aggregations.

**Q2. Why are joins expensive in Spark?**

Large joins may require shuffling data across executors, sorting records, and processing large amounts of intermediate data. Network and disk operations increase execution time.

**Q3. Does every join cause a shuffle?**

No. A broadcast hash join can avoid shuffling the large DataFrame for the join. Other strategies may require redistribution.

**Q4. What is a Sort Merge Join?**

It is a join strategy that generally shuffles both sides by the join key, sorts records within partitions, and merges matching keys.

**Q5. What is a Broadcast Hash Join?**

Spark broadcasts a small DataFrame to executors and uses a hash table to join it with local partitions of the larger DataFrame.

**Q6. When should you use a broadcast join?**

When one side is sufficiently small to be broadcast safely to executors.

**Q7. How do you identify a shuffle?**

Use `explain("formatted")` and inspect operators such as `Exchange`. Confirm actual shuffle read/write and task behavior in the Spark UI.

**Q8. How can you reduce shuffle during joins?**

Use broadcast joins when appropriate, filter early, select only necessary columns, address data skew, and avoid unnecessary repartitioning.

**Q9. What is the difference between repartition and coalesce?**

`repartition()` generally performs a shuffle and can increase or decrease partitions. `coalesce()` usually reduces partitions without a full shuffle.

**Q10. How does AQE optimize joins?**

AQE uses runtime statistics to adapt the physical plan. It can convert certain joins to broadcast joins and optimize supported skewed shuffle joins.

## 14. Practice Questions

1. You have a 500 GB fact DataFrame and a 5 MB dimension DataFrame. Which join strategy would you consider, and why?
2. Your join takes 30 minutes, and the Spark UI shows high shuffle read and write. What would you investigate?
3. One task takes 20 minutes while most tasks finish in 2 minutes. What could be the reason?
4. Your physical plan shows `SortMergeJoin` and two `Exchange` operators. Explain what is happening.
5. You broadcast a DataFrame and executors start running out of memory. What would you investigate?
6. Two large DataFrames are joined using a common key. Why might Spark choose Sort Merge Join?
7. You repartition a skewed DataFrame using its hot join key, but performance does not improve. Why?

## 15. Quick Revision

- **Join:** Combines matching records from DataFrames.
- **Shuffle:** Redistributes records across partitions.
- **Sort Merge Join:** Usually shuffles and sorts both sides before merging.
- **Broadcast Hash Join:** Broadcasts a small side and joins it locally with large-side partitions.
- **Broadcast Nested Loop Join:** Broadcast-based strategy for certain conditions that cannot use a hash equi-join.
- **Shuffle Hash Join:** Shuffles data and builds hash tables within partitions.
- **Exchange:** Physical-plan operator indicating data redistribution.
- **repartition():** Generally causes a shuffle.
- **coalesce():** Usually reduces partitions without a full shuffle.
- **AQE:** Adapts execution plans using runtime statistics.

## 16. Self-Check

Before moving to the next topic, make sure you can:

- [ ] Explain why joins can cause shuffles.
- [ ] Explain how a shuffle works.
- [ ] Differentiate Sort Merge Join and Broadcast Hash Join.
- [ ] Identify shuffle-related operators in a physical plan.
- [ ] Explain how to reduce shuffle cost.
- [ ] Differentiate `repartition()` and `coalesce()`.
- [ ] Explain how AQE can optimize joins.
- [ ] Diagnose a slow join using Spark UI metrics.

**Interview goal:** Be able to explain not only which join strategy Spark uses, but also why it uses that strategy and how you would investigate its performance.