# MySQL Interview Questions & Answers

## 1. What is MySQL?

**Answer:** MySQL is an open-source Relational Database Management System (RDBMS) that uses Structured Query Language (SQL). It is owned by Oracle Corporation and is one of the most popular databases for web applications. MySQL stores data in tables with rows and columns, supports ACID properties, and is known for its reliability, scalability, and ease of use.

---

## 2. What are the differences between MySQL and SQL?

**Answer:**
- **SQL (Structured Query Language):** A standard language used to manage and manipulate relational databases.
- **MySQL:** A database management system that uses SQL to perform database operations.

SQL is the language, while MySQL is the software that implements SQL.

---

## 3. What are the different storage engines in MySQL?

**Answer:** MySQL supports multiple storage engines:
- **InnoDB:** Default engine, supports transactions, foreign keys, and ACID compliance
- **MyISAM:** Older engine, faster for read operations but doesn't support transactions
- **MEMORY:** Stores data in RAM for fast access
- **CSV:** Stores data in CSV format
- **ARCHIVE:** Optimized for storing large amounts of historical data
- **BLACKHOLE:** Accepts data but doesn't store it (used for replication)

---

## 4. What are the differences between InnoDB and MyISAM?

**Answer:**

| Feature | InnoDB | MyISAM |
|---------|--------|--------|
| Transactions | Supported | Not supported |
| Foreign Keys | Supported | Not supported |
| Row-level locking | Yes | Table-level locking |
| ACID compliance | Yes | No |
| Crash recovery | Better | Poor |
| Full-text search | Yes (5.6+) | Yes |
| Performance | Better for write-heavy operations | Better for read-heavy operations |

---

## 5. What is a Primary Key?

**Answer:** A Primary Key is a column or set of columns that uniquely identifies each row in a table.

Characteristics:
- Must contain unique values
- Cannot contain NULL values
- A table can have only one primary key
- Automatically creates a clustered index

Example:
```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50),
    email VARCHAR(100)
);
```

---

## 6. What is a Foreign Key?

**Answer:** A Foreign Key is a column or set of columns that creates a link between two tables. It references the primary key of another table and enforces referential integrity.

Example:
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    user_id INT,
    order_date DATE,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

---

## 7. What are the different types of JOINs in MySQL?

**Answer:**

1. **INNER JOIN:** Returns records that have matching values in both tables
2. **LEFT JOIN (LEFT OUTER JOIN):** Returns all records from left table and matching records from right table
3. **RIGHT JOIN (RIGHT OUTER JOIN):** Returns all records from right table and matching records from left table
4. **CROSS JOIN:** Returns Cartesian product of both tables
5. **SELF JOIN:** A table joined with itself

Example:
```sql
-- INNER JOIN
SELECT users.username, orders.order_id
FROM users
INNER JOIN orders ON users.user_id = orders.user_id;

-- LEFT JOIN
SELECT users.username, orders.order_id
FROM users
LEFT JOIN orders ON users.user_id = orders.user_id;
```

---

## 8. What is the difference between WHERE and HAVING clause?

**Answer:**

| WHERE | HAVING |
|-------|--------|
| Filters rows before grouping | Filters groups after grouping |
| Cannot use aggregate functions | Can use aggregate functions |
| Used with SELECT, UPDATE, DELETE | Used with GROUP BY |
| Applies to individual rows | Applies to groups |

Example:
```sql
-- WHERE: Filters before grouping
SELECT department, COUNT(*)
FROM employees
WHERE salary > 50000
GROUP BY department;

-- HAVING: Filters after grouping
SELECT department, COUNT(*)
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

---

## 9. What is an Index and what are its types?

**Answer:** An Index is a data structure that improves the speed of data retrieval operations on a table.

**Types of Indexes:**

1. **Primary Index:** Automatically created on primary key
2. **Unique Index:** Ensures all values in the column are unique
3. **Composite Index:** Index on multiple columns
4. **Full-text Index:** Used for full-text searches
5. **Spatial Index:** For spatial data types

Example:
```sql
-- Create index
CREATE INDEX idx_email ON users(email);

-- Composite index
CREATE INDEX idx_name_email ON users(first_name, last_name);

-- Unique index
CREATE UNIQUE INDEX idx_username ON users(username);
```

---

## 10. What are the advantages and disadvantages of Indexes?

**Answer:**

**Advantages:**
- Faster data retrieval (SELECT queries)
- Speeds up WHERE, ORDER BY, and JOIN operations
- Improves query performance significantly

**Disadvantages:**
- Slows down INSERT, UPDATE, and DELETE operations
- Requires additional disk space
- Needs maintenance and updates

---

## 11. What is Normalization?

**Answer:** Normalization is the process of organizing data in a database to reduce redundancy and improve data integrity. It involves dividing large tables into smaller tables and defining relationships between them.

**Normal Forms:**
- **1NF:** Each column contains atomic values, no repeating groups
- **2NF:** 1NF + No partial dependencies
- **3NF:** 2NF + No transitive dependencies
- **BCNF:** 3NF + Every determinant is a candidate key

---

## 12. What is Denormalization?

**Answer:** Denormalization is the process of adding redundant data to a normalized database to improve read performance. It's the opposite of normalization and is used when read performance is more critical than write performance.

Use cases:
- Data warehousing
- Reporting systems
- Read-heavy applications

---

## 13. What are ACID properties?

**Answer:** ACID stands for:

- **Atomicity:** All operations in a transaction succeed or all fail
- **Consistency:** Database remains in a consistent state before and after transaction
- **Isolation:** Concurrent transactions don't interfere with each other
- **Durability:** Once committed, changes are permanent

Example:
```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
COMMIT;
```

---

## 14. What is a Transaction?

**Answer:** A Transaction is a sequence of one or more SQL operations treated as a single unit of work. It follows ACID properties.

Commands:
- `START TRANSACTION` or `BEGIN`: Starts a transaction
- `COMMIT`: Saves all changes permanently
- `ROLLBACK`: Undoes all changes since transaction started
- `SAVEPOINT`: Creates a point within transaction to rollback to

---

## 15. What are constraints in MySQL?

**Answer:** Constraints are rules enforced on data columns to maintain data integrity.

Types:
- **NOT NULL:** Column cannot have NULL values
- **UNIQUE:** All values must be unique
- **PRIMARY KEY:** NOT NULL + UNIQUE
- **FOREIGN KEY:** Links two tables
- **CHECK:** Validates data based on condition
- **DEFAULT:** Sets default value

Example:
```sql
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    age INT CHECK (age >= 18),
    department VARCHAR(50) DEFAULT 'IT'
);
```

---

## 16. What is the difference between CHAR and VARCHAR?

**Answer:**

| CHAR | VARCHAR |
|------|---------|
| Fixed-length | Variable-length |
| Pads with spaces | No padding |
| Faster for fixed-size data | More storage efficient |
| Max 255 characters | Max 65,535 characters |
| Uses exact specified length | Uses actual length + 1 or 2 bytes |

Example:
```sql
CHAR(10) -- Always uses 10 bytes
VARCHAR(10) -- Uses 1-11 bytes depending on content
```

---

## 17. What is the difference between DELETE, TRUNCATE, and DROP?

**Answer:**

| Feature | DELETE | TRUNCATE | DROP |
|---------|--------|----------|------|
| Type | DML | DDL | DDL |
| Removes | Rows | All rows | Table structure |
| WHERE clause | Yes | No | No |
| Rollback | Possible | Not possible (InnoDB can) | Not possible |
| Triggers | Fired | Not fired | Not fired |
| Speed | Slower | Faster | Fastest |
| Identity reset | No | Yes | Yes |

Example:
```sql
DELETE FROM users WHERE age < 18; -- Removes specific rows
TRUNCATE TABLE users; -- Removes all rows
DROP TABLE users; -- Removes table completely
```

---

## 18. What are Views in MySQL?

**Answer:** A View is a virtual table based on a SELECT query. It doesn't store data physically but provides a way to simplify complex queries.

**Advantages:**
- Simplifies complex queries
- Enhances security by restricting access
- Provides data abstraction
- Maintains logical data independence

Example:
```sql
-- Create view
CREATE VIEW active_users AS
SELECT user_id, username, email
FROM users
WHERE status = 'active';

-- Use view
SELECT * FROM active_users;
```

---

## 19. What is a Stored Procedure?

**Answer:** A Stored Procedure is a prepared SQL code that can be saved and reused. It's stored in the database and can accept parameters.

**Advantages:**
- Reduces network traffic
- Improves performance (pre-compiled)
- Enhances security
- Code reusability

Example:
```sql
DELIMITER //
CREATE PROCEDURE GetUserById(IN userId INT)
BEGIN
    SELECT * FROM users WHERE user_id = userId;
END //
DELIMITER ;

-- Call procedure
CALL GetUserById(5);
```

---

## 20. What is the difference between Stored Procedure and Function?

**Answer:**

| Stored Procedure | Function |
|-----------------|----------|
| May or may not return value | Must return a value |
| Can use CALL statement | Used in SQL expressions |
| Can return multiple values | Returns single value |
| Can have transactions | Cannot have transactions |
| Can call functions | Cannot call procedures |
| Used for business logic | Used for calculations |

Example:
```sql
-- Function
CREATE FUNCTION calculate_tax(amount DECIMAL(10,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    RETURN amount * 0.15;
END;

-- Use in query
SELECT order_id, total, calculate_tax(total) AS tax FROM orders;
```

---

## 21. What are Triggers in MySQL?

**Answer:** A Trigger is a stored program that automatically executes in response to specific events (INSERT, UPDATE, DELETE) on a table.

**Types:**
- **BEFORE triggers:** Execute before the event
- **AFTER triggers:** Execute after the event

Example:
```sql
CREATE TRIGGER before_employee_insert
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    SET NEW.created_at = NOW();
END;

CREATE TRIGGER after_order_delete
AFTER DELETE ON orders
FOR EACH ROW
BEGIN
    INSERT INTO order_audit (order_id, deleted_at)
    VALUES (OLD.order_id, NOW());
END;
```

---

## 22. What is the difference between UNION and UNION ALL?

**Answer:**

| UNION | UNION ALL |
|-------|-----------|
| Removes duplicate rows | Keeps all rows including duplicates |
| Slower (sorting required) | Faster |
| Distinct results | All results |

Example:
```sql
-- UNION: Returns unique rows
SELECT name FROM employees
UNION
SELECT name FROM contractors;

-- UNION ALL: Returns all rows
SELECT name FROM employees
UNION ALL
SELECT name FROM contractors;
```

---

## 23. What are Aggregate Functions in MySQL?

**Answer:** Aggregate functions perform calculations on a set of values and return a single value.

Common aggregate functions:
- **COUNT():** Returns number of rows
- **SUM():** Returns sum of values
- **AVG():** Returns average value
- **MIN():** Returns minimum value
- **MAX():** Returns maximum value
- **GROUP_CONCAT():** Concatenates values from multiple rows

Example:
```sql
SELECT
    COUNT(*) as total_orders,
    SUM(amount) as total_amount,
    AVG(amount) as average_amount,
    MIN(amount) as min_amount,
    MAX(amount) as max_amount
FROM orders;
```

---

## 24. What is a Subquery?

**Answer:** A Subquery (inner query) is a query nested inside another query. It can be used in SELECT, FROM, WHERE, or HAVING clauses.

**Types:**
- **Single-row subquery:** Returns one row
- **Multiple-row subquery:** Returns multiple rows
- **Correlated subquery:** References outer query

Example:
```sql
-- Subquery in WHERE
SELECT name FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Subquery in FROM
SELECT dept_name, avg_salary
FROM (
    SELECT department_id, AVG(salary) as avg_salary
    FROM employees
    GROUP BY department_id
) as dept_avg;

-- Correlated subquery
SELECT e1.name, e1.salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id
);
```

---

## 25. What is the difference between IN and EXISTS?

**Answer:**

| IN | EXISTS |
|----|--------|
| Compares values | Checks for existence |
| Returns when match found | Returns TRUE/FALSE |
| Good for small subqueries | Better for large subqueries |
| Evaluates subquery first | Short-circuits on first match |

Example:
```sql
-- IN
SELECT * FROM employees
WHERE department_id IN (SELECT id FROM departments WHERE location = 'NY');

-- EXISTS
SELECT * FROM employees e
WHERE EXISTS (
    SELECT 1 FROM departments d
    WHERE d.id = e.department_id AND d.location = 'NY'
);
```

---

## 26. What is AUTO_INCREMENT in MySQL?

**Answer:** AUTO_INCREMENT is an attribute used to automatically generate unique sequential numbers for a column (typically primary key).

Characteristics:
- Starts from 1 by default
- Increments by 1 for each new row
- Only one AUTO_INCREMENT column per table
- Must be indexed

Example:
```sql
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10,2)
);

-- Insert without specifying ID
INSERT INTO products (product_name, price) VALUES ('Laptop', 999.99);
-- product_id will be automatically assigned

-- Alter starting value
ALTER TABLE products AUTO_INCREMENT = 1000;
```

---

## 27. How do you find duplicate records in a table?

**Answer:**

```sql
-- Find duplicates
SELECT email, COUNT(*) as count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Find all duplicate records
SELECT *
FROM users
WHERE email IN (
    SELECT email
    FROM users
    GROUP BY email
    HAVING COUNT(*) > 1
);

-- Find duplicate with all details
SELECT u1.*
FROM users u1
INNER JOIN (
    SELECT email
    FROM users
    GROUP BY email
    HAVING COUNT(*) > 1
) u2 ON u1.email = u2.email
ORDER BY u1.email;
```

---

## 28. What is the difference between RANK(), DENSE_RANK(), and ROW_NUMBER()?

**Answer:**

| Function | Behavior | Example (Scores: 95,95,90,85) |
|----------|----------|-------------------------------|
| ROW_NUMBER() | Sequential numbers | 1,2,3,4 |
| RANK() | Same rank for ties, skips next | 1,1,3,4 |
| DENSE_RANK() | Same rank for ties, no skip | 1,1,2,3 |

Example:
```sql
SELECT
    name,
    score,
    ROW_NUMBER() OVER (ORDER BY score DESC) as row_num,
    RANK() OVER (ORDER BY score DESC) as rank_val,
    DENSE_RANK() OVER (ORDER BY score DESC) as dense_rank_val
FROM students;
```

---

## 29. What is the difference between NOW() and CURRENT_DATE()?

**Answer:**

- **NOW():** Returns current date and time (YYYY-MM-DD HH:MM:SS)
- **CURRENT_DATE():** Returns only current date (YYYY-MM-DD)
- **CURRENT_TIME():** Returns only current time (HH:MM:SS)
- **CURDATE():** Same as CURRENT_DATE()
- **CURTIME():** Same as CURRENT_TIME()

Example:
```sql
SELECT
    NOW() as current_datetime,        -- 2025-10-01 14:30:45
    CURRENT_DATE() as current_date,   -- 2025-10-01
    CURRENT_TIME() as current_time;   -- 14:30:45
```

---

## 30. How do you handle NULL values in MySQL?

**Answer:**

Functions for NULL handling:
- **IS NULL / IS NOT NULL:** Check for NULL
- **IFNULL(expr1, expr2):** Returns expr2 if expr1 is NULL
- **COALESCE(expr1, expr2, ...):** Returns first non-NULL value
- **NULLIF(expr1, expr2):** Returns NULL if equal, else expr1

Example:
```sql
-- Check NULL
SELECT * FROM users WHERE phone IS NULL;

-- Replace NULL
SELECT name, IFNULL(phone, 'Not provided') as phone FROM users;

-- COALESCE
SELECT name, COALESCE(mobile, phone, email, 'No contact') as contact
FROM users;

-- NULLIF
SELECT NULLIF(10, 10); -- Returns NULL
SELECT NULLIF(10, 20); -- Returns 10
```

---

## 31. What is the EXPLAIN statement?

**Answer:** EXPLAIN shows how MySQL executes a query. It's used for query optimization and analyzing query execution plans.

Key columns:
- **type:** Join type (system, const, ref, range, index, ALL)
- **possible_keys:** Possible indexes
- **key:** Actually used index
- **rows:** Estimated rows to examine
- **Extra:** Additional information

Example:
```sql
EXPLAIN SELECT * FROM orders WHERE customer_id = 100;

EXPLAIN SELECT o.*, c.name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date > '2025-01-01';
```

---

## 32. What are the different types of relationships in MySQL?

**Answer:**

1. **One-to-One (1:1):** One record in Table A relates to one record in Table B
   ```sql
   users (user_id) <-> user_profiles (user_id)
   ```

2. **One-to-Many (1:N):** One record in Table A relates to many records in Table B
   ```sql
   customers (customer_id) <-> orders (customer_id)
   ```

3. **Many-to-Many (M:N):** Many records in Table A relate to many in Table B (requires junction table)
   ```sql
   students (student_id) <-> enrollments <-> courses (course_id)
   ```

Example:
```sql
-- One-to-Many
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Many-to-Many
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

---

## 33. What is the difference between LIKE and REGEXP?

**Answer:**

| LIKE | REGEXP |
|------|--------|
| Simple pattern matching | Advanced regular expressions |
| Uses % and _ wildcards | Uses regex patterns |
| Case insensitive (default) | Case sensitive (use REGEXP_LIKE) |
| Simpler and faster | More powerful and flexible |

Example:
```sql
-- LIKE
SELECT * FROM users WHERE name LIKE 'John%';  -- Starts with John
SELECT * FROM users WHERE name LIKE '%smith%'; -- Contains smith
SELECT * FROM users WHERE name LIKE 'J__n';    -- J + 2 chars + n

-- REGEXP
SELECT * FROM users WHERE name REGEXP '^John'; -- Starts with John
SELECT * FROM users WHERE email REGEXP '@gmail\\.com$'; -- Ends with @gmail.com
SELECT * FROM users WHERE phone REGEXP '^[0-9]{10}$'; -- Exactly 10 digits
```

---

## 34. How do you perform pagination in MySQL?

**Answer:** Pagination is done using LIMIT and OFFSET clauses.

Formula:
- `LIMIT`: Number of records per page
- `OFFSET`: (page_number - 1) × records_per_page

Example:
```sql
-- Page 1 (records 1-10)
SELECT * FROM products
ORDER BY product_id
LIMIT 10 OFFSET 0;

-- Page 2 (records 11-20)
SELECT * FROM products
ORDER BY product_id
LIMIT 10 OFFSET 10;

-- Page 3 (records 21-30)
SELECT * FROM products
ORDER BY product_id
LIMIT 10 OFFSET 20;

-- Alternative syntax
SELECT * FROM products
ORDER BY product_id
LIMIT 10, 10; -- LIMIT offset, count
```

---

## 35. What is the difference between HAVING and WHERE with GROUP BY?

**Answer:**

Execution order: WHERE → GROUP BY → HAVING → SELECT → ORDER BY

Example:
```sql
-- Correct: Filter before grouping with WHERE
SELECT department, COUNT(*) as emp_count
FROM employees
WHERE salary > 50000
GROUP BY department
HAVING COUNT(*) > 5;

-- This filters rows before grouping (more efficient)
-- Then filters groups after grouping
```

Explanation:
1. WHERE filters individual rows before grouping
2. GROUP BY creates groups
3. HAVING filters the groups created
4. SELECT determines output
5. ORDER BY sorts final result

---

## 36. How do you find the Nth highest salary in MySQL?

**Answer:**

Multiple approaches:

```sql
-- Method 1: Using LIMIT with OFFSET
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 2; -- 3rd highest (N-1)

-- Method 2: Using subquery
SELECT MIN(salary) as nth_salary
FROM (
    SELECT DISTINCT salary
    FROM employees
    ORDER BY salary DESC
    LIMIT 3  -- N value
) as top_salaries;

-- Method 3: Using DENSE_RANK()
SELECT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as rank_num
    FROM employees
) as ranked
WHERE rank_num = 3; -- Nth position

-- Method 4: Without using LIMIT (traditional)
SELECT salary
FROM employees e1
WHERE 2 = ( -- N-1
    SELECT COUNT(DISTINCT salary)
    FROM employees e2
    WHERE e2.salary > e1.salary
);
```

---

## 37. What are the different types of backups in MySQL?

**Answer:**

**1. Logical Backup:**
- Uses mysqldump
- Exports SQL statements
- Portable across platforms
- Slower restore

```bash
mysqldump -u root -p database_name > backup.sql
mysql -u root -p database_name < backup.sql
```

**2. Physical Backup:**
- Copies database files directly
- Faster for large databases
- Platform specific

**3. Hot Backup:**
- Database remains online
- No downtime
- Uses tools like MySQL Enterprise Backup

**4. Cold Backup:**
- Database is shut down
- Guaranteed consistency
- Requires downtime

**5. Incremental Backup:**
- Only changed data since last backup
- Saves space and time
- Binary log based

---

## 38. What is Query Optimization and how do you optimize queries?

**Answer:**

**Optimization Techniques:**

1. **Use Indexes:**
```sql
CREATE INDEX idx_email ON users(email);
```

2. **Avoid SELECT *:**
```sql
-- Bad
SELECT * FROM users;
-- Good
SELECT user_id, name, email FROM users;
```

3. **Use LIMIT:**
```sql
SELECT * FROM users LIMIT 100;
```

4. **Optimize JOIN queries:**
```sql
-- Use proper indexes on JOIN columns
-- Use INNER JOIN instead of WHERE when possible
```

5. **Avoid functions in WHERE:**
```sql
-- Bad
SELECT * FROM users WHERE YEAR(created_at) = 2025;
-- Good
SELECT * FROM users WHERE created_at >= '2025-01-01' AND created_at < '2026-01-01';
```

6. **Use EXPLAIN:**
```sql
EXPLAIN SELECT * FROM orders WHERE customer_id = 100;
```

7. **Avoid subqueries when possible:**
```sql
-- Use JOIN instead of subquery when appropriate
```

8. **Normalize database:**
- Reduce redundancy
- Improve data integrity

---

## 39. What is the difference between MyISAM and InnoDB locking?

**Answer:**

| Feature | InnoDB | MyISAM |
|---------|--------|--------|
| Locking Level | Row-level | Table-level |
| Concurrency | High | Low |
| Write Performance | Better for concurrent writes | Poor for concurrent writes |
| Lock Overhead | Higher | Lower |
| Deadlock Detection | Yes | No |
| Best For | OLTP, concurrent operations | Read-heavy, sequential operations |

Example impact:
```sql
-- InnoDB: Other rows can be accessed
UPDATE users SET status = 'active' WHERE user_id = 100;

-- MyISAM: Entire table locked
UPDATE users SET status = 'active' WHERE user_id = 100;
```

---

## 40. What are Common Table Expressions (CTE)?

**Answer:** CTEs are temporary named result sets defined using WITH clause. They improve query readability and allow recursive queries.

**Advantages:**
- Better readability
- Can be referenced multiple times
- Supports recursion
- Easier to maintain

Example:
```sql
-- Simple CTE
WITH high_earners AS (
    SELECT employee_id, name, salary
    FROM employees
    WHERE salary > 100000
)
SELECT * FROM high_earners WHERE salary > 150000;

-- Multiple CTEs
WITH
sales_summary AS (
    SELECT
        salesperson_id,
        SUM(amount) as total_sales
    FROM sales
    GROUP BY salesperson_id
),
top_performers AS (
    SELECT salesperson_id
    FROM sales_summary
    WHERE total_sales > 50000
)
SELECT e.name, s.total_sales
FROM employees e
JOIN sales_summary s ON e.id = s.salesperson_id
JOIN top_performers t ON e.id = t.salesperson_id;

-- Recursive CTE (employee hierarchy)
WITH RECURSIVE employee_hierarchy AS (
    -- Base case
    SELECT employee_id, name, manager_id, 1 as level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case
    SELECT e.employee_id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM employee_hierarchy;
```

---

## Additional Tips for MySQL Interviews:

1. **Understand ACID properties thoroughly**
2. **Practice writing complex JOIN queries**
3. **Know the difference between clustered and non-clustered indexes**
4. **Be familiar with query optimization techniques**
5. **Understand transactions and isolation levels**
6. **Know when to use different storage engines**
7. **Practice database design and normalization**
8. **Understand performance tuning and EXPLAIN plans**
9. **Be familiar with replication and backup strategies**
10. **Know common design patterns and anti-patterns**

---

## Practice Queries:

```sql
-- Find second highest salary
SELECT MAX(salary) FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

-- Find employees without orders
SELECT e.* FROM employees e
LEFT JOIN orders o ON e.employee_id = o.employee_id
WHERE o.order_id IS NULL;

-- Running total
SELECT
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) as running_total
FROM orders;

-- Pivot data
SELECT
    year,
    SUM(CASE WHEN quarter = 'Q1' THEN revenue END) as Q1,
    SUM(CASE WHEN quarter = 'Q2' THEN revenue END) as Q2,
    SUM(CASE WHEN quarter = 'Q3' THEN revenue END) as Q3,
    SUM(CASE WHEN quarter = 'Q4' THEN revenue END) as Q4
FROM sales
GROUP BY year;
```

