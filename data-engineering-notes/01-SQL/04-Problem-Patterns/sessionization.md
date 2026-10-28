# Sessionization in SQL

## 1. What Is Sessionization?

Sessionization is the process of grouping user activities into sessions based on the time gap between consecutive events.

A session represents a period of user activity, such as browsing a website, using an application, or interacting with a platform.

**Example:** If a user performs an event and then performs another event within 30 minutes, both events belong to the same session. If the gap exceeds 30 minutes, a new session begins.

Sessionization is commonly used in:
- Website and application analytics
- User behavior analysis
- Clickstream data processing
- Session count and duration calculations
- Data engineering event pipelines

## 2. Sessionization Rule

Assume the inactivity threshold is 30 minutes.

| user_id | event_time | Gap from previous event | Session |
|---|---|---|---|
| 101 | 10:00 | — | 1 |
| 101 | 10:10 | 10 minutes | 1 |
| 101 | 10:25 | 15 minutes | 1 |
| 101 | 11:00 | 35 minutes | 2 |
| 101 | 11:15 | 15 minutes | 2 |

The first event starts a session. Each subsequent event starts a new session only when the gap from the previous event is greater than 30 minutes.

**Important:** A gap of exactly 30 minutes remains in the same session under this rule. Use `>` rather than `>=`.

## 3. Sample Table

Assume a table named `events`.

| user_id | event_time |
|---|---|
| 101 | 2026-01-01 10:00:00 |
| 101 | 2026-01-01 10:10:00 |
| 101 | 2026-01-01 10:25:00 |
| 101 | 2026-01-01 11:00:00 |
| 101 | 2026-01-01 11:15:00 |
| 102 | 2026-01-01 09:00:00 |
| 102 | 2026-01-01 09:20:00 |
| 102 | 2026-01-01 10:00:00 |

Assume each row represents one event and `event_time` is a valid timestamp.

## 4. Step 1 — Find the Previous Event

Use `LAG()` to retrieve the previous event timestamp for each user.

```sql
SELECT
    user_id,
    event_time,
    LAG(event_time) OVER (
        PARTITION BY user_id
        ORDER BY event_time
    ) AS previous_event_time
FROM events;
```

### How it works

- `PARTITION BY user_id`: Processes each user separately.
- `ORDER BY event_time`: Orders events chronologically.
- `LAG(event_time)`: Returns the timestamp of the previous event.
- The first event for each user has a `NULL` previous timestamp.

## 5. Step 2 — Identify New Sessions

Compare each event with its previous event. Mark the first event or an event with a gap greater than 30 minutes as a new session.

```sql
WITH previous_events AS (
    SELECT
        user_id,
        event_time,
        LAG(event_time) OVER (
            PARTITION BY user_id
            ORDER BY event_time
        ) AS previous_event_time
    FROM events
)
SELECT
    user_id,
    event_time,
    previous_event_time,
    CASE
        WHEN previous_event_time IS NULL THEN 1
        WHEN event_time > previous_event_time
             + INTERVAL '30 minutes' THEN 1
        ELSE 0
    END AS new_session_flag
FROM previous_events;
```

The `INTERVAL '30 minutes'` syntax is PostgreSQL-style. Other SQL engines, including SQL Server and Spark SQL, use different timestamp arithmetic syntax.

## 6. Step 3 — Assign Session IDs

Use a running sum of the new-session flags to assign a session number to each event.

```sql
WITH previous_events AS (
    SELECT
        user_id,
        event_time,
        LAG(event_time) OVER (
            PARTITION BY user_id
            ORDER BY event_time
        ) AS previous_event_time
    FROM events
),
session_flags AS (
    SELECT
        user_id,
        event_time,
        CASE
            WHEN previous_event_time IS NULL THEN 1
            WHEN event_time > previous_event_time
                 + INTERVAL '30 minutes' THEN 1
            ELSE 0
        END AS new_session_flag
    FROM previous_events
)
SELECT
    user_id,
    event_time,
    SUM(new_session_flag) OVER (
        PARTITION BY user_id
        ORDER BY event_time
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS session_id
FROM session_flags
ORDER BY user_id, event_time;
```

### Why use a running sum?

Suppose the session flags are:

| Event | New session flag | Running sum |
|---|---:|---:|
| Event 1 | 1 | 1 |
| Event 2 | 0 | 1 |
| Event 3 | 0 | 1 |
| Event 4 | 1 | 2 |
| Event 5 | 0 | 2 |
| Event 6 | 1 | 3 |

Every time a new session begins, the running sum increases by one.

**Interview pattern to remember:**

`LAG()` → identify gaps → flag new groups → running `SUM()`.

## 7. Calculate Session Start, End, and Duration

Once each event has a session ID, aggregate the events to calculate session-level metrics.

```sql
WITH previous_events AS (
    SELECT
        user_id,
        event_time,
        LAG(event_time) OVER (
            PARTITION BY user_id
            ORDER BY event_time
        ) AS previous_event_time
    FROM events
),
session_flags AS (
    SELECT
        user_id,
        event_time,
        CASE
            WHEN previous_event_time IS NULL THEN 1
            WHEN event_time > previous_event_time
                 + INTERVAL '30 minutes' THEN 1
            ELSE 0
        END AS new_session_flag
    FROM previous_events
),
numbered_events AS (
    SELECT
        user_id,
        event_time,
        SUM(new_session_flag) OVER (
            PARTITION BY user_id
            ORDER BY event_time
            ROWS BETWEEN UNBOUNDED PRECEDING
                     AND CURRENT ROW
        ) AS session_id
    FROM session_flags
)
SELECT
    user_id,
    session_id,
    MIN(event_time) AS session_start,
    MAX(event_time) AS session_end,
    COUNT(*) AS event_count,
    EXTRACT(
        EPOCH FROM MAX(event_time) - MIN(event_time)
    ) / 60.0 AS session_duration_minutes
FROM numbered_events
GROUP BY user_id, session_id
ORDER BY user_id, session_id;
```

**Note:** This calculates the elapsed time between the first and last recorded events. It does not add the 30-minute inactivity threshold to the session end.

A session containing one event has a recorded-event duration of zero minutes, even though the user may have remained active afterward.

## 8. Common Interview Questions

### Q1. Why is `LAG()` used in sessionization?

It allows us to compare each event timestamp with the immediately preceding event for the same user.

### Q2. Why partition by user ID?

Each user's activity must be evaluated independently. Otherwise, events from different users could incorrectly affect one another's sessions.

### Q3. Why use a running sum?

The new-session flag indicates where a group begins. A cumulative sum converts those flags into sequential session numbers.

### Q4. What happens when a user has only one event?

That event starts a session, receives session ID 1, and has a recorded-event duration of zero.

### Q5. What if multiple events have the same timestamp?

Add a deterministic tie-breaker, such as `event_id`, to the window ordering when available:

```sql
ORDER BY event_time, event_id
```

This ensures consistent event ordering.

### Q6. What if the inactivity threshold is 60 minutes?

Change the threshold from 30 to 60 minutes. The sessionization logic remains the same.

### Q7. What is the difference between sessionization and deduplication?

- **Sessionization:** Groups events into sessions according to a time-gap rule.
- **Deduplication:** Removes or identifies duplicate records according to a defined uniqueness rule.

Sessionization does not automatically remove duplicate events.

## 9. Common Mistakes

- Forgetting `PARTITION BY user_id`.
- Using `LEAD()` instead of `LAG()` when the rule requires comparison with the previous event.
- Assigning session IDs directly from the event row number.
- Using `>=` instead of `>` when a gap of exactly 30 minutes should remain in the same session.
- Forgetting that a running sum requires an explicit window frame for clear row-by-row accumulation.
- Ignoring duplicate timestamps and nondeterministic ordering.
- Calculating session duration using the inactivity threshold instead of the first and last event timestamps.
- Assuming session IDs are globally unique when they restart for each user.

## 10. Interview Practice

1. Identify sessions using a 30-minute inactivity threshold.
2. Count the total number of sessions for each user.
3. Find the longest session for each user.
4. Find users with more than five sessions in a day.
5. Calculate the average number of events per session.
6. Find sessions containing at least three events.
7. Sessionize events using a 10-minute threshold.
8. Calculate daily session counts across all users.

## Quick Revision

- **Sessionization:** Grouping events based on inactivity gaps.
- **`LAG()`:** Retrieves the previous event.
- **New-session flag:** Identifies the first event or a gap exceeding the threshold.
- **Running `SUM()`:** Assigns sequential session numbers.
- **`MIN()` and `MAX()`:** Calculate session start and end.
- **`COUNT(*)`:** Counts events per session.

**Core interview pattern:** Use `LAG()` to detect a time gap, `CASE` to mark session boundaries, a running `SUM()` to assign session IDs, and `GROUP BY` to calculate session metrics.
