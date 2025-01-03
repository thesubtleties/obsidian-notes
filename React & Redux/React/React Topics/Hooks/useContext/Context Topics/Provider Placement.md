## Provider Placement Hierarchy

```plaintext
src/
├── main.jsx          # Global, app-wide providers
├── App.jsx           # Feature-wide or route-level providers
└── components/       # Component-specific providers
    ├── Dashboard/
    └── Features/
```

## 1. Root Level (main.jsx)
```javascript
// src/main.jsx
import { AuthProvider } from './contexts/AuthContext'
import { ThemeProvider } from './contexts/ThemeContext'
import { StoreProvider } from './contexts/StoreContext'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <ErrorBoundary>
      <StoreProvider>        {/* Global state management */}
        <AuthProvider>       {/* Authentication */}
          <ThemeProvider>    {/* Theme/UI */}
            <App />
          </ThemeProvider>
        </AuthProvider>
      </StoreProvider>
    </ErrorBoundary>
  </React.StrictMode>
)
```
> Best for truly global contexts that affect the entire application.
> Examples: Authentication, Theme, Global Store, Internationalization

## 2. App Level (App.jsx)
```javascript
// src/App.jsx
import { ModalProvider } from './contexts/ModalContext'
import { NavigationProvider } from './contexts/NavigationContext'

function App() {
  return (
    <NavigationProvider>    {/* Route/Navigation state */}
      <ModalProvider>       {/* Modal/Dialog system */}
        <Router>
          <Layout>
            <Routes>
              {/* ... */}
            </Routes>
          </Layout>
        </Router>
      </ModalProvider>
    </NavigationProvider>
  );
}
```
> Suitable for app-wide features that aren't needed at initial load.
> Examples: Modal Systems, Navigation State, Feature Flags

## 3. Route Level
```javascript
// src/routes/Dashboard.jsx
function DashboardRoute() {
  return (
    <DashboardProvider>     {/* Dashboard-specific state */}
      <WidgetProvider>      {/* Widget management */}
        <Dashboard />
      </WidgetProvider>
    </DashboardProvider>
  );
}

// src/routes/Settings.jsx
function SettingsRoute() {
  return (
    <SettingsProvider>      {/* Settings-specific state */}
      <Settings />
    </SettingsProvider>
  );
}
```
> Good for route-specific features and state.
> Examples: Dashboard State, Settings Management

## 4. Feature Level
```javascript
// src/features/Chat/ChatFeature.jsx
function ChatFeature() {
  return (
    <ChatProvider>         {/* Chat-specific state */}
      <MessageProvider>    {/* Message handling */}
        <ChatWindow />
        <ChatInput />
      </MessageProvider>
    </ChatProvider>
  );
}
```
> Best for isolated features with their own state management.
> Examples: Chat Systems, Complex Forms, Data Grids

## 5. Component Level
```javascript
// src/components/ComplexForm/ComplexForm.jsx
function ComplexForm() {
  return (
    <FormProvider>        {/* Form-specific state */}
      <ValidationProvider>{/* Validation logic */}
        <FormFields />
        <FormControls />
      </ValidationProvider>
    </FormProvider>
  );
}
```
> Use for complex components that need internal state management.
> Examples: Complex Forms, Rich Text Editors, Data Tables

## Provider Organization Patterns

### 1. Combining Related Providers
```javascript
// src/providers/AppProviders.jsx
function AppProviders({ children }) {
  return (
    <AuthProvider>
      <ThemeProvider>
        <LocaleProvider>
          {children}
        </LocaleProvider>
      </ThemeProvider>
    </AuthProvider>
  );
}

// Usage in main.jsx
<AppProviders>
  <App />
</AppProviders>
```

### 2. Conditional Providers
```javascript
function FeatureProvider({ children }) {
  const isFeatureEnabled = useFeatureFlag('newFeature');
  
  return isFeatureEnabled ? (
    <NewFeatureProvider>
      {children}
    </NewFeatureProvider>
  ) : children;
}
```

### 3. Dynamic Providers
```javascript
function DynamicProviders({ providers, children }) {
  return providers.reduce(
    (acc, Provider) => <Provider>{acc}</Provider>,
    children
  );
}

// Usage
<DynamicProviders 
  providers={[AuthProvider, ThemeProvider, StoreProvider]}
>
  <App />
</DynamicProviders>
```

## Best Practices

### 1. Provider Ordering
```javascript
// Recommended order (outside to inside):
<ErrorBoundary>
  <StoreProvider>      {/* Global state first */}
    <AuthProvider>     {/* Auth before features */}
      <ThemeProvider>  {/* UI concerns */}
        <FeatureProvider> {/* Feature-specific last */}
          <Component />
        </FeatureProvider>
      </ThemeProvider>
    </AuthProvider>
  </StoreProvider>
</ErrorBoundary>
```

### 2. Performance Considerations
```javascript
// Split providers that update frequently
function AppProviders({ children }) {
  return (
    <StaticDataProvider>    {/* Rarely updates */}
      <DynamicDataProvider> {/* Frequently updates */}
        {children}
      </DynamicDataProvider>
    </StaticDataProvider>
  );
}
```

### 3. Testing Considerations
```javascript
// src/test/TestProviders.jsx
export function TestProviders({ children }) {
  return (
    <AuthProvider>
      <ThemeProvider theme="light">
        <TestDataProvider>
          {children}
        </TestDataProvider>
      </ThemeProvider>
    </AuthProvider>
  );
}

// In tests
render(
  <TestProviders>
    <ComponentToTest />
  </TestProviders>
);
```

## Decision Making Guide

1. **Ask These Questions:**
   - Is this state needed globally?
   - Which components need this context?
   - How often does the context value change?
   - Are there performance implications?

2. **Placement Rules:**
   - Global state → main.jsx
   - Feature state → Feature component
   - Component state → Component level
   - Route state → Route component

3. **Consider:**
   - Code splitting impact
   - Testing requirements
   - Maintenance overhead
   - Performance needs

Remember:
- Place providers as close as possible to where they're needed
- Consider code splitting implications
- Think about testing and maintenance
- Be mindful of unnecessary re-renders
- Document provider dependencies and requirements