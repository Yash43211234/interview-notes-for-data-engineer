# Error Handling in Data Pipelines

## 1. What Is Error Handling?

**Error handling** is the process of detecting, managing, and recovering from failures so that an application can fail safely or continue processing when appropriate.

In Data Engineering, pipelines can fail because of:
- Database connection failures.
- Missing or corrupted input files.
- Unexpected data types or schema changes.
- Invalid or duplicate records.
- Network timeouts.
- API rate limits.
- Insufficient permissions.
- Out-of-memory errors.
- Failures while writing output data.

A reliable pipeline should not only detect errors but also prevent incorrect data from reaching downstream systems.

## 2. Why Error Handling Matters

Imagine a pipeline that reads orders, transforms them, and loads them into a destination table.

If the pipeline fails halfway through, several problems can occur:

- Some records may be loaded while others are missing.
- A retry may insert duplicate records.
- Invalid data may reach reporting systems.
- The failure may go unnoticed.
- Operators may not know where the pipeline stopped.

Good error handling helps ensure:

1. Failures are detected and reported.
2. Resources are released properly.
3. Invalid data is isolated when appropriate.
4. Partial writes are handled safely.
5. Retries do not create duplicate results.
6. Logs contain enough information to troubleshoot failures.
7. Pipeline status accurately reflects the outcome.

## 3. Python Exceptions

Python uses exceptions to represent errors that occur during execution.

Common exceptions include:

| Exception | Typical cause |
|---|---|
| `FileNotFoundError` | Required file does not exist |
| `PermissionError` | Insufficient access permissions |
| `ValueError` | A value has an invalid format |
| `TypeError` | An operation receives an incompatible type |
| `KeyError` | A dictionary key is missing |
| `ZeroDivisionError` | Division by zero |
| `TimeoutError` | An operation exceeds a time limit |
| `ConnectionError` | A connection-related failure |
| `MemoryError` | An operation cannot allocate sufficient memory |

Some database and cloud libraries define their own exception classes. Handle the specific exceptions documented by the library whenever possible.

## 4. Using `try`, `except`, `else`, and `finally`

Python provides four important constructs for managing exceptions.

### `try`

Contains the code that might raise an exception.

### `except`

Handles a matching exception.

### `else`

Runs only when the `try` block completes without raising an exception.

### `finally`

Runs whether an exception occurred or not, making it useful for cleanup.

```python
try:
    result = 100 / 5

except ZeroDivisionError:
    print("Cannot divide by zero")

else:
    print("Result:", result)

finally:
    print("Operation completed")
```

Output:

```text
Result: 20.0
Operation completed
```

### Handling a file error

```python
try:
    with open("orders.csv", "r", encoding="utf-8") as file:
        data = file.read()

except FileNotFoundError:
    print("Required input file was not found")

else:
    print("File read successfully")
```

**Important:** Do not use `except Exception: pass` in a production pipeline. It silently hides failures and can make a failed job appear successful.

## 5. Catch Specific Exceptions

Prefer catching the errors you know how to handle.

Poor approach:

```python
try:
    amount = int("invalid")

except Exception:
    amount = 0
```

This can silently turn invalid data into a legitimate-looking value.

Better approach:

```python
try:
    amount = int("invalid")

except ValueError as error:
    print(f"Invalid amount: {error}")
    raise
```

The exception is logged or reported and then re-raised so that the pipeline does not continue as if the operation succeeded.

Catch broader exceptions only at appropriate boundaries, such as the top-level job runner, where you can record failure status and perform cleanup before propagating the error.

## 6. Raising Exceptions

Use `raise` when a condition makes continuing unsafe.

```python
def validate_batch_size(batch_size: int) -> None:
    if batch_size <= 0:
        raise ValueError("Batch size must be greater than zero")
```

Usage:

```python
validate_batch_size(0)
```

This raises a `ValueError`.

### Re-raising an exception

```python
try:
    process_data()

except ValueError:
    print("Data processing failed")
    raise
```

The bare `raise` preserves the current exception and its traceback.

### Custom exceptions

Custom exceptions make pipeline failures easier to classify.

```python
class PipelineError(Exception):
    """Base exception for pipeline failures."""


class DataValidationError(PipelineError):
    """Raised when input data fails validation."""


class DataLoadError(PipelineError):
    """Raised when loading data fails."""
```

Usage:

```python
def validate_record(record):
    if record.get("order_id") is None:
        raise DataValidationError("Missing order_id")
```

Custom exceptions are especially useful when different failures need different recovery actions.

## 7. Error Handling Across Pipeline Stages

A typical data pipeline contains multiple stages.

<box border radius="lg" padding={3} gap={2}>
  <box background="surface-secondary" radius="md" padding={3} align="center">
    **1. Extract**

    <text size="sm" color="secondary">Files, databases, APIs, cloud storage</text>
  </box>
  <row justify="center">
    <icon name="arrow-down" size="lg" />
  </row>
  <box background="surface-secondary" radius="md" padding={3} align="center">
    **2. Validate**

    <text size="sm" color="secondary">Schema, types, required fields, quality rules</text>
  </box>
  <row justify="center">
    <icon name="arrow-down" size="lg" />
  </row>
  <box background="surface-secondary" radius="md" padding={3} align="center">
    **3. Transform**

    <text size="sm" color="secondary">Cleaning, joins, calculations, business rules</text>
  </box>
  <row justify="center">
    <icon name="arrow-down" size="lg" />
  </row>
  <box background="surface-secondary" radius="md" padding={3} align="center">
    **4. Load**

    <text size="sm" color="secondary">Write output and commit safely</text>
  </box>
  <row justify="center">
    <icon name="arrow-down" size="lg" />
  </row>
  <box background="surface" border radius="md" padding={3} align="center">
    **5. Record outcome**

    <text size="sm" color="secondary">Metrics, logs, alerts, success or failure status</text>
  </box>
</box>

Each stage should have a clear failure policy.

| Stage | Example failure | Possible response |
|---|---|---|
| Extract | Database timeout | Retry if transient |
| Validate | Missing required column | Fail the batch or quarantine the input |
| Transform | Invalid business value | Reject or isolate the record |
| Load | Destination unavailable | Retry safely |
| Finalization | Metrics or status update fails | Alert and reconcile job state |

The correct response depends on whether the error is temporary, permanent, record-specific, or batch-wide.

## 8. Retry Logic

A retry means attempting an operation again after a failure.

Retries are useful for **transient errors**, such as:
- Temporary network interruptions.
- HTTP 429 rate limiting.
- Some HTTP 5xx responses.
- Temporary database unavailability.

Retries are generally not useful for permanent errors, such as:
- Invalid credentials.
- Missing required columns.
- Incorrect SQL syntax.
- Invalid business rules.

### Exponential backoff

Exponential backoff increases the delay between retry attempts.

For example:

- Attempt 1 fails.
- Wait approximately 1 second.
- Attempt 2 fails.
- Wait approximately 2 seconds.
- Attempt 3 fails.
- Wait approximately 4 seconds.

Add jitter (a small random variation) in distributed systems to reduce the chance that many workers retry simultaneously.

### Example retry function

```python
import random
import time
from collections.abc import Callable
from typing import TypeVar

T = TypeVar("T")


def retry_operation(
    operation: Callable[[], T],
    max_attempts: int = 3,
    base_delay: float = 1.0,
) -> T:
    if max_attempts < 1:
        raise ValueError("max_attempts must be at least 1")

    for attempt in range(max_attempts):
        try:
            return operation()

        except (TimeoutError, ConnectionError):
            if attempt == max_attempts - 1:
                raise

            delay = base_delay * (2 ** attempt)
            delay += random.uniform(0, 0.5)
            time.sleep(delay)

    raise RuntimeError("Unreachable")
```

Example:

```python
def fetch_data():
    # Replace with a real operation.
    return "Data received"


result = retry_operation(fetch_data)
print(result)
```

This example retries only the explicitly listed exception types. In real systems, use the database or API client's documented transient-error classification, and respect server-provided retry instructions such as `Retry-After` when applicable.

**Important:** Retrying a write operation can create duplicate data if the first attempt succeeded but its response was lost. Use idempotent operations or appropriate transaction and deduplication strategies.

## 9. Idempotency and Safe Retries

An operation is **idempotent** if repeating it produces the same intended final state as performing it once.

For data pipelines, idempotency is essential because jobs may be rerun after failures.

Example:

A pipeline loads orders for a particular processing date.

Unsafe approach:

```sql
INSERT INTO target_orders
SELECT *
FROM staging_orders;
```

If rerun without duplicate protection, this may insert the same records again.

Safer designs may use:
- Unique business keys.
- Database constraints.
- Upserts or `MERGE` operations.
- Replacement of a clearly defined partition.
- A staging table followed by a controlled transactional load.
- A load identifier or batch identifier.

The correct strategy depends on the database and target table design.

**Interview tip:** Retries improve resilience, but idempotency helps make retries safe.

## 10. Data Validation and Bad Records

Not every error requires stopping the entire pipeline.

Suppose an orders file contains these records:

| order_id | amount |
|---|---:|
| 101 | 250 |
| 102 | invalid |
| 103 | 500 |

The second record has an invalid amount.

Possible policies include:

1. **Fail the entire batch:** appropriate when every record must be valid.
2. **Reject the invalid record:** appropriate when individual records can be processed independently.
3. **Quarantine bad records:** store invalid records and error reasons for investigation.
4. **Apply a documented correction:** only when the correction is valid under business rules.

Example:

```python
def validate_order(record):
    errors = []

    if not record.get("order_id"):
        errors.append("Missing order_id")

    try:
        amount = float(record["amount"])

        if amount < 0:
            errors.append("Amount cannot be negative")

    except (KeyError, TypeError, ValueError):
        errors.append("Invalid amount")

    return errors
```

Usage:

```python
orders = [
    {"order_id": 101, "amount": "250"},
    {"order_id": 102, "amount": "invalid"},
    {"order_id": 103, "amount": "500"},
]

valid_records = []
bad_records = []

for order in orders:
    errors = validate_order(order)

    if errors:
        bad_records.append({
            "record": order,
            "errors": errors,
        })
    else:
        valid_records.append(order)
```

In production, also consider type requirements, currency precision, missing values, duplicate keys, and whether values are finite. A conversion to `float` alone does not enforce all financial or business constraints.

## 11. Logging Pipeline Errors

Logging records what happened during execution and helps engineers troubleshoot failures.

Python provides the built-in `logging` module.

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(name)s %(message)s",
)

logger = logging.getLogger(__name__)

logger.info("Pipeline started")
logger.warning("Some records failed validation")
logger.error("Pipeline failed")
```

### Log an exception with traceback

```python
try:
    process_data()

except Exception:
    logger.exception("Data processing failed")
    raise
```

`logger.exception()` includes the traceback and should be called inside an active exception handler.

### What should a pipeline log?

Useful fields include:
- Pipeline and task name.
- Run or batch ID.
- Source and destination.
- Processing date.
- Number of records read, accepted, rejected, and written.
- Retry attempt.
- Error category and traceback.
- Start time, completion time, and duration.

Avoid logging passwords, access tokens, sensitive personal data, or full records that contain confidential information.

## 12. Resource Cleanup

Resources such as files, database connections, and cursors should be released even when an operation fails.

Use context managers where supported.

### File example

```python
with open("orders.csv", "r", encoding="utf-8") as file:
    data = file.read()
```

The file is closed when the block exits, including when an exception occurs.

### Database example

```python
import sqlite3

connection = sqlite3.connect("orders.db")

try:
    with connection:
        cursor = connection.cursor()
        cursor.execute("SELECT 1")
        print(cursor.fetchone())

finally:
    connection.close()
```

The SQLite connection context manager handles transaction commit or rollback; the explicit `finally` block closes the connection.

**Important:** Context-manager behavior differs between libraries. Confirm whether a database connection's context manager closes the connection, manages transactions, or both.

## 13. Transactional Loading and Partial Failures

A pipeline can fail after writing some records but before completing the batch.

To avoid inconsistent results, consider:
- Loading into a staging table first.
- Validating the staged data.
- Applying changes in a transaction when supported.
- Using atomic table or partition replacement where available.
- Tracking committed batches.
- Designing restart logic that can determine what has already succeeded.

Example with SQLite:

```python
import sqlite3

connection = sqlite3.connect("orders.db")

try:
    with connection:
        connection.execute(
            """
            INSERT INTO orders (order_id, amount)
            VALUES (?, ?)
            """,
            (101, 250),
        )

        connection.execute(
            """
            INSERT INTO orders (order_id, amount)
            VALUES (?, ?)
            """,
            (102, 500),
        )

except sqlite3.Error:
    # The transaction context rolls back on an exception.
    raise

finally:
    connection.close()
```

The transaction groups both inserts so they can be committed together or rolled back together under normal transactional behavior.

For large distributed data platforms, transaction boundaries may differ. Do not assume a multi-step pipeline is automatically atomic simply because individual writes are transactional.

## 14. End-to-End Error Handling Example

The following example demonstrates a small file-based pipeline with validation, logging, quarantine, and failure propagation.

```python
import csv
import logging
from pathlib import Path

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(message)s",
)

logger = logging.getLogger(__name__)


def process_orders(input_path, output_path, bad_path):
    input_file = Path(input_path)
    output_file = Path(output_path)
    bad_file = Path(bad_path)

    if not input_file.is_file():
        raise FileNotFoundError(f"Input file not found: {input_file}")

    valid_count = 0
    rejected_count = 0

    try:
        with (
            input_file.open("r", encoding="utf-8", newline="") as source,
            output_file.open("w", encoding="utf-8", newline="") as output,
            bad_file.open("w", encoding="utf-8", newline="") as bad,
        ):
            reader = csv.DictReader(source)

            required_columns = {"order_id", "amount"}

            if not required_columns.issubset(reader.fieldnames or []):
                raise ValueError("Required input columns are missing")

            writer = csv.DictWriter(
                output,
                fieldnames=["order_id", "amount"],
            )
            bad_writer = csv.DictWriter(
                bad,
                fieldnames=["order_id", "amount", "error"],
            )

            writer.writeheader()
            bad_writer.writeheader()

            for row in reader:
                try:
                    order_id = row["order_id"].strip()
                    amount = float(row["amount"])

                    if not order_id:
                        raise ValueError("Missing order_id")

                    if amount < 0:
                        raise ValueError("Negative amount")

                    writer.writerow({
                        "order_id": order_id,
                        "amount": amount,
                    })
                    valid_count += 1

                except (ValueError, TypeError) as error:
                    bad_writer.writerow({
                        "order_id": row.get("order_id", ""),
                        "amount": row.get("amount", ""),
                        "error": str(error),
                    })
                    rejected_count += 1

        logger.info(
            "Pipeline completed: valid=%d rejected=%d",
            valid_count,
            rejected_count,
        )

    except Exception:
        logger.exception("Pipeline failed")
        raise


process_orders(
    "orders.csv",
    "processed_orders.csv",
    "rejected_orders.csv",
)
```

### What this example demonstrates

- Missing input files cause a clear failure.
- Required columns are checked before processing.
- Invalid records are written to a separate file.
- Valid and rejected record counts are logged.
- Unexpected failures are logged and re-raised.
- Files are managed with context managers.

### Limitations to understand

This is a learning example, not a fully production-safe implementation.

- The output files are opened in write mode, so existing files can be overwritten.
- If processing fails midway, partial output files may remain.
- The two output files are not written atomically as a single unit.
- `float()` accepts some values that may be unsuitable for financial data, such as non-finite numbers.
- A full production pipeline should use temporary outputs, validate results, and publish them only after successful completion.

These limitations illustrate why error handling must be combined with idempotency, data-quality rules, and safe output publication.

## 15. Error Handling in Spark and Databricks

In PySpark pipelines, failures can happen at different levels:
- Reading source data.
- Parsing records.
- Applying transformations.
- Executing Spark actions.
- Writing to Delta tables or external storage.
- Running out of executor memory or losing executors.

Spark transformations are often lazy: many errors occur when an action such as `count()`, `collect()`, or `write` triggers execution.

Example:

```python
try:
    df = spark.read.parquet(input_path)

    transformed_df = df.filter("amount > 0")

    transformed_df.write.mode("append").format("delta").save(output_path)

except Exception:
    logger.exception("Spark pipeline failed")
    raise
```

This illustrates logging and propagation, but broad exception handling alone does not make a Spark pipeline reliable.

Additional considerations:
- Validate schemas before writing.
- Avoid `collect()` for large datasets.
- Use bounded retries only for failures known to be transient.
- Design append and overwrite operations carefully.
- Use Delta Lake transaction guarantees appropriately.
- Track job run IDs and source batch IDs.
- Reconcile data after uncertain write outcomes.
- Distinguish record-level data-quality failures from infrastructure failures.

Spark and Delta Lake provide their own execution and transaction behavior. A multi-stage pipeline is not automatically one transaction across all its tables and storage locations.

## 16. Common Mistakes

1. **Catching every exception and continuing.** This can hide serious failures.
2. **Retrying permanent errors.** Invalid schemas and credentials usually require correction, not repeated attempts.
3. **Retrying writes without idempotency.** This can create duplicates.
4. **Logging only the exception message.** A traceback often provides necessary debugging context.
5. **Not re-raising an unexpected exception.** The orchestrator may incorrectly mark the job as successful.
6. **Ignoring partial output.** Failed jobs can leave incomplete files or partially committed results.
7. **Loading all records into memory.** Use batching or distributed processing when appropriate.
8. **Failing to validate input schemas.** Unexpected source changes may silently affect results.
9. **Logging sensitive information.** Logs must follow security and privacy policies.
10. **Using infinite retries.** Set attempt limits, backoff, and a clear terminal failure state.
11. **Treating every bad record as a pipeline-wide failure.** Define a documented reject or quarantine policy where appropriate.
12. **Assuming a successful write means the entire workflow succeeded.** Validate downstream effects and final status.

## 17. Interview Questions

### Basic

1. What is error handling in Python?
2. What is the difference between `try`, `except`, `else`, and `finally`?
3. What is the purpose of `raise`?
4. Why should you catch specific exceptions?
5. What is a custom exception?
6. Why is logging important in a data pipeline?
7. What is the difference between an error and a rejected data record?

### Intermediate

1. What is exponential backoff?
2. Which failures should be retried?
3. What is idempotency, and why does it matter?
4. How would you handle a missing input file?
5. How would you handle a schema mismatch?
6. How do you prevent duplicate records when retrying a load?
7. How would you handle a database failure during a batch insert?
8. What is the purpose of a quarantine or dead-letter location?
9. Why should unexpected exceptions normally be re-raised?
10. How would you handle resources when an exception occurs?

### Data Engineering scenarios

1. A pipeline fails after loading half its records. How would you recover safely?
2. A source API returns HTTP 429. What retry strategy would you use?
3. A source schema changes unexpectedly. Should the pipeline continue?
4. A retry succeeds but duplicates records in the destination. How would you prevent this?
5. A Spark job fails during a Delta write. What would you inspect before rerunning it?
6. Some records are invalid, but the rest are valid. How would you design the pipeline?
7. The pipeline logs show an error, but the scheduler marks the run successful. What might be wrong?
8. How would you make failures observable to the team responsible for production support?

## 18. Quick Revision

| Concept | Key point |
|---|---|
| `try/except` | Detect and handle expected failures |
| `else` | Runs when the `try` block succeeds |
| `finally` | Used for cleanup |
| `raise` | Signals an error or re-raises an exception |
| Custom exceptions | Classify application-specific failures |
| Retry | Reattempt a failed operation |
| Exponential backoff | Increase delay between retries |
| Idempotency | Repeated execution produces the same intended result |
| Quarantine | Isolate invalid records for investigation |
| Logging | Record execution details and failure context |
| Transaction | Commit or roll back a group of database changes |
| Safe restart | Resume or rerun without corrupting results |
| Schema validation | Detect unexpected input structure |
| Alerting | Notify operators when intervention is needed |

**Key takeaway:** Reliable data pipelines distinguish transient failures from permanent errors, validate data, log useful context, release resources, and recover safely. Retry logic is valuable, but it must be combined with idempotent writes and a clear strategy for handling partial failures.