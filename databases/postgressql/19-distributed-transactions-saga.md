# Chapter 19 — Distributed Transactions & Saga Pattern

One of the most important topics for a Senior Backend Engineer / Tech Lead — real systems rarely consist of PostgreSQL alone.

```text
Order Service
     ↓
PostgreSQL
     ↓
Payment Provider
     ↓
Inventory Service
     ↓
Email Service
     ↓
Kafka / RabbitMQ
```

> A PostgreSQL transaction can roll back PostgreSQL changes, but it cannot roll back an external API call.

---

## 1. Local Transaction vs Distributed Transaction

**Local transaction** — everything inside PostgreSQL:

```text
BEGIN
Create Order
Create Order Items
Reduce Stock
COMMIT
```

Failure → `ROLLBACK` — PostgreSQL undoes everything.

**Distributed workflow**:

```text
Create Order → Reserve Inventory → Charge Payment → Send Email
```

involving PostgreSQL, Inventory Service, Stripe, Email Service. Can't do:

```sql
BEGIN;
INSERT INTO orders ...
CALL stripe.charge(...); -- ❌
ROLLBACK;
```

PostgreSQL cannot tell Stripe to roll back a payment. That's where the **Saga Pattern** comes in.

---

## 2. The Classic Failure Problem

`Order = ₹5,000`, `Inventory = RESERVED`, `Payment = SUCCESS`, then the application crashes before updating the order. Possible resulting inconsistencies:

```text
Payment: SUCCESS, Inventory: RESERVED, Order: PENDING
Order: CREATED, Inventory: RESERVED, Payment: FAILED
Order: CONFIRMED, Payment: SUCCESS, Inventory: FAILED
```

Distributed-system design needs to handle these.

---

## 3. State Machines

Not "the order either exists or doesn't" — think states and valid transitions.

**Order**: `CREATED → PENDING_PAYMENT → CONFIRMED`, or `CREATED → PENDING_PAYMENT → PAYMENT_FAILED → CANCELLED`.

**Payment**: `PENDING → PROCESSING → SUCCESS`, or `PROCESSING → FAILED`, possibly `SUCCESS → REFUNDED`.

**Inventory reservation**: `AVAILABLE → RESERVED → CONFIRMED`, or `RESERVED → RELEASED`.

A good backend system doesn't blindly update status fields — e.g. `SUCCESS → PENDING` should probably be invalid (see [Chapter 18 §26-27](18-production-nestjs-project.md)).

---

## 4. Saga Pattern

A Saga breaks a distributed transaction into multiple local transactions:

```text
Create Order → Reserve Inventory → Process Payment → Confirm Order
```

Each step has its own transaction. If something fails, execute a **compensating action**.

Normal flow: `Create Order → Reserve Inventory → Payment SUCCESS → Confirm Order`

Payment fails: `Create Order → Reserve Inventory → Payment FAILED → Release Inventory → Cancel Order`

We aren't "rolling back" the inventory transaction — we perform another operation (`RESERVE → RELEASE`). That's **compensation**.

---

## 5. Saga Compensation

| Action | Compensation |
|---|---|
| Create order | Cancel order |
| Reserve inventory | Release inventory |
| Charge payment | Refund payment |
| Create shipment | Cancel shipment |
| Apply coupon | Restore coupon |

Compensation isn't always a perfect mathematical rollback — "Send email" can't be truly undone, only followed up with a correction email. Saga is a business-level consistency mechanism, not database rollback.

---

## 6. Two Types of Saga — Choreography

Services react to events, no central controller:

```text
Order Service → OrderCreated
Inventory Service → InventoryReserved
Payment Service → PaymentSucceeded
Order Service → OrderConfirmed
```

**Advantage**: less central orchestration.

**Problem**: with many services (Order → Inventory → Payment → Shipping → Notification → Loyalty → Analytics) the distributed chain of events becomes difficult to understand.

---

## 7. Orchestration

A central Saga orchestrator:

```text
             Saga Orchestrator
              /      |      \
             ↓       ↓       ↓
         Inventory Payment  Shipping
```

```text
Order Created → Saga Orchestrator → Reserve Inventory → Process Payment → Create Shipment → Confirm Order
```

Payment fails: `Payment Failed → Saga Orchestrator → Release Inventory → Cancel Order`

For complex business workflows, orchestration is often easier to reason about.

---

## 8. Our E-Commerce Example

```text
Client → POST /orders → Order Service → PostgreSQL
```

Transaction:

```text
BEGIN
Create order, status = PENDING_PAYMENT
Reserve inventory
Create payment, status = PENDING
Create outbox event: PAYMENT_REQUESTED
COMMIT
```

Then `Outbox Worker → Payment Provider`, with `idempotencyKey = payment_123`.

---

## 9. Why Outbox?

If the DB transaction succeeds, then the application publishes a Kafka event, then crashes before that publish completes — the DB shows `Order = CREATED` but Kafka never received `OrderCreated`. Inconsistent system.

**Outbox Pattern**:

```text
BEGIN
INSERT order
INSERT outbox_event
COMMIT
```

Both happen atomically.

```text
orders(id, status, total)
outbox_events(id, event_type, aggregate_id, payload, status, retry_count, created_at, processed_at)
```

```text
PostgreSQL Transaction
        ├── Order
        └── Outbox Event
```

Either both are committed or neither is.

---

## 10. Outbox Worker

Continuously reads `outbox_events` (e.g. `PAYMENT_REQUESTED`, `ORDER_CONFIRMED`, `INVENTORY_RELEASE_REQUESTED`) and publishes/processes them:

```ts
while (true) {
  const events = await getPendingEvents();

  for (const event of events) {
    await process(event);
  }
}
```

Concurrency problem: multiple workers processing the same event. PostgreSQL's `FOR UPDATE SKIP LOCKED` solves this.

---

## 11. FOR UPDATE SKIP LOCKED

```sql
SELECT *
FROM outbox_events
WHERE processed_at IS NULL
ORDER BY created_at
FOR UPDATE SKIP LOCKED
LIMIT 100;
```

Worker A locks Event 1, 2; Worker B skips them and takes Event 3, 4. Useful for database-backed job queues.

Don't do:

```text
BEGIN
  lock row
  call Stripe
  wait 5 seconds
  COMMIT
```

Holds database locks while waiting on a network request. Better:

```text
Claim/lease job → COMMIT → Call external service → Update result
```

---

## 12. Idempotent Consumers

Extremely important. Kafka/RabbitMQ delivers `PAYMENT_SUCCESS`, the consumer crashes, the broker doesn't know if processing succeeded, so it redelivers `PAYMENT_SUCCESS`. The consumer must safely handle duplicates.

**Bad**: `orderRepository.update(orderId, { status: 'CONFIRMED' });` is usually fine alone, but if the handler also adds loyalty points, sends money, creates a shipment, sends email — running it twice creates duplicate side effects.

---

## 13. Idempotency Using Event ID

```json
{ "eventId": "evt_123", "type": "PAYMENT_SUCCESS", "orderId": "order_123" }
```

Create `processed_events` with `event_id UNIQUE`:

```text
BEGIN
INSERT processed_event(event_id) → duplicate? → reject
Perform business operation
COMMIT
```

The unique constraint protects against concurrent duplicates. A very strong pattern.

---

## 14. At-Least-Once Delivery

Most practical messaging systems aim for at-least-once delivery: a message may be delivered more than once but shouldn't be silently lost. The consumer must therefore be idempotent, safely handling `Message, Message, Message` arriving three times.

---

## 15. Exactly-Once Myth

"Our system guarantees exactly-once processing" — be careful. Across database + network + message broker + external API, true exactly-once execution is extremely difficult.

Consumer → Stripe charges the card successfully, but the network connection dies before the response arrives. The application doesn't know whether payment happened. Blindly retrying could double-charge. This is why payment providers support idempotency keys.

---

## 16. Payment Idempotency

Generate a stable key `payment_123`, send `POST /payment` with `Idempotency-Key: payment_123`. On retry with the same key, the provider recognizes it as the same request:

```text
Timeout → Retry → Same idempotency key → No duplicate charge
```

One of the most important production concepts in payment systems.

---

## 17. Retry Strategy

**Temporary error** (network timeout, 503, connection reset, rate limit) → retry.

**Permanent error** (invalid card, invalid product ID, invalid request) → usually don't retry indefinitely.

**Exponential backoff**: `1s, 2s, 4s, 8s, 16s` instead of retrying immediately, with **jitter**: `delay = exponentialBackoff + randomJitter`. Without jitter, 10,000 workers retrying at the same instants create another traffic spike; jitter spreads the retries.

---

## 18. Dead Letter Queue

After repeated retries still fail, send the event to a **DLQ**:

```text
Main Queue → Consumer → Failure → Retry → Retry → Retry → DLQ
```

Engineers can then inspect, fix the problem, and replay the event.

---

## 19. Complete Payment Workflow

```text
                    Client
                      │
                      ▼
                 POST /orders
                      │
                      ▼
                Order Service
                      │
                 DB Transaction
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Order      Inventory     Payment
      PENDING      RESERVED      PENDING
          │
          ▼
      Outbox Event
   PAYMENT_REQUESTED
          │
       COMMIT
          │
          ▼
     Outbox Worker
          │
          ▼
    Payment Provider
          │
          ▼
       Webhook
          │
          ▼
     Payment Service
          │
     DB Transaction
          │
     ┌────┴─────────┐
     ▼              ▼
Payment SUCCESS   Order CONFIRMED
                      │
                      ▼
                 Outbox Event
                 ORDER_CONFIRMED
                      │
             ┌────────┼────────┐
             ▼        ▼        ▼
           Email   Warehouse Analytics
```

---

## 20. What if Payment Fails?

`Order: PENDING_PAYMENT`, `Inventory: RESERVED`, `Payment: FAILED`. Saga compensation:

```text
Payment FAILED → Release Inventory → Inventory RELEASED → Cancel Order → Order CANCELLED
```

---

## 21. What if Payment Times Out?

A tricky interview question. Request sent to the payment provider, then `TIMEOUT` — you cannot assume payment failed. It could be that the request never reached the provider, or the provider processed it but the response was lost.

```text
TIMEOUT → Payment = PROCESSING → Retry using SAME idempotency key (or query the provider) → Webhook eventually arrives → SUCCESS / FAILED
```

Never blindly create another payment request with a new idempotency key.

---

## 22. Failure Matrix

| Failure | Correct strategy |
|---|---|
| DB transaction fails | Rollback |
| Inventory reservation fails | Cancel order |
| Payment timeout | Keep processing/pending |
| Payment succeeds but webhook delayed | Wait/reconcile |
| Duplicate webhook | Idempotent handler |
| Payment permanently fails | Release inventory + cancel |
| Outbox publish fails | Retry |
| Consumer crashes | Redeliver |
| Consumer repeatedly fails | DLQ |
| External API timeout | Retry/query with idempotency |
| Worker crashes | Reclaim/lease and retry |

---

## 23. TypeORM Outbox Entity

```ts
@Entity('outbox_events')
export class OutboxEvent {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ name: 'event_type', type: 'text' })
  eventType: string;

  @Column({ name: 'aggregate_id', type: 'uuid' })
  aggregateId: string;

  @Column({ type: 'jsonb' })
  payload: Record<string, unknown>;

  @Column({ type: 'int', default: 0 })
  retryCount: number;

  @Column({ name: 'processed_at', type: 'timestamptz', nullable: true })
  processedAt: Date | null;

  @CreateDateColumn({ name: 'created_at', type: 'timestamptz' })
  createdAt: Date;
}
```

TypeScript stays camelCase (`retryCount`, `processedAt`, `createdAt`) while PostgreSQL uses `retry_count`, `processed_at`, `created_at`.

---

## 24. Creating Order + Outbox Atomically

```ts
await dataSource.transaction(async manager => {
  const order = await manager.save(Order, {
    userId,
    status: 'PENDING_PAYMENT',
    total,
  });

  await manager.save(OutboxEvent, {
    eventType: 'PAYMENT_REQUESTED',
    aggregateId: order.id,
    payload: { orderId: order.id, paymentId },
  });
});
```

Order + Outbox Event are committed together.

---

## 25. The Most Important Mental Model

**Local transaction**: `BEGIN → DB operations → COMMIT / ROLLBACK`

**Distributed workflow**: `Local transaction → Event → Another local transaction → Event → Another local transaction`

Failures handled via state machines + idempotency + retries + compensation + outbox + DLQ — not one giant transaction.

---

## 26. Senior/Tech Lead Interview Question

> "How would you design a reliable order and payment system where payment is handled by Stripe?"

> I would not keep a PostgreSQL transaction open while calling Stripe. I would create the order, reserve inventory, create a pending payment record, and write an outbox event in one local transaction. An asynchronous worker would process the payment request using a stable idempotency key. Stripe webhooks would be treated as the source of payment state changes, and the webhook handler would be idempotent. Payment and order state transitions would be validated using state machines. If payment permanently fails, the Saga would compensate by releasing the inventory and cancelling the order. Retries would use exponential backoff with jitter, and messages that repeatedly fail would go to a DLQ.

A very strong Senior Backend answer.

---

## Chapter 19 interview questions

### Q1
Why can't a PostgreSQL transaction safely include a Stripe API call?

### Q2
What happens if Stripe successfully charges the customer but your API times out?

### Q3
Why do we need idempotency keys?

### Q4
Difference between Rollback and Compensation?

### Q5
Why is at-least-once + idempotent consumer commonly preferred over trying to achieve exactly-once execution?

### Q6 — Tech Lead level

Design: `Order → Inventory → Payment → Shipping → Notification` where any service can fail and messages can be duplicated or delayed. Be able to explain: state machines, Saga, outbox, idempotency, retries, DLQ, compensation, webhook handling.

---

## Next: Chapter 20 — PostgreSQL Advanced Concurrency & Locking

MVCC, row locks, `FOR UPDATE`, `FOR NO KEY UPDATE`, `FOR SHARE`, `SKIP LOCKED`, deadlocks, advisory locks, optimistic locking, serializable transactions, race conditions, high-concurrency inventory, job queues using PostgreSQL.
