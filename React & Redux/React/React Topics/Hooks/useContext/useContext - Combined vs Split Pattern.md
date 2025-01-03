## Introduction
useContext is a React Hook that enables sharing values across your component tree without prop drilling. This cheatsheet covers implementation patterns from basic to advanced, with real-world examples and best practices.

## Basic File Structure Options

```plaintext
# Combined Pattern (Simpler Projects)
src/
├── contexts/          
│   ├── ThemeContext.js    # Contains context, provider, and hook
│   └── AuthContext.js     # Contains context, provider, and hook
└── components/       
    └── App.js

# Split Pattern (Larger Projects)
src/
├── contexts/          
│   ├── ThemeContext.js    # Just context creation
│   └── AuthContext.js     
├── hooks/            
│   ├── useTheme.js       # Custom hooks
│   └── useAuth.js        
├── providers/        
│   ├── ThemeProvider.js  # Provider components
│   └── AuthProvider.js   
└── components/
    └── App.js
```

## Pattern 1: Combined Context Pattern

```javascript
// src/contexts/ThemeContext.js

// Import necessary hooks from React
import { createContext, useContext, useState } from 'react';

// Create context with undefined default value
// Using undefined helps catch errors if context is used outside provider
export const ThemeContext = createContext();

// Custom hook for consuming context
export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};

// Provider component that wraps parts of app needing theme access
export default function ThemeProvider({ children }) {
  // State for theme management
  const [theme, setTheme] = useState('light');
  
  // Optional: Memoize value if passing objects/arrays
  const value = {
    theme,
    setTheme,
    toggleTheme: () => setTheme(prev => prev === 'light' ? 'dark' : 'light')
  };

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```
> This combined pattern keeps all related context code in one file. It's ideal for smaller applications or when context logic is straightforward.

### Usage Example:
```javascript
// src/App.js
import ThemeProvider from './contexts/ThemeContext';

function App() {
  return (
    <ThemeProvider>
      <MainContent />
    </ThemeProvider>
  );
}

// src/components/ThemeToggle.js
import { useTheme } from '../contexts/ThemeContext';

function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button onClick={toggleTheme}>
      Current theme: {theme}
    </button>
  );
}
```
> Shows how to wrap your application with the provider and consume context in components.

## Pattern 2: Split Context Pattern

```javascript
// src/contexts/AuthContext.js
import { createContext } from 'react';

// Create and export context
export const AuthContext = createContext();

// src/hooks/useAuth.js
import { useContext } from 'react';
import { AuthContext } from '../contexts/AuthContext';

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};

// src/providers/AuthProvider.js
import { useState, useCallback } from 'react';
import { AuthContext } from '../contexts/AuthContext';

export function AuthProvider({ children }) {
  // State for user authentication
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  
  // Authentication methods
  const login = useCallback(async (credentials) => {
    setLoading(true);
    try {
      // Login logic here
      setUser(/* user data */);
    } finally {
      setLoading(false);
    }
  }, []);

  const logout = useCallback(() => {
    setUser(null);
  }, []);

  return (
    <AuthContext.Provider value={{ 
      user, 
      login, 
      logout, 
      loading 
    }}>
      {children}
    </AuthContext.Provider>
  );
}
```
> Split pattern separates concerns into different files, making code more maintainable in larger applications.

### Usage Example:
```javascript
// src/components/LoginForm.js
import { useAuth } from '../hooks/useAuth';

function LoginForm() {
  const { login, loading } = useAuth();

  const handleSubmit = async (e) => {
    e.preventDefault();
    await login({ username, password });
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* form fields */}
      <button disabled={loading}>
        {loading ? 'Loading...' : 'Login'}
      </button>
    </form>
  );
}
```

## Advanced Patterns

### 1. Context with Reducer
```javascript
// src/contexts/AppContext.js
import { createContext, useContext, useReducer } from 'react';

// Define initial state and reducer
const initialState = {
  count: 0,
  theme: 'light',
  user: null
};

function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'TOGGLE_THEME':
      return { ...state, theme: state.theme === 'light' ? 'dark' : 'light' };
    case 'SET_USER':
      return { ...state, user: action.payload };
    default:
      return state;
  }
}

// Create context and custom hook
const AppContext = createContext();

export function AppProvider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <AppContext.Provider value={{ state, dispatch }}>
      {children}
    </AppContext.Provider>
  );
}

export const useApp = () => useContext(AppContext);
```
> Combines context with useReducer for more complex state management scenarios.

### 2. Multiple Contexts Pattern
```javascript
// src/providers/AppProviders.js
import { AuthProvider } from './AuthProvider';
import { ThemeProvider } from './ThemeProvider';
import { UserPreferencesProvider } from './UserPreferencesProvider';

// Combine multiple providers
export function AppProviders({ children }) {
  return (
    <AuthProvider>
      <ThemeProvider>
        <UserPreferencesProvider>
          {children}
        </UserPreferencesProvider>
      </ThemeProvider>
    </AuthProvider>
  );
}

// src/components/Dashboard.js
function Dashboard() {
  const { user } = useAuth();
  const { theme } = useTheme();
  const { preferences } = usePreferences();

  return (
    <div className={theme}>
      <h1>Welcome, {user.name}</h1>
      <SettingsPanel preferences={preferences} />
    </div>
  );
}
```
> Shows how to compose multiple contexts and consume them together.

### 3. Context with Local Storage
```javascript
// src/contexts/SettingsContext.js
import { createContext, useContext, useState, useEffect } from 'react';

const SettingsContext = createContext();

export function SettingsProvider({ children }) {
  // Initialize state from localStorage if available
  const [settings, setSettings] = useState(() => {
    const saved = localStorage.getItem('settings');
    return saved ? JSON.parse(saved) : defaultSettings;
  });

  // Sync state with localStorage
  useEffect(() => {
    localStorage.setItem('settings', JSON.stringify(settings));
  }, [settings]);

  return (
    <SettingsContext.Provider value={{ settings, setSettings }}>
      {children}
    </SettingsContext.Provider>
  );
}
```
> Demonstrates persisting context state to localStorage.

## Performance Optimization

### 1. Value Memoization
```javascript
// src/contexts/OptimizedContext.js
import { useMemo } from 'react';

export function OptimizedProvider({ children }) {
  const [state, setState] = useState(initialState);
  
  // Memoize value to prevent unnecessary rerenders
  const value = useMemo(() => ({
    state,
    setState
  }), [state]);

  return (
    <OptimizedContext.Provider value={value}>
      {children}
    </OptimizedContext.Provider>
  );
}
```

### 2. Context Splitting
```javascript
// Split frequently changing values into separate contexts
const StateContext = createContext();
const DispatchContext = createContext();

function AppProvider({ children }) {
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

## TypeScript Support
```typescript
// src/contexts/ThemeContext.tsx
interface ThemeContextType {
  theme: 'light' | 'dark';
  setTheme: (theme: 'light' | 'dark') => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);
```

## Best Practices & Common Mistakes

### Best Practices:
1. Always create custom hooks for context consumption
2. Keep context values focused and minimal
3. Place providers as close as possible to where they're needed
4. Handle missing providers with clear error messages
5. Consider performance implications when updating context values

### Common Mistakes:
1. Using context for props that should be passed directly
2. Creating unnecessary provider nesting
3. Not memoizing context values
4. Forgetting error boundaries around context usage
5. Over-centralizing state in context

Remember: Context is powerful but should be used judiciously. Not all state needs to be in context - sometimes props are simpler and more appropriate.