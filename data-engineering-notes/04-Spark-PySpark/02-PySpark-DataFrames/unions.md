

# Union Operations in PySpark

## 1. What is Union in PySpark?

Union is used to combine rows from two or more DataFrames into a single DataFrame.

It is commonly used when data comes from multiple files, sources, regions, or time periods but has a compatible structure.

**Example:**

DataFrame 1 – January sales:

| order_id | customer | amount |
|---:|---|---:|
| 101 | Rahul | 500 |
| 102 | Amit | 700 |

DataFrame 2 – February sales:

| order_id | customer | amount |
|---:|---|---:|
| 103 | Priya | 900 |
| 104 | Neha | 600 |

After union:

| order_id | customer | amount |
|---:|---|---:|
| 101 | Rahul | 500 |
| 102 | Amit | 700 |
| 103 | Priya | 900 |
| 104 | Neha | 600 |

The resulting DataFrame contains rows from both DataFrames.

---

## 2. Creating Sample DataFrames

```python
data1 = [
    (101, "Rahul", 500),
    (102, "Amit", 700)
]

data2 = [
    (103, "Priya", 900),
    (104, "Neha", 600)
]

columns = ["order_id", "customer", "amount"]

df1 = spark.createDataFrame(data1, columns)
df2 = spark.createDataFrame(data2, columns)
```

Both DataFrames have the same columns and compatible data types.

---

## 3. union()

The `union()` method combines two DataFrames by their column positions.

```python
combined_df = df1.union(df2)

combined_df.show()
```

Output:

```text
+--------+--------+------+
|order_id|customer|amount|
+--------+--------+------+
|     101|   Rahul|   500|
|     102|    Amit|   700|
|     103|   Priya|   900|
|     104|    Neha|   600|
+--------+--------+------+
```

### Important points

- Both DataFrames should have the same number of columns.
- Corresponding columns must have compatible data types.
- Columns are matched by position, not by name.
- Duplicate rows are retained.

**Remember:** `union()` behaves like SQL `UNION ALL`, not SQL `UNION`.

---

## 4. unionByName()

`unionByName()` combines DataFrames by matching column names instead of positions.

This is useful when two DataFrames contain the same columns but in a different order.

Example:

```python
data1 = [
    (101, "Rahul", 500)
]

data2 = [
    ("Amit", 700, 102)
]

df1 = spark.createDataFrame(
    data1,
    ["order_id", "customer", "amount"]
)

df2 = spark.createDataFrame(
    data2,
    ["customer", "amount", "order_id"]
)
```

### Using union()

```python
df1.union(df2).show()
```

Because `union()` matches columns by position, the second DataFrame's values may be assigned to the wrong columns.

This can cause incorrect results or type errors.

### Using unionByName()

```python
combined_df = df1.unionByName(df2)

combined_df.show()
```

Expected result:

```text
+--------+--------+------+
|order_id|customer|amount|
+--------+--------+------+
|     101|   Rahul|   500|
|     102|    Amit|   700|
+--------+--------+------+
```

**Interview point:** Prefer `unionByName()` when the order of columns may differ between input DataFrames.

---

## 5. unionByName() with allowMissingColumns

Sometimes two DataFrames have different columns.

Example:

DataFrame 1:

| employee_id | name | salary |
|---:|---|---:|
| 101 | Rahul | 45000 |
| 102 | Amit | 50000 |

DataFrame 2:

| employee_id | name | department |
|---:|---|---|
| 103 | Priya | IT |
| 104 | Neha | HR |

In this case, `unionByName()` can use `allowMissingColumns=True`.

```python
combined_df = df1.unionByName(
    df2,
    allowMissingColumns=True
)

combined_df.show()
```

Expected structure:

| employee_id | name | salary | department |
|---:|---|---:|---|
| 101 | Rahul | 45000 | NULL |
| 102 | Amit | 50000 | NULL |
| 103 | Priya | NULL | IT |
| 104 | Neha | NULL | HR |

Missing columns are filled with NULL values.

### Important points

- `allowMissingColumns=True` is supported by `unionByName()`.
- Missing columns are added to the result.
- Missing values are filled with NULL.
- Corresponding column data types must still be compatible.
- It does not automatically solve every schema mismatch.

---

## 6. union() vs unionByName()

| Feature | union() | unionByName() |
|---|---|---|
| Column matching | Position | Column name |
| Column order | Must be aligned | Can differ |
| Different column names | Not automatically matched | Not automatically matched |
| Missing columns | Not automatically handled | Can be handled with `allowMissingColumns=True` |
| Duplicate rows | Retained | Retained |
| Common use | Same column order | Different column order or evolving schemas |

**Easy way to remember:**

- `union()` → Match by position.
- `unionByName()` → Match by column name.
- `unionByName(..., allowMissingColumns=True)` → Match by name and fill missing columns with NULL.

---

## 7. Removing Duplicates After Union

Union operations retain duplicate rows.

Example:

```python
data1 = [
    (101, "Rahul"),
    (102, "Amit")
]

data2 = [
    (102, "Amit"),
    (103, "Priya")
]

df1 = spark.createDataFrame(
    data1,
    ["employee_id", "name"]
)

df2 = spark.createDataFrame(
    data2,
    ["employee_id", "name"]
)
```

### Union with duplicates

```python
combined_df = df1.union(df2)

combined_df.show()
```

The row for Amit appears twice.

### Remove exact duplicate rows

```python
distinct_df = combined_df.distinct()
```

### Alternative: dropDuplicates()

```python
distinct_df = combined_df.dropDuplicates()
```

Both remove duplicate rows across all columns.

### Remove duplicates based on selected columns

```python
distinct_df = combined_df.dropDuplicates(
    ["employee_id"]
)
```

This removes duplicate rows based on `employee_id`.

**Important:** If multiple rows share the same `employee_id` but have different values in other columns, `dropDuplicates()` does not guarantee which row will be retained. Use a window function when a specific record-selection rule is required.

---

## 8. SQL UNION vs UNION ALL

In SQL:

- `UNION` combines results and removes duplicate rows.
- `UNION ALL` combines results and retains duplicate rows.

In PySpark:

| SQL | PySpark equivalent |
|---|---|
| `UNION ALL` | `df1.union(df2)` |
| `UNION` | `df1.union(df2).distinct()` |

Example:

```python
# Similar to SQL UNION ALL
df1.union(df2)

# Similar to SQL UNION
df1.union(df2).distinct()
```

---

## 9. Real-World ETL Example

### Business requirement

A company receives customer data from two regional sources:

- North region
- South region

Both datasets need to be combined into a single customer dataset.

North region:

```python
north_data = [
    (101, "Rahul", "North"),
    (102, "Amit", "North")
]

north_df = spark.createDataFrame(
    north_data,
    ["customer_id", "name", "region"]
)
```

South region:

```python
south_data = [
    (103, "Priya", "South"),
    (104, "Neha", "South")
]

south_df = spark.createDataFrame(
    south_data,
    ["customer_id", "name", "region"]
)
```

### Combine both datasets

```python
customer_df = north_df.unionByName(south_df)
```

### Remove duplicate customers

```python
customer_df = customer_df.dropDuplicates(
    ["customer_id"]
)
```

### Save the final data

```python
(
    customer_df.write
    .format("delta")
    .mode("overwrite")
    .save("/data/customers")
)
```

This is a simple example of combining data from multiple sources in an ETL pipeline.

---

## 10. Performance Considerations

1. **Union does not automatically remove duplicates.**

   Avoid unnecessary `distinct()` or `dropDuplicates()` because duplicate removal requires additional computation.

2. **Check schemas before union.**

   Ensure corresponding columns have compatible data types.

3. **Prefer unionByName() for evolving schemas.**

   It helps avoid incorrect column alignment when source column order differs.

4. **Avoid unnecessary repartitioning.**

   Do not repartition every DataFrame before union without a performance reason.

5. **Understand lazy evaluation.**

   Union is a transformation. Spark executes the computation when an action is called.

6. **Be careful with duplicate handling.**

   Removing duplicates across all columns is different from removing duplicates based on a business key.

---

## 11. Common Mistakes

1. **Assuming union() matches columns by name**

   It matches columns by position.

2. **Assuming union() removes duplicates**

   It retains duplicates.

3. **Using DataFrames with incompatible schemas**

   Check column counts and corresponding data types.

4. **Forgetting allowMissingColumns=True**

   `unionByName()` does not automatically allow missing columns unless this option is enabled.

5. **Using distinct() unnecessarily**

   Duplicate removal can increase execution cost.

6. **Assuming dropDuplicates() selects the latest record**

   It does not guarantee which duplicate is retained. Use `row_number()` with an ordering condition when selecting the latest record.

---

## 12. Common Interview Questions

**Q1. What is union in PySpark?**

Union combines rows from two DataFrames into one DataFrame.

**Q2. Does union() remove duplicate records?**

No. It retains duplicate rows.

**Q3. What is the difference between union() and unionByName()?**

`union()` matches columns by position, while `unionByName()` matches columns by name.

**Q4. What is the difference between SQL UNION and UNION ALL?**

`UNION` removes duplicate rows, while `UNION ALL` retains them.

**Q5. How do you perform SQL UNION in PySpark?**

Use:

```python
df1.union(df2).distinct()
```

**Q6. How do you combine DataFrames with different column orders?**

Use `unionByName()`.

**Q7. What does allowMissingColumns=True do?**

It allows missing columns to be added to the result and fills their values with NULL.

**Q8. How do you remove duplicates after union?**

Use `distinct()` or `dropDuplicates()`.

**Q9. Can union combine DataFrames with different column names?**

`unionByName()` matches columns by name. Columns with different names are treated as different columns. Rename columns first if they represent the same business field.

**Q10. Is union a transformation or an action?**

Union is a transformation. It is evaluated when an action triggers execution.

**Q11. What happens if the same record exists in both DataFrames?**

It appears twice after `union()`. Use `distinct()` or `dropDuplicates()` if duplicate removal is required.

**Q12. How would you combine monthly sales files into one DataFrame?**

Read the files into DataFrames with compatible schemas and combine them using `union()` or `unionByName()`.

---

## 13. Practice Questions

1. Create two employee DataFrames and combine them using `union()`.
2. Create two DataFrames with columns in different orders. Combine them using `unionByName()`.
3. Combine DataFrames with missing columns using `allowMissingColumns=True`.
4. Find out whether union retains duplicate records.
5. Remove duplicate rows using `distinct()`.
6. Remove duplicate customers based on `customer_id`.
7. Explain the difference between `union()` and SQL `UNION ALL`.
8. Explain how you would combine data from multiple regional sources.
9. What would you do if two DataFrames have different column names for the same business field?
10. Explain why `dropDuplicates()` should not be used when a specific latest-record selection rule is required.

---

## 14. Quick Revision

| Concept | Remember |
|---|---|
| `union()` | Combines rows by position |
| `unionByName()` | Combines rows by column name |
| `allowMissingColumns=True` | Fills missing columns with NULL |
| `distinct()` | Removes exact duplicate rows |
| `dropDuplicates()` | Removes duplicates, optionally using selected columns |
| SQL UNION | Removes duplicates |
| SQL UNION ALL | Retains duplicates |
| Union execution | Lazy transformation |

### Important code to remember

```python
# Combine by position
df1.union(df2)

# Combine by column name
df1.unionByName(df2)

# Allow missing columns
df1.unionByName(
    df2,
    allowMissingColumns=True
)

# Remove exact duplicates
df1.union(df2).distinct()

# Remove duplicates based on a key
df1.union(df2).dropDuplicates(
    ["customer_id"]
)
```

---

## 15. Self-Assessment

Before moving to the next topic, make sure you can:

- [ ] Explain what union does.
- [ ] Use `union()` correctly.
- [ ] Explain positional column matching.
- [ ] Use `unionByName()`.
- [ ] Handle missing columns.
- [ ] Explain duplicate behavior.
- [ ] Remove duplicates using `distinct()` and `dropDuplicates()`.
- [ ] Explain SQL UNION versus UNION ALL.
- [ ] Combine data from multiple sources in an ETL pipeline.

**Interview readiness:** You should be able to combine multiple DataFrames, handle different column orders, manage missing columns, and explain duplicate handling with a real-world example.

