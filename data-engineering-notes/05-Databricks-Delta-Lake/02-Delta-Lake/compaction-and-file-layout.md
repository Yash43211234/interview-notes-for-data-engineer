# Compaction and File Layout in Delta Lake

## 1. What Is File Layout?

File layout refers to how data is organized and stored as files inside a Delta Lake table.

Delta Lake stores table data in Parquet files and maintains transaction information in the `_delta_log` directory.

Example:

```text
sales_delta/
│
├── part-00001.parquet
├── part-00002.parquet
├── part-00003.parquet
├── part-00004.parquet
│
└── _delta_log/
    ├── 00000000000000000000.json
    ├── 00000000000000000001.json
    └── 00000000000000000002.json
```

Each Parquet file contains a portion of the table's data.

The way these files are organized affects:

- Query performance
- Data scanning
- Storage efficiency
- Write performance
- Metadata overhead

**Remember:**
- Parquet files store the actual data.
- The Delta transaction log tracks which files belong to each committed table version.

## 2. What Are Small Files?

Small files are files containing relatively small amounts of data.

They are commonly created when:
- Frequent small batches are written.
- Streaming pipelines write many micro-batches.
- Jobs process small amounts of data repeatedly.
- Data is partitioned into too many directories.
- Write operations create many output partitions.

Example:

```text
sales_delta/
│
├── part-00001.parquet  (2 MB)
├── part-00002.parquet  (3 MB)
├── part-00003.parquet  (1 MB)
├── part-00004.parquet  (4 MB)
├── part-00005.parquet  (2 MB)
└── part-00006.parquet  (3 MB)
```

Suppose a table contains 1 GB of data stored in hundreds or thousands of small files.

Reading that data can create unnecessary overhead.

## 3. Why Are Small Files a Problem?

Small files can negatively affect query performance.

### Main problems

1. **Metadata overhead:** Spark and the storage system must manage many file references.
2. **File listing overhead:** Discovering and planning many files can take additional time.
3. **Task overhead:** Spark may create many small tasks.
4. **Poor resource utilization:** Tasks processing tiny files may not use available compute efficiently.
5. **Query latency:** Reading many files can increase execution overhead.

### Example

Consider two layouts containing the same amount of data:

```text
Layout A:
1,000 files × 1 MB = 1 GB

Layout B:
10 files × 100 MB = 1 GB
```

Layout B may be more efficient because it requires fewer files to manage.

However, there is no universal ideal file size. The best layout depends on workload, storage, partitioning, and query patterns.

**Interview point:** Small files create metadata and task-scheduling overhead, which can reduce query performance.

## 4. What Is Compaction?

Compaction is the process of combining multiple small data files into fewer, larger files.

It helps reduce file-management overhead and improve read performance.

### Before Compaction

```text
sales_delta/
│
├── part-00001.parquet  (2 MB)
├── part-00002.parquet  (3 MB)
├── part-00003.parquet  (1 MB)
├── part-00004.parquet  (4 MB)
├── part-00005.parquet  (2 MB)
└── part-00006.parquet  (3 MB)
```

### After Compaction

```text
sales_delta/
│
├── part-00001.parquet  (7 MB)
├── part-00002.parquet  (8 MB)
│
└── _delta_log/
```

The example is illustrative. Actual output file sizes and counts depend on the data, partitioning, and optimization settings.

### Benefits

- Reduces the number of small files.
- Reduces file-management overhead.
- Can improve query performance.
- Helps organize data for efficient reads.
- Can improve downstream processing efficiency.

**Remember:** Compaction changes the physical file layout, not the logical contents of the table.

## 5. OPTIMIZE Command

Databricks provides the `OPTIMIZE` command to compact small files in Delta tables.

Example:

```sql
OPTIMIZE sales;
```

This command reorganizes table data into fewer, more efficient files.

### Optimize a Specific Partition

```sql
OPTIMIZE sales
WHERE order_date = '2026-10-31';
```

This is useful when we want to compact only a particular partition.

### PySpark Example

```python
from delta.tables import DeltaTable

delta_table = DeltaTable.forPath(
    spark,
    "/mnt/delta/sales"
)

delta_table.optimize().executeCompaction()
```

The DeltaTable API shown here is available in supported Delta Lake/Databricks environments. SQL `OPTIMIZE` is also commonly used in Databricks.

**Important:** `OPTIMIZE` consumes compute resources. Running it too frequently can increase cost without providing meaningful benefits.

## 6. Compaction vs. Repartitioning

These concepts are related but different.

### Repartitioning

Repartitioning changes the number or distribution of Spark DataFrame partitions.

Example:

```python
df_repartitioned = df.repartition(10)
```

This creates a DataFrame with 10 partitions, subject to the operation's execution behavior.

### Compaction

Compaction reorganizes existing Delta table files into fewer, larger files.

Example:

```sql
OPTIMIZE sales;
```

### Difference

| Repartitioning | Compaction |
|---|---|
| Works on Spark DataFrame partitions | Works on persisted Delta table files |
| Changes data distribution across partitions | Combines small files into larger files |
| Happens during DataFrame processing | Reorganizes existing table data |
| Can cause a shuffle | Uses Delta optimization operations |
| Does not automatically optimize an existing table | Optimizes the physical file layout |

**Remember:** `repartition()` changes Spark execution partitions. `OPTIMIZE` compacts stored Delta files.

## 7. What Is Partitioning in Delta Lake?

Partitioning organizes table data into separate directory paths based on the values of selected columns.

It can help reduce the amount of data scanned when queries filter on partition columns.

Example:

Suppose a sales table is partitioned by `year`.

```text
sales_delta/
│
├── year=2024/
│   ├── part-00001.parquet
│   └── part-00002.parquet
│
├── year=2025/
│   ├── part-00003.parquet
│   └── part-00004.parquet
│
└── year=2026/
    ├── part-00005.parquet
    └── part-00006.parquet
```

### Create a Partitioned Delta Table

```python
(
    df.write
    .format("delta")
    .mode("overwrite")
    .partitionBy("year")
    .save("/mnt/delta/sales")
)
```

### Query Example

```sql
SELECT *
FROM sales
WHERE year = 2026;
```

Spark can use partition pruning to avoid reading files from other year directories when the filter is recognized and supported.

**Interview point:** Partitioning is useful when queries frequently filter on suitable partition columns.

## 8. Partition Pruning

Partition pruning means skipping table partitions that do not match a query filter.

Example:

Suppose a table has data for 2024, 2025, and 2026.

Query:

```sql
SELECT *
FROM sales
WHERE year = 2026;
```

Instead of scanning all year directories, Spark can scan only the matching partition.

```text
sales_delta/
│
├── year=2024/  ---- Skipped
├── year=2025/  ---- Skipped
└── year=2026/  ---- Read
```

### Benefits

- Reduces data scanning.
- Improves query performance.
- Can reduce I/O.
- Helps process large partitioned datasets efficiently.

**Important:** Partition pruning is not the same as data skipping. Partition pruning uses partition-column information, while data skipping uses file-level statistics to avoid reading unnecessary files.

## 9. Problems with Over-Partitioning

Creating too many partitions can reduce performance.

Example:

Partitioning a sales table by `order_id` when every order ID is unique can create an excessive number of directories.

### Problems

- Too many directories.
- Many small files.
- Increased metadata overhead.
- More expensive file listing.
- Poor query performance.

### Good Partitioning Practices

1. Choose columns frequently used in filters.
2. Prefer columns with a reasonable number of distinct values.
3. Avoid high-cardinality partition columns.
4. Consider the amount of data in each partition.
5. Avoid creating too many tiny partitions.
6. Evaluate actual query patterns before choosing partition columns.

**Remember:** Partitioning should reduce data scanning without creating excessive directory and file overhead.

## 10. Z-Ordering

Z-Ordering is a data-layout optimization used in Databricks Delta Lake to colocate related values in files.

It helps improve data skipping for supported queries.

Example:

Suppose users frequently filter sales data using:

- `customer_id`
- `product_id`

We can run:

```sql
OPTIMIZE sales
ZORDER BY (customer_id, product_id);
```

Z-Ordering reorganizes the data so that related values are more likely to be located together in files.

### Benefits

- Can improve data skipping.
- Reduces unnecessary file reads.
- Useful for frequently filtered columns.
- Can improve query performance for suitable workloads.

**Important:**
- Z-Ordering is not the same as partitioning.
- Z-Ordering does not create one directory per column value.
- It is not a replacement for good partition design.
- It consumes compute resources.
- Its effectiveness depends on query patterns and data distribution.

Z-Ordering is supported in applicable Databricks environments. Newer Databricks table features, such as liquid clustering, may be preferred for some workloads.

## 11. Data Skipping

Data skipping means avoiding the reading of files that cannot contain records matching a query condition.

Delta Lake collects file-level statistics for supported data types and operations.

These statistics can include:

- Minimum values.
- Maximum values.
- Number of records.
- Null counts.

Example:

Suppose three files contain these ranges:

| File | Minimum order amount | Maximum order amount |
|---|---:|---:|
| File 1 | 100 | 500 |
| File 2 | 501 | 1,000 |
| File 3 | 1,001 | 2,000 |

Query:

```sql
SELECT *
FROM sales
WHERE amount > 1500;
```

Files 1 and 2 cannot contain matching records based on their recorded statistics, so they can be skipped.

File 3 must be considered.

**Remember:** Data skipping reduces unnecessary file reads using file-level statistics.

## 12. Compaction vs. Z-Ordering vs. Partitioning

These three concepts solve different problems.

| Concept | Main Purpose |
|---|---|
| Compaction | Reduce small-file overhead |
| Partitioning | Organize data into directory partitions |
| Z-Ordering | Improve data skipping by reorganizing data values |

### Simple Example

Suppose we have a sales table.

- Compaction: Combine hundreds of small files.
- Partitioning: Organize data by year.
- Z-Ordering: Organize data based on customer ID and product ID to improve suitable filters.

These techniques can be used together when appropriate.

## 13. Liquid Clustering

Liquid clustering is a Databricks data-layout feature that organizes data based on selected clustering columns.

It is designed to improve data skipping and provide a more flexible alternative to traditional partitioning and Z-Ordering for suitable workloads.

Example:

```sql
CREATE TABLE sales (
    order_id BIGINT,
    customer_id BIGINT,
    order_date DATE,
    amount DOUBLE
)
USING DELTA
CLUSTER BY (customer_id, order_date);
```

For an existing supported table, clustering can be configured and data reorganized using supported `OPTIMIZE` operations.

### Benefits

- Flexible data organization.
- Useful for changing query patterns.
- Avoids creating large numbers of traditional partition directories.
- Can improve data skipping.

**Important:** Liquid clustering availability and behavior depend on Databricks Runtime, table features, and platform support. It is not simply another name for Z-Ordering.

## 14. Auto Compaction and Optimized Writes

Databricks provides features that help manage file sizes automatically.

### Auto Compaction

Auto compaction can compact small files after successful writes when the relevant configuration and workload support it.

### Optimized Writes

Optimized writes improve file sizes during write operations by adjusting how data is distributed before files are written.

### Difference

| Auto Compaction | Optimized Writes |
|---|---|
| Works after writes | Works during writes |
| Combines small files | Helps create better-sized files initially |
| Reduces small-file accumulation | Reduces the creation of small files |

Availability, default behavior, and configuration vary by Databricks Runtime and table type.

**Remember:** Optimized writes help prevent small files; auto compaction helps clean up small files after writes.

## 15. File Layout Best Practices

1. Avoid generating too many small files.
2. Use `OPTIMIZE` when compaction is beneficial.
3. Choose partition columns based on query patterns.
4. Avoid high-cardinality partition columns.
5. Use partition pruning for suitable filters.
6. Consider Z-Ordering or liquid clustering for data-skipping requirements.
7. Enable optimized writes where appropriate.
8. Use auto compaction when suitable.
9. Monitor file sizes and query performance.
10. Avoid running optimization operations unnecessarily.
11. Consider the cost of rewriting data.
12. Review the physical layout as query patterns change.

## 16. Common Interview Questions

### Q1. What is compaction in Delta Lake?

Compaction combines multiple small data files into fewer, larger files to reduce metadata and task overhead and potentially improve query performance.

### Q2. Why are small files a problem?

Small files increase file-management, metadata, listing, and task-scheduling overhead.

### Q3. What is the purpose of `OPTIMIZE`?

`OPTIMIZE` reorganizes Delta table files to improve their physical layout, commonly by compacting small files.

### Q4. What is the difference between repartitioning and compaction?

Repartitioning changes Spark DataFrame partitions. Compaction reorganizes files already stored in a Delta table.

### Q5. What is partitioning?

Partitioning organizes table data into directory paths based on selected column values.

### Q6. What is partition pruning?

Partition pruning skips partitions that cannot match a query's partition-column filter.

### Q7. Why should we avoid high-cardinality partition columns?

They can create too many directories and small files, increasing metadata and query overhead.

### Q8. What is Z-Ordering?

Z-Ordering reorganizes data values to improve file-level data skipping for suitable filter queries.

### Q9. What is data skipping?

Data skipping avoids reading files that cannot contain matching records, using file-level statistics.

### Q10. What is the difference between partitioning and Z-Ordering?

Partitioning organizes data into directory partitions. Z-Ordering reorganizes data within the table to improve data skipping without creating a directory for every value.

### Q11. What are optimized writes?

Optimized writes help produce better-sized files during write operations by adjusting how data is distributed before writing.

### Q12. What is auto compaction?

Auto compaction automatically combines small files after writes when supported and configured.

### Q13. What is liquid clustering?

Liquid clustering is a flexible data-layout technique that organizes data around selected clustering columns to improve data skipping.

### Q14. Does `OPTIMIZE` change the logical table data?

Normally, compaction preserves the logical table contents while changing the physical file layout. The operation creates a new committed table version.

## 17. Quick Revision

| Concept | Remember |
|---|---|
| File Layout | Physical organization of table files |
| Small Files | Many files containing little data |
| Compaction | Combines small files |
| `OPTIMIZE` | Reorganizes Delta table files |
| Repartitioning | Changes Spark DataFrame partitions |
| Partitioning | Organizes data into directory partitions |
| Partition Pruning | Skips irrelevant partitions |
| Z-Ordering | Improves data skipping through data reorganization |
| Data Skipping | Avoids reading irrelevant files |
| Liquid Clustering | Flexible data-layout optimization |
| Optimized Writes | Helps create better-sized files during writes |
| Auto Compaction | Compacts small files after writes |

## 18. Self-Assessment

Before moving to the next topic, make sure you can explain:

- [ ] What is file layout in Delta Lake?
- [ ] What are small files?
- [ ] Why do small files affect performance?
- [ ] What is compaction?
- [ ] How does `OPTIMIZE` work?
- [ ] What is the difference between compaction and repartitioning?
- [ ] What is partitioning?
- [ ] What is partition pruning?
- [ ] Why should high-cardinality columns be avoided for partitioning?
- [ ] What is Z-Ordering?
- [ ] What is data skipping?
- [ ] What is liquid clustering?
- [ ] What is the difference between optimized writes and auto compaction?
- [ ] How would you optimize a Delta table containing thousands of small files?

**Interview readiness:** You should be able to explain how Delta Lake file layout affects query performance and when to use compaction, partitioning, Z-Ordering, data skipping, and liquid clustering.