## If Statements
```python
# Basic if-elif-else
age = 20
if age < 13:
    print("Child")
elif age < 20:
    print("Teenager")
else:
    print("Adult")

# Ternary operator (one-line if-else)
status = "adult" if age >= 18 else "minor"

# Multiple conditions
score = 85
if score >= 90 and score <= 100:    # Can also use: 90 <= score <= 100
    print("A grade")
elif score >= 80:
    print("B grade")

# Truthy/Falsy checks
items = []
if items:    # Checks if list is not empty
    print("List has items")
```
Python's if statements are cleaner than JavaScript's, with no parentheses needed and using `elif` instead of `else if`.

## For Loops
```python
# Basic for loop with range
for i in range(5):    # 0 to 4
    print(i)

# Range with start, stop, step
for i in range(2, 10, 2):    # 2, 4, 6, 8
    print(i)

# Looping over sequences
colors = ["red", "green", "blue"]
for color in colors:
    print(color)

# Enumerate for index and value
for index, color in enumerate(colors):
    print(f"Color {index}: {color}")

# Loop over dictionary
person = {"name": "John", "age": 30}
for key in person:
    print(f"{key}: {person[key]}")

for key, value in person.items():
    print(f"{key}: {value}")
```
Python's for loops are more like JavaScript's `for...of` loops. There's no C-style `for(;;)` loop.

## While Loops
```python
# Basic while loop
count = 0
while count < 5:
    print(count)
    count += 1

# While with else (runs if no break)
num = 0
while num < 5:
    if num \=\= 3:
        break
    num += 1
else:
    print("Loop completed normally")

# Infinite loop with break
while True:
    response = input("Continue? (y/n): ")
    if response.lower() \=\= 'n':
        break
```
While loops are similar to JavaScript but can have an `else` clause that executes when the loop completes normally.

## Break, Continue, and Pass
```python
# Break - exit loop
for i in range(10):
    if i \=\= 5:
        break
    print(i)    # Prints 0-4

# Continue - skip iteration
for i in range(5):
    if i \=\= 2:
        continue
    print(i)    # Prints 0,1,3,4

# Pass - do nothing (placeholder)
for i in range(5):
    if i \=\= 2:
        pass    # Placeholder for future code
    print(i)    # Prints all numbers
```
`break` and `continue` work like in JavaScript. `pass` is unique to Python, used as a placeholder.

## Match Statement (Python 3.10+)
```python
# Basic match (similar to switch in other languages)
status = "error"
match status:
    case "success":
        print("Operation successful")
    case "error":
        print("An error occurred")
    case _:    # Default case
        print("Unknown status")

# Pattern matching with conditions
command = ("move", 10, 20)
match command:
    case ("move", x, y):
        print(f"Moving to {x},{y}")
    case ("quit" | "exit"):
        print("Exiting")
    case _:
        print("Unknown command")
```
Match statements are Python's newer, more powerful version of switch statements.

## Loop Else Clauses
```python
# For loop with else
for num in range(2, 10):
    for x in range(2, num):
        if num % x \=\= 0:
            break
    else:
        print(f"{num} is prime")

# While loop with else
count = 0
while count < 3:
    print(count)
    count += 1
else:
    print("Loop completed!")
```
The `else` clause in loops is unique to Python - it runs if the loop completes without a `break`.

## Exception Control Flow
```python
# Try-except-else-finally
try:
    num = int(input("Enter a number: "))
except ValueError:
    print("That's not a valid number!")
else:
    print(f"You entered {num}")
finally:
    print("Execution completed")

# Multiple exception types
try:
    result = 10 / num
except ZeroDivisionError:
    print("Can't divide by zero!")
except (TypeError, ValueError):
    print("Invalid operation!")
```
Python's exception handling is more granular than JavaScript's try/catch.

## Common Gotchas
```python
# Loop variable scope
for i in range(3):
    pass
print(i)    # Still accessible! (unlike let in JS)

# Else in loops vs if
# Loop else runs when no break occurs
# If else runs when condition is false

# Forgetting that range end is exclusive
for i in range(5):    # 0 to 4, not 5
    print(i)
```

## Best Practices
- Use positive conditions when possible (`if is_valid` vs `if not is_invalid`)
- Avoid deep nesting - consider early returns
- Use `for` loops when number of iterations is known
- Use `while` loops for conditional iteration
- Consider using `else` in loops for cleaner code
- Use exception handling for exceptional cases, not control flow
- Keep loop bodies simple and focused

This covers the main control flow structures in Python. Each has its use cases and idioms that might be different from JavaScript, especially regarding loop behavior and exception handling.