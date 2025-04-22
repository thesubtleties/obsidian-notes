## Definition 📝
A Queue is a linear data structure that follows the First-In-First-Out (FIFO) principle. Elements are added at the rear (enqueue) and removed from the front (dequeue). Think of it like a line of people waiting - the first person to join the line is the first one to be served.

## Complexity Analysis ⏱️
| Operation | Time Complexity |
|-----------|-----------------|
| Enqueue   | O(1)            |
| Dequeue   | O(1)            |
| Peek      | O(1)            |
| IsEmpty   | O(1)            |
| Size      | O(1)            |

Space Complexity: O(n) where n is the number of elements in the queue.

## Core Operations 🔍
- **Enqueue**: Add an element to the rear of the queue
- **Dequeue**: Remove and return the element at the front of the queue
- **Peek/Front**: View the element at the front without removing it
- **IsEmpty**: Check if the queue is empty
- **Size**: Get the number of elements in the queue

## Implementation 💻

### Array-based Implementation
```python
class Queue:
    def __init__(self):
        self.items = []
    
    def enqueue(self, item):
        self.items.append(item)
    
    def dequeue(self):
        if not self.is_empty():
            return self.items.pop(0)  # O(n) operation in Python
        return None
    
    def peek(self):
        if not self.is_empty():
            return self.items[0]
        return None
    
    def is_empty(self):
        return len(self.items) == 0
    
    def size(self):
        return len(self.items)
```

### Linked List Implementation
```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class Queue:
    def __init__(self):
        self.front = None
        self.rear = None
        self.count = 0
    
    def enqueue(self, item):
        new_node = Node(item)
        
        if self.rear is None:
            self.front = new_node
            self.rear = new_node
        else:
            self.rear.next = new_node
            self.rear = new_node
        
        self.count += 1
    
    def dequeue(self):
        if self.is_empty():
            return None
        
        temp = self.front
        self.front = self.front.next
        
        if self.front is None:
            self.rear = None
            
        self.count -= 1
        return temp.data
    
    def peek(self):
        if not self.is_empty():
            return self.front.data
        return None
    
    def is_empty(self):
        return self.front is None
    
    def size(self):
        return self.count
```

## Types of Queues 🚀

### 1. Regular Queue
The standard queue implementation as shown above, following the FIFO principle.

### 2. Circular Queue
A circular queue is a linear data structure that uses a fixed-size array and wraps around to the beginning when it reaches the end, efficiently utilizing memory.

```python
class CircularQueue:
    def __init__(self, capacity):
        self.capacity = capacity
        self.items = [None] * capacity
        self.front = -1
        self.rear = -1
        self.size = 0
    
    def enqueue(self, item):
        if self.is_full():
            return False
        
        if self.is_empty():
            self.front = 0
            self.rear = 0
        else:
            self.rear = (self.rear + 1) % self.capacity
        
        self.items[self.rear] = item
        self.size += 1
        return True
    
    def dequeue(self):
        if self.is_empty():
            return None
        
        item = self.items[self.front]
        
        if self.front == self.rear:  # Last item
            self.front = -1
            self.rear = -1
        else:
            self.front = (self.front + 1) % self.capacity
        
        self.size -= 1
        return item
    
    def peek(self):
        if self.is_empty():
            return None
        return self.items[self.front]
    
    def is_empty(self):
        return self.front == -1
    
    def is_full(self):
        return (self.rear + 1) % self.capacity == self.front or (self.front == 0 and self.rear == self.capacity - 1)
```

### 3. Priority Queue
A priority queue is a type of queue where elements have associated priorities. Elements with higher priority are served before elements with lower priority.

```python
import heapq

class PriorityQueue:
    def __init__(self):
        self.elements = []
    
    def enqueue(self, item, priority):
        # Lower values = higher priority
        heapq.heappush(self.elements, (priority, item))
    
    def dequeue(self):
        if not self.is_empty():
            return heapq.heappop(self.elements)[1]
        return None
    
    def peek(self):
        if not self.is_empty():
            return self.elements[0][1]
        return None
    
    def is_empty(self):
        return len(self.elements) == 0
    
    def size(self):
        return len(self.elements)
```

### 4. Deque (Double-Ended Queue)
A deque allows insertion and removal of elements from both ends.

```python
class Deque:
    def __init__(self):
        self.items = []
    
    def add_front(self, item):
        self.items.insert(0, item)
    
    def add_rear(self, item):
        self.items.append(item)
    
    def remove_front(self):
        if not self.is_empty():
            return self.items.pop(0)
        return None
    
    def remove_rear(self):
        if not self.is_empty():
            return self.items.pop()
        return None
    
    def peek_front(self):
        if not self.is_empty():
            return self.items[0]
        return None
    
    def peek_rear(self):
        if not self.is_empty():
            return self.items[-1]
        return None
    
    def is_empty(self):
        return len(self.items) == 0
    
    def size(self):
        return len(self.items)
```

## Visualization 📊

### Regular Queue
```
Enqueue: Add to rear
[A, B, C, D] ← (new items)
 ↑
Front

Dequeue: Remove from front
 ↓
[B, C, D]
```

### Circular Queue
```
Array: [A, B, C, D, E, _, _, _]
        ↑              ↑
      Front          Rear

After dequeue:
Array: [_, B, C, D, E, _, _, _]
           ↑           ↑
         Front       Rear

After enqueue F, G:
Array: [_, B, C, D, E, F, G, _]
           ↑                 ↑
         Front             Rear
```

## Advantages and Disadvantages ⚖️

### Regular Queue
#### Advantages ✅
- Simple implementation
- Efficient for FIFO operations
- Useful for processing tasks in order of arrival

#### Disadvantages ❌
- Array implementation has O(n) dequeue operation
- Fixed size in array implementation
- Cannot access elements by position

### Circular Queue
#### Advantages ✅
- Better memory utilization
- All operations are O(1)
- Prevents memory waste through wrapping

#### Disadvantages ❌
- Slightly more complex implementation
- Fixed capacity
- Tracking front and rear pointers adds complexity

### Priority Queue
#### Advantages ✅
- Processes elements based on priority
- Useful for scheduling and resource allocation
- Can be implemented efficiently with heaps

#### Disadvantages ❌
- More complex implementation
- Enqueue and dequeue operations are O(log n)
- Not strictly FIFO

### Deque
#### Advantages ✅
- Flexible - can be used as both stack and queue
- Supports operations at both ends
- Versatile for various algorithms

#### Disadvantages ❌
- More complex implementation
- Slightly higher memory overhead
- Array implementation may require resizing

## Real-world Applications 🌐
- **Regular Queue**: Print spooling, handling requests on a single shared resource
- **Circular Queue**: CPU scheduling, memory management
- **Priority Queue**: Operating system task scheduling, Dijkstra's algorithm, A* search
- **Deque**: Implementing both stack and queue operations, palindrome checking, sliding window problems

## Further Reading 📚
- Books:
  - "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein
  - "Data Structures and Algorithms in Python" by Goodrich, Tamassia, and Goldwasser
  
- Online Resources:
  - [GeeksforGeeks Queue Data Structure](https://www.geeksforgeeks.org/queue-data-structure/)
  - [Visualgo Queue Visualization](https://visualgo.net/en/list)
  - [HackerRank Data Structures](https://www.hackerrank.com/domains/data-structures)
  
- Practice Problems:
  - LeetCode: #232 (Implement Queue using Stacks)
  - LeetCode: #622 (Design Circular Queue)
  - LeetCode: #641 (Design Circular Deque)

---