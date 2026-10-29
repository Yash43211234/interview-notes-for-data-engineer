# References and Copying in Python

## 1. Overview

In Python, variables refer to objects. When you assign one variable to another, Python does not automatically create a copy of the object. Both variables may refer to the same object.

Understanding references and copying helps you avoid unexpected changes when working with lists, dictionaries, nested data structures, function arguments, and data-processing pipelines.

Key concepts:

- **Reference:** A way to access an object.
- **Assignment:** Binds a variable to an object.
- **Aliasing:** Multiple variables refer to the same object.
- **Shallow copy:** Creates a new outer object but shares nested objects.
- **Deep copy:** Recursively copies nested objects where supported.

---

## 2. Variables Refer to Objects

Consider this example:

```python
a = [10, 20, 30]
b = a

b.append(40)

print(a)
print(b)
```

Output:

```text
[10, 20, 30, 40]
[10, 20, 30, 40]
```

Why does this happen?

1. `a` refers to a list object.
2. `b = a` makes `b` refer to that same list.
3. `b.append(40)` modifies the shared list.
4. Both variables show the updated contents.

Conceptually:

```text
a ──┐
    ├──> [10, 20, 30, 40]
b ──┘
```

The statement `b = a` creates another reference, not another list.

### Check object identity

```python
a = [1, 2, 3]
b = a

print(a is b)
print(id(a) == id(b))
```

Output:

```text
True
True
```

- `is` checks whether two references point to the same object.
- `id()` returns an object's identity for its lifetime.

Use `is` when checking identity, not as a general substitute for `==`, which compares values for equality.

---

## 3. Assignment vs. Reassignment vs. Mutation

These three operations are different.

### Assignment

```python
a = [1, 2, 3]
b = a
```

Both variables refer to the same list.

### Mutation

```python
b.append(4)

print(a)
```

Output:

```text
[1, 2, 3, 4]
```

The shared object changes.

### Reassignment

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

Reassignment makes `b` refer to a different list. It does not modify the original list.

**Interview takeaway:** Mutation changes an object; reassignment changes what a variable refers to.

---

## 4. Shallow Copy

A shallow copy creates a new outer object while retaining references to the original object's nested elements.

For a flat list of immutable values, a shallow copy is often sufficient.

### Example

```python
original = [10, 20, 30]
copied = original.copy()

copied.append(40)

print(original)
print(copied)
```

Output:

```text
[10, 20, 30]
[10, 20, 30, 40]
```

The two outer lists are different objects.

```python
print(original is copied)
```

Output:

```text
False
```

### Common ways to create a shallow copy

For lists:

```python
copied = original.copy()
copied = original[:]
```

Using the `copy` module:

```python
import copy

copied = copy.copy(original)
```

For dictionaries:

```python
copied = original_dict.copy()
```

These methods create a shallow copy, not a recursive copy.

---

## 5. The Problem with Nested Objects

Shallow copying becomes important when a collection contains other mutable collections.

### Example

```python
original = [[1, 2], [3, 4]]
copied = original.copy()

copied[0].append(100)

print(original)
print(copied)
```

Output:

```text
[[1, 2, 100], [3, 4]]
[[1, 2, 100], [3, 4]]
```

Why did the original change?

- `original` and `copied` are different outer lists.
- Both lists refer to the same nested lists.
- Modifying `copied[0]` therefore modifies the nested list shared with `original`.

Conceptually:

```text
original ──> [ ──┐ , ──┐ ]
                │     │
copied   ──> [ ──┘ , ──┘ ]
                │     │
                v     v
             [1,2]   [3,4]
```

The diagram represents shared nested objects; after the append, the first nested list contains `[1, 2, 100]`.

### Changing the outer list behaves differently

```python
original = [[1, 2], [3, 4]]
copied = original.copy()

copied.append([5, 6])

print(original)
print(copied)
```

Output:

```text
[[1, 2], [3, 4]]
[[1, 2], [3, 4], [5, 6]]
```

Appending to the copied outer list does not change the original outer list.

---

## 6. Deep Copy

A deep copy recursively copies nested objects where supported, helping separate nested mutable data from the original.

Use `copy.deepcopy()` from Python's `copy` module.

### Example

```python
import copy

original = [[1, 2], [3, 4]]
copied = copy.deepcopy(original)

copied[0].append(100)

print(original)
print(copied)
```

Output:

```text
[[1, 2], [3, 4]]
[[1, 2, 100], [3, 4]]
```

The nested list in `copied` is separate from the nested list in `original`.

### How to create a deep copy

```python
import copy

copied = copy.deepcopy(original)
```

Deep copying can handle many nested structures and uses an internal memoization mechanism to preserve shared references and handle recursive object graphs.

However, some objects have special copying behavior. Deep copy should not be assumed to duplicate every resource, such as an external connection, in a meaningful way.

### Shallow copy vs. deep copy

| Feature | Shallow copy | Deep copy |
|---|---|---|
| New outer object | Yes | Yes |
| Recursively copies nested objects | No | Generally yes |
| Shares nested mutable objects | Usually | Usually avoids sharing copied nested objects |
| Typical memory cost | Lower | Potentially higher |
| Function | `copy.copy()` | `copy.deepcopy()` |

---

## 7. Copying Dictionaries

Dictionaries are mutable. Copying a dictionary requires the same distinction between shallow and deep copying.

### Shallow copy example

```python
employee = {
    "name": "Amit",
    "skills": ["Python", "SQL"]
}

copied = employee.copy()

copied["name"] = "Priya"
copied["skills"].append("PySpark")

print(employee)
print(copied)
```

Output:

```text
{'name': 'Amit', 'skills': ['Python', 'SQL', 'PySpark']}
{'name': 'Priya', 'skills': ['Python', 'SQL', 'PySpark']}
```

Explanation:

- Changing `copied["name"]` affects only the copied dictionary's key-value association.
- Appending to `copied["skills"]` modifies a shared nested list.

### Deep copy example

```python
import copy

employee = {
    "name": "Amit",
    "skills": ["Python", "SQL"]
}

copied = copy.deepcopy(employee)

copied["skills"].append("PySpark")

print(employee["skills"])
print(copied["skills"])
```

Output:

```text
['Python', 'SQL']
['Python', 'SQL', 'PySpark']
```

The nested lists are separate.

---

## 8. References and Function Arguments

Python passes object references to functions. A function can mutate a mutable object passed to it, but rebinding a local parameter does not rebind the caller's variable.

### Example 1: Mutation inside a function

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

The function modifies the same list object.

### Example 2: Reassignment inside a function

```python
def update_items(items):
    items = [10, 20, 30]

numbers = [1, 2, 3]

update_items(numbers)

print(numbers)
```

Output:

```text
[1, 2, 3]
```

The parameter `items` is rebound locally. The caller's variable `numbers` still refers to the original list.

### Example 3: Returning a new list

```python
def update_items(items):
    items = items + [4]
    return items

numbers = [1, 2, 3]
numbers = update_items(numbers)

print(numbers)
```

Output:

```text
[1, 2, 3, 4]
```

The `+` operation creates a new list, and the caller explicitly assigns the returned list to `numbers`.

---

## 9. Copying Immutable Objects

Immutable objects, such as strings and integers, cannot have their values changed in place.

```python
a = "Python"
b = a

b = b.upper()

print(a)
print(b)
```

Output:

```text
Python
PYTHON
```

The original string remains unchanged.

Copying an immutable object is usually unnecessary for protecting its value because it cannot be mutated. Some copy operations may return the same immutable object rather than creating a new one.

Remember that a tuple can contain mutable objects:

```python
data = ([1, 2], 3)
```

The tuple's references cannot be reassigned, but its nested list can still be modified.

---

## 10. Common Mistakes

### Mistake 1: Assuming assignment creates a copy

Incorrect assumption:

```python
a = [1, 2, 3]
b = a

b.append(4)
```

Both variables refer to the same list.

Use `a.copy()` or another suitable copy method when you need a separate outer list.

### Mistake 2: Assuming a shallow copy separates nested objects

```python
a = [[1, 2], [3, 4]]
b = a.copy()
```

The nested lists are shared. Use `copy.deepcopy(a)` if nested mutable objects must be separated.

### Mistake 3: Using deep copy everywhere

Deep copy can consume additional memory and processing time. Use it only when you need independent nested structures and the object's copying behavior supports that requirement.

### Mistake 4: Confusing equality and identity

```python
a = [1, 2]
b = [1, 2]

print(a == b)  # True
print(a is b)  # False
```

Equal contents do not mean identical objects.

### Mistake 5: Modifying a shared object unexpectedly

If multiple parts of a program refer to the same list or dictionary, a mutation in one place can affect other parts of the program. Be deliberate about ownership and copying.

---

## 11. Practical Data Engineering Examples

### Example 1: Preserve the original records

```python
records = [
    {"id": 1, "status": "valid"},
    {"id": 2, "status": "invalid"}
]

filtered_records = [
    record.copy()
    for record in records
    if record["status"] == "valid"
]

filtered_records[0]["status"] = "processed"

print(records[0]["status"])
print(filtered_records[0]["status"])
```

Output:

```text
valid
processed
```

Each selected dictionary is copied, so updating its top-level `status` field does not modify the original dictionary.

### Example 2: Understand nested data

```python
records = [
    {
        "id": 1,
        "metadata": {"source": "api"}
    }
]

copied = [record.copy() for record in records]

copied[0]["metadata"]["source"] = "file"

print(records[0]["metadata"]["source"])
```

Output:

```text
file
```

The outer dictionaries were copied, but the nested `metadata` dictionaries remain shared.

If nested isolation is required:

```python
import copy

copied = copy.deepcopy(records)
```

### Example 3: Avoid unnecessary copying

If a function only reads data and does not modify it, making a deep copy may be unnecessary.

```python
def calculate_total(orders):
    return sum(order["amount"] for order in orders)
```

This function reads the order records without modifying them. A copy is not needed merely to calculate the total.

For large datasets, unnecessary Python-level copying can increase memory use. In data engineering, prefer explicit and efficient transformations using the relevant engine, such as SQL or PySpark DataFrames, rather than copying large collections without a clear reason.

---

## 12. Interview Questions

### Q1. What happens when you write `b = a`?

The variable `b` is bound to the same object as `a`. No copy is created by the assignment itself.

### Q2. What is aliasing?

Aliasing occurs when multiple variables refer to the same object.

### Q3. What is the difference between shallow and deep copy?

A shallow copy creates a new outer object but shares nested objects. A deep copy recursively copies nested objects where supported.

### Q4. Why does modifying a nested list in a shallow copy affect the original?

Both outer lists contain references to the same nested list, so a mutation through either reference changes that shared object.

### Q5. What is the difference between `is` and `==`?

`is` checks object identity, while `==` checks value equality.

### Q6. Does passing a list to a function create a copy?

No. The function receives a reference to the same object. Mutating the list affects the shared object.

### Q7. Why does reassigning a function parameter not change the caller's variable?

The parameter is a local variable. Rebinding it changes the local reference, not the caller's reference.

### Q8. When should you use `copy.deepcopy()`?

Use it when nested mutable structures need to be independent of the original and the objects involved support meaningful deep copying.

### Q9. Is a shallow copy enough for a list of integers?

Usually, yes. Integers are immutable, so sharing references to them does not create a risk of modifying their values in place.

### Q10. Can deep copy preserve shared references within the copied structure?

Yes. Python's deep-copy mechanism uses memoization to handle repeated references and recursive structures.

---

## 13. Practice Questions

Predict the output before checking the answers.

### Easy

**1.**

```python
a = [1, 2, 3]
b = a
b[0] = 100

print(a)
```

**2.**

```python
a = [1, 2, 3]
b = a.copy()
b.append(4)

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
a = [[1, 2], [3, 4]]
b = a.copy()

b[1].append(5)

print(a)
```

**5.**

```python
import copy

a = [[1, 2], [3, 4]]
b = copy.deepcopy(a)

b[0].append(100)

print(a)
print(b)
```

**6.**

```python
def change(values):
    values = values + [4]

numbers = [1, 2, 3]
change(numbers)

print(numbers)
```

**7.**

```python
employee = {
    "name": "Amit",
    "skills": ["Python", "SQL"]
}

copied = employee.copy()
copied["name"] = "Priya"
copied["skills"].append("PySpark")

print(employee)
```

### Answers

**1.**

```text
[100, 2, 3]
```

Both variables refer to the same list.

**2.**

```text
[1, 2, 3]
[1, 2, 3, 4]
```

The outer lists are separate.

**3.**

```text
True
False
```

The lists are equal but not identical.

**4.**

```text
[[1, 2], [3, 4, 5]]
```

The nested list is shared by the shallow copy and original.

**5.**

```text
[[1, 2], [3, 4]]
[[1, 2, 100], [3, 4]]
```

The deep copy separates the nested lists.

**6.**

```text
[1, 2, 3]
```

The function rebinds its local parameter to a new list.

**7.**

```text
{'name': 'Amit', 'skills': ['Python', 'SQL', 'PySpark']}
```

The top-level `name` association is separate, but the nested `skills` list is shared.

---

## 14. Quick Revision

- **Reference:** A way to access an object.
- **Assignment:** Binds a variable to an object; it does not copy the object.
- **Aliasing:** Multiple variables refer to the same object.
- **Mutation:** Changes an object's state.
- **Reassignment:** Makes a variable refer to another object.
- **Shallow copy:** Creates a new outer object but shares nested objects.
- **Deep copy:** Recursively copies nested objects where supported.
- **`copy.copy()`:** Creates a shallow copy.
- **`copy.deepcopy()`:** Creates a deep copy.
- **`==`:** Checks equality.
- **`is`:** Checks identity.
- **Function arguments:** Mutating a shared object affects that object; rebinding a local parameter does not rebind the caller's variable.
- **Data engineering:** Copy only when needed, and consider the memory cost for large datasets.

**Remember:** Assignment shares a reference, shallow copy separates the outer object, and deep copy aims to separate the nested object structure.