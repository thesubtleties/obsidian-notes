## Basic Route Structure

### File Location
```text
src/routes/index.js (or .tsx for TypeScript)
```

### Basic Configuration Example
```javascript
// src/routes/index.js
import { createBrowserRouter } from 'react-router-dom';

export const router = createBrowserRouter([
  {
    path: "/",                    // URL path
    element: <RootLayout />,      // Component to render
    errorElement: <ErrorPage />,  // Error handling
    loader: rootLoader,           // Data loading
    action: rootAction,           // Form handling
    children: []                  // Nested routes
  }
]);
```

## Comprehensive Route Patterns

### Feature-Based Route Organization
```javascript
// src/routes/index.js
import { productRoutes } from './productRoutes';
import { userRoutes } from './userRoutes';
import { authRoutes } from './authRoutes';

export const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    errorElement: <ErrorPage />,
    children: [
      {
        index: true,
        element: <HomePage />,
        loader: homeLoader
      },
      // Feature routes spread as children
      ...productRoutes,
      ...userRoutes,
      ...authRoutes
    ]
  }
]);

// src/routes/productRoutes.js
export const productRoutes = [
  {
    path: "products",
    element: <ProductsLayout />,
    loader: productsLoader,
    children: [
      {
        index: true,
        element: <ProductsList />,
        loader: productsListLoader
      },
      {
        path: ":id",
        element: <ProductDetail />,
        loader: productLoader,
        action: productAction
      }
    ]
  }
];
```

## Route Types and Patterns

### Index Routes
```javascript
{
  path: "dashboard",
  element: <DashboardLayout />,
  children: [
    {
      index: true,  // Matches exactly /dashboard
      element: <DashboardHome />
    }
  ]
}
```

### Dynamic Routes
```javascript
{
  path: "products/:id",  // Dynamic parameter
  element: <ProductDetail />,
  loader: ({ params }) => {
    return fetchProduct(params.id);
  }
}
```

### Optional Parameters
```javascript
{
  path: "products/:id?",  // Optional parameter
  element: <ProductPage />,
  loader: ({ params }) => {
    return params.id ? fetchProduct(params.id) : fetchProducts();
  }
}
```

### Splat Routes (Catch-all)
```javascript
{
  path: "docs/*",  // Matches any path after docs/
  element: <DocsPage />,
  loader: ({ params }) => {
    return fetchDoc(params["*"]);
  }
}
```

## Advanced Configuration Patterns

### Lazy Loading Routes
```javascript
// src/routes/index.js
import { lazy } from 'react';

const ProductDetail = lazy(() => import('../pages/ProductDetail'));

{
  path: "products/:id",
  element: (
    <Suspense fallback={<LoadingSpinner />}>
      <ProductDetail />
    </Suspense>
  ),
  loader: productLoader
}
```

### Route Guards
```javascript
{
  path: "admin",
  loader: async () => {
    const user = await getUser();
    if (!user?.isAdmin) {
      throw redirect("/login?from=/admin");
    }
    return user;
  },
  element: <AdminLayout />,
  children: [
    {
      path: "dashboard",
      element: <AdminDashboard />
    }
  ]
}
```

### Shared Layouts
```javascript
{
  path: "dashboard",
  element: <DashboardLayout />,
  loader: dashboardLoader,
  // These routes share DashboardLayout
  children: [
    {
      path: "profile",
      element: <Profile />,
      loader: profileLoader
    },
    {
      path: "settings",
      element: <Settings />,
      loader: settingsLoader
    }
  ]
}
```

## TypeScript Support

### Route Type Definitions
```typescript
// src/types/routes.ts
import { RouteObject } from 'react-router-dom';

interface AppRouteObject extends RouteObject {
  children?: AppRouteObject[];
  loader?: (args: {
    params: Record<string, string>;
    request: Request;
  }) => Promise<any>;
  action?: (args: {
    params: Record<string, string>;
    request: Request;
  }) => Promise<any>;
}

// Usage
const routes: AppRouteObject[] = [
  {
    path: "/",
    element: <RootLayout />,
    children: []
  }
];
```

## Common Patterns and Best Practices

### Route Organization
```javascript
// src/routes/index.js
import { publicRoutes } from './publicRoutes';
import { privateRoutes } from './privateRoutes';
import { authRoutes } from './authRoutes';

export const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    children: [
      ...publicRoutes,    // Public accessible routes
      ...authRoutes,      // Authentication routes
      {
        element: <AuthRequired />,  // Protected route wrapper
        children: privateRoutes     // Protected routes
      }
    ]
  }
]);
```

### Route Constants
```javascript
// src/constants/routes.js
export const ROUTES = {
  HOME: "/",
  PRODUCTS: "/products",
  PRODUCT_DETAIL: (id: string) => `/products/${id}`,
  ADMIN: {
    ROOT: "/admin",
    DASHBOARD: "/admin/dashboard",
    USERS: "/admin/users"
  }
} as const;

// Usage in navigation
navigate(ROUTES.PRODUCT_DETAIL(productId));
```

## Error Handling Patterns

### Route-Level Error Handling
```javascript
{
  path: "products",
  element: <Products />,
  errorElement: <ProductsError />,
  children: [
    {
      path: ":id",
      element: <ProductDetail />,
      errorElement: <ProductError />,  // More specific error
      loader: async ({ params }) => {
        const response = await fetch(`/api/products/${params.id}`);
        if (!response.ok) {
          throw new Response("Not Found", { status: 404 });
        }
        return response.json();
      }
    }
  ]
}
```

## Common Gotchas and Solutions

### Path Resolution
```javascript
// ❌ Incorrect - Relative path
{
  path: "dashboard",  // Resolves relative to parent
  children: [
    {
      path: "/settings"  // Will NOT work as expected
    }
  ]
}

// ✅ Correct - Absolute vs Relative paths
{
  path: "dashboard",
  children: [
    {
      path: "settings"  // Correctly resolves to /dashboard/settings
    }
  ]
}
```

### Route Priority
```javascript
// ❌ Incorrect order - Specific route shadowed
{
  path: "products/*",    // Catches everything
  element: <Products />
},
{
  path: "products/:id",  // Never reached
  element: <ProductDetail />
}

// ✅ Correct order
{
  path: "products/:id",  // Specific route first
  element: <ProductDetail />
},
{
  path: "products/*",    // Catch-all last
  element: <Products />
}
```

