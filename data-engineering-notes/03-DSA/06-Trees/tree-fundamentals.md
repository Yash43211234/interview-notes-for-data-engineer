# Tree Fundamentals in Python

## 1. What Is a Tree?

A **tree** is a non-linear data structure made up of nodes connected by edges. It represents hierarchical relationships between elements.

Examples include:
- File and folder systems
- Organization hierarchies
- HTML DOM structures
- Database indexes
- Decision trees

Unlike arrays and linked lists, trees can branch into multiple paths.

## 2. Basic Tree Terminology

Consider this binary tree:

```text
            1
          /   \
         2     3
        / \     \
       4   5     6
```

| Term | Meaning | Example |
|---|---|---|
| Node | An individual element | `2` |
| Root | The topmost node | `1` |
| Edge | A connection between two nodes | `1 → 2` |
| Parent | A node directly above another node | `2` is the parent of `4` |
| Child | A node directly below another node | `4` is a child of `2` |
| Siblings | Nodes sharing the same parent | `2` and `3` |
| Leaf | A node with no children | `4`, `5`, `6` |
| Internal node | A node with at least one child | `1`, `2`, `3` |
| Ancestor | A node on the path from the root to another node | `1` is an ancestor of `5` |
| Descendant | A node below another node | `5` is a descendant of `1` |
| Subtree | A node and all its descendants | The subtree rooted at `2` |
| Depth | Number of edges from the root to a node | Depth of `5` is `2` |
| Height of a node | Number of edges on its longest downward path to a leaf | Height of `2` is `1` |
| Height of a tree | Height of its root | Height is `2` |

**Convention:** This note measures height and depth in edges. Therefore, a leaf has height `0`, and a tree containing only one node has height `0`. Some resources count nodes instead, so always check the convention used by a problem.

## 3. Types of Trees

### General Tree

A node can have any number of children.

```text
        A
      / | \
     B  C  D
       / \
      E   F
```

### Binary Tree

Each node can have at most two children: left and right.

```text
        1
       / \
      2   3
```

### Full Binary Tree

Every node has either zero or two children.

```text
        1
       / \
      2   3
```

A node with exactly one child is not allowed in a full binary tree.

### Complete Binary Tree

Every level is completely filled except possibly the last level. The last level is filled from left to right.

```text
        1
       / \
      2   3
     / \  /
    4  5 6
```

### Perfect Binary Tree

Every internal node has exactly two children, and all leaves are at the same depth.

```text
        1
       / \
      2   3
     / \ / \
    4  5 6  7
```

### Balanced Binary Tree

A balanced tree keeps its height relatively small compared with its number of nodes.

The exact balance condition depends on the tree type. For example, an AVL tree requires the heights of a node's two subtrees to differ by at most one.

### Skewed Binary Tree

Each node has only one child, so the tree resembles a linked list.

```text
    1
     \
      2
       \
        3
         \
          4
```

A skewed tree can have height `O(n)`.

## 4. Binary Tree Node in Python

A common node implementation is:

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

Each node stores three things:

- `val`: the data stored in the node.
- `left`: a reference to the left child.
- `right`: a reference to the right child.

### Creating a tree manually

```python
root = TreeNode(1)

root.left = TreeNode(2)
root.right = TreeNode(3)

root.left.left = TreeNode(4)
root.left.right = TreeNode(5)
```

The resulting structure is:

```text
        1
       / \
      2   3
     / \
    4   5
```

`root` refers to the first node. The other nodes are reachable by following its child references.

**Important:** A tree is connected through references, not by storing every node inside the root directly.

## 5. Recursive Thinking in Trees

Recursion is fundamental to tree problems because each child can itself be treated as the root of a smaller tree.

A typical recursive structure is:

```python
def solve(node):
    if node is None:
        return

    # Process the current node
    solve(node.left)
    solve(node.right)
```

This structure contains three important parts:

1. **Base case:** What happens when the node is missing?
2. **Current-node work:** What should happen at this node?
3. **Recursive calls:** How should the problem be solved for its children?

The position of the current-node work determines the traversal order.

- Before recursive calls: preorder
- Between recursive calls: inorder
- After recursive calls: postorder

Not every tree problem needs all three parts, but this is a useful starting template.

## 6. Calculate the Number of Nodes

Count the current node and add the number of nodes in both subtrees.

```python
def count_nodes(root):
    if root is None:
        return 0

    return (
        1
        + count_nodes(root.left)
        + count_nodes(root.right)
    )
```

For the tree containing `1`, `2`, `3`, `4`, and `5`, the answer is `5`.

**Complexity**

- Time: `O(n)`
- Auxiliary space: `O(h)` for the recursion stack

Here, `n` is the number of nodes and `h` is the tree height.

## 7. Calculate the Height of a Binary Tree

The height is the number of edges on the longest path from the root to a leaf.

With the edge-counting convention:

- An empty tree has height `-1`.
- A single-node tree has height `0`.
- A root with one child leaf has height `1`.

```python
def height(root):
    if root is None:
        return -1

    return 1 + max(
        height(root.left),
        height(root.right)
    )
```

Why does this work?

1. Find the height of the left subtree.
2. Find the height of the right subtree.
3. Choose the larger height.
4. Add one edge to reach the child subtree from the current node.

**Complexity**

- Time: `O(n)`
- Auxiliary space: `O(h)`

Some interview problems define the height as the number of nodes on the longest path. Under that convention, the empty tree has height `0`, and a leaf has height `1`. Adjust the base case accordingly.

## 8. Calculate the Number of Leaf Nodes

A leaf is a node with neither a left nor a right child.

```python
def count_leaves(root):
    if root is None:
        return 0

    if root.left is None and root.right is None:
        return 1

    return (
        count_leaves(root.left)
        + count_leaves(root.right)
    )
```

**Complexity**

- Time: `O(n)`
- Auxiliary space: `O(h)`

A common mistake is to count every node with at least one missing child as a leaf. A leaf must have both children missing.

## 9. Search for a Value in a Binary Tree

In an ordinary binary tree, there is no guaranteed ordering between node values. Therefore, in the worst case, you may need to inspect every node.

```python
def contains(root, target):
    if root is None:
        return False

    if root.val == target:
        return True

    return (
        contains(root.left, target)
        or contains(root.right, target)
    )
```

**Complexity**

- Worst-case time: `O(n)`
- Auxiliary space: `O(h)`

This is different from searching a Binary Search Tree (BST), where ordering can eliminate one subtree at each step.

## 10. Find the Maximum Value in a Binary Tree

Because an ordinary binary tree has no ordering rule, compare the values in both subtrees.

```python
def max_value(root):
    if root is None:
        return None

    left_max = max_value(root.left)
    right_max = max_value(root.right)

    values = [root.val]

    if left_max is not None:
        values.append(left_max)

    if right_max is not None:
        values.append(right_max)

    return max(values)
```

This implementation assumes node values are numeric and uses `None` to represent an empty tree.

**Complexity**

- Time: `O(n)`
- Auxiliary space: `O(h)`

## 11. Tree Traversal Overview

Traversal means visiting nodes in a defined order.

Using this example:

```text
        1
       / \
      2   3
     / \
    4   5
```

| Traversal | Order | Output |
|---|---|---|
| Preorder | Root, Left, Right | `[1, 2, 4, 5, 3]` |
| Inorder | Left, Root, Right | `[4, 2, 5, 1, 3]` |
| Postorder | Left, Right, Root | `[4, 5, 2, 3, 1]` |
| Level order | Level by level | `[1, 2, 3, 4, 5]` |

Preorder, inorder, and postorder are commonly implemented using DFS. Level order uses BFS and a queue.

See `traversals.md` for complete recursive and iterative implementations.

## 12. Binary Tree vs. Binary Search Tree

| Feature | Binary Tree | Binary Search Tree |
|---|---|---|
| Children per node | At most two | At most two |
| Ordering rule | None required | Left values are smaller and right values are greater, assuming distinct keys |
| Search | Worst-case `O(n)` | `O(h)` |
| Inorder traversal | No guaranteed sorted order | Sorted order |
| Insertion | Depends on the problem | Follows the ordering rule |

A BST is a special kind of binary tree.

Do not apply BST-specific assumptions to an ordinary binary tree.

## 13. Time and Space Complexity

For a binary tree containing `n` nodes and height `h`:

| Operation | Time | Auxiliary space |
|---|---:|---:|
| Count all nodes | `O(n)` | `O(h)` |
| Find height | `O(n)` | `O(h)` |
| Count leaves | `O(n)` | `O(h)` |
| Search an ordinary binary tree | `O(n)` worst case | `O(h)` |
| Find maximum value | `O(n)` | `O(h)` |
| DFS traversal | `O(n)` | `O(h)` |
| BFS traversal | `O(n)` | Up to `O(n)` |

A balanced binary tree usually has height `O(log n)`. A skewed binary tree can have height `O(n)`.

Recursive algorithms use call-stack space proportional to the maximum recursion depth. Iterative algorithms may instead use an explicit stack or queue.

## 14. Common Interview Mistakes

1. **Forgetting the base case:** Always handle `None`.
2. **Confusing height and depth:** Depth measures distance from the root; height measures distance down to the deepest leaf.
3. **Mixing edge and node conventions:** Read the question carefully before choosing the base case.
4. **Assuming every binary tree is a BST:** An ordinary binary tree has no automatic ordering.
5. **Ignoring skewed trees:** Recursion depth can become `O(n)`.
6. **Returning the wrong value for an empty tree:** The correct return value depends on the problem and its definitions.
7. **Using global variables unnecessarily:** Return results from recursive calls when possible.
8. **Confusing a leaf with a node that has one child:** A leaf has zero children.

## 15. Practice Checklist

### Easy

- [ ] Count the number of nodes.
- [ ] Calculate the height of a binary tree.
- [ ] Count the leaf nodes.
- [ ] Search for a value in a binary tree.
- [ ] Find the maximum value.
- [ ] Implement preorder, inorder, and postorder traversal.

### Medium

- [ ] Invert a binary tree.
- [ ] Check whether two trees are identical.
- [ ] Check whether a tree is symmetric.
- [ ] Find the diameter of a binary tree.
- [ ] Check whether a tree is height-balanced.
- [ ] Find the lowest common ancestor in a binary tree.

## 16. Quick Revision

- A tree is a collection of nodes connected through edges in a hierarchical structure.
- The root is the topmost node.
- A leaf has no children.
- Depth is measured downward from the root to a node.
- Height is measured downward from a node to its deepest leaf.
- Recursion works naturally because each subtree is itself a tree.
- Ordinary binary tree search takes `O(n)` in the worst case.
- DFS uses recursion or a stack.
- BFS uses a queue.
- A BST is a binary tree with an additional ordering rule.
- Always clarify the height convention used in a problem.

