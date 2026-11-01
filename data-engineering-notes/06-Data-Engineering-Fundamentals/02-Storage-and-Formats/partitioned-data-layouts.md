# Partitioned Data Layouts

## 1. Introduction

**Data partitioning** is the process of dividing a large dataset into smaller logical or physical sections based on one or more columns.

Instead of storing all records together, data is organized into partitions so that processing engines can locate and read relevant data more efficiently.

Partitioning is commonly used in:
- Data lakes.
- Data warehouses.
- Apache Spark.
- Databricks.
- Hive tables.
- Delta Lake.
- Large-scale ETL pipelines.

### Simple Example

Suppose an e-commerce company stores 100 million order records.

Each order contains:

| order_id | customer_id | order_date | amount |
|---:|---:|---|---:|
| 101 | 501 | 2026-01-10 | 500 |
| 102 | 502 | 2026-01-11 | 700 |
| 103 | 503 | 2026-02-05 | 300 |

If the data is partitioned by year and month, the storage layout may look like:

```text
orders/
│
├── year=2026/
│   ├── month=01/
│   │   ├── part-0001.parquet
│   │   └── part-0002.parquet
│   │
│   ├── month=02/
│   │   ├── part-0003.parquet
│   │   └── part-0004.parquet
│   │
│   └── month=03/
│       └── part-0005.parquet
```

If a query needs only January 2026 data, the engine may skip the other month directories.

This is called **partition pruning**.

---

## 2. Why Do We Need Partitioning?

Without an appropriate data layout, a query may need to scan a large amount of data even when it only needs a small subset.

Partitioning can reduce unnecessary data reads.

### Main Benefits

1. **Faster queries:** Relevant partitions can be read without scanning unrelated partitions.
2. **Partition pruning:** Filters on partition columns can eliminate unnecessary directories.
3. **Better data organization:** Large datasets become easier to manage.
4. **Efficient incremental processing:** Pipelines can process specific date ranges or partitions.
5. **Easier data lifecycle management:** Certain datasets can be managed by date or another logical category.

### Example

Suppose a table contains five years of order data.

A query requests orders from January 2026:

```sql
SELECT *
FROM orders
WHERE order_date >= '2026-01-01'
  AND order_date < '2026-02-01';
```

If the storage layout is partitioned by year and month, the engine may scan only the relevant partition.

Without a suitable layout, it may need to examine many more files.

**Important:** Partitioning helps only when the query filters can be used to eliminate partitions. It does not automatically make every query faster.

---

## 3. Types of Data Partitioning

There are different ways to divide data. The right method depends on the data distribution and query patterns.

### 3.1 Partitioning by Date

This is one of the most common approaches in data engineering.

Example:

```text
orders/
├── year=2025/
│   ├── month=10/
│   ├── month=11/
│   └── month=12/
│
└── year=2026/
    ├── month=01/
    ├── month=02/
    └── month=03/
```

**Use cases:**
- Transaction records.
- Application logs.
- Clickstream events.
- Historical data.
- Daily or monthly ETL pipelines.

Date partitioning is useful when queries frequently filter by date.

### 3.2 Partitioning by Category

Data can also be divided using a categorical column.

Example:

```text
sales/
├── region=East/
├── region=West/
├── region=North/
└── region=South/
```

**Use cases:**
- Regional sales.
- Country-based transactions.
- Department-level reporting.
- Business-unit datasets.

Avoid partitioning by a column with too many distinct values.

### 3.3 Partitioning by Multiple Columns

A dataset can be partitioned using more than one column.

Example:

```text
orders/
├── year=2026/
│   ├── month=01/
│   │   ├── region=East/
│   │   └── region=West/
│   │
│   └── month=02/
│       ├── region=East/
│       └── region=West/
```

Here, the data is organized by year, month, and region.

In PySpark:

```python
df.write \
    .partitionBy("year", "month") \
    .parquet("/data/orders/")
```

Multiple partition columns can improve filtering when queries commonly use those columns. However, too many partition combinations can create excessive directories and small files.

### 3.4 Hash Partitioning

Hash partitioning distributes records across partitions based on a hash function applied to a key.

Conceptually:

```text
hash(customer_id) % number_of_partitions
```

For example:

```text
customer_id = 101 → Partition 1
customer_id = 102 → Partition 3
customer_id = 103 → Partition 2
```

The actual assignment depends on the hash function and partitioning configuration.

**Use cases:**
- Distributing records by key.
- Parallel processing.
- Join and aggregation preparation.
- Balancing data across Spark partitions when keys are reasonably distributed.

**Important:** Hash partitioning is not the same as creating date-based storage directories. It is commonly used for distributing data across processing partitions.

### 3.5 Range Partitioning

Range partitioning distributes records based on ranges of values.

Example:

```text
customer_id:

Partition 1: 1–1000
Partition 2: 1001–2000
Partition 3: 2001–3000
```

**Use cases:**
- Range-based queries.
- Ordered data processing.
- Sorting and range-oriented workloads.

If the data is unevenly distributed, some partitions may contain much more data than others.

---

## 4. Storage Partitioning vs Spark Execution Partitioning

This distinction is very important for interviews.

The word *partition* is used in two different contexts.

### 4.1 Storage Partitioning

Storage partitioning organizes files into directories based on column values.

Example:

```text
orders/
├── year=2025/
│   └── part-0001.parquet
│
└── year=2026/
    └── part-0002.parquet
```

Created using:

```python
df.write \
    .partitionBy("year") \
    .parquet("/data/orders/")
```

Main purpose:
- Organize data on storage.
- Support partition pruning.
- Reduce unnecessary file reads.

### 4.2 Spark Execution Partitioning

Spark execution partitioning divides data into logical processing partitions distributed across tasks.

Example:

```python
df = df.repartition(8)
```

This creates eight Spark DataFrame partitions for subsequent processing, subject to Spark's execution behavior.

Main purpose:
- Parallel processing.
- Distributing workload.
- Controlling shuffle and task execution.

### Comparison

| Feature | Storage Partitioning | Spark Execution Partitioning |
|---|---|---|
| Purpose | Organize stored data | Distribute processing |
| Location | File storage | Spark execution |
| Example | `partitionBy("year")` | `repartition(8)` |
| Main benefit | Partition pruning | Parallelism |
| Creates directories | Usually yes, for file-based partitioned writes | No, not by itself |
| Controls task count | No direct guarantee | Influences task parallelism |

**Interview point:** `partitionBy()` and `repartition()` are not interchangeable.

---

## 5. Partitioning in PySpark

### 5.1 Create a Partitioned Dataset

```python
df.write \
    .mode("overwrite") \
    .partitionBy("year", "month") \
    .parquet("/data/orders/")
```

This creates a directory-based layout using the selected partition columns.

### 5.2 Read Partitioned Data

```python
df = spark.read.parquet("/data/orders/")

df.show()
```

Spark can discover partition columns from the directory structure when reading the partitioned dataset.

### 5.3 Filter Using Partition Columns

```python
filtered_df = df.filter(
    (df.year == 2026) &
    (df.month == 1)
)
```

When the storage layout and query allow it, Spark can use partition pruning to skip irrelevant directories.

### 5.4 Inspect the Execution Plan

```python
filtered_df.explain("formatted")
```

Look for partition-related information in the scan details, such as `PartitionFilters`.

The exact plan output depends on the Spark version and data source.

### 5.5 Repartition by a Column

```python
df_repartitioned = df.repartition(8, "customer_id")
```

This redistributes data into eight Spark partitions using the specified column as the partitioning key.

It is an execution-level operation and generally involves a shuffle.

### 5.6 Reduce the Number of Spark Partitions

```python
df_coalesced = df.coalesce(4)
```

`coalesce()` can reduce the number of Spark partitions, usually without a full shuffle.

It does not guarantee an exact number of output files in every write scenario.

---

## 6. Partition Pruning

**Partition pruning** is an optimization where Spark or another query engine skips storage partitions that cannot contain records matching the query.

### Example Storage Layout

```text
sales/
├── year=2024/
├── year=2025/
└── year=2026/
```

Query:

```sql
SELECT *
FROM sales
WHERE year = 2026;
```

The engine can skip the 2024 and 2025 directories if it can identify the partition filter.

### Benefits

- Reduces data scanned.
- Reduces unnecessary I/O.
- Can improve query performance.
- Reduces processing work.

### Important Distinction

Partition pruning and data skipping are different.

| Feature | Partition Pruning | Data Skipping |
|---|---|---|
| Works at | Storage partition/directory level | File level |
| Uses | Partition column values | File statistics |
| Example | Skip `year=2024` | Skip files whose `amount` range cannot match |
| Main benefit | Avoid reading irrelevant partitions | Avoid reading irrelevant files |

A query can benefit from both techniques.

---

## 7. Problems with Poor Partitioning

Partitioning is useful, but a poor partitioning strategy can create performance and maintenance problems.

### 7.1 Too Many Partitions

Suppose a dataset has millions of unique customer IDs.

Partitioning by `customer_id` may create an enormous number of directories.

Example:

```text
orders/
├── customer_id=101/
├── customer_id=102/
├── customer_id=103/
├── ...
```

Problems:
- Too many directories.
- Excessive metadata operations.
- Many small files.
- Increased query planning overhead.
- Difficult storage management.

**Solution:** Prefer columns with suitable cardinality and meaningful query patterns.

### 7.2 High-Cardinality Partition Columns

Cardinality means the number of distinct values in a column.

Examples:

| Column | Typical Cardinality |
|---|---|
| Country | Low to moderate |
| Region | Low |
| Year | Low |
| Month | Low |
| Customer ID | Very high |
| Transaction ID | Very high |

A high-cardinality column is generally a poor candidate for directory-based partitioning.

However, cardinality alone does not determine the decision. Data volume, distribution, and query patterns also matter.

### 7.3 Small Files Problem

Suppose a dataset is partitioned by year, month, and region.

If each partition receives very little data, the system may create many small files.

Problems:
- Increased file metadata overhead.
- More file-open operations.
- Slower query planning and execution.
- Inefficient storage operations.

**Solution:**
- Avoid unnecessary partition columns.
- Use appropriate file sizing.
- Compact small files where supported.
- Review write parallelism.

### 7.4 Data Skew

Data skew occurs when records are distributed unevenly across partitions.

Example:

```text
Partition 1: 10,000 records
Partition 2: 12,000 records
Partition 3: 9,000 records
Partition 4: 5,000,000 records
```

One partition contains much more data than the others.

This can cause:
- Uneven task execution times.
- Long-running tasks.
- Resource pressure.
- Poor cluster utilization.

**Solution:**
- Inspect key distribution.
- Use skew-handling techniques where appropriate.
- Consider salting for suitable workloads.
- Avoid relying on a highly skewed key for even distribution.

**Important:** Increasing the number of partitions does not automatically eliminate skew.

---

## 8. How to Choose a Partition Column

Before choosing a partition column, examine the following:

### 1. Query Patterns

Which columns are frequently used in filters?

Example:

```sql
SELECT *
FROM orders
WHERE order_date = '2026-01-10';
```

A date-based layout may help this workload.

### 2. Cardinality

Avoid creating too many partitions from highly distinct values.

### 3. Data Volume

Each partition should generally contain enough data to justify a separate storage partition.

### 4. Data Distribution

Check whether data is distributed reasonably across partition values.

### 5. Maintenance Cost

Consider how many directories and files the layout will create over time.

### Practical Example

For an order dataset containing several years of data:

| Column | Partitioning Decision |
|---|---|
| `order_date` | Good candidate, often at year/month granularity |
| `country` | Possible candidate if query patterns justify it |
| `customer_id` | Usually avoid directory partitioning |
| `order_id` | Avoid directory partitioning |

**Remember:** Choose partition columns based on actual query patterns and data characteristics, not simply because a column exists.

---

## 9. Partitioning in Delta Lake and Databricks

Delta Lake supports directory-based partitioning using partition columns.

Example:

```python
df.write \
    .format("delta") \
    .mode("overwrite") \
    .partitionBy("year", "month") \
    .save("/data/delta/orders/")
```

A partitioned Delta table can have a layout such as:

```text
orders/
├── _delta_log/
│
├── year=2026/
│   ├── month=01/
│   │   ├── part-0001.parquet
│   │   └── part-0002.parquet
│   │
│   └── month=02/
│       └── part-0003.parquet
```

The `_delta_log` directory contains transaction-log information used to manage the table.

The Parquet files contain the underlying data.

### Delta Partitioning Benefits

- Supports partition pruning.
- Helps organize large datasets.
- Can improve queries that filter on partition columns.
- Works with Delta Lake table transactions.

### Important Considerations

- Avoid excessive partition columns.
- Avoid high-cardinality directory partitioning.
- Watch for small files.
- Partitioning is not necessary for every Delta table.
- Choose the layout based on workload and data volume.

Modern Databricks environments also support **Liquid Clustering**, which offers an alternative to traditional directory-based partitioning for suitable workloads.

---

## 10. Traditional Partitioning vs Liquid Clustering

Liquid Clustering is a newer data-layout technique in Databricks that organizes data to improve data skipping without creating a directory for every clustering-column value.

### Traditional Partitioning

Example:

```text
orders/
├── year=2025/
└── year=2026/
```

Characteristics:
- Directory-based layout.
- Partition columns are selected in advance.
- Can be effective for suitable low-cardinality filters.
- Poor choices can create many small partitions.

### Liquid Clustering

Example:

```sql
CREATE TABLE orders (
    order_id BIGINT,
    customer_id BIGINT,
    order_date DATE,
    amount DOUBLE
)
USING DELTA
CLUSTER BY (customer_id, order_date);
```

Liquid Clustering organizes data based on selected clustering columns and supports data skipping.

It does not create a directory per customer or date value.

### Comparison

| Feature | Traditional Partitioning | Liquid Clustering |
|---|---|---|
| Layout | Directory-based | File-level data organization |
| Directory per value | Yes, for partition columns | No |
| High-cardinality columns | Often problematic | Can be suitable |
| Data skipping | Partition pruning and file statistics | File statistics and clustering |
| Flexibility | Partition layout is relatively fixed | Clustering keys can evolve in supported environments |
| Availability | Broadly supported | Databricks/runtime dependent |

**Important:** Liquid Clustering is not automatically better for every table. Choose based on workload, platform support, and data characteristics.

---

## 11. Partitioning vs Bucketing

These concepts are related but different.

### Partitioning

Divides data into separate storage partitions, often represented by directories.

Example:

```text
orders/
├── region=East/
├── region=West/
└── region=North/
```

### Bucketing

Divides records into a fixed number of buckets using a hash function on selected columns.

Conceptually:

```text
bucket_id = hash(customer_id) % 8
```

Bucketing can help organize data for certain joins and aggregations in systems that support and use bucket metadata.

### Comparison

| Feature | Partitioning | Bucketing |
|---|---|---|
| Distribution | By partition-column values | By hash of selected columns |
| Storage layout | Often separate directories | Fixed number of buckets |
| Main purpose | Filter-based data elimination | Organizing records by key |
| Number of groups | Depends on distinct partition values | Configured bucket count |
| Common consideration | Cardinality and small files | Bucket count and compatibility |

Do not assume that bucketing always improves Spark joins. Its benefit depends on the table format, catalog, execution plan, and whether the engine can use the bucket layout.

---

## 12. Real-World Data Engineering Example

Suppose an e-commerce company stores 500 million order records.

Common queries include:
- Orders by date.
- Monthly revenue.
- Regional sales.
- Customer purchase history.

A possible layout:

```text
orders/
│
├── year=2025/
│   ├── month=10/
│   ├── month=11/
│   └── month=12/
│
└── year=2026/
    ├── month=01/
    ├── month=02/
    └── month=03/
```

### Why Use Year and Month?

1. Most analytical queries filter by order date.
2. Year and month have relatively low cardinality.
3. The layout supports partition pruning.
4. Data can be processed incrementally by month.
5. It avoids creating a separate directory for every customer or transaction.

### Example Query

```sql
SELECT
    year,
    month,
    SUM(amount) AS total_revenue
FROM orders
WHERE year = 2026
  AND month IN (1, 2, 3)
GROUP BY year, month;
```

The engine may read only the relevant year/month partitions.

For customer-level analysis, the query can still filter by `customer_id`; that column does not need to be a storage partition column.

---

## 13. Common Interview Questions and Answers

### Q1. What is data partitioning?

Data partitioning is the process of dividing a large dataset into smaller sections based on selected columns or distribution rules. It helps organize data and can improve query performance.

### Q2. Why is partitioning used in data engineering?

It reduces unnecessary data reads through partition pruning, improves data organization, and can make incremental processing and maintenance easier.

### Q3. What is partition pruning?

Partition pruning is an optimization where the query engine skips storage partitions that cannot contain the requested data.

### Q4. What is the difference between `partitionBy()` and `repartition()` in PySpark?

`partitionBy()` organizes files into storage directories based on column values. `repartition()` redistributes a DataFrame into Spark execution partitions for parallel processing.

### Q5. What is a high-cardinality column?

A high-cardinality column contains many distinct values. Customer IDs and transaction IDs are common examples.

### Q6. Why should we avoid partitioning by customer ID?

It can create a very large number of storage directories, resulting in metadata overhead and small files.

### Q7. What is the small files problem?

It occurs when a dataset contains too many small files. This increases file metadata and open-operation overhead and can reduce query performance.

### Q8. What is data skew?

Data skew occurs when data is distributed unevenly across processing partitions, causing some tasks to process much more data than others.

### Q9. What is the difference between partition pruning and data skipping?

Partition pruning eliminates entire storage partitions using partition-column filters. Data skipping uses file statistics to avoid reading files that cannot match a query.

### Q10. Can we partition a Delta table?

Yes. Delta tables support directory-based partitioning using selected columns. The choice should be based on query patterns, data volume, and cardinality.

### Q11. Should every large table be partitioned?

No. Partitioning is useful only when it provides a meaningful benefit. Some large tables may perform well without traditional partitioning, especially when other layout optimizations are available.

### Q12. What is the difference between partitioning and bucketing?

Partitioning separates data using partition-column values, while bucketing distributes records into a fixed number of groups using a hash function.

### Q13. What is Liquid Clustering?

Liquid Clustering is a Databricks data-layout technique that organizes data based on selected columns to improve data skipping without creating a directory for every column value.

### Q14. Does adding more partitions always improve performance?

No. Too many partitions can increase metadata overhead and create small files. The number and size of partitions should match the workload.

### Q15. How do you identify whether partition pruning is working?

Inspect the query plan using `explain("formatted")` and look for partition filters in the scan details. Runtime metrics in the Spark UI can also help verify how much data was read.

---

## 14. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| More partitions always mean better performance | Too many partitions can create overhead |
| `partitionBy()` and `repartition()` are the same | One organizes storage; the other redistributes execution data |
| Partitioning by customer ID is always good | High-cardinality columns can create excessive directories |
| Partitioning removes all data skew | Skew can still occur |
| Partition pruning and data skipping are identical | They work at different levels |
| Every large table needs partitioning | Some tables do not benefit from traditional partitioning |
| `coalesce()` is the same as storage partitioning | DataFrame `coalesce()` changes execution partitions |
| Liquid Clustering creates partition directories | It organizes files without directory-per-value partitioning |
| Bucketing always improves joins | The benefit depends on engine and workload support |

---

## 15. Quick Revision

| Concept | Remember |
|---|---|
| Data partitioning | Dividing data into smaller sections |
| Date partitioning | Organizing data by date |
| Hash partitioning | Distributing data using a hash function |
| Range partitioning | Dividing data into value ranges |
| Partition pruning | Skipping irrelevant storage partitions |
| Data skipping | Skipping files using statistics |
| High cardinality | Many distinct values |
| Data skew | Uneven data distribution |
| Small files problem | Too many small files |
| `partitionBy()` | Storage-level directory organization |
| `repartition()` | Spark execution-level redistribution |
| Bucketing | Fixed hash-based groups |
| Liquid Clustering | Databricks file-layout optimization |

### One-Minute Interview Answer

Data partitioning is the process of dividing a large dataset into smaller sections based on selected columns. It is commonly used in data lakes and distributed processing systems.

For example, an orders dataset can be partitioned by year and month. When a query filters for January 2026, Spark may read only the relevant storage partition. This optimization is called partition pruning.

However, partitioning must be designed carefully. High-cardinality columns can create too many directories, and excessive partitioning can lead to small files and metadata overhead.

In PySpark, `partitionBy()` creates a partitioned storage layout, whereas `repartition()` redistributes data across Spark execution partitions. In Databricks, Liquid Clustering provides an alternative data-layout technique for suitable workloads.

---

## 16. Self-Assessment

- [ ] Explain data partitioning.
- [ ] Explain why partitioning is useful.
- [ ] Understand date-based partitioning.
- [ ] Understand hash partitioning.
- [ ] Understand range partitioning.
- [ ] Differentiate storage and execution partitioning.
- [ ] Explain partition pruning.
- [ ] Differentiate partition pruning and data skipping.
- [ ] Explain high-cardinality columns.
- [ ] Explain the small files problem.
- [ ] Explain data skew.
- [ ] Write a partitioned dataset using PySpark.
- [ ] Read and filter partitioned data.
- [ ] Inspect partition filters using `explain()`.
- [ ] Differentiate partitioning and bucketing.
- [ ] Explain traditional partitioning vs Liquid Clustering.

**Interview readiness:** You should be able to explain how you would partition a large orders dataset, justify your partition-column choice, and describe the performance problems caused by a poor partitioning strategy.