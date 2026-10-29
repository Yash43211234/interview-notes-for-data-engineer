# Stack

## 1. What Is a Stack?

A **stack** is a linear data structure that follows **LIFO (Last In, First Out)**.

The element inserted last is the first one removed, just like a stack of plates.

Example:

```text
Push 10       [10]
Push 20       [10, 20]
Push 30       [10, 20, 30]
                 ↑
                Top

Pop() removes 30
Stack: [10, 20]
```

A stack allows insertion and removal at one end, called the **top**.

## 2. Main Stack Operations

| Operation | Meaning | Typical time |
|---|---|---|
| `push(x)` | Add an element to the top | O(1) |
| `pop()` | Remove and return the top element | O(1) |
| `peek()` / `top()` | Read the top element without removing it | O(1) |
| `is_empty()` | Check whether the stack is empty | O(1) |
| `size()` | Return the number of elements | O(1) with Python list |

Example:

```python
stack = []

stack.append(10)  # Push
stack.append(20)
stack.append(30)

print(stack[-1])  # Peek: 30

top = stack.pop()  # Remove: 30

print(top)         # 30
print(stack)       # [10, 20]
```

In Python, `append()` and `pop()` at the end of a list are the usual way to implement a stack.

## 3. Implementing a Stack in Python

### Using a list

```python
stack = []

# Push
stack.append(10)
stack.append(20)
stack.append(30)

# Peek
print(stack[-1])  # 30

# Pop
print(stack.pop())  # 30

# Check whether empty
print(len(stack) == 0)  # False
```

**Why use a list?**

- Simple and readable.
- Appending and removing from the end are O(1) amortized.
- Supports direct access to the top using `stack[-1]`.

Avoid `stack.pop(0)` because it removes from the beginning and takes O(n) time.

### A custom stack class

```python
class Stack:
    def __init__(self):
        self.items = []

    def push(self, value):
        self.items.append(value)

    def pop(self):
        if self.is_empty():
            raise IndexError("Stack is empty")
        return self.items.pop()

    def peek(self):
        if self.is_empty():
            raise IndexError("Stack is empty")
        return self.items[-1]

    def is_empty(self):
        return len(self.items) == 0

    def size(self):
        return len(self.items)
```

Usage:

```python
stack = Stack()

stack.push(10)
stack.push(20)

print(stack.peek())  # 20
print(stack.pop())   # 20
print(stack.size())  # 1
```

This implementation checks for an empty stack before performing `pop()` or `peek()`.

## 4. Stack vs Queue

| Feature | Stack | Queue |
|---|---|---|
| Ordering | LIFO | FIFO |
| Insert | At the top | At the rear |
| Remove | From the top | From the front |
| Python implementation | `list` | `collections.deque` |
| Common use | DFS, undo operations | BFS, task scheduling |

Remember:

- **Stack:** Last in, first out.
- **Queue:** First in, first out.

## 5. Common Stack Patterns

Stack problems become easier when you recognize the pattern behind them.

### Pattern 1: Matching Brackets

**Problem:** Determine whether a string containing brackets is valid.

Example:

```text
Input:  "({[]})"
Output: True

Input:  "([)]"
Output: False
```

**Idea:** When an opening bracket appears, push it onto the stack. When a closing bracket appears, it must match the most recent unmatched opening bracket.

```python
def is_valid(s):
    stack = []
    pairs = {
        ")": "(",
        "]": "[",
        "}": "{"
    }

    for char in s:
        if char in "([{":
            stack.append(char)

        else:
            if not stack or stack[-1] != pairs.get(char):
                return False

            stack.pop()

    return len(stack) == 0
```

**Complexity:**
- Time: O(n)
- Auxiliary space: O(n)

**Why does it work?**

The most recently opened bracket must be closed first. This is exactly the LIFO behavior of a stack.

Common mistake: Returning `True` immediately after processing all characters. The stack must also be empty, or unmatched opening brackets may remain.

### Pattern 2: Monotonic Stack

A **monotonic stack** maintains elements in increasing or decreasing order.

It is useful when a problem asks for the next or previous greater or smaller element.

Common problems:
- Next Greater Element
- Next Smaller Element
- Daily Temperatures
- Stock Span
- Largest Rectangle in Histogram

Example: Next Greater Element

Given an array, find the first greater element to the right of each value. Use `-1` if none exists.

```text
Input:  [2, 1, 2, 4, 3]
Output: [4, 2, 4, -1, -1]
```

```python
def next_greater_elements(nums):
    n = len(nums)
    result = [-1] * n
    stack = []  # Store indices

    for i, value in enumerate(nums):
        while stack and nums[stack[-1]] < value:
            j = stack.pop()
            result[j] = value

        stack.append(i)

    return result
```

**Complexity:**
- Time: O(n)
- Auxiliary space: O(n)

Each index is pushed once and popped at most once. Therefore, the nested `while` loop does not make the overall algorithm O(n²).

**Important:** Store indices when you need to update results at specific positions. Store values only when their positions are unnecessary.

For more details, see `monotonic-stack.md`.

### Pattern 3: Expression Evaluation

Stacks are useful for processing mathematical expressions.

Examples:
- Evaluate Reverse Polish Notation.
- Convert infix expressions to postfix.
- Evaluate postfix expressions.
- Check operator precedence and parentheses.

Example: Evaluate Reverse Polish Notation.

```text
Input:  ["2", "1", "+", "3", "*"]
Output: 9

Calculation:
(2 + 1) * 3 = 9
```

```python
def eval_rpn(tokens):
    stack = []

    for token in tokens:
        if token in {"+", "-", "*", "/"}:
            b = stack.pop()
            a = stack.pop()

            if token == "+":
                value = a + b
            elif token == "-":
                value = a - b
            elif token == "*":
                value = a * b
            else:
                value = int(a / b)  # Truncate toward zero

            stack.append(value)

        else:
            stack.append(int(token))

    return stack[-1]
```

**Why pop `b` before `a`?**

For subtraction and division, order matters. If the stack contains `a` followed by `b`, the expression is `a - b`, not `b - a`.

**Complexity:**
- Time: O(n)
- Auxiliary space: O(n)

The input is assumed to be a valid Reverse Polish Notation expression.

### Pattern 4: Stack for Undo and History

Stacks are useful when the latest action must be reversed first.

Examples:
- Undo operations in a text editor.
- Browser history.
- Backtracking.
- Tracking previous states.

A simple undo model:

```python
history = []

history.append("Typed A")
history.append("Typed B")
history.append("Deleted B")

last_action = history.pop()

print(last_action)  # Deleted B
```

Real undo systems often store enough information to reverse each action, rather than only storing its name.

### Pattern 5: Iterative Depth-First Search (DFS)

DFS explores one path deeply before exploring other paths. It can be implemented using recursion or an explicit stack.

```python
def dfs(graph, start):
    stack = [start]
    visited = {start}
    order = []

    while stack:
        node = stack.pop()
        order.append(node)

        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                stack.append(neighbor)

    return order
```

This code marks nodes as visited when pushing them onto the stack, preventing duplicate pushes.

For a graph represented with adjacency lists:
- Time: O(V + E)
- Auxiliary space: O(V)

The traversal order depends on the order of neighbors in the adjacency lists. Iterative DFS may visit nodes in a different order from recursive DFS.

## 6. Important Stack Interview Problems

### Easy

1. Implement a stack using a list.
2. Implement a stack using queues.
3. Reverse a string using a stack.
4. Check for valid parentheses.
5. Remove adjacent duplicates from a string.

### Medium

1. Next Greater Element.
2. Daily Temperatures.
3. Evaluate Reverse Polish Notation.
4. Min Stack.
5. Simplify Path.
6. Decode String.
7. Asteroid Collision.

### Hard

1. Largest Rectangle in Histogram.
2. Maximal Rectangle.
3. Basic Calculator.
4. Trapping Rain Water using a stack.
5. Design a stack that supports retrieving the minimum in O(1).

## 7. Min Stack

A **Min Stack** supports normal stack operations and returns the minimum element in O(1) time.

One approach is to maintain a second stack containing the minimum value at each level.

```python
class MinStack:
    def __init__(self):
        self.stack = []
        self.min_stack = []

    def push(self, value):
        self.stack.append(value)

        if not self.min_stack:
            self.min_stack.append(value)
        else:
            self.min_stack.append(
                min(value, self.min_stack[-1])
            )

    def pop(self):
        if not self.stack:
            raise IndexError("Stack is empty")

        self.min_stack.pop()
        return self.stack.pop()

    def top(self):
        if not self.stack:
            raise IndexError("Stack is empty")
        return self.stack[-1]

    def get_min(self):
        if not self.min_stack:
            raise IndexError("Stack is empty")
        return self.min_stack[-1]
```

Example:

```python
s = MinStack()

s.push(5)
s.push(2)
s.push(7)

print(s.get_min())  # 2

s.pop()

print(s.get_min())  # 2

s.pop()

print(s.get_min())  # 5
```

**Complexity:**
- `push`: O(1)
- `pop`: O(1)
- `top`: O(1)
- `get_min`: O(1)
- Space: O(n)

The minimum stack stores the minimum value up to each corresponding position. When an element is removed, its matching minimum entry is also removed.

## 8. Common Mistakes

1. **Popping an empty stack.** Check whether the stack is empty before removing an element in a custom implementation.
2. **Using the wrong end.** Use `append()` and `pop()` at the same end for a normal Python stack.
3. **Confusing stack and queue.** Stack uses LIFO; queue uses FIFO.
4. **Forgetting unmatched opening brackets.** A valid-bracket solution must finish with an empty stack.
5. **Reversing operand order.** For subtraction and division, the last popped value is the right operand.
6. **Assuming nested loops always mean O(n²).** In monotonic-stack algorithms, each element may be pushed and popped only once, giving O(n) total time.
7. **Storing values when indices are needed.** Indices are useful for updating answers and calculating distances.
8. **Using recursion without considering depth.** Deep recursion can exceed Python's recursion limit; iterative DFS with a stack may be safer.

## 9. Interview Questions to Test Yourself

Before looking at a solution, explain:

1. Why does a stack follow LIFO?
2. Why are `append()` and `pop()` efficient at the end of a Python list?
3. How does a stack help validate nested brackets?
4. Why does Next Greater Element have an O(n) solution?
5. What is the difference between a normal stack and a monotonic stack?
6. Why does iterative DFS use a stack?
7. How can you retrieve the minimum element in O(1)?
8. What is the difference between a stack and a recursive call stack?

## 10. Quick Revision

- **Stack:** LIFO — last in, first out.
- **Push:** `stack.append(x)`.
- **Pop:** `stack.pop()`.
- **Peek:** `stack[-1]`, after checking that the stack is not empty.
- **Typical operation time:** O(1) amortized for push and pop.
- **Valid parentheses:** Push opening brackets; match closing brackets against the top.
- **Monotonic stack:** Maintains an ordering to efficiently find next or previous greater/smaller elements.
- **DFS:** Uses a stack to explore graph paths.
- **Min Stack:** Uses an auxiliary stack to retrieve the minimum in O(1).
- **Expression evaluation:** Operand order matters for subtraction and division.

**Interview takeaway:** Learn the basic stack operations first, then master valid parentheses, monotonic-stack problems, and iterative DFS. Recognizing when the most recent unresolved element must be processed first is the key signal that a stack may be the right data structure.