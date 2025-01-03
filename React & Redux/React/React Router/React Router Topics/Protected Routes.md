## Basic Authentication Guard

### File Location
```text
src/routes/protected.js
src/components/AuthGuard.jsx
src/utils/auth.js
```

### Basic Protected Route Setup
```javascript
// src/routes/protected.js
import { redirect } from 'react-router-dom';
import { checkAuth } from '../utils/auth';

export const protectedRoutes = {
  path: "dashboard",
  // Protect the route with a loader
  loader: async () => {
    const isAuthenticated = await checkAuth();
    if (!isAuthenticated) {
      // Save attempted URL
      const params = new URLSearchParams();
      params.set('from', window.location.pathname);
      return redirect(`/login?${params.toString()}`);
    }
    return null;
  },
  element: <DashboardLayout />,
  children: [
    {
      path: "profile",
      element: <Profile />
    },
    {
      path: "settings",
      element: <Settings />
    }
  ]
};
```
This basic setup protects routes using a loader function. It checks authentication before the route renders and redirects to login if needed, preserving the attempted URL for post-login redirection.

## Role-Based Protection

```javascript
// src/routes/adminRoutes.js
export const adminRoutes = {
  path: "admin",
  loader: async () => {
    const user = await getUser();
    if (!user) {
      throw redirect("/login");
    }
    if (!user.isAdmin) {
      throw new Response("Unauthorized", { status: 403 });
    }
    return { user };
  },
  element: <AdminLayout />,
  errorElement: <ErrorBoundary />,
  children: [
    {
      path: "users",
      element: <UserManagement />,
      // Additional role checks for specific routes
      loader: async () => {
        const user = await getUser();
        if (!user.permissions.includes('manage_users')) {
          throw new Response("Insufficient permissions", { status: 403 });
        }
        return null;
      }
    }
  ]
};
```
Role-based protection extends basic authentication by checking user roles and permissions. This example shows how to protect routes based on user roles and specific permissions.

## Higher-Order Component Protection

```javascript
// src/components/RequireAuth.jsx
import { useLocation, Navigate } from 'react-router-dom';

function RequireAuth({ children, requiredRole = null }) {
  const { user, isLoading } = useAuth();
  const location = useLocation();

  if (isLoading) {
    return <LoadingSpinner />;
  }

  if (!user) {
    return <Navigate 
      to="/login" 
      state={{ from: location.pathname }}
      replace 
    />;
  }

  if (requiredRole && !user.roles.includes(requiredRole)) {
    return <Navigate to="/unauthorized" replace />;
  }

  return children;
}

// Usage in routes
const router = createBrowserRouter([
  {
    path: "admin",
    element: (
      <RequireAuth requiredRole="admin">
        <AdminDashboard />
      </RequireAuth>
    )
  }
]);
```
This pattern uses a higher-order component to wrap protected content. It's useful when you need more complex protection logic or want to protect specific components rather than entire routes.

## Authentication Context Integration

```javascript
// src/context/AuthContext.jsx
const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const navigate = useNavigate();

  const login = async (credentials) => {
    const user = await loginUser(credentials);
    setUser(user);
    const returnTo = new URLSearchParams(window.location.search)
      .get('from') || '/dashboard';
    navigate(returnTo);
  };

  const logout = () => {
    setUser(null);
    navigate('/login');
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// Usage in protected loader
export async function protectedLoader({ request }) {
  const { user } = useAuth();
  
  if (!user) {
    const params = new URLSearchParams();
    params.set('from', new URL(request.url).pathname);
    return redirect(`/login?${params.toString()}`);
  }
  
  return null;
}
```
This pattern shows how to integrate authentication state management with protected routes, providing a centralized way to handle auth-related functionality.

## Persistent Authentication

```javascript
// src/utils/auth.js
export async function authenticatedLoader({ request }) {
  // Check for stored token
  const token = localStorage.getItem('token');
  
  if (!token) {
    return redirect('/login');
  }

  try {
    // Verify token is still valid
    const response = await fetch('/api/verify-token', {
      headers: {
        'Authorization': `Bearer ${token}`
      }
    });

    if (!response.ok) {
      // Token invalid - clear and redirect
      localStorage.removeItem('token');
      return redirect('/login');
    }

    const user = await response.json();
    return { user };
  } catch (error) {
    return redirect('/login');
  }
}

// Usage in routes
const router = createBrowserRouter([
  {
    path: "dashboard",
    loader: authenticatedLoader,
    element: <DashboardLayout />,
    children: [
      // Protected child routes
    ]
  }
]);
```
This pattern handles persistent authentication using stored tokens, verifying them on each protected route access.

## Error Handling for Protected Routes

```javascript
// src/components/AuthError.jsx
function AuthErrorBoundary() {
  const error = useRouteError();
  const navigate = useNavigate();

  if (error.status === 401) {
    return (
      <div>
        <h2>Please log in to continue</h2>
        <button onClick={() => navigate('/login')}>
          Go to Login
        </button>
      </div>
    );
  }

  if (error.status === 403) {
    return (
      <div>
        <h2>You don't have permission to access this page</h2>
        <button onClick={() => navigate(-1)}>
          Go Back
        </button>
      </div>
    );
  }

  return <div>An unexpected error occurred</div>;
}

// Usage in routes
{
  path: "protected",
  element: <ProtectedComponent />,
  errorElement: <AuthErrorBoundary />,
  loader: protectedLoader
}
```
This pattern shows how to handle different types of authentication and authorization errors gracefully.

## Common Patterns and Best Practices

### Loading States
```javascript
function ProtectedRoute({ children }) {
  const { user, isLoading } = useAuth();
  const location = useLocation();

  if (isLoading) {
    return <LoadingSpinner />;
  }

  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return children;
}
```
Handle loading states gracefully to prevent flashing of protected content or login screens.

### Redirect After Login
```javascript
function Login() {
  const [searchParams] = useSearchParams();
  const navigate = useNavigate();
  
  const handleLogin = async (credentials) => {
    await loginUser(credentials);
    const returnTo = searchParams.get('from') || '/dashboard';
    navigate(returnTo, { replace: true });
  };

  return <LoginForm onSubmit={handleLogin} />;
}
```
Properly handle post-login navigation by returning users to their intended destination.

