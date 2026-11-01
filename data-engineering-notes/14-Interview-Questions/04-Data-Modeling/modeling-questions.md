# Data Modeling Interview Questions and Answers

## 1. Data Modeling Fundamentals

### Q1. What is Data Modeling?

Data modeling is the process of defining how data is organized, related, stored, and accessed in a database or data warehouse.

It helps determine:
- What data needs to be stored.
- How entities relate to each other.
- What each record represents.
- How data should be structured for querying and reporting.
- How data quality and consistency can be maintained.

**Example:** In an e-commerce system, we may model customers, products, orders, payments, and order items as related entities.

### Q2. Why Is Data Modeling Important?

Data modeling helps:
- Maintain consistent business definitions.
- Reduce unnecessary data duplication.
- Improve query performance.
- Support reporting and analytics.
- Maintain data integrity.
- Make systems easier to understand and maintain.

A good model should represent business requirements accurately while supporting expected query patterns.

### Q3. What Are the Three Levels of Data Modeling?

| Level | Description | Example |
|---|---|---|
| Conceptual | High-level business entities and relationships | Customer places Order |
| Logical | Attributes, keys, relationships, and rules | Customer ID, Order ID, Order Date |
| Physical | Actual database implementation | Tables, columns, data types, indexes |

**Conceptual model:** What information does the business need?

**Logical model:** How are the entities and their relationships structured?

**Physical model:** How will the model be implemented in a particular database?

### Q4. What Is an Entity?

An entity is a real-world object or concept about which we store information.

Examples:
- Customer
- Employee
- Product
- Order
- Department

An entity generally becomes a table in a relational database.

### Q5. What Is an Attribute?

An attribute describes a property of an entity.

For a Customer entity:

- customer_id
- customer_name
- email
- phone_number
- city

Attributes generally become table columns.

### Q6. What Is a Relationship?

A relationship describes how two entities are associated.

Examples:
- A customer places orders.
- An order contains products.
- An employee belongs to a department.

Relationships can be one-to-one, one-to-many, or many-to-many.

### Q7. What Is Cardinality?

Cardinality describes how many records of one entity can be associated with records of another entity.

| Relationship | Meaning | Example |
|---|---|---|
| One-to-one | One record relates to one record | Person and Passport |
| One-to-many | One record relates to multiple records | Customer and Orders |
| Many-to-many | Multiple records relate to multiple records | Students and Courses |

### Q8. What Is Optionality?

Optionality indicates whether participation in a relationship is mandatory or optional.

Example:
- An order must belong to a customer: mandatory relationship.
- A customer may not have placed any orders: optional relationship from the customer side.

Cardinality and optionality together describe relationship rules.

---

## 2. Keys and Constraints

### Q9. What Is a Primary Key?

A primary key uniquely identifies each row in a table.

Properties:
- Values must be unique.
- A primary key cannot contain NULL.
- A table has one primary-key constraint, which can contain one or multiple columns.

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100)
);
```

### Q10. What Is a Foreign Key?

A foreign key establishes a relationship between tables by referencing a candidate key, usually the primary key, in another table.

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);
```

A foreign key helps enforce referential integrity.

### Q11. What Is the Difference Between Primary Key and Foreign Key?

| Primary Key | Foreign Key |
|---|---|
| Identifies a row uniquely | References a key in another table |
| Must be unique | Can contain repeated values |
| Cannot contain NULL | NULL may be allowed, depending on the constraint |
| One primary-key constraint per table | A table can have multiple foreign keys |

### Q12. What Is a Candidate Key?

A candidate key is a minimal set of columns that can uniquely identify a row.

A table may have multiple candidate keys.

Example:
- employee_id
- company_email

If both uniquely identify employees, both may be candidate keys.

One candidate key is selected as the primary key.

### Q13. What Is a Composite Key?

A composite key uses two or more columns together to uniquely identify a row.

Example:

```sql
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);
```

The combination of `order_id` and `product_id` identifies an order item in this simplified model.

### Q14. What Is a Surrogate Key?

A surrogate key is an artificial identifier created for a record, without direct business meaning.

Example:

```text
customer_key = 10001
customer_id  = CUST-458
```

The surrogate key is generated for database modeling purposes. The customer ID is the business identifier.

Surrogate keys are especially useful in dimensional modeling and for tracking historical dimension versions.

### Q15. What Is a Natural Key?

A natural key is an identifier that already exists in the business domain.

Examples:
- Government-issued identifier.
- Employee number.
- Product code.
- Customer account number.

Natural keys have business meaning but may change, be reused, or have data-quality issues. Their suitability depends on the business rules.

### Q16. What Is Referential Integrity?

Referential integrity ensures that relationships between related tables remain valid.

Example:

If an order references customer ID 100, that customer must exist in the referenced customer table, unless the relationship explicitly allows NULL.

Foreign keys are a common way to enforce referential integrity in relational databases.

---

## 3. Normalization

### Q17. What Is Normalization?

Normalization is the process of organizing relational data to reduce unnecessary duplication and prevent data anomalies.

It usually involves splitting data into related tables and defining appropriate keys and relationships.

### Q18. What Are the Advantages of Normalization?

- Reduces unnecessary data duplication.
- Improves data consistency.
- Makes updates safer.
- Reduces insertion and deletion anomalies.
- Helps maintain data integrity.

### Q19. What Is 1NF (First Normal Form)?

A table is in First Normal Form when each column contains a single value for each row, and there are no repeating groups.

**Incorrect structure:**

| customer_id | phone_numbers |
|---|---|
| 1 | 9876, 8765 |

**Better structure:**

| customer_id | phone_number |
|---|---|
| 1 | 9876 |
| 1 | 8765 |

Each phone number is stored separately.

### Q20. What Is 2NF (Second Normal Form)?

A table is in Second Normal Form when:
1. It is in 1NF.
2. Every non-key attribute depends on the whole candidate key, not just part of a composite candidate key.

Example:

```text
OrderItem
---------
order_id
product_id
product_name
quantity
```

Suppose the candidate key is `(order_id, product_id)`.

`product_name` depends only on `product_id`, not on the complete composite key. This is a partial dependency.

**Solution:** Move product details to a separate Product table.

### Q21. What Is 3NF (Third Normal Form)?

A table is in Third Normal Form when:
1. It is in 2NF.
2. Non-key attributes do not have transitive dependencies on candidate keys.

Example:

```text
Employee
--------
employee_id
department_id
department_name
```

If `department_name` depends on `department_id`, rather than directly on `employee_id`, there is a transitive dependency.

**Solution:** Separate Department into its own table.

### Q22. What Is BCNF?

Boyce-Codd Normal Form is a stronger form of 3NF.

A relation is in BCNF when every non-trivial functional dependency has a superkey as its determinant.

In simple terms, every determinant must be capable of uniquely identifying a row.

BCNF handles certain dependency situations that 3NF can still allow.

### Q23. What Is a Functional Dependency?

A functional dependency exists when one attribute or set of attributes determines another attribute.

Notation:

```text
A -> B
```

This means that for a given value of A, there is only one corresponding value of B in the relation.

Example:

```text
employee_id -> employee_name
```

An employee ID determines the employee name, assuming employee IDs are unique.

### Q24. What Are Insertion, Update, and Deletion Anomalies?

These anomalies can occur when data is poorly structured.

- **Insertion anomaly:** A fact cannot be inserted without adding unrelated information.
- **Update anomaly:** The same fact must be updated in multiple rows, and some rows may be missed.
- **Deletion anomaly:** Removing one record accidentally removes another important fact.

Normalization helps reduce these problems.

### Q25. Is Normalization Always Better Than Denormalization?

No.

Normalization is useful for transactional systems where consistency and efficient updates matter.

Denormalization can be useful in analytical systems where read performance and simpler reporting are important.

The correct approach depends on workload, data volume, update patterns, and query requirements.

---

## 4. Dimensional Modeling

### Q26. What Is Dimensional Modeling?

Dimensional modeling is a data warehouse design approach that organizes data into fact tables and dimension tables.

It makes analytical queries easier to understand and often improves reporting performance.

It focuses on business processes, measurable events, and descriptive context.

### Q27. What Is a Fact Table?

A fact table stores measurements and business events at a defined level of detail.

Examples of measures:
- Sales amount.
- Quantity sold.
- Discount amount.
- Transaction count.

A fact table usually contains foreign keys referencing dimensions.

Example:

```text
FactSales
---------
sales_key
date_key
customer_key
product_key
store_key
quantity
sales_amount
discount_amount
```

### Q28. What Is a Dimension Table?

A dimension table stores descriptive information used to analyze facts.

Examples:
- Customer dimension.
- Product dimension.
- Date dimension.
- Store dimension.

Example:

```text
DimCustomer
-----------
customer_key
customer_id
customer_name
city
state
country
```

### Q29. What Is the Difference Between Fact and Dimension Tables?

| Fact Table | Dimension Table |
|---|---|
| Stores measurable events | Stores descriptive context |
| Usually contains many rows | Often has fewer rows than a large fact table |
| Contains foreign keys | Contains descriptive attributes |
| Supports calculations and aggregations | Supports filtering and grouping |

### Q30. What Is Grain in Dimensional Modeling?

Grain defines exactly what one row in a fact table represents.

It is one of the most important decisions in dimensional modeling.

Example:

```text
One row = One product line in one order
```

This means an order with three products produces three fact rows.

Before selecting dimensions or measures, define the grain clearly.

### Q31. Why Is Grain Important?

Grain determines:
- What each fact row means.
- Which measures can be stored.
- Which dimensions can be linked.
- How aggregation should work.
- Whether joins can produce duplicate or inflated results.

Mixing different grains in the same fact table can create incorrect analytical results.

### Q32. What Are the Types of Fact Tables?

**1. Transaction Fact Table**

Stores individual business events.

Example: One row per order line.

**2. Periodic Snapshot Fact Table**

Stores measurements at regular intervals.

Example: Daily account balance.

**3. Accumulating Snapshot Fact Table**

Tracks a process through important milestones.

Example: Order created, packed, shipped, and delivered.

**4. Factless Fact Table**

Stores events or relationships without numeric measures.

Example: Student attendance or product promotion eligibility.

**5. Aggregate Fact Table**

Stores pre-aggregated measures to support common analytical queries.

Example: Daily sales by product category and region.

### Q33. What Are Additive, Semi-Additive, and Non-Additive Measures?

| Type | Meaning | Example |
|---|---|---|
| Additive | Can be summed across all relevant dimensions | Sales amount |
| Semi-additive | Can be summed across some dimensions but not all | Account balance |
| Non-additive | Cannot be meaningfully summed | Percentage, ratio |

Example:

Daily account balances can be summed across accounts at a particular point in time, but summing balances across multiple dates usually does not produce a meaningful result.

### Q34. What Is a Star Schema?

A star schema contains a central fact table connected directly to dimension tables.

```text
             DimCustomer
                  |
DimProduct -- FactSales -- DimDate
                  |
               DimStore
```

**Advantages:**
- Simple structure.
- Easy-to-understand joins.
- Suitable for reporting and analytics.
- Often performs well in analytical workloads.

### Q35. What Is a Snowflake Schema?

A snowflake schema normalizes dimension tables into additional related tables.

Example:

```text
FactSales
    |
DimProduct
    |
DimCategory
    |
DimDepartment
```

**Advantages:**
- Reduces repeated descriptive data.
- Can improve consistency in dimension maintenance.

**Disadvantages:**
- Requires more joins.
- Can make reporting queries more complex.

### Q36. Star Schema vs Snowflake Schema

| Star Schema | Snowflake Schema |
|---|---|
| Dimensions are generally denormalized | Dimensions are normalized into related tables |
| Fewer joins | More joins |
| Simpler reporting queries | More complex queries |
| May repeat descriptive attributes | Reduces some duplication |

Neither is universally better. Choose based on reporting needs, maintenance, performance, and platform behavior.

### Q37. What Is a Conformed Dimension?

A conformed dimension is a dimension shared consistently across multiple fact tables or business processes.

Example:

A common `DimDate` may be used by:
- FactSales.
- FactReturns.
- FactInventory.

This enables consistent analysis across different business processes.

### Q38. What Is a Role-Playing Dimension?

A role-playing dimension is one dimension used in multiple roles in the same fact table.

Example:

An order fact table may reference `DimDate` as:
- Order Date.
- Shipping Date.
- Delivery Date.

The underlying date dimension is shared, but each relationship has a different business meaning.

### Q39. What Is a Degenerate Dimension?

A degenerate dimension is a dimension identifier stored directly in a fact table without a separate dimension table.

Example:
- Order number.
- Invoice number.
- Transaction reference.

It is useful when the identifier has business value but has few or no descriptive attributes requiring a separate dimension.

### Q40. What Is a Junk Dimension?

A junk dimension combines multiple low-cardinality flags and indicators into one dimension.

Example:

```text
DimOrderFlags
-------------
order_status_flag
gift_wrap_flag
priority_flag
```

It avoids placing many unrelated small flags directly into a large fact table.

---

## 5. Slowly Changing Dimensions (SCD)

### Q41. What Is a Slowly Changing Dimension?

A Slowly Changing Dimension is a technique used to manage changes in dimension attributes over time.

Example:

A customer changes their city from Kolkata to Bengaluru.

The data warehouse must decide whether to overwrite the old city or preserve the history.

### Q42. What Is SCD Type 0?

Type 0 preserves the original value and does not update it.

Example:

A customer's original registration date remains unchanged.

### Q43. What Is SCD Type 1?

Type 1 overwrites the old value with the new value.

Example:

```text
Before:
customer_id | city
101         | Kolkata

After:
customer_id | city
101         | Bengaluru
```

**Characteristics:**
- No historical tracking.
- Simple implementation.
- Useful when old values are not required.

### Q44. What Is SCD Type 2?

Type 2 preserves historical versions by creating a new row when tracked attributes change.

Example:

| customer_key | customer_id | city | effective_from | effective_to | is_current |
|---|---|---|---|---|---|
| 1 | 101 | Kolkata | 2024-01-01 | 2025-06-30 | false |
| 2 | 101 | Bengaluru | 2025-07-01 | 9999-12-31 | true |

**Characteristics:**
- Maintains history.
- Uses a new surrogate key for each version.
- Commonly uses effective dates and a current-record indicator.
- Requires correct historical dimension-key assignment in fact tables.

### Q45. What Is SCD Type 3?

Type 3 stores limited history by keeping the previous value in an additional column.

Example:

| customer_id | current_city | previous_city |
|---|---|---|
| 101 | Bengaluru | Kolkata |

It is suitable when the business needs only limited historical comparison.

### Q46. Difference Between SCD Type 1, Type 2, and Type 3

| Type | Approach | History |
|---|---|---|
| Type 1 | Overwrite existing value | No |
| Type 2 | Insert a new version row | Full history for tracked changes |
| Type 3 | Store previous value in another column | Limited history |

### Q47. How Would You Implement SCD Type 2?

**Typical process:**

1. Read incoming source records.
2. Identify business keys.
3. Compare tracked attributes with current dimension records.
4. Identify new and changed records.
5. Expire the current version of changed records.
6. Insert new versions.
7. Insert completely new business entities.
8. Validate that only one current version exists per business key.

A simplified SQL-style illustration:

```sql
-- Illustrative approach:
-- Expire the old version of a changed customer.

UPDATE dim_customer
SET
    effective_to = DATE '2025-07-01',
    is_current = FALSE
WHERE customer_id = 101
  AND is_current = TRUE;

-- Insert the new version.

INSERT INTO dim_customer (
    customer_key,
    customer_id,
    city,
    effective_from,
    effective_to,
    is_current
)
VALUES (
    2,
    101,
    'Bengaluru',
    DATE '2025-07-01',
    DATE '9999-12-31',
    TRUE
);
```

This is an illustration, not a complete production SCD2 implementation. Production logic must handle concurrent writes, multiple changes, date boundaries, surrogate-key generation, and transaction behavior.

### Q48. What Is the Difference Between Business Key and Surrogate Key in SCD Type 2?

- **Business key:** Identifies the real-world entity across versions, such as `customer_id`.
- **Surrogate key:** Identifies a particular historical version, such as `customer_key`.

For one customer, multiple historical rows can share the same business key but must have different surrogate keys.

### Q49. How Do You Join a Fact Table to an SCD Type 2 Dimension?

Use the business key and the fact's event date to find the dimension version valid at that time.

```sql
SELECT
    f.order_id,
    f.customer_id,
    d.customer_key,
    f.sales_amount
FROM fact_sales f
JOIN dim_customer d
    ON f.customer_id = d.customer_id
   AND f.order_date >= d.effective_from
   AND f.order_date < d.effective_to;
```

This uses a half-open effective-date interval:

`effective_from <= event_time < effective_to`

The model should prevent overlapping validity periods for the same business key.

---

## 6. Advanced Data Modeling Concepts

### Q50. What Is a Bridge Table?

A bridge table handles many-to-many relationships in a dimensional model.

Example:

One customer can belong to multiple customer groups, and one group can contain multiple customers.

```text
BridgeCustomerGroup
-------------------
customer_key
group_key
```

For historical or weighted relationships, the bridge may also require effective dates or allocation percentages.

### Q51. What Is a Role of an Allocation Factor in a Bridge Table?

An allocation factor defines how a measure should be distributed across multiple related entities.

Example:

A sale of ₹1,000 belongs to two sales representatives:
- Representative A: 60%.
- Representative B: 40%.

The allocation factors are 0.60 and 0.40.

Allocated sales:
- A: ₹600.
- B: ₹400.

The allocation factors should follow the business rules and generally sum to 1 for a fully allocated measure.

### Q52. What Is Data Vault Modeling?

Data Vault is a data warehouse modeling approach designed for integrating data from multiple sources while preserving historical information and supporting auditability.

Its main components are:

**1. Hub**
- Stores unique business keys.
- Represents core business entities.

**2. Link**
- Stores relationships between hubs.

**3. Satellite**
- Stores descriptive attributes and their history.

Example:

```text
HubCustomer
    |
LinkCustomerOrder
    |
HubOrder

SatelliteCustomer
SatelliteOrder
```

Data Vault is often used in enterprise data integration and historical data warehousing. It is not the same as a star schema.

### Q53. What Is the Difference Between Data Vault and Dimensional Modeling?

| Data Vault | Dimensional Modeling |
|---|---|
| Focuses on integration and historical traceability | Focuses on analytical consumption |
| Uses hubs, links, satellites | Uses facts and dimensions |
| Supports multiple source systems | Supports business reporting |
| Often requires additional transformations for reporting | Usually easier for BI tools to consume |

Both approaches can be used in the same data platform.

### Q54. What Is a Wide Table?

A wide table contains many columns, often combining attributes from multiple entities or pre-aggregated information.

Advantages:
- Fewer joins for common queries.
- Simple consumption for some reporting workloads.

Disadvantages:
- Repeated data.
- More complicated updates.
- Potentially larger storage.
- Harder maintenance if business logic is duplicated.

### Q55. What Is a Data Mart?

A data mart is a subject-oriented subset of data designed for a particular business area.

Examples:
- Sales data mart.
- Finance data mart.
- Customer data mart.
- Inventory data mart.

It can be built from an enterprise data warehouse or a broader lakehouse platform.

### Q56. What Is the Difference Between OLTP and OLAP?

| OLTP | OLAP |
|---|---|
| Supports transactional operations | Supports analytical queries |
| Frequent inserts, updates, deletes | Often read-heavy workloads |
| Usually normalized | Often dimensional or denormalized |
| Many short transactions | Complex queries and aggregations |
| Example: Order placement | Example: Monthly sales reporting |

### Q57. Why Are Star Schemas Common in Data Warehouses?

Star schemas provide:
- Clear business structure.
- Simple fact-to-dimension joins.
- Easier filtering and grouping.
- Consistent reporting.
- Good compatibility with many BI tools.

Actual performance depends on the query engine, table size, statistics, storage layout, and execution plan.

---

## 7. Practical Data Modeling Scenarios

### Q58. Design a Data Model for an E-Commerce System.

First identify the business processes:
- Customer registration.
- Product management.
- Order placement.
- Payment.
- Shipment.
- Returns.

A possible operational model:

```text
Customer
--------
customer_id
customer_name
email

Product
-------
product_id
product_name
category_id
price

Orders
------
order_id
customer_id
order_date
order_status

OrderItem
---------
order_item_id
order_id
product_id
quantity
unit_price

Payment
-------
payment_id
order_id
payment_status
payment_amount

Shipment
--------
shipment_id
order_id
shipment_status
```

For analytics, create separate fact and dimension structures.

```text
FactSales
---------
date_key
customer_key
product_key
order_id
order_item_id
quantity
sales_amount

DimCustomer
DimProduct
DimDate
DimStore
```

**Important:** Define the fact grain before deciding which measures and dimensions belong in the fact table. Payment and shipment may need their own facts because they represent different business processes and grains.

### Q59. Design a Sales Data Warehouse.

**Business requirements:**
- Analyze sales by product.
- Analyze sales by customer.
- Compare sales by month and region.
- Track discounts and quantities.

**Possible model:**

```text
FactSales
---------
date_key
customer_key
product_key
store_key
order_id
quantity
gross_sales
discount_amount
net_sales

DimDate
DimCustomer
DimProduct
DimStore
```

**Grain:** One row per product line in an order.

Possible measures:
- Quantity.
- Gross sales.
- Discount amount.
- Net sales.

### Q60. How Would You Model Customer History?

Use an SCD Type 2 dimension if historical customer attributes are required.

For example, when a customer changes their city:
- Expire the existing dimension version.
- Insert a new version.
- Assign a new surrogate key.
- Ensure facts use the version valid at the event time.

If the business only needs the latest city, Type 1 may be more appropriate.

### Q61. How Would You Model an Order With Multiple Products?

Use two entities:

**Orders**

One row per order.

**OrderItem**

One row per product line within an order.

Example:

```text
Orders
------
order_id
customer_id
order_date

OrderItem
---------
order_item_id
order_id
product_id
quantity
unit_price
```

This avoids storing multiple products in a single order column and allows each product line to be analyzed separately.

### Q62. How Would You Model Daily Inventory?

Use a periodic snapshot fact table.

Example:

```text
FactInventorySnapshot
---------------------
date_key
product_key
warehouse_key
opening_quantity
received_quantity
sold_quantity
closing_quantity
```

**Grain:** One row per product, warehouse, and snapshot date.

Inventory balance is semi-additive over time. It can often be aggregated across products or warehouses at a point in time, but should not normally be summed across multiple dates.

### Q63. How Would You Model an Order Lifecycle?

Use an accumulating snapshot fact table when the business needs to track milestones for each order.

Example:

```text
FactOrderLifecycle
------------------
order_id
order_date_key
packed_date_key
shipped_date_key
delivered_date_key
order_amount
```

The row is updated as the order moves through its lifecycle.

If detailed event history is also needed, maintain a separate transaction/event fact table.

### Q64. How Would You Prevent Double Counting in a Data Warehouse?

**Common causes:**
- Joining tables with different grains.
- Duplicate dimension keys.
- Many-to-many relationships.
- Incorrect bridge-table usage.
- Aggregating before resolving duplicates.

**Solution:**
1. Define the grain of every fact table.
2. Validate key uniqueness where expected.
3. Aggregate to compatible grains before joining.
4. Use bridge allocation rules when necessary.
5. Validate totals against trusted source measures.

Do not use `DISTINCT` as a general fix for incorrect joins.

### Q65. How Would You Model Returns and Refunds?

Returns and refunds are separate business events from the original sale.

A possible design:

```text
FactSales
---------
order_id
product_key
sales_date_key
quantity_sold
sales_amount

FactReturns
-----------
return_id
order_id
product_key
return_date_key
quantity_returned
refund_amount
```

This preserves the original sale and records the return separately.

Business rules must clarify whether sales reports show gross sales, net sales, refunds, or net revenue, and which date determines the reporting period.

---

## 8. Data Modeling and Data Engineering

### Q66. How Does Data Modeling Fit Into an ETL/ELT Pipeline?

Data modeling defines the target structure that transformation logic should produce.

A typical pipeline:

```text
Source Systems
      |
   Ingestion
      |
   Raw Layer
      |
  Validation
      |
Transformation
      |
  Data Model
      |
Analytics / BI
```

The model determines:
- Target tables.
- Keys and relationships.
- Grain.
- Historical tracking.
- Measures and dimensions.

### Q67. What Is the Difference Between a Logical Model and a Physical Model?

A logical model defines entities, attributes, relationships, and business rules without committing to a particular database implementation.

A physical model defines actual tables, columns, data types, constraints, indexes, partitioning, and other implementation details for a specific platform.

### Q68. How Do You Choose Between Normalized and Denormalized Structures?

Consider:
- Transactional or analytical workload.
- Read and write frequency.
- Query complexity.
- Data duplication.
- Update patterns.
- Performance requirements.
- Data governance.

Use normalized structures when consistency and updates are important. Use denormalized structures when they simplify analytical queries and improve consumption performance.

### Q69. How Would You Handle Late-Arriving Dimensions?

A late-arriving dimension occurs when a fact arrives before its related dimension record is available.

Possible approaches:
- Delay fact processing until the dimension arrives.
- Create an inferred or placeholder dimension record.
- Update the inferred record when full details arrive.
- Reprocess affected facts if necessary.

The choice depends on reporting freshness and business requirements.

### Q70. What Is a Late-Arriving Fact?

A late-arriving fact is a business event that reaches the warehouse after its expected processing time.

Example:

An order created on Monday is received by the warehouse on Wednesday.

The pipeline should use the event's business date, not blindly treat the ingestion date as the event date.

### Q71. What Is a Data Contract?

A data contract is an agreement between data producers and consumers about the structure and expected behavior of a dataset.

It may define:
- Schema and data types.
- Required fields.
- Business definitions.
- Quality expectations.
- Ownership.
- Compatibility and change rules.
- Freshness expectations.

Data contracts help prevent unexpected source changes from breaking downstream pipelines.

### Q72. How Would You Handle a Source Schema Change?

**Approach:**
1. Detect the schema change.
2. Compare it with the existing contract.
3. Determine whether it is compatible.
4. Validate downstream dependencies.
5. Update transformations and target structures when needed.
6. Test before production deployment.
7. Record and communicate the change.

Do not automatically accept every new or changed column without checking its business meaning.

---

## 9. Common Interview Mistakes

1. Designing tables before defining business requirements.
2. Forgetting to define fact-table grain.
3. Confusing business keys with surrogate keys.
4. Treating every numeric column as an additive measure.
5. Using SCD Type 1 when historical analysis is required.
6. Assuming a star schema is always faster than a snowflake schema.
7. Joining fact tables without checking their grain.
8. Using `DISTINCT` to hide duplicate join results.
9. Ignoring late-arriving facts and dimensions.
10. Confusing operational modeling with dimensional modeling.
11. Storing multiple values in one relational column without a clear reason.
12. Assuming every business process belongs in one fact table.
13. Forgetting that a dimension can have multiple historical versions.
14. Ignoring currency, time zone, and business definitions in analytical measures.

---

## 10. Frequently Asked Comparison Questions

| Concept A | Concept B | Main Difference |
|---|---|---|
| Entity | Attribute | Object vs property |
| Primary key | Foreign key | Identifies row vs references another key |
| Natural key | Surrogate key | Business identifier vs generated identifier |
| Normalization | Denormalization | Reduce duplication vs intentionally combine data |
| Fact | Dimension | Measurement/event vs descriptive context |
| Star | Snowflake | Denormalized dimensions vs normalized dimensions |
| OLTP | OLAP | Transactions vs analytics |
| SCD Type 1 | SCD Type 2 | Overwrite vs preserve historical versions |
| Transaction fact | Snapshot fact | Individual event vs state at a point in time |
| Business key | Surrogate key | Entity identity vs version/row identity |
| Data Mart | Data Warehouse | Focused business subset vs broader analytical repository |
| Data Vault | Star Schema | Integration and history vs analytical consumption |

---

## 11. Quick Revision

- **Data modeling:** Defines how data is structured and related.
- **Entity:** Object or concept about which data is stored.
- **Attribute:** Property of an entity.
- **Cardinality:** Number of records participating in a relationship.
- **Primary key:** Unique row identifier.
- **Foreign key:** Reference to a key in another table.
- **Normalization:** Reduces unnecessary duplication and anomalies.
- **1NF:** Atomic values and no repeating groups.
- **2NF:** No partial dependency on a composite candidate key.
- **3NF:** No transitive dependency of non-key attributes on candidate keys.
- **Fact table:** Stores measurable business events at a defined grain.
- **Dimension table:** Stores descriptive context.
- **Grain:** Meaning of one fact-table row.
- **Star schema:** Central fact connected to dimensions.
- **Snowflake schema:** Normalized dimension structure.
- **SCD Type 1:** Overwrites old values.
- **SCD Type 2:** Creates a new historical version.
- **SCD Type 3:** Stores limited history in additional columns.
- **Conformed dimension:** Shared dimension with consistent meaning.
- **Bridge table:** Handles many-to-many relationships.
- **Data Vault:** Uses hubs, links, and satellites.
- **OLTP:** Transaction processing.
- **OLAP:** Analytical processing.

---

## 12. Self-Assessment Checklist

### Fundamentals
- [ ] Explain conceptual, logical, and physical models.
- [ ] Explain entities, attributes, relationships, and cardinality.
- [ ] Differentiate primary, foreign, candidate, natural, and surrogate keys.
- [ ] Explain referential integrity.

### Normalization
- [ ] Explain 1NF, 2NF, 3NF, and BCNF.
- [ ] Explain functional dependencies.
- [ ] Identify insertion, update, and deletion anomalies.
- [ ] Explain normalization versus denormalization.

### Dimensional Modeling
- [ ] Explain fact and dimension tables.
- [ ] Define grain for a business process.
- [ ] Explain transaction, periodic snapshot, and accumulating snapshot facts.
- [ ] Explain additive, semi-additive, and non-additive measures.
- [ ] Compare star and snowflake schemas.
- [ ] Explain conformed, role-playing, degenerate, and junk dimensions.

### Historical Modeling
- [ ] Explain SCD Types 0, 1, 2, and 3.
- [ ] Implement a basic SCD Type 2 process.
- [ ] Explain business keys and surrogate keys.
- [ ] Join facts to the correct historical dimension version.

### Practical Design
- [ ] Design an e-commerce data model.
- [ ] Design a sales warehouse.
- [ ] Model order items and returns.
- [ ] Model inventory snapshots.
- [ ] Prevent double counting across different fact grains.
- [ ] Explain late-arriving facts and dimensions.

**Preparation priority:** Focus first on grain, fact and dimension tables, star and snowflake schemas, normalization, surrogate keys, and SCD Type 2. Be ready to design an e-commerce or sales model and explain why you chose each table and relationship.