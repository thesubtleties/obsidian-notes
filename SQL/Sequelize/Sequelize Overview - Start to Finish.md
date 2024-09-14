# [[Models Overview]]
# [[Queries Overview]]
# [[Migrations Overview]]
# [[Seeders Overview]]

## Installation

```bash
npm install sequelize
npm install sequelize-cli
npm install pg pg-hstore  # For PostgreSQL
```

## Initialization

```bash
npx sequelize-cli init
```

This creates:
- `config/config.json`
- `models/`
- `migrations/`
- `seeders/`

## Configuration (config/config.json)

```json
{
  "development": {
    "username": "root",
    "password": null,
    "database": "database_development",
    "host": "127.0.0.1",
    "dialect": "postgres"
  },
  // ... test and production configs
}
```

## Creating a Model

```bash
npx sequelize-cli model:generate --name User --attributes name:string,email:string
```

This creates:
- `models/user.js`
- A migration file

## Model Definition (models/user.js)

```javascript
'use strict';
const { Model } = require('sequelize');

module.exports = (sequelize, DataTypes) => {
  class User extends Model {
    static associate(models) {
      // define associations here
    }
  }
  User.init({
    name: DataTypes.STRING,
    email: DataTypes.STRING
  }, {
    sequelize,
    modelName: 'User',
  });
  return User;
};
```

## Migrations

Create a migration:
```bash
npx sequelize-cli migration:generate --name add-column-to-users
```

Run migrations:
```bash
npx sequelize-cli db:migrate
```

Undo migrations:
```bash
npx sequelize-cli db:migrate:undo
```

## Seeders

Create a seeder:
```bash
npx sequelize-cli seed:generate --name demo-users
```

Run seeders:
```bash
npx sequelize-cli db:seed:all
```

Undo seeders:
```bash
npx sequelize-cli db:seed:undo:all
```

## Basic CRUD Operations

```javascript
const { User } = require('./models');

// Create
const user = await User.create({ name: 'John', email: 'john@example.com' });

// Read
const allUsers = await User.findAll();
const user = await User.findByPk(1);
const johnDoe = await User.findOne({ where: { name: 'John' } });

// Update
await User.update({ email: 'newemail@example.com' }, { where: { id: 1 } });

// Delete
await User.destroy({ where: { id: 1 } });
```

## Querying

```javascript
const { Op } = require('sequelize');

const users = await User.findAll({
  where: {
    name: {
      [Op.like]: 'John%'
    },
    age: {
      [Op.gte]: 18
    }
  },
  order: [['name', 'ASC']],
  limit: 10,
  offset: 0
});
```

## Associations

```javascript
// In models/user.js
User.associate = function(models) {
  User.hasMany(models.Post);
};

// In models/post.js
Post.associate = function(models) {
  Post.belongsTo(models.User);
};

// Usage
const userWithPosts = await User.findOne({
  include: [{ model: Post }]
});
```

## Transactions

```javascript
const t = await sequelize.transaction();

try {
  const user = await User.create({ name: 'John' }, { transaction: t });
  await user.addRole(roleId, { transaction: t });
  await t.commit();
} catch (error) {
  await t.rollback();
}
```

## Hooks

```javascript
User.beforeCreate(async (user, options) => {
  user.password = await bcrypt.hash(user.password, 10);
});
```

## Scopes

```javascript
// In model definition
static init(attributes, options) {
  super.init(attributes, {
    ...options,
    scopes: {
      active: {
        where: { status: 'active' }
      }
    }
  });
}

// Usage
const activeUsers = await User.scope('active').findAll();
```

## Raw Queries

```javascript
const [results, metadata] = await sequelize.query("SELECT * FROM users WHERE status = ?", {
  replacements: ['active'],
  type: QueryTypes.SELECT
});
```

Remember to handle errors and close database connections appropriately in your application. This cheatsheet provides a quick reference for common Sequelize operations, but always refer to the official documentation for detailed information and best practices.