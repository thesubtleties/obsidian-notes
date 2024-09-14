# SQLite Foreign Key Syntax Cheat Sheet

## Implicit Foreign Key Syntax

### Basic Syntax
```sql
column_name DATA_TYPE REFERENCES parent_table(parent_column)
```

### Example
```sql {3}
CREATE TABLE child_table (
  id INTEGER PRIMARY KEY,
  parent_id INTEGER REFERENCES parent_table(id)
);
```

### With ON DELETE / ON UPDATE
```sql
CREATE TABLE child_table (
  id INTEGER PRIMARY KEY,
  parent_id INTEGER REFERENCES parent_table(id) ON DELETE CASCADE ON UPDATE CASCADE
);
```

## Explicit Foreign Key Syntax

### Basic Syntax
```sql
FOREIGN KEY (column_name) REFERENCES parent_table(parent_column)
```

### Example
```sql {3-4}
CREATE TABLE child_table (
  id INTEGER PRIMARY KEY,
  parent_id INTEGER,
  FOREIGN KEY (parent_id) REFERENCES parent_table(id)
);
```

### With ON DELETE / ON UPDATE
```sql
CREATE TABLE child_table (
  id INTEGER PRIMARY KEY,
  parent_id INTEGER,
  FOREIGN KEY (parent_id) REFERENCES parent_table(id) ON DELETE CASCADE ON UPDATE CASCADE
);
```

## Key Points

1. Implicit syntax is more concise but limited to single-column foreign keys.
2. Explicit syntax is more verbose but supports multi-column foreign keys.
3. Both achieve the same result in SQLite.
4. Always use `PRAGMA foreign_keys = ON;` to enable foreign key support in SQLite.
5. Explicit syntax is more portable across different database systems.

## Multi-Column Foreign Key (Explicit Only)

```sql
CREATE TABLE child_table (
  id INTEGER PRIMARY KEY,
  parent_id1 INTEGER,
  parent_id2 INTEGER,
  FOREIGN KEY (parent_id1, parent_id2) REFERENCES parent_table(id1, id2)
);
```

Remember: Consistency in syntax within a single table definition is crucial to avoid errors.