# SQL Basics for Interview Preparation

## Table of Contents
1. [SQL Fundamentals](#sql-fundamentals)
2. [Data Types](#data-types)
3. [DDL (Data Definition Language)](#ddl-data-definition-language)
4. [DML (Data Manipulation Language)](#dml-data-manipulation-language)
5. [Queries and Joins](#queries-and-joins)
6. [Aggregate Functions](#aggregate-functions)
7. [Constraints](#constraints)
8. [Indexes](#indexes)
9. [Common Interview Questions](#common-interview-questions)

---

## SQL Fundamentals

### What is SQL?
SQL (Structured Query Language) is used to manage and manipulate relational databases.

### Basic Commands:
- **SELECT** - Query/retrieve data
- **INSERT** - Add new records
- **UPDATE** - Modify existing records
- **DELETE** - Remove records
- **CREATE** - Create database objects
- **ALTER** - Modify database objects
- **DROP** - Delete database objects

---

## Data Types

### Common SQL Data Types:

**Numeric:**
- `INT` or `INTEGER` - Whole numbers
- `DECIMAL(p,s)` or `NUMERIC(p,s)` - Fixed precision (p=precision, s=scale)
- `FLOAT` or `DOUBLE` - Floating point numbers
- `SMALLINT`, `BIGINT` - Smaller/larger integers

**String:**
- `VARCHAR(n)` - Variable-length string (max n characters)
- `CHAR(n)` - Fixed-length string (n characters)
- `TEXT` - Large text data

**Date/Time:**
- `DATE` - Date only (YYYY-MM-DD)
- `TIME` - Time only (HH:MM:SS)
- `DATETIME` or `TIMESTAMP` - Date and time

**Other:**
- `BOOLEAN` or `BOOL` - True/False
- `BLOB` - Binary large object

---

## DDL (Data Definition Language)

### CREATE TABLE

**Syntax:**
```sql
CREATE TABLE table_name (
    column1 datatype constraints,
    column2 datatype constraints,
    ...
);
```

**Example for Restaurant Management:**
```sql
-- Menu Items Table
CREATE TABLE menu_items (
    menu_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    category VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tables Table
CREATE TABLE tables (
    table_number INT PRIMARY KEY,
    capacity INT NOT NULL,
    status VARCHAR(20) DEFAULT 'FREE',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Orders Table
CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    table_id INT NOT NULL,
    status VARCHAR(20) DEFAULT 'CREATED',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (table_id) REFERENCES tables(table_number)
);

-- Order Items Table
CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    menu_id INT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (menu_id) REFERENCES menu_items(menu_id)
);

-- Bills Table
CREATE TABLE bills (
    bill_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL UNIQUE,
    total_amount DECIMAL(10, 2) NOT NULL,
    payment_method VARCHAR(50),
    payment_status VARCHAR(20) DEFAULT 'PENDING',
    payment_time TIMESTAMP,
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);
```

### ALTER TABLE

**Add Column:**
```sql
ALTER TABLE menu_items 
ADD COLUMN description TEXT;
```

**Modify Column:**
```sql
ALTER TABLE menu_items 
MODIFY COLUMN price DECIMAL(12, 2);
```

**Drop Column:**
```sql
ALTER TABLE menu_items 
DROP COLUMN description;
```

**Add Constraint:**
```sql
ALTER TABLE orders 
ADD CONSTRAINT fk_table FOREIGN KEY (table_id) REFERENCES tables(table_number);
```

### DROP TABLE
```sql
DROP TABLE menu_items;  -- Removes entire table
DROP TABLE IF EXISTS menu_items;  -- Safe drop (won't error if doesn't exist)
```

---

## DML (Data Manipulation Language)

### INSERT

**Syntax:**
```sql
INSERT INTO table_name (column1, column2, ...) 
VALUES (value1, value2, ...);
```

**Examples:**
```sql
-- Insert single record
INSERT INTO menu_items (menu_id, name, price, category) 
VALUES (1, 'Pizza', 299.99, 'MAIN_COURSE');

-- Insert multiple records
INSERT INTO menu_items (menu_id, name, price, category) 
VALUES 
    (2, 'Burger', 149.99, 'MAIN_COURSE'),
    (3, 'Ice Cream', 79.99, 'DESSERT');

-- Insert with default values
INSERT INTO tables (table_number, capacity) 
VALUES (1, 4);

-- Insert with NULL
INSERT INTO menu_items (menu_id, name, price, category) 
VALUES (4, 'Salad', 99.99, NULL);
```

### UPDATE

**Syntax:**
```sql
UPDATE table_name 
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

**Examples:**
```sql
-- Update single record
UPDATE menu_items 
SET price = 319.99 
WHERE menu_id = 1;

-- Update multiple columns
UPDATE tables 
SET status = 'BOOKED', capacity = 6 
WHERE table_number = 1;

-- Update multiple records
UPDATE menu_items 
SET price = price * 1.1  -- Increase price by 10%
WHERE category = 'MAIN_COURSE';

-- Always use WHERE clause (or you'll update all rows!)
```

### DELETE

**Syntax:**
```sql
DELETE FROM table_name 
WHERE condition;
```

**Examples:**
```sql
-- Delete specific record
DELETE FROM menu_items 
WHERE menu_id = 4;

-- Delete multiple records
DELETE FROM orders 
WHERE status = 'CANCELLED';

-- Delete all records (be careful!)
DELETE FROM menu_items;  -- Removes all rows, table structure remains

-- Truncate (faster, resets auto-increment)
TRUNCATE TABLE menu_items;
```

---

## Queries and Joins

### SELECT Basics

**Syntax:**
```sql
SELECT column1, column2, ... 
FROM table_name 
WHERE condition 
ORDER BY column 
LIMIT n;
```

**Examples:**
```sql
-- Select all columns
SELECT * FROM menu_items;

-- Select specific columns
SELECT menu_id, name, price FROM menu_items;

-- Select with alias
SELECT menu_id AS id, name AS item_name, price FROM menu_items;

-- Select with WHERE clause
SELECT * FROM menu_items WHERE price > 100;
SELECT * FROM menu_items WHERE category = 'MAIN_COURSE';
SELECT * FROM menu_items WHERE price BETWEEN 100 AND 500;
SELECT * FROM menu_items WHERE name LIKE '%Pizza%';  -- Pattern matching
SELECT * FROM menu_items WHERE category IN ('MAIN_COURSE', 'DESSERT');

-- Select with ORDER BY
SELECT * FROM menu_items ORDER BY price DESC;  -- Highest first
SELECT * FROM menu_items ORDER BY name ASC;    -- Alphabetical

-- Select with LIMIT
SELECT * FROM menu_items ORDER BY price DESC LIMIT 10;  -- Top 10

-- Select distinct values
SELECT DISTINCT category FROM menu_items;
```

### WHERE Clause Operators:
- `=` - Equal
- `!=` or `<>` - Not equal
- `>` `<` `>=` `<=` - Comparison
- `BETWEEN` - Range
- `IN` - List of values
- `LIKE` - Pattern matching (`%` = any chars, `_` = single char)
- `IS NULL` / `IS NOT NULL` - Null checks
- `AND` / `OR` - Logical operators

### JOINs

**INNER JOIN** (Returns matching records):
```sql
SELECT o.order_id, o.customer_id, t.table_number, t.capacity
FROM orders o
INNER JOIN tables t ON o.table_id = t.table_number;
```

**LEFT JOIN** (All from left + matching from right):
```sql
SELECT m.name, oi.quantity, oi.price
FROM menu_items m
LEFT JOIN order_items oi ON m.menu_id = oi.menu_id;
```

**RIGHT JOIN** (All from right + matching from left):
```sql
SELECT oi.order_item_id, m.name
FROM order_items oi
RIGHT JOIN menu_items m ON oi.menu_id = m.menu_id;
```

**FULL OUTER JOIN** (All from both):
```sql
SELECT * 
FROM orders o
FULL OUTER JOIN bills b ON o.order_id = b.order_id;
```

**SELF JOIN** (Join table with itself):
```sql
-- Find orders on same table
SELECT o1.order_id, o2.order_id
FROM orders o1
JOIN orders o2 ON o1.table_id = o2.table_id
WHERE o1.order_id != o2.order_id;
```

### Subqueries

**Subquery in WHERE:**
```sql
-- Find menu items not in any order
SELECT * FROM menu_items
WHERE menu_id NOT IN (
    SELECT DISTINCT menu_id FROM order_items
);
```

**Subquery in SELECT:**
```sql
-- Count orders per customer
SELECT customer_id, 
       (SELECT COUNT(*) FROM orders o2 WHERE o2.customer_id = o1.customer_id) as order_count
FROM orders o1;
```

**EXISTS:**
```sql
-- Find customers who have orders
SELECT * FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

---

## Aggregate Functions

### Common Aggregate Functions:

```sql
-- COUNT - Count rows
SELECT COUNT(*) FROM menu_items;
SELECT COUNT(DISTINCT category) FROM menu_items;

-- SUM - Sum of values
SELECT SUM(total_amount) FROM bills WHERE payment_status = 'PAID';

-- AVG - Average
SELECT AVG(price) FROM menu_items;

-- MIN / MAX
SELECT MIN(price) FROM menu_items;
SELECT MAX(price) FROM menu_items;

-- GROUP BY - Group and aggregate
SELECT category, COUNT(*) as count, AVG(price) as avg_price
FROM menu_items
GROUP BY category;

-- HAVING - Filter groups (after GROUP BY)
SELECT category, AVG(price) as avg_price
FROM menu_items
GROUP BY category
HAVING AVG(price) > 150;
```

### Example: Restaurant Analytics
```sql
-- Total revenue by category
SELECT m.category, 
       SUM(oi.quantity * oi.price) as total_revenue,
       COUNT(DISTINCT oi.order_id) as order_count
FROM order_items oi
JOIN menu_items m ON oi.menu_id = m.menu_id
JOIN orders o ON oi.order_id = o.order_id
WHERE o.status = 'PAID'
GROUP BY m.category
ORDER BY total_revenue DESC;

-- Average order value
SELECT AVG(total_amount) as avg_order_value
FROM bills
WHERE payment_status = 'PAID';

-- Most popular items
SELECT m.name, SUM(oi.quantity) as total_quantity
FROM order_items oi
JOIN menu_items m ON oi.menu_id = m.menu_id
GROUP BY m.menu_id, m.name
ORDER BY total_quantity DESC
LIMIT 10;
```

---

## Constraints

### Types of Constraints:

**PRIMARY KEY:**
```sql
CREATE TABLE menu_items (
    menu_id INT PRIMARY KEY,
    ...
);
```

**FOREIGN KEY:**
```sql
CREATE TABLE order_items (
    order_id INT,
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);
```

**NOT NULL:**
```sql
CREATE TABLE menu_items (
    name VARCHAR(100) NOT NULL,
    ...
);
```

**UNIQUE:**
```sql
CREATE TABLE bills (
    order_id INT UNIQUE,
    ...
);
```

**CHECK:**
```sql
CREATE TABLE menu_items (
    price DECIMAL(10, 2) CHECK (price > 0),
    ...
);
```

**DEFAULT:**
```sql
CREATE TABLE tables (
    status VARCHAR(20) DEFAULT 'FREE',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Indexes

### Purpose:
Improve query performance by creating index on columns.

### CREATE INDEX:
```sql
-- Single column index
CREATE INDEX idx_category ON menu_items(category);

-- Composite index
CREATE INDEX idx_order_customer ON orders(customer_id, status);

-- Unique index
CREATE UNIQUE INDEX idx_menu_name ON menu_items(name);
```

### When to Use Indexes:
- Frequently queried columns
- Foreign keys
- Columns used in WHERE, JOIN, ORDER BY
- **Don't over-index** (slows down INSERT/UPDATE)

---

## Common SQL Interview Questions & Solutions

### 1. Find the second highest price
```sql
SELECT MAX(price) 
FROM menu_items 
WHERE price < (SELECT MAX(price) FROM menu_items);
```

### 2. Find duplicate records
```sql
SELECT name, COUNT(*) 
FROM menu_items 
GROUP BY name 
HAVING COUNT(*) > 1;
```

### 3. Delete duplicates (keep one)
```sql
DELETE m1 FROM menu_items m1
INNER JOIN menu_items m2 
WHERE m1.menu_id > m2.menu_id 
AND m1.name = m2.name;
```

### 4. Find customers who ordered more than 3 times
```sql
SELECT customer_id, COUNT(*) as order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 3;
```

### 5. Find items never ordered
```sql
SELECT m.*
FROM menu_items m
LEFT JOIN order_items oi ON m.menu_id = oi.menu_id
WHERE oi.menu_id IS NULL;
```

### 6. Get running total
```sql
SELECT order_id, total_amount,
       (SELECT SUM(total_amount) 
        FROM bills b2 
        WHERE b2.bill_id <= b1.bill_id) as running_total
FROM bills b1
ORDER BY bill_id;
```

### 7. Find nth highest salary/item
```sql
-- 3rd highest price
SELECT price 
FROM menu_items 
ORDER BY price DESC 
LIMIT 1 OFFSET 2;
```

### 8. Date operations
```sql
-- Orders today
SELECT * FROM orders 
WHERE DATE(created_at) = CURDATE();

-- Orders last 7 days
SELECT * FROM orders 
WHERE created_at >= DATE_SUB(NOW(), INTERVAL 7 DAY);

-- Orders this month
SELECT * FROM orders 
WHERE MONTH(created_at) = MONTH(NOW()) 
AND YEAR(created_at) = YEAR(NOW());
```

### 9. Case statements
```sql
SELECT name, price,
       CASE 
           WHEN price > 200 THEN 'Expensive'
           WHEN price > 100 THEN 'Moderate'
           ELSE 'Cheap'
       END as price_category
FROM menu_items;
```

### 10. Window Functions (MySQL 8.0+)
```sql
-- Rank items by price within category
SELECT name, category, price,
       RANK() OVER (PARTITION BY category ORDER BY price DESC) as rank_in_category
FROM menu_items;
```

---

## Practice Questions for Interview

1. Write a query to find the total revenue for each day.
2. Find the customer who spent the most money.
3. List all menu items with their total sales quantity.
4. Find tables that are booked for more than 2 hours.
5. Get the average order value by month.
6. Find menu items that are ordered together frequently.
7. List customers who haven't placed an order in the last 30 days.
8. Calculate the profit margin for each category.
9. Find peak hours (most orders placed).
10. Get the most popular combination of menu items.

---

## Tips for SQL Interviews

1. **Always think about performance** - Use indexes, avoid SELECT *
2. **Handle NULLs properly** - Use IS NULL/IS NOT NULL
3. **Understand JOINs** - Know when to use INNER vs LEFT JOIN
4. **Use aggregate functions** - COUNT, SUM, AVG, MIN, MAX
5. **Group by and Having** - Understand the difference
6. **Subqueries** - Know when to use correlated vs non-correlated
7. **Indexes** - Understand when and why to create them
8. **Normalization** - Understand 1NF, 2NF, 3NF
9. **Transactions** - ACID properties
10. **Optimization** - Explain execution plans, query optimization

---

## Common SQL Mistakes to Avoid

1. Using SELECT * in production
2. Forgetting WHERE clause in UPDATE/DELETE
3. Not using transactions for related operations
4. Missing indexes on frequently queried columns
5. Using LIKE '%pattern%' (can't use index)
6. Not handling NULL values properly
7. Overusing subqueries when JOINs would work
8. Not considering NULLs in aggregate functions

