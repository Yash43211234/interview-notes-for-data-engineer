# Fast and Slow Pointers

## 1. Overview

Fast and Slow Pointers is a coding pattern in which two pointers move through a sequence at different speeds.

- **Slow pointer:** Usually moves one step at a time.
- **Fast pointer:** Usually moves two steps at a time.

This pattern is especially useful for linked lists, but it can also solve problems involving cycles and repeated state transitions.

The main idea is to use the difference in pointer speeds to reveal information about the structure without storing every visited element.

## 2. When to Use This Pattern

Look for Fast and Slow Pointers when a problem asks you to:

- Detect whether a linked list contains a cycle.
- Find the node where a cycle begins.
- Find the middle node of a linked list.
- Determine whether a linked list is a palindrome.
- Find a cycle in a repeated-state sequence.
- Find the duplicate number under specific array constraints.

## 3. Basic Implementation

Assume each linked-list node has a `val` attribute and a `next` pointer.

```python
slow = head
fast = head

while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
```

At each iteration:

- `slow` moves forward by one node.
- `fast` moves forward by two nodes.
- The loop stops when `fast` reaches the end or cannot move two steps.

This basic loop is not a complete solution by itself. The condition and return value depend on the problem.

---

## 4. Find the Middle of a Linked List

**Problem:** Return the middle node of a singly linked list. If the list has an even number of nodes, return the second middle node.

### Solution

```python
def middle_node(head):
    slow = head
    fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    return slow
```

### Example

For the list:

`1 → 2 → 3 → 4 → 5`

The pointers progress as follows:

| Iteration | Slow | Fast |
|---|---:|---:|
| Start | 1 | 1 |
| 1 | 2 | 3 |
| 2 | 3 | 5 |

Fast cannot move another two steps, so the loop ends. Slow points to node `3`.

For an even-length list:

`1 → 2 → 3 → 4 → 5 → 6`

Slow finishes at node `4`, the second middle node.

**Complexity**

- Time: `O(n)`
- Auxiliary space: `O(1)`

**Why it works:** Fast travels twice as far as slow. When fast reaches the end, slow has traveled approximately half the list.

---

## 5. Detect a Cycle in a Linked List

**Problem:** Determine whether a linked list contains a cycle.

A cycle occurs when a node's `next` pointer leads back to a previously visited node instead of eventually reaching `None`.

### Solution

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

### Example

Consider this list:

`1 → 2 → 3 → 4 → 5`

Suppose node `5` points back to node `3`.

The fast pointer moves through the cycle more quickly than the slow pointer. Eventually, both pointers meet at the same node.

If the list has no cycle, fast eventually reaches `None`.

**Complexity**

- Time: `O(n)` for the standard cycle-detection analysis.
- Auxiliary space: `O(1)`.

**Important:** Use `slow is fast` to compare node identity, not merely node values. Two different nodes may contain the same value.

---

## 6. Find the Beginning of a Cycle

**Problem:** Return the node where a linked-list cycle begins, or `None` if no cycle exists.

This uses Floyd's cycle-detection algorithm in two phases.

### Solution

```python
def detect_cycle(head):
    slow = head
    fast = head

    # Phase 1: Find a meeting point inside the cycle.
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

        if slow is fast:
            break
    else:
        return None

    # Phase 2: Find the start of the cycle.
    slow = head

    while slow is not fast:
        slow = slow.next
        fast = fast.next

    return slow
```

### How it works

1. Move slow by one step and fast by two steps.
2. If they meet, a cycle exists.
3. Reset slow to the head.
4. Move both pointers one step at a time.
5. Their next meeting point is the cycle's starting node.

The second phase works because of the mathematical relationship between the distance to the cycle and the distance traveled within it.

**Complexity**

- Time: `O(n)`
- Auxiliary space: `O(1)`

**Common mistake:** Resetting one pointer to the head but continuing to move the other pointer two steps at a time. In the standard algorithm's second phase, both move one step at a time.

---

## 7. Check Whether a Linked List Is a Palindrome

**Problem:** Determine whether a linked list reads the same forward and backward.

Example:

`1 → 2 → 2 → 1`

This list is a palindrome.

### Solution

```python
def is_palindrome(head):
    if not head or not head.next:
        return True

    # Find the end of the first half.
    slow = fast = head

    while fast.next and fast.next.next:
        slow = slow.next
        fast = fast.next.next

    # Reverse the second half.
    second = slow.next
    previous = None

    while second:
        next_node = second.next
        second.next = previous
        previous = second
        second = next_node

    # Compare both halves.
    first = head
    second = previous

    while second:
        if first.val != second.val:
            return False

        first = first.next
        second = second.next

    return True
```

### Approach

1. Find the midpoint using fast and slow pointers.
2. Reverse the second half of the list.
3. Compare corresponding values from both halves.
4. Return whether all compared values match.

**Complexity**

- Time: `O(n)`
- Auxiliary space: `O(1)`.

**Important:** This implementation modifies the links in the second half. If the original list must be preserved, restore the reversed half before returning, including when a mismatch is found.

---

## 8. Find the Duplicate Number

**Problem:** An array contains `n + 1` integers, each between `1` and `n`, inclusive. At least one value is duplicated. Find a duplicate without modifying the array and using constant extra space.

Treat each array value as the next index to visit.

### Solution

```python
def find_duplicate(nums):
    slow = nums[0]
    fast = nums[0]

    # Phase 1: Find a meeting point.
    while True:
        slow = nums[slow]
        fast = nums[nums[fast]]

        if slow == fast:
            break

    # Phase 2: Find the cycle entrance.
    slow = nums[0]

    while slow != fast:
        slow = nums[slow]
        fast = nums[fast]

    return slow
```

### Why this works

The array defines a sequence of index transitions:

`index → nums[index] → nums[nums[index]] → ...`

Because every value is a valid index and there are more positions than possible distinct values, the sequence eventually enters a cycle.

The cycle's entrance corresponds to a duplicated value.

**Complexity**

- Time: `O(n)`
- Auxiliary space: `O(1)`.

**Important constraints:** This approach relies on the specified value range and duplicate-number problem structure. It is not a general method for finding duplicates in any arbitrary array.

---

## 9. Happy Number

**Problem:** A positive integer is called happy if repeatedly replacing it with the sum of the squares of its digits eventually reaches `1`.

If the sequence enters a cycle that does not include `1`, the number is not happy.

### Solution

```python
def is_happy(n):
    def next_number(number):
        total = 0

        while number > 0:
            digit = number % 10
            total += digit * digit
            number //= 10

        return total

    slow = n
    fast = n

    while True:
        slow = next_number(slow)
        fast = next_number(next_number(fast))

        if fast == 1:
            return True

        if slow == fast:
            return False
```

### Example

For `19`:

`19 → 82 → 68 → 100 → 1`

The sequence reaches `1`, so `19` is a happy number.

**Pattern:** Fast and Slow Pointers on a repeated-state sequence.

**Complexity:** The number of iterations depends on the sequence. Each transformation takes time proportional to the number of digits in its input.

---

## 10. Fast and Slow Pointers vs. Other Patterns

| Pattern | Best suited for |
|---|---|
| Fast and slow pointers | Linked-list cycles and midpoints |
| Two pointers | Pair comparisons, sorted arrays, in-place movement |
| Sliding window | Contiguous ranges in arrays and strings |
| Hashing | Fast membership checks and tracking visited values |
| Binary search | Sorted data or monotonic search spaces |

Fast and slow pointers can detect cycles without a visited set, which is useful when constant extra space is required.

## 11. Common Mistakes

1. **Incorrect loop condition**

   For moving fast by two nodes, use:

   ```python
   while fast and fast.next:
   ```

   This prevents accessing `fast.next.next` when `fast` or `fast.next` is `None`.

2. **Comparing node values instead of node identity**

   Two distinct nodes can contain equal values. Cycle detection must compare the nodes themselves.

3. **Using the wrong starting positions**

   Starting both pointers at `head` is common for cycle detection. Finding the middle also typically starts both at `head`, but other problems may require different initialization.

4. **Forgetting even-length behavior**

   The standard middle-node loop returns the second middle node for an even-length list. Use a different condition if the first middle node is required.

5. **Modifying a linked list unintentionally**

   Reversing the second half of a palindrome-checking list changes its links. Restore the list if preservation is required.

6. **Applying the pattern without a valid transition structure**

   The pattern works when pointer movement has meaningful, repeatable transitions. It is not automatically appropriate for every array or linked-list problem.

---

## 12. Practice Problems

Solve these in order, focusing on the pointer movements rather than memorizing the code.

### Easy

- [ ] Find the middle of a linked list.
- [ ] Detect whether a linked list contains a cycle.

### Medium

- [ ] Find the beginning of a linked-list cycle.
- [ ] Check whether a linked list is a palindrome.
- [ ] Find the duplicate number using constant extra space.
- [ ] Determine whether a number is happy.

### Interview questions

1. Why does the fast pointer move two steps while the slow pointer moves one?
2. How can a cycle be detected without storing visited nodes?
3. Why must cycle detection compare node identity rather than node values?
4. Why do the two pointers meet inside a cycle?
5. Why does resetting one pointer to the head help find the cycle entrance?
6. What is the time and auxiliary-space complexity of Floyd's cycle-detection algorithm?
7. How would you find the first middle node instead of the second middle node?

## 13. Quick Revision

- **Middle node:** Fast moves twice as quickly as slow.
- **Cycle detection:** If fast and slow meet, a cycle exists.
- **Cycle entrance:** After the first meeting, reset one pointer to the head and move both one step at a time.
- **Palindrome list:** Find the midpoint, reverse the second half, and compare both halves.
- **Duplicate number:** Treat array values as pointers when the problem's constraints permit it.
- **Core advantage:** Many of these problems can be solved in `O(n)` time and `O(1)` auxiliary space.

**Remember:** Fast and Slow Pointers is not about moving two variables randomly. It uses different movement speeds to reveal the structure of a sequence.