useContext is a React Hook that enables consuming values from React Context without explicit Consumer wrapping. It's ideal for sharing global state like themes, user data, or app-wide settings across components.

## File Structure
```plaintext
src/
├── contexts/          # Store context definitions
│   ├── ThemeContext.js
│   ├── AuthContext.js
│   └── UserContext.js
├── hooks/            # Custom hooks using context
│   └── useUser.js
├── providers/        # Context providers
│   └── UserProvider.js
└── components/       # Components using context
    └── ThemedButton.js
```

## Basic Syntax

```javascript
// In any component file
const value = useContext(MyContext);
```
> This is the fundamental syntax for using the hook. The value returned will be the current context value from the nearest Provider above in the tree.

## Creating a Context
```javascript
// src/contexts/MyContext.js
const MyContext = React.createContext(defaultValue);
export default MyContext;
```
> Creates a new context with an optional default value. Export this to use across your application.

## Providing Context
```javascript
// src/providers/MyContextProvider.js
import MyContext from '../contexts/MyContext';

export function MyContextProvider({ children }) {
  return (
    <MyContext.Provider value={/* some value */}>
      {children}
    </MyContext.Provider>
  );
}
```
> Wrap your components with the Provider to make the context available to them. Usually placed high in the component tree.

## Common Use Cases

### 1. Theme Context
```javascript
// src/contexts/ThemeContext.js
export const ThemeContext = React.createContext('light');

// src/App.js
function App() {
  return (
    <ThemeContext.Provider value="dark">
      <ThemedButton />
    </ThemeContext.Provider>
  );
}

// src/components/ThemedButton.js
function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Themed Button</button>;
}
```
> Demonstrates theme switching functionality. Common for implementing dark/light modes in applications.

### 2. User Authentication Context
```javascript
// src/contexts/AuthContext.js
export const AuthContext = React.createContext(null);

// src/providers/AuthProvider.js
function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  );
}

// src/components/NavBar.js
function NavBar() {
  const { user } = useContext(AuthContext);
  return user ? <UserProfile /> : <LoginButton />;
}
```
> Manages user authentication state globally. Useful for controlling access and showing different UI based on auth status.

### 3. Multilevel Data Passing
```javascript
// src/contexts/LevelContext.js
export const LevelContext = React.createContext(1);

// src/components/Level.js
function Level() {
  const level = useContext(LevelContext);
  return (
    <section className="level">
      <LevelContext.Provider value={level + 1}>
        <NestedComponent />
      </LevelContext.Provider>
    </section>
  );
}
```
> Shows how to nest context providers and increment values at each level. Useful for nested UI components.

## Advanced Patterns

### 1. Multiple Contexts
```javascript
// src/components/MultiContextComponent.js
function Component() {
  const theme = useContext(ThemeContext);
  const user = useContext(UserContext);
  // Use multiple contexts in one component
}
```
> Demonstrates using multiple contexts in a single component. Keep in mind that each context subscription adds a small performance overhead.

### 2. Context with Reducer
```javascript
// src/contexts/StateContext.js
export const StateContext = React.createContext();
export const DispatchContext = React.createContext();

// src/providers/StateProvider.js
function StateProvider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
}
```
> Combines context with useReducer for more complex state management. Similar to Redux pattern but built into React.

### 3. Dynamic Context
```javascript
// src/contexts/DynamicContext.js
export const DynamicContext = React.createContext();

// src/providers/DynamicProvider.js
function DynamicProvider({ children }) {
  const [value, setValue] = useState(initialValue);
  
  // Can include additional logic here
  useEffect(() => {
    // Update value based on some condition
  }, []);

  return (
    <DynamicContext.Provider value={{ value, setValue }}>
      {children}
    </DynamicContext.Provider>
  );
}
```
> Shows how to create a context that can update its value dynamically. Useful for values that change over time.

## Custom Hook Example
```javascript
// src/contexts/UserContext.js
export const UserContext = React.createContext(null);

// src/hooks/useUser.js
export function useUser() {
  const context = useContext(UserContext);
  if (context === undefined) {
    throw new Error('useUser must be used within a UserProvider');
  }
  return context;
}

// src/components/Profile.js
function Profile() {
  const { user } = useUser();
  return <div>Hello, {user.name}!</div>;
}
```
> Creates a custom hook to handle context usage. Provides better error messages and encapsulates context logic.

## TypeScript Example
```typescript
// src/contexts/TypedContext.ts
interface ContextType {
  value: string;
  setValue: (value: string) => void;
}

const TypedContext = React.createContext<ContextType | undefined>(undefined);

// Usage in component
function TypedComponent() {
  const context = useContext(TypedContext);
  if (!context) throw new Error('Must be used within provider');
  
  const { value, setValue } = context;
  // TypeScript now knows the types
}
```
> Shows how to properly type context in TypeScript. Provides type safety and better IDE support.

## Best Practices & Tips
1. Keep context values as small as possible
2. Split contexts by functionality
3. Place providers as low as possible in the tree
4. Use TypeScript for better type safety
5. Create custom hooks for complex context usage
6. Consider performance implications of context changes
7. Use context for truly global state only

Remember: Context is not a replacement for all prop drilling. Use it judiciously for truly global or widely-used data.