## Basic Queue Implementation
```python
from collections import deque

# Create a queue
queue = deque()
queue = deque(maxlen=5)  # Fixed-size queue

# Basic operations
queue.append(1)          # Add to right (enqueue)
queue.append(2)
queue.append(3)
first = queue.popleft()  # Remove from left (dequeue)

# Check queue status
length = len(queue)
is_empty = len(queue) == 0
```
Deque (double-ended queue) is the preferred way to implement queues in Python, offering O(1) operations at both ends.

## Queue Module
```python
from queue import Queue, LifoQueue, PriorityQueue

# Standard FIFO Queue
q = Queue(maxsize=0)  # Infinite size queue
q.put(1)             # Add item
q.put(2)
item = q.get()       # Remove item
q.task_done()        # Mark task as done

# Queue status
size = q.qsize()
empty = q.empty()
full = q.full()
```
The queue module provides thread-safe queue implementations, ideal for multi-threaded applications.

## Priority Queue
```python
from queue import PriorityQueue

# Create priority queue
pq = PriorityQueue()

# Add items (priority, data)
pq.put((2, "Medium priority"))
pq.put((1, "High priority"))
pq.put((3, "Low priority"))

# Get items (automatically sorted by priority)
while not pq.empty():
    priority, item = pq.get()
    print(f"Priority {priority}: {item}")
```
Priority queues automatically sort items based on their priority value.

## Stack Implementation
```python
# Using list as a stack
stack = []
stack.append(1)      # Push
stack.append(2)
stack.append(3)
item = stack.pop()   # Pop

# Using deque as a stack
from collections import deque
stack = deque()
stack.append(1)      # Push
stack.append(2)
item = stack.pop()   # Pop
```
While lists can be used as stacks, deque provides better performance for large datasets.

## LifoQueue (Thread-safe Stack)
```python
from queue import LifoQueue

# Create a LIFO queue (stack)
stack = LifoQueue()

# Basic operations
stack.put(1)         # Push
stack.put(2)
item = stack.get()   # Pop

# Check status
size = stack.qsize()
is_empty = stack.empty()
```
LifoQueue provides a thread-safe stack implementation.

## Deque as Both Stack and Queue
```python
from collections import deque

# Create a deque
d = deque()

# Queue operations
d.append(1)          # Add to right
d.append(2)
left = d.popleft()   # Remove from left

# Stack operations
d.append(3)          # Push
d.append(4)
right = d.pop()      # Pop from right

# Additional operations
d.appendleft(0)      # Add to left
d.extend([5, 6, 7])  # Add multiple to right
d.extendleft([8, 9]) # Add multiple to left
```
Deque is versatile and can be used as both a stack and a queue.

## Custom Queue Implementation
```python
class CircularQueue:
    def __init__(self, size):
        self.size = size
        self.queue = [None] * size
        self.front = self.rear = -1

    def enqueue(self, item):
        if (self.rear + 1) % self.size == self.front:
            raise Exception("Queue is full")
        elif self.front == -1:
            self.front = self.rear = 0
        else:
            self.rear = (self.rear + 1) % self.size
        self.queue[self.rear] = item

    def dequeue(self):
        if self.front == -1:
            raise Exception("Queue is empty")
        item = self.queue[self.front]
        if self.front == self.rear:
            self.front = self.rear = -1
        else:
            self.front = (self.front + 1) % self.size
        return item
```
Custom implementations can be useful for specific requirements like circular queues.

## Thread-Safe Operations
```python
from queue import Queue
from threading import Thread

def worker(q):
    while True:
        item = q.get()
        if item is None:
            break
        process_item(item)
        q.task_done()

# Create queue and workers
q = Queue()
threads = []
for i in range(3):
    t = Thread(target=worker, args=(q,))
    t.start()
    threads.append(t)

# Add items to queue
for item in items:
    q.put(item)

# Add None to stop workers
for _ in threads:
    q.put(None)

# Wait for completion
for t in threads:
    t.join()
```
Queue module provides thread-safe implementations for concurrent programming.

## Error Handling
```python
from queue import Queue, Full, Empty

q = Queue(maxsize=2)

# Handle queue full
try:
    q.put(1, block=False)
    q.put(2, block=False)
    q.put(3, block=False)  # This will raise Full
except Full:
    print("Queue is full")

# Handle queue empty
try:
    item = q.get(block=False)  # This will raise Empty if queue is empty
except Empty:
    print("Queue is empty")
```
Always handle potential queue overflow and underflow conditions.

## Best Practices
```python
# Choose the right implementation
def get_queue(queue_type="fifo", maxsize=0):
    if queue_type == "fifo":
        return Queue(maxsize=maxsize)
    elif queue_type == "lifo":
        return LifoQueue(maxsize=maxsize)
    elif queue_type == "priority":
        return PriorityQueue(maxsize=maxsize)
    else:
        return deque(maxlen=maxsize if maxsize > 0 else None)

# Use context managers for cleanup
def process_queue(q):
    try:
        while True:
            item = q.get_nowait()
            process_item(item)
            q.task_done()
    except Empty:
        pass
```
Choose the appropriate queue implementation based on your needs.

## Common Patterns
```python
# Producer-Consumer pattern
def producer(q):
    for i in range(5):
        q.put(i)
    q.put(None)  # Signal end

def consumer(q):
    while True:
        item = q.get()
        if item is None:
            break
        process_item(item)
        q.task_done()

# Breadth-First Search using Queue
def bfs(graph, start):
    visited = set()
    queue = deque([start])
    while queue:
        vertex = queue.popleft()
        if vertex not in visited:
            visited.add(vertex)
            queue.extend(graph[vertex] - visited)
    return visited
```
Queues and stacks are fundamental for many programming patterns and algorithms.

This covers the main aspects of working with queues and stacks in Python, including different implementations and common usage patterns.