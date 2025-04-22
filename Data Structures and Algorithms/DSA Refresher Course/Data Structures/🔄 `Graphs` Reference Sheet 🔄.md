## Definition 📝
A graph is a non-linear data structure consisting of a finite set of **vertices** (or nodes) and a set of **edges** connecting these vertices. Unlike trees, graphs can have cycles and a more complex structure.

Formally, a graph G is an ordered pair G = (V, E) where:
- V is a set of vertices
- E is a set of edges, which are pairs of vertices (u, v)

## Graph Terminology 📚
- **Vertex/Node**: Basic element of a graph
- **Edge**: Connection between two vertices
- **Adjacent**: Two vertices are adjacent if connected by an edge
- **Path**: Sequence of vertices connected by edges
- **Cycle**: Path that starts and ends at the same vertex
- **Degree**: Number of edges connected to a vertex
- **Connected**: Graph where there's a path between every pair of vertices

## Types of Graphs 🔍

### Directed vs Undirected Graphs
- **Undirected Graph**: Edges have no direction (two-way connection)
  ```
  A --- B
  |     |
  C --- D
  ```

- **Directed Graph (Digraph)**: Edges have direction (one-way connection)
  ```
  A --→ B
  ↑     ↓
  C ←-- D
  ```

### Weighted vs Unweighted Graphs
- **Unweighted Graph**: All edges have equal importance
  ```
  A --- B
  |     |
  C --- D
  ```

- **Weighted Graph**: Edges have associated values (weights)
  ```
  A --5-- B
  |       |
  2       7
  |       |
  C --1-- D
  ```

### Other Types
- **Complete Graph**: Every vertex is connected to every other vertex
- **Bipartite Graph**: Vertices can be divided into two groups with no edges within the same group
- **Tree**: Connected graph with no cycles
- **DAG (Directed Acyclic Graph)**: Directed graph with no cycles

## Graph Representations ⚙️

### Adjacency Matrix
A 2D array where matrix[i][j] represents an edge from vertex i to vertex j.

```python
# Adjacency Matrix representation
graph = [
    [0, 1, 1, 0],  # Connections from vertex 0
    [1, 0, 0, 1],  # Connections from vertex 1
    [1, 0, 0, 1],  # Connections from vertex 2
    [0, 1, 1, 0]   # Connections from vertex 3
]
```

#### Pros and Cons of Adjacency Matrix
✅ **Pros**:
- Simple to implement
- Edge lookup is O(1)
- Good for dense graphs

❌ **Cons**:
- Uses O(V²) space regardless of edges
- Inefficient for sparse graphs
- Vertex operations are O(V)

### Adjacency List
An array of lists where each list contains the neighbors of the corresponding vertex.

```python
# Adjacency List representation
graph = {
    0: [1, 2],     # Vertex 0 is connected to 1, 2
    1: [0, 3],     # Vertex 1 is connected to 0, 3
    2: [0, 3],     # Vertex 2 is connected to 0, 3
    3: [1, 2]      # Vertex 3 is connected to 1, 2
}
```

#### Pros and Cons of Adjacency List
✅ **Pros**:
- Space-efficient for sparse graphs
- Faster to iterate over edges
- Vertex operations are faster

❌ **Cons**:
- Edge lookup is O(E) in worst case
- More complex to implement
- Less efficient for dense graphs

## Complexity Analysis ⏱️

| Operation | Adjacency Matrix | Adjacency List |
|-----------|------------------|---------------|
| Add Vertex | O(V²) | O(1) |
| Add Edge | O(1) | O(1) |
| Remove Vertex | O(V²) | O(V+E) |
| Remove Edge | O(1) | O(E) |
| Query | O(1) | O(V) |
| Storage | O(V²) | O(V+E) |

## Graph Traversal Algorithms 🚶

### Breadth-First Search (BFS)
Explores all neighbors at the current depth before moving to nodes at the next depth level.

```python
from collections import deque

def bfs(graph, start):
    visited = set()
    queue = deque([start])
    visited.add(start)
    
    while queue:
        vertex = queue.popleft()
        print(vertex, end=" ")  # Process vertex
        
        # Visit all neighbors
        for neighbor in graph[vertex]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
```

### Depth-First Search (DFS)
Explores as far as possible along each branch before backtracking.

```python
def dfs(graph, start, visited=None):
    if visited is None:
        visited = set()
    
    visited.add(start)
    print(start, end=" ")  # Process vertex
    
    # Visit all neighbors
    for neighbor in graph[start]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)
```

## Connected Components 🧩
Connected components are subgraphs where any two vertices are connected by a path, but no path exists between vertices in different components.

```python
def find_connected_components(graph):
    visited = set()
    components = []
    
    for vertex in graph:
        if vertex not in visited:
            component = []
            dfs_component(graph, vertex, visited, component)
            components.append(component)
    
    return components

def dfs_component(graph, vertex, visited, component):
    visited.add(vertex)
    component.append(vertex)
    
    for neighbor in graph[vertex]:
        if neighbor not in visited:
            dfs_component(graph, neighbor, visited, component)
```

## Common Graph Algorithms 🧠

### Shortest Path Algorithms
- **Dijkstra's Algorithm**: Finds shortest path in weighted graphs with non-negative weights
- **Bellman-Ford Algorithm**: Handles negative weights and detects negative cycles
- **Floyd-Warshall Algorithm**: Finds shortest paths between all pairs of vertices

### Minimum Spanning Tree Algorithms
- **Kruskal's Algorithm**: Builds MST by adding edges in order of increasing weight
- **Prim's Algorithm**: Builds MST by growing a single tree from a starting vertex

### Other Important Algorithms
- **Topological Sort**: Linear ordering of vertices in a DAG
- **Strongly Connected Components**: Kosaraju's or Tarjan's algorithm
- **Cycle Detection**: Using DFS or Union-Find

## Implementation 💻

### Basic Graph Class (Adjacency List)

```python
class Graph:
    def __init__(self, directed=False):
        self.graph = {}
        self.directed = directed
    
    def add_vertex(self, vertex):
        if vertex not in self.graph:
            self.graph[vertex] = []
    
    def add_edge(self, vertex1, vertex2, weight=None):
        if vertex1 not in self.graph:
            self.add_vertex(vertex1)
        if vertex2 not in self.graph:
            self.add_vertex(vertex2)
            
        # For weighted graphs, you can use (vertex, weight) tuples
        self.graph[vertex1].append(vertex2)
        
        # If undirected, add the reverse edge too
        if not self.directed:
            self.graph[vertex2].append(vertex1)
    
    def get_vertices(self):
        return list(self.graph.keys())
    
    def get_edges(self):
        edges = []
        for vertex in self.graph:
            for neighbor in self.graph[vertex]:
                if self.directed or neighbor > vertex:
                    edges.append((vertex, neighbor))
        return edges
    
    def __str__(self):
        result = "Vertices: " + str(self.get_vertices()) + "\n"
        result += "Edges: " + str(self.get_edges())
        return result
```

## Real-world Applications 🌐
- **Social Networks**: People as vertices, connections as edges
- **Web Pages**: Pages as vertices, hyperlinks as edges
- **Maps and Navigation**: Locations as vertices, roads as edges
- **Network Routing**: Devices as vertices, connections as edges
- **Recommendation Systems**: Users/items as vertices, preferences as edges
- **Dependency Resolution**: Tasks as vertices, dependencies as edges

## Advantages and Disadvantages ⚖️

### Advantages ✅
- Flexible representation for many real-world problems
- Can model complex relationships and systems
- Many efficient algorithms for different problems
- Can represent both hierarchical and non-hierarchical data

### Disadvantages ❌
- Can be complex to implement and understand
- Some graph algorithms have high time complexity
- Memory-intensive for large graphs
- Choosing the right representation is important

## Further Reading 📚
- Books:
  - "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein
  - "Algorithms" by Robert Sedgewick and Kevin Wayne
  - "Graph Theory and Its Applications" by Jonathan L. Gross and Jay Yellen

- Online Resources:
  - [Visualgo Graph Algorithms](https://visualgo.net/en/graphds)
  - [GeeksforGeeks Graph Data Structure](https://www.geeksforgeeks.org/graph-data-structure-and-algorithms/)
  - [Khan Academy Algorithms Course](https://www.khanacademy.org/computing/computer-science/algorithms)

- Interactive Tools:
  - [Graph Editor](https://csacademy.com/app/graph_editor/)
  - [D3 Graph Theory](https://d3gt.com/)

---