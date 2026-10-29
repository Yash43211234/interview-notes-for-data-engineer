# Common DSA Problems and Patterns

This note collects frequently used problem-solving patterns for coding interviews. The goal is to recognize the structure of a problem, select an appropriate approach, and analyze its time and space complexity.

## 1. How to Approach Any DSA Problem

Follow this sequence before writing code.

1. **Understand the problem:** Identify the input, expected output, and constraints.
2. **Work through an example:** Trace the input manually.
3. **Start with a brute-force solution:** Find a correct approach first.
4. **Identify repeated work:** Look for unnecessary loops, repeated searches, or recalculations.
5. **Choose a pattern:** Hashing, two pointers, sliding window, binary search, stack, tree traversal, or dynamic programming.
6. **Check edge cases:** Empty input, one element, duplicates, negative values, and boundary conditions.
7. **Analyze complexity:** Explain the time and auxiliary space required.
8. **Test the implementation:** Trace the code on a small example.

**Interview principle:** A correct, explainable solution is more valuable than a complicated optimization you cannot justify.

## 2. Arrays and Hashing

Use these patterns when the problem involves collections, repeated values, counting, or finding relationships between elements.

### Pattern A: Hash Set

**Use when:** You need fast membership checks or need to identify duplicates.

Example: Determine whether an array contains duplicate values.

```python
def contains_duplicate(nums: list[int]) -> bool:
    seen = set()

    for num in nums:
        if num in seen:
            return True
        seen.add(num)

    return False
```

- Time: `O(n)` average
- Auxiliary space: `O(n)`

### Pattern B: Hash Map

**Use when:** You need frequencies, value-to-index mappings, or quick lookups.

Example: Two Sum.

```python
def two_sum(nums: list[int], target: int) -> list[int]:
    seen = {}

    for i, num in enumerate(nums):
        needed = target - num

        if needed in seen:
            return [seen[needed], i]

        seen[num] = i

    return []
```

- Time: `O(n)` average
- Auxiliary space: `O(n)`

Store previously visited values so each element can be checked without scanning the entire array again.

### Common problems

- Two Sum
- Contains Duplicate
- First Unique Character
- Group Anagrams
- Top K Frequent Elements
- Longest Consecutive Sequence

## 3. Two Pointers

**Use when:** You need to compare elements from different positions, process sorted arrays, or work inward from both ends.

Example: Check whether a string is a palindrome.

```python
def is_palindrome(s: str) -> bool:
    left, right = 0, len(s) - 1

    while left < right:
        if s[left] != s[right]:
            return False

        left += 1
        right -= 1

    return True
```

- Time: `O(n)`
- Auxiliary space: `O(1)`

### Common problems

- Valid Palindrome
- Two Sum II — Input Array Is Sorted
- Remove Duplicates from Sorted Array
- Move Zeroes
- Container With Most Water
- Three Sum

**Recognition clue:** The input is sorted, or a left/right scan can eliminate candidates efficiently.

## 4. Sliding Window

**Use when:** You need a contiguous subarray or substring and can update its state as the window moves.

Example: Maximum sum of a subarray of size `k`.

```python
def max_sum_subarray(nums: list[int], k: int) -> int:
    if k <= 0 or k > len(nums):
        raise ValueError("k must be between 1 and len(nums)")

    window_sum = sum(nums[:k])
    best = window_sum

    for right in range(k, len(nums)):
        window_sum += nums[right]
        window_sum -= nums[right - k]
        best = max(best, window_sum)

    return best
```

- Time: `O(n)`
- Auxiliary space: `O(1)`

Instead of recalculating each window's sum, add the new element and remove the outgoing element.

### Common problems

- Maximum Sum Subarray of Size K
- Longest Substring Without Repeating Characters
- Minimum Size Subarray Sum
- Permutation in String
- Minimum Window Substring

**Recognition clue:** The question asks about a contiguous range and the window can be maintained efficiently.

## 5. Prefix Sums

**Use when:** You need repeated range-sum queries or subarray sums.

Example: Build prefix sums.

```python
def build_prefix(nums: list[int]) -> list[int]:
    prefix = [0]

    for num in nums:
        prefix.append(prefix[-1] + num)

    return prefix
```

For the inclusive range from index `left` to `right`:

```python
range_sum = prefix[right + 1] - prefix[left]
```

- Building the prefix array: `O(n)`
- Each range-sum query: `O(1)`
- Auxiliary space: `O(n)`

### Common problems

- Range Sum Query
- Subarray Sum Equals K
- Find Pivot Index
- Product of Array Except Self (prefix/suffix variation)
- Difference Array Range Updates

**Recognition clue:** You repeatedly need sums over ranges or relationships between cumulative totals.

## 6. Binary Search

**Use when:** The search space is sorted, or a condition changes monotonically from false to true or true to false.

Example: Standard binary search.

```python
def binary_search(nums: list[int], target: int) -> int:
    left, right = 0, len(nums) - 1

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
- Auxiliary space: `O(1)`

### Common problems

- Binary Search
- Search Insert Position
- First and Last Position of Element
- Search in Rotated Sorted Array
- Find Peak Element
- Koko Eating Bananas

**Recognition clue:** Each step can eliminate roughly half the remaining search space, or a monotonic condition permits finding a boundary.

## 7. Stack and Monotonic Stack

**Use a stack when:** The problem involves nested structures, matching pairs, reversing processing order, or the most recent unresolved item.

Example: Valid Parentheses.

```python
def is_valid(s: str) -> bool:
    stack = []
    matching = {
        ")": "(",
        "]": "[",
        "}": "{"
    }

    for char in s:
        if char in matching:
            if not stack or stack.pop() != matching[char]:
                return False
        else:
            stack.append(char)

    return not stack
```

- Time: `O(n)`
- Auxiliary space: `O(n)`

**Use a monotonic stack when:** You need the next greater/smaller element or need to process unresolved elements in order.

### Common problems

- Valid Parentheses
- Min Stack
- Daily Temperatures
- Next Greater Element
- Largest Rectangle in Histogram
- Evaluate Reverse Polish Notation

**Recognition clue:** The answer for an element depends on the nearest previous or next element satisfying a condition.

## 8. Linked Lists

**Use these patterns when:** The problem involves node references, reversing links, detecting cycles, or finding a midpoint.

### Pattern A: Fast and slow pointers

Example: Detect a cycle.

```python
def has_cycle(head) -> bool:
    slow = fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

        if slow is fast:
            return True

    return False
```

- Time: `O(n)`
- Auxiliary space: `O(1)`

### Pattern B: Pointer reversal

Example: Reverse a singly linked list.

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
- Auxiliary space: `O(1)`

### Common problems

- Reverse Linked List
- Middle of the Linked List
- Linked List Cycle
- Merge Two Sorted Lists
- Remove Nth Node From End
- Reorder List

## 9. Trees and Binary Search Trees

**Use tree traversal when:** The problem involves hierarchical data, parent-child relationships, subtree properties, or paths.

### Depth-first search

Common traversal orders:

- Preorder: Node, Left, Right
- Inorder: Left, Node, Right
- Postorder: Left, Right, Node

Example: Maximum depth of a binary tree.

```python
def max_depth(root) -> int:
    if root is None:
        return 0

    return 1 + max(
        max_depth(root.left),
        max_depth(root.right)
    )
```

- Time: `O(n)`
- Auxiliary space: `O(h)` for recursion, where `h` is tree height.

### Breadth-first search

Use a queue to process nodes level by level.

Common applications:
- Level Order Traversal
- Minimum Depth of Binary Tree
- Right Side View of Binary Tree

### BST-specific patterns

Use BST ordering to solve:
- Search and insertion
- Validation using lower and upper bounds
- Minimum and maximum
- Lowest Common Ancestor
- Kth Smallest Element

**Recognition clue:** If the problem mentions levels, use BFS as a candidate. If it asks about subtrees, paths, or recursive properties, consider DFS.

## 10. Heaps and Priority Queues

**Use when:** You repeatedly need the smallest or largest element, or need to maintain the top K elements.

Python's `heapq` provides a min-heap.

Example: Find the K largest elements.

```python
import heapq

def k_largest(nums: list[int], k: int) -> list[int]:
    if not 0 <= k <= len(nums):
        raise ValueError("k must be between 0 and len(nums)")

    return heapq.nlargest(k, nums)
```

For a direct implementation with a heap of size `k`, each new element can be compared with the smallest retained element.

- Full sorting: `O(n log n)`
- Heap of size `k`: `O(n log k)` time, `O(k)` auxiliary space
- `heapq.nlargest` is a convenient library solution; its internal strategy may depend on `k` and input size.

### Common problems

- Kth Largest Element
- Top K Frequent Elements
- Merge K Sorted Lists
- Find Median From Data Stream
- Task Scheduler

## 11. Graphs

**Use graph algorithms when:** The input represents connections, dependencies, routes, networks, or relationships.

### BFS

Useful for shortest paths in an unweighted graph.

```python
from collections import deque

def bfs(graph, start):
    visited = {start}
    queue = deque([start])
    order = []

    while queue:
        node = queue.popleft()
        order.append(node)

        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

    return order
```

- Time: `O(V + E)` for an adjacency-list graph
- Auxiliary space: `O(V)`

### DFS

Useful for reachability, connected components, cycle detection, and many traversal problems.

### Other graph patterns

- Topological sort: dependency ordering in a directed acyclic graph.
- Union-Find: connectivity and merging components.
- Dijkstra's algorithm: shortest paths with non-negative edge weights.
- Backtracking: exploring paths and combinations under constraints.

### Common problems

- Number of Islands
- Clone Graph
- Course Schedule
- Rotting Oranges
- Number of Connected Components
- Network Delay Time

## 12. Backtracking

**Use when:** You must generate combinations, permutations, subsets, or paths and explore different choices.

The typical structure is:

1. Choose an option.
2. Explore recursively.
3. Undo the choice.
4. Try another option.

Example: Generate all subsets.

```python
def subsets(nums: list[int]) -> list[list[int]]:
    result = []
    path = []

    def backtrack(index):
        result.append(path.copy())

        for i in range(index, len(nums)):
            path.append(nums[i])
            backtrack(i + 1)
            path.pop()

    backtrack(0)
    return result
```

For `n` distinct elements, there are \(2^n\) subsets.

- Time: `O(n * 2^n)` including copying each subset
- Output space: `O(n * 2^n)` for the returned subsets

### Common problems

- Subsets
- Permutations
- Combination Sum
- Generate Parentheses
- Word Search
- N-Queens

**Recognition clue:** You need to enumerate valid possibilities, and choices can be undone when exploring another branch.

## 13. Dynamic Programming (DP)

**Use when:** A problem has overlapping subproblems and an optimal solution or count can be built from smaller states.

Two common approaches:

- **Memoization:** Top-down recursion with cached results.
- **Tabulation:** Bottom-up calculation using a table or a small number of variables.

Example: Fibonacci with constant extra space.

```python
def fib(n: int) -> int:
    if n < 0:
        raise ValueError("n must be non-negative")

    previous, current = 0, 1

    for _ in range(n):
        previous, current = current, previous + current

    return previous
```

- Time: `O(n)`
- Auxiliary space: `O(1)`

### Common problems

- Climbing Stairs
- House Robber
- Coin Change
- Longest Increasing Subsequence
- Longest Common Subsequence
- 0/1 Knapsack
- Edit Distance

**Recognition clue:** The problem can be expressed using a state and a recurrence that reuses answers to smaller subproblems.

## 14. Greedy Algorithms

**Use when:** A locally optimal choice can be shown to lead to a globally optimal solution.

Example: Select the maximum number of non-overlapping intervals by sorting by end time and repeatedly choosing the next compatible interval.

```python
def max_non_overlapping(intervals):
    intervals = sorted(intervals, key=lambda item: item[1])

    count = 0
    last_end = float("-inf")

    for start, end in intervals:
        if start >= last_end:
            count += 1
            last_end = end

    return count
```

This implementation treats intervals as compatible when one starts at the exact time the previous one ends.

- Time: `O(n log n)` due to sorting
- Auxiliary space: depends on the sorting implementation; Python sorting uses additional memory.

### Common problems

- Non-overlapping Intervals
- Jump Game
- Gas Station
- Assign Cookies
- Minimum Number of Arrows to Burst Balloons

**Important:** A greedy approach needs a correctness argument. A locally attractive choice is not automatically optimal.

## 15. Interval Problems

**Use when:** The input contains start/end ranges and asks about overlap, merging, or scheduling.

A common approach is to sort intervals by start time and merge overlapping ranges.

```python
def merge_intervals(intervals):
    if not intervals:
        return []

    intervals.sort(key=lambda interval: interval[0])
    merged = [intervals[0][:]]

    for start, end in intervals[1:]:
        if start <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])

    return merged
```

- Time: `O(n log n)`
- Auxiliary space: `O(n)` for the output, excluding sorting details.

### Common problems

- Merge Intervals
- Insert Interval
- Meeting Rooms
- Meeting Rooms II
- Non-overlapping Intervals

## 16. Bit Manipulation

**Use when:** The problem involves binary representation, parity, XOR, or efficient bit-level checks.

Useful operations:

```python
# Check whether bit k is set
is_set = (n & (1 << k)) != 0

# Turn on bit k
n |= 1 << k

# Turn off bit k
n &= ~(1 << k)

# Toggle bit k
n ^= 1 << k

# Remove the lowest set bit
n &= n - 1
```

For a non-negative integer `n`, `n & (n - 1)` removes its rightmost set bit.

### Common problems

- Single Number
- Number of Set Bits
- Power of Two
- Counting Bits
- Missing Number

## 17. Pattern Recognition Summary

| Problem clue | Pattern to consider |
|---|---|
| Fast membership or frequency counts | Hash set / hash map |
| Sorted input and inward scanning | Two pointers |
| Contiguous subarray or substring | Sliding window |
| Repeated range-sum queries | Prefix sums |
| Sorted search space or monotonic condition | Binary search |
| Nested structures or nearest greater/smaller | Stack |
| Linked-list midpoint or cycle | Fast and slow pointers |
| Hierarchy or subtree properties | Tree DFS |
| Level-by-level traversal | BFS |
| Repeated minimum/maximum selection | Heap |
| Connections and dependencies | Graph algorithms |
| All combinations or permutations | Backtracking |
| Reused subproblems and state transitions | Dynamic programming |
| Provably optimal local choices | Greedy |
| Overlapping time ranges | Interval sorting and merging |
| Binary representation | Bit manipulation |

These are starting points, not rigid rules. A single problem may combine several patterns.

## 18. Complexity Reference

| Complexity | Typical example |
|---|---|
| `O(1)` | Array access by index |
| `O(log n)` | Binary search in a sorted array |
| `O(n)` | One pass through an array |
| `O(n log n)` | Comparison-based sorting |
| `O(n²)` | Two nested loops over the same input |
| `O(2^n)` | Enumerating all subsets |
| `O(n!)` | Enumerating all permutations |

The actual complexity depends on the algorithm, data structure, and input constraints.

## 19. Interview Practice Strategy

For each problem, record these five things in your notes:

1. **Pattern:** Which approach fits?
2. **Brute force:** What is the simplest correct solution?
3. **Optimization:** What repeated work can be removed?
4. **Complexity:** What are the time and auxiliary-space costs?
5. **Edge cases:** Which inputs might break the implementation?

When practicing, first try to identify the pattern without looking at the solution. If you are stuck, take a hint about the pattern, implement it yourself, and then compare your solution with the reference.

### Final revision

The most useful interview skill is not memorizing dozens of solutions. It is learning to recognize the underlying pattern, explain why it works, implement it correctly, and analyze its complexity.