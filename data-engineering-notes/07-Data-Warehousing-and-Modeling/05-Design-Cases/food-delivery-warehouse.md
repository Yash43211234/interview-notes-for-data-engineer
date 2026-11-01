# Food Delivery Data Warehouse Design

## 1. Introduction

A **Food Delivery Data Warehouse** is a centralized analytical system that stores and organizes data from a food delivery platform.

It helps businesses analyze:

- Total orders and revenue.
- Customer ordering behavior.
- Restaurant performance.
- Delivery partner performance.
- Order preparation and delivery time.
- Payment success and refunds.
- Cancellations and failed deliveries.
- Discounts and promotions.
- Customer ratings and complaints.

A Data Engineer designs pipelines to collect data from operational systems, clean and transform it, and make it available for reporting and analytics.

### Example Business Scenario

Consider a food delivery platform connecting customers, restaurants, and delivery partners.

A business team wants to answer questions such as:

1. How many orders are placed daily?
2. What is the total revenue generated?
3. Which restaurants receive the most orders?
4. Which areas have the highest order volume?
5. What is the average delivery time?
6. What percentage of orders are delivered late?
7. Which delivery partners complete the most deliveries?
8. What is the order cancellation rate?
9. How much revenue is lost through refunds and discounts?
10. Which customers order frequently?

The warehouse must support these analytical requirements without producing incorrect totals or misleading KPIs.

---

## 2. Understand the Source Systems

A food delivery platform usually has multiple operational systems.

| Source System | Data Generated |
|---|---|
| Customer System | Customer profiles and addresses |
| Restaurant System | Restaurant details, locations, menus |
| Order Management | Orders and order items |
| Payment System | Payment attempts, transactions, refunds |
| Delivery Management | Delivery partners, assignments, delivery status |
| Location System | Pickup and delivery locations |
| Promotion System | Coupons, discounts, campaigns |
| Rating System | Customer ratings and feedback |
| Support System | Complaints and support tickets |

### Example Source Tables

**customers**

| customer_id | customer_name | city |
|---|---|---|
| C001 | Rahul | Kolkata |
| C002 | Priya | Delhi |

**restaurants**

| restaurant_id | restaurant_name | city |
|---|---|---|
| R001 | Spice House | Kolkata |
| R002 | Pizza Corner | Delhi |

**orders**

| order_id | customer_id | restaurant_id | order_date | status |
|---|---|---|---|---|
| O001 | C001 | R001 | 2026-10-01 | Delivered |
| O002 | C002 | R002 | 2026-10-02 | Cancelled |

**order_items**

| order_item_id | order_id | item_name | quantity | unit_price |
|---|---|---|---:|---:|
| I001 | O001 | Biryani | 2 | 250 |
| I002 | O001 | Cold Drink | 1 | 50 |
| I003 | O002 | Pizza | 1 | 400 |

**delivery_assignments**

| assignment_id | order_id | delivery_partner_id | assigned_at | delivery_status |
|---|---|---|---|---|
| A001 | O001 | D001 | 2026-10-01 12:10 | Delivered |
| A002 | O002 | D002 | 2026-10-02 19:20 | Cancelled |

These are simplified examples. Real systems may have multiple assignment attempts, status events, payment attempts, and refunds for a single order.

---

## 3. Gather Business Requirements

Before designing the warehouse, identify the questions business teams need to answer.

### Order Analytics

- How many orders are placed daily?
- How many orders are delivered?
- How many orders are cancelled?
- What is the average order value?
- What are the peak ordering hours?

### Restaurant Analytics

- Which restaurants receive the most orders?
- Which restaurants generate the highest sales?
- What is the average food preparation time?
- Which restaurants have high cancellation rates?

### Customer Analytics

- Who are the most valuable customers?
- How frequently do customers place orders?
- How many new customers join each month?
- Which customers have stopped ordering?

### Delivery Analytics

- What is the average delivery time?
- What percentage of deliveries are on time?
- Which delivery partners complete the most orders?
- Which areas experience the most delivery delays?

### Payment Analytics

- What is the payment success rate?
- How many payments fail?
- What is the total refund amount?
- Which payment methods are used most frequently?

### Promotion Analytics

- Which coupons are used most frequently?
- How much discount is provided?
- Which campaigns generate more orders?
- Are promotional orders profitable?

These requirements help identify business processes, fact tables, dimensions, measures, and KPIs.

---

## 4. Identify Business Processes

A food delivery platform contains several business processes.

| Business Process | What It Represents |
|---|---|
| Order Placement | Customer placing an order |
| Order Items | Food items included in an order |
| Order Status History | Changes in order status |
| Restaurant Preparation | Food preparation milestones |
| Delivery Assignment | Assigning delivery partners |
| Delivery | Pickup and delivery events |
| Payments | Payment attempts and transactions |
| Refunds | Money returned to customers |
| Ratings | Customer feedback |
| Promotions | Coupon and discount usage |

Each process may require a separate fact table because the events and grains are different.

**Important:** Do not combine all these processes into one fact table. One order can have multiple items, payment attempts, delivery assignments, and status events.

---

## 5. Define the Grain of Each Fact Table

Grain defines exactly what one row represents.

It must be decided before selecting fact-table columns.

| Fact Table | Grain |
|---|---|
| `fact_order` | One row per order |
| `fact_order_item` | One row per order item |
| `fact_order_status_event` | One row per order status event |
| `fact_delivery` | One row per delivery attempt or delivery trip |
| `fact_delivery_assignment` | One row per delivery partner assignment |
| `fact_payment` | One row per payment transaction or attempt |
| `fact_refund` | One row per refund transaction |
| `fact_restaurant_preparation` | One row per order preparation instance |
| `fact_rating` | One row per submitted rating |
| `fact_promotion_usage` | One row per promotion application |

### Why Separate Grains Matter

Suppose an order has:

- Three food items.
- Two payment attempts.
- Two delivery assignment attempts.
- Four status events.

Joining all these tables directly on `order_id` can multiply rows.

As a result:
- Sales amounts may be counted multiple times.
- Payment totals may become incorrect.
- Delivery counts may be inflated.

**Rule:** Keep each fact table at its natural business-event grain. Aggregate to compatible grains before combining measures from different fact tables.

---

## 6. Design the Dimension Tables

Dimensions store descriptive information used to filter, group, and analyze facts.

### 6.1 dim_customer

Stores customer information.

| customer_key | customer_id | customer_name | city | state |
|---:|---|---|---|---|
| 101 | C001 | Rahul | Kolkata | West Bengal |
| 102 | C002 | Priya | Delhi | Delhi |

Possible attributes:
- Customer name.
- City.
- State.
- Country.
- Registration date.
- Customer segment.

Use SCD Type 2 if historical customer attributes, such as address or segment, must be preserved.

### 6.2 dim_restaurant

Stores restaurant information.

| restaurant_key | restaurant_id | restaurant_name | cuisine | city |
|---:|---|---|---|---|
| 201 | R001 | Spice House | Indian | Kolkata |
| 202 | R002 | Pizza Corner | Italian | Delhi |

Possible attributes:
- Restaurant name.
- Cuisine type.
- Restaurant category.
- City.
- State.
- Restaurant status.

### 6.3 dim_food_item

Stores food item information.

| food_item_key | food_item_id | item_name | category |
|---:|---|---|---|
| 301 | F001 | Biryani | Main Course |
| 302 | F002 | Cold Drink | Beverage |

Possible attributes:
- Item name.
- Category.
- Cuisine.
- Vegetarian/non-vegetarian classification.
- Menu status.

If menu prices change over time, preserve price history when required. The order-item fact should normally retain the actual charged unit price at purchase time.

### 6.4 dim_delivery_partner

Stores delivery partner information.

| delivery_partner_key | delivery_partner_id | partner_name | city |
|---:|---|---|---|
| 401 | D001 | Partner A | Kolkata |
| 402 | D002 | Partner B | Delhi |

Possible attributes:
- Partner name or permitted identifier.
- Operating city.
- Partner category.
- Employment or association status, where relevant.

Sensitive personal information should be restricted.

### 6.5 dim_date

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

### 6.6 dim_time

Useful for analyzing ordering patterns by time of day.

| time_key | hour | minute | time_period |
|---:|---:|---:|---|
| 1200 | 12 | 0 | Lunch |
| 1900 | 19 | 0 | Dinner |

Possible attributes:
- Hour.
- Minute.
- Day period.
- Meal period.

### 6.7 dim_location

Stores geographic information.

| location_key | area | city | state |
|---:|---|---|---|
| 501 | Salt Lake | Kolkata | West Bengal |
| 502 | Connaught Place | Delhi | Delhi |

A separate location dimension can represent customer, restaurant, pickup, or delivery locations. Use role-playing dimensions or separate keys where the business requires different location roles.

### 6.8 dim_payment_method

| payment_method_key | payment_method |
|---:|---|
| 601 | UPI |
| 602 | Credit Card |
| 603 | Net Banking |
| 604 | Cash |

### 6.9 dim_promotion

| promotion_key | coupon_code | promotion_type |
|---:|---|---|
| 701 | FIRST50 | First Order |
| 702 | SAVE20 | Percentage Discount |

A promotion dimension can store campaign descriptions, validity periods, and promotion categories.

### 6.10 dim_order_status

| order_status_key | order_status |
|---:|---|
| 801 | Placed |
| 802 | Preparing |
| 803 | Picked Up |
| 804 | Delivered |
| 805 | Cancelled |

A status dimension is useful for standardized descriptions and reporting categories. Some designs store status codes directly in facts.

---

## 7. Design the Fact Tables

Fact tables store measurable business events at a defined grain.

### 7.1 fact_order

**Grain:** One row per order.

Possible columns:

| Column | Purpose |
|---|---|
| order_id | Degenerate order identifier |
| customer_key | Customer dimension key |
| restaurant_key | Restaurant dimension key |
| order_date_key | Order date |
| order_time_key | Order time |
| delivery_location_key | Delivery location |
| order_status_key | Order status |
| subtotal_amount | Food subtotal |
| discount_amount | Discount |
| delivery_fee | Delivery charge |
| tax_amount | Tax |
| total_amount | Amount charged or payable, as defined |

This fact supports order-level reporting.

### 7.2 fact_order_item

**Grain:** One row per order item.

Possible columns:
- `order_item_id`
- `order_id`
- `food_item_key`
- `restaurant_key`
- `order_date_key`
- `quantity`
- `unit_price`
- `gross_item_amount`
- `item_discount`
- `net_item_amount`

Example:

| order_id | food_item_key | quantity | gross_item_amount | item_discount | net_item_amount |
|---|---:|---:|---:|---:|---:|
| O001 | 301 | 2 | 500 | 50 | 450 |
| O001 | 302 | 1 | 50 | 0 | 50 |

This table supports product-level and restaurant-level sales analysis.

### 7.3 fact_order_status_event

**Grain:** One row per order status event.

Possible columns:
- `order_id`
- `order_status_key`
- `status_event_date_key`
- `status_event_time_key`
- `event_timestamp`
- `status_duration_seconds`, if calculated from consecutive events

Example:

| order_id | status | event_timestamp |
|---|---|---|
| O001 | Placed | 12:00 |
| O001 | Accepted | 12:03 |
| O001 | Preparing | 12:05 |
| O001 | Picked Up | 12:25 |
| O001 | Delivered | 12:45 |

This fact supports order lifecycle analysis.

### 7.4 fact_delivery

**Grain:** One row per delivery attempt or delivery trip, based on the operational definition.

Possible columns:
- `delivery_id`
- `order_id`
- `delivery_partner_key`
- `pickup_location_key`
- `delivery_location_key`
- `pickup_date_key`
- `delivery_date_key`
- `pickup_timestamp`
- `delivery_timestamp`
- `delivery_duration_minutes`
- `delivery_status`
- `delivery_distance`

If an order is reassigned or delivery is retried, multiple delivery records may exist.

### 7.5 fact_delivery_assignment

**Grain:** One row per delivery partner assignment.

Possible columns:
- `assignment_id`
- `order_id`
- `delivery_partner_key`
- `assignment_timestamp`
- `acceptance_timestamp`
- `assignment_status`
- `rejection_reason_key`

This table helps analyze assignment acceptance, rejection, and reassignment behavior.

### 7.6 fact_payment

**Grain:** One row per payment attempt or transaction.

Possible columns:
- `payment_id`
- `order_id`
- `customer_key`
- `payment_method_key`
- `payment_date_key`
- `payment_amount`
- `payment_status`
- `transaction_reference`

A payment attempt is not necessarily a successful payment. Model payment success, failure, reversals, and refunds according to the source system's event definitions.

### 7.7 fact_refund

**Grain:** One row per refund transaction.

Possible columns:
- `refund_id`
- `order_id`
- `payment_id`
- `refund_date_key`
- `refund_amount`
- `refund_reason_key`
- `refund_status`

One order or payment may have multiple refund transactions.

### 7.8 fact_restaurant_preparation

**Grain:** One row per order preparation instance.

Possible columns:
- `order_id`
- `restaurant_key`
- `accepted_timestamp`
- `preparation_started_timestamp`
- `preparation_completed_timestamp`
- `preparation_duration_minutes`
- `preparation_status`

This supports restaurant preparation-time analysis.

### 7.9 fact_rating

**Grain:** One row per submitted rating.

Possible columns:
- `rating_id`
- `order_id`
- `customer_key`
- `restaurant_key`
- `delivery_partner_key`, if applicable
- `rating_date_key`
- `rating_value`
- `feedback_category`

An order may have separate restaurant and delivery ratings. If the platform supports both, model the rating subject explicitly.

### 7.10 fact_promotion_usage

**Grain:** One row per promotion application.

Possible columns:
- `promotion_usage_id`
- `order_id`
- `promotion_key`
- `customer_key`
- `promotion_date_key`
- `discount_amount`

If multiple promotions can apply to one order, preserve each application separately.

---

## 8. Food Delivery Warehouse Architecture

A practical architecture can use the medallion approach.

```text
                   Source Systems
                         |
       +-----------------+------------------+
       |          |      |       |          |
    Orders    Payments  Delivery Restaurant Customers
       |          |      |       |          |
       +----------+------+-------+----------+
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
        Validation   Deduplication  Standardization
            |            |             |
            +------------+-------------+
                         |
                         v
                    GOLD LAYER
                Dimensional Data Model
                         |
       +-----------------+------------------+
       |          |      |       |          |
       v          v      v       v          v
    Orders     Payments Delivery Restaurant Ratings
       |          |      |       |          |
       +----------+------+-------+----------+
                         |
                         v
                  BI / Analytics
                Power BI / Tableau
```

### Bronze Layer

Stores raw or minimally processed source data.

Examples:
- Raw orders.
- Raw order items.
- Raw customer records.
- Raw payment transactions.
- Raw delivery events.

### Silver Layer

Cleans and standardizes source data.

Examples:
- Validate schemas.
- Handle invalid records.
- Deduplicate source events.
- Standardize timestamps and status values.
- Validate order and customer identifiers.
- Handle late-arriving events.

### Gold Layer

Stores business-ready dimensional models and aggregates.

Examples:
- Order fact.
- Order-item fact.
- Payment fact.
- Delivery fact.
- Customer dimension.
- Restaurant dimension.
- Daily order and revenue aggregates.

**Important:** Bronze, Silver, and Gold are logical layers. They do not have to be separate physical systems or storage technologies.

---

## 9. Data Ingestion Strategy

Different source systems may require different ingestion methods.

| Source | Possible Ingestion Method |
|---|---|
| Orders database | Incremental query or CDC |
| Customer database | Incremental query or CDC |
| Restaurant database | Incremental query or CDC |
| Payment events | Event streaming or CDC |
| Delivery events | Event streaming |
| Menu catalog | Incremental ingestion |
| Ratings | Incremental ingestion |
| Promotions | Incremental ingestion |

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
- CDC.
- File-based ingestion.

### CDC

Change Data Capture captures source inserts, updates, and deletes when supported by the source and ingestion platform.

It is useful for tracking changes that a simple timestamp-based query may miss.

### Event-Based Ingestion

Order status, payment, and delivery events can be ingested continuously or in micro-batches.

For example:

```text
Order Placed
     |
     v
Restaurant Accepted
     |
     v
Food Preparing
     |
     v
Delivery Partner Assigned
     |
     v
Food Picked Up
     |
     v
Order Delivered
```

Not every order follows every state in this sequence. Cancellations, reassignments, failures, and other transitions must also be modeled.

### Important Design Considerations

- Maintain ingestion progress.
- Handle duplicate events.
- Handle out-of-order events.
- Process late-arriving data.
- Make target writes retry-safe.
- Preserve event timestamps and ingestion timestamps.
- Reconcile source and target counts and amounts.

**Important:** A batch extraction watermark is different from a Structured Streaming event-time watermark. They solve different problems.

---

## 10. Handling Order Status History

Order status changes are important for operational analytics.

A simple order table that stores only the current status cannot answer every historical question.

For example, if an order is currently delivered, the business may still need to know:

- When was it placed?
- When did the restaurant accept it?
- When did preparation finish?
- When was it picked up?
- When was it delivered?
- Was it reassigned to another delivery partner?

### Recommended Approach

Maintain an event-history dataset with one row per status change.

Example:

| order_id | status | event_timestamp |
|---|---|---|
| O001 | Placed | 12:00 |
| O001 | Accepted | 12:03 |
| O001 | Preparing | 12:05 |
| O001 | Picked Up | 12:25 |
| O001 | Delivered | 12:45 |

The current order status can be derived by selecting the latest valid event according to the source's event ordering rules.

### Example: Get Latest Status

```sql
WITH ranked_status AS (
    SELECT
        order_id,
        status,
        event_timestamp,
        ROW_NUMBER() OVER (
            PARTITION BY order_id
            ORDER BY event_timestamp DESC, event_id DESC
        ) AS rn
    FROM fact_order_status_event
)
SELECT
    order_id,
    status,
    event_timestamp
FROM ranked_status
WHERE rn = 1;
```

This assumes `event_id` provides a deterministic tie-breaker. If the source provides a reliable event sequence number, that may be more appropriate than event ID.

---

## 11. Handling Slowly Changing Dimensions

Customer, restaurant, and delivery partner attributes can change over time.

For example, a restaurant may change its operating area or category.

If historical reporting is required, SCD Type 2 can preserve old and new dimension versions.

### Example: dim_restaurant

| restaurant_key | restaurant_id | restaurant_name | city | effective_from | effective_to | is_current |
|---:|---|---|---|---|---|---|
| 201 | R001 | Spice House | Kolkata | 2026-01-01 | 2026-07-01 | FALSE |
| 305 | R001 | Spice House | Howrah | 2026-07-01 | NULL | TRUE |

The fact record should reference the dimension version valid at the time of the event when historical attribute reporting is required.

### SCD Strategies

- Type 0: Retain original value.
- Type 1: Overwrite existing value.
- Type 2: Insert a new version and preserve history.
- Type 3: Store a previous value in an additional column.

Use the strategy based on the business requirement for each attribute.

---

## 12. Important Food Delivery KPIs

KPI definitions must be agreed upon with business teams. The numerator, denominator, filters, and time period must be explicit.

| KPI | Example Definition |
|---|---|
| Total Orders | Count of distinct qualifying orders |
| Delivered Orders | Count of orders with delivered outcome |
| Order Cancellation Rate | Cancelled orders / eligible orders × 100 |
| Average Order Value | Qualifying order revenue / qualifying orders |
| Average Delivery Time | Average delivery duration for delivered orders |
| On-Time Delivery Rate | On-time delivered orders / eligible delivered orders × 100 |
| Payment Success Rate | Successful payment attempts / eligible payment attempts × 100 |
| Refund Amount | Sum of qualifying refund amounts |
| Average Preparation Time | Average preparation duration |
| Average Rating | Average valid submitted rating |
| Restaurant Acceptance Rate | Accepted restaurant orders / eligible restaurant order requests × 100 |

### Important KPI Considerations

- Decide whether cancelled orders are included in order totals.
- Define the revenue basis: food subtotal, customer payable amount, or platform revenue.
- Decide whether discounts and refunds reduce reported sales.
- Define delivery SLA and whether the denominator includes only completed deliveries.
- Decide whether payment success is measured by attempts or orders.
- Avoid comparing KPIs calculated with different business definitions.

---

## 13. Example Analytical Queries

The following queries assume the fact and dimension columns described in this design. Date functions and syntax may vary by database.

### 13.1 Total Orders

```sql
SELECT
    COUNT(DISTINCT order_id) AS total_orders
FROM fact_order;
```

### 13.2 Daily Order Count

```sql
SELECT
    d.full_date,
    COUNT(DISTINCT f.order_id) AS total_orders
FROM fact_order f
JOIN dim_date d
    ON f.order_date_key = d.date_key
GROUP BY
    d.full_date
ORDER BY
    d.full_date;
```

### 13.3 Total Food Sales by Restaurant

```sql
SELECT
    r.restaurant_name,
    SUM(f.net_item_amount) AS total_food_sales
FROM fact_order_item f
JOIN dim_restaurant r
    ON f.restaurant_key = r.restaurant_key
GROUP BY
    r.restaurant_name
ORDER BY
    total_food_sales DESC;
```

This measures net item sales as defined in the order-item fact. It does not automatically represent the platform's commission or total revenue.

### 13.4 Top 5 Restaurants by Order Count

```sql
SELECT
    r.restaurant_name,
    COUNT(DISTINCT f.order_id) AS total_orders
FROM fact_order f
JOIN dim_restaurant r
    ON f.restaurant_key = r.restaurant_key
GROUP BY
    r.restaurant_name
ORDER BY
    total_orders DESC
LIMIT 5;
```

### 13.5 Average Delivery Time

```sql
SELECT
    AVG(delivery_duration_minutes) AS avg_delivery_minutes
FROM fact_delivery
WHERE delivery_status = 'Delivered';
```

This assumes duration is calculated consistently from pickup to delivery. If the business measures from order placement to delivery, use that definition instead.

### 13.6 Order Cancellation Rate

```sql
SELECT
    COUNT(CASE
        WHEN order_status = 'Cancelled' THEN 1
    END) * 100.0
    / NULLIF(COUNT(*), 0) AS cancellation_rate
FROM fact_order;
```

This assumes one row per order and that all rows belong to the eligible order population.

### 13.7 On-Time Delivery Rate

Assume `fact_delivery` contains actual and promised delivery timestamps.

```sql
SELECT
    COUNT(CASE
        WHEN actual_delivery_timestamp <= promised_delivery_timestamp
        THEN 1
    END) * 100.0
    / NULLIF(COUNT(*), 0) AS on_time_delivery_rate
FROM fact_delivery
WHERE delivery_status = 'Delivered';
```

This uses delivered attempts as the denominator. If an order has multiple delivery attempts, the business must decide whether the KPI is measured per attempt or per order.

### 13.8 Payment Success Rate

```sql
SELECT
    COUNT(CASE
        WHEN payment_status = 'Success' THEN 1
    END) * 100.0
    / NULLIF(COUNT(*), 0) AS payment_success_rate
FROM fact_payment;
```

This measures successful attempts divided by all attempts. To measure the percentage of orders with at least one successful payment, calculate distinct successful orders divided by eligible distinct orders.

### 13.9 Top Delivery Partners

```sql
SELECT
    dp.partner_name,
    COUNT(DISTINCT f.delivery_id) AS completed_deliveries
FROM fact_delivery f
JOIN dim_delivery_partner dp
    ON f.delivery_partner_key = dp.delivery_partner_key
WHERE f.delivery_status = 'Delivered'
GROUP BY
    dp.partner_name
ORDER BY
    completed_deliveries DESC
LIMIT 10;
```

If one delivery ID can contain multiple orders, this query measures delivery trips, not delivered orders. Choose the measure based on the operational definition.

### 13.10 Average Restaurant Preparation Time

```sql
SELECT
    r.restaurant_name,
    AVG(f.preparation_duration_minutes) AS avg_preparation_minutes
FROM fact_restaurant_preparation f
JOIN dim_restaurant r
    ON f.restaurant_key = r.restaurant_key
WHERE f.preparation_status = 'Completed'
GROUP BY
    r.restaurant_name
ORDER BY
    avg_preparation_minutes;
```

---

## 14. Data Quality Checks

Data quality is important because incorrect operational data can produce misleading business KPIs.

### Important Checks

| Quality Check | Example |
|---|---|
| Completeness | Every valid order has an order date |
| Uniqueness | Order IDs are unique in the order-level fact |
| Validity | Quantity is greater than zero |
| Referential Integrity | Fact dimension keys resolve correctly |
| Consistency | Order status values follow standard definitions |
| Timestamp Validity | Delivery time is not before pickup time |
| Reconciliation | Source and target order counts match |
| Freshness | New order events arrive within the expected SLA |

### Example: Check Duplicate Orders

```sql
SELECT
    order_id,
    COUNT(*) AS record_count
FROM fact_order
GROUP BY order_id
HAVING COUNT(*) > 1;
```

### Example: Check Invalid Order Items

```sql
SELECT *
FROM fact_order_item
WHERE quantity IS NULL
   OR quantity <= 0
   OR unit_price < 0;
```

### Example: Check Invalid Delivery Duration

```sql
SELECT *
FROM fact_delivery
WHERE delivery_duration_minutes < 0;
```

### Example: Check Missing Restaurant Keys

```sql
SELECT *
FROM fact_order
WHERE restaurant_key IS NULL;
```

A missing dimension reference may be represented by an approved unknown or not-applicable dimension member. The business rules should define how such records are handled.

---

## 15. Performance Optimization

Food delivery platforms can generate large volumes of orders and events.

Performance should be considered during data model and pipeline design.

### 15.1 Partitioning

Partition large datasets using appropriate, commonly filtered columns.

Examples:
- Order date.
- Event date.
- Delivery date.

Avoid high-cardinality partition columns such as order IDs or customer IDs if they create excessive small partitions.

### 15.2 Columnar Storage

Formats such as Parquet are suitable for analytical workloads because they support column-oriented reads and compression.

### 15.3 Predicate Pushdown and Partition Pruning

Apply filters that allow the query engine and storage format to reduce unnecessary data reads where supported.

For example, monthly order reports should avoid scanning unrelated periods when the layout and query allow pruning.

### 15.4 File Management

Small files can increase metadata and scheduling overhead.

Use suitable file sizing, optimized writes, and compaction features supported by the platform.

### 15.5 Pre-Aggregation

Create daily or hourly aggregates for frequently accessed dashboards.

Examples:
- Daily restaurant orders.
- Hourly order volume.
- Daily delivery performance.
- City-level revenue.

### 15.6 Query Optimization

- Select only required columns.
- Filter early where appropriate.
- Avoid unnecessary shuffles.
- Use suitable join strategies.
- Check execution plans and runtime metrics.
- Cache only data that is reused.

Optimization should be validated against actual workloads. There is no universal partition count or file size that works for every dataset.

---

## 16. Security and Governance

A food delivery warehouse may contain customer addresses, contact details, payment references, and delivery partner information.

Important practices:

- Restrict access based on business roles.
- Avoid exposing unnecessary personal information.
- Mask sensitive fields where appropriate.
- Store credentials securely.
- Maintain audit logs.
- Track data lineage.
- Apply data retention policies.
- Follow applicable privacy and regulatory requirements.

In Databricks environments, Unity Catalog can help manage access, governance, lineage, and data discovery.

Cloud storage permissions and catalog permissions may be separate layers.

---

## 17. End-to-End Project Explanation

This is a useful structure for explaining a food delivery warehouse in a Data Engineer interview.

### Project Objective

Build a centralized data warehouse to analyze orders, restaurants, customers, payments, delivery performance, and promotions.

### Step 1: Source Ingestion

Collect data from:
- Orders.
- Order items.
- Customers.
- Restaurants.
- Payments.
- Delivery assignments.
- Delivery events.
- Promotions.
- Ratings.

Use full or incremental ingestion according to source capabilities.

### Step 2: Bronze Processing

Store raw source data with ingestion metadata.

### Step 3: Silver Processing

- Validate schemas.
- Clean invalid records.
- Deduplicate events.
- Standardize timestamps and statuses.
- Handle late-arriving events.
- Apply business validation rules.

### Step 4: Gold Modeling

Build:
- `fact_order`
- `fact_order_item`
- `fact_order_status_event`
- `fact_delivery`
- `fact_delivery_assignment`
- `fact_payment`
- `fact_refund`
- `fact_restaurant_preparation`
- `fact_rating`
- Required customer, restaurant, food item, date, time, location, and promotion dimensions.

### Step 5: Historical Tracking

Use SCD Type 1 or Type 2 according to the historical reporting requirements.

### Step 6: KPI Calculation

Calculate:
- Total orders.
- Order cancellation rate.
- Average order value.
- Average delivery time.
- On-time delivery rate.
- Payment success rate.
- Refund amount.
- Restaurant preparation time.

### Step 7: Data Validation

Perform:
- Duplicate checks.
- Null checks.
- Referential integrity checks.
- Timestamp validation.
- Source-to-target reconciliation.
- KPI validation.

### Step 8: Reporting

Publish Gold datasets to BI tools for business dashboards.

### Step 9: Monitoring

Monitor:
- Pipeline failures.
- Data freshness.
- Processing duration.
- Data quality.
- Event volume anomalies.
- Reconciliation differences.

---

## 18. Common Interview Questions and Answers

### Q1. How would you design a food delivery data warehouse?

I would first gather business requirements and identify processes such as order placement, order items, payments, restaurant preparation, delivery, refunds, and ratings. Then I would define the grain of each fact table, identify dimensions and measures, and design the dimensional model. I would build ingestion and transformation pipelines, validate the data, and publish business-ready datasets.

### Q2. Why do we need separate fact tables?

Different processes have different grains. Orders, order items, payments, and delivery assignments represent different business events. Separate fact tables prevent incorrect joins and double counting.

### Q3. What is the grain of a food delivery order fact?

A common grain is one row per order. It supports order-level measures such as order count and order amount.

### Q4. What is the grain of an order-item fact?

One row per item within an order. It supports food-item-level sales and quantity analysis.

### Q5. How would you calculate average delivery time?

I would define the start and end events first. For example, pickup timestamp to delivered timestamp measures the delivery journey, while order placement to delivered timestamp measures the complete customer waiting time. Then I would calculate the average over the eligible completed deliveries.

### Q6. How would you handle multiple delivery partners assigned to one order?

I would maintain a separate delivery assignment fact with one row per assignment. The delivery fact would represent the actual delivery attempt or trip according to the operational definition.

### Q7. How would you track order status changes?

I would maintain an event-history table with one row per status event. This preserves the order lifecycle and supports analysis of preparation, assignment, pickup, and delivery durations.

### Q8. How would you handle late-arriving delivery events?

I would preserve event timestamps, process late records using idempotent logic, update the affected order or delivery state, and recalculate affected aggregates when necessary.

### Q9. How would you calculate cancellation rate?

I would agree on the eligible order population, count cancelled orders, and divide by the eligible order count. The definition should clarify treatment of test orders, duplicates, and other excluded orders.

### Q10. How would you optimize a large food delivery warehouse?

I would use suitable columnar storage, date-based partitioning or clustering where appropriate, file compaction, query pruning, efficient joins, and pre-aggregations for common dashboards. I would validate improvements using execution plans and runtime metrics.

### Q11. Why should payment data be separate from order data?

An order may have multiple payment attempts, including failures and retries. Keeping payment events separate prevents payment amounts and order measures from being duplicated.

### Q12. How would you track restaurant performance?

I would use order and order-item facts for sales and volume, preparation facts for preparation time, and rating facts for customer feedback. I would define each KPI carefully and aggregate each dataset at the correct grain.

### Q13. How would you handle refunds?

I would model refund transactions separately because an order or payment may have multiple refunds. I would define how refunds affect net sales and revenue metrics.

### Q14. How would you handle customer address changes?

If historical location reporting is required, I would use SCD Type 2. Facts would reference the customer dimension version valid at the event time.

### Q15. What is the biggest risk when joining food delivery fact tables?

The biggest risk is joining facts with different grains. Multiple order items, payment attempts, and delivery assignments can multiply rows and inflate measures. I would aggregate each fact to a compatible grain before combining them.

---

## 19. Common Design Mistakes

| Mistake | Correct Approach |
|---|---|
| Combining orders, payments, and deliveries in one fact | Model separate business processes |
| Not defining grain | Define what one row represents |
| Joining all facts directly by order ID | Check cardinality and aggregate to compatible grains |
| Counting order-item rows as orders | Count distinct order IDs or use order-level fact |
| Storing only the current order status | Preserve status-event history when needed |
| Assuming one delivery assignment per order | Model assignment attempts separately |
| Treating payment attempts as successful payments | Define payment success rules |
| Ignoring refunds | Model refund events separately |
| Summing inventory balances across dates | Treat inventory as semi-additive |
| Ignoring late or out-of-order events | Preserve event time and process corrections safely |
| Assuming a successful pipeline means accurate data | Validate quality and reconciliation |

---

## 20. Quick Revision

### Main Source Systems

- Customers.
- Restaurants.
- Food catalog.
- Orders.
- Payments.
- Delivery.
- Promotions.
- Ratings.
- Support.

### Main Fact Tables

| Fact Table | Grain |
|---|---|
| `fact_order` | One row per order |
| `fact_order_item` | One row per order item |
| `fact_order_status_event` | One row per status event |
| `fact_delivery` | One row per delivery attempt or trip |
| `fact_delivery_assignment` | One row per partner assignment |
| `fact_payment` | One row per payment attempt or transaction |
| `fact_refund` | One row per refund transaction |
| `fact_restaurant_preparation` | One row per preparation instance |
| `fact_rating` | One row per submitted rating |
| `fact_promotion_usage` | One row per promotion application |

### Main Dimensions

- `dim_customer`
- `dim_restaurant`
- `dim_food_item`
- `dim_delivery_partner`
- `dim_date`
- `dim_time`
- `dim_location`
- `dim_payment_method`
- `dim_promotion`
- `dim_order_status`

### Architecture

```text
Sources → Bronze → Silver → Gold → BI
```

### Most Important Concepts

1. Gather business requirements before designing tables.
2. Identify separate business processes.
3. Define the grain of every fact table.
4. Use separate facts for orders, items, payments, and delivery events.
5. Use conformed dimensions where appropriate.
6. Preserve order status history when required.
7. Handle multiple delivery assignments and payment attempts.
8. Apply SCD based on historical reporting requirements.
9. Handle late-arriving and out-of-order events.
10. Prevent double counting across different fact grains.
11. Validate source-to-target reconciliation.
12. Monitor pipeline health and data freshness.

### One-Minute Interview Answer

For a food delivery data warehouse, I would first identify the source systems and business requirements, such as orders, restaurants, customers, payments, delivery, refunds, and ratings.

I would define a separate grain for each fact table. For example, the order fact would have one row per order, the order-item fact one row per food item in an order, the payment fact one row per payment attempt, and the delivery assignment fact one row per partner assignment.

I would create shared dimensions such as customer, restaurant, food item, date, time, and location. I would preserve order status history through an event-level fact table and use SCD Type 2 where historical attribute tracking is required.

I would build Bronze, Silver, and Gold processing layers, implement incremental ingestion, handle late-arriving events, and apply data quality checks.

Finally, I would publish business-ready datasets for KPIs such as order volume, cancellation rate, average delivery time, payment success rate, and restaurant performance. I would carefully manage fact-table grain to prevent double counting.

---

## 21. Self-Assessment

- [ ] Explain the objective of a food delivery data warehouse.
- [ ] Identify the source systems.
- [ ] Gather business requirements.
- [ ] Identify distinct business processes.
- [ ] Define the grain of every fact table.
- [ ] Design order and order-item facts.
- [ ] Design payment and refund facts.
- [ ] Design delivery and assignment facts.
- [ ] Explain order status-event history.
- [ ] Identify conformed dimensions.
- [ ] Explain star schema design.
- [ ] Explain SCD Type 1 and Type 2.
- [ ] Explain incremental ingestion and CDC.
- [ ] Handle late-arriving and out-of-order events.
- [ ] Calculate cancellation and delivery KPIs.
- [ ] Identify fact-to-fact join risks.
- [ ] Explain data quality and reconciliation.
- [ ] Explain partitioning and query optimization.
- [ ] Explain security and governance.
- [ ] Present the end-to-end architecture in an interview.

**Interview readiness:** Given a food delivery business scenario, be able to identify source systems, business processes, fact-table grains, dimensions, measures, ingestion strategy, event history, KPIs, quality checks, and analytical outputs.