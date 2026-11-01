# Delta Lake Fundamentals

## 1. What Is Delta Lake?

Delta Lake is an open-source storage framework that brings reliability and transaction capabilities to data lakes.

It is built on top of data files, commonly Parquet, and uses a transaction log to manage table changes.

Delta Lake provides important features such as:

- ACID transactions
- Schema enforcement
- Schema evolution
- Time Travel
- UPDATE, DELETE, and MERGE operations
- Transaction history
- Concurrent read and write management
- Data versioning

**Example:**

Suppose an organization stores millions of customer and sales records in cloud storage.

Using only Parquet files, managing updates, deletes, concurrent writes, and historical versions requires additional processing logic.

Delta Lake provides these capabilities through its transaction-management layer.

**Remember:** Delta Lake combines the scalability of a data lake with important reliability features traditionally associated with databases.

## 2. Why Do We Need Delta Lake?

Traditional data lakes commonly store data in file formats such as:

- CSV
- JSON
- Parquet
- Avro

These formats are useful for storing large amounts of data, but they do not independently provide full table-level transaction management.

### Problems with Traditional Data Lakes

1. Partial or failed writes can leave incomplete output files.
2. Managing concurrent writes is difficult.
3. Updating and deleting individual records requires additional logic.
4. Schema changes can create inconsistencies.
5. Tracking historical data versions requires extra implementation.
6. Maintaining reliable incremental pipelines can be challenging.

### How Delta Lake Helps

| Traditional Data Lake | Delta Lake |
|---|---|
| Stores data in files | Manages data as reliable tables |
| No built-in Delta transaction log | Uses a transaction log |
| No native ACID support from Parquet alone | Supports ACID transactions |
| Updates require additional processing | Supports UPDATE operations |
| Deletes require additional processing | Supports DELETE operations |
| No built-in table version history | Supports Time Travel |
| Schema handling needs additional controls | Supports schema enforcement and evolution |

## 3. Delta Lake Architecture

Delta Lake uses two important components:

1. Data files
2. Transaction log

### Data Files

Delta Lake commonly stores table data in Parquet format.

These files contain the actual records.

### Transaction Log

The transaction log maintains information about changes to the table.

It tracks information such as:

- Added files
- Removed files
- Table metadata
- Schema information
- Protocol information
- Transaction history

### Architecture

```text
             Delta Lake Table
                    |
          +---------+---------+
          |                   |
          v                   v
      Data Files         Transaction Log
       (Parquet)            (_delta_log)
          |                   |
          v                   v
      Actual Data        Table Versions
                         File Changes
                         Metadata
```

Example directory structure:

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

**Remember:**
- Parquet files store the data.
- `_delta_log` manages the table's committed state and history.

## 4. What Is the Delta Transaction Log?

The Delta transaction log is the foundation of Delta Lake.

It records changes made to a Delta table and allows the system to identify which data files belong to a particular committed table version.

Example:

```text
Version 0
    |
    v
Create Delta Table
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

Each successful transaction creates a new table version.

The transaction log allows Delta Lake to reconstruct the table state for a specific version.

### Why Is It Important?

- Supports ACID transactions.
- Maintains table version history.
- Tracks file additions and removals.
- Enables Time Travel.
- Helps manage concurrent operations.

## 5. How Delta Lake Works

A simplified Delta Lake write operation follows these steps:

```text
          Start Write Operation
                   |
                   v
             Read Table State
                   |
                   v
             Process Data
                   |
                   v
           Write New Parquet Files
                   |
                   v
          Validate Transaction
                   |
                   v
          Commit to Delta Log
                   |
                   v
          Create New Table Version
```

The transaction becomes visible to readers when the commit is successfully recorded.

Delta Lake generally writes new data files rather than modifying existing Parquet files in place.

## 6. Creating a Delta Table

There are multiple ways to create a Delta table.

### Method 1: Using PySpark

```python
data = [
    (1, "Amit", 5000),
    (2, "Rahul", 7000),
    (3, "Priya", 8000)
]

columns = ["id", "name", "salary"]

df = spark.createDataFrame(data, columns)

df.write.format("delta").mode("overwrite").save(
    "/mnt/delta/employees"
)
```

This writes the DataFrame in Delta format to the specified path.

### Method 2: Creating a Managed Delta Table

```sql
CREATE TABLE employees (
    id INT,
    name STRING,
    salary DOUBLE
)
USING DELTA;
```

This creates a table managed by the metastore/catalog according to the configured table management model.

### Method 3: Creating a Table from a DataFrame

```python
df.write.format("delta").mode("overwrite").saveAsTable(
    "employees"
)
```

This saves the DataFrame as a table.

**Important:** A managed table and a path-based Delta table have different metadata and lifecycle management behavior. Understand which one your project uses.

## 7. Reading Delta Tables

### Read Using PySpark

```python
df = spark.read.format("delta").load(
    "/mnt/delta/employees"
)

display(df)
```

### Read Using SQL

```sql
SELECT *
FROM employees;
```

For a path-based table, Databricks also supports querying Delta data by path:

```sql
SELECT *
FROM delta.`/mnt/delta/employees`;
```

## 8. Writing Data to Delta Tables

Delta Lake supports different write modes.

### Append Mode

Adds new records without replacing existing data.

```python
df.write.format("delta").mode("append").save(
    "/mnt/delta/employees"
)
```

### Overwrite Mode

Replaces the existing data according to the write operation and its configuration.

```python
df.write.format("delta").mode("overwrite").save(
    "/mnt/delta/employees"
)
```

### ErrorIfExists Mode

Fails if the target already exists.

```python
df.write.format("delta").mode("errorifexists").save(
    "/mnt/delta/employees"
)
```

### Ignore Mode

Does not write if the target already exists.

```python
df.write.format("delta").mode("ignore").save(
    "/mnt/delta/employees"
)
```

**Important:** Overwrite behavior depends on the target and options. A normal overwrite should not be confused with a controlled partition overwrite.

## 9. Schema Enforcement

Schema enforcement means Delta Lake checks incoming data against the target table schema during writes.

It helps prevent incompatible data from being written into a table.

Example:

Existing table schema:

```text
id       INT
name     STRING
salary   DOUBLE
```

Incoming DataFrame:

```text
id       INT
name     STRING
salary   STRING
```

The incompatible salary type may cause the write to fail.

Example:

```python
df.write.format("delta").mode("append").save(
    "/mnt/delta/employees"
)
```

If the incoming schema is incompatible with the existing table schema, Delta Lake can reject the write.

**Benefits:**
- Protects table structure.
- Prevents incompatible schema changes.
- Improves data reliability.
- Helps maintain consistent downstream processing.

**Remember:** Schema enforcement checks compatibility; it does not automatically clean invalid records or guarantee every business rule.

## 10. Schema Evolution

Schema evolution allows a Delta table's schema to change when new data introduces additional columns or compatible schema changes.

Example:

Existing table:

```text
id
name
salary
```

Incoming data:

```text
id
name
salary
department
```

The new column `department` can be added using supported schema evolution options.

### Using `mergeSchema`

```python
(
    new_df.write
    .format("delta")
    .mode("append")
    .option("mergeSchema", "true")
    .save("/mnt/delta/employees")
)
```

### Schema Enforcement vs. Schema Evolution

| Schema Enforcement | Schema Evolution |
|---|---|
| Validates incoming schema | Allows supported schema changes |
| Prevents unintended incompatibility | Supports controlled schema updates |
| Maintains table consistency | Adapts the table schema |
| Rejects incompatible writes | Can add new columns when enabled |

**Interview point:** Schema enforcement protects the existing structure, while schema evolution allows controlled changes to that structure.

## 11. UPDATE Operation

Delta Lake supports updating existing records.

Example:

```sql
UPDATE employees
SET salary = 9000
WHERE id = 1;
```

This updates the salary for the employee whose ID is 1.

Unlike a plain Parquet file, Delta Lake provides table-level update functionality through its transaction mechanism.

## 12. DELETE Operation

Delta Lake supports deleting records based on a condition.

Example:

```sql
DELETE FROM employees
WHERE id = 3;
```

This deletes the matching record from the current logical table state.

The old data file may still physically exist until it is eligible for cleanup.

## 13. MERGE Operation

MERGE is used to perform conditional inserts, updates, and deletes.

It is especially useful for:

- Incremental data loading
- CDC pipelines
- Upserts
- Synchronizing source and target tables

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

- Matching employee ID → Update existing record.
- Non-matching employee ID → Insert new record.

A MERGE operation is committed as a Delta transaction.

**Important:** Prepare the source data carefully. Multiple source rows matching the same target row can make an update ambiguous and cause the operation to fail.

## 14. ACID Transactions

Delta Lake supports ACID transactions.

ACID stands for:

| Property | Meaning |
|---|---|
| Atomicity | All changes commit or none do |
| Consistency | Preserves defined data rules |
| Isolation | Manages concurrent transactions |
| Durability | Committed changes persist |

### Example

Suppose an ETL pipeline updates 10,000 records.

If the transaction fails before commit, the incomplete changes are not exposed as a successful table update.

Delta Lake uses its transaction log and concurrency-control mechanisms to manage these operations.

**Remember:** ACID transactions make Delta Lake more reliable than managing independent Parquet files without a transaction layer.

## 15. Time Travel

Time Travel allows users to query earlier committed versions of a Delta table.

Delta Lake tracks versions through its transaction log.

### View Table History

```sql
DESCRIBE HISTORY employees;
```

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

### Benefits

- Investigate historical data.
- Compare table versions.
- Recover from accidental changes.
- Debug data pipeline issues.
- Audit table modifications.

**Important:** Time Travel depends on the availability of historical transaction-log information and the underlying data files. Retention and cleanup policies determine how far back historical versions remain queryable.

## 16. VACUUM

VACUUM removes obsolete data files that are no longer required by retained Delta table versions.

Example:

```sql
VACUUM employees;
```

Delta Lake retains removed files for a retention period before they can be physically deleted.

### Why Is VACUUM Important?

- Removes obsolete data files.
- Helps manage storage usage.
- Cleans up files no longer needed by retained table versions.

### Important Difference

| OPTIMIZE | VACUUM |
|---|---|
| Reorganizes data files | Removes obsolete data files |
| Commonly compacts small files | Cleans up unreferenced files |
| Can improve read performance | Helps control storage usage |
| Creates a new committed table version | Performs physical file cleanup |

**Warning:** Running VACUUM with an unsafe retention configuration can remove files required for Time Travel or active readers. Never reduce retention settings without understanding the workload and safety implications.

## 17. OPTIMIZE

OPTIMIZE reorganizes Delta table files to improve physical layout.

Example:

```sql
OPTIMIZE employees;
```

It is commonly used to compact small files.

For supported workloads, Z-Ordering can also be applied:

```sql
OPTIMIZE employees
ZORDER BY (id);
```

**Remember:**
- OPTIMIZE → Improves physical file layout.
- VACUUM → Removes obsolete physical files.

## 18. Delta Lake vs. Parquet vs. CSV

| Feature | CSV | Parquet | Delta Lake |
|---|---|---|---|
| Type | Text file format | Columnar file format | Table format |
| Human-readable | Yes | Not directly | Uses underlying data files |
| Compression | Limited/common external compression | Efficient columnar compression | Uses Parquet capabilities |
| ACID transactions | No | No built-in table transactions | Yes |
| Schema enforcement | No built-in table enforcement | Schema stored in file metadata | Yes |
| UPDATE and DELETE | Requires rewriting or external processing | Requires rewriting or external processing | Supported |
| MERGE | No native table operation | No native table operation | Supported |
| Time Travel | No | No | Yes |

**Remember:**
- CSV → Simple text-based data storage.
- Parquet → Efficient columnar data storage.
- Delta Lake → Reliable table management with transaction capabilities.

## 19. Delta Lake in a Data Engineering Pipeline

Delta Lake is commonly used in the Medallion Architecture.

```text
             Source Systems
                   |
                   v
              Bronze Layer
            Raw Delta Tables
                   |
                   v
              Silver Layer
          Cleaned and Validated
                   |
                   v
               Gold Layer
           Business Aggregations
                   |
                   v
          Reporting and Analytics
```

### Bronze Layer

Stores raw or minimally processed data.

### Silver Layer

Stores cleaned, validated, and transformed data.

### Gold Layer

Stores business-ready data for reporting and analytics.

Delta Lake can provide ACID transactions, schema controls, and incremental processing capabilities across these layers.

## 20. Best Practices

1. Use Delta format for tables requiring reliable transactional operations.
2. Define schemas explicitly for important production pipelines.
3. Enable schema evolution only when appropriate.
4. Use MERGE for suitable incremental and CDC workloads.
5. Use Time Travel for debugging and recovery when retained history is available.
6. Monitor transaction history.
7. Use OPTIMIZE when file layout needs improvement.
8. Run VACUUM with safe retention settings.
9. Avoid unnecessary schema changes.
10. Design pipelines to be idempotent.
11. Validate incoming data before writing.
12. Monitor small-file accumulation and query performance.

## 21. Common Interview Questions

### Q1. What is Delta Lake?

Delta Lake is an open-source storage framework that adds ACID transactions, schema enforcement, versioning, and other table-management capabilities to data lakes.

### Q2. Why do we need Delta Lake?

It helps manage reliable data pipelines by supporting transactions, updates, deletes, schema controls, and historical table versions.

### Q3. What is the difference between Parquet and Delta Lake?

Parquet is a columnar file format. Delta Lake is a table format that commonly uses Parquet data files and adds transaction management through a transaction log.

### Q4. What is the `_delta_log` directory?

It stores transaction-log information used to track table changes, metadata, and committed versions.

### Q5. What is schema enforcement?

Schema enforcement validates incoming data against the existing table schema and rejects incompatible writes.

### Q6. What is schema evolution?

Schema evolution allows supported changes to the table schema, such as adding new columns.

### Q7. What is MERGE?

MERGE performs conditional inserts, updates, and deletes based on matching conditions between source and target data.

### Q8. What are ACID transactions?

ACID stands for Atomicity, Consistency, Isolation, and Durability. These properties help ensure reliable table transactions.

### Q9. What is Time Travel?

Time Travel allows users to query earlier committed versions of a Delta table.

### Q10. What is VACUUM?

VACUUM physically removes obsolete data files that are no longer required according to the table's retention rules.

### Q11. What is the difference between OPTIMIZE and VACUUM?

OPTIMIZE reorganizes data files to improve file layout. VACUUM removes obsolete files to manage storage.

### Q12. Can Delta Lake support concurrent writes?

Yes. Delta Lake supports concurrent operations using transaction validation and concurrency-control mechanisms. Conflicting writes may fail and require handling or retry.

### Q13. Is Delta Lake a replacement for Parquet?

No. Delta Lake commonly uses Parquet as its underlying data-file format and adds transaction and table-management capabilities.

### Q14. How is Delta Lake used in a Medallion Architecture?

It can store Bronze, Silver, and Gold tables while providing reliable transactions and table-management features across the data pipeline.

## 22. Quick Revision

| Concept | Remember |
|---|---|
| Delta Lake | Reliable table format for data lakes |
| Parquet | Underlying columnar data-file format |
| `_delta_log` | Transaction log |
| ACID | Atomicity, Consistency, Isolation, Durability |
| Schema Enforcement | Validates incoming schema |
| Schema Evolution | Supports controlled schema changes |
| Append | Adds records |
| Overwrite | Replaces existing data according to write configuration |
| UPDATE | Modifies matching records |
| DELETE | Removes matching records from current table state |
| MERGE | Conditional insert, update, and delete |
| Time Travel | Queries previous table versions |
| VACUUM | Removes obsolete physical files |
| OPTIMIZE | Reorganizes table files |
| Bronze | Raw data |
| Silver | Cleaned and validated data |
| Gold | Business-ready data |

## 23. Self-Assessment

Before moving to the next topic, make sure you can explain:

- [ ] What is Delta Lake?
- [ ] Why do we need Delta Lake?
- [ ] What is the difference between a data lake and Delta Lake?
- [ ] How does Delta Lake architecture work?
- [ ] What is the `_delta_log` directory?
- [ ] How do you create and read a Delta table?
- [ ] What are the different write modes?
- [ ] What is schema enforcement?
- [ ] What is schema evolution?
- [ ] How do UPDATE, DELETE, and MERGE work?
- [ ] What are ACID transactions?
- [ ] What is Time Travel?
- [ ] What is VACUUM?
- [ ] What is the difference between OPTIMIZE and VACUUM?
- [ ] How is Delta Lake used in the Medallion Architecture?

**Interview readiness:** You should be able to explain Delta Lake from its architecture to production usage, including transaction logs, ACID properties, schema management, incremental processing, Time Travel, and file optimization.