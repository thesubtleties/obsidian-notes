# Python DS&A Syntax Reference - Table of Contents

## Quick Navigation 🚀

### **Basics & Setup**
- [[#Installation 📥|Installation]]
- [[#Basic Python Syntax 📝|Basic Python Syntax]]
- [[#Character Operations 🔤|Character Operations]]
- [[#Control Flow 🔄|Control Flow]]

### **String Operations 🔤**
- [[#String Operations 🔤|String Operations Overview]]
- [[#String Creation and Basic Operations|String Creation & Basic Operations]]
- [[#String Reversal (Multiple Methods)|String Reversal (Multiple Methods)]]
- [[#String Slicing and Indexing|String Slicing & Indexing]]
- [[#String Searching and Finding|String Searching & Finding]]
- [[#String Modification and Cleaning|String Modification & Cleaning]]
- [[#String Splitting and Joining|String Splitting & Joining]]
- [[#String Validation and Testing|String Validation & Testing]]
- [[#String Formatting (Multiple Methods)|String Formatting (Multiple Methods)]]
- [[#Advanced String Operations|Advanced String Operations]]

### **Core Data Structures 📊**
- [[#Lists 📋|Lists Overview]]
  - [[#List Creation (Multiple Methods)|List Creation (Multiple Methods)]]
  - [[#List Indexing and Slicing|List Indexing & Slicing]]
  - [[#List Modification Operations|List Modification Operations]]
  - [[#List Searching and Information|List Searching & Information]]
  - [[#List Sorting and Reversing|List Sorting & Reversing]]
  - [[#List Copying and Combining|List Copying & Combining]]
  - [[#Advanced List Operations|Advanced List Operations]]

- [[#Tuples 📦|Tuples]]

- [[#Dictionaries 🔑|Dictionaries Overview]]
  - [[#Dictionary Creation (Multiple Methods)|Dictionary Creation (Multiple Methods)]]
  - [[#Dictionary Access and Modification|Dictionary Access & Modification]]
  - [[#Dictionary Information and Iteration|Dictionary Information & Iteration]]
  - [[#Advanced Dictionary Operations|Advanced Dictionary Operations]]

- [[#Sets 🔢|Sets Overview]]
  - [[#Set Creation (Multiple Methods)|Set Creation (Multiple Methods)]]
  - [[#Set Modification Operations|Set Modification Operations]]
  - [[#Set Information and Testing|Set Information & Testing]]
  - [[#Set Operations (Mathematical)|Set Operations (Mathematical)]]
  - [[#Advanced Set Operations|Advanced Set Operations]]

### **Functions & Advanced Concepts 🧩**
- [[#Functions 🧩|Functions Overview]]
- [[#Function Definition and Parameters|Function Definition & Parameters]]
- [[#Lambda Functions and Higher-Order Functions|Lambda Functions & Higher-Order Functions]]
- [[#Function Return Patterns|Function Return Patterns]]
- [[#Decorators and Advanced Function Concepts|Decorators & Advanced Function Concepts]]
- [[#Scope and Closures|Scope & Closures]]

### **File I/O & Data Handling 📁**
- [[#File I/O and Data Persistence 📁|File I/O Overview]]
- [[#Basic File Operations|Basic File Operations]]
- [[#Advanced File Operations|Advanced File Operations]]
- [[#Structured Data Formats|Structured Data Formats]]
- [[#Configuration and Serialization|Configuration & Serialization]]

### **Error Handling ⚠️**
- [[#Exception Handling ⚠️|Exception Handling Overview]]
- [[#Basic Exception Handling|Basic Exception Handling]]
- [[#Advanced Exception Handling|Advanced Exception Handling]]
- [[#Error Logging and Debugging|Error Logging & Debugging]]

### **Advanced Data Structures 🏗️**
- [[#Advanced Data Structures 🏗️|Advanced Data Structures Overview]]
- [[#Stack Implementation and Operations|Stack Implementation & Operations]]
- [[#Queue Implementation and Operations|Queue Implementation & Operations]]
- [[#Linked List Implementation|Linked List Implementation]]
- [[#Binary Tree Implementation and Algorithms|Binary Tree Implementation & Algorithms]]
- [[#Graph Representations and Algorithms|Graph Representations & Algorithms]]

### **Algorithms & Patterns 🧠**
- [[#Common DS&A Patterns 🧠|Common DS&A Patterns]]
- [[#Array/List Traversal|Array/List Traversal]]
- [[#Recursion and Backtracking|Recursion & Backtracking]]
- [[#Sorting and Searching|Sorting & Searching]]
- [[#Dynamic Programming Patterns 💡|Dynamic Programming Patterns]]
- [[#Advanced Techniques 🚀|Advanced Techniques]]
- [[#Bit Manipulation|Bit Manipulation]]
- [[#Union-Find (Disjoint Set)|Union-Find (Disjoint Set)]]
- [[#Trie (Prefix Tree)|Trie (Prefix Tree)]]
- [[#Segment Tree|Segment Tree]]

### **Performance & Optimization ⏱️**
- [[#Time and Space Complexity Reference ⏱️|Time & Space Complexity Reference]]
- [[#Common Patterns and Tricks 🎯|Common Patterns & Tricks]]
- [[#Useful Built-in Functions and Libraries 🛠️|Useful Built-in Functions & Libraries]]

### **Testing & Debugging 🐛**
- [[#Testing and Debugging 🐛|Testing & Debugging]]

### **Resources & References 📚**
- [[#Further Reading 📚|Further Reading]]

---

## Quick Reference Shortcuts 🔍

### **Most Common Operations**
- **String Reversal**: `s[::-1]` → [[#String Reversal (Multiple Methods)|See all methods]]
- **List Sorting**: `list.sort()` vs `sorted(list)` → [[#List Sorting and Reversing|Details]]
- **Dictionary Merging**: `dict1 | dict2` → [[#Advanced Dictionary Operations|All merge methods]]
- **Set Operations**: `set1 & set2`, `set1 | set2` → [[#Set Operations (Mathematical)|Mathematical operations]]
- **Two Pointers**: → [[#Array/List Traversal|Two-pointer technique]]
- **Binary Search**: → [[#Sorting and Searching|Binary search implementation]]

### **Data Structure Quick Access**
- **Stack**: `list.append()`, `list.pop()` → [[#Stack Implementation and Operations|Full implementation]]
- **Queue**: `deque.append()`, `deque.popleft()` → [[#Queue Implementation and Operations|Queue operations]]
- **Priority Queue**: `heapq` → [[#Queue Implementation and Operations|Priority queue section]]
- **Graph**: Adjacency list/matrix → [[#Graph Representations and Algorithms|Graph implementations]]

### **Algorithm Patterns**
- **Sliding Window**: → [[#Array/List Traversal|Sliding window patterns]]
- **DFS/BFS**: → [[#Graph Representations and Algorithms|Graph traversal]]
- **Dynamic Programming**: → [[#Dynamic Programming Patterns 💡|DP patterns]]
- **Backtracking**: → [[#Recursion and Backtracking|Backtracking examples]]

### **Performance Lookups**
- **Time Complexity**: → [[#Time and Space Complexity Reference ⏱️|Big O reference]]
- **Built-in Functions**: → [[#Useful Built-in Functions and Libraries 🛠️|Function performance]]
- **Optimization Tricks**: → [[#Common Patterns and Tricks 🎯|Performance tricks]]

---

*Click any link above to jump directly to that section. Use Ctrl+Click (Cmd+Click on Mac) to open in a new pane.*

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

# Multiple assignment and swapping
a, b = 1, 2            # Multiple assignment
a, b = b, a            # Swap variables - O(1)
x = y = z = 0          # Initialize multiple variables to same value

# Conditional assignment
max_val = a if a > b else b  # Ternary operator - O(1)
result = value or default_value  # Use default if value is falsy - O(1)
```

## String Operations 🔤

### String Creation and Basic Operations
```python
# String creation
text = "Hello World"
empty = ""
multiline = """This is a
multiline string"""
raw_string = r"C:\Users\name"  # Raw string (no escape sequences)

# String concatenation (multiple ways)
s1, s2 = "Hello", "World"
concat1 = s1 + " " + s2        # "Hello World" - O(n+m)
concat2 = f"{s1} {s2}"         # "Hello World" - O(n+m) 
concat3 = "{} {}".format(s1, s2)  # "Hello World" - O(n+m)
concat4 = " ".join([s1, s2])   # "Hello World" - O(n+m)

# String repetition
repeated = "abc" * 3           # "abcabcabc" - O(n*k)
dashes = "-" * 20              # "--------------------" - O(n)
```

### String Reversal (Multiple Methods)
```python
text = "hello"

# Method 1: Slicing (most common and efficient)
reversed1 = text[::-1]         # "olleh" - O(n)

# Method 2: Using reversed() and join
reversed2 = ''.join(reversed(text))  # "olleh" - O(n)

# Method 3: Manual reversal
def reverse_string_manual(s):  # O(n) time, O(n) space
    result = ""
    for char in s:
        result = char + result
    return result

# Method 4: Using list and reverse
def reverse_string_list(s):    # O(n) time, O(n) space
    chars = list(s)
    chars.reverse()
    return ''.join(chars)

# Method 5: Recursive approach
def reverse_string_recursive(s):  # O(n) time, O(n) space
    if len(s) <= 1:
        return s
    return s[-1] + reverse_string_recursive(s[:-1])
```

### String Slicing and Indexing
```python
text = "Hello World"

# Basic indexing
first = text[0]         # 'H' - O(1)
last = text[-1]         # 'd' - O(1)
second_last = text[-2]  # 'l' - O(1)

# Slicing [start:end:step]
substring = text[1:5]   # "ello" - O(k) where k is slice length
first_five = text[:5]   # "Hello" - O(k)
last_five = text[-5:]   # "World" - O(k)
every_second = text[::2]  # "HloWrd" - O(k)
reverse = text[::-1]    # "dlroW olleH" - O(n)

# Advanced slicing
text[1:-1]              # "ello Worl" (exclude first and last)
text[::3]               # "HlWl" (every 3rd character)
text[1::2]              # "el ol" (start at index 1, every 2nd)
text[-3::-1]            # "lroW olleH" (from 3rd last, reverse)
```

### String Searching and Finding
```python
text = "Hello World Hello"

# Finding substrings
index1 = text.find('World')      # 6 (first occurrence, -1 if not found) - O(n)
index2 = text.rfind('Hello')     # 12 (last occurrence) - O(n)
index3 = text.index('World')     # 6 (raises ValueError if not found) - O(n)

# Check if substring exists
exists = 'World' in text         # True - O(n)
not_exists = 'Python' not in text  # True - O(n)

# Count occurrences
count = text.count('l')          # 3 - O(n)
count_substr = text.count('Hello')  # 2 - O(n)

# String starts/ends with
starts = text.startswith('Hello')   # True - O(k) where k is prefix length
ends = text.endswith('Hello')       # True - O(k) where k is suffix length
starts_tuple = text.startswith(('Hi', 'Hello'))  # True (multiple options)
```

### String Modification and Cleaning
```python
text = "  Hello World  "

# Case conversion
lower = text.lower()            # "  hello world  " - O(n)
upper = text.upper()            # "  HELLO WORLD  " - O(n)
title = text.title()            # "  Hello World  " - O(n)
capitalize = text.capitalize()  # "  hello world  " - O(n)
swapcase = text.swapcase()      # "  hELLO wORLD  " - O(n)

# Whitespace removal
stripped = text.strip()         # "Hello World" (both ends) - O(n)
left_strip = text.lstrip()      # "Hello World  " (left end) - O(n)
right_strip = text.rstrip()     # "  Hello World" (right end) - O(n)
strip_chars = "abc123abc".strip('abc')  # "123" (specific chars) - O(n)

# String replacement
replaced = text.replace('World', 'Python')  # "  Hello Python  " - O(n)
replaced_count = text.replace('l', 'L', 2)  # Replace first 2 occurrences - O(n)

# Remove characters
no_spaces = text.replace(' ', '')  # "HelloWorld" - O(n)
no_vowels = ''.join(c for c in text if c.lower() not in 'aeiou')  # O(n)
```

### String Splitting and Joining
```python
text = "apple,banana,cherry"
sentence = "Hello world python"

# Splitting strings
words1 = sentence.split()       # ['Hello', 'world', 'python'] (default: whitespace) - O(n)
words2 = sentence.split(' ')    # ['Hello', 'world', 'python'] - O(n)
fruits = text.split(',')        # ['apple', 'banana', 'cherry'] - O(n)
limited = text.split(',', 1)    # ['apple', 'banana,cherry'] (max splits) - O(n)

# Split from right
rsplit = text.rsplit(',', 1)    # ['apple,banana', 'cherry'] - O(n)

# Split lines
multiline = "line1\nline2\nline3"
lines = multiline.splitlines()  # ['line1', 'line2', 'line3'] - O(n)
lines_keep = multiline.splitlines(True)  # ['line1\n', 'line2\n', 'line3'] - O(n)

# Partition (split into exactly 3 parts)
before, sep, after = text.partition(',')  # ('apple', ',', 'banana,cherry') - O(n)
rbefore, rsep, rafter = text.rpartition(',')  # ('apple,banana', ',', 'cherry') - O(n)

# Joining strings
joined1 = ','.join(fruits)      # "apple,banana,cherry" - O(n)
joined2 = ' '.join(['a', 'b'])  # "a b" - O(n)
joined3 = ''.join(['h', 'i'])   # "hi" - O(n)
path = '/'.join(['home', 'user', 'docs'])  # "home/user/docs" - O(n)
```

### String Validation and Testing
```python
# Character type checking
"123".isdigit()         # True (all digits) - O(n)
"abc".isalpha()         # True (all letters) - O(n)
"abc123".isalnum()      # True (letters and digits) - O(n)
"   ".isspace()         # True (all whitespace) - O(n)
"Hello World".istitle() # True (title case) - O(n)
"HELLO".isupper()       # True (all uppercase) - O(n)
"hello".islower()       # True (all lowercase) - O(n)

# String content validation
"123.45".replace('.', '').isdigit()  # Check if valid number
email = "test@example.com"
has_at = '@' in email and '.' in email  # Basic email check

# Check if string is palindrome
def is_palindrome(s):   # O(n) time, O(1) space
    return s == s[::-1]

# Check if string contains only specific characters
def contains_only(s, allowed_chars):  # O(n) time
    return all(c in allowed_chars for c in s)
```

### String Formatting (Multiple Methods)
```python
name, age, score = "Alice", 25, 95.5

# f-strings (Python 3.6+) - Recommended
formatted1 = f"Name: {name}, Age: {age}, Score: {score:.1f}"

# .format() method
formatted2 = "Name: {}, Age: {}, Score: {:.1f}".format(name, age, score)
formatted3 = "Name: {0}, Age: {1}, Score: {2:.1f}".format(name, age, score)
formatted4 = "Name: {n}, Age: {a}, Score: {s:.1f}".format(n=name, a=age, s=score)

# % formatting (older style)
formatted5 = "Name: %s, Age: %d, Score: %.1f" % (name, age, score)

# Advanced f-string formatting
number = 1234567.89
formatted_number = f"{number:,.2f}"     # "1,234,567.89" (thousands separator)
percentage = 0.1234
formatted_percent = f"{percentage:.2%}" # "12.34%"
binary = f"{42:b}"                      # "101010" (binary)
hex_val = f"{255:x}"                    # "ff" (hexadecimal)
padded = f"{42:05d}"                    # "00042" (zero-padded)
centered = f"{'hello':^10}"             # "  hello   " (centered in 10 chars)
```

### Advanced String Operations
```python
# String translation and mapping
text = "hello world"
translation_table = str.maketrans('aeiou', '12345')  # Create translation table
translated = text.translate(translation_table)  # "h2ll4 w4rld" - O(n)

# Remove characters using translation
remove_vowels = str.maketrans('', '', 'aeiou')
no_vowels = text.translate(remove_vowels)  # "hll wrld" - O(n)

# String encoding/decoding
encoded = text.encode('utf-8')          # Convert to bytes - O(n)
decoded = encoded.decode('utf-8')       # Convert back to string - O(n)

# String comparison (case-insensitive)
def case_insensitive_compare(s1, s2):  # O(n)
    return s1.lower() == s2.lower()

# Longest common prefix
def longest_common_prefix(strings):     # O(n*m) where n=num strings, m=avg length
    if not strings:
        return ""
    
    prefix = strings[0]
    for string in strings[1:]:
        while not string.startswith(prefix):
            prefix = prefix[:-1]
            if not prefix:
                return ""
    return prefix

# String distance (Levenshtein)
def edit_distance(s1, s2):             # O(n*m) time, O(n*m) space
    m, n = len(s1), len(s2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(m + 1):
        dp[i][0] = i
    for j in range(n + 1):
        dp[0][j] = j
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
    
    return dp[m][n]
```

## Character Operations 🔤

```python
# ASCII/Unicode operations
char = 'A'
print(ord(char))        # 65 (ASCII/Unicode value) - O(1)
print(chr(65))          # 'A' (character from ASCII/Unicode) - O(1)

# Character ranges
lowercase_letters = [chr(i) for i in range(ord('a'), ord('z') + 1)]  # ['a', 'b', ..., 'z']
uppercase_letters = [chr(i) for i in range(ord('A'), ord('Z') + 1)]  # ['A', 'B', ..., 'Z']
digits = [chr(i) for i in range(ord('0'), ord('9') + 1)]             # ['0', '1', ..., '9']

# Character classification
import string
string.ascii_lowercase  # 'abcdefghijklmnopqrstuvwxyz'
string.ascii_uppercase  # 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
string.ascii_letters    # 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
string.digits          # '0123456789'
string.punctuation     # '!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~'

# Character conversion
def char_to_number(c):  # Convert 'a'-'z' to 0-25 - O(1)
    return ord(c.lower()) - ord('a')

def number_to_char(n):  # Convert 0-25 to 'a'-'z' - O(1)
    return chr(n + ord('a'))

# Caesar cipher
def caesar_cipher(text, shift):  # O(n)
    result = ""
    for char in text:
        if char.isalpha():
            base = ord('A') if char.isupper() else ord('a')
            shifted = (ord(char) - base + shift) % 26
            result += chr(shifted + base)
        else:
            result += char
    return result
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

# Multiple conditions
if 0 < x < 10:          # Chained comparison
    print("Single digit positive")

if x in [1, 2, 3, 4, 5]:  # Check membership
    print("In range")

# Loops with different patterns
for i in range(5):      # 0, 1, 2, 3, 4  - O(n)
    print(i)

for i in range(1, 6):   # 1, 2, 3, 4, 5 - O(n)
    print(i)

for i in range(0, 10, 2):  # 0, 2, 4, 6, 8 (step=2) - O(n)
    print(i)

for i in range(10, 0, -1):  # 10, 9, 8, ..., 1 (reverse) - O(n)
    print(i)

# Enumerate for index and value
for index, value in enumerate(['a', 'b', 'c']):  # O(n)
    print(f"{index}: {value}")  # 0: a, 1: b, 2: c

for index, value in enumerate(['a', 'b', 'c'], start=1):  # Start from 1
    print(f"{index}: {value}")  # 1: a, 2: b, 3: c

# Zip for parallel iteration
names = ['Alice', 'Bob', 'Charlie']
ages = [25, 30, 35]
scores = [85, 90, 95]

for name, age in zip(names, ages):  # O(min(len(names), len(ages)))
    print(f"{name}: {age}")

for name, age, score in zip(names, ages, scores):  # Multiple iterables
    print(f"{name}: {age} years old, score: {score}")

# Loop control
for i in range(10):
    if i == 3:
        continue        # Skip to next iteration
    if i == 7:
        break          # Exit loop
    print(i)

# Else clause with loops (executes if loop completes without break)
for i in range(5):
    if i == 10:  # This won't be found
        break
else:
    print("Loop completed normally")  # This will execute

# While loop patterns
while x > 0:
    x -= 1              # Decrement x

# Infinite loop with break condition
while True:
    user_input = input("Enter 'quit' to exit: ")
    if user_input == 'quit':
        break
    print(f"You entered: {user_input}")
```

## Lists 📋

### List Creation (Multiple Methods)
```python
# Basic creation
nums = [1, 2, 3, 4, 5]
mixed = [1, "hello", True, 3.14]
empty = []

# Repetition and initialization
zeros = [0] * 5         # [0, 0, 0, 0, 0] - O(n)
ones = [1] * 3          # [1, 1, 1] - O(n)
matrix = [[0] * 3 for _ in range(2)]  # [[0, 0, 0], [0, 0, 0]] - O(n*m)

# From other iterables
from_string = list("hello")     # ['h', 'e', 'l', 'l', 'o'] - O(n)
from_range = list(range(5))     # [0, 1, 2, 3, 4] - O(n)
from_tuple = list((1, 2, 3))    # [1, 2, 3] - O(n)

# List comprehensions (multiple patterns)
squares = [x**2 for x in range(5)]      # [0, 1, 4, 9, 16] - O(n)
evens = [x for x in range(10) if x % 2 == 0]  # [0, 2, 4, 6, 8] - O(n)
words = ['hello', 'world', 'python']
lengths = [len(word) for word in words]  # [5, 5, 6] - O(n)
uppercase = [word.upper() for word in words]  # ['HELLO', 'WORLD', 'PYTHON'] - O(n)

# Nested list comprehensions
matrix = [[i+j for j in range(3)] for i in range(2)]  # [[0, 1, 2], [1, 2, 3]] - O(n*m)
flattened = [item for sublist in [[1, 2], [3, 4]] for item in sublist]  # [1, 2, 3, 4] - O(n)

# Conditional list comprehensions
positive_nums = [x for x in [-2, -1, 0, 1, 2] if x > 0]  # [1, 2] - O(n)
processed = [x if x > 0 else 0 for x in [-1, 2, -3, 4]]  # [0, 2, 0, 4] - O(n)
```

### List Indexing and Slicing
```python
nums = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# Basic indexing
first = nums[0]         # 0 - O(1)
last = nums[-1]         # 9 - O(1)
second_last = nums[-2]  # 8 - O(1)

# Slicing [start:end:step]
subset = nums[1:4]      # [1, 2, 3] (start:end, end exclusive) - O(k)
first_three = nums[:3]  # [0, 1, 2] - O(k)
last_three = nums[-3:]  # [7, 8, 9] - O(k)
every_second = nums[::2]  # [0, 2, 4, 6, 8] (step=2) - O(k)
every_third = nums[1::3]  # [1, 4, 7] (start at 1, step=3) - O(k)

# Reverse slicing
reversed_list = nums[::-1]      # [9, 8, 7, 6, 5, 4, 3, 2, 1, 0] - O(n)
reverse_subset = nums[7:2:-1]   # [7, 6, 5, 4, 3] - O(k)
last_five_reversed = nums[-1:-6:-1]  # [9, 8, 7, 6, 5] - O(k)

# Advanced slicing
middle = nums[2:-2]     # [2, 3, 4, 5, 6, 7] (exclude first 2 and last 2) - O(k)
skip_first_last = nums[1:-1]  # [1, 2, 3, 4, 5, 6, 7, 8] - O(k)
```

### List Modification Operations
```python
nums = [1, 2, 3, 4, 5]

# Adding elements
nums.append(6)          # Add to end: [1, 2, 3, 4, 5, 6] - O(1)
nums.extend([7, 8])     # Add multiple elements: [1, 2, 3, 4, 5, 6, 7, 8] - O(k)
nums += [9, 10]         # Alternative to extend - O(k)
nums.insert(0, 0)       # Insert at index: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10] - O(n)

# Removing elements
last = nums.pop()       # Remove and return last element - O(1)
first = nums.pop(0)     # Remove and return element at index 0 - O(n)
nums.remove(3)          # Remove first occurrence of value - O(n)
del nums[2]             # Delete element at index - O(n)
del nums[1:3]           # Delete slice - O(n)
nums.clear()            # Remove all elements - O(n)

# Replacing elements
nums = [1, 2, 3, 4, 5]
nums[0] = 10            # Replace single element - O(1)
nums[1:3] = [20, 30]    # Replace slice - O(n)
nums[::2] = [100, 200, 300]  # Replace every 2nd element - O(n)
```

### List Searching and Information
```python
nums = [1, 2, 3, 2, 4, 2, 5]

# Finding elements
index = nums.index(2)           # 1 (first occurrence) - O(n)
index_from = nums.index(2, 2)   # 3 (first occurrence after index 2) - O(n)
count = nums.count(2)           # 3 (count occurrences) - O(n)

# Check existence
exists = 2 in nums              # True - O(n)
not_exists = 10 not in nums     # True - O(n)

# List properties
length = len(nums)              # 7 - O(1)
is_empty = len(nums) == 0       # False - O(1)
max_val = max(nums)             # 5 - O(n)
min_val = min(nums)             # 1 - O(n)
total = sum(nums)               # 19 - O(n)

# Find index of max/min
max_index = nums.index(max(nums))  # O(n)
min_index = nums.index(min(nums))  # O(n)

# All/any checks
all_positive = all(x > 0 for x in nums)    # True - O(n)
has_even = any(x % 2 == 0 for x in nums)  # True - O(n)
```

### List Sorting and Reversing
```python
nums = [3, 1, 4, 1, 5, 9, 2, 6]

# Sorting (modifies original)
nums.sort()             # [1, 1, 2, 3, 4, 5, 6, 9] - O(n log n)
nums.sort(reverse=True) # [9, 6, 5, 4, 3, 2, 1, 1] - O(n log n)

# Sorting (returns new list)
sorted_nums = sorted(nums)              # Returns new sorted list - O(n log n)
reverse_sorted = sorted(nums, reverse=True)  # Descending order - O(n log n)

# Custom sorting
students = [('Alice', 85), ('Bob', 90), ('Charlie', 85)]
students.sort(key=lambda x: x[1])       # Sort by grade - O(n log n)
students.sort(key=lambda x: x[0])       # Sort by name - O(n log n)
students.sort(key=lambda x: (-x[1], x[0]))  # Sort by grade desc, then name asc - O(n log n)

# Multiple criteria sorting
from operator import itemgetter
students.sort(key=itemgetter(1, 0))     # Sort by grade, then name - O(n log n)

# Reversing
nums.reverse()          # Reverse in-place - O(n)
reversed_nums = nums[::-1]  # Create reversed copy - O(n)
reversed_iter = list(reversed(nums))  # Using reversed() function - O(n)
```

### List Copying and Combining
```python
original = [1, 2, 3, 4, 5]

# Shallow copying (multiple methods)
copy1 = original.copy()         # Method 1 - O(n)
copy2 = original[:]             # Method 2 - O(n)
copy3 = list(original)          # Method 3 - O(n)
copy4 = [x for x in original]   # Method 4 - O(n)

# Deep copying (for nested structures)
import copy
nested = [[1, 2], [3, 4]]
deep_copy = copy.deepcopy(nested)  # O(n)

# Combining lists
list1 = [1, 2, 3]
list2 = [4, 5, 6]
combined1 = list1 + list2       # [1, 2, 3, 4, 5, 6] - O(n+m)
combined2 = [*list1, *list2]    # [1, 2, 3, 4, 5, 6] (unpacking) - O(n+m)

# Interleaving lists
def interleave(list1, list2):   # O(min(n, m))
    return [item for pair in zip(list1, list2) for item in pair]

# Repeat list
repeated = list1 * 3            # [1, 2, 3, 1, 2, 3, 1, 2, 3] - O(n*k)
```

### Advanced List Operations
```python
# List filtering (multiple methods)
nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Method 1: List comprehension
evens1 = [x for x in nums if x % 2 == 0]  # [2, 4, 6, 8, 10] - O(n)

# Method 2: filter() function
evens2 = list(filter(lambda x: x % 2 == 0, nums))  # [2, 4, 6, 8, 10] - O(n)

# List transformation
squared = [x**2 for x in nums]          # List comprehension - O(n)
squared2 = list(map(lambda x: x**2, nums))  # Using map() - O(n)

# Remove duplicates (preserving order)
def remove_duplicates(lst):             # O(n) time, O(n) space
    seen = set()
    result = []
    for item in lst:
        if item not in seen:
            seen.add(item)
            result.append(item)
    return result

# Or using dict (Python 3.7+)
def remove_duplicates_dict(lst):        # O(n) time, O(n) space
    return list(dict.fromkeys(lst))

# Chunking list into smaller lists
def chunk_list(lst, chunk_size):        # O(n) time
    return [lst[i:i + chunk_size] for i in range(0, len(lst), chunk_size)]

# Flatten nested list
def flatten(nested_list):               # O(n) where n is total elements
    result = []
    for item in nested_list:
        if isinstance(item, list):
            result.extend(flatten(item))
        else:
            result.append(item)
    return result

# Rotate list
def rotate_list(lst, k):                # O(n) time, O(n) space
    if not lst:
        return lst
    k = k % len(lst)
    return lst[-k:] + lst[:-k]

# In-place rotation
def rotate_list_inplace(lst, k):        # O(n) time, O(1) space
    if not lst:
        return
    k = k % len(lst)
    lst[:] = lst[-k:] + lst[:-k]
```

## Tuples 📦

```python
# Creating tuples (immutable)
point = (3, 4)
single = (5,)           # Note the comma for single element
empty_tuple = ()
coordinates = 1, 2, 3   # Parentheses optional
mixed = (1, "hello", True)

# Tuple operations
length = len(point)     # 2 - O(1)
exists = 3 in point     # True - O(n)
index = point.index(3)  # 0 - O(n)
count = (1, 2, 1, 3).count(1)  # 2 - O(n)

# Tuple unpacking (multiple methods)
x, y = point           # x=3, y=4 - O(1)
first, *rest = (1, 2, 3, 4)  # first=1, rest=[2, 3, 4] - O(1)
*beginning, last = (1, 2, 3, 4)  # beginning=[1, 2, 3], last=4 - O(1)
first, *middle, last = (1, 2, 3, 4, 5)  # first=1, middle=[2, 3, 4], last=5 - O(1)

# Swapping using tuples
a, b = 10, 20
a, b = b, a            # Swap values - O(1)

# Multiple assignment
name, age, city = ("Alice", 25, "NYC")

# Tuple concatenation and repetition
tuple1 = (1, 2)
tuple2 = (3, 4)
combined = tuple1 + tuple2  # (1, 2, 3, 4) - O(n+m)
repeated = tuple1 * 3       # (1, 2, 1, 2, 1, 2) - O(n*k)

# Named tuples
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(3, 4)
print(p.x, p.y)        # 3 4 - O(1)
print(p[0], p[1])      # 3 4 (still accessible by index) - O(1)

# Named tuple methods
p_dict = p._asdict()   # {'x': 3, 'y': 4} - O(n)
p_new = p._replace(x=5)  # Point(x=5, y=4) - O(n)

# Converting between lists and tuples
lst = [1, 2, 3]
tup = tuple(lst)       # (1, 2, 3) - O(n)
back_to_list = list(tup)  # [1, 2, 3] - O(n)
```

## Dictionaries 🔑

### Dictionary Creation (Multiple Methods)
```python
# Basic creation
person = {"name": "Alice", "age": 25, "is_student": True}
empty_dict = {}

# From lists/tuples
keys = ['a', 'b', 'c']
values = [1, 2, 3]
dict_from_zip = dict(zip(keys, values))  # {'a': 1, 'b': 2, 'c': 3} - O(n)

# From list of tuples
pairs = [('x', 10), ('y', 20)]
dict_from_pairs = dict(pairs)  # {'x': 10, 'y': 20} - O(n)

# Using dict() constructor
dict_constructor = dict(name="Bob", age=30)  # {'name': 'Bob', 'age': 30}

# Dictionary comprehensions
squares = {x: x**2 for x in range(5)}  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16} - O(n)
filtered = {k: v for k, v in person.items() if isinstance(v, str)}  # O(n)
inverted = {v: k for k, v in squares.items()}  # Invert key-value pairs - O(n)

# From keys with default value
keys = ['a', 'b', 'c']
default_dict = dict.fromkeys(keys, 0)  # {'a': 0, 'b': 0, 'c': 0} - O(n)
```

### Dictionary Access and Modification
```python
person = {"name": "Alice", "age": 25}

# Accessing elements (multiple methods)
name1 = person["name"]              # Alice (raises KeyError if not found) - O(1) avg
name2 = person.get("name")          # Alice - O(1) avg
age = person.get("age", 0)          # 25 (returns 0 if key doesn't exist) - O(1) avg
height = person.setdefault("height", 170)  # Sets and returns default if key missing - O(1) avg

# Modifying dictionaries
person["age"] = 26                  # Update value - O(1) avg
person["city"] = "New York"         # Add new key-value pair - O(1) avg
person.update({"country": "USA", "age": 27})  # Update multiple - O(k)
person.update(zip(['job', 'salary'], ['Engineer', 75000]))  # Update from zip - O(k)

# Removing elements (multiple methods)
del person["is_student"]            # Remove key-value pair (raises KeyError if not found) - O(1) avg
removed = person.pop("city", "Unknown")  # Remove and return value - O(1) avg
key, value = person.popitem()       # Remove and return arbitrary key-value pair - O(1) avg

# Clear dictionary
person.clear()                      # Remove all elements - O(n)
```

### Dictionary Information and Iteration
```python
person = {"name": "Alice", "age": 25, "city": "NYC"}

# Dictionary information
length = len(person)                # 3 - O(1)
is_empty = len(person) == 0         # False - O(1)

# Check if key exists (multiple methods)
has_name1 = "name" in person        # True - O(1) avg
has_name2 = "name" in person.keys() # True - O(1) avg
has_height = "height" not in person # True - O(1) avg

# Get all keys, values, items
keys = list(person.keys())          # ['name', 'age', 'city'] - O(n)
values = list(person.values())      # ['Alice', 25, 'NYC'] - O(n)
items = list(person.items())        # [('name', 'Alice'), ('age', 25), ('city', 'NYC')] - O(n)

# Iteration patterns
for key in person:                  # Iterate over keys - O(n)
    print(key, person[key])

for key in person.keys():           # Explicit key iteration - O(n)
    print(key)

for value in person.values():       # Iterate over values - O(n)
    print(value)

for key, value in person.items():   # Iterate over key-value pairs - O(n)
    print(f"{key}: {value}")

# Dictionary views (dynamic)
keys_view = person.keys()           # dict_keys object (updates with dict)
values_view = person.values()       # dict_values object
items_view = person.items()         # dict_items object
```

### Advanced Dictionary Operations
```python
# Merging dictionaries (multiple methods)
dict1 = {"a": 1, "b": 2}
dict2 = {"c": 3, "d": 4}
dict3 = {"b": 20, "e": 5}  # Note: 'b' conflicts with dict1

# Method 1: Union operator (Python 3.9+)
merged1 = dict1 | dict2             # {"a": 1, "b": 2, "c": 3, "d": 4} - O(n+m)
merged_conflict = dict1 | dict3     # {"a": 1, "b": 20, "e": 5} (dict3 wins) - O(n+m)

# Method 2: Unpacking
merged2 = {**dict1, **dict2}        # {"a": 1, "b": 2, "c": 3, "d": 4} - O(n+m)

# Method 3: update() method
merged3 = dict1.copy()
merged3.update(dict2)               # Modifies merged3 - O(m)

# Method 4: dict() constructor
merged4 = dict(dict1, **dict2)      # Only works if dict2 keys are strings

# Nested dictionary access
nested = {
    "user": {
        "profile": {
            "name": "Alice",
            "settings": {"theme": "dark"}
        }
    }
}

# Safe nested access
def safe_get(d, *keys):             # O(k) where k is number of keys
    for key in keys:
        if isinstance(d, dict) and key in d:
            d = d[key]
        else:
            return None
    return d

name = safe_get(nested, "user", "profile", "name")  # "Alice"
theme = safe_get(nested, "user", "profile", "settings", "theme")  # "dark"

# Dictionary filtering
original = {"a": 1, "b": 2, "c": 3, "d": 4}
filtered = {k: v for k, v in original.items() if v > 2}  # {"c": 3, "d": 4} - O(n)
key_filtered = {k: v for k, v in original.items() if k in ['a', 'c']}  # {"a": 1, "c": 3} - O(n)

# Dictionary sorting
sorted_by_key = dict(sorted(original.items()))  # Sort by key - O(n log n)
sorted_by_value = dict(sorted(original.items(), key=lambda x: x[1]))  # Sort by value - O(n log n)
sorted_by_value_desc = dict(sorted(original.items(), key=lambda x: x[1], reverse=True))  # O(n log n)

# Invert dictionary (swap keys and values)
inverted = {v: k for k, v in original.items()}  # {1: 'a', 2: 'b', 3: 'c', 4: 'd'} - O(n)

# Group by value
from collections import defaultdict
def group_by_value(d):              # O(n)
    groups = defaultdict(list)
    for k, v in d.items():
        groups[v].append(k)
    return dict(groups)

# Default dictionaries
from collections import defaultdict
dd_list = defaultdict(list)         # Default value is empty list
dd_int = defaultdict(int)           # Default value is 0
dd_set = defaultdict(set)           # Default value is empty set

# Counter (special dictionary for counting)
from collections import Counter
text = "hello world"
char_count = Counter(text)          # Counter({'l': 3, 'o': 2, 'h': 1, ...}) - O(n)
most_common = char_count.most_common(3)  # [('l', 3), ('o', 2), ('h', 1)] - O(n log n)

# Counter operations
counter1 = Counter([1, 2, 2, 3])    # Counter({2: 2, 1: 1, 3: 1})
counter2 = Counter([2, 3, 3, 4])    # Counter({3: 2, 2: 1, 4: 1})
combined = counter1 + counter2      # Counter({2: 3, 3: 3, 1: 1, 4: 1}) - O(n+m)
subtracted = counter1 - counter2    # Counter({1: 1, 2: 1}) - O(n+m)
intersection = counter1 & counter2  # Counter({2: 1, 3: 1}) - O(min(n,m))
union = counter1 | counter2         # Counter({2: 2, 3: 2, 1: 1, 4: 1}) - O(n+m)
```

## Sets 🔢

### Set Creation (Multiple Methods)
```python
# Basic creation
unique_nums = {1, 2, 3, 4, 5}
duplicate_nums = {1, 2, 2, 3, 4, 4, 5}  # Will store as {1, 2, 3, 4, 5}
empty_set = set()               # Note: {} creates an empty dictionary

# From other iterables
from_list = set([1, 2, 2, 3])   # {1, 2, 3} - O(n)
from_string = set("hello")      # {'h', 'e', 'l', 'o'} - O(n)
from_tuple = set((1, 2, 3))     # {1, 2, 3} - O(n)
from_range = set(range(5))      # {0, 1, 2, 3, 4} - O(n)

# Set comprehensions
squares = {x**2 for x in range(5)}  # {0, 1, 4, 9, 16} - O(n)
even_squares = {x**2 for x in range(10) if x % 2 == 0}  # {0, 4, 16, 36, 64} - O(n)
```

### Set Modification Operations
```python
unique_nums = {1, 2, 3, 4, 5}

# Adding elements
unique_nums.add(6)              # Add single element - O(1) avg, O(n) worst
unique_nums.update([7, 8, 9])   # Add multiple elements - O(k) avg
unique_nums.update({10, 11})    # Add from another set - O(k) avg
unique_nums |= {12, 13}         # Union update operator - O(k) avg

# Removing elements (multiple methods)
unique_nums.remove(1)           # Remove element (raises KeyError if not found) - O(1) avg
unique_nums.discard(10)         # Remove if present (no error if not found) - O(1) avg
popped = unique_nums.pop()      # Remove and return arbitrary element - O(1) avg
unique_nums.clear()             # Remove all elements - O(n)

# Copy set
original = {1, 2, 3}
copied = original.copy()        # Shallow copy - O(n)
copied2 = set(original)         # Alternative copy method - O(n)
```

### Set Information and Testing
```python
set1 = {1, 2, 3, 4, 5}
set2 = {3, 4, 5, 6, 7}

# Set information
length = len(set1)              # 5 - O(1)
is_empty = len(set1) == 0       # False - O(1)
exists = 3 in set1              # True - O(1) avg, O(n) worst
not_exists = 10 not in set1     # True - O(1) avg, O(n) worst

# Set relationships
is_subset = {1, 2} <= set1      # True (is subset?) - O(len(smaller_set))
is_proper_subset = {1, 2} < set1  # True (is proper subset?) - O(len(smaller_set))
is_superset = set1 >= {1, 2}    # True (is superset?) - O(len(smaller_set))
is_proper_superset = set1 > {1, 2}  # True (is proper superset?) - O(len(smaller_set))
is_disjoint = set1.isdisjoint(set2)  # False (no common elements?) - O(min(len(set1), len(set2)))

# Alternative methods for relationships
is_subset_method = {1, 2}.issubset(set1)    # True - O(len(smaller_set))
is_superset_method = set1.issuperset({1, 2})  # True - O(len(smaller_set))
```

### Set Operations (Mathematical)
```python
set1 = {1, 2, 3, 4}
set2 = {3, 4, 5, 6}
set3 = {5, 6, 7, 8}

# Union (elements in either set)
union1 = set1 | set2            # {1, 2, 3, 4, 5, 6} - O(len(set1) + len(set2))
union2 = set1.union(set2)       # {1, 2, 3, 4, 5, 6} - O(len(set1) + len(set2))
union_multiple = set1.union(set2, set3)  # Union of multiple sets - O(sum of all set lengths)

# Intersection (elements in both sets)
intersection1 = set1 & set2     # {3, 4} - O(min(len(set1), len(set2)))
intersection2 = set1.intersection(set2)  # {3, 4} - O(min(len(set1), len(set2)))
intersection_multiple = set1.intersection(set2, set3)  # Intersection of multiple sets

# Difference (elements in first set but not second)
difference1 = set1 - set2       # {1, 2} - O(len(set1))
difference2 = set1.difference(set2)  # {1, 2} - O(len(set1))

# Symmetric difference (elements in either set, but not both)
sym_diff1 = set1 ^ set2         # {1, 2, 5, 6} - O(len(set1) + len(set2))
sym_diff2 = set1.symmetric_difference(set2)  # {1, 2, 5, 6} - O(len(set1) + len(set2))

# Update operations (modify original set)
set1_copy = set1.copy()
set1_copy |= set2               # Union update - O(len(set2))
set1_copy &= set2               # Intersection update - O(len(set1))
set1_copy -= set2               # Difference update - O(len(set2))
set1_copy ^= set2               # Symmetric difference update - O(len(set1) + len(set2))

# Method versions of update operations
set1_copy = set1.copy()
set1_copy.intersection_update(set2)      # Keep only common elements
set1_copy.difference_update(set2)        # Remove elements that are in set2
set1_copy.symmetric_difference_update(set2)  # Keep only elements in one set or the other
```

### Advanced Set Operations
```python
# Remove duplicates from list while preserving order
def remove_duplicates_ordered(lst):     # O(n) time, O(n) space
    seen = set()
    result = []
    for item in lst:
        if item not in seen:
            seen.add(item)
            result.append(item)
    return result

# Find unique elements across multiple lists
def find_unique_across_lists(*lists):   # O(sum of list lengths)
    all_elements = set()
    for lst in lists:
        all_elements.update(lst)
    return all_elements

# Find common elements across multiple lists
def find_common_across_lists(*lists):   # O(sum of list lengths)
    if not lists:
        return set()
    common = set(lists[0])
    for lst in lists[1:]:
        common.intersection_update(lst)
    return common

# Set operations with conditions
numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
evens = {x for x in numbers if x % 2 == 0}      # {2, 4, 6, 8, 10}
odds = {x for x in numbers if x % 2 == 1}       # {1, 3, 5, 7, 9}
primes = {2, 3, 5, 7}
composite_evens = evens - primes                # {4, 6, 8, 10}

# Frozen sets (immutable sets)
frozen = frozenset([1, 2, 3, 4])       # Immutable set - O(n)
frozen_from_set = frozenset(set1)       # Convert set to frozenset - O(n)

# Frozen sets can be used as dictionary keys or set elements
nested_sets = {frozenset([1, 2]), frozenset([3, 4])}
set_dict = {frozenset([1, 2]): "first", frozenset([3, 4]): "second"}
```

## Functions 🧩

### Function Definition and Parameters
```python
# Basic function
def greet(name):
    return f"Hello, {name}!"

# Function with default parameters
def power(base, exponent=2):
    return base ** exponent     # O(log n) for integer exponents

# Multiple default parameters
def create_user(name, age=18, active=True, role="user"):
    return {"name": name, "age": age, "active": active, "role": role}

# Variable positional arguments (*args)
def sum_all(*args):             # *args collects positional arguments into tuple
    return sum(args)            # O(n)

def print_values(*values, separator=" "):  # Keyword-only argument after *args
    print(separator.join(map(str, values)))

# Variable keyword arguments (**kwargs)
def print_info(**kwargs):       # **kwargs collects keyword arguments into dict
    for key, value in kwargs.items():  # O(n)
        print(f"{key}: {value}")

# Combined parameters (order matters: positional, *args, keyword-only, **kwargs)
def complex_function(required, optional=None, *args, keyword_only, **kwargs):
    print(f"Required: {required}")
    print(f"Optional: {optional}")
    print(f"Args: {args}")
    print(f"Keyword only: {keyword_only}")
    print(f"Kwargs: {kwargs}")

# Function annotations (type hints)
def add_numbers(a: int, b: int) -> int:
    return a + b

def process_data(data: list[int], multiplier: float = 1.0) -> list[float]:
    return [x * multiplier for x in data]
```

### Lambda Functions and Higher-Order Functions
```python
# Lambda functions (anonymous functions)
square = lambda x: x**2         # O(1)
add = lambda x, y: x + y        # O(1)
is_even = lambda x: x % 2 == 0  # O(1)

# Lambda with multiple parameters
distance = lambda x1, y1, x2, y2: ((x2-x1)**2 + (y2-y1)**2)**0.5

# Higher-order functions
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# map() - apply function to each element
squared = list(map(square, numbers))        # [1, 4, 9, 16, 25, ...] - O(n)
squared_lambda = list(map(lambda x: x**2, numbers))  # Same result - O(n)
string_nums = list(map(str, numbers))       # ['1', '2', '3', ...] - O(n)

# filter() - keep elements that satisfy condition
evens = list(filter(is_even, numbers))      # [2, 4, 6, 8, 10] - O(n)
evens_lambda = list(filter(lambda x: x % 2 == 0, numbers))  # Same result - O(n)
positives = list(filter(lambda x: x > 0, [-2, -1, 0, 1, 2]))  # [1, 2] - O(n)

# reduce() - apply function cumulatively
from functools import reduce
product = reduce(lambda x, y: x * y, numbers)  # 3628800 (10!) - O(n)
maximum = reduce(lambda x, y: x if x > y else y, numbers)  # 10 - O(n)
concatenated = reduce(lambda x, y: x + y, ['a', 'b', 'c', 'd'])  # 'abcd' - O(n)

# any() and all() with custom conditions
has_even = any(x % 2 == 0 for x in numbers)    # True - O(n) worst case
all_positive = all(x > 0 for x in numbers)     # True - O(n) worst case
```

### Function Return Patterns
```python
# Multiple return values (returns a tuple)
def min_max(numbers):
    return min(numbers), max(numbers)  # O(n) each, so O(n) combined

def divide_with_remainder(dividend, divisor):
    quotient = dividend // divisor
    remainder = dividend % divisor
    return quotient, remainder

# Unpacking return values
minimum, maximum = min_max([1, 2, 3, 4, 5])
q, r = divide_with_remainder(17, 5)  # q=3, r=2

# Conditional returns
def safe_divide(a, b):
    if b == 0:
        return None  # or raise an exception
    return a / b

# Early returns for efficiency
def find_first_even(numbers):
    for num in numbers:
        if num % 2 == 0:
            return num  # Early return when found
    return None  # Not found

# Generator functions (yield instead of return)
def fibonacci_generator(n):     # O(1) space per call
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

# Using generator
fib_nums = list(fibonacci_generator(10))  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

### Decorators and Advanced Function Concepts
```python
# Basic decorator
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
    return "Done"

# Decorator with parameters
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def say_hello():
    print("Hello!")

# Memoization decorator
def memoize(func):
    cache = {}
    def wrapper(*args):
        if args in cache:
            return cache[args]
        result = func(*args)
        cache[args] = result
        return result
    return wrapper

@memoize
def fibonacci(n):               # O(n) with memoization instead of O(2^n)
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Using functools for better decorators
from functools import wraps, lru_cache

def better_timer(func):
    @wraps(func)  # Preserves original function metadata
    def wrapper(*args, **kwargs):
        import time
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

# Built-in LRU cache decorator
@lru_cache(maxsize=None)
def fibonacci_cached(n):        # O(n) time with caching, O(n) space
    if n <= 1:
        return n
    return fibonacci_cached(n-1) + fibonacci_cached(n-2)

# Partial functions
from functools import partial

def multiply(x, y):
    return x * y

double = partial(multiply, 2)   # Fix first argument to 2
triple = partial(multiply, 3)   # Fix first argument to 3

print(double(5))  # 10
print(triple(4))  # 12

# Function composition
def compose(f, g):
    return lambda x: f(g(x))

add_one = lambda x: x + 1
square = lambda x: x ** 2
add_one_then_square = compose(square, add_one)
print(add_one_then_square(3))  # 16 (3+1=4, 4^2=16)
```

### Scope and Closures
```python
# Global vs local scope
global_var = "I'm global"

def scope_example():
    local_var = "I'm local"
    print(global_var)   # Can access global
    print(local_var)    # Can access local

# Modifying global variables
counter = 0

def increment():
    global counter
    counter += 1

# Nonlocal scope (nested functions)
def outer_function(x):
    def inner_function(y):
        nonlocal x  # Modify variable in enclosing scope
        x += y
        return x
    return inner_function

add_to_five = outer_function(5)
print(add_to_five(3))  # 8
print(add_to_five(2))  # 10 (x retains its value)

# Closures - functions that capture variables from enclosing scope
def make_multiplier(factor):
    def multiplier(number):
        return number * factor  # 'factor' is captured from outer scope
    return multiplier

times_two = make_multiplier(2)
times_three = make_multiplier(3)

print(times_two(5))    # 10
print(times_three(4))  # 12

# Factory functions using closures
def make_counter(start=0):
    count = start
    def counter():
        nonlocal count
        count += 1
        return count
    return counter

counter1 = make_counter()
counter2 = make_counter(10)

print(counter1())  # 1
print(counter1())  # 2
print(counter2())  # 11
print(counter2())  # 12
```

## File I/O and Data Persistence 📁

### Basic File Operations
```python
# Reading files (multiple methods)
# Method 1: Read entire file
with open('file.txt', 'r') as f:
    content = f.read()          # Read entire file as string - O(n)

# Method 2: Read all lines
with open('file.txt', 'r') as f:
    lines = f.readlines()       # Read all lines into list - O(n)

# Method 3: Read line by line (memory efficient)
with open('file.txt', 'r') as f:
    for line in f:              # Read line by line - O(1) per line
        print(line.strip())     # Remove trailing newline

# Method 4: Read specific number of characters
with open('file.txt', 'r') as f:
    chunk = f.read(100)         # Read first 100 characters - O(k)

# Method 5: Read one line at a time
with open('file.txt', 'r') as f:
    first_line = f.readline()   # Read first line - O(k) where k is line length
    second_line = f.readline()  # Read second line

# Writing files (multiple methods)
# Method 1: Write string
with open('output.txt', 'w') as f:
    f.write("Hello, World!")    # Write string - O(n)

# Method 2: Write multiple lines
lines = ['Line 1\n', 'Line 2\n', 'Line 3\n']
with open('output.txt', 'w') as f:
    f.writelines(lines)         # Write list of strings - O(n)

# Method 3: Write with print function
with open('output.txt', 'w') as f:
    print("Hello", "World", file=f)  # Use```python
# Method 3: Write with print function
with open('output.txt', 'w') as f:
    print("Hello", "World", file=f)  # Use print to write to file - O(n)
    print("Second line", file=f)

# File modes
# 'r' - read (default)
# 'w' - write (overwrites existing file)
# 'a' - append
# 'x' - exclusive creation (fails if file exists)
# 'b' - binary mode (e.g., 'rb', 'wb')
# 't' - text mode (default)
# '+' - read and write (e.g., 'r+', 'w+')

# Append to file
with open('log.txt', 'a') as f:
    f.write("New log entry\n")  # Add to end of file - O(k)

# Binary file operations
with open('image.jpg', 'rb') as f:
    binary_data = f.read()      # Read binary data - O(n)

with open('copy.jpg', 'wb') as f:
    f.write(binary_data)        # Write binary data - O(n)

# File operations without context manager (not recommended)
f = open('file.txt', 'r')
content = f.read()
f.close()  # Must manually close

# Check if file exists
import os
if os.path.exists('file.txt'):
    print("File exists")

# Get file information
import os
file_size = os.path.getsize('file.txt')  # File size in bytes
file_modified = os.path.getmtime('file.txt')  # Last modified time
```

### Advanced File Operations
```python
import os
import shutil
from pathlib import Path

# Using pathlib (modern approach)
file_path = Path('data/file.txt')
if file_path.exists():
    content = file_path.read_text()     # Read entire file - O(n)
    lines = file_path.read_text().splitlines()  # Read lines without \n - O(n)

# Write using pathlib
file_path.write_text("Hello, World!")  # Write string to file - O(n)

# File operations
file_path.mkdir(parents=True, exist_ok=True)  # Create directory
file_path.rename('new_name.txt')        # Rename file
file_path.unlink()                      # Delete file

# Directory operations
for file in Path('.').iterdir():        # List files in directory
    if file.is_file():
        print(f"File: {file.name}")
    elif file.is_dir():
        print(f"Directory: {file.name}")

# Recursive file search
for txt_file in Path('.').rglob('*.txt'):  # Find all .txt files recursively
    print(txt_file)

# Copy and move files
shutil.copy2('source.txt', 'destination.txt')  # Copy file with metadata
shutil.move('old_location.txt', 'new_location.txt')  # Move file
shutil.copytree('source_dir', 'dest_dir')  # Copy entire directory tree

# Temporary files
import tempfile

# Temporary file
with tempfile.NamedTemporaryFile(mode='w', delete=False) as temp_file:
    temp_file.write("Temporary data")
    temp_name = temp_file.name

# Temporary directory
with tempfile.TemporaryDirectory() as temp_dir:
    temp_path = Path(temp_dir) / 'temp_file.txt'
    temp_path.write_text("Temporary file in temporary directory")
```

### Structured Data Formats
```python
# JSON handling
import json

# Writing JSON
data = {
    "name": "Alice",
    "age": 25,
    "hobbies": ["reading", "coding"],
    "address": {"city": "NYC", "zip": "10001"}
}

# Write to file
with open('data.json', 'w') as f:
    json.dump(data, f, indent=2)    # Pretty print with indentation - O(n)

# Write to string
json_string = json.dumps(data, indent=2)  # Convert to JSON string - O(n)

# Reading JSON
with open('data.json', 'r') as f:
    loaded_data = json.load(f)      # Read JSON from file - O(n)

# Read from string
parsed_data = json.loads(json_string)  # Parse JSON string - O(n)

# Handle JSON errors
try:
    with open('invalid.json', 'r') as f:
        data = json.load(f)
except json.JSONDecodeError as e:
    print(f"Invalid JSON: {e}")
except FileNotFoundError:
    print("File not found")

# CSV handling
import csv

# Writing CSV
data = [
    ['Name', 'Age', 'City'],
    ['Alice', 25, 'NYC'],
    ['Bob', 30, 'LA'],
    ['Charlie', 35, 'Chicago']
]

with open('data.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerows(data)          # Write all rows - O(n*m)

# Write CSV with custom delimiter
with open('data.tsv', 'w', newline='') as f:
    writer = csv.writer(f, delimiter='\t')
    writer.writerow(['Name', 'Age'])    # Write single row - O(m)
    writer.writerow(['Alice', 25])

# Reading CSV
with open('data.csv', 'r') as f:
    reader = csv.reader(f)
    header = next(reader)           # Read header row
    for row in reader:              # Read data rows - O(1) per row
        name, age, city = row
        print(f"{name} is {age} years old and lives in {city}")

# CSV with DictReader/DictWriter
with open('data.csv', 'r') as f:
    reader = csv.DictReader(f)      # Each row is a dictionary
    for row in reader:
        print(f"{row['Name']} is {row['Age']} years old")

with open('output.csv', 'w', newline='') as f:
    fieldnames = ['Name', 'Age', 'City']
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()            # Write header row
    writer.writerow({'Name': 'Alice', 'Age': 25, 'City': 'NYC'})

# Handle CSV with different formats
with open('messy.csv', 'r') as f:
    reader = csv.reader(f, delimiter=';', quotechar='"')
    for row in reader:
        print(row)
```

### Configuration and Serialization
```python
# ConfigParser for .ini files
import configparser

# Create config
config = configparser.ConfigParser()
config['DEFAULT'] = {
    'debug': 'False',
    'log_level': 'INFO'
}
config['database'] = {
    'host': 'localhost',
    'port': '5432',
    'name': 'mydb'
}

# Write config file
with open('config.ini', 'w') as f:
    config.write(f)

# Read config file
config = configparser.ConfigParser()
config.read('config.ini')
db_host = config['database']['host']    # 'localhost'
debug = config.getboolean('DEFAULT', 'debug')  # False

# Pickle for Python object serialization
import pickle

# Serialize Python objects
data = {'list': [1, 2, 3], 'dict': {'a': 1}, 'tuple': (1, 2)}
with open('data.pickle', 'wb') as f:
    pickle.dump(data, f)            # Serialize to file - O(n)

# Deserialize Python objects
with open('data.pickle', 'rb') as f:
    loaded_data = pickle.load(f)    # Deserialize from file - O(n)

# Pickle to/from bytes
pickled_bytes = pickle.dumps(data)  # Serialize to bytes - O(n)
unpickled_data = pickle.loads(pickled_bytes)  # Deserialize from bytes - O(n)

# XML handling (basic)
import xml.etree.ElementTree as ET

# Create XML
root = ET.Element("catalog")
book = ET.SubElement(root, "book", id="1")
title = ET.SubElement(book, "title")
title.text = "Python Programming"
author = ET.SubElement(book, "author")
author.text = "John Doe"

# Write XML
tree = ET.ElementTree(root)
tree.write("catalog.xml", encoding="utf-8", xml_declaration=True)

# Read XML
tree = ET.parse("catalog.xml")
root = tree.getroot()
for book in root.findall("book"):
    title = book.find("title").text
    author = book.find("author").text
    print(f"Title: {title}, Author: {author}")
```

## Exception Handling ⚠️

### Basic Exception Handling
```python
# Basic try-except
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero!")

# Multiple specific exceptions
try:
    value = int(input("Enter a number: "))
    result = 10 / value
    items = [1, 2, 3]
    print(items[value])
except ValueError:
    print("Invalid number format!")
except ZeroDivisionError:
    print("Cannot divide by zero!")
except IndexError:
    print("Index out of range!")

# Catch multiple exceptions in one block
try:
    # Some risky operation
    pass
except (ValueError, TypeError) as e:
    print(f"Value or type error: {e}")

# Catch all exceptions (use sparingly)
try:
    # Some operation
    pass
except Exception as e:
    print(f"An error occurred: {e}")
    print(f"Error type: {type(e).__name__}")

# Finally block (always executes)
try:
    file = open('data.txt', 'r')
    data = file.read()
except FileNotFoundError:
    print("File not found!")
finally:
    if 'file' in locals() and not file.closed:
        file.close()    # Always executed, even if exception occurs

# Else block (executes only if no exception)
try:
    result = 10 / 2
except ZeroDivisionError:
    print("Division by zero!")
else:
    print(f"Result: {result}")  # Only runs if no exception
finally:
    print("Cleanup code")       # Always runs
```

### Advanced Exception Handling
```python
# Raising exceptions
def validate_age(age):
    if not isinstance(age, int):
        raise TypeError("Age must be an integer")
    if age < 0:
        raise ValueError("Age cannot be negative")
    if age > 150:
        raise ValueError("Age seems unrealistic")
    return age

# Re-raising exceptions
def process_data(data):
    try:
        # Some processing
        result = risky_operation(data)
    except ValueError as e:
        print(f"Logging error: {e}")
        raise  # Re-raise the same exception

# Custom exceptions
class CustomError(Exception):
    """Base class for custom exceptions"""
    pass

class ValidationError(CustomError):
    """Raised when validation fails"""
    def __init__(self, message, code=None):
        super().__init__(message)
        self.code = code

class NetworkError(CustomError):
    """Raised when network operation fails"""
    pass

# Using custom exceptions
def validate_email(email):
    if '@' not in email:
        raise ValidationError("Email must contain @", code="MISSING_AT")
    if '.' not in email.split('@')[1]:
        raise ValidationError("Email domain must contain .", code="INVALID_DOMAIN")
    return email

try:
    email = validate_email("invalid-email")
except ValidationError as e:
    print(f"Validation failed: {e}")
    if hasattr(e, 'code'):
        print(f"Error code: {e.code}")

# Exception chaining
def outer_function():
    try:
        inner_function()
    except ValueError as e:
        raise RuntimeError("Outer function failed") from e

def inner_function():
    raise ValueError("Inner function error")

try:
    outer_function()
except RuntimeError as e:
    print(f"Main error: {e}")
    print(f"Caused by: {e.__cause__}")

# Context managers for resource management
class FileManager:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = None
    
    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file
    
    def __exit__(self, exc_type, exc_value, traceback):
        if self.file:
            self.file.close()
        # Return False to propagate exceptions
        return False

# Using custom context manager
with FileManager('data.txt', 'r') as f:
    content = f.read()

# Contextlib for simpler context managers
from contextlib import contextmanager

@contextmanager
def timer_context():
    import time
    start = time.time()
    try:
        yield
    finally:
        end = time.time()
        print(f"Operation took {end - start:.4f} seconds")

with timer_context():
    # Some time-consuming operation
    sum(range(1000000))

# Exception handling in loops
def process_items(items):
    results = []
    errors = []
    
    for item in items:
        try:
            result = risky_process(item)
            results.append(result)
        except Exception as e:
            errors.append((item, str(e)))
            continue  # Continue with next item
    
    return results, errors

# Suppress specific exceptions
from contextlib import suppress

# Instead of try-except for simple cases
with suppress(FileNotFoundError):
    os.remove('nonexistent_file.txt')

# Multiple suppressed exceptions
with suppress(KeyError, AttributeError):
    value = data['key'].attribute
```

### Error Logging and Debugging
```python
import logging
import traceback

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('app.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger(__name__)

# Exception logging
def safe_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError as e:
        logger.error(f"Division by zero: {a} / {b}", exc_info=True)
        return None
    except Exception as e:
        logger.exception(f"Unexpected error in division: {a} / {b}")
        return None

# Get detailed traceback information
def detailed_error_handler():
    try:
        # Some operation that might fail
        risky_operation()
    except Exception as e:
        # Get full traceback as string
        tb_str = traceback.format_exc()
        logger.error(f"Detailed error:\n{tb_str}")
        
        # Get traceback object for analysis
        tb = traceback.extract_tb(e.__traceback__)
        for frame in tb:
            logger.debug(f"File: {frame.filename}, Line: {frame.lineno}, Function: {frame.name}")

# Assert statements for debugging
def calculate_average(numbers):
    assert len(numbers) > 0, "Cannot calculate average of empty list"
    assert all(isinstance(n, (int, float)) for n in numbers), "All items must be numbers"
    return sum(numbers) / len(numbers)

# Debugging with pdb
import pdb

def debug_function(data):
    pdb.set_trace()  # Debugger breakpoint
    processed = process_complex_data(data)
    return processed

# Conditional debugging
DEBUG = True

def conditional_debug(data):
    if DEBUG:
        pdb.set_trace()
    return process_data(data)
```

## Advanced Data Structures 🏗️

### Stack Implementation and Operations
```python
# Stack using list (most common)
class Stack:
    def __init__(self):
        self.items = []
    
    def push(self, item):           # O(1) amortized
        self.items.append(item)
    
    def pop(self):                  # O(1)
        if self.is_empty():
            raise IndexError("Stack is empty")
        return self.items.pop()
    
    def peek(self):                 # O(1)
        if self.is_empty():
            raise IndexError("Stack is empty")
        return self.items[-1]
    
    def is_empty(self):             # O(1)
        return len(self.items) == 0
    
    def size(self):                 # O(1)
        return len(self.items)

# Simple stack operations using list
stack = []
stack.append(1)     # Push - O(1)
stack.append(2)
stack.append(3)
top = stack[-1] if stack else None     # Peek - O(1)
item = stack.pop() if stack else None  # Pop - O(1)

# Stack applications
def is_valid_parentheses(s):    # O(n) time, O(n) space
    stack = []
    mapping = {')': '(', '}': '{', ']': '['}
    
    for char in s:
        if char in mapping:
            if not stack or stack.pop() != mapping[char]:
                return False
        else:
            stack.append(char)
    
    return len(stack) == 0

def evaluate_postfix(expression):   # O(n) time, O(n) space
    stack = []
    operators = {'+', '-', '*', '/'}
    
    for token in expression.split():
        if token in operators:
            b = stack.pop()
            a = stack.pop()
            if token == '+':
                result = a + b
            elif token == '-':
                result = a - b
            elif token == '*':
                result = a * b
            elif token == '/':
                result = a / b
            stack.append(result)
        else:
            stack.append(float(token))
    
    return stack[0]

# Convert infix to postfix
def infix_to_postfix(expression):   # O(n) time, O(n) space
    precedence = {'+': 1, '-': 1, '*': 2, '/': 2, '^': 3}
    stack = []
    result = []
    
    for char in expression:
        if char.isalnum():
            result.append(char)
        elif char == '(':
            stack.append(char)
        elif char == ')':
            while stack and stack[-1] != '(':
                result.append(stack.pop())
            stack.pop()  # Remove '('
        elif char in precedence:
            while (stack and stack[-1] != '(' and
                   stack[-1] in precedence and
                   precedence[stack[-1]] >= precedence[char]):
                result.append(stack.pop())
            stack.append(char)
    
    while stack:
        result.append(stack.pop())
    
    return ''.join(result)
```

### Queue Implementation and Operations
```python
from collections import deque

# Queue using deque (recommended)
class Queue:
    def __init__(self):
        self.items = deque()
    
    def enqueue(self, item):        # O(1)
        self.items.append(item)
    
    def dequeue(self):              # O(1)
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self.items.popleft()
    
    def front(self):                # O(1)
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self.items[0]
    
    def is_empty(self):             # O(1)
        return len(self.items) == 0
    
    def size(self):                 # O(1)
        return len(self.items)

# Simple queue operations using deque
queue = deque()
queue.append(1)     # Enqueue - O(1)
queue.append(2)
queue.append(3)
front = queue[0] if queue else None    # Peek front - O(1)
item = queue.popleft() if queue else None  # Dequeue - O(1)

# Priority Queue using heapq
import heapq

class PriorityQueue:
    def __init__(self):
        self.heap = []
        self.index = 0  # For stable ordering
    
    def push(self, priority, item):     # O(log n)
        heapq.heappush(self.heap, (priority, self.index, item))
        self.index += 1
    
    def pop(self):                      # O(log n)
        if self.is_empty():
            raise IndexError("Priority queue is empty")
        return heapq.heappop(self.heap)[2]  # Return item only
    
    def peek(self):                     # O(1)
        if self.is_empty():
            raise IndexError("Priority queue is empty")
        return self.heap[0][2]
    
    def is_empty(self):                 # O(1)
        return len(self.heap) == 0

# Simple priority queue operations
min_heap = []
heapq.heappush(min_heap, (3, 'task3'))     # O(log n)
heapq.heappush(min_heap, (1, 'task1'))
heapq.heappush(min_heap, (2, 'task2'))
priority, task = heapq.heappop(min_heap)   # Returns (1, 'task1') - O(log n)

# Max heap (negate priorities)
max_heap = []
heapq.heappush(max_heap, (-3, 'task3'))
heapq.heappush(max_heap, (-1, 'task1'))
neg_priority, task = heapq.heappop(max_heap)  # Returns (-3, 'task3')
actual_priority = -neg_priority  # 3

# Queue applications
def bfs_tree(root):                 # O(n) time, O(w) space where w is max width
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

def level_order_traversal(root):    # O(n) time, O(w) space
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

# Circular queue
class CircularQueue:
    def __init__(self, capacity):
        self.capacity = capacity
        self.queue = [None] * capacity
        self.front = 0
        self.rear = 0
        self.size = 0
    
    def enqueue(self, item):            # O(1)
        if self.size == self.capacity:
            raise OverflowError("Queue is full")
        
        self.queue[self.rear] = item
        self.rear = (self.rear + 1) % self.capacity
        self.size += 1
    
    def dequeue(self):                  # O(1)
        if self.size == 0:
            raise IndexError("Queue is empty")
        
        item = self.queue[self.front]
        self.queue[self.front] = None
        self.front = (self.front + 1) % self.capacity
        self.size -= 1
        return item
    
    def is_full(self):                  # O(1)
        return self.size == self.capacity
    
    def is_empty(self):                 # O(1)
        return self.size == 0
```

### Linked List Implementation
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
    
    def __repr__(self):
        return f"ListNode({self.val})"

class LinkedList:
    def __init__(self):
        self.head = None
        self.size = 0
    
    def append(self, val):              # O(n) time
        new_node = ListNode(val)
        if not self.head:
            self.head = new_node
        else:
            current = self.head
            while current.next:
                current = current.next
            current.next = new_node
        self.size += 1
    
    def prepend(self, val):             # O(1) time
        new_node = ListNode(val)
        new_node.next = self.head
        self.head = new_node
        self.size += 1
    
    def insert(self, index, val):       # O(n) time
        if index < 0 or index > self.size:
            raise IndexError("Index out of range")
        
        if index == 0:
            self.prepend(val)
            return
        
        new_node = ListNode(val)
        current = self.head
        for _ in range(index - 1):
            current = current.next
        
        new_node.next = current.next
        current.next = new_node
        self.size += 1
    
    def delete(self, val):              # O(n) time
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
    
    def delete_at_index(self, index):   # O(n) time
        if index < 0 or index >= self.size:
            raise IndexError("Index out of range")
        
        if index == 0:
            self.head = self.head.next
            self.size -= 1
            return
        
        current = self.head
        for _ in range(index - 1):
            current = current.next
        
        current.next = current.next.next
        self.size -= 1
    
    def find(self, val):                # O(n) time
        current = self.head
        index = 0
        while current:
            if current.val == val:
                return index
            current = current.next
            index += 1
        return -1
    
    def get(self, index):               # O(n) time
        if index < 0 or index >= self.size:
            raise IndexError("Index out of range")
        
        current = self.head
        for _ in range(index):
            current = current.next
        return current.val
    
    def to_list(self):                  # O(n) time
        result = []
        current = self.head
        while current:
            result.append(current.val)
            current = current.next
        return result
    
    def __len__(self):                  # O(1) time
        return self.size
    
    def __str__(self):                  # O(n) time
        return " -> ".join(map(str, self.to_list()))

# Doubly Linked List
class DoublyListNode:
    def __init__(self, val=0, prev=None, next=None):
        self.val = val
        self.prev = prev
        self.next = next

class DoublyLinkedList:
    def __init__(self):
        self.head = None
        self.tail = None
        self.size = 0
    
    def append(self, val):              # O(1) time
        new_node = DoublyListNode(val)
        if not self.head:
            self.head = self.tail = new_node
        else:
            new_node.prev = self.tail
            self.tail.next = new_node
            self.tail = new_node
        self.size += 1
    
    def prepend(self, val):             # O(1) time
        new_node = DoublyListNode(val)
        if not self.head:
            self.head = self.tail = new_node
        else:
            new_node.next = self.head
            self.head.prev = new_node
            self.head = new_node
        self.size += 1
    
    def delete(self, val):              # O(n) time
        current = self.head
        while current:
            if current.val == val:
                if current.prev:
                    current.prev.next = current.next
                else:
                    self.head = current.next
                
                if current.next:
                    current.next.prev = current.prev
                else:
                    self.tail = current.prev
                
                self.size -= 1
                return True
            current = current.next
        return False

# Common linked list algorithms
def reverse_linked_list(head):          # O(n) time, O(1) space
    prev = None
    current = head
    
    while current:
        next_temp = current.next
        current.next = prev
        prev = current
        current = next_temp
    
    return prev

def has_cycle(head):                    # O(n) time, O(1) space - Floyd's cycle detection
    if not head or not head.next:
        return False
    
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    
    return False

def find_cycle_start(head):             # O(n) time, O(1) space
    if not head or not head.next:
        return None
    
    # Find meeting point
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            break
    else:
        return None  # No cycle
    
    # Find start of cycle
    slow = head
    while slow != fast:
        slow = slow.next
        fast = fast.next
    
    return slow

def merge_sorted_lists(l1, l2):         # O(n+m) time, O(1) space
    dummy = ListNode(0)
    current = dummy
    
    while l1 and l2:
        if l1.val <= l2.val:
            current.next = l1
            l1 = l1.next
        else:
            current.next = l2
            l2 = l2.next
        current = current.next
    
    current.next = l1 or l2
    return dummy.next

def find_middle(head):                  # O(n) time, O(1) space
    if not head:
        return None
    
    slow = fast = head
    while fast.next and fast.next.next:
        slow = slow.next
        fast = fast.next.next
    
    return slow

def remove_nth_from_end(head, n):       # O(n) time, O(1) space
    dummy = ListNode(0)
    dummy.next = head
    first = second = dummy
    
    # Move first n+1 steps ahead
    for _ in range(n + 1):
        first = first.next
    
    # Move both until first reaches end
    while first:
        first = first.next
        second = second.next
    
    # Remove nth node from end
    second.next = second.next.next
    return dummy.next
```

### Binary Tree Implementation and Algorithms
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
    
    def __repr__(self):
        return f"TreeNode({self.val})"

class BinaryTree:
    def __init__(self, root=None):
        self.root = root
    
    def insert_level_order(self, values):   # O(n) time
        if not values:
            return
        
        self.root = TreeNode(values[0])
        queue = deque([self.root])
        i = 1
        
        while queue and i < len(values):
            node = queue.popleft()
            
            if i < len(values) and values[i] is not None:
                node.left = TreeNode(values[i])
                queue.append(node.left)
            i += 1
            
            if i < len(values) and values[i] is not None:
                node.right = TreeNode(values[i])
                queue.append(node.right)
            i += 1

# Tree traversals
def inorder_traversal(root):            # O(n) time, O(h) space
    result = []
    
    def inorder(node):
        if node:
            inorder(node.left)
            result.append(node.val)
            inorder(node.right)
    
    inorder(root)
    return result

def inorder_iterative(root):            # O(n) time, O(h) space
    result = []
    stack = []
    current = root
    
    while stack or current:
        while current:
            stack.append(current)
            current = current.left
        
        current = stack.pop()
        result.append(current.val)
        current = current.right
    
    return result

def preorder_traversal(root):           # O(n) time, O(h) space
    result = []
    
    def preorder(node):
        if node:
            result.append(node.val)
            preorder(node.left)
            preorder(node.right)
    
    preorder(root)
    return result

def postorder_traversal(root):          # O(n) time, O(h) space
    result = []
    
    def postorder(node):
        if node:
            postorder(node.left)
            postorder(node.right)
            result.append(node.val)
    
    postorder(root)
    return result

def level_order_traversal(root):        # O(n) time, O(w) space
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

# Tree properties and algorithms
def max_depth(root):                    # O(n) time, O(h) space
    if not root:
        return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))

def min_depth(root):                    # O(n) time, O(h) space
    if not root:
        return 0
    if not root.left and not root.right:
        return 1
    
    depths = []
    if root.left:
        depths.append(min_depth(root.left))
    if root.right:
        depths.append(min_depth(root.right))
    
    return 1 + min(depths)

def is_balanced(root):                  # O(n) time, O(h) space
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

def diameter_of_tree(root):             # O(n) time, O(h) space
    max_diameter = 0
    
    def depth(node):
        nonlocal max_diameter
        if not node:
            return 0
        
        left_depth = depth(node.left)
        right_depth = depth(node.right)
        
        max_diameter = max(max_diameter, left_depth + right_depth)
        return 1 + max(left_depth, right_depth)
    
    depth(root)
    return max_diameter

def lowest_common_ancestor(root, p, q): # O(n) time, O(h) space
    if not root or root == p or root == q:
        return root
    
    left = lowest_common_ancestor(root.left, p, q)
    right = lowest_common_ancestor(root.right, p, q)
    
    if left and right:
        return root
    return left or right

def path_sum(root, target_sum):         # O(n) time, O(h) space
    def has_path_sum(node, remaining):
        if not node:
            return False
        
        remaining -= node.val
        if not node.left and not node.right:
            return remaining == 0
        
        return (has_path_sum(node.left, remaining) or 
                has_path_sum(node.right, remaining))
    
    return has_path_sum(root, target_sum)

def all_paths_sum(root, target_sum):    # O(n) time, O(h) space
    result = []
    
    def find_paths(node, remaining, path):
        if not node:
            return
        
        path.append(node.val)
        remaining -= node.val
        
        if not node.left and not node.right and remaining == 0:
            result.append(path[:])  # Make a copy
        else:
            find_paths(node.left, remaining, path)
            find_paths(node.right, remaining, path)
        
        path.pop()  # Backtrack
    
    find_paths(root, target_sum, [])
    return result

# Binary Search Tree operations
def is_valid_bst(root):                 # O(n) time, O(h) space
    def validate(node, min_val, max_val):
        if not node:
            return True
        
        if node.val <= min_val or node.val >= max_val:
            return False
        
        return (validate(node.left, min_val, node.val) and
                validate(node.right, node.val, max_val))
    
    return validate(root, float('-inf'), float('inf'))

def search_bst(root, val):              # O(h) time, O(h) space
    if not root or root.val == val:
        return root
    
    if val < root.val:
        return search_bst(root.left, val)
    else:
        return search_bst(root.right, val)

def insert_bst(root, val):              # O(h) time, O(h) space
    if not root:
        return TreeNode(val)
    
    if val < root.val:
        root.left = insert_bst(root.left, val)
    else:
        root.right = insert_bst(root.right, val)
    
    return root

def delete_bst(root, val):              # O(h) time, O(h) space
    if not root:
        return root
    
    if val < root.val:
        root.left = delete_bst(root.left, val)
    elif val > root.val:
        root.right = delete_bst(root.right, val)
    else:
        # Node to delete found
        if not root.left:
            return root.right
        elif not root.right:
            return root.left
        
        # Node has two children
        min_node = find_min(root.right)
        root.val = min_node.val
        root.right = delete_bst(root.right, min_node.val)
    
    return root

def find_min(root):                     # O(h) time, O(1) space
    while root.left:
        root = root.left
    return root

def find_max(root):                     # O(h) time, O(1) space
    while root.right:
        root = root.right
    return root
```

### Graph Representations and Algorithms
```python
from collections import defaultdict, deque

# Graph representations
class Graph:
    def __init__(self, directed=False):
        self.graph = defaultdict(list)
        self.directed = directed
    
    def add_edge(self, u, v, weight=1):     # O(1) time
        self.graph[u].append((v, weight))
        if not self.directed:
            self.graph[v].append((u, weight))
    
    def add_vertex(self, vertex):           # O(1) time
        if vertex not in self.graph:
            self.graph[vertex] = []
    
    def get_vertices(self):                 # O(1) time
        return list(self.graph.keys())
    
    def get_edges(self):                    # O(V + E) time
        edges = []
        for vertex in self.graph:
            for neighbor, weight in self.graph[vertex]:
                if self.directed or vertex <= neighbor:
                    edges.append((vertex, neighbor, weight))
        return edges

# Create graph from edge list
def create_graph_from_edges(edges, directed=False):  # O(E) time
    graph = defaultdict(list)
    for u, v in edges:
        graph[u].append(v)
        if not directed:
            graph[v].append(u)
    return graph

# Adjacency matrix representation
class GraphMatrix:
    def __init__(self, num_vertices, directed=False):
        self.num_vertices = num_vertices
        self.directed = directed
        self.matrix = [[0] * num_vertices for _ in range(num_vertices)]
    
    def add_edge(self, u, v, weight=1):     # O(1) time
        self.matrix[u][v] = weight
        if not self.directed:
            self.matrix[v][u] = weight
    
    def has_edge(self, u, v):               # O(1) time
        return self.matrix[u][v] != 0
    
    def get_neighbors(self, vertex):        # O(V) time
        neighbors = []
        for i in range(self.num_vertices):
            if self.matrix[vertex][i] != 0:
                neighbors.append(i)
        return neighbors

# Graph traversal algorithms
def dfs_recursive(graph, start, visited=None):  # O(V + E) time, O(V) space
    if visited is None:
        visited = set()
    
    visited.add(start)
    result = [start]
    
    for neighbor in graph[start]:
        if neighbor not in visited:
            result.extend(dfs_recursive(graph, neighbor, visited))
    
    return result

def dfs_iterative(graph, start):            # O(V + E) time, O(V) space
    visited = set()
    stack = [start]
    result = []
    
    while stack:
        vertex = stack.pop()
        if vertex not in visited:
            visited.add(vertex)
            result.append(vertex)
            
            # Add neighbors in reverse order for consistent ordering
            for neighbor in reversed(graph[vertex]):
                if neighbor not in visited:
                    stack.append(neighbor)
    
    return result

def bfs(graph, start):                      # O(V + E) time, O(V) space
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

# Shortest path algorithms
def shortest_path_unweighted(graph, start, end):  # O(V + E) time, O(V) space
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

def dijkstra(graph, start):                 # O((V + E) log V) time with heap
    import heapq
    
    distances = {vertex: float('inf') for vertex in graph}
    distances[start] = 0
    previous = {}
    heap = [(0, start)]
    
    while heap:
        current_distance, current = heapq.heappop(heap)
        
        if current_distance > distances[current]:
            continue
        
        for neighbor, weight in graph[current]:
            distance = current_distance + weight
            
            if distance < distances[neighbor]:
                distances[neighbor] = distance
                previous[neighbor] = current
                heapq.heappush(heap, (distance, neighbor))
    
    return distances, previous

def reconstruct_path(previous, start, end):  # O(V) time
    path = []
    current = end
    
    while current is not None:
        path.append(current)
        current = previous.get(current)
    
    path.reverse()
    return path if path[0] == start else []

# Topological sorting
def topological_sort_dfs(graph):           # O(V + E) time, O(V) space
    visited = set()
    stack = []
    
    def dfs(vertex):
        visited.add(vertex)
        for neighbor in graph[vertex]:
            if neighbor not in visited:
                dfs(neighbor)
        stack.append(vertex)
    
    for vertex in graph:
        if vertex not in visited:
            dfs(vertex)
    
    return stack[::-1]

def topological_sort_kahn(graph):          # O(V + E) time, O(V) space
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

# Cycle detection
def has_cycle_directed(graph):              # O(V + E) time, O(V) space
    WHITE, GRAY, BLACK = 0, 1, 2
    color = defaultdict(int)
    
    def dfs(vertex):
        if color[vertex] == GRAY:
            return True  # Back edge found
        if color[vertex] == BLACK:
            return False
        
        color[vertex] = GRAY
        for neighbor in graph[vertex]:
            if dfs(neighbor):
                return True
        
        color[vertex] = BLACK
        return False
    
    for vertex in graph:
        if color[vertex] == WHITE:
            if dfs(vertex):
                return True
    
    return False

def has_cycle_undirected(graph):            # O(V + E) time, O(V) space
    visited = set()
    
    def dfs(vertex, parent):
        visited.add(vertex)
        for neighbor in graph[vertex]:
            if neighbor not in visited:
                if dfs(neighbor, vertex):
                    return True
            elif neighbor != parent:
                return True  # Back edge to non-parent
        return False
    
    for vertex in graph:
        if vertex not in visited:
            if dfs(vertex, None):
                return True
    
    return False

# Connected components
def find_connected_components(graph):       # O(V + E) time, O(V) space
    visited = set()
    components = []
    
    def dfs(vertex, component):
        visited.add(vertex)
        component.append(vertex)
        for neighbor in graph[vertex]:
            if neighbor not in visited:
                dfs(neighbor, component)
    
    for vertex in graph:
        if vertex not in visited:
            component = []
            dfs(vertex, component)
            components.append(component)
    
    return components

# Minimum Spanning Tree (Kruskal's algorithm)
def kruskal_mst(edges, num_vertices):       # O(E log E) time
    # edges: list of (weight, u, v)
    edges.sort()  # Sort by weight
    parent = list(range(num_vertices))
    rank = [0] * num_vertices
    mst = []
    
    def find(x):
        if parent[x] != x:
            parent[x] = find(parent[x])
        return parent[x]
    
    def union(x, y):
        px, py = find(x), find(y)
        if px == py:
            return False
        if rank[px] < rank[py]:
            px, py = py, px
        parent[py] = px
        if rank[px] == rank[py]:
            rank[px] += 1
        return True
    
    for weight, u, v in edges:
        if union(u, v):
            mst.append((weight, u, v))
            if len(mst) == num_vertices - 1:
                break
    
    return mst
```

