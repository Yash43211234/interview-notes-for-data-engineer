# Advanced SQL Interview Questions

## 1. What to Know Before Advanced SQL Interviews

Advanced SQL interviews test whether you can solve business problems using relational data, not just remember syntax.

Focus on:

- Window functions
- Common Table Expressions (CTEs)
- Subqueries and correlated subqueries
- Ranking and top-N problems
- Deduplication and latest-record selection
- Running totals and moving averages
- Gaps and islands
- Joins and aggregation correctness
- Conditional aggregation
- Query optimization and execution plans
- NULL handling and date logic

The examples below use broadly supported SQL syntax. Some features, including `QUALIFY`, date arithmetic, and pagination syntax, vary by database.

## 2. Sample Tables

Use these tables to understand the examples.

### `employees`

| employee_id | employee_name | department | salary |
|---:|---|---|---:|
| 1 | Amit | IT | 80000 |
| 2 | Neha | IT | 95000 |
| 3 | Ravi | IT | 95000 |
| 4 | Sara | HR | 70000 |
| 5 | John | HR | 75000 |

### `orders`

| order_id | customer_id | order_date | amount |
|---:|---:|---|---:|
| 101 | 1 | 2026-01-05 | 500 |
| 102 | 1 | 2026-01-10 | 300 |
| 103 | 2 | 2026-01-12 | 700 |
| 104 | 1 | 2026-02-02 | 200 |
| 105 | 2 | 2026-02-15 | 400 |

---

## 3. Window Functions

A window function calculates a result across related rows **without collapsing them into one row per group**.

### Q1. What is the difference between `GROUP BY` and a window function?

`GROUP BY` combines rows into groups. A window function calculates values across rows while retaining the individual rows.

```sql
-- One row per department
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

```sql
-- Every employee remains visible
SELECT
    employee_name,
    department,
    salary,
    AVG(salary) OVER (
        PARTITION BY department
    ) AS department_avg_salary
FROM employees;
```

Use `GROUP BY` for summaries and window functions when you need both detail and group-level calculations.

### Q2. Explain `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()`.

| Function | Behavior for tied values |
|---|---|
| `ROW_NUMBER()` | Assigns a different number to each row |
| `RANK()` | Assigns equal ranks and leaves gaps |
| `DENSE_RANK()` | Assigns equal ranks without gaps |

For salaries `95000, 95000, 80000`:

| Salary | ROW_NUMBER* | RANK | DENSE_RANK |
|---:|---:|---:|---:|
| 95000 | 1 | 1 | 1 |
| 95000 | 2 | 1 | 1 |
| 80000 | 3 | 3 | 2 |

*The order of the two tied rows under `ROW_NUMBER()` is not deterministic unless a tie-breaker is specified.*

```sql
SELECT
    employee_name,
    salary,
    ROW_NUMBER() OVER (
        ORDER BY salary DESC, employee_id
    ) AS row_num,
    RANK() OVER (
        ORDER BY salary DESC
    ) AS salary_rank,
    DENSE_RANK() OVER (
        ORDER BY salary DESC
    ) AS dense_salary_rank
FROM employees;
```

### Q3. Find the second-highest distinct salary.

```sql
WITH ranked AS (
    SELECT
        salary,
        DENSE_RANK() OVER (
            ORDER BY salary DESC
        ) AS salary_rank
    FROM employees
)
SELECT DISTINCT salary
FROM ranked
WHERE salary_rank = 2;
```

**Why `DENSE_RANK()`?** It ranks distinct salary values without gaps. In this example, the second-highest distinct salary is `80000`.

An alternative is to use `MAX(salary)` below the highest salary, but handle the case where no second salary exists.

### Q4. Find the top three employees by salary in each department.

```sql
WITH ranked AS (
    SELECT
        employee_id,
        employee_name,
        department,
        salary,
        ROW_NUMBER() OVER (
            PARTITION BY department
            ORDER BY salary DESC, employee_id
        ) AS rn
    FROM employees
)
SELECT
    employee_id,
    employee_name,
    department,
    salary
FROM ranked
WHERE rn <= 3;
```

This returns at most three employees per department. Use `DENSE_RANK()` instead if the requirement is the top three distinct salary levels, including ties.

### Q5. Calculate a running total of order amounts for each customer.

```sql
SELECT
    customer_id,
    order_date,
    order_id,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS running_total
FROM orders
ORDER BY customer_id, order_date, order_id;
```

For customer 1, the running totals are `500`, `800`, and `1000`.

The explicit `ROWS` frame makes the cumulative behavior clear. A stable ordering is important when multiple records have the same date.

### Q6. Find each customer's previous order amount.

```sql
SELECT
    customer_id,
    order_date,
    order_id,
    amount,
    LAG(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
    ) AS previous_amount
FROM orders;
```

`LAG()` reads a value from a preceding row in the window order. The first order for each customer has no preceding order, so the result is `NULL`.

`LEAD()` reads from a following row.

### Q7. Calculate a three-order moving average.

```sql
SELECT
    customer_id,
    order_date,
    order_id,
    amount,
    AVG(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
        ROWS BETWEEN 2 PRECEDING
                 AND CURRENT ROW
    ) AS moving_avg
FROM orders;
```

This calculates the average of the current order and up to two preceding orders for the same customer. It is a three-row window, not necessarily a three-day window.

---

## 4. CTEs and Subqueries

### Q8. What is a CTE?

A Common Table Expression is a named query result defined using `WITH`. It can make complex queries easier to read and maintain.

```sql
WITH department_avg AS (
    SELECT
        department,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT
    e.employee_name,
    e.department,
    e.salary,
    d.avg_salary
FROM employees e
JOIN department_avg d
    ON e.department = d.department
WHERE e.salary > d.avg_salary;
```

This returns employees earning above their department's average.

A CTE is not automatically materialized. Whether the optimizer inlines or materializes it depends on the database and query.

### Q9. What is the difference between a CTE, a subquery, and a temporary table?

- **CTE:** Named query expression scoped to one statement.
- **Subquery:** Query nested inside another query.
- **Temporary table:** A stored intermediate result with a lifetime defined by the database/session or transaction.

Use a CTE for readability, a subquery for compact nested logic, and a temporary table when a reusable or explicitly materialized intermediate result is useful.

### Q10. What is a correlated subquery?

A correlated subquery references columns from the outer query.

```sql
SELECT
    e.employee_name,
    e.department,
    e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department = e.department
);
```

The subquery is logically associated with each outer employee's department. The optimizer may transform it into a more efficient plan.

### Q11. Find customers whose total spending is greater than 500.

```sql
SELECT
    customer_id,
    SUM(amount) AS total_spending
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 500;
```

`WHERE` filters rows before grouping. `HAVING` filters groups after aggregation.

---

## 5. Deduplication and Latest Records

### Q12. How do you remove duplicate records while keeping the latest record?

Assume `customer_updates` contains multiple versions of each customer record, and `updated_at` identifies the update time.

```sql
WITH ranked AS (
    SELECT
        customer_id,
        customer_name,
        updated_at,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY updated_at DESC, update_id DESC
        ) AS rn
    FROM customer_updates
)
SELECT
    customer_id,
    customer_name,
    updated_at
FROM ranked
WHERE rn = 1;
```

`update_id` is a tie-breaker. Without a deterministic tie-breaker, two updates with identical timestamps could produce an unpredictable winner.

**Important:** This query selects one latest row per customer. It does not physically delete duplicates from the source table.

### Q13. Why can `DISTINCT` be insufficient for deduplication?

`DISTINCT` removes identical selected rows. It does not determine which of several different versions should be retained.

For example, two rows may share the same `customer_id` but have different addresses or update timestamps. Use a business rule and ranking logic to select the desired record.

### Q14. Find duplicate business keys.

```sql
SELECT
    customer_id,
    COUNT(*) AS record_count
FROM customer_updates
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

This identifies keys with multiple rows. Whether those rows are errors depends on the table's intended grain.

---

## 6. Conditional Aggregation

### Q15. Count completed and cancelled orders in one query.

Assume an `orders` table contains an `order_status` column.

```sql
SELECT
    customer_id,
    SUM(
        CASE
            WHEN order_status = 'COMPLETED' THEN 1
            ELSE 0
        END
    ) AS completed_orders,
    SUM(
        CASE
            WHEN order_status = 'CANCELLED' THEN 1
            ELSE 0
        END
    ) AS cancelled_orders
FROM orders
GROUP BY customer_id;
```

Conditional aggregation calculates multiple metrics in one grouped query.

Some databases support aggregate `FILTER` syntax as an alternative.

### Q16. Calculate each customer's percentage contribution to total revenue.

```sql
WITH customer_revenue AS (
    SELECT
        customer_id,
        SUM(amount) AS revenue
    FROM orders
    GROUP BY customer_id
)
SELECT
    customer_id,
    revenue,
    100.0 * revenue
        / NULLIF(SUM(revenue) OVER (), 0) AS revenue_percentage
FROM customer_revenue;
```

`NULLIF(..., 0)` prevents division by zero. The result is `NULL` if the total revenue is zero.

The percentage is based on the `amount` field as defined in the example. Production revenue calculations must account for cancellations, refunds, discounts, and business-specific rules.

---

## 7. Gaps, Islands, and Date Problems

### Q17. What are gaps and islands?

- **Gaps:** Missing values or missing periods in a sequence.
- **Islands:** Consecutive values or periods that form a group.

They are common in attendance, activity streaks, subscription periods, and event sequences.

### Q18. Identify consecutive activity dates.

Assume `user_activity` contains `user_id` and `activity_date`. First remove duplicate user-date records.

```sql
WITH distinct_days AS (
    SELECT DISTINCT
        user_id,
        activity_date
    FROM user_activity
),
numbered AS (
    SELECT
        user_id,
        activity_date,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY activity_date
        ) AS rn
    FROM distinct_days
),
grouped AS (
    SELECT
        user_id,
        activity_date,
        activity_date - rn * INTERVAL '1 day' AS island_key
    FROM numbered
)
SELECT
    user_id,
    MIN(activity_date) AS streak_start,
    MAX(activity_date) AS streak_end,
    COUNT(*) AS consecutive_days
FROM grouped
GROUP BY user_id, island_key;
```

This illustrates the date-minus-row-number technique. The interval syntax varies across databases; for example, SQL Server uses different date arithmetic. Ensure that `activity_date` is represented as a date and that the database supports the chosen interval expression.

### Q19. Find dates missing from a sequence.

A common approach is to:
1. Generate or obtain the expected calendar dates.
2. Left join actual activity dates.
3. Select expected dates with no matching activity.

```sql
SELECT
    c.calendar_date
FROM calendar c
LEFT JOIN daily_activity a
    ON c.calendar_date = a.activity_date
WHERE a.activity_date IS NULL;
```

This requires a calendar table containing the expected date range.

---

## 8. Joins and Aggregation Traps

### Q20. Why can joins produce incorrect totals?

Joining two tables with multiple rows per key can multiply records.

For example, an order with three line items and two payment records may produce six joined rows. Summing line-item revenue after this join can count each line twice.

**Safer approach:** Aggregate each source to the intended grain before joining, or use a model that preserves the correct relationships.

### Q21. What is the difference between `EXISTS` and `IN`?

`EXISTS` tests whether a subquery returns at least one row. `IN` tests membership in a set of values.

```sql
SELECT c.customer_id
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

This returns customers with at least one matching order.

`EXISTS` is often a natural expression for existence checks. Performance depends on the database, indexes, and execution plan.

### Q22. Why can `NOT IN` behave unexpectedly with NULL?

If the subquery returns a `NULL`, `NOT IN` can evaluate to `UNKNOWN` for otherwise unmatched values, causing rows to be excluded.

For an anti-match query, `NOT EXISTS` is often safer:

```sql
SELECT c.customer_id
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

This returns customers with no matching order. Be aware that ordinary equality does not match `NULL` to `NULL`.

### Q23. What is the difference between `UNION` and `UNION ALL`?

- `UNION` combines results and removes duplicate rows.
- `UNION ALL` combines results while retaining duplicates.

`UNION ALL` is usually less expensive when duplicate elimination is unnecessary.

Both queries require compatible column counts and compatible data types in corresponding positions.

---

## 9. NULL Handling and Logic

### Q24. Why does `column = NULL` not work?

SQL uses three-valued logic: `TRUE`, `FALSE`, and `UNKNOWN`. Comparisons with `NULL` normally produce `UNKNOWN`.

Use:

```sql
SELECT *
FROM employees
WHERE department IS NULL;
```

For non-NULL values, use `IS NOT NULL`.

### Q25. What does `COALESCE()` do?

It returns the first non-NULL expression.

```sql
SELECT
    employee_name,
    COALESCE(department, 'Unassigned') AS department_name
FROM employees;
```

This provides a fallback value for missing departments.

Be careful: replacing NULL with a default can change the meaning of the data. Use it when the fallback is appropriate for the business requirement.

### Q26. How do `COUNT(*)`, `COUNT(column)`, and `COUNT(DISTINCT column)` differ?

- `COUNT(*)`: Counts rows.
- `COUNT(column)`: Counts non-NULL values in that column.
- `COUNT(DISTINCT column)`: Counts distinct non-NULL values in that column.

These differences matter when calculating customer counts, order counts, and completeness metrics.

---

## 10. Query Optimization

### Q27. How do you optimize a slow SQL query?

Use a measurement-driven process:

1. Capture the actual query and representative parameters.
2. Inspect the execution plan.
3. Identify large scans, expensive joins, sorts, aggregations, and spills.
4. Check table statistics and data distribution.
5. Verify whether indexes or partition pruning can help.
6. Reduce unnecessary columns and rows.
7. Check whether joins multiply rows unexpectedly.
8. Consider pre-aggregation or a different data model.
9. Benchmark before and after the change.

Do not assume that adding an index or rewriting a query will automatically improve performance.

### Q28. What is an index?

An index is a data structure that can help a database locate rows without scanning the entire table.

Indexes can improve selective lookups and joins, but they require storage and add maintenance overhead to inserts, updates, and deletes.

Index design depends on the database engine, query patterns, cardinality, and write workload.

### Q29. What is the difference between an index seek and an index scan?

- **Index seek:** Navigates to a relevant index range or key.
- **Index scan:** Reads a range or a larger portion of an index.

A scan is not always bad. If a query needs a large proportion of a table, scanning can be more efficient than repeated lookups.

### Q30. What is a query execution plan?

An execution plan describes how the database intends to execute a query or, in an actual plan, how it executed it.

It can reveal:
- Table and index access
- Join algorithms
- Sort and aggregation operators
- Estimated versus actual row counts
- Data movement or spills, where available

Look for expensive operations and estimation errors rather than judging a plan only by the number of operators.

### Q31. Why can functions on filtered columns reduce index usage?

A predicate such as:

```sql
WHERE YEAR(order_date) = 2026
```

may prevent an ordinary index on `order_date` from being used efficiently, depending on the database and optimizer.

A range predicate is often more index-friendly:

```sql
WHERE order_date >= '2026-01-01'
  AND order_date <  '2027-01-01'
```

Date literal and parameter syntax can vary by database. The half-open range includes all times within 2026 when `order_date` is a timestamp.

### Q32. What is the difference between a clustered and a nonclustered index?

The terminology and exact behavior are database-specific.

In SQL Server:
- A clustered index determines the logical organization of table rows by the clustering key.
- A nonclustered index is a separate index structure that points to the corresponding rows.

A table generally has at most one clustered index because it can have only one clustered row organization. Other database engines use different storage and indexing models.

---

## 11. Advanced Problem-Solving Patterns

### Q33. Find employees earning more than their department's average.

```sql
SELECT
    employee_name,
    department,
    salary
FROM (
    SELECT
        employee_name,
        department,
        salary,
        AVG(salary) OVER (
            PARTITION BY department
        ) AS department_avg
    FROM employees
) x
WHERE salary > department_avg;
```

This is a useful window-function pattern because it preserves employee-level rows.

### Q34. Find customers who placed orders in both January and February.

```sql
SELECT customer_id
FROM orders
WHERE order_date >= '2026-01-01'
  AND order_date <  '2026-03-01'
GROUP BY customer_id
HAVING COUNT(
    DISTINCT CASE
        WHEN order_date >= '2026-01-01'
         AND order_date <  '2026-02-01'
        THEN 'January'
        WHEN order_date >= '2026-02-01'
         AND order_date <  '2026-03-01'
        THEN 'February'
    END
) = 2;
```

The query counts distinct month labels among the qualifying records. In production, use date boundaries appropriate to the reporting time zone and calendar.

### Q35. Find the first order for every customer.

```sql
WITH ranked AS (
    SELECT
        order_id,
        customer_id,
        order_date,
        amount,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date, order_id
        ) AS rn
    FROM orders
)
SELECT
    order_id,
    customer_id,
    order_date,
    amount
FROM ranked
WHERE rn = 1;
```

The `order_id` tie-breaker ensures deterministic selection when orders share a date.

### Q36. Find customers whose spending exceeds the average customer spending.

```sql
WITH customer_spending AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
),
ranked AS (
    SELECT
        customer_id,
        total_spending,
        AVG(total_spending) OVER () AS avg_customer_spending
    FROM customer_spending
)
SELECT
    customer_id,
    total_spending
FROM ranked
WHERE total_spending > avg_customer_spending;
```

This calculates the average across customers who have at least one row in `orders`. If the requirement includes customers with zero orders, start from the complete customer table and left join aggregated spending.

---

## 12. SQL Concepts Interviewers Commonly Probe

| Concept | What you should explain |
|---|---|
| Window functions | Partitioning, ordering, frames, and row preservation |
| CTEs | Readability, recursion where supported, and materialization behavior |
| Ranking | Ties, gaps, deterministic ordering, top-N semantics |
| Deduplication | Business key, latest record, and tie-breaking |
| Aggregation | Grain, `GROUP BY`, `HAVING`, and conditional aggregation |
| NULL | Three-valued logic, NULL-safe comparisons, aggregate behavior |
| Joins | Cardinality, unmatched rows, fanout, and anti-joins |
| Indexes | Access paths, write overhead, and selectivity |
| Execution plans | Scans, joins, estimates, sorts, and spills |
| Date logic | Half-open intervals, time zones, and calendar boundaries |
| Data modeling | Fact grain, dimensions, and historical attributes |
| Data engineering | Incremental loads, deduplication, reconciliation, and safe reruns |

---

## 13. Common Mistakes

1. Using `ROW_NUMBER()` when ties must all be returned.
2. Using `RANK()` when the question asks for the second distinct value and expects no rank gaps.
3. Forgetting a deterministic tie-breaker for latest-record selection.
4. Using `WHERE` to filter aggregate results instead of `HAVING`.
5. Using `DISTINCT` to hide incorrect join logic.
6. Joining multiple fact tables at incompatible grains.
7. Forgetting that `COUNT(column)` ignores NULL.
8. Using `NOT IN` without considering NULL values.
9. Assuming a CTE is always materialized.
10. Assuming every index improves performance.
11. Using date functions in predicates without checking index implications.
12. Assuming the SQL syntax is identical across PostgreSQL, SQL Server, MySQL, Oracle, and cloud warehouses.
13. Forgetting to define whether “top three” means three rows or three distinct ranks.
14. Ignoring empty inputs and division-by-zero cases.
15. Optimizing a query without measuring its execution plan.

---

## 14. Quick Revision

- `ROW_NUMBER()`: Unique sequential number per row.
- `RANK()`: Ties share a rank; later ranks have gaps.
- `DENSE_RANK()`: Ties share a rank; no gaps.
- `LAG()`: Access a preceding row's value.
- `LEAD()`: Access a following row's value.
- `PARTITION BY`: Defines independent window groups.
- `ORDER BY` in a window: Defines the row sequence.
- Window frame: Defines which ordered rows contribute to a window calculation.
- CTE: Names a query expression for one statement.
- `WHERE`: Filters input rows.
- `HAVING`: Filters groups after aggregation.
- `EXISTS`: Tests whether a matching row exists.
- `NOT EXISTS`: Tests whether no matching row exists.
- `UNION`: Removes duplicate output rows.
- `UNION ALL`: Retains duplicate output rows.
- `COALESCE()`: Returns the first non-NULL expression.
- `COUNT(*)`: Counts rows.
- `COUNT(column)`: Counts non-NULL values.
- Index: Can speed up suitable access patterns but adds storage and write cost.
- Execution plan: Shows how a query is executed or planned.
- Deduplication: Requires a business rule, not merely a `DISTINCT`.
- Optimization: Measure, inspect, change, and benchmark.

## 15. Practice Checklist

- [ ] Find the second-highest distinct salary.
- [ ] Find the top three salaries per department, including ties.
- [ ] Calculate a running total by customer.
- [ ] Calculate a moving average.
- [ ] Find the latest record per business key.
- [ ] Find customers with no orders.
- [ ] Calculate conditional counts in one query.
- [ ] Find duplicate keys.
- [ ] Identify consecutive activity days.
- [ ] Find customers spending above the average.
- [ ] Explain join fanout with an example.
- [ ] Read a query execution plan and identify a likely bottleneck.
- [ ] Rewrite a date filter to use a half-open range.
- [ ] Explain why a retry-safe data pipeline needs idempotent writes.

**Interview rule:** First clarify the required result and its grain, then choose the SQL pattern. Correctness comes before optimization.