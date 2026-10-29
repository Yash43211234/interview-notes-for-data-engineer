# Lambda, Map, and Filter in Python

## 1. Overview

Python provides `lambda`, `map()`, and `filter()` to write concise operations on data collections.

These are useful for:
- Transforming data.
- Filtering records based on conditions.
- Processing lists, tuples, and other iterables.
- Writing compact data-processing logic in Python and PySpark preparation.

| Feature | Purpose |
|---|---|
| `lambda` | Creates a small anonymous function |
| `map()` | Applies a function to every item |
| `filter()` | Keeps items that satisfy a condition |

---

## 2. Lambda Functions

### What is a lambda function?

A `lambda` function is a small, anonymous function defined using the `lambda` keyword.

**Syntax:**

```python
lambda arguments: expression
```

The expression is evaluated and its result is returned automatically. A lambda function contains only one expression, although that expression can be complex.

### Example 1: Normal function vs. lambda

Using a normal function:

```python
def square(n):
    return n * n

print(square(5))
```

Output:

```text
25
```

Using a lambda function:

```python
square = lambda n: n * n

print(square(5))
```

Output:

```text
25
```

Both produce the same result.

### Example 2: Multiple arguments

```python
add = lambda a, b: a + b

print(add(10, 20))
```

Output:

```text
30
```

A lambda can accept multiple arguments but still contains only one expression.

### Example 3: Conditional expression

```python
check = lambda n: "Even" if n % 2 == 0 else "Odd"

print(check(7))
print(check(8))
```

Output:

```text
Odd
Even
```

### Important points

- Lambda functions are anonymous functions, but you can assign them to variables.
- They can accept zero or more arguments.
- They return the result of their expression automatically.
- They cannot contain ordinary statements such as `for`, `while`, or `return`.
- Use `def` when the logic is complex or needs a meaningful function name and documentation.

---

## 3. The `map()` Function

### What does `map()` do?

`map()` applies a function to every item in an iterable and returns an iterator containing the results.

**Syntax:**

```python
map(function, iterable)
```

### Example 1: Square every number

```python
numbers = [1, 2, 3, 4, 5]

result = map(lambda n: n * n, numbers)

print(list(result))
```

Output:

```text
[1, 4, 9, 16, 25]
```

**How it works:**

1. `map()` takes each number from `numbers`.
2. The lambda function calculates its square.
3. `map()` produces the transformed values.
4. `list()` consumes the iterator and displays the results.

### Example 2: Convert strings to integers

```python
values = ["10", "20", "30"]

numbers = list(map(int, values))

print(numbers)
```

Output:

```text
[10, 20, 30]
```

Here, `int` is passed directly as the function. A lambda is not necessary.

### Example 3: Convert names to uppercase

```python
names = ["alice", "bob", "charlie"]

result = list(map(str.upper, names))

print(result)
```

Output:

```text
['ALICE', 'BOB', 'CHARLIE']
```

### Example 4: Apply a function with two iterables

`map()` can accept multiple iterables when the supplied function accepts the corresponding number of arguments.

```python
a = [1, 2, 3]
b = [10, 20, 30]

result = list(map(lambda x, y: x + y, a, b))

print(result)
```

Output:

```text
[11, 22, 33]
```

Values are processed position by position.

If the iterables have different lengths, `map()` stops when the shortest iterable is exhausted.

### Equivalent using a list comprehension

```python
numbers = [1, 2, 3, 4, 5]

result = [n * n for n in numbers]
```

For straightforward transformations, list comprehensions are often more readable than `map()` with a lambda.

---

## 4. The `filter()` Function

### What does `filter()` do?

`filter()` keeps only the items for which a function returns a truthy value.

**Syntax:**

```python
filter(function, iterable)
```

The function typically returns `True` or `False` for each item.

### Example 1: Filter even numbers

```python
numbers = [1, 2, 3, 4, 5, 6]

result = list(filter(lambda n: n % 2 == 0, numbers))

print(result)
```

Output:

```text
[2, 4, 6]
```

**How it works:**

- `1 % 2 == 0` → `False`, so `1` is excluded.
- `2 % 2 == 0` → `True`, so `2` is included.
- The same check is performed for every item.

### Example 2: Filter values greater than 10

```python
numbers = [5, 12, 8, 20, 15]

result = list(filter(lambda n: n > 10, numbers))

print(result)
```

Output:

```text
[12, 20, 15]
```

### Example 3: Filter names by length

```python
names = ["Amit", "Raj", "Priya", "An"]

result = list(filter(lambda name: len(name) > 3, names))

print(result)
```

Output:

```text
['Amit', 'Priya']
```

### Example 4: Filter missing or empty values

```python
values = ["data", "", None, "python", 0, False]

result = list(filter(None, values))

print(result)
```

Output:

```text
['data', 'python']
```

When `None` is passed as the function, `filter()` keeps truthy values and removes falsy values.

Falsy values include:

```python
False
None
0
0.0
""
[]
{}
set()
```

**Important:** This removes all falsy values, not just missing values. If `0` is a valid value in your data, use a specific condition instead.

### Equivalent using a list comprehension

```python
numbers = [1, 2, 3, 4, 5, 6]

result = [n for n in numbers if n % 2 == 0]
```

This is often the clearest way to express filtering in Python.

---

## 5. Combining `map()` and `filter()`

You can filter values first and then transform them.

### Example: Square only the even numbers

```python
numbers = [1, 2, 3, 4, 5, 6]

even_numbers = filter(lambda n: n % 2 == 0, numbers)

squares = map(lambda n: n * n, even_numbers)

print(list(squares))
```

Output:

```text
[4, 16, 36]
```

Execution flow:

1. `filter()` keeps `2`, `4`, and `6`.
2. `map()` squares these values.
3. `list()` collects the final results.

### Alternative: Use a list comprehension

```python
numbers = [1, 2, 3, 4, 5, 6]

result = [n * n for n in numbers if n % 2 == 0]

print(result)
```

Output:

```text
[4, 16, 36]
```

Both approaches produce the same result. The list comprehension is often easier to read.

---

## 6. `map()` vs. `filter()` vs. List Comprehension

| Feature | `map()` | `filter()` | List comprehension |
|---|---|---|---|
| Main purpose | Transform items | Select items | Transform and/or select |
| Changes item values? | Usually | No, it selects original items | Can |
| Uses a condition? | Not inherently | Usually | Optional |
| Returns | Iterator | Iterator | List |
| Lazy evaluation? | Yes | Yes | No, builds the list immediately |

### Example comparison

```python
numbers = [1, 2, 3, 4, 5]

# Transform
a = list(map(lambda n: n * 2, numbers))

# Filter
b = list(filter(lambda n: n > 3, numbers))

# Transform and filter
c = [n * 2 for n in numbers if n > 3]

print(a)  # [2, 4, 6, 8, 10]
print(b)  # [4, 5]
print(c)  # [8, 10]
```

**Interview takeaway:** Use `map()` for transformations, `filter()` for selection, and comprehensions when they express the logic more clearly.

---

## 7. Lazy Evaluation and Iterators

In Python 3, `map()` and `filter()` return iterators instead of lists.

```python
numbers = [1, 2, 3]

result = map(lambda n: n * 2, numbers)

print(result)
```

Output resembles:

```text
<map object at ...>
```

It does not display the transformed values because the iterator has not been consumed.

To see the values:

```python
print(list(result))
```

Output:

```text
[2, 4, 6]
```

### Iterators are consumed

```python
numbers = [1, 2, 3]

result = map(lambda n: n * 2, numbers)

print(list(result))
print(list(result))
```

Output:

```text
[2, 4, 6]
[]
```

After the first conversion, the iterator is exhausted. The second conversion has no remaining values.

If you need the results more than once, store them in a list:

```python
result = list(map(lambda n: n * 2, numbers))

print(result)
print(result)
```

### Why lazy evaluation matters

- It avoids building an entire output list immediately.
- It can reduce memory usage for large sequences when values are consumed incrementally.
- It allows transformations to be chained.
- It does not guarantee that the operation will be faster than a list comprehension.

---

## 8. Practical Data Engineering Examples

### Example 1: Clean numeric strings

```python
raw_amounts = ["100", "250", "300"]

amounts = list(map(int, raw_amounts))

print(amounts)
```

Output:

```text
[100, 250, 300]
```

This works when every input string contains a valid integer. Invalid strings raise `ValueError`.

### Example 2: Select valid positive amounts

```python
amounts = [100, -20, 0, 250, 500, -10]

valid_amounts = list(filter(lambda amount: amount > 0, amounts))

print(valid_amounts)
```

Output:

```text
[100, 250, 500]
```

### Example 3: Normalize customer names

```python
names = ["  yashwant ", " AMIT", "priya  "]

clean_names = list(map(lambda name: name.strip().title(), names))

print(clean_names)
```

Output:

```text
['Yashwant', 'Amit', 'Priya']
```

### Example 4: Filter records represented as dictionaries

```python
orders = [
    {"order_id": 1, "amount": 500},
    {"order_id": 2, "amount": 100},
    {"order_id": 3, "amount": 900},
]

large_orders = list(
    filter(lambda order: order["amount"] >= 500, orders)
)

print(large_orders)
```

Output:

```text
[
    {'order_id': 1, 'amount': 500},
    {'order_id': 3, 'amount': 900}
]
```

This is useful for understanding record filtering, although a SQL `WHERE` clause or a PySpark DataFrame filter is often preferable when data is stored in a database or distributed dataset.

### Example 5: Extract a field from each record

```python
orders = [
    {"order_id": 1, "amount": 500},
    {"order_id": 2, "amount": 100},
    {"order_id": 3, "amount": 900},
]

order_ids = list(map(lambda order: order["order_id"], orders))

print(order_ids)
```

Output:

```text
[1, 2, 3]
```

---

## 9. Common Mistakes

### Mistake 1: Forgetting to consume the iterator

```python
result = map(lambda n: n * 2, [1, 2, 3])
print(result)
```

This prints a `map` object, not the results.

Correct:

```python
print(list(result))
```

### Mistake 2: Using `map()` when you need filtering

Incorrect:

```python
numbers = [1, 2, 3, 4, 5]

result = list(map(lambda n: n > 3, numbers))
```

Output:

```text
[False, False, False, True, True]
```

`map()` transforms each number into a Boolean. It does not remove the numbers that fail the condition.

Correct:

```python
result = list(filter(lambda n: n > 3, numbers))
```

Output:

```text
[4, 5]
```

### Mistake 3: Using a lambda for complex logic

Avoid writing long, difficult-to-read lambda expressions. Use `def` for multi-step logic, validation, exception handling, or reusable business rules.

### Mistake 4: Assuming `filter()` returns Boolean values

`filter()` returns the original items that pass the condition, not the Boolean results of the condition.

### Mistake 5: Reusing an exhausted iterator

Once a `map` or `filter` iterator has been consumed, it does not automatically restart. Create a new iterator or store the output in a list if you need to reuse the results.

### Mistake 6: Ignoring invalid data

```python
values = ["10", "20", "abc"]

result = list(map(int, values))
```

This raises `ValueError` because `"abc"` cannot be converted to an integer. Real data pipelines should define how to handle malformed values, such as validating them, recording rejected rows, or using controlled error handling.

---

## 10. Interview Questions

### Q1. What is a lambda function?

A lambda is a small anonymous function defined using the `lambda` keyword. It contains a single expression and returns that expression's result.

### Q2. What is the difference between `map()` and `filter()`?

- `map()` applies a function to each item and produces transformed results.
- `filter()` selects items for which a function returns a truthy value.

### Q3. What does `map()` return in Python 3?

It returns a lazy iterator. Use `list()` to collect all results into a list.

### Q4. Does `filter()` modify the original list?

No. It produces an iterator over selected items. It does not modify the original list.

### Q5. Can `map()` accept multiple iterables?

Yes. The function must accept the corresponding number of arguments. Processing stops when the shortest iterable is exhausted.

### Q6. What is the difference between a lambda and a normal function?

A lambda contains one expression and has no declared function name by default. A function defined using `def` supports multiple statements, a docstring, and more complex logic.

### Q7. Why might a list comprehension be preferred?

It can be more readable, especially when combining transformations and conditions. It also returns a list immediately, whereas `map()` and `filter()` are lazy iterators.

### Q8. What does `filter(None, values)` do?

It keeps truthy values and removes falsy values, including `0`, `False`, `None`, and empty collections.

### Q9. What happens if you consume a `map()` iterator twice?

The second consumption returns no values because the iterator is exhausted.

### Q10. Should you use `map()` and `filter()` for all data engineering transformations?

No. They are useful Python tools, but for large datasets, prefer processing within the appropriate engine, such as SQL or PySpark DataFrames, rather than collecting distributed data into Python lists.

---

## 11. Practice Questions

Try solving these without looking at the solutions first.

### Easy

1. Use `map()` to multiply every number in `[2, 4, 6, 8]` by 3.
2. Use `filter()` to select numbers greater than 10 from `[5, 12, 8, 20, 15]`.
3. Use `map()` to convert `["1", "2", "3"]` into integers.
4. Use a lambda to return the cube of a number.
5. Use `filter()` to select names longer than five characters.

### Medium

6. Filter the even numbers from `[1, 2, 3, 4, 5, 6, 7, 8]` and then square them.
7. Given a list of dictionaries containing `employee` and `salary`, filter employees whose salary is at least `50000`.
8. Given two lists, use `map()` to add corresponding values.
9. Clean a list of strings by removing leading and trailing whitespace.
10. Explain why calling `list()` twice on the same `filter()` iterator may produce an empty list the second time.

### Solutions

```python
# 1
list(map(lambda n: n * 3, [2, 4, 6, 8]))

# 2
list(filter(lambda n: n > 10, [5, 12, 8, 20, 15]))

# 3
list(map(int, ["1", "2", "3"]))

# 4
cube = lambda n: n ** 3

# 5
list(filter(lambda name: len(name) > 5, names))

# 6
list(map(lambda n: n * n,
         filter(lambda n: n % 2 == 0,
                [1, 2, 3, 4, 5, 6, 7, 8])))

# 7
list(filter(lambda emp: emp["salary"] >= 50000, employees))

# 8
list(map(lambda x, y: x + y, list_a, list_b))

# 9
list(map(str.strip, strings))
```

For question 10, `filter()` returns an iterator. The first `list()` consumes it, leaving no values for the second call.

---

## 12. Quick Revision

- **`lambda`** creates a small anonymous function.
- **`map()`** transforms each item.
- **`filter()`** selects items based on a condition.
- **`map()` and `filter()` return iterators** in Python 3.
- Use `list()` when you need to materialize all results.
- Iterators are consumed as you iterate over them.
- Use list comprehensions when they make the code easier to read.
- Handle invalid data deliberately in production pipelines.
- For distributed data processing, use SQL or PySpark transformations when appropriate.

**Remember:** `map` transforms, `filter` selects, and `lambda` defines a compact function.