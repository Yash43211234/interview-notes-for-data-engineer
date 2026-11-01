# Slowly Changing Dimension (SCD) Type 2

## 1. Introduction

A **Slowly Changing Dimension (SCD)** is a dimensional modeling technique used to manage changes in dimension attributes over time.

For example, a customer's address, department, or product category may change. A data warehouse may need to preserve both the old and new values for historical reporting.

**SCD Type 2** handles changes by inserting a new dimension row for each new version of a record instead of overwriting the existing row.

It preserves historical attribute values.

### Simple Example

Suppose a customer initially lives in Kolkata.

| customer_key | customer_id | customer_name | city |
|---:|---|---|---|
| 101 | C001 | Rahul | Kolkata |

Later, the customer moves to Bangalore.

After applying SCD Type 2:

| customer_key | customer_id | customer_name | city |
|---:|---|---|---|
| 101 | C001 | Rahul | Kolkata |
| 205 | C001 | Rahul | Bangalore |

Notice:

- The business key `customer_id` remains the same.
- A new surrogate key is generated.
- The previous row is retained.
- A new row represents the updated version.

**Key idea:** SCD Type 2 preserves historical changes by maintaining multiple versions of the same business entity.

---

## 2. Why Do We Use SCD Type 2?

Sometimes, the business needs to understand not only the current state of an entity but also its historical state.

For example:

- Where did a customer live when an order was placed?
- Which department did an employee belong to last year?
- What was a product's category when it was sold?
- Which customer segment was applicable at a particular time?

If we overwrite dimension values using SCD Type 1, the old information is lost.

SCD Type 2 solves this by maintaining historical versions.

### Common Use Cases

| Scenario | Example |
|---|---|
| Customer history | Track changes in customer address |
| Employee history | Track department changes |
| Product history | Track category changes |
| Customer segmentation | Track changes in customer classification |
| Organization structure | Track changes in reporting hierarchy |

---

## 3. How SCD Type 2 Works

Consider a customer dimension.

### Initial Data

| customer_key | customer_id | customer_name | city |
|---:|---|---|---|
| 101 | C001 | Rahul | Kolkata |

Suppose Rahul moves to Bangalore.

Instead of updating the existing row, we:

1. Close the old dimension record.
2. Insert a new dimension record.
3. Generate a new surrogate key.
4. Mark the new record as current.

### Recommended Dimension Structure

A typical Type 2 dimension includes:

- `customer_key`: Surrogate key.
- `customer_id`: Business or natural key.
- `city`: Dimension attribute.
- `effective_from`: Date when this version becomes valid.
- `effective_to`: Date when this version stops being valid.
- `is_current`: Indicates whether the row is the current version.

### Before the Change

| customer_key | customer_id | city | effective_from | effective_to | is_current |
|---:|---|---|---|---|---|
| 101 | C001 | Kolkata | Jan 1 | NULL | TRUE |

### After the Change

| customer_key | customer_id | city | effective_from | effective_to | is_current |
|---:|---|---|---|---|---|
| 101 | C001 | Kolkata | Jan 1 | Jun 30 | FALSE |
| 205 | C001 | Bangalore | Jul 1 | NULL | TRUE |

The previous version remains available for historical analysis.

---

## 4. Important Columns in SCD Type 2

### 4.1 Surrogate Key

A warehouse-generated unique identifier for each dimension version.

Example:

| customer_key | customer_id | city |
|---:|---|---|
| 101 | C001 | Kolkata |
| 205 | C001 | Bangalore |

The same business entity has different surrogate keys for different historical versions.

### 4.2 Business Key

An identifier from the source system that identifies the real-world entity.

Example:

`customer_id = C001`

It remains the same across versions of the customer.

### 4.3 Effective From

Indicates when a particular dimension version becomes valid.

Example:

`effective_from = 2026-07-01`

### 4.4 Effective To

Indicates when a dimension version stops being valid.

Example:

`effective_to = 2026-06-30`

For the current record, this is commonly set to NULL or a predefined far-future date.

### 4.5 Current Flag

Indicates which version is currently active.

Example:

| customer_key | city | is_current |
|---:|---|---|
| 101 | Kolkata | FALSE |
| 205 | Bangalore | TRUE |

A current flag is convenient for queries, but the effective dates are important for historical point-in-time analysis.

---

## 5. Step-by-Step SCD Type 2 Process

Suppose the source system sends the following data.

### Existing Dimension

| customer_key | customer_id | customer_name | city | is_current |
|---:|---|---|---|---|
| 101 | C001 | Rahul | Kolkata | TRUE |
| 102 | C002 | Priya | Delhi | TRUE |

### Incoming Source Data

| customer_id | customer_name | city |
|---|---|---|
| C001 | Rahul | Bangalore |
| C002 | Priya | Delhi |
| C003 | Amit | Mumbai |

There are three scenarios:

**Scenario 1: Existing customer with changed attributes**

C001 has changed city from Kolkata to Bangalore.

Action:
- Expire the old row.
- Insert a new version.

**Scenario 2: Existing customer without changes**

C002 still lives in Delhi.

Action:
- Do nothing.

**Scenario 3: New customer**

C003 does not exist in the dimension.

Action:
- Insert a new dimension row.

### Final Dimension

| customer_key | customer_id | customer_name | city | is_current |
|---:|---|---|---|---|
| 101 | C001 | Rahul | Kolkata | FALSE |
| 103 | C001 | Rahul | Bangalore | TRUE |
| 102 | C002 | Priya | Delhi | TRUE |
| 104 | C003 | Amit | Mumbai | TRUE |

This example shows how SCD Type 2 maintains old and new versions.

---

## 6. SQL Implementation of SCD Type 2

Suppose we have:

- `dim_customer`: Existing dimension table.
- `stg_customer`: Incoming source data.

The following example uses a two-step process: expire changed records, then insert new versions.

Assumptions:
- The source has one deduplicated row per `customer_id`.
- The source provides a valid `effective_from` timestamp.
- The target generates surrogate keys using an appropriate database mechanism.
- The dimension uses NULL as the end date for current rows.

### Step 1: Expire Existing Records

```sql
UPDATE dim_customer AS target
SET
    effective_to = source.effective_from,
    is_current = FALSE
FROM stg_customer AS source
WHERE target.customer_id = source.customer_id
  AND target.is_current = TRUE
  AND (
      target.customer_name <> source.customer_name
      OR target.city <> source.city
  );
```

This closes the previous version for customers whose tracked attributes have changed.

The example assumes the compared attributes are non-NULL. Production logic should use NULL-safe comparisons where needed.

### Step 2: Insert New Versions

```sql
INSERT INTO dim_customer (
    customer_id,
    customer_name,
    city,
    effective_from,
    effective_to,
    is_current
)
SELECT
    source.customer_id,
    source.customer_name,
    source.city,
    source.effective_from,
    NULL,
    TRUE
FROM stg_customer AS source
LEFT JOIN dim_customer AS target
    ON source.customer_id = target.customer_id
   AND target.is_current = TRUE
WHERE target.customer_id IS NULL;
```

**Important:** The two statements must be executed as one controlled transaction or an equivalent retry-safe pipeline operation. Otherwise, a failure between expiring and inserting could leave an entity without a current row.

The exact SQL syntax for `UPDATE ... FROM`, transaction handling, and surrogate-key generation varies by database platform.

---

## 7. SCD Type 2 Using Delta Lake MERGE

SCD Type 2 is commonly implemented in Databricks using Delta Lake MERGE.

A useful approach is to create two source record types:

- One record that matches the current target row and expires it when attributes have changed.
- Another record that does not match and therefore inserts the new version.

### Example PySpark Implementation

Assume:

- `source_df` contains the latest customer records.
- `dim_customer` is an existing Delta dimension table.
- The target has a generated or otherwise managed surrogate key.
- `effective_from` is provided by the source or ingestion process.

### Step 1: Prepare the Source

```python
from pyspark.sql import functions as F

source_df = source_df.dropDuplicates(
    ["customer_id"]
)
```

This is only a simplified preparation step. `dropDuplicates()` does not guarantee which duplicate is retained. Select the correct source record deterministically using an update timestamp and a tie-breaker.

### Step 2: Identify Changed Records

```python
from delta.tables import DeltaTable

target = DeltaTable.forName(
    spark,
    "dim_customer"
)

current_df = (
    target.toDF()
    .filter(F.col("is_current") == True)
)

changed_df = (
    source_df.alias("s")
    .join(
        current_df.alias("t"),
        on="customer_id",
        how="inner"
    )
    .filter(
        ~F.col("s.customer_name").eqNullSafe(
            F.col("t.customer_name")
        )
        |
        ~F.col("s.city").eqNullSafe(
            F.col("t.city")
        )
    )
)
```

This identifies customers whose tracked attributes have changed. The source and target columns are compared using NULL-safe equality.

### Step 3: Create Expiration and Insertion Records

```python
# Records that match the current row and expire it
expire_df = (
    changed_df
    .select(
        "customer_id",
        "customer_name",
        "city",
        "effective_from"
    )
    .withColumn(
        "merge_key",
        F.col("customer_id")
    )
    .withColumn(
        "record_action",
        F.lit("EXPIRE")
    )
)

# Records that must be inserted as new versions
insert_df = (
    source_df
    .select(
        "customer_id",
        "customer_name",
        "city",
        "effective_from"
    )
    .withColumn(
        "merge_key",
        F.lit(None).cast("string")
    )
    .withColumn(
        "record_action",
        F.lit("INSERT")
    )
)

merge_source = expire_df.unionByName(
    insert_df
)
```

The NULL merge key ensures that the insertion records do not match existing target rows.

### Step 4: Apply MERGE

```python
target.alias("t").merge(
    merge_source.alias("s"),
    """
    t.customer_id = s.merge_key
    AND t.is_current = true
    """
).whenMatchedUpdate(
    condition="s.record_action = 'EXPIRE'",
    set={
        "effective_to": "s.effective_from",
        "is_current": "false"
    }
).whenNotMatchedInsert(
    condition="s.record_action = 'INSERT'",
    values={
        "customer_id": "s.customer_id",
        "customer_name": "s.customer_name",
        "city": "s.city",
        "effective_from": "s.effective_from",
        "effective_to": "CAST(NULL AS TIMESTAMP)",
        "is_current": "true"
    }
).execute()
```

This demonstrates the common expire-and-insert MERGE pattern.

**Production considerations:**

- The example assumes `effective_from` is a timestamp and the target's effective-date columns use compatible types.
- The target must generate or otherwise populate a unique surrogate key for inserted versions.
- Ensure the source contains only one appropriate record per business key for the batch.
- New customers are inserted because their source records have no current target match.
- Existing unchanged customers are not expired or reinserted.
- Handle multiple changes for the same customer in one batch using a deterministic ordering strategy.
- Test retry behavior and concurrent writes before production deployment.

---

## 8. Querying Historical Data

One of the main advantages of SCD Type 2 is the ability to retrieve dimension attributes as they were valid at a particular time.

### Example: Find Customer's City on a Specific Date

Suppose we want to know where customer C001 lived on June 15, 2026.

```sql
SELECT
    customer_id,
    customer_name,
    city
FROM dim_customer
WHERE customer_id = 'C001'
  AND effective_from <= '2026-06-15'
  AND (
      effective_to > '2026-06-15'
      OR effective_to IS NULL
  );
```

This query assumes effective periods use a half-open interval:

`effective_from <= date < effective_to`

The date boundary convention must be consistent throughout the dimension.

### Find Current Customer Records

```sql
SELECT
    customer_id,
    customer_name,
    city
FROM dim_customer
WHERE is_current = TRUE;
```

### Find Customer Change History

```sql
SELECT
    customer_id,
    city,
    effective_from,
    effective_to,
    is_current
FROM dim_customer
WHERE customer_id = 'C001'
ORDER BY effective_from;
```

---

## 9. How Fact Tables Work with SCD Type 2

This is an important interview concept.

Fact tables generally store the surrogate key of the dimension version that was valid when the fact occurred.

### Customer Dimension

| customer_key | customer_id | city | effective_from | effective_to |
|---:|---|---|---|---|
| 101 | C001 | Kolkata | Jan 1 | Jul 1 |
| 205 | C001 | Bangalore | Jul 1 | NULL |

### Sales Fact Table

| sales_id | customer_key | sales_amount | sales_date |
|---:|---:|---:|---|
| 5001 | 101 | 5000 | Jun 15 |
| 5002 | 205 | 7000 | Aug 10 |

The first sale references the Kolkata version, while the second sale references the Bangalore version.

### Why Is This Important?

When a fact record arrives, the pipeline must resolve the correct dimension surrogate key using the fact's business key and event date.

A typical lookup condition is:

```sql
SELECT
    f.sales_id,
    f.sales_amount,
    d.city
FROM staging_sales f
JOIN dim_customer d
    ON f.customer_id = d.customer_id
   AND f.sales_date >= d.effective_from
   AND (
       f.sales_date < d.effective_to
       OR d.effective_to IS NULL
   );
```

The example uses half-open effective-date intervals. In a production pipeline, resolve the surrogate key and store it in the fact table rather than repeatedly joining on the business key and dates.

**Important:** Late-arriving facts may need to resolve against an older dimension version. If a dimension change is backdated, existing fact foreign keys may also need correction depending on the historical reporting policy.

---

## 10. SCD Type 1 vs SCD Type 2

| Aspect | SCD Type 1 | SCD Type 2 |
|---|---|---|
| Change handling | Overwrites existing value | Inserts a new version |
| Historical values | Not preserved | Preserved |
| Surrogate key | Normally unchanged | New key for each version |
| Row count | Usually unchanged for updates | Increases when new versions are added |
| Effective dates | Not required | Commonly maintained |
| Current flag | Not required | Commonly maintained |
| Complexity | Simple | More complex |
| Use case | Correcting a misspelled name | Tracking customer address history |

### Simple Difference

**Type 1:** Keep only the latest value.

**Type 2:** Keep both previous and current values as separate dimension versions.

---

## 11. Advantages of SCD Type 2

1. **Historical tracking:** Previous attribute values are preserved.
2. **Accurate historical reporting:** Reports can use the dimension version valid at the time of the event.
3. **Auditability:** Changes can be traced through effective dates and version records.
4. **Trend analysis:** Business changes can be analyzed over time.
5. **Flexible reporting:** Supports both current-state and historical-state analysis.

## 12. Disadvantages of SCD Type 2

1. **More rows:** Each tracked change creates another dimension version.
2. **Complex implementation:** Requires expiration and insertion logic.
3. **Surrogate key management:** Every version needs a unique key.
4. **Fact key resolution:** Facts must reference the correct historical dimension version.
5. **Late-arriving data:** Backdated changes require careful handling.
6. **Data quality risks:** Overlapping or missing effective periods can cause incorrect historical lookups.

---

## 13. Important Design Considerations

### 13.1 Define the Attributes to Track

Not every dimension attribute needs Type 2 history.

For example:

| Attribute | Possible Strategy |
|---|---|
| Customer address | Type 2 |
| Customer name correction | Type 1 |
| Customer phone number | Type 1 or Type 2, based on requirements |
| Product category | Type 2 if historical reporting is required |
| Load timestamp | Operational metadata, not automatically an SCD attribute |

The choice should follow business reporting requirements.

### 13.2 Prevent Overlapping Effective Periods

For one business key, two current or overlapping versions can cause a historical lookup to return multiple rows.

Ensure the dimension has a valid timeline.

### 13.3 Use a Consistent Date Convention

A common approach is a half-open interval:

`effective_from <= event_time < effective_to`

For the current version, `effective_to` can be NULL.

This avoids ambiguity at transition boundaries.

### 13.4 Handle Late-Arriving Changes

Suppose the customer moved on July 1, but the source system sends the change on July 10.

The warehouse may need to record July 1 as the effective date rather than July 10, depending on the business definition.

If historical facts are already loaded, decide whether their surrogate key assignments need correction.

### 13.5 Ensure Source Deduplication

If the source contains multiple records for the same business key, select the correct record or process the changes in chronological order.

Do not rely on arbitrary duplicate removal.

### 13.6 Make Pipeline Execution Retry-Safe

A failed pipeline may be retried.

The logic must prevent duplicate dimension versions and ensure the old version is not expired repeatedly with incorrect dates.

Use deterministic source ordering, controlled MERGE conditions, uniqueness checks, and appropriate transaction handling.

---

## 14. Common Interview Mistakes

| Incorrect Understanding | Correct Understanding |
|---|---|
| Type 2 overwrites the existing row | It retains the old row and inserts a new version |
| Business key changes for each version | The business key usually remains the same |
| Surrogate key remains the same | Each dimension version receives a unique surrogate key |
| Type 2 only needs a current flag | Effective dates are important for historical point-in-time queries |
| Every attribute must use Type 2 | Track only attributes requiring historical preservation |
| Historical facts always use the current dimension row | Facts should reference the dimension version valid at the event time |
| Late-arriving changes can be ignored | They may require backdated versions and fact-key correction |
| MERGE automatically prevents all duplicate versions | Source uniqueness, concurrency, and retry logic still matter |

---

## 15. Interview Questions and Answers

### Q1. What is SCD Type 2?

SCD Type 2 is a technique that preserves historical dimension changes by inserting a new row for each new version instead of overwriting the existing row.

### Q2. Why do we use SCD Type 2?

We use it when historical attribute values are required for reporting, auditing, or trend analysis.

### Q3. What is the difference between a business key and a surrogate key?

A business key identifies the real-world entity, such as `customer_id`. A surrogate key is a warehouse-generated identifier for a particular dimension version.

### Q4. Why does SCD Type 2 need a new surrogate key?

Each historical version must be uniquely identifiable so fact records can reference the correct version.

### Q5. What are effective dates?

They indicate the period during which a dimension version is valid.

### Q6. What is a current flag?

It indicates which dimension version is currently active for a business entity.

### Q7. How do you implement SCD Type 2 in SQL?

Expire the existing current row and insert a new row containing the changed attributes, new surrogate key, and effective dates.

### Q8. How do you implement SCD Type 2 in Databricks?

A common approach is to use Delta Lake MERGE with an expire-and-insert source pattern. Matching records are expired, and unmatched insertion records create new versions.

### Q9. How do fact tables use SCD Type 2 dimensions?

Fact records store the surrogate key of the dimension version valid at the time of the business event.

### Q10. What happens if a customer's address changes?

The existing customer dimension row is expired, and a new row is inserted with the new address and a new surrogate key.

### Q11. How do you find the customer's historical address?

Use the business key and event date to find the dimension version whose effective date range contains that date.

### Q12. What is a late-arriving dimension change?

It occurs when a dimension change reaches the warehouse after its actual business-effective date. The pipeline may need to create a backdated version and review affected fact-key assignments.

### Q13. What happens if effective date ranges overlap?

A point-in-time lookup may return multiple dimension versions, causing incorrect joins or duplicated fact results. The dimension timeline must be validated.

### Q14. Can SCD Type 1 and Type 2 be used together?

Yes. Different attributes can follow different change policies. For example, customer address can use Type 2, while a spelling correction can use Type 1.

### Q15. What is the biggest challenge in SCD Type 2?

Maintaining correct historical versions and ensuring every fact record references the correct dimension version, especially with late-arriving data and pipeline retries.

---

## 16. Quick Revision

| Topic | Key Point |
|---|---|
| SCD | Technique for managing dimension changes |
| Type 2 | Inserts a new version when tracked attributes change |
| Historical Data | Preserved |
| Business Key | Usually remains unchanged |
| Surrogate Key | New key for each version |
| Effective From | Start of version validity |
| Effective To | End of version validity |
| Current Flag | Identifies active version |
| SQL Implementation | Expire old row + insert new row |
| Databricks Implementation | Delta Lake MERGE |
| Fact Table | Stores the correct dimension version's surrogate key |
| Main Advantage | Historical reporting |
| Main Challenge | Version and key management |

### One-Minute Interview Answer

SCD Type 2 is a slowly changing dimension technique used to preserve historical changes.

For example, if a customer moves from Kolkata to Bangalore, we do not overwrite the existing dimension row. Instead, we expire the old Kolkata record and insert a new Bangalore record with a new surrogate key.

We commonly maintain effective-from, effective-to, and current-flag columns to track the validity of each version.

When loading facts, we resolve the correct dimension surrogate key based on the business key and event date. This allows historical reports to show the customer's attributes as they were valid when the event occurred.

In Databricks, this can be implemented using Delta Lake MERGE with an expire-and-insert pattern.

---

## 17. Self-Assessment

- [ ] Define Slowly Changing Dimensions.
- [ ] Explain SCD Type 2.
- [ ] Explain why historical tracking is required.
- [ ] Differentiate business keys and surrogate keys.
- [ ] Explain effective-from and effective-to dates.
- [ ] Explain the current flag.
- [ ] Design a customer dimension with historical versions.
- [ ] Write SQL logic to expire and insert dimension versions.
- [ ] Explain the Delta Lake MERGE approach.
- [ ] Explain how fact tables reference historical dimension versions.
- [ ] Write a point-in-time dimension lookup.
- [ ] Explain late-arriving dimension changes.
- [ ] Explain how to prevent overlapping effective periods.
- [ ] Explain retry safety and source deduplication.
- [ ] Compare SCD Type 1 and Type 2.

**Interview readiness:** Given a changing dimension attribute, decide whether history must be preserved, design the version columns, implement the expire-and-insert process, and explain how fact records resolve the correct historical surrogate key.