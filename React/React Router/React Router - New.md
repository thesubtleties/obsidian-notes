## Basic Router Setup
```javascript
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    errorElement: <ErrorPage />,
    children: [
      {
        index: true,
        element: <HomePage />
      },
      {
        path: "products",
        element: <Products />
      }
    ]
  }
]);

// App Component
function App() {
  return <RouterProvider router={router} />;
}
```
The `createBrowserRouter` is the foundation of your routing setup. It uses an object-based configuration where each route is defined as an object with properties like `path`, `element`, and optional `children` for nested routes. The `index: true` property indicates the default route for a parent path. The `RouterProvider` component wraps your app and provides routing context.

## Data Loading and Mutations
```javascript
// Route Configuration with Loader and Action
{
  path: "products",
  element: <Products />,
  loader: async () => {
    const products = await fetchProducts();
    if (!products) {
      throw new Error("Failed to load products");
    }
    return { products };
  },
  // Handle form submissions/mutations
  action: async ({ request }) => {
    const formData = await request.formData();
    return createProduct(formData);
  }
}

// Using the loaded data in component
import { useLoaderData, Form } from 'react-router-dom';

function Products() {
  const { products } = useLoaderData();
  
  return (
    <>
      {/* Automatic form handling */}
      <Form method="post">
        <input name="productName" />
        <button type="submit">Add Product</button>
      </Form>
      
      {products.map(product => (
        <div key={product.id}>{product.name}</div>
      ))}
    </>
  );
}
```
`loader` functions run before the component renders, ensuring data is available immediately. `action` functions handle form submissions and data mutations. The `useLoaderData` hook provides access to the data returned by the loader. The `Form` component automatically triggers the route's action when submitted, eliminating the need for manual form handling.

## Navigation and Actions
```javascript
import { useNavigate, Link, NavLink } from 'react-router-dom';

function Navigation() {
  const navigate = useNavigate();

  return (
    <>
      {/* Declarative navigation */}
      <Link to="/products">Products</Link>

      {/* Navigation with active states */}
      <NavLink 
        to="/products"
        className={({ isActive }) => isActive ? 'active' : ''}
      >
        Products
      </NavLink>

      {/* Programmatic navigation */}
      <button onClick={() => navigate('/products')}>
        Go to Products
      </button>
    </>
  );
}
```
React Router provides multiple ways to handle navigation. `Link` is for basic navigation, `NavLink` adds active state awareness (great for navigation menus), and `useNavigate` enables programmatic navigation (useful for redirects after form submissions or conditional navigation).

## Error Handling
```javascript
{
  path: "products",
  element: <Products />,
  // Global error element for this route and children
  errorElement: <ErrorBoundary />,
  // Loader with error handling
  loader: async () => {
    try {
      const data = await fetchProducts();
      if (!data) throw new Error("No data");
      return data;
    } catch (error) {
      throw new Response("Not Found", { status: 404 });
    }
  }
}

// Error Boundary Component
import { useRouteError } from 'react-router-dom';

function ErrorBoundary() {
  const error = useRouteError();
  
  return (
    <div>
      <h1>Oops!</h1>
      <p>{error.message || 'Something went wrong'}</p>
    </div>
  );
}
```
Error handling is built into the routing system. The `errorElement` catches errors from the route's loader, action, or rendering process. `useRouteError` provides access to the error that was thrown, allowing for custom error UI based on the error type.

## Route Parameters and Search Params
```javascript
// Route with parameters
{
  path: "products/:id",
  element: <ProductDetail />,
  loader: async ({ params }) => {
    return fetchProduct(params.id);
  }
}

// Using URL Search Params
import { useSearchParams } from 'react-router-dom';

function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();
  const category = searchParams.get('category');

  return (
    <>
      <select
        value={category}
        onChange={(e) => setSearchParams({ category: e.target.value })}
      >
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
      </select>
    </>
  );
}
```
Route parameters (`:id`) capture dynamic values from the URL path. The loader receives these parameters automatically. `useSearchParams` provides a way to read and modify URL search parameters (query string), useful for filtering, sorting, or pagination.

## Deferred Data Loading
```javascript
// Deferred loading for slow data
{
  path: "products",
  element: <Products />,
  loader: async () => {
    return defer({
      products: fetchProducts(),
      slowData: fetchSlowData()
    });
  }
}

// Component with deferred data
import { Await, useLoaderData } from 'react-router-dom';
import { Suspense } from 'react';

function Products() {
  const { products, slowData } = useLoaderData();

  return (
    <>
      {/* Regular data */}
      <ProductList products={products} />
      
      {/* Deferred data with loading state */}
      <Suspense fallback={<Loading />}>
        <Await resolve={slowData}>
          {(resolvedData) => <SlowComponent data={resolvedData} />}
        </Await>
      </Suspense>
    </>
  );
}
```
Deferred loading allows you to render a route before all data is loaded. The `defer` function lets you specify which data can load after initial render. The `Await` component, wrapped in `Suspense`, handles the loading state for deferred data. This is particularly useful for optimizing the user experience when some data takes longer to load.

## Key Points & Best Practices
- Use `loader` for data fetching before component render
- Use `action` for form submissions and data mutations
- Implement `errorElement` for graceful error handling
- Use `Form` component for automatic form handling
- Leverage `useLoaderData` to access loaded data
- Use `defer` for slow data that shouldn't block rendering
- Implement proper error boundaries with `errorElement`
- Use `NavLink` for navigation with active states
- Leverage URL parameters and search params for state management

## Common Gotchas
- Loaders must return data or throw
- Actions should handle form submissions
- Always handle errors in loaders and actions
- Don't forget to wrap app with `RouterProvider`
- Use `defer` only when necessary (slow data)
- Remember to handle loading states with `Suspense`