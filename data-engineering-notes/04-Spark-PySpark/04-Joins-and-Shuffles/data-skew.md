# Data Skew in Apache Spark

## 1. What Is Data Skew?

**Data skew** occurs when data is distributed unevenly across Spark partitions.

Some partitions contain significantly more data than others.

As a result:
- Some tasks finish quickly.
- Some tasks take much longer.
- A few executors remain busy while others become idle.
- Overall job execution time increases.

**Simple interview definition:**

Data skew is an uneven distribution of data across Spark partitions, causing some tasks to process significantly more data than others and creating performance bottlenecks.

### Simple Example

Suppose we have 100 GB of data distributed across four partitions.

**Balanced distribution:**

| Partition | Data size |
|---|---:|
| P1 | 25 GB |
| P2 | 25 GB |
| P3 | 25 GB |
| P4 | 25 GB |

**Skewed distribution:**

| Partition | Data size |
|---|---:|
| P1 | 5 GB |
| P2 | 5 GB |
| P3 | 5 GB |
| P4 | 85 GB |

In the skewed example, P4 takes much longer to process.

Even if three partitions finish quickly, the Spark stage cannot finish until the slow task completes.

This is commonly called the **straggler task problem**.

---

## 2. Why Does Data Skew Occur?

Data skew usually occurs when certain partition keys have significantly more records than others.

Common causes include:

### 2.1 Uneven Key Distribution

Suppose we have an orders dataset:

| order_id | country |
|---:|---|
| 101 | India |
| 102 | India |
| 103 | India |
| 104 | India |
| 105 | India |
| 106 | USA |
| 107 | UK |
| 108 | Germany |

Imagine the actual dataset contains:
- India: 90 million records.
- USA: 5 million records.
- UK: 3 million records.
- Germany: 2 million records.

If we group or partition by country, the India key may receive much more data than the other keys.

### 2.2 Low-Cardinality Columns

Columns with very few distinct values can create uneven partitions when their values are not evenly distributed.

Examples:
- Gender.
- Country.
- Status.
- Region.
- Boolean flags.

Low cardinality does not automatically mean skew. The important factor is whether the values have an uneven frequency distribution.

### 2.3 Join Key Skew

Suppose two large datasets are joined using `customer_id`.

If one customer ID appears millions of times, a large amount of matching data may be concentrated in a single shuffle partition.

This can make a particular join task extremely slow.

### 2.4 Null or Default Values

Suppose millions of records have:

```text
customer_id = NULL
```

or:

```text
customer_id = 0
```

If these values are used as partition or join keys, they may concentrate a large amount of data into a small number of partitions.

**Important:** The exact handling of null join keys depends on the join condition. Ordinary SQL equality joins do not match NULL to NULL.

---

## 3. How Does Data Skew Affect Spark Performance?

Data skew can cause:

1. **Uneven task duration:** Some tasks finish much later than others.
2. **Long-running stages:** The stage waits for its slowest task.
3. **Memory pressure:** Large partitions may require more execution memory.
4. **Disk spilling:** Large operations may spill intermediate data to disk.
5. **Executor failures:** Extreme memory pressure can cause out-of-memory errors.
6. **Poor resource utilization:** Some executors may remain idle while others process oversized partitions.

### Visual Example

```text
       Spark Stage
            |
    -------------------
    |    |    |       |
    v    v    v       v
   P1   P2   P3      P4
    |    |    |       |
    v    v    v       v
  10s  12s  11s     180s
    |    |    |       |
    -------------------
            |
            v
       Stage Complete
          180s
```

The entire stage is delayed by the slowest task.

---

## 4. How to Identify Data Skew

Before applying optimization, identify whether data skew is actually the problem.

### 4.1 Check Spark UI

Open the Spark UI and inspect:

- Stages tab.
- Tasks tab.
- SQL/DataFrame tab.
- Executors tab.

Look for:
- Large differences in task duration.
- Large differences in shuffle read size.
- Large differences in input size.
- High memory spill or disk spill.
- A few unusually slow tasks.

Example:

| Task | Duration | Shuffle Read |
|---|---:|---:|
| Task 1 | 10 sec | 100 MB |
| Task 2 | 12 sec | 110 MB |
| Task 3 | 11 sec | 105 MB |
| Task 4 | 180 sec | 2 GB |

Task 4 may indicate a skewed partition.

### 4.2 Check Key Distribution

In PySpark:

```python
from pyspark.sql.functions import count, col

df.groupBy("customer_id") \
  .agg(count("*").alias("record_count")) \
  .orderBy(col("record_count").desc()) \
  .show(10)
```

This helps identify keys with unusually high record counts.

### 4.3 Check Partition Sizes

For a shuffled DataFrame, you can inspect the number of partitions:

```python
df.rdd.getNumPartitions()
```

However, the number of partitions alone does not tell you whether the data is evenly distributed.

Use Spark UI metrics and key-frequency analysis together.

---

## 5. Techniques to Handle Data Skew

There are several ways to handle data skew. Choose the technique based on the operation causing the problem.

### 5.1 Salting Technique

**Salting** means adding an additional value to a skewed key to distribute its records across multiple partitions.

This is especially useful for skewed joins.

#### Problem

Suppose a customer ID appears millions of times:

```text
customer_id = 100
```

All records for that key may end up in the same shuffle partition.

#### Solution

Create a salt column.

Example:

```python
from pyspark.sql.functions import col, rand, floor

large_df = large_df.withColumn(
    "salt",
    floor(rand() * 10).cast("int")
)
```

Now the same customer ID can be divided into multiple combinations:

```text
(100, 0)
(100, 1)
(100, 2)
...
(100, 9)
```

These combinations can be distributed across different partitions.

#### Important: Salt Both Sides Correctly

Suppose we have:
- A large orders DataFrame.
- A small customer DataFrame.

If we salt the large DataFrame randomly, the matching customer records must be available for every salt value.

One approach is to replicate the relevant small-side records across all salt values.

Conceptual example:

```python
from pyspark.sql.functions import explode, sequence, lit

salt_values = sequence(lit(0), lit(9))

customers_salted = customers.withColumn(
    "salt",
    explode(salt_values)
)

orders_salted = orders.withColumn(
    "salt",
    floor(rand() * 10).cast("int")
)

result = orders_salted.join(
    customers_salted,
    ["customer_id", "salt"],
    "inner"
)
```

Now the join uses both `customer_id` and `salt`.

**Important points:**
- Use the same salt range on both sides.
- Replicate the small-side matching records across all salt values.
- Random salting is suitable for distributing a large number of records, but deterministic hashing can be useful when repeatability matters.
- Salting increases the size of the replicated side and adds processing overhead.

### 5.2 Broadcast Join

If one side of the join is genuinely small, use a broadcast join.

```python
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    "customer_id",
    "inner"
)
```

Why can it help?

- Avoids shuffling both sides in a suitable broadcast hash join.
- Removes the need to redistribute the large dataset by join key.
- Can reduce the impact of shuffle-based join skew.

**Limitation:** Broadcasting does not automatically solve every skew problem. A very large broadcast side can create memory pressure.

### 5.3 Adaptive Query Execution (AQE)

AQE can detect certain skewed shuffle partitions at runtime and optimize skewed joins.

Relevant configurations:

```text
spark.sql.adaptive.enabled
```

```text
spark.sql.adaptive.skewJoin.enabled
```

In standard Spark SQL configurations, AQE is enabled by default since Spark 3.2, while the skew-join optimization is also enabled by default when AQE is enabled. Platform settings may differ.

AQE can split a skewed partition into smaller pieces and process them in separate tasks.

#### Example

Original partition distribution:

```text
P1 = 100 MB
P2 = 120 MB
P3 = 150 MB
P4 = 5 GB
```

After skew optimization, the large partition may be divided into smaller pieces:

```text
P1 = 100 MB
P2 = 120 MB
P3 = 150 MB
P4a = 1 GB
P4b = 1 GB
P4c = 1 GB
P4d = 1 GB
P4e = 1 GB
```

This is an illustrative example, not a guaranteed split size.

**Important:** AQE skew-join optimization primarily targets eligible sort-merge and shuffled-hash joins. It does not automatically solve every skew problem, such as all aggregation or custom-processing bottlenecks.

### 5.4 Repartitioning

Repartitioning redistributes data across partitions.

Example:

```python
df = df.repartition(200)
```

You can also repartition by a column:

```python
df = df.repartition(200, "customer_id")
```

**Important:** Repartitioning by the same skewed key may reproduce the skew because records with the same key still need to go to the same partition.

Increasing the partition count alone does not guarantee that skew will disappear.

### 5.5 Increase Shuffle Partitions

Spark SQL provides:

```text
spark.sql.shuffle.partitions
```

Example:

```python
spark.conf.set(
    "spark.sql.shuffle.partitions",
    "400"
)
```

Increasing the number of shuffle partitions can reduce the average amount of data per partition.

However, it does not necessarily solve hot-key skew. Records with the same key can still be concentrated in one partition.

Use it when the overall shuffle partitions are too large, not as the only solution for a heavily skewed key.

---

## 6. Data Skew vs Uneven Partitioning

These terms are related but not identical.

| Data Skew | Uneven Partitioning |
|---|---|
| Uneven distribution of records or processing work. | Unequal partition sizes. |
| Often caused by frequent or hot keys. | Can result from skew or partitioning choices. |
| Can cause slow tasks and memory pressure. | Can lead to inefficient resource utilization. |
| May require salting or AQE skew optimization. | May require better partitioning or redistribution. |

---

## 7. Data Skew vs Data Spill

| Data Skew | Data Spill |
|---|---|
| Uneven distribution of data or work. | Intermediate data is written to disk because it cannot remain entirely in memory. |
| Can cause some tasks to run much longer. | Can increase processing time. |
| Can contribute to memory pressure. | Can occur because of memory pressure. |
| A distribution problem. | A memory-management event. |

**Remember:** Data skew can cause data spill, but data spill can also occur without skew.

---

## 8. Practical Data Engineering Scenario

Suppose you have two datasets:

**Orders:** 500 million records.

**Customers:** 10 million records.

You join them using `customer_id`.

After execution, you notice:
- Most tasks finish in 20 seconds.
- A few tasks take 5 minutes.
- Some tasks have very large shuffle reads.
- The Spark UI shows high disk spill for those tasks.

### How Would You Investigate?

1. Open the Spark UI.
2. Identify the slow stage.
3. Compare task duration and shuffle read sizes.
4. Check the frequency distribution of `customer_id`.
5. Identify whether a few customer IDs dominate the data.

### How Would You Fix It?

Depending on the findings:

1. Enable or verify AQE skew-join optimization.
2. Use salting if a hot join key remains a bottleneck.
3. Consider broadcasting a genuinely small side of the join.
4. Review partition counts and data distribution.
5. Check whether null or default keys are causing concentration.

**Interview tip:** Always explain how you identified the problem before explaining the optimization.

---

## 9. Important Interview Questions

### Q1. What is data skew in Spark?

Data skew occurs when data is distributed unevenly across partitions, causing some tasks to process significantly more data than others.

### Q2. Why does data skew reduce performance?

Some tasks take much longer than others. The stage must wait for the slowest tasks, increasing overall job execution time.

### Q3. What causes data skew?

Common causes include:
- Uneven key distribution.
- Hot join keys.
- Low-cardinality columns with uneven frequencies.
- Large numbers of NULL or default values.

### Q4. How do you identify data skew?

Use Spark UI to compare task duration, shuffle read, input size, and spill metrics. Also inspect key-frequency distributions.

### Q5. What is salting?

Salting adds an additional value to a skewed key to distribute records across multiple partitions.

### Q6. How does AQE handle skew?

AQE can detect eligible skewed shuffle partitions during execution and split them into smaller pieces for parallel processing.

### Q7. Does increasing shuffle partitions solve data skew?

Not necessarily. It may reduce average partition size, but a hot key can still concentrate records in one partition.

### Q8. Can broadcast joins help with data skew?

Yes. Broadcasting a sufficiently small side can avoid shuffling both sides in a suitable join, reducing shuffle-related skew.

### Q9. What is a hot key?

A hot key is a key that appears much more frequently than other keys and can create a processing bottleneck.

### Q10. What is the difference between data skew and data spill?

Data skew is an uneven distribution problem. Data spill occurs when intermediate data is written to disk because of insufficient available memory.

### Q11. Does AQE solve every type of data skew?

No. AQE handles certain eligible skewed join operations, but other operations may require techniques such as salting or changes to data partitioning.

### Q12. Why can repartitioning by a skewed key fail to solve skew?

Because all records with the same key are generally assigned to the same partition when partitioning by that key.

---

## 10. Common Mistakes

1. **Increasing partitions without checking key distribution:** A hot key may still end up in one partition.

2. **Using salting without modifying both join sides correctly:** The join may lose matching records or produce incorrect results.

3. **Broadcasting a large dataset:** This can create executor memory pressure.

4. **Assuming AQE solves every skew problem:** AQE has specific supported optimization scenarios.

5. **Confusing data skew with data spill:** Skew concerns distribution; spill concerns memory usage.

6. **Ignoring Spark UI metrics:** Always investigate task duration and shuffle size before changing the code.

7. **Using random salting without considering repeatability:** A deterministic salt can be more appropriate when reproducible partition assignment is needed.

---

## 11. Quick Revision

| Concept | Remember |
|---|---|
| Data Skew | Uneven data distribution across partitions. |
| Hot Key | A key with unusually high frequency. |
| Straggler Task | A task that takes much longer than others. |
| Main Impact | Slow stages, memory pressure, and possible spill. |
| Spark UI | Helps identify skewed tasks and stages. |
| Salting | Adds extra key values to distribute hot-key records. |
| Broadcast Join | Can avoid shuffling both sides in suitable joins. |
| AQE | Can split eligible skewed shuffle partitions. |
| Repartition | Redistributes data across partitions. |
| Shuffle Partitions | Controls the default number of shuffle partitions for relevant SQL operations. |
| Data Spill | Writes intermediate data to disk under memory pressure. |

### One-Minute Interview Answer

Data skew occurs when some Spark partitions contain significantly more records than others. For example, if one customer ID appears millions of times, a join or aggregation involving that key may create one very large partition. This causes some tasks to run much longer than others and delays the entire stage. I would first identify the issue using Spark UI by comparing task duration, shuffle read, and spill metrics. Depending on the situation, I would use AQE skew-join optimization, salting, broadcast joins, or appropriate repartitioning. I would not simply increase the partition count without checking the key distribution.

---

## 12. Practice Questions

Try answering without looking at the notes.

1. What is data skew?
2. Why does one skewed partition delay an entire Spark stage?
3. What is a hot key?
4. How do you identify skew using Spark UI?
5. Explain salting with a join example.
6. Why must both sides be handled correctly during salting?
7. How does AQE optimize skewed joins?
8. Why does increasing shuffle partitions not always solve skew?
9. When can broadcast joins help with skew?
10. What is the difference between data skew and data spill?
11. How would you troubleshoot a join where only a few tasks are very slow?
12. Which technique would you choose if one customer ID dominates a large join?

## 13. Self-Assessment

- [ ] I can explain data skew in simple language.
- [ ] I understand how hot keys create skew.
- [ ] I can identify skew using Spark UI.
- [ ] I can explain salting with a practical example.
- [ ] I understand AQE skew-join optimization.
- [ ] I know why repartitioning alone may not solve skew.
- [ ] I can distinguish data skew from data spill.
- [ ] I can explain a real-world skew troubleshooting approach.

**Interview readiness:** You should be able to explain why a skewed join creates slow tasks, identify the issue using Spark UI, and compare salting, broadcast joins, and AQE as possible solutions.