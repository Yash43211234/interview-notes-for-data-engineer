# coalesce() vs repartition() in PySpark

## 1. What Are repartition() and coalesce()?

Both `repartition()` and `coalesce()` are DataFrame operations used to change the number of partitions in Apache Spark.

The main difference is how they redistribute data.

- **repartition():** Redistributes data across partitions using a shuffle.
- **coalesce():** Primarily reduces the number of partitions without performing a full shuffle.

Both operations return a new DataFrame. They do not modify the original DataFrame.

## 2. What Is repartition()?

`repartition()` redistributes records across a specified number of partitions.

Example:

```python
df_repartitioned = df.repartition(10)
```

This creates a DataFrame with 10 partitions.

### Key Characteristics

- Can increase the number of partitions.
- Can decrease the number of partitions.
- Generally performs a full shuffle.
- Redistributes data across partitions.
- Can help improve partition balance.

### Example: Increasing Partitions

```python
print(df.rdd.getNumPartitions())

df = df.repartition(10)

print(df.rdd.getNumPartitions())
```

If the original DataFrame has 4 partitions, repartitioning creates 10 partitions.

### Repartition by Column

```python
df = df.repartition(
    10,
    "department"
)
```

Spark redistributes records using the specified partitioning expression.

Records with the same department value are directed to the same hash partition.

**Important:** Repartitioning by a skewed column does not automatically solve data skew. A hot key can still concentrate records in one partition.

## 3. What Is coalesce()?

`coalesce()` is primarily used to reduce the number of partitions without a full shuffle.

Example:

```python
df_coalesced = df.coalesce(5)
```

If the original DataFrame has 20 partitions, coalescing to 5 reduces the partition count.

### Key Characteristics

- Primarily reduces partitions.
- Usually avoids a full shuffle.
- Combines existing partitions.
- Can produce uneven partition sizes.
- Cannot increase the number of partitions.

### Example

```python
print(df.rdd.getNumPartitions())

df = df.coalesce(5)

print(df.rdd.getNumPartitions())
```

If the original DataFrame has 20 partitions, the resulting DataFrame has 5 partitions.

**Important:** `coalesce()` may result in fewer partitions than requested in certain situations, such as when the requested count is greater than the existing count. It does not increase the partition count.

## 4. Visual Difference

### repartition()

Suppose the original DataFrame has four partitions.

```text
Before:

P1: A, B
P2: C, D
P3: E, F
P4: G, H

       |
       | Full shuffle
       v

After repartition(2):

P1: A, C, E, G
P2: B, D, F, H
```

This is an illustrative redistribution. Actual record placement depends on the partitioning strategy.

### coalesce()

Suppose the original DataFrame has four partitions.

```text
Before:

P1: A, B
P2: C, D
P3: E, F
P4: G, H

       |
       | Combine partitions
       v

After coalesce(2):

P1: A, B, C, D
P2: E, F, G, H
```

This illustrates partition combining without a full redistribution.

## 5. repartition() vs coalesce()

| Feature | repartition() | coalesce() |
|---|---|---|
| Main purpose | Redistribute data | Reduce partitions |
| Increase partitions | Yes | No |
| Decrease partitions | Yes | Yes |
| Full shuffle | Generally yes | Usually no |
| Data redistribution | Broad redistribution | Combines existing partitions |
| Partition balance | Can improve balance | May produce uneven partitions |
| Performance cost | Higher due to shuffle | Usually lower |
| Common use | Rebalancing or increasing partitions | Reducing partitions before writing |

**Remember:**

- Need more partitions? Use `repartition()`.
- Need fewer partitions without a full shuffle? Use `coalesce()`.
- Need to redistribute data evenly? Consider `repartition()`.

## 6. repartition() with Multiple Columns

You can repartition using more than one column.

```python
df = df.repartition(
    10,
    "department",
    "location"
)
```

Spark uses the specified columns as partitioning expressions.

Records with the same combination of department and location values are directed to the same hash partition.

Example:

| department | location |
|---|---|
| IT | Kolkata |
| IT | Kolkata |
| HR | Delhi |
| HR | Mumbai |

Records with the same combination are assigned to the same target partition.

## 7. repartition() vs coalesce() Before Writing Data

Partition count can affect the number of output files produced when writing a DataFrame.

### Example 1: Use coalesce()

Suppose a DataFrame has 100 partitions, but you want fewer output files.

```python
df.coalesce(10).write.mode(
    "overwrite"
).parquet("/output/sales")
```

This reduces the number of partitions before writing and can reduce the number of output files.

### Example 2: Use repartition()

Suppose you want to redistribute data into 10 partitions before writing.

```python
df.repartition(10).write.mode(
    "overwrite"
).parquet("/output/sales")
```

This performs a shuffle and redistributes the records.

### Important Consideration

- `coalesce()` is often useful when reducing output partitions cheaply.
- `repartition()` is useful when data needs redistribution.
- Too many partitions can create many small files.
- Too few partitions can create large files and reduce write parallelism.

The number of output files is influenced by the final number of non-empty partitions and the write operation. Do not assume one partition always produces exactly one file in every writing scenario.

## 8. Does repartition() Always Improve Performance?

No.

Although repartitioning can improve parallelism and distribution, it introduces shuffle overhead.

For example:

```python
df = df.repartition(100)
```

If the dataset is small, creating 100 partitions may increase task scheduling overhead without providing meaningful benefits.

Use repartitioning when there is a clear reason, such as:
- Increasing parallelism.
- Redistributing unevenly distributed data.
- Preparing data for downstream operations.
- Controlling output partitioning.

## 9. Does coalesce() Always Improve Performance?

No.

Reducing partitions can lower task scheduling overhead, but it can also create large or uneven partitions.

For example:

```python
df = df.coalesce(2)
```

If the DataFrame is very large, only two tasks may process the data.

This can:
- Reduce parallelism.
- Increase task duration.
- Increase memory pressure.
- Create uneven execution.

Use coalesce only when reducing partitions is appropriate.

## 10. Important: DataFrame coalesce() vs functions.coalesce()

These are two different Spark operations.

### A. DataFrame coalesce()

Changes the number of partitions.

```python
df = df.coalesce(5)
```

### B. functions.coalesce()

Returns the first non-NULL value from a list of columns or expressions.

```python
from pyspark.sql.functions import coalesce, col, lit

df = df.withColumn(
    "final_salary",
    coalesce(
        col("salary"),
        col("backup_salary"),
        lit(0)
    )
)
```

Example:

| salary | backup_salary | final_salary |
|---:|---:|---:|
| 50000 | 45000 | 50000 |
| NULL | 45000 | 45000 |
| NULL | NULL | 0 |

**Interview warning:** Do not confuse DataFrame `coalesce()` with SQL/PySpark `coalesce()` for handling NULL values.

## 11. How to Check the Number of Partitions?

Use:

```python
print(df.rdd.getNumPartitions())
```

After repartitioning:

```python
df_repartitioned = df.repartition(10)

print(
    df_repartitioned.rdd.getNumPartitions()
)
```

After coalescing:

```python
df_coalesced = df.coalesce(5)

print(
    df_coalesced.rdd.getNumPartitions()
)
```

You can also inspect the physical plan:

```python
df.explain("formatted")
```

Look for partitioning and exchange information.

A repartition operation commonly introduces an `Exchange` operator.

## 12. Common Mistakes

1. Assuming `coalesce()` can increase partitions.
2. Assuming `repartition()` does not cause shuffle.
3. Using repartitioning without checking whether redistribution is necessary.
4. Reducing a large dataset to too few partitions.
5. Assuming `coalesce()` always produces balanced partitions.
6. Assuming repartitioning by a hot key automatically solves skew.
7. Confusing DataFrame `coalesce()` with `functions.coalesce()`.
8. Assuming the number of partitions always equals the exact number of output files.

## 13. Interview Questions

**Q1. What is the difference between repartition() and coalesce()?**

`repartition()` generally performs a full shuffle and can increase or decrease partitions. `coalesce()` primarily reduces partitions without a full shuffle.

**Q2. Can repartition() increase the number of partitions?**

Yes. For example, `df.repartition(20)` creates 20 partitions.

**Q3. Can coalesce() increase the number of partitions?**

No. It is primarily used to reduce the partition count.

**Q4. Why is repartition() more expensive?**

Because it generally redistributes data through a shuffle, involving network and disk I/O.

**Q5. Why is coalesce() usually faster?**

It generally combines existing partitions without a full shuffle.

**Q6. When would you use repartition()?**

When you need to increase parallelism or redistribute data across partitions.

**Q7. When would you use coalesce()?**

When you want to reduce the number of partitions, for example, before writing output with fewer files.

**Q8. Does repartition() solve data skew?**

Not necessarily. Repartitioning by the same hot key can still send all records for that key to one partition.

**Q9. What is the difference between DataFrame coalesce() and functions.coalesce()?**

DataFrame `coalesce()` changes the number of partitions. `functions.coalesce()` returns the first non-NULL value among expressions.

**Q10. How do you check the number of partitions?**

Use `df.rdd.getNumPartitions()`.

## 14. Practice Questions

1. Your DataFrame has 100 partitions, and you want to reduce them to 10 before writing. Which method would you use?
2. Your DataFrame has 5 partitions, and you want to increase them to 20. Which method would you use?
3. Why does repartition() generally cause a shuffle?
4. Why can coalesce() create uneven partitions?
5. You have a large DataFrame and use `coalesce(2)`. What performance problems could occur?
6. Does repartitioning by customer_id solve a hot-key problem?
7. Explain the difference between `df.coalesce(5)` and `functions.coalesce()`.
8. How would you decide the appropriate partition count before writing Parquet files?

## 15. Quick Revision

- **repartition():** Redistributes data across partitions.
- **coalesce():** Primarily reduces the number of partitions.
- **repartition():** Generally performs a full shuffle.
- **coalesce():** Usually avoids a full shuffle.
- **Increase partitions:** Use `repartition()`.
- **Decrease partitions:** Use `repartition()` or `coalesce()`.
- **Improve distribution:** Consider `repartition()`.
- **Reduce output files:** Consider `coalesce()`.
- **Check partition count:** `df.rdd.getNumPartitions()`.
- **DataFrame coalesce():** Changes partitions.
- **functions.coalesce():** Handles NULL values.

## 16. Self-Check

- [ ] Explain repartition() and coalesce().
- [ ] Explain why repartition() generally causes shuffle.
- [ ] Explain why coalesce() usually avoids a full shuffle.
- [ ] Understand how to increase and decrease partitions.
- [ ] Explain how partition count affects output files.
- [ ] Understand the performance risks of too few or too many partitions.
- [ ] Differentiate DataFrame coalesce() and functions.coalesce().
- [ ] Explain why repartitioning does not automatically solve data skew.

**Interview goal:** Be able to choose between `repartition()` and `coalesce()` based on data distribution, shuffle cost, parallelism, and output requirements.