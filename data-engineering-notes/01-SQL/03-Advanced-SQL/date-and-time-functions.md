# SQL Date and Time Functions

## 1. What Are Date and Time Functions?

SQL date and time functions help you work with dates, times, timestamps, and date intervals.

They are commonly used in data engineering for:
- Filtering records by date ranges.
- Extracting year, month, and day.
- Calculating durations between timestamps.
- Aggregating data by day, month, or year.
- Identifying recent records and late-arriving data.
- Building incremental ETL pipelines and daily reports.

**Important:** Date and time function names and syntax vary across databases. The examples below use a mix of common SQL concepts and explicitly labeled dialect-specific syntax.

---

## 2. Sample Data

Assume we have an `orders` table:

| order_id | customer_id | order_date | amount |
|---:|---:|---|---:|
| 101 | 1 | 2026-01-05 10:30:00 | 1000 |
| 102 | 2 | 2026-01-15 14:45:00 | 500 |
| 103 | 1 | 2026-02-02 09:15:00 | 1500 |
| 104 | 3 | 2026-02-20 18:00:00 | 800 |
| 105 | 2 | 2026-03-10 11:00:00 | 700 |

Assume `order_date` is stored as a proper timestamp, not as arbitrary text.

## 3. Get the Current Date and Time

Different databases provide different functions for the current date and timestamp.

| Database | Current date | Current timestamp |
|---|---|---|
| PostgreSQL | `CURRENT_DATE` | `CURRENT_TIMESTAMP` |
| MySQL | `CURRENT_DATE()` | `CURRENT_TIMESTAMP()` |
| SQL Server | `CAST(GETDATE() AS DATE)` | `GETDATE()` |
| Databricks SQL | `CURRENT_DATE()` | `CURRENT_TIMESTAMP()` |

Example:

```sql
SELECT CURRENT_DATE();
```

This syntax works in MySQL and Databricks SQL. In PostgreSQL, use `SELECT CURRENT_DATE;`.

**Remember:** The current date and time depend on the database's execution environment and time-zone settings.

## 4. Extract Parts of a Date

You may need to extract the year, month, day, or hour from a timestamp.

### Common syntax

In MySQL:

```sql
SELECT
    YEAR(order_date) AS order_year,
    MONTH(order_date) AS order_month,
    DAY(order_date) AS order_day
FROM orders;
```

In PostgreSQL:

```sql
SELECT
    EXTRACT(YEAR FROM order_date) AS order_year,
    EXTRACT(MONTH FROM order_date) AS order_month,
    EXTRACT(DAY FROM order_date) AS order_day
FROM orders;
```

In Databricks SQL:

```sql
SELECT
    YEAR(order_date) AS order_year,
    MONTH(order_date) AS order_month,
    DAY(order_date) AS order_day
FROM orders;
```

Example for the first order:

| order_year | order_month | order_day |
|---:|---:|---:|
| 2026 | 1 | 5 |

**Interview use case:** Group transactions by month or compare business activity across years.

## 5. Filter Records by Date Range

Filtering date ranges is one of the most important SQL skills for data engineering.

### Example: Find orders placed in February 2026

A clear and reliable pattern is a half-open interval:

```sql
SELECT *
FROM orders
WHERE order_date >= '2026-02-01'
  AND order_date <  '2026-03-01';
```

This returns orders from February 1 up to, but not including, March 1.

**Why use this pattern?**

- It works correctly when `order_date` includes a time.
- You do not need to guess the last possible time of the month.
- It is often friendly to indexes and partition pruning when the column is stored appropriately.

### Common mistake

```sql
WHERE order_date BETWEEN '2026-02-01' AND '2026-02-28'
```

If `order_date` is a timestamp, the upper bound may represent midnight at the beginning of February 28, excluding later orders that day.

For timestamp filtering, prefer `>= start_date AND < next_period_start`.

## 6. Convert a Timestamp to a Date

Sometimes you want to remove the time portion to get the calendar date.

In PostgreSQL:

```sql
SELECT CAST(order_date AS DATE) AS order_day
FROM orders;
```

In MySQL:

```sql
SELECT DATE(order_date) AS order_day
FROM orders;
```

In Databricks SQL:

```sql
SELECT CAST(order_date AS DATE) AS order_day
FROM orders;
```

Example:

`2026-01-05 10:30:00` becomes `2026-01-05`.

**Performance tip:** Applying a function to a date column inside a filter can sometimes prevent efficient index usage. When filtering a full day, prefer a range:

```sql
WHERE order_date >= '2026-01-05'
  AND order_date <  '2026-01-06';
```

## 7. Add or Subtract Dates

You may need to calculate a date in the future or past.

### MySQL

```sql
SELECT DATE_ADD('2026-01-05', INTERVAL 7 DAY);
```

Subtract seven days:

```sql
SELECT DATE_SUB('2026-01-05', INTERVAL 7 DAY);
```

### PostgreSQL

```sql
SELECT DATE '2026-01-05' + 7;
```

Subtract seven days:

```sql
SELECT DATE '2026-01-05' - 7;
```

### Databricks SQL

```sql
SELECT DATE_ADD(DATE '2026-01-05', 7);
```

Subtract seven days:

```sql
SELECT DATE_SUB(DATE '2026-01-05', 7);
```

**Use cases:** Finding dates seven days ago, creating reporting windows, and identifying records due for processing.

## 8. Calculate the Difference Between Dates

Date differences can measure days between events, customer activity gaps, or processing delays.

### MySQL

```sql
SELECT DATEDIFF('2026-02-10', '2026-02-01');
```

Result: `9`

MySQL calculates the difference in calendar days using the first date minus the second date.

### SQL Server

```sql
SELECT DATEDIFF(
    DAY,
    '2026-02-01',
    '2026-02-10'
);
```

Result: `9`

SQL Server's `DATEDIFF` counts date-part boundaries crossed, which is important when using timestamps rather than dates.

### PostgreSQL

```sql
SELECT DATE '2026-02-10' - DATE '2026-02-01';
```

Result: `9`

**Important:** Check your database's rules before using date-difference functions. The argument order and meaning can differ.

## 9. Truncate Dates to a Month, Day, or Year

Date truncation groups timestamps into a larger time period, such as a month or year.

### PostgreSQL

```sql
SELECT DATE_TRUNC('month', order_date) AS order_month
FROM orders;
```

### Databricks SQL

```sql
SELECT DATE_TRUNC('MONTH', order_date) AS order_month
FROM orders;
```

For an order dated `2026-02-20 18:00:00`, month truncation returns the start of that month, represented as `2026-02-01 00:00:00` in the relevant timestamp type.

### Example: Monthly revenue

In Databricks SQL:

```sql
SELECT
    DATE_TRUNC('MONTH', order_date) AS order_month,
    SUM(amount) AS total_revenue
FROM orders
GROUP BY DATE_TRUNC('MONTH', order_date)
ORDER BY order_month;
```

Expected result:

| order_month | total_revenue |
|---|---:|
| 2026-01-01 00:00:00 | 1500 |
| 2026-02-01 00:00:00 | 2300 |
| 2026-03-01 00:00:00 | 700 |

**Remember:** Truncation changes the timestamp to the start of a specified period. It is different from extracting the month number.

## 10. Format Dates as Strings

Formatting is useful for reports and display, but it is generally better to keep dates as date or timestamp types for filtering and calculations.

### MySQL

```sql
SELECT DATE_FORMAT(order_date, '%Y-%m-%d')
FROM orders;
```

### PostgreSQL

```sql
SELECT TO_CHAR(order_date, 'YYYY-MM-DD')
FROM orders;
```

### Databricks SQL

```sql
SELECT DATE_FORMAT(order_date, 'yyyy-MM-dd')
FROM orders;
```

Example output: `2026-01-05`

**Important:** Formatting usually returns a string. Do not convert dates to strings unnecessarily before filtering or joining.

## 11. Handle `NULL` Dates

A date column may contain `NULL` when a date is unknown or not yet available.

Example:

```sql
SELECT *
FROM orders
WHERE order_date IS NULL;
```

To find rows with known dates:

```sql
SELECT *
FROM orders
WHERE order_date IS NOT NULL;
```

**Remember:** Use `IS NULL` or `IS NOT NULL`, not `= NULL` or `!= NULL`.

Date arithmetic involving `NULL` generally returns `NULL`, and aggregate functions such as `MIN()` and `MAX()` normally ignore `NULL` values.

## 12. Time Zones and Timestamps

Time zones matter when timestamps represent real events across regions.

Important concepts:

- **Date:** A calendar date without a time.
- **Time:** A time of day without a date.
- **Timestamp:** A date and time together.
- **Time zone:** The region or offset used to interpret a time.
- **UTC:** A common reference time standard for systems that exchange event timestamps.

For data engineering, storing event instants consistently, often in UTC, helps avoid confusion when data comes from multiple regions. Convert to local time when required for reporting.

**Important:** Timestamp types and time-zone behavior differ between databases. For example, PostgreSQL distinguishes `timestamp without time zone` from `timestamp with time zone`. Always confirm what the source timestamp represents before converting it.

## 13. Common Date and Time Mistakes

1. **Using the wrong function for the database.** Date functions are not completely portable across SQL dialects.
2. **Filtering a timestamp with an incomplete upper bound.** Use a half-open interval for complete days or months.
3. **Comparing strings instead of dates.** Use proper date and timestamp types whenever possible.
4. **Ignoring time zones.** A timestamp may represent UTC or local time; do not assume.
5. **Formatting too early.** Keep dates in date or timestamp types for calculations and filtering.
6. **Using `= NULL`.** Use `IS NULL`.
7. **Applying functions to indexed columns unnecessarily.** A direct range predicate may perform better.
8. **Assuming all date-difference functions behave the same.** Verify argument order and boundary behavior.
9. **Confusing calendar days with elapsed 24-hour periods.** Time zones and timestamp types can affect the interpretation.

## 14. Interview Questions

### Q1. How do you filter all records for a particular month?

Use a start-inclusive, end-exclusive range:

```sql
WHERE order_date >= '2026-02-01'
  AND order_date <  '2026-03-01'
```

### Q2. What is the difference between `DATE` and `TIMESTAMP`?

A `DATE` represents a calendar date. A `TIMESTAMP` represents a date and time, with time-zone behavior depending on the database type.

### Q3. How do you extract the month from a date?

Use a supported function such as `MONTH(order_date)` or `EXTRACT(MONTH FROM order_date)`.

### Q4. What is date truncation?

It sets smaller date/time components to the start of a specified period, such as the beginning of a month.

### Q5. Why should you be careful with time zones?

The same event may appear on different calendar dates in different time zones, affecting daily reports and date filters.

### Q6. How would you calculate monthly revenue?

Truncate or otherwise derive the month, group by it, and calculate `SUM(amount)`.

### Q7. Why is a half-open date range useful?

It includes all timestamps from the start through the end of the desired period without relying on an exact last timestamp.

### Q8. How would you find records from the last seven days?

Define the intended current-time reference and filter with an appropriate start and end timestamp. Confirm whether the requirement means seven rolling 24-hour periods or seven calendar dates.

### Q9. What is the difference between date formatting and date truncation?

Formatting converts a date or timestamp into a string for display. Truncation changes the date/time value to the start of a specified period.

### Q10. How do date functions help incremental data pipelines?

They help define extraction windows, identify recent records, group data by processing periods, and compare event and processing timestamps.

## 15. Practice Questions

Use the `orders` table from Section 2.

### Beginner

- [ ] Find all orders placed in February 2026.
- [ ] Extract the year and month from every order timestamp.
- [ ] Find orders placed on or after February 1, 2026.
- [ ] Convert `order_date` to a date without the time component.

### Intermediate

- [ ] Calculate total revenue for each month.
- [ ] Find the number of orders placed each day.
- [ ] Calculate the number of calendar days between two supplied dates.
- [ ] Find orders from a rolling seven-day period using a clearly defined reference timestamp.

### Advanced

- [ ] Explain how to filter a full month without excluding timestamps on the last day.
- [ ] Design an incremental extraction window using a start timestamp and an exclusive end timestamp.
- [ ] Explain how a UTC timestamp can fall on a different local calendar date.
- [ ] Compare date truncation with date extraction when grouping monthly data.

## 16. Quick Revision

| Concept | Remember |
|---|---|
| Current date | `CURRENT_DATE` or dialect equivalent |
| Current timestamp | `CURRENT_TIMESTAMP` or dialect equivalent |
| Extract parts | `YEAR`, `MONTH`, `DAY`, or `EXTRACT` |
| Date range | `>= start AND < next_period_start` |
| Cast to date | `CAST(timestamp AS DATE)` |
| Add or subtract days | Use database-specific date arithmetic |
| Date difference | Check function syntax and meaning |
| Truncation | Start of day, month, year, or other period |
| Formatting | Converts date/time to a string |
| `NULL` dates | Use `IS NULL` |
| Time zones | Confirm timestamp semantics and reporting zone |

## 17. Self-Check

Before considering this topic complete, make sure you can:

- [ ] Filter records for a complete day or month.
- [ ] Extract year, month, and day.
- [ ] Calculate monthly aggregates.
- [ ] Add and subtract dates.
- [ ] Calculate date differences using your target SQL dialect.
- [ ] Explain truncation versus formatting.
- [ ] Handle `NULL` dates correctly.
- [ ] Explain why time zones matter in data engineering.
- [ ] Adapt examples to your interview database, especially Databricks SQL.

**Final takeaway:** Learn the date-range pattern first: `column >= start AND column < next_period_start`. Then practice extraction, truncation, date arithmetic, and time-zone handling in your target SQL dialect.