# Chapter 18 — Build a Production NestJS + PostgreSQL + TypeORM Project

Putting everything together into a realistic e-commerce/order backend: Users, Products, Orders, Order Items, Payments, Inventory — with PostgreSQL, NestJS, TypeORM, transactions, indexes, idempotency, concurrency protection, pagination, migrations, and QueryBuilder.

---

## 1. The architecture

```text
                         NestJS
                           │
          ┌────────────────┼────────────────┐
          │                │                │
       Users            Products          Orders
          │                │                │
          └────────────────┼────────────────┘
                           │
                       Payments
                           │
                       PostgreSQL
```

```text
Client
  │
  ▼
Controller
  │
  ▼
Service
  │
  ├──────────────┐
  ▼              ▼
Repository    Other Services
  │              │
  └──────┬───────┘
         ▼
      TypeORM
         │
         ▼
    PostgreSQL
```

---

## 2. Database model

```text
users
----------------
id
email
name
created_at

products
----------------
id
name
price
stock
created_at

orders
----------------
id
user_id
status
total_amount
created_at

order_items
----------------
id
order_id
product_id
quantity
unit_price

payments
----------------
id
order_id
idempotency_key
provider_payment_id
amount
status
created_at
```

Relationships:

```text
User → 1:N → Order → 1:N → OrderItem → N:1 → Product
Order → 1:N (or 1:1 by business rule) → Payment
```

---

## 3. Why `OrderItem` exists

Not just `Order → products[]`. Product price today (₹1,000) can differ from purchase-time price. Customer buys 2 units at ₹1,000; later the product price changes to ₹1,500 — the historical order must still show 2 × ₹1,000. Hence `order_items.unit_price` is essential (see [Chapter 5 §19](05-relationships-database-design.md)).

---

## 4. User Entity

```ts
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ type: 'text', unique: true })
  email: string;

  @Column({ type: 'text' })
  name: string;

  @CreateDateColumn({ name: 'created_at', type: 'timestamptz' })
  createdAt: Date;

  @OneToMany(() => Order, order => order.user)
  orders: Order[];
}
```

TypeScript uses `createdAt`, PostgreSQL uses `created_at`.

---

## 5. Product Entity

```ts
@Entity('products')
export class Product {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ type: 'text' })
  name: string;

  @Column({ type: 'numeric', precision: 12, scale: 2 })
  price: string;

  @Column({ type: 'integer', default: 0 })
  stock: number;

  @CreateDateColumn({ name: 'created_at', type: 'timestamptz' })
  createdAt: Date;

  @OneToMany(() => OrderItem, orderItem => orderItem.product)
  orderItems: OrderItem[];
}
```

---

## 6. Why is price a `string`?

`price: string` instead of `price: number` — PostgreSQL `NUMERIC(12,2)` is exact decimal arithmetic, while JavaScript's `number` uses floating-point. For financial values, be careful with floating-point arithmetic. Options: `NUMERIC` + a decimal library, or represent monetary amounts as integer minor units (`₹100.50 → 10050 paise`). The right approach depends on the system.

---

## 7. Order Entity

```ts
@Entity('orders')
export class Order {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => User, user => user.orders, { nullable: false })
  @JoinColumn({ name: 'user_id' })
  user: User;

  @Column({ type: 'text' })
  status: string;

  @Column({ name: 'total_amount', type: 'numeric', precision: 12, scale: 2 })
  totalAmount: string;

  @CreateDateColumn({ name: 'created_at', type: 'timestamptz' })
  createdAt: Date;

  @OneToMany(() => OrderItem, item => item.order)
  items: OrderItem[];

  @OneToMany(() => Payment, payment => payment.order)
  payments: Payment[];
}
```

---

## 8. OrderItem

```ts
@Entity('order_items')
export class OrderItem {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => Order, order => order.items, { nullable: false })
  @JoinColumn({ name: 'order_id' })
  order: Order;

  @ManyToOne(() => Product, product => product.orderItems, { nullable: false })
  @JoinColumn({ name: 'product_id' })
  product: Product;

  @Column({ type: 'integer' })
  quantity: number;

  @Column({ name: 'unit_price', type: 'numeric', precision: 12, scale: 2 })
  unitPrice: string;
}
```

---

## 9. Payment Entity

```ts
@Entity('payments')
export class Payment {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => Order, order => order.payments, { nullable: false })
  @JoinColumn({ name: 'order_id' })
  order: Order;

  @Column({ name: 'idempotency_key', type: 'text', unique: true })
  idempotencyKey: string;

  @Column({ name: 'provider_payment_id', type: 'text', unique: true, nullable: true })
  providerPaymentId: string | null;

  @Column({ type: 'numeric', precision: 12, scale: 2 })
  amount: string;

  @Column({ type: 'text' })
  status: string;

  @CreateDateColumn({ name: 'created_at', type: 'timestamptz' })
  createdAt: Date;
}
```

Two important uniqueness guarantees: `idempotency_key UNIQUE` and `provider_payment_id UNIQUE` — protection against duplicate processing.

---

## 10. Database constraints

Don't rely entirely on TypeScript. The database should protect important invariants:

```sql
CHECK (stock >= 0)
CHECK (quantity > 0)
CHECK (amount >= 0)
```

Another service, migration, admin script, or future developer could bypass application validation — database constraints are the final safety layer.

---

## 11. DTO validation

```ts
export class CreateOrderItemDto {
  @IsUUID()
  productId: string;

  @IsInt()
  @Min(1)
  quantity: number;
}

export class CreateOrderDto {
  @IsArray()
  @ValidateNested({ each: true })
  @Type(() => CreateOrderItemDto)
  items: CreateOrderItemDto[];
}
```

```text
API validation → Business validation → Database constraints
```

Each layer has a purpose.

---

## 12. Creating an order

```json
{
  "items": [
    { "productId": "product-1", "quantity": 2 },
    { "productId": "product-2", "quantity": 1 }
  ]
}
```

Needs to: validate products, reserve inventory, calculate prices, create order, create order items — consistently, hence a transaction.

---

## 13. Order transaction

```ts
await this.dataSource.transaction(async manager => {
  // 1. Load products
  // 2. Reserve stock
  // 3. Calculate total
  // 4. Create order
  // 5. Create order items
});
```

```text
BEGIN → operations → COMMIT
```

or

```text
BEGIN → error → ROLLBACK
```

---

## 14. Inventory race condition

`stock = 1`, two users order simultaneously. Don't do:

```ts
const product = await manager.findOneBy(Product, { id: productId });

if (product.stock > 0) {
  product.stock -= quantity;
  await manager.save(product);
}
```

This suffers from concurrent read/write races. Instead, use an atomic conditional update with a parameterized quantity:

```ts
const result = await manager
  .createQueryBuilder()
  .update(Product)
  .set({ stock: () => 'stock - :quantity' })
  .setParameters({ quantity })
  .where('id = :productId', { productId })
  .andWhere('stock >= :quantity', { quantity })
  .execute();
```

Don't interpolate `quantity` directly into the SQL string. Conceptually:

```sql
UPDATE products
SET stock = stock - :quantity
WHERE id = :productId
AND stock >= :quantity;
```

(See also [Chapter 14 §10](14-typeorm-transactions-concurrency.md).)

---

## 15. Check affected rows

```ts
if (result.affected !== 1) {
  throw new BadRequestException('Insufficient stock');
}
```

The database guarantees `stock >= quantity` at the time of the update.

---

## 16. Calculate order total

`Product A: ₹500 × 2 = ₹1,000`, `Product B: ₹800 × 1 = ₹800`, total = `₹1,800`.

> Calculate the order using the current product price and save that price into `order_items.unit_price`. Don't calculate historical order totals later from `products.price`, since product prices change.

---

## 17. Create the order

```ts
const order = manager.create(Order, {
  user,
  status: 'PENDING',
  totalAmount: totalAmount.toString(),
});

await manager.save(order);

const orderItem = manager.create(OrderItem, {
  order,
  product,
  quantity,
  unitPrice: product.price,
});

await manager.save(orderItem);
```

---

## 18. What should NOT happen inside this transaction?

Don't do:

```ts
await manager.save(order);
await paymentProvider.createPayment(...);
await sendEmail(...);
await generateInvoice(...);
```

Instead:

```text
DB transaction: Create order, Reserve stock, Create order items → COMMIT
```

Then payment orchestration, email, invoice, notifications happen separately — depending on consistency requirements, via an outbox/event-driven architecture.

---

## 19. The Outbox Pattern

An important Tech Lead-level pattern. After creating an order, side effects (email, warehouse notification, analytics, push notification) shouldn't risk rolling back the order if one fails:

```text
DB transaction → send email → API failure → ROLLBACK  ❌
```

Instead store an event inside the same DB transaction:

```text
orders
outbox_events
```

```text
BEGIN
INSERT order
INSERT order_items
INSERT outbox_event
COMMIT
```

A worker then processes `outbox_events → message/event worker → email, warehouse, notifications, analytics`. Much more reliable.

---

## 20. Order creation flow

```text
POST /orders
       │
       ▼
Validate DTO
       │
       ▼
Begin transaction
       │
       ├── Validate user
       ├── Reserve inventory
       ├── Calculate total
       ├── Create order
       ├── Create order_items
       └── Create outbox event
       │
       ▼
    COMMIT
       │
       ▼
Return order
       │
       ▼
Async workers
       ├── Payment
       ├── Email
       ├── Warehouse
       └── Notifications
```

---

## 21. Order listing API

```http
GET /orders?page=1&limit=20
```

Don't do `orderRepository.find();` with millions of orders. Use pagination:

```ts
const [orders, total] = await orderRepository.findAndCount({
  where: { user: { id: userId } },
  order: { createdAt: 'DESC' },
  skip,
  take: limit,
});
```

For very large datasets, use keyset/cursor pagination instead ([Chapter 16 §10](16-typeorm-performance-architecture.md)).

---

## 22. Index the query pattern

Query is `WHERE user_id = ? ORDER BY created_at DESC`:

```sql
CREATE INDEX idx_orders_user_created
ON orders(user_id, created_at DESC);
```

> Index the access pattern, not the entity definition.

---

## 23. Order details

```http
GET /orders/:id
```

Needs `Order → User` and `Items → Product`:

```ts
const order = await orderRepository
  .createQueryBuilder('order')
  .innerJoinAndSelect('order.user', 'user')
  .leftJoinAndSelect('order.items', 'item')
  .innerJoinAndSelect('item.product', 'product')
  .where('order.id = :orderId', { orderId })
  .getOne();
```

> Don't automatically load every relation for every endpoint. Design the query around the API's actual response.

---

## 24. Payment processing

```http
POST /payments
```

```json
{ "orderId": "order-123", "amount": "1800" }
```

Header: `Idempotency-Key: abc-123`

Needs to: validate order, prevent duplicate processing, call payment provider, update payment state.

---

## 25. Don't use SELECT as your only idempotency protection

Bad:

```ts
const existing = await paymentRepository.findOne({ where: { idempotencyKey } });

if (existing) {
  return existing;
}

await paymentRepository.save(...);
```

Two requests can race. `UNIQUE(idempotency_key)` is the actual database guarantee (see [Chapter 9 §20](09-window-functions-upsert-interview-queries.md)).

---

## 26. Payment state machine

Define valid states:

```text
PENDING → PROCESSING → SUCCESS
PENDING → PROCESSING → FAILED
```

Plus possibly `REFUNDED`, `PARTIALLY_REFUNDED`. Treat payment status as a state machine.

---

## 27. Why state machines matter

Avoid setting `payment.status = 'SUCCESS';` from anywhere in the codebase. Want controlled transitions:

```text
PENDING → PROCESSING
PROCESSING → SUCCESS
PROCESSING → FAILED
SUCCESS → REFUNDED
```

But `SUCCESS → PENDING` may be invalid — this belongs in business logic.

---

## 28. Webhook idempotency

Payment providers may send the same webhook multiple times (`SUCCESS, SUCCESS, SUCCESS`). The handler must be safe to run repeatedly. Use `provider_payment_id UNIQUE`:

```text
Webhook → find/insert provider reference → if already processed, return success → otherwise, transaction
```

---

## 29. Payment transaction

```ts
await dataSource.transaction(async manager => {
  const payment = await manager
    .getRepository(Payment)
    .findOne({
      where: { providerPaymentId },
      lock: { mode: 'pessimistic_write' },
    });

  if (payment.status === 'SUCCESS') {
    return;
  }

  payment.status = 'SUCCESS';
  await manager.save(payment);

  await manager.update(Order, { id: payment.order.id }, { status: 'PAID' });
});
```

Exact implementation depends on relation loading and TypeORM version — the architecture is what matters.

---

## 30. Inventory + payment consistency

> "What happens if payment succeeds but inventory reservation fails?"

Distributed-system thinking starts here. Can't rely on one giant transaction across PostgreSQL + Stripe. Use explicit states — e.g. `Order: PENDING_PAYMENT` then `Payment: SUCCESS`, with inventory state managed through a reliable workflow. Depending on requirements: reservation, timeout, compensation, refund, retry.

---

## 31. Don't confuse database transaction with distributed transaction

Extremely important. A PostgreSQL transaction guarantees atomicity across PostgreSQL operations A and B — it does not make PostgreSQL + Stripe + Redis + Kafka one ACID transaction.

For distributed systems: idempotency, outbox, retries, compensation, state machines, eventual consistency.

---

## 32. Production architecture

```text
                     API
                      │
                      ▼
                 NestJS App
                      │
       ┌──────────────┼──────────────┐
       │              │              │
    Users          Orders         Payments
       │              │              │
       └──────────────┼──────────────┘
                      │
                 PostgreSQL
                      │
             ┌────────┴────────┐
             │                 │
           Redis            Outbox
                               │
                               ▼
                            Workers
                         ┌─────┼─────┐
                         ▼     ▼     ▼
                       Email  Queue  Webhooks
```

---

## 33. Where Redis fits

Redis shouldn't replace PostgreSQL for transactional business data. Good for: caching, rate limiting, distributed locks (carefully), session storage, queues, temporary state, counters.

PostgreSQL: orders, payments, users, inventory, financial records.

---

## 34. Where queues fit

Order creation needing email, push notification, analytics, invoice generation, warehouse notification shouldn't make the HTTP request wait for all of them:

```text
Create order → Commit → Publish/process event → Workers
```

Reduces API latency and isolates failures.

---

## 35. Observability

Track: API latency, DB query latency, error rate, transaction duration, connection pool usage, lock waits, cache hit ratio, queue depth, payment failures.

Should be able to trace a slow request: `Request ID → Controller → Service → SQL query → PostgreSQL`.

---

## 36. Tech Lead architecture checklist

**Data**: What are the entities? Relationships? Invariants?

**PostgreSQL**: What constraints? Indexes? Transactions? Locking?

**TypeORM**: Repository? QueryBuilder? Raw SQL? Relations?

**Performance**: N+1? Pagination? Connection pool? Query plan?

**Distributed systems**: Idempotency? Retries? Outbox? Queues? Compensation?

**Production**: Migrations? Logging? Metrics? Tracing? Backups?

---

## 37. The complete order workflow

```text
                CREATE ORDER
                     │
                     ▼
               Validate DTO
                     │
                     ▼
             Begin DB transaction
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
      Validate    Reserve     Calculate
      products    inventory   total
          │          │          │
          └──────────┼──────────┘
                     ▼
                Create Order
                     │
                     ▼
              Create OrderItems
                     │
                     ▼
               Create Outbox
                     │
                     ▼
                   COMMIT
                     │
                     ▼
               Return response
                     │
                     ▼
                Async workers
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Payment     Email     Warehouse
```

This is backend architecture, not just TypeORM.

---

## 38. Tech Lead interview answer

> "Design an order system using NestJS, PostgreSQL and TypeORM."

> I'd model users, orders, order items, products, payments, and inventory as relational entities. Order items would preserve the purchase-time unit price rather than depending on the product's current price. I'd use foreign keys and database constraints for integrity and composite indexes based on actual access patterns.
>
> Order creation would use a short PostgreSQL transaction. Inventory reservation would use an atomic conditional update or appropriate row locking to prevent overselling. Payment processing would be idempotent using a unique idempotency key and provider transaction ID. I wouldn't keep the database transaction open while calling the external payment provider.
>
> For asynchronous side effects such as notifications and warehouse events, I'd use an outbox/queue-based approach. For read APIs, I'd use repository methods for simple queries and QueryBuilder or parameterized SQL for complex queries. I'd avoid N+1 queries, use keyset pagination for large datasets, and inspect generated SQL with EXPLAIN ANALYZE when performance issues occur.
>
> Schema changes would be managed through migrations rather than synchronize, and I'd design migrations to be backward-compatible with rolling deployments.

Demonstrates system thinking, not just TypeORM syntax.

---

## Chapter 18 challenge

**Scenario**: `Product A, stock = 1, price = ₹1,000`. Two users simultaneously `POST /orders` for `{ "items": [{ "productId": "A", "quantity": 1 }] }` at exactly the same time.

The system must guarantee: only one order succeeds, stock never goes negative, the successful order records ₹1,000, and the failed request receives "out of stock."

Design the flow. Think about: transaction, atomic UPDATE, affectedRows, order creation, OrderItem, rollback, index.

Then the harder version: what changes if payment must also happen before the order is considered successful?

---

## Next → Chapter 19 — Distributed Transactions & Saga Pattern

PostgreSQL transaction vs distributed transaction → payment failures → inventory reservation → Saga → compensating actions → Outbox → idempotent consumers → retries → dead-letter queues → exactly-once vs at-least-once delivery.
