# Data Grain and Aggregation

## 1. Introduction

**Data grain** defines the level of detail represented by each row in a dataset or fact table.

**Aggregation** is the process of combining multiple records to calculate summarized values, such as total sales, average revenue, or order counts.

These two concepts are closely related in data warehousing because the grain determines which aggregations are valid and how measures should be calculated.

### Simple Example

Consider the following sales data:

| order_id | product | quantity | sales_amount |
|---|---|---:|---:|
| O001 | Laptop | 1 | 50,000 |
| O001 | Mouse | 2 | 2,000 |
| O002 | Laptop | 1 | 50,000 |
| O003 | Keyboard | 1 | 3,000 |

Each row represents one product line within an order.

The grain is:

**One row represents one product line within one customer order.**

Total sales can be calculated by aggregating all rows:

\[
50,000 + 2,000 + 50,000 + 3,000 = 105,000
\]

But counting rows gives four order items, not four orders.

There are only three distinct orders.

This difference is important when designing analytical queries.

---

## 2. What Is Data Grain?

Data grain describes exactly what one record represents.

It is the lowest level of detail stored in a particular fact table.

### Common Examples

| Dataset | Grain |
|---|---|
| Sales fact | One row per order item |
| Order fact | One row per order |
| Payment fact | One row per payment transaction |
| Inventory fact | One row per product per store per day |
| Employee snapshot | One row per employee per month |
| Website activity | One row per user event |

### Why Grain Matters

Grain determines:

- What each row represents.
- Which measures can be stored.
- Which dimensions can be associated with a record.
- How records should be joined.
- Which aggregations are valid.
- Whether a query may double-count values.

**Important:** Always define the grain before designing a fact table.

---

## 3. Different Levels of Data Grain

Consider an e-commerce business.

The same sales information can be stored at different levels.

### 3.1 Transaction-Level Grain

One row represents an individual business event.

Example: One row per product line in an order.

| order_id | product_id | quantity | sales_amount |
|---|---|---:|---:|
| O001 | P001 | 1 | 50,000 |
| O001 | P002 | 2 | 2,000 |
| O002 | P001 | 1 | 50,000 |

This provides detailed information about individual sales events.

### 3.2 Order-Level Grain

One row represents one complete order.

| order_id | customer_id | order_total |
|---|---|---:|
| O001 | C001 | 52,000 |
| O002 | C002 | 50,000 |

This is useful for order-level reporting.

### 3.3 Daily Aggregated Grain

One row represents sales for a particular day, product, and city.

| sale_date | product_id | city | total_sales |
|---|---|---|---:|
| 2026-10-01 | P001 | Kolkata | 100,000 |
| 2026-10-01 | P002 | Delhi | 20,000 |
| 2026-10-02 | P001 | Kolkata | 50,000 |

This is useful for summarized reporting.

### 3.4 Monthly Aggregated Grain

One row represents the total sales for a month and product.

| month | product_id | total_sales |
|---|---|---:|
| October 2026 | P001 | 150,000 |
| October 2026 | P002 | 20,000 |

This reduces the level of detail and is useful for monthly analytics.

### Key Difference

As data moves from transaction-level to daily or monthly aggregation, the number of records generally decreases, but the original transaction-level details may no longer be available in that aggregated dataset.

---

## 4. What Is Aggregation?

Aggregation combines multiple records to produce summarized results.

Common aggregation functions include:

| Function | Purpose |
|---|---|
| SUM() | Calculates total |
| COUNT() | Counts records or non-NULL values, depending on expression |
| COUNT(DISTINCT) | Counts unique non-NULL values |
| AVG() | Calculates average |
| MIN() | Finds minimum value |
| MAX() | Finds maximum value |

### Example

Sales table:

| order_id | sales_amount |
|---|---:|
| O001 | 5,000 |
| O002 | 7,000 |
| O003 | 3,000 |

SQL:

```sql
SELECT
    SUM(sales_amount) AS total_sales,
    AVG(sales_amount) AS average_sales,
    MIN(sales_amount) AS minimum_sales,
    MAX(sales_amount) AS maximum_sales,
    COUNT(*) AS order_count
FROM sales;
```

Expected result:

| total_sales | average_sales | minimum_sales | maximum_sales | order_count |
|---:|---:|---:|---:|---:|
| 15,000 | 5,000 | 3,000 | 7,000 | 3 |

The result depends on the table's grain. If the table contains one row per order item, `COUNT(*)` counts order items, not orders.

---

## 5. Grain and Aggregation Relationship

Grain and aggregation must be understood together.

Suppose a fact table has one row per order item.

| order_id | product | city | sales_amount |
|---|---|---|---:|
| O001 | Laptop | Kolkata | 50,000 |
| O001 | Mouse | Kolkata | 2,000 |
| O002 | Laptop | Delhi | 50,000 |

The same data can answer different questions.

### Total Sales

```sql
SELECT SUM(sales_amount) AS total_sales
FROM fact_sales;
```

Result: ₹1,02,000.

### Sales by Product

```sql
SELECT
    product,
    SUM(sales_amount) AS total_sales
FROM fact_sales
GROUP BY product;
```

### Sales by City

```sql
SELECT
    city,
    SUM(sales_amount) AS total_sales
FROM fact_sales
GROUP BY city;
```

### Number of Orders

```sql
SELECT COUNT(DISTINCT order_id) AS total_orders
FROM fact_sales;
```

The grain determines what each row represents, while `GROUP BY` determines the level at which the result is summarized.

---

## 6. Types of Measures Based on Aggregation

Not every measure can be aggregated in the same way.

There are three important categories.

### 6.1 Additive Measures

Additive measures can be summed across all relevant dimensions.

Examples:
- Sales amount.
- Quantity sold.
- Discount amount.
- Number of units shipped.

Example:

| product | city | sales_amount |
|---|---|---:|
| Laptop | Kolkata | 50,000 |
| Laptop | Delhi | 40,000 |
| Mouse | Kolkata | 2,000 |

Total sales:

\[
50,000 + 40,000 + 2,000 = 92,000
\]

SQL:

```sql
SELECT SUM(sales_amount) AS total_sales
FROM fact_sales;
```

### 6.2 Semi-Additive Measures

Semi-additive measures can be summed across some dimensions but not across all dimensions.

A common example is account balance.

Suppose a bank stores daily balances:

| account_id | date | balance |
|---|---|---:|
| A001 | Oct 1 | 10,000 |
| A001 | Oct 2 | 12,000 |
| A001 | Oct 3 | 11,000 |

Adding the balances across dates gives ₹33,000, but this is not a meaningful account balance.

Balances can be summed across accounts for the same date, but should generally not be summed across time.

Common examples:
- Bank account balance.
- Inventory on hand.
- Outstanding loan balance.
- Headcount at a point in time.

### 6.3 Non-Additive Measures

Non-additive measures cannot be meaningfully summed directly.

Examples:
- Percentage.
- Profit margin.
- Conversion rate.
- Average.
- Ratios.

For example:

\[
\text{Profit Margin} =
\frac{\text{Profit}}{\text{Revenue}} \times 100
\]

To calculate an overall profit margin, aggregate the underlying profit and revenue first:

\[
\text{Overall Margin} =
\frac{\text{Total Profit}}{\text{Total Revenue}} \times 100
\]

Do not simply sum individual percentages or calculate an unweighted average unless that is explicitly the intended business metric.

### Comparison

| Measure Type | Can Sum Across Dimensions? | Example |
|---|---|---|
| Additive | Across all relevant dimensions | Sales amount |
| Semi-additive | Across some dimensions | Account balance |
| Non-additive | Not directly | Percentage |

---

## 7. Aggregation Using GROUP BY

`GROUP BY` divides records into groups based on one or more columns and calculates aggregate values for each group.

### Example: Sales by Product

```sql
SELECT
    product,
    SUM(sales_amount) AS total_sales
FROM fact_sales
GROUP BY product;
```

### Example: Monthly Sales

```sql
SELECT
    EXTRACT(YEAR FROM sale_date) AS sales_year,
    EXTRACT(MONTH FROM sale_date) AS sales_month,
    SUM(sales_amount) AS monthly_sales
FROM fact_sales
GROUP BY
    EXTRACT(YEAR FROM sale_date),
    EXTRACT(MONTH FROM sale_date)
ORDER BY
    sales_year,
    sales_month;
```

Date extraction syntax varies across SQL engines.

### Example: Multiple Dimensions

```sql
SELECT
    city,
    product,
    SUM(sales_amount) AS total_sales
FROM fact_sales
GROUP BY
    city,
    product;
```

This produces sales summaries for each city-product combination.

---

## 8. Aggregation and Data Granularity

Data granularity refers to how detailed or summarized the data is.

- Fine granularity: More detailed records.
- Coarse granularity: More summarized records.

### Example

| Granularity | Example |
|---|---|
| Fine | One row per order item |
| Medium | One row per day and product |
| Coarse | One row per month and product |

### Fine vs Coarse Granularity

| Aspect | Fine Granularity | Coarse Granularity |
|---|---|---|
| Detail | High | Low |
| Number of rows | Usually more | Usually fewer |
| Storage | Usually higher | Usually lower |
| Flexibility | Supports detailed analysis | Limited to stored dimensions |
| Query workload | May require more aggregation | Often faster for matching summaries |

A common warehouse design retains detailed fact data and creates aggregate tables or materialized views for frequently used reporting workloads.

However, aggregate tables must be designed carefully so that they preserve the measures needed for correct calculations.

---

## 9. Aggregation Problems in Data Warehousing

### 9.1 Double Counting

Suppose an order has two items:

| order_id | item | sales_amount |
|---|---|---:|
| O001 | Laptop | 50,000 |
| O001 | Mouse | 2,000 |

The order total is ₹52,000.

If an order-level amount of ₹52,000 is repeated on each item row, summing that column produces ₹1,04,000.

**Solution:** Store measures at their correct grain and avoid repeating higher-level measures in lower-grain fact tables unless their aggregation behavior is explicitly controlled.

### 9.2 Joining Tables with Different Grains

Suppose one table contains one row per order and another contains multiple rows per order item.

Joining them on `order_id` repeats the order-level values for every matching item.

This can inflate totals.

**Solution:** Understand the grain and relationship cardinality before joining. Aggregate one side to the required grain or use separate fact tables where appropriate.

### 9.3 Incorrect Distinct Counts

`COUNT(*)` counts rows, not unique business entities.

For example:

```sql
SELECT COUNT(DISTINCT order_id)
FROM fact_sales;
```

This counts unique orders even when an order has multiple item rows.

### 9.4 Incorrect Average Calculation

Suppose two stores have average order values of ₹1,000 and ₹2,000.

A simple average of these two averages is ₹1,500, but the true combined average depends on the number of orders in each store.

Correct calculation:

\[
\text{Combined Average Order Value} =
\frac{\text{Combined Revenue}}{\text{Combined Order Count}}
\]

Whenever possible, retain the components needed to calculate the metric correctly.

---

## 10. Pre-Aggregation and Aggregate Tables

Pre-aggregation means calculating summaries before the business query requests them.

For example, instead of calculating daily sales from millions of transaction rows every time, a pipeline can maintain a daily sales aggregate.

### Transaction-Level Fact

`fact_sales`

| sale_date | product_id | city | sales_amount |
|---|---|---|---:|
| Oct 1 | P001 | Kolkata | 50,000 |
| Oct 1 | P001 | Kolkata | 30,000 |
| Oct 1 | P002 | Delhi | 10,000 |

### Daily Aggregate Table

`agg_daily_sales`

| sale_date | product_id | city | total_sales |
|---|---|---|---:|
| Oct 1 | P001 | Kolkata | 80,000 |
| Oct 1 | P002 | Delhi | 10,000 |

### Benefits

- Faster reporting queries.
- Less repeated computation.
- Lower query processing cost for matching workloads.
- Easier consumption by dashboards.

### Limitations

- Additional storage.
- Extra pipeline maintenance.
- Late-arriving data may require corrections.
- Pre-aggregated data cannot answer every detailed question.

**Important:** Keep the detailed source fact when detailed analysis and future flexibility are required.

---

## 11. Data Grain and Aggregation in PySpark

The same concepts apply to PySpark DataFrames.

### Example Dataset

```python
from pyspark.sql import functions as F

data = [
    ("O001", "Laptop", "Kolkata", 50000),
    ("O001", "Mouse", "Kolkata", 2000),
    ("O002", "Laptop", "Delhi", 50000)
]

columns = [
    "order_id",
    "product",
    "city",
    "sales_amount"
]

df = spark.createDataFrame(data, columns)
```

The grain is one row per order item.

### Total Sales

```python
df.agg(
    F.sum("sales_amount").alias("total_sales")
).show()
```

### Sales by Product

```python
df.groupBy("product").agg(
    F.sum("sales_amount").alias("total_sales")
).show()
```

### Sales by City

```python
df.groupBy("city").agg(
    F.sum("sales_amount").alias("total_sales")
).show()
```

### Distinct Order Count

```python
df.agg(
    F.countDistinct("order_id").alias("total_orders")
).show()
```

### Daily Aggregation

```python
daily_sales = df.groupBy(
    "sale_date",
    "product"
).agg(
    F.sum("sales_amount").alias("total_sales")
)
```

This last example assumes the DataFrame contains a `sale_date` column. Add it to the source schema before running the code.

---

## 12. Practical Data Engineering Scenario

Consider an e-commerce company processing millions of order items every day.

The business needs:
- Daily revenue.
- Monthly revenue.
- Product-level sales.
- City-level sales.
- Number of unique orders.

A suitable approach:

1. Ingest detailed order-item records into the Bronze layer.
2. Validate and standardize records in Silver.
3. Build a transaction-level sales fact table.
4. Define the fact grain as one row per order item.
5. Calculate daily and monthly aggregates for reporting.
6. Use distinct order counts for order-level metrics.
7. Reconcile aggregate totals against the detailed fact table.
8. Handle late-arriving updates and corrections.

### Example Architecture

```text
Source Order Data
       |
       v
Bronze - Raw Order Items
       |
       v
Silver - Cleaned Order Items
       |
       v
Gold - Transaction Fact
       |
       +------------------+
       |                  |
       v                  v
Daily Sales Aggregate   Monthly Sales Aggregate
       |                  |
       v                  v
Daily Dashboard         Monthly Dashboard
```

The detailed fact table remains the source for flexible analysis, while aggregate tables support common reporting queries.

---

## 13. Interview Questions and Answers

### Q1. What is data grain?

Data grain defines exactly what one row in a dataset or fact table represents.

### Q2. Why is grain important in dimensional modeling?

Grain determines the level of detail, the measures that belong in the fact table, and how the data can be joined and aggregated. Incorrect grain can cause data duplication and inaccurate reporting.

### Q3. What is aggregation?

Aggregation combines multiple records to calculate summarized values, such as total sales, average revenue, or order counts.

### Q4. What is the difference between grain and aggregation?

Grain defines the detail stored in each row. Aggregation summarizes multiple rows into a higher-level result.

### Q5. What are additive, semi-additive, and non-additive measures?

- Additive: Can be summed across all relevant dimensions.
- Semi-additive: Can be summed across some dimensions, but not all.
- Non-additive: Cannot be meaningfully summed directly.

### Q6. Why is account balance semi-additive?

Account balances can be summed across accounts at the same point in time, but adding daily balances across dates does not produce a meaningful balance.

### Q7. Why can joining fact tables cause double counting?

Fact tables may have different grains and one-to-many relationships. Joining them directly can multiply rows and repeat measures.

### Q8. What is the difference between transaction-level and aggregated data?

Transaction-level data stores individual events. Aggregated data summarizes multiple events at a higher level, such as daily or monthly sales.

### Q9. What is pre-aggregation?

Pre-aggregation calculates and stores commonly used summaries before reporting queries run, reducing repeated computation.

### Q10. Does a lower row count always mean a better data model?

No. Reducing rows can remove useful detail. The model should preserve the level of detail required by business requirements.

### Q11. How do you calculate an overall average from multiple groups?

Use the combined total divided by the combined count, rather than blindly averaging group averages.

### Q12. How do you count orders when the fact table has one row per order item?

Use `COUNT(DISTINCT order_id)` or an appropriate order-level aggregation.

### Q13. Can a monthly aggregate table answer product-level daily questions?

Only if it retains the required product and daily dimensions. If daily detail has been discarded, it cannot reliably answer questions at that level.

### Q14. How do you handle late-arriving data in aggregate tables?

Identify affected periods or groups, correct the detailed fact data, and recompute or incrementally update the affected aggregates using retry-safe logic.

### Q15. How do you validate an aggregate table?

Compare its totals and counts with the detailed fact table at matching dimensions and time periods, accounting for filters and business rules.

---

## 14. Common Interview Mistakes

| Mistake | Correct Understanding |
|---|---|
| Grain means the number of rows | Grain defines what one row represents |
| Aggregation and grain are the same | Grain is stored detail; aggregation summarizes it |
| `COUNT(*)` counts unique orders | It counts rows |
| Every measure can be summed | Measures have different aggregation behavior |
| Account balance can be summed across dates | It is generally semi-additive |
| Averages can always be averaged | Use underlying totals and counts |
| Joining fact tables is always safe | Check grain and relationship cardinality |
| Pre-aggregation replaces detailed data | It usually complements detailed fact tables |
| Fewer rows always means better design | Required analytical detail must be preserved |

---

## 15. Quick Revision

| Concept | Remember |
|---|---|
| Data grain | What one row represents |
| Fine grain | More detailed records |
| Coarse grain | More summarized records |
| Aggregation | Combining records into summaries |
| Additive | Summable across relevant dimensions |
| Semi-additive | Summable across some dimensions |
| Non-additive | Not directly summable |
| Pre-aggregation | Store commonly used summaries |
| Double counting | Repeated measures due to grain or join mismatch |

### Most Important Points

1. Define grain before designing a fact table.
2. Grain determines the detail represented by each row.
3. Aggregation summarizes records using functions such as SUM and COUNT.
4. `COUNT(*)` counts rows, not unique business entities.
5. Use `COUNT(DISTINCT ...)` when unique entity counts are required.
6. Additive measures can be summed across relevant dimensions.
7. Semi-additive measures cannot be summed across every dimension.
8. Non-additive measures require appropriate formulas.
9. Joining tables with different grains can cause double counting.
10. Pre-aggregated tables improve common reporting workloads but do not replace detailed data.

### One-Minute Interview Answer

Data grain defines exactly what one row in a fact table represents, while aggregation summarizes multiple rows to calculate business metrics.

For example, if a sales fact table has one row per order item, we can calculate product revenue using SUM, but we need COUNT DISTINCT to calculate the number of orders.

Measures can be additive, semi-additive, or non-additive. Sales amount is additive, account balance is generally semi-additive, and percentages are non-additive.

Defining the grain correctly is important because it prevents incorrect joins, double counting, and invalid aggregations. In a data warehouse, we often retain detailed fact data and build daily or monthly aggregate tables to support reporting performance.

---

## 16. Self-Assessment

- [ ] Define data grain.
- [ ] Explain why grain must be defined before fact table design.
- [ ] Differentiate transaction, order, daily, and monthly grain.
- [ ] Explain aggregation with SQL examples.
- [ ] Explain additive measures.
- [ ] Explain semi-additive measures.
- [ ] Explain non-additive measures.
- [ ] Calculate unique orders from order-item data.
- [ ] Identify double-counting problems caused by joins.
- [ ] Explain why averages cannot always be averaged.
- [ ] Explain pre-aggregation and its limitations.
- [ ] Perform aggregations using PySpark.
- [ ] Explain how to reconcile aggregate tables with detailed facts.

**Interview readiness:** Given a fact table and a business question, identify the grain, determine whether the requested measure can be aggregated safely, and write a query that produces the correct result.