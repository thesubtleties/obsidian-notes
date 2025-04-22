## Definition 📝
Graphs are versatile data structures consisting of vertices (nodes) connected by edges. They represent relationships between objects and are used to model networks, maps, social connections, and more.

A graph G is defined as G = (V, E) where:
- V is a set of vertices
- E is a set of edges connecting pairs of vertices

## Graph Representations ⚙️

### Adjacency Matrix
```
    A B C D
  A 0 1 1 0
  B 1 0 0 1
  C 1 0 0 1
  D 0 1 1 0
```
- 2D array where matrix[i][j] = 1 if there's an edge from vertex i to j
- Space complexity: O(V²)
- Good for dense graphs

### Adjacency List
```
A -> [B, C]
B -> [A, D]
C -> [A, D]
D -> [B, C]
```
- Array of lists where each list stores adjacent vertices
- Space complexity: O(V + E)
- Good for sparse graphs

## Graph Types 🔄

### Undirected Graph
- Edges have no direction
- If vertex A connects to B, then B also connects to A

### Directed Graph (Digraph)
- Edges have direction
- If vertex A connects to B, B may not connect to A

### Weighted Graph
- Edges have weights/costs
- Used for representing distances, costs, etc.

### Cyclic vs Acyclic
- Cyclic: Contains at least one cycle
- Acyclic: Contains no cycles (e.g., trees, DAGs)

## Basic Graph Traversals 🚶

### Breadth-First Search (BFS)
```python
def bfs(graph, start):
    visited = set([start])
    queue = [start]
    
    while queue:
        vertex = queue.pop(0)
        print(vertex, end=" ")
        
        for neighbor in graph[vertex]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
```

- Uses a queue (FIFO)
- Visits vertices level by level
- Time complexity: O(V + E)
- Applications: Shortest path in unweighted graphs, connected components

### Depth-First Search (DFS)
```python
def dfs(graph, start, visited=None):
    if visited is None:
        visited = set()
    
    visited.add(start)
    print(start, end=" ")
    
    for neighbor in graph[start]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)
```

- Uses a stack (or recursion)
- Explores as far as possible along a branch
- Time complexity: O(V + E)
- Applications: Topological sorting, cycle detection, path finding

## Topological Sort 📊

A topological sort is a linear ordering of vertices in a directed acyclic graph (DAG) such that for every directed edge (u, v), vertex u comes before v in the ordering.

```python
def topological_sort(graph):
    visited = set()
    stack = []
    
    def dfs(vertex):
        visited.add(vertex)
        
        for neighbor in graph[vertex]:
            if neighbor not in visited:
                dfs(neighbor)
        
        stack.append(vertex)
    
    for vertex in graph:
        if vertex not in visited:
            dfs(vertex)
    
    return stack[::-1]  # Reverse to get topological order
```

- Only works on Directed Acyclic Graphs (DAGs)
- Time complexity: O(V + E)
- Applications: Task scheduling, course prerequisites, build systems

## Shortest Path Algorithms 🛣️

### Dijkstra's Algorithm
Finds the shortest path from a source vertex to all other vertices in a weighted graph with non-negative weights.

```python
import heapq

def dijkstra(graph, start):
    # Initialize distances with infinity for all vertices except start
    distances = {vertex: float('infinity') for vertex in graph}
    distances[start] = 0
    priority_queue = [(0, start)]
    
    while priority_queue:
        current_distance, current_vertex = heapq.heappop(priority_queue)
        
        # If we've found a longer path, ignore it
        if current_distance > distances[current_vertex]:
            continue
        
        for neighbor, weight in graph[current_vertex].items():
            distance = current_distance + weight
            
            # If we've found a shorter path, update it
            if distance < distances[neighbor]:
                distances[neighbor] = distance
                heapq.heappush(priority_queue, (distance, neighbor))
    
    return distances
```

- Uses a priority queue (min-heap)
- Time complexity: O((V + E) log V) with binary heap
- Cannot handle negative weights
- Applications: GPS navigation, network routing

### Bellman-Ford Algorithm
Finds the shortest path from a source vertex to all other vertices, even with negative edge weights.

```python
def bellman_ford(graph, start):
    # Initialize distances with infinity for all vertices except start
    distances = {vertex: float('infinity') for vertex in graph}
    distances[start] = 0
    
    # Relax all edges V-1 times
    for _ in range(len(graph) - 1):
        for u in graph:
            for v, weight in graph[u].items():
                if distances[u] + weight < distances[v]:
                    distances[v] = distances[u] + weight
    
    # Check for negative cycles
    for u in graph:
        for v, weight in graph[u].items():
            if distances[u] + weight < distances[v]:
                raise ValueError("Graph contains a negative-weight cycle")
    
    return distances
```

- Time complexity: O(V × E)
- Can detect negative cycles
- Applications: Currency exchange, network routing with negative weights

## Minimum Spanning Tree (MST) Algorithms 🌳

A minimum spanning tree is a subset of edges that connects all vertices with the minimum total edge weight.

### Kruskal's Algorithm
Builds the MST by adding edges in order of increasing weight, avoiding cycles.

```python
def kruskal(graph):
    # Create a list of all edges
    edges = []
    for u in graph:
        for v, weight in graph[u].items():
            if u < v:  # Avoid duplicates in undirected graph
                edges.append((weight, u, v))
    
    # Sort edges by weight
    edges.sort()
    
    # Initialize disjoint set
    parent = {vertex: vertex for vertex in graph}
    
    def find(vertex):
        if parent[vertex] != vertex:
            parent[vertex] = find(parent[vertex])
        return parent[vertex]
    
    def union(u, v):
        parent[find(u)] = find(v)
    
    # Build MST
    mst = []
    for weight, u, v in edges:
        if find(u) != find(v):
            union(u, v)
            mst.append((u, v, weight))
    
    return mst
```

- Uses Union-Find data structure
- Time complexity: O(E log E) or O(E log V)
- Applications: Network design, clustering

### Prim's Algorithm
Builds the MST by growing a single tree from a starting vertex.

```python
import heapq

def prim(graph, start):
    # Initialize visited set and MST
    visited = {start}
    mst = []
    
    # Initialize priority queue with edges from start
    edges = [(weight, start, neighbor) for neighbor, weight in graph[start].items()]
    heapq.heapify(edges)
    
    while edges and len(visited) < len(graph):
        weight, u, v = heapq.heappop(edges)
        
        if v not in visited:
            visited.add(v)
            mst.append((u, v, weight))
            
            # Add new edges to priority queue
            for neighbor, w in graph[v].items():
                if neighbor not in visited:
                    heapq.heappush(edges, (w, v, neighbor))
    
    return mst
```

- Uses a priority queue
- Time complexity: O(E log V) with binary heap
- Applications: Network design, approximation algorithms

## Union-Find (Disjoint Set) 🔗

A data structure that keeps track of elements partitioned into disjoint (non-overlapping) sets.

```python
class UnionFind:
    def __init__(self, vertices):
        self.parent = {vertex: vertex for vertex in vertices}
        self.rank = {vertex: 0 for vertex in vertices}
    
    def find(self, vertex):
        if self.parent[vertex] != vertex:
            self.parent[vertex] = self.find(self.parent[vertex])  # Path compression
        return self.parent[vertex]
    
    def union(self, u, v):
        root_u = self.find(u)
        root_v = self.find(v)
        
        if root_u == root_v:
            return
        
        # Union by rank
        if self.rank[root_u] < self.rank[root_v]:
            self.parent[root_u] = root_v
        elif self.rank[root_u] > self.rank[root_v]:
            self.parent[root_v] = root_u
        else:
            self.parent[root_v] = root_u
            self.rank[root_u] += 1
```

- Operations:
  - find(x): Find the representative of the set containing x
  - union(x, y): Merge the sets containing x and y
- Time complexity: Nearly constant time per operation with path compression and union by rank
- Applications: Kruskal's algorithm, cycle detection, connected components

## Common Graph Problems 🧩

### Cycle Detection
```python
def has_cycle(graph):
    visited = set()
    rec_stack = set()
    
    def dfs(vertex):
        visited.add(vertex)
        rec_stack.add(vertex)
        
        for neighbor in graph[vertex]:
            if neighbor not in visited:
                if dfs(neighbor):
                    return True
            elif neighbor in rec_stack:
                return True
        
        rec_stack.remove(vertex)
        return False
    
    for vertex in graph:
        if vertex not in visited:
            if dfs(vertex):
                return True
    
    return False
```

### Connected Components
```python
def connected_components(graph):
    visited = set()
    components = []
    
    def dfs(vertex, component):
        visited.add(vertex)
        component.append(vertex)
        
        for neighbor in graph[vertex]:
            if neighbor not in visited:
                dfs(neighbor, component)
    
    for vertex in graph:
        if vertex not in visited:
            component = []
            dfs(vertex, component)
            components.append(component)
    
    return components
```

## Advantages and Disadvantages ⚖️

### Advantages ✅
- Model complex relationships and networks
- Solve real-world problems like routing, scheduling, and optimization
- Versatile data structure with many algorithms
- Can represent both physical and abstract relationships

### Disadvantages ❌
- Can be memory-intensive for large graphs
- Some algorithms have high time complexity
- Implementation can be complex
- Visualization becomes challenging for large graphs

## Real-world Applications 🌐
- Social networks (friend connections)
- Web page linking (PageRank)
- GPS and navigation systems
- Network routing
- Recommendation systems
- Dependency resolution
- Game development (pathfinding)
- Compiler optimization

## Further Reading 📚

### Books
- "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein
- "Algorithms" by Robert Sedgewick and Kevin Wayne
- "Graph Algorithms" by Shimon Even

### Online Resources
- [Visualgo - Graph Algorithms Visualization](https://visualgo.net/en/graphds)
- [GeeksforGeeks - Graph Algorithms](https://www.geeksforgeeks.org/graph-data-structure-and-algorithms/)
- [Khan Academy - Graph Representation](https://www.khanacademy.org/computing/computer-science/algorithms/graph-representation/a/representing-graphs)

### Courses
- Stanford's Algorithms Specialization on Coursera
- Princeton's Algorithms course on Coursera
- MIT OpenCourseWare - Introduction to Algorithms

---