## Creating a Migration

```bash
npx sequelize-cli migration:generate --name create-users-table
```

## Basic Migration Structure

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
        type: Sequelize.STRING
      },
      email: {
        type: Sequelize.STRING,
        unique: true
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

## Running Migrations

```bash
# Run all pending migrations
npx sequelize-cli db:migrate

# Undo the most recent migration
npx sequelize-cli db:migrate:undo

# Undo all migrations
npx sequelize-cli db:migrate:undo:all

# Run migrations up to a specific one
npx sequelize-cli db:migrate --to XXXXXXXXXXXXXX-create-users-table.js

# Undo migrations down to a specific one
npx sequelize-cli db:migrate:undo --to XXXXXXXXXXXXXX-create-posts-table.js
```

## Common Migration Operations

### Adding a Column

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.addColumn('Users', 'age', {
    type: Sequelize.INTEGER
  });
},

down: async (queryInterface, Sequelize) => {
  await queryInterface.removeColumn('Users', 'age');
}
```

### Removing a Column

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.removeColumn('Users', 'age');
},

down: async (queryInterface, Sequelize) => {
  await queryInterface.addColumn('Users', 'age', {
    type: Sequelize.INTEGER
  });
}
```

### Changing Column Type

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.changeColumn('Users', 'email', {
    type: Sequelize.STRING(100),
    allowNull: false,
    unique: true
  });
},

down: async (queryInterface, Sequelize) => {
  await queryInterface.changeColumn('Users', 'email', {
    type: Sequelize.STRING,
    allowNull: true,
    unique: false
  });
}
```

### Adding an Index

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.addIndex('Users', ['email']);
},

down: async (queryInterface, Sequelize) => {
  await queryInterface.removeIndex('Users', ['email']);
}
```

### Adding a Foreign Key

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.addColumn('Posts', 'userId', {
    type: Sequelize.INTEGER,
    references: {
      model: 'Users',
      key: 'id'
    },
    onUpdate: 'CASCADE',
    onDelete: 'SET NULL'
  });
},

down: async (queryInterface, Sequelize) => {
  await queryInterface.removeColumn('Posts', 'userId');
}
```

## Using Transactions in Migrations

```javascript
up: async (queryInterface, Sequelize) => {
  const transaction = await queryInterface.sequelize.transaction();
  try {
    await queryInterface.createTable('Users', { /* ... */ }, { transaction });
    await queryInterface.addIndex('Users', ['email'], { transaction });
    await transaction.commit();
  } catch (error) {
    await transaction.rollback();
    throw error;
  }
}
```

## Key Points

1. Migrations allow version control of your database schema.
2. Always include both `up` and `down` methods for reversibility.
3. Use transactions for complex migrations to ensure data integrity.
4. Migrations are run in order based on their timestamp prefixes.
5. Test migrations thoroughly, especially the `down` methods.
6. Avoid modifying existing migration files after they've been applied to other environments.
7. Use `queryInterface` methods for database-agnostic operations.
8. Consider the impact of migrations on existing data.
9. For production, always backup your database before running migrations.
10. Use `.js` extension for migration files to allow for complex logic when needed.

Remember, migrations are powerful but can be dangerous if not used carefully. Always review and test your migrations thoroughly before applying them to production databases.