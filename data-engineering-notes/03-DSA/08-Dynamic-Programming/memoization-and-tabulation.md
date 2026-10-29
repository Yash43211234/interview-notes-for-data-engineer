# Memoization and Tabulation

## 1. What Are Memoization and Tabulation?

Memoization and tabulation are two techniques used in **Dynamic Programming (DP)** to avoid solving the same subproblem multiple times.

Both improve efficiency by storing previously calculated results and reusing them.

The main difference is how they calculate and store results:

| Feature | Memoization | Tabulation |
|---|---|---|
| Approach | Top-down | Bottom-up |
| Technique | Recursion + cache | Iteration + table |
| Starting point | Original problem | Smallest subproblems |
| Calculation order | Only required states are calculated | Usually calculates all states in the table |
| Recursion | Yes | No |
| Stack usage | Uses recursion stack | Usually avoids recursion stack |
| Common risk | Recursion depth limit | Incorrect iteration order |
| Best suited for | Problems naturally expressed recursively | Problems with a clear iterative order |

**Remember:**

- Memoization = Recursion + Caching.
- Tabulation = Iteration + Table.

---

## 2. Example Problem: Fibonacci Numbers

The Fibonacci sequence is defined as:

\[
F(n)=
\begin{cases}
0 & n=0\\
1 & n=1\\
F(n-1)+F(n-2) & n\geq 2
\end{cases}
\]

The sequence is:

`0, 1, 1, 2, 3, 5, 8, 13, 21, ...`

For example:

\[
F(5)=F(4)+F(3)=5
\]

A naive recursive solution repeatedly calculates the same values.

### Naive recursion

```python
def fib(n: int) -> int:
    if n <= 1:
        return n

    return fib(n - 1) + fib(n - 2)
```

**Complexity:**

- Time: \(O(2^n)\) as a common upper-bound estimate.
- Space: \(O(n)\) for the recursion stack.

The problem is repeated computation. For example, `fib(3)` is calculated multiple times while computing `fib(5)`.

---

## 3. Memoization: Top-Down DP

Memoization stores the answer for each subproblem after calculating it for the first time.

When the same subproblem appears again, the stored result is returned instead of calculating it again.

### Python implementation

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n: int) -> int:
    if n <= 1:
        return n

    return fib(n - 1) + fib(n - 2)


print(fib(5))  # 5
```

### How it works

For `fib(5)`:

1. Calculate `fib(5)`.
2. Recursively calculate `fib(4)` and `fib(3)`.
3. Store each computed result in the cache.
4. If a previously calculated value is requested again, return it from the cache.

Each Fibonacci state is calculated only once.

### Memoization without a decorator

```python
def fib(n: int) -> int:
    memo = {}

    def solve(k: int) -> int:
        if k <= 1:
            return k

        if k in memo:
            return memo[k]

        memo[k] = solve(k - 1) + solve(k - 2)
        return memo[k]

    return solve(n)


print(fib(5))  # 5
```

Here:

- `memo` stores calculated answers.
- `k in memo` checks whether the answer already exists.
- `memo[k] = ...` saves the result.
- `solve(k)` recursively solves the smaller problem.

**Important:** The cache must be shared across recursive calls. If a new cache is created inside every recursive call, the repeated-work problem will not be solved correctly.

### Complexity

For Fibonacci:

- Time: \(O(n)\)
- Space: \(O(n)\) for the cache and recursion stack.

Memoization is especially useful when the original problem is naturally recursive or when only a subset of possible states needs to be evaluated.

---

## 4. Tabulation: Bottom-Up DP

Tabulation starts with the smallest subproblems and builds larger answers using an iterative table.

Instead of asking for `fib(5)` and recursively breaking it down, we calculate:

`fib(0) → fib(1) → fib(2) → fib(3) → fib(4) → fib(5)`

### Python implementation

```python
def fib(n: int) -> int:
    if n <= 1:
        return n

    dp = [0] * (n + 1)

    dp[0] = 0
    dp[1] = 1

    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]

    return dp[n]


print(fib(5))  # 5
```

### How the table is built

For `n = 5`:

| `i` | `dp[i - 1]` | `dp[i - 2]` | `dp[i]` |
|---:|---:|---:|---:|
| 2 | 1 | 0 | 1 |
| 3 | 1 | 1 | 2 |
| 4 | 2 | 1 | 3 |
| 5 | 3 | 2 | 5 |

The final answer is `dp[5] = 5`.

### Complexity

- Time: \(O(n)\)
- Space: \(O(n)\) for the DP table.

Tabulation avoids recursion and makes the calculation order explicit.

---

## 5. Space Optimization

Sometimes the current answer depends only on a few previous states. In that case, we do not need to store the entire DP table.

For Fibonacci, each answer depends only on the previous two values.

```python
def fib(n: int) -> int:
    if n <= 1:
        return n

    prev2 = 0
    prev1 = 1

    for _ in range(2, n + 1):
        current = prev1 + prev2
        prev2 = prev1
        prev1 = current

    return prev1


print(fib(5))  # 5
```

**Complexity:**

- Time: \(O(n)\)
- Auxiliary space: \(O(1)\)

This is a space-optimized tabulation solution.

The optimization works because the next state needs only the two previous values, not the complete sequence.

---

## 6. How to Convert Memoization into Tabulation

A reliable conversion process is:

1. Identify the DP state and what it represents.
2. Write the recurrence relation.
3. Identify the base cases.
4. Determine which states depend on other states.
5. Calculate the states in an order where dependencies are already available.
6. Return the state representing the original problem.
7. Check whether the table can be space-optimized.

### Example: Climbing Stairs

You can climb either one or two steps at a time. Find the number of distinct ways to reach step `n`.

The recurrence is:

\[
dp[n]=dp[n-1]+dp[n-2]
\]

Base cases:

\[
dp[0]=1,\quad dp[1]=1
\]

Here, `dp[0] = 1` represents one way to stay at the starting position: take no steps.

#### Memoization

```python
def climb_stairs(n: int) -> int:
    memo = {}

    def solve(i: int) -> int:
        if i <= 1:
            return 1

        if i in memo:
            return memo[i]

        memo[i] = solve(i - 1) + solve(i - 2)
        return memo[i]

    return solve(n)
```

#### Tabulation

```python
def climb_stairs(n: int) -> int:
    if n <= 1:
        return 1

    dp = [0] * (n + 1)
    dp[0] = 1
    dp[1] = 1

    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]

    return dp[n]
```

Both return `8` for `n = 5`.

---

## 7. When Should You Use Each Technique?

### Prefer memoization when:

- You can express the problem naturally using recursion.
- You want to derive a solution from a recursive recurrence.
- The state space is large, but only some states may be reached.
- You want to start with a recursive solution and add caching.

### Prefer tabulation when:

- The dependency order is clear.
- You want to avoid recursion depth problems.
- You need predictable iterative execution.
- You want to optimize memory by retaining only necessary previous states.

Neither approach is always faster. Actual performance depends on the number of states, recursion overhead, memory usage, and implementation details.

---

## 8. Common Mistakes

### Mistake 1: Forgetting to cache a result

```python
if n in memo:
    return memo[n]

return solve(n - 1) + solve(n - 2)
```

The result is returned but never stored. Repeated calls may still recalculate the same states.

Correct:

```python
memo[n] = solve(n - 1) + solve(n - 2)
return memo[n]
```

### Mistake 2: Incorrect base cases

If the base cases do not match the recurrence, every result may be incorrect.

Always define what the smallest state means before writing the recurrence.

### Mistake 3: Incorrect iteration order

```python
for i in range(n, 1, -1):
    dp[i] = dp[i - 1] + dp[i - 2]
```

This descending order is incorrect for the standard Fibonacci tabulation because the smaller states may not have been calculated yet.

Calculate from smaller states to larger states when using this recurrence.

### Mistake 4: Confusing the two base-case conventions

For climbing stairs, `dp[0] = 1` is correct because there is one way to take zero steps. For Fibonacci, `fib(0) = 0`.

Base cases depend on the meaning of the state, not on a universal DP rule.

### Mistake 5: Assuming every DP problem can use constant space

Space optimization is possible only when the recurrence allows old states to be discarded safely. Problems that need previous states for reconstruction may require a full table or additional storage.

### Mistake 6: Ignoring recursion limits

Memoization still uses recursion. In Python, a deep recursive call chain may raise `RecursionError`. Tabulation can avoid this issue when an iterative solution is possible.

---

## 9. Interview Questions

**Q1. What is the difference between memoization and tabulation?**

Memoization is top-down DP that uses recursion and a cache. Tabulation is bottom-up DP that uses iteration to fill a table in dependency order.

**Q2. Do both techniques improve time complexity?**

They can. When overlapping subproblems are present, both can reduce repeated calculations. For Fibonacci, both reduce the time from exponential to \(O(n)\).

**Q3. Is memoization always better because it calculates only required states?**

No. It may avoid unnecessary states, but recursion and cache lookups introduce overhead. Tabulation may be simpler and faster when most states are required.

**Q4. Does tabulation always use less space?**

No. A full table can use \(O(n)\) or more space, just like memoization. However, tabulation can sometimes be optimized to \(O(1)\) auxiliary space, as in Fibonacci.

**Q5. Can memoization be used without a dictionary?**

Yes. A list or array can be used when the state space is indexed by integers. `functools.lru_cache` is another option.

**Q6. What is the time complexity of memoized Fibonacci?**

\(O(n)\), because each state from `0` through `n` is calculated at most once, assuming constant-time arithmetic operations.

**Q7. What is the most important step when designing tabulation?**

Determine the correct calculation order so every dependency is available before its dependent state is calculated.

---

## 10. Practice Problems

Practice these in order:

1. **Fibonacci Number** — implement memoization, tabulation, and space optimization.
2. **Climbing Stairs** — compare recursive and iterative DP.
3. **Min Cost Climbing Stairs** — define states and base cases carefully.
4. **House Robber** — learn how to choose between previous states.
5. **Coin Change** — practise DP with multiple possible transitions.
6. **Longest Common Subsequence** — practise two-dimensional tabulation.
7. **0/1 Knapsack** — understand state dependencies and iteration order.

---

## Quick Revision

- **Memoization:** Top-down, recursion, cache.
- **Tabulation:** Bottom-up, iteration, DP table.
- Both avoid repeated computation by reusing stored subproblem results.
- Define the state, recurrence, and base cases before coding.
- Calculate states in dependency order.
- Optimize space only when discarded states are no longer needed.
- For interview preparation, first understand the recursive recurrence, then add memoization, then convert it to tabulation, and finally consider space optimization.