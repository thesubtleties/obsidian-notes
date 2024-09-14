## Common Hooks (In Model File, e.g., models/user.js)

```javascript
const User = sequelize.define('User', {
  // attributes
}, {
  hooks: {
    beforeValidate: (user, options) => {
      // Called before validation
    },
    afterValidate: (user, options) => {
      // Called after validation
    },
    beforeCreate: (user, options) => {
      // Called before a new instance is saved
    },
    afterCreate: (user, options) => {
      // Called after a new instance is saved
    },
    beforeUpdate: (user, options) => {
      // Called before an instance is updated
    },
    afterUpdate: (user, options) => {
      // Called after an instance is updated
    },
    beforeDestroy: (user, options) => {
      // Called before an instance is destroyed
    },
    afterDestroy: (user, options) => {
      // Called after an instance is destroyed
    }
  }
});
```
These hooks are defined within the model file and are called at different stages of an instance's lifecycle.

## Bulk Hooks (In Model File, e.g., models/user.js)

```javascript
const User = sequelize.define('User', {
  // attributes
}, {
  hooks: {
    beforeBulkCreate: (users, options) => {
      // Called before multiple instances are created
    },
    afterBulkCreate: (users, options) => {
      // Called after multiple instances are created
    },
    beforeBulkUpdate: (options) => {
      // Called before a bulk update
    },
    afterBulkUpdate: (options) => {
      // Called after a bulk update
    },
    beforeBulkDestroy: (options) => {
      // Called before a bulk destroy
    },
    afterBulkDestroy: (options) => {
      // Called after a bulk destroy
    }
  }
});
```
Bulk hooks are also defined in the model file and are used for operations that affect multiple instances at once.

## [[Separate Hook Files|Adding Hooks After Model Definition (In Model File or Separate Hook File, e.g., models/user.js or hooks/userHooks.js)]]

```javascript
User.addHook('beforeCreate', (user, options) => {
  // Logic here
});

// Or use the direct method
User.beforeCreate((user, options) => {
  // Logic here
});
```
These can be added in the model file after the model definition, or in a separate file dedicated to hooks.

## Removing Hooks (In Model File or [[Separate Hook Files]], e.g., models/user.js or hooks/userHooks.js)

```javascript
const hookToRemove = User.addHook('beforeCreate', () => {
  // Logic here
});

User.removeHook('beforeCreate', hookToRemove);
```
Hook removal typically occurs in the same file where the hook was added.

## Global Hooks (In Main Sequelize Configuration File, e.g., config/database.js)

```javascript
sequelize.addHook('beforeCreate', (model, options) => {
  // Logic for all models
});
```
Global hooks are usually defined in the main Sequelize configuration file where the Sequelize instance is created.

## Async Hooks (In Model File, e.g., models/user.js)

```javascript
User.beforeCreate(async (user, options) => {
  const hashedPassword = await bcrypt.hash(user.password, 10);
  user.password = hashedPassword;
});
```
Async hooks are typically defined in the model file, often after the initial model definition.

## Hooks and Transactions (In Model File, e.g., models/user.js)

```javascript
User.afterCreate((user, options) => {
  // If this hook is called within a transaction,
  // options.transaction will be available
  if (options.transaction) {
    // Do something with the transaction
  }
});
```
Hooks that interact with transactions are usually defined in the model file.

Remember, while these are typical locations, the exact file structure may vary based on your project's architecture. Some projects might have a dedicated hooks directory or include hooks in service layers.