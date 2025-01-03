## Basic Syntax

```javascript
ReactDOM.createPortal(child, container)
```

- `child`: React element to be rendered
- `container`: DOM element to render into

## Key Concepts

1. Renders content outside the parent component's DOM hierarchy
2. Maintains React context and event bubbling
3. Useful for modals, tooltips, and floating menus

## Common Use Cases

- Modals
- Tooltips
- Popups
- Floating menus
- Notifications

## Example Implementation

```jsx
function Modal({ children, isOpen }) {
  if (!isOpen) return null;
  
  return ReactDOM.createPortal(
    <div className="modal">
      {children}
    </div>,
    document.body
  );
}
```

## Event Bubbling

- Events from portals bubble up through React tree, not DOM tree
- Parent components can catch events from portals

## Context

- Portals can access React context from parent tree
- Context providers above portal still work for portal content

## Lifecycle

- Portals follow normal React lifecycle
- Unmounting a component unmounts its portals

## Accessibility

- Ensure proper focus management for modals
- Use appropriate ARIA attributes

## Best Practices

1. Keep portal content in the same React tree for state management
2. Use refs to manage focus when necessary
3. Clean up portal content on component unmount
4. Consider using a portal for any content that needs to "break out" of layout constraints

## Pros

- Solves z-index and overflow: hidden issues
- Maintains React's synthetic event system
- Keeps related code together

## Cons

- Can complicate server-side rendering
- May need additional focus management
- Can be overused (not everything needs to be a portal)

## Example with Hooks

```jsx
function ModalWithHooks() {
  const [isOpen, setIsOpen] = useState(false);
  const modalRoot = useRef(document.getElementById('modal-root'));

  useEffect(() => {
    const el = document.createElement('div');
    modalRoot.current.appendChild(el);
    return () => modalRoot.current.removeChild(el);
  }, []);

  return isOpen ? ReactDOM.createPortal(
    <div>Modal Content</div>,
    modalRoot.current
  ) : null;
}
```

## Testing

- Use `ReactDOM.createPortal.mock` for testing portals
- Ensure to test both the triggering component and portal content

Remember, while portals are powerful, they should be used judiciously. They're great for breaking out of DOM hierarchies, but overuse can lead to confusing component structures. Always consider whether a portal is truly necessary for your use case.