# SQL: Consecutive Days and Gaps

## 1. What Is This SQL Pattern?

Consecutive-days and gaps problems involve analyzing dates or events to find:
- Users active on consecutive days.
- Missing dates in a sequence.
- Gaps between customer transactions.
- The longest consecutive activity streak.
- Groups of consecutive dates.
- Periods when a business had no activity.

These problems are common in Data Engineer interviews because they test window functions, date arithmetic, grouping, and the ability to reason about sequential records.

**Core idea:** First, order the records. Then compare each record with its neighbors or group consecutive records into streaks.

## 2. Sample Data

Assume we have a `user_activity` table:

| user_id | activity_date |
|---|---|
| U101 | 2026-01-01 |
| U101 | 2026-01-02 |
| U101 | 2026-01-03 |
| U101 | 2026-01-06 |
| U101 | 2026-01-07 |
| U102 | 2026-01-01 |
| U102 | 2026-01-03 |
| U102 | 2026-01-04 |

Assume there is at most one row per user per calendar date. If duplicates are possible, deduplicate to one row per user and date before calculating streaks.

For U101, the consecutive streaks are:

- January 1–3: three days.
- January 6–7: two days.

January 4 and 5 are missing from U101's activity sequence.

## 3. Pattern 1: Find Consecutive Dates Using `LAG()`

**Use when:** You need to identify dates that immediately follow the previous recorded date.

### Query

```sql
WITH previous_activity AS (
    SELECT
        user_id,
        activity_date,
        LAG(activity_date) OVER (
            PARTITION BY user_id
            ORDER BY activity_date
        ) AS previous_date
    FROM user_activity
)
SELECT
    user_id,
    previous_date,
    activity_date
FROM previous_activity
WHERE activity_date = previous_date + INTERVAL '1 day';
```

This uses PostgreSQL-style date arithmetic.

### Expected result

| user_id | previous_date | activity_date |
|---|---|---|
| U101 | 2026-01-01 | 2026-01-02 |
| U101 | 2026-01-02 | 2026-01-03 |
| U101 | 2026-01-06 | 2026-01-07 |
| U102 | 2026-01-03 | 2026-01-04 |

### How it works

1. `PARTITION BY user_id` separates each user's activity.
2. `ORDER BY activity_date` arranges dates chronologically.
3. `LAG(activity_date)` retrieves the preceding recorded date.
4. The `WHERE` clause keeps rows exactly one day after that date.

**Important:** `LAG()` compares rows, not calendar dates automatically. If the previous recorded activity was five days earlier, it still returns that row's date.

### Date arithmetic differs by database

| Database | Add one day |
|---|---|
| PostgreSQL | `activity_date + INTERVAL '1 day'` |
| MySQL | `DATE_ADD(activity_date, INTERVAL 1 DAY)` |
| SQL Server | `DATEADD(day, 1, activity_date)` |
| Databricks SQL | `DATE_ADD(activity_date, 1)` |

Use the syntax supported by your database.

## 4. Pattern 2: Find Gaps Between Consecutive Events

**Use when:** You need the time between a customer's transactions, user logins, or system events.

Assume a `transactions` table:

| customer_id | transaction_date |
|---|---|
| C101 | 2026-01-01 |
| C101 | 2026-01-04 |
| C101 | 2026-01-10 |
| C102 | 2026-01-02 |
| C102 | 2026-01-03 |

### Query

```sql
WITH transaction_gaps AS (
    SELECT
        customer_id,
        transaction_date,
        LAG(transaction_date) OVER (
            PARTITION BY customer_id
            ORDER BY transaction_date
        ) AS previous_transaction_date
    FROM transactions
)
SELECT
    customer_id,
    previous_transaction_date,
    transaction_date,
    transaction_date - previous_transaction_date AS gap
FROM transaction_gaps
WHERE previous_transaction_date IS NOT NULL;
```

This subtraction syntax is PostgreSQL-style and returns an interval for date/timestamp values.

### Interpretation

For C101:

- January 1 to January 4: three elapsed days.
- January 4 to January 10: six elapsed days.

For C102:

- January 2 to January 3: one elapsed day.

**Important distinction:** A three-day elapsed gap between two dates does not mean three full missing calendar dates. January 1 to January 4 has two intervening calendar dates: January 2 and January 3.

If you need the number of missing whole dates between two date-only values, subtract one from the elapsed day difference.

## 5. Pattern 3: Find Dates With a Gap Greater Than N Days

**Use when:** You need to identify inactive periods or customers who did not return within a specified time.

Example: Find transaction gaps greater than three elapsed days.

```sql
WITH transaction_gaps AS (
    SELECT
        customer_id,
        transaction_date,
        LAG(transaction_date) OVER (
            PARTITION BY customer_id
            ORDER BY transaction_date
        ) AS previous_transaction_date
    FROM transactions
)
SELECT
    customer_id,
    previous_transaction_date,
    transaction_date
FROM transaction_gaps
WHERE previous_transaction_date IS NOT NULL
  AND transaction_date > previous_transaction_date + INTERVAL '3 days';
```

This is PostgreSQL-style syntax.

For date-only values, a difference greater than three days means more than three elapsed calendar days between the dates.

To find gaps of at least three days, use the appropriate inclusive condition for the requirement.

**Interview tip:** Clarify whether the question means elapsed days, missing calendar dates, or a threshold that is inclusive.

## 6. Pattern 4: Group Consecutive Dates Into Streaks

**Use when:** You need to return each consecutive period with its start date, end date, and length.

A useful technique is the **date-minus-row-number pattern**.

### Step 1: Number the dates

```sql
SELECT
    user_id,
    activity_date,
    ROW_NUMBER() OVER (
        PARTITION BY user_id
        ORDER BY activity_date
    ) AS rn
FROM user_activity;
```

For U101, the dates receive row numbers 1 through 5.

### Step 2: Subtract the row number from the date

For consecutive dates, each date advances by one day while the row number also advances by one. Subtracting the row number in days from the date therefore produces the same grouping value for each uninterrupted streak.

For example:

| activity_date | rn | date minus rn days |
|---|---:|---|
| 2026-01-01 | 1 | 2025-12-31 |
| 2026-01-02 | 2 | 2025-12-31 |
| 2026-01-03 | 3 | 2025-12-31 |
| 2026-01-06 | 4 | 2026-01-02 |
| 2026-01-07 | 5 | 2026-01-02 |

The grouping value stays the same inside a streak and changes when a gap occurs.

### Complete query

```sql
WITH numbered_dates AS (
    SELECT
        user_id,
        activity_date,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY activity_date
        ) AS rn
    FROM user_activity
),
grouped_dates AS (
    SELECT
        user_id,
        activity_date,
        activity_date - rn * INTERVAL '1 day' AS streak_group
    FROM numbered_dates
)
SELECT
    user_id,
    MIN(activity_date) AS streak_start,
    MAX(activity_date) AS streak_end,
    COUNT(*) AS streak_length
FROM grouped_dates
GROUP BY user_id, streak_group
ORDER BY user_id, streak_start;
```

This query uses PostgreSQL-style date arithmetic.

### Expected result

| user_id | streak_start | streak_end | streak_length |
|---|---|---|---:|
| U101 | 2026-01-01 | 2026-01-03 | 3 |
| U101 | 2026-01-06 | 2026-01-07 | 2 |
| U102 | 2026-01-01 | 2026-01-01 | 1 |
| U102 | 2026-01-03 | 2026-01-04 | 2 |

**Why it works:** Each consecutive date advances at the same rate as the row number, so subtracting the row number creates a constant value for the streak.

**Important:** This pattern assumes one row per user per date. Deduplicate first if multiple activity records can occur on the same date.

## 7. Pattern 5: Find the Longest Consecutive Streak

Once the streaks have been grouped, finding the longest streak is straightforward.

Using the `grouped_dates` approach above:

```sql
WITH numbered_dates AS (
    SELECT
        user_id,
        activity_date,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY activity_date
        ) AS rn
    FROM user_activity
),
grouped_dates AS (
    SELECT
        user_id,
        activity_date,
        activity_date - rn * INTERVAL '1 day' AS streak_group
    FROM numbered_dates
),
streaks AS (
    SELECT
        user_id,
        MIN(activity_date) AS streak_start,
        MAX(activity_date) AS streak_end,
        COUNT(*) AS streak_length
    FROM grouped_dates
    GROUP BY user_id, streak_group
),
ranked_streaks AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY streak_length DESC, streak_start
        ) AS rn
    FROM streaks
)
SELECT
    user_id,
    streak_start,
    streak_end,
    streak_length
FROM ranked_streaks
WHERE rn = 1;
```

This returns one longest streak per user. If multiple streaks have the same length, it selects the one with the earliest start date.

If the requirement is to return **all** equally long streaks, use `RANK()` ordered by `streak_length DESC` and filter for rank 1.

## 8. Pattern 6: Find Missing Dates Within a Date Range

**Use when:** You need to find dates with no activity between a known start and end date.

The general approach is:

1. Generate every expected date in the range.
2. Compare the generated dates with the dates present in the activity table.
3. Return dates without a matching record.

### PostgreSQL example

```sql
WITH RECURSIVE calendar AS (
    SELECT DATE '2026-01-01' AS calendar_date

    UNION ALL

    SELECT calendar_date + INTERVAL '1 day'
    FROM calendar
    WHERE calendar_date < DATE '2026-01-07'
)
SELECT c.calendar_date
FROM calendar c
WHERE NOT EXISTS (
    SELECT 1
    FROM user_activity u
    WHERE u.user_id = 'U101'
      AND u.activity_date = c.calendar_date
)
ORDER BY c.calendar_date;
```

The calendar CTE generates dates from January 1 through January 7, inclusive. The final query returns the dates on which U101 had no activity.

### Expected result

| calendar_date |
|---|
| 2026-01-04 |
| 2026-01-05 |

This approach assumes the activity table contains date values without a time component.

**Important:** If you need to find missing dates across an entire dataset, decide whether the expected calendar is global, per user, per store, or per another business entity. The definition of a missing record depends on the expected data.

## 9. Pattern 7: Find Users Active on at Least N Consecutive Days

Suppose the requirement is to find users with a streak of at least three days.

Reuse the `streaks` CTE from Pattern 5, then filter:

```sql
SELECT
    user_id,
    streak_start,
    streak_end,
    streak_length
FROM streaks
WHERE streak_length >= 3;
```

For a standalone query, include the CTE definitions for `numbered_dates`, `grouped_dates`, and `streaks` before this final `SELECT`.

This pattern is common in:
- Daily login analysis.
- Customer engagement.
- Repeated sales activity.
- Monitoring consecutive successful pipeline runs.

## 10. Common Mistakes

### Mistake 1: Forgetting to partition by entity

Incorrect:

```sql
LAG(activity_date) OVER (
    ORDER BY activity_date
)
```

If the table contains multiple users, this can compare a user's date with a different user's date.

Correct:

```sql
LAG(activity_date) OVER (
    PARTITION BY user_id
    ORDER BY activity_date
)
```

### Mistake 2: Treating the previous row as the previous day

`LAG()` returns the preceding row, not necessarily the previous calendar day. Compare the dates explicitly when you need consecutive-day logic.

### Mistake 3: Ignoring duplicate dates

If one user has multiple records on the same date, row-based streak calculations may produce incorrect results. Deduplicate to one row per user and date first.

### Mistake 4: Confusing elapsed days with missing dates

January 1 to January 4 is three elapsed days, but only two dates lie strictly between them: January 2 and January 3.

### Mistake 5: Using a date-minus-row-number pattern without checking assumptions

The technique works when the sequence is based on one row per entity per calendar date and dates are ordered correctly. It should be adapted for duplicate dates or other sequence definitions.

### Mistake 6: Forgetting database-specific syntax

Date addition, interval arithmetic, and date generation vary across PostgreSQL, MySQL, SQL Server, and Databricks SQL.

## 11. Interview Questions

**Q1. How do you identify consecutive dates?**

Use `LAG()` to retrieve the preceding date and compare it with the current date using date arithmetic.

**Q2. How do you calculate the gap between transactions?**

Use `LAG()` partitioned by the customer and ordered by transaction date, then calculate the difference between the current and preceding dates.

**Q3. How do you find the longest consecutive streak?**

Group consecutive dates, calculate the length of each group, then rank the streaks by length.

**Q4. What is the date-minus-row-number technique?**

For an ordered sequence of consecutive dates, subtracting the row number in days from each date produces a constant grouping key for each uninterrupted streak.

**Q5. How do you find missing dates?**

Generate the expected dates for the range and use `NOT EXISTS` or an anti join to identify dates without matching activity.

**Q6. Why should you partition by `user_id`?**

Each user's sequence must be evaluated independently. Otherwise, dates belonging to different users can affect the comparison.

**Q7. How do duplicate activity dates affect streak calculations?**

They can break row-based assumptions and inflate streak counts. Deduplicate to one row per user and date before grouping consecutive days.

**Q8. How do you return all longest streaks when there is a tie?**

Calculate streak lengths, apply `RANK()` partitioned by user and ordered by length descending, and retain rank 1.

## 12. Practice Questions

Try writing these queries without looking at the examples.

1. Find all user activity dates that are consecutive.
2. Calculate the gap between each customer's transactions.
3. Find customer transaction gaps greater than seven elapsed days.
4. Group each user's consecutive activity dates into streaks.
5. Find the longest activity streak for each user.
6. Return users with at least five consecutive activity days.
7. Find missing dates between January 1 and January 31.
8. Find the longest streak when multiple records may exist per user per day.
9. Return all longest streaks when multiple streaks tie.
10. Explain the difference between elapsed days and missing calendar dates.

## 13. Quick Revision

| Requirement | Common approach |
|---|---|
| Compare with the preceding date | `LAG()` |
| Find consecutive dates | `LAG()` plus date arithmetic |
| Calculate gaps between events | `LAG()` plus date difference |
| Group consecutive dates | `ROW_NUMBER()` date-grouping technique |
| Find the longest streak | Group streaks, count rows, rank lengths |
| Find missing dates | Generate a calendar and use `NOT EXISTS` |
| Find streaks of at least N days | Group streaks and filter by count |
| Handle multiple users | `PARTITION BY user_id` |
| Avoid duplicate-date errors | Deduplicate before streak analysis |

### Self-check

- [ ] Can you use `LAG()` to compare consecutive records?
- [ ] Can you distinguish elapsed days from missing dates?
- [ ] Can you group consecutive dates into streaks?
- [ ] Can you find the longest streak per user?
- [ ] Can you generate a calendar and find missing dates?
- [ ] Can you explain why duplicates and partitioning matter?
- [ ] Can you adapt date arithmetic to your database?

**Interview takeaway:** Start by defining what “consecutive” means in the problem. Then use `LAG()` for neighboring-row comparisons or a streak-grouping technique when you need start dates, end dates, and streak lengths.