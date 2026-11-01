

# Select, Filter, and WithColumn in PySpark

These are three of the most frequently used operations when transforming data in PySpark DataFrames.

- `select()` – Selects specific columns or expressions.
- `filter()` / `where()` – Filters rows based on conditions.
- `withColumn()` – Creates a new column or replaces an existing column.

These operations are commonly used in ETL pipelines to select required data, remove unwanted records, and create transformed columns.

---

## 1. Sample DataFrame

Consider the following employee DataFrame.

| employee_id | name | department | age | salary |
|---:|---|---|---:|---:|
| 101 | Rahul | IT | 25 | 45000 |
| 102 | Amit | HR | 30 | 35000 |
| 103 | Priya | IT | 28 | 60000 |
| 104 | Neha | Finance | 35 | 55000 |
| 105 | Karan | IT | 24 | 30000 |

We will use this DataFrame throughout the examples.

```python
data = [
    (101, "Rahul", "IT", 25, 45000),
    (102, "Amit", "HR", 30, 35000),
    (103, "Priya", "IT", 28, 60000),
    (104, "Neha", "Finance", 35, 55000),
    (105, "Karan", "IT", 24, 30000)
]

columns = [
    "employee_id",
    "name",
    "department",
    "age",
    "salary"
]

df = spark.createDataFrame(data, columns)
```

Import functions:

```python
from pyspark.sql.functions import col, when, lit
```

---

## 2. select() – Selecting Columns

`select()` is used to retrieve specific columns from a DataFrame.

### A. Select a single column

```python
df.select("name").show()
```

Output:

| name |
|---|
| Rahul |
| Amit |
| Priya |
| Neha |
| Karan |

### B. Select multiple columns

```python
df.select(
    "name",
    "department",
    "salary"
).show()
```

### C. Select using col()

```python
df.select(
    col("name"),
    col("salary")
).show()
```

Both approaches produce the same result.

### D. Select all columns

```python
df.select("*").show()
```

### E. Select with expressions

You can perform calculations while selecting columns.

```python
df.select(
    col("name"),
    col("salary"),
    (col("salary") * 12).alias("annual_salary")
).show()
```

The `alias()` function assigns a new name to the resulting column.

### F. Select distinct values

```python
df.select("department").distinct().show()
```

This returns unique department values.

**Important:** `select()` changes the columns in the output DataFrame, not the original DataFrame.

---

## 3. filter() – Filtering Rows

`filter()` is used to retrieve rows that satisfy a given condition.

### A. Filter using a single condition

Find employees whose salary is greater than 40,000.

```python
df.filter(
    col("salary") > 40000
).show()
```

Result:

| employee_id | name | department | age | salary |
|---:|---|---|---:|---:|
| 101 | Rahul | IT | 25 | 45000 |
| 103 | Priya | IT | 28 | 60000 |
| 104 | Neha | Finance | 35 | 55000 |

### B. Filter using multiple conditions

Find IT employees whose salary is greater than 40,000.

```python
df.filter(
    (col("department") == "IT") &
    (col("salary") > 40000)
).show()
```

Use:
- `&` for AND
- `|` for OR
- `~` for NOT

**Important:** Always put individual conditions inside parentheses when combining them.

### C. Using OR condition

Find employees working in IT or HR.

```python
df.filter(
    (col("department") == "IT") |
    (col("department") == "HR")
).show()
```

### D. Using NOT condition

Find employees who are not working in IT.

```python
df.filter(
    ~(col("department") == "IT")
).show()
```

You can also write:

```python
df.filter(
    col("department") != "IT"
).show()
```

### E. Using isin()

Find employees working in IT or Finance.

```python
df.filter(
    col("department").isin("IT", "Finance")
).show()
```

### F. Using between()

Find employees whose salary is between 35,000 and 55,000, including both boundaries.

```python
df.filter(
    col("salary").between(35000, 55000)
).show()
```

### G. Filtering NULL values

Find employees whose department is NULL.

```python
df.filter(
    col("department").isNull()
).show()
```

Find employees whose department is not NULL.

```python
df.filter(
    col("department").isNotNull()
).show()
```

**Important:** Use `isNull()` and `isNotNull()` to check NULL values. Do not use `col("department") == None`.

---

## 4. where() – Another Way to Filter Rows

`where()` works as an alias for `filter()`.

Both produce the same result.

```python
df.filter(col("salary") > 40000).show()
```

Equivalent:

```python
df.where(col("salary") > 40000).show()
```

You can use whichever makes your code easier to read.

---

## 5. withColumn() – Adding or Updating Columns

`withColumn()` is used to add a new column or replace an existing column.

### A. Create a new column

Calculate annual salary.

```python
df = df.withColumn(
    "annual_salary",
    col("salary") * 12
)
```

Result:

| name | salary | annual_salary |
|---|---:|---:|
| Rahul | 45000 | 540000 |
| Amit | 35000 | 420000 |
| Priya | 60000 | 720000 |
| Neha | 55000 | 660000 |
| Karan | 30000 | 360000 |

### B. Update an existing column

Suppose you want to increase every employee's salary by 10%.

```python
df = df.withColumn(
    "salary",
    col("salary") * 1.10
)
```

If a column with that name already exists, `withColumn()` replaces it in the resulting DataFrame.

### C. Add a constant column using lit()

Use `lit()` to add a constant value.

```python
df = df.withColumn(
    "country",
    lit("India")
)
```

Every row will contain `"India"` in the new column.

### D. Create a column using conditional logic

Use `when()` and `otherwise()` to create conditional columns.

```python
df = df.withColumn(
    "salary_category",
    when(col("salary") >= 50000, "High")
    .otherwise("Low")
)
```

Result:

| name | salary | salary_category |
|---|---:|---|
| Rahul | 45000 | Low |
| Amit | 35000 | Low |
| Priya | 60000 | High |
| Neha | 55000 | High |
| Karan | 30000 | Low |

### E. Multiple conditions using when()

```python
df = df.withColumn(
    "experience_level",
    when(col("age") < 25, "Junior")
    .when(col("age") <= 30, "Mid-level")
    .otherwise("Senior")
)
```

Conditions are evaluated in order. The first matching condition determines the result.

---

## 6. Difference Between select() and withColumn()

Both can be used to create transformed columns, but their purposes differ.

| select() | withColumn() |
|---|---|
| Selects columns and expressions | Adds or replaces one column |
| Can return only specified columns | Retains other existing columns |
| Useful for choosing output structure | Useful for incremental transformations |
| Can create multiple expressions together | Creates or replaces one column per call |

Example using `select()`:

```python
df.select(
    "name",
    (col("salary") * 12).alias("annual_salary")
).show()
```

Example using `withColumn()`:

```python
df.withColumn(
    "annual_salary",
    col("salary") * 12
).show()
```

The first example returns only `name` and `annual_salary`. The second retains the existing columns and adds `annual_salary`.

---

## 7. Combining select(), filter(), and withColumn()

In real-world ETL pipelines, these operations are commonly combined.

### Business requirement

1. Select employees from the IT department.
2. Keep employees earning more than 40,000.
3. Calculate annual salary.
4. Create a salary category.
5. Return only the required columns.

```python
result_df = (
    df
    .filter(col("department") == "IT")
    .filter(col("salary") > 40000)
    .withColumn(
        "annual_salary",
        col("salary") * 12
    )
    .withColumn(
        "salary_category",
        when(col("salary") >= 50000, "High")
        .otherwise("Low")
    )
    .select(
        "employee_id",
        "name",
        "department",
        "salary",
        "annual_salary",
        "salary_category"
    )
)

result_df.show()
```

Expected result:

| employee_id | name | department | salary | annual_salary | salary_category |
|---:|---|---|---:|---:|---|
| 101 | Rahul | IT | 45000 | 540000 | Low |
| 103 | Priya | IT | 60000 | 720000 | High |

This is a basic example of a DataFrame transformation pipeline.

---

## 8. Lazy Evaluation

PySpark transformations such as `select()`, `filter()`, and `withColumn()` are lazy.

This means Spark does not immediately execute the complete computation when these transformations are defined.

Example:

```python
filtered_df = df.filter(
    col("salary") > 40000
)

selected_df = filtered_df.select(
    "name",
    "salary"
)
```

At this point, Spark has built a logical plan for the transformations.

Execution is triggered by an action such as:

```python
selected_df.show()
```

Other actions include:
- `count()`
- `collect()`
- `write()`

**Interview point:** Transformations define what needs to happen. Actions trigger execution.

---

## 9. Common Mistakes

1. **Using Python `and` instead of `&`**

   Incorrect:

   ```python
   df.filter(
       (col("age") > 25) and (col("salary") > 40000)
   )
   ```

   Correct:

   ```python
   df.filter(
       (col("age") > 25) &
       (col("salary") > 40000)
   )
   ```

2. **Forgetting parentheses around conditions**

   When using `&` or `|`, wrap each condition in parentheses.

3. **Assuming withColumn() modifies the original DataFrame**

   DataFrames are immutable. Assign the returned DataFrame if you want to retain the transformation.

   ```python
   df = df.withColumn(
       "annual_salary",
       col("salary") * 12
   )
   ```

4. **Using `== None` to check NULL values**

   Use `isNull()` or `isNotNull()`.

5. **Assuming select() retains all original columns**

   It returns only the columns and expressions specified.

6. **Creating too many repeated withColumn() transformations**

   For several new columns, consider using a single `select()` with multiple expressions when appropriate. This can make the code easier to maintain.

7. **Using a column name that does not exist**

   Check the schema or `df.columns` before applying transformations.

---

## 10. Common Interview Questions

**Q1. What is the purpose of select() in PySpark?**

It selects specific columns or expressions from a DataFrame.

**Q2. What is the difference between filter() and where()?**

There is no functional difference. `where()` is an alias for `filter()`.

**Q3. How do you filter records using multiple conditions?**

Use `&` for AND and `|` for OR, with parentheses around individual conditions.

**Q4. What is withColumn() used for?**

It adds a new column or replaces an existing column in a DataFrame.

**Q5. Can withColumn() update an existing column?**

Yes. If the specified column already exists, the resulting DataFrame contains the replacement column.

**Q6. Does withColumn() modify the original DataFrame?**

No. PySpark DataFrames are immutable. It returns a new DataFrame.

**Q7. What is lit() used for?**

It creates a literal column expression, commonly used to add a constant value to every row.

**Q8. How do you create a conditional column?**

Using `when()` and `otherwise()`.

**Q9. What is the difference between select() and withColumn()?**

`select()` controls which columns and expressions appear in the output. `withColumn()` adds or replaces a column while retaining the other columns.

**Q10. Are select(), filter(), and withColumn() transformations or actions?**

All three are transformations. They are evaluated when an action triggers execution.

**Q11. How do you filter NULL values?**

Using `isNull()` and `isNotNull()`.

**Q12. Why do we use col() in PySpark?**

`col()` creates a column expression that can be used in DataFrame operations.

---

## 11. Practice Questions

Use the employee DataFrame from the beginning of these notes.

1. Display only employee names and salaries.
2. Find employees whose age is greater than 25.
3. Find employees working in IT with salaries above 40,000.
4. Find employees belonging to IT, HR, or Finance using `isin()`.
5. Find employees whose salary is between 35,000 and 55,000.
6. Create a new column called `annual_salary`.
7. Add a constant column called `country` with the value `"India"`.
8. Increase every employee's salary by 15%.
9. Create a column called `salary_category` using `when()` and `otherwise()`.
10. Filter IT employees, calculate annual salary, and return only employee ID, name, and annual salary.

---

## 12. Quick Revision

| Operation | Purpose |
|---|---|
| `select()` | Select columns |
| `filter()` | Filter rows |
| `where()` | Alias for filter |
| `withColumn()` | Add or replace a column |
| `col()` | Reference a column |
| `lit()` | Create a literal value |
| `when()` | Apply conditional logic |
| `otherwise()` | Define the alternative condition |
| `isin()` | Match against multiple values |
| `between()` | Check an inclusive range |
| `isNull()` | Find NULL values |
| `isNotNull()` | Find non-NULL values |
| `alias()` | Rename an expression |

### Important code to remember

```python
from pyspark.sql.functions import col, lit, when

# Select columns
df.select("name", "salary")

# Filter rows
df.filter(col("salary") > 40000)

# Multiple conditions
df.filter(
    (col("age") > 25) &
    (col("salary") > 40000)
)

# Add a column
df.withColumn(
    "annual_salary",
    col("salary") * 12
)

# Add a constant column
df.withColumn(
    "country",
    lit("India")
)

# Conditional column
df.withColumn(
    "category",
    when(col("salary") >= 50000, "High")
    .otherwise("Low")
)
```

---

## 13. Self-Assessment

Before moving to the next topic, make sure you can:

- [ ] Select individual and multiple columns.
- [ ] Filter rows using single and multiple conditions.
- [ ] Use AND, OR, and NOT correctly.
- [ ] Filter using `isin()` and `between()`.
- [ ] Handle NULL values in conditions.
- [ ] Add and replace columns using `withColumn()`.
- [ ] Create conditional columns using `when()`.
- [ ] Combine multiple transformations into an ETL pipeline.
- [ ] Explain why these operations are lazy transformations.

**Interview readiness:** You should be able to take a raw DataFrame, filter the required records, create derived columns, and select the final output columns without assistance.

