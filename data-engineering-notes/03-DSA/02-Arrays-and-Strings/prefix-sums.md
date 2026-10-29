# Prefix Sums — DSA Interview Notes

## 1. What Is a Prefix Sum?

A prefix sum is a technique that stores cumulative sums of an array so that range-sum queries can be answered efficiently.

Instead of calculating the sum of a range from scratch every time, we calculate cumulative sums once and reuse them.

Example:

```python
nums = [2, 4, 1, 7, 3]
```

The prefix sums, including an initial zero, are:

```python
prefix = [0, 2, 6, 7, 14, 17]
```

Each prefix entry represents the sum of all elements before that index.

- `prefix[0] = 0`
- `prefix[1] = 2`
- `prefix[2] = 2 + 4 = 6`
- `prefix[3] = 2 + 4 + 1 = 7`

The initial zero makes range calculations simpler.

## 2. Why Use Prefix Sums?

Suppose you need to calculate the sum of elements from index `1` through index `3`:

```python
nums = [2, 4, 1, 7, 3]
```

The answer is:

\[
4 + 1 + 7 = 12
\]

A direct calculation takes O(n) time per query in the worst case. With prefix sums, the answer is calculated using subtraction:

\[
\text{range sum}(L,R)=prefix[R+1]-prefix[L]
\]

For this example:

\[
prefix[4]-prefix[1]=14-2=12
\]

Each range-sum query now takes O(1) time after preprocessing.

## 3. Building a Prefix Sum Array

### Python implementation

```python
def build_prefix(nums):
    prefix = [0]

    for num in nums:
        prefix.append(prefix[-1] + num)

    return prefix
```

Example:

```python
nums = [2, 4, 1, 7, 3]

prefix = build_prefix(nums)

print(prefix)
# [0, 2, 6, 7, 14, 17]
```

### How it works

1. Start with `[0]`.
2. Add the next element to the previous prefix sum.
3. Store the new cumulative total.
4. Repeat until every element has been processed.

The relationship is:

\[
prefix[i+1]=prefix[i]+nums[i]
\]

**Complexity:**

- Time: O(n)
- Extra space: O(n)

## 4. Range-Sum Queries

Once the prefix array is built, any valid inclusive range can be queried in constant time.

```python
def range_sum(prefix, left, right):
    return prefix[right + 1] - prefix[left]
```

Example:

```python
nums = [2, 4, 1, 7, 3]
prefix = build_prefix(nums)

print(range_sum(prefix, 0, 2))  # 7
print(range_sum(prefix, 1, 3))  # 12
print(range_sum(prefix, 2, 4))  # 11
```

The function assumes that `left` and `right` are valid indices and that `left <= right`.

### Why subtract the earlier prefix?

`prefix[right + 1]` includes every element from index `0` through `right`.

`prefix[left]` includes every element before index `left`.

Subtracting removes the unwanted earlier elements, leaving only the requested range.

## 5. Prefix Sum vs. Repeated Calculation

Suppose an array has `n` elements and you need to answer `q` range-sum queries.

| Approach | Total time |
|---|---|
| Calculate every range directly | O(qn) worst case |
| Build prefix sums, then query | O(n + q) |

Prefix sums are especially useful when the same array receives many queries but does not change between them.

If the array changes frequently, a prefix array may need updating or rebuilding. For frequent point updates and range queries, a Fenwick tree or segment tree may be more appropriate.

## 6. Pattern: Prefix Sum + Hash Map

Prefix sums become even more powerful when combined with a dictionary.

### Problem: Count subarrays whose sum equals `k`

Given an array `nums` and an integer `k`, return the number of contiguous subarrays whose sum equals `k`.

Example:

```python
nums = [1, 1, 1]
k = 2
```

The qualifying subarrays are:

- `nums[0:2] = [1, 1]`
- `nums[1:3] = [1, 1]`

Answer: `2`.

### Solution

```python
def subarray_sum(nums, k):
    prefix_count = {0: 1}
    prefix_sum = 0
    count = 0

    for num in nums:
        prefix_sum += num

        needed = prefix_sum - k
        count += prefix_count.get(needed, 0)

        prefix_count[prefix_sum] = (
            prefix_count.get(prefix_sum, 0) + 1
        )

    return count
```

Example:

```python
print(subarray_sum([1, 1, 1], 2))    # 2
print(subarray_sum([1, -1, 0], 0))   # 3
print(subarray_sum([1, 2, 3], 3))    # 2
```

### How it works

Let the current prefix sum be \(S\).

If an earlier prefix sum was \(S-k\), the sum of the elements between those prefix positions must be \(k\).

Therefore:

\[
S-(S-k)=k
\]

The dictionary `prefix_count` stores how many times each prefix sum has appeared.

The initial entry `{0: 1}` allows a subarray starting at index `0` to be counted.

**Important:** Check for the required prefix sum before adding the current prefix sum to the dictionary. This avoids counting an empty subarray when `k = 0`.

**Complexity:**

- Expected time: O(n)
- Extra space: O(n)

This method works even when the array contains negative numbers.

## 7. Pattern: Prefix Sum for Equilibrium Index

An equilibrium index is an index where the sum of elements on the left equals the sum of elements on the right.

Example:

```python
nums = [1, 7, 3, 6, 5, 6]
```

At index `3`, the left sum is:

\[
1+7+3=11
\]

The right sum is:

\[
5+6=11
\]

Therefore, index `3` is an equilibrium index.

### Solution

```python
def equilibrium_index(nums):
    total = sum(nums)
    left_sum = 0

    for i, num in enumerate(nums):
        right_sum = total - left_sum - num

        if left_sum == right_sum:
            return i

        left_sum += num

    return -1
```

Example:

```python
print(equilibrium_index([1, 7, 3, 6, 5, 6]))
# 3
```

The algorithm keeps a running left sum and calculates the right sum using the total.

**Complexity:**

- Time: O(n)
- Extra space: O(1)

This is a useful optimization: you do not always need to store the entire prefix array when a running total is enough.

## 8. Pattern: Prefix Sum for Divisibility

Prefix sums can help identify subarrays whose sum is divisible by a given number.

### Problem: Count subarrays whose sum is divisible by `k`

The key observation is that if two prefix sums have the same remainder when divided by `k`, their difference is divisible by `k`.

For example:

\[
14 \bmod 5=4
\]

\[
9 \bmod 5=4
\]

Their difference is \(14-9=5\), which is divisible by `5`.

### Solution

```python
def subarrays_div_by_k(nums, k):
    remainder_count = {0: 1}
    prefix_sum = 0
    count = 0

    for num in nums:
        prefix_sum += num
        remainder = prefix_sum % k

        count += remainder_count.get(remainder, 0)

        remainder_count[remainder] = (
            remainder_count.get(remainder, 0) + 1
        )

    return count
```

Example:

```python
print(subarrays_div_by_k([4, 5, 0, -2, -3, 1], 5))
# 7
```

Python's modulo operation produces a nonnegative remainder for a positive divisor, including when the input sum is negative.

**Complexity:**

- Expected time: O(n)
- Extra space: O(min(n, k)) for positive `k`, at most O(n)

The divisor `k` must be nonzero.

## 9. Pattern: Two-Dimensional Prefix Sums

Prefix sums can also answer queries about rectangular regions in a matrix.

Consider this matrix:

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]
```

A two-dimensional prefix sum stores the cumulative total of the rectangle from the top-left corner through each cell.

Using a padded prefix matrix simplifies the indexing.

### Building the prefix matrix

```python
def build_2d_prefix(matrix):
    rows = len(matrix)
    cols = len(matrix[0]) if rows else 0

    prefix = [[0] * (cols + 1) for _ in range(rows + 1)]

    for r in range(rows):
        for c in range(cols):
            prefix[r + 1][c + 1] = (
                matrix[r][c]
                + prefix[r][c + 1]
                + prefix[r + 1][c]
                - prefix[r][c]
            )

    return prefix
```

### Querying a rectangle

The following function returns the sum of a rectangle whose row indices range from `r1` to `r2` and column indices range from `c1` to `c2`, with both endpoints inclusive.

```python
def rectangle_sum(prefix, r1, c1, r2, c2):
    return (
        prefix[r2 + 1][c2 + 1]
        - prefix[r1][c2 + 1]
        - prefix[r2 + 1][c1]
        + prefix[r1][c1]
    )
```

Example:

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

prefix = build_2d_prefix(matrix)

print(rectangle_sum(prefix, 1, 1, 2, 2))
# 28, because 5 + 6 + 8 + 9 = 28
```

The rectangle formula subtracts the regions above and to the left, then adds back their overlap because it was subtracted twice.

**Complexity:**

- Build: O(rows × columns) time and space.
- Each rectangle query: O(1) time.

The example assumes a rectangular matrix with consistent row lengths.

## 10. Prefix Sum vs. Sliding Window

Both techniques work with contiguous subarrays, but they solve different kinds of problems.

| Feature | Prefix sum | Sliding window |
|---|---|---|
| Main idea | Store cumulative totals | Maintain a moving range |
| Multiple range-sum queries | Excellent | Usually not ideal |
| Fixed-size window sum | Works | Often simpler |
| Target sum with negative numbers | Prefix sum + hash map works | Standard shrinking-window logic may fail |
| Space usage | Often O(n) | Often O(1) |
| Best use | Range queries and cumulative relationships | Expanding or shrinking contiguous ranges |

Choose based on the problem's requirements rather than memorizing one technique.

## 11. Common Mistakes

1. **Off-by-one errors:** With a padded prefix array, a range from `left` to `right` uses `prefix[right + 1] - prefix[left]`.
2. **Forgetting the initial zero:** `{0: 1}` is essential for counting target-sum subarrays that start at the beginning.
3. **Confusing subarrays and subsequences:** Prefix-sum subarray techniques rely on contiguous ranges.
4. **Using a sliding window with negative values without justification:** The usual shrinking logic depends on the behavior of the window sum.
5. **Updating the hash map at the wrong time:** For target-sum counting, query the earlier prefix counts before recording the current prefix.
6. **Ignoring updates to the original array:** A precomputed prefix array becomes stale if the underlying values change.
7. **Using a full prefix array unnecessarily:** A running total may be enough for one-pass tasks such as finding an equilibrium index.

## 12. Practice Problems

### Easy

1. Running Sum of 1D Array.
2. Find the equilibrium index.
3. Range Sum Query — Immutable.
4. Find the sum of a specified range.

### Medium

1. Subarray Sum Equals K.
2. Contiguous Array.
3. Subarray Sums Divisible by K.
4. Product of Array Except Self.
5. Find Pivot Index.
6. Range Sum Query 2D — Immutable.

### Recommended practice order

1. Build a one-dimensional prefix array.
2. Answer range-sum queries.
3. Solve equilibrium-index problems.
4. Combine prefix sums with a hash map.
5. Practise negative-number and divisibility problems.
6. Move to two-dimensional prefix sums.

## Quick Revision

- Prefix sum stores cumulative totals.
- Formula: `prefix[i + 1] = prefix[i] + nums[i]`.
- Inclusive range sum: `prefix[right + 1] - prefix[left]`.
- Building the array takes O(n); each range-sum query takes O(1).
- Prefix sum + hash map can count target-sum subarrays in O(n) expected time.
- Equal prefix-sum remainders identify subarrays divisible by a divisor.
- Two-dimensional prefix sums answer rectangle-sum queries in O(1).
- Always check index boundaries, negative-number behavior, and whether the input can change.

**Interview goal:** Be able to explain the cumulative-sum idea, derive the range formula, and recognize when a hash map should be combined with prefix sums.