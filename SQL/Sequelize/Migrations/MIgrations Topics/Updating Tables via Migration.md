## Basic Structure

```javascript
'use strict';

module.exports = {
  up: async (queryInterface, Sequelize) => {
    // Changes to make to the database
  },

  down: async (queryInterface, Sequelize) => {
    // How to revert the changes
  }
};
```

## Adding a Column

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.addColumn('TableName', 'newColumnName', {
    type: Sequelize.DataType, // Replace with real DataType
    allowNull: true,
    // other options...
  });
},

down: async (queryInterface, Sequelize) => {
  await queryInterface.removeColumn('TableName', 'newColumnName');
}
```

## Removing a Column

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.removeColumn('TableName', 'columnToRemove');
},

down: async (queryInterface, Sequelize) => {
  await queryInterface.addColumn('TableName', 'columnToRemove', {
    type: Sequelize.DataType, // Replace with real DataType
    // other options...
  });
}
```

## Changing Column Type

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.changeColumn('TableName', 'columnName', {
    type: Sequelize.NEW_DATA_TYPE,
    // other options...
  });
},

down: async (queryInterface, Sequelize) => {
  await queryInterface.changeColumn('TableName', 'columnName', {
    type: Sequelize.ORIGINAL_DATA_TYPE,
    // original options...
  });
}
```

## Renaming a Column

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.renameColumn('TableName', 'oldColumnName', 'newColumnName');
},

down: async (queryInterface, Sequelize) => {
  await queryInterface.renameColumn('TableName', 'newColumnName', 'oldColumnName');
}
```

## Adding an Index

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.addIndex('TableName', ['column1', 'column2'], {
    name: 'custom_index_name',
    unique: true
  });
},

down: async (queryInterface, Sequelize) => {
  await queryInterface.removeIndex('TableName', 'custom_index_name');
}
```

## Changing Column Constraints

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.changeColumn('TableName', 'columnName', {
    allowNull: false,
    defaultValue: 'some value'
  });
},

down: async (queryInterface, Sequelize) => {
  await queryInterface.changeColumn('TableName', 'columnName', {
    allowNull: true,
    defaultValue: null
  });
}
```

## Adding a Foreign Key

```javascript
up: async (queryInterface, Sequelize) => {
  await queryInterface.addConstraint('TableName', {
    fields: ['foreignKeyColumn'],
    type: 'foreign key',
    name: 'custom_fkey_constraint_name',
    references: {
      table: 'ReferencedTableName',
      field: 'id'
    },
    onDelete: 'CASCADE',
    onUpdate: 'CASCADE'
  });
},

down: async (queryInterface, Sequelize) => {
  await queryInterface.removeConstraint('TableName', 'custom_fkey_constraint_name');
}
```

## Key Points to Remember

1. Always include both `up` and `down` methods for reversibility.
2. Test your migrations, especially the `down` methods.
3. Be cautious when changing existing data (e.g., when changing column types).
4. Use transactions for complex migrations involving multiple operations.
5. Consider the impact on existing data and application code when making schema changes.
6. When adding constraints or changing column properties, think about how it affects existing data.

Remember, while these examples cover common scenarios, Sequelize offers many more options and methods for database schema manipulation. Always refer to the official Sequelize documentation for the most up-to-date and detailed information.