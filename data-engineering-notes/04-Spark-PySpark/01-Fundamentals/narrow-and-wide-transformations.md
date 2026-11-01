# Narrow and Wide Transformations in Apache Spark

## 1. What Are Transformations?

In Apache Spark, transformations are operations that create a new DataFrame or RDD from an existing one.

Transformations are generally lazy, meaning Spark records the operations and executes them when an action is called.

Based on how data is distributed across partitions, transformations are classified into two main types:

1. Narrow transformations
2. Wide transformations

The main difference is whether an operation requires data to move between partitions.

---

## 2. Narrow Transformations

A narrow transformation is an operation where each output partition depends on data from a limited number of input partitions, typically one.

Data does not need to be redistributed across the cluster through a shuffle.

### Examples

- `filter()`
- `select()`
- `map()`
- `flatMap()`
- `withColumn()` for ordinary row-level expressions
- `union()` in common DataFrame execution plans, when no redistribution is required

### Example

Suppose we have three partitions:

```text
Partition 1: [10, 20, 30]
Partition 2: [40, 50, 60]
Partition 3: [70, 80, 90]
```

We apply a filter:

```python
filtered_df = df.filter(df.amount > 50)
```

Conceptually, Spark processes each partition independently:

```text
Partition 1              Partition 2              Partition 3
[10, 20, 30]             [40, 50, 60]             [70, 80, 90]
     |                        |                        |
   Filter                   Filter                   Filter
     |                        |                        |
    []                    [60]                    [70, 80, 90]
```

Each output partition depends only on its corresponding input partition.

No shuffle is required.

### Key Characteristics

- No shuffle is normally required.
- Data remains within its existing partitioning structure.
- Operations can often be pipelined together.
- Usually less expensive than wide transformations.
- Multiple narrow transformations can often execute within the same stage.

---

## 3. Wide Transformations

A wide transformation is an operation where an output partition depends on data from multiple input partitions.

Spark often needs to redistribute data across partitions to complete the operation.

This redistribution is called a **shuffle**.

### Examples

- `groupBy()`
- `distinct()`
- `dropDuplicates()` in common distributed execution plans
- `orderBy()` / `sort()`
- `repartition()`
- Many joins, such as shuffle-based joins

### Example: Group By

Suppose we have three partitions:

```text
Partition 1:
(A, 100)
(B, 200)

Partition 2:
(A, 300)
(C, 400)

Partition 3:
(B, 500)
(C, 600)
```

We perform:

```python
result = df.groupBy("category").sum("amount")
```

Spark needs to bring records with the same category together.

Conceptually:

```text
Input Partitions
       |
       v
Extract Grouping Key
       |
       v
Shuffle Data Across Partitions
       |
       v
Group Matching Keys
       |
       v
Aggregate Values
```

After redistribution, the data may look like:

```text
Partition 1:
(A, 100)
(A, 300)

Partition 2:
(B, 200)
(B, 500)

Partition 3:
(C, 400)
(C, 600)
```

Now Spark can calculate the aggregation for each category.

Result:

```text
A -> 400
B -> 700
C -> 1000
```

This redistribution creates network and disk I/O overhead.

### Key Characteristics

- Often requires a shuffle.
- Data may move between executors.
- Can involve network communication and disk I/O.
- Usually more expensive than narrow transformations.
- Shuffle boundaries commonly create new Spark stages.

---

## 4. Narrow vs Wide Transformations

| Feature | Narrow Transformation | Wide Transformation |
|---|---|---|
| Data dependency | Limited input partitions, typically one | Multiple input partitions |
| Shuffle | Normally not required | Often required |
| Data movement | Usually stays within partitions | May move across executors |
| Performance | Generally cheaper | Generally more expensive |
| Stage boundary | Usually does not create one by itself | Shuffle commonly creates a stage boundary |
| Execution | Often pipelined | Requires redistribution when shuffling |
| Examples | `filter()`, `map()`, `select()` | `groupBy()`, `distinct()`, `repartition()` |

---

## 5. How Transformations Affect Spark Stages

Spark divides a job into stages based on dependencies and execution requirements.

Narrow transformations can often be combined into the same stage because they do not require a shuffle.

Wide transformations commonly introduce shuffle boundaries, separating stages.

### Example

```python
result = (
    df
    .filter(df.amount > 100)
    .select("category", "amount")
    .groupBy("category")
    .sum("amount")
)

result.show()
```

Conceptual execution:

```text
Read Data
    |
    v
Filter                  Narrow
    |
    v
Select                  Narrow
    |
    v
Shuffle                 Wide dependency
    |
    v
Group By and Aggregate
    |
    v
Action: show()
```

A simplified stage representation:

```text
Stage 1:
Read -> Filter -> Select -> Partial Aggregation
                    |
                    v
                  Shuffle
                    |
                    v
Stage 2:
Shuffle Read -> Final Aggregation
                    |
                    v
                  Result
```

**Important:** The actual physical plan may contain additional operators and stages. Spark can perform partial aggregation before shuffling, so a `groupBy()` does not necessarily send every original row across the network.

---

## 6. Why Are Wide Transformations Expensive?

Wide transformations can introduce shuffle overhead.

A shuffle involves several activities:

1. Spark partitions intermediate data according to the required distribution.
2. Data may be written to local disk.
3. Data is transferred across the network when required.
4. Other tasks fetch the shuffled data.
5. Spark processes the received data.

### Common Performance Problems

**1. Network I/O**

Large amounts of data may move between executors.

**2. Disk I/O**

Intermediate shuffle data may be written to and read from disk.

**3. Data Skew**

Some partitions may receive significantly more data than others.

**4. Memory Pressure**

Large aggregations and joins may require substantial memory.

**5. Longer Execution Time**

Tasks may spend significant time waiting for shuffle data or processing uneven partitions.

---

## 7. How to Reduce Shuffle Overhead

### 7.1 Filter Data Early

Reduce unnecessary rows before expensive operations.

```python
filtered_df = df.filter(df.amount > 1000)

result = filtered_df.groupBy("category").sum("amount")
```

This can reduce the amount of data involved in the shuffle.

### 7.2 Select Only Required Columns

Avoid carrying unnecessary columns through the pipeline.

```python
selected_df = df.select(
    "category",
    "amount"
)
```

This can reduce the volume of data shuffled.

### 7.3 Use Broadcast Joins When Appropriate

If one DataFrame is small enough, Spark may broadcast it to the executors instead of performing a large shuffle-based join.

```python
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    on="customer_id",
    how="inner"
)
```

Broadcast joins can avoid shuffling the large DataFrame for the join, although they have their own memory and execution costs.

### 7.4 Avoid Unnecessary Repartitioning

```python
df = df.repartition(20)
```

`repartition()` generally causes a shuffle.

Use it only when the redistribution is useful, such as preparing data for downstream parallel processing.

### 7.5 Handle Data Skew

If one key contains much more data than others, some tasks may take significantly longer.

Possible approaches include:

- Salting skewed keys.
- Using broadcast joins when appropriate.
- Adjusting partitioning.
- Using Adaptive Query Execution (AQE) where supported.

---

## 8. Important Interview Concepts

### Does Every Join Cause a Shuffle?

No.

A join may use different physical strategies.

For example:

- Sort-merge join: commonly involves shuffling both sides.
- Shuffle hash join: commonly involves shuffling data by join key.
- Broadcast hash join: broadcasts the smaller side and can avoid shuffling the larger side for the join.

The actual strategy depends on data size, statistics, configuration, and the physical plan.

### Does Every Group By Cause a Shuffle?

A distributed `groupBy()` commonly requires data to be redistributed by grouping key.

However, Spark can perform partial aggregation locally before the shuffle. This reduces the amount of data transferred.

### Does Union Cause a Shuffle?

A normal DataFrame `union()` generally combines rows without requiring a shuffle.

However, later operations or additional requirements in the execution plan may introduce one.

### Is Repartition Narrow or Wide?

`repartition()` is generally a wide operation because it redistributes data across partitions using a shuffle.

### Is Coalesce Narrow or Wide?

`coalesce()` without shuffle generally reduces the number of partitions using a narrow dependency.

With `shuffle=True`, it can redistribute data and behave as a wide operation.

```python
# Usually narrow: reduce partitions without full redistribution
df.coalesce(5)

# Wide: redistribution through shuffle
df.coalesce(5, shuffle=True)
```

---

## 9. How to Identify Shuffle in Spark

Use the physical execution plan:

```python
df.groupBy("category").sum("amount").explain(True)
```

Look for operators such as:

- `Exchange`
- `Exchange hashpartitioning`
- `SortMergeJoin`
- `ShuffleExchange`

An `Exchange` operator commonly indicates data redistribution or a shuffle-related exchange in the physical plan.

You can also inspect the Spark UI:

1. Open the Spark application.
2. Go to the SQL tab or Jobs tab.
3. Select the relevant query or job.
4. Inspect the stages and task details.
5. Check shuffle read, shuffle write, spill, and task duration.

This helps identify expensive shuffle operations and data skew.

---

## 10. Interview Questions

### Q1. What is a narrow transformation?

A narrow transformation is an operation where each output partition depends on a limited number of input partitions, typically one. It generally does not require a shuffle.

### Q2. What is a wide transformation?

A wide transformation is an operation where output partitions depend on data from multiple input partitions. It often requires a shuffle to redistribute data.

### Q3. What is a shuffle in Spark?

Shuffle is the process of redistributing data across partitions, often across executors, so that operations such as grouping, sorting, and joining can be performed.

### Q4. Why are wide transformations expensive?

They can involve network communication, disk I/O, memory usage, and data redistribution.

### Q5. Does filter() cause a shuffle?

Normally, no. `filter()` is a narrow transformation because it processes records within their existing partitions.

### Q6. Does groupBy() cause a shuffle?

A distributed `groupBy()` commonly requires a shuffle so that records with the same grouping key can be brought together. Spark may perform partial aggregation before shuffling.

### Q7. What is the relationship between wide transformations and stages?

Wide transformations commonly introduce shuffle boundaries. Spark generally creates separate stages around shuffle dependencies.

### Q8. How can you reduce shuffle?

Filter early, select only required columns, avoid unnecessary repartitioning, use suitable broadcast joins, and handle data skew.

### Q9. What is the difference between repartition() and coalesce()?

- `repartition()` redistributes data through a shuffle and can increase or decrease partitions.
- `coalesce()` without shuffle generally reduces partitions while avoiding a full redistribution.

### Q10. Can a join happen without a shuffle?

Yes. A broadcast hash join can avoid shuffling the large side of the join by broadcasting the smaller side.

---

## 11. Quick Revision

| Concept | Remember |
|---|---|
| Narrow transformation | Limited input partition dependency |
| Wide transformation | Multiple input partition dependency |
| Shuffle | Redistribution of data across partitions |
| Narrow examples | `filter()`, `map()`, `select()` |
| Wide examples | `groupBy()`, `distinct()`, `repartition()` |
| Narrow execution | Often pipelined in the same stage |
| Wide execution | Commonly introduces a stage boundary |
| `coalesce()` | Usually narrow without shuffle |
| `repartition()` | Wide; generally involves shuffle |
| Broadcast join | Can avoid shuffling the large side |

### One-Line Interview Answer

Narrow transformations process data without requiring a shuffle because each output partition depends on a limited number of input partitions. Wide transformations require data from multiple input partitions and commonly involve a shuffle, which redistributes data across the cluster.

**Most important concept:** Narrow vs wide is fundamentally about data dependency between input and output partitions, not simply whether an operation looks simple or complex.