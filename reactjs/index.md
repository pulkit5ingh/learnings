# Frontend Developer using React.js — Full Roadmap

> A structured, project-driven curriculum to take you from React basics to production-ready applications. Each phase builds on the previous one. Every concept is reinforced with assignments and real projects.

---

## How to Use This Curriculum

- Complete assignments **in order** — each one unlocks the next.
- Build every project **from scratch** — no copy-pasting the implementation files.
- Check off deliverables before moving on.
- Read `HOW_TO_LEARN.md` for the study strategy.
- Revise with [q&A.md](q&A.md) — questions and answers for every assignment and project.

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
- **[A1.1](A1.1-components-and-jsx.md)** — Components & JSX: functional components, JSX rules, fragments, conditional rendering, lists and keys
- **[A1.2](A1.2-props-and-composition.md)** — Props & Composition: prop types, default props, children prop, render props, component composition
- **[A1.3](A1.3-usestate-and-events.md)** — useState & Events: useState hook, functional updates, event handlers, synthetic events, controlled inputs
- **[A1.4](A1.4-useeffect.md)** — useEffect: side effects, dependency array, cleanup functions, fetch on mount, subscriptions, timers

### Projects
- **[P1.1](P1.1-weather-app.md)** — Weather App: fetch weather API, current weather + 5-day forecast, search by city, loading/error states
- **[P1.2](P1.2-movie-search.md)** — Movie Search: OMDB API, results grid, movie detail view, favorites with localStorage

---

## Phase 2 — Intermediate Hooks & Patterns

> Move beyond useState and learn how to manage complex state and share logic.

### Assignments
- **[A2.1](A2.1-usereducer.md)** — useReducer: reducer function, dispatch, action types, when to prefer over useState
- **[A2.2](A2.2-usecontext.md)** — useContext: createContext, Provider, useContext, avoiding prop drilling
- **[A2.3](A2.3-useref-and-usememo-usecallback.md)** — useRef & useMemo/useCallback: DOM access, mutable values, memoization
- **[A2.4](A2.4-custom-hooks.md)** — Custom Hooks: extract logic, useLocalStorage, useFetch, useDebounce, useToggle

### Projects
- **[P2.1](P2.1-shopping-cart.md)** — Shopping Cart: useReducer for cart, Context for global state, custom useCart hook, localStorage persistence
- **[P2.2](P2.2-kanban-board.md)** — Kanban Board: drag-and-drop with @dnd-kit, useReducer for board state, custom column hooks

---

## Phase 3 — React Router & Navigation

> Build multi-page SPAs with URL-driven state.

### Assignments
- **[A3.1](A3.1-react-router-v6.md)** — React Router v6: BrowserRouter, Routes, Route, Link, NavLink, Outlet, nested routes
- **[A3.2](A3.2-route-params-and-search.md)** — Route Params & Search: useParams, useSearchParams, dynamic segments, URL state for filters
- **[A3.3](A3.3-programmatic-navigation.md)** — Programmatic Navigation: useNavigate, redirect on auth, history manipulation
- **[A3.4](A3.4-loaders-and-protected-routes.md)** — Loaders & Protected Routes: route-level data loading, auth wrappers, 404 handling, lazy routes

### Projects
- **[P3.1](P3.1-multi-page-spa.md)** — Multi-page SPA: 5+ pages, nested routing, protected routes, 404, breadcrumbs, active nav
- **[P3.2](P3.2-search-app-with-url-state.md)** — Search App with URL State: search/filter/sort/page in URL, shareable, browser back/forward

---

## Phase 4 — Data Fetching & Server State

> Stop putting API data in useState. Learn server state management.

### Assignments
- **[A4.1](A4.1-react-query-basics.md)** — React Query Basics: QueryClient, useQuery, loading/error/success, queryKey conventions, staleTime
- **[A4.2](A4.2-mutations-and-cache-updates.md)** — Mutations & Cache Updates: useMutation, onSuccess/onError, invalidateQueries, optimistic updates
- **[A4.3](A4.3-advanced-react-query.md)** — Advanced React Query: useInfiniteQuery, parallel queries, dependent queries, prefetching
- **[A4.4](A4.4-swr-alternative.md)** — SWR Alternative: useSWR, revalidation strategies, global config, compare with React Query

### Projects
- **[P4.1](P4.1-github-profile-explorer.md)** — GitHub Profile Explorer: React Query for users/repos/commits, pagination, search
- **[P4.2](P4.2-real-time-dashboard.md)** — Real-time Dashboard: refetchInterval, optimistic updates, error boundary integration

---

## Phase 5 — Forms & Validation

> Handle user input correctly — from simple fields to multi-step flows.

### Assignments
- **[A5.1](A5.1-react-hook-form.md)** — React Hook Form: useForm, register, handleSubmit, formState, watch, setValue, reset
- **[A5.2](A5.2-zod-validation.md)** — Zod Validation: z.object(), validators, transform, refine, TypeScript type inference, zodResolver
- **[A5.3](A5.3-complex-forms.md)** — Complex Forms: dynamic fields (useFieldArray), nested forms, multi-step, conditional fields
- **[A5.4](A5.4-file-uploads-and-rich-text.md)** — File Uploads & Rich Text: file input, preview, drag-and-drop zone, TipTap editor

### Projects
- **[P5.1](P5.1-multi-step-registration-form.md)** — Multi-step Registration: 4 steps, Zod per step, progress indicator, submit with mutation
- **[P5.2](P5.2-cms-style-post-editor.md)** — CMS-style Post Editor: rich text, tag input, image upload preview, autosave

---

## Phase 6 — Styling & UI

> Build beautiful, accessible, consistent interfaces at scale.

### Assignments
- **[A6.1](A6.1-tailwind-css-with-react.md)** — Tailwind CSS with React: className, responsive, dark mode, clsx/cn, Tailwind config
- **[A6.2](A6.2-component-variants.md)** — Component Variants: class-variance-authority (CVA) for Button/Badge/Input variants
- **[A6.3](A6.3-shadcn-ui.md)** — shadcn/ui: install, customize, Radix primitives, dialog/dropdown/tooltip/popover
- **[A6.4](A6.4-framer-motion.md)** — Framer Motion: motion.div, variants, AnimatePresence, layout animations, gestures

### Projects
- **[P6.1](P6.1-ui-component-library.md)** — UI Component Library: Button, Input, Select, Modal, Toast, Table, Badge + Storybook
- **[P6.2](P6.2-animated-portfolio.md)** — Animated Portfolio: Framer Motion transitions, scroll animations, dark mode

---

## Phase 7 — Performance

> Make your React apps fast and keep them fast.

### Assignments
- **[A7.1](A7.1-react-memo-and-re-renders.md)** — React.memo & Re-renders: when components re-render, React.memo, React DevTools Profiler
- **[A7.2](A7.2-usememo-and-usecallback-in-practice.md)** — useMemo & useCallback in Practice: memoize computations, stable callbacks, when NOT to memoize
- **[A7.3](A7.3-code-splitting.md)** — Code Splitting: React.lazy, Suspense, route-based splitting, Vite bundle analysis
- **[A7.4](A7.4-virtualization.md)** — Virtualization: react-window, react-virtual, windowing large lists, measuring performance

### Projects
- **[P7.1](P7.1-performance-audit.md)** — Performance Audit: profile an app, fix 3+ unnecessary re-renders, before/after documentation
- **[P7.2](P7.2-virtual-list.md)** — Virtual List: 100,000-row virtualized table, sort/filter, bundle analysis, LCP measurement

---

## Phase 8 — Testing

> Write tests that give you confidence to ship.

### Assignments
- **[A8.1](A8.1-jest-react-testing-library.md)** — Jest + React Testing Library: render, screen, userEvent, queries, async testing with waitFor
- **[A8.2](A8.2-testing-hooks-and-context.md)** — Testing Hooks & Context: renderHook, mock context values, test custom hooks
- **[A8.3](A8.3-mock-service-worker.md)** — Mock Service Worker: msw handlers, server setup, mock REST and GraphQL
- **[A8.4](A8.4-playwright-e2e-testing.md)** — Playwright E2E: install, locators, assertions, authenticated flows, CI integration

### Projects
- **[P8.1](P8.1-tested-react-app.md)** — Tested React App: 80%+ coverage, RTL for components, MSW for API mocks, Playwright flows
- **[P8.2](P8.2-accessible-and-tested-component-library.md)** — Accessible & Tested Component Library: jest-axe, interaction tests, keyboard navigation

---

## Phase 9 — TypeScript, Accessibility & Production

> Ship production-grade React apps: typed, accessible, internationalized, deployed.

### Assignments
- **[A9.1](A9.1-typescript-with-react.md)** — TypeScript with React: props typing, useState generics, event types, utility types, discriminated unions
- **[A9.2](A9.2-accessibility-a11y.md)** — Accessibility (a11y): ARIA roles/labels, keyboard navigation, focus management, screen reader
- **[A9.3](A9.3-internationalization-i18n.md)** — Internationalization: react-i18next, namespaces, pluralization, date/number formatting
- **[A9.4](A9.4-build-and-deployment.md)** — Build & Deployment: Vite config, env vars, Docker for static React, GitHub Actions to Vercel

### Projects
- **[P9.1](P9.1-fully-typed-react-app.md)** — Fully Typed React App: TypeScript strict mode, no any, typed API responses with Zod
- **[P9.2](P9.2-accessible-dashboard.md)** — Accessible Dashboard: WCAG 2.1 AA, keyboard-navigable, screen reader tested, axe audit
- **[P9.3](P9.3-production-spa.md)** — Production SPA: TypeScript + testing + i18n + a11y + Docker + CI/CD + performance budget

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
├── A1.1-components-and-jsx.md                         Phase 1 assignment
├── A1.2-props-and-composition.md                      Phase 1 assignment
├── A1.3-usestate-and-events.md                        Phase 1 assignment
├── A1.4-useeffect.md                                  Phase 1 assignment
├── A2.1-usereducer.md                                 Phase 2 assignment
├── A2.2-usecontext.md                                 Phase 2 assignment
├── A2.3-useref-and-usememo-usecallback.md             Phase 2 assignment
├── A2.4-custom-hooks.md                               Phase 2 assignment
├── A3.1-react-router-v6.md                            Phase 3 assignment
├── A3.2-route-params-and-search.md                    Phase 3 assignment
├── A3.3-programmatic-navigation.md                    Phase 3 assignment
├── A3.4-loaders-and-protected-routes.md               Phase 3 assignment
├── A4.1-react-query-basics.md                         Phase 4 assignment
├── A4.2-mutations-and-cache-updates.md                Phase 4 assignment
├── A4.3-advanced-react-query.md                       Phase 4 assignment
├── A4.4-swr-alternative.md                            Phase 4 assignment
├── A5.1-react-hook-form.md                            Phase 5 assignment
├── A5.2-zod-validation.md                             Phase 5 assignment
├── A5.3-complex-forms.md                              Phase 5 assignment
├── A5.4-file-uploads-and-rich-text.md                 Phase 5 assignment
├── A6.1-tailwind-css-with-react.md                    Phase 6 assignment
├── A6.2-component-variants.md                         Phase 6 assignment
├── A6.3-shadcn-ui.md                                  Phase 6 assignment
├── A6.4-framer-motion.md                              Phase 6 assignment
├── A7.1-react-memo-and-re-renders.md                  Phase 7 assignment
├── A7.2-usememo-and-usecallback-in-practice.md        Phase 7 assignment
├── A7.3-code-splitting.md                             Phase 7 assignment
├── A7.4-virtualization.md                             Phase 7 assignment
├── A8.1-jest-react-testing-library.md                 Phase 8 assignment
├── A8.2-testing-hooks-and-context.md                  Phase 8 assignment
├── A8.3-mock-service-worker.md                        Phase 8 assignment
├── A8.4-playwright-e2e-testing.md                     Phase 8 assignment
├── A9.1-typescript-with-react.md                      Phase 9 assignment
├── A9.2-accessibility-a11y.md                         Phase 9 assignment
├── A9.3-internationalization-i18n.md                  Phase 9 assignment
├── A9.4-build-and-deployment.md                       Phase 9 assignment
├── P1.1-weather-app.md                                Phase 1 project
├── P1.2-movie-search.md                               Phase 1 project
├── P2.1-shopping-cart.md                              Phase 2 project
├── P2.2-kanban-board.md                               Phase 2 project
├── P3.1-multi-page-spa.md                             Phase 3 project
├── P3.2-search-app-with-url-state.md                  Phase 3 project
├── P4.1-github-profile-explorer.md                    Phase 4 project
├── P4.2-real-time-dashboard.md                        Phase 4 project
├── P5.1-multi-step-registration-form.md               Phase 5 project
├── P5.2-cms-style-post-editor.md                      Phase 5 project
├── P6.1-ui-component-library.md                       Phase 6 project
├── P6.2-animated-portfolio.md                         Phase 6 project
├── P7.1-performance-audit.md                          Phase 7 project
├── P7.2-virtual-list.md                               Phase 7 project
├── P8.1-tested-react-app.md                           Phase 8 project
├── P8.2-accessible-and-tested-component-library.md    Phase 8 project
├── P9.1-fully-typed-react-app.md                      Phase 9 project
├── P9.2-accessible-dashboard.md                       Phase 9 project
└── P9.3-production-spa.md                             Phase 9 project
```
