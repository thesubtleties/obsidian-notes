## Mounting Phase

### Class Components (Old)
```javascript
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { data: null };
  }

  componentDidMount() {
    // Run once when component mounts
    fetchData();
  }
}
```

### Function Components (Modern)
```javascript
function MyComponent() {
  const [data, setData] = useState(null);

  // Constructor equivalent
  // Just declare your state with useState

  // componentDidMount equivalent
  useEffect(() => {
    fetchData();
  }, []); // Empty dependency array = run once on mount
}
```

## Updating Phase

### Class Components (Old)
```javascript
class MyComponent extends React.Component {
  componentDidUpdate(prevProps, prevState) {
    if (prevProps.userId !== this.props.userId) {
      // Run when props/state changes
      fetchUserData(this.props.userId);
    }
  }
}
```

### Function Components (Modern)
```javascript
function MyComponent({ userId }) {
  // componentDidUpdate equivalent
  useEffect(() => {
    fetchUserData(userId);
  }, [userId]); // Run when userId changes

  // No dependency array = run on every render
  useEffect(() => {
    // Runs after every render
  });
}
```

## Unmounting Phase

### Class Components (Old)
```javascript
class MyComponent extends React.Component {
  componentWillUnmount() {
    // Cleanup before component is destroyed
    clearInterval(this.interval);
  }
}
```

### Function Components (Modern)
```javascript
function MyComponent() {
  useEffect(() => {
    // Setup
    const interval = setInterval(() => {
      // Do something
    }, 1000);

    // componentWillUnmount equivalent
    return () => {
      clearInterval(interval);
    };
  }, []); // Empty array = cleanup runs only on unmount
}
```

## Common Patterns

### Data Fetching

#### Old Way
```javascript
class MyComponent extends React.Component {
  state = { data: null, loading: true };

  componentDidMount() {
    this.fetchData();
  }

  componentDidUpdate(prevProps) {
    if (prevProps.id !== this.props.id) {
      this.fetchData();
    }
  }

  fetchData = async () => {
    const data = await fetch(`/api/${this.props.id}`);
    this.setState({ data, loading: false });
  };
}
```

#### Modern Way
```javascript
function MyComponent({ id }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchData = async () => {
      const result = await fetch(`/api/${id}`);
      setData(result);
      setLoading(false);
    };

    fetchData();
  }, [id]);
}
```

## Quick Reference

### Lifecycle to Hooks Mapping

```javascript
// Constructor → useState
constructor(props) {}        → const [state, setState] = useState(initial);

// componentDidMount → useEffect with empty array
componentDidMount() {}       → useEffect(() => {}, [])

// componentDidUpdate → useEffect with dependencies
componentDidUpdate() {}      → useEffect(() => {}, [dep1, dep2])

// componentWillUnmount → useEffect cleanup
componentWillUnmount() {}    → useEffect(() => { return () => {} }, [])

// getDerivedStateFromProps → useEffect
static getDerivedStateFromProps() → useEffect(() => {
                                    setState(deriveState(props));
                                  }, [props])

// shouldComponentUpdate → React.memo
shouldComponentUpdate(nextProps) → const MemoizedComponent = React.memo(Component)
```

## Common Use Cases

### Event Listeners
```javascript
// Old
componentDidMount() {
  window.addEventListener('resize', this.handleResize);
}
componentWillUnmount() {
  window.removeEventListener('resize', this.handleResize);
}

// Modern
useEffect(() => {
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```

### Subscriptions
```javascript
// Old
componentDidMount() {
  DataSource.addSubscription(this.handleChange);
}
componentWillUnmount() {
  DataSource.removeSubscription(this.handleChange);
}

// Modern
useEffect(() => {
  DataSource.addSubscription(handleChange);
  return () => DataSource.removeSubscription(handleChange);
}, []);
```

Remember:
- Hooks must be called at the top level of your function
- Hooks can't be called conditionally
- Multiple useEffects in one component is okay and encouraged for separation of concerns
- The dependency array is key to controlling when effects run
- Empty array = once on mount
- No array = every render
- With dependencies = when those values change