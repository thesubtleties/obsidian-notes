### Comparison Operators

- `[Op.eq]`: Equal
- `[Op.ne]`: Not equal
- `[Op.is]`: Is (for NULL comparisons)
- `[Op.not]`: Is not
- `[Op.gt]`: Greater than
- `[Op.gte]`: Greater than or equal
- `[Op.lt]`: Less than
- `[Op.lte]`: Less than or equal
- `[Op.between]`: Between
- `[Op.notBetween]`: Not between

### String Operators

- `[Op.like]`: LIKE
- `[Op.notLike]`: NOT LIKE
- `[Op.startsWith]`: Starts with
- `[Op.endsWith]`: Ends with
- `[Op.substring]`: Contains substring

### Array Operators

- `[Op.in]`: In array
- `[Op.notIn]`: Not in array
- `[Op.any]`: Any element in array matches (PostgreSQL only)

### Logical Operators

- `[Op.and]`: AND
- `[Op.or]`: OR
- `[Op.not]`: NOT

### JSON Operators (PostgreSQL specific)

- `[Op.contains]`: Contains
- `[Op.contained]`: Contained in

### Date Operators

- `[Op.col]`: Column comparison

### Other Operators

- `[Op.regexp]`: Regular expression match (MySQL and PostgreSQL only)
- `[Op.notRegexp]`: Does not match regular expression (MySQL and PostgreSQL only)
- `[Op.iRegexp]`: Case-insensitive regular expression match (PostgreSQL only)
- `[Op.notIRegexp]`: Case-insensitive does not match regular expression (PostgreSQL only)

### Usage Examples

```javascript
const { Op } = require('sequelize');

// Equal
{ name: { [Op.eq]: 'John' } }

// Greater than
{ age: { [Op.gt]: 18 } }

// LIKE
{ name: { [Op.like]: '%doe%' } }

// IN
{ status: { [Op.in]: ['active', 'pending'] } }

// AND
{ [Op.and]: [{ status: 'active' }, { age: { [Op.gte]: 21 } }] }

// OR
{ [Op.or]: [{ status: 'active' }, { age: { [Op.gte]: 65 } }] }

// Complex nested
{
  [Op.and]: [
    { status: 'active' },
    {
      [Op.or]: [
        { role: 'admin' },
        { age: { [Op.gte]: 21 } }
      ]
    }
  ]
}
```

Remember to import the operators at the top of your file:
```javascript
const { Op } = require('sequelize');
```

This cheat sheet covers the most commonly used Sequelize operators. You can refer to the official Sequelize documentation for a complete list and more detailed information on each operator.