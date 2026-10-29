# Binary Search Trees (BST)

## 1. What Is a Binary Search Tree?

A **Binary Search Tree (BST)** is a binary tree in which each node follows an ordering rule:

- Values in the left subtree are smaller than the node's value.
- Values in the right subtree are greater than the node's value.
- Both left and right subtrees must also be BSTs.

A BST makes searching, inserting, and deleting values efficient when the tree is reasonably balanced.

### Example

```text
        8
       / \
      3   10
     / \    \
    1   6    14
       / \   /
      4   7 13
```

For the node `8`:
- Left subtree values are smaller than `8`.
- Right subtree values are greater than `8`.

The same rule applies recursively to every node.

## 2. BST vs. Binary Tree

| Feature | Binary Tree | Binary Search Tree |
|---|---|---|
| Maximum children | 2 | 2 |
| Ordering rule | No specific rule | Left < Node < Right |
| Searching | Usually O(n) | Average O(log n), worst O(n) |
| Insertion | Depends on the application | Follows BST ordering |
| Inorder traversal | Any order | Sorted order, with distinct keys |

**Important:** Every BST is a binary tree, but not every binary tree is a BST.

## 3. Node Structure in Python

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

Each node contains:
- `val`: the value stored in the node.
- `left`: reference to the left child.
- `right`: reference to the right child.

## 4. Searching in a BST

### Problem

Given a BST and a target value, return the node containing the target. Return `None` if it does not exist.

### Approach

At each node:

1. If the node is `None`, the target does not exist.
2. If the node's value equals the target, return the node.
3. If the target is smaller, search the left subtree.
4. Otherwise, search the right subtree.

### Recursive solution

```python
def search_bst(root, target):
    if root is None:
        return None

    if root.val == target:
        return root

    if target < root.val:
        return search_bst(root.left, target)

    return search_bst(root.right, target)
```

### Iterative solution

```python
def search_bst(root, target):
    current = root

    while current is not None:
        if current.val == target:
            return current

        if target < current.val:
            current = current.left
        else:
            current = current.right

    return None
```

### Complexity

Let `h` be the height of the tree.

- Time: `O(h)`
- Recursive auxiliary space: `O(h)`
- Iterative auxiliary space: `O(1)`

For a balanced BST, `h = O(log n)`. For a completely skewed BST, `h = O(n)`.

## 5. Insertion in a BST

### Problem

Insert a value into a BST while preserving its ordering property.

### Approach

- If the current node is `None`, create a new node.
- If the value is smaller, insert into the left subtree.
- If the value is greater, insert into the right subtree.
- If the value already exists, handle it according to the duplicate-value policy.

This implementation ignores duplicate values.

```python
def insert_bst(root, val):
    if root is None:
        return TreeNode(val)

    if val < root.val:
        root.left = insert_bst(root.left, val)
    elif val > root.val:
        root.right = insert_bst(root.right, val)

    return root
```

### Example

Insert `5` into:

```text
      8
     / \
    3   10
```

Compare `5` with `8`: go left.

Compare `5` with `3`: go right.

The result:

```text
      8
     / \
    3   10
     \
      5
```

### Complexity

- Time: `O(h)`
- Recursive auxiliary space: `O(h)`

**Important:** Always return `root`. This ensures that a newly created node is attached correctly, including when the original tree is empty.

## 6. Deletion in a BST

Deletion is a common interview problem because it has three different cases.

### Case 1: Node has no children

Delete the node by returning `None`.

```text
    8
   /
  3
```

Deleting `3` gives:

```text
    8
```

### Case 2: Node has one child

Replace the node with its only child.

Before deleting `3`:

```text
    8
   /
  3
   \
    5
```

After deleting `3`:

```text
    8
   /
  5
```

### Case 3: Node has two children

Replace the node's value with either:
- Its **inorder successor**: the smallest value in its right subtree.
- Its **inorder predecessor**: the largest value in its left subtree.

Then delete the replacement value from its original location.

### Finding the minimum value

```python
def find_min(node):
    while node.left is not None:
        node = node.left

    return node
```

### Complete deletion solution

```python
def delete_bst(root, key):
    if root is None:
        return None

    if key < root.val:
        root.left = delete_bst(root.left, key)

    elif key > root.val:
        root.right = delete_bst(root.right, key)

    else:
        # Case 1 and Case 2
        if root.left is None:
            return root.right

        if root.right is None:
            return root.left

        # Case 3: Two children
        successor = find_min(root.right)
        root.val = successor.val
        root.right = delete_bst(root.right, successor.val)

    return root
```

### Complexity

- Time: `O(h)`
- Recursive auxiliary space: `O(h)`

**Common mistake:** When deleting a node with two children, do not simply remove it and discard its children. Doing so can lose part of the tree or violate BST ordering.

## 7. Inorder Traversal of a BST

Inorder traversal visits nodes in this order:

1. Left subtree
2. Current node
3. Right subtree

For a BST with distinct values, inorder traversal returns the values in ascending order.

```python
def inorder(root):
    result = []

    def dfs(node):
        if node is None:
            return

        dfs(node.left)
        result.append(node.val)
        dfs(node.right)

    dfs(root)
    return result
```

Example:

```text
        8
       / \
      3   10
     / \
    1   6
```

Output:

```python
[1, 3, 6, 8, 10]
```

Complexity:
- Time: `O(n)`
- Auxiliary space: `O(h)` for recursion, excluding the output list.

## 8. Validate a Binary Search Tree

### Problem

Determine whether a binary tree satisfies the BST property.

### The common mistake

Checking only the immediate children is insufficient.

Consider:

```text
       8
      / \
     3   10
        /
       6
```

The node `6` is smaller than `10`, but it is in the right subtree of `8`, where every value must be greater than `8`. Therefore, this is not a valid BST.

### Correct approach: Use value ranges

Each node must lie within a valid lower and upper bound.

```python
def is_valid_bst(root):
    def validate(node, low, high):
        if node is None:
            return True

        if not (low < node.val < high):
            return False

        return (
            validate(node.left, low, node.val)
            and validate(node.right, node.val, high)
        )

    return validate(root, float("-inf"), float("inf"))
```

### How the bounds work

For the root `8`:
- Its value must be between negative and positive infinity.
- Every node in the left subtree must be less than `8`.
- Every node in the right subtree must be greater than `8`.

These bounds become stricter as recursion moves deeper.

### Complexity

- Time: `O(n)`
- Auxiliary space: `O(h)`

**Duplicate values:** This implementation rejects duplicates. If duplicates are allowed, the validation rules must explicitly define where equal values can appear.

## 9. Find the Minimum and Maximum

In a BST:
- The minimum is the leftmost node.
- The maximum is the rightmost node.

### Minimum

```python
def bst_min(root):
    if root is None:
        return None

    while root.left is not None:
        root = root.left

    return root.val
```

### Maximum

```python
def bst_max(root):
    if root is None:
        return None

    while root.right is not None:
        root = root.right

    return root.val
```

Complexity for both:
- Time: `O(h)`
- Auxiliary space: `O(1)`

## 10. Find the Lowest Common Ancestor (LCA) in a BST

The **Lowest Common Ancestor** of two nodes is the lowest node that has both nodes as descendants. A node can also be an ancestor of itself.

BST ordering allows us to find the LCA without exploring every node.

### Approach

For nodes with values `p` and `q`:

- If both are smaller than the current node, move left.
- If both are greater than the current node, move right.
- Otherwise, the current node is the split point and therefore the LCA.

```python
def lowest_common_ancestor(root, p, q):
    current = root

    while current is not None:
        if p < current.val and q < current.val:
            current = current.left
        elif p > current.val and q > current.val:
            current = current.right
        else:
            return current

    return None
```

### Example

```text
        8
       / \
      3   10
     / \
    1   6
```

For `p = 1` and `q = 6`, the LCA is `3`.

For `p = 1` and `q = 10`, the LCA is `8`.

Complexity:
- Time: `O(h)`
- Auxiliary space: `O(1)`

This implementation assumes `p` and `q` are values of nodes in the BST.

## 11. Kth Smallest Element in a BST

### Problem

Find the kth smallest value in a BST, where `k` is 1-indexed.

### Key observation

Inorder traversal produces values in ascending order. Therefore, the kth visited node is the kth smallest.

```python
def kth_smallest(root, k):
    stack = []
    current = root

    while current is not None or stack:
        while current is not None:
            stack.append(current)
            current = current.left

        current = stack.pop()
        k -= 1

        if k == 0:
            return current.val

        current = current.right

    return None
```

### Complexity

- Time: `O(h + k)` in the usual early-stopping analysis, with worst case `O(n)`.
- Auxiliary space: `O(h)`.

**Important:** The code assumes `k` is valid. If `k` is greater than the number of nodes, it returns `None`.

## 12. Convert a Sorted Array into a Balanced BST

### Problem

Given a sorted array of distinct values, construct a height-balanced BST.

### Approach

1. Select the middle element as the root.
2. Recursively construct the left subtree from the left half.
3. Recursively construct the right subtree from the right half.

Choosing the middle element keeps the two subtrees similar in size.

```python
def sorted_array_to_bst(nums):
    def build(left, right):
        if left > right:
            return None

        mid = (left + right) // 2
        root = TreeNode(nums[mid])

        root.left = build(left, mid - 1)
        root.right = build(mid + 1, right)

        return root

    return build(0, len(nums) - 1)
```

### Complexity

- Time: `O(n)`, because each element becomes one node.
- Auxiliary space: `O(log n)` for recursion when the constructed tree is balanced, excluding the output tree.

## 13. BST Successor and Predecessor

### Inorder successor

The successor of a node is the next larger value in inorder traversal.

If the node has a right subtree, its successor is the minimum node in that right subtree.

If it has no right subtree, move upward until reaching an ancestor for which the node lies in that ancestor's left subtree.

### Inorder predecessor

The predecessor is the next smaller value in inorder traversal.

If the node has a left subtree, its predecessor is the maximum node in that left subtree.

If it has no left subtree, move upward until reaching an ancestor for which the node lies in that ancestor's right subtree.

These are common follow-up questions in BST interviews. Implementations that use parent pointers can walk upward directly; without parent pointers, the search path or root may be needed.

## 14. BST Height and Performance

Let:
- `n` = number of nodes.
- `h` = height of the tree.

The time complexity of search, insertion, and deletion depends on `h`, not automatically on `log n`.

| Operation | Balanced BST | Worst-case skewed BST |
|---|---:|---:|
| Search | `O(log n)` | `O(n)` |
| Insert | `O(log n)` | `O(n)` |
| Delete | `O(log n)` | `O(n)` |
| Find minimum/maximum | `O(log n)` | `O(n)` |
| Inorder traversal | `O(n)` | `O(n)` |

A normal BST does not automatically remain balanced.

For example, inserting values in sorted order:

```python
values = [1, 2, 3, 4, 5]
```

may produce:

```text
1
 \
  2
   \
    3
     \
      4
       \
        5
```

This tree behaves like a linked list for searching.

Balanced BST variants such as AVL trees and Red-Black trees use rotations and additional rules to maintain logarithmic height.

## 15. Common Interview Mistakes

1. **Checking only immediate children when validating a BST.** Every node must satisfy all ancestor-imposed bounds.
2. **Assuming every BST operation is `O(log n)`.** This is only guaranteed when height is logarithmic.
3. **Forgetting to assign recursive results.** Use `root.left = ...` and `root.right = ...` during insertion and deletion.
4. **Forgetting to return the root.** This can break insertion into an empty tree or lose an updated subtree.
5. **Deleting a node with two children incorrectly.** Replace it with a successor or predecessor, then remove that replacement from its original position.
6. **Confusing a binary tree with a BST.** Only BSTs follow the ordering property.
7. **Using preorder traversal to obtain sorted values.** Inorder traversal is the standard traversal for sorted BST output.
8. **Ignoring duplicate-value rules.** Decide whether duplicates are rejected, counted, or placed consistently on one side.
9. **Using the wrong LCA logic.** Exploit BST ordering instead of searching both subtrees unnecessarily.
10. **Assuming a sorted array and a normal BST have the same height.** A sorted array can build a balanced BST, but repeated insertion into a normal BST may create a skewed tree.

## 16. Interview Practice Checklist

### Easy

- [ ] Search in a Binary Search Tree
- [ ] Insert into a Binary Search Tree
- [ ] Find minimum and maximum values
- [ ] Convert a BST to a sorted list using inorder traversal
- [ ] Find the kth smallest element

### Medium

- [ ] Validate Binary Search Tree
- [ ] Delete a node from a BST
- [ ] Lowest Common Ancestor of a BST
- [ ] Convert Sorted Array to Binary Search Tree
- [ ] Find inorder successor and predecessor
- [ ] Recover a BST with two swapped values

### Advanced

- [ ] Design an iterator over a BST
- [ ] Find the closest value to a target
- [ ] Count values within a given range
- [ ] Construct a BST from preorder traversal
- [ ] Balance an existing BST
- [ ] Compare BSTs and AVL/Red-Black trees

## 17. Quick Revision

- **BST property:** left subtree values are smaller; right subtree values are greater, assuming distinct keys.
- **Search:** compare the target with the current node and move left or right.
- **Insertion:** follow the ordering rule until an empty position is found.
- **Deletion:** handle zero, one, or two children separately.
- **Inorder traversal:** produces sorted values.
- **Validation:** use lower and upper bounds.
- **Minimum:** leftmost node.
- **Maximum:** rightmost node.
- **Kth smallest:** kth node visited during inorder traversal.
- **LCA:** find the split point using BST ordering.
- **Balanced BST:** search, insertion, and deletion take `O(log n)`.
- **Skewed BST:** these operations can take `O(n)`.
- **Core rule:** BST operations depend on tree height, `h`.

## 18. Questions to Explain Out Loud

Before an interview, make sure you can answer these without looking at the notes:

1. Why does inorder traversal of a BST produce sorted values?
2. Why can searching in a BST degrade to `O(n)`?
3. How do you validate a BST when a node may violate an ancestor's constraint?
4. Why is deletion with two children more complicated than deletion with one child?
5. What is the difference between an inorder successor and predecessor?
6. How can you construct a balanced BST from a sorted array?
7. How can you find the LCA without visiting every node?
8. When would you choose a balanced BST instead of a hash table?

