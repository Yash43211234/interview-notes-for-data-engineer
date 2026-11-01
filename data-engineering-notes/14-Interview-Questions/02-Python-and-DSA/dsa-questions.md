# Data Structures and Algorithms (DSA) Interview Questions

## 1. DSA Fundamentals

### 1.1 What is a Data Structure?

A data structure is a way of organizing and storing data so that it can be accessed and modified efficiently.

Common data structures:

- Array
- Linked List
- Stack
- Queue
- Hash Table
- Tree
- Heap
- Graph

### 1.2 What is an Algorithm?

An algorithm is a finite sequence of steps used to solve a problem.

A good algorithm should be:
- Correct.
- Efficient in time and memory.
- Understandable.
- Suitable for the input constraints.

### 1.3 What is Time Complexity?

Time complexity describes how an algorithm's running time grows as the input size increases.

| Complexity | Name | Example |
|---|---|---|
| O(1) | Constant | Array index access |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Linear search |
| O(n log n) | Linearithmic | Merge sort |
| O(n²) | Quadratic | Simple nested loops |
| O(2ⁿ) | Exponential | Some recursive subset algorithms |
| O(n!) | Factorial | Brute-force permutations |

### 1.4 What is Space Complexity?

Space complexity describes how memory usage grows with input size.

Consider:

```python
def find_sum(numbers):
    total = 0

    for number in numbers:
        total += number

    return total
```

The algorithm uses O(1) auxiliary space because it uses a fixed number of extra variables.

If it creates another list containing all input elements, that additional storage may be O(n).

### 1.5 What is Big O, Big Omega, and Big Theta?

- **Big O:** Asymptotic upper bound.
- **Big Omega:** Asymptotic lower bound.
- **Big Theta:** Tight asymptotic bound.

In interviews, Big O is commonly used to communicate an algorithm's growth rate.

### 1.6 What is the difference between best, average, and worst case?

- **Best case:** Minimum work for an input of a given size.
- **Average case:** Expected work under an assumed input distribution.
- **Worst case:** Maximum work for an input of a given size.

For example, linear search:
- Best case: O(1), when the first element matches.
- Worst case: O(n), when the element is last or absent.

### 1.7 What is an In-Place Algorithm?

An in-place algorithm uses a small, constant amount of extra working memory, usually O(1) auxiliary space.

Example: Reversing a list by swapping elements from both ends.

### 1.8 What is Recursion?

Recursion is a technique in which a function calls itself to solve smaller versions of a problem.

Every recursive solution should have:
- Base case.
- Recursive case.
- Progress toward the base case.

```python
def factorial(n):
    if n < 0:
        raise ValueError("n must be non-negative")

    if n == 0:
        return 1

    return n * factorial(n - 1)
```

The recursive call must eventually reach the base case.

---

## 2. Arrays and Strings

### 2.1 What is an Array?

An array stores elements in an indexed sequence.

Python lists are dynamic arrays in CPython. They store references to objects and can grow or shrink.

Common operations:

| Operation | Typical Complexity |
|---|---:|
| Access by index | O(1) |
| Search unsorted list | O(n) |
| Append | O(1) amortized |
| Insert at beginning | O(n) |
| Delete from beginning | O(n) |
| Delete by value | O(n) |

### 2.2 What is the Two-Pointer Technique?

Two pointers are used to process a sequence using two indices, often moving toward each other or in the same direction.

Example: Check whether a string is a palindrome.

```python
def is_palindrome(text):
    left = 0
    right = len(text) - 1

    while left < right:
        if text[left] != text[right]:
            return False

        left += 1
        right -= 1

    return True
```

**Complexity:** O(n) time and O(1) extra space.

Common use cases:
- Pair sum in a sorted array.
- Reverse a sequence.
- Remove duplicates from sorted arrays.
- Merge sorted arrays.
- Palindrome checking.

### 2.3 What is the Sliding Window Technique?

Sliding window maintains a range of elements and moves that range through an array or string.

Example: Maximum sum of a subarray of fixed size `k`.

```python
def max_window_sum(numbers, k):
    if k <= 0 or k > len(numbers):
        raise ValueError("Invalid window size")

    window_sum = sum(numbers[:k])
    maximum = window_sum

    for right in range(k, len(numbers)):
        window_sum += numbers[right]
        window_sum -= numbers[right - k]

        maximum = max(maximum, window_sum)

    return maximum


print(max_window_sum([2, 1, 5, 1, 3, 2], 3))
# 9
```

**Complexity:** O(n) time and O(1) extra space.

Common use cases:
- Maximum sum of a fixed-size subarray.
- Longest substring without repeating characters.
- Minimum-size subarray satisfying a condition.

### 2.4 What is Prefix Sum?

Prefix sum stores cumulative sums to answer range-sum queries efficiently.

```python
numbers = [2, 4, 6, 8]

prefix = [0]

for number in numbers:
    prefix.append(prefix[-1] + number)

# Sum of elements from index 1 to 3, inclusive
result = prefix[4] - prefix[1]

print(result)
# 18
```

After O(n) preprocessing, each range-sum query takes O(1).

### 2.5 What is Kadane's Algorithm?

Kadane's algorithm finds the maximum sum of a non-empty contiguous subarray.

```python
def max_subarray_sum(numbers):
    if not numbers:
        raise ValueError("Input cannot be empty")

    current = best = numbers[0]

    for number in numbers[1:]:
        current = max(number, current + number)
        best = max(best, current)

    return best


print(max_subarray_sum([-2, 1, -3, 4, -1, 2, 1, -5, 4]))
# 6
```

**Complexity:** O(n) time and O(1) extra space.

### 2.6 What is the difference between a substring and a subsequence?

- **Substring:** A contiguous part of a string.
- **Subsequence:** Elements selected in their original order, but not necessarily contiguous.

For `"ABCDE"`:
- `"BCD"` is both a substring and subsequence.
- `"ACE"` is a subsequence, but not a substring.

### 2.7 How do you find the second-largest distinct element?

```python
def second_largest(numbers):
    largest = None
    second = None

    for number in numbers:
        if largest is None or number > largest:
            second = largest
            largest = number

        elif number != largest and (
            second is None or number > second
        ):
            second = number

    return second


print(second_largest([10, 20, 30, 30, 40]))
# 30
```

**Complexity:** O(n) time and O(1) extra space.

Returns `None` if fewer than two distinct values exist.

---

## 3. Hashing

### 3.1 What is Hashing?

Hashing converts a key into a hash value used to locate data in a hash-based data structure.

Python's `dict` and `set` use hash tables.

Average-case lookup, insertion, and deletion are O(1), although worst-case behavior can be O(n).

### 3.2 What is a Hash Collision?

A collision occurs when two different keys produce the same hash value.

Hash tables handle collisions using implementation-specific techniques. CPython dictionaries use open addressing.

### 3.3 What makes an object usable as a dictionary key?

A dictionary key must be hashable.

A hashable object has a hash value that remains stable during its lifetime and supports equality comparison consistently.

Examples:
- Integers.
- Strings.
- Tuples containing only hashable elements.

Lists and dictionaries are not hashable.

### 3.4 What is the Two Sum Problem?

Find two indices whose values add up to a target.

```python
def two_sum(numbers, target):
    seen = {}

    for index, number in enumerate(numbers):
        required = target - number

        if required in seen:
            return [seen[required], index]

        seen[number] = index

    return None


print(two_sum([2, 7, 11, 15], 9))
# [0, 1]
```

**Complexity:** O(n) average time and O(n) space.

### 3.5 How do you find duplicate elements?

```python
def find_duplicates(numbers):
    seen = set()
    duplicates = set()

    for number in numbers:
        if number in seen:
            duplicates.add(number)
        else:
            seen.add(number)

    return duplicates
```

**Complexity:** O(n) average time and O(n) space.

### 3.6 What is Frequency Counting?

Frequency counting records how often each element occurs.

```python
from collections import Counter

numbers = [1, 2, 2, 3, 3, 3]

frequency = Counter(numbers)

print(frequency)
# Counter({3: 3, 2: 2, 1: 1})
```

Useful for:
- Finding duplicates.
- Anagram checking.
- Most frequent elements.
- Counting occurrences.

---

## 4. Linked Lists

### 4.1 What is a Linked List?

A linked list is a linear data structure made of nodes. Each node stores data and one or more references to other nodes.

Unlike arrays, linked-list nodes do not need to be stored contiguously in memory.

### 4.2 Array vs Linked List

| Feature | Dynamic Array | Linked List |
|---|---|---|
| Index access | O(1) | O(n) |
| Search | O(n) | O(n) |
| Insert at beginning | O(n) | O(1) |
| Delete at beginning | O(n) | O(1) |
| Memory layout | Contiguous reference array | Nodes connected by references |
| Extra pointer memory | No per-element link | Yes |

Linked-list insertion and deletion are O(1) when the required node or its predecessor is already known.

### 4.3 What are the types of Linked Lists?

1. **Singly Linked List:** Each node points to the next node.
2. **Doubly Linked List:** Each node points to both previous and next nodes.
3. **Circular Linked List:** The last node links back to the first node.

### 4.4 How do you create a Singly Linked List?

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None


class LinkedList:
    def __init__(self):
        self.head = None

    def append(self, data):
        new_node = Node(data)

        if self.head is None:
            self.head = new_node
            return

        current = self.head

        while current.next:
            current = current.next

        current.next = new_node

    def display(self):
        current = self.head

        while current:
            print(current.data, end=" -> ")
            current = current.next

        print("None")
```

### 4.5 How do you reverse a Linked List?

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

**Complexity:** O(n) time and O(1) extra space.

### 4.6 How do you find the middle of a Linked List?

Use slow and fast pointers.

```python
def find_middle(head):
    slow = fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    return slow
```

For an even number of nodes, this returns the second middle node.

**Complexity:** O(n) time and O(1) space.

### 4.7 How do you detect a cycle in a Linked List?

Use Floyd's cycle detection algorithm.

```python
def has_cycle(head):
    slow = fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

        if slow is fast:
            return True

    return False
```

**Complexity:** O(n) time and O(1) space.

### 4.8 What is the Nth Node from the End?

Use two pointers with a gap of `n` nodes.

```python
def nth_from_end(head, n):
    if n <= 0:
        return None

    fast = slow = head

    for _ in range(n):
        if fast is None:
            return None
        fast = fast.next

    while fast:
        slow = slow.next
        fast = fast.next

    return slow
```

This returns `None` if `n` exceeds the list length.

**Complexity:** O(n) time and O(1) space.

---

## 5. Stack and Queue

### 5.1 What is a Stack?

A stack is a linear data structure that follows LIFO: Last In, First Out.

Main operations:
- Push: Add an element.
- Pop: Remove the top element.
- Peek: View the top element.

Typical complexity: O(1) for these operations.

Python example:

```python
stack = []

stack.append(10)
stack.append(20)

print(stack.pop())  # 20
print(stack[-1])    # 10
```

### 5.2 What is a Queue?

A queue follows FIFO: First In, First Out.

Main operations:
- Enqueue: Add an element.
- Dequeue: Remove the front element.
- Front: View the first element.

Use `deque` for efficient queue operations in Python.

```python
from collections import deque

queue = deque()

queue.append(10)
queue.append(20)

print(queue.popleft())  # 10
```

### 5.3 Stack vs Queue

| Feature | Stack | Queue |
|---|---|---|
| Principle | LIFO | FIFO |
| Insert | Top | Rear |
| Remove | Top | Front |
| Common use | Undo, recursion, parsing | Scheduling, BFS, buffering |

### 5.4 What is a Priority Queue?

A priority queue removes elements according to priority rather than insertion order.

Python provides `heapq`, which implements a min-heap.

```python
import heapq

numbers = [5, 2, 8, 1]

heapq.heapify(numbers)

print(heapq.heappop(numbers))
# 1
```

### 5.5 How do you implement a Queue using Two Stacks?

```python
class QueueUsingStacks:
    def __init__(self):
        self.in_stack = []
        self.out_stack = []

    def enqueue(self, value):
        self.in_stack.append(value)

    def dequeue(self):
        if not self.out_stack:
            while self.in_stack:
                self.out_stack.append(self.in_stack.pop())

        if not self.out_stack:
            raise IndexError("Queue is empty")

        return self.out_stack.pop()
```

Each element is moved between stacks at most once, so enqueue and dequeue are amortized O(1).

---

## 6. Searching Algorithms

### 6.1 What is Linear Search?

Linear search checks each element until the target is found or the collection ends.

```python
def linear_search(numbers, target):
    for index, number in enumerate(numbers):
        if number == target:
            return index

    return -1
```

**Complexity:** O(n) time and O(1) space.

### 6.2 What is Binary Search?

Binary search repeatedly divides a sorted search range into two halves.

```python
def binary_search(numbers, target):
    left = 0
    right = len(numbers) - 1

    while left <= right:
        mid = left + (right - left) // 2

        if numbers[mid] == target:
            return mid

        if numbers[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return -1
```

**Complexity:** O(log n) time and O(1) space.

The input must be sorted according to the search order.

### 6.3 Linear Search vs Binary Search

| Feature | Linear Search | Binary Search |
|---|---|---|
| Input | Can be unsorted | Must be sorted |
| Time complexity | O(n) | O(log n) |
| Implementation | Simple | Requires boundary management |
| Suitable for | Small or unsorted data | Large sorted data |

### 6.4 What is Binary Search on Answer?

Instead of searching for an element, binary search can find the smallest or largest value satisfying a monotonic condition.

Examples:
- Minimum capacity required to ship packages.
- Minimum time to complete jobs.
- Maximum minimum distance between placements.

The key requirement is that the feasibility condition changes in one direction as the candidate answer changes.

---

## 7. Sorting Algorithms

### 7.1 What is Bubble Sort?

Bubble sort repeatedly compares adjacent elements and swaps them when they are in the wrong order.

```python
def bubble_sort(numbers):
    arr = numbers.copy()
    n = len(arr)

    for i in range(n):
        swapped = False

        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True

        if not swapped:
            break

    return arr
```

- Best case: O(n) with early stopping.
- Average case: O(n²).
- Worst case: O(n²).
- Extra space: O(1), excluding the copied output list.

### 7.2 What is Selection Sort?

Selection sort repeatedly selects the smallest remaining element and places it in the correct position.

- Best, average, and worst time: O(n²).
- Extra space: O(1) for an in-place implementation.

### 7.3 What is Insertion Sort?

Insertion sort builds a sorted portion by inserting each new element into its correct position.

- Best case: O(n).
- Average case: O(n²).
- Worst case: O(n²).
- Extra space: O(1).

It works well for small or nearly sorted arrays.

### 7.4 What is Merge Sort?

Merge sort divides the input into smaller parts, sorts them recursively, and merges the sorted parts.

- Time: O(n log n).
- Auxiliary space: O(n) for a typical array implementation.
- Stable: Yes, when the merge step preserves equal-element order.

### 7.5 What is Quick Sort?

Quick sort chooses a pivot, partitions elements around it, and recursively sorts the partitions.

- Average time: O(n log n).
- Worst-case time: O(n²).
- Typical in-place partitioning uses O(log n) average recursion stack space, but worst-case stack space can be O(n).

### 7.6 Sorting Complexity Comparison

| Algorithm | Best | Average | Worst |
|---|---:|---:|---:|
| Bubble Sort | O(n) with optimization | O(n²) | O(n²) |
| Selection Sort | O(n²) | O(n²) | O(n²) |
| Insertion Sort | O(n) | O(n²) | O(n²) |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) |
| Quick Sort | O(n log n) | O(n log n) | O(n²) |

Python's built-in `list.sort()` and `sorted()` use Timsort, a stable adaptive sorting algorithm.

---

## 8. Trees and Binary Search Trees

### 8.1 What is a Tree?

A tree is a hierarchical, non-linear data structure made of nodes connected by edges.

Important terms:
- Root: Top node.
- Parent: Node with children.
- Child: Node connected below a parent.
- Leaf: Node with no children.
- Height: Number of edges on the longest downward path from a node to a leaf.
- Depth: Number of edges from the root to a node.

Some textbooks define height using nodes instead of edges; state the convention when solving a problem.

### 8.2 What is a Binary Tree?

A binary tree is a tree in which each node has at most two children: left and right.

### 8.3 What is a Binary Search Tree (BST)?

A BST is a binary tree with an ordering property:

- Values in the left subtree follow the lower-value rule.
- Values in the right subtree follow the higher-value rule.

Duplicate-value handling depends on the chosen BST convention.

### 8.4 What are Tree Traversals?

**Inorder:** Left → Root → Right.

**Preorder:** Root → Left → Right.

**Postorder:** Left → Right → Root.

**Level Order:** Visit nodes level by level using a queue.

### 8.5 How do you perform Inorder Traversal?

```python
def inorder(root):
    if root is None:
        return

    inorder(root.left)
    print(root.val)
    inorder(root.right)
```

For a BST with distinct keys, inorder traversal returns values in sorted order.

### 8.6 How do you perform Level Order Traversal?

```python
from collections import deque

def level_order(root):
    if root is None:
        return []

    queue = deque([root])
    result = []

    while queue:
        node = queue.popleft()
        result.append(node.val)

        if node.left:
            queue.append(node.left)

        if node.right:
            queue.append(node.right)

    return result
```

**Complexity:** O(n) time and O(w) auxiliary space, where w is the maximum tree width.

### 8.7 What is the difference between a Binary Tree and BST?

| Binary Tree | Binary Search Tree |
|---|---|
| At most two children per node | At most two children per node |
| No required ordering | Follows an ordering property |
| Search can be O(n) | Search can be O(log n) when balanced |
| Used for hierarchical structures | Used for ordered searching |

An unbalanced BST can have O(n) search, insertion, and deletion complexity.

### 8.8 What is a Balanced Tree?

A balanced tree maintains height within a controlled bound to avoid becoming a long chain.

Examples:
- AVL Tree.
- Red-Black Tree.

Balanced BST operations are generally O(log n).

### 8.9 What is a Heap?

A heap is a complete binary tree that follows a heap-order property.

- Min-heap: Parent value is less than or equal to child values.
- Max-heap: Parent value is greater than or equal to child values.

Common complexities:

| Operation | Complexity |
|---|---:|
| Peek minimum in min-heap | O(1) |
| Insert | O(log n) |
| Remove root | O(log n) |
| Build heap | O(n) |

Python's `heapq` provides a min-heap.

---

## 9. Graphs

### 9.1 What is a Graph?

A graph consists of vertices (nodes) and edges connecting them.

Types:
- Directed graph.
- Undirected graph.
- Weighted graph.
- Unweighted graph.
- Cyclic graph.
- Acyclic graph.

### 9.2 What is an Adjacency List?

An adjacency list stores each vertex with its neighboring vertices.

```python
graph = {
    "A": ["B", "C"],
    "B": ["A", "D"],
    "C": ["A"],
    "D": ["B"]
}
```

For sparse graphs, adjacency lists are usually memory-efficient.

### 9.3 What is an Adjacency Matrix?

An adjacency matrix uses a two-dimensional matrix to represent connections between vertices.

For V vertices:
- Space complexity: O(V²).
- Checking whether an edge exists: O(1).

### 9.4 BFS vs DFS

| Feature | BFS | DFS |
|---|---|---|
| Full form | Breadth-First Search | Depth-First Search |
| Data structure | Queue | Stack or recursion |
| Traversal | Level by level | Goes deep first |
| Shortest path | Unweighted graphs | Not guaranteed |
| Common use | Minimum-edge path | Cycle detection, components, traversal |

### 9.5 How do you implement BFS?

```python
from collections import deque

def bfs(graph, start):
    visited = {start}
    queue = deque([start])
    result = []

    while queue:
        node = queue.popleft()
        result.append(node)

        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

    return result
```

**Complexity:** O(V + E) using an adjacency list.

### 9.6 How do you implement DFS?

```python
def dfs(graph, start):
    visited = set()
    result = []

    def explore(node):
        visited.add(node)
        result.append(node)

        for neighbor in graph[node]:
            if neighbor not in visited:
                explore(neighbor)

    explore(start)
    return result
```

**Complexity:** O(V + E) using an adjacency list.

For very deep graphs, iterative DFS can avoid Python recursion-depth limitations.

### 9.7 What is a Cycle in a Graph?

A cycle is a path that starts and ends at the same vertex without repeating intermediate vertices.

Cycle detection depends on whether the graph is directed or undirected.

### 9.8 What is Topological Sorting?

Topological sorting is a linear ordering of vertices in a directed acyclic graph (DAG), such that for every directed edge u → v, u appears before v.

Common algorithms:
- Kahn's algorithm using indegrees and a queue.
- DFS-based topological sorting.

It is useful for:
- Task scheduling.
- Dependency resolution.
- Build systems.
- Pipeline execution order.

### 9.9 What is Dijkstra's Algorithm?

Dijkstra's algorithm finds shortest paths from one source to other vertices in a graph with non-negative edge weights.

It commonly uses a priority queue.

It does not correctly handle arbitrary negative-weight edges.

---

## 10. Recursion and Backtracking

### 10.1 What is Backtracking?

Backtracking explores possible choices and reverses a choice when it cannot lead to a valid solution.

General pattern:

1. Choose an option.
2. Explore recursively.
3. Undo the choice.
4. Try another option.

### 10.2 What is the Difference Between Recursion and Backtracking?

- Recursion is a function calling itself.
- Backtracking is a problem-solving technique that often uses recursion to explore and undo choices.

### 10.3 What is a Subset Problem?

Generate all possible subsets of a collection.

```python
def subsets(numbers):
    result = []

    def backtrack(index, current):
        if index == len(numbers):
            result.append(current.copy())
            return

        # Exclude current element
        backtrack(index + 1, current)

        # Include current element
        current.append(numbers[index])
        backtrack(index + 1, current)

        current.pop()

    backtrack(0, [])
    return result
```

For n elements, there are 2ⁿ possible subsets.

**Complexity:** O(n × 2ⁿ) time when accounting for copying each subset.

### 10.4 What is Permutation?

A permutation is an arrangement of elements in a particular order.

For n distinct elements, the number of permutations is n!.

### 10.5 What is the N-Queens Problem?

The N-Queens problem places N queens on an N × N chessboard so that no two queens attack each other.

It is commonly solved using backtracking with column and diagonal checks.

---

## 11. Dynamic Programming

### 11.1 What is Dynamic Programming?

Dynamic Programming (DP) solves problems by breaking them into overlapping subproblems and storing results to avoid repeated calculations.

Two important properties:
- Overlapping subproblems.
- Optimal substructure.

### 11.2 What is Memoization?

Memoization is a top-down DP approach that stores results of previously solved subproblems.

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci(n):
    if n <= 1:
        return n

    return fibonacci(n - 1) + fibonacci(n - 2)
```

### 11.3 What is Tabulation?

Tabulation is a bottom-up DP approach that solves smaller subproblems first and stores their results in a table.

```python
def fibonacci(n):
    if n < 0:
        raise ValueError("n must be non-negative")

    if n <= 1:
        return n

    dp = [0] * (n + 1)
    dp[1] = 1

    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]

    return dp[n]
```

### 11.4 Memoization vs Tabulation

| Feature | Memoization | Tabulation |
|---|---|---|
| Approach | Top-down | Bottom-up |
| Execution | Recursive | Usually iterative |
| Stores results | Cache | Table |
| Computes | Needed states | Usually all states in chosen iteration range |
| Stack usage | Recursive stack | Usually no recursive stack |

### 11.5 What is the 0/1 Knapsack Problem?

Given items with weights and values, select items to maximize total value without exceeding capacity. Each item can be selected at most once.

A common DP solution takes O(n × capacity) time.

### 11.6 What is Longest Common Subsequence (LCS)?

LCS finds the longest sequence that appears in two strings in the same relative order, but not necessarily contiguously.

For strings `"ABCBDAB"` and `"BDCABA"`, one LCS is `"BCBA"`.

The standard DP solution takes O(m × n) time and O(m × n) space, with possible space optimization.

### 11.7 What is Longest Increasing Subsequence (LIS)?

LIS finds the longest strictly increasing subsequence.

A standard O(n²) DP solution is possible. An optimized method using binary search takes O(n log n) time, though reconstructing the actual subsequence requires additional tracking.

---

## 12. Greedy Algorithms

### 12.1 What is a Greedy Algorithm?

A greedy algorithm makes the best local choice at each step, aiming to reach a globally optimal solution.

A greedy strategy is correct only when the problem has the required greedy-choice property and optimal substructure.

### 12.2 Greedy vs Dynamic Programming

| Greedy | Dynamic Programming |
|---|---|
| Makes a local choice | Compares multiple subproblem results |
| Does not generally reconsider choices | Stores subproblem solutions |
| Often simpler and faster | Often uses more memory |
| Requires proof of greedy correctness | Uses overlapping subproblems and optimal substructure |

### 12.3 What are common Greedy problems?

- Activity selection.
- Interval scheduling.
- Fractional knapsack.
- Huffman coding.
- Minimum spanning tree algorithms such as Kruskal and Prim.

**Important:** Greedy does not solve every optimization problem. For example, the standard 0/1 Knapsack problem generally requires DP rather than the fractional-knapsack greedy strategy.

---

## 13. Disjoint Set Union (Union-Find)

### 13.1 What is Disjoint Set Union?

Disjoint Set Union (DSU), also called Union-Find, maintains a collection of disjoint sets.

Operations:
- `find(x)`: Identifies the representative of the set containing x.
- `union(a, b)`: Merges the sets containing a and b.

### 13.2 What are Path Compression and Union by Size?

- **Path compression:** Makes nodes point closer to the representative during `find`.
- **Union by size/rank:** Attaches the smaller or shallower tree under the larger or deeper one.

Together, these optimizations provide nearly constant amortized operation time.

### 13.3 Python Implementation

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.size = [1] * n

    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])

        return self.parent[x]

    def union(self, a, b):
        root_a = self.find(a)
        root_b = self.find(b)

        if root_a == root_b:
            return False

        if self.size[root_a] < self.size[root_b]:
            root_a, root_b = root_b, root_a

        self.parent[root_b] = root_a
        self.size[root_a] += self.size[root_b]

        return True
```

Common applications:
- Connected components.
- Cycle detection in undirected graphs.
- Kruskal's minimum spanning tree algorithm.

---

## 14. DSA Patterns to Recognize in Interviews

| Pattern | When to Consider It | Typical Problems |
|---|---|---|
| Two Pointers | Sorted arrays or paired comparisons | Pair sum, palindrome |
| Sliding Window | Contiguous subarray or substring | Maximum window sum |
| Prefix Sum | Repeated range-sum queries | Subarray sum |
| Hashing | Fast lookup or frequency counting | Two Sum, duplicates |
| Fast and Slow Pointers | Linked-list cycle or middle | Cycle detection |
| Binary Search | Sorted input or monotonic answer | Search, minimum capacity |
| Stack | Matching, nesting, previous/next element | Parentheses |
| Monotonic Stack | Nearest greater/smaller element | Stock span |
| Heap | Repeated minimum/maximum selection | Top K elements |
| BFS | Level-based traversal or unweighted shortest path | Graph shortest path |
| DFS | Explore paths or connected components | Cycle detection |
| Backtracking | Explore combinations and constraints | Subsets, permutations |
| Dynamic Programming | Overlapping subproblems | Knapsack, LCS |
| Greedy | Locally optimal choice can be proven | Activity selection |
| Union-Find | Dynamic connectivity | Connected components |

---

## 15. Common DSA Interview Mistakes

1. **Ignoring constraints:** An O(n²) solution may fail for a large input.
2. **Not handling empty input:** Always consider empty arrays, strings, and linked lists.
3. **Forgetting duplicate values:** Clarify whether the problem asks for distinct elements.
4. **Incorrect pointer movement:** Two-pointer and binary-search solutions can get stuck or skip valid values.
5. **Incorrect binary-search boundaries:** Decide whether the search interval is inclusive or half-open.
6. **Missing base cases:** Recursive functions must terminate.
7. **Ignoring negative values:** Algorithms based on zero or positive-only assumptions may fail.
8. **Not considering overflow:** Relevant in fixed-width integer languages; Python integers have arbitrary precision but are not free of memory limits.
9. **Confusing substring and subsequence:** Contiguous and non-contiguous selections are different.
10. **Using recursion without considering depth:** Deep recursion can exceed Python's recursion limit.
11. **Not explaining complexity:** State time and auxiliary space complexity.
12. **Not testing edge cases:** Test minimum input, duplicates, sorted input, reverse-sorted input, and no-solution cases.

---

## 16. Interview Problem-Solving Method

Use this sequence when solving a coding problem:

### Step 1: Understand the Problem

Clarify:
- What is the input?
- What is the expected output?
- Are duplicates allowed?
- Is the input sorted?
- Can values be negative?
- What are the constraints?

### Step 2: Work Through an Example

Use a small example and manually trace the expected result.

### Step 3: Identify a Suitable Pattern

Look for:
- Two pointers.
- Sliding window.
- Hashing.
- Binary search.
- Stack or queue.
- Tree or graph traversal.
- Dynamic programming.

### Step 4: Start with a Correct Approach

A brute-force solution can help establish correctness before optimization.

### Step 5: Optimize

Consider whether repeated work can be reduced using:
- Hash maps or sets.
- Sorting.
- Prefix sums.
- Sliding windows.
- Heaps.
- DP.

### Step 6: Write Clean Code

Use meaningful variable names and handle edge cases.

### Step 7: Explain Complexity

State:
- Time complexity.
- Auxiliary space complexity.
- Any assumptions.

### Step 8: Test the Solution

Test:
- Empty input.
- One element.
- Duplicates.
- Negative numbers.
- Already sorted input.
- Reverse-sorted input.
- Boundary conditions.

---

## 17. High-Priority DSA Coding Questions

### Arrays and Strings
- [ ] Reverse an array.
- [ ] Find the largest and second-largest element.
- [ ] Remove duplicates from a sorted array.
- [ ] Move zeros to the end.
- [ ] Rotate an array.
- [ ] Find a missing number.
- [ ] Find the maximum subarray sum.
- [ ] Two Sum.
- [ ] Merge two sorted arrays.
- [ ] Best time to buy and sell stock.
- [ ] Longest substring without repeating characters.
- [ ] Valid anagram.
- [ ] Group anagrams.
- [ ] Product of array except self.

### Linked Lists
- [ ] Reverse a linked list.
- [ ] Find the middle node.
- [ ] Detect a cycle.
- [ ] Find the Nth node from the end.
- [ ] Merge two sorted linked lists.
- [ ] Remove duplicates.
- [ ] Find the intersection of two linked lists.

### Stack and Queue
- [ ] Valid parentheses.
- [ ] Implement a stack using queues.
- [ ] Implement a queue using stacks.
- [ ] Next greater element.
- [ ] Min stack.
- [ ] Sliding window maximum.

### Searching and Sorting
- [ ] Binary search.
- [ ] Search in rotated sorted array.
- [ ] Find first and last occurrence.
- [ ] Find peak element.
- [ ] Merge sort.
- [ ] Quick sort.
- [ ] Search in a 2D matrix.

### Trees
- [ ] Inorder, preorder, and postorder traversal.
- [ ] Level order traversal.
- [ ] Maximum depth of a binary tree.
- [ ] Validate a BST.
- [ ] Lowest common ancestor.
- [ ] Diameter of a binary tree.
- [ ] Invert a binary tree.
- [ ] Kth smallest element in a BST.

### Graphs
- [ ] BFS and DFS.
- [ ] Number of connected components.
- [ ] Detect a cycle.
- [ ] Topological sorting.
- [ ] Number of islands.
- [ ] Shortest path in an unweighted graph.
- [ ] Dijkstra's algorithm.
- [ ] Course schedule.

### Dynamic Programming and Greedy
- [ ] Fibonacci.
- [ ] Climbing stairs.
- [ ] House robber.
- [ ] Coin change.
- [ ] 0/1 Knapsack.
- [ ] Longest common subsequence.
- [ ] Longest increasing subsequence.
- [ ] Maximum subarray.
- [ ] Activity selection.
- [ ] Minimum number of meeting rooms.

---

## 18. Quick Revision

| Topic | Key Point |
|---|---|
| Array | O(1) index access |
| Linked List | Sequential node access |
| Stack | LIFO |
| Queue | FIFO |
| Hash Table | O(1) average lookup |
| Binary Search | O(log n), sorted input |
| Merge Sort | O(n log n), stable |
| Quick Sort | O(n log n) average |
| BST | Ordered binary tree |
| Heap | Efficient minimum/maximum retrieval |
| BFS | Queue-based traversal |
| DFS | Stack/recursion-based traversal |
| Topological Sort | Ordering of DAG vertices |
| Recursion | Function calls itself |
| Backtracking | Explore and undo choices |
| Memoization | Top-down DP |
| Tabulation | Bottom-up DP |
| Greedy | Locally optimal choices |
| DSU | Maintains disjoint sets |

## 19. Self-Assessment

Before a Data Engineer or product-company coding interview, check whether you can:

- Explain time and space complexity without guessing.
- Identify the right pattern from problem constraints.
- Solve array and string problems without relying entirely on built-in functions.
- Use dictionaries and sets effectively.
- Reverse and detect cycles in linked lists.
- Implement BFS, DFS, and binary search.
- Explain recursion and backtracking.
- Recognize when dynamic programming is appropriate.
- Explain edge cases and test your code.
- Write clean Python solutions and discuss complexity.

**Preparation priority:** Master arrays, strings, hashing, two pointers, sliding window, binary search, linked lists, stacks, queues, trees, and basic graph traversal first. Then strengthen heaps, backtracking, greedy algorithms, and dynamic programming.