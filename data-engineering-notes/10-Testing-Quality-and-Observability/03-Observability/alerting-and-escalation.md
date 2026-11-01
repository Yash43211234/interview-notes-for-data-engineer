

# Alerting and Escalation in Data Engineering

## 1. Introduction

Alerting and escalation are important parts of data pipeline monitoring and observability.

They help teams identify failures, notify the right people, and ensure that critical problems are resolved within an acceptable time.

- **Alerting:** Notifies a person or team when a defined condition or threshold is reached.
- **Escalation:** Moves an unresolved issue to a higher level of support or authority when it is not handled within the expected time.

### Example

Suppose a daily data pipeline processes customer orders.

```text
Source Database
       |
       v
Data Ingestion
       |
       v
PySpark Transformation
       |
       v
Delta Lake
       |
       v
Data Warehouse
```

The pipeline normally completes by 7:00 AM.

One day, the pipeline fails because the source database is unavailable.

A monitoring system detects the failure and sends an alert to the data engineering team.

If the issue is not resolved within the defined response time, it is escalated to the senior engineer or support manager.

This helps prevent downstream reports from using incomplete or outdated data.

---

## 2. What Is Alerting?

Alerting is the process of automatically notifying responsible people or systems when an important event occurs.

Alerts are generally triggered by:
- Pipeline failures.
- Long-running jobs.
- Data freshness violations.
- High error rates.
- Missing files.
- Unexpected record counts.
- Data quality failures.
- Infrastructure problems.
- Resource utilization thresholds.

### Example

Suppose a daily pipeline should complete within 30 minutes.

Expected duration: 30 minutes.

Actual duration: 65 minutes.

An alert can be triggered when execution exceeds the defined threshold.

```text
Pipeline Started
       |
       v
Execution Monitoring
       |
       v
Duration > 30 Minutes?
       |
    Yes
       |
       v
  Send Alert
```

### Common Alert Channels

| Channel | Usage |
|---|---|
| Email | Detailed notifications |
| Microsoft Teams | Team notifications |
| Slack | Engineering alerts |
| SMS | Urgent notifications |
| PagerDuty | Incident notification and on-call response |
| Opsgenie | Alert management and on-call workflows |
| ServiceNow | Incident and ticket management |
| Webhooks | Integrating monitoring systems with external applications |

Tool availability and integration options depend on the organization's environment and subscriptions.

---

## 3. What Is Escalation?

Escalation is the process of transferring an unresolved issue to another person or support level.

It ensures that critical incidents receive attention when the initial response is insufficient.

### Example

A production pipeline fails at 6:00 AM.

| Time | Action |
|---|---|
| 6:00 AM | Pipeline failure detected |
| 6:02 AM | Alert sent to L1 support |
| 6:15 AM | No resolution; escalated to L2 |
| 6:45 AM | Issue requires senior engineering support |
| 7:00 AM | L3 team investigates the root cause |

This is an example of time-based escalation.

### Common Escalation Levels

| Level | Responsibility |
|---|---|
| L1 | Initial monitoring and basic troubleshooting |
| L2 | Detailed technical investigation |
| L3 | Advanced engineering or specialist support |
| Management | Business impact, resources, and priority decisions |

The exact structure varies between organizations.

---

## 4. Alerting vs Escalation

| Feature | Alerting | Escalation |
|---|---|---|
| Purpose | Notify about an issue | Ensure unresolved issues receive attention |
| Trigger | Defined event or condition | Lack of response, unresolved issue, or increasing severity |
| Timing | Usually immediate after detection | Often after a defined time or condition |
| Recipient | Initial owner or support team | Higher support level or responsible authority |
| Example | Pipeline failure notification | Escalate an unresolved failure to L2 |
| Goal | Awareness | Resolution and accountability |

**Remember:**

- Alerting tells someone that a problem exists.
- Escalation ensures that the problem receives appropriate attention.

---

## 5. Types of Alerts in Data Engineering

### 5.1 Pipeline Failure Alert

Triggered when a pipeline or job fails.

Examples:
- ADF pipeline failure.
- Databricks job failure.
- Spark application failure.
- Airflow DAG failure.

### 5.2 Performance Alert

Triggered when a job takes longer than expected or resource usage becomes abnormal.

Examples:
- Spark job duration exceeds the threshold.
- Executor memory usage becomes unusually high.
- Database query execution time increases.

### 5.3 Data Quality Alert

Triggered when data violates defined quality rules.

Examples:
- Duplicate primary keys.
- Unexpected NULL values.
- Invalid transaction amounts.
- Source-to-target reconciliation failure.

### 5.4 Data Freshness Alert

Triggered when data is not updated within the expected time.

Example:

A Gold table must be refreshed by 7:00 AM.

If the latest successful update is older than the agreed freshness limit, the monitoring system sends an alert.

### 5.5 Volume Anomaly Alert

Triggered when record volume changes unexpectedly.

Example:

Normal daily order volume: 10,000 records.

Today's order volume: 1,000 records.

The system detects a 90% decrease and sends an alert.

### 5.6 Infrastructure Alert

Triggered when infrastructure metrics cross configured thresholds.

Examples:
- CPU usage is consistently high.
- Memory pressure increases.
- Storage capacity approaches its limit.
- Network connectivity fails.

---

## 6. Alert Severity Levels

Alerts should be classified according to their impact and urgency.

| Severity | Meaning | Example |
|---|---|---|
| P1 / Critical | Major production impact | Critical business pipeline completely unavailable |
| P2 / High | Significant impact | Important reporting pipeline delayed |
| P3 / Medium | Limited impact | Non-critical data quality failure |
| P4 / Low | Minor issue | Warning about increased execution duration |

### Example

Suppose an organization runs three pipelines:

| Pipeline | Business Importance | Failure Severity |
|---|---|---|
| Payment Processing | Critical | P1 |
| Daily Sales Reporting | High | P2 |
| Historical Archive | Low | P4 |

Severity should depend on business impact, not just the technical error.

A pipeline failure that affects a critical financial report may be more urgent than a longer-running non-critical archive job.

---

## 7. Alerting Architecture

A typical alerting architecture looks like this:

```text
       Data Pipeline
             |
             v
       Monitoring System
             |
             v
       Alert Evaluation
             |
             v
       Severity Classification
             |
             v
       Notification System
             |
             v
       Responsible Team
             |
             v
       Incident Management
             |
       /             \
      v               v
   Resolved        Escalated
      |               |
      v               v
   Closure       Higher Support
```

### Main Components

**1. Data Pipeline**

The pipeline generates execution events, logs, and metrics.

**2. Monitoring System**

Collects and evaluates pipeline and infrastructure information.

**3. Alert Rules**

Define when notifications should be triggered.

**4. Notification System**

Sends alerts through email, Teams, Slack, or an incident-management platform.

**5. Incident Management**

Tracks ownership, investigation, resolution, and communication.

**6. Escalation Workflow**

Moves unresolved incidents to the appropriate support level.

---

## 8. Important Alerting Concepts

### 8.1 Alert Threshold

A threshold is a predefined limit that triggers an alert.

Example:

```python
pipeline_duration = 65
threshold = 30

if pipeline_duration > threshold:
    print("Pipeline duration alert")
```

Thresholds should be based on historical behavior, business requirements, and acceptable operating limits.

### 8.2 Alert Fatigue

Alert fatigue occurs when teams receive too many notifications, especially irrelevant or repeated alerts.

It can cause engineers to ignore important notifications.

Ways to reduce alert fatigue:

- Use meaningful thresholds.
- Remove duplicate alerts.
- Group related failures.
- Set severity levels.
- Use alert suppression for known maintenance windows.
- Alert only the responsible team.
- Monitor trends instead of every minor fluctuation.

### 8.3 Alert Deduplication

Deduplication prevents repeated notifications for the same underlying issue.

Example:

If a pipeline retries five times and fails each time, the system may create one incident with updated failure details rather than five separate incidents.

### 8.4 Alert Suppression

Alert suppression temporarily prevents notifications under defined conditions.

Example:

During planned maintenance, expected service-unavailable alerts may be suppressed.

Suppression should not hide unexpected critical failures.

### 8.5 Alert Acknowledgment

Acknowledgment means that a responsible person has received and accepted ownership of the alert.

It does not mean the issue is resolved.

### 8.6 Alert Resolution

Resolution means the underlying issue has been fixed or an approved workaround has restored the required service.

---

## 9. Escalation Policies

An escalation policy defines who receives an incident and what happens if it remains unresolved.

A policy may include:

- Initial responder.
- Secondary responder.
- Senior engineering contact.
- Incident manager.
- Escalation time limits.
- Severity-based routing.
- On-call schedule.

### Example Escalation Policy

| Condition | Action |
|---|---|
| Critical alert detected | Notify on-call engineer immediately |
| Not acknowledged within 10 minutes | Notify secondary engineer |
| Still unresolved after 30 minutes | Escalate to senior engineer |
| Business deadline at risk | Notify incident manager |
| Service restored | Update and close incident |

These timings are illustrative. Actual service-level objectives and response agreements should determine them.

### Types of Escalation

**Functional Escalation**

Transfers the issue to someone with deeper technical expertise.

Example: L1 support to a Spark specialist.

**Hierarchical Escalation**

Moves the issue to a higher authority or management level.

Example: Engineering team to engineering manager.

**Time-Based Escalation**

Escalates when the issue is not acknowledged or resolved within a defined period.

**Severity-Based Escalation**

Routes incidents according to their business impact.

---

## 10. Alerting in Azure Data Engineering

Azure provides several services that can support alerting and incident notification.

### Azure Monitor

Azure Monitor collects and analyzes monitoring data from Azure resources and applications.

It supports:
- Metrics.
- Logs.
- Alerts.
- Dashboards.
- Diagnostic information.

### Log Analytics Workspace

A Log Analytics workspace stores and allows querying of log data using Kusto Query Language (KQL).

Example:

```kusto
AzureDiagnostics
| where TimeGenerated > ago(1h)
| summarize FailureCount = count()
    by Resource, OperationName
| where FailureCount > 0
```

The exact table and column names depend on the diagnostic settings and resource integration.

### Action Groups

Action Groups define what happens when an Azure Monitor alert fires.

Possible actions include:
- Email notifications.
- SMS notifications.
- Voice calls, where supported.
- Webhook integrations.
- Logic Apps.
- Azure Functions.
- ITSM integrations.

### Alert Rules

Alert rules define the condition that triggers an alert.

Common categories include:
- Metric alerts.
- Log search alerts.
- Activity log alerts.
- Service health alerts.

### Example Azure Monitoring Flow

```text
Azure Data Factory
        |
        v
 Diagnostic Logs
        |
        v
 Log Analytics Workspace
        |
        v
 Azure Monitor Alert Rule
        |
        v
 Action Group
        |
        v
 Email / Teams / Webhook
```

---

## 11. Alerting in Azure Data Factory

Azure Data Factory provides pipeline monitoring and failure information.

A pipeline run can have statuses such as:

- Queued.
- In progress.
- Succeeded.
- Failed.
- Cancelled.

ADF monitoring can be used to inspect:
- Pipeline execution status.
- Activity execution status.
- Error messages.
- Execution duration.
- Input and output details.
- Run identifiers.

### Example: Failure Notification

```text
ADF Pipeline
      |
      v
Activity Execution
      |
      v
Failure Detected
      |
      v
Failure Dependency
      |
      v
Notification Activity
      |
      v
Email / Webhook / Function
```

A pipeline can use failure dependencies to invoke a notification activity after a failure.

For broader production monitoring, diagnostic logs can also be sent to Log Analytics and evaluated using Azure Monitor alert rules.

**Important:** ADF pipeline failure dependencies and Azure Monitor alerts are different mechanisms. The former handles workflow behavior; the latter supports centralized monitoring and notification.

---

## 12. Alerting in Databricks

Databricks supports monitoring through job run information, system tables, and integrations with external monitoring tools.

Common monitoring information includes:

- Job run status.
- Task failure.
- Execution duration.
- Cluster or compute problems.
- Spark application logs.
- Data quality failures.

### Job Notifications

Databricks Jobs and Workflows can be configured to send notifications for events such as:

- Job start.
- Job success.
- Job failure.
- Duration warning.

Available notification options depend on workspace features and configuration.

### Example Workflow

```text
Databricks Job
      |
      v
Task Execution
      |
      v
Task Failure
      |
      v
Job Notification
      |
      v
Email / Webhook / Integration
      |
      v
Engineer Investigates
```

For centralized monitoring, teams can integrate Databricks logs and metrics with their observability platform.

---

## 13. Alerting in Apache Airflow

Airflow provides callbacks and notification mechanisms that can be used when tasks or DAGs fail.

Common callbacks include:

- `on_failure_callback`
- `on_success_callback`
- `on_retry_callback`
- `on_execute_callback`

### Example: Failure Callback

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def notify_failure(context):
    task_instance = context["task_instance"]

    print(
        f"Task failed: {task_instance.task_id}"
    )

with DAG(
    dag_id="orders_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
    on_failure_callback=notify_failure
) as dag:

    task = PythonOperator(
        task_id="process_orders",
        python_callable=lambda: 1 / 0
    )
```

This example prints a failure message. In production, the callback should send a notification through an approved email, webhook, or incident-management integration.

**Important:** Callback behavior depends on the callback scope and Airflow version. A DAG-level failure callback and a task-level failure callback are not interchangeable in every scenario. Airflow callbacks also do not replace monitoring of scheduler, worker, and infrastructure failures.

---

## 14. Alerting Best Practices

1. Define clear alert conditions.
2. Classify alerts by severity.
3. Notify the correct owner.
4. Avoid unnecessary notifications.
5. Use deduplication and grouping.
6. Define acknowledgment and escalation timelines.
7. Include useful diagnostic details.
8. Track alert history.
9. Monitor both pipeline health and data quality.
10. Test notification integrations.
11. Document escalation contacts and procedures.
12. Review thresholds periodically.
13. Suppress only known and approved events.
14. Define who can close an incident.
15. Record root causes and corrective actions.

### What Should a Good Alert Contain?

A useful alert should include:

| Field | Purpose |
|---|---|
| Alert title | Quickly identifies the issue |
| Pipeline or job name | Identifies the affected workload |
| Severity | Indicates business impact |
| Environment | DEV, TEST, or PROD |
| Run ID | Helps locate the execution |
| Failure time | Shows when it happened |
| Error message | Provides diagnostic information |
| Business impact | Explains what is affected |
| Owner | Identifies the responsible team |
| Runbook link | Provides troubleshooting instructions |

Avoid sending only a generic message such as "Pipeline failed."

---

## 15. Practical Interview Scenario

**Question: Your daily production pipeline fails at 6:00 AM, and the business report is expected by 7:00 AM. How would you handle alerting and escalation?**

### Answer

I would follow a structured incident-handling process.

**Step 1: Detect the Failure**

Use the monitoring system to identify the failed pipeline or task.

**Step 2: Send an Alert**

Notify the responsible data engineering team with:
- Pipeline name.
- Run ID.
- Failure time.
- Error details.
- Business impact.

**Step 3: Acknowledge the Incident**

The on-call engineer acknowledges the alert and takes ownership.

**Step 4: Investigate**

Check execution logs, source availability, permissions, dependencies, and recent changes.

**Step 5: Escalate if Required**

If the issue is beyond the initial team's capability or remains unresolved within the defined response window, escalate it to the appropriate support level.

**Step 6: Recover**

Fix the issue and safely rerun the affected pipeline. Ensure that rerunning does not create duplicate or inconsistent data.

**Step 7: Validate the Output**

Check data quality, record counts, reconciliation results, and freshness before allowing downstream reporting.

**Step 8: Communicate and Close**

Confirm that the report is available, document the root cause, and close the incident.

---

## 16. Common Interview Questions and Answers

### Q1. What is alerting?

Alerting is the process of notifying responsible people or systems when a defined event or condition occurs.

### Q2. What is escalation?

Escalation transfers an unresolved issue to a higher support level or appropriate authority.

### Q3. What is the difference between alerting and escalation?

Alerting notifies someone about a problem. Escalation ensures unresolved problems receive additional attention.

### Q4. What is alert fatigue?

Alert fatigue occurs when teams receive too many notifications, making it difficult to identify important alerts.

### Q5. What is alert deduplication?

It prevents repeated notifications for the same underlying incident.

### Q6. What is alert suppression?

It temporarily prevents notifications under defined conditions, such as approved maintenance windows.

### Q7. What is an escalation policy?

It defines who should receive an incident and when it should be transferred to another support level.

### Q8. What are common alert severity levels?

Critical, high, medium, and low are commonly used severity levels.

### Q9. What is acknowledgment?

Acknowledgment confirms that a responsible person has received and accepted ownership of an alert.

### Q10. What is the purpose of Azure Monitor?

Azure Monitor collects and analyzes monitoring data and supports alerting for Azure resources and applications.

### Q11. What is an Action Group?

An Action Group defines the notification or automated actions to execute when an Azure Monitor alert fires.

### Q12. How do you monitor ADF pipeline failures?

Use ADF monitoring, diagnostic logs, Log Analytics, and Azure Monitor alert rules.

### Q13. How do you send failure notifications from Airflow?

Use failure callbacks or configured notification integrations.

### Q14. What information should a production alert contain?

Pipeline name, severity, environment, run ID, error details, failure time, business impact, and troubleshooting information.

### Q15. Why is escalation important in production data engineering?

It ensures that critical issues are handled within acceptable time limits and reduces the risk of prolonged business disruption.

---

## 17. Quick Revision

| Concept | Meaning |
|---|---|
| Alerting | Notifies about an issue |
| Escalation | Transfers unresolved issues to higher support |
| Alert Threshold | Condition that triggers an alert |
| Alert Fatigue | Too many notifications |
| Deduplication | Prevents repeated alerts |
| Suppression | Temporarily prevents selected alerts |
| Acknowledgment | Confirms ownership of an alert |
| Severity | Indicates issue impact |
| Escalation Policy | Defines escalation rules |
| Azure Monitor | Azure monitoring and alerting service |
| Log Analytics | Log storage and KQL-based analysis |
| Action Group | Defines alert-triggered actions |
| Airflow Callback | Executes code on task or DAG events |

### Remember These Five Points

1. Alerting notifies the responsible team when an issue occurs.
2. Escalation ensures unresolved issues reach the appropriate support level.
3. Alerts should include useful technical and business context.
4. Azure Monitor and Action Groups support centralized Azure alerting.
5. A good incident process includes detection, acknowledgment, investigation, recovery, validation, and closure.

---

## 18. Self-Assessment

Try answering these questions without looking at the notes.

1. What is alerting?
2. What is escalation?
3. Explain the difference between alerting and escalation.
4. What are common alert types in data engineering?
5. What are alert severity levels?
6. What is alert fatigue?
7. What is alert deduplication?
8. What is alert suppression?
9. What is an escalation policy?
10. What information should a good alert contain?
11. What is Azure Monitor?
12. What is a Log Analytics workspace?
13. What is an Action Group?
14. How do you monitor ADF pipeline failures?
15. How do you configure failure notifications in Airflow?
16. How would you handle a production pipeline failure?
17. How would you prevent duplicate alerts during retries?
18. Design an alerting and escalation strategy for a critical daily data pipeline.

**Interview readiness:** Be able to explain alerting, escalation, severity levels, alert fatigue, Azure Monitor, Action Groups, and how you would handle a production pipeline failure from detection through resolution.

