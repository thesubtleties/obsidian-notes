1. Basic Usage:
```jsx
import { Outlet } from 'react-router-dom';

function Layout() {
  return (
    <div>
      <Nav />
      <Outlet />
      <Footer />
    </div>
  );
}
```
Outlet renders the matched child route component.

2. Nested Routing Setup:
```jsx
<Routes>
  <Route path="/" element={<Layout />}>
    <Route index element={<Home />} />
    <Route path="about" element={<About />} />
    <Route path="users/*" element={<Users />} />
  </Route>
</Routes>
```
Layout component will render, with child routes appearing where Outlet is placed.

3. Outlet Context:
```jsx
<Outlet context={{ user: currentUser }} />

// In child component:
import { useOutletContext } from 'react-router-dom';

function ChildComponent() {
  const { user } = useOutletContext();
  return <h1>Welcome, {user.name}!</h1>;
}
```
Pass data to child routes via Outlet context.

4. Conditional Rendering with Outlet:
```jsx
function Layout() {
  const [isLoading, setIsLoading] = useState(true);

  return (
    <div>
      <Nav />
      {isLoading ? <LoadingSpinner /> : <Outlet />}
      <Footer />
    </div>
  );
}
```
Conditionally render Outlet or other components.

5. Multiple Outlets:
```jsx
function DashboardLayout() {
  return (
    <div>
      <Sidebar>
        <Outlet name="sidebar" />
      </Sidebar>
      <MainContent>
        <Outlet />
      </MainContent>
    </div>
  );
}

// In route configuration:
<Route path="dashboard" element={<DashboardLayout />}>
  <Route path="users" element={<Users />}>
    <Route path=":id" element={<UserDetails />} />
  </Route>
  <Route path="settings" element={<Settings />} outlet="sidebar" />
</Route>
```
Use named outlets for more complex layouts.

6. Outlet with Error Boundaries:
```jsx
import { Outlet, useRouteError } from 'react-router-dom';

function Layout() {
  return (
    <div>
      <Nav />
      <ErrorBoundary fallback={<ErrorPage />}>
        <Outlet />
      </ErrorBoundary>
      <Footer />
    </div>
  );
}

function ErrorPage() {
  const error = useRouteError();
  return <div>Oops! {error.message}</div>;
}
```
Wrap Outlet in an error boundary to handle errors in child routes.

7. Outlet with Suspense:
```jsx
import { Suspense } from 'react';
import { Outlet } from 'react-router-dom';

function Layout() {
  return (
    <div>
      <Nav />
      <Suspense fallback={<LoadingSpinner />}>
        <Outlet />
      </Suspense>
      <Footer />
    </div>
  );
}
```
Use Suspense for loading states in child routes.

Remember:
- Outlet is key for creating nested layouts in React Router.
- It allows for consistent UI elements across routes while changing only the main content area.
- Use useOutletContext to pass data to child routes if needed.
- Outlet can be combined with other React features like Suspense and Error Boundaries for more robust applications.