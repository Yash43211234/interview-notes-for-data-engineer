# Lists, Tuples, and Sets in Python

## 1. Overview

Python provides three important collection types to store multiple values: lists, tuples, and sets.

| Feature | List | Tuple | Set |
|---|---|---|---|
| Syntax | `[1, 2, 3]` | `(1, 2, 3)` | `{1, 2, 3}` |
| Ordered | Yes | Yes | No guaranteed positional order |
| Mutable | Yes | No | Yes |
| Allows duplicates | Yes | Yes | No |
| Indexing | Yes | Yes | No |
| Allows mixed types | Yes | Yes | Yes |
| Common use | Modifiable collections | Fixed collections | Unique values and set operations |

**Remember:**
- **List:** Ordered and changeable.
- **Tuple:** Ordered and immutable.
- **Set:** Stores unique, hashable elements.

---

## 2. Lists

A list stores multiple elements in a specific order. You can add, remove, or update its elements.

### Creating a list

```python
numbers = [10, 20, 30, 40]
names = ["Amit", "Rahul", "Priya"]
mixed = [10, "Python", True, 3.5]
empty = []
```

### Accessing elements

Lists use zero-based indexing.

```python
numbers = [10, 20, 30, 40]

print(numbers[0])   # 10
print(numbers[2])   # 30
print(numbers[-1])  # 40
print(numbers[-2])  # 30
```

Negative indexing starts from the end: `-1` refers to the last element.

### Slicing

Syntax:

```python
list[start:stop:step]
```

The `stop` index is excluded.

```python
numbers = [10, 20, 30, 40, 50]

print(numbers[1:4])   # [20, 30, 40]
print(numbers[:3])    # [10, 20, 30]
print(numbers[2:])    # [30, 40, 50]
print(numbers[::2])   # [10, 30, 50]
print(numbers[::-1])  # [50, 40, 30, 20, 10]
```

### Modifying a list

```python
numbers = [10, 20, 30]

numbers[1] = 25
numbers.append(40)
numbers.extend([50, 60])
numbers.insert(0, 5)

print(numbers)
# [5, 10, 25, 30, 40, 50, 60]
```

Important methods:

| Method | Purpose |
|---|---|
| `append(x)` | Adds one element to the end |
| `extend(iterable)` | Adds elements from an iterable |
| `insert(i, x)` | Inserts an element at an index |
| `remove(x)` | Removes the first matching value |
| `pop(i)` | Removes and returns the element at index `i` |
| `clear()` | Removes all elements |
| `index(x)` | Returns the first matching index |
| `count(x)` | Counts occurrences of a value |
| `sort()` | Sorts the list in place |
| `reverse()` | Reverses the list in place |
| `copy()` | Creates a shallow copy |

Example:

```python
numbers = [30, 10, 20, 10]

numbers.remove(10)
print(numbers)  # [30, 20, 10]

removed = numbers.pop()
print(removed)  # 10

numbers.sort()
print(numbers)  # [20, 30]
```

**Important:** `remove()` removes by value, whereas `pop()` removes by index and returns the removed element. Calling `pop()` without an index removes the last element.

### `append()` vs `extend()`

```python
a = [1, 2]
a.append([3, 4])

print(a)
# [1, 2, [3, 4]]
```

`append()` treats the supplied object as one element.

```python
b = [1, 2]
b.extend([3, 4])

print(b)
# [1, 2, 3, 4]
```

`extend()` adds each element from the iterable separately.

### Sorting a list

```python
numbers = [40, 10, 30, 20]

numbers.sort()
print(numbers)  # [10, 20, 30, 40]

numbers.sort(reverse=True)
print(numbers)  # [40, 30, 20, 10]
```

`sort()` modifies the original list and returns `None`.

```python
numbers = [3, 1, 2]

result = numbers.sort()
print(result)   # None
print(numbers)  # [1, 2, 3]
```

To create a sorted list without modifying the original:

```python
numbers = [3, 1, 2]

sorted_numbers = sorted(numbers)

print(sorted_numbers)  # [1, 2, 3]
print(numbers)         # [3, 1, 2]
```

### List unpacking

```python
values = [10, 20, 30]

a, b, c = values

print(a)  # 10
print(b)  # 20
print(c)  # 30
```

Use `*` to capture remaining values:

```python
numbers = [10, 20, 30, 40, 50]

first, *middle, last = numbers

print(first)   # 10
print(middle)  # [20, 30, 40]
print(last)    # 50
```

The number of elements must match the unpacking pattern unless a starred target is used.

---

## 3. Tuples

A tuple is an ordered collection that cannot be modified after creation.

### Creating tuples

```python
coordinates = (10, 20)
employee = ("Amit", 101, "Data Engineer")
empty = ()
```

A tuple containing a single element requires a trailing comma:

```python
a = (10,)
b = (10)

print(type(a))  # <class 'tuple'>
print(type(b))  # <class 'int'>
```

Parentheses alone do not create a tuple. The comma does.

### Accessing tuple elements

Tuples support indexing, slicing, and iteration, just like lists.

```python
values = (10, 20, 30, 40)

print(values[0])    # 10
print(values[-1])   # 40
print(values[1:3])  # (20, 30)
```

### Tuple immutability

You cannot replace, add, or remove tuple elements directly.

```python
values = (10, 20, 30)

# values[0] = 100  # TypeError
```

However, a tuple can contain a mutable object such as a list. That inner object can still be modified.

```python
data = (10, [20, 30])

data[1].append(40)

print(data)
# (10, [20, 30, 40])
```

The tuple's element references cannot be reassigned, but the referenced list itself is mutable.

### Tuple methods

Tuples have two main methods:

```python
values = (10, 20, 10, 30)

print(values.count(10))  # 2
print(values.index(20))  # 1
```

### Tuple unpacking

```python
employee = ("Amit", 25, "Data Engineer")

name, age, role = employee

print(name)  # Amit
print(age)   # 25
print(role)  # Data Engineer
```

### When should you use a tuple?

Use tuples when:
- The collection represents a fixed group of values.
- You want to communicate that the collection should not be structurally modified.
- You need an immutable, hashable collection of hashable elements, such as a dictionary key.

Example:

```python
location = (22.5726, 88.3639)
```

The coordinates form a fixed pair of values.

**Important:** A tuple is hashable only if all its elements are hashable. A tuple containing a list cannot be used as a dictionary key.

---

## 4. Sets

A set is a mutable collection of unique, hashable elements. Sets do not support indexing or slicing.

### Creating sets

```python
numbers = {10, 20, 30}
empty_set = set()
```

Use `set()` to create an empty set. `{}` creates an empty dictionary, not a set.

### Duplicate removal

```python
numbers = {10, 20, 10, 30, 20}

print(numbers)
# {10, 20, 30}  (display order may vary)
```

Sets automatically eliminate duplicates.

A set cannot contain unhashable elements such as lists or dictionaries.

```python
# invalid = {[1, 2], [3, 4]}  # TypeError
```

### Adding and removing elements

```python
numbers = {10, 20, 30}

numbers.add(40)
numbers.update([50, 60])

numbers.discard(20)

print(numbers)
# {10, 30, 40, 50, 60}  (display order may vary)
```

Important methods:

| Method | Purpose |
|---|---|
| `add(x)` | Adds one element |
| `update(iterable)` | Adds multiple elements |
| `remove(x)` | Removes an element; raises `KeyError` if absent |
| `discard(x)` | Removes an element if present; no error if absent |
| `pop()` | Removes and returns an arbitrary element |
| `clear()` | Removes all elements |

### Set operations

Suppose:

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}
```

**Union:** Elements present in either set.

```python
print(a | b)
# {1, 2, 3, 4, 5, 6}
```

Equivalent method: `a.union(b)`.

**Intersection:** Elements common to both sets.

```python
print(a & b)
# {3, 4}
```

Equivalent method: `a.intersection(b)`.

**Difference:** Elements in the first set but not in the second.

```python
print(a - b)
# {1, 2}
```

Equivalent method: `a.difference(b)`.

The difference is directional: `b - a` produces `{5, 6}`.

**Symmetric difference:** Elements in either set, but not in both.

```python
print(a ^ b)
# {1, 2, 5, 6}
```

Equivalent method: `a.symmetric_difference(b)`.

### Subsets and supersets

```python
a = {1, 2}
b = {1, 2, 3, 4}

print(a.issubset(b))    # True
print(b.issuperset(a))  # True
print(a.isdisjoint({5, 6}))  # True
```

- A subset contains only elements that also occur in another set.
- A superset contains every element of another set.
- Disjoint sets have no common elements.

### Removing duplicates from a list

```python
numbers = [1, 2, 2, 3, 1, 4]

unique_numbers = list(set(numbers))

print(unique_numbers)
```

This removes duplicates but does **not** guarantee preservation of the original order.

To preserve order:

```python
numbers = [1, 2, 2, 3, 1, 4]

unique_numbers = list(dict.fromkeys(numbers))

print(unique_numbers)
# [1, 2, 3, 4]
```

This approach preserves the first occurrence of each hashable value.

---

## 5. Choosing the Right Collection

| Requirement | Recommended type |
|---|---|
| Store ordered values and modify them | List |
| Store a fixed ordered group | Tuple |
| Remove duplicates | Set |
| Perform union or intersection | Set |
| Access values by index | List or tuple |
| Use a collection as a dictionary key | Tuple of hashable elements |
| Preserve insertion order while removing duplicates | `dict.fromkeys()` or a dictionary |
| Frequently add and remove elements by position | List |

For Data Engineering, these distinctions are useful when processing records, removing duplicate IDs, comparing datasets, and representing fixed configuration values.

---

## 6. Time Complexity

Typical average-case complexity for Python's built-in collections:

| Operation | List | Tuple | Set |
|---|---:|---:|---:|
| Index access | \(O(1)\) | \(O(1)\) | Not supported |
| Search by value | \(O(n)\) | \(O(n)\) | \(O(1)\) average |
| Append one element | \(O(1)\) amortized | Not supported | `add()` is \(O(1)\) average |
| Insert/remove near beginning | \(O(n)\) | Not supported | Not positional |
| Remove one element by value | \(O(n)\) | Not supported | \(O(1)\) average |
| Sort | \(O(n\log n)\) | Not in place | Not supported |

Notes:
- Set operations such as union, intersection, and difference depend on the sizes of the sets and the operation performed.
- Set operations are generally fast because sets use hash tables.
- Worst-case hash-table operations can be slower than their average-case complexity.
- Sorting a list may use less than \(O(n\log n)\) time on already ordered or partially ordered input.

---

## 7. Common Mistakes

### Mistake 1: Confusing `append()` and `extend()`

```python
a = [1, 2]
a.append([3, 4])
# [1, 2, [3, 4]]
```

`append()` adds one object; `extend()` adds elements from an iterable.

### Mistake 2: Forgetting the comma in a single-element tuple

```python
a = (5,)
b = (5)

print(type(a))  # tuple
print(type(b))  # int
```

### Mistake 3: Assuming a set preserves order

Sets do not guarantee positional ordering. Do not use a set when the order of output matters.

### Mistake 4: Using `remove()` when an element might be absent

```python
numbers = {1, 2, 3}

numbers.discard(10)  # Safe
# numbers.remove(10)  # KeyError
```

### Mistake 5: Expecting `sort()` to return a sorted list

```python
numbers = [3, 1, 2]

result = numbers.sort()
print(result)  # None
```

Use `sorted(numbers)` when you need a new sorted list.

### Mistake 6: Thinking a tuple makes every nested value immutable

```python
data = ([1, 2], 3)
data[0].append(4)

print(data)
# ([1, 2, 4], 3)
```

The tuple structure is immutable, but the inner list is not.

### Mistake 7: Modifying a list while iterating over it

Removing elements during iteration can cause elements to be skipped.

A safer approach:

```python
numbers = [1, 2, 3, 4, 5, 6]

filtered = [n for n in numbers if n % 2 == 0]

print(filtered)
# [2, 4, 6]
```

---

## 8. Interview Practice Questions

Try to answer these without running the code first.

### Easy

**Q1. What is the output?**

```python
a = [1, 2, 3]
a.append([4, 5])
print(len(a))
```

**Q2. What is the difference between a list and a tuple?**

**Q3. What is the correct way to create a set with no elements?**

**Q4. What is the output?**

```python
a = {1, 2, 2, 3}
print(len(a))
```

### Medium

**Q5. What is the output?**

```python
a = [10, 20, 30]
b = a
b.append(40)

print(a)
```

**Q6. What is the output?**

```python
a = {1, 2, 3}
b = {3, 4, 5}

print(a - b)
print(a & b)
print(a | b)
```

**Q7. Remove duplicates from a list while preserving the original order.**

Input:

```python
numbers = [4, 2, 4, 1, 2, 3, 1]
```

Expected output:

```python
[4, 2, 1, 3]
```

**Q8. Why can a tuple containing a list not be used as a dictionary key?**

### Answers

1. `4` — the nested list is one element.
2. Lists are mutable; tuples are immutable.
3. `set()`.
4. `3`.
5. `[10, 20, 30, 40]` — both names reference the same list.
6. Difference: `{1, 2}`; intersection: `{3}`; union: `{1, 2, 3, 4, 5}`. Display order may vary.
7. `list(dict.fromkeys(numbers))`.
8. The nested list is unhashable, so the tuple is also unhashable and cannot serve as a dictionary key.

---

## 9. Quick Revision

- Lists use `[]`, support indexing, and are mutable.
- Tuples use `()` and are immutable; a single-element tuple needs a comma.
- Sets use `{}` for nonempty sets and `set()` for an empty set.
- Lists and tuples allow duplicates; sets do not.
- `append()` adds one element; `extend()` adds multiple elements.
- `sort()` modifies a list and returns `None`; `sorted()` returns a new sorted list.
- `remove()` raises an error if the value is absent; `discard()` does not.
- Set operators: `|` union, `&` intersection, `-` difference, `^` symmetric difference.
- Use a set for efficient average-case membership checks and deduplication.
- Use `list(dict.fromkeys(values))` to remove duplicates while preserving order.

**Interview takeaway:** Choose a collection based on mutability, ordering, uniqueness, indexing requirements, and the operations you need to perform.