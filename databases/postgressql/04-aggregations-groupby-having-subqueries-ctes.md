# Chapter 4 — Aggregations, GROUP BY, HAVING, Subqueries & CTEs

A very important interview chapter. Coming from MongoDB:

```text
MongoDB Aggregation Pipeline
        ↓
PostgreSQL
        ↓
GROUP BY + aggregate functions + CTEs + subqueries
```

Going from easy → Tech Lead level.

---

## 1. Aggregate functions

MongoDB: `$sum`, `$avg`, `$min`, `$max`, `$count`

PostgreSQL: `SUM()`, `AVG()`, `MIN()`, `MAX()`, `COUNT()`

```text
orders
--------------------------------
id | user_id | amount
--------------------------------
1  | 101     | 500
2  | 101     | 1000
3  | 102     | 2000
4  | 102     | 3000
5  | 103     | 500
```

### SUM

```sql
SELECT SUM(amount)
FROM orders;
```

Result: `7000`

MongoDB equivalent:

```javascript
{ $group: { _id: null, total: { $sum: "$amount" } } }
```

---

## 2. COUNT

```sql
SELECT COUNT(*)
FROM orders;
```

Result: `5`

MongoDB: `{ $count: "total" }`

**Important interview distinction** — `COUNT(*)` vs `COUNT(amount)` aren't always equivalent:

- `COUNT(*)` counts rows.
- `COUNT(amount)` counts rows where `amount` is **not NULL**.

```text
amount
------
500
1000
NULL
2000
```

```text
COUNT(*)       → 4
COUNT(amount)  → 3
```

A common interview trick.

---

## 3. GROUP BY

> How much has each user spent?

MongoDB:

```javascript
db.orders.aggregate([
    { $group: { _id: "$user_id", total: { $sum: "$amount" } } }
])
```

PostgreSQL:

```sql
SELECT
    user_id,
    SUM(amount) AS total_amount
FROM orders
GROUP BY user_id;
```

Result:

```text
user_id | total_amount
--------+-------------
101     | 1500
102     | 5000
103     | 500
```

Think:

```text
GROUP BY user_id

101 → [500, 1000]
102 → [2000, 3000]
103 → [500]
```

Then `SUM()` is calculated for each group.

---

## 4. GROUP BY multiple columns

```text
orders
-----------------------------
user_id | status | amount
-----------------------------
101     | PAID   | 500
101     | PAID   | 1000
101     | FAILED | 300
102     | PAID   | 2000
```

```sql
SELECT
    user_id,
    status,
    SUM(amount) AS total
FROM orders
GROUP BY user_id, status;
```

Result:

```text
user_id | status | total
--------+--------+------
101     | PAID   | 1500
101     | FAILED | 300
102     | PAID   | 2000
```

MongoDB equivalent: grouping using a compound `_id`.

---

## 5. AVG

```sql
SELECT AVG(amount)
FROM orders;
```

Per-user average:

```sql
SELECT
    user_id,
    AVG(amount) AS average_order
FROM orders
GROUP BY user_id;
```

---

## 6. MIN and MAX

```sql
SELECT
    MIN(amount) AS minimum_order,
    MAX(amount) AS maximum_order
FROM orders;
```

Per user:

```sql
SELECT
    user_id,
    MIN(amount) AS minimum_order,
    MAX(amount) AS maximum_order
FROM orders
GROUP BY user_id;
```

---

## 7. WHERE + GROUP BY

> Calculate total spending for PAID orders only.

```sql
SELECT
    user_id,
    SUM(amount) AS total_amount
FROM orders
WHERE status = 'PAID'
GROUP BY user_id;
```

```text
orders
   ↓
WHERE status = PAID
   ↓
GROUP BY user_id
   ↓
SUM(amount)
```

`WHERE` filters **before grouping**.

---

## 8. HAVING

> Find users whose paid orders exceed ₹50,000.

```sql
SELECT
    user_id,
    SUM(amount) AS total_amount
FROM orders
WHERE status = 'PAID'
GROUP BY user_id
HAVING SUM(amount) > 50000;
```

```text
WHERE     → filters rows
GROUP BY  → creates groups
HAVING    → filters groups
```

---

## 9. WHERE vs HAVING

An interview favorite.

- **WHERE** (`status = 'PAID'`) — remove individual rows that don't satisfy the condition.
- **HAVING** (`SUM(amount) > 50000`) — remove groups whose aggregate result doesn't satisfy the condition.

---

## 10. Worked example

```text
user_id | status | amount
--------+--------+-------
1       | PAID   | 10000
1       | FAILED | 90000
2       | PAID   | 60000
```

```sql
SELECT
    user_id,
    SUM(amount)
FROM orders
WHERE status = 'PAID'
GROUP BY user_id
HAVING SUM(amount) > 50000;
```

The `FAILED` ₹90,000 order for user 1 is removed **before** aggregation, so:

```text
user 1 → 10000
user 2 → 60000
```

Only user 2 appears in the result.

---

## 11. Subqueries

> Find users whose age is greater than the average age.

```sql
SELECT AVG(age)
FROM users;
```

```sql
SELECT *
FROM users
WHERE age > (
    SELECT AVG(age)
    FROM users
);
```

`(SELECT AVG(age) FROM users)` is a **subquery**. MongoDB doesn't map directly to this, but think of it as running an intermediate computation and using its result in another operation.

---

## 12. Another subquery example

> Find users who have placed at least one order.

```sql
SELECT *
FROM users
WHERE id IN (
    SELECT user_id
    FROM orders
);
```

Inner query `SELECT user_id FROM orders;` produces `101, 102, 103`, then `WHERE id IN (...)` finds those users.

---

## 13. `EXISTS`

A very common alternative:

```sql
SELECT *
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

`EXISTS` asks: does at least one matching row exist? You don't care about the actual order data — that's why `SELECT 1` is commonly used.

---

## 14. `NOT EXISTS`

> Find users who have never placed an order.

```sql
SELECT *
FROM users u
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

Another very common interview question. Compare with the `LEFT JOIN` approach:

```sql
SELECT u.*
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id
WHERE o.id IS NULL;
```

Both solve the problem — when `EXISTS`, `JOIN`, or `IN` is preferable is discussed later.

---

## 15. CTE — Common Table Expression

> Calculate each user's total spending, then find users spending more than ₹50,000.

Subquery version:

```sql
SELECT *
FROM (
    SELECT
        user_id,
        SUM(amount) AS total_amount
    FROM orders
    GROUP BY user_id
) totals
WHERE total_amount > 50000;
```

CTE version — much easier to read:

```sql
WITH user_totals AS (
    SELECT
        user_id,
        SUM(amount) AS total_amount
    FROM orders
    GROUP BY user_id
)
SELECT *
FROM user_totals
WHERE total_amount > 50000;
```

```text
WITH
   ↓
temporary named result
   ↓
main query
```

---

## 16. Multiple CTEs

```sql
WITH user_totals AS (
    SELECT
        user_id,
        SUM(amount) AS total_amount
    FROM orders
    WHERE status = 'PAID'
    GROUP BY user_id
),
active_users AS (
    SELECT
        id,
        name
    FROM users
    WHERE is_active = TRUE
)
SELECT
    active_users.name,
    user_totals.total_amount
FROM active_users
JOIN user_totals
    ON active_users.id = user_totals.user_id;
```

Much easier to maintain than one giant nested query.

---

## 17. CTE vs MongoDB pipeline

```text
MongoDB                    PostgreSQL

$match                     WITH ...
 ↓                           ↓
$group                     WHERE
 ↓                           ↓
$lookup                    GROUP BY
 ↓                           ↓
$project                   JOIN
 ↓                           ↓
$sort                      SELECT
                              ↓
                            ORDER BY
```

Conceptually different systems, but aggregation experience transfers well.

---

## 18. A Tech Lead-level question

> Get the top 5 customers by total successful payment amount.

```text
users
-----
id
name

payments
--------
id
user_id
amount
status
```

```sql
SELECT
    u.id,
    u.name,
    SUM(p.amount) AS total_paid
FROM users u
JOIN payments p
    ON p.user_id = u.id
WHERE p.status = 'SUCCESS'
GROUP BY u.id, u.name
ORDER BY total_paid DESC
LIMIT 5;
```

Pipeline:

```text
JOIN → WHERE → GROUP BY → SUM → ORDER BY → LIMIT
```

A very useful SQL pattern.

---

## 19. Aliases

Instead of:

```sql
SELECT
    users.name,
    orders.amount
FROM users
JOIN orders
    ON orders.user_id = users.id;
```

write:

```sql
SELECT
    u.name,
    o.amount
FROM users u
JOIN orders o
    ON o.user_id = u.id;
```

`u → users`, `o → orders`. Essential once queries span 5–10 tables.

---

## 20. Tech Lead mental model

> "Give me the top 10 active users by successful payment amount this month."

Break it down:

```text
1. users
2. payments
3. JOIN users → payments
4. active users → WHERE
5. successful payments → WHERE
6. this month → WHERE
7. GROUP BY user
8. SUM(amount)
9. ORDER BY SUM(amount) DESC
10. LIMIT 10
```

```sql
SELECT
    u.id,
    u.name,
    SUM(p.amount) AS total_paid
FROM users u
JOIN payments p
    ON p.user_id = u.id
WHERE u.is_active = TRUE
  AND p.status = 'SUCCESS'
  AND p.created_at >= DATE_TRUNC('month', CURRENT_DATE)
GROUP BY u.id, u.name
ORDER BY total_paid DESC
LIMIT 10;
```

This is the kind of thinking to develop for the interview — not memorizing SQL snippets.

---

## Chapter 4 challenge

```text
users
-----
id
name
is_active

orders
------
id
user_id
amount
status
created_at
```

### Q1
Find the total number of orders.

### Q2
Find total order amount for each user. Expected: `user_id | total_amount`

### Q3
Find users whose total **PAID** orders are greater than ₹10,000.

### Q4
Find users who have **never placed an order**.

### Q5 — Tech Lead
Write a query for: top 5 active users by total successful order amount.

### Q6 — conceptual
Explain in your own words: `WHERE` vs `GROUP BY` vs `HAVING`.

Next: **Chapter 5 — Relationships & Database Design** (1:1, 1:N, N:M, foreign keys, normalization, cascading deletes, and designing PostgreSQL schemas like a Tech Lead).
