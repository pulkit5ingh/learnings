# TanStack Query — Questions & Answers

Interview questions and answers on TanStack Query (React Query) **v5**, from fundamentals to SSR, testing and production scenarios. Each answer has an explanation, a code example and an interview-ready answer you can say out loud.

- [Phase 1 — Fundamentals](#phase-1--fundamentals)
- [Phase 2 — Queries](#phase-2--queries)
- [Phase 3 — Caching & Refetching](#phase-3--caching--refetching)
- [Phase 4 — Mutations](#phase-4--mutations)
- [Phase 5 — Pagination & Infinite Queries](#phase-5--pagination--infinite-queries)
- [Phase 6 — Advanced Patterns](#phase-6--advanced-patterns)
- [Phase 7 — SSR & Next.js](#phase-7--ssr--nextjs)
- [Phase 8 — Testing](#phase-8--testing)
- [Phase 9 — Architecture & Scenarios](#phase-9--architecture--scenarios)
- [Interview Quick Revision](#interview-quick-revision)

---

## Phase 1 — Fundamentals

### What TanStack Query Is

**Q: What is TanStack Query, and what problem does it solve?**
A: TanStack Query (formerly React Query) is a **server-state management** library. It fetches, **caches**, **deduplicates**, **synchronises** and **updates** data that lives on a server, and gives components declarative loading, error and success state. It is **not** a data-fetching client: you bring `fetch`, Axios or GraphQL. It manages **what happens around** the request. It replaces the fragile `useEffect` + `useState` fetching pattern, which has no cache, no deduplication, race conditions and boilerplate in every component.

```tsx
// Without TanStack Query: loading, error, cancellation, no cache, refetch by hand
const [todos, setTodos] = useState<Todo[]>([]);
const [loading, setLoading] = useState(true);
const [error, setError] = useState<Error | null>(null);
useEffect(() => {
  let ignore = false;
  fetchTodos().then((d) => !ignore && setTodos(d)).catch((e) => !ignore && setError(e)).finally(() => !ignore && setLoading(false));
  return () => { ignore = true; };
}, []);

// With TanStack Query: cached, deduplicated, background refetch, retries
const { data: todos, isPending, error } = useQuery({ queryKey: ['todos'], queryFn: fetchTodos });
```

Interview-ready answer: *"TanStack Query is an async server-state manager. I give it a key and a function that returns a promise, and it handles caching, deduplication, background refetching, retries, stale data, pagination and cache updates after mutations. It replaces hand-written useEffect fetching, which is boilerplate-heavy, race-prone and has no cache. It's transport-agnostic, so it works with fetch, Axios or GraphQL."*

**Q: What is the difference between server state and client state?**
A: **Client state** is owned by the browser and is synchronous and always up to date: a modal being open, the selected tab, form inputs, the theme. **Server state** is **owned remotely**: you hold a **cached snapshot** that can become **stale** when someone else changes it. It's asynchronous and shared across components, and needs fetching, caching, refetching, invalidating and deduplicating. Mixing the two (putting API data in Redux or `useState`) means rebuilding a cache by hand.

```tsx
// Client state → useState / Zustand / Context
const [isCartOpen, setIsCartOpen] = useState(false);

// Server state → TanStack Query
const { data: cart } = useQuery({ queryKey: ['cart'], queryFn: getCart });
```

Interview-ready answer: *"Client state is UI state the browser owns, like whether a drawer is open. It's synchronous and always correct. Server state is a cached copy of data the backend owns: async, shared, and possibly out of date. They have different problems, so I use different tools: useState or Zustand for client state, and TanStack Query for server state. That removes most of the global state that used to live in Redux."*

**Q: How do you set up TanStack Query in a React app?**
A: Create **one `QueryClient`** (it holds the cache), provide it with **`QueryClientProvider`** at the root, and optionally add the **DevTools**. Create the client **outside** the component, or in `useState`, so it isn't recreated on every render, which would wipe the cache.

```tsx
// main.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60_000,            // treat data as fresh for 1 minute (the default is 0)
      retry: 2,
      refetchOnWindowFocus: true,
    },
  },
});

createRoot(document.getElementById('root')!).render(
  <QueryClientProvider client={queryClient}>
    <App />
    <ReactQueryDevtools initialIsOpen={false} />
  </QueryClientProvider>
);
```

Interview-ready answer: *"I create a single QueryClient, which owns the cache and the default options, and wrap the app in QueryClientProvider. The client is created once, outside render, and I set app-wide defaults like a sensible staleTime and retry count. I add the React Query DevTools in development to inspect cache entries, their status and refetches."*

**Q: What are the React Query DevTools used for?**
A: A panel showing **every query in the cache**: its key, **status** (fresh, stale, fetching, paused, inactive), data, observers count and last-updated time, plus buttons to **refetch, invalidate, reset or remove** queries and to simulate loading or error states. It's the fastest way to debug "why did this refetch?" or "why is this data stale?". It's tree-shaken out of production builds by default.

```tsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

<QueryClientProvider client={queryClient}>
  <App />
  <ReactQueryDevtools initialIsOpen={false} buttonPosition="bottom-right" />
</QueryClientProvider>
```

Interview-ready answer: *"The DevTools show the live cache: every query key, whether it's fresh, stale, fetching or inactive, its data and how many components observe it. I can manually refetch, invalidate or trigger error states. It's my first stop when debugging unexpected refetches, stale data or key mismatches, and it isn't included in production bundles."*

---

## Phase 2 — Queries

### useQuery Basics

**Q: How does `useQuery` work?**
A: `useQuery({ queryKey, queryFn, ...options })` subscribes the component to a cache entry identified by `queryKey`:
- If the data is **cached and fresh**, it returns it immediately with no request.
- If it's **missing or stale**, it calls `queryFn` (in the background if cached data exists) and re-renders when the result arrives.
- Multiple components using the **same key share one request and one cache entry**.

```tsx
type Todo = { id: number; title: string; done: boolean };

async function fetchTodos({ signal }: { signal: AbortSignal }): Promise<Todo[]> {
  const res = await fetch('/api/todos', { signal });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json();
}

function TodoList() {
  const { data, isPending, isError, error, isFetching } = useQuery({ queryKey: ['todos'], queryFn: fetchTodos });

  if (isPending) return <Spinner />;
  if (isError) return <ErrorBanner message={error.message} />;
  return (
    <>
      {isFetching && <small>Refreshing…</small>}
      <ul>{data.map((t) => <li key={t.id}>{t.title}</li>)}</ul>
    </>
  );
}
```

Interview-ready answer: *"useQuery takes a unique key and a function that returns a promise. It reads from the cache for that key: fresh data is returned instantly, missing or stale data triggers a fetch, and every component using the same key shares one request. It returns data plus status flags like isPending, isError and isFetching, and TypeScript infers the data type from the queryFn."*

**Q: What are query keys, and how should you structure them?**
A: The query key is an **array that uniquely identifies a cache entry**. It's **hashed deterministically**, so object key order doesn't matter. Include **every variable the `queryFn` depends on**: when one changes, it's a new key, so a new entry is fetched automatically. Structure keys **from generic to specific** so you can invalidate whole groups by **prefix**.

```ts
['todos']                                   // all todos
['todos', 'list', { status: 'done', page: 2 }]   // a filtered list
['todos', 'detail', 42]                     // one todo

useQuery({ queryKey: ['todos', 'list', filters], queryFn: () => fetchTodos(filters) });
// The filters change → a new key → automatic fetch and cache per filter combination

queryClient.invalidateQueries({ queryKey: ['todos'] });            // prefix: invalidates ALL of the above
queryClient.invalidateQueries({ queryKey: ['todos', 'detail', 42] }); // just one
```

Interview-ready answer: *"A query key is an array that uniquely identifies cached data, and it must contain every input the fetch depends on, like IDs and filters. When those change the key changes, and React Query fetches and caches that variant automatically. I structure keys hierarchically, from entity to scope to params, so I can invalidate everything about todos with one prefix, or just one detail."*

**Q: What is a query key factory, and why use one?**
A: A central object that **builds all keys for an entity**. It removes typo-prone inline arrays, keeps the hierarchy consistent, and makes invalidation precise. Combined with `queryOptions`, the key and `queryFn` live together and stay type-safe.

```ts
// features/todos/queries.ts
export const todoKeys = {
  all: ['todos'] as const,
  lists: () => [...todoKeys.all, 'list'] as const,
  list: (filters: TodoFilters) => [...todoKeys.lists(), filters] as const,
  details: () => [...todoKeys.all, 'detail'] as const,
  detail: (id: number) => [...todoKeys.details(), id] as const,
};

export const todoQueries = {
  list: (filters: TodoFilters) => queryOptions({ queryKey: todoKeys.list(filters), queryFn: () => api.getTodos(filters) }),
  detail: (id: number) => queryOptions({ queryKey: todoKeys.detail(id), queryFn: () => api.getTodo(id), staleTime: 30_000 }),
};

// Usage
useQuery(todoQueries.detail(42));
queryClient.invalidateQueries({ queryKey: todoKeys.lists() });   // every list, but not the details
```

Interview-ready answer: *"A query key factory is a single object per feature that generates every key hierarchically: all, lists, a list for given filters, details, a detail by ID. It prevents typos and inconsistent keys, and makes invalidation precise, like invalidating all lists after a create without touching the details. I pair it with queryOptions so keys and fetchers are defined once and reused everywhere."*

**Q: What is the `queryOptions` helper?**
A: A v5 helper that **bundles `queryKey`, `queryFn` and options into one typed object**. You can reuse it in `useQuery`, `useSuspenseQuery`, `prefetchQuery`, `ensureQueryData`, and in `getQueryData`/`setQueryData`, where the **key carries the data type**. One definition, used everywhere, fully typed.

```ts
import { queryOptions } from '@tanstack/react-query';

export const userQuery = (id: string) =>
  queryOptions({
    queryKey: ['users', id],
    queryFn: () => api.getUser(id),
    staleTime: 5 * 60_000,
  });

useQuery(userQuery(id));                                   // component
await queryClient.prefetchQuery(userQuery(id));            // prefetch (route loader or server)
const user = queryClient.getQueryData(userQuery(id).queryKey);   // typed as User | undefined
```

Interview-ready answer: *"queryOptions is a typed factory for a query's key, function and options. I define each query once with it and reuse it for useQuery, suspense queries, prefetching in loaders or on the server, and direct cache reads and writes, where the key carries the data type, so getQueryData is correctly typed without generics."*

**Q: How should the `queryFn` handle errors?**
A: The `queryFn` must **throw (or return a rejected promise)** to signal an error; that's the only way React Query knows it failed and should retry or set `isError`. **`fetch` does not throw on 4xx/5xx**, so check `res.ok` and throw yourself. Axios throws automatically. Use the provided **`signal`** for cancellation.

```ts
async function getUser(id: string, signal?: AbortSignal) {
  const res = await fetch(`/api/users/${id}`, { signal });
  if (!res.ok) {
    const body = await res.json().catch(() => ({}));
    throw new ApiError(res.status, body.message ?? `HTTP ${res.status}`);  // ✅ React Query sees the failure
  }
  return UserSchema.parse(await res.json());                              // also validate the shape
}

useQuery({ queryKey: ['users', id], queryFn: ({ signal }) => getUser(id, signal) });
```

Interview-ready answer: *"A queryFn signals failure only by throwing or rejecting. fetch resolves even for 404s and 500s, so I check res.ok and throw a typed error that includes the status. That lets retries and isError work, and lets a custom retry function skip retrying 4xx errors. I also pass the provided AbortSignal so cancelled queries abort the request."*

### Query State

**Q: What is the difference between `status` and `fetchStatus` (`isPending`, `isLoading`, `isFetching`)?**
A: Two independent dimensions:
- **`status`** describes **the data**:
  - `'pending'`: no data yet.
  - `'error'`.
  - `'success'`: there is data.
- **`fetchStatus`** describes **the queryFn**:
  - `'fetching'`: a request is running.
  - `'paused'`: wants to fetch but is offline.
  - `'idle'`.

The boolean flags derived from them:
- **`isPending`**: no data yet. Use it for the first-load skeleton.
- **`isFetching`**: any request is in flight, **including background refetches**.
- **`isLoading`** = `isPending && isFetching`: the very first fetch is in progress. It is false for a disabled query that is pending but idle.

```tsx
const { data, status, fetchStatus, isPending, isLoading, isFetching } = useQuery(todoQueries.list(filters));

// First visit:            status 'pending', fetchStatus 'fetching' → isPending, isLoading, isFetching all true
// Background refetch:     status 'success', fetchStatus 'fetching' → only isFetching is true (data still shown)
// Disabled (enabled: false): status 'pending', fetchStatus 'idle'  → isPending true, isLoading false

if (isPending) return <Skeleton />;               // nothing to show yet
return <>{isFetching && <TopBarSpinner />}<List items={data} /></>;   // subtle background indicator
```

Interview-ready answer: *"status is about the data (pending, error or success), and fetchStatus is about the network (fetching, paused or idle). isPending means no data yet, so I show a skeleton. isFetching means any request is in flight, including background refetches, so I show a subtle indicator while keeping the data on screen. isLoading is the combination of the two, the first fetch actually happening. That distinction matters for disabled queries, which are pending but not loading."*

**Q: How do you make a dependent or conditional query?**
A: Use **`enabled`**. While it's `false`, the query doesn't run (`status: 'pending'`, `fetchStatus: 'idle'`). Put the dependency in the key, so it fetches once the value exists and refetches if it changes. For type-safe disabling, v5 also offers **`skipToken`**.

```tsx
const { data: user } = useQuery({ queryKey: ['user', email], queryFn: () => getUserByEmail(email) });

const { data: projects } = useQuery({
  queryKey: ['projects', user?.id],
  queryFn: () => getProjects(user!.id),
  enabled: !!user?.id,                        // waits for the first query
});

// skipToken: no non-null assertion needed, and TS knows the fn won't run without an id
import { skipToken } from '@tanstack/react-query';
useQuery({ queryKey: ['projects', userId], queryFn: userId ? () => getProjects(userId) : skipToken });
```

Interview-ready answer: *"I set enabled to a boolean derived from the data the query needs, and include that value in the key. The query stays idle until it's ready, then fetches automatically. skipToken is the type-safe alternative. Dependent queries are sequential by nature, so if an ID is already known, from the URL for example, I let both queries run in parallel."*

**Q: What does the `select` option do?**
A: It **transforms or derives** data for a component **without changing the cache**. Each observer can select a different shape from the same entry, and the component **re-renders only when the selected result changes** (thanks to structural sharing). Memoise `select` with `useCallback`, or define it outside the component, if it's expensive.

```tsx
const todosQuery = queryOptions({ queryKey: ['todos'], queryFn: fetchTodos });

function TodoCount() {
  // Re-renders only when the COUNT changes, not when a todo's title changes
  const { data: count } = useQuery({ ...todosQuery, select: (todos) => todos.length });
  return <span>{count}</span>;
}

function DoneTodos() {
  const { data } = useQuery({ ...todosQuery, select: (todos) => todos.filter((t) => t.done) });
  return <List items={data ?? []} />;
}
```

Interview-ready answer: *"select derives a view of cached data per component, like a count, a filtered list or a mapped shape, without mutating the cache or refetching. It also acts like a selector: the component re-renders only when its selected value actually changes, which is a cheap performance win for components that need just part of a large response."*

**Q: What is `useSuspenseQuery`?**
A: A v5 hook that integrates with **React Suspense**. While loading it **suspends** (the nearest `<Suspense>` shows its fallback), and errors **throw to the nearest error boundary**. **`data` is always defined** (typed non-undefined), so components contain only the success path. It doesn't support `enabled` or `placeholderData`.

```tsx
function UserProfile({ id }: { id: string }) {
  const { data: user } = useSuspenseQuery(userQuery(id));   // user is User, never undefined
  return <h1>{user.name}</h1>;
}

<ErrorBoundary fallback={<p>Couldn’t load the profile.</p>}>
  <Suspense fallback={<ProfileSkeleton />}>
    <UserProfile id="42" />
  </Suspense>
</ErrorBoundary>
```

Avoid **waterfalls**: two `useSuspenseQuery` calls in one component run **sequentially**. Use `useSuspenseQueries`, or prefetch both first.

Interview-ready answer: *"useSuspenseQuery suspends while loading and throws errors to an error boundary, so data is always defined and the component only handles the success case. Loading and error UI move to Suspense and ErrorBoundary wrappers. I watch out for sequential waterfalls when one component has several suspense queries, and use useSuspenseQueries or prefetching for those."*

---

## Phase 3 — Caching & Refetching

### Cache Lifecycle

**Q: What is the difference between `staleTime` and `gcTime`?**
A: They control two different things:
- **`staleTime`** (default **0**) is how long data counts as **fresh**. Fresh data is served from the cache with **no refetch**. Once stale, it's still shown, but refetched in the background on the next trigger.
- **`gcTime`** (default **5 minutes**, formerly `cacheTime`) is how long an **inactive** query (one with no mounted observers) stays in memory **before it's garbage-collected**.

```ts
useQuery({ queryKey: ['countries'], queryFn: getCountries, staleTime: Infinity });  // never goes stale automatically
useQuery({ queryKey: ['feed'], queryFn: getFeed, staleTime: 30_000 });              // fresh for 30 s
useQuery({ queryKey: ['report', id], queryFn: getReport, gcTime: 30 * 60_000 });    // keep for 30 min after unmount

// Timeline with staleTime 30s, gcTime 5m:
// t=0   fetch → fresh
// t=30s stale (still shown; refetches on mount, focus or reconnect)
// unmount → inactive → removed from the cache 5 min later unless used again
```

Interview-ready answer: *"staleTime decides when cached data needs refreshing: while fresh, it's served without any request. gcTime decides when unused data is deleted from memory: after the last component using it unmounts, it lingers for gcTime so returning to the page is instant. The default staleTime of zero causes lots of refetching, so I usually set a sensible default like a minute and tune it per query."*

**Q: When does a query refetch automatically?**
A: **Stale** queries refetch on these triggers, each configurable:
- **`refetchOnMount`**: a new component mounts with the key.
- **`refetchOnWindowFocus`**: the user returns to the tab.
- **`refetchOnReconnect`**: the network comes back.
- **`refetchInterval`**: polling, which runs regardless of staleness.

**Fresh** queries don't refetch on these triggers. You can also refetch manually with `refetch()` or `invalidateQueries`.

```ts
useQuery({
  queryKey: ['notifications'],
  queryFn: getNotifications,
  staleTime: 10_000,
  refetchOnWindowFocus: true,       // default true
  refetchOnReconnect: true,         // default true
  refetchOnMount: true,             // default true ('always' = even when fresh)
  refetchInterval: 30_000,          // poll every 30 s
  refetchIntervalInBackground: false,
});
```

Interview-ready answer: *"Stale queries refetch in the background when a component mounts, when the window regains focus and when the network reconnects, and on an interval if polling is set. Fresh queries skip those triggers, which is why staleTime is the main knob for controlling request volume. I also refetch explicitly through invalidation after mutations."*

**Q: How does request deduplication work?**
A: If several components (or one component re-rendering) ask for the **same query key** while a request is **already in flight**, React Query **reuses that promise**: one network request, and every observer gets the result. Combined with `staleTime`, mounting the same data in ten places costs **one** request.

```tsx
function Header() { const { data } = useQuery(userQuery('me')); return <Avatar user={data} />; }
function Sidebar() { const { data } = useQuery(userQuery('me')); return <Greeting user={data} />; }
function Settings() { const { data } = useQuery(userQuery('me')); return <Form user={data} />; }
// All three mount together → ONE request to /api/users/me; all three re-render when it resolves
```

This means you can **call the query hook wherever the data is needed** instead of fetching high up and prop-drilling.

Interview-ready answer: *"Queries with the same key share a single in-flight request and a single cache entry, so many components can call the same hook at once and only one request is made. That lets me colocate data needs, with each component calling useQuery for what it needs, instead of fetching at the top and prop-drilling the result."*

**Q: What is structural sharing?**
A: When new data arrives, React Query **deep-compares** it with the old data and **keeps the old references for unchanged parts**. If nothing changed, `data` keeps the **same reference**, so components, `useMemo` and `React.memo` don't re-render or recompute. If one item in a list changed, only that item gets a new reference. It's on by default for JSON-compatible data.

```ts
// Refetch returns identical JSON → data === previousData (no re-render from data)
// Only todo #3 changed → newData[0] === oldData[0], newData[3] !== oldData[3]
const MemoTodo = React.memo(TodoItem);   // only #3 re-renders

useQuery({ queryKey: ['big'], queryFn: getHugePayload, structuralSharing: false });  // opt out for huge or non-JSON data
```

Interview-ready answer: *"Structural sharing means that after a refetch, React Query reuses the previous object references for any part of the data that didn't change. Identical refetches don't trigger re-renders, and memoised list items only re-render if their own item changed. It's automatic, and I only disable it for very large or non-serialisable payloads where the comparison costs too much."*

**Q: What is the difference between `initialData` and `placeholderData`?**
A: **`initialData`** is **real data persisted to the cache**. It is treated as successful, respects `staleTime` (set `initialDataUpdatedAt`), and is shared with other observers. Use it when you genuinely have the data, for example from a list response or SSR. **`placeholderData`** is **temporary display data**, **not cached**. The query still fetches, and `isPlaceholderData` is true meanwhile. Use it for previews and skeleton-like content, or for **keeping the previous page** while paginating.

```tsx
// initialData: seed a detail from the list cache (a real, cached entity)
useQuery({
  queryKey: ['todos', 'detail', id],
  queryFn: () => getTodo(id),
  initialData: () => queryClient.getQueryData<Todo[]>(['todos'])?.find((t) => t.id === id),
  initialDataUpdatedAt: () => queryClient.getQueryState(['todos'])?.dataUpdatedAt,
});

// placeholderData: a preview while the real fetch runs (not cached)
useQuery({ queryKey: ['post', id], queryFn: () => getPost(id), placeholderData: { title: 'Loading…', body: '' } });

// placeholderData for pagination: keep the previous page visible
useQuery({ queryKey: ['posts', page], queryFn: () => getPosts(page), placeholderData: keepPreviousData });
```

Interview-ready answer: *"initialData is treated as real data: it goes into the cache, counts as success and follows staleTime. I use it when I already have the entity, like seeding a detail view from the list cache. placeholderData is just a temporary stand-in for display. It's never cached, the real fetch still happens, and isPlaceholderData tells me it's not real. I use it for previews and for keeping the previous page visible during pagination."*

---

## Phase 4 — Mutations

### useMutation

**Q: How does `useMutation` work?**
A: `useMutation({ mutationFn, ...callbacks })` handles **writes** (create, update, delete). Unlike queries it **doesn't run automatically** and isn't cached by key. Call **`mutate(variables)`** (fire-and-forget; errors are handled by the callbacks) or **`mutateAsync(variables)`** (returns a promise you `await`/`try`). It exposes `isPending`, `isError`, `isSuccess`, `data`, `error`, `variables` and `reset()`.

```tsx
function NewTodoForm() {
  const queryClient = useQueryClient();
  const createTodo = useMutation({
    mutationFn: (title: string) => api.createTodo({ title }),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: todoKeys.lists() }),
    onError: (err) => toast.error(err.message),
  });

  return (
    <form onSubmit={(e) => { e.preventDefault(); createTodo.mutate(new FormData(e.currentTarget).get('title') as string); }}>
      <input name="title" required />
      <button disabled={createTodo.isPending}>{createTodo.isPending ? 'Adding…' : 'Add'}</button>
      {createTodo.isError && <p role="alert">{createTodo.error.message}</p>}
    </form>
  );
}
```

Interview-ready answer: *"useMutation wraps a function that changes server data. It runs only when I call mutate or mutateAsync, and gives me pending, error and success state plus lifecycle callbacks. The important part is the onSuccess or onSettled callback, where I keep the cache consistent, usually by invalidating the affected queries."*

**Q: What is the difference between `mutate` and `mutateAsync`?**
A: **`mutate`** returns nothing and **never throws**. Errors go to `onError`/`error`, so it's safe to call from event handlers. **`mutateAsync`** returns a **promise** that **rejects on error**, which is useful when you need to `await` the result (sequencing, form libraries), but you **must catch** it or you get an unhandled rejection. Both accept per-call callbacks.

```tsx
// mutate: fire and forget; handle errors via callbacks
updateTodo.mutate({ id, done: true }, { onSuccess: () => toast.success('Saved') });

// mutateAsync: await it, so it needs try/catch
async function onSubmit(values: FormValues) {
  try {
    const created = await createProject.mutateAsync(values);
    router.push(`/projects/${created.id}`);
  } catch (err) {
    form.setError('root', { message: (err as Error).message });
  }
}
```

Interview-ready answer: *"mutate is fire-and-forget: errors are captured in the mutation state and callbacks, so it never throws. mutateAsync returns a promise I can await, which is handy for chaining steps or integrating with form submit handlers, but I have to wrap it in try/catch. Mutation-level callbacks run before per-call ones, and per-call callbacks don't fire if the component unmounts first."*

**Q: How do you update the cache after a mutation?**
A: Two strategies, often combined:
1. **Invalidate** with `invalidateQueries({ queryKey })`. It marks the matching queries stale and **refetches the active ones**. It's simple and always correct, and best when the server computes things (ordering, totals, derived fields).
2. **Update directly** with `setQueryData(key, updater)`, writing the mutation's **response** into the cache. That avoids a round trip and fits when the response is the full updated entity. Updates must be **immutable**.

```ts
const updateTodo = useMutation({
  mutationFn: api.updateTodo,
  onSuccess: (updated) => {
    queryClient.setQueryData(todoKeys.detail(updated.id), updated);              // direct: the detail
    queryClient.setQueryData<Todo[]>(todoKeys.list({}), (old) =>                  // direct: the list item
      old?.map((t) => (t.id === updated.id ? updated : t)));
    return queryClient.invalidateQueries({ queryKey: todoKeys.lists() });        // reconcile filtered lists
  },
});
```

Returning the `invalidateQueries` promise keeps the mutation **pending until the refetch finishes**.

Interview-ready answer: *"Either invalidate the affected query keys so they refetch from the server, which is the simplest way to stay correct, or write the server's response straight into the cache with setQueryData, immutably, to skip a request. I often do both: set the entity I know, then invalidate lists whose membership or order might change. Returning the invalidation promise from onSuccess keeps isPending true until the fresh data is in."*

**Q: How do you implement optimistic updates?**
A: Update the UI **before** the server responds, and roll back on failure. There are **two approaches** in v5:

**1. Via the cache.** This updates everywhere the data is shown.
```ts
const toggleTodo = useMutation({
  mutationFn: (todo: Todo) => api.updateTodo(todo.id, { done: !todo.done }),
  onMutate: async (todo) => {
    await queryClient.cancelQueries({ queryKey: todoKeys.lists() });           // stop in-flight fetches overwriting us
    const previous = queryClient.getQueryData<Todo[]>(todoKeys.list({}));      // snapshot
    queryClient.setQueryData<Todo[]>(todoKeys.list({}), (old) =>
      old?.map((t) => (t.id === todo.id ? { ...t, done: !t.done } : t)));
    return { previous };                                                        // → context
  },
  onError: (_err, _todo, context) => queryClient.setQueryData(todoKeys.list({}), context?.previous),   // roll back
  onSettled: () => queryClient.invalidateQueries({ queryKey: todoKeys.lists() }),                      // reconcile
});
```

**2. Via the UI with `variables`.** This is simpler and local to one place, with no cache writes or rollback.
```tsx
const addTodo = useMutation({ mutationFn: api.createTodo, onSettled: () => queryClient.invalidateQueries({ queryKey: todoKeys.lists() }) });

<ul>
  {todos.map((t) => <li key={t.id}>{t.title}</li>)}
  {addTodo.isPending && <li style={{ opacity: 0.5 }}>{addTodo.variables.title}</li>}   {/* the pending item */}
  {addTodo.isError && <li className="error">{addTodo.variables.title} <button onClick={() => addTodo.mutate(addTodo.variables)}>Retry</button></li>}
</ul>
```

Interview-ready answer: *"For cache-based optimism, in onMutate I cancel outgoing queries for the key, snapshot the current data, apply the optimistic change and return the snapshot as context. onError restores it and onSettled invalidates to sync with the server. If the optimistic item only appears in one place, v5 has a simpler approach: render mutation.variables while isPending, with no cache writes or rollback. I use optimism for high-confidence, reversible actions like toggles and likes."*

**Q: Why call `cancelQueries` in `onMutate`?**
A: If a **refetch is already in flight** for that key (a background refetch, polling, or a focus refetch), it can **resolve after** your optimistic `setQueryData` and **overwrite it with old server data**. The UI then flickers back before the mutation finishes. `cancelQueries` aborts those requests (and their results) so the optimistic state survives until `onSettled` refetches properly.

```ts
onMutate: async (newTodo) => {
  await queryClient.cancelQueries({ queryKey: todoKeys.lists() });   // ← prevents a stale overwrite
  // ...snapshot + optimistic setQueryData
},
```

Interview-ready answer: *"An in-flight refetch for the same key could land after my optimistic write and replace it with stale data, making the UI jump back. Cancelling queries for that key in onMutate prevents the race, and onSettled refetches fresh data once the mutation is done. It matters most with polling or window-focus refetching."*

**Q: What do `onMutate`, `onSuccess`, `onError` and `onSettled` do, and in what order do they run?**
A: In order:
1. **`onMutate(variables)`** runs **before** `mutationFn`. Use it for optimistic updates, and return a **context** (the snapshot).
2. **`onSuccess(data, variables, context)`** runs on success: cache writes, invalidation, toasts, navigation.
3. **`onError(error, variables, context)`** runs on failure: roll back using the context, show an error.
4. **`onSettled(data, error, variables, context)`** **always** runs after success or error, like `finally`. Use it for invalidation and reconciliation.

Callbacks defined in `useMutation` run **before** the per-call callbacks passed to `mutate(vars, {...})`.

```ts
useMutation({
  mutationFn: api.deleteTodo,
  onMutate: async (id) => { /* optimistic remove */ return { previous }; },
  onError: (err, id, ctx) => { queryClient.setQueryData(key, ctx?.previous); toast.error('Delete failed'); },
  onSuccess: () => toast.success('Deleted'),
  onSettled: () => queryClient.invalidateQueries({ queryKey: todoKeys.lists() }),
});
```

Interview-ready answer: *"onMutate runs first, before the request, and its return value becomes the context. Then onSuccess or onError, and finally onSettled, which always runs. Optimistic updates go in onMutate, rollback in onError using the context, and invalidation in onSettled so the cache resyncs whatever happened. Hook-level callbacks run before the ones passed to mutate."*

---

## Phase 5 — Pagination & Infinite Queries

### Paginated Queries

**Q: How do you implement paginated queries without a loading flash between pages?**
A: Put the page in the **query key**, so each page is cached separately, and use **`placeholderData: keepPreviousData`**. The previous page stays on screen while the next one loads, and `isPlaceholderData` lets you dim the list or disable "Next". **Prefetch** the next page to make it instant.

```tsx
import { keepPreviousData, useQuery, useQueryClient } from '@tanstack/react-query';

function Projects() {
  const [page, setPage] = useState(1);
  const queryClient = useQueryClient();
  const { data, isPending, isPlaceholderData } = useQuery({
    queryKey: ['projects', page],
    queryFn: () => api.getProjects(page),
    placeholderData: keepPreviousData,
  });

  useEffect(() => {                                     // prefetch the next page
    if (data?.hasMore) queryClient.prefetchQuery({ queryKey: ['projects', page + 1], queryFn: () => api.getProjects(page + 1) });
  }, [data, page, queryClient]);

  if (isPending) return <Skeleton />;
  return (
    <>
      <ProjectList items={data.items} style={{ opacity: isPlaceholderData ? 0.6 : 1 }} />
      <button onClick={() => setPage((p) => Math.max(1, p - 1))} disabled={page === 1}>Previous</button>
      <button onClick={() => setPage((p) => p + 1)} disabled={isPlaceholderData || !data.hasMore}>Next</button>
    </>
  );
}
```

Interview-ready answer: *"Each page gets its own key, like ['projects', page], so pages are cached independently and going back is instant. placeholderData set to keepPreviousData keeps the old page visible while the new one loads, avoiding a skeleton flash, and isPlaceholderData lets me dim the list and disable Next. I prefetch the next page so forward navigation is usually instant too."*

### Infinite Queries

**Q: How does `useInfiniteQuery` work?**
A: It manages **"load more" and infinite scroll** data as a list of **pages** under one key (`data.pages`, `data.pageParams`). v5 requires **`initialPageParam`**. **`getNextPageParam(lastPage, allPages)`** returns the next cursor, or `undefined` when there are no more pages. You call **`fetchNextPage()`**, and **`hasNextPage`** and **`isFetchingNextPage`** drive the UI. `maxPages` limits how many pages stay in memory.

```tsx
function Feed() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage, status } = useInfiniteQuery({
    queryKey: ['feed'],
    queryFn: ({ pageParam, signal }) => api.getFeed({ cursor: pageParam, signal }),
    initialPageParam: null as string | null,
    getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined,   // undefined → hasNextPage false
    maxPages: 10,                                                       // optional memory cap
  });

  const sentinel = useRef<HTMLDivElement>(null);
  useEffect(() => {
    const el = sentinel.current;
    if (!el) return;
    const io = new IntersectionObserver(([e]) => { if (e.isIntersecting && hasNextPage && !isFetchingNextPage) fetchNextPage(); });
    io.observe(el);
    return () => io.disconnect();
  }, [hasNextPage, isFetchingNextPage, fetchNextPage]);

  if (status === 'pending') return <Spinner />;
  const posts = data.pages.flatMap((p) => p.items);
  return (
    <>
      {posts.map((post) => <PostCard key={post.id} post={post} />)}
      <div ref={sentinel}>{isFetchingNextPage ? 'Loading more…' : !hasNextPage && 'You’re all caught up'}</div>
    </>
  );
}
```

Interview-ready answer: *"useInfiniteQuery stores pages under one key. I give it an initialPageParam and a getNextPageParam that reads the next cursor from the last page, and call fetchNextPage when an IntersectionObserver sentinel becomes visible, guarded by hasNextPage and isFetchingNextPage. For very long feeds I combine it with maxPages and list virtualisation, and I prefer cursor pagination so inserts don't shift pages."*

**Q: How are infinite queries refetched, and why does that matter?**
A: When an infinite query refetches (invalidation, focus), React Query **refetches every loaded page sequentially, starting from the first**, so the pages stay consistent with each other. With 20 pages loaded, that's 20 requests. Use `maxPages`, a higher `staleTime`, or **reset** the query to page one when that's acceptable.

```ts
// Invalidation refetches all loaded pages in order
queryClient.invalidateQueries({ queryKey: ['feed'] });

// Alternatively, drop back to just the first page
queryClient.resetQueries({ queryKey: ['feed'] });

// Optimistically add a new post to the first page
queryClient.setQueryData<InfiniteData<FeedPage>>(['feed'], (old) =>
  old && { ...old, pages: [{ ...old.pages[0], items: [newPost, ...old.pages[0].items] }, ...old.pages.slice(1)] });
```

Interview-ready answer: *"Refetching an infinite query re-requests every loaded page in order, starting from the first, so cursors and items stay consistent. That's correct, but it can be expensive for long feeds. I cap memory and refetch cost with maxPages, raise staleTime, or reset to the first page, and for new items I update the first page of the cached InfiniteData immutably."*

---

## Phase 6 — Advanced Patterns

### Parallel Queries & Prefetching

**Q: How do you run parallel queries?**
A: Multiple `useQuery` calls in a component **run in parallel** automatically. For a **dynamic number** of queries (one per ID), use **`useQueries`**: hooks can't be called in loops. Its `combine` option merges the results into one value.

```tsx
const user = useQuery(userQuery(id));          // these two run concurrently
const stats = useQuery(statsQuery(id));

const { products, isPending } = useQueries({
  queries: productIds.map((pid) => productQuery(pid)),
  combine: (results) => ({
    products: results.map((r) => r.data).filter(Boolean),
    isPending: results.some((r) => r.isPending),
  }),
});
```

Interview-ready answer: *"Independent useQuery calls already fetch in parallel. When the number of queries depends on data, I use useQueries with an array of query options, and combine to merge them into a single shape. Each still has its own cache entry, so results are shared with other components."*

**Q: What is prefetching, and where do you do it?**
A: Loading data into the cache **before** a component needs it, so it renders instantly:
- **`prefetchQuery`** fetches only if the data is missing or stale, and never throws.
- **`ensureQueryData`** returns the cached data or fetches it, which suits route loaders.
- **`prefetchInfiniteQuery`** does the same for infinite queries.

Common places: **link hover or focus**, **route loaders** (React Router, TanStack Router), the **next page** of a paginated list, and **on the server** for SSR.

```tsx
function ProjectLink({ id, name }: { id: string; name: string }) {
  const queryClient = useQueryClient();
  const prefetch = () => queryClient.prefetchQuery(projectQuery(id));      // respects staleTime
  return <Link to={`/projects/${id}`} onMouseEnter={prefetch} onFocus={prefetch}>{name}</Link>;
}

// React Router loader: data is ready before the route renders
export const loader = (queryClient: QueryClient) => async ({ params }: LoaderFunctionArgs) =>
  queryClient.ensureQueryData(projectQuery(params.id!));
```

Interview-ready answer: *"Prefetching warms the cache ahead of time. I call prefetchQuery on link hover or focus, use ensureQueryData in route loaders so pages render without a spinner, prefetch the next page of paginated lists, and prefetch on the server for SSR. It uses the same keys and staleTime, so it never causes duplicate requests."*

### Reliability

**Q: How do retries work, and how do you customise them?**
A: By default, failed queries **retry 3 times** on the client (0 on the server) with **exponential backoff** (`retryDelay`: 1 s, 2 s, 4 s… capped at 30 s). Mutations **don't retry** by default. Customise with a number, a boolean, or a **function**, for example **don't retry 4xx errors**, since a 404 or 401 won't fix itself.

```ts
new QueryClient({
  defaultOptions: {
    queries: {
      retry: (failureCount, error) => {
        if (error instanceof ApiError && error.status >= 400 && error.status < 500) return false;  // client errors
        return failureCount < 3;
      },
      retryDelay: (attempt) => Math.min(1000 * 2 ** attempt, 30_000),
    },
    mutations: { retry: 0 },     // retrying writes can duplicate them unless the API is idempotent
  },
});
```

Interview-ready answer: *"Queries retry three times with exponential backoff by default, and mutations don't retry. I pass a retry function that skips 4xx errors like 401, 403 and 404, which won't succeed on retry, and only retries transient failures like timeouts and 5xx. I leave mutations at zero retries unless the endpoint is idempotent, for example protected with an idempotency key."*

**Q: How do you handle errors globally and with error boundaries?**
A: Several layers:
- **Per query:** `isError` and `error` for inline messages.
- **Global side effects:** `QueryCache({ onError })` and `MutationCache({ onError })` on the client, for toasts and logging, **once per query rather than once per component**. In v5, `useQuery` no longer has `onSuccess`/`onError` callbacks.
- **Error boundaries:** `throwOnError: true` (or a function) throws errors to the nearest boundary, and `QueryErrorResetBoundary` lets "Try again" refetch.

```tsx
const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (error, query) => {
      if (query.state.data !== undefined) toast.error(`Background update failed: ${error.message}`);   // only if we already showed data
      Sentry.captureException(error);
    },
  }),
  mutationCache: new MutationCache({ onError: (error) => toast.error(error.message) }),
  defaultOptions: { queries: { throwOnError: (error, query) => query.state.data === undefined } },   // only first-load errors go to the boundary
});

<QueryErrorResetBoundary>
  {({ reset }) => (
    <ErrorBoundary onReset={reset} fallbackRender={({ resetErrorBoundary }) => <button onClick={resetErrorBoundary}>Try again</button>}>
      <Dashboard />
    </ErrorBoundary>
  )}
</QueryErrorResetBoundary>
```

Interview-ready answer: *"Inline isError for local messages. Global logging and toasts in the QueryCache and MutationCache onError callbacks, so each failure is reported once, not once per component; v5 removed onError from useQuery for exactly that reason. throwOnError sends errors to error boundaries, often only for first-load failures, with QueryErrorResetBoundary so the Try Again button refetches."*

**Q: How does query cancellation work?**
A: React Query passes an **`AbortSignal`** to every `queryFn`. Forward it to `fetch` or Axios. The request is then **aborted automatically** when the query becomes unused (the component unmounts before it resolves), when the key changes mid-flight, or when you call `queryClient.cancelQueries`. Without forwarding the signal, cancellation only ignores the result.

```ts
useQuery({
  queryKey: ['search', term],
  queryFn: async ({ signal }) => {
    const res = await fetch(`/api/search?q=${encodeURIComponent(term)}`, { signal });   // aborted when the term changes
    if (!res.ok) throw new Error('Search failed');
    return res.json();
  },
});

axios.get('/api/items', { signal });               // Axios supports AbortSignal too

queryClient.cancelQueries({ queryKey: ['search'] }); // manual cancel (e.g. before an optimistic update)
```

Interview-ready answer: *"Every queryFn receives an AbortSignal. If I pass it to fetch or Axios, React Query aborts the network request when nothing needs it anymore, such as on unmount or a key change, or when I call cancelQueries. That saves bandwidth and avoids wasted work on rapid typing or navigation, and it's built in, so no manual AbortController is needed."*

**Q: How do you implement polling and real-time updates?**
A: **Polling:** `refetchInterval` (a number, or a function for adaptive intervals). It pauses in background tabs unless `refetchIntervalInBackground` is set. **Real-time pushes** (WebSocket or SSE): keep React Query as the cache and **write incoming events into it** with `setQueryData`, or **invalidate** the affected keys, so components don't change.

```tsx
// Polling, faster while a job is running
useQuery({
  queryKey: ['job', id],
  queryFn: () => api.getJob(id),
  refetchInterval: (query) => (query.state.data?.status === 'running' ? 2_000 : false),   // stops when done
});

// WebSocket → cache
function useOrderEvents() {
  const queryClient = useQueryClient();
  useEffect(() => {
    const ws = new WebSocket(WS_URL);
    ws.onmessage = (e) => {
      const event = JSON.parse(e.data);
      if (event.type === 'order.updated') {
        queryClient.setQueryData(['orders', event.order.id], event.order);          // a precise update
        queryClient.invalidateQueries({ queryKey: ['orders', 'list'] });           // lists refetch
      }
    };
    return () => ws.close();
  }, [queryClient]);
}
```

Interview-ready answer: *"For polling I use refetchInterval, often as a function that polls fast while something is in progress and stops when it's done. For push-based real-time I keep React Query as the source of truth: the WebSocket or SSE handler writes events into the cache with setQueryData, or invalidates related keys, so components keep using normal useQuery hooks."*

**Q: How does TanStack Query behave offline?**
A: By default (`networkMode: 'online'`), queries **don't fire while offline**: they're **paused** (`fetchStatus: 'paused'`) and resume on reconnect, and mutations are paused and resumed too. `'always'` ignores the network status (local or mock APIs). `'offlineFirst'` tries once and then pauses (service-worker caches). To **survive reloads**, persist the cache with `persistQueryClient`.

```tsx
import { PersistQueryClientProvider } from '@tanstack/react-query-persist-client';
import { createSyncStoragePersister } from '@tanstack/query-sync-storage-persister';

const queryClient = new QueryClient({ defaultOptions: { queries: { gcTime: 24 * 60 * 60_000 } } }); // gcTime ≥ maxAge
const persister = createSyncStoragePersister({ storage: window.localStorage });

<PersistQueryClientProvider client={queryClient} persistOptions={{ persister, maxAge: 24 * 60 * 60_000 }}>
  <App />
</PersistQueryClientProvider>

const { fetchStatus } = useQuery(todosQuery);
{fetchStatus === 'paused' && <Banner>You’re offline. Showing saved data.</Banner>}
```

Interview-ready answer: *"In the default online mode, queries and mutations pause when the browser is offline and resume automatically on reconnect, with fetchStatus 'paused' so the UI can show an offline banner while cached data stays visible. For true offline support I persist the cache to storage with the persist plugin, setting gcTime at least as long as maxAge, and for writes I rely on paused mutations resuming."*

---

## Phase 7 — SSR & Next.js

### Hydration

**Q: How do you use TanStack Query with server-side rendering?**
A: **Prefetch on the server, dehydrate the cache, hydrate it on the client**:
1. Create a QueryClient **per request** on the server.
2. `prefetchQuery` the data the page needs.
3. `dehydrate(queryClient)` serialises the cache into the HTML.
4. `<HydrationBoundary state={...}>` puts it into the client's cache.
5. Client components call `useQuery` with the **same keys**, so the data is already there: no loading state, no duplicate request.

Set **`staleTime` above 0** on the client, or it refetches immediately after hydration.

```tsx
// app/posts/page.tsx (Next.js App Router: a Server Component)
import { dehydrate, HydrationBoundary, QueryClient } from '@tanstack/react-query';

export default async function PostsPage() {
  const queryClient = new QueryClient();                      // per request
  await queryClient.prefetchQuery(postsQuery());              // the same queryOptions as the client
  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <Posts />                                               {/* a client component */}
    </HydrationBoundary>
  );
}

// components/Posts.tsx
'use client';
export function Posts() {
  const { data } = useQuery(postsQuery());                    // instantly available from hydration
  return <PostList posts={data!} />;
}
```

Interview-ready answer: *"On the server I create a fresh QueryClient per request, prefetch the queries the page needs, and pass dehydrate(queryClient) to a HydrationBoundary. On the client, useQuery with the same keys reads the hydrated data immediately: no spinner and no duplicate fetch. Afterwards it behaves like normal client cache, with background refetching. I set a staleTime above zero so it doesn't refetch right after hydration."*

**Q: How do you set up the QueryClient in the Next.js App Router?**
A: Make a **`getQueryClient`** helper. On the **server**, it always creates a **new client per request**, because sharing one between users would leak data. In the **browser**, it reuses a **singleton**, so React suspending during the initial render doesn't throw the cache away. Put the provider in a **client component**. Including **pending queries** in `dehydrate` enables **streaming**: the server can start a prefetch without awaiting it.

```tsx
// app/get-query-client.ts
import { QueryClient, defaultShouldDehydrateQuery, isServer } from '@tanstack/react-query';

function makeQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: { staleTime: 60_000 },                                       // avoid an immediate client refetch
      dehydrate: {
        shouldDehydrateQuery: (q) => defaultShouldDehydrateQuery(q) || q.state.status === 'pending', // stream pending
      },
    },
  });
}

let browserQueryClient: QueryClient | undefined;
export function getQueryClient() {
  if (isServer) return makeQueryClient();                                   // new per request
  return (browserQueryClient ??= makeQueryClient());                        // one per browser session
}

// app/providers.tsx
'use client';
export function Providers({ children }: { children: React.ReactNode }) {
  const queryClient = getQueryClient();
  return <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>;
}
```

Interview-ready answer: *"I use a getQueryClient helper: on the server it creates a new client for every request, so users never share a cache, and in the browser it's a singleton, so suspense during the first render doesn't discard it. The provider is a client component wrapped around the app. I set a default staleTime above zero for hydration, and dehydrate pending queries so the server can stream prefetches without awaiting them."*

**Q: Should you use TanStack Query or plain Server Component fetching in Next.js?**
A: They **complement** each other. **Server Components with `fetch`/DB calls** are the default for data that's **read on the server and rendered**: no client JS, and no cache to manage. Add **TanStack Query** where the **client needs server state**: interactive filtering and pagination, **infinite scroll**, **mutations with optimistic updates**, **polling or real-time** data, refetch on focus, and data shared across many client components. A common pattern is **prefetch in a Server Component, then use and keep fresh with `useQuery`** on the client.

```tsx
// Static article: a plain Server Component is enough
export default async function Article({ params }) {
  const post = await db.post.findUnique({ where: { slug: (await params).slug } });
  return <MDX source={post.body} />;
}

// An interactive comments section: TanStack Query on the client, prefetched on the server
<HydrationBoundary state={dehydrate(qc)}>
  <Comments postId={id} />       {/* useInfiniteQuery + optimistic useMutation */}
</HydrationBoundary>
```

Interview-ready answer: *"In the App Router, I fetch in Server Components by default when data just needs to render. It's simpler and ships no client JS. I bring in TanStack Query for client-side server state: interactive lists, infinite scroll, optimistic mutations, polling, and data shared by many client components. Often I prefetch on the server and hydrate, so I get both a fast first paint and a live client cache."*

---

## Phase 8 — Testing

### Testing Queries & Mutations

**Q: How do you test components that use TanStack Query?**
A: Render them inside a **fresh `QueryClient` per test** with **`retry: false`**: shared clients leak cache between tests, and retries make error tests slow or flaky. Mock the network with **MSW** rather than mocking the hooks, and assert with `findBy`/`waitFor` for async results.

```tsx
// test-utils.tsx
export function createTestQueryClient() {
  return new QueryClient({ defaultOptions: { queries: { retry: false, gcTime: Infinity }, mutations: { retry: false } } });
}
export function renderWithClient(ui: React.ReactElement) {
  const client = createTestQueryClient();
  return { client, ...render(<QueryClientProvider client={client}>{ui}</QueryClientProvider>) };
}

// TodoList.test.tsx
test('shows todos, then the error state', async () => {
  renderWithClient(<TodoList />);
  expect(await screen.findByText('Buy milk')).toBeInTheDocument();       // MSW handler returns todos

  server.use(http.get('/api/todos', () => HttpResponse.json({}, { status: 500 })));
  renderWithClient(<TodoList />);
  expect(await screen.findByRole('alert')).toHaveTextContent(/failed/i);
});
```

Interview-ready answer: *"Each test gets its own QueryClient with retries off, so cache state never leaks between tests and error cases fail immediately. I mock the API at the network level with MSW instead of mocking useQuery, so the real hooks, cache and loading states are exercised, and assert with findBy or waitFor for async UI."*

**Q: How do you test a custom query or mutation hook?**
A: Use **`renderHook`** with a **wrapper** that provides the test QueryClient, then **`waitFor`** the status to change. For mutations, call `mutate` inside `act` and assert on the result, the cache, or the invalidation.

```tsx
const wrapper = ({ children }: { children: React.ReactNode }) => (
  <QueryClientProvider client={createTestQueryClient()}>{children}</QueryClientProvider>
);

test('useTodos returns data', async () => {
  const { result } = renderHook(() => useTodos(), { wrapper });
  await waitFor(() => expect(result.current.isSuccess).toBe(true));
  expect(result.current.data).toHaveLength(2);
});

test('useCreateTodo invalidates the list', async () => {
  const client = createTestQueryClient();
  const spy = vi.spyOn(client, 'invalidateQueries');
  const { result } = renderHook(() => useCreateTodo(), {
    wrapper: ({ children }) => <QueryClientProvider client={client}>{children}</QueryClientProvider>,
  });
  act(() => result.current.mutate({ title: 'New' }));
  await waitFor(() => expect(result.current.isSuccess).toBe(true));
  expect(spy).toHaveBeenCalledWith({ queryKey: ['todos', 'list'] });
});
```

Interview-ready answer: *"renderHook with a wrapper that provides a fresh test QueryClient, MSW for the network, and waitFor until isSuccess or isError. For mutations I trigger mutate inside act and assert on the returned data, on the cache via getQueryData, or that the right keys were invalidated."*

---

## Phase 9 — Architecture & Scenarios

### Architecture

**Q: How do you organise TanStack Query code in a large app?**
A: **Per feature**:
- An **API layer** of plain functions that fetch, validate and throw.
- **Query keys and `queryOptions` factories.**
- **Custom hooks** (`useTodos`, `useCreateTodo`) that encapsulate keys, options and cache updates.

Components **never** call `useQuery` with inline keys; they use the feature's hooks. Global defaults live in one QueryClient config.

```
src/features/todos/
├── api.ts          // getTodos, getTodo, createTodo... (fetch + Zod + throw on !ok)
├── queries.ts      // todoKeys factory + queryOptions (todoQueries.list / detail)
├── hooks.ts        // useTodos(filters), useTodo(id), useCreateTodo(), useToggleTodo()
└── components/     // only use hooks
```

```ts
// hooks.ts
export const useTodos = (filters: TodoFilters) => useQuery(todoQueries.list(filters));

export function useCreateTodo() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: api.createTodo,
    onSuccess: () => queryClient.invalidateQueries({ queryKey: todoKeys.lists() }),
  });
}
```

Interview-ready answer: *"Feature-based: an API module with plain, validated fetch functions; a queries module with a key factory and queryOptions; and custom hooks that wrap useQuery and useMutation with the right keys, options and invalidation. Components only call those hooks, so cache logic lives in one place per feature and keys stay consistent."*

**Q: TanStack Query vs Redux Toolkit / RTK Query vs SWR?**
A: TanStack Query, RTK Query and SWR are all **server-state** tools. Redux Toolkit itself is a **client-state** store.

| | TanStack Query | RTK Query | SWR | Redux Toolkit (slices) |
|---|---|---|---|---|
| Purpose | Server state | Server state, on top of Redux | Server state (lightweight) | Client state |
| Cache model | Per query key | Per endpoint + tags | Per key | Manual |
| Mutations and optimistic updates | First-class | Yes (tag invalidation) | Basic | Manual |
| Infinite queries, DevTools, offline | Yes | Partial / Redux DevTools | Basic | n/a |
| Best when | Any React app; framework-agnostic | The app already uses Redux heavily | Simple, read-mostly apps | Complex shared UI state |

```ts
// RTK Query: endpoints are declared centrally
const api = createApi({ baseQuery: fetchBaseQuery({ baseUrl: '/api' }), tagTypes: ['Todo'],
  endpoints: (b) => ({ getTodos: b.query<Todo[], void>({ query: () => 'todos', providesTags: ['Todo'] }) }) });

// TanStack Query: hooks are composed where needed
useQuery({ queryKey: ['todos'], queryFn: getTodos });
```

Interview-ready answer: *"TanStack Query is a dedicated, framework-agnostic server-state library with the richest feature set: fine-grained caching, infinite queries, optimistic updates, offline support and great DevTools. RTK Query is the natural choice if the app is already built on Redux, since it uses Redux's store and tag invalidation. SWR is lighter and fine for read-heavy apps. Redux slices or Zustand still handle genuinely client-side state alongside any of them."*

**Q: Can you use TanStack Query with Zustand or Redux? How do you split responsibilities?**
A: Yes, and you should, with a **clear split**. **TanStack Query owns server data**: never copy it into Zustand. **Zustand owns client state**: filters, UI toggles, the selected ID, wizard step. Components combine them: client state feeds **query keys**.

```tsx
// Zustand: client state only
const useFilters = create<FilterState>((set) => ({
  status: 'all', search: '',
  setStatus: (status) => set({ status }),
}));

// TanStack Query: server state, keyed by the client state
function TodoList() {
  const { status, search } = useFilters();
  const { data } = useQuery(todoQueries.list({ status, search }));   // a filter change → a new key → a fetch
  return <List items={data ?? []} />;
}
// ❌ Anti-pattern: useEffect(() => store.setTodos(data), [data])  → two sources of truth
```

Interview-ready answer: *"They work well together with a strict boundary: TanStack Query is the only owner of server data, and Zustand or Redux holds client-only state like filters, selections and UI flags. Client state flows into query keys, so changing a filter automatically fetches the right data. Copying query data into a store creates two sources of truth, so I avoid it."*

**Q: What changed from React Query v4 to TanStack Query v5?**
A:
- **A single object signature only:** `useQuery({ queryKey, queryFn })`.
- **`cacheTime` → `gcTime`.**
- **`status: 'loading'` → `'pending'`**, and `isLoading` → **`isPending`** (`isLoading` now means pending and fetching).
- **`keepPreviousData` → `placeholderData: keepPreviousData`** (`isPreviousData` → `isPlaceholderData`).
- **`useErrorBoundary` → `throwOnError`.**
- **`onSuccess`/`onError`/`onSettled` removed from `useQuery`**: use the QueryCache callbacks or effects.
- **`initialPageParam` is required** for infinite queries, and `maxPages` was added.
- **Suspense hooks**: `useSuspenseQuery`, `useSuspenseInfiniteQuery`, `useSuspenseQueries`.
- **`queryOptions` / `infiniteQueryOptions`** helpers.
- Optimistic UI via **mutation `variables`**, and `useMutationState`.
- **React 18+** is required, and the bundle is smaller.

```ts
// v4
useQuery(['todos', page], () => fetchTodos(page), { keepPreviousData: true, cacheTime: 10 * 60_000, onSuccess: log });

// v5
useQuery({
  queryKey: ['todos', page],
  queryFn: () => fetchTodos(page),
  placeholderData: keepPreviousData,
  gcTime: 10 * 60_000,
});                                  // onSuccess removed from useQuery → QueryCache({ onSuccess }) or an effect on data
```

Interview-ready answer: *"v5 standardised on the object signature, renamed cacheTime to gcTime, renamed the loading status to pending with isPending, replaced keepPreviousData with placeholderData, renamed useErrorBoundary to throwOnError, and removed onSuccess and onError from useQuery because they ran once per observer. It also added first-class suspense hooks, queryOptions, required initialPageParam with maxPages for infinite queries, and simpler optimistic UI through mutation variables."*

### Scenarios

**Q (Scenario): After creating an item, the list still shows old data. Why, and how do you fix it?**
A: The mutation changed the server, but **nothing told the cache**. The list query is still **fresh** (within `staleTime`), so it isn't refetched, or the invalidation used the **wrong key** (a typo, or a key that doesn't match as a prefix). Fix it by invalidating, or updating, the correct keys in the mutation, ideally through a **key factory** so the keys can't drift.

```ts
// ❌ Wrong or missing key
onSuccess: () => queryClient.invalidateQueries({ queryKey: ['todo'] }),     // the list key is ['todos', 'list', ...]

// ✅ Key factory + prefix invalidation (all filtered lists)
onSuccess: () => queryClient.invalidateQueries({ queryKey: todoKeys.lists() }),

// ✅ Or insert directly (no refetch) and then reconcile
onSuccess: (created) => {
  queryClient.setQueryData<Todo[]>(todoKeys.list({}), (old = []) => [created, ...old]);
  queryClient.invalidateQueries({ queryKey: todoKeys.lists() });
},
```

The DevTools show quickly whether the expected key was invalidated.

Interview-ready answer: *"The cache doesn't know about server changes. Either the mutation never invalidated the list, or it invalidated a key that doesn't match the list's key, so the list stays fresh and isn't refetched. I fix it by invalidating the list keys by prefix in onSuccess or onSettled, using a key factory so the keys can't drift, or by inserting the created item with setQueryData and then reconciling."*

**Q (Scenario): The app fires far too many requests. How do you diagnose and reduce them?**
A: Usual causes:
- **`staleTime: 0`**, the default, so every mount, focus and reconnect refetches.
- **Unstable query keys**: objects with new values each render, or a `Date.now()` in the key, which makes every render a new cache entry.
- **A QueryClient recreated on every render.**
- Over-eager polling.
- Infinite queries refetching every loaded page.

Diagnose in the **DevTools** (watch the fetch counts and keys) and the **network tab**.

```tsx
// ❌ A new QueryClient on every render: the cache is wiped and everything refetches
function App() { const qc = new QueryClient(); return <QueryClientProvider client={qc}>…</QueryClientProvider>; }
// ✅
const [queryClient] = useState(() => new QueryClient());

// ❌ A key that changes every render
useQuery({ queryKey: ['events', { from: new Date() }], queryFn });
// ✅ A stable, serialisable key
useQuery({ queryKey: ['events', { from: startOfDay.toISOString() }], queryFn });

// ✅ Sensible freshness
new QueryClient({ defaultOptions: { queries: { staleTime: 60_000, refetchOnWindowFocus: true } } });
```

Interview-ready answer: *"I open the DevTools to see which keys refetch and why. The usual culprits are the default staleTime of zero, so every mount and focus refetches, query keys that change every render because of new dates or objects, a QueryClient recreated inside a component, and aggressive polling. The fixes are an app-wide staleTime, stable serialisable keys, a single client, and polling only where it's needed, with adaptive intervals."*

**Q (Scenario): A detail page shows a spinner every time you open it, even though the list already has the item. How do you make it instant?**
A: **Seed the detail query from the list cache** with `initialData` (plus `initialDataUpdatedAt`, so staleness is computed correctly), or **prefetch** the detail on hover. The detail renders immediately from cached list data, then refetches in the background for the full record.

```tsx
function useTodo(id: number) {
  const queryClient = useQueryClient();
  return useQuery({
    ...todoQueries.detail(id),
    initialData: () =>
      queryClient.getQueryData<Todo[]>(todoKeys.list({}))?.find((t) => t.id === id),
    initialDataUpdatedAt: () => queryClient.getQueryState(todoKeys.list({}))?.dataUpdatedAt,
  });
}

// Or prefetch on hover in the list
<Link to={`/todos/${t.id}`} onMouseEnter={() => queryClient.prefetchQuery(todoQueries.detail(t.id))}>{t.title}</Link>
```

If the list item is only a **partial** version of the detail, use `placeholderData` instead, so it isn't cached as the full entity.

Interview-ready answer: *"The data is already in the list cache, so I seed the detail query with initialData pulled from the list, with initialDataUpdatedAt so it still refetches when stale, or I prefetch the detail on hover. The page renders instantly and refreshes in the background. If the list only has a summary of the item, I use placeholderData instead, so a partial object never gets cached as the full detail."*

**Q (Scenario): Typing in a search box triggers a request per keystroke, and results sometimes flash. How do you fix it?**
A: **Debounce** the input value that goes into the query key, so only settled terms create queries. Use **`placeholderData: keepPreviousData`** so old results stay visible instead of flashing a spinner, and `enabled` to skip empty terms. Race conditions are handled for you: each term has its **own key**, so a slow old response can't overwrite a newer term's results, and passing `signal` aborts superseded requests.

```tsx
function Search() {
  const [term, setTerm] = useState('');
  const debounced = useDebounce(term.trim(), 300);

  const { data, isFetching, isPlaceholderData } = useQuery({
    queryKey: ['search', debounced],
    queryFn: ({ signal }) => api.search(debounced, { signal }),
    enabled: debounced.length >= 2,
    placeholderData: keepPreviousData,
    staleTime: 5 * 60_000,                       // revisiting a term is instant
  });

  return (
    <>
      <input value={term} onChange={(e) => setTerm(e.target.value)} aria-label="Search" />
      {isFetching && <small>Searching…</small>}
      <Results items={data ?? []} dimmed={isPlaceholderData} />
    </>
  );
}
```

Interview-ready answer: *"I debounce the term before it reaches the query key, skip short terms with enabled, and use keepPreviousData so the previous results stay up while the new ones load. Because every term has its own cache entry, out-of-order responses can't overwrite newer results, and forwarding the AbortSignal cancels requests that are no longer needed. A longer staleTime makes repeated searches instant."*

---

## Interview Quick Revision

**Q: What is the one-line takeaway for each TanStack Query topic?**
A:

| Topic | Key concept to remember |
|---|---|
| What it is | Async server-state manager: cache, dedupe, refetch, retry, mutations |
| Server vs client state | Remote cached snapshot vs UI state the browser owns |
| Query keys | Arrays containing every input; hierarchical, so you can invalidate by prefix; use a key factory |
| `queryOptions` | Define key + fn + options once; reuse for hooks, prefetching and cache reads |
| `status` vs `fetchStatus` | Data (pending, error, success) vs network (fetching, paused, idle) |
| `isPending` / `isFetching` / `isLoading` | No data yet / any request in flight / first fetch running |
| `staleTime` vs `gcTime` | When to refetch (default 0) vs when to delete unused data (default 5 min) |
| Refetch triggers | Mount, window focus, reconnect, interval, but only when stale |
| `select` | Derive data per component; re-render only on a change |
| `initialData` vs `placeholderData` | Real, cached data vs temporary, uncached display data |
| Mutations | `mutate` (never throws) vs `mutateAsync` (await + catch); invalidate or `setQueryData` |
| Optimistic updates | `cancelQueries`, snapshot, set, roll back on error, invalidate on settle; or render `variables` |
| Pagination | Page in the key + `keepPreviousData` + prefetch the next page |
| Infinite queries | `initialPageParam` + `getNextPageParam`; all pages refetch in order; `maxPages` |
| Retries | 3× with backoff for queries, 0 for mutations; skip 4xx errors |
| Errors | QueryCache `onError` globally; `throwOnError` + error boundaries |
| SSR | A per-request client, prefetch, `dehydrate` → `HydrationBoundary`; `staleTime` above 0 |
| Testing | A fresh client per test, `retry: false`, MSW, `renderHook` + `waitFor` |
| v5 changes | `gcTime`, `isPending`, `placeholderData`, `throwOnError`, no `useQuery` callbacks, suspense hooks |

**Q: Which mistakes do interviewers expect a senior developer to avoid?**
A:
1. **Copying query data into `useState`, Redux or Zustand**, which creates two sources of truth.
2. **Leaving `staleTime` at 0 everywhere**, causing excessive refetching.
3. **Missing variables in the query key**, so the data doesn't update when inputs change.
4. **Unstable keys or a QueryClient created on every render.**
5. **Not throwing on non-OK `fetch` responses**, so errors look like successes.
6. **Forgetting to invalidate after mutations**, or invalidating the wrong key.
7. **Sharing one QueryClient across SSR requests**, which leaks data between users.
8. **Retrying 4xx errors**, or retrying non-idempotent mutations.

```ts
// The "senior defaults" in one place
export const queryClient = new QueryClient({
  queryCache: new QueryCache({ onError: (e) => reportError(e) }),
  defaultOptions: {
    queries: {
      staleTime: 60_000,
      retry: (n, e) => !(e instanceof ApiError && e.status < 500) && n < 3,
    },
    mutations: { retry: 0 },
  },
});
```

Interview-ready answer: *"Treat TanStack Query as the single owner of server state, design query keys carefully with a factory, set a sensible staleTime, throw on HTTP errors, invalidate precisely after writes, never share a client between SSR requests, and only retry transient errors. Most production bugs I've seen with React Query come down to keys, staleTime or duplicating its data elsewhere."*
