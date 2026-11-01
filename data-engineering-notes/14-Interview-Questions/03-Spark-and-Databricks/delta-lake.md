# Delta Lake – Interview Questions and Answers

## 1. Introduction to Delta Lake

### 1.1 What is Delta Lake?

Delta Lake is an open-source storage framework that brings reliability and transactional capabilities to data lakes.

It provides features such as:
- ACID transactions.
- Schema enforcement.
- Schema evolution.
- Time Travel.
- Updates, deletes, and MERGE operations.
- Transaction history.
- Support for batch and streaming workloads.

Delta Lake commonly stores data in Parquet files and maintains a transaction log in the `_delta_log` directory.

### 1.2 Why Do We Need Delta Lake?

A traditional data lake often stores files in formats such as CSV, JSON, and Parquet.

However, managing raw files directly can create problems:

- Partial writes can leave incomplete output.
- Concurrent writes can conflict.
- Schema changes can cause inconsistencies.
- Updating or deleting individual records is difficult.
- Tracking historical versions is not straightforward.
- Batch and streaming writes need reliable coordination.

Delta Lake addresses these challenges through its transaction log and table-management capabilities.

### 1.3 Delta Lake vs Parquet

| Parquet | Delta Lake |
|---|---|
| Columnar file format | Storage framework and table format |
| Stores data in columnar files | Commonly stores data in Parquet files |
| Does not provide table-level ACID transactions by itself | Provides transactional table operations |
| No built-in transaction history | Maintains transaction history |
| No built-in Time Travel | Supports Time Travel when required history and files are retained |
| Does not provide native MERGE semantics | Supports MERGE, UPDATE, and DELETE |

**Important:** Delta Lake is not a replacement for Parquet. Delta Lake uses Parquet data files along with a transaction log.

---

## 2. Delta Lake Architecture

### 2.1 Explain Delta Lake Architecture.

A Delta table consists primarily of:

1. Data files.
2. Transaction log.
3. Table metadata and protocol information recorded through the log.

Example directory structure:

```text
customer_delta/
|
|-- part-00000-....parquet
|-- part-00001-....parquet
|
|-- _delta_log/
    |
    |-- 00000000000000000000.json
    |-- 00000000000000000001.json
    |-- 00000000000000000002.json
    |
    |-- 00000000000000000010.checkpoint.parquet
```

The actual number and names of files depend on the table's write history and configuration.

### 2.2 What is the `_delta_log` Directory?

The `_delta_log` directory stores the transaction log for a Delta table.

It contains information about:
- Table metadata.
- Schema and configuration.
- Added data files.
- Removed data files.
- Transaction commits.
- Protocol requirements.
- Commit information.

The log allows Delta Lake to determine which files belong to a particular committed table version.

### 2.3 What is a Transaction Log?

A transaction log is an ordered record of successful table changes.

Each successful commit creates a new table version.

For example:

| Version | Operation |
|---|---|
| 0 | Initial table creation |
| 1 | Insert records |
| 2 | Update records |
| 3 | Delete records |
| 4 | Merge records |

These are illustrative operations. Actual version numbers depend on the table's history.

### 2.4 What are JSON Commit Files?

Delta Lake records commits in numbered JSON files.

For example:

```text
00000000000000000000.json
00000000000000000001.json
00000000000000000002.json
```

Each JSON file contains actions describing the changes made in that commit.

Common action types include:
- `add`
- `remove`
- `metaData`
- `protocol`
- `commitInfo`
- `txn`

The exact fields and actions depend on the operation and Delta protocol version.

### 2.5 What is a Checkpoint in Delta Lake?

A checkpoint summarizes the table state at a particular version.

Without checkpoints, reconstructing the latest table state could require replaying a long sequence of JSON commits.

A checkpoint helps reduce the amount of log history that must be replayed.

Important points:
- Checkpoints are generally stored in Parquet format.
- They summarize the state of a table at a version.
- Newer JSON commits after the checkpoint are replayed to reconstruct a later version.
- Checkpointing does not mean the underlying data files are compacted.

### 2.6 How Does Delta Lake Read a Table?

At a high level:

1. Identify the required table version.
2. Read the available checkpoint and subsequent transaction-log commits.
3. Reconstruct the active table state.
4. Identify the active data files.
5. Read the required data files.

Readers use a consistent committed snapshot rather than treating every file in the directory as active table data.

### 2.7 How Does Delta Lake Write Data?

At a high level:

1. The writer reads or evaluates the required input.
2. It writes new data files when needed.
3. It attempts to commit the corresponding changes to the transaction log.
4. A successful commit creates a new table version.
5. Readers can then see the committed snapshot.

If a write fails before its commit is successful, the newly written files may be left unreferenced. They are not automatically part of the committed table snapshot.

---

## 3. ACID Transactions

### 3.1 What are ACID Properties?

ACID stands for:

| Property | Meaning |
|---|---|
| Atomicity | A transaction is committed as a whole or not committed |
| Consistency | A transaction preserves applicable table and transaction rules |
| Isolation | Concurrent operations are coordinated to provide defined isolation behavior |
| Durability | A successful commit is retained according to the storage and platform guarantees |

### 3.2 How Does Delta Lake Provide ACID Transactions?

Delta Lake uses its transaction log and commit protocol to coordinate table changes.

The log records committed changes, and readers use committed snapshots.

This helps prevent readers from seeing a partially committed table update.

### 3.3 What is Optimistic Concurrency Control?

Optimistic Concurrency Control (OCC) is a method for coordinating concurrent transactions.

A simplified process:

1. A transaction reads the table state.
2. It performs its computation.
3. It checks whether conflicting changes occurred.
4. It attempts to commit.
5. If a conflict is detected, the transaction may fail and require a retry.

Delta Lake uses optimistic concurrency control. Exact conflict detection depends on the operation, isolation level, runtime, and enabled table features.

### 3.4 What Happens When Two Users Write to the Same Delta Table?

Both writers may start from overlapping table states.

If their operations conflict, one transaction may fail during commit.

The application should handle such failures safely:
- Identify the conflicting operation.
- Re-read the required table state.
- Recompute if necessary.
- Retry only when the operation is safe to retry.

Do not assume every concurrent write will fail. Some non-conflicting operations can succeed concurrently.

---

## 4. Schema Enforcement and Schema Evolution

### 4.1 What is Schema Enforcement?

Schema enforcement ensures that incoming data is compatible with the target table's schema.

Example:

Existing table:

```text
customer_id INT
customer_name STRING
age INT
```

Incoming data:

```text
customer_id INT
customer_name STRING
age STRING
```

The write may fail because the incoming `age` type is incompatible with the table's expected type.

Exact behavior depends on the write operation and runtime settings.

### 4.2 Why is Schema Enforcement Important?

It helps prevent:
- Unexpected column types.
- Accidental schema changes.
- Inconsistent records.
- Downstream transformation failures.
- Uncontrolled data quality problems.

### 4.3 What is Schema Evolution?

Schema evolution allows a Delta table schema to change as new data requirements appear.

For example, an existing table contains:

```text
customer_id
customer_name
```

New incoming data contains:

```text
customer_id
customer_name
email
```

Schema evolution can allow the new `email` column to be added to the table.

### 4.4 How Do You Enable Schema Evolution?

A common batch-write example:

```python
(
    new_df.write
    .format("delta")
    .mode("append")
    .option("mergeSchema", "true")
    .save("/mnt/delta/customers")
)
```

Schema evolution options and behavior can differ by operation, Delta version, and Databricks Runtime.

### 4.5 Schema Enforcement vs Schema Evolution

| Schema Enforcement | Schema Evolution |
|---|---|
| Checks incoming data against the target schema | Allows supported schema changes |
| Prevents incompatible changes | Supports controlled schema updates |
| Protects table structure | Adapts table structure |
| Helps maintain consistency | Helps accommodate changing requirements |

Schema evolution should be controlled. Automatically accepting every schema change can create downstream problems.

---

## 5. Delta Lake CRUD Operations

### 5.1 How Do You Create a Delta Table?

Using PySpark:

```python
df.write.format("delta").save(
    "/mnt/delta/customers"
)
```

Using SQL:

```sql
CREATE TABLE customers
USING DELTA
AS
SELECT *
FROM source_customers;
```

### 5.2 How Do You Insert Data?

```sql
INSERT INTO customers
VALUES
(101, 'Amit', 'amit@example.com');
```

The values must be compatible with the target table schema.

### 5.3 How Do You Update Data?

```sql
UPDATE customers
SET email = 'new@example.com'
WHERE customer_id = 101;
```

### 5.4 How Do You Delete Data?

```sql
DELETE FROM customers
WHERE customer_id = 101;
```

### 5.5 What Happens Internally During UPDATE and DELETE?

Delta Lake generally uses a copy-on-write approach for these operations.

At a high level:
- It identifies affected data files.
- It writes replacement files containing the resulting records.
- It records the removal of old files and addition of new files in the transaction log.
- The commit creates a new table version.

The old physical files may remain until they become eligible for cleanup.

---

## 6. MERGE Operation

### 6.1 What is MERGE in Delta Lake?

MERGE is used to perform insert, update, and delete operations based on matching conditions between source and target datasets.

It is commonly used for:
- Incremental data loads.
- Upserts.
- CDC processing.
- Dimension-table updates.
- Deduplication workflows.

### 6.2 Example of MERGE

Assume:
- `customers` is the target Delta table.
- `updates` contains incoming customer records.
- `customer_id` is the business key.

```sql
MERGE INTO customers AS target
USING updates AS source
ON target.customer_id = source.customer_id

WHEN MATCHED THEN
    UPDATE SET
        target.customer_name = source.customer_name,
        target.email = source.email

WHEN NOT MATCHED THEN
    INSERT (
        customer_id,
        customer_name,
        email
    )
    VALUES (
        source.customer_id,
        source.customer_name,
        source.email
    );
```

### 6.3 How Does MERGE Work?

1. Compare source and target using the matching condition.
2. Identify matching records.
3. Update matching records according to the specified clauses.
4. Insert unmatched source records when an insert clause is defined.
5. Delete matching records if a delete clause is defined.
6. Commit the resulting changes as a new table version.

### 6.4 What is the Difference Between UPDATE and MERGE?

| UPDATE | MERGE |
|---|---|
| Updates matching target rows | Supports multiple conditional actions |
| Does not insert unmatched source rows | Can insert unmatched source rows |
| Simpler update operation | Useful for upsert and CDC |
| Operates on target rows | Compares source and target |

### 6.5 What Happens If Multiple Source Rows Match One Target Row?

Multiple source rows matching the same target row can make an update ambiguous.

Depending on the operation and runtime, MERGE may fail.

Before MERGE:
- Deduplicate the source by business key.
- Define a deterministic rule for selecting the latest record.
- Validate that the source has the expected grain.

Example using a window function:

```python
from pyspark.sql import Window
from pyspark.sql.functions import col, row_number

window_spec = (
    Window
    .partitionBy("customer_id")
    .orderBy(
        col("updated_at").desc(),
        col("source_sequence").desc()
    )
)

deduplicated_source = (
    updates
    .withColumn(
        "row_num",
        row_number().over(window_spec)
    )
    .filter(col("row_num") == 1)
    .drop("row_num")
)
```

The ordering columns should provide a deterministic tie-breaker.

---

## 7. Time Travel

### 7.1 What is Time Travel in Delta Lake?

Time Travel allows users to query an earlier committed version of a Delta table.

It is useful for:
- Auditing.
- Debugging.
- Comparing historical data.
- Recovering from accidental changes.
- Investigating data quality problems.

### 7.2 How Do You Query an Earlier Version?

Using SQL:

```sql
SELECT *
FROM customers VERSION AS OF 5;
```

Using a timestamp:

```sql
SELECT *
FROM customers
TIMESTAMP AS OF '2026-01-01 10:00:00';
```

### 7.3 How Do You Access an Earlier Version Using PySpark?

```python
df_old = (
    spark.read
    .format("delta")
    .option("versionAsOf", 5)
    .load("/mnt/delta/customers")
)
```

### 7.4 Does Time Travel Store a Complete Copy of the Table for Every Version?

No.

Delta Lake generally records changes through its transaction log and references data files.

Multiple versions can share unchanged data files. New files are written for changed data, while old files may remain available until cleanup.

### 7.5 What Can Prevent Time Travel?

Time Travel requires the necessary transaction-log history and underlying data files to remain available.

It may become unavailable when:
- Old log history has been cleaned up.
- Required data files have been removed by VACUUM.
- Retention settings do not preserve the requested version.

Time Travel is not a permanent backup strategy.

---

## 8. VACUUM and Retention

### 8.1 What is VACUUM?

VACUUM removes unreferenced data files that are eligible for cleanup based on the table's retention rules.

These files may include older data files no longer referenced by the current table snapshot.

Example:

```sql
VACUUM customers;
```

### 8.2 What is VACUUM DRY RUN?

DRY RUN previews the files that would be eligible for removal without deleting them.

```sql
VACUUM customers DRY RUN;
```

Use it to inspect cleanup candidates before performing actual cleanup.

### 8.3 What is the Default VACUUM Retention Period?

A commonly used default is seven days, but the effective retention behavior depends on table properties, runtime, and platform configuration.

Do not assume that every Delta table has the same retention configuration.

### 8.4 Why Should We Be Careful With VACUUM?

VACUUM can permanently remove files needed by historical versions.

After those files are deleted, queries against older versions may fail.

Before running VACUUM:
- Understand the retention requirements.
- Consider long-running readers and writers.
- Check streaming and recovery requirements.
- Avoid disabling retention safety checks without a clear reason.

### 8.5 What is the Difference Between VACUUM and Log Cleanup?

| VACUUM | Log Cleanup |
|---|---|
| Removes eligible unreferenced data files | Removes expired transaction-log history and related log artifacts |
| Affects physical data-file availability | Affects historical table-state reconstruction |
| Can make old versions unreadable | Can also make old versions inaccessible |
| Uses data-file retention rules | Uses log-retention rules |

A commonly used default for transaction-log retention is 30 days, but the effective value can be configured.

### 8.6 What is the Difference Between Data Retention and Log Retention?

- **Data-file retention:** Determines how long unreferenced physical data files remain available.
- **Log retention:** Determines how long transaction-log history is retained.

Time Travel needs both the required log history and the corresponding data files.

---

## 9. OPTIMIZE and ZORDER

### 9.1 What is OPTIMIZE?

OPTIMIZE is used to compact small files into larger files.

Small files can increase:
- File listing overhead.
- Metadata operations.
- Query planning time.
- Task scheduling overhead.

Example:

```sql
OPTIMIZE customers;
```

### 9.2 What is ZORDER?

ZORDER reorganizes data in files to improve data skipping for selected columns.

Example:

```sql
OPTIMIZE customers
ZORDER BY (customer_id);
```

It can help queries that frequently filter on suitable Z-ordered columns.

### 9.3 OPTIMIZE vs ZORDER

| OPTIMIZE | ZORDER |
|---|---|
| Compacts small files | Reorganizes data locality |
| Reduces file-management overhead | Can improve data skipping |
| Does not inherently cluster by chosen columns | Uses selected columns for multi-dimensional locality |
| Can be used independently | Commonly used with OPTIMIZE |

### 9.4 Does ZORDER Create an Index?

No. ZORDER is not a traditional database index.

It improves data layout so that file-level statistics can help Spark skip irrelevant files.

### 9.5 What is Data Skipping?

Data skipping uses file statistics and metadata to avoid reading files that cannot satisfy a query filter.

For example, if a file's minimum and maximum `customer_id` values do not overlap with a filter range, the file may be skipped.

Effectiveness depends on:
- Data layout.
- Statistics availability.
- Filter columns.
- Data distribution.
- Table configuration.

### 9.6 What is Liquid Clustering?

Liquid clustering is a newer Delta table data-layout capability supported in certain Databricks environments.

It provides an alternative to traditional partitioning and ZORDER for supported workloads.

Availability and supported features depend on the platform and runtime. Do not assume it is available in every Delta Lake deployment.

---

## 10. Delta Lake Partitioning

### 10.1 What is Partitioning?

Partitioning divides table data into directory-level groups based on one or more column values.

Example:

```python
(
    df.write
    .format("delta")
    .partitionBy("year", "month")
    .save("/mnt/delta/transactions")
)
```

Example directory structure:

```text
transactions/
|
|-- year=2025/
|   |-- month=01/
|   |-- month=02/
|
|-- year=2026/
    |-- month=01/
    |-- month=02/
```

### 10.2 What are the Benefits of Partitioning?

- Can reduce the amount of data read.
- Supports partition pruning.
- Organizes data by common filter columns.
- Can improve certain large-table workloads.

### 10.3 What are the Problems With Over-Partitioning?

Too many partitions can create:
- Small files.
- Large directory counts.
- Metadata overhead.
- Poor write performance.
- Expensive maintenance.

### 10.4 Which Columns Should Be Used for Partitioning?

Choose columns based on:
- Query filter patterns.
- Data volume.
- Cardinality.
- Data distribution.
- Typical partition size.

Date-based columns are often useful for large time-oriented datasets.

Avoid high-cardinality columns such as unique customer IDs as partition columns in typical lakehouse designs.

---

## 11. Delta Lake in Batch and Streaming

### 11.1 Can Delta Lake Support Batch Processing?

Yes.

Example:

```python
df = spark.read.parquet(
    "/mnt/raw/customers/"
)

(
    df.write
    .format("delta")
    .mode("overwrite")
    .save("/mnt/delta/customers")
)
```

### 11.2 Can Delta Lake Support Streaming?

Yes. Delta tables can be used as Structured Streaming sources and sinks.

Example:

```python
stream_df = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .load("/mnt/raw/events/")
)

query = (
    stream_df.writeStream
    .format("delta")
    .option(
        "checkpointLocation",
        "/mnt/checkpoints/events/"
    )
    .outputMode("append")
    .start("/mnt/delta/events/")
)
```

This example uses Auto Loader for file discovery in a Databricks environment. Auto Loader availability and configuration are platform-specific.

### 11.3 What is the Role of Checkpointing in Streaming?

Structured Streaming checkpointing stores query progress and, for stateful queries, relevant state information.

It helps the streaming query recover after failures.

A checkpoint does not automatically guarantee exactly-once results for every external sink. End-to-end guarantees depend on source, processing, sink, and write semantics.

### 11.4 Can Batch and Streaming Read the Same Delta Table?

Yes, subject to the table's supported features and the read mode.

A Delta table can serve as a shared storage layer for batch and streaming workloads.

---

## 12. Delta Lake Table Management

### 12.1 Managed vs External Delta Table

| Managed Table | External Table |
|---|---|
| Catalog manages the table's storage lifecycle | Data is stored at a specified external location |
| Dropping the table can remove underlying data | Dropping the table generally removes catalog metadata but leaves external data |
| Location is managed by the platform | Location is explicitly specified or externally controlled |

Exact lifecycle behavior depends on the catalog and platform.

### 12.2 How Do You Create an External Delta Table?

```sql
CREATE TABLE external_customers
USING DELTA
LOCATION '/mnt/delta/customers';
```

The specified location must contain a compatible Delta table or be suitable for table creation.

### 12.3 What Happens When You Drop a Delta Table?

The result depends on whether the table is managed or external and on the catalog implementation.

- Dropping a managed table may remove both metadata and underlying data.
- Dropping an external table generally removes the catalog registration while leaving the underlying data.

Always verify the table's ownership and storage lifecycle before dropping it.

### 12.4 What is DESCRIBE HISTORY?

It displays the transaction history of a Delta table.

```sql
DESCRIBE HISTORY customers;
```

It can show information such as:
- Version.
- Timestamp.
- Operation.
- Operation parameters.
- User or job information when available.
- Metrics when available.

Useful for auditing and debugging.

### 12.5 What is RESTORE?

RESTORE changes the current table state to match an earlier version or timestamp by creating a new commit.

Example:

```sql
RESTORE TABLE customers
TO VERSION AS OF 5;
```

RESTORE does not erase intervening history. It creates a new version representing the restored state.

---

## 13. Delta Lake vs Traditional Data Lake

| Traditional Data Lake | Delta Lake |
|---|---|
| Stores raw files | Stores data files with a transaction log |
| File-level operations are common | Table-level transactional operations |
| Schema management may be external | Schema enforcement and evolution support |
| Historical versions need separate handling | Supports Time Travel |
| Updates and deletes require custom processing | Supports UPDATE, DELETE and MERGE |
| Concurrent writes need external coordination | Provides transactional concurrency control |

Delta Lake adds reliability to a data lake; it does not eliminate the need for data governance, quality checks, or backups.

---

## 14. Practical Data Engineering Scenario

**Question:** You receive daily customer updates from a source system. Some records are new, some are updated, and some customers may be deleted. How would you build an incremental pipeline using Delta Lake?

### Step 1: Ingest Source Data

Read incoming records into a staging DataFrame.

Include:
- Customer ID.
- Customer name.
- Email.
- Updated timestamp.
- Source operation or deletion indicator, if available.

### Step 2: Validate the Data

Check:
- Required customer ID.
- Valid data types.
- Null values.
- Duplicate customer IDs.
- Invalid timestamps.
- Supported operation values.

### Step 3: Deduplicate Source Records

If multiple updates exist for one customer, select the correct record using a deterministic ordering rule.

### Step 4: Apply Changes With MERGE

For example:

```sql
MERGE INTO customers AS target
USING customer_updates AS source
ON target.customer_id = source.customer_id

WHEN MATCHED AND source.operation = 'DELETE'
    THEN DELETE

WHEN MATCHED AND source.operation <> 'DELETE'
    THEN UPDATE SET
        target.customer_name = source.customer_name,
        target.email = source.email,
        target.updated_at = source.updated_at

WHEN NOT MATCHED AND source.operation <> 'DELETE'
    THEN INSERT (
        customer_id,
        customer_name,
        email,
        updated_at
    )
    VALUES (
        source.customer_id,
        source.customer_name,
        source.email,
        source.updated_at
    );
```

This is an illustrative CDC pattern. The source must reliably provide deletion events and operation semantics.

### Step 5: Validate the Target

Check:
- Record counts.
- Duplicate business keys.
- Null constraints.
- Insert/update/delete counts.
- Source-to-target reconciliation.

### Step 6: Monitor and Recover

Record:
- Pipeline run ID.
- Source batch or CDC position.
- Processing timestamp.
- Records inserted, updated, and deleted.
- Failed records.

Make the pipeline safe to retry. For example, use deterministic source batches and idempotent target logic.

**Interview explanation:** I would ingest the source changes, validate and deduplicate them, then use Delta MERGE to apply inserts, updates, and deletes. I would also track source progress, reconcile the target, and design retries so that a rerun does not create duplicate or incorrect results.

---

## 15. Important Delta Lake Interview Questions

### Q1. What is Delta Lake?

Delta Lake is a storage framework that adds ACID transactions, schema enforcement, Time Travel, and table-management features to data lakes.

### Q2. What is the difference between Delta Lake and Parquet?

Parquet is a columnar file format. Delta Lake is a table framework that commonly uses Parquet data files and a transaction log.

### Q3. What is the purpose of `_delta_log`?

It records committed table changes and allows Delta Lake to reconstruct the active table state.

### Q4. What is a Delta transaction?

A transaction is a table operation that becomes visible through a successful commit and creates a new table version.

### Q5. What is ACID in Delta Lake?

ACID means Atomicity, Consistency, Isolation, and Durability. Delta Lake uses its transaction log and commit protocol to provide transactional table behavior.

### Q6. What is schema enforcement?

It checks incoming data against the target table schema and rejects unsupported incompatible writes.

### Q7. What is schema evolution?

It allows supported changes to a table's schema, such as adding new columns.

### Q8. What is MERGE?

MERGE compares source and target records and supports conditional updates, inserts, and deletes.

### Q9. What is Time Travel?

It allows users to query an earlier committed table version, provided the required log and data files remain available.

### Q10. What is VACUUM?

VACUUM removes eligible unreferenced data files according to retention rules.

### Q11. What is OPTIMIZE?

OPTIMIZE compacts small files to improve file layout and reduce overhead.

### Q12. What is ZORDER?

ZORDER reorganizes data to improve data skipping for suitable filter columns. It is not a traditional index.

### Q13. What is the difference between OPTIMIZE and VACUUM?

OPTIMIZE compacts active data files. VACUUM removes eligible unreferenced data files.

### Q14. What is the difference between a checkpoint and a transaction log?

Transaction-log commits record table changes. A checkpoint summarizes table state to reduce the work required to reconstruct a snapshot.

### Q15. What is the difference between log retention and data retention?

Log retention controls historical transaction-log availability. Data retention controls how long unreferenced data files remain available.

### Q16. What happens if VACUUM removes old data files?

Historical versions that depend on those files may no longer be readable.

### Q17. Can Delta Lake handle concurrent writes?

Yes. Delta Lake uses optimistic concurrency control. Conflicting operations may fail and require safe retry.

### Q18. Is Delta Lake a database?

No. It is a table and storage framework for data lakes, commonly used with distributed processing engines such as Spark.

### Q19. Does Delta Lake guarantee exactly-once processing?

Delta Lake provides transactional table writes, but end-to-end exactly-once behavior depends on the source, processing engine, sink, checkpointing, and write design.

### Q20. What is the purpose of DESCRIBE HISTORY?

It displays table transaction history and helps with auditing and troubleshooting.

---

## 16. Common Mistakes to Avoid

1. **Saying Delta Lake is a file format:** Delta Lake is a table/storage framework; Parquet is a file format.
2. **Saying VACUUM deletes all old records:** It removes eligible unreferenced physical files.
3. **Confusing OPTIMIZE and VACUUM:** OPTIMIZE compacts active files; VACUUM cleans eligible unreferenced files.
4. **Assuming Time Travel is permanent:** It depends on log and data-file retention.
5. **Assuming MERGE automatically handles duplicates:** Multiple source rows matching one target row can cause ambiguity or failure.
6. **Assuming schema evolution accepts every change:** Supported changes depend on the operation and runtime.
7. **Assuming every concurrent write succeeds:** Conflicting operations can fail.
8. **Treating ZORDER as a traditional index:** It improves data layout and data skipping.
9. **Assuming checkpoints contain the full physical dataset:** They summarize transaction-log state.
10. **Assuming Delta Lake replaces backups:** It is not a complete backup and disaster-recovery strategy.

---

## 17. Quick Revision

| Concept | Key Point |
|---|---|
| Delta Lake | Reliable table framework for data lakes |
| Parquet | Columnar file format |
| `_delta_log` | Transaction log directory |
| JSON Commit | Records a table commit |
| Checkpoint | Summarizes table state |
| ACID | Atomicity, Consistency, Isolation, Durability |
| OCC | Optimistic Concurrency Control |
| Schema Enforcement | Validates incoming schema |
| Schema Evolution | Supports controlled schema changes |
| MERGE | Conditional insert, update, and delete |
| Time Travel | Query earlier table versions |
| VACUUM | Removes eligible unreferenced files |
| Log Retention | Controls historical log availability |
| Data Retention | Controls old data-file availability |
| OPTIMIZE | Compacts small files |
| ZORDER | Improves data locality and data skipping |
| Partitioning | Organizes data by partition columns |
| Data Skipping | Avoids reading irrelevant files |
| DESCRIBE HISTORY | Displays transaction history |
| RESTORE | Creates a new version matching an older state |
| Managed Table | Platform manages table lifecycle |
| External Table | Data location is externally controlled |

---

## 18. Self-Assessment

Before a Data Engineer interview, make sure you can:

- [ ] Explain Delta Lake architecture.
- [ ] Explain the `_delta_log` directory.
- [ ] Differentiate JSON commits and checkpoints.
- [ ] Explain ACID transactions.
- [ ] Explain optimistic concurrency control.
- [ ] Differentiate schema enforcement and schema evolution.
- [ ] Write INSERT, UPDATE, and DELETE statements.
- [ ] Write a MERGE statement.
- [ ] Explain how to deduplicate MERGE source data.
- [ ] Explain Time Travel.
- [ ] Explain VACUUM and retention.
- [ ] Differentiate OPTIMIZE, ZORDER, and VACUUM.
- [ ] Explain partitioning and data skipping.
- [ ] Explain batch and streaming integration.
- [ ] Explain managed and external tables.
- [ ] Use DESCRIBE HISTORY and RESTORE.
- [ ] Design a simple incremental CDC pipeline.

**Preparation priority:** First master Delta architecture, transaction logs, ACID, MERGE, Time Travel, VACUUM, and OPTIMIZE. Then focus on concurrency, schema evolution, partitioning, streaming, and practical CDC scenarios.