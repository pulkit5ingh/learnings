# Chapter 9 — Advanced SQL: Window Functions, UPSERT & Interview Queries

Where senior backend / Tech Lead interviews get much more interesting. Already covered: `SELECT`, `WHERE`, `JOIN`, `GROUP BY`, `HAVING`, subqueries, CTEs, indexes, `EXPLAIN`. Now: SQL features that solve complex problems without multiple queries in application code.

---

## 1. `CASE`

SQL's `if/else`.

```text
orders
------
amount
```

Classify orders: `< 1000 → SMALL`, `1000-5000 → MEDIUM`, `> 5000 → LARGE`.

```sql
SELECT
    id,
    amount,
    CASE
        WHEN amount < 1000 THEN 'SMALL'
        WHEN amount <= 5000 THEN 'MEDIUM'
        ELSE 'LARGE'
    END AS order_category
FROM orders;
```

```text
id | amount | order_category
---+--------+---------------
1  | 500    | SMALL
2  | 2500   | MEDIUM
3  | 10000  | LARGE
```

MongoDB equivalent:

```javascript
{
  $switch: {
    branches: [
      { case: { $lt: ["$amount", 1000] }, then: "SMALL" }
    ]
  }
}
```

---

## 2. `CASE` inside aggregation

```sql
SELECT
    SUM(CASE WHEN status = 'SUCCESS' THEN amount ELSE 0 END) AS successful_amount,
    SUM(CASE WHEN status = 'FAILED' THEN amount ELSE 0 END) AS failed_amount
FROM payments;
```

One query gives both metrics.

---

## 3. `COALESCE`

```sql
SELECT
    u.name,
    SUM(o.amount)
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
GROUP BY u.id, u.name;
```

A user with no orders produces `SUM = NULL`. Want `0` instead:

```sql
SELECT
    u.name,
    COALESCE(SUM(o.amount), 0) AS total_amount
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
GROUP BY u.id, u.name;
```

`COALESCE(a, b)` means: return `a` if not NULL, otherwise return `b`.

---

## 4. `NULLIF`

```sql
SELECT revenue / orders
FROM statistics;
```

If `orders = 0`, division-by-zero. Use:

```sql
SELECT revenue / NULLIF(orders, 0)
FROM statistics;
```

`NULLIF(0, 0) → NULL`, so `revenue / NULL` becomes NULL instead of throwing.

---

## 5. DISTINCT

```sql
SELECT DISTINCT user_id
FROM orders;
```

MongoDB: `db.orders.distinct("userId")`

---

## 6. `DISTINCT ON` — PostgreSQL-specific

```text
orders
------------------------
user_id | created_at
------------------------
1       | Jan 1
1       | Jan 5
1       | Jan 10
2       | Jan 2
2       | Jan 8
```

Latest order per user:

```sql
SELECT DISTINCT ON (user_id)
    user_id,
    id,
    amount,
    created_at
FROM orders
ORDER BY user_id, created_at DESC;
```

```text
user_id | latest order
--------+-------------
1       | Jan 10
2       | Jan 8
```

Critical rule: `DISTINCT ON (user_id)` must align with the beginning of `ORDER BY user_id, ...`. A great PostgreSQL-specific interview feature.

---

## 7. Window Functions

`GROUP BY` collapses multiple rows into one row per group. Window functions calculate something across related rows **while keeping the original rows** — a critical distinction.

---

## 8. GROUP BY example

```text
employees
--------------------------------
id | name | department | salary
--------------------------------
1  | A    | Engineering | 100000
2  | B    | Engineering | 120000
3  | C    | Engineering | 90000
4  | D    | HR          | 80000
5  | E    | HR          | 70000
```

```sql
SELECT department, AVG(salary)
FROM employees
GROUP BY department;
```

```text
Engineering | 103333
HR          | 75000
```

Individual employee rows disappear.

---

## 9. Window function version

```sql
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS department_avg
FROM employees;
```

```text
name | department  | salary | department_avg
-----+-------------+--------+---------------
A    | Engineering | 100000 | 103333
B    | Engineering | 120000 | 103333
C    | Engineering | 90000  | 103333
D    | HR          | 80000  | 75000
E    | HR          | 70000  | 75000
```

```text
GROUP BY         → reduces rows
Window Function  → keeps rows
```

---

## 10. `ROW_NUMBER()`

```sql
SELECT
    name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS rank
FROM employees;
```

```text
name | salary | rank
-----+--------+-----
B    | 120000 | 1
A    | 100000 | 2
C    | 90000  | 3
D    | 80000  | 4
E    | 70000  | 5
```

---

## 11. `RANK()` vs `ROW_NUMBER()`

```text
salary
------
120000
100000
100000
90000
```

**ROW_NUMBER**: `120000→1, 100000→2, 100000→3, 90000→4` — every row gets a unique number.

**RANK**: `120000→1, 100000→2, 100000→2, 90000→4` — note the gap after ties.

---

## 12. `DENSE_RANK()`

`120000→1, 100000→2, 100000→2, 90000→3` — no gap.

```text
ROW_NUMBER → unique sequence
RANK       → ties + gaps
DENSE_RANK → ties + no gaps
```

A very common interview question.

---

## 13. Second-highest salary

```text
salary
------
100000
80000
70000
```

```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

Works, but with duplicates and more complex ranking requirements, window functions are often cleaner.

---

## 14. Second-highest salary using `DENSE_RANK`

```sql
WITH ranked AS (
    SELECT
        name,
        salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_rank
    FROM employees
)
SELECT name, salary
FROM ranked
WHERE salary_rank = 2;
```

With `120000, 100000, 100000, 90000`, both employees earning 100000 are rank 2.

---

## 15. Top employee in each department

```sql
WITH ranked AS (
    SELECT
        id,
        name,
        department,
        salary,
        ROW_NUMBER() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS rn
    FROM employees
)
SELECT id, name, department, salary
FROM ranked
WHERE rn = 1;
```

General pattern to memorize:

```sql
WITH ranked AS (
    SELECT
        ...,
        ROW_NUMBER() OVER (
            PARTITION BY group_column
            ORDER BY value DESC
        ) AS rn
    FROM table
)
SELECT *
FROM ranked
WHERE rn = 1;
```

Applies to: latest order per user, highest transaction per customer, latest payment per order, latest status per entity, top record per category.

---

## 16. `LAG()`

```text
date | revenue
-----+--------
Jan1 | 100
Jan2 | 120
Jan3 | 150
```

```sql
SELECT
    date,
    revenue,
    LAG(revenue) OVER (ORDER BY date) AS previous_revenue
FROM daily_revenue;
```

```text
date | revenue | previous_revenue
-----+---------+-----------------
Jan1 | 100     | NULL
Jan2 | 120     | 100
Jan3 | 150     | 120
```

Growth calculation:

```sql
SELECT
    date,
    revenue,
    revenue - LAG(revenue) OVER (ORDER BY date) AS change
FROM daily_revenue;
```

---

## 17. `LEAD()`

```sql
SELECT
    date,
    revenue,
    LEAD(revenue) OVER (ORDER BY date) AS next_revenue
FROM daily_revenue;
```

```text
LAG  → previous row
LEAD → next row
```

---

## 18. UPSERT

> Insert the user if they don't exist; otherwise update them.

```sql
INSERT INTO users (email, name)
VALUES ('pulkit@example.com', 'Pulkit')
ON CONFLICT (email)
DO UPDATE SET name = EXCLUDED.name;
```

An **UPSERT**.

---

## 19. `ON CONFLICT DO NOTHING`

```sql
INSERT INTO users (email, name)
VALUES ('pulkit@example.com', 'Pulkit')
ON CONFLICT (email)
DO NOTHING;
```

If the email already exists: no error, no duplicate.

---

## 20. Why UPSERT beats "check then insert"

Bad pattern:

```text
SELECT user WHERE email = X
if user doesn't exist: INSERT user
```

Two simultaneous requests:

```text
Request A → SELECT → doesn't exist
Request B → SELECT → doesn't exist
Request A → INSERT
Request B → INSERT
```

Race condition unless the database constraint protects you. Better:

```sql
INSERT ... ON CONFLICT ...
```

combined with `UNIQUE(email)` — let PostgreSQL enforce the invariant atomically. Very important for backend interviews.

---

## 21. TypeORM UPSERT

```typescript
await userRepository.upsert(
  { email: 'pulkit@example.com', name: 'Pulkit' },
  ['email'],
);
```

Relies on the database's conflict/unique mechanism.

> Don't implement concurrency-sensitive uniqueness with an application-level "check then insert" alone. Put the uniqueness constraint in PostgreSQL.

---

## 22. Latest record per user — two approaches

```text
orders
--------------------------------
id | user_id | created_at
--------------------------------
1  | 10      | Jan 1
2  | 10      | Jan 5
3  | 20      | Jan 2
4  | 20      | Jan 9
```

**PostgreSQL-specific**:

```sql
SELECT DISTINCT ON (user_id)
    id, user_id, created_at
FROM orders
ORDER BY user_id, created_at DESC;
```

**Window-function**:

```sql
WITH ranked AS (
    SELECT
        id, user_id, created_at,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY created_at DESC
        ) AS rn
    FROM orders
)
SELECT id, user_id, created_at
FROM ranked
WHERE rn = 1;
```

Both useful.

---

## 23. Consecutive-day problem

Login dates: `Jan 1, Jan 2, Jan 3, Jan 5, Jan 6` — identify consecutive login streaks.

Window functions are extremely useful here. The general idea: `ROW_NUMBER()` + date arithmetic + grouping. Solved later as part of interview drills.

---

## 24. SQL execution mental model

Written order (`SELECT, FROM, WHERE, GROUP BY, HAVING, ORDER BY, LIMIT`) isn't the logical processing order:

```text
FROM
 ↓
JOIN
 ↓
WHERE
 ↓
GROUP BY
 ↓
HAVING
 ↓
SELECT
 ↓
DISTINCT
 ↓
ORDER BY
 ↓
LIMIT
```

This is why `WHERE SUM(amount) > 10000` doesn't work — the aggregate hasn't been produced at the `WHERE` stage (use `HAVING` instead).

---

## Chapter 9 challenge

### Q1
Given `employees(id, name, department, salary)`, find the second-highest distinct salary.

### Q2
Find the highest-paid employee in each department.

### Q3
Given `orders(id, user_id, amount, created_at)`, find the latest order for every user.

### Q4
Explain `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`.

### Q5
Why is `SELECT → check → INSERT` unsafe for "create user if email doesn't exist"? Why is `INSERT ... ON CONFLICT ...` better?

### Q6 — Tech Lead
`users.email UNIQUE`. Two API requests arrive simultaneously with the same email. How do you guarantee only one user gets created?

---

## Next chapter: PostgreSQL Schema Design & Production Patterns

Shifting from individual SQL queries to architecture: normalization, denormalization, UUID vs BIGINT, soft delete, audit fields, timestamps, ENUM vs VARCHAR, JSONB, database constraints, foreign keys, cascades, idempotency, multi-tenant schemas.

Then the TypeORM section begins — building these concepts in NestJS and covering the TypeORM questions interviewers commonly ask.
