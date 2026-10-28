# CROSS JOIN in SQL

## 1. What Is a CROSS JOIN?

A `CROSS JOIN` combines every row from the first table with every row from the second table.

It produces the **Cartesian product** of the two tables.

**Simple definition:** Every row in table A is paired with every row in table B.

If table A contains 3 rows and table B contains 4 rows, the result contains:

\[
3 \times 4 = 12 \text{ rows}
\]

## 2. Sample Tables

### Products

| product_id | product_name |
|---:|---|
| 1 | Laptop |
| 2 | Mouse |
| 3 | Keyboard |

### Colors

| color_id | color_name |
|---:|---|
| 1 | Black |
| 2 | White |

## 3. Basic CROSS JOIN Example

**Question:** Generate every possible product and color combination.

```sql
SELECT
    p.product_name,
    c.color_name
FROM products p
CROSS JOIN colors c;
```

Result:

| product_name | color_name |
|---|---|
| Laptop | Black |
| Laptop | White |
| Mouse | Black |
| Mouse | White |
| Keyboard | Black |
| Keyboard | White |

There are 3 products and 2 colors, so the query returns 6 rows.

## 4. How CROSS JOIN Works

Suppose the first table contains:

| product |
|---|
| Laptop |
| Mouse |

The second table contains:

| color |
|---|
| Black |
| White |
| Blue |

The CROSS JOIN produces:

| product | color |
|---|---|
| Laptop | Black |
| Laptop | White |
| Laptop | Blue |
| Mouse | Black |
| Mouse | White |
| Mouse | Blue |

Each product is paired with every color.

There is no matching condition because a CROSS JOIN does not require an `ON` clause.

## 5. Calculate the Number of Rows

The number of result rows is:

\[
\text{Result rows} = \text{Rows in A} \times \text{Rows in B}
\]

Examples:

| Table A | Table B | Result rows |
|---:|---:|---:|
| 2 | 3 | 6 |
| 4 | 5 | 20 |
| 10 | 10 | 100 |
| 100 | 1,000 | 100,000 |

If either table has zero rows, the result contains zero rows.

**Interview tip:** A CROSS JOIN can produce a very large result even when both input tables are relatively small.

## 6. CROSS JOIN with WHERE

You can filter the combinations produced by a CROSS JOIN using `WHERE`.

Example:

```sql
SELECT
    p.product_name,
    c.color_name
FROM products p
CROSS JOIN colors c
WHERE c.color_name = 'Black';
```

This returns only the combinations involving the Black color.

The CROSS JOIN conceptually generates all combinations, and the filter keeps the rows that satisfy the condition. A database optimizer may use a more efficient execution strategy.

## 7. Practical Use Cases

CROSS JOIN is useful when you intentionally need every possible combination.

Common examples:

- Generate all product and color combinations.
- Create combinations of dates and stores.
- Build a reporting grid for every department and month.
- Generate test data.
- Create all combinations of product sizes and colors.
- Compare every candidate item against every option.

### Example: Departments and Months

Suppose you need a report containing every department for every month, even when a department has no sales during a month.

```sql
SELECT
    d.department_name,
    m.month_name
FROM departments d
CROSS JOIN months m;
```

If there are 5 departments and 12 months, the result contains 60 department-month combinations.

You can later use a `LEFT JOIN` to attach sales data to these combinations.

## 8. CROSS JOIN vs. INNER JOIN

| CROSS JOIN | INNER JOIN |
|---|---|
| Produces every possible pair of rows | Returns rows that satisfy the join condition |
| Does not require an `ON` condition | Usually uses an `ON` condition |
| Result size is the product of the input row counts | Result size depends on matching rows |
| Useful for generating combinations | Useful for connecting related records |

### CROSS JOIN

```sql
SELECT p.product_name, c.color_name
FROM products p
CROSS JOIN colors c;
```

Returns all product-color combinations.

### INNER JOIN

```sql
SELECT o.order_id, c.customer_name
FROM orders o
INNER JOIN customers c
    ON o.customer_id = c.customer_id;
```

Returns orders with matching customer records.

**Key difference:** CROSS JOIN does not look for matching keys. INNER JOIN uses a condition to determine which rows match.

## 9. CROSS JOIN vs. Self Join

A CROSS JOIN combines every row from one table with every row from another table.

A self join joins a table to itself, usually using aliases and a condition to relate rows.

Example of a self join:

```sql
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees e
JOIN employees m
    ON e.manager_id = m.employee_id;
```

This connects employees to their managers rather than generating every possible employee pair.

## 10. Common Mistakes

### Mistake 1: Forgetting how quickly the result grows

If a table has 5,000 rows and another has 2,000 rows, the potential result is:

\[
5000 \times 2000 = 10{,}000{,}000
\]

That is 10 million rows.

Avoid an unintended CROSS JOIN on large tables.

### Mistake 2: Expecting matching rows only

Incorrect expectation:

```sql
SELECT *
FROM products
CROSS JOIN colors;
```

This does not match products to colors using an ID. It generates every combination.

### Mistake 3: Adding an unnecessary ON condition

A standard CROSS JOIN does not need an `ON` clause.

If you need to match related rows, consider an `INNER JOIN` instead.

### Mistake 4: Confusing CROSS JOIN with UNION

- `CROSS JOIN` combines columns from pairs of rows and generates combinations.
- `UNION` combines the results of compatible `SELECT` queries vertically.

They solve different problems.

### Mistake 5: Assuming the output order is guaranteed

SQL does not guarantee row order without an `ORDER BY` clause.

## 11. Interview Questions

**Q1. What is a CROSS JOIN?**

It combines every row of one table with every row of another table.

**Q2. What is a Cartesian product?**

It is the set of all possible pairs formed from two sets of rows.

**Q3. How many rows will a CROSS JOIN produce?**

If the tables contain \(m\) and \(n\) rows, the result contains \(m \times n\) rows.

**Q4. Does CROSS JOIN require an ON condition?**

No. It does not require a matching condition.

**Q5. What happens if one table is empty?**

The result is empty.

**Q6. Can we use WHERE with CROSS JOIN?**

Yes. `WHERE` filters the generated combinations.

**Q7. When would you use CROSS JOIN in data engineering?**

When generating complete combinations for reporting, building date-store grids, creating test data, or ensuring every category-period combination exists.

**Q8. Why can CROSS JOIN be dangerous on large tables?**

The result size grows by multiplying the number of rows in the input tables, which can consume substantial processing time and memory.

**Q9. What is the difference between CROSS JOIN and INNER JOIN?**

CROSS JOIN generates all pairs. INNER JOIN returns pairs that satisfy a matching condition.

**Q10. Can CROSS JOIN be used with the same table twice?**

Yes. It produces every possible pair of rows from that table, including a row paired with itself unless filtered out.

## 12. Practice Questions

Use the `products` and `colors` tables from this file.

1. Generate all possible product-color combinations.
2. Predict the number of rows if there are 8 products and 5 colors.
3. Generate all combinations of products and colors, but return only Black combinations.
4. Generate every combination of 4 departments and 12 months.
5. Explain how CROSS JOIN differs from INNER JOIN.
6. Explain what happens when one input table is empty.
7. Write a query that pairs every employee with every department.
8. Explain why an accidental CROSS JOIN can cause performance problems.

Try writing the SQL queries before checking the examples.

## 13. Quick Revision

- `CROSS JOIN` combines every row from table A with every row from table B.
- It produces a Cartesian product.
- Result rows = rows in A × rows in B.
- It does not require an `ON` condition.
- `WHERE` can filter the generated combinations.
- It is useful for generating all possible combinations and complete reporting grids.
- It can create huge results, so use it intentionally.
- Use `ORDER BY` if the output needs a specific order.

### Remember this pattern

```sql
SELECT
    a.column_name,
    b.column_name
FROM table_a a
CROSS JOIN table_b b;
```

## 14. Self-Check

- [ ] I can explain CROSS JOIN in simple language.
- [ ] I can calculate the expected number of rows.
- [ ] I can write a CROSS JOIN query.
- [ ] I understand how WHERE filters combinations.
- [ ] I can distinguish CROSS JOIN from INNER JOIN and UNION.
- [ ] I understand its practical uses and performance risks.
- [ ] I can solve the practice questions without looking at the examples.

**Revision status:** Needs Revision / Practicing / Interview Ready