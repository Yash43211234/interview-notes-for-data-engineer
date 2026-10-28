# Recursive CTEs in SQL

## 1. What Is a Recursive CTE?

A **recursive CTE (Common Table Expression)** is a CTE that refers to itself to repeatedly process data until a stopping condition is reached.

It is useful when data has a hierarchical or repetitive structure, such as:
- Employee-manager hierarchies.
- Organizational charts.
- Parent-child relationships.
- Category trees.
- Generating sequences of numbers or dates.
- Exploring connected levels in hierarchical data.

**Simple idea:** Start with an initial result, use that result to find the next level, and repeat until no more rows are produced or a stopping condition is met.

## 2. Basic Structure

A recursive CTE usually has three parts:

1. **Anchor member:** Produces the initial rows. It does not refer to the CTE itself.
2. **Recursive member:** Refers to the CTE and produces the next set of rows.
3. **Termination:** Recursion stops when the recursive member produces no more rows, or when a configured limit or condition stops it.

### General syntax

```sql
WITH RECURSIVE cte_name AS (
    -- Anchor member
    SELECT ...

    UNION ALL

    -- Recursive member
    SELECT ...
    FROM source_table
    JOIN cte_name
        ON ...
)
SELECT *
FROM cte_name;
```

The exact syntax varies by database. PostgreSQL, MySQL 8+, and SQLite support `WITH RECURSIVE`. SQL Server uses `WITH` without the `RECURSIVE` keyword.

## 3. Example 1: Generate a Sequence of Numbers

Suppose we want to generate numbers from 1 to 5 without using a permanent numbers table.

```sql
WITH RECURSIVE numbers AS (
    -- Anchor member
    SELECT 1 AS n

    UNION ALL

    -- Recursive member
    SELECT n + 1
    FROM numbers
    WHERE n < 5
)
SELECT n
FROM numbers;
```

### Output

| n |
|---:|
| 1 |
| 2 |
| 3 |
| 4 |
| 5 |

### How it works

| Step | Operation | Result |
|---|---|---:|
| 1 | Anchor creates the first number | 1 |
| 2 | Recursive member adds 1 | 2 |
| 3 | Recursive member adds 1 | 3 |
| 4 | Recursive member adds 1 | 4 |
| 5 | Recursive member adds 1 | 5 |
| 6 | Condition `n < 5` is false for 5 | Stops |

The condition `WHERE n < 5` prevents the sequence from continuing indefinitely.

**Important:** The recursive member reads the current recursive result and generates the next result. The final query returns all rows accumulated by the CTE.

## 4. Example 2: Employee-Manager Hierarchy

Consider an `employees` table:

| employee_id | employee_name | manager_id |
|---:|---|---:|
| 1 | Anita | NULL |
| 2 | Bharat | 1 |
| 3 | Charu | 1 |
| 4 | Deepak | 2 |
| 5 | Esha | 2 |
| 6 | Farhan | 4 |

Here:
- Anita has no manager and is the top-level employee.
- Bharat and Charu report to Anita.
- Deepak and Esha report to Bharat.
- Farhan reports to Deepak.

### Goal

Return Anita and everyone who reports to her, directly or indirectly, with their hierarchy level.

```sql
WITH RECURSIVE employee_hierarchy AS (
    -- Anchor: start with the top-level employee
    SELECT
        employee_id,
        employee_name,
        manager_id,
        0 AS hierarchy_level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive member: find direct reports
    SELECT
        e.employee_id,
        e.employee_name,
        e.manager_id,
        h.hierarchy_level + 1
    FROM employees e
    JOIN employee_hierarchy h
        ON e.manager_id = h.employee_id
)
SELECT
    employee_id,
    employee_name,
    manager_id,
    hierarchy_level
FROM employee_hierarchy
ORDER BY hierarchy_level, employee_id;
```

### Output

| employee_id | employee_name | manager_id | hierarchy_level |
|---:|---|---:|---:|
| 1 | Anita | NULL | 0 |
| 2 | Bharat | 1 | 1 |
| 3 | Charu | 1 | 1 |
| 4 | Deepak | 2 | 2 |
| 5 | Esha | 2 | 2 |
| 6 | Farhan | 4 | 3 |

### Understanding the query

- **Anchor:** Finds Anita, who has no manager.
- **First recursive iteration:** Finds Bharat and Charu, whose `manager_id` is Anita's employee ID.
- **Second recursive iteration:** Finds Deepak and Esha, who report to Bharat.
- **Third recursive iteration:** Finds Farhan, who reports to Deepak.
- **Termination:** Stops when no additional employees report to the latest level.

This is a common pattern for exploring hierarchical data.

## 5. Example 3: Find All Subordinates of a Particular Manager

Suppose we want to find everyone reporting directly or indirectly to employee ID 2.

```sql
WITH RECURSIVE subordinates AS (
    -- Anchor: direct reports to employee 2
    SELECT
        employee_id,
        employee_name,
        manager_id,
        1 AS hierarchy_level
    FROM employees
    WHERE manager_id = 2

    UNION ALL

    -- Recursive member: reports to the previously found employees
    SELECT
        e.employee_id,
        e.employee_name,
        e.manager_id,
        s.hierarchy_level + 1
    FROM employees e
    JOIN subordinates s
        ON e.manager_id = s.employee_id
)
SELECT *
FROM subordinates
ORDER BY hierarchy_level, employee_id;
```

### Output

| employee_id | employee_name | manager_id | hierarchy_level |
|---:|---|---:|---:|
| 4 | Deepak | 2 | 1 |
| 5 | Esha | 2 | 1 |
| 6 | Farhan | 4 | 2 |

This query starts with the manager's direct reports rather than including the manager in the output.

## 6. `UNION ALL` vs. `UNION`

Recursive CTEs commonly use `UNION ALL` to combine the anchor and recursive members.

| `UNION ALL` | `UNION` |
|---|---|
| Keeps duplicate rows. | Removes duplicate rows. |
| Usually has less duplicate-removal overhead. | May require extra work to identify duplicates. |
| Appropriate when each recursive row should be preserved. | Can remove duplicate output rows, depending on the selected columns. |

**Important:** Do not assume that replacing `UNION ALL` with `UNION` always prevents infinite recursion. If each iteration produces a different row, such as a row with an increasing level number, duplicates may never occur. Use an appropriate termination condition and, where necessary, explicit cycle detection.

## 7. Recursive CTE vs. Regular CTE

| Regular CTE | Recursive CTE |
|---|---|
| Does not refer to itself. | Refers to itself in the recursive member. |
| Typically defines a named intermediate result. | Repeatedly generates new rows from previous results. |
| Useful for readability, filtering, and query organization. | Useful for hierarchies and iterative traversal. |
| No recursive termination condition is needed. | Needs a valid stopping condition or applicable recursion limit. |

## 8. Common Mistakes

### Mistake 1: Forgetting the termination condition

An unsafe number generator might look like this:

```sql
WITH RECURSIVE numbers AS (
    SELECT 1 AS n

    UNION ALL

    SELECT n + 1
    FROM numbers
)
SELECT *
FROM numbers;
```

The recursive member has no stopping condition. Depending on the database, the query may run until a recursion limit or resource limit is reached, or fail.

**Correction:**

```sql
WITH RECURSIVE numbers AS (
    SELECT 1 AS n

    UNION ALL

    SELECT n + 1
    FROM numbers
    WHERE n < 5
)
SELECT *
FROM numbers;
```

### Mistake 2: Using the wrong join condition

For an employee hierarchy, the recursive join should connect an employee to their manager:

```sql
ON e.manager_id = h.employee_id
```

Reversing this relationship can traverse the hierarchy in the wrong direction.

### Mistake 3: Confusing hierarchy level with employee ID

The employee ID identifies an employee. The hierarchy level indicates the employee's depth in the tree. They represent different things.

### Mistake 4: Assuming every employee will be returned

If the anchor starts at the top-level employee, the CTE only finds employees reachable from that anchor. Orphaned records and disconnected hierarchies may not appear.

### Mistake 5: Ignoring cycles

Invalid data may contain a reporting loop, such as employee A reporting to B while B reports to A. A recursive CTE may repeatedly revisit those rows.

Use cycle detection, a maximum depth, database-specific recursion safeguards, or data validation when cycles are possible.

### Mistake 6: Assuming recursion limits are identical across databases

Different database systems use different syntax and safeguards. Check the documentation for the database you're using rather than assuming one recursion-limit setting works everywhere.

## 9. Performance and Data Engineering Considerations

- Keep the anchor query selective so recursion starts with the intended rows.
- Ensure join columns, such as `manager_id` and `employee_id`, are appropriately indexed when supported and useful.
- Add a valid stopping condition and safeguards against cycles.
- Avoid carrying unnecessary columns through every recursive iteration.
- Validate parent-child relationships before processing large hierarchies.
- Consider whether a recursive query is appropriate for very deep or large hierarchies; alternative data models or precomputed hierarchy tables may be more suitable.

Recursive CTEs are particularly useful when the number of hierarchy levels is not known in advance.

## 10. Interview Questions

**Q1. What is a recursive CTE?**

A CTE that refers to itself to repeatedly process data, often used for hierarchical relationships and sequence generation.

**Q2. What are the three main parts of a recursive CTE?**

The anchor member, the recursive member, and the termination mechanism.

**Q3. What is the purpose of the anchor member?**

It establishes the initial rows from which recursion begins.

**Q4. Why is `UNION ALL` commonly used?**

It combines the anchor and recursive results without removing duplicates, generally avoiding the overhead of duplicate elimination.

**Q5. How would you find all employees reporting to a manager at any level?**

Start with the manager's direct reports, then recursively find employees whose `manager_id` matches each employee already found.

**Q6. What happens if recursion never terminates?**

The query may hit a database-specific recursion limit, consume excessive resources, or fail. A valid stopping condition and cycle safeguards are important.

**Q7. Can a recursive CTE generate dates?**

Yes. Start with an initial date and repeatedly add an interval until the required end date is reached. The date arithmetic syntax depends on the database.

**Q8. Does `UNION` always prevent infinite recursion?**

No. If each iteration creates a new distinct row, recursion can continue indefinitely despite duplicate elimination.

## 11. Practice Questions

Try writing the SQL yourself.

1. Generate numbers from 1 to 10.
2. Generate all dates between a given start date and end date.
3. Display every employee with their hierarchy level, starting from the top-level manager.
4. Find all direct and indirect subordinates of a given employee.
5. Find employees who have no manager.
6. Identify employees who cannot be reached from the organization's top-level employee.
7. Explain how you would detect or prevent a cycle in an employee-manager hierarchy.
8. Explain why `UNION ALL` is commonly preferred over `UNION` in recursive queries.

## 12. Quick Revision

- A recursive CTE refers to itself.
- The anchor member creates the starting rows.
- The recursive member produces additional rows.
- `UNION ALL` is commonly used to combine the two members.
- Recursion stops when no more rows are generated or a configured safeguard stops it.
- Common use cases include employee hierarchies, parent-child data, and generating sequences.
- Always consider termination, cycles, and performance.
- Syntax differs across database systems.

### Self-check

- [ ] Can you explain the anchor and recursive members?
- [ ] Can you generate a number sequence using a recursive CTE?
- [ ] Can you traverse an employee-manager hierarchy?
- [ ] Can you explain why a stopping condition matters?
- [ ] Can you distinguish `UNION` from `UNION ALL`?
- [ ] Can you describe how to handle cycles and recursion limits?

**Interview takeaway:** A recursive CTE is a way to process data level by level when the number of levels is not known in advance. Understand the anchor, recursive join, and termination condition first.