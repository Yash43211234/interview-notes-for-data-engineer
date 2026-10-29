# Working with APIs in Python

## 1. What Is an API?

An API (Application Programming Interface) allows one software system to communicate with another through a defined interface.

In Data Engineering, APIs are commonly used to extract data from external systems such as payment platforms, CRM tools, weather services, and business applications.

For example, a data pipeline might:

1. Request customer data from a CRM API.
2. Receive the response in JSON format.
3. Validate and transform the data using Python.
4. Write the results to a database or data lake.
5. Schedule the pipeline to run periodically.

This is a common pattern for API-based data ingestion.

## 2. How Does an API Request Work?

<box gap={1} align="center">
  <box border radius="lg" padding={3} width="100%" align="center">
    **Python client**

    <text color="secondary" size="sm">Sends an HTTP request</text>
  </box>
  <box align="center" gap={1}>
    <icon name="arrow-down" size="xl" />
    <text size="sm" color="secondary">URL + method + headers + parameters</text>
  </box>
  <box border radius="lg" padding={3} width="100%" align="center">
    **API server**

    <text color="secondary" size="sm">Authenticates, processes, and retrieves data</text>
  </box>
  <box align="center" gap={1}>
    <icon name="arrow-down" size="xl" />
    <text size="sm" color="secondary">HTTP status + response body</text>
  </box>
  <box border radius="lg" padding={3} width="100%" align="center">
    **Python pipeline**

    <text color="secondary" size="sm">Validates, transforms, and stores the response</text>
  </box>
</box>

The client sends a request to an endpoint. The server processes it and returns a response, commonly containing JSON data.

## 3. Important API Terminology

| Term | Meaning |
|---|---|
| API | Interface that allows software systems to communicate |
| Endpoint | Specific URL used to access a resource or operation |
| HTTP method | Action requested, such as GET or POST |
| Request headers | Metadata sent with the request |
| Query parameters | Values used to filter or control a request |
| Request body | Data sent to the server, often with POST or PATCH |
| Response | Data and status returned by the server |
| Status code | Indicates the result of the HTTP request |
| Authentication | Verifies the identity or credentials of the client |
| Pagination | Retrieves a large dataset in smaller batches |
| Rate limit | Restricts how frequently a client can make requests |

## 4. Common HTTP Methods

| Method | Purpose | Example |
|---|---|---|
| `GET` | Retrieve data | Fetch customer records |
| `POST` | Create a resource or submit a request | Create an order |
| `PUT` | Replace a resource | Replace customer details |
| `PATCH` | Partially update a resource | Update an email address |
| `DELETE` | Delete a resource | Delete a record |

For Data Engineering, GET is especially common during data extraction. However, some APIs use POST to submit complex search criteria for retrieving data.

## 5. Set Up Python Requests

The `requests` library is a common choice for communicating with HTTP APIs.

Install it:

```bash
pip install requests
```

Import it:

```python
import requests
```

Let's use a public example API that returns sample user data:

```python
import requests

url = "https://jsonplaceholder.typicode.com/users"

response = requests.get(url, timeout=10)

response.raise_for_status()

users = response.json()

print(users)
```

### Understanding the code

- `requests.get()` sends an HTTP GET request.
- `timeout=10` configures connection and read timeouts.
- `raise_for_status()` raises an exception for HTTP error responses.
- `response.json()` parses the JSON response into Python objects.

The timeout is not a strict total execution deadline; it limits connection and read waiting behavior.

## 6. Understanding API Responses

An API response usually contains a status code, headers, and a body.

```python
import requests

url = "https://jsonplaceholder.typicode.com/users"

response = requests.get(url, timeout=10)

print(response.status_code)
print(response.headers.get("Content-Type"))
print(response.text)
```

Common status codes:

| Status | Meaning |
|---|---|
| `200` | Successful request |
| `201` | Resource created |
| `204` | Successful request with no response body |
| `400` | Bad request |
| `401` | Authentication required or invalid |
| `403` | Access forbidden |
| `404` | Resource not found |
| `429` | Rate limit exceeded |
| `500` | Internal server error |
| `503` | Service temporarily unavailable |

A successful HTTP status does not automatically mean the response contains valid JSON or the fields your pipeline expects. Validate the content before processing it.

## 7. Passing Query Parameters

Query parameters control which records the API returns.

Example URL:

```text
https://api.example.com/orders?status=completed&limit=100
```

In Python, use the `params` argument instead of manually constructing the query string.

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

data = response.json()
```

The example endpoint is illustrative. Its actual parameters depend on the API documentation.

## 8. Sending Headers

Headers provide additional information about the request.

For example, an API may expect a JSON response or a bearer token.

```python
import requests

url = "https://api.example.com/customers"

headers = {
    "Accept": "application/json",
    "Authorization": "Bearer YOUR_ACCESS_TOKEN"
}

response = requests.get(
    url,
    headers=headers,
    timeout=10
)

response.raise_for_status()

customers = response.json()
```

Never commit real access tokens or passwords to source control. Use environment variables or a secrets manager to supply credentials securely.

## 9. API Authentication

Authentication methods vary by API.

### A. API key

Some APIs require a key in a header or query parameter.

```python
headers = {
    "X-API-Key": "YOUR_API_KEY"
}
```

The header name depends on the provider.

### B. Bearer token

A bearer token is typically passed in the `Authorization` header.

```python
headers = {
    "Authorization": "Bearer YOUR_ACCESS_TOKEN"
}
```

### C. Basic authentication

Some APIs accept a username and password through HTTP Basic Authentication.

```python
import requests

response = requests.get(
    "https://api.example.com/data",
    auth=("username", "password"),
    timeout=10
)
```

Use this only when the API supports it, and avoid hardcoding credentials.

### D. OAuth 2.0

OAuth 2.0 is a framework used to obtain and authorize access tokens. A pipeline may need to acquire a token, refresh it when required, and then use it to call the API.

The token acquisition flow depends on the provider and the application's permissions.

## 10. Handling Errors

API calls can fail due to network issues, authentication problems, server errors, or invalid responses.

```python
import requests

url = "https://jsonplaceholder.typicode.com/users"

try:
    response = requests.get(url, timeout=(5, 20))
    response.raise_for_status()

    users = response.json()
    print(f"Fetched {len(users)} users")

except requests.exceptions.Timeout:
    print("The request timed out")

except requests.exceptions.HTTPError as e:
    print(f"HTTP error: {e}")

except requests.exceptions.RequestException as e:
    print(f"Request failed: {e}")

except ValueError:
    print("The response could not be parsed as JSON")
```

The `ValueError` handler catches JSON parsing errors from `response.json()` in this context. For more complex pipelines, validate the response schema separately.

**Important:** Do not automatically retry every error. A temporary server error may be retryable, while an invalid request or missing permission usually needs a configuration or code change.

## 11. Pagination

Many APIs limit how many records they return in a single response. Pagination allows the client to retrieve the complete dataset in multiple requests.

Common strategies include:

- **Page-based:** Uses page numbers.
- **Offset-based:** Uses an offset and a limit.
- **Cursor-based:** Uses a continuation token.
- **Link-based:** Uses a URL supplied for the next page.

Example of page-based pagination:

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

    response = requests.get(
        url,
        params=params,
        timeout=10
    )
    response.raise_for_status()

    result = response.json()
    orders = result["data"]

    if not orders:
        break

    all_orders.extend(orders)

    if len(orders) < page_size:
        break

    page += 1
```

This example assumes the API returns records under `data` and that a short page indicates the final page. Follow the actual API's pagination contract instead of assuming every API behaves this way.

For large datasets, process and persist each page incrementally rather than accumulating all records in memory.

## 12. Retries and Timeouts

A robust pipeline should configure timeouts and retry appropriate transient failures.

A timeout limits how long a request waits. A retry attempts the operation again after an eligible failure.

For example, a request may fail temporarily with a `503 Service Unavailable` response. A bounded retry policy with exponential backoff may help.

Production retry policies should consider:

- Maximum attempts.
- Exponential backoff and jitter.
- HTTP status codes that can be retried.
- The API's `Retry-After` header.
- Whether the operation is safe to repeat.
- Total pipeline execution time.

For read-only GET requests, retries are generally safer than for operations that create or modify data. Even then, responses may change between attempts.

## 13. Using `requests.Session()`

A `Session` object reuses connections across requests to the same server, reducing repeated connection overhead.

This is useful when retrieving multiple pages or making many requests to the same API.

```python
import requests

url = "https://jsonplaceholder.typicode.com/users"

with requests.Session() as session:
    response = session.get(url, timeout=(5, 20))
    response.raise_for_status()

    users = response.json()

    print(f"Fetched {len(users)} users")
```

A session can also maintain shared headers, cookies, and other request settings.

## 14. Processing JSON into Tabular Data

APIs often return nested JSON, while analytics systems typically require tabular data.

Suppose an API returns:

```json
{
  "data": [
    {
      "order_id": 101,
      "customer_id": 20,
      "amount": 450.5
    },
    {
      "order_id": 102,
      "customer_id": 21,
      "amount": 700.0
    }
  ]
}
```

You can convert the records into a Pandas DataFrame.

```python
import requests
import pandas as pd

url = "https://api.example.com/orders"

response = requests.get(url, timeout=10)
response.raise_for_status()

payload = response.json()

df = pd.DataFrame(payload["data"])

print(df)
```

Install Pandas if needed:

```bash
pip install pandas
```

For large responses, create a DataFrame per batch and write the batches incrementally instead of loading the entire dataset at once.

## 15. API Integration in a Data Engineering Pipeline

A typical API ingestion pipeline follows this pattern:

<box border radius="lg" padding={3} gap={2}>
  <box background="surface-secondary" radius="md" padding={2} align="center">
    **1. Extract**

    <text size="sm">Call the API and retrieve records</text>
  </box>
  <box align="center"><icon name="arrow-down" /></box>
  <box background="surface-secondary" radius="md" padding={2} align="center">
    **2. Validate**

    <text size="sm">Check status codes, fields, types, and null values</text>
  </box>
  <box align="center"><icon name="arrow-down" /></box>
  <box background="surface-secondary" radius="md" padding={2} align="center">
    **3. Transform**

    <text size="sm">Clean and normalize the data</text>
  </box>
  <box align="center"><icon name="arrow-down" /></box>
  <box background="surface-secondary" radius="md" padding={2} align="center">
    **4. Load**

    <text size="sm">Write batches to storage or a database</text>
  </box>
  <box align="center"><icon name="arrow-down" /></box>
  <box background="surface-secondary" radius="md" padding={2} align="center">
    **5. Monitor**

    <text size="sm">Log failures, record counts, and pipeline duration</text>
  </box>
</box>

For example, a Data Engineer might extract orders from a sales API, store the raw JSON in Azure Data Lake Storage, transform the data using PySpark in Databricks, and write curated Delta tables.

This separates raw ingestion from downstream transformations and makes it easier to recover from failures.

## 16. Best Practices for Production API Pipelines

1. **Use timeouts:** Avoid requests that wait indefinitely.
2. **Validate status codes:** Call `raise_for_status()` before processing the response.
3. **Validate the response structure:** Check required fields and expected data types.
4. **Handle pagination:** Retrieve every required page using the documented mechanism.
5. **Respect rate limits:** Follow `Retry-After` and provider-specific limits.
6. **Use bounded retries:** Retry transient errors without creating endless loops.
7. **Protect credentials:** Use a secrets manager or environment variables.
8. **Avoid logging sensitive data:** Never log access tokens or unnecessary personal information.
9. **Process incrementally:** Persist batches to control memory usage.
10. **Make writes restart-safe:** Use checkpoints, deduplication, or idempotent writes where appropriate.
11. **Monitor extraction quality:** Track record counts, failures, and schema changes.
12. **Use HTTPS:** Protect data in transit and verify server certificates.

## 17. Common Mistakes

- Treating every HTTP response as successful.
- Forgetting to set a timeout.
- Assuming every API returns JSON.
- Ignoring authentication and token expiration.
- Assuming every API uses the same pagination mechanism.
- Retrying permanent errors indefinitely.
- Fetching millions of records into memory at once.
- Hardcoding credentials in Python files.
- Ignoring API rate limits.
- Assuming an HTTP success response means the returned data is complete or correct.
- Failing to handle schema changes.
- Repeating write operations without considering duplicate side effects.

## 18. Interview Questions

### Q1. What is an API?

An interface that enables software systems to communicate through defined requests and responses.

### Q2. What is a REST API?

A web API that follows REST architectural principles, commonly using HTTP methods and resource-oriented endpoints. Not every HTTP API is RESTful.

### Q3. What is the difference between an endpoint and a query parameter?

An endpoint identifies the resource or operation being accessed. A query parameter modifies or filters the request.

### Q4. Why is JSON commonly used in APIs?

JSON is a lightweight text format that supports structured objects and arrays and is easy to exchange between different programming languages.

### Q5. How do you handle API failures?

Use timeouts, status-code validation, suitable exception handling, bounded retries for transient errors, and logging.

### Q6. How do you extract millions of records from an API?

Use pagination, process batches incrementally, write to durable storage, checkpoint progress, respect rate limits, and make recovery safe.

### Q7. What is the difference between authentication and authorization?

Authentication establishes who or what the client is. Authorization determines what the authenticated client is allowed to access.

### Q8. How do you handle API rate limits?

Control request frequency, respect `Retry-After`, use bounded backoff, and avoid unnecessary repeated requests.

### Q9. Why would you use `requests.Session()`?

It reuses connections and can maintain common request settings across multiple API calls.

### Q10. How would you design an API-to-data-lake pipeline?

Extract data with authentication and pagination, validate the response, store raw data, transform it in batches, write curated outputs, checkpoint progress, and monitor failures and data quality.

## 19. Quick Revision

- **API:** Interface for communication between systems.
- **Endpoint:** URL for a resource or operation.
- **GET:** Retrieve data.
- **POST:** Submit data or create a resource.
- **Headers:** Carry metadata and often authentication information.
- **Query parameters:** Control filtering and pagination.
- **`requests.get()`:** Send an HTTP GET request.
- **`response.json()`:** Parse JSON into Python objects.
- **`raise_for_status()`:** Raise an exception for HTTP error responses.
- **Pagination:** Retrieve data in multiple requests.
- **Retries and timeouts:** Improve reliability when appropriately configured.
- **`requests.Session()`:** Reuse connections.
- **Data Engineering principle:** Extract reliably, validate carefully, process incrementally, and make pipeline recovery safe.

**For Data Engineer interviews, focus on:** REST APIs, HTTP methods, status codes, authentication, JSON parsing, pagination, retries, timeouts, rate limits, and API-to-data-lake ingestion.