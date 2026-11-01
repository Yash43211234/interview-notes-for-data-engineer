# Normalization vs Denormalization in Data Warehousing

## 1. Introduction

Normalization and denormalization are two approaches to organizing data in a database.

- **Normalization:** Organizes data into multiple related tables to reduce duplication and improve data consistency.
- **Denormalization:** Combines or duplicates data to simplify queries and improve read performance.

Both approaches are useful in data engineering. The correct choice depends on the workload, data model, and analytical requirements.

### Simple Example

Suppose an e-commerce company stores customer and order information.

A normalized design stores customer details separately from orders.

A denormalized design may store customer details directly alongside every order.

The normalized design reduces repeated customer information, while the denormalized design can reduce the number of joins needed for reporting.

---

## 2. What Is Normalization?

**Normalization** is the process of organizing data into multiple related tables to reduce unnecessary duplication and prevent data anomalies.

Instead of storing all information in one large table, related information is separated into individual tables.

### Example: Before Normalization

Suppose an e-commerce company maintains the following table:

| order_id | customer_id | customer_name | city | product_id | product_name | quantity |
|---|---|---|---|---|---|---:|
| 101 | C01 | Rahul | Kolkata | P01 | Laptop | 1 |
| 102 | C01 | Rahul | Kolkata | P02 | Mouse | 2 |
| 103 | C02 | Priya | Delhi | P01 | Laptop | 1 |

Problems:

- Customer details are repeated across multiple orders.
- Product names are repeated across multiple transactions.
- Updating customer information requires changing multiple rows.
- Inconsistent updates can create incorrect data.

### After Normalization

Separate the information into related tables.

**Customers**

| customer_id | customer_name | city |
|---|---|---|
| C01 | Rahul | Kolkata |
| C02 | Priya | Delhi |

**Products**

| product_id | product_name |
|---|---|
| P01 | Laptop |
| P02 | Mouse |

**Orders**

| order_id | customer_id | product_id | quantity |
|---|---|---|---:|
| 101 | C01 | P01 | 1 |
| 102 | C01 | P02 | 2 |
| 103 | C02 | P01 | 1 |

Now customer and product details are stored separately and referenced through keys.

### Advantages of Normalization

1. Reduces unnecessary data duplication.
2. Improves data consistency.
3. Makes updates easier.
4. Reduces storage requirements in many designs.
5. Helps prevent insertion, update, and deletion anomalies.

### Disadvantages of Normalization

1. Queries may require more joins.
2. Complex analytical queries can become harder to write.
3. Multiple joins may increase query-processing cost.
4. Reporting queries may need additional transformations.

---

## 3. Database Anomalies

Normalization helps prevent three common data anomalies.

### 3.1 Insertion Anomaly

An insertion anomaly occurs when information cannot be inserted without providing unrelated information.

Example:

If customer details and order details are stored in the same table, adding a new customer may require creating an order row even when the customer has not placed an order.

### 3.2 Update Anomaly

An update anomaly occurs when the same information is stored in multiple rows and must be updated consistently.

Example:

If Rahul's city changes from Kolkata to Howrah, every row containing Rahul's details must be updated.

If one row is missed, the database contains inconsistent information.

### 3.3 Deletion Anomaly

A deletion anomaly occurs when deleting one record unintentionally removes other useful information.

Example:

If a customer's only order is deleted from a combined customer-order table, the customer's information may also be lost.

Separating customers and orders helps avoid this problem.

---

## 4. Normal Forms

Normalization is commonly explained through normal forms.

The most frequently discussed forms in interviews are 1NF, 2NF, and 3NF. BCNF is another important form.

### 4.1 First Normal Form (1NF)

A table is in 1NF when:

- Each column contains a single value for each row.
- There are no repeating groups or multi-valued cells.
- Rows can be uniquely identified.

**Example: Not in 1NF**

| student_id | student_name | subjects |
|---|---|---|
| 1 | Rahul | SQL, Python |
| 2 | Priya | Spark, Python |

The `subjects` column contains multiple values.

**After converting to 1NF**

| student_id | student_name | subject |
|---|---|---|
| 1 | Rahul | SQL |
| 1 | Rahul | Python |
| 2 | Priya | Spark |
| 2 | Priya | Python |

Each cell now contains one value.

### 4.2 Second Normal Form (2NF)

A table is in 2NF when:

1. It is already in 1NF.
2. Every non-key attribute depends on the whole candidate key, not just part of a composite candidate key.

This removes partial dependencies.

**Example: Not in 2NF**

Consider a table with a composite primary key `(student_id, course_id)`.

| student_id | course_id | student_name | course_name | marks |
|---|---|---|---|---:|
| 1 | C01 | Rahul | SQL | 85 |
| 1 | C02 | Rahul | Python | 90 |
| 2 | C01 | Priya | SQL | 80 |

Dependencies:

- `student_id → student_name`
- `course_id → course_name`
- `(student_id, course_id) → marks`

`student_name` depends only on `student_id`, and `course_name` depends only on `course_id`. These are partial dependencies.

**After converting to 2NF**

**Students**

| student_id | student_name |
|---|---|
| 1 | Rahul |
| 2 | Priya |

**Courses**

| course_id | course_name |
|---|---|
| C01 | SQL |
| C02 | Python |

**Student_Courses**

| student_id | course_id | marks |
|---|---|---:|
| 1 | C01 | 85 |
| 1 | C02 | 90 |
| 2 | C01 | 80 |

The non-key attribute `marks` depends on the complete composite key.

**Important:** 2NF is especially relevant when a candidate key contains multiple columns. A table with no partial dependencies is in 2NF if it already satisfies 1NF.

### 4.3 Third Normal Form (3NF)

A table is in 3NF when:

1. It is already in 2NF.
2. It has no transitive dependency of non-key attributes on candidate keys.

In simpler terms, non-key attributes should depend on the key, the whole key, and nothing but the key.

**Example: Not in 3NF**

| employee_id | employee_name | department_id | department_name |
|---|---|---|---|
| 101 | Rahul | D01 | Engineering |
| 102 | Priya | D02 | Finance |
| 103 | Amit | D01 | Engineering |

Dependencies:

- `employee_id → employee_name, department_id`
- `department_id → department_name`

Therefore:

`employee_id → department_id → department_name`

Department name depends on department ID rather than directly on the employee key.

**After converting to 3NF**

**Employees**

| employee_id | employee_name | department_id |
|---|---|---|
| 101 | Rahul | D01 |
| 102 | Priya | D02 |
| 103 | Amit | D01 |

**Departments**

| department_id | department_name |
|---|---|
| D01 | Engineering |
| D02 | Finance |

Department details are now stored separately.

### 4.4 Boyce-Codd Normal Form (BCNF)

BCNF is a stronger version of 3NF.

A relation is in BCNF when every determinant is a superkey.

A determinant is an attribute or set of attributes that determines another attribute.

BCNF handles certain dependency situations that can still be permitted by 3NF.

For most entry-level Data Engineer interviews, understand the definition and how BCNF is stronger than 3NF. Detailed decomposition is usually a deeper database-design topic.

### Normal Forms Summary

| Normal Form | Main Rule |
|---|---|
| 1NF | Each cell contains an atomic value |
| 2NF | No partial dependency on part of a candidate key |
| 3NF | No disallowed transitive dependency of non-key attributes |
| BCNF | Every determinant must be a superkey |

---

## 5. What Is Denormalization?

**Denormalization** is the process of intentionally combining or duplicating data to simplify queries and improve read performance.

Instead of always separating information into multiple tables, frequently used information may be stored together.

### Example: Normalized Tables

**Customers**

| customer_id | customer_name | city |
|---|---|---|
| C01 | Rahul | Kolkata |
| C02 | Priya | Delhi |

**Orders**

| order_id | customer_id | amount |
|---|---|---:|
| 101 | C01 | 5000 |
| 102 | C01 | 3000 |
| 103 | C02 | 7000 |

To generate a report with customer names and order amounts, we need a join.

```sql
SELECT
    o.order_id,
    c.customer_name,
    c.city,
    o.amount
FROM orders o
JOIN customers c
    ON o.customer_id = c.customer_id;
```

### Denormalized Table

Customer information is stored directly with each order.

| order_id | customer_id | customer_name | city | amount |
|---|---|---|---|---:|
| 101 | C01 | Rahul | Kolkata | 5000 |
| 102 | C01 | Rahul | Kolkata | 3000 |
| 103 | C02 | Priya | Delhi | 7000 |

Now the reporting query can read from one table without joining the customer table.

### Advantages of Denormalization

1. Reduces the number of joins needed for common queries.
2. Simplifies analytical queries.
3. Can improve read performance for suitable workloads.
4. Makes reporting and dashboard development easier.
5. Can provide business-ready datasets.

### Disadvantages of Denormalization

1. Increases data duplication.
2. Requires more storage in many cases.
3. Updates can become more complex.
4. Duplicated values can become inconsistent.
5. Data pipelines may need additional synchronization and validation.

**Important:** Denormalization does not automatically make every query faster. It is a design trade-off that should be validated through workload testing.

---

## 6. Normalization vs Denormalization

| Aspect | Normalization | Denormalization |
|---|---|---|
| Main objective | Reduce redundancy | Simplify data access |
| Data duplication | Minimized | Intentionally introduced |
| Number of tables | Usually more | Often fewer for reporting |
| Joins | May require more joins | Can reduce joins |
| Storage | Often lower | May be higher |
| Updates | Easier to maintain consistently | Requires synchronization |
| Data consistency | Easier to maintain | Requires careful management |
| Read performance | Depends on query and indexes | Can improve suitable read workloads |
| Write complexity | Often simpler for independent entities | Can be more complex |
| Common usage | OLTP systems | OLAP and reporting models |

---

## 7. Normalization and Denormalization in Data Warehousing

Normalization and denormalization are both used in data warehouse design.

However, dimensional modeling often favors denormalized structures for analytical workloads.

### 7.1 Normalized Data Warehouse Design

A normalized warehouse may organize related entities into multiple tables.

For example:

```text
customers
    |
    v
customer_addresses
    |
    v
orders
    |
    v
order_items
    |
    v
products
```

This reduces duplication but may require multiple joins for analytical queries.

### 7.2 Denormalized Data Warehouse Design

A dimensional model often uses a star schema.

```text
             dim_customer
                  |
                  |
dim_product --- fact_sales --- dim_date
                  |
                  |
               dim_store
```

The fact table stores measurements and dimension keys. Dimension tables contain descriptive attributes, often with related attributes grouped together.

This reduces the number of joins compared with a highly normalized model.

### Example: Star Schema

**fact_sales**

| order_id | customer_key | product_key | date_key | sales_amount |
|---|---:|---:|---:|---:|
| 101 | 1 | 10 | 20261001 | 5000 |
| 102 | 1 | 11 | 20261002 | 3000 |

**dim_customer**

| customer_key | customer_name | city | state |
|---|---|---|---|
| 1 | Rahul | Kolkata | West Bengal |
| 2 | Priya | Delhi | Delhi |

**dim_product**

| product_key | product_name | category |
|---|---|---|
| 10 | Laptop | Electronics |
| 11 | Mouse | Accessories |

The dimension tables are joined to the fact table using surrogate keys.

### Important Distinction

A star schema is not one completely denormalized table.

It contains separate fact and dimension tables. Its dimensions are often denormalized internally, while the fact table stores references to those dimensions.

---

## 8. Star Schema vs Snowflake Schema

These schemas are important examples of denormalized and more normalized dimensional modeling.

### Star Schema

In a star schema, dimension tables are generally denormalized.

Example:

A customer dimension contains customer, city, state, and country attributes in one table.

```text
           dim_customer
                 |
dim_product -- fact_sales -- dim_date
                 |
              dim_store
```

Advantages:

- Simpler queries.
- Fewer joins.
- Easy to understand.
- Commonly used for BI and reporting.

Disadvantages:

- Dimension attributes may be repeated.
- More storage may be required.
- Updates to repeated attributes need careful handling.

### Snowflake Schema

In a snowflake schema, dimensions are normalized into related tables.

Example:

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

Advantages:

- Reduces repeated dimension attributes.
- Can improve consistency of shared hierarchical data.
- Separates complex dimension structures.

Disadvantages:

- More joins.
- More complex queries.
- Can be harder for business users to understand.

### Comparison

| Aspect | Star Schema | Snowflake Schema |
|---|---|---|
| Dimension design | Mostly denormalized | More normalized |
| Joins | Fewer | More |
| Query complexity | Usually simpler | Usually higher |
| Data redundancy | May be higher | Often lower |
| Reporting usability | High | Can be more complex |
| Common use | BI and analytical reporting | More normalized dimensional models |

**Interview point:** Star schema and snowflake schema are both dimensional models. A snowflake schema does not mean the entire warehouse is a normalized OLTP database.

---

## 9. When Should We Use Normalization or Denormalization?

### Use Normalization When:

- The application handles frequent inserts, updates, and deletes.
- Data consistency is important.
- The same entity information is referenced by multiple records.
- Avoiding unnecessary duplication is a priority.
- The system supports transactional workloads.

Example: An order management application.

### Use Denormalization When:

- The system primarily serves analytical queries.
- Queries repeatedly join the same tables.
- Reporting simplicity is important.
- Read performance needs improvement.
- Data can be refreshed or synchronized through controlled pipelines.

Example: A sales dashboard that reads from a business-ready Gold table.

### Practical Data Engineering Approach

A common architecture is:

```text
Source OLTP Database
        |
        v
     Ingestion
        |
        v
   Bronze Layer
        |
        v
   Silver Layer
        |
        v
   Gold Layer
        |
        v
   BI / Reporting
```

A typical approach is:

- Source OLTP systems use normalized transactional models.
- Bronze preserves source data in a raw or lightly processed form.
- Silver cleans, validates, standardizes, and may organize data into reusable entities.
- Gold provides analytical models and aggregates, often using star-schema or denormalized structures.

This is a common pattern, not a mandatory rule. Each layer's design depends on its purpose.

---

## 10. SQL Example: Normalized Tables vs Denormalized Reporting

### Normalized Design

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    city VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    amount DECIMAL(10, 2),
    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);
```

To retrieve customer and order information:

```sql
SELECT
    o.order_id,
    c.customer_name,
    c.city,
    o.amount
FROM orders o
JOIN customers c
    ON o.customer_id = c.customer_id;
```

### Denormalized Reporting Table

A reporting table can store the customer attributes alongside order information.

```sql
CREATE TABLE sales_reporting (
    order_id INT,
    customer_id INT,
    customer_name VARCHAR(100),
    city VARCHAR(100),
    amount DECIMAL(10, 2)
);
```

Reporting query:

```sql
SELECT
    city,
    SUM(amount) AS total_sales
FROM sales_reporting
GROUP BY city;
```

The reporting table avoids the customer join for this query.

**Production consideration:** The reporting table must be refreshed or updated correctly when customer attributes or order data change. It should not become an uncontrolled copy of source data.

---

## 11. Common Interview Mistakes

| Incorrect Understanding | Correct Understanding |
|---|---|
| Normalization always improves query performance | It improves organization and consistency; queries may require more joins |
| Denormalization always makes queries faster | Benefits depend on workload and data layout |
| Star schema is one flat table | It contains fact and dimension tables |
| Snowflake schema is an OLTP database | It is a dimensional model with normalized dimensions |
| 2NF only means removing duplicate rows | It removes partial dependencies on composite candidate keys |
| 3NF means no repeated values anywhere | It removes certain transitive dependencies; repeated values can still exist |
| Denormalization is always bad | It is useful for suitable analytical workloads |
| Normalization is only for relational databases | Normalization is a data-modeling concept applicable to relational design |
| Every Gold table must be denormalized | Gold design depends on analytical requirements |

---

## 12. Interview Questions and Answers

### Q1. What is normalization?

Normalization is the process of organizing data into related tables to reduce redundancy and prevent insertion, update, and deletion anomalies.

### Q2. What is denormalization?

Denormalization intentionally combines or duplicates data to simplify queries and potentially improve read performance.

### Q3. What are the main differences between normalization and denormalization?

Normalization reduces duplication and improves consistency, while denormalization can reduce joins and simplify analytical queries at the cost of additional storage and synchronization.

### Q4. Explain 1NF, 2NF, and 3NF.

- 1NF: Each cell contains an atomic value.
- 2NF: The table is in 1NF and has no partial dependency of non-key attributes on a candidate key.
- 3NF: The table is in 2NF and has no disallowed transitive dependency of non-key attributes on candidate keys.

### Q5. What is a partial dependency?

A partial dependency occurs when a non-key attribute depends on only part of a composite candidate key rather than the entire key.

### Q6. What is a transitive dependency?

A transitive dependency occurs when a non-key attribute depends on another non-key attribute that is itself dependent on a candidate key.

### Q7. What is the difference between star schema and snowflake schema?

A star schema generally has denormalized dimensions connected directly to a fact table. A snowflake schema normalizes dimensions into additional related tables.

### Q8. Why is denormalization commonly used in data warehouses?

Analytical queries often read large amounts of data and repeatedly join dimensions. Denormalized structures can simplify these queries and reduce the number of joins.

### Q9. Does denormalization eliminate all joins?

No. A star schema still joins fact tables to dimension tables. Denormalization reduces some joins, particularly within dimension structures or reporting tables.

### Q10. Can denormalization create data inconsistency?

Yes. When the same attribute is stored in multiple places, updates must be synchronized. Otherwise, different copies can contain different values.

### Q11. Which is better: star schema or snowflake schema?

Neither is universally better. Star schemas are often simpler for reporting, while snowflake schemas can reduce repeated dimension attributes. The choice depends on query patterns, maintainability, and business requirements.

### Q12. How would you design a Gold layer for a sales dashboard?

I would understand the dashboard's analytical requirements, define the fact table grain, identify dimensions and measures, and build a suitable dimensional model. A star schema is often a good starting point for sales reporting.

### Q13. Is a normalized data model always better for transactional systems?

Normalization is commonly preferred for transactional systems because it reduces duplication and helps maintain consistency. However, the final design also depends on workload and performance requirements.

### Q14. What is BCNF?

BCNF is a stronger normal form than 3NF. A relation is in BCNF when every determinant is a superkey.

---

## 13. Quick Revision

| Topic | Key Point |
|---|---|
| Normalization | Reduces unnecessary duplication |
| Denormalization | Combines or duplicates data for simpler reads |
| 1NF | Atomic values |
| 2NF | No partial dependency |
| 3NF | No disallowed transitive dependency |
| BCNF | Every determinant is a superkey |
| Insertion Anomaly | Cannot insert information independently |
| Update Anomaly | Repeated values become inconsistent |
| Deletion Anomaly | Deleting one record removes other useful information |
| Star Schema | Mostly denormalized dimensions |
| Snowflake Schema | More normalized dimensions |
| OLTP | Commonly uses normalized designs |
| OLAP | Often uses dimensional and denormalized structures |
| Gold Layer | Often contains reporting-ready models |

### One-Minute Interview Answer

Normalization organizes data into multiple related tables to reduce redundancy and maintain consistency. It is commonly used in transactional systems where frequent inserts, updates, and deletes happen.

Denormalization combines or duplicates related information to simplify queries and potentially improve read performance. It is commonly used in analytical systems and reporting layers.

For example, in an e-commerce application, customer information can be stored separately from orders in a normalized design. For reporting, customer attributes may be included in a denormalized reporting table to avoid repeatedly joining the customer table.

In data warehousing, star schemas generally use denormalized dimensions, while snowflake schemas normalize dimensions into additional tables.

The choice depends on the workload, query patterns, consistency requirements, and performance needs.

---

## 14. Self-Assessment

- [ ] Explain normalization and denormalization.
- [ ] Identify insertion, update, and deletion anomalies.
- [ ] Explain 1NF, 2NF, 3NF, and BCNF.
- [ ] Identify partial and transitive dependencies.
- [ ] Convert a simple table into a normalized design.
- [ ] Explain why denormalization can help analytical workloads.
- [ ] Compare star schema and snowflake schema.
- [ ] Explain why a star schema is not one flat table.
- [ ] Describe where normalization and denormalization may appear in Bronze, Silver, and Gold layers.
- [ ] Explain the trade-off between joins, storage, consistency, and read performance.

**Interview readiness:** Be able to explain normalization using a customer-order example, describe 1NF through 3NF, and justify why a Data Engineer might create a denormalized Gold reporting table from more normalized source data.