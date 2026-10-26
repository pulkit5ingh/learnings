# Chapter 25 — Senior Backend / Tech Lead Capstone

The final chapter. Combining everything into one realistic production system: a high-scale e-commerce Order + Payment + Inventory platform using NestJS + TypeORM + PostgreSQL.

Not just "build tables" but: how would I design this system so it remains correct under concurrency, failures, retries, and high traffic?

---

## 1. Requirements

```text
Browse products → Create orders → Pay → Reserve inventory → Ship → Receive notifications
```

**Functional**: User, Product, Cart, Order, Payment, Inventory, Shipment, Notification.

**Non-functional**: high availability, concurrency safety, idempotency, horizontal scalability, reliable payments, no negative inventory, no duplicate charges, eventual consistency where acceptable, auditability.

---

## 2. High-Level Architecture

```text
                         Client
                           │
                           ▼
                     Load Balancer
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          API 1          API 2          API 3
             │             │             │
             └─────────────┼─────────────┘
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                  Redis       PostgreSQL
                                Primary
                                  │
                              Replicas
```

Async systems:

```text
PostgreSQL
    │
    ▼
Outbox
    │
    ▼
Message Broker
    │
 ┌──┼──────────┬──────────┐
 ▼  ▼          ▼          ▼
Payment     Inventory   Shipping  Notification
Worker      Worker      Worker     Worker
```

---

## 3. Database Design

Core tables: `users`, `products`, `inventory`, `orders`, `order_items`, `payments`, `shipments`, `outbox_events`, `processed_events`.

Potentially: `carts`, `cart_items`, `coupons`, `refunds`, `audit_logs`.

---

## 4. Users

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email TEXT NOT NULL,
    name TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX users_email_unique
ON users(email);
```

---

## 5. Products

```sql
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    price NUMERIC(12,2) NOT NULL,
    stock INTEGER NOT NULL DEFAULT 0,
    CHECK (price >= 0),
    CHECK (stock >= 0),
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

The database guarantees `stock >= 0` even if application code has a bug.

---

## 6. Orders

```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    status TEXT NOT NULL,
    total NUMERIC(12,2) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    CHECK (total >= 0)
);
```

States: `PENDING_PAYMENT`, `CONFIRMED`, `PROCESSING`, `SHIPPED`, `DELIVERED`, `CANCELLED`.

---

## 7. Order Items

```sql
CREATE TABLE order_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(id),
    product_id UUID NOT NULL REFERENCES products(id),
    quantity INTEGER NOT NULL,
    unit_price NUMERIC(12,2) NOT NULL,
    CHECK (quantity > 0),
    CHECK (unit_price >= 0)
);
```

Why store `unit_price` instead of reading `products.price`? Because product price can change — a customer buying at ₹1,000 today shouldn't see the order show ₹1,200 after tomorrow's price change.

> Order data must preserve historical business facts.

---

## 8. Payment

```sql
CREATE TABLE payments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(id),
    amount NUMERIC(12,2) NOT NULL,
    status TEXT NOT NULL,
    idempotency_key TEXT NOT NULL UNIQUE,
    provider_payment_id TEXT UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    CHECK (amount >= 0)
);
```

`idempotency_key`, `provider_payment_id`, `status` protect against duplicate payment operations.

---

## 9. Outbox

```sql
CREATE TABLE outbox_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_type TEXT NOT NULL,
    aggregate_id UUID NOT NULL,
    payload JSONB NOT NULL,
    status TEXT NOT NULL DEFAULT 'PENDING',
    retry_count INTEGER NOT NULL DEFAULT 0,
    next_attempt_at TIMESTAMPTZ,
    processed_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_outbox_pending
ON outbox_events(next_attempt_at, created_at)
WHERE processed_at IS NULL;
```

---

## 10. Processed Events

```sql
CREATE TABLE processed_events (
    event_id UUID PRIMARY KEY,
    processed_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

The primary key gives `event_id UNIQUE`, so duplicate processing can be detected safely.

---

## 11. Creating an Order

```http
POST /orders
```

```json
{ "items": [{ "productId": "p1", "quantity": 2 }] }
```

```text
BEGIN
1. Validate products
2. Reserve inventory
3. Calculate total
4. Create order
5. Create order items
6. Create payment
7. Create outbox event
8. COMMIT
```

---

## 12. Inventory Reservation

Don't do `SELECT stock → if stock >= quantity → UPDATE stock` — races. Instead:

```sql
UPDATE products
SET stock = stock - $1
WHERE id = $2
AND stock >= $1;
```

```ts
if (result.affected !== 1) {
  throw new OutOfStockError();
}
```

Atomic inventory reservation.

---

## 13. Two Customers Buy Last Item

`stock = 1`. Request A: `UPDATE ... stock >= 1` succeeds, stock → 0. Request B: same UPDATE fails.

```text
Customer A → success
Customer B → out of stock
```

Never `stock = -1`.

---

## 14. Creating the Order Transaction

```ts
await dataSource.transaction(async manager => {
  // Reserve inventory
  for (const item of items) {
    const result = await manager
      .createQueryBuilder()
      .update(Product)
      .set({ stock: () => 'stock - :quantity' })
      .setParameters({ quantity: item.quantity })
      .where('id = :id', { id: item.productId })
      .andWhere('stock >= :quantity', { quantity: item.quantity })
      .execute();

    if (result.affected !== 1) {
      throw new OutOfStockError();
    }
  }

  // Create order
  const order = await manager.save(Order, {
    userId,
    status: 'PENDING_PAYMENT',
    total,
  });

  // Create order items
  // Create payment
  // Create outbox event
});
```

Don't interpolate user-controlled values into SQL fragments — use parameters. For simple inventory updates, direct SQL can sometimes be clearer:

```sql
UPDATE products
SET stock = stock - $1
WHERE id = $2
AND stock >= $1;
```

---

## 15. Why We Don't Call Stripe Here

Don't do `BEGIN → Reserve inventory → Create order → Call Stripe → COMMIT`. If Stripe takes 5 seconds, the transaction holds a connection, locks, and transaction state for those 5 seconds.

Instead: `DB Transaction → COMMIT → Payment Worker → Stripe` (see [Chapter 14 §6](14-typeorm-transactions-concurrency.md), [Chapter 19](19-distributed-transactions-saga.md)).

---

## 16. Outbox Event

Inside the same DB transaction:

```json
{ "orderId": "order_123", "paymentId": "payment_123", "amount": 5000 }
```

Event type: `PAYMENT_REQUESTED`. Order created + Payment created + Outbox event are atomic.

---

## 17. Payment Worker

Receives `PAYMENT_REQUESTED`, calls the payment provider:

```http
POST /payment
Idempotency-Key: payment_123
```

On timeout, don't assume failure — retry using the same idempotency key.

---

## 18. Payment Webhook

`POST /webhooks/payment` starts a short DB transaction:

```text
BEGIN
Verify webhook
Check event hasn't already been processed
Lock/update payment
Validate payment state transition
Mark payment SUCCESS
Mark order CONFIRMED
Create ORDER_CONFIRMED outbox event
Record processed event
COMMIT
```

---

## 19. Duplicate Webhook

`PAYMENT_SUCCESS` sent twice — `event_123` already exists in `processed_events`, so the duplicate is ignored safely. **Idempotent event processing**.

---

## 20. Payment Failure

`PAYMENT_FAILED` — Saga compensation:

```text
Payment FAILED → Release inventory → Cancel order
```

```sql
UPDATE products
SET stock = stock + $1
WHERE id = $2;
```

Not a database rollback — business compensation.

---

## 21. Complete Saga

```text
                Create Order
                     │
                     ▼
              Reserve Inventory
                     │
                     ▼
              Request Payment
                     │
              ┌──────┴──────┐
              │             │
           SUCCESS        FAILURE
              │             │
              ▼             ▼
       Confirm Order    Release Stock
              │             │
              ▼             ▼
        Create Shipment Cancel Order
              │
              ▼
        Send Notification
```

Every step has retry, idempotency, state transition, failure handling.

---

## 22. Read Performance

```http
GET /users/:id/orders
```

```sql
SELECT *
FROM orders
WHERE user_id = $1
ORDER BY created_at DESC
LIMIT 20;
```

```sql
CREATE INDEX idx_orders_user_created
ON orders(user_id, created_at DESC);
```

Keyset pagination for large datasets instead of huge offsets.

---

## 23. Caching

```http
GET /products/123
```

```text
API → Redis → (miss) → PostgreSQL → Redis
```

On update: `PostgreSQL UPDATE → Redis DEL`. Cache-aside.

---

## 24. Read Replicas

Route to a Read Replica for non-critical reads: product catalog, historical reports, analytics.

Payment state, order immediately after creation, account balance may need primary reads because replica lag could expose stale data.

---

## 25. Partitioning

`orders = 2 billion`, mostly time-based access:

```text
orders_2026_01
orders_2026_02
orders_2026_03
...
```

Helps with partition pruning, retention, maintenance, large time-series datasets. Don't introduce partitioning simply because the table is "large" — first understand the access pattern.

---

## 26. Monitoring

API latency, DB query latency, DB connections, pool wait time, CPU, memory, disk, locks, deadlocks, replication lag, slow queries, outbox backlog, queue depth, payment failures, webhook failures.

**Outbox backlog** is especially useful — events going `10 → 100 → 1,000 → 10,000 → 100,000` signals broken downstream processing.

---

## 27. Failure Scenario — Stripe Is Down

Don't keep API requests waiting for Stripe. Instead:

```text
Create order → Payment PENDING → Outbox event → Worker retries → Stripe recovers → Payment succeeds
```

The system remains available for order creation, depending on business rules.

---

## 28. Another Failure — Stripe Charged, Webhook Never Arrives

Don't immediately refund or mark failed. Use reconciliation:

```text
Scheduled reconciliation job → Find PROCESSING payments → Query payment provider → Determine final state
```

A powerful production pattern.

---

## 29. Exactly-Once?

The architecture doesn't depend on true exactly-once execution. Instead: at-least-once delivery + idempotent consumers + idempotency keys + unique constraints = **effectively-once business behavior** for operations designed to be idempotent. Much more realistic (see [Chapter 19 §14-15](19-distributed-transactions-saga.md)).

---

## 30. Database Constraints Are Part of the Architecture

Don't rely only on NestJS validation. `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK` — e.g. `CHECK (stock >= 0)` alongside application-level `if (stock < quantity)`. Defense in depth.

---

## 31. TypeORM Architecture

```text
src/
│
├── modules/
│   ├── users/       (controller, service, entity, dto)
│   ├── orders/
│   ├── payments/
│   ├── inventory/
│   └── products/
│
├── database/
│   ├── migrations/
│   └── data-source
│
├── workers/
│   ├── payment.worker
│   └── outbox.worker
│
└── common/
    ├── errors/
    ├── logging/
    └── infrastructure/
```

---

## 32. Responsibility Boundaries

**Controller** — HTTP, DTO validation, authentication, response.

**Service** — business logic, transactions, state transitions.

**Repository / QueryBuilder** — database access.

**Worker** — async processing, retries, external integrations.

This separation makes the system easier to reason about.

---

## 33. Interview scenario — "A user clicks Pay twice. What happens?"

Not just "frontend disables the button." Layered defense:

```text
Frontend protection
+ Backend idempotency key
+ UNIQUE database constraint
+ Payment provider idempotency key
+ Idempotent webhook handler
```

Even if `request, request, request, webhook, webhook, webhook` arrive multiple times, the business operation happens only once.

---

## 34. Interview scenario — "Two users buy the last product. How do you prevent overselling?"

Atomic conditional UPDATE:

```sql
UPDATE products
SET stock = stock - quantity
WHERE id = ?
AND stock >= quantity
```

`affected = 1` → reservation succeeded. `affected = 0` → out of stock. Plus `CHECK(stock >= 0)` and transaction handling around the order workflow.

---

## 35. Interview scenario — "What happens if payment succeeds but your server crashes?"

```text
Payment provider → SUCCESS → Server crashes → Webhook eventually arrives → Idempotent webhook handler → Payment SUCCESS → Order CONFIRMED
```

If the webhook is delayed, a reconciliation job recovers the state.

> Webhook + reconciliation is stronger than relying only on the synchronous payment response.

---

## 36. Interview scenario — "How do you make your order event reliable?"

```text
DB transaction
   ├── Order
   ├── Payment
   └── Outbox Event
          ↓
       COMMIT
          ↓
      Outbox Worker
          ↓
       Message Broker
```

The outbox solves "DB committed but event wasn't published."

---

## 37. Final system

```text
                           CLIENT
                              │
                              ▼
                       LOAD BALANCER
                              │
                 ┌────────────┼────────────┐
                 ▼            ▼            ▼
               API           API          API
                 │            │            │
                 └────────────┼────────────┘
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
                  Redis             PostgreSQL
                                      │
                          ┌───────────┴───────────┐
                          ▼                       ▼
                       Primary                Replicas
                          │
                       Outbox
                          │
                          ▼
                    Message Broker
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
       Payment         Inventory        Shipping
        Worker          Worker           Worker
          │               │               │
          ▼               ▼               ▼
      Provider         Database        Carrier
```

With: transactions, indexes, constraints, idempotency, Saga, retries, DLQ, caching, partitioning, replication, monitoring, backups, PITR.

---

## The full PostgreSQL + TypeORM journey

```text
01  PostgreSQL Mental Model            14  TypeORM Transactions
02  Tables & CRUD                      15  Migrations
03  JOINs                              16  TypeORM Performance
04  Aggregations                       17  Advanced PostgreSQL
05  Relationships                      18  Production Project
06  Transactions & ACID                19  Saga & Distributed Transactions
07  Indexes                            20  Concurrency & Locking
08  EXPLAIN ANALYZE                    21  Query Optimization
09  Advanced SQL                       22  JSONB / FTS / Extensions
10  Production Schema Design           23  Scaling & Architecture
11  TypeORM Fundamentals               24  Production PostgreSQL
12  TypeORM Relationships              25  Senior Backend Capstone
13  QueryBuilder
```

Course complete.

Next phase for interview readiness: 50 Senior-level interview questions → SQL coding problems → concurrency problems → system design → debugging scenarios → mock Senior Backend interview.
