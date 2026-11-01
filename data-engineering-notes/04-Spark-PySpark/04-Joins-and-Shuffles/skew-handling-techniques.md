# Skew Handling Techniques in Apache Spark

## 1. What Is Data Skew?

Data skew occurs when data is distributed unevenly across partitions.

Some partitions contain significantly more records than others, causing certain tasks to take much longer.

**Example:**

| Partition | Number of records |
|---|---:|
| Partition 1 | 10,000 |
| Partition 2 | 12,000 |
| Partition 3 | 500,000 |
| Partition 4 | 9,000 |

Partition 3 has significantly more records.

As a result:
- Its task takes longer.
- Other tasks may finish early.
- The entire stage waits for the slow task.
- Memory pressure and disk spill may increase.

This is called the **straggler task problem**.

## 2. Why Does Data Skew Happen?

### A. Hot Keys

A hot key is a value that appears much more frequently than other values.

Example:

| customer_id | Number of records |
|---|---:|
| 101 | 5,000 |
| 102 | 4,000 |
| 103 | 450,000 |
| 104 | 3,000 |

If Spark partitions data using `customer_id`, a large number of records for customer 103 can end up in one partition.

### B. Low Cardinality

A column with very few distinct values can create uneven partitions when the frequency of those values is different.

Example:
- Gender: Male, Female.
- Status: Active, Inactive.
- Country: A small number of countries dominate the data.

Low cardinality alone does not guarantee skew. Uneven value frequency is the important factor.

### C. NULL or Default Values

Many records may contain the same missing or default value.

Example:

| customer_id | amount |
|---|---:|
| NULL | 100 |
| NULL | 200 |
| NULL | 300 |
| NULL | 400 |
| 101 | 500 |

If records are grouped or repartitioned using this key, the repeated NULL value can create a hot partition.

**Important:** In an ordinary SQL equality join, NULL does not match NULL. However, NULL values can still create skew during operations such as grouping or partitioning.

## 3. How to Identify Data Skew?

### A. Check Key Distribution

Use PySpark to find frequently occurring keys.

```python
from pyspark.sql.functions import col, count

df.groupBy("customer_id") \
  .agg(count("*").alias("record_count")) \
  .orderBy(col("record_count").desc()) \
  .show(20)
```

This helps identify hot keys.

### B. Check Spark UI

Open the Spark UI and inspect the stage details.

Look for:

- Large differences in task duration.
- Uneven shuffle read sizes.
- Uneven shuffle write sizes.
- High memory spill.
- High disk spill.
- A small number of tasks taking much longer than others.

Example:

| Task | Duration |
|---|---:|
| Task 1 | 20 seconds |
| Task 2 | 22 seconds |
| Task 3 | 25 seconds |
| Task 4 | 12 minutes |

Task 4 may be processing a skewed partition.

### C. Compare Partition Sizes

You can inspect the number of records in each partition:

```python
partition_counts = df.rdd.mapPartitions(
    lambda rows: [sum(1 for _ in rows)]
).collect()

print(partition_counts)
```

This is useful for diagnosis, but converting an RDD result to the driver with `collect()` should be used carefully when the number of partitions is large.

## 4. Techniques to Handle Data Skew

The correct solution depends on the operation causing skew.

Common techniques include:

1. Salting.
2. Broadcast Join.
3. Adaptive Query Execution (AQE).
4. Repartitioning with an appropriate strategy.
5. Filtering or separating hot keys.
6. Pre-aggregation.

## 5. Technique 1: Salting

Salting is a technique that distributes records belonging to a hot key across multiple partitions by adding an additional value called a salt.

### Problem Without Salting

Suppose a large DataFrame contains:

| customer_id | amount |
|---|---:|
| 101 | 100 |
| 101 | 200 |
| 101 | 300 |
| 101 | 400 |
| 101 | 500 |
| 102 | 600 |

If customer 101 is a hot key, its records may be concentrated in one partition.

### Solution: Add a Salt Column

```python
from pyspark.sql.functions import (
    col, floor, rand
)

salt_count = 5

salted_df = large_df.withColumn(
    "salt",
    floor(rand() * salt_count).cast("int")
)
```

Now the hot key can be distributed across multiple salt values.

Conceptually:

| customer_id | salt | amount |
|---|---:|---:|
| 101 | 0 | 100 |
| 101 | 1 | 200 |
| 101 | 2 | 300 |
| 101 | 3 | 400 |
| 101 | 4 | 500 |

The salt values shown are illustrative; `rand()` assigns them during execution.

### Salting for Joins

Suppose:
- `large_df` contains a hot customer ID.
- `small_df` contains one matching row for that customer ID.

We add a salt to the large DataFrame and replicate the corresponding small-side rows across all salt values.

```python
from pyspark.sql.functions import (
    col, floor, rand, explode, sequence
)

salt_count = 5

# Add salt to the large DataFrame
salted_large = large_df.withColumn(
    "salt",
    floor(rand() * salt_count).cast("int")
)

# Replicate small-side rows for all salt values
salted_small = small_df.withColumn(
    "salt",
    explode(sequence(
        col("dummy").cast("int") * 0,
        col("dummy").cast("int") * 0 + salt_count - 1
    ))
)
```

The example above illustrates the idea but assumes a `dummy` column, so a cleaner implementation is to create the salt values explicitly.

```python
from pyspark.sql import functions as F

salt_count = 5

salt_values = spark.range(salt_count).select(
    F.col("id").cast("int").alias("salt")
)

salted_large = large_df.withColumn(
    "salt",
    F.floor(F.rand() * salt_count).cast("int")
)

salted_small = small_df.crossJoin(salt_values)

result = salted_large.join(
    salted_small,
    on=["customer_id", "salt"],
    how="inner"
)
```

**Why does this work?**

- Large-side records for the hot key are spread across multiple salt values.
- Small-side records are replicated for those salt values.
- Matching records still join correctly because both join keys are used.

**Important:** This basic example salts every key. In real applications, salt only the identified hot keys when possible. Otherwise, unnecessary replication increases data size.

### Advantages

- Distributes hot-key records across partitions.
- Can reduce the impact of skewed joins.
- Useful when AQE alone does not solve the problem.

### Disadvantages

- Replicates records on the other side.
- Adds processing and memory overhead.
- Requires choosing an appropriate salt count.
- Incorrect salting can produce missing or duplicate results.

## 6. Technique 2: Broadcast Join

If one DataFrame is sufficiently small, broadcast it to executors.

```python
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    on="customer_id",
    how="inner"
)
```

**Why can it help with skew?**

A broadcast join avoids shuffling the large DataFrame for the join. Each executor joins its local large-data partitions with the broadcasted small DataFrame.

This can remove the specific shuffle bottleneck caused by a skewed join.

However:
- It does not automatically fix skew in every operation.
- The broadcast DataFrame must be small enough for executor memory.
- Broadcasting a large DataFrame can cause memory pressure.

## 7. Technique 3: Adaptive Query Execution (AQE)

AQE allows Spark to modify the physical execution plan using runtime statistics.

It can help handle skewed shuffle partitions in supported joins.

Important configuration:

```python
spark.conf.set(
    "spark.sql.adaptive.enabled",
    "true"
)

spark.conf.set(
    "spark.sql.adaptive.skewJoin.enabled",
    "true"
)
```

Common standard Spark SQL defaults:
- `spark.sql.adaptive.enabled`: `true` in standard Spark SQL configurations from Spark 3.2 onward.
- `spark.sql.adaptive.skewJoin.enabled`: `true` when AQE is enabled in standard configurations.

Platform configurations may differ.

### How AQE Handles Skew

Suppose a shuffle partition contains much more data than other partitions.

AQE can:
1. Identify eligible skewed partitions using runtime statistics.
2. Split a large skewed partition into smaller pieces.
3. Create additional join tasks.
4. Replicate the corresponding partition from the other join side when required.
5. Execute the smaller tasks in parallel.

**Important limitations:**

- AQE skew-join optimization primarily targets supported shuffled join strategies, such as Sort Merge Join.
- It does not automatically solve every skewed aggregation or custom processing problem.
- Its behavior depends on runtime statistics, thresholds, and the physical plan.

## 8. Technique 4: Repartitioning

Repartitioning changes how data is distributed across partitions.

```python
df_repartitioned = df.repartition(
    20,
    "customer_id"
)
```

This performs a shuffle and redistributes records based on the specified key.

**Important:** Repartitioning by the same skewed key does not necessarily solve hot-key skew.

Why?

Records with the same key are still directed to the same hash partition.

Repartitioning can help when the problem is an unsuitable partition count or distribution strategy, but it is not a general solution for hot keys.

## 9. Technique 5: Separate Hot Keys

Sometimes only a few keys cause most of the skew.

Instead of processing all records in one join, separate hot keys from normal keys.

Example:

```python
from pyspark.sql.functions import col

hot_keys = [101, 103]

hot_df = large_df.filter(
    col("customer_id").isin(hot_keys)
)

normal_df = large_df.filter(
    ~col("customer_id").isin(hot_keys)
)
```

You can process the normal keys using the regular join strategy and handle hot keys separately using salting or another suitable strategy.

Finally, combine the results:

```python
final_result = normal_result.unionByName(
    hot_result
)
```

**Important:** This is a conceptual pattern. Make sure the two branches are mutually exclusive and cover all required records. Handle NULL keys explicitly if they are part of the business logic.

## 10. Technique 6: Pre-Aggregation

If the requirement is to calculate aggregates, reduce the data before an expensive join or shuffle whenever possible.

Example:

```python
from pyspark.sql.functions import sum

aggregated_df = transactions.groupBy(
    "customer_id"
).agg(
    sum("amount").alias("total_amount")
)
```

Instead of joining or processing every transaction record, you may be able to work with one aggregated record per customer.

This can reduce the amount of data shuffled.

**Important:** Pre-aggregation is valid only when it preserves the required business logic. Do not aggregate away information needed by later operations.

## 11. Comparison of Skew-Handling Techniques

| Technique | Best use case | Main limitation |
|---|---|---|
| Salting | Identified hot keys | Replication overhead |
| Broadcast Join | One side is sufficiently small | Executor memory |
| AQE | Supported skewed shuffle joins | Does not solve every skew problem |
| Repartitioning | Unsuitable partition distribution | Same hot key may remain concentrated |
| Separate Hot Keys | Few keys cause most skew | More complex processing |
| Pre-Aggregation | Aggregation can happen earlier | Must preserve business logic |

## 12. How to Choose the Right Technique?

Use this practical decision process:

**Case 1: One DataFrame is small.**

Consider Broadcast Hash Join.

**Case 2: A few hot keys cause severe join skew.**

Consider salting or processing hot keys separately.

**Case 3: You are using a supported shuffled join and AQE is enabled.**

Check whether AQE skew-join optimization is handling the problem.

**Case 4: The operation is an aggregation with hot keys.**

Consider pre-aggregation or a custom aggregation strategy. AQE skew-join optimization is not a universal solution for skewed aggregations.

**Case 5: The partition count or distribution is unsuitable, but there are no dominant hot keys.**

Consider repartitioning with an appropriate strategy.

## 13. Common Mistakes

1. Increasing the number of partitions and assuming skew is fixed.
2. Repartitioning by the same hot key without changing the distribution problem.
3. Broadcasting a large DataFrame without checking memory.
4. Applying salting to every key unnecessarily.
5. Choosing too few salt values.
6. Forgetting to replicate the matching side when salting a join.
7. Assuming AQE fixes all types of data skew.
8. Ignoring Spark UI task-duration differences.
9. Applying pre-aggregation without checking whether it changes business results.
10. Assuming NULL values match each other in ordinary equality joins.

## 14. Interview Questions

**Q1. What is data skew in Spark?**

Data skew occurs when records are distributed unevenly across partitions, causing some tasks to process much more data than others.

**Q2. What is a hot key?**

A key that appears much more frequently than other keys and can cause one or more partitions to become disproportionately large.

**Q3. How do you identify data skew?**

Check key frequency, partition sizes, task duration, shuffle read/write, and spill metrics in Spark UI.

**Q4. What is salting?**

Salting adds an additional value to a hot key to distribute its records across multiple partitions.

**Q5. Why do we replicate the small side during a salted join?**

Because large-side records with the same original key now have different salt values. Replicating the matching small-side records ensures every salted key can find its match.

**Q6. How does AQE handle skew?**

For supported shuffled joins, AQE can identify oversized shuffle partitions, split them into smaller pieces, and process them in parallel.

**Q7. Does repartitioning solve data skew?**

Not necessarily. If the same hot key is used as the partitioning key, its records may still be concentrated in one partition.

**Q8. When would you use broadcast join to handle skew?**

When one side is sufficiently small to broadcast safely, avoiding the shuffle of the large side for the join.

**Q9. What is the difference between salting and AQE?**

Salting is a manual data transformation that changes key distribution. AQE is Spark's runtime optimization mechanism that can adapt supported execution plans.

**Q10. How would you troubleshoot a skewed join in production?**

Inspect Spark UI task durations and shuffle metrics, identify hot keys, review the physical plan, and choose a suitable strategy such as AQE, broadcast join, or salting.

## 15. Practice Questions

1. A join has 100 partitions, but one task takes 15 minutes while others finish in 30 seconds. What could be happening?
2. One customer ID represents 60% of the records. How would you handle the skew?
3. Why does repartitioning by a hot key not always solve the problem?
4. Explain how salting works with a small dimension DataFrame.
5. How does AQE split skewed shuffle partitions?
6. When is broadcast join a better choice than salting?
7. Why can salting increase data volume?
8. How would you handle skew in a groupBy aggregation?
9. What Spark UI metrics would you inspect?
10. How would you decide between salting, AQE, and broadcast join?

## 16. Quick Revision

- **Data Skew:** Uneven data distribution across partitions.
- **Hot Key:** A key with unusually high frequency.
- **Straggler Task:** A task that takes much longer than other tasks.
- **Salting:** Adds an extra key to distribute hot-key records.
- **Broadcast Join:** Avoids shuffling the large side when a suitable small side can be broadcast.
- **AQE:** Uses runtime statistics to adapt execution plans.
- **Repartitioning:** Changes partition distribution but does not automatically solve hot-key skew.
- **Pre-Aggregation:** Reduces data before later processing when business logic allows.
- **Spark UI:** Helps identify skew through task duration, shuffle metrics, and spill.

## 17. Self-Check

- [ ] Explain data skew and hot keys.
- [ ] Identify skew using Spark UI.
- [ ] Explain why a hot key creates a slow task.
- [ ] Explain salting with a join example.
- [ ] Explain why the small side must be replicated during salted joins.
- [ ] Explain AQE skew-join optimization.
- [ ] Differentiate broadcast join and salting.
- [ ]