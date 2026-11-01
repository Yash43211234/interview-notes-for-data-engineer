# Common Coding Mistakes in Python and DSA Interviews

## 1. Why Coding Mistakes Matter

In coding interviews, knowing the correct algorithm is only part of the solution. Many incorrect submissions happen because of small implementation mistakes.

Common causes include:

- Misunderstanding the problem.
- Incorrect loop boundaries.
- Missing edge cases.
- Incorrect pointer movement.
- Mishandling duplicates.
- Incorrect recursion base cases.
- Choosing an inefficient data structure.
- Failing to analyze time and space complexity.
- Not testing the final code.

The goal is to write code that is correct, efficient, readable, and reliable.

---

## 2. Problem Understanding Mistakes

### 2.1 Starting to Code Without Understanding the Problem

**Mistake:** Reading the question quickly and immediately writing code.

This can lead to solving a different problem from the one asked.

**Better approach:** Identify:

1. Input format.
2. Expected output.
3. Constraints.
4. Whether duplicates are allowed.
5. Whether the input is sorted.
6. Whether negative values are possible.
7. What should happen when no answer exists.

### 2.2 Ignoring Input Constraints

Consider an input of size \(n = 100,000\).

A nested-loop solution with O(n²) complexity may require an impractical number of operations.

Before selecting an algorithm, estimate how its work grows with input size.

| Input size | Often suitable approaches |
|---|---|
| Very small | Brute force may be acceptable |
| Around 1,000 | O(n²) may be acceptable, depending on limits |
| Around 100,000 | O(n log n) or O(n) is often preferred |
| Around 1,000,000 | O(n) or O(n log n) with efficient implementation |

These are rough guidelines, not guaranteed limits. Actual constraints depend on the time limit, language, and operation cost.

### 2.3 Confusing Similar Requirements

Be careful with these distinctions:

| Requirement | Meaning |
|---|---|
| Substring | Contiguous characters |
| Subsequence | Characters in original order, not necessarily contiguous |
| Subarray | Contiguous array elements |
| Subset | Any selection of elements |
| Distinct values | Duplicate values count once |
| All occurrences | Every matching occurrence matters |
| First occurrence | Earliest matching position |
| Any occurrence | Any valid matching position |

Always clarify the exact requirement before choosing a pattern.

---

## 3. Python Implementation Mistakes

### 3.1 Mutable Default Arguments

**Incorrect:**

```python
def add_item(item, items=[]):
    items.append(item)
    return items

print(add_item(1))  # [1]
print(add_item(2))  # [1, 2]
```

The default list is created once when the function is defined and reused across calls.

**Correct:**

```python
def add_item(item, items=None):
    if items is None:
        items = []

    items.append(item)
    return items
```

**Remember:** Use `None` when a function needs a fresh mutable default value for each call.

### 3.2 Confusing Assignment and Copying

**Incorrect assumption:**

```python
a = [1, 2, 3]
b = a

b.append(4)

print(a)
# [1, 2, 3, 4]
```

Both variables refer to the same list.

**Correct for a shallow copy:**

```python
b = a.copy()
```

For nested objects, use `copy.deepcopy()` when an independent recursive copy is required.

### 3.3 Using `is` Instead of `==`

**Incorrect for value comparison:**

```python
a = [1, 2]
b = [1, 2]

print(a is b)  # False
```

**Correct:**

```python
print(a == b)  # True
```

- `==` compares values.
- `is` compares object identity.

Use `is None` to check for `None`.

### 3.4 Modifying a List While Iterating

**Risky approach:**

```python
numbers = [1, 2, 3, 4, 5]

for number in numbers:
    if number % 2 == 0:
        numbers.remove(number)
```

Removing items changes list positions and can cause elements to be skipped.

**Better approach:**

```python
numbers = [1, 2, 3, 4, 5]

result = [number for number in numbers if number % 2 != 0]
```

### 3.5 Using `and` Instead of `&` in PySpark or Pandas Expressions

Python's `and` and `or` expect scalar truth values. They are not suitable for combining distributed or vectorized column conditions.

**Incorrect:**

```python
df.filter(
    (df.age > 18) and (df.salary > 30000)
)
```

**Correct PySpark:**

```python
from pyspark.sql.functions import col

df.filter(
    (col("age") > 18) & (col("salary") > 30000)
)
```

Use:
- `&` for AND.
- `|` for OR.
- `~` for NOT.

Wrap each condition in parentheses.

### 3.6 Forgetting That `sort()` Returns None

**Incorrect:**

```python
numbers = [3, 1, 2]

result = numbers.sort()

print(result)
# None
```

**Correct:**

```python
numbers.sort()

# Or create a new sorted list
result = sorted(numbers)
```

### 3.7 Confusing `append()` and `extend()`

```python
a = [1, 2]
a.append([3, 4])

print(a)
# [1, 2, [3, 4]]
```

```python
b = [1, 2]
b.extend([3, 4])

print(b)
# [1, 2, 3, 4]
```

- `append()` adds one object.
- `extend()` adds elements from an iterable.

### 3.8 Forgetting That Strings Are Immutable

**Incorrect assumption:**

```python
text = "hello"
text[0] = "H"
```

This raises a `TypeError`.

**Correct:**

```python
text = "H" + text[1:]
```

Or use a list of characters when multiple modifications are required.

### 3.9 Using a List Instead of a Set for Repeated Membership Checks

**Less efficient:**

```python
if value in large_list:
    print("Found")
```

A list membership check is O(n).

**Often better:**

```python
values = set(large_list)

if value in values:
    print("Found")
```

Set membership is O(1) on average, at the cost of additional memory and set construction.

---

## 4. Loop and Boundary Mistakes

### 4.1 Off-by-One Errors

**Incorrect:**

```python
numbers = [10, 20, 30, 40, 50]

for i in range(len(numbers) - 1):
    print(numbers[i])
```

This intentionally or accidentally skips the final element.

**Correct when all elements are needed:**

```python
for i in range(len(numbers)):
    print(numbers[i])
```

Alternatively:

```python
for number in numbers:
    print(number)
```

### 4.2 Incorrect `while` Loop Condition

**Incorrect:**

```python
i = 0

while i <= len(numbers):
    print(numbers[i])
    i += 1
```

The final iteration accesses an invalid index.

**Correct:**

```python
i = 0

while i < len(numbers):
    print(numbers[i])
    i += 1
```

### 4.3 Forgetting to Update a Pointer

**Incorrect binary-search logic:**

```python
if numbers[mid] < target:
    left = mid
```

If `mid` remains unchanged, the loop may never progress.

**Correct:**

```python
if numbers[mid] < target:
    left = mid + 1
```

Every loop must make progress toward its termination condition.

### 4.4 Incorrect Loop Boundaries in Nested Loops

When comparing pairs, determine whether the same element should be compared with itself and whether pair order matters.

For unique index pairs:

```python
for i in range(len(numbers)):
    for j in range(i + 1, len(numbers)):
        print(numbers[i], numbers[j])
```

This avoids:
- Comparing an element with itself.
- Processing both `(i, j)` and `(j, i)`.

---

## 5. Array and String Mistakes

### 5.1 Not Handling Empty Arrays

**Risky:**

```python
def find_largest(numbers):
    largest = numbers[0]
    return largest
```

This fails for an empty list.

**Better:**

```python
def find_largest(numbers):
    if not numbers:
        raise ValueError("Input cannot be empty")

    largest = numbers[0]

    for number in numbers:
        if number > largest:
            largest = number

    return largest
```

Choose the expected behavior based on the problem statement.

### 5.2 Incorrectly Finding the Second-Largest Element

**Incorrect:**

```python
numbers = [10, 10, 9]

numbers.sort(reverse=True)

print(numbers[1])
# 10
```

This returns the second element, not the second-largest distinct value.

**Correct:**

```python
def second_largest(numbers):
    unique_numbers = set(numbers)

    if len(unique_numbers) < 2:
        return None

    unique_numbers.remove(max(unique_numbers))
    return max(unique_numbers)
```

For O(n) time and O(1) extra space, use a single-pass approach with two variables.

### 5.3 Assuming the Array Is Sorted

Binary search, two-pointer pair-sum techniques, and some duplicate-removal techniques depend on sorted input.

Do not apply them to unsorted input unless you first establish the required ordering.

### 5.4 Confusing Index and Value

```python
numbers = [10, 20, 30]

for index, value in enumerate(numbers):
    print(index, value)
```

- `index` is the position.
- `value` is the element.

Use `enumerate()` when both are needed.

### 5.5 Incorrectly Reversing a String

Repeated string concatenation can lead to O(n²) time because strings are immutable.

**Better:**

```python
text = "python"

result = text[::-1]
```

Or use a two-pointer approach when in-place modification of a character list is appropriate.

### 5.6 Mishandling Negative Numbers

Do not initialize maximum or minimum values to zero unless the problem guarantees non-negative input.

**Incorrect:**

```python
maximum = 0
```

For an all-negative array, this can return the wrong answer.

**Correct:**

```python
maximum = numbers[0]
```

First handle the empty-input case.

### 5.7 Incorrectly Removing Duplicates

A set removes duplicates but does not provide a reliable way to preserve the original sequence order.

If order matters:

```python
numbers = [3, 1, 3, 2, 1]

unique = list(dict.fromkeys(numbers))

print(unique)
# [3, 1, 2]
```

---

## 6. Two-Pointer and Sliding-Window Mistakes

### 6.1 Moving the Wrong Pointer

In a sorted two-sum problem:

- If the sum is too small, move the left pointer forward.
- If the sum is too large, move the right pointer backward.

```python
def two_sum_sorted(numbers, target):
    left = 0
    right = len(numbers) - 1

    while left < right:
        total = numbers[left] + numbers[right]

        if total == target:
            return [left, right]

        if total < target:
            left += 1
        else:
            right -= 1

    return None
```

### 6.2 Using Sliding Window When Negative Values Break the Assumption

A common sliding-window technique for finding a minimum-length subarray with sum at least a target relies on non-negative numbers.

With negative values, expanding and shrinking the window does not preserve the necessary monotonic behavior.

For arbitrary positive and negative values, prefix sums with a monotonic deque may be appropriate for certain shortest-subarray problems.

**Remember:** Confirm the input assumptions before choosing a window strategy.

### 6.3 Forgetting to Remove the Outgoing Element

For a fixed-size window, each new element enters and the element leaving the window must be removed.

```python
window_sum += numbers[right]
window_sum -= numbers[right - k]
```

Forgetting the second operation causes the window to grow instead of staying at size `k`.

### 6.4 Incorrectly Handling Window Size

Before processing a window, validate:

```python
if k <= 0 or k > len(numbers):
    raise ValueError("Invalid window size")
```

The correct handling of `k == 0` depends on the problem definition.

---

## 7. Hashing Mistakes

### 7.1 Overwriting Useful Information in a Dictionary

In Two Sum, store the index of a previously seen value. Check for the complement before inserting the current value.

```python
def two_sum(numbers, target):
    seen = {}

    for index, number in enumerate(numbers):
        complement = target - number

        if complement in seen:
            return [seen[complement], index]

        seen[number] = index

    return None
```

Checking before insertion avoids using the same element twice when the target is twice that value.

### 7.2 Forgetting That Dictionary Keys Must Be Hashable

**Incorrect:**

```python
data = {}
data[[1, 2]] = "value"
```

Lists cannot be dictionary keys because they are mutable and unhashable.

**Correct:**

```python
data = {}
data[(1, 2)] = "value"
```

A tuple is hashable only if all its elements are hashable.

### 7.3 Ignoring Duplicate Values in Frequency Problems

If a problem asks for distinct values, count or deduplicate accordingly.

If it asks for every occurrence, do not discard duplicates.

Read the output requirement carefully.

---

## 8. Linked List Mistakes

### 8.1 Losing the Remaining Linked List While Reversing

**Incorrect:**

```python
current.next = previous
current = current.next
```

After changing `current.next`, the reference to the original next node is lost.

**Correct:**

```python
def reverse_linked_list(head):
    previous = None
    current = head

    while current:
        next_node = current.next
        current.next = previous

        previous = current
        current = next_node

    return previous
```

Always save the next node before changing a link.

### 8.2 Forgetting to Update the Head

After reversing a linked list, the returned node is the new head.

```python
head = reverse_linked_list(head)
```

If the return value is ignored, the caller may continue using the old head.

### 8.3 Incorrect Fast and Slow Pointer Conditions

For middle-node and cycle problems, check that the fast pointer and its next node exist before advancing two steps.

```python
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
```

### 8.4 Not Handling an Empty Linked List

A function should define what happens when `head is None`.

For example, reversing an empty linked list should normally return `None`.

### 8.5 Confusing Node Identity and Node Value

Cycle detection should compare node references:

```python
if slow is fast:
    return True
```

Two different nodes may contain equal values. Comparing only their data can produce incorrect cycle detection.

---

## 9. Stack and Queue Mistakes

### 9.1 Using `pop(0)` for a Queue

**Inefficient:**

```python
queue = [10, 20, 30]

queue.pop(0)
```

Removing the first element from a Python list requires shifting remaining elements and takes O(n).

**Better:**

```python
from collections import deque

queue = deque([10, 20, 30])

queue.popleft()
```

`deque.popleft()` takes O(1) time.

### 9.2 Incorrect Parentheses Validation

Do not check only the number of opening and closing brackets.

The order and type of brackets also matter.

A stack should verify that every closing bracket matches the most recent unmatched opening bracket.

### 9.3 Forgetting Empty-Stack Conditions

Before calling `pop()` or accessing `stack[-1]`, verify that the stack is not empty.

```python
if not stack:
    return False
```

The expected result depends on the problem.

---

## 10. Binary Search Mistakes

### 10.1 Using Binary Search on Unsorted Input

Binary search requires sorted input or another valid monotonic structure.

Without that property, eliminating half the search space is not justified.

### 10.2 Mixing Inclusive and Exclusive Boundaries

Two common conventions are:

**Inclusive interval:**

```python
left = 0
right = len(numbers) - 1

while left <= right:
    ...
```

**Half-open interval:**

```python
left = 0
right = len(numbers)

while left < right:
    ...
```

Both are valid when implemented consistently. Do not mix their update rules.

### 10.3 Incorrect Midpoint Calculation

Use:

```python
mid = left + (right - left) // 2
```

This avoids potential integer overflow in fixed-width integer languages. Python integers do not have this overflow issue, but the expression is still a clear convention.

### 10.4 Forgetting Duplicate Requirements

A standard binary search may return any matching occurrence.

If the problem asks for the first or last occurrence, use a modified binary search that continues searching after finding a match.

### 10.5 Infinite Loops

Every iteration must reduce the search interval.

Check:
- Whether `left` increases.
- Whether `right` decreases.
- Whether the loop condition eventually becomes false.

---

## 11. Tree and Graph Mistakes

### 11.1 Forgetting the Base Case in Tree Recursion

Always handle an empty node:

```python
def preorder(root):
    if root is None:
        return

    print(root.val)
    preorder(root.left)
    preorder(root.right)
```

### 11.2 Marking Nodes Visited Too Late in BFS

Mark a node as visited when adding it to the queue, not only when removing it.

```python
visited = {start}
queue = deque([start])
```

This prevents the same node from being added multiple times through different neighbors.

### 11.3 Forgetting the Visited Set in Graph Traversal

Graphs can contain cycles. Without a visited set, traversal may revisit nodes indefinitely.

### 11.4 Assuming Every Graph Is Connected

Starting BFS or DFS from one vertex only visits the component reachable from that vertex.

To traverse every component, loop over all vertices and start a traversal from each unvisited vertex.

### 11.5 Confusing Directed and Undirected Cycle Detection

Cycle detection rules differ:

- In an undirected graph, DFS commonly tracks the parent node.
- In a directed graph, DFS can use recursion states such as unvisited, visiting, and visited.

Do not apply one method blindly to both graph types.

### 11.6 Assuming DFS Finds the Shortest Path

DFS explores deeply before exploring alternatives. It does not generally find the shortest path.

For an unweighted graph, BFS finds a shortest path in terms of number of edges.

---

## 12. Recursion and Dynamic Programming Mistakes

### 12.1 Missing a Base Case

A recursive function without a terminating condition can cause infinite recursion or a `RecursionError`.

### 12.2 Not Making Progress Toward the Base Case

A recursive call must reduce the problem or move it toward termination.

### 12.3 Recomputing the Same Subproblems

A naive recursive Fibonacci function repeatedly calculates the same values.

**Inefficient:**

```python
def fibonacci(n):
    if n <= 1:
        return n

    return fibonacci(n - 1) + fibonacci(n - 2)
```

Its time complexity is exponential.

**Improved with memoization:**

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci(n):
    if n <= 1:
        return n

    return fibonacci(n - 1) + fibonacci(n - 2)
```

### 12.4 Incorrect DP State Definition

Before writing a DP solution, clearly define what each state represents.

For example:

`dp[i]` = maximum sum obtainable from the first `i` elements under a specified selection rule.

An unclear state definition often leads to incorrect transitions.

### 12.5 Incorrect DP Initialization

The initial values must represent valid answers to the smallest subproblems.

For Fibonacci:

```python
dp[0] = 0
dp[1] = 1
```

Do not initialize all states to zero unless zero is a valid starting value for the problem.

### 12.6 Updating DP in the Wrong Direction

For 0/1 Knapsack using a one-dimensional DP array, iterate capacity backward so that an item is not reused in the same iteration.

```python
for weight, value in items:
    for capacity in range(max_capacity, weight - 1, -1):
        dp[capacity] = max(
            dp[capacity],
            dp[capacity - weight] + value
        )
```

For unbounded knapsack, forward iteration may be appropriate because an item can be reused.

---

## 13. Complexity Analysis Mistakes

### 13.1 Ignoring Nested Loops

```python
for i in range(n):
    for j in range(n):
        print(i, j)
```

Time complexity: O(n²).

### 13.2 Assuming Every Nested Loop Is O(n²)

```python
i = 1

while i < n:
    i *= 2
```

This takes O(log n), not O(n).

When analyzing loops, consider how the loop variable changes.

### 13.3 Ignoring Sorting Complexity

If an algorithm sorts an array before processing it, include the sorting cost.

For example:

- Sorting: O(n log n).
- One linear scan: O(n).
- Total: O(n log n).

### 13.4 Ignoring Auxiliary Space

An algorithm may use O(n) additional memory even if its input is not modified.

Consider:
- Temporary lists.
- Hash maps.
- Recursion stack.
- Queue or stack storage.
- DP tables.

### 13.5 Confusing Output Space and Auxiliary Space

If a problem requires returning an output list of size n, that output itself takes O(n) space.

When discussing auxiliary space, state whether output storage is excluded.

---

## 14. Testing and Debugging Mistakes

### 14.1 Testing Only One Example

One successful example does not prove correctness.

Use a small test set that covers different cases.

### 14.2 Important Edge Cases

| Case | Example |
|---|---|
| Empty input | `[]` |
| Single element | `[5]` |
| All duplicates | `[2, 2, 2]` |
| Negative values | `[-5, -2, -9]` |
| Already sorted | `[1, 2, 3, 4]` |
| Reverse sorted | `[4, 3, 2, 1]` |
| Target absent | Target not in array |
| Multiple valid answers | More than one pair matches |
| Large input | Test performance |
| Boundary values | Minimum and maximum constraints |

### 14.3 Not Testing the Exact Boundary

For binary search, test:
- First element.
- Last element.
- Element not found.
- One-element input.
- Empty input.

For sliding window, test:
- `k = 1`.
- `k = len(numbers)`.
- Invalid window size.

### 14.4 Ignoring Error Messages

Read the full traceback.

Common Python errors:

| Error | Common cause |
|---|---|
| `IndexError` | Invalid list or string index |
| `KeyError` | Missing dictionary key |
| `TypeError` | Operation on incompatible types |
| `ValueError` | Invalid value passed to a function |
| `AttributeError` | Object does not have requested attribute |
| `NameError` | Variable or name is undefined |
| `RecursionError` | Recursion depth exceeded |

### 14.5 Changing Too Many Things at Once

When debugging:
1. Reproduce the failure.
2. Identify the smallest failing input.
3. Locate the incorrect assumption.
4. Change one part.
5. Run the test again.

This makes the root cause easier to identify.

---

## 15. Common Interview Coding Mistakes: Quick Reference

| Mistake | Better practice |
|---|---|
| Coding before understanding requirements | Clarify input, output, and constraints |
| Ignoring input size | Estimate complexity first |
| Mutable default argument | Use `None` and initialize inside |
| Using `is` for value comparison | Use `==` |
| Modifying list during iteration | Build a new list or iterate safely |
| Using `pop(0)` for queues | Use `collections.deque` |
| Binary search on unsorted input | Verify sorted or monotonic property |
| Incorrect pointer updates | Ensure every iteration progresses |
| Forgetting linked-list next reference | Save it before changing links |
| Missing visited set | Track visited graph nodes |
| Missing recursion base case | Define termination clearly |
| Recomputing DP states | Use memoization or tabulation |
| Ignoring duplicates | Clarify distinct vs repeated values |
| Initializing maximum to zero | Initialize from valid input |
| Ignoring empty input | Handle boundary conditions |
| Ignoring sorting cost | Include it in complexity |
| Testing only normal inputs | Test edge cases |
| Not explaining complexity | State time and auxiliary space |

---

## 16. A Practical Checklist Before Submitting Code

Use this checklist for every coding problem.

### Understanding
- [ ] I understand the expected output.
- [ ] I checked the input constraints.
- [ ] I identified assumptions about duplicates, ordering, and negative values.

### Algorithm
- [ ] I selected an appropriate pattern.
- [ ] I considered a brute-force solution.
- [ ] I checked whether the optimized approach is correct.
- [ ] I know the expected time complexity.

### Implementation
- [ ] I handled empty input.
- [ ] I handled single-element input.
- [ ] I checked loop boundaries.
- [ ] I updated pointers correctly.
- [ ] I handled duplicates correctly.
- [ ] I avoided unnecessary memory usage.

### Testing
- [ ] I tested a normal example.
- [ ] I tested boundary cases.
- [ ] I tested a case with no valid answer.
- [ ] I checked the output manually.

### Explanation
- [ ] I can explain why the algorithm works.
- [ ] I can explain time complexity.
- [ ] I can explain auxiliary space complexity.
- [ ] I can discuss possible improvements.

---

## 17. Final Interview Rule

Before submitting a solution, ask yourself five questions:

1. **Correctness:** Does it solve the exact problem?
2. **Boundaries:** Does it handle empty, small, and extreme inputs?
3. **Progress:** Will every loop and recursive call terminate?
4. **Efficiency:** Is the time and memory usage acceptable?
5. **Explanation:** Can I clearly explain the approach and complexity?

A correct solution is more valuable than a clever solution that fails on edge cases. Build the habit of understanding, coding, testing, and explaining every problem.