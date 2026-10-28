# Isolation Levels in SQL

## 1. What Are Isolation Levels?

Isolation levels define how much one transaction is isolated from other concurrent transactions.

They control which changes a transaction can observe while other transactions are reading or modifying the same data.

Isolation levels help maintain data consistency when multiple users or processes access a database simultaneously.

**Example:** Two transactions attempt to update the same bank account balance. The database must control how their operations interact to avoid incorrect results.

## 2. What Is a Transaction?

A transaction is a group of database operations treated as one logical unit of work.

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE account_id = 101;

UPDATE accounts
SET balance = balance + 1000
WHERE account_id = 102;

COMMIT;
```

This transaction transfers money between two accounts.

- `BEGIN`: Starts the transaction.
- `UPDATE`: Modifies account balances.
- `COMMIT`: Makes the transaction's changes permanent.

If an error occurs before committing, the transaction may be rolled back.

The exact transaction-control syntax depends on the database engine.

## 3. Why Are Isolation Levels Needed?

Suppose two transactions access the same account balance at the same time.

| Time | Transaction A | Transaction B |
|---|---|---|
| 1 | Reads balance = 1000 | |
| 2 | | Reads balance = 1000 |
| 3 | Adds 500 and writes 1500 | |
| 4 | | Adds 200 and writes 1200 |

If both transactions overwrite the balance based on their earlier reads, one update may be lost.

The final balance could become 1200 or 1500 instead of the expected 1700.

This is an example of a **lost update**. Databases use concurrency-control mechanisms such as locks, versioning, and conflict detection to prevent or manage these problems.

## 4. The Four Standard Isolation Levels

The SQL standard defines four commonly discussed isolation levels.

| Isolation level | Dirty reads | Non-repeatable reads | Phantom reads |
|---|---|---|---|
| Read Uncommitted | Possible | Possible | Possible |
| Read Committed | Prevented | Possible | Possible |
| Repeatable Read | Prevented | Prevented | Possible under the standard |
| Serializable | Prevented | Prevented | Prevented |

These are the standard's broad guarantees. Actual behavior, including how phantom reads and write conflicts are handled, depends on the database engine and its concurrency-control implementation.

## 5. Read Uncommitted

Read Uncommitted is the weakest standard isolation level.

A transaction may read changes made by another transaction before those changes have been committed. This is called a **dirty read**.

### Example

Transaction A:

```sql
BEGIN;

UPDATE accounts
SET balance = 500
WHERE account_id = 101;

-- Transaction A has not committed.
```

Transaction B:

```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

SELECT balance
FROM accounts
WHERE account_id = 101;
```

Transaction B may read the uncommitted balance of 500 in a database that supports dirty reads at this level.

If Transaction A rolls back, Transaction B has read a value that never became permanent.

**Key point:** Read Uncommitted permits dirty reads under the SQL standard, but some databases do not actually allow them, even when this level is requested.

## 6. Read Committed

Read Committed prevents dirty reads. A transaction can read only committed data.

However, the same query executed twice within one transaction may return different results if another transaction commits changes between the reads.

This is called a **non-repeatable read** when the same row's values change.

### Example

Transaction A:

```sql
BEGIN;

SELECT balance
FROM accounts
WHERE account_id = 101;

-- Suppose the balance is 1000.
```

Transaction B:

```sql
BEGIN;

UPDATE accounts
SET balance = 1500
WHERE account_id = 101;

COMMIT;
```

Transaction A runs the same query again:

```sql
SELECT balance
FROM accounts
WHERE account_id = 101;
```

The second result may be 1500.

**Key point:** Read Committed prevents dirty reads, but it does not necessarily provide one consistent snapshot for the entire transaction.

## 7. Repeatable Read

Repeatable Read prevents dirty reads and non-repeatable reads under the SQL standard.

If a transaction reads a row and reads it again, the previously read values should remain consistent according to the level's guarantees.

However, the SQL standard permits phantom reads at this level.

### What Is a Phantom Read?

A phantom read occurs when the same condition-based query returns a different set of rows because another transaction inserts or deletes rows that match the condition.

For example, Transaction A executes:

```sql
SELECT *
FROM employees
WHERE salary > 80000;
```

Transaction B inserts a new employee whose salary is 90000 and commits.

If Transaction A repeats the query and the new row appears, it has experienced a phantom read.

**Important:** Repeatable Read implementations differ. PostgreSQL's Repeatable Read uses snapshot isolation and prevents ordinary phantom reads, while other database engines may have different behavior.

## 8. Serializable

Serializable is the strongest of the four standard isolation levels.

Its goal is to make concurrent transactions behave as though they had executed in some serial order, one transaction at a time.

This helps prevent anomalies that weaker isolation levels may permit.

### Example

Suppose a system must ensure that at least one doctor remains on call.

Two transactions each read that two doctors are on call. Each transaction then marks a different doctor as off call.

Without sufficient concurrency control, both updates could commit, leaving no doctor on call.

Serializable isolation can prevent this outcome by blocking a transaction or aborting one when the concurrent execution cannot be represented by a valid serial order.

**Trade-off:** Serializable execution can introduce more blocking, retries, or transaction aborts. Applications may need to retry transactions that fail due to serialization conflicts.

## 9. Three Important Concurrency Problems

### 9.1 Dirty Read

A transaction reads another transaction's uncommitted changes.

The other transaction may later roll back.

### 9.2 Non-Repeatable Read

A transaction reads the same row twice and gets different values because another transaction committed an update or deletion between the reads.

### 9.3 Phantom Read

A transaction repeats a condition-based query and finds a different set of qualifying rows because another transaction inserted or deleted matching rows.

### Quick Comparison

| Problem | What changes? |
|---|---|
| Dirty read | Uncommitted data is read |
| Non-repeatable read | Previously read row values or existence change |
| Phantom read | The set of rows matching a condition changes |

A related problem is a **lost update**, in which concurrent writes overwrite one another's changes. Whether a particular isolation level prevents it depends on the database engine and the transaction pattern.

## 10. Set the Isolation Level

Isolation-level syntax varies by database.

### PostgreSQL

```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

SELECT *
FROM employees
WHERE department = 'IT';

COMMIT;
```

### SQL Server

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

BEGIN TRANSACTION;

SELECT *
FROM employees
WHERE department = 'IT';

COMMIT;
```

SQL Server also supports options such as `READ_COMMITTED_SNAPSHOT` and `SNAPSHOT`, which affect concurrency behavior.

### MySQL

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

START TRANSACTION;

SELECT *
FROM employees
WHERE department = 'IT';

COMMIT;
```

MySQL's default isolation level is commonly Repeatable Read with InnoDB, but configuration and engine choices can differ.

**Interview note:** Always distinguish the isolation-level guarantees defined by the SQL standard from the behavior of a particular database engine.

## 11. Isolation Levels in Data Engineering

Isolation levels matter in data pipelines and analytical systems when multiple jobs or users read and write data concurrently.

Common situations include:

- Multiple ETL jobs updating the same relational tables.
- Incremental loads reading source data while transactions are still in progress.
- Data validation queries running alongside updates.
- Reporting queries requiring a consistent view of related records.
- Concurrent updates to inventory, financial, or customer data.

For example, a pipeline that reads several related tables may require a consistent snapshot to avoid processing values from incompatible points in time.

The appropriate isolation level depends on correctness requirements, throughput, locking behavior, and the database's snapshot implementation.

## 12. Common Interview Questions

**Q1. Which isolation level is the strongest?**

Serializable, because it aims to make concurrent execution equivalent to a serial execution order.

**Q2. Which level prevents dirty reads but allows non-repeatable reads?**

Read Committed.

**Q3. Which level prevents non-repeatable reads under the SQL standard but may permit phantom reads?**

Repeatable Read.

**Q4. What is the difference between Repeatable Read and Serializable?**

Serializable provides stronger guarantees about the overall outcome of concurrent transactions. Repeatable Read does not universally prevent every possible concurrency anomaly.

**Q5. Does Read Committed guarantee that two SELECT statements return the same result?**

No. Another transaction may commit changes between the statements.

**Q6. Does Serializable mean transactions literally execute one at a time?**

No. A database can execute operations concurrently as long as the resulting behavior is equivalent to a valid serial execution.

**Q7. Why might a database use snapshot isolation?**

Snapshot isolation lets a transaction read a consistent version of data while reducing some forms of reader-writer blocking. However, snapshot isolation is not identical to Serializable and may allow certain anomalies, such as write skew.

## 13. Common Mistakes

- Confusing dirty reads with non-repeatable reads.
- Assuming Repeatable Read behaves identically across all databases.
- Assuming Serializable always executes transactions sequentially.
- Ignoring the performance cost of stronger isolation.
- Assuming snapshot isolation is the same as Serializable.
- Forgetting that isolation-level settings can be overridden or affected by database-specific options.
- Ignoring transaction retries after serialization conflicts.

## Quick Revision

- **Read Uncommitted:** Dirty reads may occur.
- **Read Committed:** Prevents dirty reads.
- **Repeatable Read:** Prevents dirty and non-repeatable reads under the SQL standard.
- **Serializable:** Provides the strongest standard isolation guarantee.
- **Dirty read:** Reading uncommitted data.
- **Non-repeatable read:** Previously read row values change.
- **Phantom read:** The set of rows matching a query changes.
- **Snapshot isolation:** Provides a consistent data snapshot but is not universally equivalent to Serializable.

**Core interview takeaway:** Understand the four isolation levels, distinguish the three classic read anomalies, and explain how database-specific implementations balance consistency, concurrency, and performance.
