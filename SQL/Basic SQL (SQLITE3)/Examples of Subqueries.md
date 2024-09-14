Certainly! I'll create simple tables and then provide brief examples for each of the 8 subquery uses. Let's use these tables:

```sql
CREATE TABLE Products (id INT, name TEXT, category TEXT, price DECIMAL);
CREATE TABLE Categories (id INT, name TEXT, active BOOLEAN);

INSERT INTO Products VALUES 
(1, 'Laptop', 'Electronics', 1000),
(2, 'T-shirt', 'Clothing', 20),
(3, 'Book', 'Books', 15),
(4, 'Smartphone', 'Electronics', 500);

INSERT INTO Categories VALUES 
(1, 'Electronics', TRUE),
(2, 'Clothing', TRUE),
(3, 'Books', FALSE);
```

Now, let's look at each subquery type:

1. WHERE clause:
```sql
SELECT * FROM Products WHERE category IN (SELECT name FROM Categories WHERE active = TRUE);
-- Returns Laptop, T-shirt, Smartphone (active categories)
```

2. FROM clause:
```sql
SELECT * FROM (SELECT * FROM Products WHERE price > 100) AS expensive_products;
-- Returns Laptop and Smartphone (products over $100)
```

3. SELECT clause:
```sql
SELECT name, price, (SELECT AVG(price) FROM Products) AS avg_price FROM Products;
-- Returns all products with their price and the overall average price
```

4. HAVING clause:
```sql
SELECT category, AVG(price) FROM Products 
GROUP BY category 
HAVING AVG(price) > (SELECT AVG(price) FROM Products);
-- Returns categories with above-average prices
```

5. JOIN clause:
```sql
SELECT p.* FROM Products p
JOIN (SELECT name FROM Categories WHERE active = TRUE) AS ac
ON p.category = ac.name;
-- Returns products in active categories (similar to #1 but using JOIN)
```

6. INSERT statement:
```sql
INSERT INTO Categories (name, active)
SELECT DISTINCT category, TRUE FROM Products WHERE category NOT IN (SELECT name FROM Categories);
-- Inserts any product categories not already in Categories table
```

7. UPDATE statement:
```sql
UPDATE Products 
SET price = price * 0.9 
WHERE category IN (SELECT name FROM Categories WHERE active = FALSE);
-- Applies 10% discount to products in inactive categories
```

8. EXISTS clause:
```sql
SELECT * FROM Products p
WHERE EXISTS (SELECT 1 FROM Categories c WHERE c.name = p.category AND c.active = TRUE);
-- Returns products in active categories (similar to #1 but using EXISTS)
```

