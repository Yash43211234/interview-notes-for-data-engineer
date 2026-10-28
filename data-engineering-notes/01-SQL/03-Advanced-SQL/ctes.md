# SQL Common Table Expressions (CTEs)

## 1. What Is a CTE?

A **Common Table Expression (CTE)** is a temporary named result set that you define using the `WITH` keyword and use within a single SQL statement.

CTEs help you:
- Break complex queries into smaller steps.
- Improve readability and maintainability.
- Reuse a named result set within the same statement.
- Organize filtering, aggregation, and joins.
- Solve problems involving rankings, running totals, and hierarchical data.

### Basic syntax

```sql
WITH cte_name AS (
    SELECT column1, column2
    FROM table_name
    WHERE condition
)
SELECT *
FROM cte_name;
```

**Remember:** Define the CTE using `WITH`, then reference its name in the main query.

A regular CTE is not a permanent table. Its scope is the single SQL statement in which it is defined.

---

## 2. Sample Data

We will use this `employees` table throughout the notes.

| employee_id | employee_name | department | salary | manager_id |
|---:|---|---|---:|---:|
| 1 | Amit | IT | 80000 | NULL |
| 2 | Neha | IT | 60000 | 1 |
| 3 | Ravi | HR | 50000 | 1 |
| 4 | Priya | IT | 45000 | 2 |
| 5 | John | Sales | 40000 | 2 |
| 6 | Sara | HR | 55000 | 3 |

---

## 3. Basic CTE

### Example: Find employees earning more than 50,000

```sql
WITH high_salary AS (
    SELECT employee_id, employee_name, salary
    FROM employees
    WHERE salary > 50000
)
SELECT *
FROM high_salary;
```

Expected result:

| employee_id | employee_name | salary |
|---:|---|---:|
| 1 | Amit | 80000 |
| 2 | Neha | 60000 |
| 6 | Sara | 55000 |

### How it works

1. The CTE selects employees earning more than 50,000.
2. The main query reads the result named `high_salary`.
3. The final output contains the selected employees.

**Interview point:** A CTE is primarily a query-organization feature. It does not automatically make a query faster.

---

## 4. CTE with Aggregation

CTEs are useful when you want to calculate a summary first and then filter or analyze that summary.

### Example: Find departments with an average salary above 50,000

```sql
WITH department_salary AS (
    SELECT
        department,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT department, avg_salary
FROM department_salary
WHERE avg_salary > 50000;
```

Expected result:

| department | avg_salary |
|---|---:|
| IT | 61666.67 |
| HR | 52500.00 |

### Why use a CTE here?

The query has two clear stages:

- Calculate the average salary for each department.
- Filter departments based on their average.

This makes the logic easier to read and debug.

---

## 5. Multiple CTEs in One Query

You can define multiple CTEs separated by commas.

### Example: Find employees earning above their department's average

```sql
WITH department_avg AS (
    SELECT
        department,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
),
employee_details AS (
    SELECT employee_id, employee_name, department, salary
    FROM employees
)
SELECT
    e.employee_name,
    e.department,
    e.salary,
    d.avg_salary
FROM employee_details e
JOIN department_avg d
    ON e.department = d.department
WHERE e.salary > d.avg_salary;
```

Expected result:

| employee_name | department | salary | avg_salary |
|---|---|---:|---:|
| Amit | IT | 80000 | 61666.67 |
| Sara | HR | 55000 | 52500.00 |

**Important:** A later CTE can reference an earlier CTE in the same `WITH` clause in standard usage. A CTE generally cannot reference a later CTE unless the database supports a specific feature that allows it.

---

## 6. CTE vs. Subquery

Both can express similar logic, but a CTE can make multi-step queries easier to understand.

### Using a subquery

```sql
SELECT department, avg_salary
FROM (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
) AS dept_summary
WHERE avg_salary > 50000;
```

### Using a CTE

```sql
WITH dept_summary AS (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT department, avg_salary
FROM dept_summary
WHERE avg_salary > 50000;
```

Both queries return the same result.

| Feature | CTE | Subquery |
|---|---|---|
| Readability | Useful for naming query steps | Can become difficult to read when deeply nested |
| Reuse | Can be referenced multiple times in the same statement | A nested result is generally used where it is written |
| Scope | One SQL statement | The containing query expression |
| Performance | Depends on the database and query plan | Depends on the database and query plan |

**Interview takeaway:** Choose a CTE when naming intermediate steps makes the query clearer. Do not assume that replacing a subquery with a CTE automatically improves performance.

---

## 7. CTE vs. Temporary Table

A CTE and a temporary table are different.

| Feature | CTE | Temporary table |
|---|---|---|
| Lifetime | One SQL statement | Usually lasts for a session or until dropped, depending on the database |
| Creation | Defined with `WITH` | Created using database-specific temporary-table syntax |
| Storage | Often handled as part of query execution; implementation varies | Usually materialized and stored by the database |
| Indexing | Cannot generally be indexed directly as a CTE | May support indexes, depending on the database |
| Best use | Organizing one query | Reusing intermediate data across statements or managing larger multi-step workflows |

A CTE does not guarantee materialization. Some databases may inline or optimize it; others may provide materialization options.

---

## 8. CTE with Window Functions

CTEs are often combined with window functions to rank rows and filter results.

### Example: Find the highest-paid employee in each department

```sql
WITH ranked_employees AS (
    SELECT
        employee_name,
        department,
        salary,
        ROW_NUMBER() OVER (
            PARTITION BY department
            ORDER BY salary DESC, employee_id
        ) AS rn
    FROM employees
)
SELECT employee_name, department, salary
FROM ranked_employees
WHERE rn = 1;
```

Expected result:

| employee_name | department | salary |
|---|---|---:|
| Amit | IT | 80000 |
| Sara | HR | 55000 |
| John | Sales | 40000 |

### Why is a CTE useful?

In many SQL dialects, you cannot directly filter on a window-function alias in the same query block's `WHERE` clause because `WHERE` is evaluated before window functions. The CTE creates a query layer where `rn` is available for filtering.

**Important:** `ROW_NUMBER()` returns only one row per department. If you need all employees tied for the highest salary, consider `RANK()` or `DENSE_RANK()` instead.

---

## 9. Recursive CTE

A **recursive CTE** refers to itself to process hierarchical or repeated relationships, such as:
- Employee-manager hierarchies.
- Parent-child categories.
- Organizational structures.
- Generating a sequence of numbers.

Recursive syntax varies slightly by database.

### General structure

```sql
WITH RECURSIVE cte_name AS (
    -- Anchor query
    SELECT ...

    UNION ALL

    -- Recursive query
    SELECT ...
    FROM source_table
    JOIN cte_name
        ON ...
)
SELECT *
FROM cte_name;
```

The anchor query starts the result. The recursive query uses rows already produced by the CTE to find the next rows. Recursion continues until the recursive query produces no additional rows or a database limit is reached.

### Example: Generate numbers from 1 to 5

In databases supporting this `WITH RECURSIVE` syntax:

```sql
WITH RECURSIVE numbers AS (
    SELECT 1 AS n

    UNION ALL

    SELECT n + 1
    FROM numbers
    WHERE n < 5
)
SELECT n
FROM numbers;
```

Expected result:

| n |
|---:|
| 1 |
| 2 |
| 3 |
| 4 |
| 5 |

**Database note:** SQL Server uses recursive CTE syntax without the `RECURSIVE` keyword. Other dialect details also vary.

**Safety tip:** Always design a stopping condition for recursive queries. Without one, recursion may continue until a database limit or error occurs.

---

## 10. Common Mistakes

1. **Forgetting the `WITH` keyword.** A CTE must be defined before the main query.
2. **Using the CTE outside its scope.** A regular CTE is available only within its statement.
3. **Forgetting commas between multiple CTEs.** Separate each CTE definition with a comma.
4. **Using a window-function alias too early.** Use an outer query or CTE to filter its result.
5. **Assuming CTEs always improve performance.** Their main benefit is often clarity; execution depends on the database.
6. **Creating infinite recursion.** Include a valid stopping condition in recursive CTEs.
7. **Assuming a CTE is a permanent or temporary table.** It is a named query result, not a table that automatically persists.
8. **Ignoring duplicate rows.** `UNION ALL` preserves duplicates. Use `UNION` only when deduplication is required and appropriate.

---

## 11. Interview Questions

### Q1. What is a CTE?

A CTE is a named result set defined with `WITH` and available within one SQL statement.

### Q2. Why do we use CTEs?

To organize complex queries into named steps that are easier to read, debug, and maintain.

### Q3. Can we use multiple CTEs in one query?

Yes. Separate their definitions with commas.

### Q4. Can a CTE reference another CTE?

Yes, a later CTE can generally reference an earlier CTE in the same `WITH` clause.

### Q5. What is the difference between a CTE and a subquery?

They can often solve the same problem. CTEs are especially useful for naming intermediate steps and structuring multi-stage logic.

### Q6. Is a CTE faster than a subquery?

Not necessarily. Performance depends on the database optimizer, query structure, and execution plan.

### Q7. What is a recursive CTE?

A CTE that refers to itself to process hierarchical relationships or repeated steps.

### Q8. Can a CTE be used with window functions?

Yes. It is often used to calculate a ranking or running total in one query layer and filter that result in another.

### Q9. What is the difference between `UNION` and `UNION ALL` in a recursive CTE?

`UNION` removes duplicate rows, while `UNION ALL` preserves them. Recursive CTEs commonly use `UNION ALL` when duplicate removal is not required.

### Q10. When would you use a temporary table instead of a CTE?

When intermediate results need to be reused across multiple statements, or when materialization and indexing are useful for a multi-step workflow.

---

## 12. Practice Questions

Use the `employees` table from Section 2.

### Beginner

- [ ] Create a CTE that returns employees earning more than 50,000.
- [ ] Use a CTE to calculate the average salary for each department.
- [ ] Find departments whose average salary is above 50,000.

### Intermediate

- [ ] Find employees earning above their department's average salary.
- [ ] Use a CTE and `ROW_NUMBER()` to find the highest-paid employee in each department.
- [ ] Create two CTEs and join their results.
- [ ] Find the second-highest distinct salary using a CTE.

### Advanced

- [ ] Use a recursive CTE to build an employee-manager hierarchy.
- [ ] Explain how you would prevent an endlessly recursive query.
- [ ] Compare a CTE, a subquery, and a temporary table for a multi-step transformation.
- [ ] Explain how you would investigate a slow CTE-based query.

---

## 13. Quick Revision

| Concept | Remember |
|---|---|
| CTE | Named result set defined with `WITH` |
| Scope | One SQL statement |
| Multiple CTEs | Separate definitions with commas |
| Aggregation | Calculate summaries in one query layer |
| Window functions | Calculate rankings or running metrics |
| Recursive CTE | Processes hierarchical or repeated relationships |
| CTE vs. subquery | Often a readability choice |
| CTE vs. temporary table | Different scope and materialization behavior |
| Performance | Check the execution plan |
| Recursion | Always consider a stopping condition |

---

## 14. Self-Check

Before considering this topic complete, make sure you can:

- [ ] Write a basic CTE using `WITH`.
- [ ] Define and join multiple CTEs.
- [ ] Use a CTE with `GROUP BY` and `HAVING`.
- [ ] Use a CTE to filter `ROW_NUMBER()` results.
- [ ] Explain CTE vs. subquery vs. temporary table.
- [ ] Explain the purpose of recursive CTEs.
- [ ] Describe how CTE scope and performance work.
- [ ] Solve the practice questions without copying the examples.

**Final takeaway:** Use CTEs to break complex SQL into clear, named steps. Understand their scope and execution behavior rather than assuming they are automatically faster.