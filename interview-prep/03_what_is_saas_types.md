# What is SaaS & Types of SaaS

*Quick reference for interview prep.*

---

## What is SaaS?

**SaaS (Software as a Service)** is a software delivery model where:

- The application is **hosted in the cloud** by the provider.
- Customers **access it over the internet** (browser or client app), usually via subscription.
- The provider handles **hosting, updates, security, and scaling**; customers don’t install or maintain the software on their own servers.

**In short:** Pay to use software online instead of buying and running it yourself.

---

### SaaS vs other models

| Model | Who hosts/runs it | Typical payment | Example |
|-------|-------------------|------------------|---------|
| **On-premise** | Customer’s own servers | Licence + maintenance | Old enterprise ERP in your data centre |
| **IaaS** (Infrastructure) | Provider gives VMs, storage, network | Pay for resources | AWS EC2, GCP Compute |
| **PaaS** (Platform) | Provider gives runtime + DB + tools | Pay for platform use | Heroku, AWS Elastic Beanstalk |
| **SaaS** | Provider runs the full application | Subscription (monthly/yearly) | Slack, Notion, Microspace |

---

### Why SaaS matters (for interviews)

- **Recurring revenue** for the company (subscriptions).
- **Multi-tenant** architecture: one codebase, many customers (tenants), often with shared infra.
- **B2B SaaS**: sold to businesses (teams, companies) — complex roles, workflows, integrations, and sometimes SSO/enterprise features.
- **Product-led**: focus on usage, onboarding, and retention, not one-off projects.

---

## Types of SaaS

SaaS can be grouped in several ways. Below are the most useful for interviews.

---

### 1. By customer (who buys it)

| Type | Customer | Examples | Typical traits |
|------|----------|----------|-----------------|
| **B2B SaaS** | Businesses, teams, organisations | Slack, Notion, Microspace, Salesforce | Multiple users, roles, permissions, SSO, contracts, compliance |
| **B2C SaaS** | Individual consumers | Netflix, Spotify, Dropbox | Single user or family, simpler billing, high volume |
| **B2B2C** | Business buys, end users are consumers | Fitness apps white-labelled for gyms, learning apps for schools | Two “customers”: buyer (org) and end user |

**Microspace and your background:** B2B SaaS (customer-facing teams, branded spaces).

---

### 2. By vertical (industry)

| Vertical | Focus | Examples |
|----------|--------|----------|
| **Horizontal** | Broad use across industries | Slack, Zoom, Notion, CRM, HR tools |
| **Vertical** | Deep in one industry | Veeva (pharma), Procore (construction), Toast (restaurants), EMR (healthcare) |

Your experience: travel (Tripinnov, TravelBaba), foodtech (Ordercoro, Robofood), healthcare (Frontliner, EMR at DandD), photography (Fliqa).

---

### 3. By function (what it does)

| Function | Purpose | Examples |
|----------|---------|----------|
| **Collaboration & communication** | Messaging, meetings, docs, spaces | Slack, Notion, Microspace |
| **CRM** | Sales, pipeline, contacts | Salesforce, HubSpot |
| **HR / People** | Hiring, payroll, performance | BambooHR, Lattice |
| **Finance / billing** | Invoicing, subscriptions, payments | Stripe Billing, Chargebee |
| **Analytics / BI** | Dashboards, reports, data | Looker, Mixpanel, Amplitude |
| **Dev / DevOps** | Code, CI/CD, infra | GitHub, Vercel, Datadog |
| **Marketing / growth** | Campaigns, email, SEO | Mailchimp, HubSpot Marketing |
| **Support / helpdesk** | Tickets, knowledge base, chat | Zendesk, Intercom |

---

### 4. By pricing / packaging

| Model | How customers pay | Example |
|-------|-------------------|--------|
| **Subscription (recurring)** | Monthly or yearly | Most SaaS (Slack, Notion) |
| **Usage-based** | By usage (API calls, seats, storage) | Twilio, AWS, some tiers of Slack |
| **Freemium** | Free tier + paid upgrade | Notion, Slack, Zoom |
| **Tiered** | Plans (Starter / Pro / Enterprise) | Common in B2B |
| **Per-seat** | Price per user | Slack, Figma |

---

### 5. By technical / business model

| Concept | Meaning |
|--------|--------|
| **Multi-tenant** | One deployment serves many customers (tenants); data and config separated by tenant (org/workspace). |
| **Single-tenant** | Dedicated instance or DB per customer (often in enterprise). |
| **Product-led growth (PLG)** | Users try the product first (free/trial), then convert to paid. |
| **Sales-led** | Sales team drives demos, contracts, and onboarding (common in enterprise B2B). |

---

## Other important SaaS details

### Key business metrics (good to know)

| Term | Meaning |
|------|--------|
| **MRR** | Monthly Recurring Revenue — predictable income per month from subscriptions. |
| **ARR** | Annual Recurring Revenue — MRR × 12 (or sum of annual contracts). |
| **Churn** | Rate at which customers cancel or don’t renew (user churn vs revenue churn). |
| **LTV** | Customer Lifetime Value — revenue from a customer over their entire relationship. |
| **CAC** | Customer Acquisition Cost — cost to acquire one customer. |
| **Trial / POC** | Free or limited period before paid; common in B2B. |

### Technical / product concepts

| Concept | Why it matters in SaaS |
|--------|-------------------------|
| **Tenant isolation** | Data and config for one customer (tenant) must not leak to others; critical for multi-tenant. |
| **API-first** | APIs are the product or a core part of it; enables integrations, mobile apps, and partners. |
| **Webhooks** | Outbound events so customers’ systems can react (e.g. payment success, user created). |
| **Idempotency** | Same request can be retried safely (e.g. payments, webhooks) — fewer duplicates and support issues. |
| **Rate limiting & quotas** | Per-tenant or per-user limits; ties to pricing tiers and fair usage. |
| **Feature flags** | Turn features on/off per tenant or segment without new deployments. |
| **Audit logs** | Who did what, when — required for compliance and support in B2B. |

### Security & compliance (often asked in B2B)

| Area | What it means |
|------|----------------|
| **SSO / SAML / OIDC** | Single Sign-On; enterprise customers use their IdP (e.g. Okta, Azure AD). |
| **RBAC / ABAC** | Role-Based or Attribute-Based Access Control — who can do what in the product. |
| **Data residency** | Where data is stored (region/country) — matters for GDPR, contracts. |
| **Encryption** | At rest (e.g. DB, S3) and in transit (HTTPS/TLS). |
| **GDPR, SOC 2, HIPAA** | Privacy and security standards; enterprise often requires compliance. |

### Why multi-tenant matters

- **One codebase** to develop, test, and deploy for all customers.
- **Shared infrastructure** — lower cost per customer, easier to scale.
- **Faster onboarding** — new customer = new tenant (org/workspace), not new servers.
- **Trade-off:** Strong tenant isolation and “noisy neighbour” prevention (e.g. per-tenant rate limits, DB design).

---

## My projects: how each is SaaS

How each of your projects fits the SaaS model — useful for “tell me about a SaaS you built” type questions.

| Project | SaaS type (B2B/B2C/B2B2C) | Vertical | Function | How it’s SaaS |
|---------|----------------------------|----------|----------|----------------|
| **TravelBaba.ai** | B2B | Travel | AI / productivity | Software for **travel agents** (businesses); cloud-hosted, subscription or usage-based; agents use it as a service to generate itineraries and quotes. |
| **Quiksale** | B2C (or B2B2C if sellers are small businesses) | Resale / hyperlocal | Marketplace + subscriptions | **Consumers** use the platform as a service (listing, discovery, referrals); subscription or transaction-based revenue; cloud-hosted. |
| **Tripinnov** | B2B | Travel | Operations / CRM-like | **Travel agents** subscribe; itinerary management, quotations, commission tracking; multi-tenant, roles, workflows — classic B2B vertical SaaS. |
| **Ordercoro** | B2B (multi-sided) | Foodtech | Operations / ecosystem | **Restaurants, vendors, delivery** use the same platform as a service; multi-role, multi-tenant; subscription or commission; complex workflows. |
| **Robofood** | B2B | Foodtech / cloud kitchen | Operations (POS, inventory, delivery) | **Cloud kitchens** use it as their operating system; POS, billing, inventory, delivery APIs — software delivered as a service, typically subscription. |
| **ShiftPro** | B2B2C | Labour / staffing | Marketplace + operations | **Merchants** (B2B) and **helpers** (B2C) use the platform; SaaS for hiring and earning; subscription or commission; recurring bookings, payments. |
| **BakBak** | B2C + local businesses | Hyperlocal / community | Social + local services + civic | **Users** and **local businesses** use the platform; feed, services, “Mudda,” Dukaan — software as a service for local life and discovery. |
| **Frontliner** | B2B | Healthcare | EMR + staffing | **Healthcare orgs** use it for patient–caregiver management and staffing; multi-tenant, roles, compliance-sensitive — vertical B2B SaaS. |
| **Fliqa India** | B2B + B2C | Photography / creative | Platform + content | **Photographers/creatives** and **clients** use the platform; cloud-hosted (Cloudflare), admin panel, media (Cloudinary) — platform as a service. |

### One-line “how it’s SaaS” per project (for quick answers)

- **TravelBaba.ai:** B2B SaaS for travel agents — AI-powered itineraries and quotes, delivered as a cloud service.
- **Quiksale:** B2C/hyperlocal SaaS — resale platform with subscriptions and referrals, used as a service.
- **Tripinnov:** B2B vertical SaaS for travel agents — itineraries, quotations, commissions; multi-tenant, roles.
- **Ordercoro:** B2B multi-sided SaaS — one platform for restaurants, vendors, and delivery; subscription/commission.
- **Robofood:** B2B vertical SaaS for cloud kitchens — POS, inventory, delivery APIs as a service.
- **ShiftPro:** B2B2C marketplace SaaS — merchants and helpers use the platform; recurring bookings, payments.
- **BakBak:** B2C + local business SaaS — hyperlocal social, services, and civic features as a service.
- **Frontliner:** B2B healthcare SaaS — EMR and staffing for patient–caregiver management.
- **Fliqa India:** B2B/B2C platform SaaS — photography/creative platform with admin and media, cloud-hosted.

---

## One-line definitions for interviews

- **SaaS:** Cloud-hosted software sold as a subscription; provider runs and maintains it.
- **B2B SaaS:** Software sold to companies; usually multi-user, roles, workflows, and often SSO/integrations.
- **Multi-tenant:** One application serving many customers with isolated data per tenant.
- **Vertical SaaS:** SaaS built for one industry (e.g. healthcare, travel, foodtech).

Use this file to speak clearly about “what is SaaS” and “what types of SaaS” you’ve worked on (e.g. B2B, vertical, multi-tenant).
