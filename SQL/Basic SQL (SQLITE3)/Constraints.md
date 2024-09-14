

```sql
-- Ensures each employee has a unique identifier
CREATE TABLE employees (
  id INT,
  name VARCHAR(100),
  CONSTRAINT pk_employee PRIMARY KEY (id)
);

-- Establishes a relationship between orders and customers,
-- ensuring each order is associated with a valid customer
CREATE TABLE orders (
  id INT,
  customer_id INT,
  CONSTRAINT fk_customer FOREIGN KEY (customer_id) REFERENCES customers(id)
);

-- Prevents negative or zero prices for products
CREATE TABLE products (
  id INT,
  price DECIMAL(10,2),
  CONSTRAINT chk_price CHECK (price > 0)
);

-- Ensures no two users can have the same email address
CREATE TABLE users (
  id INT,
  email VARCHAR(100),
  CONSTRAINT unq_email UNIQUE (email)
);

-- Requires that every student must have a name (cannot be null)
CREATE TABLE students (
  id INT,
  name VARCHAR(100),
  CONSTRAINT nn_name NOT NULL (name)
);

-- Automatically sets the current timestamp when a log entry is created
CREATE TABLE logs (
  id INT,
  created_at TIMESTAMP,
  CONSTRAINT df_created_at DEFAULT CURRENT_TIMESTAMP
);

-- Ensures each combination of order_id and product_id is unique,
-- and that the quantity for each item is always positive
CREATE TABLE order_items (
  order_id INT,
  product_id INT,
  quantity INT,
  CONSTRAINT pk_order_item PRIMARY KEY (order_id, product_id),
  CONSTRAINT chk_quantity CHECK (quantity > 0)
);

-- Adds a constraint to ensure employee salaries are always above the minimum wage
ALTER TABLE employees
ADD CONSTRAINT chk_salary CHECK (salary > minimum_wage);
```

These constraints help maintain data integrity, enforce business rules, and prevent invalid data from being inserted into the database. They act as a first line of defense against data inconsistencies and errors.