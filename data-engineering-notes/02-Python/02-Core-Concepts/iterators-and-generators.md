# Iterators and Generators in Python

## 1. Overview

Iterators and generators allow Python to process values one at a time instead of requiring all values to be stored in memory at once.

They are especially useful in Data Engineering when processing large files, database records, API responses, or data in batches.

**Key difference:**
- **Iterable:** An object you can loop over, such as a list, tuple, or string.
- **Iterator:** An object that produces values one at a time using `next()`.
- **Generator:** A convenient way to create an iterator, usually with a function containing `yield` or a generator expression.

```python
numbers = [10, 20, 30]

for number in numbers:
    print(number)
```

A `for` loop obtains values from an iterator until there are no more values.

---

## 2. Understanding Iterables

An iterable is an object from which Python can obtain an iterator.

Common iterables include:

- Lists
- Tuples
- Strings
- Dictionaries
- Sets
- Files
- Ranges

Example:

```python
numbers = [10, 20, 30]

iterator = iter(numbers)

print(next(iterator))  # 10
print(next(iterator))  # 20
print(next(iterator))  # 30
```

### What does `iter()` do?

The built-in `iter()` function obtains an iterator from an iterable.

```python
numbers = [1, 2, 3]

iterator = iter(numbers)

print(type(numbers))   # <class 'list'>
print(type(iterator))  # <class 'list_iterator'>
```

A list is iterable, but it is not itself an iterator.

### Iterable vs. iterator

| Feature | Iterable | Iterator |
|---|---|---|
| Purpose | Provides a way to obtain an iterator | Produces values one at a time |
| `iter(obj)` | Usually returns an iterator | Returns itself |
| `next(obj)` | Not necessarily supported | Supported |
| Examples | List, tuple, string | List iterator, generator |
| Maintains iteration progress | Not necessarily | Yes |

A list can create multiple independent iterators:

```python
numbers = [10, 20, 30]

a = iter(numbers)
b = iter(numbers)

print(next(a))  # 10
print(next(a))  # 20
print(next(b))  # 10
```

Each iterator tracks its own position.

---

## 3. Understanding Iterators

An iterator produces the next value whenever `next()` is called.

```python
numbers = [10, 20, 30]
iterator = iter(numbers)

print(next(iterator))  # 10
print(next(iterator))  # 20
print(next(iterator))  # 30
```

When no values remain, Python raises `StopIteration`.

```python
numbers = [10]
iterator = iter(numbers)

print(next(iterator))  # 10

# print(next(iterator))  # StopIteration
```

`StopIteration` is the signal that an iterator has been exhausted.

### How a `for` loop works

Conceptually, Python handles a loop like this:

```python
numbers = [10, 20, 30]
iterator = iter(numbers)

while True:
    try:
        number = next(iterator)
        print(number)
    except StopIteration:
        break
```

Output:

```text
10
20
30
```

You normally use a `for` loop rather than writing this logic manually. Understanding the mechanism helps explain how Python iteration works internally.

---

## 4. Creating a Custom Iterator

You can build an iterator by defining a class that implements two methods:

- `__iter__()` returns the iterator object.
- `__next__()` returns the next value or raises `StopIteration` when exhausted.

Example:

```python
class CountUp:
    def __init__(self, limit):
        self.current = 1
        self.limit = limit

    def __iter__(self):
        return self

    def __next__(self):
        if self.current > self.limit:
            raise StopIteration

        value = self.current
        self.current += 1
        return value


counter = CountUp(3)

for number in counter:
    print(number)
```

Output:

```text
1
2
3
```

### How it works

1. `CountUp(3)` creates an object with `current = 1`.
2. The `for` loop calls `iter(counter)`.
3. `__iter__()` returns the same iterator object.
4. Each call to `__next__()` returns a value and advances the counter.
5. After returning `3`, the next call raises `StopIteration`.

**Important:** This custom iterator is single-use. After it is exhausted, iterating over the same instance again produces no values unless you explicitly implement reset behavior.

---

## 5. Generators

A generator is a special kind of iterator that Python creates for you.

Generators are commonly written using:
1. A function containing `yield`.
2. A generator expression.

### Generator function

A normal function uses `return` to return a result and finish its execution.

A generator function uses `yield` to produce a value and suspend its execution.

```python
def count_up():
    yield 1
    yield 2
    yield 3

numbers = count_up()

print(next(numbers))  # 1
print(next(numbers))  # 2
print(next(numbers))  # 3
```

Calling `count_up()` does not immediately execute the entire function. It creates a generator object.

The function's body begins executing when you request the first value.

### `return` vs. `yield`

| `return` | `yield` |
|---|---|
| Returns a result and exits the function | Produces a value and suspends execution |
| Does not resume that function call | Execution can resume from the suspension point |
| Ordinary function call returns its result | Generator function call returns a generator object |

Example:

```python
def normal_function():
    return 10

def generator_function():
    yield 10

print(normal_function())          # 10
print(generator_function())       # Generator object representation
print(next(generator_function())) # 10
```

The exact memory address shown when printing a generator object is not fixed.

---

## 6. How `yield` Works

When a generator reaches `yield`, it returns a value and pauses. Its local variables and execution position are preserved for the next request.

```python
def generate_numbers():
    print("Starting")
    yield 10

    print("Resuming")
    yield 20

    print("Finished")


gen = generate_numbers()

print("Generator created")
print(next(gen))
print(next(gen))
```

Output:

```text
Generator created
Starting
10
Resuming
20
```

Notice that `Starting` does not appear when the generator is created. The generator begins execution when `next(gen)` is called.

The function resumes after each `yield` and continues until it reaches another `yield`, returns, or raises an exception.

---

## 7. Why Generators Are Memory-Efficient

Consider generating one million integers.

### Using a list

```python
numbers = [x for x in range(1_000_000)]
```

This creates a list containing one million integer references and the associated integer objects.

### Using a generator expression

```python
numbers = (x for x in range(1_000_000))
```

This creates a generator that produces values on demand instead of storing the complete sequence in a list.

```python
numbers = (x for x in range(1_000_000))

print(next(numbers))  # 0
print(next(numbers))  # 1
print(next(numbers))  # 2
```

| List comprehension | Generator expression |
|---|---|
| `[x for x in items]` | `(x for x in items)` |
| Builds a list immediately | Produces values lazily |
| Stores the resulting collection | Keeps the generator's execution state and produces values as requested |
| Can be indexed | Cannot be indexed directly |
| Can be iterated over repeatedly | A generator object is normally consumed once |

**Interview takeaway:** Generators reduce the need to store an entire result collection in memory. They do not automatically make every computation faster.

---

## 8. Generator Expressions

A generator expression resembles a list comprehension but uses parentheses.

```python
numbers = (x * x for x in range(5))

for number in numbers:
    print(number)
```

Output:

```text
0
1
4
9
16
```

Compare the syntax:

```python
squares_list = [x * x for x in range(5)]
squares_gen = (x * x for x in range(5))
```

- `squares_list` is a list.
- `squares_gen` is a generator object.

### Generator expressions with conditions

```python
even_numbers = (x for x in range(10) if x % 2 == 0)

print(list(even_numbers))
# [0, 2, 4, 6, 8]
```

The generator produces values lazily, but `list()` consumes it and stores all produced values in a list.

### Using generators with built-in functions

Generators work well with functions that consume iterables:

```python
numbers = (x * x for x in range(1_000_000))

total = sum(numbers)

print(total)
```

`sum()` consumes the generator one value at a time. The complete list of squares is not created.

---

## 9. Generator State and Exhaustion

A generator object maintains its own progress.

```python
def generate_numbers():
    yield 10
    yield 20
    yield 30

gen = generate_numbers()

print(next(gen))  # 10
print(next(gen))  # 20
print(list(gen))  # [30]
print(list(gen))  # []
```

After the generator is exhausted, it does not restart automatically.

If you need a fresh iteration, call the generator function again:

```python
gen1 = generate_numbers()
gen2 = generate_numbers()

print(list(gen1))  # [10, 20, 30]
print(list(gen2))  # [10, 20, 30]
```

Each function call creates a new generator object.

**Common mistake:** A generator is not a reusable container like a list. Once consumed, its values are no longer available from that generator.

---

## 10. Using `yield` in Data Processing

Generators are useful when processing data incrementally.

### Example 1: Processing records in batches

```python
def batches(records, batch_size):
    for start in range(0, len(records), batch_size):
        yield records[start:start + batch_size]


records = list(range(1, 11))

for batch in batches(records, 3):
    print(batch)
```

Output:

```text
[1, 2, 3]
[4, 5, 6]
[7, 8, 9]
[10]
```

This example yields batches rather than one record at a time. However, the original `records` list is already stored in memory.

For a large or streaming source, a batching function can consume an iterator incrementally instead.

### Example 2: Batching an arbitrary iterable

```python
from itertools import islice

def batches(iterable, batch_size):
    iterator = iter(iterable)

    while batch := list(islice(iterator, batch_size)):
        yield batch


records = (x for x in range(1, 11))

for batch in batches(records, 3):
    print(batch)
```

Output:

```text
[1, 2, 3]
[4, 5, 6]
[7, 8, 9]
[10]
```

This implementation:
- Accepts any iterable.
- Reads at most `batch_size` elements for each batch.
- Does not require the complete input to be stored in memory.
- Produces one batch at a time.

Each yielded batch is still stored as a list, so memory use is proportional to the batch size rather than the entire input.

The assignment expression `:=` is available in Python 3.8 and later.

### Example 3: Reading a file line by line

```python
def read_records(file_path):
    with open(file_path, "r", encoding="utf-8") as file:
        for line in file:
            yield line.strip()


# for record in read_records("orders.csv"):
#     print(record)
```

File objects are iterable. This pattern processes one line at a time rather than loading the entire file with `read()`.

For actual CSV data, use Python's `csv` module or an appropriate data-processing library, because splitting lines manually does not handle all CSV quoting rules.

### Example 4: Transforming records lazily

```python
def valid_orders(orders):
    for order in orders:
        if order["amount"] > 0:
            yield order


orders = [
    {"order_id": 1, "amount": 100},
    {"order_id": 2, "amount": -20},
    {"order_id": 3, "amount": 250},
]

for order in valid_orders(orders):
    print(order)
```

The function yields only valid records as they are requested.

This can be useful when chaining filtering and transformation steps, provided each stage is designed to consume iterables correctly.

---

## 11. Generator Pipelines

Generators can be chained together to process data in stages without materializing every intermediate result.

```python
def read_numbers():
    for number in range(1, 11):
        yield number


def filter_even(numbers):
    for number in numbers:
        if number % 2 == 0:
            yield number


def square(numbers):
    for number in numbers:
        yield number * number


pipeline = square(filter_even(read_numbers()))

print(list(pipeline))
# [4, 16, 36, 64, 100]
```

The stages are:

1. `read_numbers()` produces integers.
2. `filter_even()` keeps even integers.
3. `square()` squares the filtered values.
4. `list()` consumes the pipeline and collects the results.

Each stage requests values from the preceding stage as needed.

This resembles a lazy ETL pipeline, although a Python generator pipeline is not the same as a distributed processing engine such as Spark.

---

## 12. `yield from`

`yield from` delegates iteration to another iterable or generator.

Without `yield from`:

```python
def combined():
    for value in [1, 2, 3]:
        yield value

    for value in [4, 5]:
        yield value
```

With `yield from`:

```python
def combined():
    yield from [1, 2, 3]
    yield from [4, 5]


print(list(combined()))
# [1, 2, 3, 4, 5]
```

It is especially useful for composing generators.

### Example: Flattening nested lists

```python
def flatten(nested):
    for items in nested:
        yield from items


data = [[1, 2], [3, 4], [5]]

print(list(flatten(data)))
# [1, 2, 3, 4, 5]
```

This example flattens one level of nesting. Recursive nesting requires a recursive implementation if all levels must be flattened.

---

## 13. Generator Methods: `send()` and `close()`

Generators support additional operations beyond `next()`.

### `send()`

`send()` resumes a generator and sends a value back into it. The value becomes the result of the suspended `yield` expression.

```python
def receiver():
    value = yield "Ready"
    yield value


gen = receiver()

print(next(gen))       # Ready
print(gen.send(100))   # 100
```

The first `next()` starts the generator and reaches `yield "Ready"`. The next call, `send(100)`, makes the expression `value = yield "Ready"` evaluate to `100`.

You generally must start a generator with `next()` or `send(None)` before sending a non-`None` value.

### `close()`

`close()` requests that the generator stop by injecting `GeneratorExit` at its suspension point.

```python
def generate():
    try:
        yield 1
        yield 2
    finally:
        print("Cleaning up")


gen = generate()

print(next(gen))  # 1
gen.close()       # Cleaning up
```

This allows cleanup code in `finally` blocks to execute when a generator is closed.

These methods are useful to know for interviews, though everyday generator usage mainly relies on `yield`, `next()`, and iteration.

---

## 14. Iterators vs. Generators

| Feature | Custom iterator | Generator |
|---|---|---|
| How it is created | Usually a class implementing `__iter__()` and `__next__()` | A function containing `yield` or a generator expression |
| State management | Explicitly implemented | Managed by Python |
| Produces values incrementally | Yes | Yes |
| Raises `StopIteration` when exhausted | Yes | Yes; generator functions handle normal completion |
| Typical code length | More boilerplate | Usually shorter |
| Can represent an infinite sequence | Yes | Yes |

Use a custom iterator when you need a class with explicit iteration behavior or additional state-management features. Use a generator when you want a straightforward way to produce a sequence of values lazily.

---

## 15. Common Mistakes

### Mistake 1: Calling `next()` on a list

Incorrect:

```python
numbers = [1, 2, 3]

# next(numbers)  # TypeError
```

Correct:

```python
numbers = [1, 2, 3]
iterator = iter(numbers)

print(next(iterator))  # 1
```

### Mistake 2: Expecting a generator to return all values at once

```python
def numbers():
    yield 1
    yield 2
    yield 3

gen = numbers()

print(gen)  # A generator object, not a list
```

Use `list(gen)` if you specifically need to materialize all remaining values.

### Mistake 3: Reusing an exhausted generator

```python
gen = (x for x in range(3))

print(list(gen))  # [0, 1, 2]
print(list(gen))  # []
```

Create a new generator when you need to iterate again.

### Mistake 4: Accidentally consuming a generator

```python
gen = (x for x in range(5))

print(sum(gen))   # 10
print(list(gen))  # []
```

`sum()` consumes the generator. The second operation sees no remaining values.

### Mistake 5: Assuming lazy processing always saves memory

```python
gen = (x for x in range(1_000_000))
values = list(gen)
```

The generator is lazy, but converting it to a list materializes the full result.

### Mistake 6: Ignoring the source's memory usage

```python
records = load_entire_file_into_list()

for record in records:
    yield record
```

The generator yields records one at a time, but the entire source list is already in memory. For genuinely incremental processing, the source must also support incremental reading.

---

## 16. Interview Practice

Try answering these questions before looking at the answers.

### Easy

**Q1. What is the difference between an iterable and an iterator?**

**Q2. What does `iter()` do?**

**Q3. What happens when an iterator has no more values?**

**Q4. What is the difference between `return` and `yield`?**

### Medium

**Q5. Predict the output.**

```python
def generate():
    yield 10
    yield 20

gen = generate()

print(next(gen))
print(next(gen))
```

**Q6. What is the output?**

```python
gen = (x * 2 for x in range(4))

print(next(gen))
print(list(gen))
```

**Q7. Why is this generator memory-efficient?**

```python
def numbers(limit):
    for i in range(limit):
        yield i
```

**Q8. What happens when you iterate over an exhausted generator?**

**Q9. Explain the difference between a generator expression and a list comprehension.**

**Q10. How would you process a large text file without loading it entirely into memory?**

### Answers

1. An iterable can provide an iterator; an iterator produces successive values and tracks its progress.
2. It obtains an iterator from an iterable.
3. It raises `StopIteration`.
4. `return` exits a function with a result; `yield` produces a value and suspends a generator.
5. `10`, then `20`.
6. `0`, then `[2, 4, 6]`. The first value was consumed by `next()`.
7. It generates each integer on demand rather than constructing a list of all integers.
8. It produces no more values; normal iteration ends.
9. A list comprehension builds a list immediately, while a generator expression produces values lazily.
10. Iterate over the file object line by line, or use an appropriate streaming reader.

---

## 17. Quick Revision

- An iterable can provide an iterator.
- `iter()` obtains an iterator; `next()` requests its next value.
- `StopIteration` signals that an iterator is exhausted.
- A generator is an iterator created using `yield` or a generator expression.
- `yield` produces a value and suspends execution.
- Generators preserve their local state between yielded values.
- Generator expressions use parentheses; list comprehensions use square brackets.
- Generators are generally consumed once.
- `yield from` delegates iteration to another iterable or generator.
- Generators are useful for streaming files, processing records in batches, and building lazy transformation pipelines.
- Lazy processing reduces the need to materialize the complete output, but it does not guarantee that the input source itself is memory-efficient.
- In Data Engineering, assess the entire pipeline: source reading, transformations, buffering, and final output materialization.

**Interview takeaway:** Be able to explain `iter()` and `next()`, implement a simple generator, describe lazy evaluation and memory efficiency, and identify when a generator is exhausted.