# Slowly Changing Dimension (SCD) Type 1

## 1. Introduction

A **Slowly Changing Dimension (SCD)** is a dimensional modeling technique used to manage changes in dimension attributes over time.

For example, a customer's address, phone number, or email address may change.

When these attributes change, the data warehouse needs a strategy to handle the updated information.

**SCD Type 1** handles changes by overwriting the existing dimension attribute with the new value.

It does not preserve the previous value.

### Simple Example

Suppose a customer initially lives in Kolkata.

| customer_key | customer_id | customer_name | city |
|---:|---|---|---|
| 101 | C001 | Rahul | Kolkata |

Later, the customer moves to Bangalore.

After applying SCD Type 1:

| customer_key | customer_id | customer_name | city |
|---:|---|---|---|
| 101 | C001 | Rahul | Bangalore |

The old city value, Kolkata, is replaced by Bangalore.

**Key idea:** SCD Type 1 overwrites existing dimension data and keeps only the latest value.

---

## 2. Why Do We Use SCD Type 1?

Dimension attributes can change because of:

- Customer address corrections.
- Typographical errors.
- Updated contact information.
- Product description corrections.
- Changes to department names.
- Standardization of inconsistent source values.

Sometimes, the business only needs the latest correct information and does not need historical values.

SCD Type 1 is suitable for these situations.

### Common Use Cases

| Scenario | Example |
|---|---|
| Data correction | Fixing a misspelled customer name |
| Address update | Replacing an old address |
| Contact update | Updating a phone number |
| Product description | Correcting a product name |
| Standardization | Changing `KOLKATA` to `Kolkata` |

---

## 3. How SCD Type 1 Works

Consider a customer dimension.

### Initial Data

| customer_key | customer_id | customer_name | city |
|---:|---|---|---|
| 101 | C001 | Rahul | Kolkata |
| 102 | C002 | Priya | Delhi |

### Source Data After Changes

| customer_id | customer_name | city |
|---|---|---|
| C001 | Rahul | Bangalore |
| C002 | Priya | Delhi |

Customer C001 has moved from Kolkata to Bangalore.

### After SCD Type 1

| customer_key | customer_id | customer_name | city |
|---:|---|---|---|
| 101 | C001 | Rahul | Bangalore |
| 102 | C002 | Priya | Delhi |

Notice:

- The existing `customer_key` remains unchanged.
- The business key `customer_id` remains unchanged.
- The changed attribute is overwritten.
- No new historical dimension row is created.

---

## 4. SQL Implementation

Suppose we have two tables:

- `dim_customer`: Existing dimension table.
- `stg_customer`: Incoming source data.

### Existing Dimension Table

| customer_key | customer_id | customer_name | city |
|---:|---|---|---|
| 101 | C001 | Rahul | Kolkata |
| 102 | C002 | Priya | Delhi |

### Staging Table

| customer_id | customer_name | city |
|---|---|---|
| C001 | Rahul | Bangalore |
| C002 | Priya | Delhi |
| C003 | Amit | Mumbai |

The incoming data contains:
- C001: Existing customer with a changed city.
- C002: Existing customer with no changes.
- C003: New customer.

### MERGE Statement

```sql
MERGE INTO dim_customer AS target
USING stg_customer AS source
ON target.customer_id = source.customer_id

WHEN MATCHED THEN
    UPDATE SET
        target.customer_name = source.customer_name,
        target.city = source.city

WHEN NOT MATCHED THEN
    INSERT (
        customer_key,
        customer_id,
        customer_name,
        city
    )
    VALUES (
        source.customer_key,
        source.customer_id,
        source.customer_name,
        source.city
    );
```

This is an illustrative MERGE pattern. The source must provide a valid surrogate key for new customers, or the target platform must generate one through an appropriate mechanism. The exact syntax for key generation and MERGE varies by database.

### Result

| customer_key | customer_id | customer_name | city |
|---:|---|---|---|
| 101 | C001 | Rahul | Bangalore |
| 102 | C002 | Priya | Delhi |
| 103 | C003 | Amit | Mumbai |

The existing customer's city is updated, while the new customer is inserted.

**Important:** Before MERGE, ensure the source has at most one row per business key. Duplicate source matches can cause ambiguous updates or runtime errors.

---

## 5. PySpark Implementation

SCD Type 1 can also be implemented using PySpark and Delta Lake.

Suppose we have:

- `source_df`: Incoming customer data.
- `dim_customer`: Existing Delta dimension table.

### Step 1: Read the Existing Dimension

```python
from delta.tables import DeltaTable

target = DeltaTable.forName(
    spark,
    "dim_customer"
)
```

### Step 2: Prepare the Source

```python
source_df = source_df.dropDuplicates(
    ["customer_id"]
)
```

`dropDuplicates()` removes duplicate keys, but it does not guarantee which duplicate record is retained.

For production pipelines, select the correct record deterministically, for example, using the latest source update timestamp and a tie-breaker.

### Step 3: Apply MERGE

```python
target.alias("target").merge(
    source_df.alias("source"),
    "target.customer_id = source.customer_id"
).whenMatchedUpdate(
    set={
        "customer_name": "source.customer_name",
        "city": "source.city"
    }
).whenNotMatchedInsert(
    values={
        "customer_id": "source.customer_id",
        "customer_name": "source.customer_name",
        "city": "source.city"
    }
).execute()
```

This illustrates the SCD Type 1 update-and-insert pattern.

The example assumes the target table generates or otherwise supplies `customer_key` for new records. Configure that mechanism according to the target table design.

---

## 6. Should We Update Every Matched Record?

Not necessarily.

A more efficient approach is to update only when an attribute has actually changed.

For example:

```sql
MERGE INTO dim_customer AS target
USING stg_customer AS source
ON target.customer_id = source.customer_id

WHEN MATCHED
    AND (
        target.customer_name <> source.customer_name
        OR target.city <> source.city
    )
THEN UPDATE SET
    target.customer_name = source.customer_name,
    target.city = source.city;
```

This example assumes the compared columns are non-NULL. If NULL values are possible, use the database's NULL-safe comparison operator or explicit NULL-handling logic.

Why is this useful?

- Avoids unnecessary updates.
- Reduces unnecessary writes.
- Makes change tracking easier.
- Can reduce transaction-log activity.

For Delta Lake, MERGE behavior and performance depend on the source size, target layout, and runtime.

---

## 7. SCD Type 1 and Surrogate Keys

SCD Type 1 normally retains the existing dimension surrogate key when a dimension attribute changes.

Example:

### Before Update

| customer_key | customer_id | city |
|---:|---|---|
| 101 | C001 | Kolkata |

### After Update

| customer_key | customer_id | city |
|---:|---|---|
| 101 | C001 | Bangalore |

The surrogate key remains `101`.

This is different from SCD Type 2, where a changed historical version usually receives a new surrogate key.

### Why Does This Matter?

Fact tables commonly reference dimension surrogate keys.

If the SCD Type 1 dimension row is updated, existing fact records that reference the same surrogate key will resolve to the updated dimension attribute when queried.

For example, historical sales records for customer C001 may now display Bangalore rather than Kolkata.

This is expected behavior in Type 1 because historical attribute values are intentionally not preserved.

---

## 8. Advantages of SCD Type 1

1. **Simple implementation:** Easy to implement using UPDATE or MERGE.
2. **Latest information:** The dimension always reflects the latest stored value.
3. **No historical row growth:** Updates do not create additional dimension versions.
4. **Useful for corrections:** Suitable for correcting inaccurate or misspelled data.
5. **Simpler reporting:** No need to resolve multiple historical versions.

## 9. Disadvantages of SCD Type 1

1. **No historical tracking:** Previous attribute values are lost.
2. **Historical reports can change:** Old fact records may display the new dimension value.
3. **Limited trend analysis:** Cannot analyze how an attribute changed over time using the dimension alone.
4. **Not suitable for every attribute:** Business requirements may require historical values.

---

## 10. SCD Type 1 vs SCD Type 2

This is one of the most frequently asked SCD interview comparisons.

| Aspect | SCD Type 1 | SCD Type 2 |
|---|---|---|
| Change handling | Overwrites existing value | Creates a new version |
| Historical values | Not preserved | Preserved |
| Surrogate key | Usually remains the same | New key for each version |
| Row count | Usually unchanged for updates | Increases for new versions |
| Effective dates | Not required | Commonly maintained |
| Current flag | Not required | Commonly maintained |
| Implementation | Simpler | More complex |
| Example | Correcting a customer name | Tracking customer address history |

### Example

**SCD Type 1**

| customer_key | customer_id | city |
|---:|---|---|
| 101 | C001 | Bangalore |

Only the latest city is retained.

**SCD Type 2**

| customer_key | customer_id | city | effective_from | effective_to | is_current |
|---:|---|---|---|---|---|
| 101 | C001 | Kolkata | Jan 1 | Jun 30 | FALSE |
| 205 | C001 | Bangalore | Jul 1 | NULL | TRUE |

Both historical and current versions are retained.

---

## 11. When Should You Use SCD Type 1?

Use SCD Type 1 when:

- The business needs only the latest value.
- Historical changes are not important.
- You are correcting inaccurate source data.
- A descriptive attribute should always show its current standardized value.
- Historical reporting based on the old value is not required.

Avoid Type 1 when the business needs to answer questions such as:

- What was the customer's address when the sale happened?
- Which department did an employee belong to last year?
- What was the product category at the time of a transaction?

Those requirements generally need historical tracking, such as SCD Type 2 or another history model.

---

## 12. Common Interview Mistakes

| Incorrect Understanding | Correct Understanding |
|---|---|
| Type 1 creates a new row for every change | It overwrites the existing attribute |
| Type 1 preserves historical values | Previous values are lost in the dimension |
| Surrogate key changes after every update | It normally remains unchanged |
| Type 1 is suitable for all dimension attributes | Use it when historical tracking is unnecessary |
| Type 1 updates only existing records | New business keys can also be inserted |
| Historical fact records remain unchanged in meaning | They may display the latest dimension attributes |
| MERGE automatically handles duplicate source keys correctly | Source keys should be unique before MERGE |

---

## 13. Interview Questions and Answers

### Q1. What is SCD Type 1?

SCD Type 1 overwrites an existing dimension attribute with a new value without maintaining historical versions.

### Q2. Does SCD Type 1 preserve historical data?

No. The old dimension attribute value is replaced by the new value.

### Q3. Does the surrogate key change in SCD Type 1?

Normally, no. The existing surrogate key is retained when an attribute is updated.

### Q4. Can SCD Type 1 insert new records?

Yes. New business keys are inserted, while matching existing business keys are updated when required.

### Q5. When would you use SCD Type 1?

For corrections, standardization, and attributes where only the latest value matters.

### Q6. What happens to historical fact records after a Type 1 update?

Their foreign keys remain the same, but joins to the dimension may display the updated attribute value.

### Q7. How do you implement SCD Type 1 in SQL?

Use an UPDATE or MERGE operation to overwrite changed attributes and insert new dimension records.

### Q8. How do you implement SCD Type 1 in Databricks?

Use Delta Lake MERGE to update matching records and insert new records. Ensure the source is deduplicated by its business key.

### Q9. What is the main disadvantage of SCD Type 1?

It does not preserve previous attribute values, so historical analysis of those changes is not possible from the Type 1 dimension alone.

### Q10. Can Type 1 and Type 2 be used together?

Yes. Different attributes can follow different change-handling policies. For example, a correction to a misspelled name may use Type 1, while customer address history may use Type 2.

### Q11. What is the difference between a business key and a surrogate key in SCD Type 1?

The business key identifies the real-world entity, such as `customer_id`. The surrogate key is the warehouse-generated identifier, such as `customer_key`, used to connect fact records to the dimension.

### Q12. Why might you update only changed records?

It avoids unnecessary writes and can improve pipeline efficiency, especially for large dimension tables.

---

## 14. Quick Revision

| Topic | Key Point |
|---|---|
| SCD | Technique for managing dimension changes |
| Type 1 | Overwrites existing attribute |
| Historical Data | Not preserved |
| Surrogate Key | Normally remains unchanged |
| New Record | Inserted |
| Existing Record | Updated when required |
| SQL Implementation | UPDATE or MERGE |
| Databricks Implementation | Delta Lake MERGE |
| Main Advantage | Simple and keeps latest values |
| Main Disadvantage | Loses previous values |

### One-Minute Interview Answer

SCD Type 1 is a slowly changing dimension technique where we overwrite the existing dimension attribute with the new value.

For example, if a customer moves from Kolkata to Bangalore, we update the customer's city in the existing dimension row instead of creating a new row.

The surrogate key normally remains unchanged, and historical attribute values are not preserved.

We commonly implement it using SQL MERGE or Delta Lake MERGE in Databricks. It is useful for corrections and attributes where only the latest value matters.

---

## 15. Self-Assessment

- [ ] Define Slowly Changing Dimensions.
- [ ] Explain SCD Type 1.
- [ ] Explain how Type 1 handles changed attributes.
- [ ] Identify situations where Type 1 is appropriate.
- [ ] Explain what happens to the surrogate key.
- [ ] Write SQL MERGE for Type 1.
- [ ] Explain Type 1 implementation using PySpark and Delta Lake.
- [ ] Explain why source deduplication matters.
- [ ] Explain how historical fact reports are affected.
- [ ] Compare SCD Type 1 and Type 2.
- [ ] Explain when Type 1 and Type 2 can be combined.

**Interview readiness:** Given a dimension attribute change, decide whether historical tracking is required. If it is not, explain how to update the existing row safely using a business key and an idempotent MERGE operation.