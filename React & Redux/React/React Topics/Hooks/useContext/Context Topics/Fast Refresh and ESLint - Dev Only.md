## What is Fast Refresh?
Fast Refresh is a development-only feature that allows React components to be edited while the app is running, without losing component state.

## Common ESLint Development Warnings

### Fast Refresh Warning
```javascript
// This pattern triggers the warning
// src/contexts/ThemeContext.jsx

export const ThemeContext = createContext();  // non-component export
export const useTheme = () => useContext(ThemeContext);  // non-component export
export default function ThemeProvider() {}  // component export
```
> Warning appears because file exports both components and non-components, limiting Fast Refresh capabilities.

### Fixing with ESLint Config
```javascript
// .eslintrc.cjs
module.exports = {
  // ... other config
  overrides: [
    {
      // Target specific directories/files
      files: ["src/context/*.jsx"],
      rules: {
        // Disable Fast Refresh warning
        'react-refresh/only-export-components': 'off'
      }
    }
  ],
}
```
> This configuration only affects development and won't impact production builds.

## Development vs Production Impact

### Development
- Fast Refresh active
- ESLint warnings visible
- Slower refresh for files with mixed exports
- ESLint rules help maintain optimal development experience

### Production
- Fast Refresh not included
- ESLint warnings irrelevant
- No performance impact
- Build process removes development-only features

## Common Development-Only Features

```javascript
// Only runs in development
if (process.env.NODE_ENV === 'development') {
  console.log('Debug info');
}

// Development-only error checks
if (process.env.NODE_ENV === 'development') {
  if (!props.required) {
    console.warn('Missing required prop');
  }
}
```

## ESLint Development Rules Examples
```javascript
// .eslintrc.cjs
module.exports = {
  overrides: [
    // Disable rules for test files
    {
      files: ['**/*.test.js'],
      rules: {
        'no-console': 'off'
      }
    },
    // Specific rules for development utilities
    {
      files: ['src/utils/debug/*.js'],
      rules: {
        'no-console': 'off',
        'no-debugger': 'off'
      }
    },
    // Context file rules
    {
      files: ["src/context/*.jsx"],
      rules: {
        'react-refresh/only-export-components': 'off'
      }
    }
  ]
}
```
> Different rules can be applied to different files based on development needs.

## Development Environment Checks
```javascript
// Utility for development-only code
const isDevelopment = process.env.NODE_ENV === 'development';

// Usage
if (isDevelopment) {
  // Development-only code
  setupDevTools();
  enableDebugLogging();
}

// Development-only component
const DevTools = isDevelopment ? require('./DevTools').default : () => null;
```

## Best Practices

1. **ESLint Configuration**
   - Keep development-specific rules in overrides
   - Target specific directories when disabling rules
   - Document why rules are disabled

2. **Development vs Production**
   - Use environment checks for development-only code
   - Keep development utilities separate
   - Don't rely on development-only features for core functionality

3. **Fast Refresh**
   - Understand which patterns break Fast Refresh
   - Accept slower refresh for maintainable code structure
   - Use ESLint overrides judiciously

4. **Code Organization**
   ```plaintext
   src/
   ├── components/     # Fast Refresh enabled
   ├── context/       # Fast Refresh warnings disabled
   ├── utils/         # Regular code
   └── dev/          # Development-only utilities
   ```

Remember:
- Fast Refresh warnings are development-only concerns
- Production builds are not affected by these ESLint rules
- Balance development experience with code organization
- Use environment checks for development-only features