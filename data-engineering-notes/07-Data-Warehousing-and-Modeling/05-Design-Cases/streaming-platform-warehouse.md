# Streaming Platform Data Warehouse Design

## 1. Business Scenario

Imagine a streaming platform similar to Netflix, YouTube, or Spotify. Users can:

- Browse movies, shows, or other content.
- Play, pause, resume, and stop content.
- Subscribe to different plans.
- Make payments and renew subscriptions.
- Rate or review content.
- Watch content across multiple devices.

The business wants to analyze:

- Daily and monthly active users.
- Total viewing hours.
- Most-watched content.
- Content completion rates.
- Subscription growth and cancellations.
- Revenue by plan and region.
- User engagement and retention.
- Device and platform usage.

The goal is to design a data warehouse that supports these analytics reliably.

---

## 2. Understand the Business Processes

Before designing tables, identify the separate business processes.

| Business process | What it records |
|---|---|
| Content viewing | User playback activity |
| Playback events | Play, pause, resume, seek, stop |
| Subscription | Subscription lifecycle and plan changes |
| Payment | Payment attempts and transactions |
| Content catalog | Movies, episodes, genres, categories |
| User activity | User interactions and engagement |
| Ratings | User ratings and reviews |

Each process may require a separate fact table because its level of detail is different.

**Important:** Do not combine all these processes into one large fact table. Their grains and business meanings are different.

---

## 3. Identify the Grain

Grain defines exactly what one row represents in a fact table.

| Fact table | Grain |
|---|---|
| `fact_playback_session` | One row per playback session |
| `fact_playback_event` | One row per playback event |
| `fact_subscription_event` | One row per subscription lifecycle event |
| `fact_payment` | One row per payment transaction or attempt |
| `fact_daily_user_activity` | One row per user per day |
| `fact_content_rating` | One row per user rating submission |

### Why are separate grains important?

Suppose one user watches a movie in one session and generates 15 playback events.

- `fact_playback_session` contains one row.
- `fact_playback_event` contains 15 rows.

Joining these tables directly by session ID can repeat session-level measures 15 times.

Always aggregate data to compatible grains before combining measures.

---

## 4. Identify the Dimensions

Dimensions describe the business entities used to analyze facts.

### 4.1 `dim_user`

| Column | Description |
|---|---|
| user_key | Surrogate key |
| user_id | Source system user ID |
| user_name | User name |
| age_group | Age category, if collected and permitted |
| country | User country |
| region | User region |
| signup_date | Registration date |
| effective_from | SCD2 start timestamp |
| effective_to | SCD2 end timestamp |
| is_current | Current record indicator |

Use SCD Type 2 if historical user attributes must be preserved.

Avoid collecting unnecessary personal information. Apply privacy and access controls to user data.

### 4.2 `dim_content`

| Column | Description |
|---|---|
| content_key | Surrogate key |
| content_id | Source content ID |
| content_title | Movie or show title |
| content_type | Movie, series, episode |
| genre | Content genre |
| language | Content language |
| release_date | Original release date |
| duration_seconds | Content duration |
| maturity_rating | Content classification |

For a series, consider modeling the series and individual episodes separately if episode-level analysis is required.

### 4.3 `dim_device`

| Column | Description |
|---|---|
| device_key | Surrogate key |
| device_id | Source device identifier |
| device_type | Mobile, TV, laptop |
| operating_system | Android, iOS, Windows |
| app_version | Application version |

### 4.4 `dim_subscription_plan`

| Column | Description |
|---|---|
| plan_key | Surrogate key |
| plan_id | Source plan ID |
| plan_name | Basic, Standard, Premium |
| billing_cycle | Monthly, yearly |
| price | Listed plan price |
| video_quality | Supported quality |
| max_devices | Maximum simultaneous devices |

### 4.5 `dim_date`

Common attributes:

- date_key
- full_date
- day
- month
- quarter
- year
- week
- weekday_name

A separate time dimension can be useful for hour-of-day analysis.

### Other useful dimensions

- `dim_region`
- `dim_payment_method`
- `dim_platform`
- `dim_content_category`

Use role-playing date dimensions when a fact has multiple dates, such as subscription start date, renewal date, and cancellation date.

---

## 5. Design the Fact Tables

### 5.1 `fact_playback_session`

**Grain:** One row per playback session.

A playback session represents a continuous viewing attempt, according to the platform's sessionization rules.

| Column | Description |
|---|---|
| session_id | Session identifier |
| user_key | User dimension key |
| content_key | Content dimension key |
| device_key | Device dimension key |
| region_key | Region dimension key |
| session_start_date_key | Session start date |
| session_end_date_key | Session end date |
| start_timestamp | Session start time |
| end_timestamp | Session end time |
| watch_duration_seconds | Actual watched duration |
| content_duration_seconds | Total content duration |
| completion_percentage | Completion percentage |
| buffering_count | Number of buffering events |
| buffering_duration_seconds | Total buffering time |

Example:

| Session | User | Content | Watch duration |
|---|---|---|---:|
| S101 | U1 | Movie A | 5,400 |
| S102 | U2 | Movie B | 3,600 |
| S103 | U1 | Movie C | 1,800 |

Completion percentage:

\[
\text{Completion \%} =
\frac{\text{Watch Duration}}{\text{Content Duration}} \times 100
\]

This simple calculation may exceed 100% if users replay content or the session spans multiple playback attempts. Define the business rule clearly.

### 5.2 `fact_playback_event`

**Grain:** One row per playback event.

| Column | Description |
|---|---|
| event_id | Unique event ID |
| session_id | Playback session ID |
| user_key | User key |
| content_key | Content key |
| device_key | Device key |
| event_timestamp | Event time |
| event_type | Play, pause, seek, resume, stop |
| playback_position_seconds | Position in content |
| event_date_key | Event date |

Example events:

| Event ID | Session | Event type |
|---|---|---|
| E101 | S101 | PLAY |
| E102 | S101 | PAUSE |
| E103 | S101 | RESUME |
| E104 | S101 | STOP |

This table helps analyze playback behavior and troubleshoot user experience.

**Important:** A playback event is not automatically equivalent to watched time. Calculate actual viewing duration using reliable playback intervals or session metrics.

### 5.3 `fact_subscription_event`

**Grain:** One row per subscription lifecycle event.

| Column | Description |
|---|---|
| subscription_event_id | Event identifier |
| user_key | User key |
| plan_key | Subscription plan key |
| event_date_key | Event date |
| subscription_id | Subscription identifier |
| event_type | Subscribe, renew, upgrade, downgrade, cancel |
| effective_timestamp | When the event occurred |
| subscription_status | Resulting status |

This table preserves the history of subscription changes.

A separate accumulating snapshot can be useful when the business needs one row per subscription with milestone dates such as start, renewal, and cancellation.

### 5.4 `fact_payment`

**Grain:** One row per payment attempt or transaction, depending on the source system.

| Column | Description |
|---|---|
| payment_id | Payment identifier |
| user_key | User key |
| plan_key | Plan key |
| payment_date_key | Payment date |
| payment_method_key | Payment method key |
| amount | Transaction amount |
| currency | Transaction currency |
| payment_status | Success, failed, refunded |
| transaction_type | Charge, refund, adjustment |

Do not count failed attempts as successful revenue.

Refunds and adjustments should be represented consistently so financial reports do not overstate revenue.

### 5.5 `fact_daily_user_activity`

**Grain:** One row per user per calendar day.

| Column | Description |
|---|---|
| user_key | User key |
| date_key | Activity date |
| total_sessions | Number of sessions |
| total_watch_seconds | Total viewing duration |
| distinct_content_count | Number of unique content items watched |
| playback_event_count | Number of playback events |

This is a periodic snapshot or daily aggregate fact.

It supports efficient daily engagement reporting without repeatedly scanning all playback events.

### 5.6 `fact_content_rating`

**Grain:** One row per rating submission.

| Column | Description |
|---|---|
| rating_id | Rating identifier |
| user_key | User key |
| content_key | Content key |
| rating_date_key | Rating date |
| rating_value | User rating |
| review_count_flag | Usually 1 per rating row |

If users can change ratings, decide whether the table records every submission or only the latest rating. These represent different business processes and grains.

---

## 6. High-Level Warehouse Architecture

A typical architecture can use cloud storage, a processing engine, and a warehouse or lakehouse.

```text
Streaming Application
        |
        v
Application Events / Databases
        |
        v
Kafka / Event Hubs / CDC / Batch APIs
        |
        v
Bronze Layer
Raw events and source data
        |
        v
Silver Layer
Validated, cleaned, deduplicated data
        |
        v
Gold Layer
Facts, dimensions, aggregates
        |
        v
Data Warehouse / SQL Analytics
        |
        v
BI Dashboards and Reports
```

### Bronze layer

Store source data with minimal transformation.

Examples:

- Raw playback events.
- Subscription changes.
- Payment records.
- Content catalog updates.

Preserve source event IDs, ingestion timestamps, and source metadata.

### Silver layer

Clean and standardize data.

Typical operations:

- Validate required fields.
- Handle invalid timestamps.
- Deduplicate events.
- Standardize event names.
- Validate user and content references.
- Handle late-arriving events.
- Quarantine invalid records when appropriate.

### Gold layer

Create business-ready tables.

Examples:

- Playback session fact.
- Daily user activity fact.
- Subscription fact.
- Revenue aggregates.
- Content engagement metrics.

---

## 7. Streaming and Batch Ingestion

A streaming platform usually needs both streaming and batch ingestion.

| Data | Possible ingestion method |
|---|---|
| Playback events | Kafka or Event Hubs |
| User activity events | Streaming |
| Subscription updates | CDC or streaming |
| Payment transactions | CDC or micro-batch |
| Content catalog | Batch or CDC |
| Historical viewing data | Batch backfill |

### Streaming ingestion example

A simplified PySpark Structured Streaming example:

```python
from pyspark.sql import functions as F

events = (
    spark.readStream
    .format("kafka")
    .option("kafka.bootstrap.servers", "<broker>")
    .option("subscribe", "playback-events")
    .load()
)

parsed_events = (
    events
    .select(
        F.col("key").cast("string").alias("event_key"),
        F.col("value").cast("string").alias("event_json"),
        F.col("timestamp").alias("ingestion_timestamp")
    )
)

query = (
    parsed_events.writeStream
    .format("delta")
    .option("checkpointLocation", "<checkpoint-path>")
    .outputMode("append")
    .start("<bronze-table-path>")
)
```

This is a simplified ingestion example. Production code must parse the event payload, define schemas, handle malformed records, and configure secure Kafka connectivity.

The checkpoint tracks streaming progress and state. It does not automatically guarantee exactly-once results in every external sink.

---

## 8. Handling Event Time, Duplicates, and Late Data

Streaming platforms can generate millions of events. Events may arrive late, out of order, or more than once.

### Event time vs ingestion time

- Event time: When the user action actually happened.
- Ingestion time: When the platform received the event.

Use event time for business reporting whenever it is reliable.

### Deduplication

Use a stable event identifier where available.

```python
deduplicated_events = (
    events_df
    .dropDuplicates(["event_id"])
)
```

For streaming deduplication, consider watermarks to limit retained state:

```python
deduplicated_stream = (
    streaming_df
    .withWatermark("event_timestamp", "30 minutes")
    .dropDuplicates(["event_id", "event_timestamp"])
)
```

The watermark controls how long state is retained for supported stateful operations. Late-data handling depends on the operation and configured watermark behavior.

### Late-arriving data

For example:

- A user watches content at 10:00 AM.
- The event reaches the data platform at 10:20 AM.
- The event belongs to the 10:00 AM business period.

Store both event and ingestion timestamps so the pipeline can distinguish these situations.

---

## 9. Important Business KPIs

| KPI | Definition |
|---|---|
| Daily Active Users (DAU) | Distinct users with qualifying activity per day |
| Monthly Active Users (MAU) | Distinct users with qualifying activity per month |
| Total Watch Hours | Total qualifying viewing seconds / 3600 |
| Average Watch Time | Total viewing duration / selected user or session count |
| Content Completion Rate | Completed sessions / eligible sessions |
| Subscription Growth | New subscriptions minus cancellations, under a defined rule |
| Churn Rate | Churned subscribers / eligible starting subscriber base |
| Payment Success Rate | Successful payment attempts / eligible payment attempts |
| Average Rating | Average valid rating value |
| Buffering Rate | Defined buffering measure relative to sessions or viewing time |

**Metric definitions must be agreed with the business.**

For example, DAU could mean users who started a playback session, watched for at least a minimum duration, or performed any qualifying activity. The definition must be consistent across reports.

---

## 10. SQL Interview Queries

Assume these tables:

- `fact_playback_session`
- `dim_user`
- `dim_content`
- `dim_date`
- `fact_payment`

### Query 1: Total watch hours by content

```sql
SELECT
    c.content_title,
    SUM(f.watch_duration_seconds) / 3600.0 AS total_watch_hours
FROM fact_playback_session f
JOIN dim_content c
    ON f.content_key = c.content_key
GROUP BY c.content_title
ORDER BY total_watch_hours DESC;
```

### Query 2: Daily active users

```sql
SELECT
    d.full_date,
    COUNT(DISTINCT f.user_key) AS daily_active_users
FROM fact_playback_session f
JOIN dim_date d
    ON f.session_start_date_key = d.date_key
GROUP BY d.full_date
ORDER BY d.full_date;
```

### Query 3: Top 5 most-watched content items

```sql
SELECT
    c.content_title,
    SUM(f.watch_duration_seconds) AS total_watch_seconds
FROM fact_playback_session f
JOIN dim_content c
    ON f.content_key = c.content_key
GROUP BY c.content_title
ORDER BY total_watch_seconds DESC
LIMIT 5;
```

### Query 4: Payment success rate

```sql
SELECT
    COUNT(CASE
        WHEN payment_status = 'SUCCESS' THEN 1
    END) * 100.0
    / NULLIF(COUNT(*), 0) AS payment_success_rate
FROM fact_payment;
```

This query assumes each row is one eligible payment attempt. Exclude non-attempt records if the source table contains adjustments or other transaction types.

### Query 5: Average rating by content

```sql
SELECT
    c.content_title,
    AVG(r.rating_value) AS average_rating,
    COUNT(*) AS rating_count
FROM fact_content_rating r
JOIN dim_content c
    ON r.content_key = c.content_key
GROUP BY c.content_title
HAVING COUNT(*) >= 10
ORDER BY average_rating DESC;
```

---

## 11. Data Quality Checks

A reliable warehouse should validate data at every stage.

| Check | Example |
|---|---|
| Uniqueness | Event ID should be unique |
| Completeness | User ID and event timestamp should not be missing |
| Validity | Rating must be within the allowed range |
| Referential integrity | Content key should map to a valid content record |
| Accuracy | Watch duration should be non-negative |
| Consistency | Payment amount and currency should follow defined rules |
| Freshness | Playback events should arrive within the expected delay |
| Reconciliation | Payment totals should match the trusted payment source |

Example SQL:

```sql
SELECT
    COUNT(*) AS invalid_event_count
FROM silver_playback_events
WHERE event_id IS NULL
   OR user_id IS NULL
   OR event_timestamp IS NULL
   OR event_type NOT IN (
       'PLAY',
       'PAUSE',
       'RESUME',
       'SEEK',
       'STOP'
   );
```

Invalid records should be handled according to a defined quarantine or rejection policy rather than silently discarded.

---

## 12. Performance Optimization

A streaming platform can generate a very large volume of playback events.

Important optimization techniques:

1. **Partitioning or clustering:** Organize data based on common filters such as event date, where appropriate.
2. **File compaction:** Reduce excessive small files.
3. **Column pruning:** Read only required columns.
4. **Predicate pushdown:** Apply supported filters as early as possible.
5. **Incremental processing:** Process new or changed data instead of rebuilding everything.
6. **Pre-aggregations:** Maintain daily user and content-level summaries.
7. **Join optimization:** Broadcast suitable small dimensions and inspect actual plans.
8. **Retention policies:** Define how long detailed event data must be retained.
9. **Data skipping:** Use file statistics and suitable table layout to reduce unnecessary reads.

Do not partition by high-cardinality fields such as `user_id` without a strong workload-based reason. This can create excessive directories and small files.

---

## 13. Important Design Challenges

### Challenge 1: Very high event volume

**Problem:** Playback events can grow rapidly.

**Solution:**
- Use scalable event ingestion.
- Store raw events in a suitable lakehouse format.
- Process in micro-batches or streaming.
- Maintain aggregates for common reports.
- Define retention and archival policies.

### Challenge 2: Duplicate events

**Problem:** Retries or source delivery behavior may produce duplicates.

**Solution:**
- Use stable event IDs.
- Apply deterministic deduplication.
- Make writes retry-safe.
- Define how long duplicate detection state must be retained.

### Challenge 3: Late-arriving events

**Problem:** Events can arrive after their actual event time.

**Solution:**
- Preserve event and ingestion timestamps.
- Apply event-time processing.
- Use watermarks for suitable stateful operations.
- Support controlled backfills or corrections.

### Challenge 4: Subscription history

**Problem:** Users may upgrade, downgrade, renew, or cancel plans.

**Solution:**
- Store lifecycle events.
- Use SCD Type 2 for historical plan or user attributes when needed.
- Preserve effective dates.
- Define the subscriber status at each reporting date.

### Challenge 5: Accurate revenue reporting

**Problem:** Payment attempts, successful charges, refunds, and renewals can be confused.

**Solution:**
- Define the grain of payment facts.
- Separate attempts from successful transactions if necessary.
- Represent refunds and adjustments explicitly.
- Reconcile against the financial source of truth.
- Distinguish gross collections from recognized revenue.

### Challenge 6: User privacy

**Problem:** Viewing history and user identifiers can be sensitive.

**Solution:**
- Apply least-privilege access.
- Mask or restrict sensitive fields.
- Follow retention and deletion requirements.
- Use aggregated data for reports where individual-level detail is unnecessary.

---

## 14. How to Explain This Design in an Interview

A clear explanation can follow this sequence:

1. Understand the business requirements and reporting questions.
2. Identify separate business processes.
3. Define the grain for each fact table.
4. Identify dimensions such as user, content, device, plan, date, and region.
5. Design fact tables for playback, events, subscriptions, payments, ratings, and daily activity.
6. Explain the ingestion architecture using streaming, CDC, and batch processing.
7. Describe Bronze, Silver, and Gold transformations.
8. Explain deduplication, late data, and retry-safe processing.
9. Define business KPIs and their formulas.
10. Discuss data quality, performance, privacy, and governance.

### Sample interview explanation

"I would first separate the platform's business processes, such as playback sessions, playback events, subscriptions, payments, and ratings.

I would define a separate grain for each fact table. For example, one row in the playback session fact represents one playback session, while one row in the playback event fact represents one user playback event.

I would create dimensions for users, content, devices, subscription plans, dates, and regions.

Playback events would be ingested through a streaming platform such as Kafka or Event Hubs. Subscription and payment data could come through CDC or micro-batch ingestion, depending on source capabilities.

I would use a Bronze layer to preserve incoming data, a Silver layer for validation and deduplication, and a Gold layer for facts, dimensions, and business aggregates.

Finally, I would optimize the warehouse for common reporting workloads, monitor data quality and freshness, and apply appropriate privacy and access controls."

---

## 15. Common Interview Questions

### Q1. Why do we need separate playback session and playback event fact tables?

Because they represent different grains. One session can generate many events. Combining them can duplicate session-level measures.

### Q2. What is the difference between DAU and MAU?

DAU counts distinct qualifying active users in a day. MAU counts distinct qualifying active users over a month.

### Q3. How would you calculate watch time?

Use reliable playback intervals or session-level watch duration. Do not simply count every event as viewing time.

### Q4. How would you handle duplicate playback events?

Use a stable event identifier, deterministic deduplication, and retry-safe writes.

### Q5. How would you handle late-arriving events?

Preserve event time and ingestion time, use event-time processing where appropriate, and support corrections or backfills.

### Q6. Why use a daily user activity fact?

It supports common daily and monthly engagement reports efficiently without repeatedly scanning detailed event data.

### Q7. What is the difference between a fact table and a dimension table?

A fact table stores measurable business events or snapshots. A dimension table stores descriptive attributes used to analyze those facts.

### Q8. How would you calculate subscription churn?

Define the starting subscriber population and the churn event first. Then divide eligible churned subscribers by the eligible starting subscriber base for the period.

### Q9. Would you store every playback event forever?

Not necessarily. Retention depends on business, regulatory, analytical, and cost requirements. Older data may be archived or retained in aggregated form.

### Q10. How would you prevent incorrect revenue calculations?

Separate payment attempts, successful charges, refunds, and adjustments. Define the revenue metric and reconcile it with the financial source of truth.

### Q11. What is the purpose of a watermark?

It helps stateful streaming operations manage late data and limit retained state based on event-time progress.

### Q12. Why should fact tables not be joined blindly?

Different fact tables may have different grains. Joining them directly can multiply rows and inflate measures.

---

## 16. Common Design Mistakes

- Creating one giant fact table for every business process.
- Designing tables before defining grain.
- Treating playback events as equivalent to watch duration.
- Counting failed payments as revenue.
- Ignoring refunds and payment adjustments.
- Calculating DAU without defining qualifying activity.
- Ignoring late-arriving and duplicate events.
- Joining fact tables with incompatible grains.
- Partitioning by high-cardinality user identifiers without justification.
- Keeping detailed event data without a retention strategy.
- Ignoring user privacy and access control.
- Assuming a streaming checkpoint guarantees exactly-once behavior for every sink.

---

## 17. Quick Revision

| Topic | Key point |
|---|---|
| Business processes | Playback, subscriptions, payments, ratings |
| Playback session grain | One row per session |
| Playback event grain | One row per event |
| Subscription grain | One row per lifecycle event |
| Payment grain | One row per attempt or transaction |
| Daily activity grain | One row per user per day |
| Dimensions | User, content, device, plan, date, region |
| Bronze | Raw incoming data |
| Silver | Cleaned and validated data |
| Gold | Business-ready facts and aggregates |
| DAU | Distinct qualifying users per day |
| MAU | Distinct qualifying users per month |
| Late data | Preserve event time and ingestion time |
| Deduplication | Use stable event identifiers |
| Performance | Incremental processing, compaction, suitable layout |
| Governance | Privacy, retention, access control |

---

## 18. Self-Assessment

Try answering these without looking at the notes:

1. What is the grain of a playback session fact?
2. Why should playback events be stored separately?
3. How would you calculate DAU?
4. Why can playback event count not directly represent watch time?
5. How would you handle duplicate and late events?
6. What is the difference between event time and ingestion time?
7. How would you model subscription upgrades and cancellations?
8. How would you avoid double counting when joining facts?
9. How would you optimize a very large playback event table?
10. How would you ensure revenue reports are accurate?

**Interview readiness:** You should be able to explain the business processes, grains, fact and dimension design, streaming ingestion, data quality, and KPI definitions without referring to the notes.