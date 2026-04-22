# Smart Structures (RADISE India) — Interview Prep

**Role:** Full Stack Developer (Frontend + Backend) — SmartHub  
**Company:** Smart Structures (RADISE India Pvt. Ltd.) — Infrastructure IoT, AEC sector

**Your profile:** Pulkit Singh — Senior Backend Engineer, 5+ years — Node.js, NestJS, TypeScript, React/Next, PostgreSQL, MongoDB, Redis, BullMQ, GCP/AWS, CI/CD.

---

## Part 1 — Step-by-step interview flow

### 1. Opening (2–3 minutes)

- Greet, thank them for time, confirm role (Full Stack, SmartHub).
- **One-line pitch:** *“I’m a senior backend-focused full-stack engineer with 5+ years building scalable B2B SaaS APIs in NestJS and TypeScript, with strong PostgreSQL/MongoDB and Redis experience, plus React/Next for product surfaces. I’ve owned features end-to-end—from RBAC and payments to media pipelines and real-time systems—and I’m excited about applying that to infrastructure IoT and sensor analytics.”*
- **Bridge to them:** *“I’d love to understand how SmartHub splits frontend vs platform services and how sensor data flows into the product.”*

### 2. Walkthrough of experience (5–7 minutes)

- **Current role (Corpusvision / SwagShots):** NestJS, TypeScript, scale, media (FFmpeg, BullMQ, HLS), live streaming integration, FCM, analytics (MongoDB aggregations), S3 + CDN, RBAC (Passport + JWT), payments, bulk Excel + queues, Docker/Nginx/PM2, CI/CD.
- **DandD:** Express, MongoDB, restaurant SaaS / EMR modules, Firebase Auth, Twilio OTP, CRON.
- **Earlier:** MERN at Fliqa — shows you can own full stack when needed.

### 3. Projects deep-dive (pick 2–3, 3–5 min each)

Prepare **STAR** (Situation, Task, Action, Result) for:

| Project | What to emphasize for RADISE |
|--------|------------------------------|
| **SwagShots / short-video platform** | High-volume async jobs, reliability, APIs for dashboards — analogous to **sensor ingestion + processing pipelines** and **ops dashboards**. |
| **Analytics (MongoDB aggregation)** | Time-series-ish aggregations, retention — maps to **trend analysis over sensor/asset data**. |
| **RBAC** | Multi-tenant permissions — maps to **project/site roles in AEC**. |
| **Media pipeline (FFmpeg, BullMQ)** | **Heavy async processing** — same pattern as batch analytics or model pre-processing. |
| **TravelBaba.ai / AI tools** | **AI awareness** — agents, SDKs, how you’d apply ML to alerts or predictions. |

### 4. Technical rounds (their stack)

- **Frontend:** React, Next.js App Router vs Pages, Redux (when/why), performance, forms, auth.
- **Backend:** NestJS modules, guards, interceptors, PostgreSQL vs MongoDB tradeoffs, transactions, indexing.
- **System design (lite):** API for projects/assets/sensors, caching, queues, real-time optional (WebSockets).

### 5. Domain fit (IoT / AEC)

- Show **curiosity**, not fake expertise: standards you’d learn, how you’d model **sites → assets → sensors → readings**.
- Mention **reliability, audit trails, and latency** for safety-related data.

### 6. Behavioral

- Ownership, conflict, missed deadline, production incident, mentoring (Team Lead).

### 7. Your questions for them

- SmartHub architecture (monolith vs services, main data stores).
- How sensor data is validated, stored, and exposed to the UI.
- AI roadmap (anomaly detection, forecasting, copilots).
- Team structure, on-call, release cadence, AEC/IoT learning resources.

---

## Part 2 — Short scripts for your projects

**SwagShots / platform:**  
*“I led backend for a short-video social product: NestJS APIs, JWT/RBAC, async video processing with FFmpeg and BullMQ on Redis, HLS for playback, and integrations for live streaming and push notifications. We used MongoDB for flexible engagement data and aggregations for analytics, plus object storage and CDN for media. That taught me to design for spikes, retries, and observable pipelines—relevant when sensor or batch jobs spike.”*

**RBAC:**  
*“We modeled roles and permissions with Passport and JWT, centralizing guards so every route enforced the same rules—similar to project-level access in construction software.”*

**AI (TravelBaba / tools):**  
*“I’ve used AI SDKs and agent-style flows for travel assistance; I’m not training models from scratch, but I’m comfortable integrating APIs, structuring prompts, and thinking about when ML is appropriate vs rule-based alerts.”*

---

## Part 3 — 50+ Q&A (JD + resume)

### A. Role, motivation, AEC/IoT (5)

**1. Why RADISE / SmartHub?**  
You want to apply full-stack and data-heavy backend skills to **physical infrastructure**—high impact, real-time monitoring, and long-term asset lifecycle—beyond pure consumer social.

**2. What do you know about IoT in civil infrastructure?**  
Sensors on bridges, buildings, or sites produce **time-series readings** (strain, tilt, vibration). Software ingests, stores, visualizes, and alerts stakeholders—aligned with safety and maintenance.

**3. How is this different from SwagShots technically?**  
Similar: **async pipelines, dashboards, scale**. Different: stronger **accuracy, auditability, domain models** (assets, compliance), and often stricter **data retention and access**.

**4. You’re backend-heavy; why full stack here?**  
You’ve shipped React/Next for admin and product surfaces; you want **end-to-end ownership**—APIs plus performant UIs for operators and engineers.

**5. How do you learn a new domain (AEC)?**  
Pair with domain experts, read internal glossaries, trace one **sensor → UI** path in staging, document assumptions, iterate.

---

### B. React / Next.js / Redux (10)

**6. When do you use Redux vs React context or server state?**  
Redux for **complex client-wide state** (many components, time-travel debugging, middleware). Context for **theme/auth** scoped trees. **TanStack Query / server cache** for server data—avoid duplicating server state in Redux unnecessarily.

**7. Next.js App Router vs Pages Router?**  
App Router: **React Server Components**, nested layouts, **streaming**, colocated routes. Pages Router: mature, simpler mental model. New projects often App Router; migrating needs clear data-fetch boundaries.

**8. Example: optimize a heavy list in React.**  
**Virtualize** (e.g. react-window), **memo** stable row components, **key** by id, **paginate or infinite scroll** on server, avoid inline object props that break memo.

**9. How do you handle forms at scale?**  
Controlled vs uncontrolled tradeoffs; **schema validation** (Zod) shared with API; **optimistic UI** only when rollback is clear.

**10. CSR vs SSR vs SSG for a dashboard?**  
Dashboards are often **auth-gated** → **SSR or client fetch after auth**; static marketing pages **SSG**. Hybrid: shell SSR, data client-side with loading states.

**11. Redux middleware you’ve used?**  
**redux-thunk** for async; **redux-saga** if complex orchestration (less common now with RTK Query).

**12. What is RTK Query?**  
Official data-fetching for Redux Toolkit: **caching, invalidation, loading states**—reduces boilerplate vs manual slices for API data.

**13. Accessibility basics you apply?**  
Semantic HTML, labels for inputs, keyboard nav, focus management in modals, color contrast.

**14. Front-end security?**  
**XSS**: sanitize user HTML, avoid `dangerouslySetInnerHTML` without sanitization; **CSRF**: cookies + same-site or tokens for state-changing requests; never put secrets in client bundles.

**15. Example from your work:**  
*“Admin panels for content or analytics—role-gated routes, loading/error boundaries, tables with pagination to stay fast.”*

---

### C. TypeScript / Node / NestJS (12)

**16. Why NestJS for SmartHub-like backends?**  
**Modular** structure (feature modules), **DI**, built-in support for **guards, pipes, interceptors**, consistent patterns for large teams—fits enterprise B2B.

**17. Explain Guards vs Pipes vs Interceptors.**  
**Guards**: *can* activate route (auth, roles). **Pipes**: transform/validate input/output. **Interceptors**: cross-cutting (logging, timeouts, response mapping).

**18. How do you structure modules?**  
By **bounded context**: `users`, `projects`, `sensors`, `readings`—each with controller, service, DTOs, entities.

**19. JWT flow you implemented (Passport).**  
Login issues **signed JWT** (short-lived access + optional refresh). Guards verify signature and claims; **RBAC** checks permissions on top.

**20. Example RBAC pattern.**  
Roles like `admin`, `site_engineer`; permissions as strings; **guard** loads user’s roles from DB/cache and checks `permissions.includes('sensor:write')`.

**21. Handling idempotent payments (Razorpay etc.).**  
**Idempotency keys** on payment intent; **webhook signature** verification; **store events**; reconcile with provider; never double-credit on retries.

**22. BullMQ at a glance.**  
**Redis-backed** queues: jobs with retries, backoff, concurrency—used for **video processing** in your resume; same for **report generation** or **batch sensor exports**.

**23. Why Redis beside PostgreSQL?**  
**Cache**, **sessions**, **rate limits**, **queue backend**—fast ephemeral data; PostgreSQL remains **source of truth** for relational integrity.

**24. Error handling in APIs.**  
Consistent **problem+json** or structured errors; **correlation IDs**; map domain errors to correct **HTTP codes** (400 vs 404 vs 409).

**25. Validation in NestJS.**  
**class-validator** + **class-transformer** on DTOs; whitelist extras to block mass assignment issues.

**26. Testing pyramid for your APIs.**  
Many **unit** tests for services; **integration** tests for controllers with test DB; **e2e** for critical paths; contract tests if microservices.

**27. Example tying to SwagShots.**  
*“Upload triggers a job; worker transcodes; status polled or pushed via websocket/notification—pattern applies to long-running IoT batch jobs.”*

---

### D. PostgreSQL vs MongoDB (8)

**28. When PostgreSQL?**  
**Relational** data: users, projects, permissions, **financial** records, **strong consistency**, **JOINs**, **transactions**.

**29. When MongoDB?**  
**Flexible schema**, **nested documents**, **high write** telemetry or **documents** that vary by type—often **sensor payloads** if schema evolves, though time-series may also go to specialized stores.

**30. Indexing example.**  
PostgreSQL: `CREATE INDEX ON readings (sensor_id, recorded_at DESC)` for time-range queries. Explain **selectivity** and **avoid over-indexing** writes.

**31. Transactions example.**  
Transfer between accounts or **create project + default roles** in one transaction—`BEGIN … COMMIT`.

**32. MongoDB aggregation example (your experience).**  
**$match** by date, **$group** by user or region, **$sort**—for **engagement metrics**; same pattern for **counts per asset**.

**33. Migration strategy.**  
**Prisma / TypeORM / Knex** migrations versioned in CI; backward-compatible deploys (**expand-contract** when needed).

**34. JSON columns in PostgreSQL.**  
Useful for **semi-structured** metadata while keeping relational core—balance vs Mongo for heavy nested queries.

**35. Read replicas.**  
Offload reporting queries; **eventual lag**—dashboards must tolerate or show “as of” time.

---

### E. APIs, performance, scalability (6)

**36. Pagination patterns.**  
**Offset/limit** (simple, bad for deep pages) vs **cursor** on `(created_at, id)` for stable large feeds.

**37. Caching strategy.**  
Cache **read-heavy** endpoints with **TTL**; **invalidate** on writes or use tags; **ETag** for static resources.

**38. Rate limiting.**  
Per IP/user token in **Redis**; protect auth and expensive endpoints.

**39. N + 1 query problem.**  
ORM loading related entities in a loop—fix with **joins**, **DataLoader**, or **batch queries**.

**40. API versioning.**  
`/v1` prefix or header; deprecate with timeline—important for **embedded hardware or mobile** clients.

**41. Your scale story (21K users).**  
*“We optimized hot paths, used queues for heavy work, and CDN for media—I'd apply the same discipline to API latency for charts and alerts.”*

---

### F. DevOps, CI/CD, cloud (5)

**42. Describe your CI/CD.**  
**GitHub Actions**: lint, test, build, deploy; **secrets** in repo settings; **branch protection**.

**43. Docker in production.**  
Immutable images; **multi-stage builds**; non-root user; healthchecks—aligns with **repeatable SmartHub deploys**.

**44. Blue-green or rolling?**  
**Rolling** on K8s/common; **zero-downtime** migrations; feature flags for risky changes.

**45. What you used on AWS.**  
**S3** storage, **Lambda** for event-driven tasks—say what *you* actually did to stay credible.

**46. Monitoring.**  
Structured logs, **metrics** (latency, error rate), **alerts** on SLO breaches—critical for IoT pipelines.

---

### G. AI / ML awareness (6)

**47. How could AI help SmartHub without training models Day 1?**  
**Anomaly detection** on streams (rules + simple stats first), **forecasting** maintenance windows, **NLP** on reports, **copilot** for querying sensor data—integrate **OpenAI/TensorFlow.js** where appropriate.

**48. When not to use ML?**  
Clear **threshold-based** safety rules, **regulatory** simplicity, **small data**—start deterministic; add ML when **signal and labels** exist.

**49. TensorFlow.js vs server ML.**  
**Client-side** for privacy/low latency demos; **server** for heavy models and large data.

**50. Prompting best practice.**  
**Structured output** (JSON schema), **system vs user** roles, **guardrails** for hallucinations in operational advice.

**51. Your AI line on resume.**  
*“I’ve integrated LLM APIs and built agent-style flows; for RADISE I’d focus on **assistive** features—summaries, suggested actions—with human approval for safety-critical decisions.”*

**52. Evaluation of an ML feature.**  
Define **precision/recall** for alerts, **false alarm rate**, **latency**; shadow mode before full rollout.

---

### H. IoT / sensors / SmartHub-shaped system design (6)

**53. High-level architecture for sensor data.**  
**Ingestion API** or **MQTT gateway** → **validation** → **time-series or relational store** → **aggregation jobs** → **API for UI** → **alerting** if thresholds crossed.

**54. Idempotency for device uploads.**  
**Device id + message id** stored; duplicates ignored—critical when networks retry.

**55. Clock skew.**  
Use **server receive time** or **synchronized NTP** on devices; document **which timestamp** is authoritative.

**56. Offline devices.**  
**Buffer** on edge; **backfill** with ordering; UI shows **data gaps** clearly.

**57. Multi-tenant isolation.**  
**tenant_id** on every row or schema-per-tenant; **tests** that forbid cross-tenant reads—like your **B2B SaaS** experience.

**58. Real-time vs poll.**  
**WebSockets** or **SSE** for live dashboards when needed; **polling** for low-frequency—your **WebSocket / real-time** experience applies.

---

### I. Behavioral & collaboration (6)

**59. Leading SwagShots technically.**  
**Ownership** of architecture, **tradeoff** communication, **mentoring** juniors, **stakeholder** updates—**outcome**: shipped features and scale.

**60. Production incident example (template).**  
Symptom → **detect** (alerts) → **mitigate** (rollback/scale) → **root cause** (bad deploy, query, queue backlog) → **prevent** (tests, monitors).

**61. Disagreement with PM.**  
**User evidence** + **cost/scope**; propose **MVP** and **phase 2**; align on **metrics**.

**62. Code review philosophy.**  
**Safety and clarity** first; **ask questions**; **suggest**; **approve** when adequate with follow-ups tracked.

**63. Agile / Jira.**  
**Sprint planning**, **estimates**, **retros**—continuous improvement.

**64. Weakness (credible).**  
e.g. *“I sometimes dive deep before scoping—I now timebox spikes and write a short design note first.”*

---

### J. Extra technical (6+)

**65. HTTPS/TLS termination.**  
Often at **Nginx** or load balancer; **cert renewal** (Let’s Encrypt); **HSTS** in production.

**66. Webhook security.**  
**HMAC signature** with shared secret; **replay** protection with timestamp window.

**67. File upload security.**  
**MIME sniffing**, **size limits**, **virus scan** if needed, **private buckets** with **signed URLs**.

**68. Feature flags.**  
Roll out to % of tenants; **kill switch** for new sensor views.

**69. Internationalization.**  
If SmartHub goes global: **i18n** strings, **date/number** formatting, **RTL** if needed.

**70. GDPR / data minimization (if asked).**  
**Retention** policies, **export/delete** flows, **audit logs** for access—especially for **enterprise** clients.

---

## Quick alignment cheat sheet (you ↔ JD)

| JD asks | Your evidence |
|--------|----------------|
| React, Next, Redux | Fliqa MERN; admin/product UIs; study RTK Query if rusty |
| NestJS, Node, TypeScript | Core of last 5 years |
| PostgreSQL, MongoDB | Both on resume—prep SQL indexing + transactions |
| MongoDB exposure | Aggregations, analytics |
| REST, performance | SwagShots, APIs at scale |
| AI awareness | TravelBaba.ai; Claude/SDK; integration mindset |
| DevOps, CI/CD | GitHub Actions, Jenkins, Docker, Nginx |
| IoT/AEC (preferred) | Honest learning + analogies from pipelines/dashboards |

---

## Related files

- **Narrowed prep** (20-minute script vs deep dives): `narrowed-prep-options.md`
- **System design** (20 questions with deep explanations): `system-design-20-questions.md`
