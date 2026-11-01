# ACID Transactions in Delta Lake

## 1. What Are ACID Transactions?

ACID is a set of four properties that ensure database transactions are reliable and consistent.

ACID stands for:

- **A — Atomicity**
- **C — Consistency**
- **I — Isolation**
- **D — Durability**

These properties help prevent incomplete, conflicting, or lost data changes.

In data engineering, ACID transactions are important when multiple users or jobs read and modify the same data.

**Example:**

Suppose an ETL pipeline updates a sales table. If the pipeline fails halfway through the update, we do not want users to see partially updated data.

ACID transactions help maintain reliable table operations.

## 2. Why Are ACID Transactions Important?

Consider a traditional data processing workflow:

1. Read existing data.
2. Transform records.
3. Update the target table.
4. Write the results.

Suppose the job fails while writing data.

Without proper transaction management, users might see incomplete or inconsistent results.

ACID transactions help ensure that table changes are committed safely.

### Benefits

- Prevent partial transactions.
- Maintain data consistency.
- Manage concurrent operations.
- Protect committed data.
- Support reliable data pipelines.

## 3. Four Properties of ACID

### 3.1 Atomicity

Atomicity means a transaction is completed entirely or not applied at all.

It follows the principle:

**All or Nothing.**

Example:

Suppose we want to update 1,000 records in a Delta table.

If the transaction fails before committing, the incomplete changes are not exposed as a successful table update.

```text
Transaction
    |
    v
Update 1,000 Records
    |
    v
Commit Successful?
    |
    +---- Yes ----> All Changes Committed
    |
    +---- No -----> Transaction Aborted
```

**Interview point:** Atomicity prevents users from seeing partially committed table changes.

### 3.2 Consistency

Consistency means a successful transaction preserves the defined rules and constraints of the data system.

Example:

Suppose a sales table has the following business rule:

```text
order_id must be unique
amount must be greater than or equal to 0
```

A transaction should not leave the table in a state that violates the applicable data rules.

In practice, Delta Lake supports certain constraints, such as `NOT NULL` and `CHECK`, while other business rules may need to be enforced through pipeline validation or application logic.

Example:

```sql
CREATE TABLE sales (
    order_id INT NOT NULL,
    amount DOUBLE
)
USING DELTA;
```

Add a check constraint:

```sql
ALTER TABLE sales
ADD CONSTRAINT valid_amount
CHECK (amount >= 0);
```

If a write violates an enforced constraint, the operation fails.

**Interview point:** ACID consistency does not automatically enforce every business rule. The required constraints and validations must be defined.

### 3.3 Isolation

Isolation means concurrent transactions should not interfere in a way that exposes invalid intermediate states.

Example:

Two jobs are updating the same Delta table.

- Job A updates customer records.
- Job B inserts new customer records.

Delta Lake uses optimistic concurrency control to manage concurrent writes.

Transactions check for conflicts before committing. Conflicting operations may fail and require retry or reconciliation.

**Interview point:** Isolation helps concurrent readers and writers work safely without exposing uncommitted changes.

### 3.4 Durability

Durability means that once a transaction is successfully committed, its changes are persisted and remain available after a failure, subject to the storage system's durability guarantees.

Example:

A pipeline successfully commits a Delta table update.

Even if the compute cluster stops afterward, the committed data remains stored in the underlying storage.

**Interview point:** Durability is supported by storing committed data and transaction-log information in reliable storage.

## 4. How Delta Lake Supports ACID Transactions

Delta Lake provides ACID transactions on top of data files stored in cloud or distributed storage.

It uses a transaction log called the **Delta transaction log**.

The transaction log is commonly stored in the `_delta_log` directory.

Example table structure:

```text
sales_delta/
│
├── part-00000.parquet
├── part-00001.parquet
├── part-00002.parquet
│
└── _delta_log/
    ├── 00000000000000000000.json
    ├── 00000000000000000001.json
    └── 00000000000000000002.json
```

The Parquet files contain the actual table data.

The `_delta_log` directory records the table's transaction history and metadata needed to determine the current table state.

**Remember:**

- Parquet files → Store table data.
- Delta transaction log → Tracks table changes and committed versions.

## 5. What Is the Delta Transaction Log?

The Delta transaction log is the foundation of Delta Lake transaction management.

It records information about table changes.

It can contain details such as:

- Added data files.
- Removed data files.
- Table metadata.
- Schema information.
- Protocol information.
- Transaction-related information.

Each successful transaction creates a new table version.

Example:

```text
Version 0
    |
    v
Initial Table Creation
    |
    v
Version 1
    |
    v
Insert New Records
    |
    v
Version 2
    |
    v
Update Existing Records
    |
    v
Version 3
```

The transaction log allows Delta Lake to reconstruct the table state for a particular committed version.

**Important:** A transaction does not necessarily modify an existing Parquet file in place. Delta Lake commonly writes new files and records file additions and removals in the transaction log.

## 6. How a Delta Lake Transaction Works

A simplified transaction workflow:

```text
        Start Transaction
               |
               v
        Read Current Table
               |
               v
        Perform Data Changes
               |
               v
        Write New Data Files
               |
               v
        Validate Conflicts
               |
               v
        Commit Transaction Log
               |
               v
        New Table Version
```

The important step is the commit.

The transaction becomes visible to readers when its commit is successfully recorded.

If a transaction fails before committing, its uncommitted changes do not become part of the table's committed state. Unreferenced files may require later cleanup.

## 7. Optimistic Concurrency Control

Delta Lake uses optimistic concurrency control for managing concurrent transactions.

It assumes that conflicting operations are relatively uncommon.

Instead of locking the entire table for every transaction, it allows transactions to perform work and checks for conflicts during commit.

### Three Main Steps

**Step 1: Read**

The transaction reads a particular version of the table.

**Step 2: Write**

The transaction performs changes and writes new data files.

**Step 3: Validate and Commit**

Before committing, Delta Lake checks whether concurrent changes conflict with the transaction.

```text
             Transaction A
                   |
             Read Version 5
                   |
                Write Data
                   |
                   v
             Validate Commit
                   |
                   v
                Commit 6


             Transaction B
                   |
             Read Version 5
                   |
                Write Data
                   |
                   v
             Validate Commit
                   |
             Conflict Detected?
                   |
              +----+----+
              |         |
             Yes        No
              |         |
           Retry or    Commit
           Handle
           Conflict
```

This is a conceptual example. The actual conflict outcome depends on the operations, files read and modified, isolation settings, and Delta Lake implementation.

### Why Is It Called Optimistic?

Because transactions are allowed to proceed without taking a full-table write lock in advance.

Conflict detection happens during the commit process.

**Interview point:** Optimistic concurrency control allows concurrent operations while protecting table consistency.

## 8. Concurrent Read and Write Operations

Delta Lake supports concurrent readers and writers.

Example:

- Job A reads the sales table.
- Job B inserts new sales records.
- Job A continues reading its original snapshot.
- Job B commits a new table version.

Job A can continue using the consistent snapshot it started with, while new readers can see the newly committed version.

This is based on Delta Lake's snapshot-based table access.

**Remember:** A reader does not normally see partially committed changes from another transaction.

## 9. ACID Transactions Example Using PySpark

Create a Delta table:

```python
data = [
    (1, "Amit", 5000),
    (2, "Rahul", 7000)
]

columns = ["id", "name", "salary"]

df = spark.createDataFrame(data, columns)

df.write.format("delta").mode("overwrite").save(
    "/mnt/delta/employees"
)
```

Read the table:

```python
df = spark.read.format("delta").load(
    "/mnt/delta/employees"
)

display(df)
```

Insert new records:

```python
new_data = [
    (3, "Priya", 8000)
]

new_df = spark.createDataFrame(new_data, columns)

new_df.write.format("delta").mode("append").save(
    "/mnt/delta/employees"
)
```

The successful append creates a new committed table version.

## 10. Delta Lake MERGE Operation

`MERGE` is commonly used for inserting, updating, and deleting records based on matching conditions.

It is especially useful in ETL pipelines and CDC workloads.

Example:

```sql
MERGE INTO employees AS target
USING employee_updates AS source
ON target.id = source.id

WHEN MATCHED THEN
    UPDATE SET
        target.name = source.name,
        target.salary = source.salary

WHEN NOT MATCHED THEN
    INSERT (id, name, salary)
    VALUES (source.id, source.name, source.salary);
```

### How It Works

- If the employee ID exists, update the record.
- If the employee ID does not exist, insert a new record.

The operation is committed as a Delta transaction.

**Important:** The source data should be prepared so that it does not contain multiple matching source rows for the same target row when that would make the update ambiguous.

## 11. Delta Lake Time Travel

Delta Lake maintains table versions through its transaction log.

Time Travel allows users to query an earlier committed version of a table, subject to available history and retained data files.

### Query by Version

```sql
SELECT *
FROM employees VERSION AS OF 2;
```

### Query by Timestamp

```sql
SELECT *
FROM employees TIMESTAMP AS OF '2026-10-30 10:00:00';
```

### View Table History

```sql
DESCRIBE HISTORY employees;
```

History provides information such as:

- Version number.
- Operation.
- Timestamp.
- User information where available.
- Operation parameters.

**Interview point:** Time Travel is possible because Delta Lake tracks table versions and file changes in the transaction log.

## 12. ACID Transactions vs. Traditional Parquet

| Feature | Parquet | Delta Lake |
|---|---|---|
| Storage format | Columnar file format | Table format built on data files |
| ACID transactions | Not provided by Parquet itself | Supported |
| Transaction log | No built-in Delta transaction log | `_delta_log` |
| Schema enforcement | Not provided as a table-level transaction feature | Supported |
| Time Travel | Not provided by Parquet itself | Supported |
| Concurrent transaction management | Not provided by Parquet itself | Supported through Delta transaction mechanisms |
| Updates and deletes | Usually require rewriting data through external processing | Supported through Delta operations |

**Remember:** Parquet is a file format. Delta Lake adds table management and transaction capabilities on top of data files.

## 13. Common Interview Questions

### Q1. What is ACID?

ACID represents Atomicity, Consistency, Isolation, and Durability. These properties help ensure reliable database transactions.

### Q2. What is the purpose of ACID transactions in Delta Lake?

They ensure that table changes are committed safely, concurrent operations are managed, and readers see consistent table versions.

### Q3. How does Delta Lake provide ACID transactions?

Delta Lake uses a transaction log to record table changes and manage committed versions. It uses transaction validation and concurrency control to manage writes.

### Q4. What is the Delta transaction log?

It is the log stored in the `_delta_log` directory that records table metadata, file additions and removals, and transaction history.

### Q5. What is optimistic concurrency control?

It is a concurrency management approach where transactions perform their work and validate conflicts before committing.

### Q6. What happens if a Delta transaction fails?

If it fails before a successful commit, the changes are not included in the committed table state. Uncommitted data files may remain temporarily until cleanup.

### Q7. Can multiple users read a Delta table simultaneously?

Yes. Delta Lake supports concurrent readers, and readers use consistent table snapshots.

### Q8. Can multiple jobs write to the same Delta table?

Yes, but conflicting writes may fail during transaction validation. The application may need to retry or handle the conflict.

### Q9. What is the difference between Parquet and Delta Lake?

Parquet is a columnar storage format. Delta Lake provides table management, ACID transactions, schema enforcement, and version history using data files and a transaction log.

### Q10. How does Time Travel work?

Delta Lake uses transaction-log versions and retained data files to reconstruct an earlier table state.

### Q11. Why is MERGE important in data engineering?

MERGE allows us to perform conditional inserts, updates, and deletes. It is commonly used for incremental loads and CDC pipelines.

### Q12. Does Delta Lake update Parquet files directly?

Typically, Delta Lake writes new data files and records the corresponding additions and removals in the transaction log rather than modifying existing Parquet files in place.

## 14. Quick Revision

| Concept | Remember |
|---|---|
| ACID | Atomicity, Consistency, Isolation, Durability |
| Atomicity | All or nothing |
| Consistency | Preserves defined data rules |
| Isolation | Manages concurrent transactions |
| Durability | Committed changes persist |
| Delta Log | Records table changes and versions |
| `_delta_log` | Transaction-log directory |
| Snapshot | Consistent view of a table version |
| Optimistic Concurrency | Validates conflicts before commit |
| MERGE | Conditional insert, update, and delete |
| Time Travel | Query earlier table versions |
| Parquet | Columnar data file format |
| Delta Lake | Table format with transaction management |

## 15. Self-Assessment

Before moving to the next topic, make sure you can explain:

- [ ] What are ACID transactions?
- [ ] Explain Atomicity, Consistency, Isolation, and Durability.
- [ ] Why are ACID transactions important in data engineering?
- [ ] How does Delta Lake support ACID?
- [ ] What is the `_delta_log` directory?
- [ ] What information does the transaction log maintain?
- [ ] What happens during a Delta transaction?
- [ ] What is optimistic concurrency control?
- [ ] How does Delta Lake handle concurrent readers and writers?
- [ ] What happens when a transaction fails?
- [ ] How does MERGE work?
- [ ] How does Time Travel work?
- [ ] What is the difference between Parquet and Delta Lake?

**Interview readiness:** You should be able to explain how Delta Lake uses its transaction log, snapshots, and optimistic concurrency control to provide reliable ACID transactions for data engineering pipelines.