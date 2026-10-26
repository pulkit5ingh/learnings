# Chapter 15 — TypeORM Migrations & Production Database Changes

A very important production topic. Creating tables is easy — the real challenge is safely changing a database that already contains millions of production records while users are actively using the application.

---

## 1. What is a migration?

A versioned database change.

```text
v1
users
├── id
├── name
└── email
```

Adding `phone` — instead of manually running `ALTER TABLE users ADD COLUMN phone TEXT;`, create a migration `v2 AddPhoneToUsers`. The database moves `v1 → v2`, and potentially back `v2 → v1` via a rollback migration.

---

## 2. MongoDB comparison

MongoDB application-level migrations:

```ts
db.users.updateMany({}, { $set: { newField: null } });
```

TypeORM migrations make schema/data changes versioned, tracked, repeatable, deployable, and reviewable in Git.

```text
MongoDB migration script ≈ TypeORM migration
```

---

## 3. `synchronize: true` vs migrations

`synchronize: true` tells TypeORM to automatically synchronize the database schema with entities — convenient in development, risky in production.

**Development**: `Entity changed → TypeORM → Database automatically changes`

**Production**: `Entity changed → Migration created → Code review → Migration tested → Deploy → Migration runs`

> Don't rely on `synchronize: true` for production schema management.

---

## 4. Why migrations are important

With `10 million users` / `500 million orders`, an automatic schema change gives up explicit control over what SQL executes, when, how long it takes, whether it locks a table, how it rolls back, and how it behaves across environments. Migrations make the database change explicit.

---

## 5. Migration structure

```ts
import { MigrationInterface, QueryRunner } from 'typeorm';

export class AddPhoneToUsers1234567890 implements MigrationInterface {
  async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      ALTER TABLE users
      ADD COLUMN phone TEXT
    `);
  }

  async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      ALTER TABLE users
      DROP COLUMN phone
    `);
  }
}
```

```text
up()   → apply migration
down() → rollback migration
```

---

## 6. `up()` and `down()`

Current: `users(id, name, email)`. `up()` runs `ALTER TABLE users ADD COLUMN phone TEXT;` → `users(id, name, email, phone)`. `down()` runs `ALTER TABLE users DROP COLUMN phone;` → back to `users(id, name, email)`.

---

## 7. Migration version tracking

TypeORM maintains migration history in the database:

```text
migrations
--------------------------------
id
timestamp
name
```

If production has run `001, 002, 003` and the deployment adds `004`, running migrations applies only `004`.

---

## 8. Typical workflow

```text
1. Modify Entity
       ↓
2. Generate/create migration
       ↓
3. Review migration SQL
       ↓
4. Test locally
       ↓
5. Test staging
       ↓
6. Deploy
       ↓
7. Run migration
```

---

## 9. Entity and database schema must stay synchronized

If the entity declares `@Column({ type: 'text' }) phone: string;` but production's database doesn't have `phone`, the application can fail. Entity + Migration must be managed together to produce the database schema.

---

## 10. Adding a nullable column

Generally one of the easier changes. `users(id, name, email)` + `phone TEXT NULL`:

```sql
ALTER TABLE users
ADD COLUMN phone TEXT;
```

Existing rows get `phone = NULL` — no immediate backfill required.

---

## 11. Dangerous migration: adding NOT NULL

`10 million users`, want `phone TEXT NOT NULL`. Existing users don't have phone numbers, so this fails against rows that violate `NOT NULL`. Don't blindly run:

```sql
ALTER TABLE users
ADD COLUMN phone TEXT NOT NULL;
```

---

## 12. Safe multi-step migration

**Step 1 — Add nullable column**, then deploy:

```sql
ALTER TABLE users
ADD COLUMN phone TEXT;
```

**Step 2 — Backfill**:

```sql
UPDATE users
SET phone = 'unknown'
WHERE phone IS NULL;
```

For millions of rows, don't run one huge update — batch it.

**Step 3 — Enforce constraint** after verifying `SELECT COUNT(*) FROM users WHERE phone IS NULL;` returns `0`:

```sql
ALTER TABLE users
ALTER COLUMN phone SET NOT NULL;
```

```text
Add nullable → Deploy application → Backfill → Verify → Enforce NOT NULL
```

A classic production migration pattern.

---

## 13. Expand → Migrate → Contract

A very important Tech Lead concept. Renaming `name` to `full_name`:

- **Phase 1 — Expand**: add `full_name`, keep `name`.
- **Phase 2 — Application compatibility**: deploy code that works with both — write both, or read `full_name` with fallback to `name`.
- **Phase 3 — Backfill**: copy `name → full_name`.
- **Phase 4 — Switch reads/writes**: application starts using `full_name`.
- **Phase 5 — Contract**: once no code depends on `name`, `DROP COLUMN name;`

```text
Expand → Migrate → Switch → Contract
```

---

## 14. Why this matters in distributed deployments

During rolling deployment, `API Server 1/2 → old code`, `API Server 3 → new code` can coexist. If new code immediately removes `name`, old servers crash.

> Database changes should usually be backward-compatible during rolling deployments.

A very strong Tech Lead concept.

---

## 15. Renaming a column safely

Bad: `Deploy new code → Rename name → full_name → Old servers crash`.

Better: `Migration 1: Add full_name → Deploy compatible code → Backfill → Deploy code using full_name → Verify → Migration 2: Remove name`.

---

## 16. Large table migrations

`orders = 500 million rows`, adding `CREATE INDEX idx_orders_user_id ON orders(user_id);`. Creating an index consumes CPU, memory, disk I/O, and locks, potentially affecting production traffic.

PostgreSQL offers `CREATE INDEX CONCURRENTLY ...` to reduce blocking of normal writes:

```sql
CREATE INDEX CONCURRENTLY idx_orders_user_id
ON orders(user_id);
```

Important: `CREATE INDEX CONCURRENTLY` cannot run inside a normal transaction, so migration transaction behavior needs to be configured appropriately.

---

## 17. Why indexes matter during migrations

Adding `user_id` alone doesn't make `SELECT * FROM orders WHERE user_id = ...;` fast — a matching index may also be needed. Think about the complete change: schema + constraints + indexes + application query pattern.

---

## 18. Data migration vs schema migration

**Schema migration** — change structure: `ALTER TABLE users ADD COLUMN phone TEXT;`

**Data migration** — change existing data: `UPDATE users SET phone = ... WHERE ...;`

Sometimes both are needed: add column → backfill → add constraint.

---

## 19. Don't put huge data migrations blindly inside one transaction

`500 million rows` in one massive `UPDATE users SET new_value = old_value;` risks a huge transaction, WAL growth, long locks, replication lag, I/O pressure, table bloat, and a difficult rollback.

For large datasets, consider batching, background jobs, controlled backfill, and monitoring:

```text
10,000 rows → commit → 10,000 rows → commit → ...
```

The exact strategy depends on the migration and consistency requirements.

---

## 20. Migration rollback isn't always simple

If a migration drops a column and the data wasn't backed up, a `down()` that runs `ADD COLUMN` doesn't restore the lost data.

> A migration's `down()` method is not a substitute for backups.

---

## 21. Never casually modify an already-applied migration

If `001-create-users.ts` has already run in production, don't edit it to mean something different — create `002-add-phone.ts` instead. Production's migration history (`001, 002, 003`) is effectively part of the database's historical record, and other environments may have already applied the old migration.

---

## 22. Migration deployment strategy

```text
Build application → Run migration → Deploy application
```

or

```text
Deploy backward-compatible application → Run migration → Switch traffic
```

The correct order depends on whether the schema change is backward compatible.

> Don't create a period where the application expects schema that doesn't exist, or the database removes schema still required by running application instances.

---

## 23. TypeORM migration commands

```bash
typeorm migration:generate
typeorm migration:run
typeorm migration:revert
```

(Exact invocation varies by project setup — CLI directly vs npm scripts.)

```text
migration:generate → create migration
migration:run      → apply migrations
migration:revert   → rollback latest migration
```

---

## 24. Production checklist

- **Schema** — what exactly changes?
- **Data** — do existing rows satisfy the new constraint?
- **Locks** — could this block queries/writes?
- **Size** — how many rows are affected?
- **Indexes** — does this need a new index?
- **Deployment** — can old and new application versions coexist?
- **Rollback** — can we safely reverse this?
- **Backup** — can lost data be recovered?
- **Monitoring** — how will we know the migration is hurting production?

---

## 25. A realistic example

`users(id, email, name)`. Requirement: users can have a profile picture.

**Bad approach** — immediately add `profilePictureUrl` as `NOT NULL` and deploy. Existing users break.

**Better approach**:

- **Migration 1**: `ALTER TABLE users ADD COLUMN profile_picture_url TEXT;` (nullable)
- **Application deployment**: new code understands the column but handles NULL
- **Backfill**: existing users get a default/profile image if applicable
- **Migration 2**: after verifying all records, `ALTER TABLE users ALTER COLUMN profile_picture_url SET NOT NULL;`

---

## 26. Zero-downtime mindset

Not "can I change the schema?" but "can I change the schema while production continues serving traffic safely?"

```text
Old App ────────┐
                ├── Database
New App ────────┘
```

Both may temporarily interact with the same database during deployment.

```text
Backward compatible schema → Rolling deployment → Data migration → Constraint tightening → Cleanup
```

---

## 27. Tech Lead interview question

> "We have 100 million users and need to make `phone` NOT NULL. How would you deploy it?"

> I wouldn't add the NOT NULL constraint immediately. I'd first add the column as nullable, deploy application code that can handle both old and new states, then backfill existing records in controlled batches. I'd monitor the backfill and verify that there are no remaining NULLs. After that I'd enforce the NOT NULL constraint. If the change involved an index or potentially blocking DDL, I'd evaluate the PostgreSQL locking behavior and use an appropriate online/concurrent strategy where supported. I'd also make sure the migration is compatible with rolling deployments.

---

## Chapter 15 mental model

```text
Entity change
     ↓
Migration
     ↓
Review
     ↓
Test
     ↓
Backward compatibility
     ↓
Deploy
     ↓
Backfill if needed
     ↓
Constraint/index
     ↓
Monitor
```

> `synchronize: true` is a development convenience; migrations are the controlled production mechanism.

---

## Interview challenges

### Q1
Why shouldn't you use `synchronize: true` in production?

### Q2
Rename `users.name` to `users.full_name` — how would you perform this with zero/minimal downtime?

### Q3
Add `email_verified BOOLEAN NOT NULL` to a table containing 50 million users. What's your migration strategy?

### Q4
Difference between schema migration and data migration?

### Q5
Why can this be dangerous on a huge production table?

```sql
UPDATE users
SET new_column = old_column;
```

### Q6 — Tech Lead
You deploy `v2` of your API, but during rolling deployment `3 servers → v1, 2 servers → v2`. Your migration immediately drops a column used by v1. What can go wrong, and how would you redesign the deployment?

---

## Next → Chapter 16 — TypeORM Performance & Production Architecture

N+1 → relation loading → indexes → QueryBuilder → pagination → connection pooling → caching → transactions → raw SQL → logging → slow-query debugging → repository/service architecture → when TypeORM becomes a bottleneck.
