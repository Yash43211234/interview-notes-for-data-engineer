

# Clickstream Analytics — System Design Case Study

## 1. Problem Statement

Design a scalable data platform that collects, processes, stores, and analyzes user activity on a website or mobile application.

Clickstream data represents the sequence of user interactions, such as page views, clicks, searches, product views, and purchases.

### Example

A user visits an e-commerce website:

```text
Homepage
   |
   v
Search for Shoes
   |
   v
View Product
   |
   v
Add to Cart
   |
   v
Purchase
```

Each interaction generates an event that can be used to understand user behavior.

### Business Goals

The platform should help the business answer questions such as:

- How many users visit the website each day?
- Which products receive the most views?
- How many users add products to their carts?
- What percentage of users complete a purchase?
- At which step do users abandon the purchase process?
- Which marketing campaigns generate the most conversions?
- How does user behavior change over time?

**Primary objective:** Build a reliable, scalable analytics platform that supports both near-real-time monitoring and historical analysis.

---

## 2. Functional Requirements

Functional requirements describe what the platform must do.

1. Collect user interaction events from websites and mobile applications.
2. Assign identifiers to events and capture event timestamps.
3. Ingest events continuously.
4. Validate incoming events and handle malformed records.
5. Remove or identify duplicate events.
6. Store raw events for future reprocessing.
7. Transform raw data into analytics-ready datasets.
8. Calculate metrics such as page views, unique users, and conversions.
9. Support funnel analysis and session-level analytics.
10. Provide dashboards and analytical queries.
11. Support historical backfills and corrections.

## 3. Non-Functional Requirements

| Requirement | Design consideration |
|---|---|
| Scalability | Handle growing event volume |
| Availability | Continue operating through expected failures |
| Reliability | Avoid losing or corrupting data |
| Data freshness | Make recent events available within the agreed latency target |
| Performance | Support analytical queries efficiently |
| Data quality | Validate events and detect duplicates |
| Security | Protect personal and sensitive information |
| Recoverability | Replay and reprocess retained data |
| Cost efficiency | Balance compute, storage, and latency |

The actual throughput, latency, retention, and availability targets must be agreed upon with the business before sizing the system.

---

## 4. High-Level Architecture

<text size="sm" color="secondary">Architecture overview</text>

```text
Website / Mobile Application
             |
             v
       Event Collection
             |
             v
     Streaming Ingestion
       Kafka / Event Hubs
             |
             v
       Raw Event Storage
        ADLS Gen2 / Bronze
             |
             v
    Stream / Batch Processing
       Spark / Databricks
             |
             v
       Cleaned Events
          Silver
             |
             v
      Aggregations / Models
           Gold
             |
        +----+-----+
        |          |
        v          v
   BI Dashboards  SQL Analytics
```

Supporting services include schema validation, access control, monitoring, orchestration, and data-quality checks.

### Component Responsibilities

| Component | Responsibility |
|---|---|
| Event collector | Captures browser and application activity |
| Message broker | Buffers and distributes events |
| Raw storage | Preserves incoming events |
| Processing engine | Cleans, enriches, deduplicates, and aggregates data |
| Data lakehouse | Stores structured analytical tables |
| SQL warehouse | Serves analytical queries |
| BI tool | Visualizes metrics |
| Monitoring | Tracks failures, latency, lag, and data quality |

The same event stream can feed multiple independent consumers, such as fraud detection, operational alerts, and analytics.

---

## 5. Event Data Model

Every event should have a clearly defined schema.

### Example Event

```json
{
  "event_id": "evt-10001",
  "event_name": "product_view",
  "event_time": "2026-10-31T10:30:00Z",
  "user_id": "user-501",
  "anonymous_id": "anon-9001",
  "session_id": "session-2001",
  "page_url": "/products/running-shoes",
  "product_id": "prod-101",
  "source": "web",
  "campaign_id": "campaign-25",
  "schema_version": 1
}
```

### Important Fields

| Field | Purpose |
|---|---|
| `event_id` | Identifies an event for deduplication |
| `event_name` | Describes the interaction |
| `event_time` | Time when the interaction occurred |
| `user_id` | Identifies a known user when available |
| `anonymous_id` | Identifies an anonymous browser or device |
| `session_id` | Groups events into a session |
| `page_url` | Identifies the page |
| `product_id` | Identifies the product involved |
| `campaign_id` | Supports campaign analysis |
| `schema_version` | Helps manage schema changes |

Not every event requires every field. For example, a page-view event may not have a `product_id`.

### Design Considerations

- Generate a stable event ID.
- Use UTC timestamps or another explicitly documented time standard.
- Distinguish event time from ingestion time.
- Define which fields are required for each event type.
- Version the schema.
- Avoid collecting unnecessary personal information.
- Define rules for anonymous and authenticated user identity.

---

## 6. Event Collection and Ingestion

The collection layer receives events from browsers, mobile applications, and backend services.

### Common Approaches

**Browser or mobile SDK:** Captures page views, clicks, searches, and user interactions.

**Backend events:** Records trusted server-side actions such as order completion and payment confirmation.

**Message broker:** Buffers incoming events and distributes them to downstream consumers.

### Why Use a Message Broker?

A broker such as Apache Kafka or Azure Event Hubs separates event producers from processing systems.

Benefits include:

- Buffering traffic spikes.
- Decoupling producers from consumers.
- Supporting multiple consumer groups.
- Allowing replay while events remain retained.
- Supporting partition-based parallelism.

A broker does not eliminate every loss scenario. Producer acknowledgment settings, replication, retention, consumer behavior, and downstream writes all affect end-to-end reliability.

### Partitioning Strategy

A broker topic may be partitioned by `user_id`, `session_id`, or another suitable key.

The key should reflect the required ordering and parallelism.

- A `user_id` key may help keep a user's events in the same partition.
- A `session_id` key may be appropriate for session-oriented processing.
- A poorly distributed key can create hot partitions.
- Ordering is guaranteed within a partition, not globally across all partitions.

---

## 7. Raw Storage and the Medallion Architecture

Raw storage preserves incoming events so that data can be audited and reprocessed.

A lakehouse design can use three logical layers.

### Bronze — Raw Events

Stores the incoming event data with minimal changes.

Useful metadata includes:

- Event payload.
- Ingestion timestamp.
- Source topic or file.
- Source partition and offset, where available.
- Processing batch or run ID.

### Silver — Cleaned Events

Contains validated, standardized, and deduplicated events.

Typical transformations include:

- Parsing timestamps.
- Validating required fields.
- Handling malformed records.
- Normalizing event names.
- Deduplicating by event ID.
- Applying privacy rules.
- Enriching events with reference data.

### Gold — Business Metrics

Contains business-ready datasets, such as:

- Daily active users.
- Product views by day.
- Session-level summaries.
- Conversion rates.
- Marketing campaign performance.
- Funnel conversion metrics.

The medallion layers are a logical organization, not a requirement that every platform must implement exactly three physical copies of the data.

---

## 8. Data Processing and Deduplication

Clickstream events may be duplicated because of retries, client behavior, producer configuration, or replay.

### Example

| event_id | event_name | user_id |
|---|---|---|
| evt-101 | page_view | user-1 |
| evt-102 | product_view | user-1 |
| evt-102 | product_view | user-1 |

The second and third rows represent the same event ID.

A common approach is to deduplicate by `event_id`, assuming it is reliably generated and unique for the logical event.

### PySpark Example

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

window_spec = (
    Window
    .partitionBy("event_id")
    .orderBy(
        F.col("ingestion_time").desc(),
        F.col("source_sequence").desc()
    )
)

deduplicated_df = (
    events_df
    .withColumn(
        "row_num",
        F.row_number().over(window_spec)
    )
    .filter(F.col("row_num") == 1)
    .drop("row_num")
)
```

This example assumes `ingestion_time` and `source_sequence` are available and that the ordering reflects which copy should be retained. Add a deterministic tie-breaker if those fields can tie.

For large streaming workloads, use supported streaming deduplication with appropriate event-time watermarks when suitable. The retention of deduplication state must balance duplicate protection against memory and state-store costs.

**Important:** Deduplicating by `user_id` would be incorrect because one user can legitimately generate many events.

---

## 9. Sessionization

A session is a group of related user interactions within a defined activity period.

For example, a session might include:

```text
10:00 — Homepage view
10:02 — Product view
10:05 — Add to cart
10:08 — Checkout
```

A common sessionization rule starts a new session after a period of inactivity, such as 30 minutes. The correct timeout is a business decision, not a universal standard.

### Sessionization Considerations

- Use a stable user or anonymous identifier.
- Order events by event time.
- Handle events that arrive late.
- Decide how to treat authenticated users across devices.
- Define the inactivity threshold.
- Avoid assuming a browser session equals a person.
- Account for privacy and consent requirements.

### Example with PySpark

For an already bounded dataset, a session boundary can be derived using a window function.

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

user_window = (
    Window
    .partitionBy("anonymous_id")
    .orderBy("event_time")
)

events_with_previous = (
    events_df
    .withColumn(
        "previous_event_time",
        F.lag("event_time").over(user_window)
    )
    .withColumn(
        "new_session",
        F.when(
            F.col("previous_event_time").isNull() |
            (
                F.col("event_time").cast("long") -
                F.col("previous_event_time").cast("long")
                > 1800
            ),
            1
        ).otherwise(0)
    )
)
```

This identifies session boundaries when the gap between consecutive events exceeds 30 minutes.

A complete implementation must then assign a session sequence or session ID and account for ties, missing identifiers, and late events. This batch-oriented example does not by itself implement robust streaming sessionization.

---

## 10. Key Analytics Metrics

### 10.1 Page Views

The total number of page-view events in a given period.

```sql
SELECT
    CAST(event_time AS DATE) AS event_date,
    COUNT(*) AS page_views
FROM silver.clickstream_events
WHERE event_name = 'page_view'
GROUP BY CAST(event_time AS DATE);
```

### 10.2 Unique Users

The number of distinct users or identifiers observed during a period.

```sql
SELECT
    CAST(event_time AS DATE) AS event_date,
    COUNT(DISTINCT anonymous_id) AS unique_visitors
FROM silver.clickstream_events
GROUP BY CAST(event_time AS DATE);
```

This measures distinct anonymous identifiers, not necessarily distinct human beings. Identity resolution and cross-device tracking rules affect the result.

### 10.3 Product Views

```sql
SELECT
    product_id,
    COUNT(*) AS total_views
FROM silver.clickstream_events
WHERE event_name = 'product_view'
GROUP BY product_id
ORDER BY total_views DESC;
```

### 10.4 Conversion Rate

A common purchase conversion rate is:

\[
\text{Conversion Rate} =
\frac{\text{Converted Units}}{\text{Eligible Units}}
\times 100
\]

The numerator and denominator must be defined consistently.

For example, if the business defines conversion as the percentage of sessions that contain a purchase:

\[
\text{Session Conversion Rate} =
\frac{\text{Sessions with a Purchase}}{\text{Eligible Sessions}}
\times 100
\]

This differs from a user-based conversion rate or an order-based rate.

### 10.5 Funnel Analysis

A purchase funnel may contain:

```text
Product View
     |
     v
Add to Cart
     |
     v
Checkout
     |
     v
Purchase
```

A proper funnel definition must specify whether the steps need to occur in order, within the same session, within a time limit, and for the same user or product.

Counting all events independently does not calculate a valid sequential funnel.

---

## 11. Data Modeling for Analytics

A warehouse-friendly model separates event facts from descriptive dimensions.

### Example Fact Table: `fact_clickstream_event`

| Column | Description |
|---|---|
| `event_key` | Warehouse surrogate key, if needed |
| `event_id` | Source event identifier |
| `user_key` | User dimension key, if available |
| `session_key` | Session dimension key |
| `page_key` | Page dimension key |
| `event_date_key` | Date dimension key |
| `event_timestamp` | Event timestamp |
| `event_name` | Event type |
| `event_count` | Often 1 per event row |

### Possible Dimensions

- `dim_user`
- `dim_session`
- `dim_page`
- `dim_product`
- `dim_campaign`
- `dim_date`
- `dim_device`

The appropriate grain should be defined first.

For an event fact table, one row might represent one accepted clickstream event. Session summaries should be stored at a different grain, such as one row per session.

Do not join event-level and session-level facts directly without considering their different grains, as doing so can multiply records and distort metrics.

---

## 12. Real-Time vs Historical Analytics

The platform can serve both near-real-time and historical workloads.

| Near-real-time analytics | Historical analytics |
|---|---|
| Recent traffic | Monthly traffic trends |
| Current product popularity | Product performance over a year |
| Campaign monitoring | Campaign comparisons |
| Checkout drop-off alerts | Historical funnel analysis |
| Operational dashboards | Cohort and retention analysis |

### Near-Real-Time Path

```text
Events
  |
  v
Broker
  |
  v
Streaming Processor
  |
  v
Recent Aggregates
  |
  v
Dashboard / Alert
```

### Historical Path

```text
Raw Event Storage
       |
       v
Batch Processing
       |
       v
Curated Tables
       |
       v
SQL Analytics / BI
```

Both paths can use the same retained raw data, although their freshness, computation, and serving requirements may differ.

---

## 13. Late Events and Event-Time Processing

Clickstream events may arrive late because of:

- Poor network connectivity.
- Mobile devices reconnecting.
- Client-side buffering.
- Broker delays.
- Processing backlogs.

A reliable system should distinguish event time from ingestion time.

### Recommended Handling

1. Preserve the original event timestamp.
2. Record the ingestion timestamp separately.
3. Validate timestamps and identify unreasonable values.
4. Use event-time windows for time-based metrics where appropriate.
5. Configure watermarks for stateful streaming operations.
6. Reconcile or backfill data when late events exceed the normal handling window.

A watermark does not guarantee that all late events will be included. Some events may arrive after the system has discarded the relevant state.

For historical accuracy, retain raw events and support correction or recomputation of affected aggregates.

---

## 14. Data Quality and Privacy

Clickstream data can be noisy and may contain sensitive information.

### Data Quality Checks

- Required event IDs are present.
- Event names belong to an approved set.
- Timestamps are parseable.
- Page URLs follow expected formats.
- Product IDs exist when required.
- Duplicate event rates are monitored.
- Schema changes are handled safely.
- Event volumes are compared with expected ranges.

### Privacy and Security

- Avoid collecting passwords, payment details, or unnecessary personal data.
- Do not place sensitive information in URLs or event payloads.
- Apply consent and retention policies.
- Restrict access to identifiable clickstream records.
- Mask or pseudonymize identifiers where appropriate.
- Apply deletion and data-subject request procedures where required.
- Secure data in transit and at rest.
- Audit access to sensitive datasets.

Pseudonymized identifiers can still be personal data in some circumstances. They should not automatically be treated as anonymous.

---

## 15. Reliability, Recovery, and Reprocessing

The platform must handle failures without losing the ability to produce correct analytics.

### Potential Failures

- Collector service failure.
- Broker or network interruption.
- Streaming job failure.
- Invalid events.
- Schema incompatibility.
- Storage or warehouse outage.
- Incorrect transformation logic.

### Recovery Strategy

1. Preserve raw events for the required retention period.
2. Use durable broker retention and appropriate replication.
3. Configure streaming checkpoints.
4. Make downstream writes safe to retry.
5. Maintain run IDs and ingestion metadata.
6. Route invalid events to quarantine.
7. Reprocess affected dates or partitions after code corrections.
8. Reconcile output metrics after recovery.

A checkpoint alone does not guarantee end-to-end exactly-once results. Correct recovery also depends on sink behavior and the handling of partial commits.

---

## 16. Scaling the Platform

Clickstream volume may increase rapidly during promotions or major events.

### Scaling Strategies

**Partitioned ingestion:** Distribute events across partitions using a suitable key.

**Horizontal processing:** Add processing capacity when the workload can use it.

**Storage partitioning:** Organize analytical data according to useful query filters, often including event date.

**Columnar storage:** Use formats such as Parquet for analytical workloads.

**Incremental processing:** Process newly arrived or affected data instead of repeatedly scanning all history.

**Efficient aggregation:** Precompute common metrics when repeated analytical queries are expensive.

**Autoscaling:** Adjust resources where supported, while accounting for startup delays, cost, and workload limits.

High-cardinality partitioning, excessive small files, skewed keys, and expensive distinct counts can reduce performance.

---

## 17. Performance and Monitoring

Monitor the platform from ingestion through serving.

### Operational Metrics

- Events received per second.
- Events processed per second.
- Broker backlog or consumer lag.
- Processing duration.
- End-to-end data freshness.
- Failed event count.
- Streaming query failures.
- Checkpoint and state-store health.
- Destination write latency.
- Storage growth and compute cost.

### Data Metrics

- Total event count.
- Duplicate event rate.
- Invalid event rate.
- Missing required fields.
- Unique visitor count.
- Purchase conversion rate.
- Unexpected traffic changes.
- Difference between source and target counts.

A pipeline can report `SUCCESS` while still producing incorrect data. Operational monitoring must be combined with data-quality and business-metric validation.

---

## 18. Capacity Estimation

Capacity estimation helps determine approximate ingestion, storage, and processing requirements.

Suppose the business expects:

- 10 million events per day.
- Average event size of 1 KB before compression.

### Average Events per Second

\[
\text{Average EPS} =
\frac{10{,}000{,}000}{86{,}400}
\approx 116
\]

The average rate is approximately 116 events per second.

Actual peak traffic may be much higher and must be measured or estimated separately.

### Raw Storage per Day

\[
10{,}000{,}000 \times 1\text{ KB}
\approx 10\text{ GB/day}
\]

This is a rough decimal-unit estimate before compression and storage overhead.

### Raw Storage per Year

\[
10\text{ GB/day} \times 365
\approx 3.65\text{ TB/year}
\]

This estimate excludes replicas, derived datasets, indexes, transaction logs, and additional metadata. The actual storage footprint depends on encoding, compression, retention, and the number of materialized datasets.

### Capacity Planning Must Consider

- Peak events per second.
- Average and maximum event size.
- Retention period.
- Broker replication.
- Compression.
- Processing complexity.
- Number of downstream consumers.
- Query workload.
- Backfill and replay capacity.
- Growth and operational headroom.

---

## 19. Design Trade-Offs

| Decision | Trade-off |
|---|---|
| Streaming vs batch | Freshness vs operational complexity |
| Raw retention | Reprocessing ability vs storage cost |
| User-based partitioning | Local ordering vs possible key skew |
| Exact distinct counts | Accuracy vs compute and memory |
| Precomputed aggregates | Faster queries vs additional storage and update complexity |
| Long retention | Historical analysis vs higher cost |
| Strict validation | Higher quality vs risk of rejecting valid new events |
| Near-real-time dashboards | Freshness vs infrastructure and serving cost |

No design is universally best. The choice depends on business requirements, data volume, correctness expectations, and cost.

---

## 20. End-to-End Design Walkthrough

Suppose an interviewer asks:

**“Design a clickstream analytics platform for an e-commerce company.”**

A structured response would be:

### Step 1: Clarify Requirements

Ask about:

- Expected events per second and peak traffic.
- Required data freshness.
- Required metrics.
- Event retention.
- Historical query volume.
- Privacy constraints.
- Availability and recovery requirements.

### Step 2: Define the Event Schema

Create a versioned event schema with event ID, event name, event timestamp, session or user identifier where available, and relevant business attributes.

### Step 3: Design Ingestion

Collect client-side and server-side events and send them to Kafka or Event Hubs.

Use suitable partitioning, authentication, retention, and producer reliability settings.

### Step 4: Preserve Raw Events

Write incoming events to durable raw storage so they can be audited and reprocessed.

### Step 5: Process and Validate

Use Spark Structured Streaming or another suitable engine to parse events, validate schemas, handle invalid records, deduplicate, and enrich data.

### Step 6: Build Analytical Datasets

Create curated event tables and appropriate aggregates for page views, sessions, funnels, and conversions.

Define the grain of each table carefully.

### Step 7: Serve Queries

Use a SQL warehouse or another analytical serving system for dashboards and historical queries.

### Step 8: Add Reliability

Configure checkpoints, safe retries, idempotent writes, monitoring, alerting, and recovery procedures.

### Step 9: Add Governance

Apply access controls, privacy rules, retention policies, and auditing.

### Step 10: Validate and Optimize

Monitor freshness, consumer lag, duplicate rates, data quality, query performance, and cost. Scale based on measured bottlenecks.

---

## 21. Common Interview Mistakes

1. Starting with technology names before clarifying requirements.
2. Ignoring event schema and identity design.
3. Treating every event as a unique human user action.
4. Assuming Kafka provides global event ordering.
5. Confusing ingestion time with event time.
6. Ignoring duplicates and late events.
7. Assuming checkpointing alone guarantees exactly-once output.
8. Defining conversion rate without defining its denominator.
9. Joining fact tables at different grains without aggregation.
10. Forgetting privacy and consent requirements.
11. Estimating capacity using average traffic only.
12. Ignoring backfills and historical corrections.
13. Monitoring infrastructure but not business data quality.
14. Overengineering the platform before establishing latency and scale requirements.

---

## 22. Interview Questions and Answers

### Q1. What is clickstream data?

Clickstream data records user interactions with websites or applications, such as page views, clicks, searches, product views, and purchases.

### Q2. Why use Kafka or Event Hubs?

They buffer and distribute events, decouple producers from consumers, and support scalable event ingestion and replay within retention limits.

### Q3. Why preserve raw events?

Raw data supports auditing, debugging, historical reprocessing, and rebuilding curated datasets after transformation logic changes.

### Q4. How do you deduplicate clickstream events?

Use a reliable event ID or another well-defined uniqueness key. Apply deduplication within the required scope and make downstream writes safe to retry.

### Q5. What is sessionization?

Sessionization groups related events into sessions using defined rules, such as an inactivity timeout, user identifier, and event-time ordering.

### Q6. How do you calculate conversion rate?

Define the business metric first. For session conversion, divide eligible sessions containing a qualifying conversion by all eligible sessions, then multiply by 100.

### Q7. How do you handle late events?

Preserve event and ingestion timestamps, use event-time processing and appropriate watermark policies, and support reconciliation or reprocessing for events that arrive too late.

### Q8. Why use Bronze, Silver, and Gold layers?

Bronze preserves raw data, Silver contains validated and standardized data, and Gold provides business-ready analytical datasets.

### Q9. How would you support both real-time and historical analytics?

Use a streaming ingestion path for fresh data and durable raw storage as the foundation for historical processing. Publish curated tables and aggregates for both workloads.

### Q10. How do you scale the platform?

Scale ingestion and processing based on measured traffic, partition effectively, optimize transformations and storage, and address bottlenecks such as skew or slow sinks.

### Q11. How do you recover from a failed streaming job?

Inspect the failure and committed output, recover using compatible checkpoints where applicable, retry safe operations, and validate the results. Use raw data for controlled replay or backfill if needed.

### Q12. How do you handle privacy?

Collect only necessary information, enforce consent and retention requirements, protect identifiers, restrict access, and provide mechanisms for applicable deletion requests.

### Q13. What is the difference between user-level and session-level conversion?

User-level conversion measures users who convert relative to eligible users. Session-level conversion measures sessions that convert relative to eligible sessions. The values can differ substantially.

### Q14. Why might dashboard counts differ from source event counts?

Possible reasons include deduplication, invalid-event filtering, late events, time-zone boundaries, identity rules, aggregation grain, or incomplete processing.

### Q15. What are the most important metrics to monitor?

Throughput, end-to-end freshness, consumer lag, failure rates, invalid and duplicate event rates, output completeness, and business metrics such as conversion rate.

---

## 23. Quick Revision

| Concept | Key point |
|---|---|
| Clickstream | Sequence of user interaction events |
| Event ID | Supports event-level deduplication |
| Event time | When the interaction occurred |
| Ingestion time | When the platform received it |
| Kafka / Event Hubs | Event ingestion and distribution |
| Bronze | Raw event preservation |
| Silver | Validated and cleaned events |
| Gold | Business-ready metrics |
| Sessionization | Group events into user sessions |
| Funnel | Measure progress through defined steps |
| Watermark | Controls event-time progress for supported operations |
| Backfill | Process historical data |
| Idempotency | Makes retries safe |
| Data grain | Defines what one row represents |
| Consumer lag | Indicates processing delay behind the source |
| Data freshness | How current the usable data is |

### Five Key Takeaways

1. Define the business metrics and event schema before choosing technologies.
2. Preserve raw data to enable reprocessing and auditability.
3. Handle duplicates, late events, and partial failures explicitly.
4. Define the grain and denominator of every analytical metric.
5. Monitor data correctness and freshness in addition to infrastructure health.

---

## 24. Self-Assessment

Try answering these questions without looking at the notes.

1. Design a clickstream platform for 100 million events per day.
2. How would you handle duplicate events generated by a browser retry?
3. Why should event time and ingestion time be stored separately?
4. How would you calculate a session-level purchase conversion rate?
5. How would you process events that arrive after a window has been finalized?
6. Why is preserving raw data useful when transformation logic changes?
7. How would you identify whether high dashboard latency comes from ingestion, processing, or serving?
8. What is the difference between event-level and session-level fact tables?
9. How would you handle a correction to historical purchase events?
10. How would you balance real-time freshness, historical accuracy, and infrastructure cost?

