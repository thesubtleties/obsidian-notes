## 1. useMemo Hook (React)

Use Case: Memoizing computed values in functional components.

```javascript
import React, { useMemo } from 'react';
import { useSelector } from 'react-redux';

function MyComponent() {
  const data = useSelector(state => state.someData);
  
  const processedData = useMemo(() => {
    return expensiveComputation(data);
  }, [data]);

  return <div>{processedData}</div>;
}
```

Key Points:
- Recalculates only when dependencies change
- Use for expensive computations or to maintain referential equality

## 2. Reselect Library

Use Case: Creating memoized selectors for Redux state.

```javascript
import { createSelector } from 'reselect';

const selectItems = state => state.items;
const selectFilter = state => state.filter;

export const selectFilteredItems = createSelector(
  [selectItems, selectFilter],
  (items, filter) => items.filter(item => item.type === filter)
);

// Usage in component
const filteredItems = useSelector(selectFilteredItems);
```

Key Points:
- Efficient for derived data from Redux state
- Recomputes only when input selectors change
- Great for complex calculations or filtering

## 3. Manual Memoization

Use Case: When you can't use libraries or need custom control.

```javascript
function memoize(func) {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = func(...args);
    cache.set(key, result);
    return result;
  };
}

const memoizedSelector = memoize((state, props) => {
  // Expensive computation here
});
```

Key Points:
- Flexible but requires careful implementation
- Consider cache invalidation strategies for long-lived memoizations

## 4. React.memo (for Components)

Use Case: Preventing unnecessary re-renders of functional components.

```javascript
const MyComponent = React.memo(function MyComponent(props) {
  // Component logic
});

// With custom comparison
const MyComponent = React.memo(function MyComponent(props) {
  // Component logic
}, (prevProps, nextProps) => {
  // Return true if passing nextProps to render would return
  // the same result as passing prevProps to render,
  // otherwise return false
});
```

Key Points:
- Shallow compares props by default
- Use custom comparison for complex props

## 5. useCallback Hook (React)

Use Case: Memoizing callbacks to maintain referential equality.

```javascript
import React, { useCallback } from 'react';

function MyComponent({ onSomeEvent }) {
  const memoizedCallback = useCallback(() => {
    doSomething(a, b);
  }, [a, b]);

  return <ChildComponent onClick={memoizedCallback} />;
}
```

Key Points:
- Useful when passing callbacks to optimized child components
- Helps prevent unnecessary re-renders

## 6. Memoization in Class Components

Use Case: When working with class components in React.

```javascript
class MyComponent extends React.Component {
  memoizedValue = memoize((a, b) => a + b);

  render() {
    const sum = this.memoizedValue(this.props.a, this.props.b);
    return <div>{sum}</div>;
  }
}
```

Key Points:
- Can use libraries like lodash's `memoize` or custom implementations
- Be cautious with memoization that persists for component lifetime

## 7. Redux Toolkit's createSelector

Use Case: Creating memoized selectors with Redux Toolkit.

```javascript
import { createSelector } from '@reduxjs/toolkit';

const selectItems = state => state.items;
const selectFilter = state => state.filter;

export const selectFilteredItems = createSelector(
  [selectItems, selectFilter],
  (items, filter) => items.filter(item => item.type === filter)
);
```

Key Points:
- Similar to Reselect, but integrated with Redux Toolkit
- Automatically uses correct equality checks

## Best Practices

1. Use memoization for expensive computations or to maintain referential equality.
2. Ensure dependency arrays are correct and complete.
3. Don't overuse – memoization itself has a cost.
4. For Redux, prefer selector memoization over in-component memoization when possible.
5. Use profiling tools to identify actual performance bottlenecks before optimizing.

Remember, premature optimization can lead to unnecessary complexity. Always measure performance before and after applying memoization techniques to ensure they're providing tangible benefits.