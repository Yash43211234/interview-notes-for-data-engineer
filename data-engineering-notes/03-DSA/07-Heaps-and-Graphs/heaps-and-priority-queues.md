# Heaps and Priority Queues

## 1. What Is a Heap?

A **heap** is a specialized tree-based data structure that satisfies a specific ordering property called the heap property.

In practice, a binary heap is usually implemented using an array rather than explicit tree nodes.

There are two main types:

- **Min-heap:** The smallest element is at the root.
- **Max-heap:** The largest element is at the root.

A heap is a **complete binary tree**, meaning every level is completely filled except possibly the last, which is filled from left to right.

### Example: Min-Heap

```text
        2
       / \
      5   8
     / \
    10  12
```

The parent is always less than or equal to its children.

The root contains `2`, the smallest element.

### Example: Max-Heap

```text
        12
       /  \
      10   8
     / \
    2   5
```

The parent is always greater than or equal to its children.

The root contains `12`, the largest element.

**Important:** A heap is not a fully sorted tree. It only guarantees the ordering relationship between parents and children.

---

## 2. Heap vs. Binary Search Tree

| Feature | Heap | Binary Search Tree (BST) |
|---|---|---|
| Main purpose | Quickly access the minimum or maximum | Search and maintain ordered data |
| Root | Minimum or maximum, depending on heap type | Depends on inserted values |
| Ordering | Parent-child heap property | Left subtree values are smaller; right subtree values are larger under the standard BST rule |
| Find minimum | \(O(1)\) in a min-heap | \(O(h)\) |
| Search for arbitrary value | \(O(n)\) | \(O(h)\) |
| Insert | \(O(\log n)\) | \(O(h)\) |
| Remove root / extremum | \(O(\log n)\) | Depends on the operation |

Here, \(h\) is the tree height. A balanced BST has \(O(\log n)\) height, but an unbalanced BST can have \(O(n)\) height.

---

## 3. What Is a Priority Queue?

A **priority queue** is an abstract data type where elements are removed according to priority rather than insertion order.

For example, suppose a hospital assigns priority to patients based on urgency. The most urgent patient should be handled first.

A priority queue supports operations such as:

- Insert an element with a priority.
- Access the highest-priority element.
- Remove the highest-priority element.

A heap is one of the most common ways to implement a priority queue.

### Heap vs. Priority Queue

- **Heap:** A data structure with a specific ordering property.
- **Priority queue:** An interface or abstract data type that defines priority-based operations.

A min-heap can implement a priority queue that processes the smallest priority value first. A max-heap can implement one that processes the largest priority value first.

---

## 4. Heap Representation Using an Array

A binary heap can be stored in an array without creating separate tree nodes.

Consider this min-heap:

```text
        2
       / \
      5   8
     / \
    10  12
```

Its array representation is:

```python
heap = [2, 5, 8, 10, 12]
```

For an element at index `i`, using zero-based indexing:

| Relationship | Formula |
|---|---|
| Parent | `(i - 1) // 2` |
| Left child | `2 * i + 1` |
| Right child | `2 * i + 2` |

For example, the element at index `1` is `5`.

- Parent index: `(1 - 1) // 2 = 0`
- Left child index: `2 * 1 + 1 = 3`
- Right child index: `2 * 1 + 2 = 4`

Therefore, `5` has parent `2` and children `10` and `12`.

These formulas work because a binary heap is a complete binary tree.

---

## 5. Heap Operations and Complexity

For a binary heap containing \(n\) elements:

| Operation | Time complexity |
|---|---:|
| Read the minimum in a min-heap | \(O(1)\) |
| Read the maximum in a max-heap | \(O(1)\) |
| Insert an element | \(O(\log n)\) |
| Remove the root | \(O(\log n)\) |
| Build a heap from an array | \(O(n)\) |
| Search for an arbitrary value | \(O(n)\) |
| Heap sort | \(O(n\log n)\) |

Why are insertion and root removal \(O(\log n)\)?

A complete binary tree has height \(O(\log n)\). These operations may need to move an element along the height of the tree to restore the heap property.

Building a heap from an entire array can be done in \(O(n)\) time using bottom-up heapification. It is not necessary to insert every element separately.

---

## 6. Using Heaps in Python

Python provides the `heapq` module for heap operations.

By default, `heapq` implements a **min-heap**.

### 6.1 Create a Min-Heap

```python
import heapq

heap = [7, 2, 9, 1, 5]

heapq.heapify(heap)

print(heap[0])
# 1
```

`heapify()` transforms the list into a valid min-heap in \(O(n)\) time.

The list is not sorted after heapification. Only the heap property is guaranteed.

### 6.2 Insert an Element

```python
import heapq

heap = [2, 5, 8]
heapq.heapify(heap)

heapq.heappush(heap, 1)

print(heap[0])
# 1
```

`heappush()` inserts an element and restores the heap property.

Time complexity: \(O(\log n)\).

### 6.3 Remove the Smallest Element

```python
import heapq

heap = [2, 5, 8]
heapq.heapify(heap)

smallest = heapq.heappop(heap)

print(smallest)
# 2
```

`heappop()` removes and returns the smallest element.

Time complexity: \(O(\log n)\).

### 6.4 Access the Smallest Element Without Removing It

```python
print(heap[0])
```

Reading the root takes \(O(1)\) time when the heap is non-empty.

### 6.5 Get the Smallest or Largest K Elements

```python
import heapq

nums = [7, 2, 9, 1, 5, 3]

print(heapq.nsmallest(3, nums))
# [1, 2, 3]

print(heapq.nlargest(3, nums))
# [9, 7, 5]
```

These functions are convenient when you need only a small number of extreme values.

Their actual performance depends on the number of requested elements relative to the input size.

---

## 7. Implementing a Max-Heap in Python

Since `heapq` is a min-heap, a common technique is to store negative values.

```python
import heapq

nums = [7, 2, 9, 1, 5]

max_heap = [-num for num in nums]
heapq.heapify(max_heap)

largest = -heapq.heappop(max_heap)

print(largest)
# 9
```

Why does this work?

The smallest negative number represents the largest original number.

For example:

```text
Original values:  7,  2,  9,  1,  5
Negative values: -7, -2, -9, -1, -5
```

The minimum negative value is `-9`, which corresponds to the original value `9`.

For insertion and removal:

```python
# Insert 12
heapq.heappush(max_heap, -12)

# Remove the largest original value
largest = -heapq.heappop(max_heap)
```

**Important:** Negating numbers is suitable for numeric priorities. For custom objects or more complex priorities, use tuples or a suitable comparison strategy.

---

## 8. Priority Queues with Tuples

A common interview pattern is storing a priority together with an associated value.

Python compares tuples lexicographically: first by the first element, then by the second element if the first elements are equal, and so on.

### Example: Process Tasks by Priority

```python
import heapq

tasks = [
    (3, "Send report"),
    (1, "Fix production issue"),
    (2, "Review code")
]

heapq.heapify(tasks)

while tasks:
    priority, task = heapq.heappop(tasks)
    print(priority, task)
```

Output:

```text
1 Fix production issue
2 Review code
3 Send report
```

The smallest numerical priority is processed first.

If two tasks have equal priority, Python compares their next tuple elements. If those elements are not comparable, the comparison can raise a `TypeError`.

A common solution is to include a unique counter:

```python
import heapq
from itertools import count

counter = count()
heap = []

heapq.heappush(heap, (1, next(counter), {"task": "A"}))
heapq.heappush(heap, (1, next(counter), {"task": "B"}))

while heap:
    priority, _, task = heapq.heappop(heap)
    print(priority, task)
```

The counter breaks priority ties before Python attempts to compare the task objects.

---

## 9. Important Heap Patterns for Interviews

### 9.1 Kth Largest Element

**Problem:** Find the kth largest element in an array.

A useful approach is to maintain a min-heap of size `k`.

The heap contains the `k` largest elements seen so far. Its root is the smallest among those elements, which is the kth largest overall.

```python
import heapq

def find_kth_largest(nums, k):
    heap = []

    for num in nums:
        heapq.heappush(heap, num)

        if len(heap) > k:
            heapq.heappop(heap)

    return heap[0]
```

Example:

```python
print(find_kth_largest([3, 2, 1, 5, 6, 4], 2))
# 5
```

Complexity:

- Time: \(O(n\log k)\)
- Extra space: \(O(k)\)

This approach is especially useful when `k` is much smaller than `n`.

### 9.2 Top K Frequent Elements

**Problem:** Return the `k` most frequent elements.

One approach is to count frequencies and maintain a min-heap of size `k`.

```python
import heapq
from collections import Counter

def top_k_frequent(nums, k):
    frequency = Counter(nums)

    heap = []

    for num, count in frequency.items():
        heapq.heappush(heap, (count, num))

        if len(heap) > k:
            heapq.heappop(heap)

    return [num for count, num in heap]
```

This implementation returns the correct set of top `k` elements, but their output order is not guaranteed to be from most frequent to least frequent. When frequencies tie, the numeric value is used as a secondary ordering key.

If the question requires a particular order, explicitly sort the selected results or use a different heap ordering.

### 9.3 Merge K Sorted Lists

A heap can efficiently merge multiple sorted sequences by tracking the smallest available element from each sequence.

The general approach:

1. Insert the first element from each non-empty list into a min-heap.
2. Remove the smallest element.
3. Add it to the result.
4. Insert the next element from the same list, if one exists.
5. Repeat until the heap is empty.

For a total of \(N\) elements across \(k\) lists, the time complexity is typically \(O(N\log k)\), with \(O(k)\) heap space, excluding the output.

### 9.4 Find the K Closest Elements

A heap can keep track of the closest points or values according to a distance function.

For points \((x,y)\), squared Euclidean distance from the origin is:

\[
d^2=x^2+y^2
\]

Squared distance preserves the ordering of actual distances and avoids computing square roots.

For large inputs and small `k`, a bounded heap can be more efficient than sorting all elements.

### 9.5 Running Median

To maintain the median of a stream of numbers, use two heaps:

- A max-heap for the smaller half.
- A min-heap for the larger half.

Maintain two rules:

1. Every element in the smaller half is less than or equal to every element in the larger half.
2. The heap sizes differ by at most one.

With Python's `heapq`, store negated values for the max-heap.

Insertion takes \(O(\log n)\). Reading the median takes \(O(1)\) after the heaps are balanced.

---

## 10. Heap vs. Sorting

Suppose you have `n` numbers and need only the largest `k` elements.

| Approach | Typical time complexity |
|---|---:|
| Sort all elements | \(O(n\log n)\) |
| Maintain a heap of size `k` | \(O(n\log k)\) |
| Build a heap of all elements and pop `k` times | \(O(n+k\log n)\) |

When `k` is small relative to `n`, a bounded heap can avoid sorting the entire input.

If you need the complete sorted order of every element, sorting is often the simpler choice.

---

## 11. Common Mistakes

1. **Assuming a heap is sorted:** The heap only guarantees the parent-child ordering property.
2. **Using a min-heap when the problem requires a max-heap:** Remember that Python's `heapq` is a min-heap by default.
3. **Removing the wrong element:** `heappop()` removes the minimum element, not an arbitrary element.
4. **Forgetting to limit heap size:** For kth-largest problems, a min-heap should usually contain at most `k` elements.
5. **Ignoring tie-breaking:** Tuples may compare later fields when priorities are equal.
6. **Using a heap for arbitrary search:** Searching for a particular value generally takes \(O(n)\).
7. **Assuming heap construction is \(O(n\log n)\):** Bottom-up `heapify()` is \(O(n)\).
8. **Confusing priority with insertion order:** A priority queue removes according to its priority rules, not necessarily insertion order.

---

## 12. Interview Questions

### Conceptual Questions

1. What is a heap?
2. What is the difference between a min-heap and a max-heap?
3. Why is a binary heap a complete binary tree?
4. How is a heap represented using an array?
5. What is the difference between a heap and a priority queue?
6. Why does insertion take \(O(\log n)\)?
7. Why does `heapify()` take \(O(n)\)?
8. Can a heap efficiently search for an arbitrary element?
9. Why can a heap be preferable to sorting?
10. How do two heaps help find the running median?

### Coding Problems

Practice in this order:

1. Implement a min-heap using Python's `heapq`.
2. Find the kth largest element.
3. Find the kth smallest element.
4. Find the top k frequent elements.
5. Find the k closest points to the origin.
6. Merge k sorted lists.
7. Find the running median of a data stream.
8. Reorganize a string using character frequencies and a heap.
9. Find the minimum cost to connect ropes.
10. Find the smallest range covering elements from k sorted lists.

---

## 13. Quick Revision

- A heap is a complete binary tree that satisfies a heap property.
- A min-heap exposes the smallest element at its root.
- A max-heap exposes the largest element at its root.
- Python's `heapq` implements a min-heap.
- `heapq.heapify()` builds a heap in \(O(n)\).
- `heappush()` and `heappop()` take \(O(\log n)\).
- Reading the root takes \(O(1)\).
- Searching for an arbitrary element takes \(O(n)\).
- A bounded heap is useful for kth-element and top-k problems.
- Two heaps can maintain the running median.
- A heap does not maintain a fully sorted array.

**Recommended learning order:** Heap fundamentals → Python `heapq` → kth largest/smallest → top-k problems → merge k sorted lists → two heaps and running median.