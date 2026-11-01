# SQL Interview Questions — Medium Level

## 1. What to Expect at the Medium Level

Medium-level SQL interviews test whether you can combine multiple SQL concepts to solve practical business problems.

Focus on:

- Subqueries and correlated subqueries
- CTEs and recursive CTEs
- Window functions
- Ranking and top-N problems
- Joins and anti-joins
- Conditional aggregation
- Running totals and moving averages
- Date-based analysis
- Deduplication
- Consecutive records and gaps
- Query execution and optimization
- Business-oriented SQL problems

## 2. Sample Tables

Use these tables throughout the examples.

### `employees`

| employee_id | employee_name | department | salary | manager_id |
|---:|---|---|---:|---:|
| 1 | Amit | IT | 80000 | NULL |
| 2 | Neha | IT | 95000 | 1 |
| 3 | Ravi | IT | 95000 | 1 |
| 4 | Sara | HR | 70000 | NULL |
| 5 | John | HR | 75000 | 4 |
| 6 | Priya | Finance | 90000 | NULL |

### `orders`

| order_id | customer_id | order_date | amount | status |
|---:|---:|---|---:|---|
| 101 | 1 | 2026-01-05 | 500 | COMPLETED |
| 102 | 1 | 2026-01-10 | 300 | COMPLETED |
| 103 | 2 | 2026-01-12 | 700 | COMPLETED |
| 104 | 1 | 2026-02-02 | 200 | COMPLETED |
| 105 | 2 | 2026-02-15 | 400 | CANCELLED |
| 106 | 3 | 2026-02-20 | 600 | COMPLETED |

### `customers`

| customer_id | customer_name | city |
|---:|---|---|
| 1 | Rahul | Kolkata |
| 2 | Priya | Delhi |
| 3 | Aman | Mumbai |
| 4 | Neha | Chennai |

---

## 3. Subqueries and CTEs

### Q1. Find employees earning more than the average salary of their department.

```sql
SELECT
    employee_id,
    employee_name,
    department,
    salary
FROM employees e
WHERE salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department = e.department
);
```

**Concept:** Correlated subquery.

The inner query calculates the average salary for the outer employee's department.

Alternative using a window function:

```sql
WITH salary_data AS (
    SELECT
        employee_id,
        employee_name,
        department,
        salary,
        AVG(salary) OVER (
            PARTITION BY department
        ) AS department_avg
    FROM employees
)
SELECT *
FROM salary_data
WHERE salary > department_avg;
```

### Q2. Find departments whose total salary is greater than the average department salary.

```sql
WITH department_salary AS (
    SELECT
        department,
        SUM(salary) AS total_salary
    FROM employees
    GROUP BY department
)
SELECT *
FROM department_salary
WHERE total_salary > (
    SELECT AVG(total_salary)
    FROM department_salary
);
```

**Concept:** Aggregate first, then compare the aggregated results.

### Q3. What is a recursive CTE?

A recursive CTE references itself to process hierarchical or sequential data.

It usually contains:
1. Anchor member: Starting rows.
2. Recursive member: Retrieves the next level.
3. Termination condition: Stops when no more rows are returned.

Example: Employee-manager hierarchy.

```sql
WITH RECURSIVE employee_hierarchy AS (
    -- Anchor: Employees without managers
    SELECT
        employee_id,
        employee_name,
        manager_id,
        1 AS hierarchy_level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive member: Find direct reports
    SELECT
        e.employee_id,
        e.employee_name,
        e.manager_id,
        h.hierarchy_level + 1
    FROM employees e
    JOIN employee_hierarchy h
        ON e.manager_id = h.employee_id
)
SELECT *
FROM employee_hierarchy;
```

Recursive CTE syntax differs across databases. SQL Server, for example, does not use the `RECURSIVE` keyword.

---

## 4. Window Function Problems

### Q4. Find the second-highest salary in each department.

```sql
WITH ranked AS (
    SELECT
        employee_id,
        employee_name,
        department,
        salary,
        DENSE_RANK() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS salary_rank
    FROM employees
)
SELECT
    employee_id,
    employee_name,
    department,
    salary
FROM ranked
WHERE salary_rank = 2;
```

**Why `DENSE_RANK()`?**

It finds the second-highest distinct salary and returns all employees tied at that salary.

### Q5. Find the top two employees in each department.

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
SELECT *
FROM ranked
WHERE rn <= 2;
```

This returns at most two employees per department.

If the requirement is to include everyone tied within the top two salary ranks, use `DENSE_RANK()` instead.

### Q6. Find employees earning more than the previous employee in their department.

```sql
WITH salary_comparison AS (
    SELECT
        employee_id,
        employee_name,
        department,
        salary,
        LAG(salary) OVER (
            PARTITION BY department
            ORDER BY salary, employee_id
        ) AS previous_salary
    FROM employees
)
SELECT *
FROM salary_comparison
WHERE salary > previous_salary;
```

`LAG()` accesses the preceding row based on the window ordering.

### Q7. Calculate a running total of sales for each customer.

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
WHERE status = 'COMPLETED';
```

The `WHERE` clause filters completed orders before the window function is calculated.

### Q8. Find the percentage contribution of each department to total salary.

```sql
WITH department_salary AS (
    SELECT
        department,
        SUM(salary) AS total_salary
    FROM employees
    GROUP BY department
)
SELECT
    department,
    total_salary,
    ROUND(
        100.0 * total_salary
        / NULLIF(SUM(total_salary) OVER (), 0),
        2
    ) AS salary_percentage
FROM department_salary;
```

`SUM(total_salary) OVER ()` calculates the overall total while preserving each department's row.

---

## 5. Joins and Business Logic

### Q9. Find customers who have never placed an order.

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

Alternative using `NOT EXISTS`:

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

Both approaches express an anti-join.

### Q10. Find customers who have placed at least one completed order.

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
      AND o.status = 'COMPLETED'
);
```

`EXISTS` avoids returning duplicate customers when a customer has multiple qualifying orders.

### Q11. Find customers who have placed orders in both January and February.

```sql
SELECT
    customer_id
FROM orders
WHERE order_date >= '2026-01-01'
  AND order_date < '2026-03-01'
GROUP BY customer_id
HAVING COUNT(
    DISTINCT CASE
        WHEN order_date >= '2026-01-01'
         AND order_date < '2026-02-01'
            THEN 'January'
        WHEN order_date >= '2026-02-01'
         AND order_date < '2026-03-01'
            THEN 'February'
    END
) = 2;
```

This checks for at least one order in each month. Add a status condition if only completed orders should qualify.

### Q12. Find customers whose total completed order amount exceeds 1,000.

```sql
SELECT
    customer_id,
    SUM(amount) AS total_amount
FROM orders
WHERE status = 'COMPLETED'
GROUP BY customer_id
HAVING SUM(amount) > 1000;
```

### Q13. Find customers with more than one order.

```sql
SELECT
    customer_id,
    COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

If the source can contain duplicate order records, deduplicate by the business key before counting.

---

## 6. Deduplication and Data Cleaning

### Q14. Find duplicate records based on a business key.

Assume `customer_updates` stores multiple versions of customer records.

```sql
SELECT
    customer_id,
    COUNT(*) AS record_count
FROM customer_updates
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

This identifies keys with multiple records. Multiple versions may be valid, depending on the table's grain.

### Q15. Keep only the latest record for each customer.

```sql
WITH ranked AS (
    SELECT
        customer_id,
        customer_name,
        city,
        updated_at,
        update_id,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY updated_at DESC, update_id DESC
        ) AS rn
    FROM customer_updates
)
SELECT
    customer_id,
    customer_name,
    city,
    updated_at
FROM ranked
WHERE rn = 1;
```

The secondary sort column ensures deterministic selection when timestamps are equal.

### Q16. Remove duplicate records while keeping the latest record.

For a database that supports DELETE with a CTE:

```sql
WITH ranked AS (
    SELECT
        update_id,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY updated_at DESC, update_id DESC
        ) AS rn
    FROM customer_updates
)
DELETE FROM customer_updates
WHERE update_id IN (
    SELECT update_id
    FROM ranked
    WHERE rn > 1
);
```

This is illustrative. CTE-based DELETE syntax and restrictions vary by database. Run a SELECT preview first, verify the business key and tie-breaker, and execute destructive changes in a controlled transaction where supported.

---

## 7. Date and Time Problems

### Q17. Calculate monthly revenue.

```sql
SELECT
    EXTRACT(YEAR FROM order_date) AS order_year,
    EXTRACT(MONTH FROM order_date) AS order_month,
    SUM(amount) AS monthly_revenue
FROM orders
WHERE status = 'COMPLETED'
GROUP BY
    EXTRACT(YEAR FROM order_date),
    EXTRACT(MONTH FROM order_date)
ORDER BY
    order_year,
    order_month;
```

Date extraction syntax varies by database. For large datasets, consider filtering using date ranges so the database can use an index or partition pruning effectively.

### Q18. Find the first order date for each customer.

```sql
SELECT
    customer_id,
    MIN(order_date) AS first_order_date
FROM orders
GROUP BY customer_id;
```

### Q19. Find customers who placed orders on consecutive days.

Assume one record per customer per calendar date is sufficient for the activity definition.

```sql
WITH distinct_days AS (
    SELECT DISTINCT
        customer_id,
        order_date
    FROM orders
),
activity AS (
    SELECT
        customer_id,
        order_date,
        LEAD(order_date) OVER (
            PARTITION BY customer_id
            ORDER BY order_date
        ) AS next_order_date
    FROM distinct_days
)
SELECT
    customer_id,
    order_date,
    next_order_date
FROM activity
WHERE next_order_date = order_date + INTERVAL '1 day';
```

This interval expression is supported by some databases, but date arithmetic syntax differs. For SQL Server, use `DATEADD(day, 1, order_date)`.

### Q20. Find the number of days between consecutive orders.

```sql
SELECT
    customer_id,
    order_date,
    LAG(order_date) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
    ) AS previous_order_date
FROM orders;
```

To calculate the difference, use the date-difference function appropriate to your database.

For example, in SQL Server:

```sql
SELECT
    customer_id,
    order_date,
    DATEDIFF(
        DAY,
        LAG(order_date) OVER (
            PARTITION BY customer_id
            ORDER BY order_date, order_id
        ),
        order_date
    ) AS days_since_previous_order
FROM orders;
```

---

## 8. Conditional Aggregation

### Q21. Count completed and cancelled orders for each customer.

```sql
SELECT
    customer_id,
    SUM(
        CASE
            WHEN status = 'COMPLETED' THEN 1
            ELSE 0
        END
    ) AS completed_orders,
    SUM(
        CASE
            WHEN status = 'CANCELLED' THEN 1
            ELSE 0
        END
    ) AS cancelled_orders
FROM orders
GROUP BY customer_id;
```

This calculates multiple metrics in one grouped query.

### Q22. Calculate the completed-order percentage for each customer.

```sql
SELECT
    customer_id,
    COUNT(*) AS total_orders,
    SUM(
        CASE
            WHEN status = 'COMPLETED' THEN 1
            ELSE 0
        END
    ) AS completed_orders,
    100.0 * SUM(
        CASE
            WHEN status = 'COMPLETED' THEN 1
            ELSE 0
        END
    ) / NULLIF(COUNT(*), 0) AS completion_percentage
FROM orders
GROUP BY customer_id;
```

The denominator includes all rows in the table for each customer. Define the intended population carefully if some statuses should be excluded.

---

## 9. Intermediate SQL Concepts

### Q23. What is the difference between UNION and UNION ALL?

| UNION | UNION ALL |
|---|---|
| Removes duplicate result rows | Retains duplicate result rows |
| Usually requires duplicate elimination | Usually avoids duplicate elimination |
| Can require additional processing | Often faster when duplicates are acceptable |

```sql
SELECT customer_id FROM orders
UNION
SELECT customer_id FROM archived_orders;
```

```sql
SELECT customer_id FROM orders
UNION ALL
SELECT customer_id FROM archived_orders;
```

### Q24. What is the difference between ROWS and RANGE in window functions?

Both define a window frame, but they use different rules.

- `ROWS` defines the frame by physical row positions.
- `RANGE` defines the frame using ordering values and peer rows.

Example:

```sql
SELECT
    order_date,
    amount,
    SUM(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 1 PRECEDING AND CURRENT ROW
    ) AS rows_total
FROM orders;
```

This includes the current row and one preceding row.

With `RANGE`, rows with equal ordering values may be included together, depending on the frame definition and database implementation.

### Q25. What is the difference between RANK and DENSE_RANK?

For values `100, 100, 90, 80`:

| Value | RANK | DENSE_RANK |
|---:|---:|---:|
| 100 | 1 | 1 |
| 100 | 1 | 1 |
| 90 | 3 | 2 |
| 80 | 4 | 3 |

Use `DENSE_RANK()` when you need consecutive ranks for distinct values.

### Q26. What is a self join?

A self join joins a table with itself.

Example: Display employees and their managers.

```sql
SELECT
    e.employee_name AS employee,
    m.employee_name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.employee_id;
```

### Q27. What is a CROSS JOIN?

A CROSS JOIN returns every possible combination of rows from two tables.

If table A contains 5 rows and table B contains 4 rows, the result contains 20 rows.

### Q28. What is a CASE expression?

It applies conditional logic inside a SQL query.

```sql
SELECT
    employee_name,
    salary,
    CASE
        WHEN salary >= 90000 THEN 'High'
        WHEN salary >= 70000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_category
FROM employees;
```

---

## 10. Query Optimization

### Q29. How do you optimize a SQL query?

Follow a measurement-driven process:

1. Inspect the execution plan.
2. Identify expensive scans, joins, sorts, and aggregations.
3. Filter unnecessary rows as early as appropriate.
4. Select only required columns.
5. Check whether indexes can support the query.
6. Verify join keys and table cardinality.
7. Check table statistics and data distribution.
8. Avoid unnecessary functions on indexed filter columns.
9. Consider partition pruning and pre-aggregation.
10. Benchmark the revised query.

### Q30. What is an index?

An index is a data structure that helps the database locate records efficiently for supported query patterns.

Benefits:
- Faster selective lookups.
- Improved join and filtering performance.
- Can help sorting and grouping.

Limitations:
- Requires additional storage.
- Adds overhead to writes.
- Too many or poorly designed indexes can reduce performance.

### Q31. What is a composite index?

A composite index contains multiple columns.

Example:

```sql
CREATE INDEX idx_orders_customer_date
ON orders (customer_id, order_date);
```

It can help queries that filter or order by the indexed columns, depending on the database and the index's leading-column rules.

Column order matters. This index is not necessarily equally useful for queries filtering only on `order_date`.

### Q32. What is an execution plan?

An execution plan describes the operations a database uses or intends to use to execute a query.

Inspect:
- Table scans and index access
- Join algorithms
- Sort operations
- Aggregations
- Estimated and actual row counts
- Memory usage and spills, where available

### Q33. What is the difference between an index seek and an index scan?

- **Index seek:** Navigates to a relevant index key or range.
- **Index scan:** Reads an index range or a larger portion of the index.

A scan is not automatically inefficient. When a query needs many rows, a scan may be the best plan.

### Q34. Why should we avoid SELECT * in production?

- Reads unnecessary columns.
- Increases data transfer.
- Can prevent effective column pruning in some systems.
- Makes queries less stable when table schemas change.
- Can make code harder to understand.

Select only the columns required by the query.

---

## 11. Data Engineering-Oriented SQL Questions

### Q35. How do you implement an incremental load using a timestamp?

A common approach is to extract records updated after the last successfully processed timestamp.

```sql
SELECT *
FROM source_orders
WHERE updated_at > :last_successful_timestamp
  AND updated_at <= :current_batch_cutoff;
```

Important considerations:
- Define timestamp precision and time zone.
- Use a reliable checkpoint.
- Advance the checkpoint only after target writes succeed.
- Use idempotent writes or deduplication.
- Consider overlapping extraction windows for late updates.
- Use CDC or reconciliation when timestamp extraction cannot capture all changes.

A timestamp-based query may miss records with unchanged timestamps, late-arriving updates, or physical deletes.

### Q36. How do you find records present in the source but missing in the target?

```sql
SELECT
    s.order_id
FROM source_orders s
WHERE NOT EXISTS (
    SELECT 1
    FROM target_orders t
    WHERE t.order_id = s.order_id
);
```

This is useful for reconciliation and identifying missing business keys.

### Q37. How do you compare source and target record counts?

```sql
SELECT
    'SOURCE' AS dataset,
    COUNT(*) AS record_count
FROM source_orders

UNION ALL

SELECT
    'TARGET' AS dataset,
    COUNT(*) AS record_count
FROM target_orders;
```

Matching counts do not prove that both datasets contain the same records. Compare keys, amounts, and other relevant attributes as well.

### Q38. How do you find records that changed between source and target?

Compare business keys and relevant columns.

```sql
SELECT
    s.order_id,
    s.amount AS source_amount,
    t.amount AS target_amount
FROM source_orders s
JOIN target_orders t
    ON s.order_id = t.order_id
WHERE s.amount <> t.amount
   OR (s.amount IS NULL AND t.amount IS NOT NULL)
   OR (s.amount IS NOT NULL AND t.amount IS NULL);
```

This handles NULL differences for `amount`. For many columns, a database-specific NULL-safe comparison or carefully designed row hash may simplify the logic.

### Q39. How do you identify the latest record for each business key?

Use `ROW_NUMBER()` with a deterministic ordering:

```sql
WITH ranked AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY order_id
            ORDER BY updated_at DESC, source_sequence DESC
        ) AS rn
    FROM order_updates
)
SELECT *
FROM ranked
WHERE rn = 1;
```

The sequence column must be meaningful and stable. If no reliable tie-breaker exists, define how conflicting records should be resolved.

---

## 12. Common Interview Traps

1. Using `ROW_NUMBER()` when the requirement expects all tied rows.
2. Forgetting `PARTITION BY` in department-wise ranking.
3. Using `COUNT(*)` when distinct business entities must be counted.
4. Joining tables with different grains and inflating totals.
5. Filtering after a LEFT JOIN in a way that unintentionally removes unmatched rows.
6. Ignoring NULL behavior in `NOT IN`.
7. Using a timestamp checkpoint without handling late updates.
8. Advancing a checkpoint before the target write succeeds.
9. Assuming `UNION` and `UNION ALL` behave the same.
10. Using `DISTINCT` to hide incorrect join logic.
11. Forgetting that `BETWEEN` includes both endpoints.
12. Assuming a CTE is always materialized.
13. Assuming an index always improves a query.
14. Ignoring deterministic ordering in latest-record logic.
15. Treating matching row counts as proof of successful reconciliation.

---

## 13. Quick Revision

| Concept | Key point |
|---|---|
| Correlated subquery | References the outer query |
| CTE | Named query expression |
| Recursive CTE | Processes hierarchical or sequential data |
| `ROW_NUMBER()` | Assigns a unique row number |
| `RANK()` | Ties share ranks; gaps follow |
| `DENSE_RANK()` | Ties share ranks; no gaps |
| `LAG()` | Reads a preceding row |
| `LEAD()` | Reads a following row |
| `PARTITION BY` | Creates independent window groups |
| `EXISTS` | Checks for a matching row |
| `NOT EXISTS` | Checks for no matching row |
| Conditional aggregation | Calculates multiple conditional metrics |
| Deduplication | Uses business keys and deterministic rules |
| Composite index | Index containing multiple columns |
| Execution plan | Shows query execution strategy |
| Incremental load | Processes new or changed data |
| Reconciliation | Compares source and target results |
| Idempotency | Makes retries safe under defined conditions |

## 14. Practice Checklist

- [ ] Find employees earning above their department average.
- [ ] Find the second-highest salary per department.
- [ ] Find the top two employees per department, including ties.
- [ ] Calculate running totals by customer.
- [ ] Calculate department-wise salary percentages.
- [ ] Find customers who never placed an order.
- [ ] Find customers with orders in two different months.
- [ ] Find customers whose spending exceeds a threshold.
- [ ] Keep the latest record per business key.
- [ ] Identify consecutive activity dates.
- [ ] Calculate the difference between consecutive orders.
- [ ] Perform conditional aggregation.
- [ ] Explain ROWS versus RANGE.
- [ ] Explain composite indexes and execution plans.
- [ ] Find records missing from a target table.
- [ ] Compare source and target values.
- [ ] Explain safe incremental loading.

**Interview strategy:** First identify the required output, grouping level, and business rules. Then select the right SQL technique. For Data Engineer interviews, be especially strong in window functions, joins, deduplication, incremental loads, and reconciliation.