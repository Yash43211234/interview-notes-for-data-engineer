# SQL Subqueries

## 1. What Is a Subquery?

A **subquery** is a SQL query written inside another SQL query. The inner query provides a result that the outer query uses to retrieve, filter, or calculate data.

Subqueries are commonly used with:
- `WHERE`
- `FROM`
- `SELECT`
- `HAVING`
- Comparison operators such as `=`, `>`, and `<`
- Operators such as `IN`, `EXISTS`, and `ANY`/`ALL` where supported

### Basic syntax

```sql
SELECT column_name
FROM table_name
WHERE column_name operator (
    SELECT column_name
    FROM another_table
);
```

**Simple way to remember:** The inner query produces a result, and the outer query uses that result.

---

## 2. Sample Data

We will use the following `employees` table throughout these notes.

| employee_id | employee_name | department | salary | manager_id |
|---:|---|---|---:|---:|
| 1 | Amit | IT | 80000 | NULL |
| 2 | Neha | IT | 60000 | 1 |
| 3 | Ravi | HR | 50000 | 1 |
| 4 | Priya | IT | 45000 | 2 |
| 5 | John | Sales | 40000 | 2 |
| 6 | Sara | HR | 55000 | 3 |

---

## 3. Single-Row Subquery

A single-row subquery returns one row, usually one value. You can use comparison operators such as `=`, `>`, `<`, `>=`, and `<=`.

### Example: Find employees earning more than the average salary

```sql
SELECT employee_name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

**How it works:**

1. The inner query calculates the average salary.
2. The outer query returns employees whose salaries are greater than that average.

The average salary is 55,000, so the result is:

| employee_name | salary |
|---|---:|
| Amit | 80000 |
| Neha | 60000 |

**Important:** A comparison such as `salary = (subquery)` expects the subquery to return a single value. If it returns multiple rows, many databases raise an error.

---

## 4. Multi-Row Subquery

A multi-row subquery returns multiple rows. Operators such as `IN`, `ANY`, and `ALL` can be used depending on the requirement and database.

### Example: Find employees working in departments that have an employee earning more than 60,000

```sql
SELECT employee_name, department
FROM employees
WHERE department IN (
    SELECT department
    FROM employees
    WHERE salary > 60000
);
```

The inner query returns the department `IT`. The outer query returns all employees in IT.

| employee_name | department |
|---|---|
| Amit | IT |
| Neha | IT |
| Priya | IT |

**Why use `IN`?** It checks whether a value matches any value returned by the subquery.

---

## 5. Subquery with `IN` and `NOT IN`

### `IN`

Returns rows whose value matches one of the values returned by the subquery.

```sql
SELECT employee_name
FROM employees
WHERE department IN (
    SELECT department
    FROM employees
    WHERE salary >= 60000
);
```

The inner query returns `IT`, so all IT employees are returned.

### `NOT IN`

Returns rows whose value does not match any value returned by the subquery.

```sql
SELECT employee_name
FROM employees
WHERE department NOT IN (
    SELECT department
    FROM employees
    WHERE salary >= 60000
);
```

This returns Ravi, John, and Sara because they are not in IT.

**Common mistake:** If the subquery used with `NOT IN` returns a `NULL`, the comparison can produce an unknown result, causing expected rows to be excluded. When checking for missing matches, `NOT EXISTS` is often a safer choice.

---

## 6. Scalar Subquery

A scalar subquery returns exactly one value: one row and one column.

### Example: Find employees earning more than the average salary

```sql
SELECT employee_name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

The `AVG()` aggregate returns one value for the whole table, so the subquery can be used as a scalar.

Scalar subqueries can also appear in the `SELECT` list.

```sql
SELECT
    employee_name,
    salary,
    (SELECT AVG(salary) FROM employees) AS avg_salary
FROM employees;
```

This displays the overall average salary beside every employee.

---

## 7. Subquery in the `FROM` Clause

A subquery in the `FROM` clause acts like a temporary result set, often called a **derived table**. In many SQL dialects, it must have an alias.

### Example: Find departments whose average salary exceeds 50,000

```sql
SELECT department, avg_salary
FROM (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
) AS dept_summary
WHERE avg_salary > 50000;
```

The inner query calculates the average salary for each department. The outer query filters those department-level results.

Expected result:

| department | avg_salary |
|---|---:|
| IT | 61666.67 |
| HR | 52500.00 |

**When useful:** Use this approach when you want to calculate a result first and filter or process that result in another query.

---

## 8. Subquery in the `SELECT` Clause

A subquery in the `SELECT` list can calculate a value for display alongside each row.

### Example: Show each employee's salary and the overall average

```sql
SELECT
    employee_name,
    salary,
    (SELECT AVG(salary) FROM employees) AS company_avg_salary
FROM employees;
```

The average is calculated by the subquery and displayed for every employee.

**Remember:** A scalar subquery in the `SELECT` list must return a single value for each place where it is evaluated. A subquery returning multiple columns or multiple rows cannot be used as a scalar expression.

---

## 9. Correlated Subquery

A **correlated subquery** refers to a column from the outer query. Its result depends on the current row being processed by the outer query.

### Example: Find employees earning more than their department's average salary

```sql
SELECT e.employee_name, e.department, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department = e.department
);
```

**How it works:**

1. The outer query considers an employee.
2. The inner query calculates the average salary for that employee's department.
3. The outer query keeps the employee if their salary is greater than that average.

Expected result:

| employee_name | department | salary |
|---|---|---:|
| Amit | IT | 80000 |
| Sara | HR | 55000 |

**Key difference:** A non-correlated subquery can be evaluated independently of the outer query. A correlated subquery depends on values from the outer query. The optimizer may transform either query, so do not assume a fixed execution count.

---

## 10. `EXISTS` and `NOT EXISTS`

These operators test whether a subquery returns at least one row.

### `EXISTS`

Example: Find employees whose department has at least one employee earning more than 70,000.

```sql
SELECT e.employee_name, e.department
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM employees e2
    WHERE e2.department = e.department
      AND e2.salary > 70000
);
```

Only the IT department meets the condition, so all IT employees are returned.

### `NOT EXISTS`

Example: Find employees who do not manage any employee.

```sql
SELECT e.employee_name
FROM employees e
WHERE NOT EXISTS (
    SELECT 1
    FROM employees e2
    WHERE e2.manager_id = e.employee_id
);
```

This returns Priya and John because no employee has either of their employee IDs as their manager ID.

**Important:** `SELECT 1` is a common convention inside `EXISTS`; the actual selected value is not what determines whether a matching row exists.

---

## 11. `ANY` and `ALL`

These operators compare a value against the values returned by a subquery. Their exact syntax and support can vary by database.

### `ANY`

The condition is true if the comparison is true for at least one returned value.

Example: Find employees earning more than at least one HR employee's salary.

```sql
SELECT employee_name, salary
FROM employees
WHERE salary > ANY (
    SELECT salary
    FROM employees
    WHERE department = 'HR'
);
```

HR salaries are 50,000 and 55,000. This condition is equivalent to earning more than at least one of those salaries, so earning more than the minimum HR salary is sufficient.

### `ALL`

The condition is true if the comparison is true for every returned value.

Example: Find employees earning more than every HR employee.

```sql
SELECT employee_name, salary
FROM employees
WHERE salary > ALL (
    SELECT salary
    FROM employees
    WHERE department = 'HR'
);
```

This requires a salary greater than 55,000, the highest HR salary.

**Interview tip:** For a nonempty set of non-NULL values, `> ANY` corresponds to being greater than at least one value, while `> ALL` requires being greater than every value. `NULL` values and empty result sets can affect the outcome.

---

## 12. Subquery vs. Join

Both subqueries and joins can solve overlapping problems, but they express different ideas.

| Feature | Subquery | Join |
|---|---|---|
| Main purpose | Use one query's result in another query | Combine rows from tables or table expressions |
| Common use | Filtering by an aggregate or testing existence | Returning related columns from multiple tables |
| Can return columns from both sources? | Depends on query structure | Yes |
| Always slower or faster? | No | No |

### Example: Find employees working in departments with salaries above 60,000

Using a subquery:

```sql
SELECT employee_name, department
FROM employees
WHERE department IN (
    SELECT department
    FROM employees
    WHERE salary > 60000
);
```

Using a join:

```sql
SELECT DISTINCT e.employee_name, e.department
FROM employees e
JOIN employees high_paid
    ON e.department = high_paid.department
WHERE high_paid.salary > 60000;
```

Both return the IT employees. `DISTINCT` prevents duplicate employee rows if multiple employees in the matched department meet the salary condition.

**Performance rule:** Do not assume subqueries are always slower than joins. The optimizer, indexes, data size, and query structure affect performance. Check the execution plan when performance matters.

---

## 13. Common Mistakes

1. **Using `=` when a subquery returns multiple rows.** Use `IN`, `ANY`, `ALL`, or redesign the query as needed.
2. **Forgetting an alias for a derived table.** Some databases require an alias for subqueries in `FROM`.
3. **Ignoring `NULL` behavior.** Comparisons involving `NULL` can evaluate to unknown.
4. **Using `NOT IN` without checking for `NULL`.** Consider `NOT EXISTS` when checking for missing matches.
5. **Writing a correlated condition incorrectly.** Make sure the inner query refers to the intended outer-query column.
6. **Assuming subqueries are always slower.** Compare execution plans and actual performance.
7. **Using a subquery when a simpler expression is clearer.** Choose the approach that is easiest to understand and maintain.

---

## 14. Interview Questions

### Q1. What is a subquery?

A query nested inside another SQL query. Its result is used by the outer query.

### Q2. What is the difference between a single-row and multi-row subquery?

A single-row subquery returns one row; a multi-row subquery returns multiple rows. The operator must be appropriate for the result.

### Q3. What is a scalar subquery?

A subquery that returns one value: one row and one column.

### Q4. What is a correlated subquery?

A subquery that references a column from its outer query and depends on the current outer row.

### Q5. What is the difference between `IN` and `EXISTS`?

`IN` checks whether a value matches a value in a result set. `EXISTS` checks whether the subquery returns any row.

### Q6. What is the difference between `NOT IN` and `NOT EXISTS`?

Both can be used to find non-matching rows, but their behavior differs when `NULL` is present. `NOT EXISTS` is often safer for anti-match checks.

### Q7. Can a subquery be used in the `FROM` clause?

Yes. It acts as a derived table and generally needs an alias.

### Q8. Can a subquery return multiple columns?

Yes, in contexts that support multi-column results, such as a derived table or certain row-value comparisons. A scalar subquery must return one column and one row.

### Q9. Are subqueries always slower than joins?

No. Performance depends on the query, database optimizer, indexes, and data distribution.

### Q10. How would you find employees earning more than their department's average?

Use a correlated subquery comparing each employee's salary with the average salary of their department.

---

## 15. Practice Questions

Use the `employees` table from Section 2.

### Beginner

- [ ] Find employees earning more than the overall average salary.
- [ ] Find employees working in the IT department using a subquery.
- [ ] Display every employee alongside the overall average salary.
- [ ] Find employees whose department has at least one employee earning above 60,000.

### Intermediate

- [ ] Find employees earning more than their department's average salary.
- [ ] Find employees earning more than every employee in HR.
- [ ] Find employees who do not manage anyone.
- [ ] Find departments whose average salary exceeds 50,000 using a derived table.

### Advanced

- [ ] Find the second-highest distinct salary using a subquery.
- [ ] Find employees who earn the highest salary in their department.
- [ ] Find employees earning more than their manager.
- [ ] Find departments with no employees earning above 60,000.

For every solution, explain why you selected `IN`, `EXISTS`, a comparison operator, or a correlated subquery.

---

## 16. Quick Revision

| Concept | Remember |
|---|---|
| Subquery | Query inside another query |
| Single-row subquery | Returns one row; comparison operators can be used |
| Scalar subquery | Returns one value |
| Multi-row subquery | Returns multiple rows |
| `IN` | Checks membership in a result set |
| `EXISTS` | Checks whether at least one row exists |
| `NOT EXISTS` | Checks whether no matching row exists |
| Correlated subquery | Refers to the outer query |
| `FROM` subquery | Creates a derived table |
| `ANY` | Comparison must be true for at least one value |
| `ALL` | Comparison must be true for every value |
| Performance | Check the execution plan; do not assume |

## 17. Self-Check

Before considering this topic complete, make sure you can:

- [ ] Explain what a subquery is in one or two sentences.
- [ ] Identify single-row, multi-row, scalar, and correlated subqueries.
- [ ] Write a query using `IN`, `EXISTS`, and `NOT EXISTS`.
- [ ] Explain how `NULL` can affect `NOT IN`.
- [ ] Use a subquery in `SELECT`, `WHERE`, and `FROM`.
- [ ] Solve a department-average salary problem.
- [ ] Explain when a join may be clearer than a subquery.

**Final takeaway:** Understand the result returned by the inner query, choose an operator that matches that result, and pay attention to `NULL` values and correlation.