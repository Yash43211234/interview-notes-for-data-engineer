# Delta Lake — Quick Revision Cheat Sheet

> **Purpose:** Fast revision for Data Engineer interviews covering Delta Lake architecture, transaction logs, ACID transactions, MERGE, schema evolution, Time Travel, OPTIMIZE, Z-ORDER, VACUUM, streaming, and performance troubleshooting.

---

## 1. What Is Delta Lake?

**Delta Lake** is an open-source storage layer that adds transactional reliability and table-management features to data stored in files, commonly Parquet.

It supports:
- ACID transactions.
- Consistent table reads.
- Schema enforcement and controlled schema evolution.
- `INSERT`, `UPDATE`, `DELETE`, and `MERGE`.
- Time Travel.
- Batch and streaming workloads.
- File compaction and data skipping.

**Key point:** Parquet is a file format. Delta Lake is a table storage layer built on data files plus a transaction log.

## 2. Delta Lake Architecture

A typical Delta table contains:

```text
sales_delta/
├── _delta_log/
│   ├── 00000000000000000000.json
│   ├── 00000000000000000001.json
│   └── checkpoint files
├── part-....parquet
├── part-....parquet
└── ...
```

- **Parquet files:** Store the table's data.
- **Transaction log:** Records committed table changes and metadata.
- **JSON commits:** Record actions for individual table versions.
- **Checkpoints:** Summarize transaction-log state to speed up reconstruction.

The exact files and checkpoint layout depend on the Delta version and table configuration.

### How a Write Works

1. The writer reads the required table state.
2. It prepares data files and transaction actions.
3. The transaction attempts to commit its changes.
4. A successful commit creates a new table version.
5. Readers use the committed snapshot rather than treating every physical file as current data.

A failed or uncommitted write may leave physical files that are not part of the committed table state.

---

## 3. Delta Lake vs Parquet

| Feature | Parquet Alone | Delta Lake |
|---|---|---|
| Columnar file format | Yes | Commonly uses Parquet data files |
| Transaction log | No | Yes |
| Table-level ACID transactions | Not provided by the format alone | Supported |
| `UPDATE` and `DELETE` | Usually require rewriting data through another mechanism | Supported through the table layer |
| `MERGE` | Not built into Parquet | Supported by Delta engines |
| Time Travel | Not built into Parquet | Supported while required history and files remain available |
| Schema enforcement | No table-level enforcement by the file format alone | Supported by Delta |
| Batch and streaming integration | Depends on the processing engine | Supported by compatible engines |

**Interview answer:** Delta Lake combines Parquet's efficient columnar storage with a transaction log that tracks committed table state and enables reliable table operations.

---

## 4. ACID Transactions

| Property | Meaning in Delta Lake |
|---|---|
| Atomicity | A transaction's changes are committed as a unit |
| Consistency | Successful writes must satisfy applicable table and transaction rules |
| Isolation | Concurrent operations follow the engine's supported isolation and conflict rules |
| Durability | Committed changes persist according to the storage system's durability guarantees |

Delta Lake uses optimistic concurrency control for supported transactions.

### Optimistic Concurrency Control (OCC)

A simplified view:

1. Read the table state.
2. Prepare changes.
3. Validate against conflicting concurrent commits.
4. Commit if valid; otherwise, report a conflict or retry when appropriate.

Two concurrent writers do not automatically succeed just because they write to different application-level records. Actual conflicts depend on the operation, read/write sets, isolation behavior, and platform features.

**Interview point:** OCC helps preserve correctness; it does not eliminate every concurrency conflict.

---

## 5. Transaction Log and Table Versions

The transaction log tracks information such as:
- Data files added to the table.
- Data files removed from the current snapshot.
- Table metadata and schema.
- Transaction-related information.
- Other protocol or feature actions.

Each successful commit advances the table version.

For example:

```text
Version 0 → Initial table
Version 1 → New data inserted
Version 2 → Records updated
Version 3 → Records deleted
```

The actual operation sequence depends on the workload.

### Why the Log Matters

Without a transaction log, a reader cannot reliably determine which physical files constitute a committed Delta table snapshot.

The log also supports table history, concurrent transaction validation, and Time Travel.

---

## 6. Creating and Writing a Delta Table

### PySpark

```python
df.write \
    .format("delta") \
    .mode("overwrite") \
    .save("/data/sales_delta")
```

Read the table:

```python
df = spark.read \
    .format("delta") \
    .load("/data/sales_delta")
```

Append new records:

```python
df_new.write \
    .format("delta") \
    .mode("append") \
    .save("/data/sales_delta")
```

**Caution:** `overwrite` can replace existing table data. Use it only when replacing the intended target is safe. Schema replacement and partition-specific overwrite behavior depend on the options and configuration.

### Register a Table

```sql
CREATE TABLE sales
USING DELTA
LOCATION '/data/sales_delta';
```

This registers the specified location as a table in the current catalog, subject to platform support and permissions.

---

## 7. INSERT, UPDATE, DELETE, and MERGE

### INSERT

```sql
INSERT INTO sales
VALUES (101, 'Laptop', 50000);
```

Use the actual table schema and compatible values.

### UPDATE

```sql
UPDATE sales
SET amount = 55000
WHERE order_id = 101;
```

### DELETE

```sql
DELETE FROM sales
WHERE order_id = 101;
```

Delta operations may rewrite affected data files rather than modifying individual bytes in place. Physical cleanup is a separate concern.

### MERGE — Upsert

`MERGE` can insert new records and update matching records.

```sql
MERGE INTO target_customers AS t
USING source_customers AS s
ON t.customer_id = s.customer_id

WHEN MATCHED THEN
    UPDATE SET
        t.customer_name = s.customer_name

WHEN NOT MATCHED THEN
    INSERT (customer_id, customer_name)
    VALUES (s.customer_id, s.customer_name);
```

### MERGE Best Practices

- Deduplicate the source by the intended business key.
- Define a deterministic winner when multiple source versions exist.
- Validate the matching key.
- Handle deletes explicitly when required.
- Check source and target schemas.
- Make retry behavior safe.
- Monitor conflicts and data quality.

**Important:** Multiple source rows matching the same target row can cause ambiguous or failed updates, depending on the operation and runtime. Deduplicate before merging.

---

## 8. Schema Enforcement vs Schema Evolution

### Schema Enforcement

Schema enforcement checks whether incoming writes are compatible with the table's existing schema.

It helps prevent accidental changes such as writing an incompatible data type into an established column.

### Schema Evolution

Schema evolution allows a table's schema to change when a write operation and its configuration explicitly permit it.

Example in a compatible Delta environment:

```python
df_new.write \
    .format("delta") \
    .mode("append") \
    .option("mergeSchema", "true") \
    .save("/data/sales_delta")
```

A new compatible column may be added through supported schema-evolution behavior.

### Difference

| Schema Enforcement | Schema Evolution |
|---|---|
| Protects the current schema | Allows approved schema changes |
| Rejects incompatible writes | Permits configured changes |
| Helps maintain data consistency | Helps accommodate legitimate source changes |

**Interview trap:** Schema evolution does not mean every schema change is safe. Validate new columns, types, nullability, and downstream dependencies.

---

## 9. Time Travel

Time Travel lets you query an earlier committed version of a Delta table.

### Query by Version

```sql
SELECT *
FROM sales VERSION AS OF 5;
```

### Query by Timestamp

```sql
SELECT *
FROM sales TIMESTAMP AS OF '2026-10-01 10:00:00';
```

Supported syntax and timestamp interpretation can vary by engine and platform.

### PySpark

```python
df_old = (
    spark.read
    .format("delta")
    .option("versionAsOf", 5)
    .load("/data/sales_delta")
)
```

### Why Use Time Travel?

- Investigate data-quality issues.
- Compare previous and current versions.
- Support debugging and recovery.
- Reproduce historical analysis.

### Important Limitation

Time Travel requires both the necessary transaction-log history and the data files for the requested version.

If `VACUUM` removes files needed by an older version, that version may no longer be queryable, even if some historical log information remains.

**Time Travel is not a substitute for a complete backup or disaster-recovery strategy.**

---

## 10. DESCRIBE HISTORY and RESTORE

### View Table History

```sql
DESCRIBE HISTORY sales;
```

History can show versions, timestamps, operations, and available operation details. Exact fields depend on the engine and version.

### Restore a Previous Version

```sql
RESTORE TABLE sales TO VERSION AS OF 5;
```

Where supported, `RESTORE` creates a new committed version whose data state reflects the selected historical version.

It does not erase the intervening history.

**Use case:** Recover a table after an accidental overwrite, provided the required files and history are still available.

---

## 11. OPTIMIZE

`OPTIMIZE` compacts small data files into fewer, larger files.

```sql
OPTIMIZE sales;
```

### Why Use It?

- Reduce small-file overhead.
- Reduce file-opening and metadata overhead.
- Improve read efficiency for many workloads.

### When Is It Useful?

- Many small files have accumulated.
- Frequent micro-batch writes produce fragmented data.
- Query performance is affected by excessive file counts.

### Important Points

- It consumes compute resources.
- It is not necessary after every write.
- File size should be chosen based on the workload and platform.
- Compaction does not automatically fix poor data modeling or expensive joins.

---

## 12. Z-ORDER and Data Skipping

Z-ORDER colocates related data values in files to improve data skipping for supported query patterns.

Example:

```sql
OPTIMIZE sales
ZORDER BY (customer_id);
```

This is supported on appropriate Delta environments; availability and recommendations depend on the platform.

### How It Helps

Suppose queries frequently filter on `customer_id`. Better data locality can help the engine skip files that are unlikely to contain matching values.

### Partitioning vs Z-ORDER

| Partitioning | Z-ORDER |
|---|---|
| Separates data into directory or partition groups | Improves data locality across files |
| Useful for suitable low-to-moderate-cardinality filters | Useful for selective filters on chosen columns |
| Too many partitions create small-file overhead | Does not create a separate partition for every value |
| Often affects write layout and directory structure | Reorganizes data files within supported table layouts |

**Remember:** Z-ORDER is not a traditional database index and does not guarantee that every query becomes faster.

---

## 13. VACUUM

`VACUUM` removes eligible obsolete data files that are no longer required by the current table snapshot, subject to retention and safety rules.

```sql
VACUUM sales;
```

Preview eligible files where supported:

```sql
VACUUM sales DRY RUN;
```

### Why Is VACUUM Needed?

Updates, deletes, merges, and overwrites can leave old data files that are no longer referenced by the current snapshot.

VACUUM can reclaim storage space by removing eligible files.

### Important Risks

- Removing files can make historical versions unavailable.
- It can interfere with long-running readers if retention is configured unsafely.
- Retention safety checks should not be disabled casually.
- It is not a command to run after every write.

Common defaults in some Delta environments include a seven-day deleted-file retention period and a longer transaction-log retention period, but values are configurable and platform-dependent.

### OPTIMIZE vs VACUUM

| OPTIMIZE | VACUUM |
|---|---|
| Reorganizes or compacts active table data | Removes eligible obsolete physical files |
| Targets file layout and read performance | Targets storage cleanup |
| Can create replacement files | Can delete old files |
| Does not by itself guarantee immediate removal of obsolete files | Can reduce the ability to query old versions |

---

## 14. Partitioning

Partitioning separates data into groups based on partition-column values.

Example:

```python
df.write \
    .format("delta") \
    .partitionBy("order_date") \
    .save("/data/sales_partitioned")
```

Queries that filter by the partition column may benefit from partition pruning.

### Good Partitioning Practices

- Select columns based on real query patterns.
- Avoid extremely high-cardinality partition columns.
- Avoid creating many tiny partitions.
- Consider table size and expected file volume.
- Reassess the layout as data volume and queries change.

**Interview point:** Partitioning is not automatically beneficial for every table. Small tables often do not need physical partitioning.

---

## 15. Delta Lake and Streaming

Delta Lake supports batch and streaming operations through compatible processing engines.

### Streaming Write Example

```python
query = (
    streaming_df.writeStream
    .format("delta")
    .outputMode("append")
    .option("checkpointLocation", "/checkpoints/sales")
    .start("/data/sales_delta")
)
```

The example assumes `streaming_df` is a streaming DataFrame and that the target and checkpoint paths are valid for the environment.

### Why Is Checkpointing Important?

Checkpointing stores streaming progress and, where applicable, state needed to recover a query.

It helps a supported streaming query resume after failure.

### Important Cautions

- Use a separate checkpoint location for each independent streaming query.
- Do not casually delete or reuse checkpoints.
- Checkpointing alone does not guarantee exactly-once effects for arbitrary external sinks.
- Delivery guarantees depend on source, sink, engine, and transaction behavior.
- Deduplication and event-time handling must be designed explicitly.
- A streaming write's output mode is not the same thing as delivery semantics.

---

## 16. Idempotency and Reliable MERGE Pipelines

An operation is idempotent when repeating it under the defined conditions produces the same intended final effect as running it once.

For reliable ingestion:

1. Define the business key.
2. Identify the source's change semantics.
3. Deduplicate records deterministically.
4. Apply changes using appropriate target semantics.
5. Track progress only after successful processing.
6. Make recovery and retries safe.
7. Reconcile the target with the source.

A Delta `MERGE` can help implement idempotent upserts, but **MERGE alone does not guarantee end-to-end idempotency**. The source ordering, matching condition, delete behavior, and retry logic must also be correct.

### Example: Latest Customer Record

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

window_spec = (
    Window
    .partitionBy("customer_id")
    .orderBy(
        F.col("updated_at").desc(),
        F.col("source_sequence").desc()
    )
)

latest_df = (
    source_df
    .withColumn("rn", F.row_number().over(window_spec))
    .filter(F.col("rn") == 1)
    .drop("rn")
)
```

The ordering columns must establish a deterministic winner. Add another stable tie-breaker if necessary.

---

## 17. Common Delta Lake Errors and Troubleshooting

| Problem | Possible Cause | First Checks |
|---|---|---|
| MERGE fails | Multiple source matches or incompatible schema | Check source duplicates and matching keys |
| Slow queries | Too many small files or poor data layout | Inspect file counts, query plan, and runtime metrics |
| Concurrent write conflict | Overlapping or incompatible transactions | Inspect error details and retry only when safe |
| Old version unavailable | Required files removed or history expired | Review retention, log history, and file availability |
| Schema mismatch | Incoming columns or types differ | Compare schemas and intended evolution policy |
| Storage keeps growing | Obsolete files or excessive output files | Review retention, compaction, and cleanup policy |
| Streaming query fails on restart | Checkpoint or source/sink state issue | Inspect checkpoint, query logs, and sink behavior |
| Duplicate records | Non-idempotent writes or inadequate deduplication | Verify business key, source ordering, and retry behavior |
| Unexpected row counts | Join fanout, duplicate source data, or bad filters | Validate grain, joins, keys, and business rules |

**Troubleshooting order:** Verify the error and table state first. Then inspect the plan, logs, file layout, schema, and source data. Avoid destructive cleanup before understanding the failure.

---

## 18. Delta Lake vs Other Table Formats

| Feature | Delta Lake | Apache Iceberg | Apache Hudi |
|---|---|---|---|
| Main purpose | Reliable data tables on data lakes | Reliable analytic tables on data lakes | Data-lake tables with strong ingestion and change-management capabilities |
| Table metadata | Transaction log | Metadata files and manifests | Timeline and table metadata |
| Schema evolution | Supported | Supported | Supported |
| Historical queries | Supported | Supported | Supported through supported timeline/query features |
| Engine support | Depends on integration and version | Depends on integration and version | Depends on integration and version |

No format is universally best. Compare the engines you use, concurrency requirements, update patterns, governance, operational complexity, and performance.

---

## 19. Delta Lake Interview Questions

### Q1. What is Delta Lake?

A storage layer that adds table-level transactions, schema controls, data-change operations, and historical querying to data stored in files.

### Q2. Why does Delta Lake use a transaction log?

To record committed table changes and determine which files belong to a valid table snapshot.

### Q3. How is Delta different from Parquet?

Parquet defines how columnar data is stored in files. Delta adds a transaction log and table-management capabilities around data files.

### Q4. What is ACID in Delta Lake?

Atomicity, consistency, isolation, and durability of supported transactions.

### Q5. What is MERGE?

An operation that matches source records to target records and applies configured updates, inserts, or deletes.

### Q6. What is Time Travel?

Querying an earlier committed table version or timestamp while the required history and data files remain available.

### Q7. What is OPTIMIZE?

A file-layout operation commonly used to compact small files.

### Q8. What is Z-ORDER?

A supported data-layout technique that can improve data skipping for suitable filters.

### Q9. What is VACUUM?

A cleanup operation that removes eligible obsolete physical data files.

### Q10. Can VACUUM affect Time Travel?

Yes. Removing files required by an older version can make that version unavailable.

### Q11. Does Delta Lake guarantee that MERGE never creates duplicates?

No. The source data, matching condition, business key, and transaction logic must be designed correctly.

### Q12. Does a checkpoint guarantee exactly-once processing?

No. End-to-end effects depend on source and sink guarantees and how transactions are committed.

### Q13. What causes a Delta write conflict?

Concurrent operations can have incompatible read/write dependencies. The exact conflict rules depend on the operation, isolation behavior, and platform features.

### Q14. What is the difference between schema enforcement and evolution?

Enforcement protects the existing schema. Evolution allows explicitly supported schema changes.

### Q15. Is Delta Lake a replacement for data modeling?

No. It provides table-storage capabilities; facts, dimensions, grain, keys, and business rules still need correct design.

---

## 20. Last-Minute Revision — 30 Key Facts

1. Delta Lake = data files + transaction log.
2. Parquet is a file format; Delta is a table layer.
3. `_delta_log` records committed table changes.
4. JSON files record commit actions.
5. Checkpoints help reconstruct table state efficiently.
6. A successful commit advances the table version.
7. ACID applies to supported table transactions.
8. Delta uses optimistic concurrency control.
9. Concurrent writes can conflict.
10. `MERGE` supports upserts and configured changes.
11. Deduplicate source data before ambiguous merges.
12. Schema enforcement protects schema compatibility.
13. Schema evolution permits approved changes.
14. Time Travel reads historical versions.
15. Historical reads require relevant logs and data files.
16. `DESCRIBE HISTORY` inspects table history.
17. `RESTORE` creates a new version reflecting a prior state.
18. `OPTIMIZE` reorganizes files.
19. Z-ORDER can improve data skipping.
20. Z-ORDER is not a traditional index.
21. `VACUUM` removes eligible obsolete files.
22. VACUUM can limit historical data availability.
23. Partitioning can support partition pruning.
24. High-cardinality partitioning can create too many small files.
25. Streaming checkpoints support recovery.
26. A checkpoint alone does not guarantee end-to-end exactly-once effects.
27. MERGE alone does not guarantee end-to-end idempotency.
28. File cleanup is different from table compaction.
29. Query plans and runtime metrics help diagnose performance.
30. A reliable pipeline requires correct business keys, data quality, recovery, and reconciliation.

---

## 21. Interview Readiness Checklist

- [ ] Explain Delta architecture and `_delta_log`.
- [ ] Explain ACID and optimistic concurrency control.
- [ ] Write a basic Delta `MERGE`.
- [ ] Explain schema enforcement vs evolution.
- [ ] Query an earlier version using Time Travel.
- [ ] Explain `OPTIMIZE`, Z-ORDER, and `VACUUM`.
- [ ] Explain partition pruning and small-file problems.
- [ ] Describe Delta streaming checkpoint behavior.
- [ ] Explain idempotency and safe retry design.
- [ ] Troubleshoot a slow query or failed merge.
- [ ] Explain why VACUUM can affect historical reads.
- [ ] Compare Delta Lake with Parquet, Iceberg, and Hudi.

**Final interview tip:** Explain each feature through three points: what it does, why it is useful, and what limitation or trade-off it introduces.