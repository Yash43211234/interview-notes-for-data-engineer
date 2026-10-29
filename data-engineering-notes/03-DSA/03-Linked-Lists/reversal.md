# Reversal Pattern in DSA

## 1. Overview

Reversal is a problem-solving pattern in which the order of elements, characters, pointers, or links is changed.

Reversal problems commonly appear in arrays, strings, linked lists, and certain data structures.

The main approaches are:

- **Two pointers:** Reverse elements in an array or string-like sequence.
- **Pointer manipulation:** Reverse the links in a linked list.
- **Repeated reversal:** Reverse selected portions of a sequence to achieve a required arrangement.
- **Stack:** Reverse the processing order using last-in, first-out behavior when appropriate.

The best approach depends on whether the input is mutable, whether extra space is allowed, and what part of the input must be reversed.

## 2. Reverse an Array In Place

**Problem:** Reverse an array without creating another array.

### Solution

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
nums = [1, 2, 3, 4, 5]
print(reverse_array(nums))
# [5, 4, 3, 2, 1]
```

### How it works

1. Place one pointer at the beginning.
2. Place the other pointer at the end.
3. Swap the elements at those positions.
4. Move both pointers toward the center.
5. Stop when the pointers meet or cross.

### Complexity

- Time: `O(n)`
- Auxiliary space: `O(1)`

**Pattern:** Two pointers.

**Common mistake:** Using `left <= right` is also correct for this swap-based implementation, but it performs an unnecessary self-swap when both pointers meet. `left < right` avoids that operation.

---

## 3. Reverse a String

Python strings are immutable, so their characters cannot be swapped in place.

### Approach 1: Slicing

```python
def reverse_string(s):
    return s[::-1]
```

- Time: `O(n)`
- Extra space: `O(n)` for the resulting string.

### Approach 2: Two pointers on a list

```python
def reverse_string(s):
    chars = list(s)
    left = 0
    right = len(chars) - 1

    while left < right:
        chars[left], chars[right] = chars[right], chars[left]
        left += 1
        right -= 1

    return "".join(chars)
```

- Time: `O(n)`
- Extra space: `O(n)` for the character list and output string.

**Important:** Python string reversal reverses Unicode code points, not necessarily user-perceived characters. Some visible characters consist of multiple code points.

---

## 4. Reverse a Singly Linked List

**Problem:** Reverse the links of a singly linked list and return the new head.

Assume each node contains `val` and `next`.

### Iterative solution

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

### Example

Original:

`1 → 2 → 3 → 4 → None`

Reversed:

`4 → 3 → 2 → 1 → None`

### How it works

The three important pointers are:

- `previous`: The already reversed portion.
- `current`: The node currently being processed.
- `next_node`: The remaining unreversed portion.

At each iteration:

1. Save the next node.
2. Point the current node backward to `previous`.
3. Move `previous` to the current node.
4. Move `current` to the saved next node.

When `current` becomes `None`, `previous` is the new head.

### Complexity

- Time: `O(n)`
- Auxiliary space: `O(1)`

**Critical mistake:** Changing `current.next` before saving its original value can disconnect the remaining list.

---

## 5. Reverse a Linked List Recursively

Recursion is an alternative to the iterative approach.

```python
def reverse_list(head):
    if head is None or head.next is None:
        return head

    new_head = reverse_list(head.next)

    head.next.next = head
    head.next = None

    return new_head
```

### How it works

Suppose the list is:

`1 → 2 → 3 → None`

The recursive calls reach node `3`. As the calls return:

- Node `3` becomes the new head.
- Node `2` is linked backward from node `3`.
- Node `1` is linked backward from node `2`.
- The original forward links are removed as needed.

### Complexity

- Time: `O(n)`
- Auxiliary space: `O(n)` due to the recursion call stack.

**Interview takeaway:** The iterative solution is usually preferable when constant auxiliary space is required.

---

## 6. Reverse a Sublist of a Linked List

**Problem:** Reverse the nodes between positions `left` and `right`, using 1-based positions.

For example:

`1 → 2 → 3 → 4 → 5`

With `left = 2` and `right = 4`, the result is:

`1 → 4 → 3 → 2 → 5`

### Solution

```python
def reverse_between(head, left, right):
    if head is None or left == right:
        return head

    dummy = ListNode(0)
    dummy.next = head
    before = dummy

    # Move to the node immediately before the sublist.
    for _ in range(left - 1):
        before = before.next

    current = before.next

    # Move each next node to the front of the sublist.
    for _ in range(right - left):
        moving = current.next
        current.next = moving.next
        moving.next = before.next
        before.next = moving

    return dummy.next
```

This code assumes a `ListNode` class is defined with `val` and `next` attributes and that `left` and `right` are valid positions.

### Key idea

Instead of reversing the entire list, repeatedly move the node immediately after `current` to the beginning of the selected sublist.

The dummy node makes the same logic work when the reversal starts at the head.

### Complexity

- Time: `O(n)` in the worst case.
- Auxiliary space: `O(1)`.

**Common mistake:** Forgetting that positions are 1-based, or failing to reconnect the reversed section to the nodes before and after it.

---

## 7. Reverse Nodes in Groups of K

**Problem:** Reverse the linked list in groups of `k` nodes. If the final group has fewer than `k` nodes, leave it unchanged.

Example:

`1 → 2 → 3 → 4 → 5`, `k = 2`

Result:

`2 → 1 → 4 → 3 → 5`

### Solution

```python
def reverse_k_group(head, k):
    if k <= 1 or head is None:
        return head

    dummy = ListNode(0)
    dummy.next = head
    group_prev = dummy

    while True:
        # Find the kth node in the current group.
        kth = group_prev

        for _ in range(k):
            kth = kth.next
            if kth is None:
                return dummy.next

        group_next = kth.next

        # Reverse the current group.
        previous = group_next
        current = group_prev.next

        while current is not group_next:
            next_node = current.next
            current.next = previous
            previous = current
            current = next_node

        # Reconnect the reversed group.
        old_group_head = group_prev.next
        group_prev.next = kth
        group_prev = old_group_head
```

This implementation assumes a defined `ListNode` class.

### Complexity

- Time: `O(n)`
- Auxiliary space: `O(1)`

**Key detail:** `group_next` marks the first node after the group. Initializing `previous` to `group_next` reconnects the reversed group to the remaining list.

**Common mistake:** Reversing a final incomplete group when the problem requires leaving it unchanged.

---

## 8. Reverse Words in a String

**Problem:** Reverse the order of words in a string.

Example:

`"data engineering is interesting"`

Result:

`"interesting is engineering data"`

### Solution

```python
def reverse_words(s):
    return " ".join(s.split()[::-1])
```

### How it works

1. `split()` separates the words and removes extra whitespace between them.
2. `[::-1]` reverses the word list.
3. `" ".join(...)` joins the words with a single space.

### Complexity

- Time: `O(n)`
- Extra space: `O(n)`

**Important:** This solution normalizes whitespace. If the problem requires preserving the exact spaces, a different approach is needed.

---

## 9. Rotate an Array Using Reversal

**Problem:** Rotate an array to the right by `k` positions.

Example:

`[1, 2, 3, 4, 5, 6, 7]`, `k = 3`

Result:

`[5, 6, 7, 1, 2, 3, 4]`

### Solution

```python
def rotate_right(nums, k):
    n = len(nums)

    if n == 0:
        return nums

    k %= n

    def reverse(left, right):
        while left < right:
            nums[left], nums[right] = nums[right], nums[left]
            left += 1
            right -= 1

    reverse(0, n - 1)
    reverse(0, k - 1)
    reverse(k, n - 1)

    return nums
```

### Why it works

For a right rotation by `k`:

1. Reverse the entire array.
2. Reverse the first `k` elements.
3. Reverse the remaining elements.

For the example:

- Original: `[1, 2, 3, 4, 5, 6, 7]`
- Reverse all: `[7, 6, 5, 4, 3, 2, 1]`
- Reverse first three: `[5, 6, 7, 4, 3, 2, 1]`
- Reverse the rest: `[5, 6, 7, 1, 2, 3, 4]`

### Complexity

- Time: `O(n)`
- Auxiliary space: `O(1)`

**Common mistake:** Forgetting `k %= n`. Rotating by more than the array length is equivalent to rotating by the remainder.

---

## 10. Reverse Only Part of an Array

**Problem:** Reverse the elements between indices `left` and `right`, inclusive.

```python
def reverse_range(nums, left, right):
    while left < right:
        nums[left], nums[right] = nums[right], nums[left]
        left += 1
        right -= 1

    return nums
```

Example:

```python
nums = [1, 2, 3, 4, 5]
print(reverse_range(nums, 1, 3))
# [1, 4, 3, 2, 5]
```

- Time: `O(right - left + 1)`
- Auxiliary space: `O(1)`

This helper is useful for array rotation and other problems involving reversal of selected portions.

---

## 11. Reversal vs. Other Patterns

| Pattern | Main idea | Typical use |
|---|---|---|
| Reversal | Change the order of elements or links | Reverse arrays or linked lists |
| Two pointers | Coordinate two positions | Pair search, palindrome checks |
| Fast and slow pointers | Move pointers at different speeds | Cycle detection, list midpoint |
| Sliding window | Maintain a contiguous range | Longest substring, range sums |
| Stack | Process the most recent item first | Bracket matching, expression processing |

Reversal often uses two pointers, but the patterns are not identical. Reversal describes the operation, while two pointers describe one way to implement it.

---

## 12. Common Mistakes

1. **Losing the remaining linked list:** Save `current.next` before changing a link.
2. **Returning the old head:** After a full linked-list reversal, return `previous`.
3. **Using extra space unnecessarily:** For a mutable array, swapping from both ends can achieve `O(1)` auxiliary space.
4. **Ignoring empty input:** Handle an empty list or array before accessing elements.
5. **Incorrect boundaries:** Be clear about whether positions are 0-based or 1-based and whether the right endpoint is included.
6. **Incorrect rotation direction:** Left rotation and right rotation use different reversal ranges.
7. **Forgetting incomplete groups:** Follow the problem's requirements for the final group in group-reversal problems.
8. **Ignoring side effects:** Array and linked-list reversal usually modify the original data structure.

---

## 13. Practice Problems

### Easy

- [ ] Reverse an array in place.
- [ ] Reverse a string.
- [ ] Reverse a singly linked list iteratively.
- [ ] Reverse the order of words in a string.

### Medium

- [ ] Reverse a linked list recursively.
- [ ] Reverse a linked-list sublist between two positions.
- [ ] Rotate an array to the right by `k` positions.
- [ ] Reverse nodes in groups of `k`.
- [ ] Check whether a linked list is a palindrome.

### Interview questions

1. Why is reversal of a mutable array possible in `O(1)` auxiliary space?
2. Why must a linked-list reversal save the next node before changing a link?
3. What is the difference between iterative and recursive linked-list reversal?
4. How does the three-reversal technique rotate an array?
5. Why is a dummy node useful when reversing a linked-list sublist?
6. What is the time complexity of reversing `n` elements?
7. How would you reverse only the first `k` nodes of a linked list?

---

## 14. Quick Revision

- **Array reversal:** Swap elements from both ends and move toward the center.
- **String reversal:** Use slicing or convert the string into a list of characters.
- **Linked-list reversal:** Track `previous`, `current`, and `next_node`.
- **Sublist reversal:** Reverse only the required section and reconnect both ends.
- **Group reversal:** Find a complete group, reverse it, and reconnect it.
- **Array rotation:** Reverse the whole array, then reverse the two sections.
- **Complexity:** Most reversal operations take `O(n)` time; in-place array reversal and iterative linked-list reversal use `O(1)` auxiliary space.

**Final principle:** In reversal problems, correctness depends on the order of operations and the handling of boundaries. Understand which references must be preserved before changing the structure.