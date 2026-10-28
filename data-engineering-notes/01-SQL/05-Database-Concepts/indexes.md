# Indexes in SQL

## 1. What Is an Index?

An index is a database structure that helps the database find rows faster without scanning the entire table in many cases.

Think of an index like the index at the back of a book. Instead of reading every page to find a topic, you use the index to locate it quickly.

Indexes are commonly used to improve:
- Filtering with `WHERE`
- Joining tables
- Sorting with `ORDER BY`
- Searching for specific values
- Enforcing uniqueness through unique indexes

**Key point:** Indexes can improve read performance, but they require additional storage and can slow down inserts, updates, and deletes.

## 2. Example Table

Assume a table named `employees`.

| employee_id | employee_name | department | salary |
|---:|---|---|---:|
| 101 | Amit | IT | 90000 |
| 102 | Priya | HR | 70000 |
| 103 | Rahul | IT | 85000 |
| 104 | Neha | Sales | 65000 |
| 105 | Ravi | HR | 75000 |

Suppose the table contains millions of rows and the following query runs frequently:

```sql
SELECT *
FROM employees
WHERE employee_id = 103;
```

Without a useful index, the database might scan many or all rows to find the employee. With a suitable index, it may locate the matching row much more efficiently.

The actual execution plan depends on the database engine, table size, data distribution, and query.

## 3. Create an Index

```sql
CREATE INDEX idx_employees_department
ON employees (department);
```

This creates an index on the `department` column.

The following query may benefit from it:

```sql
SELECT *
FROM employees
WHERE department = 'IT';
```

### Syntax

```sql
CREATE INDEX index_name
ON table_name (column_name);
```

To remove an index, the syntax depends on the database engine. For example, PostgreSQL uses:

```sql
DROP INDEX idx_employees_department;
```

## 4. Common Types of Indexes

### 4.1 B-tree Index

B-tree indexes are widely used in relational databases and support equality searches, range conditions, and ordered retrieval.

Examples:

```sql
CREATE INDEX idx_employees_salary
ON employees (salary);
```

Queries that may benefit include:

```sql
SELECT *
FROM employees
WHERE salary > 80000;
```

```sql
SELECT *
FROM employees
ORDER BY salary;
```

An index does not guarantee that the database will use it; the optimizer chooses the execution plan.

### 4.2 Unique Index

A unique index prevents duplicate values in the indexed key.

```sql
CREATE UNIQUE INDEX idx_employees_email
ON employees (email);
```

This is useful when email addresses must be unique.

**Note:** Handling of `NULL` values in unique indexes varies by database engine. Also, a unique index on a nullable column does not necessarily prohibit multiple `NULL` values.

### 4.3 Composite Index

A composite index contains multiple columns.

```sql
CREATE INDEX idx_department_salary
ON employees (department, salary);
```

It may help queries such as:

```sql
SELECT *
FROM employees
WHERE department = 'IT'
  AND salary > 80000;
```

For a B-tree index, column order matters. An index on `(department, salary)` is generally most useful when queries filter on `department`, or on `department` and `salary` together.

It may be less useful for queries filtering only on `salary`.

### 4.4 Clustered Index

A clustered index determines how table data is organized or stored according to the database engine's implementation.

For example, SQL Server supports a clustered index that organizes the table's data rows by the index key. A table can have only one clustered index.

**Important:** Clustered-index behavior is database-specific. Do not assume every database engine implements clustered indexes in the same way.

### 4.5 Nonclustered Index

A nonclustered index stores index keys and references that allow the database to locate the corresponding rows.

SQL Server supports both clustered and nonclustered indexes. Their exact implementation and relationship to table storage depend on the database engine.

## 5. How Does an Index Work?

A common index structure is a B-tree or B+ tree.

Conceptually, it organizes keys in a searchable hierarchy:

1. The database starts at the root of the index.
2. It follows the appropriate branch based on the search key.
3. It reaches the relevant leaf entries.
4. It retrieves matching rows or obtains the needed values directly from the index.

This can reduce the amount of data the database needs to examine compared with a full table scan.

However, if a query returns a large percentage of the table, a table scan may be cheaper than using an index.

## 6. Index vs. Full Table Scan

| Feature | Index access | Full table scan |
|---|---|---|
| Data examined | Often fewer rows or index entries | Usually reads the table's rows |
| Selective filters | Often beneficial | May do unnecessary work |
| Queries returning most rows | May be less efficient | Can be more efficient |
| Storage | Requires extra index storage | No additional index required |
| Write operations | Indexes must be maintained | No index maintenance for that absent index |

The database optimizer decides which access method is appropriate.

## 7. Composite Index and Column Order

Consider this index:

```sql
CREATE INDEX idx_orders_customer_date
ON orders (customer_id, order_date);
```

It may support:

```sql
SELECT *
FROM orders
WHERE customer_id = 101;
```

It may also support:

```sql
SELECT *
FROM orders
WHERE customer_id = 101
  AND order_date >= '2026-01-01';
```

But this query may not benefit as much from the same index:

```sql
SELECT *
FROM orders
WHERE order_date >= '2026-01-01';
```

For a conventional B-tree index, the leading column is important because the index is organized by the key sequence. Some engines support additional access methods or optimizations, so the execution plan should be checked.

**Interview rule:** Design composite indexes around common query patterns, filter conditions, join keys, and sorting requirements—not just the columns that appear most often.

## 8. Why Too Many Indexes Can Be a Problem

Every additional index has costs.

- **Storage:** Indexes consume disk space and may use memory.
- **Insert overhead:** New index entries must be created.
- **Update overhead:** Indexes involving changed columns may need maintenance.
- **Delete overhead:** Corresponding index entries must be removed.
- **Maintenance:** Indexes may require rebuilding, reorganizing, or other engine-specific maintenance.

An index that is rarely used may add cost without delivering meaningful benefits.

## 9. Indexes and JOINs

Consider two tables: `orders` and `customers`.

```sql
SELECT
    o.order_id,
    c.customer_name
FROM orders o
JOIN customers c
    ON o.customer_id = c.customer_id;
```

Indexes on join columns can help the database locate matching rows efficiently, depending on the execution plan.

For example:

```sql
CREATE INDEX idx_orders_customer_id
ON orders (customer_id);
```

A primary key or unique constraint on `customers.customer_id` typically provides an appropriate index in many relational database systems.

**Important:** Indexing both join columns is not automatically necessary. Consider existing indexes, table size, join strategy, and workload.

## 10. Indexes and Query Performance

Use the execution plan to understand whether an index is useful.

For PostgreSQL:

```sql
EXPLAIN
SELECT *
FROM employees
WHERE department = 'IT';
```

To execute the query and collect actual runtime statistics:

```sql
EXPLAIN ANALYZE
SELECT *
FROM employees
WHERE department = 'IT';
```

`EXPLAIN ANALYZE` actually executes the statement, so use caution with queries that modify data or have expensive side effects.

Look for:
- Sequential scans versus index scans
- Estimated versus actual row counts
- Rows examined and rows returned
- Join strategy
- Execution time and, where available, buffer activity

Do not assume an index is beneficial merely because it appears in the execution plan. Compare the overall cost and measured performance.

## 11. Common Interview Questions

**Q1. What is an index in SQL?**

An index is a data structure that helps the database locate rows efficiently, often reducing the need for a full table scan.

**Q2. Do indexes always improve performance?**

No. They can improve read queries but add storage requirements and write overhead. A table scan may be better for queries that retrieve many rows.

**Q3. What is the difference between a single-column and composite index?**

A single-column index contains one key column. A composite index contains multiple key columns whose order affects which queries it can support efficiently.

**Q4. What is the difference between a primary key and an index?**

A primary key is a logical constraint that uniquely identifies rows and does not permit `NULL` values. An index is a physical database structure used to support data access or enforce constraints. Many databases implement primary keys using a unique index, but the concepts are not identical.

**Q5. Can an index improve a LIKE query?**

Sometimes. A B-tree index may support a prefix search such as `LIKE 'A%'`, depending on the database, collation, and index definition. A leading-wildcard search such as `LIKE '%A'` generally cannot use an ordinary B-tree index for a direct prefix lookup.

**Q6. Why does column order matter in a composite index?**

The index is organized according to its key sequence. Leading columns strongly influence which equality, range, and ordering conditions can use the index efficiently.

**Q7. What is a covering index?**

A covering index contains all the columns a particular query needs, allowing the database to satisfy the query from the index without retrieving the corresponding table rows in some execution plans.

## 12. Common Mistakes

- Creating indexes on every column.
- Assuming every query will use an available index.
- Ignoring the order of columns in a composite index.
- Forgetting the additional cost to writes.
- Using functions or transformations on indexed columns without considering their effect on index usage.
- Ignoring execution plans and actual query performance.
- Confusing a primary key constraint with an index.
- Assuming clustered-index behavior is identical across database engines.

## 13. Interview Practice

1. Create an index on `employee_id`.
2. Create a composite index on `department` and `salary`.
3. Explain why an index on `(customer_id, order_date)` may not efficiently support a query filtering only by `order_date`.
4. Compare a query's execution plan before and after creating an index.
5. Explain why an index can slow down bulk inserts.
6. Identify which columns might benefit from indexes in a large orders table.
7. Explain when a full table scan can be more efficient than an index scan.

## Quick Revision

- **Index:** Helps locate rows efficiently.
- **B-tree:** Common index structure for equality and range searches.
- **Unique index:** Enforces uniqueness of the indexed key.
- **Composite index:** Uses multiple columns; key order matters.
- **Clustered index:** Organizes table data according to engine-specific behavior.
- **Covering index:** May satisfy a query directly from the index.
- **Main trade-off:** Faster eligible reads versus additional storage and write-maintenance costs.
- **Performance verification:** Use execution plans and representative workload measurements.

**Core interview takeaway:** Create indexes based on actual query patterns, understand composite-key order, and verify improvements using execution plans rather than assuming every index makes a query faster.
