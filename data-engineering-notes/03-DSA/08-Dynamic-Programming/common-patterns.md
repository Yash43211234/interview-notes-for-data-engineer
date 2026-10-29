# Common Graph Patterns

## 1. Why Graph Patterns Matter

Graph problems often look different on the surface but use the same underlying techniques.

For example:
- Finding whether two nodes are connected uses BFS or DFS.
- Counting islands in a grid uses graph traversal.
- Detecting circular dependencies uses cycle detection.
- Scheduling tasks uses topological sorting.
- Finding the shortest path depends on whether edges are weighted.

The goal is to identify the correct pattern before writing code.

---

## 2. Graph Representation

Most graph problems provide either an edge list, an adjacency list, or a grid.

### Adjacency List

```python
graph = {
    0: [1, 2],
    1: [0, 3],
    2: [0, 3],
    3: [1, 2]
}
```

Each key represents a vertex, and its list contains its neighbors.

For an undirected graph, add both directions when building the graph:

```python
graph[u].append(v)
graph[v].append(u)
```

For a directed graph, add only the outgoing edge:

```python
graph[u].append(v)
```

### Build an Adjacency List

```python
from collections import defaultdict

def build_graph(n, edges, directed=False):
    graph = defaultdict(list)

    for node in range(n):
        graph[node]

    for u, v in edges:
        graph[u].append(v)

        if not directed:
            graph[v].append(u)

    return graph
```

The initialization preserves isolated vertices, which have no edges.

Time complexity: \(O(V+E)\).

---

## 3. Pattern 1: Basic BFS

**Use when:** You need to explore nodes level by level or find the shortest path in an unweighted graph.

BFS uses a queue.

### Template

```python
from collections import deque

def bfs(graph, start):
    visited = {start}
    queue = deque([start])
    order = []

    while queue:
        node = queue.popleft()
        order.append(node)

        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

    return order
```

### Key Idea

Mark a node as visited when adding it to the queue. This prevents duplicate queue entries.

### Complexity

- Time: \(O(V+E)\)
- Space: \(O(V)\)

### Common Applications

- Shortest path in an unweighted graph
- Level-by-level traversal
- Finding reachable vertices
- Minimum number of moves in a grid

---

## 4. Pattern 2: Basic DFS

**Use when:** You need to explore paths deeply, find connected regions, or perform recursive graph processing.

DFS uses recursion or an explicit stack.

### Recursive Template

```python
def dfs(graph, start):
    visited = set()
    order = []

    def visit(node):
        visited.add(node)
        order.append(node)

        for neighbor in graph[node]:
            if neighbor not in visited:
                visit(neighbor)

    visit(start)
    return order
```

### Iterative Template

```python
def dfs_iterative(graph, start):
    visited = {start}
    stack = [start]
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

The iterative version may produce a different traversal order from recursive DFS.

### Complexity

- Time: \(O(V+E)\)
- Space: \(O(V)\)

**Important:** Recursive DFS may exceed Python's recursion limit on a deep graph. Use iterative DFS when necessary.

---

## 5. Pattern 3: Connected Components

**Use when:** You need to count separate groups of connected vertices in an undirected graph.

Example:

```text
0 ----- 1       2 ----- 3       4
```

There are three connected components:

- `{0, 1}`
- `{2, 3}`
- `{4}`

### Template

```python
def count_components(n, graph):
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

### Complexity

- Time: \(O(V+E)\)
- Space: \(O(V)\)

### Interview Examples

- Number of connected components
- Number of provinces
- Number of groups in a network
- Number of islands, when the input is a grid

**Key insight:** The outer loop is necessary because the graph may be disconnected.

---

## 6. Pattern 4: Grid as a Graph

A grid can be treated as a graph in which each cell is a vertex and neighboring cells are connected.

For four-directional movement:

```text
        Up
         |
Left -- Cell -- Right
         |
       Down
```

### Direction Array

```python
directions = [
    (-1, 0),
    (1, 0),
    (0, -1),
    (0, 1)
]
```

Each pair represents a change in row and column.

### Number of Islands

Suppose `1` represents land and `0` represents water. Count the connected groups of land.

```python
def num_islands(grid):
    if not grid or not grid[0]:
        return 0

    rows = len(grid)
    cols = len(grid[0])
    islands = 0

    def dfs(r, c):
        if (
            r < 0 or r >= rows
            or c < 0 or c >= cols
            or grid[r][c] != "1"
        ):
            return

        grid[r][c] = "0"

        for dr, dc in [
            (-1, 0), (1, 0),
            (0, -1), (0, 1)
        ]:
            dfs(r + dr, c + dc)

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == "1":
                islands += 1
                dfs(r, c)

    return islands
```

This solution modifies the input grid by turning visited land into water.

If modifying the input is not allowed, maintain a separate visited set.

### Complexity

For an \(R \times C\) grid:

- Time: \(O(RC)\)
- Space: \(O(RC)\) in the worst case, including recursion or visited state.

### Related Problems

- Flood Fill
- Max Area of Island
- Number of Closed Islands
- Surrounded Regions
- Rotten Oranges

---

## 7. Pattern 5: Shortest Path in an Unweighted Graph

**Use when:** Every edge has equal cost and you need the minimum number of edges between a source and destination.

BFS finds the shortest path because it processes vertices in increasing order of distance from the source.

### Template

```python
from collections import deque

def shortest_distance(graph, source, target):
    queue = deque([(source, 0)])
    visited = {source}

    while queue:
        node, distance = queue.popleft()

        if node == target:
            return distance

        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, distance + 1))

    return -1
```

### Complexity

- Time: \(O(V+E)\)
- Space: \(O(V)\)

The function returns `-1` if the target is unreachable.

### To Reconstruct the Actual Path

Store each node's predecessor when it is first discovered.

```python
parent[neighbor] = node
```

After reaching the target, follow the parent links backward from the target to the source, then reverse the resulting path.

**Important:** BFS is not generally the correct algorithm for finding the shortest path when edges have different weights. Use Dijkstra's algorithm for non-negative weighted edges.

---

## 8. Pattern 6: Cycle Detection in an Undirected Graph

**Use when:** You need to determine whether an undirected graph contains a cycle.

A common DFS technique tracks the parent of each vertex.

If DFS encounters an already visited neighbor that is not the current vertex's parent, a cycle exists.

### Template

```python
def has_cycle_undirected(graph):
    visited = set()

    def dfs(node, parent):
        visited.add(node)

        for neighbor in graph[node]:
            if neighbor not in visited:
                if dfs(neighbor, node):
                    return True
            elif neighbor != parent:
                return True

        return False

    for node in graph:
        if node not in visited:
            if dfs(node, -1):
                return True

    return False
```

This template assumes vertex labels do not use `-1` as a graph vertex. A unique sentinel object or a separate parent representation can avoid that assumption.

### Complexity

- Time: \(O(V+E)\)
- Space: \(O(V)\)

**Important:** The parent check is necessary because every undirected edge appears in both adjacency lists. Seeing the edge back to the parent does not itself indicate a cycle.

For large or deep graphs, use an iterative approach or Union-Find to avoid recursion-depth issues.

---

## 9. Pattern 7: Cycle Detection in a Directed Graph

**Use when:** You need to detect circular dependencies in a directed graph.

DFS can track three states:

- `0`: Not visited
- `1`: Currently being explored
- `2`: Fully processed

A directed edge to a vertex in state `1` indicates a cycle.

### Template

```python
def has_cycle_directed(graph):
    state = {}

    def dfs(node):
        state[node] = 1

        for neighbor in graph[node]:
            if state.get(neighbor, 0) == 1:
                return True

            if state.get(neighbor, 0) == 0:
                if dfs(neighbor):
                    return True

        state[node] = 2
        return False

    for node in graph:
        if state.get(node, 0) == 0:
            if dfs(node):
                return True

    return False
```

### Complexity

- Time: \(O(V+E)\)
- Space: \(O(V)\)

### Typical Applications

- Detecting circular task dependencies
- Course Schedule
- Package dependency validation
- Build-system dependency checks

**Remember:** In a directed graph, merely reaching a previously visited vertex does not prove that a cycle exists. The vertex must be on the current DFS path.

---

## 10. Pattern 8: Topological Sorting

**Use when:** You need a valid ordering of tasks subject to directed dependencies.

Kahn's algorithm repeatedly processes vertices whose in-degree is zero.

### Template

```python
from collections import deque

def topological_sort(graph):
    indegree = {node: 0 for node in graph}

    for node in graph:
        for neighbor in graph[node]:
            indegree[neighbor] += 1

    queue = deque(
        node for node in graph
        if indegree[node] == 0
    )

    order = []

    while queue:
        node = queue.popleft()
        order.append(node)

        for neighbor in graph[node]:
            indegree[neighbor] -= 1

            if indegree[neighbor] == 0:
                queue.append(neighbor)

    return order if len(order) == len(graph) else None
```

### Complexity

- Time: \(O(V+E)\)
- Space: \(O(V)\)

If the result contains fewer than \(V\) vertices, the directed graph contains a cycle and no topological ordering exists.

### Typical Applications

- Course Schedule II
- Task scheduling
- Build systems
- Data pipeline dependencies

---

## 11. Pattern 9: Shortest Path in a Weighted Graph

**Use when:** Edges have different costs or distances.

Choose the algorithm based on the edge weights.

| Graph condition | Algorithm |
|---|---|
| All edges have equal cost | BFS |
| Non-negative edge weights | Dijkstra's algorithm |
| Negative edge weights may exist | Bellman–Ford |
| Directed acyclic graph | Topological-order shortest paths can work, including with negative weights |

### Dijkstra's Algorithm

Dijkstra's algorithm uses a min-heap to process the currently closest known vertex.

```python
import heapq

def dijkstra(graph, source):
    # graph[u] contains (neighbor, weight) pairs.
    distances = {source: 0}
    heap = [(0, source)]

    while heap:
        distance, node = heapq.heappop(heap)

        if distance != distances[node]:
            continue

        for neighbor, weight in graph.get(node, []):
            new_distance = distance + weight

            if (
                neighbor not in distances
                or new_distance < distances[neighbor]
            ):
                distances[neighbor] = new_distance
                heapq.heappush(
                    heap, (new_distance, neighbor)
                )

    return distances
```

### Complexity

With an adjacency list and binary heap, the usual complexity is \(O((V+E)\log V)\), commonly written as \(O(E\log V)\) for connected graphs with at least one edge.

**Important:** Dijkstra's algorithm requires non-negative edge weights. It does not correctly handle arbitrary negative-weight edges.

---

## 12. Pattern 10: Union-Find (Disjoint Set Union)

**Use when:** You need to efficiently determine whether vertices belong to the same connected group, especially while processing edges.

Union-Find maintains disjoint sets using two main operations:

- `find(x)`: Identify the representative of the set containing `x`.
- `union(a, b)`: Merge the sets containing `a` and `b`.

### Template

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.size = [1] * n

    def find(self, x):
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]
            x = self.parent[x]

        return x

    def union(self, a, b):
        root_a = self.find(a)
        root_b = self.find(b)

        if root_a == root_b:
            return False

        if self.size[root_a] < self.size[root_b]:
            root_a, root_b = root_b, root_a

        self.parent[root_b] = root_a
        self.size[root_a] += self.size[root_b]

        return True
```

With path compression and union by size, each operation takes nearly constant amortized time, \(O(\alpha(n))\), where \(\alpha\) is the inverse Ackermann function.

### Typical Applications

- Counting connected components
- Detecting cycles in an undirected graph
- Kruskal's minimum spanning tree algorithm
- Processing connectivity queries

---

## 13. Pattern 11: Bipartite Graph Checking

A graph is bipartite if its vertices can be divided into two groups such that every edge connects vertices from different groups.

A common technique is BFS coloring.

```python
from collections import deque

def is_bipartite(graph):
    color = {}

    for start in graph:
        if start in color:
            continue

        color[start] = 0
        queue = deque([start])

        while queue:
            node = queue.popleft()

            for neighbor in graph[node]:
                if neighbor not in color:
                    color[neighbor] = 1 - color[node]
                    queue.append(neighbor)

                elif color[neighbor] == color[node]:
                    return False

    return True
```

### Complexity

- Time: \(O(V+E)\)
- Space: \(O(V)\)

A graph is bipartite if and only if it contains no odd-length cycle.

Applications include team assignment, conflict-free grouping, and checking whether a graph can be colored using two colors.

---

## 14. Pattern 12: Backtracking on Graphs

**Use when:** You must explore possible paths or arrangements, rather than simply visit each vertex once.

Examples include:

- Finding all simple paths between two vertices
- Word Search in a grid
- Hamiltonian path problems
- Exploring constrained routes

Unlike ordinary DFS, backtracking often removes a vertex from the current path's visited set after exploring its possible continuations.

```python
def all_simple_paths(graph, source, target):
    result = []
    path = [source]
    visited = {source}

    def dfs(node):
        if node == target:
            result.append(path.copy())
            return

        for neighbor in graph[node]:
            if neighbor in visited:
                continue

            visited.add(neighbor)
            path.append(neighbor)

            dfs(neighbor)

            path.pop()
            visited.remove(neighbor)

    dfs(source)
    return result
```

This finds all simple paths in a finite graph. The number of such paths can be exponential, so the runtime can be much larger than \(O(V+E)\).

**Important:** Use backtracking when the problem requires exploring different candidate paths. Do not remove nodes from the visited set during an ordinary traversal unless the algorithm specifically requires it.

---

## 15. How to Choose the Right Pattern

When reading a graph problem, ask these questions in order:

1. **What does each vertex represent?** A city, course, cell, person, or state?
2. **What does each edge represent?** A connection, dependency, movement, or cost?
3. **Is the graph directed or undirected?**
4. **Are the edges weighted?**
5. **Can the graph be disconnected?**
6. **What is the required output?** Reachability, count, ordering, shortest path, cycle detection, or all possible paths?

Use this quick guide:

| Requirement | Start with |
|---|---|
| Visit reachable vertices | BFS or DFS |
| Count connected groups | BFS, DFS, or Union-Find |
| Shortest path with equal edge costs | BFS |
| Shortest path with non-negative weights | Dijkstra |
| Detect an undirected cycle | DFS with parent or Union-Find |
| Detect a directed cycle | DFS states or Kahn's algorithm |
| Order dependency tasks | Topological sorting |
| Divide vertices into two groups | Bipartite checking |
| Explore all valid paths | Backtracking |
| Find a minimum spanning tree | Kruskal or Prim |

---

## 16. Common Interview Mistakes

1. Forgetting to mark vertices as visited.
2. Assuming every graph is connected.
3. Adding reverse edges to a directed graph.
4. Using BFS for weighted shortest paths without checking edge costs.
5. Detecting directed cycles using only a visited set.
6. Forgetting to preserve isolated vertices when building the graph.
7. Confusing a path with a cycle.
8. Using recursive DFS on a graph that may exceed Python's recursion limit.
9. Returning a topological ordering without checking whether all vertices were processed.
10. Assuming every graph problem requires a new algorithm instead of recognizing a known pattern.

---

## 17. Practice Problems

Practice in the following order:

### Beginner

1. Implement BFS.
2. Implement iterative DFS.
3. Find whether a path exists between two vertices.
4. Count connected components.
5. Flood Fill.
6. Number of Islands.

### Intermediate

7. Shortest Path in an Unweighted Graph.
8. Detect Cycle in an Undirected Graph.
9. Detect Cycle in a Directed Graph.
10. Course Schedule.
11. Is Graph Bipartite?
12. Number of Provinces.

### Advanced

13. Dijkstra's Shortest Path.
14. Network Delay Time.
15. Accounts Merge using Union-Find.
16. Minimum Spanning Tree.
17. Word Search using backtracking.
18. Alien Dictionary using topological sorting.

For each problem, first write down the graph representation, traversal strategy, visited-state requirements, and expected complexity before implementing the solution.

---

## 18. Quick Revision

- **BFS:** Queue; level-by-level traversal.
- **DFS:** Stack or recursion; deep exploration.
- **Connected components:** Run traversal from each unvisited vertex.
- **Grid problems:** Treat cells as vertices and valid moves as edges.
- **Unweighted shortest path:** BFS.
- **Undirected cycle detection:** DFS with parent tracking or Union-Find.
- **Directed cycle detection:** DFS states or Kahn's algorithm.
- **Topological sorting:** In-degree queue or DFS finishing order.
- **Weighted shortest path:** Dijkstra for non-negative weights.
- **Connectivity queries:** Union-Find.
- **Bipartite checking:** BFS or DFS coloring.
- **All possible paths:** Backtracking.

For adjacency-list graph traversal, the standard time complexity is \(O(V+E)\).

**Recommended learning order:** Graph fundamentals → BFS and DFS → connected components → grid traversal → cycle detection → topological sorting → shortest paths → Union-Find → minimum spanning trees → backtracking.