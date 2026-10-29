# Solving Problems Under Time Pressure

## 1. What Is Time-Pressure Problem Solving?

In coding interviews and online assessments, solving a problem correctly is only one part of the challenge. You must also understand the question, identify an approach, write correct code, and test it within a limited time.

The goal is not to code as quickly as possible. The goal is to **reach a correct, efficient solution without wasting time on unnecessary approaches or debugging**.

A useful process is:

1. Understand the problem.
2. Identify the pattern.
3. Choose an approach.
4. Analyze complexity.
5. Write the code.
6. Test edge cases.
7. Improve the solution if time permits.

---

## 2. Use a Fixed Time Budget

For a typical 30-minute coding problem, use this as a starting point.

| Phase | Time | What to do |
|---|---:|---|
| Understand | 3 min | Read the problem and constraints |
| Plan | 5 min | Identify the pattern and approach |
| Implement | 12 min | Write the solution |
| Test | 5 min | Check examples and edge cases |
| Review | 5 min | Check complexity and correct bugs |

These are guidelines, not strict rules. An unfamiliar hard problem may require more planning, while a familiar easy problem may need much less.

**Important:** Do not spend half the available time repeatedly thinking about possible approaches without writing down a concrete plan.

---

## 3. Step 1: Understand the Problem Before Coding

Before writing code, identify four things:

- **Input:** What data do you receive?
- **Output:** What exactly must you return?
- **Constraints:** How large can the input be?
- **Special conditions:** Are duplicates, negative values, empty inputs, or other edge cases possible?

Rewrite the problem in one sentence.

For example:

Given an array of integers, return the indices of two elements whose sum equals a target.

This tells you:

- Input: an integer array and a target.
- Output: two indices.
- Requirement: return indices, not the values.
- Potential constraint: each element may be used only once.

That last detail matters when designing the solution.

### Questions to ask yourself

1. Can I explain the problem without looking at the statement?
2. Do I know what the output should look like?
3. Do I understand the input constraints?
4. Have I identified any special cases?

If any answer is no, clarify it before coding.

---

## 4. Step 2: Identify the Pattern

Do not try to invent a completely new algorithm for every problem. Look for a familiar structure.

| Problem clue | Pattern to investigate |
|---|---|
| Find a pair with a target sum | Hashing or two pointers |
| Search a sorted array | Binary search |
| Find the longest or shortest contiguous window | Sliding window |
| Find cumulative totals or range sums | Prefix sums |
| Find next greater or smaller elements | Monotonic stack |
| Explore a tree or graph | DFS or BFS |
| Find the top `k` elements | Heap or sorting |
| Count ways or optimize choices with repeated subproblems | Dynamic programming |
| Merge overlapping ranges | Sorting and intervals |
| Generate all possible combinations | Backtracking |

These are clues, not guarantees. The input structure and constraints determine whether a pattern is appropriate.

### A practical recognition method

Ask these questions in order:

1. Is the input sorted?
2. Do I need fast lookups or frequency counts?
3. Is the problem about a contiguous range?
4. Is it about exploring connections?
5. Are there repeated subproblems?
6. Do I need to enumerate all possible choices?

You do not need to identify the perfect pattern immediately. You need a reasonable first approach that you can justify.

---

## 5. Step 3: Start with the Simplest Correct Approach

Suppose you need to find two numbers whose sum equals a target.

A brute-force solution checks every pair.

```python
def two_sum(nums: list[int], target: int) -> list[int]:
    for i in range(len(nums)):
        for j in range(i + 1, len(nums)):
            if nums[i] + nums[j] == target:
                return [i, j]

    return []
```

This approach is easy to understand and takes \(O(n^2)\) time.

If the input is large, hashing can improve the solution:

```python
def two_sum(nums: list[int], target: int) -> list[int]:
    seen = {}

    for i, num in enumerate(nums):
        complement = target - num

        if complement in seen:
            return [seen[complement], i]

        seen[num] = i

    return []
```

The improved solution takes \(O(n)\) expected time and \(O(n)\) extra space.

The important lesson is that brute force can help you understand a problem, but you should evaluate whether its complexity fits the constraints before committing to it.

---

## 6. Step 4: Analyze Constraints Quickly

The input size gives you an important clue about the acceptable algorithm.

| Input size | Common approaches to investigate |
|---|---|
| \(n \leq 20\) | Backtracking, bitmask DP, exhaustive search |
| \(n \leq 100\) | \(O(n^2)\) may be acceptable |
| \(n \leq 1{,}000\) | \(O(n^2)\) may work depending on the time limit |
| \(n \leq 100{,}000\) | Often \(O(n\log n)\) or \(O(n)\) |
| \(n \geq 1{,}000{,}000\) | Usually investigate near-linear approaches |

These are rough heuristics, not guarantees. Actual limits depend on the language, hardware, number of test cases, and work performed inside each operation.

For example, if `n` is 100,000, a nested loop that examines every pair performs roughly five billion pair checks. That is usually too expensive for a standard coding assessment.

---

## 7. Step 5: Write a Short Plan Before Coding

Before implementation, write three things:

- **Approach:** What algorithm will I use?
- **Invariant or reasoning:** Why does it work?
- **Complexity:** What are the expected time and space costs?

Example:

**Problem:** Find the first repeated number in an array.

- Approach: Use a set.
- Reasoning: A number is repeated if it already exists in the set when encountered.
- Time: \(O(n)\) expected.
- Space: \(O(n)\).

Then code:

```python
def first_duplicate(nums: list[int]) -> int | None:
    seen = set()

    for num in nums:
        if num in seen:
            return num

        seen.add(num)

    return None
```

A small amount of planning can prevent much longer debugging sessions.

---

## 8. Step 6: Write Code in Small, Verifiable Parts

While implementing:

1. Use meaningful variable names.
2. Handle base cases explicitly.
3. Keep loops and conditions simple.
4. Avoid unnecessary abstractions.
5. Check the code mentally after each major section.

For example, in binary search, establish the loop invariant and boundary conditions before writing the loop.

```python
def binary_search(nums: list[int], target: int) -> int:
    left, right = 0, len(nums) - 1

    while left <= right:
        mid = left + (right - left) // 2

        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return -1
```

This avoids common mistakes such as incorrect boundary updates or infinite loops.

---

## 9. Step 7: Test Edge Cases Before Submitting

Passing the example is not enough. Check the cases most likely to break your implementation.

| Edge case | What it tests |
|---|---|
| Empty input | Base cases and initialization |
| One element | Minimum valid size |
| Duplicate values | Duplicate handling |
| Negative values | Assumptions about signs |
| Already sorted input | Ordering assumptions |
| Reverse-sorted input | Boundary conditions |
| All values equal | Repeated-value behavior |
| Target not present | Failure return value |
| Very large input | Time and memory efficiency |

You do not need every edge case for every problem. Choose the ones relevant to the requirements.

For a pair-sum problem, for example, test:

- A pair exists.
- No pair exists.
- Two equal values are needed.
- Negative numbers are present.
- The same array index cannot be used twice.

---

## 10. What to Do When You Get Stuck

Getting stuck is normal. The important thing is to use a systematic recovery process.

### If you cannot identify the pattern

- Write a brute-force solution in plain language.
- Identify what makes it slow.
- Ask whether sorting, hashing, a stack, or a DP state could eliminate repeated work.
- Test your idea on a small example.

### If you understand the idea but cannot code it

- Write the steps as pseudocode.
- Define the variables and their meanings.
- Write the base case first.
- Implement one loop or condition at a time.

### If your solution fails test cases

- Check boundary conditions.
- Verify loop limits and updates.
- Check whether the return value matches the required format.
- Trace a small failing input by hand.
- Look for assumptions not guaranteed by the problem.

### If you have spent too long without progress

Take a short pause and change the question you are asking yourself.

Instead of asking, “What is the perfect solution?”, ask:

- What is the simplest correct solution?
- What information am I recalculating?
- What invariant should remain true?
- Which condition causes this input to fail?

If the assessment allows it, move to another problem and return later.

---

## 11. Manage Time Across Multiple Problems

In an assessment with multiple questions, do not automatically solve the questions in the order presented.

A practical strategy:

1. Scan all questions briefly.
2. Identify the easiest and most familiar ones.
3. Solve questions where you can make reliable progress.
4. Return to harder questions.
5. Reserve time for testing and submission.

For example, if a 60-minute assessment has three questions, one possible allocation is:

| Activity | Time |
|---|---:|
| Scan all questions | 3 min |
| First question | 12 min |
| Second question | 15 min |
| Third question | 20 min |
| Testing and review | 10 min |

Adjust this based on the question difficulty and scoring rules. If partial credit is available, a correct partial solution may be more valuable than spending all the remaining time chasing a fully optimized solution.

---

## 12. Common Time-Wasting Habits

Avoid these behaviors:

- Starting to code before understanding the output requirements.
- Repeatedly switching between unrelated approaches.
- Trying to memorize a complete solution instead of understanding its invariant.
- Optimizing code before you have a correct implementation.
- Ignoring constraints and discovering complexity problems at the end.
- Testing only the provided example.
- Rewriting working code unnecessarily.
- Spending excessive time on variable names or formatting.
- Giving up immediately when the first approach fails.

**A good rule:** Once you have a plausible approach, either validate it with a small example or identify a specific reason it cannot work. Do not switch approaches without evidence.

---

## 13. How to Practise for Better Speed

Speed comes primarily from familiarity with patterns, not from typing faster.

For each practice problem:

1. Try solving it independently.
2. Record the time spent understanding and planning.
3. Write the simplest correct solution.
4. Analyze its complexity.
5. Test edge cases.
6. If you get stuck, review a hint before reading the complete solution.
7. After understanding the solution, close it and implement it again.
8. Revisit the problem later without notes.

Maintain a short mistake log.

| Problem | Mistake | Lesson |
|---|---|---|
| Binary search | Incorrect boundary update | Preserve the search invariant |
| Sliding window | Failed to shrink the window correctly | Define the window condition |
| DP | Incorrect base case | Define what each state represents |
| Hashing | Stored the value too late or too early | Check whether the current element can pair with itself |

Review repeated mistakes rather than simply counting how many problems you have completed.

---

## 14. Interview Communication Under Pressure

If you are solving a problem in a live interview, explain your reasoning clearly.

A useful sequence is:

1. Restate the problem briefly.
2. Describe the brute-force approach.
3. Explain its limitation.
4. Present the improved approach.
5. State the expected complexity.
6. Implement the solution.
7. Test it with an example and edge cases.

If you are stuck, communicate what you have established instead of remaining silent.

For example:

“I can solve this by checking every pair in quadratic time. Since the array may be large, I am considering a hash map to check whether the required complement has already appeared.”

This demonstrates structured reasoning even before the final implementation is complete.

---

## 15. Quick Revision Checklist

Before submitting any coding solution, ask:

- [ ] Did I understand the exact output requirement?
- [ ] Did I check the input constraints?
- [ ] Did I select an appropriate algorithm?
- [ ] Can I explain why the approach works?
- [ ] Is the time complexity acceptable?
- [ ] Are the base cases and boundaries correct?
- [ ] Did I test at least one important edge case?
- [ ] Does the code handle the case where no answer exists?
- [ ] Have I avoided unnecessary changes to working code?

## Final Principle

**Correctness first, efficiency second, polish third.**

Under time pressure, the strongest approach is to recognize a familiar pattern, commit to a justified solution, test it systematically, and move forward without wasting time on unnecessary uncertainty.