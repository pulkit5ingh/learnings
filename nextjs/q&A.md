# Next.js — Questions & Answers

Questions and answers drawn from every file in this folder, in curriculum order. Each section links back to its source file.

- [Phase 1 — Next.js & App Router Fundamentals](#phase-1--nextjs--app-router-fundamentals)
- [Phase 2 — Data Fetching & Rendering](#phase-2--data-fetching--rendering)
- [Phase 3 — Styling](#phase-3--styling)
- [Phase 4 — State Management & Client Data Fetching](#phase-4--state-management--client-data-fetching)
- [Phase 5 — Authentication](#phase-5--authentication)
- [Phase 6 — Performance & Core Web Vitals](#phase-6--performance--core-web-vitals)
- [Phase 7 — Testing](#phase-7--testing)
- [Phase 8 — Deployment & CI/CD](#phase-8--deployment--cicd)
- [Phase 9 — Advanced Features](#phase-9--advanced-features)
- [Study Strategy & Debugging](#study-strategy--debugging)

---

## Phase 1 — Next.js & App Router Fundamentals

### [A1.1 — App Router Structure](A1.1-app-router-structure.md)

**Q: What is the App Router?**
A: A file-system-based router built on React Server Components. Every folder inside `app/` that contains a `page.tsx` becomes a public URL.

**Q: What are co-located special files?**
A: Filenames Next.js reserves with special meaning inside `app/` — `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`.

**Q: How does `layout.tsx` differ from a page?**
A: Layouts wrap one or more pages and persist across navigations — moving from `/about` to `/contact` doesn't re-render the root layout. That makes them the place for nav bars, fonts and theme providers. Layouts can be nested (`app/dashboard/layout.tsx` wraps only dashboard routes).

**Q: How does `loading.tsx` work?**
A: It's an automatic Suspense boundary: Next.js wraps `page.tsx` in `<Suspense>` with `loading.tsx` as the fallback, so the UI shell stays interactive while content loads.

**Q: What are the rules for `error.tsx`?**
A: It must be a Client Component (`"use client"`), receives `error` and `reset` props, and activates when an uncaught error is thrown in any Server or Client Component in the segment.

**Q: When does `not-found.tsx` render?**
A: When `notFound()` is called from `page.tsx` or any component it renders.

**Q: What are route groups?**
A: Folders in parentheses like `(marketing)` that organise files without affecting the URL.

### [A1.2 — Server vs Client Components](A1.2-server-vs-client-components.md)

**Q: What is the default component type in the App Router?**
A: Server Components. They render only on the server, can be `async` and `await` database queries or `fetch()`, and their code never ships to the browser — shrinking the JS bundle.

**Q: What does `"use client"` do?**
A: Marks a file as a Client Component. It renders on the server first (SSR/hydration) and again on the client, and can use browser APIs, hooks like `useState`/`useEffect`, and event handlers.

**Q: Why is `"use client"` described as contagious?**
A: It creates a boundary — everything a Client Component imports is also treated as a Client Component, even without the directive.

**Q: What is the serialisation rule?**
A: Props crossing the Server → Client boundary must be serialisable. Strings, numbers, plain objects, arrays and `null`/`undefined` are fine; functions, class instances and `Date` objects are not (unless converted to strings).

**Q: How can a Client Component wrap Server Components without making them client-side?**
A: Pass the Server Components as `children`. The Client Component doesn't pull its children into the client bundle — e.g. a context provider can wrap Server Components and they stay on the server.

### [A1.3 — Routing](A1.3-routing.md)

**Q: How do dynamic segments work?**
A: Square brackets — `[id]` — with the value available as `params.id`. A path can have several, e.g. `app/[locale]/blog/[slug]/page.tsx` matches `/en/blog/hello-world`.

**Q: What is a catch-all segment?**
A: `[...slug]` matches one or more segments (`/docs/intro`, `/docs/getting-started/install`), and `slug` is an array of strings.

**Q: What is an optional catch-all?**
A: `[[...slug]]` — like a catch-all but also matches the root (`/docs`).

**Q: What are parallel routes?**
A: `@folder` slots that a layout renders at the same time, useful for dashboards whose sections load independently. Each slot can have its own `loading.tsx`, `error.tsx` and sub-routes.

**Q: What are intercepting routes?**
A: Routes that show another route inside the current layout context — e.g. clicking a photo opens it in a modal, while visiting `/photos/1` directly shows the full page. `(.)` intercepts a sibling, `(..)` a parent, `(...)` from the root.

### [A1.4 — Next.js Config & Metadata](A1.4-nextjs-config-and-metadata.md)

**Q: What does `next.config.js` control?**
A: Central project configuration — image domains, redirects, rewrites, environment variable exposure, experimental features and more.

**Q: What replaces the old `<Head>` component in the App Router?**
A: The Metadata API: export a `metadata` object from a `layout.tsx` or `page.tsx` and Next.js injects the `<meta>` tags.

**Q: How do you generate metadata dynamically?**
A: Export an async `generateMetadata` function, which receives the same `params` as the page (e.g. to set a blog post's title from its slug).

**Q: What is OpenGraph metadata for?**
A: Controlling how pages look when shared on social platforms — chiefly `og:title`, `og:description`, `og:image` and `og:url`. Next.js can generate OG images from `/opengraph-image.tsx` with the `ImageResponse` API.

**Q: How are `robots.txt` and `sitemap.xml` generated?**
A: From `robots.ts` and `sitemap.ts` files in `app/`, which export functions returning typed objects that Next.js serialises.

### [P1.1 — Portfolio Site](P1.1-portfolio-site.md)

**Q: What does this project build?**
A: A Next.js 14 portfolio with a home page (hero, about, skills), a projects list with dynamic `[slug]` pages, a static contact page, and a custom 404.

**Q: How are SEO and shared UI handled?**
A: `generateMetadata` on every page (title, description, OG tags) and a `layout.tsx` with a shared header and footer.

**Q: Which special file provides the 404 page?**
A: A custom `not-found.tsx`.

### [P1.2 — Documentation Site](P1.2-documentation-site.md)

**Q: How is content authored and routed?**
A: As MDX files under `content/docs/`, served at the nested route `/docs/[section]/[slug]`.

**Q: Which library renders the MDX?**
A: `next-mdx-remote`, with custom components — internal links use Next.js `Link`, and code blocks are wrapped to add a copy button.

**Q: What navigation features does it include?**
A: A persistent sidebar of all sections and pages, a table of contents generated from MDX headings, syntax-highlighted code blocks (e.g. `rehype-pretty-code`), and optional search.

---

## Phase 2 — Data Fetching & Rendering

### [A2.1 — Server Component Data Fetching](A2.1-server-component-data-fetching.md)

**Q: How does Next.js extend `fetch()`?**
A: It intercepts `fetch()` in Server Components and adds caching and deduplication — which in turn decides how pages are rendered.

**Q: What do the `cache` options do?**
A: `force-cache` (default) caches indefinitely, making the page static at build time if all fetches use it. `no-store` never caches, making the page dynamic. `{ next: { revalidate: 60 } }` caches for 60 s and then refetches in the background (stale-while-revalidate) — this is ISR.

**Q: What is fetch deduplication?**
A: If the same URL is fetched more than once in a render tree, only one HTTP request is made — so you can call `getUser()` in both a layout and a page.

**Q: How do you deduplicate your own data functions?**
A: Wrap them with `cache` from `react` — the request memoisation mechanism behind fetch deduplication.

### [A2.2 — Static vs Dynamic Rendering](A2.2-static-vs-dynamic-rendering.md)

**Q: What is static rendering (SSG)?**
A: The page is rendered once at build time and served from a CDN — the default when there's no dynamic data. Zero server compute per request.

**Q: What forces dynamic rendering (SSR)?**
A: Using `cookies()`, `headers()`, `searchParams` in a page, or `fetch` with `no-store`. The HTML is generated per request.

**Q: What is ISR?**
A: Incremental Static Regeneration: a static page that Next.js re-renders in the background after `revalidate` seconds, so visitors always get a fast cached response that eventually updates.

**Q: What does `generateStaticParams` do?**
A: Lists the values of a dynamic route (e.g. slugs) so Next.js can pre-render them at build time. Without it, the route renders dynamically at request time.

**Q: What are the `dynamic` route export options?**
A: `"force-static"` forces static even with dynamic APIs; `"force-dynamic"` always renders dynamically; `"error"` fails the build if dynamic APIs are detected.

### [A2.3 — Server Actions](A2.3-server-actions.md)

**Q: What are Server Actions?**
A: Async functions that run on the server but are called from the client, removing the need for separate API routes for mutations.

**Q: How do you define one?**
A: With `"use server"` at the top of a file (every export becomes an action) or at the top of an async function inside a Server Component.

**Q: How do Server Actions connect to forms?**
A: Through the `<form>` `action` attribute. On submit the browser sends a POST (or React intercepts it, for progressive enhancement), and the action runs with full access to databases, secrets and the file system.

**Q: `useFormState` vs `useFormStatus`?**
A: `useFormState` wraps a Server Action and returns its result as state. `useFormStatus` must be used in a child of the form and reports whether the form is pending.

**Q: What does `useOptimistic` do?**
A: Updates the UI immediately before the server confirms, and rolls back if the action fails.

### [A2.4 — Route Handlers](A2.4-route-handlers.md)

**Q: What are Route Handlers?**
A: The App Router replacement for `pages/api/` — a `route.ts` file in any `app/` folder (conventionally `app/api/`) exporting functions per HTTP method: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, `OPTIONS`.

**Q: What do they receive and return?**
A: A Web API `Request` and a `Response`. `NextResponse` extends `Response` with helpers like `NextResponse.json()` and `NextResponse.redirect()`.

**Q: When should you use Route Handlers instead of Server Actions?**
A: Server Actions for form mutations. Route Handlers for webhooks, third-party callbacks, public APIs (e.g. for mobile apps), file downloads, streaming, and when you need full HTTP control over status codes and headers.

**Q: Are Route Handlers cached?**
A: A `GET` handler with no dynamic functions is cached at build time. `export const dynamic = "force-dynamic"` or reading `cookies()` makes it dynamic.

### [P2.1 — Blog with ISR](P2.1-blog-with-isr.md)

**Q: How are post pages generated and kept fresh?**
A: Statically from MDX files, with `export const revalidate = 60` on the listing and post pages so they regenerate at most every 60 s.

**Q: How is the RSS feed served?**
A: An RSS 2.0 Route Handler at `/feed.xml`, revalidated hourly (`revalidate = 3600`) with a `Cache-Control: s-maxage=3600, stale-while-revalidate=86400` header.

**Q: How is reading time calculated?**
A: `Math.ceil(wordCount / 200)` — about 200 words per minute.

**Q: What other features does it include?**
A: A date-sorted post index and tag/category filtering.

### [P2.2 — Dashboard with Server Actions](P2.2-dashboard-with-server-actions.md)

**Q: How does the task flow work?**
A: Tasks are fetched in a Server Component; add, toggle and delete all go through Server Actions against a simulated in-memory DB (swap for Prisma in a real project).

**Q: How is the cache kept in sync after a mutation?**
A: Each action calls `revalidatePath("/dashboard")`.

**Q: How does a new task appear instantly?**
A: The `TaskList` Client Component uses `useOptimistic` (with `useTransition`) — there's no `useState` for the list; all state comes from the server plus optimistic updates.

---

## Phase 3 — Styling

### [A3.1 — Tailwind CSS Fundamentals](A3.1-tailwind-css-fundamentals.md)

**Q: What is utility-first CSS and why use it?**
A: Composing small utilities in markup (`className="p-4 rounded-lg"`) instead of naming CSS classes. It removes naming, keeps styles co-located, and makes dead-code elimination trivial — only used utilities ship.

**Q: What are Tailwind's breakpoints?**
A: `sm:` 640px+, `md:` 768px+, `lg:` 1024px+, `xl:` 1280px+, `2xl:` 1536px+. They're mobile-first: `sm:flex` applies from small screens up; unprefixed styles apply everywhere.

**Q: What are the two dark mode strategies?**
A: `media` follows the OS `prefers-color-scheme`; `class` requires a `.dark` class on `<html>`. `class` is almost always better for apps with a manual toggle. Both use the `dark:` prefix.

**Q: Why use `theme.extend` instead of `theme`?**
A: Setting `theme` directly replaces Tailwind's defaults; `extend` adds custom colours, spacing, fonts and keyframes alongside them.

**Q: When should you use `@apply`?**
A: Sparingly, for truly reusable patterns (e.g. a `.btn` used in hundreds of places). Overusing it defeats utility-first.

### [A3.2 — Tailwind Component Patterns](A3.2-tailwind-component-patterns.md)

**Q: What is the class conflict problem?**
A: Utilities like `p-4` and `p-8` both exist in the stylesheet, so applying both gives an unpredictable winner. `tailwind-merge` removes the conflicting class.

**Q: What problem does CVA solve?**
A: Variant management — e.g. `size` (sm/md/lg) × `intent` (primary/secondary/danger) gets unwieldy with template literals. `class-variance-authority` provides a type-safe, composable variant system.

**Q: What is `cn()`?**
A: A two-line helper combining `clsx` (conditional classes) and `tailwind-merge` (conflict resolution) — arguably the most-copied utility in the Next.js ecosystem.

### [A3.3 — CSS Modules & Global Styles](A3.3-css-modules-and-global-styles.md)

**Q: How do CSS Modules prevent naming collisions?**
A: They scope class names automatically (`.Card` → `.Card_abc123`). You import the module as an object and use `styles.Card`.

**Q: What extra features do CSS Modules support?**
A: `:global()` for unscoped styles (e.g. resetting third-party CSS), `composes` to reuse classes from other modules, and CSS custom properties.

**Q: Why use CSS custom properties for theming?**
A: Set `--color-brand` on `:root` and override it under `.dark` to switch themes without JavaScript state at the CSS level.

**Q: What does `next/font` do?**
A: Downloads fonts at build time, generates size-adjusted fallbacks to eliminate layout shift, and self-hosts them — no runtime requests to Google Fonts, which also protects privacy. Works with Google or local fonts.

### [A3.4 — Animation](A3.4-animation.md)

**Q: What makes a good UI animation?**
A: Restraint — fast (150–400 ms), purposeful (communicating state changes or adding delight), and respecting `prefers-reduced-motion`.

**Q: Which animations does Tailwind provide?**
A: `animate-spin`, `animate-ping`, `animate-bounce`, `animate-pulse`, plus custom keyframes from `tailwind.config.ts`.

**Q: What are Framer Motion's core primitives?**
A: `motion.*` elements with `initial`/`animate`/`exit`; `variants` for named states and parent-child coordination; `AnimatePresence` for exit animations; and the `layout` prop for automatic size/position animation.

**Q: When should you use CSS transitions instead of Framer Motion?**
A: For simple hover/focus effects — Tailwind `transition` utilities have zero runtime cost. Use Framer Motion for springs, exit animations or complex orchestration.

### [P3.1 — Landing Page](P3.1-landing-page.md)

**Q: What sections does it include?**
A: An animated hero with CTA buttons, a features grid with hover effects, a 3-tier pricing section, and a footer.

**Q: How is dark mode implemented?**
A: With `next-themes`.

**Q: Which animations are used?**
A: Framer Motion fade-in-up on scroll, staggered children, and smooth page transitions.

### [P3.2 — Component Library](P3.2-component-library.md)

**Q: Which components does it build?**
A: 10+: Button, Badge, Alert, Input, Textarea, Card (Header/Content/Footer), Avatar (fallback initials), Spinner/LoadingDots, Modal/Dialog (accessible, focus-trapped), Toast (auto-dismiss), and Toggle/Switch.

**Q: How are variants managed?**
A: With CVA — base styles applied to every variant, plus variant maps (e.g. Button: default, destructive, outline, ghost, link; sizes sm/md/lg).

**Q: How is the library showcased?**
A: A `/ui` demo page showing every component and variant.

---

## Phase 4 — State Management & Client Data Fetching

### [A4.1 — React Query (TanStack Query)](A4.1-react-query-tanstack-query.md)

**Q: What problem does TanStack Query solve?**
A: Server state — data that lives on the server and must be fetched, cached and kept in sync. Without it you hand-roll loading/error states, race-condition handling, caching and refetch-on-focus.

**Q: What role does the `queryKey` play?**
A: It's the query's identity: when the key changes React Query refetches, and `invalidateQueries` marks matching queries stale and refetches them — how data stays fresh after mutations.

**Q: What does `useMutation` provide?**
A: `mutate`, `isPending`, `isSuccess`, `isError`, and callbacks `onSuccess`, `onError` and `onMutate` (for optimistic updates).

### [A4.2 — Zustand](A4.2-zustand.md)

**Q: How does Zustand differ from Redux?**
A: No boilerplate, reducers, action creators or Provider. Call `create()` with a function receiving `set` and `get`, and it returns a hook.

**Q: Why is Zustand performant by default?**
A: Stores are closures, and `set` only re-renders components subscribed to the part of state that changed.

**Q: Why use selectors?**
A: `useStore()` subscribes to everything; `useStore((s) => s.count)` subscribes only to `count`, avoiding unnecessary re-renders.

**Q: What middleware is available?**
A: `devtools` (Redux DevTools), `persist` (to `localStorage` or another storage adapter), and `immer` (write mutative code that becomes immutable updates).

**Q: What is the slice pattern?**
A: For large apps, splitting state into logical slices in separate files, composed into one store.

### [A4.3 — React Hook Form](A4.3-react-hook-form.md)

**Q: Why is React Hook Form fast?**
A: It uses uncontrolled inputs via `ref` instead of `setState` on every keystroke, re-rendering only on submit or watched fields — a big difference on a 20-field form.

**Q: What does `register` spread onto an input?**
A: `name`, `ref`, `onChange` and `onBlur`, letting RHF track the value through the ref.

**Q: When do you need `Controller`?**
A: For components without a native input — custom selects, date pickers, Radix UI components. It provides `field` props that connect them to RHF.

**Q: How does Zod integrate with RHF?**
A: Define a `z.object()` schema (runtime validation plus inferred types) and pass `resolver: zodResolver(schema)` from `@hookform/resolvers`.

### [A4.4 — URL State](A4.4-url-state.md)

**Q: Why put filter/sort/page state in the URL?**
A: e.g. `/products?category=electronics&maxPrice=500&sort=rating&order=desc&page=3` is shareable, bookmarkable, and handled correctly by back/forward.

**Q: Which Next.js hooks are involved?**
A: `useSearchParams()` reads params in a Client Component; `useRouter()` pushes or replaces navigation. Manual parsing and serialising is verbose and error-prone.

**Q: What is nuqs?**
A: A library of typed URL-state hooks: `useQueryState("page", parseAsInteger.withDefault(1))` gives a `page: number` that reads from and writes to the URL with type safety and defaults.

### [P4.1 — Product Listing with Filters](P4.1-product-listing-with-filters.md)

**Q: How is the product data fetched?**
A: With React Query from a Route Handler.

**Q: Where does filter, sort and page state live?**
A: In URL search params via `nuqs` — category checkboxes, price range, sort select and pagination are all shareable links.

**Q: How is the cart managed?**
A: In a Zustand store (with `persist` middleware) supporting add, remove and quantity updates, shown in a cart drawer with a header badge.

### [P4.2 — Multi-Step Form](P4.2-multi-step-form.md)

**Q: What are the steps?**
A: Personal info → account setup (username, password, confirm) → preferences (role, team size, newsletter) → review, with a progress indicator.

**Q: How is each step validated?**
A: A per-step Zod resolver validates only the current step's fields before moving on.

**Q: How does the form survive a reload?**
A: Persisted data is loaded from `localStorage` on mount and saved on every change, with a resume indicator.

**Q: How is it submitted?**
A: A React Query `useMutation` POSTs to `/api/onboard`.

---

## Phase 5 — Authentication

### [A5.1 — NextAuth.js v5 (Auth.js)](A5.1-nextauth-v5.md)

**Q: What changed in Auth.js v5 from v4?**
A: It's rebuilt for the App Router: config lives in `auth.ts` (not `pages/api/auth/[...nextauth].ts`), `auth()` replaces `getServerSession()`, and middleware integration is cleaner.

**Q: How do OAuth providers work?**
A: The user is redirected to Google/GitHub, logs in, and is redirected back with an access token. You create an OAuth app in each provider's console for a `CLIENT_ID` and `CLIENT_SECRET`.

**Q: What is the Credentials provider?**
A: Custom email/password logic: validate against your store and return a user object if valid.

**Q: Why must Credentials sessions use JWT?**
A: There's no OAuth token to refresh, so database sessions aren't used.

**Q: What do the `jwt` and `session` callbacks do?**
A: `jwt` runs when a token is created or updated; `session` runs when a session is read. Use them to add custom fields such as `role`.

### [A5.2 — Protected Routes](A5.2-protected-routes.md)

**Q: Why is middleware the right place for auth checks?**
A: It runs before every request and can redirect, rewrite or modify responses, so whole route groups are protected without checks in every page. It runs on the fast Edge Runtime, with only a subset of Node.js APIs.

**Q: How does Auth.js integrate with middleware?**
A: Its `middleware` export wraps your function and passes `auth` (the session) and `request`.

**Q: How is RBAC done in middleware?**
A: Check `session.user.role` — e.g. `/admin` only for `role === "admin"`; others are redirected to `/403` or `/dashboard`.

**Q: What should the `matcher` exclude?**
A: Static files (`_next/static`), public assets and API auth routes.

### [A5.3 — Session & Tokens](A5.3-session-and-tokens.md)

**Q: How do JWT sessions work and what's the trade-off?**
A: Session data is in a signed, encrypted cookie — no DB round-trip, works without a database. But you can't invalidate a session without expiry or a blocklist.

**Q: How do database sessions work and what's the trade-off?**
A: The cookie holds only a random token; each request looks up the session in the DB. This allows instant revocation and arbitrary session data, but every authenticated request hits the database.

**Q: Which should you choose?**
A: JWT for most apps; database sessions when you need instant revocation (security incidents, account deletion).

**Q: How do you access the session in each context?**
A: Server Component and Route Handler: `await auth()`. Middleware: `req.auth`. Client Component: `useSession()` from `next-auth/react` (requires `SessionProvider`).

### [A5.4 — Auth Patterns](A5.4-auth-patterns.md)

**Q: What does a production auth flow involve beyond setup?**
A: Signup, optional email verification, login, logout and account management — complicated by OAuth (e.g. signing up with Google, then trying email/password and hitting "account already exists").

**Q: How does Auth.js report auth failures?**
A: It redirects to your custom error page with an error code in the query string.

**Q: What are the most common error codes?**
A: `OAuthAccountNotLinked` (different provider for an existing email), `CredentialsSignin` (wrong email/password), and `Callback` (OAuth callback error).

**Q: What is account linking?**
A: Connecting multiple OAuth providers to one account. With a database adapter Auth.js does it automatically by email; without one you handle it manually.

### [P5.1 — Full-Stack App with Auth](P5.1-full-stack-app-with-auth.md)

**Q: What does it build?**
A: A notes app with Google OAuth and email/password sign-in, sign-up that creates a DB user, a protected `/dashboard`, a `/profile` page to edit display name, and sign-out.

**Q: How are passwords checked?**
A: With `bcryptjs` — `bcrypt.compare` against the stored hash. It uses `@auth/prisma-adapter` with Prisma.

**Q: How are routes protected?**
A: With middleware that redirects unauthenticated users to sign-in.

**Q: Where must the session be accessible?**
A: In Server Components, Client Components and Server Actions. Profile updates call `revalidatePath("/profile")`.

### [P5.2 — Role-Based Access](P5.2-role-based-access.md)

**Q: How does the role get into the session?**
A: It's stored in the DB, loaded on first sign-in in the `jwt` callback, and copied into the session; the built-in session/user types are extended to include it.

**Q: What does the middleware enforce?**
A: Redirects unauthenticated users, blocks non-admins from `/admin` (showing Access Denied), and redirects logged-in users away from auth pages.

**Q: Why also check at the layout level?**
A: As a second line of defence in addition to middleware.

**Q: What does the admin panel include?**
A: A user list with role management (role toggle).

---

## Phase 6 — Performance & Core Web Vitals

### [A6.1 — next/image](A6.1-next-image.md)

**Q: Why replace `<img>` with `next/image`?**
A: `<img>` is the most common cause of poor LCP and CLS.

**Q: How does `next/image` prevent CLS?**
A: It requires `width` and `height` (or `fill`), so the browser reserves space before the image loads.

**Q: What does the `priority` prop do?**
A: Disables lazy loading for above-the-fold images and adds a `<link rel="preload">`, improving LCP.

**Q: What else does it optimise?**
A: Serves WebP/AVIF (50–80% smaller than JPEG/PNG), generates multiple sizes via `srcSet`, and lazy-loads everything without `priority`.

**Q: Why is the `sizes` prop critical?**
A: It tells the browser what share of the viewport the image occupies at each breakpoint so it picks the right `srcSet` entry instead of an oversized file.

**Q: What are remote patterns?**
A: A `next.config.js` allowlist of external domains that may serve images through the Next.js optimiser.

### [A6.2 — Code Splitting & Lazy Loading](A6.2-code-splitting-and-lazy-loading.md)

**Q: Why does bundle size matter?**
A: Every kilobyte must be parsed and executed before the page is interactive, directly affecting TTI.

**Q: What does Next.js split automatically, and what must you split manually?**
A: Each route is its own chunk automatically. Heavy components within a page must be split with `dynamic()`.

**Q: What options does `dynamic()` take?**
A: `ssr: false` to render only on the client (browser-only libraries like charts or maps), and `loading` for a fallback while the chunk downloads.

**Q: How do Suspense boundaries help?**
A: `<Suspense fallback={<Skeleton />}>` shows a skeleton immediately while a slow section streams in — the pattern behind `loading.tsx`.

**Q: What is the target initial JS payload?**
A: Under 150 kB (uncompressed) for above-the-fold content.

### [A6.3 — Core Web Vitals](A6.3-core-web-vitals.md)

**Q: What is LCP, its target and common causes?**
A: Largest Contentful Paint — how fast the main content appears. Target < 2.5 s. Causes: render-blocking resources, large unoptimised images, slow server response, no `priority` on the hero image.

**Q: What is INP, its target and common causes?**
A: Interaction to Next Paint (replaced FID in 2024) — responsiveness. Target < 200 ms. Causes: heavy event handlers, synchronous updates blocking paint, third-party scripts.

**Q: What is CLS, its target and common causes?**
A: Cumulative Layout Shift — content jumping. Target < 0.1. Causes: images without dimensions, content injected above existing content, font FOUT, ads.

**Q: Why do Core Web Vitals matter?**
A: They're Google's UX metrics and directly affect search rankings.

**Q: What is the optimisation cycle?**
A: Measure → Identify → Fix → Measure again. Every fix must be verified.

### [A6.4 — Bundle Analysis](A6.4-bundle-analysis.md)

**Q: Why analyse the bundle?**
A: The JS bundle is the main bottleneck for TBT and TTI; the analyser shows what's in each chunk, sized proportionally — the starting point for optimisation.

**Q: What is tree shaking and when does it work?**
A: Removing unused code at build time. It only works with ES Modules, and depends on how you import: `import { debounce } from "lodash-es"` is tree-shakeable; `import _ from "lodash"` ships all of lodash.

**Q: What are the `next/script` strategies?**
A: `beforeInteractive` (before hydration, critical only), `afterInteractive` (default), `lazyOnload` (after full load — analytics, chat widgets), and `worker` (Web Worker via Partytown, experimental).

### [P6.1 — Performance-Optimized Image Gallery](P6.1-performance-optimized-image-gallery.md)

**Q: What are the performance targets?**
A: LCP < 2.5 s and a Lighthouse performance score ≥ 90, with 30+ Unsplash images.

**Q: How is the blur-up placeholder generated?**
A: Server-side with `plaiceholder` (and `sharp`): `getPlaiceholder(src, { size: 10 })` returns a base64 `blurDataURL`.

**Q: How is image loading prioritised?**
A: The hero uses `priority` for LCP; the first 4 grid images load eagerly and the rest lazily, each with a `sizes` value so the browser downloads the right size.

**Q: How are remote images and formats configured?**
A: `next.config.ts` allows Unsplash in remote patterns and prefers AVIF, then WebP.

**Q: How is the layout built?**
A: A masonry grid using CSS columns, with a lightbox and "load more"/infinite scroll.

### [P6.2 — Optimized E-Commerce Page](P6.2-optimized-e-commerce-page.md)

**Q: Which components are code-split?**
A: The Recharts review chart and the `embla-carousel` related-products carousel — both loaded with `dynamic()` so they're not in the initial bundle.

**Q: What stays statically imported?**
A: Above-the-fold content, which is part of the initial JS.

**Q: How is the bundle analysed?**
A: `@next/bundle-analyzer` via `ANALYZE=true pnpm build`, which opens client and server treemaps. Look for large libraries not needed above the fold (recharts, @mui, lodash, moment).

**Q: How are lazy sections shown while loading?**
A: `<Suspense>` wrappers with skeleton fallbacks (or a custom `loading` fallback), and `ssr: false` for window-dependent libraries. Before/after CWV scores are compared.

---

## Phase 7 — Testing

### [A7.1 — Jest + Testing Library](A7.1-jest-testing-library.md)

**Q: What is RTL's core principle?**
A: Test components the way a user uses them, not implementation details like state or instance methods.

**Q: `getBy*` vs `queryBy*` vs `findBy*`?**
A: `getBy*` is synchronous and throws on 0 or 2+ matches. `queryBy*` returns `null` if not found (for asserting absence). `findBy*` is async and waits up to 1000 ms.

**Q: What is the preferred query priority?**
A: `getByRole` → `getByLabelText` → `getByPlaceholderText` → `getByText` → `getByTestId` (last resort, via `data-testid`).

**Q: Why prefer `userEvent` over `fireEvent`?**
A: It simulates the full browser event sequence (pointerdown → mousedown → focus → … → mouseup → click), making tests realistic.

### [A7.2 — Component Testing](A7.2-component-testing.md)

**Q: Why is MSW better than `jest.spyOn(global, "fetch")`?**
A: It intercepts at the network level, so components make real `fetch()` calls and the full code path — including error handling — is tested.

**Q: How is MSW used in Jest?**
A: Via its Node package. Handlers match URL patterns and return mocked responses; each test can override handlers for different scenarios.

**Q: Which testing patterns are covered?**
A: `waitFor` for assertions after async work, `within` to scope queries to a container, `jest.useFakeTimers()` for debounce/throttle, and `jest-axe` for accessibility.

### [A7.3 — Playwright e2e](A7.3-playwright-e2e.md)

**Q: What does Playwright test that unit tests don't?**
A: Real browsers (Chromium, Firefox, WebKit) against the running app, catching integration issues unit tests miss.

**Q: What is the preferred locator order?**
A: `getByRole` → `getByLabel` → `getByText` → `getByTestId` → `page.locator("css")` as a last resort.

**Q: What does it mean that locators are lazy?**
A: They don't touch the page until an action (`.click()`, `.fill()`, `.check()`), and Playwright auto-retries until the element is visible and ready.

**Q: How is test isolation handled?**
A: Each test gets a fresh browser context; `beforeEach` sets up shared state, and `storageState` saves an authenticated session to disk so you don't log in for every test.

### [A7.4 — Testing Next.js Specifics](A7.4-testing-nextjs-specifics.md)

**Q: Why must `next/navigation` be mocked?**
A: `useRouter`, `usePathname` and `useSearchParams` depend on the Next.js router context and throw in Jest/jsdom otherwise.

**Q: How do you test Server Actions?**
A: Import and call them directly with test data, mock DB calls with `jest.mock()`, verify `revalidatePath` was called, and check success/error return values.

**Q: How do you test Route Handlers?**
A: Construct a `new Request("http://localhost/api/route", { method: "POST", body: ... })`, call the exported `GET`/`POST`, and assert on the `Response`.

### [P7.1 — Tested Component Library](P7.1-tested-component-library.md)

**Q: What is tested?**
A: The P3.2 component library — unit tests for every component, interaction tests (clicking, typing, form submission, keyboard navigation) with `userEvent`, and `jest-axe` audits (`toHaveNoViolations`).

**Q: What supporting setup is used?**
A: A custom render utility with providers.

**Q: What is the coverage target?**
A: ≥ 80%.

### [P7.2 — E2E Tested User Flow](P7.2-e2e-tested-user-flow.md)

**Q: Which journey is covered?**
A: Sign up → email verification → sign in → create/read/update/delete a resource → sign out.

**Q: How are tests kept isolated and deterministic?**
A: API route seeding and cleanup, plus Playwright projects: an auth setup project logs in and saves session state, "authenticated" tests reuse it, and others run unauthenticated.

**Q: How is the code organised?**
A: Page Object Models for the Sign-in, Sign-up and Dashboard pages, with a GitHub Actions `e2e.yml` workflow (visual snapshots optional).

**Q: How is logout verified?**
A: By navigating to the dashboard afterwards and checking it redirects.

---

## Phase 8 — Deployment & CI/CD

### [A8.1 — Vercel Deployment](A8.1-vercel-deployment.md)

**Q: Why is Vercel optimised for Next.js?**
A: It's the company behind Next.js; its Edge Network caches static assets, serves static pages from nearby CDN nodes, and routes server-rendered pages to the nearest compute region.

**Q: What is the deployment flow?**
A: Push to GitHub → webhook → build → deploy. Pushes to `main` go to production; every pull request gets a preview deployment at a unique URL.

**Q: What are the environment variable scopes?**
A: Development (via `vercel env pull`), Preview (non-production branches) and Production.

**Q: `NEXT_PUBLIC_` vs server-only variables?**
A: `NEXT_PUBLIC_` variables are embedded in the client bundle at build time. Unprefixed ones are available only in Server Components, Route Handlers and middleware.

### [A8.2 — Docker Deployment](A8.2-docker-deployment.md)

**Q: What is standalone output?**
A: `output: "standalone"` traces the minimal files needed into `.next/standalone/` — a self-contained Node server without `node_modules`, shrinking images from ~300 MB to ~50 MB.

**Q: Why use a multi-stage Dockerfile?**
A: The build stage installs all dependencies and runs `next build`; the run stage copies only production artifacts.

**Q: What does nginx add as a reverse proxy?**
A: SSL termination, gzip/brotli compression, and serving `.next/static/` and `public/` directly, bypassing Node.js for speed.

### [A8.3 — GitHub Actions](A8.3-github-actions.md)

**Q: What are GitHub Actions workflows?**
A: YAML files in `.github/workflows/` that run on events (push, pull_request, release) and contain jobs of sequential steps.

**Q: What order should CI checks run in?**
A: Lint (ESLint, fastest) → type check (`tsc`) → test (Jest) → build (the final gate).

**Q: Which jobs can run in parallel?**
A: Lint and type-check. Tests should run after type-check.

**Q: How do you speed up installs?**
A: Cache `node_modules` keyed on the lockfile hash (≈60 s → 5 s), invalidating when dependencies change.

### [A8.4 — Environment Management](A8.4-environment-management.md)

**Q: What order does Next.js load env vars in?**
A: `process.env` (highest) → `.env.$(NODE_ENV).local` → `.env.local` (not loaded in test) → `.env.$(NODE_ENV)` → `.env`.

**Q: What are the rules for `NEXT_PUBLIC_` variables?**
A: They're inlined into the bundle at build time, usable in the browser and server code, and cannot change after build.

**Q: What happens if a Client Component uses a server-only variable?**
A: It causes a build-time error; server-only variables never reach the browser.

**Q: What is the `server-only` package for?**
A: Import it in files that must never run in the browser — if a Client Component imports one, the build fails with a clear error.

### [P8.1 — Deployed Next.js App](P8.1-deployed-nextjs-app.md)

**Q: What does it set up?**
A: A Vercel production deployment with git integration, a custom domain (DNS records), per-environment variables, PR preview deployments, `vercel.json` redirects/headers, and Vercel Analytics plus Speed Insights.

**Q: How are env files handled?**
A: `.env.example` is committed with no real secrets; `.env.local` is gitignored. `NEXTAUTH_URL` must be set correctly on Vercel.

**Q: What does the Vercel CLI cover?**
A: Linking the project, preview deploys, rolling back to a previous deployment, and viewing logs.

### [P8.2 — Docker + CI/CD](P8.2-docker-ci-cd.md)

**Q: What are the Dockerfile stages?**
A: 1) install dependencies, 2) build the app (with non-secret build-time env vars), 3) a production runner that copies the standalone output and runs as a non-root user for security.

**Q: How is container health checked?**
A: A `/api/health` endpoint and a Docker `HEALTHCHECK` (30 s interval, 10 s timeout, 40 s start period, 3 retries).

**Q: What does the pipeline do?**
A: Lint → typecheck → test → build Docker image → push to GitHub Container Registry → deploy, split across `ci.yml` and `deploy.yml`.

**Q: How is local development run?**
A: Docker Compose with the app and a database.

---

## Phase 9 — Advanced Features

### [A9.1 — Internationalization](A9.1-internationalization.md)

**Q: What are the two concerns of i18n?**
A: Routing (the right URL per locale) and translation (content in the right language).

**Q: What does next-intl support?**
A: Locale routing via middleware (`/en/about`, `/es/about`), type-safe message dictionaries, pluralisation and date/number formatting, Server and Client Components, and translated metadata via `generateMetadata`.

**Q: What is the routing strategy?**
A: Nest all routes under a `[locale]` segment (`app/[locale]/page.tsx`); middleware detects the locale (URL, cookie or `Accept-Language`) and redirects/rewrites.

**Q: How are RTL languages handled?**
A: Set `dir="rtl"` on `<html>` for Arabic, Hebrew or Persian based on the locale.

### [A9.2 — Streaming & Suspense](A9.2-streaming-and-suspense.md)

**Q: What problem does streaming SSR solve?**
A: Traditional SSR waits for the whole page, so one 3 s fetch means a 3 s blank screen. Streaming sends the shell (layout, nav, skeleton) immediately and streams slow sections in as they finish.

**Q: How does Suspense enable streaming?**
A: `<Suspense fallback={...}>` is a streaming boundary — the fallback is in the first HTML chunk, and React streams and swaps in the real content when ready.

**Q: What is the waterfall problem and its fix?**
A: Sequential fetches take A + B + C. With `Promise.all` or independent Suspense boundaries they run in parallel and take max(A, B, C).

**Q: What is Partial Prerendering (PPR)?**
A: An experimental Next.js 14 feature: a static shell pre-rendered at build time (served from CDN) with dynamic holes streamed per request.

### [A9.3 — Real-time with WebSockets/SSE](A9.3-real-time-with-websockets-sse.md)

**Q: What are Server-Sent Events?**
A: One-way server → client push over HTTP from a Route Handler, read in the browser with `EventSource`. Simple and firewall-friendly — good for feeds, notifications and dashboards.

**Q: Why does Socket.io need a separate server?**
A: Next.js Route Handlers don't support WebSocket upgrades, so Socket.io runs as a separate Node server (or Express adapter) the client connects to.

**Q: When should you use SSE vs WebSockets?**
A: SSE: server → client, HTTP, native in Next.js — feeds, dashboards, notifications. WebSockets: bidirectional, `ws://`, separate server — chat, games, collaboration.

### [A9.4 — PWA & Offline](A9.4-pwa-and-offline.md)

**Q: What makes an app a PWA?**
A: It's installable, works offline (a service worker serves cached responses), and has an app manifest defining name, icons, display mode and theme colour.

**Q: How is PWA support added in Next.js 14?**
A: With `@ducanh2912/next-pwa` (the maintained fork of `next-pwa`), which generates a Workbox service worker at build time, injects registration, and precaches static pages.

**Q: What are the service worker cache strategies?**
A: Cache First (images, fonts, static assets); Network First (APIs where freshness matters); Stale-While-Revalidate (pages, dynamic content); Network Only (analytics, payments).

### [P9.1 — i18n SaaS App](P9.1-i18n-saas-app.md)

**Q: What does it build?**
A: A SaaS landing page and authenticated dashboard in English and Spanish at `/en/...` and `/es/...`.

**Q: How does the language switcher work?**
A: It swaps the locale segment in the path (`/en/` ↔ `/es/`) and persists the preference in a cookie.

**Q: What is localised besides text?**
A: Page titles and descriptions via `generateMetadata`, dates ("March 15, 2024" vs "15 de marzo de 2024"), numbers and currency, and pluralised item counts.

### [P9.2 — Real-Time Dashboard](P9.2-real-time-dashboard.md)

**Q: How is live data delivered?**
A: An SSE Route Handler returns a `text/event-stream`, sending a connection event, 3–5 metric events per second (simulating several servers), and a summary every 10 ticks.

**Q: How does the server clean up when a client disconnects?**
A: It listens for `req.signal`'s `abort` event and closes the stream controller.

**Q: How does the client consume and display the stream?**
A: A `useMetricsStream` hook wraps `EventSource` (closed on cleanup) and buffers events while paused. Data is aggregated into per-second buckets for the last 60 s (line chart), latest value per server (bar chart), and the 20 newest events (table).

**Q: What other features does it include?**
A: A connection status indicator, pause/resume, and CSV download.

### [P9.3 — Full-Stack SaaS Template](P9.3-full-stack-saas-template.md)

**Q: What does it combine?**
A: Auth.js v5 (Google + Credentials), Stripe subscriptions (Free, Pro, Enterprise), a dashboard shell with sidebar, settings (profile, billing, preferences), `next-themes` dark mode, and next-intl English + Spanish.

**Q: How are Stripe webhooks secured?**
A: The handler reads the `stripe-signature` header and verifies the event with `stripe.webhooks.constructEvent` before handling subscription created/updated/deleted events.

**Q: What does the middleware guard?**
A: Protected app paths for authentication, and admin routes with role checks.

---

## Study Strategy & Debugging

### [HOW_TO_LEARN.md](HOW_TO_LEARN.md)

**Q: What is the curriculum's philosophy?**
A: Doing, not reading — 36 assignments and 22 projects. Don't skip assignments to reach projects; projects assume what the assignments teach.

**Q: How should you work through an assignment?**
A: Read the Background fully, set up the environment, type the code yourself (don't paste), run after each step, read the exact error message when something breaks, and pass every Deliverables checkbox.

**Q: How should you work through a project?**
A: Read the whole file, create the architecture's empty files, fill stubs top-down (config → layout → page → components → utilities), commit per logical section, deploy (even Phase 1), and document one thing you learned.

**Q: What causes "Async/await is not yet supported in Client Components"?**
A: An `async` component marked `"use client"`. Remove `async` and use `useEffect` + `useState`, or move the fetch to a Server Component parent.

**Q: What causes "'window' is not defined"?**
A: Using browser APIs in a Server Component. Add `"use client"` or guard with `typeof window !== "undefined"`.

**Q: What causes a hydration mismatch?**
A: Server HTML differing from the client render — e.g. `Date.now()`, `Math.random()`, or browser-dependent rendering. Use `suppressHydrationWarning` for intentional mismatches like theme.

**Q: A new route file isn't found — what should you do?**
A: Restart `pnpm dev`; the file watcher sometimes misses new files.

**Q: What causes "Cannot read properties of undefined (reading 'params')"?**
A: Destructuring page props incorrectly. In Next.js 14 they're `{ params, searchParams }` — e.g. `function Page({ params }: { params: { id: string } })`.

**Q: How do you check progress after an assignment and a project?**
A: Assignment: can you explain it without notes, reproduce the pattern from memory, and did you research every error you didn't understand? Project: is it deployed, does it have a README with a screenshot, is the repo public, and can you explain every file?
