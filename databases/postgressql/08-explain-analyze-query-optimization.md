# Chapter 8 — EXPLAIN ANALYZE & Query Optimization

Real Tech Lead territory. Knowing SQL is one thing; answering "this query takes 8 seconds in production, what's wrong?" is a much more valuable skill.

MongoDB knowledge transfers directly — MongoDB also has query plans and `explain()`:

```text
MongoDB
db.collection.find(...).explain("executionStats")

          ↓

PostgreSQL
EXPLAIN ANALYZE
```

---

## 1. What is `EXPLAIN`?

```sql
EXPLAIN
SELECT *
FROM users
WHERE email = 'pulkit@example.com';
```

Gives a **query execution plan**. The planner decides: should I use an index? scan the table? which join algorithm? should I sort? how much data will I process?

---

## 2. EXPLAIN vs EXPLAIN ANALYZE

**EXPLAIN** — shows the planned execution strategy. Does not normally execute the query.

**EXPLAIN ANALYZE** — actually executes the query and reports what happened:

```text
Planning Time: 0.2 ms
Execution Time: 1.5 ms
```

Straightforward for `SELECT`. For production writes, be careful — `EXPLAIN ANALYZE DELETE ...` actually executes the delete.

---

## 3. Example: Sequential Scan

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE age = 28;
```

```text
Seq Scan on users
  Filter: (age = 28)
  Rows Removed by Filter: 999000
```

```text
1,000,000 rows
       ↓
Sequential Scan
       ↓
check each row
       ↓
rows matching age=28
```

---

## 4. Why did PostgreSQL use Sequential Scan?

Before assuming "I need an index!" check whether one exists:

```sql
CREATE INDEX idx_users_age
ON users(age);
```

Re-running `EXPLAIN ANALYZE` could now show `Index Scan` — but PostgreSQL is not required to use your index.

---

## 5. Index Scan

```text
Index Scan using idx_users_email on users
```

```text
Query → Index → Find matching row locations → Fetch rows from table
```

Useful when relatively few rows match.

---

## 6. Index Only Scan

```sql
SELECT id, email
FROM users
WHERE email = 'pulkit@example.com';
```

If the index contains all required information, PostgreSQL may answer the query primarily from the index without fetching the full table row — an **Index Only Scan**. Whether it truly avoids heap/table access depends on PostgreSQL's visibility information and table state.

---

## 7. Bitmap Index Scan

```text
Bitmap Index Scan
Bitmap Heap Scan
```

Often appears when PostgreSQL expects multiple matching rows and wants to efficiently combine index results with heap/table access.

```text
Index → Find many matching row locations → Build bitmap → Fetch relevant table pages
```

No need to memorize exactly when PostgreSQL picks each plan — understand what the nodes mean.

---

## 8. The most important numbers

```text
cost=0.42..100.50
rows=100
width=50
actual time=0.05..20.00
rows=95
loops=1
```

- **`cost`** — planner cost estimates, *not* milliseconds. "Cost 100 means 100ms" is wrong.
- **`rows`** — estimated number of rows.
- **`actual time`** (with `EXPLAIN ANALYZE`) — actual execution timing in milliseconds.
- **`loops`** — how many times that node executed. Extremely important with nested loops.

---

## 9. Estimated vs actual rows

One of the most useful things to inspect.

```text
estimated rows = 10
actual rows    = 500,000
```

A huge estimation error matters because PostgreSQL chooses plans based on estimates. If wrong, the optimizer may pick a bad plan — e.g. a Nested Loop when a Hash Join would have been more appropriate.

---

## 10. Statistics

PostgreSQL maintains statistics about table data. Manually update with:

```sql
ANALYZE users;
```

Normally autovacuum/autoanalyze handles this automatically. If planner estimates are badly wrong, stale or insufficient statistics are worth investigating.

---

## 11. Nested Loop Join

```text
users = 1,000 rows
orders = 10,000,000 rows
```

```sql
SELECT *
FROM users u
JOIN orders o
    ON o.user_id = u.id;
```

May use `Nested Loop`:

```text
for each user:
    find matching orders
```

If the outer side is small and the inner lookup is efficient, this can be very fast.

---

## 12. When Nested Loop becomes problematic

```text
Outer = 1,000,000 rows
Inner = 10,000,000 rows
```

repeatedly scanning a large table → `1,000,000 × expensive lookup` can be terrible. Inspect `loops`, `actual rows`, `actual time` in the plan.

---

## 13. Hash Join

```text
Smaller relation → Build hash table → Scan other relation → Lookup matches
```

PostgreSQL builds a hash structure for one side and matches the other side against it. Particularly useful for equality joins.

---

## 14. Merge Join

```text
Sorted A + Sorted B → Walk through both → Find matches
```

Useful when both inputs are already appropriately sorted or can be sorted efficiently.

---

## 15. Don't memorize "Nested Loop bad"

A common mistake. A good Tech Lead answer:

> The right join strategy depends on the data size, cardinality, indexes, statistics and query shape.

`10 users` joined against `10 million orders` with an index on `orders.user_id` — a Nested Loop could be excellent. Don't say "Nested Loop = bad."

---

## 16. Query optimization example

```sql
SELECT *
FROM orders
WHERE user_id = 123
ORDER BY created_at DESC
LIMIT 20;
```

`100 million orders`. `EXPLAIN ANALYZE` might show:

```text
Seq Scan → Filter user_id → Sort → Limit
```

Expensive.

---

## 17. Create an appropriate composite index

```sql
CREATE INDEX idx_orders_user_created
ON orders(user_id, created_at DESC);
```

Now PostgreSQL can efficiently locate `user_id = 123`, already ordered by `created_at DESC`, and take the first 20:

```text
Index → user 123 → newest first → 20 rows
```

Much better than `100 million rows → filter → sort millions of rows → take 20`.

---

## 18. Covering indexes

```sql
SELECT id, amount, created_at
FROM orders
WHERE user_id = $1
ORDER BY created_at DESC
LIMIT 20;
```

```sql
CREATE INDEX idx_orders_user_created
ON orders(user_id, created_at DESC)
INCLUDE (id, amount);
```

`INCLUDE` stores additional non-key columns in the index, enabling index-only access for suitable queries. Don't blindly create giant covering indexes — more index data means more storage and more write maintenance.

---

## 19. A very important optimization principle

> "The query is slow. Add an index."

Tech Lead response:

```text
First: EXPLAIN ANALYZE
Then: Understand the bottleneck
Then: Optimize based on evidence
```

Not: `Slow query → random index → hope`.

---

## 20. Common performance problems

1. **Sequential scan on a huge table** when a selective indexed lookup would be appropriate.
2. **Bad estimates** — `estimated rows: 10` vs `actual rows: 5,000,000`.
3. **Expensive sort** over millions of rows.
4. **Large aggregation** (`GROUP BY`) over huge datasets.
5. **Bad JOIN** — especially `Nested Loop` with `loops = millions`.
6. **Missing index** on JOIN columns, WHERE columns, ORDER BY patterns.
7. **N+1 queries** — the query itself is fast, but the application runs thousands of them.
8. **Lock contention** — the query isn't CPU-bound, it's waiting.

---

## 21. A Tech Lead debugging workflow

```text
Slow API
   ↓
Find actual SQL
   ↓
EXPLAIN ANALYZE
   ↓
Inspect execution plan
   ↓
Estimated vs actual rows
   ↓
Scan / Join / Sort / Aggregate
   ↓
Check indexes
   ↓
Check statistics
   ↓
Check locks / contention
   ↓
Optimize
   ↓
EXPLAIN ANALYZE again
   ↓
Measure improvement
```

---

## 22. TypeORM QueryBuilder + EXPLAIN

```typescript
const orders = await orderRepository
  .createQueryBuilder('order')
  .where('order.userId = :userId', { userId })
  .orderBy('order.createdAt', 'DESC')
  .limit(20)
  .getMany();
```

The important Tech Lead skill isn't just knowing TypeORM — it's asking "what SQL is TypeORM generating?" and inspecting it:

```text
TypeORM → SQL → EXPLAIN ANALYZE → PostgreSQL execution plan
```

Covered more thoroughly once QueryBuilder is reached.

---

## 23. Interview scenario

> We have an endpoint `/users/:id/orders`. It used to take 50ms but now takes 4 seconds. There are 100 million orders. What do you do?

> First I would identify the exact SQL generated by the application and run `EXPLAIN ANALYZE` in a safe environment against representative data. I'd inspect whether PostgreSQL is doing a sequential scan, an expensive sort, an inefficient join, or returning far more rows than expected. I'd check whether an index matching the `user_id` filter and ordering by `created_at` exists, for example `(user_id, created_at DESC)`. I'd also compare estimated and actual row counts, check statistics, and investigate locking or I/O if the plan itself looks reasonable. After making a change, I'd measure the new execution plan rather than assuming the index fixed it.

---

## Chapter 8 challenge

```sql
orders (
    id,
    user_id,
    status,
    amount,
    created_at
)
```

```sql
SELECT id, amount, created_at
FROM orders
WHERE user_id = 100
  AND status = 'PAID'
ORDER BY created_at DESC
LIMIT 20;
```

100 million rows.

### Q1
What composite index would you initially consider?

### Q2
Why might this be better than three separate indexes — `(user_id)`, `(status)`, `(created_at)`?

### Q3
What would you run to investigate whether PostgreSQL actually benefits from your index?

### Q4
`EXPLAIN ANALYZE` says `estimated rows: 20`, `actual rows: 4,000,000`. What does that tell you?

### Q5 — Tech Lead
The query plan shows `Nested Loop`, `loops = 5,000,000`. Would you immediately say "Nested Loop is bad"? Why or why not?

---

## Next: Chapter 9 — Advanced PostgreSQL Queries

`CASE`, `COALESCE`, `NULLIF`, `DISTINCT ON`, Window Functions (`ROW_NUMBER()`, `RANK()`, `LAG()`, `LEAD()`), Subqueries, CTEs, Recursive CTEs, `UPSERT` / `ON CONFLICT`.

Classic interview questions to solve: find the second-highest salary; find the highest-paid employee in each department; find users who made purchases on consecutive days; insert a user without creating a duplicate if the email already exists.
