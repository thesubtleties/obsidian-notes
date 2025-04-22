## Definition 📝
Bit manipulation is the act of algorithmically manipulating bits or other pieces of data shorter than a byte. It involves applying logical operations on binary representations of numbers to achieve various computational tasks efficiently.

Bit manipulation leverages the binary nature of computers to perform operations that would otherwise require more complex arithmetic or conditional logic.

## Bitwise Operators ⚙️

| Operator | Symbol | Description |
|----------|--------|-------------|
| AND | `&` | Sets each bit to 1 if both bits are 1 |
| OR | `\|` | Sets each bit to 1 if at least one bit is 1 |
| XOR | `^` | Sets each bit to 1 if only one bit is 1 |
| NOT | `~` | Inverts all the bits |
| Left Shift | `<<` | Shifts bits left, filling with 0s on the right |
| Right Shift | `>>` | Shifts bits right (logical or arithmetic depending on language) |

## Basic Operations 🔍

### Getting and Setting Bits
```python
# Check if the nth bit is set (1)
def is_bit_set(num, n):
    return (num & (1 << n)) != 0

# Set the nth bit
def set_bit(num, n):
    return num | (1 << n)

# Clear the nth bit
def clear_bit(num, n):
    return num & ~(1 << n)

# Toggle the nth bit
def toggle_bit(num, n):
    return num ^ (1 << n)
```

### Counting Bits
```python
# Count set bits (Brian Kernighan's Algorithm)
def count_set_bits(num):
    count = 0
    while num:
        num &= (num - 1)  # Clear the least significant set bit
        count += 1
    return count

# Built-in functions in some languages
# Python: bin(num).count('1')
# Java: Integer.bitCount(num)
```

### Checking Powers of Two
```python
# Check if a number is a power of 2
def is_power_of_two(num):
    return num > 0 and (num & (num - 1)) == 0
    
# Alternative method
def is_power_of_two_alt(num):
    return num > 0 and (num & -num) == num
```

## Common Bit Manipulation Tricks 🎩

### Get the Rightmost Set Bit
```python
rightmost_set_bit = num & -num  # or num & (~num + 1)
```

### Turn Off the Rightmost Set Bit
```python
with_rightmost_bit_off = num & (num - 1)
```

### Isolate the Rightmost 0 Bit
```python
rightmost_zero_bit = ~num & (num + 1)
```

### Check if Numbers Have Opposite Signs
```python
have_opposite_signs = (x ^ y) < 0  # Works in languages with signed integers
```

## Bit Masking Techniques 🎭

### Creating Masks
```python
# Create a mask with n rightmost bits set to 1
def create_mask(n):
    return (1 << n) - 1

# Example: 0b00001111 = create_mask(4)
```

### Using Masks for Extraction
```python
# Extract bits from position i to j (inclusive)
def extract_bits(num, i, j):
    mask = ((1 << (j - i + 1)) - 1) << i
    return (num & mask) >> i
```

### Using Masks for Insertion
```python
# Insert m into n at positions i through j
def insert_bits(n, m, i, j):
    # Create a mask with 0s at positions i through j
    mask = ~(((1 << (j - i + 1)) - 1) << i)
    # Clear bits in n
    n_cleared = n & mask
    # Shift m into position
    m_shifted = m << i
    # Combine
    return n_cleared | m_shifted
```

## Practical Applications 🚀

### Compact Data Storage
```python
# Using bits as flags (e.g., permissions in a file system)
READ = 1      # 0b001
WRITE = 2     # 0b010
EXECUTE = 4   # 0b100

# Grant permissions
permissions = 0
permissions |= READ | WRITE  # permissions = 0b011

# Check permissions
can_read = (permissions & READ) != 0  # True
can_execute = (permissions & EXECUTE) != 0  # False
```

### Swapping Values
```python
# Swap two integers without a temporary variable
a ^= b
b ^= a
a ^= b
```

### Finding the Missing Number
```python
# Find the missing number in an array containing n-1 numbers from 1 to n
def find_missing(arr, n):
    xor_all = 0
    for i in range(1, n+1):
        xor_all ^= i
    
    for num in arr:
        xor_all ^= num
    
    return xor_all
```

## Complexity Analysis ⏱️
Most bit manipulation operations have:
- **Time Complexity**: O(1) or O(log n) depending on the operation
- **Space Complexity**: O(1)

This makes bit manipulation extremely efficient for certain problems.

## Advantages and Disadvantages ⚖️

### Advantages ✅
- Extremely efficient for certain operations
- Reduces memory usage for storing boolean flags
- Can simplify complex arithmetic operations
- Useful in low-level programming and embedded systems
- Essential for cryptography and compression algorithms

### Disadvantages ❌
- Code can be harder to read and maintain
- Behavior may vary across programming languages
- Potential for subtle bugs due to sign extension or bit width issues
- Not as intuitive as standard arithmetic operations
- Can be overkill for simple problems with no performance constraints

## Common Bit Manipulation Problems 📋
1. Count number of set bits in an integer
2. Find the single number in an array where all other numbers appear twice
3. Generate all possible subsets of a set
4. Reverse bits of an integer
5. Count number of bit flips needed to convert one number to another

## Implementation Examples 💻

### Finding a Single Number (All Others Appear Twice)
```python
def find_single_number(nums):
    result = 0
    for num in nums:
        result ^= num
    return result
```

### Generating All Subsets of a Set
```python
def generate_subsets(nums):
    n = len(nums)
    subsets = []
    
    # 2^n possible subsets
    for i in range(1 << n):
        subset = []
        for j in range(n):
            # Check if jth bit is set
            if i & (1 << j):
                subset.append(nums[j])
        subsets.append(subset)
    
    return subsets
```

### Reversing Bits
```python
def reverse_bits(n, bit_width=32):
    result = 0
    for i in range(bit_width):
        # Shift result left and add the rightmost bit of n
        result = (result << 1) | (n & 1)
        n >>= 1
    return result
```

## Further Reading 📚

### Books
- "Hacker's Delight" by Henry S. Warren, Jr.
- "Programming Pearls" by Jon Bentley
- "Bit Twiddling Hacks" by Sean Eron Anderson

### Online Resources
- [Bit Twiddling Hacks](https://graphics.stanford.edu/~seander/bithacks.html)
- [GeeksforGeeks Bit Manipulation](https://www.geeksforgeeks.org/bitwise-algorithms/)
- [Bit Manipulation on LeetCode](https://leetcode.com/tag/bit-manipulation/)
- [Bitwise Operators in C/C++](https://www.cprogramming.com/tutorial/bitwise_operators.html)

### Interactive Learning
- [Visualgo - Bitmask](https://visualgo.net/en/bitmask)
- [BitHacks - Interactive Bit Manipulation](https://bits.stephan-brumme.com/)

---