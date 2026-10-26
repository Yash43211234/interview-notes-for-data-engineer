# SQL DISTINCT and LIMIT

## 1. What Is DISTINCT?

`DISTINCT` removes duplicate rows from the query result.

Use it when you want to see unique values instead of repeated values.

### Example Table: employees

| emp_id | name | department | salary |
|---:|---|---|---:|
| 1 | Amit | IT | 50000 |
| 2 | Neha | HR | 40000 |
| 3 | Ravi | IT | 60000 |
| 4 | Priya | HR | 45000 |
| 5 | John | Sales | 50000 |

### Example 1: Get unique departments

```sql
SELECT DISTINCT department
FROM employees;
```

Result:

| department |
|---|
| IT |
| HR |
| Sales |

Although IT and HR appear multiple times in the original table, each appears only once in the result.

**Remember:** `DISTINCT` removes duplicates from the selected columns, not from the entire original table.

---

## 2. DISTINCT on Multiple Columns

When you select multiple columns with `DISTINCT`, SQL returns unique combinations of those columns.

```sql
SELECT DISTINCT department, salary
FROM employees;
```

Result:

| department | salary |
|---|---:|
| IT | 50000 |
| HR | 40000 |
| IT | 60000 |
| HR | 45000 |
| Sales | 50000 |

Here, SQL checks the combination of `department` and `salary`.

For example, two employees in IT with the same salary would produce only one row for that combination.

**Important:** `DISTINCT department, salary` does not mean that department and salary are each individually unique. It means their combined values are unique.

---

## 3. COUNT(DISTINCT)

Use `COUNT(DISTINCT column_name)` to count unique, non-NULL values.

```sql
SELECT COUNT(DISTINCT department) AS department_count
FROM employees;
```

Result:

| department_count |
|---:|
| 3 |

This is useful when an interviewer asks questions such as:

- How many unique departments exist?
- How many different customers placed orders?
- How many unique products were sold?

---

## 4. What Is LIMIT?

`LIMIT` restricts the number of rows returned by a query.

For example, you may have thousands of records but want to see only five.

```sql
SELECT *
FROM employees
LIMIT 3;
```

This returns at most three rows.

**Important:** Without `ORDER BY`, SQL does not guarantee which three rows will be returned.

Also, `LIMIT` is supported by databases such as PostgreSQL and MySQL, but syntax differs across database systems.

---

## 5. LIMIT with ORDER BY

Use `ORDER BY` when you want specific rows, such as the highest-paid employees.

### Example: Get the three highest-paid employees

```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 3;
```

Result:

| name | salary |
|---|---:|
| Ravi | 60000 |
| Amit | 50000 |
| John | 50000 |

`ORDER BY salary DESC` sorts salaries from highest to lowest.

`LIMIT 3` returns the first three rows after sorting.

**Remember:** If multiple rows have the same salary and you need a predictable order, add a tie-breaker such as `emp_id`:

```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC, emp_id ASC
LIMIT 3;
```

---

## 6. LIMIT with OFFSET

`OFFSET` skips a specified number of rows before returning results.

```sql
SELECT *
FROM employees
ORDER BY emp_id
LIMIT 2 OFFSET 2;
```

This skips the first two rows and returns the next two rows.

| emp_id | name |
|---:|---|
| 3 | Ravi |
| 4 | Priya |

This is commonly used for pagination, where results are shown page by page.

For example, with a page size of 10:

| Page | LIMIT | OFFSET |
|---|---:|---:|
| 1 | 10 | 0 |
| 2 | 10 | 10 |
| 3 | 10 | 20 |
| 4 | 10 | 30 |

**Important:** Use a stable `ORDER BY` for pagination. Without it, rows may appear in an unpredictable order. With frequently changing data, offset-based pagination can still skip or repeat rows between requests.

---

## 7. DISTINCT vs. GROUP BY

Both can produce unique values, but they are commonly used for different purposes.

### DISTINCT

Use it when you only need unique values.

```sql
SELECT DISTINCT department
FROM employees;
```

### GROUP BY

Use it when you want to calculate something for each group.

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

**Interview takeaway:**
- `DISTINCT` returns unique selected rows.
- `GROUP BY` forms groups, often so aggregate functions can calculate results for each group.

---

## 8. DISTINCT vs. LIMIT

These solve different problems.

| DISTINCT | LIMIT |
|---|---|
| Removes duplicate selected rows | Restricts the number of returned rows |
| Helps find unique values | Helps return a smaller result |
| Does not guarantee a particular order | Does not guarantee which rows are returned without `ORDER BY` |

Example:

```sql
SELECT DISTINCT department
FROM employees
LIMIT 2;
```

This returns at most two unique departments. Which two departments appear is not guaranteed unless you specify an order.

---

## 9. Common Mistakes

### Mistake 1: Thinking DISTINCT checks only one column

```sql
SELECT DISTINCT department, salary
FROM employees;
```

SQL checks unique combinations of both columns.

### Mistake 2: Using LIMIT without ORDER BY

```sql
SELECT *
FROM employees
LIMIT 5;
```

This does not reliably return the first five records in any meaningful business order.

### Mistake 3: Using DISTINCT to hide duplicate-data problems

If duplicates should never exist in a table, using `DISTINCT` may hide a data-quality issue instead of fixing it.

### Mistake 4: Confusing LIMIT with filtering

`LIMIT` restricts the number of rows returned. `WHERE` filters rows based on a condition.

### Mistake 5: Forgetting database differences

- PostgreSQL and MySQL support `LIMIT`.
- SQL Server commonly uses `TOP` or `OFFSET ... FETCH`.
- Standard SQL also supports `FETCH FIRST ... ROWS ONLY`.

Always check the syntax for the database you are using.

---

## 10. Interview Questions

Try answering these without looking at the notes.

1. What does `DISTINCT` do?
2. What happens when `DISTINCT` is used on multiple columns?
3. What is the difference between `DISTINCT` and `COUNT(DISTINCT column)`?
4. What does `LIMIT` do?
5. Why should we use `ORDER BY` with `LIMIT`?
6. What is the difference between `DISTINCT` and `GROUP BY`?
7. What is `OFFSET` used for?
8. How would you find the five highest-paid employees?
9. How would you find the number of unique customers?
10. Can `DISTINCT` hide a data-quality problem?

---

## 11. Practice Questions

Use the `employees` table from Section 1.

**Q1.** Find all unique departments.

**Q2.** Count the number of unique departments.

**Q3.** Find unique combinations of department and salary.

**Q4.** Return any three employee rows.

**Q5.** Find the three highest-paid employees.

**Q6.** Return the second page of employee results with two employees per page.

**Q7.** Find the two highest-paid employees, breaking salary ties by the smallest employee ID first.

**Q8.** Explain why `LIMIT` without `ORDER BY` can produce unexpected results.

---

## 12. Quick Revision — Read Before an Interview

- `DISTINCT` removes duplicate selected rows.
- With multiple columns, it checks unique combinations.
- `COUNT(DISTINCT column)` counts unique, non-NULL values.
- `LIMIT` restricts the number of returned rows.
- `ORDER BY` determines which rows come first.
- `OFFSET` skips rows before returning results.
- Use `DISTINCT` for unique results and `GROUP BY` for grouped calculations.
- Do not rely on row order without `ORDER BY`.
- SQL dialects differ in how they limit results.

### Test Yourself

- [ ] Can I use `DISTINCT` on one or more columns?
- [ ] Can I count unique values?
- [ ] Can I find the top N records using `ORDER BY` and `LIMIT`?
- [ ] Can I explain `LIMIT` versus `OFFSET`?
- [ ] Can I explain `DISTINCT` versus `GROUP BY`?

**Final rule:** Practice writing these queries without looking at the examples. Understanding when to use each clause is more important than memorizing the syntax.
