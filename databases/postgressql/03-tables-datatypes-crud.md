# Chapter 2 — PostgreSQL Tables, Data Types & CRUD

Writing actual PostgreSQL, with every concept translated back to MongoDB.

---

## 1. Creating a table

MongoDB doesn't usually require explicitly defining a collection schema. PostgreSQL does.

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    age INTEGER,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

Roughly the MongoDB document shape:

```javascript
{
    _id: ObjectId,
    name: String,
    email: String,
    age: Number,
    createdAt: Date
}
```

But PostgreSQL additionally tells the database:

```text
id          → must be unique
name        → cannot be NULL
email       → cannot be NULL
email       → must be unique
created_at  → automatically generated
```

That's the important difference.

---

## 2. PostgreSQL data types

### INTEGER

```sql
age INTEGER
```

Conceptually equivalent to `age: Number`.

### BIGINT

```sql
views BIGINT
```

Use when values can become very large: `views`, `likes`, `event_count`, large IDs.

### NUMERIC — very important for payments

Don't use floating-point types for money. Use:

```sql
price NUMERIC(10,2)
```

Allows values like `99999999.99`.

```sql
CREATE TABLE payments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    amount NUMERIC(12,2) NOT NULL
);
```

**Interview: why not `FLOAT` for money?**

> Floating-point numbers can introduce precision errors, so exact numeric types such as `NUMERIC` are preferred for monetary values.

---

## 3. TEXT vs VARCHAR

```sql
name VARCHAR(100)
description TEXT
```

`VARCHAR(100)` limits the value to 100 characters. `TEXT` imposes no length limit. For many PostgreSQL applications, `TEXT` is perfectly reasonable when there's no meaningful business maximum — don't blindly use `VARCHAR(255)` everywhere just because it's common.

---

## 4. BOOLEAN

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE
);
```

MongoDB: `{ isActive: true }`

---

## 5. Dates

`DATE`, `TIMESTAMP`, `TIMESTAMPTZ`.

```sql
created_at TIMESTAMPTZ DEFAULT NOW()
```

`TIMESTAMPTZ` is generally the right choice when you need an absolute point in time and your application serves users across time zones.

---

## 6. UUID

MongoDB's `ObjectId("...")` equivalent:

```sql
id UUID PRIMARY KEY DEFAULT gen_random_uuid()
```

```text
550e8400-e29b-41d4-a716-446655440000
```

UUID isn't required everywhere — PostgreSQL can also use `BIGSERIAL` or identity columns. UUID vs BIGINT trade-offs come up in a later interview chapter.

---

## 7. INSERT

MongoDB:

```javascript
db.users.insertOne({
    name: "Pulkit",
    email: "pulkit@example.com",
    age: 28
});
```

PostgreSQL:

```sql
INSERT INTO users (name, email, age)
VALUES ('Pulkit', 'pulkit@example.com', 28);
```

Explicitly specifying columns (`INSERT INTO users (name, email, age)`) is good SQL practice.

---

## 8. Insert multiple records

MongoDB:

```javascript
db.users.insertMany([
    { name: "Pulkit", age: 28 },
    { name: "Rahul", age: 30 },
    { name: "Amit", age: 25 }
]);
```

PostgreSQL:

```sql
INSERT INTO users (name, email, age)
VALUES
    ('Pulkit', 'pulkit@example.com', 28),
    ('Rahul', 'rahul@example.com', 30),
    ('Amit', 'amit@example.com', 25);
```

---

## 9. SELECT

MongoDB: `db.users.find();`

PostgreSQL: `SELECT * FROM users;`

Avoid blindly using `SELECT *` in production. Prefer:

```sql
SELECT id, name, email
FROM users;
```

A table may grow to 30 columns while your API only needs 3 — `SELECT *` transfers unnecessary data. Interviewers notice this.

---

## 10. Find one user

MongoDB:

```javascript
db.users.findOne({ email: "pulkit@example.com" });
```

PostgreSQL:

```sql
SELECT id, name, email
FROM users
WHERE email = 'pulkit@example.com'
LIMIT 1;
```

Because `email` is `UNIQUE`, PostgreSQL knows there can only be one matching user.

---

## 11. WHERE

MongoDB:

```javascript
db.users.find({ age: { $gt: 25 } });
```

PostgreSQL:

```sql
SELECT * FROM users WHERE age > 25;
```

| MongoDB | PostgreSQL |
|---|---|
| `$gt` | `>` |
| `$gte` | `>=` |
| `$lt` | `<` |
| `$lte` | `<=` |
| `$eq` | `=` |
| `$ne` | `<>` |

---

## 12. AND

MongoDB:

```javascript
db.users.find({ age: { $gte: 25 }, isActive: true });
```

PostgreSQL:

```sql
SELECT *
FROM users
WHERE age >= 25
  AND is_active = TRUE;
```

---

## 13. OR

MongoDB:

```javascript
db.users.find({ $or: [{ age: 25 }, { age: 30 }] });
```

PostgreSQL:

```sql
SELECT * FROM users WHERE age = 25 OR age = 30;
```

Cleaner:

```sql
SELECT * FROM users WHERE age IN (25, 30);
```

---

## 14. IN

MongoDB:

```javascript
{ status: { $in: ["paid", "completed"] } }
```

PostgreSQL:

```sql
SELECT * FROM orders WHERE status IN ('paid', 'completed');
```

---

## 15. NULL — very important

You **cannot** write `WHERE email = NULL` — that's wrong. Use:

```sql
WHERE email IS NULL
WHERE email IS NOT NULL
```

`NULL` means "unknown / missing value" — it isn't equal to anything, including another `NULL`.

---

## 16. UPDATE

MongoDB:

```javascript
db.users.updateOne(
    { email: "pulkit@example.com" },
    { $set: { age: 29 } }
);
```

PostgreSQL:

```sql
UPDATE users
SET age = 29
WHERE email = 'pulkit@example.com';
```

**Never** accidentally write `UPDATE users SET age = 29;` without a `WHERE` clause unless you actually want to update every row. Same principle applies to `DELETE`.

---

## 17. DELETE

MongoDB:

```javascript
db.users.deleteOne({ email: "pulkit@example.com" });
```

PostgreSQL:

```sql
DELETE FROM users
WHERE email = 'pulkit@example.com';
```

`DELETE FROM users;` with no `WHERE` deletes every row. This is why production systems often implement **soft deletes**:

```sql
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMPTZ;

UPDATE users
SET deleted_at = NOW()
WHERE id = '...';
```

instead of physically deleting the row.

---

## 18. RETURNING — PostgreSQL superpower

```sql
INSERT INTO users (name, email, age)
VALUES ('Pulkit', 'pulkit@example.com', 28)
RETURNING id, name, email;
```

PostgreSQL can immediately return the inserted record. Same with `UPDATE`:

```sql
UPDATE users
SET age = 29
WHERE email = 'pulkit@example.com'
RETURNING id, age;
```

And `DELETE`:

```sql
DELETE FROM users
WHERE id = '...'
RETURNING id;
```

This becomes very useful with TypeORM.

---

## 19. ORDER BY

MongoDB: `db.users.find().sort({ age: -1 });`

PostgreSQL:

```sql
SELECT * FROM users ORDER BY age DESC;
```

Ascending: `ORDER BY age ASC;`

Multiple columns:

```sql
SELECT * FROM users ORDER BY age DESC, name ASC;
```

Meaning: sort by age descending; ties broken by name ascending.

---

## 20. LIMIT

MongoDB: `db.users.find().limit(10);`

PostgreSQL: `SELECT * FROM users LIMIT 10;`

---

## 21. OFFSET

MongoDB: `db.users.find().skip(20).limit(10);`

PostgreSQL:

```sql
SELECT * FROM users LIMIT 10 OFFSET 20;
```

Fine for small datasets, but can become problematic at very large offsets — a full lesson on pagination comes later.

---

## 22. A realistic example

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE,
    age INTEGER,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

INSERT INTO users (name, email, age)
VALUES
    ('Pulkit', 'pulkit@example.com', 28),
    ('Rahul', 'rahul@example.com', 30),
    ('Amit', 'amit@example.com', 24);

SELECT id, name, email
FROM users
WHERE age >= 25
ORDER BY age DESC
LIMIT 10;
```

Mentally:

```text
MongoDB:

find({ age: { $gte: 25 } })
    .sort({ age: -1 })
    .limit(10)
```

---

## 23. TypeORM preview

The SQL:

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL
);
```

as a TypeORM entity:

```typescript
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ type: 'text' })
  name: string;

  @Column({ type: 'text', unique: true })
  email: string;
}
```

```typescript
const user = await userRepository.findOne({
  where: { email: 'pulkit@example.com' },
});
```

TypeORM generates SQL behind the scenes:

```text
Your NestJS code
      ↓
TypeORM
      ↓
SQL
      ↓
PostgreSQL
      ↓
Rows
```

Learn SQL first, before becoming dependent on TypeORM — a Tech Lead should be able to look at TypeORM code and understand the SQL it produces.

---

## Interview checkpoint #2

### Q1

MongoDB:

```javascript
db.users.find({ age: { $gte: 25 }, isActive: true });
```

Write the PostgreSQL query.

### Q2

Why is this wrong?

```sql
SELECT * FROM users WHERE email = NULL;
```

### Q3

Given:

```sql
users
-----
id
email

orders
------
id
user_id
amount
```

Write a query returning `user_id | total_amount` for users whose total order value is greater than ₹10,000.

### Q4 — Tech Lead question

Why might this be dangerous?

```sql
UPDATE users
SET is_active = false;
```

### Q5 — TypeORM

What does this do?

```typescript
const user = await userRepository.findOne({
  where: { email: 'pulkit@example.com' },
});
```

Next: **Chapter 3 — SQL JOINs** (`INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL JOIN`, and mentally converting MongoDB `$lookup` pipelines into SQL).
