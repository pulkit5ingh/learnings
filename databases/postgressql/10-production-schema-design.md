# Chapter 10 — Production PostgreSQL Schema Design

Thinking like a database designer, not just a query writer. Especially important for a Tech Lead interview, where the ask is often just "design the database" — interviewers want to see the reasoning.

---

## 1. Start with entities, not tables

Building a payment system — don't start with `CREATE TABLE ...`. First identify the domain:

```text
User
Order
Payment
PaymentAttempt
Refund
```

Then relationships:

```text
User
 ↓ 1:N
Order
 ↓ 1:N
PaymentAttempt
 ↓
Refund
```

Then ask: what data belongs to each entity? What must be unique? What can be NULL? What relationships exist? What data must never be deleted? What queries will be frequent? What needs transactions?

---

## 2. Normalization vs denormalization

**Normalized**:

```text
users
-----
id
name
email

orders
------
id
user_id
```

Instead of duplicating `user_name`/`user_email` on every order row.

---

## 3. Intentional duplication is sometimes correct

```text
products
--------
id
price

order_items
-----------
product_id
unit_price
quantity
```

Product price today (₹1500) doesn't mean the price when purchased was ₹1500 — historical business data should remain correct.

```text
Normalization + Intentional denormalization = Good production design
```

---

## 4. UUID vs BIGINT

A common interview question.

**BIGINT** — smaller than UUID, efficient indexes, sequential IDs can be storage/cache friendly, simple. Downsides: predictable IDs if exposed directly; globally generated IDs need more consideration in distributed systems.

**UUID** — globally unique, difficult to guess, useful when IDs are generated across distributed systems, doesn't expose simple sequential counts. Downsides: larger than BIGINT, indexes take more space, random UUIDs can have different locality characteristics depending on version/generation strategy.

---

## 5. What would I choose?

Not "UUID is always better." Instead:

> I'd choose based on whether IDs are exposed externally, how IDs are generated across services, index/storage requirements, and operational constraints.

Distributed backend/API with publicly exposed identifiers → UUIDs often practical. Internal high-throughput tables where compact sequential IDs help → BIGINT can be attractive.

---

## 6. UUID version matters

Modern systems may use UUID versions with better ordering characteristics than fully random UUIDs. PostgreSQL's UUID support has evolved, and the exact generation strategy matters.

```text
UUID ≠ always random ≠ always optimal
```

The important question: what are the access patterns and operational requirements?

---

## 7. Soft delete

```text
users
-----
id
name
email
```

Instead of `DELETE FROM users WHERE id = $1;`, add `deleted_at TIMESTAMPTZ`:

```sql
UPDATE users
SET deleted_at = NOW()
WHERE id = $1;
```

Normal query:

```sql
SELECT * FROM users WHERE deleted_at IS NULL;
```

---

## 8. Why soft delete

Useful when data must be recoverable, audit/history matters, accidental deletion must be reversible, or related records need historical references.

Cost: must remember `WHERE deleted_at IS NULL` everywhere appropriate, or a deleted user can still appear in the API.

---

## 9. Soft delete + UNIQUE is tricky

A very good interview question. `users.email UNIQUE` + `deleted_at`. User `pulkit@example.com` is soft-deleted, then a new user tries to register with the same email — a normal unique constraint still blocks it.

PostgreSQL's answer: **partial unique indexes**.

```sql
CREATE UNIQUE INDEX users_email_active_unique
ON users(email)
WHERE deleted_at IS NULL;
```

Uniqueness now applies only to active users.

---

## 10. `created_at` and `updated_at`

```sql
created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
```

Important detail: PostgreSQL doesn't automatically update `updated_at` just because `DEFAULT NOW()` is declared — `DEFAULT` only applies on insert. Updates need application logic, a trigger, or another explicit mechanism.

In TypeORM:

```typescript
@CreateDateColumn()
createdAt: Date;

@UpdateDateColumn()
updatedAt: Date;
```

(What TypeORM actually does here is examined later.)

---

## 11. ENUM vs VARCHAR

Order status: `PENDING`, `PAID`, `CANCELLED`, `REFUNDED`.

**ENUM**:

```sql
CREATE TYPE order_status AS ENUM (
    'PENDING', 'PAID', 'CANCELLED', 'REFUNDED'
);
```

```sql
status order_status NOT NULL
```

Database-level restriction.

---

## 12. ENUM trade-offs

Changing PostgreSQL enums can require migrations and careful operational handling. For rapidly changing business states, some teams prefer:

```sql
status VARCHAR(30) NOT NULL
    CHECK (status IN ('PENDING', 'PAID', 'CANCELLED', 'REFUNDED'))
```

or a lookup/reference table. Don't blindly choose ENUM because it looks strongly typed — think about how frequently allowed values change and how migrations are managed.

---

## 13. CHECK constraints

```sql
CREATE TABLE payments (
    id UUID PRIMARY KEY,
    amount NUMERIC(12,2) NOT NULL
        CHECK (amount >= 0)
);
```

The database protects the invariant directly.

---

## 14. Application validation vs DB constraints

NestJS `@IsEmail()` is good UX, but it doesn't replace `UNIQUE(email)`. Application validation = good UX; database constraint = integrity guarantee under concurrency.

> Use both where appropriate, for important invariants.

---

## 15. JSONB

Useful when some flexibility is needed without abandoning relational structure.

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    name TEXT NOT NULL,
    preferences JSONB
);
```

```json
{ "theme": "dark", "notifications": { "email": true, "sms": false } }
```

```sql
SELECT *
FROM users
WHERE preferences->>'theme' = 'dark';
```

---

## 16. Don't turn PostgreSQL into MongoDB

A common architectural mistake: `users(id, data JSONB)` with the entire user object dumped into `data`. Loses much of the benefit of relational modeling.

Better: relational columns + JSONB for genuinely dynamic attributes.

```text
users
-----
id
email
name
preferences JSONB
```

---

## 17. Foreign key strategy

`orders.user_id → users.id`. What happens on user deletion?

- **RESTRICT** — don't allow deletion if dependent rows exist. Good for financial records, historical data.
- **CASCADE** — delete children. Good when children have no meaning without the parent (e.g. temporary preferences).
- **SET NULL** — keep the child record but remove the association. Useful when historical records should remain but the parent may disappear.

---

## 18. Don't CASCADE financial data blindly

`User → Orders → Payments`. `ON DELETE CASCADE` everywhere means deleting one user could remove orders, payments, refunds. Dangerous. Financial/audit data usually needs stronger retention/integrity rules — exactly the reasoning interviewers look for.

---

## 19. Idempotency

`POST /payments` called twice due to client retry shouldn't charge ₹2000 instead of ₹1000. Use an idempotency key:

```sql
CREATE TABLE payments (
    id UUID PRIMARY KEY,
    order_id UUID NOT NULL,
    amount NUMERIC(12,2) NOT NULL,
    idempotency_key TEXT NOT NULL UNIQUE
);
```

```text
Request 1, idempotency_key = abc123 → INSERT → SUCCESS
Request 2, idempotency_key = abc123 → UNIQUE constraint → duplicate prevented
```

A database-enforced concurrency guarantee.

---

## 20. Why not check first?

Bad: `SELECT ... WHERE idempotency_key = 'abc123'` → not found → `INSERT payment`. Two requests can both observe "not found."

Instead: `INSERT ... ON CONFLICT (idempotency_key) ...` — let PostgreSQL handle the race.

```text
Unique constraint + Transaction + Concurrency + UPSERT = Production-safe design
```

---

## 21. Multi-tenancy

```sql
users (id, tenant_id, name)
orders (id, tenant_id, user_id, amount)
```

Most queries must include `WHERE tenant_id = $1`, and indexes should account for tenant access patterns:

```sql
CREATE INDEX idx_orders_tenant_user
ON orders(tenant_id, user_id);
```

---

## 22. The dangerous multi-tenant bug

```sql
SELECT * FROM orders WHERE user_id = $1;
```

If `user_id` isn't globally unique across tenants, this could leak Tenant A data to Tenant B. Tenant isolation must be designed into schema, queries, constraints, indexes, authorization, and tests. A serious Tech Lead concern.

---

## 23. Composite uniqueness in multi-tenancy

If email only needs to be unique within a tenant, don't use `UNIQUE(email)` — use `UNIQUE(tenant_id, email)`.

```text
Tenant A + pulkit@example.com → allowed
Tenant B + pulkit@example.com → allowed
Tenant A + pulkit@example.com (again) → rejected
```

A common real-world schema pattern.

---

## 24. Production schema example

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    email TEXT NOT NULL,
    name TEXT NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at TIMESTAMPTZ,

    CONSTRAINT users_tenant_email_unique
        UNIQUE (tenant_id, email)
);

CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    user_id UUID NOT NULL,
    status TEXT NOT NULL,
    total_amount NUMERIC(12,2) NOT NULL
        CHECK (total_amount >= 0),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    FOREIGN KEY (user_id)
        REFERENCES users(id)
);

CREATE INDEX idx_orders_tenant_user_created
ON orders (tenant_id, user_id, created_at DESC);
```

Already much closer to a real production schema.

---

## 25. How a Tech Lead approaches schema design

```text
1. Identify entities
       ↓
2. Identify relationships
       ↓
3. Identify invariants
       ↓
4. Choose data types
       ↓
5. Add constraints
       ↓
6. Decide delete behavior
       ↓
7. Identify query patterns
       ↓
8. Design indexes
       ↓
9. Consider transactions
       ↓
10. Consider scale
       ↓
11. Consider migrations
       ↓
12. Consider observability
```

---

## Chapter 10 challenge — payment system design

Requirements:

```text
1. A user can have many orders.
2. An order can have multiple payment attempts.
3. A payment attempt has: amount, status, provider, provider transaction ID, idempotency key.
4. An idempotency key must never be processed twice.
5. Provider transaction ID must be unique.
6. Payment records must not disappear when a user is deleted.
7. We need created_at and updated_at.
8. We frequently query: "Give me the latest payment attempts for an order."
9. Payment amount cannot be negative.
```

Design:

- **A.** What tables?
- **B.** What relationships?
- **C.** What constraints?
- **D.** What indexes?
- **E.** What should happen if a user is deleted?
- **F.** What columns should be unique?

Plain English or SQL is fine — perfection isn't the goal yet.

---

## Then we start TypeORM

## Chapter 11 — TypeORM Fundamentals

A real NestJS-style project: Entity → Column → Primary Key → Repository → Relations → Migration → QueryBuilder → Transaction → Locking → Performance.

Taught from a MongoDB/Mongoose perspective:

```text
Mongoose Model     ↕  TypeORM Entity
Mongoose Query     ↕  TypeORM Repository
Aggregation        ↕  QueryBuilder
populate()         ↕  relations / JOIN
Mongo transaction  ↕  TypeORM transaction
```
