# Two Pointers — DSA Interview Notes

## 1. What Is the Two-Pointer Technique?

The two-pointer technique uses two indices or references to process a data structure efficiently.

Instead of repeatedly checking every possible pair of elements, you move two pointers according to the problem's conditions.

In Python, pointers are usually integer indices that refer to positions in a list or string.

Example:

```python
nums = [1, 2, 3, 4, 5]

left = 0
right = len(nums) - 1

print(nums[left])   # 1
print(nums[right])  # 5
```

Two pointers are commonly used for:

- Finding pairs that satisfy a condition.
- Reversing an array.
- Removing duplicates.
- Moving elements in place.
- Comparing strings.
- Merging sorted arrays.
- Detecting cycles in linked lists.

The key is knowing **where each pointer starts, when it moves, and why that movement is safe**.

## 2. Types of Two-Pointer Patterns

There are three common patterns.

| Pattern | Pointer movement | Typical problems |
|---|---|---|
| Opposite directions | Start at opposite ends and move inward | Pair sum, reversing an array |
| Same direction | Both pointers move forward | Removing duplicates, partitioning |
| Different speeds | One pointer moves faster than the other | Cycle detection, finding a middle node |

---

## 3. Pattern 1: Opposite-Direction Pointers

Both pointers start at opposite ends of an array and move toward each other.

```text
[1, 2, 3, 4, 5]
 ↑           ↑
left        right
```

This pattern is especially useful for sorted arrays and in-place operations.

### Example A: Reverse an array

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
print(reverse_array([1, 2, 3, 4, 5]))
# [5, 4, 3, 2, 1]
```

**How it works**

1. Swap the first and last elements.
2. Move `left` one position forward.
3. Move `right` one position backward.
4. Continue until the pointers meet or cross.

**Complexity:**

- Time: O(n)
- Extra space: O(1)

The original list is modified.

### Example B: Two Sum in a sorted array

Given a sorted array, determine whether two elements add up to a target.

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
print(has_pair_sum([1, 2, 4, 6, 8], 10))
# True
```

**Why does the pointer movement work?**

Because the array is sorted:

- If the sum is too small, the left value must increase.
- If the sum is too large, the right value must decrease.
- If the sum matches the target, the pair has been found.

This reasoning relies on sorted order. It does not generally work on an unsorted array.

**Complexity:**

- Time: O(n)
- Extra space: O(1)

### Example C: Valid palindrome

Check whether a string reads the same forward and backward.

```python
def is_palindrome(s):
    left = 0
    right = len(s) - 1

    while left < right:
        if s[left] != s[right]:
            return False

        left += 1
        right -= 1

    return True
```

Example:

```python
print(is_palindrome("racecar"))  # True
print(is_palindrome("hello"))    # False
```

This version compares characters exactly, including case and punctuation. If a problem requires ignoring spaces, punctuation, or capitalization, normalize the string first or skip irrelevant characters while moving the pointers.

**Complexity:**

- Time: O(n)
- Extra space: O(1)

---

## 4. Pattern 2: Same-Direction Pointers

Both pointers move forward, but they have different responsibilities.

Common roles:

- `read`: examines the input.
- `write`: marks where the next accepted element should be stored.

This pattern is useful when modifying an array in place.

### Example A: Remove duplicates from a sorted array

Given a sorted array, remove duplicate values in place and return the number of unique elements.

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

**How it works**

1. The first element is already unique, so `write` starts at `1`.
2. `read` examines each remaining element.
3. If the element differs from the last accepted value, copy it to `nums[write]`.
4. Advance `write`.

Only the first `length` elements are guaranteed to contain the deduplicated result. The remaining elements may still contain old values.

**Complexity:**

- Time: O(n)
- Extra space: O(1)

The input must be sorted for this specific algorithm.

### Example B: Move zeroes to the end

Move all zeroes to the end while preserving the order of nonzero elements.

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

The first loop compacts all nonzero elements. The second fills the remaining positions with zeroes.

**Complexity:**

- Time: O(n)
- Extra space: O(1)

### Example C: Remove a specific value

```python
def remove_element(nums, value):
    write = 0

    for read in range(len(nums)):
        if nums[read] != value:
            nums[write] = nums[read]
            write += 1

    return write
```

Example:

```python
nums = [3, 2, 2, 3, 4]

length = remove_element(nums, 3)

print(length)          # 3
print(nums[:length])   # [2, 2, 4]
```

This is another form of the read/write pattern.

---

## 5. Pattern 3: Fast and Slow Pointers

In this pattern, two pointers move at different speeds or according to different rules.

It is especially useful for linked lists, although related ideas can also appear in arrays.

### Example A: Find the middle of a linked list

Suppose `head` is the head node of a singly linked list.

```python
def middle_node(head):
    slow = head
    fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    return slow
```

**How it works**

- `slow` moves one node at a time.
- `fast` moves two nodes at a time.
- When `fast` reaches the end, `slow` is at the middle.

For a list with an even number of nodes, this implementation returns the second middle node.

**Complexity:**

- Time: O(n)
- Extra space: O(1)

### Example B: Detect a cycle in a linked list

```python
def has_cycle(head):
    slow = head
    fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

        if slow is fast:
            return True

    return False
```

If a cycle exists, the fast pointer eventually catches the slow pointer. If no cycle exists, the fast pointer reaches the end.

Use `is` to compare node identity rather than comparing node values.

**Complexity:**

- Time: O(n)
- Extra space: O(1)

---

## 6. Pattern 4: Merging Two Sorted Arrays

Two pointers can process two sorted arrays together without repeatedly searching either array.

### Example: Merge sorted arrays

```python
def merge_sorted(nums1, nums2):
    i = 0
    j = 0
    result = []

    while i < len(nums1) and j < len(nums2):
        if nums1[i] <= nums2[j]:
            result.append(nums1[i])
            i += 1
        else:
            result.append(nums2[j])
            j += 1

    result.extend(nums1[i:])
    result.extend(nums2[j:])

    return result
```

Example:

```python
print(merge_sorted([1, 3, 5], [2, 4, 6]))
# [1, 2, 3, 4, 5, 6]
```

At every step, select the smaller of the two current elements. Once one array is exhausted, append the remaining elements from the other array.

**Complexity:**

- Time: O(n + m)
- Extra space: O(n + m)

Here, `n` and `m` are the lengths of the two input arrays.

This pattern is fundamental to merge sort and is useful for merging sorted datasets.

---

## 7. Pattern 5: Three-Sum

The three-sum problem asks for unique triplets whose values add up to a target, commonly zero.

A standard approach combines sorting with two pointers.

```python
def three_sum(nums):
    nums.sort()
    result = []
    n = len(nums)

    for i in range(n - 2):
        if i > 0 and nums[i] == nums[i - 1]:
            continue

        left = i + 1
        right = n - 1

        while left < right:
            total = nums[i] + nums[left] + nums[right]

            if total == 0:
                result.append([nums[i], nums[left], nums[right]])
                left += 1
                right -= 1

                while left < right and nums[left] == nums[left - 1]:
                    left += 1

                while left < right and nums[right] == nums[right + 1]:
                    right -= 1

            elif total < 0:
                left += 1
            else:
                right -= 1

    return result
```

Example:

```python
print(three_sum([-1, 0, 1, 2, -1, -4]))
# [[-1, -1, 2], [-1, 0, 1]]
```

The algorithm:

1. Sorts the array.
2. Fixes one element at index `i`.
3. Uses two pointers to find pairs that complete the target sum.
4. Skips duplicate values to avoid repeated triplets.

**Complexity:**

- Time: O(n²)
- Auxiliary space: O(1) apart from sorting, depending on the sorting implementation.
- Output space is separate and depends on the number of triplets found.

Sorting usually dominates the setup cost, but the nested search for each fixed element determines the O(n²) total time.

---

## 8. Two Pointers vs. Nested Loops

Consider finding two elements that add up to a target.

### Nested-loop approach

```python
def pair_exists_brute_force(nums, target):
    for i in range(len(nums)):
        for j in range(i + 1, len(nums)):
            if nums[i] + nums[j] == target:
                return True

    return False
```

This takes O(n²) time in the worst case.

### Two-pointer approach

When the array is sorted, two pointers can solve the same decision problem in O(n) time.

The important distinction is that **two pointers do not automatically improve every problem**. The pointer movements must eliminate possibilities without discarding a valid answer.

---

## 9. How to Recognize Two-Pointer Problems

Look for these clues:

| Problem clue | Possible approach |
|---|---|
| Sorted array and target pair | Opposite-direction pointers |
| Reverse an array or string | Opposite-direction pointers |
| Remove duplicates in place | Same-direction pointers |
| Move elements while preserving order | Read/write pointers |
| Find a linked-list middle node | Fast/slow pointers |
| Detect a linked-list cycle | Fast/slow pointers |
| Merge sorted inputs | One pointer per input |
| Find triplets with a target sum | Sorting + two pointers |

Before implementing the solution, ask:

1. Does the input need to be sorted?
2. Where should each pointer start?
3. What condition makes a pointer move?
4. Why is it safe to discard the current possibility?
5. Can every element be processed only a constant number of times?

---

## 10. Common Mistakes

1. **Using two pointers on an unsorted array when the algorithm requires sorted order.**
2. **Using `left <= right` when the problem requires two distinct indices.** For pair-sum problems, `left < right` is commonly correct.
3. **Skipping duplicates incorrectly.** This is especially important in three-sum and other problems requiring unique combinations.
4. **Moving the wrong pointer.** For sorted pair-sum problems, a small sum means moving `left`; a large sum means moving `right`.
5. **Overwriting unprocessed values.** In-place algorithms must preserve values that still need to be read.
6. **Returning the wrong portion of an array.** Some in-place problems require returning a valid length, not a new list.
7. **Confusing node values with node identity.** Cycle detection must determine whether pointers reference the same node.
8. **Assuming two pointers always require O(1) extra space.** Merging arrays into a new list requires additional space.

---

## 11. Practice Problems

### Easy

1. Reverse String.
2. Valid Palindrome.
3. Two Sum II — Input Array Is Sorted.
4. Remove Duplicates from Sorted Array.
5. Move Zeroes.
6. Merge Sorted Array.
7. Linked List Cycle.
8. Middle of the Linked List.

### Medium

1. 3Sum.
2. Container With Most Water.
3. Sort Colors.
4. Remove Nth Node From End of List.
5. Find the Duplicate Number.
6. Boats to Save People.
7. Partition Labels.
8. Trapping Rain Water.

### Recommended practice order

1. Reverse an array.
2. Solve pair sum on a sorted array.
3. Practise read/write pointers.
4. Merge two sorted arrays.
5. Learn fast/slow pointers on linked lists.
6. Solve three-sum.
7. Move to problems that combine sorting, greedy reasoning, and two pointers.

---

## Quick Revision

- **Opposite-direction pointers:** Start at both ends and move inward.
- **Same-direction pointers:** Process elements while maintaining a read position and a write position.
- **Fast/slow pointers:** Move at different speeds to detect structure or cycles.
- **Merging pointers:** Process multiple sorted inputs efficiently.
- **Sorted pair sum:** Move `left` when the sum is too small; move `right` when it is too large.
- **Typical complexity:** O(n) for many basic two-pointer problems; O(n²) for patterns such as three-sum.
- **Key requirement:** Each pointer movement must be justified by the problem's properties.

**Interview goal:** Explain the pointer invariant, justify every movement, and show how the technique reduces unnecessary comparisons.