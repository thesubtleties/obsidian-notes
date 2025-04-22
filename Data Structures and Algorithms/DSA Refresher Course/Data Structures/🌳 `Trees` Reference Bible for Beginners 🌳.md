## Definition 📝
A tree is a hierarchical data structure consisting of nodes connected by edges. Each tree has a root node, and every node (except the root) is connected to exactly one parent node. A node can have zero or more child nodes.

Key characteristics:
- Non-linear structure (unlike arrays or linked lists)
- No cycles (no path leads back to the same node)
- Each node contains a value/data and references to its children
- Nodes with no children are called "leaf nodes"

## Tree Terminology 📚
- **Root**: The topmost node of the tree
- **Parent**: A node that has children
- **Child**: A node directly connected to another node when moving away from the root
- **Siblings**: Nodes that share the same parent
- **Leaf**: A node with no children
- **Edge**: The connection between nodes
- **Path**: Sequence of nodes and edges connecting two nodes
- **Depth**: Number of edges from the root to the node
- **Height**: Number of edges on the longest path from the node to a leaf
- **Level**: 1 + the number of edges between the node and the root
- **Subtree**: A tree consisting of a node and all its descendants

## Types of Trees 🌲

### Binary Tree
A tree where each node has at most two children, referred to as the left child and right child.

```
    A
   / \
  B   C
 / \   \
D   E   F
```

#### Implementation
```python
class TreeNode:
    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None
```

#### Types of Binary Trees
1. **Full Binary Tree**: Every node has 0 or 2 children
2. **Complete Binary Tree**: All levels are filled except possibly the last level, which is filled from left to right
3. **Perfect Binary Tree**: All internal nodes have two children and all leaf nodes are at the same level
4. **Balanced Binary Tree**: The height of the left and right subtrees of any node differ by at most 1

### Binary Search Tree (BST)
A binary tree with the property that for each node, all values in its left subtree are less than the node's value, and all values in its right subtree are greater than the node's value.

```
    50
   /  \
  30   70
 / \   / \
20 40 60  80
```

#### Complexity Analysis ⏱️
| Operation | Average Case | Worst Case |
|-----------|--------------|------------|
| Search    | O(log n)     | O(n)       |
| Insert    | O(log n)     | O(n)       |
| Delete    | O(log n)     | O(n)       |

#### Implementation
```python
class BSTNode:
    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None

class BST:
    def __init__(self):
        self.root = None
    
    def insert(self, value):
        if not self.root:
            self.root = BSTNode(value)
            return
        
        current = self.root
        while True:
            if value < current.value:
                if current.left is None:
                    current.left = BSTNode(value)
                    return
                current = current.left
            else:
                if current.right is None:
                    current.right = BSTNode(value)
                    return
                current = current.right
    
    def search(self, value):
        current = self.root
        while current:
            if current.value == value:
                return True
            elif value < current.value:
                current = current.left
            else:
                current = current.right
        return False
```

### AVL Tree
A self-balancing binary search tree where the difference between heights of left and right subtrees cannot be more than 1 for all nodes.

```
    50
   /  \
  30   70
 / \   / \
20 40 60  80
```

#### Balancing Operations
- **Left Rotation**: Used when right subtree becomes heavier
- **Right Rotation**: Used when left subtree becomes heavier
- **Left-Right Rotation**: Left rotation on left child, then right rotation on node
- **Right-Left Rotation**: Right rotation on right child, then left rotation on node

#### Complexity Analysis ⏱️
| Operation | Average Case | Worst Case |
|-----------|--------------|------------|
| Search    | O(log n)     | O(log n)   |
| Insert    | O(log n)     | O(log n)   |
| Delete    | O(log n)     | O(log n)   |

### Red-Black Tree
A self-balancing binary search tree where each node has an extra bit for denoting the color of the node, either red or black.

Properties:
1. Every node is either red or black
2. The root is black
3. All leaves (NIL) are black
4. If a node is red, both its children are black
5. Every path from a node to any of its descendant NIL nodes has the same number of black nodes

#### Complexity Analysis ⏱️
| Operation | Average Case | Worst Case |
|-----------|--------------|------------|
| Search    | O(log n)     | O(log n)   |
| Insert    | O(log n)     | O(log n)   |
| Delete    | O(log n)     | O(log n)   |

### N-ary Tree
A tree in which each node can have at most N children.

```
       A
    /  |  \
   B   C   D
  / \     / \
 E   F   G   H
```

#### Implementation
```python
class NaryTreeNode:
    def __init__(self, value):
        self.value = value
        self.children = []  # List to store references to child nodes
```

### Segment Tree
A tree data structure used for storing information about intervals or segments. Useful for range queries.

```
        [0-7]
       /     \
   [0-3]     [4-7]
   /   \     /   \
[0-1] [2-3] [4-5] [6-7]
```

#### Applications
- Range sum queries
- Range minimum/maximum queries
- Range update operations

## Tree Traversal Methods 🚶

### Depth-First Search (DFS)
1. **Pre-order**: Root → Left → Right
2. **In-order**: Left → Root → Right (gives sorted order in BST)
3. **Post-order**: Left → Right → Root

```python
# Pre-order traversal
def preorder(node):
    if node:
        print(node.value)  # Process the root
        preorder(node.left)  # Process left subtree
        preorder(node.right)  # Process right subtree

# In-order traversal
def inorder(node):
    if node:
        inorder(node.left)  # Process left subtree
        print(node.value)  # Process the root
        inorder(node.right)  # Process right subtree

# Post-order traversal
def postorder(node):
    if node:
        postorder(node.left)  # Process left subtree
        postorder(node.right)  # Process right subtree
        print(node.value)  # Process the root
```

### Breadth-First Search (BFS)
Also known as level-order traversal, visits nodes level by level from top to bottom.

```python
from collections import deque

def level_order(root):
    if not root:
        return
    
    queue = deque([root])
    while queue:
        node = queue.popleft()
        print(node.value)
        
        if node.left:
            queue.append(node.left)
        if node.right:
            queue.append(node.right)
```

## Common Tree Operations 🔍

### Finding Height of a Tree
```python
def height(node):
    if not node:
        return -1  # Height of empty tree is -1
    return 1 + max(height(node.left), height(node.right))
```

### Counting Nodes
```python
def count_nodes(node):
    if not node:
        return 0
    return 1 + count_nodes(node.left) + count_nodes(node.right)
```

### Checking if a Binary Tree is Balanced
```python
def is_balanced(node):
    if not node:
        return True
    
    left_height = height(node.left)
    right_height = height(node.right)
    
    if abs(left_height - right_height) <= 1 and is_balanced(node.left) and is_balanced(node.right):
        return True
    
    return False
```

## Advantages and Disadvantages ⚖️

### Advantages ✅
- Hierarchical representation of data
- Efficient search operations (especially in balanced trees)
- Dynamic size
- Can represent relationships between data points
- Fast insertions and deletions (in balanced trees)

### Disadvantages ❌
- More complex than linear data structures
- Unbalanced trees can degrade to O(n) performance
- Extra memory for pointers/references
- Balancing operations can be complex to implement

## Real-world Applications 🌐
- File systems (directories and files)
- Organization charts
- DOM (Document Object Model) in HTML
- Decision trees in machine learning
- Syntax trees in compilers
- Database indexing
- Network routing algorithms
- Game AI (minimax trees)

## Common Interview Questions 🎯
1. Implement a function to check if a binary tree is balanced
2. Check if a binary tree is a binary search tree
3. Find the lowest common ancestor of two nodes in a binary tree
4. Serialize and deserialize a binary tree
5. Convert a sorted array to a balanced binary search tree
6. Find the maximum path sum in a binary tree
7. Implement level-order traversal without using a queue

## Further Reading 📚
- Books:
  - "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein
  - "Data Structures and Algorithms in Python" by Goodrich, Tamassia, and Goldwasser
  
- Online Resources:
  - [Visualgo Tree Visualization](https://visualgo.net/en/bst)
  - [GeeksforGeeks Tree Data Structure](https://www.geeksforgeeks.org/binary-tree-data-structure/)
  - [HackerRank Tree Problems](https://www.hackerrank.com/domains/data-structures?filters%5Bsubdomains%5D%5B%5D=trees)
  
- Interactive Learning:
  - [LeetCode Tree Problems](https://leetcode.com/tag/tree/)
  - [InterviewBit Trees](https://www.interviewbit.com/courses/programming/topics/trees/)

---