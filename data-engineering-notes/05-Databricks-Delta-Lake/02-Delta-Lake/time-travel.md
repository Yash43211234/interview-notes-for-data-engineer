# Time Travel in Delta Lake

## 1. Overview

Time Travel is a Delta Lake feature that allows us to access previous versions of a table.

Whenever a successful transaction modifies a Delta table, Delta Lake creates a new version in its transaction log.

Using Time Travel, we can:
- Read historical table data.
- Compare current and previous versions.
- Investigate incorrect data changes.
- Recover data from an earlier version.
- Support auditing and debugging.

**Example:**

Suppose a Delta table has the following versions:

| Version | Operation | Description |
|---|---|---|
| 0 | CREATE TABLE | Initial table |
| 1 | INSERT | New records added |
| 2 | UPDATE | Existing records modified |
| 3 | DELETE | Records removed |

The current version is 3, but we can still query version 0, 1, or 2 if the required transaction log and data files are retained.

## 2. How Time Travel Works

Delta Lake maintains a transaction log in a directory called:

```text
_delta_log/
```

This log records table changes and metadata.

For example:

```text
customers/
│
├── _delta_log/
│   ├── 00000000000000000000.json
│   ├── 00000000000000000001.json
│   ├── 00000000000000000002.json
│   └── 00000000000000000003.json
│
├── part-00000-....parquet
├── part-00001-....parquet
└── part-00002-....parquet
```

Each committed version represents a consistent snapshot of the table.

When a record is updated or deleted, Delta Lake generally writes new data files and records the corresponding file additions and removals in the transaction log.

Old data files may remain available for historical reads until they are physically removed.

**Important:** Time Travel does not mean Delta Lake stores a complete duplicate copy of the table for every version. It reconstructs historical snapshots using transaction-log information and the data files referenced by those versions.

## 3. Two Ways to Access Historical Data

Delta Lake supports two main ways to query an older table version:

1. Version-based Time Travel.
2. Timestamp-based Time Travel.

### A. Version-Based Time Travel

Use a specific transaction version number.

**SQL syntax:**

```sql
SELECT *
FROM customers VERSION AS OF 2;
```

This reads the table as it existed at version 2.

**PySpark syntax:**

```python
df = (
    spark.read
    .format("delta")
    .option("versionAsOf", 2)
    .table("customers")
)

df.show()
```

You can also read a Delta table using its storage path:

```python
df = (
    spark.read
    .format("delta")
    .option("versionAsOf", 2)
    .load("/mnt/delta/customers")
)
```

### B. Timestamp-Based Time Travel

Instead of specifying a version, specify a timestamp.

**SQL syntax:**

```sql
SELECT *
FROM customers
TIMESTAMP AS OF '2026-10-20 10:00:00';
```

This reads the table snapshot associated with the specified timestamp, provided the required historical information is retained.

**PySpark syntax:**

```python
df = (
    spark.read
    .format("delta")
    .option(
        "timestampAsOf",
        "2026-10-20 10:00:00"
    )
    .table("customers")
)
```

**Remember:**

- `VERSION AS OF` uses a version number.
- `TIMESTAMP AS OF` uses a timestamp.

Version-based queries are generally more precise when you already know the required transaction version.

## 4. View Table History

Before accessing an older version, we can inspect the table's history.

### SQL

```sql
DESCRIBE HISTORY customers;
```

This displays information such as:

- Version number
- Timestamp
- Operation
- Operation parameters
- User information, when available
- Job or notebook information, when available

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

To retrieve only the latest 5 history entries:

```python
history_df = delta_table.history(5)

history_df.show(truncate=False)
```

**Interview point:** `DESCRIBE HISTORY` helps identify which version should be used for historical analysis or recovery.

## 5. RESTORE a Previous Version

Time Travel allows us to read historical data.

But what if we want to make an older version the current table state?

We can use the `RESTORE` command.

### Restore by version

```sql
RESTORE TABLE customers
TO VERSION AS OF 2;
```

This restores the table to the state represented by version 2.

### Restore by timestamp

```sql
RESTORE TABLE customers
TO TIMESTAMP AS OF '2026-10-20 10:00:00';
```

### Important: RESTORE vs Time Travel

| Time Travel | RESTORE |
|---|---|
| Reads an older version | Restores an older state |
| Does not change the current table version | Creates a new committed table version |
| Used for historical queries | Used for recovery |
| Historical data is returned | Current table is changed to the restored state |

**Important:** RESTORE does not erase the transaction history. It creates a new transaction that makes the table data match the selected historical version.

For example:

```text
Version 0 → Initial data
Version 1 → Insert
Version 2 → Update
Version 3 → Delete
Version 4 → RESTORE TO VERSION 2
```

Version 4 becomes the current version, with data matching version 2.

## 6. Time Travel in Real-World Data Engineering

Consider a daily ETL pipeline that loads customer data into a Delta table.

On Monday:
- 10,000 customer records are loaded.

On Tuesday:
- Customer records are updated.

On Wednesday:
- An incorrect DELETE operation removes important records.

We can investigate the issue using Time Travel.

### Step 1: Check table history

```sql
DESCRIBE HISTORY customers;
```

Identify the version before the incorrect DELETE operation.

### Step 2: Read the previous version

```sql
SELECT *
FROM customers VERSION AS OF 1;
```

### Step 3: Compare current and historical data

```sql
SELECT *
FROM customers;
```

Compare the current result with the historical result.

### Step 4: Restore the previous version if required

```sql
RESTORE TABLE customers
TO VERSION AS OF 1;
```

Now the current table reflects the selected historical version.

**Production consideration:** Before restoring, investigate the impact on downstream pipelines and consumers. A restore creates a new version and may affect data that was legitimately written after the selected version.

## 7. Time Travel Retention

Time Travel depends on the availability of two things:

1. Transaction-log history.
2. The underlying data files needed for the historical snapshot.

If required files are physically deleted, the old version may no longer be readable.

### Important retention settings

| Configuration | Common Default | Purpose |
|---|---:|---|
| `delta.logRetentionDuration` | 30 days | Controls transaction-log history retention |
| `delta.deletedFileRetentionDuration` | 7 days | Controls the minimum retention period for logically removed data files before VACUUM can delete them |

These are commonly used Delta table defaults, but table properties, platform behavior, and configurations can change them.

### Check table properties

```sql
SHOW TBLPROPERTIES customers;
```

### Set retention properties

```sql
ALTER TABLE customers
SET TBLPROPERTIES (
    'delta.logRetentionDuration' = 'interval 60 days',
    'delta.deletedFileRetentionDuration' = 'interval 30 days'
);
```

**Important:** Increasing log retention alone does not guarantee that historical data remains available. The underlying data files must also be retained.

## 8. VACUUM and Time Travel

`VACUUM` physically removes eligible data files that are no longer referenced by the current table snapshot.

Example:

```sql
VACUUM customers;
```

By default, Delta Lake commonly uses a 7-day retention threshold for eligible deleted files.

### How VACUUM affects Time Travel

Suppose:

- Version 1 contains a record.
- Version 2 updates that record.
- The old data file is no longer referenced by the current version.
- VACUUM physically deletes the old file after it becomes eligible.

The transaction log may still contain historical information, but the old data file is unavailable.

Therefore, querying version 1 may fail.

### Important distinction

| Transaction Log | Data Files |
|---|---|
| Records table transactions and metadata | Store actual table data |
| Retention controlled by log retention settings | Old files can be physically deleted by VACUUM |
| Needed to reconstruct historical snapshots | Needed to read the actual historical data |

**Interview point:** Time Travel is limited by both transaction-log retention and data-file retention. VACUUM can make older versions unreadable even if some transaction history remains.

## 9. Important Commands

| Requirement | Command |
|---|---|
| View history | `DESCRIBE HISTORY customers` |
| Read version | `SELECT * FROM customers VERSION AS OF 2` |
| Read by timestamp | `SELECT * FROM customers TIMESTAMP AS OF '2026-10-20 10:00:00'` |
| Restore version | `RESTORE TABLE customers TO VERSION AS OF 2` |
| View properties | `SHOW TBLPROPERTIES customers` |
| Remove eligible old files | `VACUUM customers` |

## 10. Best Practices

1. **Check table history before recovery:** Identify the correct version and operation.
2. **Set appropriate retention:** Choose retention periods based on recovery and compliance requirements.
3. **Understand VACUUM:** Do not assume old versions will remain available indefinitely.
4. **Test recovery procedures:** Verify that historical reads and RESTORE work as expected.
5. **Use version numbers for precise recovery:** They make it easier to identify a specific transaction.
6. **Coordinate restores with downstream systems:** A restored table can change the data expected by downstream pipelines.
7. **Avoid unsafe retention settings:** Do not disable VACUUM retention safety checks casually.
8. **Maintain separate backups when required:** Time Travel is not a replacement for a complete disaster-recovery strategy.

## 11. Common Interview Questions

**Q1. What is Time Travel in Delta Lake?**

Time Travel allows us to read previous versions of a Delta table using a version number or timestamp.

**Q2. How does Delta Lake support Time Travel?**

Delta Lake uses its transaction log and underlying data files to reconstruct historical table snapshots.

**Q3. How do you check the history of a Delta table?**

```sql
DESCRIBE HISTORY customers;
```

**Q4. What is the difference between VERSION AS OF and TIMESTAMP AS OF?**

`VERSION AS OF` accesses a table using a transaction version number, whereas `TIMESTAMP AS OF` accesses a snapshot using a timestamp.

**Q5. What is the difference between Time Travel and RESTORE?**

Time Travel reads historical data without changing the current table. RESTORE creates a new transaction that makes an older table state current.

**Q6. Does RESTORE delete newer versions?**

No. RESTORE creates a new version. Previous transaction history is not simply erased.

**Q7. How does VACUUM affect Time Travel?**

VACUUM physically deletes eligible unreferenced data files. If historical versions need those files, they may no longer be readable.

**Q8. What is the default deleted-file retention period?**

The commonly used Delta Lake default is 7 days, controlled by `delta.deletedFileRetentionDuration`. Actual settings should be checked for the table.

**Q9. Is transaction-log retention the same as data-file retention?**

No. Transaction-log retention controls historical log information, while deleted-file retention controls when old data files become eligible for physical deletion.

**Q10. Is Time Travel a backup mechanism?**

No. It is a table-versioning and recovery feature. It depends on retained transaction history and data files, so it should not replace a complete backup and disaster-recovery strategy.

**Q11. Can we restore a table using a timestamp?**

Yes.

```sql
RESTORE TABLE customers
TO TIMESTAMP AS OF '2026-10-20 10:00:00';
```

**Q12. What happens if we try to read a version whose data files were removed by VACUUM?**

The historical query can fail because the files required to reconstruct that version are no longer available.

## 12. Quick Revision

| Concept | Remember |
|---|---|
| Time Travel | Access previous table versions |
| `_delta_log` | Stores transaction history |
| VERSION AS OF | Read using version number |
| TIMESTAMP AS OF | Read using timestamp |
| DESCRIBE HISTORY | View table operations |
| RESTORE | Make an older state current |
| VACUUM | Physically removes eligible old files |
| Log retention | Common default: 30 days |
| Deleted-file retention | Common default: 7 days |
| Historical availability | Requires log and data files |

## 13. Self-Assessment

- [ ] Explain Time Travel in Delta Lake.
- [ ] Explain the role of `_delta_log`.
- [ ] Read an older version using SQL.
- [ ] Read an older version using PySpark.
- [ ] Query table history.
- [ ] Explain RESTORE with an example.
- [ ] Differentiate Time Travel and RESTORE.
- [ ] Explain how VACUUM affects historical versions.
- [ ] Differentiate log retention and deleted-file retention.
- [ ] Explain why Time Travel is not a complete backup solution.

**Interview takeaway:** Delta Lake Time Travel allows us to query historical table snapshots using version numbers or timestamps. `DESCRIBE HISTORY` helps identify changes, `RESTORE` recovers an earlier state, and `VACUUM` can permanently remove old data files required for historical queries.