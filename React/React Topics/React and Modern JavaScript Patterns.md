## 1. Destructuring in Function Parameters
```javascript
function Component({ prop1, prop2, prop3 = defaultValue }) {
  // Use prop1, prop2, prop3 directly
}
```
This pattern extracts specific properties from the object passed to the function. It allows you to directly use these properties as variables within the function, making the code cleaner and more explicit about what props are being used.

## 2. Spread Operator in JSX
```jsx
const props = { prop1: 'value1', prop2: 'value2' };
return <ChildComponent {...props} />;
```
The spread operator (...) expands an object into its individual properties. In JSX, this allows you to pass all properties of an object as individual props to a component, which is useful for prop forwarding or when you have many props to pass.

## 3. Short Circuit Evaluation
```jsx
{isLoggedIn && <UserGreeting />}
```
This uses the && operator for conditional rendering. If the left side (isLoggedIn) is true, the right side (<UserGreeting />) is rendered. If false, nothing is rendered. It's a concise way to conditionally render elements.

## 4. Ternary Operator in JSX
```jsx
{isLoggedIn ? <UserGreeting /> : <GuestGreeting />}
```
The ternary operator provides a compact way to choose between two JSX expressions based on a condition. It's like an if-else statement in a single line, often used for conditional rendering in React.

## 5. Object Property Shorthand
```javascript
const name = 'John';
const age = 30;
const user = { name, age };
```
When the variable name matches the desired object property name, you can use this shorthand. It's equivalent to { name: name, age: age }, but more concise.

## 6. Array and Object Destructuring
```javascript
const [first, second, ...rest] = array;
const { title, author, ...otherProps } = book;
```
Destructuring allows you to extract values from arrays or properties from objects into distinct variables. The rest operator (...) collects remaining elements into a new array or object.

## 7. Optional Chaining
```javascript
const authorName = book?.author?.name;
```
Optional chaining (?.) allows you to safely access nested object properties without throwing an error if an intermediate property doesn't exist. It's a safer way to access nested properties when you're unsure if they exist.

## 8. Nullish Coalescing
```javascript
const count = data ?? 0;
```
The nullish coalescing operator (??) provides a way to fallback to a default value when dealing with null or undefined. Unlike ||, it only falls back for null or undefined, not other falsy values like 0 or ''.

## 9. Template Literals
```javascript
const greeting = `Hello, ${name}!`;
```
Template literals allow you to embed expressions in strings. They use backticks (`) instead of quotes and can span multiple lines. Expressions are embedded using ${}.

## 10. Arrow Functions
```javascript
const multiply = (a, b) => a * b;
```
Arrow functions provide a concise syntax for writing function expressions. They have a shorter syntax and lexically bind 'this', making them particularly useful for callbacks and in React components.