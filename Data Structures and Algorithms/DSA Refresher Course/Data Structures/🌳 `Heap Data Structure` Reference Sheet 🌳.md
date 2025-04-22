## Definition 📝
A Heap is a specialized tree-based data structure that satisfies the heap property:
- In a **max heap**, for any given node, the value of the node is greater than or equal to the values of its children
- In a **min heap**, for any given node, the value of the node is less than or equal to the values of its children

Heaps are complete binary trees, meaning all levels are fully filled except possibly the last level, which is filled from left to right.

## Complexity Analysis ⏱️
| Operation       | Time Complexity | Space Complexity |
|-----------------|-----------------|------------------|
| Build Heap      | O(n)            | O(n)             |
| Insert          | O(log n)        | O(1)             |
| Extract Min/Max | O(log n)        | O(1)             |
| Get Min/Max     | O(1)            | O(1)             |
| Heapify         | O(log n)        | O(1)             |
| Heap Sort       | O(n log n)      | O(1)             |

## Core Operations 🔍
- **insert(value)**: Add a new element to the heap
- **extractMin/extractMax()**: Remove and return the minimum/maximum element
- **getMin/getMax()**: Return (without removing) the minimum/maximum element
- **heapify()**: Restore the heap property after modification
- **buildHeap()**: Create a heap from an unordered array

## Types of Heaps 🔄

### Min Heap
- Root node contains the minimum value in the heap
- For any node, its value is less than or equal to its children
- Useful for finding minimum elements quickly

### Max Heap
- Root node contains the maximum value in the heap
- For any node, its value is greater than or equal to its children
- Useful for finding maximum elements quickly

## Implementation 💻

### Array Representation
Heaps are typically implemented using arrays, where for a node at index `i`:
- Left child: `2*i + 1`
- Right child: `2*i + 2`
- Parent: `(i-1)/2` (integer division)

### Min Heap Implementation in Python
```python
class MinHeap:
    def __init__(self):
        self.heap = []
        
    def parent(self, i):
        return (i - 1) // 2
        
    def left_child(self, i):
        return 2 * i + 1
        
    def right_child(self, i):
        return 2 * i + 2
        
    def get_min(self):
        if len(self.heap) == 0:
            return None
        return self.heap[0]
        
    def insert(self, value):
        self.heap.append(value)
        current = len(self.heap) - 1
        
        # Heapify up (bubble up)
        while current > 0 and self.heap[current] < self.heap[self.parent(current)]:
            # Swap with parent
            self.heap[current], self.heap[self.parent(current)] = self.heap[self.parent(current)], self.heap[current]
            current = self.parent(current)
            
    def extract_min(self):
        if len(self.heap) == 0:
            return None
            
        min_val = self.heap[0]
        
        # Replace root with last element
        self.heap[0] = self.heap[-1]
        self.heap.pop()
        
        # Heapify down (sift down)
        self._heapify_down(0)
        
        return min_val
        
    def _heapify_down(self, i):
        smallest = i
        left = self.left_child(i)
        right = self.right_child(i)
        heap_size = len(self.heap)
        
        # Check if left child exists and is smaller than current smallest
        if left < heap_size and self.heap[left] < self.heap[smallest]:
            smallest = left
            
        # Check if right child exists and is smaller than current smallest
        if right < heap_size and self.heap[right] < self.heap[smallest]:
            smallest = right
            
        # If smallest is not the current node, swap and continue heapifying
        if smallest != i:
            self.heap[i], self.heap[smallest] = self.heap[smallest], self.heap[i]
            self._heapify_down(smallest)
    
    def build_heap(self, arr):
        self.heap = arr.copy()
        # Start from the last non-leaf node and heapify down
        for i in range(len(self.heap) // 2 - 1, -1, -1):
            self._heapify_down(i)
```

### Max Heap Implementation in Python
```python
class MaxHeap:
    def __init__(self):
        self.heap = []
        
    def parent(self, i):
        return (i - 1) // 2
        
    def left_child(self, i):
        return 2 * i + 1
        
    def right_child(self, i):
        return 2 * i + 2
        
    def get_max(self):
        if len(self.heap) == 0:
            return None
        return self.heap[0]
        
    def insert(self, value):
        self.heap.append(value)
        current = len(self.heap) - 1
        
        # Heapify up
        while current > 0 and self.heap[current] > self.heap[self.parent(current)]:
            # Swap with parent
            self.heap[current], self.heap[self.parent(current)] = self.heap[self.parent(current)], self.heap[current]
            current = self.parent(current)
            
    def extract_max(self):
        if len(self.heap) == 0:
            return None
            
        max_val = self.heap[0]
        
        # Replace root with last element
        self.heap[0] = self.heap[-1]
        self.heap.pop()
        
        # Heapify down
        self._heapify_down(0)
        
        return max_val
        
    def _heapify_down(self, i):
        largest = i
        left = self.left_child(i)
        right = self.right_child(i)
        heap_size = len(self.heap)
        
        # Check if left child exists and is larger than current largest
        if left < heap_size and self.heap[left] > self.heap[largest]:
            largest = left
            
        # Check if right child exists and is larger than current largest
        if right < heap_size and self.heap[right] > self.heap[largest]:
            largest = right
            
        # If largest is not the current node, swap and continue heapifying
        if largest != i:
            self.heap[i], self.heap[largest] = self.heap[largest], self.heap[i]
            self._heapify_down(largest)
    
    def build_heap(self, arr):
        self.heap = arr.copy()
        # Start from the last non-leaf node and heapify down
        for i in range(len(self.heap) // 2 - 1, -1, -1):
            self._heapify_down(i)
```

## Heap Sort 📊
Heap Sort is a comparison-based sorting algorithm that uses a binary heap data structure:

```python
def heap_sort(arr):
    n = len(arr)
    
    # Build a max heap
    for i in range(n // 2 - 1, -1, -1):
        heapify(arr, n, i)
    
    # Extract elements one by one
    for i in range(n - 1, 0, -1):
        arr[i], arr[0] = arr[0], arr[i]  # Swap
        heapify(arr, i, 0)
    
    return arr

def heapify(arr, n, i):
    largest = i
    left = 2 * i + 1
    right = 2 * i + 2
    
    if left < n and arr[left] > arr[largest]:
        largest = left
    
    if right < n and arr[right] > arr[largest]:
        largest = right
    
    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)
```

## Custom Comparators 🔧
For heaps with complex objects, you can define custom comparison logic:

```python
import heapq

# Using a tuple with priority as the first element
priority_queue = []
heapq.heappush(priority_queue, (2, "Task 2"))
heapq.heappush(priority_queue, (1, "Task 1"))
heapq.heappush(priority_queue, (3, "Task 3"))

# Items will be popped in priority order
while priority_queue:
    priority, task = heapq.heappop(priority_queue)
    print(f"Processing {task} with priority {priority}")

# For custom objects, you can use __lt__ method
class Task:
    def __init__(self, priority, name):
        self.priority = priority
        self.name = name
    
    def __lt__(self, other):
        # Define how tasks should be compared
        return self.priority < other.priority
    
    def __repr__(self):
        return f"Task({self.priority}, {self.name})"

# Now we can use our Task objects in a heap
task_queue = []
heapq.heappush(task_queue, Task(2, "Medium priority"))
heapq.heappush(task_queue, Task(1, "High priority"))
heapq.heappush(task_queue, Task(3, "Low priority"))

# Tasks will be popped in priority order
while task_queue:
    task = heapq.heappop(task_queue)
    print(f"Processing {task.name}")
```

## Visualization 📊
Min Heap:
```
       1
     /   \
    3     2
   / \   /
  6   5 4
```

Max Heap:
```
       6
     /   \
    5     4
   / \   /
  3   1 2
```

Array representation of the min heap: [1, 3, 2, 6, 5, 4]

## Advantages and Disadvantages ⚖️
### Advantages ✅
- Efficient access to the minimum/maximum element (O(1))
- Efficient insertion and deletion operations (O(log n))
- Efficient implementation of priority queues
- In-place sorting with Heap Sort
- Simple array-based implementation

### Disadvantages ❌
- Not suitable for searching for arbitrary elements (O(n) time)
- Not as cache-friendly as arrays due to non-sequential memory access
- No efficient support for joining or merging heaps
- Not stable for sorting (equal elements may change order)

## Real-world Applications 🌐
- Priority Queues
- Scheduling algorithms in operating systems
- Dijkstra's algorithm for finding shortest paths
- Huffman coding for data compression
- Selection algorithms (finding kth smallest/largest element)
- Event-driven simulation
- Memory management in systems

## Further Reading 📚
- Books:
  - "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein
  - "Data Structures and Algorithms in Python" by Goodrich, Tamassia, and Goldwasser
- Online Resources:
  - [Visualgo Heap Visualization](https://visualgo.net/en/heap)
  - [GeeksforGeeks Heap Data Structure](https://www.geeksforgeeks.org/heap-data-structure/)
  - [HackerRank Heap Tutorial](https://www.hackerrank.com/topics/heaps)
- Python Documentation:
  - [heapq — Heap queue algorithm](https://docs.python.org/3/library/heapq.html)

---