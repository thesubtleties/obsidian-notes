## 1. Basic useSelector (The Standard Way)
```javascript
// Most basic form - directly accessing state
const value = useSelector(state => state.users.currentUser)

// Accessing and transforming in useSelector
const userName = useSelector(state => 
  state.users.currentUser?.name?.toUpperCase() // Optional chaining for safety
)

// Getting multiple values with object destructuring
const { name, email } = useSelector(state => ({
  name: state.users.currentUser?.name,
  email: state.users.currentUser?.email
}))
```

Think of useSelector as your direct line to the Redux store. It's like reaching into a filing cabinet and pulling out exactly what you need. In its simplest form, you're just grabbing a piece of data. When you add transformations (like toUpperCase()), you're saying "grab this data AND do something to it before giving it to me." When destructuring multiple values, you're essentially making a shopping list of different pieces of data you want to grab all at once.

The catch? Every time anything in Redux changes, these selectors run again. It's like checking your filing cabinet every time anyone in the office touches any paper, even if your specific files haven't changed.

## 2. Introduction to createSelector (The Memoized Way)
```javascript
// In your Redux files (e.g., selectors.js)
export const selectUserData = createSelector(
  // Input selectors - specify which pieces of state to watch
  [
    state => state.users.currentUser,    // First input
    state => state.preferences           // Second input
  ],
  // Result function - only runs when inputs change
  (user, preferences) => ({
    name: user?.name,
    theme: preferences?.theme,
    displayName: `${user?.name} (${preferences?.language})`
  })
)

// In your component
const userData = useSelector(selectUserData)
```

This is where things get smarter. createSelector is like having an efficient assistant who remembers the last time they checked those specific files and only does the work again if those specific files have changed. The first part (input selectors) is your "watch list" - what files to keep an eye on. The second part (result function) is your "instruction manual" for what to do with those files when they change.

It's like saying "Only when either the user info OR preferences change, create this neat package of information. Otherwise, give me the same package you made last time."

## 3. Adding Parameters (Dynamic Selection)
```javascript
// In selectors.js
export const selectUserById = createSelector(
  [
    // First argument is always state
    state => state.users.allUsers,
    // Second argument comes from component
    (state, userId) => userId  
  ],
  // Result function receives values in same order
  (allUsers, userId) => allUsers[userId] || null
)

// In component
const user = useSelector(state => selectUserById(state, props.userId))
```

Now we're adding flexibility to our smart system. This is like having an assistant who not only remembers what they checked last time but can also handle specific requests. Instead of just saying "get me the user files," you're saying "get me the file for user #123." The selector remembers the last time it looked for that specific user and won't do the work again unless either the user data has changed or you ask for a different user.

This is particularly powerful because it combines the efficiency of memoization with the flexibility of dynamic queries.

## 4. Practical Examples

### Example 1: Filtered List
```javascript
// Selector for filtered, sorted todo items
export const selectFilteredTodos = createSelector(
  [
    state => state.todos.items,          // All todos
    state => state.filters.status,       // Active filter
    state => state.filters.searchTerm    // Search term
  ],
  (todos, status, searchTerm) => {
    return todos
      // First filter by status
      .filter(todo => status === 'all' || todo.status === status)
      // Then filter by search term
      .filter(todo => 
        searchTerm ? todo.title.toLowerCase().includes(searchTerm.toLowerCase()) : true
      )
      // Sort by date
      .sort((a, b) => new Date(b.date) - new Date(a.date))
  }
)

// In component
const filteredTodos = useSelector(selectFilteredTodos)
```

This is like having a smart filing system that automatically organizes your todo list. Instead of manually sorting through papers every time you want to find "active tasks containing the word 'urgent'", this system remembers how it organized things last time and only reorganizes when either the todos themselves, the status filter, or the search term changes. It's particularly useful for lists that need multiple types of filtering and sorting - imagine doing this work every time your component renders!

### Example 2: Dashboard Data
```javascript
// Complex data shaping for dashboard
export const selectDashboardStats = createSelector(
  [
    state => state.projects,
    state => state.tasks,
    state => state.sprints
  ],
  (projects, tasks, sprints) => ({
    activeProjects: projects.filter(p => p.status === 'active').length,
    overdueTasks: tasks.filter(t => 
      new Date(t.dueDate) < new Date() && t.status !== 'completed'
    ).length,
    currentSprint: sprints.find(s => {
      const now = new Date()
      return new Date(s.startDate) <= now && new Date(s.endDate) >= now
    }),
    completionRate: calculateCompletionRate(tasks)
  })
)
```

Think of this as your automated dashboard compiler. Instead of recalculating all these statistics every time anything changes, this selector only recalculates when the actual project data, task data, or sprint data changes. It's like having an assistant who maintains a live dashboard but only updates the numbers when the underlying reports change, not every time someone looks at the dashboard.

## 5. Common Gotchas and Solutions

### Gotcha 1: Object Creation in Selectors
```javascript
// BAD - Creates new object every time
const badSelector = state => ({
  user: state.users.current
})

// GOOD - Returns same reference if data hasn't changed
const goodSelector = createSelector(
  [state => state.users.current],
  user => ({ user })
)
```

This is a subtle but important distinction. The bad version is like making a new copy of a document every time someone asks for it, even if nothing has changed. The good version is like checking if the document has changed - if it hasn't, you just hand over the same copy you made last time. This matters because in React, new objects (even if they contain the same data) can trigger unnecessary rerenders.

### Gotcha 2: Array Operations
```javascript
// BAD - Array methods create new arrays
const badArraySelector = state => 
  state.items.filter(item => item.active)

// GOOD - Memoized array operation
const goodArraySelector = createSelector(
  [state => state.items],
  items => items.filter(item => item.active)
)
```

Array operations are like sorting through a filing cabinet and making a new stack of selected files. The bad version makes a new stack every time, even if no files have changed. The good version checks if the files have changed since last time - if they haven't, it just points to the stack it made last time.

### Gotcha 3: Props Usage
```javascript
// BAD - New selector instance for each component
const badPropSelector = (state, props) => 
  state.items[props.id]

// GOOD - Reusable selector with props
const goodPropSelector = createSelector(
  [
    state => state.items,
    (state, props) => props.id
  ],
  (items, id) => items[id]
)
```

This is like having a system for looking up files by ID. The bad version creates a new lookup system for each person who needs to find files. The good version creates one smart lookup system that everyone can use, and it remembers what it found last time for each ID.

## 6. Best Practices Recap

The key to understanding selectors is thinking of them as your data preparation system. Basic selectors (useSelector) are like checking your files every single time. createSelector is like having a smart assistant who remembers what they checked and only does the work again when something relevant has actually changed.

Remember:
- Use basic selectors for simple lookups
- Use createSelector for any complex calculations or combinations
- Be careful about creating new objects/arrays
- Keep your selectors focused and composable
- Think about what actually needs to trigger recalculations

Think of it as the difference between repeatedly checking your email manually versus having a smart notification system that only alerts you when something important changes.