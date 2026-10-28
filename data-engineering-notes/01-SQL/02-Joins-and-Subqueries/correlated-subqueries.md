# Correlated Subqueries in SQL

## 1. What Is a Correlated Subquery?

A **correlated subquery** is a subquery that depends on a value from the outer query.

The inner query references a column from the outer query, so its result depends on the current row being processed by the outer query.

**Simple definition:** The inner query uses information from the outer query to perform its calculation or filtering.

## 2. Sample Tables

Consider these two tables.

### Employees

| employee_id | name | department_id | salary |
|---:|---|---:|---:|
| 1 | Amit | 10 | 60000 |
| 2 | Neha | 10 | 45000 |
| 3 | Ravi | 20 | 50000 |
| 4 | Priya | 20 | 35000 |
| 5 | John | 30 | 40000 |

### Departments

| department_id | department_name |
|---:|---|
| 10 | IT |
| 20 | HR |
| 30 | Sales |

## 3. First Correlated Subquery Example

**Question:** Find employees who earn more than the average salary in their own department.

```sql
SELECT e.name, e.department_id, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

Result:

| name | department_id | salary |
|---|---:|---:|
| Amit | 10 | 60000 |
| Ravi | 20 | 50000 |

### How does it work?

1. The outer query considers an employee.
2. The inner query calculates the average salary for that employee's department.
3. The condition compares the employee's salary with that average.
4. The outer query returns the employee if the condition is true.

For Amit, the average salary in department 10 is \((60000 + 45000) / 2 = 52500\). Since 60,000 is greater than 52,500, Amit is returned.

For Neha, the same department average is 52,500. Since 45,000 is not greater than 52,500, Neha is excluded.

**Key point:** The condition `e2.department_id = e.department_id` connects the inner query to the current outer row. That makes the subquery correlated.

## 4. Understand the Table Aliases

```sql
FROM employees e
```

`e` is the alias for the employee row in the outer query.

```sql
FROM employees e2
```

`e2` is the alias for the employees used inside the inner query.

```sql
WHERE e2.department_id = e.department_id
```

This matches the inner-query employees to the department of the current outer-query employee.

Without this relationship, the inner query would not calculate a separate department average for each employee.

## 5. Correlated vs. Non-Correlated Subqueries

| Correlated subquery | Non-correlated subquery |
|---|---|
| References a column from the outer query | Does not depend on the outer query |
| Result depends on the current outer row | Can be evaluated independently of the outer row |
| Often used for row-specific comparisons | Often used for independent calculations or lookups |
| Example: compare salary with the employee's department average | Example: compare salary with the overall company average |

### Non-correlated example

Find employees earning more than the overall company average.

```sql
SELECT name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

The inner query calculates one company-wide average. It does not reference a column from the outer query, so it is non-correlated.

### Correlated version

Find employees earning more than their department's average.

```sql
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

The inner query uses the outer employee's department to calculate the comparison value.

## 6. Correlated Subquery with EXISTS

`EXISTS` checks whether a subquery returns at least one row.

**Question:** Find departments that have at least one employee earning more than 50,000.

```sql
SELECT d.department_id, d.department_name
FROM departments d
WHERE EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.department_id
      AND e.salary > 50000
);
```

Result:

| department_id | department_name |
|---:|---|
| 10 | IT |

### How it works

- The outer query considers each department.
- The inner query checks whether that department has an employee earning more than 50,000.
- If at least one matching row exists, `EXISTS` is true.
- The department is included in the result.

`SELECT 1` is a common convention inside `EXISTS`. The important thing is whether a row exists, not the selected value.

## 7. Correlated Subquery with NOT EXISTS

`NOT EXISTS` checks that no matching row exists.

**Question:** Find departments that have no employees.

```sql
SELECT d.department_id, d.department_name
FROM departments d
WHERE NOT EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.department_id
);
```

With the sample data, this returns no rows because all three departments have employees.

**Common use case:** Find customers who have never placed an order, products that have never been sold, or departments without employees.

## 8. Correlated Subquery with UPDATE

Correlated subqueries can also be used in data modification statements.

Example: Set each employee's salary to their department's average salary.

```sql
UPDATE employees e
SET salary = (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

**Warning:** This changes every employee's salary and calculates averages using the values present when the statement is evaluated. Do not execute an update like this on real data without authorization, a transaction or rollback plan, and a verified `WHERE` condition if only certain rows should change. Exact alias syntax varies by database.

For interview preparation, focus first on understanding correlated `SELECT` queries.

## 9. Performance Considerations

A correlated subquery is often explained as running for each row of the outer query. This is a useful mental model, but it does **not** mean every database always executes it literally that way.

A database optimizer may transform the query or use other execution strategies.

For large tables:

- Make sure the correlation condition is correct.
- Check indexes on columns used in correlation conditions when appropriate.
- Avoid unnecessary repeated calculations.
- Compare the execution plan if performance matters.
- Consider a join, aggregation, or window function when it expresses the requirement more clearly.

For example, the average salary per department can also be calculated with a window function:

```sql
SELECT name, department_id, salary
FROM (
    SELECT
        name,
        department_id,
        salary,
        AVG(salary) OVER (
            PARTITION BY department_id
        ) AS department_avg_salary
    FROM employees
) t
WHERE salary > department_avg_salary;
```

Both approaches answer the same question for this sample data. The better choice depends on readability, database support, and the execution plan.

## 10. Common Mistakes

### Mistake 1: Forgetting the correlation condition

Incorrect for finding employees above their own department's average:

```sql
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary)
    FROM employees
);
```

This compares each employee's salary with the overall company average, not the department average.

Correct:

```sql
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

### Mistake 2: Confusing inner and outer aliases

Always identify which alias belongs to which query. In the example above, `e` is the outer employee and `e2` is the inner employee.

### Mistake 3: Assuming correlated subqueries are always slow

Their performance depends on the database optimizer, table size, indexes, and query structure. Test performance rather than relying on a blanket rule.

### Mistake 4: Using NOT IN without considering NULL

`NOT IN` can behave unexpectedly if the subquery returns a `NULL` value. `NOT EXISTS` is often a safer choice for checking that no related row exists.

### Mistake 5: Using a scalar subquery that returns multiple rows

A comparison such as `salary > (subquery)` generally requires the inner query to return a single value. If it returns multiple rows, the database may raise an error. Use `IN`, `EXISTS`, or an appropriate aggregate when the requirement calls for multiple rows.

## 11. Interview Questions

**Q1. What is a correlated subquery?**

A subquery that references a column from the outer query and depends on the current outer row.

**Q2. What is the difference between correlated and non-correlated subqueries?**

A correlated subquery depends on the outer query. A non-correlated subquery can be evaluated independently of it.

**Q3. Does a correlated subquery always execute once per outer row?**

Not necessarily. That is a useful conceptual model, but the optimizer may transform the query or use a different execution strategy.

**Q4. Can EXISTS be used with a correlated subquery?**

Yes. It is commonly used to check whether related rows exist.

**Q5. What is the difference between EXISTS and NOT EXISTS?**

`EXISTS` is true when at least one matching row exists. `NOT EXISTS` is true when no matching row exists.

**Q6. How would you find employees earning more than their department's average salary?**

Use a correlated subquery to calculate the average salary for the current employee's department, then compare the employee's salary with that average.

**Q7. Can a correlated subquery be replaced with a join or window function?**

Sometimes. The best alternative depends on the problem, the desired result, and the database.

**Q8. What is the purpose of table aliases in correlated subqueries?**

They distinguish the outer row from the rows referenced inside the subquery.

## 12. Practice Questions

Using the `employees` and `departments` tables above:

1. Find employees who earn less than their department's average salary.
2. Find employees who earn the highest salary in their department.
3. Find employees whose salary is greater than the highest salary in department 20.
4. Find departments that have at least one employee earning more than 45,000 using `EXISTS`.
5. Find departments that have no employees using `NOT EXISTS`.
6. Find employees who earn more than the average salary across the whole company. Use a non-correlated subquery.
7. Find employees who earn more than their department's average salary. Use a correlated subquery.
8. Rewrite Question 7 using a window function.

**Hints:**

- Questions 1 and 7 need a department-specific average.
- Question 2 can use a correlated `MAX()` subquery.
- Questions 4 and 5 are natural uses of `EXISTS` and `NOT EXISTS`.

## 13. Quick Revision

- A correlated subquery references a column from the outer query.
- Its result depends on the current outer row.
- Table aliases help distinguish inner and outer query references.
- `EXISTS` checks whether matching rows exist.
- `NOT EXISTS` checks whether matching rows do not exist.
- A non-correlated subquery can be evaluated independently of the outer query.
- Correlated subqueries are not automatically slow; the execution plan matters.
- Window functions, joins, and aggregations can sometimes provide alternatives.

**Remember this pattern:**

```sql
SELECT outer_table.column
FROM table_name outer_table
WHERE outer_table.value > (
    SELECT AGGREGATE(inner_table.value)
    FROM table_name inner_table
    WHERE inner_table.group_id = outer_table.group_id
);
```

The key is the correlation condition that connects the inner query to the current outer row.

## 14. Self-Check

- [ ] I can explain correlated subqueries in simple language.
- [ ] I can distinguish correlated and non-correlated subqueries.
- [ ] I can write a correlated subquery using `AVG()` or `MAX()`.
- [ ] I can use `EXISTS` and `NOT EXISTS`.
- [ ] I understand why aliases matter.
- [ ] I can explain the potential performance considerations.
- [ ] I can solve the practice questions without looking at the examples.

**Revision status:** Needs Revision / Practicing / Interview Ready