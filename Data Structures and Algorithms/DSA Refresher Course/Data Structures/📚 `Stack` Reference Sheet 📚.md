## Definition 📝
A Stack is a linear data structure that follows the Last-In-First-Out (LIFO) principle. Think of it like a stack of plates - you can only add or remove items from the top. The last element added is the first one to be removed.

Key characteristics:
- Elements are added to the "top" of the stack
- Elements are removed from the "top" of the stack
- Only the top element is accessible at any time
- Follows LIFO (Last-In-First-Out) ordering

## Complexity Analysis ⏱️
| Operation | Time Complexity |
|-----------|-----------------|
| Push      | O(1)            |
| Pop       | O(1)            |
| Peek/Top  | O(1)            |
| isEmpty   | O(1)            |
| Size      | O(1)            |

Space Complexity: O(n) where n is the number of elements in the stack.

## Core Operations 🔍
- **push(element)**: Add an element to the top of the stack
- **pop()**: Remove and return the top element from the stack
- **peek()/top()**: Return the top element without removing it
- **isEmpty()**: Check if the stack is empty
- **size()**: Return the number of elements in the stack

## Implementation 💻

### Array-based Implementation
```python
class Stack:
    def __init__(self):
        self.items = []
    
    def push(self, item):
        self.items.append(item)
    
    def pop(self):
        if not self.isEmpty():
            return self.items.pop()
        return None
    
    def peek(self):
        if not self.isEmpty():
            return self.items[-1]
        return None
    
    def isEmpty(self):
        return len(self.items) == 0
    
    def size(self):
        return len(self.items)
```

### Linked List-based Implementation
```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class Stack:
    def __init__(self):
        self.top = None
        self._size = 0
    
    def push(self, item):
        new_node = Node(item)
        new_node.next = self.top
        self.top = new_node
        self._size += 1
    
    def pop(self):
        if self.isEmpty():
            return None
        popped = self.top.data
        self.top = self.top.next
        self._size -= 1
        return popped
    
    def peek(self):
        if self.isEmpty():
            return None
        return self.top.data
    
    def isEmpty(self):
        return self.top is None
    
    def size(self):
        return self._size
```

## Common Examples 🚀

### 1. Reversing a String
```python
def reverse_string(s):
    stack = []
    # Push all characters onto the stack
    for char in s:
        stack.append(char)
    
    # Pop all characters and concatenate
    reversed_str = ""
    while stack:
        reversed_str += stack.pop()
    
    return reversed_str

# Example: "hello" -> "olleh"
```

### 2. Parentheses Matching
```python
def is_balanced(expression):
    stack = []
    opening = "({["
    closing = ")}]"
    pairs = {')': '(', '}': '{', ']': '['}
    
    for char in expression:
        if char in opening:
            stack.append(char)
        elif char in closing:
            if not stack or stack.pop() != pairs[char]:
                return False
    
    return len(stack) == 0

# Example: is_balanced("({[]})")  # Returns True
# Example: is_balanced("({[})")   # Returns False
```

## Expression Evaluation 🧮

### Evaluating Postfix (RPN) Expressions
```python
def evaluate_postfix(expression):
    stack = []
    tokens = expression.split()
    
    for token in tokens:
        if token.isdigit() or (token[0] == '-' and token[1:].isdigit()):
            stack.append(int(token))
        else:  # Operator
            b = stack.pop()
            a = stack.pop()
            
            if token == '+':
                stack.append(a + b)
            elif token == '-':
                stack.append(a - b)
            elif token == '*':
                stack.append(a * b)
            elif token == '/':
                stack.append(a // b)
    
    return stack.pop()

# Example: evaluate_postfix("5 3 + 2 *")  # Returns 16
```

### Converting Infix to Postfix
```python
def infix_to_postfix(expression):
    precedence = {'+': 1, '-': 1, '*': 2, '/': 2, '^': 3}
    stack = []
    postfix = []
    
    for token in expression.split():
        if token.isalnum():  # Operand
            postfix.append(token)
        elif token == '(':
            stack.append(token)
        elif token == ')':
            while stack and stack[-1] != '(':
                postfix.append(stack.pop())
            stack.pop()  # Discard the '('
        else:  # Operator
            while (stack and stack[-1] != '(' and 
                   precedence.get(stack[-1], 0) >= precedence.get(token, 0)):
                postfix.append(stack.pop())
            stack.append(token)
    
    while stack:
        postfix.append(stack.pop())
    
    return ' '.join(postfix)

# Example: infix_to_postfix("a + b * c")  # Returns "a b c * +"
```

## Monotonic Stack Applications 📈

A monotonic stack is a stack where elements are either entirely non-increasing or entirely non-decreasing.

### Next Greater Element
```python
def next_greater_element(arr):
    n = len(arr)
    result = [-1] * n  # Default to -1 (no greater element)
    stack = []
    
    for i in range(n):
        # While stack has elements and current element is greater than the element at stack's top
        while stack and arr[i] > arr[stack[-1]]:
            result[stack.pop()] = arr[i]
        stack.append(i)
    
    return result

# Example: next_greater_element([4, 5, 2, 10, 8])  # Returns [5, 10, 10, -1, -1]
```

### Largest Rectangle in Histogram
```python
def largest_rectangle_area(heights):
    stack = []  # Stack of indices
    max_area = 0
    i = 0
    
    while i < len(heights):
        # If stack is empty or current height is >= height at stack's top
        if not stack or heights[i] >= heights[stack[-1]]:
            stack.append(i)
            i += 1
        else:
            # Calculate area with the height at stack's top as the smallest height
            top = stack.pop()
            area = heights[top] * (i - stack[-1] - 1 if stack else i)
            max_area = max(max_area, area)
    
    # Process remaining elements in the stack
    while stack:
        top = stack.pop()
        area = heights[top] * (i - stack[-1] - 1 if stack else i)
        max_area = max(max_area, area)
    
    return max_area

# Example: largest_rectangle_area([2, 1, 5, 6, 2, 3])  # Returns 10
```

## Advantages and Disadvantages ⚖️

### Advantages ✅
- Simple and easy to implement
- Efficient operations (constant time)
- Memory efficient (when implemented with arrays)
- Useful for solving many algorithmic problems
- Natural choice for problems with LIFO behavior

### Disadvantages ❌
- Limited access (only top element)
- No random access to elements
- Fixed size in array implementation (unless using dynamic arrays)
- May cause stack overflow if too many elements are added

## Real-world Applications 🌐
- Function call management (call stack)
- Expression evaluation and syntax parsing
- Undo mechanisms in text editors
- Browser history (back button)
- Backtracking algorithms
- Depth-first search in graphs
- Memory management in programming languages

## Further Reading 📚
- Books:
  - "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein
  - "Data Structures and Algorithms in Python" by Goodrich, Tamassia, and Goldwasser
  
- Online Resources:
  - [GeeksforGeeks Stack Data Structure](https://www.geeksforgeeks.org/stack-data-structure/)
  - [Visualgo Stack Visualization](https://visualgo.net/en/list)
  - [HackerRank Stack Problems](https://www.hackerrank.com/domains/data-structures?filters%5Bsubdomains%5D%5B%5D=stacks)
  
- Practice Problems:
  - LeetCode: Valid Parentheses (#20)
  - LeetCode: Min Stack (#155)
  - LeetCode: Evaluate Reverse Polish Notation (#150)
  - LeetCode: Daily Temperatures (#739)

---