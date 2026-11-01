# E-Commerce Data Warehouse Design

## 1. Introduction

An **E-Commerce Data Warehouse** is a centralized analytical system that stores and organizes business data from an online shopping platform.

It helps businesses analyze:
- Total sales and revenue.
- Customer purchasing behavior.
- Product performance.
- Order and payment trends.
- Inventory levels.
- Shipping and delivery performance.
- Returns and refunds.

A Data Engineer designs pipelines to collect data from operational systems, transform it, and make it available for reporting and analytics.

### Example Business Scenario

Consider an e-commerce company that sells electronics, clothing, and household products.

Its business team wants to answer questions such as:

1. What is the monthly revenue?
2. Which products generate the highest revenue?
3. Which customers purchase most frequently?
4. Which cities generate the most sales?
5. What percentage of orders are delivered on time?
6. How many products are returned?
7. What is the average order value?

We need to design a data warehouse that can answer these questions accurately.

**Key idea:** Design the warehouse around business processes, analytical requirements, and clearly defined fact-table grains.

---

## 2. Understand the Source Systems

An e-commerce platform may have multiple operational systems.

Each system generates different types of data.

| Source System | Data Generated |
|---|---|
| Customer System | Customer profiles and addresses |
| Product Catalog | Product names, categories, prices |
| Order Management | Orders and order items |
| Payment System | Payment transactions and payment status |
| Inventory System | Stock levels and warehouse information |
| Shipping System | Shipment and delivery information |
| Returns System | Return requests and refund details |
| Website/App | User activity and browsing events |

### Example Source Tables

**customers**

| customer_id | customer_name | city |
|---|---|---|
| C001 | Rahul | Kolkata |
| C002 | Priya | Delhi |

**products**

| product_id | product_name | category |
|---|---|---|
| P001 | Laptop | Electronics |
| P002 | Mouse | Accessories |

**orders**

| order_id | customer_id | order_date | order_status |
|---|---|---|---|
| O001 | C001 | 2026-10-01 | Delivered |
| O002 | C002 | 2026-10-02 | Shipped |

**order_items**

| order_item_id | order_id | product_id | quantity | unit_price |
|---|---|---|---:|---:|
| I001 | O001 | P001 | 1 | 50,000 |
| I002 | O001 | P002 | 2 | 1,000 |
| I003 | O002 | P001 | 1 | 50,000 |

These are simplified operational tables. A production system may have additional attributes, status history, multiple addresses, and multiple transactions per order.

---

## 3. Gather Business Requirements

Before designing tables, identify the business questions.

### Sales Analytics

- What is the total revenue?
- What is the monthly sales growth?
- Which products generate the most revenue?
- Which categories perform best?

### Customer Analytics

- Who are the highest-value customers?
- How many new customers are acquired each month?
- What is the customer repeat-purchase rate?
- Which cities generate the most revenue?

### Order Analytics

- How many orders are placed?
- How many orders are cancelled?
- What is the average order value?
- How long does order fulfillment take?

### Payment Analytics

- How many payments succeed?
- What is the payment failure rate?
- What is the total refunded amount?

### Inventory Analytics

- Which products are low in stock?
- What is the daily inventory level?
- Which products have the highest inventory turnover?

### Delivery Analytics

- What percentage of orders are delivered on time?
- What is the average delivery time?
- Which delivery regions experience delays?

These requirements determine which business processes, fact tables, dimensions, and measures are needed.

---

## 4. Identify Business Processes

An e-commerce platform contains several distinct business processes.

| Business Process | What It Represents |
|---|---|
| Sales | Products purchased by customers |
| Order Processing | Order lifecycle and status changes |
| Payments | Payment attempts and transactions |
| Inventory | Stock movements and balances |
| Shipping | Shipment and delivery events |
| Returns | Returned products and refund events |
| Website Activity | Customer browsing and interactions |

Each process may require a separate fact table because its events and grain are different.

**Important:** Do not combine unrelated business processes into one fact table just because they share an order ID.

For example, one order can contain multiple products and multiple payment attempts. Joining all these records directly can multiply rows and inflate measures.

---

## 5. Define the Grain of Each Fact Table

Grain defines exactly what one row represents.

It must be decided before selecting fact-table columns.

| Fact Table | Grain |
|---|---|
| `fact_sales` | One row per order item |
| `fact_order` | One row per order |
| `fact_payment` | One row per payment transaction or attempt |
| `fact_inventory_snapshot` | One row per product, warehouse, and snapshot time |
| `fact_shipment` | One row per shipment |
| `fact_return` | One row per returned order item or return line |
| `fact_web_event` | One row per tracked user event |

### Why Separate Grains Matter

Suppose an order contains:

- Two order items.
- Two payment attempts.
- One shipment.

Joining these tables directly on `order_id` can produce multiple combinations of rows.

This can duplicate sales amounts and payment amounts.

**Rule:** Keep fact tables at their natural business-event grain and join them through appropriate dimensions or carefully aggregated datasets.

---

## 6. Design the Dimensional Model

A common design for an e-commerce warehouse is a collection of star schemas.

Different fact tables can share common dimensions. These are called **conformed dimensions**.

### Main Dimension Tables

#### 6.1 dim_customer

Stores descriptive customer information.

| customer_key | customer_id | customer_name | city | state |
|---:|---|---|---|---|
| 101 | C001 | Rahul | Kolkata | West Bengal |
| 102 | C002 | Priya | Delhi | Delhi |

Possible attributes:
- Customer name.
- Email or contact details, subject to privacy requirements.
- City.
- State.
- Country.
- Customer segment.
- Registration date.

Customer address or segment changes may require SCD Type 2 when historical reporting is needed.

#### 6.2 dim_product

Stores product information.

| product_key | product_id | product_name | category | brand |
|---:|---|---|---|---|
| 201 | P001 | Laptop | Electronics | Brand A |
| 202 | P002 | Mouse | Accessories | Brand B |

Possible attributes:
- Product name.
- Category.
- Subcategory.
- Brand.
- Product type.
- Product status.

#### 6.3 dim_date

Stores calendar attributes.

| date_key | full_date | day | month | quarter | year |
|---:|---|---:|---|---|---:|
| 20261001 | 2026-10-01 | 1 | October | Q4 | 2026 |
| 20261002 | 2026-10-02 | 2 | October | Q4 | 2026 |

Possible attributes:
- Day.
- Week.
- Month.
- Quarter.
- Year.
- Weekend flag.
- Holiday flag.

A date dimension allows reporting by different calendar levels.

#### 6.4 dim_location

Stores geographic information.

| location_key | city | state | country |
|---:|---|---|---|
| 301 | Kolkata | West Bengal | India |
| 302 | Delhi | Delhi | India |

#### 6.5 dim_payment_method

Stores payment method descriptions.

| payment_method_key | payment_method |
|---:|---|
| 401 | UPI |
| 402 | Credit Card |
| 403 | Net Banking |

#### 6.6 dim_order_status

Stores standardized order status descriptions.

| order_status_key | order_status |
|---:|---|
| 501 | Placed |
| 502 | Shipped |
| 503 | Delivered |
| 504 | Cancelled |

A status dimension is useful when status values have business descriptions or reporting attributes. Some models may instead store a status code directly in the fact table.

---

## 7. Design the Fact Tables

Fact tables store measurable business events at a defined grain.

### 7.1 fact_sales

**Grain:** One row per order item.

Possible columns:

| Column | Purpose |
|---|---|
| sales_key | Fact-row identifier, if required |
| order_id | Degenerate order identifier |
| order_item_id | Source order-item identifier |
| date_key | Order date dimension key |
| customer_key | Customer dimension key |
| product_key | Product dimension key |
| location_key | Location dimension key |
| quantity | Units purchased |
| unit_price | Price per unit |
| gross_amount | Quantity × unit price |
| discount_amount | Discount |
| net_sales | Sales after defined deductions |

Example:

| order_id | product_key | quantity | gross_amount | discount_amount | net_sales |
|---|---:|---:|---:|---:|---:|
| O001 | 201 | 1 | 50,000 | 0 | 50,000 |
| O001 | 202 | 2 | 2,000 | 200 | 1,800 |
| O002 | 201 | 1 | 50,000 | 1,000 | 49,000 |

Revenue definitions must specify the treatment of tax, shipping, discounts, cancellations, and refunds.

### 7.2 fact_order

**Grain:** One row per order.

Possible columns:
- `order_id`
- `customer_key`
- `order_date_key`
- `order_status_key`
- `order_total`
- `shipping_amount`
- `tax_amount`

This table supports order-level reporting.

Do not sum `order_total` after joining it to multiple order-item rows unless it has first been brought to the correct grain.

### 7.3 fact_payment

**Grain:** One row per payment transaction or attempt.

Possible columns:
- `payment_id`
- `order_id`
- `customer_key`
- `payment_date_key`
- `payment_method_key`
- `payment_amount`
- `payment_status`

Example:

| payment_id | order_id | payment_method_key | payment_amount | payment_status |
|---|---|---:|---:|---|
| PAY001 | O001 | 401 | 52,000 | Success |
| PAY002 | O002 | 402 | 50,000 | Failed |
| PAY003 | O002 | 401 | 50,000 | Success |

A payment attempt is not necessarily a completed payment. Successful transactions, refunds, and reversals need clearly defined business treatment.

### 7.4 fact_inventory_snapshot

**Grain:** One row per product, warehouse, and snapshot time.

Possible columns:
- `date_key`
- `product_key`
- `warehouse_key`
- `available_quantity`
- `reserved_quantity`
- `damaged_quantity`

Inventory balance is generally semi-additive across time. It can be summed across products or warehouses at a common snapshot time, subject to business definitions, but not blindly across dates.

### 7.5 fact_shipment

**Grain:** One row per shipment.

Possible columns:
- `shipment_id`
- `order_id`
- `customer_key`
- `warehouse_key`
- `ship_date_key`
- `delivery_date_key`
- `shipping_cost`
- `delivery_duration`
- `delivery_status`

An order may have multiple shipments, so this table should not automatically be treated as one row per order.

### 7.6 fact_return

**Grain:** One row per return line or returned order item.

Possible columns:
- `return_id`
- `order_item_id`
- `product_key`
- `customer_key`
- `return_date_key`
- `return_quantity`
- `refund_amount`
- `return_reason_key`

Returns should be modeled as their own business events. Do not assume every return corresponds to a full-order refund.

---

## 8. E-Commerce Warehouse Architecture

A practical architecture can use a medallion approach.

```text
                 Source Systems
                       |
       +---------------+----------------+
       |               |                |
   Orders          Customers         Products
   Payments        Inventory         Shipping
       |               |                |
       +---------------+----------------+
                       |
                       v
                 BRONZE LAYER
              Raw Ingested Data
                       |
                       v
                 SILVER LAYER
          Cleaned and Standardized Data
                       |
          +------------+-------------+
          |            |             |
          v            v             v
      Validate      Deduplicate    Standardize
          |            |             |
          +------------+-------------+
                       |
                       v
                  GOLD LAYER
             Dimensional Data Model
                       |
       +---------------+----------------+
       |               |                |
       v               v                v
   fact_sales     fact_payment    fact_inventory
       |               |                |
       +---------------+----------------+
                       |
                       v
                BI / Analytics
             Power BI / Tableau
```

### Bronze Layer

Stores raw or minimally processed source data.

Examples:
- Raw orders.
- Raw customers.
- Raw products.
- Raw payments.

### Silver Layer

Cleans and standardizes the data.

Examples:
- Handle invalid records.
- Standardize timestamps.
- Validate business keys.
- Deduplicate source records.
- Normalize status values.
- Apply data quality rules.

### Gold Layer

Stores business-ready dimensional models and aggregate datasets.

Examples:
- Sales fact.
- Customer dimension.
- Product dimension.
- Payment fact.
- Inventory snapshots.
- Monthly sales aggregates.

**Important:** Bronze, Silver, and Gold are logical data layers. They do not have to be separate physical systems or storage technologies.

---

## 9. Data Ingestion Strategy

Different sources may require different ingestion approaches.

| Source | Possible Ingestion Method |
|---|---|
| Relational order database | Full load, incremental query, or CDC |
| Customer database | Incremental load or CDC |
| Product catalog | Incremental load or CDC |
| Payment events | Event streaming or incremental ingestion |
| Inventory database | CDC or periodic snapshots |
| Website events | Event streaming or file ingestion |

### Full Load

Loads the complete source dataset.

Useful for:
- Initial ingestion.
- Small reference datasets.
- Controlled rebuilds.

### Incremental Load

Loads only new or changed records.

Possible approaches:
- Timestamp-based extraction.
- Increasing ID.
- Change Data Capture (CDC).
- File-based ingestion.

### CDC

Captures inserts, updates, and deletes from source systems when supported by the source and ingestion platform.

CDC is useful for tracking changes that timestamp-based extraction may miss, especially physical deletes.

### Important Design Considerations

- Maintain extraction progress or watermarks.
- Handle late-arriving records.
- Make writes retry-safe.
- Deduplicate incoming records.
- Track source-to-target reconciliation.
- Avoid advancing the watermark before the target write is safely completed.

---

## 10. Handling Slowly Changing Dimensions

Customer and product information can change over time.

For example, a customer may move from Kolkata to Bangalore.

If historical location reporting is required, SCD Type 2 can preserve the old and new versions.

### Example: dim_customer

| customer_key | customer_id | city | effective_from | effective_to | is_current |
|---:|---|---|---|---|---|
| 101 | C001 | Kolkata | 2026-01-01 | 2026-07-01 | FALSE |
| 205 | C001 | Bangalore | 2026-07-01 | NULL | TRUE |

The fact table should reference the customer version valid at the time of the sale.

### Why This Matters

Suppose the customer placed an order in March while living in Kolkata and moved to Bangalore in July.

A March sale should reference the Kolkata dimension version if the business wants historical location reporting.

Using only the current customer record could incorrectly attribute historical sales to Bangalore.

### Other Possible Strategies

- Type 1: Correct a misspelled customer name.
- Type 2: Preserve customer address history.
- Type 0: Retain an immutable original registration attribute.

Different attributes can use different SCD strategies.

---

## 11. Example Analytical Queries

The following queries assume the dimension and fact tables have the columns shown in this design. SQL date functions may vary by database.

### 11.1 Total Revenue

```sql
SELECT
    SUM(net_sales) AS total_revenue
FROM fact_sales;
```

This assumes `net_sales` is defined consistently and the fact table contains only the sales records intended for the metric.

### 11.2 Monthly Revenue

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

### 11.3 Top 5 Products by Revenue

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

### 11.4 Revenue by City

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

### 11.5 Average Order Value

If the sales fact has one row per order item, calculate order revenue first.

```sql
WITH order_totals AS (
    SELECT
        order_id,
        SUM(net_sales) AS order_revenue
    FROM fact_sales
    GROUP BY order_id
)
SELECT
    AVG(order_revenue) AS average_order_value
FROM order_totals;
```

This example assumes the business defines AOV using the included sales records. Cancellations, refunds, taxes, and shipping must be handled according to the agreed definition.

### 11.6 Payment Success Rate

```sql
SELECT
    COUNT(DISTINCT CASE
        WHEN payment_status = 'Success' THEN payment_id
    END) * 100.0
    / NULLIF(COUNT(DISTINCT payment_id), 0) AS payment_success_rate
FROM fact_payment;
```

This measures successful payment attempts divided by all payment attempts. If the business instead wants the percentage of orders with at least one successful payment, the calculation must use distinct orders as the denominator.

### 11.7 On-Time Delivery Rate

Assume `fact_shipment` contains one row per shipment and the business has defined its delivery SLA.

```sql
SELECT
    COUNT(CASE
        WHEN actual_delivery_date <= promised_delivery_date
        THEN 1
    END) * 100.0
    / NULLIF(COUNT(*), 0) AS on_time_delivery_rate
FROM fact_shipment
WHERE delivery_status = 'Delivered';
```

This assumes the table includes the actual and promised delivery dates. The denominator includes delivered shipments only; the business may define a different denominator.

### 11.8 Return Rate

A quantity-based return rate can be calculated as:

```sql
SELECT
    SUM(r.return_quantity) * 100.0
    / NULLIF(SUM(s.quantity), 0) AS return_rate
FROM fact_sales s
JOIN fact_return r
    ON s.order_item_id = r.order_item_id;
```

**Caution:** This direct join can duplicate sales quantities if one order item has multiple return records. A safer approach is to aggregate returns to one row per order item before joining.

```sql
WITH return_totals AS (
    SELECT
        order_item_id,
        SUM(return_quantity) AS returned_quantity
    FROM fact_return
    GROUP BY order_item_id
),
sales_totals AS (
    SELECT
        order_item_id,
        SUM(quantity) AS sold_quantity
    FROM fact_sales
    GROUP BY order_item_id
)
SELECT
    SUM(COALESCE(r.returned_quantity, 0)) * 100.0
    / NULLIF(SUM(s.sold_quantity), 0) AS return_rate
FROM sales_totals s
LEFT JOIN return_totals r
    ON s.order_item_id = r.order_item_id;
```

This is a quantity-based return rate. Other definitions may use returned orders, returned order items, or refund value.

---

## 12. Data Quality Checks

Data quality is critical because incorrect source data can produce incorrect business metrics.

### Important Checks

| Quality Check | Example |
|---|---|
| Completeness | Every order has a valid order date |
| Uniqueness | Order-item identifiers are unique |
| Validity | Quantity is positive |
| Referential integrity | Every fact dimension key resolves correctly |
| Consistency | Order statuses follow standard values |
| Reconciliation | Source and warehouse totals match |
| Freshness | Data is loaded within the expected SLA |

### Example: Check Duplicate Order Items

```sql
SELECT
    order_item_id,
    COUNT(*) AS record_count
FROM fact_sales
GROUP BY order_item_id
HAVING COUNT(*) > 1;
```

### Example: Check Invalid Quantities

```sql
SELECT *
FROM fact_sales
WHERE quantity IS NULL
   OR quantity <= 0;
```

### Example: Check Missing Customer Keys

```sql
SELECT *
FROM fact_sales
WHERE customer_key IS NULL;
```

Whether a missing dimension key is valid depends on the business model. Unknown or not-applicable dimension members may be represented using designated dimension rows.

---

## 13. Performance Optimization

An e-commerce warehouse can contain millions or billions of records.

Performance should be considered during model and pipeline design.

### 13.1 Partitioning

Partition large datasets using appropriate, commonly filtered columns.

For example:
- Order date.
- Sales month.
- Snapshot date.

Avoid high-cardinality partition columns such as customer IDs when they create excessive small partitions.

### 13.2 Columnar Storage

Formats such as Parquet are useful for analytical workloads because they support column-oriented reading and compression.

### 13.3 Predicate Pushdown and Partition Pruning

Filter data as early as practical.

For example, a monthly sales query should avoid scanning unrelated periods when the storage layout and query engine can prune them.

### 13.4 File Management

Small files can increase metadata and scheduling overhead.

Use appropriate file sizing, optimized writes, and compaction features supported by the platform.

### 13.5 Pre-Aggregation

Create daily or monthly sales aggregates for frequently accessed dashboards.

### 13.6 Query Optimization

- Select only required columns.
- Filter early when appropriate.
- Avoid unnecessary shuffles.
- Use suitable join strategies.
- Check execution plans and runtime metrics.
- Avoid caching data that is not reused.

**Important:** Optimization techniques should be measured against actual workload behavior. There is no universal partition count or file size that works for every dataset.

---

## 14. Security and Governance

An e-commerce warehouse may contain personal and financial information.

Security must be included in the design.

Important practices:

- Restrict access based on business roles.
- Avoid exposing unnecessary customer personal information.
- Mask sensitive fields where appropriate.
- Store credentials securely.
- Maintain audit logs.
- Track data lineage.
- Apply data retention policies.
- Follow applicable privacy and regulatory requirements.

For Databricks environments, Unity Catalog can help manage data access, governance, lineage, and discovery.

Access to cloud storage and access to catalog objects may involve separate permission layers.

---

## 15. End-to-End Project Explanation

This is a useful structure for explaining an e-commerce warehouse in a Data Engineer interview.

### Project Objective

Build a centralized data warehouse to analyze e-commerce sales, customers, products, payments, inventory, and deliveries.

### Step 1: Source Ingestion

Collect data from:
- Orders.
- Customers.
- Products.
- Payments.
- Inventory.
- Shipping.
- Returns.

Use full or incremental ingestion based on source capabilities.

### Step 2: Bronze Processing

Store source data in its raw form with ingestion metadata.

### Step 3: Silver Processing

- Validate schemas.
- Clean invalid records.
- Deduplicate data.
- Standardize timestamps and statuses.
- Handle missing values.
- Apply business rules.

### Step 4: Gold Modeling

Build:
- `fact_sales`
- `fact_order`
- `fact_payment`
- `fact_inventory_snapshot`
- `fact_shipment`
- `fact_return`
- Customer, product, date, location, and other required dimensions.

### Step 5: Historical Tracking

Use SCD Type 1 or Type 2 according to attribute-level business requirements.

### Step 6: KPI Calculation

Calculate:
- Revenue.
- Average order value.
- Customer retention.
- Payment success rate.
- Return rate.
- On-time delivery rate.

### Step 7: Data Validation

Perform:
- Duplicate checks.
- Null checks.
- Referential integrity checks.
- Source-to-target reconciliation.
- Aggregate validation.

### Step 8: Reporting

Publish Gold datasets to BI tools for business dashboards.

### Step 9: Monitoring

Monitor:
- Pipeline failures.
- Data freshness.
- Processing duration.
- Data quality.
- Volume anomalies.
- Reconciliation differences.

---

## 16. Common Interview Questions and Answers

### Q1. How would you design an e-commerce data warehouse?

I would first gather business requirements and identify processes such as sales, payments, inventory, shipping, and returns. Then I would define the grain of each fact table, identify dimensions and measures, design the dimensional model, and build ingestion and transformation pipelines. Finally, I would validate the data and publish business-ready datasets.

### Q2. Why do we need multiple fact tables?

Different business processes have different grains and measures. Sales, payments, inventory, and shipping represent different events, so separate fact tables help prevent incorrect joins and double counting.

### Q3. What is the grain of a sales fact table?

A common grain is one row per order item. This allows analysis by product, customer, date, and other relevant dimensions.

### Q4. Why use a star schema?

A star schema provides a clear structure with a central fact table and descriptive dimensions. It is generally easy for analytical queries and BI tools to understand.

### Q5. How would you handle customer address changes?

If historical address reporting is required, I would use SCD Type 2. I would expire the previous dimension version, insert a new version with a new surrogate key, and ensure fact records reference the correct version.

### Q6. How would you calculate average order value?

First, calculate total qualifying revenue for each order. Then calculate the average of those order totals.

### Q7. How would you prevent duplicate sales?

I would define the order-item grain, enforce or validate the business key, deduplicate source records deterministically, and make incremental writes retry-safe.

### Q8. How would you handle late-arriving orders?

I would identify affected business dates, process the late records, update the target using idempotent logic, and recalculate affected aggregates where necessary.

### Q9. How would you handle payment failures?

I would model payment attempts separately from orders, preserve transaction status, and define whether the business KPI measures successful attempts, successful orders, or payment value.

### Q10. How would you design inventory reporting?

I would use a periodic snapshot fact table with a grain such as one row per product, warehouse, and snapshot date. Inventory balance should be treated as semi-additive across time.

### Q11. How would you calculate return rate?

I would first agree on whether return rate means returned units, returned order items, returned orders, or refund value. Then I would aggregate return records at the correct grain and calculate the agreed ratio.

### Q12. How would you optimize the warehouse?

I would use columnar storage, appropriate partitioning or clustering, file compaction, predicate and projection pushdown where supported, suitable joins, and pre-aggregation for common reporting workloads. I would validate improvements using query plans and runtime metrics.

### Q13. What is the difference between fact_sales and fact_payment?

`fact_sales` represents purchased products or order items and their sales measures. `fact_payment` represents payment transactions or attempts and their financial status. One order may have multiple payment attempts.

### Q14. How do you ensure data consistency across fact tables?

I would use conformed dimensions, consistent business definitions, controlled key resolution, source-to-target reconciliation, and appropriate validation rules.

### Q15. What happens if you join sales and payment facts directly?

If both contain multiple rows per order, the join can multiply rows and inflate sales and payment amounts. I would aggregate each fact to a compatible grain or analyze them separately through shared dimensions.

---

## 17. Common Design Mistakes

| Mistake | Correct Approach |
|---|---|
| Putting sales, payments, and shipments into one fact table | Model distinct business processes separately |
| Not defining fact-table grain | Define exactly what one row represents |
| Joining multiple fact tables directly by order ID | Check cardinality and aggregate to compatible grains |
| Counting fact rows as orders | Use distinct order IDs or an order-level fact |
| Overwriting customer history when it is needed | Use SCD Type 2 |
| Summing daily inventory balances across dates | Treat inventory as semi-additive |
| Counting failed payments as successful sales | Define payment and revenue rules clearly |
| Ignoring returns and refunds | Model return events and document revenue treatment |
| Assuming every order has one shipment | Model shipment-level grain |
| Assuming a successful pipeline means accurate data | Validate quality, reconciliation, and business rules |

---

## 18. Quick Revision

### Main Source Systems

- Customers.
- Products.
- Orders.
- Payments.
- Inventory.
- Shipping.
- Returns.
- Website events.

### Main Fact Tables

| Fact Table | Grain |
|---|---|
| `fact_sales` | One row per order item |
| `fact_order` | One row per order |
| `fact_payment` | One row per payment attempt or transaction |
| `fact_inventory_snapshot` | One row per product, warehouse, and snapshot time |
| `fact_shipment` | One row per shipment |
| `fact_return` | One row per return line |

### Main Dimensions

- `dim_customer`
- `dim_product`
- `dim_date`
- `dim_location`
- `dim_payment_method`
- `dim_order_status`

### Architecture

```text
Sources → Bronze → Silver → Gold → BI
```

### Most Important Concepts

1. Gather business requirements before designing tables.
2. Identify business processes separately.
3. Define the grain of every fact table.
4. Use separate fact tables for distinct business events.
5. Use conformed dimensions where appropriate.
6. Apply SCD based on historical reporting requirements.
7. Handle late-arriving records and retries safely.
8. Validate source-to-target reconciliation.
9. Avoid double counting when joining fact tables.
10. Monitor both pipeline execution and data quality.

### One-Minute Interview Answer

For an e-commerce data warehouse, I would first identify the source systems and business requirements, such as sales, customer analytics, payments, inventory, shipping, and returns.

I would define a separate business process and grain for each fact table. For example, the sales fact would have one row per order item, the payment fact one row per payment transaction, and the inventory snapshot one row per product, warehouse, and snapshot date.

I would create shared dimensions such as customer, product, date, and location, and use SCD Type 2 where historical attribute tracking is required.

I would build Bronze, Silver, and Gold processing layers, implement incremental ingestion and data quality checks, and publish business-ready fact and dimension tables for analytics.

Finally, I would validate the results through reconciliation, prevent double counting across different fact grains, and monitor pipeline reliability and data freshness.

---

## 19. Self-Assessment

- [ ] Explain the objective of an e-commerce data warehouse.
- [ ] Identify the source systems.
- [ ] Gather sales and operational business requirements.
- [ ] Identify distinct business processes.
- [ ] Define the grain of each fact table.
- [ ] Design a sales fact table.
- [ ] Design payment and inventory fact tables.
- [ ] Identify conformed dimensions.
- [ ] Explain star schema design.
- [ ] Explain SCD Type 1 and Type 2 use cases.
- [ ] Explain incremental ingestion and CDC.
- [ ] Handle late-arriving data.
- [ ] Calculate revenue and average order value.
- [ ] Calculate payment success and return rates.
- [ ] Identify fact-to-fact join risks.
- [ ] Explain data quality and reconciliation.
- [ ] Explain partitioning and query optimization.
- [ ] Explain security and governance.
- [ ] Present the end-to-end architecture in an interview.

**Interview readiness:** Given an e-commerce business scenario, be able to identify source systems, business processes, fact-table grains, dimensions, measures, ingestion strategy, historical tracking, quality checks, and analytical outputs.