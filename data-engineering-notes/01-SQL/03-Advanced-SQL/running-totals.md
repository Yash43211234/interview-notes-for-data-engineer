# SQL Running Totals

## 1. What Is a Running Total?

A **running total** is a cumulative sum that adds the current row's value to the values from all preceding rows in a specified order.

For example, if daily sales are 100, 200, 150, and 300, the running totals are 100, 300, 450, and 750.

Running totals are commonly used in Data Engineering and SQL interviews to:
- Calculate cumulative sales or revenue.
- Track account balances.
- Measure cumulative orders or transactions.
- Calculate month-to-date and year-to-date totals.
- Analyze cumulative quantities within each customer or product group.

The most common approach is to use the window function `SUM()` with `OVER()`.

## 2. Sample Data

Assume we have a `daily_sales` table:

| sale_date | sales |
|---|---:|
| 2026-01-01 | 100 |
| 2026-01-02 | 200 |
| 2026-01-03 | 150 |
| 2026-01-04 | 300 |
| 2026-01-05 | 250 |

We want to calculate the cumulative sales for each date.

## 3. Basic Running Total

```sql
SELECT
    sale_date,
    sales,
    SUM(sales) OVER (
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM daily_sales
ORDER BY sale_date;
```

### Output

| sale_date | sales | running_total |
|---|---:|---:|
| 2026-01-01 | 100 | 100 |
| 2026-01-02 | 200 | 300 |
| 2026-01-03 | 150 | 450 |
| 2026-01-04 | 300 | 750 |
| 2026-01-05 | 250 | 1000 |

### How it works

- First row: \(100\)
- Second row: \(100 + 200 = 300\)
- Third row: \(100 + 200 + 150 = 450\)
- Fourth row: \(100 + 200 + 150 + 300 = 750\)
- Fifth row: \(100 + 200 + 150 + 300 + 250 = 1000\)

The window function calculates the sum across a changing set of rows. It does not collapse the records into a single row like a regular aggregate query without a window function.

## 4. Understand the Window Frame

Consider this part of the query:

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

It defines which rows contribute to the sum for the current row.

| Keyword | Meaning |
|---|---|
| `ROWS` | Defines the frame using physical row positions in the ordered partition. |
| `UNBOUNDED PRECEDING` | Starts from the first row in the partition. |
| `CURRENT ROW` | Ends at the current row. |

Together, these keywords mean: **sum every row from the beginning of the partition up to the current row.**

Writing the frame explicitly is a good habit because the default frame can differ in behavior when ordering values contain ties.

## 5. Running Total for Each Customer

Suppose the `orders` table contains:

| order_id | customer_id | order_date | amount |
|---:|---|---|---:|
| 1 | C101 | 2026-01-01 | 500 |
| 2 | C101 | 2026-01-03 | 300 |
| 3 | C101 | 2026-01-05 | 200 |
| 4 | C102 | 2026-01-02 | 400 |
| 5 | C102 | 2026-01-04 | 600 |

### Requirement

Calculate the cumulative order amount for each customer separately.

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM orders
ORDER BY customer_id, order_date, order_id;
```

### Output

| order_id | customer_id | amount | running_total |
|---:|---|---:|---:|
| 1 | C101 | 500 | 500 |
| 2 | C101 | 300 | 800 |
| 3 | C101 | 200 | 1000 |
| 4 | C102 | 400 | 400 |
| 5 | C102 | 600 | 1000 |

The running total restarts for each customer because of `PARTITION BY customer_id`.

**Interview point:** `PARTITION BY` divides rows into independent groups, while `ORDER BY` determines the sequence in which the running total is calculated.

## 6. Running Total vs. Grouped Total

These queries answer different questions.

### Total sales per customer

```sql
SELECT
    customer_id,
    SUM(amount) AS total_sales
FROM orders
GROUP BY customer_id;
```

This returns one row per customer.

### Running sales per customer

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM orders;
```

This preserves each order row and calculates the cumulative amount at that point.

| `GROUP BY` | Window function |
|---|---|
| Usually reduces rows into groups. | Preserves individual rows. |
| Calculates a group-level total. | Calculates a value for each row based on a window of rows. |
| Useful for total sales by customer. | Useful for cumulative sales by customer. |

## 7. Running Total by Month

Suppose we want to calculate cumulative sales separately for each month.

Assume `daily_sales` contains one row per date.

### PostgreSQL example

```sql
SELECT
    sale_date,
    sales,
    SUM(sales) OVER (
        PARTITION BY DATE_TRUNC('month', sale_date)
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS monthly_running_total
FROM daily_sales
ORDER BY sale_date;
```

`DATE_TRUNC('month', sale_date)` identifies the month, and `PARTITION BY` restarts the running total at the beginning of each month.

**Note:** Date functions differ across database systems. For example, SQL Server commonly uses `DATEFROMPARTS(YEAR(sale_date), MONTH(sale_date), 1)` to create a month key. MySQL commonly uses `DATE_FORMAT(sale_date, '%Y-%m-01')`.

## 8. Calculate a Running Account Balance

A transaction table may contain deposits and withdrawals.

| transaction_id | transaction_date | amount |
|---:|---|---:|
| 1 | 2026-01-01 | 1000 |
| 2 | 2026-01-02 | -200 |
| 3 | 2026-01-03 | 500 |
| 4 | 2026-01-04 | -100 |

Positive values represent deposits, and negative values represent withdrawals.

```sql
SELECT
    transaction_id,
    transaction_date,
    amount,
    SUM(amount) OVER (
        ORDER BY transaction_date, transaction_id
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS balance
FROM transactions
ORDER BY transaction_date, transaction_id;
```

### Output

| transaction_id | amount | balance |
|---:|---:|---:|
| 1 | 1000 | 1000 |
| 2 | -200 | 800 |
| 3 | 500 | 1300 |
| 4 | -100 | 1200 |

The cumulative sum tracks the balance after each transaction, assuming the initial balance is zero. If an opening balance exists, add it to the cumulative total.

## 9. Calculate a Running Total with an Opening Balance

Suppose the account starts with an opening balance of 500.

```sql
SELECT
    transaction_id,
    transaction_date,
    amount,
    500 + SUM(amount) OVER (
        ORDER BY transaction_date, transaction_id
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS balance
FROM transactions
ORDER BY transaction_date, transaction_id;
```

The opening balance is added to each cumulative transaction total.

For production financial systems, also consider currency precision, transaction ordering, corrections, and whether the opening balance is specific to each account.

## 10. Common Mistakes

### Mistake 1: Forgetting `ORDER BY`

```sql
SUM(sales) OVER ()
```

This calculates the total over the entire window, not a row-by-row running total.

**Correction:** Add an appropriate ordering column and a cumulative frame.

### Mistake 2: Forgetting `PARTITION BY`

If you calculate a running total for multiple customers without partitioning by customer, the cumulative total continues across customers instead of restarting for each customer.

### Mistake 3: Using a non-deterministic order

If multiple transactions share the same timestamp, ordering only by timestamp may not determine which transaction comes first.

Use a stable tie-breaker:

```sql
ORDER BY transaction_date, transaction_id
```

This ensures a predictable sequence if `transaction_id` is unique.

### Mistake 4: Ignoring the window frame

An expression such as:

```sql
SUM(sales) OVER (ORDER BY sale_date)
```

may use a default frame that treats rows with equal ordering values as peers. This can cause multiple rows to receive the same cumulative result.

For a row-by-row running total, explicitly specify:

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

### Mistake 5: Confusing a running total with a moving total

A running total includes all rows from the beginning through the current row. A moving total considers only a limited number of preceding rows and the current row.

For example, a three-row moving total uses:

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

## 11. Running Total vs. Moving Total

| Running total | Moving total |
|---|---|
| Starts at the beginning of the partition. | Uses a fixed or specified window of rows. |
| Includes all earlier rows and the current row. | Includes only the rows inside the defined frame. |
| Useful for cumulative revenue. | Useful for recent-period performance analysis. |

Example of a three-row moving total:

```sql
SELECT
    sale_date,
    sales,
    SUM(sales) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS three_row_total
FROM daily_sales
ORDER BY sale_date;
```

The first row sums one available row, the second sums two, and subsequent rows sum the current row plus the two preceding rows.

**Important:** A three-row frame is not necessarily a three-day frame. If dates are missing, it still counts rows, not calendar days.

## 12. Interview Questions

**Q1. What is a running total?**

A cumulative sum that includes values from the beginning of an ordered partition through the current row.

**Q2. Which SQL function is commonly used to calculate a running total?**

`SUM()` with an `OVER()` clause.

**Q3. What is the purpose of `PARTITION BY` in a running total?**

It divides the data into groups so the cumulative sum restarts for each group.

**Q4. Why specify `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`?**

It explicitly defines the frame from the first row through the current row, giving row-by-row cumulative behavior.

**Q5. What is the difference between a running total and `GROUP BY` aggregation?**

A grouped aggregate usually returns one row per group, while a window-based running total preserves the individual rows.

**Q6. How would you calculate a running balance?**

Use `SUM(amount) OVER (ORDER BY transaction_date, transaction_id ...)`, where deposits are positive and withdrawals are negative.

**Q7. How do you calculate a running total separately for each customer?**

Use `PARTITION BY customer_id` and order the rows by the appropriate date and tie-breaker.

**Q8. What is the difference between a running total and a moving total?**

A running total accumulates all rows from the beginning. A moving total uses a limited frame of preceding rows and the current row.

## 13. Practice Questions

Try writing each query without looking at the examples.

1. Calculate a running total of daily sales.
2. Calculate cumulative order amounts separately for each customer.
3. Calculate a running account balance with deposits and withdrawals.
4. Calculate a monthly running total that restarts each month.
5. Calculate a running total of product quantities for each product.
6. Calculate a three-row moving total.
7. Explain why a unique tie-breaker may be necessary when multiple transactions share the same date.
8. Explain how the output differs between `SUM(amount) GROUP BY customer_id` and a running sum using a window function.

## 14. Quick Revision

- A running total is a cumulative sum.
- Use `SUM(column) OVER (...)`.
- Use `ORDER BY` to define the row sequence.
- Use `PARTITION BY` when the running total must restart for each group.
- Use `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` for explicit row-by-row cumulative behavior.
- Add a tie-breaker when the ordering column is not unique.
- A running total differs from both a grouped total and a moving total.

### Self-check

- [ ] Can you calculate a running total with `SUM()`?
- [ ] Can you restart it for each customer or department?
- [ ] Can you explain the window frame?
- [ ] Can you calculate a running account balance?
- [ ] Can you distinguish a running total from a moving total?
- [ ] Can you handle tied dates with a deterministic ordering?

**Interview takeaway:** For most running-total questions, start with `SUM(value) OVER (PARTITION BY group_column ORDER BY date_column, unique_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)`. Adjust the partition and ordering columns to match the business requirement.