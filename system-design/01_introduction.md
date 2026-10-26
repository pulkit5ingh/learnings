# 01 — Introduction: System Design Roadmap

A complete, structured, interview-focused roadmap for **Senior Backend Engineer and Tech Lead** interviews. It covers the major topics from the [roadmap.sh System Design roadmap](https://roadmap.sh/system-design), reorganized into a learning path that connects concepts in the right order, removes duplicates, and adds the practical skills interviews expect.

**Approach:** learn the fundamentals first, understand the trade-offs, then study architecture patterns, and finally apply them to real systems.

**Duration:** 10–12 weeks, studying 60–90 minutes daily. With existing Node.js, NestJS, MongoDB, PostgreSQL, Redis and messaging experience, prioritize **architectural reasoning** over basic implementation.

## Roadmap overview

| # | Phase | Weeks | Focus | File |
| --- | --- | --- | --- | --- |
| 1 | Foundations & estimation | 1 | System design process, performance, scalability, latency and throughput | [02](02_foundations_and_estimation.md) |
| 2 | Networking & communication | 2 | HTTP, TCP/UDP, DNS, CDN, REST, RPC, gRPC, GraphQL | [03](03_networking_and_communication.md) |
| 3 | Load balancing & scaling | 3 | Reverse proxies, algorithms, horizontal scaling and service discovery | [04](04_load_balancing_and_scaling.md) |
| 4 | Availability & distributed systems | 4 | CAP, consistency, replication, failover and availability mathematics | [05](05_distributed_systems_consistency_availability.md) |
| 5 | Databases & data architecture | 5–6 | SQL/NoSQL, indexes, sharding, federation and data modeling | [06](06_databases_and_data_architecture.md) |
| 6 | Caching & performance | 7 | Cache strategies, caching layers and performance antipatterns | [07](07_caching_and_performance.md) |
| 7 | Asynchronous systems | 8 | Background jobs, queues, backpressure, event-driven systems and idempotency | [08](08_asynchronous_processing_and_messaging.md) |
| 8 | Cloud & architecture patterns | 9 | Microservices, CQRS, event sourcing, gateways and cloud design patterns | [09](09_cloud_design_and_architecture_patterns.md) |
| 9 | Reliability, security & observability | 10 | Circuit breakers, bulkheads, monitoring, identity and resilience | [10](10_reliability_observability_security.md) |
| 10 | Real-world design practice | 11–12 | Apply everything to production-style systems and interview exercises | [11](11_real_world_system_design.md) |

### Supporting files

- [12 — Additional Topics](12_additional_topics.md): capacity estimation, concurrency, multi-region, IaC, cost and more
- [13 — Interview Framework](13_interview_framework.md): the 8-step approach to use for every design question
- [14 — Portfolio Designs](14_portfolio_designs.md): five portfolio-level designs with their stacks

## Progress tracker

- [ ] 1. Foundations
- [ ] 2. Networking & APIs
- [ ] 3. Load balancing & scaling
- [ ] 4. Distributed systems
- [ ] 5. Databases
- [ ] 6. Caching
- [ ] 7. Asynchronous systems
- [ ] 8. Cloud patterns
- [ ] 9. Reliability & security
- [ ] 10. Real-world design

## The most important concepts to master

Don't just memorize definitions. Learn **when and why** to use each technology.

| Concept | Questions you should be able to answer |
| --- | --- |
| Load balancer | What happens when traffic increases 10×? |
| Redis | How do you prevent stale data and cache stampedes? |
| PostgreSQL | How do you handle millions of records and concurrent writes? |
| MongoDB | When do you embed documents versus reference them? |
| Kafka / RabbitMQ | How do you handle duplicate events and failed consumers? |
| Microservices | How do you maintain consistency across services? |
| Distributed locking | What happens if a lock expires while work is still running? |
| Rate limiting | How do you enforce limits across multiple application instances? |
| Observability | How do you debug a slow request across multiple services? |
| Disaster recovery | What happens when an entire database region fails? |

## Daily study routine (~90 minutes)

- **20 min:** Learn one concept.
- **20 min:** Study a real-world architecture and its trade-offs.
- **35 min:** Draw a design or implement a small component.
- **15 min:** Explain the design aloud as if you're in an interview.

## Recommended resources

- [roadmap.sh — System Design](https://roadmap.sh/system-design): visual overview of the original roadmap.
- [System Design Primer](https://github.com/donnemartin/system-design-primer): open-source concepts, examples and interview questions.
- [Designing Data-Intensive Applications](https://dataintensive.net/): deeper study of databases, replication, distributed systems and reliability.
- [Google SRE Books](https://sre.google/books/): availability, monitoring, incident response and reliability engineering.
- [ByteByteGo](https://bytebytego.com/)

## How to know you're interview-ready

- [ ] Design a system from requirements without relying on a memorized diagram.
- [ ] Estimate traffic, storage and infrastructure needs.
- [ ] Explain why you chose PostgreSQL, MongoDB, Redis or a message broker.
- [ ] Identify bottlenecks and propose solutions without overengineering.
- [ ] Explain how the system behaves during partial failures.
- [ ] Defend your choices and discuss alternatives clearly.

## Recommended starting point

Begin with requirements gathering, performance vs. scalability, latency vs. throughput, and capacity estimation. Then move on to networking, databases, caching and distributed systems.

The biggest differentiator for a backend engineer is explaining **why** you choose an architecture, **how it fails**, and **how it scales**, not just naming the patterns.
