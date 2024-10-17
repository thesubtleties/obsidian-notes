useRef is a React Hook that provides a way to create mutable references that persist across re-renders. It's commonly used to access DOM elements directly, store mutable values that don't require re-renders, or hold onto previous values.

## Basic Syntax

```javascript
const refContainer = useRef(initialValue);
```

## Key Concepts

1. Returns a mutable ref object with a .current property
2. The returned object persists for the full lifetime of the component
3. Changing ref.current doesn't cause a re-render

## Common Use Cases

### 1. Accessing DOM Elements
```javascript
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

### 2. Storing Mutable Values
```javascript
function Timer() {
  const intervalRef = useRef();
  useEffect(() => {
    intervalRef.current = setInterval(() => {
      // do something
    }, 1000);
    return () => {
      clearInterval(intervalRef.current);
    };
  }, []);
  // ...
}
```

### 3. Preserving Previous Values
```javascript
function Counter() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();
  useEffect(() => {
    prevCountRef.current = count;
  });
  const prevCount = prevCountRef.current;
  return <h1>Now: {count}, before: {prevCount}</h1>;
}
```

## Advanced Patterns

### 1. Callback Refs
```javascript
function MeasureExample() {
  const [height, setHeight] = useState(0);

  const measuredRef = useCallback(node => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);

  return (
    <>
      <h1 ref={measuredRef}>Hello, world</h1>
      <h2>The above header is {Math.round(height)}px tall</h2>
    </>
  );
}
```

### 2. Forwarding Refs
```javascript
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// Usage
function Parent() {
  const ref = useRef(null);
  return <FancyButton ref={ref}>Click me!</FancyButton>;
}
```

### 3. Caching Expensive Computations
```javascript
function ExpensiveComponent({ data }) {
  const expensiveResultRef = useRef(null);
  
  if (expensiveResultRef.current === null) {
    expensiveResultRef.current = expensiveComputation(data);
  }

  return <div>{expensiveResultRef.current}</div>;
}
```

## Best Practices

1. Use useRef for values that don't affect the visual output of your component
2. Avoid overusing refs - often state or derived values are more appropriate
3. Don't use refs to manipulate DOM in ways that conflict with React's rendering
4. Remember that changing ref.current doesn't trigger a re-render

## Common Mistakes

1. Trying to use ref.current in render calculations (it won't trigger updates)
2. Forgetting that ref.current might be null initially
3. Using useRef when useState would be more appropriate
4. Directly manipulating DOM when React declarative approach would work

## Tips

1. Use TypeScript to add type safety to your refs:
   ```typescript
   const inputRef = useRef<HTMLInputElement>(null);
   ```
2. Combine useRef with useEffect for side effects that need access to the latest props or state
3. Use refs to store timeouts or intervals that need to be cleared
4. Remember that refs are escaped hatches - use them sparingly

## Examples

### Implementing a Stopwatch
```javascript
function Stopwatch() {
  const [time, setTime] = useState(0);
  const intervalRef = useRef(null);

  const start = () => {
    if (intervalRef.current !== null) return;
    intervalRef.current = setInterval(() => {
      setTime(t => t + 1);
    }, 1000);
  };

  const stop = () => {
    clearInterval(intervalRef.current);
    intervalRef.current = null;
  };

  const reset = () => {
    stop();
    setTime(0);
  };

  useEffect(() => {
    return () => clearInterval(intervalRef.current);
  }, []);

  return (
    <div>
      <div>Time: {time}</div>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

Remember, useRef is a powerful tool for working with mutable values and DOM elements in React, but it should be used judiciously. In most cases, React's declarative approach with state and props is preferable.