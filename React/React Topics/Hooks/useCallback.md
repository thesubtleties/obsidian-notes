useCallback is a React Hook that lets you cache a function definition between re-renders. For more information: [[React Hooks]]

## Basic Syntax

```javascript
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

## Key Concepts

1. Returns a memoized version of the callback function
2. Only changes if one of the dependencies has changed
3. Useful for optimizing child component re-renders
4. Works in conjunction with React.memo for function components

## Common Use Cases

### 1. Optimizing Child Component Renders
```javascript
const MemoizedChild = React.memo(ChildComponent);

function Parent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    setCount(prevCount => prevCount + 1);
  }, []); // Empty dependency array means this function never changes

  return <MemoizedChild onClick={handleClick} />;
}
```
This prevents unnecessary re-renders of ChildComponent when Parent re-renders.

### 2. Avoiding Infinite Loops in useEffect
```javascript
function SearchComponent() {
  const [query, setQuery] = useState('');

  const debouncedSearch = useCallback(
    debounce(q => {
      // Perform search operation
    }, 300),
    [] // Empty dependency array
  );

  useEffect(() => {
    debouncedSearch(query);
  }, [query, debouncedSearch]);

  // ...
}
```
Prevents the effect from running on every render due to a new function being created.

## Advanced Patterns

### 1. Dynamic Dependencies
```javascript
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b], // a and b are dependencies
);
```
The callback will only change if a or b changes.

### 2. Using with TypeScript
```typescript
const memoizedCallback = useCallback(
  (param: string): number => {
    return param.length;
  },
  []
);
```

## Best Practices

1. Use useCallback when passing callbacks to optimized child components that rely on reference equality to prevent unnecessary renders
2. Include all values from the component scope that the callback function uses in the dependency array
3. Avoid using useCallback for every function in your component; use it judiciously where it provides clear performance benefits

## Common Mistakes

1. Forgetting to include all necessary dependencies in the dependency array
2. Over-optimizing by using useCallback for every function
3. Not using React.memo on child components when passing callbacks
4. Including unnecessary dependencies, leading to frequent callback regeneration

## Tips

1. Use the ESLint plugin for React Hooks to catch missing dependencies
2. Consider using useCallback in conjunction with useMemo for optimal performance
3. Profile your application to identify genuine performance bottlenecks before optimizing
4. Remember that useCallback itself has a performance cost, so use it wisely

## Examples

### Event Handlers in Lists
```javascript
function List({ items }) {
  const handleItemClick = useCallback((id) => {
    console.log('Item clicked:', id);
  }, []); // No dependencies, function remains the same

  return (
    <ul>
      {items.map(item => (
        <ListItem 
          key={item.id} 
          item={item} 
          onClick={() => handleItemClick(item.id)} 
        />
      ))}
    </ul>
  );
}

const ListItem = React.memo(({ item, onClick }) => {
  // Component implementation
});
```
This pattern is useful when rendering large lists where each item has an event handler.

### Memoized Callbacks with Props
```javascript
function ParentComponent({ someProps }) {
  const memoizedCallback = useCallback(
    () => {
      doSomethingWith(someProps);
    },
    [someProps] // Callback changes only when someProps changes
  );

  return <ChildComponent callback={memoizedCallback} />;
}
```
This ensures that the callback passed to ChildComponent only changes when necessary.

Remember, useCallback is a tool for optimization. Always measure performance before and after applying useCallback to ensure it's providing the expected benefits in your specific use case.