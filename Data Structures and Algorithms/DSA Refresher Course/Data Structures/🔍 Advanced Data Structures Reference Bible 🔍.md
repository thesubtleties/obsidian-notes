This reference guide covers advanced data structures that extend beyond the basics. Each section provides a comprehensive overview with implementations, complexity analysis, and practical applications to help you master these powerful tools.

---

## 🔗 Disjoint Set / Union-Find Reference Sheet 🔗

### Definition 📝
A Disjoint-Set (Union-Find) is a data structure that keeps track of elements partitioned into non-overlapping (disjoint) subsets. It provides near-constant-time operations to:
- Add new sets
- Merge existing sets
- Determine whether elements are in the same set

This structure is particularly useful for tracking connected components in a network or graph.

### Complexity Analysis ⏱️
| Operation | Time Complexity (with optimizations) |
|-----------|--------------------------------------|
| MakeSet   | O(1)                                 |
| Find      | O(α(n)) ≈ O(1)                       |
| Union     | O(α(n)) ≈ O(1)                       |

Where α(n) is the inverse Ackermann function, which grows extremely slowly and is less than 5 for all practical values of n, making these operations effectively constant time.

### Core Operations 🔍
- **MakeSet(x)**: Creates a new set containing only element x
- **Find(x)**: Returns the representative (root) of the set containing x
- **Union(x, y)**: Merges the sets containing elements x and y

### Implementation 💻
```python
class DisjointSet:
    def __init__(self, n):
        # Each element starts as its own parent
        self.parent = list(range(n))
        # Rank helps keep the tree balanced
        self.rank = [0] * n
    
    def find(self, x):
        # Path compression: point all nodes along the path to the root
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        # Union by rank: attach smaller rank tree under root of higher rank tree
        root_x = self.find(x)
        root_y = self.find(y)
        
        if root_x == root_y:
            return
        
        if self.rank[root_x] < self.rank[root_y]:
            self.parent[root_x] = root_y
        else:
            self.parent[root_y] = root_x
            if self.rank[root_x] == self.rank[root_y]:
                self.rank[root_x] += 1
```

### Optimizations 🔄
1. **Path Compression**: During Find operations, make each node on the path point directly to the root
2. **Union by Rank/Size**: Attach the smaller tree to the root of the larger tree

### Common Examples 🚀
```python
# Example: Detecting cycles in an undirected graph
def has_cycle(edges, n):
    ds = DisjointSet(n)
    
    for u, v in edges:
        # If both vertices are already in the same set, we found a cycle
        if ds.find(u) == ds.find(v):
            return True
        ds.union(u, v)
    
    return False

# Example: Kruskal's Minimum Spanning Tree algorithm
def kruskal_mst(edges, n):
    ds = DisjointSet(n)
    edges.sort(key=lambda x: x[2])  # Sort edges by weight
    mst = []
    
    for u, v, weight in edges:
        if ds.find(u) != ds.find(v):
            ds.union(u, v)
            mst.append((u, v, weight))
    
    return mst
```

### Visualization 📊
```
Initial state (each element in its own set):
[0] [1] [2] [3] [4]

After Union(0,1) and Union(2,3):
[0,1] [2,3] [4]

After Union(0,2):
[0,1,2,3] [4]

Find(3) returns the representative of the set (0 in this case)
```

### Advantages and Disadvantages ⚖️
#### Advantages ✅
- Near-constant time operations with optimizations
- Simple implementation
- Excellent for graph connectivity problems
- Efficient for Kruskal's algorithm and cycle detection

#### Disadvantages ❌
- Cannot easily delete elements or split sets
- Not suitable for dynamic collections where elements are frequently added/removed
- Limited to set operations (can't store additional data with elements)

### Real-world Applications 🌐
- Network connectivity
- Image processing (connected component labeling)
- Kruskal's minimum spanning tree algorithm
- Detecting cycles in graphs
- Maze generation algorithms
- Percolation theory simulations

### Further Reading 📚
- "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (Chapter 21)
- [Princeton University - Union-Find](https://algs4.cs.princeton.edu/15uf/)
- [Visualgo - Disjoint Set Visualization](https://visualgo.net/en/ufds)
- [CP-Algorithms - Disjoint Set Union](https://cp-algorithms.com/data_structures/disjoint_set_union.html)

---

## 🌲 Fenwick Tree (Binary Indexed Tree) Reference Sheet 🌲

### Definition 📝
A Fenwick Tree (Binary Indexed Tree or BIT) is a data structure that efficiently updates elements and calculates prefix sums in a table of numbers. It requires less space and is easier to implement than a segment tree, while still providing logarithmic time operations.

The key insight is using a clever binary representation to track cumulative frequencies or sums.

### Complexity Analysis ⏱️
| Operation       | Time Complexity |
|-----------------|-----------------|
| Construction    | O(n)            |
| Update          | O(log n)        |
| Prefix Sum      | O(log n)        |
| Range Sum       | O(log n)        |

### Core Operations 🔍
- **Update(i, val)**: Add val to the element at index i
- **GetSum(i)**: Get the sum of all elements from index 0 to i (inclusive)
- **RangeSum(i, j)**: Get the sum of all elements from index i to j (inclusive)

### Implementation 💻
```python
class FenwickTree:
    def __init__(self, size):
        self.size = size
        self.tree = [0] * (size + 1)  # 1-indexed
    
    def update(self, index, value):
        """Add value to element at index"""
        index += 1  # Convert to 1-indexed
        while index <= self.size:
            self.tree[index] += value
            # Move to the next position that needs to be updated
            # The next position is determined by adding the least significant bit
            index += index & -index
    
    def get_sum(self, index):
        """Get sum from 0 to index (inclusive)"""
        index += 1  # Convert to 1-indexed
        result = 0
        while index > 0:
            result += self.tree[index]
            # Move to the parent
            # The parent is determined by removing the least significant bit
            index -= index & -index
        return result
    
    def range_sum(self, left, right):
        """Get sum from left to right (inclusive)"""
        return self.get_sum(right) - self.get_sum(left - 1)
    
    @classmethod
    def build(cls, arr):
        """Build a Fenwick tree from an array"""
        n = len(arr)
        fenwick = cls(n)
        for i in range(n):
            fenwick.update(i, arr[i])
        return fenwick
```

### Visualization 📊
```
Original array: [3, 2, -1, 6, 5, 4, -3, 3]

Fenwick Tree (conceptual):
             [19]
           /      \
      [10]          [9]
     /    \        /    \
  [5]     [5]    [9]    [0]
 /  \    /  \    /  \   /  \
[3] [2] [-1][6] [5] [4][-3][3]

Each node stores the sum of a specific range of elements.
```

### Bit Manipulation Insight 🧠
The key to understanding Fenwick trees is the relationship between an index and its binary representation:
- `index & -index` gives the least significant bit (LSB)
- For updates: add the LSB to move to the next node that needs updating
- For queries: subtract the LSB to move to the parent node

### Common Examples 🚀
```python
# Example: Range sum queries
arr = [3, 2, -1, 6, 5, 4, -3, 3]
bit = FenwickTree.build(arr)

# Get sum of elements from index 2 to 5
print(bit.range_sum(2, 5))  # Output: 14 (-1 + 6 + 5 + 4)

# Update element at index 3 (add 2)
bit.update(3, 2)
print(bit.range_sum(2, 5))  # Output: 16 (-1 + 8 + 5 + 4)
```

### Variations and Optimizations 🔄
- **2D Fenwick Tree**: For handling 2D range queries
- **Range Updates**: Using difference arrays to support range updates
- **Order Statistics**: Finding the k-th smallest element
- **Compressed Fenwick Tree**: For sparse data

### Advantages and Disadvantages ⚖️
#### Advantages ✅
- More memory efficient than segment trees
- Simpler implementation
- Fast updates and queries (O(log n))
- Works well for cumulative operations (sum, XOR, etc.)

#### Disadvantages ❌
- Limited to operations that have an inverse (like addition/subtraction)
- Cannot handle range updates as efficiently as segment trees without modifications
- Not as versatile as segment trees for complex operations
- 1-indexed implementation can be confusing

### Real-world Applications 🌐
- Range sum queries in databases
- Computational geometry (counting inversions)
- Suffix arrays construction
- Data compression algorithms
- Counting sort variations
- Online judge problems involving cumulative statistics

### Further Reading 📚
- Original Paper: "A New Data Structure for Cumulative Frequency Tables" by Peter M. Fenwick
- [TopCoder Tutorial on Binary Indexed Trees](https://www.topcoder.com/community/competitive-programming/tutorials/binary-indexed-trees/)
- [CP-Algorithms - Fenwick Tree](https://cp-algorithms.com/data_structures/fenwick.html)
- [Visualgo - Fenwick Tree Visualization](https://visualgo.net/en/fenwicktree)

---

## 📚 Suffix Array/Tree Reference Sheet 📚

### Definition 📝
**Suffix Array**: A sorted array of all suffixes of a string. It allows efficient string operations like pattern matching.

**Suffix Tree**: A compressed trie containing all suffixes of a string. It provides more functionality than suffix arrays but requires more space.

Both data structures are powerful tools for string processing and are used in text indexing, bioinformatics, and pattern matching.

### Complexity Analysis ⏱️
#### Suffix Array
| Operation                | Time Complexity       |
|--------------------------|------------------------|
| Construction (naive)     | O(n² log n)           |
| Construction (optimized) | O(n log n) or O(n)    |
| Pattern Matching         | O(m log n + occ)      |
| Longest Common Prefix    | O(1) with preprocessing|

#### Suffix Tree
| Operation                | Time Complexity       |
|--------------------------|------------------------|
| Construction             | O(n)                  |
| Pattern Matching         | O(m + occ)            |
| Longest Common Substring | O(n)                  |
| Space Requirement        | O(n)                  |

Where n is the string length, m is the pattern length, and occ is the number of occurrences.

### Core Operations 🔍
#### Suffix Array
- **Construction**: Build the sorted array of all suffixes
- **LCP Array**: Compute the Longest Common Prefix array
- **Pattern Search**: Find all occurrences of a pattern
- **Longest Repeated Substring**: Find the longest substring that appears multiple times

#### Suffix Tree
- **Construction**: Build the compressed trie of all suffixes
- **Pattern Matching**: Find if a pattern exists in the string
- **Longest Common Substring**: Find the longest substring common to two strings
- **All Repeats**: Find all repeated substrings

### Implementation 💻
#### Suffix Array (with naive construction)
```python
def build_suffix_array(s):
    """Build a suffix array for string s (naive approach)"""
    n = len(s)
    # Create list of (suffix, index) pairs
    suffixes = [(s[i:], i) for i in range(n)]
    # Sort by suffix
    suffixes.sort()
    # Extract the indices
    suffix_array = [index for _, index in suffixes]
    return suffix_array

def build_lcp_array(s, suffix_array):
    """Build the Longest Common Prefix array"""
    n = len(s)
    # Inverse suffix array
    rank = [0] * n
    for i in range(n):
        rank[suffix_array[i]] = i
    
    lcp = [0] * (n-1)
    k = 0  # LCP length
    
    for i in range(n):
        if rank[i] == n - 1:
            k = 0
            continue
        
        j = suffix_array[rank[i] + 1]  # Next suffix in SA
        
        # Compute LCP
        while i + k < n and j + k < n and s[i + k] == s[j + k]:
            k += 1
            
        lcp[rank[i]] = k
        
        if k > 0:
            k -= 1
            
    return lcp

def search_pattern(s, p, suffix_array):
    """Search for pattern p in string s using suffix array"""
    n, m = len(s), len(p)
    
    # Binary search
    left, right = 0, n - 1
    while left <= right:
        mid = (left + right) // 2
        suffix = s[suffix_array[mid]:]
        
        if suffix[:m] < p:
            left = mid + 1
        elif suffix[:m] > p:
            right = mid - 1
        else:
            # Found a match, now find all occurrences
            result = []
            
            # Find leftmost occurrence
            first = mid
            while first > 0 and s[suffix_array[first-1]:suffix_array[first-1]+m] == p:
                first -= 1
                
            # Find rightmost occurrence
            last = mid
            while last < n-1 and s[suffix_array[last+1]:suffix_array[last+1]+m] == p:
                last += 1
                
            # Collect all occurrences
            for i in range(first, last + 1):
                result.append(suffix_array[i])
                
            return result
            
    return []  # Pattern not found
```

#### Suffix Tree (simplified implementation using Ukkonen's algorithm)
```python
class Node:
    def __init__(self):
        self.children = {}
        self.suffix_link = None
        self.start = -1
        self.end = -1
        self.suffix_index = -1

class SuffixTree:
    def __init__(self, s):
        self.s = s + "$"  # Add terminal character
        self.root = Node()
        self.build_tree()
        
    def build_tree(self):
        # This is a simplified placeholder for Ukkonen's algorithm
        # A full implementation is complex and beyond the scope of this example
        n = len(self.s)
        
        # Naive approach: insert each suffix into the tree
        for i in range(n):
            current = self.root
            for j in range(i, n):
                if self.s[j] not in current.children:
                    new_node = Node()
                    new_node.start = j
                    new_node.end = n - 1
                    if j == i:  # This is a leaf
                        new_node.suffix_index = i
                    current.children[self.s[j]] = new_node
                    current = new_node
                else:
                    current = current.children[self.s[j]]
                    
    def search(self, pattern):
        """Search for pattern in the suffix tree"""
        current = self.root
        for char in pattern:
            if char not in current.children:
                return False
            current = current.children[char]
        return True
    
    def get_all_occurrences(self, pattern):
        """Get all occurrences of pattern"""
        current = self.root
        
        # Navigate to the node representing the pattern
        for char in pattern:
            if char not in current.children:
                return []
            current = current.children[char]
        
        # Collect all suffix indices below this node
        result = []
        self._collect_leaves(current, result)
        return result
    
    def _collect_leaves(self, node, result):
        """Collect all leaf nodes (suffix indices) below the given node"""
        if node.suffix_index != -1:
            result.append(node.suffix_index)
            return
        
        for child in node.children.values():
            self._collect_leaves(child, result)
```

### Visualization 📊
#### Suffix Array Example
```
String: "banana"
Suffixes:
0: banana
1: anana
2: nana
3: ana
4: na
5: a

Sorted Suffixes:
5: a
3: ana
1: anana
0: banana
4: na
2: nana

Suffix Array: [5, 3, 1, 0, 4, 2]
LCP Array: [0, 1, 3, 0, 0]
```

#### Suffix Tree (simplified)
```
String: "banana$"
                  root
                 /    \
                a      b
               / \      \
              n   $      a
             /           n
            a            a
           / \           n
          n   $          a
         /               $
        a
       /
      $
```

### Common Examples 🚀
```python
# Example: Find the longest repeated substring
def longest_repeated_substring(s):
    sa = build_suffix_array(s)
    lcp = build_lcp_array(s, sa)
    
    # Find the maximum LCP value
    max_lcp = 0
    max_idx = 0
    for i in range(len(lcp)):
        if lcp[i] > max_lcp:
            max_lcp = lcp[i]
            max_idx = i
    
    # If no repeats found
    if max_lcp == 0:
        return ""
    
    # Return the longest repeated substring
    start = sa[max_idx]
    return s[start:start + max_lcp]

# Example: Count distinct substrings
def count_distinct_substrings(s):
    n = len(s)
    sa = build_suffix_array(s)
    lcp = build_lcp_array(s, sa)
    
    # Total possible substrings: n(n+1)/2
    total_substrings = n * (n + 1) // 2
    
    # Subtract the overlapping substrings (sum of LCP values)
    overlapping = sum(lcp)
    
    return total_substrings - overlapping
```

### Advantages and Disadvantages ⚖️
#### Suffix Array
##### Advantages ✅
- More space-efficient than suffix trees
- Simpler to implement
- Better cache locality
- Sufficient for many string processing tasks

##### Disadvantages ❌
- Slower for some operations compared to suffix trees
- More complex algorithms for advanced operations
- Construction can be challenging to optimize

#### Suffix Tree
##### Advantages ✅
- Linear-time construction (Ukkonen's algorithm)
- O(m) pattern matching (vs O(m log n) for suffix arrays)
- Supports more complex string operations efficiently
- Directly represents all substrings

##### Disadvantages ❌
- Higher space requirements
- More complex implementation
- Poorer cache performance
- Harder to serialize/deserialize

### Real-world Applications 🌐
- **Bioinformatics**: DNA sequence alignment, genome assembly
- **Text Editors**: Efficient search and replace functionality
- **Information Retrieval**: Full-text indexing
- **Data Compression**: Finding repeated patterns
- **Plagiarism Detection**: Finding common substrings
- **Spell Checkers**: Finding similar words

### Further Reading 📚
- "Algorithms on Strings, Trees, and Sequences" by Dan Gusfield
- [Stanford CS166 - Suffix Arrays and Suffix Trees](https://web.stanford.edu/class/cs166/)
- [CP-Algorithms - Suffix Array](https://cp-algorithms.com/string/suffix-array.html)
- [Visualgo - Suffix Array and LCP Visualization](https://visualgo.net/en/suffixarray)
- "Linear Work Suffix Array Construction" by Kärkkäinen and Sanders

---

## 🌸 Bloom Filter Reference Sheet 🌸

### Definition 📝
A Bloom Filter is a space-efficient probabilistic data structure designed to test whether an element is a member of a set. It can tell you with certainty that an element is **not** in the set, but it may report false positives (incorrectly claim an element is in the set when it's not).

The structure uses multiple hash functions to map elements to positions in a bit array, enabling membership queries with O(1) time complexity.

### Complexity Analysis ⏱️
| Operation | Time Complexity | Space Complexity |
|-----------|-----------------|------------------|
| Insert    | O(k)            | O(m)             |
| Lookup    | O(k)            | O(m)             |

Where k is the number of hash functions and m is the size of the bit array.

### Core Operations 🔍
- **Insert(item)**: Add an item to the filter
- **Contains(item)**: Check if an item might be in the filter
- **OptimalParameters(n, p)**: Calculate optimal filter size and hash count for n items with false positive rate p

### Implementation 💻
```python
import math
import mmh3  # MurmurHash3 for better hash distribution

class BloomFilter:
    def __init__(self, size, hash_count):
        """
        Initialize a Bloom Filter
        
        Args:
            size: Size of the bit array
            hash_count: Number of hash functions to use
        """
        self.size = size
        self.hash_count = hash_count
        self.bit_array = [0] * size
    
    @classmethod
    def from_items_count(cls, n, false_positive_rate):
        """
        Create a Bloom Filter optimized for n items with given false positive rate
        
        Args:
            n: Expected number of items
            false_positive_rate: Desired false positive rate (0-1)
        """
        m = cls.optimal_size(n, false_positive_rate)
        k = cls.optimal_hash_count(m, n)
        return cls(m, k)
    
    @staticmethod
    def optimal_size(n, p):
        """Calculate optimal bit array size for n items and false positive rate p"""
        m = -(n * math.log(p)) / (math.log(2) ** 2)
        return math.ceil(m)
    
    @staticmethod
    def optimal_hash_count(m, n):
        """Calculate optimal number of hash functions for m bits and n items"""
        k = (m / n) * math.log(2)
        return math.ceil(k)
    
    def _get_hash_values(self, item):
        """Generate hash values for an item"""
        hash_values = []
        for i in range(self.hash_count):
            # Use different seed values for each hash function
            hash_val = mmh3.hash(str(item), i) % self.size
            hash_values.append(hash_val)
        return hash_values
    
    def add(self, item):
        """Add an item to the Bloom Filter"""
        for hash_val in self._get_hash_values(item):
            self.bit_array[hash_val] = 1
    
    def contains(self, item):
        """
        Check if an item might be in the set
        
        Returns:
            False: Item is definitely not in the set
            True: Item might be in the set (could be a false positive)
        """
        for hash_val in self._get_hash_values(item):
            if self.bit_array[hash_val] == 0:
                return False
        return True
    
    def false_positive_probability(self):
        """Calculate the current false positive probability"""
        # Count number of bits set to 1
        bits_set = sum(self.bit_array)
        # Calculate probability
        return (1 - math.exp(-self.hash_count * bits_set / self.size)) ** self.hash_count
```

### Visualization 📊
```
Bloom Filter with m=10, k=3:

Initial state:
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0]

After adding "apple":
Hash1("apple") = 2
Hash2("apple") = 5
Hash3("apple") = 8
[0, 0, 1, 0, 0, 1, 0, 0, 1, 0]

After adding "banana":
Hash1("banana") = 1
Hash2("banana") = 4
Hash3("banana") = 7
[0, 1, 1, 0, 1, 1, 0, 1, 1, 0]

Checking "apple": 
All positions (2,5,8) are set to 1, so return true (correct)

Checking "orange":
Hash1("orange") = 1
Hash2("orange") = 5
Hash3("orange") = 8
All positions (1,5,8) are set to 1, so return true (false positive!)

Checking "grape":
Hash1("grape") = 3
Hash2("grape") = 6
Hash3("grape") = 9
Position 3 is 0, so return false (correct)
```

### Mathematical Foundation 🧮
The false positive probability of a Bloom filter is approximately:

$p = (1 - e^{-kn/m})^k$

Where:
- $p$ is the false positive probability
- $k$ is the number of hash functions
- $n$ is the number of items in the filter
- $m$ is the size of the bit array

Optimal number of hash functions: $k = \frac{m}{n} \ln 2$

### Common Examples 🚀
```python
# Example: Web crawler to avoid visiting the same URL twice
def web_crawler_example():
    # Create a Bloom filter with 1 million expected URLs and 0.1% false positive rate
    bloom = BloomFilter.from_items_count(1000000, 0.001)
    
    urls_to_visit = ["https://example.com", "https://example.org", "https://example.net"]
    visited = set()  # For verification
    
    while urls_to_visit:
        url = urls_to_visit.pop(0)
        
        # Check if URL might have been visited
        if bloom.contains(url):
            print(f"Skipping {url} - already visited (or false positive)")
            continue
        
        # Process the URL
        print(f"Visiting {url}")
        visited.add(url)
        bloom.add(url)
        
        # Add new URLs to the queue
        new_urls = ["https://example.com/page1", "https://example.org/page2"]
        for new_url in new_urls:
            if not bloom.contains(new_url):
                urls_to_visit.append(new_url)

# Example: Spell checker with Bloom filter
def spell_checker_example():
    # Load dictionary
    dictionary = ["apple", "banana", "orange", "grape", "pineapple"]
    
    # Create Bloom filter and add dictionary words
    bloom = BloomFilter.from_items_count(len(dictionary), 0.01)
    for word in dictionary:
        bloom.add(word)
    
    # Check words
    words_to_check = ["apple", "aple", "banana", "bananas"]
    for word in words_to_check:
        if bloom.contains(word):
            print(f"'{word}' might be correctly spelled")
        else:
            print(f"'{word}' is definitely misspelled")
```

### Variations and Optimizations 🔄
- **Counting Bloom Filter**: Allows deletions by using counters instead of bits
- **Scalable Bloom Filter**: Dynamically grows as more elements are added
- **Compressed Bloom Filter**: Uses compression to reduce space requirements
- **Partitioned Bloom Filter**: Divides the bit array into k partitions
- **Blocked Bloom Filter**: Improves cache efficiency by grouping bits into blocks

### Advantages and Disadvantages ⚖️
#### Advantages ✅
- Space-efficient compared to exact data structures
- Constant time insertions and lookups
- No false negatives (if an element is not in the set, the filter will never report it as present)
- Privacy-preserving (difficult to enumerate the contents)
- Supports union operations

#### Disadvantages ❌
- False positives are possible and increase as the filter fills up
- Cannot retrieve the elements that were added
- Cannot delete elements (without modifications)
- Requires careful parameter tuning for optimal performance
- Multiple hash functions can be computationally expensive

### Real-world Applications 🌐
- **Web Crawlers**: Avoiding revisiting URLs
- **Databases**: Reducing disk lookups for non-existent keys
- **Caching Systems**: Quick check if item exists in cache
- **Network Routers**: Packet filtering and forwarding
- **Spell Checkers**: Quick verification of word validity
- **Cryptocurrency**: Bitcoin's SPV (Simplified Payment Verification)
- **Distributed Systems**: Reducing network traffic for set reconciliation

### Further Reading 📚
- Original Paper: "Space/Time Trade-offs in Hash Coding with Allowable Errors" by Burton H. Bloom
- [Stanford University - Bloom Filters](https://web.stanford.edu/class/cs166/lectures/05/Small05.pdf)
- [Medium - Bloom Filters Explained](https://medium.com/system-design-blog/bloom-filters-explained-5b4784858e46)
- [Interactive Bloom Filter Visualization](https://www.jasondavies.com/bloomfilter/)
- "Bloom Filters in Probabilistic Verification" by Michael Mitzenmacher and Eli Upfal

---

## 🔄 LRU Cache Reference Sheet 🔄

### Definition 📝
An LRU (Least Recently Used) Cache is a data structure that maintains a fixed-size collection of items, discarding the least recently used items when the cache reaches its capacity. It combines a hash table for O(1) lookups with a doubly linked list to track usage order.

LRU Cache implements a page replacement algorithm that removes the least recently used items first when the cache is full.

### Complexity Analysis ⏱️
| Operation | Time Complexity |
|-----------|-----------------|
| Get       | O(1)            |
| Put       | O(1)            |
| Delete    | O(1)            |

### Core Operations 🔍
- **Get(key)**: Retrieve an item from the cache and mark it as recently used
- **Put(key, value)**: Add or update an item in the cache, evicting the least recently used item if necessary
- **Delete(key)**: Remove an item from the cache

### Implementation 💻
```python
class Node:
    def __init__(self, key, value):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity):
        """
        Initialize an LRU Cache with fixed capacity
        
        Args:
            capacity: Maximum number of items the cache can hold
        """
        self.capacity = capacity
        self.cache = {}  # Hash table for O(1) lookups
        
        # Initialize doubly linked list with dummy head and tail
        self.head = Node(0, 0)  # Dummy head
        self.tail = Node(0, 0)  # Dummy tail
        self.head.next = self.tail
        self.tail.prev = self.head
    
    def _add_node(self, node):
        """Add node right after head (most recently used position)"""
        node.prev = self.head
        node.next = self.head.next
        
        self.head.next.prev = node
        self.head.next = node
    
    def _remove_node(self, node):
        """Remove a node from the linked list"""
        prev = node.prev
        next = node.next
        
        prev.next = next
        next.prev = prev
    
    def _move_to_front(self, node):
        """Move a node to the front (most recently used position)"""
        self._remove_node(node)
        self._add_node(node)
    
    def _pop_tail(self):
        """Remove the node before tail (least recently used)"""
        node = self.tail.prev
        self._remove_node(node)
        return node
    
    def get(self, key):
        """
        Get value for key and mark as recently used
        
        Returns:
            Value if key exists, -1 otherwise
        """
        if key not in self.cache:
            return -1
        
        # Update position in the linked list
        node = self.cache[key]
        self._move_to_front(node)
        
        return node.value
    
    def put(self, key, value):
        """
        Add or update key-value pair in the cache
        
        If key exists, update value and mark as recently used
        If key doesn't exist, add new entry and evict LRU item if necessary
        """
        # If key exists, update value and move to front
        if key in self.cache:
            node = self.cache[key]
            node.value = value
            self._move_to_front(node)
            return
        
        # If at capacity, remove least recently used item
        if len(self.cache) >= self.capacity:
            lru_node = self._pop_tail()
            del self.cache[lru_node.key]
        
        # Add new node
        new_node = Node(key, value)
        self.cache[key] = new_node
        self._add_node(new_node)
    
    def delete(self, key):
        """Remove a key from the cache if it exists"""
        if key in self.cache:
            node = self.cache[key]
            self._remove_node(node)
            del self.cache[key]
            return True
        return False
    
    def clear(self):
        """Clear the entire cache"""
        self.cache = {}
        self.head.next = self.tail
        self.tail.prev = self.head
```

### Visualization 📊
```
LRU Cache with capacity=3:

Initial state:
HEAD <-> TAIL
{}

After put(1, 'one'):
HEAD <-> [1:'one'] <-> TAIL
{1: Node(1, 'one')}

After put(2, 'two'):
HEAD <-> [2:'two'] <-> [1:'one'] <-> TAIL
{1: Node(1, 'one'), 2: Node(2, 'two')}

After put(3, 'three'):
HEAD <-> [3:'three'] <-> [2:'two'] <-> [1:'one'] <-> TAIL
{1: Node(1, 'one'), 2: Node(2, 'two'), 3: Node(3, 'three')}

After get(1): (1 is now most recently used)
HEAD <-> [1:'one'] <-> [3:'three'] <-> [2:'two'] <-> TAIL
{1: Node(1, 'one'), 2: Node(2, 'two'), 3: Node(3, 'three')}

After put(4, 'four'): (2 is evicted as least recently used)
HEAD <-> [4:'four'] <-> [1:'one'] <-> [3:'three'] <-> TAIL
{1: Node(1, 'one'), 3: Node(3, 'three'), 4: Node(4, 'four')}
```

### Common Examples 🚀
```python
# Example: Web page caching
def web_cache_example():
    # Create a cache with capacity for 3 pages
    page_cache = LRUCache(3)
    
    # User visits pages
    page_cache.put("home", "<html>Home Page</html>")
    page_cache.put("about", "<html>About Page</html>")
    page_cache.put("contact", "<html>Contact Page</html>")
    
    # User revisits home page (moves to most recently used)
    home_content = page_cache.get("home")
    print(f"Home page content: {home_content}")
    
    # User visits a new page, contact page gets evicted
    page_cache.put("products", "<html>Products Page</html>")
    
    # Trying to access contact page (cache miss)
    contact_content = page_cache.get("contact")
    print(f"Contact page content: {contact_content}")  # Returns -1 (not in cache)
    
    # Current cache state: products (most recent), home, about (least recent)

# Example: Database query result caching
def db_query_cache_example():
    # Create a cache for database query results
    query_cache = LRUCache(5)
    
    # Function to get data (from cache or database)
    def get_data(query):
        # Try to get from cache first
        result = query_cache.get(query)
        if result != -1:
            print(f"Cache hit for query: {query}")
            return result
        
        # Cache miss, query the database
        print(f"Cache miss for query: {query}, querying database...")
        # Simulate database query
        result = f"Result for {query}"
        
        # Store in cache for future use
        query_cache.put(query, result)
        return result
    
    # Simulate queries
    queries = [
        "SELECT * FROM users WHERE id = 1",
        "SELECT * FROM products WHERE category = 'electronics'",
        "SELECT * FROM orders WHERE date > '2023-01-01'",
        "SELECT * FROM users WHERE id = 1",  # Cache hit
        "SELECT * FROM inventory WHERE stock < 10",
        "SELECT * FROM products WHERE price < 100",
        "SELECT * FROM users WHERE id = 1",  # Cache hit
    ]
    
    for query in queries:
        result = get_data(query)
```

### Variations and Optimizations 🔄
- **LRU-K**: Considers the last K references instead of just the most recent one
- **2Q (Two Queue)**: Uses two queues to better identify frequently accessed items
- **SLRU (Segmented LRU)**: Divides the cache into protected and probationary segments
- **TLRU (Time-aware LRU)**: Incorporates time-to-live (TTL) for cached items
- **ARC (Adaptive Replacement Cache)**: Balances between recency and frequency
- **CLOCK**: A more efficient approximation of LRU using a circular buffer

### Advantages and Disadvantages ⚖️
#### Advantages ✅
- O(1) time complexity for all operations
- Simple implementation and intuitive behavior
- Works well for workloads with temporal locality
- Automatically adapts to changing access patterns
- Bounded memory usage

#### Disadvantages ❌
- Not optimal for frequency-based access patterns
- Doesn't consider item size (all items consume equal capacity)
- Requires additional memory for the linked list pointers
- Single cache miss can cause thrashing in some workloads
- No awareness of access cost differences between items

### Real-world Applications 🌐
- **Web Browsers**: Caching recently visited pages
- **Database Systems**: Query result caching
- **Operating Systems**: Page replacement algorithms
- **Content Delivery Networks (CDNs)**: Edge caching
- **Mobile Applications**: Image and data caching
- **Distributed Systems**: In-memory caches like Redis, Memcached
- **File Systems**: Buffer caching

### Further Reading 📚
- "Computer Architecture: A Quantitative Approach" by Hennessy and Patterson
- [LeetCode Problem 146: LRU Cache](https://leetcode.com/problems/lru-cache/)
- [Caffeine: A high performance caching library for Java](https://github.com/ben-manes/caffeine)
- [Redis LRU Cache Implementation](https://redis.io/topics/lru-cache)
- [Microsoft Research - Adaptive Replacement Cache](https://www.usenix.org/legacy/events/fast03/tech/full_papers/megiddo/megiddo.pdf)

---

## 🔍 Advanced Data Structures Comparison Chart 🔍

| Data Structure | Best Use Cases | Time Complexity (Key Operations) | Space Complexity | Key Advantages | Key Disadvantages |
|----------------|----------------|----------------------------------|------------------|----------------|-------------------|
| **Disjoint Set / Union-Find** | Graph connectivity, Kruskal's algorithm | Find, Union: O(α(n)) ≈ O(1) | O(n) | Near-constant time operations, Simple implementation | Cannot delete elements or split sets |
| **Fenwick Tree (BIT)** | Range sum queries, Cumulative frequencies | Update, Query: O(log n) | O(n) | More memory efficient than segment trees, Simple implementation | Limited to operations with inverse |
| **Suffix Array** | String pattern matching, Substring problems | Construction: O(n log n), Search: O(m log n) | O(n) | Space efficient, Good cache locality | Slower for some operations than suffix trees |
| **Suffix Tree** | Advanced string operations, Longest common substring | Construction: O(n), Search: O(m) | O(n) | Linear-time pattern matching, Supports complex string operations | Higher space requirements, Complex implementation |
| **Bloom Filter** | Membership testing, Avoiding duplicates | Insert, Lookup: O(k) | O(m) | Space-efficient, Constant-time operations | Allows false positives, Cannot delete elements |
| **LRU Cache** | Caching with eviction policy | Get, Put: O(1) | O(capacity) | O(1) operations, Adapts to access patterns | Not optimal for frequency-based workloads |

### When to Choose Each Data Structure 🤔

- **Choose Disjoint Set / Union-Find when:**
  - You need to track connected components in a graph
  - You're implementing Kruskal's algorithm
  - You need to determine if two elements are in the same set
  - You have mostly union operations and few find operations

- **Choose Fenwick Tree (BIT) when:**
  - You need efficient range sum queries
  - You have frequent updates to array elements
  - You need a simpler alternative to segment trees
  - Memory efficiency is important

- **Choose Suffix Array when:**
  - You need to find patterns in strings
  - Memory is a concern compared to suffix trees
  - You're solving substring problems
  - You need better cache performance

- **Choose Suffix Tree when:**
  - You need the most efficient string pattern matching
  - You're solving complex string problems (longest common substring)
  - You need to find all occurrences of patterns
  - Performance is more important than space

- **Choose Bloom Filter when:**
  - You need to test set membership with space efficiency
  - False positives are acceptable but false negatives are not
  - You're filtering items before expensive lookups
  - Privacy of the underlying data is important

- **Choose LRU Cache when:**
  - You need a fixed-size cache with automatic eviction
  - Your workload has temporal locality (recently used items likely to be used again)
  - You need O(1) access time
  - You're implementing a page replacement policy

---

This Advanced Data Structures Reference Bible provides a comprehensive overview of powerful data structures beyond the basics. Each section includes detailed explanations, implementations, complexity analysis, and practical examples to help you understand when and how to use these structures effectively in your projects.