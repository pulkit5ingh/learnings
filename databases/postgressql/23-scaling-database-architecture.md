# Chapter 23 — PostgreSQL Scaling & Database Architecture

Moving from writing and optimizing queries to: how do I keep PostgreSQL working when my application has millions of users and hundreds of millions/billions of rows? A common Senior Backend / Tech Lead interview area.

---

## 1. The Scaling Problem

```text
Client → NestJS → PostgreSQL
```

Easy at first. Then traffic grows:

```text
                 ┌── API Server
Client ── LB ────┼── API Server
                 ├── API Server
                 └── API Server
                       ↓
                   PostgreSQL
```

The database becomes the bottleneck — can't just keep adding API servers, because `10 API servers → 10,000 DB connections → 💥`. Database scaling requires a different strategy.

---

## 2. First Principle: Optimize Before Scaling

Before read replicas, sharding, partitioning, distributed databases — ask: is the query efficient? Are indexes correct? Are we returning too much data? Is there N+1? Is connection pooling configured? Is caching possible? Are transactions too long?

A bad query doesn't become good because you add replicas.

---

## 3. Connection Pooling

```text
API Requests → Connection Pool [C][C][C] → PostgreSQL
```

Instead of `Request → Create DB connection → Query → Destroy connection`, reuse connections (see [Chapter 16 §11](16-typeorm-performance-architecture.md)).

---

## 4. Pool Size

`10 API instances × 20 connections = 200`. Scale to `100 instances × 20 = 2,000` — can overwhelm PostgreSQL.

> Adding application servers can actually make database overload worse.

---

## 5. PgBouncer

```text
Applications → PgBouncer → PostgreSQL
```

A lightweight PostgreSQL connection pooler — PgBouncer manages the connection layer more efficiently than letting PostgreSQL handle thousands of application connections directly:

```text
5000 app connections → PgBouncer → 100 DB connections → PostgreSQL
```

Exact numbers depend on workload — not universal configuration values.

---

## 6. Pooling Modes

`session pooling`, `transaction pooling`, `statement pooling`.

**Session pooling** — connection stays associated with a client session.

**Transaction pooling** — connection returns to the pool after a transaction completes, supporting much higher connection reuse.

> Some PostgreSQL session-level features don't work the same way with transaction pooling.

Infrastructure choices need to consider application behavior.

---

## 7. Read Replicas

`90% reads, 10% writes` — introduce read replicas:

```text
                 ┌── Read Replica
Primary ─────────┼── Read Replica
                 └── Read Replica
```

`Writes → Primary`, `Reads → Replica`. **Read scaling**.

---

## 8. PostgreSQL Replication

```text
Primary → WAL → Replica
```

PostgreSQL's Write-Ahead Log records changes; replicas consume and apply WAL changes.

---

## 9. Replication Lag

User creates an order on Primary — order created. Immediately `SELECT order FROM replica;` — the replica may not have received the change yet:

```text
Primary: Order exists ✅
Replica: Order doesn't exist ❌
```

**Replication lag**.

---

## 10. Read-After-Write Problem

`POST /profile` writes to Primary; the frontend immediately calls `GET /profile`, which reads from a Replica — the user may temporarily see old data. **Read-after-write consistency problem**.

---

## 11. Solving Read-After-Write

- **Option 1** — read from primary immediately after a write.
- **Option 2** — sticky routing for a short period.
- **Option 3** — track replication position/LSN where appropriate.
- **Option 4** — design the UX to tolerate eventual consistency.

The correct solution depends on the business requirement.

---

## 12. Not Everything Should Go to Replicas

Good candidates: product catalog, public content, analytics, reports, search, historical data.

Bad candidates immediately after a write: payment status, order confirmation, account balance, inventory, wallet — where stale data could cause incorrect behavior.

---

## 13. Primary + Read Replicas Architecture

```text
                     ┌──────────────┐
                     │   Primary    │
                     │ PostgreSQL   │
                     └──────┬───────┘
                            │
                         Replication
                     ┌──────┴───────┐
                     ↓              ↓
                Replica 1       Replica 2
```

```text
Writes ─────────────→ Primary
Reads ──────────────→ Replicas
```

The application must understand consistency requirements.

---

## 14. Partitioning

`events = 5 billion rows` — a single huge table becomes hard to manage. PostgreSQL partitioning:

```text
events → Partition 2026-01, 2026-02, 2026-03, ...
```

Application still queries the logical parent:

```sql
SELECT *
FROM events
WHERE created_at >= '2026-09-01'
AND created_at < '2026-10-01';
```

PostgreSQL can scan only relevant partitions — **partition pruning** (see [Chapter 17 §26-27](17-advanced-postgresql-backend.md)).

---

## 15. Range Partitioning

Common for time-series data:

```sql
CREATE TABLE events (
    id BIGINT,
    created_at TIMESTAMPTZ,
    payload JSONB
) PARTITION BY RANGE (created_at);

CREATE TABLE events_2026_09
PARTITION OF events
FOR VALUES FROM ('2026-09-01')
TO ('2026-10-01');
```

Next month: `events_2026_10`, and so on.

---

## 16. Why Partitioning Helps

`events = 5 billion rows`, querying September 2026 — without partitioning, potentially enormous table/index structures; with partitioning, PostgreSQL eliminates irrelevant partitions:

```text
Partition pruning → Scan less data
```

---

## 17. Partitioning Is Not Sharding

**Partitioning** — data divided inside the database system: `One PostgreSQL database → Multiple partitions`.

**Sharding** — data distributed across multiple database instances/nodes: `DB Node 1, DB Node 2, DB Node 3, DB Node 4`.

```text
Partitioning = divide the table
Sharding = distribute data across databases
```

---

## 18. When Partitioning Is Useful

Huge tables, time-series events, logs, audit records, clickstream, transactions with retention windows — especially when queries naturally filter on `created_at`, `event_date`, `tenant_id`.

---

## 19. Partitioning for Data Retention

Retaining logs for 12 months — instead of `DELETE FROM events WHERE created_at < now() - interval '12 months';` touching millions/billions of rows, drop an old partition:

```text
DROP old partition
```

Dramatically more efficient. One reason time-based partitioning is popular for event/log data.

---

## 20. Partitioning Tradeoffs

Adds schema complexity, operational complexity, index management considerations, partition management, migration considerations. Don't partition a table with 50,000 rows just because partitioning sounds scalable.

---

## 21. Vertical Scaling

Before sharding, scale the PostgreSQL machine itself: `4 CPU / 16GB RAM → 16 CPU / 64GB RAM`. Can be surprisingly effective — database workloads often benefit substantially from RAM, fast storage, CPU, IOPS.

---

## 22. Horizontal Scaling

Instead of one bigger server, add more nodes: `DB 1, DB 2, DB 3`. For PostgreSQL this introduces more architectural complexity, since PostgreSQL isn't a simple shared-nothing distributed database where every query automatically spans nodes.

---

## 23. Sharding

`1 billion users` distributed:

```text
Shard 1 → users 1-10M
Shard 2 → users 10M-20M
Shard 3 → users 20M-30M
```

or by hash: `hash(user_id) % N`.

```text
Application → Shard Router → DB 1 / DB 2 / DB 3
```

---

## 24. Sharding Key

Extremely important choice. Multi-tenant SaaS: `tenant_id` — `tenant A → Shard 1, tenant B → Shard 2, tenant C → Shard 1`. Goal: distribute load, avoid hotspots, keep related data together.

---

## 25. Bad Shard Key

Almost all queries are `WHERE user_id = ?`, but sharded by `country` — queries don't know which shard has the user, ending up querying every shard (**scatter-gather**). Usually expensive.

---

## 26. Sharding Tradeoffs

Cross-shard joins, cross-shard transactions, data rebalancing, routing complexity, operational complexity, hot shards, migration complexity.

> Sharding should usually be a last-resort scaling strategy, not the first optimization.

---

## 27. Caching

Before sharding, caching often provides huge benefits:

```text
Client → API → Redis → (cache miss) → PostgreSQL
```

`GET /products/123` — first request: Redis miss → PostgreSQL → Redis SET. Next requests: Redis HIT. PostgreSQL receives far fewer reads.

---

## 28. Cache Invalidation

The famous problem: how do you invalidate stale data? `PostgreSQL: price = ₹1000`, `Redis: price = ₹1000`, then `UPDATE price → ₹1200` without invalidating Redis leaves `PostgreSQL = ₹1200, Redis = ₹1000` — stale data.

---

## 29. Cache-Aside Pattern

```text
GET → Redis → (miss) → PostgreSQL → Redis SET
```

On update: `UPDATE PostgreSQL → DELETE Redis key`.

```ts
await productRepository.update(id, { price });
await redis.del(`product:${id}`);
```

The next read repopulates the cache.

---

## 30. Don't Cache Everything

Good candidates: product catalog, configuration, public content, expensive computed results, frequently accessed reference data.

Be careful with: account balance, payment state, inventory, highly sensitive authorization state — unless the consistency model is explicitly designed.

---

## 31. Database Architecture

```text
                    Load Balancer
                         │
             ┌───────────┼───────────┐
             ↓           ↓           ↓
           API         API         API
             │           │           │
             └───────────┼───────────┘
                         │
                    Redis Cache
                         │
             ┌───────────┴───────────┐
             ↓                       ↓
         PostgreSQL               Read Replica
          Primary                     │
             │                        │
             └──── Replication ──────┘
```

Add Kafka/RabbitMQ, workers, object storage, search, monitoring as required.

---

## 32. Connection Pool + Replicas

```text
API → Connection Pool → DB Router → Primary / Replica
```

The data-access layer decides `write → primary`, `read → replica` — but consistency-sensitive reads may still go to primary. An architectural concern, not just a TypeORM configuration setting.

---

## 33. High Availability

Primary crashes → Standby → promoted to Primary (**failover**). A production HA system needs replication, health checks, leader/primary management, failover, backup, recovery procedures, monitoring.

Don't confuse replication with backup — they solve different problems.

---

## 34. Replication ≠ Backup

`DELETE FROM orders;` run accidentally on Primary — replication faithfully sends that change to replicas too:

```text
Primary → deleted
Replica → deleted
```

Replication doesn't protect against logical mistakes like this. Backups are still required.

---

## 35. Database Scaling Ladder

```text
Level 1: Optimize queries
Level 2: Correct indexes
Level 3: Connection pooling
Level 4: Caching
Level 5: Vertical scaling
Level 6: Read replicas
Level 7: Partitioning
Level 8: Sharding
```

Not a strict universal sequence, but a useful way to think about escalating complexity.

---

## 36. 100M Orders Example

`orders = 100 million`. Queries: get user's latest orders, today's orders, admin reporting, historical analytics.

Reasonable design: index `(user_id, created_at DESC)`; partition by `created_at` for huge time-oriented data; read replica for reports; Redis for frequently requested product/reference data; separate analytical system/warehouse for analytics.

> You don't necessarily solve every problem with one database feature.

---

## 37. Multi-Tenant Architecture

SaaS with 10,000 companies — `tenant_id` on most tenant-owned tables (`users`, `orders`, `projects`, `documents`, `payments`):

```text
orders(id, tenant_id, user_id, total, created_at)
```

```sql
CREATE INDEX idx_orders_tenant_created
ON orders(tenant_id, created_at DESC);
```

Especially important for SaaS workloads (see [Chapter 10 §21-23](10-production-schema-design.md)).

---

## 38. Row-Level Security

**RLS** — `Tenant A user → PostgreSQL → Only Tenant A rows`. Policies let the database itself enforce row visibility. Powerful for multi-tenant systems, but adds complexity around connection/session context, service accounts, migrations, admin access, debugging, performance. Use deliberately.

---

## 39. Tech Lead Interview Question

> "Your PostgreSQL database is overloaded. What would you do?"

Not immediately "add a read replica." Instead:

> First I'd identify whether the bottleneck is CPU, I/O, connections, locks, slow queries, or excessive traffic. I'd inspect query plans and database metrics, fix inefficient queries and indexes, review connection pool sizing, and add caching where appropriate. If the workload is read-heavy, I'd consider read replicas while explicitly handling replication lag. For very large tables with suitable access patterns, I'd evaluate partitioning. If the workload eventually exceeds a single PostgreSQL instance's practical capacity, I'd consider sharding, but only after understanding the operational and cross-shard tradeoffs.

---

## 40. Scaling cheat sheet

```text
Slow query                    → EXPLAIN ANALYZE
Too many connections          → Connection pooling / PgBouncer
Read-heavy workload           → Read replicas
Frequently requested data     → Redis
Huge time-series table        → Partitioning
Need data across DB nodes     → Sharding
Primary failure                → HA / failover
Accidental deletion            → Backups
Replica returns old data       → Replication lag
100 API instances              → Carefully size connection pools
Multi-tenant SaaS               → tenant_id + appropriate indexes
Database itself overloaded      → Measure first, then scale
```

---

## Senior-level mental model

```text
                  PostgreSQL
                      │
          ┌───────────┴───────────┐
          ↓                       ↓
     Query Scaling            Data Scaling
          │                       │
    ┌─────┼─────┐           ┌─────┼─────┐
    ↓     ↓     ↓           ↓     ↓     ↓
 Index  Cache  Pool      Replica Partition Shard
```

> Scaling is not just adding hardware. It's reducing unnecessary work, distributing necessary work, and preserving the consistency guarantees your business actually needs.

---

## Next — Chapter 24: Production PostgreSQL + TypeORM

Migrations & zero-downtime deployments, backups & point-in-time recovery, monitoring PostgreSQL, slow query monitoring, connection pool exhaustion, locks & deadlocks in production, transactions and failure recovery, TypeORM production configuration, health checks, graceful shutdown, database deployment strategy, disaster recovery, production incident debugging.

Then Chapter 25: the final Senior Backend / Tech Lead PostgreSQL + TypeORM system-design capstone.
