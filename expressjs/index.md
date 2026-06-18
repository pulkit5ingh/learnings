# Backend Developer using Express.js — Full Roadmap

A structured, project-driven curriculum to take you from Node.js fundamentals to production-grade Express.js APIs.

---

## How to Use This Curriculum

- Complete assignments in order within each phase before starting projects.
- Each assignment is hands-on — open your editor and write the code.
- Projects integrate everything from the phase. Build them from scratch, not from templates.
- See `HOW_TO_LEARN.md` for the full study strategy.

---

## Phase Overview

| Phase | Name | Assignments | Projects |
|-------|------|-------------|----------|
| 1 | Node.js & Express Fundamentals | A1.1–A1.4 | P1.1–P1.2 |
| 2 | Database Integration | A2.1–A2.4 | P2.1–P2.2 |
| 3 | Authentication & Security | A3.1–A3.4 | P3.1–P3.2 |
| 4 | Validation & Error Handling | A4.1–A4.4 | P4.1–P4.2 |
| 5 | Performance & Caching | A5.1–A5.4 | P5.1–P5.2 |
| 6 | File Handling & External Services | A6.1–A6.4 | P6.1–P6.2 |
| 7 | Testing | A7.1–A7.4 | P7.1–P7.2 |
| 8 | Deployment & DevOps | A8.1–A8.4 | P8.1–P8.2 |
| 9 | Advanced Patterns | A9.1–A9.4 | P9.1–P9.3 |

---

## Phase 1 — Node.js & Express Fundamentals

**Goal:** Understand Node.js internals, build and organize Express applications from scratch.

### Assignments
- **A1.1** — Node.js core: event loop, libuv, process, Buffer, streams, EventEmitter, path/fs/os modules
- **A1.2** — Express basics: app setup, routing (GET/POST/PUT/DELETE/PATCH), route params, query strings, req/res objects
- **A1.3** — Middleware deep dive: middleware chain, custom middleware, error middleware (4-arg), third-party middleware
- **A1.4** — Routing organization: express.Router(), controller pattern, nested routers, route parameters

### Projects
- **P1.1** — Recipe API with CRUD operations, in-memory store, organized routes, input validation
- **P1.2** — Notes API with tags, search functionality, full middleware stack

---

## Phase 2 — Database Integration

**Goal:** Connect Express to multiple databases using both ORMs and raw drivers.

### Assignments
- **A2.1** — MongoDB with Mongoose: Schema, Model, CRUD, virtuals, methods/statics, toJSON
- **A2.2** — PostgreSQL with Sequelize: Model.define, associations, migrations, seeders
- **A2.3** — Raw SQL with node-postgres (pg): parameterized queries, transactions, connection pool
- **A2.4** — Prisma ORM: schema.prisma, prisma migrate, CRUD operations, relations, raw queries

### Projects
- **P2.1** — Blog API with MongoDB: posts, comments, tags, Mongoose models, pagination, population
- **P2.2** — E-commerce product catalog with PostgreSQL: products, categories, Sequelize associations, migrations

---

## Phase 3 — Authentication & Security

**Goal:** Implement secure authentication flows and harden APIs against common attacks.

### Assignments
- **A3.1** — JWT authentication: sign/verify, access + refresh tokens, token rotation, Redis blacklisting
- **A3.2** — Passport.js strategies: Local, JWT, Google OAuth2, session vs stateless
- **A3.3** — Security hardening: helmet, rate limiting, CORS, CSRF, SQL injection prevention, input sanitization
- **A3.4** — Role-based authorization: roles/permissions design, middleware guards, resource ownership checks

### Projects
- **P3.1** — Auth API: register/login/logout/refresh, email verification, password reset, JWT, Redis blacklist
- **P3.2** — Multi-role application: admin/moderator/user roles, protected routes per role, audit log

---

## Phase 4 — Validation & Error Handling

**Goal:** Build robust APIs with structured validation, consistent error responses, and production-level logging.

### Assignments
- **A4.1** — Joi validation: schema validation, custom messages, nested objects, conditional validation
- **A4.2** — express-validator: check(), body(), validationResult(), custom validators, middleware chains
- **A4.3** — Global error handling: AppError class, async error wrapper, operational vs programmer errors
- **A4.4** — Logging: winston setup, log levels, file transport, JSON logs, request logging, correlation IDs

### Projects
- **P4.1** — Validated CRUD API: all endpoints validated with Joi, RFC 7807 error responses, structured logging
- **P4.2** — Error monitoring setup: winston + Sentry, error tracking, alert on 5xx errors, request ID propagation

---

## Phase 5 — Performance & Caching

**Goal:** Make Express APIs fast and scalable through caching, clustering, and query optimization.

### Assignments
- **A5.1** — Redis caching: ioredis, cache-aside pattern, TTL strategies, caching middleware
- **A5.2** — Pagination & filtering: cursor pagination, offset pagination, sort/filter, aggregation
- **A5.3** — Clustering & PM2: Node.js cluster module, PM2 config, worker processes, zero-downtime reload
- **A5.4** — Query optimization: explain(), indexes, MongoDB aggregation pipeline, query profiling

### Projects
- **P5.1** — High-traffic News API: Redis caching, pagination, sorted articles, index optimization
- **P5.2** — PM2 production setup: cluster mode, ecosystem.config.js, health checks, log management

---

## Phase 6 — File Handling & External Services

**Goal:** Handle file uploads, send emails, consume third-party APIs, and integrate payments.

### Assignments
- **A6.1** — File uploads: multer, disk vs memory storage, file type validation, sharp image resizing, S3 upload
- **A6.2** — Sending email: Nodemailer, SendGrid, HTML templates with Handlebars, Bull email queue
- **A6.3** — Third-party APIs: axios with interceptors, retry logic, timeout, webhook handling
- **A6.4** — Payment integration: Stripe checkout, webhooks, idempotency keys, subscription billing

### Projects
- **P6.1** — File upload service: image upload to S3, thumbnail generation, signed URLs, metadata in DB
- **P6.2** — E-commerce with payments: product catalog, cart, Stripe checkout, order email, webhook

---

## Phase 7 — Testing

**Goal:** Write comprehensive automated tests at unit, integration, and end-to-end levels.

### Assignments
- **A7.1** — Jest fundamentals: describe/it/expect, matchers, beforeEach/afterEach, async tests, coverage
- **A7.2** — Supertest for API testing: test HTTP endpoints, setup/teardown, in-memory MongoDB
- **A7.3** — Unit testing with mocks: jest.mock(), jest.fn(), spyOn, mock modules, mock DB calls
- **A7.4** — Integration and e2e tests: real database, test factories, test isolation, CI test setup

### Projects
- **P7.1** — Fully tested REST API: 85%+ coverage, unit + integration + e2e, CI pipeline
- **P7.2** — TDD blog API: write tests first, implement to pass, refactor, coverage report

---

## Phase 8 — Deployment & DevOps

**Goal:** Containerize Express apps and build CI/CD pipelines for reliable deployments.

### Assignments
- **A8.1** — Docker for Node.js: multi-stage Dockerfile, non-root user, docker-compose with mongo/redis
- **A8.2** — Environment configuration: dotenv, per-environment config, secrets, 12-factor app
- **A8.3** — GitHub Actions: lint, test, build Docker image, push to registry, deploy
- **A8.4** — Monitoring & health checks: /health endpoint, /metrics for Prometheus, SIGTERM graceful shutdown

### Projects
- **P8.1** — Dockerized API with CI/CD: Docker image, GitHub Actions pipeline, deploy to Railway/Render
- **P8.2** — Production-grade API: graceful shutdown, health checks, structured logging, metrics, rate limiting

---

## Phase 9 — Advanced Patterns

**Goal:** Add real-time features, GraphQL, microservices architecture, and complete API documentation.

### Assignments
- **A9.1** — WebSockets with Socket.io: server setup, events, rooms, namespaces, auth middleware, Redis adapter
- **A9.2** — GraphQL with Apollo Server: schema, resolvers, context, DataLoader, auth, subscriptions
- **A9.3** — Microservices basics: service decomposition, inter-service HTTP, Bull/RabbitMQ message queues
- **A9.4** — API versioning & documentation: Swagger with swagger-jsdoc, versioned routes, deprecation

### Projects
- **P9.1** — Real-time chat app: Socket.io rooms, message history, online users, typing indicators
- **P9.2** — Microservices order system: user/product/order/notification services with Bull queues
- **P9.3** — Full-featured REST API: versioning, OpenAPI docs, security hardening, monitoring, Docker, CI/CD

---

## Completion Checklist

- [ ] Phase 1 complete (4 assignments + 2 projects)
- [ ] Phase 2 complete (4 assignments + 2 projects)
- [ ] Phase 3 complete (4 assignments + 2 projects)
- [ ] Phase 4 complete (4 assignments + 2 projects)
- [ ] Phase 5 complete (4 assignments + 2 projects)
- [ ] Phase 6 complete (4 assignments + 2 projects)
- [ ] Phase 7 complete (4 assignments + 2 projects)
- [ ] Phase 8 complete (4 assignments + 2 projects)
- [ ] Phase 9 complete (4 assignments + 3 projects)

---

## Recommended Timeline

| Phase | Suggested Duration |
|-------|--------------------|
| 1 | 1 week |
| 2 | 1.5 weeks |
| 3 | 1.5 weeks |
| 4 | 1 week |
| 5 | 1 week |
| 6 | 1 week |
| 7 | 1.5 weeks |
| 8 | 1 week |
| 9 | 2 weeks |
| **Total** | **~12 weeks** |
