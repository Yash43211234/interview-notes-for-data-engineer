# Common SQL Performance Problems

## 1. What Is SQL Query Performance?

SQL query performance describes how efficiently a database executes a query in terms of execution time, resource usage, and workload impact.

A slow query may consume excessive CPU, memory, disk I/O, or network resources.

As a Data Engineer, you should be able to identify common performance problems, understand their causes, and choose appropriate optimization techniques.

## 2. Common SQL Performance Problems

The most common problems include:

1. Missing or ineffective indexes.
2. Retrieving unnecessary columns or rows.
3. Inefficient joins.
4. Filtering data after unnecessary processing.
5. Using functions on indexed columns.
6. Expensive subqueries and correlated subqueries.
7. Sorting and grouping large datasets.
8. Repeated scans of the same data.
9. Leading-wildcard searches.
10. Poor data types and implicit conversions.
11. Unnecessary `DISTINCT`.
12. Blocking, locking, and long-running transactions.
13. Poor execution plans.
14. Skewed data distribution and outdated statistics.

## 3. Missing Indexes

### Problem

A query frequently searches or joins on a column that has no suitable index.

```sql
SELECT employee_id, employee_name
FROM employees
WHERE department_id = 10;
```

Without a suitable index, the database may scan many or all rows in the table.

### Possible solution

```sql
CREATE INDEX idx_employees_department
ON employees (department_id);
```

This index may help queries that filter by `department_id`.

### Important points

- Indexes can speed up suitable reads.
- Indexes consume storage.
- Indexes can slow down inserts, updates, and deletes because they must be maintained.
- An index does not guarantee that the optimizer will use it.
- The right index depends on the query, data distribution, table size, and database engine.

**Interview takeaway:** Do not create indexes blindly. Inspect the query plan and workload first.

## 4. Selecting Unnecessary Columns

### Problem

Using `SELECT *` retrieves every column, even when the query needs only a few.

```sql
SELECT *
FROM employees
WHERE department_id = 10;
```

This can increase disk reads, network transfer, memory use, and processing costs, especially when the table contains large text or binary columns.

### Better approach

```sql
SELECT employee_id, employee_name
FROM employees
WHERE department_id = 10;
```

Selecting only the required columns can reduce the amount of data processed and transferred.

It may also enable an index-only or covering-index plan in databases that support it.

## 5. Retrieving Too Many Rows

### Problem

A query returns a large number of rows when only a subset is required.

```sql
SELECT employee_id, employee_name, salary
FROM employees;
```

If the application needs only employees from the IT department, retrieving every employee is unnecessary.

### Better approach

```sql
SELECT employee_id, employee_name, salary
FROM employees
WHERE department = 'IT';
```

For applications that need a limited result set:

```sql
SELECT employee_id, employee_name
FROM employees
ORDER BY employee_id
LIMIT 100;
```

`LIMIT` is supported by databases such as PostgreSQL and MySQL. Other databases may use different syntax.

**Important:** A row limit restricts returned rows but does not always prevent the database from scanning or sorting a large amount of data. An appropriate index and execution plan may still be needed.

## 6. Inefficient JOINs

### Problem

Joining large tables without suitable join keys, or joining before reducing unnecessary data, can increase the amount of work required.

```sql
SELECT
    o.order_id,
    c.customer_name
FROM orders o
JOIN customers c
    ON o.customer_id = c.customer_id;
```

This query may be perfectly efficient with suitable table sizes, statistics, and join strategies. The presence of a join alone is not a performance problem.

### Possible improvements

- Ensure join conditions use the correct columns.
- Check whether join-key data types match.
- Index frequently used join columns when appropriate.
- Filter data early when it reduces the amount of work.
- Avoid accidental many-to-many joins.
- Check whether the join returns more rows than expected.

Example of reducing the input:

```sql
SELECT
    o.order_id,
    c.customer_name
FROM (
    SELECT order_id, customer_id
    FROM orders
    WHERE order_date >= DATE '2026-01-01'
) o
JOIN customers c
    ON o.customer_id = c.customer_id;
```

The optimizer may already push the filter down automatically. Writing a subquery does not guarantee better performance; verify the execution plan.

## 7. Functions on Filtered Columns

### Problem

Applying a function to a column in a filter may prevent the optimizer from using a conventional index efficiently.

```sql
SELECT *
FROM orders
WHERE YEAR(order_date) = 2026;
```

The exact behavior depends on the database and whether it supports a suitable expression or function-based index.

### Better approach

Use a range condition where appropriate:

```sql
SELECT *
FROM orders
WHERE order_date >= DATE '2026-01-01'
  AND order_date <  DATE '2027-01-01';
```

This half-open date range includes all timestamps in 2026 without including the start of 2027.

**Interview takeaway:** Sargable predicates—conditions that can efficiently use an index or access path—often improve performance.

## 8. Leading-Wildcard Searches

### Problem

A search pattern beginning with `%` often prevents a conventional B-tree index from efficiently locating matching text.

```sql
SELECT *
FROM customers
WHERE customer_name LIKE '%sh%';
```

The database may need to examine many rows.

### Alternative when appropriate

```sql
SELECT *
FROM customers
WHERE customer_name LIKE 'Sha%';
```

A prefix search may be able to use a suitable index, depending on the database, collation, and query plan.

For substring searches, consider database-specific full-text search, trigram indexes, or specialized search systems.

## 9. Expensive Sorting and GROUP BY

### Problem

Sorting or grouping a large dataset may require significant CPU and memory. If available memory is insufficient, the database may spill intermediate data to disk.

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department
ORDER BY employee_count DESC;
```

### Possible improvements

- Filter unnecessary rows before aggregation when the logic permits.
- Return only required columns.
- Use appropriate indexes when they help the query plan.
- Check for expensive sorts and disk spills in the execution plan.
- Avoid sorting results when ordering is not required.

**Important:** Indexes do not eliminate every sort or aggregation cost. The database may choose hashing, sorting, or another strategy based on the query and available resources.

## 10. Correlated Subqueries

### Problem

A correlated subquery references columns from the outer query.

```sql
SELECT
    e.employee_id,
    e.employee_name
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department = e.department
);
```

Depending on the optimizer, the inner calculation may be repeated or transformed into a more efficient plan.

### Alternative using a window function

```sql
SELECT employee_id, employee_name, salary, department
FROM (
    SELECT
        employee_id,
        employee_name,
        salary,
        department,
        AVG(salary) OVER (
            PARTITION BY department
        ) AS department_avg
    FROM employees
) e
WHERE salary > department_avg;
```

This calculates the department average as a window value.

The window-function version is not automatically faster in every database. Compare execution plans and ensure both versions implement the same business logic.

## 11. Unnecessary DISTINCT

### Problem

`DISTINCT` removes duplicate output rows, which may require sorting or hashing.

```sql
SELECT DISTINCT department
FROM employees;
```

If duplicate rows are possible and the output must contain unique departments, `DISTINCT` is appropriate.

However, adding `DISTINCT` to hide duplicate rows caused by an incorrect join can waste resources and conceal a data-modeling or join-condition problem.

### Better approach

- Use `DISTINCT` only when unique output rows are required.
- Check join conditions when duplicates appear unexpectedly.
- Use `EXISTS` when the requirement is only to test whether a matching row exists.

Example:

```sql
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

This returns customers with at least one order without producing one output row per matching order.

## 12. Implicit Data-Type Conversions

### Problem

Comparing columns with incompatible data types can trigger implicit conversions and may prevent efficient index usage.

For example, if `customer_id` is an integer:

```sql
SELECT *
FROM customers
WHERE customer_id = '1001';
```

Some databases automatically convert the string literal to an integer. Others may apply conversions differently, depending on the types involved.

### Better approach

```sql
SELECT *
FROM customers
WHERE customer_id = 1001;
```

Use values with compatible data types and check the execution plan if conversions appear.

**Interview takeaway:** Implicit conversion is not always expensive, but conversion on the indexed column can be problematic in some engines.

## 13. Repeated Scans of the Same Data

### Problem

A query may read the same large table multiple times for related calculations.

For example, separate subqueries may calculate total sales and order counts independently.

### Possible improvement

Combine related aggregations when the logic allows:

```sql
SELECT
    customer_id,
    COUNT(*) AS order_count,
    SUM(amount) AS total_sales,
    AVG(amount) AS average_order_value
FROM orders
GROUP BY customer_id;
```

This computes multiple metrics in one aggregation.

A CTE does not necessarily prevent repeated scans or materialize results. Whether work is shared depends on the database and execution plan.

## 14. Poor Execution Plans and Outdated Statistics

### Problem

The optimizer estimates the cost of different execution strategies using information such as table statistics, row counts, and data distribution.

When these estimates are inaccurate, the chosen plan may be inefficient.

### How to investigate

```sql
EXPLAIN
SELECT employee_id, employee_name
FROM employees
WHERE department_id = 10;
```

Depending on the database, `EXPLAIN ANALYZE` or an equivalent command can show actual execution information.

Look for:

- Full table scans when an alternative access path might help.
- Unexpectedly high row counts.
- Large differences between estimated and actual rows.
- Expensive joins, sorts, or aggregations.
- Repeated scans.
- Disk spills or excessive memory use.

A full table scan is not inherently bad. It may be the best plan when a large proportion of the table must be read.

Refresh statistics using the database's supported mechanism when evidence suggests that stale statistics are affecting plan selection.

## 15. Blocking, Locks, and Long Transactions

### Problem

A query can be slow because it is waiting for another transaction rather than spending time processing data.

Long-running transactions can hold locks or keep old row versions active, depending on the database's concurrency-control model.

### Possible improvements

- Keep transactions as short as practical.
- Commit or roll back promptly.
- Investigate blocking sessions.
- Use an appropriate isolation level.
- Avoid unnecessary updates to large numbers of rows.
- Monitor lock waits and deadlocks.

Do not lower isolation levels or remove transactional safeguards simply to improve speed without evaluating correctness and concurrency requirements.

## 16. Performance Problems in Data Engineering Pipelines

SQL performance problems also occur in analytical workloads and distributed data systems.

| Problem | Possible improvement |
|---|---|
| Reading unnecessary columns | Use column pruning. |
| Reading too much data | Apply effective filters and partition pruning. |
| Too many small files | Compact files where appropriate. |
| Data skew | Reconsider partitioning, joins, and skew-handling strategies. |
| Repeated full-table processing | Consider incremental processing when correctness permits. |
| Large shuffle operations | Reduce data before shuffling; inspect the execution plan. |
| Poor partition design | Align partitions with common filters and workload patterns. |
| Repeated transformations | Reuse computed data when beneficial and supported. |

These techniques are especially relevant to Spark, Databricks, and lakehouse workloads. They complement SQL query optimization but are not identical to relational database indexing.

## 17. A Practical SQL Optimization Workflow

When a query is slow, follow this sequence:

1. **Understand the requirement:** Confirm the correct result and business logic.
2. **Measure the problem:** Record execution time and the amount of data processed.
3. **Inspect the execution plan:** Identify expensive scans, joins, sorts, and aggregations.
4. **Check filters and joins:** Look for unnecessary data processing, incorrect joins, and type conversions.
5. **Review indexes and statistics:** Determine whether an appropriate access path is available.
6. **Rewrite only when justified:** Test alternatives such as range filters, `EXISTS`, or consolidated aggregations.
7. **Measure again:** Compare actual runtime and resource use using comparable conditions.
8. **Validate correctness:** Ensure the optimized query returns the intended results, including edge cases.

Do not assume that a shorter query or a query with a CTE is necessarily faster.

## 18. Common Interview Questions

**Q1. What are the most common causes of slow SQL queries?**

Missing or unsuitable indexes, excessive data retrieval, inefficient joins, expensive sorting and aggregation, poor execution plans, data-type conversions, and lock waits are common causes.

**Q2. How do you optimize a slow SQL query?**

Start by measuring the query and inspecting its execution plan. Identify the dominant cost, make a targeted change, and compare performance and correctness.

**Q3. Why is `SELECT *` sometimes inefficient?**

It can retrieve unnecessary columns, increase data transfer, and consume more memory and I/O.

**Q4. Does creating an index always improve performance?**

No. Indexes consume storage and add write overhead. The optimizer may prefer a scan, particularly when a query reads a large percentage of a table.

**Q5. Why can a function in WHERE affect performance?**

Applying a function to a filtered column can make a conventional index less useful. A range predicate or suitable expression index may help.

**Q6. Why can a leading wildcard be slow?**

A conventional B-tree index often cannot efficiently locate the beginning of a pattern such as `'%abc'`.

**Q7. Is a full table scan always bad?**

No. A full scan can be efficient when many rows are needed or the table is small.

**Q8. How do you find the bottleneck in a query?**

Inspect the execution plan, actual row counts, I/O, CPU, memory, sort or shuffle costs, and wait information available in the database.

**Q9. Does replacing a subquery with a JOIN always improve performance?**

No. The optimizer may transform both forms similarly, and their semantics may differ. Compare execution plans and validate results.

**Q10. What is the difference between SQL tuning and Spark optimization?**

SQL tuning often focuses on access paths, joins, indexes, statistics, and execution plans. Spark optimization also considers distributed execution, shuffles, partitioning, skew, and file layout.

## 19. Common Mistakes

- Creating indexes without inspecting the workload.
- Assuming every full table scan is inefficient.
- Assuming a CTE or subquery automatically makes a query faster.
- Using `DISTINCT` to hide duplicate rows caused by an incorrect join.
- Rewriting a query without checking whether the results remain equivalent.
- Optimizing execution time without measuring resource usage.
- Ignoring lock waits and concurrency.
- Assuming an index can solve data skew or excessive distributed shuffling.
- Changing isolation or consistency safeguards without assessing correctness.

## 20. Interview Practice

1. A query filters one million employee records by department. How would you investigate its performance?
2. Why might `WHERE YEAR(order_date) = 2026` be slower than a date-range filter?
3. When is a full table scan better than an index scan?
4. How would you investigate a query that spends most of its time sorting?
5. How can an incorrect join create duplicate rows and unnecessary processing?
6. How would you identify whether a query is slow because of blocking?
7. How would you optimize a Spark job that spends most of its time shuffling data?
8. How would you prove that a rewritten query is both faster and correct?

## Quick Revision

- **Indexes:** Can accelerate suitable lookups but add storage and write overhead.
- **Column pruning:** Avoid retrieving unnecessary columns.
- **Filter early:** Reduce data before expensive processing when the logic permits.
- **Join correctly:** Verify keys, data types, cardinality, and join conditions.
- **Sargable predicates:** Write filters that allow efficient access paths where possible.
- **Execution plans:** Identify the actual bottleneck before changing SQL.
- **Statistics:** Help the optimizer estimate costs and choose plans.
- **Transactions and locks:** Investigate waiting as well as query processing.
- **Spark optimization:** Consider partition pruning, shuffles, skew, and file layout.
- **Measure and validate:** Compare runtime, resource use, and result correctness.

**Core interview takeaway:** Effective SQL optimization is a measurement-driven process: identify the bottleneck, make a targeted change, inspect the new plan, and verify that the result remains correct.
