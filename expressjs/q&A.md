# Express.js — Questions & Answers

Questions and answers drawn from every file in this folder, in curriculum order. Each section links back to its source file.

- [Phase 1 — Node.js & Express Fundamentals](#phase-1--nodejs--express-fundamentals)
- [Phase 2 — Database Integration](#phase-2--database-integration)
- [Phase 3 — Authentication & Security](#phase-3--authentication--security)
- [Phase 4 — Validation & Error Handling](#phase-4--validation--error-handling)
- [Phase 5 — Performance & Caching](#phase-5--performance--caching)
- [Phase 6 — File Handling & External Services](#phase-6--file-handling--external-services)
- [Phase 7 — Testing](#phase-7--testing)
- [Phase 8 — Deployment & DevOps](#phase-8--deployment--devops)
- [Phase 9 — Advanced Patterns](#phase-9--advanced-patterns)
- [Study Strategy](#study-strategy)

---

## Phase 1 — Node.js & Express Fundamentals

### [A1.1 — Node.js Core Internals](A1.1-nodejs-core-internals.md)

**Q: Is Node.js single-threaded?**
A: It runs JavaScript on a single V8 thread but delegates I/O (file reads, network, timers) to libuv, a C library with a thread pool and OS-level async I/O. Completed operations queue callbacks that the event loop runs.

**Q: What are the phases of the event loop?**
A: Timers (`setTimeout`/`setInterval`), I/O callbacks, idle/prepare, poll (waiting for I/O), check (`setImmediate`), and close callbacks. This order explains why `setImmediate` sometimes runs before `setTimeout(fn, 0)` and sometimes not.

**Q: When does `process.nextTick()` run?**
A: It isn't part of the event loop — it runs at the end of the current operation, before the loop continues. Promise callbacks (microtasks) also run before the next tick.

**Q: What are the four kinds of stream?**
A: Readable (emits chunks), Writable (consumes chunks), Transform (reads, modifies, writes), plus Duplex. They're EventEmitters emitting `data`, `end`, `error` and `finish`.

**Q: Why use streams instead of reading a whole file?**
A: Streams process data in chunks instead of loading everything into memory. `.pipe()` handles backpressure automatically, and `await pipeline(...)` from `node:stream/promises` gives cleaner async handling.

**Q: What is a Buffer?**
A: Binary data in a fixed-size allocation in C memory, outside V8's garbage-collected heap. Reading a file without an encoding returns a Buffer; decode with `buffer.toString('utf8')`, encode with `Buffer.from('hello', 'utf8')`.

### [A1.2 — Express Basics](A1.2-express-basics.md)

**Q: What is Express?**
A: A thin layer over Node's `http` module that adds routing, a middleware pipeline, and convenience methods on `req` and `res`.

**Q: What is the request-response cycle?**
A: A request arrives, Express matches routes in registration order, the handler receives `(req, res, next)` and responds with `res.json()`, `res.send()`, etc. If nothing matches, Express falls through to a 404.

**Q: What are the key `req` properties?**
A: `req.params` (URL segments), `req.query` (parsed query string), `req.body` (needs `express.json()`/`express.urlencoded()`), `req.headers`, `req.method`, `req.path`, `req.ip`.

**Q: What are the key `res` methods?**
A: `res.json()`, `res.send()`, chainable `res.status()`, `res.set()`, `res.redirect()` (302) and `res.sendFile()`.

**Q: What are the REST conventions?**
A: `GET /resources` lists, `GET /resources/:id` gets one, `POST` creates, `PUT` fully replaces, `PATCH` partially updates, `DELETE` deletes — with status codes 200, 201, 204, 400, 404 and 500.

### [A1.3 — Middleware Deep Dive](A1.3-middleware-deep-dive.md)

**Q: What is middleware?**
A: A `(req, res, next)` function run in registration order. It can run code, modify `req`/`res`, end the cycle by sending a response, or call `next()`.

**Q: What happens if middleware neither calls `next()` nor responds?**
A: The request hangs — a very common bug. Every code path must do one or the other.

**Q: How do you scope middleware?**
A: `app.use('/api', mw)` runs only for paths under `/api`; route-specific middleware is passed as extra arguments: `app.get('/route', mw1, mw2, handler)`.

**Q: How does error middleware work?**
A: It has four arguments `(err, req, res, next)`, runs only when `next(err)` is called, and must be registered last.

**Q: Does Express 4 catch async errors?**
A: No — rejected promises must be caught and passed to `next`. Express 5 handles this automatically.

**Q: Why must `express.json()` be registered before routes?**
A: Middleware runs in order, so routes registered earlier won't have `req.body` populated.

### [A1.4 — Routing Organization](A1.4-routing-organization.md)

**Q: What is `express.Router()`?**
A: A mini-application supporting middleware and routes. It's itself middleware, mounted with `app.use('/prefix', router)`.

**Q: What is the Router + Controller pattern?**
A: Route files define paths and methods; controllers handle the logic — like the MVC split in Rails or Django.

**Q: What should a controller do — and not do?**
A: Extract data from `req`, call the service/model layer, then respond or call `next(err)`. It shouldn't talk to the database directly; that belongs in a service layer.

**Q: How do nested routers get parent params?**
A: Create the child router with `{ mergeParams: true }`, e.g. for `/users/:userId/posts/:postId/comments`.

### [P1.1 — Recipe API](P1.1-recipe-api.md)

**Q: What does this project build?**
A: A recipe REST API with full CRUD (GET list/one, POST, PUT, PATCH, DELETE) over an in-memory array, organised with `express.Router` and controllers.

**Q: What filtering does it support?**
A: `?category=` and `?search=` on the list endpoint.

**Q: How is validation and error handling done?**
A: Manual body validation (no library) with consistent JSON error responses.

### [P1.2 — Notes API](P1.2-notes-api.md)

**Q: What features does the Notes API have?**
A: CRUD for notes (`title`, `body`, `tags[]`, `pinned`, `color`), combined filters (`?tag=&search=&pinned=`), a unique-tags endpoint, and a pin toggle.

**Q: What middleware stack does it include?**
A: Request logging (method, URL, status, duration), a request ID middleware adding `x-request-id`, CORS, and rate limiting (100 requests per 15 minutes per IP).

**Q: How is rate limiting implemented here?**
A: As a simple in-memory limiter with no external dependencies.

---

## Phase 2 — Database Integration

### [A2.1 — MongoDB with Mongoose](A2.1-mongodb-with-mongoose.md)

**Q: Mongoose vs the native MongoDB driver?**
A: The driver gives full control but no schema enforcement. Mongoose adds schemas with types, defaults, validators and hooks, making models self-documenting and blocking invalid data.

**Q: How does Mongoose manage connections?**
A: Call `mongoose.connect()` once at startup; all models share that pooled connection.

**Q: Schema vs Model vs Document?**
A: A Schema is the blueprint, a Model is the constructor compiled from it, and a Document is one instance with methods like `.save()`, `.toJSON()` and `.toObject()`.

**Q: What are the key schema options?**
A: `required`, `default`, `unique` (unique index), `index`, `trim` and `enum`.

**Q: Virtuals vs methods vs statics?**
A: Virtuals are computed, non-persisted properties (e.g. `fullName`). `schema.methods` are available on documents; `schema.statics` on the Model.

### [A2.2 — PostgreSQL with Sequelize](A2.2-postgresql-with-sequelize.md)

**Q: What is Sequelize?**
A: A promise-based ORM for PostgreSQL, MySQL, SQLite and MSSQL that turns method calls into SQL (`User.findAll()` → `SELECT * FROM "Users"`).

**Q: Which associations does Sequelize support?**
A: `hasMany`, `belongsTo` and `belongsToMany` (via a junction table). Both sides must be declared, and `include` performs JOINs.

**Q: Why use migrations instead of `sequelize.sync()` in production?**
A: `sync()` can destroy data on schema changes. Migrations have `up`/`down` functions, and `sequelize-cli` tracks which have run in a `SequelizeMeta` table.

### [A2.3 — Raw SQL with node-postgres (pg)](A2.3-raw-sql-with-node-postgres-pg.md)

**Q: When is raw SQL better than an ORM?**
A: For performance-critical paths, complex queries, or full SQL control (window functions, CTEs, `EXPLAIN ANALYZE`). `pg` is the foundation Sequelize and Prisma build on.

**Q: Why use a connection pool?**
A: Each new connection needs a TCP handshake, authentication and server state. `pg.Pool` reuses connections; key settings are `max` (default 10), `idleTimeoutMillis` and `connectionTimeoutMillis`.

**Q: How do parameterised queries prevent SQL injection?**
A: Values are sent separately from the query (`$1`, `$2` placeholders), so the database never interprets them as SQL.

**Q: How do you run a transaction with `pg`?**
A: Acquire a client from the pool, run BEGIN → statements → COMMIT (or ROLLBACK on error), and always release the client in `finally` — unreleased clients exhaust the pool.

### [A2.4 — Prisma ORM](A2.4-prisma-orm.md)

**Q: Why Prisma over Sequelize?**
A: Schema-first: `schema.prisma` plus `prisma generate` produces a fully typed client with autocomplete for every field and filter. Its plain-object query API is more readable, and Prisma Studio provides a visual browser.

**Q: `prisma migrate dev` vs `prisma db push`?**
A: `migrate dev` generates and applies tracked SQL migration files. `db push` syncs the schema without migration files — fine for prototyping, not for production.

**Q: How should you instantiate `PrismaClient`?**
A: Create one shared instance; it pools connections internally. In serverless, configure pooling with the `@prisma/adapter-*` packages.

### [P2.1 — Blog API with MongoDB](P2.1-blog-api-with-mongodb.md)

**Q: What resources does it model?**
A: Authors (register, profile), posts (CRUD, publish/unpublish, author populated), comments, and tags with post counts.

**Q: Which pagination styles are implemented?**
A: Both offset (the default) and cursor-based pagination for posts.

### [P2.2 — E-commerce Product Catalog with PostgreSQL](P2.2-e-commerce-product-catalog-with-postgresql.md)

**Q: Which associations does the catalog use?**
A: Categories to products, products to images (one-to-many), and products to inventory (one-to-one).

**Q: What querying features does it provide?**
A: List by category, full-text search and a price range filter.

**Q: How is the schema managed?**
A: With Sequelize migrations to version the schema and seeders for test data.

---

## Phase 3 — Authentication & Security

### [A3.1 — JWT Authentication](A3.1-jwt-authentication.md)

**Q: What is the structure of a JWT?**
A: Three Base64URL parts — `header.payload.signature`. The header names the algorithm; the payload holds claims like `sub`, `iat` and `exp`; the signature proves it wasn't tampered with.

**Q: What is the trade-off of JWTs?**
A: They're verified without a DB lookup (stateless and fast), but can't be invalidated before expiry without extra infrastructure like a blacklist.

**Q: Why use both access and refresh tokens?**
A: A short-lived access token (≈15 min) limits damage if stolen; a long-lived refresh token (7–30 days, HttpOnly cookie or secure storage) gets new access tokens at `/auth/refresh`.

**Q: How do you invalidate tokens on logout?**
A: Store the token's JTI in a Redis blacklist with a TTL matching its expiry, and check it on each request.

**Q: What is token rotation?**
A: Each refresh issues a new refresh token and blacklists the old one, so a replayed stolen token fails and forces a re-login.

### [A3.2 — Passport.js Strategies](A3.2-passport-js-strategies.md)

**Q: What is Passport.js?**
A: Authentication middleware built on pluggable strategies (500+: local, JWT, Google, GitHub, SAML, LDAP…). On success it calls `req.logIn(user)` for sessions or sets `req.user` for stateless auth.

**Q: Sessions vs stateless JWT?**
A: Sessions store state on the server and send a `session_id` cookie; JWTs carry state in a signed token sent in the Authorization header. Use sessions for traditional web apps you control and JWTs for mobile, SPA or third-party clients. Sessions are easier to invalidate.

**Q: What are the steps of the OAuth2 flow?**
A: User clicks "Login with Google" → consent screen → redirect back with an authorization code → server exchanges the code for an access token → server fetches the profile → server creates/updates the user and issues a session or JWT.

### [A3.3 — Security Hardening](A3.3-security-hardening.md)

**Q: Which headers does Helmet set?**
A: `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Strict-Transport-Security`, and the legacy `X-XSS-Protection`.

**Q: What does rate limiting protect against?**
A: Brute-force and DoS attacks. `express-rate-limit` returns 429 when a window is exceeded. Behind a proxy set `trustProxy: true`; across instances use a Redis store (`rate-limit-redis`).

**Q: What is the dangerous CORS misconfiguration?**
A: `Access-Control-Allow-Origin: *` together with `Access-Control-Allow-Credentials: true`.

**Q: Validation vs sanitisation?**
A: Validation checks format; sanitisation removes dangerous content. Use parameterised queries for SQL injection and `sanitize-html`/`DOMPurify` for XSS. Never trust user input.

### [A3.4 — Role-Based Authorization](A3.4-role-based-authorization.md)

**Q: Authentication vs authorisation?**
A: Authentication asks "who are you?"; authorisation asks "what can you do?". Implement them as separate middleware layers.

**Q: What is RBAC?**
A: Permissions are assigned to roles and roles to users. For flat roles, check `req.user.role` against an allowed list; for hierarchies, give numeric levels (user=1, moderator=2, admin=3) and compare with `>=`.

**Q: What is ownership-based authorisation?**
A: Checking `resource.userId === req.user.id` so users can edit only their own resources — implemented as a middleware factory taking the model and ownership field.

**Q: Why keep an audit log?**
A: Sensitive actions (admin operations, permission changes, exports) should record who did what, when, and the result, for security reviews and compliance.

### [P3.1 — Auth API](P3.1-auth-api.md)

**Q: Which endpoints does the auth API expose?**
A: Register (sends a verification email), verify-email, login (access + refresh tokens), refresh, logout, forgot-password, reset-password, and a protected `/me`.

**Q: What happens on refresh?**
A: The old refresh token is blacklisted immediately (rotation) and new tokens are issued.

**Q: What does logout do?**
A: Blacklists the refresh token in Redis.

### [P3.2 — Multi-role Application](P3.2-multi-role-application.md)

**Q: How are roles enforced?**
A: Three roles — `admin`, `moderator`, `user` — with route-level `requireRole('admin')` guards and a `requireOwnerOrAdmin` ownership check (resources need an `author` or `userId` field).

**Q: How does the audit log work?**
A: Every write is logged (who, what, when, before/after) by intercepting `res.json`. Logs are immutable, retained for a year via a TTL, and a logging failure never fails the request.

**Q: Who can see the audit log?**
A: Admins only, via a paginated `GET /api/admin/audit-log`.

---

## Phase 4 — Validation & Error Handling

### [A4.1 — Joi Validation](A4.1-joi-validation.md)

**Q: What is Joi?**
A: A declarative schema language and validator (`Joi.string().email().required()`) that keeps validation separate from business logic, gives consistent error messages, and documents what the API accepts.

**Q: What is the validation middleware pattern?**
A: A `validate` factory takes a schema (or schemas for body/params/query) and returns middleware that calls `next()` or responds 400.

**Q: Why set `abortEarly: false`?**
A: To collect all errors rather than just the first, so users see every problem at once.

**Q: How do you do conditional validation?**
A: `.when()` (e.g. `shippingAddress` required only when `requiresShipping` is true) and `.alternatives().conditional()` for more complex cases.

### [A4.2 — express-validator](A4.2-express-validator.md)

**Q: When should you choose express-validator over Joi?**
A: For inline validation next to routes, built-in sanitisers (`.trim()`, `.escape()`, `.normalizeEmail()`) and simple cases. Choose Joi for reusable central schemas, complex nested or conditional validation, and schemas as documentation.

**Q: What is a validation chain?**
A: `body('email')` returns middleware you extend with `.isEmail()`, `.isLength()`, `.notEmpty()`, `.custom()`. `validationResult(req)` collects errors (`.isEmpty()`, `.array()`).

**Q: How do you check uniqueness during validation?**
A: With an async `.custom(fn)` that throws or rejects if the value (e.g. email) already exists.

### [A4.3 — Global Error Handling](A4.3-global-error-handling.md)

**Q: Operational vs programmer errors?**
A: Operational errors are expected failures (wrong password, DB down, timeout, missing file) — handle them and return a useful response. Programmer errors are bugs (wrong types, reading `undefined`) — log prominently or crash, since the app's state is unknown.

**Q: How should the error handler respond to each?**
A: Client-friendly messages for operational errors; a generic "Internal server error" for programmer errors to avoid leaking details.

**Q: How do you avoid try/catch in every async handler?**
A: An `asyncWrapper` higher-order function that forwards rejections to `next`. Express 5 does this automatically.

**Q: What is RFC 7807?**
A: A standard JSON "Problem Details" format for API error responses.

### [A4.4 — Logging with Winston](A4.4-logging-with-winston.md)

**Q: Why structured logging?**
A: JSON logs with consistent fields (timestamp, level, message, metadata) can be filtered, grouped and alerted on by aggregators like Datadog, CloudWatch or ELK.

**Q: What are Winston's core concepts?**
A: Logger, transports (console, file, HTTP, DB), levels (`error`, `warn`, `info`, `http`, `verbose`, `debug`, `silly`) and formats (`json`, `printf`, `combine`, `colorize`, `timestamp`).

**Q: How should logging differ between dev and production?**
A: Colourised human-readable output in development; JSON to stdout or files in production.

**Q: What is a correlation ID?**
A: A UUID assigned at the start of each request and included in every log entry for it, so all logs from one request can be found together.

### [P4.1 — Validated CRUD API](P4.1-validated-crud-api.md)

**Q: How is validation applied?**
A: Strict Joi schemas on every product endpoint via a central `validate(schema)` middleware factory.

**Q: What error format is used?**
A: RFC 7807 `application/problem+json` on every error, with an `AppError` class for operational errors.

**Q: What is logged per request?**
A: Structured JSON via Winston: request ID, method, URL, status, duration and user.

### [P4.2 — Error Monitoring Setup](P4.2-error-monitoring-setup.md)

**Q: Why must Sentry be imported first?**
A: It must initialise before any route handler so it can instrument and capture errors.

**Q: How is request context shared without passing `req` everywhere?**
A: `AsyncLocalStorage` stores the request ID and user ID, so all code inside the request can read them and every log entry includes them.

**Q: Which Winston transports are used?**
A: Console (dev), File (prod), and a custom Sentry transport for error-level logs. Sensitive fields are scrubbed.

**Q: What else does it include?**
A: Custom Sentry breadcrumbs for business events, alerting on 5xx errors, and `/health` and `/metrics` endpoints with in-memory counters.

---

## Phase 5 — Performance & Caching

### [A5.1 — Redis Caching](A5.1-redis-caching.md)

**Q: How does cache-aside work?**
A: On read, check the cache; on a miss, fetch from the DB, store with a TTL, and return. On write/delete, invalidate or update the entry. Only requested data gets cached.

**Q: What are the alternatives to cache-aside?**
A: Write-through (update the cache on every write — consistent but costlier writes) and write-behind (write to cache, flush to DB asynchronously).

**Q: Why use ioredis?**
A: It supports Promises, clustering, Sentinel, Lua scripts and pipelining. Key operations: `SET … EX`, `GET`, `DEL`, `MGET`, `EXPIRE`.

**Q: How should you choose TTLs?**
A: By how often data changes: user profiles 5–15 min, product catalogue 1 h, news 15 min, tokens and sessions matching their expiry. Under 30 s gives little benefit; very long TTLs risk staleness — measure real change rates.

### [A5.2 — Pagination & Filtering](A5.2-pagination-and-filtering.md)

**Q: What are the problems with offset pagination?**
A: Inserts or deletes during paging cause duplicates or skipped items ("page drift"), and large `SKIP` values force scanning skipped rows.

**Q: How does cursor pagination fix them?**
A: It anchors to the last seen ID or timestamp ("10 items after ID 42"), avoiding drift and using indexes. You can't jump to arbitrary pages, which infinite scroll rarely needs.

**Q: How should sorting and filtering be exposed?**
A: As query params (`?sort=-createdAt&minPrice=10`). Always whitelist sort fields and convert filter types.

**Q: How should text search be done?**
A: With database full-text search (MongoDB `$text`, PostgreSQL `tsvector`) rather than regex, which can't use indexes.

### [A5.3 — Clustering & PM2](A5.3-clustering-and-pm2.md)

**Q: Why use the cluster module?**
A: A single Node process uses one core. `cluster` forks a master and N workers on separate cores, with the OS distributing connections.

**Q: What is the implication of workers sharing no memory?**
A: State must live in Redis or a database — which is why rate limiters and sessions need Redis-backed stores when clustering.

**Q: What does PM2 add?**
A: Auto-restart, log management, monitoring, cluster mode (`os.cpus().length` workers) vs fork mode, zero-downtime `pm2 reload`, `pm2 save` + `pm2 startup` for boot, `pm2 monit` and `pm2 logs`.

**Q: How does PM2 contain memory leaks?**
A: `max_memory_restart` (e.g. `'500M'`) restarts a worker that exceeds the threshold.

### [A5.4 — Query Optimization](A5.4-query-optimization.md)

**Q: Why do queries get slow?**
A: Without an index MongoDB does a collection scan, reading every document. An index (a B-tree of sorted fields) enables fast lookups.

**Q: What should you look for in `explain('executionStats')`?**
A: `COLLSCAN` vs `IXSCAN`, `totalKeysExamined`, `totalDocsExamined`, `executionTimeMillis` and `nReturned`. Aim for `totalDocsExamined / nReturned` close to 1.

**Q: What is the cost of indexes?**
A: They slow writes (every insert/update updates them) and consume memory.

**Q: Which index types exist?**
A: Single field, compound (matching sort order), text, and partial (e.g. only active documents).

**Q: What is the ESR rule?**
A: For compound indexes: Equality fields first, Sort fields second, Range fields last.

### [P5.1 — High-traffic News API](P5.1-high-traffic-news-api.md)

**Q: How are reads optimised?**
A: Redis cache-aside for single articles and trending lists (5 min), cursor pagination, and compound indexes for common queries, with `explain()` compared before and after.

**Q: How does the reusable cache middleware work?**
A: It intercepts `res.json` to capture and store the response.

**Q: How are view counts updated without slowing responses?**
A: Incremented in the background without awaiting.

**Q: How is the cache kept correct?**
A: On article update or delete, the cached article and trending list are invalidated.

### [P5.2 — PM2 Production Setup](P5.2-pm2-production-setup.md)

**Q: What does `ecosystem.config.js` configure?**
A: Cluster mode with one worker per core, env vars, log paths, and zero-downtime reloads using `wait_ready`.

**Q: How does a worker signal it's ready?**
A: It notifies PM2 once listening, so the old worker is only killed after the new one is ready.

**Q: How does graceful shutdown work?**
A: On `SIGINT`/`SIGTERM`, stop accepting connections, drain in-flight requests, close the DB, and force-kill if it takes too long.

**Q: Why also use the raw cluster module?**
A: To understand clustering before PM2 abstracts it; each worker runs the Express app independently.

---

## Phase 6 — File Handling & External Services

### [A6.1 — File Uploads](A6.1-file-uploads.md)

**Q: What does Multer do?**
A: Parses `multipart/form-data` uploads, storing files on disk (`diskStorage`) or in memory as a Buffer (`memoryStorage`). Use memory when you'll transform or upload to S3; disk when writing directly.

**Q: How should file types be validated?**
A: By checking the file's magic bytes on the server — Multer doesn't validate, and the `Content-Type` header is client-controlled.

**Q: What is sharp?**
A: A fast image library built on native libvips for resize, crop, rotate, format conversion and compression: `sharp(buffer).resize(w, h).jpeg({ quality: 80 }).toBuffer()`.

**Q: How are files stored in S3?**
A: With `@aws-sdk/client-s3` (`PutObjectCommand`, `GetObjectCommand`). Presigned URLs allow direct browser-to-S3 uploads. Credentials come from environment variables.

### [A6.2 — Sending Email](A6.2-sending-email.md)

**Q: What are transactional emails?**
A: Emails triggered by user actions — welcome, password reset, order confirmation, invoice — sent via a dedicated service (SendGrid, SES, Mailgun, Postmark) that handles deliverability and bounces.

**Q: Nodemailer vs `@sendgrid/mail`?**
A: Nodemailer is an SMTP client for any SMTP server; SendGrid's HTTP API is faster with better delivery metrics.

**Q: Why are HTML emails different from web pages?**
A: Many clients (Outlook) lack modern CSS support, so use inline styles and tables, test with Litmus/Email on Acid, and render templates with Handlebars.

**Q: Why queue emails with Bull?**
A: Sending synchronously delays responses when the email service is slow and risks duplicates. A Redis-backed queue lets the API respond immediately while a worker sends and retries.

### [A6.3 — Third-Party APIs](A6.3-third-party-apis.md)

**Q: axios or native fetch?**
A: `fetch` (Node 18+) is fine for simple cases. axios adds interceptors, automatic JSON, timeouts, cancellation, and throws on HTTP errors — worth it for production clients.

**Q: What are interceptors used for?**
A: Request: auth headers, correlation IDs, logging. Response: logging, error transformation, token refresh on 401.

**Q: How should retries work?**
A: Exponential backoff (1 s, 2 s, 4 s) for network failures and 5xx, only for idempotent methods (GET, PUT, DELETE) — never blindly retry POST. `axios-retry` helps.

**Q: How do you secure incoming webhooks?**
A: Verify the HMAC-SHA256 signature of the payload with the shared secret.

### [A6.4 — Payment Integration with Stripe](A6.4-payment-integration-with-stripe.md)

**Q: What is Stripe's payment flow?**
A: The server creates a `PaymentIntent` or `CheckoutSession` → the client confirms with Stripe.js/Elements → Stripe sends a webhook → the server updates the order on `payment_intent.succeeded`.

**Q: Why never handle raw card numbers?**
A: Stripe Elements/Checkout collect cards on Stripe's servers, keeping you out of PCI scope.

**Q: What are idempotency keys for?**
A: If a call fails mid-flight you don't know whether the charge happened; resending the same key returns the original result instead of charging twice. Generate one UUID per user action, stored with the order.

**Q: What makes up a subscription?**
A: A Product, a Price (interval and amount), and a Subscription (Customer + Price, auto-renewing). Key events: `invoice.payment_succeeded` and `customer.subscription.deleted`.

### [P6.1 — File Upload Service](P6.1-file-upload-service.md)

**Q: What does the service do?**
A: Accepts JPEG/PNG/WebP up to 10 MB, generates original, medium (800 px) and thumbnail (200 px) sizes, stores them in S3, saves metadata in MongoDB, and returns signed URLs.

**Q: How is file access controlled?**
A: Objects have no public ACL; access is only via signed URLs valid for 1 hour.

**Q: Which endpoints exist?**
A: Upload, list (paginated), get details with fresh signed URLs, and delete (from S3 and DB).

### [P6.2 — E-commerce with Payments](P6.2-e-commerce-with-payments.md)

**Q: Where is the cart stored?**
A: In Redis.

**Q: How are duplicate charges prevented at checkout?**
A: An idempotency key built from user ID plus a timestamp bucket, so double clicks don't double-charge.

**Q: Why does the Stripe webhook route use `express.raw()`?**
A: Signature verification needs the raw body, so `express.raw()` must come before `express.json()` on that route.

**Q: How does the webhook stay safe to retry?**
A: It skips creating an order if one already exists for the session, and always returns 200 because Stripe retries on non-2xx. Confirmation emails are queued with Bull + Nodemailer.

---

## Phase 7 — Testing

### [A7.1 — Jest Fundamentals](A7.1-jest-fundamentals.md)

**Q: What should tests check?**
A: Behaviour (the contract), not implementation — refactoring without changing behaviour shouldn't break them.

**Q: What should you test, and what not?**
A: Test utilities, validation, transformations and business rules. Don't test library internals, simple property access, or unmocked DB queries.

**Q: What is AAA?**
A: Arrange (set up), Act (call the function), Assert (verify).

**Q: How should you treat coverage?**
A: It shows what code ran, not whether tests are good. Aim for 80%+ on business logic rather than 100% everywhere.

### [A7.2 — Supertest for API Testing](A7.2-supertest-for-api-testing.md)

**Q: How does Supertest work?**
A: It wraps the Express `app` (not a listening server) and sends requests through the full middleware stack without a real port, letting you assert on status, body and headers.

**Q: What is mongodb-memory-server?**
A: A real MongoDB binary run in memory — started in `beforeAll`, torn down in `afterAll`, with no Docker or external DB.

**Q: Why is that useful?**
A: Tests are isolated, deterministic, and run anywhere, including CI.

### [A7.3 — Unit Testing with Mocks](A7.3-unit-testing-with-mocks.md)

**Q: What should you mock?**
A: Dependencies that are slow, unreliable or side-effectful: DB calls, email, external HTTP, file writes and time. Never mock the code under test.

**Q: `jest.mock` vs `jest.fn` vs `jest.spyOn`?**
A: `jest.mock('module')` replaces a whole module; `jest.fn()` creates a mock function to inject; `jest.spyOn(obj, 'method')` wraps an existing method to assert on calls and control returns.

**Q: How do you reset mocks?**
A: `mockReset()` clears history and implementations, `mockRestore()` restores the original (spyOn only), and `jest.clearAllMocks()` in `beforeEach` clears all history.

### [A7.4 — Integration and E2E Tests](A7.4-integration-and-e2e-tests.md)

**Q: Unit vs integration tests?**
A: Unit tests isolate one function — fast. Integration tests run route → controller → service → DB together — slower but verify the whole flow. Use unit tests for complex logic and integration tests for endpoints.

**Q: What are test data factories?**
A: Functions that build valid objects with sensible defaults plus overrides, using unique emails (timestamp or counter) to avoid collisions.

**Q: How are tests run in CI?**
A: GitHub Actions installs dependencies, starts required services (e.g. Redis), and runs `npm test` on every push and PR.

### [P7.1 — Fully Tested REST API](P7.1-fully-tested-rest-api.md)

**Q: What are the three test layers?**
A: Unit (controllers/services with mocked models), integration (Supertest + in-memory MongoDB), and e2e (real DB in CI).

**Q: How are external HTTP calls mocked?**
A: With MSW, e.g. for an external contact-enrichment API.

**Q: What are the coverage and CI requirements?**
A: 85%+ coverage, collections cleaned between tests, and a GitHub Actions workflow publishing an lcov + HTML coverage artifact on every PR.

### [P7.2 — TDD Blog API](P7.2-tdd-blog-api.md)

**Q: What is the TDD loop used here?**
A: Write a failing test → implement the minimum code to pass → refactor → next test.

**Q: What does that look like in practice?**
A: `postsService.test.js` is written before `postsService.js` exists; after tests pass, the slug logic is extracted into its own module and all tests still pass.

**Q: Why keep logic in a service layer?**
A: It's easy to unit test. The target is 90%+ coverage.

---

## Phase 8 — Deployment & DevOps

### [A8.1 — Docker for Node.js](A8.1-docker-for-nodejs.md)

**Q: Why use multi-stage builds?**
A: Build tools and devDependencies stay in early stages; only production artifacts reach the slim final image — smaller, with less attack surface.

**Q: Why run as a non-root user?**
A: Containers default to root, so an exploit would get root. A dedicated user limits the blast radius.

**Q: Why is `.dockerignore` essential?**
A: Without it, `COPY . .` copies `node_modules`, `.env` files, git history and tests into the image.

**Q: How does layer caching speed up builds?**
A: Copy `package*.json` and run `npm ci` before `COPY . .`, so dependencies reinstall only when `package.json` changes.

### [A8.2 — Environment Configuration](A8.2-environment-configuration.md)

**Q: What does 12-Factor say about config?**
A: Store it in the environment, so the same code deploys anywhere by changing only env vars.

**Q: How should `.env` files be handled?**
A: Local development only — never commit `.env`; commit `.env.example` with placeholders.

**Q: Why validate config at startup?**
A: Missing or malformed variables otherwise cause cryptic failures later. Fail fast with a clear error.

**Q: Where should production secrets come from?**
A: Secret managers — AWS Secrets Manager/Parameter Store, Vault, GitHub Actions secrets, or platform env settings. The code only reads env vars.

### [A8.3 — GitHub Actions CI/CD](A8.3-github-actions-ci-cd.md)

**Q: How are workflows structured?**
A: YAML in `.github/workflows/`; jobs run on runners and contain steps (`run` commands or `uses` actions). Triggers include `push`, `pull_request`, `schedule` and `workflow_dispatch`.

**Q: How do you require CI before merging?**
A: Branch protection rules.

**Q: What image tagging strategy is recommended?**
A: `latest` for main, `sha-<commit>` for exact versions (enables rollback), and semver tags for releases.

**Q: How is deployment done?**
A: Railway or Render deploy Docker containers on push, or via their CLI/API from the pipeline.

### [A8.4 — Monitoring & Health Checks](A8.4-monitoring-and-health-checks.md)

**Q: Liveness vs readiness?**
A: Liveness: is the process running (just 200)? Readiness: can it serve traffic (DB, Redis, dependencies OK)? Kubernetes restarts pods on failed liveness and stops routing on failed readiness — use separate endpoints. Return 503 when degraded.

**Q: What is Prometheus and `prom-client`?**
A: Prometheus scrapes time-series metrics from `/metrics`; `prom-client` exposes default Node metrics (memory, CPU, event loop lag) plus custom ones.

**Q: Counter vs gauge vs histogram?**
A: A counter only goes up (total requests); a gauge is a current value (active connections); a histogram measures distribution (latency buckets).

**Q: What should graceful shutdown do on `SIGTERM`?**
A: Stop accepting connections (`server.close()`), finish in-flight requests, close DB connections, and exit — otherwise requests drop and writes may be incomplete.

### [P8.1 — Dockerized API with CI/CD](P8.1-dockerized-api-with-ci-cd.md)

**Q: What are the Dockerfile stages?**
A: A builder that installs all deps (and builds if needed), then a production stage copying only production `node_modules` and source, owned by a non-root user.

**Q: Why use the exec form of `CMD`?**
A: So `SIGTERM` reaches Node directly rather than a shell.

**Q: What does the pipeline do?**
A: Lint → test → build and push the image → deploy to Render, with env vars managed in Render env groups and docker-compose (MongoDB, Redis) for local dev.

### [P8.2 — Production-grade API](P8.2-production-grade-api.md)

**Q: Which probes and metrics does it expose?**
A: `/health/live`, `/health/ready`, and Prometheus `/metrics` with default Node metrics, a request duration histogram, an active connections gauge and an error counter.

**Q: Why normalise routes in metric labels?**
A: To avoid high cardinality (e.g. per-ID label values).

**Q: How are rate limiting and the metrics endpoint protected?**
A: Per-IP limits backed by Redis (`rate-limiter-flexible`), and the metrics endpoint is protected in production.

**Q: What else is hardened?**
A: Graceful shutdown with a configurable timeout and structured JSON logs without colourisation.

---

## Phase 9 — Advanced Patterns

### [A9.1 — WebSockets with Socket.io](A9.1-websockets-with-socket-io.md)

**Q: How do WebSockets differ from HTTP?**
A: HTTP is client-initiated request-response; WebSockets keep a persistent TCP connection so the server can push anytime.

**Q: What does Socket.io add?**
A: Fallbacks, rooms, namespaces and an event-emitter API over raw WebSockets.

**Q: Socket vs room vs namespace vs Redis adapter?**
A: A socket is one client; a room is a named group a socket can join; a namespace is a virtual channel on the same connection (`/chat`, `/admin`); the Redis adapter syncs events across server processes.

### [A9.2 — GraphQL with Apollo Server](A9.2-graphql-with-apollo-server.md)

**Q: How does GraphQL differ from REST?**
A: One endpoint (`/graphql`) where the client specifies exactly the data it needs, instead of many URLs.

**Q: Why mount Apollo Server on Express?**
A: To keep existing REST routes and middleware alongside GraphQL.

**Q: What are typeDefs, resolvers and context?**
A: typeDefs are the SDL schema; resolvers fulfil each field; context is a per-request object (user, db, loaders) shared by resolvers.

**Q: How does DataLoader solve N+1?**
A: It batches and caches DB calls within a single request.

### [A9.3 — Microservices Basics](A9.3-microservices-basics.md)

**Q: What is a microservice?**
A: A small, independently deployable service owning its data and communicating over the network.

**Q: Synchronous vs asynchronous communication?**
A: Synchronous HTTP (axios) is simple but couples services; asynchronous queues (Bull, RabbitMQ) are fire-and-forget, more resilient, and suited to background work.

**Q: Bull vs RabbitMQ?**
A: Bull uses Redis — good for job queues within one deployment. RabbitMQ is a dedicated broker for multi-service fan-out and reliable delivery with message acknowledgement.

### [A9.4 — API Versioning & Swagger Documentation](A9.4-api-versioning-and-swagger-documentation.md)

**Q: OpenAPI vs Swagger UI?**
A: OpenAPI 3.0 is the spec; Swagger UI renders it as interactive docs (via `swagger-ui-express`).

**Q: What does swagger-jsdoc do?**
A: Generates the spec from `@swagger`/`@openapi` JSDoc blocks, so there's no separate YAML to maintain.

**Q: Why version APIs and how do you deprecate?**
A: `/api/v1`, `/api/v2` let you ship breaking changes without forcing every client to upgrade. `Deprecation` and `Sunset` headers give advance notice.

### [P9.1 — Real-time Chat App](P9.1-real-time-chat-app.md)

**Q: How are history and live messages split?**
A: History loads via REST (`GET /api/rooms/:id/messages`); new messages arrive via Socket.io.

**Q: Which socket events exist?**
A: `room:join`, `room:leave`, `message:send`, `message:new`, `typing:start`, `typing:stop` and `presence:update`.

**Q: How is online presence tracked?**
A: A Redis sorted set per room scored by last-seen timestamp; joiners receive the current online list and others are notified.

**Q: How do typing indicators avoid getting stuck?**
A: They're debounced and auto-cleared after 3 s in case the client never sends `typing:stop`.

### [P9.2 — Microservices Order System](P9.2-microservices-order-system.md)

**Q: Which services exist?**
A: user-service (3001), product-service (3002), order-service (3003) and notification-service (3004), run together with Redis and MongoDB via docker-compose.

**Q: What happens when an order is created?**
A: Verify the user → validate products and prices (parallel) → decrement inventory (parallel) → create the order → publish an `order.created` Bull job that notification-service consumes to send email.

**Q: How do services authenticate and stay resilient?**
A: A shared `INTERNAL_API_KEY` header, with internal HTTP clients using timeouts and retries.

### [P9.3 — Full-featured REST API](P9.3-full-featured-rest-api.md)

**Q: What does it combine?**
A: `/api/v1` and `/api/v2` task routes, OpenAPI docs, Helmet, CORS, Redis-backed rate limiting, CSRF on state-changing routes, Prometheus metrics, graceful shutdown with probes, a multi-stage Dockerfile and a lint → test → build/push → deploy pipeline.

**Q: What extra hardening is applied?**
A: Helmet's security headers (CSP, HSTS, X-Frame-Options, etc.) and protection against HTTP Parameter Pollution.

---

## Study Strategy

### [HOW_TO_LEARN.md](HOW_TO_LEARN.md)

**Q: What is the core principle?**
A: Reading doesn't make you a programmer — type, run, break and fix every concept yourself.

**Q: What should you do after finishing an assignment?**
A: Delete and rewrite it from memory, write three bullets (learned, confused, open question), and read the official docs.

**Q: How should you approach a project?**
A: Plan for 15 minutes (file structure, endpoints, applicable concepts), build a skeleton that starts without errors, implement one endpoint at a time testing with curl/Postman, then review deliverables.

**Q: Why keep `app.js` separate from `server.js`?**
A: So tests can import `app` without starting the HTTP server.

**Q: What is the escalation order when stuck?**
A: Re-read the error → console.log the problem area → official docs → search the exact error (with library and version) → 10-minute break → rubber-duck debugging.

**Q: Which phases matter most for interviews and production?**
A: Interviews: Phase 3 (auth), 4 (error handling) and 7 (testing). Production: Phases 5, 8 and 9 (Docker, CI/CD, monitoring).

**Q: When is an assignment "done"?**
A: It runs without errors, you can explain every line, you rewrote it from memory, and you read the relevant docs.
