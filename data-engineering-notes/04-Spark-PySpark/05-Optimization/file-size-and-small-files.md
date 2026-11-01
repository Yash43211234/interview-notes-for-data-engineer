# File Size and Small Files Problem in Spark

## 1. What Is the Small Files Problem?

The small files problem occurs when a data storage system contains a very large number of small files instead of a smaller number of reasonably sized files.

For example:

Imagine a data lake containing 10 GB of data.

**Scenario A: Many small files**

- 100,000 files
- Each file is approximately 100 KB

**Scenario B: Fewer larger files**

- 100 files
- Each file is approximately 100 MB

Both scenarios contain approximately 10 GB of data, but Scenario B is generally more efficient for Spark processing.

The problem is not simply that files are small. It is the overhead caused by managing and processing a large number of files.

## 2. Why Do Small Files Create Performance Problems?

When Spark reads files, it must do more than read the actual data.

It also needs to:
- Discover files.
- List directory contents.
- Read file metadata.
- Plan input partitions and tasks.
- Open files and read their contents.

When there are too many small files, this overhead becomes significant.

### Main Problems

**1. File listing overhead**

Spark must discover and process metadata for many files.

**2. Task scheduling overhead**

A large number of small input partitions can create many short-running tasks.

**3. Metadata overhead**

Storage systems and table formats must manage information about many individual files.

**4. Poor resource utilization**

Tasks may spend a significant amount of time opening files and processing setup rather than processing actual data.

**5. Slow query performance**

Reading many small files can increase query planning and execution overhead.

**6. Storage management overhead**

More files mean more file metadata and more complicated maintenance.

## 3. How Does the Small Files Problem Occur?

Common causes include:

### A. Writing with Too Many Partitions

Suppose a DataFrame has 1,000 partitions.

```python
df.write.mode("overwrite").parquet(
    "/output/sales"
)
```

Spark may create many output files, depending on the number of non-empty partitions and the write operation.

If each partition contains very little data, the output may consist of many small files.

### B. Frequent Incremental Writes

Suppose a pipeline runs every 10 minutes and writes a small amount of data each time.

```text
10:00 AM → 10 files
10:10 AM → 10 files
10:20 AM → 10 files
10:30 AM → 10 files
```

Over time, the table may accumulate thousands of small files.

### C. High-Cardinality Partitioning

Consider:

```python
df.write.partitionBy("customer_id").parquet(
    "/output/sales"
)
```

If there are millions of customer IDs, this can create a very large number of directories and small files.

Partitioning by high-cardinality columns is generally not a good strategy.

### D. Low Data Volume Per Partition

If a large number of Spark partitions contain only a small amount of data, each partition may produce a small output file.

## 4. What Is a Good File Size?

There is no universal ideal file size. It depends on:
- File format.
- Storage system.
- Query engine.
- Data volume.
- Workload.
- Compression.
- Available parallelism.

For many analytical data lake workloads, files around **128 MB to 1 GB** are commonly considered reasonable starting points.

A target around 256 MB is also frequently used.

These are guidelines, not fixed Spark requirements.

The correct file size should be determined through workload testing.

## 5. How to Solve the Small Files Problem?

There are several approaches.

### A. Use coalesce() Before Writing

If the DataFrame has too many partitions, reduce the partition count.

```python
df = df.coalesce(20)

df.write.mode("overwrite").parquet(
    "/output/sales"
)
```

This can reduce the number of output files.

**Limitation:** `coalesce()` avoids a full shuffle in the usual case, but it may create uneven partition sizes. It is not always the best choice for a large or skewed dataset.

### B. Use repartition() Before Writing

Use repartitioning when you need to redistribute records across a controlled number of partitions.

```python
df = df.repartition(20)

df.write.mode("overwrite").parquet(
    "/output/sales"
)
```

This performs a shuffle but can distribute the data more evenly.

**Important:** Repartitioning by a skewed key can still produce uneven partitions.

### C. Control Partitioning During Writing

You can control how data is distributed before writing.

```python
df.repartition(
    20,
    "region"
).write.mode("overwrite").parquet(
    "/output/sales"
)
```

This distributes records according to the region column.

However, if one region contains most of the data, some output partitions may still be much larger than others.

### D. Compact Existing Files

If a table already contains many small files, simply changing the partition count for future writes will not fix existing files.

You need a compaction process.

Compaction combines multiple small files into fewer, larger files.

For example:

```text
Before:

file1.parquet  → 10 MB
file2.parquet  → 15 MB
file3.parquet  → 12 MB
file4.parquet  → 18 MB

After compaction:

file1.parquet  → 55 MB
```

This is an illustrative example. Actual compaction depends on the table format and implementation.

## 6. File Compaction in Delta Lake

Delta Lake supports file compaction through the `OPTIMIZE` command.

Example:

```sql
OPTIMIZE sales;
```

This combines smaller data files into larger files to improve read performance.

### Optimize a Specific Partition

```sql
OPTIMIZE sales
WHERE sale_date = '2026-10-31';
```

This targets the specified partition when the table is partitioned appropriately and the predicate is supported.

### Optimize with Z-Ordering

```sql
OPTIMIZE sales
ZORDER BY (customer_id);
```

Z-Ordering reorganizes data to improve data skipping for queries filtering on the selected columns.

**Important:** Compaction and Z-Ordering solve different problems.

- `OPTIMIZE` compacts small files.
- `ZORDER BY` improves data layout for data skipping.
- Z-Ordering is not simply another name for compaction.

Availability and supported syntax can depend on the Delta Lake and Databricks runtime version.

## 7. What Is Auto Compaction?

In Delta Lake, auto compaction can compact small files automatically after successful writes.

Example configuration:

```python
spark.conf.set(
    "spark.databricks.delta.autoCompact.enabled",
    "true"
)
```

Auto compaction helps reduce the accumulation of small files.

However:
- It does not replace all scheduled optimization.
- It may not compact every small file immediately.
- Its behavior depends on the Databricks environment and configuration.

## 8. What Is Optimized Writes?

Optimized writes improve the way data is distributed into output files during a write operation.

They aim to reduce the creation of many small files.

In Databricks, optimized writes can help improve file sizes, especially for partitioned tables.

**Difference:**

| Feature | Purpose |
|---|---|
| Optimized Writes | Improve file sizes during writing |
| Auto Compaction | Compact small files after writes |
| OPTIMIZE | Explicitly compact existing files |
| Z-Ordering | Improve data layout and data skipping |

Availability and default behavior vary by platform and runtime.

## 9. How to Identify the Small Files Problem?

You can investigate it using several methods.

### A. Check the Number of Files

For a filesystem path accessible through the Hadoop filesystem API:

```python
path = "/output/sales"

files = dbutils.fs.ls(path)

print("Number of entries:", len(files))
```

This lists entries at the specified path. For partitioned datasets, inspect the relevant subdirectories as well.

### B. Check File Sizes

```python
for file in files:
    if not file.isDir():
        print(
            file.name,
            file.size
        )
```

This helps identify very small files.

### C. Inspect Spark UI

Useful indicators include:
- Number of tasks in a stage.
- Input size and records.
- Task duration.
- Time spent in scheduling.
- Uneven task execution.

A large number of very short tasks can be a warning sign, but it does not prove that small files are the cause.

### D. Inspect Delta Table Details

```sql
DESCRIBE DETAIL sales;
```

This provides table information, including:
- Number of files.
- Total table size.
- Partition columns.
- Table properties.

You can use this information to understand the table's storage layout.

## 10. Small Files vs Too Many Partitions

These concepts are related, but they are not the same.

| Small Files Problem | Too Many Partitions |
|---|---|
| Too many small physical files | Too many Spark execution partitions or storage partitions |
| Affects file listing and read overhead | Can increase task scheduling overhead |
| Can occur due to frequent small writes | Can occur due to excessive repartitioning |
| Often addressed through compaction | Often addressed by adjusting partition count |

There are two different meanings of partition in Spark:

1. **Spark execution partition:** A logical unit of distributed processing, usually processed by a task.
2. **Storage/table partition:** A directory-based organization of data, such as `sale_date=2026-10-31`.

Do not confuse them during interviews.

## 11. Spark Partitioning vs File Partitioning

### Spark Execution Partitioning

```python
df = df.repartition(20)
```

This changes the number and distribution of Spark execution partitions.

### Storage Partitioning

```python
df.write.partitionBy(
    "sale_date"
).parquet("/output/sales")
```

This organizes output files into directories based on `sale_date`.

Example directory structure:

```text
sales/
    sale_date=2026-10-29/
    sale_date=2026-10-30/
    sale_date=2026-10-31/
```

Each storage partition can contain multiple files.

**Important:** `partitionBy()` does not mean one file per date. The number of files depends on the execution partitions and write behavior.

## 12. Practical Scenario

Imagine you are processing daily sales data.

- Daily data volume: 50 GB.
- Pipeline runs every hour.
- Each run writes a small amount of data.
- After several days, the table contains thousands of small Parquet files.

### What would you do?

1. Inspect the number and sizes of existing files.
2. Check the table's storage partitioning strategy.
3. Identify whether frequent incremental writes are creating small files.
4. Use Delta Lake `OPTIMIZE` to compact existing files, if applicable.
5. Consider optimized writes and auto compaction.
6. Review the number of Spark partitions before writing.
7. Monitor query performance after optimization.

Do not blindly reduce all data to a fixed number of partitions. Choose a strategy based on data volume and parallelism requirements.

## 13. Interview Questions

**Q1. What is the small files problem in Spark?**

It occurs when a dataset contains too many small files, increasing file listing, metadata, task scheduling, and read overhead.

**Q2. Why are too many small files bad?**

They create additional overhead when Spark discovers, opens, schedules, and processes files.

**Q3. How do you solve the small files problem?**

Use appropriate partition counts, coalesce or repartition when writing, and compact existing files. For Delta tables, use `OPTIMIZE` where appropriate.

**Q4. What is file compaction?**

Combining multiple small files into fewer, larger files to improve storage layout and read performance.

**Q5. What is the difference between coalesce() and repartition() for file writing?**

`coalesce()` generally reduces partitions without a full shuffle. `repartition()` redistributes data through a shuffle and can increase or decrease partitions.

**Q6. What is Delta Lake OPTIMIZE?**

It is a command that compacts data files in a Delta table to improve data layout and read performance.

**Q7. What is the difference between OPTIMIZE and Z-Ordering?**

OPTIMIZE compacts files. Z-Ordering reorganizes data to improve data skipping for selected columns.

**Q8. What are optimized writes?**

They improve data distribution during writes to help produce more appropriately sized files.

**Q9. What is the difference between Spark partitions and storage partitions?**

Spark partitions are execution units. Storage partitions organize data into directories based on column values.

**Q10. Does increasing the number of partitions always improve performance?**

No. Too many partitions can create many small tasks and files, increasing scheduling and metadata overhead.

## 14. Practice Questions

1. Why can writing a DataFrame with 1,000 small partitions create many small files?
2. How would you reduce the number of output files without a full shuffle?
3. When would you choose repartition() instead of coalesce()?
4. How would you compact an existing Delta table?
5. What is the difference between OPTIMIZE and auto compaction?
6. Why can partitioning a table by customer_id create a small files problem?
7. How would you identify small files in a data lake?
8. Explain the difference between execution partitions and storage partitions.

## 15. Quick Revision

- **Small files problem:** Too many small physical files.
- **Main impact:** File listing, metadata, scheduling, and read overhead.
- **Common causes:** Too many output partitions, frequent writes, high-cardinality partitioning.
- **coalesce():** Reduce execution partitions without a full shuffle in the usual case.
- **repartition():** Redistribute data through a shuffle.
- **Compaction:** Combine small files into fewer, larger files.
- **Delta OPTIMIZE:** Explicit file compaction.
- **Auto compaction:** Automatic small-file compaction after writes.
- **Optimized writes:** Improve file sizes during writing.
- **Z-Ordering:** Improve data layout and data skipping.
- **Execution partition:** Spark processing unit.
- **Storage partition:** Directory-based data organization.

## 16. Self-Check

- [ ] Explain the small files problem.
- [ ] Understand why small files affect Spark performance.
- [ ] Identify common causes of small files.
- [ ] Differentiate Spark partitions and storage partitions.
- [ ] Explain coalesce() and repartition() in file writing.
- [ ] Explain Delta Lake OPTIMIZE.
- [ ] Differentiate optimized writes, auto compaction, and Z-Ordering.
- [ ] Explain how to inspect file counts and sizes.
- [ ] Describe a practical small-file optimization strategy.

**Interview goal:** Be able to explain why small files slow down data processing and how you would prevent, identify, and fix the problem in a real data pipeline.