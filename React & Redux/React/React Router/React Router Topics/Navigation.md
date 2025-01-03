## Basic Navigation Components

### File Location
```text
src/components/Navigation.jsx
src/hooks/useCustomNavigation.js
```

### Link and NavLink
```javascript
// src/components/Navigation.jsx
import { Link, NavLink } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      {/* Basic Link - Simple navigation */}
      <Link to="/home">Home</Link>

      {/* NavLink - Includes active state */}
      <NavLink 
        to="/dashboard"
        className={({ isActive, isPending }) => 
          isPending ? "pending" : isActive ? "active" : ""
        }
      >
        Dashboard
      </NavLink>

      {/* NavLink with custom styling */}
      <NavLink 
        to="/profile"
        style={({ isActive }) => ({
          fontWeight: isActive ? 'bold' : 'normal',
          color: isActive ? '#007bff' : '#000'
        })}
      >
        Profile
      </NavLink>
    </nav>
  );
}
```
`Link` provides basic navigation while `NavLink` adds active state awareness. `NavLink` is perfect for navigation menus where you want to highlight the current route.

## Programmatic Navigation

```javascript
// src/components/NavigationButtons.jsx
import { useNavigate, useLocation } from 'react-router-dom';

function NavigationButtons() {
  const navigate = useNavigate();
  const location = useLocation();

  return (
    <div>
      {/* Basic navigation */}
      <button onClick={() => navigate('/dashboard')}>
        Go to Dashboard
      </button>

      {/* Navigate with replace (no history entry) */}
      <button onClick={() => navigate('/login', { replace: true })}>
        Login
      </button>

      {/* Navigate relative to current route */}
      <button onClick={() => navigate('../')}>
        Go Up One Level
      </button>

      {/* Navigate with state */}
      <button onClick={() => 
        navigate('/details', { 
          state: { from: location.pathname } 
        })
      }>
        View Details
      </button>
    </div>
  );
}
```
Programmatic navigation is useful for navigation triggered by user actions or application logic. The `useNavigate` hook provides a flexible way to handle navigation in code.

## Search Parameters

```javascript
// src/components/SearchNavigation.jsx
import { useSearchParams } from 'react-router-dom';

function SearchNavigation() {
  const [searchParams, setSearchParams] = useSearchParams();
  
  return (
    <div>
      {/* Update single parameter */}
      <select
        value={searchParams.get('sort') || 'asc'}
        onChange={e => setSearchParams({ 
          ...Object.fromEntries(searchParams),
          sort: e.target.value 
        })}
      >
        <option value="asc">Ascending</option>
        <option value="desc">Descending</option>
      </select>

      {/* Update multiple parameters */}
      <button onClick={() => {
        setSearchParams({
          page: '1',
          size: '10',
          filter: 'active'
        });
      }}>
        Reset Filters
      </button>

      {/* Preserve existing parameters */}
      <button onClick={() => {
        setSearchParams(params => ({
          ...Object.fromEntries(params),
          page: String(Number(params.get('page') || '1') + 1)
        }));
      }}>
        Next Page
      </button>
    </div>
  );
}
```
Search parameters are useful for managing filter states, pagination, and other URL-based state. The `useSearchParams` hook provides a way to read and update URL parameters.

## Navigation Guards

```javascript
// src/components/ProtectedNavigation.jsx
function ProtectedLink({ to, children }) {
  const { isAuthenticated } = useAuth();
  const location = useLocation();

  if (!isAuthenticated) {
    return (
      <Link 
        to="/login" 
        state={{ from: location.pathname }}
      >
        Login to Continue
      </Link>
    );
  }

  return <Link to={to}>{children}</Link>;
}

// Usage with navigation hook
function ProtectedNavigationButton({ to, children }) {
  const navigate = useNavigate();
  const { isAuthenticated } = useAuth();
  const location = useLocation();

  const handleClick = () => {
    if (!isAuthenticated) {
      navigate('/login', { 
        state: { from: location.pathname },
        replace: true 
      });
      return;
    }
    navigate(to);
  };

  return (
    <button onClick={handleClick}>
      {children}
    </button>
  );
}
```
Navigation guards protect routes by redirecting unauthorized users. They can be implemented at the component level or as part of the routing configuration.

## Custom Navigation Hooks

```javascript
// src/hooks/useCustomNavigation.js
function useCustomNavigation() {
  const navigate = useNavigate();
  const location = useLocation();

  return {
    goBack: () => navigate(-1),
    goForward: () => navigate(1),
    goToWithReturn: (path) => navigate(path, {
      state: { returnTo: location.pathname }
    }),
    returnToPrevious: () => {
      const returnTo = location.state?.returnTo;
      navigate(returnTo || '/', { replace: true });
    }
  };
}

// Usage
function NavigationComponent() {
  const { goBack, goToWithReturn, returnToPrevious } = useCustomNavigation();

  return (
    <div>
      <button onClick={goBack}>Back</button>
      <button onClick={() => goToWithReturn('/details')}>
        View Details
      </button>
      <button onClick={returnToPrevious}>
        Return
      </button>
    </div>
  );
}
```
Custom navigation hooks encapsulate common navigation patterns and make them reusable across your application.

## Navigation Events and Interception

```javascript
// src/components/NavigationAwareComponent.jsx
import { useNavigationType, useBeforeUnload } from 'react-router-dom';

function NavigationAwareComponent() {
  const navigationType = useNavigationType(); // "POP", "PUSH", or "REPLACE"
  const formRef = useRef();

  // Warn on unsaved changes
  useBeforeUnload(
    useCallback((event) => {
      if (formRef.current?.isDirty) {
        event.preventDefault();
        return event.returnValue = 'You have unsaved changes!';
      }
    }, [])
  );

  return (
    <div>
      {navigationType === 'POP' && (
        <div>You went back/forward in history</div>
      )}
      <form ref={formRef}>
        {/* Form content */}
      </form>
    </div>
  );
}
```
Navigation events and interception allow you to respond to navigation changes and potentially prevent navigation when needed.

## Common Patterns and Best Practices

### Breadcrumb Navigation
```javascript
function Breadcrumbs() {
  const location = useLocation();
  const paths = location.pathname.split('/')
    .filter(Boolean);

  return (
    <nav>
      <Link to="/">Home</Link>
      {paths.map((path, index) => {
        const to = `/${paths.slice(0, index + 1).join('/')}`;
        return (
          <span key={to}>
            {' > '}
            <Link to={to}>
              {path.charAt(0).toUpperCase() + path.slice(1)}
            </Link>
          </span>
        );
      })}
    </nav>
  );
}
```

Breadcrumbs provide hierarchical navigation context to users. This component automatically generates a breadcrumb trail based on the current URL path. It splits the pathname into segments, creates appropriate links for each level, and capitalizes the display text. For example, a URL like `/products/electronics/laptops` would generate a trail: Home > Products > Electronics > Laptops, where each segment is clickable and navigates to its respective level in the hierarchy.
### Navigation with Loading States
```javascript
function NavigationButton({ to, children }) {
  const navigate = useNavigate();
  const navigation = useNavigation();
  const isNavigating = navigation.state === "loading";

  return (
    <button 
      onClick={() => navigate(to)}
      disabled={isNavigating}
    >
      {isNavigating ? 'Loading...' : children}
    </button>
  );
}
```

