## File Structure:
```
src/
├── components/
│   ├── Counter.js
│   ├── LifecycleExample.js
│   ├── ButtonComponent.js
│   ├── Greeting.js
│   ├── MyInput.js
│   └── ThemedButton.js
├── context/
│   └── ThemeContext.js
└── App.js
```

## Counter.js
```jsx
import React, { Component } from 'react';

class Counter extends Component {
  state = { count: 0 };

  increment = () => {
    this.setState(prevState => ({ count: prevState.count + 1 }));
  }

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>Increment</button>
      </div>
    );
  }
}

export default Counter;
```

## LifecycleExample.js
```jsx
import React, { Component } from 'react';

class LifecycleExample extends Component {
  componentDidMount() {
    console.log('Component mounted');
  }

  componentDidUpdate(prevProps, prevState) {
    console.log('Component updated');
  }

  componentWillUnmount() {
    console.log('Component will unmount');
  }

  render() {
    return <div>Lifecycle Example</div>;
  }
}

export default LifecycleExample;
```

## ButtonComponent.js
```jsx
import React, { Component } from 'react';

class ButtonComponent extends Component {
  handleClick = () => {
    console.log('Button clicked');
  }

  render() {
    return <button onClick={this.handleClick}>Click me</button>;
  }
}

export default ButtonComponent;
```

## Greeting.js
```jsx
import React, { Component } from 'react';

class Greeting extends Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}

export default Greeting;
```

## MyInput.js
```jsx
import React, { Component } from 'react';

class MyInput extends Component {
  constructor(props) {
    super(props);
    this.inputRef = React.createRef();
  }

  focusInput = () => {
    this.inputRef.current.focus();
  }

  render() {
    return (
      <div>
        <input type="text" ref={this.inputRef} />
        <button onClick={this.focusInput}>Focus Input</button>
      </div>
    );
  }
}

export default MyInput;
```

## ThemedButton.js
```jsx
import React, { Component } from 'react';
import { ThemeContext } from '../context/ThemeContext';

class ThemedButton extends Component {
  static contextType = ThemeContext;

  render() {
    const { theme, toggleTheme } = this.context;
    return (
      <button
        onClick={toggleTheme}
        style={{
          backgroundColor: theme === 'light' ? '#fff' : '#000',
          color: theme === 'light' ? '#000' : '#fff'
        }}
      >
        Toggle Theme
      </button>
    );
  }
}

export default ThemedButton;
```

## ThemeContext.js
```jsx
import React, { Component } from 'react';

export const ThemeContext = React.createContext();

export class ThemeProvider extends Component {
  state = { theme: 'light' };

  toggleTheme = () => {
    this.setState(prevState => ({
      theme: prevState.theme === 'light' ? 'dark' : 'light'
    }));
  }

  render() {
    return (
      <ThemeContext.Provider value={{
        theme: this.state.theme,
        toggleTheme: this.toggleTheme
      }}>
        {this.props.children}
      </ThemeContext.Provider>
    );
  }
}
```

## App.js
```jsx
import React from 'react';
import { ThemeProvider } from './context/ThemeContext';
import Counter from './components/Counter';
import LifecycleExample from './components/LifecycleExample';
import ButtonComponent from './components/ButtonComponent';
import Greeting from './components/Greeting';
import MyInput from './components/MyInput';
import ThemedButton from './components/ThemedButton';

function App() {
  return (
    <ThemeProvider>
      <div>
        <Counter />
        <LifecycleExample />
        <ButtonComponent />
        <Greeting name="John" />
        <MyInput />
        <ThemedButton />
      </div>
    </ThemeProvider>
  );
}

export default App;
```

## Key Differences from Functional Components:

1. Syntax: Class components use ES6 class syntax, while functional components use function syntax.

2. State Management: 
   - Class: Uses `this.state` and `this.setState()`
   - Functional: Uses the `useState` hook

3. Lifecycle Methods:
   - Class: Has built-in lifecycle methods (e.g., `componentDidMount`, `componentDidUpdate`)
   - Functional: Uses the `useEffect` hook to handle side effects

4. Handling `this`:
   - Class: Requires binding `this` or using arrow functions for methods
   - Functional: Doesn't use `this`

5. Props Access:
   - Class: Accessed via `this.props`
   - Functional: Received as function arguments

6. Context:
   - Class: Uses `static contextType` or `<Context.Consumer>`
   - Functional: Uses the `useContext` hook

7. Refs:
   - Class: Uses `createRef()` or callback refs
   - Functional: Uses the `useRef` hook

8. Performance:
   - Class: Slightly slower and more memory-intensive
   - Functional: Generally faster and more memory-efficient

9. Code Reuse:
   - Class: Uses higher-order components or render props for code reuse
   - Functional: Can use custom hooks for better code reuse

10. Simplicity:
- Class: More boilerplate code, especially for simple components
- Functional: Generally more concise and easier to understand

Remember: While class components are still supported in React, functional components with hooks are now the recommended approach for writing React components due to their simplicity and better performance characteristics.