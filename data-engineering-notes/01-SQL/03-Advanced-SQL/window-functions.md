# SQL Window Functions

## 1. What Are Window Functions?

A **window function** performs a calculation across a set of related rows while keeping the individual rows in the result.

Window functions are commonly used in Data Engineering and SQL interviews to:
- Rank employees by salary.
- Calculate running totals and moving averages.
- Compare the current row with previous or next rows.
- Find the highest value within each group.
- Calculate differences and percentages between rows.
- Identify the latest record for each customer.

The main advantage is that you can calculate values across multiple rows **without collapsing the result into one row per group**, as a regular `GROUP BY` aggregation usually does.

## 2. Basic Syntax

```sql
function_name(expression) OVER (
    PARTITION BY column_name
    ORDER BY column_name
    ROWS BETWEEN start_point AND end_point
)
```

The components are:

| Component | Purpose |
|---|---|
| `function_name` | The calculation or operation, such as `SUM()`, `AVG()`, or `ROW_NUMBER()`. |
| `OVER()` | Defines the window used by the function. |
| `PARTITION BY` | Divides rows into independent groups. Optional. |
| `ORDER BY` | Defines the order of rows within each group. Optional for some functions. |
| Window frame | Defines which rows within the ordered partition are included in the calculation. Optional, depending on the function and query. |

You do not need every component for every window function.

## 3. Sample Data

Assume we have an `employees` table:

| employee_id | employee_name | department | salary |
|---:|---|---|---:|
| 101 | Amit | IT | 90000 |
| 102 | Priya | IT | 80000 |
| 103 | Rahul | IT | 70000 |
| 104 | Neha | HR | 75000 |
| 105 | Karan | HR | 65000 |

We will use this table to understand the main types of window functions.

## 4. Window Functions vs. `GROUP BY`

### Using `GROUP BY`

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

Result:

| department | avg_salary |
|---|---:|
| IT | 80000 |
| HR | 70000 |

The query returns one row per department.

### Using a window function

```sql
SELECT
    employee_id,
    employee_name,
    department,
    salary,
    AVG(salary) OVER (
        PARTITION BY department
    ) AS department_avg_salary
FROM employees;
```

Result:

| employee_name | department | salary | department_avg_salary |
|---|---|---:|---:|
| Amit | IT | 90000 | 80000 |
| Priya | IT | 80000 | 80000 |
| Rahul | IT | 70000 | 80000 |
| Neha | HR | 75000 | 70000 |
| Karan | HR | 65000 | 70000 |

Each employee remains in the output, and the department average is added to every employee's row.

**Interview takeaway:** Use `GROUP BY` when you need one result per group. Use a window function when you need a group-level calculation alongside individual rows.

## 5. Understanding `PARTITION BY`

`PARTITION BY` divides the result into groups for window calculations. The function operates independently within each group.

```sql
SELECT
    employee_name,
    department,
    salary,
    SUM(salary) OVER (
        PARTITION BY department
    ) AS department_total_salary
FROM employees;
```

Every employee in IT receives the total IT salary, while every employee in HR receives the total HR salary.

Without `PARTITION BY`, the calculation would operate across the entire result set.

**Important:** `PARTITION BY` does not remove rows or necessarily sort the final output. It defines how rows are grouped for the window calculation.

## 6. Understanding `ORDER BY`

Within a window function, `ORDER BY` determines the sequence used by the calculation.

For example:

```sql
SELECT
    employee_name,
    salary,
    ROW_NUMBER() OVER (
        ORDER BY salary DESC, employee_id
    ) AS row_num
FROM employees;
```

The highest-paid employee receives row number 1.

The additional `employee_id` column provides a deterministic tie-breaker if salaries are equal.

**Note:** The `ORDER BY` inside `OVER()` controls the window calculation. It does not guarantee the final result's display order. Use an outer `ORDER BY` when the returned rows must be sorted.

## 7. Types of Window Functions

Window functions can be grouped into three broad categories.

### A. Ranking functions

These assign positions or ranks to rows.

| Function | Purpose |
|---|---|
| `ROW_NUMBER()` | Gives each row a unique sequential number. |
| `RANK()` | Assigns equal ranks to ties and skips subsequent ranks. |
| `DENSE_RANK()` | Assigns equal ranks to ties without skipping ranks. |
| `NTILE(n)` | Divides ordered rows into up to `n` numbered buckets. |

Example:

```sql
SELECT
    employee_name,
    salary,
    RANK() OVER (
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

Use ranking functions for top-N queries, salary rankings, and selecting one record from each group.

### B. Aggregate window functions

These apply aggregate calculations over a window while retaining individual rows.

Common examples:
- `SUM()`
- `AVG()`
- `COUNT()`
- `MIN()`
- `MAX()`

Example:

```sql
SELECT
    employee_name,
    department,
    salary,
    MAX(salary) OVER (
        PARTITION BY department
    ) AS highest_salary_in_department
FROM employees;
```

Use aggregate window functions for department totals, averages, running totals, and comparisons against group-level values.

### C. Navigation and value functions

These help retrieve values from other rows or positions in the window.

| Function | Purpose |
|---|---|
| `LAG()` | Reads a value from a preceding row. |
| `LEAD()` | Reads a value from a following row. |
| `FIRST_VALUE()` | Returns the first value in the window frame. |
| `LAST_VALUE()` | Returns the last value in the window frame. |

Example:

```sql
SELECT
    employee_name,
    salary,
    LAG(salary) OVER (
        ORDER BY salary DESC, employee_id
    ) AS previous_salary
FROM employees;
```

Use these functions to compare records, calculate changes over time, and analyze sequential events.

## 8. Running Totals

A running total calculates a cumulative sum from the beginning of the ordered partition through the current row.

Assume a `daily_sales` table contains `sale_date` and `sales`.

```sql
SELECT
    sale_date,
    sales,
    SUM(sales) OVER (
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM daily_sales
ORDER BY sale_date;
```

The frame means:
- `UNBOUNDED PRECEDING`: begin at the first row.
- `CURRENT ROW`: end at the current row.

For separate running totals by store, add `PARTITION BY store_id` before `ORDER BY`.

**Important:** Use a stable tie-breaker if multiple records have the same date and need a deterministic row-by-row sequence.

## 9. Moving Averages

A moving average calculates the average over a limited window of rows.

```sql
SELECT
    sale_date,
    sales,
    AVG(sales) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg
FROM daily_sales
ORDER BY sale_date;
```

This calculates an average over the current row and up to two preceding rows.

The frame contains up to three rows. At the beginning of the dataset, fewer than three rows are available.

**Important:** Three rows do not necessarily mean three calendar days. Missing dates or multiple records per day can change the meaning of the calculation.

## 10. Comparing Current and Previous Rows

`LAG()` is useful when you need to compare the current value with the preceding row.

```sql
SELECT
    sale_date,
    sales,
    LAG(sales) OVER (
        ORDER BY sale_date
    ) AS previous_sales,
    sales - LAG(sales) OVER (
        ORDER BY sale_date
    ) AS sales_difference
FROM daily_sales
ORDER BY sale_date;
```

The difference is:

\[
\text{Difference}=\text{Current Sales}-\text{Previous Sales}
\]

The first row has no preceding row, so `LAG()` returns `NULL` by default.

For percentage change, guard against division by zero:

```sql
SELECT
    sale_date,
    sales,
    LAG(sales) OVER (
        ORDER BY sale_date
    ) AS previous_sales,
    100.0 * (
        sales - LAG(sales) OVER (
            ORDER BY sale_date
        )
    ) / NULLIF(
        LAG(sales) OVER (
            ORDER BY sale_date
        ), 0
    ) AS percentage_change
FROM daily_sales;
```

`NULLIF(previous_sales, 0)` returns `NULL` when the previous value is zero, avoiding division by zero.

## 11. Find the Highest-Paid Employee in Each Department

A common interview problem is to find the highest-paid employee or employees in each department.

```sql
WITH ranked_employees AS (
    SELECT
        employee_id,
        employee_name,
        department,
        salary,
        RANK() OVER (
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
FROM ranked_employees
WHERE salary_rank = 1;
```

This returns all employees tied for the highest salary in their department.

If you need exactly one employee per department, use `ROW_NUMBER()` with a deterministic tie-breaker instead.

## 12. Window Frames: `ROWS` and `RANGE`

A window frame defines which rows contribute to a calculation.

### `ROWS`

Defines the frame by physical row positions in the ordered partition.

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

This includes the current row and up to two preceding rows.

### `RANGE`

Defines the frame based on ordering values and peer groups. Rows with equal ordering values can be included together.

The exact supported syntax and behavior depend on the database.

For running totals, explicitly specifying `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` is often the clearest approach when you want row-by-row cumulative behavior.

## 13. Important SQL Evaluation Rule

Window functions are evaluated after `WHERE`, `GROUP BY`, and `HAVING` in the logical processing of a query block.

This is why you generally cannot filter a window function's result directly in the same query block's `WHERE` clause.

Incorrect:

```sql
SELECT
    employee_name,
    salary,
    RANK() OVER (
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees
WHERE salary_rank = 1;
```

Correct approach using a CTE:

```sql
WITH ranked_employees AS (
    SELECT
        employee_name,
        salary,
        RANK() OVER (
            ORDER BY salary DESC
        ) AS salary_rank
    FROM employees
)
SELECT
    employee_name,
    salary,
    salary_rank
FROM ranked_employees
WHERE salary_rank = 1;
```

Some databases support `QUALIFY`, which can filter window-function results directly. Check your database's syntax before using it.

## 14. Common Mistakes

1. **Confusing `GROUP BY` with window functions:** A grouped query generally reduces rows; a window function preserves them.
2. **Forgetting `PARTITION BY`:** The calculation may operate across the entire dataset instead of restarting for each group.
3. **Using a non-unique ordering column:** Ties may make row numbering or sequential calculations unpredictable.
4. **Assuming window `ORDER BY` sorts the final result:** Use an outer `ORDER BY` for display order.
5. **Filtering a window result in `WHERE`:** Calculate it in a CTE or subquery first, or use `QUALIFY` if supported.
6. **Ignoring window frames:** Default frames can produce unexpected results, especially when ordering values have ties.
7. **Assuming offsets represent calendar days:** `LAG()` and `LEAD()` navigate rows, not necessarily dates.

## 15. Interview Questions

**Q1. What is a window function?**

A function that performs a calculation across related rows while preserving the individual rows in the result.

**Q2. What is the difference between a window function and `GROUP BY`?**

`GROUP BY` typically returns one row per group, while a window function can calculate group-level values for each individual row.

**Q3. What is the purpose of `PARTITION BY`?**

It divides the data into groups for independent window calculations.

**Q4. What is the difference between `RANK()` and `DENSE_RANK()`?**

Both assign the same rank to ties. `RANK()` leaves gaps after ties, whereas `DENSE_RANK()` does not.

**Q5. How do you calculate a running total?**

Use `SUM()` with `OVER(ORDER BY ...)` and a cumulative frame.

**Q6. What are `LAG()` and `LEAD()` used for?**

`LAG()` reads from a preceding row, and `LEAD()` reads from a following row. They are useful for comparing sequential records.

**Q7. Why can you not generally use a window alias in `WHERE`?**

The `WHERE` clause is evaluated before the window calculation in the same query block.

**Q8. What is the difference between `ROWS` and `RANGE`?**

`ROWS` uses row positions, while `RANGE` uses ordering values and peer-group behavior.

**Q9. How do you select the latest record for each customer?**

Use `ROW_NUMBER()` partitioned by customer ID and ordered by timestamp descending, with a tie-breaker where necessary. Keep row number 1.

## 16. Practice Questions

Try writing the queries yourself.

1. Rank employees by salary within each department.
2. Find the highest-paid employee in every department.
3. Calculate each employee's salary and the average salary of their department.
4. Calculate a running total of daily sales.
5. Calculate a three-row moving average.
6. Find the difference between the current and previous day's sales.
7. Find the second-highest distinct salary.
8. Select the latest record for each customer.
9. Calculate the percentage change in daily sales.
10. Explain when you would use `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()`.

## 17. Quick Revision

| Requirement | Common approach |
|---|---|
| Unique sequential numbering | `ROW_NUMBER()` |
| Ranking with gaps after ties | `RANK()` |
| Ranking without gaps | `DENSE_RANK()` |
| Total or average per group while retaining rows | `SUM() OVER()` / `AVG() OVER()` with `PARTITION BY` |
| Cumulative total | `SUM() OVER(ORDER BY ... ROWS ...)` |
| Previous row's value | `LAG()` |
| Next row's value | `LEAD()` |
| First or last value in a frame | `FIRST_VALUE()` / `LAST_VALUE()` |
| Filter a calculated rank | CTE, subquery, or supported `QUALIFY` |

### Self-check

- [ ] Can you explain `OVER()`, `PARTITION BY`, and `ORDER BY`?
- [ ] Can you distinguish aggregate window functions from grouped aggregates?
- [ ] Can you write a ranking query?
- [ ] Can you calculate running totals and moving averages?
- [ ] Can you compare rows using `LAG()` and `LEAD()`?
- [ ] Can you explain why window results usually need an outer query for filtering?
- [ ] Can you explain how ties affect window ordering and frames?

**Interview takeaway:** Understand the requirement first: ranking, group-level calculation, cumulative calculation, or row-to-row comparison. Then choose the appropriate window function, partition, ordering, and frame.