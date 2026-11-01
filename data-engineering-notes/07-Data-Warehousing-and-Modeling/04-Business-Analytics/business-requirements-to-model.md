# Translating Business Requirements into a Data Model

## 1. Introduction

In data engineering and data warehousing, a data model should be designed around the business requirements it needs to support.

A business may ask questions such as:

- What is our total monthly revenue?
- Which products generate the highest sales?
- Which customers purchase most frequently?
- How many orders are delivered late?
- How has customer revenue changed over time?

A Data Engineer or Data Modeler must translate these questions into a structure that stores data correctly and supports reliable analysis.

This process is called **translating business requirements into a data model**.

The main objective is to understand what the business wants to measure, identify the required data, define the grain, and design fact and dimension tables.

**Key idea:** Understand the business process and analytical questions before designing tables.

---

## 2. Why Is This Important?

A technically correct database can still produce incorrect business reports if the data model does not represent the business requirements properly.

For example, suppose a business wants to calculate monthly sales revenue.

A poorly designed model might:
- Count duplicate orders.
- Include canceled orders.
- Double-count order amounts after joining tables.
- Use the wrong date column.
- Mix order-level and order-item-level data.

These problems can produce incorrect business results even when the SQL query executes successfully.

A good model helps ensure:

1. Business metrics are clearly defined.
2. Data is stored at the correct level of detail.
3. Fact and dimension tables represent the business process.
4. Analytical queries produce accurate results.
5. Historical reporting works as expected.

---

## 3. Understand the Business Requirement

The first step is to understand what the business actually wants to achieve.

Consider an e-commerce company.

The business wants to analyze sales performance.

It asks:

- What is the total revenue each month?
- Which products generate the most revenue?
- Which cities have the highest sales?
- How many orders are completed?
- What is the average order value?

Before creating tables, clarify the meaning of each metric.

### Example: Revenue Requirement

Business statement:

"Show monthly revenue by product and city."

This requirement contains several pieces of information:

| Requirement | Meaning |
|---|---|
| Monthly | Analysis by month |
| Revenue | Measure to calculate |
| Product | Product dimension |
| City | Location dimension |
| Sales | Business process |

From this, we can identify the likely fact table, dimensions, measures, and date attributes.

### Questions to Ask Business Stakeholders

- What business process are we analyzing?
- What exactly does each metric mean?
- Which records should be included or excluded?
- What level of detail is required?
- Which dimensions are needed for filtering and grouping?
- Which date determines the reporting period?
- Is historical analysis required?
- How frequently should the data be refreshed?

**Important:** Do not assume that a business term such as revenue, active customer, or completed order has an obvious definition. Confirm its business meaning.

---

## 4. Identify the Business Process

A business process is an activity or operation that generates measurable business events.

Examples:

| Business | Business Process |
|---|---|
| E-commerce | Sales and order processing |
| Banking | Transactions and payments |
| Retail | Store sales |
| Logistics | Shipment and delivery |
| Healthcare | Patient visits and treatments |
| Telecom | Calls and network usage |

For our example, the business process is **sales**.

The sales process generates events such as:
- An order is placed.
- An order item is purchased.
- A payment is completed.
- An order is shipped.
- An order is delivered.

These events may require different fact tables because they represent different processes and grains.

### Important Distinction

An order and a payment are not necessarily the same business event.

One order may have multiple payment attempts or transactions. Combining them without understanding their relationship can cause duplicated sales amounts.

---

## 5. Identify Business Metrics and Measures

A measure is a numeric value that the business wants to analyze.

For a sales model, possible measures include:

| Measure | Meaning |
|---|---|
| Sales amount | Value of a sale |
| Quantity sold | Number of units sold |
| Discount amount | Discount applied |
| Tax amount | Tax charged |
| Net sales | Sales after defined deductions |
| Order count | Number of qualifying orders |

### Example: Revenue Definition

Suppose the business defines net revenue as:

\[
\text{Net Revenue} = \text{Gross Sales} - \text{Discount}
\]

For one order item:

| Product | Gross Sales | Discount | Net Revenue |
|---|---:|---:|---:|
| Laptop | 60,000 | 5,000 | 55,000 |
| Mouse | 1,000 | 100 | 900 |

Total net revenue:

\[
55,000 + 900 = 55,900
\]

The formula must reflect the business's actual definition. Some organizations include or exclude tax, shipping, refunds, and returns differently.

### Important Measure Types

| Type | Meaning | Example |
|---|---|---|
| Additive | Can be summed across all relevant dimensions | Sales amount |
| Semi-additive | Can be summed across some dimensions, but not all | Account balance over time |
| Non-additive | Cannot be meaningfully summed directly | Percentage, ratio |

For example, an average order value should generally be calculated from its components:

\[
\text{Average Order Value} =
\frac{\text{Total Revenue}}{\text{Number of Orders}}
\]

Do not simply sum or average precomputed averages without considering their denominators.

---

## 6. Identify Dimensions

Dimensions provide descriptive context for analyzing measures.

For the requirement "monthly revenue by product and city," likely dimensions include:

| Dimension | Example Attributes |
|---|---|
| Date | Date, month, quarter, year |
| Product | Product name, category, brand |
| Customer | Customer name, segment |
| Location | City, state, country |
| Order | Order number, order status |

### Example Dimension Tables

**dim_product**

| product_key | product_id | product_name | category |
|---:|---|---|---|
| 101 | P001 | Laptop | Electronics |
| 102 | P002 | Mouse | Accessories |

**dim_customer**

| customer_key | customer_id | customer_name | city |
|---:|---|---|---|
| 201 | C001 | Rahul | Kolkata |
| 202 | C002 | Priya | Delhi |

**dim_date**

| date_key | full_date | month | year |
|---:|---|---|---|
| 20261001 | 2026-10-01 | October | 2026 |
| 20261002 | 2026-10-02 | October | 2026 |

Dimensions answer questions such as:

- Which product?
- Which customer?
- Which city?
- When did the sale happen?

---

## 7. Define the Grain

**Grain defines exactly what one row in a fact table represents.**

This is one of the most important decisions in dimensional modeling.

For example, the sales fact table may have one row per order or one row per order item.

These are different grains.

### Option 1: Order-Level Grain

One row represents one order.

| order_id | customer_key | order_total |
|---|---:|---:|
| O001 | 201 | 56,000 |
| O002 | 202 | 20,000 |

### Option 2: Order-Item-Level Grain

One row represents one product item within an order.

| order_id | product_key | quantity | sales_amount |
|---|---:|---:|---:|
| O001 | 101 | 1 | 55,000 |
| O001 | 102 | 1 | 1,000 |
| O002 | 101 | 1 | 20,000 |

The second model supports product-level analysis more naturally.

### Why Grain Matters

If the business wants to analyze revenue by product, order-item-level grain is generally more appropriate.

If we store only order-level totals, we may lose the detail needed to calculate product-level revenue.

### How to Define Grain

Write one precise sentence before designing the fact table.

Example:

"One row in the sales fact table represents one product line within one customer order."

This statement helps determine:
- Which source records are needed.
- Which keys identify a row.
- Which measures belong in the fact table.
- Which dimensions can be joined.
- Which aggregations are valid.

**Rule:** Define the grain before selecting fact-table columns.

---

## 8. Identify Fact Tables

A fact table stores measurable business events at a defined grain.

For the e-commerce sales process, we can create a sales fact table.

### Example: fact_sales

| order_id | date_key | customer_key | product_key | quantity | sales_amount |
|---|---:|---:|---:|---:|---:|
| O001 | 20261001 | 201 | 101 | 1 | 55,000 |
| O001 | 20261001 | 201 | 102 | 1 | 900 |
| O002 | 20261002 | 202 | 101 | 1 | 20,000 |

The fact table contains:
- Dimension keys.
- Numeric measures.
- Degenerate identifiers when useful, such as `order_id`.

### Fact Table Design Questions

- What business event does each row represent?
- What is the grain?
- Which measures are required?
- Which dimensions describe the event?
- Can the measures be summed?
- Are there multiple events or updates for the same business entity?

---

## 9. Identify the Appropriate Fact Table Type

Different business processes require different fact table designs.

| Fact Type | Purpose | Example |
|---|---|---|
| Transaction fact | Records individual business events | Each order item |
| Periodic snapshot fact | Records state at regular intervals | Daily inventory balance |
| Accumulating snapshot fact | Tracks progress through a process | Order fulfillment lifecycle |
| Factless fact | Records an event or relationship without numeric measures | Student attendance |

### Example: Order Fulfillment

If the business wants to measure:
- Order placement date.
- Payment date.
- Shipping date.
- Delivery date.

An accumulating snapshot fact table may be appropriate because it tracks milestones in one process lifecycle.

The choice depends on the business process and required analysis.

---

## 10. Design the Dimensional Model

After identifying the business process, metrics, dimensions, and grain, design the tables.

A common analytical design is a star schema.

### Example: E-Commerce Sales Star Schema

<escape>
                 dim_date
                    |
                    |
dim_customer -- fact_sales -- dim_product
                    |
                    |
               dim_location
</escape>

### Fact Table

`fact_sales`

Measures:
- Quantity
- Gross sales
- Discount
- Net sales

Foreign keys:
- `date_key`
- `customer_key`
- `product_key`
- `location_key`

### Dimension Tables

- `dim_date`
- `dim_customer`
- `dim_product`
- `dim_location`

The fact table is at the center, and descriptive dimensions surround it.

The model supports queries such as:
- Monthly revenue by product.
- Sales by customer city.
- Quantity sold by product category.
- Revenue by year and month.

---

## 11. Translate Requirements into SQL Queries

A data model should support the actual business questions.

### Requirement 1: Monthly Revenue

```sql
SELECT
    d.year,
    d.month,
    SUM(f.net_sales) AS monthly_revenue
FROM fact_sales f
JOIN dim_date d
    ON f.date_key = d.date_key
GROUP BY
    d.year,
    d.month
ORDER BY
    d.year,
    d.month;
```

### Requirement 2: Top 5 Products by Revenue

```sql
SELECT
    p.product_name,
    SUM(f.net_sales) AS total_revenue
FROM fact_sales f
JOIN dim_product p
    ON f.product_key = p.product_key
GROUP BY
    p.product_name
ORDER BY
    total_revenue DESC
LIMIT 5;
```

### Requirement 3: Revenue by City

```sql
SELECT
    l.city,
    SUM(f.net_sales) AS total_revenue
FROM fact_sales f
JOIN dim_location l
    ON f.location_key = l.location_key
GROUP BY
    l.city
ORDER BY
    total_revenue DESC;
```

These examples assume the fact table's grain and measures support the requested aggregations. The business must also define which order statuses, refunds, cancellations, and taxes are included.

---

## 12. Validate the Data Model Against Business Requirements

After designing the model, validate whether it can answer the original business questions correctly.

### Validation Checklist

| Check | Question |
|---|---|
| Business process | Does the model represent the correct process? |
| Grain | Does each fact row represent the intended event? |
| Measures | Are calculations defined correctly? |
| Dimensions | Are all required analytical perspectives available? |
| Keys | Are joins based on correct keys? |
| History | Is historical tracking handled appropriately? |
| Data quality | Are duplicates, missing values, and invalid records handled? |
| Performance | Can the model support expected query workloads? |
| Reconciliation | Do totals match trusted source or business control totals? |

### Example

If the source system reports total net sales of ₹10,00,000 for a period, the warehouse should reconcile to the same amount after accounting for documented exclusions, transformations, and timing differences.

A successful pipeline run alone does not prove that the model is correct.

---

## 13. Common Problems When Translating Requirements

### 13.1 Ambiguous Business Definitions

The business says "revenue," but does not clarify whether it includes tax, discounts, or refunds.

**Solution:** Agree on a documented metric definition before implementation.

### 13.2 Incorrect Grain

The model stores one row per order, but the business needs product-level revenue.

**Solution:** Define the grain based on the lowest level of detail required by the analysis.

### 13.3 Double Counting

Joining an order-level table to multiple order-item or payment rows can multiply amounts.

**Solution:** Understand table grain and relationship cardinality before joining. Aggregate or model separate business processes appropriately.

### 13.4 Missing Dimensions

The business wants regional analysis, but the model does not contain location information.

**Solution:** Identify all required analytical dimensions during requirements gathering.

### 13.5 Incorrect Historical Reporting

A customer changes city, and the warehouse overwrites the old value even though historical city reporting is required.

**Solution:** Use an appropriate SCD strategy, such as Type 2.

### 13.6 Incorrect Metric Aggregation

A percentage or average is summed like a normal measure.

**Solution:** Store or derive the correct numerator and denominator and calculate the metric at query time.

### 13.7 Ignoring Late-Arriving Data

An event arrives after its reporting period has closed.

**Solution:** Define late-data handling, backfill, reconciliation, and historical key-resolution policies.

---

## 14. End-to-End Example

Consider an e-commerce business that wants a sales analytics dashboard.

### Business Requirements

The dashboard must show:
1. Monthly net revenue.
2. Revenue by product category.
3. Revenue by customer city.
4. Top-selling products.
5. Order fulfillment duration.

### Step 1: Identify Business Processes

- Sales.
- Order fulfillment.

These are related but distinct processes.

### Step 2: Identify Metrics

For sales:
- Quantity sold.
- Gross sales.
- Discount.
- Net sales.

For fulfillment:
- Time from order placement to delivery.
- Number of delivered orders.
- Number of delayed orders.

### Step 3: Identify Dimensions

- Date.
- Customer.
- Product.
- Location.
- Order status.

### Step 4: Define Grain

Sales fact:

"One row represents one product line within one order."

Fulfillment fact:

"One row represents one order and tracks its fulfillment milestones."

### Step 5: Select Fact Tables

- `fact_sales`
- `fact_order_fulfillment`

### Step 6: Select Dimensions

- `dim_date`
- `dim_customer`
- `dim_product`
- `dim_location`
- `dim_order_status`, if required by the reporting design

### Step 7: Define Historical Strategy

- Customer address: Type 2 if historical location reporting is required.
- Product description correction: Type 1 if only the corrected value matters.
- Order fulfillment milestones: Accumulating snapshot fact.

### Step 8: Validate the Model

Test whether the model can answer all five dashboard questions without incorrect joins, double counting, or missing historical context.

---

## 15. Common Interview Questions and Answers

### Q1. How do you convert business requirements into a data model?

First, understand the business process and analytical questions. Then identify metrics, dimensions, and grain. Select the appropriate fact table type, design relationships and keys, and validate the model against the business requirements.

### Q2. Why should grain be defined before designing a fact table?

Grain determines what one fact row represents. It controls which measures and dimensions belong in the table and helps prevent duplication and incorrect aggregation.

### Q3. What is the difference between a business metric and a dimension?

A metric is a value the business measures, such as revenue. A dimension provides context for analyzing that value, such as product, customer, or date.

### Q4. How do you identify fact and dimension tables?

Facts represent measurable business events at a defined grain. Dimensions contain descriptive attributes used to filter, group, and explain those events.

### Q5. What happens if the grain is incorrect?

The model may lose required detail, duplicate measures, produce incorrect aggregations, or fail to answer business questions.

### Q6. How do you prevent double counting?

Understand the grain and cardinality of each table, use correct join keys, avoid joining incompatible fact grains directly, and aggregate at the appropriate level.

### Q7. How do you handle historical attribute changes?

Choose an SCD strategy based on business requirements. Type 1 overwrites values, while Type 2 creates new historical dimension versions.

### Q8. What is the difference between a business process and grain?

A business process identifies the activity being modeled. Grain defines exactly what one row in the fact table represents for that activity.

### Q9. How do you validate a dimensional model?

Check the grain, keys, measures, dimensions, relationships, historical behavior, data quality, and reconciliation against trusted source totals.

### Q10. What would you do if a business requirement is unclear?

Ask stakeholders to clarify metric definitions, filters, time periods, expected level of detail, and historical requirements. Document the agreed definitions before building the model.

### Q11. Can one business process have multiple fact tables?

Yes. Different analytical requirements or grains may require separate fact tables. For example, sales transactions and order fulfillment milestones represent different processes.

### Q12. Why is understanding business requirements important for a Data Engineer?

Because the Data Engineer must build pipelines and models that produce reliable, meaningful data for business decisions, not simply move data between systems.

---

## 16. Quick Revision

| Step | What to Identify |
|---|---|
| 1 | Business goals and analytical questions |
| 2 | Business processes |
| 3 | Metrics and measures |
| 4 | Dimensions |
| 5 | Grain |
| 6 | Fact table type |
| 7 | Keys and relationships |
| 8 | Historical tracking requirements |
| 9 | Data quality and validation rules |
| 10 | Reporting and performance requirements |

### Most Important Concepts

- Business requirements define what the model must support.
- Business process identifies the activity being analyzed.
- Metrics define what the business measures.
- Dimensions provide analytical context.
- Grain defines what one fact row represents.
- Fact tables store measurable events.
- Dimension tables store descriptive attributes.
- SCD handles changes in dimension attributes.
- Reconciliation checks whether the model produces correct results.

### One-Minute Interview Answer

When translating business requirements into a data model, I first understand the business process and the analytical questions the stakeholders want to answer.

Then I identify the required metrics and dimensions and define the grain of the fact table.

Based on the business process, I choose the appropriate fact table type, such as transaction, periodic snapshot, or accumulating snapshot.

Next, I design the fact and dimension tables, define keys and relationships, and decide how historical changes should be handled using SCD techniques.

Finally, I validate the model using business rules, data quality checks, reconciliation, and sample analytical queries to ensure it answers the original business requirements correctly.

---

## 17. Self-Assessment

- [ ] Explain why business requirements come before data modeling.
- [ ] Identify a business process from a business scenario.
- [ ] Differentiate metrics and dimensions.
- [ ] Define additive, semi-additive, and non-additive measures.
- [ ] Explain why grain must be defined first.
- [ ] Design a fact table for a sales process.
- [ ] Identify appropriate dimensions.
- [ ] Choose a suitable fact table type.
- [ ] Explain how SCD supports historical reporting.
- [ ] Identify possible double-counting problems.
- [ ] Translate business questions into SQL queries.
- [ ] Explain how to validate a data model.
- [ ] Explain how to handle unclear business definitions.

**Interview readiness:** Given a business scenario, explain the business process, metrics, dimensions, grain, fact table design, historical requirements, and validation approach before writing SQL or building the pipeline.