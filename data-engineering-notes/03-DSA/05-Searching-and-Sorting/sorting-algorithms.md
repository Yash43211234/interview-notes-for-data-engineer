# Sorting Algorithms

## 1. What Is Sorting?

**Sorting** is the process of arranging elements in a particular order, usually ascending or descending.

Example:

```python
nums = [5, 2, 8, 1, 3]

# Ascending
# [1, 2, 3, 5, 8]

# Descending
# [8, 5, 3, 2, 1]
```

Sorting is important because it makes many other operations easier, including binary search, duplicate detection, interval processing, and finding the smallest or largest elements.

## 2. Important Sorting Concepts

Before learning individual algorithms, understand these properties.

| Property | Meaning |
|---|---|
| Time complexity | How the running time grows as input size increases |
| Space complexity | How much additional memory the algorithm uses |
| Stable sort | Equal-key elements preserve their original relative order |
| In-place sort | Uses little additional memory, typically O(1) auxiliary space |
| Adaptive sort | Can take advantage of existing order in the input |
| Comparison sort | Orders elements by comparing pairs of values |

**Stable sorting example**

Suppose employees are sorted by age:

```text
Before:
(Alice, 25)
(Bob, 20)
(Carol, 25)

Stable sort by age:
(Bob, 20)
(Alice, 25)
(Carol, 25)
```

Alice and Carol have the same age, so their original relative order is preserved.

Stability is especially useful when sorting records by multiple keys.

## 3. Sorting Algorithm Complexity

Let `n` be the number of elements.

| Algorithm | Best time | Average time | Worst time | Auxiliary space | Stable? |
|---|---:|---:|---:|---:|---|
| Bubble sort | O(n)* | O(n²) | O(n²) | O(1) | Yes |
| Selection sort | O(n²) | O(n²) | O(n²) | O(1) | Usually no |
| Insertion sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Merge sort | O(n log n) | O(n log n) | O(n log n) | O(n) for arrays | Yes |
| Quicksort | O(n log n) | O(n log n) | O(n²) | O(log n) average recursion stack | Usually no |
| Heapsort | O(n log n) | O(n log n) | O(n log n) | O(1) for an in-place implementation | No |
| Counting sort | O(n + k) | O(n + k) | O(n + k) | O(n + k) for a typical stable version | Can be |
| Radix sort | O(d(n + b)) | O(d(n + b)) | O(d(n + b)) | Depends on implementation | Can be |

Here:
- `k` is the range of integer keys in counting sort.
- `d` is the number of digits or passes in radix sort.
- `b` is the base or number of buckets per pass.

\* Bubble sort achieves O(n) best-case time only when implemented with an early-exit check that detects whether a complete pass made no swaps.

The space figures describe typical implementations and can vary by algorithm and implementation details.

## 4. Bubble Sort

Bubble sort repeatedly compares adjacent elements and swaps them when they are in the wrong order.

After each pass, the largest remaining unsorted element moves to the end.

### Example

```text
Input: [5, 3, 8, 2]

Pass 1:
[3, 5, 2, 8]

Pass 2:
[3, 2, 5, 8]

Pass 3:
[2, 3, 5, 8]
```

### Implementation

```python
def bubble_sort(nums):
    nums = nums.copy()
    n = len(nums)

    for i in range(n):
        swapped = False

        for j in range(n - 1 - i):
            if nums[j] > nums[j + 1]:
                nums[j], nums[j + 1] = nums[j + 1], nums[j]
                swapped = True

        if not swapped:
            break

    return nums
```

### Complexity

- Best: O(n), when already sorted.
- Average: O(n²).
- Worst: O(n²).
- Auxiliary space: O(1), excluding the copied input.

This implementation returns a sorted copy, so the total extra memory including the copy is O(n).

**When to learn it:** Useful for understanding sorting fundamentals, but rarely appropriate for large inputs.

## 5. Selection Sort

Selection sort repeatedly finds the smallest element in the unsorted portion and places it at the beginning of that portion.

### Example

```text
Input: [5, 3, 8, 2]

Find minimum 2:
[2, 3, 8, 5]

Find minimum 3:
[2, 3, 8, 5]

Find minimum 5:
[2, 3, 5, 8]
```

### Implementation

```python
def selection_sort(nums):
    nums = nums.copy()
    n = len(nums)

    for i in range(n):
        min_index = i

        for j in range(i + 1, n):
            if nums[j] < nums[min_index]:
                min_index = j

        nums[i], nums[min_index] = nums[min_index], nums[i]

    return nums
```

### Complexity

- Best, average, worst: O(n²).
- Auxiliary space: O(1), excluding the copied input.
- Stable: No, in the usual swapping implementation.

**Key idea:** Selection sort performs relatively few swaps, but it still performs many comparisons.

## 6. Insertion Sort

Insertion sort builds a sorted portion one element at a time.

Each new element is inserted into its correct position among the elements already processed.

It works like arranging playing cards in your hand.

### Example

```text
Input: [5, 3, 8, 2]

Insert 3:
[3, 5, 8, 2]

Insert 8:
[3, 5, 8, 2]

Insert 2:
[2, 3, 5, 8]
```

### Implementation

```python
def insertion_sort(nums):
    nums = nums.copy()

    for i in range(1, len(nums)):
        key = nums[i]
        j = i - 1

        while j >= 0 and nums[j] > key:
            nums[j + 1] = nums[j]
            j -= 1

        nums[j + 1] = key

    return nums
```

### Complexity

- Best: O(n), when already sorted.
- Average: O(n²).
- Worst: O(n²).
- Auxiliary space: O(1), excluding the copied input.
- Stable: Yes.

**When is it useful?**

- Small arrays.
- Nearly sorted data.
- As a small-array optimization inside more advanced sorting algorithms.

Insertion sort is often more practical than bubble or selection sort for small inputs.

## 7. Merge Sort

Merge sort follows the **divide-and-conquer** approach.

It:
1. Divides the array into smaller halves.
2. Recursively sorts each half.
3. Merges the sorted halves into one sorted array.

### Example

```text
[8, 3, 5, 1]

Divide:
[8, 3]     [5, 1]

Divide again:
[8] [3]    [5] [1]

Merge:
[3, 8]     [1, 5]

Final merge:
[1, 3, 5, 8]
```

### Implementation

```python
def merge_sort(nums):
    if len(nums) <= 1:
        return nums.copy()

    mid = len(nums) // 2

    left = merge_sort(nums[:mid])
    right = merge_sort(nums[mid:])

    result = []
    i = j = 0

    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1

    result.extend(left[i:])
    result.extend(right[j:])

    return result
```

### Complexity

- Best, average, worst: O(n log n).
- Auxiliary space: O(n) for a typical array implementation.
- Stable: Yes, when equal elements are merged consistently, as in this implementation.

### Why is the time complexity O(n log n)?

The recursion divides the array into halves, producing approximately log n levels.

At each level, merging processes a total of n elements.

Therefore:

\[
T(n) = O(n\log n)
\]

**When is it useful?**

- When predictable O(n log n) performance is important.
- When stable sorting is required.
- When sorting linked lists.
- In external sorting, where data may be too large to fit into memory.

Python list slicing and intermediate arrays also use memory, so this implementation is not an in-place merge sort.

## 8. Quicksort

Quicksort also uses divide and conquer, but it partitions elements around a **pivot**.

After partitioning, elements on one side belong before the pivot and elements on the other side belong after it. The two partitions are then sorted recursively.

### Example

```text
Input: [7, 2, 9, 4, 3]

Choose pivot = 4

Partition:
[2, 3]  4  [7, 9]

Sort the partitions:
[2, 3, 4, 7, 9]
```

The exact partition depends on the chosen implementation.

### Partition-based implementation

```python
def quick_sort(nums):
    nums = nums.copy()

    def sort(left, right):
        if left >= right:
            return

        pivot = nums[right]
        boundary = left

        for i in range(left, right):
            if nums[i] <= pivot:
                nums[boundary], nums[i] = nums[i], nums[boundary]
                boundary += 1

        nums[boundary], nums[right] = nums[right], nums[boundary]

        sort(left, boundary - 1)
        sort(boundary + 1, right)

    sort(0, len(nums) - 1)
    return nums
```

### Complexity

- Best: O(n log n).
- Average: O(n log n).
- Worst: O(n²).
- Auxiliary space: O(log n) average recursion stack; O(n) worst-case recursion stack.
- Stable: No, in this implementation.

### Why can quicksort become O(n²)?

If each pivot repeatedly produces one partition of size n − 1 and another of size zero, the recursion becomes highly unbalanced.

For example, choosing the last element as the pivot for already sorted data can cause this behavior in the implementation above.

Randomized pivot selection or better pivot strategies reduce the likelihood of consistently poor partitions, but do not change the theoretical worst-case bound of ordinary quicksort.

**When is it useful?**

Quicksort is important to understand because of its partitioning technique, which also appears in quickselect and related selection problems.

## 9. Heapsort

Heapsort uses a **binary heap**, a tree-like structure that allows efficient access to the largest or smallest element.

For ascending order, heapsort typically builds a max-heap and repeatedly moves its largest element to the end.

### Main steps

1. Build a max-heap.
2. Swap the root with the last element in the heap.
3. Reduce the heap size.
4. Restore the heap property.
5. Repeat until sorted.

### Implementation

```python
def heap_sort(nums):
    nums = nums.copy()
    n = len(nums)

    def sift_down(size, root):
        while True:
            largest = root
            left = 2 * root + 1
            right = 2 * root + 2

            if left < size and nums[left] > nums[largest]:
                largest = left

            if right < size and nums[right] > nums[largest]:
                largest = right

            if largest == root:
                break

            nums[root], nums[largest] = nums[largest], nums[root]
            root = largest

    # Build max-heap.
    for i in range(n // 2 - 1, -1, -1):
        sift_down(n, i)

    # Move the maximum to the end repeatedly.
    for end in range(n - 1, 0, -1):
        nums[0], nums[end] = nums[end], nums[0]
        sift_down(end, 0)

    return nums
```

### Complexity

- Best, average, worst: O(n log n).
- Auxiliary space: O(1) for the in-place algorithm, excluding the copied input.
- Stable: No.

**Important:** Building the heap takes O(n), not O(n log n). The repeated extraction operations account for the O(n log n) overall time.

Heapsort is useful when guaranteed O(n log n) time and constant auxiliary space are important, though it is not a common default choice in Python application code.

## 10. Non-Comparison Sorting

Comparison sorts use comparisons to determine the ordering of elements. Some algorithms can be faster under specific assumptions about the input.

### Counting sort

Counting sort counts how many times each integer key occurs.

It works well when the range of integer values is relatively small.

```python
def counting_sort(nums):
    if not nums:
        return []

    minimum = min(nums)
    maximum = max(nums)
    counts = [0] * (maximum - minimum + 1)

    for value in nums:
        counts[value - minimum] += 1

    result = []

    for i, count in enumerate(counts):
        result.extend([i + minimum] * count)

    return result
```

Example:

```python
print(counting_sort([4, 2, 2, 8, 3, 3, 1]))
# [1, 2, 2, 3, 3, 4, 8]
```

Complexity:
- Time: O(n + k).
- Auxiliary space: O(k + n) for this implementation.

Here, `k = maximum - minimum + 1`.

This version sorts integer values, including negative values, but may use excessive memory if the range is very large. It is not a stable record-sorting implementation.

### Radix sort

Radix sort sorts values one digit or character position at a time using a stable sorting pass.

Its complexity is commonly expressed as:

\[
O(d(n+b))
\]

where `d` is the number of passes and `b` is the number of possible digit values per pass.

Radix sort can be effective for fixed-width integers or strings when its assumptions fit the data. It is not automatically faster than comparison sorting for every input.

## 11. Python's Built-in Sorting

In real Python applications, prefer the built-in sorting tools unless a problem specifically asks you to implement an algorithm.

### `sorted()`

Returns a new sorted list.

```python
nums = [5, 2, 8, 1, 3]

result = sorted(nums)

print(result)  # [1, 2, 3, 5, 8]
print(nums)    # [5, 2, 8, 1, 3]
```

### `.sort()`

Sorts a list in place and returns `None`.

```python
nums = [5, 2, 8, 1, 3]

nums.sort()

print(nums)  # [1, 2, 3, 5, 8]
```

### Descending order

```python
nums = [5, 2, 8, 1, 3]

print(sorted(nums, reverse=True))
# [8, 5, 3, 2, 1]
```

### Sort by a key

```python
employees = [
    {"name": "Aman", "salary": 50000},
    {"name": "Riya", "salary": 70000},
    {"name": "Neha", "salary": 60000},
]

result = sorted(employees, key=lambda employee: employee["salary"])

print(result)
```

This sorts employees by salary in ascending order.

For multiple keys:

```python
employees = [
    ("Aman", 25),
    ("Riya", 22),
    ("Neha", 25),
]

result = sorted(employees, key=lambda employee: (employee[1], employee[0]))
```

This sorts first by age and then by name.

### How does Python sort?

Python uses **Timsort**, a stable hybrid sorting algorithm that combines ideas from merge sort and insertion sort and takes advantage of existing ordered runs.

- Worst-case time: O(n log n).
- Best-case time: O(n) on already ordered or suitably structured input.
- Stability: Yes.

`sorted()` creates a new list, so the returned list requires O(n) space. `.sort()` modifies the original list and also uses temporary memory internally.

## 12. How Sorting Helps Solve DSA Problems

Sorting is not only about putting numbers in order. It is often a preprocessing step that simplifies a larger problem.

### A. Two Sum

If an array is sorted, two pointers can find a pair with a target sum in O(n) time.

However, sorting an unsorted array first costs O(n log n), and preserving original indices may require storing them alongside values.

### B. Merge Intervals

Sort intervals by their starting points, then scan once to merge overlapping intervals.

Typical complexity: O(n log n) because of sorting.

### C. Remove Duplicates

In a sorted array, duplicates appear next to each other, making them easier to identify.

### D. Find the Kth Largest Element

Possible approaches include:
- Sorting the full array: O(n log n).
- Using a heap: O(n log k) for a size-k heap.
- Using quickselect: O(n) average time, O(n²) worst case.

### E. Binary Search

Sorting enables binary search when the problem requires locating a target in ordered data. Remember that sorting an unsorted array has its own cost.

## 13. Common Mistakes

1. **Confusing in-place modification with a returned result.** `list.sort()` returns `None`; `sorted()` returns a new list.
2. **Ignoring stability.** Stability matters when equal-key records must preserve their original order.
3. **Claiming quicksort always takes O(n log n).** Its worst case is O(n²).
4. **Claiming every sorting algorithm is in-place.** Merge sort on arrays usually needs O(n) extra space.
5. **Ignoring the input range in counting sort.** A large range can require excessive memory.
6. **Sorting unnecessarily.** Sorting adds cost and can destroy the relationship between elements and their original indices.
7. **Forgetting edge cases.** Test empty lists, one-element lists, duplicate values, already sorted data, reverse-sorted data, and negative numbers where supported.
8. **Writing a nested loop without analyzing it.** Bubble, selection, and insertion sort can take O(n²) time.
9. **Confusing heap construction with repeated extraction.** Building a heap is O(n); extracting all elements produces O(n log n) total time.
10. **Using a custom sort in production without a reason.** Python's built-in sorting is highly optimized and usually preferable.

## 14. Interview Questions

### Fundamentals

1. What is the difference between stable and unstable sorting?
2. What does in-place sorting mean?
3. Why does merge sort take O(n log n) time?
4. Why can quicksort degrade to O(n²)?
5. Why is insertion sort useful for small or nearly sorted arrays?
6. What is the difference between `sorted()` and `.sort()`?
7. Why does counting sort depend on the range of values?

### Coding problems

1. Implement bubble sort.
2. Implement insertion sort.
3. Implement merge sort.
4. Implement quicksort.
5. Merge two sorted arrays.
6. Merge overlapping intervals.
7. Sort an array containing only 0s, 1s, and 2s.
8. Find the kth largest element.
9. Count inversions in an array.
10. Find the minimum number of swaps needed to sort an array.

## 15. Quick Revision

- **Bubble sort:** Repeatedly swaps adjacent out-of-order elements.
- **Selection sort:** Repeatedly selects the minimum element from the unsorted region.
- **Insertion sort:** Inserts each new element into the sorted region.
- **Merge sort:** Divides, sorts recursively, and merges; O(n log n) worst-case time.
- **Quicksort:** Partitions around a pivot; O(n log n) average and O(n²) worst-case time.
- **Heapsort:** Uses a heap; O(n log n) worst-case time and O(1) auxiliary space for an in-place implementation.
- **Counting sort:** Counts integer keys; useful for small value ranges.
- **Radix sort:** Sorts digit by digit using stable passes.
- **Python sorting:** Timsort is stable and provides O(n log n) worst-case time.
- **Sorting in DSA:** Often enables simpler solutions for intervals, duplicates, two pointers, and binary search.

**Interview takeaway:** Know the time and space complexity, stability, and basic working principle of each major sorting algorithm. Focus most on merge sort, quicksort, insertion sort, and Python's built-in sorting, then practise problems where sorting enables a more efficient solution.