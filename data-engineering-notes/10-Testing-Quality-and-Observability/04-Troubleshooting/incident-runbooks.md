

# Incident Runbooks for Data Engineering

## 1. Introduction

An **incident runbook** is a documented set of instructions that helps engineers investigate, respond to, recover from, and prevent a specific operational incident.

In data engineering, incidents can affect:
- Data pipeline execution.
- Data availability and freshness.
- Data quality.
- Source and target connectivity.
- Storage and compute resources.
- Downstream reports and business applications.

A runbook provides a consistent response process so engineers do not need to figure out every troubleshooting step during an emergency.

### Example

Suppose a daily orders pipeline fails because the source database is unavailable.

Without a runbook, an engineer may spend time deciding what to check.

With a runbook, the engineer can follow documented steps:

1. Check the failed pipeline run.
2. Verify source database availability.
3. Check connectivity and credentials.
4. Review retry configuration.
5. Rerun after the source is restored.
6. Validate the loaded records.
7. Document the incident.

---

## 2. Runbook vs Playbook vs SOP

These terms are related but have different purposes.

| Document | Purpose | Example |
|---|---|---|
| Runbook | Step-by-step operational instructions | Recover a failed ADF pipeline |
| Playbook | Broader response strategy for a type of incident | Handle a major data platform outage |
| SOP | Standard Operating Procedure for recurring work | Daily pipeline health checks |

A runbook is usually more specific and execution-oriented.

A playbook may contain multiple runbooks and decision paths.

An SOP documents a standard process that may not necessarily be incident-related.

---

## 3. Why Are Incident Runbooks Important?

### 3.1 Faster Incident Resolution

Engineers can follow known troubleshooting steps instead of investigating from scratch.

### 3.2 Consistent Response

Different engineers can follow the same recovery procedure.

### 3.3 Reduced Human Error

Documented steps reduce the risk of missing important checks during an incident.

### 3.4 Knowledge Sharing

Operational knowledge is not limited to one engineer.

### 3.5 Better Recovery

Runbooks explain how to restore services and validate data after recovery.

### 3.6 Easier Onboarding

New team members can understand how to respond to common production issues.

---

## 4. Components of a Good Incident Runbook

A useful runbook should contain the following sections.

| Component | Description |
|---|---|
| Runbook ID | Unique document reference |
| Title | Name of the incident |
| Purpose | What problem the runbook addresses |
| Severity | Expected business impact |
| Symptoms | How the incident is detected |
| Prerequisites | Required access and tools |
| Investigation | Steps to identify the cause |
| Resolution | Steps to fix the problem |
| Recovery | Steps to restore normal operation |
| Validation | Checks to confirm successful recovery |
| Escalation | Who to contact if unresolved |
| Prevention | Actions to reduce recurrence |
| References | Dashboards, logs, and related documents |

### Example Runbook Header

```yaml
runbook_id: RB-001
title: Failed Orders Pipeline
category: Pipeline Operations
severity: SEV-2
owner: Data Engineering Team
last_reviewed: 2026-10-01
```

This is an illustrative metadata format. An organization may use a different template.

---

## 5. Incident Severity Levels

Severity indicates how seriously an incident affects business operations.

A common example:

| Severity | Description | Example |
|---|---|---|
| SEV-1 | Critical business or platform impact | Entire production data platform unavailable |
| SEV-2 | Major business impact | Critical daily reporting pipeline unavailable |
| SEV-3 | Limited impact | One non-critical dataset delayed |
| SEV-4 | Minor issue | Warning or isolated non-urgent data quality issue |

Severity definitions and response targets must be established by the organization.

### Important Factors

When assigning severity, consider:
- Business impact.
- Number of affected users or systems.
- Data criticality.
- Duration of the outage.
- Availability of a workaround.
- SLA or SLO impact.

A technically serious error does not always mean the highest business severity.

---

## 6. Standard Incident Response Lifecycle

A typical incident response process contains these stages:

```text
Incident Detection
        |
        v
Incident Acknowledgment
        |
        v
Severity Assessment
        |
        v
Investigation
        |
        v
Containment / Mitigation
        |
        v
Recovery
        |
        v
Validation
        |
        v
Communication and Closure
        |
        v
Post-Incident Review
```

### Step 1: Detection

An incident can be detected through:
- Automated alerts.
- Pipeline failure notifications.
- Data quality checks.
- Freshness monitoring.
- User reports.
- Dashboard anomalies.

### Step 2: Acknowledgment

A responsible engineer acknowledges the incident and begins investigation.

### Step 3: Severity Assessment

Evaluate the business impact and assign an appropriate severity.

### Step 4: Investigation

Inspect logs, metrics, pipeline execution details, and dependencies.

### Step 5: Containment or Mitigation

Prevent the issue from causing further damage.

Examples:
- Pause a repeatedly failing pipeline.
- Stop an incorrect downstream publication.
- Isolate invalid input data.
- Prevent duplicate writes.

### Step 6: Recovery

Fix the cause and restore the pipeline or data service.

### Step 7: Validation

Confirm that the system and data are correct.

### Step 8: Communication and Closure

Inform affected stakeholders and document the resolution.

### Step 9: Post-Incident Review

Identify preventive improvements and update the relevant runbook.

---

## 7. Runbook 1: Failed Data Pipeline

<badge color="danger">Example: SEV-2</badge>

### Symptoms

- Pipeline status is FAILED.
- Expected data is missing.
- Downstream tasks are not executing.
- Freshness SLO may be breached.

### Investigation Steps

1. Open the orchestration platform.
2. Identify the failed pipeline run.
3. Record the run ID and failure timestamp.
4. Identify the failed activity or task.
5. Inspect the complete error message.
6. Review upstream and downstream dependencies.
7. Check source availability and input data.
8. Check credentials, permissions, and compute resources.

### Resolution

Apply the fix according to the root cause.

| Root Cause | Resolution |
|---|---|
| Source unavailable | Restore source connectivity |
| Missing file | Verify file arrival and path |
| Invalid schema | Apply controlled schema handling |
| Permission failure | Correct identity permissions |
| Transformation error | Fix and test the code |
| Compute failure | Investigate resource constraints |

### Recovery

1. Confirm whether previous tasks wrote data.
2. Verify the last successful processing stage.
3. Determine whether the pipeline is safe to rerun.
4. Rerun the failed task or appropriate pipeline stages.
5. Validate target records and downstream outputs.

### Validation

- Pipeline completes successfully.
- Expected records are present.
- No unintended duplicates exist.
- Data quality checks pass.
- Downstream datasets are refreshed.

### Escalation

Contact the relevant platform, source-system, or application owner if the issue is outside the data engineering team's control.

---

## 8. Runbook 2: Data Freshness SLA/SLO Breach

<badge color="warning">Example: SEV-2 or SEV-3</badge>

### Symptoms

- Data is older than the expected freshness threshold.
- Reports show outdated information.
- Freshness monitoring generates an alert.
- A scheduled pipeline has not completed.

### Investigation Steps

1. Check the expected publication time.
2. Identify the last successful pipeline run.
3. Check the latest source event or ingestion timestamp.
4. Inspect pipeline execution duration.
5. Identify delayed upstream dependencies.
6. Review orchestration schedules and retries.
7. Check whether the source system delivered the expected data.

### Example

Suppose a sales dashboard must be refreshed by 8:00 AM.

At 8:30 AM, the dashboard still contains yesterday's data.

Investigate:

```text
Expected Refresh: 08:00 AM
Current Time:     08:30 AM
Last Successful: Previous Day
Status:           Freshness Breach
```

### Resolution

- Restore the failed upstream process.
- Resolve the delayed ingestion.
- Fix the transformation or orchestration issue.
- Process the missing data interval.

### Validation

- Latest expected data is available.
- Data age is within the defined threshold.
- Dashboard or downstream consumer is refreshed.
- Freshness monitoring returns to the expected state.

### Important

Do not mark a freshness incident resolved merely because the pipeline status is SUCCESS. Confirm that the expected data actually reached the target.

---

## 9. Runbook 3: Data Quality Failure

### Symptoms

- Unexpected NULL values.
- Duplicate business keys.
- Invalid data types.
- Referential integrity violations.
- Unexpected record-count changes.
- Source-to-target reconciliation failure.

### Investigation Steps

1. Identify the failed quality rule.
2. Find the affected table or partition.
3. Identify the affected batch or processing date.
4. Compare source and target records.
5. Inspect sample invalid records.
6. Check recent schema or source changes.
7. Determine whether the issue is isolated or widespread.

### Example: Duplicate Detection

```sql
SELECT
    order_id,
    COUNT(*) AS record_count
FROM silver_orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

### Resolution

Depending on the issue:

- Correct transformation logic.
- Deduplicate records using a deterministic rule.
- Quarantine invalid records.
- Request source correction.
- Reprocess affected data.

### Validation

- Failed quality rules pass.
- Reconciliation is successful.
- Invalid records are accounted for.
- Downstream consumers receive correct data.

### Important

Do not silently delete or modify business records just to make a quality check pass. Follow the approved data correction and retention process.

---

## 10. Runbook 4: ADLS Gen2 Access Failure

### Symptoms

- `403 Forbidden`.
- Permission denied.
- Databricks cannot read or write files.
- ADF Copy Activity fails to access storage.

### Investigation Steps

1. Identify the identity used by the workload.
2. Confirm whether it is a managed identity or service principal.
3. Verify authentication.
4. Check the required Azure RBAC role.
5. Check ADLS Gen2 ACLs where applicable.
6. Verify the storage account and container path.
7. Check network restrictions and private endpoint configuration.

### Resolution

- Assign the minimum required data-plane role.
- Correct directory and file ACLs.
- Grant execute permission on parent directories where needed.
- Correct the storage path.
- Fix private DNS or network connectivity if applicable.

### Validation

- The intended identity can access the required path.
- Read/write operations succeed.
- Unnecessary permissions have not been granted.

### Important

Authentication, authorization, and network connectivity are separate concerns.

A successful login does not guarantee permission to access the data.

A private endpoint also does not automatically grant storage access.

---

## 11. Runbook 5: Databricks Job Failure

### Symptoms

- Databricks job run is FAILED.
- Notebook task fails.
- Spark stage fails.
- Driver or executor is lost.
- Job execution exceeds its expected duration.

### Investigation Steps

1. Open the Databricks Jobs interface.
2. Identify the failed job run and task.
3. Inspect notebook output and error details.
4. Check driver and executor logs.
5. Inspect the Spark UI.
6. Review input size and partition distribution.
7. Check cluster or serverless compute availability.
8. Verify storage access and dependency versions.

### Common Problems

| Problem | Investigation |
|---|---|
| Out of memory | Driver/executor memory, skew, collection |
| Slow stage | Shuffle, skew, joins, spill |
| Permission denied | Identity and storage permissions |
| Missing dependency | Library and runtime compatibility |
| Invalid schema | Input schema and conversion logic |
| Job timeout | Task duration and resource bottlenecks |

### Resolution

Fix the actual issue before rerunning.

For memory or performance problems, investigate the execution plan and Spark UI instead of increasing compute resources blindly.

### Validation

- Job completes successfully.
- Expected output is committed.
- Data quality checks pass.
- No unintended duplicate writes occur.

---

## 12. Runbook 6: Kafka or Event Hubs Ingestion Delay

### Symptoms

- Consumer lag increases.
- Data arrives late in the raw layer.
- Event ingestion stops.
- Downstream streaming tables become stale.

### Investigation Steps

1. Check producer availability.
2. Verify topic or Event Hub availability.
3. Inspect partition-level incoming traffic.
4. Check consumer group health.
5. Review consumer lag and processing throughput.
6. Check authentication and network access.
7. Inspect Spark Structured Streaming or consumer logs.
8. Check checkpoint and state-store errors.

### Resolution

- Restore producer connectivity.
- Resolve consumer failures.
- Correct partition or consumer configuration if required.
- Scale processing capacity where justified.
- Recover from the appropriate checkpoint if supported and safe.

### Validation

- New events are being consumed.
- Lag is decreasing or within the expected threshold.
- No unintended data loss or duplicate processing occurred.
- Downstream data is current.

### Important

Do not delete a streaming checkpoint as a routine troubleshooting step. Doing so can change recovery behavior and may cause data to be reprocessed or skipped, depending on the source and application design.

---

## 13. Runbook 7: Unexpected Duplicate Records

### Symptoms

- Target table contains duplicate business keys.
- Record counts are higher than expected.
- Reports show inflated totals.
- A rerun appears to have inserted records again.

### Investigation Steps

1. Identify the affected table and business key.
2. Determine when duplicates first appeared.
3. Inspect recent pipeline runs.
4. Check retries and partial writes.
5. Review source deduplication logic.
6. Inspect append, overwrite, and MERGE behavior.
7. Check whether concurrent runs processed overlapping data.

### Resolution

- Identify the correct record-selection rule.
- Deduplicate the affected source or target data.
- Correct the write strategy.
- Introduce safe retry and idempotency controls.
- Prevent overlapping runs if required.

### Example: Find Duplicate Keys

```sql
SELECT
    order_id,
    COUNT(*) AS duplicate_count
FROM gold_orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

### Validation

- Duplicate business keys are resolved according to business rules.
- Aggregates and record counts are correct.
- Reprocessing does not introduce duplicates.
- Downstream reports are reconciled.

---

## 14. Communication During an Incident

Technical recovery is only one part of incident management. Stakeholders also need accurate updates.

A useful incident update should include:

- What happened.
- Which systems or datasets are affected.
- Business impact.
- Current investigation status.
- Temporary workaround, if available.
- Expected next update.
- Resolution status.

### Example Incident Update

**Subject: Orders Pipeline Failure – Production Impact**

The daily orders pipeline has failed during the transformation stage.

The Raw layer ingestion completed, but the Silver and Gold datasets have not been refreshed.

The team is investigating the transformation failure and validating the previously committed data. We will provide another update after identifying the root cause and confirming the recovery plan.

Do not provide an estimated resolution time unless there is a reasonable basis for it.

---

## 15. Post-Incident Review

After resolving an important incident, conduct a post-incident review.

The purpose is to understand what happened and improve the system, not simply assign blame.

### What to Document

| Field | Example |
|---|---|
| Incident | Orders pipeline failure |
| Impact | Daily sales dashboard delayed |
| Detection | Automated pipeline alert |
| Root cause | Unexpected source schema change |
| Resolution | Schema handling corrected |
| Recovery | Affected data interval reprocessed |
| Validation | Reconciliation and quality checks passed |
| Prevention | Schema validation added |

### Questions to Ask

1. What happened?
2. When did it start?
3. How was it detected?
4. Which systems and users were affected?
5. What was the root cause?
6. How was service restored?
7. What worked well?
8. What could be improved?
9. Which monitoring or tests should be added?
10. Does the runbook need an update?

### Corrective and Preventive Actions

- Add missing monitoring.
- Improve alert thresholds.
- Add automated data validation.
- Improve retry and recovery logic.
- Remove single points of failure.
- Improve documentation.
- Update ownership and escalation details.

---

## 16. Runbook Maintenance Best Practices

An incident runbook should remain accurate and usable.

1. Assign an owner to every runbook.
2. Include the last-reviewed date.
3. Use clear, numbered instructions.
4. Include links to relevant dashboards and logs.
5. Avoid storing passwords, tokens, or secrets.
6. Document safe recovery and rollback steps.
7. Include validation criteria.
8. Define escalation contacts or team roles.
9. Test important recovery instructions.
10. Update the runbook after significant incidents.
11. Review runbooks after platform or architecture changes.
12. Remove outdated commands and screenshots.

### What Makes a Runbook Effective?

A good runbook should allow an appropriately authorized engineer to understand:

- What is wrong?
- How serious is it?
- What should be checked?
- What actions are safe?
- When should the issue be escalated?
- How do we confirm recovery?

---

## 17. Practical End-to-End Scenario

**Scenario:** A daily orders pipeline fails at 7:30 AM. The business dashboard must be refreshed by 8:00 AM.

### Incident Details

```text
Pipeline: orders_daily_pipeline
Environment: Production
Failure Time: 07:30 AM
Failed Task: transform_orders
Impact: Gold sales table not refreshed
```

### Response

**1. Detection**

The monitoring system generates a pipeline failure alert.

**2. Acknowledgment**

The data engineering team acknowledges the incident and checks the failed run.

**3. Investigation**

The team inspects the Databricks task logs and discovers a schema conversion error.

**4. Root Cause**

The source system changed the order date format.

**5. Mitigation**

The team prevents incorrect data from reaching the Gold layer and updates the transformation logic to handle the approved input formats.

**6. Recovery**

The failed processing interval is rerun using the corrected transformation.

**7. Validation**

The team checks:
- Source and target counts.
- Duplicate order IDs.
- Invalid date records.
- Silver and Gold table freshness.
- Sales totals.

**8. Communication**

The business team is informed when the dashboard is available again.

**9. Prevention**

Schema validation and an alert for unexpected date formats are added.

This scenario demonstrates how a runbook connects detection, investigation, mitigation, recovery, validation, and prevention.

---

## 18. Common Interview Questions and Answers

### Q1. What is an incident runbook?

A documented set of instructions for investigating, resolving, and recovering from a specific operational incident.

### Q2. Why are runbooks important in data engineering?

They improve response consistency, reduce troubleshooting time, reduce human error, and preserve operational knowledge.

### Q3. What is the difference between a runbook and a playbook?

A runbook provides specific operational steps. A playbook provides a broader response strategy and may reference multiple runbooks.

### Q4. What is the difference between an incident and a problem?

An incident is an event that disrupts or reduces service quality. A problem is the underlying cause, or potential cause, of one or more incidents.

### Q5. What are severity levels?

Severity levels classify incidents according to their business and operational impact.

### Q6. What should a runbook contain?

Purpose, symptoms, severity, prerequisites, investigation steps, resolution, recovery, validation, escalation, and prevention.

### Q7. How do you handle a failed production pipeline?

Identify the failed task, inspect logs, assess impact, investigate the root cause, recover safely, validate output, and document the incident.

### Q8. What is containment or mitigation?

Actions that limit the impact of an incident while investigation or permanent resolution continues.

### Q9. Why is validation important after recovery?

A successful task status does not necessarily prove that the correct data was produced. Validation confirms the business output is correct.

### Q10. What is a post-incident review?

A review of an incident to understand its cause, impact, response, and preventive improvements.

### Q11. How do you prevent duplicate records during incident recovery?

Use idempotent writes, deterministic deduplication, safe MERGE logic, and proper batch-progress tracking.

### Q12. What should you do if a runbook does not resolve the issue?

Stop unsafe actions, collect logs and evidence, escalate to the responsible team, communicate the impact, and update the runbook after resolution.

### Q13. What is the difference between a runbook and an SOP?

A runbook focuses on operational response and recovery. An SOP documents a standard process, which may be routine rather than incident-driven.

### Q14. Why should runbooks be tested?

Untested instructions may be outdated, incomplete, or unsafe. Testing verifies that the documented recovery process works.

### Q15. What is a blameless post-incident review?

A review that focuses on system conditions, contributing factors, and improvements instead of blaming individuals.

---

## 19. Quick Revision

| Concept | Key Meaning |
|---|---|
| Incident | Event that disrupts or degrades service |
| Runbook | Step-by-step operational instructions |
| Playbook | Broader incident response strategy |
| SOP | Standard recurring procedure |
| Severity | Classification based on impact |
| Detection | Identifying an incident |
| Acknowledgment | Taking responsibility for response |
| Mitigation | Limiting incident impact |
| Recovery | Restoring normal service |
| Validation | Confirming correctness after recovery |
| RCA | Identifying the underlying cause |
| Post-Incident Review | Learning and preventing recurrence |

### Remember These Five Points

1. A runbook tells engineers what to do during a known operational issue.
2. Severity should reflect business impact.
3. Recovery must be safe and account for partial writes.
4. Successful execution does not automatically mean correct data.
5. Every significant incident should improve monitoring, testing, or documentation.

---

## 20. Self-Assessment

Try answering these questions without looking at the notes.

1. What is an incident runbook?
2. Why do data engineering teams maintain runbooks?
3. Explain runbook vs playbook vs SOP.
4. What information should a runbook contain?
5. How are incident severity levels assigned?
6. Explain the incident response lifecycle.
7. How would you respond to a failed ADF pipeline?
8. How would you investigate a data freshness breach?
9. How would you handle a data quality incident?
10. How would you troubleshoot an ADLS Gen2 permission failure?
11. How would you investigate Databricks job failure?
12. How would you investigate Kafka consumer lag?
13. How do you recover from duplicate records?
14. What is containment, and why is it important?
15. What is a post-incident review?
16. Why should recovery instructions be tested?
17. How do you communicate a production incident to business stakeholders?
18. Design a runbook for a daily orders pipeline failure.

**Interview readiness:** Be prepared to explain how you would handle a production incident from detection through recovery and post-incident review. Use a realistic pipeline example and emphasize business impact, safe recovery, data validation, and prevention.

