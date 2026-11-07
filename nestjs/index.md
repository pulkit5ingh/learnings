# NestJS Backend Developer — Full Learning Roadmap

**Track:** Backend Development  
**Framework:** NestJS v10+  
**Duration:** ~12–16 weeks (part-time), ~6–8 weeks (full-time)  
**Total Files:** 60 (index + guide + 36 assignments + 22 projects)

---

## What You Will Be Able to Build

By the end of this curriculum you will be able to:

- Design and build production-grade REST APIs with NestJS
- Model complex relational data with TypeORM and Prisma
- Implement JWT + OAuth2 authentication with role-based access control
- Write validated, typed request/response pipelines using class-validator
- Set up multi-environment configuration with secret validation
- Build async systems with Bull queues, Redis caching, and cron jobs
- Write unit, integration, and end-to-end test suites with 85%+ coverage
- Decompose a monolith into microservices using TCP, Redis, RabbitMQ, or gRPC
- Expose real-time APIs via WebSockets and GraphQL
- Ship production apps with Swagger docs, structured logs, Prometheus metrics, and health checks

---

## Prerequisites

| Skill | Level Needed |
|---|---|
| TypeScript | Comfortable with classes, generics, decorators |
| Node.js | Express or Fastify background |
| SQL / PostgreSQL | Basic queries, joins, indexes |
| Docker | Able to run `docker-compose up` |
| Git | Daily driver |

---

## Phases Overview

| Phase | Topic | Assignments | Projects |
|---|---|---|---|
| 1 | NestJS Fundamentals | A1.1 – A1.4 | P1.1 – P1.2 |
| 2 | Database with TypeORM & Prisma | A2.1 – A2.4 | P2.1 – P2.2 |
| 3 | Authentication & Guards | A3.1 – A3.4 | P3.1 – P3.2 |
| 4 | Pipes, Interceptors & Filters | A4.1 – A4.4 | P4.1 – P4.2 |
| 5 | Configuration & Environment | A5.1 – A5.4 | P5.1 – P5.2 |
| 6 | Caching, Queues & Performance | A6.1 – A6.4 | P6.1 – P6.2 |
| 7 | Testing | A7.1 – A7.4 | P7.1 – P7.2 |
| 8 | Microservices | A8.1 – A8.4 | P8.1 – P8.2 |
| 9 | Advanced & Production | A9.1 – A9.4 | P9.1 – P9.3 |

---

## Phase 1 — NestJS Fundamentals

**Goal:** Understand the NestJS module system, dependency injection, and request lifecycle.

| File | Title |
|---|---|
| [A1.1](A1.1-nestjs-architecture.md) | NestJS architecture — modules, controllers, providers, DI container, decorators |
| [A1.2](A1.2-controllers-and-routing.md) | Controllers & routing — HTTP verbs, params, query, body, response shaping |
| [A1.3](A1.3-providers-and-services.md) | Providers & services — Injectable, constructor DI, custom providers, scope |
| [A1.4](A1.4-modules.md) | Modules — feature/shared/global/dynamic modules, forRoot pattern |
| [P1.1](P1.1-tasks-api.md) | Tasks API — full CRUD with status, priority, due date |
| [P1.2](P1.2-library-management-api.md) | Library management API — books, authors, members, borrow records |

---

## Phase 2 — Database with TypeORM & Prisma

**Goal:** Persist data using two popular ORM choices, manage migrations, and model relations.

| File | Title |
|---|---|
| [A2.1](A2.1-typeorm-basics.md) | TypeORM basics — entities, columns, repositories, find options |
| [A2.2](A2.2-typeorm-relations.md) | TypeORM relations — OneToMany, ManyToOne, ManyToMany, QueryBuilder |
| [A2.3](A2.3-database-migrations.md) | Database migrations — generate, run, revert, seeders |
| [A2.4](A2.4-prisma-in-nestjs.md) | Prisma in NestJS — schema.prisma, PrismaService, transactions, select/include |
| [P2.1](P2.1-blog-platform-with-typeorm.md) | Blog platform with TypeORM — Post, Category, Tag, Comment, User |
| [P2.2](P2.2-product-catalog-with-prisma.md) | Product catalog with Prisma — products, variants, categories, inventory |

---

## Phase 3 — Authentication & Guards

**Goal:** Secure endpoints with JWT, refresh tokens, roles, and OAuth2.

| File | Title |
|---|---|
| [A3.1](A3.1-guards.md) | Guards — CanActivate, ExecutionContext, Reflector, global vs route-level |
| [A3.2](A3.2-jwt-authentication.md) | JWT authentication — @nestjs/passport, JwtStrategy, access + refresh tokens |
| [A3.3](A3.3-roles-and-permissions.md) | Roles & permissions — @Roles() decorator, RolesGuard, CASL |
| [A3.4](A3.4-oauth2-and-sessions.md) | OAuth2 & sessions — Google/GitHub strategy, express-session |
| [P3.1](P3.1-complete-auth-system.md) | Complete auth system — register/login/logout, email verify, password reset |
| [P3.2](P3.2-permission-system-rbac-abac.md) | Permission system — RBAC + ABAC hybrid, ownership checks |

---

## Phase 4 — Pipes, Interceptors & Filters

**Goal:** Build a clean, validated request/response pipeline with consistent error handling.

| File | Title |
|---|---|
| [A4.1](A4.1-pipes.md) | Pipes — ValidationPipe, ParseIntPipe, custom pipes, global pipes |
| [A4.2](A4.2-interceptors.md) | Interceptors — logging, response transform, caching, timeout |
| [A4.3](A4.3-exception-filters.md) | Exception filters — @Catch, HttpException, custom exceptions, global filter |
| [A4.4](A4.4-class-validator-and-class-transformer.md) | class-validator & class-transformer — decorators, nested validation, plainToInstance |
| [P4.1](P4.1-validated-api-with-transforms.md) | Validated API with transforms — RFC 7807 errors, DTO validation throughout |
| [P4.2](P4.2-audit-logging-interceptor.md) | Audit logging interceptor — log every request/response to database |

---

## Phase 5 — Configuration & Environment

**Goal:** Manage configuration safely across dev/staging/production.

| File | Title |
|---|---|
| [A5.1](A5.1-configmodule.md) | ConfigModule — .env files, ConfigService, Joi validation, namespaced config |
| [A5.2](A5.2-database-config-patterns.md) | Database config patterns — TypeOrmModule.forRootAsync, async config |
| [A5.3](A5.3-secrets-management.md) | Secrets management — env validation, vault integration basics |
| [A5.4](A5.4-feature-flags-and-app-settings.md) | Feature flags & app settings — database-driven settings, environment behavior |
| [P5.1](P5.1-multi-environment-api.md) | Multi-environment API — dev/staging/prod configs, typed ConfigService |
| [P5.2](P5.2-settings-service.md) | Settings service — admin-configurable settings, Redis cache, change events |

---

## Phase 6 — Caching, Queues & Performance

**Goal:** Add async processing, caching, scheduled tasks, and rate limiting.

| File | Title |
|---|---|
| [A6.1](A6.1-caching.md) | Caching — cache-manager, Redis store, @CacheKey, invalidation |
| [A6.2](A6.2-bull-queues.md) | Bull queues — @Processor/@Process, retry, progress, concurrency |
| [A6.3](A6.3-task-scheduling.md) | Task scheduling — @Cron, @Interval, @Timeout, dynamic cron |
| [A6.4](A6.4-rate-limiting.md) | Rate limiting — @nestjs/throttler, ThrottlerGuard, Redis storage |
| [P6.1](P6.1-email-notification-system.md) | Email notification system — Bull queue, Handlebars templates, DLQ |
| [P6.2](P6.2-cache-heavy-product-api.md) | Cache-heavy product API — Redis caching, invalidation, rate limiting |

---

## Phase 7 — Testing

**Goal:** Write comprehensive test suites that catch regressions early.

| File | Title |
|---|---|
| [A7.1](A7.1-unit-testing.md) | Unit testing — jest, Test.createTestingModule, mock providers |
| [A7.2](A7.2-controller-testing.md) | Controller testing — mock services, supertest, HTTP assertions |
| [A7.3](A7.3-e2e-testing.md) | e2e testing — real database, test app bootstrap, cleanup |
| [A7.4](A7.4-test-utilities.md) | Test utilities — factories, seeding, jest config, coverage |
| [P7.1](P7.1-fully-tested-nestjs-api.md) | Fully tested NestJS API — 85%+ coverage, CI pipeline |
| [P7.2](P7.2-tdd-nestjs-feature.md) | Test-driven NestJS feature — red-green-refactor cycle |

---

## Phase 8 — Microservices

**Goal:** Decompose services and communicate via message-passing transports.

| File | Title |
|---|---|
| [A8.1](A8.1-nestjs-microservices.md) | NestJS microservices — ClientProxy, @MessagePattern, TCP transport |
| [A8.2](A8.2-redis-transport.md) | Redis transport — pub/sub, service-to-service communication |
| [A8.3](A8.3-rabbitmq-transport.md) | RabbitMQ transport — durable queues, dead letter exchange |
| [A8.4](A8.4-grpc-transport.md) | gRPC transport — proto files, @GrpcMethod, streaming |
| [P8.1](P8.1-microservices-e-commerce.md) | Microservices e-commerce — user-ms, product-ms, order-ms via Redis |
| [P8.2](P8.2-event-driven-notification-system.md) | Event-driven notification system — RabbitMQ domain events, email/SMS |

---

## Phase 9 — Advanced & Production

**Goal:** Ship production-ready NestJS apps with real-time APIs, GraphQL, and full observability.

| File | Title |
|---|---|
| [A9.1](A9.1-websockets.md) | WebSockets — @WebSocketGateway, rooms, WS auth, scaling |
| [A9.2](A9.2-graphql-with-nestjs.md) | GraphQL with NestJS — @Resolver, ObjectType, DataLoader |
| [A9.3](A9.3-openapi-swagger.md) | OpenAPI/Swagger — @ApiProperty, bearer auth, SDK generation |
| [A9.4](A9.4-observability.md) | Observability — winston, Prometheus, @nestjs/terminus, OpenTelemetry |
| [P9.1](P9.1-real-time-collaboration-api.md) | Real-time collaboration API — WebSocket rooms, presence, Redis pub/sub |
| [P9.2](P9.2-graphql-api-gateway.md) | GraphQL API gateway — queries/mutations/subscriptions, DataLoader |
| [P9.3](P9.3-production-nestjs-monolith.md) | Production NestJS monolith — full observability, Docker, CI/CD |

---

## Recommended Weekly Schedule (12 weeks)

| Week | Focus |
|---|---|
| 1 | Phase 1 — all assignments + P1.1 |
| 2 | Phase 1 — P1.2 + Phase 2 A2.1–A2.2 |
| 3 | Phase 2 — A2.3–A2.4 + P2.1 |
| 4 | Phase 2 — P2.2 + Phase 3 A3.1–A3.2 |
| 5 | Phase 3 — A3.3–A3.4 + P3.1 |
| 6 | Phase 3 — P3.2 + Phase 4 all |
| 7 | Phase 4 projects + Phase 5 all |
| 8 | Phase 5 projects + Phase 6 all |
| 9 | Phase 6 projects + Phase 7 all |
| 10 | Phase 7 projects + Phase 8 A8.1–A8.2 |
| 11 | Phase 8 — A8.3–A8.4 + P8.1 |
| 12 | Phase 8 P8.2 + Phase 9 all |

---

## Key Tools & Versions

| Tool | Version |
|---|---|
| NestJS | v10.x |
| TypeORM | 0.3.x |
| Prisma | 5.x |
| class-validator | 0.14.x |
| class-transformer | 0.5.x |
| @nestjs/passport | 10.x |
| @nestjs/jwt | 10.x |
| @nestjs/bull | 10.x |
| @nestjs/schedule | 4.x |
| @nestjs/throttler | 5.x |
| @nestjs/microservices | 10.x |
| @nestjs/graphql | 12.x |
| @nestjs/swagger | 7.x |
| @nestjs/terminus | 10.x |
