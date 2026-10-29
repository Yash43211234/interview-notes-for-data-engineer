# Date and Time in Python (`datetime`)

## 1. What Is `datetime`?

Python's built-in `datetime` module provides classes and functions for working with dates, times, durations, and time zones.

In data engineering, date and time operations are common when:
- Filtering records by date.
- Processing incremental data loads.
- Creating daily or monthly reports.
- Calculating time differences.
- Handling timestamps from APIs and databases.
- Converting timestamps between time zones.

```python
import datetime

today = datetime.date.today()
print(today)
```

You can also import individual classes:

```python
from datetime import date, time, datetime, timedelta, timezone
```

This is often more convenient when writing application code.

---

## 2. Important Classes

| Class | Purpose | Example |
|---|---|---|
| `date` | Represents a calendar date | `2026-10-29` |
| `time` | Represents a time of day | `14:30:00` |
| `datetime` | Combines date and time | `2026-10-29 14:30:00` |
| `timedelta` | Represents a duration or difference | `3 days` |
| `timezone` | Represents a fixed UTC offset | UTC, UTC+05:30 |

These classes represent different concepts. For example, a `date` does not contain hours or minutes, while a `datetime` contains both a date and a time.

---

## 3. Working with Dates

### Get the Current Date

```python
from datetime import date

today = date.today()
print(today)
print(today.year)
print(today.month)
print(today.day)
```

Example output:

```text
2026-10-29
2026
10
29
```

The output depends on the system's local date.

### Create a Specific Date

```python
from datetime import date

joining_date = date(2025, 12, 11)

print(joining_date)
print(joining_date.year)
print(joining_date.month)
print(joining_date.day)
```

Output:

```text
2025-12-11
2025
12
11
```

The arguments are `year`, `month`, and `day`, in that order.

Invalid dates, such as `date(2026, 2, 30)`, raise a `ValueError`.

### Compare Dates

```python
from datetime import date

date1 = date(2026, 1, 10)
date2 = date(2026, 5, 20)

print(date1 < date2)
print(date1 == date2)
```

Output:

```text
True
False
```

Date objects support comparisons such as `<`, `>`, `<=`, and `>=`.

---

## 4. Working with Time

The `time` class represents the time of day independently of a date.

```python
from datetime import time

t = time(14, 30, 45)

print(t)
print(t.hour)
print(t.minute)
print(t.second)
```

Output:

```text
14:30:45
14
30
45
```

You can also include microseconds:

```python
t = time(14, 30, 45, 500000)
print(t)
```

Output:

```text
14:30:45.500000
```

A `time` object does not represent a complete point on a timeline because it has no date attached.

---

## 5. Working with `datetime`

The `datetime` class combines a date and a time.

### Get the Current Date and Time

```python
from datetime import datetime

now = datetime.now()

print(now)
print(now.year)
print(now.month)
print(now.day)
print(now.hour)
print(now.minute)
```

`datetime.now()` without a time zone returns the current local date and time as a **naive datetime**.

A naive datetime does not contain time-zone information.

### Create a Specific Datetime

```python
from datetime import datetime

event_time = datetime(2026, 10, 29, 14, 30, 0)

print(event_time)
```

Output:

```text
2026-10-29 14:30:00
```

The positional arguments are year, month, day, hour, minute, and second.

### Combine a Date and a Time

```python
from datetime import date, time, datetime

event_date = date(2026, 10, 29)
event_clock = time(9, 30)

event_datetime = datetime.combine(event_date, event_clock)

print(event_datetime)
```

Output:

```text
2026-10-29 09:30:00
```

---

## 6. Converting Strings to Datetimes

Data from CSV files, databases, and APIs may contain dates as strings.

Use `strptime()` to parse a string according to a specified format.

```python
from datetime import datetime

date_string = "2026-10-29 14:30:00"

parsed_datetime = datetime.strptime(
    date_string,
    "%Y-%m-%d %H:%M:%S"
)

print(parsed_datetime)
print(type(parsed_datetime))
```

Output:

```text
2026-10-29 14:30:00
<class 'datetime.datetime'>
```

### Common Format Directives

| Directive | Meaning | Example |
|---|---|---|
| `%Y` | Four-digit year | `2026` |
| `%y` | Two-digit year | `26` |
| `%m` | Month number | `10` |
| `%d` | Day of month | `29` |
| `%H` | Hour, 24-hour clock | `14` |
| `%I` | Hour, 12-hour clock | `02` |
| `%M` | Minute | `30` |
| `%S` | Second | `45` |
| `%f` | Microseconds | `123456` |
| `%p` | AM or PM | `PM` |
| `%a` | Abbreviated weekday | `Thu` |
| `%A` | Full weekday | `Thursday` |
| `%b` | Abbreviated month | `Oct` |
| `%B` | Full month | `October` |
| `%z` | UTC offset | `+0530` |
| `%Z` | Time-zone name | `UTC` |

The format must match the input string.

Example:

```python
from datetime import datetime

date_string = "29/10/2026"

parsed_date = datetime.strptime(date_string, "%d/%m/%Y")

print(parsed_date)
```

Output:

```text
2026-10-29 00:00:00
```

The result is a `datetime` object, with the time set to midnight.

**Common mistake:** Using `%m/%d/%Y` for a string written as `29/10/2026`. The day and month positions would be incorrect, and this particular input would be invalid under that format.

---

## 7. Converting Datetimes to Strings

Use `strftime()` to format a datetime as a string.

```python
from datetime import datetime

now = datetime(2026, 10, 29, 14, 30)

formatted = now.strftime("%d-%m-%Y %I:%M %p")

print(formatted)
```

Output:

```text
29-10-2026 02:30 PM
```

### `strptime()` vs. `strftime()`

| Method | Conversion | Direction |
|---|---|---|
| `strptime()` | String to datetime | Parse |
| `strftime()` | Datetime to string | Format |

Memory trick:

- **`p`** in `strptime` stands for parsing.
- **`f`** in `strftime` stands for formatting.

---

## 8. ISO 8601 Format

ISO 8601 is a widely used standard for representing dates and timestamps.

Examples:

```text
2026-10-29
2026-10-29T14:30:00
2026-10-29T14:30:00+05:30
2026-10-29T09:00:00Z
```

The `Z` suffix indicates UTC.

### Convert to an ISO String

```python
from datetime import datetime

dt = datetime(2026, 10, 29, 14, 30)

print(dt.isoformat())
```

Output:

```text
2026-10-29T14:30:00
```

### Parse an ISO String

```python
from datetime import datetime

dt = datetime.fromisoformat("2026-10-29T14:30:00+05:30")

print(dt)
print(dt.utcoffset())
```

Output:

```text
2026-10-29 14:30:00+05:30
5:30:00
```

Python's `fromisoformat()` supports common ISO-style formats, but it is not a general parser for every possible date string. Use a suitable parser or explicit format when working with other formats.

---

## 9. Adding and Subtracting Time

Use `timedelta` to represent durations.

```python
from datetime import datetime, timedelta

start = datetime(2026, 10, 29, 10, 0)

later = start + timedelta(days=3)
earlier = start - timedelta(hours=2)

print(later)
print(earlier)
```

Output:

```text
2026-11-01 10:00:00
2026-10-29 08:00:00
```

### Common `timedelta` Arguments

```python
timedelta(
    days=2,
    hours=3,
    minutes=15,
    seconds=30
)
```

You can use days, seconds, microseconds, milliseconds, minutes, hours, and weeks.

### Calculate the Difference Between Two Datetimes

```python
from datetime import datetime

start = datetime(2026, 10, 1, 9, 0)
end = datetime(2026, 10, 3, 12, 30)

difference = end - start

print(difference)
print(difference.days)
print(difference.total_seconds())
```

Output:

```text
2 days, 3:30:00
2
189000.0
```

**Important distinction:**

- `difference.days` returns the whole-day component of a `timedelta`.
- `difference.total_seconds()` returns the entire duration in seconds.

For example:

```python
from datetime import timedelta

duration = timedelta(hours=30)

print(duration.days)
print(duration.total_seconds())
```

Output:

```text
1
108000.0
```

The duration is 30 hours, but the `.days` component is only `1`.

---

## 10. Working with Time Zones

Time zones are especially important in distributed data pipelines. Servers, applications, and users may operate in different regions.

### Naive vs. Aware Datetimes

A **naive datetime** does not contain usable time-zone information.

```python
from datetime import datetime

dt = datetime(2026, 10, 29, 14, 30)

print(dt.tzinfo)
```

Output:

```text
None
```

An **aware datetime** contains time-zone information and can identify a point in time relative to UTC.

### Create a UTC Datetime

```python
from datetime import datetime, timezone

utc_now = datetime.now(timezone.utc)

print(utc_now)
```

This returns the current time as an aware UTC datetime.

Prefer `datetime.now(timezone.utc)` over the older `datetime.utcnow()` approach, which returns a naive datetime.

### Convert UTC to Another Time Zone

Use `zoneinfo` for named time zones. It is available in the Python standard library starting with Python 3.9.

```python
from datetime import datetime, timezone
from zoneinfo import ZoneInfo

utc_time = datetime(2026, 10, 29, 9, 0, tzinfo=timezone.utc)

india_time = utc_time.astimezone(ZoneInfo("Asia/Kolkata"))

print(india_time)
```

Output:

```text
2026-10-29 14:30:00+05:30
```

India is UTC+05:30, so 09:00 UTC corresponds to 14:30 in Kolkata.

On some systems, the time-zone database may need to be installed separately, for example through the `tzdata` package.

### `replace()` vs. `astimezone()`

These methods do different things.

`replace(tzinfo=...)` attaches or changes time-zone metadata without converting the clock time.

```python
dt = datetime(2026, 10, 29, 9, 0)
```

Attaching UTC:

```python
dt_utc = dt.replace(tzinfo=timezone.utc)
```

This declares that the existing 09:00 clock time should be interpreted as UTC.

By contrast, `astimezone()` converts an already-aware datetime to another time zone while preserving the represented instant.

```python
india_time = dt_utc.astimezone(ZoneInfo("Asia/Kolkata"))
```

Output:

```text
2026-10-29 14:30:00+05:30
```

Do not use `replace(tzinfo=timezone.utc)` to convert a local clock time into UTC. First determine which time zone the original naive datetime represents.

---

## 11. Comparing Datetimes

Python supports comparing datetimes, but mixing naive and aware datetime objects is usually an error.

```python
from datetime import datetime, timezone

naive = datetime(2026, 10, 29, 10, 0)
aware = datetime(2026, 10, 29, 10, 0, tzinfo=timezone.utc)

# naive < aware  # Raises TypeError
```

Convert timestamps to a consistent time-zone-aware representation before comparing them.

For example:

```python
from datetime import datetime, timezone

first = datetime(2026, 10, 29, 10, 0, tzinfo=timezone.utc)
second = datetime(2026, 10, 29, 11, 0, tzinfo=timezone.utc)

print(first < second)
```

Output:

```text
True
```

For reliable timestamp comparisons in distributed systems, UTC is a common standard representation.

---

## 12. Extracting Date Parts

You can access the individual components of a datetime.

```python
from datetime import datetime

dt = datetime(2026, 10, 29, 14, 30, 45)

print(dt.year)
print(dt.month)
print(dt.day)
print(dt.hour)
print(dt.minute)
print(dt.second)
print(dt.weekday())
```

Output:

```text
2026
10
29
14
30
45
3
```

`weekday()` returns an integer from `0` to `6`:

- Monday = `0`
- Tuesday = `1`
- ...
- Sunday = `6`

To get the ISO weekday number, where Monday is `1` and Sunday is `7`, use `isoweekday()`.

```python
print(dt.isoweekday())
```

Output:

```text
4
```

October 29, 2026, is a Thursday.

---

## 13. Practical Example: Filtering Records by Date

Suppose a CSV file contains transaction timestamps.

```python
from datetime import datetime

transactions = [
    {"id": 1, "timestamp": "2026-10-28 10:30:00"},
    {"id": 2, "timestamp": "2026-10-29 12:00:00"},
    {"id": 3, "timestamp": "2026-10-30 15:00:00"}
]

start = datetime(2026, 10, 29)
end = datetime(2026, 10, 30)

filtered = []

for transaction in transactions:
    transaction_time = datetime.strptime(
        transaction["timestamp"],
        "%Y-%m-%d %H:%M:%S"
    )

    if start <= transaction_time < end:
        filtered.append(transaction)

print(filtered)
```

Output:

```text
[{'id': 2, 'timestamp': '2026-10-29 12:00:00'}]
```

The condition includes timestamps at the start and excludes timestamps at the end.

Using a half-open interval, `[start, end)`, is useful for daily and monthly data processing because adjacent windows do not overlap.

For a daily pipeline, a robust pattern is:

```python
start = datetime(2026, 10, 29)
end = datetime(2026, 10, 30)

# Include timestamps >= start and < end
```

For production systems, ensure that the timestamps and boundaries use compatible time zones.

---

## 14. Common Mistakes

1. **Confusing `strptime()` and `strftime()`:** One parses strings; the other formats datetime objects.
2. **Assuming all datetimes have time zones:** `datetime.now()` without a time zone returns a naive datetime.
3. **Mixing naive and aware datetimes:** This can raise `TypeError` during comparisons or subtraction.
4. **Using `.days` to calculate the complete duration:** Use `.total_seconds()` when you need the entire difference in seconds.
5. **Assuming `replace(tzinfo=...)` converts time:** It attaches or replaces metadata; it does not adjust the clock time.
6. **Ignoring input formats:** A date such as `03/04/2026` can mean March 4 or April 3, depending on the format convention.
7. **Using an inclusive end boundary for consecutive batches:** This can duplicate records between adjacent date windows. Prefer `start <= timestamp < end`.
8. **Ignoring daylight-saving transitions:** Named time zones may have changing UTC offsets. Use `zoneinfo` for regions that observe these changes.
9. **Treating a date as a timestamp:** A calendar date alone does not specify a precise instant in time.

---

## 15. Interview Questions

### Q1. What is the difference between `date` and `datetime`?

`date` stores a calendar date. `datetime` stores both the date and the time.

### Q2. What is the difference between `datetime.now()` and `datetime.utcnow()`?

`datetime.now()` returns the local time as a naive datetime unless a time zone is supplied. `datetime.utcnow()` returns a naive datetime representing UTC and is deprecated since Python 3.12. Prefer `datetime.now(timezone.utc)` for an aware UTC datetime.

### Q3. What is a `timedelta`?

A `timedelta` represents a duration, such as the difference between two dates or datetimes, or a duration to add to or subtract from one.

### Q4. How do you convert a string into a datetime?

Use `datetime.strptime()` with a format string matching the input.

### Q5. How do you format a datetime as a string?

Use `datetime.strftime()`.

### Q6. What is the difference between naive and aware datetimes?

A naive datetime lacks usable time-zone information. An aware datetime has time-zone information sufficient to identify its position relative to UTC.

### Q7. Why is UTC commonly used in data engineering?

UTC provides a consistent reference for timestamps collected from different regions, simplifying comparisons, storage, and cross-system processing.

### Q8. What is the difference between `replace(tzinfo=...)` and `astimezone()`?

`replace(tzinfo=...)` changes the time-zone metadata without converting the clock time. `astimezone()` converts an aware datetime to another time zone while preserving the instant.

### Q9. How would you filter records for one day?

Use a start boundary inclusive and the next day's boundary exclusive:

```python
start <= timestamp < end
```

Ensure all timestamps and boundaries are expressed consistently.

### Q10. How do you handle time zones in Python?

Use aware datetimes, `datetime.timezone` for fixed offsets, and `zoneinfo.ZoneInfo` for named regional time zones.

---

## 16. Practice Tasks

Try solving these without looking at the examples above.

1. Print today's date and the current local time.
2. Parse `"29-10-2026"` into a datetime object.
3. Format a datetime as `"29 Oct 2026, 02:30 PM"`.
4. Calculate the number of seconds between two timestamps.
5. Add 30 days to a given date.
6. Extract the month and weekday from a datetime.
7. Convert `09:00 UTC` into `Asia/Kolkata` time.
8. Filter transactions occurring during a specific day.
9. Explain why comparing a naive datetime with an aware datetime can fail.
10. Create a daily batch window using an inclusive start and exclusive end.

---

## 17. Quick Revision

- **`date`**: Calendar date.
- **`time`**: Time of day.
- **`datetime`**: Date and time combined.
- **`timedelta`**: Duration or time difference.
- **`strptime()`**: String to datetime.
- **`strftime()`**: Datetime to string.
- **`isoformat()`**: Formats a date or datetime using an ISO-style representation.
- **`timezone.utc`**: Fixed UTC time zone.
- **`ZoneInfo("Asia/Kolkata")`**: Named regional time zone.
- **`astimezone()`**: Converts an aware datetime to another time zone.
- **`replace(tzinfo=...)`**: Changes time-zone metadata without converting the clock time.
- **Half-open interval**: `start <= timestamp < end`, useful for reliable batch filtering.

**Most important interview takeaway:** Understand string parsing and formatting, `timedelta` calculations, naive versus aware datetimes, time-zone conversion, and how to define reliable timestamp boundaries for incremental data processing.