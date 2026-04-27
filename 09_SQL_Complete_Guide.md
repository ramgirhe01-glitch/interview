# 🗄️ SQL Complete Interview Guide — SDET Focus
### Ram Girhe | 3 YOE | Exercises + Interview Questions + Deep Concepts

---

# PART 1: SQL FUNDAMENTALS

---

## 1. Basic Queries

```sql
-- SELECT: Retrieve data
SELECT * FROM users;
SELECT name, email FROM users;
SELECT DISTINCT city FROM users;

-- WHERE: Filter rows
SELECT * FROM users WHERE age > 25;
SELECT * FROM users WHERE city = 'Pune' AND age >= 18;
SELECT * FROM users WHERE city IN ('Pune', 'Mumbai', 'Delhi');
SELECT * FROM users WHERE name LIKE 'Ram%';        -- starts with Ram
SELECT * FROM users WHERE name LIKE '%girhe';       -- ends with girhe
SELECT * FROM users WHERE name LIKE '%am%';         -- contains 'am'
SELECT * FROM users WHERE email IS NULL;
SELECT * FROM users WHERE email IS NOT NULL;
SELECT * FROM users WHERE age BETWEEN 18 AND 30;

-- ORDER BY: Sort results
SELECT * FROM users ORDER BY name ASC;              -- ascending (default)
SELECT * FROM users ORDER BY age DESC;              -- descending
SELECT * FROM users ORDER BY city ASC, age DESC;    -- multiple columns

-- LIMIT / TOP
SELECT * FROM users LIMIT 10;                       -- MySQL/PostgreSQL
SELECT TOP 10 * FROM users;                         -- SQL Server
SELECT * FROM users LIMIT 10 OFFSET 20;             -- Pagination: skip 20, get 10

-- ALIASES
SELECT u.name AS user_name, u.email AS user_email
FROM users u;
```

---

## 2. Aggregate Functions

```sql
SELECT COUNT(*) FROM users;                          -- Total rows
SELECT COUNT(DISTINCT city) FROM users;              -- Unique cities
SELECT SUM(salary) FROM employees;                   -- Total salary
SELECT AVG(salary) FROM employees;                   -- Average salary
SELECT MIN(salary), MAX(salary) FROM employees;      -- Min and Max
SELECT ROUND(AVG(salary), 2) FROM employees;         -- Round to 2 decimals

-- GROUP BY: Group rows for aggregation
SELECT city, COUNT(*) AS user_count
FROM users
GROUP BY city;

SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC;

-- HAVING: Filter groups (WHERE filters rows, HAVING filters groups)
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;

-- SQL Execution Order:
-- FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

---

## 3. JOINs (Most Asked Topic!)

### Sample Tables:
```
employees:                    departments:
| id | name  | dept_id |     | id | dept_name |
|----|-------|---------|     |----|-----------|
| 1  | Ram   | 10      |     | 10 | QA        |
| 2  | Shyam | 20      |     | 20 | Dev       |
| 3  | Sita  | 10      |     | 30 | HR        |
| 4  | Gita  | NULL    |
```

```sql
-- INNER JOIN: Only matching rows from both tables
SELECT e.name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;
-- Result: Ram-QA, Shyam-Dev, Sita-QA (Gita excluded: no dept, HR excluded: no employee)

-- LEFT JOIN: All rows from left + matching from right
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;
-- Result: Ram-QA, Shyam-Dev, Sita-QA, Gita-NULL

-- RIGHT JOIN: All rows from right + matching from left
SELECT e.name, d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.id;
-- Result: Ram-QA, Shyam-Dev, Sita-QA, NULL-HR

-- FULL OUTER JOIN: All rows from both tables
SELECT e.name, d.dept_name
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.id;
-- Result: Ram-QA, Shyam-Dev, Sita-QA, Gita-NULL, NULL-HR

-- CROSS JOIN: Every combination (Cartesian product)
SELECT e.name, d.dept_name
FROM employees e
CROSS JOIN departments d;
-- Result: 4 × 3 = 12 rows

-- SELF JOIN: Join table with itself
-- Find employees with same department
SELECT e1.name, e2.name, e1.dept_id
FROM employees e1
INNER JOIN employees e2 ON e1.dept_id = e2.dept_id
WHERE e1.id < e2.id;
-- Result: Ram-Sita (both in dept 10)
```

### Visual JOIN Guide:
```
INNER JOIN:      LEFT JOIN:       RIGHT JOIN:      FULL OUTER:
  ┌───┐           ┌───┐             ┌───┐           ┌───┐
  │ A∩B│           │A  ∩B│           │A∩  B│          │A  ∪  B│
  └───┘           └───┘             └───┘           └───┘
Only overlap    All A + overlap   All B + overlap  Everything
```

---

## 4. Subqueries

```sql
-- Subquery in WHERE
SELECT name FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Subquery in FROM (derived table)
SELECT dept_name, avg_sal
FROM (
    SELECT dept_id, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY dept_id
) AS dept_avg
INNER JOIN departments d ON dept_avg.dept_id = d.id;

-- EXISTS (check if subquery returns rows)
SELECT name FROM departments d
WHERE EXISTS (
    SELECT 1 FROM employees e WHERE e.dept_id = d.id
);

-- IN with subquery
SELECT name FROM employees
WHERE dept_id IN (SELECT id FROM departments WHERE dept_name LIKE '%QA%');

-- Correlated subquery (references outer query)
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary) FROM employees WHERE dept_id = e.dept_id
);
```

---

## 5. Window Functions (Advanced — asked in senior interviews)

```sql
-- ROW_NUMBER: Assign sequential number
SELECT name, salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- RANK: Same value = same rank, gaps after ties
-- DENSE_RANK: Same value = same rank, NO gaps
SELECT name, salary,
    RANK() OVER (ORDER BY salary DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;
-- salary: 100, 100, 80
-- RANK:   1,   1,   3
-- DENSE:  1,   1,   2

-- PARTITION BY: Window within groups
SELECT name, department, salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;
-- Ranks employees within each department separately

-- Find Nth highest salary per department
SELECT * FROM (
    SELECT name, department, salary,
        DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rnk
    FROM employees
) ranked
WHERE rnk = 2;  -- 2nd highest in each department

-- Running total
SELECT name, salary,
    SUM(salary) OVER (ORDER BY id) AS running_total
FROM employees;

-- LAG / LEAD: Access previous/next row
SELECT name, salary,
    LAG(salary, 1) OVER (ORDER BY id) AS prev_salary,
    LEAD(salary, 1) OVER (ORDER BY id) AS next_salary,
    salary - LAG(salary, 1) OVER (ORDER BY id) AS salary_diff
FROM employees;
```

---

## 6. Data Modification (DML)

```sql
-- INSERT
INSERT INTO users (name, email, city) VALUES ('Ram', 'ram@test.com', 'Pune');
INSERT INTO users (name, email) VALUES
    ('Shyam', 'shyam@test.com'),
    ('Sita', 'sita@test.com');

-- UPDATE
UPDATE users SET city = 'Mumbai' WHERE name = 'Ram';
UPDATE employees SET salary = salary * 1.1 WHERE department = 'QA';

-- DELETE
DELETE FROM users WHERE id = 5;
DELETE FROM users WHERE created_at < '2024-01-01';

-- TRUNCATE vs DELETE
-- TRUNCATE: Remove ALL rows, faster, cannot rollback, resets auto-increment
-- DELETE: Remove specific rows, slower, can rollback, keeps auto-increment
TRUNCATE TABLE temp_data;
```

---

## 7. DDL (Data Definition Language)

```sql
-- CREATE TABLE
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    age INT CHECK (age >= 0 AND age <= 150),
    city VARCHAR(50) DEFAULT 'Unknown',
    dept_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (dept_id) REFERENCES departments(id)
);

-- ALTER TABLE
ALTER TABLE users ADD COLUMN phone VARCHAR(15);
ALTER TABLE users DROP COLUMN phone;
ALTER TABLE users MODIFY COLUMN name VARCHAR(200);
ALTER TABLE users ADD INDEX idx_email (email);

-- DROP
DROP TABLE users;        -- Delete table and data
DROP TABLE IF EXISTS temp;

-- CREATE INDEX
CREATE INDEX idx_city ON users(city);
CREATE UNIQUE INDEX idx_email ON users(email);
```

---

## 8. String Functions

```sql
SELECT UPPER('hello');                   -- HELLO
SELECT LOWER('HELLO');                   -- hello
SELECT LENGTH('Hello');                  -- 5
SELECT TRIM('  Hello  ');               -- 'Hello'
SELECT SUBSTRING('Hello World', 1, 5);   -- Hello
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;
SELECT REPLACE('Hello World', 'World', 'SQL');  -- Hello SQL
SELECT LEFT('Hello', 3);                -- Hel
SELECT RIGHT('Hello', 3);               -- llo
SELECT REVERSE('Hello');                -- olleH
SELECT COALESCE(email, phone, 'N/A');   -- First non-null value
SELECT IFNULL(email, 'no-email');       -- MySQL
SELECT ISNULL(email, 'no-email');       -- SQL Server
```

---

## 9. Date Functions

```sql
SELECT NOW();                            -- Current datetime
SELECT CURDATE();                        -- Current date
SELECT YEAR('2024-06-15');              -- 2024
SELECT MONTH('2024-06-15');             -- 6
SELECT DAY('2024-06-15');               -- 15
SELECT DATEDIFF('2024-12-31', '2024-01-01');  -- 365
SELECT DATE_ADD('2024-01-01', INTERVAL 30 DAY);  -- 2024-01-31
SELECT DATE_FORMAT(NOW(), '%Y-%m-%d %H:%i:%s');  -- Formatted

-- Filter by date
SELECT * FROM orders WHERE order_date >= '2024-01-01';
SELECT * FROM orders WHERE YEAR(order_date) = 2024;
```

---

# PART 2: SQL EXERCISES

---

### Setup: Assume these tables:
```sql
-- employees(id, name, department, salary, manager_id, hire_date)
-- departments(id, name, location)
-- orders(id, customer_id, amount, order_date, status)
-- customers(id, name, email, city)
```

### E1: Find 2nd highest salary
```sql
-- Method 1: Subquery
SELECT MAX(salary) FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

-- Method 2: DENSE_RANK
SELECT salary FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) ranked WHERE rnk = 2;

-- Method 3: LIMIT + OFFSET
SELECT DISTINCT salary FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

### E2: Find Nth highest salary
```sql
-- Generic: Find Nth highest
SELECT DISTINCT salary FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET N-1;  -- Replace N with the number

-- Using DENSE_RANK
SELECT salary FROM (
    SELECT DISTINCT salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) ranked WHERE rnk = N;
```

### E3: Find duplicate emails
```sql
SELECT email, COUNT(*) AS count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

### E4: Employees earning more than their manager
```sql
SELECT e.name AS employee, e.salary, m.name AS manager, m.salary AS mgr_salary
FROM employees e
INNER JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

### E5: Department with highest average salary
```sql
SELECT d.name, AVG(e.salary) AS avg_salary
FROM employees e
INNER JOIN departments d ON e.department = d.id
GROUP BY d.name
ORDER BY avg_salary DESC
LIMIT 1;
```

### E6: Customers who never ordered
```sql
-- Method 1: LEFT JOIN
SELECT c.name
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL;

-- Method 2: NOT IN
SELECT name FROM customers
WHERE id NOT IN (SELECT DISTINCT customer_id FROM orders);

-- Method 3: NOT EXISTS
SELECT name FROM customers c
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);
```

### E7: Running total of orders by date
```sql
SELECT order_date, amount,
    SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;
```

### E8: Find consecutive dates with orders
```sql
SELECT DISTINCT o1.order_date
FROM orders o1
INNER JOIN orders o2 ON DATEDIFF(o2.order_date, o1.order_date) = 1;
```

### E9: Pivot: Count orders by status per month
```sql
SELECT
    DATE_FORMAT(order_date, '%Y-%m') AS month,
    SUM(CASE WHEN status = 'COMPLETED' THEN 1 ELSE 0 END) AS completed,
    SUM(CASE WHEN status = 'PENDING' THEN 1 ELSE 0 END) AS pending,
    SUM(CASE WHEN status = 'CANCELLED' THEN 1 ELSE 0 END) AS cancelled
FROM orders
GROUP BY DATE_FORMAT(order_date, '%Y-%m')
ORDER BY month;
```

### E10: Delete duplicate rows keeping one
```sql
-- Keep lowest id for each duplicate email
DELETE FROM users
WHERE id NOT IN (
    SELECT MIN(id) FROM users GROUP BY email
);

-- MySQL workaround (can't use same table in subquery)
DELETE u1 FROM users u1
INNER JOIN users u2
WHERE u1.id > u2.id AND u1.email = u2.email;
```

---

# PART 3: SQL FOR SDET — PRACTICAL USAGE

---

### Verify API created data in database
```sql
-- After POST /users with name "Ram"
SELECT * FROM users WHERE name = 'Ram' ORDER BY created_at DESC LIMIT 1;

-- Validate all fields match API response
SELECT id, name, email, city, created_at
FROM users
WHERE id = 12345;  -- Use ID from API response
```

### Verify data isolation (multi-tenant)
```sql
-- Tenant A should NOT see Tenant B's data
SELECT * FROM agents WHERE tenant_id = 'tenant_b' AND created_by = 'tenant_a_user';
-- Should return 0 rows
```

### Check database state before/after test
```sql
-- Before test: count records
SELECT COUNT(*) FROM orders WHERE customer_id = 100;

-- After test: verify count increased
SELECT COUNT(*) FROM orders WHERE customer_id = 100;
-- Expected: previous_count + 1
```

### Performance queries for testing
```sql
-- Slow query detection
SELECT * FROM information_schema.processlist WHERE time > 5;

-- Table size
SELECT table_name, table_rows, data_length / 1024 / 1024 AS size_mb
FROM information_schema.tables
WHERE table_schema = 'my_database';

-- Index usage
SHOW INDEX FROM users;
EXPLAIN SELECT * FROM users WHERE email = 'ram@test.com';
```

---

# PART 4: TOP 30 SQL INTERVIEW QUESTIONS

---

| # | Question | Key Answer |
|---|----------|------------|
| 1 | What is the difference between WHERE and HAVING? | WHERE filters rows before grouping. HAVING filters groups after GROUP BY |
| 2 | What is the difference between INNER JOIN and LEFT JOIN? | INNER: only matching. LEFT: all left + matching right (NULL if no match) |
| 3 | What is a primary key? | Unique + NOT NULL. One per table. Identifies each row |
| 4 | What is a foreign key? | References primary key of another table. Enforces referential integrity |
| 5 | What is normalization? | Organize data to reduce redundancy. 1NF→2NF→3NF→BCNF |
| 6 | What is an index? | Data structure (B-tree) to speed up queries. Trade-off: faster reads, slower writes |
| 7 | Clustered vs Non-clustered index? | Clustered: sorts actual data (1 per table, usually PK). Non-clustered: separate structure |
| 8 | What is a view? | Virtual table from a query. Doesn't store data. Used for abstraction/security |
| 9 | What is a stored procedure? | Precompiled SQL block. Accepts parameters, returns results |
| 10 | What is a trigger? | Auto-executed SQL on INSERT/UPDATE/DELETE |
| 11 | UNION vs UNION ALL? | UNION: removes duplicates. UNION ALL: keeps all (faster) |
| 12 | DELETE vs TRUNCATE vs DROP? | DELETE: row-by-row, can rollback. TRUNCATE: all rows, fast. DROP: removes table |
| 13 | What is ACID? | Atomicity, Consistency, Isolation, Durability — transaction properties |
| 14 | What is a transaction? | Group of operations that succeed/fail together. BEGIN→COMMIT/ROLLBACK |
| 15 | What are window functions? | Calculations across related rows: ROW_NUMBER, RANK, SUM OVER, LAG/LEAD |
| 16 | What is a CTE (Common Table Expression)? | `WITH cte AS (SELECT...) SELECT FROM cte` — readable subquery |
| 17 | What is a correlated subquery? | Subquery that references the outer query. Executes once per outer row |
| 18 | How to find duplicates? | `GROUP BY column HAVING COUNT(*) > 1` |
| 19 | How to find Nth highest salary? | `DENSE_RANK() OVER (ORDER BY salary DESC)` or `LIMIT 1 OFFSET N-1` |
| 20 | What is SQL injection? | Malicious SQL via user input. Prevention: parameterized queries |
| 21 | What is EXPLAIN? | Shows query execution plan: index usage, row scans, cost |
| 22 | How to optimize a slow query? | Add indexes, avoid SELECT *, use EXPLAIN, reduce subqueries, use JOINs |
| 23 | What is denormalization? | Intentionally adding redundancy for read performance (e.g., in reporting) |
| 24 | NULL handling? | NULL ≠ empty. Use IS NULL/IS NOT NULL. NULL in math = NULL |
| 25 | COALESCE vs IFNULL? | COALESCE: multiple args, standard SQL. IFNULL: two args, MySQL-specific |
| 26 | What is a composite key? | Primary key with multiple columns |
| 27 | GROUP BY rules? | Every non-aggregated column in SELECT must be in GROUP BY |
| 28 | What is a self-join? | Table joined with itself. Used for hierarchies (employee-manager) |
| 29 | How to handle pagination? | `LIMIT n OFFSET m` or cursor-based with WHERE id > last_id |
| 30 | What is CASE WHEN? | SQL's if-else: `CASE WHEN x>5 THEN 'High' ELSE 'Low' END` |

---

*SQL is tested in 90% of SDET interviews. Master JOINs, GROUP BY, Window functions, and subqueries!*

