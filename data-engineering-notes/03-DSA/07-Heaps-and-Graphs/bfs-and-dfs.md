# BFS and DFS in Python

## 1. What Are BFS and DFS?

**Breadth-First Search (BFS)** and **Depth-First Search (DFS)** are two fundamental algorithms for traversing and searching trees and graphs.

- **BFS:** Explores nearby nodes first, level by level.
- **DFS:** Explores one path as deeply as possible before backtracking.

Both algorithms help solve problems involving connectivity, paths, tree traversal, and graph exploration.

## 2. Visual Example

Consider this binary tree:

```text
          1
        /   \
       2     3
      / \   / \
     4   5 6   7
```

BFS visits nodes level by level:

```text
1 → 2 → 3 → 4 → 5 → 6 → 7
```

DFS preorder visits the root, then the left subtree, then the right subtree:

```text
1 → 2 → 4 → 5 → 3 → 6 → 7
```

**Important:** DFS does not have just one traversal order. Preorder, inorder, and postorder are all DFS traversal variants for binary trees.

## 3. Breadth-First Search (BFS)

BFS explores all immediate neighbors of a node before moving to the next level.

### How BFS works

1. Start with the source node.
2. Add it to a queue.
3. Remove the front node from the queue and process it.
4. Add its unvisited neighbors to the queue.
5. Continue until the queue is empty.

A queue follows **FIFO (First In, First Out)**.

### BFS implementation for a binary tree

```python
from collections import deque

def bfs(root):
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

### Example output

```python
[1, 2, 3, 4, 5, 6, 7]
```

### Why use `deque`?

Removing the first item from a Python list with `pop(0)` requires shifting the remaining elements and can be expensive.

`collections.deque` supports efficient removal from the front using `popleft()`.

### BFS complexity

For a tree with `n` nodes:

- Time: `O(n)`
- Auxiliary space: `O(w)`, where `w` is the maximum width of the tree; worst case `O(n)`.

For a graph represented by an adjacency list:

- Time: `O(V + E)`
- Auxiliary space: `O(V)`

Here, `V` is the number of vertices and `E` is the number of edges.

## 4. BFS Level-by-Level Traversal

Some interview problems require each tree level to be returned as a separate list.

### Implementation

```python
from collections import deque

def level_order(root):
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
    [4, 5, 6, 7]
]
```

### Why save `level_size`?

At the beginning of each iteration, `level_size` records how many nodes belong to the current level.

Their children are added to the queue, but they are processed only during the next outer-loop iteration.

**Common interview applications:**
- Binary Tree Level Order Traversal
- Minimum Depth of Binary Tree
- Binary Tree Right Side View
- Rotting Oranges
- Shortest path in an unweighted graph

## 5. Depth-First Search (DFS)

DFS follows one path as deeply as possible before exploring alternative paths.

It can be implemented using:
- Recursion, which uses the call stack.
- An explicit stack, which you manage yourself.

### Recursive DFS for a binary tree

```python
def dfs(root):
    if root is None:
        return []

    result = []

    def visit(node):
        if node is None:
            return

        result.append(node.val)
        visit(node.left)
        visit(node.right)

    visit(root)
    return result
```

This implementation uses preorder traversal.

Output:

```python
[1, 2, 4, 5, 3, 6, 7]
```

### DFS complexity

For a tree with `n` nodes:

- Time: `O(n)`
- Auxiliary space: `O(h)` for the recursive call stack, where `h` is tree height.

For a graph represented by an adjacency list:

- Time: `O(V + E)`
- Auxiliary space: `O(V)` in the worst case for the visited set and traversal structure.

## 6. Three DFS Traversal Orders

The position of the node-processing operation determines the traversal order.

Using this tree:

```text
        1
       / \
      2   3
     / \
    4   5
```

### Preorder: Root → Left → Right

```python
def preorder(root):
    if root is None:
        return []

    return (
        [root.val]
        + preorder(root.left)
        + preorder(root.right)
    )
```

Output:

```python
[1, 2, 4, 5, 3]
```

### Inorder: Left → Root → Right

```python
def inorder(root):
    if root is None:
        return []

    return (
        inorder(root.left)
        + [root.val]
        + inorder(root.right)
    )
```

Output:

```python
[4, 2, 5, 1, 3]
```

### Postorder: Left → Right → Root

```python
def postorder(root):
    if root is None:
        return []

    return (
        postorder(root.left)
        + postorder(root.right)
        + [root.val]
    )
```

Output:

```python
[4, 5, 2, 3, 1]
```

These concise versions create intermediate lists at each recursive call. They are easy to understand, but for better performance on large trees, prefer accumulating results into a single list.

## 7. DFS on a Graph

Graphs may contain cycles, so DFS must track visited nodes to avoid revisiting them indefinitely.

Suppose the graph is represented by an adjacency list:

```python
graph = {
    1: [2, 3],
    2: [1, 4, 5],
    3: [1, 6],
    4: [2],
    5: [2],
    6: [3]
}
```

### Recursive DFS

```python
def dfs_graph(graph, start):
    visited = set()
    result = []

    def visit(node):
        visited.add(node)
        result.append(node)

        for neighbor in graph[node]:
            if neighbor not in visited:
                visit(neighbor)

    visit(start)
    return result
```

For this graph, starting from `1`, one possible result is:

```python
[1, 2, 4, 5, 3, 6]
```

The exact order depends on the order in which neighbors appear in the adjacency lists.

### Why is `visited` necessary?

Consider the edge between `1` and `2`:

- DFS visits `1`, then `2`.
- From `2`, it sees `1` again.
- Because `1` is already visited, DFS skips it.

Without a visited set, cyclic graphs could cause infinite recursion.

### Iterative DFS

```python
def dfs_graph_iterative(graph, start):
    visited = set()
    stack = [start]
    result = []

    while stack:
        node = stack.pop()

        if node in visited:
            continue

        visited.add(node)
        result.append(node)

        for neighbor in reversed(graph[node]):
            if neighbor not in visited:
                stack.append(neighbor)

    return result
```

Reversing the neighbors can preserve a natural left-to-right or listed order for the stack-based traversal, depending on the graph representation.

Marking a node as visited when it is popped is valid, although duplicate entries may temporarily enter the stack. Another common implementation marks nodes as visited when they are pushed.

Both recursive and iterative graph DFS run in `O(V + E)` time with an adjacency list.

## 8. BFS on a Graph

```python
from collections import deque

def bfs_graph(graph, start):
    visited = {start}
    queue = deque([start])
    result = []

    while queue:
        node = queue.popleft()
        result.append(node)

        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

    return result
```

Marking nodes as visited when they are added to the queue prevents the same node from being enqueued repeatedly.

### When is graph BFS useful?

- Finding the shortest path in an unweighted graph.
- Finding the minimum number of moves in a grid.
- Processing nodes by distance from a source.
- Finding connected components, when repeated from each unvisited node.

**Important:** BFS guarantees shortest paths by number of edges in an unweighted graph. It does not generally guarantee the minimum total weight in a weighted graph. Dijkstra's algorithm is commonly used for graphs with non-negative edge weights.

## 9. BFS vs. DFS

| Feature | BFS | DFS |
|---|---|---|
| Exploration | Level by level | Deep along one path |
| Main structure | Queue | Stack or recursion |
| Tree applications | Level order, minimum depth | Subtree properties, paths |
| Graph applications | Unweighted shortest paths | Cycle detection, reachability |
| Time on adjacency-list graph | `O(V + E)` | `O(V + E)` |
| Extra traversal space | Up to `O(V)` | Up to `O(V)` |
| Shortest unweighted path | Yes | Not guaranteed |
| Recursion required | No | Only in recursive implementation |

Neither algorithm is universally better. Choose based on what the problem asks you to compute.

## 10. Important Graph Patterns

### Connected components

For an undirected graph, start BFS or DFS from each unvisited vertex. Every traversal discovers one connected component.

```python
def count_components(n, edges):
    graph = {i: [] for i in range(n)}

    for u, v in edges:
        graph[u].append(v)
        graph[v].append(u)

    visited = set()
    components = 0

    for node in range(n):
        if node in visited:
            continue

        components += 1
        stack = [node]
        visited.add(node)

        while stack:
            current = stack.pop()

            for neighbor in graph[current]:
                if neighbor not in visited:
                    visited.add(neighbor)
                    stack.append(neighbor)

    return components
```

- Time: `O(V + E)`
- Auxiliary space: `O(V)`

This also correctly counts isolated vertices as components.

### Grid traversal

For a grid problem such as Number of Islands, each cell can be treated as a vertex. Its neighbors are usually the cells directly above, below, left, and right.

A typical solution:
1. Scan the grid.
2. When an unvisited land cell is found, start DFS or BFS.
3. Mark all connected land cells as visited.
4. Increase the island count.
5. Continue scanning.

If the grid contains `R` rows and `C` columns, the time complexity is usually `O(R × C)` because each cell is processed a constant number of times.

### Topological sorting

For a directed acyclic graph (DAG), BFS can be used with Kahn's algorithm:

1. Calculate the indegree of every vertex.
2. Add all zero-indegree vertices to a queue.
3. Remove a vertex and add it to the output.
4. Decrease the indegree of its outgoing neighbors.
5. Add any neighbor whose indegree becomes zero.
6. If the output contains fewer than `V` vertices, the graph contains a directed cycle.

This is useful for dependency ordering, including Course Schedule problems.

## 11. Common Interview Mistakes

1. **Forgetting the visited set in graph traversal.** Cycles can cause repeated visits or infinite recursion.
2. **Marking nodes visited too late in BFS.** Mark them when enqueuing to avoid duplicate queue entries.
3. **Using `list.pop(0)` for BFS.** Prefer `deque.popleft()`.
4. **Assuming DFS finds the shortest path.** Ordinary DFS does not guarantee this.
5. **Confusing tree and graph traversal.** A tree has no cycles, but a graph may have cycles and disconnected components.
6. **Ignoring disconnected vertices.** One traversal from a single starting node visits only its reachable component.
7. **Using recursion on very deep graphs.** Python can raise `RecursionError`; consider iterative DFS.
8. **Assuming a fixed DFS order.** The order depends on the ordering of neighbors.
9. **Mixing the current BFS level with the next level.** Save the queue length before processing the level.
10. **Forgetting isolated vertices.** They are valid components even when they have no edges.

## 12. Complexity Reference

Let `V` be the number of vertices and `E` be the number of edges.

| Operation | Time | Auxiliary space |
|---|---:|---:|
| BFS on an adjacency-list graph | `O(V + E)` | `O(V)` |
| DFS on an adjacency-list graph | `O(V + E)` | `O(V)` worst case |
| BFS on a binary tree | `O(n)` | Up to `O(n)` |
| DFS on a binary tree | `O(n)` | `O(h)` recursively |
| Count connected components | `O(V + E)` | `O(V)` |
| BFS shortest paths in an unweighted graph | `O(V + E)` | `O(V)` |

For an adjacency matrix, scanning all neighbors of each vertex may require `O(V²)` time for BFS or DFS.

## 13. Interview Practice Checklist

### Easy

- [ ] Binary Tree Level Order Traversal
- [ ] Maximum Depth of Binary Tree
- [ ] Invert Binary Tree
- [ ] Flood Fill
- [ ] Find if Path Exists in Graph

### Medium

- [ ] Number of Islands
- [ ] Rotting Oranges
- [ ] Binary Tree Right Side View
- [ ] Number of Connected Components
- [ ] Course Schedule
- [ ] Clone Graph
- [ ] Shortest Path in an Unweighted Graph

### Advanced

- [ ] Word Ladder
- [ ] Pacific Atlantic Water Flow
- [ ] Detect a Cycle in a Directed Graph
- [ ] Shortest Path in a Binary Matrix
- [ ] Topological Sorting
- [ ] Strongly Connected Components

## 14. Quick Revision

- **BFS:** Queue, level-by-level exploration.
- **DFS:** Stack or recursion, depth-first exploration.
- **Tree traversal:** DFS does not need a visited set when the input is a proper tree.
- **Graph traversal:** Use a visited set to handle cycles.
- **Shortest path in an unweighted graph:** BFS.
- **Reachability and connected components:** BFS or DFS.
- **Level order traversal:** BFS.
- **Subtree calculations:** Often DFS.
- **Disconnected graph:** Start new traversals from unvisited vertices.
- **Time complexity with an adjacency list:** `O(V + E)` for a complete BFS or DFS traversal.

