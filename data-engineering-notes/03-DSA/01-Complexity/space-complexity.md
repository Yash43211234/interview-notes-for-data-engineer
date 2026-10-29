# Space Complexity in Python

## 1. What Is Space Complexity?

**Space complexity** describes how the memory usage of an algorithm grows as the input size increases.

It helps us understand how much additional memory an algorithm needs to solve a problem.

For example, an algorithm might process 100 elements using a few variables, while another algorithm creates a new list containing all 100 elements.

Both algorithms may produce the same output, but their memory requirements differ.

Space complexity is commonly expressed using Big O notation:

- \(O(1)\): Constant space
- \(O(n)\): Linear space
- \(O(n^2)\): Quadratic space
- \(O(\log n)\): Logarithmic space

Here, \(n\) represents the input size.

---

## 2. Auxiliary Space vs. Total Space

These two terms are related but not identical.

### Auxiliary space

Auxiliary space is the additional working memory an algorithm uses, excluding the input storage itself.

### Total space

Total space includes the input storage and the additional memory required by the algorithm, depending on the analysis convention.

Consider this example:

```python
def find_maximum(numbers):
    maximum = numbers[0]

    for number in numbers:
        if number > maximum:
            maximum = number

    return maximum
```

The function uses only a few extra variables, regardless of how many elements are in `numbers`.

- Auxiliary space: \(O(1)\)
- Total space, including the input list: \(O(n)\)

In algorithm analysis, when someone says this function uses constant space, they usually mean **constant auxiliary space**, assuming the input list already exists.

---

## 3. Common Space Complexities

| Complexity | Meaning | Typical example |
|---|---|---|
| \(O(1)\) | Constant additional memory | A fixed number of variables |
| \(O(\log n)\) | Grows logarithmically | Recursive binary search stack |
| \(O(n)\) | Grows linearly | Copying an input list |
| \(O(n \log n)\) | Linearithmic memory growth | Some algorithms using additional buffers |
| \(O(n^2)\) | Grows quadratically | Storing all pairs of elements |

These describe growth rates, not exact memory consumption in bytes.

For example, an \(O(n)\) algorithm does not necessarily use more memory than every \(O(\log n)\) algorithm for every tiny input. Big O describes asymptotic growth as input size increases.

---

## 4. Constant Space — \(O(1)\)

An algorithm uses constant auxiliary space when its extra memory does not grow with the input size.

```python
def calculate_sum(numbers):
    total = 0

    for number in numbers:
        total += number

    return total
```

The function uses:
- `total`
- `number`
- A fixed amount of loop and function state

It does not create a new collection proportional to the input size.

Therefore:

- Time complexity: \(O(n)\)
- Auxiliary space complexity: \(O(1)\)

The loop processes \(n\) elements, but it does not store all those elements in additional memory.

### Important distinction

```python
def calculate_sum_copy(numbers):
    copied_numbers = list(numbers)
    return sum(copied_numbers)
```

If `numbers` contains \(n\) elements, `copied_numbers` stores another \(n\) elements.

Therefore:

- Time complexity: \(O(n)\)
- Auxiliary space complexity: \(O(n)\)

Even though both functions calculate a sum, their additional memory usage differs.

---

## 5. Linear Space — \(O(n)\)

An algorithm uses linear auxiliary space when its additional memory grows proportionally to the input size.

```python
def square_numbers(numbers):
    result = []

    for number in numbers:
        result.append(number * number)

    return result
```

If the input contains \(n\) numbers, the output contains \(n\) numbers.

Therefore:

- Time complexity: \(O(n)\)
- Auxiliary space complexity: \(O(n)\), counting the output as additional memory

If a question explicitly excludes output space, state that convention when giving your answer.

### Another example: List comprehension

```python
def get_even_numbers(numbers):
    return [number for number in numbers if number % 2 == 0]
```

In the worst case, every element is even, so the result contains \(n\) elements.

Auxiliary space: \(O(n)\), including the returned list.

---

## 6. Quadratic Space — \(O(n^2)\)

Quadratic space occurs when an algorithm stores a number of elements proportional to \(n^2\).

```python
def create_matrix(n):
    return [
        [0 for _ in range(n)]
        for _ in range(n)
    ]
```

For an \(n \times n\) matrix, the number of stored elements is:

\[
n \times n = n^2
\]

Therefore:

- Time complexity: \(O(n^2)\)
- Auxiliary space complexity: \(O(n^2)\)

For example, a matrix of size \(100 \times 100\) contains 10,000 entries.

A nested loop does not automatically imply quadratic space. The memory complexity depends on what the algorithm stores.

```python
def print_pairs(numbers):
    for first in numbers:
        for second in numbers:
            print(first, second)
```

This function has \(O(n^2)\) time complexity because it processes every pair, but its auxiliary space is \(O(1)\), excluding output and runtime buffering.

**Key point:** Nested loops often affect time complexity, while stored collections and recursive calls affect space complexity.

---

## 7. Logarithmic Space — \(O(\log n)\)

Logarithmic auxiliary space often appears in recursive algorithms that divide the problem size by a constant factor at each step.

Consider recursive binary search:

```python
def binary_search(numbers, target, left, right):
    if left > right:
        return -1

    middle = (left + right) // 2

    if numbers[middle] == target:
        return middle

    if numbers[middle] < target:
        return binary_search(
            numbers, target, middle + 1, right
        )

    return binary_search(
        numbers, target, left, middle - 1
    )
```

Each recursive call reduces the search interval approximately by half.

The number of recursive calls along the active path is \(O(\log n)\).

Therefore:

- Time complexity: \(O(\log n)\)
- Auxiliary space complexity: \(O(\log n)\), due to the recursion stack

The iterative version can use constant auxiliary space:

```python
def binary_search_iterative(numbers, target):
    left = 0
    right = len(numbers) - 1

    while left <= right:
        middle = (left + right) // 2

        if numbers[middle] == target:
            return middle

        if numbers[middle] < target:
            left = middle + 1
        else:
            right = middle - 1

    return -1
```

The iterative version uses a fixed number of variables.

Therefore, its auxiliary space is \(O(1)\).

Both versions assume that `numbers` is sorted.

---

## 8. Recursion Stack Space

Every active recursive function call requires information such as local variables and the return location.

This information is stored on the call stack.

Consider:

```python
def countdown(n):
    if n <= 0:
        return

    countdown(n - 1)
```

For an input of \(n\), the maximum recursion depth is proportional to \(n\).

Therefore:

- Time complexity: \(O(n)\)
- Auxiliary space complexity: \(O(n)\), due to the recursion stack

Compare this with an iterative implementation:

```python
def countdown_iterative(n):
    while n > 0:
        n -= 1
```

Its auxiliary space is \(O(1)\).

### Recursion and memory

A recursive solution may look shorter, but it can consume more memory than an iterative solution.

In Python, deep recursion can also raise `RecursionError` because Python limits recursion depth.

---

## 9. Space Complexity of Common Python Operations

The following are typical auxiliary-memory considerations for standard Python operations. Exact behavior can depend on the implementation and input.

| Operation | Typical additional space |
|---|---|
| Assign a reference to an existing object | \(O(1)\) |
| Create a list of \(n\) elements | \(O(n)\) |
| Copy a list containing \(n\) elements | \(O(n)\) |
| Create a set from \(n\) elements | \(O(n)\) |
| Create a dictionary with \(n\) entries | \(O(n)\) |
| Create a slice of a list containing \(k\) elements | \(O(k)\) |
| Use a generator expression | Usually \(O(1)\) retained iteration state |
| Sort a list | Depends on the algorithm and implementation |
| Build an \(n \times n\) matrix | \(O(n^2)\) |

### Reference assignment vs. copying

```python
numbers = [1, 2, 3, 4]

same_list = numbers
```

`same_list` references the same list. The assignment does not create another list containing all elements.

Additional space for the reference: \(O(1)\).

Now consider:

```python
copied_list = numbers.copy()
```

This creates a new list containing references to the elements.

Additional space: \(O(n)\).

For nested mutable objects, `.copy()` is shallow: it does not recursively duplicate every nested object.

---

## 10. List Comprehension vs. Generator Expression

This distinction is useful in Python interviews and data engineering.

### List comprehension

```python
squares = [number * number for number in range(1_000_000)]
```

The list stores one million results.

Additional space: \(O(n)\).

### Generator expression

```python
squares = (
    number * number
    for number in range(1_000_000)
)
```

The generator calculates values as they are requested instead of storing all results in a list.

Its retained iteration state is typically \(O(1)\), excluding the memory needed to store consumed results or any other objects retained by the surrounding program.

Example:

```python
total = sum(
    number * number
    for number in range(1_000_000)
)
```

The generator avoids creating a list of one million squared values.

This is useful when processing large datasets, but generators are not automatically faster for every task. They trade materialized storage for incremental computation.

---

## 11. Two-Pointer Technique and Space Complexity

The two-pointer technique is commonly used in array and string problems.

Example: reverse a list in place.

```python
def reverse_list(numbers):
    left = 0
    right = len(numbers) - 1

    while left < right:
        numbers[left], numbers[right] = (
            numbers[right],
            numbers[left],
        )

        left += 1
        right -= 1

    return numbers
```

The algorithm modifies the original list rather than creating a separate reversed list.

- Time complexity: \(O(n)\)
- Auxiliary space complexity: \(O(1)\)

Contrast this with:

```python
def reverse_list_copy(numbers):
    return numbers[::-1]
```

The slice creates a new list containing \(n\) references.

- Time complexity: \(O(n)\)
- Auxiliary space complexity: \(O(n)\)

**Important:** The in-place version modifies the input. That is only appropriate when changing the original list is allowed.

---

## 12. Hash Maps and Sets

Dictionaries and sets are useful for fast lookups, but they consume additional memory for the entries they store.

Example: find duplicates.

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

For \(n\) input elements:

- Expected time complexity: \(O(n)\)
- Auxiliary space complexity: \(O(n)\)

The sets may contain a number of elements proportional to the input size.

An alternative is to sort the input and check adjacent values. Depending on whether sorting can modify the input, this may reduce additional collection storage, but sorting has its own time and memory costs.

Choose the approach based on the problem's requirements.

---

## 13. How to Calculate Space Complexity Step by Step

Use this process during coding interviews.

### Step 1: Identify the input size

Let \(n\) be the number of input elements.

### Step 2: Identify extra data structures

Look for:
- Lists
- Dictionaries
- Sets
- Matrices
- Copies of inputs

Determine how their sizes grow with \(n\).

### Step 3: Count active recursive calls

If the function is recursive, determine the maximum recursion depth.

### Step 4: Distinguish stored output from working memory

Check whether the interviewer expects output space to be included or excluded.

### Step 5: Keep the dominant term

If an algorithm allocates \(O(n)\) space and \(O(1)\) additional variables, the total auxiliary space remains \(O(n)\).

For example:

\[
O(n) + O(1) = O(n)
\]

If two structures each grow linearly:

\[
O(n) + O(n) = O(n)
\]

Big O ignores constant factors.

---

## 14. Common Mistakes

### Mistake 1: Assuming nested loops mean quadratic space

Nested loops may increase execution time without increasing the amount of stored memory.

**Remember:** Analyze what the algorithm stores, not only how many loops it contains.

### Mistake 2: Ignoring recursion stack memory

A recursive function may create no lists or dictionaries and still use substantial memory through its call stack.

### Mistake 3: Forgetting output space

A function that returns a list of \(n\) elements needs space for that output. State whether output space is included in your analysis.

### Mistake 4: Treating assignment as copying

```python
b = a
```

This creates another reference, not another list.

```python
b = a.copy()
```

This creates a new outer list.

### Mistake 5: Assuming in-place algorithms use no memory

In-place algorithms can still use a fixed number of temporary variables or other small structures. Their auxiliary space may be \(O(1)\), not literally zero.

### Mistake 6: Assuming generators use no memory

Generators maintain execution state and may retain references to objects. Their advantage is that they usually avoid storing the entire sequence of generated results.

### Mistake 7: Optimizing space without considering other requirements

Reducing memory can increase runtime or require modifying the input. Evaluate time complexity, memory usage, readability, and correctness together.

---

## 15. Interview Questions and Answers

### Q1. What is space complexity?

Space complexity describes how an algorithm's memory requirements grow as input size increases.

### Q2. What is the difference between auxiliary space and total space?

Auxiliary space is the extra working memory used by an algorithm, while total space includes the input storage as well, according to the analysis convention.

### Q3. What is the space complexity of finding the maximum element in a list?

An iterative solution that uses a fixed number of variables has \(O(1)\) auxiliary space, excluding the input list.

### Q4. What is the space complexity of reversing a list using slicing?

For `numbers[::-1]`, the new list requires \(O(n)\) additional space.

### Q5. Why does recursive binary search use \(O(\log n)\) space?

Each recursive call reduces the search interval by approximately half, resulting in \(O(\log n)\) active calls in the worst case.

### Q6. What is the difference between list comprehensions and generators in terms of space?

A list comprehension stores all generated elements, requiring \(O(n)\) space in the general case. A generator expression produces elements incrementally and usually maintains only a small amount of iteration state.

### Q7. Can an algorithm have \(O(n^2)\) time complexity and \(O(1)\) auxiliary space?

Yes. An algorithm can process every pair of elements using nested loops without storing all the pairs.

### Q8. How can you reduce space complexity?

Possible approaches include:
- Avoiding unnecessary copies.
- Using generators for incremental processing.
- Modifying the input in place when allowed.
- Reusing existing data structures.
- Replacing recursive algorithms with iterative alternatives when appropriate.
- Processing data in batches rather than loading everything into memory.

### Q9. What is the space complexity of a dictionary used to count frequencies?

A frequency dictionary generally requires \(O(k)\) auxiliary space, where \(k\) is the number of distinct keys. In the worst case, \(k = n\), so the complexity is \(O(n)\).

### Q10. Does \(O(1)\) space mean an algorithm uses no memory?

No. It means the additional memory does not grow with input size. The algorithm still uses variables, runtime state, and possibly a fixed amount of temporary memory.

---

## 16. Practice Problems

Try determining both time and auxiliary space complexity before checking the answers.

### Problem 1

```python
def example_one(numbers):
    total = 0

    for number in numbers:
        total += number

    return total
```

### Problem 2

```python
def example_two(numbers):
    result = []

    for number in numbers:
        result.append(number * 2)

    return result
```

### Problem 3

```python
def example_three(numbers):
    for first in numbers:
        for second in numbers:
            print(first, second)
```

### Problem 4

```python
def example_four(n):
    if n <= 1:
        return

    example_four(n - 1)
```

### Problem 5

```python
def example_five(numbers):
    return list(set(numbers))
```

<text weight="medium" size="md">Answers</text>

{@body const [showAnswers, setShowAnswers] = DIL.useState(false)}
<button block variant="outline" onClick={() => setShowAnswers(!showAnswers)}>{showAnswers ? "Hide answers" : "Reveal answers"}</button>
{#if showAnswers}
  <box border radius="lg" padding={3} gap={3}>
    **Problem 1:** Time \(O(n)\), auxiliary space \(O(1)\). It stores only a fixed number of variables.

    **Problem 2:** Time \(O(n)\), auxiliary space \(O(n)\). The output list grows with the input.

    **Problem 3:** Time \(O(n^2)\), auxiliary space \(O(1)\), excluding output and runtime buffering. It processes every pair without retaining them in a collection.

    **Problem 4:** Time \(O(n)\), auxiliary space \(O(n)\). Recursive calls accumulate on the call stack.

    **Problem 5:** Expected time \(O(n)\), auxiliary space \(O(n)\). The set and returned list can each grow proportionally to the number of input elements. The exact time depends on hashing behavior.
  </box>
{/if}

---

## 17. Quick Revision

| Pattern | Auxiliary space |
|---|---:|
| Iterate and calculate a sum | \(O(1)\) |
| Create a new list of \(n\) results | \(O(n)\) |
| Copy a list | \(O(n)\) |
| Build a frequency dictionary | \(O(n)\) worst case |
| Store an \(n \times n\) matrix | \(O(n^2)\) |
| Recursive binary search | \(O(\log n)\) |
| Iterative binary search | \(O(1)\) |
| Recursive countdown | \(O(n)\) |
| Process pairs using nested loops without storing them | \(O(1)\) |
| Generate values lazily | Usually \(O(1)\) retained iteration state |

**Final rule:** To analyze space complexity, ask yourself: *As the input grows, what additional data does this algorithm keep in memory, and how much does that data grow?*