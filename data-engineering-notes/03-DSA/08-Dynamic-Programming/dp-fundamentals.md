# Dynamic Programming Fundamentals

## 1. What Is Dynamic Programming?

**Dynamic Programming (DP)** is a problem-solving technique that solves a complex problem by breaking it into smaller, overlapping subproblems and storing their results so they do not need to be calculated repeatedly.

DP is especially useful when:
- The same subproblems appear multiple times.
- The problem can be solved using previously computed smaller results.
- You need to optimize a solution or count the number of possible ways to achieve something.

Common examples include Fibonacci numbers, climbing stairs, coin change, knapsack, and longest common subsequence.

### Example: Fibonacci Numbers

The Fibonacci sequence is defined as:

\[
F(n)=F(n-1)+F(n-2)
\]

with base cases:

\[
F(0)=0,\qquad F(1)=1
\]

For example:

```text
F(0) = 0
F(1) = 1
F(2) = 1
F(3) = 2
F(4) = 3
F(5) = 5
```

A direct recursive solution repeatedly calculates the same values. Dynamic programming stores each result and reuses it.

---

## 2. Why Do We Need DP?

Consider this recursive Fibonacci implementation:

```python
def fib(n):
    if n <= 1:
        return n

    return fib(n - 1) + fib(n - 2)
```

For `fib(5)`, the function calculates `fib(3)` multiple times, and the same repetition increases rapidly as `n` grows.

The time complexity is exponential, approximately \(O(2^n)\).

We can improve it by remembering previously calculated results.

### Memoized Version

```python
def fib(n, memo=None):
    if memo is None:
        memo = {}

    if n <= 1:
        return n

    if n in memo:
        return memo[n]

    memo[n] = fib(n - 1, memo) + fib(n - 2, memo)

    return memo[n]
```

Now each Fibonacci value is calculated only once.

- Time complexity: \(O(n)\)
- Space complexity: \(O(n)\), including the cache and recursion stack.

This is the central idea behind DP: **avoid solving the same subproblem repeatedly.**

---

## 3. Two Important Properties of DP

Not every recursive problem benefits from dynamic programming. Two properties help identify suitable problems.

### 3.1 Overlapping Subproblems

A problem has overlapping subproblems when the same smaller problems are solved repeatedly.

For Fibonacci:

```text
                fib(5)
               /      \
           fib(4)     fib(3)
           /   \       /   \
       fib(3) fib(2) fib(2) fib(1)
```

Notice that `fib(3)` and `fib(2)` appear multiple times.

Instead of calculating them repeatedly, DP calculates each distinct state once and stores the result.

### 3.2 Optimal Substructure

A problem has optimal substructure when its optimal solution can be constructed from optimal solutions to smaller subproblems.

For example, in the minimum-cost climbing stairs problem, the minimum cost to reach a step depends on the minimum costs of reaching earlier steps.

This property is common in optimization problems, but DP is also used for counting and feasibility problems.

**Important:** Overlapping subproblems and optimal substructure are useful clues, not a guarantee that every problem should be solved with DP. You must still define correct states and transitions.

---

## 4. The Two Main Approaches to DP

Dynamic programming is commonly implemented using memoization or tabulation.

### 4.1 Top-Down DP: Memoization

Top-down DP starts with the original problem and recursively solves smaller subproblems. The result of each state is cached.

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    if n <= 1:
        return n

    return fib(n - 1) + fib(n - 2)


print(fib(6))  # 8
```

**How it works:**

1. Call the function for the required state.
2. Check whether the result is already cached.
3. If cached, return it.
4. Otherwise, solve the smaller states.
5. Store and return the result.

Advantages:
- Often easy to derive from a recursive solution.
- Computes only the states that are actually needed.

Disadvantages:
- Uses recursion stack space.
- Deep recursion can exceed Python's recursion limit.

### 4.2 Bottom-Up DP: Tabulation

Bottom-up DP calculates smaller states first and builds toward the final answer using a table.

```python
def fib(n):
    if n <= 1:
        return n

    dp = [0] * (n + 1)
    dp[1] = 1

    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]

    return dp[n]


print(fib(6))  # 8
```

**How it works:**

1. Define the DP table.
2. Initialize the base cases.
3. Calculate each state in a valid order.
4. Return the required state.

Advantages:
- Avoids recursive call-stack overhead.
- Often makes time and space usage easier to control.

Disadvantages:
- May calculate states that are never needed.
- Requires a correct order of computation.

### Comparison

| Feature | Memoization | Tabulation |
|---|---|---|
| Approach | Top-down | Bottom-up |
| Main tool | Recursion and cache | Loop and DP table |
| Computation order | Driven by recursive calls | Explicitly chosen |
| Recursion stack | Required | Not required |
| Calculates only needed states | Often | Not always |
| Typical space | Cache plus recursion stack | DP table |

Both approaches can have the same time complexity when they compute the same states.

---

## 5. The Most Important DP Skill: State Definition

The **state** describes the smaller problem whose answer you want to calculate.

A good state definition should make it clear what each DP value represents.

For Fibonacci:

```python
dp[i] = Fibonacci number at index i
```

For climbing stairs:

```python
dp[i] = Number of distinct ways to reach step i
```

For minimum-cost problems:

```python
dp[i] = Minimum cost required to reach state i
```

For knapsack:

```python
dp[i][w] = Maximum value using the first i items
            with capacity w
```

The exact meaning depends on the problem.

### Why Is State Definition Important?

If the state is unclear, it becomes difficult to determine:
- Which smaller states are needed.
- What the recurrence should be.
- Which values form the base cases.
- What the final answer represents.

Before writing code, explain your state in one sentence.

---

## 6. Recurrence Relation

A **recurrence relation** describes how to calculate a state from previously solved states.

For Fibonacci:

\[
dp[i]=dp[i-1]+dp[i-2]
\]

For climbing stairs, if you can climb one or two steps at a time:

\[
dp[i]=dp[i-1]+dp[i-2]
\]

For a minimum-cost transition:

\[
dp[i]=\min(dp[i-1],dp[i-2])+cost[i]
\]

The recurrence is not always the same. It depends on the choices allowed by the problem.

### A Simple Way to Derive a Recurrence

Ask:

1. What choices can I make at the current state?
2. What smaller state results from each choice?
3. How should I combine the answers?

For counting problems, the answers for valid choices are often added.

For minimization problems, take the minimum over valid choices.

For maximization problems, take the maximum over valid choices.

For feasibility problems, the choices are often combined using logical OR.

These are common patterns, not universal rules; the exact recurrence must follow the problem's requirements.

---

## 7. Base Cases

Base cases are the smallest states whose answers are known without solving a smaller problem.

For Fibonacci:

```python
dp[0] = 0
dp[1] = 1
```

For climbing stairs, if `dp[i]` counts ways to reach step `i`:

```python
dp[0] = 1
```

Here, `dp[0] = 1` means there is one way to remain at the starting position: take no steps.

This convention makes the recurrence work naturally for the first step.

### Common Base-Case Mistakes

- Forgetting the smallest valid input.
- Using inconsistent indexing.
- Initializing a minimum-cost state with zero when it should be impossible or unknown.
- Handling `n = 0` incorrectly.
- Applying a recurrence before all required smaller states are initialized.

Always test the smallest inputs before testing larger ones.

---

## 8. A Complete DP Example: Climbing Stairs

### Problem

You are climbing a staircase with `n` steps. At each move, you can climb either one or two steps.

Return the number of distinct ways to reach step `n`.

### Step 1: Define the State

```python
dp[i] = Number of ways to reach step i
```

### Step 2: Identify the Choices

To reach step `i`, the previous step could have been:
- Step `i - 1`, followed by one step.
- Step `i - 2`, followed by two steps.

### Step 3: Write the Recurrence

\[
dp[i]=dp[i-1]+dp[i-2]
\]

### Step 4: Define the Base Cases

\[
dp[0]=1,\qquad dp[1]=1
\]

### Step 5: Implement It

```python
def climb_stairs(n):
    if n <= 1:
        return 1

    dp = [0] * (n + 1)
    dp[0] = 1
    dp[1] = 1

    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]

    return dp[n]


print(climb_stairs(4))  # 5
```

For `n = 4`, the five possible step sequences are:

```text
1 + 1 + 1 + 1
1 + 1 + 2
1 + 2 + 1
2 + 1 + 1
2 + 2
```

Complexity:
- Time: \(O(n)\)
- Space: \(O(n)\)

---

## 9. Space Optimization

Sometimes the current state depends only on a small number of previous states. In that case, storing the entire DP table is unnecessary.

For Fibonacci, only the previous two values are needed.

```python
def fib(n):
    if n <= 1:
        return n

    prev2 = 0
    prev1 = 1

    for _ in range(2, n + 1):
        current = prev1 + prev2
        prev2 = prev1
        prev1 = current

    return prev1
```

Complexity:
- Time: \(O(n)\)
- Auxiliary space: \(O(1)\)

This technique is called **space optimization**.

### When Is Space Optimization Possible?

It is useful when:
- The recurrence depends only on a fixed number of previous states.
- Earlier states are not needed later.
- You can update variables without overwriting values that are still required.

Do not optimize space before the full DP solution is correct. First build a clear recurrence and table, then identify what can be safely discarded.

---

## 10. Common DP Problem Categories

| Category | Typical problem | Common state |
|---|---|---|
| One-dimensional DP | Climbing Stairs | Position |
| Grid DP | Unique Paths | Row and column |
| Knapsack DP | 0/1 Knapsack | Item index and remaining capacity |
| Subsequence DP | Longest Increasing Subsequence | Ending index or sequence state |
| String DP | Longest Common Subsequence | Two string indices |
| Interval DP | Matrix Chain Multiplication | Left and right boundaries |
| Tree DP | Maximum path or subtree value | Node and relevant state |
| Bitmask DP | Traveling Salesperson | Visited-set mask and current vertex |
| DP on DAGs | Longest path in a DAG | Vertex or dependency state |
| State-machine DP | Stock Buy and Sell | Day and holding state |

Learn one category at a time. The state dimensions and transitions become more complex as you move from simple one-dimensional DP to two-dimensional and state-based problems.

---

## 11. DP vs. Greedy vs. Recursion

### Recursion

Recursion solves a problem by calling a function on smaller inputs.

It may repeatedly solve the same subproblem unless results are cached.

### Dynamic Programming

DP stores and reuses answers to repeated subproblems.

It is useful when the problem's structure supports such reuse.

### Greedy

A greedy algorithm makes a locally preferred choice at each step.

It can be more efficient than DP, but it is correct only when the problem supports the greedy-choice property.

| Feature | Recursion | Dynamic programming | Greedy |
|---|---|---|---|
| Reuses computed results | Not automatically | Yes | Not necessarily |
| Explores multiple choices | Often | Often, through transitions | Usually commits to a choice |
| Guarantees an optimal answer | No | If recurrence is correct | Only when greedy choice is justified |
| Common implementation | Function calls | Cache or table | Selection rules |

Do not assume that every optimization problem requires DP. Some problems have a correct greedy solution, while others need DP or a different algorithm.

---

## 12. How to Recognize a DP Problem

Look for statements such as:

- Find the minimum or maximum cost.
- Count the number of ways.
- Determine whether a target is achievable.
- Find the longest or shortest valid sequence.
- Choose items under a constraint.
- Make decisions across multiple steps.
- Solve a problem where the same smaller states recur.

Then ask:

1. Can I define a state that represents a smaller version of the problem?
2. Can I derive its answer from smaller states?
3. Are those states reused?
4. Can I identify valid base cases?
5. What is the order in which states must be calculated?

If these questions lead to a consistent recurrence, DP may be a good fit.

---

## 13. Common Mistakes

1. **Starting with code instead of the state:** Define exactly what `dp[i]` or `dp[i][j]` means.
2. **Writing an incorrect recurrence:** Consider every valid choice and avoid double-counting.
3. **Using incorrect base cases:** Test the smallest inputs.
4. **Mixing indexing conventions:** Decide whether your states begin at zero or one.
5. **Confusing counting with optimization:** Counting often uses sums, while optimization often uses `min()` or `max()`.
6. **Forgetting impossible states:** Use a suitable sentinel, such as infinity, when appropriate.
7. **Optimizing space too early:** First verify the full-table solution.
8. **Assuming every recursive problem is DP:** DP is especially useful when subproblems overlap and stored results can be reused.
9. **Ignoring transition order:** Bottom-up computation must calculate dependencies before the states that use them.
10. **Failing to test edge cases:** Try the smallest inputs and boundary conditions.

---

## 14. Interview Problem-Solving Framework

Use this sequence when solving a DP problem:

**Step 1 — Define the state.**

Write one sentence explaining what each state represents.

**Step 2 — Identify the choices.**

Determine all possible actions or previous states.

**Step 3 — Derive the recurrence.**

Express the current answer using smaller states.

**Step 4 — Establish base cases.**

Determine the values that can be answered directly.

**Step 5 — Choose the implementation.**

Use memoization for a top-down solution or tabulation for a bottom-up solution.

**Step 6 — Analyze complexity.**

Count the number of distinct states and the work needed for each state.

For example, if there are \(n\) states and each state takes constant time:

\[
T(n)=O(n)
\]

If there are \(n \times m\) states and each takes constant time:

\[
T(n,m)=O(nm)
\]

**Step 7 — Consider optimization.**

Determine whether the full table or recursion stack can be reduced.

**Step 8 — Test edge cases.**

Check empty inputs where valid, minimum sizes, repeated values, and boundary conditions.

---

## 15. Practice Problems

Practice in this order:

### Beginner

1. Fibonacci Number
2. Climbing Stairs
3. Min Cost Climbing Stairs
4. House Robber
5. Maximum Sum of Non-Adjacent Elements

### Intermediate

6. Unique Paths
7. Minimum Path Sum
8. Coin Change
9. Partition Equal Subset Sum
10. Longest Increasing Subsequence
11. Longest Common Subsequence
12. Word Break

### Advanced

13. 0/1 Knapsack
14. Edit Distance
15. Matrix Chain Multiplication
16. Longest Palindromic Subsequence
17. Best Time to Buy and Sell Stock with Constraints
18. Traveling Salesperson Problem

For each problem, write the state definition, recurrence, base cases, time complexity, and space complexity before implementing the solution.

---

## 16. Quick Revision

- **Dynamic programming:** Solve subproblems and store their results.
- **Overlapping subproblems:** The same smaller problems recur.
- **Optimal substructure:** An optimal solution can be constructed from suitable optimal subproblem solutions.
- **Memoization:** Top-down recursion with caching.
- **Tabulation:** Bottom-up computation using a table.
- **State:** What a DP entry represents.
- **Recurrence:** How to calculate a state from smaller states.
- **Base case:** A state whose answer is known directly.
- **Space optimization:** Keep only the values needed for future transitions.
- **Time complexity:** Usually the number of states multiplied by the work per state.

**Recommended learning order:** Recursion → memoization → tabulation → space optimization → one-dimensional DP → grid DP → knapsack → subsequence DP → interval DP → tree and bitmask DP.