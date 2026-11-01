# Apache Iceberg Fundamentals

## 1. What Is Apache Iceberg?

**Apache Iceberg** is an open table format designed for managing large analytical datasets in data lakes.

It adds table-management capabilities on top of data files such as Parquet, enabling reliable operations across large datasets.

Without a table format, managing updates, deletes, schema changes, and historical versions across many files can become difficult.

Iceberg provides capabilities such as:

- ACID transactions.
- Schema evolution.
- Partition evolution.
- Time Travel.
- Snapshot-based reads.
- Hidden partitioning.
- Concurrent writes.
- Support for multiple processing engines.

**Simple way to remember:**

Apache Iceberg = Open table format focused on reliable analytics, flexible metadata management, and multi-engine interoperability.

## 2. Why Do We Need Apache Iceberg?

Suppose an organization stores customer and order data in a data lake using Parquet files.

```text
orders/
    |
    +-- part-001.parquet
    |
    +-- part-002.parquet
    |
    +-- part-003.parquet
```

Now consider these requirements:

1. Update existing records.
2. Delete incorrect records.
3. Add new columns.
4. Change partitioning strategy.
5. Read historical data.
6. Allow multiple engines to access the same table.

Managing these operations directly with individual Parquet files is difficult.

Iceberg manages the table metadata and snapshots so that processing engines can work with a consistent table state.

### Main Benefits

| Feature | Benefit |
|---|---|
| ACID transactions | Reliable table changes |
| Schema evolution | Change table schema safely |
| Partition evolution | Change partition strategy without rewriting all existing data |
| Time Travel | Query historical snapshots |
| Hidden partitioning | Users query logical columns instead of manually managing partition paths |
| Snapshot isolation | Readers access consistent table states |
| Multi-engine support | Different engines can work with the same table |

## 3. Apache Iceberg Architecture

Iceberg separates table metadata from the actual data files.

Its metadata architecture is one of the most important interview topics.

### Simplified Architecture

```text
             ICEBERG TABLE
                   |
                   v
             Catalog Entry
                   |
                   v
            Metadata File
                   |
                   v
            Snapshot Metadata
                   |
                   v
             Manifest List
                   |
                   v
               Manifests
                   |
                   v
              Data Files
              (Parquet)
```

### Main Components

| Component | Purpose |
|---|---|
| Catalog | Tracks the current metadata location for a table |
| Metadata File | Stores table-level metadata and snapshot information |
| Snapshot | Represents a consistent table state |
| Manifest List | References the manifests belonging to a snapshot |
| Manifest | Tracks data files and their metadata |
| Data Files | Store actual records, commonly in Parquet format |

**Important:** The catalog is not necessarily a physical file. It is the mechanism used by an engine to discover and update the table's current metadata location.

## 4. Iceberg Metadata Architecture

Iceberg uses multiple metadata layers to organize table information.

### 4.1 Metadata File

The metadata file stores information such as:

- Table schema.
- Partition specifications.
- Snapshot references.
- Sort orders.
- Table properties.
- Current snapshot information.

It acts as the entry point for understanding the current table state.

### 4.2 Snapshot

A snapshot represents a consistent version of an Iceberg table.

Whenever a successful table operation creates a new snapshot, Iceberg can preserve the previous snapshot for historical access, subject to retention and expiration.

Example:

```text
       ICEBERG TABLE
             |
             v
        Snapshot 001
             |
             v
        Snapshot 002
             |
             v
        Snapshot 003
```

Each snapshot represents a particular table state.

### 4.3 Manifest List

A manifest list belongs to a snapshot.

It references the manifest files that describe the data files included in that snapshot.

### 4.4 Manifest Files

Manifest files contain metadata about individual data files.

They can include:

- File paths.
- Partition values.
- Record counts.
- File sizes.
- Column-level statistics.
- Information about whether files are added or existing.

### 4.5 Data Files

Data files contain the actual table records.

Common formats include:

- Parquet.
- ORC.
- Avro.

**Remember this hierarchy:**

```text
Metadata File
     |
     v
  Snapshot
     |
     v
 Manifest List
     |
     v
  Manifests
     |
     v
 Data Files
```

This metadata hierarchy is a major difference between Iceberg and simpler directory-based data organization.

## 5. Apache Iceberg Snapshots

A snapshot represents the state of a table at a particular point in time.

Suppose an orders table receives three successful operations:

```text
Snapshot 001
    |
    v
Snapshot 002
    |
    v
Snapshot 003
```

- Snapshot 001: Initial data.
- Snapshot 002: New orders inserted.
- Snapshot 003: Existing orders updated.

Iceberg readers use snapshot metadata to identify the correct files for a consistent table read.

### Why Are Snapshots Important?

1. Provide consistent table reads.
2. Support Time Travel.
3. Help manage concurrent operations.
4. Support incremental reads in compatible engines.
5. Preserve historical table states.

**Important:** A snapshot is a logical table state. It does not mean Iceberg copies every data file whenever a new snapshot is created. Snapshots can reference existing files as well as newly added files.

## 6. Hidden Partitioning

Hidden partitioning is one of Iceberg's most important features.

In traditional directory-based partitioning, users may need to understand the physical partition structure.

For example:

```text
orders/
    |
    +-- year=2026/
           |
           +-- month=10/
                  |
                  +-- data.parquet
```

A query may need to account for these partition columns or paths, depending on the table system.

Iceberg supports partition transforms that allow users to query the original logical columns.

Example:

```sql
SELECT *
FROM orders
WHERE order_date = DATE '2026-10-25';
```

The query does not need to reference a manually created `year` or `month` partition column.

Iceberg uses metadata and partition transforms to identify relevant files.

### Common Partition Transforms

| Transform | Purpose |
|---|---|
| Identity | Uses the original column value |
| Year | Extracts the year |
| Month | Extracts the month |
| Day | Extracts the day |
| Hour | Extracts the hour |
| Bucket | Divides values into a fixed number of buckets |
| Truncate | Groups values using a truncation rule |

### Example

Suppose the table is partitioned by month using the `order_date` column.

```sql
PARTITIONED BY (months(order_date))
```

Users can still query using `order_date`:

```sql
SELECT *
FROM orders
WHERE order_date >= DATE '2026-10-01'
  AND order_date < DATE '2026-11-01';
```

Iceberg can use the partition transform to identify relevant files.

**Interview point:** Hidden partitioning separates the logical table schema from its physical partition layout.

## 7. Partition Evolution

Partition evolution means changing a table's partitioning strategy without requiring all existing data files to be rewritten immediately.

### Traditional Problem

Suppose a table is initially partitioned by year:

```text
year(order_date)
```

Later, the organization needs monthly partitioning:

```text
month(order_date)
```

With traditional approaches, changing the physical layout may require expensive data rewriting.

### Iceberg Solution

Iceberg supports multiple partition specifications over the lifetime of a table.

```text
Existing Data
     |
     v
Partition Spec 1
(year)
     |
     v
New Data
     |
     v
Partition Spec 2
(month)
```

Older files can continue using the earlier partition specification, while new files use the updated specification.

Iceberg understands both through its metadata.

### Benefits

- Avoids mandatory immediate full-table rewrites.
- Supports changing query patterns.
- Allows gradual data-layout improvements.
- Makes partition management more flexible.

**Important:** Partition evolution changes the table's partition specification. It does not automatically reorganize all existing files into the new layout.

## 8. Schema Evolution

Schema evolution means changing the table schema over time without rebuilding the entire table.

Common operations include:

- Adding a column.
- Renaming a column.
- Dropping a column.
- Updating certain column types when supported.

### Example

Initial schema:

| customer_id | customer_name |
|---|---|
| 101 | Rahul |
| 102 | Priya |

New requirement: Add `city`.

Updated schema:

| customer_id | customer_name | city |
|---|---|---|
| 101 | Rahul | Kolkata |
| 102 | Priya | Delhi |

Iceberg tracks schema changes through table metadata.

### Why Is Schema Evolution Useful?

- Source systems change over time.
- Business requirements evolve.
- Historical data may not contain newly added columns.
- Data pipelines can evolve without rewriting every file.

**Important:** Schema evolution support and SQL syntax depend on the processing engine and Iceberg version. Renames and type changes are managed through schema metadata and field identity, not simply by assuming every file contains the new physical column.

## 9. ACID Transactions in Iceberg

ACID stands for:

- Atomicity.
- Consistency.
- Isolation.
- Durability.

Iceberg uses metadata commits to provide reliable table operations.

### Example

Suppose a pipeline updates an orders table.

A transaction should not expose a partially committed table state.

Iceberg creates the necessary new metadata and publishes the new table state through its catalog or supported commit mechanism.

Readers can continue accessing a consistent committed snapshot.

### Optimistic Concurrency

Iceberg supports concurrent table operations using optimistic concurrency mechanisms.

A writer prepares a change based on the table's current state and attempts to commit it.

If another writer changes conflicting metadata or data during that process, the commit may need to be retried or reconciled.

**Remember:** ACID transactions protect table operations. They do not automatically enforce every business rule or prevent all logical data-quality problems.

## 10. Upserts, Updates, and Deletes

Iceberg supports row-level changes through compatible processing engines.

Common operations include:

- INSERT.
- UPDATE.
- DELETE.
- MERGE.

### Example: MERGE

Suppose the target table contains:

| order_id | amount |
|---|---:|
| 101 | 500 |
| 102 | 700 |

Incoming data:

| order_id | amount |
|---|---:|
| 101 | 1000 |
| 103 | 600 |

A compatible SQL engine may execute:

```sql
MERGE INTO orders AS target
USING incoming_orders AS source
ON target.order_id = source.order_id

WHEN MATCHED THEN
  UPDATE SET
    target.amount = source.amount

WHEN NOT MATCHED THEN
  INSERT (order_id, amount)
  VALUES (source.order_id, source.amount);
```

Result:

| order_id | amount |
|---|---:|
| 101 | 1000 |
| 102 | 700 |
| 103 | 600 |

**Important:** Row-level operation support and syntax depend on the engine, catalog, and Iceberg version. Iceberg does not use one universal SQL execution engine.

## 11. Time Travel in Iceberg

Time Travel allows users to query an earlier table snapshot.

Suppose the table has three snapshots:

```text
Snapshot 001
    |
    v
Snapshot 002
    |
    v
Snapshot 003
```

If the latest data is incorrect, users may query an earlier snapshot for investigation or recovery.

### Why Is Time Travel Useful?

- Investigate data-quality issues.
- Compare historical table states.
- Support auditing.
- Analyze previous versions.
- Recover from incorrect data operations.

### Example SQL

In a compatible Spark Iceberg environment:

```sql
SELECT *
FROM orders VERSION AS OF 1234567890123456789;
```

The snapshot identifier is illustrative. Use a real snapshot ID from the table.

Timestamp-based historical queries are also supported by compatible engines.

**Important:** Time Travel depends on the required snapshot metadata and data files still being available. Snapshot expiration and orphan-file cleanup can remove resources needed for historical reads.

## 12. Incremental Reads

Incremental reads allow processing engines to retrieve changes between snapshots instead of repeatedly reading the entire table.

Example:

```text
Snapshot 001
     |
     v
Snapshot 002
     |
     v
Snapshot 003
```

If a downstream pipeline has already processed Snapshot 002, it may request changes after that snapshot using the supported incremental scan functionality.

### Benefits

1. Reduces repeated full-table scans.
2. Supports incremental data pipelines.
3. Helps process newly added or changed data.
4. Can reduce downstream processing costs.

**Important:** Incremental scan semantics depend on the engine and API. They should not automatically be treated as a complete row-level CDC event stream containing every insert, update, and delete.

## 13. Data File Management

Iceberg manages data files through metadata rather than relying only on directory listings.

Common maintenance operations include:

### 13.1 Compaction / Data File Rewrite

Small files can negatively affect query performance.

File rewrite operations combine or reorganize files to improve the physical layout.

Benefits:

- Reduces small-file overhead.
- Improves query efficiency.
- Reorganizes data files.
- Can improve data skipping.

### 13.2 Snapshot Expiration

Old snapshots can consume metadata and storage resources.

Snapshot expiration removes snapshots that are no longer needed according to the configured retention policy.

### 13.3 Orphan File Removal

Orphan files are files that exist in storage but are not referenced by the current table metadata.

Cleanup operations can remove eligible orphan files.

**Important:**

| Operation | Purpose |
|---|---|
| Data file rewrite | Reorganizes data files |
| Snapshot expiration | Removes old snapshot references |
| Orphan file cleanup | Removes unreferenced files |

Do not confuse these operations. Each manages a different part of the table lifecycle.

## 14. Catalogs in Iceberg

A catalog helps processing engines discover and manage Iceberg tables.

It maintains or resolves the current metadata location for a table and coordinates table operations.

Common catalog approaches include:

- Hive Metastore.
- Hadoop Catalog.
- REST Catalog.
- Cloud or platform-specific catalog integrations.

### Why Is a Catalog Needed?

Imagine multiple engines accessing the same Iceberg table.

```text
       ICEBERG CATALOG
              |
       +------+------+
       |      |      |
       v      v      v
     Spark  Trino  Flink
       |      |      |
       +------+------+
              |
              v
        ICEBERG TABLE
```

The catalog provides a common way to discover the table and coordinate metadata updates.

**Important:** Multi-engine access depends on compatible catalog configuration, storage access, and feature support. Merely using Iceberg does not guarantee that every engine supports every table operation.

## 15. Practical Example: Iceberg with Spark SQL

Iceberg can be used with Spark through its compatible Spark runtime integration.

The following is an illustrative Spark SQL example. It assumes that the Iceberg Spark runtime, catalog, and storage configuration are already set up.

### Step 1: Create a Table

```sql
CREATE TABLE orders (
    order_id BIGINT,
    customer_id BIGINT,
    amount DOUBLE,
    order_date DATE
)
USING iceberg
PARTITIONED BY (months(order_date));
```

### Step 2: Insert Data

```sql
INSERT INTO orders VALUES
(101, 501, 500.0, DATE '2026-10-20'),
(102, 502, 700.0, DATE '2026-10-21'),
(103, 503, 900.0, DATE '2026-10-22');
```

### Step 3: Query Data

```sql
SELECT *
FROM orders;
```

### Step 4: Update a Record

```sql
UPDATE orders
SET amount = 1000.0
WHERE order_id = 101;
```

### Step 5: Delete a Record

```sql
DELETE FROM orders
WHERE order_id = 102;
```

### Step 6: Add a Column

```sql
ALTER TABLE orders
ADD COLUMN city STRING;
```

### Step 7: Inspect Snapshots

In a compatible Spark environment:

```sql
SELECT *
FROM orders.snapshots;
```

This metadata table can provide information about table snapshots.

**Implementation notes:**

- Iceberg SQL support varies by Spark version and Iceberg runtime.
- The catalog must be configured before table creation.
- Row-level operations require compatible engine and table configuration.
- Metadata tables and their available columns can vary by Iceberg version.
- Production pipelines should validate source data and make retries safe.

## 16. Apache Iceberg vs Delta Lake vs Hudi

| Feature | Iceberg | Delta Lake | Hudi |
|---|---|---|---|
| Main focus | Large-scale analytics and interoperability | Reliable lakehouse tables | Upserts and incremental processing |
| Metadata architecture | Metadata files, manifests, snapshots | Transaction log | Timeline and file groups |
| Common data format | Parquet | Parquet | Parquet |
| ACID transactions | Supported | Supported | Supported |
| Schema evolution | Supported | Supported | Supported |
| Partition evolution | Major strength | Different partitioning and clustering capabilities | Supported capabilities depend on version and configuration |
| Upserts | Supported by compatible engines | MERGE | Core capability |
| Incremental processing | Incremental scans | Change Data Feed and streaming | Incremental queries and CDC |
| Multi-engine interoperability | Major strength | Depends on Delta protocol and engine support | Depends on Hudi engine integration |

### When to Choose Iceberg?

Consider Iceberg when:

1. Multiple processing engines need access to the same tables.
2. Flexible partition evolution is important.
3. Large analytical datasets need reliable metadata management.
4. Snapshot-based reads and historical access are required.
5. The organization wants an open table format with broad engine interoperability.

No table format is universally best. Evaluate engine compatibility, workload, operational maturity, and feature support.

## 17. Common Interview Mistakes

| Mistake | Correct Understanding |
|---|---|
| Iceberg is a file format | It is an open table format |
| Iceberg stores all table data in metadata | Metadata references data files; records are stored in data files |
| Snapshot means a full copy of the table | Snapshots can reference existing files |
| Manifest and manifest list are identical | A manifest list references manifests; manifests describe data files |
| Hidden partitioning means no physical partitioning | Physical partitioning exists, but users query logical columns |
| Partition evolution rewrites every old file | Existing files can retain older partition specifications |
| Iceberg supports only Parquet | Parquet is common; ORC and Avro are also supported |
| Iceberg is only for Spark | Multiple engines integrate with Iceberg |
| Snapshot expiration and orphan cleanup are the same | They manage different metadata and storage resources |
| Incremental scans always provide complete CDC events | Incremental scan semantics depend on the engine and API |
| Every engine supports every Iceberg feature | Feature support varies by engine and version |

## 18. Interview Questions and Answers

**Q1. What is Apache Iceberg?**

Apache Iceberg is an open table format for data lakes. It provides ACID transactions, schema evolution, partition evolution, snapshots, and Time Travel.

**Q2. Why do we need Iceberg?**

It simplifies reliable management of large analytical datasets and supports operations such as updates, deletes, schema changes, and historical reads.

**Q3. What is an Iceberg snapshot?**

A snapshot represents a consistent state of an Iceberg table at a particular point in time.

**Q4. What is a manifest file?**

A manifest file stores metadata about data files, including paths, partition information, and statistics.

**Q5. What is a manifest list?**

A manifest list references the manifest files associated with a snapshot.

**Q6. What is hidden partitioning?**

It allows users to query logical columns while Iceberg manages the physical partition transforms through metadata.

**Q7. What is partition evolution?**

It allows a table's partitioning strategy to change over time without immediately rewriting all existing data files.

**Q8. What is schema evolution?**

It allows supported schema changes, such as adding, renaming, or dropping columns, through table metadata.

**Q9. What is the role of a catalog?**

A catalog helps engines discover Iceberg tables and manage their current metadata locations.

**Q10. What is Time Travel?**

Time Travel allows querying historical table snapshots, provided the required metadata and data files remain available.

**Q11. Does Iceberg support UPDATE and DELETE?**

Yes. Compatible processing engines support row-level operations such as UPDATE, DELETE, and MERGE.

**Q12. What is the difference between a snapshot and a data file?**

A snapshot represents a logical table state. A data file physically stores records.

**Q13. What is the difference between a manifest and a data file?**

A manifest stores metadata about data files. A data file stores the actual records.

**Q14. What is snapshot expiration?**

It removes old snapshot references according to a retention policy.

**Q15. What are orphan files?**

They are files in storage that are not referenced by the current table metadata.

**Q16. What is incremental reading?**

It retrieves changes between snapshots using supported incremental scan functionality.

**Q17. How is Iceberg different from Delta Lake?**

Iceberg uses metadata files, manifests, and snapshots. Delta Lake uses a transaction log to manage table versions and active data files.

**Q18. How is Iceberg different from Hudi?**

Iceberg is particularly known for snapshot-based metadata management, partition evolution, and interoperability. Hudi is particularly known for upserts, CDC, and incremental processing.

**Q19. Can multiple engines access the same Iceberg table?**

Yes, provided they support the required Iceberg version, catalog, storage configuration, and table features.

**Q20. When would you choose Iceberg?**

I would consider Iceberg when multi-engine interoperability, flexible partition evolution, and reliable analytical table management are important requirements.

## 19. Quick Revision

| Concept | Remember |
|---|---|
| Apache Iceberg | Open table format |
| Main focus | Analytics and interoperability |
| Catalog | Discovers and manages table metadata location |
| Metadata File | Stores table-level metadata |
| Snapshot | Consistent table state |
| Manifest List | References manifests |
| Manifest | Describes data files |
| Data File | Stores actual records |
| Hidden Partitioning | Query logical columns |
| Partition Evolution | Change partition strategy over time |
| Schema Evolution | Change table schema |
| ACID | Reliable table transactions |
| Time Travel | Read historical snapshots |
| Incremental Read | Read changes between snapshots |
| Snapshot Expiration | Removes old snapshot references |
| Orphan Cleanup | Removes unreferenced files |
| Data File Rewrite | Reorganizes data files |

### One-Minute Interview Answer

Apache Iceberg is an open table format designed for large analytical datasets in data lakes.

It provides features such as ACID transactions, schema evolution, partition evolution, Time Travel, and multi-engine interoperability.

Its architecture uses metadata files, snapshots, manifest lists, and manifest files to track the data files belonging to a table.

One important feature is hidden partitioning, which allows users to query logical columns without manually managing physical partition paths. Iceberg also supports partition evolution, so the partitioning strategy can change without immediately rewriting all existing data.

I would consider Iceberg when an organization needs reliable analytical tables, flexible partition management, and access from multiple processing engines.

## 20. Self-Assessment

- [ ] Define Apache Iceberg.
- [ ] Explain why Iceberg is needed.
- [ ] Draw its metadata architecture.
- [ ] Explain the role of a catalog.
- [ ] Explain metadata files and snapshots.
- [ ] Differentiate manifest lists and manifest files.
- [ ] Explain hidden partitioning.
- [ ] Explain partition transforms.
- [ ] Explain partition evolution.
- [ ] Explain schema evolution.
- [ ] Explain ACID transactions.
- [ ] Explain upserts, updates, and deletes.
- [ ] Explain Time Travel.
- [ ] Explain incremental reads.
- [ ] Differentiate snapshot expiration and orphan cleanup.
- [ ] Write a basic Iceberg Spark SQL example.
- [ ] Compare Iceberg with Delta Lake and Hudi.
- [ ] Identify suitable Iceberg use cases.

**Interview readiness:** You should be able to explain Iceberg's metadata hierarchy, snapshots, manifests, hidden partitioning, partition evolution, and why it is useful in multi-engine data lake environments.