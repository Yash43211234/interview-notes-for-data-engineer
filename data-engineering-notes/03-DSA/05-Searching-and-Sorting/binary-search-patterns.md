# Binary Search Patterns

## 1. What Is Binary Search?

**Binary search** is an algorithm that finds a target in a sorted search space by repeatedly eliminating half of the remaining possibilities.

Instead of checking every element, binary search compares the target with the middle element and decides which half may contain the answer.

Example:

```text
Array:  [2, 5, 8, 12, 16, 23, 38]
Target: 16

Step 1: middle = 12
        16 > 12 → search right half

Step 2: middle = 23
        16 < 23 → search left half

Step 3: middle = 16
        Target found
```

**Time complexity:** O(log n)

**Space complexity:** O(1) for iterative binary search.

Binary search works on sorted arrays, but it also works on other search spaces when there is a reliable way to eliminate half the candidates at each step.

## 2. Standard Binary Search

Use standard binary search when you need to find the exact target in a sorted array.

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

Example:

```python
nums = [2, 5, 8, 12, 16, 23, 38]

print(binary_search(nums, 16))  # 4
print(binary_search(nums, 10))  # -1
```

### How the boundaries work

- `left` is the first candidate index.
- `right` is the last candidate index.
- `left <= right` means at least one candidate remains.
- `left = mid + 1` eliminates the middle element and everything to its left.
- `right = mid - 1` eliminates the middle element and everything to its right.

Using `mid + 1` and `mid - 1` prevents the algorithm from getting stuck when the middle element is not the answer.

## 3. Pattern 1: Find an Exact Element

**Use when:** The array is sorted and the question asks whether a target exists or where it is located.

```python
def search(nums, target):
    left, right = 0, len(nums) - 1

    while left <= right:
        mid = left + (right - left) // 2

        if nums[mid] == target:
            return mid

        if nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return -1
```

Important details:

- If the target is absent, return the required missing-value indicator.
- If duplicates exist, this implementation may return any matching occurrence.
- If the question requires the first or last occurrence, use a boundary-search pattern instead.

## 4. Pattern 2: Lower Bound — First Element Greater Than or Equal to Target

A **lower bound** is the first position where the array value is greater than or equal to the target.

Example:

```text
nums   = [1, 2, 2, 2, 4, 6]
target = 2

Lower bound index = 1
```

If the target is absent, the lower bound is still meaningful.

```text
nums   = [1, 3, 5, 7]
target = 4

Lower bound index = 2
Value at index 2 = 5
```

If no element is greater than or equal to the target, the answer is `len(nums)`.

### Implementation

```python
def lower_bound(nums: list[int], target: int) -> int:
    left, right = 0, len(nums)

    while left < right:
        mid = left + (right - left) // 2

        if nums[mid] < target:
            left = mid + 1
        else:
            right = mid

    return left
```

Notice that `right = len(nums)`, not `len(nums) - 1`.

The search uses a half-open interval `[left, right)`. The right boundary is excluded, so `len(nums)` can represent the position after the final element.

**Time:** O(log n)

### Why use `right = mid`?

When `nums[mid] >= target`, the middle position could be the first valid position. Therefore, keep it in the search space by setting `right = mid`.

Using `right = mid - 1` could discard the answer.

## 5. Pattern 3: Upper Bound — First Element Strictly Greater Than Target

An **upper bound** is the first position where the array value is strictly greater than the target.

```text
nums   = [1, 2, 2, 2, 4, 6]
target = 2

Upper bound index = 4
Value = 4
```

### Implementation

```python
def upper_bound(nums: list[int], target: int) -> int:
    left, right = 0, len(nums)

    while left < right:
        mid = left + (right - left) // 2

        if nums[mid] <= target:
            left = mid + 1
        else:
            right = mid

    return left
```

### Lower bound vs upper bound

| Pattern | Finds |
|---|---|
| Lower bound | First value `>= target` |
| Upper bound | First value `> target` |
| First occurrence | Lower bound, if target exists |
| Last occurrence | Upper bound minus one, if target exists |
| Count of target | Upper bound minus lower bound |

Python's `bisect` module provides these operations through `bisect_left()` and `bisect_right()`.

## 6. Pattern 4: First and Last Occurrence

**Problem:** Find the first and last positions of a target in a sorted array that may contain duplicates.

Example:

```text
Input:
nums = [1, 2, 2, 2, 3, 4]
target = 2

Output:
[1, 3]
```

### Implementation

```python
def search_range(nums: list[int], target: int) -> list[int]:
    first = lower_bound(nums, target)

    if first == len(nums) or nums[first] != target:
        return [-1, -1]

    last = upper_bound(nums, target) - 1

    return [first, last]
```

This solution reuses the lower-bound and upper-bound functions.

**Complexity:**
- Time: O(log n)
- Auxiliary space: O(1)

### Alternative: Two binary searches

You can also write separate searches that continue looking after finding the target:

- First occurrence: when equal, save the index and continue left.
- Last occurrence: when equal, save the index and continue right.

This is a common interview variation.

## 7. Pattern 5: Binary Search on the Answer

This is one of the most important binary search patterns for coding interviews.

Sometimes the problem does not ask you to find a value in a sorted array. Instead, it asks for the **minimum or maximum feasible answer**.

The key idea is to define a condition that changes monotonically from false to true, or from true to false.

### Example: Minimum Capacity to Ship Packages

Given package weights and a number of days, find the minimum ship capacity needed to deliver all packages within the deadline. Packages must be shipped in their given order.

```text
weights = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
days = 5

Answer = 15
```

A capacity that is too small cannot meet the deadline. Increasing the capacity cannot make shipping slower.

Therefore, feasibility is monotonic:

```text
Capacity:   10  11  12  13  14  15  16  17 ...
Feasible:    No  No  No  No  No Yes Yes Yes ...
```

We need the first feasible capacity.

### Step 1: Define the search space

- Minimum capacity: `max(weights)`, because a package must fit on the ship.
- Maximum capacity: `sum(weights)`, because this can ship everything in one day.

### Step 2: Write the feasibility function

```python
def can_ship(weights, days, capacity):
    days_needed = 1
    current_load = 0

    for weight in weights:
        if current_load + weight > capacity:
            days_needed += 1
            current_load = 0

        current_load += weight

    return days_needed <= days
```

### Step 3: Binary search for the first feasible answer

```python
def ship_within_days(weights, days):
    left = max(weights)
    right = sum(weights)

    while left < right:
        mid = left + (right - left) // 2

        if can_ship(weights, days, mid):
            right = mid
        else:
            left = mid + 1

    return left
```

**Complexity:**
- Feasibility check: O(n)
- Binary search iterations: O(log(sum(weights) - max(weights) + 1))
- Total time: O(n log S), where `S` is the width of the capacity search space.
- Auxiliary space: O(1)

### When should you use this pattern?

Look for problems asking for:

- Minimum capacity
- Minimum speed
- Minimum time
- Maximum minimum distance
- Smallest feasible limit
- Largest possible value satisfying a condition

Ask yourself:

1. What is the possible range of answers?
2. Can I check whether a candidate answer is feasible?
3. If one answer is feasible, are all larger or smaller answers also feasible?
4. Am I looking for the first or last feasible answer?

If the feasibility condition is monotonic, binary search may be appropriate.

## 8. Pattern 6: Minimum Feasible Answer

Use this template when the condition becomes true as the answer increases.

```python
def first_true(left, right, feasible):
    while left < right:
        mid = left + (right - left) // 2

        if feasible(mid):
            right = mid
        else:
            left = mid + 1

    return left
```

This template assumes:

- `left` and `right` are inclusive bounds on the possible answer values.
- A feasible answer exists in the range.
- Feasibility is monotonic: false first, then true.

At termination, `left == right` at the first feasible answer.

If feasibility is not guaranteed, check the final candidate or establish feasibility before calling the template.

## 9. Pattern 7: Maximum Feasible Answer

Sometimes feasibility is true for smaller values and false for larger values. You need the largest feasible answer.

Example: Find the maximum minimum distance between placed items.

Suppose positions are:

```text
positions = [1, 2, 4, 8, 9]
items = 3
```

The goal is to place three items so that the minimum distance between any two is as large as possible.

If a distance is feasible, every smaller distance is also feasible. We need the last feasible distance.

### Feasibility check

```python
def can_place(positions, items, distance):
    placed = 1
    last_position = positions[0]

    for position in positions[1:]:
        if position - last_position >= distance:
            placed += 1
            last_position = position

            if placed >= items:
                return True

    return placed >= items
```

### Binary search

```python
def max_min_distance(positions, items):
    positions = sorted(positions)

    left = 0
    right = positions[-1] - positions[0]

    while left < right:
        mid = left + (right - left + 1) // 2

        if can_place(positions, items, mid):
            left = mid
        else:
            right = mid - 1

    return left
```

**Why add 1 to the midpoint?**

```python
mid = left + (right - left + 1) // 2
```

This rounds the midpoint upward. When `left` and `right` differ by one, it prevents the algorithm from repeatedly choosing `left` and getting stuck.

**Complexity:**
- Sorting: O(n log n)
- Each feasibility check: O(n)
- Binary search: O(log D), where `D` is the distance range.
- Total: O(n log n + n log D)

## 10. Pattern 8: Search in a Rotated Sorted Array

A sorted array may be rotated at some pivot.

Example:

```text
Original: [1, 2, 3, 4, 5, 6, 7]
Rotated:  [4, 5, 6, 7, 1, 2, 3]
Target: 2
Output: 5
```

At least one half of the search interval is sorted when all elements are distinct.

### Implementation

```python
def search_rotated(nums: list[int], target: int) -> int:
    left, right = 0, len(nums) - 1

    while left <= right:
        mid = left + (right - left) // 2

        if nums[mid] == target:
            return mid

        # Left half is sorted.
        if nums[left] <= nums[mid]:
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1

        # Right half is sorted.
        else:
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1

    return -1
```

**Complexity:**
- Time: O(log n) when all values are distinct.
- Auxiliary space: O(1).

With duplicates, identifying the sorted half can become ambiguous. In the worst case, the search may need to shrink the interval one element at a time, resulting in O(n) time.

## 11. Pattern 9: Find the Minimum in a Rotated Sorted Array

Example:

```text
Input:  [4, 5, 6, 7, 0, 1, 2]
Output: 0
```

Compare the middle element with the rightmost element:

- If `nums[mid] > nums[right]`, the minimum is to the right of `mid`.
- Otherwise, the minimum is at `mid` or to its left.

```python
def find_min(nums: list[int]) -> int:
    if not nums:
        raise ValueError("Array must not be empty")

    left, right = 0, len(nums) - 1

    while left < right:
        mid = left + (right - left) // 2

        if nums[mid] > nums[right]:
            left = mid + 1
        else:
            right = mid

    return nums[left]
```

**Complexity:**
- Time: O(log n)
- Auxiliary space: O(1)

This implementation assumes the array is nonempty and contains distinct values. Duplicates require additional handling and can degrade the worst-case time to O(n).

## 12. Pattern 10: Binary Search on a 2D Matrix

Some matrices are sorted so that:

- Each row is sorted.
- The first element of each row is greater than the last element of the previous row.

Such a matrix can be treated as one sorted, one-dimensional array.

Example:

```text
[
  [1,  3,  5,  7],
  [10, 11, 16, 20],
  [23, 30, 34, 60]
]
```

To map a one-dimensional index to a matrix position:

```python
row = mid // columns
col = mid % columns
```

### Implementation

```python
def search_matrix(matrix, target):
    if not matrix or not matrix[0]:
        return False

    rows = len(matrix)
    cols = len(matrix[0])

    left, right = 0, rows * cols - 1

    while left <= right:
        mid = left + (right - left) // 2

        value = matrix[mid // cols][mid % cols]

        if value == target:
            return True
        elif value < target:
            left = mid + 1
        else:
            right = mid - 1

    return False
```

**Complexity:**
- Time: O(log(rows × cols))
- Auxiliary space: O(1)

This approach requires the stated global ordering. If only individual rows are sorted, a different search strategy may be needed.

## 13. Pattern 11: Binary Search on Floating-Point Answers

Some problems ask for an answer that is not an integer, such as a square root or a minimum distance with decimal precision.

Example: Approximate the square root of a nonnegative number.

```python
def square_root(x, precision=1e-7):
    if x < 0:
        raise ValueError("x must be non-negative")

    left, right = 0.0, max(1.0, x)

    while right - left > precision:
        mid = (left + right) / 2

        if mid * mid < x:
            left = mid
        else:
            right = mid

    return (left + right) / 2
```

Example:

```python
print(square_root(25))  # Approximately 5.0
print(square_root(2))   # Approximately 1.4142135
```

**Important:** Floating-point binary search usually stops when the interval is smaller than a chosen tolerance rather than when `left == right`.

For fixed-precision answers, another common approach is to run a fixed number of iterations.

## 14. Pattern 12: Peak Element

A peak element is greater than its adjacent elements. For boundary elements, only the existing neighbor must be smaller.

Example:

```text
nums = [1, 2, 3, 1]
Peak index = 2
Peak value = 3
```

The array does not need to be fully sorted.

```python
def find_peak_element(nums: list[int]) -> int:
    if not nums:
        return -1

    left, right = 0, len(nums) - 1

    while left < right:
        mid = left + (right - left) // 2

        if nums[mid] < nums[mid + 1]:
            left = mid + 1
        else:
            right = mid

    return left
```

**Why does this work?**

- If `nums[mid] < nums[mid + 1]`, a peak exists to the right.
- Otherwise, a peak exists at `mid` or to the left.

The search repeatedly chooses a half that is guaranteed to contain a peak.

**Complexity:**
- Time: O(log n)
- Auxiliary space: O(1)

This implementation returns a peak index and assumes the input follows the usual peak-element problem definition.

## 15. How to Choose the Right Pattern

| Problem wording | Likely pattern |
|---|---|
| Find a target in sorted data | Standard binary search |
| First value greater than or equal to target | Lower bound |
| First value greater than target | Upper bound |
| First and last positions of a duplicate target | Two boundary searches |
| Minimum capacity, speed, or time | Binary search on the answer |
| Maximum minimum distance | Maximum feasible answer |
| Search in rotated sorted data | Rotated-array search |
| Find minimum in rotated data | Rotated minimum |
| Sorted matrix with global ordering | Flattened matrix search |
| Approximate a decimal answer | Floating-point binary search |
| Find a peak | Binary search on a directional condition |

## 16. Common Binary Search Mistakes

### Mistake 1: Incorrect loop condition

For a closed interval `[left, right]`, use:

```python
while left <= right:
```

For a half-open interval `[left, right)`, a typical condition is:

```python
while left < right:
```

Do not mix the two templates without adjusting their updates.

### Mistake 2: Incorrect boundary updates

For standard search:

```python
left = mid + 1
right = mid - 1
```

For lower bound:

```python
left = mid + 1
right = mid
```

The update depends on whether the middle position might still be the answer.

### Mistake 3: Infinite loops

Using `left = mid` with a downward-rounded midpoint can cause an infinite loop when `right = left + 1`.

For maximum feasible answers, use an upward-rounded midpoint or another carefully designed loop invariant.

### Mistake 4: Assuming every array problem supports binary search

Binary search needs a property that allows half the candidates to be safely discarded. A sorted array is the most familiar case, but monotonic feasibility is another.

### Mistake 5: Forgetting duplicates

Standard binary search can return any occurrence of a repeated value. Use lower and upper bounds when the exact boundary matters.

### Mistake 6: Incorrect answer range

For answer-space problems, ensure the range contains a valid answer. For example, shipping capacity cannot be smaller than the heaviest package.

### Mistake 7: Ignoring edge cases

Test:
- Empty input
- One element
- Target absent
- Target smaller than all elements
- Target larger than all elements
- Duplicate values
- Minimum and maximum valid answers

## 17. Complexity Summary

| Pattern | Typical time | Auxiliary space |
|---|---:|---:|
| Exact search | O(log n) | O(1) |
| Lower/upper bound | O(log n) | O(1) |
| First and last occurrence | O(log n) | O(1) |
| Search rotated array, distinct values | O(log n) | O(1) |
| Minimum in rotated array, distinct values | O(log n) | O(1) |
| Search globally sorted matrix | O(log(rows × cols)) | O(1) |
| Peak element | O(log n) | O(1) |
| Binary search on answer | O(C log S) | Depends on feasibility check |

Here, `C` is the cost of one feasibility check and `S` is the size of the answer range. More precisely, the number of iterations is logarithmic in the width of that range.

## 18. Practice Problems

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

### Hard
1. Median of Two Sorted Arrays
2. Split Array Largest Sum
3. Aggressive Cows / maximum minimum distance
4. Minimum Number of Days to Make m Bouquets
5. Find in Mountain Array

For each problem, first identify the search space and the property that lets you discard half of it. Then choose the appropriate template.

## 19. Quick Revision

- Binary search repeatedly reduces the search space by about half.
- Standard binary search on sorted data takes O(log n) time.
- Lower bound finds the first element `>= target`.
- Upper bound finds the first element `> target`.
- Binary search on the answer requires a monotonic feasibility condition.
- For the first feasible answer, use a downward-rounded midpoint and preserve the candidate with `right = mid`.
- For the last feasible answer, use an upward-rounded midpoint and discard infeasible candidates with `right = mid - 1`.
- Rotated-array search uses the sorted half to decide where to continue.
- Peak search follows a directional condition, not necessarily a globally sorted array.
- Always define the interval and loop invariant before writing the code.

**Interview takeaway:** Binary search is not just a way to find an element in a sorted array. The deeper skill is identifying a monotonic condition, defining a correct search space, and maintaining boundaries so the answer is never accidentally discarded.