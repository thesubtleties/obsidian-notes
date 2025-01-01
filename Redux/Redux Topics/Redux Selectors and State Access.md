## Core Concepts
Selectors are functions that:
- Take the entire Redux state as their first argument
- Can access any part of the state tree
- Return derived/computed data
- Can be used to combine data from multiple slices

## Basic Selector Patterns
```javascript
// Simple selector
export const selectTasks = state => state.tasks.allTasks;

// Selector with parameter
export const selectTaskById = taskId => state => state.tasks.allTasks[taskId];

// Cross-slice selector
export const selectTaskWithProject = taskId => state => {
  const task = state.tasks.allTasks[taskId];
  const project = state.projects.allProjects[task.projectId];
  return { ...task, project };
};
```
These basic patterns show how selectors can access data. The cross-slice selector demonstrates combining data without modifying the original state - it creates a new object with the combined data only when the selector is called.

## Using createSelector for Memoization
```javascript
import { createSelector } from '@reduxjs/toolkit';

// Basic memoized selector
export const selectFilteredTasks = createSelector(
  state => state.tasks.allTasks,
  state => state.filters.status,
  (tasks, status) => tasks.filter(task => task.status === status)
);

// Multiple dependencies
export const selectEnrichedTask = taskId => createSelector(
  state => state.tasks.allTasks[taskId],
  state => state.features.allFeatures,
  state => state.projects.allProjects,
  (task, features, projects) => ({
    ...task,
    feature: features[task.featureId],
    project: projects[features[task.featureId]?.projectId]
  })
);
```
createSelector is crucial for performance. It only recomputes when its input selectors return new values. The enriched task selector combines data from multiple slices without modifying the original state - it creates a new enriched object only when the underlying data changes.

## Data Enrichment Deep Dive
When enriching data across slices:
1. Original state remains unchanged
2. New combined objects are created only when needed
3. Memoization prevents unnecessary recalculations
4. References to original data are maintained

Example of data flow:
```javascript
// In your state
{
  tasks: { 1: { id: 1, featureId: 100 } },
  features: { 100: { id: 100, projectId: 200 } },
  projects: { 200: { id: 200, name: "Project A" } }
}

// Enriched selector creates
{
  id: 1,
  featureId: 100,
  feature: { id: 100, projectId: 200 },
  project: { id: 200, name: "Project A" }
}
// Only when the data is requested and only if the source data changed
```

## Using in Components
```javascript
// Simple usage
function TaskList() {
  const tasks = useSelector(selectTasks);
}

// With parameters
function TaskItem({ taskId }) {
  const task = useSelector(state => selectTaskById(taskId)(state));
}

// With memoized selector
function FilteredTaskList() {
  const filteredTasks = useSelector(selectFilteredTasks);
}
```
Components receive derived data without knowing about the complexity of the state structure. The memoization ensures components only re-render when necessary.

## Common Data Combination Patterns
```javascript
// Filtering with relationships
export const selectTasksByStatus = status => state =>
  Object.values(state.tasks.allTasks)
    .filter(task => task.status === status);

// Complex data enrichment
export const selectTasksWithContext = createSelector(
  state => state.tasks.allTasks,
  state => state.features.allFeatures,
  state => state.projects.allProjects,
  (tasks, features, projects) => Object.values(tasks).map(task => ({
    ...task,
    feature: features[task.featureId],
    project: projects[features[task.featureId]?.projectId]
  }))
);

// Computed values from relationships
export const selectProjectProgress = projectId => createSelector(
  state => Object.values(state.tasks.allTasks)
    .filter(task => task.projectId === projectId),
  tasks => {
    const total = tasks.length;
    const completed = tasks.filter(t => t.status === 'completed').length;
    return (completed / total) * 100;
  }
);
```
These patterns show how to:
1. Maintain normalized state while providing denormalized views
2. Calculate derived data efficiently
3. Handle complex relationships without duplicating data
4. Create computed values from related data

## Performance Considerations
When combining data:
1. Use memoization for expensive computations
2. Keep normalized state structure
3. Create denormalized views only when needed
4. Consider the depth of relationships
5. Watch out for circular dependencies

## State Management Best Practices
Remember:
- Keep state normalized
- Use selectors for denormalization
- Memoize complex calculations
- Document relationships between slices
- Consider the performance impact of deep object graphs

The key is understanding that selectors provide a view into your state without modifying it. They can combine data from multiple sources efficiently through memoization, while maintaining the benefits of a normalized state structure.