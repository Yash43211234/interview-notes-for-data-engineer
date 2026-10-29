# Logging in Python

## 1. What Is Logging?

**Logging** is the process of recording events that occur while a program is running.

Logs help developers and data engineers understand what happened during execution, identify errors, monitor pipelines, and troubleshoot failures.

In data engineering, logging is useful for:

- Tracking ETL pipeline execution.
- Recording successful and failed data ingestion.
- Monitoring row counts and processing duration.
- Investigating malformed records.
- Identifying database or API failures.
- Auditing important processing events.

Example:

```python
import logging

logging.basicConfig(level=logging.INFO)

logging.info("Data ingestion started")
logging.warning("Some records contain missing values")
logging.error("Failed to load the data")
```

Output resembles:

```text
INFO:root:Data ingestion started
WARNING:root:Some records contain missing values
ERROR:root:Failed to load the data
```

The exact output depends on the logging configuration.

**Key point:** Logging is generally better than using `print()` for production applications because it supports severity levels, timestamps, output destinations, formatting, and exception information.

---

## 2. Logging Levels

Python's `logging` module provides five commonly used severity levels.

| Level | Numeric value | Purpose |
|---|---:|---|
| `DEBUG` | 10 | Detailed information useful during development |
| `INFO` | 20 | Normal application progress |
| `WARNING` | 30 | Unexpected situation that does not necessarily stop execution |
| `ERROR` | 40 | An operation failed |
| `CRITICAL` | 50 | A severe failure that may prevent the application from continuing |

Example:

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format="%(levelname)s: %(message)s"
)

logging.debug("Loading configuration")
logging.info("Pipeline started")
logging.warning("Three records were skipped")
logging.error("Database write failed")
logging.critical("Pipeline cannot continue")
```

Output:

```text
DEBUG: Loading configuration
INFO: Pipeline started
WARNING: Three records were skipped
ERROR: Database write failed
CRITICAL: Pipeline cannot continue
```

### When Should You Use Each Level?

- **DEBUG:** Record detailed diagnostic information, such as intermediate processing decisions.
- **INFO:** Record meaningful milestones, such as the beginning and completion of a batch.
- **WARNING:** Record unexpected but recoverable conditions.
- **ERROR:** Record failures in individual operations.
- **CRITICAL:** Record severe failures that threaten the overall operation.

Choose levels according to the importance of the event, not merely the type of message.

---

## 3. Configuring Logging

The `basicConfig()` function provides a simple way to configure logging.

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s"
)

logging.info("Pipeline started")
```

Example output:

```text
2026-10-29 09:30:00,123 - INFO - Pipeline started
```

The timestamp is illustrative; the actual value comes from the system running the program.

### Common Configuration Parameters

| Parameter | Purpose |
|---|---|
| `level` | Sets the minimum severity to process |
| `format` | Defines the structure of each log message |
| `datefmt` | Controls timestamp formatting |
| `filename` | Sends logs to a file |
| `filemode` | Controls how the log file is opened |
| `handlers` | Specifies one or more output destinations |

For example:

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S"
)

logging.info("Records processed successfully")
```

**Important:** `basicConfig()` normally does nothing if the root logger already has handlers configured. In scripts where you intentionally want to replace existing root handlers, `force=True` is available in modern Python versions, but use it carefully because it can remove existing configuration.

---

## 4. Logging to a File

For long-running jobs, logs are often written to files instead of only appearing in the terminal.

```python
import logging

logging.basicConfig(
    filename="pipeline.log",
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s"
)

logging.info("Pipeline started")
logging.info("Data extraction completed")
logging.warning("Two invalid records were found")
```

The messages are written to `pipeline.log`.

### Append vs. Overwrite

By default, file-based logging uses append mode.

You can specify the mode explicitly:

```python
import logging

logging.basicConfig(
    filename="pipeline.log",
    filemode="a",
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s"
)
```

- `"a"` appends to an existing log file.
- `"w"` overwrites the existing file when the logging handler opens it.

For most production jobs, retaining existing logs or using managed log rotation is preferable to accidentally overwriting them.

---

## 5. Understanding Log Message Formatting

The `format` parameter supports placeholders that describe each log record.

Common placeholders include:

| Placeholder | Meaning |
|---|---|
| `%(asctime)s` | Human-readable timestamp |
| `%(levelname)s` | Severity level |
| `%(name)s` | Logger name |
| `%(message)s` | Log message |
| `%(filename)s` | Source filename |
| `%(lineno)d` | Source line number |
| `%(funcName)s` | Function where logging occurred |
| `%(process)d` | Process ID |
| `%(thread)d` | Thread ID |

Example:

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(filename)s:%(lineno)d - %(message)s"
)

def load_data():
    logging.info("Loading source data")

load_data()
```

Example output:

```text
2026-10-29 09:30:00,123 - INFO - app.py:10 - Loading source data
```

The filename and line number depend on the actual code location.

This format is useful when investigating problems across multiple modules.

---

## 6. Logging Variables Correctly

You can include dynamic values in log messages.

```python
import logging

logging.basicConfig(level=logging.INFO)

rows_processed = 1500

logging.info("Rows processed: %s", rows_processed)
```

Output:

```text
INFO:root:Rows processed: 1500
```

Prefer this parameterized form:

```python
logging.info("Rows processed: %s", rows_processed)
```

rather than formatting the string before passing it:

```python
logging.info(f"Rows processed: {rows_processed}")
```

The parameterized form lets the logging system defer message formatting when the message is filtered out by the configured level.

### Logging Multiple Values

```python
import logging

source = "orders.csv"
rows = 2500
duration_seconds = 4.8

logging.info(
    "Source=%s, rows=%d, duration_seconds=%.2f",
    source,
    rows,
    duration_seconds
)
```

Output:

```text
INFO:root:Source=orders.csv, rows=2500, duration_seconds=4.80
```

Use placeholders that match the data types where appropriate.

---

## 7. Creating a Logger

For larger applications, create a named logger instead of relying on the root logger everywhere.

```python
import logging

logger = logging.getLogger(__name__)

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s"
)

logger.info("Orders module initialized")
```

Here:

- `getLogger(__name__)` creates or retrieves a logger named after the current module.
- `__name__` is `"__main__"` when a Python file is executed directly.
- When imported, it normally contains the module's qualified name.

Named loggers make it easier to identify which module generated a message.

### Why Use Named Loggers?

In a data pipeline, different modules might handle:

- Source extraction.
- Data validation.
- Transformation.
- Destination loading.

Named loggers make the origin of each message easier to identify.

**Best practice:** Use `logging.getLogger(__name__)` in reusable modules and configure handlers and levels centrally in the application entry point.

---

## 8. Logging Exceptions

When an operation fails, logging the exception details is often essential.

### Using `logging.exception()`

```python
import logging

logging.basicConfig(level=logging.ERROR)

try:
    result = 10 / 0
except ZeroDivisionError:
    logging.exception("Calculation failed")
```

`logging.exception()` logs a message at `ERROR` level and includes the exception traceback. It should normally be called inside an exception handler.

Example output resembles:

```text
ERROR:root:Calculation failed
Traceback (most recent call last):
...
ZeroDivisionError: division by zero
```

The traceback includes the call stack that led to the exception.

### Using `exc_info=True`

You can also include exception details with another logging method:

```python
import logging

try:
    int("abc")
except ValueError:
    logging.error("Invalid integer input", exc_info=True)
```

### `logger.error()` vs. `logger.exception()`

- `logger.error("Operation failed")` records an error message without automatically including the active exception traceback.
- `logger.exception("Operation failed")` records an error message with exception information when called inside an exception handler.

Avoid logging an exception and then silently continuing if the failed operation is essential. Handle, recover from, or re-raise the exception according to the pipeline's requirements.

---

## 9. Logging in a Data Pipeline

Consider a pipeline that reads records, validates them, and writes the result.

```python
import logging
from pathlib import Path

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s"
)

logger = logging.getLogger(__name__)

def process_file(file_path):
    path = Path(file_path)

    logger.info("Processing started: %s", path.name)

    try:
        with path.open("r", encoding="utf-8") as file:
            rows = file.readlines()

        logger.info(
            "Read completed: file=%s rows=%d",
            path.name,
            len(rows)
        )

        # Transformation and loading would occur here.

        logger.info("Processing completed: %s", path.name)

    except FileNotFoundError:
        logger.exception("Input file not found: %s", path)
        raise

    except OSError:
        logger.exception("File operation failed: %s", path)
        raise

process_file("orders.txt")
```

This example records:

1. The start of processing.
2. The number of lines read.
3. Successful completion.
4. Details of file-related failures.

It also re-raises failures instead of reporting a successful outcome after a failed read.

**Production consideration:** `readlines()` loads the entire file into memory. For a large dataset, iterate over lines or use an appropriate streaming or distributed processing tool. Also, a line count is not necessarily the same as a count of valid business records.

---

## 10. Logging to Multiple Destinations

Python's logging system supports multiple handlers.

For example, a program can send logs to both the console and a file.

```python
import logging

logger = logging.getLogger("pipeline")
logger.setLevel(logging.INFO)

formatter = logging.Formatter(
    "%(asctime)s - %(levelname)s - %(message)s"
)

console_handler = logging.StreamHandler()
console_handler.setFormatter(formatter)

file_handler = logging.FileHandler(
    "pipeline.log",
    encoding="utf-8"
)
file_handler.setFormatter(formatter)

logger.addHandler(console_handler)
logger.addHandler(file_handler)

logger.info("Pipeline started")
```

The message is sent to both configured handlers.

### What Is a Handler?

A handler determines where log records are sent.

Common examples:

- `StreamHandler`: Console or another stream.
- `FileHandler`: A file.
- `RotatingFileHandler`: A file that rotates based on size.
- `TimedRotatingFileHandler`: A file that rotates based on time.

**Important:** Avoid adding duplicate handlers every time a reusable setup function runs. Doing so can cause each message to appear multiple times.

---

## 11. Log Rotation

A long-running application can generate very large log files.

Log rotation helps control disk usage by moving old logs aside and starting a new file.

### Rotate by File Size

```python
import logging
from logging.handlers import RotatingFileHandler

logger = logging.getLogger("pipeline")
logger.setLevel(logging.INFO)

handler = RotatingFileHandler(
    "pipeline.log",
    maxBytes=5_000_000,
    backupCount=3,
    encoding="utf-8"
)

handler.setFormatter(
    logging.Formatter(
        "%(asctime)s - %(levelname)s - %(message)s"
    )
)

logger.addHandler(handler)

logger.info("Pipeline started")
```

Here:

- `maxBytes=5_000_000` sets an approximate 5 MB rotation threshold.
- `backupCount=3` retains up to three backup files.

Rotation behavior depends on the handler and when records are emitted.

### Rotate by Time

`TimedRotatingFileHandler` can rotate logs at configured time intervals, such as daily or at midnight.

In production environments, log rotation may instead be managed by the operating system, a container platform, or a centralized logging service. Avoid running multiple independent rotation mechanisms against the same file.

---

## 12. Logging Best Practices

### 1. Use Appropriate Severity Levels

Do not use `ERROR` for every unusual event. Use `WARNING` for recoverable problems and `INFO` for normal milestones.

### 2. Include Useful Context

Prefer:

```python
logger.error(
    "Failed to load batch: batch_id=%s destination=%s",
    batch_id,
    destination
)
```

over:

```python
logger.error("Something went wrong")
```

Useful context helps identify the affected batch or operation.

### 3. Avoid Sensitive Information

Do not log passwords, access tokens, secret keys, or unnecessary personal data.

### 4. Avoid Excessive Logging

Logging every row in a huge dataset can generate excessive output and slow processing. Prefer batch-level summaries, with detailed diagnostics enabled only when needed.

### 5. Do Not Hide Failures

Record failures and ensure that essential errors propagate or are handled explicitly.

### 6. Use Structured Logging Where Appropriate

For distributed pipelines, logs containing consistent fields—such as `job_id`, `batch_id`, `source`, and `rows_processed`—are easier to search and aggregate. JSON-formatted logging can be useful when supported by the logging platform.

### 7. Separate Configuration from Application Logic

Reusable modules should generally create named loggers. The application entry point or deployment configuration should define the handlers, formatting, and log levels.

---

## 13. Logging vs. `print()`

| Feature | `print()` | `logging` |
|---|---|---|
| Basic output | Yes | Yes |
| Severity levels | No built-in levels | Yes |
| Timestamp formatting | Manual | Configurable |
| File output | Requires additional code | Built-in handlers |
| Exception traceback | Manual | Supported |
| Filtering by severity | No built-in filtering | Yes |
| Production monitoring | Limited | Better suited |

`print()` is appropriate for simple scripts, quick experiments, and intentionally displayed output. Logging is usually more appropriate for production services and data pipelines.

---

## 14. Common Mistakes

1. **Calling `basicConfig()` repeatedly and expecting reconfiguration:** Existing root handlers can prevent it from taking effect.
2. **Using `logger.exception()` outside an exception handler:** It may not provide the exception details you intended.
3. **Logging an error but swallowing the exception:** The pipeline may appear successful despite failed work.
4. **Logging every record at `INFO`:** This can generate excessive logs.
5. **Including secrets in log messages:** Logs may be retained or accessed by more people than the original application data.
6. **Creating duplicate handlers:** Messages may be emitted multiple times.
7. **Using unclear messages:** A useful log should identify what happened and, when possible, which batch or resource was involved.
8. **Ignoring log retention:** Log files can consume disk space without rotation or retention controls.
9. **Confusing logging with monitoring:** Logs provide event details; metrics and monitoring systems help measure trends and alert on conditions.

---

## 15. Interview Questions

### Q1. What is logging in Python?

Logging records events, errors, and diagnostic information during program execution.

### Q2. What are the five standard logging levels?

`DEBUG`, `INFO`, `WARNING`, `ERROR`, and `CRITICAL`.

### Q3. What is the difference between `print()` and logging?

Logging supports severity levels, configurable formats, handlers, timestamps, and exception tracebacks. `print()` is simpler and primarily writes output to a stream.

### Q4. What does `logging.basicConfig()` do?

It provides basic configuration for the root logger, including its minimum level, output format, and optional file destination.

### Q5. What is the purpose of `logging.getLogger(__name__)`?

It creates or retrieves a named logger associated with the current module, helping identify where messages originate.

### Q6. What is the difference between `logger.error()` and `logger.exception()`?

`logger.error()` logs an error message. `logger.exception()` also includes the active exception traceback and is intended for use inside an exception handler.

### Q7. What is a logging handler?

A handler sends log records to a destination, such as the console or a file.

### Q8. What is log rotation?

Log rotation limits the growth of log files by starting new files and retaining a configured number of older files.

### Q9. How would you use logging in an ETL pipeline?

Record job start and completion, input files, batch identifiers, row counts, durations, validation failures, and exceptions. Use appropriate levels and ensure failed processing is not reported as successful.

### Q10. Why should you avoid logging sensitive data?

Logs may be retained, indexed, or exposed to a broader audience than the original data. Secrets and unnecessary personal information should be excluded or appropriately redacted.

### Q11. What is structured logging?

Structured logging represents events as consistent fields, often JSON, so systems can search and aggregate values such as job IDs, batch IDs, and error types.

### Q12. What is the difference between logging and monitoring?

Logging records individual events and diagnostic details. Monitoring evaluates metrics, trends, and system conditions, often generating alerts when thresholds or failure conditions are met.

---

## 16. Practice Tasks

Try solving these without copying the examples above.

1. Configure logging to display only `INFO` and higher messages.
2. Add timestamps and severity levels to each message.
3. Write logs to a file instead of the console.
4. Create a named logger using `logging.getLogger(__name__)`.
5. Log an exception with its traceback.
6. Log the input filename and number of processed records.
7. Configure both console and file handlers.
8. Rotate a log file after it reaches a size limit.
9. Explain why logging every record in a million-row dataset can be problematic.
10. Design a logging strategy for an ETL job that extracts, transforms, and loads data.

---

## 17. Quick Revision

- **Logging:** Records application events and diagnostic information.
- **`DEBUG`:** Detailed diagnostic information.
- **`INFO`:** Normal progress.
- **`WARNING`:** Unexpected but potentially recoverable condition.
- **`ERROR`:** An operation failed.
- **`CRITICAL`:** Severe failure.
- **`basicConfig()`:** Basic root-logger configuration.
- **`getLogger(__name__)`:** Creates or retrieves a module-specific logger.
- **`logger.exception()`:** Logs an error with exception traceback.
- **Handler:** Sends log records to a destination.
- **Formatter:** Controls how log records appear.
- **Log rotation:** Controls log file growth and retention.
- **Structured logging:** Records events using consistent fields.
- **ETL logging:** Track job status, batches, counts, failures, and processing duration.

**Most important interview takeaway:** Understand logging levels, named loggers, handlers, formatting, exception tracebacks, log rotation, and how to design useful, secure logs for production data pipelines.