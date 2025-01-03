## Import Types

1. Default Import
```javascript
import React from 'react';
```
- Used when a module exports a single main thing
- No curly braces
- Can be renamed during import

2. Named Import
```javascript
import { useState, useEffect } from 'react';
```
- Used for specific exports from a module
- Uses curly braces
- Must use the exact name (unless renamed)

3. Renamed Import
```javascript
import { useState as useStateHook } from 'react';
```
- Allows you to rename a named import
- Useful to avoid naming conflicts

4. Namespace Import
```javascript
import * as ReactRedux from 'react-redux';
```
- Imports all exports from a module as properties of an object
- Useful when you need many exports from a module

5. Side Effect Import
```javascript
import './index.css';
```
- Doesn't import any value
- Used for scripts that run when imported

## Common React and Redux Imports

1. React Core
```javascript
import React, { useState, useEffect } from 'react';
```

2. React DOM
```javascript
import ReactDOM from 'react-dom/client';
```

3. Redux
```javascript
import { createStore } from 'redux';
```

4. React-Redux
```javascript
import { Provider, useSelector, useDispatch } from 'react-redux';
```

5. Redux Toolkit
```javascript
import { configureStore, createSlice } from '@reduxjs/toolkit';
```

## Key Points

- React components are typically default exports
- Hooks (useState, useEffect, etc.) are named exports
- Most Redux functions (createStore, combineReducers, etc.) are named exports
- The Provider component from react-redux is a named export
- Custom hooks and utility functions are often named exports

## Gotchas

- Some libraries use default exports where you might expect named exports (and vice versa)
- Always check the documentation or source code if you're unsure
- Be cautious with namespace imports (*) as they can bloat your bundle size

Remember: The type of import you use depends on how the module you're importing from has exported its contents. Always match the import style to the export style of the module.