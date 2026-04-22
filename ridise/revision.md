---
# PART 1: FRONTEND (React + Next.js)

## Section 1: Core React Fundamentals (20 Questions + Answers)
---

### 1. What is React?

**Answer:**
React is a JavaScript library for building UI, primarily using a component-based architecture. It uses a virtual DOM to optimize rendering.

---

### 2. What is Virtual DOM?

**Answer:**
A lightweight JS representation of the real DOM. React compares (diffs) it with previous state and updates only changed parts → improves performance.

---

### 3. What are components?

**Answer:**
Reusable, independent UI pieces. Two types:

- Functional (modern)
- Class (legacy)

---

### 4. What is JSX?

**Answer:**
A syntax extension that looks like HTML but compiles to `React.createElement`.

---

### 5. What are props?

**Answer:**
Read-only inputs passed from parent to child components.

---

### 6. What is state?

**Answer:**
Mutable data inside a component that controls UI behavior.

---

### 7. Difference: props vs state

**Answer:**

| Props              | State                    |
| ------------------ | ------------------------ |
| Immutable          | Mutable                  |
| Passed from parent | Managed inside component |

---

### 8. What are hooks?

**Answer:**
Functions that let you use state and lifecycle in functional components.

---

### 9. useState vs useEffect

**Answer:**

- `useState` → manage state
- `useEffect` → side effects (API calls, subscriptions)

---

### 10. What is useEffect dependency array?

**Answer:**
Controls when effect runs:

- `[]` → run once
- `[x]` → run when x changes

---

### 11. What is reconciliation?

**Answer:**
Process React uses to update DOM efficiently using diffing algorithm.

---

### 12. What is key in lists?

**Answer:**
Unique identifier for elements → helps React optimize re-rendering.

---

### 13. Controlled vs uncontrolled components

**Answer:**

- Controlled → React manages state
- Uncontrolled → DOM manages state

---

### 14. What is lifting state up?

**Answer:**
Moving state to common parent to share between components.

---

### 15. What is context API?

**Answer:**
Used to avoid prop drilling → global state sharing.

---

### 16. What is memoization in React?

**Answer:**
Optimizing re-renders using:

- `React.memo`
- `useMemo`
- `useCallback`

---

### 17. What is React.memo?

**Answer:**
Prevents unnecessary re-render if props don’t change.

---

### 18. useCallback vs useMemo

**Answer:**

- useCallback → memoizes function
- useMemo → memoizes value

---

### 19. What is fragment?

**Answer:**
`<> </>` → group elements without extra DOM node.

---

### 20. What is StrictMode?

**Answer:**
Development tool to highlight potential issues.

---

---

## Section 2: Advanced React (Performance, Patterns)

---

### 1. What causes re-render?

**Answer:**

- State change
- Props change
- Parent re-render

---

### 2. How to prevent unnecessary re-renders?

**Answer:**

- React.memo
- useMemo
- useCallback

---

### 3. What is lazy loading in React?

**Answer:**

```js
const Comp = React.lazy(() => import('./Comp'));
```

---

### 4. What is Suspense?

**Answer:**
Fallback UI during lazy loading.

---

### 5. What is error boundary?

**Answer:**
Catches JS errors in component tree.

---

### 6. What is HOC?

**Answer:**
Higher Order Component → function that returns component.

---

### 7. What is render props?

**Answer:**
Sharing logic via function as child.

---

### 8. What is prop drilling problem?

**Answer:**
Passing props through many layers unnecessarily.

---

### 9. How to solve prop drilling?

**Answer:**

- Context API
- Redux/Zustand

---

### 10. What is useRef?

**Answer:**
Access DOM or persist value without re-render.

---

### 11. useRef vs useState

**Answer:**

- useState → triggers render
- useRef → does not

---

### 12. What is batching?

**Answer:**
React groups multiple state updates for performance.

---

### 13. What is concurrent rendering?

**Answer:**
React can pause & resume rendering → smoother UI.

---

### 14. What is hydration?

**Answer:**
Attaching JS to server-rendered HTML.

---

### 15. What is reconciliation key issue?

**Answer:**
Using index as key can break UI.

---

### 16. What is forwardRef?

**Answer:**
Pass ref to child component.

---

### 17. What are portals?

**Answer:**
Render outside parent DOM hierarchy.

---

### 18. What is state colocation?

**Answer:**
Keeping state close to where it's used.

---

### 19. What is debouncing in React?

**Answer:**
Delay execution (e.g., search input).

```jsx
import { useEffect, useState } from 'react';

function SearchBox() {
  const [query, setQuery] = useState('');
  const [debouncedQuery, setDebouncedQuery] = useState('');

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedQuery(query);
    }, 500); // wait 500ms after typing stops

    return () => clearTimeout(timer); // reset timer on each keystroke
  }, [query]);

  useEffect(() => {
    if (!debouncedQuery) return;
    console.log('API call for:', debouncedQuery);
  }, [debouncedQuery]);

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder='Type to search...'
    />
  );
}
```

---

### 20. What is throttling?

**Answer:**
Limit execution frequency.

```jsx
import { useRef } from 'react';

function ThrottledScrollLogger() {
  const lastRun = useRef(0);

  const handleScroll = () => {
    const now = Date.now();
    if (now - lastRun.current < 1000) return; // run at most once/second

    lastRun.current = now;
    console.log('Scroll event handled at:', new Date(now).toLocaleTimeString());
  };

  return (
    <div onScroll={handleScroll} style={{ height: 120, overflowY: 'auto' }}>
      <div style={{ height: 500 }}>Scroll me</div>
    </div>
  );
}
```

---

### 21. What are the types of hooks in React? Explain each with examples.

**Answer:**

1. **State Hook (`useState`)**
   Manages local component state.

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}
```

2. **Effect Hook (`useEffect`)**
   Runs side effects like API calls, subscriptions, or timers.

```jsx
import { useEffect, useState } from 'react';

function UserData() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetch('/api/users')
      .then((res) => res.json())
      .then(setUsers);
  }, []);

  return <p>Total users: {users.length}</p>;
}
```

3. **Context Hook (`useContext`)**
   Consumes shared data from React Context without prop drilling.

```jsx
import { createContext, useContext } from 'react';

const ThemeContext = createContext('light');

function ThemeLabel() {
  const theme = useContext(ThemeContext);
  return <p>Current theme: {theme}</p>;
}
```

4. **Ref Hook (`useRef`)**
   Stores mutable values without re-render and accesses DOM nodes.

```jsx
import { useEffect, useRef } from 'react';

function AutoFocusInput() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} placeholder='Auto focused' />;
}
```

5. **Performance Hooks (`useMemo`, `useCallback`)**
   Optimize expensive calculations and stable function references.

```jsx
import { useCallback, useMemo, useState } from 'react';

function FilteredList({ items }) {
  const [query, setQuery] = useState('');

  const filtered = useMemo(
    () =>
      items.filter((item) => item.toLowerCase().includes(query.toLowerCase())),
    [items, query]
  );

  const clear = useCallback(() => setQuery(''), []);

  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <button onClick={clear}>Clear</button>
      <p>Results: {filtered.length}</p>
    </>
  );
}
```

6. **Reducer Hook (`useReducer`)**
   Manages complex state logic with actions.

```jsx
import { useReducer } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      return state;
  }
}

function CounterWithReducer() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <span>{state.count}</span>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
    </>
  );
}
```

---

### 22. What are the types of state in React?

**Answer:**

1. **Local state**
   State that belongs to one component (usually `useState` / `useReducer`).

```jsx
import { useState } from 'react';

function Toggle() {
  const [isOpen, setIsOpen] = useState(false);
  return (
    <button onClick={() => setIsOpen(!isOpen)}>
      {isOpen ? 'Open' : 'Closed'}
    </button>
  );
}
```

2. **Global state**
   Shared state used across many components (auth, cart, theme, etc.).

**Zustand example:**

```jsx
import { create } from 'zustand';

const useCounterStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));

function Counter() {
  const { count, increment, decrement } = useCounterStore();
  return (
    <>
      <button onClick={decrement}>-</button>
      <span>{count}</span>
      <button onClick={increment}>+</button>
    </>
  );
}
```

**Redux Toolkit example:**

```jsx
import { configureStore, createSlice } from '@reduxjs/toolkit';
import { Provider, useDispatch, useSelector } from 'react-redux';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
  },
});

const store = configureStore({ reducer: { counter: counterSlice.reducer } });
const { increment, decrement } = counterSlice.actions;

function Counter() {
  const value = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return (
    <>
      <button onClick={() => dispatch(decrement())}>-</button>
      <span>{value}</span>
      <button onClick={() => dispatch(increment())}>+</button>
    </>
  );
}

export default function App() {
  return (
    <Provider store={store}>
      <Counter />
    </Provider>
  );
}
```

3. **Derived state**
   Value calculated from existing state/props; avoid storing if you can compute it.

```jsx
const completedCount = todos.filter((todo) => todo.done).length;
```

4. **Server state**
   Data coming from backend/API (often managed with React Query/SWR).

```jsx
useEffect(() => {
  fetch('/api/products')
    .then((res) => res.json())
    .then(setProducts);
}, []);
```

5. **Form state**
   Input values, validation errors, touched state in forms.

```jsx
const [form, setForm] = useState({ email: '', password: '' });
```

---

### 23. How do you handle form data submission in React?

**Answer:**
Use controlled inputs (`useState`) and submit with `onSubmit`.

```jsx
import { useState } from 'react';

function LoginForm() {
  const [form, setForm] = useState({ email: '', password: '' });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setForm((prev) => ({ ...prev, [name]: value }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    console.log('Submitting:', form);
    // await fetch("/api/login", { method: "POST", body: JSON.stringify(form) });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name='email' value={form.email} onChange={handleChange} />
      <input
        name='password'
        type='password'
        value={form.password}
        onChange={handleChange}
      />
      <button type='submit'>Login</button>
    </form>
  );
}
```

---

### 24. What is react-hook-form and why use it?

**Answer:**
`react-hook-form` manages form state efficiently with less re-rendering and simple validation.

```jsx
import { useForm } from 'react-hook-form';

function SignupForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm();

  const onSubmit = (data) => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name', { required: 'Name is required' })} />
      {errors.name && <p>{errors.name.message}</p>}

      <input {...register('email', { required: 'Email is required' })} />
      {errors.email && <p>{errors.email.message}</p>}

      <button type='submit'>Submit</button>
    </form>
  );
}
```

---

### 25. How do you validate forms using Zod in React?

**Answer:**
Define a Zod schema and use it with `react-hook-form` via `zodResolver`.

```jsx
import { useForm } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(6, 'Password must be at least 6 characters'),
});

function ZodForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm({
    resolver: zodResolver(schema),
  });

  const onSubmit = (data) => console.log('Valid data:', data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} placeholder='Email' />
      {errors.email && <p>{errors.email.message}</p>}

      <input {...register('password')} type='password' placeholder='Password' />
      {errors.password && <p>{errors.password.message}</p>}

      <button type='submit'>Create Account</button>
    </form>
  );
}
```

---

### 26. What is TanStack Query and why is it used?

**Answer:**
TanStack Query (React Query) is used for server-state management: fetching, caching, refetching, loading/error states, and mutations.

```jsx
import {
  QueryClient,
  QueryClientProvider,
  useQuery,
} from '@tanstack/react-query';

const queryClient = new QueryClient();

function Users() {
  const { data, isLoading, isError } = useQuery({
    queryKey: ['users'],
    queryFn: async () => {
      const res = await fetch('/api/users');
      if (!res.ok) throw new Error('Failed to fetch users');
      return res.json();
    },
  });

  if (isLoading) return <p>Loading...</p>;
  if (isError) return <p>Error loading users</p>;

  return <p>Total users: {data.length}</p>;
}

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Users />
    </QueryClientProvider>
  );
}
```

---

### 27. What is Axios and why use it in React?

**Answer:**
Axios is a promise-based HTTP client used to call APIs. It provides features like interceptors, request/response transformation, timeout, and better error handling.

```jsx
import axios from 'axios';
import { useEffect, useState } from 'react';

function UsersList() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    axios.get('/api/users').then((res) => setUsers(res.data));
  }, []);

  return <p>Total users: {users.length}</p>;
}
```

---

### 28. How do you make GET and POST requests using Axios?

**Answer:**
Use `axios.get()` for reading data and `axios.post()` for sending data.

```jsx
import axios from 'axios';

async function fetchProducts() {
  const res = await axios.get('/api/products');
  return res.data;
}

async function createProduct(payload) {
  const res = await axios.post('/api/products', payload);
  return res.data;
}
```

---

### 29. What are Axios interceptors?

**Answer:**
Interceptors let you run logic before requests or after responses (e.g., attach auth token, refresh token, log errors).

```jsx
import axios from 'axios';

const api = axios.create({ baseURL: '/api' });

api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      console.log('Unauthorized - redirect to login');
    }
    return Promise.reject(error);
  }
);
```

---

### 30. How do you handle Axios errors and cancel requests?

**Answer:**
Use `try/catch` for errors and `AbortController` to cancel in-flight requests (useful during unmount or fast search typing).

```jsx
import axios from 'axios';
import { useEffect, useState } from 'react';

function SearchUsers({ query }) {
  const [data, setData] = useState([]);

  useEffect(() => {
    const controller = new AbortController();

    async function load() {
      try {
        const res = await axios.get('/api/users/search', {
          params: { q: query },
          signal: controller.signal,
        });
        setData(res.data);
      } catch (error) {
        if (axios.isCancel(error)) return;
        console.error('Request failed:', error.message);
      }
    }

    if (query) load();
    return () => controller.abort();
  }, [query]);

  return <p>Found: {data.length}</p>;
}
```

---

## Section 3: Next.js (Important Questions)

---

### 1. What is Next.js?

**Answer:**
React framework with SSR, SSG, routing, API routes.

---

### 2. SSR vs SSG

**Answer:**

- SSR → runtime render
- SSG → build-time render

---

### 3. What is ISR?

**Answer:**
Incremental Static Regeneration → update static pages.

---

### 4. What is App Router?

**Answer:**
New routing system in Next.js 13+

---

### 5. What are server components?

**Answer:**
Rendered on server → smaller bundle.

---

### 6. What are client components?

**Answer:**
Interactive → run in browser.

---

### 7. use client directive?

**Answer:**
Marks component as client-side.

---

### 8. What is middleware?

**Answer:**
Runs before request (auth, redirects).

---

### 9. What is getServerSideProps?

**Answer:**
Fetch data on every request.

---

### 10. What is getStaticProps?

**Answer:**
Fetch data at build time.

---

### 11. What is dynamic routing?

**Answer:**
`/posts/[id]`

---

### 12. What is API routes?

**Answer:**
Backend endpoints inside Next.js.

---

### 13. What is image optimization?

**Answer:**
Next.js `<Image />` component auto optimizes.

---

### 14. What is code splitting?

**Answer:**
Load only needed JS.

---

### 15. What is edge runtime?

**Answer:**
Run code closer to user → faster.

---

### 16. What is revalidation?

**Answer:**
Refreshing static content.

---

### 17. What is layout in Next.js?

**Answer:**
Reusable UI wrapper.

---

### 18. What is metadata API?

**Answer:**
SEO management.

---

### 19. What is routing difference vs React?

**Answer:**
Next.js → file-based routing.

---

### 20. When to use Next.js?

**Answer:**
SEO, performance, SSR needs.

---

Good choice — this is **high-impact for your profile** (especially with your backend + DevOps experience).
I’ll structure this cleanly into **3 parts**:

1. Node.js Core
2. Express.js
3. NestJS

Each → **20 most asked interview questions with solid answers**

---

# 🚀 PART 2: BACKEND

---

# 🟢 Section 1: Node.js Core (20 Questions + Answers)

---

### 1. What is Node.js?

**Answer:**
Node.js is a runtime built on Chrome’s V8 engine that allows running JavaScript on the server.

---

### 2. Why is Node.js single-threaded?

**Answer:**
To handle high concurrency using **event loop + async I/O**, avoiding thread overhead.

---

### 3. What is Event Loop?

**Answer:**
Core mechanism that handles async operations using callback queue + call stack.

---

### 4. Explain phases of Event Loop

**Answer:**

- Timers
- Pending callbacks
- Idle/prepare
- Poll
- Check
- Close callbacks

---

### 5. What is non-blocking I/O?

**Answer:**
Operations that don’t block execution (e.g., file read, DB calls).

---

### 6. setTimeout vs setImmediate vs process.nextTick

**Answer:**

- `process.nextTick` → highest priority
- `setImmediate` → after I/O
- `setTimeout` → scheduled

---

### 7. What is callback hell?

**Answer:**
Nested callbacks → hard to read & maintain.

---

### 8. How to avoid callback hell?

**Answer:**

- Promises
- Async/await

---

### 9. What are streams?

**Answer:**
Efficient data handling in chunks (e.g., file upload).

---

### 10. Types of streams

**Answer:**

- Readable
- Writable
- Duplex
- Transform

---

### 11. What is Buffer?

**Answer:**
Binary data handling in Node.js.

---

### 12. What is clustering?

**Answer:**
Using multiple CPU cores to scale Node apps.

---

### 13. Worker Threads vs Cluster

**Answer:**

- Cluster → multiple processes
- Worker Threads → parallel execution in same process

---

### 14. What is middleware in Node?

**Answer:**
Functions executed during request lifecycle.

---

### 15. What is REPL?

**Answer:**
Interactive Node shell.

---

### 16. What is process object?

**Answer:**
Global object for environment & runtime info.

---

### 17. What is module system?

**Answer:**

- CommonJS (`require`)
- ES Modules (`import`)

---

### 18. What is memory leak in Node?

**Answer:**
Unused memory not released → crashes.

---

### 19. How to handle errors globally?

**Answer:**

```js
process.on('uncaughtException');
process.on('unhandledRejection');
```

---

### 20. What is libuv?

**Answer:**
Library that handles async I/O and event loop internally.

---

---

# 🟡 Section 2: Express.js (20 Questions + Answers)

---

### 1. What is Express.js?

**Answer:**
Minimal web framework for Node.js to build APIs.

---

### 2. What is middleware in Express?

**Answer:**
Functions that process request/response.

---

### 3. Types of middleware

**Answer:**

- Application
- Router
- Error-handling

---

### 4. What is routing?

**Answer:**
Mapping URLs to handlers.

---

### 5. What is next()?

**Answer:**
Pass control to next middleware.

---

### 6. How to handle errors in Express?

**Answer:**

```js
app.use((err, req, res, next) => {});
```

---

### 7. What is body-parser?

**Answer:**
Parses request body (JSON, urlencoded).

---

### 8. What is CORS?

**Answer:**
Cross-origin resource sharing.

---

### 9. What is rate limiting?

**Answer:**
Restrict number of requests.

---

### 10. What is Helmet?

**Answer:**
Security middleware.

---

### 11. What is JWT authentication?

**Answer:**
Token-based authentication.

---

### 12. Session vs JWT

**Answer:**

- Session → server stores
- JWT → client stores

---

### 13. What is REST API?

**Answer:**
Standardized HTTP-based API.

---

### 14. What is idempotent API?

**Answer:**
Multiple same requests → same result (PUT, DELETE).

---

### 15. What is validation?

**Answer:**
Validating request data (Joi, Zod).

---

### 16. What is pagination?

**Answer:**
Breaking large data into pages.

---

### 17. What is compression?

**Answer:**
Reducing response size.

---

### 18. What is logging?

**Answer:**
Tracking requests (Morgan, Winston).

---

### 19. What is API versioning?

**Answer:**
v1, v2 endpoints for backward compatibility.

---

### 20. What are best practices?

**Answer:**

- MVC structure
- Error handling
- Validation
- Logging

---

---

# 🔵 Section 3: NestJS (20 Questions + Answers)

---

### 1. What is NestJS?

**Answer:**
A scalable Node.js framework built with TypeScript, inspired by Angular.

---

### 2. Why use NestJS?

**Answer:**

- Modular architecture
- Dependency Injection
- Clean structure

---

### 3. What is module in NestJS?

**Answer:**
Organizes application into features.

---

### 4. What is controller?

**Answer:**
Handles incoming requests.

---

### 5. What is service?

**Answer:**
Contains business logic.

---

### 6. What is dependency injection?

**Answer:**
Automatically inject dependencies.

---

### 7. What are providers?

**Answer:**
Classes managed by DI system.

---

### 8. What are decorators?

**Answer:**

```ts
@Controller()
@Get()
@Injectable()
```

---

### 9. What is middleware in NestJS?

**Answer:**
Similar to Express middleware.

---

### 10. What are guards?

**Answer:**
Used for authentication/authorization.

---

### 11. What are interceptors?

**Answer:**
Modify request/response.

---

### 12. What are pipes?

**Answer:**
Validation & transformation.

---

### 13. What is exception filter?

**Answer:**
Handles errors globally.

---

### 14. What is DTO?

**Answer:**
Data Transfer Object → defines structure.

---

### 15. What is validation in NestJS?

**Answer:**
Uses class-validator:

```ts
@IsEmail()
```

---

### 16. What is module communication?

**Answer:**
Via exports/imports.

---

### 17. What is async provider?

**Answer:**
Used for DB connections.

---

### 18. What is microservices support?

**Answer:**
NestJS supports Kafka, RabbitMQ, etc.

---

### 19. What is GraphQL support?

**Answer:**
Built-in GraphQL module.

---

### 20. Why NestJS over Express?

**Answer:**

- Structure
- Scalability
- Maintainability

---

Perfect — this is where strong backend engineers really stand out. I’ll split this into:

1. PostgreSQL (Relational DB deep dive)
2. MongoDB (NoSQL deep dive)

Each → **20 high-impact interview questions + solid answers**

---

# 🟢 Section 1: PostgreSQL (20 Questions + Answers)

---

### 1. What is PostgreSQL?

**Answer:**
An open-source relational database that uses SQL and supports advanced features like ACID transactions, JSON, indexing, and extensions.

---

### 2. What is ACID?

**Answer:**

- Atomicity → all or nothing
- Consistency → valid state
- Isolation → concurrent safety
- Durability → permanent storage

---

### 3. What is normalization?

**Answer:**
Process of structuring tables to reduce redundancy (1NF, 2NF, 3NF).

---

### 4. What is denormalization?

**Answer:**
Adding redundancy to improve read performance.

---

### 5. What are indexes?

**Answer:**
Data structures that improve query performance.

---

### 6. Types of indexes in PostgreSQL

**Answer:**

- B-Tree (default)
- Hash
- GIN (for JSON/text search)
- GiST

---

### 7. What is a primary key?

**Answer:**
Unique identifier for each row.

---

### 8. What is foreign key?

**Answer:**
Maintains relationship between tables.

---

### 9. What is JOIN?

**Answer:**
Combining rows from multiple tables.

---

### 10. Types of JOIN

**Answer:**

- INNER
- LEFT
- RIGHT
- FULL

---

### 11. What is transaction?

**Answer:**
A sequence of operations executed as a single unit.

---

### 12. What is isolation level?

**Answer:**

- Read Uncommitted
- Read Committed
- Repeatable Read
- Serializable

---

### 13. What is deadlock?

**Answer:**
Two transactions waiting on each other.

---

### 14. How to optimize queries?

**Answer:**

- Indexing
- Avoid SELECT \*
- Use EXPLAIN
- Proper joins

---

### 15. What is EXPLAIN?

**Answer:**
Shows query execution plan.

---

### 16. What is VACUUM?

**Answer:**
Cleans up dead rows.

---

### 17. What is partitioning?

**Answer:**
Splitting large tables into smaller pieces.

---

### 18. What is JSONB?

**Answer:**
Binary JSON format → faster than JSON.

---

### 19. What is materialized view?

**Answer:**
Precomputed query result stored for fast access.

---

### 20. When to use PostgreSQL?

**Answer:**

- Strong consistency needed
- Complex joins
- Financial systems

---

---

# 🔵 Section 2: MongoDB (20 Questions + Answers)

---

### 1. What is MongoDB?

**Answer:**
A NoSQL database that stores data as JSON-like documents.

---

### 2. Difference: SQL vs NoSQL

**Answer:**

| SQL        | NoSQL     |
| ---------- | --------- |
| Structured | Flexible  |
| Tables     | Documents |

---

### 3. What is a document?

**Answer:**
JSON-like object stored in MongoDB.

---

### 4. What is collection?

**Answer:**
Group of documents (like table).

---

### 5. What is \_id?

**Answer:**
Unique identifier (ObjectId).

---

### 6. What is indexing?

**Answer:**
Improves query performance.

---

### 7. Types of indexes

**Answer:**

- Single field
- Compound
- Text
- Geospatial

---

### 8. What is aggregation?

**Answer:**
Pipeline for data transformation.

---

### 9. Example aggregation

```js
db.users.aggregate([
  { $match: { age: { $gt: 25 } } },
  { $group: { _id: '$city', count: { $sum: 1 } } },
]);
```

---

### 10. What is sharding?

**Answer:**
Horizontal scaling across multiple servers.

---

### 11. What is replication?

**Answer:**
Copying data for availability.

---

### 12. Primary vs Secondary

**Answer:**

- Primary → writes
- Secondary → reads

---

### 13. What is schema design?

**Answer:**
Structure of documents.

---

### 14. Embedding vs referencing

**Answer:**

- Embedding → faster reads
- Referencing → avoids duplication

---

### 15. What is CAP theorem?

**Answer:**

- Consistency
- Availability
- Partition tolerance

---

### 16. What is eventual consistency?

**Answer:**
Data becomes consistent over time.

---

### 17. What is TTL index?

**Answer:**
Auto delete documents after time.

---

### 18. What is $lookup?

**Answer:**
Join-like operation.

---

### 19. How to optimize MongoDB?

**Answer:**

- Proper indexes
- Avoid large documents
- Use projections

---

### 20. When to use MongoDB?

**Answer:**

- Flexible schema
- High scalability
- Rapid development

---

# 🧠 Advanced Interview Questions (VERY IMPORTANT)

These are commonly asked at **your experience level**:

---

### 🔥 1. PostgreSQL vs MongoDB — when to choose?

**Answer:**

- PostgreSQL → transactions, relations
- MongoDB → scalability, flexibility

---

### 🔥 2. How to design schema for high-scale system?

**Answer:**

- Normalize critical data
- Denormalize for reads
- Use indexes wisely

---

### 🔥 3. How to handle millions of records?

**Answer:**

- Partitioning (Postgres)
- Sharding (MongoDB)
- Caching (Redis)

---

### 🔥 4. How to avoid N+1 query problem?

**Answer:**

- Use joins / aggregation
- Batch queries

---

### 🔥 5. How to handle transactions in MongoDB?

**Answer:**
Use multi-document transactions (with replica set).

---

# 🚀 What Next?

Now the remaining high-value sections:

1. 🟣 **System Design (MOST IMPORTANT for senior roles)**
2. 🔵 JavaScript + TypeScript internals (deep questions asked in interviews)
3. 🧠 AI + LLM integration (huge bonus for your profile)
4. ⚙️ REST APIs + Testing + CI/CD

---

## 👉 My recommendation for YOU:

Go next with **System Design** — that’s what will differentiate you at **10–15 LPA+ roles**.

Just say:
👉 “System Design” and I’ll go deep (with real-world architectures like YouTube, WhatsApp, etc.)

Good — this is the section where most senior candidates either stand out or collapse. I’ll give you **real interview-grade system design Q&A (20)** — not textbook fluff, but what interviewers _actually probe_.

---

# 🟣 SYSTEM DESIGN (Senior-Level Deep Dive)

---

## 🔥 1. How do you design a scalable system?

**Answer:**
Start structured:

1. Requirements (functional + non-functional)
2. High-level design (components)
3. Database design
4. Scaling strategy
5. Bottlenecks & trade-offs

---

## 🔥 2. What are functional vs non-functional requirements?

**Answer:**

- Functional → what system does (upload video)
- Non-functional → how well (latency, scalability, availability)

---

## 🔥 3. What is horizontal vs vertical scaling?

**Answer:**

- Vertical → increase machine size
- Horizontal → add more machines (preferred)

---

## 🔥 4. What is load balancing?

**Answer:**
Distributes traffic across servers.

Types:

- Round robin
- Least connections
- IP hash

---

## 🔥 5. What is caching? Where to use it?

**Answer:**
Store frequently accessed data in fast storage.

Use cases:

- API responses
- DB queries
- Sessions

Tools: Redis

---

## 🔥 6. What is CDN?

**Answer:**
Content Delivery Network → serves static content closer to users.

Example: Cloudflare

---

## 🔥 7. What is database sharding?

**Answer:**
Splitting data across multiple DBs.

Example:

- User-based sharding
- Region-based

---

## 🔥 8. What is replication?

**Answer:**
Copying data across nodes for availability.

---

## 🔥 9. What is CAP theorem?

**Answer:**

- Consistency
- Availability
- Partition tolerance

You can only fully guarantee 2 of 3.

---

## 🔥 10. How do you design URL shortener (like Bitly)?

**Answer:**
Components:

- API service
- DB (key-value)
- Hash generator
- Cache

Flow:
User → API → generate short ID → store → redirect

---

## 🔥 11. How do you design a chat system (like WhatsApp)?

**Answer:**

- WebSocket servers
- Message queue
- DB (messages)
- Presence service

Key:

- Real-time delivery
- Offline storage

---

## 🔥 12. How do you design video streaming (like YouTube)?

**Answer:**

- Upload service
- Encoding pipeline (FFmpeg)
- Storage (S3)
- CDN delivery

---

## 🔥 13. What is message queue? Why use it?

**Answer:**
Decouples services → async processing.

Examples:

- Kafka
- RabbitMQ

---

## 🔥 14. What is eventual consistency?

**Answer:**
Data becomes consistent over time.

Used in distributed systems.

---

## 🔥 15. What is rate limiting?

**Answer:**
Prevent abuse.

Techniques:

- Token bucket
- Sliding window

---

## 🔥 16. How do you design a high-traffic API?

**Answer:**

- Load balancer
- Stateless services
- Caching
- DB optimization
- Rate limiting

---

## 🔥 17. What is microservices architecture?

**Answer:**
Small independent services communicating over APIs.

---

## 🔥 18. Monolith vs Microservices

**Answer:**

- Monolith → simple, hard to scale
- Microservices → scalable, complex

---

## 🔥 19. How do you handle failures?

**Answer:**

- Retry mechanism
- Circuit breaker
- Fallback
- Monitoring

---

## 🔥 20. What is observability?

**Answer:**

- Logs
- Metrics
- Traces

Tools:

- Prometheus
- Grafana

---

# 🧠 REAL INTERVIEW SCENARIOS (VERY IMPORTANT)

These are the exact follow-ups you'll get:

---

## 💥 1. Design Instagram Feed

**Key points:**

- Fan-out vs fan-in
- Cache feeds
- Precompute timelines

---

## 💥 2. Design Uber

**Key points:**

- Real-time location tracking
- Matching algorithm
- Geo indexing

---

## 💥 3. Design Payment System

**Key points:**

- Idempotency
- Strong consistency
- Retry + logging

---

## 💥 4. Design Notification System

**Key points:**

- Queue-based
- Multi-channel (email, push)
- Retry + batching

---

## 💥 5. Design File Upload System

**Key points:**

- Chunk upload
- Resume support
- Background processing

---

# ⚡ Pro Tips (For YOU specifically)

Given your experience, interviewers WILL ask:

- “Explain your video pipeline architecture”
- “How did you reduce infra cost by 90%?”
- “Design a scalable live streaming system”
- “How would you scale to 10M users?”

👉 You should always talk in this order:

1. Requirements
2. High-level design
3. Deep dive (DB + scaling)
4. Trade-offs

---

# 🚀 What Next?

Remaining sections:

1. 🔵 JavaScript + TypeScript internals (VERY tricky questions)
2. 🧠 AI + LLM systems (huge advantage for you)
3. ⚙️ REST APIs + Testing + CI/CD (interview + practical)

---

## 👉 My recommendation:

Next go with:

👉 **JavaScript + TypeScript internals**

This is where even senior devs get rejected.

Just say **“JS internals”** and I’ll go deep (closures, event loop traps, tricky questions).

Perfect — this is where even experienced devs get caught. I’ll go deep but keep it **interview-focused, not academic**.

---

# 🔵 JavaScript + TypeScript Internals (Deep Dive)

I’ll split into:

1. JavaScript Internals (Core Engine Concepts)
2. Advanced JS (Tricky + Real Interview Traps)
3. TypeScript Internals

Each → **20 high-value Q&A**

---

# 🟢 Section 1: JavaScript Internals (Core) — 20 Q&A

---

### 1. What is Execution Context?

**Answer:**
Environment where JS code runs.

Types:

- Global
- Function
- Eval

Contains:

- Variable Environment
- Scope Chain
- this binding

---

### 2. What is Call Stack?

**Answer:**
Tracks function execution order (LIFO).

---

### 3. What is Hoisting?

**Answer:**
Variables/functions are moved to top during compilation.

---

### 4. var vs let vs const (deep)

**Answer:**

- var → function scoped, hoisted with `undefined`
- let/const → block scoped, in TDZ

---

### 5. What is Temporal Dead Zone (TDZ)?

**Answer:**
Time between declaration and initialization where variable can't be accessed.

---

### 6. What is Scope Chain?

**Answer:**
JS looks for variables from inner → outer scopes.

---

### 7. What is Closure?

**Answer:**
Function + its lexical environment.

```js
function outer() {
  let x = 10;
  return function inner() {
    return x;
  };
}
```

---

### 8. What is Lexical Scope?

**Answer:**
Scope determined at write-time, not runtime.

---

### 9. What is this keyword?

**Answer:**
Depends on call site:

- global → window/global
- method → object
- arrow → inherits

---

### 10. Arrow vs normal function

**Answer:**

- No `this` binding
- No `arguments`
- Cannot be constructor

---

### 11. What is Event Loop?

**Answer:**
Handles async execution using queues.

---

### 12. Microtask vs Macrotask

**Answer:**

- Micro → Promise, process.nextTick
- Macro → setTimeout

---

### 13. Promise vs async/await

**Answer:**

- async/await → syntactic sugar over promises

---

### 14. What is Promise chaining?

**Answer:**
Sequential async operations.

---

### 15. What is prototype?

**Answer:**
Mechanism for inheritance.

---

### 16. **proto** vs prototype

**Answer:**

- prototype → function property
- **proto** → object reference

---

### 17. What is garbage collection?

**Answer:**
Automatic memory cleanup.

---

### 18. What causes memory leaks?

**Answer:**

- Closures holding references
- Unremoved listeners

---

### 19. What is deep copy vs shallow copy?

**Answer:**

- Shallow → reference copy
- Deep → full clone

---

### 20. What is debouncing vs throttling?

**Answer:**

- Debounce → delay execution
- Throttle → limit frequency

---

---

# 🟡 Section 2: Advanced JavaScript (Tricky Interview Questions)

---

### 1. Output?

```js
console.log(a);
var a = 5;
```

**Answer:** `undefined` (hoisted)

---

### 2. Output?

```js
console.log(a);
let a = 5;
```

**Answer:** ReferenceError (TDZ)

---

### 3. Output?

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
```

**Answer:** 3 3 3

---

### 4. Fix above issue?

**Answer:**
Use `let` or closure.

---

### 5. Output?

```js
Promise.resolve().then(() => console.log(1));
setTimeout(() => console.log(2));
console.log(3);
```

**Answer:** 3 → 1 → 2

---

### 6. What is currying?

**Answer:**
Transform function into multiple arguments.

---

### 7. What is function composition?

**Answer:**
Combining functions.

---

### 8. What is memoization?

**Answer:**
Caching function results.

---

### 9. What is event delegation?

**Answer:**
Handle events at parent level.

---

### 10. What is Object.freeze?

**Answer:**
Makes object immutable.

---

### 11. What is Object.seal?

**Answer:**
No new properties, but can modify existing.

---

### 12. What is bind vs call vs apply?

**Answer:**

- call → immediate
- apply → array args
- bind → returns function

---

### 13. What is async error handling?

**Answer:**

```js
try {
  await fn();
} catch (e) {}
```

---

### 14. What is race condition?

**Answer:**
Multiple async operations conflict.

---

### 15. What is module pattern?

**Answer:**
Encapsulation using closures.

---

### 16. What is IIFE?

**Answer:**

```js
(function () {})();
```

---

### 17. What is WeakMap?

**Answer:**
Keys are objects, garbage collectible.

---

### 18. What is Symbol?

**Answer:**
Unique identifier.

---

### 19. What is optional chaining?

**Answer:**
`obj?.a?.b`

---

### 20. What is nullish coalescing?

**Answer:**
`??` → null/undefined fallback

---

---

# 🔵 Section 3: TypeScript Internals (20 Q&A)

---

### 1. What is TypeScript?

**Answer:**
Superset of JavaScript with static typing.

---

### 2. Why use TypeScript?

**Answer:**

- Type safety
- Better tooling
- Scalability

---

### 3. What is type inference?

**Answer:**
TS automatically detects types.

---

### 4. What is interface?

**Answer:**
Defines structure of object.

---

### 5. Interface vs type

**Answer:**

- Interface → extendable
- Type → more flexible

---

### 6. What is union type?

**Answer:**

```ts
string | number;
```

---

### 7. What is intersection type?

**Answer:**

```ts
A & B;
```

---

### 8. What is generics?

**Answer:**
Reusable type-safe components.

---

### 9. Example generics

```ts
function identity<T>(arg: T): T {
  return arg;
}
```

---

### 10. What is enum?

**Answer:**
Named constants.

---

### 11. What is unknown vs any?

**Answer:**

- any → no type checking
- unknown → safer

---

### 12. What is never type?

**Answer:**
Function that never returns.

---

### 13. What is type narrowing?

**Answer:**
Refining types at runtime.

---

### 14. What is keyof?

**Answer:**

```ts
keyof T
```

---

### 15. What is mapped type?

**Answer:**

```ts
{ [K in keyof T]: T[K] }
```

---

### 16. What is utility types?

**Answer:**

- Partial
- Pick
- Omit

---

### 17. What is decorator?

**Answer:**
Metadata for classes/functions.

---

### 18. What is tsconfig?

**Answer:**
TypeScript configuration file.

---

### 19. What is strict mode?

**Answer:**
Enforces strong type checks.

---

### 20. What is declaration file (.d.ts)?

**Answer:**
Defines types for JS libraries.

---

# 🧠 REAL INTERVIEW TRAPS (IMPORTANT)

---

### 💥 1. Explain event loop deeply

👉 They expect microtask vs macrotask clarity

---

### 💥 2. Explain closure with real use case

👉 Example: caching, private variables

---

### 💥 3. Why TypeScript over JavaScript?

👉 Scalability, maintainability

---

### 💥 4. Explain this in different contexts

👉 Most candidates fail here

---

### 💥 5. Explain prototype chain

👉 Must know inheritance deeply

---

# 🚀 What Next?

Now you’ve covered almost everything:

- ✅ Frontend
- ✅ Backend
- ✅ Database
- ✅ System Design
- ✅ JS + TS

---

## Remaining final sections:

1. ⚙️ REST APIs + Testing + CI/CD (very practical + frequently asked)
2. 🧠 AI + LLM systems (huge bonus for your profile)

---

## 👉 Recommendation:

Go next with:

👉 **REST APIs + Testing + CI/CD**

This is what interviewers use to judge **real-world engineering maturity**.

Just say:
**“REST + CI/CD”** and I’ll complete your full interview prep.

Good — this is where your profile can **differentiate massively** right now. Most candidates give generic answers; you should talk in **systems + cost + scaling** terms.

I’ll structure this into:

1. LLM Fundamentals (core concepts interviewers test)
2. AI System Design (real-world architectures)
3. Production Challenges (cost, latency, scaling)

Each → **20 high-value Q&A**

---

# 🧠 Section 1: LLM Fundamentals (20 Q&A)

---

### 1. What is an LLM?

**Answer:**
A Large Language Model is a neural network trained on massive text data to understand and generate language.

Example: GPT

---

### 2. How do LLMs work?

**Answer:**
Based on **transformer architecture** using self-attention to predict next tokens.

---

### 3. What is a token?

**Answer:**
Smallest unit of text (word/subword).

---

### 4. What is context window?

**Answer:**
Max tokens model can process in one request.

---

### 5. What is temperature?

**Answer:**
Controls randomness:

- Low → deterministic
- High → creative

---

### 6. What is top_p?

**Answer:**
Nucleus sampling → controls probability distribution.

---

### 7. What is prompt engineering?

**Answer:**
Designing input prompts to guide model output.

---

### 8. What is few-shot prompting?

**Answer:**
Providing examples in prompt.

---

### 9. What is fine-tuning?

**Answer:**
Training model on custom dataset.

---

### 10. What is embeddings?

**Answer:**
Vector representation of text.

---

### 11. What is vector database?

**Answer:**
Stores embeddings for similarity search.

Examples:

- Pinecone
- Weaviate

---

### 12. What is RAG (Retrieval Augmented Generation)?

**Answer:**
Combine LLM + external data retrieval.

---

### 13. What is hallucination?

**Answer:**
Model generates incorrect info confidently.

---

### 14. How to reduce hallucination?

**Answer:**

- RAG
- Better prompts
- Grounding

---

### 15. What is latency in LLMs?

**Answer:**
Time taken to generate response.

---

### 16. What is token cost?

**Answer:**
Billing based on input/output tokens.

---

### 17. What is streaming response?

**Answer:**
Sending tokens as they generate.

---

### 18. What is function calling/tool calling?

**Answer:**
LLM triggers backend APIs.

---

### 19. What is guardrails?

**Answer:**
Control outputs (safety, format).

---

### 20. When NOT to use LLM?

**Answer:**

- deterministic logic
- strict calculations
- real-time critical systems

---

---

# 🚀 Section 2: AI System Design (20 Q&A)

---

### 1. Design a ChatGPT-like system

**Answer:**
Components:

- API layer
- LLM service
- Prompt builder
- Cache
- DB

---

### 2. Design RAG system

**Answer:**
Flow:

1. User query
2. Convert to embeddings
3. Search vector DB
4. Pass context to LLM
5. Generate answer

---

### 3. What is embedding pipeline?

**Answer:**
Process of converting documents into vectors.

---

### 4. How to store embeddings?

**Answer:**

- Vector DB
- Chunk documents

---

### 5. Chunking strategy?

**Answer:**

- Fixed size
- Semantic chunking

---

### 6. How to improve search quality?

**Answer:**

- Hybrid search (BM25 + vector)
- Re-ranking

---

### 7. What is re-ranking?

**Answer:**
Sort retrieved results using another model.

---

### 8. How to design AI chatbot for enterprise?

**Answer:**

- RAG
- Access control
- Logging
- Monitoring

---

### 9. How to handle large documents?

**Answer:**

- Chunking
- Summarization

---

### 10. What is prompt template?

**Answer:**
Structured prompt format.

---

### 11. How to maintain conversation memory?

**Answer:**

- Store chat history
- Summarize old messages

---

### 12. What is multi-agent system?

**Answer:**
Multiple LLMs handling tasks.

---

### 13. How to integrate tools?

**Answer:**

- Function calling
- API orchestration

---

### 14. What is AI workflow orchestration?

**Answer:**
Managing multi-step AI pipelines.

---

### 15. What is fallback mechanism?

**Answer:**
If LLM fails → use rule-based system.

---

### 16. How to design recommendation system?

**Answer:**

- embeddings
- user behavior
- ranking

---

### 17. How to evaluate LLM output?

**Answer:**

- human eval
- automated metrics

---

### 18. What is caching in AI systems?

**Answer:**
Cache responses to reduce cost.

---

### 19. What is prompt injection?

**Answer:**
Malicious input manipulating LLM.

---

### 20. How to secure AI system?

**Answer:**

- input validation
- output filtering
- auth

---

---

# ⚡ Section 3: Production Challenges (20 Q&A)

---

### 1. How to reduce LLM cost?

**Answer:**

- caching
- smaller models
- token optimization

---

### 2. How to reduce latency?

**Answer:**

- streaming
- parallel calls
- caching

---

### 3. How to scale AI system?

**Answer:**

- load balancing
- async processing
- queue system

---

### 4. How to handle high traffic?

**Answer:**

- rate limiting
- autoscaling

---

### 5. How to monitor AI system?

**Answer:**

- logs
- metrics
- alerts

---

### 6. What is cold start problem?

**Answer:**
Delay when model starts.

---

### 7. How to handle failures?

**Answer:**

- retries
- fallback model

---

### 8. What is batching?

**Answer:**
Process multiple requests together.

---

### 9. What is token optimization?

**Answer:**
Reduce prompt size.

---

### 10. What is streaming vs batch?

**Answer:**

- streaming → real-time
- batch → offline

---

### 11. What is model selection strategy?

**Answer:**

- simple → small model
- complex → large model

---

### 12. What is hybrid system?

**Answer:**
LLM + rule-based logic.

---

### 13. What is observability in AI?

**Answer:**

- prompt logs
- token usage
- errors

---

### 14. How to handle hallucination in production?

**Answer:**

- RAG
- validation layer

---

### 15. What is user personalization?

**Answer:**
Tailor responses using user data.

---

### 16. What is rate limiting in AI APIs?

**Answer:**
Prevent cost explosion.

---

### 17. What is data privacy concern?

**Answer:**
Sensitive data leakage.

---

### 18. How to secure embeddings?

**Answer:**
Encrypt + access control.

---

### 19. What is multi-model architecture?

**Answer:**
Use different models for different tasks.

---

### 20. What is cost vs quality tradeoff?

**Answer:**
Better model → higher cost.

---

# 🧠 REAL INTERVIEW QUESTIONS (HIGH PROBABILITY)

---

### 💥 1. Design AI chatbot for customer support

👉 Expect:

- RAG
- fallback
- monitoring

---

### 💥 2. How did you reduce AI cost?

👉 Answer like:

- caching responses
- prompt optimization
- smaller models

---

### 💥 3. How to prevent hallucination?

👉 Must say:

- RAG + validation

---

### 💥 4. How to scale AI system to 1M users?

👉 Talk about:

- queue
- autoscaling
- caching

---

### 💥 5. Design AI video processing system (VERY relevant for you)

👉 Combine:

- LLM + FFmpeg + pipeline

---

# 🚀 FINAL INSIGHT (For YOU)

You already worked on:

- video infra
- pipelines
- backend

👉 If you combine that with AI → **you become top 5% candidate**

---

# 🎯 If You Want NEXT LEVEL

I can now:

1. 🔥 Take your **mock interview (hard mode)**
2. 📄 Build your **10–15 LPA resume**
3. 🎯 Give **company-specific questions (Flipkart, Amazon, startups)**
4. 🚀 Help you design a **real AI project (portfolio killer)**

---

## 👉 Best next step:

Say:

👉 **“Take my mock interview (senior fullstack)”**

I’ll simulate a real interview and challenge you like a hiring panel would.

Got it — I’ll give you a **clean, interview-focused deep revision pack** for **REST APIs + Testing + CI/CD**, but this time more **practical + senior-level (what interviewers actually expect you to say)**.

---

# ⚙️ 1. REST APIs (Senior-Level Q&A)

---

### 🔥 1. How do you design a production-ready REST API?

**Answer (structured — say like this in interview):**

- Clear resource-based URLs (`/users`, `/orders`)
- Proper HTTP methods
- Validation layer (Zod/Joi)
- Authentication (JWT/OAuth)
- Rate limiting + throttling
- Logging + monitoring
- Versioning (`/v1`)

---

### 🔥 2. PUT vs PATCH (real difference)

**Answer:**

- PUT → replaces full resource
- PATCH → partial update

👉 Interview tip: mention **idempotency**

---

### 🔥 3. How do you handle large-scale pagination?

**Answer:**

- Offset → simple but slow
- Cursor-based → scalable (preferred)

```bash
/users?cursor=abc123&limit=10
```

---

### 🔥 4. How do you secure APIs?

**Answer:**

- Auth → JWT / OAuth
- Rate limiting
- Input validation
- HTTPS
- CORS control

---

### 🔥 5. How do you handle API versioning?

**Answer:**

- URL versioning → `/v1/users` (most common)
- Header-based → advanced use

---

### 🔥 6. How do you design error responses?

**Answer:**

```json
{
  "success": false,
  "message": "Invalid input",
  "errorCode": "VALIDATION_ERROR"
}
```

---

### 🔥 7. How to avoid N+1 queries?

**Answer:**

- Use joins / aggregation
- Batch queries
- DataLoader (GraphQL case)

---

### 🔥 8. How to handle rate limiting?

**Answer:**

- Redis + token bucket
- IP/user-based limits

---

### 🔥 9. How to cache APIs?

**Answer:**

- CDN (static)
- Redis (dynamic)
- HTTP caching headers

---

### 🔥 10. REST vs GraphQL (when to choose?)

**Answer:**

- REST → simple, stable APIs
- GraphQL → flexible, complex frontend needs

---

---

# 🧪 2. TESTING (Senior-Level Practical)

---

### 🔥 1. What testing strategy do you follow?

**Answer (important):**

- Unit → business logic
- Integration → DB/API
- E2E → full flow

👉 Mention **test pyramid**

---

### 🔥 2. How do you test APIs in Node.js?

**Answer:**

- Jest
- Supertest for HTTP

---

### 🔥 3. Example API test

```js
await request(app).get('/users').expect(200);
```

---

### 🔥 4. What is mocking?

**Answer:**
Replace external dependencies (DB, APIs)

---

### 🔥 5. What not to mock?

**Answer:**

- Avoid mocking core logic
- Mock only external services

---

### 🔥 6. What is flaky test?

**Answer:**
Randomly failing test → avoid via deterministic data

---

### 🔥 7. How do you test DB?

**Answer:**

- Test DB (separate)
- In-memory DB
- Transactions rollback

---

### 🔥 8. What is contract testing?

**Answer:**
Ensure API compatibility between services

---

### 🔥 9. How to test async code?

**Answer:**

```js
await expect(fn()).resolves.toBe(...)
```

---

### 🔥 10. What is coverage?

**Answer:**
% of code tested — but don’t over-optimize it

---

### 🔥 11. How do you structure tests?

**Answer:**

- Arrange
- Act
- Assert

---

### 🔥 12. What is snapshot testing?

**Answer:**
UI comparison test (mostly frontend)

---

### 🔥 13. When do tests run?

**Answer:**

- On commit (CI)
- Before deploy

---

### 🔥 14. How to speed up tests?

**Answer:**

- Parallel execution
- Mock heavy dependencies

---

### 🔥 15. Best practices

**Answer:**

- Small tests
- Independent
- Fast

---

---

# 🚀 3. CI/CD (Real Production Level)

---

### 🔥 1. Design a CI/CD pipeline

**Answer (say this clearly):**

Flow:

1. Code push
2. Run tests
3. Build artifact
4. Dockerize
5. Deploy

---

### 🔥 2. Tools you should mention

- GitHub Actions
- Docker
- Kubernetes

---

### 🔥 3. Example pipeline (real-world)

```yaml
- install deps
- run tests
- build docker image
- push to registry
- deploy to server
```

---

### 🔥 4. What is blue-green deployment?

**Answer:**
Two environments → switch traffic

---

### 🔥 5. What is canary deployment?

**Answer:**
Release to small % of users first

---

### 🔥 6. What is rollback strategy?

**Answer:**

- Keep previous version
- Auto rollback on failure

---

### 🔥 7. How do you handle secrets?

**Answer:**

- Environment variables
- Secret manager

---

### 🔥 8. What is artifact?

**Answer:**
Build output (Docker image, bundle)

---

### 🔥 9. How do you monitor deployment?

**Answer:**

- Logs
- Metrics
- Alerts

---

### 🔥 10. What is zero downtime deployment?

**Answer:**
Deploy without affecting users

---

### 🔥 11. What is linting?

**Answer:**
Code quality checks

---

### 🔥 12. What is pre-commit hook?

**Answer:**
Run checks before commit

---

### 🔥 13. What is environment separation?

**Answer:**

- Dev
- Staging
- Production

---

### 🔥 14. How to handle migrations?

**Answer:**

- Versioned migrations
- Run before deploy

---

### 🔥 15. CI vs CD difference

**Answer:**

- CI → build & test
- CD → deploy

---

---

# 🧠 REAL INTERVIEW QUESTIONS (VERY IMPORTANT)

---

### 💥 1. “Design a scalable REST API”

👉 Talk:

- load balancer
- caching
- DB optimization

---

### 💥 2. “How do you ensure code quality?”

👉 Answer:

- testing
- code reviews
- CI checks

---

### 💥 3. “Explain your CI/CD pipeline”

👉 Give real example (Docker + GitHub Actions)

---

### 💥 4. “How do you deploy without downtime?”

👉 Blue-green / rolling

---

### 💥 5. “How do you test microservices?”

👉 Unit + contract + integration

---

# 🎯 FINAL ADVICE (FOR YOU)

You already have:

- backend
- devops
- system design

👉 If you answer like this (structured + practical),
you’ll easily stand out for **10–15+ LPA roles**

---

# 🚀 If You Want Next Step

Now the best thing you can do:

👉 **Mock interview (real pressure)**

Say:
**“Start mock interview”**

I’ll simulate:

- DSA + backend + system design
- Real interviewer behavior
- Feedback on your answers

This is where you actually level up.
