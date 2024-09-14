

Here's the updated cheat sheet with comments added to explain non-obvious expressions:

# SQL Advanced Features Cheat Sheet

## CASE Statements
- Conditional logic in SQL queries
- Similar to if-else statements

```sql
SELECT 
    product_name,
    price,
    CASE 
        WHEN price < 10 THEN 'Cheap'
        WHEN price BETWEEN 10 AND 50 THEN 'Moderate'
        ELSE 'Expensive'
    END AS price_category  -- Creates a new column 'price_category'
FROM products;
```


## Window Functions
- Perform calculations across row sets
- Useful for running totals, rankings, moving averages

```sql
SELECT 
    order_date,
    order_amount,
    SUM(order_amount) OVER (ORDER BY order_date) AS running_total
    -- OVER clause defines the window of rows to operate on
    -- ORDER BY within OVER determines the order for the running total
FROM orders;
```

## Common Table Expressions (CTEs)
- Named subqueries for better readability
- Use WITH clause

```sql
WITH high_value_customers AS (
    -- This CTE defines 'high_value_customers' for use in the main query
    SELECT customer_id
    FROM orders
    GROUP BY customer_id
    HAVING SUM(order_amount) > 10000  -- Filters for customers with total orders > 10000
)
SELECT c.name, c.email
FROM customers c
JOIN high_value_customers hvc ON c.id = hvc.customer_id;
-- The CTE is used here like a regular table
```

## EXISTS Clause
- Tests for existence of rows satisfying a subquery
- Often more efficient than IN for large datasets

```sql
SELECT product_name
FROM products p
WHERE EXISTS (
    SELECT 1  -- 1 is arbitrary; the query checks for any matching row
    FROM order_items oi 
    WHERE oi.product_id = p.id  -- Correlated subquery: refers to outer query
);
-- Returns products that have been ordered at least once
```

## Key Benefits
- Simplifies complex queries
- Improves query performance
- Enhances code readability
- Allows complex logic within SQL
- Reduces need for multiple queries or application-side processing
