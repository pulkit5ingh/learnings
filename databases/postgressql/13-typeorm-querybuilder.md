# Chapter 13 — TypeORM QueryBuilder

Where TypeORM becomes much more powerful. With SQL already understood, think of QueryBuilder as: writing SQL using TypeScript while letting TypeORM handle entity mapping and parameters.

For simple CRUD: `repository.find()`, `repository.findOne()`, `repository.update()`.

For complex queries: `repository.createQueryBuilder()`.

---

## 1. Why QueryBuilder?

> Find active users who have placed at least one order above ₹1,000, return their latest order, sorted by order date.

Difficult to express cleanly with `repository.find()`. QueryBuilder is designed for this.

---

## 2. Basic QueryBuilder

```ts
const users = await userRepository
  .createQueryBuilder('user')
  .where('user.is_active = :isActive', { isActive: true })
  .getMany();
```

```sql
SELECT * FROM users user WHERE user.is_active = true;
```

`'user'` is an alias — like `FROM users AS user`.

---

## 3. Parameters — very important

Don't do:

```ts
.where(`user.email = '${email}'`)
```

Dynamically constructed SQL can create SQL injection vulnerabilities. Instead:

```ts
.where('user.email = :email', { email })
```

TypeORM safely parameterizes the value.

```text
User input → Query parameter → Database
```

not:

```text
User input → String concatenation → SQL
```

**Interview answer** — "How do you prevent SQL injection with TypeORM QueryBuilder?"

> I use parameterized queries rather than interpolating user input into SQL strings.

---

## 4. `andWhere()`

```ts
const users = await userRepository
  .createQueryBuilder('user')
  .where('user.is_active = :active', { active: true })
  .andWhere('user.age >= :age', { age: 25 })
  .getMany();
```

```sql
SELECT * FROM users WHERE is_active = true AND age >= 25;
```

---

## 5. `orWhere()`

```ts
const users = await userRepository
  .createQueryBuilder('user')
  .where('user.is_active = :active', { active: true })
  .orWhere('user.is_admin = :admin', { admin: true })
  .getMany();
```

```sql
WHERE is_active = true OR is_admin = true
```

Be careful with logical precedence.

---

## 6. Grouping conditions

`active users AND (age >= 25 OR is_admin = true)` — use `Brackets`:

```ts
import { Brackets } from 'typeorm';

const users = await userRepository
  .createQueryBuilder('user')
  .where('user.is_active = :active', { active: true })
  .andWhere(
    new Brackets(qb => {
      qb.where('user.age >= :age', { age: 25 })
        .orWhere('user.is_admin = :admin', { admin: true });
    }),
  )
  .getMany();
```

```sql
WHERE is_active = true
AND (age >= 25 OR is_admin = true)
```

Very useful for dynamic search APIs.

---

## 7. JOIN

```ts
const users = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.orders', 'order')
  .getMany();
```

```sql
SELECT ...
FROM users user
LEFT JOIN orders order
  ON order.user_id = user.id;
```

---

## 8. Filtering a JOIN

```ts
const users = await userRepository
  .createQueryBuilder('user')
  .innerJoinAndSelect('user.orders', 'order')
  .where('order.status = :status', { status: 'PAID' })
  .getMany();
```

```sql
SELECT ...
FROM users user
INNER JOIN orders order
  ON order.user_id = user.id
WHERE order.status = 'PAID';
```

---

## 9. `leftJoin` vs `innerJoin`

**INNER JOIN** (`.innerJoin(...)`) — only users having matching orders.

**LEFT JOIN** (`.leftJoin(...)`) — all users, even with no orders.

```text
INNER JOIN = matching rows
LEFT JOIN  = everything from left + matching right
```

---

## 10. SELECT specific columns

```ts
const users = await userRepository
  .createQueryBuilder('user')
  .select(['user.id', 'user.name', 'user.email'])
  .getMany();
```

```sql
SELECT user.id, user.name, user.email FROM users user;
```

If a table has `password_hash`, `profile_data`, `metadata`, `large_json_document`, etc., and the API only needs `id`/`name`, don't retrieve everything unnecessarily.

---

## 11. Aggregation

Total order amount per user:

```ts
const result = await orderRepository
  .createQueryBuilder('order')
  .select('order.user_id', 'userId')
  .addSelect('SUM(order.amount)', 'totalAmount')
  .groupBy('order.user_id')
  .getRawMany();
```

```sql
SELECT user_id, SUM(amount) AS totalAmount
FROM orders
GROUP BY user_id;
```

`.getRawMany()` instead of `.getMany()` because this returns an aggregate result, not normal `Order` entities.

---

## 12. `getMany()` vs `getRawMany()`

- **`getMany()`** — returns entities (`User[]` / `Order[]`)
- **`getOne()`** — returns `User | null`
- **`getRawMany()`** — returns raw database results, e.g. `[{ userId: '123', totalAmount: '5000' }]`
- **`getRawOne()`** — one raw result

---

## 13. HAVING

> Find users whose total order value is greater than ₹10,000.

```ts
const result = await orderRepository
  .createQueryBuilder('order')
  .select('order.user_id', 'userId')
  .addSelect('SUM(order.amount)', 'totalAmount')
  .groupBy('order.user_id')
  .having('SUM(order.amount) > :amount', { amount: 10000 })
  .getRawMany();
```

```sql
SELECT user_id, SUM(amount) AS totalAmount
FROM orders
GROUP BY user_id
HAVING SUM(amount) > 10000;
```

```text
WHERE    → filters rows
GROUP BY → creates groups
HAVING   → filters groups
```

---

## 14. ORDER BY

```ts
const orders = await orderRepository
  .createQueryBuilder('order')
  .orderBy('order.created_at', 'DESC')
  .getMany();
```

Multiple sorting:

```ts
.orderBy('order.status', 'ASC')
.addOrderBy('order.created_at', 'DESC')
```

```sql
ORDER BY status ASC, created_at DESC;
```

---

## 15. Pagination

```ts
const orders = await orderRepository
  .createQueryBuilder('order')
  .orderBy('order.created_at', 'DESC')
  .skip(20)
  .take(20)
  .getMany();
```

Conceptually `OFFSET 20 LIMIT 20`. Large OFFSET values can become expensive (e.g. `OFFSET 1,000,000`) — for large datasets, cursor/keyset pagination is usually preferable (see [Chapter 3 §21](03-tables-datatypes-crud.md)).

---

## 16. Keyset pagination

Instead of `OFFSET 100000 LIMIT 20`, ask for "the next 20 records after this cursor":

```ts
const orders = await orderRepository
  .createQueryBuilder('order')
  .where('order.created_at < :createdAt', { createdAt: cursorCreatedAt })
  .orderBy('order.created_at', 'DESC')
  .take(20)
  .getMany();
```

For a robust cursor, especially when timestamps can tie, include a unique tiebreaker such as `id`.

```text
Page 1 → last record = cursor → Page 2 starts after cursor → Page 3 → ...
```

A common senior-level backend interview topic.

---

## 17. Dynamic filters

```http
GET /orders?status=PAID&minAmount=1000
```

Build the query dynamically instead of writing every combination:

```ts
const qb = orderRepository.createQueryBuilder('order');

if (status) {
  qb.andWhere('order.status = :status', { status });
}

if (minAmount) {
  qb.andWhere('order.amount >= :minAmount', { minAmount });
}

const orders = await qb.getMany();
```

A very common real-world QueryBuilder use case.

---

## 18. Dynamic sorting — be careful

```http
?sortBy=createdAt
```

Don't blindly do `.orderBy(`order.${sortBy}`, 'DESC')` — column names can't safely be treated like ordinary parameter values. Whitelist them:

```ts
const allowedSortFields = {
  createdAt: 'order.created_at',
  amount: 'order.amount',
};

const sortColumn = allowedSortFields[sortBy] ?? allowedSortFields.createdAt;

qb.orderBy(sortColumn, 'DESC');
```

A good production/security practice.

---

## 19. Subquery

> Find users whose total spending is greater than ₹10,000.

```sql
SELECT *
FROM users u
WHERE u.id IN (
  SELECT o.user_id
  FROM orders o
  GROUP BY o.user_id
  HAVING SUM(o.amount) > 10000
);
```

TypeORM can construct this with QueryBuilder, but don't force everything into it — if raw SQL is significantly clearer, use it.

---

## 20. Raw SQL is not a failure

A key Tech Lead principle: "we use TypeORM, therefore every query must use TypeORM" is wrong. Choose the right tool:

- Straightforward CRUD → `repository.find()`
- Complex entity queries → QueryBuilder
- Highly specialized PostgreSQL features → raw SQL

The goal is correctness + maintainability + performance, not "maximum ORM usage."

---

## 21. QueryBuilder + relations + aggregation

> Find the top 10 users by successful payment amount.

```ts
const result = await orderRepository
  .createQueryBuilder('order')
  .innerJoin('order.user', 'user')
  .select('user.id', 'userId')
  .addSelect('user.name', 'userName')
  .addSelect('SUM(order.amount)', 'totalAmount')
  .where('order.status = :status', { status: 'PAID' })
  .groupBy('user.id')
  .addGroupBy('user.name')
  .orderBy('SUM(order.amount)', 'DESC')
  .limit(10)
  .getRawMany();
```

```sql
SELECT
  user.id,
  user.name,
  SUM(order.amount) AS total_amount
FROM orders order
JOIN users user
  ON user.id = order.user_id
WHERE order.status = 'PAID'
GROUP BY user.id, user.name
ORDER BY SUM(order.amount) DESC
LIMIT 10;
```

Combines JOIN + WHERE + SUM + GROUP BY + ORDER BY + LIMIT — the exact PostgreSQL concepts already covered in [Chapter 4](04-aggregations-groupby-having-subqueries-ctes.md).

---

## 22. Debugging QueryBuilder

When a query is slow, don't blindly optimize — inspect the SQL first. Enable TypeORM query logging:

```ts
logging: ['query', 'error']
```

Then run `EXPLAIN ANALYZE` on the SQL it produces:

```text
TypeORM → Generated SQL → EXPLAIN ANALYZE → Execution plan → Indexes/joins/scans → Optimization
```

Exactly how a senior backend engineer approaches ORM performance (see [Chapter 8](08-explain-analyze-query-optimization.md)).

---

## 23. TypeORM vs writing SQL — three levels

**Level 1 — Repository**: `repository.find({ where: { status: 'PAID' } });` — straightforward queries.

**Level 2 — QueryBuilder**: `.createQueryBuilder().innerJoin(...).where(...).groupBy(...).having(...)` — complex dynamic queries.

**Level 3 — Raw SQL**: `repository.query('SELECT ...')` — very specific SQL/database functionality, or when the ORM expression becomes unnecessarily complicated.

---

## 24. Tech Lead mental model

Not "I know TypeORM." Instead:

```text
Business requirement
       ↓
Access pattern
       ↓
SQL query
       ↓
Index strategy
       ↓
TypeORM implementation
       ↓
Generated SQL
       ↓
EXPLAIN ANALYZE
       ↓
Measure
```

The progression from developer → senior engineer → tech lead.

---

## Chapter 13 interview checkpoint

### Q1
Difference between `getMany()` and `getRawMany()`?

### Q2
How would you prevent SQL injection in `.createQueryBuilder()`?

### Q3
Write a QueryBuilder query to find all users who have at least one PAID order.

### Q4
Why can `.skip(1000000).take(20)` be problematic? What would you use instead for a large dataset?

### Q5
`GET /orders?status=PAID&minAmount=1000&sortBy=createdAt` — how would you design the QueryBuilder so all filters are optional?

### Q6 — Senior-level

```ts
repository.find({ relations: { orders: true } });
```

looks simple but is slow in production. What would you investigate before changing the code?

```text
Generated SQL → EXPLAIN ANALYZE → Indexes → Row counts → JOIN strategy → Data volume → N+1 → Network / serialization
```

---

## Next: Chapter 14 — TypeORM Transactions & Concurrency

Connecting directly to the PostgreSQL transaction chapter ([Chapter 6](06-transactions-acid-concurrency.md)) with real examples: payment → wallet → inventory → idempotency → row locking → `QueryRunner` → isolation levels → optimistic/pessimistic locking → preventing double payments/orders.
