# INNER, LEFT, RIGHT, and FULL OUTER JOINs in SQL

## 1. What Is a JOIN?

A `JOIN` combines rows from two tables based on a related column or a specified matching condition.

In data engineering, joins are commonly used to combine customer data with orders, products with sales, and source records with lookup tables.

The four main joins covered here are:

- `INNER JOIN`
- `LEFT JOIN`
- `RIGHT JOIN`
- `FULL OUTER JOIN`

## 2. Sample Tables

We will use these two tables throughout the notes.

### Employees

| employee_id | employee_name | department_id |
|---:|---|---:|
| 1 | Amit | 10 |
| 2 | Neha | 20 |
| 3 | Ravi | 30 |
| 4 | Priya | NULL |

### Departments

| department_id | department_name |
|---:|---|
| 10 | IT |
| 20 | HR |
| 40 | Finance |

Notice that:

- Amit and Neha have matching departments.
- Ravi belongs to department 30, which is missing from the Departments table.
- Priya has no department ID.
- Finance has no employee in the Employees table.

These differences help us understand how each join behaves.

## 3. INNER JOIN

`INNER JOIN` returns only rows that have a match in both tables according to the join condition.

### Example

```sql
SELECT
    e.employee_name,
    d.department_name
FROM employees e
INNER JOIN departments d
    ON e.department_id = d.department_id;
```

Result:

| employee_name | department_name |
|---|---|
| Amit | IT |
| Neha | HR |

### How it works

1. SQL compares the department IDs in both tables.
2. It finds matching IDs: 10 and 20.
3. It returns the rows that match.

Ravi is excluded because department 30 does not exist in the Departments table. Priya is excluded because `NULL` does not equal 10, 20, or 40.

Finance is also excluded because no employee matches department 40.

**Remember:** `INNER JOIN` returns matching rows only.

## 4. LEFT JOIN

`LEFT JOIN`, also called `LEFT OUTER JOIN`, returns:

- Every row from the left table.
- Matching rows from the right table.
- `NULL` values for right-table columns when no match exists.

### Example

```sql
SELECT
    e.employee_name,
    e.department_id,
    d.department_name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id;
```

Result:

| employee_name | department_id | department_name |
|---|---:|---|
| Amit | 10 | IT |
| Neha | 20 | HR |
| Ravi | 30 | NULL |
| Priya | NULL | NULL |

### How it works

- Amit matches IT.
- Neha matches HR.
- Ravi remains in the result even though department 30 has no match.
- Priya remains in the result even though her department ID is `NULL`.

**Remember:** A LEFT JOIN preserves every row from the left table.

### Common use case

Find employees who do not have a matching department.

```sql
SELECT
    e.employee_id,
    e.employee_name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
WHERE d.department_id IS NULL;
```

Result:

| employee_id | employee_name |
|---:|---|
| 3 | Ravi |
| 4 | Priya |

This works because `d.department_id` is `NULL` when no matching department is found. In this example, the department IDs in the Departments table are non-NULL.

## 5. RIGHT JOIN

`RIGHT JOIN`, also called `RIGHT OUTER JOIN`, returns:

- Every row from the right table.
- Matching rows from the left table.
- `NULL` values for left-table columns when no match exists.

### Example

```sql
SELECT
    e.employee_name,
    d.department_id,
    d.department_name
FROM employees e
RIGHT JOIN departments d
    ON e.department_id = d.department_id;
```

Result:

| employee_name | department_id | department_name |
|---|---:|---|
| Amit | 10 | IT |
| Neha | 20 | HR |
| NULL | 40 | Finance |

Finance appears even though no employee belongs to it.

Ravi and Priya do not appear because a RIGHT JOIN preserves all rows from the Departments table, not all rows from the Employees table.

**Remember:** A RIGHT JOIN preserves every row from the right table.

### Common use case

Find departments that have no employees.

```sql
SELECT
    d.department_id,
    d.department_name
FROM employees e
RIGHT JOIN departments d
    ON e.department_id = d.department_id
WHERE e.employee_id IS NULL;
```

Result:

| department_id | department_name |
|---:|---|
| 40 | Finance |

This assumes `employee_id` is a non-NULL identifier for every real employee.

## 6. FULL OUTER JOIN

`FULL OUTER JOIN` returns:

- All matching rows from both tables.
- Unmatched rows from the left table.
- Unmatched rows from the right table.
- `NULL` values for the missing side of an unmatched row.

### Example

```sql
SELECT
    e.employee_name,
    e.department_id AS employee_department_id,
    d.department_id AS matched_department_id,
    d.department_name
FROM employees e
FULL OUTER JOIN departments d
    ON e.department_id = d.department_id;
```

Result:

| employee_name | employee_department_id | matched_department_id | department_name |
|---|---:|---:|---|
| Amit | 10 | 10 | IT |
| Neha | 20 | 20 | HR |
| Ravi | 30 | NULL | NULL |
| Priya | NULL | NULL | NULL |
| NULL | NULL | 40 | Finance |

### How it works

- Amit and Neha match their departments.
- Ravi appears despite having no matching department.
- Priya appears despite having no matching department.
- Finance appears despite having no matching employee.

**Remember:** A FULL OUTER JOIN preserves all rows from both tables, whether or not they match.

**Database note:** PostgreSQL and SQL Server support `FULL OUTER JOIN`. Some databases, including MySQL, do not support it directly, so you may need to combine a LEFT JOIN and a RIGHT JOIN using `UNION` or another equivalent approach.

## 7. Compare All Four Joins

<box gap={3}>
  <row align="start" gap={3}>
    <AsyncImage query="Venn diagram inner join SQL two circles overlapping center highlighted" aspectRatio="1:1" width="104px" />
    <box flex="1" gap={1}>
      **INNER JOIN**

      Only matching rows from both tables.

      <text color="secondary" size="sm">Unmatched rows from both sides are excluded.</text>
    </box>
  </row>
  <divider color="subtle" />
  <row align="start" gap={3}>
    <AsyncImage query="Venn diagram left outer join SQL left circle highlighted including intersection" aspectRatio="1:1" width="104px" />
    <box flex="1" gap={1}>
      **LEFT JOIN**

      All rows from the left table, plus matching right-table rows.

      <text color="secondary" size="sm">Missing right-side values appear as NULL.</text>
    </box>
  </row>
  <divider color="subtle" />
  <row align="start" gap={3}>
    <AsyncImage query="Venn diagram right outer join SQL right circle highlighted including intersection" aspectRatio="1:1" width="104px" />
    <box flex="1" gap={1}>
      **RIGHT JOIN**

      All rows from the right table, plus matching left-table rows.

      <text color="secondary" size="sm">Missing left-side values appear as NULL.</text>
    </box>
  </row>
  <divider color="subtle" />
  <row align="start" gap={3}>
    <AsyncImage query="Venn diagram full outer join SQL both circles highlighted" aspectRatio="1:1" width="104px" />
    <box flex="1" gap={1}>
      **FULL OUTER JOIN**

      All rows from both tables, including unmatched rows.

      <text color="secondary" size="sm">Unmatched sides contain NULL values.</text>
    </box>
  </row>
</box>

| Join type | Preserves all rows from | Unmatched rows included? |
|---|---|---|
| INNER JOIN | Neither side | No |
| LEFT JOIN | Left table | Left side only |
| RIGHT JOIN | Right table | Right side only |
| FULL OUTER JOIN | Both tables | Both sides |

## 8. LEFT JOIN vs. INNER JOIN

Suppose you want a list of employees and their department names.

With an INNER JOIN, employees without a matching department are excluded.

With a LEFT JOIN, those employees remain, and their department name is `NULL`.

```sql
-- Only employees with a matching department
SELECT e.employee_name, d.department_name
FROM employees e
INNER JOIN departments d
    ON e.department_id = d.department_id;
```

```sql
-- All employees, including those without a matching department
SELECT e.employee_name, d.department_name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id;
```

**Interview tip:** When the requirement says “include all records from the first table, even if there is no match,” use a LEFT JOIN.

## 9. Find Unmatched Records

This is a common data engineering and SQL interview pattern.

### Find employees without matching departments

```sql
SELECT e.employee_id, e.employee_name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
WHERE d.department_id IS NULL;
```

### Find departments without employees

```sql
SELECT d.department_id, d.department_name
FROM departments d
LEFT JOIN employees e
    ON d.department_id = e.department_id
WHERE e.employee_id IS NULL;
```

### Find unmatched records from either table

Using FULL OUTER JOIN:

```sql
SELECT
    e.employee_name,
    d.department_name
FROM employees e
FULL OUTER JOIN departments d
    ON e.department_id = d.department_id
WHERE e.employee_id IS NULL
   OR d.department_id IS NULL;
```

This returns employees without a matching department and departments without a matching employee.

These patterns are useful for data validation, reconciliation, and finding missing relationships between datasets.

## 10. Important Difference: ON vs. WHERE

Conditions in `ON` and `WHERE` can produce different results with outer joins.

### Example A: Condition inside ON

```sql
SELECT e.employee_name, d.department_name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
   AND d.department_name = 'IT';
```

This preserves all employees but matches department details only for IT. Employees without an IT match have `NULL` for the department name.

### Example B: Condition inside WHERE

```sql
SELECT e.employee_name, d.department_name
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
WHERE d.department_name = 'IT';
```

This returns only employees whose matched department is IT. Unmatched employees are removed because their department name is `NULL`.

**Remember:** A condition on the right table in `WHERE` can remove unmatched rows from a LEFT JOIN, making the result behave like an INNER JOIN for that condition.

## 11. Common Mistakes

### Mistake 1: Confusing the left and right tables

In:

```sql
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.department_id
```

`employees` is the left table. `departments` is the right table.

### Mistake 2: Using `=` to match NULL values

```sql
ON e.department_id = d.department_id
```

A `NULL` department ID does not match another `NULL` using ordinary equality. NULL-safe matching is database-specific and should only be used when it matches the business requirement.

### Mistake 3: Filtering the right table in WHERE unintentionally

A condition such as `WHERE d.department_name = 'IT'` removes unmatched rows from a LEFT JOIN. Decide whether the filter belongs in `ON` or `WHERE` based on the intended result.

### Mistake 4: Using the wrong column to detect unmatched rows

Use a column that cannot be `NULL` in a genuine matched row, typically a primary key. Otherwise, real matched records with a nullable column might be mistaken for unmatched records.

### Mistake 5: Assuming a join always returns one row per employee

If multiple department rows match the same join key, an employee can appear multiple times. A join returns matching row combinations, not necessarily one row per entity.

### Mistake 6: Forgetting that duplicate keys can multiply rows

If a join key occurs twice in the left table and three times in the right table, that key can produce six matching combinations.

## 12. Interview Questions

**Q1. What is an INNER JOIN?**

It returns only rows that match the join condition in both tables.

**Q2. What is a LEFT JOIN?**

It preserves every row from the left table and includes matching right-table data when available.

**Q3. What is a RIGHT JOIN?**

It preserves every row from the right table and includes matching left-table data when available.

**Q4. What is a FULL OUTER JOIN?**

It returns all matching and unmatched rows from both tables.

**Q5. How do you find employees without a department?**

Use a LEFT JOIN from employees to departments, then filter for a missing department-side key.

**Q6. How do you find departments without employees?**

Use a LEFT JOIN from departments to employees, then filter for a missing employee-side key.

**Q7. What is the difference between ON and WHERE in an outer join?**

`ON` determines which rows match. `WHERE` filters the resulting rows, potentially removing unmatched rows.

**Q8. Can joins return duplicate rows?**

Yes. Multiple matching records on either side can produce multiple row combinations.

**Q9. Is LEFT JOIN the same as LEFT OUTER JOIN?**

Yes. `LEFT JOIN` is shorthand for `LEFT OUTER JOIN`.

**Q10. What if the database does not support FULL OUTER JOIN?**

Use an equivalent query, commonly a LEFT JOIN combined with a RIGHT JOIN using `UNION`, while taking care to avoid duplicate matching rows.

## 13. Practice Questions

Use the Employees and Departments tables from this file.

1. Display employees with their department names using INNER JOIN.
2. Display all employees and their department names using LEFT JOIN.
3. Display all departments and any matching employees using RIGHT JOIN.
4. Display all employees and all departments using FULL OUTER JOIN.
5. Find employees whose department does not exist in the Departments table.
6. Find departments that do not have employees.
7. Return only IT employees while using a LEFT JOIN.
8. Explain the difference between placing a department filter in `ON` and in `WHERE`.
9. Predict the number of rows returned by each join using the sample tables.
10. Explain how duplicate keys can multiply the number of rows in a join.

## 14. Quick Revision

- **INNER JOIN:** Matching rows only.
- **LEFT JOIN:** All left-table rows, with matching right-table data.
- **RIGHT JOIN:** All right-table rows, with matching left-table data.
- **FULL OUTER JOIN:** All rows from both tables.
- Unmatched columns from the missing side are filled with `NULL`.
- Use `LEFT JOIN ... WHERE right_key IS NULL` to find left-side records without matches.
- Use `ON` to define matching conditions and `WHERE` to filter the resulting rows.
- Duplicate join keys can multiply result rows.
- SQL does not guarantee row order without `ORDER BY`.

### Remember these patterns

```sql
-- Matching rows
SELECT *
FROM A
INNER JOIN B ON A.id = B.id;

-- All rows from A
SELECT *
FROM A
LEFT JOIN B ON A.id = B.id;

-- All rows from B
SELECT *
FROM A
RIGHT JOIN B ON A.id = B.id;

-- All rows from both tables
SELECT *
FROM A
FULL OUTER JOIN B ON A.id = B.id;
```

## 15. Self-Check

- [ ] I can explain all four join types.
- [ ] I know which table is preserved by each join.
- [ ] I can find unmatched records.
- [ ] I understand how `NULL` values affect matching.
- [ ] I can explain the difference between `ON` and `WHERE`.
- [ ] I understand why duplicate keys can increase row counts.
- [ ] I can solve the practice questions without looking at the examples.

**Revision status:** Needs Revision / Practicing / Interview Ready