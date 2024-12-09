1. Conditional Syntax
   - JavaScript: `if(isGood) { ... }` or `isGood && doSomething()`
   - Python: Must use `if is_good:` (no shorthand evaluations)

2. String Interpolation
   - JavaScript: `${variable}` with backticks
   - Python: Must include `f` prefix: `f"{variable}"`
   - Python without f: `"{variable}"` will print literal "{variable}"

3. Variable Naming
   - JavaScript: camelCase (`isGood`, `privateVar`)
   - Python: snake_case (`is_good`, `private_var`)
   - Python "private": single underscore (`_private_var`)

4. Boolean Comparison
   - JavaScript: `if (isGood === true)`
   - Python: Just `if is_good:` (comparing to True/False not Pythonic)

5. Ternary Operators
   - JavaScript: `condition ? true : false`
   - Python: `true_value if condition else false_value`

6. Conditional Syntax Colons
   - JavaScript: `if (x > 10) { ... } else if (x < 0) { ... }`
   - Python: `if x > 10:` (colon, no braces)
   - Python elif: `elif x < 0:` (no colon after elif keyword)
   - Common error: `elif: x < 0:` (extra colon is wrong!)

7. Method Definitions
   - JavaScript: `class Pizza { makePizza(name) { this.name = name } }`
   - Python: `def make_pizza(self, name):` (must include self)

8. Iterating with Index
   - JavaScript: `array.forEach((item, i) => {})`
   - Python: `for index, item in enumerate(list):`
   - JavaScript: `for(let i=0; i<arr.length; i++)`
   - Python: `for i in range(len(list)):`

9. Dictionary Iteration
   - JavaScript: `Object.entries(obj).forEach()` or `for([key, val] of Object.entries)`
   - Python: `for key, value in dict.items():`
   - JavaScript: `for(key in obj)`
   - Python: `for key in dict:` (keys only) or `for key, val in dict.items():` (both)