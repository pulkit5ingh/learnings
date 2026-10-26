# PostgreSQL for MongoDB Developers (Tech Lead Track)

Learning PostgreSQL + TypeORM by mapping everything back to MongoDB knowledge, aimed at Tech Lead-level interview depth: database design, transactions, indexes, locking, query optimization, concurrency, migrations, TypeORM architecture, and production patterns.

## Roadmap

### Part 1 — PostgreSQL fundamentals

1. MongoDB → PostgreSQL mental model
2. Tables, rows, columns, data types
3. CRUD + SQL fundamentals
4. Filtering, sorting, pagination
5. Joins — the biggest conceptual jump from MongoDB
6. Aggregations, `GROUP BY`, `HAVING`
7. Subqueries and CTEs
8. Constraints and relationships
9. Transactions & ACID
10. Indexes and query performance
11. `EXPLAIN` / `EXPLAIN ANALYZE`
12. PostgreSQL advanced features

### Part 2 — PostgreSQL system design

13. Database normalization
14. Schema design
15. One-to-one / one-to-many / many-to-many
16. Concurrency & isolation levels
17. Locks & deadlocks
18. Optimistic vs pessimistic locking
19. Pagination at scale
20. Partitioning
21. Replication
22. Connection pooling
23. Production database architecture

### Part 3 — PostgreSQL + TypeORM

24. TypeORM mental model
25. Entities
26. Relations
27. Repositories
28. QueryBuilder
29. Transactions
30. Migrations
31. Custom repositories/services
32. N+1 problems
33. TypeORM performance
34. TypeORM production architecture
35. NestJS + TypeORM

### Part 4 — Tech Lead interview

- SQL coding questions
- MongoDB → PostgreSQL migration questions
- TypeORM coding questions
- System-design questions
- Database architecture questions
- Production incident scenarios
- Mock Tech Lead interviews

---

## Chapter 1 — MongoDB → PostgreSQL Mental Model

The most important chapter for anyone coming from MongoDB.

> Think of PostgreSQL as a strongly structured relational database where relationships are first-class citizens.

### 1. MongoDB vs PostgreSQL

MongoDB:

```javascript
{
  _id: ObjectId("..."),
  name: "Pulkit",
  email: "pulkit@example.com",
  age: 28
}
```

```text
Database
 └── Collection
      ├── Document
      ├── Document
      └── Document
```

PostgreSQL:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255),
    age INTEGER
);
```

```text
Database
 └── Schema
      └── Table
           ├── Row
           ├── Row
           └── Row
```

#### Mapping table

| MongoDB | PostgreSQL |
|---|---|
| Database | Database |
| Collection | Table |
| Document | Row |
| Field | Column |
| `_id` | Primary Key |
| ObjectId | Usually UUID / BIGINT |
| Embedded document | JSON/JSONB or separate table |
| Reference | Foreign Key |
| Aggregation Pipeline | SQL |
| `$match` | `WHERE` |
| `$group` | `GROUP BY` |
| `$lookup` | `JOIN` |
| `$sort` | `ORDER BY` |
| `$limit` | `LIMIT` |
| `$skip` | `OFFSET` |
| Index | Index |
| Transaction | Transaction |

### 2. The biggest difference

MongoDB allows flexible documents per collection — different documents can have different shapes. PostgreSQL expects the table structure to be defined up front and every row follows that schema. This gives PostgreSQL something MongoDB doesn't emphasize as strongly:

> Data integrity enforced by the database itself.

### 3. Primary Key

MongoDB: `_id: ObjectId(...)`

PostgreSQL:

```sql
id BIGSERIAL PRIMARY KEY
```

or, increasingly:

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL
);
```

### 4. `NOT NULL`

MongoDB lets you omit a field entirely. PostgreSQL can enforce its presence:

```sql
email VARCHAR(255) NOT NULL
```

**Interview framing** — "Where should validation happen?"

> Application-level validation provides good user experience, but critical data integrity should also be enforced at the database level using constraints such as `NOT NULL`, `UNIQUE`, `CHECK`, `FOREIGN KEY`, and primary keys.

### 5. UNIQUE

MongoDB:

```javascript
db.users.createIndex({ email: 1 }, { unique: true })
```

PostgreSQL:

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL
);
```

A second insert with the same email fails due to the unique constraint.

### 6. Foreign Keys — very important

MongoDB doesn't inherently require a reference field (e.g. `userId`) to point to an existing document. PostgreSQL can enforce that relationship:

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL
);

CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    amount NUMERIC(10,2) NOT NULL,

    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

```text
orders.user_id
       │
       ▼
users.id
```

The database guarantees an order can't reference a nonexistent user.

### 7. JOIN = `$lookup`

MongoDB:

```javascript
db.orders.aggregate([
    {
        $lookup: {
            from: "users",
            localField: "userId",
            foreignField: "_id",
            as: "user"
        }
    }
])
```

PostgreSQL:

```sql
SELECT
    orders.id,
    orders.amount,
    users.name,
    users.email
FROM orders
JOIN users ON orders.user_id = users.id;
```

### 8. Filtering

MongoDB:

```javascript
db.users.find({ age: { $gt: 25 } });
db.users.find({ age: { $gte: 25, $lte: 35 } });
```

PostgreSQL:

```sql
SELECT * FROM users WHERE age > 25;

SELECT * FROM users WHERE age >= 25 AND age <= 35;
```

### 9. Sorting

MongoDB: `db.users.find().sort({ age: -1 });`

PostgreSQL: `SELECT * FROM users ORDER BY age DESC;` (or `ASC`)

### 10. Pagination

MongoDB:

```javascript
db.users.find().skip(20).limit(10);
```

PostgreSQL:

```sql
SELECT * FROM users LIMIT 10 OFFSET 20;
```

**First Tech Lead interview trap**: with 100M rows, `LIMIT 20 OFFSET 50000000` is expensive — PostgreSQL still has to scan and discard 50 million rows. Prefer **cursor/keyset pagination**:

```sql
SELECT * FROM users
WHERE id > 50000000
ORDER BY id
LIMIT 20;
```

(Covered in depth later in Part 2.)

### 11. Aggregation

MongoDB:

```javascript
db.orders.aggregate([
    { $group: { _id: "$userId", total: { $sum: "$amount" } } }
])
```

PostgreSQL:

```sql
SELECT user_id, SUM(amount) AS total
FROM orders
GROUP BY user_id;
```

### 12. The relational mindset shift

MongoDB mindset: "How should I structure my document so I can retrieve it efficiently?"

PostgreSQL mindset: "What are my entities, relationships, constraints, and access patterns?"

```text
User
 │
 ├── Profile
 │
 ├── Orders
 │     ├── OrderItems
 │     └── Payment
 │
 └── Addresses
```

→ `users`, `profiles`, `orders`, `order_items`, `payments`, `addresses` tables with relationships between them.

### 13. Example: e-commerce schema

```text
users
  | 1:N
  ↓
orders
  | 1:N
  ↓
order_items
  | N:1
  ↓
products
```

```sql
users
-----
id
name
email

orders
------
id
user_id
status
total_amount
created_at

order_items
-----------
id
order_id
product_id
quantity
price

products
--------
id
name
price
```

### 14. Why not put everything in one table?

A single flat `orders` table repeating `user_name`/`user_email` on every row duplicates that data across every order the user places, creating consistency problems when the user updates their info. This is the motivation for **normalization** (covered fully in Part 2):

> Normalization reduces unnecessary duplication and maintains data consistency.

MongoDB often intentionally denormalizes for read performance. PostgreSQL commonly starts with normalized relational models and selectively denormalizes when justified by workload.

### 15. PostgreSQL data types to know

```text
INTEGER, BIGINT, NUMERIC, BOOLEAN
VARCHAR, TEXT
DATE, TIMESTAMP, TIMESTAMPTZ
UUID
JSON, JSONB
ARRAY
```

**`TIMESTAMPTZ`** — for backend systems:

```sql
created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
```

Unlike MongoDB's single BSON Date type, PostgreSQL gives several temporal types, so choosing the right one matters.

**`JSONB`** — PostgreSQL isn't purely "rigid." It can still hold flexible data:

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    name TEXT,
    metadata JSONB
);

INSERT INTO users (id, name, metadata)
VALUES (
    gen_random_uuid(),
    'Pulkit',
    '{"theme":"dark","language":"en"}'
);

SELECT * FROM users WHERE metadata->>'theme' = 'dark';
```

So PostgreSQL can combine: relational data + JSONB + arrays + strong constraints + transactions + powerful querying — extremely useful for modern backend systems.

### 16. First interview question

Given `users` and `orders` where a user can have many orders — embed orders inside the user document (MongoDB-style), or use two tables joined by `orders.user_id → users.id`?

For a relational design, the two-table approach is the normal starting point since orders are an independent entity in a one-to-many relationship. But don't memorize "always normalize" — the Tech Lead-level answer weighs data ownership, cardinality, update patterns, query patterns, transaction boundaries, data growth, consistency requirements, and performance.

---

## Chapter 1 checkpoint

Be comfortable translating:

```text
MongoDB                         PostgreSQL

collection                 →   table
document                   →   row
field                      →   column
ObjectId                   →   UUID / BIGINT
$match                     →   WHERE
$lookup                    →   JOIN
$group                     →   GROUP BY
$sort                      →   ORDER BY
$limit                     →   LIMIT
$skip                      →   OFFSET
unique index                →   UNIQUE
reference                   →   FOREIGN KEY
embedded document           →   JSONB / separate table
aggregation pipeline        →   SQL
```

### Exercise

Given these MongoDB collections:

```javascript
users
{
    _id,
    name,
    email
}

orders
{
    _id,
    userId,
    amount,
    status,
    createdAt
}
```

Design the PostgreSQL equivalent:

1. `users` table
2. `orders` table
3. Primary keys
4. Foreign key
5. `amount` data type
6. `createdAt` data type
7. Query to get all orders of a particular user
8. Query to get each user's total order amount

Next: **Chapter 2 — PostgreSQL Tables, Data Types & CRUD**.
