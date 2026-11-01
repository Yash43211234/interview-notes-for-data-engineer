# Data Modeling – Quick Revision Cheat Sheet

> **Purpose:** Fast revision for Data Engineer interviews covering relational modeling, normalization, dimensional modeling, fact and dimension tables, SCD, warehouse design, and common SQL interview traps.

---

## 1. Data Modeling Fundamentals

**Data modeling** is the process of organizing data, defining relationships, and establishing rules so data can be stored, maintained, and queried correctly.

### Three Levels of Data Modeling

| Level | Focus | Example |
|---|---|---|
| Conceptual | Business entities and relationships | Customer places Order |
| Logical | Attributes, keys, relationships, and business rules | Customer has CustomerID and Name |
| Physical | Actual database implementation | SQL types, indexes, partitions, constraints |

### Common Terms

| Term | Meaning |
|---|---|
| Entity | Something the business tracks |
| Attribute | A property of an entity |
| Relationship | Association between entities |
| Primary Key | Uniquely identifies a row |
| Foreign Key | References a key in another or the same table |
| Candidate Key | Minimal attribute set that uniquely identifies a row |
| Surrogate Key | Artificial key created for identification |
| Natural Key | Key derived from business data |
| Grain | What one row represents |
| Constraint | Rule that data must satisfy |

### Example

```text
Customer
--------
customer_id (PK)
customer_name
email

Order
-----
order_id (PK)
customer_id (FK)
order_date
order_total
```

Relationship: one customer can place many orders.

---

## 2. Primary Key vs Foreign Key

| Primary Key | Foreign Key |
|---|---|
| Identifies a row | References a key in another or the same table |
| Must be unique | May repeat |
| Cannot be NULL | May be NULL if allowed by the design |
| One primary-key constraint per table | Multiple foreign-key constraints are possible |

A foreign key helps enforce referential integrity when the database constraint is enabled and enforced.

**Interview point:** A data warehouse may contain logical relationships that are not physically enforced through foreign-key constraints.

---

## 3. Normalization

Normalization reduces unnecessary duplication and modification anomalies in relational databases.

### Normal Forms

| Normal Form | Quick Rule |
|---|---|
| 1NF | Each attribute contains a value from its defined domain; no repeating groups |
| 2NF | In 1NF, with no partial dependency of a non-prime attribute on a candidate key |
| 3NF | In 2NF, with no prohibited transitive dependency of non-prime attributes on a candidate key |
| BCNF | Every determinant of a non-trivial functional dependency is a superkey |

### Example: Before Normalization

| OrderID | CustomerName | Product |
|---|---|---|
| 101 | Rahul | Laptop |
| 102 | Rahul | Mouse |

Customer details repeat.

### After Normalization

**Customer**

| CustomerID | CustomerName |
|---|---|
| 1 | Rahul |

**Orders**

| OrderID | CustomerID | Product |
|---|---|---|
| 101 | 1 | Laptop |
| 102 | 1 | Mouse |

### Why Normalize?

- Reduce redundant data.
- Improve consistency.
- Prevent insert, update, and delete anomalies.
- Make transactional systems easier to maintain.

### Normalization vs Denormalization

| Normalization | Denormalization |
|---|---|
| Reduces redundancy | Intentionally duplicates or combines data |
| Often useful in OLTP | Often useful in analytics |
| More relationships and joins may be needed | Queries may require fewer joins |
| Supports consistent updates | Can simplify reads but complicate updates |

---

## 4. OLTP vs OLAP

| OLTP | OLAP |
|---|---|
| Online Transaction Processing | Online Analytical Processing |
| Supports daily operations | Supports analysis and reporting |
| Many short transactions | Complex analytical queries |
| Frequent inserts and updates | Large scans and aggregations |
| Usually normalized | Often dimensional or denormalized |
| Example: order checkout | Example: yearly sales analysis |

**Remember:** OLTP and OLAP describe workload patterns, not an absolute rule that a particular schema or technology can only serve one purpose.

---

## 5. Dimensional Modeling

Dimensional modeling organizes analytical data around business processes and descriptive dimensions.

### Core Components

- **Fact table:** Stores measurements or business events at a defined grain.
- **Dimension table:** Stores descriptive context.
- **Grain:** Defines exactly what one fact row represents.
- **Measure:** A value that can be analyzed or aggregated.

### Example: Sales Warehouse

**FactSales**

| DateKey | CustomerKey | ProductKey | Quantity | SalesAmount |
|---|---|---|---:|---:|
| 20261001 | 10 | 501 | 2 | 2000 |
| 20261001 | 11 | 502 | 1 | 750 |

Dimensions:

- DimDate
- DimCustomer
- DimProduct
- DimStore

### Star Schema

<text color="secondary" size="sm">Structure: one central fact table connected to denormalized dimensions.</text>

```text
             DimDate
                |
DimCustomer -- FactSales -- DimProduct
                |
             DimStore
```

Advantages:
- Easy to understand.
- Fewer joins for common analytical queries.
- Suitable for BI and reporting.

### Snowflake Schema

Dimensions are further normalized into related tables.

Advantages:
- Less repeated descriptive data in some designs.
- Clearer representation of hierarchies in some cases.

Disadvantages:
- More joins.
- Potentially more complex queries.

| Star | Snowflake |
|---|---|
| Dimensions usually denormalized | Dimensions more normalized |
| Simpler analytical queries | More dimension joins |
| Common BI-friendly choice | Useful when normalized hierarchies are justified |

---

## 6. Fact Tables

A fact table records business events, measurements, or snapshots.

### Types of Fact Tables

| Type | Meaning | Example |
|---|---|---|
| Transaction Fact | One row per business event | One order line |
| Periodic Snapshot | State measured at regular intervals | Daily inventory balance |
| Accumulating Snapshot | Tracks progress through a process | Order lifecycle |
| Factless Fact | Records an event or relationship without numeric measures | Student attendance |
| Aggregate Fact | Stores pre-aggregated measurements | Monthly product sales |

### Additivity of Measures

| Type | Meaning | Example |
|---|---|---|
| Additive | Can be summed across relevant dimensions | Sales amount |
| Semi-additive | Can be summed across some dimensions, but not all | Account balance across accounts, but generally not across time |
| Non-additive | Cannot be meaningfully summed directly | Percentage, ratio |

**Interview trap:** A daily inventory balance should not normally be summed across dates to calculate inventory at a point in time.

### Fact Table Design Rules

1. Define the grain first.
2. Identify the business process.
3. Choose dimensions.
4. Identify measures.
5. Select keys.
6. Define additive behavior.
7. Handle late-arriving data and corrections.
8. Validate that joins do not multiply fact rows unexpectedly.

---

## 7. Dimension Tables

Dimensions describe the business context of facts.

Examples:
- Customer
- Product
- Date
- Store
- Geography
- Employee

### Common Dimension Attributes

| Dimension | Attributes |
|---|---|
| Customer | Name, segment, city |
| Product | Product name, category, brand |
| Date | Day, month, quarter, year |
| Store | Store name, region, country |

### Surrogate Key vs Natural Key

| Surrogate Key | Natural Key |
|---|---|
| Artificial identifier | Business-derived identifier |
| Can remain stable when business attributes change | May change or be reused |
| Useful for historical dimension versions | Useful for matching source entities |
| Example: `CustomerKey = 1001` | Example: `CustomerID = C123` |

A warehouse commonly uses surrogate keys for dimensions while retaining the natural or source key for matching.

---

## 8. Slowly Changing Dimensions (SCD)

SCD techniques manage changes to dimension attributes over time.

| Type | Behavior | Typical Use |
|---|---|---|
| Type 0 | Preserve original value | Immutable attribute |
| Type 1 | Overwrite old value | Correction without history |
| Type 2 | Insert a new version row | Historical tracking |
| Type 3 | Store limited previous value | Simple before-and-after tracking |

### SCD Type 1

Before:

| CustomerID | City |
|---|---|
| C101 | Kolkata |

After the customer moves:

| CustomerID | City |
|---|---|
| C101 | Bengaluru |

The previous city is overwritten.

### SCD Type 2

| CustomerKey | CustomerID | City | EffectiveFrom | EffectiveTo | IsCurrent |
|---:|---|---|---|---|---|
| 101 | C101 | Kolkata | 2025-01-01 | 2026-04-01 | false |
| 205 | C101 | Bengaluru | 2026-04-01 | 9999-12-31 | true |

The exact end-date convention varies by implementation.

### SCD Type 2 Rules

- Keep a stable business or natural key to identify the entity.
- Create a new surrogate key for each historical version.
- Preserve the previous version.
- Track effective dates or an equivalent validity range.
- Mark the current version when useful.
- Map facts to the correct dimension version for the event time.

A common half-open validity rule is:

```sql
event_time >= effective_from
AND event_time < effective_to
```

### Common SCD Interview Questions

**Q: When do you use Type 1?**

When the latest corrected value is needed and historical changes are not required.

**Q: When do you use Type 2?**

When historical reporting must reflect the dimension attributes that were valid at the time of the event.

**Q: What happens when a late-arriving fact comes in?**

Resolve the dimension version using the fact's business event time. If the relevant historical dimension is missing, follow a defined late-arriving dimension process.

---

## 9. Keys and Relationships

### Relationship Types

| Type | Example |
|---|---|
| One-to-One | Person and a uniquely associated record |
| One-to-Many | Customer and orders |
| Many-to-Many | Students and courses |

### Many-to-Many Relationship

Use a bridge or junction table when the relationship cannot be represented correctly by a single foreign key.

```text
Student
-------
student_id

Course
------
course_id

StudentCourse
-------------
student_id
course_id
```

In dimensional models, bridge tables can represent multi-valued relationships, such as a customer associated with several segments.

If a fact is associated with multiple dimension members, joins can multiply fact rows. Allocation weights or another explicit business rule may be required when measures must be apportioned.

---

## 10. Data Vault Basics

Data Vault is a modeling approach designed for integrating and preserving historical business data.

| Component | Purpose |
|---|---|
| Hub | Business key and identity |
| Link | Relationship between business entities |
| Satellite | Descriptive attributes and history |

Example:

- HubCustomer
- HubProduct
- LinkCustomerOrder
- SatelliteCustomerDetails

**Remember:** Data Vault is often used as an integration and historical storage layer. Dimensional models may still be built for BI and reporting.

---

## 11. Data Mart, Data Warehouse, and Data Lake

| Concept | Purpose |
|---|---|
| Data Warehouse | Integrated analytical data |
| Data Mart | Data organized for a business domain or group |
| Data Lake | Stores raw and processed data in varied formats |
| Lakehouse | Combines data-lake storage with transactional table capabilities and governance |

A lakehouse commonly uses object storage, a table format such as Delta Lake, and catalog/governance capabilities.

---

## 12. Choosing the Right Grain

**Grain is one of the most important decisions in dimensional modeling.**

Examples:

| Fact Table | Possible Grain |
|---|---|
| Sales | One row per order line |
| Payments | One row per payment transaction |
| Inventory | One row per product, store, and snapshot time |
| Web Events | One row per recorded event |
| Shipment Events | One row per shipment status event |

### Why Grain Matters

Suppose an order contains three product lines and two payment transactions. Joining order lines directly to payment transactions can produce six rows.

This can inflate sales and payment totals.

**Solution:** Aggregate or model each process at a compatible grain before combining measures.

Do not use `DISTINCT` as a general fix for incorrect join cardinality.

---

## 13. Common Warehouse Design Questions

### Q1. Design an E-Commerce Warehouse

Possible facts:
- FactOrderLine
- FactPaymentTransaction
- FactRefund
- FactShipmentEvent

Possible dimensions:
- DimCustomer
- DimProduct
- DimDate
- DimStore
- DimGeography

Define separate grains for orders, order lines, payments, refunds, and shipment events.

### Q2. How Would You Model Inventory?

Use a periodic snapshot at a defined grain, such as product-store-day.

Measures may include:
- Opening quantity.
- Received quantity.
- Sold quantity.
- Closing quantity.

Closing inventory is semi-additive across time. Choose the correct snapshot date for point-in-time reporting.

### Q3. How Would You Handle Customer History?

Use SCD Type 2 if reports need to reflect historical customer attributes.

### Q4. How Would You Prevent Duplicate Sales?

- Define the order-line business key.
- Deduplicate source records deterministically.
- Define update and cancellation rules.
- Validate join cardinality.
- Reconcile totals against the source.
- Handle refunds and currency consistently.

### Q5. How Would You Handle Late-Arriving Data?

- Preserve business event time and ingestion time separately.
- Use event time for historical reporting.
- Update affected partitions or aggregates.
- Reprocess the appropriate time range.
- Make writes idempotent.
- Reconcile corrected outputs.

---

## 14. SQL Patterns for Modeling

### Find Duplicate Business Keys

```sql
SELECT order_id, order_line_id, COUNT(*) AS row_count
FROM fact_sales
GROUP BY order_id, order_line_id
HAVING COUNT(*) > 1;
```

Use the actual business key for the table's grain.

### Find Records Without a Matching Dimension

```sql
SELECT f.*
FROM fact_sales AS f
LEFT JOIN dim_product AS d
    ON f.product_key = d.product_key
WHERE d.product_key IS NULL;
```

This pattern can identify unmatched dimension keys, provided the join key and NULL assumptions are correct.

### Select the Latest Source Record

```sql
WITH ranked AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY updated_at DESC, source_sequence DESC
        ) AS rn
    FROM customer_updates
)
SELECT *
FROM ranked
WHERE rn = 1;
```

The tie-breaker must define a deterministic order. If the source sequence is not unique, add a suitable stable tie-breaker.

### Important SQL Caution

A successful row-count comparison does not prove two datasets contain the same records. Compare keys, values, business rules, and relevant aggregates.

---

## 15. Data Modeling vs Physical Optimization

| Logical Modeling | Physical Optimization |
|---|---|
| Defines entities and relationships | Defines storage and execution choices |
| Establishes grain | Chooses partitions or clustering |
| Defines keys and constraints | Chooses indexes where supported |
| Defines facts and dimensions | Chooses file sizes and layout |
| Defines business semantics | Optimizes query execution |

Do not use physical optimization to compensate for an incorrect data model.

Examples of physical design choices include:
- Partitioning.
- Clustering.
- Indexing.
- File format.
- Materialized aggregates.
- Compression.

The right choice depends on the database or lakehouse platform and the query workload.

---

## 16. Common Interview Traps

1. Designing fact tables before defining the grain.
2. Mixing order, payment, and refund measures at incompatible grains.
3. Summing semi-additive balances across time.
4. Treating every dimension change as Type 2.
5. Using natural keys as the only key for every historical dimension version.
6. Joining fact tables directly without checking cardinality.
7. Using `DISTINCT` to hide fanout.
8. Assuming every data warehouse enforces foreign keys physically.
9. Confusing a data warehouse with a data lake.
10. Assuming a denormalized schema is always faster.
11. Ignoring late-arriving facts and dimensions.
12. Treating ingestion time as business event time.
13. Failing to define cancellation, refund, and revenue rules.
14. Ignoring time zones and currency conversion.
15. Treating row-count equality as complete data validation.

---

## 17. Last-Minute Revision – 25 Key Facts

1. Conceptual = business view.
2. Logical = entities, attributes, and relationships.
3. Physical = implementation details.
4. Primary key = uniquely identifies a row.
5. Foreign key = references another key.
6. Normalization = reduce redundancy and anomalies.
7. OLTP = operational workload.
8. OLAP = analytical workload.
9. Fact = measurement or business event.
10. Dimension = descriptive context.
11. Grain = exact meaning of one fact row.
12. Star = central fact with denormalized dimensions.
13. Snowflake = more normalized dimensions.
14. Transaction fact = one row per event at its defined grain.
15. Periodic snapshot = state at regular intervals.
16. Accumulating snapshot = process lifecycle.
17. Additive = sum across relevant dimensions.
18. Semi-additive = sum across some dimensions.
19. SCD Type 1 = overwrite.
20. SCD Type 2 = preserve history in new versions.
21. Bridge = represent many-to-many relationships.
22. Hub = business identity in Data Vault.
23. Link = relationship in Data Vault.
24. Satellite = descriptive attributes and history.
25. Compatible grain = essential for correct joins and aggregates.

---

## 18. Interview Self-Assessment

- [ ] Explain conceptual, logical, and physical models.
- [ ] Explain primary, foreign, natural, and surrogate keys.
- [ ] Explain 1NF, 2NF, 3NF, and BCNF.
- [ ] Compare OLTP and OLAP.
- [ ] Design a star schema.
- [ ] Explain fact-table types.
- [ ] Explain additive and semi-additive measures.
- [ ] Explain SCD Types 1 and 2.
- [ ] Handle late-arriving facts and dimensions.
- [ ] Explain bridge tables and many-to-many relationships.
- [ ] Identify fact-to-fact join fanout.
- [ ] Design an e-commerce warehouse.
- [ ] Explain Data Vault hubs, links, and satellites.
- [ ] Explain data warehouses, marts, lakes, and lakehouses.
- [ ] Validate model correctness using SQL.

**Final reminder:** In a modeling interview, explain the business process first, define the grain, identify the measures and dimensions, and then choose keys, relationships, and physical optimizations.