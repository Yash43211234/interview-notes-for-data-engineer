# Bridge Tables in Dimensional Modeling

## 1. Introduction

A **Bridge Table** is a table used in dimensional modeling to handle relationships that cannot be represented accurately using a simple one-to-one or many-to-one relationship between a fact table and a dimension.

It is especially useful for handling **many-to-many relationships** between facts and dimensions.

### Simple Example: Customers and Accounts

Suppose one customer can have multiple bank accounts, and one bank account can be jointly owned by multiple customers.

This creates a many-to-many relationship:

- One customer can be associated with multiple accounts.
- One account can be associated with multiple customers.

A direct relationship between customer and account dimensions may create duplication or incorrect reporting.

A bridge table solves this problem.

```text
Customer Dimension
        |
        |
        v
   Bridge Table
        ^
        |
        |
Account Dimension
```

**Key idea:** A bridge table connects entities when a direct relationship cannot represent the business relationship correctly.

---

## 2. Why Do We Need Bridge Tables?

In a dimensional model, fact tables usually connect to dimensions through foreign keys.

A simple relationship often looks like this:

```text
Dimension  ----  Fact Table
```

However, some business scenarios involve many-to-many relationships.

Examples:

| Business Scenario | Many-to-Many Relationship |
|---|---|
| Bank accounts | Customers and accounts |
| Healthcare | Patients and doctors |
| Sales | Sales transactions and salespeople |
| Insurance | Policies and beneficiaries |
| Projects | Employees and projects |
| Marketing | Customers and campaigns |

A bridge table represents these relationships without forcing an incorrect one-to-many design.

---

## 3. Basic Structure of a Bridge Table

Consider two dimensions:

### Customer Dimension

| customer_key | customer_name |
|---:|---|
| 101 | Rahul |
| 102 | Priya |
| 103 | Amit |

### Account Dimension

| account_key | account_number |
|---:|---|
| 501 | ACC001 |
| 502 | ACC002 |

### Customer-Account Bridge Table

| customer_key | account_key |
|---:|---:|
| 101 | 501 |
| 102 | 501 |
| 101 | 502 |
| 103 | 502 |

This means:

- Rahul is associated with ACC001 and ACC002.
- Priya is associated with ACC001.
- Amit is associated with ACC002.

The bridge table stores the relationship between customers and accounts.

### Relationship Diagram

```text
dim_customer
     |
     | customer_key
     |
     v
bridge_customer_account
     |
     | account_key
     |
     v
dim_account
```

The bridge table typically contains the keys of the related entities. It can also contain relationship attributes such as ownership percentage or effective dates.

---

## 4. Bridge Tables in Fact and Dimension Relationships

Bridge tables are also used when one fact record can be associated with multiple dimension members.

### Example: Sales Transaction and Salespeople

Suppose one sales transaction is handled by two salespeople.

A simple fact table with one salesperson key cannot represent both salespeople without either duplicating the transaction or losing information.

A bridge table can represent the relationship.

### Sales Fact Table

| sales_id | amount |
|---:|---:|
| 1001 | 10000 |
| 1002 | 8000 |

### Salesperson Dimension

| salesperson_key | salesperson_name |
|---:|---|
| 11 | Rahul |
| 12 | Priya |
| 13 | Amit |

### Salesperson Bridge Table

| sales_id | salesperson_key | allocation_pct |
|---:|---:|---:|
| 1001 | 11 | 60 |
| 1001 | 12 | 40 |
| 1002 | 13 | 100 |

The bridge table represents which salespeople participated in each sale and their allocation percentages.

**Important:** The allocation percentage is useful for distributing measures such as sales revenue without counting the full transaction amount for every associated salesperson.

---

## 5. Allocation Factors and Double Counting

One of the most important concepts when working with bridge tables is **double counting**.

Suppose a sale of ₹10,000 is associated with two salespeople.

| sales_id | salesperson | sales_amount |
|---:|---|---:|
| 1001 | Rahul | 10000 |
| 1001 | Priya | 10000 |

If we simply join the fact table with the bridge table and sum the sales amount, the result becomes ₹20,000.

But the actual sale amount is ₹10,000.

### Correct Approach: Allocation Percentage

| sales_id | salesperson | allocation_pct |
|---:|---|---:|
| 1001 | Rahul | 60 |
| 1001 | Priya | 40 |

Allocated sales:

- Rahul: ₹10,000 × 60% = ₹6,000
- Priya: ₹10,000 × 40% = ₹4,000

Total allocated sales = ₹10,000.

### SQL Example

```sql
SELECT
    b.salesperson_key,
    SUM(f.sales_amount * b.allocation_pct / 100.0)
        AS allocated_sales
FROM fact_sales f
JOIN bridge_salesperson b
    ON f.sales_id = b.sales_id
GROUP BY b.salesperson_key;
```

The allocation percentages should follow the business rules. If the purpose is to report full transaction amounts by participant rather than allocate revenue, the report must clearly explain that totals across participants can exceed the original sales total.

---

## 6. Bridge Tables for Multivalued Dimensions

A common dimensional modeling use case is a dimension with multiple values for one entity.

### Example: Customer and Multiple Skills

Suppose a customer can have multiple interests or preferences, or an employee can have multiple skills.

A single dimension row with one skill key cannot naturally represent multiple skills.

A bridge table can store the relationship.

### Employee Dimension

| employee_key | employee_name |
|---:|---|
| 101 | Rahul |
| 102 | Priya |

### Skill Dimension

| skill_key | skill_name |
|---:|---|
| 1 | SQL |
| 2 | Python |
| 3 | Spark |

### Employee-Skill Bridge

| employee_key | skill_key |
|---:|---:|
| 101 | 1 |
| 101 | 2 |
| 101 | 3 |
| 102 | 1 |
| 102 | 2 |

This allows one employee to have multiple skills.

It also allows one skill to be associated with multiple employees.

---

## 7. Bridge Tables and Slowly Changing Dimensions

Bridge tables may also be used with time-dependent relationships.

For example, an employee may belong to different departments over time.

A bridge table can contain effective dates to represent when a relationship is valid.

Example:

| employee_key | department_key | effective_from | effective_to |
|---:|---:|---|---|
| 101 | 10 | Jan 1 | Jun 30 |
| 101 | 20 | Jul 1 | NULL |

This represents the employee's department relationship over time.

However, time-dependent relationships require careful design:

- Define whether dates are inclusive or exclusive.
- Avoid overlapping effective periods unless the business allows them.
- Decide how historical fact records should resolve to the correct relationship.
- Consider whether a Type 2 dimension alone is sufficient.

A bridge table with effective dates is often called a **time-variant bridge** or **effective-dated bridge**.

---

## 8. SQL Example: Creating and Querying a Bridge Table

### Create Dimension Tables

```sql
CREATE TABLE dim_customer (
    customer_key INT PRIMARY KEY,
    customer_name VARCHAR(100)
);

CREATE TABLE dim_account (
    account_key INT PRIMARY KEY,
    account_number VARCHAR(30)
);
```

### Create the Bridge Table

```sql
CREATE TABLE bridge_customer_account (
    customer_key INT,
    account_key INT,
    ownership_percentage DECIMAL(5, 2),
    PRIMARY KEY (customer_key, account_key),
    FOREIGN KEY (customer_key)
        REFERENCES dim_customer(customer_key),
    FOREIGN KEY (account_key)
        REFERENCES dim_account(account_key)
);
```

The composite primary key prevents the same customer-account relationship from being inserted more than once. Additional columns may be needed if the relationship itself has multiple versions over time.

### Find All Accounts for Each Customer

```sql
SELECT
    c.customer_name,
    a.account_number
FROM bridge_customer_account b
JOIN dim_customer c
    ON b.customer_key = c.customer_key
JOIN dim_account a
    ON b.account_key = a.account_key;
```

### Find Customers Who Share an Account

```sql
SELECT
    a.account_number,
    COUNT(DISTINCT b.customer_key) AS customer_count
FROM bridge_customer_account b
JOIN dim_account a
    ON b.account_key = a.account_key
GROUP BY a.account_number
HAVING COUNT(DISTINCT b.customer_key) > 1;
```

### Calculate Account Ownership by Customer

```sql
SELECT
    c.customer_name,
    a.account_number,
    b.ownership_percentage
FROM bridge_customer_account b
JOIN dim_customer c
    ON b.customer_key = c.customer_key
JOIN dim_account a
    ON b.account_key = a.account_key;
```

---

## 9. Bridge Table vs Direct Foreign Key

| Aspect | Direct Foreign Key | Bridge Table |
|---|---|---|
| Relationship | Usually one-to-many | Many-to-many |
| Structure | Key stored directly in fact/dimension | Separate relationship table |
| Complexity | Simpler | More joins and relationship logic |
| Multiple associations | Limited by chosen grain and columns | Naturally supports multiple associations |
| Example | One customer per order | Multiple salespeople per sale |

A bridge table should be introduced when the business relationship requires it, not simply to add another table to the model.

---

## 10. Bridge Table vs Factless Fact Table

These concepts are related but not identical.

### Bridge Table

A bridge table represents a relationship between entities or between facts and dimensions.

Example:

| employee_key | skill_key |
|---:|---:|
| 101 | 1 |
| 101 | 2 |

### Factless Fact Table

A factless fact table records that an event or relationship occurred, even when there are no numeric measures.

Example: Student attendance.

| student_key | class_key | date_key |
|---:|---:|---:|
| 101 | 10 | 20261001 |
| 102 | 10 | 20261001 |

### Key Difference

- A bridge table primarily resolves or represents complex relationships.
- A factless fact table records events or coverage without numeric measures.

A bridge table may have no numeric measures, but that does not automatically make every bridge table a factless fact table. The purpose and grain determine its role.

---

## 11. Challenges and Design Considerations

### 11.1 Double Counting

Joining a fact table to a bridge can multiply fact rows.

**Solution:** Use allocation factors when measures need to be distributed, or aggregate at the correct grain before joining.

### 11.2 Duplicate Relationships

The same pair of entities may accidentally appear multiple times.

**Solution:** Use appropriate uniqueness constraints or deduplication rules.

### 11.3 Time-Dependent Relationships

A relationship may change over time.

**Solution:** Store effective dates and define how facts resolve to the correct historical relationship.

### 11.4 Query Complexity

Bridge tables introduce additional joins.

**Solution:** Document the relationship and provide reusable views or semantic models where appropriate.

### 11.5 Allocation Rules

Some relationships require weighted allocation; others do not.

**Solution:** Define the allocation method with business stakeholders. Do not assume equal allocation unless the business requires it.

### 11.6 Performance

Large bridge tables can increase join costs.

**Solution:** Use appropriate keys, statistics, filtering, and query plans. Optimize based on actual workload rather than adding indexes or partitions without measurement.

---

## 12. Common Interview Mistakes

| Incorrect Understanding | Correct Understanding |
|---|---|
| Bridge tables are only for joining two tables | They are commonly used to represent many-to-many relationships |
| Every bridge needs allocation percentages | Allocation factors are needed only when business measures must be distributed |
| Bridge tables always connect two dimensions | They can also connect fact records to multiple dimension members |
| Joining a bridge never changes row counts | A join can multiply fact rows |
| Bridge tables and factless fact tables are identical | Their modeling purposes are different |
| A bridge automatically handles historical relationships | Effective dates and historical resolution must be designed |
| Duplicate bridge rows are harmless | They can distort joins and aggregates |

---

## 13. Interview Questions and Answers

### Q1. What is a bridge table?

A bridge table is a dimensional modeling structure used to represent many-to-many relationships between entities or between fact records and dimension members.

### Q2. Why do we need a bridge table?

We use it when one entity can be associated with multiple members of another entity and a direct foreign key cannot represent the relationship correctly.

### Q3. Give a real-world example.

A sales transaction may involve multiple salespeople. A bridge table connects the transaction to each salesperson and can store each person's allocation percentage.

### Q4. What is an allocation factor?

An allocation factor is a percentage or weight used to distribute a measure across multiple related entities without unintentionally duplicating the total.

### Q5. How does a bridge table cause double counting?

If one fact row joins to multiple bridge rows, its measures are repeated in the joined result. Summing those repeated values can inflate totals.

### Q6. How can you prevent double counting?

Use appropriate allocation factors, aggregate at the correct grain, or report distinct transaction totals separately from participant-level totals.

### Q7. Can a bridge table contain effective dates?

Yes. Effective dates allow the bridge to represent relationships that change over time.

### Q8. What is a multivalued dimension?

A dimension in which one entity can have multiple values or associations, such as an employee having multiple skills.

### Q9. Is a bridge table the same as a factless fact table?

No. A bridge primarily represents relationships, while a factless fact table records events or coverage without numeric measures.

### Q10. Can a bridge table connect a fact table to a dimension?

Yes. For example, a sales fact can connect to multiple salespeople through a bridge table.

### Q11. What should you consider when designing a bridge table?

Define its grain, relationship keys, uniqueness rules, effective dates if needed, allocation logic, and how downstream queries should avoid double counting.

### Q12. When should you avoid using a bridge table?

Avoid it when a simple foreign-key relationship accurately represents the business relationship. Unnecessary bridges increase query complexity.

---

## 14. Quick Revision

| Topic | Key Point |
|---|---|
| Bridge Table | Represents complex relationships |
| Common Use | Many-to-many relationships |
| Multivalued Dimension | One entity has multiple dimension members |
| Allocation Factor | Distributes measures across related entities |
| Double Counting | Can occur when fact rows multiply during joins |
| Effective Dates | Represent time-dependent relationships |
| Factless Fact Table | Records events or coverage without numeric measures |
| Main Benefit | Accurate representation of complex relationships |
| Main Challenge | Join complexity and measure duplication |

### One-Minute Interview Answer

A bridge table is used in dimensional modeling to handle many-to-many relationships.

For example, one sales transaction may involve multiple salespeople. A normal fact-to-dimension foreign key may not represent all participants correctly.

We create a bridge table containing the sales transaction key and salesperson key. We can also store an allocation percentage to distribute the sales amount among the salespeople.

One important challenge is double counting. When a fact row joins to multiple bridge rows, its measures may be repeated. We handle this through allocation factors or by aggregating at the correct grain.

Bridge tables can also represent multivalued dimensions and time-dependent relationships.

---

## 15. Self-Assessment

- [ ] Define a bridge table.
- [ ] Explain why many-to-many relationships need special modeling.
- [ ] Design a customer-account bridge.
- [ ] Design a sales transaction-salesperson bridge.
- [ ] Explain allocation percentages.
- [ ] Demonstrate how a bridge join can cause double counting.
- [ ] Write SQL to query a bridge table.
- [ ] Explain multivalued dimensions.
- [ ] Explain effective-dated bridges.
- [ ] Compare bridge tables and factless fact tables.
- [ ] Identify the correct grain of a bridge table.
- [ ] Explain how to avoid duplicate relationships.

**Interview readiness:** Given a many-to-many business relationship, identify the entities, define the bridge grain, choose relationship keys, decide whether allocation is required, and explain how to prevent double counting.