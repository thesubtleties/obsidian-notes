# DSA Redundant Patterns - Python Learning Sheet

## 1. Boolean Logic Redundancies

### ❌ Redundant Boolean Returns
```python
# DON'T
def is_even(n):
    if n % 2 == 0:
        return True
    else:
        return False

def is_palindrome(s):
    return True if s == s[::-1] else False
```

### ✅ Clean Boolean Returns
```python
# DO
def is_even(n):
    return n % 2 == 0

def is_palindrome(s):
    return s == s[::-1]
```

### ❌ Redundant Boolean Comparisons
```python
# DON'T
if is_valid == True:
    process()

if found == False:
    search_again()
```

### ✅ Direct Boolean Usage
```python
# DO
if is_valid:
    process()

if not found:
    search_again()
```

## 2. Loop and Iteration Redundancies

### ❌ Unnecessary Index Tracking
```python
# DON'T - Manual index when you don't need it
arr = [1, 2, 3, 4, 5]
i = 0
for item in arr:
    print(f"Index {i}: {item}")
    i += 1
```

### ✅ Use enumerate() When Needed
```python
# DO - Use enumerate when you need index
for i, item in enumerate(arr):
    print(f"Index {i}: {item}")

# DO - Direct iteration when index not needed
for item in arr:
    print(item)
```

### ❌ Redundant Range with len()
```python
# DON'T
arr = [1, 2, 3, 4, 5]
for i in range(len(arr)):
    print(arr[i])
```

### ✅ Direct Iteration
```python
# DO
for item in arr:
    print(item)

# ONLY use range(len()) when you need to modify the list
for i in range(len(arr)):
    arr[i] *= 2  # This is valid use case
```

### ❌ Manual List Building
```python
# DON'T
def get_squares(numbers):
    result = []
    for num in numbers:
        result.append(num ** 2)
    return result

def filter_evens(numbers):
    result = []
    for num in numbers:
        if num % 2 == 0:
            result.append(num)
    return result
```

### ✅ List Comprehensions
```python
# DO
def get_squares(numbers):
    return [num ** 2 for num in numbers]

def filter_evens(numbers):
    return [num for num in numbers if num % 2 == 0]
```

## 3. Search and Find Redundancies

### ❌ Linear Search in Sorted Arrays
```python
# DON'T - O(n) when you could do O(log n)
def find_in_sorted(arr, target):
    for i, val in enumerate(arr):
        if val == target:
            return i
    return -1
```

### ✅ Binary Search
```python
# DO - Use binary search
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
```

### ❌ Redundant Existence Checks
```python
# DON'T
def has_duplicates(arr):
    seen = set()
    for item in arr:
        if item in seen:
            return True
        seen.add(item)
    return False

# DON'T
if target in arr:
    index = arr.index(target)
else:
    index = -1
```

### ✅ Pythonic Approaches
```python
# DO - More Pythonic
def has_duplicates(arr):
    return len(arr) != len(set(arr))

# DO
try:
    index = arr.index(target)
except ValueError:
    index = -1
```

## 4. Data Structure Redundancies

### ❌ Wrong Data Structure Choice
```python
# DON'T - Using list for membership testing
def count_unique_chars(s):
    unique = []
    for char in s:
        if char not in unique:  # O(n) operation!
            unique.append(char)
    return len(unique)

# DON'T - Using list when order doesn't matter
def find_common_elements(list1, list2):
    common = []
    for item in list1:
        if item in list2 and item not in common:
            common.append(item)
    return common
```

### ✅ Appropriate Data Structures
```python
# DO - Use set for O(1) membership
def count_unique_chars(s):
    return len(set(s))

# DO - Use set operations
def find_common_elements(list1, list2):
    return list(set(list1) & set(list2))
```

### ❌ Redundant Dictionary Operations
```python
# DON'T
def count_frequency(arr):
    freq = {}
    for item in arr:
        if item in freq:
            freq[item] += 1
        else:
            freq[item] = 1
    return freq
```

### ✅ Efficient Dictionary Usage
```python
# DO - Use get() or defaultdict
from collections import defaultdict, Counter

def count_frequency(arr):
    freq = defaultdict(int)
    for item in arr:
        freq[item] += 1
    return dict(freq)

# OR
def count_frequency(arr):
    freq = {}
    for item in arr:
        freq[item] = freq.get(item, 0) + 1
    return freq

# BEST - Use Counter
def count_frequency(arr):
    return dict(Counter(arr))
```

## 5. String Processing Redundancies

### ❌ Manual String Building
```python
# DON'T
def reverse_words(s):
    words = s.split()
    result = ""
    for i in range(len(words) - 1, -1, -1):
        result += words[i]
        if i > 0:
            result += " "
    return result

# DON'T
def remove_vowels(s):
    result = ""
    vowels = "aeiouAEIOU"
    for char in s:
        if char not in vowels:
            result += char
    return result
```

### ✅ Efficient String Operations
```python
# DO
def reverse_words(s):
    return " ".join(s.split()[::-1])

# DO
def remove_vowels(s):
    vowels = "aeiouAEIOU"
    return "".join(char for char in s if char not in vowels)
```

## 6. Algorithm Pattern Redundancies

### ❌ Redundant Two-Pointer Setup
```python
# DON'T - Unnecessary complexity
def is_palindrome(s):
    cleaned = ""
    for char in s:
        if char.isalnum():
            cleaned += char.lower()
    
    left, right = 0, len(cleaned) - 1
    while left < right:
        if cleaned[left] != cleaned[right]:
            return False
        left += 1
        right -= 1
    return True
```

### ✅ Direct Comparison
```python
# DO - Direct comparison
def is_palindrome(s):
    cleaned = "".join(char.lower() for char in s if char.isalnum())
    return cleaned == cleaned[::-1]
```

### ❌ Redundant Sorting
```python
# DON'T - Sort when you only need min/max
def find_min_max(arr):
    sorted_arr = sorted(arr)
    return sorted_arr[0], sorted_arr[-1]

# DON'T - Sort for counting
def count_inversions_naive(arr):
    sorted_arr = sorted(arr)
    # ... complex comparison logic
```

### ✅ Appropriate Algorithms
```python
# DO
def find_min_max(arr):
    return min(arr), max(arr)

# DO - Use appropriate algorithm
def count_inversions(arr):
    # Merge sort based approach - O(n log n)
    pass
```

## 7. Memory and Performance Redundancies

### ❌ Unnecessary Space Usage
```python
# DON'T - Store all results when you only need count
def count_valid_items(arr, condition):
    valid_items = []
    for item in arr:
        if condition(item):
            valid_items.append(item)
    return len(valid_items)

# DON'T - Create intermediate lists
def process_data(data):
    step1 = [x * 2 for x in data]
    step2 = [x + 1 for x in step1]
    step3 = [x for x in step2 if x > 10]
    return step3
```

### ✅ Memory Efficient Approaches
```python
# DO
def count_valid_items(arr, condition):
    return sum(1 for item in arr if condition(item))

# DO - Chain operations
def process_data(data):
    return [x * 2 + 1 for x in data if x * 2 + 1 > 10]
```

## 8. Quick Reference: Common Replacements

| ❌ Redundant Pattern | ✅ Better Approach |
|---------------------|-------------------|
| `if condition: return True else: return False` | `return condition` |
| `for i in range(len(arr)): use arr[i]` | `for item in arr:` |
| `item in list` (repeated) | `item in set` |
| `manual string concatenation in loop` | `"".join()` |
| `if key in dict: dict[key] += 1 else: dict[key] = 1` | `dict[key] = dict.get(key, 0) + 1` |
| `sorted(arr)[0]` or `sorted(arr)[-1]` | `min(arr)` or `max(arr)` |
| `len(arr) != len(set(arr))` for duplicates | Direct set comparison |
| `manual list building` | List comprehension |

## 9. Performance Impact Summary

| Pattern Type | Time Complexity Impact | Space Complexity Impact |
|-------------|----------------------|------------------------|
| Wrong data structure | O(n) → O(1) for lookups | Often same |
| Redundant sorting | O(n log n) → O(n) | Often same |
| Manual iteration | Same time, more code | Often same |
| String concatenation | O(n²) → O(n) | Often same |
| Redundant searches | O(n) → O(log n) or O(1) | Often same |

Remember: The goal isn't just shorter code, but more readable, maintainable, and efficient code!