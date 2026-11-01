# Null Handling in PySpark

## 1. Overview

Null handling is the process of identifying, replacing, removing, or managing missing values in a DataFrame.

In real-world data engineering, incoming data often contains missing or incomplete values because of:

- Missing source-system information.
- Failed data ingestion.
- Optional fields.
- Incorrect data entry.
- Missing values from joins.
- Incomplete API or CSV records.

PySpark provides built-in functions to handle null values during data cleaning and transformation.

Import the required functions:

```python
from pyspark.sql import functions as F
```

---

## 2. Understanding Null Values

A null value represents missing or unknown data.

Consider this DataFrame:

```python
data = [
    (1, "Alice", 5000),
    (2, "Bob", None),
    (3, None, 3000),
    (4, "David", None)
]

df = spark.createDataFrame(
    data,
    ["id", "name", "salary"]
)
```

| id | name | salary |
|---:|---|---:|
| 1 | Alice | 5000 |
| 2 | Bob | null |
| 3 | null | 3000 |
| 4 | David | null |

**Important:** Null is different from zero, an empty string, or a string containing spaces.

| Value | Meaning |
|---|---|
| `NULL` | Missing or unknown value |
| `0` | Numeric zero |
| `""` | Empty string |
| `" "` | String containing a space |

---

## 3. Identifying Null Values

### 3.1 isNull()

Checks whether a column contains null values.

```python
df.filter(
    F.col("salary").isNull()
).show()
```

### 3.2 isNotNull()

Checks whether a column does not contain null values.

```python
df.filter(
    F.col("salary").isNotNull()
).show()
```

### 3.3 Count Null Values in a Column

```python
df.filter(
    F.col("salary").isNull()
).count()
```

### 3.4 Count Null Values in Multiple Columns

```python
df.select(
    F.count(
        F.when(F.col("name").isNull(), 1)
    ).alias("null_names"),

    F.count(
        F.when(F.col("salary").isNull(), 1)
    ).alias("null_salaries")
).show()
```

### 3.5 Check Null Values Across All Columns

```python
df.select([
    F.count(
        F.when(F.col(c).isNull(), c)
    ).alias(c)
    for c in df.columns
]).show()
```

This gives the null count for every column.

---

## 4. Removing Null Values with dropna()

`dropna()` removes rows containing null values.

### 4.1 Drop Rows with Any Null Value

```python
df.dropna().show()
```

This removes a row if at least one column contains null.

### 4.2 Drop Rows Only When All Values Are Null

```python
df.dropna(how="all").show()
```

This removes a row only when all relevant columns are null.

### 4.3 Drop Rows Based on Specific Columns

```python
df.dropna(
    subset=["name", "salary"]
).show()
```

This removes rows where either `name` or `salary` is null.

### 4.4 Drop Rows Based on a Threshold

```python
df.dropna(
    thresh=2
).show()
```

`thresh=2` means a row must contain at least two non-null values to be retained.

**Important:** `dropna()` returns a new DataFrame. It does not modify the original DataFrame in place.

---

## 5. Replacing Null Values with fillna()

`fillna()` replaces null values with a specified value.

### 5.1 Replace Null in One Column

```python
df.fillna(
    0,
    subset=["salary"]
).show()
```

### 5.2 Replace Null in Multiple Columns

```python
df.fillna(
    {
        "name": "Unknown",
        "salary": 0
    }
).show()
```

### 5.3 Replace Null Using Different Data Types

```python
df = df.fillna({
    "name": "Not Available",
    "salary": 0
})
```

**Important:** Use replacement values compatible with the column's data type.

---

## 6. Conditional Null Handling with when() and otherwise()

Use `when()` when replacement depends on a condition.

Example:

```python
df = df.withColumn(
    "salary",
    F.when(
        F.col("salary").isNull(),
        0
    ).otherwise(F.col("salary"))
)
```

Another example: Replace missing names with `"Unknown"`.

```python
df = df.withColumn(
    "name",
    F.when(
        F.col("name").isNull(),
        "Unknown"
    ).otherwise(F.col("name"))
)
```

You can also create a new column to identify missing values:

```python
df = df.withColumn(
    "salary_status",
    F.when(
        F.col("salary").isNull(),
        "Missing"
    ).otherwise("Available")
)
```

---

## 7. coalesce()

`coalesce()` returns the first non-null value from a list of expressions.

Example:

```python
df = df.withColumn(
    "final_salary",
    F.coalesce(
        F.col("salary"),
        F.lit(0)
    )
)
```

If `salary` is null, the result is 0.

### Multiple Columns

Suppose salary information is available from two sources.

```python
df = df.withColumn(
    "final_salary",
    F.coalesce(
        F.col("primary_salary"),
        F.col("secondary_salary"),
        F.lit(0)
    )
)
```

The function checks values from left to right and returns the first non-null value.

**Important:** `coalesce()` is also a Spark DataFrame method used to reduce the number of partitions. Do not confuse the DataFrame method with `pyspark.sql.functions.coalesce()`.

---

## 8. Handling Null Values in Aggregations

Aggregation functions handle null values differently depending on the expression.

Consider:

```python
data = [
    ("Alice", 5000),
    ("Alice", None),
    ("Bob", 3000),
    ("Bob", 2000)
]

df = spark.createDataFrame(
    data,
    ["name", "salary"]
)
```

Example:

```python
df.agg(
    F.count("*").alias("total_rows"),
    F.count("salary").alias("non_null_salaries"),
    F.sum("salary").alias("total_salary"),
    F.avg("salary").alias("average_salary")
).show()
```

Expected output:

| total_rows | non_null_salaries | total_salary | average_salary |
|---:|---:|---:|---:|
| 4 | 3 | 10000 | 3333.33 |

Important differences:

| Function | Null behavior |
|---|---|
| `count("*")` | Counts all rows |
| `count("salary")` | Counts non-null values |
| `sum("salary")` | Ignores null values |
| `avg("salary")` | Ignores null values |
| `min("salary")` | Ignores null values |
| `max("salary")` | Ignores null values |

If every value in a group is null, `sum()` and `avg()` return null.

---

## 9. Null Handling in Joins

When joining two DataFrames using ordinary equality conditions, null values do not match other null values.

Example:

```python
df1 = spark.createDataFrame(
    [(1, "Alice"), (None, "Bob")],
    ["id", "name"]
)

df2 = spark.createDataFrame(
    [(1, "Kolkata"), (None, "Delhi")],
    ["id", "city"]
)
```

Ordinary join:

```python
result = df1.join(
    df2,
    df1.id == df2.id,
    "inner"
)
```

Only the records with `id = 1` match.

### Null-Safe Equality

If null values should match each other, use `eqNullSafe()`:

```python
result = df1.join(
    df2,
    df1.id.eqNullSafe(df2.id),
    "inner"
)
```

Now the null IDs can match.

**Important:** Use null-safe equality only when matching missing keys is logically correct for the business requirement.

---

## 10. Null Handling in String Columns

Null values and empty strings need separate treatment.

Example:

```python
df = spark.createDataFrame(
    [
        (1, "Alice"),
        (2, ""),
        (3, None),
        (4, "   ")
    ],
    ["id", "name"]
)
```

### Identify Empty Strings

```python
df.filter(
    F.col("name") == ""
)
```

### Identify Strings Containing Only Spaces

```python
df.filter(
    F.trim(F.col("name")) == ""
)
```

### Convert Empty or Space-Only Strings into Null

```python
df = df.withColumn(
    "name",
    F.when(
        F.trim(F.col("name")) == "",
        None
    ).otherwise(F.col("name"))
)
```

### Replace Null Values After Cleaning

```python
df = df.fillna({
    "name": "Unknown"
})
```

This is a common data-cleaning pattern when source systems represent missing text values inconsistently.

---

## 11. Practical Data Engineering Example

Suppose customer data arrives from a source system with missing names, ages, and email addresses.

```python
data = [
    (1, "Alice", 25, "alice@gmail.com"),
    (2, None, 30, None),
    (3, "Bob", None, "bob@gmail.com"),
    (4, "   ", 22, "")
]

df = spark.createDataFrame(
    data,
    ["customer_id", "name", "age", "email"]
)
```

Clean the data:

```python
df_clean = (
    df
    .withColumn(
        "name",
        F.when(
            F.trim(F.col("name")) == "",
            None
        ).otherwise(F.trim(F.col("name")))
    )
    .withColumn(
        "email",
        F.when(
            F.trim(F.col("email")) == "",
            None
        ).otherwise(F.lower(F.trim(F.col("email"))))
    )
    .fillna({
        "name": "Unknown",
        "age": 0
    })
)
```

This transformation:

1. Removes unwanted spaces from names.
2. Converts empty and space-only names into null.
3. Converts empty emails into null.
4. Standardizes email addresses to lowercase.
5. Replaces missing names and ages with default values.

**Data engineering point:** Do not automatically replace every null with zero or `"Unknown"`. Choose a replacement based on the meaning of the column and the business requirement.

---

## 12. Common Mistakes

1. Using `== None` instead of `isNull()` to check null values.
2. Treating null, zero, and empty strings as the same value.
3. Using `dropna()` without understanding that it may remove many valid records.
4. Filling numeric columns with incompatible string values.
5. Assuming null values match in ordinary joins.
6. Forgetting that `count("*")` and `count(column)` behave differently.
7. Replacing missing values without understanding their business meaning.
8. Assuming `fillna()` modifies the original DataFrame.
9. Forgetting to clean space-only strings separately from null values.

---

## 13. Common Interview Questions

**Q1. What is a null value in PySpark?**

A null value represents missing or unknown information in a DataFrame.

**Q2. What is the difference between `isNull()` and `isNotNull()`?**

- `isNull()` identifies null values.
- `isNotNull()` identifies non-null values.

**Q3. What is the difference between `dropna()` and `fillna()`?**

- `dropna()` removes rows containing null values according to the specified condition.
- `fillna()` replaces null values with specified values.

**Q4. What is the difference between `how="any"` and `how="all"` in dropna()?**

- `how="any"` removes a row if any relevant value is null.
- `how="all"` removes a row only if all relevant values are null.

**Q5. What does `thresh` mean in dropna()?**

It specifies the minimum number of non-null values a row must contain to be retained.

**Q6. What is `coalesce()` used for?**

It returns the first non-null value from multiple expressions.

**Q7. What is the difference between null and an empty string?**

- Null represents missing or unknown information.
- An empty string is a valid string containing zero characters.

**Q8. How does count handle null values?**

- `count("*")` counts all rows.
- `count(column)` counts only non-null values.

**Q9. Do null values match during a normal PySpark join?**

No. Ordinary equality comparisons do not match null with null. Use `eqNullSafe()` when null-safe matching is required.

**Q10. How would you handle missing values in a real data pipeline?**

First identify missing values, understand why they are missing, and then decide whether to remove, replace, preserve, or flag them based on business requirements.

---

## 14. Practice Questions

Using a DataFrame containing `customer_id`, `name`, `age`, `salary`, and `email`:

1. Count null values in every column.
2. Find rows where salary is null.
3. Find rows where name is not null.
4. Remove rows where all columns are null.
5. Remove rows where salary is null.
6. Replace missing salary values with zero.
7. Replace missing names with `"Unknown"`.
8. Replace null values using `coalesce()`.
9. Convert empty and space-only strings into null.
10. Calculate total rows and non-null salary counts.
11. Perform a join using null-safe equality.
12. Create a new column indicating whether customer email is missing.

---

## 15. Quick Revision

| Function | Purpose |
|---|---|
| `isNull()` | Checks null values |
| `isNotNull()` | Checks non-null values |
| `dropna()` | Removes rows containing null values |
| `fillna()` | Replaces null values |
| `when()` | Applies conditional logic |
| `otherwise()` | Defines the alternative condition |
| `coalesce()` | Returns the first non-null expression |
| `eqNullSafe()` | Compares values using null-safe equality |
| `trim()` | Removes spaces from both ends |
| `count("*")` | Counts all rows |
| `count(column)` | Counts non-null values |

**Remember these key differences:**

- Null is not the same as zero or an empty string.
- `dropna()` removes records; `fillna()` replaces missing values.
- `count("*")` counts all rows; `count(column)` ignores nulls.
- Ordinary equality joins do not match null keys.
- `coalesce()` returns the first non-null value.

---

## 16. Self-Assessment

- [ ] Identify null values in individual and multiple columns.
- [ ] Explain `dropna()` and its parameters.
- [ ] Replace missing values using `fillna()`.
- [ ] Use `when()` and `otherwise()` for conditional cleaning.
- [ ] Explain how `coalesce()` works.
- [ ] Handle null values in aggregation.
- [ ] Explain null behavior in joins.
- [ ] Differentiate null, empty strings, and space-only strings.
- [ ] Clean missing values in a practical DataFrame.

**Interview readiness:** You should be able to inspect a raw DataFrame, identify missing values, choose an appropriate cleaning strategy, and explain how null values affect joins and aggregations.