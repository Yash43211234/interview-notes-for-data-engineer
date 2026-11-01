# Apache Hudi Fundamentals

## 1. What Is Apache Hudi?

**Apache Hudi** stands for **Hadoop Upserts Deletes and Incrementals**.

It is an open-source table format designed to manage large datasets in data lakes, especially when data is frequently inserted, updated, or deleted.

Traditional data lakes commonly store data as Parquet files. However, managing updates and deletes directly in large collections of files can be difficult.

Hudi adds table-management capabilities such as:

- Upserts and deletes.
- ACID transactions.
- Incremental queries.
- Change Data Capture (CDC) support.
- Schema evolution.
- Time Travel.
- Record-level indexing.
- Data compaction.

**Simple way to remember:**

Apache Hudi = Data lake table format focused on upserts, incremental processing, and managing changing data.

## 2. Why Do We Need Apache Hudi?

Suppose an e-commerce company receives order data every day.

Initially:

| order_id | customer_id | amount |
|---|---|---:|
| 101 | 501 | 500 |
| 102 | 502 | 700 |
| 103 | 503 | 900 |

Later, the source system sends an update:

| order_id | customer_id | amount |
|---|---|---:|
| 101 | 501 | 1000 |
| 104 | 504 | 600 |

The pipeline needs to:

1. Update order 101.
2. Insert order 104.
3. Keep unchanged records.
4. Avoid duplicate records.
5. Track changes over time.

With ordinary Parquet files, the pipeline needs to manage these operations itself.

Hudi provides table-management mechanisms to handle these changing records.

### Main Benefits

1. Efficient upserts and deletes.
2. Incremental data processing.
3. Support for CDC pipelines.
4. Reliable table operations.
5. Record-level indexing.
6. Batch and streaming integration.
7. Historical data access.

## 3. Apache Hudi Architecture

Hudi manages table metadata and data files through a combination of file groups, timelines, and metadata structures.

A simplified architecture:

```text
              HUDI TABLE
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
    Timeline   File Groups   Metadata
       |           |
       |      +----+----+
       |      |         |
       v      v         v
    Commits  Base     Log Files
             Files    (MOR)
```

### Main Components

| Component | Purpose |
|---|---|
| Timeline | Tracks table actions and commits |
| File Group | Organizes the files associated with a record group |
| Base File | Stores the main data, commonly in Parquet |
| Log File | Stores changes in Merge-on-Read tables |
| Index | Helps locate records for updates |
| Metadata Table | Stores auxiliary metadata to improve operations |

## 4. Hudi Timeline

The **timeline** is one of the most important concepts in Apache Hudi.

It tracks actions performed on a Hudi table.

Examples include:

- Commit.
- Delta commit.
- Compaction.
- Clean.
- Rollback.
- Savepoint.
- Replace commit.

### Example

Suppose a Hudi table receives three successful writes.

```text
          HUDI TIMELINE

          Commit 001
              |
              v
          Commit 002
              |
              v
          Commit 003
```

Each completed write creates a new table state.

The timeline helps Hudi understand:

- Which operations were completed.
- Which table state is committed.
- Which data files belong to a particular table state.
- How to support incremental and historical queries.

### Important Timeline Concepts

| Action | Purpose |
|---|---|
| Commit | Records a completed write operation |
| Delta Commit | Records changes in a Merge-on-Read write |
| Compaction | Merges log records into base files |
| Clean | Removes eligible obsolete file versions |
| Rollback | Reverts an unsuccessful or rolled-back operation |
| Savepoint | Protects selected file versions from cleaning |

**Remember:** The timeline provides the history of table actions and helps readers identify valid committed table states.

## 5. Hudi File Groups and File Slices

These concepts help explain how Hudi organizes records.

### File Group

A file group represents a logical group of files associated with a particular file ID and partition.

A file group can contain multiple versions of a base file over time and associated log files.

### File Slice

A file slice represents a particular version of a file group at a specific point in the timeline.

It generally consists of:

- A base file, if present.
- Log files associated with that base file.

### Example

```text
Partition: 2026-10

File Group: file_group_01
       |
       +-- Base File
       |
       +-- Log File 1
       |
       +-- Log File 2
```

A later file slice may contain a newer base file and a different set of log files.

**Interview point:** A file group is the logical identity of a group of files. A file slice represents its state at a particular point in time.

## 6. Hudi Table Types

Hudi supports two main table types:

1. Copy-on-Write (COW).
2. Merge-on-Read (MOR).

The difference is mainly how updates are stored and how reads access them.

### 6.1 Copy-on-Write (COW)

In Copy-on-Write tables, updates rewrite affected base files.

Example:

Before update:

```text
Base File
    |
    v
order_id | amount
-----------------
101      | 500
102      | 700
```

After updating order 101:

```text
New Base File
    |
    v
order_id | amount
-----------------
101      | 1000
102      | 700
```

The updated data is written into a new base-file version.

### Characteristics

- Updates rewrite affected base files.
- Reads generally access base files directly.
- Read behavior is relatively simple.
- Frequent updates can increase write amplification.
- No log-file merging is required for ordinary reads.

**Best suited for:** Workloads where read performance is important and updates are less frequent.

### 6.2 Merge-on-Read (MOR)

In Merge-on-Read tables, updates can be stored in log files instead of immediately rewriting the entire base file.

Example:

```text
              MOR TABLE
                  |
          +-------+-------+
          |               |
          v               v
       Base File       Log File
          |               |
          |         Updated Records
          |               |
          +-------+-------+
                  |
                  v
             Merged View
```

When a query reads the table, Hudi can merge the base and log records according to the selected query mode.

### Characteristics

- Updates can be written to log files.
- Reads may need to merge base and log files.
- Supports write-efficient update patterns.
- Compaction merges log records into base files.
- Read performance can be affected by accumulated log files.

**Best suited for:** Workloads with frequent updates and requirements for efficient data ingestion.

### COW vs MOR

| Feature | Copy-on-Write | Merge-on-Read |
|---|---|---|
| Update behavior | Rewrites affected base files | Can write changes to log files |
| Base files | Yes | Yes |
| Log files | Not used for the normal COW write path | Used for updates |
| Read behavior | Reads base files | May merge base and log files |
| Write behavior | More write amplification for updates | Can support lower-latency updates |
| Compaction | No MOR log compaction required | Important for merging log files |
| Suitable workload | Read-heavy | Update-heavy |

**Quick memory trick:**

- COW = Rewrite base files.
- MOR = Write logs and merge later.

## 7. Upserts in Apache Hudi

An upsert combines two operations:

- Update an existing record.
- Insert a new record if it does not exist.

Example:

Existing data:

| order_id | amount |
|---|---:|
| 101 | 500 |
| 102 | 700 |

Incoming data:

| order_id | amount |
|---|---:|
| 101 | 1000 |
| 103 | 600 |

After upsert:

| order_id | amount |
|---|---:|
| 101 | 1000 |
| 102 | 700 |
| 103 | 600 |

Hudi identifies records using a record key and uses its indexing and write mechanisms to locate and update existing records.

### Why Are Upserts Important?

- Avoid duplicate business records.
- Handle changing source data.
- Support CDC pipelines.
- Maintain current table state.
- Process updates efficiently.

## 8. Record Keys and Indexing

Hudi uses record keys to identify records.

A record key uniquely identifies a record within the relevant Hudi table and partitioning model.

Example:

```text
order_id = 101
```

If `order_id` is configured as the record key, Hudi uses it to identify the order during updates and deletes.

### Indexing

An index helps Hudi locate existing records efficiently.

Without an effective lookup mechanism, finding records to update can require expensive data-file scans.

Hudi supports different indexing strategies depending on the configuration and version.

Examples include:

- Simple index.
- Bloom filter index.
- Record-level indexing capabilities.
- Other supported index implementations.

### Why Is Indexing Important?

1. Helps locate existing records.
2. Supports upserts.
3. Reduces unnecessary file scanning in suitable workloads.
4. Improves write performance.

**Important:** Index behavior and available implementations depend on the Hudi version and configuration.

## 9. Incremental Processing in Hudi

Incremental processing means reading only the changes that occurred after a particular point instead of reading the entire table again.

Suppose a Hudi table contains one billion records.

A pipeline needs only the records updated since its previous successful run.

Instead of scanning the entire table, it can use Hudi's incremental query capabilities.

### Example

```text
Initial Table
     |
     v
Commit 001
     |
     v
Commit 002
     |
     v
Commit 003
```

If the pipeline has already processed Commit 002, it can request changes after that point using supported incremental query functionality.

### Benefits

- Reduces unnecessary data processing.
- Supports efficient downstream pipelines.
- Useful for CDC.
- Reduces repeated full-table scans.
- Helps build near-real-time data pipelines.

**Important:** Incremental query semantics depend on the selected query mode and Hudi version. A commit-based incremental query is not automatically identical to a complete row-level CDC event stream.

## 10. Change Data Capture (CDC)

CDC means capturing changes made to source data.

Common change types include:

- INSERT.
- UPDATE.
- DELETE.

Example:

Source database changes:

```text
INSERT: order_id 104
UPDATE: order_id 101
DELETE: order_id 102
```

A Hudi pipeline can ingest and apply these changes to maintain the target table.

### CDC Pipeline

```text
       SOURCE DATABASE
               |
               v
         CDC EVENTS
               |
               v
        DATA INGESTION
               |
               v
          HUDI TABLE
               |
               v
       INCREMENTAL QUERY
               |
               v
       DOWNSTREAM TABLE
```

### Why Is Hudi Useful for CDC?

- Supports upserts.
- Supports deletes.
- Tracks committed changes.
- Provides incremental query capabilities.
- Supports batch and streaming ingestion patterns.

**Important:** Correct CDC processing requires handling event ordering, duplicate events, delete events, and source transaction semantics. Hudi does not automatically solve every source-system CDC problem.

## 11. Compaction in Hudi

Compaction is especially important for Merge-on-Read tables.

In MOR tables, updates can accumulate in log files.

Over time, reading the table may require merging many log records with base files.

Compaction combines these changes into new base files.

### Before Compaction

```text
Base File
    |
    +-- Log File 1
    |
    +-- Log File 2
    |
    +-- Log File 3
```

### After Compaction

```text
New Base File
    |
    v
Updated Records
```

### Benefits

1. Reduces accumulated log files.
2. Improves read performance.
3. Creates more efficient base files.
4. Simplifies subsequent reads.

**Important:** Compaction is not the same as cleaning.

- Compaction merges log data into base files.
- Cleaning removes eligible obsolete file versions.

## 12. Cleaning in Hudi

Cleaning removes older file versions that are no longer required according to the configured retention policy.

Hudi can retain older file versions to support historical reads and other operations.

However, keeping too many old versions increases storage consumption.

### Example

```text
Old File Version
       |
       v
New File Version
       |
       v
Latest File Version
```

After the configured retention conditions are met, cleaning can remove eligible obsolete files.

### Why Is Cleaning Important?

- Controls storage growth.
- Removes unnecessary file versions.
- Helps maintain table storage efficiently.

**Remember:**

| Operation | Purpose |
|---|---|
| Compaction | Merges log files into base files |
| Cleaning | Removes eligible old file versions |

## 13. Time Travel in Hudi

Time Travel allows users to query historical table states.

Hudi tracks table actions through its timeline.

This helps identify table states associated with earlier completed commits.

Example:

```text
Commit 001
    |
    v
Commit 002
    |
    v
Commit 003
```

A historical query may access the table state associated with an earlier commit, provided the required data files and timeline information remain available.

### Why Is Time Travel Useful?

- Investigate incorrect data.
- Compare historical table states.
- Support auditing.
- Recover or analyze previous data versions.

**Important:** Historical access depends on retention, cleaning, and the availability of the required files.

## 14. Hudi Query Types

Hudi provides different query modes depending on the table type and required data view.

Common query types include:

### 14.1 Snapshot Query

Returns the latest committed table state.

For a Merge-on-Read table, the snapshot view includes the applicable merging of base and log records.

### 14.2 Read Optimized Query

Primarily reads base files without merging the latest log files.

This is relevant to Merge-on-Read tables.

It can provide faster reads but may not include the latest un-compacted updates.

### 14.3 Incremental Query

Reads changes after a specified commit or point in the timeline, according to the supported incremental query semantics.

### Comparison

| Query Type | Purpose |
|---|---|
| Snapshot | Read the latest committed table state |
| Read Optimized | Read base files without merging recent log files |
| Incremental | Read changes since a specified point |

**Interview point:** Snapshot and read-optimized queries can return different results for MOR tables when recent changes have not yet been compacted.

## 15. Practical Example: Hudi with PySpark

Hudi can be used with Spark through its supported Spark integration.

The following is a simplified example. It assumes the compatible Hudi Spark bundle is available in the Spark environment.

### Step 1: Create Sample Data

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("HudiExample") \
    .getOrCreate()

data = [
    (101, "Rahul", 500),
    (102, "Priya", 700),
    (103, "Amit", 900)
]

columns = ["order_id", "customer_name", "amount"]

df = spark.createDataFrame(data, columns)
```

### Step 2: Configure Hudi Options

```python
hudi_options = {
    "hoodie.table.name": "orders",
    "hoodie.datasource.write.recordkey.field": "order_id",
    "hoodie.datasource.write.precombine.field": "amount",
    "hoodie.datasource.write.operation": "upsert",
    "hoodie.datasource.write.table.type": "COPY_ON_WRITE"
}
```

The record key identifies records.

The precombine field helps choose the appropriate record when multiple incoming records share the same key, according to the configured ordering behavior.

In production, use a suitable source-update timestamp or sequence field rather than an arbitrary business measure such as `amount`.

### Step 3: Write Data

```python
table_path = "/tmp/hudi/orders"

df.write \
    .format("hudi") \
    .options(**hudi_options) \
    .mode("overwrite") \
    .save(table_path)
```

### Step 4: Read Data

```python
hudi_df = spark.read \
    .format("hudi") \
    .load(table_path)

hudi_df.show()
```

### Step 5: Prepare Updated Records

```python
updated_data = [
    (101, "Rahul", 1000),
    (104, "Neha", 600)
]

updated_df = spark.createDataFrame(
    updated_data,
    columns
)
```

### Step 6: Upsert Updated Records

```python
updated_df.write \
    .format("hudi") \
    .options(**hudi_options) \
    .mode("append") \
    .save(table_path)
```

The configured upsert operation updates order 101 and inserts order 104.

**Important implementation notes:**

- The Hudi Spark bundle must match the Spark and Scala versions in the environment.
- Hudi write options and APIs vary by Hudi version.
- A production pipeline should use a deterministic precombine field.
- Record-key and partition-path design must match the table's requirements.
- `overwrite` can replace table data and should be used carefully.
- A real CDC pipeline must handle delete events and duplicate or out-of-order records.

## 16. Hudi vs Delta Lake vs Iceberg

| Feature | Hudi | Delta Lake | Iceberg |
|---|---|---|---|
| Main focus | Incremental processing and upserts | Reliable lakehouse tables | Analytical tables and interoperability |
| Metadata | Timeline and file groups | Transaction log | Metadata files and snapshots |
| Common data format | Parquet | Parquet | Parquet |
| Upserts | Core capability | Supported through MERGE | Supported through engines |
| Incremental processing | Strong incremental query capabilities | Change Data Feed and streaming | Incremental scan capabilities |
| Table types | COW and MOR | No equivalent COW/MOR table-type model | No equivalent COW/MOR table-type model |
| Compaction | Important for MOR | OPTIMIZE and file management | File rewrite and maintenance operations |
| Best-known strength | CDC and frequent updates | Databricks integration | Multi-engine interoperability |

No table format is universally best. Selection depends on workload, engine support, and operational requirements.

## 17. When Should We Use Apache Hudi?

Hudi is worth considering when:

1. Source systems frequently update existing records.
2. CDC ingestion is required.
3. Incremental downstream processing is important.
4. Large datasets need upsert and delete operations.
5. The organization needs Hudi's indexing and table-management capabilities.
6. The processing ecosystem supports the required Hudi features.

### Example Use Cases

| Use Case | Why Hudi Helps |
|---|---|
| E-commerce orders | Handles order updates |
| Customer master data | Supports changing customer records |
| Banking transactions | Supports incremental data processing |
| IoT data | Handles continuously arriving data |
| CDC pipelines | Supports inserts, updates, and deletes |
| Event processing | Supports incremental downstream consumption |

## 18. Common Interview Mistakes

| Mistake | Correct Understanding |
|---|---|
| Hudi is only a file format | It is an open table format |
| Hudi is only for streaming | It supports batch and streaming |
| COW stores updates in log files | COW rewrites affected base files |
| MOR never rewrites base files | Compaction eventually creates updated base files |
| Compaction and cleaning are identical | Compaction merges logs; cleaning removes eligible old files |
| Snapshot and read-optimized queries always return identical results | MOR tables may expose different results before compaction |
| Hudi automatically guarantees correct CDC ordering | Event ordering and deduplication need proper pipeline design |
| Incremental queries always return a complete CDC event history | Query semantics depend on the selected mode and configuration |
| Record key and precombine field are the same | Record key identifies a record; precombine resolves competing records |
| Hudi supports only Parquet | Parquet is common, but supported storage formats depend on Hudi capabilities and configuration |

## 19. Interview Questions and Answers

**Q1. What is Apache Hudi?**

Apache Hudi is an open-source table format for data lakes. It supports upserts, deletes, incremental processing, and reliable table management.

**Q2. What does Hudi stand for?**

Hadoop Upserts Deletes and Incrementals.

**Q3. Why is Hudi useful in data engineering?**

It is useful for pipelines that frequently receive inserts, updates, and deletes, particularly CDC and incremental-processing workloads.

**Q4. What are the two main Hudi table types?**

Copy-on-Write and Merge-on-Read.

**Q5. What is Copy-on-Write?**

It rewrites affected base files when records are updated.

**Q6. What is Merge-on-Read?**

It can store updates in log files and merge them with base files during reads or compaction.

**Q7. What is the Hudi timeline?**

It tracks table actions such as commits, compaction, cleaning, and rollback.

**Q8. What is a file group?**

A file group is a logical group of files associated with a file ID and partition.

**Q9. What is a file slice?**

A file slice represents a particular version of a file group, including its base file and associated log files.

**Q10. What is a record key?**

A record key uniquely identifies a record within the Hudi table's record-key and partitioning model.

**Q11. What is a precombine field?**

It helps determine which incoming record should take precedence when multiple records share the same record key.

**Q12. What is compaction?**

Compaction merges log-file changes into base files in Merge-on-Read tables.

**Q13. What is cleaning?**

Cleaning removes eligible obsolete file versions according to the configured retention policy.

**Q14. What is the difference between compaction and cleaning?**

Compaction merges log records into base files. Cleaning removes older file versions that are no longer required.

**Q15. What is an incremental query?**

It retrieves changes after a specified point in the Hudi timeline, according to the selected query mode.

**Q16. What is the difference between snapshot and read-optimized queries?**

Snapshot queries include the latest committed state, merging base and log data where required. Read-optimized queries read base files without merging recent log files.

**Q17. Why is Hudi useful for CDC?**

It supports upserts, deletes, and incremental queries, making it suitable for processing changing source records.

**Q18. Does Hudi support batch processing?**

Yes. Hudi supports both batch and streaming data-processing patterns.

**Q19. Is Hudi always better than Delta Lake?**

No. The choice depends on the workload, processing engine, platform integration, and operational requirements.

**Q20. When would you choose Hudi over Delta Lake?**

I would evaluate Hudi when frequent upserts, CDC, and incremental processing are central requirements and the existing platform supports the required Hudi capabilities.

## 20. Quick Revision

| Concept | Remember |
|---|---|
| Apache Hudi | Open table format |
| Full form | Hadoop Upserts Deletes and Incrementals |
| Main focus | Upserts and incremental processing |
| Timeline | Tracks table actions |
| File Group | Logical group of files |
| File Slice | Version of a file group |
| Base File | Main data file |
| Log File | Stores changes in MOR |
| COW | Rewrites affected base files |
| MOR | Writes changes to logs and merges later |
| Record Key | Identifies records |
| Precombine Field | Resolves competing records |
| Compaction | Merges log files into base files |
| Cleaning | Removes eligible old file versions |
| Snapshot Query | Latest committed table state |
| Read Optimized | Reads base files without merging recent logs |
| Incremental Query | Reads changes after a specified point |
| CDC | Captures source data changes |

### One-Minute Interview Answer

Apache Hudi is an open-source table format designed for data lakes, particularly for workloads involving frequent inserts, updates, deletes, and incremental processing.

Its main capabilities include upserts, CDC support, indexing, Time Travel, and incremental queries.

Hudi has two main table types: Copy-on-Write and Merge-on-Read. In COW, updates rewrite affected base files. In MOR, updates can be stored in log files and merged with base files during reads or compaction.

Hudi also maintains a timeline to track commits and other table operations. Compaction merges log files into base files, while cleaning removes eligible old file versions.

I would consider Hudi for CDC-heavy and update-intensive pipelines where its incremental-processing capabilities provide a practical advantage.

## 21. Self-Assessment

- [ ] Define Apache Hudi.
- [ ] Explain why Hudi is needed.
- [ ] Draw the basic Hudi architecture.
- [ ] Explain the Hudi timeline.
- [ ] Explain file groups and file slices.
- [ ] Differentiate COW and MOR.
- [ ] Explain upserts.
- [ ] Explain record keys and precombine fields.
- [ ] Explain Hudi indexing.
- [ ] Explain incremental processing.
- [ ] Explain CDC.
- [ ] Explain compaction and cleaning.
- [ ] Explain snapshot and read-optimized queries.
- [ ] Write a basic Hudi read and write example.
- [ ] Compare Hudi with Delta Lake and Iceberg.
- [ ] Identify suitable Hudi use cases.

**Interview readiness:** You should be able to explain Hudi's architecture, describe COW versus MOR, explain its timeline and file organization, and demonstrate how Hudi supports CDC and incremental data pipelines.