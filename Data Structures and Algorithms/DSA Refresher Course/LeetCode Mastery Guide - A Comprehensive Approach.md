# LeetCode Mastery Guide: A Comprehensive Approach

*Based on wisdom from [yangshunz](https://old.reddit.com/user/yangshunz), who did 500+ LeetCode questions, created Blind 75, and interviewed hundreds of candidates.*

## Introduction

LeetCode has become an unavoidable part of the software engineering interview process. While many engineers (including the original author) dislike this approach to technical assessment, it remains a standard hurdle in the industry. This guide aims to help you navigate this challenge efficiently and effectively.

## Essential Resources

- **[Tech Interview Handbook](https://www.techinterviewhandbook.org/software-engineering-interview-guide/)**: A comprehensive guide covering all aspects of technical interviews
- **[Grind 75](https://www.techinterviewhandbook.org/grind75)**: An interactive tool that creates a customized study plan based on your available time, with problems ordered by importance and difficulty
  - Adjust for your schedule (1-26 weeks)
  - Select your target companies
  - Get a week-by-week breakdown of problems to solve
  - Problems range from the core 75 to 169+ depending on your settings

## Foundational Principles

### 1. Master CS Fundamentals First

Before diving into LeetCode problems, ensure you have a solid understanding of:

- **Data Structures**: Arrays, linked lists, stacks, queues, hash tables, trees, graphs, heaps
- **Algorithms**: Sorting, searching, recursion, dynamic programming
- **Complexity Analysis**: Big O notation for time and space complexity

Understanding when to use each data structure and algorithm is more important than memorizing solutions. Create a quick reference table for yourself:

| Data Structure | Time Complexity (Average) | Best Used For |
|----------------|---------------------------|--------------|
| Array | Access: O(1), Search: O(n) | Sequential data, random access |
| Hash Table | Access/Search: O(1) | Fast lookups, counting |
| Binary Tree | Search: O(log n) | Hierarchical data, sorted data |
| Graph | Varies | Relationship networks, pathfinding |

### 2. Strategic Problem Selection

- **Start with Easy**: Build confidence and fundamentals
- **Focus on Medium**: These represent the majority of interview questions
- **Sample Hard Problems**: Tackle famous ones like:
  - Word Ladder
  - Serialize/Deserialize Binary Tree
  - Trapping Rain Water
  - LRU Cache

## Effective Practice Techniques

### 1. Simulate Real Interview Conditions

Real interviews differ from LeetCode in several key ways:

- **Vague Requirements**: Questions are intentionally underspecified
- **No Test Cases**: You'll need to generate your own examples
- **Time Pressure**: Usually 30-45 minutes per problem
- **Communication Focus**: Explaining your thought process is critical

**Practice Tips:**
- Set a timer for 30 minutes
- Verbalize your thought process (even when alone)
- Determine approach and complexity before coding
- Write complexity as a comment above your solution
- Test your solution with your own examples before submitting

### 2. Structured Problem-Solving Approach

For each problem:

1. **Understand**: Clarify the problem, identify constraints
2. **Plan**: Develop an approach, analyze complexity
3. **Implement**: Write clean, readable code
4. **Test**: Verify with examples, edge cases
5. **Optimize**: Consider improvements if time permits

### 3. When You're Stuck

- Give yourself 30 minutes of focused effort
- If close to a solution, continue a bit longer
- Otherwise, study the solution carefully
- **Important**: Revisit the problem in a few days without looking at the answer
- Try similar problems to reinforce the pattern

## Priority Topics

Focus your efforts on these high-value areas:

### Tier 1 (Essential)
- Graph traversals (DFS, BFS)
- Binary search
- Tree algorithms
- Hash table applications
- Two-pointer techniques

### Tier 2 (Important)
- Dynamic programming
- Backtracking with memoization
- Sliding window
- Heap operations
- Union-find

### Tier 3 (Company-Specific)
- Google: Dynamic programming, graph algorithms
- Amazon: System design, OOP
- Facebook: Array manipulation, string problems
- Microsoft: Tree and graph problems

## Language Considerations

- **Mastery > Language Choice**: Better to use a language you know well
- **If Time Permits**: Python offers concise syntax and built-in data structures
- **Language-Specific Cheat Sheet**: Create a reference for useful built-ins and patterns

Example Python patterns:
```python
# Counter for frequency analysis
from collections import Counter
counts = Counter([1,2,2,3,3,3])  # {1:1, 2:2, 3:3}

# DefaultDict for graph building
from collections import defaultdict
graph = defaultdict(list)
for u, v in edges:
    graph[u].append(v)
    
# Heap for priority queue
import heapq
heap = []
heapq.heappush(heap, (priority, item))
```

## Quantity vs. Quality

- **Optimal Range**: 100-200 carefully selected problems
- **Diminishing Returns**: After ~300 problems, patterns become repetitive
- **Recommended Resources**: 
  - [Grind 75](https://www.techinterviewhandbook.org/grind75) - Interactive and customizable study plan
  - [Tech Interview Handbook](https://www.techinterviewhandbook.org/software-engineering-interview-guide/) - Comprehensive interview preparation guide
- **Quality Metrics**: Can you solve without hints? Explain the solution? Identify patterns?

## Sustainable Practice Schedule

- **Timeline**: Start 2-3 months before interviews
- **Daily Practice**: 1-2 hours per day
- **Spaced Repetition**: Revisit problems at increasing intervals
- **Weekly Assessment**: Participate in LeetCode contests to gauge progress
- **Success Metric**: Consistently solving 3+ problems in weekly contests

## Mental Approach

- **Perfectionism is the Enemy**: You'll never feel 100% ready
- **Interviewing is Probabilistic**: Performance is a function of preparation and luck
- **Failure is Normal**: Even top engineers fail interviews
- **Focus on Progress**: Track improvements in your problem-solving speed and accuracy

## Creating Your Knowledge Base

For each problem you solve, document:
- Problem statement (brief)
- Your approach and reasoning
- Time and space complexity
- Code solution
- Key insights or patterns
- Related problems

## Sample Problem Documentation

```markdown
# Two Sum (Easy)

## Problem
Given an array of integers and a target sum, return indices of two numbers that add up to the target.

## Approach
Use a hash map to store values we've seen and their indices. For each number, check if (target - num) exists in our map.

## Complexity
- Time: O(n) - single pass through the array
- Space: O(n) - storing up to n elements in the hash map

## Code
```python
def twoSum(nums, target):
    seen = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []
```

## Key Insight
This demonstrates the "complement pattern" - using a hash map to find pairs with a specific relationship.

## Related Problems
- 3Sum
- 4Sum
- Two Sum II (sorted array)
```

## Using the Tech Interview Handbook

The [Tech Interview Handbook](https://www.techinterviewhandbook.org/software-engineering-interview-guide/) provides comprehensive resources beyond just coding problems:

1. **Algorithm Cheatsheets**: Quick references for common algorithms and data structures
2. **Behavioral Interview Preparation**: STAR method examples and common questions
3. **System Design Primer**: Approaches to designing scalable systems
4. **Negotiation Tactics**: Maximizing your compensation package
5. **Company-Specific Guides**: Tailored advice for top tech companies

Use the handbook alongside your LeetCode practice for a well-rounded interview preparation strategy.

Remember, the goal isn't just to pass interviews but to become a better problem solver. These skills will serve you throughout your engineering career.

*Credit: This guide expands on wisdom shared by [yangshunz](https://old.reddit.com/user/yangshunz), creator of Blind 75 and the Tech Interview Handbook.*