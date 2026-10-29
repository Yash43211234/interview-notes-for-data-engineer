# Time Complexity in Python

## 1. What Is Time Complexity?

**Time complexity** describes how the number of operations performed by an algorithm grows as the input size increases.

It helps us estimate how an algorithm will scale when processing small or large inputs.

For example, suppose you need to find a number in a list.

- One approach checks every element.
- Another approach uses binary search on a sorted list.

Both may find the same answer, but they perform different amounts of work as the list grows.

Time complexity is usually expressed using **Big O notation**.

---

## 2. What Is Big O Notation?

Big O describes the asymptotic growth rate of an algorithm, typically focusing on its upper bound as input size becomes large.

Let \(n\) represent the input size.

Common time complexities include:

| Complexity | Name | Example |
|---|---|---|
| \(O(1)\) | Constant | Accessing a list element by index |
| \(O(\log n)\) | Logarithmic | Binary search |
| \(O(n)\) | Linear | Traversing a list |
| \(O(n \log n)\) | Linearithmic | Merge sort |
| \(O(n^2)\) | Quadratic | Comparing every pair of elements |
| \(O(n^3)\) | Cubic | Three nested loops over the same input |
| \(O(2^n)\) | Exponential | Generating all subsets |
| \(O(n!)\) | Factorial | Generating all permutations |

These are growth rates, not exact execution times in seconds.

For example, an \(O(n)\) algorithm does not necessarily run faster than every \(O(n^2)\) algorithm for tiny inputs. Big O helps compare how algorithms scale as inputs grow.

---

## 3. Constant Time — \(O(1)\)

An operation has constant time complexity when its work does not grow with input size.

```python
def get_first(numbers):
    return numbers[0]
```

Accessing an element by index in a Python list takes constant time.

Whether the list contains 10 elements or 1 million elements, the indexing operation requires approximately the same amount of work.

Another example:

```python
def is_even(number):
    return number % 2 == 0
```

This performs a fixed number of arithmetic operations for ordinary integer inputs.

**Important:** A function is not automatically \(O(1)\) just because it contains only one line. Its complexity depends on what that line does.

For example:

```python
def copy_list(numbers):
    return numbers.copy()
```

Copying a list containing \(n\) elements takes \(O(n)\) time, even though the function contains one line.

---

## 4. Linear Time — \(O(n)\)

An algorithm has linear time complexity when its work grows proportionally to the number of input elements.

```python
def calculate_sum(numbers):
    total = 0

    for number in numbers:
        total += number

    return total
```

If the list contains \(n\) elements, the loop processes each element once.

Therefore:

- Time complexity: \(O(n)\)
- Auxiliary space complexity: \(O(1)\)

Another example:

```python
def find_target(numbers, target):
    for index, number in enumerate(numbers):
        if number == target:
            return index

    return -1
```

In the worst case, the function examines every element.

- Best case: \(O(1)\), if the first element matches.
- Worst case: \(O(n)\), if the target is absent or appears last.

Unless a question specifies otherwise, state the worst-case complexity when analyzing an algorithm.

---

## 5. Quadratic Time — \(O(n^2)\)

Quadratic time often occurs when one loop processes all elements and another loop processes all elements for each iteration of the first loop.

```python
def print_pairs(numbers):
    for first in numbers:
        for second in numbers:
            print(first, second)
```

If the input contains \(n\) elements:

- The outer loop runs \(n\) times.
- The inner loop runs \(n\) times for every outer iteration.

Total operations are proportional to:

\[
n \times n = n^2
\]

Therefore, time complexity is \(O(n^2)\).

### Important distinction

Nested loops do not always mean \(O(n^2)\).

```python
def print_triangle(n):
    for first in range(n):
        for second in range(first):
            print(first, second)
```

The inner loop runs a different number of times on each iteration.

The total number of iterations is:

\[
0 + 1 + 2 + \cdots + (n-1)
= \frac{n(n-1)}{2}
\]

Ignoring constants and lower-order terms, the complexity is still \(O(n^2)\).

However, if the inner loop runs a fixed number of times, the overall complexity may be \(O(n)\).

---

## 6. Logarithmic Time — \(O(\log n)\)

Logarithmic time occurs when an algorithm repeatedly reduces the problem size by a constant factor, such as dividing it in half.

Binary search is a classic example.

```python
def binary_search(numbers, target):
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

This algorithm requires the list to be sorted.

At each iteration, approximately half of the remaining search range is discarded.

For example, if the list has 1,024 elements, the search range can be reduced approximately as follows:

1. 1,024 elements
2. 512 elements
3. 256 elements
4. 128 elements
5. Continue halving until the target is found or the range becomes empty.

The number of iterations grows logarithmically.

- Time complexity: \(O(\log n)\)
- Auxiliary space complexity: \(O(1)\)

This iterative implementation uses a fixed number of variables.

---

## 7. Linearithmic Time — \(O(n \log n)\)

Linearithmic complexity commonly appears in efficient comparison-based sorting algorithms.

For example, merge sort divides a list into smaller parts, sorts those parts, and merges them.

Its typical recurrence is:

\[
T(n) = 2T(n/2) + O(n)
\]

There are approximately \(\log n\) levels of division, and each level performs a total of \(O(n)\) merging work.

Therefore:

\[
T(n) = O(n \log n)
\]

Python's built-in sorting methods, such as `list.sort()` and `sorted()`, use Timsort. Their worst-case time complexity is \(O(n \log n)\), with better performance possible for certain existing order patterns.

```python
numbers = [5, 2, 8, 1, 3]

numbers.sort()
```

- `numbers.sort()` modifies the original list.
- `sorted(numbers)` creates a new sorted list.

Do not assume that the two operations have identical memory requirements.

---

## 8. Exponential Time — \(O(2^n)\)

Exponential complexity can occur when an algorithm explores every possible combination of binary choices.

For example, generating all subsets of a collection containing \(n\) elements produces \(2^n\) subsets.

Each element can either be included or excluded.

The number of subsets is:

\[
2^n
\]

A simple recursive implementation is:

```python
def generate_subsets(numbers):
    if not numbers:
        return [[]]

    remaining = generate_subsets(numbers[1:])

    return (
        remaining
        + [[numbers[0]] + subset for subset in remaining]
    )
```

For \(n\) input elements, the number of returned subsets is \(2^n\).

Because the algorithm constructs and stores all subsets, the actual work includes the cost of constructing their contents, not just counting the subsets. Thus, its total running time can exceed \(O(2^n)\); a typical bound for explicitly constructing all subsets this way is \(O(n2^n)\).

**Key point:** When an algorithm must generate an exponentially large output, it cannot produce that entire output in polynomial time.

---

## 9. Factorial Time — \(O(n!)\)

Factorial complexity often appears when an algorithm generates every possible ordering of \(n\) elements.

The number of permutations is:

\[
n! = n(n-1)(n-2)\cdots 1
\]

For example:

```python
from itertools import permutations

numbers = [1, 2, 3]

for arrangement in permutations(numbers):
    print(arrangement)
```

For \(n\) distinct elements, there are \(n!\) permutations.

If an algorithm explicitly constructs every permutation, the total running time also depends on how much work is required to construct or process each permutation.

Factorial algorithms become impractical quickly as input size grows.

---

## 10. Time Complexity of Common Python Operations

These are typical complexities for built-in Python lists, sets, and dictionaries. They assume ordinary operations and, where noted, average-case hash-table behavior.

| Operation | Typical time complexity |
|---|---|
| `numbers[i]` | \(O(1)\) |
| `numbers.append(x)` | Amortized \(O(1)\) |
| `numbers.pop()` | \(O(1)\) |
| `numbers.insert(0, x)` | \(O(n)\) |
| `numbers.pop(0)` | \(O(n)\) |
| `x in numbers` for a list | \(O(n)\) |
| `numbers.copy()` | \(O(n)\) |
| `numbers.sort()` | Worst case \(O(n \log n)\) |
| `sorted(numbers)` | Worst case \(O(n \log n)\) |
| `x in my_set` | Average \(O(1)\) |
| `key in my_dict` | Average \(O(1)\) |
| Building a set from \(n\) items | Average \(O(n)\) |
| Building a dictionary from \(n\) entries | Average \(O(n)\) |

Dictionary and set operations can degrade under unusual hash-collision conditions. Also, amortized \(O(1)\) does not mean every individual append operation takes constant time: an occasional resize can take longer.

---

## 11. Single Loops vs. Nested Loops

Consider these examples.

### Example A: One loop

```python
def example_a(numbers):
    for number in numbers:
        print(number)
```

Time complexity: \(O(n)\).

### Example B: Two independent loops

```python
def example_b(numbers):
    for number in numbers:
        print(number)

    for number in numbers:
        print(number)
```

The total work is approximately \(2n\).

\[
O(n) + O(n) = O(2n) = O(n)
\]

Time complexity: \(O(n)\).

### Example C: Nested loops

```python
def example_c(numbers):
    for first in numbers:
        for second in numbers:
            print(first, second)
```

The total work is approximately \(n^2\).

Time complexity: \(O(n^2)\).

**Remember:**

- Sequential loops usually add their costs.
- Nested loops often multiply their costs when each loop depends on the same input size.
- Keep the dominant term when simplifying Big O.

---

## 12. Different Input Sizes

Not all nested loops operate on the same input size.

```python
def compare_lists(first, second):
    for a in first:
        for b in second:
            print(a, b)
```

Let:

- \(n\) = length of `first`
- \(m\) = length of `second`

The inner loop runs \(m\) times for every one of the \(n\) outer iterations.

Time complexity:

\[
O(nm)
\]

Do not automatically write \(O(n^2)\) unless both input sizes are represented by the same variable or are assumed to grow together.

This distinction is important in interview problems involving two arrays, strings, or tables.

---

## 13. Multiple Operations and Dominant Terms

Suppose an algorithm performs \(n^2\) operations and then \(n\) additional operations.

Its total work can be represented as:

\[
T(n) = 3n^2 + 5n + 10
\]

Big O ignores constant factors and lower-order terms.

Therefore:

\[
T(n) = O(n^2)
\]

Similarly:

\[
O(n^2) + O(n) = O(n^2)
\]

The quadratic term dominates as \(n\) grows.

However, when two different inputs are involved, retain their variables where necessary rather than removing meaningful differences.

---

## 14. Best, Average, and Worst Cases

An algorithm's running time can depend on the input arrangement.

Consider linear search:

```python
def find_target(numbers, target):
    for index, number in enumerate(numbers):
        if number == target:
            return index

    return -1
```

| Case | Situation | Time complexity |
|---|---|---|
| Best | Target is first | \(O(1)\) |
| Average | Target is found after examining a portion of the list | \(O(n)\) |
| Worst | Target is last or absent | \(O(n)\) |

The average-case result assumes a suitable input distribution. For linear search, a uniformly positioned target still requires a number of comparisons proportional to \(n\) on average.

When explaining an algorithm, state which case you are analyzing.

---

## 15. Amortized Time Complexity

Amortized analysis evaluates the average cost per operation over a sequence of operations, even if some individual operations are expensive.

Python list appending is a common example.

```python
numbers = []

for number in range(1_000_000):
    numbers.append(number)
```

Most append operations take constant time. Occasionally, Python allocates a larger underlying storage area and copies existing references.

- Typical individual append: \(O(1)\)
- Amortized append: \(O(1)\)
- Appending \(n\) elements overall: \(O(n)\)

Amortized complexity is different from average-case complexity. Amortized analysis concerns a sequence of operations, while average-case analysis depends on a probability distribution over inputs.

---

## 16. How to Analyze Time Complexity Step by Step

Use this method in coding interviews.

### Step 1: Define the input size

Identify what \(n\) represents.

For a list, \(n\) is usually its length. For two lists, use \(n\) and \(m\) if their lengths can differ.

### Step 2: Count repeated operations

Identify loops, recursive calls, and expensive operations.

### Step 3: Analyze loops

- One full traversal: often \(O(n)\).
- Two sequential traversals: often \(O(n)\).
- Two nested traversals over \(n\) elements: often \(O(n^2)\).
- Repeatedly halving a range: often \(O(\log n)\).

### Step 4: Analyze library operations

Remember that a single line can perform significant work.

For example:

```python
if target in numbers:
    ...
```

For a list, membership checking is \(O(n)\) in the worst case.

For a set, membership is average \(O(1)\).

### Step 5: Analyze recursion

Determine:
- How many recursive calls are made?
- How much does the input shrink per call?
- How much work happens at each level?

### Step 6: Simplify

Remove constant factors and lower-order terms.

For example:

\[
O(4n^2 + 3n + 7) = O(n^2)
\]

### Step 7: State assumptions

Mention whether you are describing best-case, average-case, worst-case, or amortized complexity.

---

## 17. Common Mistakes

### Mistake 1: Counting lines instead of operations

A single line can traverse an entire collection or sort thousands of elements.

**Correction:** Analyze how the work changes with input size.

### Mistake 2: Assuming every nested loop is \(O(n^2)\)

Different loop bounds can produce different complexities.

**Correction:** Count the actual iterations or form a sum.

### Mistake 3: Forgetting built-in operation costs

A membership check in a list is not the same as a membership check in a set.

**Correction:** Know the typical complexity of common data structures and operations.

### Mistake 4: Confusing time and space complexity

A nested loop may use quadratic time but constant auxiliary space if it does not store the pairs.

**Correction:** Analyze execution work and additional memory separately.

### Mistake 5: Ignoring early returns

A search algorithm may finish immediately for some inputs.

**Correction:** Distinguish best-case from worst-case complexity.

### Mistake 6: Treating Big O as an exact runtime

\(O(n)\) does not mean the program takes \(n\) seconds.

**Correction:** Big O describes asymptotic growth, not a fixed time measurement.

### Mistake 7: Assuming \(O(1)\) means the same number of CPU cycles in every situation

Constant complexity means the number of relevant operations does not grow with input size under the stated model. Real runtime can still vary with hardware, data types, memory allocation, and system conditions.

---

## 18. Interview Questions and Answers

### Q1. What is time complexity?

Time complexity describes how the amount of work an algorithm performs grows as its input size increases.

### Q2. What is Big O notation?

Big O expresses an asymptotic upper bound on an algorithm's growth rate. It is commonly used to describe worst-case complexity, although it can describe other upper bounds when specified.

### Q3. What is the difference between \(O(n)\) and \(O(n^2)\)?

An \(O(n)\) algorithm's work grows proportionally to input size. An \(O(n^2)\) algorithm's work grows proportionally to the square of input size. Quadratic algorithms generally scale much less efficiently for large inputs.

### Q4. Why is binary search \(O(\log n)\)?

Each iteration eliminates approximately half the remaining search range. The number of times the range can be halved before it becomes empty grows logarithmically with the input size.

### Q5. Why is checking membership in a list \(O(n)\)?

A list may require checking every element until a match is found or the list ends.

### Q6. Why is set membership usually \(O(1)\)?

Python sets use hash tables, which generally support constant-time membership checks on average. Unusual hash collisions can make an operation slower.

### Q7. What is the difference between time complexity and space complexity?

Time complexity describes how computational work grows. Space complexity describes how memory requirements grow.

### Q8. What is amortized time complexity?

Amortized analysis evaluates the cost per operation across a sequence of operations, accounting for occasional expensive operations. Python list append is amortized \(O(1)\).

### Q9. What is the time complexity of sorting a Python list?

Python's built-in list sorting uses Timsort and has worst-case time complexity \(O(n \log n)\).

### Q10. Can an algorithm have \(O(n^2)\) time complexity but \(O(1)\) auxiliary space?

Yes. An algorithm can compare every pair of elements using nested loops without storing the pairs.

### Q11. What is the time complexity of two sequential loops?

If each loop processes \(n\) elements, the total work is proportional to \(2n\), which simplifies to \(O(n)\).

### Q12. What is the time complexity of a loop that halves its input each time?

It is typically \(O(\log n)\), assuming each iteration performs constant work and reduces the remaining input by a constant factor.

---

## 19. Practice Problems

Determine the time complexity of each function before revealing the answers.

### Problem 1

```python
def problem_one(numbers):
    for number in numbers:
        print(number)
```

### Problem 2

```python
def problem_two(numbers):
    for first in numbers:
        for second in numbers:
            print(first, second)
```

### Problem 3

```python
def problem_three(numbers):
    for number in numbers:
        print(number)

    for number in numbers:
        print(number)
```

### Problem 4

```python
def problem_four(n):
    while n > 1:
        n //= 2
```

### Problem 5

```python
def problem_five(first, second):
    for a in first:
        for b in second:
            print(a, b)
```

### Problem 6

```python
def problem_six(numbers, target):
    return target in numbers
```

{@body const [showAnswers, setShowAnswers] = DIL.useState(false)}
<button block variant="outline" onClick={() => setShowAnswers(!showAnswers)}>{showAnswers ? "Hide answers" : "Reveal answers"}</button>
{#if showAnswers}
  <box border radius="lg" padding={3} gap={3}>
    **Problem 1:** \(O(n)\). Each element is processed once.

    **Problem 2:** \(O(n^2)\). Every element is paired with every other element.

    **Problem 3:** \(O(n)\). Two sequential traversals produce \(2n\) operations, which simplifies to \(O(n)\).

    **Problem 4:** \(O(\log n)\). The value of `n` is halved each iteration.

    **Problem 5:** \(O(nm)\), where \(n\) and \(m\) are the lengths of the two inputs.

    **Problem 6:** \(O(n)\) worst case because `numbers` is a list. For a set, membership is average \(O(1)\).
  </box>
{/if}

---

## 20. Quick Revision

| Pattern | Time complexity |
|---|---:|
| Access a list element by index | \(O(1)\) |
| Traverse a list | \(O(n)\) |
| Search a list in the worst case | \(O(n)\) |
| Binary search | \(O(\log n)\) |
| Two sequential loops over the same list | \(O(n)\) |
| Two nested loops over the same list | \(O(n^2)\) |
| Compare every element of two lists | \(O(nm)\) |
| Efficient comparison-based sorting | \(O(n \log n)\) |
| Generate all subsets | At least \(O(2^n)\) outputs; construction work can be \(O(n2^n)\) |
| Generate all permutations | \(n!\) outputs; total work depends on construction and processing |

**Final rule:** To determine time complexity, ask: *As the input grows, how many times does the algorithm perform its work?* Identify the dominant growth rate, state your assumptions, and simplify using Big O notation.