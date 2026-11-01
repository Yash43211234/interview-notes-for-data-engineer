# PySpark Transformation Patterns

This document covers common PySpark DataFrame transformation patterns used in real-world Data Engineering pipelines and technical interviews.

The goal is to understand how to transform raw data into clean, structured, and analysis-ready datasets.

## 1. What Are Transformation Patterns?

Transformation patterns are reusable ways of processing data using Spark DataFrame operations.

Common examples include:

- Filtering unwanted records.
- Selecting and renaming columns.
- Creating derived columns.
- Handling missing values.
- Removing duplicates.
- Joining datasets.
- Aggregating data.
- Applying window functions.
- Flattening nested data.
- Implementing incremental transformations.

A good transformation should be readable, reusable, scalable, and maintainable.

---

## 2. Pattern 1: Select and Rename Columns

**Scenario:** A source system provides columns with inconsistent or unclear names. You need to standardize them.

```python
from pyspark.sql.functions import col

result = df.select(
    col("CustomerID").alias("customer_id"),
    col("CustomerName").alias("customer_name"),
    col("TotalAmount").alias("total_amount")
)
```

**Alternative:**

```python
result = (
    df.withColumnRenamed("CustomerID", "customer_id")
      .withColumnRenamed("CustomerName", "customer_name")
)
```

**When to use:**
- Standardizing source column names.
- Selecting only required fields.
- Preparing data for downstream transformations.

**Interview point:** `select()` can select, rename, and calculate columns in one operation. `withColumnRenamed()` is useful for straightforward renaming.

---

## 3. Pattern 2: Filter Invalid Records

**Scenario:** Remove orders with missing IDs, invalid amounts, or unsuccessful statuses.

```python
from pyspark.sql.functions import col

valid_orders = df.filter(
    col("order_id").isNotNull() &
    col("amount").isNotNull() &
    (col("amount") > 0) &
    (col("status") == "SUCCESS")
)
```

**Why use this pattern?**

- Removes records that do not meet business rules.
- Reduces unnecessary downstream processing.
- Helps maintain data quality.

**Important:** Filtering invalid records is not always the correct business decision. Sometimes invalid records should be quarantined for investigation instead of discarded.

---

## 4. Pattern 3: Create Derived Columns

**Scenario:** Calculate tax and final order amount.

```python
from pyspark.sql.functions import col, round

result = df.withColumn(
    "tax",
    round(col("amount") * 0.18, 2)
).withColumn(
    "final_amount",
    round(col("amount") * 1.18, 2)
)
```

**Common use cases:**

- Tax calculation.
- Profit calculation.
- Discount calculation.
- Derived business metrics.
- Data standardization.

**Interview point:** `withColumn()` returns a new DataFrame. It does not modify the original DataFrame.

---

## 5. Pattern 4: Conditional Transformation

**Scenario:** Categorize customers according to their total purchase amount.

```python
from pyspark.sql.functions import col, when

result = df.withColumn(
    "customer_category",
    when(col("total_amount") >= 100000, "Premium")
    .when(col("total_amount") >= 50000, "Gold")
    .otherwise("Regular")
)
```

**Expected logic:**

| Total Amount | Category |
|---:|---|
| 120000 | Premium |
| 75000 | Gold |
| 30000 | Regular |

**When to use:**
- Business classification.
- Status mapping.
- Conditional calculations.
- Data quality flags.

**Important:** `when()` conditions are evaluated in order. The first matching condition is selected.

---

## 6. Pattern 5: Handle NULL Values

**Scenario:** Replace missing customer city values with `"Unknown"`.

```python
from pyspark.sql.functions import col

result = df.fillna({
    "city": "Unknown",
    "amount": 0
})
```

You can also use `coalesce()` to select the first non-NULL value:

```python
from pyspark.sql.functions import coalesce, col, lit

result = df.withColumn(
    "city",
    coalesce(col("city"), lit("Unknown"))
)
```

**Common operations:**

| Operation | Purpose |
|---|---|
| `isNull()` | Identify NULL values |
| `isNotNull()` | Identify non-NULL values |
| `fillna()` | Replace NULL values |
| `dropna()` | Remove rows containing NULL values |
| `coalesce()` | Return the first non-NULL expression |

**Interview point:** Do not confuse `pyspark.sql.functions.coalesce()` with `DataFrame.coalesce()`. One handles NULL expressions; the other reduces partitions.

---

## 7. Pattern 6: Remove Duplicate Records

**Scenario:** A customer dataset contains repeated customer IDs.

```python
result = df.dropDuplicates([
    "customer_id"
])
```

To remove completely identical rows:

```python
result = df.distinct()
```

**Important:** `dropDuplicates(["customer_id"])` does not guarantee which record is retained when multiple rows have the same customer ID.

If you need to keep the latest record, use a window function.

---

## 8. Pattern 7: Keep the Latest Record

**Scenario:** A customer can have multiple updates. You need only the latest record for each customer.

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import (
    col,
    row_number
)

window_spec = (
    Window
    .partitionBy("customer_id")
    .orderBy(
        col("updated_at").desc(),
        col("record_id").desc()
    )
)

result = (
    df.withColumn(
        "row_num",
        row_number().over(window_spec)
    )
    .filter(col("row_num") == 1)
    .drop("row_num")
)
```

**Why use this pattern?**

- Removes duplicate business keys.
- Retains the latest available record.
- Useful in customer master and dimension tables.

**Important:** Include a deterministic tie-breaker such as `record_id` when timestamps can be equal.

---

## 9. Pattern 8: Group and Aggregate Data

**Scenario:** Calculate total revenue and order count for each customer.

```python
from pyspark.sql.functions import (
    count,
    sum,
    avg
)

result = (
    df.groupBy("customer_id")
      .agg(
          sum("amount").alias("total_revenue"),
          count("order_id").alias("order_count"),
          avg("amount").alias("average_order_value")
      )
)
```

**Common aggregation functions:**

| Function | Purpose |
|---|---|
| `sum()` | Total |
| `count()` | Number of non-NULL values |
| `count("*")` | Number of rows |
| `avg()` | Average |
| `min()` | Minimum |
| `max()` | Maximum |
| `countDistinct()` | Number of distinct values |

**Interview point:** `groupBy()` returns a `GroupedData` object. Use `.agg()` or an aggregation method to perform the actual aggregation.

---

## 10. Pattern 9: Conditional Aggregation

**Scenario:** Calculate successful and failed order counts separately.

```python
from pyspark.sql.functions import (
    col,
    sum,
    when
)

result = (
    df.groupBy("customer_id")
      .agg(
          sum(
              when(col("status") == "SUCCESS", 1)
              .otherwise(0)
          ).alias("successful_orders"),

          sum(
              when(col("status") == "FAILED", 1)
              .otherwise(0)
          ).alias("failed_orders")
      )
)
```

**When to use:**
- Successful versus failed transactions.
- Category-wise revenue.
- Conditional business metrics.
- Reporting pipelines.

---

## 11. Pattern 10: Join Multiple DataFrames

**Scenario:** Combine orders with customer information.

```python
from pyspark.sql.functions import broadcast

result = orders.join(
    broadcast(customers),
    on="customer_id",
    how="left"
)
```

**Common join types:**

| Join | Result |
|---|---|
| Inner | Matching rows from both DataFrames |
| Left | All left rows and matching right rows |
| Right | All right rows and matching left rows |
| Full | All rows from both sides |
| Left Semi | Left rows having a match |
| Left Anti | Left rows without a match |

**Important:** Broadcast only when the smaller side is suitable for broadcasting and the join is supported. Validate duplicate keys because joins can multiply records.

---

## 12. Pattern 11: Compare Current and Previous Records

**Scenario:** Calculate the difference between the current order amount and the previous order amount for each customer.

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import (
    col,
    lag
)

window_spec = (
    Window
    .partitionBy("customer_id")
    .orderBy("order_date", "order_id")
)

result = df.withColumn(
    "previous_amount",
    lag("amount").over(window_spec)
).withColumn(
    "amount_difference",
    col("amount") - col("previous_amount")
)
```

**When to use:**
- Month-over-month comparisons.
- Previous transaction analysis.
- Change detection.
- Customer activity analysis.

---

## 13. Pattern 12: Running Total

**Scenario:** Calculate cumulative order revenue for each customer.

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import (
    col,
    sum
)

window_spec = (
    Window
    .partitionBy("customer_id")
    .orderBy("order_date", "order_id")
    .rowsBetween(
        Window.unboundedPreceding,
        Window.currentRow
    )
)

result = df.withColumn(
    "running_total",
    sum("amount").over(window_spec)
)
```

**Expected behavior:**

| Customer | Amount | Running Total |
|---|---:|---:|
| 101 | 100 | 100 |
| 101 | 200 | 300 |
| 101 | 150 | 450 |

**Important:** The ordering columns should establish the intended row sequence. Adding a unique tie-breaker helps make the result deterministic.

---

## 14. Pattern 13: Union Multiple DataFrames

**Scenario:** Combine January and February sales data.

```python
result = january_df.unionByName(
    february_df
)
```

If columns differ and missing columns should be filled with NULL:

```python
result = january_df.unionByName(
    february_df,
    allowMissingColumns=True
)
```

**Important differences:**

- `union()` combines columns by position.
- `unionByName()` combines columns by name.
- Both retain duplicate rows.
- Use `distinct()` or `dropDuplicates()` separately when deduplication is required.

**When to use:**
- Monthly files.
- Multiple source systems.
- Combining historical and current data.

---

## 15. Pattern 14: Flatten Nested JSON Data

**Scenario:** A DataFrame contains a nested customer structure.

Example schema:

```text
customer
 ├── customer_id
 ├── name
 └── address
      ├── city
      └── country
```

**Transformation:**

```python
from pyspark.sql.functions import col

result = df.select(
    col("customer.customer_id").alias("customer_id"),
    col("customer.name").alias("customer_name"),
    col("customer.address.city").alias("city"),
    col("customer.address.country").alias("country")
)
```

**When to use:**
- Nested JSON.
- API response processing.
- Semi-structured source data.

For arrays, use `explode()` when you need to convert array elements into separate rows.

```python
from pyspark.sql.functions import explode

result = df.select(
    "customer_id",
    explode("orders").alias("order")
)
```

---

## 16. Pattern 15: Explode Arrays

**Scenario:** One customer can have multiple purchased products stored in an array.

Input:

| Customer ID | Products |
|---|---|
| 101 | [Laptop, Mouse] |
| 102 | [Keyboard, Monitor] |

**Transformation:**

```python
from pyspark.sql.functions import explode

result = df.select(
    "customer_id",
    explode("products").alias("product")
)
```

Expected result:

| Customer ID | Product |
|---|---|
| 101 | Laptop |
| 101 | Mouse |
| 102 | Keyboard |
| 102 | Monitor |

**Related functions:**

- `explode()` – creates a row for each array element.
- `explode_outer()` – preserves rows with NULL or empty arrays by producing a NULL element.
- `posexplode()` – also returns the element position.

---

## 17. Pattern 16: Parse and Standardize Dates

**Scenario:** Source data contains dates as strings in `dd-MM-yyyy` format.

```python
from pyspark.sql.functions import (
    col,
    to_date,
    date_format
)

result = (
    df.withColumn(
        "order_date",
        to_date(col("order_date"), "dd-MM-yyyy")
    )
    .withColumn(
        "order_month",
        date_format(col("order_date"), "yyyy-MM")
    )
)
```

**When to use:**
- Standardizing source date fields.
- Creating month and year columns.
- Preparing data for time-based aggregation.

**Important:** Invalid date strings can become NULL or cause errors depending on Spark version and ANSI-related settings. Validate parsing results.

---

## 18. Pattern 17: Incremental Data Filtering

**Scenario:** Process only records updated after the previous successful pipeline run.

```python
from pyspark.sql.functions import col, lit

last_processed_time = "2026-10-01 00:00:00"

incremental_df = df.filter(
    col("updated_at") > lit(last_processed_time).cast("timestamp")
)
```

**Why use this pattern?**

- Avoids processing the entire source repeatedly.
- Reduces processing cost.
- Supports incremental ETL pipelines.

**Important:** Production pipelines need reliable watermark management. Consider late-arriving records, timestamp precision, time zones, and retry behavior. Update the stored watermark only after successful processing.

---

## 19. Pattern 18: Data Quality Flagging

**Scenario:** Identify records with invalid amounts instead of immediately deleting them.

```python
from pyspark.sql.functions import (
    col,
    when
)

result = df.withColumn(
    "quality_status",
    when(
        col("order_id").isNull(),
        "INVALID_ORDER_ID"
    )
    .when(
        col("amount").isNull(),
        "MISSING_AMOUNT"
    )
    .when(
        col("amount") <= 0,
        "INVALID_AMOUNT"
    )
    .otherwise("VALID")
)
```

Separate valid and invalid records:

```python
valid_df = result.filter(
    col("quality_status") == "VALID"
)

invalid_df = result.filter(
    col("quality_status") != "VALID"
)
```

**When to use:**
- Data validation.
- Error handling.
- Quarantine pipelines.
- Bronze-to-Silver transformations.

---

## 20. Pattern 19: Pivot Data

**Scenario:** Convert order status values into separate columns showing their counts.

```python
from pyspark.sql.functions import count

result = (
    df.groupBy("customer_id")
      .pivot(
          "status",
          ["SUCCESS", "FAILED", "PENDING"]
      )
      .agg(count("*"))
)
```

**Expected structure:**

| Customer ID | SUCCESS | FAILED | PENDING |
|---|---:|---:|---:|
| 101 | 5 | 1 | 2 |
| 102 | 3 | 0 | 1 |

**When to use:**
- Reporting.
- Cross-tab analysis.
- Status-wise summaries.

**Important:** Specify pivot values when known to avoid Spark having to discover distinct values from the data.

---

## 21. Pattern 20: Reusable Transformation Functions

**Scenario:** You need to apply the same cleaning logic to multiple source DataFrames.

```python
from pyspark.sql.functions import col

def clean_orders(df):
    return (
        df.filter(
            col("order_id").isNotNull()
        )
        .filter(
            col("amount").isNotNull()
        )
        .filter(
            col("amount") > 0
        )
        .dropDuplicates(["order_id"])
    )

cleaned_orders = clean_orders(orders_df)
cleaned_returns = clean_orders(returns_df)
```

**Why use this pattern?**

- Avoids repeated code.
- Improves maintainability.
- Makes testing easier.
- Encourages consistent business rules.

**Important:** Reusable functions should document assumptions about input schema and business keys. The example assumes `order_id` is the correct deduplication key for both inputs.

---

## 22. End-to-End Transformation Example

**Scenario:** Build a clean customer-level sales dataset.

Requirements:

1. Read order data.
2. Remove invalid orders.
3. Keep successful transactions.
4. Join customer details.
5. Calculate total revenue.
6. Calculate order count.
7. Generate a customer category.

```python
from pyspark.sql.functions import (
    col,
    sum,
    count,
    when,
    broadcast
)

orders = spark.read.parquet(
    "/data/orders"
)

customers = spark.read.parquet(
    "/data/customers"
)

clean_orders = (
    orders
    .filter(
        col("order_id").isNotNull() &
        col("customer_id").isNotNull() &
        col("amount").isNotNull() &
        (col("amount") > 0) &
        (col("status") == "SUCCESS")
    )
    .select(
        "order_id",
        "customer_id",
        "amount"
    )
)

customer_sales = (
    clean_orders
    .groupBy("customer_id")
    .agg(
        sum("amount").alias("total_revenue"),
        count("order_id").alias("order_count")
    )
)

result = (
    customer_sales
    .join(
        customers.select(
            "customer_id",
            "customer_name"
        ),
        "customer_id",
        "left"
    )
    .withColumn(
        "customer_category",
        when(
            col("total_revenue") >= 100000,
            "Premium"
        )
        .when(
            col("total_revenue") >= 50000,
            "Gold"
        )
        .otherwise("Regular")
    )
)

result.show()
```

**Why aggregate before joining?**

The orders are reduced to one row per customer before joining customer details. This can reduce the amount of data shuffled or processed by the join.

However, whether this is faster depends on the data distribution, execution plan, and join strategy.

---

## 23. Common Transformation Mistakes

| Mistake | Better Approach |
|---|---|
| Using Python `and` / `or` in DataFrame filters | Use `&` / `\|` with parentheses |
| Using `collect()` on huge datasets | Use distributed transformations |
| Using `dropDuplicates()` to keep the latest record | Use `row_number()` with ordering |
| Using `union()` when column order differs | Use `unionByName()` |
| Repeatedly writing the same transformation | Create reusable functions |
| Using Python UDFs for simple calculations | Prefer built-in Spark functions |
| Broadcasting a large DataFrame | Check size and memory first |
| Dropping all NULL records blindly | Apply business-specific rules |
| Repartitioning before every operation | Repartition only when beneficial |
| Ignoring duplicate keys before joining | Validate join cardinality |

---

## 24. Interview Questions

**Q1. What is the difference between transformation and action?**

A transformation creates a new DataFrame or RDD. An action triggers execution and returns a result or writes data.

**Q2. Why does PySpark DataFrame transformation use lazy evaluation?**

Spark builds a logical execution plan and can optimize it before executing the required computation.

**Q3. How do you remove duplicates while keeping the latest record?**

Use `row_number()` over a window partitioned by the business key and ordered by the timestamp in descending order, with a deterministic tie-breaker.

**Q4. How do you handle missing values?**

Use `isNull()`, `fillna()`, `dropna()`, or `coalesce()` depending on the business requirement.

**Q5. What is the difference between union and unionByName?**

`union()` combines columns by position, while `unionByName()` matches columns by name.

**Q6. What is explode() used for?**

It converts array elements into separate rows.

**Q7. How do you calculate a running total?**

Use a window specification with `partitionBy()`, `orderBy()`, and an appropriate `rowsBetween()` frame.

**Q8. How do you implement incremental processing?**

Filter records using a stored watermark or last-processed timestamp, then update the watermark after successful processing.

**Q9. Why should you validate data before joining?**

Invalid or duplicate keys can cause incorrect matches, unexpected row multiplication, and inaccurate aggregations.

**Q10. How do you make transformation code reusable?**

Encapsulate common logic in functions, define clear input and output expectations, and test the functions independently.

---

## 25. Quick Revision

| Requirement | PySpark Operation |
|---|---|
| Select columns | `select()` |
| Rename columns | `alias()` / `withColumnRenamed()` |
| Filter records | `filter()` / `where()` |
| Add a column | `withColumn()` |
| Conditional logic | `when()` / `otherwise()` |
| Replace NULL values | `fillna()` |
| Remove NULL rows | `dropna()` |
| Remove duplicates | `distinct()` / `dropDuplicates()` |
| Aggregate records | `groupBy()` / `agg()` |
| Join datasets | `join()` |
| Combine datasets | `unionByName()` |
| Rank records | `row_number()` / `rank()` |
| Previous record | `lag()` |
| Next record | `lead()` |
| Running total | `sum().over()` |
| Flatten arrays | `explode()` |
| Flatten nested structures | Nested column selection |
| Convert strings to dates | `to_date()` |
| Reshape rows into columns | `pivot()` |

---

## 26. Practice Tasks

Try solving these without looking at the examples above.

- [ ] Select only customer ID, order ID, and amount.
- [ ] Remove records where customer ID is NULL.
- [ ] Create a column for 18% tax.
- [ ] Categorize orders as High, Medium, or Low.
- [ ] Replace missing city values with `"Unknown"`.
- [ ] Remove duplicate customer IDs.
- [ ] Keep the latest record for each customer.
- [ ] Calculate total revenue by customer.
- [ ] Calculate successful and failed order counts.
- [ ] Join orders and customers.
- [ ] Calculate the previous order amount.
- [ ] Calculate a running total.
- [ ] Combine two monthly DataFrames.
- [ ] Flatten a nested JSON structure.
- [ ] Explode an array column.
- [ ] Filter records using an incremental timestamp.
- [ ] Create a data quality status column.
- [ ] Pivot order status values into separate columns.

**Final takeaway:** Strong PySpark skills require more than knowing individual functions. You should understand how to combine transformations into reliable, reusable, and efficient data pipelines.