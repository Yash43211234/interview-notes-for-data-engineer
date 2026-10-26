# SELECT, WHERE, and ORDER BY in SQL

## 1. What Is SELECT?

`SELECT` retrieves data from a table. You can select specific columns or all columns.

### Select specific columns

```sql
SELECT name, department
FROM employees;
```

This returns only the employee names and departments.

### Select all columns

```sql
SELECT *
FROM employees;
```

`*` means all columns.

**Best practice:** Select only the columns you need instead of using `SELECT *` in production queries.

### Use column aliases

Aliases give columns more readable names.

```sql
SELECT
    name AS employee_name,
    salary AS monthly_salary
FROM employees;
```

The aliases change the output column names, not the original table columns.

---

## 2. What Is WHERE?

`WHERE` filters rows based on a condition. Only rows that satisfy the condition are returned.

Suppose the `employees` table contains:

| name | department | salary |
|---|---|---:|
| Amit | IT | 60000 |
| Neha | HR | 40000 |
| Ravi | IT | 30000 |
| Priya | Sales | 50000 |
| John | HR | NULL |

### Filter using a comparison operator

Find employees earning more than 40,000.

```sql
SELECT name, salary
FROM employees
WHERE salary > 40000;
```

Result:

| name | salary |
|---|---:|
| Amit | 60000 |
| Priya | 50000 |

### Common comparison operators

| Operator | Meaning |
|---|---|
| `=` | Equal to |
| `<>` | Not equal to |
| `!=` | Not equal to; supported by many databases |
| `>` | Greater than |
| `<` | Less than |
| `>=` | Greater than or equal to |
| `<=` | Less than or equal to |

Example:

```sql
SELECT *
FROM employees
WHERE department <> 'IT';
```

This returns employees whose department is not IT, excluding rows where `department` is `NULL` because the comparison is unknown.

---

## 3. WHERE with AND, OR, and NOT

### AND

All conditions must be true.

```sql
SELECT *
FROM employees
WHERE department = 'IT'
  AND salary > 40000;
```

Returns IT employees whose salary is greater than 40,000.

### OR

At least one condition must be true.

```sql
SELECT *
FROM employees
WHERE department = 'IT'
   OR department = 'HR';
```

Returns employees in IT or HR.

### NOT

Negates a condition.

```sql
SELECT *
FROM employees
WHERE NOT department = 'IT';
```

Returns rows where the department is known and is not IT. Rows with a `NULL` department do not pass this condition.

### Combine conditions carefully

Use parentheses when mixing `AND` and `OR`.

```sql
SELECT *
FROM employees
WHERE department = 'IT'
  AND (salary > 50000 OR salary < 35000);
```

This returns IT employees earning either more than 50,000 or less than 35,000.

**Remember:** `AND` is evaluated before `OR` in standard SQL operator precedence. Parentheses make your intention clear.

---

## 4. Common WHERE Conditions

### BETWEEN

Checks whether a value falls within an inclusive range.

```sql
SELECT *
FROM employees
WHERE salary BETWEEN 40000 AND 60000;
```

Both 40,000 and 60,000 are included.

Equivalent condition:

```sql
WHERE salary >= 40000 AND salary <= 60000
```

### IN

Checks whether a value matches any value in a list.

```sql
SELECT *
FROM employees
WHERE department IN ('IT', 'HR');
```

This is often cleaner than writing multiple `OR` conditions.

### LIKE

Matches a text pattern.

| Pattern | Meaning |
|---|---|
| `'A%'` | Starts with A |
| `'%a'` | Ends with a |
| `'%mit%'` | Contains mit |
| `'_a%'` | Second character is a |

Example:

```sql
SELECT *
FROM employees
WHERE name LIKE 'A%';
```

This finds names starting with A.

**Note:** Whether `LIKE` is case-sensitive depends on the database and its collation settings.

### IS NULL

Use `IS NULL` to find missing values.

```sql
SELECT *
FROM employees
WHERE salary IS NULL;
```

Use `IS NOT NULL` to find rows with a known salary.

```sql
SELECT *
FROM employees
WHERE salary IS NOT NULL;
```

**Important:** Do not use `salary = NULL` or `salary <> NULL`. Use `IS NULL` or `IS NOT NULL`.

---

## 5. What Is ORDER BY?

`ORDER BY` sorts the returned rows.

### Ascending order

```sql
SELECT name, salary
FROM employees
ORDER BY salary ASC;
```

`ASC` sorts from lowest to highest. It is the default sort direction in SQL.

### Descending order

```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC;
```

`DESC` sorts from highest to lowest.

### Sort by multiple columns

```sql
SELECT name, department, salary
FROM employees
ORDER BY department ASC, salary DESC;
```

SQL sorts by department first. Within each department, employees are sorted by salary from highest to lowest.

### Sort using a column alias

```sql
SELECT department, AVG(salary) AS average_salary
FROM employees
GROUP BY department
ORDER BY average_salary DESC;
```

This sorts departments by their average salary.

**Important:** The position of `NULL` values in sorted results can vary by database. Use explicit `NULLS FIRST` or `NULLS LAST` where supported if the position matters.

---

## 6. Combine SELECT, WHERE, and ORDER BY

Example: Find employees earning at least 40,000 and sort them by salary from highest to lowest.

```sql
SELECT name, department, salary
FROM employees
WHERE salary >= 40000
ORDER BY salary DESC;
```

How it works:

1. `FROM` identifies the source table.
2. `WHERE` filters employees earning at least 40,000.
3. `SELECT` determines which columns appear in the output.
4. `ORDER BY` sorts the returned rows.

This is a simplified explanation of logical processing; SQL's written order differs from its logical execution order.

---

## 7. Limit the Number of Results

`LIMIT` restricts the number of returned rows in databases that support this syntax.

Example: Find the three highest-paid employees.

```sql
SELECT name, salary
FROM employees
WHERE salary IS NOT NULL
ORDER BY salary DESC
LIMIT 3;
```

**Important:** Use `ORDER BY` with `LIMIT` when you want specific rows, such as the highest salaries. Without sorting, the selected rows are not guaranteed to be the highest-paid.

Syntax varies across databases. For example, SQL Server commonly uses `TOP` or `OFFSET ... FETCH`.

---

## 8. Logical Query Processing Order

A simplified logical processing order is:

1. `FROM` — identify the source table.
2. `WHERE` — filter rows.
3. `GROUP BY` — form groups, if used.
4. `HAVING` — filter groups, if used.
5. `SELECT` — produce the selected expressions.
6. `ORDER BY` — sort the result.
7. `LIMIT` — restrict the number of rows, where supported.

**Interview tip:** Although a query starts with `SELECT`, `FROM` and `WHERE` are logically processed earlier.

---

## 9. Common Mistakes

### Mistake 1: Using `= NULL`

Incorrect:

```sql
SELECT *
FROM employees
WHERE salary = NULL;
```

Correct:

```sql
SELECT *
FROM employees
WHERE salary IS NULL;
```

### Mistake 2: Forgetting the sorting direction

```sql
ORDER BY salary;
```

This sorts in ascending order by default. Use `DESC` when you need the highest values first.

### Mistake 3: Using LIMIT without ORDER BY

```sql
SELECT *
FROM employees
LIMIT 5;
```

This returns up to five rows, but does not guarantee which five rows you will get.

### Mistake 4: Mixing AND and OR without parentheses

Incorrect if you intend both conditions to apply to IT employees:

```sql
WHERE department = 'IT' AND salary > 50000 OR salary < 35000
```

Because `AND` is evaluated before `OR`, the last condition can also include low-salary employees from other departments.

Correct:

```sql
WHERE department = 'IT'
  AND (salary > 50000 OR salary < 35000)
```

### Mistake 5: Forgetting NULL behavior

Comparisons involving `NULL` generally produce an unknown result. `WHERE` keeps only rows for which its condition is true.

For example:

```sql
SELECT *
FROM employees
WHERE salary > 40000;
```

Employees with a `NULL` salary will not appear.

### Mistake 6: Assuming ORDER BY is optional when the order matters

Tables do not have a guaranteed display order unless you specify `ORDER BY`.

---

## 10. Interview Questions

**Q1. What is the purpose of SELECT?**

It retrieves the requested columns or expressions from a query.

**Q2. What is the purpose of WHERE?**

It filters individual rows according to a condition.

**Q3. What is the purpose of ORDER BY?**

It sorts the result in ascending or descending order.

**Q4. What is the default sorting direction?**

Ascending (`ASC`).

**Q5. What is the difference between WHERE and HAVING?**

`WHERE` filters individual rows before grouping. `HAVING` filters groups after grouping.

**Q6. Can ORDER BY use multiple columns?**

Yes. SQL sorts by the first column and uses subsequent columns to break ties.

**Q7. Can we use WHERE with aggregate functions such as COUNT()?**

Not directly in the usual way. Use `HAVING` to filter groups based on aggregate results.

**Q8. Why should we use ORDER BY with LIMIT?**

To control which rows are returned, such as the top three salaries.

**Q9. How do you find rows where a value is missing?**

Use `IS NULL`.

**Q10. What happens when a WHERE condition evaluates to UNKNOWN?**

The row does not pass the filter.

---

## 11. Practice Questions

Use the `employees` table shown above.

1. Display all columns from the employees table.
2. Display only employee names and salaries.
3. Find employees earning more than 45,000.
4. Find employees who belong to the IT department.
5. Find employees in IT or HR.
6. Find employees earning between 30,000 and 50,000, inclusive.
7. Find employees whose names start with P.
8. Find employees whose salary is `NULL`.
9. Display employees from highest to lowest salary.
10. Display employees sorted by department and then by salary in descending order.
11. Find the two highest-paid employees, excluding unknown salaries.
12. Find HR employees earning at least 35,000 and sort them by salary in descending order.

Try solving these without looking at the examples.

---

## 12. Quick Revision

- `SELECT` chooses the columns or expressions to return.
- `FROM` identifies the source table.
- `WHERE` filters individual rows.
- `AND` requires all conditions to be true.
- `OR` requires at least one condition to be true.
- `BETWEEN` checks an inclusive range.
- `IN` matches values from a list.
- `LIKE` matches a text pattern.
- Use `IS NULL` to check for missing values.
- `ORDER BY column ASC` sorts from low to high.
- `ORDER BY column DESC` sorts from high to low.
- Use multiple `ORDER BY` columns to break ties.
- Use `LIMIT` with `ORDER BY` to retrieve a predictable top-N result.
- Comparisons involving `NULL` do not behave like ordinary comparisons.

### The basic query pattern

```sql
SELECT column1, column2
FROM table_name
WHERE condition
ORDER BY column1 ASC
LIMIT 10;
```

Use only the clauses needed for the question.

## 13. Self-Check

- [ ] I can select specific columns and all columns.
- [ ] I can filter rows using comparison operators.
- [ ] I can combine conditions using `AND`, `OR`, and `NOT`.
- [ ] I understand `BETWEEN`, `IN`, and `LIKE`.
- [ ] I can filter `NULL` values correctly.
- [ ] I can sort by one or multiple columns.
- [ ] I understand why `ORDER BY` matters when using `LIMIT`.
- [ ] I can solve the practice questions without looking at the notes.

**Revision status:** Needs Revision / Practicing / Interview Ready