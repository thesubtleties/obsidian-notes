## Lazy Loading

Definition: Associations are loaded only when they are specifically requested.

### Syntax:
```javascript
const user = await User.findOne({ where: { id: 1 } });
const posts = await user.getPosts();
```

### Pros:
- Loads only what's immediately necessary
- Can be more memory-efficient for large datasets
- Useful when you're not sure if you'll need the associated data

### Cons:
- Can lead to the N+1 query problem
- Multiple database queries if you need associated data

### Use When:
- You don't always need the associated data
- Dealing with large amounts of data where eager loading might be too memory-intensive

## Eager Loading

Definition: Associations are loaded automatically along with the main model in a single query.

### Syntax:
```javascript
const user = await User.findOne({
  where: { id: 1 },
  include: [{ model: Post }]
});
```

### Pros:
- Reduces number of database queries
- Solves N+1 query problem
- Can be more performant when you know you'll need the associated data

### Cons:
- Loads all data upfront, which might be unnecessary
- Can be memory-intensive for large datasets

### Use When:
- You know you'll need the associated data
- Optimizing for fewer database queries
- Dealing with manageable amounts of data

## Advanced Eager Loading

### Nested Includes:
```javascript
const user = await User.findOne({
  include: [{
    model: Post,
    include: [{
      model: Comment
    }]
  }]
});
```

### Specific Attributes:
```javascript
const user = await User.findOne({
  include: [{
    model: Post,
    attributes: ['title', 'content']
  }]
});
```

### Conditional Loading:
```javascript
const user = await User.findOne({
  include: [{
    model: Post,
    where: { status: 'published' }
  }]
});
```

## Lazy Loading After Eager Loading

```javascript
const user = await User.findOne({ include: [Post] });
const comments = await user.Posts[0].getComments();
```

## Key Differences

1. **Query Execution:**
   - Lazy: Multiple queries as needed
   - Eager: Single query with joins

2. **Data Availability:**
   - Lazy: On-demand
   - Eager: Immediately available

3. **Performance:**
   - Lazy: Better for memory, potentially slower for multiple accesses
   - Eager: Better for reducing query numbers, potentially memory-intensive

4. **Flexibility:**
   - Lazy: More flexible, can decide what to load at runtime
   - Eager: Less flexible, need to specify inclusions upfront

## Best Practices

1. Use eager loading when you know you'll need the associated data.
2. Use lazy loading for optional or rarely-needed associations.
3. Be aware of the N+1 query problem with lazy loading.
4. Use eager loading with specific attributes to optimize data transfer.
5. Consider using separate queries for very complex data structures.

Remember, the choice between lazy and eager loading often depends on your specific use case, the amount of data you're dealing with, and your application's performance requirements.