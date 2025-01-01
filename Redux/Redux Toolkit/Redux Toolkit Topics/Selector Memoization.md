## Basic Syntax
```javascript
import { createSelector } from '@reduxjs/toolkit';

// Basic selector directly accesses state
const simpleSelector = (state) => state.something;

// Memoized selector takes input selectors and a result function
const memoizedSelector = createSelector(
  // Array of input selectors - these run first
  [inputSelector1, inputSelector2],
  // Result function - only runs if inputs change
  (input1, input2) => {
    // Computed result based on inputs
    return someComputedValue;
  }
);
```
**Explanation**: The basic pattern shows the difference between a simple state accessor and a memoized selector. The memoized version only recalculates when its inputs change, saving performance.

## Common Patterns

### 1. Simple Selectors (No Memoization Needed)
```javascript
// Direct state access - no computation needed
const selectUser = (state) => state.user;
const selectIsLoading = (state) => state.isLoading;
const selectErrors = (state) => state.errors;

// These are fine without memoization because:
// 1. They're just accessing state directly
// 2. No computation involved
// 3. Very lightweight operations
```
**Explanation**: Simple selectors that just access state properties don't need memoization. They're already fast and adding memoization would add unnecessary complexity.

### 2. Single Input Memoization
```javascript
// Single input selector with computation
const selectUserDetails = createSelector(
  // Input selector array with single selector
  [selectUser],
  // Transform function creates new object from user data
  (user) => ({
    // Only runs this computation if user data changes
    fullName: `${user?.firstName} ${user?.lastName}`,
    email: user?.email,
    // Optional chaining (?.) handles null/undefined cases
  })
);
```
**Explanation**: When you need to transform data from a single source, this pattern prevents unnecessary recalculations when other parts of state change.

### 3. Multiple Input Memoization
```javascript
// Combining multiple pieces of state with computation
const selectProjectDetails = createSelector(
  // Multiple input selectors
  [
    selectProject,  // First input
    selectTasks,    // Second input
    selectUser      // Third input
  ],
  // Result function receives values in same order as input selectors
  (project, tasks, user) => ({
    // Computed properties combining multiple inputs
    projectName: project.name,
    taskCount: tasks.length,
    isOwner: project.ownerId === user.id
  })
);
```
**Explanation**: When combining multiple pieces of state, memoization prevents recalculation unless one of the input values changes.

### 4. Parameterized Selectors
```javascript
// Factory function that creates a memoized selector
const selectProjectById = (projectId) => 
  createSelector(
    // Input selector
    [selectProjects],
    // Result function uses both state data and parameter
    (projects) => {
      // Uses closure to access projectId parameter
      return projects[projectId];
    }
  );

// Usage in component
// Each unique projectId gets its own memoized selector
const project = useSelector(selectProjectById(id));
```
**Explanation**: When you need to select data based on parameters, this pattern creates unique memoized selectors for each parameter value.

## Advanced Patterns

### 1. Nested Selectors
```javascript
// First level of filtering
const selectFilteredProjects = createSelector(
  [selectProjects, selectFilters],
  // First transformation
  (projects, filters) => projects.filter(p => 
    p.status === filters.status
  )
);

// Second level using results from first selector
const selectSortedFilteredProjects = createSelector(
  // Uses output from previous selector as input
  [selectFilteredProjects, selectSortCriteria],
  // Second transformation
  (filteredProjects, sortBy) => [...filteredProjects].sort(/* ... */)
);
```
**Explanation**: Selectors can be composed, with each level maintaining its own memoization. Changes are only propagated through the chain when necessary.

### 2. Re-reselect Pattern
```javascript
// Import specialized selector creator
import createCachedSelector from 're-reselect';

// Create cached selector with parameter
const selectUserProjects = createCachedSelector(
  // Input selectors (including parameter selector)
  [
    selectProjects,
    (_, userId) => userId  // Parameter selector
  ],
  // Result function
  (projects, userId) => projects.filter(p => p.userId === userId)
)(
  // Cache key generator
  (_, userId) => userId // Uses userId as cache key
);
```
**Explanation**: Re-reselect adds parameter-based caching to memoized selectors, perfect for lists or filtered data.

### 3. Error Handling
```javascript
// Safe selector with error handling
const selectSafeUserData = createSelector(
  [selectUser],
  (user) => {
    try {
      // Attempt to transform data
      return {
        name: user?.name || 'Guest',
        settings: user?.settings || defaultSettings
      };
    } catch (error) {
      // Fallback if transformation fails
      console.error('Error in selector:', error);
      return defaultUserData;
    }
  }
);
```
**Explanation**: Adding error handling ensures selectors don't break the application when data is unexpected or malformed.

## Performance Tips

### 1. Avoid Inline Objects
```javascript
// ❌ Bad - Creates new reference every time
const selectBadConfig = createSelector(
  [selectUser],
  (user) => ({ name: user.name }) // New object each render
);

// ✅ Good - Returns primitive or stable reference
const selectGoodConfig = createSelector(
  [selectUser],
  (user) => user.name // Returns primitive value
);
```
**Explanation**: Creating new objects in selectors defeats memoization because the reference changes every time. Return primitives or stable references instead.

### 2. Memoization Dependencies
```javascript
// Optimized dependencies
const selectDashboardData = createSelector(
  [
    // ✅ Only select required data
    selectRelevantDataOnly,
    
    // ❌ Avoid selecting entire state tree
    // selectEntireStore
  ],
  (relevantData) => processData(relevantData)
);
```
**Explanation**: Minimizing dependencies reduces unnecessary recalculations. Only select the data you actually need.

## Testing Example
```javascript
describe('selectUserDetails', () => {
  // Setup test state
  const mockState = {
    user: {
      firstName: 'John',
      lastName: 'Doe'
    }
  };

  it('should select user details', () => {
    // Test selector output
    const result = selectUserDetails(mockState);
    
    // Verify expected transformation
    expect(result).toEqual({
      fullName: 'John Doe',
      email: undefined
    });
  });

  it('should memoize results', () => {
    // First call
    const result1 = selectUserDetails(mockState);
    // Second call with same state
    const result2 = selectUserDetails(mockState);
    
    // Verify same reference
    expect(result1).toBe(result2);
  });
});
```
**Explanation**: Testing should verify both the correct transformation of data and proper memoization behavior.

Remember: Memoization is a trade-off between computation time and memory usage. Use it when the benefits of preventing expensive recomputations outweigh the memory overhead of caching results.