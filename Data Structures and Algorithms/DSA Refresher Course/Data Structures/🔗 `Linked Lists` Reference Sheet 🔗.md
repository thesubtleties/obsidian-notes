## Definition 📝
A linked list is a linear data structure where elements (nodes) are stored in non-contiguous memory locations. Each node contains data and a reference (link) to the next node in the sequence. Unlike arrays, linked lists don't require contiguous memory allocation, allowing for dynamic size changes.

A typical node in a linked list contains:
- Data (the value stored)
- Pointer(s) to other node(s)

## Types of Linked Lists 📋

### 1. Singly Linked List
- Each node points to the next node
- The last node points to NULL
- Only forward traversal is possible

```
[Node 1] → [Node 2] → [Node 3] → NULL
  data       data       data
  next →     next →     next → NULL
```

### 2. Doubly Linked List
- Each node has two pointers: one to the next node and one to the previous node
- Allows traversal in both directions
- Requires more memory than singly linked lists

```
NULL ← [Node 1] ⇄ [Node 2] ⇄ [Node 3] → NULL
        data      data      data
      ← prev      prev ←    prev ←
        next →    next →    next → NULL
```

### 3. Circular Linked List
- Similar to singly linked list, but the last node points back to the first node
- No NULL references
- Can be singly or doubly circular

```
    ┌────────────────────┐
    ↓                    │
[Node 1] → [Node 2] → [Node 3]
  data      data      data
  next →    next →    next ↑
```

## Complexity Analysis ⏱️

| Operation        | Singly Linked | Doubly Linked | Array (for comparison) |
|------------------|---------------|---------------|------------------------|
| Access           | O(n)          | O(n)          | O(1)                   |
| Insert at start  | O(1)          | O(1)          | O(n)                   |
| Insert at end    | O(n)*         | O(1)          | O(1) amortized         |
| Insert in middle | O(n)          | O(n)          | O(n)                   |
| Delete at start  | O(1)          | O(1)          | O(n)                   |
| Delete at end    | O(n)*         | O(1)          | O(1)                   |
| Delete in middle | O(n)          | O(n)          | O(n)                   |
| Search           | O(n)          | O(n)          | O(n)                   |

*O(1) if tail pointer is maintained

## Core Operations 🔍

### Basic Operations
```
- insertAtBeginning(data): Insert a new node at the beginning
- insertAtEnd(data): Insert a new node at the end
- insertAfter(node, data): Insert a new node after a given node
- delete(key): Delete the first occurrence of a node with given key
- search(key): Search for a node with given key
- display(): Print all elements in the linked list
```

### Advanced Operations
```
- reverse(): Reverse the linked list
- detectCycle(): Check if the linked list has a cycle
- findMiddle(): Find the middle node of the linked list
- getNthNode(n): Get the nth node from the beginning
- removeDuplicates(): Remove duplicate nodes
- merge(list1, list2): Merge two sorted linked lists
```

## Implementation 💻

### Singly Linked List Implementation

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class SinglyLinkedList:
    def __init__(self):
        self.head = None
    
    def is_empty(self):
        return self.head is None
    
    def insert_at_beginning(self, data):
        new_node = Node(data)
        new_node.next = self.head
        self.head = new_node
    
    def insert_at_end(self, data):
        new_node = Node(data)
        if self.is_empty():
            self.head = new_node
            return
        
        current = self.head
        while current.next:
            current = current.next
        current.next = new_node
    
    def delete_node(self, key):
        # If linked list is empty
        if self.is_empty():
            return
        
        # If head node itself holds the key to be deleted
        if self.head.data == key:
            self.head = self.head.next
            return
        
        # Search for the key to be deleted, keep track of the
        # previous node as we need to change 'prev.next'
        current = self.head
        while current.next and current.next.data != key:
            current = current.next
        
        # If key was not present in linked list
        if not current.next:
            return
        
        # Unlink the node from linked list
        current.next = current.next.next
    
    def search(self, key):
        current = self.head
        while current:
            if current.data == key:
                return True
            current = current.next
        return False
    
    def display(self):
        elements = []
        current = self.head
        while current:
            elements.append(current.data)
            current = current.next
        return elements
```

### Doubly Linked List Implementation

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None
        self.prev = None

class DoublyLinkedList:
    def __init__(self):
        self.head = None
    
    def insert_at_beginning(self, data):
        new_node = Node(data)
        if self.head is None:
            self.head = new_node
            return
        
        new_node.next = self.head
        self.head.prev = new_node
        self.head = new_node
    
    def insert_at_end(self, data):
        new_node = Node(data)
        if self.head is None:
            self.head = new_node
            return
        
        current = self.head
        while current.next:
            current = current.next
        
        current.next = new_node
        new_node.prev = current
```

## Advanced Operations Implementation 🚀

### Reversing a Linked List

```python
def reverse(self):
    prev = None
    current = self.head
    
    while current:
        next_temp = current.next  # Store next
        current.next = prev       # Reverse the link
        prev = current            # Move prev forward
        current = next_temp       # Move current forward
    
    self.head = prev  # Update head to the new front
```

### Detecting a Cycle (Floyd's Cycle-Finding Algorithm)

```python
def has_cycle(self):
    if not self.head or not self.head.next:
        return False
    
    slow = self.head
    fast = self.head
    
    while fast and fast.next:
        slow = slow.next          # Move one step
        fast = fast.next.next     # Move two steps
        
        if slow == fast:  # If they meet, there's a cycle
            return True
    
    return False  # If fast reaches the end, no cycle
```

### Finding the Middle Node

```python
def find_middle(self):
    if not self.head:
        return None
    
    slow = self.head
    fast = self.head
    
    while fast and fast.next:
        slow = slow.next          # Move one step
        fast = fast.next.next     # Move two steps
    
    return slow  # When fast reaches the end, slow is at the middle
```

### Merging Two Sorted Linked Lists

```python
def merge_sorted_lists(list1, list2):
    dummy = Node(0)
    tail = dummy
    
    while list1 and list2:
        if list1.data <= list2.data:
            tail.next = list1
            list1 = list1.next
        else:
            tail.next = list2
            list2 = list2.next
        tail = tail.next
    
    # Attach the remaining nodes
    if list1:
        tail.next = list1
    if list2:
        tail.next = list2
    
    return dummy.next  # Return the merged list (excluding the dummy head)
```

## Common Examples 🚀

### Example 1: Implementing a Stack using a Linked List
```python
class LinkedListStack:
    def __init__(self):
        self.linked_list = SinglyLinkedList()
    
    def push(self, data):
        self.linked_list.insert_at_beginning(data)
    
    def pop(self):
        if self.linked_list.is_empty():
            return None
        data = self.linked_list.head.data
        self.linked_list.delete_node(data)
        return data
    
    def peek(self):
        if self.linked_list.is_empty():
            return None
        return self.linked_list.head.data
    
    def is_empty(self):
        return self.linked_list.is_empty()
```

### Example 2: Removing Duplicates from an Unsorted Linked List
```python
def remove_duplicates(self):
    if not self.head:
        return
    
    seen = set()
    current = self.head
    prev = None
    
    while current:
        if current.data in seen:
            # Remove the duplicate
            prev.next = current.next
        else:
            # Add to set and move prev
            seen.add(current.data)
            prev = current
        current = current.next
```

## Advantages and Disadvantages ⚖️

### Advantages ✅
- Dynamic size (grows and shrinks as needed)
- Efficient insertions and deletions at any position
- No need for contiguous memory allocation
- Efficient implementation of abstract data types like stacks and queues
- Easy implementation of more complex data structures

### Disadvantages ❌
- Random access is not allowed (must traverse from beginning)
- Extra memory needed for storing pointers
- Not cache-friendly due to non-contiguous memory allocation
- Reverse traversal is difficult in singly linked lists
- Higher memory usage compared to arrays

## Real-world Applications 🌐
- Implementation of stacks, queues, and hash tables
- Symbol tables in compiler design
- Undo functionality in applications
- Music playlists (next/previous song)
- Browser history navigation
- Image viewers (next/previous image)
- Task scheduling in operating systems

## Common Interview Questions 🎯
1. How do you find the middle element of a linked list in one pass?
2. How do you detect a cycle in a linked list?
3. How do you reverse a linked list?
4. How do you find the nth node from the end?
5. How do you check if a linked list is a palindrome?
6. How do you merge two sorted linked lists?
7. How do you remove duplicates from an unsorted linked list?

## Further Reading 📚
- Books:
  - "Cracking the Coding Interview" by Gayle Laakmann McDowell
  - "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein
  
- Online Resources:
  - [Visualgo - Linked List Visualization](https://visualgo.net/en/list)
  - [GeeksforGeeks - Linked List Data Structure](https://www.geeksforgeeks.org/data-structures/linked-list/)
  - [HackerRank - Linked Lists Practice Problems](https://www.hackerrank.com/domains/data-structures?filters%5Bsubdomains%5D%5B%5D=linked-lists)
  
- Interactive Learning:
  - [LeetCode - Linked List Problems](https://leetcode.com/tag/linked-list/)
  - [CodeSignal - Linked Lists Practice](https://app.codesignal.com/interview-practice/topics/linked-lists)

---