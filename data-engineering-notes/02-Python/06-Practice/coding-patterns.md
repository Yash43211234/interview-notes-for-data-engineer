# Coding Patterns in Python

## 1. What Are Coding Patterns?

Coding patterns are reusable approaches to solving common programming problems. They help you organize code, reduce duplication, and make solutions easier to understand, test, and maintain.

In data engineering, coding patterns are useful for:
- Processing records consistently.
- Validating and transforming data.
- Handling errors and retries.
- Reading paginated API responses.
- Avoiding duplicated logic in ETL pipelines.
- Writing reliable, maintainable Python code.

This note focuses on practical Python patterns rather than advanced object-oriented design patterns.

---

## 2. Guard Clauses

A **guard clause** checks an invalid or exceptional condition early and exits the function before continuing with the main logic.

### Without guard clauses

```python
def calculate_discount(price, discount_rate):
    if price >= 0:
        if 0 <= discount_rate <= 1:
            return price * (1 - discount_rate)

    raise ValueError("Invalid price or discount rate")
```

### With guard clauses

```python
def calculate_discount(price, discount_rate):
    if price < 0:
        raise ValueError("Price cannot be negative")

    if not 0 <= discount_rate <= 1:
        raise ValueError("Invalid discount rate")

    return price * (1 - discount_rate)
```

**Why it helps:** The main logic stays at the normal indentation level, making the function easier to read.

Use guard clauses for invalid inputs, missing required values, and conditions where processing should stop immediately.

---

## 3. Early Returns

An early return stops a function as soon as its result is known.

```python
def get_customer_status(customer):
    if customer is None:
        return "UNKNOWN"

    if customer["is_active"]:
        return "ACTIVE"

    return "INACTIVE"
```

This avoids unnecessary nesting and makes each decision clear.

Early returns are particularly useful when validating records before transforming them.

---

## 4. Transform, Then Filter

This pattern separates data transformation from the decision to keep or reject a record.

For example, normalize order amounts before applying business rules.

```python
def normalize_order(order):
    return {
        **order,
        "amount": float(order["amount"]),
    }


def is_valid_order(order):
    return order["amount"] >= 0
```

Combine them:

```python
orders = [
    {"order_id": 1, "amount": "100.50"},
    {"order_id": 2, "amount": "-20"},
]

normalized = [normalize_order(order) for order in orders]
valid_orders = [
    order for order in normalized
    if is_valid_order(order)
]
```

**Why it helps:** Transformation rules and filtering rules can be tested independently.

Be aware that normalization can raise errors for missing or malformed amounts. Define how those records should be handled in production.

---

## 5. Separate Pure Logic from Side Effects

Pure logic calculates a result from inputs. Side effects interact with external systems, such as files, APIs, and databases.

A maintainable application separates these responsibilities where practical.

### Less maintainable approach

```python
def process_orders():
    import csv

    with open("orders.csv", newline="") as file:
        orders = list(csv.DictReader(file))

    total = sum(float(order["amount"]) for order in orders)

    print(total)
```

The function reads a file, calculates a value, and prints it.

### Better approach

```python
def calculate_total(orders):
    return sum(float(order["amount"]) for order in orders)
```

File reading can be handled separately:

```python
import csv


def read_orders(file_path):
    with open(file_path, newline="") as file:
        return list(csv.DictReader(file))
```

Application orchestration:

```python
orders = read_orders("orders.csv")
total = calculate_total(orders)
print(total)
```

**Key benefit:** The calculation can be tested using a small list of records without needing a file.

---

## 6. DRY: Don't Repeat Yourself

The DRY principle means avoiding duplicated knowledge or business rules.

### Repeated logic

```python
def calculate_order_total(orders):
    return sum(float(order["amount"]) for order in orders)


def calculate_invoice_total(invoices):
    return sum(float(invoice["amount"]) for invoice in invoices)
```

If both calculations have exactly the same rules, they can share a function:

```python
def calculate_total(records):
    return sum(float(record["amount"]) for record in records)
```

Usage:

```python
order_total = calculate_total(orders)
invoice_total = calculate_total(invoices)
```

However, do not combine functions merely because they look similar. If orders and invoices follow different business rules, separate functions may be clearer.

**Remember:** DRY means avoiding duplicated knowledge, not eliminating every repeated line of code.

---

## 7. Single Responsibility Principle

The Single Responsibility Principle (SRP) says a component should have one coherent responsibility.

For example, an ETL pipeline can be separated into:

```python
def extract_data(source):
    ...


def transform_data(records):
    ...


def load_data(records, destination):
    ...
```

An orchestration function coordinates the stages:

```python
def run_pipeline(source, destination):
    records = extract_data(source)
    transformed = transform_data(records)
    load_data(transformed, destination)
```

Each function has a distinct purpose.

This makes it easier to:
- Test transformations without accessing storage.
- Change a data source without rewriting business logic.
- Diagnose failures by pipeline stage.
- Reuse transformations in different workflows.

---

## 8. Composition: Build Small Functions into a Workflow

Function composition means combining smaller operations to create a larger operation.

```python
def normalize_amount(value):
    return float(value)


def is_valid_amount(amount):
    return amount >= 0


def process_amount(value):
    amount = normalize_amount(value)

    if not is_valid_amount(amount):
        return None

    return amount
```

Each function does one small task.

Composition is useful when transformations need to be reused in multiple workflows. Keep the order of operations explicit when it affects the result.

---

## 9. Strategy Pattern

The Strategy Pattern allows different algorithms to be selected through a common interface.

In Python, simple functions can often serve as strategies without requiring a class hierarchy.

```python
def percentage_discount(price):
    return price * 0.10


def fixed_discount(price):
    return 50


def apply_discount(price, discount_strategy):
    discount = discount_strategy(price)
    return max(0, price - discount)
```

Usage:

```python
print(apply_discount(500, percentage_discount))
print(apply_discount(500, fixed_discount))
```

The `apply_discount` function does not need to know how each discount is calculated.

In data engineering, this pattern can support different validation rules, file formats, or transformation methods without filling one function with many conditional branches.

Use it when algorithms genuinely vary. A simple `if` statement is often enough for a small, stable decision.

---

## 10. Configuration Instead of Hard-Coding

Configuration separates environment-specific values from program logic.

### Hard-coded approach

```python
def load_data():
    database = "production_db"
    table = "orders"
    ...
```

This makes it difficult to run the same code in development and production.

### Better approach

```python
def load_data(database, table):
    ...
```

Usage:

```python
load_data(
    database="analytics_db",
    table="orders",
)
```

For larger applications, configuration may come from environment variables or configuration files.

Never hard-code passwords, API keys, or other secrets into source code.

---

## 11. Context Managers for Resource Safety

A context manager handles setup and cleanup around a resource.

The `with` statement is commonly used for files, locks, and database transactions.

```python
with open("orders.csv", encoding="utf-8") as file:
    contents = file.read()
```

Python closes the file when the block exits, including when an exception occurs.

Without a context manager, code must carefully ensure that resources are released in every execution path.

This pattern helps prevent resource leaks in long-running data pipelines.

---

## 12. Retry Pattern for Temporary Failures

Some operations fail temporarily because of network interruptions, rate limits, or service availability.

A retry pattern repeats an operation after a failure, usually with a limited number of attempts and a delay.

A basic example:

```python
import time


def run_with_retries(operation, max_attempts=3):
    for attempt in range(max_attempts):
        try:
            return operation()
        except TimeoutError:
            if attempt == max_attempts - 1:
                raise

            time.sleep(2 ** attempt)
```

The waits are 1 second and then 2 seconds before the final attempt.

Important considerations:
- Retry only errors that may be temporary.
- Limit the number of attempts.
- Use exponential backoff for appropriate network failures.
- Respect service rate limits.
- Avoid blindly retrying non-idempotent operations, such as a payment or insert that could happen twice.

In production, add logging and consider jitter to reduce simultaneous retry spikes.

---

## 13. Idempotency Pattern

An operation is **idempotent** if repeating it with the same intended input does not change the final result beyond the first successful application.

For example, setting a customer's status to `"ACTIVE"` is naturally idempotent if the operation simply overwrites the same status.

By contrast, adding 100 to an account balance is not idempotent: repeating it adds another 100.

### Data engineering example

A pipeline may receive the same source record more than once. To avoid duplicate output, it can use a stable record key and an appropriate deduplication or upsert strategy.

Conceptually:

```python
def deduplicate_records(records):
    unique = {}

    for record in records:
        unique[record["order_id"]] = record

    return list(unique.values())
```

This keeps one record per `order_id`, but when duplicates differ, the last record wins. A real pipeline should explicitly define which record is authoritative, such as the record with the latest update timestamp.

Idempotency is important when pipelines are rerun after failures.

---

## 14. Batch Processing

Batch processing groups records and processes them together rather than handling every record as a separate operation.

```python
def batches(records, batch_size):
    for start in range(0, len(records), batch_size):
        yield records[start:start + batch_size]
```

Usage:

```python
records = list(range(10))

for batch in batches(records, 3):
    print(batch)
```

Output:

```text
[0, 1, 2]
[3, 4, 5]
[6, 7, 8]
[9]
```

Batching is useful for:
- Database inserts.
- API requests with a maximum batch size.
- Processing large input collections.
- Controlling memory use when combined with streaming or generators.

This particular function receives a list-like sequence. For very large sources, avoid loading the entire dataset into memory just to create batches; build a batching approach around an iterator instead.

---

## 15. Pipeline Pattern

A pipeline passes data through a sequence of processing stages.

```python
def extract():
    return [
        {"id": 1, "amount": "100"},
        {"id": 2, "amount": "200"},
    ]


def transform(records):
    return [
        {
            "id": record["id"],
            "amount": float(record["amount"]),
        }
        for record in records
    ]


def validate(records):
    return [
        record for record in records
        if record["amount"] >= 0
    ]


def run_pipeline():
    records = extract()
    records = transform(records)
    records = validate(records)
    return records
```

Each stage accepts the output of the previous stage.

This structure makes individual transformations easier to test and makes the overall flow easier to understand.

For production pipelines, also define what happens when a stage fails, how rejected records are handled, and how processing progress is recorded.

---

## 16. Common Mistakes

### Mistake 1: Making one giant function

**Problem:** Logic, file access, validation, and database writing become difficult to test independently.

**Better approach:** Break the workflow into cohesive functions.

### Mistake 2: Overusing abstractions

**Problem:** Too many classes and layers make simple logic harder to follow.

**Better approach:** Prefer straightforward functions until a real need for abstraction appears.

### Mistake 3: Retrying every exception

**Problem:** Invalid input or permanent failures are repeated unnecessarily.

**Better approach:** Retry only appropriate transient errors.

### Mistake 4: Assuming deduplication always means keeping the last record

**Problem:** Input order may not reflect the most recent or correct record.

**Better approach:** Define a business rule for selecting the authoritative record.

### Mistake 5: Applying DRY too aggressively

**Problem:** Unrelated business rules become coupled in one generic function.

**Better approach:** Share code when the underlying behavior and rules are genuinely the same.

### Mistake 6: Ignoring resource cleanup

**Problem:** Files, connections, or other resources may remain open after failures.

**Better approach:** Use context managers or explicit cleanup mechanisms.

---

## 17. Interview Questions and Answers

### Q1. What are coding patterns?

Coding patterns are reusable approaches to recurring programming problems. They provide a consistent way to structure logic and improve readability, maintainability, and reliability.

### Q2. What is the difference between DRY and SRP?

DRY focuses on avoiding duplicated knowledge or business rules. SRP focuses on giving a component one coherent responsibility. They complement each other but solve different design problems.

### Q3. What is a guard clause?

A guard clause checks a condition early and exits or raises an exception if the condition prevents normal processing. It reduces unnecessary nesting.

### Q4. Why separate business logic from input/output operations?

It makes the business logic easier to test without requiring real files, databases, or APIs. It also allows the same logic to be reused with different data sources.

### Q5. What is the Strategy Pattern?

It allows interchangeable algorithms to be supplied to a component. In Python, this can often be implemented using functions passed as arguments.

### Q6. What is idempotency in data pipelines?

Idempotency means rerunning an operation with the same intended input does not create additional unintended effects. It helps pipelines recover safely from failures and duplicate deliveries.

### Q7. What is the benefit of batch processing?

Batching processes multiple records together, which can reduce per-operation overhead and help control resource usage. The appropriate batch size depends on the system and workload.

### Q8. How do you handle temporary failures in an API pipeline?

Use bounded retries for transient errors, apply an appropriate backoff strategy, respect rate limits, log failures, and ensure repeated requests do not cause duplicate side effects.

---

## 18. Quick Revision

| Pattern or principle | Main purpose |
|---|---|
| Guard clauses | Handle invalid conditions early |
| Early returns | Keep control flow simple |
| Transform, then filter | Separate data preparation from selection |
| Pure logic and side effects | Make business logic easier to test |
| DRY | Avoid duplicated business rules |
| Single Responsibility Principle | Keep each component focused |
| Composition | Build workflows from smaller functions |
| Strategy Pattern | Swap algorithms without rewriting the caller |
| Configuration | Separate settings from application logic |
| Context managers | Ensure resource cleanup |
| Retry pattern | Recover from appropriate temporary failures |
| Idempotency | Make repeated processing safe |
| Batch processing | Process groups of records efficiently |
| Pipeline pattern | Organize sequential processing stages |

**Remember:** Coding patterns are tools, not rules to apply everywhere. Choose the simplest pattern that makes the code clearer, easier to test, and more reliable.