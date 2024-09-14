
## [[Adding One to Many via Migration|Adding a Simple Foreign Key]]

```javascript
'use strict';

module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.addColumn('Posts', 'userId', {
      type: Sequelize.INTEGER,
      allowNull: false,
      references: {
        model: 'Users',
        key: 'id'
      },
      onUpdate: 'CASCADE',
      onDelete: 'CASCADE'
    });
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.removeColumn('Posts', 'userId');
  }
};
```

## Key Options for Foreign Keys

- `type`: Data type of the foreign key (usually INTEGER)
- `allowNull`: Whether the foreign key can be null
- `references.model`: The table being referenced
- `references.key`: The column in the referenced table (usually 'id')
- `onUpdate`: Action on update of referenced record
- `onDelete`: Action on deletion of referenced record

## Common ON UPDATE/ON DELETE Actions

- `CASCADE`: Update/delete the referencing rows
- `SET NULL`: Set the referencing columns to NULL
- `RESTRICT`: Prevent the update/delete if there are referencing rows
- `NO ACTION`: Similar to RESTRICT, but checked after other constraints

## Adding Foreign Key to Existing Column

```javascript
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.addConstraint('Posts', {
      fields: ['userId'],
      type: 'foreign key',
      name: 'fk_user_id',
      references: {
        table: 'Users',
        field: 'id'
      },
      onDelete: 'CASCADE',
      onUpdate: 'CASCADE'
    });
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.removeConstraint('Posts', 'fk_user_id');
  }
};
```

## [[Adding Many to Many via Migration|Adding Many-to-Many Relationships]]

1. Create the junction table:

```javascript
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('UserRoles', {
      userId: {
        type: Sequelize.INTEGER,
        references: {
          model: 'Users',
          key: 'id'
        },
        onUpdate: 'CASCADE',
        onDelete: 'CASCADE'
      },
      roleId: {
        type: Sequelize.INTEGER,
        references: {
          model: 'Roles',
          key: 'id'
        },
        onUpdate: 'CASCADE',
        onDelete: 'CASCADE'
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

    // Add a composite unique constraint
    await queryInterface.addConstraint('UserRoles', {
      fields: ['userId', 'roleId'],
      type: 'unique',
      name: 'unique_user_role'
    });
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.dropTable('UserRoles');
  }
};
```

2. Define the associations in model files:

```javascript
// In User model
User.belongsToMany(models.Role, { through: 'UserRoles' });

// In Role model
Role.belongsToMany(models.User, { through: 'UserRoles' });
```

## Key Points

1. Always include both `up` and `down` methods for reversibility.
2. Use `queryInterface` methods for database-agnostic operations.
3. Foreign keys typically reference the 'id' column of another table.
4. Many-to-many relationships require a junction table with foreign keys to both related tables.
5. Add a unique constraint on junction table to prevent duplicate relationships.
6. The `through` option in model associations should match the junction table name.
7. Consider the impact of `onUpdate` and `onDelete` actions on data integrity.
8. Test migrations thoroughly, especially the `down` methods.
9. Use transactions for complex migrations to ensure data integrity.
10. For production, always backup your database before running migrations.

Remember, while foreign keys and relationships are powerful for maintaining data integrity, they can also impact performance and flexibility. Design your database schema carefully to balance these concerns.