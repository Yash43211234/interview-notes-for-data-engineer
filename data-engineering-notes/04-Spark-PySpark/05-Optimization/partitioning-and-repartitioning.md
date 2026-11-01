# Partitioning and Repartitioning in PySpark

## 1. What Is Partitioning?

Partitioning is the process of dividing a large dataset into smaller logical chunks called **partitions**.

Spark processes these partitions in parallel across available executors.

For example, imagine a dataset containing 10 million records:

- Partition 1 → 1 million records
- Partition 2 → 1 million records
- Partition 3 → 1 million records
- ...
- Partition 10 → 1 million records

Instead of processing the entire dataset on one machine, Spark can process multiple partitions in parallel.

**Key idea:** Partitions are the basic units of parallel processing in Spark.

## 2. Why Does Spark Use Partitions?

Partitioning helps Spark:

1. Process large datasets in parallel.
2. Distribute workloads across executors.
3. Improve resource utilization.
4. Control data distribution during operations.
5. Reduce processing time when partitions are balanced.

However, having more partitions does not always mean better performance.

Too few partitions can limit parallelism, while too many can create scheduling overhead and small tasks.

## 3. How Does Partitioning Work?

A Spark DataFrame is distributed across partitions.

Each partition contains a subset of the dataset, and Spark schedules tasks to process those partitions.

For example:

```python
df = spark.read.parquet("/data/orders")

print(df.rdd.getNumPartitions())
```

To inspect the number of partitions:

```python
df.rdd.getNumPartitions()
```

To display the records:

```python
df.show()
```

**Important:**

- A partition is a logical division of distributed data.
- A task generally processes one partition for a particular stage.
- Multiple tasks can run simultaneously depending on available resources.
- The number of partitions does not necessarily equal the number of executors.

## 4. What Is Repartitioning?

Repartitioning means changing the number or distribution of partitions in a DataFrame.

PySpark provides the `repartition()` method.

### Example: Increase the number of partitions

```python
df = df.repartition(10)

print(df.rdd.getNumPartitions())
```

This requests 10 partitions.

### Example: Repartition using a column

```python
df = df.repartition(10, "customer_id")
```

Spark distributes records based on the specified column using hash partitioning.

Records with the same partitioning key are assigned to the same partition in this repartitioning operation.

### Example: Repartition using multiple columns

```python
df = df.repartition(10, "country", "city")
```

Spark uses the combination of the specified columns to determine partition placement.

**Important:** `repartition()` generally triggers a shuffle because Spark redistributes data across partitions.

## 5. What Is Coalesce?

`coalesce()` is used to reduce the number of partitions, generally without performing a full shuffle.

Example:

```python
df = df.coalesce(4)

print(df.rdd.getNumPartitions())
```

If the original DataFrame has 10 partitions, this requests a reduction to 4.

Coalesce is useful when:
- The dataset has too many partitions.
- You want fewer output files.
- A full shuffle is unnecessary.

However, because it generally avoids a full shuffle, the resulting partitions may not have evenly distributed data.

## 6. Repartition vs Coalesce

| Feature | repartition() | coalesce() |
|---|---|---|
| Main purpose | Change partition count/distribution | Reduce partition count |
| Increase partitions | Yes | No |
| Reduce partitions | Yes | Yes |
| Full shuffle | Generally yes | Generally no |
| Data distribution | Can redistribute data | May remain uneven |
| Performance cost | Usually higher | Usually lower |
| Common use | Increase parallelism or redistribute data | Reduce partitions before writing |

### Example

Suppose a DataFrame has 20 partitions.

**Using repartition:**

```python
df_repartitioned = df.repartition(5)
```

Spark redistributes the data into 5 partitions, generally through a shuffle.

**Using coalesce:**

```python
df_coalesced = df.coalesce(5)
```

Spark reduces the partition count without a full shuffle in the normal case.

### Interview answer

`repartition()` performs a shuffle to redistribute data and can increase or decrease the number of partitions. `coalesce()` generally avoids a full shuffle and is mainly used to reduce the number of partitions. Repartitioning is useful when better distribution is needed, while coalesce is useful when reducing partitions with less overhead.

## 7. What Is Hash Partitioning?

Hash partitioning distributes records based on the hash value of one or more columns.

Example:

```python
df = df.repartition(8, "customer_id")
```

Conceptually:

1. Spark reads the `customer_id`.
2. It calculates a hash for the partitioning key.
3. It maps the hash to one of the target partitions.
4. Records with the same key are assigned to the same partition.

This is useful for operations such as joins and aggregations because matching keys can be colocated in partitions.

**Important:** Hash partitioning does not guarantee equal numbers of records in every partition. Uneven key frequencies can cause data skew.

## 8. What Is Data Skew in Partitioning?

Data skew occurs when data is distributed unevenly across partitions.

Example:

| Partition | Records |
|---|---:|
| P1 | 1,000 |
| P2 | 1,200 |
| P3 | 1,100 |
| P4 | 500,000 |

Partition 4 contains significantly more records than the others.

As a result:
- Some tasks finish quickly.
- One or more tasks take much longer.
- Executors may experience memory pressure.
- The entire stage waits for slow tasks to finish.

This is called the **straggler task problem**.

### Why can skew happen?

- One key occurs much more frequently than others.
- A few customers generate most of the records.
- Many records share a NULL or default key.
- Data has low cardinality.

### How can we handle skew?

1. Use salting for heavily skewed keys.
2. Use broadcast joins when one side is sufficiently small.
3. Enable or use Adaptive Query Execution (AQE) skew handling where applicable.
4. Process heavily skewed keys separately.
5. Pre-aggregate data when appropriate.

**Important:** Repartitioning by the same skewed key does not solve the underlying skew. Records with the same key can still end up together.

## 9. Partitioning vs Storage Partitioning

These two concepts are different.

### Spark execution partitioning

This controls how data is distributed for Spark processing.

Example:

```python
df = df.repartition(8)
```

It changes the number and distribution of Spark partitions.

### Storage partitioning

This organizes data into directories based on column values when writing files.

Example:

```python
df.write \
    .partitionBy("country") \
    .parquet("/data/orders")
```

This can create a directory structure like:

```text
orders/
    country=India/
    country=USA/
    country=UK/
```

Storage partitioning can improve query performance through partition pruning when queries filter on partition columns.

**Important:** `partitionBy()` is not the same as `repartition()`.

- `repartition()` controls Spark execution partitions.
- `partitionBy()` organizes output data into storage directories.

High-cardinality storage partition columns can create too many directories and small files.

## 10. How Many Partitions Should We Use?

There is no universal ideal partition count.

It depends on:

- Dataset size.
- Available CPU cores.
- Executor resources.
- Type of transformation.
- Shuffle volume.
- Data distribution.
- Task execution time.

### Practical considerations

- Too few partitions can reduce parallelism.
- Too many partitions can increase scheduling overhead.
- Very large partitions can create memory pressure and long-running tasks.
- Uneven partitions can cause stragglers.

A practical approach is to inspect Spark UI task durations, data sizes, shuffle metrics, and available resources before changing the partition count.

Do not blindly set the number of partitions based only on the number of executors.

## 11. Repartitioning Before Writing Data

Repartitioning can be useful before writing output files.

Example:

```python
df = df.repartition(8)

df.write \
    .mode("overwrite") \
    .parquet("/output/orders")
```

This redistributes data into approximately 8 Spark partitions before the write.

Another example:

```python
df = df.coalesce(4)

df.write \
    .mode("overwrite") \
    .parquet("/output/orders")
```

This reduces the number of Spark partitions before writing.

**Remember:**

- Spark writes data into output directories containing part files.
- The number of output files is influenced by execution partitions and write behavior.
- It is not guaranteed to equal the requested partition count in every situation.
- Storage partitioning with `partitionBy()` can produce multiple files within each directory.

## 12. Common Mistakes

1. Assuming more partitions always improve performance.
2. Assuming `coalesce()` redistributes data evenly.
3. Assuming `repartition()` is always better than `coalesce()`.
4. Using repartitioning without considering shuffle cost.
5. Assuming repartitioning by a skewed key removes skew.
6. Confusing `repartition()` with `partitionBy()`.
7. Assuming the number of partitions equals the number of executors.
8. Blindly setting partition counts without checking Spark UI metrics.
9. Creating too many storage partitions using high-cardinality columns.

## 13. Interview Questions

**Q1. What is partitioning in Spark?**

Partitioning divides a dataset into smaller chunks so Spark can process them in parallel across available resources.

**Q2. What is repartitioning?**

Repartitioning changes the number or distribution of partitions. It generally involves a shuffle.

**Q3. What is the difference between repartition and coalesce?**

Repartition generally performs a shuffle and can increase or decrease partitions. Coalesce generally avoids a full shuffle and is mainly used to reduce partitions.

**Q4. Does repartition always perform a shuffle?**

The DataFrame `repartition()` operation generally introduces a shuffle to redistribute data.

**Q5. Can coalesce increase the number of partitions?**

No. DataFrame `coalesce()` is used to reduce the number of partitions.

**Q6. What happens if we have too many partitions?**

It can increase task scheduling overhead and create many small tasks and output files.

**Q7. What happens if we have too few partitions?**

It can reduce parallelism, create larger tasks, and increase memory pressure.

**Q8. What is hash partitioning?**

It distributes records based on the hash value of specified columns.

**Q9. Does repartitioning by customer_id solve data skew?**

Not necessarily. If one customer ID occurs much more frequently than others, those records can still be concentrated in one partition.

**Q10. What is the difference between repartition and partitionBy?**

Repartition changes Spark execution partitions. `partitionBy()` organizes files into storage directories based on column values.

**Q11. How do you identify partition imbalance?**

Check task duration, input size, shuffle read/write, and spill metrics in Spark UI.

**Q12. When would you use coalesce before writing?**

When the DataFrame has more partitions than needed and reducing them can avoid producing excessive small output files.

## 14. Practice Questions

1. A DataFrame has 100 partitions. You need 20 partitions. Which method would you choose and why?
2. A DataFrame has 10 partitions. You need 30 partitions. Which method would you use?
3. A Spark job has one task running much longer than the others. What would you investigate?
4. You repartition by `customer_id`, but one partition is still much larger. Why?
5. What is the difference between `df.repartition(10, "country")` and `df.write.partitionBy("country")`?
6. Why might repartitioning before writing improve output file distribution?
7. Why can too many partitions negatively affect Spark performance?

## 15. Quick Revision

| Concept | Remember |
|---|---|
| Partition | Basic unit of distributed processing |
| Parallelism | Processing multiple partitions simultaneously |
| repartition() | Redistributes data, generally with a shuffle |
| coalesce() | Reduces partitions, generally without a full shuffle |
| Hash partitioning | Distributes records based on hash values |
| Data skew | Uneven data distribution across partitions |
| partitionBy() | Creates storage directories based on column values |
| Too few partitions | Less parallelism and potentially larger tasks |
| Too many partitions | More scheduling overhead and potentially small files |
| Spark UI | Helps investigate task and partition imbalance |

## 16. Self-Check

Before moving to the next topic, make sure you can:

- [ ] Explain why Spark uses partitions.
- [ ] Explain how repartitioning works.
- [ ] Explain the difference between repartition and coalesce.
- [ ] Explain hash partitioning.
- [ ] Identify data skew and its causes.
- [ ] Explain why repartitioning cannot always fix skew.
- [ ] Differentiate execution partitioning from storage partitioning.
- [ ] Explain how partition counts affect performance.
- [ ] Diagnose partition imbalance using Spark UI.

**Final takeaway:** Partitioning controls how Spark distributes data for parallel processing. Repartitioning redistributes data, coalesce reduces partitions with less shuffle overhead, and good partitioning helps improve parallelism and performance.