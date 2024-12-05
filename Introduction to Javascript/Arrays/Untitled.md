# JavaScript Array Transformation Strategies Cheat Sheet

## Quick Reference - Common Organization Patterns
- [Array to Map (by value)](#array-to-map)
- [Array to Object (by value)](#array-to-object)
- [Array to Map/Object (by ID)](#by-id)
- [Array to Map/Object (by Index)](#by-index)
- [Grouping/Categorizing](#grouping)
- [Counting/Frequency](#counting)
- [Lookup Tables](#lookup-tables)

## Common Use Cases

### Looking up items by unique ID
**Best Strategy:** Array to Object by ID
```javascript
const users = [
    { id: 1, name: 'John' },
    { id: 2, name: 'Jane' }
];

const userMap = users.reduce((obj, user) => {
    obj[user.id] = user;
    return obj;
}, {});
```

Alternative Approaches:
```javascript
// Using Map
const userMap = new Map(users.map(user => [user.id, user]));

// Using Object.fromEntries
const userObj = Object.fromEntries(
    users.map(user => [user.id, user])
);
```

### Fast existence checking
**Best Strategy:** Array to Set
```javascript
const items = ['apple', 'banana', 'orange'];
const itemSet = new Set(items);

// Usage
itemSet.has('apple'); // true
```

Alternative Approaches:
```javascript
// Using Object
const itemObj = items.reduce((obj, item) => {
    obj[item] = true;
    return obj;
}, {});

// Using Map
const itemMap = new Map(items.map(item => [item, true]));
```

### Grouping related items
**Best Strategy:** Reduce to Object with arrays
```javascript
const people = [
    { age: 20, name: 'John' },
    { age: 20, name: 'Jane' },
    { age: 30, name: 'Bob' }
];

const grouped = people.reduce((groups, person) => {
    (groups[person.age] = groups[person.age] || []).push(person);
    return groups;
}, {});
```

Alternative Approaches:
```javascript
// Using Map
const groupedMap = people.reduce((map, person) => {
    if (!map.has(person.age)) map.set(person.age, []);
    map.get(person.age).push(person);
    return map;
}, new Map());
```

<details>
<summary>More Complex Grouping Strategies</summary>

```javascript
// Multi-level grouping
const multiGroup = people.reduce((groups, person) => {
    const { age, gender } = person;
    groups[age] = groups[age] || {};
    groups[age][gender] = groups[age][gender] || [];
    groups[age][gender].push(person);
    return groups;
}, {});

// Grouping with custom key generator
const groupBy = (array, keyFn) => {
    return array.reduce((groups, item) => {
        const key = keyFn(item);
        (groups[key] = groups[key] || []).push(item);
        return groups;
    }, {});
};
```
</details>

### Sequential access with numeric keys
**Best Strategy:** Array to Object by Index
```javascript
const items = ['apple', 'banana', 'orange'];
const indexed = items.reduce((obj, item, index) => {
    obj[index] = item;
    return obj;
}, {});
```

Alternative Approaches:
```javascript
// Using Map with index
const indexMap = new Map(items.map((item, i) => [i, item]));

// Using Object.fromEntries
const indexObj = Object.fromEntries(items.entries());
```

<details>
<summary>Advanced Index-Based Strategies</summary>

```javascript
// Bidirectional lookup
const biLookup = items.reduce((obj, item, index) => {
    obj.byIndex[index] = item;
    obj.byValue[item] = index;
    return obj;
}, { byIndex: {}, byValue: {} });

// Chunked indexing
const chunk = (arr, size) => {
    return arr.reduce((chunks, item, i) => {
        const chunkIndex = Math.floor(i / size);
        (chunks[chunkIndex] = chunks[chunkIndex] || []).push(item);
        return chunks;
    }, {});
};
```
</details>

## Selection Guide

### Use Map when:
- Keys are non-strings
- Need to preserve insertion order
- Frequent additions/removals
- Need to track size easily
- Keys are unknown until runtime

### Use Object when:
- All keys are strings
- Need JSON serialization
- Simpler syntax preferred
- Working with prototype methods
- Performance is critical

### Use Set when:
- Only need to track existence
- Need unique values
- Fast lookup is priority
- Don't need key-value pairs

## Performance Considerations
```javascript
// For large arrays:
// 1. Pre-size when possible
const map = new Map();
map.set = map.set.bind(map);
array.forEach((item, i) => map.set(i, item));

// 2. Direct object assignment
const obj = {};
array.forEach((item, i) => obj[i] = item);
```

