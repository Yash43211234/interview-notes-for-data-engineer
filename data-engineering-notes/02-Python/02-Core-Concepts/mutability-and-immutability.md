# Mutability and Immutability in Python

## 1. Overview

In Python, **mutability** describes whether an object can be changed after it has been created.

- **Mutable objects:** Their contents can be changed without creating a different object.
- **Immutable objects:** Their value cannot be changed after creation. An operation that appears to modify an immutable object creates a new object instead.

Understanding this concept helps with debugging, function arguments, copying objects, memory references, and writing reliable data-processing code.

---

## 2. Mutable vs. Immutable Types

### Common mutable types

- `list`
- `dict`
- `set`
- Most user-defined class instances

### Common immutable types

- `int`
- `float`
- `bool`
- `str`
- `tuple` (with an important exception explained below)
- `frozenset`
- `bytes`

| Type | Mutable or immutable? | Example |
|---|---|---|
| `int` | Immutable | `x = 10` |
| `float` | Immutable | `x = 3.14` |
| `bool` | Immutable | `x = True` |
| `str` | Immutable | `x = "Python"` |
| `list` | Mutable | `x = [1, 2, 3]` |
| `dict` | Mutable | `x = {"a": 1}` |
| `set` | Mutable | `x = {1, 2, 3}` |
| `tuple` | Immutable container | `x = (1, 2, 3)` |
| `frozenset` | Immutable | `x = frozenset({1, 2})` |

---

## 3. Understanding Mutable Objects

A mutable object can be modified after it is created.

### Example: Modifying a list

```python
numbers = [10, 20, 30]

print(numbers)

numbers.append(40)

print(numbers)
```

Output:

```text
[10, 20, 30]
[10, 20, 30, 40]
```

The list was modified in place. It remains the same list object.

### Example: Modifying a dictionary

```python
employee = {
    "name": "Amit",
    "salary": 50000
}

employee["salary"] = 60000
employee["department"] = "Data Engineering"

print(employee)
```

Output:

```text
{'name': 'Amit', 'salary': 60000, 'department': 'Data Engineering'}
```

Both updating an existing value and adding a new key modify the dictionary in place.

### Example: Modifying a set

```python
numbers = {1, 2, 3}

numbers.add(4)
numbers.remove(2)

print(numbers)
```

Output:

```text
{1, 3, 4}
```

The set's contents changed. Remember that sets are unordered collections, so their displayed order is not guaranteed.

---

## 4. Understanding Immutable Objects

An immutable object cannot have its existing value changed.

### Example: Strings

```python
name = "Python"

name.upper()

print(name)
```

Output:

```text
Python
```

`upper()` returns a new string. It does not modify the original string.

To keep the uppercase result:

```python
name = name.upper()

print(name)
```

Output:

```text
PYTHON
```

The variable now refers to the uppercase string.

### Example: Integers

```python
x = 10

x = x + 5

print(x)
```

Output:

```text
15
```

It may look like the integer was modified, but it was not. The addition produces an integer value, and the variable `x` is assigned that result.

The original integer object, `10`, was not changed.

### Example: Tuples

```python
values = (10, 20, 30)

values[0] = 100
```

Output:

```text
TypeError
```

A tuple does not allow its elements to be reassigned.

---

## 5. Variables, Objects, and References

A key concept is that **variables refer to objects**. A variable is not a container that permanently owns a value.

### Example: Two variables refer to the same list

```python
a = [1, 2, 3]
b = a

b.append(4)

print(a)
print(b)
```

Output:

```text
[1, 2, 3, 4]
[1, 2, 3, 4]
```

Why?

1. `a` refers to a list object.
2. `b = a` makes `b` refer to the same object.
3. `b.append(4)` modifies that shared list.
4. Both variables show the updated contents.

Conceptually:

```text
a ──┐
    ├──> [1, 2, 3, 4]
b ──┘
```

This is called **aliasing**: multiple references point to the same object.

### Example: Reassigning a variable

```python
a = [1, 2, 3]
b = a

b = [10, 20, 30]

print(a)
print(b)
```

Output:

```text
[1, 2, 3]
[10, 20, 30]
```

Here, `b = [10, 20, 30]` does not modify the original list. It makes `b` refer to a different list.

This distinction between **mutating an object** and **reassigning a variable** is important in Python interviews.

---

## 6. Mutability and Function Arguments

Python passes object references to functions. When a function receives a reference to a mutable object, it can modify that object.

### Example: A function modifies a list

```python
def add_item(items):
    items.append(100)

numbers = [1, 2, 3]

add_item(numbers)

print(numbers)
```

Output:

```text
[1, 2, 3, 100]
```

The function modifies the same list object that `numbers` refers to.

### Example: A function cannot modify an integer in place

```python
def increase(number):
    number = number + 10

x = 5

increase(x)

print(x)
```

Output:

```text
5
```

The addition creates a new integer value, and the local variable `number` is reassigned. The caller's variable `x` remains unchanged.

### Important distinction

| Operation inside a function | Effect on caller |
|---|---|
| Mutate a shared list with `append()` | Original list changes |
| Add a key to a shared dictionary | Original dictionary changes |
| Reassign the local list variable | Caller’s reference is unchanged |
| Reassign a local integer variable | Caller’s variable is unchanged |
| Return a new object and assign it in the caller | Caller can use the returned object |

---

## 7. The Important Exception: Tuples Containing Mutable Objects

A tuple is immutable, but an element inside it can itself be mutable.

```python
data = ([1, 2], 3)

data[0].append(100)

print(data)
```

Output:

```text
([1, 2, 100], 3)
```

This works because:

- The tuple's elements cannot be replaced.
- The list stored in the first element is mutable.
- The list can change its contents without changing which object the tuple references.

However, replacing the list itself is not allowed:

```python
data[0] = [10, 20]
```

This raises `TypeError`.

**Interview takeaway:** A tuple is immutable as a container, but it does not guarantee that all objects contained inside it are immutable.

---

## 8. The Difference Between `==` and `is`

These operators answer different questions.

- `==` checks whether two objects have equal values.
- `is` checks whether two references point to the exact same object.

### Example

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)
print(a is b)
print(a is c)
```

Output:

```text
True
False
True
```

Explanation:

- `a == b` is `True` because their contents are equal.
- `a is b` is `False` because they are separate list objects.
- `a is c` is `True` because `c` refers to the same object as `a`.

Use `is None` when checking whether a value is `None`:

```python
value = None

if value is None:
    print("No value")
```

Do not use `is` as a general replacement for `==`. Python may reuse certain immutable objects internally, but you should not rely on object identity for value comparisons.

---

## 9. Shallow Copy vs. Deep Copy

Copying is especially important when working with nested mutable objects.

### Shallow copy

A shallow copy creates a new outer object but keeps references to the original nested objects.

```python
import copy

original = [[1, 2], [3, 4]]
shallow = copy.copy(original)

shallow.append([5, 6])
shallow[0].append(100)

print(original)
print(shallow)
```

Output:

```text
[[1, 2, 100], [3, 4]]
[[1, 2, 100], [3, 4], [5, 6]]
```

Explanation:

- `shallow` is a different outer list.
- Appending a new element to `shallow` does not change the outer list `original`.
- The nested list at index `0` is shared, so modifying it affects both.

For a list, the slice syntax `original[:]` also makes a shallow copy.

### Deep copy

A deep copy recursively copies nested objects, subject to the behavior of the objects involved.

```python
import copy

original = [[1, 2], [3, 4]]
deep = copy.deepcopy(original)

deep[0].append(100)

print(original)
print(deep)
```

Output:

```text
[[1, 2], [3, 4]]
[[1, 2, 100], [3, 4]]
```

The nested list in the deep copy is separate from the nested list in the original.

### Comparison

| Feature | Shallow copy | Deep copy |
|---|---|---|
| Creates a new outer object | Yes | Yes |
| Recursively copies nested objects | No | Generally yes |
| Shares nested mutable objects | Usually | Usually avoids sharing copied nested objects |
| Typical use | Simple, flat collections | Nested structures requiring isolation |
| Function | `copy.copy()` | `copy.deepcopy()` |

Deep copying can use more memory and processing time. Some objects also have special copying behavior, so deep copy should not be assumed to duplicate every possible resource.

---

## 10. Mutable Default Arguments

One of the most common Python interview mistakes is using a mutable object as a default function argument.

### Incorrect approach

```python
def add_item(item, items=[]):
    items.append(item)
    return items

print(add_item(1))
print(add_item(2))
print(add_item(3))
```

Output:

```text
[1]
[1, 2]
[1, 2, 3]
```

Why does this happen?

Default argument values are evaluated once when the function is defined, not every time it is called. The same list is reused across calls.

### Correct approach

```python
def add_item(item, items=None):
    if items is None:
        items = []

    items.append(item)
    return items

print(add_item(1))
print(add_item(2))
print(add_item(3))
```

Output:

```text
[1]
[2]
[3]
```

Each call creates a new list when no list is supplied.

**Rule:** Use `None` as the default when you need a fresh mutable object for each function call.

---

## 11. How to Check Object Identity and Mutability

### Check identity with `id()`

```python
numbers = [1, 2, 3]

print(id(numbers))

numbers.append(4)

print(id(numbers))
```

The identity remains the same when the list is modified in place.

However, do not rely on the numeric value of `id()` to remain unique forever: Python only guarantees identity uniqueness for objects whose lifetimes overlap.

### Check whether two variables refer to the same object

```python
a = [1, 2]
b = a
c = [1, 2]

print(a is b)  # True
print(a is c)  # False
```

There is no universal built-in function that reliably determines whether every Python object is mutable. You generally need to understand the object's type and its behavior.

---

## 12. Practical Data Engineering Applications

### Example 1: Avoid accidental modification of input data

```python
def clean_records(records):
    cleaned = records.copy()
    cleaned.append({"id": 4, "status": "valid"})
    return cleaned

records = [
    {"id": 1, "status": "valid"},
    {"id": 2, "status": "invalid"}
]

result = clean_records(records)

print(len(records))  # 2
print(len(result))   # 3
```

The shallow copy protects the outer list from the append operation.

However, the dictionaries inside `cleaned` are still shared with `records`. If the function changes an existing dictionary, that change will also be visible through the original list.

### Example 2: Avoid shared configuration changes

```python
default_config = {
    "format": "parquet",
    "compression": "snappy"
}

job_config = default_config.copy()
job_config["format"] = "delta"

print(default_config["format"])  # parquet
print(job_config["format"])      # delta
```

A shallow copy is sufficient here because the values are immutable strings. For nested mutable configuration values, consider whether a deeper copy is necessary.

### Example 3: Understand references to data

When several variables refer to the same mutable collection, a change through one reference may affect the others. In data pipelines, this can cause unexpected changes to intermediate records or shared configuration objects.

A good practice is to make copies deliberately when isolation is required, and avoid unnecessary copies when the data can safely be shared.

---

## 13. Common Interview Questions

### Q1. What is mutability in Python?

Mutability is the ability to change an object's contents or state after it has been created.

### Q2. Are strings mutable?

No. Strings are immutable. Operations such as `upper()` and `replace()` return strings rather than changing the original string.

### Q3. Are lists mutable?

Yes. Lists support in-place operations such as `append()`, `extend()`, and item assignment.

### Q4. Why does assigning one list to another variable not create a copy?

Because assignment binds another variable to the same object. Use a shallow or deep copy when an independent object is required.

### Q5. What is aliasing?

Aliasing occurs when multiple variables refer to the same object.

### Q6. What is the difference between `==` and `is`?

`==` compares values for equality, while `is` checks object identity.

### Q7. Can a tuple contain a list?

Yes. The tuple's references cannot be reassigned, but the contained list can still be modified.

### Q8. What is the difference between shallow and deep copy?

A shallow copy creates a new outer object but shares nested objects. A deep copy recursively copies nested objects, generally allowing changes to copied nested collections without affecting the originals.

### Q9. Why are mutable default arguments dangerous?

They are created once when the function is defined and then reused across calls. Use `None` and create the mutable object inside the function instead.

### Q10. Does passing a list to a function copy the list?

No. The function receives a reference to the same list object. It can mutate the list, but reassigning its local parameter does not reassign the caller's variable.

### Q11. Does `x = x + 1` modify an integer in place?

No. Integers are immutable. The operation produces a result, and the variable is rebound to it.

### Q12. Does a shallow copy always protect the original object?

No. It creates a separate outer object, but nested mutable objects may remain shared.

---

## 14. Practice Questions

Try predicting the output before running each snippet.

### Easy

**1.**

```python
a = [1, 2, 3]
b = a
b.append(4)

print(a)
```

**2.**

```python
a = "hello"
b = a
b = b.upper()

print(a)
print(b)
```

**3.**

```python
a = [1, 2]
b = [1, 2]

print(a == b)
print(a is b)
```

### Medium

**4.**

```python
data = ([1, 2], 3)
data[0].append(4)

print(data)
```

**5.**

```python
import copy

a = [[1, 2], [3, 4]]
b = copy.copy(a)

b[0].append(5)

print(a)
print(b)
```

**6.**

```python
def update(items):
    items = items + [4]

numbers = [1, 2, 3]
update(numbers)

print(numbers)
```

**7.**

```python
def add(value, items=[]):
    items.append(value)
    return items

print(add(10))
print(add(20))
```

### Answers

**1.**

```text
[1, 2, 3, 4]
```

Both variables reference the same list.

**2.**

```text
hello
HELLO
```

Strings are immutable, and `b` is rebound to the uppercase result.

**3.**

```text
True
False
```

The lists have equal contents but different identities.

**4.**

```text
([1, 2, 4], 3)
```

The tuple is immutable, but its contained list is mutable.

**5.**

```text
[[1, 2, 5], [3, 4]]
[[1, 2, 5], [3, 4]]
```

The shallow copy shares the nested lists.

**6.**

```text
[1, 2, 3]
```

The expression creates a new list and rebinds the local parameter. It does not modify the caller's list.

**7.**

```text
[10]
[10, 20]
```

The same default list is reused across calls.

---

## 15. Quick Revision

- **Mutable:** The object can be changed in place. Examples: `list`, `dict`, `set`.
- **Immutable:** The object's value cannot be changed after creation. Examples: `int`, `str`, `tuple`, `frozenset`.
- **Assignment:** Binds a variable to an object; it does not automatically copy the object.
- **Aliasing:** Multiple variables refer to the same object.
- **`==`:** Compares values.
- **`is`:** Compares identity.
- **Shallow copy:** Copies the outer object but can share nested objects.
- **Deep copy:** Recursively copies nested objects where supported.
- **Mutable defaults:** Avoid them when a fresh object is required for each call.
- **Tuple exception:** A tuple can contain mutable objects.
- **Function arguments:** Mutating a shared object affects that object; rebinding a local parameter does not rebind the caller's variable.

**Remember:** Mutability belongs to the object, while variable assignment determines which object a variable refers to.