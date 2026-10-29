# Topological Sort

## 1. What Is Topological Sorting?

**Topological sorting** is a linear ordering of vertices in a **directed acyclic graph (DAG)** such that for every directed edge \(u \rightarrow v\), vertex \(u\) appears before vertex \(v\).

In simple terms, if one task depends on another, the prerequisite task must come first.

### Example

Consider these task dependencies:

```text
A ----> C
|
v
B ----> D
```

Edges:
- `A → B`
- `A → C`
- `B → D`

One valid topological ordering is:

```text
A, B, C, D
```

Another valid ordering is:

```text
A, C, B, D
```

Both are correct because every prerequisite appears before its dependent task.

**Important:** A DAG can have multiple valid topological orderings.

---

## 2. Where Is Topological Sorting Used?

Topological sorting is useful when tasks have dependencies.

Common applications include:

- Scheduling tasks with prerequisites.
- Ordering modules during compilation.
- Resolving package dependencies.
- Determining the execution order of data pipelines.
- Scheduling courses based on prerequisites.
- Detecting cycles in directed graphs.

For example, a data pipeline might have these dependencies:

```text
Extract Data
     |
     v
Clean Data
     |
     v
Transform Data
     |
     v
Load Data
```

The tasks must run in dependency order. Topological sorting helps determine a valid execution sequence.

---

## 3. Conditions for Topological Sorting

Topological sorting is defined for a **directed acyclic graph**.

### Directed graph

Edges have directions.

```text
A ----> B
```

This indicates that `A` must precede `B`.

### Acyclic graph

The graph must not contain a directed cycle.

For example:

```text
A ----> B
^       |
|       v
+-------C
```

This represents:

```text
A → B → C → A
```

There is no valid topological ordering because every vertex depends, directly or indirectly, on the next one.

**Key rule:** If a directed graph contains a cycle, it has no topological ordering.

---

## 4. Approach 1: Kahn's Algorithm (BFS)

Kahn's algorithm uses **in-degree** and a queue.

### What Is In-Degree?

The in-degree of a vertex is the number of incoming edges.

For example:

```text
A ----> C <---- B
        |
        v
        D
```

- `in_degree(A) = 0`
- `in_degree(B) = 0`
- `in_degree(C) = 2`
- `in_degree(D) = 1`

A vertex with in-degree `0` has no remaining prerequisites and can be processed next.

### Algorithm

1. Calculate the in-degree of every vertex.
2. Add every vertex with in-degree `0` to a queue.
3. Remove a vertex from the queue and append it to the result.
4. For each outgoing neighbor, decrease its in-degree by `1`.
5. If a neighbor's in-degree becomes `0`, add it to the queue.
6. Continue until the queue is empty.
7. If the result contains all vertices, the graph is acyclic and the result is a valid topological ordering.
8. Otherwise, the graph contains a directed cycle.

### Python Implementation

```python
from collections import deque

def topological_sort(graph):
    indegree = {node: 0 for node in graph}

    # Calculate in-degree of every vertex.
    for node in graph:
        for neighbor in graph[node]:
            indegree[neighbor] += 1

    # Start with vertices that have no prerequisites.
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

    # Not all vertices could be processed.
    if len(order) != len(graph):
        return None

    return order
```

### Example

```python
graph = {
    "A": ["B", "C"],
    "B": ["D"],
    "C": ["D"],
    "D": []
}

print(topological_sort(graph))
```

One possible output:

```text
['A', 'B', 'C', 'D']
```

The exact ordering can vary depending on the order in which vertices enter the queue.

### Complexity

Let \(V\) be the number of vertices and \(E\) the number of edges.

- **Time:** \(O(V+E)\)
- **Space:** \(O(V)\) auxiliary space, excluding the graph itself.

Each vertex is added to and removed from the queue at most once, and each edge is processed once.

---

## 5. Approach 2: DFS-Based Topological Sort

DFS-based topological sorting uses the order in which vertices finish processing.

### Main Idea

When DFS explores a vertex, it first processes its unvisited neighbors. Once all its neighbors are finished, it adds the current vertex to a result list.

Finally, reverse that list.

### Python Implementation

```python
def topological_sort_dfs(graph):
    visited = set()
    active = set()
    order = []

    def dfs(node):
        if node in active:
            return False

        if node in visited:
            return True

        active.add(node)

        for neighbor in graph[node]:
            if not dfs(neighbor):
                return False

        active.remove(node)
        visited.add(node)
        order.append(node)

        return True

    for node in graph:
        if node not in visited:
            if not dfs(node):
                return None

    return order[::-1]
```

### Example

```python
graph = {
    "A": ["B", "C"],
    "B": ["D"],
    "C": ["D"],
    "D": []
}

print(topological_sort_dfs(graph))
```

One possible output:

```text
['A', 'C', 'B', 'D']
```

This is valid because `A` appears before `B` and `C`, and both appear before `D`.

### Why Use Two Sets?

- `visited`: Vertices whose DFS processing is complete.
- `active`: Vertices currently on the DFS recursion path.

If DFS reaches a vertex already in `active`, it has found a directed cycle.

This implementation assumes every neighbor appears as a key in `graph`. For a graph constructed from edge lists, initialize all vertices, including those with no outgoing edges.

### Complexity

- **Time:** \(O(V+E)\)
- **Space:** \(O(V)\) auxiliary space, including visited sets and the result.

Recursive DFS can exceed Python's recursion limit on very deep graphs. Kahn's algorithm or an iterative DFS may be preferable for large inputs.

---

## 6. Kahn's Algorithm vs. DFS

| Feature | Kahn's algorithm | DFS-based approach |
|---|---|---|
| Main technique | BFS-style queue | Depth-first search |
| Key concept | In-degree | Completion order and recursion states |
| Cycle detection | Result contains fewer than \(V\) vertices | Edge reaches a vertex in the active recursion path |
| Time complexity | \(O(V+E)\) | \(O(V+E)\) |
| Auxiliary space | \(O(V)\) | \(O(V)\) |
| Common use | Scheduling and dependency processing | DFS-oriented graph problems |

Both approaches are correct for a DAG. Choose the one that is easiest to reason about for the problem.

---

## 7. Important Interview Pattern: Course Schedule

A common interview problem asks whether all courses can be completed given prerequisite relationships.

Suppose:

```python
num_courses = 4

prerequisites = [
    [1, 0],
    [2, 0],
    [3, 1],
    [3, 2]
]
```

Each pair `[course, prerequisite]` means the prerequisite must be completed first.

The dependencies are:

```text
0 → 1
0 → 2
1 → 3
2 → 3
```

### Determine Whether All Courses Can Be Completed

```python
from collections import deque

def can_finish(num_courses, prerequisites):
    graph = [[] for _ in range(num_courses)]
    indegree = [0] * num_courses

    for course, prerequisite in prerequisites:
        graph[prerequisite].append(course)
        indegree[course] += 1

    queue = deque(
        course
        for course in range(num_courses)
        if indegree[course] == 0
    )

    completed = 0

    while queue:
        course = queue.popleft()
        completed += 1

        for neighbor in graph[course]:
            indegree[neighbor] -= 1

            if indegree[neighbor] == 0:
                queue.append(neighbor)

    return completed == num_courses
```

Example:

```python
print(can_finish(4, [[1, 0], [2, 0], [3, 1], [3, 2]]))
# True
```

If the prerequisites contain a cycle, not all courses can be completed.

For example:

```python
print(can_finish(2, [[1, 0], [0, 1]]))
# False
```

**Interview insight:** If the question asks only whether all tasks can be completed, you do not need to return the full topological order. Counting processed vertices is enough.

---

## 8. Common Mistakes

### Mistake 1: Applying topological sorting to an undirected graph

Topological sorting is defined for directed acyclic graphs, not ordinary undirected graphs.

### Mistake 2: Forgetting isolated vertices

A vertex with no incoming or outgoing edges must still appear in the result.

If the graph is represented using an adjacency list, include every vertex as a key, even if its neighbor list is empty.

### Mistake 3: Building the graph in the wrong direction

If `[course, prerequisite]` means that the prerequisite must come first, create:

```python
graph[prerequisite].append(course)
```

Reversing this direction reverses the intended dependency relationship.

### Mistake 4: Forgetting to decrease in-degree

For each processed vertex, decrease the in-degree of every outgoing neighbor.

Otherwise, dependent vertices may never enter the queue.

### Mistake 5: Assuming the answer is unique

A DAG can have multiple valid topological orderings. Do not expect one exact ordering unless the problem specifies a tie-breaking rule.

### Mistake 6: Failing to detect cycles

If Kahn's algorithm processes fewer than \(V\) vertices, the graph contains a directed cycle.

For DFS, detecting a previously visited vertex alone is insufficient. Use an active recursion-path set or three-state coloring.

---

## 9. Interview Questions

1. What is topological sorting?
2. Why must the graph be a DAG?
3. What is the difference between BFS and Kahn's algorithm?
4. How does in-degree help determine which vertex to process?
5. How can Kahn's algorithm detect a cycle?
6. How does DFS-based topological sorting work?
7. Can a DAG have multiple valid topological orderings?
8. What is the time complexity of topological sorting?
9. How would you solve the Course Schedule problem?
10. How would you return the actual course ordering instead of a Boolean?
11. How would you detect whether a directed graph contains a cycle?
12. How is topological sorting useful in data pipeline scheduling?

### Short Answers to Remember

- **Topological sort:** An ordering where every directed edge \(u \rightarrow v\) places \(u\) before \(v\).
- **Required graph:** A directed acyclic graph.
- **Kahn's algorithm:** Repeatedly process vertices with in-degree zero.
- **DFS method:** Append vertices after their descendants finish, then reverse the result.
- **Cycle detection with Kahn's algorithm:** Processed vertex count is less than \(V\).
- **Time complexity:** \(O(V+E)\).
- **Uniqueness:** Not guaranteed; a DAG may have several valid orders.

---

## 10. Practice Problems

Practice in this order:

1. Implement topological sorting using Kahn's algorithm.
2. Implement topological sorting using DFS.
3. Detect a cycle in a directed graph.
4. Solve Course Schedule: determine whether all courses can be completed.
5. Solve Course Schedule II: return a valid course order.
6. Find all vertices that can be processed when some dependencies form a cycle.
7. Determine whether a topological ordering is unique.
8. Use topological sorting to schedule tasks with dependencies.
9. Find the longest path in a DAG.
10. Solve the Alien Dictionary problem.

For each problem, identify the direction of the edges, include all vertices, and decide whether the output requires a valid ordering, a cycle check, or both.

---

## 11. Quick Revision

- Topological sorting orders tasks according to dependencies.
- It applies to directed acyclic graphs (DAGs).
- Kahn's algorithm uses in-degree and a queue.
- DFS-based sorting uses completion order and reverses the result.
- Both algorithms run in \(O(V+E)\) time.
- A directed cycle makes topological ordering impossible.
- Multiple valid orderings may exist.
- Always include isolated vertices.
- For Course Schedule, count the number of processed courses.
- For Course Schedule II, return the actual topological ordering.

**Recommended learning order:** Graph fundamentals → BFS and DFS → cycle detection → topological sorting → Course Schedule → shortest paths → Union-Find.