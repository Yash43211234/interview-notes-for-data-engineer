# Facts and Dimensions in Data Warehousing

## 1. Introduction

Facts and dimensions are the two fundamental building blocks of **dimensional modeling** in a data warehouse.

They help organize business data so that analytical queries, reporting, and business intelligence become easier.

- **Fact:** Stores measurable business events, such as sales amount, quantity, revenue, and profit.
- **Dimension:** Stores descriptive information that provides context for analyzing facts, such as customer, product, date, and location.

### Simple Example

Consider an e-commerce company.

A customer purchases two products:

| Order ID | Customer | Product | Quantity | Amount |
|---|---|---|---:|---:|
| 101 | Rahul | Laptop | 1 | 50000 |
| 102 | Priya | Mobile | 2 | 30000 |

Here:

- Quantity and Amount are facts or measures.
- Customer, Product, and Order Date provide descriptive context.

**Key idea:** Facts tell us *what happened and how much*. Dimensions tell us *who, what, when, where, and how*.

---

## 2. What Is a Fact Table?

A **Fact Table** stores measurable business events and the foreign keys that connect those events to dimension tables.

It is generally the central table in a dimensional model.

### Characteristics

- Stores numeric measures.
- Contains foreign keys referencing dimensions.
- Often contains a large number of rows.
- Represents business events or observations.
- Supports aggregations such as SUM, COUNT, and AVG.
- Its grain defines what one row represents.

### Example: Fact Sales Table

| sales_id | customer_key | product_key | date_key | quantity | sales_amount |
|---|---:|---:|---:|---:|---:|
| 1 | 101 | 201 | 20261001 | 1 | 50000 |
| 2 | 102 | 202 | 20261002 | 2 | 30000 |
| 3 | 101 | 203 | 20261003 | 1 | 15000 |

Here:

- `customer_key` references the customer dimension.
- `product_key` references the product dimension.
- `date_key` references the date dimension.
- `quantity` and `sales_amount` are measures.

### Example Query

Calculate total sales:

```sql
SELECT
    SUM(sales_amount) AS total_sales
FROM fact_sales;
```

Calculate total quantity sold:

```sql
SELECT
    SUM(quantity) AS total_quantity
FROM fact_sales;
```

---

## 3. What Is a Dimension Table?

A **Dimension Table** stores descriptive attributes that provide context for analyzing business events.

Dimensions help users filter, group, and understand facts.

### Characteristics

- Stores descriptive information.
- Contains a primary or surrogate key.
- Usually has fewer rows than a large fact table.
- Contains attributes used for filtering and grouping.
- Often includes hierarchical information.
- Can maintain historical changes using Slowly Changing Dimensions.

### Example: Customer Dimension

| customer_key | customer_id | customer_name | city | state |
|---|---|---|---|---|
| 101 | C01 | Rahul | Kolkata | West Bengal |
| 102 | C02 | Priya | Delhi | Delhi |
| 103 | C03 | Amit | Mumbai | Maharashtra |

### Example: Product Dimension

| product_key | product_id | product_name | category |
|---|---|---|---|
| 201 | P01 | Laptop | Electronics |
| 202 | P02 | Mobile | Electronics |
| 203 | P03 | Chair | Furniture |

### Example Query

Find sales by customer city:

```sql
SELECT
    c.city,
    SUM(f.sales_amount) AS total_sales
FROM fact_sales f
JOIN dim_customer c
    ON f.customer_key = c.customer_key
GROUP BY c.city;
```

The fact table provides the sales measure, while the dimension provides the city attribute.

---

## 4. Fact Table vs Dimension Table

| Feature | Fact Table | Dimension Table |
|---|---|---|
| Purpose | Stores business events and measurements | Stores descriptive context |
| Data | Measures and dimension keys | Descriptive attributes |
| Rows | Usually many | Usually fewer |
| Columns | Often fewer descriptive columns | Often more descriptive columns |
| Keys | Foreign keys to dimensions; may have other identifiers | Primary or surrogate key |
| Usage | Aggregation and calculations | Filtering, grouping, and labeling |
| Example | Fact Sales | Dim Customer |
| Common operations | SUM, COUNT, AVG | JOIN, GROUP BY, FILTER |

**Remember:**

- Fact tables answer questions about measurements.
- Dimension tables help explain those measurements.

---

## 5. Understanding Grain

**Grain** defines exactly what one row in a fact table represents.

It is one of the most important decisions in dimensional modeling.

### Example

Suppose an order contains three products.

| order_id | product_id | quantity | amount |
|---|---|---:|---:|
| 101 | P01 | 1 | 500 |
| 101 | P02 | 2 | 1000 |
| 101 | P03 | 1 | 300 |

If the grain is **one row per order line**, the fact table contains three rows.

If the grain is **one row per order**, the table may contain one row with the total order amount.

### Common Fact Table Grains

- One row per order.
- One row per order line.
- One row per payment transaction.
- One row per product per day.
- One row per account per month.

### Why Is Grain Important?

Grain determines:

1. What each row represents.
2. Which dimensions can be associated with the fact.
3. Which measures can be stored correctly.
4. How data should be aggregated.
5. How duplicate records should be identified.

**Interview tip:** Define the grain before selecting fact table measures and dimension keys. Never mix different grains in the same fact table without a clear design.

---

## 6. Types of Fact Tables

There are three commonly discussed fact table types:

1. Transaction Fact Table.
2. Periodic Snapshot Fact Table.
3. Accumulating Snapshot Fact Table.

### 6.1 Transaction Fact Table

A transaction fact table stores one row for each individual business transaction or event at the defined grain.

Example: Each order line generates one row.

| transaction_id | product_key | date_key | quantity | amount |
|---|---:|---:|---:|---:|
| T01 | 201 | 20261001 | 1 | 500 |
| T02 | 202 | 20261001 | 2 | 1000 |
| T03 | 203 | 20261002 | 1 | 300 |

**Use cases:**

- Sales transactions.
- Payment events.
- Website clicks.
- Bank transactions.

### 6.2 Periodic Snapshot Fact Table

A periodic snapshot fact table stores measurements at regular intervals, such as daily, weekly, or monthly.

Example: Daily inventory snapshot.

| product_key | date_key | available_stock |
|---|---:|---:|
| 201 | 20261001 | 100 |
| 201 | 20261002 | 90 |
| 201 | 20261003 | 85 |
| 202 | 20261001 | 50 |

**Use cases:**

- Daily inventory balances.
- Monthly account balances.
- Weekly employee headcount.
- Daily website traffic summaries.

The table can contain a row even when no individual transaction occurred during the period.

### 6.3 Accumulating Snapshot Fact Table

An accumulating snapshot fact table tracks a business process with multiple milestones.

A row is created when the process begins and updated as milestones are reached.

Example: Order fulfillment.

| order_id | order_date | shipped_date | delivered_date | order_amount |
|---|---|---|---|---:|
| 101 | Oct 1 | Oct 2 | Oct 4 | 1500 |
| 102 | Oct 2 | Oct 3 | NULL | 2500 |

Order 102 has not yet been delivered.

**Use cases:**

- Order processing.
- Loan approval.
- Insurance claims.
- Recruitment processes.

### Comparison

| Type | What It Stores | Example |
|---|---|---|
| Transaction | Individual events | Each order line |
| Periodic Snapshot | State at regular intervals | Daily inventory |
| Accumulating Snapshot | Progress through a process | Order fulfillment |

---

## 7. Types of Facts and Measures

Measures are the numeric values stored in fact tables.

They are commonly classified into three types.

### 7.1 Additive Facts

Additive measures can be summed across all relevant dimensions.

Examples:

- Sales amount.
- Quantity sold.
- Total cost.
- Transaction count.

Example:

```sql
SELECT
    SUM(sales_amount) AS total_sales
FROM fact_sales;
```

Sales amount can generally be aggregated across customers, products, and dates, provided the measure's business definition supports that aggregation.

### 7.2 Semi-Additive Facts

Semi-additive measures can be summed across some dimensions but not across all dimensions.

Examples:

- Account balance.
- Inventory balance.
- Available stock.

For example, daily inventory balances should not normally be summed across dates to calculate a meaningful inventory balance.

A more appropriate query might select the latest snapshot:

```sql
SELECT
    product_key,
    SUM(available_stock) AS latest_stock
FROM fact_inventory
WHERE date_key = 20261003
GROUP BY product_key;
```

This assumes the selected date represents the intended snapshot for every product.

### 7.3 Non-Additive Facts

Non-additive measures cannot be meaningfully summed across dimensions.

Examples:

- Percentage.
- Profit margin.
- Conversion rate.
- Unit price.

For example, averaging individual profit margins may produce an incorrect overall margin.

A better approach is often to calculate the ratio from its additive components:

```sql
SELECT
    SUM(profit) / NULLIF(SUM(revenue), 0)
        AS overall_profit_margin
FROM fact_sales;
```

### Comparison

| Measure Type | Can Be Summed? | Example |
|---|---|---|
| Additive | Across all relevant dimensions | Sales amount |
| Semi-additive | Across some dimensions | Account balance |
| Non-additive | Not directly | Percentage |

**Important:** Whether a measure is additive depends on its business meaning and the dimensions across which it is aggregated.

---

## 8. Types of Dimension Tables

### 8.1 Conformed Dimension

A conformed dimension is a shared dimension used consistently across multiple fact tables or business processes.

Example:

```text
           dim_date
           /      \
          /        \
 fact_sales      fact_returns
```

Both fact tables use the same date dimension and consistent date definitions.

**Benefits:**

- Consistent reporting.
- Easier cross-process analysis.
- Shared business definitions.

### 8.2 Role-Playing Dimension

A role-playing dimension is the same dimension used in multiple roles within a fact table.

Example: An order may have different dates.

```text
             dim_date
                |
       +--------+--------+
       |        |        |
    Order     Ship    Delivery
     Date     Date      Date
```

The fact table can contain:

- `order_date_key`
- `ship_date_key`
- `delivery_date_key`

All three keys reference the same date dimension but represent different roles.

### 8.3 Junk Dimension

A junk dimension combines multiple low-cardinality flags and indicators into one dimension.

Example:

- Is gift wrapped?
- Is express delivery?
- Is promotional order?

Instead of creating separate dimensions for each small flag, they can be combined into a junk dimension.

### 8.4 Degenerate Dimension

A degenerate dimension is a business identifier stored directly in the fact table without a separate dimension table.

Examples:

- Order number.
- Invoice number.
- Transaction number.

Example:

| order_number | customer_key | sales_amount |
|---|---:|---:|
| ORD101 | 101 | 1500 |
| ORD102 | 102 | 2500 |

The order number provides a way to identify a transaction but does not require a separate descriptive dimension table.

### 8.5 Slowly Changing Dimension

A Slowly Changing Dimension (SCD) manages changes in dimension attributes over time.

Example:

A customer moves from Kolkata to Delhi.

The warehouse must decide whether to overwrite the old city or preserve the historical value.

Common approaches:

- Type 0: Retain the original value.
- Type 1: Overwrite the old value.
- Type 2: Create a new historical row.
- Type 3: Store a limited previous value in another column.

SCDs are covered in more detail in the dedicated Slowly Changing Dimensions note.

---

## 9. Surrogate Keys vs Natural Keys

### Natural Key

A natural key is an identifier that comes from the source business system.

Examples:

- `customer_id`
- `product_id`
- `order_id`

### Surrogate Key

A surrogate key is an artificial identifier generated for the warehouse.

Example:

| customer_key | customer_id | customer_name |
|---:|---|---|
| 101 | C01 | Rahul |
| 102 | C02 | Priya |
| 103 | C03 | Amit |

Here:

- `customer_key` is the surrogate key.
- `customer_id` is the natural or business key.

### Why Use Surrogate Keys?

- Separates warehouse identity from source-system identifiers.
- Supports historical versions of dimensions.
- Helps integrate records from different source systems.
- Provides consistent warehouse keys.

**Important:** A surrogate key does not automatically solve duplicate or incorrect source records. Business-key matching and data quality are still required.

---

## 10. How Facts and Dimensions Work Together

Facts and dimensions are connected through primary and foreign key relationships.

### Example Star Schema

```text
                 dim_customer
                      |
                      |
dim_product ---- fact_sales ---- dim_date
                      |
                      |
                  dim_store
```

### Fact Table

`fact_sales`

| sales_id | customer_key | product_key | date_key | sales_amount |
|---|---:|---:|---:|---:|
| 1 | 101 | 201 | 20261001 | 50000 |
| 2 | 102 | 202 | 20261002 | 30000 |
| 3 | 101 | 203 | 20261003 | 15000 |

### Dimension Tables

`dim_customer`

| customer_key | customer_name | city |
|---:|---|---|
| 101 | Rahul | Kolkata |
| 102 | Priya | Delhi |

`dim_product`

| product_key | product_name | category |
|---:|---|---|
| 201 | Laptop | Electronics |
| 202 | Mobile | Electronics |
| 203 | Chair | Furniture |

`dim_date`

| date_key | full_date | month | year |
|---:|---|---|---:|
| 20261001 | 2026-10-01 | October | 2026 |
| 20261002 | 2026-10-02 | October | 2026 |
| 20261003 | 2026-10-03 | October | 2026 |

### Query: Revenue by Product Category

```sql
SELECT
    p.category,
    SUM(f.sales_amount) AS total_revenue
FROM fact_sales f
JOIN dim_product p
    ON f.product_key = p.product_key
GROUP BY p.category
ORDER BY total_revenue DESC;
```

### Query: Revenue by Customer City

```sql
SELECT
    c.city,
    SUM(f.sales_amount) AS total_revenue
FROM fact_sales f
JOIN dim_customer c
    ON f.customer_key = c.customer_key
GROUP BY c.city
ORDER BY total_revenue DESC;
```

### Query: Revenue by Month

```sql
SELECT
    d.year,
    d.month,
    SUM(f.sales_amount) AS total_revenue
FROM fact_sales f
JOIN dim_date d
    ON f.date_key = d.date_key
GROUP BY
    d.year,
    d.month
ORDER BY
    d.year,
    MIN(d.date_key);
```

The fact table provides the measurements, while the dimensions provide the attributes used to group and filter those measurements.

---

## 11. Common Mistakes in Fact and Dimension Modeling

| Mistake | Correct Approach |
|---|---|
| Defining measures before defining grain | Define the fact table grain first |
| Treating every numeric column as a fact | Determine whether it is a measure, key, or descriptive attribute |
| Storing descriptive attributes unnecessarily in fact tables | Keep descriptive context in dimensions where appropriate |
| Summing account balances across dates | Treat balances as semi-additive |
| Summing percentages | Recalculate ratios from their components |
| Using source IDs as the only dimension keys | Consider surrogate keys for warehouse identity and history |
| Creating a separate dimension for every flag | Consider a junk dimension for related low-cardinality flags |
| Creating a dimension for every transaction number | Consider a degenerate dimension |
| Mixing transaction and snapshot grains | Keep fact table grain consistent |
| Ignoring historical dimension changes | Select an appropriate SCD strategy |

---

## 12. Interview Questions and Answers

### Q1. What is a fact table?

A fact table stores measurable business events and foreign keys that connect those events to dimension tables. Examples include sales amount, quantity, and transaction count.

### Q2. What is a dimension table?

A dimension table stores descriptive attributes such as customer name, product category, city, and date. It provides context for analyzing facts.

### Q3. What is the difference between facts and dimensions?

Facts represent measurable business events, while dimensions describe the context in which those events occur.

### Q4. What is grain in dimensional modeling?

Grain defines exactly what one row in a fact table represents, such as one order, one order line, or one product per day.

### Q5. Why should grain be defined first?

Grain determines the meaning of each row, which dimensions can be associated with it, and how measures can be aggregated correctly.

### Q6. What are the three main types of fact tables?

- Transaction fact table.
- Periodic snapshot fact table.
- Accumulating snapshot fact table.

### Q7. What is an additive fact?

An additive fact can be summed across all relevant dimensions. Sales amount is a common example.

### Q8. What is a semi-additive fact?

A semi-additive fact can be summed across some dimensions but not all. Account balance is a common example because balances should not normally be summed across time.

### Q9. What is a non-additive fact?

A non-additive measure cannot be meaningfully summed directly. Percentages and ratios are common examples.

### Q10. What is a surrogate key?

A surrogate key is a warehouse-generated identifier used to uniquely identify a dimension record independently of the source business key.

### Q11. What is a conformed dimension?

A conformed dimension is shared across multiple fact tables or business processes with consistent definitions and attributes.

### Q12. What is a role-playing dimension?

A role-playing dimension is one dimension used for multiple purposes. For example, the same date dimension can represent order date, shipping date, and delivery date.

### Q13. What is a junk dimension?

A junk dimension combines related low-cardinality flags and indicators into one dimension table.

### Q14. What is a degenerate dimension?

A degenerate dimension is a business identifier, such as an order number, stored directly in the fact table without a separate dimension table.

### Q15. Can a fact table contain text columns?

Yes. Fact tables can contain identifiers and sometimes descriptive or status attributes. However, descriptive attributes are generally modeled in dimensions when they are useful for analysis.

### Q16. Can a fact table have no measures?

Yes. A factless fact table records the occurrence of events or relationships without numeric measures. For example, it can record student attendance or promotional eligibility.

### Q17. What is a factless fact table?

A factless fact table contains dimension keys but no numeric measures. Its rows represent events, coverage, or relationships that can be counted or analyzed.

### Q18. Why are dimension tables commonly smaller than fact tables?

Dimension tables typically contain one row per business entity or historical version, while fact tables contain many rows representing individual events or snapshots.

---

## 13. Quick Revision

| Concept | Remember |
|---|---|
| Fact Table | Stores measurable business events |
| Dimension Table | Stores descriptive context |
| Grain | Defines what one fact row represents |
| Transaction Fact | One row per business event at its grain |
| Periodic Snapshot | Measurements at regular intervals |
| Accumulating Snapshot | Tracks process milestones |
| Additive Fact | Can be summed across relevant dimensions |
| Semi-Additive Fact | Can be summed across some dimensions |
| Non-Additive Fact | Cannot be summed directly |
| Conformed Dimension | Shared, consistent dimension |
| Role-Playing Dimension | Same dimension used in multiple roles |
| Junk Dimension | Combines low-cardinality flags |
| Degenerate Dimension | Business identifier stored in fact table |
| Surrogate Key | Warehouse-generated identifier |
| Natural Key | Source or business identifier |
| Factless Fact | Records events or relationships without measures |

### One-Minute Interview Answer

Facts and dimensions are the main components of dimensional modeling.

A fact table stores measurable business events, such as sales amount, quantity, and transaction count, along with keys that reference dimensions. A dimension table stores descriptive information such as customer, product, date, and location.

For example, in an e-commerce warehouse, a sales fact table can store customer keys, product keys, date keys, quantity, and sales amount. Customer, product, and date dimension tables provide the descriptive context.

There are three common fact table types: transaction, periodic snapshot, and accumulating snapshot. Measures can be additive, semi-additive, or non-additive depending on how they can be aggregated.

One important design principle is to define the grain of a fact table before choosing its measures and dimensions.

---

## 14. Self-Assessment

- [ ] Define fact tables and dimension tables.
- [ ] Explain the difference between facts and dimensions.
- [ ] Understand fact table grain.
- [ ] Identify transaction fact tables.
- [ ] Identify periodic snapshot fact tables.
- [ ] Identify accumulating snapshot fact tables.
- [ ] Explain additive, semi-additive, and non-additive facts.
- [ ] Understand conformed dimensions.
- [ ] Understand role-playing dimensions.
- [ ] Understand junk dimensions.
- [ ] Understand degenerate dimensions.
- [ ] Compare surrogate keys and natural keys.
- [ ] Understand factless fact tables.
- [ ] Write SQL queries joining facts and dimensions.
- [ ] Design a simple e-commerce star schema.

**Interview readiness:** Given an e-commerce business scenario, identify the fact table, its grain, the measures, and the required dimensions. Explain why each column belongs in a fact or dimension table and how the tables connect through keys.