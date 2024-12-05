## Basic Array Types
```python
# Standard array (array module)
from array import array
numbers = array('i', [1, 2, 3, 4, 5])  # Signed integer array
floats = array('d', [1.0, 2.0, 3.0])   # Double-precision float

# Common type codes
# 'i' - signed int
# 'd' - double float
# 'f' - single float
# 'b' - signed char
# 'u' - unicode char
```
Python arrays are more memory-efficient than lists for numerical data as they store only homogeneous data.

## NumPy Arrays
```python
import numpy as np

# Creating arrays
arr1 = np.array([1, 2, 3])              # 1D array
arr2 = np.array([[1, 2], [3, 4]])       # 2D array
zeros = np.zeros((3, 3))                # Array of zeros
ones = np.ones((2, 2))                  # Array of ones
range_arr = np.arange(0, 10, 2)         # Array with range
linspace = np.linspace(0, 1, 5)         # Evenly spaced numbers
```
NumPy arrays are the foundation for numerical computing in Python, offering powerful operations and better performance.

## Basic Operations with NumPy
```python
# Arithmetic operations
arr = np.array([1, 2, 3])
print(arr * 2)                # Element-wise multiplication
print(arr + 5)                # Element-wise addition
print(arr ** 2)               # Element-wise square

# Array operations
print(arr.sum())              # Sum of elements
print(arr.mean())             # Mean of elements
print(arr.max())              # Maximum value
print(arr.min())              # Minimum value
```
NumPy operations are vectorized, making them much faster than Python loops.

## Array Indexing and Slicing
```python
# Standard array
numbers = array('i', [1, 2, 3, 4, 5])
first = numbers[0]                    # Single element
subset = numbers[1:4]                 # Slice

# NumPy array
arr = np.array([[1, 2, 3], 
                [4, 5, 6]])
print(arr[0, 1])                      # Element at row 0, col 1
print(arr[:, 1])                      # All rows, column 1
print(arr[0, :])                      # Row 0, all columns
```
NumPy provides more powerful indexing capabilities than standard Python arrays.

## Array Reshaping and Dimensions
```python
import numpy as np

# Reshaping arrays
arr = np.array([1, 2, 3, 4, 5, 6])
reshaped = arr.reshape(2, 3)          # 2x3 array
flattened = reshaped.flatten()        # Back to 1D

# Array properties
print(arr.shape)                      # Array dimensions
print(arr.ndim)                       # Number of dimensions
print(arr.size)                       # Total number of elements
print(arr.dtype)                      # Data type of elements
```
NumPy arrays can be easily reshaped while maintaining their data.

## Array Operations with NumPy
```python
# Mathematical operations
arr = np.array([1, 2, 3])
print(np.sqrt(arr))                   # Square root
print(np.exp(arr))                    # Exponential
print(np.log(arr))                    # Natural logarithm

# Statistical operations
print(np.mean(arr))                   # Mean
print(np.median(arr))                 # Median
print(np.std(arr))                    # Standard deviation
print(np.var(arr))                    # Variance
```
NumPy provides a wide range of mathematical and statistical operations.

## Array Manipulation
```python
# Concatenation
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])
combined = np.concatenate([arr1, arr2])
stacked = np.vstack([arr1, arr2])     # Vertical stack
hstacked = np.hstack([arr1, arr2])    # Horizontal stack

# Splitting
arr = np.array([1, 2, 3, 4, 5, 6])
split = np.split(arr, 3)              # Split into 3 equal parts
```
Arrays can be combined and split in various ways to create new array structures.

## Broadcasting
```python
# Broadcasting rules
arr = np.array([[1, 2, 3],
                [4, 5, 6]])
print(arr + 1)                        # Add 1 to all elements
print(arr * np.array([10, 20, 30]))   # Multiply each column
```
Broadcasting allows operations between arrays of different shapes.

## Memory Views and Buffers
```python
# Working with memory views
numbers = array('i', [1, 2, 3, 4])
memory = memoryview(numbers)
print(memory[0])                      # Access through memory view

# Converting between types
bytes_data = numbers.tobytes()        # Convert to bytes
new_array = array('i')
new_array.frombytes(bytes_data)       # Create from bytes
```
Memory views provide low-level access to array data.

## Performance Considerations
```python
import time

# Compare list vs array performance
list_nums = list(range(1000000))
array_nums = array('i', list_nums)
numpy_nums = np.array(list_nums)

# Operation timing
def time_operation(func):
    start = time.time()
    func()
    return time.time() - start

# Example operations
list_time = time_operation(lambda: [x * 2 for x in list_nums])
array_time = time_operation(lambda: array_nums * 2)
numpy_time = time_operation(lambda: numpy_nums * 2)
```
Arrays and NumPy arrays are much more efficient for numerical operations.

## Error Handling
```python
# Type checking
def safe_array_operation(arr, value):
    try:
        return arr * value
    except TypeError:
        return None

# Bounds checking
def safe_get(arr, index):
    try:
        return arr[index]
    except IndexError:
        return None

# Shape validation
def validate_shape(arr, expected_shape):
    try:
        return arr.reshape(expected_shape)
    except ValueError:
        return None
```
Always handle potential errors when working with arrays.

## Best Practices
```python
# Choose the right array type
def create_appropriate_array(data):
    if all(isinstance(x, int) for x in data):
        return array('i', data)
    elif all(isinstance(x, float) for x in data):
        return array('d', data)
    else:
        return np.array(data)

# Use appropriate data types
def optimize_memory(arr):
    # Convert to smallest possible integer type
    return np.array(arr, dtype=np.min_scalar_type(arr.max()))

# Vectorize operations
def vectorized_calculation(arr):
    return np.sum(np.square(arr))  # Instead of loop
```
Choose the appropriate array type and operations for your use case.

## Common Gotchas
```python
# Copy vs View
arr = np.array([1, 2, 3])
view = arr[:]                # Creates a view
copy = arr.copy()            # Creates a copy

# Type restrictions
numbers = array('i', [1, 2, 3])
# numbers.append(3.14)       # TypeError: integer required

# Shape mismatches
def handle_shape_mismatch(arr1, arr2):
    try:
        return arr1 + arr2
    except ValueError:
        return None
```
Be aware of array views vs copies and type restrictions.

This covers the main aspects of working with arrays in Python, including both standard arrays and NumPy arrays, which are essential for numerical computing and data science applications.