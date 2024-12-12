Additional help for: https://appacademy.instructure.com/courses/319/pages/set-up-for-the-sqlalchemy-articles-2?module_item_id=56614
## 1. Project Setup

### Initial Directory Setup
```bash
# Create and enter project directory
mkdir learning-sqlalchemy
cd learning-sqlalchemy

# Check Python version
python --version  # Should be 3.8+
```
Make sure you're using a recent Python version - SQLAlchemy works best with Python 3.8 or newer.

### Virtual Environment Setup
```bash
# Install SQLAlchemy
pipenv install sqlalchemy~=1.4

# Activate virtual environment
pipenv shell
```
The ~=1.4 ensures compatibility while allowing minor version updates.

## 2. Database Setup

### Creating the Database
```bash
# Create and open SQLite database
sqlite3 dev.db
```
We're using SQLite for simplicity, but SQLAlchemy works with many databases.

### Table Structures

#### Owners Table
```sql
CREATE TABLE owners (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  first_name VARCHAR(255) NOT NULL,
  last_name VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL
);
```
Basic user information table with auto-incrementing primary key.

#### Ponies Table
```sql
CREATE TABLE ponies (  
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name VARCHAR(255) NOT NULL,
  birth_year INTEGER NOT NULL,
  breed VARCHAR(255),
  owner_id INTEGER NOT NULL,
  FOREIGN KEY (owner_id) REFERENCES owners(id)
);
```
Main entity table with a foreign key relationship to owners.

#### Handlers Table
```sql
CREATE TABLE handlers (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  first_name VARCHAR(50) NOT NULL,
  last_name VARCHAR(50) NOT NULL,
  employee_id VARCHAR(12) NOT NULL
);
```
Employee/handler information table.

#### Junction Table (Many-to-Many)
```sql
CREATE TABLE pony_handlers (
  pony_id INTEGER NOT NULL,
  handler_id INTEGER NOT NULL,
  PRIMARY KEY (pony_id, handler_id),
  FOREIGN KEY (pony_id) REFERENCES ponies(id),
  FOREIGN KEY (handler_id) REFERENCES handlers(id)
);
```
This demonstrates a composite primary key for many-to-many relationships.

## 3. Sample Data

### Inserting Owners
```sql
INSERT INTO owners (first_name, last_name, email)
VALUES
('Joey', 'Harker', 'joey@harker.edu'),
('Jay', 'Harker', 'jay@harker.edu'),
('Josetta', 'Harker', 'josetta@harker.edu');
```
Basic user records to start with.

### Inserting Ponies
```sql
INSERT INTO ponies (name, birth_year, breed, owner_id)
VALUES
('Lucky Loser', 2017, 'Halfinger', 2),
('Unlucky Usurper', 2012, 'Fleuve', 1),
('Impassive Emperor', 2016, 'Hirzai', 1);
```
Each pony is associated with an owner through owner_id.

### Inserting Handlers
```sql
INSERT INTO handlers (first_name, last_name, employee_id)
VALUES
('Zap', 'Branagan', 'O4F'),
('The', 'Crushinator', '00100010'),
('Bubblegum', 'Tate', 'bball117');
```
Employee records for handlers.

### Creating Relationships
```sql
INSERT INTO pony_handlers (pony_id, handler_id)
VALUES
(1, 1),
(1, 2),
(2, 2),
(3, 1),
(3, 3);
```
Creates many-to-many relationships between ponies and handlers.

## 4. Verifying Setup

### Check Data
```sql
-- View all tables
SELECT * FROM owners;
SELECT * FROM ponies;
SELECT * FROM handlers;
SELECT * FROM pony_handlers;
```
These commands help verify your setup is correct.

## 5. Database Relationships

### Relationship Types
```plaintext
One-to-Many:
- Owner → Ponies (one owner can have many ponies)

Many-to-Many:
- Ponies ↔ Handlers (through pony_handlers table)
```
Understanding these relationships is crucial for SQLAlchemy modeling.

## 6. Important Concepts

### Primary Keys
```plaintext
Single Column:
- owners.id
- ponies.id
- handlers.id

Composite (Multiple Columns):
- pony_handlers (pony_id, handler_id)
```
Primary keys ensure unique identification of records.

### Foreign Keys
```plaintext
Simple:
- ponies.owner_id → owners.id

Multiple:
- pony_handlers.pony_id → ponies.id
- pony_handlers.handler_id → handlers.id
```
Foreign keys maintain referential integrity between tables.

Remember:
- Keep track of your database file location
- Understand the relationships between tables
- Note the composite primary key in pony_handlers
- Verify data after insertion
- Maintain referential integrity
- Use appropriate data types
- Include NOT NULL constraints where needed
