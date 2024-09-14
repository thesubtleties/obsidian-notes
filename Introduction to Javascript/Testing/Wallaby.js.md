### Using Wallaby.js with Non-Module Format (CommonJS)

If your JavaScript project does not use ES modules (i.e., it does not have `"type": "module"` in the `package.json`), you should use CommonJS syntax for configuring Wallaby.js. The CommonJS format is more straightforward and is compatible with Node.js by default.

### Example Wallaby.js Configuration for Non-Module Format

```javascript
module.exports = function () {
  return {
    files: [
      'problems/*.js' // adjust if required
    ],

    tests: [
      'test/*.js' // adjust if required
    ],

    env: {
      type: 'node'
    }
  };
};
```

### Sample File (CommonJS)

**problems/sample.js**
```javascript
function add(a, b) {
  return a + b;
}

module.exports = add;
```

**test/sample-test.js**
```javascript
const expect = require('chai').expect;
const add = require('../problems/sample');

describe('add', () => {
  it('should add two numbers correctly', () => {
    const result = add(2, 3);
    expect(result).to.equal(5);
  });
});
```

### When to Use CommonJS (Non-Module Format)

Use the CommonJS format if:

- Your project does not use `"type": "module"` in the `package.json`.
- Your JavaScript files use `require` and `module.exports` for importing and exporting modules.
- You want to ensure compatibility with older Node.js versions or certain libraries that expect CommonJS modules.

## Using Wallaby.js with ES Modules

If your project uses ES modules (i.e., it has `"type": "module"` in the `package.json`), you should configure Wallaby.js using ES module syntax. This setup leverages the modern JavaScript module system and is required for compatibility with ES module features.

### Example Wallaby.js Configuration for ES Modules

```javascript
export default function (wallaby) {
  return {
    files: [
      'Sorts/**/*.mjs',    // Include all source files with .mjs extension
      '!Sorts/test/**/*.mjs'   // Exclude test files from the files list
    ],

    tests: [
      'Sorts/test/**/*.mjs'    // Include all test files
    ],

    env: {
      type: 'node',
      runner: 'node'
    },

    testFramework: 'mocha',
  };
};
```

### Sample File (ES Modules)

**Sorts/sample.mjs**
```javascript
export function add(a, b) {
  return a + b;
}
```

**Sorts/test/sample-test.mjs**
```javascript
import { expect } from 'chai';
import { add } from '../sample.mjs';

describe('add', () => {
  it('should add two numbers correctly', () => {
    const result = add(2, 3);
    expect(result).to.equal(5);
  });
});
```

### When to Use ES Modules

Use the ES module format if:

- Your project specifies `"type": "module"` in the `package.json`.
- Your JavaScript files use `import` and `export` for importing and exporting modules.
- You are using modern JavaScript features that require ES modules, such as top-level `await`, dynamic `import()`, and more.

### Summary

In summary, the choice between CommonJS and ES module formats depends on your project's module system:

- **CommonJS (Non-Module Format)**: Use for traditional Node.js projects without `"type": "module"`.
- **ES Modules**: Use for modern JavaScript projects that specify `"type": "module"` and leverage ES module features.

Choose the configuration that aligns with your project's module system to ensure compatibility and take advantage of the appropriate JavaScript features.
