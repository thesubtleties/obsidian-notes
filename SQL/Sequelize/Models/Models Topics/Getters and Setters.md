## Basic Getter

```javascript
// In model definition
fullName: {
  type: DataTypes.VIRTUAL,
  get() {
    return `${this.firstName} ${this.lastName}`;
  }
}

// Usage
console.log(user.fullName);
```
Getters allow you to define computed properties that don't exist in the database.

## Basic Setter

```javascript
// In model definition
fullName: {
  type: DataTypes.VIRTUAL,
  set(value) {
    const [firstName, lastName] = value.split(' ');
    this.setDataValue('firstName', firstName);
    this.setDataValue('lastName', lastName);
  }
}

// Usage
user.fullName = 'John Doe';
```
Setters allow you to process data before it's set on the model instance.

#### More Examples

Getter Example:
```javascript
fullName: {
  type: DataTypes.VIRTUAL,
  get() {
    return `${this.firstName} ${this.lastName}`;
  }
}
```
This getter creates a virtual `fullName` field that combines `firstName` and `lastName`.

Setter Example:
```javascript
password: {
  type: DataTypes.STRING,
  set(value) {
    this.setDataValue('password', bcrypt.hashSync(value, 10));
  }
}
```
This setter automatically hashes the password before it's stored in the database.

#### [[Getter Setter Example|Longer Example 1]]
#### [[Getter Setter Example 2|Longer Example 2]]

## Getter for Existing Column

```javascript
// In model definition
email: {
  type: DataTypes.STRING,
  get() {
    const rawValue = this.getDataValue('email');
    return rawValue ? rawValue.toLowerCase() : null;
  }
}
```
You can also define getters for existing database columns to transform data when it's accessed.

## Setter for Existing Column

```javascript
// In model definition
password: {
  type: DataTypes.STRING,
  set(value) {
    this.setDataValue('password', bcrypt.hashSync(value, 10));
  }
}
```
Setters on existing columns allow you to transform data before it's saved to the database.

## Combining Getter and Setter

```javascript
// In model definition
price: {
  type: DataTypes.INTEGER,
  get() {
    const rawValue = this.getDataValue('price');
    return rawValue / 100; // Convert cents to dollars
  },
  set(value) {
    this.setDataValue('price', value * 100); // Convert dollars to cents
  }
}
```
You can combine getters and setters to handle both reading and writing of a property.

## Using with JSON

```javascript
// In model definition
preferences: {
  type: DataTypes.JSON,
  get() {
    const rawValue = this.getDataValue('preferences');
    return rawValue ? JSON.parse(rawValue) : {};
  },
  set(value) {
    this.setDataValue('preferences', JSON.stringify(value));
  }
}
```
Useful for storing and retrieving JSON data in databases that don't have native JSON support.

## Virtual Fields with Dependencies

```javascript
// In model definition
class User extends Model {
  fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}

User.init({
  firstName: DataTypes.STRING,
  lastName: DataTypes.STRING,
  fullName: {
    type: DataTypes.VIRTUAL,
    get() {
      return this.fullName();
    }
  }
}, { sequelize });
```
You can use instance methods to create more complex virtual fields.

## Getters in Queries

```javascript
const users = await User.findAll({
  attributes: ['id', 'firstName', 'lastName', 'fullName']
});
```
Virtual fields with getters can be included in query results.

## Key Points and Gotchas

1. Virtual fields (with getters/setters) are not stored in the database.
2. Getters are called automatically when accessing the property.
3. Setters are called automatically when setting the property.
4. Use `this.getDataValue()` and `this.setDataValue()` to avoid infinite loops.
5. Getters and setters can impact performance if overused or poorly implemented.
6. Virtual fields are not included in `toJSON()` output by default. To include them:

```javascript
{
  toJSON() {
    const values = Object.assign({}, this.get());
    return values;
  }
}
```

7. Be cautious with setters that modify other fields, as it may lead to unexpected behavior.
8. Getters and setters are not called for bulk operations (e.g., `bulkCreate`).

Remember, while getters and setters are powerful, they should be used judiciously to maintain code clarity and performance.