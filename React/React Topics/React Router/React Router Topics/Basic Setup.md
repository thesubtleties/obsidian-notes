# React Router v6.4+ Basic Setup Cheatsheet

## Initial Setup

### Installation
```bash
# In your project directory
npm install react-router-dom@latest
```

## Basic Router Configuration

### File Location
```text
src/App.jsx
```

### Minimal Setup
```javascript
// src/App.jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

// Import your components
import RootLayout from './layouts/RootLayout';
import HomePage from './pages/HomePage';
import ErrorPage from './pages/ErrorPage';

const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    errorElement: <ErrorPage />,
    children: [
      {
        index: true,  // This makes it the default route
        element: <HomePage />
      }
    ]
  }
]);

export default function App() {
  return <RouterProvider router={router} />;
}
```

### Complete Example with Common Patterns
```javascript
// src/App.jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

// Layouts
import RootLayout from './layouts/RootLayout';
import ProductsLayout from './layouts/ProductsLayout';

// Pages
import HomePage from './pages/HomePage';
import ProductsList from './pages/ProductsList';
import ProductDetail from './pages/ProductDetail';
import ErrorPage from './pages/ErrorPage';

// Loaders and Actions
import { 
  productsLoader, 
  productLoader 
} from './loaders/productLoaders';
import { 
  createProductAction, 
  updateProductAction 
} from './actions/productActions';

const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    errorElement: <ErrorPage />,
    children: [
      {
        index: true,
        element: <HomePage />,
        loader: async () => {
          // Load any data needed for homepage
          return { message: "Welcome!" };
        }
      },
      {
        path: "products",
        element: <ProductsLayout />,
        loader: productsLoader,
        children: [
          {
            index: true,
            element: <ProductsList />,
            loader: productsLoader
          },
          {
            path: ":id",
            element: <ProductDetail />,
            loader: productLoader,
            action: updateProductAction
          },
          {
            path: "new",
            element: <ProductForm />,
            action: createProductAction
          }
        ]
      }
    ]
  }
]);

export default function App() {
  return <RouterProvider router={router} />;
}
```

## Basic Layout Setup

### File Location
```text
src/layouts/RootLayout.jsx
```

```javascript
// src/layouts/RootLayout.jsx
import { Outlet, useNavigation } from 'react-router-dom';
import MainNav from '../components/MainNav';

function RootLayout() {
  const navigation = useNavigation();
  
  return (
    <div className="app">
      <MainNav />
      
      <main className={navigation.state === "loading" ? "loading" : ""}>
        <Outlet />
      </main>
      
      <footer>Your App Footer</footer>
    </div>
  );
}

export default RootLayout;
```

## Basic Error Page

### File Location
```text
src/pages/ErrorPage.jsx
```

```javascript
// src/pages/ErrorPage.jsx
import { useRouteError } from 'react-router-dom';

function ErrorPage() {
  const error = useRouteError();
  
  return (
    <div className="error-page">
      <h1>Oops!</h1>
      <p>Sorry, an unexpected error has occurred.</p>
      <p>
        <i>{error.statusText || error.message}</i>
      </p>
    </div>
  );
}

export default ErrorPage;
```

## Initial Project Structure
```text
src/
├── App.jsx                # Main router setup
├── components/           
│   └── MainNav.jsx       # Main navigation component
├── layouts/              
│   └── RootLayout.jsx    # Root layout with outlet
├── pages/                
│   ├── HomePage.jsx      # Home page component
│   └── ErrorPage.jsx     # Error page component
├── loaders/              
│   └── index.js          # Data loading functions
└── actions/              
    └── index.js          # Form actions
```

## Key Concepts
- Router setup is object-based
- All routes are defined in a single configuration
- Layouts use Outlet for nested routes
- Error boundaries catch all errors
- Loaders handle data fetching
- Actions handle form submissions

## Common Patterns

### Navigation Setup
```javascript
// src/components/MainNav.jsx
import { Link, NavLink } from 'react-router-dom';

function MainNav() {
  return (
    <nav>
      <NavLink 
        to="/"
        className={({ isActive }) => 
          isActive ? "active" : ""
        }
      >
        Home
      </NavLink>
      <NavLink 
        to="/products"
        className={({ isActive }) => 
          isActive ? "active" : ""
        }
      >
        Products
      </NavLink>
    </nav>
  );
}
```

### Basic Loader Example
```javascript
// src/loaders/index.js
export async function basicLoader() {
  const response = await fetch('/api/data');
  if (!response.ok) {
    throw new Response("Not Found", { status: 404 });
  }
  return response.json();
}
```

### Basic Action Example
```javascript
// src/actions/index.js
export async function basicAction({ request }) {
  const formData = await request.formData();
  const data = Object.fromEntries(formData);
  
  const response = await fetch('/api/data', {
    method: 'POST',
    body: JSON.stringify(data)
  });
  
  if (!response.ok) {
    return { error: "Failed to save" };
  }
  
  return { success: true };
}
```

## Common Gotchas
- Always wrap app with RouterProvider
- Don't forget errorElement for error handling
- Loaders must return something or throw
- Use index: true for default child routes
- Remember to export components
- Keep route configuration clean and organized

## TypeScript Support
```typescript
// If using TypeScript, define your route types
import { RouteObject } from 'react-router-dom';

const routes: RouteObject[] = [
  {
    path: "/",
    element: <RootLayout />,
    children: []
  }
];
```

This basic setup provides a foundation for building a React Router v6.4+ application. You can expand upon this structure as your application grows.