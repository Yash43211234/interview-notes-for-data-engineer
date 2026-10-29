# Tree Traversals in Python

## 1. What Is Tree Traversal?

**Tree traversal** is the process of visiting every node in a tree in a specific order.

For binary trees, the main traversal methods are:

1. **Preorder:** Root → Left → Right
2. **Inorder:** Left → Root → Right
3. **Postorder:** Left → Right → Root
4. **Level Order:** Visit nodes level by level, from left to right

The first three are depth-first search (DFS) traversals. Level order is breadth-first search (BFS).

## 2. Example Binary Tree

We will use this tree for all the examples.

```text
          1
         / \
        2   3
       / \   \
      4   5   6
```

- Root: `1`
- Left subtree: `2`, `4`, `5`
- Right subtree: `3`, `6`
- Leaf nodes: `4`, `5`, `6`

## 3. Preorder Traversal

**Order: Root → Left → Right**

Visit the current node first, then traverse the left subtree, and finally traverse the right subtree.

### Step-by-step

For the example tree:

1. Visit `1`.
2. Traverse the left subtree: visit `2`, then `4`, then `5`.
3. Traverse the right subtree: visit `3`, then `6`.

Output:

```python
[1, 2, 4, 5, 3, 6]
```

### Recursive implementation

```python
def preorder(root):
    result = []

    def dfs(node):
        if node is None:
            return

        result.append(node.val)  # Root
        dfs(node.left)           # Left
        dfs(node.right)          # Right

    dfs(root)
    return result
```

### When is preorder useful?

- Copying or serializing a tree.
- Creating a prefix expression.
- Processing a parent before its children.
- Constructing a BST from preorder traversal when the required rules are known.

## 4. Inorder Traversal

**Order: Left → Root → Right**

Traverse the left subtree first, visit the current node, and then traverse the right subtree.

### Step-by-step

1. Traverse the left subtree of `1`: visit `4`, `2`, `5`.
2. Visit `1`.
3. Traverse the right subtree: visit `3`, `6`.

Output:

```python
[4, 2, 5, 1, 3, 6]
```

### Recursive implementation

```python
def inorder(root):
    result = []

    def dfs(node):
        if node is None:
            return

        dfs(node.left)            # Left
        result.append(node.val)   # Root
        dfs(node.right)           # Right

    dfs(root)
    return result
```

### When is inorder useful?

- Retrieving values in ascending order from a BST.
- Checking whether a BST's values are ordered correctly.
- Finding the kth smallest value in a BST.

**Important:** Inorder traversal produces sorted values only when the tree satisfies the BST ordering property. An arbitrary binary tree does not guarantee sorted output.

## 5. Postorder Traversal

**Order: Left → Right → Root**

Traverse both child subtrees before processing the current node.

### Step-by-step

1. Traverse the left subtree: visit `4`, `5`, then `2`.
2. Traverse the right subtree: visit `6`, then `3`.
3. Visit the root `1`.

Output:

```python
[4, 5, 2, 6, 3, 1]
```

### Recursive implementation

```python
def postorder(root):
    result = []

    def dfs(node):
        if node is None:
            return

        dfs(node.left)           # Left
        dfs(node.right)          # Right
        result.append(node.val)  # Root

    dfs(root)
    return result
```

### When is postorder useful?

- Deleting a tree because children are processed before their parent.
- Calculating subtree properties.
- Evaluating expression trees.
- Calculating the height of a binary tree.

## 6. Level Order Traversal (BFS)

Level order visits nodes one level at a time, from left to right.

For our example:

```text
Level 0:       1
Level 1:      2 3
Level 2:     4 5 6
```

Output:

```python
[1, 2, 3, 4, 5, 6]
```

### Why use a queue?

A queue follows FIFO: First In, First Out.

Nodes discovered earlier are processed before nodes discovered later. This naturally processes the tree level by level.

### Implementation

```python
from collections import deque

def level_order(root):
    if root is None:
        return []

    result = []
    queue = deque([root])

    while queue:
        node = queue.popleft()
        result.append(node.val)

        if node.left is not None:
            queue.append(node.left)

        if node.right is not None:
            queue.append(node.right)

    return result
```

### Complexity

- Time: `O(n)`
- Auxiliary space: `O(w)`, where `w` is the maximum width of the tree. In the worst case, this can be `O(n)`.

### Level-by-level output

Some problems require a separate list for each level.

```python
from collections import deque

def level_order_by_level(root):
    if root is None:
        return []

    result = []
    queue = deque([root])

    while queue:
        level_size = len(queue)
        level = []

        for _ in range(level_size):
            node = queue.popleft()
            level.append(node.val)

            if node.left is not None:
                queue.append(node.left)

            if node.right is not None:
                queue.append(node.right)

        result.append(level)

    return result
```

Output:

```python
[
    [1],
    [2, 3],
    [4, 5, 6]
]
```

**Key idea:** Capture `level_size` before processing the level. This prevents newly added children from being included in the current level.

## 7. DFS vs. BFS

| Feature | DFS | BFS |
|---|---|---|
| Main methods | Preorder, inorder, postorder | Level order |
| Common structure | Recursion or stack | Queue |
| Traversal style | Explore a branch deeply | Explore one level at a time |
| Typical applications | Paths, subtrees, tree properties | Levels, minimum depth |
| Time complexity | `O(n)` | `O(n)` |
| Auxiliary space | `O(h)` recursively | `O(w)` for the queue |

Here, `n` is the number of nodes, `h` is tree height, and `w` is maximum tree width.

## 8. Iterative DFS Traversals

Recursive traversal is easy to understand, but iterative traversal uses an explicit stack. It can help avoid Python's recursion-depth limit for very deep trees.

### Iterative preorder

Push the right child first so the left child is processed first.

```python
def preorder_iterative(root):
    if root is None:
        return []

    result = []
    stack = [root]

    while stack:
        node = stack.pop()
        result.append(node.val)

        if node.right is not None:
            stack.append(node.right)

        if node.left is not None:
            stack.append(node.left)

    return result
```

### Iterative inorder

Continue going left while storing ancestors on the stack. Once there is no left child, visit the top node and move to its right subtree.

```python
def inorder_iterative(root):
    result = []
    stack = []
    current = root

    while current is not None or stack:
        while current is not None:
            stack.append(current)
            current = current.left

        current = stack.pop()
        result.append(current.val)
        current = current.right

    return result
```

### Iterative postorder using two stacks

The first stack processes nodes; the second stores them in an order that can be reversed into postorder.

```python
def postorder_iterative(root):
    if root is None:
        return []

    stack = [root]
    result = []

    while stack:
        node = stack.pop()
        result.append(node.val)

        if node.left is not None:
            stack.append(node.left)

        if node.right is not None:
            stack.append(node.right)

    result.reverse()
    return result
```

Each iterative DFS method takes `O(n)` time and up to `O(h)` auxiliary space for these implementations.

## 9. Morris Inorder Traversal

Morris traversal performs inorder traversal using `O(1)` auxiliary space, excluding the output list.

It temporarily modifies right pointers to create links back to ancestors, then restores them.

```python
def morris_inorder(root):
    result = []
    current = root

    while current is not None:
        if current.left is None:
            result.append(current.val)
            current = current.right
        else:
            predecessor = current.left

            while (
                predecessor.right is not None
                and predecessor.right is not current
            ):
                predecessor = predecessor.right

            if predecessor.right is None:
                predecessor.right = current
                current = current.left
            else:
                predecessor.right = None
                result.append(current.val)
                current = current.right

    return result
```

- Time: `O(n)`
- Auxiliary space: `O(1)`, excluding output.

Morris traversal is an advanced technique. Use ordinary recursive or iterative traversal unless constant auxiliary space is specifically required. It temporarily changes the tree, so the temporary links must be restored correctly.

## 10. Common Traversal Problems

| Problem | Useful traversal or technique |
|---|---|
| Maximum Depth of Binary Tree | DFS |
| Invert Binary Tree | DFS or BFS |
| Same Tree | DFS |
| Symmetric Tree | Recursive DFS or BFS |
| Level Order Traversal | BFS |
| Zigzag Level Order Traversal | BFS |
| Right Side View | BFS or DFS |
| Minimum Depth of Binary Tree | BFS is often convenient |
| Validate Binary Search Tree | Inorder or bounds-based DFS |
| Kth Smallest Element in a BST | Inorder |
| Path Sum | DFS |
| Binary Tree Level Averages | BFS |
| Serialize and Deserialize Binary Tree | Preorder or level order with null markers |

## 11. Common Mistakes

1. **Using the wrong order:** Preorder visits the root first; postorder visits it last.
2. **Forgetting the base case:** Return when `node is None`.
3. **Using a list as a queue with `pop(0)`:** This can make queue processing inefficient. Prefer `collections.deque`.
4. **Forgetting to copy a recursive path:** Use `path.copy()` when saving a mutable path in backtracking problems.
5. **Mixing levels in BFS:** Save the queue length before processing each level.
6. **Assuming every inorder result is sorted:** This is true for BSTs, not arbitrary binary trees.
7. **Ignoring deep recursion:** A highly skewed tree can cause `RecursionError` in Python. Consider an iterative method when necessary.
8. **Not checking an empty tree:** Traversal should handle `root is None`.

## 12. Complexity Summary

For a tree containing `n` nodes:

| Traversal | Time | Auxiliary space |
|---|---:|---:|
| Recursive preorder | `O(n)` | `O(h)` |
| Recursive inorder | `O(n)` | `O(h)` |
| Recursive postorder | `O(n)` | `O(h)` |
| Iterative DFS | `O(n)` | Up to `O(h)` |
| Level order | `O(n)` | Up to `O(w)` |
| Morris inorder | `O(n)` | `O(1)`, excluding output |

A balanced binary tree has height `O(log n)`. A skewed tree can have height `O(n)`.

If the traversal returns a list containing all node values, the output itself requires `O(n)` space regardless of the auxiliary-space complexity.

## 13. Interview Quick Revision

Remember these orders:

- **Preorder:** Root → Left → Right
- **Inorder:** Left → Root → Right
- **Postorder:** Left → Right → Root
- **Level order:** Level by level using a queue

For recursive DFS, the main difference is where the node is processed relative to the recursive calls:

```python
# Preorder
process(node)
dfs(node.left)
dfs(node.right)

# Inorder
dfs(node.left)
process(node)
dfs(node.right)

# Postorder
dfs(node.left)
dfs(node.right)
process(node)
```

All three DFS traversals visit every node once, so their time complexity is `O(n)`.

## 14. Practice Checklist

- [ ] Implement recursive preorder, inorder, and postorder.
- [ ] Implement level order using `deque`.
- [ ] Return level-by-level lists.
- [ ] Implement iterative preorder and inorder.
- [ ] Find the maximum depth of a binary tree.
- [ ] Find the right-side view.
- [ ] Validate a BST using inorder traversal.
- [ ] Explain why recursive space depends on tree height.
- [ ] Explain when BFS is preferable to DFS.

