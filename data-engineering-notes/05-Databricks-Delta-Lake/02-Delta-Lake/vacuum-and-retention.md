# VACUUM and Retention in Delta Lake

## 1. Overview

Delta Lake uses **VACUUM** to physically remove old data files that are no longer required by the current table snapshot.

When we perform operations such as UPDATE, DELETE, or MERGE, Delta Lake generally creates new Parquet files and marks older files as removed from the current table snapshot.

However, those old files are not necessarily deleted immediately.

They remain in storage for a retention period so that historical versions can remain accessible.

**VACUUM** cleans up eligible unreferenced files after the configured retention period.

### Key concepts

- **VACUUM:** Physically removes eligible unreferenced data files.
- **Retention:** Defines how long old files or transaction-log history are retained.
- **Time Travel:** Allows access to historical table versions while the required log and data files remain available.

## 2. Why Does Delta Lake Need VACUUM?

Consider a Delta table containing:

```text
customer_id | name
------------|------
1           | Rahul
2           | Amit
```

Now execute:

```sql
UPDATE customers
SET name = 'Rahul Kumar'
WHERE customer_id = 1;
```

Delta Lake generally performs the following:

1. Reads the relevant existing data.
2. Writes replacement Parquet data files.
3. Records the old files as removed from the current snapshot.
4. Records the new files as added.
5. Commits the transaction.

The old Parquet files may still exist physically.

This happens because Delta Lake preserves the ability to access older snapshots, subject to retention and other conditions.

Over time, these unreferenced files can consume significant storage.

VACUUM helps remove eligible files and control storage growth.

## 3. What Does VACUUM Do?

VACUUM identifies files that are no longer referenced by the current table snapshot and have passed the applicable retention threshold.

Example:

```sql
VACUUM customers;
```

This removes eligible old data files from the table's storage.

**Important:**

- VACUUM physically deletes eligible data files.
- It does not simply remove rows from the current table.
- It does not delete the entire transaction log.
- It does not automatically remove every old file.
- It can make historical versions unavailable if they require deleted files.

## 4. How VACUUM Works

Suppose the table has these versions:

| Version | Operation | Data change |
|---|---|---|
| 0 | CREATE | Initial data files |
| 1 | UPDATE | Replacement files created |
| 2 | DELETE | More files replaced or removed |
| 3 | MERGE | Additional files added and removed |

Some files referenced by older versions are no longer referenced by the current version.

Conceptually, VACUUM:

1. Identifies files in the table storage.
2. Determines which files are no longer required by the current snapshot.
3. Checks whether they have passed the retention threshold.
4. Deletes eligible files.

The actual implementation also considers transaction-log information, retention settings, and supported table features.

## 5. VACUUM Syntax

### A. Basic VACUUM

```sql
VACUUM customers;
```

Uses the configured retention threshold, commonly seven days.

### B. VACUUM with a custom retention period

```sql
VACUUM customers RETAIN 168 HOURS;
```

168 hours = 7 days.

Another example:

```sql
VACUUM customers RETAIN 720 HOURS;
```

720 hours = 30 days.

A longer retention period preserves eligible historical data files for longer, but consumes more storage.

### C. Dry Run

Before deleting files, we can inspect which files would be removed.

```sql
VACUUM customers DRY RUN;
```

Dry Run lists eligible files without physically deleting them.

**Best practice:** Use DRY RUN before a production VACUUM operation, especially when changing retention settings or cleaning up an important table.

## 6. Deleted-File Retention

Delta Lake uses the table property:

```text
delta.deletedFileRetentionDuration
```

It controls the minimum retention period for logically removed data files before they become eligible for VACUUM.

The commonly used default is:

```text
7 days
```

### Check table properties

```sql
SHOW TBLPROPERTIES customers;
```

### Change deleted-file retention

```sql
ALTER TABLE customers
SET TBLPROPERTIES (
    'delta.deletedFileRetentionDuration' = 'interval 30 days'
);
```

Now logically removed files must satisfy the configured 30-day retention period before they become eligible for VACUUM.

**Important:** Changing the property does not bring back files that have already been deleted.

## 7. Transaction Log Retention

Delta Lake also maintains transaction-log history.

The table property is:

```text
delta.logRetentionDuration
```

The commonly used default is:

```text
30 days
```

It controls how long transaction-log history is retained before old log information becomes eligible for cleanup.

### Change log retention

```sql
ALTER TABLE customers
SET TBLPROPERTIES (
    'delta.logRetentionDuration' = 'interval 60 days'
);
```

This increases the configured log-history retention period to 60 days.

### Deleted-file retention vs log retention

| Property | Common Default | Purpose |
|---|---:|---|
| `delta.deletedFileRetentionDuration` | 7 days | Controls eligibility of logically removed data files for VACUUM |
| `delta.logRetentionDuration` | 30 days | Controls transaction-log history retention |

**Important:** These settings control different things.

Increasing transaction-log retention does not guarantee that the data files needed for historical queries will remain available.

Similarly, retaining data files does not guarantee that all historical transaction-log information will remain available indefinitely.

## 8. How VACUUM Affects Time Travel

Time Travel requires:

1. Transaction-log information for the requested version.
2. The underlying data files needed to reconstruct that version.

Consider this example:

| Version | Operation |
|---|---|
| 0 | Initial data |
| 1 | UPDATE |
| 2 | DELETE |

Suppose version 0 requires an old Parquet file that is no longer referenced by version 2.

If VACUUM physically deletes that file, reading version 0 may fail.

Example:

```sql
SELECT *
FROM customers VERSION AS OF 0;
```

This query requires the historical data files and log information to still be available.

### Important interview point

**VACUUM can permanently remove the physical data required for historical Time Travel queries.**

Time Travel is therefore limited by both log retention and data-file availability.

## 9. Retention Safety Check

Delta Lake includes a retention safety check to help prevent unsafe VACUUM operations.

For example, attempting to run:

```sql
VACUUM customers RETAIN 0 HOURS;
```

may fail because the requested retention period is shorter than the configured safety threshold.

A related configuration in Databricks is:

```python
spark.conf.set(
    "spark.databricks.delta.retentionDurationCheck.enabled",
    "false"
)
```

This disables the retention-duration safety check in supported Databricks environments.

**Warning:** Do not disable this check casually.

A short retention period can delete files that are still needed by:
- Long-running readers.
- Concurrent jobs.
- Historical Time Travel queries.
- Recovery processes.

Only consider changing this protection after understanding the operational impact and ensuring the environment is safe.

## 10. VACUUM vs DELETE vs DROP TABLE

These operations have different purposes.

| Operation | Purpose | Physical File Removal |
|---|---|---|
| DELETE | Removes matching rows logically | Not necessarily immediate |
| VACUUM | Removes eligible unreferenced data files | Yes |
| DROP TABLE | Removes the table definition and affects its data according to table type and platform behavior | Depends on managed/external table lifecycle |

### Example: DELETE

```sql
DELETE FROM customers
WHERE customer_id = 10;
```

This removes matching rows from the current table snapshot.

### Example: VACUUM

```sql
VACUUM customers;
```

This physically removes eligible old data files.

**Remember:** DELETE changes table contents. VACUUM cleans up eligible physical files.

## 11. VACUUM and Long-Running Jobs

Suppose a data pipeline starts reading an older table snapshot.

At the same time, another job executes VACUUM with an unsafe retention period.

The reader may still need files that VACUUM removes.

This can cause failures.

That is why retention must account for:
- Maximum job execution time.
- Long-running queries.
- Concurrent readers and writers.
- Recovery requirements.
- Time Travel requirements.

The retention period should be selected based on actual workload needs, not just storage savings.

## 12. Real-World Data Engineering Example

Suppose a company processes customer data every day.

Its Delta table receives:
- Daily INSERT operations.
- Periodic UPDATE operations.
- DELETE operations for removed customers.
- MERGE operations from CDC data.

Over time, old data files accumulate.

A controlled maintenance process might be:

### Step 1: Check table history

```sql
DESCRIBE HISTORY customers;
```

### Step 2: Check retention properties

```sql
SHOW TBLPROPERTIES customers;
```

### Step 3: Preview eligible files

```sql
VACUUM customers DRY RUN;
```

### Step 4: Execute VACUUM

```sql
VACUUM customers;
```

### Step 5: Validate the table

```sql
SELECT COUNT(*)
FROM customers;
```

Also test required historical queries and downstream workloads according to the recovery policy.

**Production consideration:** VACUUM is a maintenance operation. Schedule it carefully and align its retention period with the organization's Time Travel and recovery requirements.

## 13. Best Practices

1. **Understand retention requirements:** Decide how long historical data must remain accessible.
2. **Use DRY RUN:** Preview files before physical deletion.
3. **Avoid very short retention periods:** They can interfere with long-running jobs and historical reads.
4. **Monitor storage usage:** VACUUM can help control storage growth.
5. **Coordinate with Time Travel:** Ensure retention supports the required historical access period.
6. **Review both retention properties:** Log retention and deleted-file retention serve different purposes.
7. **Schedule maintenance carefully:** Avoid conflicts with long-running workloads.
8. **Do not disable safety checks casually:** Understand the consequences first.
9. **Maintain backups when required:** VACUUM is not a backup or disaster-recovery mechanism.

## 14. Common Interview Questions

**Q1. What is VACUUM in Delta Lake?**

VACUUM physically removes eligible data files that are no longer referenced by the current Delta table snapshot.

**Q2. Why do we need VACUUM?**

UPDATE, DELETE, and MERGE operations can leave old data files in storage. VACUUM removes eligible files to reduce unnecessary storage usage.

**Q3. Does DELETE immediately remove Parquet files?**

No. DELETE logically removes records from the current snapshot. Old files may remain physically stored until they become eligible for VACUUM.

**Q4. What is the commonly used default deleted-file retention period?**

Seven days, controlled by `delta.deletedFileRetentionDuration`.

**Q5. What is the commonly used default transaction-log retention period?**

Thirty days, controlled by `delta.logRetentionDuration`.

**Q6. What is VACUUM DRY RUN?**

It previews eligible files that would be deleted without physically deleting them.

**Q7. How does VACUUM affect Time Travel?**

VACUUM can physically delete old data files required by historical versions, making those versions unreadable.

**Q8. What is the difference between log retention and deleted-file retention?**

Log retention controls historical transaction-log information. Deleted-file retention controls when logically removed data files become eligible for VACUUM.

**Q9. Why should we avoid VACUUM with zero-hour retention?**

It can delete files still needed by concurrent readers, long-running queries, or historical operations.

**Q10. Does VACUUM delete the transaction log?**

VACUUM primarily removes eligible unreferenced data files. Transaction-log cleanup is a separate process governed by log retention.

**Q11. Is VACUUM a backup mechanism?**

No. It is a storage-cleanup operation. It does not provide a separate recoverable copy of the table.

**Q12. How do you change the deleted-file retention period?**

```sql
ALTER TABLE customers
SET TBLPROPERTIES (
    'delta.deletedFileRetentionDuration' = 'interval 30 days'
);
```

## 15. Quick Revision

| Concept | Remember |
|---|---|
| VACUUM | Physically removes eligible unreferenced files |
| DELETE | Logically removes rows |
| DRY RUN | Previews files to be deleted |
| Deleted-file retention | Common default: 7 days |
| Log retention | Common default: 30 days |
| `delta.deletedFileRetentionDuration` | Controls deleted-file eligibility |
| `delta.logRetentionDuration` | Controls transaction-log history retention |
| Time Travel | Can fail if required files are deleted |
| Retention safety check | Helps prevent unsafe cleanup |
| Production practice | Align retention with workload and recovery needs |

## 16. Self-Assessment

- [ ] Explain why Delta Lake needs VACUUM.
- [ ] Explain what happens to files after UPDATE and DELETE.
- [ ] Write the VACUUM SQL command.
- [ ] Explain VACUUM DRY RUN.
- [ ] Explain deleted-file retention.
- [ ] Explain transaction-log retention.
- [ ] Differentiate the two retention properties.
- [ ] Explain how VACUUM affects Time Travel.
- [ ] Explain the retention safety check.
- [ ] Differentiate DELETE, VACUUM, and DROP TABLE.
- [ ] Describe a safe production maintenance workflow.

**Interview takeaway:** VACUUM physically removes eligible old data files from Delta tables. The deleted-file retention property controls when those files become eligible, while log retention controls historical transaction-log cleanup. Both matter when designing Time Travel, recovery, and production maintenance policies.