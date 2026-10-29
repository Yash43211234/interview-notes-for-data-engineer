# Queue and Deque

## 1. Queue

A **queue** is a linear data structure that follows **FIFO (First In, First Out)**.

The element inserted first is removed first, just like people waiting in a line.

### Main operations

| Operation | Meaning | Time Complexity |
|---|---|---|
| `enqueue(x)` | Add an element to the rear | O(1) |
| `dequeue()` | Remove the front element | O(1) with a proper implementation |
| `peek()` / `front()` | View the front element | O(1) |
| `is_empty()` | Check whether the queue is empty | O(1) |

### Example

```text
Enqueue 10 → [10]
Enqueue 20 → [10, 20]
Enqueue 30 → [10, 20, 30]

Dequeue → removes 10
Queue: [20, 30]
```

The first element inserted, `10`, is the first one removed.

## 2. Implementing a Queue in Python

### Recommended: `collections.deque`

```python
from collections import deque

queue = deque()

queue.append(10)
queue.append(20)
queue.append(30)

print(queue[0])       # 10: front element
print(queue.popleft())  # 10: remove front
print(queue)          # deque([20, 30])
print(len(queue))     # 2
```

- `append(x)` inserts at the rear.
- `popleft()` removes from the front.
- `queue[0]` reads the front element.

Both insertion at the rear and removal from the front take **O(1)** time.

### Why not use a list?

You can implement a queue using a list:

```python
queue = []

queue.append(10)
queue.append(20)
queue.append(30)

queue.pop(0)  # Removes 10
```

However, `pop(0)` takes **O(n)** time because the remaining elements must shift left.

For efficient queue operations, prefer `collections.deque`.

## 3. Deque (Double-Ended Queue)

A **deque** is a double-ended queue that allows insertion and removal at both ends.

Unlike a normal queue, a deque can operate from either the front or the rear.

```text
             Front                 Rear
               ↓                    ↓
             [10, 20, 30, 40]
               ↑                    ↑
          Remove/add             Add/remove
```

### Main operations

| Operation | Meaning | Time Complexity |
|---|---|---|
| `append(x)` | Add to the right end | O(1) |
| `appendleft(x)` | Add to the left end | O(1) |
| `pop()` | Remove from the right end | O(1) |
| `popleft()` | Remove from the left end | O(1) |
| `queue[0]` | Access the leftmost element | O(1) |
| `queue[-1]` | Access the rightmost element | O(1) |

### Example

```python
from collections import deque

dq = deque([20, 30])

dq.appendleft(10)  # Add to front
dq.append(40)      # Add to rear

print(dq)
# deque([10, 20, 30, 40])

dq.popleft()
print(dq)
# deque([20, 30, 40])

dq.pop()
print(dq)
# deque([20, 30])
```

A deque is useful when a problem requires efficient operations at both ends.

## 4. Queue vs Deque

| Feature | Queue | Deque |
|---|---|---|
| Ordering | FIFO | Depends on how it is used |
| Insert at rear | Yes | Yes |
| Remove from front | Yes | Yes |
| Insert at front | Not normally allowed | Yes |
| Remove from rear | Not normally allowed | Yes |
| Typical Python implementation | `deque` | `collections.deque` |
| Common applications | BFS, task scheduling | Sliding window, monotonic queue, both-end operations |

**Important:** Python's `collections.deque` can implement both a standard FIFO queue and a double-ended queue. The difference is in how you use it.

## 5. Queue Using a Class

Understanding a basic implementation helps explain the queue's behavior.

```python
from collections import deque

class Queue:
    def __init__(self):
        self.items = deque()

    def enqueue(self, value):
        self.items.append(value)

    def dequeue(self):
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self.items.popleft()

    def peek(self):
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self.items[0]

    def is_empty(self):
        return len(self.items) == 0

    def size(self):
        return len(self.items)
```

Usage:

```python
q = Queue()

q.enqueue(10)
q.enqueue(20)

print(q.peek())      # 10
print(q.dequeue())   # 10
print(q.size())      # 1
```

This class uses a deque internally to keep enqueue and dequeue efficient.

## 6. Common Queue Problems

### Problem 1: Implement a Queue Using Two Stacks

**Idea:** Use two stacks, one for incoming elements and one for outgoing elements.

- Push every new element onto the input stack.
- When removing an element, transfer elements to the output stack only if the output stack is empty.
- Pop from the output stack.

```python
class MyQueue:
    def __init__(self):
        self.in_stack = []
        self.out_stack = []

    def push(self, x):
        self.in_stack.append(x)

    def pop(self):
        if not self.out_stack:
            while self.in_stack:
                self.out_stack.append(self.in_stack.pop())

        if not self.out_stack:
            raise IndexError("Queue is empty")

        return self.out_stack.pop()

    def peek(self):
        if not self.out_stack:
            while self.in_stack:
                self.out_stack.append(self.in_stack.pop())

        if not self.out_stack:
            raise IndexError("Queue is empty")

        return self.out_stack[-1]

    def empty(self):
        return not self.in_stack and not self.out_stack
```

**Complexity:**
- `push`: O(1)
- `pop`: O(1) amortized
- `peek`: O(1) amortized
- Space: O(n)

Each element transfers from the input stack to the output stack at most once. Therefore, the total work across many operations is linear, even though a single `pop()` can take O(n).

### Problem 2: Reverse the First K Elements of a Queue

Given a queue and an integer `k`, reverse its first `k` elements while keeping the remaining elements in their original relative order.

Example:

```text
Input:  queue = [1, 2, 3, 4, 5], k = 3
Output: [3, 2, 1, 4, 5]
```

**Approach:**
1. Remove the first `k` elements and push them onto a stack.
2. Pop the stack elements back into the queue.
3. Move the remaining `n-k` elements from the front to the rear.

```python
from collections import deque

def reverse_first_k(queue, k):
    n = len(queue)

    if k < 0 or k > n:
        raise ValueError("k must be between 0 and queue length")

    stack = []

    for _ in range(k):
        stack.append(queue.popleft())

    while stack:
        queue.append(stack.pop())

    for _ in range(n - k):
        queue.append(queue.popleft())

    return queue
```

**Complexity:** O(n) time and O(k) auxiliary space.

### Problem 3: Sliding Window Maximum

Given an array and a window size `k`, return the maximum value in every contiguous window of size `k`.

Example:

```text
Input:  nums = [1, 3, -1, -3, 5, 3, 6, 7], k = 3
Output: [3, 3, 5, 5, 6, 7]
```

A **monotonic deque** solves this in O(n) time.

Key idea:
- Store indices, not values, in the deque.
- Remove indices that fall outside the current window.
- Remove indices from the rear while their values are less than or equal to the current value.
- The front index always points to the maximum of the current window.

```python
from collections import deque

def max_sliding_window(nums, k):
    if not nums or k <= 0 or k > len(nums):
        return []

    dq = deque()
    result = []

    for i, value in enumerate(nums):
        # Remove indices outside the current window.
        while dq and dq[0] <= i - k:
            dq.popleft()

        # Maintain decreasing values from front to rear.
        while dq and nums[dq[-1]] <= value:
            dq.pop()

        dq.append(i)

        # A complete window exists from index k - 1 onward.
        if i >= k - 1:
            result.append(nums[dq[0]])

    return result
```

**Complexity:**
- Time: O(n)
- Auxiliary space: O(k), excluding the output

Each index enters the deque once and leaves it at most once, which gives linear time.

## 7. Monotonic Queue

A **monotonic queue** is a deque maintained in increasing or decreasing order of the values associated with its elements.

It is not a separate built-in Python data structure. It is a technique implemented using `collections.deque`.

### Decreasing monotonic deque

Values decrease from front to rear.

Useful for:
- Sliding window maximum
- Maintaining the maximum among candidates

When a new value arrives, remove smaller or equal values from the rear because the new value is at least as large and will remain in the window longer.

### Increasing monotonic deque

Values increase from front to rear.

Useful for:
- Sliding window minimum
- Maintaining the minimum among candidates

When a new value arrives, remove larger or equal values from the rear.

For more details, see `monotonic-stack.md` and your sliding-window notes. A monotonic queue is closely related to a monotonic stack, but it supports removing expired elements from the front.

## 8. Breadth-First Search (BFS)

Queues are essential for **Breadth-First Search**, a graph and tree traversal algorithm.

BFS visits nodes level by level. The queue stores discovered nodes that still need to be processed.

```python
from collections import deque

def bfs(graph, start):
    queue = deque([start])
    visited = {start}
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

Example graph:

```python
graph = {
    1: [2, 3],
    2: [4, 5],
    3: [6],
    4: [],
    5: [],
    6: []
}

print(bfs(graph, 1))
# [1, 2, 3, 4, 5, 6]
```

**Why mark nodes as visited when enqueuing them?**

This prevents the same node from being added to the queue multiple times when several nodes point to it.

For a graph represented with adjacency lists, BFS takes O(V + E) time and O(V) auxiliary space, where `V` is the number of vertices and `E` is the number of edges.

## 9. Circular Queue

A circular queue treats the last position of an array as connected to the first position. When the rear reaches the end, it can wrap around to the beginning.

It avoids shifting elements after each removal.

For a fixed capacity `capacity`, the next index is calculated as:

```python
next_index = (index + 1) % capacity
```

A circular queue is useful in:
- Fixed-size buffers
- Streaming systems
- Producer-consumer workflows
- Operating-system scheduling

Python's `collections.deque` is usually the simplest choice for ordinary interview queue problems. Implement a circular queue when the problem specifically asks for a fixed-capacity implementation.

## 10. Common Mistakes

1. **Using `list.pop(0)` repeatedly.** It shifts elements and costs O(n) per removal.
2. **Confusing FIFO and LIFO.** A queue is FIFO; a stack is LIFO.
3. **Using the wrong deque end.** For a normal FIFO queue, use `append()` and `popleft()`.
4. **Ignoring an empty queue.** Check before removing or reading the front element when implementing a custom queue.
5. **Storing values instead of indices in sliding-window problems.** Indices help determine whether an element has expired.
6. **Forgetting to remove expired indices.** The deque may then contain elements outside the current window.
7. **Claiming every queue operation is always O(1).** Complexity depends on the implementation; `list.pop(0)` is O(n).
8. **Forgetting edge cases.** Consider an empty input, `k = 1`, `k = len(nums)`, and invalid window sizes.

## 11. Interview Questions

Practice these in order:

### Easy
1. Implement a queue using a Python deque.
2. Implement a stack using a deque.
3. Reverse a queue.
4. Generate binary numbers from 1 to `n` using a queue.
5. Implement a queue using two stacks.

### Medium
1. Reverse the first `k` elements of a queue.
2. Generate the first negative number in every window of size `k`.
3. Find the maximum of every sliding window.
4. Implement a circular queue.
5. Design a deque with efficient operations at both ends.
6. Perform BFS on a graph or binary tree.

### Hard
1. Design a data structure that supports queue operations and retrieves the maximum efficiently.
2. Find the shortest path in an unweighted graph using BFS.
3. Solve a shortest-path problem on a grid using BFS.
4. Use a monotonic deque to optimize a dynamic programming problem.

## 12. Quick Revision

- **Queue:** FIFO — first in, first out.
- **Deque:** Insert and remove at both ends.
- **Python implementation:** `collections.deque`.
- **Enqueue:** `append(x)`.
- **Dequeue:** `popleft()`.
- **Front:** `queue[0]`.
- **Rear:** `queue[-1]`.
- **BFS:** Uses a queue to process nodes level by level.
- **Sliding window maximum:** Uses a decreasing monotonic deque of indices.
- **Efficient queue operations:** O(1) per insertion or removal with a suitable deque implementation.
- **BFS complexity:** O(V + E) time for an adjacency-list graph.

**Interview takeaway:** Know when to use a queue, why `deque` is better than removing from the front of a list, how BFS works, and how a monotonic deque can reduce a sliding-window maximum problem from O(nk) to O(n).