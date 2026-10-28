# Self Joins in SQL

## 1. What Is a Self Join?

A **self join** is a join in which a table is joined with itself.

It is useful when you need to compare rows within the same table or connect related records stored in one table.

**Simple definition:** A self join lets you use the same table twice in one query, treating each use as a separate table with its own alias.

Common use cases:
- Find employees and their managers.
- Compare employee salaries.
- Find customers from the same city.
- Identify duplicate records.
- Compare records within the same dataset.

## 2. Sample Table

Consider the following `employees` table:

| employee_id | employee_name | manager_id | department | salary |
|---:|---|---:|---|---:|
| 1 | Amit | NULL | IT | 80000 |
| 2 | Neha | 1 | IT | 60000 |
| 3 | Ravi | 1 | HR | 50000 |
| 4 | Priya | 2 | IT | 45000 |
| 5 | John | 2 | Sales | 40000 |

Here:
- Amit has no manager.
- Neha and Ravi report to Amit.
- Priya and John report to Neha.

The `manager_id` refers to another employee's `employee_id` in the same table.

## 3. Basic Self Join Example

**Question:** Display each employee with their manager's name.

```sql
SELECT
    e.employee_name AS employee,
    m.employee_name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.employee_id;
```

Result:

| employee | manager |
|---|---|
| Amit | NULL |
| Neha | Amit |
| Ravi | Amit |
| Priya | Neha |
| John | Neha |

### How it works

1. `employees e` represents the employee whose information we want.
2. `employees m` represents the manager.
3. `e.manager_id = m.employee_id` connects an employee to their manager.
4. `LEFT JOIN` keeps employees who do not have a manager.

**Important:** Both aliases refer to the same physical table, but they represent different roles in the query.

## 4. Why Do We Need Table Aliases?

Consider:

```sql
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.employee_id
```

- `e` refers to the employee.
- `m` refers to the manager.

Both aliases have the same columns because they refer to the same table.

Without aliases, it would be difficult to distinguish between the employee's name and the manager's name.

**Interview tip:** When explaining a self join, clearly state what each alias represents.

## 5. Find Employees Who Have a Manager

**Question:** Find employees who report to someone.

```sql
SELECT
    e.employee_name,
    m.employee_name AS manager
FROM employees e
INNER JOIN employees m
    ON e.manager_id = m.employee_id;
```

Result:

| employee_name | manager |
|---|---|
| Neha | Amit |
| Ravi | Amit |
| Priya | Neha |
| John | Neha |

The INNER JOIN excludes Amit because his `manager_id` is `NULL` and no manager row matches it.

Use `INNER JOIN` when employees without a matching manager should be excluded. Use `LEFT JOIN` when all employees should remain in the result.

## 6. Find Employees Earning More Than Their Managers

**Question:** Find employees whose salary is higher than their manager's salary.

```sql
SELECT
    e.employee_name AS employee,
    e.salary AS employee_salary,
    m.employee_name AS manager,
    m.salary AS manager_salary
FROM employees e
JOIN employees m
    ON e.manager_id = m.employee_id
WHERE e.salary > m.salary;
```

With the sample data, this returns no rows because every employee with a manager earns less than that manager.

To understand the condition, suppose Priya earned 90,000 instead of 45,000. She would appear in the result because 90,000 is greater than Neha's 60,000.

**Key point:** A self join allows you to compare values from two different rows of the same table.

## 7. Find Employees in the Same Department

**Question:** Find pairs of employees who work in the same department.

```sql
SELECT
    e.employee_name AS employee_1,
    e2.employee_name AS employee_2,
    e.department
FROM employees e
JOIN employees e2
    ON e.department = e2.department
   AND e.employee_id < e2.employee_id;
```

The condition `e.employee_id < e2.employee_id` ensures that:
- An employee is not paired with themselves.
- The same pair is not returned twice in reverse order.

For example, Amit and Neha belong to the same department, so they form a pair.

**Remember:** If you remove the ID comparison, the result may include self-pairs and reversed duplicate pairs.

## 8. Find Possible Duplicate Records

Self joins can help identify records that share the same business values.

**Question:** Find employees with the same name.

```sql
SELECT
    e.employee_id AS id_1,
    e.employee_name AS name_1,
    e2.employee_id AS id_2,
    e2.employee_name AS name_2
FROM employees e
JOIN employees e2
    ON e.employee_name = e2.employee_name
   AND e.employee_id < e2.employee_id;
```

This query returns pairs of records with the same name but different employee IDs.

With the current sample data, it returns no rows because all employee names are unique.

**Important:** Matching names alone does not prove that two records are duplicates. Use the correct business keys and validation rules for your dataset.

## 9. Self Join vs. Regular Join

| Self join | Regular join |
|---|---|
| Joins a table with itself | Usually joins different tables |
| Uses aliases to distinguish table roles | May also use aliases |
| Useful for relationships between rows in one table | Useful for relationships between tables |
| Example: employee and manager | Example: employee and department |

A self join is not a separate SQL join type. It uses ordinary join operations, such as INNER JOIN or LEFT JOIN, on the same table.

## 10. Self Join vs. Correlated Subquery

Both can compare data across related rows, but they use different query structures.

### Self join

```sql
SELECT e.employee_name, m.employee_name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.employee_id;
```

### Correlated subquery

```sql
SELECT
    e.employee_name,
    (
        SELECT m.employee_name
        FROM employees m
        WHERE m.employee_id = e.manager_id
    ) AS manager
FROM employees e;
```

Both queries can return employee-manager information when each employee has at most one matching manager row.

The self join makes the relationship explicit in the join condition. The correlated subquery calculates the manager value using the current employee's `manager_id`.

Performance depends on the database optimizer and query structure.

## 11. Common Mistakes

### Mistake 1: Forgetting aliases

Incorrect:

```sql
SELECT employee_name, employee_name
FROM employees
JOIN employees
    ON manager_id = employee_id;
```

The column references are ambiguous, and the table references are not clearly distinguished.

Correct:

```sql
SELECT
    e.employee_name AS employee,
    m.employee_name AS manager
FROM employees e
JOIN employees m
    ON e.manager_id = m.employee_id;
```

### Mistake 2: Using the wrong matching columns

For an employee-manager relationship, use:

```sql
ON e.manager_id = m.employee_id
```

Do not compare `e.employee_id` with `m.employee_id` unless you intentionally want to match each employee with the same employee record.

### Mistake 3: Accidentally matching each row with itself

If the requirement is to compare different employees, add a condition such as:

```sql
AND e.employee_id < e2.employee_id
```

This prevents self-pairs and reversed duplicate pairs when IDs are unique.

### Mistake 4: Using INNER JOIN when unmatched rows must remain

An INNER JOIN excludes employees who have no matching manager. Use a LEFT JOIN if those employees must remain in the result.

### Mistake 5: Assuming a self join always returns one row per employee

If the join condition matches multiple rows, an employee may appear multiple times. The number of results depends on the matching relationship.

## 12. Interview Questions

**Q1. What is a self join?**

A self join joins a table with itself to compare or connect rows from that table.

**Q2. Why are aliases required in a self join?**

Aliases distinguish the different roles played by the same table in the query.

**Q3. How do you find an employee's manager?**

Join the employee's `manager_id` to another row's `employee_id`.

**Q4. What is the difference between a self join and an INNER JOIN?**

A self join describes which tables are joined: the same table twice. INNER JOIN describes how matching rows are returned. A self join can use INNER JOIN or another join type.

**Q5. Can a self join use LEFT JOIN?**

Yes. A LEFT self join can preserve employees who do not have a manager.

**Q6. How do you find pairs of employees in the same department without duplicate pairs?**

Join the table to itself on the department and add a condition such as `e.employee_id < e2.employee_id`.

**Q7. Can self joins help identify duplicates?**

Yes. They can compare records based on selected fields, although matching values do not always prove that records are duplicates.

**Q8. Can a self join be replaced with a window function?**

Sometimes. For example, window functions can compare a row with another row in an ordered dataset. The best approach depends on the relationship and the required output.

## 13. Practice Questions

Use the `employees` table shown earlier.

1. Display every employee with their manager's name.
2. Find employees who do not have a manager.
3. Find employees who report to Amit.
4. Find employees earning more than their managers.
5. Find pairs of employees who work in the same department.
6. Find pairs of employees who earn the same salary.
7. Find records with the same employee name but different IDs.
8. Explain why `e.employee_id < e2.employee_id` is useful when finding pairs.
9. Rewrite the employee-manager query using a correlated subquery.
10. Explain when you would choose LEFT JOIN instead of INNER JOIN for a self join.

## 14. Quick Revision

- A self join joins a table with itself.
- It is useful for comparing rows or representing relationships within one table.
- Aliases distinguish the different roles of the table.
- Employee-manager relationships are a common example.
- Use LEFT JOIN to preserve employees without a manager.
- Use INNER JOIN when unmatched rows should be excluded.
- Use `e.employee_id < e2.employee_id` to avoid self-pairs and reversed duplicate pairs when finding unique pairs.
- A self join can use INNER, LEFT, RIGHT, or FULL OUTER JOIN where supported.

### Remember this pattern

```sql
SELECT
    a.column_name,
    b.column_name
FROM table_name a
JOIN table_name b
    ON a.related_id = b.id;
```

## 15. Self-Check

- [ ] I can explain a self join in simple language.
- [ ] I understand why aliases are necessary.
- [ ] I can write an employee-manager query.
- [ ] I can compare values from two rows of the same table.
- [ ] I can find pairs without returning reversed duplicates.
- [ ] I understand the difference between INNER and LEFT self joins.
- [ ] I can solve the practice questions without looking at the examples.

**Revision status:** Needs Revision / Practicing / Interview Ready