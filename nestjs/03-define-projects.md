# Project definitions

## TravelBaba.ai

### Description

TravelBaba is an AI-powered productivity platform built for travel agents to automate the manual, time-consuming process of creating itineraries. It turns work that often takes hours into roughly a **60-second** flow so agents can generate professional, branded itineraries, quotations, and invoices with much higher efficiency.

### The problem

Small to mid-sized travel agents often struggle with:

- Manual data entry
- Unpolished Word-document quotes
- Hours of research to build custom trips

### The solution

An intelligent ecosystem that uses AI to analyze large amounts of data and provides:

| Pillar                    | What it does                                                               |
| ------------------------- | -------------------------------------------------------------------------- |
| **Smart recommendations** | Automated route and activity planning                                      |
| **Professional branding** | High-quality PDF templates so a home-based agent can look enterprise-grade |
| **Workflow automation**   | One-click generation of vouchers, invoices, and multilingual translations  |

### Key features (talking points)

If asked about product capabilities, focus on:

1. **Hybrid content creation** — AI mode (speed) and manual mode (precision) for different agent needs.
2. **Financial management** — Not only planning: dynamic pricing, payment receipts, and service vouchers.
3. **Localization** — Multilingual support (e.g. translating a Kerala quote into Hindi/Marathi) for regional markets.
4. **Omnichannel sharing** — WhatsApp integration so agents reach customers where they already are.

### Tech stack

**Frontend**

- Next.js (App Router) in a **Turborepo** monorepo — shared UI packages, env handling, and build caching across apps.
- **Styling & UI:** Tailwind CSS, shadcn/ui, Radix primitives (accessible components).
- **State & data:** Zustand (client state), TanStack Query (server cache, retries, stale-while-revalidate).
- **Forms & validation:** React Hook Form + Zod (schema-driven validation shared with backend DTOs where possible).
- **HTTP:** Axios (interceptors for auth headers, refresh flow, global error handling).
- **Auth:** JWT access tokens + refresh-token rotation (short-lived access, longer-lived refresh, secure httpOnly cookies or secure storage patterns as designed).

**Backend**

- **NestJS microservices** — domain-driven modules per bounded context; REST (and optional internal gRPC) between services.
- **API docs:** Swagger/OpenAPI on each service for contracts and handoffs to the frontend.
- **Auth:** Passport JWT strategies, guards, role-based access (RBAC) for admin vs agent.
- **Queues & jobs:** Bull/BullMQ (often backed by Redis) for PDF generation, email sends, heavy AI calls, and webhook retries.
- **Real-time (if used):** WebSockets (Socket.IO or native WS gateway) for notifications or live collaboration.

**Data & storage**

- **MongoDB** — primary document store for itineraries, quotes, users, and flexible nested content (packages, line items, translations).
- **Redis** — sessions, caching hot reads (e.g. templates, config), rate limiting, and Bull queue backing.
- **Search (optional):** MongoDB text indexes or OpenSearch/Elasticsearch for hotel/activity search at scale.

**Media & files**

- **AWS S3** — PDFs, images, brand assets; presigned URLs for secure direct uploads from the browser; lifecycle rules for old exports.
- **CloudFront (optional)** — CDN in front of S3 for faster PDF/image delivery to agents and customers.

**Email**

- **Nodemailer** — transactional mail (quotes, invoices, OTPs) with HTML templates; often paired with SES or another SMTP provider in production.

**Third-party integrations**

- **Twilio** — SMS OTP, transactional SMS, sometimes WhatsApp Business API for notifications.
- **Anthropic Claude** — itinerary copy, summarization, multilingual drafts, structured JSON outputs for forms.
- **Puppeteer** — headless Chrome for PDF rendering from HTML templates (quotes, vouchers) when pixel-perfect output matters.
- **Razorpay** — payments, webhooks for payment success/failure, reconciliation with invoices.

**Infrastructure & ops (typical for this stack)**

- **AWS:** ECS/EKS or EC2 + ALB, VPC, IAM roles for services (no long-lived keys on servers), Secrets Manager for API keys.
- **CI/CD:** GitHub Actions (lint, test, build, deploy), environment-specific configs (dev/staging/prod).
- **Observability:** Structured logging (Pino/Winston), CloudWatch or similar; optional Sentry for errors; health checks per service for load balancers.

### Services / modules (counts)

**Domain / feature areas (examples)**

- Quotation, Packages, Itineraries, Activities, Terms & conditions
- Templates, AI services, Invoice, Customers, Hotels

**Module buckets**

- **7** user-related modules (e.g. Admin, Auth, User, Role, …)
- **7** sales/commercial modules (e.g. Product, Category, Lead, Subscription, …)
- **12** utility and infrastructure modules (e.g. Config, Core, Webhooks, i18n, …)

---

## Quiksale

### Tech stack

**Frontend**

- Next.js (Turborepo monorepo)
- Tailwind CSS, Zustand, TanStack Query, React Hook Form, Axios, Zod
- JWT authentication with refresh-token rotation
- shadcn/ui

**Backend**

- NestJS mondular monolothics
-

### Services / modules (counts)

- **7** user-related modules (Admin, Auth, User, Role, …)
- **7** sales/commercial modules (Product, Category, Lead, Subscription, …)
- **12** utility and infrastructure modules (Config, Core, Webhooks, i18n, …)
- **Total:** approximately **30–40** services
