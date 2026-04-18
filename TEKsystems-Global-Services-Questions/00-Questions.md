# TEKsystems Global Services — interview questions

## Role & experience

- Can you explain your current role and responsibilities?

- - Hi guys, my name is pulkit, and i have 5+ years of exprence as an Nodejs developer.
- - “I’m currently working as a Senior Backend Engineer at Corpusvision Technologies Pvt. Ltd., where I’m responsible for designing and building scalable backend systems, primarily using Node.js.

1. Backend System Design
   Design scalable APIs and microservices architecture
   Build high-performance systems using Node.js, Express/NestJS
   Ensure clean architecture and modular code

2. Microservices & Architecture
   Design event-driven systems using messaging queues (RabbitMQ, BullMQ)
   Handle inter-service communication and reliability (retries, DLQ)
   Optimize system for scalability and fault tolerance

3. Database Design & Optimization
   Design schemas (MongoDB, PostgreSQL)
   Optimize queries and indexing
   Implement caching strategies (Redis)

4. DevOps & Deployment
   Containerize applications using Docker
   Manage multi-service environments using Docker Compose
   Deploy and manage services on VPS/cloud
   Work with CI/CD pipelines

5. Team Leadership
   Lead a team of developers
   Conduct code reviews and maintain code quality
   Mentor junior developers
   Collaborate with product & clients

- What technologies are you currently working on?

- - I mainly work with Nodejs and its frameworks and libraries like Reactjs nextjs for frontend, expressjs and NestJS for backend development, RabbitMQ and BullMQ for messaging and background jobs, MongoDB and PostgreSQL for databases, redis and velki for caching, s3 for media file storage, and Docker-based setups for deployment and scaling.

1. Backend Development
   JavaScript / TypeScript
   Node.js (Express, NestJS)
   REST APIs & microservices architecture

2. Messaging & Async Processing
   RabbitMQ (event-driven communication)
   BullMQ (background jobs, retries, DLQ)
   Redis (caching + queue backend)

3. Databases
   MongoDB (NoSQL, aggregation pipelines)
   PostgreSQL (relational data, complex queries)

4. DevOps & Deployment
   Docker (containerization)
   Docker Compose (multi-service setup)
   Basic experience with Kubernetes (scaling & orchestration)

5. Cloud & Infrastructure
   VPS (DigitalOcean / cloud servers)
   CI/CD pipelines (GitHub Actions)

6. Other Tools & Concepts
   JWT authentication & RBAC
   API optimization & caching strategies
   Event-driven architecture
   System design & scalability

- What is your current role and responsibility?

- - “I’m currently working as a Senior Backend Engineer at Corpusvision Technologies Pvt. Ltd., where I focus on building scalable backend systems and leading technical development.”

- - Key Responsibilities
    1. Backend Development & System Design
       Design and develop scalable APIs using Node.js (Express / NestJS)
       Build modular, maintainable microservices architecture
       Ensure performance, scalability, and clean code practices

    2. Microservices & Messaging
       Design event-driven systems using RabbitMQ and BullMQ
       Handle inter-service communication with retries, DLQ, and async workflows
       Build background job processing systems
    3. Database Design
       Work with MongoDB and PostgreSQL
       Optimize queries, indexing, and data models
       Implement caching using Redis

    4. DevOps & Deployment
       Containerize applications using Docker
       Manage multi-service environments using Docker Compose
       Deploy and maintain applications on cloud/VPS
       Set up CI/CD pipelines

    5. Leadership & Collaboration
       Lead a team of developers and conduct code reviews
       Mentor junior engineers
       Collaborate with product teams and clients
       Take ownership of features from design to production

    6. Performance & Scalability
       Optimize APIs for high concurrency
       Design systems that handle real-world scale
       Monitor, debug, and improve system reliability

- - “I’ve led development of scalable platforms, improved system performance, and built production-ready systems that handle real-world traffic and business use cases.”

- How comfortable are you with Node.js (rate yourself out of 10)?

- How comfortable are you with SQL?

## Project & architecture

- How many services were there in your project?
- What features were handled in the admin panel?
- How did you manage live tracking?
- What is latitude and longitude?
- How many parallel requests were handled by your application?
- How many microservices were there and what were they?
- Can you explain a microservices-based architecture you have worked on?
- How do you communicate between different microservices?
- Are microservices stateful or stateless?
- Have you worked with microservices?
- Have you handled high-concurrency scenarios?
- Which platform is your application running on—AWS or GCP?

## Node.js runtime & concurrency

- Is Node.js single-threaded or multi-threaded?

- - Nodejs Runs on single thread
- - It uses Event-loop to handle multiple request efficiently
- - That is why it is called non-blocking /asynchronous

- What us EVENT-LOOP ?
- - The Event Loop is a mechanism in Node.js that continuously checks and executes callbacks from the queue, enabling asynchronous, non-blocking execution.
- - The event loop in Node.js is responsible for handling asynchronous operations by continuously checking the call stack and callback queue, ensuring non-blocking execution.
- - In simple words
    It’s like a manager that:
    1. Takes tasks
    2. Sends heavy work to background
    3. Executes callbacks when work is done
- - Step-by-step flow:
    1. Execute synchronous code
    2. Send async tasks (I/O, timers) to libuv
    3. Continue execution
    4. When tasks complete → callbacks go to queue
    5. Event loop picks them and executes

- Is Nodejs is synchronous or asynchronous / blocking or non-blocking in nature ?

- - Node.js is asynchronous and non-blocking by default, but it can also execute synchronous (blocking) code when explicitly written.
- - Node.js is single-threaded for JavaScript execution, but uses asynchronous non-blocking I/O via libuv, making it highly scalable for I/O-heavy applications.

- How do you manage multiple concurrent requests in Node.js?

- - For high concurrency, I use async non-blocking APIs, Redis caching, DB pooling, worker threads for CPU tasks, and horizontal scaling using clustering with a load balancer.

- How would you run a Node.js application in a multi-threaded environment?

- - To run Node.js in a multi-threaded environment, I use Worker Threads for CPU-intensive tasks and clustering to utilize multiple CPU cores. In production, I combine this with load balancing to scale horizontally.

- Can you explain how Node.js handles concurrency?

- - Node.js handles concurrency through its event loop and non-blocking I/O, delegating heavy operations to libuv while continuing to process other requests.

- when does Nodejs performs well ?

- - I/O-heavy apps (APIs, real-time, streaming)
- - Chat apps, notifications, live dashboards

- When does Node.js perform poorly?

- - Node.js performs poorly when your workload blocks the event loop—especially with CPU-heavy tasks, synchronous code, or massive in-memory processing.
- - CPU-Intensive Tasks - Node.js struggles when your app needs heavy computation.
- - Blocking (Synchronous) Code.
- - High Memory Consumption Workloads.
- - Poor Fit for Multi-Threaded Parallelism
- - Heavy Real-Time Systems Without Optimization
- - Frequent Large Database Joins / Complex Queries
- - Poor Error Handling & Unhandled Promises
- - use Use Worker Threads to fix it or / Offload to microservices (Python, Go, Rust)
- -

- Can you explain the phases of the event loop?

## SQL

- How comfortable are you with SQL?

- - I use SQL regularly with PostgreSQL (and sometimes MySQL) for reporting, joins, aggregations, and tuning slow queries with indexes and EXPLAIN.
- - I’m comfortable writing CRUD, JOINs, subqueries, GROUP BY, transactions, and basic performance tuning.

- What is SQL? What are the main types of SQL statements?

- - SQL (Structured Query Language) is used to define, read, update, and manage data in relational databases.
- - **DDL** — CREATE, ALTER, DROP (schema / objects)
- - **DML** — SELECT, INSERT, UPDATE, DELETE (data)
- - **DCL** — GRANT, REVOKE (permissions)
- - **TCL** — COMMIT, ROLLBACK, SAVEPOINT (transactions)

- What is the difference between `WHERE` and `HAVING`?

- - **`WHERE`** filters **rows before** grouping (runs on raw rows).
- - **`HAVING`** filters **groups after** `GROUP BY` (runs on aggregated results).
- - Example pattern: `WHERE status = 'active'` then `GROUP BY user_id` then `HAVING COUNT(*) > 5`.

- Explain `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, and `FULL OUTER JOIN`.

- - **INNER JOIN** — only rows that **match in both** tables.
- - **LEFT JOIN** — all rows from the **left** table + matching right rows (unmatched right columns are NULL).
- - **RIGHT JOIN** — same idea as LEFT but **right** table is preserved.
- - **FULL OUTER JOIN** — rows from **both** tables when either side matches (where supported).

- What are primary key and foreign key?

- - **Primary key** — uniquely identifies a row in a table; usually one per table; should be NOT NULL and unique.
- - **Foreign key** — column(s) that reference a primary key in another table; enforces **referential integrity** (prevents orphan rows).

- What is an index? When do you add one?

- - An **index** is a data structure (often B-tree) that speeds up **lookups, sorting, and joins** on indexed columns.
- - Add indexes for columns often used in **WHERE**, **JOIN**, **ORDER BY**; avoid over-indexing (slows writes and uses storage).
- - Composite index order matters: `(a, b)` helps queries filtering on `a` or `a`+`b`, not always on `b` alone.

- What is normalization? Why use it?

- - **Normalization** organizes data to reduce **redundancy** and **anomalies** (update/insert/delete).
- - Common forms: **1NF** (atomic columns), **2NF** (no partial dependency on composite keys), **3NF** (no transitive dependency on non-key attributes).
- - Sometimes denormalize **on purpose** for read-heavy reporting (trade space for speed).

- What are ACID properties?

- - **Atomicity** — all steps in a transaction succeed or none apply (all-or-nothing).
- - **Consistency** — data stays valid per rules/constraints before and after.
- - **Isolation** — concurrent transactions don’t corrupt each other’s intermediate state (depends on isolation level).
- - **Durability** — committed data survives crashes (written to durable storage).

- What is a transaction? When would you use `COMMIT` and `ROLLBACK`?

- - A **transaction** is a sequence of operations executed as **one logical unit**.
- - Use **`COMMIT`** to persist changes when everything succeeded.
- - Use **`ROLLBACK`** to undo changes on error or business rule failure (e.g. payment failed after inventory reserved).

- What is the difference between `DELETE`, `TRUNCATE`, and `DROP`?

- - **`DELETE`** — removes rows (can use WHERE); **logged row-by-row**; can rollback in a transaction (in most DBs).
- - **`TRUNCATE`** — removes **all rows** quickly; often **resets** identity counters; typically DDL-like, less flexible than DELETE.
- - **`DROP`** — removes the **whole table** (structure + data).

- What is the difference between `GROUP BY` and `ORDER BY`?

- - **`GROUP BY`** groups rows so you can use **aggregates** (`COUNT`, `SUM`, `AVG`, etc.).
- - **`ORDER BY`** sorts the **result set** (after SELECT); does not define grouping.
- - Typical: `GROUP BY category` then `ORDER BY total_sales DESC`.

- What is a subquery? When would you use it?

- - A **subquery** is a query nested inside another query (`WHERE`, `FROM`, `SELECT` list).
- - Use for **filtering** against computed sets, **existence** checks (`EXISTS`), or when a join is awkward.
- - Example idea: `WHERE user_id IN (SELECT user_id FROM orders WHERE total > 1000)`.

- How do you work with `NULL` in SQL?

- - `NULL` means **unknown** — not the same as `0` or empty string.
- - Use **`IS NULL` / `IS NOT NULL`** (not `= NULL`).
- - **`COALESCE(a, b)`** returns first non-NULL; **`NULLIF(a, b)`** returns NULL if equal.
- - Aggregates like `SUM`/`COUNT(column)` **ignore NULLs** in those cells (behavior differs for `COUNT(*)` vs `COUNT(col)`).

- What is the difference between `UNION` and `UNION ALL`?

- - **`UNION`** combines result sets and **removes duplicate** rows (extra sort/distinct work).
- - **`UNION ALL`** **keeps all rows** including duplicates — **faster** when duplicates are OK or impossible.

- How do you approach optimizing a slow SQL query?

- - Run **`EXPLAIN` / `EXPLAIN ANALYZE`** to see plan: seq scans vs index scans, cost, rows.
- - Add or fix **indexes** for filter/join columns; watch **select *** and unnecessary columns.
- - Avoid **leading wildcard** `LIKE '%x'` on large tables without full-text search.
- - Reduce **N+1** patterns in apps (batch joins or two queries instead of one query per row).
- - For huge reads, use **pagination** (`LIMIT`/`OFFSET` or keyset/cursor pagination).

- What are `VIEW`s and when would you use a stored procedure vs application code?

- - A **VIEW** is a saved **SELECT** (virtual table) — simplifies repeated queries and can hide complexity; underlying data still lives in base tables.
- - **Stored procedures** — logic runs **inside the DB** (good for heavy reporting, batch jobs, strict performance); harder to version in Git vs app code.
- - **Application code** — easier to test, deploy with your API; use when business rules change often or you need portability across DB vendors.

- What is a **CTE** (`WITH` clause)?

- - A **Common Table Expression** names a temporary result set for **one statement** — improves readability for multi-step logic.
- - You can chain CTEs and reference earlier CTEs; useful for **recursive** queries (e.g. org trees) with `WITH RECURSIVE` (PostgreSQL).

- What are **window functions**? Give a simple example idea.

- - Window functions compute over a **partition** of rows **without collapsing** them to one row per group (unlike plain `GROUP BY`).
- - **`ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC)`** — rank employees per department.
- - Others: **`RANK`**, **`DENSE_RANK`**, **`SUM(...) OVER (...)`** for running totals.

- What are **transaction isolation levels** and common read phenomena?

- - **Phenomena:** *dirty read* (uncommitted), *non-repeatable read*, *phantom read* (new rows appear in range).
- - **Isolation levels** (SQL standard): READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE — stronger isolation = fewer anomalies, often more locking.
- - Pick level based on consistency needs vs performance; many apps default to **READ COMMITTED**.

- What are **constraints** (`UNIQUE`, `CHECK`, `NOT NULL`, `DEFAULT`)?

- - **`NOT NULL`** — column must have a value.
- - **`UNIQUE`** — no duplicate values in column(s); can be multi-column unique.
- - **`CHECK`** — row must satisfy a boolean expression (e.g. `age >= 0`).
- - **`DEFAULT`** — value inserted when none provided — keeps data valid at insert time.

## TypeORM

- What is TypeORM? Why use it with NestJS?

- - **TypeORM** is an **ORM** for TypeScript/JavaScript that maps **classes (entities)** to database tables and uses **decorators** for schema definition.
- - Works with **PostgreSQL, MySQL, MariaDB, SQLite, MSSQL**, etc.
- - In **NestJS**, you plug it in via `@nestjs/typeorm` — **repository injection**, migrations, and typed queries instead of raw SQL everywhere.
- - Trade-off: faster development and type safety vs learning ORM behavior and sometimes heavier queries than hand-written SQL.

- What is an Entity in TypeORM?

- - An **entity** is a class decorated with `@Entity()` that represents a **table**; each property maps to a **column** (`@Column()`, `@PrimaryGeneratedColumn()`, etc.).
- - TypeORM can **sync** schema in dev (`synchronize: true` — avoid in production) or use **migrations** for safe changes.

- What is a Repository vs `DataSource` / `EntityManager`?

- - **`Repository<Entity>`** — CRUD for **one entity** (`find`, `findOne`, `save`, `delete`, `createQueryBuilder`).
- - **`EntityManager`** — same operations but can work across **multiple entities** in one transaction.
- - **`DataSource`** — single DB connection config + access to managers, repositories, and running migrations.

- How do you define relations (`@OneToMany`, `@ManyToOne`, `@ManyToMany`)?

- - **`@ManyToOne` / `@OneToMany`** — classic parent/child (e.g. User → many Orders); put `@JoinColumn()` on the **owning** side (FK column).
- - **`@ManyToMany`** — needs a **join table**; use `@JoinTable()` on one owning side or model an explicit **junction entity** for extra columns.
- - **`@OneToOne`** — one row maps to one row; still mark the owning side with `@JoinColumn()` where the FK lives.

- What is eager vs lazy loading of relations?

- - **Eager** (`{ eager: true }` on relation) — relation **always** loaded with the parent (easy but can fetch too much data).
- - **Lazy** — relation is a **Promise** loaded when accessed (less common in Nest; can surprise you with async).
- - **Best practice:** default **lazy off**, load explicitly with `relations: ['profile']` or **QueryBuilder** `leftJoinAndSelect` when needed.

- How do you avoid the N+1 problem in TypeORM?

- - **N+1** — one query for parents + one per child row when relations load in a loop.
- - Fix: use **`relations`** in `find` options, **`leftJoinAndSelect`** in QueryBuilder, or **`In()`** batch queries instead of looping `findOne` per id.
- - Use **`QueryBuilder`** for one SQL round-trip with joins when listing related data.

- Repository methods vs QueryBuilder — when use which?

- - **`find` / `findOne`** — simple filters, pagination (`skip`/`take`), `order`, `relations` — good for straightforward reads.
- - **`createQueryBuilder`** — complex **joins**, conditional WHERE, **aggregates**, subqueries, raw snippets — closer to SQL control.
- - **`QueryRunner` + raw query** — rare reporting or performance-critical paths (`dataSource.query(...)`).

- How do you run transactions in TypeORM?

- - **`dataSource.transaction(async (manager) => { ... })`** — pass transactional `EntityManager`; all operations inside commit or rollback together.
- - In NestJS, inject `DataSource` and wrap service methods in `transaction()` for **transfer money**, **create order + line items**, etc.
- - Avoid long-held transactions — keep them short to reduce lock contention.

- What are migrations and why not rely on `synchronize: true`?

- - **Migrations** are versioned SQL (or TypeORM API) scripts that alter schema **predictably** across dev/staging/prod.
- - **`synchronize: true`** auto-alters DB from entities — risky in production (**data loss**, unexpected drops).
- - Workflow: change entity → `typeorm migration:generate` → review → `migration:run` in CI/CD.

- What are cascades (`Cascade` option on relations)?

- - **`cascade: true`** on a relation can **insert/update/remove** related entities when you save/remove the parent (e.g. save User + new Profiles in one `save`).
- - Use carefully — wrong cascade can **delete** child rows you didn’t intend (`onDelete: 'CASCADE'` at DB level vs ORM cascade).
- - Prefer explicit service logic for **critical** deletes unless the domain is clearly parent-owned children only.

- How do you index columns in TypeORM?

- - Use **`@Index()`** on entity class or **`@Column({ unique: true })`** / **`@Index(['colA', 'colB'])`** for composite indexes.
- - Matches DB indexes — helps TypeORM-generated queries that filter/sort on those columns.
- - Still verify with **`EXPLAIN`** — ORM doesn’t remove the need for DB tuning.

- What is the difference between `save()` and `insert()` / `update()`?

- - **`save(entity)`** — **UPSERT-like** behavior for persisted entities; can cascade; runs **SELECT** first if needed to merge state.
- - **`insert()` / `update()`** — bulk-oriented, **no** entity lifecycle hooks in the same way; good for **batch** writes when you don’t need loaded entities.
- - For “create if not exists” logic, often **`upsert`** (where supported) or explicit **find + save**.

- Any pitfalls you watch for in TypeORM?

- - **Select n+1** and **over-fetching** relations — always check query logs in dev.
- - **Decimal/money** — use `decimal`/`numeric` column types, not JS float for money fields.
- - **Timezone** — `timestamp with time zone` in PostgreSQL; know how TypeORM serializes `Date`.
- - **Connection pool** settings (`extra` in DataSource) for production load.

- How do you implement **soft deletes** in TypeORM?

- - Use **`@DeleteDateColumn()`** — `softRemove` / `softDelete` sets `deletedAt` instead of removing the row; **`restore`** brings it back.
- - **`find`** options include `withDeleted: true` when admins need to see archived rows.
- - Add **partial indexes** in DB if you filter `WHERE deletedAt IS NULL` on large tables.

- What are **Entity Subscribers** / lifecycle hooks?

- - **`@EventSubscriber()`** + `EntitySubscriberInterface` — listen to **insert/update/remove** before/after events for cross-cutting logic (audit fields, side effects).
- - Prefer **keeping domain rules in services**; use subscribers for technical concerns (timestamps, logging).

- How do you map **enum** and **JSON** columns?

- - **`@Column({ type: 'enum', enum: Role })`** — stored as DB enum or string depending on driver; keep enum values in sync with migrations.
- - **`@Column({ type: 'jsonb' })`** (PostgreSQL) — flexible schema for metadata; validate at app layer with **class-validator** / Zod.

- How do you use **TypeORM with multiple databases** or **read replicas**?

- - Register **multiple connections** in NestJS (`TypeOrmModule.forRoot` + `forRoot` name) and inject the right `DataSource` / `getRepository` per connection.
- - **Read replicas** — point read-only repositories to replica URL; writes to primary; handle **replication lag** for read-your-writes if needed.

- What is **`@Column({ transformer })`** used for?

- - **Transformers** map DB value ↔ JS value (e.g. encrypt at rest, **ValueTransformer** for bigint ↔ string, masking).
- - Useful when DB type doesn’t match JS cleanly (e.g. `decimal` as string for money).

- How do you **log and debug** slow queries in TypeORM?

- - Enable **`logging: true`** or **`['query', 'error', 'warn']`** in DataSource; use **`maxQueryExecutionTime`** to log slow queries.
- - In production, ship logs to **Datadog/CloudWatch** and correlate with **APM**; pair with DB **`EXPLAIN`**.

- What are **polymorphic associations** in TypeORM?

- - TypeORM has **no first-class polymorphic relation** like some ORMs — usually model with **discriminator column** (`commentableType` + `commentableId`) or separate tables.
- - Document the pattern in your team so queries stay predictable.

- How do you run **raw SQL** safely in TypeORM?

- - Use **`dataSource.query(sql, parameters)`** or **`QueryBuilder`** with **bound parameters** — never string-concatenate user input (SQL injection).
- - Prefer ORM/query builder for most CRUD; raw SQL for **reports**, **migrations**, or **DB-specific** features (CTEs, window functions) when needed.

## MongoDB

- How comfortable are you with MongoDB?

- - I use MongoDB for **document-oriented** data: flexible schemas, aggregation pipelines, and indexing for read-heavy workloads alongside SQL where relationships are strict.

- What is MongoDB? How does it differ from a relational (SQL) database?

- - **MongoDB** is a **document** database — data stored as **BSON** documents in **collections** (not rows in normalized tables).
- - **Schema flexibility** — nested objects and arrays in one document; good when shape evolves or read patterns are document-centric.
- - **Trade-offs:** joins are possible (`$lookup`) but relational integrity and complex multi-table constraints are **not** the same as SQL FKs — design matters.

- What are **BSON** and **JSON** in MongoDB?

- - **JSON** — human-readable format MongoDB APIs often expose.
- - **BSON** — **binary** JSON-like encoding stored on disk; extra types (**ObjectId**, **Date**, **Binary**, **Decimal128**) and efficient traversal.

- What is `_id` and **ObjectId**?

- - Every document has **`_id`** — unique primary key in its collection; if omitted, MongoDB creates an **`ObjectId`** (12 bytes: timestamp + randomness).
- - You can use **UUID strings** or custom `_id` if your app generates them — still must be unique.

- What is the difference between **embedding** vs **referencing**?

- - **Embed** — store related data **inside** the document (e.g. address inside user) — **one read**, atomic updates within one doc; document size limit (**16MB** per doc).
- - **Reference** — store **ObjectId** to another collection — **normalizes** data, avoids duplication; requires **second query** or **`$lookup`**.

- What is the **aggregation pipeline**? Name common stages.

- - Pipeline = **ordered stages**; each stage transforms documents (`$match` → `$group` → `$sort` …).
- - Common stages: **`$match`** (filter), **`$group`** (aggregate), **`$project`** (shape), **`$lookup`** (join), **`$sort`**, **`$limit`**, **`$skip`**, **`$unwind`** (arrays).

- What is **`$lookup`**? How does it relate to SQL JOIN?

- - **`$lookup`** joins documents from another collection into an array field (left outer join style by default).
- - Performance depends on **indexes** on foreign fields; heavy pipelines may need **denormalization** or **precomputed** fields.

- What **indexes** exist in MongoDB?

- - **Single-field**, **compound** (order matters for prefix queries), **multikey** (arrays), **text** (search), **2d / 2dsphere** (geo), **TTL** (expiry).
- - Use **`explain("executionStats")`** to verify **IXSCAN** vs **COLLSCAN**.

- What is a **replica set**?

- - **Replica set** — one **primary** (writes) and **secondaries** (async replication) for **high availability** and read scaling.
- - **Election** if primary fails; **write concern** controls how many nodes must ack a write.

- What is **sharding**? When would you shard?

- - **Sharding** splits data across **shards** (horizontal scaling) using a **shard key**.
- - Use when a single replica set **can’t hold** data or write throughput — operational complexity increases; choose shard key carefully to avoid **hot shards**.

- Does MongoDB support **multi-document ACID transactions**?

- - **Yes** (from MongoDB 4.0+ for replica sets, 4.2+ sharded) — **`startSession`**, **`withTransaction`** for multi-doc atomicity.
- - Prefer **single-document** atomicity when possible — transactions have overhead vs one-document updates.

- What are **write concern** and **read preference**?

- - **Write concern** — how many nodes must acknowledge a write (`w: 'majority'`, `journal: true`).
- - **Read preference** — which nodes to read from (`primary`, `secondary`, `nearest`) — useful for scaling reads; watch **staleness**.

- What is **GridFS**?

- - **GridFS** splits files **larger than 16MB** into chunks in **`fs.files`** and **`fs.chunks`** collections — stream large uploads/downloads.
- - Alternative: store files in **S3** and keep only metadata + URL in MongoDB.

- What are **Change Streams**?

- - **Change Streams** — **real-time** notifications when data changes (watch collection or DB) — useful for **sync**, **notifications**, **event-driven** flows.
- - Requires **replica set**; respects **resume tokens** for reliability.

- What is a **TTL index**?

- - **TTL index** on a date field — MongoDB **auto-deletes** documents after expiration — great for **sessions**, **OTP**, **temp logs**.

- How do you **optimize** a slow MongoDB query?

- - **`explain`** — avoid **COLLSCAN**; add **matching indexes**; reduce **working set** (`$match` early in pipeline).
- - Limit **`$lookup`** fan-out; avoid **unbounded** `skip` for pagination — prefer **range/cursor** on `_id` or indexed field.

- When would you **choose MongoDB vs PostgreSQL**?

- - **MongoDB** — flexible/evolving schema, document aggregates, horizontal scale, rapid iteration.
- - **PostgreSQL** — strong **relational** model, complex joins, constraints, mature SQL analytics — pick per **access patterns** and **consistency** needs.

- What is **schema validation** on a collection?

- - **`validator`** on `createCollection` or **`$jsonSchema`** rules — enforce required fields, types, ranges **at insert/update** (server-side), even if the app misbehaves.
- - Complements **Mongoose** validation — defense in depth for data quality.

- What is the **oplog** (operations log)?

- - **Oplog** — capped collection of **replicated** write operations on the **primary**; secondaries **tail** it to stay in sync.
- - Also powers **Change Streams** (with resume tokens) and some **migration** tools.

- What are **atomic update operators** (`$set`, `$inc`, `$push`, etc.)?

- - Updates can modify fields **without** reading the whole document into the app first — **`$inc`** for counters, **`$push`** / **`$pull`** for arrays, **`$set`** for partial updates.
- - Single-document updates are **atomic** — use for **inventory**, **votes**, **counters** to avoid lost updates.

## Mongoose

- What is **Mongoose**? Why use it with Node.js?

- - **Mongoose** is an **ODM** (Object Document Mapper) for MongoDB in **Node.js** — **schemas**, **validation**, **middleware**, **population**.
- - Adds structure on top of flexible MongoDB — great for **team consistency** and **NestJS** integration (`@nestjs/mongoose`).

- What is the difference between **Schema**, **Model**, and **Document**?

- - **Schema** — defines shape, types, validators, defaults, indexes at the **application** layer.
- - **Model** — constructor bound to a collection — **`create`**, **`find`**, **`aggregate`**.
- - **Document** — one **instance** of a model (has `.save()`, getters/setters, methods).

- How does **validation** work in Mongoose?

- - **Built-in validators** — `required`, `min`/`max`, `enum`, `match` (regex).
- - **Custom** validators and **`validate` async** hooks — reject bad data **before** DB write; combine with **class-validator** at API layer for defense in depth.

- What are **pre** / **post** middleware hooks?

- - **`pre('save')`**, **`post('findOneAndUpdate')`**, etc. — run logic around operations (hash password, denormalize fields, audit).
- - Order matters; **avoid heavy work** in hooks without understanding **bulk** operations (`updateMany` may **not** trigger `save` hooks).

- What is **`populate()`**? How is it different from aggregation **`$lookup`**?

- - **`populate`** replaces refs with documents from other collections (convenient, similar to JOIN for refs).
- - **`aggregate` + `$lookup`** — more control for **filters**, **pipelines**, **multiple** joins; use when populate is **not enough** or performance tuning needs pipeline **`$match` first**.

- Why use **`.lean()`**?

- - **`.lean()`** returns **plain JS objects** (not full Mongoose documents) — **faster**, less memory, no getters/virtuals unless `virtuals: true`.
- - Use for **read-only** API responses and lists when you don’t need `save()` or document methods.

- What are **virtuals**?

- - **Virtuals** — computed properties not stored in DB (`fullName` from `firstName` + `lastName`) — **`get`/`set`**.
- - **`virtual('field').get(...)`** — good for API shaping; **`toJSON: { virtuals: true }`** to expose in responses.

- How do you define **indexes** in Mongoose?

- - **`schema.index({ email: 1 }, { unique: true })`** — compound / unique / TTL same as native driver concepts.
- - **`autoIndex`** in production — often **disabled**; create indexes via **migrations/scripts** for control.

- What are **subdocuments** vs separate **collections**?

- - **Subdocuments** — nested schemas in one doc; atomic updates within parent; good for **owned** small arrays (line items).
- - **Separate collection** + ref — when subdocs grow large or need **independent** queries/indexes.

- How do you run **transactions** in Mongoose?

- - **`session.startTransaction()`** … **`commitTransaction` / `abortTransaction`** with **`session`** passed to operations (same as MongoDB driver).
- - Keep transactions **short**; design so many flows use **single-document** updates when possible.

- What is the **discriminator** pattern?

- - **Discriminators** — multiple **models** share one collection with a **discriminator key** (`type: 'Car' | 'Bike'`) — polymorphic documents with different schemas.
- - Useful for **similar** entities with **type-specific** fields.

- What is **`strict` mode** and **`__v`** (version key)?

- - **`strict: true`** (default) — strips keys not in schema; **`false`** allows arbitrary fields (use carefully).
- - **`__v`** — **optimistic concurrency** for document versioning; increments on updates — helps detect **conflicting** writes.

- How do you use **Mongoose with TypeScript**?

- - Define **interfaces** + **`Schema<Type>`** / **HydratedDocument** patterns; community patterns: **typegoose** or manual typing.
- - Type **`lean()`** results explicitly — lean returns **plain objects** not `Document` types.

- How do you handle **errors** (`CastError`, `ValidationError`)?

- - **`CastError`** — bad **ObjectId** format or wrong type in query — return **400** to client.
- - **`ValidationError`** — schema validation failed — map **`errors`** field to user-friendly messages in NestJS filter.

- What are **cursors** / streaming for large result sets?

- - **`find().cursor()`** — stream documents **one by one** — avoids loading millions into memory.
- - Use for **exports**, **bulk processing**, **ETL** jobs.

- Any **NestJS + Mongoose** tips?

- - Register **`MongooseModule.forFeature`** per module; inject **`@InjectModel(Name)`** in services.
- - **Global filters** for Mongo errors; **connection** options: **pool size**, **retryWrites**, **readPreference** for replicas.

- How do you use **`select()` / projection**?

- - **`User.find().select('name email')`** — include only listed fields; **`select('-password')`** excludes sensitive fields from API responses.
- - Combine with **`lean()`** for fast list endpoints; never return **hashed passwords** or secrets by mistake.

- What is the difference between **`create()`** / **`insertMany()`** and **`save()`**?

- - **`create()`** — one or more docs; runs **validation** and **`save` middleware** per doc.
- - **`insertMany()`** — bulk insert; **`ordered: false`** continues on error; **may skip** some `save` hooks — check docs for your Mongoose version.
- - **`save()`** on a document instance — full lifecycle for that doc.

- What are **Mongoose plugins**?

- - **Plugins** extend **Schema** behavior once (`schema.plugin(myPlugin)` — e.g. pagination, soft delete, common timestamps).
- - Keeps cross-cutting logic **DRY** across models.

- How do you run **aggregation** from Mongoose?

- - **`Model.aggregate([{ $match: ... }, { $group: ... }])`** — same pipeline as the shell/driver; returns **plain objects** (not documents).
- - Use for **reports**, **joins** via `$lookup`, **analytics** — often faster than multiple `populate` calls for heavy read paths.

## Messaging, queues & observability

- What is messaging queue ?

- - A messaging queue lets one service send a message and another service process it later, without both needing to be active at the same time.
- - Producer → Queue → Consumer - producer messeges to the queue - queue stores the message temporarily - producer Processes messages
- - Exampels :- rabbitMq, Kafka,Amazon SQS,BullMq.
- -

- How do you manage messaging between services?

- - NestJS provides microservice transport strategies by using - TCP , RabitMq, Kafka, and Redis.
- - https://dev.to/pulkit5ingh/nestjs-microservices-with-rabbitmq-retries-dlq-production-setup-that-actually-scales-55eh

- What do you know about rabitMq, kafka, AWSSQS, bullMq

- - https://dev.to/pulkit5ingh/kafka-vs-rabbitmq-vs-sqs-vs-bullmq-stop-guessing-choose-the-right-one-2026-guide-1cp5

- What is Amazon SQS?
- How do you enable notifications using SQS?
- When would you use SQS versus RabbitMQ?
- In what scenarios did you use SQS?
- In what scenarios did you use SNS for notifications?
- How do you trace or track requests across the system?
- What tools are used for monitoring?
- How do you handle alerting and monitoring in a production environment?
- Can you explain Kafka architecture (producer, topic, consumer)?
- What happens when producers are faster than consumers in Kafka?

- What is a dead letter queue?

- - A Dead Letter Queue (DLQ) is a special queue where failed or unprocessed messages are sent instead of being lost.
- - A DLQ stores messages that couldn’t be processed successfully after retries, so you can inspect and fix them later.

- Why do we need dead letter queue?

- - With DLW - Failed messages are stored, Can debug later, Prevent infinite retry loops, System becomes reliable.
- - without DLQ - Message fails, Gets lost, No visibility, Data inconsistency.

## Cloud, containers & Kubernetes

- What cloud providers do you know?

- - AWS, Google Cloud & Digital Ocean.

- What is containerization? Why we need it ? pros & cons ?

- - Containerization is a way to package an application along with all its dependencies (code, runtime, libraries, configs) into a single lightweight unit called a container.
- - Containerization packages your application with its dependencies into portable, isolated units that run consistently across environments.
- - Portibility - Run anywhere (local, cloud, VPS)
- - Consistency - Same environment across dev → prod
- - Isolation - Each app runs independently
- - Scalability - Easily scale containers (Kubernetes)
- - Faster Deployment - Start in seconds (lighter than VMs)
- - Microservices Friendly - Each service in its own container

- what is docker why we use docker?

- - Code → Dockerfile → Docker Image → Docker Container
- - create a Dockerfile & .dockerignore file
- - Dockerfile tells docker how to build your app
- - .dockerignore - avoid sending unnecessary file to docker
- - build docker image - `docker build -t my-node-app .` Creates an image (snapshot of your app)
- - run container `docker run -p 3000:3000 my-node-app` Your app is now running inside a container

- What is docker compose ? why do we use docker compose ? why do we use it ?

- -

- What is Kubernetes and why is it used?

- - Kubernetes (K8s) is an open-source system used to manage, scale, and run containerized applications automatically.
- - Kubernetes is a platform that automates deployment, scaling, and maintenance of containerized apps.
- - Kubernetes solves the problem of - auto scalling, self healing, load balancing, zero down time, Service Discovery
- - Developer → Docker Image → Kubernetes Deployment → Pods → Users

- What is a Pod in Kubernetes?
- Which AWS services have you worked with?

## Microservices communication & APIs

- How do services communicate with each other in a microservices architecture?
- Can you show how one service calls another service?
- Can you write code to fetch a user’s geolocation (latitude and longitude)?
- What kind of optimization or performance improvements have you implemented?
- How do you consume a third-party API that returns a very large response?
- What considerations are needed when handling large JSON responses?
- How would you handle JWT expiry and rate limiting when consuming third-party APIs?
- How would you ensure no data loss while fetching large volumes of data?
- How would you paginate or chunk large API responses?
- How would you limit API calls to comply with rate limits?
- How do you implement delays or throttling in Node.js?
- How would you make a POST API call with a given JSON payload?
- How would you write a production-ready POST API in Node.js?
- Which framework would you use to build APIs in Node.js?
- How do you structure APIs using controllers and routers?
- What security practices do you follow while developing Node.js applications?
- How would you implement a secured GET API using JWT?
- Which logging mechanism do you use?
- What caching mechanisms have you used?
- Can you write code for a GET API that returns user details and related data?
- Have you heard about Express Router? Why do we use it?

## JavaScript & TypeScript

- Can you write a function to count character occurrences?
- How do you handle errors in asynchronous Node.js code?
- What is the difference between `process.nextTick` and `setImmediate`?
- What are `call`, `apply`, and `bind` in JavaScript?
- What is a closure? Can you give a real-time use case?
- What is the difference between `interface` and `type` in TypeScript?

## Frontend (HTML, CSS, React)

- Have you worked with HTML and CSS?
- What is the difference between `<div>` and `<section>`?
- What is the difference between `id` and `class`?
- What are variables in SASS?
- Have you worked with mixins?
- Have you optimized CSS performance? If yes, how?
- What does the `navigate` method do in React?
- How do you pass data while navigating between pages in React?

## Testing, resilience & deployment

- Do you write test cases for your services?
- Have you worked with circuit breakers?
- How do you mock a service while writing tests?
- How do you deploy a Node.js application to Azure?

```

```
