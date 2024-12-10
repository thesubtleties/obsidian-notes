## For Loops

### Basic For Loops
```python
# Iterate over sequence
for item in sequence:
    print(item)

# Range
for i in range(5):         # 0 to 4
for i in range(1, 5):      # 1 to 4
for i in range(0, 10, 2):  # 0,2,4,6,8 (step by 2)

# Enumerate (get index and value)
for index, value in enumerate(items):
    print(f"{index}: {value}")

# Enumerate with start
for index, value in enumerate(items, start=1):
    print(f"{index}: {value}")
```

### Loop Over Data Structures
```python
# Lists
for item in list:
    print(item)

# Dictionaries
for key in dict:                    # Keys only
for value in dict.values():         # Values only
for key, value in dict.items():     # Both

# Sets
for item in set:
    print(item)

# Strings (by character)
for char in string:
    print(char)
```

## While Loops
```python
# Basic while
while condition:
    do_something()

# With counter
count = 0
while count < 5:
    print(count)
    count += 1

# Infinite loop with break
while True:
    if condition:
        break
```

## Control Flow
```python
# Break - exit loop
for item in items:
    if condition:
        break  # Exit loop completely

# Continue - skip to next iteration
for item in items:
    if condition:
        continue  # Skip rest of this iteration
    print(item)

# Else in loops (runs if no break)
for item in items:
    if condition:
        break
else:
    print("Loop completed without break")
```

## Common Patterns
```python
# Loop with index
for i in range(len(list)):
    item = list[i]

# Zip multiple lists
for x, y in zip(list1, list2):
    print(f"x: {x}, y: {y}")

# Filter while looping
for item in items:
    if not some_condition:
        continue
    process(item)

# Loop with flag
found = False
for item in items:
    if condition:
        found = True
        break
```

## List Comprehensions
```python
# Basic comprehension
squares = [x**2 for x in range(10)]

# With condition
evens = [x for x in range(10) if x % 2 == 0]

# Nested loops
pairs = [(x,y) for x in range(2) for y in range(2)]

# Dictionary comprehension
squares_dict = {x: x**2 for x in range(5)}
```

## Common Gotchas & Tips
```python
# Modifying while iterating (DON'T)
for item in list:
    list.remove(item)  # Bad!

# Correct way to modify
items = [item for item in items if condition]
# or
items = filter(condition, items)

# Creating copies
for item in list[:]:  # Create slice copy
    list.remove(item)
```

Quick Tips:
- 🔄 Use `for` when sequence length is known
- ⭐ Use `while` for conditional loops
- 📝 Use `enumerate()` when you need index
- 🎯 List comprehensions for simple transformations
- 🚫 Don't modify sequence while iterating
- ✨ `break` exits loop, `continue` skips iteration

Remember:
- For loops are preferred for finite sequences
- While loops for uncertain number of iterations
- List comprehensions for simple transformations
- Always consider infinite loop protection
- Use else clause to detect completed loops
- Be careful modifying what you're iterating over