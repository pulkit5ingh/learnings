# Chapter 11 — TypeORM Fundamentals

Learning TypeORM by mapping every concept to what's already known from MongoDB/Mongoose.

## 1. What is TypeORM?

**TypeORM = ORM (Object-Relational Mapper) for TypeScript/JavaScript.** Lets you work with PostgreSQL using TypeScript objects instead of writing SQL for every basic operation.

### MongoDB → TypeORM mental model

| MongoDB / Mongoose | PostgreSQL / TypeORM |
|---|---|
| Schema | Entity |
| Model | Repository / Entity |
| Document | Entity instance / Row |
| Field | Column |
| `_id` | Primary key |
| `find()` | `repository.find()` |
| `findOne()` | `repository.findOne()` |
| `populate()` | Relation + JOIN |
| Aggregation | QueryBuilder / SQL |
| Mongo transaction | TypeORM transaction |
| `createIndex()` | `@Index()` / SQL index |

For a Tech Lead: TypeORM doesn't replace SQL knowledge — it generates SQL for you. Always be able to understand the SQL underneath the ORM query.

---

## 2. Your first Entity

Mongoose:

```ts
const UserSchema = new Schema({
  name: String,
  email: { type: String, unique: true },
  age: Number,
});
```

TypeORM:

```ts
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
} from 'typeorm';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ type: 'text' })
  name: string;

  @Column({ type: 'text', unique: true })
  email: string;

  @Column({ type: 'int', nullable: true })
  age?: number;

  @CreateDateColumn({ type: 'timestamptz' })
  createdAt: Date;

  @UpdateDateColumn({ type: 'timestamptz' })
  updatedAt: Date;
}
```

Represents:

```text
users
--------------------------------
id
name
email
age
created_at
updated_at
```

`@Entity('users')` means this TypeScript class maps to the `users` table.

---

## 3. Primary Key

UUID:

```ts
@PrimaryGeneratedColumn('uuid')
id: string;
```

Roughly equivalent to `id UUID PRIMARY KEY DEFAULT gen_random_uuid()`.

Or auto-incrementing integer:

```ts
@PrimaryGeneratedColumn()
id: number;
```

Conceptually an identity-generated integer.

MongoDB: `_id: ObjectId(...)` → TypeORM: `id: UUID`.

---

## 4. Columns

```ts
@Column()
name: string;

@Column({ type: 'text' })
name: string;

@Column({ type: 'int' })
age: number;

@Column({ type: 'boolean' })
isActive: boolean;

@Column({ type: 'text', nullable: true })
phone?: string;

@Column({ type: 'numeric', precision: 12, scale: 2 })
balance: string;
```

For money: prefer `NUMERIC`, not floating-point types (see [Chapter 3](03-tables-datatypes-crud.md)).

---

## 5. CreateDateColumn / UpdateDateColumn

Instead of manually managing `createdAt`/`updatedAt`:

```ts
@CreateDateColumn({ type: 'timestamptz' })
createdAt: Date;

@UpdateDateColumn({ type: 'timestamptz' })
updatedAt: Date;
```

TypeORM manages these columns automatically.

---

## 6. camelCase vs snake_case

TypeScript prefers `createdAt`, `updatedAt`, `isActive`; production PostgreSQL commonly uses `created_at`, `updated_at`, `is_active`. These are separate concerns — map explicitly:

```ts
@CreateDateColumn({ name: 'created_at', type: 'timestamptz' })
createdAt: Date;
```

```text
TypeScript             PostgreSQL
createdAt       --->    created_at
updatedAt       --->    updated_at
isActive        --->    is_active
```

A naming strategy can automate this across the project.

---

## 7. Repository

Mongoose:

```ts
User.findOne({ email: 'pulkit@example.com' });
```

TypeORM:

```ts
const user = await userRepository.findOne({
  where: { email: 'pulkit@example.com' },
});
```

The repository is the main interface for interacting with an entity's table.

---

## 8. Repository injection in NestJS

```ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
}
```

```ts
const user = await this.userRepository.findOne({ where: { email } });
```

---

## 9. Basic CRUD — CREATE

Mongoose:

```ts
await User.create({ name: 'Pulkit', email: 'pulkit@example.com' });
```

TypeORM:

```ts
const user = this.userRepository.create({
  name: 'Pulkit',
  email: 'pulkit@example.com',
});

await this.userRepository.save(user);
```

`create()` does **not** insert into PostgreSQL — it creates an entity object. `save()` actually persists it.

```text
create() → TypeScript object
save()   → SQL INSERT/UPDATE
```

---

## 10. Find all

Mongoose: `User.find();`

TypeORM: `const users = await this.userRepository.find();`

With conditions:

```ts
const users = await this.userRepository.find({
  where: { isActive: true },
});
```

Equivalent SQL:

```sql
SELECT * FROM users WHERE is_active = true;
```

---

## 11. Find one

```ts
const user = await this.userRepository.findOne({
  where: { email: 'pulkit@example.com' },
});
```

Or, for simple conditions:

```ts
const user = await this.userRepository.findOneBy({
  email: 'pulkit@example.com',
});
```

---

## 12. Find by ID

```ts
const user = await this.userRepository.findOneBy({ id: userId });
```

Mongo equivalent: `User.findById(userId);`

---

## 13. INSERT directly

```ts
await this.userRepository.insert({
  name: 'Pulkit',
  email: 'pulkit@example.com',
});
```

Useful when you just want an INSERT without the full entity lifecycle behavior.

**Interview distinction**: `save()` is entity-oriented; `insert()` is direct INSERT-oriented. For bulk/high-throughput operations, direct operations can be preferable.

---

## 14. UPDATE

```ts
await this.userRepository.update(
  { id: userId },
  { name: 'Pulkit Singh' },
);
```

Conceptually:

```sql
UPDATE users SET name = 'Pulkit Singh' WHERE id = '...';
```

No need to fetch the user first. Don't unnecessarily do:

```ts
const user = await repository.findOneBy({ id });
user.name = 'Pulkit';
await repository.save(user);
```

when a simple direct update suffices:

```ts
await repository.update({ id }, { name: 'Pulkit' });
```

---

## 15. DELETE

```ts
await this.userRepository.delete({ id: userId });
```

Equivalent: `DELETE FROM users WHERE id = ...;`

Or by ID directly: `await this.userRepository.delete(userId);`

---

## 16. `delete()` vs `remove()`

A common interview question.

**`delete()`** — direct database operation, doesn't require loading the entity first:

```ts
await repository.delete(id);
```

**`remove()`** — works with an entity instance:

```ts
const user = await repository.findOneBy({ id });
await repository.remove(user);
```

```text
delete() → Direct SQL DELETE
remove() → Entity-based removal
```

For simple deletion, `delete()` avoids an unnecessary SELECT.

---

## 17. `save()` vs `insert()` vs `update()`

Very important for interviews.

| Method | Purpose |
|---|---|
| `create()` | Create entity object |
| `save()` | Insert or update entity |
| `insert()` | Direct INSERT |
| `update()` | Direct UPDATE |
| `delete()` | Direct DELETE |
| `remove()` | Remove entity instance |

```ts
const user = repository.create(data);
await repository.save(user);
```

vs the more direct:

```ts
await repository.insert(data);
```

---

## 18. TypeORM doesn't hide SQL completely

```ts
await repository.find({ where: { isActive: true } });
```

eventually executes SQL similar to:

```sql
SELECT * FROM users WHERE is_active = true;
```

As a Tech Lead, always ask "what SQL is this generating?" — watch for slow queries, missing indexes, N+1 queries, unnecessary joins, huge result sets, inefficient pagination. This is why the PostgreSQL fundamentals matter.

---

## 19. A complete NestJS service

```ts
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}

  async create(data: { name: string; email: string }) {
    const user = this.userRepository.create(data);
    return this.userRepository.save(user);
  }

  async findAll() {
    return this.userRepository.find();
  }

  async findById(id: string) {
    return this.userRepository.findOneBy({ id });
  }

  async findByEmail(email: string) {
    return this.userRepository.findOneBy({ email });
  }

  async updateName(id: string, name: string) {
    return this.userRepository.update({ id }, { name });
  }

  async delete(id: string) {
    return this.userRepository.delete(id);
  }
}
```

The basic TypeORM foundation.

---

## 20. Production warning: `synchronize: true`

```ts
TypeOrmModule.forRoot({
  type: 'postgres',
  ...
  synchronize: true,
});
```

Automatically tries to synchronize entities with the database schema.

- **Development** — can be convenient.
- **Production** — generally do not use it. Use **migrations** instead (covered later).

---

## MongoDB → TypeORM cheat sheet

```ts
// Mongo: User.findOne({ email });
userRepository.findOneBy({ email });

// Mongo: User.find();
userRepository.find();

// Mongo: User.create(data);
userRepository.create(data);
await userRepository.save(user);

// Mongo: User.updateOne({ _id: id }, { $set: { name } });
userRepository.update({ id }, { name });

// Mongo: User.deleteOne({ _id: id });
userRepository.delete(id);
```

```text
Mongo populate()          → TypeORM relations / JOIN / QueryBuilder
Mongo aggregation pipeline → TypeORM QueryBuilder / raw SQL
```

---

## Chapter 11 interview questions

1. What's the difference between `save()`, `insert()`, and `update()` in TypeORM?
2. What's the difference between `delete()` and `remove()`?
3. Why shouldn't `synchronize: true` be used in production?
4. How does a TypeORM Entity map to a PostgreSQL table?
5. What SQL would TypeORM approximately generate for `repository.findOneBy({ email });`?
6. If you need to update one field and don't need the existing entity, would you use `findOne() + save()` or `update()`? Why?

---

## Next: Chapter 12 — TypeORM Relationships

Building:

```text
User
 └── Orders
      └── OrderItems
           └── Product
```

`@OneToMany`, `@ManyToOne`, `@OneToOne`, `@ManyToMany`, `@JoinColumn`, `@JoinTable`, eager/lazy loading, JOINs, and the N+1 problem.
