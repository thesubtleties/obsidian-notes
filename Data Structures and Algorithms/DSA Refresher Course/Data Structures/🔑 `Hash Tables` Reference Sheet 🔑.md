## Definition 📝
A hash table (also called hash map) is a data structure that implements an associative array abstract data type, a structure that can map keys to values. It uses a hash function to compute an index into an array of buckets or slots, from which the desired value can be found. Ideally, the hash function will assign each key to a unique bucket, but most hash table designs employ an imperfect hash function, which might cause hash collisions where the hash function generates the same index for more than one key.

## Complexity Analysis ⏱️
| Operation | Average Case | Worst Case |
|-----------|--------------|------------|
| Insert    | O(1)         | O(n)       |
| Delete    | O(1)         | O(n)       |
| Search    | O(1)         | O(n)       |

The worst-case scenario occurs when there are many collisions, causing the hash table to degenerate into a linked list. However, with a good hash function and appropriate collision resolution, the average case remains O(1).

## Core Operations 🔍
- `insert(key, value)`: Add a new key-value pair to the hash table
- `delete(key)`: Remove the key-value pair with the given key
- `search(key)`: Find the value associated with the given key
- `update(key, value)`: Modify the value associated with an existing key
- `contains(key)`: Check if a key exists in the hash table
- `keys()`: Return all keys in the hash table
- `values()`: Return all values in the hash table

## Implementation 💻

### Basic Hash Table Implementation
```python
class HashTable:
    def __init__(self, size=10):
        self.size = size
        self.table = [None] * size
    
    def _hash(self, key):
        # Simple hash function for demonstration
        if isinstance(key, str):
            return sum(ord(c) for c in key) % self.size
        return key % self.size
    
    def insert(self, key, value):
        index = self._hash(key)
        if self.table[index] is None:
            self.table[index] = []
        
        # Check if key already exists and update
        for i, (k, v) in enumerate(self.table[index]):
            if k == key:
                self.table[index][i] = (key, value)
                return
        
        # Key doesn't exist, add new key-value pair
        self.table[index].append((key, value))
    
    def get(self, key):
        index = self._hash(key)
        if self.table[index] is None:
            return None
        
        for k, v in self.table[index]:
            if k == key:
                return v
        
        return None
    
    def delete(self, key):
        index = self._hash(key)
        if self.table[index] is None:
            return False
        
        for i, (k, v) in enumerate(self.table[index]):
            if k == key:
                del self.table[index][i]
                return True
        
        return False
```

## Dictionary Implementation 📚
Most programming languages provide built-in hash table implementations:

### Python Dictionary
```python
# Creating a dictionary
student = {
    "name": "John",
    "age": 21,
    "courses": ["Math", "CS"]
}

# Accessing values
print(student["name"])  # Output: John

# Adding/updating entries
student["gpa"] = 3.8
student["age"] = 22

# Checking if key exists
if "name" in student:
    print("Name exists")

# Iterating through keys and values
for key in student:
    print(f"{key}: {student[key]}")

# Alternative iteration methods
for key, value in student.items():
    print(f"{key}: {value}")
```

### JavaScript Object/Map
```javascript
// Using Object as a hash table
const student = {
    name: "John",
    age: 21,
    courses: ["Math", "CS"]
};

// Using Map (better for hash tables)
const studentMap = new Map();
studentMap.set("name", "John");
studentMap.set("age", 21);
studentMap.set("courses", ["Math", "CS"]);

// Accessing values
console.log(studentMap.get("name"));  // Output: John

// Checking if key exists
console.log(studentMap.has("name"));  // Output: true

// Iterating
studentMap.forEach((value, key) => {
    console.log(`${key}: ${value}`);
});
```

## Collision Handling 💥
When two keys hash to the same index, a collision occurs. There are several strategies to handle collisions:

### 1. Chaining (Using Linked Lists)
```
[0] → (key1, value1) → (key2, value2)
[1] → (key3, value3)
[2] → null
[3] → (key4, value4) → (key5, value5) → (key6, value6)
```

Each bucket contains a linked list of key-value pairs that hash to the same index.

### 2. Open Addressing
When a collision occurs, look for the next available slot in the array.

#### Linear Probing
```python
def insert(key, value):
    index = hash(key) % size
    while table[index] is not None:
        # If key already exists, update value
        if table[index][0] == key:
            table[index] = (key, value)
            return
        # Otherwise, try next slot
        index = (index + 1) % size
    table[index] = (key, value)
```

#### Quadratic Probing
```python
def insert(key, value):
    index = hash(key) % size
    i = 0
    while table[index] is not None:
        if table[index][0] == key:
            table[index] = (key, value)
            return
        i += 1
        index = (hash(key) + i*i) % size
    table[index] = (key, value)
```

#### Double Hashing
```python
def insert(key, value):
    index = hash1(key) % size
    i = 0
    while table[index] is not None:
        if table[index][0] == key:
            table[index] = (key, value)
            return
        i += 1
        index = (hash1(key) + i * hash2(key)) % size
    table[index] = (key, value)
```

## Common Dictionary Patterns 🔄

### Frequency Counter
```python
def count_characters(string):
    char_count = {}
    for char in string:
        if char in char_count:
            char_count[char] += 1
        else:
            char_count[char] = 1
    return char_count

# Example
print(count_characters("hello"))  # {'h': 1, 'e': 1, 'l': 2, 'o': 1}
```

### Two-Sum Problem
```python
def two_sum(nums, target):
    seen = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []

# Example
print(two_sum([2, 7, 11, 15], 9))  # [0, 1]
```

### Caching/Memoization
```python
def fibonacci_with_memo(n, memo={}):
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fibonacci_with_memo(n-1, memo) + fibonacci_with_memo(n-2, memo)
    return memo[n]
```

### Group/Categorize Items
```python
def group_anagrams(words):
    anagram_groups = {}
    for word in words:
        # Sort the word to use as a key
        sorted_word = ''.join(sorted(word))
        if sorted_word in anagram_groups:
            anagram_groups[sorted_word].append(word)
        else:
            anagram_groups[sorted_word] = [word]
    return list(anagram_groups.values())

# Example
print(group_anagrams(["eat", "tea", "tan", "ate", "nat", "bat"]))
# [['eat', 'tea', 'ate'], ['tan', 'nat'], ['bat']]
```

## Custom Hash Functions 🧮
A good hash function should:
1. Be deterministic (same input always yields same output)
2. Distribute keys uniformly across the hash table
3. Be efficient to compute
4. Minimize collisions

### String Hashing Examples
```python
# Simple string hash function
def simple_hash(s, table_size):
    hash_value = 0
    for char in s:
        hash_value += ord(char)
    return hash_value % table_size

# Polynomial rolling hash function (better distribution)
def polynomial_hash(s, table_size, p=31):
    hash_value = 0
    p_pow = 1
    for char in s:
        hash_value = (hash_value + (ord(char) - ord('a') + 1) * p_pow) % table_size
        p_pow = (p_pow * p) % table_size
    return hash_value
```

### Object Hashing
For custom objects, you often need to implement a hash function based on the object's properties:

```python
class Person:
    def __init__(self, name, age, id):
        self.name = name
        self.age = age
        self.id = id
    
    def __hash__(self):
        # Use a combination of all attributes
        return hash((self.name, self.age, self.id))
    
    def __eq__(self, other):
        # Required for hash table lookups
        if not isinstance(other, Person):
            return False
        return (self.name, self.age, self.id) == (other.name, other.age, other.id)
```

## Advantages and Disadvantages ⚖️

### Advantages ✅
- Fast lookups, insertions, and deletions (average O(1))
- Flexible keys (can use strings, numbers, tuples, or custom objects as keys)
- Memory efficient for storing key-value pairs
- Built-in implementation in most programming languages
- Ideal for caching and indexing

### Disadvantages ❌
- Unordered (elements are not stored in insertion order by default)
- Performance degrades with high collision rate
- Requires a good hash function
- Potentially high memory overhead for small hash tables
- Worst-case O(n) operations if many collisions occur

## Real-world Applications 🌐
- Database indexing
- Caching systems
- Symbol tables in compilers and interpreters
- Spell checkers
- Password verification
- Blockchain mining
- Implementing sets
- Web session storage
- Network routing tables

## Further Reading 📚
- Books:
  - "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein
  - "The Art of Computer Programming, Vol. 3: Sorting and Searching" by Donald Knuth
- Online Resources:
  - [Hash Tables on GeeksforGeeks](https://www.geeksforgeeks.org/hashing-data-structure/)
  - [Hash Table Visualization](https://visualgo.net/en/hashtable)
  - [Stanford CS Library: Hash Tables](http://infolab.stanford.edu/~ullman/focs/ch07.pdf)
- Academic Papers:
  - "Cuckoo Hashing" by Pagh and Rodler
  - "Hopscotch Hashing" by Herlihy, Shavit, and Tzafrir

---