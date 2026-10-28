# SQL Transactions and ACID Properties

## 1. What Is a Transaction?

A **transaction** is a group of database operations treated as one logical unit of work.

A transaction should either complete successfully or, if it fails, leave the database without the partial changes that the transaction was supposed to make.

For example, transferring ₹1,000 from Account A to Account B requires two operations:

1. Deduct ₹1,000 from Account A.
2. Add ₹1,000 to Account B.

Both operations should succeed together. If the deduction succeeds but the addition fails, the transaction should be rolled back so the money is not lost.

## 2. Basic Transaction Commands

| Command | Purpose |
|---|---|
| `BEGIN` / `START TRANSACTION` | Starts a transaction explicitly in supported databases. |
| `COMMIT` | Makes the transaction's changes permanent. |
| `ROLLBACK` | Undoes uncommitted changes in the transaction. |
| `SAVEPOINT` | Creates a point to which a transaction can partially roll back. |

Exact syntax and behavior vary by database. Some systems automatically start transactions, and some statements may cause an implicit commit.

## 3. Example: Bank Transfer

Assume a table named `accounts`.

| account_id | account_name | balance |
|---:|---|---:|
| 101 | Account A | 5000 |
| 102 | Account B | 3000 |

Transfer ₹1,000 from Account A to Account B:

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

After a successful transaction:

| account_id | account_name | balance |
|---:|---|---:|
| 101 | Account A | 4000 |
| 102 | Account B | 4000 |

The total balance remains ₹8,000.

### What if an operation fails?

If an error occurs after the deduction but before the deposit, the application should roll back the transaction rather than commit a partial transfer.

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE account_id = 101;

-- If the transfer cannot continue:
ROLLBACK;
```

`ROLLBACK` undoes the uncommitted deduction.

**Important:** Real applications should check that both account IDs exist, validate the available balance, verify that the expected rows were updated, and handle errors before committing. Transaction syntax alone does not enforce every business rule.

## 4. What Are ACID Properties?

ACID describes four important properties of reliable database transactions:

- **A — Atomicity**
- **C — Consistency**
- **I — Isolation**
- **D — Durability**

<box></box>

### A. Atomicity

**Definition:** All operations in a transaction succeed together, or the transaction's changes are rolled back.

Example: A bank transfer deducts money from one account and credits another. The transaction must not leave only the deduction committed.

Without atomicity, partial updates can leave the database in an unintended state.

**Interview keyword:** All or nothing.

### C. Consistency

**Definition:** A transaction takes the database from one valid state to another, preserving the applicable integrity rules.

Example: If a database enforces that an account balance cannot be negative, a transaction must not leave the balance negative.

Consistency depends on the constraints and rules the system actually enforces. ACID does not automatically create every business rule.

**Interview keyword:** Valid state before and after the transaction.

### I. Isolation

**Definition:** Concurrent transactions should not interfere with one another in ways prohibited by the database's isolation level.

Example: Two customers try to purchase the last available item simultaneously. The system must handle the concurrent updates so that the inventory rules are preserved.

Isolation is controlled by mechanisms such as locks, multiversion concurrency control (MVCC), and transaction isolation levels.

Different isolation levels permit different concurrency behaviors; isolation does not always mean that transactions execute one at a time.

**Interview keyword:** Safe concurrent execution.

### D. Durability

**Definition:** Once a transaction has successfully committed, its changes should survive subsequent failures covered by the database's durability guarantees.

Example: After a bank transfer is committed, a server restart should not cause the committed transfer to disappear.

Databases use mechanisms such as transaction logs and recovery procedures to provide durability.

**Interview keyword:** Committed changes persist.

## 5. ACID Summary Table

| Property | Meaning | Example |
|---|---|---|
| Atomicity | All or nothing | Both sides of a transfer succeed together. |
| Consistency | Preserve database rules | A balance constraint remains satisfied. |
| Isolation | Control interference between transactions | Concurrent withdrawals are handled safely. |
| Durability | Committed changes persist | A committed transfer survives a restart. |

**Easy way to remember:** Atomicity = together; Consistency = valid; Isolation = independent; Durability = permanent.

## 6. What Happens Without Proper Transaction Handling?

Suppose Account A has ₹5,000 and Account B has ₹3,000.

The application deducts ₹1,000 from A, but the database connection fails before B receives the money.

If the deduction has already been committed separately, the balances may become:

- Account A: ₹4,000
- Account B: ₹3,000

The total is now ₹7,000 instead of ₹8,000.

Using a correctly managed transaction allows the database to roll back the uncommitted deduction if the transfer fails.

## 7. COMMIT vs. ROLLBACK

| COMMIT | ROLLBACK |
|---|---|
| Finalizes the transaction's changes. | Undoes uncommitted changes. |
| Makes changes durable according to the database's guarantees. | Returns the affected data to its prior transactional state. |
| Used when the transaction succeeds. | Used when the transaction fails or must be cancelled. |

Example:

```sql
BEGIN;

UPDATE employees
SET salary = salary + 5000
WHERE employee_id = 101;

COMMIT;
```

To cancel the uncommitted update, use `ROLLBACK` instead of `COMMIT`.

After a successful commit, a normal rollback cannot undo that committed transaction. A new compensating transaction may be required.

## 8. SAVEPOINT

A savepoint allows a transaction to roll back part of its work while keeping earlier work pending.

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE account_id = 101;

SAVEPOINT after_deduction;

UPDATE accounts
SET balance = balance + 1000
WHERE account_id = 102;

-- Undo changes made after the savepoint:
ROLLBACK TO SAVEPOINT after_deduction;

-- Complete the transaction:
ROLLBACK;
```

In this example, the final `ROLLBACK` undoes the remaining uncommitted deduction as well.

Savepoint syntax and support vary by database. A savepoint is not the same as a commit: it does not make the transaction's changes permanently committed.

## 9. Transaction Isolation Levels

Isolation levels determine which effects of concurrent transactions one transaction can observe.

The four standard SQL isolation levels are:

| Isolation level | General behavior |
|---|---|
| Read Uncommitted | May allow dirty reads, depending on the database. |
| Read Committed | Prevents dirty reads. |
| Repeatable Read | Prevents dirty reads and non-repeatable reads; phantom behavior depends on the standard and implementation. |
| Serializable | Provides behavior equivalent to serial execution of transactions, subject to successful completion and retry handling. |

### Common concurrency problems

- **Dirty read:** A transaction reads another transaction's uncommitted changes.
- **Non-repeatable read:** Reading the same row twice returns different committed values because another transaction updated it.
- **Phantom read:** Repeating a range query returns a different set of rows because another transaction inserted or deleted matching rows.
- **Lost update:** Concurrent changes overwrite one another, causing an update to be lost.

Actual behavior depends on the database engine and its concurrency-control implementation. For example, some databases implement `READ UNCOMMITTED` similarly to `READ COMMITTED`, and snapshot-based implementations can behave differently from lock-based ones.

## 10. Transactions in Data Engineering

Transactions matter in data engineering because data pipelines often insert, update, delete, merge, or replace large sets of records.

Common uses include:

- Maintaining consistency across related database updates.
- Preventing partially completed batches from being treated as successful.
- Updating warehouse tables safely.
- Managing concurrent writes.
- Recovering from failures during data processing.

### Example: Updating a reporting table

```sql
BEGIN;

DELETE FROM daily_sales
WHERE sales_date = DATE '2026-10-27';

INSERT INTO daily_sales (
    sales_date,
    total_sales
)
SELECT
    order_date,
    SUM(amount)
FROM orders
WHERE order_date = DATE '2026-10-27'
GROUP BY order_date;

COMMIT;
```

The intent is to replace one day's summary within a transaction so readers do not see a committed delete without the corresponding insert.

The exact date-literal syntax and transaction support depend on the database. Some data warehouses or distributed table formats have different transaction boundaries and guarantees.

## 11. ACID in Traditional Databases vs. Data Lakes

Traditional relational databases commonly provide transactions for supported table operations.

In data engineering, platforms such as Delta Lake can also provide ACID transactions for supported tables and operations. This helps with concurrent reads and writes, reliable updates, and recovery from failed operations.

However, **ACID guarantees are not automatically provided for every file or storage operation**. They depend on the table format, storage system, and operation being used.

For example:

- Writing a collection of ordinary CSV files to object storage does not inherently make the entire write an atomic transaction.
- Writing to a supported Delta table uses Delta Lake's transaction mechanism.
- A transaction on one table does not automatically make a multi-table pipeline atomic.

This distinction is important when designing reliable ETL and ELT pipelines.

## 12. Common Interview Questions

**Q1. What is a transaction?**

A transaction is a group of database operations treated as one logical unit of work.

**Q2. What does ACID stand for?**

Atomicity, Consistency, Isolation, and Durability.

**Q3. What is the difference between atomicity and consistency?**

Atomicity concerns whether the transaction's operations succeed or roll back together. Consistency concerns whether the transaction preserves the database's enforced rules and valid states.

**Q4. What is the difference between isolation and durability?**

Isolation controls how concurrent transactions interact. Durability ensures committed changes persist under the database's guarantees.

**Q5. What is the difference between COMMIT and ROLLBACK?**

`COMMIT` finalizes the transaction's changes, whereas `ROLLBACK` undoes uncommitted changes.

**Q6. Does ACID mean transactions cannot fail?**

No. Transactions can fail. ACID properties describe how the database manages transaction behavior and recovery.

**Q7. Does ACID guarantee that a business operation is correct?**

Not by itself. The application and database must implement the correct business rules and constraints.

**Q8. Why are transactions important in data pipelines?**

They help prevent partially applied updates, preserve data integrity, and manage concurrent changes.

**Q9. Are ordinary CSV writes automatically ACID-compliant?**

No. Ordinary file writes do not inherently provide table-level ACID guarantees. Those guarantees require appropriate transactional support.

**Q10. Is a successful COMMIT always permanent even if the server crashes?**

A successful commit should survive failures covered by the database's durability guarantees. Exact guarantees depend on the database configuration and system design.

## 13. Common Mistakes

- Confusing atomicity with consistency.
- Assuming ACID means a transaction can never fail.
- Assuming all transactions run sequentially.
- Assuming `ROLLBACK` can undo a transaction after it has committed.
- Believing a savepoint commits earlier changes.
- Assuming every SQL database implements isolation levels identically.
- Assuming a multi-table data pipeline is automatically one atomic transaction.
- Assuming ordinary files in cloud storage provide ACID guarantees.
- Forgetting to handle errors and verify affected rows before committing.

## 14. Interview Practice

1. Explain all four ACID properties using a bank-transfer example.
2. What happens when the first update succeeds but the second update fails?
3. Explain the difference between `COMMIT`, `ROLLBACK`, and `SAVEPOINT`.
4. Differentiate dirty reads, non-repeatable reads, and phantom reads.
5. Why is transaction handling important when refreshing a data warehouse table?
6. How do ACID transactions in Delta Lake help data engineers?
7. Does a transaction on one table guarantee atomicity across an entire pipeline?
8. Explain how consistency differs from application-level business validation.

## Quick Revision

- **Transaction:** A group of operations treated as one unit.
- **Atomicity:** All or nothing.
- **Consistency:** Preserve enforced integrity rules.
- **Isolation:** Manage concurrent transactions safely.
- **Durability:** Committed changes persist.
- **COMMIT:** Finalize changes.
- **ROLLBACK:** Undo uncommitted changes.
- **SAVEPOINT:** Mark a point for partial rollback.
- **Isolation levels:** Define permitted interactions between concurrent transactions.
- **Data engineering:** Transactions help make database updates and supported lakehouse table operations reliable.

**Core interview takeaway:** ACID makes transaction behavior more reliable, but the actual guarantees depend on the database, its configuration, and the scope of the transaction.
