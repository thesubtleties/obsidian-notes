## Definition 📝
Greedy algorithms make locally optimal choices at each step with the hope of finding a global optimum. They choose the best option available at the moment without reconsidering previous choices. Unlike dynamic programming, greedy algorithms don't evaluate all possible solutions but build the solution incrementally.

Key characteristics:
- Make the locally optimal choice at each step
- Never reconsider previous choices
- Simple and often intuitive approach
- Work for problems with "greedy-choice property" and "optimal substructure"

## Complexity Analysis ⏱️
Greedy algorithms typically have efficient time complexity:
- Usually O(n log n) due to an initial sorting step
- Sometimes as efficient as O(n) for simpler problems
- Space complexity is typically O(n) or O(1)

The efficiency makes greedy algorithms attractive, but they don't guarantee optimal solutions for all problems.

## Core Operations 🔍
1. **Sort or organize data** - Arrange elements based on a greedy criterion
2. **Iterate through elements** - Process each element once
3. **Make locally optimal choice** - Select best option at current step
4. **Update solution** - Add the choice to the solution being built
5. **Repeat** - Continue until a complete solution is formed

## When to Use Greedy Algorithms 🎯
Greedy algorithms work well when:
- The problem has optimal substructure (optimal solution contains optimal solutions to subproblems)
- The problem has the greedy-choice property (locally optimal choices lead to globally optimal solution)
- The problem requires maximizing or minimizing some value

## Common Greedy Algorithms 🚀

### 1. Activity Selection Problem
**Goal**: Select maximum number of non-overlapping activities

```python
def activity_selection(start, finish):
    # Sort activities by finish time
    activities = sorted(zip(start, finish), key=lambda x: x[1])
    
    selected = [activities[0]]  # Select first activity
    last_finish_time = activities[0][1]
    
    # Consider rest of the activities
    for i in range(1, len(activities)):
        # If this activity starts after the last selected activity finishes
        if activities[i][0] >= last_finish_time:
            selected.append(activities[i])
            last_finish_time = activities[i][1]
            
    return selected

# Example
start_times = [1, 3, 0, 5, 8, 5]
finish_times = [2, 4, 6, 7, 9, 9]
print(activity_selection(start_times, finish_times))
# Output: [(1, 2), (3, 4), (5, 7), (8, 9)]
```

**Time Complexity**: O(n log n) due to sorting
**Space Complexity**: O(n)

### 2. Huffman Coding
**Goal**: Create optimal prefix code for data compression

```python
import heapq

class Node:
    def __init__(self, freq, symbol, left=None, right=None):
        self.freq = freq
        self.symbol = symbol
        self.left = left
        self.right = right
        self.huff = ''
        
    def __lt__(self, nxt):
        return self.freq < nxt.freq
        
def print_nodes(node, val=''):
    # Huffman code for current node
    new_val = val + str(node.huff)
    
    # If not leaf node, traverse inside it
    if node.left:
        print_nodes(node.left, new_val)
    if node.right:
        print_nodes(node.right, new_val)
        
    # If leaf node, print the code
    if not node.left and not node.right:
        print(f"{node.symbol} -> {new_val}")

def huffman_coding(data):
    # Calculate frequency of each character
    freq = {}
    for char in data:
        if char in freq:
            freq[char] += 1
        else:
            freq[char] = 1
            
    # Create priority queue (min heap)
    heap = [Node(freq[symbol], symbol) for symbol in freq]
    heapq.heapify(heap)
    
    # Build Huffman Tree
    while len(heap) > 1:
        # Extract two nodes with lowest frequency
        left = heapq.heappop(heap)
        right = heapq.heappop(heap)
        
        # Assign directional value (0/1)
        left.huff = 0
        right.huff = 1
        
        # Create new internal node with these two nodes as children
        # and frequency equal to sum of their frequencies
        new_node = Node(left.freq + right.freq, left.symbol + right.symbol, left, right)
        
        heapq.heappush(heap, new_node)
        
    # Print Huffman codes
    print_nodes(heap[0])

# Example
data = "BCAADDDCCACACAC"
huffman_coding(data)
# Output:
# B -> 000
# A -> 01
# D -> 10
# C -> 11
```

**Time Complexity**: O(n log n) where n is the number of unique characters
**Space Complexity**: O(n)

### 3. Interval Scheduling
**Goal**: Schedule maximum number of intervals without overlap

```python
def interval_scheduling(intervals):
    # Sort intervals by end time
    intervals.sort(key=lambda x: x[1])
    
    selected = []
    if not intervals:
        return selected
        
    # Select first interval
    selected.append(intervals[0])
    last_end_time = intervals[0][1]
    
    # Consider rest of intervals
    for i in range(1, len(intervals)):
        # If current interval starts after the last selected interval ends
        if intervals[i][0] >= last_end_time:
            selected.append(intervals[i])
            last_end_time = intervals[i][1]
            
    return selected

# Example
intervals = [(1, 3), (2, 5), (3, 9), (6, 8)]
print(interval_scheduling(intervals))
# Output: [(1, 3), (6, 8)]
```

**Time Complexity**: O(n log n) due to sorting
**Space Complexity**: O(n)

### 4. Fractional Knapsack
**Goal**: Maximize value in knapsack with fractional items allowed

```python
def fractional_knapsack(items, capacity):
    # Calculate value/weight ratio for each item
    for item in items:
        item.append(item[0] / item[1])  # [value, weight, ratio]
    
    # Sort items by value/weight ratio in descending order
    items.sort(key=lambda x: x[2], reverse=True)
    
    total_value = 0
    remaining_capacity = capacity
    
    for item in items:
        if remaining_capacity <= 0:
            break
            
        # If we can take the whole item
        if item[1] <= remaining_capacity:
            total_value += item[0]
            remaining_capacity -= item[1]
        else:
            # Take a fraction of the item
            fraction = remaining_capacity / item[1]
            total_value += item[0] * fraction
            remaining_capacity = 0
            
    return total_value

# Example
# Format: [value, weight]
items = [[60, 10], [100, 20], [120, 30]]
capacity = 50
print(fractional_knapsack(items, capacity))
# Output: 240.0
```

**Time Complexity**: O(n log n) due to sorting
**Space Complexity**: O(1) (excluding input)

## Advantages and Disadvantages ⚖️

### Advantages ✅
- Simple and intuitive implementation
- Efficient time complexity
- Works well for many optimization problems
- Often provides good approximations even when not optimal
- Requires minimal memory

### Disadvantages ❌
- Doesn't always yield the optimal solution
- Difficult to prove correctness
- May get stuck in local optima
- Not suitable for problems without greedy-choice property
- Can be challenging to identify the correct greedy criterion

## Real-world Applications 🌐
- Network routing protocols (Dijkstra's algorithm)
- Data compression (Huffman coding)
- Task scheduling in operating systems
- Coin change problems in financial systems
- Minimum spanning trees in network design (Kruskal's, Prim's)
- Resource allocation in cloud computing
- Job scheduling in manufacturing

## Common Pitfalls and Tips 💡
- Always verify that the problem has the greedy-choice property
- Test your algorithm on edge cases
- Consider sorting as a preprocessing step
- Compare with dynamic programming approach if unsure
- Prove correctness through mathematical induction when possible
- Remember that greedy algorithms build solutions incrementally

## Further Reading 📚
- Books:
  - "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (Chapter 16)
  - "Algorithm Design" by Kleinberg and Tardos
  - "The Algorithm Design Manual" by Steven Skiena

- Online Resources:
  - [GeeksforGeeks Greedy Algorithms](https://www.geeksforgeeks.org/greedy-algorithms/)
  - [Khan Academy Greedy Algorithms](https://www.khanacademy.org/computing/computer-science/algorithms/greedy-algorithms/a/overview-of-greedy-algorithms)
  - [Visualgo - Algorithm Visualization](https://visualgo.net/en)

- Courses:
  - Stanford's Algorithms Specialization on Coursera
  - MIT OpenCourseWare - Introduction to Algorithms

---