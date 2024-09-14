Absolutely. You're right to point that out - understanding exports is crucial when dealing with modules. Here's an updated cheat sheet that includes exports for both `require` and `import` systems:

```js
=== CommonJS (require/exports) ===

Importing:
const module = require('module-name');

Exporting:
module.exports = thing
// or
exports.namedThing = thing

Characteristics:
- Synchronous
- Dynamic loading possible
- Default in Node.js

Examples:
// Importing
const fs = require('fs');
const { readFile } = require('fs');

// Exporting
module.exports = class MyClass {};
// or
exports.myFunction = () => {};

=== ES6 Modules (import/export) ===

Importing:
import module from 'module-name';
import { namedExport } from 'module-name';

Exporting:
export default thing
export { thing }
export const namedThing = thing

Characteristics:
- Asynchronous
- Static analysis possible at compile time
- Native in modern browsers and newer Node.js versions

Examples:
// Importing
import React from 'react';
import { useState } from 'react';

// Exporting
export default class MyClass {};
export const myFunction = () => {};
export { myFunction, MyClass };

```


Now, let's dive into some of the gotchas and important points:

1. In CommonJS, `module.exports` and `exports` are not exactly the same thing. `exports` is initially a reference to `module.exports`, but if you reassign `exports`, it breaks that reference. This can bite you:

   ```javascript
   exports = { foo: 'bar' }; // This doesn't work as expected!
   module.exports = { foo: 'bar' }; // This does work
   ```

2. With ES6 modules, you can have only one default export per module, but any number of named exports:

   ```javascript
   export default function() {} // Only one of these
   export const foo = 1; // As many of these as you want
   export const bar = 2;
   ```

3. When importing a default export, you can name it whatever you want:

   ```javascript
   import MyCustomName from './module';
   ```

   But for named exports, you need to use the exact name (or use aliasing):

   ```javascript
   import { exportedName as myCustomName } from './module';
   ```

4. You can combine default and named imports:

   ```javascript
   import React, { useState, useEffect } from 'react';
   ```

5. In CommonJS, you can do dynamic property naming on exports:

   ```javascript
   const name = 'myExport';
   exports[name] = () => {};
   ```

   This isn't directly possible with ES6 export syntax, but you can achieve similar results with computed property names in the exported object.

6. ES6 modules support live bindings. This means when you import a value, it can be updated if it changes in the source module. This doesn't happen with CommonJS - you get a copy of the value at the time of require.

7. Circular dependencies are handled differently between the two systems. CommonJS will give you partial exports if there's a circular dependency, while ES6 modules have a temporal dead zone and will throw an error if you try to use a value that hasn't been initialized yet.

Remember, mixing these systems can lead to unexpected behavior. Stick to one system within a project if possible, and if you must mix them, be very careful about how you're exporting and importing. Always consider the environment you're working in - browser, Node.js, or something else - as it affects which system you can or should use.