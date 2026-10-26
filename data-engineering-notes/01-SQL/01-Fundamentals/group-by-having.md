# GROUP BY and HAVING in SQL

## 1. What is GROUP BY?

`GROUP BY` groups rows that have the same value in one or more columns. It is commonly used with aggregate functions such as `COUNT()`, `SUM()`, `AVG()`, `MIN()`, and `MAX()`.

**Example:** Find the number of employees in each department.

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

This query groups employees by department and counts the employees in each group.

## 2. Why Do We Need GROUP BY?

Use `GROUP BY` when you need summary information for each category instead of individual rows.

Common use cases:
- Count employees in each department.
- Calculate total sales for each customer.
- Find average salary by department.
- Calculate total orders for each month.
- Find the highest salary in each department.

## 3. How GROUP BY Works

Suppose we have this `employees` table:

| name | department | salary |
|---|---|---:|
| Amit | IT | 60000 |
| Neha | HR | 40000 |
| Ravi | IT | 30000 |
| Priya | Sales | 50000 |
| John | HR | NULL |

Query:

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

Result:

| department | employee_count |
|---|---:|
| IT | 2 |
| HR | 2 |
| Sales | 1 |

SQL creates one group for each department and counts the rows in each group. `COUNT(*)` includes rows even when some column values are `NULL`.

## 4. GROUP BY with Aggregate Functions

### COUNT()

Count the employees in each department.

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

### SUM()

Calculate the total salary in each department.

```sql
SELECT department, SUM(salary) AS total_salary
FROM employees
GROUP BY department;
```

`SUM()` ignores `NULL` salary values.

### AVG()

Calculate the average salary in each department.

```sql
SELECT department, AVG(salary) AS average_salary
FROM employees
GROUP BY department;
```

`AVG()` ignores `NULL` salary values.

### MAX() and MIN()

Find the highest and lowest salary in each department.

```sql
SELECT
    department,
    MAX(salary) AS highest_salary,
    MIN(salary) AS lowest_salary
FROM employees
GROUP BY department;
```

**Important:** Aggregate functions such as `SUM()`, `AVG()`, `MIN()`, and `MAX()` ignore `NULL` values. `COUNT(column)` ignores `NULL`, while `COUNT(*)` counts rows.

## 5. GROUP BY with Multiple Columns

You can group by more than one column.

Example: Count employees for each department and job title.

```sql
SELECT department, job_title, COUNT(*) AS employee_count
FROM employees
GROUP BY department, job_title;
```

Each unique combination of `department` and `job_title` forms a separate group.

**Remember:** Grouping by two columns does not mean grouping by either column independently. It groups by their combination.

## 6. What is HAVING?

`HAVING` filters groups after `GROUP BY` has formed them. It is commonly used to filter results based on aggregate calculations.

Example: Find departments with more than one employee.

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 1;
```

Result:

| department | employee_count |
|---|---:|
| IT | 2 |
| HR | 2 |

The Sales department is excluded because it has only one employee.

## 7. WHERE vs HAVING

This is a very common SQL interview question.

| WHERE | HAVING |
|---|---|
| Filters individual rows | Filters groups |
| Applied before grouping | Applied after grouping |
| Commonly filters regular column values | Commonly filters aggregate results |
| Cannot directly use aggregate functions in its condition in standard SQL | Can filter using aggregate functions |

### Example using WHERE and HAVING together

Find departments whose average salary is greater than ₹40,000, considering only employees with a known salary.

```sql
SELECT department, AVG(salary) AS average_salary
FROM employees
WHERE salary IS NOT NULL
GROUP BY department
HAVING AVG(salary) > 40000;
```

How it works:

1. `WHERE` removes rows with `NULL` salaries.
2. `GROUP BY` groups the remaining employees by department.
3. `AVG()` calculates the average salary for each department.
4. `HAVING` keeps only departments whose average is greater than 40,000.

Result:

| department | average_salary |
|---|---:|
| IT | 45000 |
| Sales | 50000 |

HR is excluded because its only known salary is 40,000, which is not greater than 40,000.

## 8. SQL Query Execution Order

A simplified logical execution order is:

1. `FROM` — identifies the source table.
2. `WHERE` — filters individual rows.
3. `GROUP BY` — creates groups.
4. `HAVING` — filters groups.
5. `SELECT` — selects output columns and calculates expressions.
6. `ORDER BY` — sorts the result.
7. `LIMIT` — restricts the number of returned rows, where supported.

**Interview tip:** SQL is written starting with `SELECT`, but its logical processing order is different.

## 9. GROUP BY with ORDER BY

Example: Show departments from highest to lowest total salary.

```sql
SELECT department, SUM(salary) AS total_salary
FROM employees
GROUP BY department
ORDER BY total_salary DESC;
```

- `GROUP BY` creates department groups.
- `SUM()` calculates each department's total.
- `ORDER BY` sorts the resulting groups by total salary.

## 10. GROUP BY with Conditional Aggregation

Conditional aggregation calculates multiple summaries using conditions inside aggregate functions.

Example: Count employees by department in a single result row.

```sql
SELECT
    COUNT(CASE WHEN department = 'IT' THEN 1 END) AS it_count,
    COUNT(CASE WHEN department = 'HR' THEN 1 END) AS hr_count,
    COUNT(CASE WHEN department = 'Sales' THEN 1 END) AS sales_count
FROM employees;
```

Result:

| it_count | hr_count | sales_count |
|---:|---:|---:|
| 2 | 2 | 1 |

This query does not need `GROUP BY` because it returns one summary row for the entire table.

## 11. Common Mistakes

### Mistake 1: Selecting a column that is not grouped or aggregated

Incorrect:

```sql
SELECT department, name, COUNT(*)
FROM employees
GROUP BY department;
```

Why? A department can contain multiple employee names, so SQL cannot determine which single name to display for each group.

Correct:

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

In standard SQL, selected expressions generally need to be grouped or aggregated.

### Mistake 2: Using WHERE to filter an aggregate

Incorrect:

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department
WHERE COUNT(*) > 1;
```

Correct:

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 1;
```

### Mistake 3: Confusing COUNT(*) and COUNT(column)

```sql
SELECT
    department,
    COUNT(*) AS total_rows,
    COUNT(salary) AS employees_with_salary
FROM employees
GROUP BY department;
```

- `COUNT(*)` counts every row.
- `COUNT(salary)` counts only rows where salary is not `NULL`.

### Mistake 4: Using HAVING when WHERE is sufficient

If you need to filter individual employees before grouping, use `WHERE`. If you need to filter the resulting groups, use `HAVING`.

### Mistake 5: Forgetting the difference between `>` and `>=`

- `HAVING COUNT(*) > 2` keeps groups with more than two rows.
- `HAVING COUNT(*) >= 2` keeps groups with at least two rows.

Read the condition carefully.

## 12. Interview Questions

**Q1. What is GROUP BY?**

It groups rows with the same values so aggregate functions can calculate summaries for each group.

**Q2. What is HAVING?**

It filters groups, commonly using aggregate conditions.

**Q3. What is the difference between WHERE and HAVING?**

`WHERE` filters rows before grouping. `HAVING` filters groups after grouping.

**Q4. Can we use GROUP BY without aggregate functions?**

Yes. It can return unique combinations of grouped columns, although `DISTINCT` may be clearer when only unique values are needed.

**Q5. Can we use HAVING without GROUP BY?**

Yes, in many SQL databases. It can filter the aggregate result for the entire table. Exact behavior and support can vary by database.

Example:

```sql
SELECT COUNT(*) AS employee_count
FROM employees
HAVING COUNT(*) > 3;
```

This returns the count only if the table has more than three rows.

**Q6. Can we group by multiple columns?**

Yes. Each unique combination of those column values becomes a separate group.

**Q7. Does COUNT(*) count NULL values?**

It counts rows, including rows containing `NULL`. `COUNT(column)` ignores `NULL` values in that column.

**Q8. What happens if a grouped column contains NULL?**

Rows with `NULL` in the grouped column are generally placed into one group together.

## 13. Practice Questions

Use the `employees` table shown earlier.

1. Count employees in each department.
2. Find the total salary for each department.
3. Find the average salary for each department.
4. Display departments having at least two employees.
5. Find departments whose total salary is greater than 70,000.
6. Count employees with a known salary in each department.
7. Display departments in descending order of average salary.
8. Find departments whose average salary is greater than 40,000, excluding employees with `NULL` salaries.
9. Count employees for each combination of department and job title.
10. Find departments with more than one employee whose salary is greater than 40,000.

**Hint for Question 10:** Filter salary rows with `WHERE`, group by department, and use `HAVING` to filter the groups.

## 14. Quick Revision

- `GROUP BY` groups rows with matching values.
- Aggregate functions calculate summaries for each group.
- `HAVING` filters groups after grouping.
- `WHERE` filters individual rows before grouping.
- Use `WHERE` for row conditions and `HAVING` for aggregate conditions.
- Multiple columns in `GROUP BY` create groups based on unique combinations.
- `COUNT(*)` counts rows; `COUNT(column)` ignores `NULL`.
- `SUM()`, `AVG()`, `MIN()`, and `MAX()` ignore `NULL` values.
- Use `ORDER BY` to sort the final results.

**Remember the pattern:**

```sql
SELECT grouping_column, AGGREGATE_FUNCTION(column)
FROM table_name
WHERE row_condition
GROUP BY grouping_column
HAVING aggregate_condition
ORDER BY grouping_column;
```

You do not need every clause in every query. Use only the clauses required by the question.

## 15. Self-Check

Before marking this topic as interview-ready, confirm that you can:

- [ ] Explain `GROUP BY` in simple language.
- [ ] Use `COUNT`, `SUM`, and `AVG` with grouping.
- [ ] Group by multiple columns.
- [ ] Explain the difference between `WHERE` and `HAVING`.
- [ ] Filter groups using aggregate conditions.
- [ ] Explain how `NULL` affects `COUNT` and `AVG`.
- [ ] Solve the practice questions without looking at the notes.

**Revision status:** Needs Revision / Practicing / Interview Ready