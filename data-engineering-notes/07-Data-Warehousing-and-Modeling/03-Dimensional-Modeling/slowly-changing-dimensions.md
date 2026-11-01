# Slowly Changing Dimensions (SCD)

## 1. Introduction

A **Slowly Changing Dimension (SCD)** is a data warehousing technique used to manage changes in dimension attributes over time.

In real-world systems, dimension information can change.

For example:
- A customer changes their address.
- An employee moves to another department.
- A product changes its category.
- A customer changes their membership level.

The data warehouse needs a strategy to handle these changes.

This is where Slowly Changing Dimensions are used.

### Simple Example

Suppose a customer initially lives in Kolkata and later moves to Bangalore.

**Original customer record:**

| customer_id | customer_name | city |
|---|---|---|
| C001 | Rahul | Kolkata |

After the customer moves:

| customer_id | customer_name | city |
|---|---|---|
| C001 | Rahul | Bangalore |

The question is:

**Should we overwrite the old city, or should we preserve the historical value?**

The answer depends on the business requirement and the SCD strategy.

---

## 2. Why Do We Need Slowly Changing Dimensions?

Dimension tables contain descriptive information about business entities.

When this information changes, the warehouse must decide how to store the changes.

For example, consider a sales report.

A customer placed an order when they lived in Kolkata. Later, they moved to Bangalore.

The business may want to answer:

- What is the customer's current city?
- Where did the customer live when the order was placed?
- How many customers moved between cities?
- Which department did an employee belong to during a previous period?

Without a proper change-management strategy, historical reporting can become inaccurate.

SCD techniques help maintain the required level of historical information.

---

## 3. Important SCD Terminology

Before understanding the different SCD types, understand these terms.

### 3.1 Dimension

A dimension stores descriptive information used to analyze business data.

Examples:
- Customer dimension
- Product dimension
- Employee dimension
- Date dimension
- Location dimension

### 3.2 Business Key (Natural Key)

A business key identifies a real-world entity in the source system.

Example:

`customer_id = C001`

It identifies the customer independently of the warehouse.

### 3.3 Surrogate Key

A surrogate key is a warehouse-generated identifier used to uniquely identify a dimension row.

Example:

| customer_key | customer_id | city |
|---:|---|---|
| 101 | C001 | Kolkata |
| 205 | C001 | Bangalore |

Both rows represent the same customer but different dimension versions.

### 3.4 Historical Tracking

Historical tracking means preserving previous values so the warehouse can reconstruct what was valid at an earlier point in time.

### 3.5 Effective Dates

Effective dates indicate when a particular dimension version is valid.

Common columns:

- `effective_from`
- `effective_to`

### 3.6 Current Flag

A Boolean or status column indicating whether a dimension row represents the current version.

Example:

| customer_key | city | is_current |
|---:|---|---|
| 101 | Kolkata | FALSE |
| 205 | Bangalore | TRUE |

---

## 4. Types of Slowly Changing Dimensions

Several SCD types are commonly discussed in data warehousing.

The most important for Data Engineer interviews are:

1. Type 0 – Retain Original
2. Type 1 – Overwrite
3. Type 2 – Add a New Version
4. Type 3 – Add a Previous-Value Column

Other specialized approaches include Type 4, Type 6, and hybrid designs.

---

## 5. SCD Type 0 – Retain Original

SCD Type 0 keeps the original dimension value unchanged.

Once the value is inserted, subsequent changes are not applied to that attribute.

### Example

Original record:

| customer_id | customer_name | original_city |
|---|---|---|
| C001 | Rahul | Kolkata |

The customer later moves to Bangalore.

Under Type 0, the stored original city remains Kolkata.

| customer_id | customer_name | original_city |
|---|---|---|
| C001 | Rahul | Kolkata |

### Characteristics

- Original values are preserved.
- Changes to protected attributes are ignored.
- No historical versions are created.
- Useful for immutable or first-recorded attributes.

### Common Use Cases

- Original customer registration date.
- Original account creation date.
- Initial purchase channel.
- First recorded acquisition source.

**Key idea:** Never change the original stored value.

---

## 6. SCD Type 1 – Overwrite Existing Data

SCD Type 1 overwrites the existing dimension attribute with the latest value.

It does not preserve the previous value.

### Example

Before the change:

| customer_key | customer_id | city |
|---:|---|---|
| 101 | C001 | Kolkata |

After the change:

| customer_key | customer_id | city |
|---:|---|---|
| 101 | C001 | Bangalore |

The old city is replaced.

### Characteristics

- Existing row is updated.
- No new historical version is created.
- Surrogate key normally remains unchanged.
- Simple to implement using UPDATE or MERGE.

### SQL Example

```sql
UPDATE dim_customer
SET city = 'Bangalore'
WHERE customer_id = 'C001';
```

### Common Use Cases

- Correcting misspelled names.
- Updating standardized descriptions.
- Fixing incorrect contact information.
- Maintaining only the latest value.

**Key idea:** Keep only the latest value.

---

## 7. SCD Type 2 – Add a New Historical Version

SCD Type 2 preserves historical changes by inserting a new dimension row whenever a tracked attribute changes.

The old row is retained, and the new version receives a different surrogate key.

### Example

Before the change:

| customer_key | customer_id | city | is_current |
|---:|---|---|---|
| 101 | C001 | Kolkata | TRUE |

After the customer moves:

| customer_key | customer_id | city | is_current |
|---:|---|---|---|
| 101 | C001 | Kolkata | FALSE |
| 205 | C001 | Bangalore | TRUE |

### Common Columns

- `customer_key`
- `customer_id`
- `city`
- `effective_from`
- `effective_to`
- `is_current`

### Characteristics

- Previous dimension versions are preserved.
- New surrogate key is generated for each version.
- Old version is expired.
- New version is marked current.
- Supports historical point-in-time reporting.

### SQL Example

```sql
-- Expire the old version
UPDATE dim_customer
SET
    effective_to = '2026-07-01',
    is_current = FALSE
WHERE customer_id = 'C001'
  AND is_current = TRUE;

-- Insert the new version
INSERT INTO dim_customer (
    customer_key,
    customer_id,
    city,
    effective_from,
    effective_to,
    is_current
)
VALUES (
    205,
    'C001',
    'Bangalore',
    '2026-07-01',
    NULL,
    TRUE
);
```

The example assumes the warehouse generates a unique surrogate key and uses a consistent effective-date convention. In production, expiration and insertion should be part of a controlled, retry-safe operation.

### Common Use Cases

- Customer address history.
- Employee department history.
- Product category history.
- Customer classification history.
- Organization hierarchy changes.

**Key idea:** Keep every required historical version.

---

## 8. SCD Type 3 – Add a Previous-Value Column

SCD Type 3 stores the current value and a limited amount of previous information in additional columns.

Instead of creating a new row for every change, it adds columns to retain selected previous values.

### Example

Before the change:

| customer_id | current_city | previous_city |
|---|---|---|
| C001 | Kolkata | NULL |

After the customer moves to Bangalore:

| customer_id | current_city | previous_city |
|---|---|---|
| C001 | Bangalore | Kolkata |

### Characteristics

- Existing row is updated.
- Previous value is stored in a separate column.
- Limited historical information is retained.
- No new surrogate key is normally required for each change.

### SQL Example

```sql
UPDATE dim_customer
SET
    previous_city = current_city,
    current_city = 'Bangalore'
WHERE customer_id = 'C001';
```

### Common Use Cases

- Reporting current and previous region.
- Tracking the immediately previous sales territory.
- Comparing current and previous customer categories.

**Key idea:** Keep the current value and a limited previous value.

---

## 9. SCD Type 4 – Historical Data in a Separate Table

SCD Type 4 maintains the current dimension separately from a historical table.

The main dimension contains the current values, while a separate history table stores previous versions.

### Example

**Current Dimension: `dim_customer`**

| customer_id | customer_name | city |
|---|---|---|
| C001 | Rahul | Bangalore |

**History Table: `dim_customer_history`**

| customer_id | customer_name | city | effective_from | effective_to |
|---|---|---|---|---|
| C001 | Rahul | Kolkata | Jan 1 | Jul 1 |

### Characteristics

- Current records are stored separately.
- Historical records are maintained in another table.
- Historical queries require accessing the history table.
- Useful when current and historical access patterns need separation.

**Key idea:** Keep current data and historical data in separate tables.

---

## 10. SCD Type 6 – Hybrid Approach

SCD Type 6 combines characteristics of Type 1, Type 2, and Type 3.

It is designed to support both historical tracking and convenient access to current values.

A common design includes:

- Type 2: Creates a new row for historical versions.
- Type 3: Stores a current-value or previous-value attribute.
- Type 1: Updates selected attributes across existing historical rows.

### Example

Suppose a customer moves from Kolkata to Bangalore.

A simplified Type 6 structure may look like this:

| customer_key | customer_id | historical_city | current_city | effective_from | is_current |
|---:|---|---|---|---|---|
| 101 | C001 | Kolkata | Bangalore | Jan 1 | FALSE |
| 205 | C001 | Bangalore | Bangalore | Jul 1 | TRUE |

Here:
- `historical_city` preserves the city associated with each version.
- `current_city` provides the latest city on all versions.
- New versions are created when tracked historical attributes change.

The exact Type 6 implementation varies. The important concept is combining multiple SCD behaviors.

**Key idea:** Combine Type 1, Type 2, and Type 3 behavior.

---

## 11. Comparison of SCD Types

| SCD Type | Change Handling | Historical Data | New Row? |
|---|---|---|---|
| Type 0 | Retains original value | Original only | No |
| Type 1 | Overwrites existing value | No previous value | No |
| Type 2 | Creates a new version | Full version history | Yes |
| Type 3 | Stores previous value in another column | Limited history | No |
| Type 4 | Stores history in a separate table | History table | Yes, in history table |
| Type 6 | Combines Type 1, 2, and 3 | Historical and current values | Yes |

These are common conceptual patterns. Implementations can vary by warehouse and business requirements.

---

## 12. SCD Type 1 vs Type 2 vs Type 3

This is one of the most important interview comparisons.

| Aspect | Type 1 | Type 2 | Type 3 |
|---|---|---|---|
| Main approach | Overwrite | New row | New column |
| Old value | Lost | Preserved | Limited previous value |
| Surrogate key | Usually unchanged | New key per version | Usually unchanged |
| Row count | Usually unchanged | Increases | Usually unchanged |
| Historical tracking | No | Yes | Limited |
| Complexity | Low | High | Medium |
| Example | Correct a name | Track address history | Store current and previous city |

### Easy Way to Remember

- Type 1 = Overwrite.
- Type 2 = New row.
- Type 3 = New column.

---

## 13. How to Choose the Correct SCD Type

The correct SCD type depends on the business requirement.

| Business Requirement | Suitable Approach |
|---|---|
| Never change the original value | Type 0 |
| Keep only the latest value | Type 1 |
| Preserve complete dimension versions | Type 2 |
| Keep only current and previous values | Type 3 |
| Separate current and historical storage | Type 4 |
| Combine current and historical behavior | Type 6 |

### Practical Examples

**Scenario 1: Customer Name Correction**

A customer's name was misspelled.

Suitable approach: Type 1.

Reason: The warehouse should retain the corrected name rather than the incorrect value.

**Scenario 2: Customer Address History**

The business wants to know where the customer lived when each order was placed.

Suitable approach: Type 2.

Reason: Historical address versions must be preserved.

**Scenario 3: Current and Previous Sales Region**

The business only needs to compare the current sales region with the previous region.

Suitable approach: Type 3.

Reason: Only limited history is required.

**Scenario 4: Original Registration Date**

The original registration date must never change.

Suitable approach: Type 0.

Reason: The original value must be retained.

---

## 14. SCD Type 2 and Fact Table Integration

This is especially important for Data Engineer interviews.

When using SCD Type 2, fact records should reference the correct dimension version using its surrogate key.

### Customer Dimension

| customer_key | customer_id | city | effective_from | effective_to |
|---:|---|---|---|---|
| 101 | C001 | Kolkata | Jan 1 | Jul 1 |
| 205 | C001 | Bangalore | Jul 1 | NULL |

### Sales Fact

| sales_id | customer_key | sales_amount | sales_date |
|---:|---:|---:|---|
| 5001 | 101 | 5000 | Jun 15 |
| 5002 | 205 | 7000 | Aug 10 |

The first sale references the Kolkata version, and the second references the Bangalore version.

### Historical Lookup

```sql
SELECT
    s.sales_id,
    s.sales_amount,
    d.city
FROM staging_sales s
JOIN dim_customer d
    ON s.customer_id = d.customer_id
   AND s.sales_date >= d.effective_from
   AND (
       s.sales_date < d.effective_to
       OR d.effective_to IS NULL
   );
```

This example assumes half-open effective-date intervals:

`effective_from <= event_date < effective_to`

In a production pipeline, resolve the correct surrogate key during fact loading and store it in the fact table.

---

## 15. Implementing SCD in a Data Engineering Pipeline

A typical SCD pipeline can follow these steps:

```text
Source System
     |
     v
Extract Incoming Data
     |
     v
Staging / Bronze Layer
     |
     v
Validate and Deduplicate
     |
     v
Compare with Dimension
     |
     v
Identify New and Changed Records
     |
     v
Apply SCD Logic
     |
     v
Dimension Table
     |
     v
Fact Table Key Resolution
```

### Step-by-Step Explanation

**Step 1: Extract Source Data**

Read customer, employee, product, or other dimension records from the source.

**Step 2: Stage the Data**

Store incoming records in a staging or Bronze layer.

**Step 3: Validate and Deduplicate**

Check business keys, required attributes, and duplicate records.

**Step 4: Compare with Existing Dimension**

Compare incoming records with existing dimension records using the business key.

**Step 5: Identify Record Categories**

Classify records as:
- New records.
- Unchanged records.
- Changed records.

**Step 6: Apply SCD Logic**

Apply the selected strategy:
- Type 1: Update.
- Type 2: Expire and insert.
- Type 3: Update current and previous columns.

**Step 7: Load Fact Tables**

Resolve the correct dimension surrogate keys using the business key and relevant event date.

**Step 8: Validate the Result**

Check:
- One current row per business key when the model requires it.
- No overlapping effective periods.
- Unique surrogate keys.
- Correct historical key resolution.
- Expected row counts and changes.

---

## 16. Important Design Considerations

### 16.1 Choose Attributes Carefully

Different attributes can use different SCD strategies.

For example:

| Attribute | Possible SCD Strategy |
|---|---|
| Customer name correction | Type 1 |
| Customer address | Type 2 |
| Original registration date | Type 0 |
| Current and previous region | Type 3 |

The choice should be based on reporting requirements.

### 16.2 Business Key vs Surrogate Key

The business key identifies the real-world entity.

The surrogate key identifies a particular warehouse dimension row or version.

In Type 2, the same business key can have multiple surrogate keys.

### 16.3 Handle Late-Arriving Changes

A source system may send a change after its actual effective date.

The warehouse may need to create a backdated dimension version and review affected fact-key assignments.

### 16.4 Prevent Duplicate Versions

Before applying SCD logic:
- Deduplicate source records.
- Select the correct source version deterministically.
- Check for an existing current row.
- Make the operation retry-safe.

### 16.5 Maintain Effective-Date Consistency

For Type 2, define a consistent date convention.

A common convention is:

`effective_from <= event_time < effective_to`

This avoids ambiguity at version boundaries.

### 16.6 Handle NULL Values

NULL-safe comparisons are important when dimension attributes can be NULL.

A normal comparison such as:

```sql
target.city <> source.city
```

does not correctly detect every change involving NULL.

Use the database's NULL-safe comparison functionality or explicit NULL-handling conditions.

### 16.7 Consider Fact Table Impact

Type 1 updates can change how historical facts appear when joined to the dimension.

Type 2 requires facts to reference the correct historical dimension version.

---

## 17. Common Interview Mistakes

| Incorrect Understanding | Correct Understanding |
|---|---|
| SCD is only Type 1 and Type 2 | Several types and hybrid patterns exist |
| Type 1 preserves history | Type 1 overwrites previous values |
| Type 2 updates the existing row | Type 2 inserts a new version |
| Type 3 creates a new row | Type 3 stores limited history in additional columns |
| Business key changes for each Type 2 version | Business key usually remains the same |
| Surrogate key is always the same in Type 2 | Each version receives a unique surrogate key |
| Every dimension attribute needs Type 2 | Use historical tracking only where required |
| Current flag alone is sufficient for historical lookup | Effective-date ranges are important |
| Fact tables always reference the latest dimension row | Facts should reference the appropriate version for their event time |

---

## 18. Interview Questions and Answers

### Q1. What is a Slowly Changing Dimension?

It is a data warehousing technique used to manage changes in dimension attributes over time.

### Q2. Why do we need SCD?

We need it to maintain correct dimension information and support current-state or historical reporting based on business requirements.

### Q3. What are the main SCD types?

The commonly discussed types include Type 0, Type 1, Type 2, Type 3, Type 4, and Type 6.

### Q4. What is the difference between Type 1 and Type 2?

Type 1 overwrites existing values and does not preserve history. Type 2 inserts a new dimension version and preserves previous values.

### Q5. What is the difference between Type 2 and Type 3?

Type 2 creates a new row for each version. Type 3 stores limited previous values in additional columns.

### Q6. What is a surrogate key?

A surrogate key is a warehouse-generated identifier used to uniquely identify a dimension row or historical version.

### Q7. Why does SCD Type 2 need effective dates?

Effective dates identify when each version of a dimension record is valid and support historical point-in-time lookups.

### Q8. What is a current flag?

It identifies the active dimension version for a business entity.

### Q9. Can different attributes use different SCD types?

Yes. For example, a customer's address may use Type 2, while a corrected spelling may use Type 1.

### Q10. How does SCD Type 2 affect fact tables?

Fact records should store the surrogate key of the dimension version valid at the time of the business event.

### Q11. How do you implement SCD Type 2 in Databricks?

A common approach uses Delta Lake MERGE with an expire-and-insert pattern. The pipeline must handle source deduplication, surrogate-key generation, retries, and effective dates.

### Q12. What happens when a source record has not changed?

No new Type 2 version is required. The existing current dimension row remains unchanged.

### Q13. What is a late-arriving dimension change?

It is a dimension update received after its actual business-effective date. It may require a backdated version and correction of affected fact references.

### Q14. Why is SCD Type 2 more complex than Type 1?

It requires version management, surrogate-key generation, effective-date handling, historical lookups, and safe updates.

### Q15. How do you decide which SCD type to use?

Understand whether the business needs the original value, latest value, full history, limited previous history, or separate historical storage. Choose the SCD strategy accordingly.

---

## 19. Quick Revision

| Type | Change Handling | Historical Tracking |
|---|---|---|
| Type 0 | Retain original | Original value only |
| Type 1 | Overwrite | No |
| Type 2 | Insert new row | Full version history |
| Type 3 | Add previous-value column | Limited |
| Type 4 | Separate history table | Yes |
| Type 6 | Hybrid approach | Yes, with current-value convenience |

### Most Important Interview Points

1. SCD manages changes in dimension attributes.
2. Type 1 overwrites existing values.
3. Type 2 creates a new row for each tracked version.
4. Type 3 stores limited history in additional columns.
5. Type 2 uses surrogate keys to identify historical versions.
6. Effective dates support point-in-time queries.
7. Fact records should reference the correct Type 2 surrogate key.
8. Different attributes can use different SCD strategies.

### One-Minute Interview Answer

Slowly Changing Dimensions are techniques used in data warehousing to manage changes in dimension attributes over time.

The most important types are Type 1, Type 2, and Type 3.

Type 1 overwrites the existing value and does not preserve history.

Type 2 inserts a new row for every tracked change, using a new surrogate key and commonly maintaining effective dates and a current flag. It supports historical reporting.

Type 3 stores the current value and a limited previous value in additional columns.

We choose the appropriate type based on business requirements. In a Data Engineering pipeline, we identify new and changed records, apply the SCD logic, and ensure fact records reference the correct dimension version.

---

## 20. Self-Assessment

- [ ] Define Slowly Changing Dimensions.
- [ ] Explain why SCD is required in a data warehouse.
- [ ] Explain business keys and surrogate keys.
- [ ] Explain SCD Type 0.
- [ ] Explain SCD Type 1.
- [ ] Explain SCD Type 2.
- [ ] Explain SCD Type 3.
- [ ] Understand the purpose of Type 4 and Type 6.
- [ ] Compare Type 1, Type 2, and Type 3.
- [ ] Design a customer dimension using Type 2.
- [ ] Explain effective-from and effective-to dates.
- [ ] Explain the current flag.
- [ ] Write SQL for Type 1 and Type 2.
- [ ] Explain the Delta Lake MERGE approach.
- [ ] Explain how fact tables resolve historical dimension keys.
- [ ] Handle late-arriving dimension changes.
- [ ] Explain how different attributes can use different SCD types.

**Interview readiness:** Given a changing dimension attribute, identify whether the business needs the original value, latest value, limited history, or complete historical versions. Choose the correct SCD strategy and explain how the pipeline implements it safely.