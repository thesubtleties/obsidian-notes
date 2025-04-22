## Folder 1: Algorithm Patterns

1. **Two Pointers**
   - Regular two pointers (left/right)
   - Fast/slow pointers
   - Three pointers
   - Multiple arrays pointers

2. **Sliding Window**
   - Fixed-size window
   - Variable-size window
   - String window problems
   - Maximum/minimum subarray

3. **Binary Search**
   - Standard implementation
   - Rotated array search
   - Search in 2D matrix
   - Binary search on answer space

4. **Breadth-First Search (BFS)**
   - Tree level-order traversal
   - Shortest path in unweighted graph
   - Connected components
   - Grid traversal

5. **Depth-First Search (DFS)**
   - Tree traversals (pre/in/post-order)
   - Path finding
   - Connected components
   - Cycle detection

6. **Backtracking**
   - Permutations
   - Combinations
   - Subsets
   - N-Queens, Sudoku solver

7. **Dynamic Programming**
   - 1D problems (fibonacci, climbing stairs)
   - 2D problems (grid traversal, edit distance)
   - Knapsack variations
   - Longest common subsequence/substring
   - State machine approaches

8. **Greedy Algorithms**
   - Activity selection
   - Huffman coding
   - Interval scheduling
   - Fractional knapsack

9. **Divide and Conquer**
   - Merge sort
   - Quick sort
   - Closest pair of points
   - Karatsuba multiplication

10. **Graph Algorithms**
    - Topological sort
    - Shortest path (Dijkstra's, Bellman-Ford)
    - Minimum spanning tree (Prim's, Kruskal's)
    - Union-Find (Disjoint Set)

11. **Bit Manipulation**
    - Basic operations
    - Counting bits
    - Power of two checks
    - Bit masking techniques

12. **Math and Geometry**
    - GCD/LCM
    - Prime numbers
    - Factorial/Combinations
    - Line intersection

13. **Prefix Sum/Cumulative Sum**
    - Range sum queries
    - Equilibrium index
    - Subarray sum equals k

14. **Monotonic Stack/Queue**
    - Next greater/smaller element
    - Largest rectangle in histogram
    - Sliding window maximum

## Folder 2: Data Structures

1. **Arrays**
   - Basic operations
   - In-place modifications
   - Subarrays/subsequences
   - Matrix operations

2. **Strings**
   - String manipulation
   - String matching
   - Palindromes
   - Anagrams

3. **Hash Tables**
   - Dictionary implementation
   - Collision handling
   - Common dictionary patterns
   - Custom hash functions

4. **Linked Lists**
   - Singly linked list
   - Doubly linked list
   - Circular linked list
   - Operations (reverse, detect cycle, merge)

5. **Stacks**
   - Implementation
   - Expression evaluation
   - Parentheses matching
   - Monotonic stack applications

6. **Queues**
   - Regular queue
   - Circular queue
   - Priority queue
   - Deque (double-ended queue)

7. **Trees**
   - Binary tree
   - Binary search tree
   - AVL tree
   - Red-black tree
   - N-ary tree
   - Segment tree

8. **Heaps**
   - Min heap
   - Max heap
   - Heap operations
   - Heap sort
   - Custom comparators

9. **Tries**
   - Basic implementation
   - Prefix matching
   - Word dictionary
   - Autocomplete

10. **Graphs**
    - Adjacency matrix
    - Adjacency list
    - Directed vs undirected
    - Weighted vs unweighted
    - Connected components

11. **Advanced Data Structures**
    - Disjoint Set/Union-Find
    - Fenwick Tree (Binary Indexed Tree)
    - Suffix Array/Tree
    - Bloom Filter
    - LRU Cache

## For Each Pattern/Data Structure Sheet:

1. **Conceptual Overview**
   - Brief explanation of the pattern/structure
   - When to use it
   - Time/space complexity

2. **Python Implementation**
   - Clean, reusable code template
   - Common variations

3. **Common Problem Types**
   - List of problem categories that use this pattern
   - Example problems from LeetCode (with numbers)

4. **Gotchas & Tips**
   - Edge cases to watch for
   - Optimization techniques
   - Python-specific tricks

## Example Sheet: Two Pointers Pattern

```markdown
# Two Pointers Pattern

## Concept
Using two pointers to solve problems with less time/space complexity, typically O(n).

## When to Use
- Searching pairs in sorted array
- Removing duplicates
- Finding triplets/subarrays with certain properties
- Palindrome checking

## Python Implementation

### Basic Two Pointers (Left/Right)
```python
def two_pointers(arr):
    left, right = 0, len(arr) - 1
    
    while left < right:
        # Process elements at arr[left] and arr[right]
        
        # Move pointers based on condition
        if CONDITION:
            left += 1
        else:
            right -= 1
    
    return result
```

### Fast/Slow Pointers
```python
def fast_slow_pointers(arr):
    slow = fast = 0
    
    while fast < len(arr):
        # Process or compare elements
        
        # Move pointers at different speeds
        slow += 1
        fast += 2
    
    return result
```

## Common Problem Types
1. Two Sum (LeetCode #1)
2. Container With Most Water (LeetCode #11)
3. 3Sum (LeetCode #15)
4. Remove Duplicates from Sorted Array (LeetCode #26)
5. Valid Palindrome (LeetCode #125)

## Gotchas & Tips
- Always check for empty arrays
- Be careful with pointer bounds
- For sorted arrays, consider binary search instead for certain problems
- In Python, use `while left < right:` to avoid overlap
```

This organizational structure will give you a comprehensive reference system that you can gradually build as you learn and practice. Start with the most common patterns and data structures, and expand your sheets as you encounter new concepts.