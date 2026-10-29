# Graph Fundamentals

## 1. What Is a Graph?

A **graph** is a non-linear data structure made up of vertices (nodes) and edges (connections between nodes).

A graph is represented as:

\[
G = (V, E)
\]

- **V (Vertices):** The set of nodes.
- **E (Edges):** The set of connections between nodes.

Graphs are used to represent relationships between objects, such as cities connected by roads, users connected on social media, and servers connected in a network.

### Example

Consider a graph with four nodes:

```text
    A ----- B
    |       |
    |       |
    C ----- D
```

Vertices: `A, B, C, D`

Edges: `(A, B), (A, C), (B, D), (C, D)`

The edge `(A, B)` means that nodes `A` and `B` are connected.

---

## 2. Important Graph Terminology

| Term | Meaning |
|---|---|
| Vertex / Node | An individual element in a graph |
| Edge | A connection between two vertices |
| Adjacent vertices | Two vertices connected by an edge |
| Degree | Number of edges connected to a vertex in an undirected graph |
| In-degree | Number of incoming edges in a directed graph |
| Out-degree | Number of outgoing edges in a directed graph |
| Path | A sequence of vertices connected by edges |
| Cycle | A path that starts and ends at the same vertex |
| Connected graph | Every pair of vertices has a path between them |
| Connected component | A maximal group of vertices connected to one another |
| Weight | A value associated with an edge, such as distance or cost |
| Self-loop | An edge connecting a vertex to itself |
| Isolated vertex | A vertex with no edges |

### Example

```text
A ----- B ----- C

D
```

- `degree(A) = 1`
- `degree(B) = 2`
- `degree(C) = 1`
- `D` is an isolated vertex.
- `{A, B, C}` forms one connected component.
- `D` forms another connected component.

**Important:** In an undirected graph, the sum of all vertex degrees equals twice the number of edges.

\[
\sum_{v \in V}\deg(v)=2|E|
\]

Each edge contributes one degree to each endpoint.

---

## 3. Types of Graphs

### 3.1 Undirected Graph

Edges have no direction. If `A` is connected to `B`, you can travel from `A` to `B` and from `B` to `A`.

```text
A ----- B
```

Example: A mutual friendship network.

### 3.2 Directed Graph (Digraph)

Edges have a direction, represented by arrows.

```text
A ----> B
```

This means there is an edge from `A` to `B`. It does not automatically mean that `B` can reach `A`.

Example: Following someone on social media.

### 3.3 Weighted Graph

Each edge has a numerical value called a weight.

```text
A --5-- B
 \      |
  2     3
   \    |
     C
```

The weights could represent distance, travel time, cost, or network latency.

### 3.4 Unweighted Graph

Edges represent connections without assigning numerical weights.

Example: A simple friendship network.

### 3.5 Cyclic and Acyclic Graphs

A **cyclic graph** contains at least one cycle.

```text
A ----- B
|       |
C ----- D
```

An **acyclic graph** contains no cycles.

A directed acyclic graph is called a **DAG**. DAGs are commonly used for task scheduling, dependency resolution, and data pipelines.

### 3.6 Connected and Disconnected Graphs

A graph is connected if every vertex can be reached from every other vertex.

```text
A ----- B       C ----- D
```

This graph is disconnected because there is no path between the two groups.

For directed graphs, strong connectivity is stricter: every vertex must be reachable from every other vertex following edge directions.

### 3.7 Complete Graph

A simple undirected graph is complete when every pair of distinct vertices has an edge.

For a complete graph with \(n\) vertices:

\[
|E|=\frac{n(n-1)}{2}
\]

For example, a complete graph with four vertices has six edges.

### 3.8 Tree

A tree is a connected, undirected graph with no cycles.

For a tree containing \(V\) vertices:

\[
E=V-1
\]

A tree is a special type of graph. A forest is an undirected acyclic graph that may contain multiple disconnected trees.

---

## 4. Graph Representation in Python

Graphs are commonly represented using an adjacency list or an adjacency matrix.

### 4.1 Adjacency List

An adjacency list stores each vertex along with its neighboring vertices.

Consider this graph:

```text
A ----- B
|       |
C ----- D
```

Its adjacency list is:

```python
graph = {
    "A": ["B", "C"],
    "B": ["A", "D"],
    "C": ["A", "D"],
    "D": ["B", "C"]
}
```

To access the neighbors of `A`:

```python
print(graph["A"])
# ['B', 'C']
```

To iterate through the neighbors of a vertex:

```python
for neighbor in graph["A"]:
    print(neighbor)
```

**Time complexity:** Accessing a vertex's neighbor list is typically \(O(1)\), and iterating through its neighbors takes \(O(\deg(v))\).

**Space complexity:** \(O(V+E)\).

This is generally the preferred representation for sparse graphs, where the number of edges is relatively small.

### 4.2 Adjacency Matrix

An adjacency matrix uses a two-dimensional matrix to represent connections between vertices.

For the graph above, use the vertex order `A, B, C, D`:

```text
    A B C D
A   0 1 1 0
B   1 0 0 1
C   1 0 0 1
D   0 1 1 0
```

Python implementation:

```python
matrix = [
    [0, 1, 1, 0],
    [1, 0, 0, 1],
    [1, 0, 0, 1],
    [0, 1, 1, 0]
]
```

Here, `matrix[i][j] == 1` means an edge exists between vertices `i` and `j`.

For a weighted graph, the matrix can store edge weights instead of `1`. A separate convention is needed to distinguish missing edges from valid zero-weight edges.

**Space complexity:** \(O(V^2)\).

**Edge lookup:** \(O(1)\).

Adjacency matrices are useful for dense graphs, where many possible edges exist.

### 4.3 Comparison

| Operation | Adjacency list | Adjacency matrix |
|---|---:|---:|
| Space | \(O(V+E)\) | \(O(V^2)\) |
| Check whether an edge exists | \(O(\deg(v))\) | \(O(1)\) |
| Iterate over neighbors | \(O(\deg(v))\) | \(O(V)\) |
| Add an edge | \(O(1)\) average | \(O(1)\) |
| Best suited for | Sparse graphs | Dense graphs |

For an adjacency list, edge lookup is \(O(\deg(v))\) in the usual list-based implementation. A set-based neighbor representation can improve lookup at the cost of different memory and iteration characteristics.

---

## 5. Building a Graph from an Edge List

In coding interviews, graphs are frequently provided as a list of edges.

Example input:

```python
edges = [
    ("A", "B"),
    ("A", "C"),
    ("B", "D"),
    ("C", "D")
]
```

Build an undirected adjacency list:

```python
from collections import defaultdict

def build_graph(edges):
    graph = defaultdict(list)

    for u, v in edges:
        graph[u].append(v)
        graph[v].append(u)

    return dict(graph)


edges = [
    ("A", "B"),
    ("A", "C"),
    ("B", "D"),
    ("C", "D")
]

graph = build_graph(edges)
print(graph)
```

The important operation is:

```python
graph[u].append(v)
graph[v].append(u)
```

Both directions are added because the graph is undirected.

For a **directed graph**, add only the outgoing edge:

```python
graph[u].append(v)
```

If the input also supplies the number of vertices, initialize every vertex when isolated vertices must be preserved.

---

## 6. Graph Traversal

Graph traversal means visiting vertices by following edges.

The two fundamental traversal algorithms are:

1. Breadth-First Search (BFS)
2. Depth-First Search (DFS)

### 6.1 Breadth-First Search (BFS)

BFS explores the graph level by level. It uses a queue.

```python
from collections import deque

def bfs(graph, start):
    visited = {start}
    queue = deque([start])
    order = []

    while queue:
        node = queue.popleft()
        order.append(node)

        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

    return order
```

Example:

```python
graph = {
    "A": ["B", "C"],
    "B": ["A", "D"],
    "C": ["A", "D"],
    "D": ["B", "C"]
}

print(bfs(graph, "A"))
# ['A', 'B', 'C', 'D']
```

The exact order depends on the order of neighbors in the adjacency list.

**Why mark a node visited when adding it to the queue?**

It prevents the same vertex from being added multiple times, especially when multiple vertices share a neighbor or the graph contains cycles.

**Complexity:** \(O(V+E)\) time and \(O(V)\) auxiliary space for a full traversal in an adjacency-list graph.

**When to use BFS:**
- Shortest path in an unweighted graph
- Level-order exploration
- Minimum number of edges between two vertices
- Finding connected components

### 6.2 Depth-First Search (DFS)

DFS explores one path as deeply as possible before backtracking. It can be implemented recursively or with a stack.

Recursive implementation:

```python
def dfs(graph, start):
    visited = set()
    order = []

    def visit(node):
        visited.add(node)
        order.append(node)

        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                visit(neighbor)

    visit(start)
    return order
```

Iterative implementation:

```python
def dfs_iterative(graph, start):
    visited = {start}
    stack = [start]
    order = []

    while stack:
        node = stack.pop()
        order.append(node)

        for neighbor in reversed(graph.get(node, [])):
            if neighbor not in visited:
                visited.add(neighbor)
                stack.append(neighbor)

    return order
```

**Complexity:** \(O(V+E)\) time and \(O(V)\) auxiliary space in the usual adjacency-list traversal.

**When to use DFS:**
- Cycle detection
- Connected components
- Path exploration
- Topological sorting
- Backtracking-style graph problems

**Important:** Recursive DFS can exceed Python's recursion limit on deep graphs. Use iterative DFS when the graph may be very deep.

---

## 7. Disconnected Graphs

A traversal starting from one vertex visits only the vertices reachable from that starting point.

To traverse every component, loop over all vertices:

```python
def dfs_all_components(graph):
    visited = set()
    components = []

    for node in graph:
        if node not in visited:
            component = []
            stack = [node]
            visited.add(node)

            while stack:
                current = stack.pop()
                component.append(current)

                for neighbor in graph.get(current, []):
                    if neighbor not in visited:
                        visited.add(neighbor)
                        stack.append(neighbor)

            components.append(component)

    return components
```

Example:

```python
graph = {
    1: [2],
    2: [1],
    3: [4],
    4: [3],
    5: []
}

print(dfs_all_components(graph))
# [[1, 2], [3, 4], [5]]
```

The component ordering depends on the input iteration order and traversal order.

This pattern is useful for counting connected components in an undirected graph.

---

## 8. Common Graph Patterns

| Problem | Common technique |
|---|---|
| Visit all reachable nodes | BFS or DFS |
| Shortest path in an unweighted graph | BFS |
| Count connected components | BFS or DFS |
| Detect a cycle in an undirected graph | DFS with parent tracking or Union-Find |
| Detect a cycle in a directed graph | DFS with recursion states or topological sorting |
| Shortest path with non-negative weights | Dijkstra's algorithm |
| Shortest path with negative edge weights | Bellman–Ford |
| Dependency ordering | Topological sorting |
| Minimum spanning tree | Kruskal's or Prim's algorithm |
| Dynamic connectivity | Disjoint Set Union (Union-Find) |

Do not try to memorize every algorithm at once. First become comfortable with graph representation, BFS, DFS, visited sets, and disconnected components.

---

## 9. Common Mistakes

### Mistake 1: Forgetting the reverse edge

Incorrect for an undirected graph:

```python
graph[u].append(v)
```

Correct:

```python
graph[u].append(v)
graph[v].append(u)
```

For a directed graph, only the first line is needed.

### Mistake 2: Not using a visited set

A cyclic graph can cause repeated visits or an infinite loop.

Always track visited vertices when traversing a graph.

### Mistake 3: Marking nodes visited too late in BFS

Mark a node as visited when it is added to the queue, not only when it is removed. This avoids duplicate queue entries.

### Mistake 4: Assuming every graph is connected

A starting-node traversal may miss isolated vertices and other components. Iterate through all vertices when the problem requires processing the entire graph.

### Mistake 5: Confusing a tree with a graph

Every tree is a graph, but not every graph is a tree. An undirected tree must be connected and acyclic.

### Mistake 6: Ignoring edge direction

In a directed graph, an edge from `A` to `B` does not imply an edge from `B` to `A`.

### Mistake 7: Using BFS for every shortest-path problem

BFS finds shortest paths by number of edges in unweighted graphs, or equivalently when all edges have equal cost. Weighted graphs may require Dijkstra's algorithm or another shortest-path algorithm.

---

## 10. Interview Questions

1. What is a graph, and how does it differ from a tree?
2. What is the difference between directed and undirected graphs?
3. What is the difference between an adjacency list and an adjacency matrix?
4. When would you use an adjacency list instead of an adjacency matrix?
5. Why do BFS and DFS need a visited set?
6. Why does BFS find the shortest path in an unweighted graph?
7. What is the time complexity of BFS and DFS using an adjacency list?
8. How do you count connected components?
9. What is the difference between a cycle and a connected component?
10. What is a DAG, and where is it useful?
11. How would you detect a cycle in an undirected graph?
12. Why can recursive DFS fail on a very deep graph?

### Short answers to remember

- **BFS:** Queue; explores level by level.
- **DFS:** Stack or recursion; explores deeply before backtracking.
- **Unweighted shortest path:** BFS.
- **Adjacency-list traversal:** \(O(V+E)\).
- **Adjacency-matrix space:** \(O(V^2)\).
- **Undirected tree:** Connected and acyclic.
- **Tree edges:** \(V-1\).
- **DAG:** Directed acyclic graph.
- **Connected components:** Maximal connected groups of vertices in an undirected graph.

---

## 11. Practice Problems

Practice in this order:

1. Build an adjacency list from an edge list.
2. Implement BFS from a given starting vertex.
3. Implement iterative DFS.
4. Determine whether a path exists between two vertices.
5. Count connected components in an undirected graph.
6. Find the shortest path in an unweighted graph.
7. Detect a cycle in an undirected graph.
8. Count the number of islands in a grid.
9. Check whether a graph is bipartite.
10. Find a valid ordering of tasks with dependencies.

For each problem, first identify the graph representation, whether the graph is directed, whether it is weighted, and whether disconnected components are possible.

---

## 12. Quick Revision

Before moving to advanced graph algorithms, make sure you can explain and implement:

- [ ] Vertices, edges, degree, paths, cycles, and components
- [ ] Directed, undirected, weighted, and unweighted graphs
- [ ] Adjacency lists and adjacency matrices
- [ ] Building an adjacency list from an edge list
- [ ] BFS using `collections.deque`
- [ ] DFS using recursion and an explicit stack
- [ ] A visited set and its correct usage
- [ ] Traversing disconnected components
- [ ] The \(O(V+E)\) traversal complexity

**Recommended learning order:** Graph fundamentals → BFS and DFS → connected components → cycle detection → topological sorting → shortest paths → Union-Find → minimum spanning trees.