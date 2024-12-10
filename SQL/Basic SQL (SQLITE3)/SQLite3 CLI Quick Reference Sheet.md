## Starting SQLite3
```bash
# Open database (creates if doesn't exist)
sqlite3 database.db

# Open with common options
sqlite3 -column -header database.db

# Open in-memory database
sqlite3 :memory:
```

## Meta Commands
```sql
.help           -- Show help
.quit           -- Exit
.databases      -- List databases
.tables         -- List tables
.schema         -- Show schema
.schema TABLE   -- Show schema for specific table
.mode column    -- Set output mode to column
.headers on     -- Turn headers on
.timer on       -- Turn timer on
.separator "|"  -- Change separator
.dump          -- Dump database as SQL
.dump TABLE    -- Dump specific table
```

## Output Formatting
```sql
.mode csv      -- CSV output
.mode column   -- Column output
.mode box      -- Box-styled output
.mode line     -- Line output
.mode list     -- List output
.mode html     -- HTML output
.mode insert   -- SQL insert statements
```

## Table Operations
```sql
-- Create table
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT UNIQUE
);

-- Delete table
DROP TABLE users;

-- Modify table
ALTER TABLE users ADD COLUMN age INTEGER;
```

## Basic Queries
```sql
-- Insert data
INSERT INTO users (name, email) VALUES ('John', 'john@email.com');

-- Select data
SELECT * FROM users;
SELECT name, email FROM users WHERE id = 1;

-- Update data
UPDATE users SET name = 'Johnny' WHERE id = 1;

-- Delete data
DELETE FROM users WHERE id = 1;
```

## Importing/Exporting
```sql
-- Export database
.output dump.sql
.dump
.output stdout

-- Export query results
.output results.csv
.mode csv
SELECT * FROM users;
.output stdout

-- Import SQL file
.read file.sql

-- Import CSV
.mode csv
.import data.csv table_name
```

## Backup and Restore
```sql
-- Backup
.backup 'backup.db'

-- Restore
.restore 'backup.db'
```

## Common SQLite Pragmas
```sql
-- Show all pragmas
.pragma

-- Common settings
PRAGMA foreign_keys = ON;
PRAGMA journal_mode = WAL;
PRAGMA synchronous = NORMAL;
PRAGMA cache_size = -2000; -- 2MB cache
```

## Indexes
```sql
-- Create index
CREATE INDEX idx_name ON users(name);

-- Show indexes
.indexes
.indexes users

-- Remove index
DROP INDEX idx_name;
```

## Transactions
```sql
BEGIN TRANSACTION;
-- ... SQL statements ...
COMMIT;
-- or
ROLLBACK;
```

## Database Analysis
```sql
-- Table info
.schema users
PRAGMA table_info(users);

-- Database size
PRAGMA page_size;
PRAGMA page_count;

-- Index list
PRAGMA index_list(users);
```

## Common Gotchas
- Case-sensitive by default
- No strict type checking
- No built-in date/time type
- Single-writer at a time
- 2GB max file size on some systems

## Best Practices
- Use transactions for multiple operations
- Create indexes for frequently queried columns
- Regular backups
- Use PRAGMA foreign_keys = ON
- Use prepared statements in applications
- Regular VACUUM for used space recovery

## Quick Tips
```sql
-- Count rows
SELECT COUNT(*) FROM users;

-- Limit results
SELECT * FROM users LIMIT 10;

-- Order results
SELECT * FROM users ORDER BY name DESC;

-- Simple search
SELECT * FROM users WHERE name LIKE '%John%';
```