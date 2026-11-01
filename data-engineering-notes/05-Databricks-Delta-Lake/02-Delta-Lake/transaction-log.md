# Delta Lake Transaction Log (_delta_log)

## 1. Overview

The **Delta Lake Transaction Log** is the foundation of Delta Lake.

It keeps track of every committed change made to a Delta table, including:
- Data file additions.
- Data file removals.
- Table metadata changes.
- Schema information.
- Transaction and protocol information.

The transaction log allows Delta Lake to provide ACID transactions, consistent reads, concurrency control, and Time Travel.

The transaction log is stored in a directory named:

```text
_delta_log/
```

**Interview definition:**

The Delta Lake transaction log is a record of committed table changes. It maintains the information required to identify the files and metadata belonging to a particular table version.

## 2. Delta Table Structure

A Delta table typically contains Parquet data files and a transaction-log directory.

Example:

```text
customers/
│
├── _delta_log/
│   ├── 00000000000000000000.json
│   ├── 00000000000000000001.json
│   ├── 00000000000000000002.json
│   ├── 00000000000000000003.json
│   └── checkpoint files
│
├── part-00000-....parquet
├── part-00001-....parquet
└── part-00002-....parquet
```

### Main components

| Component | Purpose |
|---|---|
| `_delta_log` | Stores transaction history and metadata changes |
| JSON commit files | Record individual committed transactions |
| Checkpoint files | Store a summarized table state to speed up log reconstruction |
| Parquet files | Store the actual table data |

**Important:** Parquet stores the data, while the transaction log tracks which data files belong to each committed table version.

## 3. How the Transaction Log Works

Every successful transaction creates a new table version.

For example, consider a customers table.

### Version 0: Initial table creation

```text
customer_id | name
------------|------
1           | Rahul
2           | Amit
```

Delta Lake creates an initial transaction-log commit.

```text
00000000000000000000.json
```

This commit contains actions describing the initial table state.

### Version 1: Insert new data

```sql
INSERT INTO customers
VALUES (3, 'Priya');
```

Delta Lake writes the new data and commits the corresponding changes.

```text
00000000000000000001.json
```

The new version references the required data files.

### Version 2: Update existing data

```sql
UPDATE customers
SET name = 'Rahul Kumar'
WHERE customer_id = 1;
```

Delta Lake generally:
1. Reads the relevant existing data.
2. Writes replacement data files.
3. Records removal actions for the old files.
4. Records addition actions for the new files.
5. Commits the transaction as a new version.

```text
00000000000000000002.json
```

### Version 3: Delete a record

```sql
DELETE FROM customers
WHERE customer_id = 2;
```

Delta Lake records the logical removal of the relevant data through a new transaction.

```text
00000000000000000003.json
```

The old Parquet files are not necessarily physically deleted immediately.

### Version history

| Version | Operation | Result |
|---|---|---|
| 0 | CREATE | Initial table |
| 1 | INSERT | New record added |
| 2 | UPDATE | Existing record modified |
| 3 | DELETE | Record removed |

Each successful commit represents a new consistent table version.

## 4. What Does a JSON Commit File Contain?

A Delta transaction-log JSON file contains one or more actions.

Important actions include:

| Action | Purpose |
|---|---|
| `add` | Records a data file added to the table |
| `remove` | Records a data file removed from the current table snapshot |
| `metaData` | Stores table metadata and schema information |
| `protocol` | Specifies the reader and writer protocol requirements |
| `commitInfo` | Contains information about the transaction |
| `txn` | Tracks application transaction identifiers and versions |

Other actions may also be present depending on the table features and runtime.

### Example: Add action

A simplified example:

```json
{
  "add": {
    "path": "part-00001.parquet",
    "size": 1024,
    "dataChange": true
  }
}
```

This describes a data file being added to the table.

### Example: Remove action

```json
{
  "remove": {
    "path": "part-00000.parquet",
    "dataChange": true
  }
}
```

This describes a file being removed from the current table snapshot.

**Important:** These are simplified examples. Actual transaction-log actions contain additional fields and may include partition values, statistics, timestamps, and other metadata.

## 5. How Delta Lake Reads a Table

When a user queries a Delta table, Delta Lake does not simply read every Parquet file in the directory.

Instead, it identifies the files belonging to a consistent table snapshot.

Conceptually:

1. Locate the transaction log.
2. Identify the latest committed table version.
3. Reconstruct the table state using the relevant log commits and checkpoints.
4. Identify active data files.
5. Read the required Parquet files.

For example:

```sql
SELECT *
FROM customers;
```

The query returns data from the latest committed table snapshot.

This prevents files removed by earlier UPDATE or DELETE operations from appearing as active records in the current table.

## 6. What Is a Checkpoint?

As a Delta table receives more transactions, its transaction log grows.

Replaying every JSON commit file from the beginning can become expensive.

Delta Lake uses **checkpoints** to speed up snapshot reconstruction.

A checkpoint stores a summarized representation of the table state at a particular version.

Example:

```text
_delta_log/
│
├── 00000000000000000000.json
├── 00000000000000000001.json
├── 00000000000000000002.json
├── 00000000000000000003.json
├── 00000000000000000004.json
├── 00000000000000000005.checkpoint.parquet
├── 00000000000000000006.json
└── 00000000000000000007.json
```

Here, the checkpoint represents the table state through version 5.

To reconstruct version 7, Delta Lake can use:
- The checkpoint at version 5.
- The JSON commits for versions 6 and 7.

It does not need to replay all earlier commits from version 0.

### Why are checkpoints useful?

- Reduce transaction-log replay.
- Improve table-loading performance.
- Speed up snapshot reconstruction.
- Support efficient management of long transaction histories.

**Important:** A checkpoint is not a backup of the complete table data. It stores table-state information, not a separate copy of all Parquet records.

## 7. JSON Commit Files vs Checkpoint Files

| JSON Commit Files | Checkpoint Files |
|---|---|
| Record individual transactions | Summarize table state |
| Usually one commit per successful version | Created periodically or according to checkpoint policies |
| Contain transaction actions | Contain a snapshot representation |
| Used to apply changes | Used to accelerate reconstruction |
| Stored as JSON | Commonly stored as Parquet |

Modern Delta implementations may support different checkpoint formats and policies, so do not assume every checkpoint must use the same format.

## 8. Transaction Log and ACID Properties

The transaction log plays a central role in Delta Lake's ACID guarantees.

### A. Atomicity

A transaction is committed as a complete operation or does not become visible as a committed table version.

Readers do not see a partially committed table update.

### B. Consistency

Transactions preserve the table's defined structural and transactional rules.

Schema enforcement and supported constraints help maintain data consistency.

However, the transaction log does not automatically validate every business rule.

### C. Isolation

Concurrent operations work against consistent table snapshots.

Delta Lake uses optimistic concurrency control to detect conflicting changes.

A conflicting transaction may fail and need to be retried.

### D. Durability

Once a transaction is successfully committed, its changes are recorded in the transaction log and the underlying storage.

Durability ultimately depends on the reliability and guarantees of the configured storage system.

**Interview point:** The transaction log is a key component that enables Delta Lake to manage reliable table transactions over Parquet files.

## 9. Transaction Log and Time Travel

Time Travel depends on the transaction log.

Each committed version represents a historical table state.

Example:

```sql
SELECT *
FROM customers VERSION AS OF 1;
```

This reads the table as it existed at version 1.

To inspect previous operations:

```sql
DESCRIBE HISTORY customers;
```

The transaction log and retained data files allow Delta Lake to reconstruct historical snapshots.

**Important:** Keeping transaction-log history alone is not sufficient. The data files required by the historical version must also remain available.

## 10. Transaction Log and VACUUM

When UPDATE or DELETE operations remove files from the current table snapshot, those files may remain physically stored.

`VACUUM` removes eligible unreferenced data files after the configured retention period.

```sql
VACUUM customers;
```

### Relationship

| Transaction Log | VACUUM |
|---|---|
| Records file additions and removals | Physically removes eligible old files |
| Maintains table-version information | Cleans up unreferenced data files |
| Supports snapshot reconstruction | Can limit historical data availability |

**Important:** VACUUM does not simply delete the entire transaction log. Log cleanup and data-file cleanup are separate processes.

## 11. Optimistic Concurrency Control

Delta Lake uses optimistic concurrency control to manage concurrent writes.

The general process is:

1. **Read:** A transaction reads a consistent snapshot of the table.
2. **Write:** It prepares changes and writes new data files where required.
3. **Validate and commit:** It checks for conflicting concurrent changes and attempts to commit a new version.

If a conflict is detected, the transaction may fail.

Example:

- Job A reads table version 5.
- Job B also reads table version 5.
- Job A updates certain records and commits version 6.
- Job B attempts to commit its changes.

Delta Lake validates Job B's transaction against concurrent changes. Depending on the operations and their read/write sets, Job B may succeed or encounter a concurrency conflict.

**Interview point:** Optimistic concurrency control allows multiple operations to work concurrently while detecting conflicting changes during commit.

## 12. How to Inspect Delta Table History

### SQL

```sql
DESCRIBE HISTORY customers;
```

### PySpark

```python
from delta.tables import DeltaTable

delta_table = DeltaTable.forName(
    spark,
    "customers"
)

history_df = delta_table.history()

history_df.show(truncate=False)
```

### Read a historical version

```python
df = (
    spark.read
    .format("delta")
    .option("versionAsOf", 2)
    .table("customers")
)
```

### Inspect the transaction-log directory

For a path-based Delta table, the log is located inside its `_delta_log` directory.

```python
display(
    dbutils.fs.ls(
        "dbfs:/path/to/customers/_delta_log/"
    )
)
```

The exact path and filesystem access method depend on the Databricks environment and storage configuration.

For managed tables, use the table's actual storage location rather than assuming a fixed path.

## 13. Real-World Data Engineering Example

Suppose we have a daily customer ETL pipeline.

Every day, the pipeline:
- Reads new customer records.
- Updates existing customer information.
- Deletes records based on source-system changes.
- Writes the result into a Delta table.

The transaction log helps manage these changes.

A simplified workflow:

```text
Source Data
     |
     v
Data Validation
     |
     v
PySpark Transformations
     |
     v
MERGE / UPDATE / DELETE
     |
     v
Delta Transaction Log
     |
     v
Committed Table Version
     |
     v
Parquet Data Files
```

The transaction log enables the table to maintain consistent snapshots even as the pipeline modifies its data.

It also supports historical queries and recovery workflows.

## 14. Common Interview Questions

**Q1. What is the Delta Lake transaction log?**

It is a record of committed changes to a Delta table. It tracks data-file additions and removals, table metadata, schema, protocol information, and transaction details.

**Q2. Where is the transaction log stored?**

Inside the `_delta_log` directory of a Delta table.

**Q3. What is the purpose of JSON commit files?**

They record the actions belonging to individual committed table versions.

**Q4. What is a checkpoint in Delta Lake?**

A checkpoint is a summarized representation of a table snapshot. It reduces the amount of transaction-log history that needs to be replayed when reconstructing a table version.

**Q5. What is the difference between JSON commit files and checkpoint files?**

JSON files record individual transactions, while checkpoint files summarize the table state at a particular version.

**Q6. How does Delta Lake identify which Parquet files are active?**

It reconstructs the table snapshot using transaction-log actions and checkpoints, then identifies the data files belonging to that snapshot.

**Q7. How does the transaction log support ACID transactions?**

It provides the committed table-state information required for atomic commits, consistent snapshots, and concurrency validation.

**Q8. How does the transaction log support Time Travel?**

It records the history of table versions and the file changes needed to reconstruct historical snapshots, provided the required log and data files are retained.

**Q9. What happens during an UPDATE operation?**

Delta Lake generally writes replacement data files and commits actions that remove the old files from the active snapshot and add the new files.

**Q10. Does DELETE immediately remove Parquet files?**

No. It logically removes records from the current snapshot. Physical deletion of eligible unreferenced files happens later through VACUUM.

**Q11. What is optimistic concurrency control?**

It is a concurrency mechanism where transactions work on snapshots and validate conflicts before committing their changes.

**Q12. Is a checkpoint a backup of the table?**

No. A checkpoint summarizes table-state information to accelerate snapshot reconstruction. It is not a separate backup of all table records.

**Q13. What is the relationship between the transaction log and VACUUM?**

The transaction log records file removals. VACUUM physically deletes eligible unreferenced data files after the retention period.

**Q14. Can we read a historical version if its required data files have been deleted?**

Generally, no. The transaction log may describe the old version, but the actual data files are required to read its records.

## 15. Quick Revision

| Concept | Remember |
|---|---|
| Transaction log | Tracks committed table changes |
| `_delta_log` | Transaction-log directory |
| JSON commit | Records an individual transaction |
| `add` | Adds a file to the table snapshot |
| `remove` | Removes a file from the table snapshot |
| `metaData` | Stores table metadata and schema |
| `protocol` | Defines reader/writer requirements |
| `commitInfo` | Stores transaction information |
| Checkpoint | Summarizes table state |
| Parquet | Stores actual table data |
| ACID | Reliable transactional behavior |
| Time Travel | Reads historical snapshots |
| VACUUM | Physically removes eligible old files |
| Concurrency control | Detects conflicting transactions |

## 16. Self-Assessment

- [ ] Explain the Delta Lake transaction log.
- [ ] Explain the purpose of `_delta_log`.
- [ ] Differentiate JSON commit files and checkpoints.
- [ ] Explain `add`, `remove`, `metaData`, and `protocol` actions.
- [ ] Explain how Delta Lake identifies active Parquet files.
- [ ] Describe how a table version is created.
- [ ] Explain how the transaction log supports ACID.
- [ ] Explain how checkpoints improve performance.
- [ ] Explain the relationship between Time Travel and the transaction log.
- [ ] Explain how VACUUM affects historical data.
- [ ] Explain optimistic concurrency control.

**Interview takeaway:** The transaction log is the foundation of Delta Lake. Parquet files store the actual data, while `_delta_log` tracks committed changes and identifies the files belonging to each table version. Checkpoints speed up snapshot reconstruction, and together these mechanisms support ACID transactions, Time Travel, and reliable data processing.