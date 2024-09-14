## Model Definition Styles

### Standard Look
```javascript
Model.init({
  fieldName: DataTypes.STRING
}, { sequelize });
```

### Object Look with Type
```javascript
Model.init({
  fieldName: {
    type: DataTypes.STRING
  }
}, { sequelize });
```

### With Validations and Constraints
```javascript
Model.init({
  fieldName: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true,
    validate: {
      isEmail: true,
      len: [3, 50]
    }
  }
}, { sequelize });
```

## Common Validations and Constraints

```javascript
{
  // Validations
  validate: {
    isEmail: true,           // Checks for email format
    isUrl: true,             // Checks for URL format
    isIP: true,              // Checks for IPv4 or IPv6 format
    isAlpha: true,           // Will only allow letters
    isAlphanumeric: true,    // Will only allow alphanumeric characters
    isNumeric: true,         // Will only allow numbers
    isInt: true,             // Checks for valid integers
    isFloat: true,           // Checks for valid floating point numbers
    isDecimal: true,         // Checks for any numbers
    isLowercase: true,       // Checks for lowercase
    isUppercase: true,       // Checks for uppercase
    notNull: true,           // Won't allow null
    isNull: true,            // Only allows null
    notEmpty: true,          // Won't allow empty strings
    equals: 'specific value',// Only allow a specific value
    contains: 'foo',         // Force a string to contain a substring
    notIn: [['foo', 'bar']], // Check the value is not one of these
    isIn: [['foo', 'bar']],  // Check the value is one of these
    notContains: 'bar',      // Don't allow specific substrings
    len: [2,10],             // Only allow values with length between 2 and 10
    isUUID: 4,               // Only allow uuids
    isDate: true,            // Only allow date strings
    isAfter: "2011-11-05",   // Only allow date strings after a specific date
    isBefore: "2011-11-05",  // Only allow date strings before a specific date
    max: 23,                 // Only allow values <= 23
    min: 23,                 // Only allow values >= 23
    isCreditCard: true,      // Check for valid credit card numbers
    
    // Custom validations
    isEven(value) {
      if (parseInt(value) % 2 !== 0) {
        throw new Error('Only even values are allowed!');
      }
    }
  },
  
  // Constraints
  allowNull: false,          // Won't allow null values
  unique: true,              // Ensures a unique value in the column
  primaryKey: true,          // Defines the primary key in the table
  autoIncrement: true,       // Automatically increments the value
  defaultValue: 'default',   // Sets a default value
  comment: 'This is a column', // Adds a comment to the column
}
```

## [[Hooks (Lifecycle Events)]]

Hooks are functions that are called before and after certain operations in Sequelize.

```javascript
Model.init({
  // ... fields
}, {
  hooks: {
    beforeValidate: (instance, options) => {
      // Example: Normalize email to lowercase before validation
      if (instance.email) {
        instance.email = instance.email.toLowerCase();
      }
    },
    afterValidate: (instance, options) => {
      // Example: Log validation completion
      console.log(`Validation completed for ${instance.constructor.name}`);
    },
    beforeCreate: (instance, options) => {
      // Example: Set a default value if not provided
      if (!instance.status) {
        instance.status = 'pending';
      }
    },
    afterCreate: (instance, options) => {
      // Example: Send welcome email after user creation
      sendWelcomeEmail(instance.email);
    },
    beforeUpdate: (instance, options) => {
      // Example: Update 'lastModified' timestamp before any update
      instance.lastModified = new Date();
    },
    afterUpdate: (instance, options) => {
      // Example: Log the changes after an update
      console.log(`Updated ${instance.constructor.name}:`, instance.changed());
    },
    beforeSave: (instance, options) => {
      // Example: Encrypt sensitive data before saving
      if (instance.changed('password')) {
        instance.password = encryptPassword(instance.password);
      }
    },
    afterSave: (instance, options) => {
      // Example: Clear cache after saving
      clearRelatedCache(instance);
    },
    beforeDestroy: (instance, options) => {
      // Example: Prevent deletion of admin user
      if (instance.role === 'admin') {
        throw new Error('Cannot delete admin user');
      }
    },
    afterDestroy: (instance, options) => {
      // Example: Remove related records after deletion
      deleteRelatedRecords(instance.id);
    }
  }
});
```

Hooks are useful for:
- Data transformation (e.g., converting to lowercase, formatting dates)
- Setting default values dynamically
- Logging or auditing
- Triggering external processes (e.g., sending emails, invalidating caches)
- Enforcing complex business rules that span multiple fields
