# Chapter 16 — TypeORM Performance & Production Architecture

Combining PostgreSQL + TypeORM + backend architecture. Not "how do I make TypeORM faster?" but: how do I design a TypeORM-based service that remains fast and predictable as traffic and data grow?

---

## 1. The TypeORM performance stack

```text
API
 ↓
Service
 ↓
TypeORM
 ↓
Generated SQL
 ↓
PostgreSQL
 ↓
Indexes / Query Plan
 ↓
Disk / Memory
```

A 2-second endpoint isn't automatically TypeORM's fault. Could be: N+1 queries, missing index, bad JOIN, large result set, OFFSET pagination, connection pool exhaustion, lock contention, slow PostgreSQL query, network latency, serialization.

---

## 2. First rule: measure before optimizing

```ts
const orders = await orderRepository.find({ relations: { items: true } });
```

Before rewriting, determine: how many SQL queries? How much time per query? How many rows? What execution plan? Are indexes being used?

```text
Slow API → Find SQL → Measure query time → EXPLAIN ANALYZE → Inspect indexes → Inspect result size → Fix → Measure again
```

---

## 3. N+1 — the classic ORM problem

```ts
const users = await userRepository.find();

for (const user of users) {
  user.orders = await orderRepository.find({ where: { user: { id: user.id } } });
}
```

1,000 users → `1 query + 1,000 queries = 1,001 queries`. Instead:

```ts
const users = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.orders', 'order')
  .getMany();
```

Potentially 1 SQL query.

---

## 4. But JOINs can create another problem

100 users × 1,000 orders each = a JOIN can produce 100,000 rows, which TypeORM then hydrates into entities.

> Eliminating N+1 doesn't mean "JOIN everything." Still control: selected columns, number of rows, pagination, relation depth.

---

## 5. Select only what you need

Bad: `userRepository.find();` when the endpoint only needs `id`, `name`. Prefer:

```ts
const users = await userRepository
  .createQueryBuilder('user')
  .select(['user.id', 'user.name'])
  .getMany();
```

Or a DTO/projection strategy. Retrieving unnecessary columns (`password_hash`, `metadata JSONB`, `large_profile_data`, ...) wastes DB I/O, network, memory, and serialization.

---

## 6. Entity hydration has a cost

`.getMany()` maps DB rows into entity objects — costs CPU/memory for large result sets. For reporting/aggregation, prefer `.getRawMany()`:

```ts
const result = await orderRepository
  .createQueryBuilder('order')
  .select('order.status', 'status')
  .addSelect('COUNT(*)', 'count')
  .groupBy('order.status')
  .getRawMany();
```

No need for 1 million `Order` objects just to calculate counts.

---

## 7. Repository vs QueryBuilder

**Simple query** → repository API: `repository.find({ where: { status: 'PAID' } });`

**Complex query** → QueryBuilder: `.join(...).where(...).groupBy(...).having(...)`

**Specialized PostgreSQL query** → consider raw SQL. Don't force the ORM to express something that becomes unreadable.

---

## 8. Indexes are still PostgreSQL's responsibility

TypeORM doesn't magically make queries fast.

```ts
repository.find({ where: { userId, status: 'PAID' } });
```

may benefit from:

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

```ts
@Index('idx_orders_user_status', ['userId', 'status'])
@Entity('orders')
export class Order { ... }
```

Don't create indexes just because a column exists — indexes have costs (see [Chapter 7](07-indexes-query-performance.md)).

---

## 9. Composite index ordering

```sql
CREATE INDEX idx_orders_user_status_created
ON orders(user_id, status, created_at DESC);
```

Useful for `WHERE user_id = ? AND status = ? ORDER BY created_at DESC`. Index design should come from actual query patterns:

```text
Query pattern → WHERE → ORDER BY → Index design
```

Not: `Entity columns → Index everything`.

---

## 10. Pagination performance

`.skip(100000).take(20)` is offset pagination — at large offsets PostgreSQL still processes/skips many rows. Use keyset pagination for large datasets:

```ts
const orders = await orderRepository
  .createQueryBuilder('order')
  .where('(order.created_at, order.id) < (:createdAt, :id)', {
    createdAt: cursor.createdAt,
    id: cursor.id,
  })
  .orderBy('order.created_at', 'DESC')
  .addOrderBy('order.id', 'DESC')
  .take(20)
  .getMany();
```

Cursor = `created_at + id`. `id` as a tie-breaker matters because multiple rows can share a timestamp.

---

## 11. Connection pooling

```text
Node.js
   ↓
Connection Pool
   ├── connection 1
   ├── connection 2
   ├── ...
   └── connection N
          ↓
      PostgreSQL
```

TypeORM uses the underlying PostgreSQL driver's connection pool rather than opening a new connection per request.

---

## 12. Why connection pools matter

`10,000 requests/sec` shouldn't mean `10,000 PostgreSQL connections` — PostgreSQL can't efficiently handle unlimited connections.

```text
10,000 API requests → connection pool → limited DB connections
```

Requests wait when all connections are busy.

---

## 13. Pool sizing

"More connections = more performance" is wrong. With `16 CPU cores`, hundreds/thousands of active connections cause context switching, memory pressure, contention, increased latency.

Pool size should be based on DB capacity, query latency, application instances, workload, CPU/memory, connection limits.

> Pool size is per application instance.

`10 API instances × poolSize 20 = 200` database connections. A classic production architecture consideration.

---

## 14. TypeORM pool configuration

```ts
TypeOrmModule.forRoot({
  type: 'postgres',
  host, port, username, password, database,
  extra: { max: 20 },
});
```

Exact configuration depends on the PostgreSQL driver and deployment architecture — don't blindly copy `max: 20` into production.

---

## 15. Connection pool exhaustion

`Pool = 20 connections`, each request holds a connection for 5 seconds → soon all 20 are busy, new requests wait. API becomes slow even at `CPU = 30%`.

Long transactions are dangerous, especially:

```ts
await dataSource.transaction(async manager => {
  await manager.save(...);
  await externalApiCall(); // 🚨
  await manager.update(...);
});
```

Ties up a DB connection while waiting on the network (see [Chapter 14 §6](14-typeorm-transactions-concurrency.md)).

---

## 16. Slow query vs pool problem

**Slow query**: DB query = 3 seconds.

**Pool exhaustion**: DB query = 20ms, but the request waits 2 seconds for a connection.

Need metrics to distinguish them.

---

## 17. Logging

Development: `logging: ['query', 'error']` is useful. In production, don't blindly log every SQL query/parameter/request — creates huge logs, sensitive-data exposure, additional overhead, and harder debugging. Use structured logging and targeted slow-query monitoring instead.

---

## 18. Slow query logging

Endpoint goes from 2s to 8s:

```text
API latency → DB query duration → slow query → EXPLAIN ANALYZE
```

Then check: Seq Scan? Bad JOIN? Wrong row estimates? Missing index? Huge sort? Large result? Lock wait? (Connects directly to [Chapter 8](08-explain-analyze-query-optimization.md).)

---

## 19. TypeORM + EXPLAIN ANALYZE

```sql
SELECT ...
FROM orders
WHERE user_id = '123'
ORDER BY created_at DESC
LIMIT 20;
```

```sql
EXPLAIN ANALYZE
SELECT ...
FROM orders
WHERE user_id = '123'
ORDER BY created_at DESC
LIMIT 20;
```

Might reveal a Seq Scan instead of an index scan. Create a suitable index:

```sql
CREATE INDEX idx_orders_user_created
ON orders(user_id, created_at DESC);
```

Then measure again.

---

## 20. Transactions and performance

Transactions aren't free — can hold locks, hold a connection, create contention, increase latency.

> Keep transactions short and focused.

Good: `BEGIN → update payment → update order → create ledger → COMMIT`

Bad: `BEGIN → query DB → call Stripe → call another API → send email → generate PDF → wait → update DB → COMMIT`

---

## 21. Cache the right things

`GET /products/123` where product data changes rarely:

```text
API → Redis → (cache miss) → PostgreSQL
```

Don't cache everything — introduces cache invalidation, stale data, memory cost, consistency questions. Careless caching of rapidly changing data like wallet balances is dangerous.

---

## 22. Read-heavy architecture

```text
              ┌── Redis
              │
API ──────────┼── PostgreSQL
              │
              └── Read Replica
```

`writes → primary`, `reads → replica` — but introduces replication lag:

```text
WRITE primary → replication → READ replica
```

The read may temporarily not see the latest write. Don't blindly send every read to a replica.

---

## 23. Repository architecture

```text
Controller → Service → Repository → PostgreSQL
```

```ts
@Controller('orders')
export class OrdersController {
  constructor(private readonly ordersService: OrdersService) {}

  @Get()
  findAll() {
    return this.ordersService.findAll();
  }
}

@Injectable()
export class OrdersService {
  constructor(
    @InjectRepository(Order)
    private readonly orderRepository: Repository<Order>,
  ) {}

  async findAll() {
    return this.orderRepository.find();
  }
}
```

---

## 24. Don't put SQL in controllers

Avoid:

```ts
@Get()
async getOrders() {
  return this.dataSource.query(`SELECT ...`);
}
```

Controllers handle HTTP, validation, auth, response. Business and persistence logic belong lower in the stack.

---

## 25. Service layer

```ts
async createOrder(userId: string, dto: CreateOrderDto) {
  // validate business rules
  // transaction
  // inventory
  // payment state
  // persistence
}
```

The service owns the business operation.

---

## 26. Repository abstraction — don't overdo it

```text
Controller → Service → OrderRepository → TypeORM Repository → PostgreSQL
```

Useful for a domain-specific persistence abstraction, but don't create `IUserRepository`, `UserRepository`, `UserRepositoryImpl`, `TypeOrmUserRepository`, `UserRepositoryFactory`, `UserRepositoryAdapter` for a trivial CRUD application. Architecture for architecture's sake — use abstraction where it provides real value.

---

## 27. TypeORM and raw SQL

```ts
const result = await dataSource.query(
  `SELECT ... FROM ... WHERE user_id = $1`,
  [userId],
);
```

`$1` + `[userId]` — parameterized. Never `WHERE user_id = '${userId}'`.

---

## 28. When should you bypass the ORM?

- **Complex analytics** — window functions, CTEs, advanced PostgreSQL features
- **Performance-critical query** — QueryBuilder produces unnecessarily complicated SQL
- **Database-specific features** — PostgreSQL capabilities matter more than ORM portability
- **Bulk operations** — direct SQL is substantially more efficient

> Use ORM where it improves productivity; use SQL where SQL gives better control.

---

## 29. TypeORM isn't your database architecture

Don't design "TypeORM first." Instead:

```text
Business domain → Data model → Access patterns → PostgreSQL schema → Indexes → Transactions → TypeORM implementation
```

Database design shouldn't be constrained by whatever is easiest to express in TypeORM.

---

## 30. Production request example

```http
GET /users/123/orders?page=1
```

```text
Request → Controller → Auth → OrdersService → QueryBuilder → PostgreSQL
  → Index (user_id, created_at DESC) → 20 rows → DTO → Response
```

If slow: `Measure → Generated SQL → EXPLAIN ANALYZE → Index → Rows → Connection pool → Serialization`

---

## 31. Production anti-patterns

- ❌ **Fetch everything** — `repository.find();` on a table with millions of rows
- ❌ **N+1** — `for (...) { await repository.find(...); }`
- ❌ **Deep relations everywhere** — `relations: { orders: { items: { product: { category: true } } } }` without considering result size
- ❌ **Huge OFFSET** — `.skip(1000000)`
- ❌ **Long transaction** — DB transaction + external API + file processing
- ❌ **No indexes** for frequently queried columns
- ❌ **Index everything** — hurts writes/storage
- ❌ **`synchronize: true`** in production
- ❌ **String-interpolated SQL** — `WHERE id = '${id}'`

---

## 32. The complete optimization framework

```text
                API SLOW
                   │
                   ▼
            Measure latency
                   │
                   ▼
          How much is DB time?
             /           \
           YES            NO
            │              │
            ▼              ▼
       Inspect SQL      App/network
            │
            ▼
       N+1 queries?
            │
            ▼
       EXPLAIN ANALYZE
            │
      ┌─────┼─────┐
      ▼     ▼     ▼
   Index   JOIN  Rows
      │     │     │
      └─────┼─────┘
            ▼
       Optimize
            │
            ▼
        Re-measure
```

---

## 33. Tech Lead answer: "How do you optimize TypeORM?"

> I first measure where the latency is coming from rather than assuming TypeORM is the bottleneck. I inspect generated SQL and look for N+1 queries, excessive relation loading, unnecessary columns, and inefficient pagination. For database performance I'd use EXPLAIN ANALYZE and verify indexes, join strategies, row estimates, and lock contention. I'd also check connection-pool utilization and transaction duration. For high-volume queries I'd consider keyset pagination, projections, caching, or read replicas where appropriate. If TypeORM produces inefficient SQL for a critical query, I'd use QueryBuilder or parameterized raw SQL.

---

## The complete TypeORM architecture

```text
                   NestJS
                     │
              ┌──────▼──────┐
              │ Controller  │
              └──────┬──────┘
                     │
              ┌──────▼──────┐
              │   Service   │
              └──────┬──────┘
                     │
              ┌──────▼──────┐
              │ Repository  │
              │ / Query     │
              │   Builder   │
              └──────┬──────┘
                     │
              ┌──────▼──────┐
              │   TypeORM   │
              └──────┬──────┘
                     │
              ┌──────▼──────┐
              │ PostgreSQL  │
              └─────────────┘
```

Surrounded by: indexes, transactions, locks, migrations, caching, connection pool, monitoring, EXPLAIN ANALYZE.

---

## Chapter 16 interview questions

### Q1
An endpoint returns 10,000 users and their orders, taking 5 seconds. What would you investigate first?

### Q2
Why can this be slow?

```ts
repository.find({ relations: { orders: true } });
```

### Q3
Difference between N+1 queries and one huge JOIN — can both cause performance problems?

### Q4
Why can increasing PostgreSQL connection pool size actually make performance worse?

### Q5
When would you use `repository.find()` vs QueryBuilder vs raw SQL?

### Q6 — Tech Lead
`10 API instances`, `100 DB connections per instance`. PostgreSQL becomes unstable. What would you investigate?

Hint: `10 × 100 = ?` Also consider query duration, connection utilization, CPU, memory, locks, pool wait time.

---

## Next → Chapter 17 — Advanced PostgreSQL for Backend Engineers

CTEs → Window Functions → JSONB → ARRAY → full-text search → PostgreSQL-specific indexes → UPSERT → locking patterns → materialized views → partitioning → database functions → triggers → advanced pagination.
