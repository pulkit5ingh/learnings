# SaaS Technical Q&A — Must-Know Before Building

*Questions and answers on SaaS fundamentals, system design, database design, auth, and other critical topics. Use this before building or discussing a SaaS product.*

---

## 1. SaaS fundamentals

### Q: What makes an application “SaaS” and what are the core technical implications?

**A:** SaaS means the software is **hosted by the provider** and **consumed over the internet** (browser or app), usually via **subscription**. Core technical implications:

- **Multi-tenancy:** One deployment serves many customers (tenants). You must isolate data and config per tenant so Customer A never sees Customer B’s data.
- **No “install on your server”:** You own hosting, scaling, backups, and updates. You need robust deployment, monitoring, and incident response.
- **Subscription and usage:** Billing is recurring and often tied to usage (seats, API calls, storage). You need metering, quotas, and sometimes usage-based pricing logic.
- **APIs and integrations:** B2B customers expect APIs, webhooks, and sometimes SSO. Design for API-first and extensibility from day one.
- **Security and compliance:** You hold customer data; you need encryption, access control, audit logs, and often compliance (GDPR, SOC 2, etc.).

---

### Q: What is multi-tenancy and what are the main approaches?

**A:** **Multi-tenancy** = one application instance serves multiple customers (tenants), with logical or physical separation so data doesn’t leak.

**Main approaches:**

| Approach | Description | Pros | Cons |
|----------|-------------|------|------|
| **Shared DB, tenant column** | One database; every table has a `tenant_id` (or `organization_id`). Every query is scoped by tenant. | Simple, one codebase, easy to add tenants. | Must never forget to filter by tenant; risk of cross-tenant leak if a query misses the filter. |
| **Schema per tenant** | One DB, one schema per tenant (e.g. PostgreSQL schemas). | Strong isolation, can restore one tenant. | More schemas to manage; migrations run per schema; harder to do cross-tenant analytics. |
| **Database per tenant** | Separate database per tenant. | Strong isolation, easy to move/restore a tenant. | Operational overhead, migrations × N, cost. |
| **Instance per tenant** | Separate app instance (and often DB) per tenant. | Maximum isolation. | Highest cost and ops; usually only for very large enterprises. |

**Most common for B2B SaaS:** Shared DB with a **tenant_id (or org_id) on every tenant-scoped table**. Enforce it in middleware/guards (set tenant from JWT or context) and in the data layer (e.g. global query scope, or always passing tenant_id into repos).

---

### Q: Why is tenant isolation critical and how do you enforce it?

**A:** **Tenant isolation** means Tenant A cannot read or modify Tenant B’s data. Without it you have a critical security and compliance failure.

**How to enforce it:**

1. **Identity:** Every request is tied to a user who belongs to one or more tenants. Resolve tenant from auth (e.g. JWT claim `org_id`, or session).
2. **Authorization:** Before any read/write, check that the user has access to that tenant and that the resource (e.g. space, document) belongs to that tenant.
3. **Data layer:** Never query tenant-scoped tables without a tenant filter. Use a **tenant context** (e.g. from middleware) and:
   - Pass `tenantId` into every repository method, or
   - Use a global scope / wrapper that injects `WHERE tenant_id = :tenantId`, or
   - Use row-level security (RLS) in PostgreSQL so the DB enforces it.
4. **Code review and tests:** Explicit tests for “user from tenant A cannot access tenant B’s resource.” Consider integration tests that try to bypass tenant checks.
5. **Audit:** Log access and changes with tenant and user so you can detect and investigate leaks.

**Common bug:** A query that joins tables but forgets to add `tenant_id` on one of them, returning cross-tenant data. Always double-check JOINs and filters.

---

## 2. System design for SaaS

### Q: How would you design a high-level architecture for a B2B SaaS (e.g. “spaces” and “members”)?

**A:** High-level building blocks:

- **API layer:** Stateless REST (and/or GraphQL) API. Auth (JWT or session) and tenant context resolved per request. Rate limiting and quotas per tenant.
- **Application layer:** Business logic in services; tenant_id flows from auth into all operations. No global mutable state so you can scale horizontally.
- **Data layer:** PostgreSQL (or similar) with tenant_id on tenant-scoped tables; connection pooling (e.g. PgBouncer). Optionally read replicas for heavy read workloads.
- **Cache:** Redis for sessions, rate limits, and hot data (e.g. tenant config, feature flags). Cache keys include tenant_id so data is never shared across tenants.
- **Async jobs:** Queue (e.g. BullMQ, SQS) for email, notifications, reports, webhooks. Job payloads include tenant_id so workers scope work correctly.
- **File storage:** S3 (or equivalent) with path prefix or bucket strategy per tenant; access via signed URLs or through your API so you enforce auth.
- **Real-time (if needed):** WebSockets or SSE; user is authenticated and associated with a tenant; broadcast only within that tenant’s context.

**Principles:** Stateless APIs, tenant in every context, horizontal scaling, queue for long-running work, and observability (logs, metrics, traces with tenant_id).

---

### Q: How do you handle scaling for a growing SaaS?

**A:**

- **Application:** Stateless app servers behind a load balancer; scale out by adding instances. No sticky sessions if you can (use external session store like Redis).
- **Database:** Vertical scaling first; then read replicas for read-heavy workloads; connection pooling to avoid exhausting connections. For very large scale: sharding by tenant_id (e.g. tenant_id % N).
- **Caching:** Cache hot data (e.g. tenant settings, user permissions) in Redis; invalidate on update. Use cache-aside; consider CDN for static assets.
- **Async:** Offload email, notifications, exports, webhooks to queues so API stays fast and failures can retry.
- **Rate limiting and quotas:** Per-tenant limits so one tenant can’t starve others (“noisy neighbour”). Enforce at API gateway or in app layer with Redis counters.
- **Monitoring:** Metrics and logs tagged by tenant where useful; alert on error rate, latency, queue depth. Use for capacity planning and incident response.

---

### Q: How would you design real-time features (e.g. “someone joined the space”) in a SaaS?

**A:** Options:

1. **WebSockets:** Persistent connection; server pushes events. On connect, authenticate user and resolve tenant; subscribe to channels/rooms scoped by tenant (e.g. `space:123` where space belongs to tenant). Only emit to users who are allowed to see that space. Use a scalable WS layer (e.g. Socket.io with Redis adapter, or managed service) if you have many connections.
2. **Server-Sent Events (SSE):** Simpler than WebSockets; one-way server→client. Good for live updates to a single page; authenticate and scope by tenant/resource.
3. **Long polling:** Client repeatedly polls; server holds request until there’s an update or timeout. Simpler but more overhead than WebSockets/SSE.

**Design:** Emit only after checking authorization (user has access to this tenant/space). Include tenant_id in event payloads for logging and debugging. For “someone joined,” publish an event to the space’s channel after the join is committed to the DB.

---

## 3. Database design for SaaS

### Q: How do you design schemas for multi-tenant SaaS? Where do you put tenant_id?

**A:**

- **Tenant-scoped tables:** Every table that holds tenant-specific data has a `tenant_id` (or `organization_id`) column. It’s the first column in composite primary keys and in (tenant_id, …) indexes so all tenant queries are efficient.
- **Core tables:** e.g. `tenants` (id, name, slug, settings, plan, …), `users` (id, email, …), `tenant_members` (tenant_id, user_id, role) linking users to tenants and roles.
- **Resource tables:** e.g. `spaces` (id, tenant_id, name, …), `documents` (id, tenant_id, space_id, …). Foreign keys: space → tenant, document → space (and tenant).
- **Global tables (no tenant_id):** e.g. `users` if the same user can belong to multiple tenants; then `tenant_members` is the join. If user is global, never put tenant_id on `users`; only on membership and resources.
- **Indexes:** Always have (tenant_id, …) for tenant-scoped queries. Avoid indexes that don’t start with tenant_id for tenant-scoped tables, or you’ll do full table scans per tenant.

**Rule of thumb:** If a row “belongs to” a customer (organization), it gets tenant_id. If it’s global (e.g. list of countries), it doesn’t.

---

### Q: How do you avoid N+1 queries in a SaaS API (e.g. list spaces with member count)?

**A:** N+1 = one query for the list + one query per item (e.g. per space to get member count) = 1 + N queries.

**Ways to fix:**

1. **Eager loading / JOIN:** Single query that joins spaces and aggregates (e.g. `SELECT s.*, COUNT(m.id) FROM spaces s LEFT JOIN space_members m ON m.space_id = s.id WHERE s.tenant_id = :tid GROUP BY s.id`).
2. **Batch loading:** Fetch all space IDs from the list query; then one query `SELECT space_id, COUNT(*) FROM space_members WHERE space_id IN (:ids) GROUP BY space_id` and map results back to spaces in code.
3. **ORM:** Use your ORM’s eager load (e.g. `include`, `with`) so it generates a single or batched query instead of one per row.

Always scope by tenant_id in these queries. Prefer one or two queries over N+1.

---

### Q: How do you handle database migrations in a multi-tenant SaaS?

**A:**

- **Single schema (shared DB, tenant_id):** One migration history; migrations run once and apply to the shared schema. Use a migration tool (e.g. Flyway, Liquibase, or Node migrations like node-pg-migrate). Never skip migrations; run them in CI and as part of deployment.
- **Long-running migrations:** Avoid long-held locks. Prefer additive changes (add column as nullable, backfill in a job, then add constraint). For big backfills, do in batches with a background job and track progress.
- **Schema-per-tenant:** Migrations must run against every tenant schema (or on-demand when tenant is created). More complex; often scripted (loop over schemas and run same migration).
- **Rollback:** Prefer reversible migrations (e.g. up/down). If you only have “up,” document manual rollback steps. Test migrations on staging with production-like data volume.

---

### Q: Offset vs cursor (keyset) pagination — when to use which?

**A:**

- **Offset (`LIMIT n OFFSET m`):** Simple, but slow and unstable on large tables. As offset grows, the DB still has to skip m rows. If data changes between requests, you can skip or duplicate rows.
- **Cursor (keyset):** Use a stable sort key (e.g. `(created_at, id)`) and “WHERE (created_at, id) < (:last_created_at, :last_id) ORDER BY created_at DESC, id DESC LIMIT n”. Fast (index range scan), stable under inserts. Slightly more complex API (client passes cursor, server returns next_cursor).

**For SaaS:** Prefer **cursor pagination** for large, tenant-scoped lists (e.g. activity logs, documents). Use offset only for small, admin-style lists or when the client needs “page 5” by number.

---

## 4. Authentication and authorization

### Q: What’s the difference between authentication and authorization?

**A:**

- **Authentication (AuthN):** “Who are you?” — verifying identity (e.g. password, OTP, SSO). Result: you know the **user** (and often which **tenant(s)** they belong to).
- **Authorization (AuthZ):** “What are you allowed to do?” — checking permissions on a resource. Result: allow or deny the action (e.g. “can this user edit this space?”).

Flow: Authenticate first → get user (and tenant context) → then for each request, authorize that user (and tenant) for the requested resource and action.

---

### Q: How does JWT work and when would you use it vs sessions?

**A:**

- **JWT:** Signed token (e.g. HS256 or RS256) containing claims (e.g. `sub` = user id, `org_id` = tenant, `exp` = expiry). Client sends it (e.g. in `Authorization: Bearer <token>`). Server verifies signature and expiry and reads claims; no DB lookup for every request.
- **Pros:** Stateless, scales horizontally, works across domains; good for APIs and mobile.
- **Cons:** Hard to revoke before expiry (need blocklist or short expiry + refresh); payload size; can’t change permissions until next token.

- **Sessions:** Server stores session (e.g. in Redis or DB); client sends session id (cookie or header). Server looks up session each request.
- **Pros:** Easy to invalidate (delete session); can change permissions immediately.
- **Cons:** Stateful; need shared store for multi-instance; cookies and CSRF to handle.

**For SaaS APIs:** JWTs are common (stateless, multi-instance). Use short-lived access token + refresh token; store refresh tokens in DB or Redis so you can revoke them. Include tenant (org_id) and optionally role in JWT so the API can enforce tenant and permissions without a DB hit on every request.

---

### Q: How would you implement RBAC (Role-Based Access Control) in a SaaS app?

**A:**

1. **Model:** **Users** belong to **tenants** via **tenant_members** (user_id, tenant_id, role). **Roles** (e.g. admin, member, viewer) have a set of **permissions** (e.g. `space:create`, `space:delete`, `member:invite`). Optionally store permissions per role in DB or in code (e.g. role → list of permissions).
2. **Resolve user and tenant:** From JWT or session: user_id, tenant_id (and maybe role).
3. **Check permission:** Before an action (e.g. “delete space”), check that the user’s role in that tenant has the required permission (e.g. `space:delete`). If you need resource-level checks (“can edit *this* space”), also verify the resource belongs to the tenant and that the user has access to that resource (e.g. space membership).
4. **Where to enforce:** In a **guard or middleware** (reject request if not allowed) and optionally in **service layer** (e.g. `requirePermission(user, tenant, 'space:delete')`). Never rely only on the frontend.
5. **Multi-tenant:** Always check permission in the context of the **tenant** from the request (and ensure the resource’s tenant_id matches).

---

### Q: What is SSO and what are SAML and OIDC?

**A:**

- **SSO (Single Sign-On):** User signs in once at their organization’s identity provider (IdP), and can access your app without entering a separate password. Enterprise customers often require this.
- **SAML 2.0:** XML-based. Flow: user hits your app → redirect to IdP → user logs in at IdP → IdP POSTs a SAML assertion to your callback URL → you validate assertion, create or link user, establish session. You need to configure SP (Service Provider) metadata and consume IdP metadata.
- **OIDC (OpenID Connect):** JSON-based, built on OAuth 2.0. Flow: redirect to IdP → user logs in → IdP redirects back with an **authorization code** → you exchange code for **id_token** (JWT with user claims) and optionally **access_token**. You validate the id_token (signature, issuer, audience, expiry) and create or link user.

**For SaaS:** Support **OIDC** (e.g. Google, Okta, Azure AD) for most modern IdPs. Support **SAML** if enterprise customers require it. In both cases, map the IdP’s user identifier to your user (and tenant) so the same person isn’t duplicated when they use SSO vs email/password.

---

### Q: How do you prevent a user from accessing another tenant’s data?

**A:**

1. **Tenant from token:** Resolve tenant_id from JWT/session (e.g. `org_id` claim or session record). Never trust tenant from query/body alone for authorization.
2. **Scope all queries:** For any tenant-scoped table, always filter by this tenant_id. Use a shared helper or scope so you can’t forget.
3. **Resource checks:** For “get space by id,” ensure `space.tenant_id === request.tenantId`. If not, return 404 (don’t leak existence). Same for update/delete.
4. **Row-Level Security (RLS):** In PostgreSQL, RLS policies can enforce “user can only see rows where tenant_id = current_setting('app.tenant_id')”. Set `app.tenant_id` per request; then even raw SQL is constrained.
5. **Tests:** Integration tests that call API with user from Tenant A and resource from Tenant B; expect 403/404.

---

## 5. Other critical topics for SaaS

### Q: How would you design a subscription and billing system?

**A:**

- **Plans and features:** Store plans (e.g. Free, Pro, Enterprise) and what they include (seats, storage, features). Use a **feature flag / entitlement** layer that checks “does this tenant’s plan allow X?” for billing-related features.
- **Subscription state:** Store current plan, billing cycle, status (active, past_due, cancelled). Use a payment provider (e.g. Stripe) for invoices and payments; store subscription_id and customer_id from the provider.
- **Webhooks:** Receive `invoice.paid`, `customer.subscription.updated`, etc. from the provider. Update your DB in a **idempotent** way (use event id or idempotency key so duplicate webhooks don’t double-apply). Then sync state (plan, limits) so the app enforces quotas.
- **Usage-based:** If you charge by usage (API calls, storage), emit usage events to your billing system or the provider; aggregate per billing period. Enforce soft limits (warn) and hard limits (block or throttle) in the app.
- **Multi-tenant:** All subscription and usage data is keyed by tenant_id. Never mix tenants in billing logic.

---

### Q: How do you implement idempotency for payments and webhooks?

**A:**

- **Idempotency key:** Client (or webhook sender) sends a unique key (e.g. `Idempotency-Key: uuid`) per logical operation. Server stores (key → result) for a time window (e.g. 24 hours). If the same key is seen again, return the stored result instead of re-running the operation.
- **Storage:** Redis or DB: `idempotency_keys(key, tenant_id?, response_status, response_body, created_at)`. Key might be (tenant_id, client_key) or just client_key depending on scope.
- **Webhooks:** Use the provider’s event id (e.g. `evt_xxx`) as idempotency key. First time: process and store success. Duplicate: return 200 and do nothing. Prevents double-crediting or double-provisioning when the provider retries.
- **Payments:** For “charge this card,” client sends idempotency key; server creates charge once and caches the outcome so retries don’t create multiple charges.

---

### Q: How do you secure APIs (auth, input, injection, leaks)?

**A:**

- **Auth:** Every protected route validates JWT or session; reject with 401 if missing or invalid. Resolve tenant and user from token.
- **Authorization:** After auth, check permission for this tenant and resource; 403 if not allowed.
- **Input validation:** Validate and sanitise all inputs (body, query, params). Use allowlists and types (e.g. DTOs with class-validator). Reject invalid input with 400.
- **Injection:** Use parameterised queries only; never concatenate user input into SQL. Same for NoSQL (e.g. avoid building queries from raw strings). Use ORM/query builder correctly.
- **Leaks:** Don’t return internal errors or stack traces to the client. Don’t log passwords or tokens. In responses, don’t expose other tenants’ existence (prefer 404 over “forbidden for this org”). Sanitise error messages.
- **HTTPS only;** secure cookies (SameSite, HttpOnly, Secure). CORS configured for your frontend origins only.

---

### Q: What would you cache in a SaaS and how do you invalidate?

**A:**

- **Cache:** Tenant settings, feature flags, user permissions (or role), hot reference data. Key format: `tenant:{id}:settings`, `user:{id}:permissions` so keys are tenant/user scoped.
- **Invalidation:** On update (e.g. tenant settings changed), delete or update the cache entry. Use cache-aside: read from cache; on miss read from DB and set cache. On write: update DB then invalidate cache.
- **TTL:** Set a short TTL (e.g. 5–15 min) as a safety net so stale data doesn’t live forever if invalidation is missed.
- **Don’t cache:** Per-request data, or anything that must be real-time (e.g. “current user’s draft”) unless you have a clear invalidation path.

---

### Q: How do you make the system observable (logs, metrics, errors)?

**A:**

- **Logs:** Structured logs (JSON) with request_id, user_id, tenant_id, action, duration, status. Never log secrets or full tokens. Use levels (info, warn, error). Centralise (e.g. CloudWatch, Datadog, ELK) and search by tenant_id or request_id.
- **Metrics:** Counters for requests, errors, queue depth; histograms for latency. Tag by tenant_id (or tenant bucket) and endpoint. Alert on error rate, p99 latency, and dependency failures.
- **Errors:** Capture exceptions and report to an error tracking service (e.g. Sentry). Include tenant_id and user_id (not PII) so you can see which tenant is affected. Use for debugging and incident response.
- **Tracing:** Distributed trace id across API → DB → queue so you can see the full path of a request. Helps with performance and debugging in microservices.

---

## Quick reference checklist before building SaaS

- [ ] **Tenancy:** Tenant_id on every tenant-scoped table; resolve tenant from auth; never query without tenant scope.
- [ ] **Auth:** JWT or session; include tenant (and optionally role) in token/session; short-lived access + refresh.
- [ ] **AuthZ:** RBAC or permission checks on every protected action; enforce in backend, not only frontend.
- [ ] **APIs:** REST (or GraphQL) with consistent errors, validation, rate limiting per tenant.
- [ ] **DB:** Indexes on (tenant_id, …); avoid N+1; use cursor pagination for large lists; migrations tested and reversible where possible.
- [ ] **Billing:** Plans and entitlements; integrate with payment provider; idempotent webhook handling.
- [ ] **Security:** Input validation, parameterised queries, no secret/token in logs, HTTPS, CORS.
- [ ] **Observability:** Logs and metrics with tenant_id; error tracking; alerts on errors and latency.
