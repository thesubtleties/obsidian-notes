## Definition 📝
Strings are sequences of characters used to represent text. In most programming languages, strings are a primitive or built-in data type that store a series of characters (letters, numbers, symbols) enclosed within quotation marks.

Key characteristics:
- Immutable in many languages (Java, Python, JavaScript)
- Can be indexed like arrays (0-based indexing)
- Have built-in methods for manipulation and searching
- Can be empty or contain any number of characters (limited by memory)

## Complexity Analysis ⏱️
| Operation | Time Complexity | Space Complexity |
|-----------|-----------------|------------------|
| Access by index | O(1) | O(1) |
| Search for substring | O(n*m) naive, O(n+m) with KMP | O(1) naive, O(m) with KMP |
| Concatenation | O(n+m) | O(n+m) |
| Slice/Substring | O(k) where k is length of substring | O(k) |
| String comparison | O(min(n,m)) | O(1) |

*Note: n and m represent the lengths of strings being operated on*

## Core Operations 🔍
```
- charAt(index): Get character at specific position
- substring(start, end): Extract portion of string
- indexOf(substring): Find first occurrence of substring
- length(): Get string length
- concat(str2): Combine strings
- split(delimiter): Convert string to array
- replace(old, new): Replace occurrences of substring
- toLowerCase()/toUpperCase(): Change case
```

## String Manipulation Techniques 🛠️

### Basic Manipulation
```python
# Python examples
s = "Hello, World!"

# Length
length = len(s)  # 13

# Accessing characters
first_char = s[0]  # 'H'
last_char = s[-1]  # '!'

# Slicing
substring = s[0:5]  # "Hello"
without_first = s[1:]  # "ello, World!"
without_last = s[:-1]  # "Hello, World"

# Concatenation
new_string = s + " How are you?"  # "Hello, World! How are you?"

# Repetition
repeated = "abc" * 3  # "abcabcabc"

# Case conversion
lowercase = s.lower()  # "hello, world!"
uppercase = s.upper()  # "HELLO, WORLD!"
```

```javascript
// JavaScript examples
let s = "Hello, World!";

// Length
let length = s.length;  // 13

// Accessing characters
let firstChar = s[0];  // 'H'
let lastChar = s[s.length-1];  // '!'

// Slicing
let substring = s.substring(0, 5);  // "Hello"
let withoutFirst = s.substring(1);  // "ello, World!"

// Concatenation
let newString = s + " How are you?";  // "Hello, World! How are you?"
let template = `${s} How are you?`;   // "Hello, World! How are you?"

// Case conversion
let lowercase = s.toLowerCase();  // "hello, world!"
let uppercase = s.toUpperCase();  // "HELLO, WORLD!"
```

### Advanced Manipulation
```python
# Splitting and joining
words = "apple,banana,orange".split(",")  # ["apple", "banana", "orange"]
joined = "-".join(words)  # "apple-banana-orange"

# Replacing
replaced = "Hello World".replace("World", "Python")  # "Hello Python"

# Stripping whitespace
padded = "  text with spaces  "
stripped = padded.strip()  # "text with spaces"
left_stripped = padded.lstrip()  # "text with spaces  "
right_stripped = padded.rstrip()  # "  text with spaces"

# Checking content
starts_with = "Hello".startswith("He")  # True
ends_with = "Hello".endswith("lo")  # True
contains = "Hello" in "Hello World"  # True
```

## String Matching Algorithms 🔍

### Naive String Matching
The simplest approach is to check every position in the text for a match with the pattern.

```python
def naive_search(text, pattern):
    n, m = len(text), len(pattern)
    occurrences = []
    
    for i in range(n - m + 1):
        match = True
        for j in range(m):
            if text[i + j] != pattern[j]:
                match = False
                break
        if match:
            occurrences.append(i)
            
    return occurrences
```

**Time Complexity**: O(n*m) in worst case

### Knuth-Morris-Pratt (KMP) Algorithm
KMP improves efficiency by avoiding unnecessary comparisons using a prefix function.

```python
def kmp_search(text, pattern):
    if not pattern:
        return []
    
    # Build the prefix table
    prefix = [0] * len(pattern)
    j = 0
    for i in range(1, len(pattern)):
        while j > 0 and pattern[j] != pattern[i]:
            j = prefix[j-1]
        if pattern[j] == pattern[i]:
            j += 1
        prefix[i] = j
    
    # Search for pattern
    occurrences = []
    j = 0
    for i in range(len(text)):
        while j > 0 and pattern[j] != text[i]:
            j = prefix[j-1]
        if pattern[j] == text[i]:
            j += 1
        if j == len(pattern):
            occurrences.append(i - j + 1)
            j = prefix[j-1]
    
    return occurrences
```

**Time Complexity**: O(n+m)

## Palindromes 🔄

A palindrome is a string that reads the same backward as forward.

### Checking for Palindromes
```python
def is_palindrome(s):
    # Remove non-alphanumeric characters and convert to lowercase
    s = ''.join(c.lower() for c in s if c.isalnum())
    return s == s[::-1]

# Examples
print(is_palindrome("racecar"))  # True
print(is_palindrome("A man, a plan, a canal: Panama"))  # True
print(is_palindrome("hello"))  # False
```

### Finding Longest Palindromic Substring
```python
def longest_palindrome(s):
    if not s:
        return ""
    
    start = 0
    max_len = 1
    
    # Helper function to expand around center
    def expand_around_center(left, right):
        while left >= 0 and right < len(s) and s[left] == s[right]:
            left -= 1
            right += 1
        return right - left - 1
    
    for i in range(len(s)):
        # Odd length palindromes
        len1 = expand_around_center(i, i)
        # Even length palindromes
        len2 = expand_around_center(i, i + 1)
        
        # Update if we found a longer palindrome
        length = max(len1, len2)
        if length > max_len:
            max_len = length
            start = i - (length - 1) // 2
    
    return s[start:start + max_len]
```

## Anagrams 📝

Anagrams are strings that contain the same characters but in a different order.

### Checking if Two Strings are Anagrams
```python
def are_anagrams(s1, s2):
    # Remove spaces and convert to lowercase
    s1 = s1.replace(" ", "").lower()
    s2 = s2.replace(" ", "").lower()
    
    # Check if lengths are different
    if len(s1) != len(s2):
        return False
    
    # Count characters
    char_count = {}
    for char in s1:
        char_count[char] = char_count.get(char, 0) + 1
    
    for char in s2:
        if char not in char_count or char_count[char] == 0:
            return False
        char_count[char] -= 1
    
    return True

# Examples
print(are_anagrams("listen", "silent"))  # True
print(are_anagrams("triangle", "integral"))  # True
print(are_anagrams("hello", "world"))  # False
```

### Finding All Anagrams in a String
```python
def find_all_anagrams(s, p):
    if len(p) > len(s):
        return []
    
    p_count = {}
    s_count = {}
    
    # Initialize character counts for pattern
    for char in p:
        p_count[char] = p_count.get(char, 0) + 1
    
    # Initialize sliding window
    for i in range(len(p)):
        char = s[i]
        s_count[char] = s_count.get(char, 0) + 1
    
    result = []
    if p_count == s_count:
        result.append(0)
    
    # Slide the window
    for i in range(len(p), len(s)):
        # Add new character
        s_count[s[i]] = s_count.get(s[i], 0) + 1
        
        # Remove old character
        s_count[s[i - len(p)]] -= 1
        if s_count[s[i - len(p)]] == 0:
            del s_count[s[i - len(p)]]
        
        # Check if current window is an anagram
        if p_count == s_count:
            result.append(i - len(p) + 1)
    
    return result
```

## Common String Problems and Solutions 🧩

### Reverse a String
```python
# Using slicing
def reverse_string(s):
    return s[::-1]

# Using iteration
def reverse_string_manual(s):
    chars = list(s)
    left, right = 0, len(chars) - 1
    while left < right:
        chars[left], chars[right] = chars[right], chars[left]
        left += 1
        right -= 1
    return ''.join(chars)
```

### Count Character Occurrences
```python
def count_chars(s):
    char_count = {}
    for char in s:
        char_count[char] = char_count.get(char, 0) + 1
    return char_count
```

### Remove Duplicates
```python
def remove_duplicates(s):
    result = ""
    seen = set()
    for char in s:
        if char not in seen:
            seen.add(char)
            result += char
    return result
```

## Advantages and Disadvantages ⚖️

### Advantages ✅
- Universal data type across programming languages
- Rich set of built-in methods for manipulation
- Intuitive representation of text data
- Immutability in many languages prevents accidental modification
- Efficient for many text processing tasks

### Disadvantages ❌
- Immutability can lead to performance issues with frequent modifications
- String operations can be memory-intensive
- Character encoding issues (UTF-8, ASCII, etc.) can cause bugs
- String comparison and searching can be slow for large texts
- Different languages have different string handling methods

## Real-world Applications 🌐
- Text processing and natural language processing
- Web development (HTML parsing, URL handling)
- Data validation and sanitization
- File I/O operations
- Configuration parsing
- Regular expressions
- Cryptography and encryption
- Database queries

## Further Reading 📚
- Books:
  - "Algorithms on Strings, Trees, and Sequences" by Dan Gusfield
  - "Cracking the Coding Interview" by Gayle Laakmann McDowell (String chapters)
  
- Online Resources:
  - [LeetCode String Problems](https://leetcode.com/tag/string/)
  - [GeeksforGeeks String Algorithms](https://www.geeksforgeeks.org/string-data-structure/)
  - [HackerRank String Manipulation Challenges](https://www.hackerrank.com/domains/algorithms?filters%5Bsubdomains%5D%5B%5D=strings)
  
- Advanced Topics:
  - Suffix Trees and Arrays
  - Rabin-Karp Algorithm
  - Boyer-Moore Algorithm
  - Regular Expressions
  - Unicode and Character Encoding

---