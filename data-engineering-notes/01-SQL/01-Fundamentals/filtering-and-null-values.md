# SQL Filtering and NULL Values

## 1. What Is Filtering?

Filtering means selecting only the rows that meet a condition.

In SQL, we usually use the `WHERE` clause to filter rows.

For example, if a table contains 1,000 employees and we want only employees earning more than 50,000, we can use `WHERE`.

```sql
SELECT *
FROM employees
WHERE salary > 50000;
```

**Remember:** `WHERE` filters rows before grouping and aggregation.

---

## 2. Common Comparison Operators

Assume we have an `employees` table:

| emp_id | name | department | salary |
|---:|---|---|---:|
| 1 | Amit | IT | 60000 |
| 2 | Neha | HR | 40000 |
| 3 | Ravi | IT | 30000 |
| 4 | Priya | Sales | 50000 |
| 5 | John | HR | NULL |

| Operator | Meaning | Example |
|---|---|---|
| `=` | Equal to | `salary = 50000` |
| `<>` | Not equal to | `department <> 'IT'` |
| `!=` | Not equal to; supported by many databases | `salary != 50000` |
| `>` | Greater than | `salary > 50000` |
| `<` | Less than | `salary < 50000` |
| `>=` | Greater than or equal to | `salary >= 50000` |
| `<=` | Less than or equal to | `salary <= 50000` |

Example:

```sql
SELECT name, salary
FROM employees
WHERE salary >= 50000;
```

This returns employees whose salary is at least 50,000.

**Important:** A condition comparing a value with NULL does not evaluate to true or false. It evaluates to unknown, so the row does not pass the `WHERE` filter.

---

## 3. AND, OR, and NOT

These operators help combine conditions.

### AND

Both conditions must be true.

```sql
SELECT *
FROM employees
WHERE department = 'IT'
  AND salary > 40000;
```

Returns employees who work in IT and earn more than 40,000.

### OR

At least one condition must be true.

```sql
SELECT *
FROM employees
WHERE department = 'IT'
   OR department = 'HR';
```

Returns employees from IT or HR.

### NOT

Reverses a condition.

```sql
SELECT *
FROM employees
WHERE NOT department = 'IT';
```

This returns rows where the department is known and is not IT. Rows with a NULL department are not returned because the condition is unknown for them.

**Tip:** Use parentheses when combining several `AND` and `OR` conditions to make the intended logic clear.

```sql
SELECT *
FROM employees
WHERE department = 'IT'
  AND (salary > 50000 OR salary IS NULL);
```

---

## 4. BETWEEN

`BETWEEN` checks whether a value falls within a range. Both endpoints are included.

```sql
SELECT *
FROM employees
WHERE salary BETWEEN 40000 AND 50000;
```

This includes salaries of 40,000 and 50,000, as well as values between them.

It is equivalent to:

```sql
SELECT *
FROM employees
WHERE salary >= 40000
  AND salary <= 50000;
```

**Remember:** `BETWEEN` includes both boundaries.

---

## 5. IN

`IN` checks whether a value matches any value in a list.

```sql
SELECT *
FROM employees
WHERE department IN ('IT', 'HR');
```

This is a shorter alternative to:

```sql
SELECT *
FROM employees
WHERE department = 'IT'
   OR department = 'HR';
```

### NOT IN

Returns rows whose value does not match any value in the list, subject to SQL's NULL rules.

```sql
SELECT *
FROM employees
WHERE department NOT IN ('IT', 'HR');
```

**Important:** If a `NOT IN` list contains NULL, the comparison can become unknown for otherwise nonmatching values, producing surprising results. Be careful when the list comes from a subquery that might return NULL.

---

## 6. LIKE

`LIKE` searches for text that matches a pattern.

| Symbol | Meaning |
|---|---|
| `%` | Matches zero or more characters |
| `_` | Matches exactly one character |

### Starts with A

```sql
SELECT *
FROM employees
WHERE name LIKE 'A%';
```

Matches names beginning with A, such as Amit.

### Ends with a

```sql
SELECT *
FROM employees
WHERE name LIKE '%a';
```

Matches names ending with a, such as Neha and Priya, depending on database comparison rules.

### Contains it

```sql
SELECT *
FROM employees
WHERE name LIKE '%it%';
```

Matches names containing the text `it`.

### Second character is m

```sql
SELECT *
FROM employees
WHERE name LIKE '_m%';
```

Matches names whose second character is `m`, such as Amit.

**Note:** Whether text matching is case-sensitive depends on the database and its settings.

---

## 7. What Is NULL?

NULL represents a missing, unknown, or unavailable value.

For example, John's salary is NULL. We do not know his salary from the table.

NULL does **not** mean:
- Zero.
- An empty string.
- A space.
- The number is negative.

These values have different meanings.

---

## 8. How to Check NULL Values

Use `IS NULL` to find missing values.

```sql
SELECT *
FROM employees
WHERE salary IS NULL;
```

This returns John.

Use `IS NOT NULL` to find rows with known salary values.

```sql
SELECT *
FROM employees
WHERE salary IS NOT NULL;
```

This returns the four employees with known salaries.

### Common mistake

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

**Remember:** Use `IS NULL` or `IS NOT NULL`, never `= NULL` or `<> NULL` to check for NULL.

---

## 9. Why NULL Behaves Differently

SQL uses three possible outcomes for many conditions:

- **TRUE:** The condition is satisfied.
- **FALSE:** The condition is not satisfied.
- **UNKNOWN:** The condition cannot be determined, often because NULL is involved.

Example:

```sql
SELECT *
FROM employees
WHERE salary > 40000;
```

John's salary is NULL, so SQL cannot determine whether his salary is greater than 40,000. His row is not returned.

### Simple truth rules

| Condition | Result |
|---|---|
| `NULL = 10` | UNKNOWN |
| `NULL = NULL` | UNKNOWN |
| `NULL > 10` | UNKNOWN |
| `TRUE AND UNKNOWN` | UNKNOWN |
| `FALSE AND UNKNOWN` | FALSE |
| `TRUE OR UNKNOWN` | TRUE |
| `FALSE OR UNKNOWN` | UNKNOWN |
| `NOT UNKNOWN` | UNKNOWN |

A `WHERE` clause keeps only rows for which its condition is TRUE. FALSE and UNKNOWN rows are both excluded.

---

## 10. Handling NULL with COALESCE

`COALESCE()` returns the first non-NULL value from its arguments.

Example:

```sql
SELECT
    name,
    COALESCE(salary, 0) AS salary
FROM employees;
```

For John, this displays `0` instead of NULL.

**Important:** This changes the displayed result, not the original table data.

Only replace NULL with zero when zero makes sense for the business requirement. A missing salary is not necessarily the same as a salary of zero.

You can also use text:

```sql
SELECT
    name,
    COALESCE(department, 'Unknown') AS department
FROM employees;
```

---

## 11. Filtering with Aggregate Functions

Use `WHERE` to filter individual rows before aggregation.

```sql
SELECT
    department,
    AVG(salary) AS average_salary
FROM employees
WHERE salary >= 40000
GROUP BY department;
```

This removes rows with salaries below 40,000 and NULL salaries before calculating each department's average.

Use `HAVING` to filter groups based on an aggregate result.

```sql
SELECT
    department,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 45000;
```

This keeps departments whose average known salary is greater than 45,000.

**Remember:**
- `WHERE` filters rows.
- `HAVING` filters groups.

---

## 12. Common Mistakes

1. **Checking NULL with `=`.** Use `IS NULL`.
2. **Assuming NULL equals zero.** NULL means the value is unknown or missing.
3. **Forgetting that BETWEEN includes both endpoints.**
4. **Using `NOT IN` with a list or subquery containing NULL.** This can exclude more rows than expected.
5. **Mixing AND and OR without parentheses.** Use parentheses to make the intended logic clear.
6. **Assuming `WHERE` keeps unknown conditions.** It keeps only TRUE conditions.
7. **Replacing every NULL with zero.** Only do this when zero is meaningful.
8. **Using WHERE for aggregate results.** Use `HAVING` when filtering based on an aggregate calculation.

---

## 13. Interview Questions

Try answering without looking at the notes.

1. What does the `WHERE` clause do?
2. What is the difference between AND and OR?
3. What does BETWEEN do? Are its boundaries included?
4. What is the difference between IN and NOT IN?
5. What do `%` and `_` mean in LIKE?
6. What is NULL in SQL?
7. Why does `salary = NULL` not work?
8. What is the difference between NULL, zero, and an empty string?
9. What is the difference between WHERE and HAVING?
10. How does COALESCE handle NULL?
11. What happens when a WHERE condition evaluates to UNKNOWN?
12. Why can NOT IN behave unexpectedly if its list contains NULL?

---

## 14. Practice Questions

Use the `employees` table from Section 2.

**Q1.** Find employees earning more than 45,000.

**Q2.** Find employees from IT and HR.

**Q3.** Find employees whose salary is between 30,000 and 50,000, including both endpoints.

**Q4.** Find employees whose names start with P.

**Q5.** Find employees whose salary is NULL.

**Q6.** Find employees whose salary is known.

**Q7.** Display NULL salaries as zero without updating the table.

**Q8.** Find employees who are not in IT. Consider how NULL departments should be handled.

**Q9.** Find the average salary for each department using only salaries of at least 40,000.

**Q10.** Explain why a comparison between NULL and a number does not return TRUE or FALSE.

---

## 15. Quick Revision — Before an Interview

- `WHERE` filters individual rows.
- `AND` requires both conditions to be true.
- `OR` requires at least one condition to be true.
- `NOT` reverses a condition.
- `BETWEEN` includes both endpoints.
- `IN` checks for a match in a list.
- `LIKE` matches text patterns: `%` means any number of characters; `_` means one character.
- NULL represents a missing or unknown value.
- Use `IS NULL` and `IS NOT NULL`.
- SQL conditions can evaluate to TRUE, FALSE, or UNKNOWN.
- `WHERE` keeps only rows for which the condition is TRUE.
- `COALESCE()` returns the first non-NULL value.
- `WHERE` filters rows; `HAVING` filters groups.
- Be careful with `NOT IN` when the list may contain NULL.

### Final Self-Test

- [ ] Can I combine conditions using AND, OR, and NOT?
- [ ] Can I use BETWEEN, IN, and LIKE?
- [ ] Can I explain why NULL behaves differently?
- [ ] Can I handle NULL using IS NULL and COALESCE?
- [ ] Can I explain WHERE versus HAVING?

**Final rule:** Practice these conditions on real tables. The most important skill is knowing which rows SQL will return, especially when NULL values are involved.
