## Quick Links
- [[#Basic Setup]]
- [[#Route Configuration]]
- [[#Data Loading]]
- [[#Form Handling]]
- [[#Layouts & Outlets]]
- [[#Navigation]]
- [[#Protected Routes]]
- [[#Error Handling]]
- [[#Project Structure]]

## [[Basic Setup]]
### File Location
```text
src/App.jsx
```

### Code Example
```javascript
// Import core router components
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import RootLayout from './layouts/RootLayout';
import ErrorPage from './pages/ErrorPage';

const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    errorElement: <ErrorPage />,
    children: [
      {
        index: true,
        element: <HomePage />,
        loader: homeLoader
      }
    ]
  }
]);

export default function App() {
  return <RouterProvider router={router} />;
}
```

### Key Concepts
- Router setup is now object-based instead of JSX
- `RouterProvider` is the single entry point
- Routes can have loaders, actions, and error boundaries

### Common Patterns
- One root layout with error handling
- Nested routes for feature sections
- Data loading at route level

## [[Route Configuration]]

### File Location
```text
src/routes/index.js
```

### Code Example
```javascript
// Centralized route configuration
export const routes = [
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
      {
        path: "products",
        element: <ProductsLayout />,
        loader: productsLoader,
        children: [
          {
            index: true,
            element: <ProductsList />
          },
          {
            path: ":id",
            element: <ProductDetail />,
            loader: productLoader
          }
        ]
      }
    ]
  }
];
```

### Key Concepts
- Nested routing structure
- Index routes for parent paths
- Dynamic route parameters
- Shared layouts for sections

### Common Patterns
- Group related routes together
- Use index routes for parent paths
- Separate route configs by feature

## [[Data Loading]]

### File Location
```text
src/loaders/productLoader.js
```

### Code Example
```javascript
// Loader function for product routes
export async function productsLoader({ request, params }) {
  // Access URL search params
  const url = new URL(request.url);
  const search = url.searchParams.get("q");
  
  try {
    const products = await fetchProducts({ search });
    return { products };
  } catch (error) {
    throw new Response("Failed to load products", { 
      status: 500 
    });
  }
}

// In component
import { useLoaderData, useNavigation } from 'react-router-dom';

function Products() {
  const { products } = useLoaderData();
  const navigation = useNavigation();
  
  if (navigation.state === "loading") {
    return <Spinner />;
  }
  
  return <ProductList products={products} />;
}
```

### Key Concepts
- Loaders run before component renders
- Access to URL and params
- Built-in error handling
- Loading states via useNavigation

### Common Patterns
- Data fetching in loaders
- Error handling with throw
- Loading states in components

## [[Form Handling]]

### File Location
```text
src/actions/productActions.js
src/components/ProductForm.jsx
```

### Code Example
```javascript
// src/actions/productActions.js
export async function createProductAction({ request }) {
  const formData = await request.formData();
  const productData = Object.fromEntries(formData);
  
  try {
    const response = await fetch('/api/products', {
      method: 'POST',
      body: JSON.stringify(productData)
    });
    
    if (!response.ok) {
      const error = await response.json();
      return { errors: error.details };
    }
    
    return redirect('/products');
  } catch (error) {
    return { errors: { _form: 'Failed to create product' } };
  }
}

// src/components/ProductForm.jsx
import { Form, useActionData, useNavigation } from 'react-router-dom';

function ProductForm() {
  const actionData = useActionData();
  const navigation = useNavigation();
  
  return (
    <Form method="post">
      <input 
        name="name" 
        aria-invalid={actionData?.errors?.name ? true : undefined}
      />
      {actionData?.errors?.name && (
        <span className="error">{actionData.errors.name}</span>
      )}
      
      <button 
        type="submit" 
        disabled={navigation.state === "submitting"}
      >
        {navigation.state === "submitting" ? "Saving..." : "Save"}
      </button>
    </Form>
  );
}
```

### Key Concepts
- Forms submit to route actions
- Built-in form data handling
- Validation and error returns
- Loading states during submission

### Common Patterns
- Form validation in actions
- Error display in components
- Redirect after success
- Disable during submission

## Layouts & Outlets

### File Location
```text
src/layouts/RootLayout.jsx
src/layouts/ProductsLayout.jsx
```

### Code Example
```javascript
// src/layouts/RootLayout.jsx
import { Outlet, useNavigation } from 'react-router-dom';

function RootLayout() {
  const navigation = useNavigation();
  
  return (
    <div>
      <Header />
      <main className={navigation.state === "loading" ? "loading" : ""}>
        <Outlet context={{ theme: 'light' }} />
      </main>
      <Footer />
    </div>
  );
}

// Using outlet context in child route
function ChildComponent() {
  const { theme } = useOutletContext();
  return <div className={theme}>...</div>;
}
```

### Key Concepts
- Shared UI for multiple routes
- Context passing to child routes
- Loading states for navigation
- Nested layouts possible

### Common Patterns
- Navigation in layouts
- Shared data via context
- Loading indicators
- Persistent UI elements

## Navigation

### File Location
```text
src/components/Navigation.jsx
```

### Code Example
```javascript
import { 
  Link, 
  NavLink, 
  useNavigate, 
  useSearchParams 
} from 'react-router-dom';

function Navigation() {
  const navigate = useNavigate();
  const [searchParams, setSearchParams] = useSearchParams();

  return (
    <nav>
      {/* Basic link */}
      <Link to="/products">Products</Link>

      {/* Active state aware link */}
      <NavLink 
        to="/dashboard"
        className={({ isActive, isPending }) => 
          isPending ? "pending" : isActive ? "active" : ""
        }
      >
        Dashboard
      </NavLink>

      {/* Programmatic navigation */}
      <button onClick={() => navigate('/cart')}>
        View Cart
      </button>

      {/* Search params handling */}
      <select
        value={searchParams.get("sort") || ""}
        onChange={e => setSearchParams({ sort: e.target.value })}
      >
        <option value="">Default</option>
        <option value="price">Price</option>
      </select>
    </nav>
  );
}
```

### Key Concepts
- Multiple navigation methods
- Active state handling
- Search params management
- Programmatic navigation

### Common Patterns
- NavLink for navigation menus
- Search params for filters
- Programmatic redirects
- Pending states for loading

## Protected Routes

### File Location
```text
src/routes/protected.js
src/utils/auth.js
```

### Code Example
```javascript
// src/routes/protected.js
import { redirect } from 'react-router-dom';
import { checkAuth } from '../utils/auth';

export const protectedRoutes = {
  path: "admin",
  loader: async () => {
    const isAuthenticated = await checkAuth();
    if (!isAuthenticated) {
      throw redirect("/login?from=/admin");
    }
    return null;
  },
  element: <AdminLayout />,
  children: [
    {
      path: "dashboard",
      element: <AdminDashboard />,
      loader: adminLoader
    }
  ]
};

// Using in component if needed
function AdminComponent() {
  const navigate = useNavigate();
  const { user } = useAuth();

  useEffect(() => {
    if (!user) navigate('/login');
  }, [user, navigate]);

  return <div>Admin Content</div>;
}
```

### Key Concepts
- Route protection via loaders
- Redirect unauthorized access
- Authentication state handling
- Nested protected routes

### Common Patterns
- Auth check in loaders
- Redirect with return path
- Protected layouts
- Role-based access

## Error Handling

### File Location
```text
src/pages/ErrorPage.jsx
src/components/ErrorBoundary.jsx
```

### Code Example
```javascript
// Route configuration with error element
{
  path: "products",
  element: <Products />,
  errorElement: <ErrorBoundary />,
  loader: async () => {
    const response = await fetch('/api/products');
    if (!response.ok) {
      throw new Response("Failed to load products", { 
        status: response.status 
      });
    }
    return response.json();
  }
}

// Error boundary component
import { useRouteError } from 'react-router-dom';

function ErrorBoundary() {
  const error = useRouteError();
  
  if (error.status === 404) {
    return <h2>Not Found</h2>;
  }
  
  return (
    <div className="error-container">
      <h2>Something went wrong!</h2>
      <p>{error.message || 'Unknown error occurred'}</p>
    </div>
  );
}
```

### Key Concepts
- Route level error handling
- Error boundary components
- Different error types
- Error recovery options

### Common Patterns
- Custom error pages
- Status-based errors
- Error recovery UI
- Nested error boundaries

## Project Structure

### File Location
```text
/src
```

### Structure Example
```text
src/
├── App.jsx                 # Main router setup
├── components/            # Reusable components
│   ├── forms/
│   ├── layout/
│   └── ui/
├── layouts/              # Layout components
│   ├── RootLayout.jsx
│   └── feature layouts...
├── pages/               # Page components
│   └── feature pages...
├── routes/              # Route configurations
│   ├── index.js
│   └── feature routes...
├── loaders/             # Data loading functions
│   └── feature loaders...
├── actions/             # Form actions
│   └── feature actions...
├── utils/              # Utility functions
│   ├── auth.js
│   └── api.js
└── types/              # TypeScript types (if using TS)
    └── feature types...
```

### Key Concepts
- Feature-based organization
- Clear separation of concerns
- Modular route configuration
- Scalable structure

### Common Patterns
- Group by feature
- Separate data fetching
- Centralize route config
- Reusable components

Would you like me to add any additional sections or expand on any of these topics?