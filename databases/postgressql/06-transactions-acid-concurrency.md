# Chapter 6 — Transactions, ACID & Concurrency

One of the most important chapters for a Tech Lead interview.

If asked "Why PostgreSQL over MongoDB?", a strong answer often includes:

> PostgreSQL provides strong relational integrity and mature transactional/concurrency capabilities, which are particularly useful when multiple related records must change atomically.

Everything mapped to MongoDB transaction concepts already known.

---

## 1. What is a transaction?

A group of database operations treated as **one logical unit of work**.

Classic example — money transfer:

```text
Account A: ₹10,000
Account B: ₹5,000
```

Transfer ₹2,000: `A → -₹2,000`, `B → +₹2,000`. Both operations must succeed — we don't want money to disappear (`A → ₹8,000, B → ₹5,000`) or be created (`A → ₹10,000, B → ₹7,000`).

```text
Debit A + Credit B = ONE TRANSACTION
```

---

## 2. PostgreSQL transaction syntax

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 2000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 2000
WHERE id = 2;

COMMIT;
```

`COMMIT` makes changes permanent if everything succeeds. `ROLLBACK;` undoes the transaction if something fails.

---

## 3. MongoDB comparison

```javascript
const session = await mongoose.startSession();
session.startTransaction();

try {
    await Account.updateOne({ _id: accountA }, { $inc: { balance: -2000 } }, { session });
    await Account.updateOne({ _id: accountB }, { $inc: { balance: 2000 } }, { session });
    await session.commitTransaction();
} catch (error) {
    await session.abortTransaction();
}
```

Conceptually very similar to the PostgreSQL `BEGIN ... COMMIT` block above.

---

## 4. ACID

```text
A → Atomicity
C → Consistency
I → Isolation
D → Durability
```

Classic interview questions.

---

## 5. Atomicity

Either the entire transaction happens, or none of it happens.

```text
Transfer ₹2,000
Debit A → Success
Credit B → FAIL
```

Atomicity says `ROLLBACK` — so `A` and `B` return to their original balances. No partial transaction.

---

## 6. Consistency

A transaction must take the database from one valid state to another valid state while respecting defined constraints/invariants — e.g. `accounts.balance >= 0`, or `orders.user_id` must reference `users.id`.

PostgreSQL constraints enforce this: `PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `NOT NULL`, `CHECK`.

```sql
CREATE TABLE accounts (
    id UUID PRIMARY KEY,
    balance NUMERIC(12,2)
        CHECK (balance >= 0)
);
```

`UPDATE accounts SET balance = -100 WHERE id = '...';` will now fail.

---

## 7. Isolation

Two simultaneous requests updating the same account, without proper concurrency control, can produce incorrect results. Isolation defines how much one transaction can see/interact with another concurrent transaction. PostgreSQL provides multiple isolation levels (§17–20).

---

## 8. Durability

After `COMMIT;`, the database promises committed changes survive failures.

```text
Transaction → COMMIT → Database crash → Database recovery → Committed data remains
```

One reason databases use mechanisms such as **WAL — Write-Ahead Logging**.

---

## 9. Classic interview question

> What happens if the server crashes after COMMIT?

PostgreSQL uses Write-Ahead Logging (WAL) so committed changes can be recovered after a crash. (WAL, replication and recovery covered later.)

---

## 10. Real-world payment transaction

```text
orders
------
id
status
total_amount

payments
--------
id
order_id
amount
status
```

```sql
BEGIN;

INSERT INTO payments (order_id, amount, status)
VALUES ('order-123', 1000, 'SUCCESS');

UPDATE orders
SET status = 'PAID'
WHERE id = 'order-123';

UPDATE inventory
SET stock = stock - 1
WHERE product_id = 'product-123';

COMMIT;
```

If the inventory update fails, `ROLLBACK;` undoes all the prior changes in that transaction too.

---

## 11. Tech Lead trap — keep transactions short

Transactions should generally be kept as short as practical. Don't wrap external calls inside a DB transaction:

```text
BEGIN
  → Database operation
  → Call Stripe
  → Wait 5 seconds
  → Call another API
  → Send email
  → Database operation
COMMIT
```

This keeps database resources/locks held far too long. Instead:

```text
Payment provider → Payment result → Short DB transaction → Update database
```

Coordinating external APIs with database transactions is a common system-design interview topic.

---

## 12. Lost Update Problem

```text
balance = ₹10,000
```

Request A reads ₹10,000, Request B also reads ₹10,000. A calculates `10,000 - 2,000 = 8,000`, B calculates `10,000 - 3,000 = 7,000`. A writes 8,000, then B writes 7,000. Final balance: ₹7,000 — but expected `10,000 - 2,000 - 3,000 = ₹5,000`.

This is a **lost update**.

---

## 13. Simple solution: atomic UPDATE

Instead of `SELECT balance → calculate in Node.js → UPDATE balance`, do the calculation inside the database:

```sql
UPDATE accounts
SET balance = balance - 2000
WHERE id = 'account-1'
  AND balance >= 2000;
```

Then check affected rows in the application:

- `affected rows = 1` → success
- `affected rows = 0` → insufficient balance or account doesn't exist

An excellent production pattern.

---

## 14. Row-level locking

```sql
SELECT *
FROM accounts
WHERE id = 'account-1'
FOR UPDATE;
```

Locks this row for update within the current transaction.

```sql
BEGIN;

SELECT balance
FROM accounts
WHERE id = 'account-1'
FOR UPDATE;

UPDATE accounts
SET balance = balance - 2000
WHERE id = 'account-1';

COMMIT;
```

Another transaction trying to update the same row may have to wait.

---

## 15. `FOR UPDATE` mental model

```text
Transaction A
SELECT ... FOR UPDATE → 🔒 Row locked → UPDATE → COMMIT → 🔓 Unlock

Transaction B
UPDATE same row → waits → Transaction A commits → B continues
```

This is **pessimistic locking**.

---

## 16. Optimistic vs Pessimistic locking

**Pessimistic**: assume someone else might modify this row, so lock it (`SELECT ... FOR UPDATE`).

**Optimistic**: assume conflicts are uncommon; store a version.

```text
id
balance
version
```

Read: `balance = 10000, version = 5`. Then:

```sql
UPDATE accounts
SET balance = 8000, version = 6
WHERE id = 'account-1'
  AND version = 5;
```

If another transaction already bumped `version` to `6`, this update affects `0 rows` — the application detects the conflict.

---

## 17. Isolation levels

PostgreSQL: `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`. It also recognizes `READ UNCOMMITTED`, but treats it effectively as `READ COMMITTED`. The important three for interviews: `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`.

---

## 18. READ COMMITTED

PostgreSQL's default isolation level. A transaction sees data committed before each statement begins.

Transaction A reads `balance = 10000`. Transaction B updates to `8000` and commits. A executes another query — under READ COMMITTED, A can now see the newly committed value. Different statements within the same transaction can see different committed states.

---

## 19. REPEATABLE READ

Provides a consistent snapshot for the whole transaction.

```text
BEGIN → Snapshot created → Query 1 → Query 2 → same transaction snapshot
```

Other committed changes generally aren't visible to subsequent statements in that transaction. PostgreSQL's implementation is stronger than the minimum SQL standard requirement.

---

## 20. SERIALIZABLE

The strongest isolation level — attempts to make concurrent transactions behave as if executed serially. Cost: transactions can fail with serialization errors, so the application must be prepared to retry. An important production concept.

---

## 21. Interview question

> What's the difference between pessimistic and optimistic locking?

> Pessimistic locking assumes conflicts are likely and locks the relevant database row, for example with `SELECT ... FOR UPDATE`. Optimistic locking doesn't hold a database lock during the read; instead, it detects conflicts using a version or timestamp during the update. Pessimistic locking is useful for high-contention critical resources, while optimistic locking is useful when conflicts are relatively uncommon.

---

## 22. TypeORM transactions

```typescript
await dataSource.transaction(async manager => {
  const payment = manager.create(Payment, {
    orderId,
    amount,
    status: 'SUCCESS',
  });

  await manager.save(payment);

  await manager.update(
    Order,
    { id: orderId },
    { status: 'PAID' },
  );
});
```

```text
dataSource.transaction() → BEGIN → INSERT payment → UPDATE order → COMMIT
```

An exception triggers `ROLLBACK`.

---

## 23. Very important TypeORM rule

Inside a transaction, use the transaction's `manager`.

Correct:

```typescript
await dataSource.transaction(async manager => {
  await manager.save(payment);
  await manager.update(Order, { id: orderId }, { status: 'PAID' });
});
```

Don't use a globally injected repository inside the transaction:

```typescript
await dataSource.transaction(async manager => {
  await paymentRepository.save(payment); // ⚠️
});
```

That repository may use a different connection/context and may not participate in the transaction as intended. Interviewers love this question.

---

## 24. TypeORM pessimistic locking

```typescript
const account = await manager
  .getRepository(Account)
  .createQueryBuilder('account')
  .setLock('pessimistic_write')
  .where('account.id = :id', { id: accountId })
  .getOne();
```

Conceptually:

```sql
SELECT *
FROM accounts
WHERE id = $1
FOR UPDATE;
```

---

## 25. Tech Lead scenario

`Product stock = 1`. Two users click Buy simultaneously. Neither `stock = -1` nor two successful orders for one item is allowed.

```sql
UPDATE products
SET stock = stock - 1
WHERE id = 'product-1'
  AND stock > 0;
```

Then check `affectedRows === 1`. Better than `SELECT stock → Node.js checks stock → UPDATE stock`, which has a race condition.

---

## Interview checkpoint

### Q1
Explain ACID in your own words.

### Q2
Why is this potentially unsafe under concurrent load?

```typescript
const account = await accountRepo.findOneBy({ id });

if (account.balance >= 1000) {
  account.balance -= 1000;
  await accountRepo.save(account);
}
```

### Q3
What does this do?

```sql
SELECT *
FROM accounts
WHERE id = '123'
FOR UPDATE;
```

### Q4
Difference between optimistic locking and pessimistic locking?

### Q5
In a TypeORM transaction, why use `manager.save(...)` instead of a globally injected repository?

---

## Next chapter

**Chapter 7 — Indexes & Query Performance**: B-Tree, composite indexes, unique indexes, partial indexes, GIN, JSONB indexes, index order, covering indexes, `EXPLAIN`, `EXPLAIN ANALYZE`, Sequential Scan vs Index Scan — and how to answer "this PostgreSQL query is taking 5 seconds in production, how would you debug and optimize it?"
