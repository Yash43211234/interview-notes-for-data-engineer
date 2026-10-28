# SQL Ranking Functions

## 1. What Are Ranking Functions?

Ranking functions assign a position or rank to rows based on the order specified in a window function.

They are commonly used in Data Engineering and SQL interviews to:
- Find the highest- or lowest-paid employees.
- Identify the top N records in each group.
- Find the second-highest salary.
- Rank products by sales within each category.
- Identify duplicate records and select the latest record.

The three most important ranking functions are:

| Function | Behavior |
|---|---|
| `ROW_NUMBER()` | Assigns a unique sequential number to every row. |
| `RANK()` | Gives tied rows the same rank and leaves gaps afterward. |
| `DENSE_RANK()` | Gives tied rows the same rank without leaving gaps afterward. |

## 2. Sample Data

Assume we have an `employees` table:

| employee_id | employee_name | department | salary |
|---:|---|---|---:|
| 101 | Amit | IT | 90000 |
| 102 | Priya | IT | 80000 |
| 103 | Rahul | IT | 80000 |
| 104 | Neha | IT | 70000 |
| 105 | Karan | HR | 75000 |

We will use this data to understand the ranking functions.

## 3. `ROW_NUMBER()`

`ROW_NUMBER()` assigns a different sequential number to each row, even when multiple rows have the same salary.

### Syntax

```sql
ROW_NUMBER() OVER (
    PARTITION BY department
    ORDER BY salary DESC
)
```

### Example

```sql
SELECT
    employee_id,
    employee_name,
    department,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department
        ORDER BY salary DESC, employee_id
    ) AS row_num
FROM employees;
```

### Result

| employee_name | department | salary | row_num |
|---|---|---:|---:|
| Amit | IT | 90000 | 1 |
| Priya | IT | 80000 | 2 |
| Rahul | IT | 80000 | 3 |
| Neha | IT | 70000 | 4 |
| Karan | HR | 75000 | 1 |

**Important:** `employee_id` is added as a tie-breaker so that employees with equal salaries have a predictable order.

### When to use it

Use `ROW_NUMBER()` when you need exactly one row per position, such as:
- Selecting the latest record for each customer.
- Removing duplicates while retaining one record.
- Choosing exactly one employee per department.

## 4. `RANK()`

`RANK()` assigns the same rank to rows with equal values. The next rank skips positions.

### Example

```sql
SELECT
    employee_name,
    department,
    salary,
    RANK() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

### Result

| employee_name | department | salary | salary_rank |
|---|---|---:|---:|
| Amit | IT | 90000 | 1 |
| Priya | IT | 80000 | 2 |
| Rahul | IT | 80000 | 2 |
| Neha | IT | 70000 | 4 |
| Karan | HR | 75000 | 1 |

Priya and Rahul both receive rank 2 because their salaries are equal. Neha receives rank 4 because rank 3 is skipped.

### When to use it

Use `RANK()` when tied values should share a rank and the next rank should reflect the number of rows ahead.

Example: competition rankings, where two people tie for second place and the next person comes fourth.

## 5. `DENSE_RANK()`

`DENSE_RANK()` also assigns the same rank to tied values, but it does not skip the next rank.

### Example

```sql
SELECT
    employee_name,
    department,
    salary,
    DENSE_RANK() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS dense_salary_rank
FROM employees;
```

### Result

| employee_name | department | salary | dense_salary_rank |
|---|---|---:|---:|
| Amit | IT | 90000 | 1 |
| Priya | IT | 80000 | 2 |
| Rahul | IT | 80000 | 2 |
| Neha | IT | 70000 | 3 |
| Karan | HR | 75000 | 1 |

Neha receives rank 3 because `DENSE_RANK()` does not leave gaps.

### When to use it

Use `DENSE_RANK()` when you need to rank distinct values without gaps, such as finding the second-highest distinct salary.

## 6. The Difference Between All Three

Consider these salaries in descending order:

`100000, 90000, 90000, 80000`

| Salary | `ROW_NUMBER()`* | `RANK()` | `DENSE_RANK()` |
|---:|---:|---:|---:|
| 100000 | 1 | 1 | 1 |
| 90000 | 2 | 2 | 2 |
| 90000 | 3 | 2 | 2 |
| 80000 | 4 | 4 | 3 |

<text>*The two rows with salary 90000 can receive row numbers 2 and 3 in either order unless a tie-breaker is specified.</text>

**Remember:**
- `ROW_NUMBER()`: every row gets a unique number.
- `RANK()`: ties share a rank; gaps appear.
- `DENSE_RANK()`: ties share a rank; no gaps appear.

## 7. Understanding `PARTITION BY` and `ORDER BY`

### `ORDER BY`

Determines how rows are ranked.

```sql
RANK() OVER (ORDER BY salary DESC)
```

This ranks all employees together, from highest salary to lowest.

### `PARTITION BY`

Divides rows into groups and ranks each group independently.

```sql
RANK() OVER (
    PARTITION BY department
    ORDER BY salary DESC
)
```

Here, each department starts at rank 1.

**Interview point:** Without `PARTITION BY`, the ranking is calculated across the entire result set. With `PARTITION BY department`, ranking restarts for each department.

## 8. Find the Second-Highest Distinct Salary

This is a common SQL interview problem.

```sql
WITH ranked_employees AS (
    SELECT
        employee_id,
        employee_name,
        salary,
        DENSE_RANK() OVER (
            ORDER BY salary DESC
        ) AS salary_rank
    FROM employees
)
SELECT employee_id, employee_name, salary
FROM ranked_employees
WHERE salary_rank = 2;
```

Why use `DENSE_RANK()`? It ranks distinct salary values without gaps, so rank 2 represents the second-highest distinct salary.

If multiple employees share that salary, the query returns all of them.

## 9. Find the Top 3 Employees in Each Department

```sql
WITH ranked_employees AS (
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
    salary,
    salary_rank
FROM ranked_employees
WHERE salary_rank <= 3
ORDER BY department, salary_rank, employee_id;
```

This returns employees whose salary ranks are 1, 2, or 3 within their department. Because it uses `DENSE_RANK()`, ties may result in more than three employees per department.

### Important variation: Exactly three employees per department

If the requirement is **at most three employees per department**, regardless of ties, use `ROW_NUMBER()`:

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
        ) AS row_num
    FROM employees
)
SELECT employee_id, employee_name, department, salary
FROM ranked_employees
WHERE row_num <= 3;
```

The tie-breaker ensures a deterministic choice when salaries are equal.

## 10. Find the Latest Record for Each Customer

Ranking functions are useful in ETL pipelines when you receive multiple records for the same business key and need to retain the latest one.

Assume a table named `customer_updates` has:

- `customer_id`
- `customer_name`
- `updated_at`

```sql
WITH ranked_updates AS (
    SELECT
        customer_id,
        customer_name,
        updated_at,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY updated_at DESC
        ) AS row_num
    FROM customer_updates
)
SELECT customer_id, customer_name, updated_at
FROM ranked_updates
WHERE row_num = 1;
```

This keeps one latest record for each customer.

**Production consideration:** If multiple records for a customer have the same `updated_at`, add another stable column, such as a unique update ID, to the `ORDER BY` clause.

## 11. Common Mistakes

### Mistake 1: Using a window function directly in `WHERE`

This generally does not work:

```sql
SELECT
    employee_name,
    RANK() OVER (ORDER BY salary DESC) AS salary_rank
FROM employees
WHERE salary_rank = 2;
```

The `WHERE` clause is evaluated before the window function in the same query block.

**Correct approach:** Calculate the rank in a CTE or subquery, then filter it in the outer query.

### Mistake 2: Adding a tie-breaker to `RANK()` when equal values must tie

```sql
RANK() OVER (
    ORDER BY salary DESC, employee_id
)
```

Because `employee_id` is unique, employees with equal salaries will no longer be tied under this ordering.

If equal salaries must share a rank, order by salary alone:

```sql
RANK() OVER (ORDER BY salary DESC)
```

Use a tie-breaker with `ROW_NUMBER()` when you need a predictable order among equal salaries.

### Mistake 3: Confusing the second row with the second-highest distinct value

- Second row: use `ROW_NUMBER()` if rows are ordered by salary.
- Second-highest distinct salary: use `DENSE_RANK()` and filter for rank 2.

These can produce different results when duplicate salaries exist.

### Mistake 4: Assuming `PARTITION BY` is mandatory

It is optional. If omitted, all rows are treated as one partition.

### Mistake 5: Assuming `RANK()` and `DENSE_RANK()` return the same result

They match until a tie causes a gap. After that, `RANK()` skips positions while `DENSE_RANK()` does not.

## 12. Interview Questions

**Q1. What is the difference between `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()`?**

`ROW_NUMBER()` assigns unique numbers to rows. `RANK()` gives tied rows the same rank and leaves gaps. `DENSE_RANK()` gives tied rows the same rank without gaps.

**Q2. How do you find the second-highest salary?**

Use `DENSE_RANK()` ordered by salary descending, then filter for rank 2. This returns all employees earning the second-highest distinct salary.

**Q3. How do you find the top three salaries in each department?**

Use `DENSE_RANK()` with `PARTITION BY department` and `ORDER BY salary DESC`, then filter for rank less than or equal to 3.

**Q4. What happens if `PARTITION BY` is omitted?**

The ranking is calculated across the entire result set instead of restarting within groups.

**Q5. Can ranking functions be used to remove duplicates?**

Yes. Assign `ROW_NUMBER()` within each duplicate group and keep row number 1. Choose the ordering carefully to decide which record to retain.

**Q6. Does `ROW_NUMBER()` guarantee a stable order for tied values?**

No. Add a stable tie-breaker to the `ORDER BY` clause when deterministic results are required.

**Q7. Can a window function be filtered directly in `WHERE`?**

Generally, no. Use a CTE, subquery, or a dialect-specific `QUALIFY` clause where supported.

## 13. Practice Questions

Try solving these without looking at the examples above.

1. Assign a unique row number to employees ordered by salary from highest to lowest.
2. Rank employees by salary across the entire company, with equal salaries sharing a rank.
3. Rank employees separately within each department using `DENSE_RANK()`.
4. Return every employee earning the second-highest distinct salary.
5. Return the top two distinct salary levels within each department.
6. Return exactly one highest-paid employee from each department.
7. From a customer updates table, keep the latest record per customer.
8. Explain the result of `RANK()` and `DENSE_RANK()` for values `100, 100, 90, 80`.

## 14. Quick Revision

| Requirement | Best choice |
|---|---|
| Give every row a unique sequential number | `ROW_NUMBER()` |
| Rank values and leave gaps after ties | `RANK()` |
| Rank values without gaps after ties | `DENSE_RANK()` |
| Find the second-highest distinct salary | `DENSE_RANK()` |
| Keep exactly one selected row per group | `ROW_NUMBER()` |
| Find the top N distinct values per group | `DENSE_RANK()` |
| Rank each department independently | `PARTITION BY department` |

### Self-check

Before moving on, make sure you can explain:

- [ ] The difference between all three ranking functions.
- [ ] Why `DENSE_RANK()` is useful for the second-highest distinct salary.
- [ ] How `PARTITION BY` changes the ranking.
- [ ] Why window-function results usually need a CTE or subquery before filtering.
- [ ] How to use `ROW_NUMBER()` to select the latest record per customer.
- [ ] How ties affect the number of rows returned by a top-N query.

**Interview takeaway:** First identify what the problem means by “top N”: exactly N rows, N ranking positions, or N distinct values. That determines whether `ROW_NUMBER()`, `RANK()`, or `DENSE_RANK()` is the best choice.