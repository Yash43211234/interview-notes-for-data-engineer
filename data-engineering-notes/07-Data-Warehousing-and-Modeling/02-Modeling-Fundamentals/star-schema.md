# Star Schema in Data Warehousing

## 1. Introduction

A **Star Schema** is a dimensional modeling technique used in data warehouses to organize data for analytical queries and reporting.

It consists of:

- A central **fact table** containing measurable business events and dimension keys.
- Multiple **dimension tables** containing descriptive information.
- Relationships between the fact table and dimension tables.

It is called a star schema because the structure resembles a star, with the fact table in the center and dimension tables around it.

### Simple Example

An e-commerce company wants to analyze sales by customer, product, date, and store.

The star schema may look like this:

<AsyncImage query="star schema data warehouse diagram fact sales customer product date store dimension tables" aspectRatio="16:9" width="100%" />

```text
              dim_customer
                    |
                    |
dim_product --- fact_sales --- dim_date
                    |
                    |
                 dim_store
```

The fact table stores sales measurements, while dimension tables provide the context needed to analyze those measurements.

---

## 2. What Is a Star Schema?

A star schema is a dimensional data model in which dimension tables are generally denormalized and connected directly to a central fact table.

### Main Characteristics

1. One central fact table.
2. Multiple dimension tables.
3. Dimension tables contain descriptive attributes.
4. Fact tables contain measures and foreign keys.
5. Dimension tables generally have fewer joins between themselves.
6. Designed to support analytical queries and reporting.

### Example

Suppose a retail company wants to analyze sales.

**fact_sales**

| order_id | customer_key | product_key | date_key | sales_amount |
|---|---:|---:|---:|---:|
| 101 | 1 | 10 | 20261001 | 5000 |
| 102 | 2 | 11 | 20261002 | 3000 |
| 103 | 1 | 12 | 20261003 | 2000 |

**dim_customer**

| customer_key | customer_name | city | state | country |
|---|---|---|---|---|
| 1 | Rahul | Kolkata | West Bengal | India |
| 2 | Priya | Delhi | Delhi | India |

**dim_product**

| product_key | product_name | category | department |
|---|---|---|---|
| 10 | Laptop | Electronics | Technology |
| 11 | Mouse | Accessories | Technology |
| 12 | Chair | Furniture | Home |

**dim_date**

| date_key | full_date | month | year |
|---:|---|---|---:|
| 20261001 | 2026-10-01 | October | 2026 |
| 20261002 | 2026-10-02 | October | 2026 |
| 20261003 | 2026-10-03 | October | 2026 |

Notice that descriptive attributes such as city, state, and country are stored together in the customer dimension.

This is a typical denormalized dimension design.

---

## 3. Main Components of a Star Schema

### 3.1 Fact Table

A fact table stores measurable business events and references to dimensions.

Example: `fact_sales`

| order_id | customer_key | product_key | date_key | quantity | sales_amount |
|---|---:|---:|---:|---:|---:|
| 101 | 1 | 10 | 20261001 | 1 | 5000 |
| 102 | 2 | 11 | 20261002 | 2 | 3000 |
| 103 | 1 | 12 | 20261003 | 1 | 2000 |

Typical measures include:

- Sales amount.
- Quantity sold.
- Discount amount.
- Cost.
- Profit.

The fact table also contains foreign keys connecting it to dimension tables.

**Important:** Define the grain of the fact table before deciding which facts and dimensions to include.

For example:

> One row in `fact_sales` represents one order line.

This ensures that measures are stored and aggregated at the correct level of detail.

### 3.2 Dimension Tables

Dimension tables provide descriptive information about business entities.

Examples:

| Dimension | Attributes |
|---|---|
| Customer | Name, city, state, country |
| Product | Name, category, brand |
| Date | Day, month, quarter, year |
| Store | Store name, location, region |
| Promotion | Campaign name, discount type |

Dimensions help answer questions such as:

- Which products generated the highest sales?
- Which city has the most customers?
- What were the monthly sales?
- Which product category performed best?

---

## 4. How a Star Schema Works

Consider a sales dashboard.

The business wants to calculate total sales by product category and customer city.

The process is:

1. Read sales measurements from `fact_sales`.
2. Join `dim_product` to retrieve product categories.
3. Join `dim_customer` to retrieve customer cities.
4. Group the sales amount by category and city.
5. Calculate total sales.

### SQL Example

```sql
SELECT
    p.category,
    c.city,
    SUM(f.sales_amount) AS total_sales
FROM fact_sales f
JOIN dim_product p
    ON f.product_key = p.product_key
JOIN dim_customer c
    ON f.customer_key = c.customer_key
GROUP BY
    p.category,
    c.city
ORDER BY total_sales DESC;
```

### Why Is This Useful?

The business can analyze sales from different perspectives without storing all descriptive information directly in the fact table.

The fact table provides measurements, while dimensions provide analytical context.

---

## 5. Types of Fact Tables Used in Star Schemas

A star schema can use different types of fact tables depending on the business process.

### 5.1 Transaction Fact Table

Stores individual business events.

Example: One row per order line.

| order_id | product_key | quantity | sales_amount |
|---|---:|---:|---:|
| 101 | 10 | 1 | 5000 |
| 101 | 11 | 2 | 1000 |
| 102 | 10 | 1 | 5000 |

Common use cases:

- Sales transactions.
- Payment transactions.
- Product purchases.

### 5.2 Periodic Snapshot Fact Table

Stores measurements at regular intervals.

Example: One row per product per warehouse per day.

| date_key | product_key | warehouse_key | stock_quantity |
|---:|---:|---:|---:|
| 20261001 | 10 | 1 | 100 |
| 20261002 | 10 | 1 | 90 |
| 20261001 | 11 | 1 | 50 |

Common use cases:

- Daily inventory.
- Monthly account balances.
- Daily website traffic summaries.

### 5.3 Accumulating Snapshot Fact Table

Tracks a business process through multiple milestones.

Example: One row per order fulfillment process.

| order_id | order_date_key | shipped_date_key | delivered_date_key |
|---|---:|---:|---:|
| 101 | 20261001 | 20261002 | 20261004 |
| 102 | 20261002 | 20261003 | NULL |

Common use cases:

- Order fulfillment.
- Loan processing.
- Recruitment lifecycle.

**Remember:** The fact table type describes how the business process is recorded. Grain describes exactly what one row represents.

---

## 6. Star Schema vs Snowflake Schema

Both are dimensional modeling techniques used in data warehousing.

The primary difference is how dimension tables are organized.

### Star Schema

Dimension attributes are generally stored together in one denormalized dimension table.

```text
              dim_customer
                    |
dim_product --- fact_sales --- dim_date
                    |
                 dim_store
```

### Snowflake Schema

Dimension attributes are normalized into additional related tables.

```text
dim_country
     |
dim_state
     |
dim_city
     |
dim_customer
     |
 fact_sales
```

### Comparison

| Aspect | Star Schema | Snowflake Schema |
|---|---|---|
| Dimension design | Mostly denormalized | More normalized |
| Dimension tables | Usually fewer | Usually more |
| Joins | Fewer | More |
| Query complexity | Usually simpler | Can be more complex |
| Data redundancy | May be higher | Often lower |
| Storage | May be higher | Can be lower |
| BI usability | Generally easier | Can require more modeling knowledge |
| Common usage | Reporting and dashboards | Normalized dimensional hierarchies |

### Which Is Better?

Neither is universally better.

- Choose a star schema when simpler analytical queries and reporting usability are important.
- Choose a snowflake schema when normalized dimension hierarchies and reduced duplication are important.

Actual query performance depends on the database engine, data layout, statistics, and workload.

---

## 7. Advantages of Star Schema

### 7.1 Simple Data Model

The structure is easy to understand because the fact table connects directly to dimensions.

### 7.2 Fewer Joins

Queries can retrieve descriptive attributes without joining multiple normalized hierarchy tables.

### 7.3 Easier Reporting

BI tools and reporting users can work with straightforward relationships.

### 7.4 Supports Analytical Queries

It supports analysis across different dimensions such as customer, product, date, and location.

### 7.5 Easy Aggregation

Fact measures can be grouped by dimension attributes.

Example:

```sql
SELECT
    d.year,
    SUM(f.sales_amount) AS yearly_sales
FROM fact_sales f
JOIN dim_date d
    ON f.date_key = d.date_key
GROUP BY d.year;
```

### 7.6 Suitable for BI Tools

Star schemas are commonly used in analytical models because their relationships are relatively simple to understand and navigate.

---

## 8. Disadvantages of Star Schema

### 8.1 Data Redundancy

Dimension attributes may be repeated across multiple rows.

For example, the country name may appear repeatedly in the customer dimension.

### 8.2 More Storage

Repeated descriptive attributes can increase storage requirements, although compression may reduce the impact.

### 8.3 Dimension Maintenance

When descriptive attributes change, the warehouse must manage those changes correctly.

For historical tracking, Slowly Changing Dimensions (SCDs) may be required.

### 8.4 Large Dimensions

Some dimensions can become very large and require careful handling of history, updates, and query performance.

---

## 9. Star Schema and Slowly Changing Dimensions

A star schema commonly uses Slowly Changing Dimensions (SCDs) to manage changes in descriptive attributes over time.

Example:

A customer moves from Kolkata to Delhi.

The warehouse may need to preserve the historical location for past sales.

### SCD Type 1

Overwrites the old value.

| customer_key | customer_id | city |
|---|---|---|
| 1 | C01 | Delhi |

The previous city is not retained in this dimension record.

### SCD Type 2

Creates a new dimension row to preserve history.

| customer_key | customer_id | city | start_date | end_date | is_current |
|---:|---|---|---|---|---|
| 1 | C01 | Kolkata | 2025-01-01 | 2026-05-31 | false |
| 2 | C01 | Delhi | 2026-06-01 | NULL | true |

Here:

- `customer_id` is the business key.
- `customer_key` is the surrogate key.
- The new customer version receives a new surrogate key.
- Historical fact rows continue referencing the old customer version.

This allows reports to analyze sales using the customer's location at the time of the transaction.

---

## 10. Designing a Star Schema: Step by Step

Suppose an e-commerce company wants to build a sales data warehouse.

### Step 1: Identify the Business Process

Business process: Sales.

### Step 2: Define the Grain

One row per order line.

### Step 3: Identify Dimensions

- Customer.
- Product.
- Date.
- Store.
- Promotion.

### Step 4: Identify Facts

- Quantity.
- Sales amount.
- Discount amount.
- Cost amount.

### Step 5: Define Relationships

```text
                  dim_customer
                       |
                       |
dim_product ------ fact_sales ------ dim_date
                       |
                       |
                    dim_store
                       |
                       |
                  dim_promotion
```

### Step 6: Implement the Model

Create the fact table and dimension tables, define keys and relationships, and load data according to the declared grain.

### Step 7: Validate the Model

Check:

- Fact table grain.
- Dimension key uniqueness.
- Foreign-key relationships.
- Duplicate records.
- Measure aggregation.
- Historical dimension behavior.
- Reconciliation against source data.

---

## 11. Star Schema in a Modern Data Engineering Pipeline

A typical modern architecture may look like this:

```text
Source Systems
      |
      v
   Bronze
      |
      v
   Silver
      |
      v
   Gold
      |
      v
Star Schema
      |
      v
BI / Dashboards
```

### Bronze Layer

- Ingests raw or lightly processed source data.
- Preserves source information.
- Supports traceability and reprocessing.

### Silver Layer

- Cleans and validates records.
- Standardizes business entities.
- Handles duplicates and schema differences.
- Produces reusable datasets.

### Gold Layer

- Builds business-ready analytical tables.
- Creates fact and dimension tables.
- Implements star schemas where appropriate.
- Supports dashboards and reporting.

### Example: Databricks Implementation

A Data Engineer may:

1. Ingest orders, customers, and products into Bronze Delta tables.
2. Clean and standardize data in Silver.
3. Create `dim_customer`, `dim_product`, and `dim_date`.
4. Build `fact_sales` using the appropriate grain.
5. Store Gold tables in Delta format.
6. Connect Power BI or another BI tool for reporting.

**Important:** Delta Lake is a table format and storage framework. A star schema is a logical data model. They solve different problems and can be used together.

---

## 12. Common Interview Mistakes

| Incorrect Understanding | Correct Understanding |
|---|---|
| Star schema is a single flat table | It contains a fact table and multiple dimension tables |
| Fact tables contain only measures | They also contain dimension keys and may contain other identifiers |
| Dimensions contain only IDs | They contain descriptive attributes |
| Star schema has no joins | It still joins facts to dimensions |
| Star schema is always faster | Performance depends on the workload and platform |
| Every dimension must be denormalized completely | Dimension design depends on analytical requirements |
| Star schema and snowflake schema are the same | They differ in dimension normalization |
| All facts can be summed | Some measures are semi-additive or non-additive |
| A fact table can have any grain | A fact table should have a clearly declared and consistent grain |

---

## 13. Interview Questions and Answers

### Q1. What is a star schema?

A star schema is a dimensional data model with a central fact table connected directly to multiple dimension tables. It is commonly used for analytical queries and reporting.

### Q2. Why is it called a star schema?

It is called a star schema because its structure resembles a star, with the fact table in the center and dimension tables around it.

### Q3. What are the main components of a star schema?

The main components are a fact table and dimension tables. The fact table stores measures and dimension keys, while dimension tables store descriptive attributes.

### Q4. What is the difference between a fact table and a dimension table?

A fact table stores measurable business events, such as sales amount and quantity. A dimension table stores descriptive context, such as customer name, product category, and date attributes.

### Q5. Why is a star schema commonly used in data warehouses?

It provides a simple structure for analytical queries, reduces the number of joins compared with normalized dimension hierarchies, and is convenient for reporting and BI tools.

### Q6. What is the difference between a star schema and a snowflake schema?

A star schema generally uses denormalized dimensions, while a snowflake schema normalizes dimensions into multiple related tables.

### Q7. What is the grain of a fact table?

Grain defines exactly what one row represents. For example, one row per order line or one row per product per warehouse per day.

### Q8. Can a star schema have multiple fact tables?

Yes. A warehouse can have multiple fact tables for different business processes, such as sales, payments, and inventory. They can share conformed dimensions.

### Q9. What is a conformed dimension?

A conformed dimension is a dimension shared consistently across multiple fact tables or business processes.

For example, the same date dimension may be used by sales, payments, and inventory facts.

### Q10. What are the different types of fact tables?

The common types are:

- Transaction fact table.
- Periodic snapshot fact table.
- Accumulating snapshot fact table.

### Q11. What is the role of surrogate keys in a star schema?

Surrogate keys are warehouse-generated identifiers used to uniquely identify dimension records. They are especially useful for managing historical dimension versions, such as SCD Type 2.

### Q12. How do you handle changes in customer information?

Depending on the requirement, use an SCD strategy. Type 1 overwrites the previous value, while Type 2 creates a new dimension record to preserve history.

### Q13. Can a star schema be implemented using Delta Lake?

Yes. Fact and dimension tables can be stored as Delta tables. Delta Lake provides transactional table capabilities, while the star schema defines the logical relationships and analytical structure.

### Q14. How would you design a star schema for an e-commerce company?

I would identify the sales business process, define the grain as one row per order line, identify dimensions such as customer, product, date, and store, and define measures such as quantity and sales amount. Then I would build the fact and dimension tables and validate their keys and relationships.

### Q15. Why should grain be defined before creating a fact table?

Grain determines what each row represents, which measures belong in the table, and how those measures can be aggregated without double counting.

---

## 14. Quick Revision

| Topic | Key Point |
|---|---|
| Star Schema | Dimensional model with a central fact table |
| Fact Table | Stores measurements and dimension keys |
| Dimension Table | Stores descriptive attributes |
| Grain | Defines what one fact row represents |
| Surrogate Key | Warehouse-generated dimension identifier |
| Transaction Fact | One row per business event |
| Periodic Snapshot | One row per entity per period |
| Accumulating Snapshot | One row per process instance |
| SCD Type 1 | Overwrites old attribute value |
| SCD Type 2 | Creates a new row to preserve history |
| Conformed Dimension | Shared, consistently defined dimension |
| Main Advantage | Simple analytical queries and reporting |
| Main Trade-off | Potentially more dimension redundancy |

### One-Minute Interview Answer

A star schema is a dimensional modeling technique used in data warehouses. It consists of a central fact table connected directly to multiple dimension tables.

For example, in an e-commerce sales warehouse, the fact table stores sales amount and quantity, while dimensions provide customer, product, date, and store information.

The main advantage is that it provides a simple structure for analytical queries and reporting, with fewer joins than a snowflake schema in many cases.

Before designing it, I identify the business process, define the grain, select dimensions, and then identify the facts.

In a modern data engineering pipeline, star schemas are commonly built in the Gold layer and can be stored as Delta tables in Databricks.

---

## 15. Self-Assessment

- [ ] Define a star schema.
- [ ] Explain why it is called a star schema.
- [ ] Identify fact and dimension tables.
- [ ] Explain the difference between facts and dimensions.
- [ ] Define the grain of a sales fact table.
- [ ] Design a star schema for an e-commerce business.
- [ ] Write SQL queries joining fact and dimension tables.
- [ ] Compare star schema and snowflake schema.
- [ ] Explain transaction, periodic snapshot, and accumulating snapshot fact tables.
- [ ] Explain surrogate keys and conformed dimensions.
- [ ] Describe SCD Type 1 and Type 2.
- [ ] Explain how star schemas fit into a Databricks medallion architecture.
- [ ] Identify common grain and double-counting mistakes.

**Interview readiness:** Be able to draw a sales star schema, explain the grain of its fact table, identify its dimensions and measures, and describe how you would build it in the Gold layer of a data engineering pipeline.