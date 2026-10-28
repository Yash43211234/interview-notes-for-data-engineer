# Retention and Cohorts

## 1. What Is Retention?

Retention measures how many users or customers return after their first activity or within a defined period.

It helps answer questions such as:
- How many customers return after their first purchase?
- How many users remain active after 7 or 30 days?
- Are customers continuing to use a product over time?

**Example:** If 100 users sign up in January and 40 of them return in February, the monthly retention rate is 40%, assuming the calculation uses the January signup cohort.

## 2. What Is a Cohort?

A cohort is a group of users who share a common characteristic or starting event during a specific period.

Common cohort types:

- **Acquisition cohort:** Users grouped by signup date or month.
- **First-purchase cohort:** Customers grouped by the month of their first purchase.
- **Activity cohort:** Users grouped by the period in which they first became active.

Cohort analysis tracks the behavior of each group over subsequent periods.

## 3. Retention Rate Formula

\[
\text{Retention Rate} =
\frac{\text{Cohort Users Active in Period}}
{\text{Total Users in Original Cohort}}
\times 100
\]

For example, if a cohort contains 200 users and 70 return in the following month:

\[
\text{Retention Rate} = \frac{70}{200}\times100=35\%
\]

The exact definition of activity and the retention period must be consistent.

## 4. Sample Table

Assume a table named `user_activity`:

| user_id | activity_date |
|---|---|
| 101 | 2026-01-05 |
| 101 | 2026-02-10 |
| 102 | 2026-01-12 |
| 103 | 2026-02-02 |
| 103 | 2026-03-15 |
| 104 | 2026-01-20 |

Each row represents a user's activity on a particular date.

## 5. Find Each User's First Activity Month

The first activity month defines the acquisition cohort in this example.

```sql
SELECT
    user_id,
    MIN(activity_date) AS first_activity_date,
    DATE_TRUNC('month', MIN(activity_date)) AS cohort_month
FROM user_activity
GROUP BY user_id;
```

**Key point:** `MIN(activity_date)` finds the earliest recorded activity for each user. This represents signup only if the table records signup events or the first activity is explicitly being used as a proxy.

`DATE_TRUNC` is supported by PostgreSQL and several other SQL engines, but syntax may differ across platforms.

## 6. Calculate Monthly Cohort Retention

The following PostgreSQL-style query calculates the percentage of users in each first-activity cohort who were active in each month, including their cohort month.

```sql
WITH first_activity AS (
    SELECT
        user_id,
        DATE_TRUNC('month', MIN(activity_date)) AS cohort_month
    FROM user_activity
    GROUP BY user_id
),
monthly_activity AS (
    SELECT DISTINCT
        user_id,
        DATE_TRUNC('month', activity_date) AS activity_month
    FROM user_activity
),
cohort_activity AS (
    SELECT
        f.cohort_month,
        m.activity_month,
        COUNT(DISTINCT m.user_id) AS active_users
    FROM first_activity f
    JOIN monthly_activity m
        ON f.user_id = m.user_id
    GROUP BY
        f.cohort_month,
        m.activity_month
),
cohort_size AS (
    SELECT
        cohort_month,
        COUNT(*) AS total_users
    FROM first_activity
    GROUP BY cohort_month
)
SELECT
    ca.cohort_month,
    ca.activity_month,
    ca.active_users,
    cs.total_users,
    ROUND(
        100.0 * ca.active_users / cs.total_users,
        2
    ) AS retention_rate
FROM cohort_activity ca
JOIN cohort_size cs
    ON ca.cohort_month = cs.cohort_month
WHERE ca.activity_month >= ca.cohort_month
ORDER BY
    ca.cohort_month,
    ca.activity_month;
```

### How it works

1. `first_activity`: Identifies each user's cohort month.
2. `monthly_activity`: Removes duplicate user-month activity records.
3. `cohort_activity`: Counts active users by cohort and activity month.
4. `cohort_size`: Calculates the original size of each cohort.
5. Final query: Divides active users by the original cohort size.

**Important:** This query returns months in which activity occurred. It does not automatically generate rows for months with zero activity. A calendar table or generated month series is needed to display those periods as 0% retention.

## 7. Calculate Retention by Month Number

Product-company interviews often ask for retention in Month 0, Month 1, Month 2, and so on.

The month number is the number of calendar-month boundaries between the cohort month and the activity month.

```sql
WITH first_activity AS (
    SELECT
        user_id,
        DATE_TRUNC('month', MIN(activity_date)) AS cohort_month
    FROM user_activity
    GROUP BY user_id
),
monthly_activity AS (
    SELECT DISTINCT
        user_id,
        DATE_TRUNC('month', activity_date) AS activity_month
    FROM user_activity
),
cohort_size AS (
    SELECT
        cohort_month,
        COUNT(*) AS total_users
    FROM first_activity
    GROUP BY cohort_month
),
retention AS (
    SELECT
        f.cohort_month,
        (
            EXTRACT(YEAR FROM m.activity_month)
            - EXTRACT(YEAR FROM f.cohort_month)
        ) * 12
        +
        (
            EXTRACT(MONTH FROM m.activity_month)
            - EXTRACT(MONTH FROM f.cohort_month)
        ) AS month_number,
        COUNT(DISTINCT f.user_id) AS active_users
    FROM first_activity f
    JOIN monthly_activity m
        ON f.user_id = m.user_id
    WHERE m.activity_month >= f.cohort_month
    GROUP BY
        f.cohort_month,
        m.activity_month
)
SELECT
    r.cohort_month,
    r.month_number,
    r.active_users,
    cs.total_users,
    ROUND(
        100.0 * r.active_users / cs.total_users,
        2
    ) AS retention_rate
FROM retention r
JOIN cohort_size cs
    ON r.cohort_month = cs.cohort_month
ORDER BY
    r.cohort_month,
    r.month_number;
```

## 8. Common Interview Questions

**Q1. What is the difference between retention and churn?**

Retention measures the proportion of an original cohort that remains active. Churn measures the proportion that stops using a product or service, according to a defined churn rule. They are complements only when their definitions and observation periods align.

**Q2. Why use `COUNT(DISTINCT user_id)`?**

A user may perform many activities in one month. Counting distinct users prevents multiple events from inflating the number of active users.

**Q3. Why calculate cohort size separately?**

Retention uses the original cohort size as its denominator. Calculating it separately keeps the denominator consistent across periods.

**Q4. What is the difference between cohort month and activity month?**

- `cohort_month`: The month when the user first entered the cohort.
- `activity_month`: The month when the user performed an activity.

**Q5. What is Month 0 retention?**

Month 0 retention is the percentage of the original cohort active during its cohort month. It is generally 100% when the cohort is defined by the same activity used to measure retention.

**Q6. What is the difference between classic and rolling retention?**

- **Classic retention:** A user is counted if active in the specified period.
- **Rolling retention:** A user is counted if active on or after a specified point, depending on the chosen definition.

Rolling retention can be higher than classic retention because it may include users who return in later periods.

## 9. Common Mistakes

- Counting activity events instead of distinct users.
- Using the current active-user count as the denominator.
- Defining cohorts by signup month but measuring retention from first activity without clarifying the difference.
- Mixing calendar-month retention with exact 30-day retention.
- Ignoring months with zero activity.
- Comparing incomplete recent cohorts with older, fully observed cohorts.
- Using inconsistent time zones or date boundaries.

## 10. Interview Practice

1. Calculate Day 1 retention for users who signed up on a particular date.
2. Find monthly retention for each signup cohort.
3. Identify the cohort with the highest Month 1 retention.
4. Calculate retention for users who made a second purchase within 30 days.
5. Build a cohort retention matrix with cohorts as rows and Month 0, Month 1, Month 2, and Month 3 as columns.
6. Find users who were active in Month 1 but inactive in Month 2.

## Quick Revision

- **Cohort:** A group of users sharing a starting event or period.
- **Retention:** The percentage of the original cohort that remains active.
- **Cohort size:** The number of users in the original group.
- **Activity month:** The period in which user activity occurs.
- **Month number:** The offset between the cohort month and activity month.
- **Key SQL techniques:** `MIN()`, `DATE_TRUNC()`, `COUNT(DISTINCT)`, CTEs, joins, and conditional aggregation.
