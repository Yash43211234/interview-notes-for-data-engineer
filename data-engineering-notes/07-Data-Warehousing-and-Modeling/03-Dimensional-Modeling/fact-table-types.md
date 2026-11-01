# Types of Fact Tables in Dimensional Modeling

## 1. Introduction

A **Fact Table** is the central table in a dimensional model that stores measurable business events, activities, or process states.

It usually contains:
- **Foreign keys:** Connect the fact table to dimension tables.
- **Measures:** Numeric values used for analysis.
- **Degenerate dimensions (sometimes):** Business identifiers stored directly in the fact table, such as an order number.

Fact tables are designed around a clearly defined **grain**.

The grain explains exactly what one row represents.

### Example: Sales Fact Table

| order_id | date_key | product_key | customer_key | quantity | sales_amount |
|---|---:|---:|---:|---:|---:|
| 1001 | 20261001 | 101 | 501 | 2 | 2000 |
| 1002 | 20261001 | 102 | 502 | 1 | 1500 |
| 1003 | 20261002 | 101 | 503 | 3 | 3000 |

If the grain is one row per product line in an order, each row represents one product line rather than one complete order.

---

## 2. Main Types of Fact Tables

There are three commonly taught fundamental fact table types:

1. Transaction Fact Table
2. Periodic Snapshot Fact Table
3. Accumulating Snapshot Fact Table

Other useful dimensional modeling patterns include:
- Factless Fact Table
- Aggregate Fact Table

These patterns describe different modeling purposes. They are not all mutually exclusive categories.

---

## 3. Transaction Fact Table

A **Transaction Fact Table** stores one row for each individual business event or transaction.

It captures what happened at a particular point in time.

### Example: Retail Sales

Suppose a customer purchases three products in one order.

The transaction fact table may contain one row per product line.

| transaction_id | order_id | product_key | date_key | quantity | sales_amount |
|---|---|---:|---:|---:|---:|
| 1 | 1001 | 101 | 20261001 | 2 | 2000 |
| 2 | 1001 | 102 | 20261001 | 1 | 1500 |
| 3 | 1002 | 101 | 20261002 | 3 | 3000 |

### Characteristics

- Each row represents an individual transaction or event at the defined grain.
- New rows are inserted as new events occur.
- Transaction details are preserved at the chosen level of detail.
- Measures can include quantity, sales amount, discount, and tax.
- Useful for detailed analysis and historical reporting.

### Common Use Cases

- Retail sales
- Bank transactions
- Payment processing
- Website clicks
- Product purchases
- Order line items

### SQL Example

```sql
SELECT
    product_key,
    SUM(quantity) AS total_quantity,
    SUM(sales_amount) AS total_sales
FROM fact_sales
GROUP BY product_key;
```

This calculates total sales and quantity by product.

---

## 4. Periodic Snapshot Fact Table

A **Periodic Snapshot Fact Table** stores the state or measurement of a business process at regular, predefined intervals.

Each row represents an entity or process at a particular reporting period.

### Example: Daily Inventory

Suppose a company wants to track product stock every day.

| date_key | product_key | warehouse_key | stock_quantity |
|---|---:|---:|---:|
| 20261001 | 101 | 10 | 100 |
| 20261002 | 101 | 10 | 90 |
| 20261003 | 101 | 10 | 80 |
| 20261001 | 102 | 10 | 50 |
| 20261002 | 102 | 10 | 45 |

### Characteristics

- One row represents an entity or process at a particular time interval.
- New rows are added for subsequent reporting periods.
- It captures historical states over time.
- Some measures are semi-additive or non-additive.
- Useful for trend analysis and periodic reporting.

### Common Use Cases

- Daily inventory balances
- Monthly account balances
- Daily website traffic summaries
- Monthly employee headcount
- Daily machine status
- Monthly loan balances

### SQL Example

```sql
SELECT
    date_key,
    SUM(stock_quantity) AS total_stock
FROM fact_inventory_snapshot
GROUP BY date_key
ORDER BY date_key;
```

This calculates total inventory across products and warehouses for each date.

**Important:** Inventory balance is generally semi-additive. It can be summed across products and warehouses for a particular date, but summing daily balances across dates usually does not produce a meaningful inventory measure.

---

## 5. Accumulating Snapshot Fact Table

An **Accumulating Snapshot Fact Table** tracks the progress of a business process through multiple milestones.

It usually contains one row per process instance. The same row is updated as the process progresses.

### Example: Order Fulfillment

An order moves through several stages:

```text
Order Placed
     |
     v
Payment Confirmed
     |
     v
Order Shipped
     |
     v
Order Delivered
```

The fact table stores milestone dates in separate columns.

| order_id | order_date | payment_date | shipped_date | delivered_date |
|---|---|---|---|---|
| 1001 | Oct 1 | Oct 1 | Oct 2 | Oct 4 |
| 1002 | Oct 2 | Oct 2 | Oct 3 | NULL |
| 1003 | Oct 3 | NULL | NULL | NULL |

### Characteristics

- One row represents one business process instance.
- Multiple milestone dates are stored in separate columns.
- Existing rows are updated as milestones occur.
- Useful for measuring the time taken between stages.
- NULL values can represent milestones that have not yet occurred.

### Common Use Cases

- Order fulfillment
- Loan approval
- Recruitment processes
- Insurance claims
- Manufacturing workflows
- Customer support tickets

### SQL Example

```sql
SELECT
    order_id,
    order_date,
    delivered_date,
    DATEDIFF(day, order_date, delivered_date)
        AS fulfillment_days
FROM fact_order_fulfillment
WHERE delivered_date IS NOT NULL;
```

The exact `DATEDIFF` syntax varies by database platform.

**Important:** An accumulating snapshot does not automatically preserve every historical milestone change. An event-history table may be needed when complete event history is required.

---

## 6. Factless Fact Table

A **Factless Fact Table** is a fact table that contains foreign keys but no numeric measures.

It records that an event occurred or that a relationship or condition existed.

There are two common uses:

1. Event tracking
2. Coverage tracking

### Example 1: Student Attendance

Suppose a school wants to track student attendance.

| student_key | class_key | date_key |
|---:|---:|---:|
| 101 | 10 | 20261001 |
| 102 | 10 | 20261001 |
| 103 | 10 | 20261001 |

Each row records an attendance event. There is no numeric measure, but the existence of a row represents attendance.

### Example 2: Course Coverage

Suppose a school wants to record which students are eligible for which courses.

| student_key | course_key | semester_key |
|---:|---:|---:|
| 101 | 201 | 1 |
| 101 | 202 | 1 |
| 102 | 201 | 1 |

This records a coverage or eligibility relationship.

### SQL Example: Count Attendance

```sql
SELECT
    date_key,
    class_key,
    COUNT(*) AS attendance_count
FROM fact_attendance
GROUP BY date_key, class_key;
```

A count can be derived from the number of rows even though the fact table contains no numeric measure.

### Characteristics

- Contains dimension foreign keys.
- Does not require numeric measures.
- Records events, participation, eligibility, or relationships.
- Can support counts and coverage analysis.

**Important:** A factless fact table and a bridge table are related concepts, but they are not identical. A bridge represents relationships; a factless fact table records events or coverage at a defined grain.

---

## 7. Aggregate Fact Table

An **Aggregate Fact Table** stores precomputed summaries of detailed fact data.

It is used to improve query performance when reports repeatedly calculate the same aggregations.

### Example: Detailed Sales Data

`fact_sales`

| date_key | product_key | store_key | sales_amount |
|---|---:|---:|---:|
| 20261001 | 101 | 10 | 2000 |
| 20261001 | 101 | 10 | 3000 |
| 20261001 | 102 | 10 | 1500 |
| 20261002 | 101 | 10 | 2500 |

### Aggregate Sales Table

`fact_daily_product_sales`

| date_key | product_key | store_key | total_sales |
|---|---:|---:|---:|
| 20261001 | 101 | 10 | 5000 |
| 20261001 | 102 | 10 | 1500 |
| 20261002 | 101 | 10 | 2500 |

The aggregate table stores summarized results instead of every original transaction.

### SQL Example

```sql
CREATE TABLE fact_daily_product_sales AS
SELECT
    date_key,
    product_key,
    store_key,
    SUM(sales_amount) AS total_sales
FROM fact_sales
GROUP BY
    date_key,
    product_key,
    store_key;
```

### Characteristics

- Stores precomputed summaries.
- Reduces the amount of data scanned for suitable queries.
- Can improve reporting performance.
- Requires refresh or incremental maintenance.
- Must preserve the correct grain and aggregation logic.

**Important:** An aggregate fact table is a performance optimization. It does not replace the detailed fact table when transaction-level analysis is required.

---

## 8. Comparison of Fact Table Types

| Fact Table Type | Grain | Data Behavior | Example |
|---|---|---|---|
| Transaction | One row per event or transaction | Inserts new rows | Individual sales |
| Periodic Snapshot | One row per entity per period | Inserts periodic rows | Daily inventory |
| Accumulating Snapshot | One row per process instance | Updates existing row | Order fulfillment |
| Factless Fact | One row per event or relationship | Usually inserts rows | Student attendance |
| Aggregate Fact | One row per summarized group | Rebuilt or incrementally maintained | Daily product sales |

---

## 9. Transaction vs Periodic vs Accumulating Snapshot

These three types are especially important for dimensional modeling interviews.

| Aspect | Transaction | Periodic Snapshot | Accumulating Snapshot |
|---|---|---|---|
| Purpose | Capture individual events | Capture state at intervals | Track process milestones |
| Grain | One row per event | One row per entity per period | One row per process instance |
| Data behavior | Insert | Insert periodically | Update existing row |
| Time representation | Event timestamp/date | Snapshot period | Multiple milestone dates |
| Historical detail | Detailed events | Historical periodic states | Process progress |
| Example | Sales transaction | Daily inventory | Order fulfillment |

### Easy Way to Remember

- **Transaction:** What happened?
- **Periodic Snapshot:** What was the state at a particular time?
- **Accumulating Snapshot:** How far has the process progressed?

---

## 10. Measures and Additivity

Fact tables commonly contain numeric measures. Understanding their additivity is important because not every measure can be summed across every dimension.

### 10.1 Additive Measures

Measures that can be summed across all relevant dimensions.

Examples:
- Sales amount
- Quantity sold
- Units shipped
- Transaction count

### 10.2 Semi-Additive Measures

Measures that can be summed across some dimensions but not across certain time dimensions.

Examples:
- Account balance
- Inventory balance
- Available stock

For example, inventory across warehouses can be summed for a given date, but daily inventory balances should not normally be summed across dates.

### 10.3 Non-Additive Measures

Measures that cannot be meaningfully summed.

Examples:
- Percentage
- Ratio
- Conversion rate
- Average price

For example, the average of two percentages is not necessarily the correct overall percentage. Calculate it using the underlying numerator and denominator when possible.

### Example: Correct Average Price

Incorrect approach:

```sql
SELECT
    AVG(unit_price) AS average_price
FROM fact_sales;
```

This calculates the unweighted average of row-level prices.

If the required metric is average selling price per unit, use:

```sql
SELECT
    SUM(sales_amount) / NULLIF(SUM(quantity), 0)
        AS average_selling_price
FROM fact_sales;
```

This calculates a weighted average based on quantity.

---

## 11. How to Choose the Correct Fact Table Type

Start by understanding the business process and defining the grain.

Ask these questions:

### Question 1: Do you need to record every individual event?

Use a **Transaction Fact Table**.

Example: Every customer purchase.

### Question 2: Do you need to capture the state at regular intervals?

Use a **Periodic Snapshot Fact Table**.

Example: Daily inventory balances.

### Question 3: Do you need to track a process through milestones?

Use an **Accumulating Snapshot Fact Table**.

Example: Loan application stages.

### Question 4: Do you need to record an event without numeric measures?

Use a **Factless Fact Table**.

Example: Student attendance.

### Question 5: Do reports repeatedly query summarized data?

Consider an **Aggregate Fact Table**.

Example: Daily sales by product and store.

---

## 12. Common Interview Mistakes

| Incorrect Understanding | Correct Understanding |
|---|---|
| Every fact table must contain numeric measures | Factless fact tables can contain only keys |
| Transaction facts and periodic snapshots are the same | They have different grains and purposes |
| Accumulating snapshots insert a new row at every milestone | They usually update the existing process row |
| Periodic snapshots must always be daily | They can be daily, weekly, monthly, or another defined interval |
| Inventory balance can be summed across dates | It is generally semi-additive across time |
| Percentages can always be summed or averaged | Ratios require appropriate numerator and denominator logic |
| Aggregate facts replace detailed facts | They complement detailed facts |
| A fact table always has one foreign key | It usually has multiple dimension keys, depending on its grain |

---

## 13. Interview Questions and Answers

### Q1. What are the three main types of fact tables?

Transaction fact tables, periodic snapshot fact tables, and accumulating snapshot fact tables.

### Q2. What is a transaction fact table?

It stores one row per individual business event at the defined grain, such as a sales transaction or payment.

### Q3. What is a periodic snapshot fact table?

It stores the state of a business entity or process at regular reporting intervals, such as daily inventory or monthly account balances.

### Q4. What is an accumulating snapshot fact table?

It tracks the progress of a business process through milestones, usually by updating one row per process instance.

### Q5. What is a factless fact table?

It records events or relationships without numeric measures. Its rows can still be counted to analyze activity or coverage.

### Q6. What is an aggregate fact table?

It stores precomputed summaries of detailed fact data to improve reporting performance.

### Q7. Can a fact table contain no measures?

Yes. A factless fact table contains dimension keys without numeric measures.

### Q8. What is the difference between periodic and accumulating snapshots?

A periodic snapshot adds a row for each reporting period. An accumulating snapshot updates an existing row as a process reaches new milestones.

### Q9. What is the difference between transaction and accumulating snapshot facts?

Transaction facts store individual events as separate rows. Accumulating snapshots store process milestones in one row that is updated over time.

### Q10. What is a semi-additive measure?

A measure that can be summed across some dimensions but not all. Account balance and inventory balance are common examples.

### Q11. Why do we create aggregate fact tables?

To reduce query processing and improve performance for frequently requested summaries. They require proper refresh and consistency management.

### Q12. Can one data warehouse contain multiple fact table types?

Yes. A warehouse can contain transaction, periodic snapshot, accumulating snapshot, factless, and aggregate fact tables for different analytical requirements.

### Q13. How would you model a bank account balance?

A periodic snapshot is suitable when the business needs daily or monthly account balances. A transaction fact table can separately record deposits and withdrawals.

### Q14. How would you model order fulfillment?

An accumulating snapshot can track order, payment, shipment, and delivery milestones. A transaction or event table can preserve every status change.

### Q15. How do you select the correct fact table type?

Understand the business process, define the grain, and determine whether the requirement is to record events, capture periodic states, track milestones, record relationships, or accelerate summary reporting.

---

## 14. Quick Revision

| Type | Remember |
|---|---|
| Transaction Fact | One row per event |
| Periodic Snapshot | One row per entity per period |
| Accumulating Snapshot | One row per process instance, updated over time |
| Factless Fact | Records events or relationships without numeric measures |
| Aggregate Fact | Stores precomputed summaries |

### Three Most Important Types

1. **Transaction:** Records what happened.
2. **Periodic Snapshot:** Records the state at a regular interval.
3. **Accumulating Snapshot:** Records how a process progresses.

### One-Minute Interview Answer

There are three main types of fact tables in dimensional modeling.

A transaction fact table stores individual business events, such as sales or payments, with one row per event at the defined grain.

A periodic snapshot fact table stores the state of a business process or entity at regular intervals, such as daily inventory or monthly account balances.

An accumulating snapshot fact table tracks the progress of a process through milestones. For example, an order fulfillment table can store order, payment, shipping, and delivery dates in one row, which gets updated as the process progresses.

Other useful patterns include factless fact tables, which record events or relationships without numeric measures, and aggregate fact tables, which store precomputed summaries to improve reporting performance.

---

## 15. Self-Assessment

- [ ] Define a fact table and explain its grain.
- [ ] Explain transaction fact tables with an example.
- [ ] Explain periodic snapshot fact tables with an example.
- [ ] Explain accumulating snapshot fact tables with an example.
- [ ] Compare the three main fact table types.
- [ ] Explain factless fact tables.
- [ ] Explain aggregate fact tables.
- [ ] Differentiate additive, semi-additive, and non-additive measures.
- [ ] Explain why inventory balance is semi-additive.
- [ ] Choose the correct fact table for a banking scenario.
- [ ] Choose the correct fact table for an order fulfillment scenario.
- [ ] Explain why a warehouse can use multiple fact table types.

**Interview readiness:** Given a business requirement, identify the business process, define the grain, select the appropriate fact table type, and explain how its data is inserted, updated, or summarized.