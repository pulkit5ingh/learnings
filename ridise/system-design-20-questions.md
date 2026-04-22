# 20 System Design Questions & Answers (Deep Dive)

Context: **SmartHub-style** platforms — project/asset management, **sensor analytics**, AEC, multi-tenant B2B. Use these to practice **clarifying requirements**, **tradeoffs**, and **failure modes** before whiteboarding.

---

## How to use this in an interview

1. **Clarify** functional + non-functional requirements (scale, latency, consistency, compliance).
2. **Sketch** data flow: clients → API gateway → services → storage → async workers.
3. **Name tradeoffs** (SQL vs TSDB, sync vs async, strong vs eventual consistency).
4. **Discuss** failure: retries, idempotency, dead-letter queues, observability.

---

## 1. Design an end-to-end IoT platform for civil infrastructure (SmartHub-style)

**What they’re really asking:** Can you decompose a large product into **bounded contexts**, storage choices, and operational concerns?

**Deep explanation**

- **Clarify requirements**
  - **Users:** org admins, project engineers, field operators, possibly read-only clients.
  - **Entities:** organizations (tenants) → projects/sites → **assets** (bridge span, building) → **sensors** → **readings** (time-stamped values) + **alerts**, **reports**, **documents**.
  - **Non-functional:** durability of sensor data, **auditability**, reasonable **dashboard latency** (seconds to minutes for batch aggregates), **multi-tenant isolation**, **RBAC**.

- **High-level architecture**
  - **Clients:** Web (React/Next), mobile, optional **edge gateways** aggregating device traffic.
  - **Edge/API:** **API Gateway** or load-balanced **NestJS** (or multiple services) — authn (JWT/OIDC), **tenant context**, rate limits.
  - **Core services (logical modules):** Identity & RBAC; Project/Asset registry; **Ingestion** (HTTP/MQTT adapter); **Query API** for charts; **Alert engine**; **Reporting** (async); **Notification** (email/SMS/push).
  - **Data:** **PostgreSQL** for relational core (orgs, projects, users, roles, asset metadata). **Time-series store** or PostgreSQL **partitioned tables / TimescaleDB** for high-volume readings. **Object storage** (S3-compatible) for PDFs, exports, large attachments. **Redis** for cache, rate limits, **pub/sub** or short-lived session features. **Message queue** (Kafka, RabbitMQ, or **BullMQ** on Redis) for async analytics and notifications.

- **Why split relational vs time-series**
  - Relational DBs excel at **joins, constraints, transactions** for “who owns what.” Raw sensor streams are **append-heavy, time-range query** workloads — specialized storage or careful partitioning avoids bloating OLTP tables.

- **Failure & ops**
  - **Idempotent ingestion** (device + message id). **Observability:** traces from API to DB, **metrics** on ingest lag and error rates. **Backups:** PITR for Postgres; lifecycle policies on object storage.

**One sentence takeaway:** Separate **authoritative metadata** (Postgres) from **high-volume telemetry** (TSDB or partitioned SQL) and **push heavy work async** (queues).

---

## 2. How would you design sensor data **ingestion** at scale?

**Deep explanation**

- **Protocols:** Devices may use **HTTPS** (POST batches), **MQTT** (broker + subscribers), or **gateway** that normalizes to one internal format.
- **Ingestion path:** Request hits **ingestion service** → **validate** (schema, signature, tenant) → **dedupe** by `(device_id, message_id)` stored in Redis or DB **unique constraint** → **write** to buffer/queue or direct **append** to TS layer.
- **Batching:** Devices often send **batches** every N seconds to save battery/network — server should **ack** at batch level but store **per-reading** granularity if product needs it.
- **Backpressure:** If downstream is slow, use **queue** so ingestion returns quickly; workers drain at sustainable rate. **Protect** with **max queue depth** + alerts + shedding only as last resort (drops are dangerous for safety data — prefer slowing producers via 429/coordinated firmware).

- **Idempotency:** Network retries duplicate POSTs — **mandatory** dedupe keys.

**Takeaway:** Ingestion should be **fast, dumb, and safe** (validate + dedupe + enqueue); **heavy** validation/analytics happens **asynchronously**.

---

## 3. Where and how would you **store** millions of sensor readings per day?

**Deep explanation**

- **Options**
  - **PostgreSQL + partitioning** by time (monthly partitions) on `(sensor_id, time)` — good if team is SQL-first and volume fits.
  - **TimescaleDB / Citus** — Postgres extension for hypertables; **compression** of old chunks.
  - Dedicated **TSDB** (InfluxDB, Timescale, cloud offerings) — optimized **downsampling**, **retention policies**.

- **Data model (typical)**
  - Table/collection: `sensor_id`, `timestamp` (or `time` bucket), `value`, optional `quality` flag, `metadata` JSON for anomalies.

- **Writes:** Mostly **append-only**; avoid updates except **corrections** (then either **tombstone + insert** or versioned row — audit may require **immutable** log).

- **Reads:** **Time-range scans** — **indexes** on `(sensor_id, time DESC)`. **Aggregates** precomputed in **rollups** (hourly/daily) via scheduled jobs to keep dashboards cheap.

**Takeaway:** Match storage to **query pattern** (last 24h raw vs 1-year hourly aggregates); use **rollups** so BI doesn’t scan billions of raw points.

---

## 4. Design a **real-time dashboard** showing live sensor metrics

**Deep explanation**

- **Latency target:** “Real-time” often means **1–10s** refresh for infra monitoring, not always sub-100ms — clarify.
- **Data path:** Recent readings from **cache** (Redis) or **last-N** query optimized on TSDB; **WebSocket** or **SSE** pushes updates to browser when new batch arrives.
- **Aggregation:** UI rarely needs every raw point — **server** can stream **downsampled** series (e.g. last 1000 points or 1/min buckets).
- **Frontend:** Chart lib subscribes to channel `project:{id}:sensor:{id}`; **reconnect** with **last timestamp** to fill gaps.
- **Scale:** **Shard** WebSocket connections; use **Redis pub/sub** or **Kafka consumer groups** to fan out updates to connection servers.

**Takeaway:** Combine **efficient recent reads**, **push** for freshness, and **gap recovery** on reconnect.

---

## 5. Design an **alerting system** when sensor values cross thresholds

**Deep explanation**

- **Rules:** Static thresholds, **hysteresis** (alert at >X, clear at <Y) to avoid flapping. Optional **rate-of-change** rules.
- **Execution:** **Stream processing** (evaluate on ingest) vs **scheduled** (poll last value every minute). For safety, **ingest-time** evaluation gives lowest latency.
- **State:** Track **alert state** per rule per sensor (OK / FIRING / ACKED) in **Postgres** or Redis with persistence for **audit**.
- **Notifications:** **Queue** alert events → **notification service** (email, SMS, webhook). **Dedupe** repeated firings (only notify every T minutes unless severity changes).
- **Idempotency:** Same condition firing repeatedly should not spam — **cooldown** windows.

**Takeaway:** Separate **rule evaluation** from **notification delivery**; persist state for **acknowledgment** and **post-incident review**.

---

## 6. Design **multi-tenant SaaS** for construction / engineering orgs

**Deep explanation**

- **Isolation models**
  - **Shared DB, tenant_id column** — simplest; **strict** middleware sets tenant from JWT; **row-level security** in Postgres optional defense-in-depth.
  - **Schema per tenant** — stronger isolation, harder migrations.
  - **DB per tenant** — enterprise tier; heavy ops.

- **Cross-cutting:** Every API resolves **tenant** from token + **membership** check. **No** tenant id from client body without verification.

- **Noisy neighbor:** **Rate limits per tenant**, **query timeouts**, optional **resource quotas** (storage, API calls).

**Takeaway:** Default **tenant_id everywhere** + tests that **cross-tenant reads are impossible**.

---

## 7. Design **RBAC** for projects, sites, and assets (hierarchical permissions)

**Deep explanation**

- **Model:** **Roles** (OrgAdmin, ProjectManager, Viewer) **scoped** to org or project. **Permissions** as strings (`asset:read`, `sensor:write`, `report:export`).
  - Alternative: **attribute-based** (ABAC) for complex rules — heavier.
- **Storage:** `users`, `roles`, `permissions`, **junction** `user_project_roles`. **Inherited** roles (org admin → all projects) implemented in **service layer** or **materialized** view.
- **Enforcement:** **NestJS Guards** — load user’s roles for `projectId` from cache (Redis) to avoid DB hit per request.
- **Audit:** Log **permission denials** sparingly; log **sensitive grants** always.

**Takeaway:** **Normalize** roles in DB; **resolve** effective permissions in one service with **caching**.

---

## 8. Design **file storage** for reports, drawings, and large exports

**Deep explanation**

- **Never** stream large files through app servers long-term — use **object storage** (S3, GCS, MinIO).
- **Flow:** Client requests **signed upload URL** or POSTs to API that streams to bucket with **virus scan** optional. **Metadata** (name, tenant, project, hash) in **Postgres**.
- **Download:** **Signed GET URLs** with **short TTL**; **private** buckets, no public ACLs.
- **Versioning:** S3 versioning or **append-only** new key per version + pointer in DB.

**Takeaway:** **Metadata in SQL**, **bytes in object store**, **signed URLs** for transfer.

---

## 9. Design **async processing** for heavy analytics (reports, rollups, ML features)

**Deep explanation**

- **Queue:** **BullMQ** (Redis), **RabbitMQ**, or **Kafka** depending on ordering and replay needs. Kafka for **event log**; BullMQ fine for **job** workloads.
- **Workers:** Stateless **worker** processes scale horizontally; **concurrency** per queue; **retry** with **exponential backoff**; **dead-letter queue** for poison messages.
- **Exactly-once** is hard — aim for **at-least-once** + **idempotent** workers (same job key doesn’t double-apply).

- **Scheduling:** **Cron** triggers daily rollups; **backfill** jobs for new aggregation logic over historical data (chunk by time range).

**Takeaway:** Same pattern as **FFmpeg/BullMQ** on your resume — **decouple** user-facing latency from **batch** work.

---

## 10. Design **APIs** for web, mobile, and optional edge gateways

**Deep explanation**

- **REST** with **versioning** (`/v1`) — predictable for diverse clients. **GraphQL** optional if many clients need flexible queries (cost: complexity, abuse).
- **Consistency:** **Pagination** — **cursor-based** for large lists. **Filtering** on indexed fields only.
- **Mobile:** **Payload size** limits; **delta sync** endpoints if offline-first ever needed.
- **Edge:** **API keys** + **mTLS** for gateways; **narrow** surface (ingest-only endpoints).

**Takeaway:** **Stable contracts**, **pagination**, **auth per client type**, **observable** per-route latency.

---

## 11. **Caching** strategy for read-heavy dashboards and metadata

**Deep explanation**

- **What to cache:** **Hot** project/asset metadata; **last reading** per sensor; **precomputed** chart fragments for common ranges.
- **Where:** **Redis** with **TTL**; **cache-aside** pattern (read Redis → on miss read DB → set Redis).
- **Invalidation:** Hardest problem — on asset update, **delete** keys `asset:{id}:*` or use **version** in key. For readings, **short TTL** (seconds) often enough.
- **CDN:** Static assets, **public** maps tiles if any — not for **private** JSON.

**Takeaway:** Cache **aggregates** and **metadata**; be careful caching **security-sensitive** blobs without **tenant** in key.

---

## 12. **Audit logging** and compliance for infrastructure data

**Deep explanation**

- **Events:** Who accessed which **report**, who changed **threshold**, who **exported** data — **append-only** `audit_log` table or stream to **immutable** store (WORM bucket, SIEM).
- **Sensor data:** May need **legal hold** — retention beyond default. **Separate** policy engine from app code.
- **PII:** Minimize; **hash** where possible; **regional** residency if required.

**Takeaway:** Treat audit as **first-class** stream with **tamper-evidence** (hash chains) if regulation demands.

---

## 13. Handle **late-arriving** and **out-of-order** sensor data

**Deep explanation**

- **Causes:** Clock skew, offline buffering, network retries.
- **Ingestion:** Accept batch with **device timestamp** + **server receive time**; store **both**; **document** which drives **billing** vs **analytics**.
- **Ordering:** **Per sensor**, process by **event time** in analytics; **watermarks** in stream processing (“ignore updates older than T”).
- **Replay:** **Recompute** rollups for affected windows when late data arrives (small **correction jobs**).

**Takeaway:** **Dual timestamps** + **reprocessing** strategy beats pretending streams are perfectly ordered.

---

## 14. **Rate limiting** and abuse prevention on public or partner APIs

**Deep explanation**

- **Layers:** **WAF**/gateway (IP), **application** (API key / user), **per-tenant** quotas.
- **Algorithm:** **Token bucket** or **sliding window** in **Redis** (`INCR` with TTL or sorted sets).
- **Response:** **429** + `Retry-After`; **circuit breaker** on client side documented.

**Takeaway:** **Protect** ingestion and expensive **export** endpoints most aggressively.

---

## 15. **Search** across projects, assets, and sensors (names, tags, IDs)

**Deep explanation**

- **Small scale:** Postgres **`ILIKE`** + **trigram** indexes (`pg_trgm`) or **full-text** for descriptions.
- **Larger / fuzzy:** **Elasticsearch** / **OpenSearch** — index denormalized docs `{tenant_id, project_id, type, title, tags}`; **sync** via **CDC** (Debezium) or **async** on write.
- **Security:** Search **must** filter by **tenant** and **RBAC** — usually **search after** resolving allowed `project_ids` or embed **tenant_id** in index queries.

**Takeaway:** Don’t build global search in SQL alone at scale — **dedicated search** + **strict** ACL filters.

---

## 16. **WebSockets** vs **polling** vs **SSE** for live updates

**Deep explanation**

- **Polling:** Simple, **wasteful** at high frequency; OK for **30s–5min** refresh.
- **SSE:** **One-way** server→client over HTTP, **reconnect** friendly — great for **live metrics** stream if no bidirectional need.
- **WebSockets:** **Bidirectional**, **stateful** connections — harder to scale (sticky sessions or shared **pub/sub** backplane). Use when **collaborative** features or **binary** streams matter.

**Takeaway:** Prefer **SSE** for push dashboards; **WebSockets** when you need **two-way** or **binary** efficiently.

---

## 17. **Data retention**, archival, and **cold storage**

**Deep explanation**

- **Policy:** Raw data **90 days**, minute aggregates **1 year**, hourly **10 years** — **business + legal** driven.
- **Implementation:** **Partition** by time; **detach** old partitions to **cheaper** storage (S3 Parquet) via **ETL** jobs; **query** via Athena/Spark for rare audits.
- **Delete:** **GDPR** right to erasure may conflict with **engineering retention** — **legal** process required.

**Takeaway:** **Tiered storage** + **automated lifecycle** jobs; **test restore** from cold path annually.

---

## 18. **Disaster recovery** and backups for a critical IoT platform

**Deep explanation**

- **RPO/RTO:** How much data loss vs downtime is acceptable — drives **async replication** vs **sync**, **multi-region**.
- **Postgres:** **PITR** with WAL archiving; **regular** restore drills. **Object storage:** **versioning** + **cross-region replication**.
- **Queues:** **Durability** settings; **Kafka** replication factor; understand **split-brain** scenarios.

**Takeaway:** Document **runbooks**; **automate** failover where cloud allows; **chaos** tests on restore.

---

## 19. **Idempotent** remote configuration and **OTA**-style updates to edge devices

**Deep explanation**

- **Problem:** Commands may **retry**; device may apply **twice** without care.
- **Pattern:** Each config message has **monotonic version** or **hash**; device applies only if **newer** than local state; **ack** includes **applied version**.
- **Server:** Store **desired state** vs **reported state**; **reconcile** loop (similar to **Kubernetes** controller pattern).

**Takeaway:** **Versioned desired state** + **explicit acks** beats fire-and-forget commands.

---

## 20. **Integrate** with external systems (BIM, ticketing, cloud ML)

**Deep explanation**

- **Anti-corruption layer:** Don’t leak vendor models into core — **translate** in **integration service**.
- **Sync:** **Webhook** inbound + **outbound** jobs with **retry**; **idempotency keys** on callbacks.
- **ML:** **Batch** export to **feature store** or call **hosted API** with **PII scrubbed**; **shadow** mode before production alerts driven by ML.

**Takeaway:** **Isolate** integrations; **contract tests** against vendor sandboxes; **feature flags** for rollout.

---

## Quick reference — patterns map

| Concern | Common pattern |
|--------|----------------|
| High-volume writes | Queue → TSDB / partitioned tables |
| Dashboards | Cache + rollups + SSE/WebSocket |
| Tenancy | `tenant_id` + middleware + tests |
| Reliability | Idempotency, DLQ, retries, observability |
| Security | RBAC, signed URLs, audit log |

---

## Related files

- `interview-prep-smart-structures-radise.md` — full interview prep  
- `narrowed-prep-options.md` — shortened study paths  
