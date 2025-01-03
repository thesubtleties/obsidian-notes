## Basic Layout Structure

### File Location
```text
src/layouts/RootLayout.jsx
src/layouts/FeatureLayout.jsx
src/routes/index.js
```

### Basic Layout Example
```javascript
// src/layouts/RootLayout.jsx
import { Outlet, useNavigation } from 'react-router-dom';

function RootLayout() {
  const navigation = useNavigation();
  
  return (
    <div className="app-container">
      <header>
        <nav>{/* Navigation components */}</nav>
      </header>
      
      <main className={navigation.state === "loading" ? "loading" : ""}>
        <Outlet /> {/* Child routes render here */}
      </main>
      
      <footer>{/* Footer content */}</footer>
    </div>
  );
}
```
The root layout provides the main structure of your application. The `Outlet` component is a placeholder where child routes will render their content. The `useNavigation` hook helps manage loading states during navigation.

## Nested Layouts

### File Location
```text
src/layouts/DashboardLayout.jsx
```

```javascript
// src/layouts/DashboardLayout.jsx
import { Outlet, NavLink } from 'react-router-dom';

function DashboardLayout() {
  return (
    <div className="dashboard-container">
      <aside className="sidebar">
        <nav>
          <NavLink to="profile">Profile</NavLink>
          <NavLink to="settings">Settings</NavLink>
          <NavLink to="notifications">Notifications</NavLink>
        </nav>
      </aside>
      
      <section className="content">
        <Outlet />
      </section>
    </div>
  );
}

// src/routes/index.js
const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    children: [
      {
        path: "dashboard",
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
      }
    ]
  }
]);
```
Nested layouts allow you to create complex UI hierarchies. Each layout can have its own navigation and structure while maintaining the parent layout's context.

## Context-Aware Layouts

```javascript
// src/layouts/FeatureLayout.jsx
import { Outlet, useOutletContext, useNavigation } from 'react-router-dom';

function FeatureLayout() {
  const navigation = useNavigation();
  const [theme, setTheme] = useState('light');
  
  return (
    <div className={`feature-container ${theme}`}>
      <aside>
        <ThemeSwitcher onChange={setTheme} />
      </aside>
      
      <section>
        <Outlet context={{ theme, setTheme }} />
      </section>
    </div>
  );
}

// In child component
function FeatureComponent() {
  const { theme, setTheme } = useOutletContext();
  
  return (
    <div>
      Current theme: {theme}
      <button onClick={() => setTheme('dark')}>
        Toggle Theme
      </button>
    </div>
  );
}
```
Layouts can pass context to their child routes using `Outlet context`. This is useful for sharing state or functions across related routes while keeping them encapsulated within the layout.

## Dynamic Layouts

```javascript
// src/layouts/DynamicLayout.jsx
function DynamicLayout() {
  const { user } = useAuth();
  const navigation = useNavigation();

  return (
    <div className="dynamic-layout">
      {user.isAdmin ? (
        <AdminSidebar />
      ) : (
        <UserSidebar />
      )}
      
      <main>
        {navigation.state === "loading" ? (
          <LoadingSpinner />
        ) : (
          <Outlet />
        )}
      </main>
    </div>
  );
}

// Usage in routes
{
  path: "dashboard",
  element: <DynamicLayout />,
  loader: async () => {
    const user = await getUser();
    return { user };
  },
  children: [
    // Different routes for admin/user
  ]
}
```
Dynamic layouts can adapt their structure based on conditions like user roles or preferences. This pattern allows for flexible UI arrangements while maintaining consistent navigation.

## Layout with Data Dependencies

```javascript
// src/layouts/DataAwareLayout.jsx
function DataAwareLayout() {
  const { organizations } = useLoaderData();
  const navigation = useNavigation();
  
  return (
    <div className="org-layout">
      <aside>
        <OrganizationList organizations={organizations} />
      </aside>
      
      <main className={navigation.state === "loading" ? "loading" : ""}>
        <Outlet context={{ organizations }} />
      </main>
    </div>
  );
}

// In routes configuration
{
  path: "organizations",
  element: <DataAwareLayout />,
  loader: async () => {
    const organizations = await fetchOrganizations();
    return { organizations };
  },
  children: [
    {
      path: ":orgId",
      element: <OrganizationDetail />,
      loader: async ({ params }) => {
        return fetchOrgDetails(params.orgId);
      }
    }
  ]
}
```
Layouts can load and provide data to their child routes. This is useful when multiple child routes need access to the same data, reducing redundant data fetching.

## Error Boundary Layouts

```javascript
// src/layouts/ErrorBoundaryLayout.jsx
function ErrorBoundaryLayout() {
  return (
    <div className="error-aware-layout">
      <ErrorBoundary fallback={<ErrorMessage />}>
        <Outlet />
      </ErrorBoundary>
    </div>
  );
}

// Error component
function ErrorMessage() {
  const error = useRouteError();
  
  return (
    <div className="error-container">
      <h2>Something went wrong</h2>
      <p>{error.message}</p>
    </div>
  );
}
```
Layouts can include error boundaries to gracefully handle errors in their child routes while maintaining the overall application structure.

## Common Patterns and Best Practices

### Loading States
```javascript
function LoadingAwareLayout() {
  const navigation = useNavigation();
  const [isTransitioning, startTransition] = useTransition();
  
  return (
    <div>
      {/* Global loading indicator */}
      {navigation.state === "loading" && (
        <LoadingBar />
      )}
      
      {/* Content with loading class */}
      <main className={
        navigation.state === "loading" ? "content loading" : "content"
      }>
        <Outlet />
      </main>
    </div>
  );
}
```
This pattern shows how to handle loading states gracefully in layouts, providing visual feedback during navigation.

### Layout Guards
```javascript
function ProtectedLayout() {
  const { user } = useAuth();
  
  if (!user) {
    return <Navigate to="/login" replace />;
  }
  
  return (
    <div className="protected-layout">
      <UserNav user={user} />
      <Outlet />
    </div>
  );
}
```
Layout guards protect sections of your application, ensuring only authorized users can access certain routes while maintaining the layout structure.

