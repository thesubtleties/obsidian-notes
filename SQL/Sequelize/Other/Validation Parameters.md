
1. Validation parameters:
You're right to ask about other parameters like "min". In Sequelize, these are part of the validation object. Here are some common validation parameters you can use:

- `min` and `max`: For numbers and dates
- `len`: For strings (array with min and max length)
- `isEmail`: Checks if the string is an email
- `isUrl`: Checks if the string is a URL
- `isIP`: Checks if the string is an IP address
- `isAlpha`: Checks if the string contains only letters
- `isAlphanumeric`: Checks if the string contains only letters and numbers
- `isNumeric`: Checks if the string contains only numbers
- `isInt`: Checks if the string is a valid integer
- `isFloat`: Checks if the string is a valid float
- `isDecimal`: Checks if the string is a valid decimal
- `isLowercase`: Checks if the string is lowercase
- `isUppercase`: Checks if the string is uppercase
- `notNull`: Prevents null values
- `is`: Checks if the string matches a regular expression
- `not`: Checks if the string does not match a regular expression

You can use these in your model definition like this:

```javascript
heightFt: {
  type: DataTypes.FLOAT,
  validate: {
    min: 0,
    max: 1000,
    isFloat: true
  }
},
```

2. Regarding `CURRENT_TIMESTAMP`:

You're correct that Sequelize typically handles `createdAt` and `updatedAt` automatically. However, the discrepancy you're seeing is likely due to the difference between what Sequelize does by default and what your specific requirements or expectations are.

By default, Sequelize will:
- Create `createdAt` and `updatedAt` fields
- Set them to the current time when a record is created or updated
- Use the JavaScript `Date` object to handle this, which doesn't always translate directly to `CURRENT_TIMESTAMP` in SQL

If you specifically need `CURRENT_TIMESTAMP` as the default value in your SQL schema, you can set it like this:

```javascript
createdAt: {
  type: DataTypes.DATE,
  defaultValue: Sequelize.literal('CURRENT_TIMESTAMP'),
},
updatedAt: {
  type: DataTypes.DATE,
  defaultValue: Sequelize.literal('CURRENT_TIMESTAMP'),
},
```

However, be aware that this might interfere with Sequelize's automatic handling of these fields. You might need to set `timestamps: false` in your model options and handle the updating of `updatedAt` manually.

The reason your expected schema includes `CURRENT_TIMESTAMP` could be because:
1. It's a requirement specified by your project or team
2. It's what you'd typically see in a raw SQL schema, even though Sequelize handles it differently by default
3. There might be some miscommunication between what Sequelize does and what the schema expectations are

Remember, Sequelize is an abstraction layer, and sometimes its default behavior doesn't exactly match raw SQL expectations. In many cases, this doesn't matter because Sequelize handles the discrepancies internally. However, if you need the exact SQL schema to match a specific requirement, you might need to use raw SQL or migrations to set up your table exactly as needed.