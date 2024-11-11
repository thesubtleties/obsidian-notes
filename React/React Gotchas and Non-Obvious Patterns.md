## Event Handlers

### 1. Function Calls vs Function References
```javascript
// ❌ Wrong - Calls function immediately
<button onClick={doSomething()}>Click</button>

// ✅ Correct - Passes function reference
<button onClick={doSomething}>Click</button>

// ✅ Correct - Passes arrow function
<button onClick={() => doSomething()}>Click</button>
```

### 2. Passing Parameters
```javascript
// ❌ Wrong - Calls immediately
<button onClick={handleClick(id)}>Click</button>

// ✅ Correct ways:
<button onClick={() => handleClick(id)}>Click</button>
<button onClick={(e) => handleClick(id, e)}>Click</button>

// Alternative using bind
<button onClick={handleClick.bind(null, id)}>Click</button>
```

## State Updates

### 1. State Updates are Batched
```javascript
// ❌ Won't work as expected
function increment() {
  setCount(count + 1);
  setCount(count + 1);
} // Only increases by 1

// ✅ Use functional update
function increment() {
  setCount(prev => prev + 1);
  setCount(prev => prev + 1);
} // Increases by 2
```

### 2. Object State Updates
```javascript
// ❌ Wrong - Incomplete object update
setState({ name: 'New Name' }); // Removes other properties

// ✅ Correct - Spread existing state
setState(prev => ({ ...prev, name: 'New Name' }));

// ✅ Alternative using multiple state variables
const [name, setName] = useState('');
const [age, setAge] = useState(0);
```

## useEffect Dependencies

### 1. Empty Dependencies
```javascript
// ❌ Missing dependencies warning
useEffect(() => {
  console.log(props.value);
}, []); // ESLint warning if props.value is used

// ✅ Correct
useEffect(() => {
  console.log(props.value);
}, [props.value]);

// 🤔 If you really need empty deps, explain why
useEffect(() => {
  // Only on mount
  const listener = () => {};
  window.addEventListener('resize', listener);
  return () => window.removeEventListener('resize', listener);
}, []); // OK because it's truly mount/unmount only
```

### 2. Function Dependencies
```javascript
// ❌ Problematic - function recreated every render
useEffect(() => {
  doSomething();
}, [doSomething]); // Causes infinite loop

// ✅ Use useCallback for function deps
const doSomething = useCallback(() => {
  // function body
}, [/* deps */]);

// ✅ Or move function inside effect
useEffect(() => {
  function doSomething() {
    // function body
  }
  doSomething();
}, [/* deps */]);
```

## Conditional Rendering

### 1. Conditional JSX
```javascript
// ❌ Verbose
{isLoading ? <Loading /> : null}

// ✅ Cleaner
{isLoading && <Loading />}

// ⚠️ Be careful with numbers
{count && <Display />} // Won't render if count is 0

// ✅ Better for numbers
{count > 0 && <Display />}
```

### 2. Multiple Conditions
```javascript
// ❌ Hard to read
{isLoading ? (
  <Loading />
) : error ? (
  <Error />
) : data ? (
  <Data />
) : null}

// ✅ More readable
{isLoading && <Loading />}
{error && <Error />}
{!isLoading && !error && data && <Data />}

// ✅ Or use early returns in component
if (isLoading) return <Loading />;
if (error) return <Error />;
if (!data) return null;
return <Data />;
```

## Props and Destructuring

### 1. Props Destructuring
```javascript
// ❌ Repetitive
function Component(props) {
  return <div>{props.name} {props.age}</div>;
}

// ✅ Destructure in parameters
function Component({ name, age }) {
  return <div>{name} {age}</div>;
}

// ✅ Destructure with defaults
function Component({ name = 'Anonymous', age = 0 }) {
  return <div>{name} {age}</div>;
}
```

### 2. Rest Props
```javascript
// ✅ Forwarding DOM props
function Button({ className, children, ...rest }) {
  return (
    <button 
      className={`btn ${className}`} 
      {...rest}
    >
      {children}
    </button>
  );
}
```

## Component Organization

### 1. Conditional Class Names
```javascript
// ❌ Messy
<div className={`base ${isActive ? 'active' : ''} ${isDisabled ? 'disabled' : ''}`}>

// ✅ Using classnames library
import classNames from 'classnames';

<div className={classNames('base', {
  'active': isActive,
  'disabled': isDisabled
})}>

// ✅ Or template literal with functions
<div className={`base ${isActive && 'active'} ${isDisabled && 'disabled'}`}>
```

### 2. Component Logic Organization
```javascript
// ✅ Group related state
const [isLoading, setIsLoading] = useState(false);
const [error, setError] = useState(null);
const [data, setData] = useState(null);

// ✅ Group related handlers
const handleSubmit = async (e) => {
  e.preventDefault();
  setIsLoading(true);
  try {
    const result = await submitData();
    setData(result);
  } catch (err) {
    setError(err);
  } finally {
    setIsLoading(false);
  }
};
```

## Performance Optimization

### 1. Memoization
```javascript
// ✅ Memoize expensive calculations
const expensiveValue = useMemo(() => 
  compute(prop1, prop2), 
  [prop1, prop2]
);

// ✅ Memoize callbacks
const handleClick = useCallback(() => {
  doSomething(prop);
}, [prop]);

// ✅ Memoize components
const MemoizedChild = memo(ChildComponent);
```

### 2. Render Optimization
```javascript
// ❌ Creates new object every render
<Component style={{ margin: 10 }} />

// ✅ Move static styles outside
const styles = { margin: 10 };
<Component style={styles} />

// ❌ New array/object every render
<Component items={[1, 2, 3]} />

// ✅ Move to state or constant
const items = [1, 2, 3];
<Component items={items} />
```

## Common Mistakes to Watch For

1. Not cleaning up effects
2. Mutating state directly
3. Using hooks conditionally
4. Forgetting key prop in lists
5. Not handling loading/error states
6. Over-optimizing too early

Remember:
- Always use functions for event handlers
- Be careful with dependency arrays
- Clean up side effects
- Handle all possible states
- Think about component re-renders
- Use TypeScript for better error catching