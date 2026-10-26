# SQL CASE Expressions

## 1. What Is a CASE Expression?

A `CASE` expression lets us return different values depending on whether conditions are true or false.

Think of it as **IF-ELSE logic in SQL**.

For example:
- If salary is greater than or equal to 50,000, return `High`.
- Otherwise, return `Low`.

**Remember:** `CASE` checks conditions and returns a result based on the first matching condition.

---

## 2. Basic Syntax

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE default_result
END
```

How it works:

1. SQL checks the first condition.
2. If it is true, SQL returns its result.
3. Otherwise, SQL checks the next condition.
4. If no condition matches, SQL returns the `ELSE` result.
5. If there is no `ELSE` and no condition matches, SQL returns `NULL`.

The `ELSE` part is optional, but it is often useful.

---

## 3. Example Table

Assume we have an `employees` table:

| emp_id | name | department | salary |
|---:|---|---|---:|
| 1 | Amit | IT | 60000 |
| 2 | Neha | HR | 40000 |
| 3 | Ravi | IT | 30000 |
| 4 | Priya | Sales | 50000 |
| 5 | John | HR | NULL |

---

## 4. Example 1: Categorize Values

Give employees a salary category.

```sql
SELECT
    name,
    salary,
    CASE
        WHEN salary >= 50000 THEN 'High'
        ELSE 'Low'
    END AS salary_category
FROM employees;
```

Result:

| name | salary | salary_category |
|---|---:|---|
| Amit | 60000 | High |
| Neha | 40000 | Low |
| Ravi | 30000 | Low |
| Priya | 50000 | High |
| John | NULL | Low |

**Important:** In this example, John's salary is NULL. The condition `salary >= 50000` is not true for him, so the `ELSE` result is returned. If missing salaries should have a separate label, handle NULL explicitly.

---

## 5. Example 2: Multiple Conditions

Classify salaries into three categories.

```sql
SELECT
    name,
    salary,
    CASE
        WHEN salary >= 50000 THEN 'High'
        WHEN salary >= 35000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_category
FROM employees;
```

The conditions are checked from top to bottom.

- Salary 60,000 → High
- Salary 40,000 → Medium
- Salary 30,000 → Low

**Important:** Order matters. SQL returns the result for the first matching condition.

For example, if `salary >= 35000` came first, an employee earning 60,000 would match that condition and be labeled `Medium`.

---

## 6. Example 3: Handle NULL Values

Use `IS NULL` to check for missing values.

```sql
SELECT
    name,
    salary,
    CASE
        WHEN salary IS NULL THEN 'Unknown'
        WHEN salary >= 50000 THEN 'High'
        ELSE 'Low'
    END AS salary_category
FROM employees;
```

Now John is labeled `Unknown`.

**Remember:** Use `IS NULL`, not `= NULL`.

---

## 7. Example 4: CASE with GROUP BY

Count employees in different salary categories.

```sql
SELECT
    CASE
        WHEN salary IS NULL THEN 'Unknown'
        WHEN salary >= 50000 THEN 'High'
        WHEN salary >= 35000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_category,
    COUNT(*) AS employee_count
FROM employees
GROUP BY
    CASE
        WHEN salary IS NULL THEN 'Unknown'
        WHEN salary >= 50000 THEN 'High'
        WHEN salary >= 35000 THEN 'Medium'
        ELSE 'Low'
    END;
```

This query first assigns a category to each employee, then counts employees in each category.

**Use case:** Grouping sales into ranges, customers into age groups, or employees into salary categories.

Some databases let you group by a select-list alias, but support varies. Repeating the `CASE` expression is broadly portable SQL.

---

## 8. Example 5: Conditional Aggregation

`CASE` is commonly used with aggregate functions to calculate counts or totals based on conditions.

### Count IT employees

```sql
SELECT
    COUNT(*) AS total_employees,
    SUM(
        CASE
            WHEN department = 'IT' THEN 1
            ELSE 0
        END
    ) AS it_employees
FROM employees;
```

Result:

| total_employees | it_employees |
|---:|---:|
| 5 | 2 |

How it works:

- Each IT employee contributes `1`.
- Each non-IT employee contributes `0`.
- `SUM()` adds those values.

### Calculate total salary for IT employees

```sql
SELECT
    SUM(
        CASE
            WHEN department = 'IT' THEN salary
            ELSE 0
        END
    ) AS it_total_salary
FROM employees;
```

Result: `90000`

**Interview tip:** Conditional aggregation is useful when several condition-based counts or totals are needed in a single query.

---

## 9. Example 6: CASE with UPDATE

`CASE` can also be used to update values.

Suppose we want to assign a bonus based on salary.

```sql
UPDATE employees
SET salary = CASE
    WHEN salary >= 50000 THEN salary + 5000
    WHEN salary IS NOT NULL THEN salary + 2000
    ELSE NULL
END;
```

This increases the salary by 5,000 for employees earning at least 50,000 and by 2,000 for other employees with a known salary. Missing salaries remain NULL.

**Important:** `UPDATE` changes table data. Run it only when you intend to modify the records, and follow your database's transaction and backup practices.

---

## 10. Simple CASE vs. Searched CASE

There are two common forms.

### Searched CASE

Checks conditions such as comparisons.

```sql
CASE
    WHEN salary >= 50000 THEN 'High'
    WHEN salary >= 35000 THEN 'Medium'
    ELSE 'Low'
END
```

Use this when conditions involve comparisons, ranges, or multiple rules.

### Simple CASE

Compares one expression with specific values.

```sql
CASE department
    WHEN 'IT' THEN 'Technology'
    WHEN 'HR' THEN 'Human Resources'
    ELSE 'Other'
END
```

Use this when matching one expression against particular values.

**Difference to remember:**

- Searched CASE: each `WHEN` contains a condition.
- Simple CASE: each `WHEN` contains a value to match.

---

## 11. Common Mistakes

### Mistake 1: Wrong condition order

Incorrect:

```sql
CASE
    WHEN salary >= 35000 THEN 'Medium'
    WHEN salary >= 50000 THEN 'High'
    ELSE 'Low'
END
```

A salary of 60,000 matches the first condition and returns `Medium`.

Correct: Put the more specific or higher-threshold condition first.

### Mistake 2: Forgetting NULL

Comparisons with NULL do not evaluate to true.

If missing values need their own category, use:

```sql
WHEN salary IS NULL THEN 'Unknown'
```

### Mistake 3: Forgetting `END`

Every `CASE` expression must end with `END`.

### Mistake 4: Returning incompatible data types

Try to keep all `THEN` and `ELSE` results compatible with one another.

For example, avoid mixing text and numeric values unless your database supports the intended conversion.

### Mistake 5: Confusing CASE with control flow

`CASE` is an expression that returns a value. It is not the same as a procedural `IF` statement in a stored procedure.

### Mistake 6: Using `ELSE 0` without thinking

In conditional aggregation, `ELSE 0` can be useful for counting or totaling. But for other calculations, returning zero may change the meaning of the result.

Choose the result that matches the business requirement.

---

## 12. Common Interview Questions

Try answering these without reading the answers.

1. What is a CASE expression in SQL?
2. What happens when no CASE condition matches?
3. Why does the order of WHEN conditions matter?
4. What is the difference between simple CASE and searched CASE?
5. How do you handle NULL values in CASE?
6. How can CASE be used with SUM() and COUNT()?
7. Can CASE be used in SELECT, ORDER BY, GROUP BY, and UPDATE?
8. What is conditional aggregation?
9. What is the difference between CASE and COALESCE?
10. What problems can occur if CASE returns incompatible data types?

---

## 13. Practice Questions

Use the `employees` table from Section 3.

**Q1.** Label salaries as `High` if they are at least 50,000 and `Low` otherwise.

**Q2.** Create three salary categories: `High` (50,000 or more), `Medium` (35,000 to less than 50,000), and `Low` (less than 35,000).

**Q3.** Label employees with missing salaries as `Unknown`.

**Q4.** Count IT employees and non-IT employees in one query.

**Q5.** Calculate the total salary for IT and HR employees in separate columns.

**Q6.** Sort employees so that IT employees appear first, followed by HR and then Sales.

**Q7.** Use a simple CASE expression to convert department names into department descriptions.

**Q8.** Explain why the order of conditions matters in a salary classification query.

---

## 14. Quick Revision — Read Before an Interview

- `CASE` is SQL's way to return different values based on conditions.
- It returns the result of the **first matching** `WHEN`.
- `ELSE` handles cases where no condition matches.
- Without a matching condition or `ELSE`, the result is NULL.
- Use `IS NULL` to check for NULL values.
- Searched CASE checks conditions; simple CASE matches specific values.
- `CASE` can be used in queries, sorting, grouping, and data updates.
- `CASE` with `SUM()` or `COUNT()` is useful for conditional aggregation.
- Keep result data types compatible.
- Check condition order carefully.

### Test Yourself

- [ ] Can I write a CASE expression with multiple conditions?
- [ ] Can I handle NULL values correctly?
- [ ] Can I use CASE with GROUP BY?
- [ ] Can I write conditional aggregation?
- [ ] Can I explain simple CASE versus searched CASE?
- [ ] Can I solve the practice questions without looking at the notes?

**Final rule:** Close the notes and write three queries using `CASE` from memory. If you can explain why each condition is ordered that way, you understand the topic much better than if you only memorize the syntax.
