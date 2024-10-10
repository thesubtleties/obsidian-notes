## Basic Usage

```jsx
import { Link } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
    </nav>
  );
}
```

## Advanced Features

### 1. Relative Links

```jsx
<Link to="user" relative="path">User</Link>
// If current URL is /app/dashboard, this will link to /app/dashboard/user
```
Use relative links when you want the destination to be relative to the current route. This is useful in nested routes or reusable components.

### 2. Replace (Don't add to history)

```jsx
<Link to="/login" replace>Login</Link>
```
Use `replace` when you don't want to add the new location to the history stack. This is useful for login pages or other transient states.

### 3. State (Pass data to the linked route)

```jsx
<Link to="/user" state={{ from: 'navigation' }}>User</Link>
```
Use `state` to pass data to the destination route that isn't part of the URL. This is useful for passing context about where the user came from.

### 4. Custom className

```jsx
<Link to="/contact" className="nav-link">Contact</Link>
```

### 5. Prevent Navigation

```jsx
<Link
  to="/dashboard"
  onClick={(event) => {
    if (!isAuthorized) {
      event.preventDefault();
      // Handle unauthorized access
    }
  }}
>
  Dashboard
</Link>
```
Use this pattern when you need to add conditional logic before navigation occurs. It's useful for access control or showing confirmation dialogs.

## Key Points

- Use `Link` for internal navigation instead of `<a>` tags to prevent full page reloads.
- The `to` prop can be a string or an object for more complex URLs.
- `Link` renders an `<a>` tag under the hood, so it supports all standard attributes like `className`, `id`, etc.
- Use `NavLink` instead of `Link` if you need active state styling.

## Gotchas

- Don't use `Link` outside of a `Router` context.
- The `to` prop is required; omitting it will cause an error.
- For external links, use a regular `<a>` tag instead of `Link`.

## Advanced Usage: Link with URL Parameters

```jsx
<Link to={`/user/${userId}`}>User Profile</Link>
```
Use this when you need to include dynamic values in your URL. It's common for detail pages or user-specific routes.

## Link with Query Parameters

```jsx
<Link to={{
  pathname: '/search',
  search: '?query=react'
}}>
  Search React
</Link>
```
Use this syntax when you need to include query parameters in your URL. It's useful for search pages or filtering results.

Remember: The `Link` component is fundamental for navigation in React Router applications. It provides a declarative, accessible way to navigate around your app while maintaining the single-page application experience.