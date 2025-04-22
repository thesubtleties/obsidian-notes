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
const text = "Hello";     // Strings
const isTrue = true;      // Booleans
const nothing = null;     // Null
const notDefined = undefined; // Undefined

// Complex data types
const arr = [1, 2, 3];    // Arrays
const obj = {             // Objects
  key: "value",
  number: 42
};
```

### Arrays - Essential for DS&A 📊
```javascript
// Creating arrays
const arr = [1, 2, 3, 4, 5];
const empty = [];
const mixed = [1, "two", {three: 3}, [4]];

// Accessing elements (0-indexed)
arr[0];                   // 1 (first element)
arr[arr.length - 1];      // 5 (last element)

// Common array methods
arr.push(6);              // Add to end: [1,2,3,4,5,6]
arr.pop();                // Remove from end: [1,2,3,4,5]
arr.unshift(0);           // Add to beginning: [0,1,2,3,4,5]
arr.shift();              // Remove from beginning: [1,2,3,4,5]
arr.slice(1, 3);          // Extract [1,3): [2,3]
arr.splice(1, 2, 'a');    // Remove 2 elements at index 1, insert 'a': [1,'a',4,5]

// Iteration methods
arr.forEach(num => console.log(num));  // Iterate without creating new array
const doubled = arr.map(num => num * 2);  // Create new array: [2,4,6,8,10]
const evens = arr.filter(num => num % 2 === 0);  // Filter: [2,4]
const sum = arr.reduce((total, num) => total + num, 0);  // Reduce to single value: 15
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
obj.name;                 // "Alice" (dot notation)
obj["age"];               // 25 (bracket notation)

// Adding/modifying properties
obj.location = "New York";
obj["isStudent"] = false;

// Checking if property exists
"name" in obj;            // true
obj.hasOwnProperty("age"); // true

// Object methods
Object.keys(obj);         // ["name", "age", "isStudent", "location"]
Object.values(obj);       // ["Alice", 25, false, "New York"]
Object.entries(obj);      // [["name", "Alice"], ["age", 25], ...]
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

// Switch statement
switch (value) {
  case 1:
    // code
    break;
  case 2:
    // code
    break;
  default:
    // code
}
```

### Loops - Critical for Algorithms ⭐
```javascript
// For loop
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}

// For...of (iterates over values)
for (const element of arr) {
  console.log(element);
}

// For...in (iterates over keys/indices)
for (const key in obj) {
  console.log(key, obj[key]);
}

// While loop
let i = 0;
while (i < arr.length) {
  console.log(arr[i]);
  i++;
}

// Do-while loop
let j = 0;
do {
  console.log(arr[j]);
  j++;
} while (j < arr.length);
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

// Default parameters
function greet(name = "Guest") {
  return `Hello, ${name}!`;
}

// Rest parameters
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}
```

## Common Data Structures Implementation 📚

### Stack
```javascript
class Stack {
  constructor() {
    this.items = [];
  }
  
  push(element) {
    this.items.push(element);
  }
  
  pop() {
    if (this.isEmpty()) return "Underflow";
    return this.items.pop();
  }
  
  peek() {
    return this.items[this.items.length - 1];
  }
  
  isEmpty() {
    return this.items.length === 0;
  }
}

// Usage
const stack = new Stack();
stack.push(10);
stack.push(20);
console.log(stack.pop());  // 20
```

### Queue
```javascript
class Queue {
  constructor() {
    this.items = [];
  }
  
  enqueue(element) {
    this.items.push(element);
  }
  
  dequeue() {
    if (this.isEmpty()) return "Underflow";
    return this.items.shift();
  }
  
  front() {
    if (this.isEmpty()) return "Queue is empty";
    return this.items[0];
  }
  
  isEmpty() {
    return this.items.length === 0;
  }
}

// Usage
const queue = new Queue();
queue.enqueue("A");
queue.enqueue("B");
console.log(queue.dequeue());  // "A"
```

### Linked List
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
  }
  
  append(data) {
    const newNode = new Node(data);
    
    if (!this.head) {
      this.head = newNode;
      return;
    }
    
    let current = this.head;
    while (current.next) {
      current = current.next;
    }
    current.next = newNode;
  }
  
  prepend(data) {
    const newNode = new Node(data);
    newNode.next = this.head;
    this.head = newNode;
  }
  
  delete(data) {
    if (!this.head) return;
    
    if (this.head.data === data) {
      this.head = this.head.next;
      return;
    }
    
    let current = this.head;
    while (current.next && current.next.data !== data) {
      current = current.next;
    }
    
    if (current.next) {
      current.next = current.next.next;
    }
  }
  
  print() {
    let current = this.head;
    const values = [];
    while (current) {
      values.push(current.data);
      current = current.next;
    }
    console.log(values.join(" -> "));
  }
}

// Usage
const list = new LinkedList();
list.append(1);
list.append(2);
list.prepend(0);
list.print();  // 0 -> 1 -> 2
```

## Algorithm Patterns 🧩

### Two Pointers
```javascript
function isPalindrome(str) {
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

// Usage
console.log(isPalindrome("racecar"));  // true
```

### Sliding Window
```javascript
function maxSubarraySum(arr, k) {
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

// Usage
console.log(maxSubarraySum([2, 1, 5, 1, 3, 2], 3));  // 9
```

### Binary Search
```javascript
function binarySearch(arr, target) {
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
  
  return -1;  // Not found
}

// Usage
console.log(binarySearch([1, 2, 3, 4, 5], 3));  // 2
```

### Depth-First Search (DFS)
```javascript
// For a binary tree
class TreeNode {
  constructor(val) {
    this.val = val;
    this.left = null;
    this.right = null;
  }
}

function dfs(root) {
  if (!root) return [];
  
  const result = [];
  
  function traverse(node) {
    if (!node) return;
    
    // Pre-order traversal (Node -> Left -> Right)
    result.push(node.val);
    traverse(node.left);
    traverse(node.right);
    
    // For in-order (Left -> Node -> Right):
    // traverse(node.left);
    // result.push(node.val);
    // traverse(node.right);
    
    // For post-order (Left -> Right -> Node):
    // traverse(node.left);
    // traverse(node.right);
    // result.push(node.val);
  }
  
  traverse(root);
  return result;
}

// Usage
const root = new TreeNode(1);
root.left = new TreeNode(2);
root.right = new TreeNode(3);
console.log(dfs(root));  // [1, 2, 3]
```

### Breadth-First Search (BFS)
```javascript
function bfs(root) {
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

// Usage
const root = new TreeNode(1);
root.left = new TreeNode(2);
root.right = new TreeNode(3);
console.log(bfs(root));  // [1, 2, 3]
```

## Common JavaScript Tricks for DS&A 💡

### Array Manipulation
```javascript
// Create array of size n filled with value
const arr = new Array(5).fill(0);  // [0, 0, 0, 0, 0]

// Create 2D array (matrix)
const matrix = Array(3).fill().map(() => Array(3).fill(0));
// [[0, 0, 0], [0, 0, 0], [0, 0, 0]]

// Sort array
const numbers = [3, 1, 4, 1, 5];
numbers.sort((a, b) => a - b);  // [1, 1, 3, 4, 5] (ascending)
numbers.sort((a, b) => b - a);  // [5, 4, 3, 1, 1] (descending)

// Find max/min
Math.max(...numbers);  // 5
Math.min(...numbers);  // 1

// Check if all elements satisfy condition
numbers.every(num => num > 0);  // true

// Check if any element satisfies condition
numbers.some(num => num > 4);  // true
```

### String Manipulation
```javascript
// Convert string to array and back
const str = "hello";
const chars = str.split("");  // ["h", "e", "l", "l", "o"]
chars.join("");  // "hello"

// Check if string contains substring
str.includes("ell");  // true

// Replace all occurrences
str.replace(/l/g, "x");  // "hexxo"

// Convert case
str.toUpperCase();  // "HELLO"
str.toLowerCase();  // "hello"
```

### Set and Map
```javascript
// Set (for unique values)
const set = new Set([1, 2, 2, 3]);  // {1, 2, 3}
set.add(4);
set.has(2);  // true
set.delete(1);
set.size;  // 3

// Map (key-value pairs)
const map = new Map();
map.set("key1", "value1");
map.set("key2", "value2");
map.get("key1");  // "value1"
map.has("key2");  // true
map.delete("key1");
map.size;  // 1

// Convert between array and Set/Map
Array.from(set);  // [2, 3, 4]
[...map.entries()];  // [["key2", "value2"]]
```

## Further Reading 📚

- Official Documentation:
  - [MDN JavaScript Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide)
  - [MDN JavaScript Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference)

- Algorithm Resources:
  - [JavaScript Algorithms GitHub Repository](https://github.com/trekhleb/javascript-algorithms)
  - [LeetCode](https://leetcode.com/) - Practice platform
  - [HackerRank](https://www.hackerrank.com/) - Practice platform

- Books:
  - "Eloquent JavaScript" by Marijn Haverbeke
  - "Cracking the Coding Interview" by Gayle Laakmann McDowell
  - "Grokking Algorithms" by Aditya Bhargava

---