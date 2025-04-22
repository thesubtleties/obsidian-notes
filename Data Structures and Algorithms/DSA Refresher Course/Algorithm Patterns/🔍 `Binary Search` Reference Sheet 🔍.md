## Definition 📝
Binary Search is a divide-and-conquer search algorithm that finds the position of a target value within a **sorted array**. It works by repeatedly dividing the search interval in half, comparing the middle element with the target value, and eliminating the half that cannot contain the target.

Key characteristics:
- Requires a **sorted** collection
- Significantly faster than linear search
- Divides the search space in half with each step

## Complexity Analysis ⏱️
| Case | Time Complexity | Space Complexity |
|------|-----------------|------------------|
| Best | O(1)            | O(1) iterative   |
| Average | O(log n)     | O(1) iterative / O(log n) recursive |
| Worst | O(log n)       | O(1) iterative / O(log n) recursive |

Binary search is extremely efficient because it eliminates half of the remaining elements at each step, resulting in logarithmic time complexity.

## Core Operations 🔍
- **Find target**: Determine if a specific value exists in the array
- **Find first occurrence**: Find the first position of a target value
- **Find last occurrence**: Find the last position of a target value
- **Find insertion point**: Determine where a value should be inserted to maintain sorted order

## Standard Implementation 💻

### Iterative Approach
```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    
    while left <= right:
        mid = left + (right - left) // 2  # Avoids integer overflow
        
        # Check if target is present at mid
        if arr[mid] == target:
            return mid
        
        # If target is greater, ignore left half
        elif arr[mid] < target:
            left = mid + 1
        
        # If target is smaller, ignore right half
        else:
            right = mid - 1
    
    # Target is not present in the array
    return -1
```

### Recursive Approach
```python
def binary_search_recursive(arr, target, left, right):
    # Base case: element not found
    if left > right:
        return -1
    
    mid = left + (right - left) // 2
    
    # If element is present at the middle
    if arr[mid] == target:
        return mid
    
    # If element is smaller than mid, search in left subarray
    elif arr[mid] > target:
        return binary_search_recursive(arr, target, left, mid - 1)
    
    # Else search in right subarray
    else:
        return binary_search_recursive(arr, target, mid + 1, right)
```

## Common Variations 🔄

### Finding First Occurrence
```python
def find_first_occurrence(arr, target):
    left, right = 0, len(arr) - 1
    result = -1
    
    while left <= right:
        mid = left + (right - left) // 2
        
        if arr[mid] == target:
            result = mid  # Save this as a potential result
            right = mid - 1  # Continue searching in the left half
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
            
    return result
```

### Finding Last Occurrence
```python
def find_last_occurrence(arr, target):
    left, right = 0, len(arr) - 1
    result = -1
    
    while left <= right:
        mid = left + (right - left) // 2
        
        if arr[mid] == target:
            result = mid  # Save this as a potential result
            left = mid + 1  # Continue searching in the right half
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
            
    return result
```

### Finding Insertion Point
```python
def find_insertion_point(arr, target):
    left, right = 0, len(arr) - 1
    
    while left <= right:
        mid = left + (right - left) // 2
        
        if arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
            
    return left  # This is where target should be inserted
```

## Advanced Applications 🚀

### Search in Rotated Sorted Array
```python
def search_rotated(nums, target):
    left, right = 0, len(nums) - 1
    
    while left <= right:
        mid = left + (right - left) // 2
        
        if nums[mid] == target:
            return mid
        
        # Check if left half is sorted
        if nums[left] <= nums[mid]:
            # Check if target is in the left half
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1
        # Right half is sorted
        else:
            # Check if target is in the right half
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1
                
    return -1
```

### Search in 2D Sorted Matrix
```python
def search_matrix(matrix, target):
    if not matrix or not matrix[0]:
        return False
    
    rows, cols = len(matrix), len(matrix[0])
    left, right = 0, rows * cols - 1
    
    while left <= right:
        mid = left + (right - left) // 2
        # Convert 1D index to 2D coordinates
        row, col = mid // cols, mid % cols
        
        if matrix[row][col] == target:
            return True
        elif matrix[row][col] < target:
            left = mid + 1
        else:
            right = mid - 1
            
    return False
```

### Binary Search on Answer Space
This technique is used when the answer itself has a range, and we can binary search on that range.

```python
def minimum_capacity(weights, days):
    # Function to check if we can ship all packages within 'days' days
    # with a maximum capacity of 'capacity'
    def can_ship(capacity):
        day_count = 1
        current_weight = 0
        
        for weight in weights:
            if current_weight + weight > capacity:
                day_count += 1
                current_weight = weight
            else:
                current_weight += weight
                
            if day_count > days:
                return False
                
        return True
    
    # Binary search on the capacity
    left = max(weights)  # Minimum capacity must be at least the heaviest package
    right = sum(weights)  # Maximum capacity is sum of all weights
    
    while left < right:
        mid = left + (right - left) // 2
        
        if can_ship(mid):
            right = mid  # Try to find a smaller capacity
        else:
            left = mid + 1  # Need a larger capacity
            
    return left
```

## Visualization 📊
```
Array: [1, 3, 5, 7, 9, 11, 13, 15, 17, 19]
Target: 11

Step 1: left=0, right=9, mid=4, arr[mid]=9 < target, so search right half
Step 2: left=5, right=9, mid=7, arr[mid]=15 > target, so search left half
Step 3: left=5, right=6, mid=5, arr[mid]=11 == target, return mid=5
```

## Common Mistakes ⚠️
1. **Off-by-one errors**: Incorrect handling of array indices
2. **Infinite loops**: Improper updating of left and right pointers
3. **Integer overflow**: Using `(left + right) / 2` instead of `left + (right - left) // 2`
4. **Not handling duplicates**: Special care needed when array has duplicates
5. **Forgetting the sorted requirement**: Binary search only works on sorted arrays

## Advantages and Disadvantages ⚖️

### Advantages ✅
- Very efficient for large datasets (O(log n) vs O(n) for linear search)
- Works well with arrays that support random access
- Minimal space requirements for iterative implementation
- Can be adapted for various related problems

### Disadvantages ❌
- Requires a sorted array (sorting may add O(n log n) overhead)
- Not efficient for small arrays (linear search may be faster)
- Not suitable for linked lists (no random access)
- Requires careful implementation to avoid off-by-one errors

## Real-world Applications 🌐
- Database indexing and searching
- Finding entries in a phone book or dictionary
- Debugging with git bisect
- Machine learning algorithms (e.g., finding optimal hyperparameters)
- Computer graphics (e.g., ray tracing)
- Network routing algorithms

## Further Reading 📚
- [Binary Search Algorithm - Khan Academy](https://www.khanacademy.org/computing/computer-science/algorithms/binary-search/a/binary-search)
- [Binary Search - GeeksforGeeks](https://www.geeksforgeeks.org/binary-search/)
- [Binary Search Visualization - VisuAlgo](https://visualgo.net/en/binarysearch)
- Book: "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein (Chapter on Binary Search)
- Book: "Algorithms" by Robert Sedgewick and Kevin Wayne

---