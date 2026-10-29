# Common Problems in Coding Interviews

## 1. Overview

Common coding problems test whether you can recognize a problem pattern and choose an efficient solution.

Instead of memorizing individual solutions, learn to identify:
- The input structure: array, string, linked list, tree, graph, or matrix.
- The operation: search, count, sort, compare, or traverse.
- The constraint: time limit, memory limit, or input size.
- The pattern: hashing, two pointers, sliding window, binary search, or dynamic programming.

The same pattern often solves many different problems.

---

## 2. Find Duplicates

**Problem:** Given an array, determine whether any value appears more than once.

### Approach 1: Nested loops

Compare each element with every element after it.

```python
def contains_duplicate(nums):
    for i in range(len(nums)):
        for j in range(i + 1, len(nums)):
            if nums[i] == nums[j]:
                return True
    return False
```

- Time: `O(n²)`
- Extra space: `O(1)`

### Approach 2: Set

```python
def contains_duplicate(nums):
    seen = set()

    for num in nums:
        if num in seen:
            return True
        seen.add(num)

    return False
```

- Expected time: `O(n)`
- Extra space: `O(n)`

**Pattern:** Hashing.

**Interview takeaway:** Use a set when you need to track which values have already appeared.

---

## 3. Two Sum

**Problem:** Find two indices whose corresponding values add up to a target.

```python
def two_sum(nums, target):
    seen = {}

    for i, num in enumerate(nums):
        complement = target - num

        if complement in seen:
            return [seen[complement], i]

        seen[num] = i

    return []
```

Example:

```python
nums = [2, 7, 11, 15]
target = 9

print(two_sum(nums, target))
# [0, 1]
```

How it works:

1. Calculate the number needed to reach the target.
2. Check whether that number is already in the dictionary.
3. If found, return both indices.
4. Otherwise, store the current value and index.

- Time: `O(n)` expected
- Extra space: `O(n)`

**Pattern:** Hash map.

**Common mistake:** Store the current number before checking its complement. That can incorrectly use the same element twice in some cases.

---

## 4. Reverse a String

**Problem:** Reverse a string.

```python
def reverse_string(s):
    return s[::-1]
```

A manual approach:

```python
def reverse_string(s):
    result = []

    for char in s:
        result.insert(0, char)

    return "".join(result)
```

The manual approach above repeatedly inserts at the beginning of a list, so it can take `O(n²)` time.

A more efficient manual approach:

```python
def reverse_string(s):
    result = []

    for char in s:
        result.append(char)

    result.reverse()
    return "".join(result)
```

- Time: `O(n)`
- Extra space: `O(n)`

**Pattern:** String manipulation.

**Interview takeaway:** Understand the difference between concise syntax and the complexity of the underlying operation.

---

## 5. Check for a Palindrome

**Problem:** Determine whether a string reads the same forward and backward.

```python
def is_palindrome(s):
    return s == s[::-1]
```

For a version that ignores punctuation, spaces, and letter case:

```python
def is_palindrome(s):
    cleaned = "".join(
        char.lower()
        for char in s
        if char.isalnum()
    )

    return cleaned == cleaned[::-1]
```

Example:

```python
print(is_palindrome("A man, a plan, a canal: Panama"))
# True
```

- Time: `O(n)`
- Extra space: `O(n)`

**Pattern:** Two pointers or string reversal.

For a two-pointer solution, compare characters from both ends and move inward.

---

## 6. Find the Maximum and Minimum

**Problem:** Find the largest and smallest values without sorting the array.

```python
def find_min_max(nums):
    if not nums:
        return None

    smallest = largest = nums[0]

    for num in nums[1:]:
        if num < smallest:
            smallest = num

        if num > largest:
            largest = num

    return smallest, largest
```

- Time: `O(n)`
- Extra space: `O(1)`

**Common mistake:** Sorting first. Sorting takes `O(n log n)` time when a single traversal is sufficient.

**Pattern:** Array traversal.

---

## 7. Find the Second-Largest Distinct Element

**Problem:** Return the second-largest distinct value.

```python
def second_largest(nums):
    largest = None
    second = None

    for num in nums:
        if largest is None or num > largest:
            second = largest
            largest = num
        elif num != largest and (
            second is None or num > second
        ):
            second = num

    return second
```

Example:

```python
print(second_largest([10, 5, 10, 8]))
# 8
```

- Time: `O(n)`
- Extra space: `O(1)`

The function returns `None` if no second-largest distinct value exists.

**Pattern:** Single-pass tracking.

**Common mistake:** Assuming the second array element is the second-largest value.

---

## 8. Move Zeros to the End

**Problem:** Move all zeros to the end while preserving the relative order of nonzero values.

```python
def move_zeros(nums):
    position = 0

    for i in range(len(nums)):
        if nums[i] != 0:
            nums[position] = nums[i]
            position += 1

    while position < len(nums):
        nums[position] = 0
        position += 1

    return nums
```

Example:

```python
print(move_zeros([0, 1, 0, 3, 12]))
# [1, 3, 12, 0, 0]
```

- Time: `O(n)`
- Extra space: `O(1)` excluding the returned reference.

**Pattern:** Two pointers / write pointer.

The read pointer scans every element. The write pointer marks where the next nonzero element belongs.

---

## 9. Remove Duplicates from a Sorted Array

**Problem:** Keep each distinct value once in a sorted array.

```python
def remove_duplicates(nums):
    if not nums:
        return 0

    write = 1

    for read in range(1, len(nums)):
        if nums[read] != nums[write - 1]:
            nums[write] = nums[read]
            write += 1

    return write
```

Example:

```python
nums = [1, 1, 2, 2, 3]
length = remove_duplicates(nums)

print(length)       # 3
print(nums[:length])  # [1, 2, 3]
```

- Time: `O(n)`
- Extra space: `O(1)`

**Pattern:** Two pointers.

**Important:** This function modifies the input array and returns the valid length. Values beyond that length should be ignored.

---

## 10. Find the Missing Number

**Problem:** An array contains distinct numbers from `0` through `n`, with one number missing. Find the missing number.

### Sum approach

```python
def missing_number(nums):
    n = len(nums)
    expected = n * (n + 1) // 2
    return expected - sum(nums)
```

- Time: `O(n)`
- Extra space: `O(1)` auxiliary space.

### XOR approach

```python
def missing_number(nums):
    result = len(nums)

    for i, num in enumerate(nums):
        result ^= i ^ num

    return result
```

XOR cancels identical values because `x ^ x == 0`.

**Pattern:** Mathematical reasoning / bit manipulation.

**Common mistake:** Using the formula without verifying that the input satisfies the stated constraints.

---

## 11. Maximum Subarray Sum

**Problem:** Find the maximum sum of any nonempty contiguous subarray.

Use Kadane's algorithm.

```python
def max_subarray(nums):
    if not nums:
        return None

    current = best = nums[0]

    for num in nums[1:]:
        current = max(num, current + num)
        best = max(best, current)

    return best
```

Example:

```python
print(max_subarray([-2, 1, -3, 4, -1, 2, 1, -5, 4]))
# 6
```

The best subarray is `[4, -1, 2, 1]`.

- Time: `O(n)`
- Extra space: `O(1)`

**Pattern:** Dynamic programming / Kadane's algorithm.

**Important:** Initialize with the first element, not zero. Otherwise, an array containing only negative numbers can incorrectly produce zero.

---

## 12. Binary Search

**Problem:** Find a target in a sorted array.

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

- Time: `O(log n)`
- Extra space: `O(1)`

**Pattern:** Binary search.

**Common mistakes:**
- Applying ordinary binary search to an unsorted array.
- Updating `left` or `right` incorrectly.
- Using the wrong loop condition.
- Forgetting that the target may not exist.

---

## 13. Merge Two Sorted Arrays

**Problem:** Merge two sorted arrays into one sorted array.

```python
def merge_sorted(a, b):
    i = j = 0
    result = []

    while i < len(a) and j < len(b):
        if a[i] <= b[j]:
            result.append(a[i])
            i += 1
        else:
            result.append(b[j])
            j += 1

    result.extend(a[i:])
    result.extend(b[j:])

    return result
```

- Time: `O(n + m)`
- Extra space: `O(n + m)`

**Pattern:** Two pointers.

**Interview takeaway:** Since both arrays are already sorted, there is no need to sort the combined result again.

---

## 14. Find the First Non-Repeating Character

**Problem:** Return the first character that occurs exactly once.

```python
from collections import Counter

def first_unique_char(s):
    counts = Counter(s)

    for char in s:
        if counts[char] == 1:
            return char

    return None
```

Example:

```python
print(first_unique_char("swiss"))
# "w"
```

- Time: `O(n)` expected
- Extra space: `O(k)`, where `k` is the number of distinct characters.

**Pattern:** Frequency counting / hashing.

**Important:** The second traversal preserves the original order. A frequency dictionary alone does not directly identify the first unique character unless you also consider the input order.

---

## 15. Valid Parentheses

**Problem:** Determine whether brackets are correctly matched and nested.

```python
def is_valid(s):
    stack = []
    pairs = {
        ")": "(",
        "]": "[",
        "}": "{"
    }

    for char in s:
        if char in "([{":
            stack.append(char)
        else:
            if not stack or stack.pop() != pairs.get(char):
                return False

    return len(stack) == 0
```

- Time: `O(n)`
- Extra space: `O(n)`

**Pattern:** Stack.

A closing bracket must match the most recently opened bracket. This is why a stack works.

**Common mistake:** Checking only whether the total counts of opening and closing brackets match. Correct nesting also matters.

---

## 16. Longest Substring Without Repeating Characters

**Problem:** Find the length of the longest substring containing no repeated characters.

```python
def longest_unique_substring(s):
    last_seen = {}
    left = 0
    best = 0

    for right, char in enumerate(s):
        if char in last_seen and last_seen[char] >= left:
            left = last_seen[char] + 1

        last_seen[char] = right
        best = max(best, right - left + 1)

    return best
```

Example:

```python
print(longest_unique_substring("abcabcbb"))
# 3
```

- Time: `O(n)` expected
- Extra space: `O(k)`

**Pattern:** Sliding window + hash map.

**Important:** Move `left` only when the previous occurrence is inside the current window. Otherwise, the window could move backward.

---

## 17. Product of Array Except Self

**Problem:** For each position, return the product of every other element without using division.

```python
def product_except_self(nums):
    n = len(nums)
    result = [1] * n

    prefix = 1

    for i in range(n):
        result[i] = prefix
        prefix *= nums[i]

    suffix = 1

    for i in range(n - 1, -1, -1):
        result[i] *= suffix
        suffix *= nums[i]

    return result
```

Example:

```python
print(product_except_self([1, 2, 3, 4]))
# [24, 12, 8, 6]
```

- Time: `O(n)`
- Extra auxiliary space: `O(1)`, excluding the output array.

**Pattern:** Prefix and suffix products.

**Common mistake:** Using the total product and division. That fails when zeros are present and may violate the problem's requirements.

---

## 18. Group Anagrams

**Problem:** Group words that contain the same letters with the same frequencies.

```python
from collections import defaultdict

def group_anagrams(words):
    groups = defaultdict(list)

    for word in words:
        key = tuple(sorted(word))
        groups[key].append(word)

    return list(groups.values())
```

Example:

```python
words = ["eat", "tea", "tan", "ate", "nat", "bat"]
print(group_anagrams(words))
```

The order of groups is not guaranteed by the problem unless explicitly required.

- Time: approximately `O(n * k log k)`, where `n` is the number of words and `k` is their maximum length.
- Extra space: proportional to the stored keys and grouped words.

**Pattern:** Hashing + canonical representation.

For lowercase English letters, a 26-element frequency tuple can be used as a key instead of sorting each word.

---

## 19. Merge Overlapping Intervals

**Problem:** Merge intervals that overlap.

```python
def merge_intervals(intervals):
    if not intervals:
        return []

    intervals.sort(key=lambda interval: interval[0])
    merged = [intervals[0][:]]

    for start, end in intervals[1:]:
        last = merged[-1]

        if start <= last[1]:
            last[1] = max(last[1], end)
        else:
            merged.append([start, end])

    return merged
```

Example:

```python
print(merge_intervals([[1, 3], [2, 6], [8, 10], [9, 12]]))
# [[1, 6], [8, 12]]
```

- Time: `O(n log n)` because of sorting.
- Extra space: `O(n)` for the merged output.

**Pattern:** Sorting + interval processing.

**Common mistake:** Comparing a new interval with the original previous interval instead of the most recently merged interval.

---

## 20. Reverse a Linked List

**Problem:** Reverse a singly linked list.

Assume each node has `val` and `next` attributes.

```python
def reverse_list(head):
    previous = None
    current = head

    while current:
        next_node = current.next
        current.next = previous
        previous = current
        current = next_node

    return previous
```

- Time: `O(n)`
- Extra space: `O(1)`

**Pattern:** Pointer manipulation.

**Critical detail:** Save `current.next` before changing it. Otherwise, the remaining list becomes inaccessible.

---

## 21. Tree Level-Order Traversal

**Problem:** Visit tree nodes level by level.

Assume each node has `val`, `left`, and `right` attributes.

```python
from collections import deque

def level_order(root):
    if not root:
        return []

    queue = deque([root])
    result = []

    while queue:
        level = []

        for _ in range(len(queue)):
            node = queue.popleft()
            level.append(node.val)

            if node.left:
                queue.append(node.left)

            if node.right:
                queue.append(node.right)

        result.append(level)

    return result
```

- Time: `O(n)`
- Extra space: `O(n)` in the worst case.

**Pattern:** Breadth-first search (BFS) + queue.

**Common mistake:** Using a list with repeated `pop(0)`, which requires shifting elements and can make traversal unnecessarily expensive.

---

## 22. Count Connected Components in a Grid

**Problem:** Given a grid of land (`"1"`) and water (`"0"`), count the number of connected land regions. Assume horizontal and vertical neighbors are connected.

```python
def count_islands(grid):
    if not grid or not grid[0]:
        return 0

    rows = len(grid)
    cols = len(grid[0])
    islands = 0

    def dfs(r, c):
        if (
            r < 0 or r >= rows
            or c < 0 or c >= cols
            or grid[r][c] != "1"
        ):
            return

        grid[r][c] = "0"

        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == "1":
                islands += 1
                dfs(r, c)

    return islands
```

- Time: `O(rows * cols)`
- Extra space: up to `O(rows * cols)` due to recursion in the worst case.

**Pattern:** Graph traversal / DFS.

**Important:** This implementation modifies the input grid. For very large grids, iterative DFS or BFS may be safer because Python has a recursion-depth limit.

---

## 23. Common Problem Patterns at a Glance

| Problem clue | Likely pattern |
|---|---|
| Find duplicates or pair sums | Hashing |
| Find a contiguous range | Sliding window or prefix sums |
| Compare values from both ends | Two pointers |
| Search a sorted array | Binary search |
| Next greater element or matching brackets | Stack |
| Level-by-level tree traversal | BFS + queue |
| Explore connected regions | DFS or BFS |
| Maximum subarray sum | Kadane's algorithm |
| Repeated choices with overlapping subproblems | Dynamic programming |
| Merge overlapping ranges | Sorting + intervals |
| Count occurrences | Hash map / frequency counter |
| Generate all combinations | Backtracking |
| Process tasks in priority order | Heap / priority queue |

These are starting clues, not absolute rules. The exact constraints determine the best approach.

---

## 24. How to Solve an Interview Problem

Use this process before writing code.

1. **Clarify the requirements.** Identify the input, expected output, duplicates, empty inputs, and edge cases.
2. **Work through a small example.** Trace the expected behavior manually.
3. **Start with a correct approach.** A simple brute-force solution helps establish correctness.
4. **Identify repeated work.** Ask whether hashing, sorting, two pointers, or a different data structure can remove it.
5. **State complexity.** Explain both time and auxiliary space.
6. **Test edge cases.** Try empty input, one element, duplicates, negative values, and missing results where applicable.
7. **Explain trade-offs.** An `O(n)` solution using `O(n)` space may be preferable to an `O(n²)` solution using `O(1)` space.

## 25. Quick Revision

- Use a **set** for membership checks and duplicate detection.
- Use a **dictionary** for value-to-index mappings and frequency counting.
- Use **two pointers** when the positions can move through the input in a coordinated way.
- Use a **sliding window** for many contiguous substring and subarray problems.
- Use **prefix sums** when range-sum queries or cumulative totals are involved.
- Use a **stack** for nested structures and last-in-first-out processing.
- Use **BFS** for level-order traversal and shortest paths in unweighted graphs.
- Use **DFS** to explore connected regions and paths.
- Use **binary search** when a sorted structure or monotonic condition allows the search space to shrink.
- Use **dynamic programming** when a problem has overlapping subproblems and reusable states.

**Final principle:** Learn to recognize the pattern, explain why it works, and analyze its complexity. That is more valuable than memorizing a large number of solutions.