# Variables and Data Types in Python

## 1. Overview

Variables store references to objects, and data types describe the kind of values those objects represent.

```python
name = "Yashwant"
age = 26
salary = 90000.50
is_employed = True
```

Python determines the type automatically at runtime. You do not need to declare it explicitly.

```python
x = 10

print(x)        # 10
print(type(x))  # <class 'int'>
```

Python is **dynamically typed**: a variable can refer to objects of different types at different points in the program.

```python
x = 10
x = "Python"

print(x)        # Python
print(type(x))  # <class 'str'>
```

The variable name `x` now refers to a string rather than an integer.

---

## 2. Variables in Python

A variable is a name that refers to an object.

### Naming rules

Valid variable names:
```python
employee_name = "Amit"
age2 = 25
_total = 100
```

Invalid variable names:
```python
# 2age = 25          # Cannot start with a digit
# employee-name = "" # Hyphens are not allowed
# class = "Python"   # Reserved keyword
```

Rules to remember:

- Names can contain letters, digits, and underscores.
- Names cannot start with a digit.
- Names are case-sensitive: `age`, `Age`, and `AGE` are different.
- Python keywords such as `if`, `for`, and `class` cannot be used as variable names.
- Use descriptive names such as `customer_id` instead of unclear names such as `x`, except where short names are conventional.

Python conventionally uses `snake_case` for variable and function names.

### Assigning values

```python
a = 10
b = 20

a = b

print(a)  # 20
print(b)  # 20
```

The assignment `a = b` makes `a` refer to the object referenced by `b`. It does not erase or change `b`.

### Multiple assignment

```python
a, b, c = 10, 20, 30

print(a)  # 10
print(b)  # 20
print(c)  # 30
```

Assign the same value to multiple variables:

```python
x = y = z = 0
```

Be careful when assigning mutable objects this way:

```python
a = b = []

a.append(10)

print(a)  # [10]
print(b)  # [10]
```

Both names refer to the same list. To create independent lists, use:

```python
a = []
b = []
```

### Swapping variables

Python supports swapping without a temporary variable.

```python
a = 10
b = 20

a, b = b, a

print(a)  # 20
print(b)  # 10
```

---

## 3. Understanding Objects, Variables, and References

Python variables are names bound to objects; they are not fixed containers that permanently hold one type.

```python
a = [1, 2, 3]
b = a

b.append(4)

print(a)  # [1, 2, 3, 4]
print(b)  # [1, 2, 3, 4]
```

Both `a` and `b` refer to the same list object.

Compare this with reassignment:

```python
a = 10
b = a

b = 20

print(a)  # 10
print(b)  # 20
```

Reassigning `b` does not change the object referenced by `a`.

### Checking object identity

The `is` operator checks whether two references point to the same object.

```python
a = [1, 2]
b = a
c = [1, 2]

print(a == b)  # True
print(a is b)  # True

print(a == c)  # True
print(a is c)  # False
```

- `==` checks value equality.
- `is` checks object identity.

For ordinary value comparisons, use `==`. Use `is` when identity is what matters, especially when checking against `None`.

```python
result = None

if result is None:
    print("No result")
```

---

## 4. Built-in Data Types

Python has several built-in data types.

| Category | Type | Example |
|---|---|---|
| Numeric | `int` | `10`, `-5` |
| Numeric | `float` | `3.14`, `-0.5` |
| Numeric | `complex` | `2 + 3j` |
| Boolean | `bool` | `True`, `False` |
| Text | `str` | `"Python"` |
| Sequence | `list` | `[1, 2, 3]` |
| Sequence | `tuple` | `(1, 2, 3)` |
| Sequence | `range` | `range(5)` |
| Set | `set` | `{1, 2, 3}` |
| Set | `frozenset` | `frozenset({1, 2})` |
| Mapping | `dict` | `{"id": 1}` |
| Binary | `bytes` | `b"hello"` |
| Binary | `bytearray` | `bytearray(b"hi")` |
| Binary | `memoryview` | `memoryview(b"hi")` |
| Null value | `NoneType` | `None` |

For Data Engineering interviews, pay particular attention to numeric types, strings, booleans, collections, dictionaries, and `None`.

---

## 5. Numeric Types

### Integer (`int`)

Integers represent whole numbers.

```python
age = 26
count = 100
temperature = -5

print(type(age))  # <class 'int'>
```

Python integers can grow beyond the range of fixed-width integer types, limited by available memory.

### Float (`float`)

Floats represent numbers with fractional parts.

```python
price = 99.99
percentage = 0.75

print(type(price))  # <class 'float'>
```

Floating-point values use finite precision, so some decimal values cannot be represented exactly.

```python
print(0.1 + 0.2)
# 0.30000000000000004
```

For financial calculations requiring exact decimal arithmetic, consider the `decimal.Decimal` type.

### Complex (`complex`)

Complex numbers contain real and imaginary components.

```python
z = 2 + 3j

print(z.real)  # 2.0
print(z.imag)  # 3.0
```

Complex numbers are less common in everyday Data Engineering work but are part of Python's built-in numeric types.

### Numeric operators

```python
a = 10
b = 3

print(a + b)   # 13
print(a - b)   # 7
print(a * b)   # 30
print(a / b)   # 3.333...
print(a // b)  # 3
print(a % b)   # 1
print(a ** b)  # 1000
```

Important distinctions:

- `/` performs true division and returns a float.
- `//` performs floor division.
- `%` returns the remainder.
- `**` performs exponentiation.

Floor division rounds down, not simply toward zero:

```python
print(7 // 2)    # 3
print(-7 // 2)   # -4
```

---

## 6. Boolean Type (`bool`)

A Boolean represents either `True` or `False`.

```python
is_valid = True
has_error = False

print(type(is_valid))  # <class 'bool'>
```

Booleans are commonly used in conditions and data validation.

```python
age = 26

if age >= 18:
    print("Eligible")
else:
    print("Not eligible")
```

### Truthy and falsy values

Python allows many values to be evaluated as true or false in a condition.

Common falsy values include:

- `False`
- `None`
- Numeric zero: `0`, `0.0`, `0j`
- Empty strings: `""`
- Empty collections: `[]`, `()`, `{}`, `set()`

Most other values are truthy.

```python
name = ""

if not name:
    print("Name is empty")
```

This pattern is useful, but distinguish between an empty value and a missing value when your application needs to treat them differently.

---

## 7. String Type (`str`)

Strings represent text and are immutable.

```python
name = "Python"
city = 'Kolkata'

print(type(name))  # <class 'str'>
```

You can concatenate strings using `+`:

```python
first = "Data"
second = "Engineer"

role = first + " " + second

print(role)  # Data Engineer
```

Convert other values to strings when needed:

```python
age = 26

message = "Age: " + str(age)

print(message)  # Age: 26
```

For more readable formatting, use an f-string:

```python
age = 26

print(f"Age: {age}")
```

---

## 8. Collection Types

### List

An ordered, mutable collection that allows duplicates.

```python
numbers = [10, 20, 20, 30]

numbers.append(40)

print(numbers)  # [10, 20, 20, 30, 40]
```

### Tuple

An ordered, immutable collection that allows duplicates.

```python
coordinates = (10, 20)

print(coordinates[0])  # 10
```

### Set

A collection of unique, hashable elements.

```python
numbers = {10, 20, 20, 30}

print(len(numbers))  # 3
```

Sets do not support positional indexing.

### Dictionary (`dict`)

A mapping of keys to values.

```python
employee = {
    "id": 101,
    "name": "Amit",
    "role": "Data Engineer"
}

print(employee["name"])  # Amit
```

Dictionary keys must be hashable. Values can be of different types.

Dictionaries preserve insertion order in modern Python.

### Choosing the correct type

| Requirement | Suitable type |
|---|---|
| Store ordered, changeable values | `list` |
| Store a fixed sequence | `tuple` |
| Store unique values | `set` |
| Map identifiers to values | `dict` |
| Store text | `str` |
| Represent a missing value | `None` |

---

## 9. Mutable vs Immutable Types

This distinction is important in Python interviews.

**Mutable objects** can be changed after creation.

Examples:
- `list`
- `dict`
- `set`
- `bytearray`

**Immutable objects** cannot be changed after creation.

Examples:
- `int`
- `float`
- `bool`
- `str`
- `tuple` (its element references cannot be reassigned)
- `bytes`
- `frozenset`

A tuple can contain a mutable object, so its nested contents are not necessarily immutable.

```python
data = (1, [2, 3])

data[1].append(4)

print(data)  # (1, [2, 3, 4])
```

The tuple itself remains unchanged in structure, while its inner list changes.

### Why does this matter?

Consider two names referencing one list:

```python
a = [1, 2]
b = a

a.append(3)

print(b)  # [1, 2, 3]
```

The mutation is visible through both names because they reference the same object.

With an immutable integer:

```python
a = 10
b = a

a = a + 5

print(a)  # 15
print(b)  # 10
```

The expression creates a new integer object for `a` to reference; it does not mutate the original integer.

---

## 10. Type Checking and Type Conversion

### Using `type()`

```python
x = 10
print(type(x))  # <class 'int'>
```

### Using `isinstance()`

```python
x = 10

print(isinstance(x, int))  # True
print(isinstance(x, str))  # False
```

`isinstance()` is generally preferred when checking whether a value belongs to a type or one of its subclasses.

A Python-specific detail: `bool` is a subclass of `int`.

```python
print(isinstance(True, int))  # True
print(type(True) is int)      # False
```

### Explicit type conversion

```python
x = "100"
y = int(x)

print(y)        # 100
print(type(y))  # <class 'int'>
```

Common conversions:

```python
int("25")          # 25
float("3.14")      # 3.14
str(100)           # "100"
list((1, 2, 3))    # [1, 2, 3]
tuple([1, 2, 3])   # (1, 2, 3)
set([1, 2, 2, 3])  # {1, 2, 3}
```

### Invalid conversions

```python
# int("hello")  # ValueError
# int("3.14")   # ValueError
```

Convert a numeric string to a float first if decimal precision is not a concern:

```python
value = int(float("3.14"))

print(value)  # 3
```

This truncates the fractional part; it does not round to the nearest integer.

### Implicit conversion

Python sometimes converts numeric types automatically.

```python
a = 10
b = 2.5

result = a + b

print(result)        # 12.5
print(type(result))  # <class 'float'>
```

However, Python does not automatically convert every compatible-looking value.

```python
# result = "10" + 5  # TypeError
```

Convert explicitly when mixing strings and numbers.

---

## 11. `None` and `NoneType`

`None` represents the absence of a value or a missing result.

```python
result = None

print(result)        # None
print(type(result))  # <class 'NoneType'>
```

Check it using identity:

```python
if result is None:
    print("No result available")
```

Do not confuse `None` with:

- `0`: a numeric value
- `False`: a Boolean value
- `""`: an empty string
- `[]`: an empty list

All of these represent different values and have different types.

### Example: a function returning no explicit value

```python
def greet():
    print("Hello")

result = greet()

print(result)  # None
```

A function without an explicit `return` statement returns `None`.

---

## 12. Variable Scope

Scope determines where a variable name can be accessed.

### Local scope

A variable created inside a function is generally local to that function.

```python
def calculate():
    total = 100
    print(total)

calculate()  # 100

# print(total)  # NameError
```

### Global scope

A variable defined at the module level can be accessed inside a function.

```python
total = 100

def show_total():
    print(total)

show_total()  # 100
```

To reassign a global variable inside a function, use `global` when that behavior is intended.

```python
count = 0

def increment():
    global count
    count += 1

increment()

print(count)  # 1
```

In general, passing values as function arguments and returning results is often easier to test and maintain than relying on global mutable state.

---

## 13. Common Mistakes

### Mistake 1: Using `=` instead of `==`

```python
age = 26  # Assignment

print(age == 26)  # True: comparison
```

`=` assigns a value; `==` compares values.

### Mistake 2: Assuming type declarations are permanent

```python
x = 10
x = "hello"
```

This is valid Python. The name `x` can refer to objects of different types over time.

### Mistake 3: Confusing identity with equality

```python
a = [1, 2]
b = [1, 2]

print(a == b)  # True
print(a is b)  # False
```

Use `==` for value equality, not `is`.

### Mistake 4: Accidentally sharing a mutable object

```python
a = []
b = a

b.append(10)

print(a)  # [10]
```

If independent lists are required, create separate lists or make an appropriate copy.

### Mistake 5: Confusing conversion with formatting

```python
value = 100

print(str(value))  # "100"
print(f"{value}")  # "100"
```

Both produce strings, but `str()` converts a value while f-strings format text containing expressions.

### Mistake 6: Assuming every number is a float

```python
print(type(10))    # int
print(type(10.0))  # float
```

Python distinguishes integers and floating-point numbers.

---

## 14. Interview Practice

Try to predict the output before checking the answers.

### Easy

**Q1. What is the output?**

```python
x = 10
x = "Python"

print(type(x))
```

**Q2. What is the difference between `==` and `is`?**

**Q3. What is the output?**

```python
x = "100"
y = int(x)

print(y + 50)
```

**Q4. Which of these values are falsy?**

```python
0
""
[]
[0]
```

### Medium

**Q5. What is the output?**

```python
a = [1, 2]
b = a
b.append(3)

print(a)
```

**Q6. Explain the output.**

```python
a = [1, 2]
b = [1, 2]

print(a == b)
print(a is b)
```

**Q7. What is the output?**

```python
print(7 / 2)
print(7 // 2)
print(-7 // 2)
```

**Q8. What is the output?**

```python
x = 10

def change():
    x = 20

change()

print(x)
```

**Q9. Why does `isinstance(True, int)` return `True`?**

**Q10. What does a function return if it has no explicit `return` statement?**

### Answers

1. `<class 'str'>`
2. `==` checks value equality; `is` checks whether two references identify the same object.
3. `150`
4. `0`, `""`, and `[]` are falsy. `[0]` is truthy because the list is not empty.
5. `[1, 2, 3]`
6. `True`, then `False`. The lists have equal values but are different objects.
7. `3.5`, `3`, and `-4`.
8. `10`. The `x` inside `change()` is local.
9. Because `bool` is a subclass of `int` in Python.
10. `None`.

---

## 15. Quick Revision

- Python is dynamically typed.
- Variables are names that refer to objects.
- `type()` identifies an object's type; `isinstance()` checks type membership.
- `==` compares values; `is` checks identity.
- Lists, dictionaries, and sets are mutable.
- Integers, floats, strings, and bytes are immutable.
- Tuples are immutable in structure, but can contain mutable objects.
- Use `int()`, `float()`, and `str()` for explicit conversions.
- `None` represents the absence of a value; use `is None` to check it.
- `/` performs true division; `//` performs floor division.
- `bool` is a subclass of `int`.
- Local variables are defined within a function; global variables are defined at module scope.

**Interview takeaway:** Understand Python's dynamic typing, object references, mutability, type conversion, identity, and variable scope. These fundamentals help you reason about function behavior, data transformations, and bugs in Data Engineering pipelines.