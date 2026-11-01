# SQL Interview Questions — Easy Level

## 1. SQL Fundamentals

### Q1. What is SQL?

SQL (Structured Query Language) is used to communicate with relational databases.

It allows us to:
- Retrieve data.
- Insert new records.
- Update existing records.
- Delete records.
- Create and modify database objects.
- Control access to database objects.

### Q2. What is a database?

A database is an organized collection of data that can be stored, accessed, managed, and updated.

A relational database stores data in tables consisting of rows and columns.

### Q3. What is a table?

A table stores related data in rows and columns.

- **Row:** Represents one record.
- **Column:** Represents one attribute.
- **Primary key:** Identifies a row uniquely.

Example: An `employees` table may contain employee ID, name, department, and salary.

### Q4. What is the difference between SQL and MySQL?

- **SQL:** A language used to work with relational databases.
- **MySQL:** A relational database management system that uses SQL.

Other SQL database systems include PostgreSQL, SQL Server, Oracle, and SQLite.

### Q5. What is an RDBMS?

RDBMS stands for Relational Database Management System.

It stores related data in tables and supports relationships through keys and constraints.

Examples:
- MySQL
- PostgreSQL
- SQL Server
- Oracle Database

---

## 2. SQL Command Categories

### Q6. What are DDL, DML, DQL, DCL, and TCL?

| Category | Full form | Purpose | Examples |
|---|---|---|---|
| DDL | Data Definition Language | Defines database structures | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| DML | Data Manipulation Language | Changes table data | `INSERT`, `UPDATE`, `DELETE` |
| DQL | Data Query Language | Retrieves data | `SELECT` |
| DCL | Data Control Language | Controls permissions | `GRANT`, `REVOKE` |
| TCL | Transaction Control Language | Controls transactions | `COMMIT`, `ROLLBACK`, `SAVEPOINT` |

Some database references classify `SELECT` under DML rather than treating DQL as a separate category. The distinction is commonly used for learning and interviews.

### Q7. What is the difference between DELETE, TRUNCATE, and DROP?

| Command | Purpose | WHERE supported? | Removes table structure? |
|---|---|---|---|
| DELETE | Removes selected rows | Yes | No |
| TRUNCATE | Removes all rows | No | No |
| DROP | Removes the database object | No | Yes |

Examples:

```sql
DELETE FROM employees
WHERE employee_id = 10;
```

```sql
TRUNCATE TABLE employees;
```

```sql
DROP TABLE employees;
```

Important:
- `DELETE` is a row-level operation and can remove selected records.
- `TRUNCATE` removes all rows, generally with less logging overhead, but exact behavior varies by database.
- `DROP` removes the table itself.
- Transactional, identity-reset, trigger, and rollback behavior varies by database system.

### Q8. What is the difference between CHAR and VARCHAR?

| CHAR | VARCHAR |
|---|---|
| Fixed-length character type | Variable-length character type |
| Suitable for consistently sized values | Suitable for varying-length values |
| May pad values to the declared length | Stores values using variable length within the declared limit |

Example:

```sql
CREATE TABLE employees (
    country_code CHAR(2),
    employee_name VARCHAR(100)
);
```

Exact storage behavior depends on the database.

---

## 3. SELECT and Filtering

### Q9. How do you retrieve all columns from a table?

```sql
SELECT *
FROM employees;
```

`*` selects all columns.

In production queries, selecting only required columns is generally better for readability and efficiency.

### Q10. How do you select specific columns?

```sql
SELECT
    employee_id,
    employee_name,
    salary
FROM employees;
```

### Q11. What is the WHERE clause?

`WHERE` filters individual rows before grouping and aggregation.

```sql
SELECT *
FROM employees
WHERE salary > 50000;
```

### Q12. What are comparison operators?

| Operator | Meaning |
|---|---|
| `=` | Equal |
| `<>` | Not equal |
| `!=` | Not equal in many databases |
| `>` | Greater than |
| `<` | Less than |
| `>=` | Greater than or equal |
| `<=` | Less than or equal |

`<>` is the standard SQL not-equal operator.

### Q13. What are AND, OR, and NOT?

These are logical operators used to combine or reverse conditions.

```sql
SELECT *
FROM employees
WHERE department = 'IT'
  AND salary > 50000;
```

```sql
SELECT *
FROM employees
WHERE department = 'IT'
   OR department = 'HR';
```

```sql
SELECT *
FROM employees
WHERE NOT department = 'HR';
```

Use parentheses when combining multiple conditions to make the intended logic clear.

### Q14. What is BETWEEN?

`BETWEEN` checks whether a value falls within an inclusive range.

```sql
SELECT *
FROM employees
WHERE salary BETWEEN 40000 AND 80000;
```

Both boundary values are included.

For timestamps, half-open ranges are often safer when filtering complete calendar periods.

### Q15. What is IN?

`IN` checks whether a value matches any value in a specified list.

```sql
SELECT *
FROM employees
WHERE department IN ('IT', 'HR', 'Finance');
```

### Q16. What is LIKE?

`LIKE` performs pattern matching.

| Wildcard | Meaning |
|---|---|
| `%` | Zero or more characters |
| `_` | Exactly one character |

Examples:

```sql
-- Names starting with A
SELECT *
FROM employees
WHERE employee_name LIKE 'A%';
```

```sql
-- Names ending with a
SELECT *
FROM employees
WHERE employee_name LIKE '%a';
```

```sql
-- Names with exactly five characters
SELECT *
FROM employees
WHERE employee_name LIKE '_____';
```

Case sensitivity depends on the database and collation.

### Q17. What is DISTINCT?

`DISTINCT` removes duplicate rows from the selected result.

```sql
SELECT DISTINCT department
FROM employees;
```

It returns each distinct department once.

### Q18. What is ORDER BY?

`ORDER BY` sorts query results.

```sql
SELECT *
FROM employees
ORDER BY salary DESC;
```

- `ASC`: Ascending order (default).
- `DESC`: Descending order.

### Q19. What is LIMIT?

`LIMIT` restricts the number of rows returned in databases such as PostgreSQL and MySQL.

```sql
SELECT *
FROM employees
ORDER BY salary DESC
LIMIT 5;
```

SQL Server commonly uses `TOP` or `OFFSET ... FETCH`, while Oracle supports `FETCH FIRST` syntax. Pagination syntax varies across databases.

---

## 4. Aggregate Functions

### Q20. What are aggregate functions?

Aggregate functions calculate a single result from multiple rows or groups of rows.

Common functions:

| Function | Purpose |
|---|---|
| `COUNT()` | Counts rows or non-NULL values |
| `SUM()` | Calculates total |
| `AVG()` | Calculates average |
| `MIN()` | Finds minimum |
| `MAX()` | Finds maximum |

Examples:

```sql
SELECT COUNT(*) AS employee_count
FROM employees;
```

```sql
SELECT SUM(salary) AS total_salary
FROM employees;
```

```sql
SELECT AVG(salary) AS average_salary
FROM employees;
```

```sql
SELECT
    MIN(salary) AS minimum_salary,
    MAX(salary) AS maximum_salary
FROM employees;
```

### Q21. What is the difference between COUNT(*) and COUNT(column)?

- `COUNT(*)` counts all rows.
- `COUNT(column)` counts only rows where that column is not NULL.

```sql
SELECT
    COUNT(*) AS total_rows,
    COUNT(department) AS non_null_departments
FROM employees;
```

### Q22. How does AVG handle NULL values?

`AVG(column)` ignores NULL values.

For example, if the values are `100, 200, NULL`, the average is:

\[
\frac{100+200}{2}=150
\]

It does not divide by three.

---

## 5. GROUP BY and HAVING

### Q23. What is GROUP BY?

`GROUP BY` groups rows that share the same values in specified columns.

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

This returns one row per department.

### Q24. What is HAVING?

`HAVING` filters groups after aggregation.

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```

### Q25. What is the difference between WHERE and HAVING?

| WHERE | HAVING |
|---|---|
| Filters individual rows | Filters groups |
| Applied before grouping | Applied after grouping |
| Cannot normally filter on an aggregate calculated at the same query level | Commonly filters aggregate results |

Example:

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employees
WHERE salary > 20000
GROUP BY department
HAVING AVG(salary) > 50000;
```

This first removes employees earning 20,000 or less, then calculates department averages and filters the resulting groups.

---

## 6. Joins

### Q26. What is a JOIN?

A JOIN combines rows from two or more tables based on a related column or join condition.

Assume:

**employees**

| employee_id | employee_name | department_id |
|---:|---|---:|
| 1 | Amit | 10 |
| 2 | Neha | 20 |
| 3 | Ravi | 10 |
| 4 | Sara | NULL |

**departments**

| department_id | department_name |
|---:|---|
| 10 | IT |
| 20 | HR |
| 30 | Finance |

### Q27. What is an INNER JOIN?

Returns only rows that have matching values in both tables.

```sql
SELECT
    e.employee_name,
    d.department_name
FROM employees e
INNER JOIN departments d
    ON e.department_id = d.department_id;
```

Amit, Neha, and Ravi appear. Sara has no matching department.

### Q28. What is a LEFT JOIN?

Returns all rows from the left table and matching rows from the right table. Unmatched right-side columns are NULL.

```sql
SELECT
    e.employee_name,
    d.department_name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id;
```

All four employees appear.

### Q29. What is a RIGHT JOIN?

Returns all rows from the right table and matching rows from the left table.

```sql
SELECT
    e.employee_name,
    d.department_name
FROM employees e
RIGHT JOIN departments d
    ON e.department_id = d.department_id;
```

All three departments appear, including Finance, which has no matching employee.

### Q30. What is a FULL OUTER JOIN?

Returns:
- Matching rows from both tables.
- Unmatched rows from the left table.
- Unmatched rows from the right table.

```sql
SELECT
    e.employee_name,
    d.department_name
FROM employees e
FULL OUTER JOIN departments d
    ON e.department_id = d.department_id;
```

Not every database supports `FULL OUTER JOIN` directly.

### Q31. What is a CROSS JOIN?

Returns the Cartesian product: every row from the first table is combined with every row from the second table.

```sql
SELECT
    e.employee_name,
    d.department_name
FROM employees e
CROSS JOIN departments d;
```

With four employees and three departments, the result contains 12 rows.

### Q32. What is a SELF JOIN?

A self join joins a table to itself.

Example: Find each employee's manager.

Assume the table has a `manager_id` column referencing another employee's `employee_id`.

```sql
SELECT
    e.employee_name AS employee,
    m.employee_name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.employee_id;
```

A self join is useful for hierarchical or parent-child relationships.

---

## 7. Keys and Constraints

### Q33. What is a PRIMARY KEY?

A primary key uniquely identifies every row in a table.

Properties:
- Values must be unique.
- NULL values are not allowed.
- A table has one primary key constraint, which may contain multiple columns.

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100)
);
```

### Q34. What is a FOREIGN KEY?

A foreign key establishes a relationship between tables by referencing a candidate key, commonly a primary key, in another table.

```sql
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(100)
);

CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
);
```

A foreign key helps maintain referential integrity.

### Q35. What is a UNIQUE constraint?

A UNIQUE constraint prevents duplicate values in the constrained column or column combination.

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    email VARCHAR(150) UNIQUE
);
```

NULL handling under UNIQUE constraints varies by database.

### Q36. What is NOT NULL?

It ensures that a column cannot contain NULL values.

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100) NOT NULL
);
```

### Q37. What is CHECK?

A CHECK constraint enforces a condition on inserted or updated values.

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    salary DECIMAL(10,2)
        CHECK (salary >= 0)
);
```

### Q38. What is DEFAULT?

A DEFAULT constraint supplies a value when an INSERT statement does not provide one for that column.

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    status VARCHAR(20) DEFAULT 'ACTIVE'
);
```

An explicitly supplied NULL is not automatically replaced by the default.

---

## 8. NULL Handling

### Q39. What is NULL?

NULL represents missing, unknown, or unavailable information. It is not the same as zero or an empty string.

### Q40. Why does `= NULL` not work?

SQL uses three-valued logic: TRUE, FALSE, and UNKNOWN.

Comparisons such as `column = NULL` evaluate to UNKNOWN rather than TRUE.

Use:

```sql
SELECT *
FROM employees
WHERE department_id IS NULL;
```

For non-NULL values:

```sql
SELECT *
FROM employees
WHERE department_id IS NOT NULL;
```

### Q41. What is COALESCE()?

`COALESCE()` returns the first non-NULL expression.

```sql
SELECT
    employee_name,
    COALESCE(department, 'Unassigned') AS department
FROM employees;
```

### Q42. What is NULLIF()?

`NULLIF(a, b)` returns NULL if the two expressions are equal; otherwise, it returns the first expression.

Example: Prevent division by zero.

```sql
SELECT
    total_sales / NULLIF(total_orders, 0) AS average_order_value
FROM sales_summary;
```

If `total_orders` is zero, the denominator becomes NULL.

---

## 9. Basic SQL Functions

### Q43. What are string functions?

String functions manipulate text values.

Common examples:

```sql
SELECT UPPER('data engineer');
```

```sql
SELECT LOWER('SQL INTERVIEW');
```

```sql
SELECT LENGTH('Database');
```

```sql
SELECT TRIM('  SQL  ');
```

```sql
SELECT CONCAT('Data', ' ', 'Engineer');
```

Function names and behavior can vary. For example, SQL Server commonly uses `LEN()` rather than `LENGTH()`.

### Q44. What are numeric functions?

Common numeric functions include:

```sql
SELECT ROUND(123.456, 2);
```

```sql
SELECT ABS(-100);
```

```sql
SELECT CEIL(10.2);
```

```sql
SELECT FLOOR(10.8);
```

Some databases use `CEILING()` instead of `CEIL()`.

### Q45. What are date functions?

Date functions help retrieve, compare, and manipulate dates.

Examples using common SQL concepts:

```sql
SELECT CURRENT_DATE;
```

```sql
SELECT CURRENT_TIMESTAMP;
```

```sql
SELECT EXTRACT(YEAR FROM CURRENT_DATE);
```

Date arithmetic and function syntax differ across database engines.

### Q46. What is CASE WHEN?

`CASE` implements conditional logic in SQL.

```sql
SELECT
    employee_name,
    salary,
    CASE
        WHEN salary >= 90000 THEN 'High'
        WHEN salary >= 60000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_category
FROM employees;
```

The conditions are checked in order. The first matching branch is returned.

---

## 10. Subqueries

### Q47. What is a subquery?

A subquery is a query nested inside another SQL statement.

```sql
SELECT *
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

This returns employees earning above the overall average salary.

### Q48. What is the difference between a single-row and multi-row subquery?

- **Single-row subquery:** Returns one row, often used with operators such as `=`, `>`, or `<`.
- **Multi-row subquery:** Returns multiple rows, commonly used with `IN`, `ANY`, or `ALL`.

### Q49. What is EXISTS?

`EXISTS` checks whether a subquery returns at least one row.

```sql
SELECT *
FROM departments d
WHERE EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.department_id
);
```

This returns departments with at least one employee.

### Q50. What is the difference between IN and EXISTS?

- `IN` checks whether a value belongs to a set of returned values.
- `EXISTS` checks whether at least one matching row exists.

The optimizer may produce similar execution plans for equivalent queries. Performance depends on the database, data distribution, indexes, and query structure.

---

## 11. Basic Data Modification

### Q51. How do you insert a record?

```sql
INSERT INTO employees (
    employee_id,
    employee_name,
    department_id
)
VALUES (
    5,
    'John',
    20
);
```

### Q52. How do you update a record?

```sql
UPDATE employees
SET department_id = 10
WHERE employee_id = 5;
```

Always check the WHERE condition before executing an UPDATE.

### Q53. How do you delete a record?

```sql
DELETE FROM employees
WHERE employee_id = 5;
```

Without a WHERE condition, the statement can delete every row in the table.

### Q54. What is INSERT INTO ... SELECT?

It inserts the result of a query into another table.

```sql
INSERT INTO employee_backup (
    employee_id,
    employee_name
)
SELECT
    employee_id,
    employee_name
FROM employees;
```

The source and destination columns must have compatible types and matching positions.

---

## 12. Easy SQL Coding Problems

Use this sample table for practice.

### `employees`

| employee_id | employee_name | department | salary |
|---:|---|---|---:|
| 1 | Amit | IT | 80000 |
| 2 | Neha | IT | 95000 |
| 3 | Ravi | HR | 60000 |
| 4 | Sara | HR | 75000 |
| 5 | John | Finance | 50000 |

### Problem 1. Find all employees.

```sql
SELECT *
FROM employees;
```

### Problem 2. Find employees earning more than 70,000.

```sql
SELECT *
FROM employees
WHERE salary > 70000;
```

### Problem 3. Find the highest salary.

```sql
SELECT MAX(salary) AS highest_salary
FROM employees;
```

### Problem 4. Find the lowest salary.

```sql
SELECT MIN(salary) AS lowest_salary
FROM employees;
```

### Problem 5. Calculate the average salary.

```sql
SELECT AVG(salary) AS average_salary
FROM employees;
```

### Problem 6. Count employees in each department.

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

### Problem 7. Find employees from the IT department.

```sql
SELECT *
FROM employees
WHERE department = 'IT';
```

### Problem 8. Sort employees by salary from highest to lowest.

```sql
SELECT *
FROM employees
ORDER BY salary DESC;
```

### Problem 9. Find the second-highest distinct salary.

```sql
SELECT MAX(salary) AS second_highest_salary
FROM employees
WHERE salary < (
    SELECT MAX(salary)
    FROM employees
);
```

If there is no second distinct salary, the result is NULL.

### Problem 10. Find employees earning more than the average salary.

```sql
SELECT *
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

### Problem 11. Find the highest-paid employee in each department.

```sql
SELECT
    department,
    MAX(salary) AS highest_salary
FROM employees
GROUP BY department;
```

This returns the highest salary per department, not necessarily the employee name. To return employee names, use a join or a window function.

### Problem 12. Find departments with more than one employee.

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 1;
```

### Problem 13. Find employees whose names start with S.

```sql
SELECT *
FROM employees
WHERE employee_name LIKE 'S%';
```

### Problem 14. Find employees without a department.

```sql
SELECT *
FROM employees
WHERE department IS NULL;
```

### Problem 15. Find the total salary paid by each department.

```sql
SELECT
    department,
    SUM(salary) AS total_salary
FROM employees
GROUP BY department;
```

---

## 13. Common Mistakes

1. Using `= NULL` instead of `IS NULL`.
2. Forgetting the WHERE clause in UPDATE or DELETE statements.
3. Using `WHERE` instead of `HAVING` for aggregate filtering.
4. Confusing `COUNT(*)` with `COUNT(column)`.
5. Assuming `BETWEEN` excludes its boundary values.
6. Forgetting that `DISTINCT` applies to the complete selected row combination.
7. Assuming every database supports the same SQL functions.
8. Using `SELECT *` unnecessarily in production queries.
9. Assuming a JOIN always returns one row per key.
10. Confusing a primary key with a foreign key.
11. Treating NULL as zero or an empty string.
12. Assuming `LIMIT` is supported by every SQL database.
13. Forgetting to consider duplicate values when finding the second-highest salary.

---

## 14. Quick Revision

| Topic | Remember |
|---|---|
| SQL | Language for relational databases |
| RDBMS | Software that manages relational databases |
| Table | Rows and columns |
| Primary key | Uniquely identifies a row |
| Foreign key | References a key in another table |
| DDL | Defines database objects |
| DML | Changes data |
| DQL | Retrieves data |
| DCL | Controls permissions |
| TCL | Controls transactions |
| WHERE | Filters rows |
| GROUP BY | Creates groups |
| HAVING | Filters groups |
| ORDER BY | Sorts results |
| DISTINCT | Removes duplicate result rows |
| INNER JOIN | Returns matching rows |
| LEFT JOIN | Keeps every left-side row |
| RIGHT JOIN | Keeps every right-side row |
| FULL OUTER JOIN | Keeps matching and unmatched rows |
| CROSS JOIN | Cartesian product |
| SELF JOIN | Joins a table to itself |
| COUNT(*) | Counts rows |
| COUNT(column) | Counts non-NULL values |
| COALESCE | First non-NULL value |
| NULLIF | Returns NULL when values are equal |
| CASE | Conditional logic |
| Subquery | Query inside another query |

## 15. Practice Checklist

- [ ] Explain SQL and RDBMS.
- [ ] Explain DDL, DML, DQL, DCL, and TCL.
- [ ] Explain DELETE, TRUNCATE, and DROP.
- [ ] Write SELECT queries with WHERE conditions.
- [ ] Use LIKE, IN, and BETWEEN.
- [ ] Use aggregate functions.
- [ ] Explain GROUP BY and HAVING.
- [ ] Explain all major JOIN types.
- [ ] Understand primary and foreign keys.
- [ ] Handle NULL values correctly.
- [ ] Use CASE WHEN and COALESCE.
- [ ] Write basic subqueries.
- [ ] Find the highest and second-highest salary.
- [ ] Find department-wise employee counts.
- [ ] Find employees earning above the average salary.

**Interview advice:** For easy SQL questions, focus on writing queries correctly without hesitation. Understand what each clause does and the order in which SQL logically processes a query. Build accuracy with basic filtering, joins, grouping, and subqueries before moving to advanced SQL patterns.