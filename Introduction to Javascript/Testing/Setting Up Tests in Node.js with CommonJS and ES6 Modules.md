

## CommonJS Modules

### Step 1: Initialize a New Project

```bash
mkdir my-commonjs-project
cd my-commonjs-project
npm init -y
```

### Step 2: Install Mocha and Chai

```bash
npm install mocha chai --save-dev
```

### Step 3: Create the Directory Structure

```plaintext
my-commonjs-project/
├── index.js
├── test/
│   └── index-spec.js
├── package.json
└── ...
```

### Step 4: Write the Main Module (`index.js`)

```javascript
// index.js
function insertionSort(arr) {
    for (let i = 1; i < arr.length; i++) {
        let item = arr[i];
        let j = i - 1;
        while (j >= 0 && arr[j] > item) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = item;
    }
    return arr;
}

module.exports = insertionSort;
```

### Step 5: Write the Test File (`test/index-spec.js`)

```javascript
// test/index-spec.js
const chai = require('chai');
const expect = chai.expect;
const insertionSort = require('../index.js');

describe('insertionSort', () => {
    it('should sort an array of numbers correctly', () => {
        const result = insertionSort([3, 8, 12, 5, 22, 14, 9, 7, 1, 6]);
        const expected = [1, 3, 5, 6, 7, 8, 9, 12, 14, 22];
        expect(result).to.eql(expected);
    });
});
```

### Step 6: Update `package.json` to Run Tests

```json
{
  "name": "my-commonjs-project",
  "version": "1.0.0",
  "scripts": {
    "test": "mocha"
  },
  "devDependencies": {
    "chai": "^5.1.1",
    "mocha": "^10.4.0"
  }
}
```

### Step 7: Run the Tests

```bash
npm test
```

---

## ES6 Modules

### Step 1: Initialize a New Project

```bash
mkdir my-es6-project
cd my-es6-project
npm init -y
```

### Step 2: Install Mocha and Chai

```bash
npm install mocha chai --save-dev
```

### Step 3: Create the Directory Structure

```plaintext
my-es6-project/
├── index.js
├── test/
│   └── index-spec.js
├── package.json
└── ...
```

### Step 4: Update `package.json` to Use ES Modules

```json
{
  "name": "my-es6-project",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "test": "mocha"
  },
  "devDependencies": {
    "chai": "^5.1.1",
    "mocha": "^10.4.0"
  }
}
```

### Step 5: Write the Main Module (`index.js`)

```javascript
// index.js
function insertionSort(arr) {
    for (let i = 1; i < arr.length; i++) {
        let item = arr[i];
        let j = i - 1;
        while (j >= 0 && arr[j] > item) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = item;
    }
    return arr;
}

export { insertionSort };
```

### Step 6: Write the Test File (`test/index-spec.js`)

```javascript
// test/index-spec.js
import { expect } from 'chai';
import { insertionSort } from '../index.js';

describe('insertionSort', () => {
    it('should sort an array of numbers correctly', () => {
        const result = insertionSort([3, 8, 12, 5, 22, 14, 9, 7, 1, 6]);
        const expected = [1, 3, 5, 6, 7, 8, 9, 12, 14, 22];
        expect(result).to.eql(expected);
    });
});
```

### Step 7: Run the Tests

```bash
npm test
```

---

## Explanation

### CommonJS Modules

- **Syntax:** Uses `require` for importing modules and `module.exports` for exporting functions or objects.
- **Usage:** Historically used in Node.js environments.

### ES6 Modules

- **Syntax:** Uses `import` and `export` statements for module management.
- **Standardization:** ES6 modules provide a standardized module system for both browsers and Node.js.
- **Configuration:** The `"type": "module"` field in `package.json` indicates that the project uses ES modules.

### Benefits of ES6 Modules

1. **Standardization:** ES6 modules are the standardized module system in JavaScript, working across different environments.
2. **Static Analysis:** Better support for static analysis, which helps in optimizations like tree shaking.
3. **Modern Tooling:** Modern JavaScript tools and libraries increasingly rely on ES6 modules.

### Additional Notes

- **File Extensions:** Ensure that all JavaScript files use the `.js` extension unless you need to distinguish between CommonJS (`.cjs`) and ES6 modules (`.mjs`).
- **Import Statements:** Ensure that import statements include the correct file paths and extensions when necessary.
- **Running Tests:** Ensure your test runner is configured to handle the module type you are using.

----

