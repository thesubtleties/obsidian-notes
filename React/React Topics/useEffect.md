## Overview

useEffect is a crucial React Hook that manages side effects in functional components. It serves as a replacement for lifecycle methods in class components, allowing you to perform operations like data fetching, subscriptions, or manually changing the DOM. The hook takes two arguments: a function where you put your effect code, and an optional array of dependencies that control when the effect runs. useEffect runs after every render by default, but you can optimize its behavior by specifying dependencies. Understanding useEffect is key to building responsive, efficient React applications that can interact with external systems and handle complex state management scenarios.

## Basic Syntax

```javascript
useEffect(() => {
  // Effect code
  return () => {
    // Cleanup code (optional)
  };
}, [dependencies]);
```

## Key Points

1. Runs after every render by default
2. Second argument (dependency array) controls when effect runs
3. Return function is for cleanup (optional)

## Common Use Cases

### 1. Run on Every Render
```javascript
useEffect(() => {
  console.log('I run after every render');
});
```

### 2. Run Only Once (On Mount)
```javascript
useEffect(() => {
  console.log('I run only once');
}, []);
```

### 3. Run on Specific Prop/State Changes
```javascript
useEffect(() => {
  console.log('I run when count changes');
}, [count]);
```

### 4. Cleanup on Unmount
```javascript
useEffect(() => {
  const timer = setInterval(() => {
    // do something
  }, 1000);

  return () => clearInterval(timer);
}, []);
```

## Advanced Patterns

### 1. Fetching Data
```javascript
useEffect(() => {
  let isMounted = true;
  const fetchData = async () => {
    const result = await api.getData();
    if (isMounted) setData(result);
  };
  fetchData();
  return () => { isMounted = false };
}, []);
```

### 2. Subscribing to Events
```javascript
useEffect(() => {
  const handleResize = () => setWindowWidth(window.innerWidth);
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```

### 3. Updating Document Title
```javascript
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]);
```

### 4. Debouncing
```javascript
useEffect(() => {
  const debounceTimer = setTimeout(() => {
    // do something with searchTerm
  }, 500);
  return () => clearTimeout(debounceTimer);
}, [searchTerm]);
```

## Best Practices

1. Don't omit the dependency array unless necessary
2. Include all variables used in the effect in the dependency array
3. Use multiple effects for unrelated logic
4. Avoid infinite loops by carefully managing dependencies
5. Use the eslint-plugin-react-hooks for catching mistakes

## Common Mistakes

1. Missing dependencies
2. Unnecessary dependencies
3. Forgetting to clean up subscriptions or timers
4. Using stale closures (not updating effect when props/state change)

## Tips

1. Use the exhaustive-deps ESLint rule
2. Consider extracting complex logic into custom hooks
3. Use useCallback for function dependencies to prevent unnecessary re-renders
4. Remember that effects run after render, not during

## Async Effects

```javascript
useEffect(() => {
  const fetchData = async () => {
    const result = await api.getData();
    setData(result);
  };
  fetchData();
}, []);
```

Note: Don't make the effect function itself async. Instead, define an async function inside the effect and call it.

Remember, `useEffect` is a powerful tool for handling side effects in functional components. Use it wisely to manage component lifecycle, data fetching, subscriptions, and more.