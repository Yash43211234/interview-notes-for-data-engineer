# EXISTS and NOT EXISTS in SQL

## 1. What Are EXISTS and NOT EXISTS?

`EXISTS` and `NOT EXISTS` check whether a subquery returns any rows.

- `EXISTS` returns true if the subquery returns at least one row.
- `NOT EXISTS` returns true if the subquery returns no rows.

**Simple definition:** `EXISTS` checks whether matching data is present. `NOT EXISTS` checks whether matching data is absent.

They are commonly used with correlated subqueries to find records that have or do not have related records in another table.

## 2. Sample Tables

### Customers

| customer_id | customer_name |
|---:|---|
| 1 | Amit |
| 2 | Neha |
| 3 | Ravi |
| 4 | Priya |

### Orders

| order_id | customer_id | amount |
|---:|---:|---:|
| 101 | 1 | 2000 |
| 102 | 1 | 1500 |
| 103 | 2 | 3000 |
| 104 | 4 | 1000 |

Ravi has no orders. Amit has two orders, Neha has one, and Priya has one.

## 3. EXISTS in SQL

**Question:** Find customers who have placed at least one order.

```sql
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

Result:

| customer_id | customer_name |
|---:|---|
| 1 | Amit |
| 2 | Neha |
| 4 | Priya |

### How it works

1. The outer query considers a customer.
2. The inner query checks whether an order exists for that customer.
3. If at least one matching order exists, `EXISTS` returns true.
4. The customer is included in the result.

Ravi is excluded because no order has `customer_id = 3`.

### Why use SELECT 1?

```sql
SELECT 1
FROM orders o
WHERE o.customer_id = c.customer_id
```

Inside `EXISTS`, the actual selected value does not matter. SQL checks whether a row exists, not whether the value `1` is meaningful.

You could select a column instead, but `SELECT 1` is a common convention.

## 4. NOT EXISTS in SQL

**Question:** Find customers who have never placed an order.

```sql
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

Result:

| customer_id | customer_name |
|---:|---|
| 3 | Ravi |

### How it works

1. The outer query considers a customer.
2. The inner query searches for matching orders.
3. If no matching order exists, `NOT EXISTS` returns true.
4. The customer is included in the result.

**Common use case:** Finding customers without orders, products without sales, or employees without assigned projects.

## 5. EXISTS vs. NOT EXISTS

| EXISTS | NOT EXISTS |
|---|---|
| Checks whether at least one row exists | Checks whether no rows exist |
| Returns true when a matching row is found | Returns true when no matching row is found |
| Finds customers who have orders | Finds customers who have no orders |
| Commonly used to check presence | Commonly used to check absence |

Both return a Boolean condition that can be used in `WHERE`.

## 6. EXISTS with Additional Conditions

You can add conditions inside the subquery to check for specific matching rows.

**Question:** Find customers who have placed an order worth more than 2,000.

```sql
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
      AND o.amount > 2000
);
```

Result:

| customer_id | customer_name |
|---:|---|
| 2 | Neha |

Amit's orders are 2,000 and 1,500. Neither is greater than 2,000. Neha has an order worth 3,000.

**Important:** `EXISTS` needs only one matching row. It does not require every order belonging to the customer to satisfy the condition.

## 7. NOT EXISTS with Additional Conditions

**Question:** Find customers who have never placed an order worth more than 2,000.

```sql
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
      AND o.amount > 2000
);
```

Result:

| customer_id | customer_name |
|---:|---|
| Amit |
| Ravi |
| Priya |

Neha is excluded because she has an order worth 3,000.

Notice that Amit is included even though he has orders. Neither of his orders is greater than 2,000.

**Remember:** `NOT EXISTS` means no row satisfies the condition inside the subquery. It does not necessarily mean no related rows exist at all.

## 8. Are EXISTS and NOT EXISTS Correlated Subqueries?

They are often used with correlated subqueries.

Example:

```sql
SELECT c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

The inner query references `c.customer_id`, which belongs to the outer query. This makes it a correlated subquery.

The database optimizer may use different execution strategies, so do not assume that the inner query must physically execute from scratch for every customer.

## 9. EXISTS vs. JOIN

Both can be useful when working with related tables, but they solve slightly different problems.

### Using EXISTS

Find customers who have orders.

```sql
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

Each qualifying customer appears once, even if they have multiple orders.

### Using INNER JOIN

```sql
SELECT c.customer_id, c.customer_name
FROM customers c
INNER JOIN orders o
    ON o.customer_id = c.customer_id;
```

This returns one row per matching customer-order combination. Amit appears twice because he has two orders.

If you need each customer only once, you could use `DISTINCT` with the join, but `EXISTS` often expresses the requirement more directly.

**Interview tip:** If the requirement is simply to check whether a related record exists, consider `EXISTS` rather than joining tables and then removing duplicates.

## 10. EXISTS vs. IN

Both can be used to check membership or matching records.

### Using IN

```sql
SELECT customer_id, customer_name
FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM orders
);
```

### Using EXISTS

```sql
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

For this example, both queries return the customers who have orders.

Important differences:

- `IN` compares a value with the values returned by a subquery.
- `EXISTS` checks whether the subquery returns any matching rows.
- `NOT IN` can produce unexpected results when its subquery contains `NULL`.
- `NOT EXISTS` avoids that particular `NULL` issue when checking for related rows.

Performance depends on the database optimizer and query structure; neither approach is always faster.

## 11. Common Mistakes

### Mistake 1: Forgetting the matching condition

Incorrect:

```sql
SELECT c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
);
```

If the orders table contains at least one row, this condition is true for every customer because the subquery is not linked to the current customer.

Correct:

```sql
SELECT c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

### Mistake 2: Thinking EXISTS counts matching rows

`EXISTS` only checks whether at least one row exists. It does not return the number of matching rows.

Use `COUNT()` if you need the number of orders.

### Mistake 3: Confusing NOT EXISTS with no related records

Consider:

```sql
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
      AND o.amount > 2000
)
```

This means the customer has no order worth more than 2,000. The customer may still have other orders.

### Mistake 4: Assuming EXISTS returns unique outer rows automatically

`EXISTS` does not multiply outer rows because multiple matching inner rows exist. However, if the outer query itself produces duplicates through another join, `EXISTS` will not automatically remove them.

### Mistake 5: Assuming EXISTS is always faster than JOIN

Performance depends on indexes, table sizes, the execution plan, and the database. Choose the clearest query first and inspect performance when needed.

## 12. Interview Questions

**Q1. What is EXISTS in SQL?**

It checks whether a subquery returns at least one row.

**Q2. What is NOT EXISTS?**

It checks whether a subquery returns no rows.

**Q3. What is the difference between EXISTS and JOIN?**

`EXISTS` filters outer rows based on whether matching rows exist. A join combines matching rows and can return multiple rows for one outer record.

**Q4. Why is SELECT 1 used inside EXISTS?**

Because the actual selected value does not matter. Only the existence of a row matters.

**Q5. Can EXISTS be used with a correlated subquery?**

Yes. This is a common pattern, where the inner query references a column from the outer query.

**Q6. How do you find customers who have never placed an order?**

Use `NOT EXISTS` with a condition matching orders to the current customer.

**Q7. What is the difference between NOT EXISTS and NOT IN?**

`NOT EXISTS` checks whether matching rows are absent. `NOT IN` compares against a set of values and can behave unexpectedly if that set contains `NULL`.

**Q8. Does EXISTS stop checking after finding one row?**

Logically, it only needs one matching row to be true. A database may optimize execution accordingly, but the physical execution strategy is implementation-dependent.

**Q9. Can EXISTS be used with conditions such as amount greater than 2,000?**

Yes. Put the condition inside the subquery to check whether at least one matching row satisfies it.

**Q10. When should you use NOT EXISTS in data engineering?**

It is useful for finding missing relationships, detecting records without matching data, and identifying customers, products, or entities with no related transactions.

## 13. Practice Questions

Use the `customers` and `orders` tables from this file.

1. Find customers who have placed at least one order.
2. Find customers who have never placed an order.
3. Find customers who have at least one order worth more than 1,500.
4. Find customers who have no order worth more than 1,500.
5. Find customers who have at least one order worth exactly 1,000.
6. Find customers who have never placed an order worth exactly 1,000.
7. Solve Question 1 using both `IN` and `EXISTS`.
8. Solve Question 2 using both `NOT EXISTS` and a `LEFT JOIN`.
9. Explain why Amit appears twice in the `INNER JOIN` example but once in the `EXISTS` example.
10. Explain how `NOT EXISTS` differs from `NOT IN` when `NULL` values are present.

## 14. Quick Revision

- `EXISTS` is true if a subquery returns at least one row.
- `NOT EXISTS` is true if a subquery returns no rows.
- The selected value inside `EXISTS` does not matter.
- Correlation conditions link inner rows to the current outer row.
- `EXISTS` is useful for checking whether related data exists.
- `NOT EXISTS` is useful for finding missing relationships.
- Multiple matching inner rows do not multiply the outer rows.
- `NOT EXISTS` is often a safer alternative to `NOT IN` when `NULL` values may be present.
- Performance depends on the query and database optimizer.

### Remember these patterns

**Find records with matching data:**

```sql
SELECT c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

**Find records without matching data:**

```sql
SELECT c.customer_name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

## 15. Self-Check

- [ ] I can explain EXISTS and NOT EXISTS in simple language.
- [ ] I can write correlated subqueries using both.
- [ ] I can filter matching rows using additional conditions.
- [ ] I understand the difference between EXISTS and JOIN.
- [ ] I understand the difference between EXISTS and IN.
- [ ] I know why NOT EXISTS can be safer than NOT IN with NULL values.
- [ ] I can solve the practice questions without looking at the examples.

**Revision status:** Needs Revision / Practicing / Interview Ready