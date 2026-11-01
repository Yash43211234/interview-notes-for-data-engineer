# OPTIMIZE and File Management in Databricks

## 1. What Is File Management in Databricks?

File management in Databricks means organizing, maintaining, and optimizing the physical data files stored in Delta Lake tables.

When data is written to a Delta table, Spark creates data files, generally in Parquet format.

Over time, frequent writes, updates, deletes, and streaming ingestion can create many small files.

These small files can negatively affect query performance and increase file-management overhead.

**Main objective:** Maintain an efficient file layout so queries can read data efficiently.

## 2. Why Do Small Files Become a Problem?

Suppose an orders table receives data every few minutes.

Each write creates several small files.

After multiple ingestion runs, the table may look like this:

```text
orders/
|
|-- part-0001.parquet  (2 MB)
|-- part-0002.parquet  (3 MB)
|-- part-0003.parquet  (1 MB)
|-- part-0004.parquet  (4 MB)
|-- part-0005.parquet  (2 MB)
|-- part-0006.parquet  (3 MB)
|-- part-0007.parquet  (1 MB)
```

Instead of a few reasonably sized files, the table contains many small files.

### Problems Caused by Small Files

1. **Metadata overhead:** The system must manage information about many files.
2. **More file operations:** Opening and processing many files creates overhead.
3. **Slower queries:** Query planning and file scanning can become less efficient.
4. **Poor resource utilization:** Small tasks may not use available compute efficiently.
5. **Increased maintenance:** More files may need to be managed during optimization.

**Important:** Small files are not always bad. The problem occurs when file count and file size create unnecessary overhead for the workload.

## 3. What Is OPTIMIZE?

`OPTIMIZE` is a Databricks SQL command used to reorganize Delta table data files.

Its common purpose is to compact smaller files into fewer, larger files.

### Before OPTIMIZE

```text
Delta Table
|
|-- file1.parquet  (2 MB)
|-- file2.parquet  (3 MB)
|-- file3.parquet  (1 MB)
|-- file4.parquet  (4 MB)
|-- file5.parquet  (2 MB)
```

### After OPTIMIZE

```text
Delta Table
|
|-- file1.parquet  (12 MB)
```

This is a simplified illustration. Actual output file sizes and file counts depend on the data volume, configuration, partitioning, and runtime behavior.

### Basic Syntax

```sql
OPTIMIZE main.silver.orders;
```

### Benefits

- Reduces small-file overhead.
- Improves read performance for suitable workloads.
- Reduces the number of files that queries may need to process.
- Helps maintain an efficient Delta table layout.

**Remember:** OPTIMIZE consumes compute resources. Running it too frequently can increase costs without providing enough benefit.

## 4. How OPTIMIZE Works

At a high level, OPTIMIZE:

1. Identifies files that can benefit from reorganization.
2. Reads the selected data files.
3. Rewrites the data into a more efficient file layout.
4. Commits the new file set through the Delta transaction log.
5. Marks replaced files as removed from the current table snapshot.

Old files are not necessarily physically deleted immediately. They may remain available for historical versions until they become eligible for VACUUM.

### Conceptual Flow

```text
       EXISTING DELTA TABLE
                |
                v
        MANY SMALL FILES
                |
                v
          OPTIMIZE
                |
                v
       REWRITE DATA FILES
                |
                v
       DELTA LOG COMMIT
                |
                v
       FEWER, LARGER FILES
                |
                v
       IMPROVED READ LAYOUT
```

## 5. OPTIMIZE with Z-Ordering

OPTIMIZE can also be used with Z-Ordering.

Z-Ordering reorganizes data so that related values of selected columns are colocated more effectively.

This can improve data skipping for suitable query patterns.

### Example

Suppose users frequently query orders using `customer_id`.

```sql
SELECT *
FROM main.silver.orders
WHERE customer_id = 501;
```

We can run:

```sql
OPTIMIZE main.silver.orders
ZORDER BY (customer_id);
```

For multiple columns:

```sql
OPTIMIZE main.silver.orders
ZORDER BY (customer_id, order_date);
```

### How Does It Help?

Delta Lake maintains file-level statistics such as minimum and maximum values for eligible columns.

When a query filters on a column, Databricks can use these statistics to skip files that cannot contain matching records.

Z-Ordering can improve the effectiveness of this data skipping by organizing related values more closely.

**Important:**

- Z-Ordering is not the same as partitioning.
- Z-Ordering is not a replacement for good query filters.
- More Z-Order columns do not automatically mean better performance.
- Its usefulness depends on data distribution and query patterns.
- Z-Ordering consumes compute resources.

## 6. What Is Liquid Clustering?

Liquid Clustering is a newer Databricks table-layout technique that organizes data around selected clustering columns.

Example:

```sql
CREATE TABLE main.silver.orders (
    order_id BIGINT,
    customer_id BIGINT,
    amount DECIMAL(12,2),
    order_date DATE
)
USING DELTA
CLUSTER BY (customer_id, order_date);
```

For a supported table, run:

```sql
OPTIMIZE main.silver.orders;
```

Databricks uses the clustering definition during optimization.

### Liquid Clustering vs Z-Ordering

| Feature | Z-Ordering | Liquid Clustering |
|---|---|---|
| Purpose | Improve data skipping | Improve data layout and data skipping |
| Configuration | `ZORDER BY` during OPTIMIZE | `CLUSTER BY` in table definition |
| Flexibility | Requires specifying columns for optimization | Clustering definition can evolve |
| Maintenance | Explicit Z-Order optimization | Uses supported OPTIMIZE behavior |
| Use case | Suitable stable filter patterns | Large tables with evolving query patterns |

**Important:** These are alternative layout strategies. Do not assume Z-Ordering should also be applied to a table using Liquid Clustering.

## 7. What Is VACUUM?

`VACUUM` is a Delta Lake command that removes eligible unreferenced data files from a table.

When records are updated or deleted, Delta Lake does not necessarily delete the old physical files immediately.

Instead, the transaction log changes the active file set.

Old files may remain in storage until they become eligible for VACUUM.

### Example

Suppose a table contains:

```text
orders/
|
|-- file1.parquet
|-- file2.parquet
|-- file3.parquet
```

After an update, the current table snapshot may use:

```text
orders/
|
|-- file4.parquet
|-- file2.parquet
|-- file3.parquet
```

The old `file1.parquet` may remain physically stored even though it is no longer part of the current snapshot.

VACUUM can eventually remove it when it is eligible.

### Basic Syntax

```sql
VACUUM main.silver.orders;
```

### Preview Eligible Files

```sql
VACUUM main.silver.orders DRY RUN;
```

`DRY RUN` previews files that the command would remove without deleting them.

### Benefits

- Removes obsolete data files.
- Helps reclaim storage.
- Controls storage growth over time.

**Important:** VACUUM is primarily a storage cleanup operation. It is not a query-performance optimization command like OPTIMIZE.

## 8. OPTIMIZE vs VACUUM

This is one of the most important Databricks interview questions.

| Feature | OPTIMIZE | VACUUM |
|---|---|---|
| Main purpose | Reorganize data files | Remove eligible obsolete files |
| Main benefit | Improve file layout and read performance | Reclaim storage |
| Operation | Reads and rewrites selected data | Deletes eligible unreferenced files |
| Delta log | Commits new file actions | Does not create a normal table-data rewrite |
| Time Travel | Old files may remain available | Can make older versions unavailable |
| Compute usage | Requires compute to process and rewrite data | Requires compute to identify and delete eligible files |

**Easy way to remember:**

- OPTIMIZE = Organize files.
- VACUUM = Remove obsolete files.

## 9. Retention Period and Time Travel

Delta Lake retains historical information to support features such as Time Travel.

However, Time Travel requires both the relevant transaction-log history and the underlying data files.

Two commonly encountered table properties are:

| Property | Common Default | Purpose |
|---|---|---|
| `delta.deletedFileRetentionDuration` | 7 days | Minimum age before deleted data files become eligible for VACUUM |
| `delta.logRetentionDuration` | 30 days | Retention period for transaction-log history |

Actual values can vary by configuration and environment.

### Example

Suppose a record is updated today.

The old data file may remain available for historical versions until it becomes eligible for removal.

After VACUUM removes that file, an older version that depends on it may no longer be readable.

**Important:**

- Log retention and data-file retention are different.
- Keeping transaction-log history does not guarantee that the old data files still exist.
- VACUUM can limit Time Travel even when historical metadata remains.
- Never reduce retention settings without understanding recovery and audit requirements.

## 10. OPTIMIZE and VACUUM in an ETL Pipeline

Suppose an e-commerce company ingests order data every day.

The pipeline writes data into a Silver Delta table.

Over time, repeated writes create many small files.

A maintenance process can be designed as follows:

```text
       DAILY DATA INGESTION
                |
                v
          BRONZE TABLE
                |
                v
        SILVER TRANSFORMATION
                |
                v
        SILVER DELTA TABLE
                |
                v
          OPTIMIZE
                |
                v
       IMPROVED FILE LAYOUT
                |
                v
       GOLD AGGREGATIONS
                |
                v
        REPORTING TABLES
                |
                v
       PERIODIC VACUUM
                |
                v
       STORAGE CLEANUP
```

### Example SQL

```sql
-- Reorganize the Silver table
OPTIMIZE main.silver.orders;

-- Preview files eligible for cleanup
VACUUM main.silver.orders DRY RUN;

-- Remove eligible obsolete files
VACUUM main.silver.orders;
```

**Production consideration:** Maintenance should be scheduled according to table activity, query requirements, storage growth, and retention policies. It should not automatically run after every ingestion batch.

## 11. Auto Compaction and Optimized Writes

Databricks provides features that can help reduce small-file creation.

### 11.1 Optimized Writes

Optimized writes improve how data is written to Delta tables by managing output file layout and reducing small-file problems.

They can reduce the need for frequent manual compaction.

### 11.2 Auto Compaction

Auto compaction can compact small files after successful writes, subject to the supported configuration and runtime behavior.

### Difference

| Feature | Purpose |
|---|---|
| Optimized Writes | Improves file layout during writes |
| Auto Compaction | Compacts small files after writes |
| OPTIMIZE | Explicitly reorganizes existing table files |
| VACUUM | Removes eligible obsolete files |

**Important:** Availability, defaults, and behavior vary by Databricks runtime, table configuration, and platform features.

## 12. Practical Example: File Management Strategy

Suppose a company has a large orders table with:

- Daily incremental ingestion.
- Frequent customer-level queries.
- Regular updates and deletes.
- Historical reporting requirements.

A reasonable strategy could be:

| Requirement | Approach |
|---|---|
| Reduce small-file creation | Evaluate optimized writes |
| Compact files automatically | Evaluate auto compaction |
| Improve current file layout | OPTIMIZE |
| Improve filtering by customer | Evaluate Z-Ordering or Liquid Clustering |
| Remove obsolete files | VACUUM |
| Preserve historical data | Configure suitable retention |
| Monitor table health | Review file counts, sizes, and query metrics |

The correct approach depends on the table's workload and Databricks capabilities.

## 13. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| OPTIMIZE deletes all old files | It rewrites data and updates the active file set; old files may remain |
| VACUUM improves query performance directly | Its primary purpose is storage cleanup |
| OPTIMIZE and VACUUM are identical | They perform different operations |
| VACUUM can always be run safely at any retention period | Retention affects Time Travel and recovery |
| Z-Ordering and Liquid Clustering are identical | They are different data-layout strategies |
| OPTIMIZE must run after every write | Run it based on workload and maintenance needs |
| Small files should always be eliminated | File size should suit the workload |
| VACUUM removes the transaction log completely | It removes eligible unreferenced data files; log cleanup is separate |
| OPTIMIZE guarantees a fixed output file size | Output sizing depends on data and configuration |

## 14. Interview Questions and Answers

**Q1. What is OPTIMIZE in Databricks?**

OPTIMIZE is a Delta Lake command used to reorganize data files, commonly by compacting smaller files into fewer, larger files.

**Q2. Why do we need OPTIMIZE?**

Frequent writes can create many small files. OPTIMIZE helps reduce file-management overhead and can improve query performance.

**Q3. What is VACUUM?**

VACUUM removes eligible unreferenced data files from a Delta table after the configured retention period.

**Q4. What is the difference between OPTIMIZE and VACUUM?**

OPTIMIZE reorganizes active data files to improve layout. VACUUM deletes eligible obsolete files to reclaim storage.

**Q5. Does OPTIMIZE delete old files immediately?**

No. Replaced files may remain physically stored until they become eligible for VACUUM.

**Q6. What is Z-Ordering?**

Z-Ordering reorganizes data to improve data skipping for selected columns.

**Q7. What is the purpose of VACUUM DRY RUN?**

It previews files eligible for deletion without actually deleting them.

**Q8. What is the default VACUUM retention period?**

The commonly used default deleted-file retention is seven days, but table settings and environment configuration can differ.

**Q9. How does VACUUM affect Time Travel?**

If VACUUM deletes data files needed by an older table version, that version may no longer be readable.

**Q10. What are optimized writes?**

They improve file layout during writes and can help reduce small-file problems.

**Q11. What is auto compaction?**

It automatically compacts small files after writes when supported and configured.

**Q12. Should OPTIMIZE run after every batch?**

Not necessarily. Its frequency should depend on file creation rate, query workload, and the cost of optimization.

**Q13. How would you handle a Delta table containing thousands of small files?**

I would inspect file sizes and query performance, review the write pattern, evaluate optimized writes and auto compaction, and run OPTIMIZE when beneficial. I would separately review VACUUM retention to manage obsolete storage safely.

**Q14. Does VACUUM remove the Delta transaction log?**

No. VACUUM removes eligible unreferenced data files. Transaction-log cleanup is a separate process.

## 15. Quick Revision

| Concept | Remember |
|---|---|
| Small files | Increase file-management overhead |
| OPTIMIZE | Reorganizes and compacts Delta files |
| Z-Ordering | Improves data skipping for selected columns |
| Liquid Clustering | Flexible clustering-based table layout |
| VACUUM | Removes eligible obsolete data files |
| DRY RUN | Previews files eligible for VACUUM |
| Auto Compaction | Compacts small files after writes |
| Optimized Writes | Improves file layout during writes |
| Time Travel | Requires retained log history and data files |
| Retention | Controls when deleted files can be removed |

### One-Minute Interview Answer

In Databricks, file management is important because frequent batch and streaming writes can create many small files in Delta tables.

I use OPTIMIZE to reorganize files and improve the table's read layout. For suitable query patterns, I can use Z-Ordering or Liquid Clustering to improve data skipping.

VACUUM serves a different purpose. It removes eligible unreferenced data files and helps reclaim storage.

I also evaluate optimized writes and auto compaction to reduce small-file creation during ingestion.

Before running VACUUM, I check retention requirements because deleting old files can affect Time Travel and historical recovery.

## 16. Self-Assessment

- [ ] Explain why small files are a problem.
- [ ] Explain how OPTIMIZE works.
- [ ] Write basic OPTIMIZE syntax.
- [ ] Explain Z-Ordering.
- [ ] Differentiate Z-Ordering and Liquid Clustering.
- [ ] Explain VACUUM.
- [ ] Explain VACUUM DRY RUN.
- [ ] Differentiate OPTIMIZE and VACUUM.
- [ ] Explain deleted-file retention.
- [ ] Explain how VACUUM affects Time Travel.
- [ ] Differentiate optimized writes and auto compaction.
- [ ] Design a file-maintenance strategy for a production Delta table.

**Interview readiness:** You should be able to explain why small files occur, how OPTIMIZE improves file layout, how VACUUM reclaims storage, and why retention settings must be handled carefully.