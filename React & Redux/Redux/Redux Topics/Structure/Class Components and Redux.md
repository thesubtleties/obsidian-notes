## File Structure:
```
src/
├── components/
│   ├── Counter.js
│   ├── LifecycleExample.js
│   ├── ButtonComponent.js
│   ├── Greeting.js
│   ├── MyInput.js
│   └── ContextExample.js
├── context/
│   └── ThemeContext.js
├── actions/
│   └── counterActions.js
├── store/
│   └── configureStore.js
└── App.js
```

## Counter.js
```jsx
import React, { Component } from 'react';
import { connect } from 'react-redux';
import { incrementAsync } from '../actions/counterActions';

class Counter extends Component {
  render() {
    return (
      <div>
        <p>Count: {this.props.count}</p>
        <button onClick={this.props.incrementAsync}>Increment Async</button>
      </div>
    );
  }
}

const mapStateToProps = state => ({
  count: state.count
});

export default connect(mapStateToProps, { incrementAsync })(Counter);
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

## ContextExample.js
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

## counterActions.js
```jsx
export const incrementAsync = () => {
  return dispatch => {
    setTimeout(() => {
      dispatch({ type: 'INCREMENT' });
    }, 1000);
  };
};
```

## configureStore.js
```jsx
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';

const initialState = { count: 0 };

function rootReducer(state = initialState, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    default:
      return state;
  }
}

const store = createStore(rootReducer, applyMiddleware(thunk));

export default store;
```

## App.js
```jsx
import React from 'react';
import { Provider } from 'react-redux';
import store from './store/configureStore';
import { ThemeProvider } from './context/ThemeContext';
import Counter from './components/Counter';
import LifecycleExample from './components/LifecycleExample';
import ButtonComponent from './components/ButtonComponent';
import Greeting from './components/Greeting';
import MyInput from './components/MyInput';
import ThemedButton from './components/ContextExample';

function App() {
  return (
    <Provider store={store}>
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
    </Provider>
  );
}

export default App;
```

Key Differences from Functional Components:

1. Syntax: Class components use ES6 class syntax.
2. State: Uses `this.state` and `this.setState()`.
3. Lifecycle Methods: Has built-in lifecycle methods.
4. Context: Uses `static contextType` or `Context.Consumer`.
5. Redux Connection: Often uses `connect` HOC instead of hooks.
6. Refs: Uses `createRef()` or callback refs.
7. This Keyword: Requires `this` to access props, state, and methods.

Remember: While class components are still supported, functional components with hooks are now the recommended approach for writing React components.