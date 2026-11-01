# Date and String Functions in PySpark

## 1. Overview

PySpark provides built-in functions to work with date, timestamp, and string columns.

They are commonly used in data engineering for:

- Cleaning raw data.
- Standardizing date and string formats.
- Extracting date components.
- Calculating date differences.
- Filtering records by date.
- Transforming text columns.

Import the required functions:

```python
from pyspark.sql import functions as F
```

---

## 2. Date and Timestamp Data Types

| Data type | Description | Example |
|---|---|---|
| `DateType` | Stores a calendar date | `2026-10-30` |
| `TimestampType` | Stores date and time | `2026-10-30 14:30:00` |

Example:

```python
df.printSchema()
```

Convert a string column into a date:

```python
df = df.withColumn(
    "order_date",
    F.to_date("order_date", "yyyy-MM-dd")
)
```

Convert a string column into a timestamp:

```python
df = df.withColumn(
    "order_timestamp",
    F.to_timestamp("order_timestamp", "yyyy-MM-dd HH:mm:ss")
)
```

**Important:** Use the format that matches the actual input string.

---

## 3. Common Date Functions

### 3.1 Current Date and Timestamp

```python
df = df.withColumn("today", F.current_date())

df = df.withColumn("current_time", F.current_timestamp())
```

- `current_date()` returns the current date.
- `current_timestamp()` returns the current timestamp.

### 3.2 Extract Date Components

```python
df = df.withColumn("year", F.year("order_date"))

df = df.withColumn("month", F.month("order_date"))

df = df.withColumn("day", F.dayofmonth("order_date"))
```

Other useful functions:

```python
F.dayofweek("order_date")
F.dayofyear("order_date")
F.weekofyear("order_date")
F.quarter("order_date")
```

### 3.3 Format Dates

`date_format()` converts a date or timestamp into a formatted string.

```python
df = df.withColumn(
    "formatted_date",
    F.date_format("order_date", "dd-MM-yyyy")
)
```

Common patterns:

| Pattern | Meaning |
|---|---|
| `yyyy` | Four-digit year |
| `MM` | Month |
| `dd` | Day |
| `HH` | Hour (24-hour format) |
| `mm` | Minute |
| `ss` | Second |

**Interview point:** `MM` represents the month, while `mm` represents the minute.

### 3.4 Date Difference

`datediff()` returns the number of days between two dates.

```python
df = df.withColumn(
    "days_difference",
    F.datediff(F.current_date(), "order_date")
)
```

### 3.5 Add and Subtract Days

```python
df = df.withColumn(
    "next_date",
    F.date_add("order_date", 7)
)

df = df.withColumn(
    "previous_date",
    F.date_sub("order_date", 7)
)
```

### 3.6 Add Months

```python
df = df.withColumn(
    "future_date",
    F.add_months("order_date", 3)
)
```

### 3.7 Difference Between Months

```python
df = df.withColumn(
    "month_difference",
    F.months_between("end_date", "start_date")
)
```

`months_between()` can return a decimal value.

### 3.8 Last Day of Month

```python
df = df.withColumn(
    "month_end",
    F.last_day("order_date")
)
```

### 3.9 Truncate Dates

`trunc()` truncates a date to the beginning of a specified month or year.

```python
df = df.withColumn(
    "month_start",
    F.trunc("order_date", "month")
)
```

`date_trunc()` works with timestamps and can truncate to units such as hour, day, month, or year.

```python
df = df.withColumn(
    "month_start_timestamp",
    F.date_trunc("month", "order_timestamp")
)
```

---

## 4. Filtering Records Using Dates

Example: Get orders from 2026.

```python
df.filter(
    F.year("order_date") == 2026
)
```

Example: Get orders after January 1, 2026.

```python
df.filter(
    F.col("order_date") > F.lit("2026-01-01").cast("date")
)
```

Example: Get orders between two dates.

```python
df.filter(
    (F.col("order_date") >= F.lit("2026-01-01").cast("date")) &
    (F.col("order_date") <= F.lit("2026-01-31").cast("date"))
)
```

**Practical tip:** When filtering a timestamp column by a complete calendar day, a half-open interval is often safer:

```python
df.filter(
    (F.col("order_timestamp") >= F.lit("2026-01-01 00:00:00").cast("timestamp")) &
    (F.col("order_timestamp") < F.lit("2026-02-01 00:00:00").cast("timestamp"))
)
```

This includes every timestamp on January 31, regardless of its time.

---

## 5. Common String Functions

String functions are frequently used to clean and standardize incoming data.

### 5.1 Remove Extra Spaces

```python
df = df.withColumn("name", F.trim("name"))
```

| Function | Purpose |
|---|---|
| `trim()` | Removes spaces from both ends |
| `ltrim()` | Removes spaces from the left |
| `rtrim()` | Removes spaces from the right |

### 5.2 Change Letter Case

```python
df = df.withColumn("name_upper", F.upper("name"))

df = df.withColumn("name_lower", F.lower("name"))

df = df.withColumn("name_title", F.initcap("name"))
```

- `upper()` converts text to uppercase.
- `lower()` converts text to lowercase.
- `initcap()` capitalizes the first letter of each word.

### 5.3 Find String Length

```python
df = df.withColumn("name_length", F.length("name"))
```

### 5.4 Extract Part of a String

`substring()` extracts characters from a string.

```python
df = df.withColumn(
    "first_three",
    F.substring("name", 1, 3)
)
```

Syntax:

```python
F.substring(column, position, length)
```

The starting position is 1-based.

### 5.5 Split a String

`split()` divides a string into an array.

```python
df = df.withColumn(
    "email_parts",
    F.split("email", "@")
)
```

To access the first element:

```python
df = df.withColumn(
    "email_username",
    F.split("email", "@")[0]
)
```

PySpark array indexing with `[]` is zero-based.

### 5.6 Concatenate Strings

`concat()` combines multiple columns.

```python
df = df.withColumn(
    "full_name",
    F.concat("first_name", "last_name")
)
```

To add a space between values, use `concat_ws()`:

```python
df = df.withColumn(
    "full_name",
    F.concat_ws(" ", "first_name", "last_name")
)
```

**Important difference:**

- `concat()` returns null if an input value is null.
- `concat_ws()` skips null values when joining strings.

### 5.7 Replace Text

`regexp_replace()` replaces text matching a regular expression.

```python
df = df.withColumn(
    "clean_phone",
    F.regexp_replace("phone", "[^0-9]", "")
)
```

This removes all non-digit characters from the phone column.

### 5.8 Extract Text Using Regular Expressions

`regexp_extract()` extracts a matching part of a string.

Example: Extract the domain from an email address.

```python
df = df.withColumn(
    "domain",
    F.regexp_extract("email", "@(.+)", 1)
)
```

The last argument (`1`) specifies the first capturing group.

If no match is found, `regexp_extract()` returns an empty string.

### 5.9 Find a String Position

```python
df = df.withColumn(
    "position",
    F.instr("email", "@")
)
```

`instr()` returns the position of the first occurrence of a substring. It returns 0 when the substring is not found.

Another function:

```python
F.locate("@", "email")
```

`locate()` also returns the position of a substring.

### 5.10 Extract Text Before or After a Delimiter

`substring_index()` extracts text based on a delimiter.

```python
df = df.withColumn(
    "username",
    F.substring_index("email", "@", 1)
)
```

For an email such as `yash@gmail.com`, this returns `yash`.

To extract the domain:

```python
df = df.withColumn(
    "domain",
    F.substring_index("email", "@", -1)
)
```

---

## 6. Practical Data Cleaning Example

Suppose incoming customer data contains inconsistent names, emails, and phone numbers.

```python
df_clean = (
    df
    .withColumn("name", F.initcap(F.trim("name")))
    .withColumn("email", F.lower(F.trim("email")))
    .withColumn(
        "phone",
        F.regexp_replace("phone", "[^0-9]", "")
    )
    .withColumn(
        "email_domain",
        F.substring_index("email", "@", -1)
    )
)
```

This transformation:

1. Removes extra spaces from names.
2. Standardizes names.
3. Converts emails to lowercase.
4. Removes unwanted characters from phone numbers.
5. Extracts email domains.

---

## 7. Handling Null and Empty Strings

Null and empty strings are different.

| Value | Meaning |
|---|---|
| `NULL` | Missing or unknown value |
| `""` | Empty string |
| `" "` | String containing a space |

Check for null:

```python
df.filter(F.col("name").isNull())
```

Check for empty strings:

```python
df.filter(F.trim(F.col("name")) == "")
```

Replace null values:

```python
df = df.fillna({"name": "Unknown"})
```

Replace empty strings:

```python
df = df.withColumn(
    "name",
    F.when(
        F.trim(F.col("name")) == "",
        None
    ).otherwise(F.col("name"))
)
```

---

## 8. Common Interview Questions

**Q1. What is the difference between `DateType` and `TimestampType`?**

- `DateType` stores a date.
- `TimestampType` stores a date and time.

**Q2. What is the difference between `to_date()` and `date_format()`?**

- `to_date()` converts a value into a date.
- `date_format()` converts a date or timestamp into a formatted string.

**Q3. What is the difference between `datediff()` and `months_between()`?**

- `datediff()` calculates the difference in days.
- `months_between()` calculates the difference in months.

**Q4. What is the difference between `trim()`, `ltrim()`, and `rtrim()`?**

- `trim()` removes spaces from both ends.
- `ltrim()` removes spaces from the left.
- `rtrim()` removes spaces from the right.

**Q5. What is the difference between `concat()` and `concat_ws()`?**

- `concat()` joins values directly and returns null if an input is null.
- `concat_ws()` joins values using a separator and skips null values.

**Q6. What is the difference between `split()` and `substring()`?**

- `split()` divides a string into an array.
- `substring()` extracts a specific part of a string.

**Q7. What is `regexp_replace()` used for?**

It replaces text matching a regular expression. It is useful for removing unwanted characters and cleaning raw data.

**Q8. What is `regexp_extract()` used for?**

It extracts a matching part of a string using a regular expression and a capturing group.

**Q9. How do you extract the year and month from a date column?**

```python
df.select(
    F.year("order_date").alias("year"),
    F.month("order_date").alias("month")
)
```

**Q10. How do you get records from the current month?**

```python
df.filter(
    (F.year("order_date") == F.year(F.current_date())) &
    (F.month("order_date") == F.month(F.current_date()))
)
```

---

## 9. Common Mistakes

1. Using `MM` for minutes instead of `mm`.
2. Applying date functions directly to improperly formatted strings.
3. Confusing null values with empty strings.
4. Forgetting that `substring()` uses 1-based positions.
5. Forgetting that PySpark array indexing with `[]` is zero-based.
6. Assuming `concat()` and `concat_ws()` handle null values in the same way.
7. Forgetting to use the correct date format while converting strings.
8. Comparing date or timestamp columns with incorrectly formatted values.

---

## 10. Practice Questions

Use a DataFrame containing `customer_id`, `name`, `email`, `phone`, and `order_date`.

1. Convert `order_date` from string to DateType.
2. Extract the year, month, and day from `order_date`.
3. Find records belonging to the current year.
4. Calculate the number of days since each order.
5. Find the last day of each order's month.
6. Convert all customer names into title case.
7. Remove leading and trailing spaces from customer names.
8. Convert all email addresses to lowercase.
9. Extract usernames and domains from email addresses.
10. Remove special characters from phone numbers.
11. Find customers whose names are empty or null.
12. Create a full name by combining first and last names.

---

## 11. Quick Revision

| Function | Purpose |
|---|---|
| `to_date()` | Convert to date |
| `to_timestamp()` | Convert to timestamp |
| `current_date()` | Current date |
| `current_timestamp()` | Current timestamp |
| `year()` | Extract year |
| `month()` | Extract month |
| `dayofmonth()` | Extract day |
| `date_format()` | Format date as string |
| `datediff()` | Difference in days |
| `months_between()` | Difference in months |
| `date_add()` | Add days |
| `date_sub()` | Subtract days |
| `add_months()` | Add months |
| `last_day()` | Last day of month |
| `trunc()` | Truncate date to month/year |
| `date_trunc()` | Truncate timestamp |
| `trim()` | Remove spaces from both ends |
| `upper()` / `lower()` | Change letter case |
| `length()` | Find string length |
| `substring()` | Extract part of string |
| `split()` | Split string into array |
| `concat()` | Join strings |
| `concat_ws()` | Join strings with separator |
| `regexp_replace()` | Replace matching text |
| `regexp_extract()` | Extract matching text |
| `instr()` / `locate()` | Find substring position |
| `substring_index()` | Extract text using delimiter |

---

## 12. Self-Assessment

Before moving forward, make sure you can:

- [ ] Convert string columns into date and timestamp types.
- [ ] Extract year, month, and day.
- [ ] Calculate date differences.
- [ ] Add and subtract days or months.
- [ ] Filter records using date conditions.
- [ ] Clean strings using trim and case functions.
- [ ] Split and concatenate strings.
- [ ] Use regular expressions for data cleaning.
- [ ] Handle null and empty string values.

**Interview readiness:** You should be able to explain the purpose of each function and write a simple transformation without looking at the notes.