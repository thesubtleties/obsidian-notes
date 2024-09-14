
**Definition**: A graph is a data structure consisting of a set of nodes (vertices) and a set of edges connecting pairs of nodes.

**Types of Graphs**:
- **Undirected Graph**: Edges have no direction (e.g., friendships in a social network).
- **Directed Graph (Digraph)**: Edges have direction (e.g., Twitter follow relationships).
- **Weighted Graph**: Edges have weights (e.g., distances between cities).
- **Unweighted Graph**: Edges do not have weights.

### Adjacency List Representation

**Using an Object**

```javascript
const adjacencyListObject = {
    'NY': ['LA', 'CHI', 'MIA'],
    'LA': ['NY', 'SF', 'SEA'],
    'CHI': ['NY', 'DEN', 'DAL'],
    'MIA': ['NY', 'DAL', 'ATL'],
    'SF': ['LA', 'SEA'],
    'SEA': ['LA', 'SF'],
    'DEN': ['CHI', 'DAL'],
    'DAL': ['CHI', 'MIA', 'DEN'],
    'ATL': ['MIA']
};
```

**Using a Map**

```javascript
const adjacencyListMap = new Map();
adjacencyListMap.set('NY', ['LA', 'CHI', 'MIA']);
adjacencyListMap.set('LA', ['NY', 'SF', 'SEA']);
adjacencyListMap.set('CHI', ['NY', 'DEN', 'DAL']);
adjacencyListMap.set('MIA', ['NY', 'DAL', 'ATL']);
adjacencyListMap.set('SF', ['LA', 'SEA']);
adjacencyListMap.set('SEA', ['LA', 'SF']);
adjacencyListMap.set('DEN', ['CHI', 'DAL']);
adjacencyListMap.set('DAL', ['CHI', 'MIA', 'DEN']);
adjacencyListMap.set('ATL', ['MIA']);
```

### Adjacency Matrix Representation

#### Cities and Direct Flights:

- NY -> LA, CHI, MIA
- LA -> NY, SF, SEA
- CHI -> NY, DEN, DAL
- MIA -> NY, DAL, ATL
- SF -> LA, SEA
- SEA -> LA, SF
- DEN -> CHI, DAL
- DAL -> CHI, MIA, DEN
- ATL -> MIA

#### Adjacency Matrix (Unweighted Example)

|         | NY  | LA  | CHI | MIA | SF  | SEA | DEN | DAL | ATL |
| ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **NY**  | 0   | 1   | 1   | 1   | 0   | 0   | 0   | 0   | 0   |
| **LA**  | 1   | 0   | 0   | 0   | 1   | 1   | 0   | 0   | 0   |
| **CHI** | 1   | 0   | 0   | 0   | 0   | 0   | 1   | 1   | 0   |
| **MIA** | 1   | 0   | 0   | 0   | 0   | 0   | 0   | 1   | 1   |
| **SF**  | 0   | 1   | 0   | 0   | 0   | 1   | 0   | 0   | 0   |
| **SEA** | 0   | 1   | 0   | 0   | 1   | 0   | 0   | 0   | 0   |
| **DEN** | 0   | 0   | 1   | 0   | 0   | 0   | 0   | 1   | 0   |
| **DAL** | 0   | 0   | 1   | 1   | 0   | 0   | 1   | 0   | 0   |
| **ATL** | 0   | 0   | 0   | 1   | 0   | 0   | 0   | 0   | 0   |

#### Adjacency Matrix (Weighted Example with Distances)

|     | NY  | LA  | CHI | MIA | SF  | SEA | DEN | DAL | ATL |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|
| **NY**  | 0   | 2451| 790 | 1275| Inf | Inf | Inf | Inf | Inf |
| **LA**  | 2451| 0   | Inf | Inf | 381 | 1135| Inf | Inf | Inf |
| **CHI** | 790 | Inf | 0   | Inf | Inf | Inf | 1004| 802 | Inf |
| **MIA** | 1275| Inf | Inf | 0   | Inf | Inf | Inf | 1121| 664 |
| **SF**  | Inf | 381 | Inf | Inf | 0   | 807 | Inf | Inf | Inf |
| **SEA** | Inf | 1135| Inf | Inf | 807 | 0   | Inf | Inf | Inf |
| **DEN** | Inf | Inf | 1004| Inf | Inf | Inf | 0   | 661 | Inf |
| **DAL** | Inf | Inf | 802 | 1121| Inf | Inf | 661 | 0   | Inf |
| **ATL** | Inf | Inf | Inf | 664 | Inf | Inf | Inf | Inf | 0   |

### Key Differences to Remember
- **Graphs**:
  - Can have cycles.
  - Might not be connected.
  - Multiple paths or no paths between nodes.

- **Adjacency List vs. Adjacency Matrix**:
  - **Adjacency List**: Efficient for sparse graphs, easy to iterate over neighbors.
  - **Adjacency Matrix**: Efficient for dense graphs, constant time complexity for edge checks.

### Graph Traversal: Breadth-First Search (BFS) Example

#### Using Adjacency List (Object)

```javascript
const bfsObject = (graph, startNode) => {
    const visited = new Set();
    const queue = [startNode];

    while (queue.length > 0) {
        const node = queue.shift();

        if (!visited.has(node)) {
            visited.add(node);
            console.log(node); // Process the node

            const neighbors = graph[node];
            for (const neighbor of neighbors) {
                if (!visited.has(neighbor)) {
                    queue.push(neighbor);
                }
            }
        }
    }
};

const graphObject = {
    'NY': ['LA', 'CHI', 'MIA'],
    'LA': ['NY', 'SF', 'SEA'],
    'CHI': ['NY', 'DEN', 'DAL'],
    'MIA': ['NY', 'DAL', 'ATL'],
    'SF': ['LA', 'SEA'],
    'SEA': ['LA', 'SF'],
    'DEN': ['CHI', 'DAL'],
    'DAL': ['CHI', 'MIA', 'DEN'],
    'ATL': ['MIA']
};

bfsObject(graphObject, 'NY');
// Output: NY, LA, CHI, MIA, SF, SEA, DEN, DAL, ATL
```

#### Using Adjacency Matrix

```javascript
const bfsMatrix = (matrix, startNode, nodes) => {
    const visited = new Array(matrix.length).fill(false);
    const queue = [startNode];

    while (queue.length > 0) {
        const node = queue.shift();

        if (!visited[node]) {
            visited[node] = true;
            console.log(nodes[node]); // Process the node

            for (let i = 0; i < matrix[node].length; i++) {
                if (matrix[node][i] !== Infinity && !visited[i]) {
                    queue.push(i);
                }
            }
        }
    }
};

const adjacencyMatrix = [
    [0, 2451, 790, 1275, Infinity, Infinity, Infinity, Infinity, Infinity],
    [2451, 0, Infinity, Infinity, 381, 1135, Infinity, Infinity, Infinity],
    [790, Infinity, 0, Infinity, Infinity, Infinity, 1004, 802, Infinity],
    [1275, Infinity, Infinity, 0, Infinity, Infinity, Infinity, 1121, 664],
    [Infinity, 381, Infinity, Infinity, 0, 807, Infinity, Infinity, Infinity],
    [Infinity, 1135, Infinity, Infinity, 807, 0, Infinity, Infinity, Infinity],
    [Infinity, Infinity, 1004, Infinity, Infinity, Infinity, 0, 661, Infinity],
    [

```javascript
    [Infinity, Infinity, 802, 1121, Infinity, Infinity, 661, 0, Infinity],
    [Infinity, Infinity, Infinity, 664, Infinity, Infinity, Infinity, Infinity, 0]
];

const nodes = ['NY', 'LA', 'CHI', 'MIA', 'SF', 'SEA', 'DEN', 'DAL', 'ATL'];

// Perform BFS starting from 'NY' (index 0)
bfsMatrix(adjacencyMatrix, 0, nodes);
// Output: NY, LA, CHI, MIA, SF, SEA, DEN, DAL, ATL
```

### Summary

- **Graphs**: Data structures with nodes and edges, can be directed or undirected, weighted or unweighted.
- **Adjacency List**: Efficient for sparse graphs, easy to iterate over neighbors, can be represented using an object or a Map.
- **Adjacency Matrix**: Efficient for dense graphs, constant time complexity for edge checks, can be unweighted (0/1) or weighted (with distances, costs, etc.).

#### Example of Adjacency List and Matrix

- **Adjacency List (Object)**:
  ```javascript
  const adjacencyListObject = {
      'NY': ['LA', 'CHI', 'MIA'],
      'LA': ['NY', 'SF', 'SEA'],
      'CHI': ['NY', 'DEN', 'DAL'],
      'MIA': ['NY', 'DAL', 'ATL'],
      'SF': ['LA', 'SEA'],
      'SEA': ['LA', 'SF'],
      'DEN': ['CHI', 'DAL'],
      'DAL': ['CHI', 'MIA', 'DEN'],
      'ATL': ['MIA']
  };
  ```

- **Adjacency Matrix (Weighted Example)**:
  ```javascript
  const adjacencyMatrix = [
      [0, 2451, 790, 1275, Infinity, Infinity, Infinity, Infinity, Infinity],
      [2451, 0, Infinity, Infinity, 381, 1135, Infinity, Infinity, Infinity],
      [790, Infinity, 0, Infinity, Infinity, Infinity, 1004, 802, Infinity],
      [1275, Infinity, Infinity, 0, Infinity, Infinity, Infinity, 1121, 664],
      [Infinity, 381, Infinity, Infinity, 0, 807, Infinity, Infinity, Infinity],
      [Infinity, 1135, Infinity, Infinity, 807, 0, Infinity, Infinity, Infinity],
      [Infinity, Infinity, 1004, Infinity, Infinity, Infinity, 0, 661, Infinity],
      [Infinity, Infinity, 802, 1121, Infinity, Infinity, 661, 0, Infinity],
      [Infinity, Infinity, Infinity, 664, Infinity, Infinity, Infinity, Infinity, 0]
  ];
  const nodes = ['NY', 'LA', 'CHI', 'MIA', 'SF', 'SEA', 'DEN', 'DAL', 'ATL'];
  ```

- **Traversal Example (BFS)**:
  ```javascript
  const bfsMatrix = (matrix, startNode, nodes) => {
      const visited = new Array(matrix.length).fill(false);
      const queue = [startNode];

      while (queue.length > 0) {
          const node = queue.shift();

          if (!visited[node]) {
              visited[node] = true;
              console.log(nodes[node]); // Process the node

              for (let i = 0; i < matrix[node].length; i++) {
                  if (matrix[node][i] !== Infinity && !visited[i]) {
                      queue.push(i);
                  }
              }
          }
      }
  };

  bfsMatrix(adjacencyMatrix, 0, nodes); // Output: NY, LA, CHI, MIA, SF, SEA, DEN, DAL, ATL
  ```



---

### Handling Disconnected Components

To handle disconnected components, you need to perform a BFS (or DFS) starting from each unvisited node. This ensures you cover all parts of the graph, even those not connected to the initial starting node.

#### Example Graph with Disconnected Components

```javascript
const graphWithDisconnections = {
    'NY': ['LA', 'CHI', 'MIA'],
    'LA': ['NY', 'SF', 'SEA'],
    'CHI': ['NY', 'DEN', 'DAL'],
    'MIA': ['NY', 'DAL', 'ATL'],
    'SF': ['LA', 'SEA'],
    'SEA': ['LA', 'SF'],
    'DEN': ['CHI', 'DAL'],
    'DAL': ['CHI', 'MIA', 'DEN'],
    'ATL': ['MIA'],
    'Nashville': ['Charleston', 'Raleigh'],
    'Charleston': ['Nashville'],
    'Raleigh': ['Nashville']
};
```

#### BFS for Disconnected Graphs

```javascript
const bfsComplete = (graph) => {
    const visited = new Set();

    const bfs = (startNode) => {
        const queue = [startNode];

        while (queue.length > 0) {
            const node = queue.shift();

            if (!visited.has(node)) {
                visited.add(node);
                console.log(node); // Process the node

                const neighbors = graph[node];
                for (const neighbor of neighbors) {
                    if (!visited.has(neighbor)) {
                        queue.push(neighbor);
                    }
                }
            }
        }
    };

    for (const node in graph) {
        if (!visited.has(node)) {
            bfs(node);
        }
    }
};

bfsComplete(graphWithDisconnections);
// Output: NY, LA, CHI, MIA, SF, SEA, DEN, DAL, ATL, Nashville, Charleston, Raleigh
```

### Summary

- **Graphs**: Data structures with nodes and edges, can be directed or undirected, weighted or unweighted.
- **Adjacency List**: Efficient for sparse graphs, easy to iterate over neighbors, can be represented using an object or a Map.
- **Adjacency Matrix**: Efficient for dense graphs, constant time complexity for edge checks, can be unweighted (0/1) or weighted (with distances, costs, etc.).
- **Traversal**: Use BFS or DFS to traverse the graph. Ensure all nodes are visited by handling disconnected components.

This cheat sheet should provide a comprehensive guide to understanding and working with graphs, including handling disconnected components. Let me know if you have any more questions or need further examples!