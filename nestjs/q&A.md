# NestJS — Questions & Answers

Questions and answers drawn from every file in this folder, in curriculum order. Each section links back to its source file.

- [Phase 1 — NestJS Fundamentals](#phase-1--nestjs-fundamentals)
- [Phase 2 — Database with TypeORM & Prisma](#phase-2--database-with-typeorm--prisma)
- [Phase 3 — Authentication & Guards](#phase-3--authentication--guards)
- [Phase 4 — Pipes, Interceptors & Filters](#phase-4--pipes-interceptors--filters)
- [Phase 5 — Configuration & Environment](#phase-5--configuration--environment)
- [Phase 6 — Caching, Queues & Performance](#phase-6--caching-queues--performance)
- [Phase 7 — Testing](#phase-7--testing)
- [Phase 8 — Microservices](#phase-8--microservices)
- [Phase 9 — Advanced & Production](#phase-9--advanced--production)
- [Study Strategy & Debugging](#study-strategy--debugging)

---

## Phase 1 — NestJS Fundamentals

### [A1.1 — NestJS Architecture](A1.1-nestjs-architecture.md)

**Q: What is NestJS built on?**
A: Express (or Fastify), adding an opinionated, Angular-inspired structure centred on the module system and a dependency injection (DI) container.

**Q: What happens at startup?**
A: NestJS reads `@Module()` metadata from the root `AppModule` and feature modules, builds a directed acyclic graph of providers, resolves constructor dependencies, and instantiates providers (usually singletons) before handling requests.

**Q: How do NestJS decorators work?**
A: They call `Reflect.defineMetadata` (via `reflect-metadata`). `@Get('/hello')` stores the path and method as metadata, which NestJS reads at bootstrap to register Express routes.

**Q: What are the three building blocks?**
A: **Module** — a feature slice declaring `providers`, `controllers`, `exports` and `imports`. **Controller** — receives requests and delegates to services, with no business logic. **Provider** — any `@Injectable()` class (service, repository, guard, pipe, interceptor) whose lifetime the DI container manages.

### [A1.2 — Controllers & Routing](A1.2-controllers-and-routing.md)

**Q: What is a controller responsible for?**
A: Accepting input and returning output; business logic belongs in services. HTTP methods map to `@Get()`, `@Post()`, `@Put()`, `@Patch()`, `@Delete()`, `@Options()`, `@Head()` and `@All()`.

**Q: `@Param` vs `@Query` vs `@Body`?**
A: `@Param` reads dynamic path segments (`/users/:id`), `@Query` reads `?key=value` pairs, and `@Body` reads the parsed payload.

**Q: Library-agnostic vs library-specific responses?**
A: Agnostic (recommended): return a value and NestJS serialises it (200 for GET, 201 for POST). Specific: inject `@Res()` and call `res.json()`. Prefer agnostic — it works with interceptors and is testable without mocking Express.

**Q: How do you change status codes and headers?**
A: `@HttpCode(status)` and `@Header(name, value)` for static values; `@Res({ passthrough: true })` for dynamic headers while staying library-agnostic.

### [A1.3 — Providers & Services](A1.3-providers-and-services.md)

**Q: How does the DI container identify providers?**
A: By a token — the class by default, or a string or `Symbol`. Providers are created lazily on first injection and cached by scope.

**Q: What are the three provider scopes?**
A: `DEFAULT` (singleton, the 99% case), `REQUEST` (one instance per HTTP request, e.g. for current-user data), and `TRANSIENT` (a new instance per injection, rare).

**Q: What are the custom provider forms?**
A: `useClass` (swap implementations, e.g. by environment), `useValue` (inject a pre-built object such as config or test mocks), and `useFactory` (a possibly async factory that can receive injected dependencies).

### [A1.4 — Modules](A1.4-modules.md)

**Q: What is a feature module?**
A: A module owning one domain that imports what it needs and exports only what others need — the default from `nest g module`.

**Q: What is a shared module, and does importing it twice create two instances?**
A: A module that exports providers for others to import. No — modules are singletons, so providers are shared (e.g. a `DatabaseModule` exporting a `DataSource`).

**Q: When should you use a global module?**
A: Sparingly, for config or logging. `@Global()` exports are available everywhere without imports, but overuse creates hidden dependencies.

**Q: What is a dynamic module?**
A: One whose providers/imports/exports are computed from options at runtime — `forRoot(options)` for sync and `forRootAsync({ useFactory })` for async config (e.g. from `ConfigService`). `TypeOrmModule`, `JwtModule` and `BullModule` all use this pattern.

### [P1.1 — Tasks API](P1.1-tasks-api.md)

**Q: What does it build?**
A: A Tasks REST API — create, list (filter by status/priority), get, partial update, delete — with title, description, status, priority and due date.

**Q: How is data stored?**
A: In memory, in a service holding a `Map<string, Task>`.

**Q: How are requests validated?**
A: With DTOs using class-validator, and typed responses with a consistent shape.

### [P1.2 — Library Management API](P1.2-library-management-api.md)

**Q: Which resources does it include?**
A: Books (search by title/genre, linked to author), authors (with their books), members (active/inactive), and borrow records (borrow, return, history), using in-memory repositories.

**Q: What does it teach about modules?**
A: Multi-feature organisation and cross-module DI: a `SharedModule` exports a `DateHelperService` used by several features, and a `LibraryModule` groups the feature modules.

---

## Phase 2 — Database with TypeORM & Prisma

### [A2.1 — TypeORM Basics](A2.1-typeorm-basics.md)

**Q: What does TypeORM provide?**
A: Mapping of TypeScript entity classes to tables via decorators, connection pooling, and a `Repository<T>` per entity with `find`, `findOne`, `save`, `update`, `delete` and a query builder.

**Q: How is TypeORM wired into NestJS?**
A: `TypeOrmModule.forRoot()` registers the connection; `TypeOrmModule.forFeature([Entity])` plus `@InjectRepository(Entity)` injects repositories into services.

**Q: What are the key entity decorators?**
A: `@Entity()` (table name defaults to lowercase class name), `@PrimaryGeneratedColumn()` (or `'uuid'`), `@Column()` with options, `@CreateDateColumn()`/`@UpdateDateColumn()`, and `@DeleteDateColumn()` for soft deletes.

**Q: What are find options?**
A: Structured filters like `{ where, order, skip, take }`; use `QueryBuilder` for complex queries.

### [A2.2 — TypeORM Relations](A2.2-typeorm-relations.md)

**Q: Where does the foreign key live in ManyToOne/OneToMany?**
A: On the ManyToOne side — e.g. the Post table for Post → User.

**Q: How do ManyToMany and OneToOne work?**
A: ManyToMany needs a join table, created with `@JoinTable()` on one side (Articles ↔ Tags). OneToOne puts the FK on the side with `@JoinColumn()` (User ↔ UserProfile).

**Q: What does `cascade` do and why be careful?**
A: It persists related entities when the parent is saved, but can trigger unexpected bulk operations.

**Q: Eager vs lazy vs explicit loading?**
A: `eager: true` always loads the relation; lazy relations return a `Promise` to await. The recommended approach is explicit loading with the `relations` option or `.leftJoinAndSelect()`, controlling what each query loads.

### [A2.3 — Database Migrations](A2.3-database-migrations.md)

**Q: Why is `synchronize: true` dangerous in production?**
A: It alters the schema on every startup, can drop columns and lose data, and bypasses review.

**Q: What is the migration workflow?**
A: Change entities → `migration:generate` (diff of DB vs entities) → review the SQL → `migration:run` (tracked in a `migrations` table) → `migration:revert` if needed.

**Q: How are seeders done in TypeORM 0.3+?**
A: They aren't built in — write a standalone script or NestJS command using the same `DataSource`. Seeders must be idempotent.

### [A2.4 — Prisma in NestJS](A2.4-prisma-in-nestjs.md)

**Q: How does Prisma differ from TypeORM?**
A: You define models in `schema.prisma` and `prisma generate` produces a typed `PrismaClient`; types come from `@prisma/client` rather than hand-written classes.

**Q: What is `PrismaService`?**
A: A NestJS service extending `PrismaClient` that calls `$connect()` on module init and `$disconnect()` on destroy.

**Q: What are nested writes?**
A: Creating or linking related records in one query with nested `create`, `connect` or `connectOrCreate`.

**Q: `select` vs `include`?**
A: `select` returns only chosen fields (more efficient); `include` returns the full record plus relations.

**Q: What are the two `$transaction` modes?**
A: Interactive (callback-based, supports rollback) and sequential (an array of promises).

### [P2.1 — Blog Platform with TypeORM](P2.1-blog-platform-with-typeorm.md)

**Q: Which entities and relations does it model?**
A: Users, posts (publish/draft) belonging to users, categories, tags (ManyToMany), and comments nested under posts and users.

**Q: How are schema changes and complex queries handled?**
A: Migrations for every change, and `QueryBuilder` for filtered queries (posts by tag + category + search).

**Q: How is dev data populated?**
A: With a seeder command.

### [P2.2 — Product Catalog with Prisma](P2.2-product-catalog-with-prisma.md)

**Q: What does the data model include?**
A: Hierarchical categories, products, variants (SKU, price, colour/size attributes), and per-variant inventory.

**Q: How is stock reservation kept correct?**
A: Buying decrements inventory atomically inside a Prisma transaction, with reserve/release operations.

**Q: How is Prisma integrated?**
A: A typed `PrismaService` extending `PrismaClient`, plus Prisma migrations and a seeder.

---

## Phase 3 — Authentication & Guards

### [A3.1 — Guards](A3.1-guards.md)

**Q: What is a guard and when does it run?**
A: A `CanActivate` class deciding whether a request proceeds. Guards run after middleware and before interceptors and pipes.

**Q: What does `ExecutionContext` add?**
A: It extends `ArgumentsHost` with `getClass()` and `getHandler()`, needed to read decorator metadata, and works across HTTP, WebSocket, gRPC and microservice transports.

**Q: What is the idiomatic way to read metadata?**
A: `reflector.getAllAndOverride(key, [context.getHandler(), context.getClass()])`, letting handler metadata override class metadata.

**Q: What do guards return?**
A: `true` to allow, `false` to deny (403 by default), or they throw for custom codes (401 for missing tokens vs 403 for insufficient permissions).

**Q: At which scopes can guards apply?**
A: Route (`@UseGuards` on a method), controller (on the class), or global — preferably via an `APP_GUARD` provider so it supports DI.

### [A3.2 — JWT Authentication](A3.2-jwt-authentication.md)

**Q: How does `@nestjs/passport` work?**
A: Extend `PassportStrategy(Strategy)` and implement `validate()`; whatever it returns becomes `request.user`.

**Q: How are the JWT strategy and guard built?**
A: A `JwtStrategy` using `passport-jwt` (secret from `ConfigService`) and a `JwtAuthGuard` extending `AuthGuard('jwt')`.

**Q: What is the two-token pattern?**
A: A 15-minute access token sent as `Authorization: Bearer` (user ID and roles), and a 7-day refresh token sent only to `/auth/refresh`.

**Q: How should refresh tokens be secured?**
A: Store them hashed (bcrypt) in the DB, rotate on each use, and revoke all on logout.

### [A3.3 — Roles & Permissions](A3.3-roles-and-permissions.md)

**Q: How is RBAC implemented in NestJS?**
A: A `@Roles()` decorator sets metadata and a `RolesGuard` checks the user's role against it.

**Q: What does CASL add?**
A: Attribute-based access control — abilities like `can('update', 'Post')` only if the user is the author — built by an `AbilityFactory` per user.

**Q: What are the two levels of permission checks?**
A: Guard-level (coarse, before the handler, using roles or action+subject metadata) and service-level (fine-grained, after loading the resource, checking ownership or attributes).

### [A3.4 — OAuth2 & Sessions](A3.4-oauth2-and-sessions.md)

**Q: What is the OAuth2 flow with Passport?**
A: `GET /auth/google` redirects to consent → Google calls `/auth/google/callback?code=…` → Passport exchanges the code → `GoogleStrategy.validate()` returns the user → redirect to the frontend with a JWT or start a session.

**Q: How do sessions differ from JWT here?**
A: The server keeps auth state in a session store (Redis, DB) and sends a cookie; NestJS uses `express-session` with `passport.session()`.

**Q: When are sessions a poor fit?**
A: For API-only backends with multiple instances, which need a shared session store.

### [P3.1 — Complete Auth System](P3.1-complete-auth-system.md)

**Q: Which endpoints does it provide?**
A: Register (with verification email), login (15-min access + 7-day refresh), logout, refresh (rotation), verify-email, forgot-password and reset-password.

**Q: How are routes and users accessed?**
A: A `JwtAuthGuard` protects routes and a `CurrentUser` decorator extracts the user from the JWT payload.

**Q: How are refresh tokens revocable?**
A: They're stored in the DB.

### [P3.2 — Permission System (RBAC + ABAC)](P3.2-permission-system-rbac-abac.md)

**Q: How are roles and permissions modelled?**
A: `Role` (ADMIN, EDITOR, VIEWER), `Permission` (action + resource, e.g. `posts:update`) and a `RolePermission` join table, seeded with defaults.

**Q: Which decorators and guards are used?**
A: `@Roles()` + `RolesGuard` (coarse), `@RequirePermission()` (specific), `OwnershipGuard` (ABAC ownership), and `@Public()` to bypass auth.

**Q: Do admins need to pass ownership checks?**
A: No — admins bypass them. An admin endpoint assigns roles and manages role permissions.

---

## Phase 4 — Pipes, Interceptors & Filters

### [A4.1 — Pipes](A4.1-pipes.md)

**Q: What do pipes do?**
A: Sit between argument extraction and the handler to validate (throw on bad input) and transform (convert types), via `transform(value, metadata)`.

**Q: What are the key `ValidationPipe` options?**
A: `whitelist` strips unknown properties, `forbidNonWhitelisted` rejects them, `transform` converts to DTO instances and primitives (`"1"` → `1`), and `enableImplicitConversion` coerces types without `@Type()`.

**Q: What are `ParseIntPipe` and `ParseUUIDPipe` for?**
A: Validating that URL params are valid integers/UUIDs before the handler.

**Q: When are custom pipes useful?**
A: Parsing pagination params, trimming, splitting comma-separated strings, or loading an entity by ID (the param pipe pattern).

### [A4.2 — Interceptors](A4.2-interceptors.md)

**Q: How do interceptors work?**
A: `intercept(context, next)` — code before `next.handle()` runs before the handler; RxJS operators (`map`, `tap`, `catchError`, `timeout`) on the returned Observable run after.

**Q: Why are interceptors the place for response transformation and caching?**
A: Unlike guards and pipes, they see the response — so they suit transformation, timing, caching and tracing headers.

**Q: How does an interceptor short-circuit a handler?**
A: By returning an Observable without calling `next.handle()` — e.g. a cached value. `handle()` can only be called once.

### [A4.3 — Exception Filters](A4.3-exception-filters.md)

**Q: What does NestJS do with exceptions by default?**
A: Converts `HttpException` subclasses to matching responses and anything else to 500.

**Q: How do you write an exception filter?**
A: Implement `ExceptionFilter`, decorate with `@Catch(Type)` (or `@Catch()` for everything), and use `catch(exception, host)` with `ArgumentsHost` to build the response.

**Q: How should custom exceptions be designed?**
A: Extend `HttpException` with a status code, optionally carrying structured data like error codes and details. RFC 7807 Problem Details is the standard response format.

### [A4.4 — class-validator & class-transformer](A4.4-class-validator-and-class-transformer.md)

**Q: How does class-validator work?**
A: Decorators attach metadata to properties; `validate(instance)` (or `ValidationPipe`) checks them and returns `ValidationError`s with paths and failed constraints.

**Q: Why is class-transformer needed?**
A: Validator decorators only work on class instances. `plainToInstance()` converts plain objects to instances; `instanceToPlain()` converts back.

**Q: How do you hide fields like passwords?**
A: Class-level `@Exclude()` with `@Expose()` on properties to include, so only exposed fields serialise.

**Q: When do you need custom validators?**
A: For domain rules built-ins can't express — e.g. password must not contain the username, or endDate after startDate.

### [P4.1 — Validated API with Transforms](P4.1-validated-api-with-transforms.md)

**Q: How is the global `ValidationPipe` configured?**
A: With `whitelist`, `forbidNonWhitelisted` and `transform`.

**Q: How are responses and errors shaped?**
A: A `TransformInterceptor` wraps success responses in `{ data, meta }`; a `ProblemDetailsFilter` converts all exceptions to RFC 7807 (including class-validator message arrays).

**Q: Which validation techniques are used?**
A: A custom `ParsePositiveIntPipe`, `@Transform()` to trim strings and lowercase emails, `@ValidateNested()` for nested DTOs, and `@ValidateIf()` (e.g. a field only required when status is ACTIVE).

### [P4.2 — Audit Logging Interceptor](P4.2-audit-logging-interceptor.md)

**Q: What does the interceptor capture?**
A: Method, path, status, duration, user ID, IP, and request/response bodies, stored in an `AuditLog` entity.

**Q: How does it avoid slowing responses?**
A: Logging is fire-and-forget — not awaited.

**Q: How are sensitive data and volume handled?**
A: `password`, `token` and `secret` are scrubbed; `@SkipAudit()` excludes health checks and high-volume routes; a scheduled job deletes logs older than 90 days.

**Q: How are logs queried?**
A: `GET /admin/audit-logs` with userId, method, path and date-range filters.

---

## Phase 5 — Configuration & Environment

### [A5.1 — ConfigModule](A5.1-configmodule.md)

**Q: What does `@nestjs/config` do?**
A: Loads `.env` files (via dotenv) and exposes values through `ConfigService`. With a Joi `validationSchema` it fails at startup on missing or mistyped variables.

**Q: What is namespaced config?**
A: Grouping related variables with `registerAs` factories and injecting `ConfigType<typeof factory>` for compile-time type safety.

**Q: What is the config loading order (lowest to highest)?**
A: `.env` file → OS shell environment variables → additional files in `envFilePath`.

### [A5.2 — Database Config Patterns](A5.2-database-config-patterns.md)

**Q: Why use `forRootAsync`?**
A: When config isn't known at import time (it depends on env vars loaded by `ConfigModule`). Its `useFactory` receives injected providers like `ConfigService`.

**Q: Why include `imports` inside `forRootAsync`?**
A: To make `ConfigModule` available when the factory runs.

**Q: How do you use multiple databases with TypeORM?**
A: Give each connection a unique `name` (default `'default'`) and inject with `@InjectRepository(Entity, 'connectionName')`.

### [A5.3 — Secrets Management](A5.3-secrets-management.md)

**Q: What are the risks of env-var secrets?**
A: Committing `.env`, logging secrets, returning them in responses, and not rotating them.

**Q: What are the best practices?**
A: Commit only `.env.example`, validate secrets at startup, redact them from logs, `@Exclude()` sensitive fields, and rotate regularly.

**Q: What does HashiCorp Vault provide?**
A: Encrypted storage, audit logs of every access, and dynamic short-lived credentials. NestJS loads them at startup via the Vault SDK before `ConfigModule` initialises.

### [A5.4 — Feature Flags & App Settings](A5.4-feature-flags-and-app-settings.md)

**Q: Which settings belong in the database rather than env vars?**
A: Ones that change at runtime without a restart — rate limits, maintenance mode, feature flags, email templates.

**Q: What are feature flags?**
A: Toggles that enable features per environment or user segment without deploying — basic version in a DB table, production version via LaunchDarkly or Unleash.

**Q: How is the settings cache invalidated?**
A: An `EventEmitter` on a single node; Redis pub/sub across multiple nodes.

### [P5.1 — Multi-Environment API](P5.1-multi-environment-api.md)

**Q: How are environments configured?**
A: `.env.development`, `.env.staging` and `.env.production`, validated by a Joi schema at startup.

**Q: How is config typed?**
A: Namespaced configs (database, app, auth, redis) behind a `TypedConfigService` — business code never uses string keys.

**Q: How is environment-specific behaviour gated?**
A: An `APP_ENV` variable (e.g. Swagger only in dev/staging), with `TypeOrmModule.forRootAsync` and `JwtModule.registerAsync` reading typed config.

### [P5.2 — Settings Service](P5.2-settings-service.md)

**Q: What is stored per setting?**
A: Key, JSON value, data type, description and last-updated-by, grouped into categories.

**Q: How are caching and change notification handled?**
A: Redis-backed caching invalidated on update (scan-based deletion), and `EventEmitter2` emits `setting.changed` with old and new values.

**Q: What conveniences does it provide?**
A: An `@InjectSetting(key)` decorator, admin endpoints to list/update/reset, and a default-settings upsert seeder on startup.

---

## Phase 6 — Caching, Queues & Performance

### [A6.1 — Caching](A6.1-caching.md)

**Q: What are the two levels of caching?**
A: Route caching with `CacheInterceptor` for GET responses (`@CacheKey`, `@CacheTTL`), and programmatic caching by injecting `CACHE_MANAGER` and calling `get`, `set` and `del`.

**Q: What are the invalidation strategies?**
A: TTL-based (simple, may be stale), event-based (delete on write), and tags/patterns (group keys by prefix and invalidate together, e.g. with Redis SCAN).

### [A6.2 — Bull Queues](A6.2-bull-queues.md)

**Q: Why use Bull?**
A: Jobs persist in Redis, survive restarts, and can be processed by many workers horizontally.

**Q: What are the key concepts?**
A: Queues (`queue.add(name, data, options)`), processors (`@Processor` with `@Process` methods), job options (`attempts`, `backoff`, `priority`, `delay`, `removeOnComplete`/`removeOnFail`), `job.progress()`, lifecycle events (`@OnQueueActive`/`Completed`/`Failed`) and concurrency (`@Process({ concurrency: 5 })`).

### [A6.3 — Task Scheduling](A6.3-task-scheduling.md)

**Q: What scheduling decorators exist?**
A: `@Cron(expression)`, `@Interval(ms)` and `@Timeout(ms)`, using the six-field cron format (seconds first) and the `CronExpression` enum.

**Q: What goes wrong with cron in a scaled deployment?**
A: Every instance runs the job, so it executes N times.

**Q: How do you fix it?**
A: A distributed lock (e.g. `redlock`), or a trigger cron that adds a Bull job so it's processed exactly once.

**Q: What is `SchedulerRegistry` for?**
A: Adding, deleting and stopping jobs at runtime — e.g. user-configured reminders.

### [A6.4 — Rate Limiting](A6.4-rate-limiting.md)

**Q: How does `@nestjs/throttler` work?**
A: `ThrottlerGuard` counts requests per client (IP by default) within a TTL window and returns 429 past the limit.

**Q: How do you rate-limit across multiple windows?**
A: Define multiple named throttlers (e.g. `short` and `long`) and stack them.

**Q: How do you limit by API key or user?**
A: Override `generateKey`.

**Q: What should production use?**
A: A Redis-backed store so counters survive restarts and are shared between instances.

### [P6.1 — Email Notification System](P6.1-email-notification-system.md)

**Q: How are emails sent?**
A: Bull jobs (welcome, reset, order confirmation) rendered with Handlebars and sent by an `EmailProcessor` via Nodemailer/SendGrid.

**Q: What is the retry and failure strategy?**
A: 3 attempts with exponential backoff (1 s, 2 s, 4 s); exhausted jobs go to an `email-failed` Dead Letter Queue.

**Q: How do callers send emails?**
A: Through a `NotificationService` facade — they never touch the queue. Every attempt is recorded in `EmailLog`, and `bull-board` monitors queues.

### [P6.2 — Cache-Heavy Product API](P6.2-cache-heavy-product-api.md)

**Q: How is caching done?**
A: `@CacheKey`/`@CacheTTL` on reads (Redis, 5 min) with keys like `products:all`, `products:{id}`, `products:category:{id}`, invalidated on update/delete, and warmed for top-N products on startup.

**Q: How does per-key rate limiting work?**
A: An `ApiKeyGuard` reads `X-API-Key`; standard keys get 100 req/min and premium 1000, enforced by `ThrottlerGuard` with Redis storage.

---

## Phase 7 — Testing

### [A7.1 — Unit Testing](A7.1-unit-testing.md)

**Q: What does `Test.createTestingModule` do?**
A: Builds a sandboxed module with the real class under test and mocks for its dependencies, so tests run in milliseconds without DB or HTTP.

**Q: `jest.fn()` vs `jest.spyOn()`?**
A: `jest.fn()` creates a tracking mock; `jest.spyOn()` replaces a method on an existing object. Both support `mockResolvedValue`, `mockRejectedValue`, `mockReturnValue` and `mockImplementation`.

**Q: What pattern should unit tests follow?**
A: AAA — Arrange, Act, Assert.

### [A7.2 — Controller Testing](A7.2-controller-testing.md)

**Q: What do controller tests cover?**
A: Routing, pipes, guard application and response formatting, using a real NestJS app with mocked services and supertest — no real database.

**Q: How do you handle `JwtAuthGuard` in tests?**
A: Override it with a guard that returns `true` or sets `request.user` to a fixture.

### [A7.3 — E2E Testing](A7.3-e2e-testing.md)

**Q: What do e2e tests catch?**
A: Integration problems unit tests miss — missing imports, wrong module wiring, broken migrations, ORM bugs.

**Q: How should the test database be managed?**
A: A separate test DB (SQLite in memory or PostgreSQL in Docker): run migrations first, seed in `beforeEach`, truncate in `afterEach`.

**Q: What are the key patterns?**
A: `createTestingModule` + `createNestApplication`, `moduleRef.get(Service)` for setup, `TRUNCATE` or `synchronize({ dropTables: true })` for cleanup, and generated JWTs for protected routes.

### [A7.4 — Test Utilities](A7.4-test-utilities.md)

**Q: Why build test utilities?**
A: Factories, setup helpers and seed helpers remove repeated `beforeEach` code, making tests shorter and easier to maintain.

**Q: Why use `@faker-js/faker`?**
A: Realistic data avoids dummy values like "test@test.com" that can hide bugs.

**Q: What are coverage thresholds for?**
A: `coverageThreshold` makes CI fail when coverage drops, preventing regressions.

### [P7.1 — Fully Tested NestJS API](P7.1-fully-tested-nestjs-api.md)

**Q: What does the test suite cover?**
A: Unit tests for every `TasksService` branch and error path, controller tests with a mocked service, and e2e tests with supertest against a test PostgreSQL DB.

**Q: What supporting tooling is included?**
A: Coverage thresholds (85%+), test factories, custom Jest matchers, `beforeAll`/`afterAll` DB setup and teardown, and a GitHub Actions pipeline.

### [P7.2 — TDD NestJS Feature](P7.2-tdd-nestjs-feature.md)

**Q: What is built with TDD?**
A: A shopping cart (add, remove, update quantity, clear, total) and discount codes, persisted in Redis with a 24-hour TTL.

**Q: What cycle is followed?**
A: Red-green-refactor, with spec files documenting what failed first and what was implemented.

**Q: What is the coverage target?**
A: 100% for the cart module.

---

## Phase 8 — Microservices

### [A8.1 — NestJS Microservices](A8.1-nestjs-microservices.md)

**Q: What are the two message patterns?**
A: Request-response (`@MessagePattern`) where the caller waits for a reply, and events (`@EventPattern`), fire-and-forget for notifications and side effects.

**Q: Why start with the TCP transport?**
A: It's the simplest way to learn the patterns, though it lacks resilience and persistence for production.

**Q: How does `ClientProxy` send messages?**
A: `client.send(pattern, data)` returns an Observable for request-response; `client.emit(pattern, data)` fires an event.

### [A8.2 — Redis Transport](A8.2-redis-transport.md)

**Q: What advantages does Redis have over TCP?**
A: Resilience (messages queue while a service is down), multiple instances sharing a channel with each message delivered to one (competing consumers), and pub/sub for events many services subscribe to.

**Q: Do handlers change when switching to Redis?**
A: No — set `transport: Transport.REDIS`; `@MessagePattern` and `@EventPattern` work the same (ioredis underneath).

**Q: What is service-to-service communication?**
A: Services calling each other directly without the HTTP gateway — e.g. Order Service asking Inventory to reserve stock.

### [A8.3 — RabbitMQ Transport](A8.3-rabbitmq-transport.md)

**Q: Why RabbitMQ over Redis pub/sub?**
A: Redis pub/sub loses messages if no subscriber is listening; RabbitMQ persists them in durable queues until consumers come back.

**Q: What are exchanges, queues and bindings?**
A: Exchanges route messages by routing key, queues store them, and bindings connect exchanges to queues. Durable resources survive broker restarts.

**Q: What does `noAck: false` do?**
A: Manual acknowledgement — the broker waits for `channel.ack()`, and requeues the message if the consumer crashes first.

**Q: What is a Dead Letter Exchange?**
A: Where rejected or expired messages are routed; bind a dead letter queue to it to inspect failures.

### [A8.4 — gRPC Transport](A8.4-grpc-transport.md)

**Q: What does gRPC provide?**
A: A `.proto` file as a strongly typed contract, protobuf binary serialisation (3–10× smaller than JSON), HTTP/2 multiplexing and header compression.

**Q: What are the four gRPC patterns?**
A: Unary, server streaming, client streaming and bidirectional streaming.

**Q: How is gRPC used in NestJS?**
A: Define the proto, handle methods with `@GrpcMethod('Service', 'Method')`, and generate TypeScript types with `protoc` or `ts-proto`.

### [P8.1 — Microservices E-Commerce](P8.1-microservices-e-commerce.md)

**Q: Which services exist?**
A: user-ms (4001), product-ms (4002) and order-ms (4003) on Redis transport, each with its own database, behind an HTTP api-gateway (3000) using `ClientProxy`.

**Q: What happens when an order is created?**
A: order-ms validates the user exists, reserves stock for each item, then creates the order record — via `@MessagePattern` request-response.

**Q: How are errors handled across services?**
A: Each service maps its errors to NestJS RPC exceptions.

### [P8.2 — Event-Driven Notification System](P8.2-event-driven-notification-system.md)

**Q: What events are emitted and who consumes them?**
A: core-service emits `user.registered`, `order.placed` and `payment.failed` to RabbitMQ; notification-ms handles them with `@EventPattern`, sending email and SMS (Twilio stub, SMS for high-value orders).

**Q: How is reliable delivery achieved?**
A: Durable queues, manual `ack`/`nack`, retries with backoff using the `x-death` header, and a Dead Letter Exchange for failures.

**Q: How are event shapes shared?**
A: As TypeScript interfaces in `libs/events`.

---

## Phase 9 — Advanced & Production

### [A9.1 — WebSockets](A9.1-websockets.md)

**Q: How do NestJS gateways work?**
A: `@WebSocketGateway()` creates a WebSocket server (socket.io by default, or `ws`) alongside HTTP; `@SubscribeMessage('event')` handles client events.

**Q: What are rooms?**
A: Subsets of clients you can broadcast to.

**Q: How are WebSocket connections authenticated?**
A: The client sends a JWT in the socket.io handshake `auth` object, validated with the same `JwtService` as HTTP guards.

**Q: How do you scale across instances?**
A: Each instance has its own socket store, so use `@socket.io/redis-adapter` to sync events via Redis pub/sub.

### [A9.2 — GraphQL with NestJS](A9.2-graphql-with-nestjs.md)

**Q: Code-first vs schema-first?**
A: Code-first (recommended) generates the schema from decorated classes; schema-first writes `.graphql` and generates types.

**Q: What are the key decorators?**
A: `@ObjectType`, `@InputType`, `@Resolver`, `@Query`, `@Mutation`, `@ResolveField` (computed fields and relations) and `@Context` (req, user, DataLoaders).

**Q: How does DataLoader solve N+1?**
A: It batches per-item lookups (e.g. authors for 10 posts) into one call, with a per-request cache.

### [A9.3 — OpenAPI / Swagger](A9.3-openapi-swagger.md)

**Q: How does `@nestjs/swagger` generate docs?**
A: From decorators and TypeScript metadata — DTOs with class-validator are inferred automatically; add `@ApiProperty`, `@ApiOperation` and `@ApiTags`.

**Q: What can the spec be used for?**
A: Swagger UI at `/api/docs`, an exported JSON committed to the repo, and client SDK generation with `openapi-generator-cli`.

**Q: What makes good API docs?**
A: Descriptions for every endpoint, request/response examples, error codes, auth requirements and field constraints.

### [A9.4 — Observability](A9.4-observability.md)

**Q: What are the three pillars of observability?**
A: Logs (structured JSON with request/user IDs — never `console.log` in production), metrics (Prometheus scraping, Grafana dashboards), and traces (OpenTelemetry spans exported to Jaeger or Zipkin).

**Q: What do terminus health checks verify?**
A: DB and Redis connectivity, memory usage and disk space, via `/health` for Kubernetes, load balancers and uptime monitors.

### [P9.1 — Real-Time Collaboration API](P9.1-real-time-collaboration-api.md)

**Q: What real-time features does it have?**
A: Document rooms, presence, live cursors and document updates over socket.io, with JWT handshake auth.

**Q: Who receives cursor and document updates?**
A: All other users in the room — not the sender. Presence updates go to all members, and the joiner gets a confirmation.

**Q: How does it scale?**
A: `@socket.io/redis-adapter` plus room membership tracked in Redis.

### [P9.2 — GraphQL API Gateway](P9.2-graphql-api-gateway.md)

**Q: What does the schema expose?**
A: Queries (`posts`, `post`, `user`, `me`), mutations (`createPost`, `updatePost`, `deletePost`, `addComment`), and subscriptions (`postAdded`, `commentAdded(postId)`).

**Q: How is N+1 avoided?**
A: `@ResolveField` resolves `author` per post through a request-scoped DataLoader injected into the GraphQL context.

**Q: How are auth and pagination handled?**
A: JWT guards on resolvers, `@Roles` on mutations, and cursor-based pagination for `posts`.

### [P9.3 — Production NestJS Monolith](P9.3-production-nestjs-monolith.md)

**Q: What makes it production-ready?**
A: Swagger with bearer auth (non-production only), Winston JSON logs with correlation IDs, Prometheus metrics (request count, duration, DB pool, default CPU/memory/GC), and terminus checks for DB, Redis and disk.

**Q: How is it containerised and deployed?**
A: A multi-stage Dockerfile (~150 MB final image), docker-compose with hot reload, and a lint → test → build → push → deploy pipeline that waits for the health check.

**Q: What else is configured?**
A: Global pipes, filters and interceptors, graceful shutdown, Helmet, CORS and compression.

---

## Study Strategy & Debugging

### [HOW_TO_LEARN.md](HOW_TO_LEARN.md)

**Q: How should you approach an assignment?**
A: Read the Background first, type code by hand, run each step and read NestJS's error messages fully, and tick deliverables only when you can explain why they work.

**Q: How should you approach a project?**
A: Create the full directory tree first, write module files before services (NestJS bootstraps via modules), test each layer as you build, and keep the architecture diagram accurate.

**Q: How do providers become available across modules?**
A: Every provider lives in one module; the owner must export it and the consumer must import the owning module.

**Q: What is the NestJS request lifecycle?**
A: Middleware → Guards → Interceptors (before) → Pipes → Handler → Interceptors (after) → Exception Filters (on error) → Response.

**Q: How do you fix "provider not found"?**
A: Check the module is imported where used, the service is in its `providers`, and it's exported if used outside.

**Q: How do you fix a circular dependency?**
A: `forwardRef(() => XxxModule)` in both modules' imports — or better, extract the shared logic into a third module.

**Q: Why might validation not run?**
A: `ValidationPipe` isn't applied globally, the `@Body()`/`@Query()`/`@Param()` decorator is missing, or the DTO is an interface instead of a class with class-validator decorators.
