# Hashing in Python — DSA Interview Notes

## 1. What Is Hashing?

Hashing is a technique used to store and retrieve information efficiently using a **hash function**.

In Python, hashing is commonly used through:

- `dict` — stores key-value pairs.
- `set` — stores unique elements.
- `collections.Counter` — counts occurrences of elements.

Hashing is especially useful when a problem requires frequent lookups, duplicate detection, frequency counting, or finding relationships between values.

### Example: Searching with a list vs. a set

```python
nums = [10, 20, 30, 40, 50]

print(30 in nums)       # True
print(30 in set(nums))  # True
```

For a list, membership checking takes O(n) time in the worst case. For a set, membership checking takes O(1) expected time.

Note that constructing `set(nums)` also takes O(n) time. If you need many lookups, build the set once and reuse it.

## 2. How Hashing Works

A hash table uses a hash function to determine where a key should be stored.

Conceptually:

1. You provide a key.
2. A hash function computes a hash value.
3. The hash table uses that value to locate a storage position.
4. The value is stored or retrieved from the corresponding location.

For example, a dictionary:

```python
student = {
    "name": "Aman",
    "age": 24
}

print(student["name"])  # Aman
```

Python uses hashing internally to locate the value associated with `"name"`.

You generally do not need to calculate hash values manually when solving DSA problems.

## 3. Hash Maps and Hash Sets

### Hash map — `dict`

A dictionary maps keys to values.

```python
scores = {
    "Aman": 85,
    "Riya": 92,
    "Rahul": 78
}

print(scores["Riya"])  # 92
```

Common operations:

```python
scores["Aman"] = 90       # Insert or update
print(scores.get("Riya")) # Retrieve safely
print("Rahul" in scores)  # Check whether key exists
del scores["Rahul"]      # Delete
```

Expected time complexity for lookup, insertion, and deletion is O(1), assuming good hash distribution.

### Hash set — `set`

A set stores unique elements.

```python
nums = {10, 20, 30}

nums.add(40)
nums.add(20)

print(nums)       # {10, 20, 30, 40} (order may vary)
print(20 in nums) # True
```

Sets automatically eliminate duplicates.

```python
nums = [1, 2, 2, 3, 3, 3]
unique_nums = set(nums)

print(unique_nums)  # {1, 2, 3} (order may vary)
```

Use a set when you need to know whether an element exists. Use a dictionary when you need to associate information with a key.

## 4. Time Complexity of Hashing

| Operation | Average / expected | Worst case |
|---|---:|---:|
| Dictionary lookup | O(1) | O(n) |
| Dictionary insertion | O(1) amortized | O(n) |
| Set membership | O(1) | O(n) |
| Set insertion | O(1) amortized | O(n) |
| Build a set from `n` elements | O(n) | O(n²) in a pathological collision scenario |

Worst-case behavior is uncommon in ordinary interview problems, but it is important to understand that O(1) is an expected or average-case bound, not a universal guarantee.

## 5. Pattern: Frequency Counting

Use frequency counting when you need to know how many times each element appears.

### Example: Count occurrences

```python
def count_frequency(nums):
    freq = {}

    for num in nums:
        freq[num] = freq.get(num, 0) + 1

    return freq


print(count_frequency([1, 2, 1, 3, 2, 1]))
# {1: 3, 2: 2, 3: 1}
```

`freq.get(num, 0)` returns the current count, or `0` if the key does not exist.

The same task can be written using `Counter`:

```python
from collections import Counter

nums = [1, 2, 1, 3, 2, 1]
freq = Counter(nums)

print(freq[1])  # 3
```

**Complexity:** O(n) expected time and O(k) extra space, where `k` is the number of distinct values.

Typical applications:

- Find duplicates.
- Find the most frequent element.
- Compare character frequencies.
- Count products, transactions, or events by category.
- Identify elements that occur exactly once.

## 6. Pattern: Duplicate Detection

Use a set to track which elements have already appeared.

```python
def contains_duplicate(nums):
    seen = set()

    for num in nums:
        if num in seen:
            return True

        seen.add(num)

    return False


print(contains_duplicate([1, 2, 3, 2]))  # True
print(contains_duplicate([1, 2, 3, 4]))  # False
```

The algorithm returns as soon as it finds a duplicate.

**Complexity:** O(n) expected time and O(n) extra space.

An alternative is:

```python
def contains_duplicate(nums):
    return len(nums) != len(set(nums))
```

The second approach is concise, but the first approach can stop early when a duplicate appears near the beginning.

## 7. Pattern: Two Sum

**Problem:** Given an array and a target, return the indices of two elements whose sum equals the target.

A nested-loop solution takes O(n²) time. Hashing can reduce the expected time to O(n).

```python
def two_sum(nums, target):
    seen = {}

    for i, num in enumerate(nums):
        complement = target - num

        if complement in seen:
            return [seen[complement], i]

        seen[num] = i

    return []


print(two_sum([2, 7, 11, 15], 9))
# [0, 1]
```

### How it works

For each number:

1. Calculate the required complement.
2. Check whether that complement has already appeared.
3. If it has, return the previous index and current index.
4. Otherwise, store the current number and its index.

The lookup happens before insertion, so the algorithm does not use the same element twice.

**Complexity:** O(n) expected time and O(n) extra space.

## 8. Pattern: Grouping Elements

A dictionary can group values that share a property.

### Example: Group words by their first letter

```python
def group_by_first_letter(words):
    groups = {}

    for word in words:
        key = word[0].lower()
        groups.setdefault(key, []).append(word)

    return groups


print(group_by_first_letter(["Apple", "Ant", "Banana", "Berry"]))
# {'a': ['Apple', 'Ant'], 'b': ['Banana', 'Berry']}
```

`setdefault(key, [])` creates the key with an empty list if it does not exist, then returns the list associated with that key.

For production code, handle empty strings if they may appear in the input.

Grouping patterns are useful for:

- Grouping anagrams.
- Grouping records by customer ID.
- Categorizing events by type.
- Aggregating transactions by account.
- Organizing data by date or region.

## 9. Pattern: Group Anagrams

Two words are anagrams if they contain the same characters with the same frequencies.

A sorted tuple of characters can serve as a dictionary key.

```python
def group_anagrams(words):
    groups = {}

    for word in words:
        key = tuple(sorted(word))

        if key not in groups:
            groups[key] = []

        groups[key].append(word)

    return list(groups.values())


print(group_anagrams(["eat", "tea", "tan", "ate", "nat", "bat"]))
# Groups equivalent to:
# [['eat', 'tea', 'ate'], ['tan', 'nat'], ['bat']]
```

The exact order of groups is not important unless the problem requires a particular order.

For `n` words of average length `m`, sorting each word takes approximately O(n × m log m) time, ignoring variations in word lengths.

An alternative is to use a character-frequency tuple as the key. This can be more efficient when the character set is fixed, such as lowercase English letters.

## 10. Pattern: Prefix Sum + Hash Map

This pattern solves subarray problems where a running total and previously seen totals reveal an answer.

### Example: Count subarrays whose sum equals `k`

```python
def subarray_sum(nums, k):
    prefix_count = {0: 1}
    prefix_sum = 0
    count = 0

    for num in nums:
        prefix_sum += num

        needed = prefix_sum - k
        count += prefix_count.get(needed, 0)

        prefix_count[prefix_sum] = (
            prefix_count.get(prefix_sum, 0) + 1
        )

    return count


print(subarray_sum([1, 1, 1], 2))  # 2
print(subarray_sum([1, -1, 0], 0)) # 3
```

### Why it works

Suppose the current prefix sum is `P`. If an earlier prefix sum was `P - k`, the sum of the elements between those prefix positions is `k`.

The dictionary stores how often each prefix sum has appeared.

The initial entry `{0: 1}` handles subarrays that start at index `0`.

**Complexity:** O(n) expected time and O(n) extra space.

This technique works with negative numbers, unlike the usual variable-size sliding-window method for positive-number arrays.

## 11. Pattern: First Non-Repeating Element

Use a frequency map to count occurrences, then scan the input in its original order.

```python
from collections import Counter

def first_unique(nums):
    freq = Counter(nums)

    for num in nums:
        if freq[num] == 1:
            return num

    return None


print(first_unique([4, 5, 1, 2, 1, 4, 5]))
# 2
```

Why use two passes?

- First pass: calculate the frequency of every element.
- Second pass: find the first element with frequency `1`.

**Complexity:** O(n) expected time and O(n) extra space.

The same idea works for strings, such as finding the first non-repeating character.

## 12. Hashing and Mutable Objects in Python

Python dictionaries and sets require hashable keys or elements.

Common hashable values include:

- Integers.
- Strings.
- Tuples whose elements are all hashable.
- Booleans.

Lists, dictionaries, and sets are unhashable because they are mutable.

```python
data = {}

data["name"] = "Aman"  # Valid
data[10] = "value"     # Valid

# data[[1, 2]] = "value"
# TypeError: unhashable type: 'list'
```

A tuple of integers can be used as a key:

```python
data = {}
data[(1, 2)] = "point"

print(data[(1, 2)])  # point
```

However, a tuple containing a list is also unhashable:

```python
# data[(1, [2, 3])] = "value"
# TypeError: unhashable type: 'list'
```

### Why does this matter?

Hash tables rely on a key's hash value remaining stable while it is stored. Mutable objects can change, which would make reliable lookup difficult.

Also remember that Python requires equal objects to have equal hash values. Hash collisions can occur, so a hash value is not a unique identifier.

## 13. Common Hashing Mistakes

### Mistake 1: Using a list for repeated membership checks

```python
# Repeated linear searches
if value in nums:
    ...
```

When there are many lookups, consider building a set once.

### Mistake 2: Overwriting duplicate information

```python
seen[num] = i
```

A dictionary stores one value per key. Repeated assignments replace the previous value. If you need all positions, store a list of indices.

```python
positions = {}

for i, num in enumerate(nums):
    positions.setdefault(num, []).append(i)
```

### Mistake 3: Checking the wrong thing in a dictionary

```python
if value in my_dict:
    ...
```

This checks dictionary **keys**, not values. To search values, use `value in my_dict.values()`, which generally takes O(n) time.

### Mistake 4: Assuming hashing preserves sorted order

A set is not a substitute for a sorted collection. If sorted output is required, explicitly sort the result.

### Mistake 5: Using a set when duplicates matter

A set removes duplicate occurrences. Use a dictionary or `Counter` if the number of occurrences is important.

### Mistake 6: Mishandling empty input

Always check what the problem expects for an empty list, missing result, or no matching pair.

## 14. How to Recognize Hashing Problems

Consider hashing when a question includes:

- Find duplicates or unique elements.
- Count occurrences or frequencies.
- Find two values that meet a condition.
- Check whether an element has appeared before.
- Group elements by a shared property.
- Find subarrays with a particular sum.
- Find the first non-repeating element.
- Compare two collections efficiently.

A useful interview question to ask yourself is:

**Can I store information about previously seen elements so that I do not need to scan them again?**

If yes, a dictionary or set may improve the solution.

## 15. Practice Problems

### Easy

1. Contains Duplicate.
2. Two Sum.
3. Valid Anagram.
4. Intersection of Two Arrays.
5. First Unique Character in a String.
6. Majority Element.

### Medium

1. Group Anagrams.
2. Top K Frequent Elements.
3. Longest Consecutive Sequence.
4. Subarray Sum Equals K.
5. Longest Substring Without Repeating Characters.
6. 4Sum II.
7. Isomorphic Strings.
8. Find All Anagrams in a String.

### Recommended practice order

1. Frequency counting.
2. Duplicate detection.
3. Two Sum.
4. Character-frequency problems.
5. Group Anagrams.
6. Prefix Sum + Hash Map.
7. Longest Consecutive Sequence.
8. Combined hashing and sliding-window problems.

## Quick Revision

- **Dictionary:** Stores key-value pairs.
- **Set:** Stores unique elements.
- **Frequency map:** Counts occurrences.
- **Two Sum:** Stores previously seen values and looks for complements.
- **Grouping:** Uses a shared key to collect related elements.
- **Prefix Sum + Hash Map:** Counts subarrays or finds target sums efficiently.
- **Expected lookup time:** O(1).
- **Space trade-off:** Hashing often improves time complexity by using O(n) additional space.
- **Python restriction:** Dictionary keys and set elements must be hashable.

For interviews, focus on explaining why hashing avoids repeated searches, what information the hash table stores, and how the additional space affects complexity.