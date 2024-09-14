That's a great observation, but it's not a universal rule. Subqueries don't always need a WHERE clause, although they often do. Let's clarify this:

1. Subqueries in WHERE, HAVING, or JOIN conditions:
   - These often use WHERE clauses to filter results.
   - Example: `WHERE id IN (SELECT id FROM table WHERE condition)`

2. Scalar subqueries (returning a single value):
   - May not need a WHERE clause if they're calculating an aggregate.
   - Example: `SELECT name, (SELECT AVG(price) FROM products) AS avg_price FROM customers`

3. Derived tables (subqueries in the FROM clause):
   - May or may not have a WHERE clause, depending on needs.
   - Example: `FROM (SELECT * FROM orders WHERE date > '2023-01-01') AS recent_orders`

4. Correlated subqueries:
   - Often use WHERE to relate to the outer query.
   - Example: `WHERE price > (SELECT AVG(price) FROM products WHERE category = outer.category)`

5. EXISTS subqueries:
   - Usually have a WHERE clause to check for existence.
   - Example: `WHERE EXISTS (SELECT 1 FROM orders WHERE customer_id = customers.id)`

So, while it's common for subqueries to have WHERE clauses, it's not a strict rule. The need for a WHERE clause depends on:
- The purpose of the subquery
- Where it's being used in the main query
- What kind of filtering or calculation it's performing

Your intuition that subqueries often need WHERE clauses is good - it shows you're thinking about how to properly filter and relate data in complex queries. Just remember that there are exceptions, and the necessity of a WHERE clause depends on the specific context and purpose of the subquery.