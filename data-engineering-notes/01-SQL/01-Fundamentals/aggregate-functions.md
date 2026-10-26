# SQL Aggregate Functions

## 1. What Are Aggregate Functions?

Aggregate functions perform a calculation on multiple rows and return a single result.

For example, they can calculate:
- The total salary of employees.
- The average salary.
- The number of employees.
- The highest and lowest salary.

**Remember:** Aggregate functions help us summarize data.

---

## 2. Important Aggregate Functions

Assume we have this `employees` table:

| emp_id | name | department | salary |
|---:|---|---|---:|
| 1 | Amit | IT | 50000 |
| 2 | Neha | IT | 60000 |
| 3 | Ravi | HR | 40000 |
| 4 | Priya | HR | 45000 |
| 5 | John | IT | NULL |

### `COUNT()`

Counts rows or non-NULL values, depending on what we pass to it.

```sql
-- Count all rows
SELECT COUNT(*) AS total_rows
FROM employees;
```

Result: `5`

```sql
-- Count non-NULL salaries
SELECT COUNT(salary) AS salary_count
FROM employees;
```

Result: `4`

**Remember:**
- `COUNT(*)` counts rows.
- `COUNT(column_name)` counts non-NULL values in that column.
- `COUNT(DISTINCT column_name)` counts unique, non-NULL values.

### `SUM()`

Calculates the total of non-NULL numeric values.

```sql
SELECT SUM(salary) AS total_salary
FROM employees;
```

Result: `195000`

### `AVG()`

Calculates the average of non-NULL numeric values.

```sql
SELECT AVG(salary) AS average_salary
FROM employees;
```

Result: `48750`

**Important:** NULL salaries are ignored. The average is calculated using the four known salaries, not all five rows.

### `MIN()`

Returns the smallest non-NULL value.

```sql
SELECT MIN(salary) AS minimum_salary
FROM employees;
```

Result: `40000`

### `MAX()`

Returns the largest non-NULL value.

```sql
SELECT MAX(salary) AS maximum_salary
FROM employees;
```

Result: `60000`

---

## 3. Quick Comparison

| Function | Purpose | Ignores NULL values? |
|---|---|---|
| `COUNT(*)` | Counts rows | No; counts rows even when columns contain NULL |
| `COUNT(column)` | Counts non-NULL values | Yes |
| `SUM(column)` | Calculates total | Yes |
| `AVG(column)` | Calculates average | Yes |
| `MIN(column)` | Finds minimum | Yes |
| `MAX(column)` | Finds maximum | Yes |

**Special case:** If there are no non-NULL values to calculate from, `SUM()`, `AVG()`, `MIN()`, and `MAX()` return NULL. `COUNT()` returns `0`.

---

## 4. Aggregate Functions with `GROUP BY`

`GROUP BY` divides rows into groups. Aggregate functions then calculate a result for each group.

### Example: Count employees in each department

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

Result:

| department | employee_count |
|---|---:|
| HR | 2 |
| IT | 3 |

### Example: Calculate average salary by department

```sql
SELECT
    department,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department;
```

Result:

| department | average_salary |
|---|---:|
| HR | 42500 |
| IT | 55000 |

**Remember:** Every group gets its own aggregate result.

---

## 5. Aggregate Functions with `HAVING`

`HAVING` filters groups after they have been created and their aggregate values calculated.

### Example: Find departments with more than two employees

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 2;
```

Result:

| department | employee_count |
|---|---:|
| IT | 3 |

**Remember the difference:**
- `WHERE` filters individual rows before grouping.
- `HAVING` filters groups after grouping.

Example:

```sql
SELECT department, AVG(salary) AS average_salary
FROM employees
WHERE salary IS NOT NULL
GROUP BY department
HAVING AVG(salary) > 45000;
```

This query removes rows with NULL salaries, groups the remaining rows by department, and returns groups whose average salary is greater than 45,000.

---

## 6. Aggregate Functions and NULL

NULL means a value is missing or unknown.

Most aggregate functions ignore NULL values.

Consider these salaries:

`10000, 20000, NULL`

| Expression | Result | Reason |
|---|---:|---|
| `COUNT(*)` | 3 | Three rows exist |
| `COUNT(salary)` | 2 | Two salaries are not NULL |
| `SUM(salary)` | 30000 | NULL is ignored |
| `AVG(salary)` | 15000 | Average of 10000 and 20000 |

### Common mistake

Do not assume that `AVG(salary)` divides the total by the total number of rows. It divides by the number of non-NULL salary values.

---

## 7. `COUNT(DISTINCT ...)`

Use this when you want to count unique values.

```sql
SELECT COUNT(DISTINCT department) AS department_count
FROM employees;
```

Result: `2`

The departments are IT and HR.

**Remember:** Duplicate values are counted once, and NULL is not counted.

---

## 8. Conditional Aggregation

Conditional aggregation calculates different totals or counts based on conditions.

### Example: Count employees in each department category

```sql
SELECT
    COUNT(*) AS total_employees,
    SUM(CASE WHEN department = 'IT' THEN 1 ELSE 0 END) AS it_count,
    SUM(CASE WHEN department = 'HR' THEN 1 ELSE 0 END) AS hr_count
FROM employees;
```

Result:

| total_employees | it_count | hr_count |
|---:|---:|---:|
| 5 | 3 | 2 |

This is useful when an interviewer asks for several counts or totals in a single query.

---

## 9. Aggregate Functions vs. Window Functions

Aggregate functions with `GROUP BY` usually return one row per group. Window functions calculate results while keeping the individual rows.

### Aggregate example

```sql
SELECT department, AVG(salary) AS average_salary
FROM employees
GROUP BY department;
```

Returns one row per department.

### Window function example

```sql
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER (
        PARTITION BY department
    ) AS department_average
FROM employees;
```

Returns individual employee rows along with their department's average salary.

**Interview takeaway:** Use `GROUP BY` to summarize groups. Use a window function when you need the aggregate result alongside individual rows.

---

## 10. Common Mistakes

1. **Using `COUNT(column)` when you need all rows.**  
   It ignores NULL values. Use `COUNT(*)` to count rows.

2. **Using `WHERE` to filter an aggregate result.**  
   Use `HAVING` when filtering based on an aggregate calculation.

3. **Forgetting NULL behavior.**  
   `AVG()` does not treat NULL as zero.

4. **Selecting a regular column without grouping it.**  
   In standard SQL, a selected column generally must be grouped or used in an aggregate function.

5. **Using `DISTINCT` unnecessarily.**  
   Add it when unique values are required, not automatically.

6. **Confusing `GROUP BY` with window functions.**  
   Grouping summarizes rows; window functions can preserve individual rows.

---

## 11. Interview Questions

Try answering these without looking at the notes.

1. What are aggregate functions?
2. What is the difference between `COUNT(*)` and `COUNT(column)`?
3. How does `AVG()` handle NULL values?
4. What is the difference between `WHERE` and `HAVING`?
5. How do you count unique departments?
6. How do you find the average salary of each department?
7. How do you find departments with more than five employees?
8. What is conditional aggregation?
9. What is the difference between `GROUP BY` and a window function?
10. What do `SUM()` and `COUNT()` return when there are no matching rows?

---

## 12. Practice Questions

Use the `employees` table defined at the beginning.

**Q1.** Find the total number of employees.

**Q2.** Count employees whose salary is not NULL.

**Q3.** Find the highest and lowest salary.

**Q4.** Calculate the total salary for each department.

**Q5.** Find departments with at least two employees.

**Q6.** Count the number of unique departments.

**Q7.** Count how many employees have a salary greater than 45,000.

**Q8.** Find the average salary for each department, considering only salaries greater than 45,000.

**Q9.** Return the total employee count and the IT employee count in one row.

**Q10.** Show each employee's salary alongside their department's average salary.

---

## 13. Quick Revision — Read Before an Interview

- `COUNT(*)` counts rows.
- `COUNT(column)` counts non-NULL values.
- `COUNT(DISTINCT column)` counts unique, non-NULL values.
- `SUM()` calculates a total.
- `AVG()` calculates an average of non-NULL values.
- `MIN()` and `MAX()` find the smallest and largest non-NULL values.
- `GROUP BY` creates groups for aggregation.
- `HAVING` filters groups.
- `WHERE` filters rows before grouping.
- Conditional aggregation lets you calculate multiple condition-based results in one query.
- Use window functions when you need aggregate results without losing individual rows.

### Test Yourself

Before marking this topic as interview-ready, make sure I can:

- [ ] Explain all five main aggregate functions.
- [ ] Explain how NULL affects aggregate results.
- [ ] Write a query using `GROUP BY` and `HAVING`.
- [ ] Use `COUNT(DISTINCT ...)`.
- [ ] Write a conditional aggregation query.
- [ ] Explain the difference between grouping and window functions.

**Final rule:** Do not just read these notes. Close them and solve the practice questions yourself.
