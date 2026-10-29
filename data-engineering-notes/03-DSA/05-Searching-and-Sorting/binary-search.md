# Binary Search

## 1. What Is Binary Search?

**Binary search** is an efficient searching algorithm that repeatedly divides a search space into two halves to find a target or determine its position.

It works when the data is sorted or when the search space has a property that allows half of the candidates to be safely eliminated.

### Example

```text
Array:  [2, 5, 8, 12, 16, 23, 38]
Target: 16

Step 1: middle = 12
        16 > 12 → search right half

Step 2: middle = 23
        16 < 23 → search left half

Step 3: middle = 16
        Target found at index 4
```

Instead of checking every element, binary search eliminates roughly half the remaining candidates at each step.

### Complexity

| Complexity | Value |
|---|---|
| Best-case time | O(1) |
| Average-case time | O(log n) |
| Worst-case time | O(log n) |
| Iterative auxiliary space | O(1) |
| Recursive auxiliary space | O(log n) |

Here, `n` is the number of elements.

## 2. How Binary Search Works

Binary search maintains two boundaries:

- `left`: the beginning of the search range.
- `right`: the end of the search range.

Calculate the middle index:

```python
mid = left + (right - left) // 2
```

Compare `nums[mid]` with the target.

- If they are equal, the target is found.
- If the middle value is smaller, search the right half.
- If the middle value is larger, search the left half.

For standard binary search, the search interval is inclusive: `[left, right]`.

## 3. Iterative Binary Search

This is the standard implementation to learn first.

```python
def binary_search(nums: list[int], target: int) -> int:
    left = 0
    right = len(nums) - 1

    while left <= right:
        mid = left + (right - left) // 2

        if nums[mid] == target:
            return mid

        elif nums[mid] < target:
            left = mid + 1

        else:
            right = mid - 1

    return -1
```

### Example

```python
nums = [2, 5, 8, 12, 16, 23, 38]

print(binary_search(nums, 16))  # 4
print(binary_search(nums, 10))  # -1
```

Python uses zero-based indexing, so the element `16` is at index `4`.

### Why use `left <= right`?

The interval includes both boundaries. When `left == right`, one candidate remains and must still be checked.

When `left > right`, the search space is empty.

### Why use `mid + 1` and `mid - 1`?

Once `nums[mid]` is compared with the target and is not equal, the middle element can be eliminated.

```python
left = mid + 1
right = mid - 1
```

These updates ensure the search range shrinks and prevent infinite loops.

## 4. Dry Run

Consider:

```python
nums = [2, 5, 8, 12, 16, 23, 38]
target = 16
```

| Step | Left | Right | Mid | Middle value | Decision |
|---|---:|---:|---:|---:|---|
| 1 | 0 | 6 | 3 | 12 | Search right |
| 2 | 4 | 6 | 5 | 23 | Search left |
| 3 | 4 | 4 | 4 | 16 | Found |

The target is found at index `4` after only three comparisons.

For seven elements, linear search might require seven comparisons in the worst case. Binary search needs only a few because it reduces the search space exponentially.

## 5. Recursive Binary Search

Binary search can also be implemented using recursion.

```python
def binary_search_recursive(
    nums: list[int],
    target: int,
    left: int,
    right: int
) -> int:
    if left > right:
        return -1

    mid = left + (right - left) // 2

    if nums[mid] == target:
        return mid

    if nums[mid] < target:
        return binary_search_recursive(
            nums, target, mid + 1, right
        )

    return binary_search_recursive(
        nums, target, left, mid - 1
    )
```

Usage:

```python
nums = [2, 5, 8, 12, 16, 23, 38]

print(
    binary_search_recursive(
        nums, 16, 0, len(nums) - 1
    )
)  # 4
```

### Iterative vs recursive

| Feature | Iterative | Recursive |
|---|---|---|
| Uses | Loop | Function calls |
| Time | O(log n) | O(log n) |
| Auxiliary space | O(1) | O(log n) |
| Stack overhead | No recursive calls | Yes |
| Typical interview preference | Often preferred | Useful for understanding recursion |

The iterative version is generally preferred in Python because it avoids recursive-call overhead.

## 6. Binary Search vs Linear Search

| Feature | Linear search | Binary search |
|---|---|---|
| Search method | Check elements one by one | Eliminate half the candidates |
| Requires sorted data | No | Yes, for ordinary array search |
| Worst-case time | O(n) | O(log n) |
| Auxiliary space | O(1) | O(1) iteratively |
| Best suited for | Small or unsorted data | Sorted data or monotonic search spaces |

Example:

```python
def linear_search(nums, target):
    for i, value in enumerate(nums):
        if value == target:
            return i

    return -1
```

Linear search is still the correct choice when the data is unsorted and sorting it is not worthwhile.

## 7. Common Variations

Binary search is a family of algorithms, not just one implementation.

### A. Search Insert Position

Find the index where a target should be inserted to maintain sorted order.

```python
def search_insert(nums: list[int], target: int) -> int:
    left, right = 0, len(nums)

    while left < right:
        mid = left + (right - left) // 2

        if nums[mid] < target:
            left = mid + 1
        else:
            right = mid

    return left
```

Example:

```python
print(search_insert([1, 3, 5, 6], 5))  # 2
print(search_insert([1, 3, 5, 6], 2))  # 1
print(search_insert([1, 3, 5, 6], 7))  # 4
```

This implementation finds the **lower bound**: the first index whose value is greater than or equal to the target.

It returns `len(nums)` when the target belongs after all existing elements.

### B. First and Last Occurrence

When duplicates exist, ordinary binary search may return any matching index.

To find the first occurrence, keep searching left after finding a match. To find the last occurrence, keep searching right.

For reusable implementations of these variations, see `binary-search-patterns.md`.

### C. Binary Search on the Answer

Sometimes the array is not sorted, but the answer has a monotonic feasibility condition.

For example:
- Find the minimum shipping capacity needed to deliver packages within a deadline.
- Find the minimum speed required to finish a task on time.
- Find the maximum minimum distance between placed items.

These problems search a range of possible answers instead of searching array indices.

The key requirement is that feasibility must change monotonically across the search range.

See `binary-search-patterns.md` for detailed templates and examples.

## 8. Python's Built-in Binary Search Tools

Python provides the `bisect` module for finding insertion positions in sorted lists.

```python
from bisect import bisect_left, bisect_right

nums = [1, 2, 2, 2, 4, 6]

print(bisect_left(nums, 2))   # 1
print(bisect_right(nums, 2))  # 4
```

- `bisect_left()` returns the first position where the target can be inserted without violating sorted order, before existing equal values.
- `bisect_right()` returns the first position after existing equal values.

Count the occurrences of a value in a sorted list:

```python
count = bisect_right(nums, 2) - bisect_left(nums, 2)

print(count)  # 3
```

Both searches take O(log n) time.

**Interview tip:** Know how to implement binary search manually, even though Python provides `bisect`.

## 9. Common Mistakes

1. **Using binary search on unsorted data.** Ordinary binary search requires sorted input.
2. **Incorrect loop condition.** With inclusive boundaries, use `while left <= right`.
3. **Incorrect boundary updates.** Use `mid + 1` and `mid - 1` for standard exact search.
4. **Infinite loops.** Ensure each iteration strictly reduces the search space.
5. **Ignoring empty input.** Initializing `right = len(nums) - 1` correctly gives `-1` for an empty list.
6. **Assuming duplicates have one specific matching index.** Standard binary search can return any matching occurrence.
7. **Confusing an index with a value.** Return `mid` when asked for an index; return `nums[mid]` when asked for the element.
8. **Sorting unnecessarily.** Sorting first costs O(n log n) and may change the original indices.
9. **Forgetting the search invariant.** Be clear about which indices are still possible answers.

## 10. Practice Problems

### Easy

1. Binary Search
2. Search Insert Position
3. Sqrt(x)
4. Valid Perfect Square
5. First Bad Version

### Medium

1. Find First and Last Position of Element in Sorted Array
2. Search in Rotated Sorted Array
3. Find Minimum in Rotated Sorted Array
4. Find Peak Element
5. Search a 2D Matrix
6. Koko Eating Bananas
7. Capacity to Ship Packages Within D Days

### Practice method

For each problem:

1. Identify whether the data or answer space has a usable ordering.
2. Define `left`, `right`, and what the interval represents.
3. Decide what happens when the middle candidate is too small or too large.
4. Test a one-element array and a missing target.
5. Explain the time and auxiliary space complexity.

## 11. Quick Revision

- Binary search reduces the search space by approximately half per iteration.
- Ordinary array binary search requires sorted data.
- Time complexity is O(log n).
- Iterative auxiliary space is O(1).
- Use `left = mid + 1` when the answer must be to the right.
- Use `right = mid - 1` when the answer must be to the left.
- Use `while left <= right` for the standard inclusive-interval template.
- Use lower and upper bounds when duplicates matter.
- Binary search can also operate on answers when feasibility is monotonic.

**Interview takeaway:** Master the standard iterative implementation first. Then learn lower and upper bounds, rotated arrays, and binary search on the answer. Correct boundary handling and a clear search invariant are more important than memorizing multiple code templates.