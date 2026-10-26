# Chapter 20 — PostgreSQL Advanced Concurrency & Locking

Extremely important for Senior Backend / Tech Lead interviews. Core problem: what happens when 100 requests try to modify the same data at the same time? Building the mental model from MongoDB → PostgreSQL.

---

## 1. The Concurrency Problem

`Product stock = 1`, two users buy simultaneously:

```text
Request A          Request B
   ↓                  ↓
Read stock = 1     Read stock = 1
   ↓                  ↓
stock > 0          stock > 0
   ↓                  ↓
buy product        buy product
```

Both think the product is available. Without proper concurrency control: `Orders = 2, Stock = -1`. A **race condition**.

---

## 2. PostgreSQL Uses MVCC

**MVCC** — Multi-Version Concurrency Control. Instead of overwriting a row, PostgreSQL maintains different row versions internally:

```text
Old version → New version
```

Allows readers and writers to work concurrently in many situations. A transaction updating `users.name` doesn't necessarily block a concurrent `SELECT` on the same row.

---

## 3. MVCC Mental Model

`User { id: 1, name: "John" }`. Transaction A updates to `"Pulkit"`. Conceptually PostgreSQL keeps `Version 1: John` and `Version 2: Pulkit`. Different transactions see different versions depending on their transaction snapshot/isolation level — managed by PostgreSQL, not manually.

---

## 4. MVCC Doesn't Eliminate Race Conditions

MVCC does **not** mean concurrency can be ignored.

```sql
SELECT stock FROM products WHERE id = 1;
```

Application gets `stock = 1`, then:

```ts
if (stock > 0) { /* ... */ }
```

Another request can modify stock before the next query. `SELECT → Application logic → UPDATE` can still race.

---

## 5. The Safe Atomic Update

Instead of `SELECT stock` then `UPDATE products SET stock = stock - 1 WHERE id = 1;`, do:

```sql
UPDATE products
SET stock = stock - 1
WHERE id = 1
  AND stock >= 1;
```

```ts
if (result.affected === 0) {
  throw new Error('Out of stock');
}
```

The database evaluates `stock >= 1` and `stock = stock - 1` as one operation (see [Chapter 14 §10](14-typeorm-transactions-concurrency.md)).

---

## 6. Row-Level Locks

When you need `Read row → several calculations → Modify row`, an atomic UPDATE isn't enough. Use a row lock:

```sql
SELECT *
FROM products
WHERE id = 1
FOR UPDATE;
```

Locks this row for the current transaction. Another transaction locking the same row may have to wait.

---

## 7. Example: Bank Transfer

`Account A = ₹10,000`, `Account B = ₹5,000`, transfer ₹2,000:

```sql
BEGIN;

SELECT * FROM accounts WHERE id = 'A' FOR UPDATE;
SELECT * FROM accounts WHERE id = 'B' FOR UPDATE;

UPDATE accounts SET balance = balance - 2000 WHERE id = 'A';
UPDATE accounts SET balance = balance + 2000 WHERE id = 'B';

COMMIT;
```

Another transaction can't simultaneously modify those locked rows incompatibly.

---

## 8. The Deadlock Problem

Transaction A: lock Account A, then lock Account B. Transaction B: lock Account B, then lock Account A.

```text
A waits for B
B waits for A
```

Deadlock — PostgreSQL detects this and aborts one transaction.

---

## 9. Preventing Deadlocks

Always acquire locks in a consistent order. Instead of "transfer A→B: lock A, lock B" and "transfer B→A: lock B, lock A", sort by ID (smaller first, larger second) so both transactions lock in the same order, avoiding circular wait.

**Tech Lead rule**: establish a consistent lock acquisition order across the application (see [Chapter 14 §22](14-typeorm-transactions-concurrency.md)).

---

## 10. FOR UPDATE

The most commonly used lock:

```sql
SELECT *
FROM orders
WHERE id = $1
FOR UPDATE;
```

Typical use cases: payment processing, inventory, order state transition, job processing, wallet balance, seat booking.

```sql
UPDATE orders SET status = 'CONFIRMED' WHERE id = 'order_1';
```

after locking — a second transaction trying to lock the same order waits.

---

## 11. FOR SHARE

```sql
SELECT *
FROM products
WHERE id = 1
FOR SHARE;
```

A weaker locking scenario — protects against certain concurrent modifications while allowing compatible locks. `FOR UPDATE` is encountered far more frequently in backend applications.

---

## 12. FOR NO KEY UPDATE

```sql
FOR NO KEY UPDATE
```

Weaker than `FOR UPDATE` — relevant when modifying a row without changing key-related values in a way that requires the stronger lock. `FOR UPDATE` is the first lock to understand for most application code.

---

## 13. SKIP LOCKED

Particularly useful for job queues. `jobs(job1..job5)`, Workers A/B/C. Worker A locks `job1, job2`; Worker B shouldn't wait:

```sql
SELECT *
FROM jobs
WHERE status = 'PENDING'
ORDER BY created_at
FOR UPDATE SKIP LOCKED
LIMIT 2;
```

Worker B skips locked rows and gets `job3, job4`; Worker C gets `job5`. Enables multiple workers to process jobs concurrently.

---

## 14. PostgreSQL as a Job Queue

```text
jobs(id, type, payload, status, attempts, created_at)
```

Each worker:

```sql
SELECT *
FROM jobs
WHERE status = 'PENDING'
ORDER BY created_at
FOR UPDATE SKIP LOCKED
LIMIT 10;
```

Works surprisingly well for moderate workloads. For very large-scale messaging, use a dedicated broker (Kafka, RabbitMQ, SQS) rather than turning PostgreSQL into the entire messaging platform.

---

## 15. Important SKIP LOCKED Caveat

Don't do:

```text
BEGIN
  SELECT ... FOR UPDATE SKIP LOCKED
  call external API
  wait 10 seconds
  COMMIT
```

Holds locks during the network call. Instead:

```text
Claim jobs → COMMIT → Process externally → Update result
```

A common implementation is a **lease**: `status = PROCESSING, locked_until = timestamp, worker_id = ...` — another worker can reclaim jobs whose lease expired.

---

## 16. Optimistic Locking

Pessimistic: "lock it before changing it." Optimistic: "assume conflicts are rare, but detect them when updating."

`Product { id: 1, stock: 10, version: 7 }`. Read `version = 7`, then:

```sql
UPDATE products
SET stock = 9, version = 8
WHERE id = 1
  AND version = 7;
```

`affected rows = 1` → success. `affected rows = 0` → someone else changed the row (see [Chapter 14 §15-16](14-typeorm-transactions-concurrency.md)).

---

## 17. TypeORM Optimistic Locking

```ts
@Entity('products')
export class Product {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ type: 'int' })
  stock: number;

  @VersionColumn()
  version: number;
}
```

```text
version 5 → update → version 6
```

Detects conflicts if another request already changed the row.

---

## 18. Pessimistic vs Optimistic

| | Pessimistic | Optimistic |
|---|---|---|
| Strategy | Lock first | Detect conflict |
| DB lock | Yes | Usually no long-held lock |
| Best when | High conflict | Low conflict |
| Waiting | Possible | Usually retry/fail |
| Example | Wallet balance, seat booking, order processing | Editing profile, updating article, admin editing |

Many requests fighting over the same row → pessimistic. Conflicts uncommon → optimistic.

---

## 19. Serializable Isolation

`READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`. `SERIALIZABLE` provides the strongest isolation — concurrent transactions behave as if executed one after another.

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

PostgreSQL may detect a serialization conflict and abort a transaction — the application must retry:

```text
Transaction → Serialization failure → Rollback → Retry transaction
```

---

## 20. Why Not Use SERIALIZABLE Everywhere?

Stronger isolation means more conflicts, more retries, more overhead. Choose isolation based on the business requirement.

Not "serializable is always better" — instead: "use the weakest isolation level that correctly satisfies the business invariant."

---

## 21. Real Example — Seat Booking

`Flight seat A1, status = AVAILABLE`. Users A and B both try to book it.

**Option 1: Pessimistic**

```sql
BEGIN;

SELECT * FROM seats WHERE id = 'A1' FOR UPDATE;
-- verify AVAILABLE

UPDATE seats SET status = 'BOOKED' WHERE id = 'A1';

COMMIT;
```

User B waits, then sees `BOOKED` and fails.

---

## 22. Option 2: Atomic UPDATE

Often simpler:

```sql
UPDATE seats
SET status = 'BOOKED'
WHERE id = 'A1'
  AND status = 'AVAILABLE';
```

```ts
if (result.affected === 0) {
  throw new Error('Seat already booked');
}
```

The invariant is enforced directly by the database operation.

---

## 23. Option 3: Unique Constraint

```text
flight_id
seat_number
UNIQUE(flight_id, seat_number)
```

Two requests cannot create two records for the same seat.

> Use database constraints whenever possible to enforce invariants — don't rely exclusively on application code.

---

## 24. Concurrency Hierarchy

1. **Can I enforce it with a constraint?** e.g. `UNIQUE(user_id, product_id)`
2. **Can I use an atomic UPDATE?** `UPDATE ... WHERE condition;`
3. **Do I need a row lock?** `FOR UPDATE`
4. **Do I need optimistic locking?** `version`
5. **Do I need stronger transaction isolation?** `SERIALIZABLE`

A useful design progression.

---

## 25. TypeORM + Pessimistic Lock

```ts
await dataSource.transaction(async manager => {
  const product = await manager
    .getRepository(Product)
    .createQueryBuilder('product')
    .setLock('pessimistic_write')
    .where('product.id = :id', { id: productId })
    .getOne();

  if (!product) {
    throw new NotFoundException();
  }

  if (product.stock < quantity) {
    throw new BadRequestException('Out of stock');
  }

  product.stock -= quantity;
  await manager.save(product);
});
```

`manager` is transaction-scoped — don't accidentally use the global repository inside the transaction.

---

## 26. High-Concurrency Inventory — three approaches

**❌ Bad**:

```ts
const product = await repo.findOneBy({ id });

if (product.stock >= quantity) {
  product.stock -= quantity;
  await repo.save(product);
}
```

Race condition.

**✅ Better** — atomic update:

```sql
UPDATE products
SET stock = stock - :quantity
WHERE id = :id
AND stock >= :quantity;
```

```ts
if (affected !== 1) {
  throw new OutOfStockError();
}
```

**✅ Another valid approach**:

```sql
SELECT * FROM products WHERE id = :id FOR UPDATE;
```

Then check and update within the transaction.

---

## 27. A Subtle Senior-Level Question

`stock = 100`, `1,000 simultaneous buyers`. Should every request go through `FOR UPDATE`? Not necessarily — creates a lock bottleneck: `1000 requests → one product row → serialized access`.

An atomic update:

```sql
UPDATE products
SET stock = stock - 1
WHERE id = ?
AND stock > 0;
```

can often be simpler and more efficient.

> Don't use locks merely because concurrency exists. Use the simplest mechanism that correctly enforces the invariant.

---

## 28. PostgreSQL Concurrency Toolkit

```text
MVCC                    → Concurrent reads/writes
Atomic UPDATE           → Simple race-condition prevention
FOR UPDATE              → Pessimistic row locking
SKIP LOCKED             → Parallel workers / job queues
Version column          → Optimistic locking
UNIQUE / CHECK / FK     → Database-level invariants
SERIALIZABLE            → Strongest isolation
Consistent lock ordering → Deadlock prevention
```

---

## 29. Tech Lead Interview Question

> "Two users purchase the last item simultaneously. How do you guarantee only one succeeds?"

> I wouldn't perform a separate SELECT followed by an UPDATE because that creates a race condition. I'd prefer an atomic conditional UPDATE such as `UPDATE products SET stock = stock - 1 WHERE id = ? AND stock > 0`, then check the affected row count. If the affected count is zero, the product is unavailable. The inventory update and order creation would be inside the appropriate local transaction. I'd also enforce `stock >= 0` at the database level as a defensive invariant.

---

## 30. Another Senior-Level Question

> "When would you use `FOR UPDATE SKIP LOCKED`?"

> It's useful when multiple workers need to claim pending jobs concurrently without waiting on rows already claimed by another worker. Each worker can lock a batch of available rows using `FOR UPDATE SKIP LOCKED`. For external processing, I would avoid holding those locks during network calls and instead use a claim/lease mechanism.

---

## 31. Final mental model

```text
Concurrent requests
    ↓
What invariant am I protecting?
    ↓
Can a DB constraint enforce it?
    ↓
Can an atomic statement enforce it?
    ↓
Do I need a row lock?
    ↓
Could optimistic locking work?
    ↓
Do I need stronger isolation?
    ↓
How do I handle retries/deadlocks?
```

More valuable than memorizing PostgreSQL commands.

---

## Next — Chapter 21: PostgreSQL Advanced Query Optimization & Execution Plans

`EXPLAIN ANALYZE`, Query Planner, Seq Scan vs Index Scan, Bitmap Heap Scan, Nested Loop, Hash Join, Merge Join, composite indexes, covering indexes, partial indexes, index-only scans, statistics, cardinality estimation, CTE optimization, subquery optimization, keyset pagination, 5M/100M+ row tables.

Taking PostgreSQL knowledge from "I can write SQL" to "I can diagnose why production SQL is slow."
