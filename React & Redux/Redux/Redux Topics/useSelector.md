useSelector is a hook that allows you to extract data from the Redux store state in your functional components. For more information: [[Selectors]]

## Basic Syntax

```javascript
const result = useSelector(selector)
```

## Key Concepts

1. Runs whenever the Redux store state changes
2. Performs a reference equality check (===) on the returned value
3. Re-renders the component only if the selected value has changed

## Common Use Cases

### 1. Selecting a Single Value
```javascript
const count = useSelector(state => state.counter.value)
```

### 2. Selecting Multiple Values
```javascript
const { user, posts } = useSelector(state => ({
  user: state.user,
  posts: state.posts
}))
```

### 3. Using Reselect for Memoized Selectors
```javascript
import { createSelector } from 'reselect'

const selectUser = state => state.user
const selectPosts = state => state.posts

const selectUserPosts = createSelector(
  [selectUser, selectPosts],
  (user, posts) => posts.filter(post => post.userId === user.id)
)

const UserPosts = () => {
  const userPosts = useSelector(selectUserPosts)
  // ...
}
```

## Advanced Patterns

### 1. Equality Function
```javascript
import { shallowEqual } from 'react-redux'

const result = useSelector(selector, shallowEqual)
```

### 2. Accessing Props in Selectors
```javascript
const TodoItem = ({ id }) => {
  const todo = useSelector(state => state.todos.find(todo => todo.id === id))
  // ...
}
```

### 3. Using with TypeScript
```typescript
interface RootState {
  user: UserState
  posts: PostState
}

const user = useSelector((state: RootState) => state.user)
```

## Best Practices

1. Keep selectors simple and focused
2. Use memoized selectors for expensive computations
3. Avoid creating new object references in selectors unless necessary
4. Consider using `shallowEqual` for selecting multiple values

## Common Mistakes

1. Selecting too much data, causing unnecessary re-renders
2. Creating new objects or arrays in selectors without memoization
3. Not handling potential undefined states
4. Overusing useSelector when props would suffice

## Tips

1. Use `useSelector` multiple times in a component for different pieces of state
2. Combine with `useMemo` for complex derived data
3. Use with `useDispatch` for a complete Redux integration
4. Consider creating custom hooks that use `useSelector` for reusable logic

## Examples

### Selecting Derived Data
```javascript
const getTotalPrice = state => state.cart.items.reduce((total, item) => total + item.price, 0)

const CartTotal = () => {
  const totalPrice = useSelector(getTotalPrice)
  return <div>Total: ${totalPrice}</div>
}
```

### Conditional Selecting
```javascript
const getVisibleTodos = state => {
  switch (state.visibilityFilter) {
    case 'SHOW_COMPLETED':
      return state.todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return state.todos.filter(t => !t.completed)
    default:
      return state.todos
  }
}

const VisibleTodoList = () => {
  const todos = useSelector(getVisibleTodos)
  // ...
}
```

### Using with useDispatch
```javascript
import { useSelector, useDispatch } from 'react-redux'

const Counter = () => {
  const count = useSelector(state => state.counter.value)
  const dispatch = useDispatch()

  return (
    <div>
      <span>{count}</span>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>Increment</button>
    </div>
  )
}
```

Remember, `useSelector` is a powerful tool for connecting your React components to your Redux store. Use it wisely to keep your components efficient and your state management clean.