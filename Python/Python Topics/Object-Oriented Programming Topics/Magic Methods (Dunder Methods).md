## Basic Magic Methods
```python
class Book:
    def __init__(self, title: str, price: float):
        # Constructor (like constructor() in JS)
        self.title = title
        self.price = price
    
    def __str__(self) -> str:
        # Called when using str() or print() (like toString() in JS)
        return f"{self.title} - ${self.price}"
    
    def __repr__(self) -> str:
        # Official string representation (for debugging)
        # Should ideally be able to recreate the object
        return f"Book(title='{self.title}', price={self.price})"
    
    def __len__(self) -> int:
        # Called when using len() function
        return len(self.title)

# Usage
book = Book("Python 101", 29.99)
print(book)          # Calls __str__
print(repr(book))    # Calls __repr__
print(len(book))     # Calls __len__
```
Magic (or dunder = double underscore) methods are Python's way of implementing operator overloading and special behaviors. They're similar to JavaScript's Symbol methods but more extensive.

## Comparison Magic Methods
```python
class Temperature:
    def __init__(self, celsius: float):
        self.celsius = celsius
    
    def __eq__(self, other: 'Temperature') -> bool:
        # Equality (==)
        if not isinstance(other, Temperature):
            return NotImplemented
        return self.celsius == other.celsius
    
    def __lt__(self, other: 'Temperature') -> bool:
        # Less than (<)
        if not isinstance(other, Temperature):
            return NotImplemented
        return self.celsius < other.celsius
    
    def __le__(self, other: 'Temperature') -> bool:
        # Less than or equal (<=)
        return self < other or self == other

    # Python automatically provides __gt__, __ge__ if you define __lt__ and __eq__

# Usage
t1 = Temperature(20)
t2 = Temperature(25)
print(t1 < t2)      # True
print(t1 == t2)     # False
```
These methods allow custom objects to be compared using standard comparison operators. In JavaScript, you'd typically need to create separate comparison methods.

## Numeric Magic Methods
```python
class Vector:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y
    
    def __add__(self, other: 'Vector') -> 'Vector':
        # Addition (+)
        if not isinstance(other, Vector):
            return NotImplemented
        return Vector(self.x + other.x, self.y + other.y)
    
    def __sub__(self, other: 'Vector') -> 'Vector':
        # Subtraction (-)
        if not isinstance(other, Vector):
            return NotImplemented
        return Vector(self.x - other.x, self.y - other.y)
    
    def __mul__(self, scalar: float) -> 'Vector':
        # Multiplication (*)
        if not isinstance(scalar, (int, float)):
            return NotImplemented
        return Vector(self.x * scalar, self.y * scalar)
    
    def __truediv__(self, scalar: float) -> 'Vector':
        # Division (/)
        if not isinstance(scalar, (int, float)):
            return NotImplemented
        return Vector(self.x / scalar, self.y / scalar)

# Usage
v1 = Vector(1, 2)
v2 = Vector(3, 4)
v3 = v1 + v2        # Calls __add__
v4 = v1 * 2         # Calls __mul__
```
These methods allow objects to work with arithmetic operators. JavaScript doesn't have built-in operator overloading.

## Container Magic Methods
```python
class Deck:
    def __init__(self):
        self.cards = ['A', 'K', 'Q', 'J']
    
    def __getitem__(self, index: int) -> str:
        # Access with square brackets (like array access)
        return self.cards[index]
    
    def __setitem__(self, index: int, value: str) -> None:
        # Set with square brackets
        self.cards[index] = value
    
    def __delitem__(self, index: int) -> None:
        # Delete with del keyword
        del self.cards[index]
    
    def __contains__(self, item: str) -> bool:
        # Membership test (in operator)
        return item in self.cards
    
    def __iter__(self):
        # Make object iterable (for loops)
        return iter(self.cards)

# Usage
deck = Deck()
print(deck[0])      # 'A' (calls __getitem__)
deck[0] = '2'       # Calls __setitem__
del deck[0]         # Calls __delitem__
print('K' in deck)  # Calls __contains__
for card in deck:   # Calls __iter__
    print(card)
```
These methods allow objects to behave like containers (lists, dictionaries). Similar to implementing Iterator protocol in JavaScript but more comprehensive.

## Context Manager Magic Methods
```python
class FileHandler:
    def __init__(self, filename: str):
        self.filename = filename
        self.file = None
    
    def __enter__(self):
        # Called when entering 'with' block
        self.file = open(self.filename, 'r')
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        # Called when exiting 'with' block
        if self.file:
            self.file.close()
        # Return True to suppress exceptions
        return False

# Usage
with FileHandler('data.txt') as file:
    content = file.read()
# File is automatically closed after block
```
These methods implement the context manager protocol, similar to JavaScript's `with` statement (which is less common). Great for resource management.

## Callable Magic Method
```python
class Multiplier:
    def __init__(self, factor: float):
        self.factor = factor
    
    def __call__(self, x: float) -> float:
        # Makes instance callable like a function
        return x * self.factor

# Usage
double = Multiplier(2)
print(double(5))    # 10 (calls __call__)
```
Makes an object callable like a function. Similar to JavaScript's Proxy with apply handler, but more straightforward.

## Property Access Magic Methods
```python
class Proxy:
    def __init__(self, obj):
        self._obj = obj
    
    def __getattr__(self, name: str):
        # Called for missing attributes
        print(f"Accessing {name}")
        return getattr(self._obj, name)
    
    def __setattr__(self, name: str, value):
        # Called when setting attributes
        print(f"Setting {name} = {value}")
        super().__setattr__(name, value)
    
    def __delattr__(self, name: str):
        # Called when deleting attributes
        print(f"Deleting {name}")
        super().__delattr__(name)

# Usage
class Person:
    def __init__(self, name: str):
        self.name = name

p = Proxy(Person("John"))
print(p.name)       # Calls __getattr__
p.age = 30          # Calls __setattr__
del p.age           # Calls __delattr__
```
These methods intercept attribute access, similar to JavaScript's Proxy but with more specific control.

## Understanding Magic Methods
```python
# Python automatically looks for these specific method names
class ExampleClass:
    # Built-in magic methods that Python recognizes
    def __init__(self, value):
        self.value = value    # Constructor
    
    def __str__(self):
        return str(self.value)  # Called by str() and print()
    
    # Regular method - NOT a magic method
    def __my_custom_thing__(self):
        return "Not magic!"

# Python knows when to call magic methods based on syntax:
example = ExampleClass(42)
print(example)      # Python calls __str__ automatically

# This syntax...     # Calls this magic method...
x + y               # x.__add__(y)
x['key']            # x.__getitem__('key')
len(x)              # x.__len__()
x()                 # x.__call__()
x < y               # x.__lt__(y)

# Example showing magic vs regular methods
class ShoppingCart:
    def __init__(self):
        self.items = []
    
    # Magic method - Python uses this for len()
    def __len__(self):
        return len(self.items)
    
    # Regular method - we have to call it explicitly
    def get_count(self):
        return len(self.items)

cart = ShoppingCart()
print(len(cart))            # Uses __len__
print(cart.get_count())     # Regular method call
```
Magic methods are predefined hooks that Python looks for to implement specific behaviors. You can't create new magic methods - you can only implement the ones Python recognizes. Think of them like special interfaces that Python knows about and calls automatically based on how you use your objects. The double underscores (`__`) are Python's way of saying "this is a special method that Python should look for."

## Best Practices
```python
class BestPractices:
    def __init__(self, value):
        self.value = value
    
    def __repr__(self) -> str:
        # Always implement __repr__
        return f"{self.__class__.__name__}({self.value!r})"
    
    def __str__(self) -> str:
        # Implement __str__ for human-readable output
        return str(self.value)
    
    def __eq__(self, other) -> bool:
        # Type check in comparison methods
        if not isinstance(other, type(self)):
            return NotImplemented
        return self.value == other.value
    
    def __bool__(self) -> bool:
        # Implement truthiness
        return bool(self.value)
```
Key differences from JavaScript:
1. Python has many more special methods than JavaScript
2. Magic methods provide operator overloading (not possible in JavaScript)
3. More structured approach to object behavior customization
4. Built-in support for context managers and container protocols
5. More explicit control over object behavior

These magic methods are a key feature of Python, allowing objects to integrate seamlessly with Python's built-in operations and syntax.