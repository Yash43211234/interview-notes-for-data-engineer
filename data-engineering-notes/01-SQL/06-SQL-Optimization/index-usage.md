# SQL Index Usage

## 1. What Is an Index?

An **index** is a database structure that helps the database find rows without necessarily scanning the entire table.

Think of an index in a book: instead of reading every page to find a topic, you use the index to locate the relevant pages.

In a database, an index stores searchable key values and information that helps locate the corresponding rows.

Example table: `employees`

| employee_id | employee_name | department_id | salary |
|---:|---|---:|---:|
| 101 | Amit | 10 | 90000 |
| 102 | Priya | 20 | 85000 |
| 103 | Rahul | 10 | 70000 |
| 104 | Neha | 30 | 65000 |
| 105 | Ravi | 20 | 75000 |

Without a suitable index, a query filtering by `department_id` may need to scan the table.

```sql
SELECT employee_id, employee_name
FROM employees
WHERE department_id = 10;
```

An index on `department_id` may help the database locate matching employees more efficiently.

## 2. Creating an Index

Basic syntax:

```sql
CREATE INDEX index_name
ON table_name (column_name);
```

Example:

```sql
CREATE INDEX idx_employees_department
ON employees (department_id);
```

This creates an index on `department_id`.

The index may improve queries that filter, join, or otherwise access rows using this column.

**Important:** Creating an index does not guarantee that the database will use it. The optimizer chooses an execution plan based on its estimates, the query, available indexes, and other factors.

## 3. When Does a Database Use an Index?

Indexes are commonly useful for the following operations.

### A. Equality filtering

```sql
SELECT *
FROM employees
WHERE employee_id = 101;
```

An index on `employee_id` may help locate the matching row.

### B. Range filtering

```sql
SELECT *
FROM employees
WHERE salary BETWEEN 60000 AND 85000;
```

An appropriate index on `salary` may help locate rows within the specified range.

### C. JOIN conditions

```sql
SELECT
    o.order_id,
    c.customer_name
FROM orders o
JOIN customers c
    ON o.customer_id = c.customer_id;
```

An index on a frequently used join key can help some join strategies, depending on the query plan and table sizes.

### D. Sorting

```sql
SELECT employee_id, salary
FROM employees
ORDER BY salary DESC;
```

An index on `salary` may help provide rows in the required order, depending on the database and index definition.

### E. GROUP BY

```sql
SELECT department_id, COUNT(*)
FROM employees
GROUP BY department_id;
```

An index on `department_id` may help some execution strategies, but the database may still choose a table scan and hash aggregation.

## 4. When Might a Database Not Use an Index?

Having an index does not mean it will be selected for every query.

Common reasons include:

- The query needs a large percentage of the table.
- The table is small enough that scanning it is cheaper.
- The filter is not selective.
- The query applies a function or conversion that makes the index less useful.
- The query uses a pattern that a conventional index cannot efficiently support.
- The index does not match the query's filtering or ordering requirements.
- Statistics or cost estimates favor another execution plan.

The correct approach is to inspect the query plan rather than assume an index is being used.

## 5. Selectivity and Index Usage

**Selectivity** describes how effectively a condition narrows down the rows being considered. In index discussions, a highly selective condition typically matches a small proportion of the table.

Suppose a table contains one million employees.

| Filter | Matching rows | Typical selectivity |
|---|---:|---|
| `employee_id = 101` | 1 | High |
| `department_id = 10` | 200,000 | Lower |
| `country = 'India'` | 900,000 | Very low |

An index on `employee_id` may be highly effective for finding one employee.

An index on `country` may be less beneficial for a query returning 90% of the table.

However, selectivity alone does not determine the plan. Row size, clustering, caching, available indexes, and other factors also matter.

## 6. Composite Indexes

A **composite index** is an index built on multiple columns.

Example:

```sql
CREATE INDEX idx_orders_customer_date
ON orders (customer_id, order_date);
```

This index contains both `customer_id` and `order_date`, in the specified order.

It may be useful for queries such as:

```sql
SELECT order_id, order_date
FROM orders
WHERE customer_id = 101;
```

It may also help with:

```sql
SELECT order_id, order_date
FROM orders
WHERE customer_id = 101
  AND order_date >= DATE '2026-01-01';
```

### Why does column order matter?

For a conventional B-tree composite index, the leading columns are particularly important.

An index on `(customer_id, order_date)` commonly supports lookups by:

- `customer_id`
- `customer_id` and `order_date`

It may not be as effective for a query filtering only on `order_date`.

For example:

```sql
SELECT order_id
FROM orders
WHERE order_date >= DATE '2026-01-01';
```

A separate index on `order_date` may be useful for this workload.

**Interview takeaway:** Design composite indexes around actual query patterns, including filters, join conditions, sorting, and the order of the indexed columns.

## 7. Covering Indexes

A **covering index** contains all the columns needed by a particular query, so the database may be able to answer the query using the index without fetching additional table rows.

Suppose the query is:

```sql
SELECT employee_id, employee_name
FROM employees
WHERE department_id = 10;
```

A candidate index in a database that supports suitable covering-index designs could include:

- `department_id`
- `employee_id`
- `employee_name`

In PostgreSQL, one possible definition is:

```sql
CREATE INDEX idx_employees_department_cover
ON employees (department_id)
INCLUDE (employee_id, employee_name);
```

`INCLUDE` is supported by PostgreSQL and some other database systems, but syntax and behavior differ by engine.

A covering index can reduce table lookups, but it also increases index storage and maintenance costs. Whether it improves performance should be verified using the query plan.

## 8. Unique Indexes

A **unique index** enforces uniqueness for its indexed key values.

```sql
CREATE UNIQUE INDEX idx_employees_email
ON employees (email);
```

This prevents duplicate non-NULL email values under the database's uniqueness rules. Handling of `NULL` values varies by database.

Unique indexes are useful for enforcing rules such as unique email addresses or account identifiers.

**Note:** A primary key generally creates or uses a unique index in many relational databases, but the exact implementation is database-specific.

## 9. Indexes and the WHERE Clause

Compare these queries.

### Query A: Potentially index-friendly

```sql
SELECT *
FROM orders
WHERE order_date >= DATE '2026-01-01'
  AND order_date < DATE '2027-01-01';
```

A suitable index on `order_date` may support this range condition.

### Query B: Potentially less index-friendly

```sql
SELECT *
FROM orders
WHERE EXTRACT(YEAR FROM order_date) = 2026;
```

Applying a function to the column can make a conventional index less useful.

A database may support an expression index or another optimization, but the range-filter version is often a good starting point.

**General principle:** Prefer predicates that allow efficient access through available indexes, and verify the actual plan.

## 10. LIKE and Index Usage

A conventional B-tree index may help with prefix searches:

```sql
SELECT *
FROM customers
WHERE customer_name LIKE 'Sam%';
```

A leading-wildcard search is often harder to accelerate with a conventional B-tree index:

```sql
SELECT *
FROM customers
WHERE customer_name LIKE '%sam%';
```

For substring searches, some databases support specialized indexes or full-text search.

Index behavior also depends on collation, case sensitivity, and database implementation.

## 11. Indexes on JOIN Columns

Consider these tables:

- `customers(customer_id, customer_name)`
- `orders(order_id, customer_id, amount)`

Query:

```sql
SELECT
    c.customer_name,
    o.order_id,
    o.amount
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id;
```

A primary key or suitable index on `customers.customer_id` commonly supports efficient customer lookups.

An index on `orders.customer_id` may also help, especially for queries that retrieve orders for a small set of customers.

However, the best join strategy depends on the size of each input and the optimizer's estimates. A full scan with a hash join may be more efficient for a query that needs most of both tables.

**Important:** Some databases automatically index primary keys but do not automatically index foreign-key columns. Check your database's behavior.

## 12. Indexes and ORDER BY

Consider:

```sql
SELECT employee_id, salary
FROM employees
ORDER BY salary DESC;
```

A suitable index on `salary` may allow the database to retrieve rows in the required order without a separate sort.

For a composite index:

```sql
CREATE INDEX idx_employees_department_salary
ON employees (department_id, salary);
```

The index may help with:

```sql
SELECT employee_id, salary
FROM employees
WHERE department_id = 10
ORDER BY salary;
```

This is because the filter uses the leading column and the following column provides the ordering within that department.

Direction, null ordering, collation, and engine-specific rules can affect whether an index satisfies an `ORDER BY` clause.

## 13. Indexes and NULL Values

Index behavior for `NULL` values differs across database systems and index types.

Consider:

```sql
SELECT *
FROM employees
WHERE department_id IS NULL;
```

Some indexes can help with this query, while the usefulness depends on the database implementation and index definition.

Do not assume that all indexes omit `NULL` values or that all database engines handle them identically.

## 14. Index Maintenance and Trade-Offs

Indexes improve some read operations, but they also have costs.

| Advantage | Trade-off |
|---|---|
| Faster suitable lookups | Additional storage |
| Can improve joins | Extra work during inserts |
| Can help range queries | Extra work during updates and deletes |
| Can provide ordering | Maintenance and fragmentation considerations in some engines |
| Can support uniqueness | Index creation and maintenance overhead |

Too many indexes can slow down write-heavy workloads.

A good indexing strategy considers the overall workload rather than optimizing one query in isolation.

## 15. How to Check Whether an Index Is Used

Use the database's execution-plan tools.

For example, in PostgreSQL:

```sql
EXPLAIN
SELECT employee_id, employee_name
FROM employees
WHERE department_id = 10;
```

To see actual execution measurements:

```sql
EXPLAIN ANALYZE
SELECT employee_id, employee_name
FROM employees
WHERE department_id = 10;
```

Look for operations such as:

- Sequential scan.
- Index scan.
- Index-only scan.
- Bitmap index scan.
- Actual and estimated row counts.
- Execution time and buffer activity, where available.

The operation names and details vary by database.

**Important:** An index scan is not automatically better than a sequential scan. The correct plan is the one that performs the workload efficiently while returning the required result.

## 16. Index Usage in Data Engineering

Indexing is especially relevant to relational databases and operational data stores. In analytical platforms, optimization may also rely on partitioning, clustering, file statistics, and data layout rather than traditional row-level indexes.

| System or workload | Common optimization techniques |
|---|---|
| PostgreSQL / MySQL | B-tree indexes, composite indexes, execution plans |
| SQL Server | Clustered and nonclustered indexes, included columns, execution plans |
| Data warehouses | Partition pruning, clustering, columnar storage, workload-specific optimizations |
| Spark / Databricks | Partition pruning, file statistics, data skipping, clustering, join optimization |

Do not assume that an index strategy from a traditional relational database applies directly to every lakehouse table.

## 17. Common Interview Questions

**Q1. What is an index?**

A database structure that helps locate rows efficiently without necessarily scanning the entire table.

**Q2. Does an index always improve query performance?**

No. It adds storage and write overhead, and the optimizer may choose a different access path.

**Q3. What is the difference between a single-column and composite index?**

A single-column index indexes one column. A composite index indexes multiple columns in a specified order.

**Q4. Why does the order of columns matter in a composite index?**

The leading columns often determine which filters and ordering requirements the index can support efficiently.

**Q5. What is a covering index?**

An index that contains all columns needed for a particular query, potentially avoiding additional table-row lookups.

**Q6. Why might a database ignore an index?**

The index may be unselective, the table may be small, the query may retrieve many rows, or another plan may have a lower estimated cost.

**Q7. Can an index help a JOIN?**

Yes. Suitable indexes on join keys can improve some join strategies, but the benefit depends on the query and data.

**Q8. Why can a leading wildcard reduce index usefulness?**

A conventional B-tree index generally cannot efficiently seek to an unknown beginning of a substring pattern.

**Q9. What is the difference between a unique index and a primary key?**

A unique index enforces uniqueness for its indexed key values. A primary key is a table constraint that uniquely identifies each row and does not permit `NULL` values.

**Q10. How do you verify index usage?**

Inspect the execution plan and, where supported, compare estimated and actual execution measurements.

## 18. Common Mistakes

- Creating indexes on every column.
- Assuming indexes always make queries faster.
- Ignoring column order in composite indexes.
- Forgetting that indexes add write overhead.
- Assuming foreign-key columns are automatically indexed.
- Using indexes to compensate for incorrect join conditions.
- Assuming every table scan is bad.
- Ignoring the actual execution plan.
- Applying traditional index concepts directly to all distributed data platforms.
- Optimizing one query without considering the broader workload.

## 19. Interview Practice

1. How would you decide whether to create an index on `department_id`?
2. What is the difference between an index scan and a table scan?
3. Why might a composite index on `(customer_id, order_date)` not efficiently support a filter on `order_date` alone?
4. What is a covering index, and what are its trade-offs?
5. How can indexes improve join performance?
6. Why might a database ignore an index even when one exists?
7. How would you investigate a query that filters on a date column?
8. How can too many indexes affect inserts and updates?
9. What is the difference between indexing in a relational database and optimizing a Delta table?
10. How would you validate that a new index improves the workload?

## Quick Revision

- **Index:** Helps locate rows efficiently.
- **B-tree:** A common index structure suitable for equality and range searches.
- **Composite index:** Indexes multiple columns in a specific order.
- **Selectivity:** How effectively a condition narrows down rows.
- **Covering index:** Contains the columns needed by a query.
- **Unique index:** Enforces uniqueness.
- **Index trade-off:** Faster suitable reads, but more storage and write overhead.
- **Execution plan:** Shows whether and how an index is used.
- **Composite index rule:** Leading columns are especially important.
- **Best practice:** Design indexes for real query patterns and verify their impact.

**Core interview takeaway:** Indexes are a tool for improving suitable access patterns, not a universal solution. Understand the query, design the index around the workload, inspect the execution plan, and measure the result.
