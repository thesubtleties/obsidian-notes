1. Definition of a Selector:
   A selector is a function that takes the entire Redux state as an argument and returns a specific piece of that state.

2. File Structure:
   Typically, selectors are organized in a separate file, often named `selectors.js`, within the relevant feature folder:

   ```
   src/
     features/
       users/
         usersSlice.js
         usersSelectors.js
         UsersList.js
   ```

3. Creating Selectors:
   In `usersSelectors.js`:
   ```javascript
   export const selectAllUsers = (state) => state.users.list;
   export const selectUserById = (state, userId) => state.users.list.find(user => user.id === userId);
   ```

   These selectors encapsulate the knowledge of the state structure, making it easier to reshape or compute derived data.

4. Usage in Components:
   In `UsersList.js`:
   ```javascript
   import { useSelector } from 'react-redux';
   import { selectAllUsers } from './usersSelectors';

   function UsersList() {
     const users = useSelector(selectAllUsers);
     // ...
   }
   ```

   Selectors are typically used within the `useSelector` hook in functional components.

5. Complex Selectors with Reselect:
   In `usersSelectors.js`:
   ```javascript
   import { createSelector } from 'reselect';

   const selectUsers = state => state.users.list;
   const selectCurrentUserId = state => state.users.currentUserId;

   export const selectCurrentUser = createSelector(
     [selectUsers, selectCurrentUserId],
     (users, currentUserId) => users.find(user => user.id === currentUserId)
   );
   ```

   Reselect is a library for creating memoized selectors. `createSelector` takes an array of input selectors and a transform function. It only recomputes the result when one of the input selectors changes, optimizing performance for derived data.

6. Selector Naming Conventions:
   - Prefix with `select`: e.g., `selectAllUsers`, `selectUserById`
   - Be descriptive: `selectActiveUsers`, `selectCompletedTodos`

7. Organizing Selectors:
   - Group related selectors in the same file
   - For large applications, consider a `selectors` folder within each feature

8. Selectors vs. Other Files:
   - Selectors: Focus on retrieving and computing state data
   - Reducers (e.g., `usersSlice.js`): Handle state updates
   - Components (e.g., `UsersList.js`): Render UI and use selectors

9. Benefits of Separate Selector Files:
   - Improved organization and maintainability
   - Easy to locate and update state access logic
   - Facilitates reuse across multiple components

10. Testing Selectors:
    Create a separate test file, e.g., `usersSelectors.test.js`, to ensure selectors work correctly

Remember:
- Keep selectors pure functions for easy testing and predictability
- Use memoized selectors (via Reselect) for performance with derived data
- Selectors can be composed: simple selectors can be used to build more complex ones

This structure keeps your state access logic separate from your UI components and state update logic, promoting a clean and maintainable codebase.

Key Points:
- You don't import selectors; you define them based on your state structure.
- Selectors are just pure functions that know how to extract and possibly transform your state data.
- You can define selectors in the same file as your component, in a separate 'selectors' file, or wherever makes sense in your project structure.

Remember, the power of selectors comes from their ability to encapsulate the knowledge of your state structure, making your components more maintainable and your state access more consistent.