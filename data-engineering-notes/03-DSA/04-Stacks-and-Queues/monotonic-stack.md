# Monotonic Stack

## 1. Overview

A **monotonic stack** is a stack that maintains elements in a specific increasing or decreasing order.

It is useful when a problem asks you to find the next or previous greater or smaller element, or to calculate how long something must wait until a condition is satisfied.

Instead of comparing every element with every other element, a monotonic stack allows many problems to be solved in **O(n) time**.

Common applications include:

- Next greater element.
- Next smaller element.
- Previous greater or smaller element.
- Daily temperatures.
- Stock span.
- Largest rectangle in a histogram.
- Sum of subarray minimums.

## 2. Types of Monotonic Stacks

There are two primary types.

### Monotonic increasing stack

Values are maintained in increasing order from the bottom to the top.

Example:

`[2, 4, 6, 8]`

When a new value `5` arrives, values larger than `5` are removed from the top until the ordering can be maintained.

The resulting stack is:

`[2, 4, 5]`

Use this pattern when larger values on top must be removed to find a smaller value that satisfies the problem's condition.

### Monotonic decreasing stack

Values are maintained in decreasing order from the bottom to the top.

Example:

`[9, 7, 5, 3]`

When a new value `6` arrives, values smaller than `6` are removed from the top.

The resulting stack is:

`[9, 7, 6]`

Use this pattern when a new, larger value resolves the waiting elements with smaller values.

**Important:** These names describe the order from the bottom of the stack to the top. Always check the actual `while` condition in your code because some explanations use the opposite naming convention.

## 3. The Basic Algorithm

A monotonic stack typically follows these steps:

1. Traverse the array from left to right or right to left.
2. Compare the current value with the stack's top.
3. Pop elements while they violate the required ordering or their waiting condition is satisfied.
4. Use the popped elements to calculate the answer.
5. Push the current element or its index.

In most interview problems, **store indices rather than values**. Indices allow you to calculate distances and retrieve the corresponding array values.

Basic template:

```python
def monotonic_stack_template(nums):
    stack = []

    for i, num in enumerate(nums):
        while stack and nums[stack[-1]] < num:
            index = stack.pop()

            # nums[index] has found a greater element: num
            # Calculate or store the answer here.

        stack.append(i)

    return stack
```

This template maintains a decreasing stack of values from bottom to top and finds the next greater element for each popped index.

The exact comparison operator depends on whether the problem asks for a strictly greater element or a greater-than-or-equal element.

---

## 4. Next Greater Element

**Problem:** For every array element, find the first element to its right that is strictly greater. If no such element exists, return `-1`.

Example:

Input: `[2, 1, 2, 4, 3]`

Output: `[4, 2, 4, -1, -1]`

### Solution

```python
def next_greater_element(nums):
    n = len(nums)
    result = [-1] * n
    stack = []

    for i, num in enumerate(nums):
        while stack and nums[stack[-1]] < num:
            index = stack.pop()
            result[index] = num

        stack.append(i)

    return result
```

### How it works

When the current element is greater than the value at the top index, it becomes the next greater element for that index.

For `[2, 1, 2, 4, 3]`:

- The first `2` waits in the stack.
- The `1` is also added because it is smaller than `2`.
- The next `2` resolves the waiting `1`.
- The `4` resolves the remaining waiting `2` values.
- The final `3` cannot resolve anything remaining.

### Complexity

- Time: `O(n)`
- Auxiliary space: `O(n)`

**Why is the time complexity O(n)?**

Each index is pushed onto the stack once and popped at most once. Although there is a nested `while` loop, the total number of stack operations is linear.

---

## 5. Next Smaller Element

**Problem:** For every element, find the first strictly smaller element to its right. Return `-1` when no such element exists.

Example:

Input: `[4, 8, 5, 2, 25]`

Output: `[2, 5, 2, -1, -1]`

### Solution

```python
def next_smaller_element(nums):
    n = len(nums)
    result = [-1] * n
    stack = []

    for i, num in enumerate(nums):
        while stack and nums[stack[-1]] > num:
            index = stack.pop()
            result[index] = num

        stack.append(i)

    return result
```

### Key difference

For next greater, pop while:

```python
nums[stack[-1]] < num
```

For next smaller, pop while:

```python
nums[stack[-1]] > num
```

**Pattern:** The current element resolves previous elements that have been waiting for a qualifying value.

---

## 6. Previous Greater Element

**Problem:** For every element, find the nearest strictly greater element to its left.

Example:

Input: `[4, 5, 2, 10, 8]`

Output: `[-1, -1, 5, -1, 10]`

### Solution

```python
def previous_greater_element(nums):
    result = []
    stack = []

    for num in nums:
        while stack and stack[-1] <= num:
            stack.pop()

        result.append(stack[-1] if stack else -1)
        stack.append(num)

    return result
```

### How it works

Before answering for the current element, remove all values that are less than or equal to it.

Those values cannot be the previous strictly greater element for the current value. They also cannot be useful candidates for future elements because the current value is at least as large and is closer.

The remaining top, if any, is the nearest strictly greater value to the left.

### Complexity

- Time: `O(n)`
- Auxiliary space: `O(n)`

This implementation stores values because it only needs to return the previous greater value. Store indices instead if positions or distances are required.

---

## 7. Daily Temperatures

**Problem:** Given daily temperatures, calculate how many days each day must wait for a warmer temperature. If there is no warmer day, return `0`.

Example:

Input: `[73, 74, 75, 71, 69, 72, 76, 73]`

Output: `[1, 1, 4, 2, 1, 1, 0, 0]`

### Solution

```python
def daily_temperatures(temperatures):
    n = len(temperatures)
    result = [0] * n
    stack = []

    for i, temperature in enumerate(temperatures):
        while (
            stack
            and temperatures[stack[-1]] < temperature
        ):
            previous_day = stack.pop()
            result[previous_day] = i - previous_day

        stack.append(i)

    return result
```

### Explanation

The stack stores indices of days that have not yet found a warmer day.

When a warmer temperature appears:

1. Pop the previous day.
2. Calculate the distance between the current day and the previous day.
3. Store that distance in the result.

The stack contains unresolved days, not necessarily the most recent days in chronological order at all times.

**Complexity:** `O(n)` time and `O(n)` auxiliary space.

**Interview insight:** This is the next greater element pattern with an answer based on index differences rather than the temperature itself.

---

## 8. Stock Span Problem

**Problem:** For each day's price, calculate the number of consecutive days ending today for which every price was less than or equal to today's price.

Example:

Input: `[100, 80, 60, 70, 60, 75, 85]`

Output: `[1, 1, 1, 2, 1, 4, 6]`

### Solution

```python
def stock_span(prices):
    spans = []
    stack = []

    for i, price in enumerate(prices):
        while stack and prices[stack[-1]] <= price:
            stack.pop()

        previous_greater = stack[-1] if stack else -1
        spans.append(i - previous_greater)
        stack.append(i)

    return spans
```

### How it works

The stack maintains indices of prices in strictly decreasing order from bottom to top.

For each price:

1. Remove previous prices less than or equal to the current price.
2. Find the nearest previous greater price.
3. Calculate the span using the distance from that index.
4. Push the current index.

If no previous greater price exists, the span extends to the beginning of the array.

### Complexity

- Time: `O(n)`
- Auxiliary space: `O(n)`

**Important:** Use `<=` because previous prices equal to today's price are included in the span.

---

## 9. Largest Rectangle in a Histogram

**Problem:** Given bar heights in a histogram, find the largest rectangular area that can fit within the bars.

Example:

Input: `[2, 1, 5, 6, 2, 3]`

Output: `10`

The largest rectangle uses heights `5` and `6`, giving area `5 × 2 = 10`.

### Solution

```python
def largest_rectangle_area(heights):
    stack = []
    max_area = 0

    for i, height in enumerate(heights):
        while stack and heights[stack[-1]] > height:
            popped = stack.pop()
            h = heights[popped]

            left_smaller = stack[-1] if stack else -1
            width = i - left_smaller - 1

            max_area = max(max_area, h * width)

        stack.append(i)

    # Process bars that extend to the end.
    n = len(heights)

    while stack:
        popped = stack.pop()
        h = heights[popped]

        left_smaller = stack[-1] if stack else -1
        width = n - left_smaller - 1

        max_area = max(max_area, h * width)

    return max_area
```

### How it works

The stack holds indices of bars with nondecreasing heights.

When a shorter bar appears, taller bars can no longer extend past the current position. Each popped bar is therefore evaluated as the height of a rectangle.

For a popped bar:

- `h` is the rectangle height.
- The current index is the first smaller bar to the right.
- The remaining stack top identifies the nearest smaller bar to the left.
- `width = right - left - 1`.

The final loop handles bars that never encountered a shorter bar before the array ended.

### Complexity

- Time: `O(n)`
- Auxiliary space: `O(n)`

**Common mistake:** Calculating width without accounting for the popped bar's nearest smaller boundary on the left.

---

## 10. Sum of Subarray Minimums

**Problem:** Find the sum of the minimum element of every nonempty contiguous subarray.

Example:

Input: `[3, 1, 2, 4]`

Output: `17`

The subarray minimums are:

- `[3]` → `3`
- `[1]` → `1`
- `[2]` → `2`
- `[4]` → `4`
- `[3, 1]` → `1`
- `[1, 2]` → `1`
- `[2, 4]` → `2`
- `[3, 1, 2]` → `1`
- `[1, 2, 4]` → `1`
- `[3, 1, 2, 4]` → `1`

Their sum is `17`.

### Solution

```python
def sum_subarray_mins(arr):
    mod = 10**9 + 7
    stack = []
    total = 0

    for i in range(len(arr) + 1):
        current = arr[i] if i < len(arr) else float("-inf")

        while stack and arr[stack[-1]] > current:
            mid = stack.pop()

            left = stack[-1] if stack else -1
            left_count = mid - left
            right_count = i - mid

            total += arr[mid] * left_count * right_count

        stack.append(i)

    return total % mod
```

### How it works

Instead of generating every subarray, count how many subarrays use each element as their minimum.

For an element at index `mid`:

- `left_count` is the number of possible starting positions for which it remains the selected minimum.
- `right_count` is the number of possible ending positions for which it remains the selected minimum.

Its contribution is:

`arr[mid] × left_count × right_count`

Summing these contributions produces the answer.

The loop uses a virtual negative-infinity value at the end to flush the stack. The sentinel index is not dereferenced because it is only used as the current boundary.

### Complexity

- Time: `O(n)`
- Auxiliary space: `O(n)`

**Important:** When duplicates exist, the comparison rules must assign each subarray to exactly one occurrence of its minimum. This implementation uses a strict `>` condition and a negative-infinity sentinel, which gives a consistent tie-handling rule.

---

## 11. Circular Next Greater Element

**Problem:** Find the next greater element for each array position, treating the array as circular.

Example:

Input: `[1, 2, 1]`

Output: `[2, -1, 2]`

### Solution

```python
def next_greater_circular(nums):
    n = len(nums)
    result = [-1] * n
    stack = []

    for i in range(2 * n):
        index = i % n

        while stack and nums[stack[-1]] < nums[index]:
            previous = stack.pop()
            result[previous] = nums[index]

        if i < n:
            stack.append(index)

    return result
```

### How it works

The array is traversed twice using modulo arithmetic:

- The first traversal adds each index to the stack.
- The second traversal gives unresolved elements another chance to find a greater value after wrapping around to the beginning.

Only indices from the first traversal are pushed. This prevents the same index from being added repeatedly.

### Complexity

- Time: `O(n)`
- Auxiliary space: `O(n)`

**Common mistake:** Pushing indices during both traversals, which can introduce duplicates into the stack and complicate correctness.

---

## 12. Choosing the Correct Comparison

Small differences in comparison operators can change the answer.

| Requirement | Typical pop condition |
|---|---|
| Next strictly greater element | `stack_top_value < current` |
| Next greater-or-equal element | `stack_top_value <= current` |
| Next strictly smaller element | `stack_top_value > current` |
| Next smaller-or-equal element | `stack_top_value >= current` |
| Previous strictly greater element | Pop values `<= current` |
| Previous strictly smaller element | Pop values `>= current` |

These are typical conditions for the stated traversal directions. Always confirm the exact problem definition, especially when duplicate values occur.

## 13. Why Monotonic Stack Is O(n)

A monotonic-stack solution may contain a `for` loop with a nested `while` loop, but it does not usually have quadratic time complexity.

For an array of `n` elements:

- Each element is pushed onto the stack at most once.
- Each element is popped at most once.
- Every push and pop takes amortized `O(1)` time.

Therefore, the total number of stack operations is proportional to `n`.

Overall time complexity: **`O(n)`**.

This is an example of amortized analysis: one iteration may perform multiple pops, but the total work across all iterations remains linear.

## 14. Common Mistakes

1. **Storing values instead of indices:** Distances and array positions often require indices.
2. **Using the wrong comparison:** `<` and `<=` produce different behavior when duplicates exist.
3. **Processing the wrong direction:** Next-element and previous-element problems may need different traversal directions.
4. **Forgetting unresolved elements:** Some problems require a default answer such as `-1` or `0` for elements that never find a match.
5. **Assuming nested loops mean O(n²):** Analyze the total number of pushes and pops.
6. **Incorrect histogram boundaries:** The width of a rectangle depends on the nearest smaller bars on both sides.
7. **Ignoring circular behavior:** Circular problems may require two traversals.
8. **Incorrect duplicate handling:** Problems involving sums or ranges may require a consistent tie-breaking rule.

## 15. Practice Problems

### Easy

- [ ] Next Greater Element.
- [ ] Next Smaller Element.
- [ ] Previous Greater Element.
- [ ] Previous Smaller Element.

### Medium

- [ ] Daily Temperatures.
- [ ] Stock Span.
- [ ] Next Greater Element II (circular array).
- [ ] Sum of Subarray Minimums.

### Hard

- [ ] Largest Rectangle in a Histogram.
- [ ] Maximal Rectangle.
- [ ] Sum of Subarray Ranges.
- [ ] Trapping Rain Water using a monotonic stack.

## 16. Quick Revision

- A monotonic stack maintains a chosen order among unresolved elements.
- Store indices when positions or distances matter.
- Pop elements when the current element resolves their condition.
- Next greater usually pops smaller values; next smaller usually pops larger values.
- Daily Temperatures is a next-greater problem that returns distances.
- Stock Span uses previous-greater boundaries.
- Histogram problems use smaller elements as rectangle boundaries.
- Each index is generally pushed and popped at most once, giving `O(n)` time.

**Final principle:** Whenever a problem asks for the nearest greater or smaller element, or requires processing unresolved elements as soon as a boundary appears, consider a monotonic stack.