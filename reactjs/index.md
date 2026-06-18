# Frontend Developer using React.js — Full Roadmap

> A structured, project-driven curriculum to take you from React basics to production-ready applications. Each phase builds on the previous one. Every concept is reinforced with assignments and real projects.

---

## How to Use This Curriculum

- Complete assignments **in order** — each one unlocks the next.
- Build every project **from scratch** — no copy-pasting the implementation files.
- Check off deliverables before moving on.
- Read `HOW_TO_LEARN.md` for the study strategy.

---

## Phase Overview

| Phase | Name | Assignments | Projects | Duration |
|-------|------|-------------|----------|----------|
| 1 | React Fundamentals | A1.1–A1.4 | P1.1–P1.2 | 2 weeks |
| 2 | Intermediate Hooks & Patterns | A2.1–A2.4 | P2.1–P2.2 | 2 weeks |
| 3 | React Router & Navigation | A3.1–A3.4 | P3.1–P3.2 | 1.5 weeks |
| 4 | Data Fetching & Server State | A4.1–A4.4 | P4.1–P4.2 | 2 weeks |
| 5 | Forms & Validation | A5.1–A5.4 | P5.1–P5.2 | 1.5 weeks |
| 6 | Styling & UI | A6.1–A6.4 | P6.1–P6.2 | 2 weeks |
| 7 | Performance | A7.1–A7.4 | P7.1–P7.2 | 1.5 weeks |
| 8 | Testing | A8.1–A8.4 | P8.1–P8.2 | 2 weeks |
| 9 | TypeScript, A11y & Production | A9.1–A9.4 | P9.1–P9.3 | 2 weeks |

**Total: ~16.5 weeks (4 months)**

---

## Phase 1 — React Fundamentals

> Learn the core mental model: components, data flow, side effects.

### Assignments
- **A1.1** — Components & JSX: functional components, JSX rules, fragments, conditional rendering, lists and keys
- **A1.2** — Props & Composition: prop types, default props, children prop, render props, component composition
- **A1.3** — useState & Events: useState hook, functional updates, event handlers, synthetic events, controlled inputs
- **A1.4** — useEffect: side effects, dependency array, cleanup functions, fetch on mount, subscriptions, timers

### Projects
- **P1.1** — Weather App: fetch weather API, current weather + 5-day forecast, search by city, loading/error states
- **P1.2** — Movie Search: OMDB API, results grid, movie detail view, favorites with localStorage

---

## Phase 2 — Intermediate Hooks & Patterns

> Move beyond useState and learn how to manage complex state and share logic.

### Assignments
- **A2.1** — useReducer: reducer function, dispatch, action types, when to prefer over useState
- **A2.2** — useContext: createContext, Provider, useContext, avoiding prop drilling
- **A2.3** — useRef & useMemo/useCallback: DOM access, mutable values, memoization
- **A2.4** — Custom Hooks: extract logic, useLocalStorage, useFetch, useDebounce, useToggle

### Projects
- **P2.1** — Shopping Cart: useReducer for cart, Context for global state, custom useCart hook, localStorage persistence
- **P2.2** — Kanban Board: drag-and-drop with @dnd-kit, useReducer for board state, custom column hooks

---

## Phase 3 — React Router & Navigation

> Build multi-page SPAs with URL-driven state.

### Assignments
- **A3.1** — React Router v6: BrowserRouter, Routes, Route, Link, NavLink, Outlet, nested routes
- **A3.2** — Route Params & Search: useParams, useSearchParams, dynamic segments, URL state for filters
- **A3.3** — Programmatic Navigation: useNavigate, redirect on auth, history manipulation
- **A3.4** — Loaders & Protected Routes: route-level data loading, auth wrappers, 404 handling, lazy routes

### Projects
- **P3.1** — Multi-page SPA: 5+ pages, nested routing, protected routes, 404, breadcrumbs, active nav
- **P3.2** — Search App with URL State: search/filter/sort/page in URL, shareable, browser back/forward

---

## Phase 4 — Data Fetching & Server State

> Stop putting API data in useState. Learn server state management.

### Assignments
- **A4.1** — React Query Basics: QueryClient, useQuery, loading/error/success, queryKey conventions, staleTime
- **A4.2** — Mutations & Cache Updates: useMutation, onSuccess/onError, invalidateQueries, optimistic updates
- **A4.3** — Advanced React Query: useInfiniteQuery, parallel queries, dependent queries, prefetching
- **A4.4** — SWR Alternative: useSWR, revalidation strategies, global config, compare with React Query

### Projects
- **P4.1** — GitHub Profile Explorer: React Query for users/repos/commits, pagination, search
- **P4.2** — Real-time Dashboard: refetchInterval, optimistic updates, error boundary integration

---

## Phase 5 — Forms & Validation

> Handle user input correctly — from simple fields to multi-step flows.

### Assignments
- **A5.1** — React Hook Form: useForm, register, handleSubmit, formState, watch, setValue, reset
- **A5.2** — Zod Validation: z.object(), validators, transform, refine, TypeScript type inference, zodResolver
- **A5.3** — Complex Forms: dynamic fields (useFieldArray), nested forms, multi-step, conditional fields
- **A5.4** — File Uploads & Rich Text: file input, preview, drag-and-drop zone, TipTap editor

### Projects
- **P5.1** — Multi-step Registration: 4 steps, Zod per step, progress indicator, submit with mutation
- **P5.2** — CMS-style Post Editor: rich text, tag input, image upload preview, autosave

---

## Phase 6 — Styling & UI

> Build beautiful, accessible, consistent interfaces at scale.

### Assignments
- **A6.1** — Tailwind CSS with React: className, responsive, dark mode, clsx/cn, Tailwind config
- **A6.2** — Component Variants: class-variance-authority (CVA) for Button/Badge/Input variants
- **A6.3** — shadcn/ui: install, customize, Radix primitives, dialog/dropdown/tooltip/popover
- **A6.4** — Framer Motion: motion.div, variants, AnimatePresence, layout animations, gestures

### Projects
- **P6.1** — UI Component Library: Button, Input, Select, Modal, Toast, Table, Badge + Storybook
- **P6.2** — Animated Portfolio: Framer Motion transitions, scroll animations, dark mode

---

## Phase 7 — Performance

> Make your React apps fast and keep them fast.

### Assignments
- **A7.1** — React.memo & Re-renders: when components re-render, React.memo, React DevTools Profiler
- **A7.2** — useMemo & useCallback in Practice: memoize computations, stable callbacks, when NOT to memoize
- **A7.3** — Code Splitting: React.lazy, Suspense, route-based splitting, Vite bundle analysis
- **A7.4** — Virtualization: react-window, react-virtual, windowing large lists, measuring performance

### Projects
- **P7.1** — Performance Audit: profile an app, fix 3+ unnecessary re-renders, before/after documentation
- **P7.2** — Virtual List: 100,000-row virtualized table, sort/filter, bundle analysis, LCP measurement

---

## Phase 8 — Testing

> Write tests that give you confidence to ship.

### Assignments
- **A8.1** — Jest + React Testing Library: render, screen, userEvent, queries, async testing with waitFor
- **A8.2** — Testing Hooks & Context: renderHook, mock context values, test custom hooks
- **A8.3** — Mock Service Worker: msw handlers, server setup, mock REST and GraphQL
- **A8.4** — Playwright E2E: install, locators, assertions, authenticated flows, CI integration

### Projects
- **P8.1** — Tested React App: 80%+ coverage, RTL for components, MSW for API mocks, Playwright flows
- **P8.2** — Accessible & Tested Component Library: jest-axe, interaction tests, keyboard navigation

---

## Phase 9 — TypeScript, Accessibility & Production

> Ship production-grade React apps: typed, accessible, internationalized, deployed.

### Assignments
- **A9.1** — TypeScript with React: props typing, useState generics, event types, utility types, discriminated unions
- **A9.2** — Accessibility (a11y): ARIA roles/labels, keyboard navigation, focus management, screen reader
- **A9.3** — Internationalization: react-i18next, namespaces, pluralization, date/number formatting
- **A9.4** — Build & Deployment: Vite config, env vars, Docker for static React, GitHub Actions to Vercel

### Projects
- **P9.1** — Fully Typed React App: TypeScript strict mode, no any, typed API responses with Zod
- **P9.2** — Accessible Dashboard: WCAG 2.1 AA, keyboard-navigable, screen reader tested, axe audit
- **P9.3** — Production SPA: TypeScript + testing + i18n + a11y + Docker + CI/CD + performance budget

---

## Technology Stack

| Category | Tool |
|----------|------|
| Build tool | Vite |
| Framework | React 18+ |
| Language | TypeScript |
| Routing | React Router v6 |
| Server state | React Query v5 (TanStack Query) |
| Forms | React Hook Form + Zod |
| Styling | Tailwind CSS + shadcn/ui |
| Animation | Framer Motion |
| Testing | Jest + React Testing Library + MSW + Playwright |
| Drag and drop | @dnd-kit |
| Rich text | TipTap |
| Virtualization | react-virtual (TanStack Virtual) |

---

## Prerequisites

- Comfortable with HTML, CSS, JavaScript (ES2020+)
- Know what a function, array, object, and promise are
- Have Node.js 20+ and pnpm installed

---

## File Index

```
reactjs/
├── index.md              ← you are here
├── HOW_TO_LEARN.md
├── A1.1.md – A1.4.md     Phase 1 assignments
├── A2.1.md – A2.4.md     Phase 2 assignments
├── A3.1.md – A3.4.md     Phase 3 assignments
├── A4.1.md – A4.4.md     Phase 4 assignments
├── A5.1.md – A5.4.md     Phase 5 assignments
├── A6.1.md – A6.4.md     Phase 6 assignments
├── A7.1.md – A7.4.md     Phase 7 assignments
├── A8.1.md – A8.4.md     Phase 8 assignments
├── A9.1.md – A9.4.md     Phase 9 assignments
├── P1.1.md – P1.2.md     Phase 1 projects
├── P2.1.md – P2.2.md     Phase 2 projects
├── P3.1.md – P3.2.md     Phase 3 projects
├── P4.1.md – P4.2.md     Phase 4 projects
├── P5.1.md – P5.2.md     Phase 5 projects
├── P6.1.md – P6.2.md     Phase 6 projects
├── P7.1.md – P7.2.md     Phase 7 projects
├── P8.1.md – P8.2.md     Phase 8 projects
└── P9.1.md – P9.3.md     Phase 9 projects
```
