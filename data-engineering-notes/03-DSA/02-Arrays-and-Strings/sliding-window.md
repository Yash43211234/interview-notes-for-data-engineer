# Sliding Window — DSA Interview Notes

## 1. What Is the Sliding Window Technique?

Sliding window is an algorithmic technique for solving problems involving **contiguous subarrays or substrings**.

Instead of recalculating the result for every possible range, maintain a window and update it as the window moves through the input.

A window is usually defined by two pointers:

- `left`: the starting index.
- `right`: the ending index.

For example, consider:

```python
nums = [2, 1, 5, 1, 3, 2]
```

A window of size `3` moves through these ranges:

```text
[2, 1, 5], 1, 3, 2
 2, [1, 5, 1], 3, 2
 2, 1, [5, 1, 3], 2
 2, 1, 5, [1, 3, 2]
```

The key idea is to reuse information from the previous window instead of starting from scratch.

## 2. Why Use Sliding Window?

Suppose you need the maximum sum of any contiguous subarray of size `k`.

A brute-force approach calculates every window's sum separately. For `n` elements and window size `k`, this takes O(n × k) time.

Sliding window updates the sum by removing the outgoing element and adding the incoming element. This reduces the time complexity to O(n).

The technique is useful for:

- Maximum or minimum sum of a fixed-size subarray.
- Longest substring satisfying a condition.
- Shortest subarray satisfying a condition.
- Counting or finding qualifying windows.
- Problems involving consecutive elements.

There are two main types:

1. Fixed-size sliding window.
2. Variable-size sliding window.

---

## 3. Fixed-Size Sliding Window

Use a fixed-size window when the problem specifies a particular number of consecutive elements.

Common clues include:

- Maximum sum of `k` consecutive elements.
- Average of every subarray of size `k`.
- Maximum number of vowels in a substring of length `k`.

### Example: Maximum sum of a subarray of size `k`

Given an array and an integer `k`, find the maximum sum among all contiguous subarrays of size `k`.

```python
def max_window_sum(nums, k):
    if k <= 0 or k > len(nums):
        return None

    window_sum = sum(nums[:k])
    maximum = window_sum

    for right in range(k, len(nums)):
        window_sum += nums[right]
        window_sum -= nums[right - k]

        maximum = max(maximum, window_sum)

    return maximum
```

Example:

```python
print(max_window_sum([2, 1, 5, 1, 3, 2], 3))
# 9
```

The maximum-sum window is `[5, 1, 3]`, whose sum is `9`.

### How it works

Initially, calculate the sum of the first `k` elements.

For each new element:

1. Add the incoming element.
2. Subtract the element that has left the window.
3. Update the maximum sum.

The update is:

\[
new\_sum = old\_sum + incoming - outgoing
\]

**Complexity:**

- Time: O(n)
- Extra space: O(1)

The initial sum takes O(k), followed by O(n − k) updates, giving O(n) total time.

### Example: Average of every window

```python
def window_averages(nums, k):
    if k <= 0 or k > len(nums):
        return []

    window_sum = sum(nums[:k])
    averages = [window_sum / k]

    for right in range(k, len(nums)):
        window_sum += nums[right]
        window_sum -= nums[right - k]
        averages.append(window_sum / k)

    return averages
```

Example:

```python
print(window_averages([1, 3, 2, 6, -1, 4, 1, 8, 2], 5))
# [2.2, 2.8, 2.4, 3.6, 2.8]
```

This pattern is useful whenever each window has the same size.

---

## 4. Variable-Size Sliding Window

Use a variable-size window when the length changes according to a condition.

Typically:

- Expand the window by moving `right`.
- Update the window's state.
- Shrink the window by moving `left` when a condition is met or violated.

Common clues include:

- Longest substring without repeating characters.
- Minimum-length subarray with a sum at least a target.
- Longest subarray satisfying a constraint.

The exact order of expansion and shrinking depends on the problem.

### Example: Minimum-length subarray with sum at least a target

Assume all array elements are positive.

```python
def min_subarray_length(target, nums):
    left = 0
    window_sum = 0
    minimum = float("inf")

    for right, num in enumerate(nums):
        window_sum += num

        while window_sum >= target:
            minimum = min(minimum, right - left + 1)
            window_sum -= nums[left]
            left += 1

    return 0 if minimum == float("inf") else minimum
```

Example:

```python
print(min_subarray_length(7, [2, 3, 1, 2, 4, 3]))
# 2
```

The shortest qualifying subarray is `[4, 3]`.

### How it works

Suppose the current window sum reaches or exceeds the target.

1. Record the current window length.
2. Remove the leftmost element.
3. Move `left` forward.
4. Continue shrinking while the sum still meets the target.

The window is shrunk repeatedly because a smaller valid window might exist.

**Complexity:**

- Time: O(n)
- Extra space: O(1)

Although there is a nested `while` loop, each pointer moves forward at most `n` times. Therefore, the total work is linear, not O(n²).

**Important limitation:** This method relies on positive elements. With negative values, shrinking the window does not necessarily decrease its sum. For arbitrary integers, a prefix-sum-based method may be necessary.

---

## 5. Pattern: Longest Substring Without Repeating Characters

This is a classic variable-size sliding-window problem.

Given a string, return the length of its longest substring containing no repeated characters.

Example:

```text
Input:  "abcabcbb"
Output: 3
```

The longest valid substrings include `"abc"`, `"bca"`, and `"cab"`.

### Solution using a set

```python
def longest_unique_substring(s):
    left = 0
    seen = set()
    maximum = 0

    for right, char in enumerate(s):
        while char in seen:
            seen.remove(s[left])
            left += 1

        seen.add(char)
        maximum = max(maximum, right - left + 1)

    return maximum
```

Example:

```python
print(longest_unique_substring("abcabcbb"))  # 3
print(longest_unique_substring("bbbbb"))     # 1
print(longest_unique_substring("pwwkew"))     # 3
```

### How it works

The set contains the characters in the current window.

When the incoming character already exists:

1. Remove the leftmost character.
2. Move `left` forward.
3. Continue until the incoming character is no longer present.
4. Add the incoming character and update the maximum length.

The current window length is:

\[
length = right - left + 1
\]

**Complexity:**

- Time: O(n) expected.
- Extra space: O(min(n, alphabet size)).

For Unicode strings, the number of distinct characters can vary, so O(n) is a safe general space bound.

---

## 6. Pattern: Sliding Window with a Frequency Map

A frequency map is useful when the condition depends on how many times elements or characters appear.

### Example: Longest substring with at most `k` distinct characters

```python
def longest_k_distinct(s, k):
    if k <= 0:
        return 0

    left = 0
    freq = {}
    maximum = 0

    for right, char in enumerate(s):
        freq[char] = freq.get(char, 0) + 1

        while len(freq) > k:
            left_char = s[left]
            freq[left_char] -= 1

            if freq[left_char] == 0:
                del freq[left_char]

            left += 1

        maximum = max(maximum, right - left + 1)

    return maximum
```

Example:

```python
print(longest_k_distinct("eceba", 2))
# 3
```

The longest valid substring is `"ece"`.

### Key idea

The dictionary stores the frequency of each character inside the current window.

When the number of distinct characters exceeds `k`, shrink the window until the condition becomes valid again.

**Complexity:**

- Time: O(n) expected.
- Extra space: O(min(n, alphabet size)).

This pattern also applies to arrays, such as finding the longest subarray containing at most `k` distinct values.

---

## 7. Pattern: Count Windows That Satisfy a Condition

Sometimes the goal is to count valid windows rather than find the longest or shortest one.

For example, consider the number of substrings containing at most `k` distinct characters.

After expanding the right boundary and shrinking the window until it is valid, the number of valid substrings ending at `right` is:

\[
right - left + 1
\]

Why? Every substring starting at an index from `left` through `right` is valid when the window is valid and the condition is based on having at most `k` distinct characters.

### Solution

```python
def count_at_most_k_distinct(s, k):
    if k < 0:
        return 0

    left = 0
    freq = {}
    count = 0

    for right, char in enumerate(s):
        freq[char] = freq.get(char, 0) + 1

        while len(freq) > k:
            left_char = s[left]
            freq[left_char] -= 1

            if freq[left_char] == 0:
                del freq[left_char]

            left += 1

        count += right - left + 1

    return count
```

Example:

```python
print(count_at_most_k_distinct("abc", 2))
# 5
```

The qualifying substrings are `"a"`, `"b"`, `"c"`, `"ab"`, and `"bc"`.

This technique is especially useful in counting-substring interview questions.

---

## 8. Pattern: Exactly K vs. At Most K

Some problems ask for the number of subarrays or substrings with **exactly** `k` distinct elements.

A common technique is:

\[
exactly(k)=atMost(k)-atMost(k-1)
\]

This works because the set of windows with at most `k` distinct elements contains all windows with at most `k - 1` distinct elements. Subtracting removes the smaller category.

### Example: Count substrings with exactly `k` distinct characters

```python
def count_exactly_k_distinct(s, k):
    if k <= 0:
        return 0

    return (
        count_at_most_k_distinct(s, k)
        - count_at_most_k_distinct(s, k - 1)
    )
```

Example:

```python
print(count_exactly_k_distinct("pqpqs", 2))
# 7
```

This solution assumes the helper function `count_at_most_k_distinct` from the previous section is available.

**Complexity:**

- Time: O(n) expected.
- Extra space: O(min(n, alphabet size)).

The same subtraction technique is useful for counting subarrays with exactly `k` distinct integers.

---

## 9. Sliding Window vs. Two Pointers

Sliding window is a specific use of the two-pointer technique.

| Feature | Two pointers | Sliding window |
|---|---|---|
| Main idea | Maintain two indices | Maintain a contiguous range |
| Typical use | Pair sums, reversing, partitioning | Subarray and substring conditions |
| Pointers move toward each other? | Often possible | Usually move forward |
| Track a window's sum or frequency? | Not always | Usually yes |
| Requires sorted input? | Some problems do | Usually not inherently |

Use two pointers when two positions solve the problem directly. Use sliding window when you need to maintain information about a contiguous range.

---

## 10. Common Mistakes

1. **Confusing a subarray with a subsequence.** A subarray must be contiguous.
2. **Shrinking the window incorrectly.** Use `while` when the condition may remain invalid after removing one element.
3. **Using `if` instead of `while`.** This can leave an invalid window when multiple removals are necessary.
4. **Forgetting the current element.** Add the incoming element to the window's state before checking conditions when the algorithm requires it.
5. **Updating the answer at the wrong time.** For longest-valid-window problems, update after the window becomes valid.
6. **Assuming every variable window takes O(n²).** If each pointer moves forward at most `n` times, the total time can be O(n).
7. **Using standard shrinking logic with negative numbers.** The window sum may not behave predictably.
8. **Incorrectly handling empty input or invalid `k`.** Check the problem's required return value.
9. **Forgetting to remove zero-frequency keys.** When counting distinct elements, delete keys when their frequency reaches zero.

---

## 11. How to Recognize Sliding-Window Problems

Look for these clues:

- The question mentions a contiguous subarray or substring.
- It asks for the longest or shortest range satisfying a condition.
- It specifies a fixed window size `k`.
- It asks for the maximum or minimum sum over consecutive elements.
- It asks for counts of substrings satisfying a frequency constraint.

Before implementing the solution, ask:

1. Is the required range contiguous?
2. Is the window size fixed or variable?
3. What information must be maintained: sum, count, frequency, or something else?
4. When should the window expand?
5. When should it shrink?
6. Can the window condition be maintained correctly for the input values, including negative numbers?

If the answer to the first question is no, sliding window may not be the appropriate technique.

---

## 12. Practice Problems

### Easy

1. Maximum Average Subarray I.
2. Maximum Sum of a Subarray of Size `k`.
3. Contains Duplicate II.
4. Minimum Recolors to Get K Consecutive Black Blocks.

### Medium

1. Longest Substring Without Repeating Characters.
2. Longest Repeating Character Replacement.
3. Minimum Size Subarray Sum.
4. Permutation in String.
5. Find All Anagrams in a String.
6. Fruit Into Baskets.
7. Max Consecutive Ones III.
8. Subarray Product Less Than K.
9. Count Number of Nice Subarrays.

### Recommended practice order

1. Fixed-size window sum.
2. Fixed-size maximum or average.
3. Variable-size minimum-length subarray.
4. Longest substring without repeating characters.
5. Frequency-map window problems.
6. Count windows using `right - left + 1`.
7. Exactly `k` using the `atMost(k)` technique.

---

## Quick Revision

- **Fixed window:** Add the incoming element and remove the outgoing element.
- **Variable window:** Expand, then shrink until the condition is valid.
- **Window length:** `right - left + 1`.
- **Longest valid window:** Update the maximum after restoring validity.
- **Minimum valid window:** Record the answer while shrinking.
- **Frequency map:** Tracks element or character counts in the window.
- **Count valid windows:** Sometimes add `right - left + 1` for each right boundary.
- **Exactly K:** Often calculate `atMost(K) - atMost(K - 1)`.
- **Typical complexity:** O(n) time because each pointer moves forward at most `n` times.
- **Important limitation:** Standard sum-based shrinking logic may fail with negative values.

**Interview goal:** Explain why the window can move without recalculating every possible range, maintain its state correctly, and justify the time and space complexity.