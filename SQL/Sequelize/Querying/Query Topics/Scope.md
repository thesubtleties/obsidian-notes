
## Setting Up Scopes (In Model Files)

### Basic Scope
```javascript
// In models/user.js
const User = sequelize.define('User', {
  // ... attributes
}, {
  scopes: {
    active: {
      where: { status: 'active' }
    }
  }
});
```
This defines a basic scope named 'active' that filters for users with 'active' status.
### Scope with Associations
```javascript
// In models/user.js
const User = sequelize.define('User', {
  // ... attributes
}, {
  scopes: {
    withPosts: {
      include: [{ model: Post }]
    }
  }
});
```
This scope includes associated Posts when querying Users.
### Dynamic Scope
```javascript
// In models/user.js
const User = sequelize.define('User', {
  // ... attributes
}, {
  scopes: {
    olderThan(age) {
      return {
        where: {
          age: { [Op.gt]: age }
        }
      }
    }
  }
});
```
This dynamic scope allows passing an age parameter to filter users older than the specified age.
### Scope with Attributes
```javascript
// In models/user.js
const User = sequelize.define('User', {
  // ... attributes
}, {
  scopes: {
    nameOnly: {
      attributes: ['id', 'name']
    }
  }
});
```
This scope limits the returned attributes to only 'id' and 'name'.
### Default Scope
```javascript
// In models/user.js
const User = sequelize.define('User', {
  // ... attributes
}, {
  defaultScope: {
    where: { status: 'active' }
  }
});
```
This default scope is automatically applied to all queries on the User model unless overridden.
## Querying with Scopes (In Route Files or Service Layer)

### Using a Single Scope
```javascript
// In routes/userRoutes.js or services/userService.js
const activeUsers = await User.scope('active').findAll();
```
This query applies the 'active' scope to fetch only active users.
### Chaining Multiple Scopes
```javascript
// In routes/userRoutes.js or services/userService.js
const users = await User.scope('active', 'withPosts').findAll();
```
This applies both 'active' and 'withPosts' scopes, fetching active users with their associated posts.
### Using Dynamic Scope
```javascript
// In routes/userRoutes.js or services/userService.js
const olderUsers = await User.scope({ method: ['olderThan', 30] }).findAll();
```
This uses the dynamic 'olderThan' scope to find users older than 30.
### Combining Scopes with Additional Options
```javascript
// In routes/userRoutes.js or services/userService.js
const users = await User.scope('active').findAll({
  where: { country: 'USA' }
});
```
This applies the 'active' scope and adds an additional condition to filter by country.
### Removing Default Scope
```javascript
// In routes/userRoutes.js or services/userService.js
const allUsers = await User.unscoped().findAll();
```
This query removes the default scope, fetching all users regardless of their status.
### Scope on Included Model
```javascript
// In routes/userRoutes.js or services/userService.js
const users = await User.findAll({
  include: [{
    model: Post,
    scope: 'published'
  }]
});
```
This applies a scope on the included Post model, assuming Post has a 'published' scope defined.
## Key Points and Gotchas

1. Scopes are defined in model files (e.g., `models/user.js`).
2. Scope usage typically occurs in route handlers (e.g., `routes/userRoutes.js`) or service layers (e.g., `services/userService.js`).
3. Default scopes are always applied unless explicitly removed with `unscoped()`.
4. Scopes can be combined and chained for complex queries.
5. Dynamic scopes allow for flexible, parameterized queries.
6. Be cautious with default scopes as they can lead to unexpected query results if forgotten about.
7. Scopes are merged in the order they are chained, with later scopes potentially overriding earlier ones.
8. Always test scoped queries with representative data to ensure they perform as expected.

Remember, while scopes are powerful, they should be used judiciously. Overuse of complex scopes can make your code harder to understand and maintain. Always consider the balance between reusability and clarity in your database queries.