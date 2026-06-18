# Backend Developer using FastAPI — Full Curriculum

A structured, project-driven curriculum to take you from Python basics to production-ready FastAPI services. Nine phases, 36 assignments, 22 projects.

---

## How to Use This Curriculum

1. Read `HOW_TO_LEARN.md` first.
2. Complete assignments in order within each phase — they build on each other.
3. After the assignments in a phase, build the projects before moving on.
4. Check deliverables honestly. Do not skip tests or Docker steps.

---

## Phase Overview

| Phase | Name | Assignments | Projects |
|-------|------|-------------|----------|
| 1 | Python & FastAPI Fundamentals | A1.1–A1.4 | P1.1–P1.2 |
| 2 | Database with SQLAlchemy & PostgreSQL | A2.1–A2.4 | P2.1–P2.2 |
| 3 | Authentication & Authorization | A3.1–A3.4 | P3.1–P3.2 |
| 4 | Testing | A4.1–A4.4 | P4.1–P4.2 |
| 5 | Performance & Caching | A5.1–A5.4 | P5.1–P5.2 |
| 6 | API Design & Documentation | A6.1–A6.4 | P6.1–P6.2 |
| 7 | Microservices & Messaging | A7.1–A7.4 | P7.1–P7.2 |
| 8 | Deployment & CI/CD | A8.1–A8.4 | P8.1–P8.2 |
| 9 | Advanced Features | A9.1–A9.4 | P9.1–P9.3 |

---

## Phase 1 — Python & FastAPI Fundamentals

**Goal:** Write idiomatic async Python and stand up a fully typed FastAPI application with proper request/response models.

### Assignments
- **A1.1** — Python async/await: asyncio, coroutines, event loop, `async def`, `await`, `asyncio.gather`, Task
- **A1.2** — FastAPI basics: app creation, path/query params, request body with Pydantic, response models, status codes
- **A1.3** — Pydantic v2: BaseModel, field validators, model_validator, computed_fields, model_config, nested models, discriminated unions
- **A1.4** — Dependency injection: `Depends()`, shared deps, class-based deps, sub-dependencies, override in tests

### Projects
- **P1.1** — URL Shortener API (FastAPI + in-memory store, full CRUD)
- **P1.2** — Personal Finance Tracker API (categories, transactions, monthly summaries)

---

## Phase 2 — Database with SQLAlchemy & PostgreSQL

**Goal:** Persist data with SQLAlchemy 2.0, write async queries, and manage schema with Alembic.

### Assignments
- **A2.1** — SQLAlchemy Core: engine, connection, MetaData, Table, Column, insert/select/update/delete, transactions
- **A2.2** — SQLAlchemy ORM: DeclarativeBase, mapped_column, relationships, session, lazy/eager loading
- **A2.3** — Alembic migrations: init, env.py, autogenerate, upgrade/downgrade, data migrations
- **A2.4** — Async SQLAlchemy: AsyncEngine, AsyncSession, async_sessionmaker, repository pattern

### Projects
- **P2.1** — Blog API (posts, tags, authors, async SQLAlchemy, Alembic)
- **P2.2** — Inventory Management System (products, categories, suppliers, stock levels)

---

## Phase 3 — Authentication & Authorization

**Goal:** Implement secure JWT auth, refresh token rotation, RBAC, and API key schemes.

### Assignments
- **A3.1** — Password hashing & JWT: bcrypt, python-jose, JWT structure, sign/verify, access/refresh tokens
- **A3.2** — OAuth2 with FastAPI: OAuth2PasswordBearer, token endpoint, protected routes
- **A3.3** — Role-based access control: roles table, permission checking, Depends() guards
- **A3.4** — API keys & OAuth2 scopes: API key header auth, multiple security schemes, scope enforcement

### Projects
- **P3.1** — Auth service (registration, login, refresh token rotation, logout, email verification)
- **P3.2** — Multi-tenant SaaS API (organizations, members, roles per org, JWT with org_id)

---

## Phase 4 — Testing

**Goal:** Write a comprehensive test suite with unit, integration, and end-to-end tests at 90%+ coverage.

### Assignments
- **A4.1** — pytest fundamentals: fixtures, parametrize, markers, conftest.py, monkeypatch
- **A4.2** — FastAPI testing: TestClient, AsyncClient, override_dependency, test database
- **A4.3** — Mocking & faking: unittest.mock, MagicMock, patch, pytest-mock, factory_boy
- **A4.4** — Coverage & test organization: pytest-cov, reports, unit/integration/e2e structure, CI

### Projects
- **P4.1** — Fully tested Task API (90%+ coverage, unit + integration + e2e)
- **P4.2** — Test suite for auth service from Phase 3

---

## Phase 5 — Performance & Caching

**Goal:** Handle high load with caching, background workers, async optimization, and rate limiting.

### Assignments
- **A5.1** — Background tasks: FastAPI BackgroundTasks, Celery + Redis, Celery beat
- **A5.2** — Redis caching: redis-py async, cache-aside, TTL, cache invalidation decorator
- **A5.3** — Async optimization: connection pooling, N+1 detection, selectin loading, profiling
- **A5.4** — Rate limiting & pagination: slowapi, cursor-based + offset pagination, Link headers

### Projects
- **P5.1** — Image processing service (upload, Celery resize, Redis job status, webhook)
- **P5.2** — High-performance Products API (Redis cache, pagination, rate limiting, load test)

---

## Phase 6 — API Design & Documentation

**Goal:** Design clean, versioned, well-documented REST APIs following industry standards.

### Assignments
- **A6.1** — OpenAPI & Swagger: tags, response examples, deprecated, custom schemas
- **A6.2** — API versioning: URL versioning, header versioning, router prefixes
- **A6.3** — Input validation & error handling: HTTPException, custom handlers, RFC 7807
- **A6.4** — CORS, middleware & security headers: CORSMiddleware, custom middleware, security headers

### Projects
- **P6.1** — E-commerce Product Catalog API (versioned, comprehensive docs, custom errors)
- **P6.2** — Public API with SDK (Python client SDK generated from OpenAPI spec)

---

## Phase 7 — Microservices & Messaging

**Goal:** Decompose a monolith, communicate via HTTP and message brokers, implement event-driven patterns.

### Assignments
- **A7.1** — HTTP microservices: httpx async, circuit breaker, timeout/retry
- **A7.2** — RabbitMQ with aio-pika: producer/consumer, exchanges, DLQ
- **A7.3** — Event-driven patterns: domain events, idempotency keys, at-least-once delivery
- **A7.4** — gRPC with Python: proto files, stubs, server/client, streaming

### Projects
- **P7.1** — Order processing system (order-service → RabbitMQ → inventory + notification, saga)
- **P7.2** — Notification service (consume events, email/SMS, template engine, retry)

---

## Phase 8 — Deployment & CI/CD

**Goal:** Containerize, test, and deploy FastAPI services with automated pipelines.

### Assignments
- **A8.1** — Dockerfile for FastAPI: multi-stage build, non-root user, health check
- **A8.2** — Docker Compose: app + postgres + redis + celery + flower, volumes, env files
- **A8.3** — GitHub Actions CI/CD: lint, type check, test with postgres service, push to GHCR
- **A8.4** — Environment management: pydantic-settings, Settings class, secrets

### Projects
- **P8.1** — Production-ready FastAPI deploy (Docker, GitHub Actions, Railway/Fly.io)
- **P8.2** — Full CI/CD pipeline (PR checks, staging, production on tag)

---

## Phase 9 — Advanced Features

**Goal:** Add real-time communication, GraphQL, file handling, and full observability.

### Assignments
- **A9.1** — WebSockets: FastAPI WebSocket, connection manager, broadcast, auth over WS
- **A9.2** — GraphQL with Strawberry: schema-first, Query/Mutation/Subscription, DataLoaders
- **A9.3** — File uploads & streaming: UploadFile, chunked uploads, S3, streaming responses
- **A9.4** — Observability: structlog, Prometheus metrics, OpenTelemetry tracing

### Projects
- **P9.1** — Real-time chat API (WebSocket rooms, Redis pub/sub, presence)
- **P9.2** — GraphQL API (full blog schema, subscriptions, DataLoader, auth)
- **P9.3** — Production observability setup (logs, metrics, traces, Grafana)

---

## Technology Stack

| Category | Tools |
|----------|-------|
| Language | Python 3.11+ |
| Framework | FastAPI 0.110+ |
| Validation | Pydantic v2 |
| ORM | SQLAlchemy 2.0 (async) |
| Migrations | Alembic |
| Database | PostgreSQL 16 |
| Cache/Broker | Redis 7 |
| Task Queue | Celery 5 |
| Messaging | RabbitMQ 3 (aio-pika) |
| RPC | gRPC (grpcio) |
| GraphQL | Strawberry |
| Testing | pytest, httpx, factory_boy |
| Auth | python-jose, passlib (bcrypt) |
| HTTP Client | httpx |
| Server | Uvicorn + Gunicorn |
| Containers | Docker, Docker Compose |
| CI/CD | GitHub Actions |
| Observability | structlog, Prometheus, OpenTelemetry |

---

## Estimated Time

| Phase | Estimated Hours |
|-------|----------------|
| 1 | 15–20 hrs |
| 2 | 20–25 hrs |
| 3 | 20–25 hrs |
| 4 | 15–20 hrs |
| 5 | 20–25 hrs |
| 6 | 15–18 hrs |
| 7 | 25–30 hrs |
| 8 | 15–20 hrs |
| 9 | 25–30 hrs |
| **Total** | **170–213 hrs** |

At 2 hours/day, this is a 3–4 month curriculum.
