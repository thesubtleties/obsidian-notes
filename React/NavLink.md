1. Basic Usage:
```jsx
<NavLink to="/about">About</NavLink>
```

2. Styling Active Links:
```jsx
<NavLink 
  to="/about"
  style={({ isActive }) => isActive ? {color: 'red'} : undefined}
>
  About
</NavLink>
```

3. Using className for Active Links:
```jsx
<NavLink 
  to="/about"
  className={({ isActive }) => isActive ? 'active-link' : ''}
>
  About
</NavLink>
```

4. End Prop (Exact Matching):
```jsx
<NavLink to="/" end>Home</NavLink>
```

5. Custom Active Checking:
```jsx
<NavLink
  to="/messages"
  isActive={(match, location) => location.pathname.startsWith('/messages')}
>
  Messages
</NavLink>
```

6. Passing Props to Rendered Component:
```jsx
<NavLink
  to="/profile"
  children={({ isActive }) => (
    <span className={isActive ? 'active' : ''}>Profile</span>
  )}
/>
```

7. Using with 'reloadDocument':
```jsx
<NavLink to="/docs" reloadDocument>Documentation</NavLink>
```

8. Handling onClick:
```jsx
<NavLink
  to="/dashboard"
  onClick={(event) => {
    if (!confirm('Are you sure?')) {
      event.preventDefault();
    }
  }}
>
  Dashboard
</NavLink>
```

Watch out for these gotchas:

1. The 'end' prop: Without it, "/about" would be considered active for "/about/team". Use it on your home route.

2. Nested Routes: NavLink considers a link active if its 'to' path is a prefix of the current URL. This can lead to unexpected behavior with nested routes.

3. Case Sensitivity: By default, matching is case-sensitive. You can change this with the 'caseSensitive' prop.

4. Active Styling: If you're using a CSS-in-JS solution, remember that the style prop takes a function, not an object.

Pro tip: You can create a custom NavLink wrapper to standardize your link styling across the app:

```jsx
const CustomNavLink = ({ children, to, ...props }) => {
  return (
    <NavLink
      to={to}
      className={({ isActive }) => 
        isActive ? 'active-link' : 'normal-link'
      }
      {...props}
    >
      {children}
    </NavLink>
  );
};
```

Remember, NavLink is all about enhancing the user experience by providing visual feedback on the current location. Use it wisely to guide users through your app's navigation structure.