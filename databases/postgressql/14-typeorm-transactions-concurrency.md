# Chapter 14 — TypeORM Transactions & Concurrency

One of the most important chapters for a Tech Lead interview — database knowledge becomes real backend-system design here. Connects TypeORM to the PostgreSQL transaction concepts from [Chapter 6](06-transactions-acid-concurrency.md).

---

## 1. What is a transaction?

A group of database operations that must behave as one logical unit. Transfer ₹1,000: Account A `-₹1,000`, Account B `+₹1,000`. Both must succeed.

```text
BEGIN
  deduct A
  credit B
COMMIT
```

If anything fails: `ROLLBACK`.

---

## 2. MongoDB comparison

```ts
// MongoDB
const session = await mongoose.startSession();
await session.withTransaction(async () => {
  // operations
});

// TypeORM
await dataSource.transaction(async manager => {
  // operations
});
```

Almost identical mental model.

---

## 3. Basic TypeORM transaction

`Account A = ₹10,000`, `Account B = ₹5,000`. Transfer ₹2,000:

```ts
await dataSource.transaction(async manager => {
  await manager.update(
    Account,
    { id: accountAId },
    { balance: () => 'balance - 2000' },
  );

  await manager.update(
    Account,
    { id: accountBId },
    { balance: () => 'balance + 2000' },
  );
});
```

Success → `COMMIT`. Exception → `ROLLBACK`.

---

## 4. The most important TypeORM transaction rule

Inside `dataSource.transaction(async manager => {`, use `manager` for database operations. Don't casually use a globally injected repository (`this.userRepository.save(...)`) inside the transaction — use `manager.save(...)`.

The `manager` represents the transaction's database context:

```text
Global repository    → normal DB connection
Transaction manager  → transaction connection/context
```

(See [Chapter 6 §23](06-transactions-acid-concurrency.md).)

---

## 5. Real payment example

`Payment`, `Wallet`, `Order`. User pays ₹1,000: create payment, mark order paid, update wallet/accounting state — atomic.

```ts
await dataSource.transaction(async manager => {
  const payment = manager.create(Payment, {
    orderId,
    amount: 1000,
    status: 'SUCCESS',
  });

  await manager.save(payment);

  await manager.update(Order, { id: orderId }, { status: 'PAID' });
});
```

If the order update fails, the payment INSERT is rolled back too.

---

## 6. Don't keep transactions open during external APIs

A very important architecture rule. Don't do:

```ts
await dataSource.transaction(async manager => {
  await manager.save(payment);
  await stripe.charge(...); // ❌
  await manager.update(order, ...);
});
```

External calls can take 500ms–10s or timeout/retry, while the DB transaction stays open — causing locks, connection pool exhaustion, reduced throughput, deadlocks, and long-running transactions.

---

## 7. Payment architecture

```text
Client
  ↓
Create payment intent
  ↓
Payment provider
  ↓
Webhook
  ↓
Database transaction
  ├── mark payment SUCCESS
  ├── mark order PAID
  └── create ledger record
  ↓
COMMIT
```

The external payment operation happens outside the DB transaction; webhook processing uses a short transaction.

---

## 8. Concurrency problem

`stock = 1`. Two users purchase simultaneously; both read `stock = 1`, both think they can purchase, both write `stock = 0`. Two products sold when only one existed.

---

## 9. Bad approach

```ts
const product = await productRepository.findOneBy({ id: productId });

if (product.stock > 0) {
  product.stock -= 1;
  await productRepository.save(product);
}
```

Two requests can read the same stock before either updates it.

---

## 10. Better: atomic update

```ts
const result = await productRepository
  .createQueryBuilder()
  .update(Product)
  .set({ stock: () => 'stock - 1' })
  .where('id = :id', { id: productId })
  .andWhere('stock > 0')
  .execute();
```

```ts
if (result.affected === 0) {
  throw new Error('Out of stock');
}
```

```sql
UPDATE product
SET stock = stock - 1
WHERE id = ?
AND stock > 0;
```

The condition and update happen atomically.

---

## 11. Why this is better

`stock = 1`, two requests arrive.

Request A: `UPDATE ... WHERE stock > 0` → succeeds, `stock = 0, affected = 1`.

Request B: same UPDATE → fails, `stock = 0, affected = 0`.

Result: A purchases, B gets "out of stock." No race condition.

---

## 12. Pessimistic locking

When you need to read a row and make decisions based on its current state — e.g. Wallet balance:

```ts
const wallet = await manager
  .getRepository(Wallet)
  .createQueryBuilder('wallet')
  .setLock('pessimistic_write')
  .where('wallet.id = :id', { id })
  .getOne();
```

```sql
SELECT * FROM wallets WHERE id = ... FOR UPDATE;
```

The row is locked until the transaction finishes.

---

## 13. What happens with concurrent requests?

Transaction A: `SELECT ... FOR UPDATE` → wallet locked. Transaction B tries the same → waits.

```text
A → update balance → COMMIT → B continues
```

Prevents both transactions from simultaneously modifying the same locked row based on stale data.

---

## 14. When to use pessimistic locking

Good candidates: wallet balance, inventory allocation, booking seats, limited resources, counters where read-modify-write must be serialized.

Don't lock everything — locks reduce concurrency. Use only when the business invariant actually requires serialized access.

---

## 15. Optimistic locking

```ts
@VersionColumn()
version: number;
```

`balance = 10000, version = 5`. Request A and Request B both read `version = 5`. A updates version 5→6. B tries to update based on `version = 5`, but the current version is 6 — conflict detected.

---

## 16. Optimistic locking conceptually

```sql
UPDATE accounts
SET balance = 8000, version = 6
WHERE id = '...'
AND version = 5;
```

`affected rows = 1` → success. `affected rows = 0` → someone modified the record first → retry or return conflict.

---

## 17. Pessimistic vs Optimistic

| | Pessimistic | Optimistic |
|---|---|---|
| Strategy | Lock first | Detect conflict |
| DB lock | Yes | Usually no |
| Concurrent access | Serialized | Concurrent |
| Conflict handling | Wait/block | Retry/fail |
| Good for | High-contention critical rows | Lower contention |
| Example | Wallet | Profile editing |

The right choice depends on the workload.

---

## 18. Isolation levels

PostgreSQL: `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`. TypeORM lets you specify one:

```ts
await dataSource.transaction('SERIALIZABLE', async manager => {
  // operations
});
```

**READ COMMITTED** — PostgreSQL's default; a query generally sees data committed before that query begins; another transaction can commit changes between your queries.

---

## 19. REPEATABLE READ

Reads within a transaction use a consistent snapshot. Useful for a stable view of data, but concurrent updates can still produce serialization/conflict behavior your application needs to handle.

---

## 20. SERIALIZABLE

Strongest isolation — concurrent transactions behave as if executed serially. Doesn't mean everything magically works: a transaction can fail with a serialization error, and the application should retry appropriate failures.

```text
Transaction → serialization conflict → retry
```

---

## 21. Deadlocks

Transaction A locks User 1, then wants User 2. Transaction B locks User 2, then wants User 1.

```text
A → User 1 → waiting for User 2
B → User 2 → waiting for User 1
```

Deadlock — PostgreSQL detects this and aborts one transaction.

---

## 22. How to reduce deadlocks

**Acquire locks in a consistent order** — e.g. always lock accounts by ascending ID, never inconsistently (100→200 here, 200→100 there). Also: keep transactions short, avoid unnecessary locks, avoid external API calls inside transactions, ensure useful indexes exist.

---

## 23. QueryRunner

For most application transactions, `dataSource.transaction(async manager => {...})` is enough. For more explicit control:

```ts
const queryRunner = dataSource.createQueryRunner();

await queryRunner.connect();
await queryRunner.startTransaction();

try {
  await queryRunner.manager.save(...);
  await queryRunner.manager.update(...);

  await queryRunner.commitTransaction();
} catch (error) {
  await queryRunner.rollbackTransaction();
  throw error;
} finally {
  await queryRunner.release();
}
```

```text
connect → BEGIN → queries → COMMIT / ROLLBACK → release
```

Use when manual control over the transaction lifecycle is needed.

---

## 24. Idempotency + transactions

`POST /payments` with header `Idempotency-Key: abc123`.

```sql
CREATE TABLE payments (
    id UUID PRIMARY KEY,
    idempotency_key TEXT UNIQUE NOT NULL,
    order_id UUID NOT NULL,
    amount NUMERIC(12,2) NOT NULL,
    status TEXT NOT NULL
);
```

`abc123` can only exist once — enforced by the database.

---

## 25. Why SELECT-then-INSERT is dangerous

```ts
const existing = await paymentRepository.findOne({ where: { idempotencyKey } });

if (!existing) {
  await paymentRepository.save(payment);
}
```

Two requests can both SELECT → nothing found → both INSERT. Race condition.

---

## 26. Database constraint is the final protection

`UNIQUE(idempotency_key)` plus:

```ts
await paymentRepository.upsert(
  { idempotencyKey, orderId, amount, status: 'PENDING' },
  ['idempotencyKey'],
);
```

or:

```sql
INSERT ...
ON CONFLICT (idempotency_key)
DO NOTHING;
```

The database itself enforces the invariant (see [Chapter 9 §18-20](09-window-functions-upsert-interview-queries.md)).

---

## 27. Complete payment architecture

```text
Client
  │ idempotency-key
  ▼
API
  │
  ├── Payment Provider
  │       │
  │       ▼
  │    Webhook
  │       │
  ▼       ▼
Database Transaction
  │
  ├── payment
  ├── order
  └── ledger
  │
  ▼
COMMIT
```

Inside the transaction: check/claim idempotency, lock relevant order if necessary, update payment, update order, create ledger entry, commit. External network calls stay outside.

---

## 28. The Tech Lead interview question

> "How do you prevent a user from being charged twice?"

Not just "I use a transaction." Better:

> I'd use an idempotency key generated by the client or payment orchestration layer and enforce a unique constraint on it at the database level. I'd also enforce uniqueness on the payment provider's transaction/reference ID. Payment webhook processing would be idempotent, and related database state changes would happen in a short transaction. For concurrent state changes such as inventory or wallet balances, I'd use atomic conditional updates or appropriate row locking.

---

## 29. The complete concurrency toolbox

```text
Atomic UPDATE     → Best when operation can be expressed directly
Pessimistic Lock  → SELECT ... FOR UPDATE — read → decision → update must be serialized
Optimistic Lock   → version column — conflicts relatively uncommon
Unique Constraint → Database invariant — uniqueness/idempotency
Transaction       → Multiple DB operations as one unit
Serializable      → Strong concurrency guarantees, potential retries
```

---

## 30. Tech Lead decision tree

```text
Can I make it one atomic SQL statement?
  YES → Use atomic UPDATE/INSERT

Do I need to read the row and then decide based on its current state?
  YES → Consider row locking

Are conflicts relatively uncommon, and can I detect stale updates?
  YES → Optimistic locking

Is uniqueness the invariant, and can the database enforce it?
  YES → UNIQUE constraint

Must multiple changes succeed together?
  → Transaction
```

---

## Chapter 14 summary

```text
TypeORM transaction → dataSource.transaction() → transaction manager → COMMIT / ROLLBACK
```

```text
Concurrency
   ├── Atomic updates
   ├── Pessimistic locking
   ├── Optimistic locking
   ├── Isolation levels
   ├── Deadlocks
   └── Idempotency
```

> Transactions guarantee atomicity; they don't automatically solve every concurrency problem.

---

## Interview challenges

### Q1
Two users try to buy the last product simultaneously. Would you use:

```text
A. find → check stock → save
B. atomic UPDATE WHERE stock > 0
C. SELECT FOR UPDATE
D. Both B and C are possible depending on the workflow
```

Explain why.

### Q2
Why is this dangerous?

```ts
await repository.findOne(...);
await stripe.charge(...);
await repository.save(...);
```

### Q3
Difference between optimistic locking and pessimistic locking?

### Q4
How would you prevent duplicate payment processing if the same webhook arrives five times?

### Q5 — Tech Lead
Design the database transaction for:

```text
Place Order
 ├── create order
 ├── reserve inventory
 ├── create order items
 └── create payment record
```

What would you keep inside the transaction, and what would you keep outside?

---

## Next → Chapter 15 — TypeORM Migrations & Production Database Changes

Migrations vs `synchronize`, generating migrations, writing migrations manually, zero-downtime schema changes, adding columns safely, indexes on large tables, rollback strategy, and how a Tech Lead handles production DB deployments.
