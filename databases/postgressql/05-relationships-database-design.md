# Chapter 5 — PostgreSQL Relationships & Database Design

Moving from "I can write SQL" to "I can design a production database." Constantly comparing MongoDB embedding vs referencing with PostgreSQL relationships.

---

## 1. The 3 relationships you must know

```text
1. One-to-One       1 : 1
2. One-to-Many      1 : N
3. Many-to-Many     N : M
```

---

## 2. One-to-One — 1:1

Example: `User → Profile` — one user has exactly one profile.

MongoDB could embed:

```javascript
{
    _id: "...",
    name: "Pulkit",
    profile: {
        bio: "Backend Engineer",
        avatar: "..."
    }
}
```

or use separate documents. In PostgreSQL:

```text
users
-----
id
name

profiles
--------
id
user_id
bio
avatar
```

```sql
CREATE TABLE profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL UNIQUE,
    bio TEXT,
    avatar TEXT,

    FOREIGN KEY (user_id)
        REFERENCES users(id)
);
```

The important part is `user_id UUID UNIQUE`. Without `UNIQUE`, `user_id = 1` could appear multiple times, meaning one user has multiple profiles. `UNIQUE` enforces `User 1 → Profile 1`.

---

## 3. One-to-Many — 1:N

The most common relationship. Example: `User → Orders`.

MongoDB:

```javascript
orders
{
    _id: "...",
    userId: "user123",
    amount: 1000
}
```

PostgreSQL:

```text
users
-----
id
name

orders
------
id
user_id
amount
```

```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    amount NUMERIC(12,2) NOT NULL,

    FOREIGN KEY (user_id)
        REFERENCES users(id)
);
```

```text
users.id
   ↑
   |
orders.user_id
```

**The foreign key is stored on the "many" side.**

---

## 4. Why the FK lives on the many side

```text
User 1
 ├── Order 101
 ├── Order 102
 └── Order 103
```

Each order needs to know "who owns me?" — so `orders.user_id` references `users.id`. A core database-design rule.

---

## 5. One-to-Many in TypeORM

```typescript
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @OneToMany(() => Order, order => order.user)
  orders: Order[];
}

@Entity('orders')
export class Order {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column('numeric', { precision: 12, scale: 2 })
  amount: number;

  @ManyToOne(() => User, user => user.orders)
  user: User;
}
```

```text
User
 @OneToMany
      ↓
 Order
 @ManyToOne
```

---

## 6. The owning side

An important TypeORM interview concept: which side actually stores the foreign key? `orders.user_id` — so:

```typescript
@ManyToOne(() => User)
user: User;
```

is the **owning side**. `User.orders` is the inverse side.

```text
User
  │ inverse
  ▼
orders
  │ owns FK
  ▼
user_id
```

---

## 7. Many-to-Many

`Users ↔ Courses` — one user can take many courses, one course can have many users (`N : M`).

You can't put `course_id` inside `users` (a user can have multiple courses), and can't put `user_id` inside `courses` (a course can have multiple users). So create a **junction table**:

```text
users
-----
id
name

courses
-------
id
name

user_courses
------------
user_id
course_id
```

---

## 8. Junction table

```sql
CREATE TABLE user_courses (
    user_id UUID NOT NULL,
    course_id UUID NOT NULL,

    PRIMARY KEY (user_id, course_id),

    FOREIGN KEY (user_id)
        REFERENCES users(id),

    FOREIGN KEY (course_id)
        REFERENCES courses(id)
);
```

`PRIMARY KEY (user_id, course_id)` prevents the same `(user 1, course 10)` pair from being inserted twice.

---

## 9. MongoDB equivalent

```javascript
{
    _id: "user1",
    courses: ["course1", "course2", "course3"]
}
```

Fine for some workloads, but PostgreSQL's junction table gives a structured relationship:

```text
user_courses
----------------
user_id | course_id
-------------------
1       | 10
1       | 20
1       | 30
2       | 10
2       | 30
```

---

## 10. Many-to-Many in TypeORM

```typescript
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToMany(() => Course)
  @JoinTable()
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

`@JoinTable()` tells TypeORM which side manages the junction table.

---

## 11. A Tech Lead trap

Suppose the relationship has extra information — `enrolled_at`, `completed_at`, `progress`, `status`. Don't blindly use `@ManyToMany()`. Instead create a real entity:

```text
users
courses
enrollments
```

```text
enrollments
-----------
id
user_id
course_id
status
progress
enrolled_at
completed_at
```

A much better production design, because the relationship itself now carries business data. A strong Tech Lead interview point.

---

## 12. Cascading deletes

`User → Orders` — what happens when the user is deleted?

### Option 1 — Prevent deletion (default FK behavior)

```sql
FOREIGN KEY (user_id)
REFERENCES users(id)
```

If orders exist, deletion may fail depending on the FK action.

### Option 2 — Delete orders automatically

```sql
FOREIGN KEY (user_id)
REFERENCES users(id)
ON DELETE CASCADE
```

```text
DELETE user
     ↓
DELETE user's orders
```

Be careful: `User → Orders → Order Items → Payments` — a cascade can delete a huge amount of data. Don't blindly use `CASCADE` everywhere.

---

## 13. `ON DELETE SET NULL`

```sql
FOREIGN KEY (user_id)
REFERENCES users(id)
ON DELETE SET NULL
```

```text
User deleted
     ↓
Order remains
     ↓
user_id = NULL
```

Useful when historical records must remain. Requires `user_id UUID` (nullable), not `user_id UUID NOT NULL`.

---

## 14. `ON DELETE RESTRICT`

```sql
FOREIGN KEY (user_id)
REFERENCES users(id)
ON DELETE RESTRICT
```

You cannot delete the user while dependent records exist. Often useful for important business records.

---

## 15. Normalization

```text
orders
------------------------------------------------
id
user_name
user_email
user_phone
product_name
product_price
quantity
```

If Pulkit has 100 orders, `Pulkit / pulkit@example.com / 987xxxxxxx` is duplicated 100 times. Instead:

```text
users
-----
id
name
email
phone

orders
------
id
user_id
...
```

```text
users.id
   ↑
   |
orders.user_id
```

That's normalization.

---

## 16. Why normalize

- **Reduce duplication** — store the user once instead of once per order.
- **Improve consistency** — an email change updates `users.email` in one place.
- **Maintain relationships** — foreign keys enforce valid references.

---

## 17. Don't over-normalize

A Tech Lead-level point: normalization isn't "more tables = better." If an API needs `user / profile / address / preferences / subscription` and every request requires six joins, consider:

- denormalization
- materialized views
- caching
- read models
- JSONB
- dedicated query models

Database design depends on read patterns, write patterns, data consistency, scale, and latency requirements.

---

## 18. MongoDB vs PostgreSQL design mindset

**MongoDB**: "What data is retrieved together?" → consider embedding.

**PostgreSQL**: "What are my entities? What are their relationships? What constraints must always hold? What are my access patterns?" → normalize appropriately, then optimize based on actual workload.

---

## 19. Tech Lead scenario

> We're building an e-commerce application. Design the database for users, products, orders and payments.

Don't start writing tables immediately. First identify entities:

```text
User
Product
Order
OrderItem
Payment
```

Then relationships:

```text
User 1 ─── N Orders
Order 1 ─── N OrderItems
Product 1 ─── N OrderItems
Order 1 ─── N Payments
```

Potential schema:

```text
users
-----
id
name
email

products
--------
id
name
price

orders
------
id
user_id
status
total_amount
created_at

order_items
-----------
id
order_id
product_id
quantity
unit_price

payments
--------
id
order_id
amount
status
provider
transaction_id
created_at
```

Notice `order_items.unit_price` even though `products.price` already exists. Product price today might be ₹1,000; tomorrow it becomes ₹1,500 — but the historical order must still say `unit_price = ₹1,000`. Intentional duplication/denormalization for historical correctness — exactly the reasoning interviewers want from a Tech Lead.

---

## 20. Design challenge — food delivery application

Requirements:

```text
Users can place orders.
A restaurant has many menu items.
An order belongs to one user.
An order belongs to one restaurant.
An order contains multiple menu items.
A menu item can appear in many orders.
Each order item must remember the price at the time of purchase.
A user can have multiple addresses.
An order should store which address was used.
```

Identify:

- **Tables** — what tables would you create?
- **Relationships** — for each, is it `1:1`, `1:N`, or `N:M`?
- **Where does `price_at_purchase` belong?**
- **If the user changes their address after placing an order, should the old order's delivery address change?**

Think about the data model carefully before answering.

Next: **Chapter 6 — Transactions & ACID**, one of the highest-value PostgreSQL interview topics.
