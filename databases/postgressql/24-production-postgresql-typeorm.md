# Chapter 24 — Production PostgreSQL + TypeORM

How do I safely run PostgreSQL + TypeORM in production? Writing a query is one thing; running a database that handles real money, users, and traffic is another.

---

## 1. Production Architecture

```text
                     Internet
                         │
                         ▼
                   Load Balancer
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
           API 1       API 2       API 3
             │           │           │
             └───────────┼───────────┘
                         │
                    Connection Pool
                         │
                         ▼
                   PostgreSQL
                    Primary
                         │
                  WAL Replication
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
        Read Replica             Read Replica
```

Around it: Redis, message broker, object storage, monitoring, logging, backups.

---

## 2. TypeORM Production Configuration

```ts
TypeOrmModule.forRoot({
  type: 'postgres',
  host: process.env.DB_HOST,
  port: Number(process.env.DB_PORT),
  username: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,

  entities: [User, Order, Product],

  synchronize: false,
  migrationsRun: true,
  logging: false,
});
```

The most important setting: `synchronize: false`. Never rely on `synchronize: true` for production — TypeORM may attempt schema changes automatically, and production database changes should be controlled through migrations (see [Chapter 15 §3](15-typeorm-migrations-production.md)).

---

## 3. Production Migrations

```text
Migration 001: Create users
Migration 002: Create products
Migration 003: Create orders
Migration 004: Add payment status
Migration 005: Add index
```

Production has run `001–004`; deployment runs `005` and records it as applied.

---

## 4. Never Modify Applied Migrations

`1700000000000-create-users.ts` has already run in production — don't edit it. If it's edited, production has "Migration 001 = old version" while a developer's local copy differs, and different environments end up with different histories.

Instead, create a new migration (`002`, `003`, ...) for the change.

---

## 5. Expand → Migrate → Contract

One of the most important production deployment patterns. `users.name` → split into `first_name`/`last_name`.

**Step 1 — Expand**: add `first_name`, `last_name` while keeping `name`.

**Step 2 — Migrate**: backfill `name = "Pulkit Singh"` into `first_name = "Pulkit"`, `last_name = "Singh"`. Deploy code that understands both versions if necessary.

**Step 3 — Contract**: after all application instances use the new fields, `DROP name`.

Allows rolling deployments without breaking old/new application versions (see [Chapter 15 §13](15-typeorm-migrations-production.md)).

---

## 6. Zero-Downtime Migration

During deployment, `API v1, API v1, API v2, API v2` may run simultaneously. Schema changes need to be backward compatible, and ideally forward compatible, for the transition period — hence Expand → Deploy → Migrate → Contract.

---

## 7. Adding NOT NULL to a Huge Table

`users = 100 million`, want `email TEXT NOT NULL`. Don't blindly run `ALTER TABLE users ALTER COLUMN email SET NOT NULL;` without understanding cost and existing data.

Safer pattern: add nullable column → deploy code writing it → backfill existing rows (in batches, not one massive transaction) → verify no NULLs remain → add NOT NULL (see [Chapter 15 §11-12](15-typeorm-migrations-production.md)).

---

## 8. Index Creation in Production

`orders = 200 million`, need `CREATE INDEX idx_orders_user ON orders(user_id);`. A regular index creation can consume substantial resources and may block or interfere with writes.

```sql
CREATE INDEX CONCURRENTLY idx_orders_user
ON orders(user_id);
```

Minimizes blocking, but has its own operational tradeoffs — not universally better (see [Chapter 15 §16](15-typeorm-migrations-production.md)).

---

## 9. Connection Pool Exhaustion

One of the most common production problems. `20 API instances × pool size 20 = 400` potential connections, but PostgreSQL may only comfortably support far fewer active connections.

```text
Requests → Connection pool → No available connection → Requests wait → Timeout
```

The API can appear slow even when the SQL itself isn't slow (see [Chapter 16 §12-15](16-typeorm-performance-architecture.md)).

---

## 10. Slow Query vs Pool Wait

A critical distinction. API latency of 2 seconds doesn't necessarily mean SQL takes 2 seconds — it could be `Pool wait = 1.8 sec, SQL = 200 ms`. Monitoring should distinguish connection acquisition time from query execution time.

---

## 11. Transactions Holding Connections

```ts
await dataSource.transaction(async manager => {
  await manager.save(order);
  await stripe.charge(); // ❌ external network call
  await manager.save(payment);
});
```

Holds a DB connection + transaction + locks while waiting for Stripe. `100 requests → 100 long transactions → pool exhausted`. A classic production failure mode (see [Chapter 14 §6](14-typeorm-transactions-concurrency.md)).

---

## 12. Better Payment Architecture

```text
DB transaction: create order, create payment PENDING, create outbox event → COMMIT
        ↓
Payment worker → Stripe
        ↓
Webhook → short DB transaction → update payment/order
```

Keep database transactions short.

---

## 13. Monitoring

Important metrics: CPU, memory, disk I/O, disk space, connections, active queries, query latency, lock waits, deadlocks, transactions, cache hit ratio, replication lag, WAL generation, autovacuum activity, table/index bloat.

`CPU = 30%` doesn't mean healthy — the connection pool could be exhausted while the application is down.

---

## 14. Slow Query Monitoring

A query at `2 seconds` once per day isn't necessarily a huge issue. `50ms × 10 million calls` can matter far more. Think `latency × frequency`, not just individual query latency.

---

## 15. `pg_stat_statements`

An extension for query statistics: `query`, `calls`, `total_time`, `mean_time`, `rows`. Identifies most expensive queries, most frequently executed queries, and high total database time.

`2ms × 20 million calls` can consume more total database resources than `2 seconds × 100 calls`. Extremely useful for production optimization.

---

## 16. Locks in Production

API suddenly slow, queries found waiting — a lock problem:

```text
Transaction A → locks row → slow operation
Transaction B → waiting for same row
Transaction C → waiting
```

A queue forms. Lock monitoring matters (see [Chapter 20](20-advanced-concurrency-locking.md)).

---

## 17. Deadlocks

Transaction A: lock user → lock wallet. Transaction B: lock wallet → lock user. PostgreSQL detects the deadlock and aborts one transaction — the application should be prepared for transient errors like this.

---

## 18. Retry Carefully

Retryable: deadlock, serialization failure, temporary connection issue.

```ts
for (let attempt = 1; attempt <= 3; attempt++) {
  try {
    return await runTransaction();
  } catch (error) {
    if (!isRetryable(error)) {
      throw error;
    }
    await backoff(attempt);
  }
}
```

Don't retry: invalid email, foreign key violation, insufficient balance.

---

## 19. Backups

At minimum: logical backups, physical backups, point-in-time recovery. Exact approach depends on PostgreSQL hosting and operational requirements.

---

## 20. Point-in-Time Recovery

Someone accidentally deletes critical data at `11:35`. With WAL archiving + backups, restore to `11:34:59` instead of only last night's backup. **PITR**.

---

## 21. Backup ≠ Recovery

"Do you have backups?" isn't enough — "have we tested restoring them?" A backup that can't be restored isn't a reliable disaster-recovery strategy. Periodically test: backup → restore → verify.

---

## 22. RPO and RTO

**RPO — Recovery Point Objective**: how much data can we afford to lose? `RPO = 5 minutes` means losing up to five minutes might be acceptable.

**RTO — Recovery Time Objective**: how quickly must the service recover? `RTO = 30 minutes` means service should be restored within roughly 30 minutes.

---

## 23. Disaster Recovery

```text
Primary DB → Replication → Standby
```

plus backups, WAL, monitoring, failover, recovery procedures. Documented runbooks: `Primary failed → verify → promote standby → redirect application → verify writes → investigate failed node`.

---

## 24. Health Checks

```http
GET /health
```

Checks: application alive, database reachable, Redis reachable, message broker reachable. Don't make `/health` perform expensive operations.

---

## 25. Liveness vs Readiness

Especially important with Kubernetes.

**Liveness** — is the process alive?

**Readiness** — can this instance receive traffic? (`DB unavailable → not ready`)

Prevents a broken instance from receiving traffic.

---

## 26. Graceful Shutdown

Kubernetes sends `SIGTERM` — the application shouldn't disappear while requests are running:

```text
Stop accepting new requests → Finish active requests → Stop workers → Close Redis → Close DB pool → Exit
```

NestJS supports lifecycle hooks: `app.enableShutdownHooks();`

---

## 27. TypeORM + Graceful Shutdown

The DB connection pool should be properly closed on shutdown, or deployments can leave connections/operations in undesirable states.

```text
old instance: SIGTERM → drain requests → close DB resources → exit
new instance: starts → health check → receives traffic
```

---

## 28. Database Health vs Application Health

"PostgreSQL is reachable" ≠ "PostgreSQL is healthy." It could be reachable but have an exhausted connection pool, piling locks, nearly full disk, lagging replication, timing-out queries. Production monitoring needs more than a simple ping.

---

## 29. Disk Full Is a Serious Problem

`Disk = 99%` — PostgreSQL may start failing writes or other operations. Causes: huge tables, WAL accumulation, logs, temporary files, indexes, backups. Disk monitoring is essential — don't wait until 100%.

---

## 30. Autovacuum

MVCC means old row versions accumulate. **VACUUM / autovacuum** cleans up dead tuples and maintains table health. Don't disable autovacuum casually — poor vacuum behavior leads to table bloat, index bloat, poor performance, transaction ID issues.

---

## 31. Why DELETE Isn't Always "Gone"

```sql
DELETE FROM users WHERE id = 123;
```

MVCC means the old row version isn't necessarily physically removed immediately:

```text
DELETE → row becomes invisible → VACUUM eventually reclaims space
```

One reason PostgreSQL maintenance matters.

---

## 32. Soft Delete Has Its Own Cost

`users(id, deleted_at)` with millions of deleted rows. `WHERE deleted_at IS NULL` benefits from a partial index:

```sql
CREATE INDEX idx_users_active_email
ON users(email)
WHERE deleted_at IS NULL;
```

Soft delete doesn't physically remove data — for huge datasets, retention/archive strategies may still be necessary (see [Chapter 10 §7-9](10-production-schema-design.md)).

---

## 33. TypeORM Logging

Development: `logging: true` is useful. Production: `logging: false` or carefully controlled logging is preferable — high-volume SQL logging creates huge logs, I/O overhead, sensitive data exposure, difficult log analysis. Use dedicated query monitoring and structured application logs instead.

---

## 34. Transactions + Observability

Log useful identifiers: `requestId`, `userId`, `orderId`, transaction/business operation ID — e.g. `requestId=req_123 orderId=ord_456 paymentId=pay_789`. Trace: `HTTP request → service → transaction → outbox → worker → payment provider → webhook`. Extremely valuable in distributed systems.

---

## 35. Production Incident Example

"Checkout is very slow."

1. API latency: 3 sec
2. DB query: 200ms — SQL isn't the whole problem
3. Connection acquisition: 2.5 sec — DB pool exhausted

Cause: long-running transactions doing `DB update → Stripe API → wait 2 sec → commit`.

Fix: `DB transaction → commit → Stripe worker`. A real production-level diagnosis.

---

## 36. Production checklist

**Schema**: PKs, FKs, UNIQUE constraints, CHECK constraints, appropriate indexes, correct data types.

**TypeORM**: `synchronize=false`, migrations, connection pool configured, transaction-scoped manager, no accidental N+1, no uncontrolled query logging.

**Database**: backups, PITR strategy, monitoring, disk monitoring, connection monitoring, slow query monitoring, lock/deadlock monitoring, autovacuum.

**Application**: health checks, readiness/liveness, graceful shutdown, retry transient failures, idempotency, timeouts, structured logging.

---

## 37. Tech Lead interview question

> "Your production PostgreSQL database suddenly becomes slow. How do you investigate?"

> First I'd determine whether the bottleneck is query execution, connection acquisition, locks, CPU, memory, I/O, or replication. I'd check database and application metrics, active connections, pool utilization, slow queries, `pg_stat_statements`, and `EXPLAIN ANALYZE` for suspicious queries. I'd also check for lock waits, deadlocks, long-running transactions, disk pressure, and replication lag. I wouldn't immediately increase database resources because the root cause could be an application-level issue such as connection leaks, N+1 queries, or transactions holding connections while calling external services.

---

## 38. The complete production mental model

```text
                    ┌──────────────┐
                    │ Load Balancer│
                    └──────┬───────┘
                           ↓
                     NestJS APIs
                           ↓
                    Connection Pool
                           ↓
                    ┌──────────────┐
                    │ PostgreSQL   │
                    │   Primary    │
                    └──────┬───────┘
                           │
                    WAL Replication
                           │
                  ┌────────┴────────┐
                  ↓                 ↓
             Read Replica      Read Replica

             Redis ←──── API ────→ Workers

             Outbox → Queue → External Services
```

Around everything: monitoring, logging, tracing, backups, PITR, alerts, health checks.

---

## Chapter 24 cheat sheet

```text
synchronize: false
        ↓
Migrations
        ↓
Expand → Migrate → Contract
        ↓
Connection pooling
        ↓
Monitor pool exhaustion
        ↓
Short transactions
        ↓
Never hold DB transaction during external API calls
        ↓
Backups + PITR
        ↓
Monitor locks/deadlocks
        ↓
Monitor slow queries
        ↓
Test disaster recovery
```

---

## Next — Chapter 25: Senior Backend / Tech Lead Capstone

Designing a complete production system from scratch, applying everything: PostgreSQL, TypeORM, transactions, concurrency, indexes, query optimization, Redis, outbox, Saga, payments, inventory, queues, read replicas, partitioning, caching, idempotency, migrations, monitoring, disaster recovery — a Senior Backend interview-style system design where the architectural decisions are made rather than concepts merely recalled.
