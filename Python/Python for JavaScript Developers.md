## Basic Syntax Comparison
```python
# Python                    # JavaScript
name = "John"              # let name = "John";
age = 30                   # let age = 30;
is_active = True           # let isActive = true;
CONSTANT = "FIXED"         # const CONSTANT = "FIXED";

# Multi-line strings
text = """                 # text = `
Multiple                   #   Multiple
lines                      #   lines
"""                        # `;
```

## Variables & Data Types
```python
# Dynamic typing (like JS) but with optional type hints
name: str = "John"         # let name = "John";
age: int = 30             # let age = 30;

# Python's None vs JavaScript's null/undefined
none_value = None         # let nullValue = null;

# Type checking
isinstance(name, str)     # typeof name === "string"
type(name)               # Returns <class 'str'>
```

## Collections (vs JS Arrays/Objects)
```python
# Lists (similar to JS arrays)
fruits = ["apple", "banana"]   # let fruits = ["apple", "banana"];
fruits.append("orange")        # fruits.push("orange");
fruits.pop()                   # fruits.pop();
first = fruits[0]             # let first = fruits[0];

# Dictionaries (similar to JS objects)
person = {                    # let person = {
    "name": "John",          #   name: "John",
    "age": 30               #   age: 30
}                           # };

# Sets (unique values)
unique_nums = {1, 2, 3}     # let uniqueNums = new Set([1, 2, 3])

# Tuples (immutable lists)
coordinates = (1, 2)        # No direct JS equivalent
```

## Functions
```python
# Basic function
def greet(name):            # function greet(name) {
    return f"Hello {name}"  #   return `Hello ${name}`;
                           # }

# Default parameters
def greet(name="World"):    # function greet(name = "World") {
    return f"Hello {name}"  #   return `Hello ${name}`;
                           # }

# Lambda functions (arrow functions in JS)
add = lambda x, y: x + y    # const add = (x, y) => x + y;

# *args and **kwargs (rest parameters in JS)
def func(*args, **kwargs):  # function func(...args) {
    print(args)            #   console.log(args);
    print(kwargs)          # }
```

## String Operations
```python
# f-strings (template literals in JS)
name = "John"
msg = f"Hello {name}"      # msg = `Hello ${name}`;

# String methods
text = "hello"
text.upper()               # text.toUpperCase()
text.strip()              # text.trim()
",".join(["a", "b"])      # ["a", "b"].join(",")
"hello".split("l")        # "hello".split("l")
```

## Control Flow
```python
# If statements
if x > 0:                 # if (x > 0) {
    print("Positive")     #   console.log("Positive");
elif x < 0:              # } else if (x < 0) {
    print("Negative")    #   console.log("Negative");
else:                    # } else {
    print("Zero")        #   console.log("Zero");
                        # }

# Loops
for item in items:       # for (let item of items) {
    print(item)         #   console.log(item);
                       # }

# While loops
while condition:        # while (condition) {
    do_something()     #   doSomething();
                      # }

# List comprehension (map/filter in JS)
squares = [x**2 for x in range(5)]          # squares = Array.from({length: 5}, (_, i) => i**2);
evens = [x for x in range(10) if x % 2==0]  # evens = Array.from({length: 10}).filter((_, i) => i % 2 === 0);
```

## Common Built-in Functions
```python
# Python                  # JavaScript
len(sequence)            # sequence.length
print("Hello")           # console.log("Hello")
range(5)                 # Array.from({length: 5}, (_, i) => i)
enumerate(items)         # items.entries()
```

# Key Differences from JavaScript

## Code Blocks
```python
# Python (uses indentation)
def example():
    if condition:
        do_something()
        for item in items:
            process(item)
```
```javascript
// JavaScript (uses braces)
function example() {
    if (condition) {
        doSomething();
        for (let item of items) {
            process(item);
        }
    }
}
```

## Variable Declaration
```python
# Python
name = "John"          # No declaration keyword needed
CONSTANT = "FIXED"     # Constants are by convention (uppercase)
```
```javascript
// JavaScript
let name = "John";     // Must use let/var/const
const CONSTANT = "FIXED";
```

## Boolean Values
```python
# Python
is_valid = True
is_active = False
```
```javascript
// JavaScript
let isValid = true;
let isActive = false;
```

## Null/None
```python
# Python
value = None
```
```javascript
// JavaScript
let value = null;
let undefined_value = undefined;
```

## Equality Comparison
```python
# Python
x == y    # Value equality
x is y    # Identity comparison (like === for objects)
```
```javascript
// JavaScript
x == y     // Loose equality
x === y    // Strict equality
```

## Increment/Decrement
```python
# Python
count += 1    # Increment
count -= 1    # Decrement
```
```javascript
// JavaScript
count++;      // Increment
count--;      // Decrement
```

## List/Array Slicing
```python
# Python
numbers = [0, 1, 2, 3, 4]
subset = numbers[1:4]     # [1, 2, 3]
reversed = numbers[::-1]  # [4, 3, 2, 1, 0]
step_by_two = numbers[::2]  # [0, 2, 4]
```
```javascript
// JavaScript
// No direct equivalent, would need:
numbers.slice(1, 4);
numbers.reverse();
numbers.filter((_, i) => i % 2 === 0);
```

## Multiple Assignment/Destructuring
```python
# Python
a, b = [1, 2]              # Tuple unpacking
x, *rest = [1, 2, 3, 4]    # Rest unpacking
```
```javascript
// JavaScript
let [a, b] = [1, 2];       // Array destructuring
let [x, ...rest] = [1, 2, 3, 4];
```


## Common Gotchas for JS Developers
```python
# Mutable default arguments
def bad(items=[]):        # Creates shared list across calls
def good(items=None):     # Better practice
    items = items or []

# Variable scope
x = 1
def func():
    x = 2        # Creates new local variable
    global x     # Needed to modify global variable

# Copy vs Reference
list2 = list1.copy()      # Shallow copy
import copy
list3 = copy.deepcopy(list1)  # Deep copy
```

## Common Python Idioms
```python
# Context managers
with open("file.txt") as f:
    content = f.read()

# List/dict comprehensions
squares = {x: x**2 for x in range(5)}

# Unpacking
first, *rest = [1, 2, 3, 4]

# Truthiness checking
if value:                  # Instead of if value != None
if value is not None:     # Explicit None check
```