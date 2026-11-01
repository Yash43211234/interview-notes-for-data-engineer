# Production Incidents – Data Engineering Interview Notes

## 1. What Is a Production Incident?

A production incident is an unexpected problem in a live data system that affects its availability, reliability, performance, security, or data correctness.

Examples:
- A scheduled data pipeline fails.
- A dashboard displays incorrect revenue.
- A Databricks job runs much longer than usual.
- Duplicate records appear in a Gold table.
- An ADLS Gen2 permission error prevents data ingestion.
- A streaming pipeline stops processing events.
- Sensitive data becomes accessible to unauthorized users.

The objective during an incident is to:
1. Understand the impact.
2. Restore the service or data as safely as possible.
3. Identify the root cause.
4. Prevent the problem from happening again.

## 2. What Is Incident Management?

Incident management is the process of identifying, investigating, resolving, and documenting production problems.

A typical incident lifecycle:

```text
Incident Detected
       |
       v
Assess Impact and Severity
       |
       v
Assign Ownership
       |
       v
Investigate and Diagnose
       |
       v
Mitigate / Restore Service
       |
       v
Validate Recovery
       |
       v
Root Cause Analysis
       |
       v
Preventive Actions
       |
       v
Close Incident
```

### Important Incident Management Terms

| Term | Meaning |
|---|---|
| Incident | An unexpected production problem |
| Alert | Notification that a condition needs attention |
| Severity | Level of business or operational impact |
| Priority | Order in which an issue should be handled |
| Escalation | Involving a team with additional responsibility or expertise |
| Mitigation | Reducing the impact of a problem |
| Resolution | Fixing the problem |
| RCA | Root Cause Analysis |
| Postmortem | Review of an incident and its causes |
| Runbook | Documented operational procedure |
| SLA | Service Level Agreement |
| SLO | Service Level Objective |
| SLI | Service Level Indicator |

## 3. Severity and Priority

Severity describes the impact of an incident. Priority determines how urgently it should be addressed.

Example severity classification:

| Severity | Example |
|---|---|
| Sev 1 – Critical | Production data platform unavailable or major business process stopped |
| Sev 2 – High | Critical pipeline failing and affecting important reports |
| Sev 3 – Medium | Limited data delay or non-critical workload failure |
| Sev 4 – Low | Minor issue with a workaround and limited impact |

Actual severity definitions and response times depend on company policy.

### How Do You Decide Incident Severity?

Consider:
- Number of affected users.
- Business-criticality of the pipeline.
- Data loss or corruption.
- Security or privacy impact.
- Duration of the outage.
- Availability of a workaround.
- Downstream dependencies.

A pipeline failure is not automatically a critical incident. Its impact determines its severity.

## 4. First Response to a Production Incident

### Q1. What Would You Do When a Production Pipeline Fails?

Use a structured approach:

**Step 1: Understand the alert**
- Identify the failed pipeline.
- Check the run ID.
- Note the failure timestamp.
- Read the error message.

**Step 2: Assess the impact**
- Which datasets are affected?
- Are downstream jobs blocked?
- Are reports delayed?
- Is data missing or incorrect?
- Is there a business deadline?

**Step 3: Investigate**
- Check source availability.
- Check authentication and permissions.
- Inspect pipeline and activity logs.
- Check schema and data-quality errors.
- Review recent code or configuration changes.

**Step 4: Mitigate**
- Retry only if the failure is temporary and retry is safe.
- Restore access if permissions changed.
- Correct configuration if approved.
- Rerun from a safe source position.
- Use a documented fallback if available.

**Step 5: Validate**
- Confirm pipeline completion.
- Compare source and target.
- Check duplicates and missing records.
- Verify downstream reports.

**Step 6: Communicate and document**
- Share impact and current status.
- Record the cause.
- Document corrective and preventive actions.

### Interview Answer

"When a production pipeline fails, I first check the run details and error logs to understand what failed. Then I assess the business impact and identify affected downstream datasets. I investigate the source, connectivity, permissions, schema, and recent changes. After identifying the issue, I apply a safe mitigation, rerun from the last successful processing point if appropriate, and validate the output. Finally, I document the root cause and preventive actions."

---

## 5. Common Data Engineering Production Incidents

### Incident 1: ADF Pipeline Failure

**Scenario:** A scheduled Azure Data Factory pipeline fails unexpectedly.

Possible causes:
- Source database unavailable.
- Linked Service authentication failure.
- Integration Runtime unavailable.
- Network connectivity issue.
- Invalid source query.
- Schema mismatch.
- Storage permission failure.

Investigation:

1. Open ADF Monitor.
2. Identify the failed pipeline run.
3. Open the failed activity.
4. Review the error code and message.
5. Check the linked service and dataset configuration.
6. Verify Integration Runtime health.
7. Check source and target connectivity.
8. Review recent changes.

Resolution depends on the actual error.

**Important:** Do not repeatedly retry a permanent failure such as an invalid query or missing permission.

### Incident 2: Databricks Job Failure

**Scenario:** A scheduled Databricks job fails during execution.

Possible causes:
- Executor out of memory.
- Driver out of memory.
- Invalid transformation.
- Missing input files.
- Schema incompatibility.
- Cluster startup failure.
- Access denied.
- Delta transaction conflict.

Investigation:

- Check job run logs.
- Inspect notebook output and exception details.
- Open Spark UI.
- Identify the failed stage and task.
- Review executor logs.
- Inspect the query execution plan.
- Check recent code and configuration changes.

Possible solutions:
- Correct the transformation.
- Fix the input schema.
- Adjust join strategy.
- Address data skew.
- Reduce unnecessary data collection to the driver.
- Correct access permissions.
- Handle transaction conflicts safely.

### Incident 3: Pipeline Runs Slowly

**Scenario:** A pipeline that normally completes in 30 minutes now takes 3 hours.

Possible causes:
- Increased input data volume.
- Data skew.
- Excessive shuffle.
- Small files.
- Inefficient joins.
- Poor partition pruning.
- Source database slowdown.
- Resource contention.

Investigation:

1. Compare current and historical execution times.
2. Identify the slowest activity or Spark stage.
3. Compare input data volume.
4. Check task-duration distribution.
5. Review shuffle read/write and spill.
6. Inspect the execution plan.
7. Check source and target performance.

Possible solutions:
- Filter unnecessary records early.
- Select only required columns.
- Optimize joins.
- Handle skewed keys.
- Compact small files where beneficial.
- Improve partitioning based on workload.
- Scale compute only when metrics justify it.

**Interview point:** Increasing cluster size should not be the first response. Identify the bottleneck first.

### Incident 4: Duplicate Records in Production

**Scenario:** A Gold table contains duplicate orders after a pipeline retry.

Possible causes:
- Append-only write logic.
- Retry after a partial write.
- Duplicate source events.
- Overlapping pipeline executions.
- Incorrect business key.
- Non-deterministic deduplication.

Investigation:

1. Identify the business key.
2. Check duplicate counts.
3. Compare ingestion timestamps.
4. Inspect pipeline run history.
5. Check whether retries occurred.
6. Review the target write logic.

Example duplicate check:

```sql
SELECT
    order_id,
    COUNT(*) AS record_count
FROM gold_orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

Possible solutions:
- Deduplicate the source deterministically.
- Use an idempotent MERGE.
- Control concurrent writes.
- Correct retry behavior.
- Reconcile and repair affected target records.

**Important:** `dropDuplicates()` does not guarantee which duplicate record is retained. Use a deterministic ordering rule when selecting the latest record.

### Incident 5: Missing Records in Target

**Scenario:** Source has 100,000 records, but the target contains only 95,000 expected records.

Possible causes:
- Incorrect incremental filter.
- Watermark advanced too early.
- Source records arrived late.
- Records were rejected by validation.
- Incorrect joins or filters.
- Failed or incomplete writes.
- Deletes or updates were handled incorrectly.

Investigation:

1. Confirm the source and target time windows.
2. Compare business keys.
3. Check rejected and quarantined records.
4. Review watermark values.
5. Check pipeline execution history.
6. Inspect transformation filters.
7. Compare source and target aggregates.

Possible solutions:
- Reprocess missing records.
- Correct watermark logic.
- Add an overlap window or CDC.
- Fix filtering and transformation logic.
- Add source-to-target reconciliation.

### Incident 6: Incorrect Business Metrics

**Scenario:** A Power BI dashboard shows revenue that is higher than the source system.

Possible causes:
- Join fanout.
- Duplicate facts.
- Incorrect aggregation.
- Refunds not deducted.
- Cancelled orders included.
- Different currency conversions.
- Incorrect reporting date or timezone.

Investigation:

1. Confirm the business definition of revenue.
2. Check the fact-table grain.
3. Validate joins and dimension relationships.
4. Check duplicate business keys.
5. Compare totals at multiple levels.
6. Verify refunds, cancellations, discounts, and currency handling.

Example:

```sql
SELECT
    COUNT(*) AS row_count,
    COUNT(DISTINCT order_id) AS unique_orders,
    SUM(net_amount) AS net_revenue
FROM gold_orders;
```

**Important:** A correct row count does not guarantee correct business metrics. Verify the calculation and business rules.

### Incident 7: ADLS Gen2 Access Denied

**Scenario:** A Databricks job can connect to ADLS Gen2 but receives an authorization error when reading or writing a directory.

Possible causes:
- Managed identity lacks data permissions.
- Incorrect service principal.
- Missing Storage Blob Data role.
- Missing ACL permissions.
- Missing execute permission on parent directories.
- Storage firewall or private networking issue.

Investigation:

1. Identify the identity used by the job.
2. Confirm authentication succeeded.
3. Check Azure RBAC data roles.
4. Check filesystem and directory ACLs.
5. Verify parent-directory traversal permissions.
6. Check storage networking and DNS.

**Important:** Authentication proves the identity. Authorization determines whether it can access the requested data.

### Incident 8: Schema Mismatch

**Scenario:** A pipeline fails because the source schema changed.

Example:

Earlier:

```text
customer_id: integer
customer_name: string
```

Current:

```text
customer_id: string
customer_name: string
```

Possible causes:
- Source data type changed.
- A required column was removed.
- A new column was introduced.
- Incorrect file format or schema inference.
- Incompatible target schema.

Resolution:

1. Compare source and expected schemas.
2. Identify whether the change is additive or breaking.
3. Check business impact.
4. Validate compatibility.
5. Apply approved schema evolution or transformation.
6. Quarantine incompatible records if needed.
7. Update downstream consumers.

Do not automatically accept every schema change in a production pipeline.

### Incident 9: Streaming Pipeline Stops Processing

**Scenario:** A Structured Streaming job is running, but new events are not appearing in the target.

Possible causes:
- Source connectivity failure.
- Invalid event format.
- Query failure.
- Checkpoint or state-store problem.
- Target write failure.
- Authentication failure.
- Processing backlog.

Investigation:

- Check streaming query status.
- Review recent progress information.
- Inspect input and processed row counts.
- Check processing latency.
- Review source offsets and checkpoint state.
- Inspect sink errors.
- Check event-time and watermark behavior.

Resolution:
- Restore source or target connectivity.
- Correct invalid input handling.
- Fix transformation logic.
- Restart safely using the correct checkpoint.
- Reconcile source and target after recovery.

**Important:** Do not delete a streaming checkpoint simply to make a query start. It can cause data to be reprocessed or state to be lost.

### Incident 10: Small Files Affecting Performance

**Scenario:** A Delta table has millions of small files and queries are slow.

Possible causes:
- Frequent micro-batch writes.
- Excessive parallel output partitions.
- Poor file sizing.
- Too many tiny input files.

Investigation:
- Check file count and file-size distribution.
- Review partition layout.
- Check query scan metrics.
- Identify the write pattern.

Possible solutions:
- Use OPTIMIZE where appropriate.
- Tune output partitioning.
- Configure optimized writes or auto compaction when supported.
- Avoid unnecessary partition columns.

### Incident 11: Pipeline Produces Stale Data

**Scenario:** A daily report is expected by 7 AM but still displays yesterday's data at 10 AM.

Possible causes:
- Upstream pipeline delay.
- Failed dependency.
- Source data not available.
- Slow processing.
- Failed Gold-table publication.
- Incorrect freshness monitoring.

Investigation:

1. Check the latest successful source ingestion.
2. Check pipeline dependencies.
3. Identify the slowest task.
4. Check target table update time.
5. Confirm whether the report uses the latest committed data.

Resolution:
- Fix the upstream bottleneck.
- Rerun missing processing safely.
- Validate the Gold dataset.
- Notify consumers of the delay.
- Add freshness alerts.

### Incident 12: Delta MERGE Failure

**Scenario:** A MERGE operation fails because multiple source records match the same target key.

Possible causes:
- Duplicate source business keys.
- Multiple CDC events for one key.
- Missing deterministic ordering.
- Incorrect join condition.

Investigation:

```sql
SELECT
    customer_id,
    COUNT(*) AS record_count
FROM source_customers
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

Resolution:
- Deduplicate the source.
- Select the correct event using a source sequence or timestamp.
- Validate business keys.
- Retry the MERGE after confirming the source is unambiguous.

### Incident 13: Source Database Becomes Slow

**Scenario:** A data pipeline causes high load on the production source database.

Possible causes:
- Full-table scans.
- Too many parallel extraction queries.
- Missing or ineffective indexes.
- Large unfiltered queries.
- Excessive concurrent pipeline runs.

Possible solutions:
- Use incremental extraction or CDC.
- Apply suitable source filters.
- Limit extraction parallelism.
- Schedule workloads appropriately.
- Use replicas or staging systems where supported.
- Coordinate with database owners.

Production source performance should be protected even when the data pipeline is behind schedule.

---

## 6. Root Cause Analysis (RCA)

### What Is Root Cause Analysis?

Root Cause Analysis identifies the underlying reason an incident occurred.

It goes beyond identifying the immediate error.

Example:

**Incident:** Daily customer pipeline failed.

- Symptom: Databricks could not write to ADLS.
- Immediate cause: Authorization failure.
- Root cause: The managed identity's required data role was removed during a security configuration change.
- Corrective action: Restore the approved permission.
- Preventive action: Add access-change monitoring and review identity permissions during deployment.

### What Is the 5 Whys Technique?

The 5 Whys technique repeatedly asks why a problem occurred to reach an underlying cause.

Example:

**Problem:** Gold sales data was not updated.

1. Why? The Databricks job failed.
2. Why? The input schema changed.
3. Why? The source team changed a column's data type.
4. Why? The change was deployed without notifying the data team.
5. Why? No formal schema-change communication process existed.

Potential preventive action:
- Introduce data contracts.
- Add schema compatibility checks.
- Establish change notification procedures.

Five questions are a guide, not a requirement. Stop when the evidence identifies an actionable underlying cause.

### Corrective vs Preventive Actions

| Corrective Action | Preventive Action |
|---|---|
| Fixes the current problem | Reduces the chance of recurrence |
| Restores service or data | Improves system design or controls |
| Example: Rerun a failed pipeline | Example: Add automated failure detection |

### What Should an RCA Document Contain?

- Incident ID.
- Incident date and duration.
- Severity.
- Affected systems.
- Business impact.
- Detection method.
- Timeline of events.
- Root cause.
- Immediate mitigation.
- Corrective actions.
- Preventive actions.
- Owners and deadlines.
- Lessons learned.

---

## 7. Incident Communication

### How Should You Communicate During an Incident?

Communication should be:
- Clear.
- Factual.
- Timely.
- Focused on impact.
- Free from unsupported assumptions.

A useful status update:

```text
Incident: Daily Orders Pipeline Failure

Status: Under Investigation

Impact:
The latest orders dataset has not been refreshed.
The reporting layer may contain stale data.

Current Findings:
The Databricks job failed during the Silver-layer write.

Next Steps:
The team is checking storage permissions and job logs.
The target will be validated before downstream processing resumes.

Next Update:
As per the incident communication schedule.
```

Do not announce that data is fixed until validation is complete.

### When Should You Escalate an Incident?

Escalate when:
- Business-critical data is unavailable.
- Data corruption or loss is suspected.
- Security or privacy may be affected.
- The issue is outside your access or expertise.
- The incident exceeds its response target.
- Repeated mitigation attempts fail.
- Multiple dependent systems are affected.

Follow the organization's incident-response and escalation policy.

---

## 8. SLA, SLO, and SLI

### What Is an SLA?

A Service Level Agreement is an agreement defining the service expectations and responsibilities between a provider and a customer.

Example: A reporting service must be available 99.9% of the agreed measurement period.

### What Is an SLO?

A Service Level Objective is an internal or external target for a service-level indicator.

Example: 99% of daily pipelines should complete before 7 AM over the measurement period.

### What Is an SLI?

A Service Level Indicator is the actual measurement of service performance.

Examples:
- Pipeline success rate.
- Data freshness.
- Query latency.
- Service availability.
- Processing error rate.

### SLA vs SLO vs SLI

| Term | Meaning | Example |
|---|---|---|
| SLA | Formal service agreement | 99.9% service availability |
| SLO | Target performance objective | 99% of reports refreshed by 7 AM |
| SLI | Actual measured performance | 97% refreshed by 7 AM |

### What Is an Error Budget?

An error budget is the amount of unreliability allowed under an SLO.

For example, if the availability objective is 99.9%, the remaining 0.1% is the permitted unavailability within the defined measurement period.

Error-budget policies help teams balance reliability work and feature development.

---

## 9. Production Readiness Checklist

Before deploying a data pipeline to production, verify the following.

### Design
- [ ] Business requirements are clear.
- [ ] Source and target systems are identified.
- [ ] Data volume and frequency are understood.
- [ ] Full or incremental loading strategy is defined.
- [ ] Data-grain and business-key rules are documented.

### Reliability
- [ ] Retry policies are configured appropriately.
- [ ] Target writes are idempotent where required.
- [ ] Checkpoint or watermark handling is correct.
- [ ] Recovery and backfill procedures exist.
- [ ] Concurrent execution is controlled.

### Data Quality
- [ ] Schema validation exists.
- [ ] Null and duplicate checks exist.
- [ ] Business rules are validated.
- [ ] Source-to-target reconciliation is available.
- [ ] Invalid-record handling is defined.

### Security
- [ ] Managed identity or approved authentication is configured.
- [ ] Least-privilege access is applied.
- [ ] Secrets are not hardcoded.
- [ ] Sensitive data is protected.
- [ ] Network restrictions are reviewed.

### Monitoring
- [ ] Pipeline success and failure are monitored.
- [ ] Duration and throughput are tracked.
- [ ] Data freshness is monitored.
- [ ] Alerts and escalation paths are configured.
- [ ] Logs contain useful diagnostic information.

### Operations
- [ ] Runbook is documented.
- [ ] Owners are identified.
- [ ] Deployment and rollback procedures exist.
- [ ] Incident communication process is defined.
- [ ] Production support team knows the recovery steps.

---

## 10. Interview Scenarios and Answer Framework

### Q1. A Production Pipeline Failed at 6 AM. Business Reports Are Due at 8 AM. What Would You Do?

**Answer framework:**

1. Check the failed run and identify the error.
2. Determine the affected datasets and reports.
3. Assess whether a safe retry or workaround exists.
4. Inform relevant stakeholders about the impact.
5. Resolve the issue or involve the responsible team.
6. Rerun the pipeline from a safe processing position.
7. Validate the output and downstream reports.
8. Document the incident and follow-up actions.

**Key point:** Restore correct data safely. Do not sacrifice data quality just to meet the deadline.

### Q2. The Pipeline Shows Success, but the Data Is Incorrect. What Would You Do?

1. Treat it as a data-quality incident.
2. Identify affected tables and reporting periods.
3. Compare source and target keys and values.
4. Review transformations, joins, filters, and aggregations.
5. Identify the incorrect logic or input.
6. Correct and reprocess affected data.
7. Validate downstream metrics.
8. Add tests to prevent recurrence.

A successful job status proves execution completed, not that the business output is correct.

### Q3. A Retry May Create Duplicate Data. Would You Retry?

Not immediately.

First:
1. Check whether the previous run wrote any data.
2. Inspect the target state.
3. Identify the target business key.
4. Confirm whether the write is idempotent.
5. Use a safe recovery strategy.
6. Validate the result after retry.

### Q4. A Critical Pipeline Is Delayed Because the Source System Is Down. What Would You Do?

1. Confirm source unavailability.
2. Check whether an approved replica or alternate source exists.
3. Inform stakeholders about the delay.
4. Avoid generating misleading or incomplete output.
5. Resume extraction when the source is available.
6. Backfill missing data.
7. Reconcile the recovered data.

### Q5. A Team Member Suggests Deleting a Checkpoint to Fix a Streaming Failure. Would You Do It?

Not without understanding the consequences.

Deleting a checkpoint may cause:
- Previously processed data to be read again.
- Streaming state to be lost.
- Duplicate output.
- Incorrect stateful aggregations.

First investigate the checkpoint error and recovery options. Follow a controlled reset or new-checkpoint procedure only when the source and sink can safely support it.

### Q6. How Do You Handle a Data Breach or Unauthorized Data Access Incident?

1. Follow the organization's security incident process immediately.
2. Escalate to the security team.
3. Restrict or revoke affected access as directed.
4. Preserve relevant audit logs and evidence.
5. Identify affected systems and data.
6. Avoid unauthorized investigation or disclosure.
7. Support the approved containment and recovery process.
8. Follow applicable notification and compliance requirements.

Security incidents require immediate escalation; do not treat them as ordinary pipeline failures.

---

## 11. How to Explain Production Experience Honestly

Interviewers may ask:

- Have you handled a production incident?
- Have you resolved a failed pipeline?
- Have you performed root cause analysis?
- Have you worked on production monitoring?
- Have you supported data recovery?

If you personally handled an incident, explain:
1. The actual problem.
2. Your responsibility.
3. How you investigated it.
4. What you changed or recommended.
5. How the result was validated.

If you only observed or assisted, say so clearly.

### Example: If You Have Assisted With an Incident

"I have worked with data pipeline workflows and understand how to investigate failures using execution logs, error messages, and data validation. I would first identify the affected component and business impact, then investigate the source, transformation, permissions, or target write. I would coordinate with the responsible team, validate recovery, and document preventive actions."

Do not claim to have independently resolved a production incident if you only studied the scenario.

---

## 12. Common Production Incident Mistakes

1. Restarting a failed pipeline without understanding the cause.
2. Repeating retries for permanent failures.
3. Deleting streaming checkpoints without impact analysis.
4. Advancing watermarks before successful target writes.
5. Ignoring partial writes.
6. Assuming successful execution means correct data.
7. Fixing symptoms without investigating the root cause.
8. Failing to communicate business impact.
9. Making unapproved production changes.
10. Ignoring security incidents or delaying escalation.
11. Failing to preserve logs and evidence.
12. Closing incidents without validating recovery.
13. Not documenting lessons learned.
14. Blaming individuals instead of improving system controls.

---

## 13. Quick Revision

| Concept | Key Point |
|---|---|
| Production Incident | Unexpected issue affecting a live system |
| Incident Management | Detect, investigate, mitigate, recover, and learn |
| Severity | Degree of impact |
| Priority | Urgency of handling |
| Mitigation | Reduces immediate impact |
| Resolution | Fixes the issue |
| RCA | Identifies underlying cause |
| Postmortem | Reviews the incident and improvements |
| Runbook | Documented operational steps |
| SLA | Formal service agreement |
| SLO | Target service objective |
| SLI | Actual service measurement |
| Error Budget | Allowed unreliability under an SLO |
| Escalation | Involves additional ownership or expertise |
| Checkpoint | Stores progress and recovery information |
| Reprocessing | Processes data again |
| Idempotency | Makes safe repeated execution possible |
| Data Reconciliation | Compares source and target data |
| Data Freshness | Measures how recently data was updated |

---

## 14. Self-Assessment Checklist

### Incident Response
- [ ] Explain the production incident lifecycle.
- [ ] Assess business impact and severity.
- [ ] Explain mitigation versus resolution.
- [ ] Know when to escalate.
- [ ] Communicate incident status clearly.

### Pipeline Troubleshooting
- [ ] Investigate ADF failures.
- [ ] Investigate Databricks job failures.
- [ ] Troubleshoot slow pipelines.
- [ ] Diagnose duplicate and missing records.
- [ ] Troubleshoot ADLS access errors.
- [ ] Diagnose streaming failures.

### Recovery and Reliability
- [ ] Explain idempotent recovery.
- [ ] Handle retries safely.
- [ ] Explain checkpoint risks.
- [ ] Recover incremental pipelines.
- [ ] Plan backfills and reconciliation.

### Root Cause Analysis
- [ ] Explain the 5 Whys technique.
- [ ] Distinguish symptoms from root causes.
- [ ] Write corrective and preventive actions.
- [ ] Prepare an incident timeline.
- [ ] Document lessons learned.

### Production Readiness
- [ ] Explain SLA, SLO, and SLI.
- [ ] Explain monitoring and alerting.
- [ ] Understand security escalation.
- [ ] Explain runbooks and rollback procedures.
- [ ] Validate recovery before closing an incident.

**Preparation priority:** Practice explaining five scenarios clearly: ADF failure, slow Databricks job, duplicate records, missing data, and incorrect business metrics. For each one, explain the impact, investigation, safe mitigation, validation, root cause, and prevention. This is more valuable than memorizing a long list of error messages.