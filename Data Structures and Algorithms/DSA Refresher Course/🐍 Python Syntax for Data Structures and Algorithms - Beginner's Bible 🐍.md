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
items = None            # NoneType

# Type checking
print(type(x))          # <class 'int'>
print(isinstance(x, int))  # True - O(1)

# Printing and formatting
print(f"Value: {x}")    # f-strings (Python 3.6+)
print("Value:", x)      # Multiple arguments
print(f"Formatted: {y:.2f}")  # 3.14 (2 decimal places)
```

## String Operations 🔤

```python
# String creation and manipulation
text = "Hello World"
chars = list(text)      # ['H', 'e', 'l', 'l', 'o', ' ', 'W', 'o', 'r', 'l', 'd'] - O(n)
words = text.split()    # ['Hello', 'World'] - O(n)
words_custom = text.split('l')  # ['He', '', 'o Wor', 'd'] - O(n)

# Joining
joined = ''.join(chars)         # "Hello World" - O(n)
csv_line = ','.join(words)      # "Hello,World" - O(n)
spaced = ' '.join(['a', 'b'])   # "a b" - O(n)

# String methods
text.lower()            # "hello world" - O(n)
text.upper()            # "HELLO WORLD" - O(n)
text.strip()            # Remove whitespace from ends - O(n)
text.replace('o', '0')  # "Hell0 W0rld" - O(n)
text.startswith('Hello')  # True - O(k) where k is prefix length
text.endswith('World')    # True - O(k) where k is suffix length
text.find('World')        # 6 (index of substring, -1 if not found) - O(n)
text.count('l')           # 3 (count occurrences) - O(n)

# String formatting
name, age = "Alice", 25
formatted = f"{name} is {age} years old"
formatted2 = "{} is {} years old".format(name, age)
formatted3 = "%s is %d years old" % (name, age)

# Character operations
char = 'A'
print(ord(char))        # 65 (ASCII value) - O(1)
print(chr(65))          # 'A' (character from ASCII) - O(1)

# String validation
"123".isdigit()         # True - O(n)
"abc".isalpha()         # True - O(n)
"abc123".isalnum()      # True - O(n)
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

# Ternary operator
result = "positive" if x > 0 else "non-positive"

# Loops
for i in range(5):      # 0, 1, 2, 3, 4  - O(n)
    print(i)

for i in range(1, 6):   # 1, 2, 3, 4, 5 - O(n)
    print(i)

for i in range(0, 10, 2):  # 0, 2, 4, 6, 8 (step=2) - O(n)
    print(i)

for item in [1, 2, 3]:  # Iterate over a list - O(n)
    print(item)

# Enumerate for index and value
for index, value in enumerate(['a', 'b', 'c']):  # O(n)
    print(f"{index}: {value}")  # 0: a, 1: b, 2: c

# Zip for parallel iteration
names = ['Alice', 'Bob']
ages = [25, 30]
for name, age in zip(names, ages):  # O(min(len(names), len(ages)))
    print(f"{name}: {age}")

while x > 0:
    x -= 1              # Decrement x - O(n) where n is initial value of x

# Loop control
for i in range(10):
    if i == 3:
        continue        # Skip to next iteration
    if i == 7:
        break          # Exit loop
    print(i)
```

## Lists 📋

```python
# Creating lists
nums = [1, 2, 3, 4, 5]
mixed = [1, "hello", True, 3.14]
empty = []
repeated = [0] * 5      # [0, 0, 0, 0, 0] - O(n)
matrix = [[0] * 3 for _ in range(2)]  # [[0, 0, 0], [0, 0, 0]] - O(n*m)

# List comprehensions
squares = [x**2 for x in range(5)]      # [0, 1, 4, 9, 16] - O(n)
evens = [x for x in range(10) if x % 2 == 0]  # [0, 2, 4, 6, 8] - O(n)
words = ['hello', 'world', 'python']
lengths = [len(word) for word in words]  # [5, 5, 6] - O(n)

# Nested list comprehension
matrix = [[i+j for j in range(3)] for i in range(2)]  # [[0, 1, 2], [1, 2, 3]] - O(n*m)

# Accessing elements (0-indexed)
first = nums[0]         # 1 - O(1)
last = nums[-1]         # 5 - O(1)
second_last = nums[-2]  # 4 - O(1)

# Slicing
subset = nums[1:4]      # [2, 3, 4] (start:end, end exclusive) - O(k) where k is slice size
first_three = nums[:3]  # [1, 2, 3] - O(k)
last_three = nums[-3:]  # [3, 4, 5] - O(k)
every_second = nums[::2]  # [1, 3, 5] (step=2) - O(k)
reversed_list = nums[::-1]  # [5, 4, 3, 2, 1] - O(n)

# Common operations
nums.append(6)          # Add to end: [1, 2, 3, 4, 5, 6] - O(1)
nums.extend([7, 8])     # Add multiple elements: [1, 2, 3, 4, 5, 6, 7, 8] - O(k)
nums.insert(0, 0)       # Insert at index: [0, 1, 2, 3, 4, 5, 6, 7, 8] - O(n)
nums.pop()              # Remove and return last element - O(1)
nums.pop(0)             # Remove and return element at index 0 - O(n)
nums.remove(3)          # Remove first occurrence of value - O(n)
index = nums.index(4)   # Find index of first occurrence - O(n)
count = nums.count(1)   # Count occurrences - O(n)
length = len(nums)      # Get length of list - O(1)

# List operations
list1 = [1, 2, 3]
list2 = [4, 5, 6]
combined = list1 + list2  # [1, 2, 3, 4, 5, 6] - O(n+m)
repeated = list1 * 3      # [1, 2, 3, 1, 2, 3, 1, 2, 3] - O(n*k)

# Copying lists
shallow_copy = nums.copy()     # or nums[:] or list(nums) - O(n)
import copy
deep_copy = copy.deepcopy(nums)  # For nested structures - O(n)

# Sorting and reversing
nums.sort()             # Sort in-place - O(n log n)
nums.sort(reverse=True) # Descending order - O(n log n)
sorted_nums = sorted(nums)  # Returns new sorted list - O(n log n)
nums.reverse()          # Reverse in-place - O(n)
```

## Tuples 📦

```python
# Creating tuples (immutable)
point = (3, 4)
single = (5,)           # Note the comma for single element
empty_tuple = ()
coordinates = 1, 2, 3   # Parentheses optional

# Tuple unpacking
x, y = point           # x=3, y=4 - O(1)
first, *rest = (1, 2, 3, 4)  # first=1, rest=[2, 3, 4] - O(1)
*beginning, last = (1, 2, 3, 4)  # beginning=[1, 2, 3], last=4 - O(1)

# Named tuples
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(3, 4)
print(p.x, p.y)        # 3 4 - O(1)
```

## Dictionaries 🔑

```python
# Creating dictionaries
person = {"name": "Alice", "age": 25, "is_student": True}
empty_dict = {}
dict_from_lists = dict(zip(['a', 'b', 'c'], [1, 2, 3]))  # {'a': 1, 'b': 2, 'c': 3} - O(n)

# Dictionary comprehensions
squares = {x: x**2 for x in range(5)}  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16} - O(n)
filtered = {k: v for k, v in person.items() if isinstance(v, str)}  # O(n)

# Accessing elements
name = person["name"]           # Alice - O(1) average, O(n) worst case
age = person.get("age", 0)      # 25 (returns 0 if key doesn't exist) - O(1) average
height = person.setdefault("height", 170)  # Sets and returns default if key missing - O(1) average

# Modifying dictionaries
person["age"] = 26              # Update value - O(1) average
person["city"] = "New York"     # Add new key-value pair - O(1) average
person.update({"country": "USA", "age": 27})  # Update multiple - O(k) where k is number of items
del person["is_student"]        # Remove key-value pair - O(1) average
removed = person.pop("city", "Unknown")  # Remove and return value - O(1) average

# Useful operations
keys = list(person.keys())      # Get all keys as list - O(n)
values = list(person.values())  # Get all values as list - O(n)
items = list(person.items())    # Get all key-value pairs as list of tuples - O(n)

# Check if key exists
if "name" in person:            # O(1) average, O(n) worst case
    print("Name exists")

# Merging dictionaries (Python 3.9+)
dict1 = {"a": 1, "b": 2}
dict2 = {"c": 3, "d": 4}
merged = dict1 | dict2          # {"a": 1, "b": 2, "c": 3, "d": 4} - O(n+m)

# Default dictionaries
from collections import defaultdict
dd = defaultdict(list)          # Default value is empty list
dd['key'].append('value')       # No KeyError, creates list automatically - O(1) average
```

## Sets 🔢

```python
# Creating sets (unordered collections of unique elements)
unique_nums = {1, 2, 3, 4, 5}
duplicate_nums = {1, 2, 2, 3, 4, 4, 5}  # Will store as {1, 2, 3, 4, 5}
empty_set = set()               # Note: {} creates an empty dictionary
from_list = set([1, 2, 2, 3])  # {1, 2, 3} - O(n)
from_string = set("hello")      # {'h', 'e', 'l', 'o'} - O(n)

# Set comprehensions
squares = {x**2 for x in range(5)}  # {0, 1, 4, 9, 16} - O(n)

# Common operations
unique_nums.add(6)              # Add element - O(1) average, O(n) worst case
unique_nums.update([7, 8, 9])   # Add multiple elements - O(k) average
unique_nums.remove(1)           # Remove element (raises error if not found) - O(1) average
unique_nums.discard(10)         # Remove if present (no error if not found) - O(1) average
popped = unique_nums.pop()      # Remove and return arbitrary element - O(1) average

# Set operations
set1 = {1, 2, 3}
set2 = {3, 4, 5}
union = set1 | set2             # {1, 2, 3, 4, 5} - O(len(set1) + len(set2))
intersection = set1 & set2      # {3} - O(min(len(set1), len(set2)))
difference = set1 - set2        # {1, 2} - O(len(set1))
symmetric_diff = set1 ^ set2    # {1, 2, 4, 5} (elements in either set, but not both) - O(len(set1) + len(set2))

# Set comparisons
is_subset = set1 <= set2        # False (is set1 subset of set2?) - O(len(set1))
is_superset = set1 >= set2      # False (is set1 superset of set2?) - O(len(set2))
is_disjoint = set1.isdisjoint(set2)  # False (no common elements?) - O(min(len(set1), len(set2)))
```

## Functions 🧩

```python
# Defining functions
def greet(name):
    return f"Hello, {name}!"

# Function with default parameter
def power(base, exponent=2):
    return base ** exponent     # O(log n) for integer exponents

# Variable arguments
def sum_all(*args):             # *args collects positional arguments into tuple
    return sum(args)            # O(n)

def print_info(**kwargs):       # **kwargs collects keyword arguments into dict
    for key, value in kwargs.items():  # O(n)
        print(f"{key}: {value}")

# Lambda functions (anonymous functions)
square = lambda x: x**2         # O(1)
add = lambda x, y: x + y        # O(1)

# Higher-order functions
numbers = [1, 2, 3, 4, 5]
squared = list(map(square, numbers))        # [1, 4, 9, 16, 25] - O(n)
evens = list(filter(lambda x: x % 2 == 0, numbers))  # [2, 4] - O(n)

from functools import reduce
product = reduce(lambda x, y: x * y, numbers)  # 120 (1*2*3*4*5) - O(n)

# Multiple return values (returns a tuple)
def min_max(numbers):
    return min(numbers), max(numbers)  # O(n) each, so O(n) combined

# Unpacking return values
minimum, maximum = min_max([1, 2, 3, 4, 5])

# Decorators (basic example)
def timer(func):
    import time
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timer
def slow_function():
    import time
    time.sleep(1)
```

## File I/O 📁

```python
# Reading files
with open('file.txt', 'r') as f:
    content = f.read()          # Read entire file - O(n)
    
with open('file.txt', 'r') as f:
    lines = f.readlines()       # Read all lines into list - O(n)
    
with open('file.txt', 'r') as f:
    for line in f:              # Read line by line (memory efficient) - O(1) per line
        print(line.strip())

# Writing files
with open('output.txt', 'w') as f:
    f.write("Hello, World!")    # Write string - O(n)
    
with open('output.txt', 'w') as f:
    lines = ['Line 1\n', 'Line 2\n']
    f.writelines(lines)         # Write list of strings - O(n)

# JSON handling
import json

data = {"name": "Alice", "age": 25}
with open('data.json', 'w') as f:
    json.dump(data, f)          # Write JSON to file - O(n)

with open('data.json', 'r') as f:
    loaded_data = json.load(f)  # Read JSON from file - O(n)

# CSV handling
import csv

with open('data.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(['Name', 'Age'])    # Write header - O(k)
    writer.writerow(['Alice', 25])      # Write row - O(k)

with open('data.csv', 'r') as f:
    reader = csv.reader(f)
    for row in reader:          # Read row by row - O(1) per row
        print(row)
```

## Exception Handling ⚠️

```python
# Basic try-except
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero!")

# Multiple exceptions
try:
    value = int(input("Enter a number: "))
    result = 10 / value
except ValueError:
    print("Invalid number format!")
except ZeroDivisionError:
    print("Cannot divide by zero!")
except Exception as e:          # Catch all other exceptions
    print(f"An error occurred: {e}")

# Finally block
try:
    file = open('data.txt', 'r')
    data = file.read()
except FileNotFoundError:
    print("File not found!")
finally:
    if 'file' in locals():
        file.close()            # Always executed

# Raising exceptions
def validate_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative")
    if age > 150:
        raise ValueError("Age seems unrealistic")
    return age
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
def two_sum_sorted(nums, target):  # O(n) time, O(1) space
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

# Sliding window - fixed size
def max_subarray_sum_k(arr, k):  # O(n) time, O(1) space
    if len(arr) < k:
        return None
    
    window_sum = sum(arr[:k])
    max_sum = window_sum
    
    for i in range(k, len(arr)):
        window_sum = window_sum + arr[i] - arr[i-k]
        max_sum = max(max_sum, window_sum)
    
    return max_sum

# Sliding window - variable size
def longest_substring_without_repeating(s):  # O(n) time, O(min(m,n)) space
    char_set = set()
    left = 0
    max_length = 0
    
    for right in range(len(s)):
        while s[right] in char_set:
            char_set.remove(s[left])
            left += 1
        char_set.add(s[right])
        max_length = max(max_length, right - left + 1)
    
    return max_length

# Prefix sum
def range_sum_query(nums):  # O(n) preprocessing, O(1) query
    prefix_sum = [0]
    for num in nums:
        prefix_sum.append(prefix_sum[-1] + num)
    
    def query(left, right):  # Sum from index left to right (inclusive)
        return prefix_sum[right + 1] - prefix_sum[left]
    
    return query
```

### Recursion and Backtracking

```python
# Factorial
def factorial(n):  # O(n) time, O(n) space due to call stack
    if n <= 1:
        return 1
    return n * factorial(n-1)

# Fibonacci with memoization
def fibonacci_memo(n, memo={}):  # O(n) time, O(n) space
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fibonacci_memo(n-1, memo) + fibonacci_memo(n-2, memo)
    return memo[n]

# Permutations
def permutations(nums):  # O(n! * n) time, O(n! * n) space
    result = []
    
    def backtrack(current_perm):
        if len(current_perm) == len(nums):
            result.append(current_perm[:])  # Make a copy
            return
        
        for num in nums:
            if num not in current_perm:
                current_perm.append(num)
                backtrack(current_perm)
                current_perm.pop()  # Backtrack
    
    backtrack([])
    return result

# Combinations
def combinations(nums, k):  # O(C(n,k) * k) time
    result = []
    
    def backtrack(start, current_combo):
        if len(current_combo) == k:
            result.append(current_combo[:])
            return
        
        for i in range(start, len(nums)):
            current_combo.append(nums[i])
            backtrack(i + 1, current_combo)
            current_combo.pop()
    
    backtrack(0, [])
    return result
```

### Sorting and Searching

```python
# Built-in sorting
sorted_list = sorted(nums)              # Returns new sorted list - O(n log n)
nums.sort()                             # Sorts in-place - O(n log n)
nums.sort(reverse=True)                 # Descending order - O(n log n)
students.sort(key=lambda x: x["age"])   # Sort by specific key - O(n log n)

# Custom sorting with multiple criteria
students = [("Alice", 85), ("Bob", 90), ("Charlie", 85)]
# Sort by grade (descending), then by name (ascending)
students.sort(key=lambda x: (-x[1], x[0]))  # O(n log n)

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

# Binary search for insertion point
def binary_search_insert(arr, target):  # O(log n) time, O(1) space
    left, right = 0, len(arr)
    while left < right:
        mid = (left + right) // 2
        if arr[mid] < target:
            left = mid + 1
        else:
            right = mid
    return left

# Quick sort implementation
def quicksort(arr):  # O(n log n) average, O(n²) worst case
    if len(arr) <= 1:
        return arr
    
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    
    return quicksort(left) + middle + quicksort(right)

# Merge sort implementation
def mergesort(arr):  # O(n log n) time, O(n) space
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = mergesort(arr[:mid])
    right = mergesort(arr[mid:])
    
    return merge(left, right)

def merge(left, right):  # O(n) time, O(n) space
    result = []
    i = j = 0
    
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    
    result.extend(left[i:])
    result.extend(right[j:])
    return result
```

## Advanced Data Structures 🏗️

### Stack (using list)

```python
stack = []
stack.append(1)     # Push - O(1)
stack.append(2)
stack.append(3)
top = stack[-1] if stack else None     # Peek at top element - O(1)
item = stack.pop() if stack else None  # Pop (removes and returns top element) - O(1)
is_empty = len(stack) == 0             # Check if empty - O(1)

# Stack for parentheses matching
def is_valid_parentheses(s):  # O(n) time, O(n) space
    stack = []
    mapping = {')': '(', '}': '{', ']': '['}
    
    for char in s:
        if char in mapping:
            if not stack or stack.pop() != mapping[char]:
                return False
        else:
            stack.append(char)
    
    return len(stack) == 0
```

### Queue (using collections.deque)

```python
from collections import deque

queue = deque()
queue.append(1)     # Enqueue - O(1)
queue.append(2)
queue.append(3)
front = queue[0] if queue else None    # Peek at front element - O(1)
item = queue.popleft() if queue else None  # Dequeue (removes and returns front element) - O(1)
is_empty = len(queue) == 0             # Check if empty - O(1)

# BFS using queue
def bfs_tree(root):  # O(n) time, O(w) space where w is max width
    if not root:
        return []
    
    result = []
    queue = deque([root])
    
    while queue:
        node = queue.popleft()
        result.append(node.val)
        
        if node.left:
            queue.append(node.left)
        if node.right:
            queue.append(node.right)
    
    return result
```

### Priority Queue (using heapq)

```python
import heapq

# Min heap (default)
min_heap = []
heapq.heappush(min_heap, 3)     # O(log n)
heapq.heappush(min_heap, 1)
heapq.heappush(min_heap, 4)
smallest = heapq.heappop(min_heap)  # Returns 1 - O(log n)

# Max heap (negate values)
max_heap = []
heapq.heappush(max_heap, -3)    # Store negative values
heapq.heappush(max_heap, -1)
heapq.heappush(max_heap, -4)
largest = -heapq.heappop(max_heap)  # Returns 4 - O(log n)

# Heapify existing list
nums = [3, 1, 4, 1, 5, 9, 2, 6]
heapq.heapify(nums)             # Convert to min-heap in-place - O(n)

# K largest/smallest elements
k_largest = heapq.nlargest(3, nums)    # O(n log k)
k_smallest = heapq.nsmallest(3, nums)  # O(n log k)
```

### Linked List

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

class LinkedList:
    def __init__(self):
        self.head = None
        self.size = 0
    
    def append(self, val):  # O(n) time
        new_node = ListNode(val)
        if not self.head:
            self.head = new_node
        else:
            current = self.head
            while current.next:
                current = current.next
            current.next = new_node
        self.size += 1
    
    def prepend(self, val):  # O(1) time
        new_node = ListNode(val)
        new_node.next = self.head
        self.head = new_node
        self.size += 1
    
    def delete(self, val):  # O(n) time
        if not self.head:
            return False
        
        if self.head.val == val:
            self.head = self.head.next
            self.size -= 1
            return True
        
        current = self.head
        while current.next:
            if current.next.val == val:
                current.next = current.next.next
                self.size -= 1
                return True
            current = current.next
        return False
    
    def find(self, val):  # O(n) time
        current = self.head
        index = 0
        while current:
            if current.val == val:
                return index
            current = current.next
            index += 1
        return -1
    
    def to_list(self):  # O(n) time
        result = []
        current = self.head
        while current:
            result.append(current.val)
            current = current.next
        return result

# Common linked list operations
def reverse_linked_list(head):  # O(n) time, O(1) space
    prev = None
    current = head
    
    while current:
        next_temp = current.next
        current.next = prev
        prev = current
        current = next_temp
    
    return prev

def has_cycle(head):  # O(n) time, O(1) space - Floyd's cycle detection
    if not head or not head.next:
        return False
    
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    
    return False
```

### Binary Tree

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

# Tree traversals
def inorder_traversal(root):  # O(n) time, O(h) space where h is height
    result = []
    
    def inorder(node):
        if node:
            inorder(node.left)
            result.append(node.val)
            inorder(node.right)
    
    inorder(root)
    return result

def preorder_traversal(root):  # O(n) time, O(h) space
    result = []
    
    def preorder(node):
        if node:
            result.append(node.val)
            preorder(node.left)
            preorder(node.right)
    
    preorder(root)
    return result

def postorder_traversal(root):  # O(n) time, O(h) space
    result = []
    
    def postorder(node):
        if node:
            postorder(node.left)
            postorder(node.right)
            result.append(node.val)
    
    postorder(root)
    return result

# Level order traversal (BFS)
def level_order_traversal(root):  # O(n) time, O(w) space where w is max width
    if not root:
        return []
    
    result = []
    queue = deque([root])
    
    while queue:
        level_size = len(queue)
        level = []
        
        for _ in range(level_size):
            node = queue.popleft()
            level.append(node.val)
            
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        
        result.append(level)
    
    return result

# Tree properties
def max_depth(root):  # O(n) time, O(h) space
    if not root:
        return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))

def is_balanced(root):  # O(n) time, O(h) space
    def check_balance(node):
        if not node:
            return 0, True
        
        left_height, left_balanced = check_balance(node.left)
        right_height, right_balanced = check_balance(node.right)
        
        balanced = (left_balanced and right_balanced and 
                   abs(left_height - right_height) <= 1)
        height = 1 + max(left_height, right_height)
        
        return height, balanced
    
    _, balanced = check_balance(root)
    return balanced
```

### Graph Representations and Algorithms

```python
# Adjacency list representation
def create_graph(edges):  # O(E) time, O(V + E) space
    graph = defaultdict(list)
    for u, v in edges:
        graph[u].append(v)
        graph[v].append(u)  # For undirected graph
    return graph

# DFS
def dfs(graph, start, visited=None):  # O(V + E) time, O(V) space
    if visited is None:
        visited = set()
    
    visited.add(start)
    result = [start]
    
    for neighbor in graph[start]:
        if neighbor not in visited:
            result.extend(dfs(graph, neighbor, visited))
    
    return result

# BFS
def bfs(graph, start):  # O(V + E) time, O(V) space
    visited = set([start])
    queue = deque([start])
    result = []
    
    while queue:
        vertex = queue.popleft()
        result.append(vertex)
        
        for neighbor in graph[vertex]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    
    return result

# Shortest path in unweighted graph
def shortest_path(graph, start, end):  # O(V + E) time, O(V) space
    if start == end:
        return [start]
    
    visited = set([start])
    queue = deque([(start, [start])])
    
    while queue:
        vertex, path = queue.popleft()
        
        for neighbor in graph[vertex]:
            if neighbor == end:
                return path + [neighbor]
            
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, path + [neighbor]))
    
    return []  # No path found

# Topological sort (for DAG)
def topological_sort(graph):  # O(V + E) time, O(V) space
    in_degree = defaultdict(int)
    
    # Calculate in-degrees
    for vertex in graph:
        for neighbor in graph[vertex]:
            in_degree[neighbor] += 1
    
    # Find vertices with no incoming edges
    queue = deque([v for v in graph if in_degree[v] == 0])
    result = []
    
    while queue:
        vertex = queue.popleft()
        result.append(vertex)
        
        for neighbor in graph[vertex]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)
    
    return result if len(result) == len(graph) else []  # Check for cycles
```

## Dynamic Programming Patterns 💡

```python
# 1D DP - Fibonacci
def fibonacci_dp(n):  # O(n) time, O(n) space
    if n <= 1:
        return n
    
    dp = [0] * (n + 1)
    dp[1] = 1
    
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    
    return dp[n]

# Space optimized version
def fibonacci_optimized(n):  # O(n) time, O(1) space
    if n <= 1:
        return n
    
    prev2, prev1 = 0, 1
    for i in range(2, n + 1):
        current = prev1 + prev2
        prev2, prev1 = prev1, current
    
    return prev1

# Climbing stairs
def climb_stairs(n):  # O(n) time, O(1) space
    if n <= 2:
        return n
    
    prev2, prev1 = 1, 2
    for i in range(3, n + 1):
        current = prev1 + prev2
        prev2, prev1 = prev1, current
    
    return prev1

# Coin change problem
def coin_change(coins, amount):  # O(amount * len(coins)) time, O(amount) space
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    
    for coin in coins:
        for i in range(coin, amount + 1):
            dp[i] = min(dp[i], dp[i - coin] + 1)
    
    return dp[amount] if dp[amount] != float('inf') else -1

# Longest increasing subsequence
def longest_increasing_subsequence(nums):  # O(n²) time, O(n) space
    if not nums:
        return 0
    
    dp = [1] * len(nums)
    
    for i in range(1, len(nums)):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    
    return max(dp)

# 2D DP - Unique paths
def unique_paths(m, n):  # O(m*n) time, O(m*n) space
    dp = [[1] * n for _ in range(m)]
    
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = dp[i-1][j] + dp[i][j-1]
    
    return dp[m-1][n-1]

# Space optimized version
def unique_paths_optimized(m, n):  # O(m*n) time, O(n) space
    dp = [1] * n
    
    for i in range(1, m):
        for j in range(1, n):
            dp[j] += dp[j-1]
    
    return dp[n-1]

# Edit distance (Levenshtein distance)
def edit_distance(word1, word2):  # O(m*n) time, O(m*n) space
    m, n = len(word1), len(word2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    # Initialize base cases
    for i in range(m + 1):
        dp[i][0] = i
    for j in range(n + 1):
        dp[0][j] = j
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if word1[i-1] == word2[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(
                    dp[i-1][j],    # deletion
                    dp[i][j-1],    # insertion
                    dp[i-1][j-1]   # substitution
                )
    
    return dp[m][n]
```

## Useful Built-in Functions and Libraries 🛠️

```python
# Built-in functions
max_val = max(nums)                # Maximum value - O(n)
min_val = min(nums)                # Minimum value - O(n)
total = sum(nums)                  # Sum of values - O(n)
exists = any([False, True, False]) # True if any element is True - O(n)
all_true = all([True, True, True]) # True if all elements are True - O(n)
length = len(nums)                 # Length of sequence - O(1)
absolute = abs(-5)                 # Absolute value - O(1)
rounded = round(3.14159, 2)        # Round to 2 decimal places - O(1)

# Math operations
import math
math.ceil(3.2)      # 4 (ceiling)
math.floor(3.8)     # 3 (floor)
math.sqrt(16)       # 4.0 (square root)
math.log(8, 2)      # 3.0 (logarithm base 2)
math.gcd(12, 18)    # 6 (greatest common divisor)
math.factorial(5)   # 120

# Random operations
import random
random.randint(1, 10)           # Random integer between 1 and 10 (inclusive)
random.choice(['a', 'b', 'c'])  # Random element from sequence
random.shuffle(nums)            # Shuffle list in-place - O(n)
random.sample(nums, 3)          # Random sample of 3 elements without replacement - O(k)

# Collections module
from collections import Counter, defaultdict, deque, OrderedDict

# Count occurrences
word = "mississippi"
counts = Counter(word)  # Counter({'i': 4, 's': 4, 'p': 2, 'm': 1}) - O(n)
most_common = counts.most_common(2)  # [('i', 4), ('s', 4)] - O(n log n)

# Default dictionary (never raises KeyError)
word_lengths = defaultdict(int)  # Default value is 0
for word in ["apple", "banana", "cherry"]:  # O(n)
    word_lengths[word] = len(word)  # O(1) per operation

# Ordered dictionary (maintains insertion order)
od = OrderedDict()
od['first'] = 1
od['second'] = 2
od.move_to_end('first')  # Move to end - O(1)

# Itertools module
import itertools

# Combinations and permutations
list(itertools.combinations([1, 2, 3], 2))     # [(1, 2), (1, 3), (2, 3)]
list(itertools.permutations([1, 2, 3], 2))     # [(1, 2), (1, 3), (2, 1), (2, 3), (3, 1), (3, 2)]
list(itertools.product([1, 2], ['a', 'b']))    # [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]

# Infinite iterators
counter = itertools.count(start=0, step=2)     # 0, 2, 4, 6, 8, ...
cycler = itertools.cycle(['A', 'B', 'C'])      # A, B, C, A, B, C, ...
repeater = itertools.repeat('hello', 3)        # hello, hello, hello

# Grouping
data = [('a', 1), ('a', 2), ('b', 3), ('b', 4)]
grouped = itertools.groupby(data, key=lambda x: x[0])
for key, group in grouped:
    print(key, list(group))  # a [('a', 1), ('a', 2)], b [('b', 3), ('b', 4)]

# Heapq module (priority queue)
import heapq

nums = [5, 7, 9, 1, 3]
heapq.heapify(nums)               # Convert list to min-heap in-place - O(n)
smallest = heapq.heappop(nums)    # Remove and return smallest item - O(log n)
heapq.heappush(nums, 4)           # Add new item - O(log n)
three_smallest = heapq.nsmallest(3, [5, 7, 9, 1, 3])  # [1, 3, 5] - O(n log k)

# Bisect module (binary search)
import bisect

sorted_list = [1, 3, 4, 7, 9]
index = bisect.bisect_left(sorted_list, 4)   # 2 (leftmost position) - O(log n)
index = bisect.bisect_right(sorted_list, 4)  # 3 (rightmost position) - O(log n)
bisect.insort(sorted_list, 5)               # Insert 5 in sorted order - O(n)

# Functools module
from functools import reduce, lru_cache

# Reduce
product = reduce(lambda x, y: x * y, [1, 2, 3, 4])  # 24 - O(n)

# LRU Cache for memoization
@lru_cache(maxsize=None)
def fibonacci_cached(n):  # O(n) time with caching, O(n) space
    if n <= 1:
        return n
    return fibonacci_cached(n-1) + fibonacci_cached(n-2)

# Operator module
import operator
sorted_students = sorted(students, key=operator.itemgetter(1))  # Sort by second element
```

## Advanced Techniques 🚀

### Bit Manipulation

```python
# Basic bit operations
a = 5   # 101 in binary
b = 3   # 011 in binary

print(a & b)    # 1 (AND: 001)
print(a | b)    # 7 (OR: 111)
print(a ^ b)    # 6 (XOR: 110)
print(~a)       # -6 (NOT: inverts all bits)
print(a << 1)   # 10 (left shift: 1010)
print(a >> 1)   # 2 (right shift: 10)

# Check if number is power of 2
def is_power_of_two(n):  # O(1)
    return n > 0 and (n & (n - 1)) == 0

# Count set bits
def count_set_bits(n):  # O(log n)
    count = 0
    while n:
        count += n & 1
        n >>= 1
    return count

# Brian Kernighan's algorithm for counting set bits
def count_set_bits_optimized(n):  # O(number of set bits)
    count = 0
    while n:
        n &= n - 1  # Remove rightmost set bit
        count += 1
    return count

# Find single number (all others appear twice)
def single_number(nums):  # O(n) time, O(1) space
    result = 0
    for num in nums:
        result ^= num  # XOR cancels out duplicates
    return result
```

### Union-Find (Disjoint Set)

```python
class UnionFind:
    def __init__(self, n):  # O(n)
        self.parent = list(range(n))
        self.rank = [0] * n
        self.components = n
    
    def find(self, x):  # O(α(n)) amortized with path compression
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # Path compression
        return self.parent[x]
    
    def union(self, x, y):  # O(α(n)) amortized
        root_x, root_y = self.find(x), self.find(y)
        
        if root_x != root_y:
            # Union by rank
            if self.rank[root_x] < self.rank[root_y]:
                self.parent[root_x] = root_y
            elif self.rank[root_x] > self.rank[root_y]:
                self.parent[root_y] = root_x
            else:
                self.parent[root_y] = root_x
                self.rank[root_x] += 1
            
            self.components -= 1
            return True
        return False
    
    def connected(self, x, y):  # O(α(n)) amortized
        return self.find(x) == self.find(y)
```

### Trie (Prefix Tree)

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end_word = False

class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):  # O(m) where m is word length
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end_word = True
    
    def search(self, word):  # O(m)
        node = self.root
        for char in word:
            if char not in node.children:
                return False
            node = node.children[char]
        return node.is_end_word
    
    def starts_with(self, prefix):  # O(m)
        node = self.root
        for char in prefix:
            if char not in node.children:
                return False
            node = node.children[char]
        return True
    
    def get_words_with_prefix(self, prefix):  # O(p + n) where p is prefix length, n is number of nodes
        node = self.root
        for char in prefix:
            if char not in node.children:
                return []
            node = node.children[char]
        
        words = []
        self._dfs(node, prefix, words)
        return words
    
    def _dfs(self, node, current_word, words):
        if node.is_end_word:
            words.append(current_word)
        
        for char, child_node in node.children.items():
            self._dfs(child_node, current_word + char, words)
```

### Segment Tree

```python
class SegmentTree:
    def __init__(self, arr):  # O(n)
        self.n = len(arr)
        self.tree = [0] * (4 * self.n)
        self.build(arr, 0, 0, self.n - 1)
    
    def build(self, arr, node, start, end):  # O(n)
        if start == end:
            self.tree[node] = arr[start]
        else:
            mid = (start + end) // 2
            self.build(arr, 2 * node + 1, start, mid)
            self.build(arr, 2 * node + 2, mid + 1, end)
            self.tree[node] = self.tree[2 * node + 1] + self.tree[2 * node + 2]
    
    def update(self, node, start, end, idx, val):  # O(log n)
        if start == end:
            self.tree[node] = val
        else:
            mid = (start + end) // 2
            if idx <= mid:
                self.update(2 * node + 1, start, mid, idx, val)
            else:
                self.update(2 * node + 2, mid + 1, end, idx, val)
            self.tree[node] = self.tree[2 * node + 1] + self.tree[2 * node + 2]
    
    def query(self, node, start, end, l, r):  # O(log n)
        if r < start or end < l:
            return 0
        if l <= start and end <= r:
            return self.tree[node]
        
        mid = (start + end) // 2
        left_sum = self.query(2 * node + 1, start, mid, l, r)
        right_sum = self.query(2 * node + 2, mid + 1, end, l, r)
        return left_sum + right_sum
    
    def update_value(self, idx, val):  # O(log n)
        self.update(0, 0, self.n - 1, idx, val)
    
    def range_sum(self, l, r):  # O(log n)
        return self.query(0, 0, self.n - 1, l, r)
```

## Time and Space Complexity Reference ⏱️

```python
# Common time complexities with examples

# O(1) - Constant time
def get_first(arr):
    return arr[0] if arr else None

def hash_lookup(dictionary, key):
    return dictionary.get(key)

# O(log n) - Logarithmic time
def binary_search(arr, target):
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

# O(n) - Linear time
def linear_search(arr, target):
    for i, val in enumerate(arr):
        if val == target:
            return i
    return -1

def sum_array(arr):
    total = 0
    for num in arr:
        total += num
    return total

# O(n log n) - Linearithmic time
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    
    return merge(left, right)

# O(n²) - Quadratic time
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
    return arr

def find_all_pairs(arr):
    pairs = []
    for i in range(len(arr)):
        for j in range(i + 1, len(arr)):
            pairs.append((arr[i], arr[j]))
    return pairs

# O(2^n) - Exponential time
def fibonacci_naive(n):
    if n <= 1:
        return n
    return fibonacci_naive(n - 1) + fibonacci_naive(n - 2)

def generate_all_subsets(arr):
    if not arr:
        return [[]]
    
    subsets = generate_all_subsets(arr[1:])
    return subsets + [subset + [arr[0]] for subset in subsets]

# Space complexity examples

# O(1) - Constant space
def reverse_array_inplace(arr):
    left, right = 0, len(arr) - 1
    while left < right:
        arr[left], arr[right] = arr[right], arr[left]
        left += 1
        right -= 1

# O(n) - Linear space
def create_copy(arr):
    return arr[:]

def fibonacci_dp(n):
    if n <= 1:
        return n
    dp = [0] * (n + 1)
    dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]

# O(log n) - Logarithmic space (recursion depth)
def binary_search_recursive(arr, target, left=0, right=None):
    if right is None:
        right = len(arr) - 1
    
    if left > right:
        return -1
    
    mid = (left + right) // 2
    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        return binary_search_recursive(arr, target, mid + 1, right)
    else:
        return binary_search_recursive(arr, target, left, mid - 1)
```

## Common Patterns and Tricks 🎯

```python
# Fast input/output for competitive programming
import sys
input = sys.stdin.readline

# Multiple assignment
a, b = b, a  # Swap variables
x = y = z = 0  # Initialize multiple variables

# Conditional assignment
max_val = a if a > b else b  # Ternary operator
result = value or default_value  # Use default if value is falsy

# List/string reversal
reversed_list = arr[::-1]
reversed_string = s[::-1]

# Check if all/any elements satisfy condition
all_positive = all(x > 0 for x in numbers)
has_negative = any(x < 0 for x in numbers)

# Find index of max/min element
max_index = numbers.index(max(numbers))
min_index = numbers.index(min(numbers))

# Enumerate with custom start
for i, val in enumerate(arr, start=1):
    print(f"Position {i}: {val}")

# Dictionary get with default and immediate assignment
count = counter.get(key, 0)
counter[key] = count + 1
# Or more concisely:
counter[key] = counter.get(key, 0) + 1

# List flattening
nested = [[1, 2], [3, 4], [5, 6]]
flat = [item for sublist in nested for item in sublist]  # [1, 2, 3, 4, 5, 6]

# Remove duplicates while preserving order
def remove_duplicates(arr):
    seen = set()
    result = []
    for item in arr:
        if item not in seen:
            seen.add(item)
            result.append(item)
    return result

# Or using dict (Python 3.7+)
unique_items = list(dict.fromkeys(arr))

# Frequency counting
from collections import Counter
freq = Counter(arr)
most_common_item = freq.most_common(1)[0][0]

# Matrix operations
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

# Transpose
transposed = list(zip(*matrix))  # [(1, 4, 7), (2, 5, 8), (3, 6, 9)]
transposed = [list(row) for row in zip(*matrix)]  # [[1, 4, 7], [2, 5, 8], [3, 6, 9]]

# Rotate 90 degrees clockwise
rotated = [list(row) for row in zip(*matrix[::-1])]

# Get diagonal
diagonal = [matrix[i][i] for i in range(len(matrix))]

# Sliding window maximum
from collections import deque

def sliding_window_maximum(nums, k):  # O(n) time
    dq = deque()  # Store indices
    result = []
    
    for i in range(len(nums)):
        # Remove indices outside window
        while dq and dq[0] <= i - k:
            dq.popleft()
        
        # Remove indices of smaller elements
        while dq and nums[dq[-1]] <= nums[i]:
            dq.pop()
        
        dq.append(i)
        
        # Add to result when window is complete
        if i >= k - 1:
            result.append(nums[dq[0]])
    
    return result

# Kadane's algorithm for maximum subarray sum
def max_subarray_sum(nums):  # O(n) time, O(1) space
    max_sum = current_sum = nums[0]
    
    for num in nums[1:]:
        current_sum = max(num, current_sum + num)
        max_sum = max(max_sum, current_sum)
    
    return max_sum

# Boyer-Moore majority element
def majority_element(nums):  # O(n) time, O(1) space
    candidate = count = 0
    
    for num in nums:
        if count == 0:
            candidate = num
        count += 1 if num == candidate else -1
    
    return candidate

# Dutch national flag (3-way partitioning)
def sort_colors(nums):  # O(n) time, O(1) space
    low = mid = 0
    high = len(nums) - 1
    
    while mid <= high:
        if nums[mid] == 0:
            nums[low], nums[mid] = nums[mid], nums[low]
            low += 1
            mid += 1
        elif nums[mid] == 1:
            mid += 1
        else:  # nums[mid] == 2
            nums[mid], nums[high] = nums[high], nums[mid]
            high -= 1
```

## Testing and Debugging 🐛

```python
# Basic assertions for testing
def test_function():
    assert binary_search([1, 2, 3, 4, 5], 3) == 2
    assert binary_search([1, 2, 3, 4, 5], 6) == -1
    print("All tests passed!")

# Using unittest module
import unittest

class TestAlgorithms(unittest.TestCase):
    def test_binary_search(self):
        self.assertEqual(binary_search([1, 2, 3, 4, 5], 3), 2)
        self.assertEqual(binary_search([1, 2, 3, 4, 5], 6), -1)
    
    def test_edge_cases(self):
        self.assertEqual(binary_search([], 1), -1)
        self.assertEqual(binary_search([1], 1), 0)

if __name__ == '__main__':
    unittest.main()

# Debugging techniques
import pdb

def debug_function(arr):
    pdb.set_trace()  # Debugger breakpoint
    result = some_complex_operation(arr)
    return result

# Print debugging
def debug_binary_search(arr, target):
    left, right = 0, len(arr) - 1
    
    while left <= right:
        mid = (left + right) // 2
        print(f"left={left}, right={right}, mid={mid}, arr[mid]={arr[mid]}")
        
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return -1

# Timing code execution
import time

def time_function(func, *args):
    start = time.time()
    result = func(*args)
    end = time.time()
    print(f"{func.__name__} took {end - start:.6f} seconds")
    return result

# Memory profiling
import tracemalloc

def profile_memory(func, *args):
    tracemalloc.start()
    result = func(*args)
    current, peak = tracemalloc.get_traced_memory()
    tracemalloc.stop()
    print(f"Current memory usage: {current / 1024 / 1024:.2f} MB")
    print(f"Peak memory usage: {peak / 1024 / 1024:.2f} MB")
    return result
```

## Further Reading 📚

### Essential Resources
- **Official Documentation**: [Python Documentation](https://docs.python.org/3/)
- **Python Standard Library**: [Python Standard Library](https://docs.python.org/3/library/index.html)
- **PEP 8 Style Guide**: [Style Guide for Python Code](https://pep8.org/)

### Algorithm Practice Platforms
- **LeetCode**: [leetcode.com](https://leetcode.com/) - Extensive problem collection with difficulty levels
- **HackerRank**: [hackerrank.com](https://www.hackerrank.com/) - Structured learning paths
- **Codeforces**: [codeforces.com](https://codeforces.com/) - Competitive programming contests
- **AtCoder**: [atcoder.jp](https://atcoder.jp/) - High-quality algorithmic contests
- **CodeChef**: [codechef.com](https://www.codechef.com/) - Monthly contests and practice

### Learning Resources
- **GeeksforGeeks**: [geeksforgeeks.org](https://www.geeksforgeeks.org/python-programming-language/) - Comprehensive DS&A tutorials
- **Algorithm Visualizer**: [algorithm-visualizer.org](https://algorithm-visualizer.org/) - Visual algorithm explanations
- **VisuAlgo**: [visualgo.net](https://visualgo.net/) - Interactive algorithm visualizations

### Books
- **"Cracking the Coding Interview"** by Gayle McDowell - Interview preparation
- **"Introduction to Algorithms"** by Cormen, Leiserson, Rivest, Stein - Comprehensive algorithms textbook
- **"Python Tricks"** by Dan Bader - Advanced Python techniques
- **"Effective Python"** by Brett Slatkin - Best practices and idioms

### Advanced Topics
- **Competitive Programming**: [cp-algorithms.com](https://cp-algorithms.com/) - Advanced algorithms and techniques
- **Python Performance**: [wiki.python.org/moin/PythonSpeed](https://wiki.python.org/moin/PythonSpeed) - Optimization techniques
- **Big O Cheat Sheet**: [bigocheatsheet.com](https://www.bigocheatsheet.com/) - Time and space complexity reference

This comprehensive guide covers the essential Python syntax and patterns needed for data structures and algorithms. The efficiency annotations help you understand the performance characteristics of different operations, which is crucial for writing optimal solutions.