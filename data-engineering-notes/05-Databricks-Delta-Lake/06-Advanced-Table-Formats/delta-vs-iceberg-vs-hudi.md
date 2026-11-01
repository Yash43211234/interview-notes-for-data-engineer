# Delta Lake vs Apache Iceberg vs Apache Hudi

## 1. Introduction

Delta Lake, Apache Iceberg, and Apache Hudi are open table formats used in modern data lake and lakehouse architectures.

They bring database-like capabilities to data stored in cloud object storage or distributed file systems.

Traditional data lakes store files such as:

- Parquet
- ORC
- Avro
- CSV
- JSON

However, storing files alone does not provide complete table-management capabilities.

For example, a traditional collection of Parquet files does not automatically provide:

- ACID transactions.
- Reliable concurrent writes.
- Consistent table snapshots.
- Schema evolution.
- Efficient updates and deletes.
- Time Travel.
- Metadata-based query optimization.

Delta Lake, Iceberg, and Hudi address these challenges through metadata and transaction-management mechanisms.

**Simple way to remember:**

- Delta Lake = Open table format with transaction-log-based architecture.
- Apache Iceberg = Open table format with snapshot and metadata-file architecture.
- Apache Hudi = Open table format with a strong focus on incremental data processing and upserts.

## 2. What Is Delta Lake?

**Delta Lake** is an open-source storage framework that adds ACID transactions, schema enforcement, and reliable table management to data lakes.

It uses Parquet files for data storage and a transaction log to track table changes.

### Architecture

```text
          DELTA TABLE
               |
       +-------+-------+
       |               |
       v               v
   Parquet Files    _delta_log
   (Data Storage)   (Transaction Log)
```

### Main Features

1. ACID transactions.
2. Schema enforcement.
3. Schema evolution.
4. Time Travel.
5. UPDATE, DELETE, and MERGE.
6. Concurrent transaction handling.
7. Batch and streaming integration.
8. OPTIMIZE and file compaction.

### Transaction Log

Delta Lake maintains a transaction log inside:

```text
_delta_log/
```

It contains commit information and metadata actions used to reconstruct table snapshots.

Example:

```text
orders/
    |
    +-- part-00001.parquet
    +-- part-00002.parquet
    |
    +-- _delta_log/
          |
          +-- 00000000000000000000.json
          +-- 00000000000000000001.json
          +-- 00000000000000000002.json
```

Each successful transaction creates a new table version.

The log can also contain checkpoints that summarize table state and help avoid replaying every historical commit.

### Example

Create a Delta table:

```python
df.write \
    .format("delta") \
    .mode("overwrite") \
    .save("/mnt/delta/orders")
```

Read it:

```python
df = spark.read \
    .format("delta") \
    .load("/mnt/delta/orders")
```

Update records:

```sql
UPDATE orders
SET amount = 1000
WHERE order_id = 101;
```

**Key point:** Delta Lake is especially well integrated with Databricks and Apache Spark.

## 3. What Is Apache Iceberg?

**Apache Iceberg** is an open table format designed to manage large analytical datasets reliably across distributed processing engines.

It provides table-level metadata management, snapshot isolation, schema evolution, and partition evolution.

Iceberg is designed to support interoperability across multiple query engines.

### Architecture

```text
             ICEBERG TABLE
                   |
                   v
             Metadata Layer
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
   Metadata     Manifest    Manifest
    Files        Lists       Files
       |           |           |
       +-----------+-----------+
                   |
                   v
              Data Files
               (Parquet)
```

### Important Metadata Components

| Component | Purpose |
|---|---|
| Catalog | Identifies and manages table metadata |
| Metadata File | Stores table-level metadata and snapshot information |
| Manifest List | References manifests belonging to a snapshot |
| Manifest File | Tracks data files and their metadata |
| Data Files | Store actual records, commonly in Parquet |

### Main Features

1. ACID transactions.
2. Snapshot isolation.
3. Schema evolution.
4. Partition evolution.
5. Time Travel through snapshots.
6. Hidden partitioning.
7. Metadata-based file pruning.
8. Support across multiple processing engines.

### What Is Hidden Partitioning?

In traditional partitioned tables, users often need to understand physical partition columns.

Iceberg supports hidden partitioning, where queries can filter using logical columns and Iceberg handles partition transformation and pruning.

For example:

```sql
SELECT *
FROM orders
WHERE order_date = DATE '2026-10-25';
```

The query does not necessarily need to reference a separate physical partition column such as `year` or `month`.

Iceberg uses its partition metadata and transforms to identify relevant data files.

### What Is Partition Evolution?

Iceberg allows partition specifications to evolve without requiring users to manually rewrite all existing data into a completely new partition layout.

For example, a table may initially partition by year and later introduce a month-based partition specification.

Old and new data can coexist under different partition specifications, with metadata tracking how the files are organized.

**Key point:** Iceberg is particularly useful when organizations want an open table format with strong multi-engine interoperability and flexible metadata management.

## 4. What Is Apache Hudi?

**Apache Hudi** stands for Hadoop Upserts Deletes and Incrementals.

It is an open data lake table format designed for managing large datasets with frequent inserts, updates, and deletes.

Hudi is especially useful for incremental data pipelines and change-data-capture (CDC) workloads.

### Architecture

```text
             HUDI TABLE
                  |
         +--------+--------+
         |                 |
         v                 v
    Data Files        Metadata
    (Base Files)      and Timeline
         |
         v
   Log Files
 (For MOR Tables)
```

### Main Features

1. ACID-style table operations with supported storage and concurrency configurations.
2. Upserts and deletes.
3. Incremental queries.
4. Change Data Capture support.
5. Record-level indexing.
6. Data management and compaction.
7. Streaming and batch integration.

### Hudi Table Types

Hudi mainly supports two table types:

**1. Copy-on-Write (COW)**

In Copy-on-Write tables, updates rewrite affected base files.

```text
Before Update:

Base File
    |
    v
Old Records

After Update:

New Base File
    |
    v
Updated Records
```

Characteristics:

- Updates rewrite affected data files.
- Read performance is generally simpler.
- Write operations can be more expensive for update-heavy workloads.

**2. Merge-on-Read (MOR)**

In Merge-on-Read tables, updates can be stored in log files and merged with base files during reads or compaction.

```text
             MOR TABLE
                 |
         +-------+-------+
         |               |
         v               v
      Base File       Log Files
         |               |
         +-------+-------+
                 |
                 v
          Merged Results
```

Characteristics:

- Supports efficient write patterns for frequent updates.
- Reads may need to merge base and log data.
- Compaction is important for managing accumulated log files.

### Key Point

Hudi is particularly useful when the pipeline frequently receives changes to existing records and needs incremental processing.

## 5. Comparison: Delta Lake vs Iceberg vs Hudi

| Feature | Delta Lake | Apache Iceberg | Apache Hudi |
|---|---|---|---|
| Main focus | Reliable lakehouse tables | Large analytical tables and interoperability | Incremental processing and upserts |
| Metadata architecture | Transaction log | Metadata files, manifests, snapshots | Timeline, metadata, file groups |
| Data file format | Commonly Parquet | Commonly Parquet, also supports other formats | Commonly Parquet with log files for MOR |
| ACID transactions | Yes | Yes | Yes, with supported configurations |
| Schema evolution | Yes | Yes | Yes |
| Time Travel | Yes | Yes, through snapshots | Yes, through timeline and supported query capabilities |
| UPDATE and DELETE | Yes | Yes | Yes |
| Upserts | MERGE and related operations | Supported through engine-specific operations | A major design focus |
| Partition evolution | Supported through newer capabilities such as liquid clustering in Databricks; not equivalent to Iceberg partition evolution | Native partition evolution | Supports table partitioning and supported evolution mechanisms |
| Incremental processing | Supported through Change Data Feed and streaming capabilities | Supports incremental scan capabilities | Strong incremental query and CDC capabilities |
| Multi-engine support | Supported by multiple engines; capabilities vary | Broad multi-engine ecosystem | Supported by multiple engines; capabilities vary |
| Databricks integration | Native and deeply integrated | Supported in Databricks with feature and runtime considerations | Support depends on integration and runtime |

**Important:** Features and SQL syntax vary by engine, runtime, and table-format version. A capability supported by one engine may not be available in another.

## 6. Understanding Their Metadata Architectures

One of the most important interview differences is how each format manages table metadata and changes.

### Delta Lake: Transaction Log

Delta Lake uses:

```text
_delta_log/
```

The transaction log records changes such as:

- Adding data files.
- Removing data files.
- Metadata changes.
- Protocol changes.
- Transaction information.

A reader uses the log to determine the active files for a consistent table snapshot.

### Iceberg: Metadata Files and Snapshots

Iceberg maintains metadata files that describe:

- Table schema.
- Partition specifications.
- Snapshots.
- Manifest lists.
- Manifest files.
- Data-file information.

A snapshot identifies a consistent state of the table.

Readers use the metadata hierarchy to determine which data files belong to the selected snapshot.

### Hudi: Timeline and File Groups

Hudi uses a timeline to track table actions and commits.

It also organizes data into file groups.

Depending on the table type:

- Copy-on-Write primarily uses base files.
- Merge-on-Read uses base files and log files.

The timeline helps readers understand committed table changes and supported incremental queries.

### Quick Comparison

| Format | Main Metadata Mechanism |
|---|---|
| Delta Lake | Transaction log |
| Iceberg | Metadata files, manifests, and snapshots |
| Hudi | Timeline, file groups, and metadata |

## 7. ACID Transactions

All three formats provide transaction-management capabilities for supported operations and configurations.

ACID stands for:

- Atomicity
- Consistency
- Isolation
- Durability

### Example

Suppose a pipeline updates an order:

```text
order_id = 101
amount = 500
```

The pipeline changes the amount to:

```text
order_id = 101
amount = 1000
```

A reliable table format ensures readers see a committed table state rather than a partially applied transaction.

### Important Differences

- Delta Lake uses optimistic concurrency control with its transaction-log architecture.
- Iceberg uses snapshot-based table metadata and atomic metadata commits.
- Hudi uses its timeline and transaction mechanisms to manage supported writes and table changes.

Exact concurrency behavior depends on storage systems, engines, isolation settings, and table-format implementations.

## 8. Schema Evolution

Schema evolution means changing a table's schema as business requirements change.

Example:

Initial schema:

```text
order_id
customer_id
amount
```

New schema:

```text
order_id
customer_id
amount
payment_method
```

All three formats support schema evolution, but their implementation details and supported operations differ.

| Format | Schema Evolution |
|---|---|
| Delta Lake | Supports schema changes and controlled schema evolution |
| Iceberg | Supports metadata-based schema evolution, including field tracking |
| Hudi | Supports schema evolution with configuration and compatibility considerations |

**Interview point:** Schema evolution allows table structures to change without requiring every historical data file to be rewritten immediately.

## 9. Time Travel and Snapshots

Time Travel allows users to access an earlier table state.

### Delta Lake

Delta Lake uses transaction-log versions.

Example:

```sql
SELECT *
FROM orders VERSION AS OF 5;
```

This reads a historical version if the required transaction-log information and data files are still available.

### Iceberg

Iceberg uses snapshots.

A query engine can select a historical snapshot using its supported time-travel syntax or API.

Conceptually:

```text
Current Snapshot: 105
Previous Snapshot: 104
Older Snapshot: 103
```

### Hudi

Hudi uses its timeline to track commits and supports historical queries through its supported query modes and APIs.

**Important:** Historical reads depend on retention, cleaning, and the availability of required data files and metadata.

## 10. Upserts and Incremental Processing

An upsert means:

- Update a record if it already exists.
- Insert a record if it does not exist.

Example:

```text
Existing Table:

order_id | amount
-----------------
101      | 500
102      | 700

Incoming Data:

order_id | amount
-----------------
101      | 900
103      | 600
```

After an upsert:

```text
order_id | amount
-----------------
101      | 900
102      | 700
103      | 600
```

### Delta Lake

Delta Lake supports upserts using `MERGE`.

```sql
MERGE INTO target_orders AS target
USING source_orders AS source
ON target.order_id = source.order_id

WHEN MATCHED THEN
    UPDATE SET *

WHEN NOT MATCHED THEN
    INSERT *;
```

### Iceberg

Iceberg supports row-level updates, deletes, and upserts through supported query engines and implementations.

The actual SQL syntax and execution behavior depend on the engine.

### Hudi

Upserts are a central Hudi capability.

Hudi uses record keys and indexing mechanisms to identify records that need to be inserted or updated.

**Interview point:** Hudi is particularly associated with incremental ingestion and CDC workloads, but Delta Lake and Iceberg also support upsert and incremental-processing use cases.

## 11. Copy-on-Write vs Merge-on-Read

This is an important Hudi interview topic.

| Feature | Copy-on-Write | Merge-on-Read |
|---|---|---|
| Update behavior | Rewrites affected base files | Can write updates to log files |
| Read behavior | Reads base files | May merge base and log files |
| Read performance | Generally simpler | Can be more expensive before compaction |
| Write behavior | More write amplification for updates | Can support lower-latency update ingestion |
| Compaction | Not required for merging log files | Important for merging log files into base files |
| Common use | Read-heavy workloads | Update-heavy workloads |

**Remember:**

- COW = Updates rewrite affected base files.
- MOR = Updates can be stored in log files and merged during reads or compaction.

## 12. When Should We Use Each Format?

The correct choice depends on the processing engines, workload, ecosystem, operational requirements, and platform support.

### Choose Delta Lake When:

- The organization uses Databricks heavily.
- Spark-based pipelines are common.
- ACID transactions and MERGE are required.
- Delta-specific capabilities such as Change Data Feed are useful.
- The team wants deep integration with the Databricks lakehouse platform.

### Choose Apache Iceberg When:

- Multiple query engines need to work with the same tables.
- Open table-format interoperability is important.
- Partition evolution is a major requirement.
- Large analytical datasets need snapshot-based metadata management.
- The organization has a multi-engine lakehouse architecture.

### Choose Apache Hudi When:

- Frequent upserts and updates are required.
- CDC ingestion is a major workload.
- Incremental data processing is important.
- The pipeline needs Hudi's record-level indexing and incremental query capabilities.
- The organization can operate and maintain the Hudi ecosystem.

## 13. Practical Scenario

Suppose an e-commerce company receives millions of order records every day.

Its requirements are:

1. Ingest new orders.
2. Update existing orders.
3. Handle duplicate records.
4. Support historical queries.
5. Provide analytics through multiple engines.

### Possible Design

```text
           SOURCE SYSTEM
                 |
                 v
           DATA INGESTION
                 |
                 v
            DATA LAKE
                 |
       +---------+---------+
       |         |         |
       v         v         v
     DELTA     ICEBERG    HUDI
       |         |         |
       +---------+---------+
                 |
                 v
         ANALYTICS LAYER
                 |
                 v
          BI / REPORTING
```

This diagram represents alternative table-format choices, not a recommendation to store the same table in all three formats.

### Evaluation

| Requirement | Important Consideration |
|---|---|
| Frequent updates | Evaluate MERGE and upsert performance |
| CDC ingestion | Evaluate Hudi incremental capabilities and other supported CDC options |
| Multiple query engines | Evaluate Iceberg interoperability and engine support |
| Databricks-first environment | Evaluate Delta Lake integration |
| Historical queries | Compare snapshot and retention capabilities |
| Operational simplicity | Consider existing platform support and team expertise |

There is no universally best format. Benchmark the actual workload before choosing.

## 14. Common Interview Mistakes

| Mistake | Correct Understanding |
|---|---|
| Delta Lake, Iceberg, and Hudi are file formats like CSV | They are open table formats |
| All three store data in exactly the same way | Their metadata and storage architectures differ |
| Delta Lake is just Parquet | Delta uses Parquet data files plus a transaction log |
| Iceberg stores all table metadata in one transaction log | It uses metadata files, manifests, and snapshots |
| Hudi is only for streaming | It supports batch and streaming workloads |
| Hudi MOR always gives faster reads | Reads may require merging log and base files |
| COW and MOR are Delta Lake table types | They are Hudi table types |
| Only Hudi supports upserts | Delta and Iceberg also support upsert operations through supported engines |
| Iceberg partition evolution requires rewriting all historical data immediately | Iceberg tracks different partition specifications through metadata |
| All engines support every table-format feature | Feature support varies by engine and version |
| Time Travel works forever | Retention and cleanup can remove historical data and metadata |
| One format is always better | Selection depends on workload and ecosystem |

## 15. Interview Questions and Answers

**Q1. What are Delta Lake, Iceberg, and Hudi?**

They are open table formats that add database-like capabilities such as transactions, schema evolution, and reliable table management to data lakes.

**Q2. What is the main difference between a data lake and a lakehouse table format?**

A traditional data lake stores files. A lakehouse table format adds metadata and transaction management to make those files behave more like reliable analytical tables.

**Q3. What is the main metadata difference between Delta Lake and Iceberg?**

Delta Lake uses a transaction log. Iceberg uses metadata files, manifests, and snapshots to track table state.

**Q4. What is the main metadata mechanism in Hudi?**

Hudi uses a timeline to track commits and table actions, along with file groups to organize data files.

**Q5. Which format is closely integrated with Databricks?**

Delta Lake has deep native integration with Databricks.

**Q6. What is Iceberg's hidden partitioning?**

It allows users to query using logical columns while Iceberg handles partition transformations and pruning through table metadata.

**Q7. What is partition evolution in Iceberg?**

It allows a table's partition specification to change over time without requiring all historical data files to be rewritten immediately.

**Q8. What are Hudi's two main table types?**

Copy-on-Write and Merge-on-Read.

**Q9. What is Copy-on-Write?**

Updates rewrite affected base files, making reads generally simpler but potentially increasing write costs.

**Q10. What is Merge-on-Read?**

Updates can be stored in log files and merged with base files during reads or compaction.

**Q11. Why is Hudi useful for CDC?**

Hudi provides upsert, delete, and incremental-processing capabilities that are useful for managing changes from source systems.

**Q12. Does Delta Lake support upserts?**

Yes. Delta Lake supports upserts through operations such as SQL `MERGE`.

**Q13. Does Iceberg support Time Travel?**

Yes. Iceberg supports historical snapshots, provided the required metadata and data files remain available.

**Q14. Which format should we choose for a multi-engine architecture?**

Iceberg is often a strong candidate because of its broad interoperability focus. However, the actual decision should consider supported features and performance across the required engines.

**Q15. Which format should we choose for a Databricks-first architecture?**

Delta Lake is often a natural choice because of its deep Databricks integration.

**Q16. Which format is associated with incremental data processing?**

Apache Hudi is particularly known for incremental queries, upserts, and CDC-oriented workloads. Delta Lake and Iceberg also offer incremental capabilities.

**Q17. Are these formats ACID compliant?**

All three provide ACID transaction capabilities for supported operations and configurations. Exact guarantees depend on the implementation, storage, and processing engine.

**Q18. Can we use all three formats in one organization?**

Yes. Different teams or workloads may use different formats. However, operating multiple formats increases complexity, so the decision should be based on actual requirements.

## 16. Quick Revision

| Concept | Delta Lake | Iceberg | Hudi |
|---|---|---|---|
| Main architecture | Transaction log | Metadata and snapshots | Timeline and file groups |
| Data storage | Commonly Parquet | Commonly Parquet | Commonly Parquet |
| Main strength | Databricks integration | Multi-engine interoperability | Incremental processing and upserts |
| Time Travel | Versions | Snapshots | Timeline |
| Upserts | MERGE | Engine-supported operations | Core capability |
| Partition evolution | Platform-specific capabilities | Native support | Supported mechanisms |
| Special concept | `_delta_log` | Hidden partitioning | COW and MOR |

### One-Minute Interview Answer

Delta Lake, Apache Iceberg, and Apache Hudi are open table formats that bring reliable table management and transaction capabilities to data lakes.

Delta Lake uses Parquet data files and a transaction log. It has deep integration with Databricks and supports ACID transactions, schema evolution, MERGE, and Time Travel.

Iceberg uses metadata files, manifests, and snapshots. Its major strengths include multi-engine interoperability, hidden partitioning, and partition evolution.

Hudi uses a timeline and file groups. It is particularly useful for incremental data processing, CDC, and frequent upserts. Its two main table types are Copy-on-Write and Merge-on-Read.

The best choice depends on the workload and ecosystem. For a Databricks-first architecture, Delta Lake is a natural choice; for multi-engine interoperability, Iceberg is worth evaluating; and for CDC-heavy incremental workloads, Hudi is worth considering.

## 17. Self-Assessment

- [ ] Define an open table format.
- [ ] Explain why traditional data lakes need table formats.
- [ ] Explain Delta Lake architecture.
- [ ] Explain the purpose of `_delta_log`.
- [ ] Explain Iceberg metadata files and snapshots.
- [ ] Explain hidden partitioning.
- [ ] Explain partition evolution.
- [ ] Explain Hudi architecture and timeline.
- [ ] Differentiate COW and MOR.
- [ ] Compare the three metadata architectures.
- [ ] Compare ACID and Time Travel capabilities.
- [ ] Explain upserts and incremental processing.
- [ ] Explain CDC use cases.
- [ ] Identify suitable workloads for each format.
- [ ] Explain why engine and runtime compatibility matters.

**Interview readiness:** You should be able to draw the basic architecture of all three formats, explain their metadata differences, compare Hudi's COW and MOR table types, and justify a table-format choice based on a real data engineering workload.