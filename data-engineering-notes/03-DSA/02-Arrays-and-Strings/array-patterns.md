# Array Patterns

Array patterns are reusable techniques for solving problems involving lists or arrays. Recognizing the correct pattern helps you write simpler solutions and avoid unnecessary nested loops.

In Python, most array problems use `list`, but the same concepts apply to arrays in other languages.

## 1. Basic Array Operations

```python
nums = [10, 20, 30, 40, 50]

print(nums[0])       # 10
print(nums[-1])      # 50
print(len(nums))     # 5

nums.append(60)
nums.pop()
```

Common operation costs:

| Operation | Typical time complexity |
|---|---|
| Access by index | O(1) |
| Update by index | O(1) |
| Search for a value | O(n) |
| Append to a list | O(1) amortized |
| Insert at the beginning | O(n) |
| Delete from the middle | O(n) |
| Sort | O(n log n) |

## 2. Pattern: Single Pass

Use a single pass when you need to calculate a result by examining each element once.

### Example: Find the maximum

```python
def find_max(nums):
    if not nums:
        return None

    maximum = nums[0]

    for num in nums:
        if num > maximum:
            maximum = num

    return maximum
```

**How it works**

1. Initialize the answer using the first element.
2. Visit each element.
3. Update the answer when a better value is found.

**Complexity:** O(n) time and O(1) extra space.

### Example: Count positive numbers

```python
def count_positive(nums):
    count = 0

    for num in nums:
        if num > 0:
            count += 1

    return count
```

Use this pattern for sums, counts, minimums, maximums, and simple validation.

## 3. Pattern: Two Pointers

Two pointers maintain two positions in the array. They may move toward each other or move in the same direction.

### A. Pointers moving toward each other

Useful for sorted arrays, reversing arrays, and checking palindromes.

**Example: Check whether a sorted array contains two numbers with a target sum**

```python
def has_pair_sum(nums, target):
    left = 0
    right = len(nums) - 1

    while left < right:
        total = nums[left] + nums[right]

        if total == target:
            return True
        elif total < target:
            left += 1
        else:
            right -= 1

    return False
```

Example:

```python
print(has_pair_sum([1, 2, 4, 6, 8], 10))  # True
```

Why it works:

- If the sum is too small, move `left` rightward to increase the sum.
- If the sum is too large, move `right` leftward to decrease the sum.
- This reasoning depends on the array being sorted.

**Complexity:** O(n) time and O(1) extra space.

### B. Pointers moving in the same direction

Useful for removing duplicates, moving zeroes, and partitioning an array.

**Example: Move zeroes to the end**

```python
def move_zeroes(nums):
    write = 0

    for read in range(len(nums)):
        if nums[read] != 0:
            nums[write] = nums[read]
            write += 1

    for i in range(write, len(nums)):
        nums[i] = 0

    return nums
```

Example:

```python
print(move_zeroes([0, 1, 0, 3, 12]))
# [1, 3, 12, 0, 0]
```

The `read` pointer examines each element. The `write` pointer tracks where the next nonzero element belongs.

This solution preserves the order of nonzero elements.

**Complexity:** O(n) time and O(1) extra space.

## 4. Pattern: Sliding Window

Use a sliding window when the problem involves a **contiguous subarray** or substring.

A window is a range between two boundaries. Instead of recalculating everything for every range, update the current result as the window moves.

### A. Fixed-size sliding window

**Problem:** Find the maximum sum of any subarray of size `k`.

```python
def max_window_sum(nums, k):
    if k <= 0 or k > len(nums):
        return None

    window_sum = sum(nums[:k])
    maximum = window_sum

    for right in range(k, len(nums)):
        window_sum += nums[right]
        window_sum -= nums[right - k]
        maximum = max(maximum, window_sum)

    return maximum
```

Example:

```python
print(max_window_sum([2, 1, 5, 1, 3, 2], 3))
# 9
```

The first window is `[2, 1, 5]`, with sum `8`. The next window removes `2` and adds `1`, giving `7`. The following window removes `1` and adds `3`, giving `9`.

**Complexity:** O(n) time and O(1) extra space.

Without the sliding-window technique, repeatedly summing each window could take O(n × k) time.

### B. Variable-size sliding window

Use this when the window expands or shrinks according to a condition.

**Problem:** Find the minimum length of a subarray whose sum is at least `target`, assuming all numbers are positive.

```python
def min_subarray_length(target, nums):
    left = 0
    window_sum = 0
    minimum = float("inf")

    for right, num in enumerate(nums):
        window_sum += num

        while window_sum >= target:
            minimum = min(minimum, right - left + 1)
            window_sum -= nums[left]
            left += 1

    return 0 if minimum == float("inf") else minimum
```

Example:

```python
print(min_subarray_length(7, [2, 3, 1, 2, 4, 3]))
# 2
```

The shortest qualifying subarray is `[4, 3]`.

**Important:** This shrinking-window logic relies on positive numbers. With negative numbers, removing an element does not necessarily reduce the sum, so a different approach may be required.

## 5. Pattern: Prefix Sum

A prefix sum stores cumulative totals so that the sum of a range can be calculated quickly.

For an array `nums`, define:

\[
prefix[i] = nums[0] + nums[1] + \cdots + nums[i-1]
\]

Here, `prefix[0] = 0`. The sum from index `left` through index `right`, inclusive, is:

\[
sum(left,right)=prefix[right+1]-prefix[left]
\]

### Example: Range-sum queries

```python
def build_prefix(nums):
    prefix = [0]

    for num in nums:
        prefix.append(prefix[-1] + num)

    return prefix


def range_sum(prefix, left, right):
    return prefix[right + 1] - prefix[left]
```

Example:

```python
nums = [2, 4, 1, 7, 3]
prefix = build_prefix(nums)

print(range_sum(prefix, 1, 3))
# 12, because 4 + 1 + 7 = 12
```

Building the prefix array takes O(n) time. Each range-sum query then takes O(1) time.

Use prefix sums when many range-sum queries are required.

### Prefix sum with a hash map

Prefix sums can also solve problems involving negative numbers.

**Problem:** Count subarrays whose sum equals `k`.

```python
def subarray_sum(nums, k):
    prefix_count = {0: 1}
    prefix_sum = 0
    count = 0

    for num in nums:
        prefix_sum += num

        count += prefix_count.get(prefix_sum - k, 0)

        prefix_count[prefix_sum] = (
            prefix_count.get(prefix_sum, 0) + 1
        )

    return count
```

Example:

```python
print(subarray_sum([1, 1, 1], 2))
# 2
```

The hash map stores how many times each prefix sum has appeared. If the current prefix sum is `current`, an earlier prefix sum of `current - k` means the intervening subarray sums to `k`.

**Complexity:** O(n) expected time and O(n) extra space.

## 6. Pattern: Hash Map or Hash Set

Use a hash map or set when you need fast membership checks, frequency counting, or relationships between values.

### A. Two Sum

**Problem:** Return the indices of two elements whose sum equals the target.

```python
def two_sum(nums, target):
    seen = {}

    for i, num in enumerate(nums):
        needed = target - num

        if needed in seen:
            return [seen[needed], i]

        seen[num] = i

    return []
```

Example:

```python
print(two_sum([2, 7, 11, 15], 9))
# [0, 1]
```

Instead of checking every pair, store values already visited and look for the complement.

**Complexity:** O(n) expected time and O(n) extra space.

### B. Frequency counting

```python
def frequencies(nums):
    count = {}

    for num in nums:
        count[num] = count.get(num, 0) + 1

    return count
```

Example:

```python
print(frequencies([1, 2, 1, 3, 2, 1]))
# {1: 3, 2: 2, 3: 1}
```

Use this pattern for duplicate detection, anagrams, majority-related problems, and counting occurrences.

## 7. Pattern: Sorting

Sorting can simplify problems involving ordering, duplicates, intervals, and pairs.

### Example: Check whether duplicates exist

```python
def contains_duplicate(nums):
    ordered = sorted(nums)

    for i in range(1, len(ordered)):
        if ordered[i] == ordered[i - 1]:
            return True

    return False
```

**Complexity:** O(n log n) time and O(n) extra space because `sorted()` creates a new list.

An alternative uses a set:

```python
def contains_duplicate(nums):
    return len(set(nums)) != len(nums)
```

This is O(n) expected time and O(n) extra space.

**Choose sorting when:**

- The problem needs values in order.
- Adjacent comparisons can simplify the logic.
- You need to process intervals or merge ordered data.

**Choose a hash set when:** You only need to determine whether a value has appeared before.

## 8. Pattern: Binary Search

Use binary search when the search space is ordered or when a condition has a monotonic true/false boundary.

### Example: Find a target in a sorted array

```python
def binary_search(nums, target):
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
print(binary_search([1, 3, 5, 7, 9], 7))
# 3
```

At each step, eliminate half of the remaining search range.

**Complexity:** O(log n) time and O(1) extra space.

Common mistake: Binary search does not work on an arbitrary unsorted array unless the problem provides another suitable monotonic property.

## 9. Pattern: Kadane's Algorithm

Use Kadane's algorithm to find the maximum sum of a **non-empty contiguous subarray**.

```python
def max_subarray_sum(nums):
    if not nums:
        return None

    current = nums[0]
    best = nums[0]

    for num in nums[1:]:
        current = max(num, current + num)
        best = max(best, current)

    return best
```

Example:

```python
print(max_subarray_sum([-2, 1, -3, 4, -1, 2, 1, -5, 4]))
# 6
```

The maximum-sum subarray is `[4, -1, 2, 1]`, whose sum is `6`.

At every element, decide whether to:

- Start a new subarray at the current element.
- Extend the existing subarray.

**Complexity:** O(n) time and O(1) extra space.

Initializing both values to the first element handles arrays containing only negative numbers correctly.

## 10. Pattern: In-Place Reversal

Use two pointers to reverse an array without creating another array.

```python
def reverse_array(nums):
    left = 0
    right = len(nums) - 1

    while left < right:
        nums[left], nums[right] = nums[right], nums[left]
        left += 1
        right -= 1

    return nums
```

Example:

```python
print(reverse_array([1, 2, 3, 4]))
# [4, 3, 2, 1]
```

**Complexity:** O(n) time and O(1) extra space.

This modifies the original list. If you need to preserve it, use `nums[::-1]` or `list(reversed(nums))`, both of which create a new list.

## 11. Pattern: Difference Array

A difference array is useful when many range updates must be applied to an array.

Suppose an operation adds `value` to every element between indices `left` and `right`, inclusive. Instead of updating every element immediately, record changes at the boundaries and reconstruct the result with a prefix sum.

```python
def range_updates(n, updates):
    diff = [0] * (n + 1)

    for left, right, value in updates:
        diff[left] += value
        diff[right + 1] -= value

    result = []
    current = 0

    for i in range(n):
        current += diff[i]
        result.append(current)

    return result
```

Example:

```python
updates = [
    (1, 3, 2),
    (2, 4, 3)
]

print(range_updates(5, updates))
# [0, 2, 5, 5, 3]
```

The first update adds `2` to indices `1` through `3`. The second adds `3` to indices `2` through `4`.

**Complexity:** O(n + q) time and O(n) extra space, where `q` is the number of updates.

This is useful when there are many range updates and the final array is needed after all updates.

## 12. Pattern: Cyclic Placement

Some problems involve numbers that should occupy specific positions, commonly when the array contains values in the range `1` through `n`.

For example, value `1` belongs at index `0`, value `2` at index `1`, and so on.

### Example: Find the missing number from `1` through `n`

A simpler mathematical solution is:

```python
def missing_number(nums):
    n = len(nums) + 1
    expected = n * (n + 1) // 2
    return expected - sum(nums)
```

For an array containing distinct numbers from `1` through `n` with exactly one missing, the formula works in O(n) time and O(1) extra space.

Cyclic placement is useful for problems such as finding missing or duplicate values when the input has specific range and repetition constraints. Always verify those constraints before using it.

## 13. How to Recognize the Right Pattern

| Problem clue | Pattern to consider |
|---|---|
| Find a maximum, minimum, or count | Single pass |
| Sorted array and pair sum | Two pointers |
| Reverse or rearrange elements in place | Two pointers |
| Contiguous range of fixed length | Fixed sliding window |
| Longest or shortest valid contiguous range | Variable sliding window |
| Many range-sum queries | Prefix sum |
| Count subarrays with a target sum, including negative values | Prefix sum + hash map |
| Find complements or count frequencies | Hash map / set |
| Ordered search | Binary search |
| Maximum sum of a contiguous subarray | Kadane's algorithm |
| Many range updates | Difference array |
| Values constrained to `1..n` | Consider cyclic placement |

These are starting points, not rigid rules. Some problems combine multiple patterns.

## 14. Common Mistakes

1. **Using sliding windows with negative numbers without checking the logic.** A shrinking window often relies on positive values.
2. **Using two pointers on an unsorted array without a valid justification.** The usual pair-sum logic requires sorted values.
3. **Confusing a subarray with a subsequence.** A subarray must be contiguous; a subsequence does not have to be.
4. **Incorrect boundaries.** Check whether the right endpoint is inclusive and whether an empty array is possible.
5. **Overwriting values too early.** In-place modifications can destroy values that have not yet been processed.
6. **Ignoring duplicates.** Confirm whether duplicates are allowed and whether the output should contain indices or values.
7. **Forgetting negative numbers.** Initializing a maximum-sum calculation to zero incorrectly returns zero for an all-negative array when a non-empty subarray is required.
8. **Using nested loops unnecessarily.** Look for a hash map, two pointers, a sliding window, or prefix sums when the problem structure permits them.

## 15. Practice Problems

Try solving these without looking at solutions first.

### Easy

1. Find the maximum and minimum values in an array.
2. Reverse an array in place.
3. Move all zeroes to the end while preserving the order of other elements.
4. Remove duplicates from a sorted array in place.
5. Check whether a sorted array contains a pair with a given sum.
6. Find the missing number from `1` through `n`.

### Medium

1. Two Sum.
2. Maximum sum of a subarray of size `k`.
3. Maximum Subarray.
4. Minimum Size Subarray Sum.
5. Product of Array Except Self.
6. Count subarrays whose sum equals `k`.
7. Sort an array containing only `0`, `1`, and `2`.
8. Find the longest consecutive sequence.
9. Merge overlapping intervals.
10. Find the first and last position of a target in a sorted array.

### Suggested practice order

1. Start with single-pass problems and in-place reversal.
2. Learn two pointers and hash maps.
3. Practise fixed-size and variable-size sliding windows.
4. Move to prefix sums and Kadane's algorithm.
5. Finish with binary search, intervals, and combined patterns.

## Quick Revision

- **Single pass:** Examine each element once.
- **Two pointers:** Maintain two positions to reduce unnecessary comparisons.
- **Sliding window:** Reuse information while moving across a contiguous range.
- **Prefix sum:** Answer range-sum queries efficiently.
- **Hash map/set:** Store values or counts for fast lookups.
- **Sorting:** Use order to simplify comparisons.
- **Binary search:** Halve a suitable search space repeatedly.
- **Kadane's algorithm:** Track the best subarray ending at the current element.
- **Difference array:** Record range updates at boundaries.
- **Cyclic placement:** Use value-to-index relationships when the input constraints allow it.

The goal is not to memorize every solution. Learn to identify the structure of a problem, explain why the pattern works, and state its time and space complexity.