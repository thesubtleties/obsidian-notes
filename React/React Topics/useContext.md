useContext is a React Hook that allows you to consume values from a React Context without wrapping your component in a Context Consumer. It provides a way to share values like themes, user data, or any other global state across your component tree without explicitly passing props through every level.

## Basic Syntax

```javascript
const value = useContext(MyContext);
```

## Key Concepts

1. Accepts a context object (created by React.createContext) and returns the current context value
2. The value returned is determined by the nearest matching Provider above in the component tree
3. Triggers a re-render when the context value changes

## Creating a Context

```javascript
const MyContext = React.createContext(defaultValue);
```

## Providing Context

```javascript
<MyContext.Provider value={/* some value */}>
  {/* child components */}
</MyContext.Provider>
```

## Common Use Cases

### 1. Theme Context
```javascript
const ThemeContext = React.createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <ThemedButton />
    </ThemeContext.Provider>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Themed Button</button>;
}
```

### 2. User Authentication Context
```javascript
const AuthContext = React.createContext(null);

function App() {
  const [user, setUser] = useState(null);
  return (
    <AuthContext.Provider value={{ user, setUser }}>
      <NavBar />
      <Content />
    </AuthContext.Provider>
  );
}

function NavBar() {
  const { user } = useContext(AuthContext);
  return user ? <UserProfile /> : <LoginButton />;
}
```

### 3. Multilevel Data Passing
```javascript
const LevelContext = React.createContext(1);

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

## Advanced Patterns

### 1. Multiple Contexts
```javascript
function Component() {
  const theme = useContext(ThemeContext);
  const user = useContext(UserContext);
  // ...
}
```

### 2. Context with Reducer
```javascript
const StateContext = React.createContext();
const DispatchContext = React.createContext();

function App() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>
        <ComponentA />
        <ComponentB />
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
}
```

### 3. Dynamic Context
```javascript
const DynamicContext = React.createContext();

function DynamicProvider({ children }) {
  const [value, setValue] = useState(initialValue);
  return (
    <DynamicContext.Provider value={{ value, setValue }}>
      {children}
    </DynamicContext.Provider>
  );
}
```

## Best Practices

1. Use context for global or widely-used data (theme, user auth, language)
2. Don't overuse - props are often simpler for shallow hierarchies
3. Split contexts when values change at different frequencies
4. Consider using a separate file for context creation and a custom hook for using the context

## Common Mistakes

1. Forgetting to wrap components with Provider
2. Unnecessarily nesting Providers
3. Overusing context for data that could be passed as props
4. Not considering performance implications of context changes

## Tips

1. Use the useContext hook in the component that needs the data, not in intermediary components
2. Combine useContext with useReducer for complex state management
3. For TypeScript, provide a type for your context:
   ```typescript
   const MyContext = React.createContext<MyContextType | undefined>(undefined);
   ```
4. Use a default value that matches the shape of your context for better error handling

## Example: Custom Hook with Context

```javascript
const UserContext = React.createContext(null);

export function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
}

export function useUser() {
  const context = useContext(UserContext);
  if (context === undefined) {
    throw new Error('useUser must be used within a UserProvider');
  }
  return context;
}

// Usage
function Profile() {
  const { user } = useUser();
  return <div>Hello, {user.name}!</div>;
}
```

Remember, useContext is powerful for sharing data across your React tree, but it's not a replacement for all prop passing. Use it judiciously to keep your components reusable and your data flow clear.