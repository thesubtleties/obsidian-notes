# JavaScript DS&A Syntax Reminder with Efficiency Annotations

## Installation 📥
JavaScript comes pre-installed with all modern browsers. For command-line usage:
```bash
# Install Node.js
# macOS
brew install node

# Ubuntu/Debian
apt install nodejs npm

# Windows
choco install nodejs
# or
winget install OpenJS.NodeJS
```

## Basic JavaScript Syntax for DS&A 📝

### Variables and Data Types
```javascript
// Variable declarations
let variable = 10;        // Block-scoped, can be reassigned
const constant = 5;       // Block-scoped, cannot be reassigned
var oldVariable = 15;     // Function-scoped (avoid when possible)

// Primitive data types
const number = 42;        // Numbers (integers and floats)
const bigInt = 123n;      // BigInt for large integers
const text = "Hello";     // Strings
const isTrue = true;      // Booleans
const nothing = null;     // Null
const notDefined = undefined; // Undefined
const symbol = Symbol('id'); // Symbol (unique identifier)

// Complex data types
const arr = [1, 2, 3];    // Arrays
const obj = {             // Objects
  key: "value",
  number: 42
};

// Type checking
typeof 42;                // "number"
Array.isArray([1, 2, 3]); // true
Number.isInteger(42);     // true
Number.isNaN(NaN);        // true
```

### Arrays - Essential for DS&A 📊
```javascript
// Creating arrays
const arr = [1, 2, 3, 4, 5];
const empty = [];
const mixed = [1, "two", {three: 3}, [4]];
const fromString = "hello".split("");  // ["h", "e", "l", "l", "o"] - O(n)
const range = Array.from({length: 5}, (_, i) => i);  // [0, 1, 2, 3, 4] - O(n)

// Accessing elements (0-indexed)
arr[0];                   // 1 (first element) - O(1)
arr[arr.length - 1];      // 5 (last element) - O(1)
arr.at(-1);               // 5 (last element, modern syntax) - O(1)
arr.at(-2);               // 4 (second to last) - O(1)

// Common array methods
arr.push(6);              // Add to end: [1,2,3,4,5,6] - O(1)
arr.pop();                // Remove from end: [1,2,3,4,5] - O(1)
arr.unshift(0);           // Add to beginning: [0,1,2,3,4,5] - O(n)
arr.shift();              // Remove from beginning: [1,2,3,4,5] - O(n)
arr.slice(1, 3);          // Extract [1,3): [2,3] - O(k) where k is slice size
arr.splice(1, 2, 'a');    // Remove 2 elements at index 1, insert 'a': [1,'a',4,5] - O(n)

// Finding elements
arr.indexOf(3);           // 2 (first occurrence) - O(n)
arr.lastIndexOf(3);       // 2 (last occurrence) - O(n)
arr.find(x => x > 3);     // 4 (first element matching condition) - O(n)
arr.findIndex(x => x > 3); // 3 (index of first match) - O(n)
arr.includes(3);          // true - O(n)

// Iteration methods
arr.forEach(num => console.log(num));  // Iterate without creating new array - O(n)
const doubled = arr.map(num => num * 2);  // Create new array: [2,4,6,8,10] - O(n)
const evens = arr.filter(num => num % 2 === 0);  // Filter: [2,4] - O(n)
const sum = arr.reduce((total, num) => total + num, 0);  // Reduce to single value: 15 - O(n)
const product = arr.reduceRight((acc, num) => acc * num, 1);  // Right to left - O(n)

// Array flattening
const nested = [[1, 2], [3, 4], [5]];
nested.flat();            // [1, 2, 3, 4, 5] - O(n)
nested.flat(2);           // Flatten 2 levels deep - O(n)
nested.flatMap(x => x);   // Same as flat() but with map - O(n)

// Sorting and reversing
const numbers = [3, 1, 4, 1, 5];
numbers.sort((a, b) => a - b);  // [1, 1, 3, 4, 5] (ascending) - O(n log n)
numbers.sort((a, b) => b - a);  // [5, 4, 3, 1, 1] (descending) - O(n log n)
numbers.reverse();        // Reverse in place - O(n)
[...numbers].reverse();   // Reverse without mutating original - O(n)

// Array concatenation and spreading
const arr1 = [1, 2];
const arr2 = [3, 4];
arr1.concat(arr2);        // [1, 2, 3, 4] - O(n + m)
[...arr1, ...arr2];       // [1, 2, 3, 4] (spread syntax) - O(n + m)

// Array destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first = 1, second = 2, rest = [3, 4, 5]

// Array from other iterables
Array.from("hello");      // ["h", "e", "l", "l", "o"] - O(n)
Array.from(new Set([1, 2, 2, 3])); // [1, 2, 3] - O(n)
[...new Set([1, 2, 2, 3])]; // [1, 2, 3] (spread syntax) - O(n)
```

### Objects - For Hash Maps/Dictionaries 🗂️
```javascript
// Creating objects
const obj = {
  name: "Alice",
  age: 25,
  isStudent: true
};

// Accessing properties
obj.name;                 // "Alice" (dot notation) - O(1) average
obj["age"];               // 25 (bracket notation) - O(1) average
obj?.optionalProp;        // undefined (optional chaining) - O(1) average

// Adding/modifying properties
obj.location = "New York";  // O(1) average
obj["isStudent"] = false;   // O(1) average

// Checking if property exists
"name" in obj;            // true - O(1) average
obj.hasOwnProperty("age"); // true - O(1) average
obj.name !== undefined;   // true - O(1) average

// Object methods
Object.keys(obj);         // ["name", "age", "isStudent", "location"] - O(n)
Object.values(obj);       // ["Alice", 25, false, "New York"] - O(n)
Object.entries(obj);      // [["name", "Alice"], ["age", 25], ...] - O(n)
Object.fromEntries([["a", 1], ["b", 2]]); // {a: 1, b: 2} - O(n)

// Object copying and merging
const copy = {...obj};    // Shallow copy - O(n)
const merged = {...obj, newProp: "value"}; // Merge with new properties - O(n)
Object.assign({}, obj);   // Another way to shallow copy - O(n)

// Object destructuring
const {name, age, ...others} = obj;
// name = "Alice", age = 25, others = {isStudent: false, location: "New York"}

// Computed property names
const key = "dynamicKey";
const dynamicObj = {
  [key]: "value",
  [`${key}2`]: "value2"
};
```

### Strings - Critical for Many Problems 📝
```javascript
const str = "Hello World";

// Basic string operations
str.length;               // 11 - O(1)
str[0];                   // "H" - O(1)
str.charAt(0);            // "H" - O(1)
str.charCodeAt(0);        // 72 (ASCII code) - O(1)
String.fromCharCode(72);  // "H" - O(1)

// String searching
str.indexOf("o");         // 4 (first occurrence) - O(n)
str.lastIndexOf("o");     // 7 (last occurrence) - O(n)
str.includes("World");    // true - O(n)
str.startsWith("Hello");  // true - O(k) where k is prefix length
str.endsWith("World");    // true - O(k) where k is suffix length

// String manipulation
str.slice(0, 5);          // "Hello" - O(k) where k is slice length
str.substring(0, 5);      // "Hello" - O(k)
str.substr(0, 5);         // "Hello" (deprecated) - O(k)
str.split(" ");           // ["Hello", "World"] - O(n)
str.split("");            // ["H", "e", "l", "l", "o", " ", "W", "o", "r", "l", "d"] - O(n)

// String transformation
str.toLowerCase();        // "hello world" - O(n)
str.toUpperCase();        // "HELLO WORLD" - O(n)
str.trim();               // Remove whitespace from both ends - O(n)
str.trimStart();          // Remove whitespace from start - O(n)
str.trimEnd();            // Remove whitespace from end - O(n)

// String replacement
str.replace("World", "JS"); // "Hello JS" (first occurrence) - O(n)
str.replaceAll("l", "x");   // "Hexxo Worxd" (all occurrences) - O(n)
str.replace(/l/g, "x");     // "Hexxo Worxd" (regex global) - O(n)

// String repetition and padding
"abc".repeat(3);          // "abcabcabc" - O(n * k)
"5".padStart(3, "0");     // "005" - O(n)
"5".padEnd(3, "0");       // "500" - O(n)

// Template literals
const name = "Alice";
const greeting = `Hello, ${name}!`; // "Hello, Alice!" - O(n)

// String to array conversions
"hello".split("");        // ["h", "e", "l", "l", "o"] - O(n)
[..."hello"];             // ["h", "e", "l", "l", "o"] (spread) - O(n)
Array.from("hello");      // ["h", "e", "l", "l", "o"] - O(n)

// Array to string conversions
["h", "e", "l", "l", "o"].join(""); // "hello" - O(n)
["a", "b", "c"].join("-"); // "a-b-c" - O(n)
```

## Control Flow 🔄

### Conditionals
```javascript
// If statements
if (condition) {
  // code
} else if (anotherCondition) {
  // code
} else {
  // code
}

// Ternary operator
const result = condition ? "true case" : "false case";

// Nullish coalescing
const value = null ?? "default";  // "default"
const value2 = 0 ?? "default";    // 0 (only null/undefined trigger default)

// Logical operators for short-circuiting
const result1 = condition && "truthy value";  // Returns "truthy value" if condition is true
const result2 = condition || "default";       // Returns "default" if condition is falsy

// Switch statement
switch (value) {
  case 1:
    // code
    break;
  case 2:
  case 3:  // Fall-through for multiple cases
    // code
    break;
  default:
    // code
}
```

### Loops - Critical for Algorithms ⭐
```javascript
const arr = [1, 2, 3, 4, 5];

// For loop
for (let i = 0; i < arr.length; i++) {  // O(n)
  console.log(arr[i]);
}

// For loop with step
for (let i = 0; i < arr.length; i += 2) {  // O(n/2)
  console.log(arr[i]);  // Prints elements at even indices
}

// Reverse for loop
for (let i = arr.length - 1; i >= 0; i--) {  // O(n)
  console.log(arr[i]);
}

// For...of (iterates over values)
for (const element of arr) {  // O(n)
  console.log(element);
}

// For...of with index
for (const [index, element] of arr.entries()) {  // O(n)
  console.log(index, element);
}

// For...in (iterates over keys/indices)
for (const key in obj) {  // O(n)
  if (obj.hasOwnProperty(key)) {  // Good practice to check
    console.log(key, obj[key]);
  }
}

// While loop
let i = 0;
while (i < arr.length) {  // O(n)
  console.log(arr[i]);
  i++;
}

// Do-while loop
let j = 0;
do {  // O(n)
  console.log(arr[j]);
  j++;
} while (j < arr.length);

// Break and continue
for (let i = 0; i < 10; i++) {
  if (i === 3) continue;  // Skip iteration
  if (i === 7) break;     // Exit loop
  console.log(i);  // Prints: 0, 1, 2, 4, 5, 6
}

// Labeled breaks (for nested loops)
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) break outer;  // Breaks out of both loops
    console.log(i, j);
  }
}
```

## Functions 🛠️

### Function Declarations
```javascript
// Regular function
function add(a, b) {
  return a + b;
}

// Function expression
const multiply = function(a, b) {
  return a * b;
};

// Arrow function
const divide = (a, b) => a / b;
const square = x => x * x;  // Single parameter, no parentheses needed
const greet = () => "Hello!";  // No parameters

// Default parameters
function greet(name = "Guest", greeting = "Hello") {
  return `${greeting}, ${name}!`;
}

// Rest parameters
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);  // O(n)
}

// Destructuring parameters
function processUser({name, age, email = "N/A"}) {
  return `${name} (${age}) - ${email}`;
}

// Higher-order functions
function createMultiplier(factor) {
  return function(number) {
    return number * factor;
  };
}
const double = createMultiplier(2);
console.log(double(5));  // 10

// Immediately Invoked Function Expression (IIFE)
(function() {
  console.log("This runs immediately");
})();

// Recursive functions
function factorial(n) {  // O(n) time, O(n) space
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}

function fibonacciMemo(n, memo = {}) {  // O(n) time with memoization
  if (n in memo) return memo[n];
  if (n <= 2) return 1;
  memo[n] = fibonacciMemo(n - 1, memo) + fibonacciMemo(n - 2, memo);
  return memo[n];
}
```

## Common Data Structures Implementation 📚

### Stack
```javascript
class Stack {
  constructor() {
    this.items = [];
  }
  
  push(element) {  // O(1)
    this.items.push(element);
  }
  
  pop() {  // O(1)
    if (this.isEmpty()) return undefined;
    return this.items.pop();
  }
  
  peek() {  // O(1)
    if (this.isEmpty()) return undefined;
    return this.items[this.items.length - 1];
  }
  
  isEmpty() {  // O(1)
    return this.items.length === 0;
  }
  
  size() {  // O(1)
    return this.items.length;
  }
  
  clear() {  // O(1)
    this.items = [];
  }
  
  toArray() {  // O(n)
    return [...this.items];
  }
}

// Usage
const stack = new Stack();
stack.push(10);
stack.push(20);
console.log(stack.pop());  // 20
console.log(stack.peek()); // 10
```

### Queue (Optimized Implementation)
```javascript
class Queue {
  constructor() {
    this.items = {};
    this.front = 0;
    this.rear = 0;
  }
  
  enqueue(element) {  // O(1)
    this.items[this.rear] = element;
    this.rear++;
  }
  
  dequeue() {  // O(1) - much better than array.shift()
    if (this.isEmpty()) return undefined;
    const item = this.items[this.front];
    delete this.items[this.front];
    this.front++;
    return item;
  }
  
  peek() {  // O(1)
    if (this.isEmpty()) return undefined;
    return this.items[this.front];
  }
  
  isEmpty() {  // O(1)
    return this.rear === this.front;
  }
  
  size() {  // O(1)
    return this.rear - this.front;
  }
  
  toArray() {  // O(n)
    const result = [];
    for (let i = this.front; i < this.rear; i++) {
      result.push(this.items[i]);
    }
    return result;
  }
}

// Usage
const queue = new Queue();
queue.enqueue("A");
queue.enqueue("B");
console.log(queue.dequeue());  // "A"
console.log(queue.peek());     // "B"
```

### Deque (Double-ended Queue)
```javascript
class Deque {
  constructor() {
    this.items = {};
    this.front = 0;
    this.rear = 0;
  }
  
  addFront(element) {  // O(1)
    this.front--;
    this.items[this.front] = element;
  }
  
  addRear(element) {  // O(1)
    this.items[this.rear] = element;
    this.rear++;
  }
  
  removeFront() {  // O(1)
    if (this.isEmpty()) return undefined;
    const item = this.items[this.front];
    delete this.items[this.front];
    this.front++;
    return item;
  }
  
  removeRear() {  // O(1)
    if (this.isEmpty()) return undefined;
    this.rear--;
    const item = this.items[this.rear];
    delete this.items[this.rear];
    return item;
  }
  
  peekFront() {  // O(1)
    if (this.isEmpty()) return undefined;
    return this.items[this.front];
  }
  
  peekRear() {  // O(1)
    if (this.isEmpty()) return undefined;
    return this.items[this.rear - 1];
  }
  
  isEmpty() {  // O(1)
    return this.rear === this.front;
  }
  
  size() {  // O(1)
    return this.rear - this.front;
  }
}
```

### Linked List (Enhanced)
```javascript
class Node {
  constructor(data) {
    this.data = data;
    this.next = null;
  }
}

class LinkedList {
  constructor() {
    this.head = null;
    this.tail = null;  // Keep track of tail for O(1) append
    this.length = 0;
  }
  
  append(data) {  // O(1) with tail pointer
    const newNode = new Node(data);
    
    if (!this.head) {
      this.head = newNode;
      this.tail = newNode;
    } else {
      this.tail.next = newNode;
      this.tail = newNode;
    }
    this.length++;
  }
  
  prepend(data) {  // O(1)
    const newNode = new Node(data);
    newNode.next = this.head;
    this.head = newNode;
    
    if (!this.tail) {
      this.tail = newNode;
    }
    this.length++;
  }
  
  insertAt(index, data) {  // O(n)
    if (index < 0 || index > this.length) return false;
    if (index === 0) return this.prepend(data);
    if (index === this.length) return this.append(data);
    
    const newNode = new Node(data);
    let current = this.head;
    
    for (let i = 0; i < index - 1; i++) {
      current = current.next;
    }
    
    newNode.next = current.next;
    current.next = newNode;
    this.length++;
    return true;
  }
  
  delete(data) {  // O(n)
    if (!this.head) return false;
    
    if (this.head.data === data) {
      this.head = this.head.next;
      if (!this.head) this.tail = null;
      this.length--;
      return true;
    }
    
    let current = this.head;
    while (current.next && current.next.data !== data) {
      current = current.next;
    }
    
    if (current.next) {
      if (current.next === this.tail) {
        this.tail = current;
      }
      current.next = current.next.next;
      this.length--;
      return true;
    }
    
    return false;
  }
  
  find(data) {  // O(n)
    let current = this.head;
    let index = 0;
    
    while (current) {
      if (current.data === data) return index;
      current = current.next;
      index++;
    }
    
    return -1;
  }
  
  get(index) {  // O(n)
    if (index < 0 || index >= this.length) return null;
    
    let current = this.head;
    for (let i = 0; i < index; i++) {
      current = current.next;
    }
    
    return current.data;
  }
  
  toArray() {  // O(n)
    const result = [];
    let current = this.head;
    while (current) {
      result.push(current.data);
      current = current.next;
    }
    return result;
  }
  
  reverse() {  // O(n)
    let prev = null;
    let current = this.head;
    this.tail = this.head;
    
    while (current) {
      const next = current.next;
      current.next = prev;
      prev = current;
      current = next;
    }
    
    this.head = prev;
  }
  
  size() {  // O(1)
    return this.length;
  }
  
  isEmpty() {  // O(1)
    return this.length === 0;
  }
}
```

### Hash Table/Map Implementation
```javascript
class HashTable {
  constructor(size = 53) {  // Prime number for better distribution
    this.keyMap = new Array(size);
  }
  
  _hash(key) {  // O(1) - Simple hash function
    let total = 0;
    const WEIRD_PRIME = 31;
    for (let i = 0; i < Math.min(key.length, 100); i++) {
      const char = key[i];
      const value = char.charCodeAt(0) - 96;
      total = (total * WEIRD_PRIME + value) % this.keyMap.length;
    }
    return total;
  }
  
  set(key, value) {  // O(1) average
    const index = this._hash(key);
    if (!this.keyMap[index]) {
      this.keyMap[index] = [];
    }
    
    // Check if key already exists
    for (let i = 0; i < this.keyMap[index].length; i++) {
      if (this.keyMap[index][i][0] === key) {
        this.keyMap[index][i][1] = value;
        return;
      }
    }
    
    this.keyMap[index].push([key, value]);
  }
  
  get(key) {  // O(1) average
    const index = this._hash(key);
    if (this.keyMap[index]) {
      for (let i = 0; i < this.keyMap[index].length; i++) {
        if (this.keyMap[index][i][0] === key) {
          return this.keyMap[index][i][1];
        }
      }
    }
    return undefined;
  }
  
  keys() {  // O(n)
    const keysArr = [];
    for (let i = 0; i < this.keyMap.length; i++) {
      if (this.keyMap[i]) {
        for (let j = 0; j < this.keyMap[i].length; j++) {
          keysArr.push(this.keyMap[i][j][0]);
        }
      }
    }
    return keysArr;
  }
  
  values() {  // O(n)
    const valuesArr = [];
    for (let i = 0; i < this.keyMap.length; i++) {
      if (this.keyMap[i]) {
        for (let j = 0; j < this.keyMap[i].length; j++) {
          if (!valuesArr.includes(this.keyMap[i][j][1])) {
            valuesArr.push(this.keyMap[i][j][1]);
          }
        }
      }
    }
    return valuesArr;
  }
}
```

### Binary Search Tree
```javascript
class TreeNode {
  constructor(value) {
    this.value = value;
    this.left = null;
    this.right = null;
  }
}

class BinarySearchTree {
  constructor() {
    this.root = null;
  }
  
  insert(value) {  // O(log n) average, O(n) worst case
    const newNode = new TreeNode(value);
    
    if (!this.root) {
      this.root = newNode;
      return this;
    }
    
    let current = this.root;
    while (true) {
      if (value === current.value) return undefined;  // Duplicate
      
      if (value < current.value) {
        if (!current.left) {
          current.left = newNode;
          return this;
        }
        current = current.left;
      } else {
        if (!current.right) {
          current.right = newNode;
          return this;
        }
        current = current.right;
      }
    }
  }
  
  find(value) {  // O(log n) average, O(n) worst case
    if (!this.root) return false;
    
    let current = this.root;
    while (current) {
      if (value === current.value) return true;
      if (value < current.value) {
        current = current.left;
      } else {
        current = current.right;
      }
    }
    return false;
  }
  
  // Tree traversals
  inOrder() {  // O(n) - Returns sorted array for BST
    const result = [];
    
    function traverse(node) {
      if (node) {
        traverse(node.left);
        result.push(node.value);
        traverse(node.right);
      }
    }
    
    traverse(this.root);
    return result;
  }
  
  preOrder() {  // O(n)
    const result = [];
    
    function traverse(node) {
      if (node) {
        result.push(node.value);
        traverse(node.left);
        traverse(node.right);
      }
    }
    
    traverse(this.root);
    return result;
  }
  
  postOrder() {  // O(n)
    const result = [];
    
    function traverse(node) {
      if (node) {
        traverse(node.left);
        traverse(node.right);
        result.push(node.value);
      }
    }
    
    traverse(this.root);
    return result;
  }
  
  levelOrder() {  // O(n) - BFS
    if (!this.root) return [];
    
    const result = [];
    const queue = [this.root];
    
    while (queue.length) {
      const node = queue.shift();
      result.push(node.value);
      
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    
    return result;
  }
}
```

## Algorithm Patterns 🧩

### Two Pointers
```javascript
// Palindrome check
function isPalindrome(str) {  // O(n) time, O(1) space
  let left = 0;
  let right = str.length - 1;
  
  while (left < right) {
    if (str[left] !== str[right]) {
      return false;
    }
    left++;
    right--;
  }
  
  return true;
}

// Two sum in sorted array
function twoSumSorted(arr, target) {  // O(n) time, O(1) space
  let left = 0;
  let right = arr.length - 1;
  
  while (left < right) {
    const sum = arr[left] + arr[right];
    if (sum === target) {
      return [left, right];
    } else if (sum < target) {
      left++;
    } else {
      right--;
    }
  }
  
  return [-1, -1];
}

// Remove duplicates from sorted array
function removeDuplicates(nums) {  // O(n) time, O(1) space
  if (nums.length === 0) return 0;
  
  let slow = 0;
  for (let fast = 1; fast < nums.length; fast++) {
    if (nums[fast] !== nums[slow]) {
      slow++;
      nums[slow] = nums[fast];
    }
  }
  
  return slow + 1;
}

// Container with most water
function maxArea(height) {  // O(n) time, O(1) space
  let left = 0;
  let right = height.length - 1;
  let maxWater = 0;
  
  while (left < right) {
    const width = right - left;
    const currentHeight = Math.min(height[left], height[right]);
    maxWater = Math.max(maxWater, width * currentHeight);
    
    if (height[left] < height[right]) {
      left++;
    } else {
      right--;
    }
  }
  
  return maxWater;
}
```

### Sliding Window
```javascript
// Maximum sum subarray of size k
function maxSubarraySum(arr, k) {  // O(n) time, O(1) space
  if (arr.length < k) return null;
  
  let maxSum = 0;
  let tempSum = 0;
  
  // Calculate sum of first window
  for (let i = 0; i < k; i++) {
    maxSum += arr[i];
  }
  
  tempSum = maxSum;
  
  // Slide window and update maxSum
  for (let i = k; i < arr.length; i++) {
    tempSum = tempSum - arr[i - k] + arr[i];
    maxSum = Math.max(maxSum, tempSum);
  }
  
  return maxSum;
}

// Longest substring without repeating characters
function lengthOfLongestSubstring(s) {  // O(n) time, O(min(m,n)) space
  const charSet = new Set();
  let left = 0;
  let maxLength = 0;
  
  for (let right = 0; right < s.length; right++) {
    while (charSet.has(s[right])) {
      charSet.delete(s[left]);
      left++;
    }
    charSet.add(s[right]);
    maxLength = Math.max(maxLength, right - left + 1);
  }
  
  return maxLength;
}

// Minimum window substring
function minWindow(s, t) {  // O(|s| + |t|) time, O(|s| + |t|) space
  if (s.length < t.length) return "";
  
  const tCount = {};
  for (const char of t) {
    tCount[char] = (tCount[char] || 0) + 1;
  }
  
  let required = Object.keys(tCount).length;
  let formed = 0;
  const windowCounts = {};
  
  let left = 0;
  let minLen = Infinity;
  let minStart = 0;
  
  for (let right = 0; right < s.length; right++) {
    const char = s[right];
    windowCounts[char] = (windowCounts[char] || 0) + 1;
    
    if (tCount[char] && windowCounts[char] === tCount[char]) {
      formed++;
    }
    
    while (left <= right && formed === required) {
      if (right - left + 1 < minLen) {
        minLen = right - left + 1;
        minStart = left;
      }
      
      const leftChar = s[left];
      windowCounts[leftChar]--;
      if (tCount[leftChar] && windowCounts[leftChar] < tCount[leftChar]) {
        formed--;
      }
      left++;
    }
  }
  
  return minLen === Infinity ? "" : s.substring(minStart, minStart + minLen);
}
```

### Binary Search
```javascript
// Basic binary search
function binarySearch(arr, target) {  // O(log n) time, O(1) space
  let left = 0;
  let right = arr.length - 1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    
    if (arr[mid] === target) {
      return mid;
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  
  return -1;
}

// Find first occurrence
function findFirst(arr, target) {  // O(log n) time, O(1) space
  let left = 0;
  let right = arr.length - 1;
  let result = -1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    
    if (arr[mid] === target) {
      result = mid;
      right = mid - 1;  // Continue searching left
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  
  return result;
}

// Find last occurrence
function findLast(arr, target) {  // O(log n) time, O(1) space
  let left = 0;
  let right = arr.length - 1;
  let result = -1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    
    if (arr[mid] === target) {
      result = mid;
      left = mid + 1;  // Continue searching right
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  
  return result;
}

// Search in rotated sorted array
function searchRotated(nums, target) {  // O(log n) time, O(1) space
  let left = 0;
  let right = nums.length - 1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    
    if (nums[mid] === target) return mid;
    
    // Left half is sorted
    if (nums[left] <= nums[mid]) {
      if (target >= nums[left] && target < nums[mid]) {
        right = mid - 1;
      } else {
        left = mid + 1;
      }
    }
    // Right half is sorted
    else {
      if (target > nums[mid] && target <= nums[right]) {
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
  }
  
  return -1;
}

// Find peak element
function findPeakElement(nums) {  // O(log n) time, O(1) space
  let left = 0;
  let right = nums.length - 1;
  
  while (left < right) {
    const mid = Math.floor((left + right) / 2);
    
    if (nums[mid] > nums[mid + 1]) {
      right = mid;
    } else {
      left = mid + 1;
    }
  }
  
  return left;
}
```

### Depth-First Search (DFS)
```javascript
// Binary tree DFS traversals
class TreeNode {
  constructor(val) {
    this.val = val;
    this.left = null;
    this.right = null;
  }
}

// Recursive DFS
function dfsRecursive(root) {  // O(n) time, O(h) space where h is tree height
  if (!root) return [];
  
  const result = [];
  
  function traverse(node) {
    if (!node) return;
    
    // Pre-order traversal (Node -> Left -> Right)
    result.push(node.val);
    traverse(node.left);
    traverse(node.right);
  }
  
  traverse(root);
  return result;
}

// Iterative DFS using stack
function dfsIterative(root) {  // O(n) time, O(h) space
  if (!root) return [];
  
  const result = [];
  const stack = [root];
  
  while (stack.length > 0) {
    const node = stack.pop();
    result.push(node.val);
    
    // Push right first, then left (so left is processed first)
    if (node.right) stack.push(node.right);
    if (node.left) stack.push(node.left);
  }
  
  return result;
}

// DFS for graph (adjacency list)
function dfsGraph(graph, start, visited = new Set()) {  // O(V + E) time, O(V) space
  const result = [];
  
  function dfs(node) {
    visited.add(node);
    result.push(node);
    
    for (const neighbor of graph[node] || []) {
      if (!visited.has(neighbor)) {
        dfs(neighbor);
      }
    }
  }
  
  dfs(start);
  return result;
}

// Path finding with DFS
function hasPath(graph, start, end, visited = new Set()) {  // O(V + E) time, O(V) space
  if (start === end) return true;
  if (visited.has(start)) return false;
  
  visited.add(start);
  
  for (const neighbor of graph[start] || []) {
    if (hasPath(graph, neighbor, end, visited)) {
      return true;
    }
  }
  
  return false;
}

// Maximum depth of binary tree
function maxDepth(root) {  // O(n) time, O(h) space
  if (!root) return 0;
  
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}

// Validate binary search tree
function isValidBST(root, min = -Infinity, max = Infinity) {  // O(n) time, O(h) space
  if (!root) return true;
  
  if (root.val <= min || root.val >= max) return false;
  
  return isValidBST(root.left, min, root.val) && 
         isValidBST(root.right, root.val, max);
}
```

### Breadth-First Search (BFS)
```javascript
// Binary tree BFS (level order traversal)
function bfs(root) {  // O(n) time, O(w) space where w is max width of tree
  if (!root) return [];
  
  const result = [];
  const queue = [root];
  
  while (queue.length > 0) {
    const node = queue.shift();
    result.push(node.val);
    
    if (node.left) queue.push(node.left);
    if (node.right) queue.push(node.right);
  }
  
  return result;
}

// Level order traversal with levels separated
function levelOrder(root) {  // O(n) time, O(w) space
  if (!root) return [];
  
  const result = [];
  const queue = [root];
  
  while (queue.length > 0) {
    const levelSize = queue.length;
    const currentLevel = [];
    
    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      currentLevel.push(node.val);
      
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    
    result.push(currentLevel);
  }
  
  return result;
}

// BFS for graph (shortest path)
function bfsGraph(graph, start) {  // O(V + E) time, O(V) space
  const visited = new Set();
  const queue = [start];
  const result = [];
  
  visited.add(start);
  
  while (queue.length > 0) {
    const node = queue.shift();
    result.push(node);
    
    for (const neighbor of graph[node] || []) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.push(neighbor);
      }
    }
  }
  
  return result;
}

// Shortest path in unweighted graph
function shortestPath(graph, start, end) {  // O(V + E) time, O(V) space
  if (start === end) return [start];
  
  const visited = new Set();
  const queue = [[start, [start]]];  // [node, path]
  
  visited.add(start);
  
  while (queue.length > 0) {
    const [node, path] = queue.shift();
    
    for (const neighbor of graph[node] || []) {
      if (neighbor === end) {
        return [...path, neighbor];
      }
      
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.push([neighbor, [...path, neighbor]]);
      }
    }
  }
  
  return null;  // No path found
}

// Right side view of binary tree
function rightSideView(root) {  // O(n) time, O(w) space
  if (!root) return [];
  
  const result = [];
  const queue = [root];
  
  while (queue.length > 0) {
    const levelSize = queue.length;
    
    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      
      // Add the rightmost node of each level
      if (i === levelSize - 1) {
        result.push(node.val);
      }
      
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
  }
  
  return result;
}
```

### Dynamic Programming
```javascript
// Fibonacci with memoization
function fibMemo(n, memo = {}) {  // O(n) time, O(n) space
  if (n in memo) return memo[n];
  if (n <= 2) return 1;
  
  memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
  return memo[n];
}

// Fibonacci with tabulation
function fibTab(n) {  // O(n) time, O(n) space
  if (n <= 2) return 1;
  
  const dp = [0, 1, 1];
  
  for (let i = 3; i <= n; i++) {
    dp[i] = dp[i - 1] + dp[i - 2];
  }
  
  return dp[n];
}

// Fibonacci space optimized
function fibOptimized(n) {  // O(n) time, O(1) space
  if (n <= 2) return 1;
  
  let prev2 = 1;
  let prev1 = 1;
  
  for (let i = 3; i <= n; i++) {
    const current = prev1 + prev2;
    prev2 = prev1;
    prev1 = current;
  }
  
  return prev1;
}

// Climbing stairs
function climbStairs(n) {  // O(n) time, O(1) space
  if (n <= 2) return n;
  
  let prev2 = 1;
  let prev1 = 2;
  
  for (let i = 3; i <= n; i++) {
    const current = prev1 + prev2;
    prev2 = prev1;
    prev1 = current;
  }
  
  return prev1;
}

// House robber
function rob(nums) {  // O(n) time, O(1) space
  if (nums.length === 0) return 0;
  if (nums.length === 1) return nums[0];
  
  let prev2 = nums[0];
  let prev1 = Math.max(nums[0], nums[1]);
  
  for (let i = 2; i < nums.length; i++) {
    const current = Math.max(prev1, prev2 + nums[i]);
    prev2 = prev1;
    prev1 = current;
  }
  
  return prev1;
}

// Coin change
function coinChange(coins, amount) {  // O(amount * coins.length) time, O(amount) space
  const dp = new Array(amount + 1).fill(Infinity);
  dp[0] = 0;
  
  for (let i = 1; i <= amount; i++) {
    for (const coin of coins) {
      if (coin <= i) {
        dp[i] = Math.min(dp[i], dp[i - coin] + 1);
      }
    }
  }
  
  return dp[amount] === Infinity ? -1 : dp[amount];
}

// Longest increasing subsequence
function lengthOfLIS(nums) {  // O(n²) time, O(n) space
  if (nums.length === 0) return 0;
  
  const dp = new Array(nums.length).fill(1);
  
  for (let i = 1; i < nums.length; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[j] < nums[i]) {
        dp[i] = Math.max(dp[i], dp[j] + 1);
      }
    }
  }
  
  return Math.max(...dp);
}
```

### Backtracking
```javascript
// Generate all permutations
function permute(nums) {  // O(n! * n) time, O(n! * n) space
  const result = [];
  
  function backtrack(current) {
    if (current.length === nums.length) {
      result.push([...current]);
      return;
    }
    
    for (let i = 0; i < nums.length; i++) {
      if (current.includes(nums[i])) continue;
      
      current.push(nums[i]);
      backtrack(current);
      current.pop();
    }
  }
  
  backtrack([]);
  return result;
}

// Generate all combinations
function combine(n, k) {  // O(C(n,k) * k) time, O(C(n,k) * k) space
  const result = [];
  
  function backtrack(start, current) {
    if (current.length === k) {
      result.push([...current]);
      return;
    }
    
    for (let i = start; i <= n; i++) {
      current.push(i);
      backtrack(i + 1, current);
      current.pop();
    }
  }
  
  backtrack(1, []);
  return result;
}

// Generate all subsets
function subsets(nums) {  // O(2^n * n) time, O(2^n * n) space
  const result = [];
  
  function backtrack(start, current) {
    result.push([...current]);
    
    for (let i = start; i < nums.length; i++) {
      current.push(nums[i]);
      backtrack(i + 1, current);
      current.pop();
    }
  }
  
  backtrack(0, []);
  return result;
}

// N-Queens problem
function solveNQueens(n) {  // O(n!) time, O(n²) space
  const result = [];
  const board = Array(n).fill().map(() => Array(n).fill('.'));
  
  function isValid(row, col) {
    // Check column
    for (let i = 0; i < row; i++) {
      if (board[i][col] === 'Q') return false;
    }
    
    // Check diagonal (top-left to bottom-right)
    for (let i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
      if (board[i][j] === 'Q') return false;
    }
    
    // Check diagonal (top-right to bottom-left)
    for (let i = row - 1, j = col + 1; i >= 0 && j < n; i--, j++) {
      if (board[i][j] === 'Q') return false;
    }
    
    return true;
  }
  
  function backtrack(row) {
    if (row === n) {
      result.push(board.map(row => row.join('')));
      return;
    }
    
    for (let col = 0; col < n; col++) {
      if (isValid(row, col)) {
        board[row][col] = 'Q';
        backtrack(row + 1);
        board[row][col] = '.';
      }
    }
  }
  
  backtrack(0);
  return result;
}

// Word search in grid
function wordSearch(board, word) {  // O(m*n*4^L) time, O(L) space where L is word length
  const rows = board.length;
  const cols = board[0].length;
  
  function dfs(row, col, index) {
    if (index === word.length) return true;
    
    if (row < 0 || row >= rows || col < 0 || col >= cols || 
        board[row][col] !== word[index]) {
      return false;
    }
    
    const temp = board[row][col];
    board[row][col] = '#';  // Mark as visited
    
    const found = dfs(row + 1, col, index + 1) ||
                  dfs(row - 1, col, index + 1) ||
                  dfs(row, col + 1, index + 1) ||
                  dfs(row, col - 1, index + 1);
    
    board[row][col] = temp;  // Restore
    return found;
  }
  
  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      if (dfs(i, j, 0)) return true;
    }
  }
  
  return false;
}
```

## Common JavaScript Tricks for DS&A 💡

### Array Manipulation
```javascript
// Create array of size n filled with value
const arr = new Array(5).fill(0);  // [0, 0, 0, 0, 0] - O(n)
const arr2 = Array(5).fill().map((_, i) => i);  // [0, 1, 2, 3, 4] - O(n)

// Create 2D array (matrix) - CORRECT way
const matrix = Array(3).fill().map(() => Array(3).fill(0));  // O(n*m)
// WRONG: Array(3).fill(Array(3).fill(0)) - creates references to same array

// Create matrix with values
const matrix2 = Array(3).fill().map((_, i) => 
  Array(3).fill().map((_, j) => i * 3 + j)
);  // [[0,1,2], [3,4,5], [6,7,8]]

// Sort array
const numbers = [3, 1, 4, 1, 5];
numbers.sort((a, b) => a - b);  // [1, 1, 3, 4, 5] (ascending) - O(n log n)
numbers.sort((a, b) => b - a);  // [5, 4, 3, 1, 1] (descending) - O(n log n)

// Sort strings
const words = ["banana", "apple", "cherry"];
words.sort();  // ["apple", "banana", "cherry"] - O(n log n)
words.sort((a, b) => a.length - b.length);  // Sort by length

// Sort objects
const people = [{name: "Alice", age: 25}, {name: "Bob", age: 20}];
people.sort((a, b) => a.age - b.age);  // Sort by age

// Find max/min
Math.max(...numbers);  // 5 - O(n)
Math.min(...numbers);  // 1 - O(n)
numbers.reduce((max, num) => Math.max(max, num), -Infinity);  // Alternative max

// Check if all elements satisfy condition
numbers.every(num => num > 0);  // true - O(n)

// Check if any element satisfies condition
numbers.some(num => num > 4);  // true - O(n)

// Remove duplicates
const unique = [...new Set(numbers)];  // O(n)
const unique2 = numbers.filter((num, index) => numbers.indexOf(num) === index);  // O(n²)

// Chunk array into smaller arrays
function chunk(arr, size) {  // O(n)
  const chunks = [];
  for (let i = 0; i < arr.length; i += size) {
    chunks.push(arr.slice(i, i + size));
  }
  return chunks;
}

// Rotate array
function rotateRight(arr, k) {  // O(n) time, O(1) space
  k = k % arr.length;
  arr.unshift(...arr.splice(-k));
  return arr;
}

// Binary search on array
function binarySearchInsert(arr, target) {  // O(log n)
  let left = 0, right = arr.length;
  while (left < right) {
    const mid = Math.floor((left + right) / 2);
    if (arr[mid] < target) left = mid + 1;
    else right = mid;
  }
  return left;
}
```

### String Manipulation
```javascript
// Convert string to array and back
const str = "hello";
const chars = str.split("");  // ["h", "e", "l", "l", "o"] - O(n)
const chars2 = [...str];      // ["h", "e", "l", "l", "o"] (spread) - O(n)
const chars3 = Array.from(str); // ["h", "e", "l", "l", "o"] - O(n)
chars.join("");  // "hello" - O(n)

// Check if string contains substring
str.includes("ell");  // true - O(n)
str.indexOf("ell") !== -1;  // true - O(n)

// Replace all occurrences
str.replace(/l/g, "x");     // "hexxo" (regex global) - O(n)
str.replaceAll("l", "x");   // "hexxo" (modern method) - O(n)
str.split("l").join("x");   // "hexxo" (alternative) - O(n)

// Convert case
str.toUpperCase();  // "HELLO" - O(n)
str.toLowerCase();  // "hello" - O(n)

// String repetition and padding
"abc".repeat(3);          // "abcabcabc" - O(n * k)
"5".padStart(3, "0");     // "005" - O(n)
"5".padEnd(3, "0");       // "500" - O(n)

// Template literals
const name = "Alice";
const greeting = `Hello, ${name}!`; // "Hello, Alice!" - O(n)

// String to number conversions
parseInt("123");          // 123
parseFloat("123.45");     // 123.45
Number("123");            // 123
+"123";                   // 123 (unary plus)

// Check if string is palindrome
function isPalindrome(s) {  // O(n)
  const cleaned = s.toLowerCase().replace(/[^a-z0-9]/g, '');
  return cleaned === cleaned.split('').reverse().join('');
}

// Count character frequency
function charFrequency(str) {  // O(n)
  const freq = {};
  for (const char of str) {
    freq[char] = (freq[char] || 0) + 1;
  }
  return freq;
}

// Check if two strings are anagrams
function areAnagrams(s1, s2) {  // O(n log n)
  return s1.split('').sort().join('') === s2.split('').sort().join('');
}

// Alternative anagram check with frequency
function areAnagramsFreq(s1, s2) {  // O(n)
  if (s1.length !== s2.length) return false;
  
  const freq = {};
  for (const char of s1) {
    freq[char] = (freq[char] || 0) + 1;
  }
  
  for (const char of s2) {
    if (!freq[char]) return false;
    freq[char]--;
  }
  
  return true;
}
```

### Set and Map Operations
```javascript
// Set (for unique values)
const set = new Set([1, 2, 2, 3]);  // {1, 2, 3} - O(n) to create
set.add(4);  // O(1) average
set.has(2);  // true - O(1) average
set.delete(1);  // O(1) average
set.size;  // 3 - O(1)
set.clear();  // Remove all elements - O(1)

// Set operations
const set1 = new Set([1, 2, 3]);
const set2 = new Set([3, 4, 5]);

// Union
const union = new Set([...set1, ...set2]);  // {1, 2, 3, 4, 5} - O(n + m)

// Intersection
const intersection = new Set([...set1].filter(x => set2.has(x)));  // {3} - O(n)

// Difference
const difference = new Set([...set1].filter(x => !set2.has(x)));  // {1, 2} - O(n)

// Map (key-value pairs)
const map = new Map();
map.set("key1", "value1");  // O(1) average
map.set("key2", "value2");
map.get("key1");  // "value1" - O(1) average
map.has("key2");  // true - O(1) average
map.delete("key1");  // O(1) average
map.size;  // 1 - O(1)

// Map with initial values
const map2 = new Map([
  ["a", 1],
  ["b", 2],
  ["c", 3]
]);

// Iterate over Map
for (const [key, value] of map2) {  // O(n)
  console.log(key, value);
}

// Convert between array and Set/Map
Array.from(set);  // [2, 3, 4] - O(n)
[...set];         // [2, 3, 4] (spread) - O(n)
[...map2.entries()];  // [["a", 1], ["b", 2], ["c", 3]] - O(n)
[...map2.keys()];     // ["a", "b", "c"] - O(n)
[...map2.values()];   // [1, 2, 3] - O(n)

// Object to Map and vice versa
const obj = {a: 1, b: 2};
const mapFromObj = new Map(Object.entries(obj));  // O(n)
const objFromMap = Object.fromEntries(map2);      // O(n)

// Frequency counter using Map
function frequencyCounter(arr) {  // O(n)
  const freq = new Map();
  for (const item of arr) {
    freq.set(item, (freq.get(item) || 0) + 1);
  }
  return freq;
}
```

### Number Operations
```javascript
// Math operations
Math.abs(-5);           // 5
Math.ceil(4.3);         // 5
Math.floor(4.7);        // 4
Math.round(4.5);        // 5
Math.trunc(4.7);        // 4 (removes decimal part)
Math.pow(2, 3);         // 8
2 ** 3;                 // 8 (exponentiation operator)
Math.sqrt(16);          // 4
Math.min(1, 2, 3);      // 1
Math.max(1, 2, 3);      ```javascript
Math.max(1, 2, 3);      // 3
Math.random();          // Random number between 0 and 1

// Random integer between min and max (inclusive)
function randomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

// Check if number is integer/float
Number.isInteger(42);   // true
Number.isInteger(42.5); // false
Number.isNaN(NaN);      // true
Number.isFinite(42);    // true

// Convert to different bases
const num = 255;
num.toString(2);        // "11111111" (binary)
num.toString(8);        // "377" (octal)
num.toString(16);       // "ff" (hexadecimal)

// Parse from different bases
parseInt("1010", 2);    // 10 (from binary)
parseInt("ff", 16);     // 255 (from hex)

// Bitwise operations
5 & 3;                  // 1 (AND)
5 | 3;                  // 7 (OR)
5 ^ 3;                  // 6 (XOR)
~5;                     // -6 (NOT)
5 << 1;                 // 10 (left shift)
5 >> 1;                 // 2 (right shift)

// Check if power of 2
function isPowerOfTwo(n) {  // O(1)
  return n > 0 && (n & (n - 1)) === 0;
}

// Count set bits
function countSetBits(n) {  // O(log n)
  let count = 0;
  while (n) {
    count += n & 1;
    n >>= 1;
  }
  return count;
}

// GCD and LCM
function gcd(a, b) {  // O(log min(a,b))
  while (b !== 0) {
    [a, b] = [b, a % b];
  }
  return a;
}

function lcm(a, b) {  // O(log min(a,b))
  return (a * b) / gcd(a, b);
}

// Check if prime
function isPrime(n) {  // O(√n)
  if (n <= 1) return false;
  if (n <= 3) return true;
  if (n % 2 === 0 || n % 3 === 0) return false;
  
  for (let i = 5; i * i <= n; i += 6) {
    if (n % i === 0 || n % (i + 2) === 0) return false;
  }
  return true;
}

// Generate primes up to n (Sieve of Eratosthenes)
function sieveOfEratosthenes(n) {  // O(n log log n)
  const primes = new Array(n + 1).fill(true);
  primes[0] = primes[1] = false;
  
  for (let i = 2; i * i <= n; i++) {
    if (primes[i]) {
      for (let j = i * i; j <= n; j += i) {
        primes[j] = false;
      }
    }
  }
  
  return primes.map((isPrime, num) => isPrime ? num : null)
               .filter(num => num !== null);
}
```

### Advanced Array Techniques
```javascript
// Prefix sum array
function prefixSum(arr) {  // O(n)
  const prefix = [0];
  for (let i = 0; i < arr.length; i++) {
    prefix[i + 1] = prefix[i] + arr[i];
  }
  return prefix;
}

// Range sum query using prefix sum
function rangeSum(prefixSum, left, right) {  // O(1)
  return prefixSum[right + 1] - prefixSum[left];
}

// Kadane's algorithm (maximum subarray sum)
function maxSubarraySum(arr) {  // O(n)
  let maxSoFar = arr[0];
  let maxEndingHere = arr[0];
  
  for (let i = 1; i < arr.length; i++) {
    maxEndingHere = Math.max(arr[i], maxEndingHere + arr[i]);
    maxSoFar = Math.max(maxSoFar, maxEndingHere);
  }
  
  return maxSoFar;
}

// Dutch national flag algorithm (3-way partitioning)
function dutchFlag(arr, pivot) {  // O(n) time, O(1) space
  let low = 0, mid = 0, high = arr.length - 1;
  
  while (mid <= high) {
    if (arr[mid] < pivot) {
      [arr[low], arr[mid]] = [arr[mid], arr[low]];
      low++;
      mid++;
    } else if (arr[mid] > pivot) {
      [arr[mid], arr[high]] = [arr[high], arr[mid]];
      high--;
    } else {
      mid++;
    }
  }
  
  return arr;
}

// Boyer-Moore majority element
function majorityElement(arr) {  // O(n) time, O(1) space
  let candidate = null;
  let count = 0;
  
  // Find candidate
  for (const num of arr) {
    if (count === 0) {
      candidate = num;
    }
    count += (num === candidate) ? 1 : -1;
  }
  
  // Verify candidate (optional if majority is guaranteed)
  count = 0;
  for (const num of arr) {
    if (num === candidate) count++;
  }
  
  return count > arr.length / 2 ? candidate : null;
}

// Next greater element
function nextGreaterElement(arr) {  // O(n)
  const result = new Array(arr.length).fill(-1);
  const stack = [];
  
  for (let i = 0; i < arr.length; i++) {
    while (stack.length > 0 && arr[i] > arr[stack[stack.length - 1]]) {
      const index = stack.pop();
      result[index] = arr[i];
    }
    stack.push(i);
  }
  
  return result;
}

// Sliding window maximum
function slidingWindowMaximum(arr, k) {  // O(n)
  const result = [];
  const deque = [];  // Store indices
  
  for (let i = 0; i < arr.length; i++) {
    // Remove indices outside window
    while (deque.length > 0 && deque[0] <= i - k) {
      deque.shift();
    }
    
    // Remove smaller elements from back
    while (deque.length > 0 && arr[deque[deque.length - 1]] <= arr[i]) {
      deque.pop();
    }
    
    deque.push(i);
    
    // Add to result if window is complete
    if (i >= k - 1) {
      result.push(arr[deque[0]]);
    }
  }
  
  return result;
}

// Merge intervals
function mergeIntervals(intervals) {  // O(n log n)
  if (intervals.length <= 1) return intervals;
  
  intervals.sort((a, b) => a[0] - b[0]);
  const merged = [intervals[0]];
  
  for (let i = 1; i < intervals.length; i++) {
    const current = intervals[i];
    const last = merged[merged.length - 1];
    
    if (current[0] <= last[1]) {
      last[1] = Math.max(last[1], current[1]);
    } else {
      merged.push(current);
    }
  }
  
  return merged;
}
```

### Graph Algorithms
```javascript
// Graph representation
class Graph {
  constructor() {
    this.adjacencyList = {};
  }
  
  addVertex(vertex) {  // O(1)
    if (!this.adjacencyList[vertex]) {
      this.adjacencyList[vertex] = [];
    }
  }
  
  addEdge(v1, v2) {  // O(1)
    this.adjacencyList[v1].push(v2);
    this.adjacencyList[v2].push(v1);  // For undirected graph
  }
  
  removeEdge(v1, v2) {  // O(E)
    this.adjacencyList[v1] = this.adjacencyList[v1].filter(v => v !== v2);
    this.adjacencyList[v2] = this.adjacencyList[v2].filter(v => v !== v1);
  }
  
  removeVertex(vertex) {  // O(V + E)
    while (this.adjacencyList[vertex].length) {
      const adjacentVertex = this.adjacencyList[vertex].pop();
      this.removeEdge(vertex, adjacentVertex);
    }
    delete this.adjacencyList[vertex];
  }
}

// Topological sort (DFS-based)
function topologicalSort(graph) {  // O(V + E)
  const visited = new Set();
  const stack = [];
  
  function dfs(vertex) {
    visited.add(vertex);
    
    for (const neighbor of graph[vertex] || []) {
      if (!visited.has(neighbor)) {
        dfs(neighbor);
      }
    }
    
    stack.push(vertex);
  }
  
  for (const vertex in graph) {
    if (!visited.has(vertex)) {
      dfs(vertex);
    }
  }
  
  return stack.reverse();
}

// Detect cycle in directed graph
function hasCycleDirected(graph) {  // O(V + E)
  const WHITE = 0, GRAY = 1, BLACK = 2;
  const colors = {};
  
  // Initialize all vertices as WHITE
  for (const vertex in graph) {
    colors[vertex] = WHITE;
  }
  
  function dfs(vertex) {
    colors[vertex] = GRAY;
    
    for (const neighbor of graph[vertex] || []) {
      if (colors[neighbor] === GRAY) return true;  // Back edge found
      if (colors[neighbor] === WHITE && dfs(neighbor)) return true;
    }
    
    colors[vertex] = BLACK;
    return false;
  }
  
  for (const vertex in graph) {
    if (colors[vertex] === WHITE && dfs(vertex)) {
      return true;
    }
  }
  
  return false;
}

// Detect cycle in undirected graph
function hasCycleUndirected(graph) {  // O(V + E)
  const visited = new Set();
  
  function dfs(vertex, parent) {
    visited.add(vertex);
    
    for (const neighbor of graph[vertex] || []) {
      if (!visited.has(neighbor)) {
        if (dfs(neighbor, vertex)) return true;
      } else if (neighbor !== parent) {
        return true;  // Back edge to non-parent
      }
    }
    
    return false;
  }
  
  for (const vertex in graph) {
    if (!visited.has(vertex) && dfs(vertex, null)) {
      return true;
    }
  }
  
  return false;
}

// Dijkstra's shortest path
function dijkstra(graph, start) {  // O((V + E) log V) with priority queue
  const distances = {};
  const previous = {};
  const pq = new MinPriorityQueue();
  
  // Initialize distances
  for (const vertex in graph) {
    distances[vertex] = vertex === start ? 0 : Infinity;
    pq.enqueue(vertex, distances[vertex]);
  }
  
  while (!pq.isEmpty()) {
    const current = pq.dequeue().element;
    
    for (const neighbor of graph[current] || []) {
      const alt = distances[current] + neighbor.weight;
      
      if (alt < distances[neighbor.node]) {
        distances[neighbor.node] = alt;
        previous[neighbor.node] = current;
        pq.enqueue(neighbor.node, alt);
      }
    }
  }
  
  return { distances, previous };
}

// Union-Find (Disjoint Set)
class UnionFind {
  constructor(n) {
    this.parent = Array(n).fill().map((_, i) => i);
    this.rank = Array(n).fill(0);
  }
  
  find(x) {  // O(α(n)) with path compression
    if (this.parent[x] !== x) {
      this.parent[x] = this.find(this.parent[x]);  // Path compression
    }
    return this.parent[x];
  }
  
  union(x, y) {  // O(α(n)) with union by rank
    const rootX = this.find(x);
    const rootY = this.find(y);
    
    if (rootX !== rootY) {
      if (this.rank[rootX] < this.rank[rootY]) {
        this.parent[rootX] = rootY;
      } else if (this.rank[rootX] > this.rank[rootY]) {
        this.parent[rootY] = rootX;
      } else {
        this.parent[rootY] = rootX;
        this.rank[rootX]++;
      }
      return true;
    }
    return false;
  }
  
  connected(x, y) {  // O(α(n))
    return this.find(x) === this.find(y);
  }
}
```

### Sorting Algorithms
```javascript
// Quick sort
function quickSort(arr, low = 0, high = arr.length - 1) {  // O(n log n) average, O(n²) worst
  if (low < high) {
    const pi = partition(arr, low, high);
    quickSort(arr, low, pi - 1);
    quickSort(arr, pi + 1, high);
  }
  return arr;
}

function partition(arr, low, high) {
  const pivot = arr[high];
  let i = low - 1;
  
  for (let j = low; j < high; j++) {
    if (arr[j] < pivot) {
      i++;
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
  }
  
  [arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
  return i + 1;
}

// Merge sort
function mergeSort(arr) {  // O(n log n) time, O(n) space
  if (arr.length <= 1) return arr;
  
  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));
  
  return merge(left, right);
}

function merge(left, right) {
  const result = [];
  let i = 0, j = 0;
  
  while (i < left.length && j < right.length) {
    if (left[i] <= right[j]) {
      result.push(left[i++]);
    } else {
      result.push(right[j++]);
    }
  }
  
  return result.concat(left.slice(i)).concat(right.slice(j));
}

// Heap sort
function heapSort(arr) {  // O(n log n) time, O(1) space
  buildMaxHeap(arr);
  
  for (let i = arr.length - 1; i > 0; i--) {
    [arr[0], arr[i]] = [arr[i], arr[0]];
    heapify(arr, 0, i);
  }
  
  return arr;
}

function buildMaxHeap(arr) {
  for (let i = Math.floor(arr.length / 2) - 1; i >= 0; i--) {
    heapify(arr, i, arr.length);
  }
}

function heapify(arr, i, heapSize) {
  const left = 2 * i + 1;
  const right = 2 * i + 2;
  let largest = i;
  
  if (left < heapSize && arr[left] > arr[largest]) {
    largest = left;
  }
  
  if (right < heapSize && arr[right] > arr[largest]) {
    largest = right;
  }
  
  if (largest !== i) {
    [arr[i], arr[largest]] = [arr[largest], arr[i]];
    heapify(arr, largest, heapSize);
  }
}

// Counting sort (for integers in range)
function countingSort(arr, maxVal) {  // O(n + k) where k is range
  const count = new Array(maxVal + 1).fill(0);
  const output = new Array(arr.length);
  
  // Count occurrences
  for (const num of arr) {
    count[num]++;
  }
  
  // Cumulative count
  for (let i = 1; i <= maxVal; i++) {
    count[i] += count[i - 1];
  }
  
  // Build output array
  for (let i = arr.length - 1; i >= 0; i--) {
    output[count[arr[i]] - 1] = arr[i];
    count[arr[i]]--;
  }
  
  return output;
}

// Radix sort
function radixSort(arr) {  // O(d * (n + k)) where d is digits, k is radix
  const max = Math.max(...arr);
  
  for (let exp = 1; Math.floor(max / exp) > 0; exp *= 10) {
    countingSortByDigit(arr, exp);
  }
  
  return arr;
}

function countingSortByDigit(arr, exp) {
  const output = new Array(arr.length);
  const count = new Array(10).fill(0);
  
  for (const num of arr) {
    count[Math.floor(num / exp) % 10]++;
  }
  
  for (let i = 1; i < 10; i++) {
    count[i] += count[i - 1];
  }
  
  for (let i = arr.length - 1; i >= 0; i--) {
    const digit = Math.floor(arr[i] / exp) % 10;
    output[count[digit] - 1] = arr[i];
    count[digit]--;
  }
  
  for (let i = 0; i < arr.length; i++) {
    arr[i] = output[i];
  }
}
```

### Advanced String Algorithms
```javascript
// KMP (Knuth-Morris-Pratt) pattern matching
function kmpSearch(text, pattern) {  // O(n + m) time
  const lps = computeLPS(pattern);
  const matches = [];
  let i = 0, j = 0;
  
  while (i < text.length) {
    if (pattern[j] === text[i]) {
      i++;
      j++;
    }
    
    if (j === pattern.length) {
      matches.push(i - j);
      j = lps[j - 1];
    } else if (i < text.length && pattern[j] !== text[i]) {
      if (j !== 0) {
        j = lps[j - 1];
      } else {
        i++;
      }
    }
  }
  
  return matches;
}

function computeLPS(pattern) {  // Longest Proper Prefix which is also Suffix
  const lps = new Array(pattern.length).fill(0);
  let len = 0;
  let i = 1;
  
  while (i < pattern.length) {
    if (pattern[i] === pattern[len]) {
      len++;
      lps[i] = len;
      i++;
    } else {
      if (len !== 0) {
        len = lps[len - 1];
      } else {
        lps[i] = 0;
        i++;
      }
    }
  }
  
  return lps;
}

// Rabin-Karp rolling hash
function rabinKarp(text, pattern) {  // O(n + m) average, O(nm) worst
  const prime = 101;
  const patternLength = pattern.length;
  const textLength = text.length;
  const matches = [];
  
  let patternHash = 0;
  let textHash = 0;
  let h = 1;
  
  // Calculate h = pow(256, patternLength - 1) % prime
  for (let i = 0; i < patternLength - 1; i++) {
    h = (h * 256) % prime;
  }
  
  // Calculate hash for pattern and first window
  for (let i = 0; i < patternLength; i++) {
    patternHash = (256 * patternHash + pattern.charCodeAt(i)) % prime;
    textHash = (256 * textHash + text.charCodeAt(i)) % prime;
  }
  
  // Slide pattern over text
  for (let i = 0; i <= textLength - patternLength; i++) {
    if (patternHash === textHash) {
      // Check character by character
      let match = true;
      for (let j = 0; j < patternLength; j++) {
        if (text[i + j] !== pattern[j]) {
          match = false;
          break;
        }
      }
      if (match) matches.push(i);
    }
    
    // Calculate hash for next window
    if (i < textLength - patternLength) {
      textHash = (256 * (textHash - text.charCodeAt(i) * h) + 
                  text.charCodeAt(i + patternLength)) % prime;
      
      if (textHash < 0) textHash += prime;
    }
  }
  
  return matches;
}

// Longest Common Subsequence
function longestCommonSubsequence(text1, text2) {  // O(mn) time, O(mn) space
  const m = text1.length;
  const n = text2.length;
  const dp = Array(m + 1).fill().map(() => Array(n + 1).fill(0));
  
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (text1[i - 1] === text2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1] + 1;
      } else {
        dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
      }
    }
  }
  
  return dp[m][n];
}

// Edit distance (Levenshtein distance)
function editDistance(word1, word2) {  // O(mn) time, O(mn) space
  const m = word1.length;
  const n = word2.length;
  const dp = Array(m + 1).fill().map(() => Array(n + 1).fill(0));
  
  // Initialize base cases
  for (let i = 0; i <= m; i++) dp[i][0] = i;
  for (let j = 0; j <= n; j++) dp[0][j] = j;
  
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (word1[i - 1] === word2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1];
      } else {
        dp[i][j] = 1 + Math.min(
          dp[i - 1][j],     // Delete
          dp[i][j - 1],     // Insert
          dp[i - 1][j - 1]  // Replace
        );
      }
    }
  }
  
  return dp[m][n];
}
```

### Utility Functions and Helpers
```javascript
// Deep clone object/array
function deepClone(obj) {  // O(n) where n is number of properties
  if (obj === null || typeof obj !== "object") return obj;
  if (obj instanceof Date) return new Date(obj.getTime());
  if (obj instanceof Array) return obj.map(item => deepClone(item));
  if (typeof obj === "object") {
    const clonedObj = {};
    for (const key in obj) {
      if (obj.hasOwnProperty(key)) {
        clonedObj[key] = deepClone(obj[key]);
      }
    }
    return clonedObj;
  }
}

// Alternative using JSON (limited but fast)
function deepCloneJSON(obj) {  // O(n)
  return JSON.parse(JSON.stringify(obj));
}

// Debounce function
function debounce(func, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func.apply(this, args), delay);
  };
}

// Throttle function
function throttle(func, limit) {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// Memoization decorator
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// Curry function
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    } else {
      return function(...args2) {
        return curried.apply(this, args.concat(args2));
      };
    }
  };
}

// Pipe function (left to right composition)
function pipe(...fns) {
  return function(value) {
    return fns.reduce((acc, fn) => fn(acc), value);
  };
}

// Compose function (right to left composition)
function compose(...fns) {
  return function(value) {
    return fns.reduceRight((acc, fn) => fn(acc), value);
  };
}

// Generate unique ID
function generateId() {
  return Date.now().toString(36) + Math.random().toString(36).substr(2);
}

// Flatten nested array
function flattenArray(arr, depth = Infinity) {  // O(n)
  return depth > 0 ? arr.reduce((acc, val) => 
    acc.concat(Array.isArray(val) ? flattenArray(val, depth - 1) : val), [])
    : arr.slice();
}

// Group array by property
function groupBy(array, key) {  // O(n)
  return array.reduce((groups, item) => {
    const group = typeof key === 'function' ? key(item) : item[key];
    groups[group] = groups[group] || [];
    groups[group].push(item);
    return groups;
  }, {});
}

// Partition array based on predicate
function partition(array, predicate) {  // O(n)
  return array.reduce((acc, item) => {
    acc[predicate(item) ? 0 : 1].push(item);
    return acc;
  }, [[], []]);
}

// Get random element from array
function randomElement(arr) {  // O(1)
  return arr[Math.floor(Math.random() * arr.length)];
}

// Shuffle array (Fisher-Yates)
function shuffle(arr) {  // O(n)
  const shuffled = [...arr];
  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
  }
  return shuffled;
}

// Check if arrays are equal
function arraysEqual(a, b) {  // O(n)
  if (a.length !== b.length) return false;
  return a.every((val, i) => val === b[i]);
}

// Deep equality check
function deepEqual(a, b) {  // O(n)
  if (a === b) return true;
  if (a == null || b == null) return false;
  if (Array.isArray(a) && Array.isArray(b)) {
    if (a.length !== b.length) return false;
    return a.every((val, i) => deepEqual(val, b[i]));
  }
  if (typeof a === 'object' && typeof b === 'object') {
    const keysA = Object.keys(a);
    const keysB = Object.keys(b);
    if (keysA.length !== keysB.length) return false;
    return keysA.every(key => deepEqual(a[key], b[key]));
  }
  return false;
}
```

### Performance Optimization Tips
```javascript
// Use Map for frequent lookups instead of objects
const map = new Map();  // O(1) average lookup
const obj = {};         // O(1) average lookup, but slower

// Use Set for unique values and membership testing
const set = new Set();  // O(1) average lookup
const arr = [];         // O(n) lookup with includes()

// Avoid nested loops when possible
// Bad: O(n²)
for (let i = 0; i < arr1.length; i++) {
  for (let j = 0; j < arr2.length; j++) {
    if (arr1[i] === arr2[j]) {
      // found match
    }
  }
}

// Good: O(n + m)
const set1 = new Set(arr1);
const matches = arr2.filter(item => set1.has(item));

// Use appropriate data structures
// For LIFO: Use array with push/pop
// For FIFO: Use custom queue implementation (not array with shift)
// For priority queue: Use heap implementation
// For range queries: Use segment tree or Fenwick tree

// Avoid unnecessary array copying
// Bad: O(n) for each operation
arr.slice().reverse()

// Good: O(1) space if you can modify in place
arr.reverse()

// Use binary search for sorted arrays
// Bad: O(n)
arr.find(x => x === target)

// Good: O(log n)
binarySearch(arr, target)

// Cache expensive computations
const expensiveFunction = memoize((n) => {
  // expensive computation
  return result;
});
```

## Testing and Debugging 🧪

### Basic Testing Patterns
```javascript
// Simple assertion function
function assert(condition, message) {
  if (!condition) {
    throw new Error(message || "Assertion failed");
  }
}

// Test runner
function runTests() {
  const tests = [
    () => assert(binarySearch([1, 2, 3, 4, 5], 3) === 2, "Binary search failed"),
    () => assert(isPalindrome("racecar") === true, "Palindrome check failed"),
    () => assert(maxSubarraySum([1, 2, 3], 2) === 5, "Max subarray sum failed")
  ];
  
  tests.forEach((test, index) => {
    try {
      test();
      console.log(`✅ Test ${index + 1} passed`);
    } catch (error) {
      console.log(`❌ Test ${index + 1} failed: ${error.message}`);
    }
  });
}

// Performance testing
function timeFunction(fn, ...args) {
  const start = performance.now();
  const result = fn(...args);
  const end = performance.now();
  console.log(`Function took ${end - start} milliseconds`);
  return result;
}

// Memory usage (Node.js)
function memoryUsage() {
  const used = process.memoryUsage();
  for (let key in used) {
    console.log(`${key}: ${Math.round(used[key] / 1024 / 1024 * 100) / 100} MB`);
  }
}
```

## Further Reading 📚

### Essential Resources
- **Official Documentation:**
  - [MDN JavaScript Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide)
  - [MDN JavaScript Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference)
  - [ECMAScript Specifications](https://tc39.es/ecma262/)

- **Algorithm Practice Platforms:**
  - [LeetCode](https://leetcode.com/) - Comprehensive problem set with discussions
  - [HackerRank](https://www.hackerrank.com/) - Structured learning paths
  - [CodeSignal](https://codesignal.com/) - Interview preparation
  - [AtCoder](https://atcoder.jp/) - Competitive programming
  - [Codeforces](https://codeforces.com/) - Advanced competitive programming

- **Algorithm Visualization:**
  - [VisuAlgo](https://visualgo.net/) - Interactive algorithm visualizations
  - [Algorithm Visualizer](https://algorithm-visualizer.org/) - Code execution visualization
  - [Big-O Cheat Sheet](https://www.bigocheatsheet.com/) - Time/space complexity reference

### Books
- **JavaScript Specific:**
  - "Eloquent JavaScript" by Marijn Haverbeke
  - "You Don't Know JS" series by Kyle Simpson
  - "JavaScript: The Good Parts" by Douglas Crockford

- **Algorithms & Data Structures:**
  - "Cracking the Coding Interview" by Gayle Laakmann McDowell
  - "Introduction to Algorithms" by Cormen, Leiserson, Rivest, and Stein
  - "Grokking Algorithms" by Aditya Bhargava
  - "Algorithm Design Manual" by Steven Skiena
  - "Elements of Programming Interviews" by Aziz, Lee, and Prakash

### Advanced Topics to Explore
- **Advanced Data Structures:**
  - Segment Trees, Fenwick Trees (Binary Indexed Trees)
  - Trie (Prefix Tree), Suffix Arrays
  - Disjoint Set Union (Union-Find) with optimizations
  - Red-Black Trees, AVL Trees
  - B-Trees, B+ Trees

- **Advanced Algorithms:**
  - Network Flow (Ford-Fulkerson, Edmonds-Karp)
  - String Matching (KMP, Rabin-Karp, Z-algorithm)
  - Graph Algorithms (Bellman-Ford, Floyd-Warshall, Tarjan's)
  - Geometric Algorithms
  - Number Theory Algorithms

- **Optimization Techniques:**
  - Memoization and Dynamic Programming patterns
  - Greedy Algorithms
  - Divide and Conquer strategies
  - Branch and Bound
  - Approximation Algorithms

This comprehensive guide covers the essential JavaScript syntax and patterns needed for data structures and algorithms. The efficiency annotations help you understand the performance characteristics of different operations, which is crucial for writing optimal solutions in coding interviews and competitive programming.

