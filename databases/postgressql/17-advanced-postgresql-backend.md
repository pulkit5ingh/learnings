# Chapter 17 — Advanced PostgreSQL for Backend Engineers

Moving into PostgreSQL features that matter for production systems, not just CRUD APIs — connecting SQL, joins, transactions, indexes, and TypeORM into a coherent picture.

---

## 1. CTE — Common Table Expression

`WITH ...` creates a temporary named result set for the duration of one query.

> Find users whose total spending is above ₹10,000.

```sql
WITH user_totals AS (
    SELECT
        user_id,
        SUM(amount) AS total_spent
    FROM orders
    WHERE status = 'PAID'
    GROUP BY user_id
)
SELECT
    u.id,
    u.name,
    ut.total_spent
FROM users u
JOIN user_totals ut
    ON ut.user_id = u.id
WHERE ut.total_spent > 10000;
```

```text
orders → GROUP BY → user_totals → JOIN users → final result
```

---

## 2. Why CTEs are useful

Without one, complex SQL nests deeply:

```sql
SELECT ...
FROM users
WHERE id IN (
    SELECT ...
    FROM ...
    WHERE ...
);
```

CTEs name intermediate steps, improving readability, maintainability, complex reporting queries, recursive queries, and multi-stage transformations.

> A CTE is primarily a query-structuring tool — not automatically faster. Performance still depends on the query plan and PostgreSQL version/behavior.

---

## 3. CTE in TypeORM

QueryBuilder doesn't elegantly abstract every advanced construct — use raw SQL:

```ts
const result = await dataSource.query(`
  WITH user_totals AS (
    SELECT
      user_id,
      SUM(amount) AS total_spent
    FROM orders
    WHERE status = $1
    GROUP BY user_id
  )
  SELECT
    u.id,
    u.name,
    ut.total_spent
  FROM users u
  JOIN user_totals ut
    ON ut.user_id = u.id
  WHERE ut.total_spent > $2
`, ['PAID', 10000]);
```

`$1`, `$2` — still parameterized.

---

## 4. Window Functions

> Every order plus the customer's total spending.

`GROUP BY` collapses rows; a window function does not:

```sql
SELECT
    id,
    user_id,
    amount,
    SUM(amount) OVER (PARTITION BY user_id) AS user_total
FROM orders;
```

```text
order   user   amount   user_total
----------------------------------
101     1      500      2500
102     1      1000     2500
103     1      1000     2500
104     2      700      700
```

Every order remains a row.

---

## 5. GROUP BY vs Window Function

**GROUP BY**:

```sql
SELECT user_id, SUM(amount)
FROM orders
GROUP BY user_id;
```

```text
user_id | total
--------|------
1       | 2500
2       | 700
```

**Window function**:

```sql
SELECT id, user_id, amount, SUM(amount) OVER (PARTITION BY user_id)
FROM orders;
```

```text
id  user_id amount total
------------------------
1   1       500    2500
2   1       1000   2500
3   1       1000   2500
4   2       700    700
```

```text
GROUP BY         → collapse rows
WINDOW FUNCTION  → calculate across rows while keeping rows
```

(See also [Chapter 9 §7-9](09-window-functions-upsert-interview-queries.md).)

---

## 6. `ROW_NUMBER()`

Latest order for every user:

```sql
SELECT *
FROM (
    SELECT
        o.*,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY created_at DESC
        ) AS rn
    FROM orders o
) x
WHERE rn = 1;
```

`WHERE rn = 1` picks the latest per user.

---

## 7. Top N per group

> Give me the top 3 orders for each user.

```sql
SELECT *
FROM (
    SELECT
        o.*,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY amount DESC
        ) AS rn
    FROM orders o
) x
WHERE rn <= 3;
```

Much more powerful than plain `LIMIT 3`, which applies to the entire result rather than per group.

---

## 8. `RANK()` vs `ROW_NUMBER()` vs `DENSE_RANK()`

Salaries: `100000, 90000, 90000, 80000`.

```text
ROW_NUMBER: 1, 2, 3, 4
RANK:       1, 2, 2, 4
DENSE_RANK: 1, 2, 2, 3
```

```text
ROW_NUMBER → every row gets unique number
RANK       → ties share rank, gaps appear
DENSE_RANK → ties share rank, no gaps
```

---

## 9. `LAG()` and `LEAD()`

Each order's previous order amount:

```sql
SELECT
    id,
    user_id,
    amount,
    LAG(amount) OVER (
        PARTITION BY user_id
        ORDER BY created_at
    ) AS previous_amount
FROM orders;
```

```text
order   amount   previous
-------------------------
A       500      NULL
B       1000     500
C       700      1000
```

Useful for financial analysis, event sequences, user activity, price changes, time-series data.

---

## 10. JSONB

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    name TEXT NOT NULL,
    metadata JSONB
);
```

```json
{ "theme": "dark", "language": "en", "notifications": true }
```

---

## 11. Query JSONB

```sql
SELECT *
FROM users
WHERE metadata @> '{"theme": "dark"}';
```

`@>` means: does the left JSON contain the right JSON?

---

## 12. JSON field access

```sql
SELECT id, metadata->>'theme' AS theme
FROM users;
```

`metadata->>'theme'` returns text (`dark`).

---

## 13. JSONB in TypeORM

```ts
@Column({ type: 'jsonb', nullable: true })
metadata: Record<string, unknown>;
```

```ts
const users = await userRepository
  .createQueryBuilder('user')
  .where('user.metadata @> :metadata', {
    metadata: JSON.stringify({ theme: 'dark' }),
  })
  .getMany();
```

---

## 14. JSONB index

```sql
CREATE INDEX idx_users_metadata
ON users
USING GIN(metadata);
```

```text
B-tree → traditional scalar/range/order queries
GIN    → JSONB / arrays / certain search structures
```

---

## 15. Don't turn PostgreSQL into MongoDB

`users(id, data JSONB)` with everything dumped into `data` throws away relational advantages.

Use JSONB when the structure is genuinely dynamic or semi-structured. Use normal columns/tables when data has stable business meaning and relationships — `user.email`, `user.created_at`, `order.user_id`, `order.status` should generally be relational columns.

---

## 16. PostgreSQL ARRAY

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    skills TEXT[]
);

INSERT INTO users (id, skills)
VALUES (gen_random_uuid(), ARRAY['Node.js', 'PostgreSQL', 'Redis']);

SELECT * FROM users WHERE 'Node.js' = ANY(skills);
```

TypeORM:

```ts
@Column({ type: 'text', array: true })
skills: string[];
```

---

## 17. JSONB vs ARRAY vs separate table

**ARRAY** — good for a simple list, no metadata, no relationship: `skills = ['Node.js', 'Redis']`

**JSONB** — good for dynamic structure: `{ "Node.js": { "level": "advanced", "years": 5 } }`

**Separate table** — better if skills are actual entities needing relationships, metadata, indexing, referential integrity, or complex querying (`users`, `skills`, `user_skills`).

---

## 18. UPSERT

```sql
INSERT INTO users (email, name)
VALUES ('pulkit@example.com', 'Pulkit')
ON CONFLICT (email)
DO UPDATE SET name = EXCLUDED.name;
```

Atomic at the database level (see [Chapter 9 §18-20](09-window-functions-upsert-interview-queries.md)).

---

## 19. `EXCLUDED`

Represents the row you attempted to insert:

```sql
INSERT INTO users(email, name)
VALUES ('pulkit@example.com', 'New Name')
ON CONFLICT(email)
DO UPDATE SET name = EXCLUDED.name;
```

```text
existing row + new attempted row → conflict → EXCLUDED = attempted row
```

---

## 20. Full-text search

```sql
SELECT *
FROM articles
WHERE to_tsvector('english', title || ' ' || content)
    @@ plainto_tsquery('english', 'postgresql performance');
```

```text
text → tsvector → tsquery → search
```

For larger/more sophisticated search needs, specialized systems (Elasticsearch/OpenSearch) can still make sense.

---

## 21. GIN and full-text search

```sql
CREATE INDEX idx_articles_search
ON articles
USING GIN (to_tsvector('english', title || ' ' || content));
```

PostgreSQL can use this index for matching full-text queries.

---

## 22. `LIKE` vs full-text search

`WHERE name LIKE 'Pul%'` is different from `WHERE to_tsvector(...) @@ plainto_tsquery(...)`. Full-text search understands words, lexemes, linguistic normalization, relevance — not just substring matching.

For `'%pulkit%'`, a trigram index is needed for efficient substring search.

---

## 23. Trigram search

`pg_trgm` extension for fuzzy/substring matching:

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;

CREATE INDEX idx_users_name_trgm
ON users
USING GIN(name gin_trgm_ops);
```

Now `WHERE name ILIKE '%pul%'` can be accelerated. Particularly useful for autocomplete, fuzzy matching, names, product searches.

---

## 24. Materialized Views

Dashboard repeatedly calculates total sales, sales by country, sales by month, top products — expensive every time. A materialized view stores the result:

```sql
CREATE MATERIALIZED VIEW sales_summary AS
SELECT
    DATE_TRUNC('month', created_at) AS month,
    SUM(amount) AS total
FROM orders
WHERE status = 'PAID'
GROUP BY 1;
```

```sql
SELECT * FROM sales_summary;
```

Avoids recalculating the entire aggregation every time.

---

## 25. Refreshing materialized views

Not automatically current:

```sql
REFRESH MATERIALIZED VIEW sales_summary;
```

or, when requirements are satisfied:

```sql
REFRESH MATERIALIZED VIEW CONCURRENTLY sales_summary;
```

```text
Orders → expensive aggregation → Materialized View → Dashboard
```

Trades freshness for query speed.

---

## 26. Partitioning

`events` table with 5 billion rows, queries frequently filter by `created_at` — partition by date:

```text
events
 ├── events_2026_01
 ├── events_2026_02
 ├── events_2026_03
 ├── events_2026_04
 └── ...
```

Application still queries:

```sql
SELECT *
FROM events
WHERE created_at >= '2026-09-01'
  AND created_at < '2026-10-01';
```

PostgreSQL can scan only the relevant partition(s) — **partition pruning**.

---

## 27. When partitioning makes sense

Don't partition every table. Reasons: extremely large tables, time-series/event data, lifecycle management, dropping old data efficiently, queries naturally filter on the partition key. Common candidates: `audit_logs`, `events`, `clickstream`, `telemetry`. A table with 50,000 rows doesn't need it.

---

## 28. Database functions

```sql
CREATE FUNCTION calculate_discount(price NUMERIC)
RETURNS NUMERIC
AS $$
BEGIN
    RETURN price * 0.9;
END;
$$ LANGUAGE plpgsql;
```

```sql
SELECT calculate_discount(1000);
```

Don't put all business logic into database functions — use them only when database-side computation provides a real advantage.

---

## 29. Triggers

```text
INSERT → Trigger → audit_log
```

Useful for audit trails, database-level invariants, derived bookkeeping. But triggers make application behavior harder to understand — an `UPDATE` silently cascading through triggers into other tables and triggers makes debugging difficult.

**Tech Lead rule**: use triggers deliberately, not because they're available.

---

## 30. Advanced pagination

**Offset**: `LIMIT 20 OFFSET 100000` — simple, but increasingly expensive at large offsets.

**Cursor** (`created_at + id`):

```sql
WHERE (created_at, id) < (:cursorCreatedAt, :cursorId)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

Generally much better for infinite scroll, feeds, large datasets, high-volume APIs (see [Chapter 16 §10](16-typeorm-performance-architecture.md)).

---

## 31. PostgreSQL feature selection

| Requirement | PostgreSQL feature |
|---|---|
| Normal relational data | Tables |
| Relationships | FK |
| Flexible fields | JSONB |
| JSONB search | GIN |
| Simple lists | ARRAY |
| Text search | Full-text search |
| Fuzzy matching | pg_trgm |
| Complex transformations | CTE |
| Ranking | Window functions |
| Repeated expensive aggregation | Materialized view |
| Huge time-series table | Partitioning |
| Atomic insert/update | UPSERT |
| Concurrency | Locks/transactions |
| Historical integrity | Constraints |

---

## 32. TypeORM doesn't expose everything elegantly

TypeORM gives: repository, QueryBuilder, entity, relations, transactions, migrations. PostgreSQL gives far more: CTEs, window functions, JSONB, GIN, GiST, trigram, materialized views, partitioning, extensions, advanced locking.

Don't expect the ORM to perfectly abstract every PostgreSQL feature.

> Senior backend engineers need both ORM knowledge and SQL knowledge.

---

## 33. Example: real analytics query

> For each user, show their latest order, total spending, and rank them by spending.

```sql
WITH user_stats AS (
    SELECT
        user_id,
        SUM(amount) AS total_spent,
        MAX(created_at) AS latest_order_at
    FROM orders
    WHERE status = 'PAID'
    GROUP BY user_id
)
SELECT
    u.id,
    u.name,
    us.total_spent,
    us.latest_order_at,
    RANK() OVER (ORDER BY us.total_spent DESC) AS spending_rank
FROM users u
JOIN user_stats us
    ON us.user_id = u.id
ORDER BY us.total_spent DESC;
```

Combines: CTE + GROUP BY + JOIN + MAX + Window Function + RANK + ORDER BY.

---

## 34. The senior backend mindset

Not "can TypeORM do this?" but "what is the correct SQL/data-access strategy?"

```text
Can Repository API express it cleanly?
  YES → Repository
  NO →
    Can QueryBuilder express it clearly?
      YES → QueryBuilder
      NO / PostgreSQL-specific → Parameterized raw SQL
```

---

## Progression so far

```text
1. MongoDB → PostgreSQL          10. Production schema design
2. CRUD                          11. TypeORM fundamentals
3. JOINs / data types            12. TypeORM relationships
4. Aggregation                   13. QueryBuilder
5. Relationships                 14. Transactions + concurrency
6. Transactions                  15. Migrations
7. Indexes                       16. Performance architecture
8. EXPLAIN ANALYZE               17. Advanced PostgreSQL
9. Advanced SQL
```

Next: putting everything together in an actual production-grade NestJS project.

---

## Chapter 17 interview questions

### Q1
Difference between GROUP BY and Window Functions?

### Q2
When would you choose JSONB instead of a normal relational table?

### Q3
What is a GIN index useful for?

### Q4
What problem does partitioning solve?

### Q5
Difference between a View and a Materialized View?

### Q6
When would you use a CTE instead of a subquery?

### Q7 — Senior-level

`5 billion events` with `created_at`, `user_id`, `event_type`, `payload JSONB`. Queries usually:

```sql
WHERE user_id = ?
AND created_at BETWEEN ? AND ?
```

and sometimes `WHERE payload @> ...`. How would you think about partitioning + indexes + JSONB GIN + retention + query patterns before implementing anything?

---

## Next → Chapter 18 — Build a Production NestJS + PostgreSQL + TypeORM Project

Auth, Users, Products, Orders, Payments, Inventory — with entities → relations → migrations → DTOs → services → repositories → QueryBuilder → transactions → idempotency → locking → indexes → pagination → error handling → production architecture.
