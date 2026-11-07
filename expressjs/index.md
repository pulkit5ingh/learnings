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
- **[A1.1](A1.1-nodejs-core-internals.md)** — Node.js core: event loop, libuv, process, Buffer, streams, EventEmitter, path/fs/os modules
- **[A1.2](A1.2-express-basics.md)** — Express basics: app setup, routing (GET/POST/PUT/DELETE/PATCH), route params, query strings, req/res objects
- **[A1.3](A1.3-middleware-deep-dive.md)** — Middleware deep dive: middleware chain, custom middleware, error middleware (4-arg), third-party middleware
- **[A1.4](A1.4-routing-organization.md)** — Routing organization: express.Router(), controller pattern, nested routers, route parameters

### Projects
- **[P1.1](P1.1-recipe-api.md)** — Recipe API with CRUD operations, in-memory store, organized routes, input validation
- **[P1.2](P1.2-notes-api.md)** — Notes API with tags, search functionality, full middleware stack

---

## Phase 2 — Database Integration

**Goal:** Connect Express to multiple databases using both ORMs and raw drivers.

### Assignments
- **[A2.1](A2.1-mongodb-with-mongoose.md)** — MongoDB with Mongoose: Schema, Model, CRUD, virtuals, methods/statics, toJSON
- **[A2.2](A2.2-postgresql-with-sequelize.md)** — PostgreSQL with Sequelize: Model.define, associations, migrations, seeders
- **[A2.3](A2.3-raw-sql-with-node-postgres-pg.md)** — Raw SQL with node-postgres (pg): parameterized queries, transactions, connection pool
- **[A2.4](A2.4-prisma-orm.md)** — Prisma ORM: schema.prisma, prisma migrate, CRUD operations, relations, raw queries

### Projects
- **[P2.1](P2.1-blog-api-with-mongodb.md)** — Blog API with MongoDB: posts, comments, tags, Mongoose models, pagination, population
- **[P2.2](P2.2-e-commerce-product-catalog-with-postgresql.md)** — E-commerce product catalog with PostgreSQL: products, categories, Sequelize associations, migrations

---

## Phase 3 — Authentication & Security

**Goal:** Implement secure authentication flows and harden APIs against common attacks.

### Assignments
- **[A3.1](A3.1-jwt-authentication.md)** — JWT authentication: sign/verify, access + refresh tokens, token rotation, Redis blacklisting
- **[A3.2](A3.2-passport-js-strategies.md)** — Passport.js strategies: Local, JWT, Google OAuth2, session vs stateless
- **[A3.3](A3.3-security-hardening.md)** — Security hardening: helmet, rate limiting, CORS, CSRF, SQL injection prevention, input sanitization
- **[A3.4](A3.4-role-based-authorization.md)** — Role-based authorization: roles/permissions design, middleware guards, resource ownership checks

### Projects
- **[P3.1](P3.1-auth-api.md)** — Auth API: register/login/logout/refresh, email verification, password reset, JWT, Redis blacklist
- **[P3.2](P3.2-multi-role-application.md)** — Multi-role application: admin/moderator/user roles, protected routes per role, audit log

---

## Phase 4 — Validation & Error Handling

**Goal:** Build robust APIs with structured validation, consistent error responses, and production-level logging.

### Assignments
- **[A4.1](A4.1-joi-validation.md)** — Joi validation: schema validation, custom messages, nested objects, conditional validation
- **[A4.2](A4.2-express-validator.md)** — express-validator: check(), body(), validationResult(), custom validators, middleware chains
- **[A4.3](A4.3-global-error-handling.md)** — Global error handling: AppError class, async error wrapper, operational vs programmer errors
- **[A4.4](A4.4-logging-with-winston.md)** — Logging: winston setup, log levels, file transport, JSON logs, request logging, correlation IDs

### Projects
- **[P4.1](P4.1-validated-crud-api.md)** — Validated CRUD API: all endpoints validated with Joi, RFC 7807 error responses, structured logging
- **[P4.2](P4.2-error-monitoring-setup.md)** — Error monitoring setup: winston + Sentry, error tracking, alert on 5xx errors, request ID propagation

---

## Phase 5 — Performance & Caching

**Goal:** Make Express APIs fast and scalable through caching, clustering, and query optimization.

### Assignments
- **[A5.1](A5.1-redis-caching.md)** — Redis caching: ioredis, cache-aside pattern, TTL strategies, caching middleware
- **[A5.2](A5.2-pagination-and-filtering.md)** — Pagination & filtering: cursor pagination, offset pagination, sort/filter, aggregation
- **[A5.3](A5.3-clustering-and-pm2.md)** — Clustering & PM2: Node.js cluster module, PM2 config, worker processes, zero-downtime reload
- **[A5.4](A5.4-query-optimization.md)** — Query optimization: explain(), indexes, MongoDB aggregation pipeline, query profiling

### Projects
- **[P5.1](P5.1-high-traffic-news-api.md)** — High-traffic News API: Redis caching, pagination, sorted articles, index optimization
- **[P5.2](P5.2-pm2-production-setup.md)** — PM2 production setup: cluster mode, ecosystem.config.js, health checks, log management

---

## Phase 6 — File Handling & External Services

**Goal:** Handle file uploads, send emails, consume third-party APIs, and integrate payments.

### Assignments
- **[A6.1](A6.1-file-uploads.md)** — File uploads: multer, disk vs memory storage, file type validation, sharp image resizing, S3 upload
- **[A6.2](A6.2-sending-email.md)** — Sending email: Nodemailer, SendGrid, HTML templates with Handlebars, Bull email queue
- **[A6.3](A6.3-third-party-apis.md)** — Third-party APIs: axios with interceptors, retry logic, timeout, webhook handling
- **[A6.4](A6.4-payment-integration-with-stripe.md)** — Payment integration: Stripe checkout, webhooks, idempotency keys, subscription billing

### Projects
- **[P6.1](P6.1-file-upload-service.md)** — File upload service: image upload to S3, thumbnail generation, signed URLs, metadata in DB
- **[P6.2](P6.2-e-commerce-with-payments.md)** — E-commerce with payments: product catalog, cart, Stripe checkout, order email, webhook

---

## Phase 7 — Testing

**Goal:** Write comprehensive automated tests at unit, integration, and end-to-end levels.

### Assignments
- **[A7.1](A7.1-jest-fundamentals.md)** — Jest fundamentals: describe/it/expect, matchers, beforeEach/afterEach, async tests, coverage
- **[A7.2](A7.2-supertest-for-api-testing.md)** — Supertest for API testing: test HTTP endpoints, setup/teardown, in-memory MongoDB
- **[A7.3](A7.3-unit-testing-with-mocks.md)** — Unit testing with mocks: jest.mock(), jest.fn(), spyOn, mock modules, mock DB calls
- **[A7.4](A7.4-integration-and-e2e-tests.md)** — Integration and e2e tests: real database, test factories, test isolation, CI test setup

### Projects
- **[P7.1](P7.1-fully-tested-rest-api.md)** — Fully tested REST API: 85%+ coverage, unit + integration + e2e, CI pipeline
- **[P7.2](P7.2-tdd-blog-api.md)** — TDD blog API: write tests first, implement to pass, refactor, coverage report

---

## Phase 8 — Deployment & DevOps

**Goal:** Containerize Express apps and build CI/CD pipelines for reliable deployments.

### Assignments
- **[A8.1](A8.1-docker-for-nodejs.md)** — Docker for Node.js: multi-stage Dockerfile, non-root user, docker-compose with mongo/redis
- **[A8.2](A8.2-environment-configuration.md)** — Environment configuration: dotenv, per-environment config, secrets, 12-factor app
- **[A8.3](A8.3-github-actions-ci-cd.md)** — GitHub Actions: lint, test, build Docker image, push to registry, deploy
- **[A8.4](A8.4-monitoring-and-health-checks.md)** — Monitoring & health checks: /health endpoint, /metrics for Prometheus, SIGTERM graceful shutdown

### Projects
- **[P8.1](P8.1-dockerized-api-with-ci-cd.md)** — Dockerized API with CI/CD: Docker image, GitHub Actions pipeline, deploy to Railway/Render
- **[P8.2](P8.2-production-grade-api.md)** — Production-grade API: graceful shutdown, health checks, structured logging, metrics, rate limiting

---

## Phase 9 — Advanced Patterns

**Goal:** Add real-time features, GraphQL, microservices architecture, and complete API documentation.

### Assignments
- **[A9.1](A9.1-websockets-with-socket-io.md)** — WebSockets with Socket.io: server setup, events, rooms, namespaces, auth middleware, Redis adapter
- **[A9.2](A9.2-graphql-with-apollo-server.md)** — GraphQL with Apollo Server: schema, resolvers, context, DataLoader, auth, subscriptions
- **[A9.3](A9.3-microservices-basics.md)** — Microservices basics: service decomposition, inter-service HTTP, Bull/RabbitMQ message queues
- **[A9.4](A9.4-api-versioning-and-swagger-documentation.md)** — API versioning & documentation: Swagger with swagger-jsdoc, versioned routes, deprecation

### Projects
- **[P9.1](P9.1-real-time-chat-app.md)** — Real-time chat app: Socket.io rooms, message history, online users, typing indicators
- **[P9.2](P9.2-microservices-order-system.md)** — Microservices order system: user/product/order/notification services with Bull queues
- **[P9.3](P9.3-full-featured-rest-api.md)** — Full-featured REST API: versioning, OpenAPI docs, security hardening, monitoring, Docker, CI/CD

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
