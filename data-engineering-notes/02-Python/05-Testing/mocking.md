# Mocking in Python

## 1. What Is Mocking?

Mocking is a testing technique in which we replace a real dependency with a controlled substitute called a **mock object**.

It allows us to test a piece of code without actually calling its external dependencies.

Common dependencies that we mock include:

- External APIs
- Databases
- File systems
- Cloud storage services
- Email services
- Message queues

### Example from Data Engineering

Suppose a pipeline downloads customer data from an external API.

Without mocking, every test might send a real HTTP request. This can make testing slow, unreliable, or dependent on API availability.

With mocking, we simulate the API response and test how our pipeline processes the data.

**Main idea:** Test your code's behavior without depending on the real external system.

## 2. Why Is Mocking Important?

Mocking helps us:

1. Test code without accessing real external services.
2. Avoid unnecessary API calls and database queries.
3. Simulate failures such as timeouts and connection errors.
4. Test edge cases that are difficult to reproduce in production.
5. Make unit tests faster and more predictable.
6. Verify that dependencies are called with the expected arguments.

For Data Engineers, mocking is especially useful when testing ETL pipelines, API extraction functions, database integrations, and error-handling logic.

## 3. Understanding `unittest.mock`

Python provides a built-in module called `unittest.mock`.

No separate installation is required.

```python
from unittest.mock import Mock
```

The `Mock` class creates an object whose behavior you can configure.

### Basic example

```python
from unittest.mock import Mock

api = Mock()

api.get_customer.return_value = {
    "customer_id": 101,
    "name": "Rahul"
}

customer = api.get_customer(101)

print(customer)
```

Output:

```text
{'customer_id': 101, 'name': 'Rahul'}
```

Here:

- `Mock()` creates a mock object.
- `return_value` defines what a mocked function returns.
- `get_customer(101)` returns the configured value without calling a real API.

## 4. Mocking a Function

Suppose you have a function that calculates the total order amount.

```python
def calculate_total(price, quantity):
    return price * quantity
```

You could test the real function directly. But if the function depends on another function, you may want to replace that dependency.

Consider this code:

```python
def get_price(product_id):
    # Imagine this function queries an external system.
    raise NotImplementedError


def calculate_order(product_id, quantity):
    price = get_price(product_id)
    return price * quantity
```

We can mock `get_price()` to test `calculate_order()` without accessing the external system.

```python
from unittest.mock import patch


def test_calculate_order():
    with patch("__main__.get_price", return_value=100):
        result = calculate_order(10, 3)

    assert result == 300
```

The example assumes the functions are defined in the same script, so `__main__` is their namespace. In a real project, patch the name where the code under test looks it up.

The test verifies the calculation while replacing the dependency with a predictable value.

## 5. Using `patch()`

`patch()` temporarily replaces an object with a mock or another specified value during a test.

It is useful when your code imports or calls a function from another module.

Suppose you have the following project structure:

```text
project/
├── api_client.py
├── pipeline.py
└── test_pipeline.py
```

In `api_client.py`:

```python
import requests


def fetch_customers():
    response = requests.get(
        "https://api.example.com/customers",
        timeout=10
    )
    response.raise_for_status()
    return response.json()
```

In `pipeline.py`:

```python
from api_client import fetch_customers


def count_customers():
    customers = fetch_customers()
    return len(customers)
```

In `test_pipeline.py`:

```python
from unittest.mock import patch
from pipeline import count_customers


def test_count_customers():
    fake_customers = [
        {"id": 1, "name": "A"},
        {"id": 2, "name": "B"}
    ]

    with patch(
        "pipeline.fetch_customers",
        return_value=fake_customers
    ):
        result = count_customers()

    assert result == 2
```

### Why patch `pipeline.fetch_customers`?

Because `pipeline.py` imported `fetch_customers` directly, and `count_customers()` looks up that name in the `pipeline` module.

**Important rule: Patch the name where it is used, not necessarily where it was originally defined.**

## 6. Mocking HTTP API Responses

For Data Engineering, one of the most useful applications of mocking is testing API extraction code.

Suppose your function retrieves orders:

```python
import requests


def fetch_orders(url):
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    return response.json()
```

We can mock `requests.get()` so that the test does not send a real HTTP request.

```python
from unittest.mock import Mock, patch


def test_fetch_orders():
    fake_response = Mock()

    fake_response.json.return_value = [
        {"order_id": 1, "amount": 500},
        {"order_id": 2, "amount": 700}
    ]

    fake_response.raise_for_status.return_value = None

    with patch(
        "your_module.requests.get",
        return_value=fake_response
    ) as mock_get:

        orders = fetch_orders(
            "https://api.example.com/orders"
        )

    assert len(orders) == 2
    mock_get.assert_called_once_with(
        "https://api.example.com/orders",
        timeout=10
    )
```

Replace `your_module` with the actual module containing `fetch_orders()`.

This test verifies that:

- The function uses the mocked response.
- The returned records are processed correctly.
- The HTTP request is made with the expected arguments.
- No real network request is necessary.

## 7. Testing API Failures with Mocking

A reliable pipeline must handle API failures. Mocking makes it possible to test these cases without deliberately breaking a real service.

### Example: HTTP error

```python
import requests
from unittest.mock import Mock, patch


def fetch_data(url):
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    return response.json()


def test_fetch_data_http_error():
    fake_response = Mock()

    fake_response.raise_for_status.side_effect = (
        requests.exceptions.HTTPError("500 Server Error")
    )

    with patch(
        "your_module.requests.get",
        return_value=fake_response
    ):
        try:
            fetch_data("https://api.example.com/data")
            assert False, "Expected HTTPError"
        except requests.exceptions.HTTPError:
            pass
```

Replace `your_module` with the module containing `fetch_data()`.

### Understanding `side_effect`

`side_effect` specifies what happens when a mocked function is called.

It can be used to:

- Raise an exception.
- Return different values on successive calls.
- Execute a custom function.

For example:

```python
from unittest.mock import Mock

api_call = Mock(
    side_effect=[
        TimeoutError("Temporary failure"),
        {"status": "success"}
    ]
)

try:
    api_call()
except TimeoutError:
    print("First call failed")

print(api_call())
```

Output:

```text
First call failed
{'status': 'success'}
```

The first call raises an exception, and the second call returns the configured dictionary.

This is useful for testing retry logic.

## 8. Mocking Retry Logic

Suppose a function retries an API request after a temporary connection failure.

```python
import requests


def fetch_with_retry(url):
    for attempt in range(3):
        try:
            response = requests.get(url, timeout=10)
            response.raise_for_status()
            return response.json()

        except requests.exceptions.ConnectionError:
            if attempt == 2:
                raise
```

We can simulate a failure followed by success.

```python
import requests
from unittest.mock import Mock, patch


def test_retry_success():
    failed_response = Mock()
    successful_response = Mock()

    successful_response.json.return_value = [
        {"id": 1}
    ]

    with patch(
        "your_module.requests.get",
        side_effect=[
            requests.exceptions.ConnectionError("Network error"),
            successful_response
        ]
    ) as mock_get:

        result = fetch_with_retry(
            "https://api.example.com/data"
        )

    assert result == [{"id": 1}]
    assert mock_get.call_count == 2
```

Replace `your_module` with the actual module name.

The test confirms that the function makes another request after a connection error and returns the successful response.

If the production retry function includes delays, inject or mock the sleep function so tests do not wait unnecessarily.

## 9. `return_value` vs. `side_effect`

These are two important mock features.

| Feature | `return_value` | `side_effect` |
|---|---|---|
| Purpose | Specify a return value | Define more flexible behavior |
| Repeated calls | Usually return the same configured value | Can return different values or raise exceptions |
| Best use | Simulate a predictable successful response | Simulate failures, retries, or changing responses |

Example:

```python
from unittest.mock import Mock

# Same return value on each call
mock_a = Mock(return_value=100)

print(mock_a())
print(mock_a())

# Different values on successive calls
mock_b = Mock(side_effect=[100, 200])

print(mock_b())
print(mock_b())
```

Output:

```text
100
100
100
200
```

When an iterable is used as `side_effect`, calling the mock after the iterable is exhausted raises `StopIteration`.

## 10. Verifying Mock Calls

Mocking is not only about controlling returned values. You can also verify how your code interacts with dependencies.

```python
from unittest.mock import Mock

database = Mock()

database.insert_record(101, "Rahul")

database.insert_record.assert_called_once_with(
    101,
    "Rahul"
)
```

Useful assertions include:

```python
mock_function.assert_called_once()
mock_function.assert_called_once_with("orders")
mock_function.assert_called_with("orders")
```

You can also inspect:

```python
print(mock_function.call_count)
print(mock_function.call_args)
print(mock_function.call_args_list)
```

These are useful when checking that a pipeline makes the correct API calls, sends the correct query parameters, or writes the expected records.

## 11. Mocking vs. Patching vs. Stubbing

These terms are related, but they describe slightly different ideas.

| Term | Meaning |
|---|---|
| Mocking | Replacing a dependency with a controlled test double |
| Mock object | An object that can simulate behavior and record interactions |
| Patching | Temporarily replacing a name or object during a test |
| Stubbing | Providing predefined responses for a dependency |
| Integration testing | Testing how real components work together |
| Unit testing | Testing a small unit of code in isolation |

A stub usually focuses on providing a response. A mock often also verifies interactions, such as how many times a function was called.

In practice, Python's `Mock` object can be used for both purposes.

## 12. Mocking in ETL Pipelines

Consider a pipeline with three steps:

1. Extract records from an API.
2. Transform the records.
3. Load them into a database.

You can test the transformation logic independently by replacing the extraction and loading dependencies.

```python
def transform_orders(orders):
    return [
        {
            "order_id": order["order_id"],
            "amount_with_tax": order["amount"] * 1.18
        }
        for order in orders
    ]


def run_pipeline(fetch_orders, load_orders):
    orders = fetch_orders()
    transformed = transform_orders(orders)
    load_orders(transformed)
```

Test the pipeline:

```python
from unittest.mock import Mock


def test_run_pipeline():
    fetch_orders = Mock(
        return_value=[
            {"order_id": 1, "amount": 100}
        ]
    )

    load_orders = Mock()

    run_pipeline(fetch_orders, load_orders)

    fetch_orders.assert_called_once()

    load_orders.assert_called_once_with([
        {
            "order_id": 1,
            "amount_with_tax": 118.0
        }
    ])
```

This test checks that the pipeline calls the extraction function, transforms the records correctly, and passes the transformed data to the loading function.

The example assumes the functions are supplied as dependencies. This design makes the pipeline easier to test because its external operations can be replaced.

## 13. Best Practices

1. **Mock external dependencies, not every function.** Keep the actual business logic under test whenever possible.
2. **Patch the correct namespace.** Replace the name that the code under test actually uses.
3. **Test success and failure paths.** Include timeouts, connection errors, and HTTP errors where relevant.
4. **Verify interactions.** Check arguments and call counts when those interactions are part of the expected behavior.
5. **Keep mocks realistic.** Mock response structures should match the real API's contract.
6. **Avoid over-mocking.** Excessive mocking can make tests pass even when the real components do not work together.
7. **Use integration tests too.** Mock-based unit tests do not prove that the actual API or database integration works.
8. **Use `spec` or `autospec` where appropriate.** These can help catch incorrect attribute access or function signatures.
9. **Keep tests independent.** Each test should set up its own mock behavior.
10. **Avoid real credentials and sensitive data.** Use synthetic test data.

## 14. Common Mistakes

- Mocking the wrong module path.
- Forgetting to configure `return_value`.
- Using `return_value` when successive calls need different results.
- Confusing `side_effect` with `return_value`.
- Testing only successful responses.
- Mocking the transformation logic itself instead of testing it.
- Assuming mocked tests prove the external API is working.
- Allowing a test to make a real network request unintentionally.
- Making assertions about internal implementation details that are not part of the expected behavior.

## 15. Interview Questions

### Q1. What is mocking?

Mocking replaces a dependency with a controlled object so code can be tested independently of external systems.

### Q2. Why is mocking useful in Data Engineering?

It enables testing API extraction, database interactions, retry behavior, and ETL pipeline logic without requiring real external services for every test.

### Q3. What is `unittest.mock`?

A built-in Python module that provides tools such as `Mock`, `MagicMock`, and `patch` for replacing dependencies during tests.

### Q4. What is the difference between `Mock` and `patch`?

`Mock` creates a mock object. `patch` temporarily replaces a name or object with a mock or another value.

### Q5. What is the difference between `return_value` and `side_effect`?

`return_value` defines what a mock returns. `side_effect` can define sequential return values, raise exceptions, or execute custom behavior.

### Q6. How would you test an API call without making a real HTTP request?

Patch the HTTP request function, configure a mock response, and assert that the application handles the response correctly.

### Q7. How would you test retry logic?

Configure a mock to raise a transient exception on the first call and return a successful response on a later call. Then verify the returned result and number of attempts.

### Q8. Can mocking replace integration testing?

No. Mocking helps test isolated behavior, but integration tests are still needed to verify that real components communicate correctly.

### Q9. What is `assert_called_once_with()`?

An assertion that verifies a mock was called exactly once with the specified arguments.

### Q10. What is a risk of over-mocking?

Tests can become coupled to implementation details and pass even when the real system integration is broken.

## 16. Quick Revision

- **Mocking:** Replace a dependency with a controlled test double.
- **`Mock()`:** Create a mock object.
- **`patch()`:** Temporarily replace a name or object.
- **`return_value`:** Define the mock's returned value.
- **`side_effect`:** Simulate exceptions or changing behavior.
- **`assert_called_once_with()`:** Verify a call and its arguments.
- **API testing:** Simulate responses without real network requests.
- **Retry testing:** Simulate failure followed by success.
- **ETL testing:** Mock extraction and loading while testing transformation logic.
- **Best practice:** Combine unit tests using mocks with integration tests using real components.

**For Data Engineer interviews, focus on:** `Mock`, `patch`, `return_value`, `side_effect`, call assertions, API testing, retry testing, and unit testing ETL pipelines.