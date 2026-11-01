

# SLA, SLO, and Data Freshness in Data Engineering

## 1. Introduction

In data engineering, pipelines must deliver reliable and up-to-date data within an expected time.

For example, a business may require its daily sales report to be available by 7:00 AM every day.

To define and measure this expectation, organizations use concepts such as:

- **SLA:** Service Level Agreement.
- **SLO:** Service Level Objective.
- **SLI:** Service Level Indicator.
- **Data Freshness:** How recently data has been updated.

These concepts help data engineering teams measure reliability, track delivery performance, and meet business expectations.

---

## 2. What Is SLA?

SLA stands for **Service Level Agreement**.

It is a formal agreement between a service provider and a customer or business stakeholder that defines expected service levels.

An SLA may include:

- Service availability.
- Data delivery deadlines.
- Response time.
- Incident resolution expectations.
- Reliability targets.
- Consequences of not meeting agreed commitments.

### Example

Suppose a company runs a daily sales pipeline.

The business agreement states:

> The daily sales dataset must be available by 7:00 AM on at least 99% of business days.

This is an SLA-style commitment.

### Data Engineering Examples

| Service | SLA Example |
|---|---|
| Daily sales pipeline | Data available by 7:00 AM |
| Customer data pipeline | Data refreshed every 2 hours |
| Financial reporting | Daily dataset available before 8:00 AM |
| Critical streaming pipeline | Data delivered within an agreed latency limit |

An SLA should be clearly defined and measurable.

---

## 3. What Is SLO?

SLO stands for **Service Level Objective**.

It is a measurable performance target that a team aims to achieve.

An SLO usually supports an SLA or an internal reliability goal.

### Example

Suppose the business expects sales data by 7:00 AM.

The data engineering team defines this SLO:

> At least 99% of scheduled sales pipeline runs should complete before 6:45 AM over a rolling 30-day period.

This gives the engineering team a measurable target with a 15-minute buffer before the business deadline.

### Common Data Engineering SLOs

| SLO | Example Target |
|---|---|
| Pipeline success | 99% successful scheduled runs |
| Data freshness | Data updated within 30 minutes |
| Pipeline completion | 99% completed before 6:45 AM |
| Streaming latency | 95% of events processed within 2 minutes |
| Data quality | 99.9% of records pass defined validation rules |

These are illustrative targets. Actual objectives depend on business requirements and system capabilities.

---

## 4. What Is SLI?

SLI stands for **Service Level Indicator**.

It is the actual measurement used to evaluate service performance.

An SLI provides the data needed to determine whether an SLO has been achieved.

### Example

SLO:

At least 99% of scheduled pipeline runs should complete before 6:45 AM.

SLI:

Percentage of scheduled pipeline runs that actually completed before 6:45 AM.

Suppose:

- Total scheduled runs = 100.
- Runs completed before 6:45 AM = 98.

\[
SLI = \frac{98}{100}\times100 = 98\%
\]

The measured SLI is 98%, while the SLO target is 99%.

Therefore, the SLO was not achieved.

### Common Data Engineering SLIs

| SLI | Measurement |
|---|---|
| Availability | Percentage of time a service is available |
| Success rate | Percentage of successful pipeline runs |
| Freshness | Age of the latest available data |
| Latency | Time taken to process or deliver data |
| Completeness | Percentage of expected records received |
| Quality | Percentage of records passing defined checks |

An SLI must have a precise measurement method and a clearly defined measurement window.

---

## 5. SLA vs SLO vs SLI

| Feature | SLA | SLO | SLI |
|---|---|---|---|
| Full form | Service Level Agreement | Service Level Objective | Service Level Indicator |
| Meaning | Agreement about service expectations | Target for service performance | Actual performance measurement |
| Purpose | Defines commitment | Defines reliability goal | Measures actual performance |
| Nature | Often contractual or business-facing | Operational target | Measured value |
| Example | Data available by 7:00 AM | 99% on-time delivery | Actual on-time delivery is 98% |

### Easy Way to Remember

- **SLA:** What have we promised?
- **SLO:** What target are we trying to achieve?
- **SLI:** What did we actually measure?

---

## 6. Understanding Data Freshness

Data freshness describes how recently the available data was updated.

It is especially important when businesses depend on current information for reporting, analytics, and decision-making.

### Example

A sales dashboard should display data updated by 7:00 AM.

However, the latest available data was updated at 6:00 AM.

At 7:00 AM, the data is one hour old.

The dashboard may therefore be considered stale, depending on the defined freshness requirement.

### Why Is Freshness Important?

- Prevents business teams from using outdated information.
- Helps identify delayed ingestion.
- Supports timely reporting.
- Detects upstream pipeline failures.
- Helps maintain streaming and near-real-time data delivery.

### Common Freshness Requirements

| Data Type | Example Requirement |
|---|---|
| Daily sales | Updated by 7:00 AM |
| Customer analytics | Updated every 2 hours |
| Inventory | Updated every 15 minutes |
| Payment events | Available within 1 minute |
| Historical archive | Updated once per day |

Freshness requirements should reflect how frequently the business needs new data.

---

## 7. How to Measure Data Freshness

A common approach is to compare the current time with the latest relevant data timestamp.

\[
\text{Data Age} =
\text{Current Time} - \text{Latest Data Timestamp}
\]

Example:

- Current time = 10:00 AM.
- Latest data timestamp = 9:30 AM.

\[
\text{Data Age} = 30 \text{ minutes}
\]

The dataset is 30 minutes old.

### Important: Choose the Correct Timestamp

Different timestamps answer different questions:

| Timestamp | What It Represents |
|---|---|
| Event timestamp | When the business event occurred |
| Ingestion timestamp | When the data entered the platform |
| Processing timestamp | When a processing operation handled the data |
| Publication timestamp | When the final dataset became available |

For example, an event may occur at 9:00 AM but arrive in the data platform at 9:20 AM.

If the business wants to know when the platform received the data, use ingestion time.

If the business wants to know how old the underlying business events are, use event time.

Do not confuse event freshness with pipeline execution freshness.

---

## 8. SQL Example: Checking Data Freshness

Suppose an orders table contains:

| order_id | order_timestamp | ingestion_timestamp |
|---|---|---|
| 101 | 09:00:00 | 09:05:00 |
| 102 | 09:15:00 | 09:20:00 |
| 103 | 09:25:00 | 09:30:00 |

To find the latest ingestion timestamp:

```sql
SELECT
    MAX(ingestion_timestamp) AS latest_ingestion_time
FROM orders;
```

To calculate data age:

```sql
SELECT
    CURRENT_TIMESTAMP AS current_time,
    MAX(ingestion_timestamp) AS latest_ingestion_time,
    CURRENT_TIMESTAMP - MAX(ingestion_timestamp)
        AS data_age
FROM orders;
```

**Note:** Timestamp subtraction and interval syntax vary between SQL engines. Use the appropriate date-difference function for your database.

### Freshness Status Example

```sql
SELECT
    MAX(ingestion_timestamp) AS latest_ingestion_time,
    CASE
        WHEN MAX(ingestion_timestamp) IS NULL
            THEN 'NO DATA'
        WHEN MAX(ingestion_timestamp) <
             CURRENT_TIMESTAMP - INTERVAL '30' MINUTE
            THEN 'STALE'
        ELSE 'FRESH'
    END AS freshness_status
FROM orders;
```

The interval syntax shown is supported by some SQL engines, but should be adapted for the target database.

---

## 9. PySpark Example: Checking Data Freshness

Suppose a DataFrame contains an `ingestion_timestamp` column.

```python
from pyspark.sql import functions as F

latest_data = df.agg(
    F.max("ingestion_timestamp").alias("latest_timestamp")
)

result = latest_data.withColumn(
    "data_age_minutes",
    (
        F.unix_timestamp(F.current_timestamp())
        - F.unix_timestamp("latest_timestamp")
    ) / 60
)

result.show(truncate=False)
```

### Interpret the Result

Suppose the result is:

```text
latest_timestamp       data_age_minutes
2026-10-25 09:30:00    30.0
```

The latest ingested data is approximately 30 minutes old.

For production monitoring, also handle:
- Empty datasets.
- NULL timestamps.
- Timezone differences.
- Late-arriving events.
- Incorrect source timestamps.

A freshness check should not silently treat missing data as fresh.

---

## 10. Freshness vs Latency

Freshness and latency are related, but they measure different things.

### Freshness

Measures how old the latest available data is.

Example:

Latest data timestamp = 9:00 AM.

Current time = 9:30 AM.

Data age = 30 minutes.

### Latency

Measures the time taken for data to move through a system or complete a processing operation.

Example:

- Event generated at 9:00 AM.
- Event available in the target table at 9:05 AM.

End-to-end delivery latency = 5 minutes.

| Feature | Freshness | Latency |
|---|---|---|
| Meaning | Age of available data | Time taken for processing or delivery |
| Focus | How current is the data? | How long did it take? |
| Example | Latest data is 30 minutes old | Event took 5 minutes to arrive |

A pipeline can have low processing latency but still contain stale data if no new source data arrives.

---

## 11. Error Budget

An error budget is the amount of unreliability allowed under a defined SLO.

It helps teams balance reliability and development speed.

### Example

Suppose an SLO requires 99% successful scheduled pipeline runs over 100 runs.

Allowed failure percentage:

\[
100\%-99\%=1\%
\]

Total allowed failures:

\[
100\times1\%=1
\]

Therefore, the error budget is one failed run.

If two runs fail, the team has exceeded the allowed failure budget.

### Error Budget Formula

For a run-based SLO:

\[
\text{Error Budget} =
\text{Total Eligible Runs}
\times
(1-\text{SLO Target})
\]

For a time-based availability SLO, the budget is calculated using the allowed unavailable time within the measurement window.

### Why Is an Error Budget Useful?

- Measures reliability against a target.
- Helps prioritize production stability.
- Highlights repeated failures.
- Supports decisions about risky deployments.
- Encourages teams to improve reliability.

**Important:** The error budget calculation depends on the SLI definition. A run-based SLO and a time-based availability SLO use different eligible events and denominators.

---

## 12. Freshness Monitoring Architecture

A typical freshness monitoring process looks like this:

```text
Source System
     |
     v
Data Ingestion
     |
     v
Target Table
     |
     v
Read Latest Timestamp
     |
     v
Calculate Data Age
     |
     v
Compare with Freshness Target
     |
   /   \
  v     v
Fresh  Stale
  |      |
  v      v
Pass   Generate Alert
         |
         v
    Investigation
```

### Example

Expected freshness: 30 minutes.

Actual freshness: 50 minutes.

The dataset is 20 minutes beyond the freshness threshold.

The monitoring system should generate an alert according to the configured severity and notification policy.

---

## 13. Implementing SLA and SLO Monitoring

A practical implementation requires the following steps.

### Step 1: Understand Business Requirements

Identify:
- When should data be available?
- How frequently should it be refreshed?
- What is the acceptable delay?
- Which reports or applications depend on it?

### Step 2: Define SLIs

Choose measurable indicators.

Examples:
- Percentage of pipeline runs completed on time.
- Age of the latest available record.
- Percentage of records passing quality checks.
- Percentage of successful pipeline runs.

### Step 3: Define SLOs

Set realistic targets based on business requirements and historical performance.

Example:

```text
Pipeline: Daily Sales

On-time completion target: 99%
Freshness target: 30 minutes
Success target: 99.5%
Measurement window: Rolling 30 days
```

### Step 4: Collect Measurements

Collect:
- Pipeline start and end times.
- Execution status.
- Latest data timestamps.
- Record counts.
- Quality-check results.

### Step 5: Calculate SLI

Calculate the actual performance against each objective.

### Step 6: Configure Alerts

Generate alerts when performance approaches or violates defined thresholds.

### Step 7: Review Performance

Review SLI trends and error-budget consumption regularly.

---

## 14. Monitoring SLA and Freshness in Azure

Azure services can be combined to implement SLA and freshness monitoring.

### Azure Data Factory

Use ADF Monitor to track:
- Pipeline execution status.
- Activity execution duration.
- Trigger runs.
- Pipeline failures.

### Azure Monitor

Use Azure Monitor for:
- Metrics.
- Log-based monitoring.
- Alert rules.
- Notification integrations.

### Log Analytics

Use KQL to analyze pipeline execution information.

Illustrative example:

```kusto
ADFPipelineRun
| where TimeGenerated > ago(30d)
| summarize
    TotalRuns = count(),
    SuccessfulRuns = countif(Status == "Succeeded"),
    FailedRuns = countif(Status == "Failed")
| extend SuccessRate =
    100.0 * SuccessfulRuns / TotalRuns
```

This calculates the success rate for the pipeline runs represented in the table.

For an on-time SLO, the query must also compare each run's completion time against the agreed deadline.

### Databricks

A Databricks pipeline can record:
- Job run start and end time.
- Latest source event timestamp.
- Latest ingestion timestamp.
- Latest target publication timestamp.
- Data quality results.

These values can be stored in a monitoring table and evaluated through dashboards or alerting integrations.

### Example Monitoring Table

| pipeline_name | run_id | status | records_processed | completed_at |
|---|---|---|---:|---|
| orders_pipeline | run_101 | SUCCESS | 100000 | 06:30 |
| orders_pipeline | run_102 | SUCCESS | 105000 | 06:35 |
| orders_pipeline | run_103 | FAILED | 0 | NULL |

A monitoring table provides historical information for trend analysis and SLO calculation.

---

## 15. Common Mistakes

1. Defining an SLA without a measurable condition.
2. Confusing SLA, SLO, and SLI.
3. Measuring freshness using the wrong timestamp.
4. Ignoring late-arriving data.
5. Treating a successful pipeline run as proof of fresh data.
6. Calculating success rate without defining the measurement window.
7. Ignoring empty datasets.
8. Mixing UTC and local timestamps.
9. Setting unrealistic freshness targets.
10. Failing to alert when an SLO is at risk.
11. Confusing processing latency with data age.
12. Ignoring planned maintenance or approved exclusions in SLO calculations.

---

## 16. Practical Interview Scenario

**Question: Your business expects daily sales data by 7:00 AM, but the dataset is sometimes delayed. How would you define and monitor an SLA, SLO, and freshness requirement?**

### Answer

I would first understand the business delivery requirement.

**Step 1: Define the SLA**

The business requires the daily sales dataset to be available by 7:00 AM.

**Step 2: Define the SLO**

The engineering team targets completion before 6:45 AM on at least 99% of scheduled business days over a rolling 30-day period.

**Step 3: Define the SLI**

Measure the percentage of eligible scheduled runs completed before 6:45 AM.

**Step 4: Define Freshness**

For example, require the latest relevant source event timestamp to be no more than 30 minutes behind the expected event-time boundary.

The exact freshness calculation must account for the source's expected delivery pattern and late-arriving events.

**Step 5: Collect Monitoring Data**

Record pipeline status, start time, completion time, ingestion timestamp, event timestamp, and record count.

**Step 6: Configure Alerts**

Generate alerts when:
- The pipeline fails.
- The expected completion time is at risk.
- The data freshness threshold is exceeded.
- The record count changes unexpectedly.

**Step 7: Review SLO Performance**

Calculate the on-time delivery percentage and track error-budget consumption.

This approach makes delivery expectations measurable and helps the team identify reliability problems before they affect business users.

---

## 17. Common Interview Questions and Answers

### Q1. What is SLA?

An SLA is an agreement that defines expected service levels between a provider and a customer or business stakeholder.

### Q2. What is SLO?

An SLO is a measurable target for service performance.

### Q3. What is SLI?

An SLI is the actual measurement used to evaluate service performance.

### Q4. Explain SLA vs SLO vs SLI.

SLA defines the commitment, SLO defines the target, and SLI measures actual performance.

### Q5. What is data freshness?

Data freshness measures how recently the available data was updated.

### Q6. How do you calculate data age?

Subtract the latest relevant data timestamp from the current time.

### Q7. What is the difference between freshness and latency?

Freshness measures the age of available data, while latency measures the time taken for an operation or data delivery.

### Q8. What is an error budget?

An error budget is the allowed amount of unreliability under a defined SLO.

### Q9. How do you monitor data freshness?

Track the latest relevant timestamp, calculate its age, compare it with the freshness target, and trigger alerts when the target is violated.

### Q10. Can a pipeline succeed but violate its freshness SLO?

Yes. A pipeline may complete successfully but process old or delayed source data.

### Q11. Why is the measurement window important?

It defines which observations are included when calculating the SLI and evaluating the SLO.

### Q12. What is a rolling 30-day SLO?

It evaluates performance using the most recent 30 days of eligible measurements, rather than a fixed calendar month.

### Q13. How do you prevent stale data from reaching reporting systems?

Use freshness checks, data validation, controlled publishing, and downstream dependencies that wait for successful validation.

### Q14. Why should freshness use the correct timestamp?

Different timestamps measure different things. Event time measures when the business event happened, while ingestion time measures when the platform received it.

### Q15. How do you monitor on-time pipeline delivery in Azure?

Use ADF run information, diagnostic logs, Log Analytics, and Azure Monitor alerts to measure completion against the agreed deadline.

---

## 18. Quick Revision

| Concept | Meaning |
|---|---|
| SLA | Service Level Agreement |
| SLO | Service Level Objective |
| SLI | Service Level Indicator |
| Data Freshness | How recently data was updated |
| Data Age | Current time minus latest relevant data timestamp |
| Latency | Time taken for processing or delivery |
| Error Budget | Allowed unreliability under an SLO |
| Measurement Window | Period used to evaluate an SLI |
| Rolling Window | Continuously moving measurement period |
| On-Time Delivery | Data available before the agreed deadline |

### Remember These Five Points

1. SLA defines the business commitment.
2. SLO defines the measurable reliability target.
3. SLI measures actual performance.
4. Freshness measures how current the data is.
5. Error budgets help teams manage reliability against an SLO.

---

## 19. Self-Assessment

Try answering these questions without looking at the notes.

1. What is SLA?
2. What is SLO?
3. What is SLI?
4. Explain the difference between SLA, SLO, and SLI.
5. What is data freshness?
6. How do you calculate data age?
7. What is the difference between event time and ingestion time?
8. What is the difference between freshness and latency?
9. What is an error budget?
10. How do you calculate a run-based error budget?
11. Why is the measurement window important?
12. What is a rolling 30-day SLO?
13. Can a successful pipeline violate a freshness SLO?
14. How would you monitor daily sales data freshness?
15. How would you define an on-time delivery SLO?
16. How would you monitor SLA performance using Azure services?
17. How would you handle late-arriving data?
18. Design an SLA, SLO, and freshness strategy for a production data pipeline.

**Interview readiness:** Be able to clearly explain SLA, SLO, SLI, freshness, latency, and error budgets, and design a measurable delivery-monitoring strategy for a production data pipeline.

