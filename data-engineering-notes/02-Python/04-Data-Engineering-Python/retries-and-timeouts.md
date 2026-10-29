# Retries and Timeouts in Python

## 1. Why Retries and Timeouts Matter

In Data Engineering, pipelines frequently communicate with external APIs, databases, cloud storage, and other services.

These services can fail temporarily because of:

- Network interruptions.
- Slow server responses.
- API rate limits.
- Temporary service outages.
- Database connection problems.
- Cloud service throttling.

Two important techniques help make pipelines more reliable:

- **Timeouts:** Limit how long an operation can wait.
- **Retries:** Attempt an operation again after a failure.

Example: If an API request fails because of a temporary network problem, a pipeline can retry it instead of immediately failing the entire job.

However, retries should be limited and applied only when appropriate.

## 2. What Is a Timeout?

A timeout specifies how long a program should wait for an operation before treating it as unsuccessful.

Without timeouts, a pipeline may wait too long for an unresponsive service, delaying other tasks.

### Example using `requests`

```python
import requests

url = "https://api.example.com/orders"

try:
    response = requests.get(url, timeout=10)
    response.raise_for_status()

    print(response.json())

except requests.exceptions.Timeout:
    print("The request timed out")

except requests.exceptions.RequestException as e:
    print(f"Request failed: {e}")
```

Here, `timeout=10` specifies a timeout of 10 seconds for connection establishment and read inactivity.

**Important:** In the `requests` library, this is not a strict ten-second limit for the entire operation. A server that keeps sending data can cause the request to take longer.

### Connection timeout vs. read timeout

You can configure these separately:

```python
response = requests.get(
    url,
    timeout=(5, 30)
)
```

| Timeout | Meaning |
|---|---|
| Connection timeout: `5` | Maximum wait for establishing a connection |
| Read timeout: `30` | Maximum wait between receiving data from the server |

This is useful when an API is slow to establish a connection but normally responds quickly once connected, or when responses take longer to arrive.

### Why should every API request have a timeout?

Without an explicit timeout, a request may wait much longer than your pipeline can tolerate.

Timeouts help protect:

- Pipeline execution time.
- Worker resources.
- Downstream processing.
- Scheduled job reliability.

A timeout does not guarantee that the server stopped processing the request. It only means your client stopped waiting according to its timeout behavior.

## 3. What Is a Retry?

A retry means attempting an operation again after a failure.

Consider this sequence:

1. A pipeline requests data from an API.
2. The API returns a temporary server error.
3. The pipeline waits briefly.
4. The pipeline sends the request again.
5. The request succeeds, and processing continues.

Retries are useful for **temporary failures**, not for every possible error.

Examples of potentially retryable failures:

- Temporary connection errors.
- Read timeouts.
- HTTP `429 Too Many Requests`.
- HTTP `500`, `502`, `503`, or `504`, depending on the API and operation.

Examples that usually require fixing the request or configuration:

- HTTP `400 Bad Request`.
- HTTP `401 Unauthorized`.
- HTTP `403 Forbidden`.
- HTTP `404 Not Found`.

Some APIs use particular status codes differently, so follow their documentation.

## 4. Implementing Retries with a Loop

The simplest retry strategy uses a loop and a maximum attempt count.

```python
import time
import requests


def fetch_data(url, max_attempts=3):
    for attempt in range(1, max_attempts + 1):
        try:
            response = requests.get(url, timeout=(5, 20))
            response.raise_for_status()

            return response.json()

        except requests.exceptions.RequestException as e:
            print(f"Attempt {attempt} failed: {e}")

            if attempt == max_attempts:
                raise

            time.sleep(2)

    raise RuntimeError("Request failed")
```

### How it works

- `max_attempts=3` permits at most three attempts in total.
- `timeout=(5, 20)` limits connection and read waiting.
- The exception handler catches request-related failures.
- `time.sleep(2)` waits two seconds before another attempt.
- `raise` on the final attempt propagates the last exception to the caller.

This example is suitable for understanding the concept. In production, retries should generally distinguish transient errors from permanent errors and avoid using the same delay for every retry.

## 5. What Is Exponential Backoff?

Exponential backoff increases the delay between retry attempts.

Instead of retrying every two seconds, the waiting period increases after each failure.

A common formula is:

\[
\text{delay} = \min(\text{base} \times 2^{\text{retry index}}, \text{maximum delay})
\]

For a base delay of one second and a maximum delay of eight seconds:

| Retry number | Delay |
|---|---:|
| First retry | 1 second |
| Second retry | 2 seconds |
| Third retry | 4 seconds |
| Fourth retry | 8 seconds |
| Later retries | 8 seconds |

This reduces pressure on an overloaded service.

### Python implementation

```python
import time
import requests


def fetch_data(url, max_attempts=5):
    for attempt in range(max_attempts):
        try:
            response = requests.get(url, timeout=(5, 20))
            response.raise_for_status()

            return response.json()

        except requests.exceptions.RequestException:
            if attempt == max_attempts - 1:
                raise

            delay = min(2 ** attempt, 30)
            print(f"Retrying in {delay} seconds...")
            time.sleep(delay)
```

This example retries any `RequestException`. A production implementation should refine the exception handling and status-code policy so permanent failures are not repeatedly retried.

## 6. What Is Jitter?

When many workers fail simultaneously, they may all retry at the same time. This can overload the service again.

**Jitter** introduces a random variation in the retry delay.

```python
import random
import time

base_delay = 2
maximum_delay = 30

for retry_index in range(5):
    delay = min(base_delay * (2 ** retry_index), maximum_delay)
    delay = random.uniform(0, delay)

    print(f"Waiting {delay:.2f} seconds")
    time.sleep(delay)
```

This is an example of *full jitter*: the actual delay is randomly selected between zero and the calculated backoff limit.

Jitter is especially useful in distributed systems, where many tasks may retry simultaneously.

## 7. Retry HTTP Requests with `urllib3`

The `requests` library uses `urllib3` internally. You can configure retries through an HTTP adapter.

```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry


retry_strategy = Retry(
    total=4,
    connect=3,
    read=2,
    status=3,
    backoff_factor=1,
    status_forcelist=[429, 500, 502, 503, 504],
    allowed_methods=frozenset(["GET", "HEAD"]),
    respect_retry_after_header=True
)

session = requests.Session()

adapter = HTTPAdapter(max_retries=retry_strategy)

session.mount("https://", adapter)
session.mount("http://", adapter)

response = session.get(
    "https://api.example.com/orders",
    timeout=(5, 20)
)

response.raise_for_status()
data = response.json()
```

### Understanding the configuration

| Setting | Purpose |
|---|---|
| `total` | Overall retry budget |
| `connect` | Retry budget for connection errors |
| `read` | Retry budget for read errors |
| `status` | Retry budget for selected HTTP status responses |
| `backoff_factor` | Controls exponential backoff delay |
| `status_forcelist` | HTTP statuses eligible for status-based retries |
| `allowed_methods` | HTTP methods eligible for these retries |
| `respect_retry_after_header` | Respects the server's `Retry-After` header where applicable |

The retry counts are not interchangeable. The exact behavior depends on the `urllib3` version and the type of failure.

**Important:** `Retry(total=4)` does not mean exactly four retries in every situation when more specific budgets and exception types are configured. Test the behavior with the version installed in your environment.

## 8. Handling HTTP 429 and `Retry-After`

An API may return HTTP `429` when the client exceeds its rate limit.

The response may include a `Retry-After` header telling the client when to try again.

For example:

```text
HTTP/1.1 429 Too Many Requests
Retry-After: 10
```

This indicates that the client should wait approximately ten seconds before retrying.

With the `urllib3` retry configuration above, `respect_retry_after_header=True` enables support for this behavior on applicable responses.

**Best practice:** Respect the server's instructions, use bounded retries, and avoid retrying more aggressively than the API permits.

## 9. Which Errors Should Be Retried?

A good retry strategy classifies failures rather than retrying everything.

| Failure | Typical handling |
|---|---|
| Connection timeout | Retry within a limit |
| Temporary connection error | Retry within a limit |
| Read timeout | Retry if repeating the operation is safe |
| HTTP `429` | Respect `Retry-After`; retry within a limit |
| HTTP `500` | Often retry, depending on the operation |
| HTTP `502`, `503`, `504` | Often retry |
| HTTP `400` | Usually fix the request |
| HTTP `401` | Fix authentication or refresh credentials |
| HTTP `403` | Check permissions |
| HTTP `404` | Usually check the endpoint or resource |
| Invalid JSON | Investigate the response; do not automatically assume retrying will help |

These are general guidelines, not universal rules. The service's documented behavior determines the correct policy.

## 10. The Danger of Retrying POST Requests

Consider a pipeline that submits a payment or creates an order using POST.

1. The client sends the request.
2. The server creates the order.
3. The response is lost because of a network interruption.
4. The client sees a timeout and retries.
5. The server creates a second order.

The operation may have happened twice even though the client did not receive the first response.

This is called a duplicate-operation risk.

### How can you prevent it?

- Use an API-supported idempotency key when available.
- Check the operation's status before repeating it, if the API supports that.
- Use an idempotent design.
- Follow the API's retry guidance for POST, PUT, PATCH, and DELETE.

A method's HTTP definition alone does not guarantee that the real operation is safe to repeat. For example, a PUT endpoint may still have application-specific side effects.

For read-only GET requests, retries are generally safer, although the data returned may change between attempts.

## 11. Retries in Data Engineering Pipelines

Imagine a pipeline that extracts orders from an API and writes them to a data lake.

A reliable workflow might look like this:

1. Send a request with a connection and read timeout.
2. Check the HTTP status code.
3. Retry transient failures with exponential backoff and jitter.
4. Stop after the retry budget is exhausted.
5. Log the failure and relevant request context without exposing secrets.
6. Process and persist successful data in batches.
7. Advance the checkpoint only after the batch has been safely persisted.
8. Resume failed jobs without creating duplicate records.

### Why are checkpoints important?

Suppose a pipeline successfully processes 20 pages and fails on page 21.

Without checkpoints, it may need to fetch the first 20 pages again.

With suitable checkpointing, it can resume from the last safely completed position, provided the API's pagination and consistency rules support this.

Retries handle individual failed operations; checkpoints help recover the larger workflow.

## 12. Retries vs. Timeouts

| Feature | Timeout | Retry |
|---|---|---|
| Purpose | Limits waiting | Repeats an operation after failure |
| Prevents | Excessive waiting for a request | Immediate failure from some temporary problems |
| Example | Stop waiting for a response after read inactivity | Try the request again after a timeout |
| Main risk | Operation may fail even if the service eventually responds | Repeated requests can overload services or duplicate side effects |
| Should be bounded? | Yes, according to the operation's needs | Yes |

**They work together:** a request can time out, and a retry policy can decide whether to attempt it again.

## 13. Common Mistakes

1. **No timeout configured:** Requests can wait longer than expected.
2. **Infinite retries:** A permanently failing service can keep a worker busy indefinitely.
3. **Retrying every HTTP error:** Invalid requests will not usually succeed through repeated attempts.
4. **No backoff:** Immediate retries can overload a failing service.
5. **Ignoring `Retry-After`:** This can violate API rate limits.
6. **Retrying non-idempotent operations blindly:** This can create duplicate records or transactions.
7. **Logging secrets:** Access tokens, passwords, and sensitive payloads should not appear in logs.
8. **Ignoring total pipeline duration:** Even bounded individual retries can make a job exceed its overall execution deadline.
9. **Not monitoring retries:** Frequent retries may indicate a degraded API or a configuration problem.
10. **Advancing checkpoints too early:** A failed write may cause records to be skipped during recovery.

## 14. Interview Questions

### Q1. What is a timeout?

A limit on how long a program waits for an operation, such as establishing a connection or receiving data.

### Q2. What is the difference between connection timeout and read timeout?

A connection timeout limits waiting while establishing a connection. A read timeout limits how long the client waits for data to arrive during the response.

### Q3. What is exponential backoff?

A retry strategy in which the waiting period increases exponentially after repeated failures.

### Q4. Why is jitter useful?

It spreads retries across time so many clients do not retry simultaneously.

### Q5. Should every error be retried?

No. Retry transient failures when appropriate. Fix permanent problems such as invalid requests, missing permissions, and incorrect authentication.

### Q6. What does HTTP 429 mean?

The client has exceeded a rate limit or is being throttled. Follow the API's rate-limit policy and respect `Retry-After` when provided.

### Q7. Why can retrying a POST request be dangerous?

The server may have completed the original operation even though the client did not receive the response. Repeating the request can cause duplicate side effects.

### Q8. How would you make an API extraction pipeline reliable?

Use timeouts, bounded retries, backoff, appropriate status-code handling, structured logging, rate-limit awareness, checkpointing, and safe incremental writes.

### Q9. What is the difference between retry and recovery?

A retry repeats a failed operation. Recovery restores a larger workflow after failure, potentially using checkpoints, persisted state, and deduplication.

### Q10. Can retries guarantee success?

No. Retries can help with temporary failures, but permanent errors, prolonged outages, expired credentials, and invalid requests still require appropriate intervention.

## 15. Quick Revision

- **Timeout:** Limits waiting for an operation.
- **Retry:** Attempts an operation again after a failure.
- **Connection timeout:** Limits time spent establishing a connection.
- **Read timeout:** Limits waiting for incoming data.
- **Exponential backoff:** Increases the delay between attempts.
- **Jitter:** Adds randomness to retry delays.
- **HTTP 429:** Respect rate limits and `Retry-After`.
- **Retry budget:** Limits repeated attempts.
- **Idempotency:** Helps make repeated operations safe.
- **Checkpointing:** Helps resume a pipeline after failure.

**For Data Engineer interviews, focus on:** timeouts, transient vs. permanent errors, exponential backoff, jitter, HTTP 429, idempotency, retries in API extraction, and checkpoint-based recovery.