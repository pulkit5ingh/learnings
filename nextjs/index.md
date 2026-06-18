# Frontend Developer using Next.js — Full Curriculum

**Stack:** Next.js 14+ (App Router) · TypeScript · Tailwind CSS  
**Duration:** ~6 months (part-time) or ~3 months (full-time)  
**Goal:** Go from zero to production-ready Next.js developer

---

## Roadmap Overview

| Phase | Name | Assignments | Projects |
|-------|------|-------------|----------|
| 1 | Next.js & App Router Fundamentals | A1.1–A1.4 | P1.1–P1.2 |
| 2 | Data Fetching & Rendering | A2.1–A2.4 | P2.1–P2.2 |
| 3 | Styling | A3.1–A3.4 | P3.1–P3.2 |
| 4 | State Management & Client Data Fetching | A4.1–A4.4 | P4.1–P4.2 |
| 5 | Authentication | A5.1–A5.4 | P5.1–P5.2 |
| 6 | Performance & Core Web Vitals | A6.1–A6.4 | P6.1–P6.2 |
| 7 | Testing | A7.1–A7.4 | P7.1–P7.2 |
| 8 | Deployment & CI/CD | A8.1–A8.4 | P8.1–P8.2 |
| 9 | Advanced Features | A9.1–A9.4 | P9.1–P9.3 |

---

## Phase 1 — Next.js & App Router Fundamentals
**Duration:** 1.5 weeks  
**Prerequisites:** React basics, TypeScript basics

### What you will learn
- How the App Router's file-system conventions work
- The difference between Server and Client Components and why it matters
- All routing strategies: static, dynamic, catch-all, parallel, intercepting
- Configuration, Metadata API, SEO

### Assignments
| File | Topic |
|------|-------|
| A1.1 | App Router structure — app/ directory, layout.tsx, page.tsx, loading.tsx, error.tsx, not-found.tsx |
| A1.2 | Server vs Client Components — "use client", serialization, composition patterns |
| A1.3 | Routing — static, dynamic [id], catch-all [...slug], parallel @slot, intercepting (.) |
| A1.4 | Next.js config & Metadata API — next.config.js, generateMetadata, OG tags, sitemap |

### Projects
| File | Topic |
|------|-------|
| P1.1 | Portfolio site — App Router, metadata, dynamic project pages, 404 |
| P1.2 | Documentation site — MDX, nested routing, sidebar, table of contents |

---

## Phase 2 — Data Fetching & Rendering
**Duration:** 1.5 weeks  
**Prerequisites:** Phase 1 complete

### What you will learn
- fetch() with cache semantics in Server Components
- Static Generation, Server-Side Rendering, Incremental Static Regeneration
- Server Actions for mutations
- Route Handlers (the App Router replacement for API routes)

### Assignments
| File | Topic |
|------|-------|
| A2.1 | Server Component data fetching — fetch(), cache options, async components |
| A2.2 | Static vs Dynamic rendering — SSG, SSR, ISR, generateStaticParams |
| A2.3 | Server Actions — "use server", form actions, useFormState, useOptimistic |
| A2.4 | Route Handlers — app/api/ route.ts, HTTP methods, NextResponse, streaming |

### Projects
| File | Topic |
|------|-------|
| P2.1 | Blog with ISR — MDX posts, generateStaticParams, revalidation, RSS feed |
| P2.2 | Dashboard with Server Actions — form submit, DB writes, optimistic UI |

---

## Phase 3 — Styling
**Duration:** 1 week  
**Prerequisites:** Phase 1 complete (can run in parallel with Phase 2)

### What you will learn
- Tailwind CSS from utility classes through custom config
- Component variant patterns with CVA
- CSS Modules, CSS variables, next/font
- Animation with Framer Motion

### Assignments
| File | Topic |
|------|-------|
| A3.1 | Tailwind CSS fundamentals — utilities, responsive, dark mode, custom config |
| A3.2 | Tailwind component patterns — CVA, cn() with clsx + tailwind-merge |
| A3.3 | CSS Modules & global styles — module.css, CSS variables, next/font |
| A3.4 | Animation — Tailwind animate, Framer Motion variants, AnimatePresence |

### Projects
| File | Topic |
|------|-------|
| P3.1 | Landing page — marketing page, responsive, dark mode, animations |
| P3.2 | Component library — 10+ reusable components with CVA variants |

---

## Phase 4 — State Management & Client Data Fetching
**Duration:** 1.5 weeks  
**Prerequisites:** Phases 1–2 complete

### What you will learn
- TanStack Query for server-state on the client
- Zustand for client-state
- React Hook Form + Zod for forms
- URL state with useSearchParams and nuqs

### Assignments
| File | Topic |
|------|-------|
| A4.1 | React Query — useQuery, useMutation, invalidation, optimistic updates |
| A4.2 | Zustand — stores, actions, persist middleware, slice pattern |
| A4.3 | React Hook Form — register, Controller, Zod validation, nested fields |
| A4.4 | URL state — useSearchParams, useRouter, nuqs for type-safe URL params |

### Projects
| File | Topic |
|------|-------|
| P4.1 | Product listing — React Query + Zustand cart + URL filter/sort/page |
| P4.2 | Multi-step form — RHF + Zod, step navigation, persistence, mutation |

---

## Phase 5 — Authentication
**Duration:** 1 week  
**Prerequisites:** Phases 1–4 complete

### What you will learn
- NextAuth.js v5 (Auth.js) setup and providers
- Protecting routes with middleware
- JWT vs database sessions, accessing session everywhere
- Full auth flow patterns

### Assignments
| File | Topic |
|------|-------|
| A5.1 | NextAuth.js v5 — setup, providers (Google/GitHub/Credentials), callbacks |
| A5.2 | Protected routes — middleware.ts, role-based routing, auth() helper |
| A5.3 | Session & tokens — JWT vs DB sessions, accessing session server/client |
| A5.4 | Auth patterns — login, signup, logout, refresh, OAuth linking |

### Projects
| File | Topic |
|------|-------|
| P5.1 | Full-stack app with auth — NextAuth, Google + credentials, protected dashboard |
| P5.2 | Role-based access — admin/user roles, middleware guards, access denied |

---

## Phase 6 — Performance & Core Web Vitals
**Duration:** 1 week  
**Prerequisites:** Phases 1–5 complete

### What you will learn
- next/image for optimized images
- Code splitting with dynamic() and Suspense
- Core Web Vitals measurement and improvement
- Bundle analysis and dependency optimization

### Assignments
| File | Topic |
|------|-------|
| A6.1 | next/image — sizes, priority, blur placeholder, remote patterns |
| A6.2 | Code splitting — dynamic() import, Suspense, loading skeletons |
| A6.3 | Core Web Vitals — LCP, INP, CLS — measure, diagnose, fix |
| A6.4 | Bundle analysis — @next/bundle-analyzer, tree shaking, font/script opt |

### Projects
| File | Topic |
|------|-------|
| P6.1 | Performance-optimized image gallery — LCP < 2.5 s |
| P6.2 | Optimized e-commerce page — bundle analysis, code splitting, CWV report |

---

## Phase 7 — Testing
**Duration:** 1.5 weeks  
**Prerequisites:** Phases 1–6 complete

### What you will learn
- Jest + Testing Library for unit/integration tests
- MSW for mocking APIs
- Playwright for end-to-end testing
- Testing Next.js-specific patterns (navigation, Server Actions, Route Handlers)

### Assignments
| File | Topic |
|------|-------|
| A7.1 | Jest + Testing Library — render, screen, userEvent, async queries |
| A7.2 | Component testing — interactions, forms, MSW mock API calls |
| A7.3 | Playwright e2e — locators, fill/click, assertions, isolation |
| A7.4 | Testing Next.js — mock navigation, Server Actions, Route Handlers |

### Projects
| File | Topic |
|------|-------|
| P7.1 | Tested component library — unit tests, interaction tests, jest-axe |
| P7.2 | e2e tested user flow — signup→login→CRUD→logout Playwright suite |

---

## Phase 8 — Deployment & CI/CD
**Duration:** 1 week  
**Prerequisites:** Phases 1–7 complete

### What you will learn
- Vercel deployment, preview deployments, environment variables
- Docker with standalone output mode
- GitHub Actions pipeline (lint, typecheck, test, build, deploy)
- Environment variable management strategies

### Assignments
| File | Topic |
|------|-------|
| A8.1 | Vercel deployment — push-to-deploy, env vars, preview, domains |
| A8.2 | Docker — Dockerfile standalone, nginx reverse proxy, docker-compose |
| A8.3 | GitHub Actions — ESLint, tsc, Jest, build, deploy to Vercel |
| A8.4 | Environment management — .env files, NEXT_PUBLIC_, server-only vars |

### Projects
| File | Topic |
|------|-------|
| P8.1 | Deployed Next.js app — Vercel production, custom domain, preview branches |
| P8.2 | Docker + CI/CD — Dockerfile standalone, GitHub Actions build+push+deploy |

---

## Phase 9 — Advanced Features
**Duration:** 2 weeks  
**Prerequisites:** Phases 1–8 complete

### What you will learn
- Internationalization with next-intl
- Streaming SSR and Partial Prerendering
- Real-time features with WebSockets and SSE
- PWA / offline capabilities

### Assignments
| File | Topic |
|------|-------|
| A9.1 | Internationalization — next-intl, locale routing, message dictionaries |
| A9.2 | Streaming & Suspense — streaming SSR, waterfall avoidance, PPR |
| A9.3 | Real-time with WebSockets — Socket.io, SSE for server push |
| A9.4 | PWA & offline — next-pwa, service worker, manifest, install prompt |

### Projects
| File | Topic |
|------|-------|
| P9.1 | i18n-ready SaaS app — EN + one other language, locale switch, RTL |
| P9.2 | Real-time dashboard — SSE/WebSocket live data, streaming table, charts |
| P9.3 | Full-stack SaaS template — auth, Stripe billing, dashboard, settings, teams |

---

## Recommended Learning Order

```
Week 1-2:   Phase 1 (App Router Fundamentals)
Week 3-4:   Phase 2 (Data Fetching) + Phase 3 (Styling) in parallel
Week 5-6:   Phase 4 (State Management)
Week 7:     Phase 5 (Authentication)
Week 8:     Phase 6 (Performance)
Week 9-10:  Phase 7 (Testing)
Week 11:    Phase 8 (Deployment & CI/CD)
Week 12-13: Phase 9 (Advanced Features)
```

---

## Tools & Libraries Reference

| Category | Library | Version |
|----------|---------|---------|
| Framework | next | 14.x |
| Language | typescript | 5.x |
| Styling | tailwindcss | 3.x |
| State | zustand | 4.x |
| Server state | @tanstack/react-query | 5.x |
| Forms | react-hook-form | 7.x |
| Validation | zod | 3.x |
| Auth | next-auth | 5.x (beta) |
| Animation | framer-motion | 11.x |
| Testing | jest, @testing-library/react | latest |
| e2e | @playwright/test | latest |
| i18n | next-intl | 3.x |
