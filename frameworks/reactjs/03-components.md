# Components

### What is a component in reactjs ?

In React, a **component** is a **reusable piece of UI** written as JavaScript (usually JSX) that controls how part of the screen looks and behaves.

Think of components like **building blocks** of a website or app.

Examples:

- Navbar
- Button
- Login Form
- Product Card
- Sidebar
- Footer

Instead of writing one huge page, you split UI into small components.

---

## Simple Example

```jsx
function Welcome() {
  return <h1>Hello User</h1>;
}

export default Welcome;
```

Use it inside another component:

```jsx
function App() {
  return (
    <div>
      <Welcome />
    </div>
  );
}
```

Here:

- `App` is a component
- `Welcome` is a component

---

## Why Components Are Powerful

### Reusability

Write once, use many times.

```jsx
<Button />
<Button />
<Button />
```

### Maintainability

Small files are easier to manage than one 2000-line page.

### Scalability

Large apps are built from many components working together.

---

## Real Admin Panel Example

```bash
components/
│── Button.jsx
│── Input.jsx
│── Modal.jsx
│── Table.jsx
│── Loader.jsx
```

Pages use them:

```jsx
<UserTable />
<CreateUserModal />
<DeleteConfirmModal />
```

---

## Types of Components

### 1. UI Components

Generic reusable pieces:

- Button
- Card
- Input
- Modal

### 2. Feature Components

Business logic related:

- UserForm
- OrderTable
- ProductEditor

### 3. Layout Components

Page structure:

- Sidebar
- Header
- DashboardLayout

---

## Components Can Receive Props

Props = data passed into component.

```jsx
function Button({ title }) {
  return <button>{title}</button>;
}
```

Use:

```jsx
<Button title="Save" />
<Button title="Delete" />
```

---

## Components Can Have State

Using hooks like `useState`.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return <button>{count}</button>;
}
```

---

## Modern React Uses Functional Components

Preferred style:

```jsx
function Navbar() {
  return <nav>Menu</nav>;
}
```

Older class components are mostly legacy now.

---

## Best Practice Structure

```bash
src/
│── components/
│   ├── ui/
│   ├── common/
│   └── forms/
```

---

## Easy Analogy

A website is like a car:

- Engine = logic
- Wheels = forms
- Doors = buttons
- Dashboard = navbar

Each part = a component.

---

## Final Definition

A React component is a **self-contained reusable UI + logic unit** used to build applications efficiently in React.

---

# explain **how components re-render internally in React using Virtual DOM**, which is important for interviews.

Yes — understanding **how components re-render internally in React** is one of the most important concepts for interviews and real projects.

# How React Components Re-render Internally

A component **re-renders** when React decides it needs to update the UI.

That means React **runs the component function again** to produce updated JSX.

---

# 1. Functional Component Is Just a Function

```jsx id="4q1h6v"
function Counter() {
  return <h1>Hello</h1>;
}
```

When React renders it:

```js id="h4l3rn"
Counter();
```

It returns JSX.

So re-render = React calls the function again.

---

# 2. When Does a Component Re-render?

Main triggers:

### A. State Changes

```jsx id="zh8n8z"
const [count, setCount] = useState(0);
```

When `setCount(1)` runs:

✅ React schedules update
✅ Calls component again
✅ New UI appears

---

### B. Props Change

```jsx id="02l8h0"
<UserCard name='Pulkit' />
```

If parent sends new prop:

```jsx id="6wmpqy"
<UserCard name='Rahul' />
```

Child re-renders.

---

### C. Parent Re-renders

If parent re-renders, children usually re-render too.

---

# 3. Example Re-render Flow

```jsx id="0r4xyf"
function App() {
  const [count, setCount] = useState(0);

  return (
    <>
      <h1>{count}</h1>
      <button onClick={() => setCount(count + 1)}>Add</button>
    </>
  );
}
```

Click button:

1. `setCount(1)`
2. React marks component dirty
3. Calls `App()` again
4. Gets new JSX with `1`
5. Updates DOM efficiently

---

# 4. Does Full Page Reload?

❌ No

React updates only changed parts.

Example:

Old:

```html id="frl26z"
<h1>0</h1>
```

New:

```html id="b7p4iw"
<h1>1</h1>
```

Only text changes.

---

# 5. Virtual DOM Concept

React creates in-memory tree of UI.

Old tree vs new tree.

Then compares differences.

This process = **Reconciliation**

Then updates real DOM minimally.

---

# 6. Example Visualization

```jsx id="l7r7px"
<h1>0</h1>
```

becomes:

```jsx id="zv4zui"
<h1>1</h1>
```

Only node text updated.

---

# 7. Child Re-render Problem

```jsx id="m84v7k"
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <>
      <Child />
      <button onClick={() => setCount(count + 1)}>Add</button>
    </>
  );
}
```

When parent updates:

✅ Parent re-renders
✅ Child also re-renders by default

Even if Child data unchanged.

---

# 8. Prevent Unnecessary Re-renders

Use:

### React.memo

```jsx id="ndd9km"
const Child = React.memo(function Child() {
  return <h1>Child</h1>;
});
```

Now Child skips if props same.

---

### useMemo

Memoize expensive values.

### useCallback

Memoize functions passed as props.

---

# 9. Important Interview Question

## Does state update immediately?

```jsx id="5k6tp4"
setCount(count + 1);
console.log(count);
```

Often logs old value.

Because React batches updates asynchronously.

---

# 10. Real Internal Lifecycle

On update:

1. Event occurs
2. setState called
3. React scheduler queues update
4. Component function runs again
5. Hooks re-evaluated in same order
6. New virtual DOM created
7. Diff with old tree
8. Real DOM patched

---

# 11. Hooks Rule Exists Because of Re-renders

React tracks hooks by order.

```jsx id="snm7ra"
useState();
useEffect();
useState();
```

Same order every render required.

That’s why no hooks inside if/loops.

---

# 12. Debug Example

```jsx id="b31jlwm"
function Test() {
  console.log('rendered');
  return <h1>Hello</h1>;
}
```

Every re-render prints:

```bash id="34b25w"
rendered
```

Great way to learn.

---

# 13. Performance Tip for Admin Panels

For large tables:

- Use `React.memo`
- Pagination
- Virtualization
- Stable props
- Zustand selectors
- TanStack Query caching

---

# 14. Super Simple Summary

Re-render means:

> React runs your component again, compares old vs new UI, updates only what changed.

---

# 15. Interview One-Line Answer

“In React, a component re-renders when its state, props, or parent changes. React re-executes the component function, compares new virtual DOM with previous DOM, and updates only changed nodes.”

---

# What is JSX ?

In React, **JSX** stands for **JavaScript XML**.

It is a syntax extension that lets you write **HTML-like code inside JavaScript** to describe UI.

Instead of writing raw JavaScript DOM code, JSX makes components easier to read and build.

---

# Simple Example

```jsx id="d9s2zq"
const element = <h1>Hello World</h1>;
```

This looks like HTML, but it is JSX.

React converts it behind the scenes into JavaScript.

---

# What JSX Becomes Internally

```jsx id="v1b6ul"
const element = <h1>Hello World</h1>;
```

becomes:

```js id="g0i8oe"
React.createElement('h1', null, 'Hello World');
```

So JSX is just a cleaner way to write UI.

---

# Why JSX Is Useful

### Readable

```jsx id="j4tkcw"
<div>
  <h1>Dashboard</h1>
  <button>Save</button>
</div>
```

Much easier than manual DOM creation.

### Mix UI + Logic

```jsx id="p6p7e9"
const name = 'Pulkit';

<h1>Hello {name}</h1>;
```

### Great for Components

```jsx id="ghhnhd"
<UserCard name='Pulkit' />
```

---

# JSX in Components

```jsx id="6z2vul"
function App() {
  return <h1>Welcome</h1>;
}
```

This component returns JSX.

---

# Dynamic Values in JSX

Use `{}` to run JavaScript inside JSX.

```jsx id="n2p8zq"
const age = 25;

<p>Age: {age}</p>;
```

```jsx id="4zjz3p"
<p>Next year: {age + 1}</p>
```

---

# Conditional Rendering

```jsx id="u3g6gm"
{
  isLoggedIn ? <Dashboard /> : <Login />;
}
```

---

# List Rendering

```jsx id="4tlyhh"
users.map((user) => <li key={user.id}>{user.name}</li>);
```

---

# JSX Rules

## 1. Return One Parent Element

✅ Correct

```jsx id="7lr7y9"
return (
  <div>
    <h1>Hello</h1>
    <p>Text</p>
  </div>
);
```

Or use fragment:

```jsx id="2a0k8x"
return (
  <>
    <h1>Hello</h1>
    <p>Text</p>
  </>
);
```

---

## 2. Use `className`, not `class`

```jsx id="ntrdjr"
<div className='box'></div>
```

Because `class` is reserved in JavaScript.

---

## 3. Self Closing Tags

```jsx id="n3e9v7"
<img src="x.jpg" />
<input />
```

---

# JSX Is Not HTML

Looks similar, but different:

- `className` instead of `class`
- `htmlFor` instead of `for`
- JavaScript expressions allowed
- Event handlers use camelCase

```jsx id="95vdpo"
<button onClick={saveData}>Save</button>
```

---

# In Vite Projects

Files usually use:

```bash id="m7c7g0"
App.jsx
Navbar.jsx
Login.jsx
```

`.jsx` means JavaScript with JSX syntax.

---

# Interview Answer

“JSX is a syntax extension for JavaScript used in React to write HTML-like UI code inside JavaScript. It gets compiled into `React.createElement()` calls.”

---

# Super Simple Analogy

Without JSX:

```js id="ymp9b6"
createElement(...)
```

With JSX:

```jsx id="8m35tw"
<div>Hello</div>
```

JSX = cleaner UI language for React.

---

# Provs VS STate

In React, **Props** and **State** are two core ways components handle data.

Many beginners confuse them, but the difference is simple:

- **Props** = data passed **from parent to child**
- **State** = data managed **inside the component itself**

---

# Quick Comparison

| Feature              | Props                        | State              |
| -------------------- | ---------------------------- | ------------------ |
| Owned by             | Parent component             | Current component  |
| Mutable by component | ❌ No                        | ✅ Yes             |
| Used for             | Passing data/configuration   | Dynamic UI data    |
| Causes re-render     | Yes (when changed by parent) | Yes (when updated) |

---

# 1. Props Example

Parent sends a name to child.

```jsx id="2n4h8a"
function UserCard(props) {
  return <h1>Hello {props.name}</h1>;
}

function App() {
  return <UserCard name='Pulkit' />;
}
```

### What happens?

- `App` passes `name="Pulkit"`
- `UserCard` receives it as props
- Child displays it

Props are **read-only**.

---

# Cleaner Version (Destructuring)

```jsx id="h0k8qz"
function UserCard({ name }) {
  return <h1>Hello {name}</h1>;
}
```

---

# 2. State Example

Component manages its own counter.

```jsx id="m6j9pz"
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <>
      <h1>{count}</h1>
      <button onClick={() => setCount(count + 1)}>Add</button>
    </>
  );
}
```

### What happens?

- `count` is state
- Clicking button updates state
- Component re-renders with new value

---

# Real Difference

## Props = Input from Outside

```jsx id="j8w2ey"
<ProductCard title='iPhone' price={999} />
```

## State = Internal Memory

```jsx id="mk3g8q"
const [isOpen, setIsOpen] = useState(false);
```

Used for:

- Modal open/close
- Form values
- Toggle switches
- Counter
- Search text

---

# Real Admin Panel Example

## Parent Component

```jsx id="r1v8dt"
<UserTable users={users} />
```

Here `users` is props.

## Inside UserTable

```jsx id="y7g5lf"
const [page, setPage] = useState(1);
```

Here `page` is state.

---

# Props + State Together Example

```jsx id="9e6klt"
function ProductCard({ name }) {
  const [liked, setLiked] = useState(false);

  return (
    <div>
      <h1>{name}</h1>
      <button onClick={() => setLiked(!liked)}>
        {liked ? 'Liked' : 'Like'}
      </button>
    </div>
  );
}
```

### Here:

- `name` = props from parent
- `liked` = internal state

---

# Important Rule

## Never Modify Props Directly

❌ Wrong

```jsx id="dbm5q8"
props.name = 'Rahul';
```

✅ Correct: parent changes props by sending new value.

---

# State Update Best Practice

When depending on previous value:

```jsx id="q8o4ve"
setCount((prev) => prev + 1);
```

---

# Easy Analogy

Think of a **TV**:

- Props = channels sent by cable provider
- State = TV volume level you control

---

# Interview Answer

“Props are read-only values passed from parent to child components. State is mutable data managed within a component using hooks like `useState`, and changing state causes re-rendering.”

---

# One-Line Memory Trick

**Props = Passed Data**
**State = Stored Data**

---

# When to Use Which?

## Use Props For:

- Title
- Product data
- User data from parent
- Settings from parent

## Use State For:

- Form input
- Toggle UI
- Loading state
- Pagination page
- Search query

---

# Next Level Concept

Sometimes parent state becomes child props:

```jsx id="54v7gh"
const [user, setUser] = useState(...);

<UserCard user={user} />
```

So parent state can flow down as child props.

---

# explain **Props Drilling vs Zustand vs Context API vs Local State** with real-world examples for enterprise React apps.

Yes — this is a **very important real-world React concept**.

When apps grow, passing data through many components becomes painful. That leads to:

- Props Drilling
- Context API
- Global State (Zustand)
- Local State

Let’s understand clearly.

---

# 1. Local State (`useState`)

Used inside one component only.

```jsx id="x1c8ra"
function SearchBox() {
  const [text, setText] = useState('');
}
```

Best for:

✅ Input fields
✅ Modal open/close
✅ Toggle button
✅ Small UI state

---

# Example

```jsx id="2ldx8q"
const [isOpen, setIsOpen] = useState(false);
```

Only this component needs it.

---

# 2. Props Drilling

Passing data through many levels.

```jsx id="f7g2me"
<App user={user} />
  <Layout user={user} />
    <Sidebar user={user} />
      <Profile user={user} />
```

Even if `Layout` and `Sidebar` don’t need `user`, they must pass it.

This is called **props drilling**.

---

# Why It Becomes Bad

❌ Too much repetitive code
❌ Hard to maintain
❌ Deep trees become messy

---

# Real Example

Admin panel:

```jsx id="s6u9wh"
App -> Dashboard -> Header -> UserMenu
```

Need logged-in user everywhere.

Passing props manually is annoying.

---

# 3. Context API

React built-in solution for shared data.

```jsx id="f4q0ls"
<AuthProvider>
  <App />
</AuthProvider>
```

Then any child can access:

```jsx id="e9g6nt"
const { user } = useContext(AuthContext);
```

---

# Good For:

✅ Theme (dark/light)
✅ Auth user
✅ Language
✅ Small shared app state

---

# Limitations of Context

❌ Frequent updates can cause re-renders
❌ Large state logic gets messy
❌ Not ideal for complex dashboards

---

# 4. Zustand (Recommended for Real Apps)

Zustand is modern global state.

Simple store:

```jsx id="g1u8tv"
import { create } from 'zustand';

const useAuthStore = create((set) => ({
  user: null,
  setUser: (user) => set({ user }),
}));
```

Use anywhere:

```jsx id="0e8snb"
const user = useAuthStore((state) => state.user);
```

---

# Why Zustand Is Great

✅ Very small
✅ Fast
✅ Easy syntax
✅ No provider needed
✅ Selective re-renders
✅ Perfect for admin panels

---

# Real Admin Panel Usage

Use Zustand for:

- Logged-in user
- Sidebar collapsed state
- Filters
- Cart
- Notifications
- UI preferences

---

# 5. Comparison Table

| Solution    | Best For                   | Avoid When               |
| ----------- | -------------------------- | ------------------------ |
| Local State | Single component           | Shared across app        |
| Props       | Parent → child direct data | Deep component trees     |
| Context API | Small global data          | Complex frequent updates |
| Zustand     | Medium/Large apps          | Tiny toy apps            |

---

# Real Example: Logged-in User

## Bad (Props Drilling)

```jsx id="73hy4w"
<App user={user} />
<Layout user={user} />
<Header user={user} />
<UserMenu user={user} />
```

---

## Better (Context)

```jsx id="m8s3fa"
const { user } = useContext(AuthContext);
```

---

## Best for Enterprise (Zustand)

```jsx id="84k2zi"
const user = useAuthStore((s) => s.user);
```

Anywhere directly.

---

# How I’d Build Your Admin Panel

## Local State

```jsx id="n6p4ey"
const [openModal, setOpenModal] = useState(false);
```

## Zustand

```jsx id="v2h8yu"
auth user
sidebar state
filters
theme
```

## TanStack Query

TanStack Query for API data:

```jsx id="t5f9ad"
users;
orders;
products;
```

---

# Golden Rule

Use:

- **Local State** for UI state
- **Zustand** for client global state
- **TanStack Query** for server/API state

This is modern architecture.

---

# Interview Answer

“Props drilling is passing props through intermediate components unnecessarily. Context API avoids this for shared data, but Zustand is better for scalable global state management with optimized re-renders.”

---

# My Recommendation for You

Since you want enterprise React + admin panel:

✅ Zustand + TanStack Query
❌ Avoid overusing Context
✅ useState for local UI only

---

# Final Memory Trick

```text
useState = local room
Props = hand delivery
Context = building announcement
Zustand = central control room
```

---

# explain **Zustand vs Redux Toolkit vs Context API in 2026 with performance comparison and code examples**.

Yes — this is one of the most common interview and architecture questions in modern React apps.

# Zustand vs Redux Toolkit vs Context API (2026)

If you’re building dashboards, admin panels, ecommerce apps, or SaaS products, choosing state management matters.

---

# Quick Verdict (Modern Recommendation)

✅ Small app → Context API
✅ Medium/Large React app → Zustand
✅ Very large enterprise team / strict patterns → Redux Toolkit

---

# 1. Context API

Built into React.

```jsx id="q1x7pl"
<AuthProvider>
  <App />
</AuthProvider>
```

Use:

```jsx id="5f9mwe"
const user = useContext(AuthContext);
```

---

## Best For

- Theme mode
- Logged in user
- Language
- Simple shared settings

---

## Problems

❌ Frequent updates may re-render many consumers
❌ Large logic becomes messy
❌ Boilerplate with many providers

---

# 2. Zustand

Minimal modern global store.

```jsx id="n8r3lw"
import { create } from 'zustand';

const useStore = create((set) => ({
  count: 0,
  inc: () => set((s) => ({ count: s.count + 1 })),
}));
```

Use:

```jsx id="p2d6va"
const count = useStore((s) => s.count);
```

---

## Why Developers Love It

✅ Super simple
✅ Tiny bundle size
✅ No Provider needed
✅ Selective subscriptions
✅ Great performance
✅ Less boilerplate

---

## Best For

- Admin panels
- Dashboards
- Ecommerce cart
- Filters
- Auth state
- UI preferences

---

# 3. Redux Toolkit

Official modern Redux way.

```jsx id="j6u1ck"
const counterSlice = createSlice({
  name: 'counter',
  initialState: { count: 0 },
  reducers: {
    increment: (state) => {
      state.count++;
    },
  },
});
```

---

## Why Companies Use It

✅ Predictable architecture
✅ DevTools time travel
✅ Strong standards
✅ Middleware ecosystem
✅ Large team consistency

---

## Downsides

❌ More setup than Zustand
❌ More concepts to learn
❌ More files/slices/actions

---

# Comparison Table

| Feature           | Context API | Zustand   | Redux Toolkit |
| ----------------- | ----------- | --------- | ------------- |
| Built into React  | ✅          | ❌        | ❌            |
| Easy to Learn     | ✅          | ✅        | Medium        |
| Boilerplate       | Low         | Very Low  | Medium        |
| Performance       | Medium      | High      | High          |
| Large App Scaling | Medium      | High      | Very High     |
| DevTools          | Limited     | Available | Excellent     |
| Best for Teams    | Medium      | High      | Very High     |

---

# Performance Example

## Context Problem

```jsx id="a7x4nz"
<AuthContext.Provider value={{ user }}>
```

If value changes, many consumers may re-render.

---

## Zustand Better

```jsx id="3k1zts"
const user = useStore((s) => s.user);
```

Only subscribed components update.

---

# What I’d Use for Your Admin Panel

## Use Zustand For:

```text id="l6n2wy"
auth user
sidebar collapse
filters
selected rows
theme
notifications
```

## Use TanStack Query For:

```text id="c4m8ha"
users API
orders API
products API
analytics API
```

## Use Local State For:

```text id="e9w7bf"
modal open
form input
tabs
dropdown
```

---

# Real Architecture (Best in 2026)

```text id="0z4hyq"
Local UI State = useState
Global Client State = Zustand
Server State = TanStack Query
Forms = React Hook Form
Validation = Zod
```

This is elite modern stack.

---

# Interview Answer

“Context API is good for small shared state. Zustand provides simpler and faster scalable global state with less boilerplate. Redux Toolkit is ideal for very large teams needing strict predictable patterns and tooling.”

---

# My Honest Advice for You

Since you’re building:

- Admin panel
- Users / Orders / Products
- Auth
- CRUD dashboard

👉 Choose:

Zustand + TanStack Query

Not Redux unless company requires it.

---

# Final Ranking (2026)

### Personal / Freelance / Startup

🥇 Zustand

### Large Corporate Team

🥇 Redux Toolkit

### Tiny App

🥇 Context API

---

# show you **complete production folder structure using Zustand + TanStack Query + React Router + JWT Auth** used by senior React engineers.

# Production-Level React + Vite Folder Structure (2026)

For your admin panel (Users / Orders / Products / Auth), this is a **senior-engineer level scalable structure** using:

- React
- Vite
- React Router
- Zustand
- TanStack Query
- Axios
- React Hook Form
- Zod
- Tailwind CSS

---

# Full Folder Structure

```bash id="g7u2fw"
src/
│── app/
│   ├── router.jsx
│   ├── providers.jsx
│   └── constants.js
│
│── assets/
│   ├── images/
│   ├── icons/
│   └── fonts/
│
│── components/
│   ├── ui/
│   │   ├── Button.jsx
│   │   ├── Input.jsx
│   │   ├── Select.jsx
│   │   ├── Modal.jsx
│   │   ├── Table.jsx
│   │   └── Loader.jsx
│   │
│   ├── common/
│   │   ├── Header.jsx
│   │   ├── Sidebar.jsx
│   │   ├── Pagination.jsx
│   │   └── EmptyState.jsx
│   │
│   └── forms/
│       ├── UserForm.jsx
│       ├── ProductForm.jsx
│       └── LoginForm.jsx
│
│── features/
│   ├── auth/
│   │   ├── api.js
│   │   ├── hooks.js
│   │   ├── store.js
│   │   ├── schema.js
│   │   └── pages/
│   │       └── Login.jsx
│   │
│   ├── dashboard/
│   │   ├── api.js
│   │   └── pages/
│   │       └── Dashboard.jsx
│   │
│   ├── users/
│   │   ├── api.js
│   │   ├── hooks.js
│   │   ├── schema.js
│   │   ├── components/
│   │   │   └── UserTable.jsx
│   │   └── pages/
│   │       └── UsersPage.jsx
│   │
│   ├── products/
│   │   ├── api.js
│   │   ├── hooks.js
│   │   ├── schema.js
│   │   └── pages/
│   │
│   └── orders/
│       ├── api.js
│       ├── hooks.js
│       └── pages/
│
│── hooks/
│   ├── useDebounce.js
│   ├── usePagination.js
│   ├── useModal.js
│   └── usePermission.js
│
│── layouts/
│   ├── AdminLayout.jsx
│   └── AuthLayout.jsx
│
│── lib/
│   ├── axios.js
│   ├── queryClient.js
│   ├── cookie.js
│   └── helpers.js
│
│── store/
│   └── appStore.js
│
│── styles/
│   └── index.css
│
│── main.jsx
```

---

# Why This Structure Is Elite

## 1. `features/`

Each business module owns itself.

Example:

```text id="b6r3qm"
users/
products/
orders/
auth/
```

Inside each:

- API
- hooks
- schema
- pages
- components

This scales beautifully.

---

# Example Users Module

```bash id="6m1hyv"
features/users/
│── api.js
│── hooks.js
│── schema.js
│── components/
│   └── UserTable.jsx
│── pages/
│   └── UsersPage.jsx
```

---

# 2. `components/ui/`

Reusable design system components.

```text id="w8j4ce"
Button
Input
Modal
Table
Badge
Loader
```

Use anywhere.

---

# 3. `lib/`

App infrastructure.

```text id="v4h8oz"
axios instance
query client
cookie helpers
utility functions
```

---

# 4. `hooks/`

Reusable custom hooks.

```text id="2n9qyl"
useDebounce
useModal
usePagination
```

---

# Routing Structure

`app/router.jsx`

```jsx id="j2w8cr"
/
/login
/users
/products
/orders
/dashboard
```

Protected routes for admin pages.

---

# Zustand Stores

## Auth Store

```jsx id="e4y2zt"
user;
isAuthenticated;
login();
logout();
```

## UI Store

```jsx id="7f0kna"
sidebarOpen;
theme;
notifications;
```

---

# TanStack Query Usage

## users/hooks.js

```jsx id="x6m1vp"
useUsers();
useCreateUser();
useUpdateUser();
useDeleteUser();
```

Same pattern for products/orders.

---

# Validation with Zod

## users/schema.js

```jsx id="s9v3jk"
name required
email valid
role enum
password min 6
```

---

# Forms with React Hook Form

Use inside:

```text id="k3q1ub"
UserForm.jsx
ProductForm.jsx
LoginForm.jsx
```

---

# Best Import Pattern

Use aliases in Vite:

```jsx id="9r8ghy"
import Button from '@/components/ui/Button';
import useAuthStore from '@/features/auth/store';
```

---

# Real Admin Layout

```text id="y0p2vd"
Sidebar
Top Header
Content Area
Breadcrumb
Footer
```

---

# Best Practices Used by Seniors

## Keep API per feature

```text id="d8n5la"
users/api.js
orders/api.js
```

## Keep server state out of Zustand

Use TanStack Query instead.

## Keep local modal state local

```jsx id="n5z1qy"
const [open, setOpen] = useState(false);
```

---

# What NOT To Do

❌ Huge `components/` folder only
❌ Put all APIs in one file
❌ Use Context for everything
❌ Use Zustand for API caching
❌ Massive App.jsx

---

# Enterprise Stack Summary

```text id="8k3rce"
UI State → useState
Global Client State → Zustand
Server State → TanStack Query
Forms → React Hook Form
Validation → Zod
Routing → React Router
API → Axios
Styling → Tailwind
```

---

# My Recommendation for You

Since you’re backend + scaling minded:

Use **feature-first architecture** immediately.

This is what senior engineers prefer.

---

# If You Want Next Level

I can generate a **complete ready-to-run React + Vite admin panel source code** with:

✅ Login
✅ JWT Cookie Auth
✅ Dashboard
✅ Users CRUD
✅ Products CRUD
✅ Orders CRUD
✅ Sidebar UI
✅ Zustand
✅ TanStack Query
✅ Tailwind CSS

Just say **Build complete project code**

# Import and export methods in reactjs

In React, importing/exporting components correctly is important for clean architecture, maintainability, and team scalability.

# Best Way to Import / Export Components (2026)

## Quick Recommendation

✅ **Default export** for one main component per file
✅ **Named exports** for utility components / multiple exports
✅ **Barrel exports (`index.js`)** for cleaner imports
✅ Use path aliases (`@/`) in Vite

---

# 1. Default Export (Best for Main Component Files)

Use when file contains one component.

## `Button.jsx`

```jsx id="x3k7pa"
function Button() {
  return <button>Save</button>;
}

export default Button;
```

Import:

```jsx id="n4w8hz"
import Button from './Button';
```

---

## Why Good?

✅ Clean syntax
✅ Most common in React apps
✅ Great for pages/components

---

# Best Use Cases

```text id="d1r6mu"
Navbar.jsx
Sidebar.jsx
LoginPage.jsx
Dashboard.jsx
UserTable.jsx
```

---

# 2. Named Export (Best for Helpers / Multiple Components)

## `modals.jsx`

```jsx id="r6z3py"
export function DeleteModal() {
  return <div>Delete</div>;
}

export function EditModal() {
  return <div>Edit</div>;
}
```

Import:

```jsx id="a8v4ko"
import { DeleteModal, EditModal } from './modals';
```

---

## Why Good?

✅ Multiple exports in one file
✅ Better auto-import tooling
✅ Exact names enforced

---

# 3. Senior Engineer Rule

## For Components

Use **default export**

```jsx id="y7t2ce"
export default UserCard;
```

## For Hooks / Utils / Constants

Use **named exports**

```jsx id="m3q8nz"
export const useDebounce = () => {};
export const formatDate = () => {};
```

---

# Example

## `useDebounce.js`

```jsx id="p4j6vk"
export function useDebounce() {}
```

Import:

```jsx id="k2r9os"
import { useDebounce } from '@/hooks/useDebounce';
```

---

# 4. Barrel Export Pattern (Best for Clean Imports)

Instead of:

```jsx id="j0w3el"
import Button from './components/ui/Button';
import Input from './components/ui/Input';
import Modal from './components/ui/Modal';
```

Create:

## `components/ui/index.js`

```jsx id="t9f7zu"
export { default as Button } from './Button';
export { default as Input } from './Input';
export { default as Modal } from './Modal';
```

Now import:

```jsx id="q8m1ra"
import { Button, Input, Modal } from '@/components/ui';
```

🔥 Much cleaner.

---

# 5. Best Folder Import Example

```bash id="z2v6hy"
src/
components/ui/
features/users/
hooks/
lib/
```

Use:

```jsx id="u7p4je"
import { Button } from '@/components/ui';
import UserTable from '@/features/users/components/UserTable';
import { useDebounce } from '@/hooks/useDebounce';
```

---

# 6. Path Alias in Vite

## vite.config.js

```js id="v3c9ks"
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

Now use:

```jsx id="n1x5qd"
import Button from '@/components/ui/Button';
```

Instead of ugly:

```jsx id="f8u2lm"
import Button from '../../../components/ui/Button';
```

---

# 7. Best Naming Convention

## File Names

```text id="b6r4qe"
Button.jsx
UserTable.jsx
LoginPage.jsx
```

## Components

```jsx id="r9j1tk"
function UserTable() {}
```

PascalCase always.

---

# 8. Avoid This

❌ Mixed export styles randomly

```jsx id="x7k5pm"
export default Button;
export const helper = ...
```

❌ Deep relative imports

```jsx id="n6a2wy"
../../../../components/Button
```

❌ Anonymous default exports

```jsx id="e2t8uv"
export default () => <div />;
```

---

# 9. My Recommended Standard (Enterprise)

## Components

```jsx id="j4m9cz"
export default ComponentName;
```

## Hooks

```jsx id="z1q7lw"
export function useAuth() {}
```

## Utils

```jsx id="w3n5ke"
export const formatPrice = () => {};
```

## UI Folder

Use barrel export.

---

# 10. Real Admin Panel Example

```jsx id="f5r2ho"
import { Button, Input, Modal } from '@/components/ui';
import UsersPage from '@/features/users/pages/UsersPage';
import { useAuthStore } from '@/features/auth/store';
import { useUsers } from '@/features/users/hooks';
```

---

# 11. Interview Answer

“Default exports are ideal for single React components per file, while named exports are better for hooks, utilities, and multiple exports. In scalable apps, barrel exports and path aliases improve maintainability.”

---

# Final Best Practice Summary

```text id="a7m3pd"
Pages/Components = default export
Hooks/Utils = named export
Use index.js barrel files
Use @ alias
Use PascalCase for components
```

---

# My Honest Advice for You

For your admin panel:

✅ default export components
✅ named export hooks/utils
✅ barrel export ui components
✅ alias imports

That is senior-level clean code.

---

# Want Next Level?

I can also show you **how big companies structure imports automatically with ESLint + Prettier + absolute imports + auto sorting**.

# react js hooks

# Complete React Hooks Guide with Examples (2026)

In React, **Hooks** are built-in functions that let functional components use state, lifecycle behavior, refs, context, performance optimization, and more.

Think of hooks as the tools that make modern React components powerful.

---

# Core Hooks You Must Know First

## 1. `useState()` → Local State

Used when component data changes over time.

```jsx id="a1p9xe"
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <>
      <h1>{count}</h1>
      <button onClick={() => setCount(count + 1)}>Add</button>
    </>
  );
}
```

### Use Cases

- Counter
- Input values
- Modal open/close
- Toggle buttons

---

## 2. `useEffect()` → Side Effects / Lifecycle

Used for API calls, timers, subscriptions, DOM updates.

```jsx id="b2r7yk"
import { useEffect } from 'react';

function App() {
  useEffect(() => {
    console.log('Component mounted');
  }, []);

  return <h1>Hello</h1>;
}
```

### Dependency Rules

Runs once:

```jsx id="u4w9ge"
useEffect(() => {}, []);
```

Runs on `count` change:

```jsx id="n8m5qa"
useEffect(() => {}, [count]);
```

Runs every render:

```jsx id="x3c1lo"
useEffect(() => {});
```

### Cleanup Example

```jsx id="g7k2wp"
useEffect(() => {
  const id = setInterval(() => {}, 1000);

  return () => clearInterval(id);
}, []);
```

---

## 3. `useRef()` → Mutable Value / DOM Access

Stores value without re-rendering.

```jsx id="p6f3mz"
import { useRef } from 'react';

function InputFocus() {
  const inputRef = useRef();

  return (
    <>
      <input ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>Focus</button>
    </>
  );
}
```

### Use Cases

- Focus input
- Scroll to element
- Store timer IDs
- Previous values

---

## 4. `useMemo()` → Cache Expensive Calculations

Prevents recalculating unless dependencies change.

```jsx id="s8j4tw"
import { useMemo } from 'react';

function App({ users }) {
  const activeUsers = useMemo(() => {
    return users.filter((u) => u.active);
  }, [users]);

  return <div>{activeUsers.length}</div>;
}
```

### Use When:

- Large filtering
- Sorting
- Heavy calculations

---

## 5. `useCallback()` → Cache Function Reference

Useful when passing functions to memoized children.

```jsx id="q4n7be"
const handleDelete = useCallback((id) => {
  console.log(id);
}, []);
```

### Use When:

- Props to `React.memo` child
- Prevent unnecessary re-renders

---

## 6. `useContext()` → Shared Data

Avoid props drilling.

```jsx id="r1v5ko"
import { createContext, useContext } from 'react';

const ThemeContext = createContext();

function App() {
  return (
    <ThemeContext.Provider value='dark'>
      <Navbar />
    </ThemeContext.Provider>
  );
}

function Navbar() {
  const theme = useContext(ThemeContext);
  return <h1>{theme}</h1>;
}
```

### Use Cases

- Theme
- Auth user
- Language

---

## 7. `useReducer()` → Complex State Logic

Best when many related updates.

```jsx id="h6z2qm"
import { useReducer } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'inc':
      return { count: state.count + 1 };
    case 'dec':
      return { count: state.count - 1 };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, {
    count: 0,
  });

  return (
    <>
      <h1>{state.count}</h1>
      <button onClick={() => dispatch({ type: 'inc' })}>+</button>
    </>
  );
}
```

### Use Cases

- Complex forms
- Multi-step state
- State machines

---

# Other Important Built-in Hooks

## 8. `useLayoutEffect()`

Like `useEffect`, but runs **before browser paint**.

Use for DOM measurements.

```jsx id="m9k3wp"
useLayoutEffect(() => {
  console.log('Before paint');
}, []);
```

---

## 9. `useImperativeHandle()`

Used with `forwardRef` to expose custom methods.

```jsx id="t2x8yn"
useImperativeHandle(ref, () => ({
  focus: () => inputRef.current.focus(),
}));
```

Advanced use.

---

## 10. `useId()`

Generates unique IDs.

```jsx id="w7r4qe"
const id = useId();
<label htmlFor={id}>Name</label>
<input id={id} />
```

Great for accessibility.

---

## 11. `useTransition()`

For smoother UI updates.

```jsx id="y3f1ok"
const [isPending, startTransition] = useTransition();

startTransition(() => {
  setSearch(text);
});
```

Useful in search/filter UIs.

---

## 12. `useDeferredValue()`

Delay expensive rendering.

```jsx id="n1g6tv"
const deferredSearch = useDeferredValue(search);
```

---

## 13. `useSyncExternalStore()`

For external stores/library integration.

Advanced state libraries use this internally.

---

# Real Admin Panel Hook Usage

## `useState`

```text id="x8p3la"
modal open
selected row
search text
```

## `useEffect`

```text id="b4n2fd"
fetch on mount
listen resize
timer
```

## `useRef`

```text id="j5r7ky"
focus search input
scroll table
```

## `useMemo`

```text id="d6q9um"
filtered users
sorted products
dashboard totals
```

## `useContext`

```text id="v3h1ep"
theme
language
```

## `useReducer`

```text id="k7m5zs"
complex form wizard
```

---

# Golden Rule: When Not to Overuse

## Avoid `useMemo` everywhere

Only use for real expensive work.

## Avoid `useEffect` for everything

Many things belong directly in render or event handlers.

## Avoid `useContext` for rapidly changing large state

Use Zustand instead.

---

# Hooks Rules (Very Important)

✅ Call hooks at top level
✅ Call hooks inside React components or custom hooks only

❌ No hooks in loops
❌ No hooks inside conditions

Bad:

```jsx id="f2u7co"
if (show) {
  useState();
}
```

---

# Easy Memory Trick

```text id="s4r8yn"
useState = memory
useEffect = side effects
useRef = box
useMemo = cache
useCallback = cached function
useContext = shared data
useReducer = state machine
```

---

# Interview Answer

“Hooks are built-in React functions that allow functional components to manage state, lifecycle effects, refs, context, reducers, and performance optimization.”

---

# My Recommended Priority for You

Learn deeply in this order:

1. `useState`
2. `useEffect`
3. `useRef`
4. `useContext`
5. `useMemo`
6. `useCallback`
7. `useReducer`

---

# Want Next Level?

I can also teach you **how these hooks work internally inside React Fiber**, especially `useState` and `useEffect`, which is excellent for senior interviews.

# Custom hooks

In React, **custom hooks** are reusable functions that use built-in hooks (`useState`, `useEffect`, etc.) to share logic across components.

In real projects, custom hooks are used heavily to avoid duplicated code.

# Top Most Used Custom Hooks (Industry Standard)

---

# 1. `useFetch()` / `useApi()`

Used for API requests.

```jsx id="m1a7qp"
function useFetch(url) {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch(url)
      .then((res) => res.json())
      .then(setData);
  }, [url]);

  return data;
}
```

Use:

```jsx id="j4w8cz"
const users = useFetch('/api/users');
```

### Real Use

Today often replaced by TanStack Query, but still common.

---

# 2. `useDebounce()`

Very common for search inputs.

```jsx id="k2n5xe"
function useDebounce(value, delay = 500) {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const id = setTimeout(() => {
      setDebounced(value);
    }, delay);

    return () => clearTimeout(id);
  }, [value, delay]);

  return debounced;
}
```

Use:

```jsx id="p8r3mt"
const debouncedSearch = useDebounce(search, 400);
```

### Use For

- Search bars
- Filters
- Live typing requests

---

# 3. `useToggle()`

Simple boolean logic.

```jsx id="v3k7az"
function useToggle(initial = false) {
  const [value, setValue] = useState(initial);

  const toggle = () => setValue((v) => !v);

  return [value, toggle];
}
```

Use:

```jsx id="x9f2qy"
const [open, toggle] = useToggle();
```

### Use For

- Modals
- Sidebar
- Dropdowns

---

# 4. `useLocalStorage()`

Persist state in browser storage.

```jsx id="n4m8jw"
function useLocalStorage(key, initial) {
  const [value, setValue] = useState(
    () => JSON.parse(localStorage.getItem(key)) || initial
  );

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}
```

Use:

```jsx id="t6p1le"
const [theme, setTheme] = useLocalStorage('theme', 'dark');
```

### Use For

- Theme
- Preferences
- Saved drafts

---

# 5. `usePrevious()`

Get previous value.

```jsx id="d7q5rb"
function usePrevious(value) {
  const ref = useRef();

  useEffect(() => {
    ref.current = value;
  });

  return ref.current;
}
```

Use:

```jsx id="f1h8vc"
const prevCount = usePrevious(count);
```

---

# 6. `useWindowSize()`

Responsive UI hook.

```jsx id="g3k9nz"
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const onResize = () =>
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });

    window.addEventListener('resize', onResize);

    return () => window.removeEventListener('resize', onResize);
  }, []);

  return size;
}
```

Use:

```jsx id="r8y2op"
const { width } = useWindowSize();
```

---

# 7. `useAuth()`

Very common in admin panels.

```jsx id="u5j1wa"
function useAuth() {
  const user = useAuthStore((s) => s.user);

  return {
    user,
    isLoggedIn: !!user,
  };
}
```

Use:

```jsx id="c9n6xt"
const { user, isLoggedIn } = useAuth();
```

---

# 8. `usePagination()`

For tables and lists.

```jsx id="q2m4hz"
function usePagination() {
  const [page, setPage] = useState(1);

  return { page, setPage };
}
```

Use:

```jsx id="b7p8ve"
const { page, setPage } = usePagination();
```

---

# 9. `useModal()`

Reusable modal logic.

```jsx id="l6r3ty"
function useModal() {
  const [open, setOpen] = useState(false);

  return {
    open,
    openModal: () => setOpen(true),
    closeModal: () => setOpen(false),
  };
}
```

---

# 10. `useClickOutside()`

Close dropdown/modal when clicking outside.

```jsx id="z1w7kn"
function useClickOutside(ref, callback) {
  useEffect(() => {
    const handler = (e) => {
      if (!ref.current.contains(e.target)) {
        callback();
      }
    };

    document.addEventListener('mousedown', handler);

    return () => document.removeEventListener('mousedown', handler);
  }, [ref, callback]);
}
```

---

# 11. `useInterval()`

Timers without messy setup.

```jsx id="h4x9cu"
function useInterval(callback, delay) {
  useEffect(() => {
    const id = setInterval(callback, delay);
    return () => clearInterval(id);
  }, [callback, delay]);
}
```

---

# 12. `useForm()` (Custom Wrapper)

Many teams wrap React Hook Form logic into custom hooks.

```jsx id="s3j6ql"
function useUserForm() {
  return useForm({
    defaultValues: { name: '', email: '' },
  });
}
```

---

# Most Used in Enterprise Admin Panels

## Must Have

```text id="m8p2rw"
useDebounce
useAuth
usePagination
useModal
useLocalStorage
useClickOutside
```

## Good To Have

```text id="n7v5ka"
usePrevious
useWindowSize
useToggle
useInterval
```

---

# My Recommended Hooks for Your React Admin Panel

```bash id="x6q1od"
src/hooks/
│── useDebounce.js
│── useAuth.js
│── usePagination.js
│── useModal.js
│── useLocalStorage.js
│── useClickOutside.js
│── usePermission.js
│── useQueryParams.js
```

---

# Interview Answer

“The most commonly used custom hooks are `useDebounce`, `useAuth`, `useLocalStorage`, `usePagination`, `useModal`, and `useClickOutside`, because they encapsulate reusable stateful logic.”

---

# Golden Rule

If you copy the same hook logic twice → make a custom hook.

---

# Want Next Level?

I can also give you **Top 15 custom hooks with production-ready source code for React + Vite admin panel**.
