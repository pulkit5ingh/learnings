# React.js — Questions & Answers

Questions and answers drawn from every file in this folder, in curriculum order. Each section links back to its source file.

- [Phase 1 — React Fundamentals](#phase-1--react-fundamentals)
- [Phase 2 — Intermediate Hooks & Patterns](#phase-2--intermediate-hooks--patterns)
- [Phase 3 — React Router & Navigation](#phase-3--react-router--navigation)
- [Phase 4 — Data Fetching & Server State](#phase-4--data-fetching--server-state)
- [Phase 5 — Forms & Validation](#phase-5--forms--validation)
- [Phase 6 — Styling & UI](#phase-6--styling--ui)
- [Phase 7 — Performance](#phase-7--performance)
- [Phase 8 — Testing](#phase-8--testing)
- [Phase 9 — TypeScript, A11y & Production](#phase-9--typescript-a11y--production)
- [Study Strategy & Common Mistakes](#study-strategy--common-mistakes)

---

## Phase 1 — React Fundamentals

### [A1.1 — Components & JSX](A1.1-components-and-jsx.md)

**Q: What is a React component?**
A: A JavaScript function that returns JSX. Components are the building blocks of every React UI — as small as a single button or as large as an entire page.

**Q: What is JSX and what does it compile to?**
A: A syntax extension that looks like HTML but compiles to `React.createElement()` calls.

**Q: Are class components still used?**
A: In React 18+ all components are written as plain functions. Class components still work but are considered legacy.

**Q: Why must a component name start with a capital letter?**
A: Lowercase names are treated as HTML elements, not components.

**Q: What are the main JSX rules?**
A: Every element must be closed (including `<br />` and `<img />`); adjacent elements must be wrapped in a single parent (a real element or a fragment `<>...</>`); attributes use camelCase (`className`, `onClick`); JavaScript expressions go inside curly braces `{}`.

**Q: Why use a fragment instead of a `<div>`?**
A: Fragments group elements without adding an extra DOM node.

**Q: How do you do conditional rendering in React?**
A: With plain JavaScript — there is no `v-if`/`*ngIf`. Use `&&` to render something only when the left side is truthy, a ternary to choose between two things, or an early return to exit before rendering.

**Q: Why does every list item need a `key`?**
A: Lists are rendered with `.map()`, and a stable, unique `key` is how React tracks which items changed, were added, or were removed.

### [A1.2 — Props & Composition](A1.2-props-and-composition.md)

**Q: What are props?**
A: Properties a parent passes to a child component. They are read-only — a component must never modify its own props. When a prop changes, React re-renders the child.

**Q: Why is one-way data flow useful?**
A: Data only flows downward through props, which makes React UIs predictable.

**Q: How do you type props and give them defaults?**
A: Define a TypeScript interface or type for each component's props (for autocomplete and compile-time safety) and provide defaults with JavaScript default parameters in the function signature.

**Q: What is the `children` prop?**
A: A special prop every component receives containing whatever JSX is nested between its opening and closing tags. Wrappers like `Card`, `Modal` and `Layout` use it to render content they don't know ahead of time. Its type is `React.ReactNode` (elements, strings, numbers, arrays, `null`, `undefined`).

**Q: Why does React prefer composition over inheritance?**
A: Instead of `SpecialButton extends Button`, you build a `Button` with a `variant` prop or a `ButtonGroup` that wraps buttons — combining components rather than extending classes.

**Q: What is the render props pattern?**
A: Passing a function as a prop; the child calls it to render, giving the parent full control over what is rendered in a specific slot.

### [A1.3 — useState & Events](A1.3-usestate-and-events.md)

**Q: What does `useState` return?**
A: A tuple: the current value and a setter function. Calling the setter schedules a re-render with the new value.

**Q: Are state updates synchronous?**
A: No. They are asynchronous — the new value isn't visible until the next render.

**Q: Why must you never mutate state directly?**
A: Mutations like `state.push(item)` or `state.count++` bypass the setter, so React doesn't re-render. Always call the setter with a new value.

**Q: When should you use the functional update form?**
A: When the next value depends on the previous one: `setValue(prev => prev + 1)`. It stays correct even when multiple updates are batched.

**Q: What is a SyntheticEvent?**
A: React's wrapper around native DOM events that normalises behaviour across browsers. Since React 17 events are no longer pooled, so `e.persist()` is rarely needed.

**Q: What are the common TypeScript event types?**
A: `React.ChangeEvent<HTMLInputElement>`, `React.FormEvent<HTMLFormElement>`, `React.MouseEvent<HTMLButtonElement>`, etc.

**Q: What is the difference between controlled and uncontrolled inputs?**
A: A controlled input's `value` is driven by React state and `onChange` updates that state, giving React full control. An uncontrolled input is read from the DOM via a ref (the approach React Hook Form uses).

### [A1.4 — useEffect](A1.4-useeffect.md)

**Q: What is a side effect?**
A: Anything a component does beyond returning JSX — fetching data, timers, subscriptions, updating the document title, writing to localStorage. React keeps rendering pure and runs effects after render.

**Q: When does `useEffect` run?**
A: After the browser paints. The deps array controls re-runs: `[]` runs only on mount, `[a, b]` runs when any listed value changes, and no array runs after every render (rarely wanted).

**Q: What is the cleanup function and when is it called?**
A: A function returned from the effect. React calls it before the component unmounts and before the effect re-runs.

**Q: What goes wrong without cleanup?**
A: Event listeners pile up, intervals run forever, and cancelled fetches try to update state on unmounted components.

**Q: Why can't you pass an `async` function to `useEffect`?**
A: An async function returns a Promise, but React expects either `undefined` or a cleanup function. Define an async function inside the effect and call it immediately.

### [P1.1 — Weather App](P1.1-weather-app.md)

**Q: What does this project practise?**
A: Composing components, managing loading and error states with `useState` + `useEffect`, and rendering lists with keys, using the OpenWeatherMap API.

**Q: What features does it include?**
A: City search, a current weather card (temperature, feels-like, humidity, wind, icon), a 5-day forecast strip, a loading skeleton, an error banner for city-not-found/network errors, and a °C/°F toggle.

**Q: Where does the fetching logic live?**
A: In a custom `useWeather` hook, keeping data logic out of the UI components.

**Q: How is the 5-day forecast built from the API data?**
A: One entry per day is taken — the noon reading.

### [P1.2 — Movie Search](P1.2-movie-search.md)

**Q: What does the app do?**
A: Searches the OMDB API, shows a results grid, a detail view (plot, ratings, director, cast), and favourites that persist in localStorage, with loading, error and empty states.

**Q: How are excessive API calls avoided while typing?**
A: The query is debounced with `useDebounce(query, 400)`, so a request fires only after 400 ms without typing.

**Q: How are favourites persisted?**
A: A `useFavorites` hook reads its initial list from `localStorage` (falling back to `[]`) and writes the updated list back with `localStorage.setItem` whenever it changes.

**Q: How is the logic organised?**
A: Into three custom hooks — `useMovieSearch`, `useMovieDetail`, `useFavorites` — with presentational components on top.

---

## Phase 2 — Intermediate Hooks & Patterns

### [A2.1 — useReducer](A2.1-usereducer.md)

**Q: Why use `useReducer`?**
A: When state grows — multiple related fields, transitions that depend on current state, many handlers updating the same state — a reducer keeps all state logic in one place, easy to test and reason about.

**Q: What is a reducer?**
A: A pure function `(state, action) => newState`. The action is an object with a `type` (by convention a string) plus any payload.

**Q: How does `useReducer` relate to Redux?**
A: It uses the same pattern — `useReducer` is essentially a local, built-in Redux. You dispatch actions from handlers and the reducer decides the next state.

**Q: When should you prefer `useReducer` over `useState`?**
A: When state is an object with fields that update together, the next state depends on the current one in a non-trivial way, many handlers modify the same state, or you want all transitions explicit and traceable. Use `useState` for simple flags, single values, or independent state.

### [A2.2 — useContext](A2.2-usecontext.md)

**Q: What is prop drilling?**
A: Passing state through every intermediate component — even ones that don't use it — to reach a deeply nested component.

**Q: How does Context work?**
A: `createContext(defaultValue)` creates a context. `Context.Provider` wraps a subtree and provides the value; `useContext(Context)` reads it from the nearest Provider above.

**Q: When is the context `defaultValue` used?**
A: Only when there is no Provider above. In practice you usually throw an error if the hook is used outside a Provider.

**Q: What is the performance pitfall of Context?**
A: Every consumer re-renders when the value changes. If the value is an object recreated on every render, all consumers re-render every time the Provider's parent renders.

**Q: How do you fix that?**
A: Wrap the value in `useMemo` and functions in `useCallback`, and split contexts by concern (theme, auth, cart) so a theme change doesn't re-render cart consumers.

### [A2.3 — useRef & useMemo/useCallback](A2.3-useref-and-usememo-usecallback.md)

**Q: What is `useRef`?**
A: A mutable container whose `.current` persists across renders without triggering a re-render when changed.

**Q: What are the two use cases of `useRef`?**
A: (1) DOM access — attach it via the `ref` prop for focus, scrolling, measuring, or third-party libraries. (2) A mutable instance variable — previous values, timer IDs, abort controllers, flags.

**Q: What does `useMemo` do and when should you use it?**
A: `useMemo(fn, deps)` memoises a computed result, re-running only when deps change. Use it for genuinely expensive work (sorting/filtering large arrays, complex transforms) — not simple computations, where memoisation overhead can cost more.

**Q: What does `useCallback` do and why does it matter?**
A: `useCallback(fn, deps)` memoises a function reference. It matters when passing callbacks to `React.memo` children or using a callback as an effect dependency — otherwise a new reference each render causes unnecessary re-renders.

### [A2.4 — Custom Hooks](A2.4-custom-hooks.md)

**Q: What makes a function a custom hook?**
A: Its name starts with `use` and it calls other hooks. The prefix tells React and linters it follows the rules of hooks.

**Q: Why extract logic into custom hooks?**
A: To share stateful behaviour (not UI) between components without rendering anything — e.g. one `useFetch` or `useLocalStorage` used in many components, centralising rather than duplicating logic.

**Q: What are the Rules of Hooks?**
A: 1) Only call hooks at the top level — never in conditions, loops or nested functions. 2) Only call hooks from function components or custom hooks.

**Q: Why do those rules exist?**
A: React identifies hooks by call order; breaking the rules causes hard-to-debug bugs. `eslint-plugin-react-hooks` enforces them.

### [P2.1 — Shopping Cart](P2.1-shopping-cart.md)

**Q: How is cart state managed?**
A: With `useReducer`, shared via Context, and wrapped in a custom `useCart` hook — separating state management from UI.

**Q: How does the cart survive a page reload?**
A: `useReducer(cartReducer, undefined, loadCart)` uses a lazy initialiser that reads from localStorage (falling back to `{ items: [] }` on error), and a `useEffect` writes the state back on every change.

**Q: What features does it include?**
A: A product grid, add to cart, a cart drawer with quantities and subtotal, increase/decrease/remove, a header badge, clear cart, and totals.

### [P2.2 — Kanban Board](P2.2-kanban-board.md)

**Q: What libraries/patterns does it use?**
A: `useReducer` for board state, `@dnd-kit` for drag-and-drop, Context to share the board, and per-column custom hooks.

**Q: How does the reducer handle a drag?**
A: It distinguishes reordering within the same column from moving a card to a different column.

**Q: What makes determining the drop target tricky?**
A: The "over" target can be either a column or a card, so the board resolves the source column and then the destination from whichever was hit.

**Q: How is the board persisted?**
A: Its state is saved to and restored from localStorage.

---

## Phase 3 — React Router & Navigation

### [A3.1 — React Router v6](A3.1-react-router-v6.md)

**Q: What is client-side routing?**
A: In an SPA the browser loads one HTML page and JavaScript handles navigation. React Router intercepts URL changes and renders components without a server request — the URL changes (back button and sharing work) but there's no full reload.

**Q: What is the React Router v6 mental model?**
A: Everything is a `<Route>` that renders its `element` when the URL matches its `path`. Nested routes let a parent render a layout while children render into an `<Outlet>`.

**Q: What is the difference between `<Link>` and `<NavLink>`?**
A: `<Link>` changes the URL without reloading. `<NavLink>` also knows whether it's active, so you can style it differently.

**Q: What changed from v5 to v6?**
A: `<Routes>` replaces `<Switch>`, matching is exact by default (no `exact` prop), nested routes use `<Outlet>` instead of `{children}`, and `element` (JSX) replaces `component` (a reference).

### [A3.2 — Route Params & Search](A3.2-route-params-and-search.md)

**Q: Why treat the URL as state?**
A: Filters, search, pagination and selections in the URL can be bookmarked, shared, and undone with the back button. The same state in `useState` is lost on refresh.

**Q: Route params vs search params?**
A: Route params (`/users/:id`) identify a resource and are required — use them for IDs and slugs. Search params (`?sort=asc&page=2`) are optional key-value pairs for filters, sorting and pagination.

**Q: Which hooks read them?**
A: `useParams()` reads route params. `useSearchParams()` reads and writes search params, returning a `URLSearchParams` object and a setter.

### [A3.3 — Programmatic Navigation](A3.3-programmatic-navigation.md)

**Q: How does `useNavigate` work?**
A: It returns a function: pass a path to go there, or a number (`-1`, `-2`) to move through history. Options: `{ replace: true }` replaces the current entry; `{ state: {...} }` passes data without putting it in the URL.

**Q: When should you use `navigate()` vs `<Link>`?**
A: `<Link>` for navigation the user sees and initiates (nav links, "View details"). `navigate()` for navigation as a side effect — after a form submit, delete, login, or timeout.

**Q: Why use `replace: true` when redirecting unauthenticated users?**
A: With push, the protected page stays in history and pressing back redirects them again. Replace overwrites it.

### [A3.4 — Loaders & Protected Routes](A3.4-loaders-and-protected-routes.md)

**Q: What are route loaders?**
A: Functions (React Router v6.4+) attached to routes that run before the route renders, moving fetching to the routing layer. Read results with `useLoaderData()`. They prevent the empty-content flash of `useEffect` fetching.

**Q: How do you lazy-load routes?**
A: `React.lazy()` dynamically imports a component when needed, and `<Suspense>` shows a fallback while the chunk loads. Route-based splitting is the most effective form.

**Q: What are the two kinds of 404 handling?**
A: A catch-all `<Route path="*">` for unknown URLs, and an in-component/loader check for routes that exist but whose data doesn't (e.g. `/users/99999`).

### [P3.1 — Multi-page SPA](P3.1-multi-page-spa.md)

**Q: What does this project focus on?**
A: Routing architecture — 5+ pages, nested dashboard routes, protected routes, a 404 page, breadcrumbs, active NavLinks, and redirect-after-login — not data fetching.

**Q: How does `ProtectedRoute` send users back after login?**
A: It renders `<Navigate to="/login" state={{ from: location }} replace />`, passing the attempted URL.

**Q: How does the login page use that?**
A: It reads `location.state.from.pathname` (defaulting to `/dashboard`) and navigates there after a successful (mock) login.

**Q: How is auth implemented?**
A: A mock `AuthContext` where any email/password works, exposed through a `useAuth` hook.

### [P3.2 — Search App with URL State](P3.2-search-app-with-url-state.md)

**Q: What is the core requirement?**
A: Every piece of state — query (`?q=`), category (`?category=`), sort (`?sort=`), page (`?page=`) — lives in the URL, so the page is bookmarkable, shareable, and back/forward restores state.

**Q: How is the search input kept responsive without flooding the URL?**
A: The input holds local state and syncs back to the URL after a debounce (`setTimeout`).

**Q: Where is URL state handled?**
A: In a `useSearchState` hook, with filtering in a `filter.ts` utility over 100+ mock items.

**Q: What does "Reset filters" do?**
A: Clears all params and returns to page 1.

---

## Phase 4 — Data Fetching & Server State

### [A4.1 — React Query Basics](A4.1-react-query-basics.md)

**Q: What is the difference between server state and client state?**
A: Client state (selected tab, form inputs, panel open) lives in `useState`/`useReducer`. Server state is owned by the server, can go stale, needs refreshing, and may be shared across components — React Query manages it.

**Q: How do you define a query?**
A: `useQuery({ queryKey, queryFn })`. The `queryKey` array uniquely identifies the data and is the cache key; `queryFn` fetches and returns it.

**Q: What does React Query handle for you?**
A: Caching, loading/error/success states, and background refetching (e.g. on window focus).

**Q: What is `staleTime`?**
A: How long fetched data stays "fresh" and is served from cache without refetching. After that it's stale — still shown, but refetched in the background. Default is `0`; use e.g. `5 * 60 * 1000` for slowly changing data.

### [A4.2 — Mutations & Cache Updates](A4.2-mutations-and-cache-updates.md)

**Q: What is `useMutation` for?**
A: Writing data (POST, PUT, PATCH, DELETE). It returns a `mutate` function, with `onSuccess`, `onError` and `onSettled` callbacks.

**Q: What are the two cache update strategies?**
A: **Invalidation** — `queryClient.invalidateQueries(queryKey)` marks data stale and refetches; simple and safe, best when the server reorders/recalculates. **Direct update** — `setQueryData(key, updater)` updates the cache instantly with no fetch; best when you know the new data exactly.

**Q: What is an optimistic update?**
A: Updating the UI immediately, assuming success, and rolling back on failure. Use `onMutate` (before the request), `onError` (restore previous cache), and `onSettled` (refetch for consistency).

### [A4.3 — Advanced React Query](A4.3-advanced-react-query.md)

**Q: What is `useInfiniteQuery`?**
A: A hook for paginated data that keeps a list of pages. `getNextPageParam` provides the next cursor/page, `fetchNextPage()` loads more, and `hasNextPage` says whether more exist.

**Q: How do you run parallel queries?**
A: Call `useQuery` multiple times — React Query runs them in parallel.

**Q: How do you make a dependent query?**
A: Set `enabled: !!userId` on the second query so it waits until the first provides the value.

**Q: What is prefetching?**
A: `queryClient.prefetchQuery()` loads data into the cache before navigation (e.g. on link hover), so the next page renders instantly.

### [A4.4 — SWR Alternative](A4.4-swr-alternative.md)

**Q: What is SWR?**
A: A data-fetching library from Vercel ("stale-while-revalidate") that caches, refetches in the background, and handles loading/error states. It's lighter and simpler than React Query.

**Q: SWR vs React Query?**
A: React Query is more feature-rich (mutations, infinite queries, devtools, offline support); SWR is lighter. For many projects either works.

**Q: How does SWR's revalidation model work?**
A: It serves cached (stale) data immediately, then refetches in the background. By default it revalidates on focus and reconnect; deduplication intervals and revalidation behaviour are configurable.

**Q: What are the key API differences?**
A: React Query: `useQuery({ queryKey, queryFn })` and `useMutation`. SWR: `useSWR(key, fetcher)` (fetcher receives the key) and `mutate` from `useSWRMutation` or the global `mutate`.

### [P4.1 — GitHub Profile Explorer](P4.1-github-profile-explorer.md)

**Q: What does it practise?**
A: TanStack Query v5 — dependent queries, pagination and cache management — across user profiles, repositories and commits.

**Q: How is retry configured?**
A: A custom `retry` function skips retries for 404 (not found) and 403 (rate limit) errors. `staleTime` is 5 minutes.

**Q: How is the empty flash between pages avoided?**
A: `placeholderData: previousData => previousData` keeps the previous page visible while the next loads.

**Q: Why is the commits query dependent?**
A: It only runs once a repository is selected.

### [P4.2 — Real-time Dashboard](P4.2-real-time-dashboard.md)

**Q: How is auto-refresh implemented?**
A: `refetchInterval` polling — metrics every 5 s, alerts every 10 s, services every 15 s.

**Q: How does pause/resume work?**
A: The toggle sets `refetchInterval` to `false`.

**Q: How do optimistic updates work for acknowledging alerts or toggling services?**
A: `onMutate` first calls `queryClient.cancelQueries` for that key (so an in-flight poll doesn't overwrite the change), then updates the cache; errors roll back.

**Q: How are render errors handled?**
A: A class-based `DashboardErrorBoundary` using `getDerivedStateFromError` and `componentDidCatch`, with a retry button that resets its state.

---

## Phase 5 — Forms & Validation

### [A5.1 — React Hook Form](A5.1-react-hook-form.md)

**Q: Why React Hook Form instead of controlled inputs?**
A: Controlled inputs re-render on every keystroke. RHF uses uncontrolled inputs read via refs — zero re-renders while typing; only error changes and `watch` trigger re-renders.

**Q: What does the core API provide?**
A: `register(name)` (props to spread on an input), `handleSubmit(fn)`, `formState` (`errors`, `isSubmitting`, `isDirty`, `isValid`…), `watch(name)`, `setValue(name, value)`, and `reset(values)`.

**Q: How do you add validation?**
A: Pass rules as the second argument to `register` (e.g. `required`, `pattern` with messages), or use a schema validator like Zod via `zodResolver`.

### [A5.2 — Zod Validation](A5.2-zod-validation.md)

**Q: What is Zod?**
A: A TypeScript-first schema validation library that validates data at runtime and infers TypeScript types at compile time — one schema, no duplication.

**Q: What is schema-first validation?**
A: Define one schema, get the type with `z.infer<typeof schema>`, and pass the schema to `zodResolver` for React Hook Form. Changing the schema updates both validation and types.

**Q: What are the key Zod APIs?**
A: Primitives (`z.string()`, `z.number()`, `z.boolean()`, `z.date()`); validators (`.min()`, `.max()`, `.email()`, `.url()`, `.regex()`); `.optional()`/`.nullable()`; `.transform()`; `.refine()`; `.superRefine()`; `z.infer`.

**Q: `.refine()` vs `.superRefine()`?**
A: `.refine()` is custom validation with access to the value; `.superRefine()` has access to the whole object, for cross-field validation.

### [A5.3 — Complex Forms](A5.3-complex-forms.md)

**Q: What is `useFieldArray` for?**
A: Dynamic lists of fields (phone numbers, work experience). It provides `fields`, `append`, `prepend`, `remove`, `move`, `swap`.

**Q: What key should dynamic fields use?**
A: Each field's stable `id` — not the array index.

**Q: How do multi-step forms work?**
A: Store `currentStep` in state and conditionally render steps; each step validates only its own fields. Share one `useForm` across steps, or keep each step's data in `useState` and merge at the end.

**Q: How are nested objects handled?**
A: With dot notation (`register('address.city')`) and nested `z.object()` schemas.

### [A5.4 — File Uploads & Rich Text](A5.4-file-uploads-and-rich-text.md)

**Q: Why can't file inputs be controlled normally?**
A: Their value is write-protected by the browser for security. Read the file from `e.target.files` in `onChange`, store it in state, and use `setValue` to update the form.

**Q: How do you preview an image before upload?**
A: `URL.createObjectURL(file)` creates a temporary in-memory URL. Call `URL.revokeObjectURL(url)` when done to free memory.

**Q: What is TipTap?**
A: A headless rich text editor built on ProseMirror — it provides editing logic but no UI, so you build the toolbar and control styling.

**Q: How does TipTap integrate with React Hook Form?**
A: Call `editor.getHTML()` to get the content and `setValue` to update the form field.

### [P5.1 — Multi-step Registration Form](P5.1-multi-step-registration-form.md)

**Q: What are the steps?**
A: Account (email, password, confirm), Personal (name, DOB, phone), Address, and Review (read-only summary + submit), with a progress indicator.

**Q: How is each step validated?**
A: Each step has its own `useForm` with `zodResolver(stepSchema)`, so it can't advance until its fields pass.

**Q: How is "passwords must match" validated?**
A: With `.refine(data => data.password === data.confirmPassword, …)` on the account schema.

**Q: How is data preserved when going back?**
A: A `useRegistration` hook merges each step's data into shared `formData`; steps receive it as `defaultValues`.

### [P5.2 — CMS-style Post Editor](P5.2-cms-style-post-editor.md)

**Q: What are the main features?**
A: Title with character count, TipTap body with toolbar, auto-generated editable slug, keyboard tag input, cover image drag-and-drop with preview, autosave status, Draft/Publish toggle, and manual save.

**Q: How does autosave work?**
A: A `useAutosave` hook debounces with a `setTimeout` (default 2000 ms) stored in a ref, saving 2 s after the user stops typing. `saveFn` is intentionally left out of the deps to avoid re-triggering.

**Q: How does the tag input handle the keyboard?**
A: Enter or comma adds a tag; Backspace removes the last.

**Q: How is the slug generated?**
A: Automatically from the title via a `slugify` utility, while remaining editable.

---

## Phase 6 — Styling & UI

### [A6.1 — Tailwind CSS with React](A6.1-tailwind-css-with-react.md)

**Q: What is utility-first CSS?**
A: Small single-purpose classes written in JSX (`className="p-4 rounded-lg"`) instead of separate CSS. It co-locates styles, avoids specificity battles and makes unused styles obvious; the trade-off is long class strings.

**Q: How do responsive and dark mode work?**
A: Prefixes `sm:`, `md:`, `lg:`, `xl:` apply at breakpoints; `dark:` applies in dark mode. Tailwind can be configured to use a `dark` class on `<html>` for manual control.

**Q: What are `clsx` and `cn`?**
A: `clsx` builds conditional class lists. `cn` (used by shadcn/ui) combines `clsx` with `tailwind-merge` to de-duplicate conflicting classes — `cn('p-4', isPadded && 'p-8')` outputs `p-8`.

### [A6.2 — Component Variants](A6.2-component-variants.md)

**Q: What problem does CVA solve?**
A: Variant explosion — e.g. 4 variants × 3 sizes × 2 states = 24 combinations of ternaries. CVA (class-variance-authority) declares base styles, variants and compound variants and generates the className.

**Q: What is the CVA API?**
A: `cva(base, { variants, compoundVariants, defaultVariants })` returns a function you call with `{ variant, size, … }` to get the class string.

**Q: What are compound variants?**
A: Styles applied only when a specific combination of variants is active.

**Q: What is `VariantProps`?**
A: A TypeScript utility, `VariantProps<typeof fn>`, that extracts prop types from a CVA function so you don't repeat them.

### [A6.3 — shadcn/ui](A6.3-shadcn-ui.md)

**Q: What is shadcn/ui?**
A: Not an installed dependency — a collection of accessible components you copy into your project and own. Most use Radix UI primitives for accessibility and Tailwind for styling.

**Q: What are Radix UI primitives?**
A: Headless, unstyled components that correctly implement hard accessibility patterns — focus-trapping dialogs, dropdowns closing on Escape, tooltips announced to screen readers.

**Q: Why is this better than Material UI or Ant Design?**
A: You own the source: change any class, add props, integrate your design system, and avoid being tied to their update cycle.

### [A6.4 — Framer Motion](A6.4-framer-motion.md)

**Q: How are animations declared?**
A: Declaratively: describe `initial`, `animate` and `exit` states and Framer Motion animates between them — simpler than manual CSS transitions or `requestAnimationFrame`.

**Q: Why is `AnimatePresence` needed?**
A: React removes elements immediately on unmount. `AnimatePresence` intercepts unmounting to run the `exit` animation first — essential for modals, list removals and page transitions.

**Q: What are layout animations?**
A: The `layout` prop animates position/size changes from reflow (e.g. list reordering). `layoutId` enables shared element transitions, like a card expanding into a modal.

### [P6.1 — UI Component Library](P6.1-ui-component-library.md)

**Q: Which components does it include?**
A: Button, Input, Select, Modal, Toast, Table and Badge, plus a demo page.

**Q: How are variants and accessibility handled?**
A: CVA manages variants; Modal uses Radix Dialog (focus trap) and Toast uses Radix Toast (4 variants with a queue).

**Q: What is special about the Table?**
A: It's a typed generic table with sortable columns and striped rows.

**Q: What does the `cn` helper do here?**
A: Lives in `lib/utils.ts` and merges class names for all components.

### [P6.2 — Animated Portfolio](P6.2-animated-portfolio.md)

**Q: What Framer Motion APIs does it practise?**
A: `motion.div`, `AnimatePresence`, `variants`, `useInView` and `useScroll`.

**Q: How do the sticky header and skill bars animate?**
A: The header increases backdrop blur and adds a shadow as the page scrolls; skill bars animate when scrolled into view.

**Q: How does the dark mode hook pick its initial value?**
A: From the saved `theme` in localStorage, or, if none is saved, the OS `prefers-color-scheme: dark` setting; changes are saved back.

---

## Phase 7 — Performance

### [A7.1 — React.memo & Re-renders](A7.1-react-memo-and-re-renders.md)

**Q: When does a component re-render?**
A: When (1) its own state changes, (2) a context it consumes changes, or (3) its parent re-renders — by default all children re-render even if props didn't change.

**Q: When is that a problem?**
A: Only when the child is expensive to render and the parent re-renders frequently. Otherwise it's by design and fine.

**Q: What does `React.memo` do?**
A: Memoises a component's output and skips re-rendering if a shallow comparison of props shows no change.

**Q: Why does an inline object prop defeat `React.memo`?**
A: Shallow comparison compares objects/arrays by reference; `prop={{ key: 'val' }}` creates a new object each render.

**Q: How do you use the Profiler?**
A: In React DevTools: Record, interact, Stop, then read the flame graph. Enable "Record why each component rendered" to see whether props, state or a hook caused it.

### [A7.2 — useMemo & useCallback in Practice](A7.2-usememo-and-usecallback-in-practice.md)

**Q: What does memoisation cost?**
A: Storing previous results, comparing deps every render, and cognitive overhead. Memoise only for a measured problem or a referential equality requirement.

**Q: When is `useMemo` worth it?**
A: For expensive operations (sorting/filtering thousands of items, complex math, deep transforms — not under ~1 ms), and for stable references passed to `React.memo` children or effect deps.

**Q: When is `useCallback` worth it?**
A: For callbacks passed to `React.memo` children, callbacks used as `useEffect` deps, and callbacks passed to event listeners in custom hooks.

### [A7.3 — Code Splitting](A7.3-code-splitting.md)

**Q: Why does code splitting matter?**
A: A single 500 kB+ bundle makes users download every page up front. Splitting ships only what the first render needs and loads the rest on demand.

**Q: What is the highest-impact form of splitting?**
A: Route-based — each page is its own chunk, downloaded when visited and cached afterwards.

**Q: How do you analyse a Vite bundle?**
A: `rollup-plugin-visualizer` produces an interactive treemap. Large single-page dependencies (PDF, charts, rich text editors) are good candidates for dynamic imports.

### [A7.4 — Virtualization](A7.4-virtualization.md)

**Q: What problem does virtualisation solve?**
A: 10,000 items means 10,000 DOM nodes costing memory and layout time. Virtualisation ("windowing") renders only visible items plus a small buffer, swapping them as you scroll.

**Q: TanStack Virtual vs react-window?**
A: `@tanstack/react-virtual` is headless — it gives measurements/offsets and you control the DOM (more flexible, actively maintained). `react-window` is older with pre-built list components.

**Q: How do you measure the improvement?**
A: Record scrolling in the Chrome DevTools Performance tab (compare FPS) and compare DOM node counts with `document.querySelectorAll('li').length`.

### [P7.1 — Performance Audit](P7.1-performance-audit.md)

**Q: What re-render problems does the profiler reveal in the slow app?**
A: All 100 `ProductItem`s re-render on each keystroke; `CartSidebar` re-renders on search (shared context); `StatsBar` re-renders on cart changes; `ExpensiveChart` re-renders every keystroke.

**Q: What causes them?**
A: One context holding everything, inline functions creating new references each render, and filtering computed on every render (including cart changes).

**Q: What fixes are applied?**
A: Split context (separate `CartContext` and `SearchContext`), `useMemo` for the filtered list (only recomputes on search/sort), `useCallback` for stable handlers, `React.memo` on `ProductItem`, memoised derived cart values, and `memo` + a memoised data prop for `ExpensiveChart`.

**Q: How are results documented?**
A: Before/after metrics in a `PERFORMANCE_NOTES.md`.

### [P7.2 — Virtual List](P7.2-virtual-list.md)

**Q: What does it build?**
A: A 100,000-row virtualised table with TanStack Virtual — sortable headers, debounced per-column filters, global search, a row count, sticky headers, and scroll-to-top.

**Q: Why generate rows at module level?**
A: So the 100k rows are created once, not on every component render.

**Q: How is performance measured?**
A: Bundle analysis with `rollup-plugin-visualizer` (`dist/stats.html`) and LCP measured with a `PerformanceObserver`.

---

## Phase 8 — Testing

### [A8.1 — Jest + React Testing Library](A8.1-jest-react-testing-library.md)

**Q: What is RTL's testing philosophy?**
A: Test the way users interact — by visible text, roles, labels — not component internals. Tests survive refactoring as long as behaviour stays the same.

**Q: What is the query priority?**
A: `getByRole` → `getByLabelText` → `getByPlaceholderText` → `getByText` → `getByDisplayValue` → `getByAltText` → `getByTitle` → `getByTestId` (last resort).

**Q: `getBy` vs `queryBy` vs `findBy`?**
A: `getBy` throws if not found, `queryBy` returns `null`, `findBy` is async and waits for the element.

**Q: Why prefer `userEvent` over `fireEvent`?**
A: `userEvent` simulates real interactions — `userEvent.type` fires keydown, keypress, keyup, input and change, like a real browser.

### [A8.2 — Testing Hooks & Context](A8.2-testing-hooks-and-context.md)

**Q: How do you test a custom hook?**
A: With `renderHook`, which wraps the hook in a minimal component so you can call it, assert on its return value and trigger updates.

**Q: What is `act()` for?**
A: Wrapping state-updating calls so React processes all updates before assertions.

**Q: How do you test context consumers?**
A: Either render with a real Provider and control its props (more realistic) or mock the context value directly (simpler when only part of the context is needed).

**Q: What is the wrapper pattern?**
A: `renderHook`'s `wrapper` option supplies providers — context, router, React Query client — to hooks that need them.

### [A8.3 — Mock Service Worker](A8.3-mock-service-worker.md)

**Q: Why use MSW instead of mocking `fetch`?**
A: Mocking `global.fetch` mocks the transport, not the API contract, and is brittle. MSW intercepts at the network level, so components, React Query hooks and Axios run exactly as in production.

**Q: What are request handlers?**
A: Functions matching a URL and returning a mocked response, e.g. `http.get('/api/posts', () => HttpResponse.json(data))`. Organise them by domain.

**Q: `setupServer` vs `setupWorker`?**
A: `setupServer` (`msw/node`) for Node/tests; `setupWorker` (`msw/browser`) for mocking APIs in the browser during development.

### [A8.4 — Playwright E2E Testing](A8.4-playwright-e2e-testing.md)

**Q: How does Playwright differ from RTL?**
A: RTL tests components in isolation; Playwright drives a real (or headless) Chromium, Firefox or WebKit browser against the running app.

**Q: What are its key APIs?**
A: `page.goto(url)`, `page.locator()` (prefer `getByRole`/`getByText`/`getByLabel`), `page.fill()`/`page.click()`, `expect(locator).toBeVisible()`, `storageState`, and fixtures.

**Q: How do you avoid logging in for every test?**
A: A global setup logs in once and saves auth state (cookies + localStorage) with `storageState`; authenticated test projects reuse it.

**Q: What is the Page Object Model for?**
A: Encapsulating page interactions in reusable classes to keep tests maintainable.

**Q: How is it run in CI?**
A: The config starts the dev server before tests, and the suite runs in GitHub Actions.

### [P8.1 — Tested React App](P8.1-tested-react-app.md)

**Q: What is the coverage target and app under test?**
A: 80%+ coverage on the P1.2 Movie Search app (or a similar app).

**Q: Which kinds of tests does it include?**
A: Unit tests for utilities and hooks, RTL component tests, MSW handlers for the OMDB API, integration tests for search, and Playwright E2E.

**Q: Which E2E flow is tested?**
A: Search → view detail → add to favourites → verify persistence.

**Q: How are MSW mocks split?**
A: `mocks/server.ts` for the Node/test environment and `mocks/browser.ts` for development.

### [P8.2 — Accessible & Tested Component Library](P8.2-accessible-and-tested-component-library.md)

**Q: Which components and ARIA patterns are built?**
A: Tabs (`aria-selected`, `aria-controls`, Arrow keys), Combobox (`role="combobox"`, `aria-autocomplete`, `aria-activedescendant`, Arrow/Enter/Escape), Disclosure (`aria-expanded`), Switch (`role="switch"`, `aria-checked`, Space), and AlertDialog (`role="alertdialog"`, focus trap).

**Q: What must every component pass?**
A: Automated axe audits, full keyboard navigation, and interaction tests verifying ARIA state changes and focus behaviour.

**Q: Why does the Combobox close on blur after a `setTimeout`?**
A: The 150 ms delay lets a click on an option register before the list closes.

---

## Phase 9 — TypeScript, A11y & Production

### [A9.1 — TypeScript with React](A9.1-typescript-with-react.md)

**Q: Why use TypeScript with React?**
A: To catch wrong prop types, missing null checks and bad handler signatures before the browser does. `strict: true` gives the strongest guarantees.

**Q: `React.FC<Props>` or a plain function?**
A: Prefer a plain function declaration. `React.FC` added an implicit `children` prop (pre-React 18) and hides return type issues.

**Q: How do you type `useState` and `useRef`?**
A: Use an explicit generic when the initial value is `null`/`undefined` (`useState<User | null>(null)`). For DOM refs, the generic is the element type and the initial value is `null`; TypeScript knows `.current` may be `null` before mount.

**Q: Which utility types are useful?**
A: `Pick` (only what a card needs), `Omit` (drop internal fields), `Partial` (update payloads), `Required`, `ReturnType`, and `Record` (typed dictionaries).

**Q: Why use discriminated unions for variants?**
A: Instead of conflicting boolean props, a union lets TypeScript narrow types per branch and enforce the right props per variant (e.g. a missing `onDismiss` is an error).

**Q: How do you type API responses?**
A: Derive types from Zod schemas — a single source of truth.

### [A9.2 — Accessibility (a11y)](A9.2-accessibility-a11y.md)

**Q: What standard applies?**
A: WCAG 2.1 AA — accessibility is a baseline, not a feature.

**Q: What are common a11y problems in React?**
A: `<div>`/`<span>` instead of semantic elements, missing `aria-label` on icon buttons, broken or missing focus traps in modals, and dynamic content invisible to screen readers.

**Q: What do the main ARIA attributes do?**
A: `aria-label` names elements without visible text; `aria-describedby` links fields to helper text; `aria-expanded` marks toggleable regions; `role` overrides semantics; `aria-live` announces dynamic updates.

**Q: What are the `tabIndex` guidelines?**
A: `0` adds an interactive element to the tab order; `-1` makes it focusable programmatically but not via Tab; `1+` should be avoided because it breaks the natural order.

**Q: Do buttons need `onKeyDown`?**
A: No — buttons already respond to Enter and Space. Only non-button interactive elements need it.

**Q: How should modal focus be managed?**
A: Move focus into the modal when it opens, trap it inside, and return it to the trigger on close.

**Q: Which tools are used?**
A: `axe-core`/`jest-axe` (automated), `eslint-plugin-jsx-a11y` (lint), NVDA/VoiceOver (manual screen reader testing), and the DevTools accessibility tree.

### [A9.3 — Internationalization (i18n)](A9.3-internationalization-i18n.md)

**Q: What does react-i18next provide?**
A: `useTranslation(namespace)`, the `Trans` component, built-in pluralisation rules per language, and `Intl` integration for dates and numbers.

**Q: Where do translation files live?**
A: `public/locales/{lang}/{namespace}.json`, lazy-loaded with `i18next-http-backend`.

**Q: How does pluralisation work?**
A: i18next follows ICU rules; for English the plural key suffix is `_other` (count=1 → "1 item", count=5 → "5 items").

**Q: When do you use `Trans`?**
A: When a translated string contains React elements like links or bold text.

**Q: What should the language switcher also update?**
A: `<html lang>`, so screen readers use the right language.

**Q: Can translation keys be type-checked?**
A: Yes — with typed resources, `t('nav.home')` typos become TypeScript errors.

### [A9.4 — Build & Deployment](A9.4-build-and-deployment.md)

**Q: What does a React SPA build into?**
A: Static files — HTML, JS bundles, CSS and assets. Vite tree-shakes, code-splits and fingerprints them.

**Q: How do environment variables work in Vite?**
A: Only `VITE_`-prefixed variables are exposed via `import.meta.env`, at build time — so never put secrets there. Centralise and validate them (e.g. `src/config/env.ts`).

**Q: How is the Vite build tuned?**
A: Manual vendor chunks for code splitting, and a warning when any chunk exceeds 500 kB.

**Q: How is the app containerised?**
A: Static output served by an `nginx:alpine` container, with `try_files $uri $uri/ /index.html` so client-side routes work, and long caching for hashed assets.

**Q: What does the CI/CD pipeline do?**
A: Quality gates (lint, test) → build → E2E tests → Docker push (main only) → optional deploy to Vercel, using repository secrets like `VITE_API_URL`, `VERCEL_TOKEN` and `CODECOV_TOKEN`.

### [P9.1 — Fully Typed React App](P9.1-fully-typed-react-app.md)

**Q: What are the typing requirements?**
A: TypeScript strict mode plus `noUncheckedIndexedAccess`, zero `any`, Zod-validated API responses, type-safe route params, and explicit return types for every custom hook.

**Q: What does the app do?**
A: A task manager with CRUD, categories and priorities, filtering by status and category, and a modal form.

**Q: How does the API client stay type-safe?**
A: It parses responses with Zod — throwing a `ZodError` if the shape is wrong — and types are derived from the schemas.

**Q: How does `PriorityBadge` use the type system?**
A: A config keyed by the `Priority` type guarantees `config[priority]` always exists.

### [P9.2 — Accessible Dashboard](P9.2-accessible-dashboard.md)

**Q: What does it need to meet?**
A: WCAG 2.1 AA — keyboard access everywhere, screen reader announcements, colour contrast, and zero axe violations.

**Q: What contrast ratio is required for text?**
A: At least 4.5:1 for text under 18pt; all status badge colour combinations are verified against it.

**Q: How does the focus trap work?**
A: It saves the currently focused element, moves focus into the trap, and restores focus when closed.

**Q: How do screen reader announcements work?**
A: A `useAnnounce` hook writes to hidden `aria-live` regions (`polite` or `assertive`) rendered at the app root. It clears the text and then sets it, so repeated messages are still announced.

**Q: What else is included?**
A: A skip navigation link, a sortable keyboard-selectable data table, charts with text alternatives, announced loading states, and dark mode with maintained contrast.

### [P9.3 — Production SPA](P9.3-production-spa.md)

**Q: What does it combine?**
A: Everything in the curriculum — TypeScript strict mode, Zod validation, 80%+ test coverage (unit, integration, E2E), English + French i18n, WCAG 2.1 AA, Docker + nginx, and GitHub Actions CI/CD — in a Bookmark Manager.

**Q: What is the performance budget?**
A: Enforced with Lighthouse CI: LCP < 2.5 s, TBT < 200 ms, CLS < 0.1.

**Q: How is "no `any`" enforced?**
A: With an ESLint rule.

**Q: What accessibility extras does it require beyond P9.2?**
A: The page title updates on route change, and loading states are announced to screen readers.

---

## Study Strategy & Common Mistakes

### [HOW_TO_LEARN.md](HOW_TO_LEARN.md)

**Q: What is the core learning loop?**
A: Read the assignment → type the code by hand → break it on purpose → fix it → build the project. Breaking it on purpose is where most of the learning happens.

**Q: Should you look at a project's implementation while building it?**
A: No. Build first, then compare. The implementation is there to unblock you if you've been stuck for more than 20 minutes.

**Q: What are the phases of a project build?**
A: Scaffold → static shell with fake data → wire up state → connect real data → polish (loading/error/empty states, responsive) → stretch feature.

**Q: Why is treating React like jQuery a mistake?**
A: React is declarative — describe the UI for the current state instead of manipulating the DOM. If you reach for `document.getElementById`, ask how to express it with state.

**Q: Why is `items.push(4); setItems(items)` wrong?**
A: It mutates state, so React won't re-render. Use `setItems(prev => [...prev, 4])`.

**Q: Why not use the index as a list key?**
A: If the list can reorder, index keys break React's tracking. Use a stable unique id.

**Q: What about missing `useEffect` dependencies?**
A: Run `eslint-plugin-react-hooks` and never suppress its warnings without understanding them.

**Q: Why shouldn't server data go in `useState` + `useEffect`?**
A: From Phase 4 on, all API data goes through React Query, which handles caching, staleness and refetching.
