# Technical Interview Questions (Senior Backend/Fullstack)

*Questions likely to come up for a role like Microspace — Node.js, NestJS, TypeScript, PostgreSQL, AWS, Auth/SSO, SaaS.*

Use these to prepare; add your own answers or bullet points as you revise.

---

## Node.js & TypeScript

- What is the **event loop**? How do `setTimeout`, `Promise`, and `process.nextTick` differ in order of execution?
- **Sync vs async** in Node — when would you use sync APIs, and what are the risks?
- How does **error handling** work in async code? Unhandled promise rejections, try/catch with async/await, error boundaries.
- What are **worker threads** and when would you use them instead of the main thread or a separate process?
- **Streams** — what are readable/writable/transform streams? When would you use streams instead of loading everything in memory?
- **TypeScript:** How do you model strict APIs (e.g. DTOs, request/response types)? Difference between `interface` and `type`?
- How do you handle **environment config** and secrets in a Node app (e.g. env vars, validation)?

*They’re probing:* Fundamentals, async behaviour, performance and scalability, type safety.

---

## NestJS

- How does **dependency injection** work in NestJS? Why use it instead of manual instantiation?
- Explain **modules, controllers, and services** — how do they relate and when do you split into separate modules?
- What are **guards, interceptors, and pipes**? Give one use case for each (e.g. auth guard, logging interceptor, validation pipe).
- How would you implement **request validation** (e.g. with `class-validator` / DTOs) and return consistent error responses?
- How do you structure a **multi-tenant** app in NestJS (e.g. tenant from JWT or header, scoping queries)?
- **Testing:** How do you unit test a service that depends on a repository? How do you mock the repository?

*They’re probing:* Framework depth, structure, testability, multi-tenant awareness.

---

## PostgreSQL & databases

- How do you **design a schema** for a multi-tenant SaaS (shared DB vs schema-per-tenant; pros/cons)?
- **Indexes** — when do you add one? What’s the trade-off? Difference between B-tree, partial index, composite index?
- **Migrations:** How do you manage them (tool, naming, rollback)? How do you handle long-running migrations in production?
- **Transactions:** When do you need a transaction? What are isolation levels and when might you change from default?
- **N+1 queries** — what are they and how do you avoid them (e.g. eager loading, batch loading, JOINs)?
- **Normalisation vs denormalisation** — when would you denormalise for performance?
- How would you **paginate** a large list in PostgreSQL (offset vs cursor/keyset)? Trade-offs.

*They’re probing:* Schema design, performance, migrations, multi-tenant data model.

---

## APIs (REST, design, security)

- How do you **design a REST API** for a resource (e.g. spaces, members)? Status codes, idempotency, versioning.
- **Idempotency** — why is it important for payments or webhooks? How would you implement it?
- **Rate limiting** — why and how? Per-user vs per-tenant; where would you implement it (app vs gateway)?
- How do you **version APIs** (URL vs header)? How do you deprecate old versions?
- **Pagination and filtering** — how do you expose filter/sort params safely (e.g. avoid injection, limit fields)?
- How do you **secure an API** (auth, HTTPS, CORS, input validation, SQL injection prevention)?
- **Webhooks** — how would you design delivery (retries, idempotency keys, signing)?

*They’re probing:* API design, safety, production readiness, integrations.

---

## Authentication & authorisation (including SSO)

- **JWT vs session** — when would you use each? How do you handle token refresh and logout with JWTs?
- How would you implement **Role-Based Access Control (RBAC)**? Where do you enforce it (guard, middleware, service)?
- **SSO:** What are **SAML** and **OIDC**? What’s the high-level flow (e.g. redirect to IdP, callback, user creation/linking)?
- How do you **store and validate** tokens securely? What do you put in the JWT (claims) and what do you avoid?
- **Multi-tenant auth** — how do you ensure a user can only access data for tenants they belong to?
- How would you add **“Login with Google”** or **“Login with Okta”** to an existing email/password app?

*They’re probing:* Auth flows, RBAC, SSO concepts, security and tenant isolation.

---

## AWS & DevOps

- What **AWS services** have you used (EC2, S3, Lambda, RDS, etc.)? Describe one production use case.
- **S3:** How do you secure uploads/downloads (IAM, signed URLs, bucket policy)? When would you use pre-signed URLs?
- **Lambda:** When would you choose Lambda vs a long-running service? Cold starts, timeouts, concurrency.
- How do you **deploy** a Node/NestJS app (e.g. Docker, ECS, EC2, PM2)? How do you do zero-downtime deployments?
- **CI/CD:** What do you run in the pipeline (lint, test, build, deploy)? How do you handle secrets?
- **Observability:** How do you do logging, metrics, and alerts? What would you log for a failing API request?
- **Scaling:** How would you make a stateless API scale horizontally? What about the database?

*They’re probing:* Real AWS experience, deployment, reliability, observability.

---

## System design & architecture

- Design a **multi-tenant SaaS** where each tenant has “spaces” and “members.” How do you model data and APIs?
- How would you design **real-time updates** (e.g. “someone joined the space”) — WebSockets vs polling vs SSE?
- How do you **handle a spike in traffic** (caching, queues, auto-scaling)? Where are the bottlenecks?
- **Event-driven** — when would you use a message queue (e.g. BullMQ, Kafka) instead of direct API calls?
- How would you **break a monolith** into services? How do services communicate and stay consistent?
- **Caching:** Where would you cache (application, Redis, CDN)? Cache invalidation strategy for user-specific data?

*They’re probing:* Architecture, multi-tenant, scalability, trade-offs.

---

## Frontend (React) — for fullstack discussion

- How does **React** render and update the UI (virtual DOM, reconciliation)? When does a component re-render?
- **State:** When do you use local state vs context vs a global store? How do you avoid unnecessary re-renders?
- How do you **fetch data** in React (useEffect, SWR, React Query)? How do you handle loading and errors?
- **SSR vs CSR** — when would you use Next.js SSR vs a SPA? What’s the impact on SEO and performance?
- How do **backend and frontend** agree on types (e.g. shared types, OpenAPI-generated clients)?

*They’re probing:* Enough to collaborate with frontend, not necessarily deep React expertise.

---

## Reliability, performance & security

- How do you make an API **resilient** (retries, timeouts, circuit breakers)? Where would you use a circuit breaker?
- **Performance:** How do you find and fix a slow API (logging, profiling, DB queries, N+1)?
- **Security:** What are common vulnerabilities (injection, XSS, CSRF, broken auth)? How do you mitigate them?
- How do you **test** backend code (unit, integration, e2e)? What do you mock (DB, external APIs)?
- **Secure coding:** How do you validate and sanitise user input? How do you avoid leaking sensitive data in logs or errors?

*They’re probing:* Production mindset, debugging, security awareness, testing.

---

## Behavioural / experience-based (technical)

- Describe a **complex backend feature** you built (e.g. RBAC, payments, bulk upload). What were the challenges?
- Tell me about a time you **improved performance** or **reliability** of a system. What did you measure?
- Describe how you’ve worked **across the stack** or with frontend on an end-to-end feature.
- How do you **balance speed of delivery with long-term maintainability**? Give an example.
- Describe a **technical decision** you made that had trade-offs. What would you do differently?
- How do you **review code**? What do you look for in a PR?

*They’re probing:* Ownership, collaboration, trade-offs, quality.

---

## Quick checklist before the interview

- [ ] Can explain event loop and async in Node in 1–2 minutes.
- [ ] Can draw NestJS request flow (middleware → guard → pipe → controller → service).
- [ ] Can design a simple multi-tenant schema (tenants, users, spaces, members).
- [ ] Can explain JWT flow and one SSO flow (OIDC or SAML at high level).
- [ ] Can describe one production deployment (Docker/AWS/CI) and one debugging story.
- [ ] Have 2–3 project stories ready (e.g. Tripinnov/TravelBaba/Corpusvision) with problem → approach → outcome.
