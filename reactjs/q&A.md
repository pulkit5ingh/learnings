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
A: An independent, reusable piece of UI: a JavaScript function that takes props and returns JSX describing what should appear on screen. Components are the building blocks of every React UI, as small as a single button or as large as an entire page, and an app is a tree of them (`App` → `Page` → `Button`). A component should be **pure**: the same props and state always give the same output, with no side effects during render.

```jsx
function Avatar({ user, size = 48 }) {
  return <img src={user.avatarUrl} alt={user.name} width={size} height={size} />;
}

function ProfileCard({ user }) {
  return (
    <div className="card">
      <Avatar user={user} />        {/* components compose into a tree */}
      <h2>{user.name}</h2>
    </div>
  );
}
```

Interview-ready answer: *"A component is a reusable piece of UI. In modern React it's a function that takes props and returns JSX. Components should be pure and small, and you build the app by composing them into a tree."*

**Q: What is JSX and what does it compile to?**
A: JSX is a syntax extension that lets you write HTML-like markup inside JavaScript. Browsers can't run it: a compiler (Babel, SWC, esbuild) turns each tag into a function call that returns a plain object, a **React element**. Since React 17, the "automatic runtime" compiles to `jsx()` from `react/jsx-runtime`, so you no longer need `import React` in every file. The classic runtime compiles to `React.createElement()`.

```jsx
// You write
const el = <h1 className="title">Hello, {name}</h1>;

// Classic runtime compiles to
const el = React.createElement('h1', { className: 'title' }, 'Hello, ', name);

// Automatic runtime (React 17+) compiles to
import { jsx as _jsx } from 'react/jsx-runtime';
const el = _jsx('h1', { className: 'title', children: ['Hello, ', name] });

// Either way, the result is a plain object
// { type: 'h1', props: { className: 'title', children: ['Hello, ', 'Pulkit'] }, key: null, ... }
```

Interview-ready answer: *"JSX is syntax sugar for creating React elements. The compiler turns each tag into a `jsx()` or `React.createElement()` call that returns a plain object describing the UI. JSX expressions are just JavaScript, so you can use variables, `map` and conditions inside them."*

**Q: Why must a component name start with a capital letter?**
A: The JSX compiler uses the case to decide what the tag is. A **lowercase** tag becomes a **string**, treated as a built-in HTML element. A **capitalised** tag becomes a **variable reference**, treated as your component.

```jsx
<button />  // → jsx('button', ...)   the string 'button': an HTML element
<Button />  // → jsx(Button, ...)     the Button variable: your component

function button() { return <b>hi</b>; }
<button />  // ❌ renders an empty HTML <button>, not your component
```

Interview-ready answer: *"JSX treats lowercase tags as HTML elements (passed as strings) and capitalised tags as component references. If you name a component in lowercase, React renders an unknown HTML element instead of calling your function."*

**Q: What are the main JSX rules?**
A:
1. **Return a single root element.** Wrap siblings in a parent element or a fragment `<>...</>`.
2. **Close every tag**, including void elements: `<br />`, `<img />`, `<input />`.
3. **Attributes are camelCase:** `className` (not `class`), `htmlFor` (not `for`), `onClick`, `tabIndex`. `aria-*` and `data-*` keep their dashes.
4. **JavaScript goes inside `{}`.** These must be expressions, not statements: no `if` or `for` inside JSX, so use ternaries, `&&` and `.map()`.
5. **`style` takes an object** with camelCase properties.
6. **Comments** are `{/* ... */}`.

```jsx
function Profile({ user, isOnline }) {
  return (
    <>                                                     {/* 1. one root */}
      <label htmlFor="bio">Bio</label>                     {/* 3. htmlFor */}
      <input id="bio" className="input" />                 {/* 2. self-closed, className */}
      <img src={user.avatar} alt={user.name} />           {/* 4. expressions in {} */}
      <p style={{ color: isOnline ? 'green' : 'gray', fontSize: 14 }}>  {/* 5. style object */}
        {isOnline ? 'Online' : 'Offline'}
      </p>
    </>
  );
}
```

Interview-ready answer: *"One root element (or a fragment), every tag closed, camelCase attributes like className and htmlFor, JavaScript expressions in curly braces, and style as an object. Because JSX compiles to function calls, it only accepts expressions, so we use ternaries and map instead of if and for."*

**Q: Why use a fragment instead of a `<div>`?**
A: A fragment groups children **without adding a DOM node**. Extra `<div>`s can break CSS layouts (flexbox and grid children), produce **invalid HTML** (a `<div>` inside `<tr>` or `<ul>`), and bloat the DOM. Use the long form `<Fragment key={...}>` when you need a key, for example inside `.map()`.

```jsx
// ❌ A <div> inside <tr> is invalid HTML and breaks the table
function Columns() {
  return <div><td>Name</td><td>Age</td></div>;
}

// ✅ A fragment adds nothing to the DOM
function Columns() {
  return <><td>Name</td><td>Age</td></>;
}

// A keyed fragment in a list
import { Fragment } from 'react';
items.map((item) => (
  <Fragment key={item.id}>
    <dt>{item.term}</dt>
    <dd>{item.description}</dd>
  </Fragment>
));
```

Interview-ready answer: *"A component must return one root, and a fragment satisfies that without adding a wrapper element. That keeps the DOM clean and avoids breaking layouts or creating invalid HTML, such as a div inside a table row. If the fragment needs a key in a list, I use the explicit `<Fragment key>` form."*

**Q: How do you do conditional rendering in React?**
A: With plain JavaScript; there's no `v-if` or `*ngIf`. Common patterns:

```jsx
function Dashboard({ user, isLoading, error, notifications }) {
  if (isLoading) return <Spinner />;                    // 1. early return
  if (error) return <ErrorBanner message={error} />;

  return (
    <div>
      {user.isAdmin ? <AdminPanel /> : <UserPanel />}   {/* 2. ternary: either/or */}

      {notifications.length > 0 && (                    /* 3. &&: show or nothing */
        <Badge count={notifications.length} />
      )}

      {{ admin: <AdminMenu />, user: <UserMenu /> }[user.role] ?? null}  {/* 4. lookup object for many cases */}
    </div>
  );
}
```

**The `&&` gotcha:** `{count && <Badge />}` renders a literal **`0`** when `count` is `0`, because `0` is falsy but React still renders numbers. Use a real boolean: `{count > 0 && <Badge />}`, or a ternary. Returning `null` renders nothing.

Interview-ready answer: *"Conditional rendering in React is plain JavaScript: an early return for loading and error states, a ternary for either/or, `&&` for show-or-nothing, and a lookup object or switch when there are many cases. The one trap is `&&` with a number: zero gets rendered, so I make the condition an explicit boolean."*

**Q: What is React.js, and how does it work?**
A: React is an open-source JavaScript library from Meta for building user interfaces out of components. You describe *what* the UI should look like for a given state, and React works out *how* to update the DOM. When state or props change, React re-renders the affected components into a new Virtual DOM tree. It then compares that tree with the previous one (reconciliation) and applies only the minimal set of changes to the real DOM.

```jsx
import { createRoot } from 'react-dom/client';
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);              // state
  return <button onClick={() => setCount(count + 1)}>Clicked {count} times</button>;
}

createRoot(document.getElementById('root')).render(<Counter />);
// Click → setCount → React re-renders Counter → diffs the old and new trees →
// updates only the button's text node in the real DOM
```

Interview-ready answer: *"React is a JavaScript library for building UIs from components. You write declarative components that describe the UI for the current state. When state or props change, React re-renders those components into a new virtual tree, diffs it against the previous one, and applies only the minimal changes to the real DOM."*

**Q: What are React's key features?**
A:
- **Component-based:** the UI is split into reusable, self-contained pieces.
- **Declarative:** you describe the UI for each state, not the DOM steps.
- **JSX:** HTML-like syntax inside JavaScript.
- **Virtual DOM and reconciliation:** keep DOM updates efficient.
- **One-way data flow:** data moves down through props, which keeps the UI predictable.
- **Hooks:** state and lifecycle in function components.
- **Ecosystem and platforms:** a large library ecosystem, React Native for mobile, and server rendering through Next.js and React Server Components.

```jsx
// Declarative: describe the result for each state, and React handles the DOM steps
function TodoList({ todos }) {
  return todos.length === 0
    ? <p>Nothing to do 🎉</p>
    : <ul>{todos.map((t) => <li key={t.id}>{t.title}</li>)}</ul>;
}
// Imperative (vanilla JS) would mean: find the ul, clear it, createElement('li') for each item, append...
```

Interview-ready answer: *"Its key features are a component-based architecture, a declarative style where I describe the UI for each state, JSX, an efficient update model through the virtual DOM and reconciliation, one-way data flow, and hooks for state and side effects. It also has a huge ecosystem and runs beyond the browser, with React Native and server rendering through Next.js."*

**Q: Why is React called a library rather than a framework?**
A: React only handles the view layer, meaning it renders UI and manages component state. It ships no built-in router, HTTP client, form handling, global state store or project structure. You pick those yourself (React Router, React Query, Redux, and so on). A framework such as Angular gives you all of these and controls the application flow (inversion of control). With React, your code calls the library. Frameworks like Next.js are built *on top of* React.

```jsx
// A typical React stack: you choose each piece
import { BrowserRouter, Routes, Route } from 'react-router-dom';   // routing
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'; // data fetching
import { useForm } from 'react-hook-form';                          // forms
// Angular ships its own router, HttpClient, forms and DI built in
```

Interview-ready answer: *"React is a library because it only handles the view layer: rendering UI and managing component state. Routing, data fetching, forms and global state come from libraries I choose. A framework like Angular provides all of these and dictates the structure. That's the difference: I call React, whereas a framework calls my code. Next.js is a framework built on React that adds routing, server rendering and a build system."*

**Q: What is the Virtual DOM?**
A: A lightweight JavaScript object representation of the real DOM. JSX compiles to `React.createElement()` calls, which return plain objects describing elements (`{ type, props, children }`). React keeps this tree in memory. On each update it builds a new tree, diffs it against the old one, and touches the real DOM only where something changed.

```jsx
// This JSX...
<div className="card"><h2>{title}</h2><p>{body}</p></div>

// ...is just a JavaScript object (simplified)
{
  type: 'div',
  props: {
    className: 'card',
    children: [
      { type: 'h2', props: { children: 'Hello' } },
      { type: 'p',  props: { children: 'World' } },
    ],
  },
}
// If only `title` changes, the diff finds a single text change, and React updates
// one text node instead of rebuilding the card
```

Interview-ready answer: *"The Virtual DOM is an in-memory tree of plain JavaScript objects describing the UI. On every update React builds a new tree, compares it with the previous one, and applies only the differences to the real DOM. That batches and minimises the expensive DOM work while letting us write simple declarative code."*

**Q: What is the difference between the Virtual DOM and the Real DOM?**
A:

| | Real DOM | Virtual DOM |
|---|---|---|
| What it is | The browser's actual document tree | A JS object copy kept by React |
| Updating | Slow: can trigger layout/reflow and repaint | Fast: it's just creating JS objects |
| Updates applied | Directly, every change | Batched and diffed, then minimal changes applied |
| Can render to screen | Yes | No, it must be synced to the real DOM |
| Memory | Heavier | Lighter |

The Virtual DOM isn't "faster than the DOM" in itself. What it gives you is a way to write declarative code without updating the DOM by hand, while avoiding unnecessary DOM operations.

```js
// Real DOM, done by hand: easy to get wrong, and each write can trigger layout work
const list = document.querySelector('#list');
list.innerHTML = '';                           // throws away every node, even unchanged ones
items.forEach((i) => {
  const li = document.createElement('li');
  li.textContent = i.name;
  list.appendChild(li);
});

// React: declare the result, and the diff touches only the items that changed
<ul id="list">{items.map((i) => <li key={i.id}>{i.name}</li>)}</ul>
```

Interview-ready answer: *"The real DOM is the browser's actual document, and changing it can trigger expensive layout and repaint. The Virtual DOM is React's lightweight in-memory copy. React diffs virtual trees and batches the minimal set of real DOM updates. The benefit isn't raw speed so much as letting me write declarative UI without hand-managing DOM updates, while still keeping those updates efficient."*

**Q: How does React's reconciliation process work?**
A: Reconciliation is the diffing algorithm React uses to compare the new element tree with the previous one. A perfect tree diff costs O(n³), so React uses two heuristics that bring it down to O(n):
1. **Different element types produce different trees.** If `<div>` becomes `<span>` (or `ComponentA` becomes `ComponentB`), React destroys the old subtree, including its state, and builds a new one.
2. **Keys identify list items.** Among siblings, React matches children by `key`, so reordering moves existing nodes instead of re-creating them.

When the type stays the same, React keeps the DOM node or component instance and updates only the changed attributes and props, then recurses into the children. The process has two phases: the **render phase**, which is pure, can be interrupted, and works out what changed; and the **commit phase**, which runs synchronously and applies the changes to the DOM, then runs layout effects and effects.

```jsx
// Same type: the DOM node is kept and only className is updated
<button className="primary" />  →  <button className="secondary" />

// Different type: the old subtree is destroyed (Counter's state is LOST), a new one is mounted
{isEditing ? <div><Counter /></div> : <section><Counter /></section>}

// Keys: React matches children by key, so prepending an item moves nodes
// instead of re-rendering every <li>
<ul>{items.map((item) => <li key={item.id}>{item.name}</li>)}</ul>
```

Interview-ready answer: *"Reconciliation is how React decides what changed between two renders. It uses two heuristics that make the diff O(n): elements of a different type produce entirely new subtrees, and keys identify list items across renders. If the type matches, React keeps the node and patches its props. That's also why changing an element's type or key resets a component's state."*

**Q: What is React Fiber?**
A: Fiber is React's reconciliation engine, rewritten in React 16. Each component instance becomes a "fiber", a unit of work in a linked tree. The old stack reconciler was recursive and synchronous. Fiber can **pause, resume, prioritise and abort** rendering work. This makes the following possible:
- Concurrent rendering (React 18): `startTransition` and `useDeferredValue`.
- Splitting rendering into chunks so long renders don't block user input.
- Priority lanes: typing is urgent, while a large list update can wait.
- Suspense and error boundaries.

Example: while a user types into a search box and a large list is re-rendering, React can pause the list work, handle the keystroke first, then resume. Senior-level point: Fiber is an implementation detail. You never touch fibers directly; you use it through APIs such as `startTransition`, `useDeferredValue` and Suspense.

```jsx
const [isPending, startTransition] = useTransition();

function handleChange(e) {
  setQuery(e.target.value);                                       // urgent: Fiber handles it first
  startTransition(() => setResults(filterHugeList(e.target.value))); // low priority: can be paused or abandoned
}
```

Interview-ready answer: *"Fiber is React's reconciliation engine since React 16. It breaks rendering into small units of work that React can pause, prioritise, resume or throw away, instead of one blocking recursive pass. That's what makes concurrent features like transitions, Suspense and keeping input responsive during heavy renders possible."*

**Q: What is the difference between rendering and committing in React?**
A: Every update goes through two phases:

| | Render phase | Commit phase |
|---|---|---|
| What happens | React calls your components, builds the new element tree and diffs it with the previous one | React applies the computed changes to the real DOM |
| Side effects allowed? | No, it must be pure | Yes |
| Interruptible? | Yes, in concurrent rendering (can pause, restart or be thrown away) | No, it runs synchronously in one go |
| Also does | Works out *what* changed | Attaches and detaches refs, runs `useLayoutEffect`; `useEffect` runs after paint |

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  console.log('Component rendered'); // render phase
  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}
```

Clicking schedules an update. React re-renders `Counter`, finds that only the button's text changed, and commits that single text update.

**Rendering does not mean the DOM changed.** A component can render and React can find nothing to update. It's also why render code may run more than once (StrictMode, interrupted renders), and why side effects belong in effects or event handlers, not in the component body.

Interview-ready answer: *"Rendering is React calling my components to work out what the UI should look like. It's pure and, in concurrent mode, interruptible. Committing is React applying the resulting changes to the real DOM, attaching refs and running layout effects, all synchronously; regular effects run after paint. A component can render without any DOM change being committed."*

**Q: Why are keys important, and why avoid array indexes as keys?**
A: Keys give each list item a stable **identity**, so React can match the old and new children during reconciliation: which were added, removed or moved.

```jsx
const users = [{ id: 1, name: 'Pulkit' }, { id: 2, name: 'Rahul' }];

<ul>
  {users.map((user) => <li key={user.id}>{user.name}</li>)}
</ul>
```

With index keys, when items are inserted at the top, removed or reordered, the indexes shift. React then associates existing component state, DOM nodes and input values with the **wrong item**. Typical symptoms are a typed value or checkbox jumping to another row, plus extra re-renders.

Best practices:
- Use unique, stable IDs (database IDs).
- Never use random values (`Math.random()`, `uuid()` during render). A new key every render remounts the item and loses its state.
- Index keys are acceptable only for static lists that never reorder, filter or change membership.
- Keys must be unique among siblings, not globally.
- Changing a component's `key` on purpose forces a remount, which is a useful way to reset its state (`<Form key={userId} />`).

Interview-ready answer: *"Keys give list items a stable identity, so React can match old and new children during reconciliation and move, add or remove only what changed. I use unique IDs from the data. Index keys break when the list is reordered, filtered or prepended, because state and DOM nodes attach to the wrong item; they're only acceptable for static lists. Random keys are worse, since they remount every item on every render."*

**Q: What is the difference between functional and class components? Are class components still used?**
A: Since React 16.8 (hooks), new code is written as functions. Class components still work but are legacy; the one thing that still needs a class is an Error Boundary (see A9.4).

| | Functional | Class |
|---|---|---|
| Syntax | Plain function returning JSX | `class X extends React.Component` with `render()` |
| State | `useState` / `useReducer` | `this.state` + `this.setState` |
| Lifecycle | `useEffect`, `useLayoutEffect` | `componentDidMount`, `componentDidUpdate`, `componentWillUnmount` |
| `this` | Not used | Needs `this` and method binding |
| Logic reuse | Custom hooks | HOCs and render props |
| Status | Recommended | Legacy (still needed for error boundaries) |

```jsx
// Functional
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}

// Class
class Counter extends React.Component {
  state = { count: 0 };
  render() {
    return (
      <button onClick={() => this.setState({ count: this.state.count + 1 })}>
        {this.state.count}
      </button>
    );
  }
}
```

Lifecycle mapping, which interviewers often ask about:

```jsx
useEffect(() => { /* componentDidMount */ return () => { /* componentWillUnmount */ }; }, []);
useEffect(() => { /* componentDidUpdate when `id` changes */ }, [id]);
const Memo = React.memo(Component); // ≈ shouldComponentUpdate / PureComponent
```

Interview-ready answer: *"Function components are plain functions that use hooks for state and side effects. Class components extend React.Component and use this.state and lifecycle methods. Since hooks arrived in 16.8, function components are the standard: less boilerplate, no `this` binding issues, and logic reuse through custom hooks. Classes are legacy, though Error Boundaries still require one."*

**Q: What are the major new features introduced in React 19?**
A:
1. **Actions:** async functions passed to `<form action={fn}>` or `startTransition`. React tracks their pending state, errors and form resets for you. (See A5.1.)
2. **`useActionState`:** manages an Action's result and pending state. (See A5.1.)
3. **`useFormStatus`:** lets a child component, such as a submit button, read the parent form's pending state without prop drilling.
4. **`useOptimistic`:** shows an optimistic UI immediately while the server request is in flight. (See A4.2.)
5. **`use()`:** reads a Promise (it suspends until resolved) or Context. Unlike other hooks, it can be called inside conditions.
6. **React Server Components and Server Actions:** components rendered on the server that ship no component JS to the browser. (See A9.4.)
7. **`ref` as a prop:** function components receive `ref` like any other prop, so `forwardRef` is no longer needed.
8. **Other improvements:**
   - `<Context>` can be used directly as a provider (no `.Provider`).
   - Ref callbacks can return a cleanup function.
   - Document metadata (`<title>`, `<meta>`) can be rendered from any component.
   - Stylesheet and script loading is integrated with Suspense.
   - Better hydration error messages.

```jsx
// React 19: no forwardRef
function TextInput({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}
```

Related: the **React Compiler**, a separate build-time tool, memoises components automatically, which reduces the need for manual `useMemo`, `useCallback` and `React.memo`.

Senior-level insight: React 19 removes a lot of hand-written loading, error and optimistic state for async UI. Adopt the features where they fit, but they don't replace server-side validation, proper error handling, or a server-state library like TanStack Query for complex caching.

### [A1.2 — Props & Composition](A1.2-props-and-composition.md)

**Q: What are props?**
A: Props ("properties") are the inputs a parent passes to a child, like arguments to a function. They can be any JS value: strings, numbers, objects, arrays, functions (callbacks) and even JSX. They are **read-only**: a component must never modify its own props. When a parent re-renders with new props, the child re-renders with them.

```jsx
function Greeting({ name, age, onWave }) {          // destructure props
  return (
    <p>
      Hello {name}, you are {age}
      <button onClick={() => onWave(name)}>👋</button>
    </p>
  );
}

<Greeting name="Pulkit" age={28} onWave={(n) => console.log(`${n} waved`)} />
// strings in quotes; everything else (numbers, objects, functions) in {}
```

Interview-ready answer: *"Props are the read-only inputs a parent passes to a child component, like function arguments. They can be data, callbacks or JSX. A component never mutates its props; when the parent passes new values, React re-renders the child."*

**Q: Why is one-way data flow useful?**
A: In React, data flows **down** from parent to child through props, and changes flow **up** only through callbacks that the parent provides. The state lives in one owner, and only that owner changes it. So when something is wrong on screen, you trace it to a **single source of truth**, instead of hunting through two-way bindings that could have changed it from anywhere.

```jsx
function Parent() {
  const [temp, setTemp] = useState(20);                        // single source of truth
  return (
    <>
      <Thermostat value={temp} onChange={setTemp} />          {/* data down, events up */}
      <Display value={temp} />                                 {/* always in sync */}
    </>
  );
}
```

Interview-ready answer: *"One-way data flow means state is owned in one place and passed down as props, while children request changes through callbacks. That makes the UI predictable and easy to debug, because I always know which component owns a piece of data and who can change it."*

**Q: How do you type props and give them defaults?**
A: Declare a TypeScript `type` or `interface` for the props, and give defaults with **default parameter values** while destructuring. (`defaultProps` on function components is deprecated in React 19.)

```tsx
type ButtonProps = {
  label: string;                                   // required
  variant?: 'primary' | 'secondary' | 'danger';    // optional union
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  onClick?: (e: React.MouseEvent<HTMLButtonElement>) => void;
};

function Button({ label, variant = 'primary', size = 'md', disabled = false, onClick }: ButtonProps) {
  return (
    <button className={`btn btn-${variant} btn-${size}`} disabled={disabled} onClick={onClick}>
      {label}
    </button>
  );
}

// Accept every native <button> attribute as well:
type Props = React.ComponentProps<'button'> & { variant?: 'primary' | 'secondary' };
```

Interview-ready answer: *"I define a TypeScript type for each component's props, using optional fields and union types for variants, and set defaults with default parameters in the destructuring. That gives autocomplete and compile-time errors when a prop is missing or wrong. To extend native elements I use `React.ComponentProps<'button'>`."*

**Q: What is the `children` prop?**
A: A special prop every component receives containing whatever JSX is nested between its opening and closing tags. Wrappers like `Card`, `Modal` and `Layout` use it to render content they don't know ahead of time. Its type is `React.ReactNode` (elements, strings, numbers, arrays, `null`, `undefined`).

```tsx
function Card({ title, children }: { title: string; children: React.ReactNode }) {
  return (
    <section className="card">
      <h2>{title}</h2>
      <div className="card-body">{children}</div>
    </section>
  );
}

<Card title="Profile">
  <Avatar user={user} />           {/* everything here arrives as `children` */}
  <p>{user.bio}</p>
</Card>
```

Interview-ready answer: *"`children` is the content nested between a component's tags. It lets me build generic wrappers like Card, Modal or Layout that control the frame while the caller decides what goes inside. It's the simplest form of composition, typed as `React.ReactNode`."*

**Q: Why does React prefer composition over inheritance?**
A: Instead of `SpecialButton extends Button`, you build a `Button` with a `variant` prop or a `ButtonGroup` that wraps buttons — combining components rather than extending classes. Composition uses props, `children` and named "slot" props to customise a component, so pieces stay small, independent and recombinable. Inheritance creates rigid hierarchies, where a change in the parent class ripples into every subclass. The React team has said they haven't found a use case where inheritance hierarchies beat composition.

```jsx
// ❌ Inheritance thinking
class DangerButton extends Button { /* override styles... */ }

// ✅ Composition: specialise through props
const DangerButton = (props) => <Button {...props} variant="danger" />;

// ✅ Composition with named slots
function Layout({ header, sidebar, children }) {
  return (
    <div className="layout">
      <header>{header}</header>
      <aside>{sidebar}</aside>
      <main>{children}</main>
    </div>
  );
}
<Layout header={<NavBar />} sidebar={<Filters />}>
  <ProductGrid />
</Layout>
```

Interview-ready answer: *"React favours composition: you build complex components by combining simple ones and customising them through props, children and slot props, rather than extending class hierarchies. That keeps components decoupled and flexible. A specialised component is just a general one rendered with specific props, and shared logic goes into custom hooks."*

**Q: What is the render props pattern?**
A: Passing a **function** as a prop. The component owns some logic or state, and calls the function to let the **caller decide what to render** with it. Before hooks it was the main way to share stateful logic. Today a **custom hook** usually replaces it, but it's still used for headless components and libraries (Formik's `<Field>`, React Router's old `<Route render>`, Downshift).

```jsx
function MouseTracker({ render }) {
  const [pos, setPos] = useState({ x: 0, y: 0 });
  return (
    <div onMouseMove={(e) => setPos({ x: e.clientX, y: e.clientY })} style={{ height: 300 }}>
      {render(pos)}                                   {/* the caller decides the UI */}
    </div>
  );
}

<MouseTracker render={({ x, y }) => <p>Mouse at {x}, {y}</p>} />

// The modern equivalent: a custom hook
function useMousePosition() { /* same state + listener */ return pos; }
```

Interview-ready answer: *"Render props means passing a function as a prop. The component manages some state or behaviour and calls that function to let the caller render whatever it wants with it. It was the main logic-sharing pattern before hooks. Now I'd usually write a custom hook instead, but the pattern is still useful for headless, flexible UI components."*

**Q: What are reusable components?**
A: Generic components that are configured through props instead of being hard-coded for one use, so the same component works in many places. Good reusable components:
- Accept data and behaviour through props (`label`, `variant`, `onClick`).
- Use `children` for flexible content.
- Hold no business logic or API calls, so they stay presentational.
- Have sensible defaults and typed props.

```jsx
function Button({ variant = 'primary', children, ...rest }) {
  return <button className={`btn btn-${variant}`} {...rest}>{children}</button>;
}

<Button onClick={save}>Save</Button>
<Button variant="danger" onClick={remove}>Delete</Button>
```

Interview-ready answer: *"A reusable component is configured through props rather than hard-coded for one screen. It takes data, variants and callbacks as props, uses children for flexible content, forwards native attributes with rest props, and contains no business logic or API calls. That way the same Button, Modal or Card works everywhere and stays consistent."*

**Q: What is the difference between props and state?**
A:

| | Props | State |
|---|---|---|
| Owner | Passed in by the parent | Owned by the component itself |
| Mutable? | Read-only inside the child | Changed through the setter (`setX`) |
| Purpose | Configure or customise a component | Data that changes over time (input, toggles, fetched data) |
| Triggers re-render? | Yes, when the parent passes new values | Yes, when the setter is called with a new value |

Put simply, props are a function's arguments and state is the function's memory.

```jsx
function Counter({ step = 1 }) {               // `step` is a prop, configured by the parent
  const [count, setCount] = useState(0);       // `count` is state, owned and changed here
  return <button onClick={() => setCount((c) => c + step)}>{count}</button>;
}
<Counter step={5} />
```

Interview-ready answer: *"Props are inputs passed in by the parent and are read-only inside the child. State is data a component owns and changes over time through its setter. Both trigger a re-render when they change. If a value is needed by several components, I lift it into the state of their common parent and pass it down as props."*

**Q: Can we modify props?**
A: No. Props are read-only, and React requires components to be pure with respect to their props. Mutating a prop (`props.user.name = 'x'`) won't trigger a re-render and corrupts the parent's data. If a child needs to change the data, the parent should own it as state and pass down a callback (for example `onChange`) for the child to call. To derive a modified value, copy it into a new variable or into local state.

```jsx
// ❌ Mutating a prop: no re-render, and the parent's data is corrupted
function UserCard({ user }) {
  user.name = user.name.toUpperCase();
  return <h2>{user.name}</h2>;
}

// ✅ Derive a new value during render
function UserCard({ user }) {
  const displayName = user.name.toUpperCase();
  return <h2>{displayName}</h2>;
}

// ✅ Ask the owner to change it
function NameInput({ name, onNameChange }) {
  return <input value={name} onChange={(e) => onNameChange(e.target.value)} />;
}
```

Interview-ready answer: *"No. Props are read-only, and components must be pure with respect to them. Mutating one doesn't re-render and silently corrupts the parent's data. To display a modified version I compute a new value during render. To actually change it, the parent owns it as state and passes a callback that the child calls."*

**Q: How does data flow from parent to child?**
A: Through props. The parent passes values as attributes, and the child reads them.

```jsx
function Parent() {
  const [user] = useState({ name: 'Asha' });
  return <Child user={user} />;
}
function Child({ user }) {
  return <p>Hello {user.name}</p>;
}
```

Interview-ready answer: *"Parent to child is through props: the parent passes values as JSX attributes, and the child reads them as read-only inputs. Whenever the parent re-renders with new values, the child re-renders with them. This one-way flow keeps data ownership clear."*

**Q: How does a child communicate with its parent?**
A: The parent passes a callback function as a prop, and the child calls it with data. Data still flows down; events flow up.

```jsx
function Parent() {
  const [value, setValue] = useState('');
  return <SearchBox onSearch={setValue} />;
}
function SearchBox({ onSearch }) {
  return <input onChange={(e) => onSearch(e.target.value)} />;
}
```

To share state between siblings, **lift the state up** to their closest common parent and pass it down to both.

```jsx
// Lifting state up: two siblings share one value through their parent
function Converter() {
  const [celsius, setCelsius] = useState(0);
  return (
    <>
      <CelsiusInput value={celsius} onChange={setCelsius} />
      <FahrenheitDisplay value={celsius * 9 / 5 + 32} />
    </>
  );
}
```

Interview-ready answer: *"Parent to child is props. Child to parent is a callback the parent passes down, which the child calls with data, so data flows down and events flow up. Siblings communicate by lifting the shared state to their closest common parent. For deeply nested or app-wide data I use Context or a state library to avoid prop drilling."*

### [A1.3 — useState & Events](A1.3-usestate-and-events.md)

**Q: What does `useState` return?**
A: An array of exactly two items, the **current value** and a **setter function**, which you destructure and name yourself. Calling the setter stores the new value and **schedules a re-render**. The setter's identity is stable across renders, so it never needs to be an effect dependency.

```jsx
const [count, setCount] = useState(0);                 // number
const [user, setUser] = useState(null);                // object or null
const [items, setItems] = useState(() => loadItems()); // lazy initialiser: runs only on the first render

setCount(5);                       // set a new value
setCount((prev) => prev + 1);      // or compute it from the previous value
```

Interview-ready answer: *"useState returns a pair: the current state value and a setter. Calling the setter replaces the value and schedules a re-render, and the setter's identity is stable. For an expensive initial value I pass a function so it only runs once."*

**Q: Are state updates synchronous?**
A: No. The setter **schedules** an update. Inside the current render, the state variable is a **snapshot** that never changes, so reading it right after calling the setter gives the **old** value. The new value shows up on the next render.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
    console.log(count);        // still 0: this render's snapshot
  }

  useEffect(() => {
    console.log('updated:', count);  // to react to the new value, use an effect (or compute it during render)
  }, [count]);

  return <button onClick={handleClick}>{count}</button>;
}
```

Interview-ready answer: *"State updates are asynchronous and batched. The setter schedules a re-render, and within the current render the state value is a fixed snapshot, so logging it right after setting it shows the old value. To act on the new value, I compute it directly, use the functional updater, or react to it in an effect."*

**Q: Why must you never mutate state directly?**
A: React decides whether to re-render by comparing the old and new state **by reference** (`Object.is`). Mutating an object or array keeps the **same reference**, so React sees "no change" and skips the re-render. It also breaks `React.memo`, `useMemo` and effect dependencies, and corrupts the previous-render snapshot. Always create a **new** object or array.

```jsx
// ❌ Mutation: same reference, so no re-render
items.push(newItem);       setItems(items);
user.name = 'Pulkit';      setUser(user);

// ✅ Immutable updates: a new reference every time
setItems([...items, newItem]);                                   // add
setItems(items.filter((i) => i.id !== id));                      // remove
setItems(items.map((i) => (i.id === id ? { ...i, done: true } : i))); // update one
setUser({ ...user, name: 'Pulkit' });                            // update a field
setUser({ ...user, address: { ...user.address, city: 'Delhi' } }); // nested: copy every level
```

Interview-ready answer: *"React detects changes by reference. If I mutate an array or object and pass the same reference to the setter, React thinks nothing changed and skips the re-render, and memoisation breaks too. So I always create new copies with spread, map and filter, or use Immer for deeply nested state."*

**Q: When should you use the functional update form?**
A: Whenever the next state **depends on the previous state**. `setCount(count + 1)` uses this render's snapshot, so multiple updates in one event, or updates from stale closures (timers, async callbacks), compute from an outdated value. `setCount(prev => prev + 1)` always receives the **latest** queued value.

```jsx
function handleTripleClick() {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
  // ❌ +1 in total: all three compute 0 + 1
}

function handleTripleClickFixed() {
  setCount((c) => c + 1);
  setCount((c) => c + 1);
  setCount((c) => c + 1);
  // ✅ +3: each updater gets the latest value
}

// Also essential in intervals and async callbacks (avoids stale closures)
useEffect(() => {
  const id = setInterval(() => setCount((c) => c + 1), 1000);
  return () => clearInterval(id);
}, []);
```

Interview-ready answer: *"Whenever the new state is computed from the old one. The updater function receives the latest pending value, so it stays correct when several updates are batched or when the code runs in a stale closure, such as an interval or an async callback."*

**Q: What is a SyntheticEvent?**
A: React's cross-browser wrapper around the native DOM event. It has the same interface (`target`, `currentTarget`, `preventDefault()`, `stopPropagation()`) but behaves consistently across browsers, and `e.nativeEvent` gives you the original. React uses **event delegation**: it attaches one listener per event type at the **root container** (since React 17; earlier versions used `document`), not on every element. Since React 17 events are also **no longer pooled**, so you can read `e` asynchronously without `e.persist()`.

```jsx
function SearchForm() {
  function handleSubmit(e) {
    e.preventDefault();                     // stop the full page reload
    const data = new FormData(e.currentTarget);
    console.log(data.get('q'));
  }
  function handleClick(e) {
    e.stopPropagation();                    // don't trigger the parent's onClick
    console.log(e.nativeEvent);             // the underlying browser event
  }
  return (
    <form onSubmit={handleSubmit}>
      <input name="q" />
      <button onClick={handleClick}>Search</button>
    </form>
  );
}
```

Note the event props: they are camelCase, and you pass a **function**, not a call. `onClick={handleClick}` is correct, while `onClick={handleClick()}` calls it during render.

Interview-ready answer: *"A SyntheticEvent is React's wrapper around the browser's native event. It gives a consistent API across browsers, with the original available as `nativeEvent`. React delegates events to the root container rather than attaching listeners to every node, and since React 17 events aren't pooled, so they can be used asynchronously."*

**Q: What are the common TypeScript event types?**
A:

| Event | Type | Typical handler |
|---|---|---|
| Input or textarea change | `React.ChangeEvent<HTMLInputElement>` / `<HTMLTextAreaElement>` | `onChange` |
| Select change | `React.ChangeEvent<HTMLSelectElement>` | `onChange` |
| Form submit | `React.FormEvent<HTMLFormElement>` | `onSubmit` |
| Click | `React.MouseEvent<HTMLButtonElement>` | `onClick` |
| Key press | `React.KeyboardEvent<HTMLInputElement>` | `onKeyDown` |
| Focus or blur | `React.FocusEvent<HTMLInputElement>` | `onFocus`, `onBlur` |

```tsx
function LoginForm() {
  const [email, setEmail] = useState('');

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => setEmail(e.target.value);
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    login(email);
  };
  const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Escape') setEmail('');
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={email} onChange={handleChange} onKeyDown={handleKeyDown} />
    </form>
  );
}
// Typing a handler prop: onClick?: React.MouseEventHandler<HTMLButtonElement>
```

Interview-ready answer: *"React's event types are generic over the element: `ChangeEvent<HTMLInputElement>` for inputs, `FormEvent<HTMLFormElement>` for submit, `MouseEvent<HTMLButtonElement>` for clicks, `KeyboardEvent` for keys. When the handler is written inline, TypeScript infers the type, so I only annotate handlers declared separately, or props, using types like `MouseEventHandler`."*

**Q: What is the difference between controlled and uncontrolled inputs?**
A: A controlled input's `value` is driven by React state and `onChange` updates that state, giving React full control. An uncontrolled input is read from the DOM via a ref (the approach React Hook Form uses).

```jsx
// Controlled: React state is the source of truth
const [name, setName] = useState('');
<input value={name} onChange={(e) => setName(e.target.value)} />

// Uncontrolled: the DOM is the source of truth
const inputRef = useRef(null);
<input defaultValue="" ref={inputRef} />
// read it when needed: inputRef.current.value
```

Use controlled inputs for instant validation, formatting, or enabling and disabling buttons based on what the user typed. Use uncontrolled inputs for simple forms, file inputs (`<input type="file">` is always uncontrolled), or for performance in large forms.

Interview-ready answer: *"In a controlled input, React state is the source of truth: `value` comes from state and `onChange` updates it, which makes validation, formatting and conditional UI easy. In an uncontrolled input, the DOM keeps the value and I read it through a ref or FormData when needed, which means less code and fewer re-renders. File inputs are always uncontrolled. For large forms I use React Hook Form, which is uncontrolled under the hood."*

**Q: How does `useState()` work?**
A: `const [value, setValue] = useState(initialValue)` declares a state variable. React stores the value on the component's fiber, in a list of hooks ordered by call order. That ordering is why hooks must always be called in the same order. On the first render React uses `initialValue`; on later renders it returns the stored value. Notes:
- Lazy initialisation: `useState(() => expensiveCompute())` runs the function only on the first render.
- The setter accepts a value or an updater function: `setCount(c => c + 1)`.
- React compares the new value with the old one using `Object.is`. If they are the same, React skips the re-render.

```jsx
function Form() {
  const [name, setName] = useState('');     // hook #1
  const [age, setAge] = useState(0);        // hook #2: React tracks them by call order
  const [rows] = useState(() => parseHugeCsv(csv)); // lazy: parses once, not on every render

  setName('Pulkit'); // ❌ never call a setter during render (infinite loop); do it in handlers or effects
}
```

Interview-ready answer: *"useState stores the value in React, on the component's fiber, not in the function, which is why it survives re-renders even though the function runs again. Hooks are tracked by call order, so they must run unconditionally at the top level. The setter queues an update. React skips the re-render if the new value is `Object.is`-equal to the old one, and I use lazy initialisation for expensive initial values."*

**Q: What happens when state changes?**
A:
1. Calling the setter queues an update. It does not change the variable immediately, because the current render's value is a snapshot.
2. React **batches** multiple updates. Since React 18 this includes updates inside promises, timeouts and native events.
3. React re-renders that component and, by default, all of its children.
4. It builds a new Virtual DOM tree and reconciles it with the previous one.
5. In the commit phase it applies only the changed parts to the real DOM.
6. Effects whose dependencies changed run after the browser paints.

```jsx
function App() {
  const [count, setCount] = useState(0);
  console.log('App rendered');               // runs again on each click
  return (
    <>
      <button onClick={() => setCount(count + 1)}>{count}</button>
      <Header />                             {/* re-renders too, even with no props (unless memoised) */}
    </>
  );
}
```

Interview-ready answer: *"Calling a setter queues an update. React batches updates, re-renders the component and its children, diffs the new virtual tree against the old one, commits only the changed DOM nodes, and then runs any effects whose dependencies changed. The state variable itself doesn't change in the current render; the new value appears in the next one."*

**Q: What is automatic batching in React 18?**
A: Batching means React groups several state updates into **one re-render**. Before React 18, it batched only inside React event handlers; updates in promises, `setTimeout` or native listeners each caused a separate render. With React 18's `createRoot`, batching is **automatic everywhere**: event handlers, promises, `setTimeout`/`setInterval` and native event handlers.

```jsx
function handleClick() {
  setCount((c) => c + 1);
  setName('Pulkit');
  // one re-render, not two
}

fetch('/api').then(() => {
  setLoading(false);
  setData(result);
  // React 17: two renders. React 18: one.
});
```

Senior-level points:
- It is enabled by default only with the `createRoot` API. The legacy `ReactDOM.render` keeps the old behaviour.
- To force an immediate, separate render (rare, for example to read the DOM right after an update), wrap the update in `flushSync(() => setX(...))` from `react-dom`.

Interview-ready answer: *"Batching means React groups multiple state updates into a single re-render. Before React 18 that only happened inside React event handlers. With createRoot in React 18 it's automatic everywhere: in promises, timeouts and native listeners too. That reduces wasted renders. If I need an update applied immediately, there's flushSync."*

### [A1.4 — useEffect](A1.4-useeffect.md)

**Q: What is a side effect?**
A: Anything a component does **beyond computing its JSX**, anything that touches the world outside the render: fetching data, timers, subscriptions and WebSockets, manually changing the DOM, `document.title`, `localStorage`, analytics and logging. Rendering must be **pure** (same inputs give the same output, with no outside changes) because React may render a component many times, abandon renders, or render twice in StrictMode. Side effects therefore belong in **event handlers**, for effects caused by a user action, or in **`useEffect`**, for effects caused by rendering or syncing with an external system.

```jsx
function Profile({ userId }) {
  // ❌ A side effect during render: runs on every render, can loop forever
  // fetch(`/api/users/${userId}`).then(r => r.json()).then(setUser);

  const [user, setUser] = useState(null);

  useEffect(() => {                                    // ✅ syncing with an external system
    fetch(`/api/users/${userId}`).then((r) => r.json()).then(setUser);
  }, [userId]);

  useEffect(() => {
    document.title = user ? `${user.name}'s profile` : 'Loading…';
  }, [user]);

  const handleFollow = () => api.follow(userId);       // ✅ caused by a user action, so an event handler

  return <button onClick={handleFollow}>Follow</button>;
}
```

Interview-ready answer: *"A side effect is anything beyond calculating the UI: data fetching, subscriptions, timers, DOM manipulation, storage, logging. React requires rendering to be pure, so effects go either in event handlers, when a user action causes them, or in useEffect, when they're about keeping the component in sync with something external after render."*

**Q: What is `useEffect`, when does it run, and how does its dependency array work?**
A: `useEffect(setup, deps?)` runs side effects after the component commits, usually after the browser paints. Before re-running, React compares each dependency with its previous value using `Object.is`:

| Dependency array | When the effect runs |
|---|---|
| None: `useEffect(fn)` | After **every** render |
| Empty: `useEffect(fn, [])` | Once, after the first render (mount); cleanup on unmount |
| `[a, b]` | After mount, then whenever `a` or `b` changes |

List every reactive value (props, state, or values derived from them) that the effect uses; the `react-hooks/exhaustive-deps` lint rule enforces this. Objects and functions created during render are new on every render, so as dependencies they make the effect run every time. Move them inside the effect, or stabilise them with `useMemo`/`useCallback`. Never silence the lint warning without understanding it; a missing dependency is the usual cause of stale closures (see the counter scenario below).

```jsx
useEffect(() => { console.log('every render'); });                  // no array
useEffect(() => { console.log('mount only'); }, []);               // []
useEffect(() => { fetchUser(userId); }, [userId]);                  // when userId changes

// ❌ A new object every render, so the effect runs every render
const options = { userId, includePosts: true };
useEffect(() => { fetchUser(options); }, [options]);

// ✅ Depend on primitives, and build the object inside the effect
useEffect(() => {
  fetchUser({ userId, includePosts: true });
}, [userId]);
```

Interview-ready answer: *"useEffect runs after React commits a render, usually after paint, to sync the component with something external. The dependency array decides when it re-runs: none means every render, an empty array means only on mount, and listed values mean whenever one of them changes by `Object.is`. I list every reactive value the effect reads, as the lint rule requires, and avoid objects or functions recreated each render as dependencies."*

**Q: Why can't you pass an `async` function to `useEffect`?**
A: An `async` function always returns a **Promise**, but React expects the effect to return either nothing or a **cleanup function**. React would treat the Promise as the cleanup, which breaks cleanup and triggers a warning. Define the async function **inside** the effect and call it, and use an `AbortController` or an `ignore` flag so a late response can't update state after the effect has been cleaned up.

```jsx
// ❌ The effect returns a Promise
useEffect(async () => {
  const res = await fetch(`/api/users/${id}`);
  setUser(await res.json());
}, [id]);

// ✅ An async function inside the effect, with cancellation
useEffect(() => {
  const controller = new AbortController();

  async function loadUser() {
    try {
      const res = await fetch(`/api/users/${id}`, { signal: controller.signal });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      setUser(await res.json());
    } catch (err) {
      if (err.name !== 'AbortError') setError(err);
    }
  }

  loadUser();
  return () => controller.abort();   // a real cleanup function
}, [id]);
```

Interview-ready answer: *"Because an async function returns a Promise, and React expects an effect to return either undefined or a cleanup function. So I define an async function inside the effect and call it, and return a cleanup that aborts the request so stale responses can't set state. In real apps I'd usually use React Query instead of hand-written fetch effects."*

**Q: What is the cleanup function, and what goes wrong without it?**
A: The function returned from an effect. React runs it **before the effect re-runs**, with the old values, and **when the component unmounts**. Use it to undo whatever the effect set up: remove listeners, clear timers, unsubscribe, close sockets, abort fetches. Without cleanup, event listeners pile up, intervals run forever, and stale requests overwrite newer data (see memory leaks below).

```jsx
useEffect(() => {
  const id = setInterval(tick, 1000);
  window.addEventListener('resize', onResize);
  return () => {
    clearInterval(id);
    window.removeEventListener('resize', onResize);
  };
}, []);
```

In development, StrictMode deliberately runs mount, cleanup, then mount again, to surface missing cleanups.

Interview-ready answer: *"The cleanup is the function an effect returns. React calls it before re-running the effect with new dependencies and when the component unmounts. I use it to undo whatever the effect set up: remove listeners, clear timers, unsubscribe and abort requests. Without it, you get duplicate listeners, timers running forever, memory leaks and stale responses overwriting new data."*

**Q: What is the difference between `useEffect` and `useLayoutEffect`?**
A:

| | `useEffect` | `useLayoutEffect` |
|---|---|---|
| Timing | After the browser **paints** (asynchronous) | After DOM changes but **before paint** (synchronous) |
| Blocks painting? | No | Yes |
| Use for | Data fetching, subscriptions, logging, timers | Measuring the DOM (size, position) and changing layout before the user sees it, to avoid flicker |

Default to `useEffect`. Use `useLayoutEffect` only when you see a visual flicker, for example positioning a tooltip based on an element's measured size. Because `useLayoutEffect` runs synchronously before paint, slow work inside it delays what the user sees. It also doesn't run during server rendering, and React warns when it is used in SSR.

```jsx
useEffect(() => { console.log('effect: after paint'); }, []);
useLayoutEffect(() => { console.log('layout effect: before paint'); }, []);
// Logs "layout effect" first, then "effect"

// A real use: position a tooltip before the user sees it, so it doesn't jump
function Tooltip({ anchorRef, children }) {
  const ref = useRef(null);
  const [top, setTop] = useState(0);
  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTop(anchorRef.current.offsetTop - height - 8);   // measure, then adjust before paint
  }, [anchorRef]);
  return <div ref={ref} style={{ position: 'absolute', top }}>{children}</div>;
}
```

Interview-ready answer: *"Both run after React updates the DOM. useEffect runs asynchronously after the browser paints, so it doesn't block the screen; I use it for fetching, subscriptions and timers. useLayoutEffect runs synchronously before paint, so I use it only when I need to measure the DOM and adjust layout without a visible flicker, like positioning a tooltip. It blocks painting, so I keep it light and default to useEffect."*

**Q: Where does `useInsertionEffect` fit in, compared with `useEffect` and `useLayoutEffect`?**
A: It runs first. The three hooks run at different points in the commit process. In execution order:

| Hook | When it runs | Purpose | Can read layout or update state? |
|---|---|---|---|
| `useInsertionEffect` | During commit, **before** any layout effects or DOM reads | Inject `<style>` tags so styles exist before layout is measured. Meant for **CSS-in-JS libraries** (styled-components, Emotion) | No: no refs, no state updates |
| `useLayoutEffect` | After DOM changes, **before paint**, synchronously | Measure the DOM and adjust layout without flicker | Yes, but it blocks paint |
| `useEffect` | After commit, **usually after paint** | Sync with external systems: fetching, subscriptions, listeners, timers | Yes |

```jsx
useEffect(() => console.log('3. effect'), []);
useLayoutEffect(() => console.log('2. layout effect'), []);
useInsertionEffect(() => console.log('1. insertion effect'), []);
```

You'll almost never write `useInsertionEffect` in application code; it exists for library authors.

Interview-ready answer: *"All three run during or after commit, in this order: useInsertionEffect first, to inject styles before any layout is read, which is meant for CSS-in-JS libraries; then useLayoutEffect, synchronously before paint, for DOM measurement; then useEffect, after paint, for everything else. In app code I use useEffect by default, useLayoutEffect rarely, and useInsertionEffect essentially never."*

**Q: How do you prevent unnecessary API calls?**
A:
- **Correct dependencies:** use `[]` for fetch-once and `[id]` to refetch only when `id` changes. Don't put objects or functions recreated on each render in the array.
- **Abort stale requests:** use `AbortController` in the cleanup so outdated responses are ignored and race conditions avoided.
- **Debounce user input:** for search-as-you-type, request only after the user pauses.
- **Cache:** React Query or SWR dedupe identical requests, cache results and handle refetching.
- **Guard conditions:** for example `if (!query) return;`.

```jsx
// Guard + correct primitive dependency + cancellation, together
useEffect(() => {
  if (!query.trim()) return;                       // guard: nothing to fetch
  const controller = new AbortController();
  fetch(`/api/search?q=${encodeURIComponent(query)}`, { signal: controller.signal })
    .then((r) => r.json())
    .then(setResults)
    .catch((e) => { if (e.name !== 'AbortError') setError(e); });
  return () => controller.abort();                 // cancel the outdated request
}, [query]);                                       // re-runs only when the query changes

// Or let a library cache and dedupe for you
const { data } = useQuery({ queryKey: ['search', query], queryFn: () => search(query), enabled: !!query });
```

The full debounce and race-condition versions are in the scenarios below.

Interview-ready answer: *"First, correct dependencies: depend on primitives like an ID, not objects recreated each render. Then guard conditions so the effect skips work when there's nothing to fetch, debounce user input, and abort stale requests in the cleanup. In production I use React Query or SWR, which cache, deduplicate and refetch intelligently, so the same data isn't requested twice."*

**Q: How do you prevent memory leaks in React?**
A: Leaks happen when something a component set up outlives it: timers, event listeners, subscriptions, in-flight requests. The fix is always the same: **undo every setup in the effect's cleanup function**, which runs on unmount and before the effect re-runs.

```jsx
// 1. Clear intervals and timeouts
useEffect(() => {
  const timer = setInterval(() => console.log('Running...'), 1000);
  return () => clearInterval(timer);
}, []);

// 2. Remove event listeners (pass the same function reference)
useEffect(() => {
  const handleResize = () => console.log(window.innerWidth);
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);

// 3. Cancel API requests
useEffect(() => {
  const controller = new AbortController();
  fetch('/api/users', { signal: controller.signal });
  return () => controller.abort();
}, []);

// 4. Unsubscribe from WebSockets, event emitters and stores
useEffect(() => {
  const socket = new WebSocket(url);
  socket.onmessage = (e) => setMessages((m) => [...m, e.data]);
  return () => socket.close();
}, [url]);
```

Other sources of leaks:
- Module-level caches or arrays that grow forever.
- Closures holding large objects in long-lived listeners.
- Third-party libraries (charts, maps) that need a `.destroy()` call on unmount.

Find them with the Chrome DevTools **Memory** tab: take heap snapshots before and after mounting and unmounting, and look for detached DOM nodes.

Senior-level point: React 18 removed the "Can't perform a React state update on an unmounted component" warning, because it was often a false alarm. Proper cleanup still matters for releasing resources and avoiding stale operations. StrictMode's double mount in development helps reveal missing cleanups.

Interview-ready answer: *"Memory leaks in React come from things a component sets up that outlive it: intervals, event listeners, subscriptions, sockets and in-flight requests. I prevent them by returning a cleanup from every effect that clears timers, removes the same listener reference, unsubscribes and aborts fetches. I also destroy third-party instances on unmount. To find leaks, I compare heap snapshots in the DevTools Memory tab."*

**Q (Scenario): Product search sends an API request on every keystroke, which floods the backend. How would you implement a debounced search?**
A: Call the API only after the user **stops typing** for about 500 ms, and **cancel outdated requests** so older responses can't overwrite newer ones. The effect's cleanup does both: it clears the pending timer and aborts the in-flight request.

```jsx
function ProductSearch() {
  const [search, setSearch] = useState('');
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (!search.trim()) {
      setProducts([]);
      return;
    }
    const controller = new AbortController();

    const timer = setTimeout(async () => {
      try {
        setLoading(true);
        const response = await fetch(
          `/api/products?search=${encodeURIComponent(search)}`,
          { signal: controller.signal }
        );
        if (!response.ok) throw new Error('Request failed');
        setProducts(await response.json());
      } catch (error) {
        if (error.name !== 'AbortError') console.error(error);
      } finally {
        if (!controller.signal.aborted) setLoading(false);
      }
    }, 500);

    return () => {
      clearTimeout(timer);   // user typed again: cancel the pending call
      controller.abort();    // and abort any request already in flight
    };
  }, [search]);

  return (
    <>
      <input value={search} onChange={(e) => setSearch(e.target.value)} placeholder="Search products..." />
      {loading && <p>Searching...</p>}
      {products.map((p) => <p key={p.id}>{p.name}</p>)}
    </>
  );
}
```

Senior-level insight: in production I'd extract a `useDebounce(value, 500)` hook and give the debounced value to **TanStack Query** (`queryKey: ['products', debouncedSearch]`), which adds caching, cancellation and deduplication. I'd also add:
- A minimum query length (for example 2+ characters).
- Pagination.
- Proper error and empty states.
- Server-side search indexing (Elasticsearch, Postgres full-text search), because debouncing reduces load but doesn't make each query faster.

**Q (Scenario): A user quickly searches "React", then "Node.js", then "MongoDB". The React response arrives last and the UI shows React results. How do you fix it?**
A: This is a **race condition**: responses arrive in a different order from the requests, and the last one to arrive wins. Fixes:

1. **Abort the previous request** with `AbortController` in the effect cleanup. A new search cancels the old one.

```jsx
useEffect(() => {
  if (!search.trim()) return;
  const controller = new AbortController();

  fetch(`/api/search?q=${encodeURIComponent(search)}`, { signal: controller.signal })
    .then((r) => r.json())
    .then(setResults)
    .catch((e) => { if (e.name !== 'AbortError') console.error(e); });

  return () => controller.abort();
}, [search]);
```

2. **Ignore stale responses** when cancellation isn't possible (a third-party SDK, a non-fetch API) by using an `ignore` flag or a request ID:

```jsx
useEffect(() => {
  let ignore = false;
  searchApi(search).then((data) => {
    if (!ignore) setResults(data); // only the latest effect may update state
  });
  return () => { ignore = true; };
}, [search]);
```

3. **Use a query library.** TanStack Query keys each result by `['search', query]`, so an old response can never overwrite the current query's data.

Senior-level insight: aborting also saves bandwidth and server work, while the ignore flag only protects the UI. Both guarantee that older responses can't overwrite newer results.

**Q (Scenario): A counter should increment every second, but it shows `1` forever. Why, and how do you fix it?**

```jsx
useEffect(() => {
  const timer = setInterval(() => {
    setCount(count + 1);  // `count` is always 0 here
  }, 1000);
  return () => clearInterval(timer);
}, []);
```

A: This is a **stale closure**. The effect runs once, and the interval callback *closes over* the `count` from the first render, which is `0`. Every tick calls `setCount(0 + 1)`. A closure remembers the variables from the render it was created in, and each render has its own snapshot of state.

Fixes:
1. **Functional updater** (best here). It doesn't read `count` from the closure at all:
   ```jsx
   setCount((prev) => prev + 1);
   ```
2. **Correct dependencies:** `[count]` re-creates the interval on every change. This works but resets the timer each time.
3. **A ref for the latest value** when you need to *read* fresh state inside a long-lived callback without re-subscribing:
   ```jsx
   const countRef = useRef(count);
   useEffect(() => { countRef.current = count; }, [count]);
   // inside the interval or a socket handler: countRef.current is always current
   ```

Stale closures also show up in event listeners added once, WebSocket handlers, debounced functions and `useCallback` with missing dependencies. The `exhaustive-deps` lint rule catches most of them.

Interview-ready answer: *"It's a stale closure. The effect ran once with an empty dependency array, so the interval callback captured `count` from the first render, which is 0, and keeps setting it to 1. Each render has its own snapshot of state. I fix it with the functional updater `setCount(c => c + 1)`, which doesn't read the captured value. The alternatives are listing the dependency or using a ref holding the latest value."*

### [P1.1 — Weather App](P1.1-weather-app.md)

**Q: What does this project practise?**
A: Composing components, managing **loading, error and data states** with `useState` + `useEffect`, extracting logic into a **custom hook**, and rendering lists with keys, using the OpenWeatherMap API.

```jsx
function App() {
  const [city, setCity] = useState('Delhi');
  const [unit, setUnit] = useState('metric');                 // °C / °F toggle
  const { current, forecast, loading, error } = useWeather(city, unit);

  return (
    <main>
      <SearchBar onSearch={setCity} />
      <UnitToggle unit={unit} onChange={setUnit} />
      {loading && <WeatherSkeleton />}
      {error && <ErrorBanner message={error} />}
      {current && <CurrentWeather data={current} unit={unit} />}
      {forecast && <ForecastStrip days={forecast} />}
    </main>
  );
}
```

Interview-ready answer (project walkthrough): *"It's a weather app where I practised component composition and async UI state. Every fetch can be loading, an error or data, and the UI handles all three explicitly. The fetching lives in a custom useWeather hook, so the components stay presentational."*

**Q: What features does it include?**
A: City search, a current weather card (temperature, feels-like, humidity, wind, icon), a 5-day forecast strip, a **loading skeleton**, an **error banner** for city-not-found and network errors, and a °C/°F toggle.

```jsx
function CurrentWeather({ data, unit }) {
  const symbol = unit === 'metric' ? '°C' : '°F';
  return (
    <section className="card">
      <h2>{data.name}</h2>
      <img src={`https://openweathermap.org/img/wn/${data.weather[0].icon}@2x.png`} alt={data.weather[0].description} />
      <p className="temp">{Math.round(data.main.temp)}{symbol}</p>
      <ul>
        <li>Feels like {Math.round(data.main.feels_like)}{symbol}</li>
        <li>Humidity {data.main.humidity}%</li>
        <li>Wind {data.wind.speed} {unit === 'metric' ? 'm/s' : 'mph'}</li>
      </ul>
    </section>
  );
}
```

Interview-ready answer: *"Search by city, a current-conditions card, a 5-day forecast, a skeleton while loading, a clear error message for a city that doesn't exist versus a network failure, and a unit toggle that refetches in the other unit."*

**Q: Where does the fetching logic live?**
A: In a custom **`useWeather`** hook. It owns the fetching, loading and error state and the cancellation, so the UI components only receive data. That separates concerns, makes the logic **reusable and testable** on its own (with `renderHook`), and keeps components simple.

```jsx
function useWeather(city, unit = 'metric') {
  const [state, setState] = useState({ current: null, forecast: null, loading: false, error: null });

  useEffect(() => {
    if (!city) return;
    const controller = new AbortController();
    const key = import.meta.env.VITE_OWM_KEY;
    const q = `q=${encodeURIComponent(city)}&units=${unit}&appid=${key}`;

    setState((s) => ({ ...s, loading: true, error: null }));

    Promise.all([
      fetch(`https://api.openweathermap.org/data/2.5/weather?${q}`, { signal: controller.signal }),
      fetch(`https://api.openweathermap.org/data/2.5/forecast?${q}`, { signal: controller.signal }),
    ])
      .then(async ([cur, fc]) => {
        if (cur.status === 404) throw new Error(`City "${city}" not found`);
        if (!cur.ok || !fc.ok) throw new Error('Weather service unavailable');
        const [current, forecastRaw] = await Promise.all([cur.json(), fc.json()]);
        setState({ current, forecast: toDaily(forecastRaw.list), loading: false, error: null });
      })
      .catch((err) => {
        if (err.name !== 'AbortError') setState((s) => ({ ...s, loading: false, error: err.message }));
      });

    return () => controller.abort();          // a new city or unmount cancels the old request
  }, [city, unit]);

  return state;
}
```

Interview-ready answer: *"In a custom useWeather hook that returns current, forecast, loading and error. The current and forecast requests run in parallel with Promise.all, a 404 becomes a 'city not found' message, and the cleanup aborts the request when the city changes. Components just render what the hook returns, and the hook can be tested on its own."*

**Q: How is the 5-day forecast built from the API data?**
A: The forecast endpoint returns **40 entries at 3-hour intervals**. The app keeps **one entry per day, the 12:00 reading**, as a representative daytime value, giving 5 cards.

```js
function toDaily(list) {
  return list
    .filter((entry) => entry.dt_txt.endsWith('12:00:00'))   // one reading per day
    .slice(0, 5)
    .map((entry) => ({
      date: new Date(entry.dt * 1000),
      temp: Math.round(entry.main.temp),
      icon: entry.weather[0].icon,
      description: entry.weather[0].description,
    }));
}

// Rendered with stable keys
<ul className="forecast">{days.map((d) => <ForecastDay key={d.date.toISOString()} day={d} />)}</ul>
```

Interview-ready answer: *"The API gives 3-hourly data, so I filter to the noon reading for each day and map it to a small view model with the date, temperature and icon. It's a simple transformation that keeps the component free of API details. A more accurate version would group entries by date and take each day's min and max, and `dt_txt` is UTC, so a production version should adjust for the city's timezone."*

### [P1.2 — Movie Search](P1.2-movie-search.md)

**Q: What does the app do?**
A: It searches the OMDB API and shows a results grid, a detail view (plot, ratings, director, cast), and **favourites that persist in localStorage**, with explicit **loading, error and empty** states.

```jsx
function MovieSearchPage() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 400);
  const { movies, loading, error } = useMovieSearch(debouncedQuery);
  const { favorites, toggleFavorite, isFavorite } = useFavorites();

  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} placeholder="Search movies…" />
      {loading && <Spinner />}
      {error && <ErrorBanner message={error} />}
      {!loading && !error && debouncedQuery && movies.length === 0 && <p>No movies found.</p>}
      <MovieGrid movies={movies} isFavorite={isFavorite} onToggleFavorite={toggleFavorite} />
    </>
  );
}
```

Interview-ready answer (project walkthrough): *"A movie search app on the OMDB API with debounced search, a details view and persistent favourites. The focus was handling every UI state (loading, error, empty and results) and keeping the logic in custom hooks so the components stay presentational."*

**Q: How are excessive API calls avoided while typing?**
A: The input updates state on every keystroke, but the request depends on a **debounced** copy of the query, `useDebounce(query, 400)`. That value only updates after 400 ms with no typing, so typing "inception" fires **1** request instead of 9. Each pending request is also **aborted** when a newer query arrives, so an old, slow response can't overwrite newer results.

```jsx
function useDebounce(value, delay = 400) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);            // each keystroke resets the timer
  }, [value, delay]);
  return debounced;
}

function useMovieSearch(query) {
  const [state, setState] = useState({ movies: [], loading: false, error: null });
  useEffect(() => {
    if (query.trim().length < 2) {            // guard: too short to search
      setState({ movies: [], loading: false, error: null });
      return;
    }
    const controller = new AbortController();
    setState((s) => ({ ...s, loading: true, error: null }));
    fetch(`https://www.omdbapi.com/?apikey=${import.meta.env.VITE_OMDB_KEY}&s=${encodeURIComponent(query)}`,
      { signal: controller.signal })
      .then((r) => r.json())
      .then((data) => setState({
        movies: data.Response === 'True' ? data.Search : [],
        loading: false,
        error: data.Response === 'False' && data.Error !== 'Movie not found!' ? data.Error : null,
      }))
      .catch((e) => e.name !== 'AbortError' && setState({ movies: [], loading: false, error: 'Network error' }));
    return () => controller.abort();          // cancel outdated searches
  }, [query]);
  return state;
}
```

Interview-ready answer: *"I debounce the query with a useDebounce hook. The input stays responsive, but the API is only called after 400 ms of no typing. I also skip very short queries and abort the previous request in the effect cleanup, which prevents race conditions where an older response arrives last."*

**Q: How are favourites persisted?**
A: A `useFavorites` hook **lazily initialises** its state from `localStorage` (parsed safely, falling back to `[]`), and an effect **writes it back** whenever it changes, so favourites survive a page reload.

```jsx
function useFavorites() {
  const [favorites, setFavorites] = useState(() => {          // lazy: reads storage once
    try {
      return JSON.parse(localStorage.getItem('favorites')) ?? [];
    } catch {
      return [];                                              // corrupted JSON or blocked storage
    }
  });

  useEffect(() => {
    localStorage.setItem('favorites', JSON.stringify(favorites));
  }, [favorites]);

  const isFavorite = (id) => favorites.some((m) => m.imdbID === id);
  const toggleFavorite = (movie) =>
    setFavorites((prev) =>
      prev.some((m) => m.imdbID === movie.imdbID)
        ? prev.filter((m) => m.imdbID !== movie.imdbID)       // immutable remove
        : [...prev, movie]                                    // immutable add
    );

  return { favorites, isFavorite, toggleFavorite };
}
```

Interview-ready answer: *"A useFavorites hook initialises state lazily from localStorage inside a try/catch, so corrupted data or blocked storage can't crash the app, and an effect syncs every change back. Updates are immutable and use the functional updater. Because it's a hook, any component can use favourites without knowing about storage."*

**Q: How is the logic organised?**
A: Into **three custom hooks**, `useMovieSearch` (search results), `useMovieDetail` (one movie by ID) and `useFavorites` (persistence), plus the generic `useDebounce`. **Presentational components** (`MovieGrid`, `MovieCard`, `MovieDetail`) on top only receive props and render. This is the "container logic in hooks, UI in components" separation.

```
src/
├── hooks/
│   ├── useDebounce.js
│   ├── useMovieSearch.js
│   ├── useMovieDetail.js
│   └── useFavorites.js
└── components/
    ├── MovieGrid.jsx      // props in, JSX out
    ├── MovieCard.jsx
    └── MovieDetail.jsx
```

```jsx
function MovieDetailPage({ id }) {
  const { movie, loading, error } = useMovieDetail(id);   // logic
  if (loading) return <Spinner />;
  if (error) return <ErrorBanner message={error} />;
  return <MovieDetail movie={movie} />;                   // presentation
}
```

Interview-ready answer: *"Data logic lives in focused custom hooks: search, detail, favourites and a generic debounce. Components are purely presentational. That makes each hook reusable and testable on its own, keeps components small, and means I could swap fetch for React Query inside the hooks without touching any UI."*

---

## Phase 2 — Intermediate Hooks & Patterns

### [A2.1 — useReducer](A2.1-usereducer.md)

**Q: Why use `useReducer`?**
A: When state grows (multiple related fields, transitions that depend on the current state, many handlers updating the same state), scattered `setX` calls become hard to follow. `useReducer` moves **all the update logic into one pure function**, which is easy to read, to test without rendering, and to debug: every change is a named action.

```jsx
const initialState = { status: 'idle', data: null, error: null };

function fetchReducer(state, action) {
  switch (action.type) {
    case 'FETCH_START':   return { ...state, status: 'loading', error: null };
    case 'FETCH_SUCCESS': return { status: 'success', data: action.payload, error: null };
    case 'FETCH_ERROR':   return { ...state, status: 'error', error: action.error };
    default: throw new Error(`Unknown action: ${action.type}`);
  }
}

function Users() {
  const [state, dispatch] = useReducer(fetchReducer, initialState);
  useEffect(() => {
    dispatch({ type: 'FETCH_START' });
    fetch('/api/users').then((r) => r.json())
      .then((data) => dispatch({ type: 'FETCH_SUCCESS', payload: data }))
      .catch((error) => dispatch({ type: 'FETCH_ERROR', error }));
  }, []);
  // impossible states (loading + error at once) can't happen: each action sets a consistent state
}
```

Interview-ready answer: *"useReducer centralises complex state logic in one pure reducer function. Components just dispatch named actions. It shines when several fields change together or the next state depends on the current one, because the transitions become explicit, consistent and unit-testable without rendering anything."*

**Q: What is a reducer?**
A: A **pure function** `(state, action) => newState`. It takes the current state and an action describing *what happened*, and **returns a new state** without mutating the old one and without side effects (no fetching, no randomness). By convention, an action is an object with a `type` string plus any payload.

```js
function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      return { ...state, items: [...state.items, action.item] };          // new array
    case 'REMOVE_ITEM':
      return { ...state, items: state.items.filter((i) => i.id !== action.id) };
    case 'CLEAR':
      return { ...state, items: [] };
    default:
      return state;                                                        // unknown action: unchanged
  }
}

// Pure, so it's trivially testable
expect(cartReducer({ items: [] }, { type: 'ADD_ITEM', item: { id: 1 } })).toEqual({ items: [{ id: 1 }] });
```

Interview-ready answer: *"A reducer is a pure function that takes the current state and an action and returns the next state. It must be immutable and side-effect free, which makes it predictable and easy to unit test. Actions are plain objects with a type and an optional payload."*

**Q: How does `useReducer` relate to Redux?**
A: Same pattern (state + action → reducer → new state), different scope. `useReducer` is **local to one component** (or one subtree when combined with Context). **Redux** is an **app-wide store** with selectors, middleware (thunks, sagas), DevTools time-travel, and subscriptions that only re-render the components whose slice changed.

```jsx
// useReducer: local
const [state, dispatch] = useReducer(cartReducer, { items: [] });
dispatch({ type: 'ADD_ITEM', item });

// Redux Toolkit: global store; createSlice generates the reducer and actions (and uses Immer, so "mutating" code is safe)
const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [] },
  reducers: { addItem: (state, action) => { state.items.push(action.payload); } },
});
const items = useSelector((s) => s.cart.items);   // re-renders only when cart.items changes
useDispatch()(cartSlice.actions.addItem(item));
```

Interview-ready answer: *"useReducer is essentially Redux's reducer pattern built into React, but scoped to a component. Redux adds a single global store, selector-based subscriptions, middleware for async and side effects, and DevTools. For local or feature-level complex state, useReducer (optionally with Context) is enough. I reach for Redux Toolkit when many distant components share frequently changing state."*

**Q: When should you prefer `useReducer` over `useState`?**
A: When state is an object with fields that update together, the next state depends on the current one in a non-trivial way, many handlers modify the same state, or you want all transitions explicit and traceable. Use `useState` for simple flags, single values, or independent state.

```jsx
// useState is fine: independent, simple values
const [isOpen, setIsOpen] = useState(false);
const [query, setQuery] = useState('');

// useReducer is better: related fields and rules about how they change together
function formReducer(state, action) {
  switch (action.type) {
    case 'FIELD':  return { ...state, values: { ...state.values, [action.name]: action.value }, touched: { ...state.touched, [action.name]: true } };
    case 'SUBMIT': return { ...state, submitting: true, errors: {} };
    case 'FAIL':   return { ...state, submitting: false, errors: action.errors };
    case 'RESET':  return initialForm;
    default: return state;
  }
}
```

Interview-ready answer: *"useState for simple, independent values; useReducer when several pieces of state change together, the next state depends on the previous one, or many event handlers update the same state. The reducer keeps those rules in one tested place and the handlers become one-line dispatches."*

### [A2.2 — useContext](A2.2-usecontext.md)

**Q: How does Context work?**
A: Three steps: **create** a context, **provide** a value high in the tree, and **consume** it anywhere below with `useContext`, with no prop drilling. A consumer reads the value of the **nearest** Provider above it and re-renders when that value changes. A common pattern wraps all three in a custom provider and hook.

```jsx
const AuthContext = createContext(null);                          // 1. create

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const value = useMemo(() => ({ user, login: setUser, logout: () => setUser(null) }), [user]);
  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>; // 2. provide
}

export function useAuth() {                                       // 3. consume, via a custom hook
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be used within <AuthProvider>');
  return ctx;
}

// Anywhere below:
function Navbar() {
  const { user, logout } = useAuth();
  return user ? <button onClick={logout}>Logout {user.name}</button> : <LoginLink />;
}
// (React 19: <AuthContext value={value}> works without .Provider)
```

Interview-ready answer: *"createContext creates a context, a Provider supplies a value to its subtree, and useContext reads the nearest Provider's value, re-rendering when it changes. I usually wrap it in a custom Provider component plus a useX hook that throws if it's used outside the provider. That gives a clean API and a clear error."*

**Q: When is the context `defaultValue` used?**
A: **Only when there is no matching Provider** above the consumer. It is not a fallback for `undefined` values passed to a Provider. In practice you pass `null` and throw a helpful error in the custom hook, because a missing Provider is almost always a bug. The default is mainly useful for tests or for components that genuinely work without a provider.

```jsx
const ThemeContext = createContext('light');            // used only when there's no Provider

function Button() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>OK</button>;
}

<Button />                                              // 'light' (no provider)
<ThemeContext.Provider value="dark"><Button /></ThemeContext.Provider>        // 'dark'
<ThemeContext.Provider value={undefined}><Button /></ThemeContext.Provider>   // undefined, NOT 'light'
```

Interview-ready answer: *"The default value is used only when a component reads the context with no Provider above it. It isn't used when the Provider passes undefined. I typically default to null and make the custom hook throw, so a missing Provider fails loudly instead of silently using a fake value."*

**Q: What is the performance pitfall of Context?**
A: **Every consumer re-renders whenever the Provider's `value` changes**, compared by reference, even if it only uses a part of the value that didn't change. Two common traps:
1. A value **object recreated on every render** (`value={{ user, setUser }}`), so all consumers re-render whenever the Provider's parent renders.
2. **One big context** holding unrelated data, so changing the theme re-renders every cart and auth consumer.

```jsx
function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  // ❌ A new object every render, mixing concerns: every consumer re-renders on any change
  return <AppContext.Provider value={{ user, setUser, theme, setTheme }}>{children}</AppContext.Provider>;
}
```

Interview-ready answer: *"Context has no selectors: every consumer re-renders when the provided value changes by reference. If the value is an inline object, or one context holds unrelated state, a small change re-renders large parts of the tree. That's why context suits low-frequency data, like the user, theme or locale, rather than rapidly changing state."*

**Q: How do you fix that?**
A: Wrap the value in `useMemo` and functions in `useCallback`, and split contexts by concern (theme, auth, cart) so a theme change doesn't re-render cart consumers.

```jsx
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const toggle = useCallback(() => setTheme((t) => (t === 'light' ? 'dark' : 'light')), []);
  const value = useMemo(() => ({ theme, toggle }), [theme, toggle]);  // stable unless theme changes
  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

// Split state and dispatch: components that only dispatch never re-render on state changes
const CartStateContext = createContext(null);
const CartDispatchContext = createContext(null);
function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, { items: [] });
  return (
    <CartDispatchContext.Provider value={dispatch}>       {/* dispatch is stable */}
      <CartStateContext.Provider value={state}>{children}</CartStateContext.Provider>
    </CartDispatchContext.Provider>
  );
}
```

For high-frequency shared state, use a store with selectors (Zustand, Redux, Jotai) instead.

Interview-ready answer: *"Memoise the provider value with useMemo and the callbacks with useCallback, split contexts by concern (and even state vs dispatch), and keep providers as low in the tree as possible. If state changes often and many components read slices of it, I switch to Zustand or Redux, whose selectors re-render only the components whose slice changed."*

**Q: What is prop drilling, why is it a problem, and how do you avoid it?**
A: Prop drilling is passing data through every intermediate component, even ones that don't use it, to reach a deeply nested component. Those intermediate components receive props they don't use, which makes them harder to reuse and refactor, and every level re-renders when the value changes.

```jsx
function App() {
  const user = { name: 'Pulkit' };
  return <Dashboard user={user} />;
}
function Dashboard({ user }) {      // doesn't use `user`, only forwards it
  return <Profile user={user} />;
}
function Profile({ user }) {
  return <h1>{user.name}</h1>;
}
```

The same tree fixed with composition: `Dashboard` takes `children` and no longer needs to know about `user`.

```jsx
function App() {
  const user = { name: 'Pulkit' };
  return <Dashboard><Profile user={user} /></Dashboard>;
}
function Dashboard({ children }) {
  return <main>{children}</main>;
}
```

Ways to avoid prop drilling:
- **Component composition:** pass components as `children` so the data skips intermediate levels.
- **Context API:** share data across a subtree.
- **A state library:** Redux, Zustand or Jotai for global, frequently changing app state.
- **React Query:** for server data.

Interview-ready answer: *"Prop drilling is threading props through components that don't need them just to reach a deep child. It couples those components to data they don't use and makes refactoring painful. My first fix is composition, passing components as children, then Context for genuinely global values, and a store like Zustand or Redux for frequently changing shared state. Two or three levels of props is fine; it's not something to eliminate at all costs."*

**Q: When should you use the Context API, and when Redux?**
A:

| | Context API | Redux (Redux Toolkit) |
|---|---|---|
| What it is | A built-in way to *pass* values deep without props | A state *management* library with a single store |
| Setup | None (built in) | Extra dependency and some boilerplate (much less with RTK) |
| Updates | All consumers re-render when the value changes | Components subscribe with selectors and re-render only when their slice changes |
| Logic | You write it (often with `useReducer`) | Reducers, actions and middleware (thunks for async) |
| Dev tools | None | Redux DevTools with time-travel debugging |
| Best for | Low-frequency global data: theme, auth user, locale | Large apps with complex, frequently updated shared state |

Example use cases:
- **Context:** the authenticated user, theme preference, language settings.
- **Redux Toolkit or Zustand:** a shopping cart, multi-step workflows, state updated from many places, and state that needs middleware or predictable debugging.

```jsx
// Context: theme
const theme = useContext(ThemeContext);

// Redux Toolkit: cart, with selector-based subscriptions
const count = useSelector((s) => s.cart.items.length);   // re-renders only when the count changes
const dispatch = useDispatch();
dispatch(addItem(product));
```

Rule of thumb: start with local state, then Context. Move to Redux or Zustand when many components read and write shared state that changes often, or when you need middleware and debugging tools. Server data belongs in React Query, not in either of these.

Interview-ready answer: *"Context is a dependency-injection mechanism: great for low-frequency global values like the theme, current user or locale. Redux is a state management library: a single store, selector-based subscriptions so only affected components re-render, middleware and DevTools. I choose based on how often the state changes and how many components touch it, and I keep server data in React Query rather than in either."*

**Q: What is the difference between `useContext` and `useSyncExternalStore`?**
A: `useContext` reads a value that **React owns**, provided by a `<Provider>` higher in the tree. `useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)` subscribes to a store **outside React**, such as a custom store, browser APIs (`navigator.onLine`, `matchMedia`) or a third-party library. It keeps the UI consistent with that store during concurrent rendering, preventing **"tearing"**, where different components show different values of the same store within one render.

```jsx
// useContext: a React-managed value
const ThemeContext = createContext('light');
function Profile() {
  const theme = useContext(ThemeContext);
  return <div className={theme}>Profile</div>;
}

// useSyncExternalStore: a store outside React
let count = 0;
const listeners = new Set();
const store = {
  subscribe(cb) { listeners.add(cb); return () => listeners.delete(cb); },
  getSnapshot() { return count; },
  increment() { count++; listeners.forEach((cb) => cb()); },
};

function Counter() {
  const value = useSyncExternalStore(store.subscribe, store.getSnapshot);
  return <button onClick={store.increment}>Count: {value}</button>;
}

// A real-world case: online status
const isOnline = useSyncExternalStore(
  (cb) => {
    window.addEventListener('online', cb);
    window.addEventListener('offline', cb);
    return () => {
      window.removeEventListener('online', cb);
      window.removeEventListener('offline', cb);
    };
  },
  () => navigator.onLine,
  () => true // server snapshot for SSR
);
```

| | `useContext` | `useSyncExternalStore` |
|---|---|---|
| Data source | React Context (a Provider) | Any external store |
| Mechanism | Provider-based propagation | Subscription with snapshots |
| Re-renders | Every consumer, on any value change | Only when the selected snapshot changes |
| Good for | Theme, auth and locale configuration | Store libraries, browser APIs, legacy or non-React state |

Senior-level insight: Zustand, Redux (`react-redux`) and Jotai all use `useSyncExternalStore` internally, which is why they are safe under concurrent rendering. `getSnapshot` must return a **cached or immutable** value. Returning a new object every call causes an infinite re-render loop.

**Q (Scenario): An e-commerce app has auth, a cart, product listings, notifications, user preferences and API data. How would you manage global state?**
A: First **categorise** the state instead of putting everything into Redux or Context:

| State type | Examples | Solution |
|---|---|---|
| Local UI state | Modal open, input value, active tab | `useState` / `useReducer` in the component |
| URL state | Filters, search, page, sort | Router search params (shareable, survives refresh) |
| Global client state | Auth user, theme, cart, notifications | Context (low-frequency data) / Zustand / Redux Toolkit |
| Server state | Products, orders, users | **TanStack Query** (or RTK Query) |
| Form state | Checkout and profile forms | React Hook Form |

Example: the cart in Zustand. Selectors mean a component re-renders only when the slice it reads changes.

```jsx
import { create } from 'zustand';

const useCartStore = create((set) => ({
  cart: [],
  addToCart: (product) => set((state) => ({ cart: [...state.cart, product] })),
  removeFromCart: (id) => set((state) => ({ cart: state.cart.filter((i) => i.id !== id) })),
}));

function CartButton() {
  const count = useCartStore((s) => s.cart.length);
  const addToCart = useCartStore((s) => s.addToCart);
  return <button onClick={() => addToCart({ id: 1, name: 'Laptop' })}>Cart Items: {count}</button>;
}
```

Senior-level insight: **separate server state from client state.** Copying API responses into Redux means you hand-write caching, loading flags, refetching and invalidation. TanStack Query gives you caching, background refetching, request deduplication and loading and error states out of the box. What's left as true global client state is usually small. A cart may be both: client state for guests, server state once the user is logged in.

### [A2.3 — useRef & useMemo/useCallback](A2.3-useref-and-usememo-usecallback.md)

**Q: What is `useRef`?**
A: A hook returning a **mutable container** `{ current: value }` that **persists across renders** (the same object every time) and does **not trigger a re-render** when you change `.current`. Think of it as an instance variable for function components, or a box React won't watch.

```jsx
function StopWatch() {
  const [elapsed, setElapsed] = useState(0);       // shown on screen, so state
  const intervalRef = useRef(null);                // not shown, so a ref

  const start = () => {
    if (intervalRef.current) return;               // already running
    intervalRef.current = setInterval(() => setElapsed((e) => e + 1), 1000);
  };
  const stop = () => {
    clearInterval(intervalRef.current);
    intervalRef.current = null;
  };

  useEffect(() => () => clearInterval(intervalRef.current), []); // cleanup on unmount

  return <><p>{elapsed}s</p><button onClick={start}>Start</button><button onClick={stop}>Stop</button></>;
}
```

Interview-ready answer: *"useRef returns a stable object whose current property I can read and write freely. It survives re-renders but changing it doesn't cause one. I use it for DOM access and for mutable values the UI doesn't display, like timer IDs or the previous value. If the value should appear on screen, it belongs in state instead."*

**Q: What are the two use cases of `useRef`?**
A: (1) **DOM access**: attach it with the `ref` prop for focus, scrolling, measuring, media control, or third-party libraries. (2) **A mutable instance variable**: previous values, timer IDs, abort controllers, "is mounted" or "has run" flags, the latest callback.

```jsx
// 1. DOM access
function SearchBox() {
  const inputRef = useRef(null);
  useEffect(() => { inputRef.current.focus(); }, []);           // autofocus on mount
  return <input ref={inputRef} />;
}

function Chat({ messages }) {
  const endRef = useRef(null);
  useEffect(() => endRef.current?.scrollIntoView({ behavior: 'smooth' }), [messages]);
  return <div>{messages.map((m) => <p key={m.id}>{m.text}</p>)}<div ref={endRef} /></div>;
}

// 2. A mutable value with no re-render
function useInterval(callback, delay) {
  const savedCallback = useRef(callback);
  useEffect(() => { savedCallback.current = callback; });       // always the latest callback
  useEffect(() => {
    const id = setInterval(() => savedCallback.current(), delay);
    return () => clearInterval(id);
  }, [delay]);
}
```

Interview-ready answer: *"Two things. Accessing DOM nodes imperatively, for focus, scroll, measurement or integrating non-React libraries. And holding mutable values that must persist between renders without re-rendering, such as interval IDs, abort controllers, previous values or the latest version of a callback, which avoids stale closures."*

**Q: What is the difference between `useState` and `useRef`?**
A: Both keep a value between renders. `useState` holds data that **affects the UI**, and updating it triggers a re-render. `useRef` holds a **mutable value** in `.current`, and changing it never triggers a re-render.

| | `useState` | `useRef` |
|---|---|---|
| Re-render on change | Yes | No |
| How you update it | Setter (`setX`), asynchronous and batched | Assign `ref.current = x` directly, synchronous |
| Value during a render | A snapshot for that render | Always the latest value |
| Use for | Data shown in the UI | DOM nodes, timer IDs, previous values, flags, AbortControllers |

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  const renderCount = useRef(0);
  renderCount.current++; // counts renders without causing another one

  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <p>Render count: {renderCount.current}</p>
    </div>
  );
}
```

Senior-level point: don't read or write `ref.current` during render for anything that affects the output (the counter above is for demonstration). The UI won't update when the ref changes, and it breaks render purity under concurrent rendering. Use refs in effects and event handlers.

**Q: How do you keep track of the previous value of a prop or state?**
A: Store it in a ref inside an effect. The effect runs after render, so during render the ref still holds the previous value.

```jsx
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => { ref.current = value; }, [value]);
  return ref.current;
}

function Price({ amount }) {
  const prev = usePrevious(amount);
  const trend = prev === undefined ? '' : amount > prev ? '▲' : amount < prev ? '▼' : '';
  return <span>{amount} {trend}</span>;
}
```

Interview-ready answer: *"I use a small usePrevious hook: a ref updated in an effect after each render. During the next render the ref still holds the old value, so I can compare old and new, for example to show a price trend or animate a change."*

**Q: What does `useMemo` do and when should you use it?**
A: `useMemo(fn, deps)` memoises a computed result, re-running only when deps change. Use it for genuinely expensive work (sorting/filtering large arrays, complex transforms) — not simple computations, where memoisation overhead can cost more. It's also used to keep a **stable reference** for an object or array passed to a memoised child or used as an effect dependency.

```jsx
function ProductList({ products, query, sortBy }) {
  const visible = useMemo(() => {
    const q = query.toLowerCase();
    return products
      .filter((p) => p.name.toLowerCase().includes(q))
      .sort((a, b) => a[sortBy] - b[sortBy]);              // filter returns a new array, so sort is safe
  }, [products, query, sortBy]);                          // recomputes only when these change

  const chartOptions = useMemo(() => ({ type: 'bar', stacked: true }), []); // stable reference
  return <><Chart options={chartOptions} /><Grid items={visible} /></>;
}
```

Interview-ready answer: *"useMemo caches the result of a calculation between renders and recomputes only when its dependencies change. I use it for genuinely expensive derivations, like filtering or sorting large lists, and to keep object references stable for memoised children or effect dependencies. I don't wrap cheap computations, because memoisation has its own cost."*

**Q: What does `useCallback` do and why does it matter?**
A: `useCallback(fn, deps)` memoises a function reference. It matters when passing callbacks to `React.memo` children or using a callback as an effect dependency — otherwise a new reference each render causes unnecessary re-renders.

```jsx
const Row = React.memo(function Row({ item, onDelete }) {
  return <li>{item.name} <button onClick={() => onDelete(item.id)}>×</button></li>;
});

function List({ items, setItems }) {
  // Without useCallback: a new function every render → every Row re-renders
  const handleDelete = useCallback(
    (id) => setItems((prev) => prev.filter((i) => i.id !== id)),   // functional update, so no `items` dependency
    [setItems]
  );
  return <ul>{items.map((i) => <Row key={i.id} item={i} onDelete={handleDelete} />)}</ul>;
}
```

Interview-ready answer: *"useCallback returns the same function instance between renders until its dependencies change. Functions are compared by reference, so this matters only when the function goes to a React.memo child or is an effect or hook dependency. Otherwise it adds overhead for nothing. Using functional state updates helps keep its dependency list short."*

**Q: What is memoization?**
A: An optimisation that caches a function's result for a given set of inputs and returns the cached result when the same inputs come back, instead of recomputing it. In React it means reusing a value, function or rendered output from the previous render when its inputs (dependencies or props) haven't changed. It trades memory for speed.

```js
// Plain JavaScript memoisation
function memoize(fn) {
  const cache = new Map();
  return (n) => {
    if (cache.has(n)) return cache.get(n);   // cache hit
    const result = fn(n);
    cache.set(n, result);
    return result;
  };
}
const slowSquare = (n) => { for (let i = 0; i < 1e8; i++); return n * n; };
const fastSquare = memoize(slowSquare);
fastSquare(9);   // slow the first time
fastSquare(9);   // instant: cached

// React's three tools: useMemo (values), useCallback (functions), React.memo (components)
```

Interview-ready answer: *"Memoisation means caching the output of a pure function for given inputs so repeat calls with the same inputs skip the work. React offers it at three levels: useMemo for values, useCallback for function references and React.memo for component output. Each caches only the previous inputs and is worth it only when the work saved outweighs the comparison cost."*

**Q: What is the difference between `useMemo` and `useCallback`?**
A: `useMemo` caches the **result** of calling a function. `useCallback` caches the **function itself**. `useCallback(fn, deps)` is the same as `useMemo(() => fn, deps)`.

```jsx
const sorted = useMemo(() => [...items].sort(byPrice), [items]);      // a value (copy first: .sort() mutates)
const handleClick = useCallback((id) => select(id), [select]);        // a function
// Equivalent: const handleClick = useMemo(() => (id) => select(id), [select]);
```

Interview-ready answer: *"useMemo memoises the value a function returns; useCallback memoises the function itself. useCallback(fn, deps) is literally useMemo(() => fn, deps). I use useMemo for expensive derived data or stable objects, and useCallback for stable handlers passed to memoised children or used as dependencies."*

**Q: When should you use `useMemo` and `useCallback`, and how do they improve performance?**
A: Use them when:
- A calculation is **expensive** (filtering or sorting thousands of items, complex maths, deep transforms; as a rule of thumb, work that takes more than ~1 ms): `useMemo`.
- You pass an object, array or function to a child wrapped in **`React.memo`**. A stable reference lets the child skip re-rendering.
- A value or function is a **dependency of `useEffect`**, and you don't want the effect to re-run on every render.
- A callback is registered as an event listener inside a custom hook, so the listener isn't removed and re-added on every render.

They help in two ways: they skip repeated expensive work, and they keep **referential equality** so memoised children and effects don't run again. Don't wrap everything, though. Each hook adds comparison and memory cost, and for cheap computations it can make code slower and harder to read. Profile first. The React Compiler can memoise automatically.

```jsx
// ✅ Worth it: an expensive calculation + a stable prop for a memoised child
const stats = useMemo(() => computeStats(hugeDataset), [hugeDataset]);
const onSelect = useCallback((id) => setSelected(id), []);
<MemoizedChart data={stats} onSelect={onSelect} />

// ❌ Not worth it: cheap work, and a child that isn't memoised anyway
const fullName = useMemo(() => `${first} ${last}`, [first, last]);
const onClick = useCallback(() => setOpen(true), []);
<button onClick={onClick}>Open</button>
```

Interview-ready answer: *"For expensive calculations, and to keep references stable for memoised children or effect dependencies. They improve performance by skipping repeated work and by preserving referential equality, so React.memo children and effects don't re-run needlessly. I apply them after profiling, not by default, because each one has a cost; React Compiler can now handle much of this automatically."*

### [A2.4 — Custom Hooks](A2.4-custom-hooks.md)

**Q: What makes a function a custom hook?**
A: Two things: its **name starts with `use`**, and it **calls other hooks** (`useState`, `useEffect`, other custom hooks). The prefix is a contract. It tells React's linter to enforce the Rules of Hooks inside it and at its call sites. A function that calls no hooks should be a plain utility, not a hook.

```jsx
function useToggle(initial = false) {
  const [on, setOn] = useState(initial);
  const toggle = useCallback(() => setOn((v) => !v), []);
  return [on, toggle];
}

function Accordion() {
  const [open, toggle] = useToggle();
  return <><button onClick={toggle}>Details</button>{open && <p>…</p>}</>;
}

// ❌ Not a hook, just a helper: no hooks inside, so don't name it useX
function formatPrice(n) { return `₹${n.toFixed(2)}`; }
```

Interview-ready answer: *"A custom hook is a JavaScript function whose name starts with use and which calls other hooks. The naming convention lets React and the ESLint plugin apply the Rules of Hooks to it. It's how we package stateful logic into a reusable unit."*

**Q: Why extract logic into custom hooks?**
A: To share **stateful behaviour, not UI**, between components without wrappers, HOCs or render props: one `useFetch`, `useDebounce` or `useLocalStorage` used in many components. It also makes components **thin and declarative** (they describe UI, and the hook handles the how), and lets you **test the logic on its own** with `renderHook`.

```jsx
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    try { return JSON.parse(localStorage.getItem(key)) ?? initialValue; }
    catch { return initialValue; }
  });
  useEffect(() => { localStorage.setItem(key, JSON.stringify(value)); }, [key, value]);
  return [value, setValue];
}

// Reused anywhere, and each caller gets its own independent state
const [theme, setTheme] = useLocalStorage('theme', 'light');
const [recent, setRecent] = useLocalStorage('recentSearches', []);
```

Interview-ready answer: *"To reuse stateful logic across components without duplicating it or wrapping the tree in HOCs or render props. Components become thin and declarative, the logic lives in one place, and I can test it with renderHook. Each component calling the hook gets its own state: hooks share logic, not data."*

**Q: What are the Rules of Hooks?**
A: 1) **Only call hooks at the top level**: never inside conditions, loops, nested functions, or after an early `return`. 2) **Only call hooks from React functions**: function components or custom hooks, never from regular functions, class components or event handlers.

```jsx
function Profile({ userId }) {
  const [user, setUser] = useState(null);            // ✅ top level
  useEffect(() => { /* ... */ }, [userId]);          // ✅ top level

  if (!userId) return null;                          // early return AFTER all hooks

  // ❌ useState inside if / for / handleClick / a helper function
  return <h1>{user?.name}</h1>;
}
```

Interview-ready answer: *"Two rules: call hooks only at the top level of a component or custom hook, never conditionally, in loops or nested functions; and call them only from React function components or custom hooks. The eslint-plugin-react-hooks plugin enforces both."*

**Q: Why do those rules exist?**
A: React doesn't know hooks by name. It stores each component's hook state in an **ordered list** and matches every hook call to its slot **by call order**. If a hook is skipped on one render (for example inside an `if`), every hook after it reads the **wrong slot**, giving another hook's state. `eslint-plugin-react-hooks` enforces them.

```jsx
// Render 1 (isAdmin = true):  slot0 useState(name) | slot1 useState(role) | slot2 useEffect
// Render 2 (isAdmin = false): slot0 useState(name) | (role skipped)       | slot1 useEffect ← reads role's slot!
function Bad({ isAdmin }) {
  const [name] = useState('');
  if (isAdmin) { const [role] = useState('admin'); }   // ❌ conditional hook
  useEffect(() => {});
}
```

Interview-ready answer: *"React identifies each hook purely by the order it's called in, storing state in a list per component. If hooks are called conditionally, the order shifts between renders and state gets attached to the wrong hook, which causes very confusing bugs. Calling them unconditionally at the top level guarantees a stable order."*

**Q: Show an example of breaking the Rules of Hooks and the fix.**
A: Don't call hooks inside loops, conditions, nested functions or event handlers.

```jsx
// ❌ Wrong: the hook count changes between renders
if (isLoggedIn) {
  const [user, setUser] = useState(null);
}

// ✅ Correct: always call the hook, then branch
const [user, setUser] = useState(null);
if (!isLoggedIn) {
  return <p>Please log in</p>;
}

// ❌ A hook inside an event handler
<button onClick={() => { const [x] = useState(0); }} />
// ✅ Hooks at the top; the handler only calls the setter
const [count, setCount] = useState(0);
<button onClick={() => setCount((c) => c + 1)} />
```

React stores hook state in a list indexed by call order. If a hook is skipped on one render, every hook after it reads another hook's state. An early `return` must come **after** all hook calls. React 19's `use()` is the one exception: it can be called inside conditions.

Interview-ready answer: *"The classic mistake is a useState inside an if, or an early return placed before some hooks. The fix is always to call every hook unconditionally at the top, then branch in the rendering logic, or move the conditional part into a child component that has its own hooks."*

**Q: Show a production-quality custom hook.**
A: A `useFetch` hook that handles loading, error, HTTP errors, and cancellation when the URL changes or the component unmounts:

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();

    async function fetchData() {
      try {
        setLoading(true);
        setError(null);
        const response = await fetch(url, { signal: controller.signal });
        if (!response.ok) throw new Error(`Request failed: ${response.status}`);
        setData(await response.json());
      } catch (err) {
        if (err.name !== 'AbortError') setError(err);
      } finally {
        if (!controller.signal.aborted) setLoading(false);
      }
    }

    fetchData();
    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}

function Users() {
  const { data, loading, error } = useFetch('/api/users');
  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error loading users</p>;
  return <pre>{JSON.stringify(data)}</pre>;
}
```

Benefits: reusable logic, thinner components, separation of concerns, and logic you can test on its own with `renderHook`. Each component that calls the hook gets its **own** state; hooks share logic, not state. In production, prefer TanStack Query for server data, because it adds caching, deduplication and retries.

Interview-ready answer: *"A good custom hook encapsulates one concern completely: here, data, loading and error state, HTTP error handling, and aborting stale requests in the cleanup so URL changes and unmounts are safe. Components just consume `{ data, loading, error }`. For real server state I'd still prefer React Query, which adds caching and retries on top of the same idea."*

**Q: What are React Hooks?**
A: Functions, introduced in React 16.8, that let function components use state, lifecycle behaviour, context, refs and other React features without classes. Their names start with `use`. They make it possible to reuse stateful logic through custom hooks and keep related logic together, whereas classes split it across lifecycle methods.

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);           // state
  const theme = useContext(ThemeContext);                 // context
  const listRef = useRef(null);                           // DOM ref

  useEffect(() => {                                       // subscribe + cleanup together, in ONE place
    const conn = connect(roomId, (m) => setMessages((prev) => [...prev, m]));
    return () => conn.disconnect();
  }, [roomId]);
  // In a class this logic would be split across componentDidMount, componentDidUpdate and componentWillUnmount
  return <ul ref={listRef} className={theme}>{messages.map((m) => <li key={m.id}>{m.text}</li>)}</ul>;
}
```

Interview-ready answer: *"Hooks are functions like useState, useEffect and useContext that let function components hold state, run side effects and use other React features without classes. They keep related logic together instead of splitting it across lifecycle methods, and custom hooks make stateful logic reusable. They must follow the Rules of Hooks."*

**Q: Explain `useState`, `useEffect` and `useContext`.**
A:
- **`useState`** adds local state: `const [count, setCount] = useState(0)`. Calling the setter re-renders the component with the new value.
- **`useEffect`** runs side effects (fetching, subscriptions, timers, DOM updates) after render. The dependency array controls when it re-runs, and the cleanup function it returns undoes the effect.
- **`useContext`** reads a value from the nearest `Context.Provider` without prop drilling: `const theme = useContext(ThemeContext)`. The component re-renders when that value changes.

Other built-in hooks: `useReducer` (complex state), `useRef` (DOM refs and mutable values), `useMemo`/`useCallback` (memoisation), `useLayoutEffect`, `useId`, `useTransition`, `useDeferredValue`.

```jsx
function Clock() {
  const [time, setTime] = useState(new Date());              // useState
  const { locale } = useContext(SettingsContext);            // useContext

  useEffect(() => {                                          // useEffect + cleanup
    const id = setInterval(() => setTime(new Date()), 1000);
    return () => clearInterval(id);
  }, []);

  return <p>{time.toLocaleTimeString(locale)}</p>;
}
```

Interview-ready answer: *"useState gives a component its own state and re-renders when it changes. useEffect synchronises the component with external systems after render, with a dependency array and a cleanup. useContext reads shared values from the nearest provider without prop drilling. Together they cover most component needs: local data, side effects and shared configuration."*

### [P2.1 — Shopping Cart](P2.1-shopping-cart.md)

**Q: How is cart state managed?**
A: With `useReducer`, shared via Context, and wrapped in a custom `useCart` hook, which separates state management from the UI. Totals are **derived** during render rather than stored, so they can never drift out of sync.

```jsx
function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD': {
      const existing = state.items.find((i) => i.id === action.product.id);
      return {
        items: existing
          ? state.items.map((i) => (i.id === action.product.id ? { ...i, qty: i.qty + 1 } : i))
          : [...state.items, { ...action.product, qty: 1 }],
      };
    }
    case 'DECREMENT':
      return { items: state.items.map((i) => (i.id === action.id ? { ...i, qty: i.qty - 1 } : i)).filter((i) => i.qty > 0) };
    case 'REMOVE': return { items: state.items.filter((i) => i.id !== action.id) };
    case 'CLEAR':  return { items: [] };
    default: return state;
  }
}

const CartContext = createContext(null);
export function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, undefined, loadCart);
  const value = useMemo(() => {
    const count = state.items.reduce((n, i) => n + i.qty, 0);                 // derived
    const subtotal = state.items.reduce((sum, i) => sum + i.price * i.qty, 0); // derived
    return { items: state.items, count, subtotal, dispatch };
  }, [state]);
  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
}
export const useCart = () => useContext(CartContext);
```

Interview-ready answer (project walkthrough): *"The cart uses a reducer with ADD, DECREMENT, REMOVE and CLEAR actions, provided through Context and consumed through a useCart hook. Components just dispatch actions, and count and subtotal are derived from the items rather than stored, so there's a single source of truth."*

**Q: How does the cart survive a page reload?**
A: `useReducer(cartReducer, undefined, loadCart)` uses a **lazy initialiser** (the third argument) that reads from localStorage, falling back to `{ items: [] }` on error, and a `useEffect` writes the state back on every change.

```jsx
function loadCart() {
  try {
    const saved = JSON.parse(localStorage.getItem('cart'));
    return saved && Array.isArray(saved.items) ? saved : { items: [] };   // validate the shape
  } catch {
    return { items: [] };                                                 // corrupt JSON or blocked storage
  }
}

const [state, dispatch] = useReducer(cartReducer, undefined, loadCart);   // loadCart runs once

useEffect(() => {
  localStorage.setItem('cart', JSON.stringify(state));
}, [state]);
```

Interview-ready answer: *"I pass a lazy initialiser as useReducer's third argument, so localStorage is read once on mount, inside a try/catch with shape validation. An effect persists the state whenever it changes. It's resilient to corrupted or missing data and doesn't re-read storage on every render."*

**Q: What features does it include?**
A: A product grid, add to cart, a cart drawer with quantities and subtotal, increase, decrease and remove, a header badge, clear cart, and totals.

```jsx
function CartBadge() {
  const { count } = useCart();
  return <span className="badge" aria-label={`${count} items in cart`}>{count}</span>;
}

function CartDrawer() {
  const { items, subtotal, dispatch } = useCart();
  if (items.length === 0) return <p>Your cart is empty.</p>;
  return (
    <aside>
      {items.map((i) => (
        <div key={i.id}>
          {i.name} × {i.qty}
          <button onClick={() => dispatch({ type: 'ADD', product: i })}>+</button>
          <button onClick={() => dispatch({ type: 'DECREMENT', id: i.id })}>−</button>
          <button onClick={() => dispatch({ type: 'REMOVE', id: i.id })}>Remove</button>
        </div>
      ))}
      <p>Subtotal: ₹{subtotal.toFixed(2)}</p>
      <button onClick={() => dispatch({ type: 'CLEAR' })}>Clear cart</button>
    </aside>
  );
}
```

Interview-ready answer: *"A product grid, add to cart, a drawer with quantity controls, removal and clearing, a live badge in the header, and a subtotal. All components read and update the same cart through the useCart hook, so the badge and drawer always stay in sync."*

### [P2.2 — Kanban Board](P2.2-kanban-board.md)

**Q: What libraries/patterns does it use?**
A: `useReducer` for board state, **`@dnd-kit`** for accessible drag-and-drop (mouse, touch and keyboard), **Context** to share the board, and per-column **custom hooks**.

```jsx
import { DndContext, closestCorners, PointerSensor, KeyboardSensor, useSensor, useSensors } from '@dnd-kit/core';
import { SortableContext, verticalListSortingStrategy } from '@dnd-kit/sortable';

function Board() {
  const { columns, dispatch } = useBoard();                // Context + useReducer
  const sensors = useSensors(useSensor(PointerSensor), useSensor(KeyboardSensor)); // keyboard = accessible

  return (
    <DndContext sensors={sensors} collisionDetection={closestCorners} onDragEnd={(e) => dispatch({ type: 'DRAG_END', event: e })}>
      {columns.map((col) => (
        <SortableContext key={col.id} items={col.cardIds} strategy={verticalListSortingStrategy}>
          <Column column={col} />
        </SortableContext>
      ))}
    </DndContext>
  );
}
```

Interview-ready answer (project walkthrough): *"A Kanban board built on @dnd-kit, chosen because it's accessible with keyboard sensors, unlike older DnD libraries. Board state lives in a reducer shared through Context, so drag results become actions, and custom hooks encapsulate per-column logic."*

**Q: How does the reducer handle a drag?**
A: It distinguishes **reordering within the same column** (`arrayMove`) from **moving a card to a different column** (remove from the source, insert into the destination at the hovered index). Both return new arrays, so the update is immutable.

```js
import { arrayMove } from '@dnd-kit/sortable';

case 'MOVE_CARD': {
  const { cardId, fromCol, toCol, toIndex } = action;
  if (fromCol === toCol) {                                           // same column: reorder
    const ids = state.columns[fromCol].cardIds;
    const reordered = arrayMove(ids, ids.indexOf(cardId), toIndex);
    return { ...state, columns: { ...state.columns, [fromCol]: { ...state.columns[fromCol], cardIds: reordered } } };
  }
  const source = state.columns[fromCol].cardIds.filter((id) => id !== cardId); // different column: move
  const dest = [...state.columns[toCol].cardIds];
  dest.splice(toIndex, 0, cardId);
  return {
    ...state,
    columns: {
      ...state.columns,
      [fromCol]: { ...state.columns[fromCol], cardIds: source },
      [toCol]: { ...state.columns[toCol], cardIds: dest },
    },
  };
}
```

Interview-ready answer: *"The drag-end event becomes a MOVE_CARD action. If the source and destination columns are the same, I reorder with arrayMove. Otherwise I remove the card ID from the source and splice it into the destination at the target index. Both paths produce new objects, so React sees the change and the reducer stays pure and testable."*

**Q: What makes determining the drop target tricky?**
A: The "over" target can be **either a column** (an empty column, or the gap below the cards) **or a card** inside a column. So the board first resolves the source column from the dragged card, then finds the destination column from whichever was hit: if a card was hit, its parent column at that card's index; if a column was hit, the end of that column.

```js
function resolveDrop(state, active, over) {
  const findColumnOf = (id) =>
    state.columns[id] ? id : Object.keys(state.columns).find((c) => state.columns[c].cardIds.includes(id));

  const fromCol = findColumnOf(active.id);
  const toCol = findColumnOf(over.id);                       // works for a column ID or a card ID
  const toIndex = state.columns[over.id]
    ? state.columns[toCol].cardIds.length                    // dropped on the column: append
    : state.columns[toCol].cardIds.indexOf(over.id);         // dropped on a card: take its position
  return { cardId: active.id, fromCol, toCol, toIndex };
}
```

Interview-ready answer: *"dnd-kit reports what you're hovering over, which might be a column or a card. I normalise it with a helper that maps any ID to its column and computes the index: append when dropped on a column, take the card's position when dropped on a card. That keeps the reducer simple and handles empty columns correctly."*

**Q: How is the board persisted?**
A: Its state is saved to and restored from localStorage, using the same pattern as the cart: a lazy initialiser to load, an effect to save. Because state changes on every drag frame in some setups, the save can be **debounced** so storage isn't written dozens of times per second.

```jsx
const [state, dispatch] = useReducer(boardReducer, undefined, () => {
  try { return JSON.parse(localStorage.getItem('board')) ?? defaultBoard; }
  catch { return defaultBoard; }
});

useEffect(() => {
  const id = setTimeout(() => localStorage.setItem('board', JSON.stringify(state)), 300); // debounced save
  return () => clearTimeout(id);
}, [state]);
```

Interview-ready answer: *"The reducer is initialised lazily from localStorage, with a fallback to a default board, and an effect writes the state back, debounced, so rapid drag updates don't hammer storage. For a multi-user version I'd persist to an API with optimistic updates instead."*

---

## Phase 3 — React Router & Navigation

### [A3.1 — React Router v6](A3.1-react-router-v6.md)

**Q: What is client-side routing?**
A: In an SPA the browser loads **one HTML page**, and JavaScript handles navigation after that. React Router listens to URL changes (the History API: `pushState`/`popstate`) and **renders the matching components without asking the server for a new page**. The URL still changes, so the back button, bookmarks and shared links work, but there's no full reload: state is kept and transitions are instant.

```jsx
import { createBrowserRouter, RouterProvider, Link } from 'react-router-dom';

const router = createBrowserRouter([
  { path: '/', element: <Home /> },
  { path: '/about', element: <About /> },
]);

createRoot(document.getElementById('root')).render(<RouterProvider router={router} />);

// <a href="/about"> → full page reload, app state lost
// <Link to="/about"> → history.pushState + re-render, no reload
```

The server must serve `index.html` for every route (an SPA fallback such as `try_files $uri /index.html`), or refreshing `/about` gives a 404.

Interview-ready answer: *"Client-side routing means the app is loaded once, and navigation changes the URL through the History API while React swaps components, with no new page request. It's faster and keeps app state, while URLs, the back button and deep links still work. The server just needs to serve index.html for every path."*

**Q: What is the React Router v6 mental model?**
A: Routes are a **tree** that mirrors your UI. Each route renders its `element` when its `path` matches. **Nested routes** render inside their parent's **`<Outlet />`**, so a parent provides the layout (navbar, sidebar) and the child provides the page content.

```jsx
const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,                 // navbar + <Outlet />
    errorElement: <ErrorPage />,
    children: [
      { index: true, element: <Home /> },    // "/"
      { path: 'products', element: <Products /> },
      {
        path: 'dashboard',
        element: <DashboardLayout />,        // sidebar + <Outlet />
        children: [
          { index: true, element: <Overview /> },        // "/dashboard"
          { path: 'settings', element: <Settings /> },   // "/dashboard/settings"
        ],
      },
      { path: '*', element: <NotFound /> },
    ],
  },
]);

function RootLayout() {
  return (<><Navbar /><main><Outlet /></main></>);       // the child route renders here
}
```

Interview-ready answer: *"In v6 the route config is a nested tree that matches the UI hierarchy. A route renders its element when the path matches, and nested child routes render into the parent's Outlet. That makes shared layouts natural. Index routes give a parent its default child, and a `*` route catches everything else."*

**Q: What is the difference between `<Link>` and `<NavLink>`?**
A: Both navigate without a reload. **`<NavLink>`** also knows whether it matches the **current URL**. It gets an `active` class and `aria-current="page"` by default, and its `className`, `style` and `children` can be functions of `{ isActive, isPending }`. Use it for navigation menus and tabs, and `<Link>` everywhere else.

```jsx
<Link to="/products/42">View product</Link>

<nav>
  <NavLink to="/" end className={({ isActive }) => (isActive ? 'nav-link active' : 'nav-link')}>
    Home
  </NavLink>
  <NavLink to="/dashboard">
    {({ isActive }) => <span className={isActive ? 'font-bold' : ''}>Dashboard</span>}
  </NavLink>
</nav>
// `end` stops "/" from matching every URL (by default it would be active for "/dashboard" too)
```

Interview-ready answer: *"Link is client-side navigation. NavLink is a Link that knows if it's active, so it can style itself and sets aria-current for accessibility. I use NavLink for navbars and tabs, with the `end` prop on root links so they aren't active for every nested route."*

**Q: What changed from v5 to v6?**
A: `<Routes>` replaces `<Switch>`, matching is exact by default (no `exact` prop), nested routes use `<Outlet>` instead of `{children}`, and `element` (JSX) replaces `component` (a reference).
- `useNavigate()` replaces `useHistory()`.
- `<Navigate>` replaces `<Redirect>`.
- Routes are ranked by **specificity**, not order.
- Paths in nested routes are **relative**.
- v6.4+ adds **data routers** (`createBrowserRouter`) with loaders, actions and `errorElement`.

```jsx
// v5
<Switch>
  <Route exact path="/" component={Home} />
  <Route path="/users/:id" render={(props) => <User {...props} />} />
  <Redirect from="/old" to="/new" />
</Switch>
const history = useHistory(); history.push('/login');

// v6
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/users/:id" element={<User />} />          {/* params via useParams() */}
  <Route path="/old" element={<Navigate to="/new" replace />} />
</Routes>
const navigate = useNavigate(); navigate('/login');
```

Interview-ready answer: *"v6 replaced Switch with Routes, which ranks routes by specificity, so `exact` is gone. Routes take an element instead of component or render props. Nesting uses relative paths and Outlet. useHistory became useNavigate, and Redirect became Navigate. From 6.4 there are data routers with loaders, actions and error elements."*

### [A3.2 — Route Params & Search](A3.2-route-params-and-search.md)

**Q: Why treat the URL as state?**
A: Filters, search, sort, pagination, the selected tab or an open modal all describe *what the user is looking at*. Kept in the URL, that view can be **bookmarked, shared, refreshed, and moved through with back and forward**. Kept in `useState`, it's lost on refresh and a shared link shows something different.

```jsx
// ❌ Lost on refresh, can't be shared
const [category, setCategory] = useState('all');

// ✅ The URL is the source of truth: /products?category=shoes&sort=price&page=2
const [searchParams, setSearchParams] = useSearchParams();
const category = searchParams.get('category') ?? 'all';
const setCategory = (c) =>
  setSearchParams((prev) => { prev.set('category', c); prev.set('page', '1'); return prev; });
```

Interview-ready answer: *"Anything that defines what the user is viewing (filters, search, sort, page, tabs) belongs in the URL. Then the view survives refreshes, can be bookmarked and shared, and back and forward work as users expect. Ephemeral UI state, like a hover or an input mid-typing, stays in component state."*

**Q: Route params vs search params?**
A: Route params (`/users/:id`) identify a resource and are required; use them for IDs and slugs. Search params (`?sort=asc&page=2`) are optional key-value pairs for filters, sorting and pagination.

| | Route params | Search params |
|---|---|---|
| Example | `/products/:productId` → `/products/42` | `/products?category=shoes&page=2` |
| Purpose | **Which** resource (identity) | **How** to view it (filters, sort, page) |
| Required? | Yes; part of the route match | No; optional, have defaults |
| Read with | `useParams()` | `useSearchParams()` |

```jsx
// Route: { path: 'products/:productId', element: <ProductPage /> }
function ProductPage() {
  const { productId } = useParams();                       // "42"
  const [searchParams] = useSearchParams();
  const tab = searchParams.get('tab') ?? 'details';       // /products/42?tab=reviews
  return <ProductView id={productId} tab={tab} />;
}
```

Interview-ready answer: *"Route params identify the resource and are part of the path, like a product ID or slug. Search params are optional modifiers for filtering, sorting and pagination. A rule of thumb: if removing it means a different page, it's a route param; if it's a view option with a default, it's a search param."*

**Q: Which hooks read them?**
A: `useParams()` reads route params. `useSearchParams()` reads **and writes** search params: it returns a `URLSearchParams` object and a setter (which pushes a history entry; pass `{ replace: true }` to avoid that). All values are **strings**, so parse numbers and booleans yourself.

```jsx
function UserOrders() {
  const { userId } = useParams();                          // always a string
  const [searchParams, setSearchParams] = useSearchParams();

  const page = Number(searchParams.get('page') ?? 1);      // parse it
  const status = searchParams.get('status') ?? 'all';
  const tags = searchParams.getAll('tag');                 // ?tag=a&tag=b → ['a', 'b']

  const goToPage = (p) =>
    setSearchParams((prev) => { prev.set('page', String(p)); return prev; }); // keeps other params

  const setStatus = (s) => setSearchParams({ status: s, page: '1' }, { replace: true }); // replaces all params
}
```

Interview-ready answer: *"useParams for path parameters, useSearchParams for the query string. useSearchParams returns a URLSearchParams object plus a setter. I use the functional form of the setter to update one key while keeping the others, parse values since they're always strings, and use `replace` for updates that shouldn't add history entries, like typing in a search box."*

### [A3.3 — Programmatic Navigation](A3.3-programmatic-navigation.md)

**Q: How does `useNavigate` work?**
A: It returns a `navigate` function. Pass a **path** (absolute, or relative to the current route) to go there, or a **number** (`-1`, `1`) to move through history. Options: `{ replace: true }` replaces the current entry instead of pushing a new one, and `{ state: {...} }` passes data to the next page without putting it in the URL (read it with `useLocation().state`).

```jsx
function CheckoutForm() {
  const navigate = useNavigate();

  async function handleSubmit(data) {
    const order = await placeOrder(data);
    navigate(`/orders/${order.id}`, {
      replace: true,                         // back button shouldn't return to the submitted form
      state: { justPlaced: true },           // a one-off flag, not in the URL
    });
  }

  return <OrderForm onSubmit={handleSubmit} onCancel={() => navigate(-1)} />;
}

function OrderPage() {
  const { state } = useLocation();
  return state?.justPlaced ? <SuccessBanner /> : null;     // state is lost on refresh, so treat it as optional
}
```

Interview-ready answer: *"useNavigate returns a function I call with a path or a history delta. I use it after actions like form submits, logins and deletes. `replace` swaps the current history entry, which suits redirects, and `state` passes transient data to the next page that isn't in the URL. Since state disappears on refresh, anything important belongs in the URL instead."*

**Q: When should you use `navigate()` vs `<Link>`?**
A: `<Link>` for navigation the **user sees and initiates** (nav links, "View details"). It renders a real `<a>`, so it's accessible, can be opened in a new tab with Ctrl+click or middle-click, and works for SEO. `navigate()` for navigation as a **side effect of logic**: after a form submit, a delete, a login, a timeout or a permission check.

```jsx
// ✅ Link: a real anchor, open-in-new-tab works
<Link to={`/products/${p.id}`}>{p.name}</Link>

// ❌ Fake link: not keyboard-friendly, can't open in a new tab, no href for crawlers
<div onClick={() => navigate(`/products/${p.id}`)}>{p.name}</div>

// ✅ navigate after an action
async function handleDelete() {
  await deleteProduct(id);
  navigate('/products', { replace: true });
}
```

Interview-ready answer: *"If the user clicks something to go somewhere, it should be a Link, because it renders a real anchor with proper accessibility, new-tab support and an href. I call navigate only when navigation results from logic, like after a successful submit, login or delete, or a redirect in an effect."*

**Q: Why use `replace: true` when redirecting unauthenticated users?**
A: With the default **push**, the protected URL stays in history. After landing on `/login`, pressing **back** returns to `/dashboard`, which immediately redirects to `/login` again, and the user is stuck in a loop. **Replace** overwrites the entry, so back goes to wherever they came from. The same applies after login: replace `/login`, so back doesn't show the login form to a logged-in user.

```jsx
function ProtectedRoute() {
  const { user } = useAuth();
  const location = useLocation();
  if (!user) return <Navigate to="/login" replace state={{ from: location }} />;
  return <Outlet />;
}

// After a successful login
navigate(location.state?.from?.pathname ?? '/dashboard', { replace: true });
```

Interview-ready answer: *"A redirect shouldn't leave a history entry that redirects again. With push, back from the login page returns to the protected route, which bounces straight back to login. replace: true overwrites that entry, so back behaves normally. I use it for every redirect, both to login and after login."*

### [A3.4 — Loaders & Protected Routes](A3.4-loaders-and-protected-routes.md)

**Q: What are route loaders?**
A: Functions (React Router v6.4+, data routers) attached to routes that **fetch data before the route renders**, moving data loading from components into the routing layer. Read results with `useLoaderData()`. Benefits:
- No loading flash or empty state from `useEffect` fetching.
- Parallel loading for nested routes, instead of waterfalls.
- Built-in error handling via `errorElement`.
- `defer`/`Await` for streaming slow parts.

**Actions** handle mutations (forms).

```jsx
const router = createBrowserRouter([
  {
    path: 'users/:userId',
    element: <UserPage />,
    errorElement: <UserError />,
    loader: async ({ params, request }) => {
      const res = await fetch(`/api/users/${params.userId}`, { signal: request.signal });
      if (res.status === 404) throw new Response('Not found', { status: 404 });   // → errorElement
      return res.json();
    },
  },
]);

function UserPage() {
  const user = useLoaderData();                       // already loaded, no spinner state
  return <h1>{user.name}</h1>;
}

function UserError() {
  const error = useRouteError();
  return isRouteErrorResponse(error) && error.status === 404 ? <p>User not found</p> : <p>Something went wrong</p>;
}
```

Interview-ready answer: *"Loaders are functions on a route that fetch its data before rendering, so the component receives ready data through useLoaderData. That removes useEffect loading flashes and waterfalls, since nested loaders run in parallel. Errors flow to the route's errorElement. Many teams combine loaders with React Query, using the loader to prefetch into the query cache."*

**Q: How do you lazy-load routes?**
A: `React.lazy()` dynamically imports a component when needed, and `<Suspense>` shows a fallback while the chunk loads. Route-based splitting is the most effective form. (The full example and trade-offs are in A7.3.) Data routers also support a route-level **`lazy`** property, which loads the component **and** its loader in parallel with navigation.

```jsx
// Component-level
const Settings = lazy(() => import('./pages/Settings'));
{ path: 'settings', element: <Suspense fallback={<Spinner />}><Settings /></Suspense> }

// Route-level lazy (v6.4+): the module exports Component, loader, ErrorBoundary...
{ path: 'reports', lazy: () => import('./routes/reports') }
// routes/reports.jsx
export async function loader() { return fetchReports(); }
export function Component() { const data = useLoaderData(); return <Reports data={data} />; }
```

Interview-ready answer: *"Each route's page is loaded on demand with React.lazy and dynamic import, wrapped in Suspense, so users download only the code for the pages they visit. With data routers I use the route `lazy` property, which loads the component and its loader together. Route-level splitting gives the biggest bundle reduction for the least effort."*

**Q: What are the two kinds of 404 handling?**
A: 1) **Unknown URL**: a catch-all `path="*"` route for paths nothing matches. 2) **Known route, missing data**: `/users/99999` matches `users/:id`, but the record doesn't exist. Detect it in the loader (throw a 404 `Response`) or in the component after fetching, and show a "not found" state with the right message.

```jsx
// 1. Catch-all
{ path: '*', element: <NotFound /> }

// 2. A valid route whose data doesn't exist
loader: async ({ params }) => {
  const res = await fetch(`/api/users/${params.id}`);
  if (res.status === 404) throw new Response('User not found', { status: 404 });
  return res.json();
},
errorElement: <RouteError />,     // checks isRouteErrorResponse(error) && error.status === 404

// Without loaders: handle it in the component
if (!isLoading && !user) return <NotFound message="This user doesn't exist." />;
```

Interview-ready answer: *"Two cases: a catch-all star route for URLs that match no route, and a not-found state for valid routes whose resource doesn't exist, like a deleted user. I handle the second by throwing a 404 response from the loader to the route's errorElement, or by checking the fetch result in the component. In an SSR setup you'd also return a real 404 status for SEO."*

**Q (Scenario): An admin dashboard has roles: Admin sees everything, Manager sees reports, Employee sees only their profile. How do you implement authentication and protected routes?**
A: Architecture:
1. A **login API** authenticates the user, and the backend issues a JWT.
2. Tokens go in **HTTP-only, Secure, SameSite cookies**, not in `localStorage`, where any XSS can read them. A common pattern is a short-lived access token plus a refresh token cookie.
3. An **auth context** (or store) holds the current user and roles. It is restored on app load from `/api/auth/me`.
4. **Protected routes** redirect unauthenticated users to login.
5. **Role-based access control (RBAC)** at both the route and the UI level.
6. **Backend authorisation middleware** checks the token and role on every API request.

```jsx
const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {                         // restore the session on refresh
    fetch('/api/auth/me', { credentials: 'include' })
      .then((r) => (r.ok ? r.json() : null))
      .then((data) => setUser(data?.user ?? null))
      .finally(() => setLoading(false));
  }, []);

  const login = async (credentials) => {
    const res = await fetch('/api/auth/login', {
      method: 'POST',
      credentials: 'include',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credentials),
    });
    if (!res.ok) throw new Error('Login failed');
    setUser((await res.json()).user);
  };

  const logout = async () => {
    await fetch('/api/auth/logout', { method: 'POST', credentials: 'include' });
    setUser(null);
  };

  const value = useMemo(() => ({ user, loading, login, logout }), [user, loading]);
  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export const useAuth = () => {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be used inside AuthProvider');
  return ctx;
};
```

```jsx
function ProtectedRoute({ allowedRoles }) {
  const { user, loading } = useAuth();
  const location = useLocation();

  if (loading) return <Spinner />;                       // don't redirect before the session is restored
  if (!user) return <Navigate to="/login" replace state={{ from: location }} />;
  if (allowedRoles && !allowedRoles.includes(user.role)) return <Navigate to="/unauthorized" replace />;
  return <Outlet />;
}

<Routes>
  <Route path="/login" element={<Login />} />
  <Route element={<ProtectedRoute />}>
    <Route path="/profile" element={<Profile />} />
    <Route element={<ProtectedRoute allowedRoles={['admin', 'manager']} />}>
      <Route path="/reports" element={<Reports />} />
    </Route>
    <Route element={<ProtectedRoute allowedRoles={['admin']} />}>
      <Route path="/admin/*" element={<AdminPanel />} />
    </Route>
  </Route>
</Routes>
```

Senior-level insight: **frontend route protection is a UX feature, not security.** Anyone can change the JS or call the API directly, so the backend must validate the session and permissions on **every** protected request. Also handle:
- Token expiry, with a silent refresh on a 401 through an interceptor.
- **Refresh token rotation** and reuse detection.
- Logout that invalidates the session on the server.
- **CSRF protection** when using cookies (SameSite plus a CSRF token).
- A loading state while auth is being restored, so logged-in users don't briefly see the login page.

### [P3.1 — Multi-page SPA](P3.1-multi-page-spa.md)

**Q: What does this project focus on?**
A: Routing architecture, not data fetching:
- 5+ pages, with nested dashboard routes and shared layouts.
- Protected routes and redirect-after-login.
- A 404 page.
- Breadcrumbs.
- Active `NavLink`s.

```jsx
const router = createBrowserRouter([
  {
    element: <RootLayout />,
    children: [
      { path: '/', element: <Home /> },
      { path: '/about', element: <About /> },
      { path: '/login', element: <Login /> },
      {
        element: <ProtectedRoute />,                  // guards everything below
        children: [{
          path: '/dashboard',
          element: <DashboardLayout />,
          handle: { crumb: 'Dashboard' },
          children: [
            { index: true, element: <Overview /> },
            { path: 'settings', element: <Settings />, handle: { crumb: 'Settings' } },
          ],
        }],
      },
      { path: '*', element: <NotFound /> },
    ],
  },
]);

function Breadcrumbs() {                               // built from matched routes' handle.crumb
  const crumbs = useMatches().filter((m) => m.handle?.crumb);
  return <nav aria-label="Breadcrumb">{crumbs.map((m) => <Link key={m.id} to={m.pathname}>{m.handle.crumb}</Link>)}</nav>;
}
```

Interview-ready answer (project walkthrough): *"A multi-page SPA focused purely on routing: nested layouts for the dashboard, a pathless protected route that guards a group of routes, redirect back to the original page after login, a 404 page, active nav links, and breadcrumbs generated from route metadata with useMatches."*

**Q: How does `ProtectedRoute` send users back after login?**
A: It renders `<Navigate to="/login" state={{ from: location }} replace />`, passing the **attempted URL** in navigation state. It's a pathless layout route, so one guard protects a whole group of child routes through `<Outlet />`.

```jsx
function ProtectedRoute() {
  const { user, isLoading } = useAuth();
  const location = useLocation();

  if (isLoading) return <Spinner />;                       // don't redirect while the session is restoring
  if (!user) return <Navigate to="/login" replace state={{ from: location }} />;
  return <Outlet />;
}
```

Interview-ready answer: *"The guard is a layout route. If there's no user, it redirects to login with `replace`, and stores the current location in navigation state, so the login page knows where the user was going. If authenticated, it renders the Outlet. It also waits for the auth check to finish, so a refresh doesn't bounce logged-in users to the login page."*

**Q: How does the login page use that?**
A: It reads `location.state?.from?.pathname` (defaulting to `/dashboard`) and navigates there with `replace` after a successful (mock) login. It keeps the original query string and hash too.

```jsx
function Login() {
  const { login } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();
  const from = location.state?.from;
  const redirectTo = from ? `${from.pathname}${from.search}${from.hash}` : '/dashboard';

  async function handleSubmit(e) {
    e.preventDefault();
    const form = new FormData(e.currentTarget);
    await login(form.get('email'), form.get('password'));
    navigate(redirectTo, { replace: true });               // back won't return to /login
  }

  return <form onSubmit={handleSubmit}>{/* email, password, submit */}</form>;
}
```

Interview-ready answer: *"After a successful login it reads the `from` location that ProtectedRoute stored in navigation state, rebuilds the full URL including the query string, and navigates there with replace. If there is no `from`, it falls back to the dashboard. Users land exactly where they were trying to go."*

**Q: How is auth implemented?**
A: A **mock `AuthContext`** where any email and password works, exposed through a `useAuth` hook. The project's point is routing, so the auth layer is a stand-in with the **same interface** a real one would have (`user`, `login`, `logout`, `isLoading`), and it can be swapped for a real API without touching the routes.

```jsx
const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(() => JSON.parse(sessionStorage.getItem('user') ?? 'null'));

  const login = useCallback(async (email) => {
    await new Promise((r) => setTimeout(r, 300));                  // simulate network latency
    const u = { email, name: email.split('@')[0] };
    sessionStorage.setItem('user', JSON.stringify(u));
    setUser(u);
  }, []);

  const logout = useCallback(() => { sessionStorage.removeItem('user'); setUser(null); }, []);

  const value = useMemo(() => ({ user, login, logout, isLoading: false }), [user, login, logout]);
  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export const useAuth = () => {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be used inside AuthProvider');
  return ctx;
};
```

Interview-ready answer: *"Auth is a mock context behind a useAuth hook that exposes user, login, logout and isLoading. Because components only depend on that interface, replacing the mock with a real API (HttpOnly cookie sessions and a /me endpoint to restore them) wouldn't change any routing code. Real security would still be enforced on the backend."*

### [P3.2 — Search App with URL State](P3.2-search-app-with-url-state.md)

**Q: What is the core requirement?**
A: **Every** piece of view state lives in the URL: query (`?q=`), category (`?category=`), sort (`?sort=`) and page (`?page=`). The page is then bookmarkable and shareable, a refresh keeps it, and back and forward restore earlier searches. Nothing that defines the results is kept only in `useState`.

```jsx
// /search?q=lamp&category=home&sort=price-asc&page=2
function SearchPage() {
  const { q, category, sort, page, setParam } = useSearchState();
  const results = useMemo(() => filterItems(items, { q, category, sort }), [q, category, sort]);
  const pageItems = results.slice((page - 1) * 20, page * 20);

  return (
    <>
      <CategorySelect value={category} onChange={(c) => setParam('category', c)} />
      <SortSelect value={sort} onChange={(s) => setParam('sort', s)} />
      <ResultsGrid items={pageItems} />
      <Pagination page={page} total={results.length} onChange={(p) => setParam('page', p)} />
    </>
  );
}
```

Interview-ready answer (project walkthrough): *"A search page where the URL is the single source of truth for query, category, sort and page. Any URL fully reproduces a view, so links are shareable, refresh-safe and work with back and forward. Results are derived from the URL params with a pure filter function."*

**Q: How is the search input kept responsive without flooding the URL?**
A: The input holds **local state** that updates on every keystroke, so typing is instant. A **debounce** (`setTimeout` in an effect) syncs it to the URL only after the user pauses, using `replace: true` so each keystroke doesn't create a history entry. An effect also syncs **URL → input** when back or forward changes the query.

```jsx
function SearchInput() {
  const { q, setParam } = useSearchState();
  const [text, setText] = useState(q);

  useEffect(() => { setText(q); }, [q]);                     // URL → input (back/forward)

  useEffect(() => {                                          // input → URL, debounced
    if (text === q) return;
    const id = setTimeout(() => setParam('q', text, { replace: true }), 300);
    return () => clearTimeout(id);
  }, [text, q, setParam]);

  return <input value={text} onChange={(e) => setText(e.target.value)} placeholder="Search…" />;
}
```

Interview-ready answer: *"Two-layer state: the input has local state for instant feedback, and a debounced effect writes it to the URL after 300 ms of no typing, with replace so history isn't flooded. Another effect syncs the other way when the URL changes through back or forward. The URL stays the source of truth without making typing laggy."*

**Q: Where is URL state handled?**
A: In a `useSearchState` hook that **parses and validates** the params (with defaults and number parsing) and exposes typed values plus setters, with filtering in a pure `filter.ts` utility over 100+ mock items. Components never touch `URLSearchParams` directly.

```ts
const SORTS = ['relevance', 'price-asc', 'price-desc'] as const;

export function useSearchState() {
  const [params, setParams] = useSearchParams();

  const q = params.get('q') ?? '';
  const category = params.get('category') ?? 'all';
  const sortParam = params.get('sort');
  const sort = SORTS.includes(sortParam as any) ? (sortParam as (typeof SORTS)[number]) : 'relevance'; // validate
  const page = Math.max(1, Number(params.get('page')) || 1);                                        // sanitise

  const setParam = useCallback((key: string, value: string | number, opts?: { replace?: boolean }) => {
    setParams((prev) => {
      value === '' || value === 'all' ? prev.delete(key) : prev.set(key, String(value)); // keep URLs clean
      if (key !== 'page') prev.delete('page');                                           // new filter → page 1
      return prev;
    }, opts);
  }, [setParams]);

  const reset = useCallback(() => setParams({}), [setParams]);
  return { q, category, sort, page, setParam, reset };
}
```

Interview-ready answer: *"All URL logic lives in a useSearchState hook. It parses and validates the params with defaults, so a hand-edited or malformed URL can't break the page, and it exposes typed values and setters that reset pagination when a filter changes. Filtering is a pure utility function, easy to unit test, and components only see clean values."*

**Q: What does "Reset filters" do?**
A: It **clears all search params**, so the URL becomes `/search`, which returns every filter to its default and the page to 1. Because the URL is the source of truth, there's no separate state to reset: the inputs re-sync from the now-empty params.

```jsx
function ResetButton() {
  const { reset, q, category, sort } = useSearchState();
  const hasFilters = q || category !== 'all' || sort !== 'relevance';
  return (
    <button onClick={reset} disabled={!hasFilters}>
      Reset filters
    </button>
  );
}
// reset = () => setParams({})  →  /search
```

Interview-ready answer: *"Reset just clears the query string. Since every control reads its value from the URL, everything snaps back to its default and the page resets to 1, with no extra state to clear. It's one of the nicest payoffs of keeping the URL as the single source of truth."*

---

## Phase 4 — Data Fetching & Server State

### [A4.1 — React Query Basics](A4.1-react-query-basics.md)

**Q: What is the difference between server state and client state?**
A: **Client state** is owned by the browser and is synchronous: the selected tab, form inputs, whether a modal is open, the theme. `useState`, `useReducer`, Context or Zustand handle it. **Server state** is owned by the server and you only hold a **cached copy**. It's asynchronous, can **go stale** when others change it, needs refetching, deduplicating and invalidating, and is often shared by many components. React Query (TanStack Query) or SWR manage it.

```jsx
// Client state
const [isFilterOpen, setIsFilterOpen] = useState(false);

// Server state: cached, deduplicated, refetched in the background
const { data: products, isPending, error } = useQuery({
  queryKey: ['products', { category }],
  queryFn: () => fetchProducts(category),
});

// ❌ The anti-pattern: copying server data into useState/Redux and syncing it by hand
// useEffect(() => { fetchProducts().then(setProducts) }, [])  → no cache, no refetch, stale data
```

Interview-ready answer: *"Client state belongs to the UI and is synchronous, like an open modal or form input, so I use useState or a client store. Server state lives on the backend: it's async, shared and can go stale, so it needs caching, background refetching and invalidation. That's what React Query is for. Separating the two removes most of the hand-written loading and syncing code."*

**Q: How do you define a query?**
A: `useQuery({ queryKey, queryFn })`. The **`queryKey`** array uniquely identifies the data and is the **cache key**: include every variable the fetch depends on, and the query refetches when they change. **`queryFn`** fetches and returns the data, or throws on error. It receives an `AbortSignal` for cancellation.

```jsx
function useProduct(productId) {
  return useQuery({
    queryKey: ['products', productId],                    // cache key, varies with the ID
    queryFn: async ({ signal }) => {
      const res = await fetch(`/api/products/${productId}`, { signal });
      if (!res.ok) throw new Error(`HTTP ${res.status}`); // fetch doesn't throw on 4xx/5xx
      return res.json();
    },
    staleTime: 60_000,
    enabled: !!productId,
  });
}

function ProductPage({ productId }) {
  const { data, isPending, isError, error } = useProduct(productId);
  if (isPending) return <Spinner />;
  if (isError) return <ErrorBanner message={error.message} />;
  return <ProductView product={data} />;
}

// Provider once at the root
const queryClient = new QueryClient();
<QueryClientProvider client={queryClient}><App /></QueryClientProvider>
```

Interview-ready answer: *"useQuery takes a queryKey, an array that is the cache key and should include every parameter the request depends on, and a queryFn that returns the data or throws. I wrap queries in custom hooks like useProduct, throw on non-OK responses since fetch doesn't, and pass the abort signal through."*

**Q: What does React Query handle for you?**
A: The things you'd otherwise hand-write and get subtly wrong:
- **Caching** by key, with **deduplication**: ten components asking for the same key make one request.
- Loading, error and success states (`isPending`, `isError`, `isFetching`).
- **Background refetching**: on window focus, on reconnect, on mount when stale, or on an interval.
- Retries with exponential backoff.
- **Garbage collection** of unused cache (`gcTime`, 5 min by default).
- Pagination and infinite queries, mutations, invalidation and optimistic updates.
- Request cancellation, prefetching, and DevTools.

```jsx
// Before: ~25 lines per fetch with useState + useEffect + AbortController, no cache
// After:
const { data, isPending, isFetching, error, refetch } = useQuery({
  queryKey: ['orders', page],
  queryFn: () => getOrders(page),
  staleTime: 30_000,
  retry: 2,
  refetchOnWindowFocus: true,
});
```

Interview-ready answer: *"React Query takes over the whole server-state lifecycle: caching and deduplication by key, loading and error states, background refetching on focus or reconnect, retries, garbage collection, pagination, mutations with invalidation or optimistic updates, and DevTools. It replaces the fragile useEffect-and-useState fetching pattern."*

**Q: What is `staleTime`?**
A: How long fetched data stays **fresh**. While fresh, it's served from the cache **without refetching**. After that it's **stale**: still shown instantly, but refetched in the background on the next trigger (a mount, window focus, reconnect). The default is **`0`**, so data is stale immediately and refetches often. Don't confuse it with **`gcTime`** (formerly `cacheTime`), which is how long **unused** data stays in memory before it is garbage-collected (5 min by default).

```jsx
useQuery({
  queryKey: ['countries'],
  queryFn: fetchCountries,
  staleTime: Infinity,               // practically never changes, so never refetch automatically
});

useQuery({
  queryKey: ['user', id],
  queryFn: () => fetchUser(id),
  staleTime: 5 * 60 * 1000,          // fresh for 5 min: navigating back shows it instantly with no request
});

// Set a sensible default for the whole app
new QueryClient({ defaultOptions: { queries: { staleTime: 60_000 } } });
```

Interview-ready answer: *"staleTime is how long cached data counts as fresh. During that window React Query serves it without refetching. After it, the data is still displayed but refetched in the background when something triggers it. The default is zero, so I usually set an app-wide default like a minute, and tune it per query based on how often the data really changes. gcTime is a different setting: how long unused data stays in the cache."*

**Q: What is the difference between `Promise.all`, `Promise.allSettled`, `Promise.race` and `Promise.any`?**
A: All four take an array of promises and are commonly used to run several API requests in parallel.

| Method | Resolves when | Rejects when | Use when |
|---|---|---|---|
| `Promise.all` | **All** fulfil, giving an array of values | **Any one** rejects (fail-fast) | Every response is required (a page needs users, products and orders) |
| `Promise.allSettled` | **All** settle, giving `{ status, value \| reason }[]` | Never | Independent calls where partial success is fine (dashboard widgets) |
| `Promise.race` | The **first to settle** fulfils | The first to settle rejects | Timeouts: race a fetch against a timer |
| `Promise.any` | The **first to fulfil** | **All** reject (`AggregateError`) | First successful response from several mirrors or CDNs |

```js
const [users, products, orders] = await Promise.all([
  fetch('/api/users').then((r) => r.json()),
  fetch('/api/products').then((r) => r.json()),
  fetch('/api/orders').then((r) => r.json()),
]);

const results = await Promise.allSettled([getStats(), getFeed(), getAlerts()]);
results.forEach((r) => r.status === 'fulfilled' ? render(r.value) : logError(r.reason));

const timeout = (ms) => new Promise((_, reject) => setTimeout(() => reject(new Error('Timeout')), ms));
const data = await Promise.race([fetch('/api/slow'), timeout(5000)]);
```

Gotchas:
- When `Promise.all` rejects, the other requests **keep running**; promises can't be cancelled. To actually stop them, share an `AbortController` and call `abort()`.
- `fetch` only rejects on network errors. A 404 or 500 still *fulfils*, so check `response.ok`.
- In React Query, use `useQueries` for parallel queries instead of a manual `Promise.all`.

Interview-ready answer: *"Promise.all waits for every promise and fails fast if any rejects, for when everything is required. allSettled waits for all and never rejects, for independent calls where partial success is fine. race settles with the first promise to finish either way, which is useful for timeouts. any resolves with the first success and only rejects if all fail. None of them cancels the losers, so I use AbortController for that."*

### [A4.2 — Mutations & Cache Updates](A4.2-mutations-and-cache-updates.md)

**Q: What is `useMutation` for?**
A: **Writing** data: POST, PUT, PATCH, DELETE. Unlike queries, mutations don't run automatically. It returns `mutate`/`mutateAsync` to trigger them, status flags (`isPending`, `isError`, `isSuccess`), and lifecycle callbacks (`onMutate`, `onSuccess`, `onError`, `onSettled`), where you update or invalidate the cache.

```jsx
function AddTodo() {
  const queryClient = useQueryClient();
  const { mutate, isPending, error } = useMutation({
    mutationFn: (title) =>
      fetch('/api/todos', { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ title }) })
        .then((r) => { if (!r.ok) throw new Error('Failed'); return r.json(); }),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['todos'] }),   // refresh the list
  });

  return (
    <form onSubmit={(e) => { e.preventDefault(); mutate(new FormData(e.currentTarget).get('title')); }}>
      <input name="title" />
      <button disabled={isPending}>{isPending ? 'Adding…' : 'Add'}</button>
      {error && <p role="alert">{error.message}</p>}
    </form>
  );
}
```

Interview-ready answer: *"useMutation is for create, update and delete operations. It doesn't run on its own: I call mutate, it gives me pending and error state, and its callbacks are where I keep the cache consistent, usually invalidating related queries on success or doing an optimistic update in onMutate."*

**Q: What are the two cache update strategies?**
A: **Invalidation**: `queryClient.invalidateQueries({ queryKey })` marks matching queries stale and refetches the active ones. It's simple and always correct, and best when the server computes things (sorting, totals, derived fields). **Direct update**: `setQueryData(key, updater)` writes the new data into the cache instantly with no fetch. It's best when the mutation response contains exactly the new entity.

```jsx
// 1. Invalidation: refetch from the source of truth
onSuccess: () => {
  queryClient.invalidateQueries({ queryKey: ['todos'] });     // prefix match: ['todos', ...] too
},

// 2. Direct update: use the response, no extra request
onSuccess: (updatedTodo) => {
  queryClient.setQueryData(['todos', updatedTodo.id], updatedTodo);
  queryClient.setQueryData(['todos'], (old = []) =>
    old.map((t) => (t.id === updatedTodo.id ? updatedTodo : t)));   // immutable update
},
```

Interview-ready answer: *"Either invalidate the affected queries so they refetch, which is simple and guaranteed consistent with the server, or write the mutation's response directly into the cache with setQueryData, which avoids a round trip. I default to invalidation, use direct updates when the response is the full updated entity, and often do both: set the data now, invalidate to reconcile."*

**Q: What is an optimistic update?**
A: Updating the UI **immediately, assuming the request will succeed**, then rolling back if it fails. The app feels instant (likes, toggles, reordering). Use `onMutate` (before the request), `onError` (restore the previous cache) and `onSettled` (refetch for consistency).

```jsx
const toggleTodo = useMutation({
  mutationFn: (todo) => api.updateTodo(todo.id, { done: !todo.done }),

  onMutate: async (todo) => {
    await queryClient.cancelQueries({ queryKey: ['todos'] });      // stop in-flight fetches overwriting us
    const previous = queryClient.getQueryData(['todos']);          // snapshot for rollback
    queryClient.setQueryData(['todos'], (old) =>
      old.map((t) => (t.id === todo.id ? { ...t, done: !t.done } : t)));
    return { previous };                                           // → context
  },
  onError: (err, todo, context) => {
    queryClient.setQueryData(['todos'], context.previous);         // roll back
    toast.error('Could not update. Please try again.');
  },
  onSettled: () => queryClient.invalidateQueries({ queryKey: ['todos'] }), // reconcile with the server
});
```

Interview-ready answer: *"An optimistic update applies the expected result to the cache before the server responds, so the UI feels instant. In onMutate I cancel in-flight queries for that key, snapshot the current data, and write the optimistic version. onError restores the snapshot and tells the user, and onSettled invalidates to sync with the server. I use it for high-confidence, easily reversible actions like toggles and likes, not for payments."*

**Q: How does React 19's `useOptimistic` work?**
A: `useOptimistic(state, updateFn)` returns `[optimisticState, addOptimistic]`. While an Action or transition is pending, the UI shows the optimistic value. When the Action finishes, React automatically switches back to the real `state` from props or the server. If the request failed, the optimistic item simply disappears, with no manual rollback.

```jsx
import { useOptimistic } from 'react';

function Comments({ comments, addComment }) {
  const [optimisticComments, addOptimisticComment] = useOptimistic(
    comments,
    (state, newComment) => [...state, { ...newComment, pending: true }]
  );

  async function handleSubmit(formData) {
    const text = formData.get('comment');
    addOptimisticComment({ id: crypto.randomUUID(), text }); // shows instantly
    await addComment(text);                                  // the real request
  }

  return (
    <>
      {optimisticComments.map((c) => (
        <p key={c.id} style={{ opacity: c.pending ? 0.5 : 1 }}>{c.text}</p>
      ))}
      <form action={handleSubmit}>
        <input name="comment" required />
        <button type="submit">Post</button>
      </form>
    </>
  );
}
```

How it compares with React Query optimistic updates: `useOptimistic` is built in and local to one component, so it suits form Actions. React Query's `onMutate`/`onError` pattern updates the **shared cache**, so every component that shows that data updates. Use that one when the data is shown in several places.

Interview-ready answer: *"useOptimistic takes the real state and a reducer-like function and returns an optimistic version I can add to while an async Action runs. When the Action settles, React drops the optimistic layer and shows the real state again, so failed requests roll back automatically. It's ideal for form Actions in one component. For data shared across the app I use React Query's cache-based optimistic updates."*

### [A4.3 — Advanced React Query](A4.3-advanced-react-query.md)

**Q: What is `useInfiniteQuery`?**
A: A query for **"load more" / infinite scroll** lists that keeps all loaded **pages** in one cache entry (`data.pages`). `initialPageParam` sets the first cursor, `getNextPageParam` derives the next one from the last page (returning `undefined` when there are no more), `fetchNextPage()` loads more, and `hasNextPage` / `isFetchingNextPage` drive the UI.

```jsx
function Feed() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage, status } = useInfiniteQuery({
    queryKey: ['posts'],
    queryFn: ({ pageParam, signal }) => fetch(`/api/posts?cursor=${pageParam}`, { signal }).then((r) => r.json()),
    initialPageParam: 0,
    getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined,   // undefined → no more pages
  });

  // Auto-load when the sentinel scrolls into view
  const sentinelRef = useRef(null);
  useEffect(() => {
    const obs = new IntersectionObserver(([e]) => e.isIntersecting && hasNextPage && fetchNextPage());
    if (sentinelRef.current) obs.observe(sentinelRef.current);
    return () => obs.disconnect();
  }, [hasNextPage, fetchNextPage]);

  if (status === 'pending') return <Spinner />;
  return (
    <>
      {data.pages.flatMap((p) => p.items).map((post) => <PostCard key={post.id} post={post} />)}
      <div ref={sentinelRef}>{isFetchingNextPage ? 'Loading more…' : hasNextPage ? '' : 'You’re all caught up'}</div>
    </>
  );
}
```

Interview-ready answer: *"useInfiniteQuery manages paginated data as a list of pages under one key. getNextPageParam reads the cursor from the last page, fetchNextPage loads the next one, and hasNextPage tells me when to stop. I pair it with an IntersectionObserver sentinel for infinite scroll, and with virtualisation when the list gets very long. Cursor-based pagination is preferable to offsets for feeds that change."*

**Q: How do you run parallel queries?**
A: Multiple `useQuery` calls in the same component **run in parallel automatically**. For a **dynamic number** of queries (one per ID in a list), use **`useQueries`**, since hooks can't be called in a loop.

```jsx
// Fixed number: just call useQuery several times, and they fetch concurrently
const user = useQuery({ queryKey: ['user', id], queryFn: () => getUser(id) });
const stats = useQuery({ queryKey: ['stats', id], queryFn: () => getStats(id) });

// Dynamic number: useQueries
const results = useQueries({
  queries: productIds.map((pid) => ({
    queryKey: ['product', pid],
    queryFn: () => getProduct(pid),
    staleTime: 60_000,
  })),
  combine: (rs) => ({ products: rs.map((r) => r.data).filter(Boolean), isPending: rs.some((r) => r.isPending) }),
});
```

Interview-ready answer: *"Independent useQuery calls already run in parallel, with no Promise.all needed. When the number of queries depends on data, like one request per selected ID, I use useQueries, which takes an array of query options and can combine the results. Each one still gets its own cache entry, so they're reused elsewhere."*

**Q: How do you make a dependent query?**
A: Use **`enabled`**: the second query stays idle until the value it depends on exists. The key includes that value, so it fetches once it's available and refetches if it changes.

```jsx
const { data: user } = useQuery({
  queryKey: ['user', email],
  queryFn: () => getUserByEmail(email),
});

const userId = user?.id;
const { data: projects, isPending } = useQuery({
  queryKey: ['projects', userId],
  queryFn: () => getProjectsByUser(userId),
  enabled: !!userId,                       // waits for the first query
});
// While disabled: status 'pending', fetchStatus 'idle'
```

Dependent queries are a **waterfall** by nature. If the backend can return both in one call, or the ID is already known (from the URL), fetch in parallel instead.

Interview-ready answer: *"I set `enabled: !!value` on the query that needs data from another, and include that value in its key. It stays idle until the first query resolves, then fetches automatically. Because it's a sequential waterfall, I avoid it when the ID is already available, for example from the route params, so both can load in parallel."*

**Q: What is prefetching?**
A: Loading data into the cache **before it's needed**: `queryClient.prefetchQuery()` on link hover or focus, in a route loader, or for the likely next page. When the user arrives, the page renders **instantly from the cache**. Prefetching respects `staleTime`, so it won't refetch fresh data.

```jsx
function ProductLink({ id, name }) {
  const queryClient = useQueryClient();
  const prefetch = () =>
    queryClient.prefetchQuery({ queryKey: ['product', id], queryFn: () => getProduct(id), staleTime: 60_000 });

  return <Link to={`/products/${id}`} onMouseEnter={prefetch} onFocus={prefetch}>{name}</Link>;
}

// Paginated table: prefetch the next page in the background
useEffect(() => {
  if (hasMore) queryClient.prefetchQuery({ queryKey: ['orders', page + 1], queryFn: () => getOrders(page + 1) });
}, [page, hasMore, queryClient]);
```

Interview-ready answer: *"Prefetching fills the React Query cache ahead of time with prefetchQuery, on hover or focus of a link, in a route loader, or for the next page of a table. When the user navigates, the data is already there and the page renders without a spinner. It reuses the same keys and staleTime, so it doesn't cause duplicate requests."*

### [A4.4 — SWR Alternative](A4.4-swr-alternative.md)

**Q: What is SWR?**
A: A small data-fetching library from Vercel named after the HTTP caching strategy **stale-while-revalidate**: return the cached (stale) data immediately, then revalidate in the background and update. It handles caching, deduplication, loading and error states, focus and reconnect revalidation, and polling, with a very small API.

```jsx
import useSWR from 'swr';

const fetcher = (url) => fetch(url).then((r) => { if (!r.ok) throw new Error('Failed'); return r.json(); });

function Profile() {
  const { data, error, isLoading } = useSWR('/api/user', fetcher);   // the key is the URL
  if (isLoading) return <Spinner />;
  if (error) return <p>Failed to load</p>;
  return <h1>Hello {data.name}</h1>;
}

// A global fetcher and config
<SWRConfig value={{ fetcher, revalidateOnFocus: true, dedupingInterval: 2000 }}><App /></SWRConfig>
```

Interview-ready answer: *"SWR is Vercel's lightweight data-fetching hook built on the stale-while-revalidate idea: show cached data instantly, refetch in the background, then update. It gives caching, deduplication and focus revalidation with a one-line API, which makes it a nice fit for simpler apps, especially in the Next.js ecosystem."*

**Q: SWR vs React Query?**
A: Both solve server state with caching and background revalidation.

| | React Query (TanStack Query) | SWR |
|---|---|---|
| Size and API | Larger, more options | Very small, minimal API |
| Mutations | First-class `useMutation` with lifecycle callbacks and optimistic updates | `useSWRMutation` + `mutate`; simpler |
| Infinite and paginated data | `useInfiniteQuery` | `useSWRInfinite` |
| Cache control | Fine-grained: `staleTime`, `gcTime`, query-key prefixes, invalidation | Key-based `mutate`, fewer knobs |
| DevTools | Official, excellent | Community |
| Offline and persistence | Built-in plugins | Limited |
| Frameworks | React, Vue, Solid, Svelte, Angular | React |

```jsx
// React Query
const { data } = useQuery({ queryKey: ['user', id], queryFn: () => getUser(id) });
// SWR
const { data } = useSWR(`/api/users/${id}`, fetcher);
```

Interview-ready answer: *"Both handle caching and revalidating server data. SWR is lighter with a minimal API, great for read-heavy apps. React Query is more complete: first-class mutations with optimistic updates, fine-grained cache control and invalidation, infinite queries, offline support and excellent DevTools. For complex apps with lots of writes I choose React Query; for simple read-mostly data, either works."*

**Q: How does SWR's revalidation model work?**
A: It serves cached (stale) data immediately, then refetches in the background. By default it revalidates **on mount, on window focus and on network reconnect**. Requests for the same key within `dedupingInterval` (2 s) are **deduplicated**. You can poll with `refreshInterval`, or turn behaviours off for static data.

```jsx
useSWR('/api/notifications', fetcher, {
  refreshInterval: 10_000,       // poll every 10 s
  revalidateOnFocus: true,       // default
  revalidateOnReconnect: true,   // default
  dedupingInterval: 2000,        // default
});

// Static data: never revalidate automatically
useSWR('/api/countries', fetcher, { revalidateIfStale: false, revalidateOnFocus: false, revalidateOnReconnect: false });
// (or import useSWRImmutable from 'swr/immutable')
```

Interview-ready answer: *"Stale-while-revalidate: SWR returns whatever is cached immediately, then refetches in the background and updates if it changed. By default it revalidates on mount, window focus and reconnect, dedupes identical requests within a couple of seconds, and can poll with refreshInterval. For static data I disable revalidation or use useSWRImmutable."*

**Q: What are the key API differences?**
A: React Query uses an **options object** with an explicit `queryKey` array and `queryFn`, and a separate `useMutation`. SWR takes a **key and a fetcher** (the fetcher receives the key as its argument), and mutation goes through `useSWRMutation` or the bound or global `mutate`.

```jsx
// React Query
const { data } = useQuery({ queryKey: ['todos', filter], queryFn: () => getTodos(filter) });
const add = useMutation({ mutationFn: createTodo, onSuccess: () => qc.invalidateQueries({ queryKey: ['todos'] }) });
add.mutate({ title: 'New' });

// SWR
const { data, mutate } = useSWR(['/api/todos', filter], ([url, f]) => getTodos(f));  // the key is passed to the fetcher
const { trigger } = useSWRMutation('/api/todos', (url, { arg }) => createTodo(arg));
await trigger({ title: 'New' });
mutate();                                          // revalidate this key
// A null key = conditional fetch (like enabled: false)
useSWR(userId ? `/api/users/${userId}` : null, fetcher);
```

Interview-ready answer: *"React Query is configured with an object: a queryKey array, a queryFn, and options like enabled and staleTime, with mutations through useMutation and invalidation by key prefix. SWR takes a key and a fetcher, passes the key into the fetcher, uses a null key for conditional fetching, and handles writes with useSWRMutation and mutate. React Query is more explicit; SWR is more compact."*

### [P4.1 — GitHub Profile Explorer](P4.1-github-profile-explorer.md)

**Q: What does it practise?**
A: TanStack Query v5 across GitHub user profiles, repositories and commits: **dependent queries**, **pagination**, **cache management** (keys, `staleTime`, smart `retry`), and smooth page transitions.

```jsx
function useGitHubUser(username) {
  return useQuery({
    queryKey: ['gh-user', username],
    queryFn: () => gh(`/users/${username}`),
    enabled: !!username,
    staleTime: 5 * 60 * 1000,
  });
}

function useRepos(username, page) {
  return useQuery({
    queryKey: ['gh-repos', username, page],
    queryFn: () => gh(`/users/${username}/repos?per_page=20&page=${page}&sort=updated`),
    enabled: !!username,
    placeholderData: keepPreviousData,      // no empty flash between pages
  });
}
```

Interview-ready answer (project walkthrough): *"A GitHub explorer on TanStack Query v5: look up a user, page through their repos, and drill into a repo's commits. It practises dependent queries, paginated keys, keeping the previous page visible between pages, and a retry policy that respects GitHub's 404 and rate-limit responses."*

**Q: How is retry configured?**
A: A custom **`retry` function** skips retries for **404** (the user doesn't exist, so retrying won't help) and **403** (rate limit hit, so retrying makes it worse), and retries other failures up to 2 times. `staleTime` is 5 minutes, which also reduces calls against GitHub's rate limit.

```js
class HttpError extends Error {
  constructor(status, message) { super(message); this.status = status; }
}

async function gh(path) {
  const res = await fetch(`https://api.github.com${path}`);
  if (!res.ok) throw new HttpError(res.status, res.status === 404 ? 'Not found' : `GitHub error ${res.status}`);
  return res.json();
}

new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,
      retry: (failureCount, error) => {
        if (error instanceof HttpError && [403, 404].includes(error.status)) return false; // don't retry
        return failureCount < 2;
      },
    },
  },
});
```

Interview-ready answer: *"Retries only make sense for transient errors. I throw a typed HttpError carrying the status code, and the retry function returns false for 404s, because a missing user won't appear, and for 403s, because hammering a rate-limited API makes it worse. Other errors retry twice. A 5-minute staleTime keeps the request count low."*

**Q: How is the empty flash between pages avoided?**
A: Each page has its own query key (`['gh-repos', user, page]`), so changing the page would normally show a loading state. **`placeholderData: keepPreviousData`** (in v5, or the equivalent `previousData => previousData`) keeps showing the **previous page's data** while the next one loads. `isPlaceholderData` lets you dim the list or disable "Next".

```jsx
import { keepPreviousData } from '@tanstack/react-query';

const { data, isPlaceholderData, isFetching } = useQuery({
  queryKey: ['gh-repos', username, page],
  queryFn: () => getRepos(username, page),
  placeholderData: keepPreviousData,
});

return (
  <>
    <RepoList repos={data} style={{ opacity: isPlaceholderData ? 0.5 : 1 }} />
    <button onClick={() => setPage((p) => p - 1)} disabled={page === 1}>Prev</button>
    <button onClick={() => setPage((p) => p + 1)} disabled={isPlaceholderData || data?.length < 20}>Next</button>
    {isFetching && <small>Updating…</small>}
  </>
);
```

Interview-ready answer: *"Because each page is a different query key, switching pages would briefly show a loading state. Setting placeholderData to keepPreviousData keeps the old page on screen until the new one arrives, and isPlaceholderData lets me dim the list and disable Next meanwhile. Prefetching the next page removes the wait entirely."*

**Q: Why is the commits query dependent?**
A: Commits need a **selected repository** (`/repos/:owner/:repo/commits`), which doesn't exist until the user picks one. `enabled: !!selectedRepo` keeps the query idle until then, avoiding a request that would fail or return nothing.

```jsx
const [selectedRepo, setSelectedRepo] = useState(null);

const commits = useQuery({
  queryKey: ['gh-commits', username, selectedRepo],
  queryFn: () => gh(`/repos/${username}/${selectedRepo}/commits?per_page=10`),
  enabled: !!selectedRepo,           // waits for a selection
});

{!selectedRepo ? <p>Select a repository to see commits</p> : <CommitList query={commits} />}
```

Interview-ready answer: *"The commits endpoint needs a repository name, and that only exists after the user picks one. So the query is disabled with `enabled: !!selectedRepo` and includes the repo in its key. It fires as soon as a repo is selected and caches commits per repo, so switching back is instant."*

### [P4.2 — Real-time Dashboard](P4.2-real-time-dashboard.md)

**Q: How is auto-refresh implemented?**
A: **`refetchInterval` polling**, tuned per data type by how fast it changes and how much it matters: metrics every **5 s**, alerts every **10 s**, services every **15 s**. React Query handles the timers, deduplication and cleanup. `refetchIntervalInBackground: false` (the default) pauses polling while the tab is hidden.

```jsx
const metrics  = useQuery({ queryKey: ['metrics'],  queryFn: getMetrics,  refetchInterval: 5_000 });
const alerts   = useQuery({ queryKey: ['alerts'],   queryFn: getAlerts,   refetchInterval: 10_000 });
const services = useQuery({ queryKey: ['services'], queryFn: getServices, refetchInterval: 15_000 });

// Adaptive: poll faster while there are open alerts
useQuery({ queryKey: ['alerts'], queryFn: getAlerts, refetchInterval: (q) => (q.state.data?.open > 0 ? 3_000 : 15_000) });
```

For truly real-time needs (sub-second, or many clients), push updates with WebSockets or SSE and write them into the cache with `setQueryData`.

Interview-ready answer (project walkthrough): *"Polling through React Query's refetchInterval, with a different frequency per widget based on how fast its data changes. React Query manages the timers, stops polling in background tabs and dedupes requests. For genuinely real-time data I'd switch to WebSockets or server-sent events and push the messages into the query cache."*

**Q: How does pause/resume work?**
A: A toggle in state sets `refetchInterval` to **`false`** when paused, which stops polling, and back to the interval when resumed. Nothing else needs to change, because the query options are reactive.

```jsx
function Dashboard() {
  const [live, setLive] = useState(true);

  const metrics = useQuery({
    queryKey: ['metrics'],
    queryFn: getMetrics,
    refetchInterval: live ? 5_000 : false,   // false = stop polling
  });

  return (
    <>
      <button onClick={() => setLive((l) => !l)} aria-pressed={live}>{live ? '⏸ Pause' : '▶ Resume'}</button>
      <MetricsPanel data={metrics.data} lastUpdated={metrics.dataUpdatedAt} />
    </>
  );
}
```

Interview-ready answer: *"refetchInterval is just a reactive option, so pause/resume is a boolean in state: when paused it's false and polling stops, and when resumed it goes back to the interval. I also show the last-updated time from dataUpdatedAt, so users know how fresh the numbers are."*

**Q: How do optimistic updates work for acknowledging alerts or toggling services?**
A: `onMutate` first calls **`queryClient.cancelQueries`** for that key, so an **in-flight poll** can't land after the optimistic write and overwrite it with old data. It then snapshots and updates the cache. `onError` rolls back, and `onSettled` invalidates to sync.

```jsx
const acknowledge = useMutation({
  mutationFn: (alertId) => api.acknowledgeAlert(alertId),
  onMutate: async (alertId) => {
    await queryClient.cancelQueries({ queryKey: ['alerts'] });            // crucial with polling
    const previous = queryClient.getQueryData(['alerts']);
    queryClient.setQueryData(['alerts'], (old) =>
      old.map((a) => (a.id === alertId ? { ...a, acknowledged: true } : a)));
    return { previous };
  },
  onError: (_e, _id, ctx) => queryClient.setQueryData(['alerts'], ctx.previous),
  onSettled: () => queryClient.invalidateQueries({ queryKey: ['alerts'] }),
});
```

Interview-ready answer: *"The standard optimistic pattern, with one detail that matters when polling: cancelQueries first, because a poll already in flight could resolve after my optimistic write and revert it with stale server data. Then snapshot, update the cache, roll back in onError, and invalidate in onSettled to reconcile."*

**Q: How are render errors handled?**
A: A class-based **`DashboardErrorBoundary`** using `getDerivedStateFromError` (show the fallback) and `componentDidCatch` (log it), with a **retry button** that resets its state. Each widget is wrapped separately, so one failing panel doesn't blank the whole dashboard. Combined with React Query's `QueryErrorResetBoundary`, retry also refetches the failed queries.

```jsx
class DashboardErrorBoundary extends React.Component {
  state = { error: null };
  static getDerivedStateFromError(error) { return { error }; }
  componentDidCatch(error, info) { logError(error, info.componentStack); }
  reset = () => { this.props.onReset?.(); this.setState({ error: null }); };
  render() {
    if (this.state.error) {
      return (
        <div role="alert">
          <p>This panel failed to load.</p>
          <button onClick={this.reset}>Retry</button>
        </div>
      );
    }
    return this.props.children;
  }
}

<QueryErrorResetBoundary>
  {({ reset }) => (
    <DashboardErrorBoundary onReset={reset}>
      <MetricsPanel />            {/* useQuery({ ..., throwOnError: true }) */}
    </DashboardErrorBoundary>
  )}
</QueryErrorResetBoundary>
```

Interview-ready answer: *"Each dashboard panel is wrapped in its own error boundary, a class component using getDerivedStateFromError for the fallback and componentDidCatch for logging, so a crash stays local. The fallback has a Retry button that resets the boundary. Paired with React Query's QueryErrorResetBoundary, that retry also refetches the failed query."*

---

## Phase 5 — Forms & Validation

### [A5.1 — React Hook Form](A5.1-react-hook-form.md)

**Q: Why React Hook Form instead of controlled inputs?**
A: Controlled inputs store every field in state, so **every keystroke re-renders the form** (and often its children), plus you hand-write validation, touched and dirty tracking, and submit state. RHF registers **uncontrolled inputs read via refs**, so typing causes **zero re-renders**. Only the things you subscribe to (errors, `watch`ed fields, `formState`) re-render. It also has built-in validation, schema resolvers, field arrays and a tiny API.

```jsx
// Controlled: 3 useState + a re-render per keystroke + hand-written validation
const [email, setEmail] = useState('');
<input value={email} onChange={(e) => setEmail(e.target.value)} />

// React Hook Form: no re-render while typing
import { useForm } from 'react-hook-form';
function SignupForm() {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm();
  const onSubmit = async (data) => api.signup(data);   // { email, password }
  return (
    <form onSubmit={handleSubmit(onSubmit)} noValidate>
      <input type="email" {...register('email', { required: 'Email is required' })} />
      {errors.email && <p role="alert">{errors.email.message}</p>}
      <input type="password" {...register('password', { minLength: { value: 8, message: 'Min 8 characters' } })} />
      <button disabled={isSubmitting}>Sign up</button>
    </form>
  );
}
```

Interview-ready answer: *"React Hook Form keeps inputs uncontrolled and reads them through refs, so typing doesn't re-render the form. Only subscribed state like errors does, which matters a lot in large forms. It also removes boilerplate: validation, touched and dirty tracking, submit state, schema integration and dynamic field arrays come built in."*

**Q: What does the core API provide?**
A: `useForm()` returns:
- `register(name, rules)`: props to spread on an input.
- `handleSubmit(onValid, onInvalid)`: validates, then calls your function with the typed data.
- `formState`: `errors`, `isSubmitting`, `isDirty`, `isValid`, `touchedFields`.
- `watch(name)`: subscribes to a value, which re-renders.
- `setValue(name, value)` and `getValues()`.
- `reset(values)`.
- `trigger(name)`: validates on demand.
- `setError`, for server errors.
- `control`, for `Controller`/`useFieldArray` with non-native inputs.

```jsx
const {
  register, handleSubmit, watch, setValue, reset, trigger, setError, control,
  formState: { errors, isDirty, isSubmitting },
} = useForm({ defaultValues: { plan: 'free', country: '' }, mode: 'onBlur' });

const plan = watch('plan');                                  // conditional UI
useEffect(() => { reset(loadedProfile); }, [loadedProfile, reset]); // load data into the form

async function onSubmit(data) {
  const res = await api.save(data);
  if (res.status === 409) setError('email', { message: 'Email already registered' }); // server error → field
}

// A third-party component (e.g. react-select) via Controller
<Controller name="country" control={control} rules={{ required: true }}
  render={({ field }) => <Select {...field} options={countries} />} />
```

Interview-ready answer: *"register wires native inputs, handleSubmit validates then gives me typed data, and formState exposes errors and flags like isDirty and isSubmitting. watch, setValue, reset and trigger cover dynamic behaviour, setError maps server errors onto fields, and Controller with control integrates UI-library inputs that aren't native."*

**Q: How do you add validation?**
A: Two ways. **Inline rules** as `register`'s second argument (`required`, `min`/`max`, `minLength`, `pattern`, a custom `validate`), each with a message. Or a **schema** (Zod or Yup) via `zodResolver`, which keeps all rules in one reusable, typed place and is the better option for anything non-trivial.

```jsx
// 1. Inline rules
<input {...register('username', {
  required: 'Username is required',
  pattern: { value: /^[a-z0-9_]{3,16}$/, message: '3–16 lowercase letters, numbers or _' },
  validate: async (v) => (await isTaken(v)) ? 'Already taken' : true,   // async custom rule
})} />

// 2. A schema with Zod
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const schema = z.object({
  email: z.string().email('Invalid email'),
  age: z.coerce.number().int().min(18, 'Must be 18+'),
});
type FormValues = z.infer<typeof schema>;

const { register, handleSubmit, formState: { errors } } = useForm<FormValues>({ resolver: zodResolver(schema) });
```

Interview-ready answer: *"For simple forms, inline rules on register with custom messages. For anything real, a Zod schema through zodResolver: the rules live in one place, can be shared with the backend, and give me the TypeScript type for free with z.infer. I set the validation mode, such as onBlur or onTouched, to balance feedback against noise."*

**Q: What are Actions and `useActionState` in React 19?**
A: An **Action** is an async function that handles a submission or mutation. Pass it to `<form action={...}>` and React calls it with the `FormData`, runs it as a transition, tracks the pending state, and resets uncontrolled fields on success. `useActionState(actionFn, initialState)` wraps an Action and returns `[state, formAction, isPending]`. The action receives `(previousState, formData)`, and whatever it returns becomes the new `state`, which typically holds the success or error message.

```jsx
import { useActionState } from 'react';
import { useFormStatus } from 'react-dom';

async function createUser(prevState, formData) {
  const name = formData.get('name');
  try {
    const res = await fetch('/api/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name }),
    });
    if (!res.ok) throw new Error('Failed to create user');
    return { success: true, message: 'User created!' };
  } catch (error) {
    return { success: false, message: error.message };
  }
}

function UserForm() {
  const [state, formAction, isPending] = useActionState(createUser, { success: false, message: '' });

  return (
    <form action={formAction}>
      <input name="name" required />
      <SubmitButton />
      <p role="status">{state.message}</p>
    </form>
  );
}

function SubmitButton() {
  const { pending } = useFormStatus(); // reads the parent <form>'s state, no props needed
  return <button disabled={pending}>{pending ? 'Submitting...' : 'Submit'}</button>;
}
```

In Next.js the action can be a **Server Action** (`'use server'`) that runs on the server directly, without a hand-written API route. Forms built this way also work before JavaScript loads (progressive enhancement).

Interview-ready answer: *"In React 19 an Action is an async function passed to a form's action prop. React runs it as a transition with FormData and tracks its pending state. useActionState wraps it to give me the last returned result, such as success or validation messages, plus isPending, and useFormStatus lets a nested submit button read the pending state. With Server Actions in Next.js, the same form works even before JavaScript loads."*

**Q: When would you use Actions vs React Hook Form?**
A: Actions plus `useActionState` suit simple-to-medium forms, especially with Server Actions, and need no library. React Hook Form plus Zod is still stronger for complex client-side forms: field-level validation as you type, dynamic field arrays, multi-step wizards, and fine-grained re-render control. They can also be combined. For complex server-state caching, still use TanStack Query.

```jsx
// Simple: an Action + server-side validation, no library
<form action={subscribeAction}><input name="email" type="email" required /><SubmitButton /></form>

// Complex: RHF for instant field validation, then hand the validated data to an Action or mutation
const { register, handleSubmit } = useForm({ resolver: zodResolver(schema) });
const onSubmit = handleSubmit((data) => startTransition(() => saveProfileAction(data)));
```

Interview-ready answer: *"For simple forms, like a newsletter, a contact form or a settings toggle, especially in Next.js with Server Actions, I use Actions and useActionState: no dependency, progressive enhancement, pending state built in. For complex client-heavy forms with live validation, dynamic arrays and multi-step flows, I use React Hook Form with Zod, and submit through an Action or a React Query mutation. Server-side validation is required either way."*

### [A5.2 — Zod Validation](A5.2-zod-validation.md)

**Q: What is Zod?**
A: A **TypeScript-first schema validation** library. You describe the shape of data once. Zod **validates it at runtime** (with readable error messages) and **infers the TypeScript type at compile time**, so validation and types can't drift apart. It's used for forms, API responses, environment variables and route params, anywhere untrusted data enters the app.

```ts
import { z } from 'zod';

const UserSchema = z.object({
  id: z.number(),
  name: z.string().min(1),
  email: z.string().email(),
  role: z.enum(['admin', 'user']),
});
type User = z.infer<typeof UserSchema>;          // { id: number; name: string; email: string; role: 'admin' | 'user' }

const result = UserSchema.safeParse(await res.json());   // validate an API response at runtime
if (!result.success) {
  console.error(result.error.flatten());                 // { fieldErrors: { email: ['Invalid email'] } }
} else {
  const user: User = result.data;                        // fully typed and trusted
}
```

Interview-ready answer: *"Zod is a TypeScript-first validation library: one schema gives runtime validation and a static type via z.infer. TypeScript types disappear at runtime, so Zod is how I actually check untrusted data (form input, API responses, env vars) and get typed results out of it."*

**Q: What is schema-first validation?**
A: The **schema is the single source of truth**. Define it once, then derive the TypeScript type with `z.infer<typeof schema>`, validate the form via `zodResolver`, and (in a full-stack TS app) validate the same payload on the server. Change the schema and validation, types and errors all update together.

```ts
// schemas/signup.ts, shared by the client and the server
export const signupSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});
export type SignupInput = z.infer<typeof signupSchema>;

// Client
useForm<SignupInput>({ resolver: zodResolver(signupSchema) });

// Server (an Express route or a Server Action)
const parsed = signupSchema.safeParse(req.body);
if (!parsed.success) return res.status(400).json(parsed.error.flatten());
```

Interview-ready answer: *"Schema-first means the validation schema defines everything: I infer the TypeScript type from it, plug it into React Hook Form through a resolver, and reuse it on the server. There's no duplicated type or rule, so the form, the API contract and the backend validation can't drift apart."*

**Q: What are the key Zod APIs?**
A:
- **Primitives:** `z.string()`, `z.number()`, `z.boolean()`, `z.date()`.
- **Composites:** `z.object()`, `z.array()`, `z.enum()`, `z.union()`, `z.discriminatedUnion()`.
- **Validators:** `.min()`, `.max()`, `.email()`, `.url()`, `.regex()`.
- **Modifiers:** `.optional()`, `.nullable()`, `.default()`.
- **Coercion:** `z.coerce.number()`, for form strings.
- **Transforms and refinements:** `.transform()`, `.refine()`, `.superRefine()`.
- **Parsing:** `.parse()` throws, `.safeParse()` returns a result.
- **Types:** `z.infer`.

```ts
const ProductSchema = z.object({
  title: z.string().trim().min(3, 'Too short').max(80),
  price: z.coerce.number().positive(),                      // "19.99" from an input → 19.99
  tags: z.array(z.string()).max(5).default([]),
  website: z.string().url().optional(),
  status: z.enum(['draft', 'published']),
  slug: z.string().regex(/^[a-z0-9-]+$/, 'lowercase, numbers, dashes'),
  launchDate: z.coerce.date().refine((d) => d > new Date(), 'Must be in the future'),
}).transform((p) => ({ ...p, title: p.title.replace(/\s+/g, ' ') }));
```

Interview-ready answer: *"Primitives and composites to describe shape (object, array, enum, union), chained validators like min, max, email and regex, optional, nullable and default, coerce for converting form strings, transform to normalise data, refine and superRefine for custom rules, safeParse for non-throwing validation, and z.infer to get the type."*

**Q: `.refine()` vs `.superRefine()`?**
A: Both add custom validation. **`.refine(check, { message, path })`** takes a predicate that returns true or false, giving **one error**. Placed on the object, it can compare fields, with `path` choosing which field shows the error. **`.superRefine((val, ctx) => …)`** gives you a context to **add any number of issues**, each with its own path and message, for multiple cross-field rules or conditional validation.

```ts
// refine: one rule, one error
const passwordSchema = z.object({
  password: z.string().min(8),
  confirm: z.string(),
}).refine((d) => d.password === d.confirm, { message: 'Passwords must match', path: ['confirm'] });

// superRefine: several conditional issues
const bookingSchema = z.object({
  checkIn: z.coerce.date(),
  checkOut: z.coerce.date(),
  guests: z.number(),
  childAges: z.array(z.number()).optional(),
}).superRefine((d, ctx) => {
  if (d.checkOut <= d.checkIn)
    ctx.addIssue({ code: z.ZodIssueCode.custom, path: ['checkOut'], message: 'Check-out must be after check-in' });
  if (d.guests > 1 && !d.childAges)
    ctx.addIssue({ code: z.ZodIssueCode.custom, path: ['childAges'], message: 'Add ages for all guests' });
});
```

Interview-ready answer: *"refine adds a single boolean check with one message. On an object schema I use it for simple cross-field rules like 'passwords match', with path pointing the error at the right field. superRefine hands me a context so I can add multiple issues with different paths and conditions, which suits complex business rules like date ranges plus conditional requirements."*

### [A5.3 — Complex Forms](A5.3-complex-forms.md)

**Q: What is `useFieldArray` for?**
A: **Dynamic lists of fields** where the user adds and removes rows: phone numbers, work experience, line items, passengers. It provides `fields` (with stable `id`s), `append`, `prepend`, `insert`, `remove`, `move`, `swap` and `update`, all wired into validation and form state.

```jsx
const schema = z.object({
  phones: z.array(z.object({ number: z.string().min(10, 'Invalid number') })).min(1, 'Add at least one'),
});

function PhonesForm() {
  const { control, register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(schema),
    defaultValues: { phones: [{ number: '' }] },
  });
  const { fields, append, remove } = useFieldArray({ control, name: 'phones' });

  return (
    <form onSubmit={handleSubmit(console.log)}>
      {fields.map((field, index) => (
        <div key={field.id}>
          <input {...register(`phones.${index}.number`)} />
          <button type="button" onClick={() => remove(index)} disabled={fields.length === 1}>Remove</button>
          {errors.phones?.[index]?.number && <span>{errors.phones[index].number.message}</span>}
        </div>
      ))}
      <button type="button" onClick={() => append({ number: '' })}>Add phone</button>
      <button>Save</button>
    </form>
  );
}
```

Interview-ready answer: *"useFieldArray manages repeatable groups of fields in React Hook Form. It gives me fields with stable IDs and helpers to append, remove, move and swap rows, and validation applies per item through paths like `phones.0.number`. I use it for things like line items or passengers, with an array schema in Zod for min and max counts."*

**Q: What key should dynamic fields use?**
A: Each field's stable **`field.id`** generated by `useFieldArray`, **not the array index**. With index keys, removing row 2 makes row 3 take over key 2, so React reuses row 2's DOM input (and its uncontrolled value) for different data. Values, focus and errors then jump to the wrong row.

```jsx
// ❌ Removing a middle row shows the wrong values in the inputs
{fields.map((field, index) => <input key={index} {...register(`items.${index}.name`)} />)}

// ✅ A stable identity per row
{fields.map((field, index) => <input key={field.id} {...register(`items.${index}.name`)} />)}
// Note: still use `index` in the register path; only the key should be field.id
```

Interview-ready answer: *"field.id, the stable ID useFieldArray generates, never the index. Inputs in RHF are uncontrolled, so their values live in the DOM. With index keys, removing or reordering rows makes React reuse the wrong DOM inputs, and values and errors show up on the wrong row. The index is still used in the register path, just not as the key."*

**Q: How do multi-step forms work?**
A: Keep **`currentStep`** in state and render one step at a time, with **each step validating only its own fields** before moving forward. Two common designs: **one shared `useForm`** across steps, validating the current step's fields with `trigger(fieldsForStep)`, or **one `useForm` per step**, merging each step's data into shared state and submitting it all at the end.

```jsx
const steps = [
  { title: 'Account', fields: ['email', 'password'] },
  { title: 'Profile', fields: ['name', 'dob'] },
  { title: 'Review',  fields: [] },
];

function Wizard() {
  const [step, setStep] = useState(0);
  const methods = useForm({ resolver: zodResolver(fullSchema), mode: 'onTouched' });

  const next = async () => {
    const ok = await methods.trigger(steps[step].fields);    // validate this step only
    if (ok) setStep((s) => s + 1);
  };

  return (
    <FormProvider {...methods}>
      <Progress current={step} total={steps.length} />
      <form onSubmit={methods.handleSubmit(submitAll)}>
        {step === 0 && <AccountStep />}                      {/* children use useFormContext() */}
        {step === 1 && <ProfileStep />}
        {step === 2 && <ReviewStep />}
        {step > 0 && <button type="button" onClick={() => setStep((s) => s - 1)}>Back</button>}
        {step < steps.length - 1 ? <button type="button" onClick={next}>Next</button> : <button>Submit</button>}
      </form>
    </FormProvider>
  );
}
```

Interview-ready answer: *"A step index in state, one step rendered at a time, and validation scoped to the current step. With a single useForm and FormProvider I call trigger on that step's field names before advancing, so going back keeps everything automatically. Alternatively each step has its own form and schema, and I merge the data into shared state. Either way, the final submit validates the whole thing again."*

**Q: How are nested objects handled?**
A: With **dot notation** in `register` (`'address.city'`, and `'items.0.name'` for arrays). RHF builds the nested object for you, matched by a **nested `z.object()`** schema. Errors follow the same shape (`errors.address?.city`).

```jsx
const schema = z.object({
  name: z.string().min(1),
  address: z.object({
    street: z.string().min(1, 'Required'),
    city: z.string().min(1, 'Required'),
    zip: z.string().regex(/^\d{6}$/, '6-digit PIN code'),
  }),
});

<input {...register('address.street')} />
<input {...register('address.city')} />
{errors.address?.city && <span>{errors.address.city.message}</span>}

// onSubmit receives: { name: '…', address: { street: '…', city: '…', zip: '…' } }
```

Interview-ready answer: *"Dot-notation paths in register, like `address.city`, which RHF turns into a nested object on submit. A nested Zod object schema validates it, and errors come back in the same nested shape. Arrays use the same notation with an index, like `items.0.price`."*

**Q (Scenario): A travel booking form has customer info, multiple passengers, dynamic hotel selections, conditional fields, document uploads and complex validation. How would you architect it?**
A: Don't manage every field with its own `useState`; that means dozens of states and a re-render on every keystroke. Use **React Hook Form**, which is uncontrolled and fast, with **schema validation in Zod** (or Yup), so the validation rules live in one typed schema.

```jsx
import { useForm, useFieldArray } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const passengerSchema = z.object({
  name: z.string().min(3, 'Name is required'),
  age: z.number({ invalid_type_error: 'Age is required' }).min(0),
});

const schema = z.object({
  email: z.string().email('Invalid email'),
  passengers: z.array(passengerSchema).min(1, 'Add at least one passenger'),
  needsHotel: z.boolean(),
  hotelId: z.string().optional(),
}).refine((d) => !d.needsHotel || !!d.hotelId, {   // conditional validation
  path: ['hotelId'],
  message: 'Select a hotel',
});

function BookingForm() {
  const { register, control, watch, handleSubmit, formState: { errors, isSubmitting } } = useForm({
    resolver: zodResolver(schema),
    defaultValues: { passengers: [{ name: '', age: undefined }], needsHotel: false },
  });
  const { fields, append, remove } = useFieldArray({ control, name: 'passengers' });
  const needsHotel = watch('needsHotel');

  return (
    <form onSubmit={handleSubmit(submitBooking)}>
      <input {...register('email')} />
      <p>{errors.email?.message}</p>

      {fields.map((field, i) => (
        <div key={field.id}>
          <input {...register(`passengers.${i}.name`)} />
          <input type="number" {...register(`passengers.${i}.age`, { valueAsNumber: true })} />
          <button type="button" onClick={() => remove(i)}>Remove</button>
          <p>{errors.passengers?.[i]?.name?.message}</p>
        </div>
      ))}
      <button type="button" onClick={() => append({ name: '', age: undefined })}>Add passenger</button>

      <label><input type="checkbox" {...register('needsHotel')} /> Need a hotel?</label>
      {needsHotel && <select {...register('hotelId')}>{/* options */}</select>}
      <p>{errors.hotelId?.message}</p>

      <button type="submit" disabled={isSubmitting}>Book</button>
    </form>
  );
}
```

Senior-level insight: for production I would also add:
1. **Reusable field components** (`<TextField name label />` using `useFormContext`) so every form looks and behaves the same.
2. `useFieldArray` for dynamic sections, using `field.id` as the key, not the index.
3. Conditional validation through `refine`/`superRefine` or discriminated unions.
4. **Server-side validation** with the same Zod schema shared with a Node backend. Map server errors back with `setError`.
5. File uploads with progress, size and type checks, and retry.
6. Preventing **duplicate submissions** (`isSubmitting`, plus an idempotency key on the API).
7. **Draft saving** for long forms (debounced `watch` written to localStorage or the server).
8. A multi-step wizard that validates each step with `trigger()`.

Client-side validation is for UX. **Backend validation is still mandatory.**

### [A5.4 — File Uploads & Rich Text](A5.4-file-uploads-and-rich-text.md)

**Q: Why can't file inputs be controlled normally?**
A: For security, the browser makes a file input's **value read-only**: JavaScript can't set it to an arbitrary path, so `value={…}` can't work. Read the selected `File` objects from **`e.target.files`** in `onChange`, then store them in state or hand them to the form with `setValue` (or use RHF's `register`, which gives you a `FileList`). Validate the size and type yourself.

```jsx
function AvatarField({ setValue, error }) {
  const handleChange = (e) => {
    const file = e.target.files?.[0];
    if (!file) return;
    if (!['image/png', 'image/jpeg', 'image/webp'].includes(file.type)) return alert('PNG, JPEG or WebP only');
    if (file.size > 2 * 1024 * 1024) return alert('Max 2 MB');
    setValue('avatar', file, { shouldValidate: true });
  };
  return <input type="file" accept="image/png,image/jpeg,image/webp" onChange={handleChange} />;
}

// Uploading: send a File with FormData (don't set Content-Type; the browser adds the boundary)
const body = new FormData();
body.append('avatar', data.avatar);
await fetch('/api/avatar', { method: 'POST', body });
```

Interview-ready answer: *"The browser forbids setting a file input's value from script, so file inputs are always uncontrolled. I read e.target.files in onChange, validate type and size client-side for UX, store the File in state or the form via setValue, and upload it with FormData. The server must re-validate, since client checks can be bypassed."*

**Q: How do you preview an image before upload?**
A: `URL.createObjectURL(file)` creates a temporary **in-memory URL** (`blob:...`) pointing at the local file, with no upload and no base64 conversion. **Revoke it** with `URL.revokeObjectURL(url)` when the file changes or the component unmounts, or the memory leaks.

```jsx
function ImagePreview({ file }) {
  const [url, setUrl] = useState(null);

  useEffect(() => {
    if (!file) return;
    const objectUrl = URL.createObjectURL(file);
    setUrl(objectUrl);
    return () => URL.revokeObjectURL(objectUrl);   // cleanup: free memory on change or unmount
  }, [file]);

  return url ? <img src={url} alt="Preview" width={160} /> : null;
}
```

Interview-ready answer: *"URL.createObjectURL gives a blob URL for the local file that an img tag can show instantly. It's cheaper than reading the file as base64 with FileReader. I create it in an effect and revoke it in the cleanup when the file changes or the component unmounts, otherwise the memory isn't released."*

**Q: What is TipTap?**
A: A **headless rich-text editor** framework built on **ProseMirror**. It provides the editing engine (document model, commands, history, extensions like bold, lists, links and images) but **no UI**, so you build the toolbar and style everything to match your design system. Content can be exported as HTML or JSON.

```jsx
import { useEditor, EditorContent } from '@tiptap/react';
import StarterKit from '@tiptap/starter-kit';
import Link from '@tiptap/extension-link';

function RichTextEditor({ content, onChange }) {
  const editor = useEditor({
    extensions: [StarterKit, Link.configure({ openOnClick: false })],
    content,
    onUpdate: ({ editor }) => onChange(editor.getHTML()),
  });
  if (!editor) return null;

  return (
    <div className="editor">
      <div role="toolbar" aria-label="Formatting">
        <button type="button" aria-pressed={editor.isActive('bold')} onClick={() => editor.chain().focus().toggleBold().run()}>B</button>
        <button type="button" aria-pressed={editor.isActive('bulletList')} onClick={() => editor.chain().focus().toggleBulletList().run()}>• List</button>
      </div>
      <EditorContent editor={editor} />
    </div>
  );
}
```

Interview-ready answer: *"TipTap is a headless editor built on ProseMirror. It handles the hard parts (document model, selection, undo history, extensions) and leaves the UI to me, so the toolbar and styling match our design system. Content comes out as HTML or JSON, and HTML must be sanitised before it's rendered elsewhere, to prevent XSS."*

**Q: How does TipTap integrate with React Hook Form?**
A: TipTap isn't a native input, so wire it through **`Controller`** (or `setValue`): call `editor.getHTML()` (or `getJSON()`) in TipTap's `onUpdate` and pass it to the field's `onChange`, so RHF validation and dirty tracking work. Validate on the *text* content, because an "empty" editor still produces `<p></p>`.

```jsx
<Controller
  name="body"
  control={control}
  rules={{ validate: (html) => stripTags(html).trim().length >= 50 || 'Write at least 50 characters' }}
  render={({ field, fieldState }) => (
    <>
      <RichTextEditor content={field.value} onChange={field.onChange} />
      {fieldState.error && <p role="alert">{fieldState.error.message}</p>}
    </>
  )}
/>
```

Interview-ready answer: *"I wrap the editor in RHF's Controller. TipTap's onUpdate passes editor.getHTML() into field.onChange, and field.value seeds the initial content, so the editor behaves like any other field with validation and dirty state. Validation runs on the stripped text, since an empty editor still outputs a paragraph tag, and the HTML is sanitised server-side."*

### [P5.1 — Multi-step Registration Form](P5.1-multi-step-registration-form.md)

**Q: What are the steps?**
A: **Account** (email, password, confirm), **Personal** (name, DOB, phone), **Address**, and **Review** (a read-only summary plus submit), with a **progress indicator**. Each data step has its own schema. The review step shows everything and lets the user jump back to edit.

```jsx
const STEPS = ['Account', 'Personal', 'Address', 'Review'];

function Progress({ current }) {
  return (
    <ol className="steps" aria-label="Registration progress">
      {STEPS.map((label, i) => (
        <li key={label} aria-current={i === current ? 'step' : undefined} className={i <= current ? 'done' : ''}>
          {label}
        </li>
      ))}
    </ol>
  );
}

function ReviewStep({ data, goTo, onSubmit, isSubmitting }) {
  return (
    <>
      <dl><dt>Email</dt><dd>{data.email}</dd><dt>Name</dt><dd>{data.name}</dd></dl>
      <button type="button" onClick={() => goTo(0)}>Edit account</button>
      <button onClick={onSubmit} disabled={isSubmitting}>Create account</button>
    </>
  );
}
```

Interview-ready answer (project walkthrough): *"A four-step registration (account, personal details, address, and a read-only review) with an accessible progress indicator using aria-current. Each step validates independently, data persists when moving back and forth, and the review step lets users jump back to any section before submitting."*

**Q: How is each step validated?**
A: Each step has its **own `useForm` with `zodResolver(stepSchema)`**, and "Next" is just that form's `handleSubmit`. The user can't advance until the step's fields pass. The final submit validates the merged data against the **combined schema** as a last check.

```jsx
const accountSchema = z.object({ email: z.string().email(), password: z.string().min(8), confirmPassword: z.string() })
  .refine((d) => d.password === d.confirmPassword, { message: 'Passwords must match', path: ['confirmPassword'] });
const personalSchema = z.object({ name: z.string().min(2), dob: z.coerce.date(), phone: z.string().regex(/^\d{10}$/) });

function AccountStep({ defaultValues, onNext }) {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(accountSchema),
    defaultValues,
  });
  return (
    <form onSubmit={handleSubmit(onNext)}>          {/* onNext runs only if this step is valid */}
      <input {...register('email')} />
      <input type="password" {...register('password')} />
      <input type="password" {...register('confirmPassword')} />
      {errors.confirmPassword && <p role="alert">{errors.confirmPassword.message}</p>}
      <button>Next</button>
    </form>
  );
}
```

Interview-ready answer: *"Every step is its own small form with its own Zod schema through zodResolver, and Next is that form's handleSubmit. Invalid steps simply can't advance, and errors show inline. On the final submit I validate the merged object against the full schema again, and the server validates once more."*

**Q: How is "passwords must match" validated?**
A: With **`.refine()` on the account object schema** (cross-field rules need both values), using **`path: ['confirmPassword']`** so the error shows under the confirm field rather than on the form as a whole.

```ts
const accountSchema = z
  .object({
    email: z.string().email('Enter a valid email'),
    password: z.string().min(8, 'At least 8 characters').regex(/[A-Z]/, 'One uppercase letter').regex(/\d/, 'One number'),
    confirmPassword: z.string(),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: 'Passwords must match',
    path: ['confirmPassword'],          // attach the error to this field
  });
```

To re-check the match when `password` changes after `confirmPassword` was filled in, call `trigger('confirmPassword')` in an effect, or use `mode: 'onTouched'`.

Interview-ready answer: *"It's a cross-field rule, so it goes in a refine on the object schema, comparing password and confirmPassword, with path pointing at confirmPassword so the message shows under that input. I also re-trigger that field's validation when the password changes, so a stale 'match' can't slip through."*

**Q: How is data preserved when going back?**
A: A **`useRegistration` hook** holds the accumulated `formData` and the current step. Each step's valid data is **merged** in on "Next", and every step receives `formData` as its **`defaultValues`**, so returning to a step shows what was entered. Optionally the data is persisted to `sessionStorage` so a refresh doesn't lose progress (never the password).

```jsx
function useRegistration() {
  const [step, setStep] = useState(0);
  const [formData, setFormData] = useState(() => JSON.parse(sessionStorage.getItem('reg') ?? '{}'));

  const saveStep = (data) => {
    setFormData((prev) => {
      const next = { ...prev, ...data };
      const { password, confirmPassword, ...safe } = next;            // never persist passwords
      sessionStorage.setItem('reg', JSON.stringify(safe));
      return next;
    });
    setStep((s) => s + 1);
  };
  const back = () => setStep((s) => s - 1);
  return { step, formData, saveStep, back, goTo: setStep };
}

// <PersonalStep defaultValues={formData} onNext={saveStep} onBack={back} />
```

Interview-ready answer: *"A useRegistration hook owns the merged form data and the step index. Each step submits its validated slice, which is merged in, and every step is mounted with the merged data as defaultValues, so going back shows previous answers. I optionally mirror it to sessionStorage without passwords, so a refresh doesn't lose progress."*

### [P5.2 — CMS-style Post Editor](P5.2-cms-style-post-editor.md)

**Q: What are the main features?**
A: A title with a character count, a TipTap body with a toolbar, an auto-generated editable slug, keyboard tag input, cover image drag-and-drop with preview, autosave status, a Draft/Publish toggle, and manual save.

```jsx
function PostEditor({ post }) {
  const methods = useForm({ resolver: zodResolver(postSchema), defaultValues: post });
  const values = methods.watch();
  const { status } = useAutosave(values, (v) => api.saveDraft(post.id, v));   // 'saving' | 'saved' | 'error'

  return (
    <FormProvider {...methods}>
      <TitleField max={80} />                        {/* shows "42 / 80" */}
      <SlugField />                                  {/* auto from title, editable */}
      <Controller name="body" control={methods.control} render={({ field }) => <RichTextEditor content={field.value} onChange={field.onChange} />} />
      <TagInput name="tags" />
      <CoverDropzone name="cover" />
      <StatusToggle name="status" />                 {/* draft | published */}
      <span aria-live="polite">{status === 'saving' ? 'Saving…' : status === 'saved' ? 'All changes saved' : ''}</span>
      <button onClick={methods.handleSubmit((v) => api.save(post.id, v))}>Save</button>
    </FormProvider>
  );
}
```

Interview-ready answer (project walkthrough): *"A CMS-style editor: a title with a live character count, a TipTap rich-text body, a slug generated from the title but editable, a keyboard-driven tag input, cover image drag-and-drop with preview, a draft/publish status, manual save, and debounced autosave with an accessible status indicator. It's all one React Hook Form with a Zod schema."*

**Q: How does autosave work?**
A: A `useAutosave` hook **debounces** with a `setTimeout` stored in a ref, saving 2 s (2000 ms by default) after the user stops typing, and exposes a status (`saving`, `saved`, `error`). The original design left `saveFn` out of the deps so a new function identity doesn't re-trigger saving. A cleaner way is to keep the **latest `saveFn` in a ref**, which avoids both the stale closure and the lint warning.

```jsx
function useAutosave(data, saveFn, delay = 2000) {
  const [status, setStatus] = useState('idle');
  const saveRef = useRef(saveFn);
  useEffect(() => { saveRef.current = saveFn; });               // always the latest function

  const first = useRef(true);
  useEffect(() => {
    if (first.current) { first.current = false; return; }       // don't save on mount
    const id = setTimeout(async () => {
      setStatus('saving');
      try {
        await saveRef.current(data);
        setStatus('saved');
      } catch {
        setStatus('error');
      }
    }, delay);
    return () => clearTimeout(id);                              // each change resets the timer
  }, [JSON.stringify(data), delay]);                            // deep compare (fine for small forms)

  return { status };
}
```

Interview-ready answer: *"It's a debounced effect: every change resets a timeout, and two seconds after the last edit it calls the save function and updates a status the UI announces through an aria-live region. I keep the latest save function in a ref rather than in the dependencies, so a new callback identity doesn't trigger extra saves and it never calls a stale one. A production version would also handle out-of-order responses and save on page unload."*

**Q: How does the tag input handle the keyboard?**
A: **Enter or comma** adds the typed tag (trimmed, deduplicated, and ignored if empty), with `preventDefault` so Enter doesn't submit the form. **Backspace** in an **empty** input removes the last tag. Each chip also has a remove button for mouse and screen-reader users.

```jsx
function TagInput({ value: tags, onChange, max = 10 }) {
  const [text, setText] = useState('');

  const add = () => {
    const t = text.trim().toLowerCase();
    if (t && !tags.includes(t) && tags.length < max) onChange([...tags, t]);
    setText('');
  };

  const handleKeyDown = (e) => {
    if (e.key === 'Enter' || e.key === ',') { e.preventDefault(); add(); }
    else if (e.key === 'Backspace' && text === '' && tags.length) onChange(tags.slice(0, -1));
  };

  return (
    <div className="tags">
      {tags.map((t) => (
        <span key={t} className="chip">{t}
          <button type="button" aria-label={`Remove ${t}`} onClick={() => onChange(tags.filter((x) => x !== t))}>×</button>
        </span>
      ))}
      <input value={text} onChange={(e) => setText(e.target.value)} onKeyDown={handleKeyDown} onBlur={add}
        aria-label="Add tag" placeholder="Add a tag…" />
    </div>
  );
}
```

Interview-ready answer: *"onKeyDown handles it: Enter or comma adds the trimmed tag, after preventDefault so the form doesn't submit, skipping duplicates and empties. Backspace on an empty input removes the last tag. Each chip has a labelled remove button, so it works for mouse and screen-reader users too, and the tags are an array value wired into the form."*

**Q: How is the slug generated?**
A: Automatically from the title via a `slugify` utility (lowercase, accents stripped, non-alphanumerics turned into dashes, trimmed), **until the user edits the slug by hand**. A "touched" flag then stops the auto-sync, so a title tweak never overwrites a custom slug.

```js
export function slugify(input) {
  return input
    .normalize('NFKD').replace(/[̀-ͯ]/g, '')   // é → e
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, '-')                           // spaces, punctuation → -
    .replace(/^-+|-+$/g, '')                               // trim dashes
    .slice(0, 80);
}
```

```jsx
function SlugField() {
  const { watch, setValue, register, getFieldState } = useFormContext();
  const title = watch('title');

  useEffect(() => {
    if (!getFieldState('slug').isDirty) setValue('slug', slugify(title));   // only while the user hasn't edited it
  }, [title, setValue, getFieldState]);

  return <input {...register('slug', { setValueAs: slugify })} />;          // also normalise manual edits
}
```

Interview-ready answer: *"A pure slugify function normalises the title: lowercase, accents stripped, non-alphanumerics turned into dashes, trimmed and length-capped. An effect keeps the slug in sync with the title only until the user edits the slug field, using RHF's dirty state, after which their custom slug is respected. Manual edits are normalised too, and uniqueness is checked on the server."*

---

## Phase 6 — Styling & UI

### [A6.1 — Tailwind CSS with React](A6.1-tailwind-css-with-react.md)

**Q: What is utility-first CSS?**
A: Styling with small, single-purpose classes written directly in JSX (`className="p-4 rounded-lg"`) instead of writing custom CSS classes in separate files.

Benefits:
- **Co-location:** the styles sit with the markup they style.
- **No naming** of classes, and no specificity battles.
- **A consistent design scale** for spacing, colours and type.
- **Tiny production CSS:** Tailwind only generates the classes you use.
- **Safe deletion:** removing markup removes its styles.

Trade-offs:
- Long class strings.
- A learning curve.
- Repeated patterns should be extracted into **components**, not `@apply`.

```jsx
// Traditional CSS: .card { padding: 1rem; border-radius: .5rem; box-shadow: ...; } in another file
// Utility-first:
function Card({ title, children }) {
  return (
    <div className="rounded-lg border bg-white p-4 shadow-sm hover:shadow-md transition-shadow">
      <h3 className="mb-2 text-lg font-semibold text-gray-900">{title}</h3>
      <div className="text-sm text-gray-600">{children}</div>
    </div>
  );
}
```

Interview-ready answer: *"Utility-first CSS means composing designs from small single-purpose classes in the markup, like Tailwind's p-4 or text-lg, instead of writing bespoke CSS. Styles live next to the component, follow a consistent design scale, never fight specificity, and produce a small CSS bundle. Long class lists are the trade-off, which I manage by extracting reusable components and using variant helpers like CVA."*

**Q: How do responsive and dark mode work?**
A: Tailwind is **mobile-first**. Unprefixed classes apply everywhere, and the prefixes `sm:`, `md:`, `lg:`, `xl:` and `2xl:` apply **from that breakpoint up**. `dark:` applies in dark mode, which by default follows the OS (`prefers-color-scheme`). It can also be driven manually by a `dark` class on `<html>`, for a user toggle.

```jsx
<div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-4">        {/* 1 → 2 → 4 columns */}
  <div className="bg-white text-gray-900 dark:bg-gray-900 dark:text-gray-100 p-4 md:p-6">…</div>
</div>
```

```css
/* Tailwind v4: switch dark mode to class-based (v3: darkMode: 'class' in tailwind.config.js) */
@custom-variant dark (&:where(.dark, .dark *));
```

```js
// Toggle
document.documentElement.classList.toggle('dark', theme === 'dark');
```

Interview-ready answer: *"Breakpoint prefixes are mobile-first: base classes apply to all sizes, and md: or lg: kick in at that width and up, so I design for small screens first. dark: variants apply in dark mode, either from the OS preference or, for a toggle, a dark class on the html element that I set from the saved user preference."*

**Q: What are `clsx` and `cn`?**
A: **`clsx`** builds class strings conditionally from strings, objects and arrays, dropping falsy values. **`cn`** (from shadcn/ui) is `twMerge(clsx(...))`: it also **resolves conflicting Tailwind classes**, keeping the last one. That lets a component's defaults be overridden safely by a `className` prop: `cn('p-4', isPadded && 'p-8')` outputs `p-8`, not both.

```ts
// lib/utils.ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';
export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

```jsx
clsx('btn', isActive && 'btn-active', { 'opacity-50': disabled });   // 'btn btn-active opacity-50'

function Button({ className, ...props }) {
  return <button className={cn('rounded-md bg-blue-600 px-4 py-2 text-white', className)} {...props} />;
}
<Button className="bg-red-600 px-8" />   // → 'rounded-md py-2 text-white bg-red-600 px-8' (conflicts resolved)
```

Interview-ready answer: *"clsx conditionally joins class names. cn wraps clsx with tailwind-merge, which removes conflicting Tailwind utilities so the last one wins. That's what lets reusable components accept a className prop that reliably overrides their defaults, instead of ending up with both px-4 and px-8 and an unpredictable result."*

### [A6.2 — Component Variants](A6.2-component-variants.md)

**Q: What problem does CVA solve?**
A: **Variant explosion.** A button with 4 variants × 3 sizes × 2 states means 24 combinations, usually handled with nested ternaries and string concatenation that are hard to read and easy to break. CVA (class-variance-authority) lets you **declare** base styles, variants, defaults and compound variants in one object, and generates the right `className`, typed.

```jsx
// ❌ Without CVA
const cls = `btn ${variant === 'primary' ? 'bg-blue-600 text-white' : variant === 'danger' ? 'bg-red-600 text-white' : 'bg-gray-100'} ${size === 'sm' ? 'h-8 px-3' : size === 'lg' ? 'h-12 px-6' : 'h-10 px-4'} ${disabled ? 'opacity-50' : ''}`;

// ✅ With CVA: declarative and typed
const button = cva('inline-flex items-center rounded-md font-medium transition-colors', {
  variants: {
    variant: { primary: 'bg-blue-600 text-white hover:bg-blue-700', danger: 'bg-red-600 text-white', ghost: 'hover:bg-gray-100' },
    size: { sm: 'h-8 px-3 text-sm', md: 'h-10 px-4', lg: 'h-12 px-6 text-lg' },
  },
  defaultVariants: { variant: 'primary', size: 'md' },
});
```

Interview-ready answer: *"CVA solves variant explosion in component styling. Instead of tangled ternaries for every combination of variant, size and state, I declare the base classes and each variant's classes once, and CVA composes the className. It's readable, typed through VariantProps, and it's the pattern shadcn/ui uses."*

**Q: What is the CVA API?**
A: `cva(base, { variants, compoundVariants, defaultVariants })` returns a **function**. Call it with `{ variant, size, … }` (plus an optional `class`) to get the class string.

```tsx
import { cva, type VariantProps } from 'class-variance-authority';

const badge = cva('inline-flex rounded-full px-2 py-0.5 text-xs font-medium', {
  variants: {
    tone: { neutral: 'bg-gray-100 text-gray-800', success: 'bg-green-100 text-green-800', danger: 'bg-red-100 text-red-800' },
    outline: { true: 'bg-transparent ring-1 ring-inset', false: '' },
  },
  defaultVariants: { tone: 'neutral', outline: false },
});

badge();                                    // neutral, filled
badge({ tone: 'success' });                 // success, filled
badge({ tone: 'danger', outline: true, class: 'ml-2' });

type BadgeProps = React.HTMLAttributes<HTMLSpanElement> & VariantProps<typeof badge>;
const Badge = ({ tone, outline, className, ...props }: BadgeProps) => (
  <span className={cn(badge({ tone, outline }), className)} {...props} />
);
```

Interview-ready answer: *"cva takes the base classes and a config of variants, compound variants and default variants, and returns a function. I call that function with the variant props to get the className, then merge it with any className override through cn. VariantProps derives the component's prop types from the same config."*

**Q: What are compound variants?**
A: Styles applied **only when a specific combination** of variants is active, for cases a single variant can't express, such as "a ghost button that is *also* danger should have red text", or "small + icon-only should be square".

```ts
const button = cva('rounded-md font-medium', {
  variants: {
    variant: { solid: 'bg-blue-600 text-white', ghost: 'bg-transparent' },
    tone: { default: '', danger: '' },
    size: { sm: 'h-8 px-3', md: 'h-10 px-4' },
    iconOnly: { true: '' },
  },
  compoundVariants: [
    { variant: 'solid', tone: 'danger', class: 'bg-red-600 hover:bg-red-700' },
    { variant: 'ghost', tone: 'danger', class: 'text-red-600 hover:bg-red-50' },
    { size: 'sm', iconOnly: true, class: 'w-8 px-0' },        // a square small icon button
  ],
  defaultVariants: { variant: 'solid', tone: 'default', size: 'md' },
});
```

Interview-ready answer: *"Compound variants add classes only when several variant values occur together. For example, ghost plus danger gets red text, while solid plus danger gets a red background. They express interactions between variants declaratively instead of with extra conditional logic."*

**Q: What is `VariantProps`?**
A: A TypeScript utility, `VariantProps<typeof fn>`, that **extracts the prop types** from a CVA function, giving a union type for each variant. The component's props stay in sync with the style config automatically: add a variant to `cva` and the type updates.

```tsx
const button = cva('…', {
  variants: { variant: { primary: '…', secondary: '…', danger: '…' }, size: { sm: '…', md: '…', lg: '…' } },
});

type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> & VariantProps<typeof button>;
// variant?: 'primary' | 'secondary' | 'danger' | null
// size?: 'sm' | 'md' | 'lg' | null

function Button({ variant, size, className, ...props }: ButtonProps) {
  return <button className={cn(button({ variant, size }), className)} {...props} />;
}

<Button variant="danger" size="lg" />
<Button variant="warning" />   // ❌ TS error: not a valid variant
```

Interview-ready answer: *"VariantProps infers prop types from a cva definition, so variant and size become exact union types without my writing them twice. Adding or removing a variant in the style config updates the component's TypeScript API automatically, and invalid values fail at compile time."*

### [A6.3 — shadcn/ui](A6.3-shadcn-ui.md)

**Q: What is shadcn/ui?**
A: **Not an npm dependency** but a collection of well-built, accessible components you **copy into your project** with a CLI and then **own**. Most are built on **Radix UI** primitives (for behaviour and accessibility), styled with **Tailwind**, with variants via **CVA** and the `cn` helper. Since the code lives in `components/ui`, you edit it like your own.

```bash
npx shadcn@latest init           # sets up Tailwind tokens, lib/utils.ts (cn), components.json
npx shadcn@latest add button dialog dropdown-menu
```

```jsx
import { Button } from '@/components/ui/button';
import { Dialog, DialogTrigger, DialogContent, DialogHeader, DialogTitle } from '@/components/ui/dialog';

<Dialog>
  <DialogTrigger asChild><Button variant="outline">Edit profile</Button></DialogTrigger>
  <DialogContent>
    <DialogHeader><DialogTitle>Edit profile</DialogTitle></DialogHeader>
    <ProfileForm />
  </DialogContent>
</Dialog>
```

Interview-ready answer: *"shadcn/ui isn't a component library you install. It's a set of components whose source a CLI copies into your repo. They're built on Radix primitives for accessibility, Tailwind for styling and CVA for variants. You own and customise the code directly, which makes it a great base for a design system."*

**Q: What are Radix UI primitives?**
A: **Headless, unstyled** React components that correctly implement the **hard accessibility and interaction patterns**:
- Focus-trapping dialogs that restore focus on close.
- Dropdowns with arrow-key navigation that close on Escape.
- Tooltips wired up with `aria-describedby`.
- Proper ARIA roles and states, typeahead, and portals.

You bring all the styling.

```jsx
import * as DropdownMenu from '@radix-ui/react-dropdown-menu';

<DropdownMenu.Root>
  <DropdownMenu.Trigger className="btn">Options</DropdownMenu.Trigger>
  <DropdownMenu.Portal>
    <DropdownMenu.Content className="rounded-md bg-white p-1 shadow-lg" sideOffset={4}>
      <DropdownMenu.Item className="px-2 py-1 data-[highlighted]:bg-gray-100" onSelect={edit}>Edit</DropdownMenu.Item>
      <DropdownMenu.Item className="px-2 py-1 text-red-600" onSelect={remove}>Delete</DropdownMenu.Item>
    </DropdownMenu.Content>
  </DropdownMenu.Portal>
</DropdownMenu.Root>
// Arrow keys, Enter, Escape, focus management and ARIA all handled. State is exposed through data-* attributes for styling.
```

Interview-ready answer: *"Radix primitives are unstyled components that implement complex WAI-ARIA patterns correctly: dialogs, menus, popovers, tabs, tooltips, with keyboard navigation, focus trapping and screen-reader semantics. They handle behaviour and accessibility, which is very hard to get right yourself, and leave the visuals entirely to me, usually through Tailwind."*

**Q: Why is this better than Material UI or Ant Design?**
A: You **own the source**, so you can change any class, add props, and match your design system exactly, without fighting theme overrides or `!important`. There's **no runtime styling library** and no lock-in to someone else's update cycle or breaking changes, and you include only the components you use. The trade-off is that **you maintain** that code, and MUI and AntD ship far more ready-made components (data grids, date pickers), which suits internal tools that need breadth fast.

```jsx
// MUI: customising often means theme overrides or sx props fighting the defaults
<Button sx={{ borderRadius: 0, textTransform: 'none', '&:hover': { bgcolor: '#111' } }} />

// shadcn: open components/ui/button.tsx and edit the cva classes directly, done
```

Interview-ready answer: *"With shadcn/ui I own the component code, so customising it to a custom design system is just editing Tailwind classes. There's no theme-override fight, no CSS-in-JS runtime and no dependency upgrade cycle, while Radix keeps it accessible. MUI or AntD are better when I need a huge set of ready-made components quickly and the default look is fine. It depends on whether brand fidelity or breadth matters more."*

### [A6.4 — Framer Motion](A6.4-framer-motion.md)

**Q: How are animations declared?**
A: **Declaratively.** Replace an element with its `motion.*` version and describe states such as `initial`, `animate`, `exit`, `whileHover` and `whileTap`. Framer Motion (now published as "Motion") animates between them with spring physics. **Variants** name states and propagate through children for orchestration (`staggerChildren`). This is simpler than juggling CSS transitions or `requestAnimationFrame`.

```jsx
import { motion } from 'framer-motion';

<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.4, ease: 'easeOut' }}
  whileHover={{ scale: 1.03 }}
  whileTap={{ scale: 0.97 }}
/>

// Variants + stagger
const list = { hidden: {}, show: { transition: { staggerChildren: 0.08 } } };
const item = { hidden: { opacity: 0, y: 10 }, show: { opacity: 1, y: 0 } };

<motion.ul variants={list} initial="hidden" animate="show">
  {items.map((i) => <motion.li key={i.id} variants={item}>{i.name}</motion.li>)}
</motion.ul>
```

Interview-ready answer: *"Framer Motion is declarative: I use motion components and describe the initial, animate and exit states, plus gesture states like whileHover, and it interpolates between them. Variants name reusable states and cascade to children, which makes staggered, orchestrated animations easy. It animates transform and opacity for performance, and I respect prefers-reduced-motion."*

**Q: Why is `AnimatePresence` needed?**
A: React **removes elements from the DOM immediately** when they unmount, so there's no time to animate out. `AnimatePresence` **keeps an exiting child mounted** until its `exit` animation finishes. It's essential for modals, toasts, list removals and page transitions. Each direct child needs a stable, unique `key`.

```jsx
import { AnimatePresence, motion } from 'framer-motion';

function Modal({ isOpen, onClose, children }) {
  return (
    <AnimatePresence>
      {isOpen && (
        <motion.div
          key="modal"
          className="fixed inset-0 grid place-items-center bg-black/50"
          initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }}
          onClick={onClose}
        >
          <motion.div initial={{ scale: 0.95 }} animate={{ scale: 1 }} exit={{ scale: 0.95 }} onClick={(e) => e.stopPropagation()}>
            {children}
          </motion.div>
        </motion.div>
      )}
    </AnimatePresence>
  );
}
// Route transitions: <AnimatePresence mode="wait"><motion.div key={location.pathname}>…</motion.div></AnimatePresence>
```

Interview-ready answer: *"React unmounts immediately, so exit animations would never play. AnimatePresence detects when a keyed child is removed, keeps it rendered while its exit animation runs, then lets it unmount. I use it for modals, toasts, removing list items, and route transitions with mode wait, keyed by pathname."*

**Q: What are layout animations?**
A: The **`layout`** prop automatically animates an element's **position and size changes caused by re-layout**: reordering, filtering, expanding, changing grid columns. It uses transforms (FLIP), so it stays smooth. **`layoutId`** links two different elements so one **morphs into the other** (a shared element transition), such as a card expanding into a modal or a tab underline sliding.

```jsx
// List reorder or filter: items glide to their new positions
<motion.ul layout>
  {filtered.map((p) => <motion.li layout key={p.id}>{p.name}</motion.li>)}
</motion.ul>

// Sliding tab indicator with layoutId
{tabs.map((t) => (
  <button key={t} onClick={() => setActive(t)} className="relative px-4 py-2">
    {t}
    {active === t && <motion.span layoutId="tab-underline" className="absolute inset-x-0 -bottom-px h-0.5 bg-blue-600" />}
  </button>
))}

// Card → modal: give both the same layoutId
<motion.div layoutId={`card-${id}`} onClick={() => setOpen(id)} />
<AnimatePresence>{open && <motion.div layoutId={`card-${open}`} className="modal" />}</AnimatePresence>
```

Interview-ready answer: *"The layout prop animates changes in an element's size or position caused by re-rendering, such as reordering or filtering, using performant FLIP transforms. layoutId connects two separate elements so Motion morphs one into the other, which is how you build shared-element transitions like an expanding card or a sliding tab indicator."*

### [P6.1 — UI Component Library](P6.1-ui-component-library.md)

**Q: Which components does it include?**
A: Button, Input, Select, Modal, Toast, Table and Badge, plus a demo page that shows every variant and state (a mini Storybook).

```
src/components/ui/
├── button.tsx    // cva variants: primary | secondary | danger | ghost × sm | md | lg, loading state
├── input.tsx     // label, error, helper text, forwards ref
├── select.tsx    // Radix Select
├── modal.tsx     // Radix Dialog
├── toast.tsx     // Radix Toast + a queue
├── table.tsx     // generic, sortable
└── badge.tsx
src/lib/utils.ts  // cn()
src/pages/Demo.tsx
```

```tsx
const Input = ({ label, error, id, className, ref, ...props }: InputProps) => {   // React 19: ref as a prop
  const generatedId = useId();            // always call the hook (Rules of Hooks)
  const inputId = id ?? generatedId;
  return (
    <div className="space-y-1">
      <label htmlFor={inputId} className="text-sm font-medium">{label}</label>
      <input id={inputId} ref={ref} aria-invalid={!!error} aria-describedby={error ? `${inputId}-error` : undefined}
        className={cn('w-full rounded-md border px-3 py-2', error && 'border-red-500', className)} {...props} />
      {error && <p id={`${inputId}-error`} className="text-sm text-red-600">{error}</p>}
    </div>
  );
};
```

Interview-ready answer (project walkthrough): *"A small design-system library: Button, Input, Select, Modal, Toast, Table and Badge, with a demo page showing all variants. Every component accepts a className override through cn, forwards refs and native props, and is accessible: labelled inputs with aria-describedby errors, and Radix for complex widgets."*

**Q: How are variants and accessibility handled?**
A: **CVA** manages the variants (typed through `VariantProps`). Accessibility comes from **Radix**: the Modal uses **Radix Dialog** (focus trap, Escape to close, restoring focus, `aria-modal`, labelled by its title), and Toast uses **Radix Toast** (4 variants with a **queue**, announced via a live region, swipe or Escape to dismiss).

```tsx
function Modal({ open, onOpenChange, title, description, children }: ModalProps) {
  return (
    <Dialog.Root open={open} onOpenChange={onOpenChange}>
      <Dialog.Portal>
        <Dialog.Overlay className="fixed inset-0 bg-black/50" />
        <Dialog.Content className="fixed left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 rounded-lg bg-white p-6">
          <Dialog.Title className="text-lg font-semibold">{title}</Dialog.Title>   {/* labels the dialog */}
          {description && <Dialog.Description>{description}</Dialog.Description>}
          {children}
          <Dialog.Close aria-label="Close" className="absolute right-3 top-3">×</Dialog.Close>
        </Dialog.Content>
      </Dialog.Portal>
    </Dialog.Root>
  );
}

// Toast queue: a context holding an array; Radix renders each toast in an aria-live viewport
const { toast } = useToast();
toast({ variant: 'success', title: 'Saved', description: 'Your changes were saved.' });
```

Interview-ready answer: *"Styling variants come from CVA with types from VariantProps. For accessibility I didn't reinvent hard patterns: Modal is Radix Dialog, which handles the focus trap, Escape, focus restoration and labelling, and Toast is Radix Toast with a queue in context, announced through a live region. Simple components get correct native semantics and ARIA by hand."*

**Q: What is special about the Table?**
A: It's a **typed generic** component: `Table<T>` with column definitions typed against the row type (`accessor: keyof T`), so passing a wrong column key is a compile error. It supports **sortable columns** (accessible via `aria-sort`) and striped rows, with sorting computed in a `useMemo`.

```tsx
type Column<T> = { key: keyof T; header: string; sortable?: boolean; render?: (row: T) => React.ReactNode };

function Table<T extends { id: string | number }>({ data, columns }: { data: T[]; columns: Column<T>[] }) {
  const [sort, setSort] = useState<{ key: keyof T; dir: 'asc' | 'desc' } | null>(null);

  const rows = useMemo(() => {
    if (!sort) return data;
    return [...data].sort((a, b) => {
      const r = String(a[sort.key]).localeCompare(String(b[sort.key]), undefined, { numeric: true });
      return sort.dir === 'asc' ? r : -r;
    });
  }, [data, sort]);

  const toggle = (key: keyof T) =>
    setSort((s) => (s?.key === key ? { key, dir: s.dir === 'asc' ? 'desc' : 'asc' } : { key, dir: 'asc' }));

  return (
    <table className="w-full">
      <thead><tr>{columns.map((c) => (
        <th key={String(c.key)} aria-sort={sort?.key === c.key ? (sort.dir === 'asc' ? 'ascending' : 'descending') : 'none'}>
          {c.sortable ? <button onClick={() => toggle(c.key)}>{c.header}</button> : c.header}
        </th>
      ))}</tr></thead>
      <tbody>{rows.map((row) => (
        <tr key={row.id} className="odd:bg-gray-50">{columns.map((c) => <td key={String(c.key)}>{c.render ? c.render(row) : String(row[c.key])}</td>)}</tr>
      ))}</tbody>
    </table>
  );
}

<Table<User> data={users} columns={[{ key: 'name', header: 'Name', sortable: true }, { key: 'email', header: 'Email' }]} />
```

Interview-ready answer: *"It's a generic TypeScript component: column keys are typed against the row type, so a typo in a column key is a compile error, and custom render functions get a typed row. Sorting is memoised, toggles ascending and descending through header buttons, and exposes aria-sort for screen readers. Striped rows use Tailwind's odd: variant."*

**Q: What does the `cn` helper do here?**
A: It lives in **`lib/utils.ts`** and merges class names for every component: `clsx` for conditional classes plus `tailwind-merge` to resolve conflicts. Each component composes its base and variant classes with the consumer's `className`, so overrides **reliably win** (`<Button className="px-8" />` replaces `px-4` rather than conflicting with it).

```ts
// lib/utils.ts
export const cn = (...inputs: ClassValue[]) => twMerge(clsx(inputs));

// In every component
<button className={cn(buttonVariants({ variant, size }), isLoading && 'cursor-wait opacity-70', className)} />
```

Interview-ready answer: *"cn is the shared class-merging utility: clsx for conditionals and tailwind-merge for conflicts. Every component builds its className with it, combining the CVA variant classes, state classes and the consumer's className, so custom overrides always win over defaults without duplicates."*

### [P6.2 — Animated Portfolio](P6.2-animated-portfolio.md)

**Q: What Framer Motion APIs does it practise?**
A:
- **`motion.div`** for declarative enter and hover animations.
- **`AnimatePresence`** for exit animations (page transitions, a mobile menu).
- **`variants`**, staggered across children (hero text, project cards).
- **`useInView`** to trigger animations when sections scroll into view.
- **`useScroll`** (plus `useTransform`) for scroll-linked effects: a header blur and a progress bar.

```jsx
function Projects({ projects }) {
  const container = { hidden: {}, show: { transition: { staggerChildren: 0.1 } } };
  const card = { hidden: { opacity: 0, y: 30 }, show: { opacity: 1, y: 0 } };
  return (
    <motion.section variants={container} initial="hidden" whileInView="show" viewport={{ once: true, amount: 0.2 }}>
      {projects.map((p) => <motion.article key={p.id} variants={card} whileHover={{ y: -4 }}>{p.title}</motion.article>)}
    </motion.section>
  );
}

// Scroll progress bar
const { scrollYProgress } = useScroll();
<motion.div className="fixed left-0 top-0 h-1 bg-blue-600 origin-left" style={{ scaleX: scrollYProgress }} />
```

Interview-ready answer (project walkthrough): *"A portfolio built to practise Motion: motion components with variants and staggered children, whileInView and useInView for scroll-triggered reveals, useScroll with useTransform for scroll-linked effects like a progress bar and header blur, and AnimatePresence for the menu and route exits. Everything respects prefers-reduced-motion."*

**Q: How do the sticky header and skill bars animate?**
A: The **header** maps scroll position to style with `useScroll` + `useTransform`: backdrop blur and shadow increase as the page scrolls (as motion values, so there are **no React re-renders** per scroll frame). **Skill bars** use `useInView`, animating their width from 0 to their percentage the first time they enter the viewport.

```jsx
function Header() {
  const { scrollY } = useScroll();
  const blur = useTransform(scrollY, [0, 120], ['blur(0px)', 'blur(12px)']);
  const shadow = useTransform(scrollY, [0, 120], ['0 0 0 rgba(0,0,0,0)', '0 4px 20px rgba(0,0,0,0.08)']);
  return <motion.header className="sticky top-0 z-50" style={{ backdropFilter: blur, boxShadow: shadow }}>…</motion.header>;
}

function SkillBar({ name, level }) {
  const ref = useRef(null);
  const inView = useInView(ref, { once: true, margin: '-50px' });
  return (
    <div ref={ref}>
      <span>{name}</span>
      <div className="h-2 rounded bg-gray-200">
        <motion.div className="h-2 rounded bg-blue-600"
          initial={{ width: 0 }} animate={{ width: inView ? `${level}%` : 0 }} transition={{ duration: 1, ease: 'easeOut' }} />
      </div>
    </div>
  );
}
```

Interview-ready answer: *"The header maps scrollY to blur and shadow with useTransform. These are motion values applied straight to styles, so scrolling doesn't re-render React. Skill bars use useInView with once: true, animating their width from zero to the skill level the first time they scroll into view."*

**Q: How does the dark mode hook pick its initial value?**
A: From the saved `theme` in localStorage, or, if none is saved, the OS **`prefers-color-scheme: dark`** setting. Changes are saved back and applied as a `dark` class on `<html>`. To avoid a **flash of the wrong theme** on load, a tiny inline script in `index.html` applies the class before React renders.

```jsx
function useDarkMode() {
  const [theme, setTheme] = useState(() => {
    const saved = localStorage.getItem('theme');
    if (saved === 'light' || saved === 'dark') return saved;
    return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
  });

  useEffect(() => {
    document.documentElement.classList.toggle('dark', theme === 'dark');
    localStorage.setItem('theme', theme);
  }, [theme]);

  return [theme, () => setTheme((t) => (t === 'dark' ? 'light' : 'dark'))];
}
```

```html
<!-- index.html: runs before paint, so there's no flash -->
<script>
  const t = localStorage.getItem('theme');
  if (t === 'dark' || (!t && matchMedia('(prefers-color-scheme: dark)').matches)) document.documentElement.classList.add('dark');
</script>
```

Interview-ready answer: *"Lazy state initialisation: use the saved preference if there is one, otherwise the OS prefers-color-scheme setting. An effect toggles the dark class on the html element and saves the choice. A small inline script in index.html applies the class before first paint, which prevents the flash of the light theme that you'd otherwise see before React loads."*

---

## Phase 7 — Performance

### [A7.1 — React.memo & Re-renders](A7.1-react-memo-and-re-renders.md)

**Q: When does a component re-render?**
A: In three cases: (1) its **own state** changes, (2) a **context** it consumes changes, or (3) its **parent re-renders**. By default **all children re-render** even if their props didn't change. Props changing is *not* a separate trigger: props only change because the parent re-rendered.

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  return (
    <>
      <button onClick={() => setCount((c) => c + 1)}>{count}</button>
      <Child />                 {/* re-renders on every click, despite having no props */}
      <MemoChild />             {/* skipped: React.memo and no changed props */}
    </>
  );
}
const Child = () => { console.log('Child render'); return <p>child</p>; };
const MemoChild = React.memo(() => { console.log('MemoChild render'); return <p>memo</p>; });
```

A re-render isn't a DOM update: React still diffs, and commits only what changed.

Interview-ready answer: *"A component re-renders when its state changes, when a context it reads changes, or when its parent re-renders, which by default re-renders all of its children regardless of props. Re-rendering just means calling the function again; the DOM is only touched if the output differs."*

**Q: When is that a problem?**
A: Only when the child is **expensive to render** (big lists, charts, heavy computation) **and** the parent re-renders **frequently** (on every keystroke, scroll or timer tick), which shows up as input lag or dropped frames. Most re-renders are cheap and by design, so measure with the Profiler before optimising.

```jsx
function SearchPage() {
  const [query, setQuery] = useState('');                 // changes on every keystroke
  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <ExpensiveChart data={data} />                       {/* 50 ms render × every keystroke = laggy typing */}
    </>
  );
}
// Fixes: move `query` state into its own <SearchBox/> (colocation), or wrap ExpensiveChart in React.memo
```

Interview-ready answer: *"Re-renders are normal and usually cheap. They become a problem when an expensive component re-renders on a high-frequency update and the UI janks. I confirm it with the Profiler first, then fix it structurally by moving state down or passing children, and use memo only where it's still needed."*

**Q: Why does an inline object prop defeat `React.memo`?**
A: `React.memo` compares props **shallowly by reference** (`Object.is`). An inline object, array or arrow function, such as `style={{ color: 'red' }}`, `items={[1, 2]}` or `onClick={() => …}`, is a **brand new reference on every render**, so the comparison always says "changed" and the memoised child re-renders anyway.

```jsx
const Avatar = React.memo(({ style, onClick }) => <img style={style} onClick={onClick} />);

function Profile({ user }) {
  // ❌ New object and function every render, so memo never skips
  return <Avatar style={{ borderRadius: '50%' }} onClick={() => openUser(user.id)} />;
}

const avatarStyle = { borderRadius: '50%' };                 // ✅ constant: hoisted outside the component
function ProfileFixed({ user }) {
  const handleClick = useCallback(() => openUser(user.id), [user.id]);   // ✅ stable function
  return <Avatar style={avatarStyle} onClick={handleClick} />;
}
```

Interview-ready answer: *"memo does a shallow reference comparison of props. An inline object, array or function is recreated on every render, so it's never equal to the previous one and memo always re-renders. I hoist constants out of the component, and use useMemo or useCallback for values that depend on props or state."*

**Q: How do you use the Profiler?**
A: In **React DevTools → Profiler**: click **Record**, perform the slow interaction, click **Stop**, then read the **flame graph** (wide or yellow bars are slow renders) and the **ranked chart**. In settings, enable **"Record why each component rendered"** to see whether props, state, hooks or the parent caused each render. **"Highlight updates when components render"** shows re-renders live. For production numbers, use the `<Profiler>` component API.

```jsx
import { Profiler } from 'react';

function onRender(id, phase, actualDuration, baseDuration) {
  // phase: 'mount' | 'update' | 'nested-update'; actualDuration = ms spent rendering this commit
  if (actualDuration > 16) console.warn(`[slow render] ${id} ${phase}: ${actualDuration.toFixed(1)}ms`);
}

<Profiler id="ProductList" onRender={onRender}>
  <ProductList />
</Profiler>
```

Interview-ready answer: *"I record the slow interaction in the React DevTools Profiler, look at the flame graph and ranked view for slow or unnecessary renders, and turn on 'why did this render' to see the cause: props, state, hooks or the parent. Then I fix that specific cause and record again to verify. The Profiler component can also log render timings programmatically."*

**Q: What is `React.memo()`, and how does it prevent unnecessary re-renders?**
A: `React.memo` is a higher-order component that wraps a component. Before re-rendering it, React **shallowly compares** the old and new props (`Object.is` on each prop). If none changed, React skips rendering that component and its subtree and reuses the last output. It does not block re-renders caused by the component's own state or by a context it uses.

```jsx
const ProductItem = React.memo(function ProductItem({ product, onAdd }) {
  return <li onClick={() => onAdd(product.id)}>{product.name}</li>;
});

// Parent: keep the props stable, or memo is useless
const handleAdd = useCallback((id) => addToCart(id), []);
<ProductItem product={product} onAdd={handleAdd} />
```

You can pass a custom comparison as the second argument: `React.memo(Comp, (prev, next) => prev.id === next.id)`.

Interview-ready answer: *"React.memo wraps a component so React shallow-compares its props before re-rendering it. If they're all equal by reference, it reuses the previous output and skips the subtree. It only helps if the parent passes stable props, so it's usually paired with useMemo and useCallback, and it doesn't stop re-renders from the component's own state or context."*

**Q: When should you use `React.memo`, and when not?**
A: Use it for:
- Components that are **expensive to render** (charts, large tables, rich content).
- Components inside **large trees or lists** whose parent re-renders often.
- Components that frequently receive **the same props**.

Avoid it when:
- The component is cheap to render.
- Its props change on almost every render anyway, so the comparison is wasted work.
- It receives `children` or inline objects and functions. These are new references every render, so the memo never skips anything unless the parent stabilises them.

Memoisation has a cost (prop comparison and keeping the previous output), so apply it where the Profiler shows wasted renders, not everywhere.

```jsx
// ✅ Good: a 1,000-row list item; the parent re-renders on every keystroke, and row props rarely change
const Row = React.memo(({ item, onSelect }) => <tr onClick={() => onSelect(item.id)}>…</tr>);

// ❌ Pointless: cheap, and `children` is a new element every render
const Card = React.memo(({ children }) => <div className="card">{children}</div>);
```

Interview-ready answer: *"I use memo for expensive components, or list items, that re-render because of a busy parent while their own props stay the same. I skip it for cheap components, components whose props change every time, or ones that take children or inline props, where it only adds comparison overhead. The Profiler decides, not habit."*

**Q: What is the difference between `React.memo`, `useMemo` and `useCallback`?**
A:

| | `React.memo` | `useMemo` | `useCallback` |
|---|---|---|---|
| Type | Higher-order component | Hook | Hook |
| What it memoises | A **component's rendered output** | A **computed value** | A **function reference** |
| Skips work when | Props are shallowly equal | Dependencies are unchanged | Dependencies are unchanged |
| Used in | Wrapping the child component | Inside a component body | Inside a component body |

They work together. `React.memo` on a child is only effective if the parent passes stable props, so objects and arrays come from `useMemo` and callbacks from `useCallback`.

```jsx
const Chart = React.memo(ChartImpl);                                          // memoised component

function Dashboard({ rows }) {
  const series = useMemo(() => toSeries(rows), [rows]);                       // memoised value
  const onPointClick = useCallback((p) => console.log(p), []);                // memoised function
  return <Chart series={series} onPointClick={onPointClick} />;              // memo actually skips now
}
```

Interview-ready answer: *"React.memo memoises a whole component based on its props. useMemo memoises a value computed inside a component, and useCallback memoises a function reference. The two hooks are what make memo effective, because they keep the props you pass to a memoised child referentially stable."*

**Q: How do you optimise the performance of a React application in production?**
A: Answer it as a systematic process. **Measure first** (React DevTools Profiler, Lighthouse, the Chrome Performance tab), then work through the layers:
1. **Component level:**
   - Keep state close to where it's used, and avoid unnecessary state updates.
   - Split large contexts.
   - Avoid inline object and array props going to memoised children.
   - Use `React.memo`/`useMemo`/`useCallback` where profiling shows a benefit.
2. **Bundle:**
   - Lazy loading and route-level code splitting, plus dynamic `import()` for big libraries.
   - Removing unused or heavy dependencies, and tree-shaking.
   - Analysing the bundle (`rollup-plugin-visualizer`, `source-map-explorer`).
   - Optimised assets: compressed, lazy-loaded images, and always a production build.
3. **Rendering:**
   - Stable, unique keys (not indexes for dynamic lists).
   - Virtualising large lists (`react-window`, TanStack Virtual).
   - `useTransition`/`useDeferredValue` to keep typing responsive during heavy updates.
   - SSR or SSG (for example Next.js) for faster first paint where it matters.
4. **API and data:** debouncing and throttling, caching responses (React Query or SWR), deduplicating requests, and pagination or infinite scroll instead of loading everything.
5. **Production monitoring:**
   - The React DevTools Profiler during development.
   - **Core Web Vitals** (LCP, INP, CLS) through the `web-vitals` package or Lighthouse.
   - Error tracking (Sentry).
   - Real-user monitoring (RUM) to find slow pages and devices.

The key message for the interviewer: optimise for **measured bottlenecks**, not by applying memoisation everywhere. Then measure again to confirm the fix worked.

```jsx
// Real-user Core Web Vitals → analytics
import { onCLS, onINP, onLCP } from 'web-vitals';
[onCLS, onINP, onLCP].forEach((fn) => fn((metric) => navigator.sendBeacon('/analytics', JSON.stringify(metric))));

// Route-level splitting + an image that doesn't block LCP
const Reports = lazy(() => import('./pages/Reports'));
<img src="/hero.avif" width={1200} height={600} fetchPriority="high" alt="" />   // the LCP image: high priority
<img src={thumb} loading="lazy" width={300} height={200} alt={title} />          // below the fold: lazy
```

Interview-ready answer: *"I measure first with the Profiler, Lighthouse and real-user Core Web Vitals, then fix by layer. At the component level: state colocation, split contexts, targeted memoisation. For the bundle: route-level code splitting, lazy imports, removing heavy dependencies. For rendering: stable keys, virtualisation, transitions. For data: caching, deduplication, debouncing and pagination. Then I re-measure to prove the improvement."*

**Q: What are debouncing and throttling?**
A: Both limit how often a function runs during rapid events.
- **Debounce:** run only after the events have **stopped** for X ms. Good for search-as-you-type, autosave and validation.
- **Throttle:** run at most **once every** X ms while events keep firing. Good for scroll, resize, mouse move and infinite scroll.

```jsx
function useDebounce(value, delay = 400) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id); // reset the timer on every keystroke
  }, [value, delay]);
  return debounced;
}

const debouncedQuery = useDebounce(query);
useEffect(() => { if (debouncedQuery) search(debouncedQuery); }, [debouncedQuery]);

// Throttle: at most once per `ms`, kept stable across renders
function useThrottledCallback(fn, ms) {
  const last = useRef(0);
  const fnRef = useRef(fn);
  useEffect(() => { fnRef.current = fn; });
  return useCallback((...args) => {
    const now = Date.now();
    if (now - last.current >= ms) { last.current = now; fnRef.current(...args); }
  }, [ms]);
}
const onScroll = useThrottledCallback(() => setShowTop(window.scrollY > 400), 200);
```

Interview-ready answer: *"Debounce waits until events stop for a set time and then runs once. I use it for search inputs, autosave and validation. Throttle runs at most once per interval while events keep firing, which suits scroll, resize and mouse move. In React I implement them with hooks and refs, so the timer or last-run time survives re-renders and the latest callback is always used."*

**Q (Scenario): A dashboard has 20 components, and typing in a search input re-renders almost all of them. How do you find and fix it?**
A: **Measure first, then fix the cause.**

1. **Identify:** use the React DevTools Profiler with "Record why each component rendered", or the "Highlight updates when components render" setting.
2. **Investigate the usual causes:**
   - State held too high, so search state in `Dashboard` re-renders all of its children.
   - Objects and functions recreated every render and passed as props.
   - Props that change unnecessarily.
   - One large Context whose value changes on every keystroke.
   - Expensive calculations running during render.
3. **Fix, starting with the cheapest structural changes:**
   - **State colocation:** move `search` state into a `SearchBar` component so only that component re-renders.
   - **Composition:** pass static sections as `children` so they aren't re-rendered by the stateful parent.
   - Split the Context.
   - Only then reach for `React.memo` with `useMemo`/`useCallback`.

```jsx
// ❌ Problem: a new `user` object every render breaks React.memo
function Dashboard() {
  const [search, setSearch] = useState('');
  const user = { name: 'Pulkit' };
  return (
    <>
      <input value={search} onChange={(e) => setSearch(e.target.value)} />
      <UserProfile user={user} />
    </>
  );
}

// ✅ Fix: a stable reference plus a memoised child
const UserProfile = React.memo(({ user }) => <h2>{user.name}</h2>);

function Dashboard() {
  const [search, setSearch] = useState('');
  const user = useMemo(() => ({ name: 'Pulkit' }), []);
  return (
    <>
      <input value={search} onChange={(e) => setSearch(e.target.value)} />
      <UserProfile user={user} />
    </>
  );
}

// ✅✅ Often better: colocate state so Dashboard doesn't re-render at all
function SearchBar() {
  const [search, setSearch] = useState('');
  return <input value={search} onChange={(e) => setSearch(e.target.value)} />;
}
```

(For a truly constant object, define it **outside** the component, which needs no hook at all.)

Senior-level insight: don't add memoisation everywhere. Find the actual bottleneck with the Profiler, fix the structure first (colocation and composition), memoise only the components responsible, and profile again to confirm the fix worked.

### [A7.2 — useMemo & useCallback in Practice](A7.2-usememo-and-usecallback-in-practice.md)

**Q: What does memoisation cost?**
A: It isn't free:
- **Memory** to store the previous result and dependencies.
- **Work on every render** to compare the dependencies.
- **Complexity**: more code, dependency arrays to keep right, and stale-closure bugs when they're wrong.

For cheap computations the bookkeeping can cost **more** than recomputing. Memoise only for a **measured problem** or a **referential equality requirement** (props to a memoised child, or effect dependencies).

```jsx
// ❌ Overhead with no benefit: string concatenation is cheap
const fullName = useMemo(() => `${first} ${last}`, [first, last]);
// ✅ Just compute it
const fullName = `${first} ${last}`;

// ✅ Worth it: an expensive calculation, or a stable reference a memoised child needs
const stats = useMemo(() => computeStats(rows), [rows]);   // rows.length = 50,000
```

Interview-ready answer: *"Each useMemo or useCallback costs memory for the cached value, a dependency comparison on every render, and extra code that's easy to get wrong. For cheap work that's slower than just recomputing. So I memoise only when profiling shows an expensive calculation or when referential stability is required, and React Compiler increasingly does this automatically."*

**Q: What is Concurrent Rendering in React?**
A: The ability, built on Fiber and enabled by `createRoot` in React 18, to **prepare multiple versions of the UI** and to **schedule, pause, resume or abandon** rendering work so urgent interactions stay responsive. Before React 18, rendering was synchronous: once started it blocked the main thread until it finished. Now React assigns priorities. Typing, clicking and hovering are **urgent**. Updates marked with `startTransition` or `useDeferredValue` are **non-urgent** and can be interrupted when an urgent update arrives; React throws away the half-finished render and starts again with the latest state.

Example: typing in a search box while thousands of products are filtered. The input update is urgent, and the list update is wrapped in `startTransition`, so keystrokes are never blocked. The code is in the next question.

Features that depend on it: transitions, `useDeferredValue`, Suspense for data fetching, streaming SSR and selective hydration.

Senior-level insight: **concurrent does not mean parallel or multithreaded.** JavaScript still runs on one main thread. React renders in small chunks and **yields** back to the browser between them (cooperative scheduling), so it can handle input in between. A single slow component render still blocks, so heavy computation belongs in `useMemo` or a Web Worker.

```jsx
const root = createRoot(document.getElementById('root'));   // concurrent features are enabled with createRoot
root.render(<App />);

startTransition(() => setTab('reports'));   // non-urgent: interruptible, and keeps the old UI until ready
```

Interview-ready answer: *"Concurrent rendering lets React treat rendering as interruptible, prioritised work instead of one blocking pass. Urgent updates like typing go first, while updates marked as transitions render in the background and can be abandoned if newer input arrives. It powers useTransition, useDeferredValue, Suspense and streaming SSR. It's cooperative scheduling on one thread, not multithreading."*

**Q: What is the difference between `useTransition` and `useDeferredValue`?**
A: Both are React 18 concurrent features. They mark some rendering work as **low priority**, so urgent updates such as typing stay responsive and React can interrupt the slow render.

`useTransition` wraps a **state update** and marks it as non-urgent:

```jsx
function Search() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    const value = e.target.value;
    setQuery(value);                              // urgent: the input updates right away
    startTransition(() => {
      setResults(filterLargeList(value));         // non-urgent: can be interrupted
    });
  }

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && <p>Updating results...</p>}
      <ResultList items={results} />
    </>
  );
}
```

`useDeferredValue` takes a **value** and returns a copy that lags behind during heavy renders:

```jsx
const deferredQuery = useDeferredValue(query);
const results = useMemo(() => filterLargeList(deferredQuery), [deferredQuery]);
const isStale = query !== deferredQuery; // use this to dim stale results
```

| | `useTransition` | `useDeferredValue` |
|---|---|---|
| Works on | A state **update** (you own the setter) | A **value** (often a prop you don't control) |
| API | `[isPending, startTransition]` | `deferredValue` |
| Pending flag | `isPending` built in | Compare `value !== deferredValue` |
| Typical use | Tab switches, filters, navigation | Expensive child rendering from a fast-changing prop |

Senior-level point: neither hook debounces network requests, and neither makes a slow calculation faster. They only let React **prioritise and interrupt rendering**. For API calls, still debounce. For slow computations, memoise or move them to a Web Worker. A child that re-renders with the deferred value should be wrapped in `React.memo`, or it re-renders on every keystroke anyway.

### [A7.3 — Code Splitting](A7.3-code-splitting.md)

**Q: Why does code splitting matter?**
A: Without it, the bundler produces **one big bundle**, so users download, parse and execute the code for **every page and feature** before seeing anything. On mobile, a 500 kB+ bundle means slow **LCP and INP**. Splitting ships only what the first screen needs and loads the rest **on demand**, and unchanged chunks stay cached across deploys.

```jsx
// Everything statically imported means one bundle with the PDF lib, charts, the editor...
import ReportsPage from './pages/Reports';     // pulls in chart.js (~200 kB) even for users who never open Reports

// Split: Reports and its heavy dependencies become a separate chunk, loaded only when visited
const ReportsPage = lazy(() => import('./pages/Reports'));
```

Interview-ready answer: *"A single bundle makes every user pay up front for every feature, which hurts load time and responsiveness, especially on mobile. Code splitting breaks the app into chunks loaded on demand, so the initial download only covers the first screen and rarely used features are fetched when needed. Separate chunks also cache better between deployments."*

**Q: What is the highest-impact form of splitting?**
A: **Route-based.** Each page becomes its own chunk, downloaded when the route is visited and cached afterwards. Routes are natural boundaries (the user is waiting for navigation anyway), it covers the whole app with little code, and it typically cuts the initial bundle the most. After that, split **heavy, rarely used components**: modals, charts, rich-text editors, PDF viewers.

```jsx
const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

<Suspense fallback={<PageSkeleton />}>
  <Routes>
    <Route path="/" element={<Home />} />
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/settings" element={<Settings />} />
  </Routes>
</Suspense>

// Component-level: the editor loads only when the modal opens
const RichEditor = lazy(() => import('./RichEditor'));
{isEditing && <Suspense fallback={<Spinner />}><RichEditor /></Suspense>}
```

Interview-ready answer: *"Route-level splitting. Every page becomes its own chunk, so users only download the routes they visit, and navigation is a natural moment to show a brief loading state. It gives the biggest reduction in the initial bundle for minimal effort. Then I split heavy, on-demand components like editors, charts and modals."*

**Q: How do you analyse a Vite bundle?**
A: Add **`rollup-plugin-visualizer`** to produce an interactive **treemap** of every chunk and dependency by size (rendered, gzip and brotli). Look for large dependencies used on only one page (PDF, charts, rich-text editors, moment or lodash in full), which are good candidates for dynamic imports, plus duplicated packages and accidental full-library imports.

```ts
// vite.config.ts
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [react(), visualizer({ filename: 'dist/stats.html', gzipSize: true, brotliSize: true, open: true })],
  build: {
    rollupOptions: {
      output: { manualChunks: { react: ['react', 'react-dom'], charts: ['recharts'] } }, // vendor chunks cache well
    },
  },
});
// npm run build → opens dist/stats.html
```

Fixes: dynamic `import()` for page-specific libraries, lighter alternatives (`date-fns` instead of `moment`), per-function imports (`lodash-es`), and removing unused dependencies.

Interview-ready answer: *"I run rollup-plugin-visualizer on the production build to get a treemap of chunk and dependency sizes. I look for big libraries pulled into the main chunk but used on one page, duplicates, and whole-library imports, then fix them with dynamic imports, lighter alternatives or tree-shakeable imports, and put a bundle-size check in CI so it doesn't regress."*

**Q: What are lazy loading and code splitting, and how do you implement them in React?**
A: **Code splitting** breaks the bundle into smaller chunks. **Lazy loading** downloads a chunk only when it is needed. In React, `React.lazy` takes a dynamic `import()`, and `Suspense` shows a fallback while the chunk loads.

```jsx
const Dashboard = React.lazy(() => import('./pages/Dashboard'));
const Settings = React.lazy(() => import('./pages/Settings'));

<Suspense fallback={<Spinner />}>
  <Routes>
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/settings" element={<Settings />} />
  </Routes>
</Suspense>

// Preload on hover, so the chunk is usually ready by the time the user clicks
const loadSettings = () => import('./pages/Settings');
<Link to="/settings" onMouseEnter={loadSettings}>Settings</Link>
```

The result is a smaller initial bundle and a faster first load. Good candidates are routes, modals, and heavy components such as charts and editors. Wrap lazy components in an error boundary to handle chunk-load failures. The trade-off is a short loading state the first time a chunk is requested. You can soften it by preloading likely next routes, for example calling the `import()` on link hover.

Interview-ready answer: *"Code splitting divides the bundle into chunks; lazy loading fetches a chunk only when it's needed. In React I use React.lazy with a dynamic import and a Suspense fallback, mostly per route and for heavy components. I wrap them in an error boundary for chunk-load failures after a deploy, and preload likely next routes on hover to hide the loading state."*

### [A7.4 — Virtualization](A7.4-virtualization.md)

**Q: What problem does virtualisation solve?**
A: 10,000 items means 10,000 DOM nodes costing memory and layout time. Virtualisation ("windowing") renders only visible items plus a small overscan buffer, usually 20–30 nodes. A tall spacer element keeps the scrollbar the right size, the visible rows are positioned with offsets, and the same small set of DOM nodes is reused as you scroll. Performance stays constant however long the list is.

```jsx
import { useVirtualizer } from '@tanstack/react-virtual';

function BigList({ items }) {
  const parentRef = useRef(null);
  const v = useVirtualizer({ count: items.length, getScrollElement: () => parentRef.current, estimateSize: () => 40, overscan: 8 });

  return (
    <div ref={parentRef} style={{ height: 600, overflow: 'auto' }}>
      <div style={{ height: v.getTotalSize(), position: 'relative' }}>          {/* spacer = full height */}
        {v.getVirtualItems().map((row) => (
          <div key={items[row.index].id} style={{ position: 'absolute', top: 0, left: 0, width: '100%', height: row.size, transform: `translateY(${row.start}px)` }}>
            {items[row.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

Trade-offs: browser find (Ctrl+F) can't see off-screen rows, and variable heights need measuring.

Interview-ready answer: *"Rendering thousands of rows creates thousands of DOM nodes, which makes the initial render slow, uses lots of memory, and makes scrolling janky. Virtualisation renders only the rows in the viewport plus a small buffer, positioned inside a full-height spacer, and recycles them as you scroll. The DOM stays around 30 nodes whether the list has a thousand rows or a million."*

**Q: TanStack Virtual vs react-window?**
A: **`@tanstack/react-virtual`** is **headless**: a hook (`useVirtualizer`) that gives you sizes and offsets while you render the markup. That makes it flexible for tables, grids, sticky headers, **dynamic heights** (`measureElement`) and horizontal lists, and it's actively maintained. **`react-window`** is older and gives you **ready-made components** (`FixedSizeList`, `VariableSizeList`, `FixedSizeGrid`). It's quicker for simple lists but less flexible.

```jsx
// react-window: a component does the rendering
import { FixedSizeList } from 'react-window';
<FixedSizeList height={600} itemCount={items.length} itemSize={40} width="100%">
  {({ index, style }) => <div style={style}>{items[index].name}</div>}
</FixedSizeList>

// TanStack Virtual: a hook, your markup (shown in the previous answer); dynamic heights:
<div ref={v.measureElement} data-index={row.index}>…</div>
```

Interview-ready answer: *"react-window gives ready-made list and grid components, which are great for simple fixed-size lists. TanStack Virtual is a headless hook that just computes what to render and where, so I control the markup. That's better for tables, sticky headers, variable or dynamic row heights and custom layouts, and it's actively maintained. For new work I usually choose TanStack Virtual."*

**Q: How do you measure the improvement?**
A: Compare **before and after** with numbers:
- The **DevTools Performance tab**: record scrolling and compare FPS, long tasks and scripting/rendering time.
- **DOM node count**: `document.querySelectorAll('li').length`, or Performance Monitor → DOM Nodes.
- **Memory**: a heap snapshot.
- **Initial render time**: the React Profiler, or `performance.now()` around the mount.

```js
// Console: DOM nodes before (10,000) vs after (~30)
document.querySelectorAll('[data-row]').length;

// Time the first render
const t0 = performance.now();
root.render(<BigList items={rows} />);
requestAnimationFrame(() => console.log('render ms', (performance.now() - t0).toFixed(0)));
// Typical: ~1800 ms → ~25 ms for 10k rows; scrolling 15 fps → 60 fps
```

Interview-ready answer: *"I measure before and after: DOM node count, the initial render time in the React Profiler, and scrolling FPS and long tasks in the Chrome Performance panel, plus memory from a heap snapshot. For a 10,000-row list you typically go from thousands of nodes and second-long renders to a few dozen nodes, millisecond renders and smooth 60 fps scrolling."*

**Q (Scenario): An admin table shows 50,000 customer records and scrolling is very slow. How do you optimise it?**
A: Use **list virtualisation (windowing)**: render only the rows visible in the viewport plus a small overscan buffer, instead of 50,000 DOM nodes. Libraries: TanStack Virtual (headless), react-window, React Virtuoso (handles variable heights out of the box).

```jsx
import { useVirtualizer } from '@tanstack/react-virtual';

function CustomerList({ customers }) {
  const parentRef = useRef(null);

  const virtualizer = useVirtualizer({
    count: customers.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 45,   // row height in px
    overscan: 5,              // extra rows above and below the viewport
  });

  return (
    <div ref={parentRef} style={{ height: 500, overflow: 'auto' }}>
      {/* a spacer with the full height keeps the scrollbar correct */}
      <div style={{ height: virtualizer.getTotalSize(), position: 'relative' }}>
        {virtualizer.getVirtualItems().map((item) => (
          <div
            key={customers[item.index].id}
            style={{
              position: 'absolute', top: 0, left: 0, width: '100%',
              height: item.size,
              transform: `translateY(${item.start}px)`,
            }}
          >
            {customers[item.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

Senior-level insight: virtualisation fixes **DOM and render cost**, but not the cost of **downloading and holding 50,000 records** in memory. Ask what the business actually needs (does anyone scroll 50k rows?) and combine virtualisation with:
- **Server-side pagination** or cursor-based **infinite scrolling** (`useInfiniteQuery` plus a virtualiser).
- **Server-side filtering, sorting and search** (indexed queries) instead of filtering 50k rows in the browser.
- Caching with TanStack Query.
- Memoised row components and stable handlers, so scrolling doesn't re-render every visible row.

### [P7.1 — Performance Audit](P7.1-performance-audit.md)

**Q: What re-render problems does the profiler reveal in the slow app?**
A:
- All 100 `ProductItem`s re-render on **each keystroke** in search.
- `CartSidebar` re-renders on search, because of a **shared context**.
- `StatsBar` re-renders on **cart changes** it doesn't display.
- `ExpensiveChart` (about 40 ms per render) re-renders **on every keystroke**, which makes typing laggy.

```
Profiler, typing "lamp" (4 commits):
  App ▸ ProductList ▸ ProductItem ×100    why: parent rendered, prop `onAdd` changed
       ▸ CartSidebar                     why: context AppContext changed
       ▸ ExpensiveChart  41.2 ms         why: prop `data` changed (new array every render)
```

Interview-ready answer (project walkthrough): *"I profiled typing in the search box. The flame graph showed every product row, the cart sidebar and an expensive chart re-rendering on each keystroke, and 'why did this render' pointed to a single shared context, unstable function and array props, and a 40 ms chart render in the hot path."*

**Q: What causes them?**
A:
1. **One context holding everything** (search, cart and stats), so any change re-renders every consumer.
2. **Inline functions and arrays** (`onAdd={() => …}`, `data={rows.map(...)}`) create new references every render, defeating memoisation.
3. **Filtering and derived values computed on every render**, including renders caused by unrelated cart changes.

```jsx
// The original
<AppContext.Provider value={{ search, setSearch, cart, addToCart, stats }}>   {/* new object every render */}
  {filtered = products.filter((p) => p.name.includes(search))}                 {/* recomputed every render */}
  <ProductItem onAdd={() => addToCart(p.id)} />                                {/* new function per item */}
  <ExpensiveChart data={sales.map(toPoint)} />                                 {/* new array every render */}
</AppContext.Provider>
```

Interview-ready answer: *"Three root causes: a god context mixing search and cart state, so every consumer re-rendered on any change; unstable inline callbacks and arrays as props, which defeat any memo; and expensive derivations computed during every render instead of only when their inputs change."*

**Q: What fixes are applied?**
A:
- **Split the context** into `CartContext` and `SearchContext`.
- **`useMemo`** for the filtered list, so it only recomputes on search or sort.
- **`useCallback`** for stable handlers.
- **`React.memo`** on `ProductItem`.
- **Memoised derived cart values.**
- **`memo` + a memoised `data` prop** for `ExpensiveChart`.

```jsx
// Split contexts, each with a memoised value
<SearchContext.Provider value={searchValue}><CartContext.Provider value={cartValue}>…</CartContext.Provider></SearchContext.Provider>

// Memoised derivations
const filtered = useMemo(() => products.filter((p) => p.name.toLowerCase().includes(q)).sort(bySort), [products, q, bySort]);
const cartTotal = useMemo(() => cart.reduce((s, i) => s + i.price * i.qty, 0), [cart]);
const chartData = useMemo(() => sales.map(toPoint), [sales]);

// Stable handlers + memoised components
const handleAdd = useCallback((id) => dispatch({ type: 'ADD', id }), []);
const ProductItem = React.memo(ProductItemImpl);
const ExpensiveChart = React.memo(ChartImpl);
```

Interview-ready answer: *"I split the single context by concern, memoised the filtered list and derived cart totals, stabilised handlers with useCallback, and wrapped ProductItem and the chart in React.memo with a memoised data prop. The result: a keystroke now re-renders only the search input and the matching list, not the cart, stats or chart."*

**Q: How are results documented?**
A: Before and after metrics in a **`PERFORMANCE_NOTES.md`**: what was measured, how, the numbers, and which fix caused which change, so the work is reviewable and repeatable. Measure the same interaction, on the same build type, under the same CPU throttling.

```md
## Search typing ("lamp", 4×CPU throttle, production build)
| Metric                         | Before | After |
|--------------------------------|-------:|------:|
| Components rendered/keystroke  |   104  |    6  |
| Commit duration (avg)          | 48 ms  | 4 ms  |
| INP (p75, web-vitals)          | 310 ms | 90 ms |
| ExpensiveChart renders         |    4   |    0  |
Fixes: split contexts (#12), useMemo filtered list (#13), React.memo ProductItem + stable onAdd (#14)
```

Interview-ready answer: *"I document before and after numbers for the same interaction under the same conditions: render counts, commit durations from the Profiler, and INP or LCP. Each improvement is mapped to the change that caused it, in a PERFORMANCE_NOTES file. It proves the work had an effect and gives reviewers and future developers the context."*

### [P7.2 — Virtual List](P7.2-virtual-list.md)

**Q: What does it build?**
A: A **100,000-row virtualised table** with TanStack Virtual: sortable headers, **debounced per-column filters**, global search, a live row count, **sticky headers** and scroll-to-top. Only about 30 rows exist in the DOM at any time.

```jsx
function VirtualTable({ rows }) {
  const [sort, setSort] = useState({ key: 'id', dir: 'asc' });
  const [filters, setFilters] = useState({});
  const debouncedFilters = useDebounce(filters, 250);

  const visible = useMemo(() => sortRows(filterRows(rows, debouncedFilters), sort), [rows, debouncedFilters, sort]);

  const parentRef = useRef(null);
  const v = useVirtualizer({ count: visible.length, getScrollElement: () => parentRef.current, estimateSize: () => 36, overscan: 10 });

  return (
    <>
      <p aria-live="polite">{visible.length.toLocaleString()} rows</p>
      <div ref={parentRef} className="h-[600px] overflow-auto">
        <div className="sticky top-0 z-10 bg-white"><HeaderRow sort={sort} onSort={setSort} onFilter={setFilters} /></div>
        <div style={{ height: v.getTotalSize(), position: 'relative' }}>
          {v.getVirtualItems().map((r) => <Row key={visible[r.index].id} row={visible[r.index]} start={r.start} />)}
        </div>
      </div>
      <button onClick={() => v.scrollToIndex(0)}>↑ Top</button>
    </>
  );
}
```

Interview-ready answer (project walkthrough): *"A 100k-row data table that stays smooth through virtualisation with TanStack Virtual: sticky headers, sorting, debounced per-column filters and global search, a row count and scroll-to-top. Filtering and sorting are memoised and debounced so they don't run on every keystroke, and only the visible rows are ever rendered."*

**Q: Why generate rows at module level?**
A: So the 100k mock rows are created **once, when the module loads**, not on **every component render**. Generating them inside the component body would rebuild 100,000 objects on every render (and give `useMemo`'s dependencies a new array each time).

```js
// data.js: runs once at import
export const ROWS = Array.from({ length: 100_000 }, (_, i) => ({
  id: i + 1,
  name: `User ${i + 1}`,
  email: `user${i + 1}@example.com`,
  age: 18 + (i % 60),
}));

// ❌ Inside the component: recreated on every render
function Table() { const rows = Array.from({ length: 100_000 }, makeRow); … }
// (If it must be inside: useMemo(() => generate(), []) or useState(() => generate()))
```

Interview-ready answer: *"Generating 100,000 objects is expensive, and a component body runs on every render. Creating the data once at module scope means it's built a single time and is a stable reference, so memoised filters and sorts don't invalidate needlessly. Inside a component, the equivalent is lazy useState or useMemo with no dependencies."*

**Q: How is performance measured?**
A: **Bundle analysis** with `rollup-plugin-visualizer` (`dist/stats.html`), plus **LCP** measured with a `PerformanceObserver`, alongside scroll FPS in the Performance panel and DOM node counts.

```js
new PerformanceObserver((list) => {
  const entries = list.getEntries();
  const lcp = entries[entries.length - 1];
  console.log('LCP', Math.round(lcp.startTime), 'ms', lcp.element);
}).observe({ type: 'largest-contentful-paint', buffered: true });

// Simpler, in production: import { onLCP, onINP } from 'web-vitals'
```

Interview-ready answer: *"Two angles: load and runtime. For load, the bundle treemap from rollup-plugin-visualizer, and LCP captured with a PerformanceObserver or the web-vitals library. For runtime, scroll FPS and long tasks in the Performance panel, plus the DOM node count, which stays around thirty regardless of the 100k rows."*

---

## Phase 8 — Testing

### [A8.1 — Jest + React Testing Library](A8.1-jest-react-testing-library.md)

**Q: What is RTL's testing philosophy?**
A: *"The more your tests resemble the way your software is used, the more confidence they give you."* Test what the **user sees and does** (visible text, roles, labels, typing and clicking), **not implementation details** (state values, internal methods, component instances, CSS classes). Tests then survive refactoring as long as behaviour stays the same, and they also nudge you towards accessible markup.

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('adds a todo', async () => {
  const user = userEvent.setup();
  render(<TodoApp />);

  await user.type(screen.getByRole('textbox', { name: /new todo/i }), 'Buy milk');
  await user.click(screen.getByRole('button', { name: /add/i }));

  expect(screen.getByRole('listitem')).toHaveTextContent('Buy milk');   // what the user sees
  // ❌ Not: expect(component.state.todos).toHaveLength(1)
});
```

Interview-ready answer: *"RTL tests components the way users use them: find elements by role, label or text, interact with user-event, and assert on what's rendered, never on internal state or implementation. That gives confidence the feature actually works, and tests don't break when I refactor internals. It also pushes the markup towards being accessible."*

**Q: What is the query priority?**
A: Prefer queries that reflect how **users and assistive technology** find elements:
1. `getByRole` (with a `name`).
2. `getByLabelText`.
3. `getByPlaceholderText`.
4. `getByText`.
5. `getByDisplayValue`.
6. `getByAltText`.
7. `getByTitle`.
8. `getByTestId`, as a last resort when nothing user-visible identifies the element.

```jsx
screen.getByRole('button', { name: /save/i });          // ✅ best: role + accessible name
screen.getByRole('heading', { level: 2, name: /orders/i });
screen.getByLabelText(/email/i);                        // ✅ form fields
screen.getByText(/no results/i);                        // ✅ non-interactive text
screen.getByAltText(/profile photo/i);
screen.getByTestId('chart-canvas');                     // ⚠️ last resort

screen.logTestingPlaygroundURL();                       // suggests the best query for the current DOM
```

Interview-ready answer: *"getByRole first, with the accessible name, because that's how screen-reader users find things and it doubles as an accessibility check. Then getByLabelText for form fields, getByText for content, and test IDs only as a last resort when nothing user-visible can identify the element."*

**Q: `getBy` vs `queryBy` vs `findBy`?**
A:

| | Not found | Found | Async? | Use for |
|---|---|---|---|---|
| `getBy…` | **Throws** | Returns the element | No | Elements that should be there **now** |
| `queryBy…` | Returns **`null`** | Returns the element | No | Asserting something is **absent** |
| `findBy…` | **Rejects** (after a ~1 s timeout) | Resolves with the element | **Yes** (await it) | Elements that **appear later** (after a fetch, a timer, a state update) |

The `getAllBy`, `queryAllBy` and `findAllBy` variants return arrays.

```jsx
expect(screen.getByRole('button', { name: /submit/i })).toBeEnabled();     // present now
expect(screen.queryByRole('alert')).not.toBeInTheDocument();               // absent
expect(await screen.findByText(/welcome, pulkit/i)).toBeInTheDocument();   // appears after an async load

await waitForElementToBeRemoved(() => screen.queryByText(/loading/i));     // waits for removal
```

Interview-ready answer: *"getBy throws if the element is missing, for things that must be there now. queryBy returns null, which is the right tool for asserting that something is not rendered. findBy returns a promise and retries until the element appears or it times out, for anything that shows up asynchronously, like data after a fetch."*

**Q: Why prefer `userEvent` over `fireEvent`?**
A: `fireEvent` dispatches **one synthetic DOM event**. `userEvent` simulates a **real user interaction**, with the full sequence of events a browser fires. `user.type` fires focus, keydown, keypress, input, keyup and change for each character. `user.click` fires pointer and mouse events and focus changes, and respects `disabled` and `pointer-events`. Bugs that depend on that sequence show up in tests too. It's async: call `userEvent.setup()` and `await` each action.

```jsx
// fireEvent: just sets the value + one change event, with no focus, no keystrokes, and maxLength ignored
fireEvent.change(input, { target: { value: 'hello' } });

// userEvent: realistic
const user = userEvent.setup();
await user.type(input, 'hello');            // per-key events; respects maxLength and disabled
await user.keyboard('{Enter}');
await user.tab();                           // tests focus order
await user.selectOptions(select, 'India');
await user.upload(fileInput, new File(['x'], 'a.png', { type: 'image/png' }));
```

Interview-ready answer: *"fireEvent fires a single event, while user-event simulates what a person actually does: focus, each keystroke, pointer events, and rules like disabled elements and maxLength. Tests are more realistic and catch bugs fireEvent misses. I use userEvent.setup() and await each interaction, and keep fireEvent for rare events user-event doesn't model, like scroll."*

**Q (Scenario): You lead a team on a production React app. How do you make sure new features don't break existing ones?**
A: Use a layered testing strategy (the testing pyramid or "testing trophy") and run it in CI on every pull request.

1. **Unit tests** cover pure functions and isolated logic: utils, reducers, Zod schemas, custom hooks with `renderHook`.
   ```js
   const calculateTotal = (price, qty) => price * qty;
   test('calculates total correctly', () => {
     expect(calculateTotal(100, 2)).toBe(200);
   });
   ```
2. **Component tests** use React Testing Library and test what the user sees and does, not implementation details.
   ```jsx
   import { render, screen } from '@testing-library/react';
   import userEvent from '@testing-library/user-event';

   test('increments counter', async () => {
     const user = userEvent.setup();
     render(<Counter />);
     await user.click(screen.getByRole('button', { name: /increment/i }));
     expect(screen.getByText('Count: 1')).toBeInTheDocument();
   });
   ```
3. **Integration tests** render a whole feature (a page with providers, router and React Query) with the API mocked at the network level by **MSW**. In this testing approach these give the most confidence per test.
4. **End-to-end tests** use Playwright or Cypress on a real browser and build, covering critical user journeys: login, create product, add to cart, checkout.

| Test type | Tools |
|---|---|
| Unit | Vitest / Jest |
| Component | React Testing Library + user-event |
| API mocking | MSW |
| E2E | Playwright / Cypress |
| Coverage | Vitest / Jest (v8 or istanbul) |
| Static checks | TypeScript strict, ESLint (including `jsx-a11y`) |
| Visual and accessibility (optional) | Storybook + Chromatic, axe |

Senior-level insight:
- Focus on **business-critical flows and user behaviour**, not on reaching 100% coverage. A coverage threshold (for example 80%) is a guardrail, not the goal.
- Make tests part of the definition of done.
- Block merges on failing CI.
- Add a **regression test for every production bug**.
- Keep E2E tests few and stable, because they are slow and can be flaky.

### [A8.2 — Testing Hooks & Context](A8.2-testing-hooks-and-context.md)

**Q: How do you test a custom hook?**
A: With **`renderHook`** from RTL, which renders the hook inside a minimal test component. You read its latest return value from `result.current`, trigger updates inside `act`, pass new props with `rerender`, and use `waitFor` for async hooks. Hooks that need providers get a `wrapper`.

```jsx
import { renderHook, act, waitFor } from '@testing-library/react';

test('useCounter increments and resets', () => {
  const { result } = renderHook(() => useCounter(5));
  expect(result.current.count).toBe(5);

  act(() => result.current.increment());
  expect(result.current.count).toBe(6);

  act(() => result.current.reset());
  expect(result.current.count).toBe(5);
});

test('useDebounce updates after the delay', () => {
  vi.useFakeTimers();
  const { result, rerender } = renderHook(({ v }) => useDebounce(v, 300), { initialProps: { v: 'a' } });
  rerender({ v: 'ab' });
  expect(result.current).toBe('a');
  act(() => vi.advanceTimersByTime(300));
  expect(result.current).toBe('ab');
  vi.useRealTimers();
});
```

Interview-ready answer: *"I use renderHook, which hosts the hook in a tiny component. result.current gives the latest return value, state changes go inside act, rerender passes new arguments, waitFor handles async results, and fake timers handle debounce or interval logic. If the hook needs context, a router or React Query, I pass a wrapper with those providers."*

**Q: What is `act()` for?**
A: It makes sure all **state updates, effects and re-renders** triggered by an action are **processed and flushed before you assert**, matching how React behaves in the browser. RTL's `render`, `userEvent`, `findBy` and `waitFor` already wrap things in `act`. You call it **yourself** mainly when invoking hook functions directly via `renderHook`, or when advancing fake timers. The "not wrapped in act(...)" warning usually means an **async update happened after the test finished**: await it with `findBy`/`waitFor`.

```jsx
const { result } = renderHook(() => useToggle());
act(() => result.current.toggle());                // a direct state update → wrap it
expect(result.current.on).toBe(true);

await act(async () => { await result.current.save(); });   // async version

act(() => vi.advanceTimersByTime(1000));          // timers that set state
```

Interview-ready answer: *"act flushes all pending state updates and effects for a unit of work, so assertions see the UI as the user would. RTL's render, userEvent, findBy and waitFor already use it, so I only call act directly when triggering hook functions from renderHook or advancing fake timers. An act warning almost always means I forgot to await something asynchronous."*

**Q: How do you test context consumers?**
A: Two options. **Render with the real Provider** and control it through its props or initial state; this is more realistic and tests the integration. Or **provide a test value** directly through `Context.Provider`, which is simpler when you only care about part of the context or want edge-case values. A reusable `renderWithProviders` helper keeps tests tidy.

```jsx
// 1. The real provider
test('shows the logged-in user', () => {
  render(<AuthProvider initialUser={{ name: 'Pulkit' }}><Navbar /></AuthProvider>);
  expect(screen.getByText(/pulkit/i)).toBeInTheDocument();
});

// 2. A controlled test value through the raw context
test('logout calls the handler', async () => {
  const logout = vi.fn();
  render(<AuthContext.Provider value={{ user: { name: 'A' }, logout }}><Navbar /></AuthContext.Provider>);
  await userEvent.click(screen.getByRole('button', { name: /logout/i }));
  expect(logout).toHaveBeenCalledOnce();
});
```

Interview-ready answer: *"I prefer rendering the consumer inside the real provider with controlled initial values, because it tests what ships. When I need a specific edge case or want to assert a callback, I render the raw Context.Provider with a hand-made value and mock functions. A shared renderWithProviders helper removes the boilerplate."*

**Q: What is the wrapper pattern?**
A: The `wrapper` option of `render`/`renderHook` takes a component that wraps the thing under test with **providers**: context, router, a React Query client, a theme, i18n. Bundle it into a **custom render** helper that every test uses. For React Query, create a **fresh `QueryClient` per test**, with `retry: false`, so the cache doesn't leak between tests.

```jsx
// test-utils.jsx
export function createWrapper({ route = '/' } = {}) {
  const queryClient = new QueryClient({ defaultOptions: { queries: { retry: false } } });
  return function Wrapper({ children }) {
    return (
      <QueryClientProvider client={queryClient}>
        <MemoryRouter initialEntries={[route]}>
          <AuthProvider>{children}</AuthProvider>
        </MemoryRouter>
      </QueryClientProvider>
    );
  };
}
export const renderWithProviders = (ui, opts) => render(ui, { wrapper: createWrapper(opts), ...opts });

// Usage
const { result } = renderHook(() => useUser(1), { wrapper: createWrapper() });
await waitFor(() => expect(result.current.isSuccess).toBe(true));
```

Interview-ready answer: *"wrapper lets me render a component or hook inside whatever providers it needs, such as a router, React Query, auth or theme. I centralise that in a renderWithProviders helper and create a new QueryClient per test with retries off, so tests are isolated and fail fast."*

### [A8.3 — Mock Service Worker](A8.3-mock-service-worker.md)

**Q: Why use MSW instead of mocking `fetch`?**
A: Mocking `global.fetch` (or `axios`) mocks the **transport**, not the **API contract**. It's brittle (tied to how the code calls the network), and it **skips your real data layer**. MSW intercepts requests **at the network level**, so components, React Query hooks, Axios interceptors and error handling run **exactly as in production**. You can switch data-fetching libraries without touching a test, and the same handlers work in tests, Storybook and the browser during development.

```js
// ❌ Brittle: coupled to fetch, bypasses headers and interceptors, breaks if you switch to axios
global.fetch = vi.fn().mockResolvedValue({ ok: true, json: async () => [{ id: 1 }] });

// ✅ MSW: real requests, a mocked network
import { http, HttpResponse } from 'msw';
export const handlers = [
  http.get('/api/posts', () => HttpResponse.json([{ id: 1, title: 'Hello' }])),
];
```

Interview-ready answer: *"Mocking fetch replaces part of the app itself, so the tests stop exercising the real request code and break whenever the fetching implementation changes. MSW intercepts at the network layer: the app makes real requests, React Query and Axios behave normally, and I just define the API's responses. The tests are more realistic, more resilient, and the handlers are reusable in the browser too."*

**Q: What are request handlers?**
A: Functions that **match a method and URL** and **return a mocked response**, for example `http.get('/api/posts', () => HttpResponse.json(data))`. They can read params, the query and the body, return errors and delays, and be **overridden per test** with `server.use()` for edge cases. Organise them by domain (`handlers/users.ts`, `handlers/posts.ts`).

```js
import { http, HttpResponse, delay } from 'msw';

export const postHandlers = [
  http.get('/api/posts', ({ request }) => {
    const q = new URL(request.url).searchParams.get('q') ?? '';
    return HttpResponse.json(posts.filter((p) => p.title.includes(q)));
  }),
  http.get('/api/posts/:id', ({ params }) => {
    const post = posts.find((p) => p.id === Number(params.id));
    return post ? HttpResponse.json(post) : new HttpResponse(null, { status: 404 });
  }),
  http.post('/api/posts', async ({ request }) => {
    const body = await request.json();
    await delay(100);
    return HttpResponse.json({ id: 99, ...body }, { status: 201 });
  }),
];

// Per-test override: the error path
server.use(http.get('/api/posts', () => HttpResponse.json({ message: 'boom' }, { status: 500 })));
```

Interview-ready answer: *"Handlers map a method and URL to a mocked response, and can read route params, query strings and request bodies, add delays or return errors. I keep happy-path handlers organised by domain as the default, and override them in individual tests with server.use to cover errors, empty states and slow responses."*

**Q: `setupServer` vs `setupWorker`?**
A: **`setupServer`** (`msw/node`) intercepts requests in **Node**: Jest or Vitest tests, using request interception. **`setupWorker`** (`msw/browser`) registers a real **Service Worker** in the **browser**, to mock APIs during development, demos or Storybook when the backend isn't ready. Both use the **same handlers**.

```js
// mocks/server.js (tests)
import { setupServer } from 'msw/node';
export const server = setupServer(...handlers);

// vitest.setup.js
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));   // fail on un-mocked calls
afterEach(() => server.resetHandlers());                           // undo per-test overrides
afterAll(() => server.close());

// mocks/browser.js (development)
import { setupWorker } from 'msw/browser';
export const worker = setupWorker(...handlers);

// main.jsx
if (import.meta.env.DEV && import.meta.env.VITE_MOCK_API) {
  const { worker } = await import('./mocks/browser');
  await worker.start();
}
```

Interview-ready answer: *"setupServer is for Node, so unit and integration tests; setupWorker runs a real service worker in the browser to mock the API during development or demos. They share the same handlers. In tests I listen with onUnhandledRequest set to error, reset handlers after each test, and close the server at the end."*

### [A8.4 — Playwright E2E Testing](A8.4-playwright-e2e-testing.md)

**Q: How does Playwright differ from RTL?**
A: **RTL** renders components **in isolation** in a simulated DOM (jsdom). It's fast and great for component and integration behaviour. **Playwright** drives a **real browser** (Chromium, Firefox, WebKit, headed or headless) against the **running app**, testing full user journeys across pages, with real routing, the network, layout, cookies and storage. It's slower, so you keep **fewer** E2E tests for the critical flows.

```ts
// RTL: one component, a fake DOM
render(<LoginForm onSubmit={fn} />);

// Playwright: the whole app, a real browser
test('user can log in and see the dashboard', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('pulkit@example.com');
  await page.getByLabel('Password').fill('secret123');
  await page.getByRole('button', { name: 'Sign in' }).click();
  await expect(page).toHaveURL(/\/dashboard/);
  await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
});
```

Interview-ready answer: *"RTL tests components in a simulated DOM. It's fast, which makes it ideal for most behaviour tests. Playwright runs the real app in real browsers end to end, covering navigation, network, auth, layout and cross-browser issues that jsdom can't. I use many RTL tests and a small set of Playwright tests for critical journeys like signup, login and checkout."*

**Q: What are its key APIs?**
A: `page.goto(url)`. **Locators**, preferably `getByRole`, `getByLabel`, `getByText` or `getByTestId`, which **auto-wait** and retry. Actions: `.fill()`, `.click()`, `.check()`, `.selectOption()`. **Web-first assertions** that retry until they pass: `expect(locator).toBeVisible()`, `toHaveText`, `toHaveURL`. `storageState` for authentication, **fixtures** for setup, and `page.route()` to mock the network.

```ts
import { test, expect } from '@playwright/test';

test('search filters products', async ({ page }) => {
  await page.route('**/api/products*', (route) => route.fulfill({ json: [{ id: 1, name: 'Lamp' }] })); // optional mock
  await page.goto('/products');
  await page.getByRole('searchbox', { name: 'Search' }).fill('lamp');
  await expect(page.getByRole('listitem')).toHaveCount(1);       // retries until true; no manual waits
  await expect(page.getByRole('listitem').first()).toHaveText(/lamp/i);
});
// Debugging: npx playwright test --ui, trace viewer on failure, codegen to record locators
```

Interview-ready answer: *"page.goto to navigate, role-based locators that auto-wait, actions like fill and click, and web-first expect assertions that retry until they pass, so there are no sleeps and less flakiness. On top of that: storageState for auth, fixtures for setup, page.route for mocking, and the trace viewer and UI mode for debugging."*

**Q: How do you avoid logging in for every test?**
A: A **setup project** logs in **once** through the UI (or the API) and saves the auth state (cookies + localStorage) to a file with `storageState`. Authenticated test projects then **start already logged in** by loading that file. This is faster and less flaky, and a separate small test still covers the login flow itself.

```ts
// auth.setup.ts
import { test as setup, expect } from '@playwright/test';
setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill(process.env.E2E_USER!);
  await page.getByLabel('Password').fill(process.env.E2E_PASS!);
  await page.getByRole('button', { name: 'Sign in' }).click();
  await expect(page).toHaveURL(/dashboard/);
  await page.context().storageState({ path: 'playwright/.auth/user.json' });
});

// playwright.config.ts
projects: [
  { name: 'setup', testMatch: /.*\.setup\.ts/ },
  { name: 'chromium', use: { ...devices['Desktop Chrome'], storageState: 'playwright/.auth/user.json' }, dependencies: ['setup'] },
],
```

Interview-ready answer: *"I log in once in a setup project, save the browser's storage state (cookies and localStorage) to a file, and make the authenticated projects depend on it and load that state. Every test starts already signed in, which saves time and removes a common source of flakiness, and one dedicated test still covers the login UI."*

**Q: What is the Page Object Model for?**
A: Encapsulating a page's **locators and actions** in a class (`LoginPage.login()`, `CartPage.checkout()`), so tests read like user stories and **a UI change is fixed in one place** instead of in every test.

```ts
// pages/LoginPage.ts
export class LoginPage {
  constructor(private page: Page) {}
  readonly email = () => this.page.getByLabel('Email');
  readonly password = () => this.page.getByLabel('Password');
  async goto() { await this.page.goto('/login'); }
  async login(email: string, password: string) {
    await this.email().fill(email);
    await this.password().fill(password);
    await this.page.getByRole('button', { name: 'Sign in' }).click();
  }
}

// The test reads like a story
test('invalid password shows an error', async ({ page }) => {
  const login = new LoginPage(page);
  await login.goto();
  await login.login('pulkit@example.com', 'wrong');
  await expect(page.getByRole('alert')).toHaveText(/invalid credentials/i);
});
```

Interview-ready answer: *"The Page Object Model wraps each page's locators and actions in a class, so tests call meaningful methods like login or addToCart instead of raw selectors. When the UI changes, I update one page object instead of dozens of tests, and the tests stay readable. In Playwright I often expose page objects as fixtures."*

**Q: How is it run in CI?**
A: The Playwright config's **`webServer`** starts the app (ideally a **production build plus preview**) before the tests. The suite runs in **GitHub Actions** with the browsers installed, **retries** on CI, and uploads the **HTML report and traces** as artifacts when tests fail.

```ts
// playwright.config.ts
export default defineConfig({
  retries: process.env.CI ? 2 : 0,
  reporter: [['html', { open: 'never' }]],
  use: { baseURL: 'http://localhost:4173', trace: 'on-first-retry' },
  webServer: { command: 'npm run build && npm run preview', url: 'http://localhost:4173', reuseExistingServer: !process.env.CI },
});
```

```yaml
# .github/workflows/e2e.yml
- uses: actions/setup-node@v4
  with: { node-version: 22 }
- run: npm ci
- run: npx playwright install --with-deps
- run: npx playwright test
- uses: actions/upload-artifact@v4
  if: failure()
  with: { name: playwright-report, path: playwright-report/ }
```

Interview-ready answer: *"Playwright's webServer option builds and starts the app before tests run. In GitHub Actions I install dependencies and browsers, run the suite against the production preview with a couple of retries, record traces on retry, and upload the HTML report when something fails, so failures can be debugged from the trace without reproducing them locally."*

### [P8.1 — Tested React App](P8.1-tested-react-app.md)

**Q: What is the coverage target and app under test?**
A: **80%+ coverage** on the P1.2 Movie Search app (or a similar one), enforced in CI through **coverage thresholds**, so the build fails if coverage drops. Coverage is a guardrail; the aim is to cover **user-visible behaviour and edge cases**, not lines for their own sake.

```ts
// vitest.config.ts
export default defineConfig({
  test: {
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov'],
      thresholds: { lines: 80, functions: 80, branches: 75, statements: 80 },
      exclude: ['src/mocks/**', 'src/main.tsx', '**/*.d.ts'],
    },
  },
});
```

Interview-ready answer (project walkthrough): *"I took the Movie Search app to 80%+ coverage, enforced with Vitest thresholds in CI. The goal wasn't the number itself but meaningful coverage of behaviour: debounced search, loading, error and empty states, details, and persisted favourites."*

**Q: Which kinds of tests does it include?**
A: The full testing pyramid (or trophy):
- **Unit** tests for utilities and hooks (`useDebounce`, `useFavorites`).
- **RTL component** tests.
- **MSW handlers** for the OMDB API.
- **Integration** tests for the search flow, with real hooks, React Query and MSW.
- **Playwright E2E** for the critical journey.

```jsx
// Integration: real components + hooks, with the network mocked by MSW
test('searching shows results, then the empty state', async () => {
  const user = userEvent.setup();
  renderWithProviders(<App />);

  await user.type(screen.getByRole('searchbox'), 'inception');
  expect(await screen.findByText('Inception')).toBeInTheDocument();

  server.use(http.get('*/omdbapi.com/*', () => HttpResponse.json({ Response: 'False', Error: 'Movie not found!' })));
  await user.clear(screen.getByRole('searchbox'));
  await user.type(screen.getByRole('searchbox'), 'zzzz');
  expect(await screen.findByText(/no movies found/i)).toBeInTheDocument();
});
```

Interview-ready answer: *"Unit tests for pure utilities and custom hooks, component tests with RTL, integration tests that render whole features with real hooks and React Query against MSW-mocked APIs, which give the most confidence per test, and a Playwright E2E test for the main user journey."*

**Q: Which E2E flow is tested?**
A: The critical journey: **search → view detail → add to favourites → verify persistence** (reload the page and check the favourite is still there). It proves routing, the API, state and localStorage work together in a real browser.

```ts
test('favourite persists across reloads', async ({ page }) => {
  await page.goto('/');
  await page.getByRole('searchbox').fill('Inception');
  await page.getByRole('link', { name: /inception/i }).first().click();
  await expect(page.getByRole('heading', { name: /inception/i })).toBeVisible();

  await page.getByRole('button', { name: /add to favourites/i }).click();
  await page.reload();                                               // persistence check
  await page.goto('/favorites');
  await expect(page.getByRole('listitem').filter({ hasText: /inception/i })).toBeVisible();
});
```

Interview-ready answer: *"The core journey end to end: search for a movie, open its details, add it to favourites, reload, and confirm it's still in the favourites list. That one test proves search, routing, the detail fetch, state updates and localStorage persistence work together in a real browser."*

**Q: How are MSW mocks split?**
A: Shared **`handlers`** live in one place. **`mocks/server.ts`** wires them into the **Node/test** environment (`setupServer`), and **`mocks/browser.ts`** wires them into the **browser** for development (`setupWorker`), so tests and local dev use the **same fake API**.

```
src/mocks/
├── handlers/
│   └── omdb.ts     // http.get('*/omdbapi.com/*', ...) search + detail
├── handlers.ts     // export const handlers = [...omdbHandlers]
├── server.ts       // setupServer(...handlers)   ← vitest.setup.ts
└── browser.ts      // setupWorker(...handlers)   ← main.tsx in dev when VITE_MOCK_API=true
```

Interview-ready answer: *"One set of handlers, two entry points: server.ts uses setupServer for Vitest, and browser.ts uses setupWorker so I can run the app locally against the same mocked OMDB API without a key or network. Tests and development can't drift apart, because they share the definitions."*

### [P8.2 — Accessible & Tested Component Library](P8.2-accessible-and-tested-component-library.md)

**Q: Which components and ARIA patterns are built?**
A: Following the **WAI-ARIA Authoring Practices**:

| Component | Pattern |
|---|---|
| **Tabs** | `role="tablist"/"tab"/"tabpanel"`, `aria-selected`, `aria-controls`, Arrow, Home and End keys, roving `tabIndex` |
| **Combobox** | `role="combobox"`, `aria-expanded`, `aria-autocomplete`, `aria-activedescendant`, Arrow, Enter and Escape |
| **Disclosure** | A button with `aria-expanded` and `aria-controls` |
| **Switch** | `role="switch"`, `aria-checked`, toggled with Space |
| **AlertDialog** | `role="alertdialog"`, a focus trap, initial focus on the least destructive action, Escape |

```jsx
function Tabs({ tabs }) {
  const [active, setActive] = useState(0);
  const refs = useRef([]);
  const onKeyDown = (e) => {
    const last = tabs.length - 1;
    const next = { ArrowRight: active === last ? 0 : active + 1, ArrowLeft: active === 0 ? last : active - 1, Home: 0, End: last }[e.key];
    if (next === undefined) return;
    e.preventDefault(); setActive(next); refs.current[next].focus();
  };
  return (
    <>
      <div role="tablist" onKeyDown={onKeyDown}>
        {tabs.map((t, i) => (
          <button key={t.id} ref={(el) => (refs.current[i] = el)} role="tab" id={`tab-${t.id}`}
            aria-selected={i === active} aria-controls={`panel-${t.id}`} tabIndex={i === active ? 0 : -1}
            onClick={() => setActive(i)}>{t.label}</button>
        ))}
      </div>
      {tabs.map((t, i) => (
        <div key={t.id} role="tabpanel" id={`panel-${t.id}`} aria-labelledby={`tab-${t.id}`} hidden={i !== active}>{t.content}</div>
      ))}
    </>
  );
}
```

Interview-ready answer (project walkthrough): *"I built Tabs, Combobox, Disclosure, Switch and AlertDialog from scratch following the WAI-ARIA Authoring Practices: the right roles and states, keyboard support like arrow keys with roving tabindex, aria-activedescendant for the combobox, and focus trapping for the alert dialog. Each one has tests for its ARIA state, keyboard and focus behaviour."*

**Q: What must every component pass?**
A: Three gates:
1. **Automated axe audits** (`jest-axe`/`vitest-axe`) with no violations.
2. **Full keyboard navigation**: every interaction works without a mouse, with a visible focus.
3. **Interaction tests** that verify the **ARIA state changes and focus behaviour** (focus moves, focus is restored, `aria-expanded` flips).

Automated tools catch only part of the problems, so behaviour tests and manual screen-reader checks fill the gap.

```jsx
import { axe } from 'vitest-axe';

test('Switch is accessible and keyboard-operable', async () => {
  const user = userEvent.setup();
  const { container } = render(<Switch label="Notifications" />);
  expect(await axe(container)).toHaveNoViolations();                      // 1. axe

  const sw = screen.getByRole('switch', { name: /notifications/i });
  await user.tab();
  expect(sw).toHaveFocus();                                               // 2. reachable by keyboard
  await user.keyboard(' ');
  expect(sw).toHaveAttribute('aria-checked', 'true');                     // 3. ARIA state updates
});
```

Interview-ready answer: *"Every component has to pass an axe audit in its tests, be fully operable by keyboard, and have interaction tests asserting the ARIA states and focus management, such as aria-expanded toggling and focus returning to the trigger when a dialog closes. Axe only catches part of the issues, so the behaviour tests and manual screen-reader passes matter."*

**Q: Why does the Combobox close on blur after a `setTimeout`?**
A: Clicking an option makes the input **blur before the option's click fires**. If the list closed immediately on blur, the option would unmount and the click would be **lost**. The **150 ms delay** lets the click register first. Cleaner alternatives are to call `preventDefault()` on the option's `onMouseDown`, which keeps focus in the input, or to check `relatedTarget` on blur.

```jsx
// The delay approach
const onBlur = () => { closeTimer.current = setTimeout(() => setOpen(false), 150); };
useEffect(() => () => clearTimeout(closeTimer.current), []);

// ✅ Often cleaner: don't let the option steal focus at all
<li role="option" aria-selected={i === active} onMouseDown={(e) => e.preventDefault()} onClick={() => select(opt)}>
  {opt.label}
</li>
// Or: onBlur={(e) => { if (!listRef.current?.contains(e.relatedTarget)) setOpen(false); }}
```

Interview-ready answer: *"Event order: mousedown on an option blurs the input before the click event fires. Closing immediately on blur would unmount the option and swallow the click, so a short delay lets the selection land first. The more robust fix is to preventDefault on the option's mousedown, or check relatedTarget, which avoids timing assumptions altogether."*

---

## Phase 9 — TypeScript, A11y & Production

### [A9.1 — TypeScript with React](A9.1-typescript-with-react.md)

**Q: Why use TypeScript with React?**
A: To catch **wrong prop types, missing props, missing null checks and bad handler signatures at compile time** instead of in the browser or in production. You also get autocomplete and inline docs for every prop, **safe refactoring** (renaming a prop flags every call site), and types that document intent. Turn on **`strict: true`** for the strongest guarantees, since much of the benefit comes from `strictNullChecks`.

```tsx
type UserCardProps = { user: { name: string; email?: string }; onSelect: (id: string) => void };

function UserCard({ user, onSelect }: UserCardProps) {
  return (
    <div onClick={() => onSelect(user.name)}>
      {user.name}
      {user.email.toLowerCase()}      {/* ❌ TS: 'user.email' is possibly 'undefined' */}
    </div>
  );
}

<UserCard user={{ nme: 'Pulkit' }} onSelect={(id: number) => {}} />
// ❌ TS: 'nme' doesn't exist (did you mean 'name'?); ❌ TS: number is not assignable to string
```

Interview-ready answer: *"TypeScript turns a whole class of runtime bugs into compile errors: wrong or missing props, unhandled null values, mismatched callback signatures. It gives great autocomplete, makes refactoring safe across a large codebase, and the types act as living documentation of each component's API. I always enable strict mode."*

**Q: `React.FC<Props>` or a plain function?**
A: Prefer a **plain function with typed props**. Historically `React.FC` added an implicit `children` prop (before React 18), so components accepted children they ignored. It also made generic components awkward and didn't allow `defaultProps`-style patterns. Since React 18 it's mostly harmless, but plain functions are simpler, work with generics, and are the community convention. Declare `children` explicitly when you accept it.

```tsx
// ✅ Preferred
type ButtonProps = { label: string; onClick?: () => void; children?: React.ReactNode };
function Button({ label, onClick, children }: ButtonProps) {
  return <button onClick={onClick}>{label}{children}</button>;
}

// Works, but less flexible: generics are awkward with React.FC
const Button2: React.FC<ButtonProps> = ({ label }) => <button>{label}</button>;

// Generic components need plain functions
function List<T>({ items, render }: { items: T[]; render: (item: T) => React.ReactNode }) {
  return <ul>{items.map(render)}</ul>;
}
```

Interview-ready answer: *"I type props on a plain function. React.FC used to add an implicit children prop and doesn't play well with generic components, so plain functions are simpler and more explicit. I declare children as ReactNode only when the component actually renders it."*

**Q: How do you type `useState` and `useRef`?**
A: **`useState`** infers the type from the initial value. Add an explicit generic when the initial value doesn't describe all possible values, such as `null` or an empty array (`useState<User | null>(null)`, `useState<string[]>([])`). **`useRef`** for the DOM takes the **element type** and `null` (`useRef<HTMLInputElement>(null)`); TypeScript knows `.current` may be `null` before mount. For a **mutable value ref**, type it with the value (`useRef<number | null>(null)`).

```tsx
const [count, setCount] = useState(0);                     // inferred: number
const [user, setUser] = useState<User | null>(null);       // explicit: starts null
const [tags, setTags] = useState<string[]>([]);            // otherwise never[]
const [status, setStatus] = useState<'idle' | 'loading' | 'error'>('idle');  // a union

const inputRef = useRef<HTMLInputElement>(null);           // DOM ref
useEffect(() => { inputRef.current?.focus(); }, []);       // ?. because it may be null

const timerRef = useRef<ReturnType<typeof setTimeout> | null>(null);  // mutable value
timerRef.current = setTimeout(tick, 1000);
```

Interview-ready answer: *"useState infers from its initial value, so I add a generic when that value is null or an empty array, like `User | null` or `string[]`, or to restrict values to a union. DOM refs get the element type with a null initial value and are read with optional chaining. Value refs, like timer IDs, are typed with the stored value's type."*

**Q: Which utility types are useful?**
A:

| Utility | Use in React |
|---|---|
| `Pick<T, K>` | A card only needs some fields: `Pick<User, 'name' \| 'avatar'>` |
| `Omit<T, K>` | Drop internal or server fields: `Omit<User, 'passwordHash'>`, create forms `Omit<Todo, 'id'>` |
| `Partial<T>` | Update payloads and patch forms |
| `Required<T>` | Make every field mandatory (resolved config) |
| `Record<K, V>` | Typed dictionaries: `Record<Status, string>` for colour maps |
| `ReturnType<typeof fn>` | Reuse a function's or hook's return type |
| `React.ComponentProps<'button'>` | Extend native element props |

```tsx
type User = { id: string; name: string; email: string; avatar: string; passwordHash: string };

type UserCardProps = Pick<User, 'name' | 'avatar'>;
type PublicUser = Omit<User, 'passwordHash'>;
type UpdateUserInput = Partial<Omit<User, 'id' | 'passwordHash'>>;
const statusColor: Record<'todo' | 'doing' | 'done', string> = { todo: 'gray', doing: 'blue', done: 'green' };
type UseCartResult = ReturnType<typeof useCart>;

type ButtonProps = React.ComponentProps<'button'> & { variant?: 'primary' | 'danger' };  // all native props + ours
```

Interview-ready answer: *"Pick and Omit to derive narrower prop types from domain models instead of duplicating them, Partial for update payloads, Record for typed lookup maps, ReturnType to reuse a hook's return type, and React.ComponentProps to extend native element props. They keep types derived from one source of truth."*

**Q: Why use discriminated unions for variants?**
A: Instead of several **booleans that can conflict** (`isError` and `isSuccess` both true?) or optional props that only make sense together, a **union with a shared literal field** (`kind`/`status`) lets TypeScript **narrow** to the right shape in each branch. It also **enforces the correct props per variant**: a dismissible alert *must* have `onDismiss`, a link button *must* have `href`. Impossible states can't be expressed.

```tsx
type AlertProps =
  | { variant: 'info'; message: string }
  | { variant: 'error'; message: string; onRetry: () => void }
  | { variant: 'dismissible'; message: string; onDismiss: () => void };

function Alert(props: AlertProps) {
  switch (props.variant) {
    case 'error':       return <p role="alert">{props.message} <button onClick={props.onRetry}>Retry</button></p>;
    case 'dismissible': return <p>{props.message} <button onClick={props.onDismiss}>×</button></p>;
    default:            return <p>{props.message}</p>;
  }
}

<Alert variant="dismissible" message="Saved" />   // ❌ TS: 'onDismiss' is missing

type FetchState<T> = { status: 'loading' } | { status: 'error'; error: Error } | { status: 'success'; data: T };
```

Interview-ready answer: *"Discriminated unions model variants as distinct shapes sharing a literal tag. TypeScript narrows on the tag, so each branch only sees valid props, and required props per variant are enforced at the call site. That makes impossible states unrepresentable, unlike a pile of optional props and booleans that can contradict each other."*

**Q: How do you type API responses?**
A: TypeScript types **don't exist at runtime**, so `as User` only *claims* the shape. **Derive types from Zod schemas** and **validate responses at the boundary**. One schema is the single source of truth for both the runtime check and the static type, and a backend change that breaks the contract fails loudly at the API layer, not deep inside a component.

```ts
const UserSchema = z.object({ id: z.string(), name: z.string(), email: z.string().email(), createdAt: z.coerce.date() });
type User = z.infer<typeof UserSchema>;

async function getUser(id: string): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return UserSchema.parse(await res.json());    // throws ZodError if the shape is wrong; also converts createdAt to a Date
}

// ❌ Unsafe: trusts whatever the server sends
const user = (await res.json()) as User;
```

(Alternatively, generate types from an OpenAPI spec, or use tRPC for end-to-end types.)

Interview-ready answer: *"A type assertion on res.json() is a lie the compiler can't check. I define Zod schemas for API responses, infer the TypeScript types from them, and parse every response at the API-client boundary. The types are guaranteed to match reality, and contract changes surface immediately as clear validation errors."*

### [A9.2 — Accessibility (a11y)](A9.2-accessibility-a11y.md)

**Q: What standard applies?**
A: **WCAG 2.1 AA** (2.2 is the current version), the level most laws and procurement rules reference (the ADA, the EU's EN 301 549). It's organised around **POUR**: Perceivable, Operable, Understandable, Robust. Accessibility is a **baseline requirement**, not a feature: it matters legally, it's good UX for everyone, and it's far cheaper to build in than to retrofit.

```jsx
// A few concrete AA requirements in code
<img src={chart} alt="Revenue grew 24% from Q1 to Q2" />                 // 1.1.1 text alternatives
<button onClick={save}>Save</button>                                      // 2.1.1 keyboard accessible (native button)
<label htmlFor="email">Email</label><input id="email" />                 // 3.3.2 labels
// 1.4.3 contrast ≥ 4.5:1 for body text; 2.4.7 visible focus indicator; 4.1.2 name, role, value
```

Interview-ready answer: *"The target is WCAG 2.1 (or 2.2) level AA, which is what accessibility laws generally reference. It's built on being perceivable, operable, understandable and robust. In practice that means semantic HTML, full keyboard support with visible focus, text alternatives, sufficient contrast, labelled form controls and correct ARIA. I treat it as part of the definition of done."*

**Q: What are common a11y problems in React?**
A:
1. **`<div onClick>`/`<span>` instead of semantic elements**: not focusable, no role, no keyboard support.
2. **Icon-only buttons without an accessible name** (`aria-label`).
3. **Modals without a focus trap or focus return.**
4. **Dynamic content that screen readers miss**: toasts, async errors, loading states with no live region.
5. **Inputs without labels**: placeholder-only.
6. **SPA route changes** that don't move focus or update `document.title`.
7. **Missing `alt`**, and colour-only status.

```jsx
// ❌                                             // ✅
<div onClick={open}>Open</div>                    <button type="button" onClick={open}>Open</button>
<button onClick={close}><XIcon /></button>        <button onClick={close} aria-label="Close dialog"><XIcon aria-hidden /></button>
<input placeholder="Email" />                     <label>Email <input type="email" /></label>
{error && <p>{error}</p>}                         <p role="alert">{error}</p>     {/* announced */}
<span className="red">Failed</span>               <span className="red">❌ Failed</span>  {/* not colour alone */}
```

Interview-ready answer: *"The usual suspects: clickable divs instead of buttons and links, icon buttons with no accessible name, modals that don't trap and restore focus, async updates like errors and toasts that screen readers never hear because there's no live region, placeholder-only inputs, and route changes that don't manage focus or the page title. Semantic HTML fixes most of them for free."*

**Q: What do the main ARIA attributes do?**
A:

| Attribute | Purpose | Example |
|---|---|---|
| `aria-label` | Names an element with no visible text | An icon button: `aria-label="Search"` |
| `aria-labelledby` | Names an element using another element's text | A dialog labelled by its heading |
| `aria-describedby` | Links extra description or help, or an error | An input → its error message |
| `aria-expanded` | Whether a toggled region is open | Accordion or menu buttons |
| `aria-controls` | Which element this control affects | A tab → its panel |
| `aria-invalid` | Field validity | A field with errors |
| `aria-live` | Announces dynamic changes (`polite`/`assertive`) | Toasts, search results count |
| `aria-hidden` | Hides decorative content from assistive tech | Decorative icons |
| `role` | Overrides or sets semantics | `role="dialog"`, `role="switch"` |

```jsx
<label htmlFor="email">Email</label>
<input id="email" aria-invalid={!!error} aria-describedby={error ? 'email-err' : 'email-hint'} />
<p id="email-hint">We'll never share it.</p>
{error && <p id="email-err" role="alert">{error}</p>}

<button aria-expanded={open} aria-controls="faq-1" onClick={() => setOpen(!open)}>Shipping</button>
<div id="faq-1" hidden={!open}>…</div>
```

The first rule of ARIA: **don't use ARIA if a native element does the job.** A `<button>` beats `<div role="button">`.

Interview-ready answer: *"aria-label and aria-labelledby give elements an accessible name, aria-describedby links hints and errors to fields, aria-expanded and aria-controls describe disclosure widgets, aria-invalid flags errors, aria-live announces dynamic updates, and role sets semantics when no native element fits. But native HTML comes first; ARIA is for filling gaps, and wrong ARIA is worse than none."*

**Q: What are the `tabIndex` guidelines?**
A:
- **`0`** puts an element in the **natural tab order**. Only needed for custom interactive elements; native buttons, links and inputs already have it.
- **`-1`** makes an element focusable **programmatically** (`ref.focus()`) but **not via Tab**. Use it for headings you move focus to after a route change, modal containers and roving tabindex.
- **`1` or more** should be **avoided**, because it overrides the DOM order and creates a confusing, fragile focus sequence.

```jsx
// Route change: move focus to the main heading
const headingRef = useRef(null);
useEffect(() => { headingRef.current?.focus(); }, [location.pathname]);
<h1 tabIndex={-1} ref={headingRef}>{title}</h1>

// Roving tabindex: only the active tab is tabbable; the arrow keys move between tabs
<button role="tab" tabIndex={isActive ? 0 : -1}>…</button>

// ❌ Never
<input tabIndex={3} />
```

Interview-ready answer: *"tabIndex 0 adds a custom interactive element to the natural tab order. -1 makes something focusable from script but skipped by Tab, which suits focus management after route changes, dialogs or roving tabindex in widgets. Positive values break the logical order, so I never use them and fix the DOM order instead."*

**Q: Do buttons need `onKeyDown`?**
A: **No.** A native `<button>` is already focusable and fires `onClick` for **Enter and Space** (and for clicks, taps and assistive tech). You only need `tabIndex={0}`, a `role` and an `onKeyDown` handler if you build a **non-button** interactive element, which is the reason to just use `<button>`.

```jsx
// ✅ Keyboard support comes free
<button type="button" onClick={toggle}>Toggle</button>

// ❌ All this extra code just to imitate a button, and it's still incomplete
<div role="button" tabIndex={0} onClick={toggle}
  onKeyDown={(e) => { if (e.key === 'Enter' || e.key === ' ') { e.preventDefault(); toggle(); } }}>
  Toggle
</div>
```

Interview-ready answer: *"No. Native buttons already handle Enter and Space and are focusable, so onClick covers mouse, keyboard and assistive technology. Custom key handlers are only needed for non-native widgets or extra keys, like arrow keys in a menu. Needing one for a simple action usually means a div should have been a button."*

**Q: How should modal focus be managed?**
A: Three steps:
1. On open, **move focus into the modal**: the first focusable element, the least-destructive action, or the dialog itself.
2. **Trap focus** inside, so Tab and Shift+Tab cycle within the dialog while the background is inert.
3. On close, **return focus to the trigger** element.

Also close on Escape, label the dialog (`aria-labelledby`), and use `aria-modal="true"`. The native **`<dialog>` with `showModal()`** or **Radix Dialog** give you most of this for free.

```jsx
function Modal({ open, onClose, title, children }) {
  const dialogRef = useRef(null);

  useEffect(() => {
    const dialog = dialogRef.current;
    if (open && !dialog.open) dialog.showModal();     // native: focus moves in, background becomes inert, Escape works
    if (!open && dialog.open) dialog.close();
  }, [open]);

  return (
    <dialog ref={dialogRef} aria-labelledby="modal-title" onClose={onClose}>
      <h2 id="modal-title">{title}</h2>
      {children}
      <button onClick={onClose}>Close</button>
    </dialog>
  );
}
// The browser restores focus to the previously focused element when a modal <dialog> closes.
```

Interview-ready answer: *"When it opens, focus moves inside; while it's open, focus is trapped and the background is inert; Escape closes it; and on close, focus returns to the element that opened it. The dialog is labelled by its title. I use the native dialog element with showModal, or Radix Dialog, rather than hand-rolling a focus trap."*

**Q: Which tools are used?**
A:
- **`axe-core`**, via `jest-axe`/`vitest-axe` in tests or the axe DevTools extension, for automated checks.
- **`eslint-plugin-jsx-a11y`** to catch issues while writing code.
- **Lighthouse** accessibility audits.
- **Manual screen-reader testing**: NVDA on Windows, VoiceOver on macOS and iOS, TalkBack on Android.
- The **DevTools accessibility tree** to inspect names and roles.
- **Keyboard-only** walkthroughs.

Automated tools find only part of the issues.

```js
// eslint.config.js
import jsxA11y from 'eslint-plugin-jsx-a11y';
export default [jsxA11y.flatConfigs.recommended];

// In tests
import { axe } from 'vitest-axe';
expect(await axe(container)).toHaveNoViolations();

// Playwright: whole-page audit
import AxeBuilder from '@axe-core/playwright';
const results = await new AxeBuilder({ page }).withTags(['wcag2a', 'wcag2aa']).analyze();
expect(results.violations).toEqual([]);
```

Interview-ready answer: *"Layered tooling: the jsx-a11y ESLint plugin while coding, axe in unit and Playwright tests so violations fail CI, and Lighthouse audits. Automation catches only part of the problems, so I also do keyboard-only passes and test key flows with NVDA and VoiceOver, inspecting the accessibility tree in DevTools when something sounds wrong."*

### [A9.3 — Internationalization (i18n)](A9.3-internationalization-i18n.md)

**Q: What does react-i18next provide?**
A:
- **`useTranslation(namespace)`**, which returns `t` (and `i18n` for changing the language).
- The **`Trans`** component for strings containing JSX.
- **Pluralisation and interpolation** per language rules.
- **Namespaces**, lazy-loaded.
- Language detection and fallbacks.
- **`Intl`** integration for dates, numbers and currency.
- Suspense support while translations load.

```jsx
// i18n.js
i18n.use(HttpBackend).use(LanguageDetector).use(initReactI18next).init({
  fallbackLng: 'en',
  supportedLngs: ['en', 'fr', 'hi'],
  ns: ['common', 'dashboard'],
  defaultNS: 'common',
  interpolation: { escapeValue: false },          // React already escapes
});

function Greeting({ name, unread }) {
  const { t, i18n } = useTranslation('dashboard');
  return (
    <>
      <h1>{t('welcome', { name })}</h1>                    {/* "Welcome, {{name}}!" */}
      <p>{t('unread', { count: unread })}</p>              {/* pluralised */}
      <p>{new Intl.NumberFormat(i18n.language, { style: 'currency', currency: 'INR' }).format(1999)}</p>
    </>
  );
}
```

Interview-ready answer: *"react-i18next gives me the useTranslation hook with a t function for interpolated, pluralised strings, the Trans component for translations containing markup, namespaces that load lazily per feature, language detection with fallbacks, and Intl formatting for dates, numbers and currency."*

**Q: Where do translation files live?**
A: In **`public/locales/{lang}/{namespace}.json`**, **lazy-loaded at runtime** with `i18next-http-backend`. Only the active language and the namespaces in use are downloaded, rather than every translation being bundled into the JavaScript. Splitting by namespace (`common`, `dashboard`, `settings`) keeps each file small and loads it with the feature.

```
public/locales/
├── en/
│   ├── common.json      { "nav": { "home": "Home" }, "actions": { "save": "Save" } }
│   └── dashboard.json   { "welcome": "Welcome, {{name}}!" }
└── fr/
    ├── common.json      { "nav": { "home": "Accueil" }, "actions": { "save": "Enregistrer" } }
    └── dashboard.json   { "welcome": "Bienvenue, {{name}} !" }
```

```js
i18n.use(HttpBackend).init({ backend: { loadPath: '/locales/{{lng}}/{{ns}}.json' } });
```

Interview-ready answer: *"Translations are JSON files per language and namespace under public/locales, fetched on demand by the HTTP backend. Users only download the language they use and the namespaces for the features they open, which keeps the JS bundle free of translation data and lets translators work on plain JSON."*

**Q: How does pluralisation work?**
A: Pass **`count`** to `t`. i18next selects the key suffix using the language's **CLDR plural rules** via `Intl.PluralRules`. English has `_one` and `_other` (plus an optional `_zero`). Other languages need more forms: Arabic has six, Polish has `_few` and `_many`. With count=1 you get "1 item", with count=5 you get "5 items". Never build plurals with `count === 1 ? … : …` in code, because that breaks for other languages.

```json
// en/common.json
{ "cart_one": "{{count}} item in your cart", "cart_other": "{{count}} items in your cart", "cart_zero": "Your cart is empty" }
```

```jsx
t('cart', { count: 0 });   // "Your cart is empty"
t('cart', { count: 1 });   // "1 item in your cart"
t('cart', { count: 5 });   // "5 items in your cart"
// ❌ count === 1 ? 'item' : 'items'   → wrong in Polish, Arabic, Russian…
```

Interview-ready answer: *"I pass count to t, and i18next picks the right plural key suffix using each language's CLDR plural rules, so English uses one and other while languages like Polish or Arabic get their extra forms. Translators supply each form in JSON. Hand-written ternaries for plurals don't work beyond English, so I never use them."*

**Q: When do you use `Trans`?**
A: When a translated sentence **contains React elements** (links, bold text, components) that must stay **inside** the sentence, because word order changes between languages. Splitting the sentence into separate `t()` calls around a `<Link>` would hard-code English word order.

```json
{ "terms": "By signing up you agree to our <termsLink>Terms</termsLink> and <bold>Privacy Policy</bold>." }
```

```jsx
<Trans
  i18nKey="terms"
  components={{ termsLink: <Link to="/terms" />, bold: <strong /> }}
/>
// French can reorder freely: "En vous inscrivant, vous acceptez nos <termsLink>Conditions</termsLink>…"

// ❌ Concatenation breaks word order in other languages
{t('agree')} <Link to="/terms">{t('terms')}</Link> {t('and')} …
```

Interview-ready answer: *"Trans is for translations that include markup or components, like a link or bold text mid-sentence. The translator controls the whole sentence, including where the tags go, and I map named tags to React elements. Concatenating separate translated fragments around JSX breaks in languages with different word order."*

**Q: What should the language switcher also update?**
A: **`<html lang>`** (and **`dir`** for right-to-left languages like Arabic or Hebrew), so **screen readers pronounce text with the right voice and rules**, browsers offer correct translation and spell-checking, and CSS `:lang()` and RTL layouts work. Also persist the choice (localStorage, a cookie or the user profile) and update date and number formatting.

```jsx
function LanguageSwitcher() {
  const { i18n } = useTranslation();

  useEffect(() => {
    document.documentElement.lang = i18n.language;
    document.documentElement.dir = i18n.dir(i18n.language);    // 'rtl' for ar/he
  }, [i18n.language]);

  return (
    <select value={i18n.language} onChange={(e) => i18n.changeLanguage(e.target.value)} aria-label="Language">
      <option value="en">English</option>
      <option value="fr">Français</option>
      <option value="ar">العربية</option>
    </select>
  );
}
```

Interview-ready answer: *"Besides changing the i18next language, it must set the html lang attribute, so screen readers switch pronunciation and browsers handle translation and spell-check correctly, and set dir for right-to-left languages. I persist the choice, and use logical CSS properties so RTL layouts flip correctly."*

**Q: Can translation keys be type-checked?**
A: **Yes.** Declare your resources' types for i18next (module augmentation in `i18next.d.ts`). Then `t('nav.home')` autocompletes, a **typo or missing key is a TypeScript error**, and interpolation variables can be checked too. This prevents raw keys like `"nav.hoem"` from showing up in the UI.

```ts
// i18next.d.ts
import 'i18next';
import type common from '../public/locales/en/common.json';
import type dashboard from '../public/locales/en/dashboard.json';

declare module 'i18next' {
  interface CustomTypeOptions {
    defaultNS: 'common';
    resources: { common: typeof common; dashboard: typeof dashboard };
  }
}
```

```tsx
const { t } = useTranslation();
t('nav.home');      // ✅ autocompleted
t('nav.hoem');      // ❌ TS error: not a valid key
```

Interview-ready answer: *"Yes. With i18next's CustomTypeOptions I declare the English JSON files as the resource types. The t function then autocompletes keys and turns typos or missing keys into compile errors, so untranslated keys never ship to users. CI can additionally check that the other languages have every key."*

### [A9.4 — Build & Deployment](A9.4-build-and-deployment.md)

**Q: What does a React SPA build into?**
A: **Static files**: one `index.html`, JS bundles, CSS and assets. No Node server is needed at runtime, so it can be hosted on any static host or CDN (S3 + CloudFront, Netlify, Vercel, nginx). During `vite build`, Vite (with Rollup) **tree-shakes** unused code, **minifies**, **code-splits** dynamic imports into chunks, and **fingerprints** filenames with content hashes for long-term caching.

```
dist/
├── index.html                       # cache: no-cache (always revalidate)
└── assets/
    ├── index-3f9a1c2b.js            # hashed: cache-control: public, max-age=31536000, immutable
    ├── Dashboard-8b21e4f0.js        # lazy route chunk
    ├── vendor-react-1d2c3e4f.js
    └── index-a7e5c9d1.css
```

Interview-ready answer: *"A production build of an SPA is just static files: index.html plus hashed, minified, tree-shaken and code-split JS and CSS chunks. That means it deploys to any CDN or static host. Hashed assets get a year-long immutable cache, while index.html is never cached, so users pick up new deploys immediately."*

**Q: How do environment variables work in Vite?**
A: Only **`VITE_`-prefixed** variables are exposed to client code via **`import.meta.env`**, and they're **inlined at build time**. Changing them means **rebuilding**, and they end up **visible in the shipped JavaScript**, so **never put secrets there** (API secret keys, DB passwords). Secrets belong on a backend or proxy. Centralise and **validate** them at startup in one module (for example `src/config/env.ts`) so a missing variable fails fast.

```bash
# .env.production
VITE_API_URL=https://api.example.com
VITE_SENTRY_DSN=https://abc@sentry.io/1     # public by design
STRIPE_SECRET_KEY=sk_live_...                # no VITE_ prefix, so not exposed (and not usable in client code anyway)
```

```ts
// src/config/env.ts
import { z } from 'zod';
const EnvSchema = z.object({
  VITE_API_URL: z.string().url(),
  VITE_SENTRY_DSN: z.string().url().optional(),
});
export const env = EnvSchema.parse(import.meta.env);   // throws at startup if misconfigured
// usage: fetch(`${env.VITE_API_URL}/users`)
```

Interview-ready answer: *"Vite only exposes variables prefixed with VITE_ through import.meta.env, and it bakes them into the bundle at build time. They're public, so they're for configuration like API base URLs, never secrets. I validate them with a Zod schema in one config module so a bad deploy fails immediately, and remember that changing them needs a rebuild."*

**Q: How is the Vite build tuned?**
A:
- **Manual vendor chunks**, so rarely changing libraries (React, routing, charts) get their own cache-friendly files.
- A **chunk-size warning** if any chunk exceeds 500 kB.
- **Source maps** uploaded to Sentry but not served publicly.
- A modern build target.
- The bundle visualizer, to find heavy dependencies.

```ts
// vite.config.ts
export default defineConfig({
  plugins: [react(), visualizer({ filename: 'dist/stats.html', gzipSize: true })],
  build: {
    target: 'es2020',
    sourcemap: 'hidden',                              // generated for Sentry, not referenced in the bundle
    chunkSizeWarningLimit: 500,                       // kB
    rollupOptions: {
      output: {
        manualChunks: {
          react: ['react', 'react-dom', 'react-router-dom'],
          query: ['@tanstack/react-query'],
          charts: ['recharts'],
        },
      },
    },
  },
});
```

Interview-ready answer: *"I split stable vendor libraries into their own chunks, so a code change doesn't invalidate React's cached bundle. I set a chunk-size warning, generate hidden source maps for error tracking, target modern browsers, and use the visualizer to find bloat. Combined with route-level lazy loading, that keeps the initial download small and the cache hit rate high."*

**Q: How is the app containerised?**
A: A **multi-stage Dockerfile**: build with Node, then serve the static output from a small **`nginx:alpine`** image. nginx uses **`try_files $uri $uri/ /index.html`** so client-side routes like `/dashboard` don't 404 on refresh, applies **long caching for hashed assets** and **no caching for `index.html`**, and enables gzip.

```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
ARG VITE_API_URL
ENV VITE_API_URL=$VITE_API_URL
RUN npm run build

FROM nginx:alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
```

```nginx
server {
  listen 80;
  root /usr/share/nginx/html;
  gzip on; gzip_types text/css application/javascript application/json image/svg+xml;

  location /assets/ { add_header Cache-Control "public, max-age=31536000, immutable"; }
  location = /index.html { add_header Cache-Control "no-cache"; }
  location / { try_files $uri $uri/ /index.html; }      # SPA fallback
}
```

Interview-ready answer: *"A multi-stage build: Node compiles the app, then only the dist folder is copied into a small nginx:alpine image. nginx serves index.html for unknown paths so client-side routing survives refreshes and deep links, caches hashed assets for a year, never caches index.html, and gzips responses. The final image is tiny and has no Node runtime."*

**Q: What does the CI/CD pipeline do?**
A: Every PR runs **quality gates**: lint, type-check and unit/integration tests with coverage. Then it **builds**, runs **E2E** tests against the build, and on **main only** pushes the **Docker image** and (optionally) **deploys** to Vercel, using **repository secrets** (`VITE_API_URL`, `VERCEL_TOKEN`, `CODECOV_TOKEN`) rather than committed values.

```yaml
# .github/workflows/ci.yml
on: [push, pull_request]
jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npm run lint && npm run typecheck
      - run: npm test -- --coverage
      - uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
  e2e:
    needs: quality
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npx playwright install --with-deps
      - run: npm run build && npx playwright test
        env:
          VITE_API_URL: ${{ secrets.VITE_API_URL }}
  deploy:
    if: github.ref == 'refs/heads/main'
    needs: e2e
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker build --build-arg VITE_API_URL=${{ secrets.VITE_API_URL }} -t ghcr.io/me/app:${{ github.sha }} .
      - run: docker push ghcr.io/me/app:${{ github.sha }}
```

Interview-ready answer: *"On every push and PR: install, lint, type-check, and run unit and integration tests with coverage reported to Codecov. Then build and run Playwright E2E tests against the production build. Only on main, after everything passes, it builds and pushes a Docker image tagged with the commit SHA and deploys. Secrets come from repository secrets, and failing gates block the merge."*

**Q: What is the difference between CSR, SSR and SSG?**
A: They are rendering strategies that differ in **where and when** the HTML is produced.

| | CSR (Client-Side Rendering) | SSR (Server-Side Rendering) | SSG (Static Site Generation) |
|---|---|---|---|
| HTML generated | In the browser, by JS | On the server, **per request** | At **build time** |
| First paint | Slower: blank page until JS loads | Fast: full HTML sent | Fastest: served from a CDN |
| SEO | Weaker | Good | Good |
| Data freshness | Live (fetched on the client) | Fresh on every request | Stale until rebuilt (or ISR) |
| Server cost | Static hosting only | Server runs on every request | Static hosting only |
| Use for | Admin dashboards, internal tools, apps behind login | Personalised or frequently changing public pages (product pages, feeds) | Blogs, docs, marketing pages |

With SSR and SSG, the HTML is then **hydrated**: React attaches event handlers in the browser to make the page interactive.

Senior-level point: Next.js lets you mix these strategies **per page or route** based on data freshness. Related options:
- **ISR (Incremental Static Regeneration):** static pages re-generated in the background every N seconds.
- **Streaming SSR with Suspense:** sends HTML in chunks as it becomes ready.
- **React Server Components:** server-only components that send zero JS to the client.

A Vite SPA like the one in this curriculum is pure CSR.

```jsx
// CSR (Vite SPA): the server sends an empty <div id="root">; data loads in the browser
function Products() {
  const { data } = useQuery({ queryKey: ['products'], queryFn: getProducts });
  return <ProductGrid items={data ?? []} />;
}

// SSR (Next.js App Router): rendered on the server for every request
export const dynamic = 'force-dynamic';
export default async function Page() {
  const products = await getProducts();          // runs on the server
  return <ProductGrid items={products} />;
}

// SSG / ISR (Next.js): rendered at build time, re-generated every hour
export const revalidate = 3600;
export default async function Blog() {
  const posts = await getPosts();
  return <PostList posts={posts} />;
}
```

Interview-ready answer: *"CSR renders in the browser after the JS loads: simple static hosting, great for dashboards behind login, but a slower first paint and weaker SEO. SSR renders HTML on the server per request, giving fast first content and fresh, SEO-friendly pages at the cost of server work. SSG renders at build time and serves from a CDN, the fastest option, for content that rarely changes. Frameworks like Next.js let me choose per route, plus ISR and streaming."*

**Q: What are React Server Components (RSC), and how are they different from Client Components?**
A: **Server Components** run only on the server, at build or request time. Their code is **never sent to the browser**; only their rendered output is. They can be `async` and can read databases, files or secrets directly. **Client Components** are the classic React components: their JS ships to the browser, they are pre-rendered to HTML and then hydrated, and they can use state, effects, event handlers and browser APIs. In the Next.js App Router, every component is a **Server Component by default**, and `'use client'` at the top of a file marks it (and everything it imports) as client code.

```jsx
// Server Component: async, fetches directly, ships no JS
async function Products() {
  const products = await db.product.findMany(); // or fetch(...) with caching options
  return (
    <div>
      {products.map((p) => <p key={p.id}>{p.name}</p>)}
      <AddToCartButton productId={products[0].id} /> {/* a client "island" */}
    </div>
  );
}
```

```jsx
'use client';
import { useState } from 'react';

export function AddToCartButton({ productId }) {
  const [added, setAdded] = useState(false);
  return <button onClick={() => setAdded(true)}>{added ? 'Added' : 'Add to cart'}</button>;
}
```

| | Server Components | Client Components |
|---|---|---|
| Runs on | Server only | Server (pre-render) and browser |
| JS shipped to the browser | None | Yes |
| Hooks (`useState`, `useEffect`) and event handlers | No | Yes |
| Direct DB, filesystem or secret access | Yes | No |
| `async` component | Yes | No |
| Best for | Data fetching, static or heavy content, large dependencies (markdown, syntax highlighting) | Interactivity: forms, modals, anything with state |

Composition rules:
- A Server Component can render Client Components and pass them **serialisable** props (no functions, except Server Actions).
- A Client Component **cannot import** a Server Component, but it can receive one as `children`.

Senior-level insight: keep most of the tree as Server Components and push `'use client'` **down to the smallest interactive leaves**. This keeps the client bundle small, removes request waterfalls (data is fetched next to the database), and keeps secrets on the server. RSC is not the same as SSR. SSR renders *Client Components* to HTML for the first load; RSC is a separate component type that never hydrates. They are usually used together.

**Q: What is hydration, and what causes hydration mismatches?**
A: **Hydration** is React attaching event handlers and state to HTML that was already rendered on the server, so the page becomes interactive:
1. The server renders HTML.
2. The browser displays it immediately.
3. The JS bundle downloads.
4. React (`hydrateRoot`) walks the existing DOM and attaches to it instead of re-creating it.

A **hydration mismatch** happens when the client's first render produces different output from the server HTML. React then warns and may re-render that part on the client, which costs performance and can cause visible flicker.

```jsx
// ❌ The server time differs from the client time
function Home() {
  return <h1>Current time: {new Date().toLocaleTimeString()}</h1>;
}

// ✅ Render a stable value first, then update after hydration
'use client';
function Home() {
  const [time, setTime] = useState(null);
  useEffect(() => setTime(new Date().toLocaleTimeString()), []);
  return <h1>Current time: {time ?? 'Loading...'}</h1>;
}
```

Common causes:
- Dates, times and timezone or locale formatting.
- Browser-only APIs during render (`window`, `localStorage`, `navigator`).
- `Math.random()` or generated IDs. Use `useId` instead.
- **Invalid HTML nesting**, for example `<div>` inside `<p>` or `<a>` inside `<a>`. The browser repairs the HTML, so it no longer matches.
- Different data on the server and the client.
- Browser extensions injecting attributes.

Senior-level insight: fix the root cause rather than silencing the warning. `suppressHydrationWarning` is acceptable only for a single unavoidable text node, such as a timestamp. For browser-only widgets, use a client-only render (an effect or a mounted flag, or `dynamic(..., { ssr: false })` in Next.js).

**Q: What are Error Boundaries in React?**
A: Components that catch JavaScript errors thrown **while rendering, in lifecycle methods, and in constructors** anywhere in their child tree. They log the error and show a fallback UI instead of unmounting the whole app; without one, an uncaught render error blanks the entire page. They must be **class components**, because there is no hook equivalent of `getDerivedStateFromError` and `componentDidCatch`.

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };               // render the fallback next
  }

  componentDidCatch(error, info) {
    logToService(error, info.componentStack); // e.g. Sentry
  }

  render() {
    if (this.state.hasError) return this.props.fallback ?? <h2>Something went wrong!</h2>;
    return this.props.children;
  }
}

<ErrorBoundary fallback={<DashboardError />}>
  <Dashboard />
</ErrorBoundary>
```

In practice, most teams use the `react-error-boundary` package (`<ErrorBoundary FallbackComponent={...} onReset={...}>`), which adds a reset/retry option.

Interview-ready answer: *"Error boundaries are class components that catch errors thrown while rendering their children, using getDerivedStateFromError to show a fallback UI and componentDidCatch to log it. Without one, a single render error unmounts the whole app. In practice I use the react-error-boundary package for a hook-friendly API with reset and retry, and report errors to Sentry."*

**Q: What do Error Boundaries *not* catch?**
A:
- Errors in **event handlers**. Use `try/catch` inside the handler.
- **Asynchronous code**: `setTimeout`, promise rejections, `fetch` failures. Catch these and store the error in state. React Query exposes them as `error`, and you can re-throw during render so the boundary catches them.
- Errors during **server-side rendering**.
- Errors thrown **in the boundary itself**. Only its children are covered.

```jsx
// Event handler: catch it yourself
async function handleSave() {
  try { await save(); } catch (err) { setError(err); toast.error('Save failed'); }
}

// Async error → push it into render so the boundary catches it
const { showBoundary } = useErrorBoundary();          // from react-error-boundary
useEffect(() => { loadData().catch(showBoundary); }, [showBoundary]);

// React Query: throwOnError sends query errors to the nearest boundary
useQuery({ queryKey: ['stats'], queryFn: getStats, throwOnError: true });
```

Interview-ready answer: *"They only catch errors thrown during rendering, lifecycle methods and constructors of their children. They don't catch errors in event handlers, in async code like timeouts and promises, in server rendering, or in the boundary itself. For handlers I use try/catch and error state. For async errors I either handle them locally or rethrow them into render, for example with showBoundary from react-error-boundary or React Query's throwOnError."*

**Q: Where should you place Error Boundaries?**
A: At several levels: one at the app root as a last-resort fallback, one per route or page, and around risky widgets (third-party components, charts, lazy-loaded chunks). That way one broken widget doesn't take down the whole page. Report caught errors to a monitoring tool such as Sentry.

```jsx
import { ErrorBoundary } from 'react-error-boundary';

<ErrorBoundary FallbackComponent={AppCrash} onError={(e, info) => Sentry.captureException(e, { extra: info })}>  {/* app root */}
  <Layout>
    <ErrorBoundary FallbackComponent={PageError} resetKeys={[location.pathname]}>                                  {/* per route */}
      <Routes>…</Routes>
    </ErrorBoundary>
    <ErrorBoundary fallback={<p>Chart unavailable</p>}>                                                           {/* risky widget */}
      <RevenueChart />
    </ErrorBoundary>
  </Layout>
</ErrorBoundary>
```

Interview-ready answer: *"In layers: a root boundary as the last resort, one per route that resets when the path changes, so navigating away clears the error, and granular ones around risky or independent widgets like third-party embeds, charts and lazy-loaded chunks. A failure then degrades one section instead of the whole page, and every boundary reports to Sentry."*

**Q (Scenario): A dashboard has Profile, Analytics, Notifications and Reports. Analytics throws a rendering error. How do you stop it taking down the whole dashboard?**
A: Wrap **each independent section** in its own Error Boundary, so a failure is contained to that section and shows a fallback while the rest stays usable.

```jsx
function Dashboard() {
  return (
    <>
      <UserProfile />
      <ErrorBoundary fallback={<p>Unable to load analytics.</p>}>
        <Analytics />
      </ErrorBoundary>
      <ErrorBoundary fallback={<p>Unable to load reports.</p>}>
        <Reports />
      </ErrorBoundary>
    </>
  );
}
```

With `react-error-boundary` you also get **retry**:

```jsx
<ErrorBoundary
  fallbackRender={({ resetErrorBoundary }) => (
    <div>Unable to load this section. <button onClick={resetErrorBoundary}>Retry</button></div>
  )}
  onError={(error, info) => Sentry.captureException(error, { extra: info })}
>
  <Analytics />
</ErrorBoundary>
```

Senior-level insight: use a layered strategy:
- **Error Boundaries** for rendering errors.
- **try/catch** in event handlers and async code.
- **API-level handling** for failed requests: React Query `error` states, retries, and an Axios interceptor for 401s.
- **Sentry** (with source maps) for production tracking.
- A **retry** option wherever recovery is possible.

**Q (Scenario): The app works on localhost, but in production some pages are blank, API calls fail, there are hydration errors and some components behave differently. How do you debug it?**
A: Follow a systematic process: **reproduce, locate the layer, find the root cause, fix, then add a regression test**. Don't start changing code at random.

1. **Browser DevTools:**
   - Console errors.
   - The Network tab: JS chunks returning 404, failed API calls, status codes.
   - CORS errors.
   - Source maps, so stack traces are readable.
2. **Environment variables:**
   - Is the API base URL set for production?
   - With Vite, only `VITE_`-prefixed variables are exposed, and they are baked in at **build time**, so changing them requires a rebuild.
   - Missing auth config.
3. **Run the production build locally:** `npm run build && npm run preview`. Minification, tree-shaking and dead-code removal can surface bugs that dev mode hides. Dev mode also has StrictMode double-invocation and more lenient behaviour.
4. **Hydration errors (SSR or Next.js):** the server HTML differs from the first client render. Common causes:
   - `window` or `localStorage` read during render.
   - `Math.random()` or `Date.now()` in render, or timezone and locale differences.
   - Invalid HTML nesting (`<div>` inside `<p>`).
   - Different data on the server and the client.

   Fix by moving browser-only code into `useEffect`, using `useId`, or rendering client-only.
5. **Deployment and infrastructure:**
   - Blank page on refresh of a deep route: the SPA fallback is missing (`try_files ... /index.html`).
   - Wrong `base` path when deploying to a subfolder.
   - Case-sensitive imports that work on macOS but fail on Linux CI.
   - CORS and SSL.
   - A reverse proxy or CDN serving a **stale `index.html`** that points to deleted chunks (fix with `no-cache` on `index.html` and long caching for hashed assets).
   - Cookie `Secure`/`SameSite`/domain settings.
6. **Monitoring:** Sentry for errors, Core Web Vitals or RUM for performance, and server logs, to catch the problem before users report it.

Senior-level insight: first determine whether the problem is in the **frontend, backend, infrastructure or rendering**, then fix the root cause. After the fix, add a regression test (unit or E2E) and, where possible, a CI check (for example running E2E tests against the production build) so it can't come back.

```bash
npm run build && npm run preview     # reproduce against the real production bundle locally
```

```js
// Typical root causes, in code
fetch(`${import.meta.env.VITE_API_URL}/users`);   // undefined in prod → "undefined/users" (the variable wasn't set at build time)
import Header from './header';                    // works on macOS, breaks on Linux CI (file is Header.jsx)
<p>{new Date().toLocaleTimeString()}</p>          // SSR hydration mismatch: server time ≠ client time
// nginx missing `try_files $uri /index.html` → refreshing /dashboard returns 404
// CDN caching index.html → it points to deleted chunk hashes → "Failed to fetch dynamically imported module"
```

Interview-ready answer: *"I work systematically. Reproduce it against the production build locally with build and preview. Check the browser console and network tab with source maps. Verify environment variables, since Vite inlines VITE_ variables at build time. Then look for SSR hydration mismatches and deployment issues: missing SPA fallback routes, a wrong base path, case-sensitive imports, CORS, and a stale index.html cached by the CDN. I fix the root cause and add a regression test and monitoring so it can't recur."*

### [P9.1 — Fully Typed React App](P9.1-fully-typed-react-app.md)

**Q: What are the typing requirements?**
A:
- TypeScript **strict mode** plus **`noUncheckedIndexedAccess`**, so `arr[i]` and `record[key]` are typed as possibly `undefined`.
- **Zero `any`**.
- **Zod-validated API responses.**
- **Type-safe route params.**
- **Explicit return types** for every custom hook.

```json
// tsconfig.json
{ "compilerOptions": { "strict": true, "noUncheckedIndexedAccess": true, "noImplicitOverride": true, "exactOptionalPropertyTypes": true } }
```

```ts
const first = tasks[0];          // Task | undefined (because of noUncheckedIndexedAccess)
first.title;                     // ❌ must check first: first?.title

function useTasks(filter: TaskFilter): { tasks: Task[]; isLoading: boolean; error: Error | null } {  // explicit return type
  /* ... */
}

const { taskId } = useParams<{ taskId: string }>();   // typed params (still string | undefined), validated:
const id = z.string().uuid().parse(taskId);
```

Interview-ready answer (project walkthrough): *"A task manager built with maximum type safety: strict mode plus noUncheckedIndexedAccess, so array and record lookups must handle undefined, no any anywhere, Zod validation on every API response, validated route params, and explicit return types on custom hooks so their contracts can't drift silently."*

**Q: What does the app do?**
A: A **task manager** with CRUD, **categories and priorities**, **filtering by status and category**, and a **modal form** for create and edit. It's small in scope on purpose, so the focus stays on modelling the domain with types.

```ts
export const PrioritySchema = z.enum(['low', 'medium', 'high', 'urgent']);
export const StatusSchema = z.enum(['todo', 'in_progress', 'done']);
export const TaskSchema = z.object({
  id: z.string().uuid(),
  title: z.string().min(1).max(120),
  category: z.string(),
  priority: PrioritySchema,
  status: StatusSchema,
  dueDate: z.coerce.date().nullable(),
});
export type Task = z.infer<typeof TaskSchema>;
export type Priority = z.infer<typeof PrioritySchema>;
export type NewTask = Omit<Task, 'id'>;
```

Interview-ready answer: *"A task manager: create, edit and delete tasks with categories, priorities and due dates, filter by status and category, and edit in a modal form. The domain is defined once as Zod schemas, and every type (Task, NewTask, Priority) is derived from them."*

**Q: How does the API client stay type-safe?**
A: Every response is **parsed with Zod**, which **throws a `ZodError`** if the shape is wrong, and the types are **derived from the schemas** (`z.infer`). The client is generic over a schema, so each call returns exactly the validated type, and a backend contract change surfaces immediately at the API boundary.

```ts
class ApiError extends Error { constructor(public status: number, message: string) { super(message); } }

async function request<S extends z.ZodTypeAny>(path: string, schema: S, init?: RequestInit): Promise<z.infer<S>> {
  const res = await fetch(`${env.VITE_API_URL}${path}`, {
    ...init,
    headers: { 'Content-Type': 'application/json', ...init?.headers },
  });
  if (!res.ok) throw new ApiError(res.status, `Request failed: ${res.status}`);
  return schema.parse(await res.json());          // runtime check + static type
}

export const api = {
  listTasks: () => request('/tasks', z.array(TaskSchema)),                 // Promise<Task[]>
  createTask: (t: NewTask) => request('/tasks', TaskSchema, { method: 'POST', body: JSON.stringify(t) }),
};
```

Interview-ready answer: *"The API client has a generic request function that takes a Zod schema, parses the JSON with it, and returns z.infer of that schema. Every call is both validated at runtime and precisely typed at compile time, with no casts. If the backend changes a field, the ZodError points to exactly what broke instead of the UI failing somewhere downstream."*

**Q: How does `PriorityBadge` use the type system?**
A: Its style config is a **`Record<Priority, …>`** keyed by the `Priority` union, so TypeScript **guarantees an entry exists for every priority**. Adding `'urgent'` to the schema makes the config a **compile error** until it's handled, and `config[priority]` can never be `undefined`.

```tsx
const priorityConfig = {
  low:    { label: 'Low',    className: 'bg-gray-100 text-gray-700' },
  medium: { label: 'Medium', className: 'bg-blue-100 text-blue-800' },
  high:   { label: 'High',   className: 'bg-orange-100 text-orange-800' },
  urgent: { label: 'Urgent', className: 'bg-red-100 text-red-800' },
} satisfies Record<Priority, { label: string; className: string }>;   // missing key → compile error

export function PriorityBadge({ priority }: { priority: Priority }) {
  const { label, className } = priorityConfig[priority];              // always defined
  return <span className={`rounded px-2 py-0.5 text-xs ${className}`}>{label}</span>;
}
```

Interview-ready answer: *"The badge looks up its styles in a config typed as Record<Priority, …>, with `satisfies` to keep the literal types. Because the keys must cover the whole Priority union, adding a new priority fails compilation until the badge supports it, and the lookup can never return undefined. It's exhaustiveness checking through the type system."*

### [P9.2 — Accessible Dashboard](P9.2-accessible-dashboard.md)

**Q: What does it need to meet?**
A: **WCAG 2.1 AA**:
- **Keyboard access** to everything, with visible focus.
- **Screen-reader announcements** for dynamic changes.
- **Colour contrast** of at least 4.5:1.
- Proper landmarks and headings.
- **Zero axe violations**, enforced in tests and CI.

```jsx
function DashboardLayout() {
  return (
    <>
      <a href="#main" className="sr-only focus:not-sr-only">Skip to main content</a>
      <header role="banner"><Nav /></header>
      <main id="main" tabIndex={-1}>
        <h1>Dashboard</h1>
        <section aria-labelledby="kpis-h"><h2 id="kpis-h">Key metrics</h2>…</section>
      </main>
      <LiveRegions />          {/* aria-live announcers */}
    </>
  );
}
```

Interview-ready answer (project walkthrough): *"A dashboard built to WCAG 2.1 AA: a skip link, landmarks and a logical heading structure, full keyboard operability with visible focus, live-region announcements for async updates, verified colour contrast including dark mode, text alternatives for charts, and zero axe violations enforced in CI."*

**Q: What contrast ratio is required for text?**
A: At least **4.5:1** for normal text, and **3:1** for **large text** (≥ 18pt, or ≥ 14pt bold) and for **UI components and graphical objects** (borders, icons, focus rings, chart lines). Every status badge's colour combination is verified against these, in **both light and dark mode**, and status is never shown by colour alone.

```jsx
// Verified pairs (e.g. with the WebAIM checker or axe):
// text-green-800 on bg-green-100 → 6.8:1 ✅      text-yellow-600 on white → 2.9:1 ❌ (fails body text)
const statusStyles = {
  healthy:  'bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-100',
  degraded: 'bg-amber-100 text-amber-900 dark:bg-amber-900 dark:text-amber-100',
  down:     'bg-red-100 text-red-800 dark:bg-red-900 dark:text-red-100',
};
<span className={statusStyles[status]}>{statusIcon[status]} {statusLabel[status]}</span>   // icon + text, not colour only
```

Interview-ready answer: *"WCAG AA requires 4.5:1 for normal text and 3:1 for large text and for UI components like icons, borders and focus indicators. I verified every badge and text colour pair in both themes with axe and a contrast checker, and status always uses text or an icon as well, never colour alone."*

**Q: How does the focus trap work?**
A: Three steps:
1. On open it **saves the currently focused element** (`document.activeElement`) and **moves focus into the trap** (its first focusable element).
2. While open, it **wraps Tab and Shift+Tab** between the first and last focusable elements.
3. On close it **restores focus** to the saved element, so keyboard users land back where they were.

```jsx
function useFocusTrap(active) {
  const containerRef = useRef(null);

  useEffect(() => {
    if (!active) return;
    const previouslyFocused = document.activeElement;
    const container = containerRef.current;
    const focusables = () => container.querySelectorAll(
      'a[href], button:not([disabled]), input:not([disabled]), select, textarea, [tabindex]:not([tabindex="-1"])');

    focusables()[0]?.focus();                                  // move focus in

    const onKeyDown = (e) => {
      if (e.key !== 'Tab') return;
      const els = focusables(); const first = els[0]; const last = els[els.length - 1];
      if (e.shiftKey && document.activeElement === first) { e.preventDefault(); last.focus(); }
      else if (!e.shiftKey && document.activeElement === last) { e.preventDefault(); first.focus(); }
    };
    container.addEventListener('keydown', onKeyDown);

    return () => {
      container.removeEventListener('keydown', onKeyDown);
      previouslyFocused?.focus();                              // restore on close
    };
  }, [active]);

  return containerRef;
}
```

Interview-ready answer: *"When activated it remembers the element that had focus, moves focus to the first focusable element inside, and intercepts Tab and Shift+Tab to cycle within the container. Its cleanup restores focus to the remembered element. In production I'd rather use the native dialog element or the inert attribute on the background, which handle this more robustly."*

**Q: How do screen reader announcements work?**
A: A **`useAnnounce`** hook writes messages into **visually hidden `aria-live` regions** (`polite` for status updates, `assertive` for urgent errors) that are **always rendered at the app root**. Live regions must exist *before* their content changes to be announced. The hook **clears the text, then sets it** on the next tick, so **repeating the same message** ("Saved") is still announced.

```jsx
const AnnounceContext = createContext(() => {});

export function AnnouncerProvider({ children }) {
  const [polite, setPolite] = useState('');
  const [assertive, setAssertive] = useState('');

  const announce = useCallback((message, priority = 'polite') => {
    const set = priority === 'assertive' ? setAssertive : setPolite;
    set('');                                           // clear first...
    setTimeout(() => set(message), 50);                // ...then set, so identical messages are re-announced
  }, []);

  return (
    <AnnounceContext.Provider value={announce}>
      {children}
      <div aria-live="polite" aria-atomic="true" className="sr-only">{polite}</div>
      <div aria-live="assertive" aria-atomic="true" className="sr-only">{assertive}</div>
    </AnnounceContext.Provider>
  );
}
export const useAnnounce = () => useContext(AnnounceContext);

// const announce = useAnnounce(); announce('Report exported'); announce('Failed to save', 'assertive');
```

Interview-ready answer: *"Persistent visually hidden live regions sit at the app root: polite for status updates, assertive for errors. A useAnnounce hook updates their text. It clears the region first and sets the message a tick later, because screen readers only announce changes, so the same message twice in a row would otherwise be skipped."*

**Q: What else is included?**
A:
- A **skip navigation link**.
- A **sortable, keyboard-selectable data table** (`aria-sort`, header buttons, arrow-key row selection).
- **Charts with text alternatives**: a summary and a data table fallback.
- **Announced loading states.**
- **Dark mode with maintained contrast.**
- `prefers-reduced-motion` support.

```jsx
// A chart with a text alternative
<figure aria-labelledby="rev-cap">
  <RevenueChart data={data} aria-hidden="true" />
  <figcaption id="rev-cap">Revenue rose 24% from January to June, peaking in May at ₹4.2M.</figcaption>
  <details><summary>View data table</summary><DataTable rows={data} /></details>
</figure>

// Announced loading
{isLoading ? <p role="status">Loading metrics…</p> : <Metrics data={data} />}

// Reduced motion
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
```

Interview-ready answer: *"A skip link to jump past navigation, a data table with accessible sorting and keyboard row selection, charts with written summaries and a data table fallback, loading states exposed via role status so they're announced, a dark mode verified for contrast, and reduced-motion support. Each was checked with axe plus a manual screen-reader pass."*

### [P9.3 — Production SPA](P9.3-production-spa.md)

**Q: What does it combine?**
A: Everything in the curriculum, in a **Bookmark Manager**:
- **TypeScript strict** mode and **Zod** validation.
- **80%+ test coverage**: unit, integration (MSW) and E2E (Playwright).
- **English + French i18n.**
- **WCAG 2.1 AA.**
- **Docker + nginx**, and **GitHub Actions CI/CD.**

```
bookmark-manager/
├── src/
│   ├── features/bookmarks/   (api.ts with Zod, hooks with React Query, components, tests)
│   ├── components/ui/        (accessible, CVA-based)
│   ├── config/env.ts         (validated VITE_ vars)
│   ├── i18n/                 (react-i18next setup, typed keys)
│   └── mocks/                (MSW handlers)
├── public/locales/{en,fr}/
├── e2e/                      (Playwright + axe)
├── Dockerfile, nginx.conf
├── lighthouserc.json         (performance budget)
└── .github/workflows/ci.yml
```

Interview-ready answer (project walkthrough): *"A capstone bookmark manager that combines every practice: strict TypeScript with Zod at the boundaries, React Query for server state, accessible components, English and French i18n with typed keys, unit, integration and E2E tests above 80% coverage, a Lighthouse CI performance budget, and Docker with nginx deployed through GitHub Actions."*

**Q: What is the performance budget?**
A: Enforced with **Lighthouse CI** on every PR: **LCP < 2.5 s**, **TBT < 200 ms** (the lab proxy for INP) and **CLS < 0.1**, plus a minimum performance score, so a regression **fails the build** instead of being discovered in production.

```json
// lighthouserc.json
{
  "ci": {
    "collect": { "staticDistDir": "./dist", "numberOfRuns": 3 },
    "assert": {
      "assertions": {
        "categories:performance": ["error", { "minScore": 0.9 }],
        "largest-contentful-paint": ["error", { "maxNumericValue": 2500 }],
        "total-blocking-time": ["error", { "maxNumericValue": 200 }],
        "cumulative-layout-shift": ["error", { "maxNumericValue": 0.1 }],
        "categories:accessibility": ["error", { "minScore": 1 }]
      }
    }
  }
}
```

Interview-ready answer: *"Lighthouse CI runs against the production build on every pull request and asserts LCP under 2.5 seconds, total blocking time under 200 ms and CLS under 0.1, plus minimum performance and accessibility scores. A regression fails the pipeline, so performance is guarded continuously rather than audited once. Real-user web-vitals monitoring covers production."*

**Q: How is "no `any`" enforced?**
A: With **ESLint**: `@typescript-eslint/no-explicit-any` set to **error**, plus the `no-unsafe-*` rules from the type-checked config, which catch `any` leaking in from libraries or `JSON.parse`, run in CI so it can't be merged. Unknown data is typed as **`unknown`** and narrowed or validated with Zod.

```js
// eslint.config.js
import tseslint from 'typescript-eslint';
export default tseslint.config(
  ...tseslint.configs.strictTypeChecked,
  {
    languageOptions: { parserOptions: { projectService: true } },
    rules: {
      '@typescript-eslint/no-explicit-any': 'error',
      '@typescript-eslint/no-unsafe-assignment': 'error',
      '@typescript-eslint/no-unsafe-member-access': 'error',
    },
  },
);
```

```ts
const data: unknown = JSON.parse(raw);          // ✅ unknown, not any
const settings = SettingsSchema.parse(data);    // narrowed by validation
```

Interview-ready answer: *"The typescript-eslint no-explicit-any rule is an error, and the type-checked unsafe-* rules catch implicit any coming from libraries or JSON.parse. Lint runs in CI, so it can't be merged. When data is genuinely unknown I type it as unknown and narrow it with Zod or type guards, rather than switching type checking off."*

**Q: What accessibility extras does it require beyond P9.2?**
A: SPA-specific ones. The **page title updates on route change** (and focus moves to the new page's heading), so screen-reader users know navigation happened, because an SPA never triggers a page load announcement. **Loading states are announced** to screen readers.

```jsx
function usePageTitle(titleKey) {
  const { t } = useTranslation();
  const headingRef = useRef(null);
  const location = useLocation();

  useEffect(() => {
    document.title = `${t(titleKey)} · Bookmarks`;     // announced, and shows in the tab and history
    headingRef.current?.focus();                       // move focus to the new page's <h1 tabIndex={-1}>
  }, [location.pathname, titleKey, t]);

  return headingRef;
}

// Loading
{isPending && <p role="status">{t('loading.bookmarks')}</p>}
```

Interview-ready answer: *"Route changes in an SPA are silent to screen readers, so each route sets a translated document title and moves focus to its main heading, which is announced and puts keyboard users at the start of the new content. Async loading states use role status, so users hear that content is loading and when it's ready."*

**Q (Scenario): How would you design a scalable folder structure for an enterprise app with auth, dashboards, reports, payments, users and notifications?**
A: Use a **feature-based** structure, where code for one business domain lives together, rather than grouping every file by technical type (`components/`, `hooks/`, `services/` at the root). A type-based layout scatters one feature across ten folders.

```
src/
├── app/                    # app shell: wiring only
│   ├── App.tsx
│   ├── routes.tsx          # lazy-loaded feature routes
│   └── providers/          # QueryClient, Auth, Theme, i18n, ErrorBoundary
├── features/
│   ├── auth/
│   │   ├── api/            # API calls + React Query hooks (useLogin)
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── pages/
│   │   ├── types.ts
│   │   └── index.ts        # public API: other features import only from here
│   ├── users/
│   ├── payments/
│   ├── reports/
│   └── notifications/
├── components/             # shared, feature-agnostic
│   ├── ui/                 # Button, Modal, Input (design system)
│   └── layout/             # Header, Sidebar, PageLayout
├── hooks/                  # shared hooks (useDebounce, useMediaQuery)
├── lib/                    # configured clients: axios instance, queryClient
├── utils/
├── constants/
├── types/
└── assets/
```

Architectural principles:
1. Keep business logic **close to its feature**, and colocate its tests.
2. Each feature exposes a **public API** (`index.ts`). Avoid deep imports and cross-feature dependencies; if two features need the same thing, move it to the shared layer. You can enforce this with ESLint (`import/no-restricted-paths`, `eslint-plugin-boundaries`).
3. Dependencies go one way: `app → features → shared`. Shared code never imports from features.
4. Keep the API or service layer separate from presentational components.
5. Keep global providers in one place, in `app/providers`.
6. Use TypeScript, path aliases (`@/features/...`) and **lazy-loaded routes per feature**.

Senior-level insight: for a large team, also define coding conventions, code ownership (`CODEOWNERS`), testing standards and clear module boundaries. As the app grows, the next steps are a monorepo (Turborepo or Nx) with shared packages, and micro-frontends only if teams need independent deployments. Don't over-engineer a small app; a flat structure is fine until it starts to hurt.

**Q (Senior / Tech Lead scenario): How would you architect the frontend of a marketplace or social platform with millions of users, real-time notifications, large media, complex dashboards and high API traffic?**
A: Design for **modularity, performance, reliability and efficient communication with the backend**.

1. **Frontend architecture:**
   - React + TypeScript.
   - **Next.js** where SSR, SSG, ISR or Server Components help (public, SEO-relevant pages such as product and profile pages); CSR is fine for authenticated dashboards.
   - Feature-based modules and a shared design-system component library.
2. **State management:**
   - Local state: `useState`/`useReducer`.
   - URL state: search params.
   - Global client state: Zustand or Redux Toolkit.
   - **Server state: TanStack Query.**
3. **Performance:**
   - Route-level code splitting and lazy loading.
   - Virtualisation for feeds and tables.
   - **Image optimisation** (responsive `srcset`, AVIF/WebP, lazy loading, an image CDN).
   - CDN and edge caching for static assets and pages.
   - Debouncing and throttling.
   - Memoisation guided by profiling.
   - Performance budgets in CI.
4. **API communication:**
   - REST or GraphQL depending on the data shape (GraphQL or a BFF for complex dashboards that need many resources).
   - A central API client with interceptors for auth refresh, retries with backoff, and request cancellation.
   - Response caching.
   - Cursor-based pagination and infinite scroll.
5. **Real-time:**
   - **WebSockets** for two-way features (chat, presence).
   - **Server-Sent Events** for one-way streams (notifications, live counts).
   - **Push notifications** for background alerts.
   - Reconnection with backoff, and merging live events into the React Query cache.
6. **Large media:** direct-to-storage uploads with presigned URLs, chunked or resumable uploads with progress, and video through adaptive streaming (HLS).
7. **Security:**
   - Secure auth: HTTP-only cookies, refresh token rotation.
   - RBAC, with checks on the backend too.
   - XSS prevention: no `dangerouslySetInnerHTML` without sanitising, and a Content Security Policy.
   - CSRF protection.
   - Input validation on both client and server.
   - HTTPS everywhere.
   - Dependency auditing.
8. **Monitoring:**
   - Core Web Vitals from real users (RUM).
   - JS errors in Sentry.
   - API latency.
   - Bundle size tracking.
   - Feature flags and gradual rollouts, for safe releases.

```
                Users / Browser
                       │
                CDN / Edge Cache
                       │
                React / Next.js
          ┌────────────┼────────────┐
      UI State      API Layer     WebSocket / SSE
   (Zustand/Redux) (TanStack Query)  (real-time)
                       │
            Backend services / BFF
          ┌────────────┼────────────┐
         Auth       Products      Orders
          └────────────┼────────────┘
                   Databases
```

Senior-level insight: **don't start with micro-frontends or complex infrastructure** just because "it needs to scale". Start with a well-structured modular app, measure the real bottlenecks, and add complexity (micro-frontends, edge rendering, a BFF) only when team size, independent deployments or performance data justify it. The frontend's scalability also depends on backend capacity, caching, database performance and infrastructure, so design it together with the backend team.

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
A: jQuery is **imperative**: you find DOM nodes and mutate them step by step. React is **declarative**: you describe **what the UI should look like for the current state**, and React updates the DOM. Mixing the two (querying and mutating the DOM React manages) makes the DOM and React's state disagree, and React will overwrite or crash on your changes. If you reach for `document.getElementById`, ask how to express it **with state**, or, for genuine imperative needs like focus, measuring or third-party widgets, use a **ref**.

```jsx
// ❌ jQuery thinking: React doesn't know about this change and will overwrite it on the next render
function Toggle() {
  return <button onClick={() => document.getElementById('panel').style.display = 'none'}>Hide</button>;
}

// ✅ React: state describes the UI
function Toggle() {
  const [open, setOpen] = useState(true);
  return (
    <>
      <button onClick={() => setOpen((o) => !o)} aria-expanded={open}>{open ? 'Hide' : 'Show'}</button>
      {open && <div id="panel">…</div>}
    </>
  );
}

// ✅ Legitimate imperative work goes through a ref
const inputRef = useRef(null);
useEffect(() => inputRef.current.focus(), []);
```

Interview-ready answer: *"jQuery manipulates the DOM directly, while React derives the DOM from state. If I mutate the DOM behind React's back, the two get out of sync and React will overwrite or break my changes. In React I change state and let rendering update the UI, and I keep direct DOM access to refs for things state can't express, like focus, measurement or integrating a non-React library."*

**Q: Why shouldn't server data go in `useState` + `useEffect`?**
A: From Phase 4 on, all API data goes through React Query, which handles caching, staleness and refetching. Hand-rolled `useEffect` fetching has to reimplement all of that, and it's easy to get wrong:
- **No cache or deduplication**: every mount refetches, and five components make five requests.
- **Race conditions and missing cancellation.**
- **Loading and error boilerplate** in every component.
- **Stale data**: no refetch on focus or reconnect, and nothing invalidated after mutations.
- **Waterfalls**, and **no retries or pagination** helpers.

```jsx
// ❌ ~20 lines, and still no cache, retries, dedupe or invalidation
const [user, setUser] = useState(null);
const [loading, setLoading] = useState(true);
const [error, setError] = useState(null);
useEffect(() => {
  let ignore = false;
  setLoading(true);
  fetchUser(id).then((u) => !ignore && setUser(u)).catch((e) => !ignore && setError(e)).finally(() => !ignore && setLoading(false));
  return () => { ignore = true; };
}, [id]);

// ✅ One line, with caching, deduplication, background refetch, retries and invalidation
const { data: user, isPending, error } = useQuery({ queryKey: ['user', id], queryFn: () => fetchUser(id) });
```

Interview-ready answer: *"Server data isn't really component state. It's a cache of something the server owns. useState plus useEffect gives you none of the cache behaviour: no deduplication, no background refetching, race conditions unless you add cancellation, and no invalidation after writes. React Query handles all of that in one hook, so I keep useState for UI state and use React Query, or loaders, for server data."*

### Interview Quick Revision

**Q: What is the one-line takeaway for each common interview topic?**
A:

| Topic | Key concept to remember |
|---|---|
| Virtual DOM vs Real DOM | An in-memory JS tree, diffed and minimally applied to the DOM |
| Reconciliation / Fiber | O(n) diff by element type and keys; interruptible, prioritised work |
| Props vs State | Read-only input vs owned, changeable memory |
| useState vs useRef | Triggers a re-render vs a persistent mutable value |
| useEffect vs useLayoutEffect | After paint vs before paint |
| Prop drilling | Composition, Context, or a state library |
| Controlled vs Uncontrolled | React state vs the DOM as the source of truth |
| Redux vs Context | Complex, frequently changing state vs shared low-frequency values |
| React.memo | Memoises a component (shallow prop comparison) |
| useMemo vs useCallback | Caches a value vs caches a function |
| Lazy loading | `React.lazy` + `Suspense`, route-level code splitting |
| Error Boundaries | Catch render errors (not event handlers or async code) |
| Production optimisation | Component, bundle, rendering, API, monitoring |
| Render vs Commit | Compute the changes (pure, interruptible) vs apply them to the DOM |
| useTransition vs useDeferredValue | Lower the priority of an update vs a value |
| Custom Hooks | Reuse stateful logic, not state |
| Rules of Hooks | Same call order every render |
| Keys vs Index | Stable identity; indexes break on reorder |
| Automatic Batching | Many setState calls produce one render (React 18, `createRoot`) |
| CSR vs SSR vs SSG | Browser vs per-request server vs build time |
| Promise methods | all = all or fail, allSettled = all results, race = first settled, any = first success |
| Memory leaks | Clean up timers, listeners, requests and subscriptions |
| React 19 | Actions, `useActionState`, `useOptimistic`, `use()`, ref as a prop |
| Server Components | Server-only, zero JS shipped; `'use client'` only for interactive leaves |
| Concurrent Rendering | Interruptible, prioritised rendering on one thread, not multithreading |
| Effect hooks | insertion, then layout (before paint), then effect (after paint) |
| Actions | Async form handling with built-in pending state |
| External stores | `useContext` (React value) vs `useSyncExternalStore` (external store, no tearing) |
| Hydration | Server HTML must match the first client render |
| Architecture | Feature-based folders with public APIs and one-way dependencies |
| Testing | Vitest/Jest, RTL, MSW, Playwright; test behaviour, not coverage % |
| Scalability | Modular first, measure, add complexity only when justified |

**Q: Which scenario topics matter most in a senior React interview?**
A: Make sure you can explain each of these without notes. The scenario answers are in the linked sections above.

| Priority | Topic | Importance | Where in this file |
|---|---|---|---|
| 1 | Re-render optimisation | Very high | A7.1 |
| 2 | Debouncing and throttling | Very high | A1.4, A7.1 |
| 3 | Race conditions | Very high | A1.4 |
| 4 | Stale closures | Very high | A1.4 |
| 5 | JWT authentication and protected routes | Very high | A3.4 |
| 6 | State management architecture | Very high | A2.2 |
| 7 | Virtualisation | High | A7.4 |
| 8 | Complex form handling | High | A5.3 |
| 9 | Error boundaries | High | A9.4 |
| 10 | Production debugging | Very high | A9.4 |

**Q: How should you structure an answer to a scenario-based question?**
A:
1. **Identify the problem.** Restate it, and ask clarifying questions about scale, users and constraints.
2. **Explain the root cause.**
3. **Describe your solution**, ideally with a short code sketch.
4. **Explain why you chose it** over the alternatives, including the trade-offs.
5. **Cover performance, scalability and security considerations**, plus how you'd test and monitor it.

Wherever you can, tie it to **a real production app you worked on**, with a measured result (for example "route-level splitting cut the initial bundle from 900 kB to 350 kB"). This shows engineering ownership, not just theoretical React knowledge.

**Q: Which five topics must you be able to explain confidently before going deeper?**
A:
1. React Fiber and reconciliation (A1.1).
2. `useEffect` and stale closures (A1.4).
3. React 19 and Server Components (A1.1, A5.1, A9.4).
4. State management architecture (A2.2).
5. React performance optimisation (A7.1–A7.4).
