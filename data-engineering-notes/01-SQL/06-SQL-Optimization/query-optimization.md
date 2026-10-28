# SQL Query Optimization

## 1. What Is Query Optimization?

**Query optimization** is the process of improving a SQL query's performance while preserving its intended results.

The goal is to reduce unnecessary work, such as reading excessive data, performing inefficient joins, sorting large datasets, or repeatedly calculating the same results.

Query optimization can improve:

- Query execution time.
- CPU and memory usage.
- Disk I/O.
- Network data transfer.
- Database resource consumption.
- Data pipeline processing time.

As a Data Engineer, you should understand both how to write efficient SQL and how to identify performance problems using execution plans.

## 2. The Query Optimization Process

Follow this workflow when a query is slow:

1. Understand the expected output.
2. Measure the current query performance.
3. Inspect the execution plan.
4. Identify the most expensive operation.
5. Check filters, joins, indexes, and data volume.
6. Rewrite the query or adjust the database design if necessary.
7. Compare the new execution plan and runtime.
8. Verify that the results remain correct.

**Important:** Never assume that a shorter query is automatically faster. The database optimizer may execute different SQL formulations using the same physical plan.

## 3. Select Only Required Columns

Avoid retrieving unnecessary columns.

### Less efficient

```sql
SELECT *
FROM employees
WHERE department_id = 10;
```

### Better when only these columns are needed

```sql
SELECT employee_id, employee_name
FROM employees
WHERE department_id = 10;
```

Selecting fewer columns can reduce data retrieval, memory use, and network transfer. It may also allow an index-only or covering-index access path.

This is particularly useful when tables contain large text, binary, or other expensive-to-read columns.

## 4. Filter Data Early

Apply conditions that reduce the input data before expensive operations, when doing so preserves the required logic.

### Example

```sql
SELECT
    customer_id,
    SUM(amount) AS total_sales
FROM orders
WHERE order_date >= DATE '2026-01-01'
  AND order_date < DATE '2027-01-01'
GROUP BY customer_id;
```

The query calculates sales only for orders in 2026.

Filtering before aggregation can reduce the number of rows that need to be grouped.

Modern optimizers may automatically push filters down, so adding a subquery solely to move a filter earlier does not guarantee better performance.

## 5. Use Indexes Appropriately

Indexes can improve suitable lookups, range filters, joins, and ordering operations.

Example:

```sql
CREATE INDEX idx_orders_customer
ON orders (customer_id);
```

This index may help queries such as:

```sql
SELECT order_id, amount
FROM orders
WHERE customer_id = 101;
```

However, indexes have costs:

- Additional storage.
- Extra work during inserts, updates, and deletes.
- Maintenance overhead.
- Potentially limited benefit when a query reads most of the table.

Use execution plans and workload evidence to decide whether an index is appropriate.

## 6. Write Index-Friendly Filter Conditions

A predicate is often called **sargable** when it can use an available index or access path efficiently.

### Potentially less efficient

```sql
SELECT *
FROM orders
WHERE EXTRACT(YEAR FROM order_date) = 2026;
```

Applying a function to the column can make a conventional index less useful.

### Often preferable

```sql
SELECT *
FROM orders
WHERE order_date >= DATE '2026-01-01'
  AND order_date < DATE '2027-01-01';
```

The range condition may allow an index on `order_date` to be used efficiently.

The exact benefit depends on the database, index definition, and execution plan. Some databases support expression indexes that can help with the first formulation.

## 7. Choose the Appropriate JOIN

Use the join that matches the required result.

### Example: Retrieve customers who have orders

One option is an inner join:

```sql
SELECT DISTINCT
    c.customer_id,
    c.customer_name
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id;
```

Another option is `EXISTS`:

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

The second query expresses an existence requirement directly and avoids producing one customer row per matching order.

The first query needs `DISTINCT` because a customer may have multiple orders. If the business requirement is only to identify customers with at least one order, `EXISTS` is often a clear choice.

Neither formulation is universally faster. Optimizers may transform them into similar plans, so compare performance when necessary.

## 8. Avoid Unnecessary DISTINCT

`DISTINCT` removes duplicate result rows and may require sorting or hashing.

### Potentially unnecessary

```sql
SELECT DISTINCT department_id
FROM employees;
```

If the requirement is to list unique departments, `DISTINCT` is necessary unless another mechanism guarantees uniqueness.

However, if duplicates appear unexpectedly after a join, investigate the join condition and data relationships instead of automatically adding `DISTINCT`.

Example:

```sql
SELECT
    c.customer_id,
    o.order_id
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id;
```

Multiple rows per customer are expected when the customer has multiple orders. They are not necessarily duplicates.

## 9. Optimize Subqueries and Repeated Calculations

Subqueries are not inherently inefficient. Their performance depends on the query structure and the optimizer.

### Example: Correlated subquery

Find employees earning more than their department's average salary:

```sql
SELECT
    e.employee_id,
    e.employee_name,
    e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

### Alternative using a window function

```sql
WITH employee_stats AS (
    SELECT
        employee_id,
        employee_name,
        department_id,
        salary,
        AVG(salary) OVER (
            PARTITION BY department_id
        ) AS avg_department_salary
    FROM employees
)
SELECT
    employee_id,
    employee_name,
    salary
FROM employee_stats
WHERE salary > avg_department_salary;
```

The second formulation computes department averages using a window function.

It may avoid repeated work in some execution plans, but it is not guaranteed to be faster. Validate the results and compare execution plans.

## 10. Optimize GROUP BY and Aggregations

Aggregation can become expensive when the database processes a large number of rows.

### Example

```sql
SELECT
    department_id,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id;
```

Possible improvements include:

- Filtering irrelevant rows before aggregation.
- Computing related metrics in one aggregation.
- Avoiding unnecessary columns and joins.
- Checking whether the plan uses an efficient aggregation strategy.
- Reducing the data processed when the business logic allows it.

For example, combine multiple metrics instead of scanning the same data separately for each one:

```sql
SELECT
    customer_id,
    COUNT(*) AS order_count,
    SUM(amount) AS total_amount,
    AVG(amount) AS average_amount
FROM orders
GROUP BY customer_id;
```

A single aggregation calculates three metrics for each customer.

## 11. Optimize Sorting and Top-N Queries

Sorting a large dataset can require substantial CPU and memory.

### Retrieve all rows and sort them

```sql
SELECT employee_id, salary
FROM employees
ORDER BY salary DESC;
```

### Retrieve only the top 10

```sql
SELECT employee_id, salary
FROM employees
ORDER BY salary DESC, employee_id
LIMIT 10;
```

The second query requests fewer rows. An appropriate index or top-N execution strategy may reduce the work required.

However, `LIMIT` does not guarantee that the database can avoid reading or sorting many rows. Inspect the execution plan.

`LIMIT` is not supported in every database; SQL Server, for example, commonly uses `TOP` or `OFFSET ... FETCH`.

The additional `employee_id` ordering makes the result deterministic when salaries tie, assuming `employee_id` uniquely identifies each employee.

## 12. Use EXISTS for Existence Checks

When you only need to determine whether a matching record exists, `EXISTS` often expresses the requirement clearly.

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

This returns customers with at least one order.

For customers without orders:

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

`EXISTS` and `NOT EXISTS` are particularly useful for existence and anti-existence checks. The optimizer determines the actual execution strategy.

## 13. Avoid Unnecessary OR Conditions When They Hinder the Plan

Consider:

```sql
SELECT *
FROM employees
WHERE department_id = 10
   OR department_id = 20;
```

For conditions on the same column, `IN` can express the requirement more clearly:

```sql
SELECT *
FROM employees
WHERE department_id IN (10, 20);
```

The optimizer may produce the same plan for both versions.

For more complex `OR` conditions, the best strategy depends on available indexes, data distribution, and the database engine. Do not automatically replace every `OR` with `UNION`; doing so can change duplicate-handling behavior and may add extra work.

## 14. Avoid Implicit Data-Type Conversions

Comparing incompatible data types may introduce conversions that reduce index effectiveness.

Suppose `customer_id` is an integer.

### Prefer a compatible value

```sql
SELECT *
FROM customers
WHERE customer_id = 1001;
```

Instead of relying on implicit conversion:

```sql
SELECT *
FROM customers
WHERE customer_id = '1001';
```

Some databases convert the string literal safely to an integer. The issue depends on the engine and which side of the comparison is converted.

Use compatible data types and inspect the plan when conversion warnings or unexpected scans appear.

## 15. Understand JOIN Algorithms

A database optimizer may choose among several join strategies.

| Join algorithm | General behavior | Often useful when |
|---|---|---|
| Nested-loop join | Iterates through one input and searches for matches in the other | One input is small and efficient lookups are available |
| Hash join | Builds a hash structure from one input and probes it with the other | Equi-joins with suitable input sizes and memory |
| Merge join | Combines inputs ordered by the join keys | Inputs are already ordered or can be processed efficiently in order |

These are general patterns, not strict rules. The best algorithm depends on the database, data size, available indexes, and execution conditions.

A hash join is not automatically better than a nested-loop join, and a nested-loop join is not automatically inefficient.

## 16. Use Execution Plans to Find the Real Bottleneck

Use `EXPLAIN` or the database's execution-plan tools.

Example in PostgreSQL:

```sql
EXPLAIN
SELECT employee_id, employee_name
FROM employees
WHERE department_id = 10;
```

For actual execution information:

```sql
EXPLAIN ANALYZE
SELECT employee_id, employee_name
FROM employees
WHERE department_id = 10;
```

Inspect:

- Table and index scans.
- Estimated versus actual row counts.
- Join algorithms.
- Sort and aggregation operations.
- Repeated scans.
- Buffer usage and disk spills where available.
- Lock waits and other runtime information from suitable monitoring tools.

In PostgreSQL, `EXPLAIN ANALYZE` executes the query. Be careful with statements that modify data.

A full scan is not inherently a problem. Focus on the operation that contributes most to the query's actual cost.

## 17. Query Optimization in Data Engineering

SQL optimization in a relational database is related to, but different from, optimization in distributed data processing.

| Problem | Possible approach |
|---|---|
| Reading unnecessary columns | Column pruning |
| Scanning unnecessary partitions | Partition pruning |
| Large shuffle in Spark | Reduce data before shuffling and inspect join strategy |
| Skewed join keys | Use appropriate skew-handling techniques |
| Too many small files | Compact files when beneficial |
| Repeated full data processing | Consider incremental processing |
| Inefficient relational lookup | Evaluate indexes and join keys |
| Expensive aggregation | Reduce input data and examine aggregation strategy |

In Spark and Databricks, useful tools include `EXPLAIN`, Spark UI, query profiles where available, and execution metrics.

A distributed shuffle is not always avoidable. Joins and aggregations often require data movement, so the goal is to reduce unnecessary work rather than eliminate every shuffle.

## 18. Common Query Optimization Mistakes

- Assuming a shorter query is faster.
- Creating indexes without analyzing the workload.
- Using `DISTINCT` to conceal an incorrect join.
- Replacing subqueries with joins without checking semantics.
- Applying functions to indexed columns without considering alternatives.
- Assuming a CTE always materializes or always improves performance.
- Adding `LIMIT` and assuming the database stops all processing immediately.
- Ignoring actual row counts and execution plans.
- Optimizing runtime without checking the returned data.
- Assuming SQL tuning and Spark optimization use exactly the same techniques.

## 19. Common Interview Questions

**Q1. What is query optimization?**

Improving query efficiency while preserving the intended results.

**Q2. How do you optimize a slow SQL query?**

Measure its performance, inspect the execution plan, identify the bottleneck, make a targeted change, and compare both performance and correctness.

**Q3. Does adding an index always improve query performance?**

No. Indexes require storage and maintenance and may be less efficient when a query retrieves a large proportion of the table.

**Q4. Why is `SELECT *` sometimes inefficient?**

It can retrieve unnecessary columns, increasing I/O, network transfer, and memory consumption.

**Q5. Is EXISTS always faster than JOIN?**

No. `EXISTS` is often appropriate for existence checks, but the optimizer may produce similar plans for equivalent queries.

**Q6. How can you improve a query that filters on a date column?**

Consider a range predicate instead of applying a function to the column, and check whether an appropriate index or partition-pruning strategy is available.

**Q7. What is the role of EXPLAIN?**

It helps reveal the database's planned execution strategy and potential performance bottlenecks.

**Q8. How do you optimize a query that aggregates millions of rows?**

Investigate the plan, filter unnecessary data when valid, reduce unnecessary joins and columns, and evaluate the aggregation strategy.

**Q9. What is the difference between SQL optimization and Spark optimization?**

SQL optimization may emphasize indexes, relational joins, statistics, and access paths. Spark optimization also considers distributed shuffles, partitioning, skew, and file layout.

**Q10. How do you prove that an optimization worked?**

Compare execution time, resource use, and execution plans under comparable conditions, then verify that the results remain correct.

## 20. Interview Practice

1. A query filtering by `customer_id` is slow. How would you investigate it?
2. Rewrite a year-based date filter using a date range.
3. Find customers who have orders without returning duplicate customer rows.
4. Explain when a nested-loop join can be more efficient than a hash join.
5. How would you optimize a query that performs a large sort before returning 10 rows?
6. Why might an index not be used even when it exists?
7. How would you investigate a large gap between estimated and actual rows?
8. How would you optimize a Spark query that performs a large shuffle?
9. How do you check that a rewritten query preserves the original results?
10. Describe a step-by-step method for diagnosing a slow production query.

## Quick Revision

- **Select fewer columns:** Reduce unnecessary data retrieval.
- **Filter effectively:** Reduce input rows before expensive processing when valid.
- **Use indexes appropriately:** Improve suitable access patterns.
- **Write index-friendly predicates:** Prefer efficient range and equality conditions where appropriate.
- **Optimize joins:** Check keys, cardinality, indexes, and join strategies.
- **Use EXISTS:** Express existence requirements directly.
- **Control sorting and aggregation:** Avoid unnecessary work.
- **Inspect execution plans:** Find the actual bottleneck.
- **Measure before and after:** Verify real improvements.
- **Validate correctness:** Performance improvements must not change the intended result.

**Core interview takeaway:** SQL query optimization is a systematic process: understand the requirement, measure performance, inspect the plan, optimize the real bottleneck, and validate both speed and correctness.
