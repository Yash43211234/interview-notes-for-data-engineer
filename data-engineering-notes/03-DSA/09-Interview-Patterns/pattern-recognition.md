# Dynamic Programming Pattern Recognition

## 1. What Is Pattern Recognition in DP?

Pattern recognition means identifying the type of problem you are solving and selecting the appropriate algorithmic approach.

In Dynamic Programming (DP), many problems look different but share similar structures. Recognizing these structures helps you decide:

- What the DP state should represent.
- Which previous states are needed.
- What recurrence relation to use.
- Whether memoization or tabulation is suitable.
- Whether the solution can be optimized.

**The goal is not to memorize solutions. It is to recognize the underlying pattern.**

---

## 2. The DP Decision Process

When you encounter a problem, follow this process:

1. Identify the choices available at each step.
2. Check whether the same subproblems can occur repeatedly.
3. Determine whether the answer depends on smaller subproblems.
4. Define the state.
5. Write the recurrence.
6. Establish the base cases.
7. Choose memoization or tabulation.
8. Analyze time and space complexity.

A useful first question is:

**Does the problem ask me to count, maximize, minimize, or determine whether something is possible?**

This often provides an initial clue about the pattern.

---

## 3. Common DP Patterns

<text color="secondary" size="sm">Use this table as a recognition guide, not as a replacement for deriving the recurrence.</text>

| Pattern | Clues in the problem | Typical examples |
|---|---|---|
| Fibonacci / Linear DP | Current state depends on a few previous states | Fibonacci, Climbing Stairs |
| Take or Skip | Choose whether to select an item | House Robber |
| Grid DP | Move through a grid using allowed directions | Unique Paths, Minimum Path Sum |
| 0/1 Knapsack | Select items with limited capacity; each item used at most once | Subset Sum, Partition Equal Subset Sum |
| Unbounded Knapsack | An item can be selected repeatedly | Coin Change, Rod Cutting |
| Subsequence DP | Find the longest, shortest, or number of valid subsequences | LIS, LCS |
| String DP | Transform, match, or compare strings | Edit Distance, Distinct Subsequences |
| Interval DP | Solve subproblems over contiguous ranges | Matrix Chain Multiplication, Burst Balloons |
| State Machine DP | Decisions depend on a small state such as holding or not holding stock | Stock Buy/Sell |
| Partition DP | Divide an array or range into groups | Palindrome Partitioning |
| Bitmask DP | Track a subset using bits | Traveling Salesperson Problem |
| Tree DP | Combine answers from child nodes | Maximum Independent Set on a Tree |
| DAG DP | Calculate answers along directed acyclic graph dependencies | Longest Path in a DAG |

---

## 4. Pattern 1: Fibonacci and Linear DP

### Recognition clues

- The current answer depends on one or more previous positions.
- You move through a sequence or a set of positions.
- The number of previous states needed is relatively small.

### Example: Climbing Stairs

You can climb either one or two steps at a time. Find the number of ways to reach step `n`.

At step `n`, you must have arrived from step `n - 1` or `n - 2`.

\[
dp[i]=dp[i-1]+dp[i-2]
\]

```python
def climb_stairs(n: int) -> int:
    if n <= 1:
        return 1

    prev2 = 1
    prev1 = 1

    for _ in range(2, n + 1):
        current = prev1 + prev2
        prev2 = prev1
        prev1 = current

    return prev1
```

**Complexity:** \(O(n)\) time and \(O(1)\) auxiliary space.

Recognize this pattern when each state can be derived from a fixed number of earlier states.

---

## 5. Pattern 2: Take or Skip

### Recognition clues

- Each item presents a choice: select it or leave it.
- Selecting an item may restrict which other items can be selected.
- The objective is to maximize or minimize a result.

### Example: House Robber

Given an array of money in houses, maximize the amount collected without robbing adjacent houses.

At each house, either:

- Skip it and keep the previous best result.
- Rob it and add its money to the best result from two positions earlier.

\[
dp[i]=\max(dp[i-1],dp[i-2]+nums[i])
\]

```python
def rob(nums: list[int]) -> int:
    prev2 = 0
    prev1 = 0

    for money in nums:
        current = max(prev1, prev2 + money)
        prev2 = prev1
        prev1 = current

    return prev1
```

**Complexity:** \(O(n)\) time and \(O(1)\) auxiliary space.

The key recognition signal is a decision at each position that affects the choices available later.

---

## 6. Pattern 3: Grid DP

### Recognition clues

- The problem describes a grid, matrix, or board.
- You can move only in specified directions.
- The answer for a cell depends on neighboring cells.

### Example: Unique Paths

A robot starts at the top-left corner and moves only right or down. Count the paths to the bottom-right corner.

For a non-boundary cell:

\[
dp[r][c]=dp[r-1][c]+dp[r][c-1]
\]

The answer is the sum of paths arriving from above and from the left.

```python
def unique_paths(m: int, n: int) -> int:
    dp = [[1] * n for _ in range(m)]

    for r in range(1, m):
        for c in range(1, n):
            dp[r][c] = dp[r - 1][c] + dp[r][c - 1]

    return dp[m - 1][n - 1]
```

**Complexity:** \(O(mn)\) time and \(O(mn)\) space.

Recognize grid DP when the problem asks for the number of paths, the minimum cost to reach a cell, or the maximum value collected along a path.

---

## 7. Pattern 4: 0/1 Knapsack

### Recognition clues

- You have items with weights, costs, or values.
- There is a capacity, target sum, or limited resource.
- Each item can be selected at most once.
- You must maximize a value or determine whether a target is achievable.

### Example: Subset Sum

Determine whether a subset of numbers can sum to a target.

For each number, you can either skip it or use it once.

A two-dimensional state can be defined as:

`dp[i][s]` = whether a sum `s` can be formed using the first `i` numbers.

For a current value `x`:

\[
dp[i][s]=dp[i-1][s]\lor dp[i-1][s-x]
\]

The second option is considered only when \(s\ge x\).

**Important:** In a one-dimensional implementation, iterate sums in descending order. This prevents reusing the same item multiple times.

```python
def can_partition_sum(nums: list[int], target: int) -> bool:
    dp = [False] * (target + 1)
    dp[0] = True

    for x in nums:
        for s in range(target, x - 1, -1):
            dp[s] = dp[s] or dp[s - x]

    return dp[target]
```

**Complexity:** \(O(nT)\) time and \(O(T)\) space, where \(T\) is the target sum.

This is pseudo-polynomial time because the complexity depends on the numeric target, not just its number of digits.

---

## 8. Pattern 5: Unbounded Knapsack

### Recognition clues

- Items can be used repeatedly.
- Each selection contributes to a target, capacity, or total value.
- You need a minimum number of items, a maximum value, or the number of combinations.

### Example: Minimum Coin Change

Given coin denominations and a target amount, find the minimum number of coins needed.

For every amount, try using each coin that does not exceed that amount.

\[
dp[a]=1+\min_{c\le a}dp[a-c]
\]

Initialize unreachable amounts to infinity and set `dp[0] = 0`.

```python
def coin_change(coins: list[int], amount: int) -> int:
    dp = [float("inf")] * (amount + 1)
    dp[0] = 0

    for a in range(1, amount + 1):
        for coin in coins:
            if coin <= a:
                dp[a] = min(dp[a], 1 + dp[a - coin])

    return dp[amount] if dp[amount] != float("inf") else -1
```

**Complexity:** \(O(A \cdot C)\) time and \(O(A)\) space, where `A` is the amount and `C` is the number of coin denominations.

### How to distinguish 0/1 from unbounded knapsack

- **0/1:** Each item is available once.
- **Unbounded:** Each item can be used repeatedly.

For one-dimensional tabulation, 0/1 knapsack typically iterates capacities downward, while unbounded knapsack typically iterates upward when using the standard item-by-item recurrence.

---

## 9. Pattern 6: Subsequences

### Recognition clues

- You need the longest increasing subsequence.
- You need a longest common subsequence between two sequences.
- You want to count subsequences satisfying a condition.
- The selected elements must remain in their original order, but they do not need to be adjacent.

### Example: Longest Increasing Subsequence (LIS)

Find the length of the longest strictly increasing subsequence.

Define `dp[i]` as the length of the longest increasing subsequence ending at index `i`.

For every earlier element smaller than `nums[i]`, extend its subsequence.

\[
dp[i]=1+\max_{\substack{j<i\\nums[j]<nums[i]}}dp[j]
\]

If no valid earlier element exists, `dp[i] = 1`.

```python
def length_of_lis(nums: list[int]) -> int:
    if not nums:
        return 0

    dp = [1] * len(nums)

    for i in range(len(nums)):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)

    return max(dp)
```

**Complexity:** \(O(n^2)\) time and \(O(n)\) space.

A faster \(O(n\log n)\) solution also exists using a different approach. The quadratic DP is useful for learning the recurrence.

---

## 10. Pattern 7: String DP

### Recognition clues

- Two strings need to be compared or transformed.
- The problem asks for insertions, deletions, substitutions, matches, or subsequence counts.
- The state depends on prefixes of one or two strings.

### Example: Longest Common Subsequence

Given strings `a` and `b`, find the length of their longest common subsequence.

Define `dp[i][j]` as the LCS length of the first `i` characters of `a` and the first `j` characters of `b`.

If the last characters match:

\[
dp[i][j]=1+dp[i-1][j-1]
\]

Otherwise:

\[
dp[i][j]=\max(dp[i-1][j],dp[i][j-1])
\]

```python
def longest_common_subsequence(a: str, b: str) -> int:
    m, n = len(a), len(b)
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if a[i - 1] == b[j - 1]:
                dp[i][j] = 1 + dp[i - 1][j - 1]
            else:
                dp[i][j] = max(
                    dp[i - 1][j],
                    dp[i][j - 1]
                )

    return dp[m][n]
```

**Complexity:** \(O(mn)\) time and \(O(mn)\) space.

The extra row and column represent empty prefixes, making the base cases straightforward.

---

## 11. Pattern 8: Interval DP

### Recognition clues

- The problem operates on contiguous ranges of an array or string.
- You repeatedly split a range into smaller ranges.
- The order in which ranges are combined matters.
- The answer for an interval depends on smaller intervals.

### Example: Matrix Chain Multiplication

Given matrix dimensions, determine the minimum number of scalar multiplications needed to multiply the matrices.

Define `dp[i][j]` as the minimum cost of multiplying matrices from index `i` through `j`.

Try every possible split point `k`:

\[
dp[i][j]=\min_{i\le k<j}
\left(dp[i][k]+dp[k+1][j]+cost(i,k,j)\right)
\]

The exact multiplication cost depends on the matrix dimensions.

A common implementation fills intervals from shorter lengths to longer lengths, because smaller intervals must be calculated first.

**Recognition tip:** If the problem asks which split point produces the best result for a range, investigate interval DP.

---

## 12. Pattern 9: State Machine DP

### Recognition clues

- The problem involves changing between a small number of states.
- Current choices depend on the state you are in.
- Actions may be restricted by previous actions.

### Example: Stock Buy/Sell

For a stock trading problem, a state might represent whether you are currently holding a stock.

Possible states:

- `hold[i]`: best profit after day `i` while holding a stock.
- `cash[i]`: best profit after day `i` while not holding a stock.

With unlimited transactions and no transaction fee or cooldown:

\[
hold[i]=\max(hold[i-1],cash[i-1]-price[i])
\]

\[
cash[i]=\max(cash[i-1],hold[i-1]+price[i])
\]

The transitions represent holding or buying, and selling or staying out of the market.

**Recognition tip:** When the problem has explicit modes such as holding/not holding, bought/not bought, or cooldown/not cooldown, define a state for each mode.

---

## 13. Pattern 10: Bitmask DP

### Recognition clues

- The problem involves visiting or selecting a subset of items.
- The number of items is relatively small.
- You must track which items have already been selected or visited.
- The order of choices matters.

A bitmask represents a subset using the bits of an integer.

For example, with four items:

- `0000`: no items selected.
- `0101`: items 0 and 2 selected.
- `1111`: all four items selected.

A typical state is:

`dp[mask][i]` = the best answer for the subset represented by `mask`, ending at item `i`.

This is useful in problems such as the Traveling Salesperson Problem.

A common complexity is \(O(n^2 2^n)\), depending on the exact recurrence. Since the number of subsets grows exponentially, this approach is intended for relatively small `n`.

---

## 14. Pattern 11: Tree DP

### Recognition clues

- The input is a tree.
- The answer for a node depends on answers from its children.
- A node's decision may restrict the decisions of its children.

For example, in the maximum-weight independent set problem, you cannot select two directly connected nodes.

For each node, maintain two states:

- `take`: best answer when selecting the node.
- `skip`: best answer when not selecting the node.

The recurrence is:

\[
take[u]=w[u]+\sum_{v\in children(u)}skip[v]
\]

\[
skip[u]=\sum_{v\in children(u)}\max(take[v],skip[v])
\]

Here, `w[u]` is the weight of node `u`.

This is an example of DP on a tree, often calculated using a postorder traversal so child results are available before the parent is processed.

---

## 15. Pattern 12: DAG DP

### Recognition clues

- The problem involves a directed acyclic graph (DAG).
- A node's answer depends on its predecessors or successors.
- You need the longest path, shortest path in a DAG, or number of ways to reach a node.

Because a DAG has no directed cycles, its nodes can be processed in topological order.

For a longest path ending at node `v`:

\[
dp[v]=\max_{u\to v}(dp[u]+w(u,v))
\]

The exact initialization depends on whether the path can start at any node or must start at a specific source.

**Recognition tip:** When graph dependencies are acyclic, consider topological ordering combined with DP.

---

## 16. How to Choose the DP State

The state is one of the most important parts of a DP solution.

Ask yourself:

1. What information is necessary to describe the remaining problem?
2. Which decisions have already been made?
3. What information affects future choices?
4. Can the state be represented using an index, two indices, a capacity, a subset mask, or a small number of flags?

Examples:

| Problem | Possible state |
|---|---|
| Climbing Stairs | `dp[i]`: ways to reach step `i` |
| House Robber | `dp[i]`: maximum money from the first `i` houses |
| Grid Paths | `dp[r][c]`: paths to cell `(r, c)` |
| Subset Sum | `dp[i][s]`: whether sum `s` is achievable using the first `i` items |
| LCS | `dp[i][j]`: LCS length for two prefixes |
| LIS | `dp[i]`: LIS length ending at index `i` |
| Tree DP | `dp[u][state]`: answer for subtree rooted at `u` under a chosen state |

A good state contains enough information to calculate future results without needing the full history of decisions.

---

## 17. DP Pattern Recognition vs. Other Algorithms

Not every problem involving choices needs Dynamic Programming.

| Problem characteristics | Technique to investigate |
|---|---|
| Repeated subproblems with reusable results | Dynamic Programming |
| Local best choice can be proven globally optimal | Greedy |
| Explore possibilities and undo choices | Backtracking |
| Find shortest path in an unweighted graph | BFS |
| Find connectivity or reachability | DFS or BFS |
| Search a sorted array | Binary Search |
| Find the best answer over a moving contiguous range | Sliding Window, if its conditions apply |
| Find a shortest path with suitable nonnegative edge weights | Dijkstra's algorithm |

Some problems can be solved using more than one technique. Choose based on the constraints and correctness argument, not merely the wording.

---

## 18. Common Mistakes in Pattern Recognition

1. **Choosing DP too quickly:** First check whether a simpler algorithm solves the problem.
2. **Memorizing a pattern name:** Understand the recurrence and why it is correct.
3. **Defining an incomplete state:** If future decisions require missing information, the recurrence will be incorrect.
4. **Ignoring constraints:** \(O(n^2)\) may work for small input sizes but fail for large ones.
5. **Confusing subsequences with subarrays:** A subsequence preserves order but can skip elements; a subarray must be contiguous.
6. **Using the wrong item iteration order:** This can accidentally allow repeated use in a 0/1 knapsack problem.
7. **Incorrect initialization:** Base cases must match the exact meaning of each state.
8. **Ignoring impossible states:** Use an appropriate sentinel, such as infinity for minimization or a negative sentinel for maximization, when necessary.
9. **Optimizing space too early:** First implement and validate the full recurrence.
10. **Assuming keywords guarantee a pattern:** Words like “maximum,” “minimum,” and “count” are clues, not proofs.

---

## 19. Interview Practice Strategy

For each DP problem, write down these six things before coding:

1. **Pattern:** What structure does the problem have?
2. **State:** What does `dp[...]` mean?
3. **Transition:** How is the current state derived?
4. **Base cases:** What are the smallest known answers?
5. **Order:** In what order should states be calculated?
6. **Complexity:** How many states are there, and how much work does each state perform?

Then solve in this order:

- Write the recursive solution.
- Add memoization.
- Convert it to tabulation.
- Optimize space if the recurrence allows it.

This progression helps you explain the reasoning in an interview rather than simply presenting code.

---

## 20. Practice Problems by Pattern

| Pattern | Problems to practise |
|---|---|
| Linear DP | Fibonacci Number, Climbing Stairs |
| Take or Skip | House Robber, House Robber II |
| Grid DP | Unique Paths, Minimum Path Sum |
| 0/1 Knapsack | Subset Sum, Partition Equal Subset Sum |
| Unbounded Knapsack | Coin Change, Combination Sum IV |
| Subsequences | Longest Increasing Subsequence |
| String DP | Longest Common Subsequence, Edit Distance |
| Interval DP | Matrix Chain Multiplication, Burst Balloons |
| State Machine | Best Time to Buy and Sell Stock variants |
| Bitmask DP | Traveling Salesperson Problem |
| Tree DP | House Robber III |
| DAG DP | Longest Path in a DAG |

## Quick Revision

- Recognize the **structure**, not just the wording.
- Define the state before writing the recurrence.
- Identify the available choices at each state.
- Check whether subproblems overlap.
- Choose the correct base cases and dependency order.
- Start with a correct solution before optimizing.
- Always use input constraints to decide whether the complexity is acceptable.

**Most important habit:** When you see a new problem, ask, “What information must I remember about the decisions made so far to solve the remaining problem?” That question often leads directly to the correct DP state.