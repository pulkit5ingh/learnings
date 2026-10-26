# Chapter 7 — PostgreSQL Indexes & Query Performance

A high-priority Tech Lead interview topic. Coming from MongoDB indexes, the concepts are easy — the important part is understanding **how PostgreSQL chooses an index and how to prove whether an index actually helps**.

---

## 1. What is an index?

```text
users
-----
id
name
email
age
```

10 million rows. Running:

```sql
SELECT *
FROM users
WHERE email = 'pulkit@example.com';
```

Without an appropriate index, PostgreSQL may need to inspect a huge portion of the table:

```text
10 million rows
       ↓
check row 1
check row 2
...
check row 10,000,000
```

That's a **Sequential Scan**. An index provides a separate data structure that helps PostgreSQL find matching rows efficiently — like a book's index versus reading every page.

---

## 2. Creating an index

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Now the earlier query can potentially use that index.

---

## 3. MongoDB comparison

MongoDB: `db.users.createIndex({ email: 1 });`

PostgreSQL: `CREATE INDEX idx_users_email ON users(email);`

Same fundamental idea.

---

## 4. Unique index

```sql
CREATE UNIQUE INDEX idx_users_email_unique
ON users(email);
```

If it's a business constraint, prefer expressing the intent as a `UNIQUE` constraint instead:

```sql
ALTER TABLE users
ADD CONSTRAINT users_email_unique
UNIQUE (email);
```

PostgreSQL creates an underlying unique index to enforce it: `UNIQUE constraint = database integrity + unique index`.

---

## 5. Primary key also gets an index

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY
);
```

PostgreSQL automatically creates an index to enforce the primary key's uniqueness — `CREATE INDEX idx_users_id ON users(id);` would be redundant.

---

## 6. B-Tree

The default PostgreSQL index type. `CREATE INDEX idx_users_email ON users(email);` normally produces a B-tree.

Useful for `=`, `>`, `>=`, `<`, `<=`, `BETWEEN`, `ORDER BY` — e.g. `WHERE age > 25` or `ORDER BY created_at DESC`.

---

## 7. Don't index everything

```text
users
-----
id, name, email, age, gender, country, status, created_at, updated_at
```

Should you index all 8 columns? Not necessarily — indexes have costs.

- **Read benefit**: `SELECT → faster lookup`
- **Write cost**: every `INSERT`/`UPDATE`/`DELETE` may need index maintenance

```text
More indexes → potentially faster reads + more storage + slower writes + more maintenance
```

> Create indexes based on actual query patterns, constraints, and measured performance — not simply on every column.

---

## 8. How do I know if PostgreSQL uses my index?

```sql
EXPLAIN
SELECT *
FROM users
WHERE email = 'pulkit@example.com';
```

Might show:

```text
Index Scan using idx_users_email on users
```

---

## 9. EXPLAIN ANALYZE

`EXPLAIN` shows the intended query plan. `EXPLAIN ANALYZE` actually executes the query and reports runtime information.

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'pulkit@example.com';
```

```text
Index Scan using idx_users_email on users
(actual time=0.03..0.04 rows=1)
```

**Production warning**: `EXPLAIN ANALYZE` executes the query — be careful with `EXPLAIN ANALYZE DELETE ...` or `EXPLAIN ANALYZE UPDATE ...` on production data. Understand the consequences before running it on writes.

---

## 10. Sequential Scan vs Index Scan

PostgreSQL could choose a Sequential Scan or an Index Scan for the same query. Don't assume "if I create an index, PostgreSQL must use it" — the optimizer chooses the plan it estimates will be cheapest.

---

## 11. Why might PostgreSQL NOT use an index?

A table with only 100 rows, query `WHERE age > 18`, and 95 of 100 rows match — scanning the whole table may be cheaper than using an index.

```text
Index exists ≠ Index will always be used
```

A very important interview point.

---

## 12. Selectivity

10 million users, `WHERE email = 'pulkit@example.com'` → probably one row matches → very selective.

`WHERE is_active = TRUE` might match 9 million rows → low selectivity. An index on a low-cardinality column may not always be useful for a particular query.

---

## 13. Composite indexes

```sql
SELECT *
FROM orders
WHERE user_id = 123
  AND status = 'PAID';
```

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

A **composite index** on `(user_id, status)`.

---

## 14. Column order matters

`CREATE INDEX idx_orders_user_status ON orders(user_id, status);` — order is `user_id → status`. Useful for `WHERE user_id = ...` and `WHERE user_id = ... AND status = ...`, but not automatically useful for `WHERE status = 'PAID'` alone, since `user_id` is the leading column.

This is the **leftmost-prefix principle** for B-tree composite indexes.

---

## 15. Think about your query patterns

```sql
SELECT *
FROM orders
WHERE user_id = $1
ORDER BY created_at DESC
LIMIT 20;
```

Instead of separate `user_id` and `created_at` indexes, consider:

```sql
CREATE INDEX idx_orders_user_created
ON orders(user_id, created_at DESC);
```

Aligns the index with the actual access pattern — much more useful than "index every column."

---

## 16. Index for the query, not the table

A great Tech Lead principle. Don't ask "what indexes should this table have?" — ask "what are our important queries and access patterns?"

```text
GET /users/:id/orders
```

```sql
SELECT *
FROM orders
WHERE user_id = $1
ORDER BY created_at DESC
LIMIT 20;
```

Design the index around that query.

---

## 17. Partial indexes

```sql
SELECT *
FROM payments
WHERE status = 'SUCCESS'
  AND user_id = $1;
```

Instead of indexing every payment:

```sql
CREATE INDEX idx_success_payments_user
ON payments(user_id)
WHERE status = 'SUCCESS';
```

The index only contains rows matching `status = 'SUCCESS'` — significantly smaller.

---

## 18. Partial index example

`100 million payments — SUCCESS → 10 million, FAILED → 90 million`. If the app almost exclusively queries successful payments:

```sql
CREATE INDEX idx_success_payments
ON payments(user_id, created_at DESC)
WHERE status = 'SUCCESS';
```

A very nice PostgreSQL-specific optimization to know for interviews.

---

## 19. JSONB indexes

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    metadata JSONB
);
```

```sql
SELECT *
FROM users
WHERE metadata @> '{"theme":"dark"}';
```

For JSONB-heavy querying, PostgreSQL commonly uses **GIN** indexes:

```sql
CREATE INDEX idx_users_metadata
ON users
USING GIN (metadata);
```

---

## 20. B-tree vs GIN

**B-tree** — great for `=`, `>`, `<`, `>=`, `<=`, `ORDER BY`, ranges. Typical: `CREATE INDEX ... ON users(email);`

**GIN** — useful for JSONB, arrays, full-text search scenarios. `CREATE INDEX idx_users_metadata ON users USING GIN(metadata);`

---

## 21. Index and `LIKE`

`WHERE email LIKE 'pulkit%'` — a B-tree index can potentially help with a left-anchored pattern.

`WHERE email LIKE '%gmail.com'` — different problem, wildcard at the start. For substring searches, trigram indexes may be appropriate (PostgreSQL's `pg_trgm` extension).

---

## 22. A production debugging scenario

> "An API endpoint used to respond in 50ms. Now it takes 5 seconds. How would you investigate?"

Don't immediately say "I'll add an index." Instead:

1. **Identify the actual SQL** the application generates.
2. **Run `EXPLAIN ANALYZE`**.
3. **Look for**: Sequential Scan, large row counts, unexpected joins, bad estimates, large sorts, high execution time.
4. **Check indexes** — does the query have an appropriate one?
5. **Check statistics** — PostgreSQL's planner relies on statistics; run `ANALYZE users;`. Usually autovacuum/autoanalyze handles this, but manual `ANALYZE` helps diagnose stale statistics.
6. **Check data growth** — maybe `1 million → 100 million rows` and the old query no longer scales.
7. **Check application behavior** — maybe the DB query is fine but the API runs 1000 queries instead of 1. The classic **N+1 problem**.

---

## 23. N+1 problem

Fetch 100 users, then for each user run `SELECT orders WHERE user_id = ?` → `1 + 100 = 101 queries`. Can kill performance.

Use a JOIN instead:

```sql
SELECT
    u.id,
    u.name,
    o.id AS order_id,
    o.amount
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id;
```

Or another carefully designed query. Covered in depth again with TypeORM.

---

## 24. Index doesn't fix everything

```sql
SELECT * FROM orders WHERE user_id = $1;
```

with `CREATE INDEX idx_orders_user ON orders(user_id);` already in place — still slow. Possible reasons:

```text
1. Huge number of matching rows
2. Query needs many columns/pages
3. Poor query design
4. Expensive JOIN
5. Sorting
6. Aggregation
7. Lock contention
8. Disk/I/O pressure
9. Bad planner estimates
10. Application-level N+1
```

> Performance tuning starts with measurement, not guessing.

---

## 25. Index trade-off — interview answer

> "Why don't we put an index on every column?"

> Indexes improve some reads but consume storage and require maintenance during inserts, updates and deletes. Too many indexes can increase write latency and may not provide any benefit. Indexes should therefore be based on actual query patterns and measured performance.

---

## 26. TypeORM indexes

```typescript
@Entity('users')
@Index(['email'], { unique: true })
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  email: string;
}
```

Composite:

```typescript
@Entity('orders')
@Index(['userId', 'createdAt'])
export class Order {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  userId: string;

  @Column()
  createdAt: Date;
}
```

> TypeORM decorators are only a way to define the database schema. You still need to understand what PostgreSQL index that produces and whether it matches your query.

---

## Chapter 7 challenge

```text
orders
--------------------------------
id
user_id
status
amount
created_at
```

Executed millions of times per day:

```sql
SELECT id, amount, status, created_at
FROM orders
WHERE user_id = $1
  AND status = 'PAID'
ORDER BY created_at DESC
LIMIT 20;
```

### Q1
What index would you consider?

### Q2
Why does column order matter?

### Q3
Would you instead create:

```sql
CREATE INDEX idx1 ON orders(user_id);
CREATE INDEX idx2 ON orders(status);
CREATE INDEX idx3 ON orders(created_at);
```

Explain your reasoning.

### Q4
The query suddenly becomes slow after the table grows from 1 million to 100 million rows. What would you inspect before changing the schema?

### Q5 — TypeORM
How would you represent the composite index `(user_id, created_at)` in a TypeORM entity?

---

## Next chapter: `EXPLAIN ANALYZE` in depth

Reading PostgreSQL query plans: `Seq Scan`, `Index Scan`, `Index Only Scan`, `Bitmap Heap Scan`, `Nested Loop`, `Hash Join`, `Merge Join`, `Sort`, `Aggregate`, `cost`, `rows`, `actual time`.

This is where the thinking shifts from writing SQL to debugging a production PostgreSQL database.
