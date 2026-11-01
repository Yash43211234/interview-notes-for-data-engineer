# Data Warehouse Design Case Studies – Interview Preparation

## 1. How to Approach a Data Warehouse Design Interview

A data warehouse design interview tests whether you can convert business requirements into a reliable analytical data model.

Do not start by immediately creating tables. First understand the business process and what users need to analyze.

### Recommended design approach

1. **Understand the business:** What does the organization do?
2. **Identify business processes:** Sales, orders, payments, inventory, returns, etc.
3. **Identify analytical requirements:** What questions should the warehouse answer?
4. **Define the grain:** What does one row in each fact table represent?
5. **Identify dimensions:** Customer, product, date, location, and others.
6. **Identify measures:** Revenue, quantity, discount, balance, and counts.
7. **Define relationships and keys:** Business keys, surrogate keys, and foreign keys.
8. **Handle historical changes:** Decide whether SCD Type 1 or Type 2 is needed.
9. **Design ingestion and transformations:** Source-to-target processing.
10. **Add quality and performance controls:** Validation, partitioning, clustering, and monitoring.

### Important interview principle

**Always define the grain before designing the fact table.**

For example:

- One row per order.
- One row per order item.
- One row per customer per day.
- One row per product per warehouse per day.

These represent different grains and should not be mixed without a clear reason.

---

# Case Study 1: E-Commerce Data Warehouse

## 2. Business Scenario

An e-commerce company sells products through its website and mobile application.

The company wants to analyze:

- Total sales and revenue.
- Sales by product and category.
- Sales by customer and location.
- Daily and monthly order trends.
- Discounts and returns.
- Payment performance.
- Customer purchasing behavior.

Your task is to design a data warehouse.

## 2.1 Identify Business Processes

Separate the business into processes:

| Business Process | What It Represents |
|---|---|
| Sales | Products sold through orders |
| Payments | Payment attempts, captures, and refunds |
| Returns | Products returned by customers |
| Shipping | Order shipment and delivery |
| Customer activity | Customer interactions and behavior |

These processes may require separate fact tables because their grains are different.

## 2.2 Identify Source Systems

Possible sources:

- Customer database.
- Product catalog.
- Order management system.
- Payment gateway.
- Inventory system.
- Shipping application.
- Website or mobile event stream.

Each source may have different schemas, update frequencies, and identifiers.

## 2.3 Define the Sales Fact Grain

For the primary sales fact:

**One row represents one product line in one order.**

Example:

An order contains three products. It creates three rows in the sales fact table.

This allows analysis by product, order, customer, date, and other dimensions.

## 2.4 Design the Star Schema

<AsyncImage query="ecommerce sales star schema fact sales dimension customer product date store data warehouse diagram" aspectRatio="16:9" width="100%" />

**FactSales**

| Column | Purpose |
|---|---|
| sales_key | Surrogate row identifier, if required |
| order_id | Business order identifier |
| order_item_id | Business order-line identifier |
| date_key | Date dimension reference |
| customer_key | Customer dimension reference |
| product_key | Product dimension reference |
| shipping_location_key | Location dimension reference |
| quantity | Units sold |
| unit_price | Selling price per unit |
| gross_amount | Amount before discounts |
| discount_amount | Discount amount |
| net_sales_amount | Sales amount after discount |

**DimCustomer**

| Column | Purpose |
|---|---|
| customer_key | Surrogate key |
| customer_id | Source business key |
| customer_name | Customer name |
| city | City |
| state | State |
| country | Country |
| effective_from | Version start |
| effective_to | Version end |
| is_current | Current version indicator |

**DimProduct**

| Column | Purpose |
|---|---|
| product_key | Surrogate key |
| product_id | Source product identifier |
| product_name | Product name |
| category | Product category |
| brand | Brand |
| effective_from | Version start |
| effective_to | Version end |
| is_current | Current version indicator |

**DimDate**

| Column | Purpose |
|---|---|
| date_key | Date identifier |
| full_date | Actual date |
| day | Day number |
| month | Month |
| quarter | Quarter |
| year | Year |
| weekday_name | Weekday |

**DimLocation**

| Column | Purpose |
|---|---|
| location_key | Surrogate key |
| city | City |
| state | State |
| country | Country |
| postal_code | Postal code |

## 2.5 Additional Fact Tables

Do not force every business process into `FactSales`.

**FactPayment**

Grain: One row per payment transaction or payment attempt, depending on the source and reporting requirement.

Possible measures:
- Payment amount.
- Captured amount.
- Refunded amount.

Possible dimensions:
- Customer.
- Date.
- Payment method.
- Order.

**FactReturns**

Grain: One row per returned order item or return event, depending on the business process.

Possible measures:
- Returned quantity.
- Refund amount.
- Return processing cost.

**FactShipment**

Grain: One row per shipment or shipment event, depending on the required analysis.

Possible measures:
- Shipping cost.
- Delivery duration.
- Number of shipment events.

The exact grain must be documented before implementation.

## 2.6 Handle Slowly Changing Dimensions

Suppose a customer changes their city from Kolkata to Bengaluru.

If historical reporting must preserve the customer's city at the time of each sale, use SCD Type 2.

| customer_key | customer_id | city | effective_from | effective_to | is_current |
|---|---|---|---|---|---|
| 101 | C1001 | Kolkata | 2024-01-01 | 2025-06-30 | false |
| 205 | C1001 | Bengaluru | 2025-07-01 | 9999-12-31 | true |

When loading a sale, use the customer dimension version valid at the sale's event time.

Use Type 1 instead when historical values do not need to be retained.

## 2.7 Data Warehouse Pipeline

```text
Customer DB ───────┐
Product DB ────────┤
Order DB ──────────┤
Payment System ────┼──> Ingestion
Shipping System ───┤       |
Website Events ────┘       v
                         Bronze
                           |
                           v
                     Validation
                           |
                           v
                         Silver
                           |
                    Business Rules
                           |
                           v
                          Gold
                           |
                 Fact and Dimension Tables
                           |
                           v
                    BI / Reporting
```

**Bronze**
- Preserve source records.
- Retain ingestion metadata.
- Support replay and auditing.

**Silver**
- Standardize schemas.
- Clean invalid records.
- Deduplicate according to business rules.
- Resolve keys and timestamps.

**Gold**
- Build fact and dimension tables.
- Apply business definitions.
- Publish analytical datasets.

## 2.8 Example Business Queries

**Monthly sales by product category**

```sql
SELECT
    d.year,
    d.month,
    p.category,
    SUM(f.net_sales_amount) AS total_sales
FROM fact_sales f
JOIN dim_date d
    ON f.date_key = d.date_key
JOIN dim_product p
    ON f.product_key = p.product_key
GROUP BY
    d.year,
    d.month,
    p.category
ORDER BY
    d.year,
    d.month,
    p.category;
```

**Top 10 customers by net sales**

```sql
SELECT
    c.customer_id,
    c.customer_name,
    SUM(f.net_sales_amount) AS total_sales
FROM fact_sales f
JOIN dim_customer c
    ON f.customer_key = c.customer_key
GROUP BY
    c.customer_id,
    c.customer_name
ORDER BY total_sales DESC
LIMIT 10;
```

`LIMIT` syntax is database-dependent. Also, ensure the fact table uses the correct customer dimension version for the sale date.

## 2.9 Important Design Challenges

- Orders may contain multiple product lines.
- Payment attempts may occur multiple times for one order.
- A payment authorization is not necessarily a successful capture.
- Returns and refunds may happen after the original sale.
- Customer and product attributes can change.
- Events can arrive late or out of order.
- Duplicate source events must be handled.
- Revenue definitions must account for discounts, cancellations, and refunds.
- Currency conversion rules must be documented.

**Interview takeaway:** Separate sales, payments, returns, and shipments when their business events and grains differ.

---

# Case Study 2: Banking Data Warehouse

## 3. Business Scenario

A bank wants a centralized warehouse to analyze:

- Customer accounts.
- Deposits and withdrawals.
- Account balances.
- Loan performance.
- Interest income.
- Branch performance.
- Customer activity.

## 3.1 Identify Business Processes

| Process | Example |
|---|---|
| Transactions | Deposits and withdrawals |
| Account balances | Daily account balances |
| Loans | Loan disbursements and repayments |
| Interest | Interest charges and earnings |
| Customer activity | Customer account relationships |

## 3.2 Define Fact Table Grains

**FactTransaction**

One row per posted account transaction.

**FactAccountSnapshot**

One row per account per snapshot date.

**FactLoanPayment**

One row per loan repayment transaction.

Do not mix transaction events and periodic account balances in one fact table.

## 3.3 Design the Model

<AsyncImage query="banking data warehouse star schema fact transaction account balance customer branch date diagram" aspectRatio="16:9" width="100%" />

**FactTransaction**

| Column | Purpose |
|---|---|
| transaction_key | Surrogate identifier |
| transaction_id | Business transaction ID |
| account_key | Account dimension reference |
| customer_key | Customer dimension reference |
| branch_key | Branch dimension reference |
| date_key | Transaction date |
| transaction_type_key | Transaction type reference |
| transaction_amount | Transaction amount |

**FactAccountSnapshot**

| Column | Purpose |
|---|---|
| account_key | Account reference |
| date_key | Snapshot date |
| opening_balance | Opening balance |
| closing_balance | Closing balance |
| available_balance | Available balance |

**Dimensions**
- DimCustomer.
- DimAccount.
- DimBranch.
- DimDate.
- DimTransactionType.
- DimLoan.

## 3.4 Important Modeling Decisions

**1. Account balance is semi-additive.**

You can aggregate balances across accounts for a specific date, but adding daily balances across multiple dates is generally meaningless.

**2. Account ownership may be many-to-many.**

A joint account may have multiple customers. A customer may hold multiple accounts.

A bridge table can represent this relationship.

**3. Transaction reversals need clear treatment.**

A reversed transaction should not simply disappear from the audit trail. Model reversal events or reversal references according to the banking system's rules.

**4. Historical customer details may be required.**

Use SCD Type 2 when reports must reflect customer attributes as they were at a historical point in time.

## 3.5 Example Query: Daily Branch Transaction Amount

```sql
SELECT
    d.full_date,
    b.branch_name,
    SUM(f.transaction_amount) AS total_transaction_amount
FROM fact_transaction f
JOIN dim_date d
    ON f.date_key = d.date_key
JOIN dim_branch b
    ON f.branch_key = b.branch_key
GROUP BY
    d.full_date,
    b.branch_name
ORDER BY
    d.full_date,
    b.branch_name;
```

For financial reporting, distinguish deposits, withdrawals, transfers, reversals, and other transaction types. A signed amount or separate debit/credit measures may be more appropriate than blindly summing all positive amounts.

---

# Case Study 3: Retail Sales and Inventory Warehouse

## 4. Business Scenario

A retail organization operates multiple stores and warehouses.

It wants to analyze:
- Store sales.
- Product performance.
- Inventory levels.
- Stock movements.
- Replenishment requirements.
- Supplier performance.

## 4.1 Identify Business Processes

1. Retail sales.
2. Inventory snapshots.
3. Inventory movements.
4. Purchase orders and receipts.
5. Product returns.

## 4.2 Define Fact Table Grains

| Fact Table | Grain |
|---|---|
| FactSales | One row per sold product line |
| FactInventorySnapshot | One row per product, warehouse, and snapshot date |
| FactInventoryMovement | One row per inventory movement |
| FactPurchaseReceipt | One row per received purchase-order line |

## 4.3 Model Structure

<AsyncImage query="retail inventory data warehouse dimensional model fact sales inventory snapshot product store supplier date" aspectRatio="16:9" width="100%" />

**FactInventorySnapshot**

| Column | Purpose |
|---|---|
| date_key | Snapshot date |
| product_key | Product reference |
| warehouse_key | Warehouse reference |
| opening_quantity | Opening stock |
| received_quantity | Received units |
| sold_quantity | Sold units |
| closing_quantity | Closing stock |

**FactInventoryMovement**

| Column | Purpose |
|---|---|
| movement_id | Movement identifier |
| date_key | Movement date |
| product_key | Product reference |
| warehouse_key | Warehouse reference |
| movement_type | Receipt, transfer, adjustment, etc. |
| quantity_change | Signed quantity change |

**Dimensions**
- DimProduct.
- DimStore.
- DimWarehouse.
- DimSupplier.
- DimDate.

## 4.4 Important Modeling Decisions

- Inventory snapshots are periodic snapshots.
- Inventory movements are transaction facts.
- Inventory balance is semi-additive over time.
- Transfers between warehouses should be modeled consistently to avoid double counting.
- Supplier performance may require purchase-order and receipt facts.
- Product category changes may require historical tracking.

## 4.5 Example Query: Current Inventory by Warehouse

```sql
WITH latest_snapshot AS (
    SELECT
        product_key,
        warehouse_key,
        closing_quantity,
        ROW_NUMBER() OVER (
            PARTITION BY product_key, warehouse_key
            ORDER BY date_key DESC
        ) AS rn
    FROM fact_inventory_snapshot
)
SELECT
    warehouse_key,
    SUM(closing_quantity) AS total_inventory
FROM latest_snapshot
WHERE rn = 1
GROUP BY warehouse_key;
```

This assumes one valid snapshot per product, warehouse, and date. If multiple snapshots can exist for the same date, define a deterministic tie-breaker.

---

# Case Study 4: Healthcare Data Warehouse

## 5. Business Scenario

A healthcare organization wants to analyze:

- Patient visits.
- Doctor performance.
- Hospital admissions.
- Treatments.
- Medical procedures.
- Billing and payments.
- Bed utilization.

## 5.1 Identify Business Processes

| Process | Example |
|---|---|
| Patient visits | Outpatient appointments |
| Admissions | Inpatient stays |
| Procedures | Medical procedures |
| Billing | Charges and payments |
| Bed utilization | Occupancy snapshots |

## 5.2 Define Fact Grains

| Fact Table | Grain |
|---|---|
| FactPatientVisit | One row per patient visit |
| FactProcedure | One row per performed procedure |
| FactAdmission | One row per admission |
| FactBilling | One row per billing line |
| FactBedSnapshot | One row per bed per snapshot time |

## 5.3 Dimensions

- DimPatient.
- DimDoctor.
- DimHospital.
- DimDepartment.
- DimDate.
- DimDiagnosis.
- DimProcedure.

## 5.4 Important Modeling Decisions

- A patient can have many visits.
- A visit can include multiple procedures.
- A patient can have multiple diagnoses.
- Diagnosis relationships may require a bridge table.
- Admissions and visits represent different business processes.
- Patient information requires strict privacy and access controls.
- Historical patient attributes may need SCD Type 2, depending on reporting requirements.

## 5.5 Important Data Quality Checks

- Patient identifiers must be valid.
- Visit timestamps must be valid.
- Admission and discharge dates must follow business rules.
- Procedure codes must be recognized.
- Billing amounts must reconcile with billing source systems.
- Duplicate visits must be identified using business rules, not simply identical timestamps.

---

# Case Study 5: Subscription and SaaS Analytics Warehouse

## 6. Business Scenario

A SaaS company wants to understand:

- New subscriptions.
- Renewals.
- Cancellations.
- Monthly recurring revenue.
- Customer churn.
- Subscription plans.
- Customer lifetime value.

## 6.1 Identify Business Processes

- Subscription lifecycle.
- Billing and invoices.
- Payments.
- Product usage.
- Customer acquisition.
- Cancellations and renewals.

## 6.2 Define Fact Grains

| Fact Table | Grain |
|---|---|
| FactSubscriptionEvent | One row per subscription lifecycle event |
| FactInvoiceLine | One row per invoice line |
| FactPayment | One row per payment transaction |
| FactDailySubscriptionSnapshot | One row per subscription per day |
| FactUsageEvent | One row per usage event |

## 6.3 Dimensions

- DimCustomer.
- DimSubscriptionPlan.
- DimDate.
- DimProduct.
- DimRegion.
- DimAcquisitionChannel.

## 6.4 Important Modeling Decisions

**1. Subscription events and snapshots are different.**

An event records a change, such as activation or cancellation.

A daily snapshot records the subscription's state on a particular day.

**2. Revenue and MRR are not identical.**

Monthly recurring revenue is a normalized recurring-revenue metric. It is not automatically equal to invoice cash collected during the month.

**3. Churn needs a precise definition.**

Clarify whether churn means:
- Customer churn.
- Subscription churn.
- Revenue churn.
- Gross churn.
- Net revenue churn.

**4. Refunds and failed payments need separate treatment.**

A failed payment does not necessarily mean a subscription has been cancelled.

## 6.5 Example Query: Active Subscriptions by Plan

```sql
SELECT
    p.plan_name,
    COUNT(*) AS active_subscriptions
FROM fact_daily_subscription_snapshot f
JOIN dim_subscription_plan p
    ON f.plan_key = p.plan_key
JOIN dim_date d
    ON f.date_key = d.date_key
WHERE d.full_date = DATE '2026-10-31'
  AND f.subscription_status = 'ACTIVE'
GROUP BY p.plan_name;
```

The query assumes the snapshot table has one row per subscription per date and stores subscription status at that grain.

---

# Case Study 6: Logistics and Delivery Warehouse

## 7. Business Scenario

A logistics company wants to analyze:
- Shipment volume.
- Delivery performance.
- Delivery delays.
- Transportation costs.
- Carrier performance.
- Warehouse operations.

## 7.1 Identify Business Processes

- Shipment creation.
- Pickup.
- Warehouse processing.
- Transportation.
- Delivery.
- Failed delivery and return.

## 7.2 Define Fact Grains

| Fact Table | Grain |
|---|---|
| FactShipment | One row per shipment |
| FactShipmentEvent | One row per shipment event |
| FactDeliveryAttempt | One row per delivery attempt |
| FactTransportCost | One row per shipment cost item |

## 7.3 Dimensions

- DimCustomer.
- DimCarrier.
- DimWarehouse.
- DimLocation.
- DimDate.
- DimShipmentStatus.

## 7.4 Important Modeling Decisions

- A shipment may have many tracking events.
- A shipment may have multiple delivery attempts.
- A shipment may be split into multiple packages.
- Delivery time should use clearly defined start and end events.
- Failed delivery attempts should remain available for operational analysis.
- Carrier changes and shipment rerouting may need historical tracking.

**Interview takeaway:** A shipment-level fact cannot represent every tracking event without changing its grain. Use separate shipment and shipment-event facts when both analyses are required.

---

# 8. Cross-Case Study Design Concepts

## 8.1 Transaction Fact vs Periodic Snapshot Fact

| Transaction Fact | Periodic Snapshot Fact |
|---|---|
| Records individual events | Records state at regular intervals |
| Usually event-driven | Usually time-driven |
| Example: Sales transaction | Example: Daily inventory |
| Rows added when events occur | Rows added for each snapshot period |

## 8.2 Fact Table vs Dimension Table

| Fact Table | Dimension Table |
|---|---|
| Stores measurements and events | Stores descriptive attributes |
| Defines a specific grain | Describes business entities |
| Often large | Often smaller |
| Used for aggregation | Used for filtering and grouping |

## 8.3 When Should You Use SCD Type 1 or Type 2?

| Requirement | Suitable Approach |
|---|---|
| Only latest value is needed | Type 1 |
| Historical changes must be retained | Type 2 |
| Only previous value is required | Type 3 |
| Original value must never change | Type 0 |

## 8.4 How to Avoid Fact-to-Fact Join Problems

Suppose:
- FactSales has one row per order item.
- FactPayment has one row per payment transaction.

An order can have multiple order items and multiple payment attempts.

Joining both facts directly on `order_id` can multiply rows.

**Better approach:**
1. Aggregate each fact to a compatible order-level grain.
2. Join the aggregated results.
3. Keep detailed facts separate when detailed analysis is required.

Example:

```sql
WITH sales_by_order AS (
    SELECT
        order_id,
        SUM(net_sales_amount) AS sales_amount
    FROM fact_sales
    GROUP BY order_id
),
payments_by_order AS (
    SELECT
        order_id,
        SUM(captured_amount) AS captured_amount
    FROM fact_payment
    GROUP BY order_id
)
SELECT
    s.order_id,
    s.sales_amount,
    p.captured_amount
FROM sales_by_order s
LEFT JOIN payments_by_order p
    ON s.order_id = p.order_id;
```

The business must define which payment records count toward captured amount, including reversals, refunds, and failed attempts.

## 8.5 How to Handle Late-Arriving Dimensions

A fact may arrive before its related dimension record.

Possible solutions:
- Delay fact loading.
- Create an inferred dimension member.
- Load a placeholder key.
- Update the dimension when full details arrive.
- Reprocess facts if the dimension key needs correction.

The selected method depends on freshness, audit, and reporting requirements.

## 8.6 How to Handle Late-Arriving Facts

A late-arriving fact represents an event that occurred earlier but reached the warehouse later.

Recommended approach:
- Preserve the original event timestamp.
- Use the business event date for reporting.
- Reprocess affected aggregates where needed.
- Make incremental loads safe to rerun.
- Track source progress and ingestion timestamps separately.

---

# 9. Data Warehouse Physical Design and Performance

## 9.1 Partitioning

Partition large tables based on common query filters and workload characteristics.

Examples:
- Sales by order date.
- Events by event date.
- Snapshots by snapshot date.

Avoid high-cardinality partition columns that create many small files.

Partitioning strategy depends on the storage and query engine.

## 9.2 Clustering and Data Skipping

Clustering organizes data to improve data locality for common filters.

In Delta Lake environments, options may include:
- `OPTIMIZE`.
- `ZORDER`.
- Liquid clustering.

Availability and recommended usage depend on the platform and runtime.

ZORDER and liquid clustering are not traditional database indexes.

## 9.3 File Format

| Format | Typical Use |
|---|---|
| CSV | Simple exchange and ingestion |
| JSON | Semi-structured records |
| Parquet | Columnar analytical storage |
| ORC | Columnar analytical storage |
| Delta | Transactional table framework using Parquet and a transaction log |

For analytical workloads, columnar formats such as Parquet and ORC are often more efficient than CSV because they support column-oriented reading and compression.

## 9.4 Data Quality Checks

A warehouse should validate:

**Completeness**
- Required records and fields are present.

**Uniqueness**
- Business keys are unique where expected.

**Validity**
- Values follow expected formats and business rules.

**Consistency**
- Related data agrees across tables.

**Accuracy**
- Measures match trusted business sources.

**Freshness**
- Data arrives within the required time.

## 9.5 Monitoring

Monitor:
- Pipeline success and failure.
- Data freshness.
- Source-to-target row counts.
- Duplicate records.
- Null rates.
- Schema changes.
- Processing duration.
- Storage growth.
- Query performance.

A successful pipeline run alone does not prove the warehouse data is correct.

---

# 10. Azure Data Warehouse Architecture Example

A possible Azure-based architecture:

```text
Source Systems
     |
     v
Azure Data Factory / Event Hubs
     |
     v
ADLS Gen2 - Bronze
     |
     v
Azure Databricks
     |
     v
ADLS Gen2 - Silver
     |
     v
Databricks SQL / Spark Transformations
     |
     v
ADLS Gen2 - Gold (Delta Tables)
     |
     v
Power BI / SQL Analytics
```

**Component responsibilities:**

| Component | Responsibility |
|---|---|
| Azure Data Factory | Batch orchestration and data movement |
| Event Hubs | Event ingestion |
| ADLS Gen2 | Scalable data storage |
| Azure Databricks | Distributed processing and transformations |
| Delta Lake | Transactional data tables |
| Databricks SQL | SQL-based analytical serving |
| Power BI | Reporting and visualization |

The architecture should be selected based on latency, scale, security, cost, and existing platform requirements. Not every system needs every component.

---

# 11. Interview Questions Based on These Cases

### Q1. How do you start designing a data warehouse?

First understand the business requirements and identify the business processes. Then define the grain, facts, dimensions, measures, keys, and historical requirements. Finally, design ingestion, transformations, quality checks, and physical storage.

### Q2. Why do you define grain before dimensions?

Grain determines what each fact row represents. It controls which measures and dimensions can be associated with that row and helps prevent incorrect aggregation and double counting.

### Q3. Can multiple fact tables share the same dimension?

Yes. A shared conformed dimension allows different business processes to use consistent definitions.

For example, `DimDate` can be shared by sales, returns, and inventory facts.

### Q4. Can we join two fact tables directly?

It is possible, but dangerous when the fact tables have different grains. First aggregate each fact to a compatible grain or use a model designed for the relationship.

### Q5. How do you handle historical customer changes?

Use SCD Type 2 when historical values are required. Maintain a new dimension row with a new surrogate key and effective dates.

### Q6. How do you handle duplicate source records?

Define the business key and deduplication rule. Use deterministic ordering, such as source update timestamp and sequence number. Do not remove records based only on arbitrary row order.

### Q7. How do you handle source schema changes?

Detect the change, compare it with the data contract, assess compatibility, update transformations and target schema where appropriate, and test downstream dependencies before deployment.

### Q8. How do you ensure source-to-target consistency?

Use key-level reconciliation, record counts, aggregate comparisons, duplicate checks, null checks, and business-rule validation.

### Q9. How do you design a warehouse for both historical and current reporting?

Use historical dimension tracking where needed, retain event or snapshot facts at the required grain, and provide current-state views or tables for consumers that need the latest values.

### Q10. How do you improve a slow warehouse query?

Inspect the execution plan and query metrics. Check scan volume, joins, data skew, partition pruning, file sizes, statistics, and aggregation strategy. Optimize based on the measured bottleneck.

---

# 12. Common Design Mistakes

1. Starting table design without understanding business requirements.
2. Not defining fact-table grain.
3. Combining unrelated business processes into one fact.
4. Joining facts at incompatible grains.
5. Double counting measures due to many-to-many relationships.
6. Treating semi-additive measures as fully additive.
7. Using SCD Type 1 when history is required.
8. Ignoring late-arriving records.
9. Treating payment attempts as successful payments.
10. Confusing invoices, payments, and recognized revenue.
11. Using ingestion time instead of business event time for reporting.
12. Over-partitioning large tables.
13. Treating successful pipeline execution as proof of data correctness.
14. Designing a complicated architecture before understanding scale and latency requirements.

---

# 13. Quick Revision Table

| Case | Main Fact Tables | Important Modeling Concept |
|---|---|---|
| E-commerce | Sales, Payment, Returns, Shipment | Multiple business processes and grains |
| Banking | Transaction, Account Snapshot, Loan Payment | Semi-additive balances |
| Retail | Sales, Inventory Snapshot, Inventory Movement | Transaction vs snapshot facts |
| Healthcare | Visit, Procedure, Admission, Billing | Many-to-many diagnoses and privacy |
| SaaS | Subscription Event, Invoice, Payment, Daily Snapshot | Lifecycle and recurring revenue |
| Logistics | Shipment, Shipment Event, Delivery Attempt | Event-level vs shipment-level grain |

## Final Interview Checklist

- [ ] Identify business processes from requirements.
- [ ] Define the grain of every fact table.
- [ ] Identify facts, dimensions, and measures.
- [ ] Design a star schema.
- [ ] Explain when a snowflake schema is appropriate.
- [ ] Use conformed dimensions across business processes.
- [ ] Implement SCD Type 2 conceptually.
- [ ] Model transaction and periodic snapshot facts.
- [ ] Explain additive and semi-additive measures.
- [ ] Prevent fact-to-fact join multiplication.
- [ ] Handle late-arriving facts and dimensions.
- [ ] Explain partitioning and clustering choices.
- [ ] Add data quality and reconciliation checks.
- [ ] Explain the end-to-end ingestion and transformation architecture.

**Preparation priority:** Practice the e-commerce, banking, and retail cases first. They cover most core data warehouse concepts: grain, fact and dimension design, star schema, SCD Type 2, snapshots, and avoiding double counting. Then practice explaining how you would adapt the same principles to SaaS, healthcare, and logistics.