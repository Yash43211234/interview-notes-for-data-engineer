# Snowflake Schema in Data Warehousing

## 1. Introduction

A **Snowflake Schema** is a dimensional data model in which dimension tables are normalized into multiple related tables.

It is commonly used in data warehouses to organize descriptive data and reduce unnecessary duplication.

The structure resembles a snowflake because dimension tables branch into additional related tables.

### Simple Example

In an e-commerce data warehouse, a customer dimension may contain:

- Customer name
- City
- State
- Country

In a star schema, these attributes may be stored together in one customer dimension table.

In a snowflake schema, they may be separated into related tables:

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

This structure reduces repeated location information but requires additional joins.

---

## 2. What Is a Snowflake Schema?

A snowflake schema is a dimensional modeling approach where:

- A central fact table stores measures and foreign keys.
- Dimension tables store descriptive attributes.
- Some dimensions are normalized into multiple related tables.
- Relationships between dimension tables represent hierarchies.

### Basic Structure

<AsyncImage query="snowflake schema data warehouse dimensional modeling diagram fact table customer product date normalized dimensions" aspectRatio="16:9" width="100%" />

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
dim_category -- dim_product -- fact_sales -- dim_date
                       |
                   dim_brand
```

The fact table is at the center of the analytical model. Dimension tables provide descriptive context, and normalized dimension hierarchies create the snowflake structure.

**Important:** A snowflake schema is still a dimensional model. It is not the same as a fully normalized transactional database.

---

## 3. Main Components

### 3.1 Fact Table

A fact table stores measurable business events and foreign keys that connect to dimensions.

Example: `fact_sales`

| order_id | customer_key | product_key | date_key | sales_amount | quantity |
|---|---:|---:|---:|---:|---:|
| 101 | 1 | 10 | 20261001 | 5000 | 1 |
| 102 | 2 | 11 | 20261002 | 3000 | 2 |
| 103 | 1 | 12 | 20261003 | 2000 | 1 |

Typical measures:

- Sales amount
- Quantity
- Discount
- Profit

The fact table contains foreign keys that connect to the relevant dimension tables.

### 3.2 Dimension Tables

Dimension tables store descriptive information used to filter, group, and analyze facts.

Examples:

- Customer
- Product
- Date
- Store
- Geography

In a snowflake schema, some dimensions are divided into smaller related tables.

### 3.3 Normalized Dimension Tables

Consider a product hierarchy.

Instead of storing product, category, and department details in one table, they can be separated.

```text
dim_department
       |
  dim_category
       |
   dim_product
       |
   fact_sales
```

This allows shared category and department information to be maintained separately.

---

## 4. Practical Example: E-Commerce Snowflake Schema

Suppose an e-commerce company wants to analyze sales by customer, product category, and geographical location.

### Step 1: Create the Fact Table

```sql
CREATE TABLE fact_sales (
    sales_key INT PRIMARY KEY,
    order_id INT,
    customer_key INT,
    product_key INT,
    date_key INT,
    sales_amount DECIMAL(12, 2),
    quantity INT
);
```

### Step 2: Create Customer Dimension

```sql
CREATE TABLE dim_customer (
    customer_key INT PRIMARY KEY,
    customer_id INT,
    customer_name VARCHAR(100),
    city_key INT
);
```

### Step 3: Normalize the Geography Dimension

```sql
CREATE TABLE dim_city (
    city_key INT PRIMARY KEY,
    city_name VARCHAR(100),
    state_key INT
);

CREATE TABLE dim_state (
    state_key INT PRIMARY KEY,
    state_name VARCHAR(100),
    country_key INT
);

CREATE TABLE dim_country (
    country_key INT PRIMARY KEY,
    country_name VARCHAR(100)
);
```

Relationship:

```text
fact_sales
    |
dim_customer
    |
dim_city
    |
dim_state
    |
dim_country
```

### Step 4: Create Product Hierarchy

```sql
CREATE TABLE dim_product (
    product_key INT PRIMARY KEY,
    product_id INT,
    product_name VARCHAR(100),
    category_key INT
);

CREATE TABLE dim_category (
    category_key INT PRIMARY KEY,
    category_name VARCHAR(100),
    department_key INT
);

CREATE TABLE dim_department (
    department_key INT PRIMARY KEY,
    department_name VARCHAR(100)
);
```

Relationship:

```text
fact_sales
    |
dim_product
    |
dim_category
    |
dim_department
```

The product hierarchy is now separated into multiple related tables.

---

## 5. Querying a Snowflake Schema

Suppose the business wants total sales by country and product category.

Because customer geography and product category are normalized, the query requires multiple joins.

```sql
SELECT
    co.country_name,
    ca.category_name,
    SUM(fs.sales_amount) AS total_sales
FROM fact_sales fs
JOIN dim_customer cu
    ON fs.customer_key = cu.customer_key
JOIN dim_city ci
    ON cu.city_key = ci.city_key
JOIN dim_state st
    ON ci.state_key = st.state_key
JOIN dim_country co
    ON st.country_key = co.country_key
JOIN dim_product pr
    ON fs.product_key = pr.product_key
JOIN dim_category ca
    ON pr.category_key = ca.category_key
GROUP BY
    co.country_name,
    ca.category_name
ORDER BY total_sales DESC;
```

### What Happens in This Query?

1. `fact_sales` provides the sales measurements.
2. Customer and geography tables provide country information.
3. Product and category tables provide product classification.
4. `SUM()` calculates total sales.
5. `GROUP BY` aggregates sales by country and category.

**Interview point:** Snowflake schemas can require more joins because dimension hierarchies are split across multiple tables.

---

## 6. Snowflake Schema vs Star Schema

A star schema and a snowflake schema are both dimensional modeling techniques.

The primary difference is how dimension tables are organized.

### Star Schema

In a star schema, dimension attributes are generally stored together in a denormalized dimension table.

```text
              dim_customer
                    |
dim_product -- fact_sales -- dim_date
                    |
                 dim_store
```

Example customer dimension:

| customer_key | customer_name | city | state | country |
|---|---|---|---|---|
| 1 | Rahul | Kolkata | West Bengal | India |
| 2 | Priya | Delhi | Delhi | India |
| 3 | Amit | Mumbai | Maharashtra | India |

### Snowflake Schema

In a snowflake schema, the customer geography attributes are separated into related tables.

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
| Number of dimension tables | Usually fewer | Usually more |
| Joins | Fewer | More |
| Query complexity | Usually simpler | Can be more complex |
| Data redundancy | May be higher | Often lower |
| Storage | May be higher | Can be lower |
| Maintenance | Simpler for many reporting models | Requires managing relationships |
| BI usability | Often easier | May require more modeling knowledge |
| Common use | Reporting and dashboards | Normalized dimensional hierarchies |

### Example

**Star schema:** Customer, city, state, and country are stored in one customer dimension.

**Snowflake schema:** Customer, city, state, and country are stored in separate related tables.

---

## 7. Advantages of Snowflake Schema

### 7.1 Reduces Data Redundancy

Common descriptive attributes are stored once in their respective tables.

For example, the country name does not need to be repeated for every customer in a normalized geography hierarchy.

### 7.2 Improves Data Consistency

When a shared attribute changes, it can be updated in one location rather than in many repeated rows.

### 7.3 Supports Hierarchical Relationships

It can represent hierarchies such as:

```text
Country
   |
 State
   |
  City
   |
Customer
```

And:

```text
Department
    |
 Category
    |
 Product
```

### 7.4 Supports Reusable Dimensions

A normalized hierarchy can be reused by multiple related entities or analytical models where appropriate.

### 7.5 Can Reduce Dimension Storage

Reducing repeated descriptive attributes may lower storage requirements, particularly for large dimensions with repeated hierarchy values.

The actual storage benefit depends on data volume, encoding, compression, and platform.

---

## 8. Disadvantages of Snowflake Schema

### 8.1 More Joins

Queries may need to join several dimension tables to retrieve descriptive attributes.

### 8.2 Complex Queries

Business reporting queries can become longer and harder to maintain.

### 8.3 More Complex BI Modeling

BI developers may need to understand additional relationships and join paths.

### 8.4 Query Performance Can Be Affected

Additional joins may increase query-processing work.

However, more joins do not automatically mean poor performance. Query optimizers, indexing, data layout, caching, and execution engines affect actual performance.

### 8.5 More Complex Data Maintenance

Loading and updating normalized dimension hierarchies requires managing relationships, keys, and dependencies.

---

## 9. When Should We Use a Snowflake Schema?

A snowflake schema may be appropriate when:

- Dimension hierarchies contain substantial repeated attributes.
- Reducing dimension redundancy is important.
- Hierarchical relationships need to be represented separately.
- Shared reference data needs centralized maintenance.
- The organization can manage the additional joins and modeling complexity.

### When a Star Schema May Be Better

Prefer a star schema when:

- Reporting queries should be simple.
- BI users need easy-to-understand dimensions.
- Query patterns benefit from fewer joins.
- Dimension redundancy is acceptable.
- Simplicity and reporting usability are priorities.

**Important:** Choose based on analytical requirements and measured performance, not on the assumption that one schema is always faster.

---

## 10. Snowflake Schema in a Modern Data Engineering Pipeline

A modern data platform may use Bronze, Silver, and Gold layers.

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
BI / Analytics
```

A possible design:

### Bronze Layer

- Stores ingested source data.
- Preserves source information for traceability.
- Applies limited transformations where required.

### Silver Layer

- Cleans and validates data.
- Standardizes entities and attributes.
- May use normalized or reusable entity structures.

### Gold Layer

- Provides business-ready analytical models.
- May use star schemas, snowflake schemas, or aggregate tables.
- Supports reporting and dashboards.

A snowflake schema can be implemented in the Gold layer when normalized dimension hierarchies are useful.

It is not mandatory to use a snowflake schema in every Gold layer.

---

## 11. Snowflake Schema vs Snowflake Platform

These two terms refer to different things.

| Term | Meaning |
|---|---|
| Snowflake Schema | A dimensional data modeling technique |
| Snowflake | A cloud data platform and data warehouse service |

A snowflake schema can be implemented in Snowflake, Databricks, SQL Server, or other systems that support relational tables.

Do not confuse the schema design with the Snowflake platform.

---

## 12. Common Interview Mistakes

| Incorrect Understanding | Correct Understanding |
|---|---|
| Snowflake schema is a fully normalized OLTP database | It is a dimensional model with normalized dimensions |
| It has no fact table | It generally has a central fact table |
| It always performs better than star schema | Performance depends on workload and platform |
| It eliminates all data redundancy | It reduces certain repeated dimension attributes |
| It does not support reporting | It supports reporting, but queries may require more joins |
| Star schema and snowflake schema are unrelated | Both are dimensional modeling approaches |
| More joins always mean slower queries | Actual performance depends on the execution engine and workload |
| Snowflake schema can only be implemented in Snowflake | It can be implemented in many relational data platforms |

---

## 13. Interview Questions and Answers

### Q1. What is a snowflake schema?

A snowflake schema is a dimensional data model in which dimension tables are normalized into multiple related tables around a central fact table.

### Q2. Why is it called a snowflake schema?

It is called a snowflake schema because the normalized dimension tables branch into multiple related tables, creating a structure that resembles a snowflake.

### Q3. What are the main components of a snowflake schema?

The main components are a central fact table, dimension tables, and normalized dimension hierarchies.

### Q4. What is the difference between a star schema and a snowflake schema?

A star schema generally stores descriptive attributes together in denormalized dimensions. A snowflake schema separates dimension attributes into additional related tables.

### Q5. Why do we use a snowflake schema?

It can reduce repeated dimension data, improve consistency, and represent hierarchical relationships clearly.

### Q6. What are the disadvantages of a snowflake schema?

It can require more joins, make queries more complex, and increase the effort needed to maintain relationships.

### Q7. Does a snowflake schema improve query performance?

Not necessarily. It may reduce dimension storage and improve consistency, but additional joins can increase query complexity. Performance depends on the workload and database engine.

### Q8. Can a snowflake schema contain multiple fact tables?

Yes. A dimensional warehouse can contain multiple fact tables, each representing a different business process and grain. They may share conformed dimensions.

### Q9. What is a dimension hierarchy?

A dimension hierarchy represents levels of descriptive information.

For example:

```text
Country → State → City → Customer
```

### Q10. Is a snowflake schema suitable for OLAP?

Yes. It can support OLAP workloads, although star schemas are often preferred when simple reporting queries and BI usability are priorities.

### Q11. What is the relationship between normalization and snowflake schema?

A snowflake schema applies normalization to dimension tables by separating related descriptive attributes into additional tables.

### Q12. How would you explain a snowflake schema using an e-commerce example?

I would use a sales fact table connected to customer and product dimensions. The customer dimension can connect to city, state, and country tables, while the product dimension can connect to category and department tables. This reduces repeated hierarchy information but requires additional joins for reporting.

---

## 14. Quick Revision

| Topic | Key Point |
|---|---|
| Snowflake Schema | Dimensional model with normalized dimensions |
| Fact Table | Stores measurements and dimension keys |
| Dimension Table | Stores descriptive attributes |
| Dimension Hierarchy | Represents relationships between descriptive levels |
| Main Benefit | Reduces repeated dimension attributes |
| Main Trade-off | More joins and query complexity |
| Star Schema | Mostly denormalized dimensions |
| Snowflake Schema | More normalized dimensions |
| OLAP | Can support analytical workloads |
| Gold Layer | Can contain snowflake dimensional models |

### One-Minute Interview Answer

A snowflake schema is a dimensional modeling technique in which dimension tables are normalized into multiple related tables.

For example, in an e-commerce data warehouse, a customer dimension may be separated into customer, city, state, and country tables. Similarly, a product dimension may be separated into product, category, and department tables.

The main advantage is reduced data redundancy and better management of hierarchical attributes. The main disadvantage is that analytical queries may require more joins and become more complex.

Compared with a star schema, a snowflake schema has more normalized dimensions, while a star schema generally keeps descriptive attributes together in fewer dimension tables.

The choice depends on reporting requirements, data maintenance, and query performance.

---

## 15. Self-Assessment

- [ ] Define a snowflake schema.
- [ ] Explain why it is called a snowflake schema.
- [ ] Identify its fact and dimension tables.
- [ ] Explain normalized dimension hierarchies.
- [ ] Design a customer geography hierarchy.
- [ ] Design a product category hierarchy.
- [ ] Write a SQL query joining a fact table to normalized dimensions.
- [ ] Compare star and snowflake schemas.
- [ ] Explain the advantages and disadvantages.
- [ ] Identify when a snowflake schema is appropriate.
- [ ] Explain how it can be used in a Gold layer.
- [ ] Distinguish Snowflake schema from the Snowflake cloud data platform.

**Interview readiness:** Be able to draw a snowflake schema for an e-commerce sales warehouse, explain its dimension hierarchies, and compare its trade-offs with a star schema.