## Redux State Management

### Top-Level Approach
```javascript
// src/layouts/RootLayout.jsx
function RootLayout() {
  // Pull data at top level
  const userData = useSelector(selectUserData);
  const products = useSelector(selectProducts);
  
  return (
    <div>
      <Header userData={userData} />
      <Outlet context={{ userData, products }} />
      <Footer />
    </div>
  );
}

// Child component
function ProductList() {
  // Get data from outlet context
  const { products } = useOutletContext();
  
  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```
**Pros:**
- Single source of truth
- Predictable data flow
- Easier to debug
- Better performance (fewer selector calls)

**Cons:**
- More complex prop drilling
- Less component independence
- Harder to track what data is needed where

### Component-Level Approach
```javascript
// Each component pulls what it needs
function Header() {
  const userData = useSelector(selectUserData);
  return <header>{userData.name}</header>;
}

function ProductList() {
  const products = useSelector(selectProducts);
  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```
**Pros:**
- Components are more independent
- Clearer data dependencies
- Easier to move components around
- More flexible

**Cons:**
- Multiple selector calls
- Potential performance impact
- Harder to track state updates

## API Calls

### Top-Level Loader Approach
```javascript
// src/routes/index.js
{
  path: "/",
  element: <RootLayout />,
  loader: async () => {
    const [userData, products] = await Promise.all([
      fetch('/api/user').then(r => r.json()),
      fetch('/api/products').then(r => r.json())
    ]);
    return { userData, products };
  },
  children: [
    {
      path: "products",
      element: <ProductList />,
      // No loader needed here, data already loaded
    }
  ]
}

// Components use loader data
function ProductList() {
  const { products } = useLoaderData();
  return <div>{/* Use products */}</div>;
}
```
**Pros:**
- Data is ready before render
- Fewer API calls
- Better loading states
- Parallel data loading

**Cons:**
- Might load unnecessary data
- Less granular loading states
- Longer initial load time

### Component-Level Loader Approach
```javascript
// Each route handles its own data
{
  path: "/",
  element: <RootLayout />,
  children: [
    {
      path: "products",
      element: <ProductList />,
      loader: async () => {
        const products = await fetch('/api/products')
          .then(r => r.json());
        return { products };
      }
    }
  ]
}
```
**Pros:**
- Only load needed data
- More granular loading states
- Faster initial load
- Clearer data dependencies

**Cons:**
- Multiple API calls
- Potential waterfall effect
- More complex error handling

## Recommended Approach

### For Redux:
```javascript
// Use a hybrid approach
function RootLayout() {
  // Pull only global state at top level
  const user = useSelector(selectUser);
  const theme = useSelector(selectTheme);
  
  return (
    <div className={theme}>
      <Header user={user} />
      <Outlet />
    </div>
  );
}

// Components pull specific data they need
function ProductList() {
  // Pull specific product data in component
  const products = useSelector(selectProducts);
  const productFilters = useSelector(selectProductFilters);
  
  return (
    <div>
      <ProductFilters filters={productFilters} />
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

### For API Calls:
```javascript
// Use route-level loaders for main data
{
  path: "products",
  element: <ProductsLayout />,
  // Load main product data
  loader: async () => {
    const products = await fetch('/api/products')
      .then(r => r.json());
    return { products };
  },
  children: [
    {
      path: ":id",
      element: <ProductDetail />,
      // Load specific product details
      loader: async ({ params }) => {
        const details = await fetch(`/api/products/${params.id}/details`)
          .then(r => r.json());
        return { details };
      }
    }
  ]
}
```

### Best Practices:
1. **For Redux:**
   - Pull global state (user, theme, etc.) at top level
   - Pull feature-specific state in feature components
   - Use memoized selectors for performance
   - Consider using RTK Query for API calls

2. **For API Calls:**
   - Use route loaders for main data requirements
   - Load granular data at component level when needed
   - Implement proper loading states
   - Use parallel data loading when possible

3. **General Guidelines:**
   - Keep components focused and independent
   - Avoid prop drilling more than 2-3 levels
   - Use context for truly global state
   - Implement proper error boundaries
   - Consider code splitting for large features

This hybrid approach gives you the best of both worlds: good performance and maintainable code.