Custom hooks are a powerful feature in React that allow you to extract component logic into reusable functions. For more information: [[React Hooks]]

## Basic Syntax

```javascript
function useCustomHook(initialValue) {
  // Hook logic here
  return [value, setValue];
}
```
This is the basic structure of a custom hook. It's a function that can use other hooks and returns values that can be used in components.

## Common Use Cases

### 1. Managing Form State
```javascript
function useFormInput(initialValue) {
  const [value, setValue] = useState(initialValue);
  const handleChange = e => setValue(e.target.value);
  return { value, onChange: handleChange };
}

// Usage
const nameInput = useFormInput('');
<input {...nameInput} />
```
This hook simplifies form input handling by managing the state of an input field and providing a change handler. It returns an object that can be spread directly onto an input element.

### 2. Fetching Data
```javascript
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(data => setData(data))
      .catch(err => setError(err))
      .finally(() => setLoading(false));
  }, [url]);

  return { data, loading, error };
}

// Usage
const { data, loading, error } = useFetch('https://api.example.com/data');
```
This hook encapsulates the logic for fetching data from an API. It manages loading and error states, making it easy to handle asynchronous operations in components.

### 3. Managing Window Events
```javascript
function useWindowSize() {
  const [size, setSize] = useState({ width: window.innerWidth, height: window.innerHeight });

  useEffect(() => {
    const handleResize = () => setSize({ width: window.innerWidth, height: window.innerHeight });
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}

// Usage
const { width, height } = useWindowSize();
```
This hook tracks the window size and updates when it changes. It demonstrates how to work with browser APIs and clean up event listeners in useEffect.

## Advanced Patterns

### 1. Composing Hooks
```javascript
function useCustomComposedHook() {
  const [count, setCount] = useState(0);
  const size = useWindowSize();

  // Combine logic from multiple hooks
  return { count, setCount, size };
}
```
This example shows how to compose multiple hooks into a single custom hook, combining their functionalities.

### 2. Memoization in Custom Hooks
```javascript
function useMemoizedCallback(callback, dependencies) {
  return useCallback(callback, dependencies);
}

// Usage
const memoizedFn = useMemoizedCallback(() => {
  // Complex computation
}, [dep1, dep2]);
```
This hook demonstrates how to use memoization within custom hooks, which is useful for optimizing performance by preventing unnecessary re-renders.

### 3. Using with TypeScript
```typescript
function useCounter<T>(initialValue: T): [T, () => void] {
  const [count, setCount] = useState<T>(initialValue);
  const increment = () => setCount((prevCount: T) => prevCount + 1);
  return [count, increment];
}

// Usage
const [count, increment] = useCounter<number>(0);
```
This example shows how to use TypeScript with custom hooks, providing type safety and better developer experience.

## Examples

### Debounce Hook
```javascript
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}

// Usage
const debouncedSearchTerm = useDebounce(searchTerm, 500);
```
This hook implements debouncing, which is useful for delaying the execution of a function until after a certain amount of time has passed since the last call. It's commonly used for search inputs to reduce API calls.

### Local Storage Hook
```javascript
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.log(error);
      return initialValue;
    }
  });

  const setValue = value => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.log(error);
    }
  };

  return [storedValue, setValue];
}

// Usage
const [name, setName] = useLocalStorage('name', 'Bob');
```
This hook provides a way to use localStorage with a React-like state interface. It syncs the state with localStorage, allowing for persistent storage across page reloads.

### Media Query Hook
```javascript
function useMedia(query) {
  const [matches, setMatches] = useState(window.matchMedia(query).matches);

  useEffect(() => {
    const media = window.matchMedia(query);
    if (media.matches !== matches) setMatches(media.matches);
    const listener = () => setMatches(media.matches);
    media.addListener(listener);
    return () => media.removeListener(listener);
  }, [query]);

  return matches;
}

// Usage
const isWideScreen = useMedia('(min-width: 1200px)');
```
This hook allows components to respond to media queries. It's useful for creating responsive designs that can adapt to different screen sizes or device capabilities.

These explanations should provide more context and understanding for each custom hook example.