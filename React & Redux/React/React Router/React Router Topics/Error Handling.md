## Basic Error Boundary Setup

### File Location
```text
src/components/ErrorBoundary.jsx
src/routes/index.js
```

### Basic Error Boundary
```javascript
// src/components/ErrorBoundary.jsx
import { useRouteError } from 'react-router-dom';

function ErrorBoundary() {
  const error = useRouteError();
  
  // Handle different types of errors
  if (isRouteErrorResponse(error)) {
    if (error.status === 404) {
      return <h2>Page not found!</h2>;
    }
    if (error.status === 401) {
      return <h2>You aren't authorized to see this</h2>;
    }
    if (error.status === 503) {
      return <h2>Looks like our API is down</h2>;
    }
  }

  // Handle unexpected errors
  return (
    <div className="error-container">
      <h2>Something went wrong!</h2>
      <pre>{error.message || 'Unknown error'}</pre>
    </div>
  );
}

// In routes configuration
const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    errorElement: <ErrorBoundary />,
    children: [
      // Routes...
    ]
  }
]);
```
This basic setup catches all errors in your application. The `useRouteError` hook provides access to the error that was thrown, allowing you to show appropriate error messages based on the error type.

## Hierarchical Error Handling

```javascript
// src/routes/index.js
const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    errorElement: <RootErrorBoundary />, // Catches all uncaught errors
    children: [
      {
        path: "products",
        element: <ProductsLayout />,
        errorElement: <ProductsErrorBoundary />, // Specific to products
        children: [
          {
            path: ":id",
            element: <ProductDetail />,
            errorElement: <ProductErrorBoundary />, // Even more specific
            loader: productLoader
          }
        ]
      }
    ]
  }
]);

// src/components/ProductErrorBoundary.jsx
function ProductErrorBoundary() {
  const error = useRouteError();
  const navigate = useNavigate();

  return (
    <div className="product-error">
      <h2>Product Error</h2>
      <p>{error.message}</p>
      <button onClick={() => navigate('/products')}>
        Back to Products
      </button>
    </div>
  );
}
```
Error boundaries can be nested, allowing for more specific error handling at different levels of your application.

## Loader Error Handling

```javascript
// src/loaders/productLoader.js
export async function productLoader({ params }) {
  try {
    const response = await fetch(`/api/products/${params.id}`);
    
    if (!response.ok) {
      if (response.status === 404) {
        throw new Response("Product not found", { 
          status: 404,
          statusText: "Not Found"
        });
      }
      
      if (response.status === 401) {
        throw new Response("Unauthorized", { 
          status: 401,
          statusText: "Please login to view this product"
        });
      }
      
      throw new Response("Failed to load product", { 
        status: response.status 
      });
    }

    return response.json();
  } catch (error) {
    if (error instanceof Response) throw error;
    throw new Response("Server Error", { status: 500 });
  }
}
```
Loaders can throw different types of errors that will be caught by error boundaries. This allows for specific error handling based on the type of error encountered during data loading.

## Action Error Handling

```javascript
// src/actions/productActions.js
export async function productAction({ request }) {
  const formData = await request.formData();
  
  try {
    const response = await fetch('/api/products', {
      method: 'POST',
      body: JSON.stringify(Object.fromEntries(formData))
    });

    if (!response.ok) {
      // Return validation errors to the form
      if (response.status === 422) {
        const errors = await response.json();
        return { errors };
      }
      
      // Throw other errors to error boundary
      throw new Response(
        `Failed to create product: ${response.statusText}`, 
        { status: response.status }
      );
    }

    return redirect('/products');
  } catch (error) {
    if (error instanceof Response) throw error;
    throw new Response("Server Error", { status: 500 });
  }
}
```
Actions can either return errors to the form or throw them to be caught by error boundaries, depending on the type of error.

## Custom Error Components

```javascript
// src/components/ErrorComponents.jsx
export function NotFoundError() {
  const error = useRouteError();
  const navigate = useNavigate();

  return (
    <div className="error-container">
      <h1>Page Not Found</h1>
      <p>{error.statusText}</p>
      <button onClick={() => navigate(-1)}>Go Back</button>
    </div>
  );
}

export function NetworkError() {
  const error = useRouteError();
  const revalidator = useRevalidator();

  return (
    <div className="error-container">
      <h1>Network Error</h1>
      <p>{error.message}</p>
      <button onClick={() => revalidator.revalidate()}>
        Try Again
      </button>
    </div>
  );
}

// Usage in routes
{
  path: "products/:id",
  element: <ProductDetail />,
  errorElement: error.status === 404 
    ? <NotFoundError /> 
    : <NetworkError />,
  loader: productLoader
}
```
Custom error components can provide specific UI and functionality for different types of errors.

## Error Recovery Patterns

```javascript
// src/components/ErrorRecovery.jsx
function ErrorBoundaryWithRetry() {
  const error = useRouteError();
  const navigation = useNavigation();
  const revalidator = useRevalidator();

  return (
    <div className="error-container">
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      
      {/* Show different recovery options */}
      <div className="error-actions">
        {/* Retry data loading */}
        <button 
          onClick={() => revalidator.revalidate()}
          disabled={navigation.state !== "idle"}
        >
          Try Again
        </button>

        {/* Navigate away */}
        <button onClick={() => navigate(-1)}>
          Go Back
        </button>

        {/* Reset to known good state */}
        <button onClick={() => navigate('/', { replace: true })}>
          Go Home
        </button>
      </div>
    </div>
  );
}
```
Error boundaries can include recovery mechanisms like retrying failed operations or navigating to safe states.

## Common Patterns and Best Practices

### Global Error Handler
```javascript
// src/components/GlobalError.jsx
function GlobalErrorBoundary() {
  const error = useRouteError();
  const { user } = useAuth();

  // Log errors to your error tracking service
  useEffect(() => {
    if (error) {
      logError(error, {
        user: user?.id,
        path: window.location.pathname
      });
    }
  }, [error, user]);

  return (
    <div className="global-error">
      <h1>Oops!</h1>
      <p>Something went wrong</p>
      {process.env.NODE_ENV === 'development' && (
        <pre>{error.stack}</pre>
      )}
    </div>
  );
}
```
A global error handler can provide consistent error handling and logging across your application.

### Error Types and Handling
```javascript
// Common error handling patterns
if (!response.ok) {
  // API errors
  throw new Response(response.statusText, { 
    status: response.status 
  });
}

// Validation errors
if (!isValid) {
  return { errors: validationErrors };
}

// Authentication errors
if (!user) {
  throw redirect('/login');
}

// Not found
if (!data) {
  throw new Response('Not Found', { status: 404 });
}
```
These patterns show common ways to handle different types of errors in your application.

