## Quick Links
- [[#State Management: Class State vs useState]]
- [[#Complex State Management: Class State vs useReducer]]
- [[#Lifecycle Methods: ComponentDidMount vs useEffect]]
- [[#Lifecycle Updates: ComponentDidUpdate vs useEffect Dependencies]]
- [[#Lifecycle Cleanup: ComponentWillUnmount vs useEffect Cleanup]]
- [[#Refs and DOM Manipulation: createRef vs useRef]]
- [[#Context Usage: Context Consumer vs useContext]]
- [[#Performance Optimization: shouldComponentUpdate vs React.memo]]
- [[#Testing Patterns: Class Component Testing vs Hook Testing]]
- [[#Side Effects and Data Fetching: Class Methods vs useEffect]]
- [[#Component Composition: Render Props vs Custom Hooks]]
- [[#Animation Handling: Class Animations vs Hook Animations]]
- [[#Routing Integration: Class Router vs Modern Hooks]]
- [[#State Management with Redux: Classic Redux vs Redux Toolkit]]
- [[#Styling Patterns: Classic CSS vs Modern CSS-in-JS]]
- [[#Error Handling: Error Boundaries vs Error Hooks]]

## Not Yet Covered
- [[#Advanced Patterns: Compound Components and Render Props]]
- [[#React 18 Features: Concurrent Mode and Suspense]]
- [[#Data Management: REST vs GraphQL Approaches]]
- [[#TypeScript Integration: Types and Generics]]
- [[#Server-Side Rendering: Classic vs Modern Approaches]]
## State Management: Class State vs useState

```jsx
// Classic: Class Component State
class UserForm extends React.Component {
  constructor(props) {
    super(props);
    // Need to initialize all state properties upfront
    this.state = {
      username: '',
      email: '',
      isValid: false
    };
    // Need to bind methods to access 'this'
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange(e) {
    // Must use setState, can't modify state directly
    this.setState({ 
      [e.target.name]: e.target.value 
    }, () => {
      // State updates are batched, callback ensures latest state
      this.validateForm();
    });
  }

  render() {
    return (
      <form>
        <input
          name="username"
          value={this.state.username}
          onChange={this.handleChange}
        />
      </form>
    );
  }
}

// Modern: useState Hook
function UserForm() {
  // Can declare state variables individually
  const [username, setUsername] = useState('');
  const [email, setEmail] = useState('');
  const [isValid, setIsValid] = useState(false);

  // No binding needed, functions are scoped to component
  const handleChange = (e) => {
    const { name, value } = e.target;
    // Direct state setters, more intuitive
    if (name === 'username') setUsername(value);
    if (name === 'email') setEmail(value);
    
    // State updates are automatically batched in React 18
    validateForm();
  };

  // Can extract logic into separate functions easily
  const validateForm = () => {
    setIsValid(username.length > 0 && email.includes('@'));
  };

  return (
    <form>
      <input
        name="username"
        value={username}
        onChange={handleChange}
      />
    </form>
  );
}
```

**Key Benefits of useState:**
1. More granular state updates
2. No need for binding
3. Simpler syntax for updates
4. Better code organization
5. Easier to extract and reuse logic
6. More predictable behavior with state updates

## Complex State Management: Class State vs useReducer

```jsx
// Classic: Complex Class State
class ShoppingCart extends React.Component {
  state = {
    items: [],
    total: 0,
    loading: false,
    error: null
  };

  addItem = (item) => {
    this.setState(prevState => ({
      items: [...prevState.items, item],
      total: prevState.total + item.price
    }));
  };

  removeItem = (itemId) => {
    this.setState(prevState => {
      const item = prevState.items.find(i => i.id === itemId);
      return {
        items: prevState.items.filter(i => i.id !== itemId),
        total: prevState.total - item.price
      };
    });
  };
}

// Modern: useReducer Hook
function ShoppingCart() {
  // Define reducer outside component (can be in separate file)
  const cartReducer = (state, action) => {
    switch (action.type) {
      case 'ADD_ITEM':
        return {
          ...state,
          items: [...state.items, action.payload],
          total: state.total + action.payload.price
        };
      case 'REMOVE_ITEM':
        const item = state.items.find(i => i.id === action.payload);
        return {
          ...state,
          items: state.items.filter(i => i.id !== action.payload),
          total: state.total - item.price
        };
      default:
        return state;
    }
  };

  // Initialize with useReducer
  const [state, dispatch] = useReducer(cartReducer, {
    items: [],
    total: 0,
    loading: false,
    error: null
  });

  // Action creators (optional but recommended)
  const addItem = (item) => {
    dispatch({ type: 'ADD_ITEM', payload: item });
  };

  const removeItem = (itemId) => {
    dispatch({ type: 'REMOVE_ITEM', payload: itemId });
  };

  return (
    <div>
      {/* Cart UI */}
    </div>
  );
}
```

**Benefits of useReducer:**
1. Centralized state logic
2. Easier testing of state changes
3. More predictable state updates
4. Better for complex state interactions
5. Familiar pattern for Redux developers
6. Easier debugging with action types

## Refs and DOM Manipulation: createRef vs useRef

```jsx
// Classic: React.createRef
class VideoPlayer extends React.Component {
  constructor(props) {
    super(props);
    // Must create ref in constructor
    this.videoRef = React.createRef();
    this.intervalRef = null; // For non-DOM refs
  }

  componentDidMount() {
    // DOM node is accessed through .current
    this.videoRef.current.play();
    
    // Setting up interval
    this.intervalRef = setInterval(() => {
      this.checkVideoProgress();
    }, 1000);
  }

  componentWillUnmount() {
    // Need to clean up interval
    if (this.intervalRef) {
      clearInterval(this.intervalRef);
    }
  }

  checkVideoProgress() {
    const video = this.videoRef.current;
    if (video.currentTime >= video.duration) {
      this.handleVideoEnd();
    }
  }

  render() {
    return <video ref={this.videoRef} src={this.props.src} />;
  }
}

// Modern: useRef Hook
function VideoPlayer({ src }) {
  // Create ref with initial value null
  const videoRef = useRef(null);
  // Can also use useRef for mutable values that don't trigger re-renders
  const intervalRef = useRef(null);

  useEffect(() => {
    // Access DOM node through .current
    videoRef.current.play();

    // Store interval ID in ref
    intervalRef.current = setInterval(() => {
      checkVideoProgress();
    }, 1000);

    // Cleanup function
    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, []); // Empty deps array = run once on mount

  const checkVideoProgress = () => {
    const video = videoRef.current;
    if (video.currentTime >= video.duration) {
      handleVideoEnd();
    }
  };

  return <video ref={videoRef} src={src} />;
}
```

**Benefits of useRef:**
1. Simpler syntax
2. Can be declared anywhere in component
3. Perfect for both DOM refs and mutable values
4. No need for constructor
5. Works well with useEffect for cleanup
6. More flexible usage patterns

## Context Usage: Context Consumer vs useContext

```jsx
// Classic: Context Consumer
// First, create context
const ThemeContext = React.createContext('light');

class ThemedButton extends React.Component {
  render() {
    return (
      <ThemeContext.Consumer>
        {theme => ( // Requires render prop pattern
          <button className={`btn-${theme}`}>
            {this.props.children}
          </button>
        )}
      </ThemeContext.Consumer>
    );
  }
}

// For multiple contexts
class ComplexComponent extends React.Component {
  render() {
    return (
      <ThemeContext.Consumer>
        {theme => (
          <UserContext.Consumer>
            {user => (
              <LanguageContext.Consumer>
                {language => (
                  // Nested render props lead to "pyramid of doom"
                  <div className={`theme-${theme}`}>
                    {user.name} - {language}
                  </div>
                )}
              </LanguageContext.Consumer>
            )}
          </UserContext.Consumer>
        )}
      </ThemeContext.Consumer>
    );
  }
}

// Modern: useContext Hook
function ThemedButton({ children }) {
  // Direct access to context value
  const theme = useContext(ThemeContext);
  
  return (
    <button className={`btn-${theme}`}>
      {children}
    </button>
  );
}

// Multiple contexts are much cleaner
function ComplexComponent() {
  const theme = useContext(ThemeContext);
  const user = useContext(UserContext);
  const language = useContext(LanguageContext);

  return (
    <div className={`theme-${theme}`}>
      {user.name} - {language}
    </div>
  );
}

// Modern Context Provider Pattern
function App() {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={theme}>
      <div className="app">
        <ThemedButton onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
          Toggle Theme
        </ThemedButton>
      </div>
    </ThemeContext.Provider>
  );
}
```

**Benefits of useContext:**
1. Cleaner syntax
2. No render props needed
3. Multiple contexts without nesting
4. Better TypeScript support
5. Easier to combine with other hooks
6. More readable code
7. Better performance (avoids unnecessary nesting)

## Side Effects and Data Fetching: Component Lifecycle vs useEffect

```jsx
// Classic: Multiple Lifecycle Methods for Data Fetching
class UserDashboard extends React.Component {
  state = {
    userData: null,
    loading: true,
    error: null,
    subscription: null
  };

  componentDidMount() {
    // Initial data fetch
    this.fetchUserData();
    // Set up subscriptions
    this.subscription = api.subscribe(this.handleUpdates);
  }

  componentDidUpdate(prevProps) {
    // Check if we need to re-fetch
    if (prevProps.userId !== this.props.userId) {
      this.fetchUserData();
    }
  }

  componentWillUnmount() {
    // Clean up subscriptions
    if (this.subscription) {
      this.subscription.unsubscribe();
    }
    // Cancel any pending requests
    if (this.abortController) {
      this.abortController.abort();
    }
  }

  fetchUserData = async () => {
    this.setState({ loading: true });
    try {
      const data = await api.fetchUser(this.props.userId);
      this.setState({ userData: data, loading: false });
    } catch (error) {
      this.setState({ error, loading: false });
    }
  };

  render() {
    const { loading, error, userData } = this.state;
    if (loading) return <Loader />;
    if (error) return <ErrorMessage error={error} />;
    return <UserProfile data={userData} />;
  }
}

// Modern: useEffect with Custom Hook
function useUser(userId) {
  const [state, setState] = useState({
    userData: null,
    loading: true,
    error: null
  });

  useEffect(() => {
    // Create abort controller for cleanup
    const abortController = new AbortController();
    let mounted = true; // Flag to prevent updates after unmount

    const fetchData = async () => {
      setState(s => ({ ...s, loading: true }));
      try {
        const data = await api.fetchUser(userId, {
          signal: abortController.signal
        });
        // Check if component is still mounted
        if (mounted) {
          setState({
            userData: data,
            loading: false,
            error: null
          });
        }
      } catch (error) {
        if (error.name === 'AbortError') return;
        if (mounted) {
          setState({
            userData: null,
            loading: false,
            error
          });
        }
      }
    };

    fetchData();

    // Set up subscription
    const subscription = api.subscribe(handleUpdates);

    // Cleanup function
    return () => {
      mounted = false;
      abortController.abort();
      subscription.unsubscribe();
    };
  }, [userId]); // Re-run effect when userId changes

  return state;
}

// Modern Component Implementation
function UserDashboard({ userId }) {
  const { userData, loading, error } = useUser(userId);

  if (loading) return <Loader />;
  if (error) return <ErrorMessage error={error} />;
  return <UserProfile data={userData} />;
}
```

**Benefits of Modern Approach:**
1. Encapsulation of data fetching logic in custom hook
2. Automatic cleanup handling
3. Better handling of race conditions
4. Easier to test
5. Reusable across components
6. More declarative approach
7. Better error boundary integration

## Performance Optimization: shouldComponentUpdate vs React.memo and useMemo

```jsx
// Classic: shouldComponentUpdate
class ExpensiveList extends React.Component {
  shouldComponentUpdate(nextProps) {
    // Manual comparison of props
    return (
      nextProps.items.length !== this.props.items.length ||
      JSON.stringify(nextProps.items) !== JSON.stringify(this.props.items)
    );
  }

  render() {
    return (
      <ul>
        {this.props.items.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    );
  }
}

// Modern: React.memo with useMemo
const ExpensiveList = React.memo(function ExpensiveList({ items }) {
  // Memoize expensive calculations
  const sortedItems = useMemo(() => {
    return [...items].sort((a, b) => b.priority - a.priority);
  }, [items]);

  // Memoize event handlers if needed
  const handleItemClick = useCallback((id) => {
    console.log(`Clicked item ${id}`);
  }, []); // Empty deps array as it doesn't depend on props/state

  return (
    <ul>
      {sortedItems.map(item => (
        <li 
          key={item.id}
          onClick={() => handleItemClick(item.id)}
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
}, (prevProps, nextProps) => {
  // Optional custom comparison function
  return (
    prevProps.items.length === nextProps.items.length &&
    JSON.stringify(prevProps.items) === JSON.stringify(nextProps.items)
  );
});

// Usage with parent component
function ParentComponent() {
  const [items, setItems] = useState([]);
  
  // Memoize expensive prop calculations
  const processedItems = useMemo(() => {
    return items.map(item => ({
      ...item,
      processed: expensiveProcess(item)
    }));
  }, [items]);

  return <ExpensiveList items={processedItems} />;
}
```

**Benefits of Modern Optimization:**
1. More declarative optimization
2. Easier to implement
3. Better separation of concerns
4. More flexible memoization options
5. Built-in performance hooks
6. Better TypeScript support
7. Easier to maintain

## Error Handling: Error Boundaries vs Hooks Pattern

```jsx
// Classic: Error Boundary Class Component
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error) {
    // Update state to show fallback UI
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Log error to error reporting service
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <ErrorFallback error={this.state.error} />;
    }
    return this.props.children;
  }
}

// Modern: Error Boundary + Hook Pattern
// Note: Error Boundaries still need to be class components, but we can combine with hooks
function useErrorHandler() {
  const [error, setError] = useState(null);

  const handleError = useCallback((error) => {
    logErrorToService(error);
    setError(error);
  }, []);

  return [error, handleError];
}

// Usage in functional component
function UserProfile({ userId }) {
  const [error, handleError] = useErrorHandler();

  useEffect(() => {
    async function fetchUser() {
      try {
        const user = await fetchUserData(userId);
        // Handle success
      } catch (error) {
        handleError(error);
      }
    }
    fetchUser();
  }, [userId, handleError]);

  if (error) {
    return <ErrorFallback error={error} />;
  }

  return (
    <ErrorBoundary>
      {/* Component content */}
    </ErrorBoundary>
  );
}
```

## Event Handling: Class Methods vs Hooks

```jsx
// Classic: Class Methods and Binding
class SearchComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { query: '' };
    // Need to bind methods
    this.handleSearch = this.handleSearch.bind(this);
    this.handleInputChange = this.handleInputChange.bind(this);
  }

  handleSearch(e) {
    e.preventDefault();
    this.props.onSearch(this.state.query);
  }

  handleInputChange(e) {
    this.setState({ query: e.target.value });
  }

  // Class property syntax alternative
  handleKeyPress = (e) => {
    if (e.key === 'Enter') {
      this.handleSearch(e);
    }
  }

  render() {
    return (
      <form onSubmit={this.handleSearch}>
        <input
          value={this.state.query}
          onChange={this.handleInputChange}
          onKeyPress={this.handleKeyPress}
        />
      </form>
    );
  }
}

// Modern: Hooks with Event Handlers
function SearchComponent({ onSearch }) {
  const [query, setQuery] = useState('');
  
  // Memoize handlers if needed (for optimization)
  const handleSearch = useCallback((e) => {
    e.preventDefault();
    onSearch(query);
  }, [query, onSearch]);

  // Simple handlers don't need memoization
  const handleInputChange = (e) => {
    setQuery(e.target.value);
  };

  // Event handler with debounce
  const debouncedSearch = useMemo(() => 
    debounce((value) => {
      onSearch(value);
    }, 300)
  , [onSearch]);

  // Cleanup on unmount
  useEffect(() => {
    return () => {
      debouncedSearch.cancel();
    };
  }, [debouncedSearch]);

  return (
    <form onSubmit={handleSearch}>
      <input
        value={query}
        onChange={(e) => {
          handleInputChange(e);
          debouncedSearch(e.target.value);
        }}
      />
    </form>
  );
}
```

## Props Handling: HOCs vs Custom Hooks

```jsx
// Classic: Higher Order Component
function withUser(WrappedComponent) {
  return class WithUser extends React.Component {
    state = {
      user: null,
      loading: true
    };

    componentDidMount() {
      this.fetchUser();
    }

    fetchUser = async () => {
      const user = await fetchUserData();
      this.setState({ user, loading: false });
    };

    render() {
      const { user, loading } = this.state;
      return (
        <WrappedComponent
          user={user}
          loading={loading}
          {...this.props}
        />
      );
    }
  };
}

// Usage of HOC
const UserProfile = withUser(({ user, loading }) => {
  if (loading) return <Loader />;
  return <div>{user.name}</div>;
});

// Modern: Custom Hook
function useUser() {
  const [state, setState] = useState({
    user: null,
    loading: true
  });

  useEffect(() => {
    let mounted = true;

    async function loadUser() {
      const user = await fetchUserData();
      if (mounted) {
        setState({ user, loading: false });
      }
    }

    loadUser();

    return () => {
      mounted = false;
    };
  }, []);

  return state;
}

// Usage with Hook
function UserProfile() {
  const { user, loading } = useUser();

  if (loading) return <Loader />;
  return <div>{user.name}</div>;
}

// Composing multiple hooks
function useEnhancedUser() {
  const { user, loading } = useUser();
  const { permissions } = usePermissions(user?.id);
  const { preferences } = usePreferences(user?.id);

  return {
    user,
    loading,
    permissions,
    preferences,
    isAdmin: permissions?.includes('admin') ?? false
  };
}
```

**Benefits of Modern Patterns:**
1. More composable
2. Easier to test
3. Better type inference
4. Less boilerplate
5. More flexible
6. Easier to debug
7. Better separation of concerns

# Modern React vs Classic React Comprehensive Cheatsheet - Part 5

## Form Handling: Class Forms vs Modern Form Hooks

```jsx
// Classic: Form in Class Component
class RegistrationForm extends React.Component {
  state = {
    formData: {
      username: '',
      email: '',
      password: ''
    },
    errors: {},
    isSubmitting: false
  };

  handleChange = (e) => {
    const { name, value } = e.target;
    this.setState(prevState => ({
      formData: {
        ...prevState.formData,
        [name]: value
      }
    }));
  };

  validate = () => {
    const errors = {};
    const { username, email, password } = this.state.formData;

    if (!username) errors.username = 'Username is required';
    if (!email) errors.email = 'Email is required';
    if (!password) errors.password = 'Password is required';

    return errors;
  };

  handleSubmit = async (e) => {
    e.preventDefault();
    const errors = this.validate();

    if (Object.keys(errors).length > 0) {
      this.setState({ errors });
      return;
    }

    this.setState({ isSubmitting: true });
    try {
      await submitForm(this.state.formData);
      // Handle success
    } catch (error) {
      this.setState({ errors: { submit: error.message } });
    } finally {
      this.setState({ isSubmitting: false });
    }
  };

  render() {
    const { formData, errors, isSubmitting } = this.state;
    return (
      <form onSubmit={this.handleSubmit}>
        <input
          name="username"
          value={formData.username}
          onChange={this.handleChange}
        />
        {errors.username && <span>{errors.username}</span>}
        {/* Other fields */}
        <button disabled={isSubmitting}>Submit</button>
      </form>
    );
  }
}

// Modern: Custom Form Hook
function useForm(initialValues, validate) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [touched, setTouched] = useState({});

  // Handle field changes
  const handleChange = useCallback((e) => {
    const { name, value } = e.target;
    setValues(prev => ({
      ...prev,
      [name]: value
    }));
  }, []);

  // Handle field blur
  const handleBlur = useCallback((e) => {
    const { name } = e.target;
    setTouched(prev => ({
      ...prev,
      [name]: true
    }));
  }, []);

  // Validate form
  const validateForm = useCallback(() => {
    const validationErrors = validate(values);
    setErrors(validationErrors);
    return Object.keys(validationErrors).length === 0;
  }, [values, validate]);

  // Reset form
  const resetForm = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  }, [initialValues]);

  // Submit handler
  const handleSubmit = useCallback(async (onSubmit) => {
    setIsSubmitting(true);
    try {
      if (validateForm()) {
        await onSubmit(values);
        resetForm();
      }
    } catch (error) {
      setErrors(prev => ({
        ...prev,
        submit: error.message
      }));
    } finally {
      setIsSubmitting(false);
    }
  }, [values, validateForm, resetForm]);

  return {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    resetForm
  };
}

// Modern Form Implementation
function RegistrationForm() {
  const validate = useCallback((values) => {
    const errors = {};
    if (!values.username) errors.username = 'Required';
    if (!values.email) errors.email = 'Required';
    if (!values.password) errors.password = 'Required';
    return errors;
  }, []);

  const {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit
  } = useForm(
    { username: '', email: '', password: '' },
    validate
  );

  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      handleSubmit(submitForm);
    }}>
      <input
        name="username"
        value={values.username}
        onChange={handleChange}
        onBlur={handleBlur}
      />
      {touched.username && errors.username && (
        <span>{errors.username}</span>
      )}
      {/* Other fields */}
      <button type="submit" disabled={isSubmitting}>
        Submit
      </button>
    </form>
  );
}

// Even More Modern: Using Form Library (e.g., Formik or React Hook Form)
import { useForm } from 'react-hook-form';

function ModernRegistrationForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting }
  } = useForm();

  const onSubmit = async (data) => {
    await submitForm(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('username', { required: 'Required' })} />
      {errors.username && <span>{errors.username.message}</span>}
      {/* Other fields */}
      <button disabled={isSubmitting}>Submit</button>
    </form>
  );
}
```

**Benefits of Modern Form Handling:**
1. Reusable form logic through custom hooks
2. Better separation of concerns
3. More maintainable validation logic
4. Built-in field tracking
5. Easy integration with form libraries
6. Better type safety with TypeScript
7. More predictable form state
8. Easier testing
9. Better performance through optimized re-renders

## Component Composition: Render Props vs Hooks

```jsx
// Classic: Render Props Pattern
class DataFetcher extends React.Component {
  state = {
    data: null,
    loading: true,
    error: null
  };

  componentDidMount() {
    this.fetchData();
  }

  fetchData = async () => {
    try {
      const data = await fetch(this.props.url);
      const json = await data.json();
      this.setState({ data: json, loading: false });
    } catch (error) {
      this.setState({ error, loading: false });
    }
  };

  render() {
    return this.props.children(this.state);
  }
}

// Usage of Render Props
class UserList extends React.Component {
  render() {
    return (
      <DataFetcher url="/api/users">
        {({ data, loading, error }) => {
          if (loading) return <Loader />;
          if (error) return <Error error={error} />;
          return (
            <ul>
              {data.map(user => (
                <li key={user.id}>{user.name}</li>
              ))}
            </ul>
          );
        }}
      </DataFetcher>
    );
  }
}

// Modern: Custom Hook Pattern
function useFetch(url) {
  const [state, setState] = useState({
    data: null,
    loading: true,
    error: null
  });

  useEffect(() => {
    let mounted = true;
    const abortController = new AbortController();

    const fetchData = async () => {
      try {
        const response = await fetch(url, {
          signal: abortController.signal
        });
        const json = await response.json();
        if (mounted) {
          setState({
            data: json,
            loading: false,
            error: null
          });
        }
      } catch (error) {
        if (error.name === 'AbortError') return;
        if (mounted) {
          setState({
            data: null,
            loading: false,
            error
          });
        }
      }
    };

    fetchData();

    return () => {
      mounted = false;
      abortController.abort();
    };
  }, [url]);

  return state;
}

// Modern Usage
function UserList() {
  const { data, loading, error } = useFetch('/api/users');

  if (loading) return <Loader />;
  if (error) return <Error error={error} />;
  
  return (
    <ul>
      {data?.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

## Component Communication: Events vs Context + Hooks

```jsx
// Classic: Event-based Communication
class ParentComponent extends React.Component {
  handleChildEvent = (data) => {
    // Handle event from child
    console.log(data);
  };

  render() {
    return (
      <ChildComponent onEvent={this.handleChildEvent} />
    );
  }
}

class ChildComponent extends React.Component {
  triggerEvent = () => {
    this.props.onEvent('Data from child');
  };

  render() {
    return <button onClick={this.triggerEvent}>Click Me</button>;
  }
}

// Modern: Context + Hooks Pattern
const AppContext = React.createContext();

function AppProvider({ children }) {
  const [state, setState] = useState({
    data: null,
    theme: 'light'
  });

  // Create stable reference for context value
  const contextValue = useMemo(() => ({
    ...state,
    updateData: (newData) => {
      setState(prev => ({
        ...prev,
        data: newData
      }));
    },
    toggleTheme: () => {
      setState(prev => ({
        ...prev,
        theme: prev.theme === 'light' ? 'dark' : 'light'
      }));
    }
  }), [state]);

  return (
    <AppContext.Provider value={contextValue}>
      {children}
    </AppContext.Provider>
  );
}

// Custom hook for using context
function useApp() {
  const context = useContext(AppContext);
  if (!context) {
    throw new Error('useApp must be used within AppProvider');
  }
  return context;
}

// Modern Components
function ParentComponent() {
  const { data } = useApp();
  return (
    <div>
      <ChildComponent />
      <div>Data: {data}</div>
    </div>
  );
}

function ChildComponent() {
  const { updateData, toggleTheme } = useApp();
  
  return (
    <button onClick={() => {
      updateData('New Data');
      toggleTheme();
    }}>
      Update Parent
    </button>
  );
}

// Usage
function App() {
  return (
    <AppProvider>
      <ParentComponent />
    </AppProvider>
  );
}
```

## Component Loading States: HOC vs Suspense

```jsx
// Classic: HOC Loading Pattern
const withLoading = (WrappedComponent) => {
  return class extends React.Component {
    render() {
      const { isLoading, ...props } = this.props;
      if (isLoading) {
        return <LoadingSpinner />;
      }
      return <WrappedComponent {...props} />;
    }
  };
};

// Usage
const UserListWithLoading = withLoading(UserList);

// Modern: Suspense Pattern
// Create a resource
const createResource = (promise) => {
  let status = 'pending';
  let result;
  let suspender = promise.then(
    (data) => {
      status = 'success';
      result = data;
    },
    (error) => {
      status = 'error';
      result = error;
    }
  );

  return {
    read() {
      if (status === 'pending') {
        throw suspender;
      } else if (status === 'error') {
        throw result;
      } else if (status === 'success') {
        return result;
      }
    }
  };
};

// Component using Suspense
const UserList = () => {
  const users = usersResource.read();
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
};

// Usage with Suspense
function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <ErrorBoundary>
        <UserList />
      </ErrorBoundary>
    </Suspense>
  );
}
```

**Benefits of Modern Patterns:**
1. More declarative code
2. Better error handling
3. Automatic cleanup
4. Simpler testing
5. Better TypeScript support
6. More maintainable code structure
7. Better performance characteristics
8. Easier to compose and reuse logic
## Animation Handling: Class Components vs Hooks

```jsx
// Classic: Animation with Class Component
class AnimatedComponent extends React.Component {
  state = {
    isVisible: false,
    animation: ''
  };

  componentDidMount() {
    // Start entrance animation
    this.setState({ 
      isVisible: true,
      animation: 'fade-in'
    });
  }

  componentDidUpdate(prevProps) {
    if (prevProps.shouldShow !== this.props.shouldShow) {
      this.setState({
        animation: this.props.shouldShow ? 'fade-in' : 'fade-out'
      });
    }
  }

  handleAnimationEnd = () => {
    if (this.state.animation === 'fade-out') {
      this.setState({ isVisible: false });
    }
  };

  render() {
    if (!this.state.isVisible) return null;

    return (
      <div 
        className={`animated ${this.state.animation}`}
        onAnimationEnd={this.handleAnimationEnd}
      >
        {this.props.children}
      </div>
    );
  }
}

// Modern: Animation with Hooks
function useAnimation(initialState = false) {
  const [isVisible, setIsVisible] = useState(initialState);
  const [animation, setAnimation] = useState('');
  
  // Ref for tracking mounted state
  const mounted = useRef(false);

  useEffect(() => {
    mounted.current = true;
    return () => {
      mounted.current = false;
    };
  }, []);

  const animate = useCallback((show) => {
    if (show) {
      setIsVisible(true);
      setAnimation('fade-in');
    } else {
      setAnimation('fade-out');
    }
  }, []);

  const handleAnimationEnd = useCallback(() => {
    if (animation === 'fade-out' && mounted.current) {
      setIsVisible(false);
    }
  }, [animation]);

  return {
    isVisible,
    animation,
    animate,
    handleAnimationEnd
  };
}

function AnimatedComponent({ shouldShow, children }) {
  const { 
    isVisible, 
    animation, 
    handleAnimationEnd 
  } = useAnimation(shouldShow);

  useEffect(() => {
    animate(shouldShow);
  }, [shouldShow, animate]);

  if (!isVisible) return null;

  return (
    <div 
      className={`animated ${animation}`}
      onAnimationEnd={handleAnimationEnd}
    >
      {children}
    </div>
  );
}

// Modern: Using Framer Motion
import { motion, AnimatePresence } from 'framer-motion';

function ModernAnimatedComponent({ isVisible, children }) {
  return (
    <AnimatePresence>
      {isVisible && (
        <motion.div
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
          transition={{ duration: 0.3 }}
        >
          {children}
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

## Routing Integration: Class vs Modern Hooks

```jsx
// Classic: Route Component with Class
class UserProfile extends React.Component {
  state = {
    user: null,
    loading: true
  };

  componentDidMount() {
    this.loadUser();
  }

  componentDidUpdate(prevProps) {
    if (prevProps.match.params.id !== this.props.match.params.id) {
      this.loadUser();
    }
  }

  loadUser = async () => {
    const { id } = this.props.match.params;
    try {
      const user = await fetchUser(id);
      this.setState({ user, loading: false });
    } catch (error) {
      this.props.history.push('/error');
    }
  };

  handleEdit = () => {
    const { id } = this.props.match.params;
    this.props.history.push(`/users/${id}/edit`);
  };

  render() {
    const { user, loading } = this.state;
    if (loading) return <Loader />;
    
    return (
      <div>
        <h1>{user.name}</h1>
        <button onClick={this.handleEdit}>Edit</button>
      </div>
    );
  }
}

// Classic HOC usage
export default withRouter(UserProfile);

// Modern: Route Component with Hooks
import { 
  useParams, 
  useNavigate, 
  useLocation,
  useSearchParams 
} from 'react-router-dom';

function UserProfile() {
  const { id } = useParams();
  const navigate = useNavigate();
  const location = useLocation();
  const [searchParams, setSearchParams] = useSearchParams();
  
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function loadUser() {
      try {
        const userData = await fetchUser(id);
        setUser(userData);
        setLoading(false);
      } catch (error) {
        navigate('/error', { 
          state: { error: error.message } 
        });
      }
    }

    loadUser();
  }, [id, navigate]);

  const handleEdit = useCallback(() => {
    navigate(`/users/${id}/edit`, {
      // Preserve current location state
      state: { returnTo: location.pathname }
    });
  }, [id, navigate, location]);

  // Handle query parameters
  const handleFilter = useCallback((filter) => {
    setSearchParams({ filter });
  }, [setSearchParams]);

  if (loading) return <Loader />;

  return (
    <div>
      <h1>{user.name}</h1>
      <button onClick={handleEdit}>Edit</button>
      <select 
        value={searchParams.get('filter') || ''}
        onChange={(e) => handleFilter(e.target.value)}
      >
        <option value="">All</option>
        <option value="active">Active</option>
        <option value="inactive">Inactive</option>
      </select>
    </div>
  );
}

// Modern usage - no HOC needed
export default UserProfile;
```

**Benefits of Modern Routing:**
1. More intuitive API
2. No HOC wrapping needed
3. Better TypeScript support
4. Easier testing
5. More flexible navigation options
6. Better state management
7. Simpler query parameter handling
8. Cleaner code structure
## State Management Patterns: Redux Class vs Modern Redux Toolkit

```jsx
// Classic Redux Pattern
// Action Types
const ADD_TODO = 'ADD_TODO';
const TOGGLE_TODO = 'TOGGLE_TODO';

// Action Creators
const addTodo = (text) => ({
  type: ADD_TODO,
  payload: {
    id: Date.now(),
    text,
    completed: false
  }
});

const toggleTodo = (id) => ({
  type: TOGGLE_TODO,
  payload: id
});

// Reducer
const initialState = {
  todos: []
};

function todoReducer(state = initialState, action) {
  switch (action.type) {
    case ADD_TODO:
      return {
        ...state,
        todos: [...state.todos, action.payload]
      };
    case TOGGLE_TODO:
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    default:
      return state;
  }
}

// Class Component with Redux
class TodoList extends React.Component {
  handleAddTodo = (text) => {
    this.props.dispatch(addTodo(text));
  };

  handleToggle = (id) => {
    this.props.dispatch(toggleTodo(id));
  };

  render() {
    return (
      <div>
        {this.props.todos.map(todo => (
          <Todo
            key={todo.id}
            {...todo}
            onClick={() => this.handleToggle(todo.id)}
          />
        ))}
      </div>
    );
  }
}

const mapStateToProps = (state) => ({
  todos: state.todos
});

export default connect(mapStateToProps)(TodoList);

// Modern Redux Toolkit Pattern
import { createSlice, configureStore } from '@reduxjs/toolkit';

// Slice
const todoSlice = createSlice({
  name: 'todos',
  initialState: {
    todos: [],
    status: 'idle',
    error: null
  },
  reducers: {
    addTodo: {
      reducer(state, action) {
        state.todos.push(action.payload);
      },
      prepare(text) {
        return {
          payload: {
            id: Date.now(),
            text,
            completed: false
          }
        };
      }
    },
    toggleTodo(state, action) {
      const todo = state.todos.find(todo => todo.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchTodos.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchTodos.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.todos = action.payload;
      })
      .addCase(fetchTodos.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.error.message;
      });
  }
});

// Thunk
export const fetchTodos = createAsyncThunk(
  'todos/fetchTodos',
  async () => {
    const response = await fetch('/api/todos');
    return response.json();
  }
);

// Modern Functional Component with Redux Toolkit
function TodoList() {
  const dispatch = useDispatch();
  const todos = useSelector(state => state.todos.todos);
  const status = useSelector(state => state.todos.status);
  
  // Use hooks for side effects
  useEffect(() => {
    if (status === 'idle') {
      dispatch(fetchTodos());
    }
  }, [status, dispatch]);

  // Memoize handlers
  const handleAddTodo = useCallback((text) => {
    dispatch(todoSlice.actions.addTodo(text));
  }, [dispatch]);

  const handleToggle = useCallback((id) => {
    dispatch(todoSlice.actions.toggleTodo(id));
  }, [dispatch]);

  if (status === 'loading') return <Loader />;
  if (status === 'failed') return <Error />;

  return (
    <div>
      <AddTodoForm onAdd={handleAddTodo} />
      {todos.map(todo => (
        <Todo
          key={todo.id}
          {...todo}
          onClick={() => handleToggle(todo.id)}
        />
      ))}
    </div>
  );
}

// Modern Store Setup
const store = configureStore({
  reducer: {
    todos: todoSlice.reducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(logger)
});

// TypeScript Support
interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

interface TodoState {
  todos: Todo[];
  status: 'idle' | 'loading' | 'succeeded' | 'failed';
  error: string | null;
}

// Modern Selectors with Reselect
import { createSelector } from '@reduxjs/toolkit';

const selectTodos = (state) => state.todos.todos;

const selectCompletedTodos = createSelector(
  [selectTodos],
  (todos) => todos.filter(todo => todo.completed)
);

const selectTodosByStatus = createSelector(
  [selectTodos, (state, status) => status],
  (todos, status) => todos.filter(todo => todo.completed === status)
);

// Usage in component
function CompletedTodos() {
  const completedTodos = useSelector(selectCompletedTodos);
  const activeTodos = useSelector(state => 
    selectTodosByStatus(state, false)
  );

  return (
    <div>
      <h2>Completed ({completedTodos.length})</h2>
      {/* render todos */}
    </div>
  );
}
```

**Benefits of Modern Redux Toolkit:**
1. Immutable updates with Immer
2. Simplified action creation
3. Built-in thunk middleware
4. Automatic action types generation
5. Better TypeScript support
6. Simplified store setup
7. Built-in devtools configuration
8. Reduced boilerplate
9. Better performance with createSelector
10. Simpler async logic handling

## Performance Optimization: Class vs Modern Techniques

```jsx
// Classic: Performance Optimization
class ExpensiveComponent extends React.Component {
  shouldComponentUpdate(nextProps, nextState) {
    return this.props.value !== nextProps.value;
  }

  // Expensive calculation
  calculateValue() {
    return this.props.value.map(x => x * 2);
  }

  render() {
    const calculatedValue = this.calculateValue();
    return <div>{calculatedValue.join(', ')}</div>;
  }
}

// Classic: PureComponent
class OptimizedList extends React.PureComponent {
  render() {
    return (
      <ul>
        {this.props.items.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    );
  }
}

// Modern: Performance Optimization with Hooks
function ExpensiveComponent({ value }) {
  // Memoize expensive calculation
  const calculatedValue = useMemo(() => {
    return value.map(x => x * 2);
  }, [value]);

  // Memoize callback functions
  const handleClick = useCallback(() => {
    console.log(calculatedValue);
  }, [calculatedValue]);

  // Prevent unnecessary re-renders
  return useMemo(() => (
    <div onClick={handleClick}>
      {calculatedValue.join(', ')}
    </div>
  ), [calculatedValue, handleClick]);
}

// Modern: React.memo with custom comparison
const OptimizedList = React.memo(
  function OptimizedList({ items, onItemClick }) {
    return (
      <ul>
        {items.map(item => (
          <li 
            key={item.id}
            onClick={() => onItemClick(item.id)}
          >
            {item.name}
          </li>
        ))}
      </ul>
    );
  },
  (prevProps, nextProps) => {
    return (
      prevProps.items.length === nextProps.items.length &&
      prevProps.items.every((item, index) => 
        item.id === nextProps.items[index].id
      )
    );
  }
);

// Modern: Performance Hooks Pattern
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]);

  return debouncedValue;
}

function useThrottle(value, limit) {
  const [throttledValue, setThrottledValue] = useState(value);
  const lastRan = useRef(Date.now());

  useEffect(() => {
    const handler = setTimeout(() => {
      if (Date.now() - lastRan.current >= limit) {
        setThrottledValue(value);
        lastRan.current = Date.now();
      }
    }, limit - (Date.now() - lastRan.current));

    return () => {
      clearTimeout(handler);
    };
  }, [value, limit]);

  return throttledValue;
}

// Modern Usage Example
function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);

  // Only triggers search after debounce
  useEffect(() => {
    if (debouncedSearchTerm) {
      performSearch(debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);

  return (
    <input
      value={searchTerm}
      onChange={e => setSearchTerm(e.target.value)}
    />
  );
}

// Modern: Virtual List for Large Data Sets
function VirtualizedList({ items }) {
  const [visibleRange, setVisibleRange] = useState({ start: 0, end: 20 });
  const containerRef = useRef();

  const handleScroll = useCallback(() => {
    const { scrollTop, clientHeight } = containerRef.current;
    const itemHeight = 50; // fixed height for each item
    const start = Math.floor(scrollTop / itemHeight);
    const end = start + Math.ceil(clientHeight / itemHeight);
    
    setVisibleRange({ start, end });
  }, []);

  useEffect(() => {
    const container = containerRef.current;
    container.addEventListener('scroll', handleScroll);
    return () => container.removeEventListener('scroll', handleScroll);
  }, [handleScroll]);

  return (
    <div 
      ref={containerRef}
      style={{ height: '400px', overflow: 'auto' }}
    >
      <div style={{ height: `${items.length * 50}px`, position: 'relative' }}>
        {items
          .slice(visibleRange.start, visibleRange.end)
          .map((item, index) => (
            <div
              key={item.id}
              style={{
                position: 'absolute',
                top: `${(index + visibleRange.start) * 50}px`,
                height: '50px'
              }}
            >
              {item.content}
            </div>
          ))}
      </div>
    </div>
  );
}

// Modern: Performance Monitoring Hook
function usePerformanceMonitor(componentName) {
  const renderCount = useRef(0);
  const lastRenderTime = useRef(performance.now());

  useEffect(() => {
    const currentTime = performance.now();
    const renderTime = currentTime - lastRenderTime.current;
    renderCount.current++;

    console.log(`${componentName} Render #${renderCount.current}`);
    console.log(`Render time: ${renderTime}ms`);

    lastRenderTime.current = currentTime;
  });
}
```

**Benefits of Modern Performance Optimization:**
1. More granular control over memoization
2. Built-in hooks for common optimization patterns
3. Easier implementation of virtual scrolling
4. Better handling of expensive calculations
5. Simplified debounce and throttle patterns
6. More efficient list rendering
7. Better performance monitoring tools
8. Automatic batching in React 18
9. Easier integration with profiling tools
## Testing Patterns: Class vs Modern Testing

```javascript
// Classic: Testing Class Components
// Component
class Counter extends React.Component {
  state = { count: 0 };

  increment = () => {
    this.setState(prev => ({ count: prev.count + 1 }));
  };

  render() {
    return (
      <div>
        <span data-testid="count">{this.state.count}</span>
        <button onClick={this.increment}>Increment</button>
      </div>
    );
  }
}

// Classic Test
import { shallow, mount } from 'enzyme';

describe('Counter (Classic)', () => {
  it('renders initial count', () => {
    const wrapper = shallow(<Counter />);
    expect(wrapper.state('count')).toBe(0);
  });

  it('increments count', () => {
    const wrapper = mount(<Counter />);
    wrapper.find('button').simulate('click');
    expect(wrapper.state('count')).toBe(1);
  });
});

// Modern: Testing Hooks and Functions
// Component
function Counter() {
  const [count, setCount] = useState(0);

  const increment = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);

  return (
    <div>
      <span data-testid="count">{count}</span>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

// Modern Test with React Testing Library
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('Counter (Modern)', () => {
  it('renders initial count', () => {
    render(<Counter />);
    expect(screen.getByTestId('count')).toHaveTextContent('0');
  });

  it('increments count', async () => {
    const user = userEvent.setup();
    render(<Counter />);
    
    await user.click(screen.getByRole('button'));
    expect(screen.getByTestId('count')).toHaveTextContent('1');
  });
});

// Testing Custom Hooks
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  
  const increment = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);

  return { count, increment };
}

// Testing Hook with renderHook
import { renderHook, act } from '@testing-library/react-hooks';

describe('useCounter', () => {
  it('should increment counter', () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });
});

// Modern: Testing Async Operations
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function fetchUser() {
      try {
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        setUser(data);
      } catch (err) {
        setError(err);
      }
    }
    fetchUser();
  }, [userId]);

  if (error) return <div>Error loading user</div>;
  if (!user) return <div>Loading...</div>;
  
  return <div>{user.name}</div>;
}

// Modern Async Test
describe('UserProfile', () => {
  beforeEach(() => {
    fetch.resetMocks();
  });

  it('loads and displays user', async () => {
    const mockUser = { id: 1, name: 'John Doe' };
    fetch.mockResponseOnce(JSON.stringify(mockUser));

    render(<UserProfile userId={1} />);

    // Check loading state
    expect(screen.getByText('Loading...')).toBeInTheDocument();

    // Wait for user data
    await screen.findByText('John Doe');
    
    expect(fetch).toHaveBeenCalledWith('/api/users/1');
  });

  it('handles error state', async () => {
    fetch.mockRejectOnce(new Error('Failed to fetch'));

    render(<UserProfile userId={1} />);

    await screen.findByText('Error loading user');
  });
});

// Testing Redux Connected Components
// Modern approach using Redux Toolkit and Testing Library
import { configureStore } from '@reduxjs/toolkit';
import { Provider } from 'react-redux';

function renderWithRedux(
  ui,
  {
    preloadedState = {},
    store = configureStore({
      reducer: rootReducer,
      preloadedState
    }),
    ...renderOptions
  } = {}
) {
  function Wrapper({ children }) {
    return <Provider store={store}>{children}</Provider>;
  }

  return {
    store,
    ...render(ui, { wrapper: Wrapper, ...renderOptions })
  };
}

// Usage
test('connected component', () => {
  const { store } = renderWithRedux(<ConnectedComponent />, {
    preloadedState: {
      todos: [{
        id: 1,
        text: 'Test todo',
        completed: false
      }]
    }
  });

  expect(screen.getByText('Test todo')).toBeInTheDocument();
});
```

**Benefits of Modern Testing Approaches:**
1. More focus on user behavior
2. Less brittle tests
3. Better accessibility testing
4. Simpler async testing
5. Better hook testing support
6. More maintainable tests
7. Better error messages
8. More realistic user interactions
9. Better integration with modern React features
10. Encourages better component design

## Styling Patterns: Classic vs Modern Approaches

```jsx
// Classic: CSS Classes and Stylesheets
// styles.css
.button {
  background: blue;
  color: white;
}
.button.primary {
  background: green;
}
.button.disabled {
  opacity: 0.5;
}

// Classic Component
class Button extends React.Component {
  render() {
    const { primary, disabled, children } = this.props;
    const className = `button ${primary ? 'primary' : ''} ${disabled ? 'disabled' : ''}`;
    
    return (
      <button className={className} disabled={disabled}>
        {children}
      </button>
    );
  }
}

// Classic: Inline Styles
class StyledButton extends React.Component {
  render() {
    const styles = {
      button: {
        background: 'blue',
        color: 'white',
        ...(this.props.primary && {
          background: 'green'
        }),
        ...(this.props.disabled && {
          opacity: 0.5
        })
      }
    };

    return (
      <button style={styles.button} disabled={this.props.disabled}>
        {this.props.children}
      </button>
    );
  }
}

// Modern: Styled Components
import styled, { css } from 'styled-components';

const Button = styled.button`
  background: blue;
  color: white;
  padding: 10px 20px;
  border-radius: 4px;
  border: none;
  cursor: pointer;

  ${props => props.primary && css`
    background: green;
    font-weight: bold;
  `}

  ${props => props.disabled && css`
    opacity: 0.5;
    cursor: not-allowed;
  `}

  &:hover {
    opacity: 0.9;
  }

  @media (max-width: 768px) {
    width: 100%;
  }
`;

// Modern: CSS Modules with TypeScript
// Button.module.css
.button {
  background: blue;
  color: white;
}
.primary {
  background: green;
}
.disabled {
  opacity: 0.5;
}

// Button.tsx
import styles from './Button.module.css';

interface ButtonProps {
  primary?: boolean;
  disabled?: boolean;
  children: React.ReactNode;
}

const Button: React.FC<ButtonProps> = ({ primary, disabled, children }) => {
  const className = [
    styles.button,
    primary && styles.primary,
    disabled && styles.disabled
  ].filter(Boolean).join(' ');

  return (
    <button className={className} disabled={disabled}>
      {children}
    </button>
  );
};

// Modern: Emotion with Theme Support
import { css, ThemeProvider } from '@emotion/react';
import styled from '@emotion/styled';

const theme = {
  colors: {
    primary: 'blue',
    secondary: 'green',
    disabled: 'gray'
  },
  spacing: {
    small: '8px',
    medium: '16px',
    large: '24px'
  }
};

const Button = styled.button`
  padding: ${props => props.theme.spacing.medium};
  background: ${props => props.theme.colors.primary};
  color: white;

  ${props => props.variant === 'secondary' && css`
    background: ${props.theme.colors.secondary};
  `}
`;

// Modern: Tailwind CSS Integration
function ModernButton({ primary, disabled, children }) {
  return (
    <button
      className={`
        px-4 py-2 rounded
        ${primary 
          ? 'bg-blue-500 hover:bg-blue-600' 
          : 'bg-gray-500 hover:bg-gray-600'}
        ${disabled && 'opacity-50 cursor-not-allowed'}
        transition-colors duration-200
      `}
      disabled={disabled}
    >
      {children}
    </button>
  );
}

// Modern: CSS-in-JS with Style Objects
import { makeStyles } from '@material-ui/core/styles';

const useStyles = makeStyles(theme => ({
  button: {
    background: theme.palette.primary.main,
    color: 'white',
    padding: theme.spacing(1, 2),
    '&:hover': {
      background: theme.palette.primary.dark,
    },
    '&:disabled': {
      opacity: 0.5,
    }
  },
  primary: {
    background: theme.palette.secondary.main,
    '&:hover': {
      background: theme.palette.secondary.dark,
    }
  }
}));

function StyledButton({ primary, disabled, children }) {
  const classes = useStyles();
  
  return (
    <button
      className={`${classes.button} ${primary ? classes.primary : ''}`}
      disabled={disabled}
    >
      {children}
    </button>
  );
}
```

**Benefits of Modern Styling Approaches:**
1. Better scoping and isolation
2. Type safety with CSS Modules
3. Dynamic styles based on props
4. Theme support
5. Better performance optimization
6. Better developer experience
7. Easier maintenance
8. Better IDE support
9. Runtime style injection
10. Better testing capabilities
## Error Handling Patterns: Classic vs Modern Approaches

```javascript
// Classic: Error Boundary Class Component
class ClassErrorBoundary extends React.Component {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Log to error service
    console.error('Error caught:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h1>Something went wrong.</h1>
          <pre>{this.state.error.message}</pre>
        </div>
      );
    }

    return this.props.children;
  }
}

// Modern: Error Boundary with Hooks (Note: Still needs class component)
class ModernErrorBoundary extends React.Component {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Modern error logging service
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback?.(this.state.error) || (
        <DefaultErrorUI error={this.state.error} />
      );
    }

    return this.props.children;
  }
}

// Modern: Custom Error Hook for Async Operations
function useAsyncError() {
  const [error, setError] = useState(null);
  
  const handleError = useCallback((error) => {
    setError(error);
    // Optional: Log to service
    logErrorToService(error);
  }, []);

  const clearError = useCallback(() => {
    setError(null);
  }, []);

  return [error, handleError, clearError];
}

// Modern: Try-Catch Pattern with Hooks
function useTryCatch(asyncFn) {
  const [state, setState] = useState({
    data: null,
    error: null,
    loading: false
  });

  const execute = useCallback(async (...args) => {
    setState(prev => ({ ...prev, loading: true }));
    try {
      const data = await asyncFn(...args);
      setState({ data, error: null, loading: false });
      return data;
    } catch (error) {
      setState({ data: null, error, loading: false });
      throw error; // Re-throw if needed
    }
  }, [asyncFn]);

  return { ...state, execute };
}

// Modern: Component with Multiple Error Handling Strategies
function UserProfile({ userId }) {
  const [error, handleError, clearError] = useAsyncError();
  const { data: user, error: fetchError, loading, execute: fetchUser } = 
    useTryCatch(async (id) => {
      const response = await fetch(`/api/users/${id}`);
      if (!response.ok) throw new Error('Failed to fetch user');
      return response.json();
    });

  useEffect(() => {
    fetchUser(userId).catch(handleError);
  }, [userId, fetchUser, handleError]);

  if (loading) return <LoadingSpinner />;
  if (error || fetchError) {
    return (
      <ErrorDisplay 
        error={error || fetchError}
        onRetry={() => {
          clearError();
          fetchUser(userId);
        }}
      />
    );
  }

  return <UserDetails user={user} />;
}

// Modern: Suspense Integration with Error Boundaries
const UserProfileWithSuspense = React.lazy(() => import('./UserProfile'));

function App() {
  return (
    <ModernErrorBoundary
      fallback={(error) => (
        <ErrorUI 
          error={error}
          onRetry={() => window.location.reload()}
        />
      )}
    >
      <Suspense fallback={<LoadingSpinner />}>
        <UserProfileWithSuspense userId="123" />
      </Suspense>
    </ModernErrorBoundary>
  );
}

// Modern: Error Handling with Custom Hook for Forms
function useFormWithValidation(initialValues) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});

  const validate = useCallback((fieldName, value) => {
    let fieldErrors = {};
    try {
      // Validation logic here
      if (!value) {
        throw new Error(`${fieldName} is required`);
      }
    } catch (error) {
      fieldErrors[fieldName] = error.message;
    }
    return fieldErrors;
  }, []);

  const handleChange = useCallback((e) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
    
    const fieldErrors = validate(name, value);
    setErrors(prev => ({ ...prev, ...fieldErrors }));
  }, [validate]);

  const handleBlur = useCallback((e) => {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
  }, []);

  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur
  };
}

// Modern: Global Error Handler
function useGlobalErrorHandler() {
  useEffect(() => {
    const handler = (event) => {
      // Handle unhandled promise rejections
      console.error('Unhandled error:', event.reason);
      // Show user-friendly error message
      showErrorToast(event.reason.message);
    };

    window.addEventListener('unhandledrejection', handler);
    return () => window.removeEventListener('unhandledrejection', handler);
  }, []);
}
```

**Benefits of Modern Error Handling:**
1. More granular error control
2. Better separation of concerns
3. Reusable error handling logic
4. Better integration with async operations
5. More flexible fallback UI options
6. Better type safety
7. Easier testing
8. Better user experience
9. More consistent error handling
10. Better error reporting capabilities

