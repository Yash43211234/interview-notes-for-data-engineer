# DSA Patterns — Quick Revision Cheat Sheet

> **Purpose:** Recognize common Data Structures and Algorithms (DSA) patterns, understand when to use them, and solve coding interview problems efficiently using Python.

## 1. How to Recognize a Pattern

Before writing code, identify:

1. **Input:** Array, string, linked list, tree, graph, or number?
2. **Goal:** Search, count, optimize, find a pair, traverse, or enumerate?
3. **Constraints:** How large can the input be?
4. **Brute force:** What is the simplest correct solution?
5. **Repeated work:** Are you recalculating something?
6. **Structure:** Is the input sorted, contiguous, hierarchical, or connected?
7. **Pattern:** Which technique reduces the work?
8. **Complexity:** What are the time and space costs?

### Quick Pattern Selector

| Problem Clue                                      | Likely Pattern                             |
| ------------------------------------------------- | ------------------------------------------ |
| Sorted array, pair or boundary search             | Two pointers or binary search              |
| Contiguous subarray or substring                  | Sliding window or prefix sum               |
| Fast lookup, frequency, duplicates                | Hash map or set                            |
| Next greater/smaller element                      | Monotonic stack                            |
| Top K or repeatedly retrieve min/max              | Heap                                       |
| Shortest path in an unweighted graph              | BFS                                        |
| Explore paths or connected components             | DFS                                        |
| Prerequisites or dependency order                 | Topological sort                           |
| Many overlapping subproblems                      | Dynamic programming                        |
| Choose locally optimal options under a valid rule | Greedy                                     |
| Combine overlapping intervals                     | Sorting and interval merging               |
| Many connectivity queries                         | Disjoint Set Union (DSU)                   |
| Search all possible choices                       | Backtracking                               |
| Repeated range queries or updates                 | Prefix sums, Fenwick tree, or segment tree |

These clues suggest patterns; they do not prove that a pattern is correct. Verify the input assumptions and problem requirements.

---

## 2. Big-O Complexity

| Complexity   | Typical Example              |
| ------------ | ---------------------------- |
| (O(1))       | Array indexing               |
| (O(\log n))  | Binary search                |
| (O(n))       | One array traversal          |
| (O(n\log n)) | Efficient comparison sorting |
| (O(n^2))     | All pairs in an array        |
| (O(2^n))     | Enumerating all subsets      |
| (O(n!))      | Enumerating all permutations |

**Interview rule:** Explain both time and auxiliary space. Recursion, hash maps, heaps, and temporary arrays can require extra memory.

---

## 3. Hash Map and Hash Set

**Use when:** You need fast average-case lookup, counting, duplicate detection, or mapping a value to information.

Typical complexity: average (O(1)) lookup and insertion, with worst-case behavior potentially worse.

### Example: Two Sum

Find two indices whose values add up to a target.

```python
def two_sum(nums, target):
    seen = {}

    for i, num in enumerate(nums):
        need = target - num

        if need in seen:
            return [seen[need], i]

        seen[num] = i

    return []
```

- Time: (O(n)) average.
- Space: (O(n)).
- Pattern: Complement lookup.

**Common applications**

- Two Sum.
- Frequency counting.
- Duplicate detection.
- Anagram checks.
- Grouping records by key.

**Common mistake:** Storing the current value before checking the complement can allow an element to match itself in some implementations.

---

## 4. Two Pointers

**Use when:** The problem involves sorted data, pairs, opposite ends of an array, or in-place processing.

### Opposite-direction pointers

```python
def has_pair_sum(nums, target):
    left, right = 0, len(nums) - 1

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

- Time: (O(n)) after sorting.
- Extra space: (O(1)), excluding any sorting space.

**Why it works:** In a sorted array, if the sum is too small, moving the right pointer left cannot increase the sum. Moving the left pointer right can increase it.

### Same-direction pointers

Useful for:

- Removing duplicates from sorted arrays.
- Moving zeroes.
- Comparing sequences.
- In-place filtering.

**Common mistake:** Applying the sorted-array logic to unsorted input without first establishing the required ordering.

---

## 5. Sliding Window

**Use when:** The question involves a contiguous subarray or substring.

Two common forms:

- **Fixed-size window:** The window length is known.
- **Variable-size window:** Expand and shrink the window according to a condition.

### Fixed-size example: Maximum sum of length K

```python
def max_sum_subarray(nums, k):
    if k <= 0 or k > len(nums):
        return None

    window_sum = sum(nums[:k])
    best = window_sum

    for right in range(k, len(nums)):
        window_sum += nums[right]
        window_sum -= nums[right - k]
        best = max(best, window_sum)

    return best
```

- Time: (O(n)).
- Extra space: (O(1)).

### Variable-size example: Longest substring without repeating characters

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

- Time: (O(n)) average.
- Space: (O(k)), where (k) is the number of distinct characters tracked.

**Important limitation:** A standard shrinking sliding window is not automatically correct for every subarray-sum problem. Negative numbers can break the monotonic behavior needed by many window approaches; prefix sums or other methods may be required.

---

## 6. Prefix Sum

**Use when:** You need repeated range-sum queries or need to reason about sums ending at different positions.

Define:

[
prefix[i] = nums[0] + nums[1] + \cdots + nums[i-1]
]

Then the sum of the half-open range ([l,r)) is:

[
sum(l,r) = prefix[r] - prefix[l]
]

### Python

```python
def build_prefix(nums):
    prefix = [0]

    for num in nums:
        prefix.append(prefix[-1] + num)

    return prefix
```

Example:

```python
nums = [2, 4, 1, 5]
prefix = build_prefix(nums)

range_sum = prefix[3] - prefix[1]  # 4 + 1 = 5
```

- Build time: (O(n)).
- Each range query: (O(1)).
- Space: (O(n)).

### Prefix Sum + Hash Map

Useful for counting subarrays whose sum equals a target, including arrays containing negative numbers.

Key idea:

[
current_prefix - earlier_prefix = target
]

```python
def count_subarrays_sum_k(nums, k):
    counts = {0: 1}
    prefix = 0
    result = 0

    for num in nums:
        prefix += num
        result += counts.get(prefix - k, 0)
        counts[prefix] = counts.get(prefix, 0) + 1

    return result
```

- Time: (O(n)) average.
- Space: (O(n)).

**Common mistake:** Forgetting the initial `{0: 1}`, which accounts for a valid subarray beginning at index zero.

---

## 7. Binary Search

**Use when:** The search space is sorted, or a condition has a monotonic true/false boundary.

### Standard binary search

```python
def binary_search(nums, target):
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

- Time: (O(\log n)).
- Extra space: (O(1)).

### Binary Search on Answer

Useful when asking for the minimum feasible or maximum feasible value.

Examples:

- Minimum capacity needed to ship packages.
- Minimum speed needed to finish within a deadline.
- Maximum minimum distance between placements.

**Checklist**

1. Define the answer range.
2. Write a feasibility function.
3. Prove the condition is monotonic.
4. Decide which half to discard.
5. Maintain a clear loop invariant.
6. Test boundary cases.

**Common mistake:** Binary searching a condition that is not monotonic.

---

## 8. Sorting and Intervals

**Use when:** The task involves overlapping ranges, scheduling, or ordered events.

### Merge overlapping intervals

```python
def merge_intervals(intervals):
    if not intervals:
        return []

    intervals = sorted(intervals)
    merged = [intervals[0][:]]

    for start, end in intervals[1:]:
        if start <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])

    return merged
```

- Time: (O(n\log n)).
- Space: (O(n)) for the output, plus sorting overhead.

**Common applications**

- Merge Intervals.
- Meeting Rooms.
- Scheduling.
- Event ordering.
- Removing overlapping ranges.

**Common mistake:** Confusing touching intervals with overlapping intervals. Whether `[1, 3]` and `[3, 5]` merge depends on the problem's interval convention.

---

## 9. Stack and Monotonic Stack

**Use a stack when:** The most recent unresolved item should be handled first.

Common problems:

- Valid Parentheses.
- Expression parsing.
- Undo operations.
- Next greater element.
- Daily Temperatures.

### Monotonic stack example: Next greater element

```python
def next_greater_elements(nums):
    result = [-1] * len(nums)
    stack = []

    for i, num in enumerate(nums):
        while stack and nums[stack[-1]] < num:
            index = stack.pop()
            result[index] = num

        stack.append(i)

    return result
```

- Time: (O(n)).
- Space: (O(n)).

Each index is pushed and popped at most once.

**Common mistake:** Choosing `<` versus `<=` without checking whether equal values count as greater or block the next greater value.

---

## 10. Queue and Breadth-First Search (BFS)

**Use BFS when:** You need level-order traversal or shortest paths in an unweighted graph.

### Python

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

- Time: (O(V+E)) for an adjacency-list graph.
- Space: (O(V)), excluding graph storage.

**Important:** Mark nodes visited when enqueuing them to avoid inserting the same node repeatedly.

---

## 11. Depth-First Search (DFS)

**Use when:** You need to explore connected structures, traverse trees or graphs, or examine possible paths.

### Iterative DFS

```python
def dfs(graph, start):
    visited = set()
    stack = [start]
    order = []

    while stack:
        node = stack.pop()

        if node in visited:
            continue

        visited.add(node)
        order.append(node)

        for neighbor in graph[node]:
            if neighbor not in visited:
                stack.append(neighbor)

    return order
```

- Time: (O(V+E)) for an adjacency-list graph.
- Space: (O(V)).

Common applications:

- Number of Islands.
- Connected components.
- Tree traversal.
- Cycle detection with appropriate state tracking.
- Backtracking.

**Common mistake:** Assuming DFS always gives the shortest path. BFS is the usual choice for shortest paths in unweighted graphs.

---

## 12. Trees and Binary Search Trees

### Tree traversal patterns

| Traversal   | Order             | Typical Use                       |
| ----------- | ----------------- | --------------------------------- |
| Preorder    | Root, Left, Right | Copying or serializing a tree     |
| Inorder     | Left, Root, Right | Sorted order in a BST             |
| Postorder   | Left, Right, Root | Processing children before parent |
| Level order | Level by level    | BFS, minimum depth                |

### Binary Search Tree (BST)

For the usual unique-key BST convention:

- Values in the left subtree are smaller than the node.
- Values in the right subtree are larger than the node.

Duplicates require an explicitly defined convention.

Typical average search/insert/delete time is (O(\log n)) for a reasonably balanced BST, but worst-case time is (O(n)) in an unbalanced tree.

**Common mistake:** Checking only each node's immediate children instead of validating the full subtree constraints.

---

## 13. Heap and Top K

**Use when:** You repeatedly need the smallest or largest element, or only a limited number of best candidates.

Python's `heapq` provides a min-heap.

### K Largest Elements

```python
import heapq

def k_largest(nums, k):
    if k <= 0:
        return []

    return heapq.nlargest(k, nums)
```

A heap-based solution can achieve (O(n\log k)) time for maintaining the top (k) values; the exact implementation and output-order costs matter.

Typical uses:

- Kth Largest Element.
- Top K Frequent Elements.
- Merge K Sorted Lists.
- Scheduling by priority.

**Common mistake:** Sorting the entire dataset when the problem only requires a small number of best elements and a heap is more efficient.

---

## 14. Greedy Algorithms

**Use when:** A locally optimal decision can be shown to lead to a globally optimal solution.

Examples:

- Activity selection.
- Interval scheduling.
- Certain minimum-spanning-tree algorithms.
- Certain shortest-path algorithms with suitable constraints.

### How to Validate Greedy

1. State the greedy choice.
2. Explain why it is safe.
3. Prove an exchange argument or another correctness property.
4. Check edge cases.

**Important:** Greedy is not simply “choose the largest” or “choose the smallest.” The choice must be justified.

---

## 15. Backtracking

**Use when:** You need to explore combinations, permutations, or paths and reject choices that cannot lead to a valid solution.

General structure:

```python
def backtrack(state):
    if is_complete(state):
        record_solution(state)
        return

    for choice in available_choices(state):
        if not is_valid(choice, state):
            continue

        apply(choice, state)
        backtrack(state)
        undo(choice, state)
```

Typical problems:

- Subsets.
- Permutations.
- Combination Sum.
- N-Queens.
- Sudoku.

**Core idea:** Choose, explore, undo.

Complexity is often exponential because the algorithm may explore many combinations.

**Common mistake:** Forgetting to undo a mutable state or accidentally sharing the same mutable result across solutions.

---

## 16. Dynamic Programming (DP)

**Use when:** The problem has overlapping subproblems and an appropriate recurrence that allows reused results.

### Two Main Approaches

| Memoization                         | Tabulation                        |
| ----------------------------------- | --------------------------------- |
| Top-down recursion                  | Bottom-up iteration               |
| Stores previously computed states   | Builds states in dependency order |
| Often easier to derive              | Often avoids recursion overhead   |
| Uses recursion stack when recursive | Can often optimize space          |

### DP Design Checklist

1. Define the state.
2. Write the recurrence.
3. Establish base cases.
4. Determine evaluation order.
5. Identify the final answer.
6. Analyze time and space.
7. Reduce space only if dependencies permit it.

### Example: Fibonacci

```python
def fib(n):
    if n < 0:
        raise ValueError("n must be non-negative")

    if n < 2:
        return n

    previous, current = 0, 1

    for _ in range(2, n + 1):
        previous, current = current, previous + current

    return current
```

- Time: (O(n)).
- Extra space: (O(1)).

Common DP patterns:

- One-dimensional DP.
- Grid DP.
- Knapsack.
- Subsequence DP.
- Interval DP.
- Tree DP.
- State-machine DP.

**Common mistake:** Starting to code before defining exactly what `dp[i]` or `dp[i][j]` represents.

---

## 17. Graph Patterns

| Pattern               | Typical Problem                                                          |
| --------------------- | ------------------------------------------------------------------------ |
| BFS                   | Shortest path in an unweighted graph                                     |
| DFS                   | Connectivity and exploration                                             |
| Topological sort      | Dependency ordering in a DAG                                             |
| Dijkstra              | Single-source shortest paths with non-negative edge weights              |
| Bellman–Ford          | Shortest paths with negative edges; can detect reachable negative cycles |
| Union-Find / DSU      | Dynamic connectivity and cycle-related problems                          |
| Minimum Spanning Tree | Connect vertices with minimum total edge weight                          |
| Multi-source BFS      | Nearest-source distances in an unweighted graph                          |

### Topological Sort

Use when tasks have prerequisite relationships and the dependency graph is a directed acyclic graph (DAG).

If the algorithm cannot process every vertex, the graph contains a cycle under the standard Kahn's algorithm setup.

**Common mistake:** Applying topological sort to a cyclic dependency graph and expecting a valid ordering of all vertices.

---

## 18. Disjoint Set Union (Union-Find)

**Use when:** You need to maintain connected groups as edges are added.

Operations:

- `find(x)`: identify the representative of a set.
- `union(a, b)`: merge two sets.

With path compression and union by rank or size, operations have amortized near-constant complexity, conventionally expressed using the inverse Ackermann function.

Typical applications:

- Number of Connected Components.
- Cycle detection in undirected graphs.
- Kruskal's Minimum Spanning Tree algorithm.

**Common mistake:** Treating DSU as a general solution for shortest paths or arbitrary dynamic graph deletions.

---

## 19. Linked List Patterns

| Pattern                | Use                             |
| ---------------------- | ------------------------------- |
| Fast and slow pointers | Cycle detection, middle node    |
| Dummy node             | Simplify insertion and deletion |
| Reverse pointers       | Reverse a linked list           |
| Merge two pointers     | Merge sorted lists              |
| Track previous node    | Remove or reconnect nodes       |

### Cycle Detection

Floyd's tortoise-and-hare algorithm uses a slow pointer moving one step and a fast pointer moving two steps.

If they meet, a cycle exists.

- Time: (O(n)).
- Extra space: (O(1)).

**Common mistake:** Dereferencing `fast.next.next` without first checking that `fast` and `fast.next` exist.

---

## 20. Trie

**Use when:** Many strings share prefixes and prefix-based lookup is important.

Typical problems:

- Prefix search.
- Autocomplete.
- Word dictionary.
- Word Search II.

A trie can insert or search a word in (O(L)) time, where (L) is its length, assuming child lookup is constant-time or appropriately bounded.

Trade-off: It may use more memory than a hash set of complete strings.

---

## 21. Bit Manipulation

**Use when:** The problem involves binary representations, flags, parity, or bitwise properties.

| Operator | Meaning     |
| -------- | ----------- |
| `&`      | Bitwise AND |
| `\|`     | Bitwise OR  |
| `^`      | Bitwise XOR |
| `~`      | Bitwise NOT |
| `<<`     | Left shift  |
| `>>`     | Right shift |

Useful identities:

- `x & 1` checks the least-significant bit.
- `x ^ x == 0`.
- `x ^ 0 == x`.
- `x & (x - 1)` clears the lowest set bit for non-negative integers.
- `1 << k` represents (2^k) in the usual non-negative integer setting.

**Common mistake:** Assuming fixed-width signed-integer behavior in Python. Python integers have arbitrary precision, and bitwise behavior can differ from fixed-width languages.

---

## 22. Monotonic Queue

**Use when:** You need a running maximum or minimum over a sliding window.

A deque stores candidate indices in an order that allows expired or dominated candidates to be removed efficiently.

Typical problem: Sliding Window Maximum.

A well-designed monotonic deque can solve it in (O(n)) time.

**Common mistake:** Storing only values when the solution needs indices to determine whether an element has expired from the window.

---

## 23. Range Query Data Structures

| Structure        | Typical Capability                                                  |
| ---------------- | ------------------------------------------------------------------- |
| Prefix Sum       | Static range-sum queries                                            |
| Difference Array | Efficient range updates followed by reconstruction                  |
| Fenwick Tree     | Prefix sums with point updates                                      |
| Segment Tree     | Range queries and updates for supported operations                  |
| Sparse Table     | Static range queries for suitable operations, often idempotent ones |

Choose based on:

- Whether the input changes.
- Query type.
- Update type.
- Required time complexity.
- Implementation complexity.

---

## 24. Common Interview Mistakes

1. Choosing a pattern based only on a keyword.
2. Ignoring input constraints.
3. Using sliding window when negative values break its assumptions.
4. Using binary search without a sorted or monotonic search space.
5. Forgetting to initialize prefix sums with zero.
6. Marking graph nodes visited too late.
7. Assuming DFS returns the shortest path.
8. Forgetting a backtracking undo step.
9. Defining DP states vaguely.
10. Using a greedy strategy without proving it.
11. Ignoring duplicates and tie-breaking.
12. Modifying a list while iterating over it incorrectly.
13. Using `list.pop(0)` as a queue operation in performance-sensitive code.
14. Forgetting empty input and single-element cases.
15. Giving time complexity without space complexity.
16. Returning a value with the wrong meaning, such as an index instead of a value.
17. Failing to test the smallest valid input.
18. Claiming average-case hash-map complexity is a strict worst-case guarantee.

---

## 25. Pattern Recognition Practice

For each problem, identify the likely pattern before coding.

| Problem                                        | Likely Pattern                           |
| ---------------------------------------------- | ---------------------------------------- |
| Two Sum                                        | Hash map                                 |
| Valid Anagram                                  | Frequency counting                       |
| Best Time to Buy and Sell Stock                | Running minimum                          |
| Maximum Sum Subarray of Size K                 | Fixed sliding window                     |
| Longest Substring Without Repeating Characters | Variable sliding window                  |
| Subarray Sum Equals K                          | Prefix sum + hash map                    |
| Binary Search                                  | Binary search                            |
| Search in Rotated Sorted Array                 | Modified binary search                   |
| Merge Intervals                                | Sorting + intervals                      |
| Daily Temperatures                             | Monotonic stack                          |
| Top K Frequent Elements                        | Hash map + heap or bucket strategy       |
| Number of Islands                              | DFS or BFS                               |
| Binary Tree Level Order Traversal              | BFS                                      |
| Lowest Common Ancestor                         | Tree recursion or a specialized approach |
| Course Schedule                                | Cycle detection or topological sort      |
| Network Delay Time                             | Dijkstra for non-negative edge weights   |
| Redundant Connection                           | DSU                                      |
| Climbing Stairs                                | Dynamic programming                      |
| Coin Change                                    | Dynamic programming                      |
| Permutations                                   | Backtracking                             |
| Word Search                                    | Backtracking + DFS                       |
| Sliding Window Maximum                         | Monotonic deque                          |
| Kth Largest Element                            | Heap or selection algorithm              |
| Reverse Linked List                            | Pointer reversal                         |
| Linked List Cycle                              | Fast and slow pointers                   |

These are common approaches, not mandatory solutions for every variant.

---

## 26. A Repeatable Problem-Solving Workflow

1. Restate the problem in your own words.
2. Clarify inputs, outputs, and constraints.
3. Work through a small example manually.
4. Explain the brute-force approach.
5. Identify the repeated work or exploitable structure.
6. Choose a pattern and explain why it applies.
7. State the algorithm before coding.
8. Write clean Python code.
9. Test normal cases and edge cases.
10. State time and auxiliary-space complexity.

### Minimum Test Cases

- Empty input, if allowed.
- One element.
- Duplicate values.
- Negative values, if allowed.
- Already sorted input.
- Reverse-sorted input.
- No valid answer.
- Multiple valid answers.
- Maximum-size input.

---

## 27. Final Revision Checklist

- [ ] Hash maps and sets
- [ ] Two pointers
- [ ] Sliding window
- [ ] Prefix sums
- [ ] Binary search
- [ ] Sorting and intervals
- [ ] Stack and monotonic stack
- [ ] Queue and BFS
- [ ] DFS and tree traversals
- [ ] Heaps and Top K
- [ ] Greedy
- [ ] Backtracking
- [ ] Dynamic programming
- [ ] Graph algorithms
- [ ] Disjoint Set Union
- [ ] Linked list patterns
- [ ] Tries and bit manipulation
- [ ] Monotonic deque
- [ ] Range-query data structures
- [ ] Time and space complexity
- [ ] Edge-case testing

**Final reminder:** Do not memorize patterns as isolated tricks. Understand the condition that makes each pattern correct, then practice recognizing that condition in unfamiliar problems.
