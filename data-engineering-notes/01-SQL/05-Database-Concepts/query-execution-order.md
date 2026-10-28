# SQL Query Execution Order

## 1. What Is SQL Query Execution Order?

SQL query execution order is the logical sequence in which the database processes the clauses of a query.

Although we usually write a query starting with `SELECT`, the database logically processes several other clauses before determining which rows and columns appear in the final result.

Understanding this order helps explain:
- Why a column alias may not be available in `WHERE`.
- Why aggregate functions are filtered using `HAVING`.
- Why window functions generally cannot be filtered directly in `WHERE`.
- How joins, grouping, and sorting affect query results.

## 2. Logical Execution Order

A common logical order for a `SELECT` query is:

1. `FROM`
2. `ON`
3. `JOIN`
4. `WHERE`
5. `GROUP BY`
6. Aggregate calculations
7. `HAVING`
8. Window functions in the `SELECT` list
9. `SELECT`
10. `DISTINCT`
11. `ORDER BY`
12. `LIMIT` / `OFFSET` or the database-specific row-limiting clause

**Important:** This is a simplified logical model, not a universal physical execution plan. SQL engines may optimize and execute operations in a different physical order while preserving the query's required semantics. Exact clause support and processing details vary by database.

## 3. Example Table

Assume an `employees` table:

| employee_id | employee_name | department | salary |
|---:|---|---|---:|
| 101 | Amit | IT | 90000 |
| 102 | Priya | IT | 85000 |
| 103 | Rahul | HR | 70000 |
| 104 | Neha | HR | 65000 |
| 105 | Ravi | IT | 75000 |

## 4. Step-by-Step Example

Consider this query:

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employees
WHERE salary > 65000
GROUP BY department
HAVING AVG(salary) > 75000
ORDER BY avg_salary DESC
LIMIT 2;
```

### Step 1: FROM

The database identifies the source table:

```sql
FROM employees
```

The rows come from `employees`.

### Step 2: WHERE

The database filters individual rows before grouping:

```sql
WHERE salary > 65000
```

Remaining employees:

| employee_id | employee_name | department | salary |
|---:|---|---|---:|
| 101 | Amit | IT | 90000 |
| 102 | Priya | IT | 85000 |
| 103 | Rahul | HR | 70000 |
| 105 | Ravi | IT | 75000 |

Neha is removed because her salary is 65000, not greater than 65000.

### Step 3: GROUP BY

The remaining rows are grouped by department:

```sql
GROUP BY department
```

- IT: 90000, 85000, 75000
- HR: 70000

### Step 4: Aggregate Calculations

The database calculates the average salary for each group:

- IT: 83333.33
- HR: 70000

### Step 5: HAVING

The database filters groups using:

```sql
HAVING AVG(salary) > 75000
```

Only IT remains because its average salary exceeds 75000.

### Step 6: SELECT

The output columns and alias are determined:

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
```

Result:

| department | avg_salary |
|---|---:|
| IT | 83333.33 |

### Step 7: ORDER BY

The result is sorted by `avg_salary` in descending order:

```sql
ORDER BY avg_salary DESC
```

### Step 8: LIMIT

The query returns at most two rows:

```sql
LIMIT 2
```

Only one department qualifies, so the final result contains one row.

## 5. WHERE vs. HAVING

This is one of the most frequently tested SQL interview concepts.

### WHERE

Filters individual rows before grouping.

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
WHERE salary > 65000
GROUP BY department;
```

### HAVING

Filters groups after grouping and aggregation.

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 2;
```

**Remember:**

- `WHERE` → filters rows.
- `HAVING` → filters groups.

## 6. Why Can't We Usually Use a SELECT Alias in WHERE?

Consider:

```sql
SELECT
    salary * 12 AS annual_salary
FROM employees
WHERE annual_salary > 900000;
```

This query is invalid in many SQL databases because the `WHERE` clause is logically evaluated before the `SELECT` list defines the alias.

Instead, write:

```sql
SELECT
    salary * 12 AS annual_salary
FROM employees
WHERE salary * 12 > 900000;
```

Or use a subquery:

```sql
SELECT annual_salary
FROM (
    SELECT salary * 12 AS annual_salary
    FROM employees
) AS salary_data
WHERE annual_salary > 900000;
```

**Note:** Alias rules vary by clause and database. Some engines permit aliases in additional clauses, but `WHERE` generally cannot reference a same-level `SELECT` alias.

## 7. Why Can't We Usually Filter a Window Function in WHERE?

Consider finding the highest-paid employee in each department:

```sql
SELECT
    employee_id,
    employee_name,
    department,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS rn
FROM employees
WHERE rn = 1;
```

This generally fails because the window function result is not available when `WHERE` is evaluated.

Use a CTE:

```sql
WITH ranked_employees AS (
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
FROM ranked_employees
WHERE rn = 1;
```

The inner query calculates `rn`. The outer query filters it after that result exists.

Some databases support a `QUALIFY` clause for filtering window-function results.

## 8. Logical Order of JOINs

Consider:

```sql
SELECT
    c.customer_name,
    o.order_id
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.order_id IS NOT NULL;
```

Conceptually:

1. `FROM` identifies the source.
2. `ON` determines matching rows.
3. `LEFT JOIN` preserves unmatched customer rows with `NULL` values on the order side.
4. `WHERE` removes rows for which `o.order_id IS NOT NULL` is false or unknown.

As a result, customers without orders are removed. For this condition, the query behaves like an inner join.

**Interview trap:** Moving a condition between `ON` and `WHERE` can change the results of an outer join.

## 9. DISTINCT, ORDER BY, and LIMIT

Consider:

```sql
SELECT DISTINCT department
FROM employees
ORDER BY department
LIMIT 2;
```

Conceptually:

1. Read the source rows.
2. Select the department values.
3. Remove duplicate department values.
4. Sort the distinct values.
5. Return at most two rows.

`LIMIT` is supported by PostgreSQL and MySQL. SQL Server commonly uses `TOP` or `OFFSET ... FETCH`, while other engines may use different syntax.

Without `ORDER BY`, the rows selected by a row-limiting clause are not guaranteed to be a particular set.

## 10. Logical Order vs. Physical Execution Order

**Logical execution order** describes how SQL clauses determine the meaning of the result.

**Physical execution order** describes how the database engine actually performs the work.

A database optimizer may:
- Use an index instead of scanning the whole table.
- Push filters closer to the data source.
- Reorder joins.
- Choose hash, merge, or nested-loop joins.
- Perform aggregation using different execution strategies.

These optimizations can change the physical plan without changing the query's required result.

Use `EXPLAIN` or the database's execution-plan tools to inspect the physical plan.

## 11. Common Interview Questions

**Q1. Which clause is logically processed first?**

`FROM`, which identifies the source data. For joins, the join conditions and matching operations are part of forming the source result.

**Q2. Which is processed first: WHERE or GROUP BY?**

`WHERE` logically filters rows before `GROUP BY` forms groups.

**Q3. Which is processed first: WHERE or HAVING?**

`WHERE` is processed first; `HAVING` filters the groups formed by aggregation.

**Q4. Why can ORDER BY often use a SELECT alias?**

`ORDER BY` is logically evaluated after the `SELECT` list has been formed, so many databases allow aliases defined in `SELECT` to be referenced there.

**Q5. Why can't a window-function alias usually be used in WHERE?**

The window function is calculated after `WHERE` in the logical processing model. Use a CTE, subquery, or a supported `QUALIFY` clause.

**Q6. Does the database physically execute every clause in this exact order?**

No. The optimizer can choose a different physical execution strategy while preserving the query's semantics.

**Q7. What is the difference between WHERE and ON in a LEFT JOIN?**

`ON` determines matching rows. `WHERE` filters the joined result and may remove rows that the outer join originally preserved.

## 12. Common Mistakes

- Assuming SQL executes in the order clauses are written.
- Using an aggregate function in `WHERE` instead of `HAVING`.
- Filtering a window-function alias in the same query's `WHERE`.
- Assuming aliases are available in every clause.
- Moving conditions from `ON` to `WHERE` without checking outer-join behavior.
- Assuming the logical order is identical to the physical execution plan.
- Using `LIMIT` without `ORDER BY` when a specific result order is required.

## 13. Interview Practice

1. Write a query to find departments with more than five employees earning over 50000.
2. Explain why a `SELECT` alias cannot generally be used in `WHERE`.
3. Find the top three highest-paid employees in each department.
4. Explain how moving a condition from `ON` to `WHERE` changes a `LEFT JOIN`.
5. Identify the logical execution order of a query containing `DISTINCT`, `GROUP BY`, `HAVING`, and `ORDER BY`.
6. Explain the difference between logical and physical execution order.

## Quick Revision

- **FROM:** Identify source tables and form joined input.
- **ON / JOIN:** Determine how rows match and how tables are combined.
- **WHERE:** Filter individual rows.
- **GROUP BY:** Form groups.
- **HAVING:** Filter groups.
- **Window functions:** Calculate values over related rows.
- **SELECT:** Produce the output expressions and aliases.
- **DISTINCT:** Remove duplicate result rows.
- **ORDER BY:** Sort the output.
- **LIMIT / OFFSET:** Restrict or paginate the result.

**Core interview takeaway:** Remember the logical processing order to understand aliases, filtering, aggregation, joins, and window functions—but distinguish it from the database's optimized physical execution plan.
