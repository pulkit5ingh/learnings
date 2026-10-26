# Chapter 12 — TypeORM Relationships

Connecting TypeORM to the PostgreSQL relationship concepts already covered ([Chapter 5](05-relationships-database-design.md)).

> MongoDB `populate()` → PostgreSQL FK + TypeORM relations + JOIN

---

## 1. The basic relationship

```text
User
 |
 | 1
 |
 |------< many
 |
Order
```

```text
users
  id
  name

orders
  id
  user_id  → users.id
  amount
```

The foreign key lives in `orders` because `orders` is the "many" side.

---

## 2. `@ManyToOne`

```ts
@Entity('orders')
export class Order {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ type: 'numeric', precision: 12, scale: 2 })
  amount: string;

  @ManyToOne(() => User, user => user.orders)
  user: User;
}
```

Tells TypeORM: many orders belong to one user. Specify the actual FK column:

```ts
@ManyToOne(() => User, user => user.orders)
@JoinColumn({ name: 'user_id' })
user: User;
```

Now the database has:

```text
orders
----------------
id
user_id
amount
```

---

## 3. `@OneToMany`

```ts
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ type: 'text' })
  name: string;

  @OneToMany(() => Order, order => order.user)
  orders: Order[];
}
```

Together:

```ts
// User
@OneToMany(() => Order, order => order.user)
orders: Order[];

// Order
@ManyToOne(() => User, user => user.orders)
@JoinColumn({ name: 'user_id' })
user: User;
```

---

## 4. Which side owns the relationship?

A very important interview question. The `@ManyToOne` side owns the relationship because it contains the FK.

```text
User → OneToMany → Order → ManyToOne → User
```

Physically, `orders.user_id` is the foreign key. Therefore `@ManyToOne` is normally the owning side — same principle as the PostgreSQL rule that the FK belongs on the "many" side.

---

## 5. Loading the relationship

```ts
const user = await userRepository.findOne({ where: { id } });
```

Does **not** necessarily load orders. Request the relation explicitly:

```ts
const user = await userRepository.findOne({
  where: { id },
  relations: { orders: true },
});
```

Conceptually:

```sql
SELECT ...
FROM users
LEFT JOIN orders
ON orders.user_id = users.id
WHERE users.id = ...;
```

---

## 6. MongoDB comparison

```ts
// MongoDB
const user = await User.findById(id).populate('orders');

// TypeORM
const user = await userRepository.findOne({
  where: { id },
  relations: { orders: true },
});
```

```text
MongoDB: populate() → fetch referenced documents
TypeORM: relations → JOIN related tables
```

---

## 7. QueryBuilder for relationships

```ts
const user = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.orders', 'order')
  .where('user.id = :id', { id })
  .getOne();
```

Returns `User` with `orders[]`. Note `.where('user.id = :id', { id })` uses parameters — never `.where(`user.id = '${id}'`)`. Always use parameterized queries.

---

## 8. `leftJoinAndSelect` vs `leftJoin`

**`leftJoinAndSelect`** — `.leftJoinAndSelect('user.orders', 'order')` — JOIN the orders and include them in the returned entity.

**`leftJoin`** — `.leftJoin('user.orders', 'order')` — JOIN the table for filtering/querying, but don't automatically include the joined data in the returned entity.

```ts
const users = await userRepository
  .createQueryBuilder('user')
  .leftJoin('user.orders', 'order')
  .where('order.amount > :amount', { amount: 1000 })
  .getMany();
```

Useful when only filtering users based on their orders, not returning order data.

---

## 9. One-to-One

```text
User
 |
 | 1
 |
Profile
```

```ts
@Entity('profiles')
export class Profile {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @OneToOne(() => User)
  @JoinColumn({ name: 'user_id' })
  user: User;
}

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @OneToOne(() => Profile, profile => profile.user)
  profile: Profile;
}
```

Database rule: a one-to-one relationship requires a **UNIQUE foreign key**:

```sql
user_id UUID UNIQUE REFERENCES users(id)
```

Otherwise `profile 1 → user 1, profile 2 → user 1, profile 3 → user 1` could exist — which is actually one-to-many.

---

## 10. Many-to-Many

```text
User
 |
 | many
 |------ many
 |
Course
```

Junction table:

```text
user_courses
user_id
course_id
```

```ts
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToMany(() => Course)
  @JoinTable({ name: 'user_courses' })
  courses: Course[];
}

@Entity('courses')
export class Course {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @ManyToMany(() => User, user => user.courses)
  users: User[];
}
```

`@JoinTable()` is placed on **one side only** — that side owns the many-to-many relationship.

---

## 11. Don't blindly use `ManyToMany`

A common architecture mistake. `User ←→ Course` starts as:

```ts
@ManyToMany()
courses: Course[];
```

but then needs `enrolled_at`, `progress`, `status`, `completed_at` — the relationship now has business data. Create a real entity instead:

```text
Enrollment
----------------
id
user_id
course_id
progress
status
enrolled_at
completed_at
```

```text
User → 1 → Enrollment → many → Course
```

Usually much more flexible.

**Tech Lead rule**: if the relationship itself has attributes, make the relationship a real entity. (See [Chapter 5 §11](05-relationships-database-design.md).)

---

## 12. Cascade

```ts
@OneToMany(() => Order, order => order.user, { cascade: true })
orders: Order[];
```

Can cause related entities to be automatically persisted. `cascade: true` is **not** the same as PostgreSQL `ON DELETE CASCADE` — they operate at different layers.

- **TypeORM cascade** — controls ORM persistence behavior.
- **PostgreSQL `ON DELETE CASCADE`** — controls database FK deletion behavior.

Don't confuse them.

---

## 13. Eager loading

```ts
@ManyToOne(() => User, user => user.orders, { eager: true })
user: User;
```

Whenever an Order is loaded, TypeORM automatically loads the User. Convenient, but dangerous:

```text
Order → User → Profile → ...
```

Automatically loading relationships everywhere can cause unnecessary JOINs, large queries, performance problems, and hard-to-understand query behavior. For large production systems, explicitly controlling what's loaded is usually easier to reason about.

---

## 14. The N+1 problem

One of the most important TypeORM interview topics.

```ts
const users = await userRepository.find();

for (const user of users) {
  const orders = await orderRepository.find({
    where: { user: { id: user.id } },
  });
}
```

100 users → `1 query (users) + 100 queries (orders) = 101 queries`. The **N+1 problem**.

---

## 15. Fixing N+1

```ts
const users = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.orders', 'order')
  .getMany();
```

Fetches everything through one SQL query:

```sql
SELECT ...
FROM users
LEFT JOIN orders
  ON orders.user_id = users.id;
```

A major reason to understand SQL even when using TypeORM.

---

## 16. JOIN isn't always automatically better

"Always use JOIN" is also wrong. `1 million users`, `10 million orders`, but only 20 users with basic fields are needed — fetching millions of joined rows would be bad.

The correct question: what data do I actually need, and what is the query pattern? Depending on the use case: JOIN, separate queries, aggregation, pagination, caching, denormalized data, or materialized views.

---

## 17. Real-world example

```http
GET /users/:id/orders
```

```json
{
  "id": "user-123",
  "name": "Pulkit",
  "orders": [{ "id": "order-1", "amount": 2500, "status": "PAID" }]
}
```

```ts
const user = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.orders', 'order')
  .where('user.id = :id', { id })
  .getOne();
```

But for `GET /orders?status=PAID&page=1`, there's no need to fetch the User entity and all relationships — query Orders directly.

---

## 18. The architecture pattern

```text
users/
├── users.controller.ts
├── users.service.ts
├── user.entity.ts
└── dto/

orders/
├── orders.controller.ts
├── orders.service.ts
├── order.entity.ts
└── dto/
```

```text
Controller → Service → Repository → TypeORM → PostgreSQL
```

Service owns business logic, repository handles persistence, controller handles HTTP.

---

## 19. Tech Lead interview answer

> "How would you model User → Order in TypeORM?"

> I'd model it as a one-to-many relationship. The `Order` entity would have a `ManyToOne` relationship to `User` and own the foreign key through `user_id`. The `User` entity would expose the inverse `OneToMany` relationship. I'd explicitly load orders when required rather than making them globally eager, and I'd use QueryBuilder or optimized repository queries for more complex access patterns. I'd also watch for N+1 queries and make sure the foreign key and common filtering columns are indexed.

Much stronger than just naming the decorators.

---

## Mental model

```text
                    PostgreSQL
                         │
                         ▼
                    Foreign Key
                         │
                         ▼
                    TypeORM Entity
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
       @ManyToOne               @OneToMany
             │                       │
             └───────────┬───────────┘
                         ▼
                    JOIN / Query
                         │
                         ▼
                    API Response
```

```text
MongoDB:     Reference → populate() → Documents
PostgreSQL:  Foreign Key → JOIN → Rows
```

---

## Chapter 12 checkpoint

### Q1
Where does the FK go: `User 1 ────────< Order`?

### Q2
Why is `@ManyToOne` usually the owning side?

### Q3
Difference between `leftJoin()` and `leftJoinAndSelect()`?

### Q4
What is N+1 and how would you detect/fix it?

### Q5
`User ↔ Course` needs `progress`, `enrolledAt`, `status`. Would you use `@ManyToMany` or create an `Enrollment` entity?

---

## Next → Chapter 13: TypeORM QueryBuilder

Complex WHERE → JOIN → GROUP BY → HAVING → subqueries → aggregates → pagination → raw SQL → dynamic filters → SQL injection prevention → performance.
