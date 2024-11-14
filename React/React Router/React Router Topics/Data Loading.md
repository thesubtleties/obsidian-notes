## Basic Loader Setup

### File Location
```text
src/loaders/productLoaders.js
src/routes/index.js
```

### Basic Loader Example
```javascript
// src/loaders/productLoaders.js
export async function productLoader({ params, request }) {
  // Access URL params and search params
  const url = new URL(request.url);
  const searchTerm = url.searchParams.get("search");
  const { id } = params;

  try {
    const response = await fetch(`/api/products/${id}?search=${searchTerm}`);
    
    if (!response.ok) {
      throw new Response("Product not found", { status: 404 });
    }
    
    return response.json();
  } catch (error) {
    throw new Response("Failed to load product", { status: 500 });
  }
}

// src/routes/index.js
import { productLoader } from '../loaders/productLoaders';

const router = createBrowserRouter([
  {
    path: "products/:id",
    element: <ProductDetail />,
    loader: productLoader
  }
]);
```

## Using Loader Data

### File Location
```text
src/components/ProductDetail.jsx
```

```javascript
// src/components/ProductDetail.jsx
import { useLoaderData, useNavigation } from 'react-router-dom';

function ProductDetail() {
  const product = useLoaderData();
  const navigation = useNavigation();
  
  // Show loading state during navigation
  if (navigation.state === "loading") {
    return <LoadingSpinner />;
  }

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </div>
  );
}
```

## Advanced Loading Patterns

### Parallel Data Loading
```javascript
// src/loaders/dashboardLoader.js
export async function dashboardLoader() {
  // Load multiple data sources in parallel
  const [users, products, orders] = await Promise.all([
    fetch('/api/users').then(r => r.json()),
    fetch('/api/products').then(r => r.json()),
    fetch('/api/orders').then(r => r.json())
  ]);

  return {
    users,
    products,
    orders
  };
}
```

### Deferred Loading
```javascript
// src/loaders/productLoader.js
import { defer } from 'react-router-dom';

export async function productLoader({ params }) {
  // Load critical data immediately
  const productPromise = fetch(`/api/products/${params.id}`)
    .then(r => r.json());

  // Defer slow data
  const reviewsPromise = fetch(`/api/products/${params.id}/reviews`)
    .then(r => r.json());

  return defer({
    product: await productPromise, // Wait for critical data
    reviews: reviewsPromise        // Load after render
  });
}

// src/components/ProductDetail.jsx
import { Await, useLoaderData, Suspense } from 'react-router-dom';

function ProductDetail() {
  const { product, reviews } = useLoaderData();

  return (
    <div>
      {/* Critical data available immediately */}
      <h1>{product.name}</h1>
      
      {/* Deferred data with loading state */}
      <Suspense fallback={<LoadingSpinner />}>
        <Await resolve={reviews}>
          {(resolvedReviews) => (
            <ReviewsList reviews={resolvedReviews} />
          )}
        </Await>
      </Suspense>
    </div>
  );
}
```

## Error Handling in Loaders

### File Location
```text
src/loaders/productLoaders.js
```

```javascript
// Comprehensive error handling
export async function productLoader({ params }) {
  try {
    const response = await fetch(`/api/products/${params.id}`);

    // Handle different error scenarios
    if (response.status === 404) {
      throw new Response("Product not found", { status: 404 });
    }

    if (response.status === 401) {
      throw new Response("Unauthorized", { 
        status: 401,
        statusText: "Please login to view this product"
      });
    }

    if (!response.ok) {
      throw new Response("Failed to load product", { 
        status: response.status 
      });
    }

    return response.json();
  } catch (error) {
    // Handle network errors or other exceptions
    if (error instanceof Response) throw error;
    throw new Response("Server Error", { status: 500 });
  }
}
```

## Authentication in Loaders

### File Location
```text
src/loaders/protectedLoader.js
```

```javascript
// Protected route loader
export async function protectedLoader() {
  const token = localStorage.getItem('token');

  if (!token) {
    // Save attempted URL
    const params = new URLSearchParams();
    params.set('from', window.location.pathname);
    
    throw redirect(`/login?${params.toString()}`);
  }

  try {
    const response = await fetch('/api/protected-data', {
      headers: {
        Authorization: `Bearer ${token}`
      }
    });

    if (response.status === 401) {
      throw redirect('/login');
    }

    return response.json();
  } catch (error) {
    throw new Response("Failed to load data", { status: 500 });
  }
}
```

## Loading States and UI

### File Location
```text
src/components/LoadingUI.jsx
```

```javascript
import { useNavigation } from 'react-router-dom';

function LoadingAwareComponent() {
  const navigation = useNavigation();
  
  return (
    <div className={navigation.state !== "idle" ? "loading" : ""}>
      {/* Navigation states: "idle" | "loading" | "submitting" */}
      {navigation.state === "loading" && (
        <LoadingBar />
      )}
      
      {/* Show loading location */}
      {navigation.state === "loading" && (
        <p>Loading {navigation.location.pathname}</p>
      )}
      
      <Content />
    </div>
  );
}
```

## TypeScript Support

### File Location
```text
src/types/loaders.ts
```

```typescript
// Loader type definitions
import { LoaderFunction } from 'react-router-dom';

interface ProductData {
  id: string;
  name: string;
  price: number;
}

export const productLoader: LoaderFunction = async ({
  params,
  request
}): Promise<ProductData> => {
  const response = await fetch(`/api/products/${params.id}`);
  if (!response.ok) {
    throw new Response('Not Found', { status: 404 });
  }
  return response.json();
};
```

## Common Patterns and Best Practices

### Data Revalidation
```javascript
// Force reload of data
import { useRevalidator } from 'react-router-dom';

function RevalidateButton() {
  const revalidator = useRevalidator();
  
  return (
    <button
      onClick={() => revalidator.revalidate()}
      disabled={revalidator.state === "loading"}
    >
      Refresh Data
    </button>
  );
}
```

### Loader Organization
```javascript
// src/loaders/index.js
export { productLoader } from './productLoader';
export { categoryLoader } from './categoryLoader';
export { searchLoader } from './searchLoader';

// Group related loaders
export const shopLoaders = {
  product: productLoader,
  category: categoryLoader,
  search: searchLoader
};
```

## Common Gotchas

### Loader Return Values
```javascript
// ❌ Incorrect - Undefined return
export async function badLoader() {
  const response = await fetch('/api/data');
  if (response.ok) {
    return response.json();
  }
  // Missing return/throw for error case
}

// ✅ Correct - Always return or throw
export async function goodLoader() {
  const response = await fetch('/api/data');
  if (!response.ok) {
    throw new Response('Error', { status: response.status });
  }
  return response.json();
}
```

### Error Handling
```javascript
// ❌ Incorrect - Raw error throwing
export async function badLoader() {
  try {
    const data = await fetchData();
    return data;
  } catch (error) {
    throw error; // Raw error might expose sensitive details
  }
}

// ✅ Correct - Proper error response
export async function goodLoader() {
  try {
    const data = await fetchData();
    return data;
  } catch (error) {
    console.error(error); // Log for debugging
    throw new Response('Failed to load data', { status: 500 });
  }
}
```

