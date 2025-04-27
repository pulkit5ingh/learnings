Kundli AI. app
plant finder.

- Ecomm
- https://flowbite.com/blocks/e-commerce/order-summary/

---

Hi,

Thank you for considering my profile for the backend developer position. I appreciate the opportunity to move forward in the selection process. Below are my responses to the technical questions you've shared:

---

1. System Design:

To design a scalable and secure REST or GraphQL API for an app with millions of users, I’d start by defining the core features and user flows, followed by structuring the system into microservices or modular monoliths, depending on the complexity and future scaling needs.

Key components of my design would include:

- API Gateway: To centralize authentication, logging, and rate-limiting.
- Horizontal Scaling: Deploy services behind a load balancer with stateless logic to scale horizontally.
- Caching Layer: Use Redis or CDN for caching frequent queries and static resources.
- Database Design: Normalize writes and denormalize reads using MongoDB for flexibility or Postgres for relational needs. Also implement replication and sharding strategies for large datasets.
- Authentication: Use JWT or OAuth2 with refresh tokens, stored securely (e.g., HttpOnly cookies).
- Monitoring: Integrate Prometheus + Grafana or ELK stack to monitor API health, errors, and latency.

Challenges:

- Handling N+1 queries in GraphQL – resolved via data loaders.
- Rate limiting for abusive users – addressed using Redis-backed token buckets.
- Versioning – important in REST APIs, done via URI or header.

---

2. Performance Under Load:

If the Node.js API is under heavy load, I follow these steps:

- Check Metrics: CPU/memory usage, response times, DB queries, and request per second using tools like PM2, New Relic, or Grafana.
- Logging & Tracing: Analyze slow requests using distributed tracing (OpenTelemetry, Jaeger).
- Identify Bottlenecks: Look at DB queries, synchronous/blocking code, external API calls, etc.
- Optimize:
  - Move heavy logic to background jobs (e.g., using BullMQ + Redis).
  - Introduce caching at the app and DB level.
  - Batch DB queries where possible.
  - Use cluster mode or scale services via Docker/Kubernetes.

If needed, I might even introduce a CDN for static and semi-static content, and a message queue (like RabbitMQ or Kafka) to offload async operations.

---

3. Async/await Pitfalls:

Common mistakes:

- Not using try/catch with await, causing unhandled promise rejections.
- Using await inside loops, which leads to serial execution instead of parallel – better to use Promise.all.
- Ignoring Promise chaining or nesting await, leading to deeply nested logic.
- Memory leaks due to hanging async operations or not cleaning up resources.

Prevention:

- Enforce linting rules like eslint-plugin-promise.
- Use wrapper utilities like asyncHandler in Express.
- Review code during PRs for unnecessary blocking patterns.
- Adopt good observability and error tracking tools like Sentry.

---

4. Rate Limiting:

In a distributed setup:

- Single Instance: Use express-rate-limit with in-memory or Redis store.
- Multi-instance: Redis is preferred since it provides a shared data store. Libraries like rate-limiter-flexible work great with Redis and allow flexibility in handling IP, user, or token-based limits.

Advanced options:

- API Gateway (e.g., Kong, NGINX with Lua) for centralized rate limiting.
- Use sliding window or token bucket algorithm for better smoothing.
- Return meaningful HTTP headers to clients about their quota status.

---

5. Database Load Management:

When DB fails under load, I follow this plan:

- Monitor: Check number of open connections, slow queries, lock contention.
- Connection Pooling: Misconfigured pooling can quickly exhaust DB. Ensure connection pooling is optimized via tools like pg-pool or Mongoose options.
- Indexing: Run EXPLAIN ANALYZE for slow queries to optimize.
- Caching: Use Redis or memory cache for repeated reads.
- Read Replicas: Offload read-heavy traffic.
- Queue Writes: For high write volumes, buffer non-critical writes via a queue system.

Long-term fixes:

- Enable autoscaling for DB in managed services.
- Break down monolithic DB into services or domains if possible.
- Perform regular load tests.

---

6. Security:

Top risks and mitigations:

- Injection Attacks: Always use query builders or ORM to avoid raw queries.
- Authentication flaws: Use secure password hashing (bcrypt), 2FA, and proper token expiration.
- Insecure Headers: Add Helmet middleware for HTTP headers.
- XSS/CSRF: Sanitize inputs, use Content Security Policies, and CSRF tokens where needed.
- Rate limiting & brute-force attacks: As discussed, Redis-based limiter.
- Logging sensitive data: Avoid logging tokens, passwords, or PII.

I also make use of tools like npm audit, snyk, and OWASP ZAP for security testing and dependency scanning.

---

7. Availability to Join:

I’m currently serving a 15-day notice period, which is negotiable. I can join earlier if needed based on the final offer and project urgency.

Let me know if you'd like me to elaborate further or walk you through any of the solutions in a call. Looking forward to the next steps.

Best regards,
Pulkit Singh
Senior Backend Developer
