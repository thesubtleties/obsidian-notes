## Definition 📝
Dynamic Programming (DP) is a method for solving complex problems by breaking them down into simpler subproblems. It solves each subproblem just once and stores its answer in a table, avoiding the work of recalculating the answer every time the subproblem is encountered.

Key characteristics:
- **Optimal substructure**: An optimal solution contains optimal solutions to its subproblems
- **Overlapping subproblems**: The same subproblems are solved multiple times
- **Memoization**: Storing results of expensive function calls to avoid redundant calculations

## Complexity Analysis ⏱️
- Time complexity: Typically O(n²) or O(n*m) for many DP problems, but varies based on state space
- Space complexity: Usually matches the dimensions of the DP table (e.g., O(n) for 1D problems, O(n*m) for 2D)

The efficiency gain from DP comes from trading space for time - storing solutions to avoid recomputation.

## Core Operations 🔍
- **Identify subproblems**: Break down the original problem into smaller, overlapping pieces
- **Define state**: Create a state representation that captures all necessary information
- **Formulate recurrence relation**: Express solution to current state in terms of previous states
- **Determine base cases**: Define solutions for the smallest subproblems
- **Implement solution**: Either top-down (recursion + memoization) or bottom-up (tabulation)

## DP Approaches 💻

### Top-Down (Memoization)
```python
# Example: Fibonacci with memoization
def fibonacci(n, memo={}):
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    
    memo[n] = fibonacci(n-1, memo) + fibonacci(n-2, memo)
    return memo[n]
```

### Bottom-Up (Tabulation)
```python
# Example: Fibonacci with tabulation
def fibonacci(n):
    if n <= 1:
        return n
        
    dp = [0] * (n+1)
    dp[0], dp[1] = 0, 1
    
    for i in range(2, n+1):
        dp[i] = dp[i-1] + dp[i-2]
        
    return dp[n]
```

## Common DP Patterns 🔄

### 1D Problems 📏
Problems where state depends on previous positions in a single dimension.

#### Fibonacci Sequence
```python
def fibonacci(n):
    if n <= 1:
        return n
    
    prev, curr = 0, 1
    for i in range(2, n+1):
        prev, curr = curr, prev + curr
    
    return curr
```

#### Climbing Stairs
Problem: Count ways to climb n stairs taking 1 or 2 steps at a time.

```python
def climbStairs(n):
    if n <= 2:
        return n
    
    dp = [0] * (n+1)
    dp[1], dp[2] = 1, 2
    
    for i in range(3, n+1):
        dp[i] = dp[i-1] + dp[i-2]
    
    return dp[n]
```

#### Coin Change
Problem: Find minimum number of coins needed to make a given amount.

```python
def coinChange(coins, amount):
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    
    for coin in coins:
        for x in range(coin, amount + 1):
            dp[x] = min(dp[x], dp[x - coin] + 1)
    
    return dp[amount] if dp[amount] != float('inf') else -1
```

### 2D Problems 📊
Problems where state depends on positions in two dimensions.

#### Grid Traversal
Problem: Count ways to move from top-left to bottom-right in a grid.

```python
def uniquePaths(m, n):
    dp = [[1 for _ in range(n)] for _ in range(m)]
    
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = dp[i-1][j] + dp[i][j-1]
    
    return dp[m-1][n-1]
```

#### Edit Distance
Problem: Find minimum operations to convert string1 to string2.

```python
def minDistance(word1, word2):
    m, n = len(word1), len(word2)
    dp = [[0 for _ in range(n+1)] for _ in range(m+1)]
    
    # Base cases
    for i in range(m+1):
        dp[i][0] = i
    for j in range(n+1):
        dp[0][j] = j
    
    # Fill dp table
    for i in range(1, m+1):
        for j in range(1, n+1):
            if word1[i-1] == word2[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(dp[i-1][j],    # Delete
                                   dp[i][j-1],    # Insert
                                   dp[i-1][j-1])  # Replace
    
    return dp[m][n]
```

### Knapsack Variations 🎒
Problems involving selection from items with constraints.

#### 0/1 Knapsack
Problem: Maximize value while keeping total weight under capacity.

```python
def knapsack(values, weights, capacity):
    n = len(values)
    dp = [[0 for _ in range(capacity + 1)] for _ in range(n + 1)]
    
    for i in range(1, n + 1):
        for w in range(capacity + 1):
            if weights[i-1] <= w:
                dp[i][w] = max(values[i-1] + dp[i-1][w-weights[i-1]], dp[i-1][w])
            else:
                dp[i][w] = dp[i-1][w]
    
    return dp[n][capacity]
```

#### Subset Sum
Problem: Determine if a subset of elements can sum to a target value.

```python
def canPartition(nums, target_sum):
    dp = [False] * (target_sum + 1)
    dp[0] = True
    
    for num in nums:
        for j in range(target_sum, num - 1, -1):
            dp[j] |= dp[j - num]
    
    return dp[target_sum]
```

### Longest Common Subsequence/Substring 📝
Problems finding common elements between sequences.

#### Longest Common Subsequence
```python
def longestCommonSubsequence(text1, text2):
    m, n = len(text1), len(text2)
    dp = [[0 for _ in range(n+1)] for _ in range(m+1)]
    
    for i in range(1, m+1):
        for j in range(1, n+1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    
    return dp[m][n]
```

#### Longest Common Substring
```python
def longestCommonSubstring(text1, text2):
    m, n = len(text1), len(text2)
    dp = [[0 for _ in range(n+1)] for _ in range(m+1)]
    max_length = 0
    
    for i in range(1, m+1):
        for j in range(1, n+1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
                max_length = max(max_length, dp[i][j])
    
    return max_length
```

### State Machine Approaches 🤖
Problems modeled as transitions between states.

#### Stock Trading with Cooldown
```python
def maxProfit(prices):
    if not prices:
        return 0
    
    n = len(prices)
    # hold: max profit when holding stock
    # sold: max profit when just sold stock
    # rest: max profit when not holding stock and not in cooldown
    hold, sold, rest = -prices[0], 0, 0
    
    for i in range(1, n):
        prev_hold, prev_sold, prev_rest = hold, sold, rest
        
        # Either keep holding or buy stock after rest
        hold = max(prev_hold, prev_rest - prices[i])
        # Sell the stock
        sold = prev_hold + prices[i]
        # Either stay in rest or come from cooldown
        rest = max(prev_rest, prev_sold)
    
    # Final state must be either sold or rest
    return max(sold, rest)
```

## Visualization 📊
### Fibonacci Sequence DP Table
```
n:     0  1  2  3  4  5  6  7  8
dp[n]: 0  1  1  2  3  5  8  13 21
```

### Grid Traversal DP Table (3x3 grid)
```
[1, 1, 1]
[1, 2, 3]
[1, 3, 6]
```

## Step-by-Step DP Problem Solving 🚶
1. **Identify if it's a DP problem**:
   - Does it ask for optimization (min/max/count)?
   - Can it be broken into overlapping subproblems?
   - Is there a recursive solution?

2. **Define state**:
   - What information do we need to solve a subproblem?
   - For 1D: `dp[i]` might represent solution for first i elements
   - For 2D: `dp[i][j]` might represent solution for subproblem defined by i and j

3. **Establish recurrence relation**:
   - How does the current state relate to previous states?
   - Example: `dp[i] = min(dp[i-1] + cost[i-1], dp[i-2] + cost[i-2])`

4. **Identify base cases**:
   - What are the smallest subproblems we can solve directly?
   - Example: `dp[0] = 0, dp[1] = cost[0]`

5. **Determine calculation order**:
   - Bottom-up: Start from base cases and build up
   - Top-down: Start from the final state and recursively solve subproblems

6. **Implement solution**:
   - Write code for either approach
   - Consider space optimization if possible

7. **Test with examples**:
   - Verify solution with small test cases
   - Trace through the algorithm step by step

## Advantages and Disadvantages ⚖️
### Advantages ✅
- Solves problems with exponential naive solutions in polynomial time
- Avoids redundant calculations
- Can solve a wide variety of optimization problems
- Often leads to elegant solutions

### Disadvantages ❌
- Requires identifying the correct state and recurrence relation
- May use significant memory for large problems
- Can be difficult to debug
- Sometimes harder to understand than greedy or divide-and-conquer approaches

## Real-world Applications 🌐
- Sequence alignment in bioinformatics
- Resource allocation in economics
- Shortest path algorithms in networking
- Text similarity and spell checking
- Speech recognition
- Image processing and computer vision
- Natural language processing

## Further Reading 📚
- Books:
  - "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein
  - "Algorithms" by Robert Sedgewick and Kevin Wayne
  - "Dynamic Programming for Coding Interviews" by Meenakshi and Kamal Rawat

- Online Resources:
  - [Geeks for Geeks - Dynamic Programming](https://www.geeksforgeeks.org/dynamic-programming/)
  - [LeetCode - Dynamic Programming Patterns](https://leetcode.com/discuss/general-discussion/458695/dynamic-programming-patterns)
  - [Topcoder DP Tutorial](https://www.topcoder.com/community/competitive-programming/tutorials/dynamic-programming-from-novice-to-advanced/)
  - [Visualgo - DP Visualization](https://visualgo.net/en)

- Courses:
  - MIT OpenCourseWare - Introduction to Algorithms
  - Stanford's Algorithms Specialization on Coursera
  - Algorithms, Part II on Coursera by Princeton University

---