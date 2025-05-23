# Python DS&A Syntax Reminder with Efficiency Annotations

## Installation 📥
```bash
# Install Python
# Windows
winget install Python.Python.3
# or
choco install python

# macOS
brew install python

# Linux
apt install python3     # Debian/Ubuntu
dnf install python3     # Fedora
pacman -S python        # Arch
```

## Basic Python Syntax 📝

```python
# Variables and data types
x = 5                   # Integer
y = 3.14                # Float
name = "Algorithm"      # String
is_valid = True         # Boolean

# Printing and formatting
print(f"Value: {x}")    # f-strings (Python 3.6+)
print("Value:", x)      # Multiple arguments
```

## Control Flow 🔄

```python
# Conditional statements
if x > 0:
    print("Positive")
elif x < 0:
    print("Negative")
else:
    print("Zero")

# Loops
for i in range(5):      # 0, 1, 2, 3, 4  - O(n)
    print(i)

for item in [1, 2, 3]:  # Iterate over a list - O(n)
    print(item)

while x > 0:
    x -= 1              # Decrement x - O(n) where n is initial value of x
```

## Lists 📋

```python
# Creating lists
nums = [1, 2, 3, 4, 5]
mixed = [1, "hello", True, 3.14]
empty = []

# Accessing elements (0-indexed)
first = nums[0]         # 1 - O(1)
last = nums[-1]         # 5 - O(1)

# Slicing
subset = nums[1:4]      # [2, 3, 4] (start:end, end exclusive) - O(k) where k is slice size
first_three = nums[:3]  # [1, 2, 3] - O(k) where k is slice size
last_three = nums[-3:]  # [3, 4, 5] - O(k) where k is slice size

# Common operations
nums.append(6)          # Add to end: [1, 2, 3, 4, 5, 6] - O(1)
nums.insert(0, 0)       # Insert at index: [0, 1, 2, 3, 4, 5, 6] - O(n)
nums.pop()              # Remove and return last element - O(1)
nums.pop(0)             # Remove and return element at index 0 - O(n)
nums.remove(3)          # Remove first occurrence of value - O(n)
length = len(nums)      # Get length of list - O(1)
```

## Dictionaries 🔑

```python
# Creating dictionaries
person = {"name": "Alice", "age": 25, "is_student": True}
empty_dict = {}

# Accessing elements
name = person["name"]           # Alice - O(1) average, O(n) worst case
# Safe access with default
age = person.get("age", 0)      # 25 (returns 0 if key doesn't exist) - O(1) average, O(n) worst case

# Modifying dictionaries
person["age"] = 26              # Update value - O(1) average, O(n) worst case
person["city"] = "New York"     # Add new key-value pair - O(1) average, O(n) worst case
del person["is_student"]        # Remove key-value pair - O(1) average, O(n) worst case

# Useful operations
keys = person.keys()            # Get all keys - O(1)
values = person.values()        # Get all values - O(1)
items = person.items()          # Get all key-value pairs as tuples - O(1)

# Check if key exists
if "name" in person:            # O(1) average, O(n) worst case
    print("Name exists")
```

## Sets 🔢

```python
# Creating sets (unordered collections of unique elements)
unique_nums = {1, 2, 3, 4, 5}
duplicate_nums = {1, 2, 2, 3, 4, 4, 5}  # Will store as {1, 2, 3, 4, 5}
empty_set = set()               # Note: {} creates an empty dictionary

# Common operations
unique_nums.add(6)              # Add element - O(1) average, O(n) worst case
unique_nums.remove(1)           # Remove element (raises error if not found) - O(1) average, O(n) worst case
unique_nums.discard(10)         # Remove if present (no error if not found) - O(1) average, O(n) worst case

# Set operations
set1 = {1, 2, 3}
set2 = {3, 4, 5}
union = set1 | set2             # {1, 2, 3, 4, 5} - O(len(set1) + len(set2))
intersection = set1 & set2      # {3} - O(min(len(set1), len(set2)))
difference = set1 - set2        # {1, 2} - O(len(set1))
```

## Functions 🧩

```python
# Defining functions
def greet(name):
    return f"Hello, {name}!"

# Function with default parameter
def power(base, exponent=2):
    return base ** exponent     # O(log n) for integer exponents

# Multiple return values (returns a tuple)
def min_max(numbers):
    return min(numbers), max(numbers)  # O(n) each, so O(n) combined

# Unpacking return values
minimum, maximum = min_max([1, 2, 3, 4, 5])
```

## Common DS&A Patterns 🧠

### Array/List Traversal

```python
# Linear search
def linear_search(arr, target):  # O(n) time, O(1) space
    for i, val in enumerate(arr):
        if val == target:
            return i
    return -1

# Two-pointer technique
def two_sum(nums, target):  # O(n) time, O(1) space
    left, right = 0, len(nums) - 1
    while left < right:
        current_sum = nums[left] + nums[right]
        if current_sum == target:
            return [left, right]
        elif current_sum < target:
            left += 1
        else:
            right -= 1
    return [-1, -1]
```

### Recursion

```python
# Factorial
def factorial(n):  # O(n) time, O(n) space due to call stack
    if n <= 1:
        return 1
    return n * factorial(n-1)

# Fibonacci
def fibonacci(n):  # O(2^n) time, O(n) space due to call stack
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

### Sorting and Searching

```python
# Built-in sorting
sorted_list = sorted(nums)              # Returns new sorted list - O(n log n)
nums.sort()                             # Sorts in-place - O(n log n)
nums.sort(reverse=True)                 # Descending order - O(n log n)
students.sort(key=lambda x: x["age"])   # Sort by specific key - O(n log n)

# Binary search (on sorted list)
def binary_search(arr, target):  # O(log n) time, O(1) space
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1
```

## Common Data Structures 🏗️

### Stack (using list)

```python
stack = []
stack.append(1)     # Push - O(1)
stack.append(2)
stack.append(3)
top = stack[-1]     # Peek at top element - O(1)
item = stack.pop()  # Pop (removes and returns top element) - O(1)
```

### Queue (using collections.deque)

```python
from collections import deque

queue = deque()
queue.append(1)     # Enqueue - O(1)
queue.append(2)
queue.append(3)
front = queue[0]    # Peek at front element - O(1)
item = queue.popleft()  # Dequeue (removes and returns front element) - O(1)
```

### Linked List

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class LinkedList:
    def __init__(self):
        self.head = None
    
    def append(self, data):  # O(n) time as we traverse to the end
        new_node = Node(data)
        if not self.head:
            self.head = new_node
            return
        
        current = self.head
        while current.next:
            current = current.next
        current.next = new_node
```

## Time and Space Complexity ⏱️

```python
# O(1) - Constant time
def get_first(arr):
    return arr[0] if arr else None

# O(n) - Linear time
def sum_array(arr):
    total = 0
    for num in arr:  # n iterations
        total += num
    return total

# O(n²) - Quadratic time
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):  # n iterations
        for j in range(0, n-i-1):  # n iterations (decreasing)
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]
    return arr
```

## Problem-Solving Techniques 🧩

### Sliding Window

```python
def max_subarray_sum(arr, k):  # O(n) time, O(1) space
    n = len(arr)
    if n < k:
        return None
    
    # Compute sum of first window
    window_sum = sum(arr[:k])  # O(k)
    max_sum = window_sum
    
    # Slide window and update max_sum
    for i in range(k, n):  # O(n-k)
        window_sum = window_sum + arr[i] - arr[i-k]
        max_sum = max(max_sum, window_sum)
    
    return max_sum
```

### Dynamic Programming

```python
# Memoization (top-down)
def fibonacci_memo(n, memo={}):  # O(n) time, O(n) space
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fibonacci_memo(n-1, memo) + fibonacci_memo(n-2, memo)
    return memo[n]

# Tabulation (bottom-up)
def fibonacci_tab(n):  # O(n) time, O(n) space
    if n <= 1:
        return n
    dp = [0] * (n+1)
    dp[1] = 1
    
    for i in range(2, n+1):
        dp[i] = dp[i-1] + dp[i-2]
    
    return dp[n]
```

## Useful Built-in Functions and Libraries 🛠️

```python
# Built-in functions
max_val = max(nums)                # Maximum value - O(n)
min_val = min(nums)                # Minimum value - O(n)
total = sum(nums)                  # Sum of values - O(n)
exists = any([False, True, False]) # True if any element is True - O(n)
all_true = all([True, True, True]) # True if all elements are True - O(n)

# Collections module
from collections import Counter, defaultdict, deque

# Count occurrences
word = "mississippi"
counts = Counter(word)  # {'i': 4, 's': 4, 'p': 2, 'm': 1} - O(n)

# Default dictionary (never raises KeyError)
word_lengths = defaultdict(int)  # Default value is 0
for word in ["apple", "banana", "cherry"]:  # O(n)
    word_lengths[word] = len(word)  # O(1) per operation

# Heapq module (priority queue)
import heapq

nums = [5, 7, 9, 1, 3]
heapq.heapify(nums)               # Convert list to min-heap in-place - O(n)
smallest = heapq.heappop(nums)    # Remove and return smallest item - O(log n)
heapq.heappush(nums, 4)           # Add new item - O(log n)
```

## Further Reading 📚
- Official Documentation: [Python Documentation](https://docs.python.org/3/)
- Python Standard Library: [Python Standard Library](https://docs.python.org/3/library/index.html)
- Algorithm Resources:
  - [LeetCode](https://leetcode.com/) - Practice platform
  - [HackerRank](https://www.hackerrank.com/) - Practice platform
  - [GeeksforGeeks](https://www.geeksforgeeks.org/python-programming-language/) - Python DS&A tutorials
  - [Cracking the Coding Interview](http://www.crackingthecodinginterview.com/) - Popular DS&A book