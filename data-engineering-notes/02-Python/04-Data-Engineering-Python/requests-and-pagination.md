# Requests and Pagination in Python

## 1. What Is the `requests` Library?

The `requests` library allows Python programs to communicate with web servers and APIs using HTTP.

In Data Engineering, it is commonly used to:

- Extract data from REST APIs.
- Fetch JSON responses from external systems.
- Retrieve paginated datasets.
- Send data to APIs.
- Integrate external data sources into ETL/ELT pipelines.

Install it:

```bash
pip install requests
```

Import it:

```python
import requests
```

## 2. Understanding HTTP Methods

HTTP methods describe the action a client wants to perform.

| Method | Purpose | Example |
|---|---|---|
| `GET` | Retrieve data | Fetch customer records |
| `POST` | Create or submit data | Send a new order |
| `PUT` | Replace a resource | Replace customer details |
| `PATCH` | Partially update a resource | Update a customer's email |
| `DELETE` | Delete a resource | Remove a record |

For data extraction pipelines, **GET is the most common method**.

## 3. Making a GET Request

```python
import requests

url = "https://jsonplaceholder.typicode.com/users"

response = requests.get(url)

print(response.status_code)
print(response.json())
```

### Understanding the code

- `requests.get(url)` sends an HTTP GET request.
- `response.status_code` returns the HTTP status code.
- `response.json()` parses a JSON response into Python objects, usually a list or dictionary.

Common status codes:

| Code | Meaning |
|---|---|
| `200` | Request successful |
| `201` | Resource created |
| `204` | Successful request with no response body |
| `400` | Bad request |
| `401` | Authentication required or invalid |
| `403` | Access forbidden |
| `404` | Resource not found |
| `429` | Too many requests |
| `500` | Internal server error |
| `503` | Service temporarily unavailable |

## 4. Check Whether a Request Succeeded

A request can return an HTTP error even when Python itself executes without an exception.

Use `raise_for_status()` to raise an exception for unsuccessful HTTP status codes, generally 4xx and 5xx.

```python
import requests

url = "https://jsonplaceholder.typicode.com/users"

try:
    response = requests.get(url, timeout=10)
    response.raise_for_status()

    users = response.json()
    print(users)

except requests.exceptions.Timeout:
    print("The API request timed out")

except requests.exceptions.HTTPError as e:
    print(f"HTTP error: {e}")

except requests.exceptions.RequestException as e:
    print(f"Request failed: {e}")
```

### Important distinction

- `raise_for_status()` checks the HTTP status.
- `response.json()` parses the response body.
- `timeout=10` limits how long the request waits for a connection or response activity. It is not necessarily a strict ten-second total deadline.

A response with status `200` can still contain invalid JSON. Handle JSON parsing errors separately if the API response is not fully trusted.

## 5. Passing Query Parameters

APIs often accept parameters to filter or control returned data.

Example URL:

```text
https://api.example.com/orders?status=completed&limit=100
```

Instead of constructing the URL manually, use the `params` argument.

```python
import requests

url = "https://api.example.com/orders"

params = {
    "status": "completed",
    "limit": 100
}

response = requests.get(
    url,
    params=params,
    timeout=10
)

response.raise_for_status()

orders = response.json()
print(orders)
```

The library encodes the parameters correctly.

**Why use `params`?**

- Easier to read and maintain.
- Handles URL encoding.
- Makes filters and pagination parameters easier to change.
- Avoids many manual URL-construction mistakes.

## 6. Sending Authentication Information

Many APIs require authentication.

### Bearer token

```python
import requests

url = "https://api.example.com/customers"

headers = {
    "Authorization": "Bearer YOUR_ACCESS_TOKEN",
    "Accept": "application/json"
}

response = requests.get(
    url,
    headers=headers,
    timeout=10
)

response.raise_for_status()

customers = response.json()
```

Other APIs may use API keys, basic authentication, or OAuth flows.

**Best practice:** Keep real credentials in environment variables or a secret manager. Do not hardcode tokens in source code or commit them to Git.

## 7. What Is Pagination?

Pagination is the process of retrieving a large dataset in smaller portions called *pages*.

Suppose an API contains 100,000 orders. Returning every order in one response could consume excessive memory, take too long, or exceed the API's response limits.

The API might return 1,000 records per request:

- Page 1: records 1–1,000
- Page 2: records 1,001–2,000
- Page 3: records 2,001–3,000
- Continue until no more records remain.

Pagination is especially important in Data Engineering because APIs often limit the number of records returned by each request.

## 8. Common Pagination Strategies

Different APIs use different pagination mechanisms. Always follow the API's documentation.

### A. Page-number pagination

The API accepts a page number and page size.

```python
params = {
    "page": 1,
    "limit": 100
}
```

Example:

```python
import requests

url = "https://api.example.com/orders"
all_orders = []
page = 1
page_size = 100

while True:
    params = {
        "page": page,
        "limit": page_size
    }

    response = requests.get(url, params=params, timeout=10)
    response.raise_for_status()

    data = response.json()
    orders = data["data"]

    if not orders:
        break

    all_orders.extend(orders)

    if len(orders) < page_size:
        break

    page += 1

print(f"Fetched {len(all_orders)} orders")
```

This example assumes the API returns a response shaped like:

```json
{
  "data": [
    {"order_id": 1},
    {"order_id": 2}
  ]
}
```

It also assumes that a short page indicates the end of the dataset. That assumption is valid only if the API documents this behavior.

### B. Offset-based pagination

The API accepts an offset and a limit.

```python
params = {
    "offset": 0,
    "limit": 100
}
```

The offset indicates how many records to skip.

| Request | Offset | Limit |
|---|---:|---:|
| First | 0 | 100 |
| Second | 100 | 100 |
| Third | 200 | 100 |
| Fourth | 300 | 100 |

Example:

```python
import requests

url = "https://api.example.com/orders"

all_orders = []
offset = 0
limit = 100

while True:
    params = {
        "offset": offset,
        "limit": limit
    }

    response = requests.get(url, params=params, timeout=10)
    response.raise_for_status()

    data = response.json()
    orders = data["data"]

    if not orders:
        break

    all_orders.extend(orders)
    offset += len(orders)

    if len(orders) < limit:
        break

print(f"Fetched {len(all_orders)} orders")
```

Using `offset += len(orders)` advances by the number of records actually returned.

**Limitation:** Offset pagination can become slower for large offsets. If records are inserted or deleted during extraction, pages can also contain duplicates or skip records, depending on the API's ordering and consistency guarantees.

### C. Cursor-based pagination

The API returns a cursor that identifies where the next request should continue.

Example response:

```json
{
  "data": [
    {"order_id": 1},
    {"order_id": 2}
  ],
  "next_cursor": "abc123"
}
```

The next request passes that cursor:

```python
params = {
    "cursor": "abc123",
    "limit": 100
}
```

Example:

```python
import requests

url = "https://api.example.com/orders"

all_orders = []
cursor = None
page_size = 100

while True:
    params = {"limit": page_size}

    if cursor is not None:
        params["cursor"] = cursor

    response = requests.get(url, params=params, timeout=10)
    response.raise_for_status()

    result = response.json()

    orders = result["data"]
    all_orders.extend(orders)

    cursor = result.get("next_cursor")

    if not cursor:
        break

print(f"Fetched {len(all_orders)} orders")
```

This example assumes that a missing or empty `next_cursor` means the extraction is complete.

**Advantages:**

- Often performs better for large datasets.
- Can be more reliable when new records are added during extraction.
- Avoids repeatedly skipping large numbers of records.

However, cursor pagination is not automatically consistent or duplicate-free. Those guarantees depend on the API.

### D. Link-based pagination

Some APIs provide the URL for the next page directly.

Example response:

```json
{
  "data": [
    {"order_id": 1}
  ],
  "next": "https://api.example.com/orders?page=2"
}
```

Example:

```python
import requests

url = "https://api.example.com/orders"
all_orders = []

while url:
    response = requests.get(url, timeout=10)
    response.raise_for_status()

    result = response.json()

    all_orders.extend(result["data"])
    url = result.get("next")

print(f"Fetched {len(all_orders)} orders")
```

In production, validate that pagination links point to trusted hosts before sending authentication headers or other sensitive information to them. Some APIs use a `Link` response header instead of a JSON `next` field.

## 9. Pagination: Important Production Considerations

### A. Prevent infinite loops

An API might repeatedly return the same cursor or next-page URL due to a server bug.

Consider implementing:

- A maximum page count.
- Repeated-cursor detection.
- Logging for every page.
- Monitoring for unexpectedly large datasets.

### B. Handle rate limits

Some APIs respond with `429 Too Many Requests`.

Do not repeatedly retry immediately. Respect `Retry-After` when provided, and use bounded retries with backoff.

### C. Use timeouts

Never allow an API request to wait indefinitely.

```python
response = requests.get(url, timeout=(5, 30))
```

Here, `5` is the connection timeout and `30` is the read timeout.

### D. Keep memory usage under control

This pattern stores the entire dataset in memory:

```python
all_orders.extend(orders)
```

For large datasets, process or persist each page incrementally instead of accumulating everything.

```python
while url:
    response = requests.get(url, timeout=30)
    response.raise_for_status()

    result = response.json()

    process_and_write_batch(result["data"])

    url = result.get("next")
```

`process_and_write_batch()` is a placeholder for your own database, file, or data-lake write logic.

### E. Design for retries and duplicates

A failed extraction may need to be restarted. A robust pipeline should consider:

- Idempotent writes or deduplication.
- Checkpointing the last successfully processed page or cursor.
- Avoiding advancement of the checkpoint before the batch is safely persisted.
- API consistency and ordering guarantees.

A cursor can expire, so a restart strategy should follow the API's documentation.

## 10. Reusable Function for Page-Based Extraction

The following example separates fetching from processing and includes a maximum-page safeguard.

```python
import requests


def fetch_all_pages(url, page_size=100, max_pages=10000):
    all_records = []

    with requests.Session() as session:
        for page in range(1, max_pages + 1):
            params = {
                "page": page,
                "limit": page_size
            }

            response = session.get(
                url,
                params=params,
                timeout=(5, 30)
            )
            response.raise_for_status()

            result = response.json()
            records = result["data"]

            if not records:
                return all_records

            all_records.extend(records)

            if len(records) < page_size:
                return all_records

    raise RuntimeError("Maximum page limit reached")


url = "https://api.example.com/orders"

# Example only: the endpoint must exist and use this response format.
# orders = fetch_all_pages(url)
```

`requests.Session()` reuses connections across requests, reducing connection overhead.

This is a teaching example. For a large production pipeline, replace the in-memory list with incremental writes, and implement the exact termination rules required by the API.

## 11. GET vs. POST for Data Extraction

A common misconception is that POST is used only to insert records.

Some APIs use POST to submit complex search filters or query requests, even when the purpose is to retrieve data. In such cases, follow the API's documentation.

The HTTP method alone does not tell you whether the returned data is paginated. Check the API response and pagination documentation.

## 12. Common Mistakes

1. **Not calling `raise_for_status()`** — HTTP errors can be mistaken for successful responses.
2. **Ignoring timeouts** — a request may stall for a long time.
3. **Assuming every API uses page numbers** — APIs may use offsets, cursors, or links.
4. **Stopping based on a short page without checking the contract** — some APIs use different completion signals.
5. **Fetching all records into memory** — large extractions may exhaust memory.
6. **Retrying every failure indefinitely** — permanent errors will never recover through retries alone.
7. **Ignoring rate limits** — excessive requests can cause throttling or temporary access restrictions.
8. **Hardcoding credentials** — secrets can leak into source control or logs.
9. **Assuming pagination produces a consistent snapshot** — records may change during extraction.
10. **Failing to checkpoint progress** — a pipeline may need to repeat all previous requests after a failure.

## 13. Interview Questions

### Q1. What is the `requests` library?

A Python library for making HTTP requests to web servers and APIs.

### Q2. What is the difference between `response.json()` and `response.text`?

- `response.json()` parses a JSON response into Python objects.
- `response.text` returns the response body as decoded text.

### Q3. Why do we use `raise_for_status()`?

To raise an HTTP-related exception when the response status indicates a client or server error.

### Q4. What is pagination?

A technique for retrieving a large dataset in smaller batches rather than one large response.

### Q5. What is the difference between offset and cursor pagination?

Offset pagination skips a specified number of records. Cursor pagination uses a continuation token to identify where the next request should resume.

### Q6. Which pagination strategy is better for large datasets?

Cursor-based pagination is often preferable for large datasets, but the best choice depends on the API's implementation, consistency guarantees, and available pagination options.

### Q7. Why is `requests.Session()` useful?

It reuses connections across requests, which can reduce connection overhead when fetching many pages.

### Q8. How would you handle API rate limits?

Use bounded retries, exponential backoff, respect `Retry-After`, limit concurrency, and log retry attempts.

### Q9. How would you extract millions of API records?

Use the documented pagination method, process data in batches, write incrementally to durable storage, checkpoint progress, handle retries, and make writes safe to repeat.

### Q10. What happens if the API fails after ten pages?

A robust pipeline should preserve completed work and resume from a valid checkpoint where possible. It should also ensure that repeated batches do not create duplicate or inconsistent data.

## 14. Quick Revision

- `requests.get()` — send a GET request.
- `params` — pass query parameters.
- `headers` — pass headers, including authentication when required.
- `response.status_code` — inspect the HTTP status.
- `response.raise_for_status()` — raise an exception for HTTP errors.
- `response.json()` — parse a JSON response.
- `timeout` — limit connection and read waiting times.
- `requests.Session()` — reuse connections across requests.
- **Page-based** — navigate by page number.
- **Offset-based** — navigate by skipped-record count.
- **Cursor-based** — continue using a cursor or token.
- **Link-based** — follow a provided next-page URL.
- **Production principle** — process incrementally, handle failures safely, and avoid relying on undocumented pagination assumptions.

**For Data Engineer interviews, focus on:** API extraction, authentication, status codes, timeouts, pagination strategies, rate limits, retries, incremental writes, and restartability.