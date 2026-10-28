# Normalization in SQL

## 1. What Is Normalization?

Normalization is the process of organizing data into database tables to reduce duplication and prevent data anomalies.

It typically involves splitting a large table into smaller related tables and defining relationships between them.

**Main objectives:**
- Reduce redundant data.
- Maintain data consistency.
- Prevent insertion, update, and deletion anomalies.
- Make relationships between entities clear.

## 2. Example of an Unnormalized Table

Consider an `orders` table:

| order_id | customer_id | customer_name | customer_city | product |
|---:|---:|---|---|---|
| 101 | 1 | Amit | Delhi | Laptop |
| 102 | 1 | Amit | Delhi | Mouse |
| 103 | 2 | Priya | Mumbai | Keyboard |
| 104 | 1 | Amit | Delhi | Monitor |

Customer information is repeated for every order.

If Amit's city changes, multiple rows must be updated. If one row is missed, the data becomes inconsistent.

Normalization reduces this duplication by separating customer information from order information.

## 3. Data Anomalies

Normalization helps prevent three common anomalies.

### 3.1 Insertion Anomaly

You cannot insert one type of information without inserting unrelated information.

Example: You want to add a new customer, but the table requires an order ID. You cannot store the customer until they place an order.

### 3.2 Update Anomaly

The same information appears in multiple rows, so every occurrence must be updated.

Example: Amit moves from Delhi to Pune. If only some of his rows are updated, the database contains conflicting city values.

### 3.3 Deletion Anomaly

Deleting a record unintentionally removes other information that should have been retained.

Example: If Priya's only order is deleted and her customer details exist only in that order row, her customer information is lost too.

## 4. First Normal Form (1NF)

A table is in First Normal Form when each column contains a single value for each row, and repeating groups are not stored as multiple values in one field.

### Before 1NF

| student_id | student_name | subjects |
|---:|---|---|
| 1 | Amit | SQL, Python |
| 2 | Priya | Python, Spark |

The `subjects` column contains multiple values.

### After 1NF

| student_id | student_name | subject |
|---:|---|---|
| 1 | Amit | SQL |
| 1 | Amit | Python |
| 2 | Priya | Python |
| 2 | Priya | Spark |

Each cell contains one subject.

**Interview point:** 1NF removes repeating groups and ensures values are atomic relative to the database's intended structure. Atomicity depends on the meaning of the data; it does not mean every value must be a single character or word.

## 5. Second Normal Form (2NF)

A table is in Second Normal Form when:

1. It is already in 1NF.
2. Every non-key attribute depends on the whole of every candidate key, not just part of a composite candidate key.

This removes **partial dependencies**.

### Example

Consider an `order_items` table:

| order_id | product_id | product_name | quantity |
|---:|---:|---|---:|
| 101 | 10 | Laptop | 1 |
| 101 | 20 | Mouse | 2 |
| 102 | 10 | Laptop | 1 |

Assume the composite primary key is `(order_id, product_id)`.

Dependencies:

- `(order_id, product_id)` determines `quantity`.
- `product_id` alone determines `product_name`.

Therefore, `product_name` depends only on part of the composite key, violating 2NF.

### After 2NF

**order_items**

| order_id | product_id | quantity |
|---:|---:|---:|
| 101 | 10 | 1 |
| 101 | 20 | 2 |
| 102 | 10 | 1 |

**products**

| product_id | product_name |
|---:|---|
| 10 | Laptop |
| 20 | Mouse |

Product information is now stored separately.

**Important:** A table with a single-column candidate key cannot have a partial dependency on that key. If a table has multiple candidate keys, check the dependencies against all of them.

## 6. Third Normal Form (3NF)

A table is in Third Normal Form when:

1. It is already in 2NF.
2. Non-key attributes do not have inappropriate transitive dependencies on candidate keys.

In simple terms, non-key attributes should depend on the key, the whole key, and nothing but the key.

### Example

Consider an `employees` table:

| employee_id | employee_name | department_id | department_name |
|---:|---|---:|---|
| 101 | Amit | 10 | IT |
| 102 | Priya | 20 | HR |
| 103 | Rahul | 10 | IT |

Assume:

- `employee_id` determines `department_id`.
- `department_id` determines `department_name`.

Therefore, `department_name` depends on `employee_id` indirectly through `department_id`.

This creates a transitive dependency.

### After 3NF

**employees**

| employee_id | employee_name | department_id |
|---:|---|---:|
| 101 | Amit | 10 |
| 102 | Priya | 20 |
| 103 | Rahul | 10 |

**departments**

| department_id | department_name |
|---:|---|
| 10 | IT |
| 20 | HR |

Department names are stored once in the `departments` table.

**Technical note:** The formal definition of 3NF is based on functional dependencies and candidate keys. The simplified rule about transitive dependencies is useful for common interview examples but is not the complete formal definition.

## 7. Boyce-Codd Normal Form (BCNF)

BCNF is a stronger form of 3NF.

A relation is in BCNF if, for every non-trivial functional dependency \(X \rightarrow Y\), \(X\) is a superkey.

A superkey is a set of attributes that uniquely identifies each row.

### Example

Consider a table `teaching_assignments`:

| student_id | course | instructor |
|---:|---|---|
| 1 | SQL | Amit |
| 2 | SQL | Amit |
| 3 | Python | Priya |

Assume these business rules:

- Each instructor teaches only one course.
- A student can have multiple instructors.
- Each student-instructor assignment appears once.

The functional dependency is:

`instructor → course`

However, `instructor` is not a superkey because an instructor can teach multiple students.

This violates BCNF.

### Possible decomposition

**instructors**

| instructor | course |
|---|---|
| Amit | SQL |
| Priya | Python |

**student_instructors**

| student_id | instructor |
|---:|---|
| 1 | Amit |
| 2 | Amit |
| 3 | Priya |

This decomposition removes the dependency that caused the BCNF violation under the stated business rules.

## 8. Summary of Normal Forms

| Normal form | Main requirement | Problem addressed |
|---|---|---|
| 1NF | Atomic values and no repeating groups | Repeating or multi-valued fields |
| 2NF | 1NF and no partial dependency on a candidate key | Partial dependencies |
| 3NF | 2NF and compliance with formal 3NF dependency rules | Certain transitive dependencies |
| BCNF | Every determinant of a non-trivial dependency is a superkey | Remaining determinant-related anomalies |

Normalization is cumulative: a relation must satisfy the preceding normal forms before it can satisfy the next one.

## 9. Normalization vs. Denormalization

**Normalization** separates related information into tables to reduce duplication and anomalies.

**Denormalization** deliberately introduces redundancy or combines data to improve read performance or simplify analytical queries.

| Feature | Normalization | Denormalization |
|---|---|---|
| Data duplication | Generally reduced | May increase |
| Table design | More related tables | May use wider or combined tables |
| Joins | Often more joins | May require fewer joins |
| Data consistency | Easier to maintain with appropriate constraints | Requires careful management of duplicated data |
| Typical use | Transactional systems (OLTP) | Analytical systems and reporting (OLAP), when justified |

Denormalization is not automatically faster. The decision depends on query patterns, data size, update frequency, and database capabilities.

## 10. Normalization in Data Engineering

Normalization is particularly relevant when designing operational relational databases and integrating data from multiple sources.

For data engineering interviews, understand how it affects:

- Relational data modeling.
- Primary and foreign key relationships.
- Data quality and consistency.
- ETL transformations and data integration.
- Analytical warehouse design.

Transactional systems often use normalized structures to maintain consistency. Analytical warehouses often use dimensional models, such as star schemas, which intentionally denormalize dimension data for simpler reporting and efficient analytical queries.

Normalization and dimensional modeling solve related but different problems.

## 11. Common Interview Questions

**Q1. What is normalization?**

Organizing database tables to reduce unnecessary redundancy and prevent data anomalies.

**Q2. What is the difference between 2NF and 3NF?**

2NF removes partial dependencies on candidate keys. 3NF addresses additional functional-dependency problems, commonly explained through transitive dependencies.

**Q3. Can a table be in 2NF but not 3NF?**

Yes. A table may have no partial dependencies but still contain a transitive dependency between non-key attributes.

**Q4. What is the difference between 3NF and BCNF?**

BCNF requires every determinant in a non-trivial functional dependency to be a superkey. 3NF permits certain dependencies where the dependent attribute is prime, meaning it belongs to at least one candidate key.

**Q5. Is normalization always better?**

No. Normalization improves consistency and reduces redundancy, but analytical workloads may benefit from carefully designed denormalization.

**Q6. What is a functional dependency?**

A functional dependency \(X \rightarrow Y\) means that each value of attribute set \(X\) determines exactly one value of attribute set \(Y\).

**Q7. What is a candidate key?**

A minimal set of attributes that uniquely identifies every row in a relation.

**Q8. Why do data warehouses often use denormalized dimensions?**

Denormalized dimensions can reduce the number of joins and make analytical queries easier to write and understand.

## 12. Common Mistakes

- Assuming 1NF means a column must contain only one word.
- Confusing partial dependencies with transitive dependencies.
- Forgetting that 2NF must be evaluated against candidate keys, not only the selected primary key.
- Assuming 3NF and BCNF are identical.
- Assuming normalization eliminates every form of duplication.
- Assuming every data warehouse should use a fully normalized schema.
- Decomposing tables without considering keys, relationships, and whether the original data can be reconstructed correctly.

## 13. Interview Practice

1. Identify insertion, update, and deletion anomalies in a sample table.
2. Convert a table containing multiple phone numbers in one field into 1NF.
3. Identify a partial dependency in a table with a composite key.
4. Decompose an employee table that stores department details repeatedly into 3NF.
5. Explain a functional dependency that violates BCNF.
6. Compare normalized and denormalized schemas for an e-commerce warehouse.
7. Explain why a star schema intentionally stores descriptive attributes together in dimension tables.

## Quick Revision

- **Normalization:** Reduce redundancy and prevent data anomalies.
- **1NF:** Atomic values and no repeating groups.
- **2NF:** No partial dependency of non-prime attributes on a candidate key.
- **3NF:** Satisfy the formal 3NF dependency condition; commonly removes transitive dependencies.
- **BCNF:** Every determinant of a non-trivial functional dependency is a superkey.
- **Denormalization:** Introduce controlled redundancy when it benefits a workload.
- **OLTP:** Often favors normalized relational designs.
- **OLAP:** Often uses dimensional models with deliberate denormalization.

**Core interview takeaway:** Understand functional dependencies and candidate keys first; they explain why a table violates a normal form and how to decompose it correctly.
