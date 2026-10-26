# Chapter 21 — PostgreSQL Query Optimization & Execution Plans

Moving from "I can write SQL" to "I can look at a slow production query and explain exactly why it's slow." A major Senior Backend / Tech Lead skill.

---

## 1. The Query Optimization Mindset

`users = 10 million rows`, `orders = 100 million rows`, and this takes 4 seconds:

```sql
SELECT * FROM orders WHERE user_id = '123';
```

Don't immediately say "add an index." Ask:

```text
Why is it slow?
↓
What is PostgreSQL actually doing?
↓
How many rows does it scan?
↓
Is the estimate correct?
↓
Is the index useful?
↓
Is the query returning too much data?
```

`EXPLAIN` and `EXPLAIN ANALYZE` answer this.

---

## 2. EXPLAIN

```sql
EXPLAIN
SELECT * FROM orders WHERE user_id = '123';
```

```text
Index Scan using idx_orders_user_id on orders
  Index Cond: (user_id = '123')
```

Shows the planned execution strategy — doesn't actually execute the query.

---

## 3. EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = '123';
```

Actually executes and gives real statistics:

```text
Index Scan using idx_orders_user_id on orders
  (cost=0.43..120.12 rows=25 width=80)
  (actual time=0.032..0.155 rows=27 loops=1)
```

Important fields: `cost`, `rows`, `actual time`, `loops`.

---

## 4. Cost ≠ Milliseconds

A common interview trap. `cost=0.43..120.12` are planner cost units, not `0.43ms`/`120.12ms`. Actual timing comes from `actual time=...`.

---

## 5. Estimated Rows vs Actual Rows

`rows=100` but `actual rows=100000` — a huge mismatch. PostgreSQL planned for 100 rows but got 100,000, which can cause it to choose a poor execution strategy. One of the most important things to check in `EXPLAIN ANALYZE`.

---

## 6. Why Estimates Matter

PostgreSQL estimates 10 rows and chooses a Nested Loop, but actual data is 10 million rows — now that Nested Loop may be terrible. The planner decided using incorrect statistics.

---

## 7. ANALYZE

```sql
ANALYZE orders;
```

or `ANALYZE;` for the whole database. Autovacuum/auto-analyze normally handles this automatically, but understanding it matters when diagnosing planner problems.

---

## 8. Sequential Scan

```text
Seq Scan on orders
```

Scans the table sequentially: row 1, row 2, ... row 100,000,000. Sounds bad, but **Sequential Scan is not automatically bad**.

---

## 9. When Sequential Scan Is Actually Better

`orders = 1,000,000 rows`, `WHERE status = 'ACTIVE'` matching 800,000 rows. Using an index means: index → 800,000 row locations → table fetches. A sequential scan just reads 1,000,000 rows straight through — can be cheaper.

Don't say "Seq Scan means missing index." Instead: "I need to understand why the planner chose a sequential scan."

---

## 10. Index Scan

```text
Index Scan using idx_orders_user_id
```

```sql
SELECT * FROM orders WHERE user_id = '123';
```

with `CREATE INDEX idx_orders_user_id ON orders(user_id);`

---

## 11. Bitmap Index Scan

```text
Bitmap Index Scan → Bitmap Heap Scan
```

Useful when PostgreSQL expects multiple matching rows:

```text
Index → Find many matching row locations → Build bitmap → Fetch table pages efficiently
```

Sits conceptually between Index Scan and Seq Scan.

---

## 12. Index-Only Scan

```sql
CREATE INDEX idx_orders_user_created
ON orders(user_id, created_at);
```

```sql
SELECT user_id, created_at
FROM orders
WHERE user_id = '123'
ORDER BY created_at DESC;
```

If the index contains everything needed, PostgreSQL may avoid fetching table rows entirely — significantly faster.

---

## 13. Covering Indexes

```sql
CREATE INDEX idx_orders_user
ON orders(user_id)
INCLUDE (created_at, status);
```

`INCLUDE` stores additional columns in the index without making them part of the search key — helps enable index-only scans. `user_id` is the search key; `created_at`, `status` are included payload columns.

---

## 14. Composite Indexes

```sql
SELECT *
FROM orders
WHERE user_id = '123'
ORDER BY created_at DESC
LIMIT 20;
```

```sql
CREATE INDEX idx_orders_user_created
ON orders(user_id, created_at DESC);
```

Matches the access pattern: `user_id → created_at DESC → LIMIT 20`. PostgreSQL can efficiently find the user's newest orders.

---

## 15. Index Column Order Matters

`(user_id, created_at)` ≠ `(created_at, user_id)`. For `WHERE user_id = ? ORDER BY created_at DESC`, the natural index is `(user_id, created_at DESC)`:

```text
First narrow by user → Then order within that user
```

---

## 16. Leftmost Prefix

Index `(user_id, created_at, status)` is naturally useful for `WHERE user_id = ?` or `WHERE user_id = ? AND created_at > ?`, but not automatically ideal for `WHERE status = ?` alone, since `status` is the third column (see [Chapter 7 §14](07-indexes-query-performance.md)).

---

## 17. JOIN Performance

```sql
SELECT u.name, o.total
FROM users u
JOIN orders o ON o.user_id = u.id
WHERE u.id = '123';
```

Potentially good indexes: `users(id)`, `orders(user_id)`. Primary keys already have indexes, but FK columns don't automatically get every useful index. A common production mistake is forgetting indexes on frequently joined/filtering FK columns.

---

## 18. Nested Loop Join

```text
Nested Loop
```

```text
for each row from A: find matching rows in B
```

`A = 10 rows` with an efficient index on B → excellent. `A = 1 million`, `B = 100 million`, no efficient lookup → very expensive. Nested Loop is not inherently bad.

---

## 19. Hash Join

```text
Table A → Build hash table
Table B → Probe hash table
```

Good for many equality joins, e.g.:

```sql
SELECT * FROM users u JOIN orders o ON o.user_id = u.id;
```

---

## 20. Merge Join

```text
Sorted A + Sorted B → Merge
```

Useful when inputs are already sorted or can be efficiently sorted. PostgreSQL chooses among Nested Loop, Hash Join, Merge Join based on estimated cost.

---

## 21. A Very Important Example

`orders = 100 million rows`:

```sql
SELECT *
FROM orders
WHERE user_id = 'u123'
ORDER BY created_at DESC
LIMIT 20;
```

Weak: `CREATE INDEX idx_orders_user ON orders(user_id);` — finds the user's orders but may still need to sort many rows.

Better: `CREATE INDEX idx_orders_user_created ON orders(user_id, created_at DESC);` — find user → already ordered by created_at → take first 20. No large sort required. A classic production optimization.

---

## 22. OFFSET Pagination Problem

```sql
SELECT *
FROM orders
ORDER BY created_at DESC
LIMIT 20 OFFSET 1_000_000;
```

The database may still need to walk past a huge number of rows. Performance degrades as offset grows (`OFFSET 20 → 10,000 → 1,000,000`).

---

## 23. Keyset Pagination

```sql
SELECT *
FROM orders
WHERE created_at < :lastCreatedAt
ORDER BY created_at DESC
LIMIT 20;
```

with `CREATE INDEX idx_orders_created ON orders(created_at DESC);` — "give me the next 20 rows after this cursor." Scales much better for large datasets.

---

## 24. Tie-Breaker in Keyset Pagination

Don't rely only on `created_at` — multiple rows can share a timestamp.

```sql
SELECT *
FROM orders
WHERE (created_at, id) < (:createdAt, :id)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

```sql
CREATE INDEX idx_orders_created_id
ON orders(created_at DESC, id DESC);
```

Deterministic pagination.

---

## 25. Partial Index

`payments = 100 million`, only `2 million = SUCCESS`, and most queries filter `WHERE status = 'SUCCESS' AND user_id = ?`:

```sql
CREATE INDEX idx_payments_success_user
ON payments(user_id)
WHERE status = 'SUCCESS';
```

The index only contains relevant rows — dramatically smaller.

---

## 26. Partial Index + Query Matching

The query must logically match the partial-index predicate:

```sql
CREATE INDEX idx_active_users_email
ON users(email)
WHERE deleted_at IS NULL;
```

```sql
SELECT *
FROM users
WHERE email = 'x@example.com'
AND deleted_at IS NULL;
```

can use the index. Particularly useful for soft-delete systems (see [Chapter 10 §9](10-production-schema-design.md)).

---

## 27. LIKE Queries

`WHERE name LIKE 'Pulkit%'` — a normal B-tree index can potentially help with prefix searches (depending on collation/operator class). `WHERE name LIKE '%kit%'` starts with a wildcard — a normal B-tree generally can't efficiently use that.

For substring/fuzzy search, `pg_trgm`:

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;

CREATE INDEX idx_users_name_trgm
ON users
USING GIN(name gin_trgm_ops);
```

Now `WHERE name ILIKE '%kit%'` can be accelerated.

---

## 28. Statistics and Cardinality

`status: ACTIVE 99%, FAILED 1%` — low selectivity. `user_id` with millions of distinct users has much higher selectivity. The planner uses statistics to estimate how many rows a condition returns — **cardinality estimation**. Bad estimates produce bad plans.

---

## 29. Extended Statistics

Correlated columns like `country` and `city` (e.g. `country = India` implies certain cities) — PostgreSQL can use extended statistics to better estimate correlated columns:

```sql
CREATE STATISTICS user_location_stats
ON country, city
FROM users;
```

```sql
ANALYZE users;
```

Useful when planner estimates are consistently wrong due to column correlations.

---

## 30. CTE Optimization

```sql
WITH active_users AS (
    SELECT * FROM users WHERE status = 'ACTIVE'
)
SELECT * FROM active_users WHERE age > 25;
```

Modern PostgreSQL can often optimize/inline CTEs when appropriate. Explicit control:

```sql
WITH active_users AS MATERIALIZED (...)
WITH active_users AS NOT MATERIALIZED (...)
```

Don't assume "CTE always creates a temporary table" — outdated for modern PostgreSQL.

---

## 31. Query Optimization Workflow

1. Get the actual query.
2. Run `EXPLAIN (ANALYZE, BUFFERS) ...`
3. Look for `actual time`, `actual rows`, `estimated rows`, `loops`.
4. Check Seq Scan / Index Scan / Bitmap Scan / join strategy / sort.
5. Look for huge estimate mismatches.
6. Check indexes.
7. Check whether the query returns too much data.
8. Add/change index if justified.
9. Run `EXPLAIN ANALYZE` again.
10. Compare actual performance.

---

## 32. BUFFERS

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT ...
```

```text
Buffers:
  shared hit=1000
  shared read=200
```

`shared hit` — pages found in shared buffers/cache. `shared read` — pages read from storage into shared buffers. Helps distinguish CPU/query-plan problems from I/O-heavy queries.

---

## 33. Don't Optimize Blindly

Query takes 200ms; a complicated index brings it to 190ms but adds 20GB and slows writes — maybe not worth it. Optimization trades off read performance, write performance, storage, memory, and complexity. A very important Tech Lead mindset.

---

## 34. TypeORM QueryBuilder + Optimization

```ts
const orders = await repository
  .createQueryBuilder('order')
  .where('order.userId = :userId', { userId })
  .orderBy('order.createdAt', 'DESC')
  .take(20)
  .getMany();
```

If slow, don't blame TypeORM first:

```text
TypeORM → SQL → EXPLAIN ANALYZE → Execution plan
```

Then optimize the actual bottleneck.

---

## 35. Raw SQL Is Sometimes Better

For `DISTINCT ON`, `FOR UPDATE SKIP LOCKED`, complex window functions, special indexes — don't force everything through repository methods:

```ts
const result = await dataSource.query(`SELECT ...`);
```

> Don't use raw SQL because the ORM is unfamiliar. Use it because SQL is the right tool for the query.

---

## 36. The 100-Million-Row Question

> "You have a 100-million-row orders table. This query is slow: `WHERE user_id = ? ORDER BY created_at DESC LIMIT 20`. What do you do?"

1. `EXPLAIN ANALYZE`
2. Check estimated vs actual rows
3. Verify composite index `(user_id, created_at DESC)`
4. Check whether the query selects unnecessary columns
5. Consider covering/index-only strategy
6. Use keyset pagination for subsequent pages
7. Check table/index bloat and statistics
8. Consider partitioning only if the access/retention pattern justifies it

> Partitioning isn't the first answer — first fix the query and indexing.

---

## 37. The Senior-Level Mental Model

```text
Query
  ↓
Planner
  ↓
Execution Plan
  ↓
Rows estimated?
  ↓
Rows actually returned?
  ↓
Correct scan?
  ↓
Correct join?
  ↓
Correct index?
  ↓
Too much data?
  ↓
I/O?
  ↓
CPU?
  ↓
Fix
  ↓
Measure again
```

Not "add an index" but "let's inspect the execution plan and identify the bottleneck first." The difference between junior and senior optimization.

---

## Interview challenge

`orders = 200 million rows`:

```sql
SELECT id, total, created_at
FROM orders
WHERE user_id = $1
AND status = 'PAID'
ORDER BY created_at DESC
LIMIT 20;
```

Current index: `CREATE INDEX idx_orders_user ON orders(user_id);`

1. What composite index would you consider?
2. Should `status` be in the index?
3. Where should `created_at` go?
4. Could `INCLUDE` help?
5. What would you check in `EXPLAIN ANALYZE`?
6. Would you use OFFSET or keyset pagination for page 100,000?

---

## Next → Chapter 22 — Advanced PostgreSQL Features: JSONB, Full-Text Search, Arrays & Extensions
