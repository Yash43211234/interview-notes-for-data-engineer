# Azure Data Factory: Triggers

## 1. What Is a Trigger?

A **Trigger** in Azure Data Factory (ADF) starts a pipeline run based on a configured condition or event.

Instead of manually starting a pipeline every time, triggers automate pipeline execution.

Triggers are commonly used for:

- Running pipelines at scheduled times.
- Processing data for fixed time intervals.
- Starting pipelines when files arrive in storage.
- Automating recurring data ingestion.
- Coordinating time-based data processing.

### Simple Example

Suppose a company receives sales data every day and wants to load it into ADLS Gen2 at 2:00 AM.

A Schedule Trigger can start the pipeline automatically every day.

```text
       Schedule Trigger
        Every day 2 AM
              |
              v
         ADF Pipeline
              |
              v
         Copy Activity
              |
              v
           ADLS Gen2
```

**Remember:** A trigger starts a pipeline. Activities perform the actual work.

---

## 2. Types of Triggers in ADF

Azure Data Factory provides three main trigger types:

1. Schedule Trigger.
2. Tumbling Window Trigger.
3. Event-based Trigger.

Event-based triggers include storage event triggers and custom event triggers.

<AsyncImageGroup query={["Azure Data Factory schedule trigger architecture diagram","Azure Data Factory tumbling window trigger dependency diagram","Azure Data Factory storage event trigger architecture diagram"]} aspectRatio="5:4" layout="carousel"/>

---

## 3. Schedule Trigger

### What Is a Schedule Trigger?

A Schedule Trigger starts a pipeline at a configured date and time or according to a recurring schedule.

It is useful when data processing must happen at fixed times.

### Example

A company wants to load customer data every day at 1:00 AM.

Configuration:

```text
Trigger Type: Schedule
Frequency: Day
Interval: 1
Start Time: 01:00 AM
Time Zone: Asia/Kolkata
```

The pipeline runs every day according to the configured schedule.

### Common Schedule Frequencies

- Minute.
- Hour.
- Day.
- Week.
- Month.

The exact recurrence settings depend on the selected frequency.

### Example: Every 30 Minutes

```text
Frequency: Minute
Interval: 30
```

### Example: Every Week

```text
Frequency: Week
Interval: 1
Selected Days: Monday, Wednesday, Friday
```

### Characteristics

- Time-based execution.
- Supports recurring schedules.
- Does not depend on a file-arrival event.
- Useful for batch data processing.
- Uses the configured time zone.

### Use Cases

- Daily warehouse loading.
- Hourly data ingestion.
- Weekly reporting pipelines.
- Scheduled data quality checks.

---

## 4. Tumbling Window Trigger

### What Is a Tumbling Window Trigger?

A Tumbling Window Trigger starts pipeline runs for fixed, continuous, non-overlapping time intervals.

Each run is associated with a specific time window.

It is useful when data must be processed in time-based batches with clear window boundaries.

### Example

Suppose a company processes sales data every hour.

The trigger creates windows such as:

```text
Window 1: 00:00 - 01:00
Window 2: 01:00 - 02:00
Window 3: 02:00 - 03:00
Window 4: 03:00 - 04:00
```

Each window can start a separate pipeline run.

### Architecture

```text
       Tumbling Window Trigger
                  |
        +---------+---------+
        |         |         |
        v         v         v
     00-01     01-02     02-03
        |         |         |
        v         v         v
    Pipeline  Pipeline  Pipeline
      Run 1     Run 2     Run 3
```

### Important Characteristics

- Windows are fixed and non-overlapping.
- Each run has a defined start and end time.
- Supports dependencies between windows.
- Supports retry policies for failed windows.
- Useful for time-based batch processing and backfills.

### Trigger Parameters

Tumbling Window Trigger exposes useful values:

```text
@trigger().startTime
```

```text
@trigger().endTime
```

These values can be passed to a pipeline to identify the data interval to process.

### Example: Processing One Hour of Data

Suppose the current window is:

```text
Start Time: 2026-10-25T01:00:00Z
End Time:   2026-10-25T02:00:00Z
```

The pipeline can use these values to extract records for that interval.

Example SQL:

```sql
SELECT *
FROM dbo.Sales
WHERE order_timestamp >= @WindowStart
  AND order_timestamp < @WindowEnd;
```

The half-open interval avoids overlapping records between adjacent windows.

### Window Dependencies

A Tumbling Window Trigger can be configured with dependencies on earlier windows.

Example:

```text
Window 1
   |
   v
Window 2
   |
   v
Window 3
```

This is useful when a later processing window must wait for an earlier window to complete.

### Retry and Recovery

If a window fails, its retry policy can be used to retry execution.

Failed windows can also be addressed through monitoring and rerun or backfill procedures.

**Important:** A Tumbling Window Trigger is not the same as a Schedule Trigger. It provides explicit time-window boundaries and window dependency capabilities.

---

## 5. Event-Based Trigger

An Event-Based Trigger starts a pipeline when a configured event occurs.

It is useful when processing should begin because something happened rather than because a specific time arrived.

ADF supports storage event triggers and custom event triggers.

### 5.1 Storage Event Trigger

A Storage Event Trigger starts a pipeline when a supported storage event occurs.

Common examples include:

- Blob created.
- Blob deleted.

### Example

Suppose a company uploads a sales CSV file to ADLS Gen2.

When the file is created, ADF automatically starts a pipeline.

```text
       ADLS Gen2
            |
            v
      File Created
            |
            v
    Storage Event Trigger
            |
            v
       ADF Pipeline
            |
            v
      Process CSV File
            |
            v
       Curated Data
```

### Common Configuration

```text
Trigger Type: Storage Event
Storage Account: mystorageaccount
Container: raw
Event: Blob created
Folder Path: sales/
File Extension: .csv
```

The exact filtering options depend on the trigger configuration and supported storage events.

### Common Use Cases

- Process files after upload.
- Ingest customer files automatically.
- Start data validation when a file arrives.
- Trigger downstream processing after landing data.

### Important Considerations

- Storage events are delivered through Azure Event Grid.
- The required event subscription and permissions must be configured.
- Folder and file filters help avoid triggering on unrelated files.
- File arrival does not guarantee that the file is valid or complete.
- Duplicate or repeated event handling should be considered in pipeline design.

### 5.2 Custom Event Trigger

A Custom Event Trigger starts a pipeline when a matching custom event is published to Azure Event Grid.

It is useful when a pipeline must respond to an application or business event.

### Example

Suppose an application publishes a custom event after completing an export.

```text
Application
    |
    v
Publish Custom Event
    |
    v
Azure Event Grid
    |
    v
ADF Custom Event Trigger
    |
    v
ADF Pipeline
    |
    v
Process Exported Data
```

### Use Cases

- Start processing after an application completes a task.
- Trigger a pipeline after an upstream system publishes a business event.
- Coordinate workflows across different applications.

---

## 6. Comparison of Trigger Types

| Feature | Schedule | Tumbling Window | Event-Based |
|---|---|---|---|
| Starts pipeline based on | Time schedule | Fixed time windows | Events |
| Recurring execution | Yes | Yes | When matching events occur |
| Explicit window boundaries | No | Yes | No |
| Window dependencies | No | Yes | No |
| Typical use case | Daily batch load | Hourly interval processing | File arrival |
| Example | Every day at 2 AM | Process each hour | Blob created |

### Easy Way to Remember

- Schedule Trigger → Run at a particular time.
- Tumbling Window Trigger → Process fixed time intervals.
- Event-Based Trigger → Run when an event occurs.

---

## 7. Trigger vs Pipeline vs Activity

These are different components of ADF.

| Component | Responsibility |
|---|---|
| Trigger | Starts pipeline execution |
| Pipeline | Coordinates the workflow |
| Activity | Performs an individual operation |

### Example

```text
Schedule Trigger
       |
       v
   ADF Pipeline
       |
       v
   Lookup Activity
       |
       v
   Copy Activity
       |
       v
 Databricks Activity
```

The trigger starts the pipeline, and the pipeline executes its activities according to their dependencies.

---

## 8. Trigger Parameters and Pipeline Parameters

Triggers can pass values to pipeline parameters when starting a pipeline.

This is useful for dynamic processing.

### Example

Suppose a Tumbling Window Trigger starts a pipeline for a particular time interval.

The pipeline has two parameters:

```text
windowStart: String
windowEnd: String
```

Pass trigger values:

```text
windowStart = @trigger().startTime
windowEnd   = @trigger().endTime
```

The pipeline can use these values in a source query.

### Example SQL

```sql
SELECT *
FROM dbo.Orders
WHERE order_timestamp >= @WindowStart
  AND order_timestamp < @WindowEnd;
```

The query is illustrative. Actual parameter binding depends on the source connector and Copy Activity configuration.

### Benefits

- Dynamic time-based extraction.
- Reusable pipelines.
- Easier backfill processing.
- Avoids hardcoding processing dates.

---

## 9. Trigger States and Management

Triggers can be managed from the Manage section of ADF Studio.

Common operations include:

- Create.
- Start.
- Stop.
- Edit.
- Monitor.

### Important

Creating a trigger does not automatically mean it is running.

For schedule and tumbling window triggers, the trigger must be started after configuration and publishing as required.

For event-based triggers, the event subscription must also be correctly configured.

When a trigger is stopped, it does not start new pipeline runs through that trigger. Already-started pipeline runs are separate executions and are not automatically cancelled.

---

## 10. Trigger Monitoring

ADF provides monitoring information for trigger execution.

You can inspect:

- Trigger name.
- Trigger type.
- Trigger run time.
- Trigger status.
- Pipeline run associated with the trigger.
- Failure details, where available.

### Steps

1. Open Azure Data Factory Studio.
2. Navigate to Monitor.
3. Select Trigger Runs.
4. Select the required time range.
5. Find the trigger execution.
6. Inspect its status and associated pipeline run.

### Important Difference

- Pipeline Runs show pipeline execution.
- Activity Runs show individual activity execution.
- Trigger Runs show trigger execution information.

---

## 11. Common Trigger Problems

| Problem | Possible Cause |
|---|---|
| Pipeline does not start | Trigger is stopped or not started |
| Schedule runs at an unexpected time | Incorrect time zone or schedule configuration |
| File upload does not trigger pipeline | Event subscription or storage configuration issue |
| Unwanted files trigger pipeline | Filters are too broad |
| Tumbling window is delayed | Previous window dependency or execution issue |
| Pipeline receives incorrect dates | Incorrect trigger-to-pipeline parameter mapping |
| Duplicate processing | Repeated events or non-idempotent pipeline design |

### Troubleshooting Steps

1. Confirm the trigger is started.
2. Check the trigger configuration.
3. Verify the configured time zone.
4. Inspect Trigger Runs.
5. Inspect the associated Pipeline Run.
6. For storage events, verify Event Grid subscription and permissions.
7. Check file path and extension filters.
8. Verify trigger parameter mapping.
9. Check whether the pipeline is safe to rerun.

---

## 12. Best Practices

1. Choose the trigger based on the business requirement.
2. Use Schedule Triggers for ordinary recurring batch jobs.
3. Use Tumbling Window Triggers when explicit time windows and dependencies are needed.
4. Use Event-Based Triggers when processing should start after an event.
5. Pass time-window values dynamically to pipelines.
6. Use half-open intervals for time-based extraction.
7. Configure event filters carefully.
8. Make event-triggered processing idempotent.
9. Monitor trigger runs and associated pipeline runs.
10. Check time-zone settings for schedule-based execution.
11. Configure retries and recovery procedures for appropriate workloads.
12. Avoid assuming that file creation means the file is fully uploaded and ready for processing.
13. Use logging and alerts for important production pipelines.
14. Document trigger schedules, dependencies, and expected behavior.

---

## 13. Common Interview Questions

### Q1. What is a Trigger in Azure Data Factory?

A Trigger starts a pipeline based on a configured schedule, time window, or event.

### Q2. What are the three main trigger types in ADF?

Schedule Trigger, Tumbling Window Trigger, and Event-Based Trigger.

### Q3. What is a Schedule Trigger?

It starts a pipeline at configured times or according to a recurring schedule.

### Q4. What is a Tumbling Window Trigger?

It starts pipeline runs for fixed, continuous, non-overlapping time intervals. It also supports window dependencies and retry policies.

### Q5. What is an Event-Based Trigger?

It starts a pipeline when a configured event occurs, such as a supported storage event or a matching custom event.

### Q6. What is the difference between Schedule and Tumbling Window Trigger?

A Schedule Trigger starts pipelines according to a time schedule. A Tumbling Window Trigger creates explicit fixed time windows and supports dependencies between windows.

### Q7. When would you use a Storage Event Trigger?

When a pipeline should start after a supported storage event, such as a file being created in ADLS Gen2 or Blob Storage.

### Q8. What is the difference between a trigger and a pipeline?

A trigger starts pipeline execution. A pipeline contains activities and coordinates the workflow.

### Q9. Can a trigger pass parameters to a pipeline?

Yes. Trigger properties and configured values can be passed to pipeline parameters when the pipeline is started.

### Q10. How do you process hourly data using ADF?

I can use a Tumbling Window Trigger with an hourly interval, pass the window start and end times to the pipeline, and extract records for that interval.

### Q11. How do you process a file automatically after upload?

I can configure a Storage Event Trigger for the relevant storage account, container, and file filters. When the matching event occurs, ADF starts the pipeline.

### Q12. What is Azure Event Grid's role in Storage Event Triggers?

Azure Event Grid delivers storage events to the configured event subscription, which allows ADF to respond to supported events.

### Q13. How do you troubleshoot a trigger that is not starting a pipeline?

I check whether the trigger is started, inspect Trigger Runs, validate its configuration and time zone, and verify Event Grid subscription settings for event-based triggers.

### Q14. Can a stopped trigger cancel an already-running pipeline?

No. Stopping a trigger prevents new runs from being started by that trigger. It does not automatically cancel pipeline runs that have already started.

### Q15. Why is idempotency important for event-triggered pipelines?

Events may be delivered or handled more than once. Idempotent processing helps prevent duplicate or inconsistent target data.

---

## 14. Quick Revision

| Topic | Key Point |
|---|---|
| Trigger | Starts a pipeline |
| Schedule Trigger | Time-based recurring execution |
| Tumbling Window Trigger | Fixed, non-overlapping time windows |
| Event-Based Trigger | Starts on a matching event |
| Storage Event | Example: Blob created |
| Custom Event | Application publishes an event |
| Event Grid | Delivers supported events |
| `@trigger().startTime` | Trigger window start time |
| `@trigger().endTime` | Trigger window end time |
| Trigger Run | Trigger execution information |
| Pipeline Run | Pipeline execution |
| Activity Run | Individual activity execution |

### Most Important Takeaways

1. Know the three trigger types.
2. Understand Schedule vs Tumbling Window.
3. Know how Storage Event Triggers work with Event Grid.
4. Understand how trigger values are passed to pipeline parameters.
5. Know how to monitor and troubleshoot triggers.
6. Understand why event-driven pipelines should be idempotent.

---

## 15. Self-Assessment

Try answering these questions without looking at the notes:

1. What is a Trigger in ADF?
2. What are the three main trigger types?
3. How does a Schedule Trigger work?
4. What is a Tumbling Window Trigger?
5. What is the difference between Schedule and Tumbling Window?
6. What is an Event-Based Trigger?
7. How does a Storage Event Trigger work?
8. What is Azure Event Grid's role?
9. How can you pass trigger start and end times to a pipeline?
10. How would you process hourly sales data?
11. How would you trigger a pipeline after a CSV file arrives?
12. How do you monitor trigger execution?
13. What can cause a trigger not to start a pipeline?
14. Does stopping a trigger cancel an already-running pipeline?
15. Why is idempotency important for event-driven pipelines?

**Interview readiness:** You should be able to select the correct trigger for a business requirement, explain the difference between schedule and tumbling window execution, and design a file-arrival pipeline using a Storage Event Trigger.