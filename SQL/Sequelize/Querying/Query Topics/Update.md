## <= Open to see model files for reference
1. User Model (models/user.js):

```javascript
'use strict';
const { Model } = require('sequelize');

module.exports = (sequelize, DataTypes) => {
  class User extends Model {
    static associate(models) {
      // Associations can be defined here
      User.hasMany(models.Post);
      User.belongsToMany(models.Role, { through: 'UserRoles' });
    }
  }
  
  User.init({
    name: {
      type: DataTypes.STRING,
      allowNull: false
    },
    email: {
      type: DataTypes.STRING,
      allowNull: false,
      unique: true,
      validate: {
        isEmail: true
      }
    },
    status: {
      type: DataTypes.ENUM('active', 'inactive'),
      defaultValue: 'active'
    },
    age: {
      type: DataTypes.INTEGER,
      validate: {
        min: 0
      }
    },
    credits: {
      type: DataTypes.INTEGER,
      defaultValue: 0
    },
    lastLogin: DataTypes.DATE
  }, {
    sequelize,
    modelName: 'User',
  });
  
  return User;
};
```

2. Post Model (models/post.js):

```javascript
'use strict';
const { Model } = require('sequelize');

module.exports = (sequelize, DataTypes) => {
  class Post extends Model {
    static associate(models) {
      // Associations can be defined here
      Post.belongsTo(models.User);
    }
  }
  
  Post.init({
    title: {
      type: DataTypes.STRING,
      allowNull: false
    },
    content: DataTypes.TEXT
  }, {
    sequelize,
    modelName: 'Post',
  });
  
  return Post;
};
```

3. User Migration (migrations/YYYYMMDDHHMMSS-create-user.js):

```javascript
'use strict';

module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('Users', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      name: {
        type: Sequelize.STRING,
        allowNull: false
      },
      email: {
        type: Sequelize.STRING,
        allowNull: false,
        unique: true
      },
      status: {
        type: Sequelize.ENUM('active', 'inactive'),
        defaultValue: 'active'
      },
      age: {
        type: Sequelize.INTEGER
      },
      credits: {
        type: Sequelize.INTEGER,
        defaultValue: 0
      },
      lastLogin: {
        type: Sequelize.DATE
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  down: async (queryInterface, Sequelize) => {
    await queryInterface.dropTable('Users');
  }
};
```

4. Post Migration (migrations/YYYYMMDDHHMMSS-create-post.js):

```javascript
'use strict';

module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('Posts', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      title: {
        type: Sequelize.STRING,
        allowNull: false
      },
      content: {
        type: Sequelize.TEXT
      },
      UserId: {
        type: Sequelize.INTEGER,
        references: {
          model: 'Users',
          key: 'id'
        },
        onUpdate: 'CASCADE',
        onDelete: 'SET NULL'
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  down: async (queryInterface, Sequelize) => {
    await queryInterface.dropTable('Posts');
  }
};
```

These files provide a basic structure that corresponds to the update examples in the cheatsheet. The User model includes fields like name, email, status, age, and credits, which are used in various update examples. The Post model is included to demonstrate association updates.

Key points:
1. The User model has a hasMany association with Post and a belongsToMany association with Role (though the Role model isn't shown here for brevity).
2. The Post model has a belongsTo association with User.
3. The migrations create the necessary tables and fields, including the foreign key for the User-Post association.
4. The User model includes validations (like isEmail) which correspond to the validation example in the update cheatsheet.

These models and migrations provide a foundation for all the update operations described in the cheatsheet. You can expand on these as needed for more complex scenarios.

## Basic Update Operations

### Update a single record by ID
```javascript
const updatedRows = await User.update(
  { name: 'John Doe' },  // Fields to update
  { where: { id: 1 } }   // Condition for update
);
// Returns number of rows affected
```
This updates the name of the user with id 1.

### Update multiple records
```javascript
const updatedRows = await User.update(
  { status: 'inactive' },
  { where: { lastLogin: { [Op.lt]: new Date() - 30 * 24 * 60 * 60 * 1000 } } }
);
```
This sets status to 'inactive' for all users who haven't logged in for 30 days.

### Update or create (upsert)
```javascript
const [user, created] = await User.upsert(
  { id: 1, name: 'John Doe', email: 'john@example.com' }
);
// 'created' is true if a new record was created, false if updated
```
This creates a new user if id 1 doesn't exist, or updates if it does.

## Advanced Update Operations

### Update with returning updated records (PostgreSQL)
```javascript
const [updatedRowsCount, updatedRows] = await User.update(
  { name: 'John Doe' },
  { where: { id: 1 }, returning: true }
);
```
This updates the user and returns the updated record (PostgreSQL only).

### Update with validation
```javascript
const updatedRows = await User.update(
  { email: 'invalid-email' },
  { where: { id: 1 }, validate: true }
);
```
This attempts to update the email, but will throw an error if it doesn't pass model validations.

### Update specific fields
```javascript
const updatedRows = await User.update(
  { name: 'John Doe', email: 'john@example.com' },
  { where: { id: 1 }, fields: ['name'] }
);
```
This updates only the 'name' field, ignoring other fields in the update object.

## Updating Associations

### Update belongsTo association
```javascript
const user = await User.findByPk(1);
await user.setProfile(profileId);
```
This sets a new profile for the user (assuming a User belongsTo Profile association).

### Update hasMany association
```javascript
const user = await User.findByPk(1);
await user.setPosts([postId1, postId2]);
```
This sets the posts for a user, removing any previous associations (assuming a User hasMany Posts association).

### Update belongsToMany association
```javascript
const user = await User.findByPk(1);
await user.setRoles([roleId1, roleId2]);
```
This sets the roles for a user, removing any previous role associations (assuming a User belongsToMany Roles association).

## Bulk Update Operations

### Bulk update with individual hooks
```javascript
const updatedUsers = await User.bulkCreate(
  [
    { id: 1, name: 'John Doe' },
    { id: 2, name: 'Jane Doe' }
  ],
  { updateOnDuplicate: ['name'] }
);
```
This updates multiple users in a single query, triggering hooks for each update.

### Bulk update without individual hooks
```javascript
const updatedCount = await User.bulkUpdate(
  { status: 'active' },
  { id: [1, 2, 3] }
);
```
This updates multiple users in a single query without triggering individual hooks, which is faster for large updates.

## Atomic Updates

### Increment a value
```javascript
await User.increment('age', { where: { id: 1 } });
```
This safely increments the 'age' field by 1 for the user with id 1.

### Decrement a value
```javascript
await User.decrement('credits', { by: 5, where: { id: 1 } });
```
This safely decrements the 'credits' field by 5 for the user with id 1.

## Conditional Updates

### Update with case statement (SQL-specific)
```javascript
await sequelize.query(`
  UPDATE Users
  SET status = CASE
    WHEN lastLogin < :date THEN 'inactive'
    ELSE 'active'
  END
`, {
  replacements: { date: new Date() - 30 * 24 * 60 * 60 * 1000 }
});
```
This performs a conditional update directly in SQL, setting users to 'inactive' if they haven't logged in for 30 days, otherwise 'active'.

## Transactions in Updates

### Update within a transaction
```javascript
const t = await sequelize.transaction();

try {
  await User.update(
    { name: 'John Doe' },
    { where: { id: 1 }, transaction: t }
  );
  await Profile.update(
    { userName: 'John Doe' },
    { where: { userId: 1 }, transaction: t }
  );
  await t.commit();
} catch (error) {
  await t.rollback();
  throw error;
}
```
This performs multiple updates within a transaction, ensuring all updates succeed or all fail together.

## Key Points

1. Always use `await` with update operations as they are asynchronous.
2. The `update` method returns an array with the number of affected rows.
3. Use `returning: true` to get updated records (PostgreSQL only).
4. Validate updates using `validate: true` to trigger model validations.
5. Use transactions for multiple related updates to ensure data consistency.
6. Bulk updates are more efficient for updating multiple records simultaneously.
7. Atomic updates (increment/decrement) are useful for updating numeric fields safely.

Remember: Always handle potential errors and validate input data before performing updates to maintain data integrity.