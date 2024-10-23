useImperativeHandle customizes the instance value that is exposed to parent components when using ref. For more information: [[React Hooks]]

## Basic Syntax

```javascript
useImperativeHandle(ref, createHandle, [dependencies])
```
This hook takes three arguments: the ref to be modified, a function that returns an object with the methods to be exposed, and an optional array of dependencies.

## Common Use Case

```javascript
const ChildComponent = forwardRef((props, ref) => {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));

  return <input ref={inputRef} />;
});

// Parent component
function ParentComponent() {
  const childRef = useRef();

  const handleClick = () => {
    childRef.current.focus();
  };

  return (
    <>
      <ChildComponent ref={childRef} />
      <button onClick={handleClick}>Focus Input</button>
    </>
  );
}
```
In this example, the ChildComponent exposes a `focus` method to its parent. The parent can then call this method imperatively using a ref, allowing it to focus the input in the child component when a button is clicked.

## Advanced Patterns

### 1. Exposing Multiple Methods
```javascript
useImperativeHandle(ref, () => ({
  focus: () => { /* ... */ },
  reset: () => { /* ... */ },
  validate: () => { /* ... */ }
}), [/* dependencies */]);
```
This pattern shows how to expose multiple methods to the parent component. Each method in the returned object becomes available on the ref in the parent component.

### 2. Using with TypeScript
```typescript
interface ChildHandle {
  focus: () => void;
}

const ChildComponent = forwardRef<ChildHandle, Props>((props, ref) => {
  useImperativeHandle(ref, () => ({
    focus: () => { /* ... */ }
  }));
  // ...
});
```
This example demonstrates how to use useImperativeHandle with TypeScript. The ChildHandle interface defines the shape of the exposed methods, providing type safety when using the ref in the parent component.

## Examples

### Custom Input Component
```javascript
const CustomInput = forwardRef((props, ref) => {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    },
    getValue: () => {
      return inputRef.current.value;
    },
    setValue: (newValue) => {
      inputRef.current.value = newValue;
    }
  }));

  return <input ref={inputRef} {...props} />;
});
```
This custom input component exposes methods to focus the input, get its current value, and set a new value. This allows parent components to have more control over the input while keeping the implementation details encapsulated.

### Animation Control
```javascript
const AnimatedComponent = forwardRef((props, ref) => {
  const [isAnimating, setIsAnimating] = useState(false);

  useImperativeHandle(ref, () => ({
    startAnimation: () => {
      setIsAnimating(true);
    },
    stopAnimation: () => {
      setIsAnimating(false);
    },
    isAnimating: () => isAnimating
  }), [isAnimating]);

  return (
    <div className={isAnimating ? 'animate' : ''}>
      {props.children}
    </div>
  );
});
```
This example shows how to use useImperativeHandle to control animations. The component exposes methods to start and stop an animation, as well as check its current state. The parent component can use these methods to control the animation imperatively.

Remember, useImperativeHandle is a powerful tool but should be used judiciously. It's most appropriate for scenarios where you need to expose imperative methods to parent components, such as managing focus, controlling animations, or exposing complex internal state that can't be easily managed through props.