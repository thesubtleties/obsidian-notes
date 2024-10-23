
```javascript
// Common Utility File Types
📁 utils/
  ├── stringUtils.js      // String manipulations, formatting
  ├── dateUtils.js        // Date formatting, calculations
  ├── numberUtils.js      // Number formatting, math operations
  ├── arrayUtils.js       // Array manipulations, sorting
  ├── validationUtils.js  // Input validation, type checking
  ├── formatUtils.js      // General formatting (money, phones, etc)
  ├── httpUtils.js        // API calls, fetch wrappers
  └── storageUtils.js     // localStorage/sessionStorage helpers

// Best Practice Structure
// stringUtils.js example
import { ERROR_MESSAGES } from '../constants';

// 1. Constants at top
const DEFAULT_SEPARATOR = '-';

// 2. TypeScript types if using TS
type StringTransformFn = (str: string) => string;

// 3. Private helper functions
const isValidString = (str) => typeof str === 'string' && str.length > 0;

// 4. Exported functions with documentation
/**
 * Capitalizes first letter of string
 * @param {string} str - Input string
 * @returns {string} Capitalized string
 * @throws {Error} If input is not a string
 */
export const capitalize = (str) => {
    if (!isValidString(str)) {
        throw new Error(ERROR_MESSAGES.INVALID_STRING);
    }
    return str.charAt(0).toUpperCase() + str.slice(1);
};
```

Key Rules:
```javascript
// 1. Keep functions pure
// ❌ Bad
export const addTax = (price) => {
    globalTaxRate *= 1.1; // Side effect!
    return price * globalTaxRate;
};

// ✅ Good
export const addTax = (price, taxRate) => price * (1 + taxRate);

// 2. Single Responsibility
// ❌ Bad
export const processUserData = (user) => {
    // Does too many things
    validateUser(user);
    formatUserName(user);
    saveToDatabase(user);
};

// ✅ Good
// Split into separate utilities or services

// 3. Consistent Error Handling
export const divide = (a, b) => {
    if (typeof a !== 'number' || typeof b !== 'number') {
        throw new TypeError('Arguments must be numbers');
    }
    if (b === 0) {
        throw new Error('Division by zero');
    }
    return a / b;
};

// 4. Default Parameters
export const truncate = (
    str, 
    length = 50, 
    ending = '...'
) => {
    // Implementation
};
```

Import/Export Pattern:
```javascript
// Named exports for multiple functions
export { capitalize, truncate, slugify };

// Import what you need
import { capitalize } from '@utils/stringUtils';

// Or import all
import * as StringUtils from '@utils/stringUtils';
```

Organization Tips:
1. Group related functions together
2. Export functions individually (better tree-shaking)
3. Use consistent naming conventions
   - get* for retrievers
   - is* for booleans
   - format* for formatting
4. Add unit tests for utilities
5. Use TypeScript for better type safety
6. Document edge cases and throws
7. Consider performance implications

Watch out for:
- Circular dependencies
- Duplicated logic across utils
- Over-abstracting simple operations
- Missing error handling
- Inconsistent return types
- Side effects
- Global state modifications

Setup in project:
```javascript
// vite.config.js or webpack.config.js
{
    resolve: {
        alias: {
            '@utils': path.resolve(__dirname, './src/utils')
        }
    }
}

// jsconfig.json or tsconfig.json
{
    "compilerOptions": {
        "baseUrl": "src",
        "paths": {
            "@utils/*": ["utils/*"]
        }
    }
}
```

This gives you a solid foundation for organizing utility functions that scale well with your project.