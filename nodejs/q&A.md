# Node.js — Questions & Answers

Questions and answers drawn from every file in this folder, in curriculum order. Each section links back to its source file.

- [Phase 1 — Node.js Runtime Fundamentals](#phase-1--nodejs-runtime-fundamentals)
- [Phase 2 — Asynchronous Programming](#phase-2--asynchronous-programming)
- [Phase 3 — File System & Streams](#phase-3--file-system--streams)
- [Phase 4 — Networking & HTTP](#phase-4--networking--http)
- [Phase 5 — Security & Cryptography](#phase-5--security--cryptography)
- [Phase 6 — Performance & Concurrency](#phase-6--performance--concurrency)
- [Phase 7 — Testing & Debugging](#phase-7--testing--debugging)
- [Backend Architecture & Leadership](#backend-architecture--leadership)
- [Study Strategy](#study-strategy)

---

## Phase 1 — Node.js Runtime Fundamentals

### [A1.1 — How Node.js Runs JavaScript](A1.1-how-nodejs-runs-javascript.md)

**Q: What is Node.js, what is it made of, and how does it work internally?**
A: Node.js is an open-source, cross-platform **JavaScript runtime** that runs JavaScript outside the browser. It gives JS access to the file system, network sockets, HTTP servers, processes and OS features, which makes it usable for APIs, CLIs, scripts and real-time apps. It is made of:
- **V8:** Google's engine. It parses, JIT-compiles and garbage-collects JavaScript.
- **libuv:** a C library that provides the **event loop**, non-blocking I/O, and a thread pool.
- **Bindings and the standard library** (`node:fs`, `node:http`, `node:crypto`…), which expose OS features to JS.

How a request flows:
1. V8 executes your JavaScript on the main thread.
2. When you start an async operation (read a file, query a DB, make an HTTP call), Node hands it off. Network I/O goes to the OS's non-blocking APIs (epoll, kqueue, IOCP); file system, DNS and some crypto work goes to libuv's thread pool.
3. The main thread keeps running other JavaScript instead of waiting.
4. When the operation completes, its callback (or promise continuation) is queued.
5. The **event loop** picks it up and runs it on the main thread.

Interview-ready answer: *"Node.js is a JavaScript runtime built on V8. V8 executes the JavaScript, while libuv provides the event loop and asynchronous I/O. Because of its non-blocking I/O model, one process can handle many concurrent connections without a thread per request. That makes it a strong fit for I/O-heavy work such as REST APIs, real-time apps and microservices."*

Senior-level insight: Node is **not** the right default for CPU-heavy work (image processing, large computations). That work blocks the single JS thread. Move it to Worker Threads (A6.2), a separate service, or a job queue.

**Q: What is libuv, what is its role, and what runs on its thread pool?**
A: libuv is the cross-platform **C library** that gives Node its asynchronous engine. Think of Node as a manager that delegates work to a team, and libuv as that team. Its responsibilities:
1. The **event loop** and its phases.
2. **Async I/O** on top of the OS's non-blocking mechanisms (epoll on Linux, kqueue on macOS, IOCP on Windows) for TCP/UDP sockets, pipes, TTYs and signals.
3. The **thread pool** for work the OS can't do asynchronously.
4. Timers, child processes, and file-system watching.

What runs on the **thread pool**: file system operations, DNS lookups (`dns.lookup`) and some crypto, plus zlib compression. Examples are `crypto.pbkdf2`, `scrypt` and `randomBytes` with a callback. It has **4 threads by default**, configurable with `UV_THREADPOOL_SIZE` (up to 1024). Set it **before** the pool is first used, ideally as an environment variable:

```bash
UV_THREADPOOL_SIZE=8 node app.js
```

Network sockets use the OS's non-blocking APIs instead, so 10,000 open connections don't need 10,000 threads; they don't use the pool at all.

What happens on `fs.readFile`: JS calls into Node's C++ binding, libuv queues the read on a pool thread, the thread does the blocking `read()`, libuv is notified on completion, and the event loop runs your callback on the main thread.

Interview-ready answer: *"libuv is the C library behind Node's event loop and async I/O. It uses OS mechanisms like epoll for network I/O, and a thread pool of 4 threads by default for work like file system access, DNS lookups, compression and some crypto. That's what lets Node do non-blocking I/O while running JavaScript on one thread."*

Senior-level insight: if an app does lots of `fs`, `pbkdf2`/`bcrypt` or zlib work, the 4 pool threads become a **hidden bottleneck**. Requests queue for a thread even though the CPU is idle; a classic symptom is slow password hashing slowing down unrelated file reads. Raising `UV_THREADPOOL_SIZE` helps that specific case, but it does nothing for network I/O or for CPU-heavy JavaScript, which needs workers.

**Q: Is Node.js single-threaded or multithreaded?**
A: Both, depending on what you mean:
- **Your JavaScript** runs on **one main thread**. Callbacks never run in parallel with each other.
- **The runtime is multithreaded:**
  - The libuv thread pool runs file system work, DNS lookups, and crypto such as `pbkdf2`, `scrypt` and zlib.
  - V8 uses extra threads for garbage collection and compilation.
  - Network I/O uses the OS's async mechanisms.
- **You can add threads yourself** with Worker Threads for CPU-heavy JavaScript.

```js
const fs = require('node:fs');

console.log('Start');
fs.readFile('data.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log('File reading completed');
});
console.log('End');
// Start → End → File reading completed
```

`readFile` hands the work to the thread pool, so the main thread keeps going and prints `End` before the file is read.

Interview-ready answer: *"JavaScript execution in Node is single-threaded, but the runtime isn't. libuv's thread pool handles file system, DNS and some crypto work, and network I/O uses the OS's async APIs. For CPU-heavy work I use Worker Threads."*

**Q: How can one Node process handle thousands of connections?**
A: JavaScript runs on one thread, but I/O is handed to libuv; the main thread never waits idle for I/O, and callbacks run when results are ready. Single-threaded JS does **not** mean one request at a time. Node doesn't create 1,000 threads for 1,000 requests; one thread serves them all, because each request spends most of its time waiting on I/O, not running JS.

**Q: What is the downside of single-threaded execution?**
A: CPU-heavy JavaScript blocks everything — a 2-second loop delays all timers and requests by 2 seconds.

**Q: Which web-standard globals does Node provide?**
A: `fetch`, `URL`, `AbortController`, `TextEncoder`, `structuredClone`, `setTimeout`, `crypto.randomUUID()` — plus Node-specific `process`, `Buffer` and `globalThis`. There is no `window` or `document`.

**Q: What do `--watch`, `--env-file` and `--inspect` do?**
A: Restart on file changes, load environment variables from a file, and enable the debugger.

### [A1.2 — Modules: CommonJS & ESM](A1.2-modules-commonjs-and-esm.md)

**Q: CommonJS vs ES Modules (`require()` vs `import`)?**
A: CJS uses synchronous `require()` and `module.exports`, with cached results. ESM uses static `import`/`export`, loads asynchronously, and allows top-level `await`.

```js
// CommonJS: user.js
const getUser = () => ({ id: 1, name: 'Pulkit' });
module.exports = getUser;
// app.js
const getUser = require('./user');

// ES Modules: user.js
export const getUser = () => ({ id: 1, name: 'Pulkit' });
// app.js
import { getUser } from './user.js'; // the extension is required in ESM
```

| Feature | CommonJS | ES Modules |
|---|---|---|
| Import / export | `require()` / `module.exports` | `import` / `export` |
| Loading | Synchronous, at runtime (`require` can sit inside an `if`) | Static, parsed before execution; asynchronous loading |
| Exports | A copied value | **Live bindings** |
| Top-level `await` | No | Yes |
| Tree shaking | Limited (dynamic) | Good (static analysis) |
| Dynamic loading | `require()` or `import()` | `import()` |
| `__dirname`, `__filename` | Available | Use `import.meta.dirname`/`filename` |
| Status | The traditional Node system | The JavaScript standard |

Interview-ready answer: *"CommonJS uses `require` and `module.exports` and loads modules synchronously. ES Modules use `import`/`export`, are statically analysable, and support live bindings, top-level await and better tree-shaking. For new projects I prefer ESM (or TypeScript compiled to it), keeping CJS only where older dependencies or tooling need it."* How to enable ESM is covered in the next question.

**Conditional loading** is a common follow-up. `require()` is an ordinary function call, so it can sit inside an `if`. Static `import` declarations are resolved **before** the module runs and must be at the top level, so use dynamic `import()` instead. It returns a promise and works in both systems.

```js
// CommonJS
if (process.env.NODE_ENV === 'development') {
  const logger = require('./logger');
}

// ESM
if (process.env.NODE_ENV === 'development') {
  const { default: logger } = await import('./logger.js');
}
```

Can one project use both? Yes. See "How do ESM and CJS interoperate?" below. Pick one system per package and keep it consistent, because mixing them is a common source of configuration bugs.

**Q: How does Node decide which system a file uses?**
A: `.mjs` is always ESM, `.cjs` always CJS; `.js` is ESM if the nearest `package.json` has `"type": "module"`, otherwise CJS.

**Q: What are the key ESM differences?**
A: Relative imports need file extensions; there's no `require`, `module`, `__filename` or `__dirname` (use `import.meta.filename`/`dirname`); bindings are live; top-level `this` is `undefined`.

**Q: What is a live binding?**
A: If a module changes an exported `let`, importers see the new value. CJS exports are copies taken at `require` time.

**Q: Why use the `node:` prefix?**
A: It makes clear you mean the built-in module, not an npm package with the same name.

**Q: How do ESM and CJS interoperate?**
A: ESM can import CJS (`module.exports` becomes the default export). CJS loads ESM with dynamic `import()`; newer Node versions also support `require()` of synchronous ESM.

**Q: How do you import JSON in ESM?**
A: With an import attribute: `import pkg from './package.json' with { type: 'json' }`.

### [A1.3 — npm & package.json](A1.3-npm-and-package-json.md)

**Q: What are the key `package.json` fields?**
A: `type`, `scripts`, `dependencies`, `devDependencies`, `engines`, `main`/`exports`, and `bin`.

**Q: What do `^`, `~` and exact versions mean?**
A: `^1.4.2` allows any `1.x.x` ≥ 1.4.2; `~1.4.2` allows `1.4.x`; `1.4.2` is exact. For `0.x`, `^0.4.2` only allows `0.4.x`.

**Q: Why commit `package-lock.json`, and when use `npm ci`?**
A: The lockfile records exact versions and integrity hashes of the whole tree. `npm ci` installs exactly that and fails on mismatch — use it in CI and Docker.

**Q: What is a phantom dependency?**
A: A package you import that you never declared, which works only because npm hoisted it into `node_modules`.

**Q: How do npm scripts find local CLIs?**
A: `npm run` puts `./node_modules/.bin` on the `PATH`. `pre`/`post` scripts run automatically, and arguments after `--` are passed through.

**Q: How do you reduce supply-chain risk?**
A: Few, well-maintained dependencies, `npm audit`, reviewing install scripts, and `npm ci --ignore-scripts` when native builds aren't needed.

### [A1.4 — Process, Globals & the CLI](A1.4-process-globals-and-the-cli.md)

**Q: What does `process.argv` contain?**
A: `[0]` is the node binary, `[1]` the script path, and the rest are user arguments.

**Q: `process.cwd()` vs `import.meta.dirname`?**
A: `cwd()` is where the user ran the command; `import.meta.dirname` is the directory of the script file.

**Q: Why prefer `process.exitCode` over `process.exit()`?**
A: `exit()` quits immediately and can cut off pending async work and unflushed output; setting `exitCode` lets the process finish naturally.

**Q: Why write diagnostics to stderr?**
A: So users can pipe stdout results (`tool | jq`) without errors mixed in.

**Q: How do you parse arguments without a dependency?**
A: `util.parseArgs` supports long/short options, booleans, strings, defaults and positionals, and throws on unknown options.

**Q: How do you read piped vs interactive input?**
A: Iterate a `readline` interface over `process.stdin` for piped lines; use `readline/promises` `rl.question()` for prompts; check `process.stdin.isTTY` to tell them apart.

**Q: What happens when you add a SIGINT listener?**
A: Node no longer exits on Ctrl+C by default — you're responsible for cleanup and exiting (conventionally code 130).

### [P1.1 — CLI Todo App](P1.1-cli-todo-app.md)

**Q: Where is data stored and how is it overridden?**
A: In `~/.todo.json`, overridable with the `TODO_FILE` environment variable.

**Q: How are saves made crash-safe?**
A: Write to a temp file, then `rename` over the real file, so a crash never leaves a half-written file.

**Q: How are exit codes used?**
A: 0 for success, 1 for runtime errors, 2 for usage errors (via a `UsageError` class).

**Q: How does colour output behave when piped?**
A: `picocolors` disables colours automatically when stdout isn't a TTY or `NO_COLOR` is set.

**Q: How is the tool made runnable as `todo`?**
A: A `bin` field in `package.json`, a shebang in `bin/todo.js`, and `npm link` during development.

### [P1.2 — Project Scaffolder CLI](P1.2-project-scaffolder-cli.md)

**Q: How does the scaffolder work?**
A: It asks questions (or reads flags), copies a template directory recursively, substitutes `{{name}}`-style placeholders in contents and file names, and prints next steps.

**Q: Why are templates' `.gitignore` files named `_gitignore`?**
A: npm strips `.gitignore` files when publishing, so they're renamed on copy.

**Q: How are templates found regardless of the user's cwd?**
A: Relative to the tool with `import.meta.dirname`.

**Q: What safety checks does it have?**
A: It refuses non-empty target directories unless `--force`, and re-asks or rejects invalid template choices.

---

## Phase 2 — Asynchronous Programming

### [A2.1 — Callbacks & Error-First Patterns](A2.1-callbacks-and-error-first-patterns.md)

**Q: What is the difference between synchronous and asynchronous programming?**
A: **Synchronous** code runs one step at a time and **blocks** the thread until each operation finishes. **Asynchronous** code **starts** an operation, carries on with other work, and handles the result when it's ready, through a callback, a promise or `await`.

| Synchronous | Asynchronous |
|---|---|
| Blocks while waiting | Doesn't block while waiting |
| Strictly sequential | Other work continues meanwhile |
| Simpler to read | You have to handle results later |
| On a server, one slow call stalls **every** request | High I/O concurrency on one thread |

```js
// Synchronous: the whole process waits for the disk
console.log('Start');
const data = fs.readFileSync('data.txt', 'utf8');
console.log(data);
console.log('End');            // Start → (file content) → End

// Asynchronous: Node carries on and handles the result later
console.log('Start');
fs.readFile('data.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});
console.log('End');            // Start → End → (file content)
```

Interview-ready answer: *"Synchronous code blocks the thread until each operation finishes. Asynchronous code starts the operation and keeps executing, handling the result later. In Node backends we use async, non-blocking I/O, because a blocked main thread blocks every request."*

Senior-level insight: async does **not** make an individual operation faster; the file takes as long to read either way. The benefit is **throughput**: the thread does useful work while waiting for I/O. Sync APIs are fine at startup, in CLIs and in scripts (see A3.1), never in a request handler.

**Q: What is a callback?**
A: A function passed as an argument to another function, to be called later when a task completes or an event happens. Callbacks can be synchronous (`array.map(fn)`) or asynchronous (`fs.readFile(path, cb)`, `server.on('request', cb)`).

```js
function greet(name, callback) {
  console.log('Hello ' + name);
  callback();
}
greet('Pulkit', () => console.log('Goodbye!')); // Hello Pulkit → Goodbye!

fs.readFile('data.txt', 'utf8', (err, data) => {   // async, error-first
  if (err) return console.error(err);
  console.log(data);
});
```

Interview-ready answer: *"A callback is a function passed to another function and called when an operation completes or an event fires. Node's core APIs historically use error-first callbacks for async I/O. Deeply nested callbacks lead to callback hell, which we solve with Promises and async/await."*

**Q: What is the error-first callback convention?**
A: The callback is the last argument and receives `(err, ...results)`; `err` is `null` on success. Always check `err` first.

**Q: Why can't try/catch catch errors in callbacks?**
A: The callback runs later on a fresh call stack, after the `try` block has finished; errors must be passed as values.

**Q: What are the rules for writing callback APIs?**
A: Call back exactly once (`return callback(err)`), always asynchronously, and never throw from an async function.

**Q: What is "releasing Zalgo"?**
A: An API that sometimes calls back synchronously (e.g. on a cache hit) and sometimes asynchronously, making ordering unpredictable. Defer sync results with `process.nextTick`.

**Q: What is callback hell?**
A: Deeply nested sequential callbacks with duplicated error handling and manual counters for parallelism. It is also called the "pyramid of doom".

```js
getUser(userId, (err, user) => {
  if (err) return handle(err);
  getOrders(user.id, (err, orders) => {
    if (err) return handle(err);
    getPayment(orders[0].id, (err, payment) => {
      if (err) return handle(err);
      console.log(payment);
    });
  });
});

// The same flow with async/await: flat, with one error handler
try {
  const user = await getUser(userId);
  const orders = await getOrders(user.id);
  const payment = await getPayment(orders[0].id);
  console.log(payment);
} catch (err) {
  handle(err);
}
```

Fix it with promises and async/await (convert old APIs with `util.promisify`), or at least with named functions instead of nested anonymous ones.

**Q: How do you convert callback APIs to promises?**
A: `util.promisify(fn)` — or use built-in promise modules like `node:fs/promises`.

### [A2.2 — Promises & async/await](A2.2-promises-and-async-await.md)

**Q: What is a Promise, and what are its states?**
A: An object representing the **eventual result of an async operation**. Think of a food order: it is being prepared (pending), then delivered (fulfilled) or cancelled (rejected).

| State | Meaning |
|---|---|
| Pending | Still running |
| Fulfilled | Completed successfully, with a value |
| Rejected | Failed, with a reason (an `Error`) |

A promise settles **exactly once**; after that its state never changes. `.then` callbacks always run asynchronously as microtasks, even if the promise is already settled.

```js
const getUser = new Promise((resolve, reject) => {
  const success = true;
  success ? resolve({ id: 1, name: 'Pulkit' }) : reject(new Error('User not found'));
});

getUser
  .then((user) => console.log(user))           // fulfilment
  .catch((error) => console.error(error.message)) // rejection, from anywhere above in the chain
  .finally(() => console.log('Operation completed')); // cleanup, runs either way
```

For the combinators (`all`, `allSettled`, `race`, `any`), see below.

**Q: What is async/await, and how is it different from Promises?**
A: `async/await` is **syntax built on top of Promises**. It is not a replacement for them. It lets async code read top to bottom instead of as a `.then()` chain.

```js
// Promise chain
function showOrders() {
  return fetchUser()
    .then((user) => fetchOrders(user.id))
    .then((orders) => console.log(orders))
    .catch((error) => console.error(error));
}

// async/await
async function showOrders() {
  try {
    const user = await fetchUser();
    const orders = await fetchOrders(user.id);
    console.log(orders);
  } catch (error) {
    console.error(error);
  }
}
```

Rules:
1. An `async` function **always returns a Promise**. A returned value is wrapped; a thrown error becomes a rejection.
2. `await` pauses **only the surrounding async function** until the promise settles. It does **not** block the main thread; other requests keep being served.
3. Handle rejections with `try/catch` (or `.catch()` on the returned promise).
4. Top-level `await` works only in ES Modules.

Interview-ready answer: *"async/await is syntactic sugar over Promises. An async function always returns a Promise, and await suspends that function, not the thread, until the promise settles. I use try/catch for errors and Promise.all for independent operations so they run concurrently."*

**Q: What is the common sequential-await bug? (Which is faster?)**
A: Awaiting independent operations one by one, which takes time A + B, instead of starting them together with `Promise.all`, which takes max(A, B).

```js
// ❌ Sequential: getOrders doesn't start until getUser finishes
const user = await getUser();
const orders = await getOrders();

// ✅ Concurrent: both start immediately
const [user, orders, notifications] = await Promise.all([getUser(), getOrders(), getNotifications()]);
```

Use sequential `await` only when one call needs the previous result (for example `getOrders(user.id)`). Note that `Promise.all` rejects as soon as one input rejects, but it does **not cancel** the others; they keep running. Use an `AbortSignal` if they must stop.

**Q: `all` vs `allSettled` vs `race` vs `any`?**
A: `all` fails fast on any rejection; `allSettled` waits for all and never rejects; `race` settles with the first to settle; `any` resolves with the first success and rejects with `AggregateError` if all fail.

**Q: Why is unbounded parallelism a bug?**
A: `Promise.all` over 50,000 files opens them all at once → `EMFILE`. Use a concurrency limit (N workers pulling from a queue).

**Q: How do you cancel async work?**
A: Pass an `AbortController`'s `signal` to APIs that accept it and call `abort()`. `AbortSignal.timeout(ms)` aborts automatically; `AbortSignal.any()` combines signals.

**Q: What happens to an unhandled rejection?**
A: It triggers `unhandledRejection`, and Node crashes by default since v15.

### [A2.3 — The Event Loop in Depth](A2.3-the-event-loop-in-depth.md)

**Q: What is the Event Loop?**
A: The mechanism, provided by libuv, that lets single-threaded Node do **non-blocking I/O**. It repeatedly checks for completed operations and due timers, and runs their callbacks on the main thread. Think of a waiter: they take an order, pass it to the kitchen, and serve other tables instead of standing in the kitchen waiting.

How it works:
1. Synchronous code runs on the call stack until it's empty.
2. Async operations are started and handed to the OS or the libuv thread pool.
3. When they complete, their callbacks or promise continuations are queued.
4. The event loop runs those callbacks one at a time, phase by phase (see the next question).

```js
console.log('A');
setTimeout(() => console.log('B'), 1000);
console.log('C');
// A → C → B: the timer is registered, the script carries on, and B runs once the timer is due
```

Interview-ready answer: *"The event loop coordinates the execution of asynchronous callbacks. Instead of blocking while waiting for file reads, network calls or database queries, Node starts them, keeps executing other code, and the event loop schedules each callback when its operation completes."*

**Q: What are the event loop phases?**
A: Timers → pending callbacks → idle/prepare → poll (I/O) → check (`setImmediate`) → close callbacks. Between every callback, the microtask queues are drained (see below).

| Phase | What runs there |
|---|---|
| 1. **Timers** | `setTimeout`/`setInterval` callbacks whose threshold has passed |
| 2. Pending callbacks | Some deferred system I/O callbacks (for example certain TCP errors) |
| 3. Idle / prepare | Internal use only |
| 4. **Poll** | Gets new I/O events and runs their callbacks (sockets, file reads). It waits here when nothing else is scheduled |
| 5. **Check** | `setImmediate` callbacks |
| 6. Close callbacks | `'close'` handlers such as `socket.on('close')` |

The three to know well are **Timers, Poll and Check**. A timer's delay is a *minimum*, not an exact time: the callback runs at the first timers phase after the delay has passed.

Interview-ready answer: *"The loop has six phases: timers, pending callbacks, idle/prepare, poll, check and close. Timers run due `setTimeout` callbacks, poll processes I/O, and check runs `setImmediate`. Knowing this helps me reason about execution order and avoid blocking the loop."*

**Q: What is the difference between `process.nextTick()`, `setImmediate()` and `setTimeout()`?**
A:

| Method | When it runs |
|---|---|
| `process.nextTick(fn)` | Right after the **current operation**, before promise microtasks and before the loop continues. It is not part of any phase |
| `Promise.resolve().then(fn)` | Microtask queue, right after the nextTick queue |
| `setImmediate(fn)` | The **check** phase of the current or next loop iteration |
| `setTimeout(fn, 0)` | A later **timers** phase, once at least ~1 ms has passed |

```js
console.log('Start');
setTimeout(() => console.log('Timeout'), 0);
setImmediate(() => console.log('Immediate'));
Promise.resolve().then(() => console.log('Promise'));
process.nextTick(() => console.log('Next Tick'));
console.log('End');

// Guaranteed: Start → End → Next Tick → Promise
// Then Timeout and Immediate in either order (from the main module; see "setTimeout(0) vs setImmediate" below)
```

- **`nextTick`:** use it to defer something until the current function finishes but before any I/O, for example emitting an event after the constructor returns. Recursive `nextTick` **starves** the loop: I/O never gets a turn.
- **`setImmediate`:** use it to yield to I/O, for example splitting a long job into chunks.
- **`setTimeout`:** use it for real delays. Its timing is not exact.

Interview-ready answer: *"`process.nextTick` runs right after the current operation, before promises and before the event loop continues. `setImmediate` runs in the check phase, and `setTimeout` in the timers phase once its delay has passed. Recursive nextTick can starve the event loop, so I prefer setImmediate when I want to yield."*

**Q: When do microtasks run?**
A: After every callback and after the main script: first the `process.nextTick` queue, then promise microtasks, before the loop moves on.

**Q: `setTimeout(0)` vs `setImmediate`?**
A: From the main module their order is non-deterministic; inside an I/O callback `setImmediate` always runs first because check follows poll.

**Q: Starvation vs blocking?**
A: Recursive `nextTick` or endless microtasks starve the loop (I/O never runs); synchronous CPU work blocks it for its duration.

**Q: How do you keep the loop responsive during long work?**
A: Yield between chunks with `setImmediate`, or move the work to a worker thread.

**Q: How do you measure loop health?**
A: `perf_hooks.monitorEventLoopDelay()`; healthy p99 delay is a few milliseconds.

**Q: When does a Node process exit?**
A: When nothing remains that could produce a callback — no timers, sockets, servers or pending operations.

**Q: Event loop output questions: how do you solve "predict the output" problems?**
A: Work through the queues in Node's order. Interviewers love these questions, and every one of the 10 below follows the same process:
1. Run all **synchronous** code first (the call stack).
2. Drain the **`process.nextTick` queue**.
3. Drain the **microtask queue** (Promise `.then`, `queueMicrotask`, `await` continuations) in **FIFO** order. Microtasks added while draining go to the **back** of the queue and still run before moving on.
4. Move to the **next event loop phase**: timers (`setTimeout`/`setInterval`), then poll (I/O callbacks), then check (`setImmediate`).
5. **After every single macrotask callback** (each timer, each immediate), repeat steps 2–3 before the next callback. This has been the behaviour since Node 11.

| Operation | Where it runs |
|---|---|
| Synchronous code | Immediately, on the call stack |
| `process.nextTick()` | The next-tick queue, before Promise microtasks |
| `Promise.then()`, `queueMicrotask()`, `await` continuation | The microtask queue (FIFO) |
| `setTimeout(fn, 0)` / `setInterval` | The timers phase (minimum ~1 ms) |
| `setImmediate()` | The check phase, right after poll |
| `fs.readFile()` callback | The poll (I/O) phase |

Caveats:
- These answers assume **CommonJS**. In an **ES module**, top-level code itself runs inside a promise job, so a top-level `Promise.then` can run **before** `process.nextTick`.
- `setTimeout(0)` vs `setImmediate` is **non-deterministic from the main module**, but deterministic inside an I/O callback (see Q5).

Interview-ready answer: *"I simulate the queues. First all synchronous code, then the nextTick queue, then the promise microtask queue in FIFO order, including anything newly queued, then the event loop phases: timers, I/O poll, then setImmediate. Node drains nextTicks and microtasks again after every individual timer or immediate callback, so I re-check the queues after each one."*

**Q (Output Q1): Synchronous code vs `setTimeout` vs Promise. What does this print?**

```js
console.log('A');
setTimeout(() => console.log('B'), 0);
Promise.resolve().then(() => console.log('C'));
console.log('D');
```

A. A → B → C → D  B. **A → D → C → B**  C. A → C → D → B  D. A → D → B → C

A: **B: `A, D, C, B`.** `A` and `D` are synchronous. The Promise callback goes to the microtask queue, which drains as soon as the stack is empty, so `C` comes next. The timer callback waits for the timers phase, so `B` is last.

Interview-ready answer: *"A and D first because they're synchronous, then C because promise callbacks are microtasks that run as soon as the call stack empties, and B last because setTimeout callbacks wait for the timers phase of the next loop iteration, even with a zero delay."*

**Q (Output Q2): `process.nextTick` vs Promise vs `setTimeout`. What does this print?**

```js
console.log('Start');
setTimeout(() => console.log('Timeout'), 0);
Promise.resolve().then(() => console.log('Promise'));
process.nextTick(() => console.log('NextTick'));
console.log('End');
```

A. Start → End → Promise → NextTick → Timeout  B. Start → NextTick → End → Promise → Timeout  C. **Start → End → NextTick → Promise → Timeout**  D. Start → End → Timeout → NextTick → Promise

A: **C.** After the synchronous `Start` and `End`, Node drains the **next-tick queue before the Promise microtask queue**, so `NextTick` then `Promise`. `Timeout` runs in the timers phase. (In an ES module, `Promise` can come first; see the caveats above.)

Interview-ready answer: *"Start and End run synchronously. Then Node processes the nextTick queue, which has priority over promise microtasks, so NextTick comes before Promise. The timeout runs last, in the timers phase."*

**Q (Output Q3): Nested Promises. What does this print?**

```js
console.log('A');
Promise.resolve().then(() => {
  console.log('B');
  Promise.resolve().then(() => console.log('C'));
});
Promise.resolve().then(() => console.log('D'));
console.log('E');
```

A. A → E → B → C → D  B. A → B → D → C → E  C. **A → E → B → D → C**  D. A → E → D → C → B

A: **C.** The microtask queue starts as `[B-callback, D-callback]`. Running the `B` callback **appends** the `C` callback to the back: `[D, C]`. Microtasks are **FIFO**, so `D` runs before `C`.

Interview-ready answer: *"A and E are synchronous. The queue then holds the B and D callbacks. Running B queues C, but C goes behind D, which was already waiting, so the order is B, D, C. Microtasks are first in, first out."*

**Q (Output Q4): async/await. What does this print?**

```js
async function test() {
  console.log('B');
  await Promise.resolve();
  console.log('C');
}
console.log('A');
test();
console.log('D');
```

A. A → B → C → D  B. A → D → B → C  C. **A → B → D → C**  D. B → A → D → C

A: **C.** An async function runs **synchronously until its first `await`**, so `B` prints right after `A`. `await` suspends **only that function** and schedules the rest (`C`) as a microtask. Control returns to the caller, which prints `D`, then the continuation prints `C`.

Interview-ready answer: *"Calling an async function runs its body synchronously up to the first await, so B prints immediately. await pauses just that function and queues its continuation as a microtask. The caller continues with D, and C runs once the stack is empty."*

**Q (Output Q5): `setImmediate` vs `setTimeout` inside an I/O callback. What does this print?**

```js
const fs = require('node:fs');
fs.readFile(__filename, () => {
  setTimeout(() => console.log('Timeout'), 0);
  setImmediate(() => console.log('Immediate'));
});
```

A. Timeout → Immediate  B. **Immediate → Timeout**  C. Both run together  D. Unpredictable

A: **B, always.** The `readFile` callback runs in the **poll** phase. The next phase is **check**, where `setImmediate` runs. Timers only run when the loop wraps around to the timers phase. (From the **main module**, the order of the same two calls is **non-deterministic**, because it depends on whether 1 ms has passed when the loop starts.)

Interview-ready answer: *"Inside an I/O callback we're in the poll phase, and check comes next, so setImmediate always runs before a zero-delay timeout, which has to wait for the next timers phase. From the main module the order isn't guaranteed, which is a classic follow-up question."*

**Q (Output Q6): `queueMicrotask` vs Promise. What does this print?**

```js
console.log('Start');
queueMicrotask(() => console.log('Microtask 1'));
Promise.resolve().then(() => console.log('Promise 1'));
queueMicrotask(() => console.log('Microtask 2'));
console.log('End');
```

A. Start → End → Promise 1 → Microtask 1 → Microtask 2  B. **Start → End → Microtask 1 → Promise 1 → Microtask 2**  C. Start → Microtask 1 → Microtask 2 → Promise 1 → End  D. Start → End → Microtask 1 → Microtask 2 → Promise 1

A: **B.** `queueMicrotask` and `Promise.then` use the **same** microtask queue, so the callbacks simply run in the order they were scheduled.

Interview-ready answer: *"queueMicrotask and promise callbacks share one FIFO microtask queue, with no priority between them, so after the synchronous Start and End they run in scheduling order: Microtask 1, Promise 1, Microtask 2."*

**Q (Output Q7): Multiple `await`s. What does this print?**

```js
async function first() {
  console.log('1');
  await Promise.resolve();
  console.log('2');
  await Promise.resolve();
  console.log('3');
}
console.log('A');
first();
console.log('B');
Promise.resolve().then(() => console.log('C'));
```

A. A → 1 → 2 → 3 → B → C  B. A → B → 1 → C → 2 → 3  C. **A → 1 → B → 2 → C → 3**  D. A → 1 → B → C → 2 → 3

A: **C.** Trace the microtask queue:
1. Synchronously, `A` and `1` print, the first `await` queues continuation **k1**, `B` prints, and `C` is queued. The queue is `[k1, C]`.
2. **k1** prints `2`, and the second `await` queues **k2**. The queue is `[C, k2]`.
3. `C` prints, then **k2** prints `3`.

Every `await` creates a new continuation, even for an already-resolved promise.

Interview-ready answer: *"Each await queues the rest of the function as a new microtask. After A, 1 and B, the queue holds the first continuation and C. The continuation prints 2 and queues another continuation behind C, so C prints before 3. The result is A, 1, B, 2, C, 3."*

**Q (Output Q8): `process.nextTick` inside a Promise callback. What does this print?**

```js
console.log('Start');
Promise.resolve().then(() => {
  console.log('Promise 1');
  process.nextTick(() => console.log('NextTick'));
});
Promise.resolve().then(() => console.log('Promise 2'));
console.log('End');
```

A. Start → End → Promise 1 → NextTick → Promise 2  B. **Start → End → Promise 1 → Promise 2 → NextTick**  C. Start → Promise 1 → Promise 2 → NextTick → End  D. Start → End → NextTick → Promise 1 → Promise 2

A: **B.** Once Node starts draining the **Promise microtask queue**, V8 runs it **to completion**. A `nextTick` scheduled inside a microtask is only picked up **after** that drain, so `Promise 2` (already queued) runs first. "nextTick beats Promise" applies only when **both are waiting** at the same checkpoint.

Interview-ready answer: *"nextTick has priority only when both queues are waiting at the same time. Here, by the time the nextTick is scheduled, Node is already draining the promise queue, and that drain runs to completion, so Promise 2 runs. Node then checks the nextTick queue and prints NextTick."*

**Q (Output Q9): A timer that schedules a Promise and a nextTick. What does this print?**

```js
console.log('Start');
setTimeout(() => {
  console.log('Timer');
  Promise.resolve().then(() => console.log('Promise'));
  process.nextTick(() => console.log('NextTick'));
}, 0);
setTimeout(() => console.log('Timer 2'), 0);
console.log('End');
```

A. Start → End → Timer → Timer 2 → Promise → NextTick  B. **Start → End → Timer → NextTick → Promise → Timer 2**  C. Start → End → NextTick → Timer → Promise → Timer 2  D. Start → End → Timer 2 → Timer → NextTick → Promise

A: **B.** Since **Node 11**, Node drains the nextTick and microtask queues **after each individual timer callback**, not after the whole batch of timers. That matches browsers. So the first timer's `NextTick` and `Promise` run before `Timer 2`. In Node 10 and earlier the answer was A.

Interview-ready answer: *"Since Node 11, after each timer callback Node drains nextTicks, then promise microtasks, before running the next timer. So Timer is followed by NextTick and Promise, and only then Timer 2. Older Node versions ran all expired timers first, which is a nice version detail to mention."*

**Q (Output Q10): Nested timers, Promises and nextTicks. What does this print?**

```js
console.log('A');
setTimeout(() => {
  console.log('B');
  Promise.resolve().then(() => console.log('C'));
  process.nextTick(() => console.log('D'));
}, 0);
Promise.resolve().then(() => {
  console.log('E');
  setTimeout(() => console.log('F'), 0);
  process.nextTick(() => console.log('G'));
});
console.log('H');
```

A. **A → H → E → G → B → D → C → F**  B. A → H → E → B → D → C → G → F  C. A → E → H → G → B → C → D → F  D. A → H → B → E → D → C → G → F

A: **A.**
1. The synchronous `A` and `H` print.
2. The microtask prints `E`, schedules timer F and queues nextTick G.
3. After the microtask drain, the nextTick queue runs `G`.
4. In the timers phase, timer **B** (scheduled first) prints `B`, then its nextTick `D`, then its promise `C`.
5. Timer **F** was scheduled later, so it runs last.

Interview-ready answer: *"A and H synchronously. Then the promise prints E, which schedules timer F and a nextTick; that nextTick prints G once the microtask drain finishes. In the timers phase, B runs first because it was scheduled earlier, followed by its own nextTick D and promise C before the next timer, and F comes last. A, H, E, G, B, D, C, F."*

### [A2.4 — EventEmitter](A2.4-eventemitter.md)

**Q: What is EventEmitter, and how do you use it?**
A: The built-in class (`node:events`) that implements the **observer / pub-sub pattern** inside one process. One part of the code `emit`s named events, and any number of listeners react to them. Think of a doorbell: someone presses it (emit), and whoever is listening responds. Much of Node's core is built on it: streams, `http.Server`, sockets and `process`.

```js
const EventEmitter = require('node:events');
const events = new EventEmitter();

events.on('userRegistered', (user) => console.log('Welcome', user.name));
events.emit('userRegistered', { id: 1, name: 'Pulkit' }); // Welcome Pulkit
```

| Method | Purpose |
|---|---|
| `on(name, fn)` / `addListener` | Register a listener |
| `once(name, fn)` | Listener that runs only once, then is removed |
| `emit(name, ...args)` | Trigger the event (synchronously; see below) |
| `off(name, fn)` / `removeListener` | Remove a specific listener |
| `removeAllListeners(name?)` | Remove all listeners (for one event or for all) |
| `listenerCount(name)` | Number of listeners |

Real-world use: decoupling side effects. After an order is created, the controller emits one event, and the email, analytics and audit-log modules each subscribe independently. Adding a new reaction doesn't touch the order code.

```js
events.on('orderPlaced', async (order) => sendConfirmation(order));
events.on('orderPlaced', async (order) => trackAnalytics(order));

// in the order service, after the order is saved
events.emit('orderPlaced', { id: 101, total: 2500 });
```

Interview-ready answer: *"EventEmitter is Node's built-in implementation of the event-driven, pub-sub pattern. Components emit named events and others subscribe with `on`/`once`. I use it to decouple in-process side effects such as notifications, analytics or audit logs from the core business logic."*

Senior-level insight:
- It is **in-process and in-memory**. Events aren't persisted, so they are lost on a crash or restart, and they don't reach other instances. For reliable background work across services, use a queue (BullMQ, RabbitMQ, Kafka, SQS).
- `async` listeners aren't awaited by `emit`. A rejection inside one becomes an **unhandled rejection** unless you catch it in the listener, or create the emitter with `{ captureRejections: true }`.
- An `'error'` event with no listener crashes the process (see below).

**Q: Is `emit()` asynchronous?**
A: No — it calls listeners synchronously in registration order, and a listener's exception propagates out of `emit()`.

**Q: Why must `off` receive the same function?**
A: Listeners are identified by function reference, so an anonymous arrow can't be removed.

**Q: What is special about the `error` event?**
A: If emitted with no `error` listener, Node throws it — usually crashing the process.

**Q: What does `MaxListenersExceededWarning` usually mean?**
A: Listeners are being added repeatedly (e.g. per request) and never removed — a memory leak.

**Q: How do `events.once` and `events.on` help?**
A: `await once(emitter, 'name')` resolves with the event args (rejecting on `error`, accepting a signal); `for await (... of on(emitter, 'name'))` consumes events as an async iterator.

### [P2.1 — Concurrent URL Health Checker](P2.1-concurrent-url-health-checker.md)

**Q: How is concurrency bounded?**
A: A `mapLimit` pool runs at most `--concurrency` checks at once.

**Q: What is retried, and how?**
A: Network errors, timeouts and 5xx — with exponential backoff plus jitter. 4xx responses are not retried.

**Q: How is each attempt timed out and cancelled?**
A: `AbortSignal.any([userSignal, AbortSignal.timeout(ms)])`; Ctrl+C aborts the user signal and stops retries.

**Q: Why cancel the response body?**
A: Only the status is needed; `res.body.cancel()` avoids downloading it and frees the connection.

**Q: What do the exit codes mean?**
A: 0 all healthy, 1 any unhealthy, 2 usage error.

### [P2.2 — Event-Driven Task Runner](P2.2-event-driven-task-runner.md)

**Q: How are tasks ordered?**
A: A depth-first plan resolves transitive dependencies in topological order and detects cycles and unknown tasks before anything runs.

**Q: How does scheduling work?**
A: Every time a task settles, `schedule()` starts any pending task whose dependencies are all done, up to the concurrency limit.

**Q: Why are there no race conditions on shared counters?**
A: `schedule()` runs synchronously between awaits on a single thread.

**Q: What happens when a task fails?**
A: Its dependents are skipped; unrelated tasks still finish.

**Q: Why does the runner never log?**
A: It only emits lifecycle events; a separate reporter subscribes and prints — separating logic from presentation.

---

## Phase 3 — File System & Streams

### [A3.1 — The fs Module](A3.1-the-fs-module.md)

**Q: When should you use sync fs calls?**
A: In scripts and startup code only — never inside request handlers, because they block the event loop.

**Q: What do common fs error codes mean?**
A: `ENOENT` missing, `EEXIST` already exists, `EACCES`/`EPERM` permission denied, `EISDIR`/`ENOTDIR` wrong type, `EMFILE` too many open files, `ENOTEMPTY` directory not empty.

**Q: Why is check-then-act wrong?**
A: The file can change between the check and the operation; just attempt it and handle `ENOENT`.

**Q: What does the `wx` flag do?**
A: Creates a file only if it doesn't exist — an atomic "create once", useful for lock files.

**Q: How do you write a file atomically?**
A: Write a temp file in the same directory, `sync` it, then `rename` it over the target.

**Q: Why close file handles in `finally`?**
A: Leaked descriptors eventually cause `EMFILE`.

### [A3.2 — Paths, Directories & File Watching](A3.2-paths-directories-and-file-watching.md)

**Q: `path.join` vs `path.resolve`?**
A: `join` glues and normalises segments; `resolve` always returns an absolute path from the cwd, with a later absolute segment acting as a new root.

**Q: How do you read a file next to your module?**
A: `join(import.meta.dirname, 'file')` or `new URL('./file', import.meta.url)` — not a bare relative path, which resolves from the cwd.

**Q: How do you walk a directory tree?**
A: A recursive async generator over `readdir(dir, { withFileTypes: true })`, or `readdir(dir, { recursive: true })`, or `fs.glob`.

**Q: What are `fs.watch`'s quirks?**
A: One save can fire several events, event types are only `rename`/`change`, and `filename` can be `null`. Debounce events and re-check the file with `stat`.

### [A3.3 — Streams](A3.3-streams.md)

**Q: What are streams, and why use them?**
A: Streams process data **incrementally, in chunks** (64 KB by default for files), instead of loading everything into memory. To send a 5 GB video, you either load all 5 GB and then send it, or read and send it a chunk at a time. Streams do the second. Benefits:
- Memory stays flat regardless of data size.
- Processing starts before all the data has arrived.
- Stages can be composed into pipelines (read → gunzip → parse → write).
- Real-time transfer is possible.

Interview-ready answer: *"Streams let Node process data in chunks instead of loading it all into memory. There are four types: Readable, Writable, Duplex and Transform. I use them for large uploads and downloads, file transfers, log processing and media, anywhere incremental processing keeps memory low. With `pipeline`, backpressure and errors are handled for me."*

**Q: What are the four stream types?**
A: Readable (file read, `req`), Writable (file write, `res`), Duplex (TCP socket), Transform (gzip, hash).

| Type | What it does | Examples |
|---|---|---|
| **Readable** | A source you read from | `fs.createReadStream`, HTTP `req` (server side), `process.stdin` |
| **Writable** | A destination you write to | `fs.createWriteStream`, HTTP `res`, `process.stdout` |
| **Duplex** | Readable **and** writable, independently | TCP `net.Socket`, WebSocket |
| **Transform** | A Duplex whose output is computed from its input | `zlib.createGzip()`, `crypto.createCipheriv`, CSV parsers |

```js
// Readable
const readable = fs.createReadStream('large-video.mp4');
readable.on('data', (chunk) => console.log('Received chunk:', chunk.length));
readable.on('end', () => console.log('Reading completed'));
readable.on('error', (err) => console.error(err));

// Writable
const writable = fs.createWriteStream('output.txt');
writable.write('Hello ');
writable.write('Node.js');
writable.end();                        // flush and close; emits 'finish'

// Composing them: copy and gzip a file. pipeline handles errors and backpressure
const { pipeline } = require('node:stream/promises');
const zlib = require('node:zlib');
await pipeline(
  fs.createReadStream('input.log'),
  zlib.createGzip(),                   // Transform
  fs.createWriteStream('input.log.gz')
);
```

**Q: What is the difference between `fs.readFile()` and `fs.createReadStream()`?**
A: `readFile` reads the **whole file into memory**, then gives you the result. `createReadStream` gives it to you **in chunks as it reads**.

| | `readFile()` | `createReadStream()` |
|---|---|---|
| Reads | The entire file | Chunk by chunk |
| Memory | The whole file size (a 2 GB file means about 2 GB of RAM) | About the buffer size (64 KB) |
| Processing starts | After the full read | During the read |
| Best for | Small files: config, templates, JSON | Large files, downloads, uploads, log processing |
| Limit | Fails above about 2 GB (the maximum buffer size) | None |

```js
// readFile: fine for small files
const config = JSON.parse(await fs.promises.readFile('config.json', 'utf8'));

// Streaming a 2 GB video to the client with constant memory
const { pipeline } = require('node:stream/promises');

app.get('/video', async (req, res, next) => {
  try {
    res.setHeader('Content-Type', 'video/mp4');
    await pipeline(fs.createReadStream('video.mp4'), res);
  } catch (err) {
    if (!res.headersSent) return next(err); // e.g. ENOENT, before anything was sent
    res.destroy(err);                        // mid-stream failure: abort the connection
  }
});
```

Interview-ready answer: *"`readFile` loads the complete file and returns its contents; `createReadStream` reads it incrementally. I use readFile for small files like configuration, and streams for large files, uploads, downloads and data processing, because memory stays constant and processing starts immediately."* (A real video endpoint should also support `Range` requests; see P4.2.)

**Q: What is backpressure?**
A: When a producer outpaces a consumer, `write()` returns `false` once the buffer exceeds `highWaterMark`; the producer must wait for `'drain'`.

Analogy: a tap filling a bottle faster than the bottle's narrow neck can take the water, so it overflows. With a fast disk read streaming to a slow network client, ignoring backpressure means every unsent chunk piles up **in memory**. A few slow clients downloading large files can then run the server out of memory.

```js
// Handling it by hand: pause the reader when the writer's buffer is full
readable.on('data', (chunk) => {
  if (!writable.write(chunk)) readable.pause(); // false: buffer above highWaterMark
});
writable.on('drain', () => readable.resume());  // buffer flushed, so continue
readable.on('end', () => writable.end());

// What to actually use: handles backpressure, errors and cleanup
const { pipeline } = require('node:stream/promises');
await pipeline(readable, writable);
```

`.pipe()` also handles backpressure, but **not errors**: if one stream fails, the others aren't destroyed, which leaks file descriptors (see the next question).

Interview-ready answer: *"Backpressure is what happens when a readable produces data faster than a writable can consume it. Unhandled, the data buffers in memory. Streams signal it through `write()` returning false and the `'drain'` event. In practice I use `stream.pipeline()`, which manages backpressure and also propagates errors and cleans up every stream."*

**Q: Why use `pipeline` instead of `.pipe()`?**
A: `pipeline` propagates errors, destroys every stream on failure, handles backpressure, returns a promise, and accepts async generators as stages.

**Q: How do you consume a stream simply?**
A: `for await (const chunk of readable)`, which respects backpressure.

**Q: Why must line splitting keep a leftover?**
A: A chunk can end mid-line; the partial line is carried into the next chunk.

**Q (Scenario): Users upload 500 MB–2 GB videos. How do you handle that without exhausting server memory?**
A: **Never buffer the whole file in Node.** Ideally, don't proxy it through Node at all.

```
Client ──(1) ask for upload URLs──▶ Node API (auth, quota, ownership, create video row = UPLOADING)
Client ──(2) PUT parts directly───▶ S3 / R2 (multipart, presigned URLs)
Client ──(3) "complete"───────────▶ Node API ─▶ status = UPLOADED ─▶ enqueue BullMQ job
                                    Worker + FFmpeg ─▶ compress, thumbnails, HLS ─▶ status = COMPLETED / FAILED
```

1. **Direct-to-cloud uploads with presigned URLs.** The backend only authenticates, authorises and signs URLs. The bytes never touch your API servers, so they don't use your memory, bandwidth or event loop.
   ```js
   const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');
   const { getSignedUrl } = require('@aws-sdk/s3-request-presigner');

   const key = `uploads/${req.user.id}/${crypto.randomUUID()}.mp4`;   // never the user's file name
   const url = await getSignedUrl(s3, new PutObjectCommand({ Bucket, Key: key, ContentType: 'video/mp4' }), { expiresIn: 900 });
   ```
2. **Multipart upload** for large files (S3 `CreateMultipartUpload`, a presigned URL per part, then `CompleteMultipartUpload`). A 500 MB file becomes, for example, 50 × 10 MB parts. That gives **retries per part**, **resumable** uploads, and **parallel** part uploads. Libraries like Uppy or tus handle the client side.
3. **If it must go through Node**, stream it. Use `busboy` or `multer.diskStorage`, or `pipeline(req, s3UploadStream)`, with a strict `fileSize` limit. **Never** use `memoryStorage` here.
4. **Validate:**
   - Size limit (in the presigned policy and checked after upload).
   - **Real file type**, from the magic bytes or `ffprobe`, not the extension.
   - Duration and resolution from metadata.
   - The user's permissions and **storage quota**.
5. **Process asynchronously.** On completion (the client calls `/complete`, or an S3 event fires), enqueue a job: `videoQueue.add('process-video', { videoId, storageKey })`. A **separate worker fleet** runs FFmpeg to compress, generate thumbnails, produce several resolutions and package **HLS**.
6. **Track status** (`UPLOADING → UPLOADED → PROCESSING → COMPLETED | FAILED`). The client polls `GET /videos/:id`, or gets a WebSocket or push update.
7. **Clean up:**
   - An S3 lifecycle rule to **abort incomplete multipart uploads** and delete orphaned files.
   - A job that marks stuck `UPLOADING`/`PROCESSING` rows as failed.
   - Serve videos through a CDN.

Interview-ready answer: *"I don't load large videos into Node's memory. The backend authenticates the user, checks permissions and quota, and returns presigned multipart upload URLs, so the client uploads directly to S3 or R2 with retryable, resumable parts. On completion I validate the file and enqueue a BullMQ job. Separate workers run FFmpeg for compression, thumbnails and HLS, and update a status that the client can poll or receive over WebSockets. Lifecycle rules and cleanup jobs remove abandoned uploads."*

Senior-level insight: keep the **upload path** (I/O-bound, cheap) and the **transcoding path** (CPU-heavy, slow) as separate systems. Then they **scale independently**: you can add transcoding workers or use spot instances or a managed service (MediaConvert) without touching the API, and a transcoding backlog never slows down uploads.

### [A3.4 — Buffers & Binary Data](A3.4-buffers-and-binary-data.md)

**Q: What is a Buffer?**
A: A fixed-length byte sequence, a subclass of `Uint8Array`, stored outside the V8 heap. It's how Node handles **raw binary data**: files, images, video, PDFs, network packets, and the chunks that streams emit.

```js
const buffer = Buffer.from('Hello Node.js');
console.log(buffer);            // <Buffer 48 65 6c 6c 6f 20 4e 6f 64 65 2e 6a 73>  (UTF-8 bytes)
console.log(buffer.toString()); // 'Hello Node.js'
console.log(buffer.length);     // 13 (bytes, not characters)
```

Interview-ready answer: *"A Buffer is Node's built-in class for raw binary data. It's used for files, images, network I/O and stream chunks. For example, an uploaded image arrives as a Buffer that we can validate and then send to disk or S3."*

**Q: How do you create a Buffer? (`from` vs `alloc` vs `allocUnsafe`)**
A:

| Method | What it does |
|---|---|
| `Buffer.from(data)` | Creates a buffer from a string, array or another buffer (copies the data) |
| `Buffer.alloc(n)` | Allocates `n` **zero-filled** bytes. Safe |
| `Buffer.allocUnsafe(n)` | Allocates `n` bytes **without zero-filling**. Faster, but may contain old memory |

Only use `allocUnsafe` if you overwrite every byte before reading or sending it, otherwise you can leak another request's data (secrets, tokens).

Senior-level insight: in uploads with Multer's `memoryStorage`, `req.file.buffer` holds the **entire file in RAM**. That is fine for small images with a size limit, but a few concurrent 1 GB uploads will run the server out of memory. For large files, **stream** directly to disk or S3 (`multer.diskStorage`, Busboy, presigned S3 uploads) and always set `limits.fileSize`.

**Q: Why do string length and byte length differ?**
A: UTF-8 characters can take 1–4 bytes; `'é'` is 2 bytes and an emoji 4.

**Q: How do you avoid broken characters across chunks?**
A: Decode with `StringDecoder` or `TextDecoder({ stream: true })`, or set an encoding on the stream.

**Q: Does `subarray` copy?**
A: No — it's a view over the same memory; use `Buffer.from(view)` for a copy.

**Q: Big-endian vs little-endian?**
A: Big-endian stores the most significant byte first (network order, PNG); little-endian the least significant first (x86, most file formats).

### [P3.1 — Log File Analyzer](P3.1-log-file-analyzer.md)

**Q: How does it keep memory flat on huge logs?**
A: It streams lines with `readline`, aggregates counts in maps keyed by distinct values, and computes percentiles from a fixed-size latency histogram.

**Q: How are `.gz` files and stdin handled?**
A: All sources become one line stream; gzip goes through `createGunzip`, with file errors forwarded into it.

**Q: Why normalise paths?**
A: `/api/users/42` → `/api/users/:id` keeps top-N meaningful and maps small.

**Q: What happens to malformed lines?**
A: They're counted and skipped, never crashing the tool.

### [P3.2 — File Backup Tool](P3.2-file-backup-tool.md)

**Q: How does deduplication work?**
A: Content-addressed storage: each file's SHA-256 names its object, so unchanged or duplicate files are stored once.

**Q: What is in a snapshot manifest?**
A: Each file's relative path, size, mode, mtime and hash.

**Q: How are writes kept atomic?**
A: Objects and manifests are written to temp files and renamed into place.

**Q: How is restore kept safe?**
A: It refuses manifest paths that resolve outside the target directory, and restores permissions and mtimes.

**Q: How does `verify` detect corruption?**
A: It decompresses and re-hashes every object and compares with its name.

---

## Phase 4 — Networking & HTTP

### [A4.1 — HTTP Servers with node:http](A4.1-http-servers-with-node-http.md)

**Q: What are `req` and `res`?**
A: `req` is an `IncomingMessage` (a Readable stream with `method`, `url`, `headers`); `res` is a `ServerResponse` (a Writable stream) on which you set status and headers, write the body, and call `end()` exactly once.

**Q: What causes `ERR_HTTP_HEADERS_SENT`?**
A: Setting headers after the first `write()`/`end()`.

**Q: How should you parse `req.url`?**
A: `new URL(req.url, 'http://localhost')`, then use `pathname` and `searchParams`.

**Q: Why must you limit body size yourself?**
A: Node doesn't buffer bodies; without a limit (enforced while reading, since `content-length` can lie) one client can exhaust memory.

**Q: Which server timeouts exist?**
A: `requestTimeout` (whole request), `headersTimeout` (headers), `keepAliveTimeout` (idle keep-alive connections).

**Q: How do you clean up when a client disconnects?**
A: Listen for `res.on('close')` and clear timers or streams.

### [A4.2 — Routing & Request Parsing from Scratch](A4.2-routing-and-request-parsing-from-scratch.md)

**Q: How are dynamic routes matched?**
A: Each pattern is compiled once into a regex with named groups; params are extracted and URL-decoded.

**Q: 404 vs 405?**
A: 404 if no route matches the path; 405 with an `Allow` header if the path matches but not the method.

**Q: What is onion-style middleware?**
A: Each middleware gets `(ctx, next)`; code before `await next()` runs on the way in and code after it on the way out — ideal for timing and headers.

**Q: What is middleware in Express.js, and how does it work?**
A: A function `(req, res, next)` that runs during the request-response cycle, **in the order it was registered**. Think of checkpoints in an office building: security, then reception, then access control, then your desk. Each middleware can:
1. Read or modify `req`/`res` (for example attach `req.user`).
2. Run logic such as logging, validation or authentication.
3. **End the cycle** by sending a response (`res.status(401).json(...)`).
4. **Pass control on** with `next()`, or jump to the error handler with `next(err)`.

If it does neither 3 nor 4, the request **hangs**.

```js
function authenticate(req, res, next) {
  const header = req.headers.authorization;           // "Bearer <token>"
  if (!header?.startsWith('Bearer ')) {
    return res.status(401).json({ message: 'Unauthorized' }); // end the cycle
  }
  try {
    req.user = verifyToken(header.slice(7));          // e.g. { id: 1, role: 'admin' }
    next();                                           // continue
  } catch {
    res.status(401).json({ message: 'Invalid token' });
  }
}

app.use(express.json());                              // global: parse JSON bodies
app.get('/profile', authenticate, (req, res) => {     // route-level
  res.json({ message: 'Profile fetched', user: req.user });
});
```

| Type | How it's applied | Examples |
|---|---|---|
| Application-level | `app.use(fn)` / `app.get(path, fn, handler)` | Logging, auth |
| Router-level | `router.use(fn)` on an `express.Router()` | Auth for `/admin/*` only |
| Built-in | Shipped with Express | `express.json()`, `express.urlencoded()`, `express.static()` |
| Third-party | npm packages | `cors`, `helmet` (security headers), `morgan` (logging), `express-rate-limit`, `compression` |
| Error-handling | **Four** arguments `(err, req, res, next)`, registered last | Central error handler (see A7.4) |

Interview-ready answer: *"Express middleware is a function with access to `req`, `res` and `next` that runs in order during the request-response cycle. It can modify the request, run logic, end the response, or call `next()` to pass control on. I use it for authentication and authorisation, validation, logging, rate limiting, CORS, security headers and centralised error handling."*

Senior-level insight: **order matters.**
- Put `helmet`/`cors` and body parsers before the routes that need them.
- Put auth before protected routers.
- Put the 404 handler after all routes, and the error handler last.

Express 4 does **not** catch rejected promises from `async` handlers; you must `try/catch` and call `next(err)`, or use a wrapper. **Express 5** forwards async errors to the error handler automatically.

**Q: How should errors be handled centrally?**
A: Handlers throw; one place maps `HttpError`s to their status and safe message, and anything unexpected to a logged, generic 500.

**Q: What should happen with malformed percent-encoding?**
A: `decodeURIComponent` throws `URIError`, which should become a 400.

### [A4.3 — HTTP Clients & fetch](A4.3-http-clients-and-fetch.md)

**Q: Does `fetch` reject on a 500?**
A: No — it rejects only on network failure; check `res.ok`/`res.status` yourself.

**Q: Can a body be read twice?**
A: No — `json()`/`text()` consume it; use `res.clone()` first if needed.

**Q: How do you stream a download to disk?**
A: `pipeline(Readable.fromWeb(res.body), createWriteStream(path))`.

**Q: Does fetch have a default timeout?**
A: No — always pass `signal: AbortSignal.timeout(ms)`.

**Q: What should be retried?**
A: Network errors, timeouts, 502/503/504 and 429 (honouring `Retry-After`), with backoff — and only for idempotent methods.

**Q: What does a keep-alive agent do?**
A: Reuses TCP/TLS connections across requests, avoiding a handshake per request.

### [A4.4 — TCP & UDP with net and dgram](A4.4-tcp-and-udp-with-net-and-dgram.md)

**Q: Why does every TCP protocol need framing?**
A: TCP is a byte stream with no message boundaries — writes can merge or split.

**Q: What are the two framing approaches?**
A: Delimiters (e.g. newline — SMTP, Redis inline) and length prefixes (e.g. a 4-byte length — many binary protocols).

**Q: `'end'` vs `'close'` on a socket?**
A: `'end'` means the peer finished sending (half-close); `'close'` means the socket is fully closed.

**Q: How does UDP differ from TCP?**
A: No connections or streams: whole datagrams, with no delivery or ordering guarantees — used for DNS, metrics and games.

**Q: How do WebSockets work in Node.js, and when would you use them?**
A: HTTP is request-response: the server can only answer. With polling, a chat client keeps asking "any new messages?" A **WebSocket** is a **persistent, full-duplex** connection over a single TCP socket, so **either side can send at any time**.

How it works:
1. The client sends a normal HTTP request with `Upgrade: websocket`.
2. The server replies `101 Switching Protocols`.
3. The same TCP connection now carries lightweight WebSocket **frames** in both directions.
4. It stays open until one side closes it. Ping/pong heartbeats detect dead connections.

```js
// Server: Socket.IO (adds rooms, reconnection, acknowledgements and a polling fallback on top of WebSockets)
const { Server } = require('socket.io');
const io = new Server(httpServer, { cors: { origin: process.env.CLIENT_URL } });

io.use(async (socket, next) => {                // authenticate the handshake
  try {
    socket.user = await verifyToken(socket.handshake.auth.token);
    next();
  } catch {
    next(new Error('Unauthorized'));
  }
});

io.on('connection', (socket) => {
  socket.join(`user:${socket.user.id}`);        // a room per user, for targeted pushes
  socket.on('sendMessage', async ({ roomId, text }) => {
    if (!(await canPostIn(socket.user, roomId))) return;    // authorise every action
    const msg = await saveMessage(roomId, socket.user.id, text);  // persist first
    io.to(roomId).emit('newMessage', msg);
  });
  socket.on('disconnect', () => { /* presence cleanup */ });
});

// Client
const socket = io('https://api.example.com', { auth: { token } });
socket.emit('sendMessage', { roomId, text: 'Hello' });
socket.on('newMessage', (m) => console.log(m));
```

Use them for chat, live notifications, live-stream interactions (comments, gifts, PK battles), multiplayer games, collaborative editing, live dashboards and auctions.

| | HTTP polling | Server-Sent Events | WebSocket |
|---|---|---|---|
| Direction | Client asks repeatedly | Server → client only | **Both ways** |
| Overhead | A full HTTP request each time | One long HTTP response | Small frames after the handshake |
| Best for | Infrequent updates | Notifications, feeds, AI token streams | Chat, games, anything interactive |

Interview-ready answer: *"WebSockets upgrade an HTTP connection into a persistent, two-way channel, so the server can push messages without the client polling. I use them, usually through Socket.IO, for chat, live notifications, live-stream interactions and dashboards. To scale across instances I add the Socket.IO Redis adapter, so an event emitted on one server reaches clients connected to the others."*

Senior-level insight:
- **Authenticate the handshake and authorise every message.**
- **Heartbeats** and client reconnection with backoff.
- With multiple instances, use the **Redis adapter** (`@socket.io/redis-adapter`), plus **sticky sessions** at the load balancer if the HTTP polling fallback is enabled.
- Each connection holds memory, so plan file descriptor limits and connections per instance.
- The Redis adapter is **pub/sub, not durable**. A client that is offline misses events, so **persist messages** in the database and let clients fetch what they missed on reconnect.
- Rate-limit incoming events, and cap the message size.

### [P4.1 — Framework-Free REST API](P4.1-framework-free-rest-api.md)

**Q: How are concurrent writes to the JSON file kept safe?**
A: Changes are applied in memory, then saves are chained on a promise queue so writes never interleave; each uses an atomic temp-file rename, and a failed write doesn't block later ones.

**Q: How does conditional GET work?**
A: Responses carry an `ETag`; a matching `If-None-Match` returns 304.

**Q: How is CORS preflight handled?**
A: `OPTIONS` requests with `Access-Control-Request-Method` get a 204 with allowed methods and headers, skipping the router.

**Q: What does graceful shutdown do?**
A: Stops accepting connections, closes idle ones, and flushes pending writes before exit.

**Q (Scenario): A user double-clicks "Pay", or the client retries a slow request. How do you prevent duplicate orders, payments or records?**
A: Never rely only on disabling the button or an in-memory flag; retries, double taps and multiple instances get past both. Combine these:

**1. Idempotency keys.** The client generates a unique key per *logical* operation and resends the **same** key on every retry: `POST /orders` with `Idempotency-Key: 7f3c…`. The server stores the key with its outcome, and a repeat returns the **stored response** instead of running the operation again (the same approach Stripe uses).

Naive version: `findOne(key)` → create → save. This has a **race**: two simultaneous requests both see "not found" and both create an order. **Reserve the key atomically** with a unique constraint:

```js
app.post('/orders', async (req, res, next) => {
  const key = req.get('Idempotency-Key');
  if (!key) return res.status(400).json({ message: 'Idempotency-Key header required' });
  const requestHash = sha256(JSON.stringify(req.body));

  try {
    // Atomic reservation: only one request can insert this key (PRIMARY KEY / UNIQUE)
    const { rowCount } = await pool.query(
      `INSERT INTO idempotency_keys (key, user_id, request_hash, status)
       VALUES ($1, $2, $3, 'processing') ON CONFLICT (key) DO NOTHING`,
      [key, req.user.id, requestHash]
    );

    if (rowCount === 0) {                              // seen before
      const { rows: [row] } = await pool.query('SELECT * FROM idempotency_keys WHERE key = $1', [key]);
      if (row.request_hash !== requestHash) return res.status(422).json({ message: 'Key reused with a different payload' });
      if (row.status === 'completed') return res.status(row.status_code).json(row.response);
      return res.status(409).json({ message: 'Request already in progress' });
    }

    const order = await createOrder(req.user.id, req.body);   // ideally in the same transaction as the update below
    const response = { orderId: order.id, status: 'created' };
    await pool.query(
      `UPDATE idempotency_keys SET status = 'completed', status_code = 201, response = $2 WHERE key = $1`,
      [key, response]
    );
    res.status(201).json(response);
  } catch (err) {
    // Release the key so the client can retry after a failure
    await pool.query(`DELETE FROM idempotency_keys WHERE key = $1 AND status = 'processing'`, [key]);
    next(err);
  }
});
```

Production details:
- Scope keys per user.
- **Expire** them (24 h or so).
- Reject a reused key with a different payload.
- Recover keys stuck in `processing` after a crash.

**2. Database unique constraints**, the final safety net that holds even under races:

```sql
CREATE UNIQUE INDEX unique_payment_reference ON payments (payment_reference);
CREATE UNIQUE INDEX one_active_cart_per_user ON carts (user_id) WHERE status = 'active';
```

Catch the duplicate-key error (Postgres `23505`, Mongo `11000`) and return the existing record.

**3. Redis locks** (`SET lock:order:123 <token> NX PX 30000`, released only by the token's owner) can serialise a critical section across instances. But locks can expire mid-operation, so they **supplement** constraints and transactions, never replace them.

Interview-ready answer: *"I use idempotency keys plus database unique constraints. The client sends a key per logical operation. The server atomically reserves it, runs the operation once, stores the response, and returns that stored response for any retry. A plain findOne check isn't enough, because two concurrent requests can both pass it, so the reservation relies on a unique constraint. For payments and orders I never rely on frontend button-disabling or in-memory flags."*

**Q (Scenario): Two users buy the last item in stock at the same moment. How do you prevent overselling? (Transactions and race conditions)**
A: A **race condition** is when the outcome depends on the timing of concurrent operations. **Read-check-write** in application code is the classic bug:

```js
// ❌ Both requests read stock = 1, both pass the check, both decrement: oversold
const product = await Product.findById(productId);
if (product.stock > 0) {
  product.stock -= 1;
  await product.save();
}
```

**Fix 1: an atomic conditional update.** The check and the write happen in **one** database operation:

```js
// MongoDB
const product = await Product.findOneAndUpdate(
  { _id: productId, stock: { $gt: 0 } },   // condition
  { $inc: { stock: -1 } },                  // update
  { new: true }
);
if (!product) throw new AppError('Product out of stock', 409);
```

```sql
-- SQL equivalent: 0 rows updated means out of stock
UPDATE products SET stock = stock - 1 WHERE id = $1 AND stock > 0 RETURNING id;
```

**Fix 2: a transaction**, when several writes must succeed or fail **together** (decrement stock **and** create the order):

```js
const client = await pool.connect();          // one client for the whole transaction
try {
  await client.query('BEGIN');
  const { rowCount } = await client.query(
    'UPDATE products SET stock = stock - 1 WHERE id = $1 AND stock > 0', [productId]);
  if (rowCount === 0) throw new AppError('Product out of stock', 409);
  await client.query('INSERT INTO orders (product_id, user_id) VALUES ($1, $2)', [productId, userId]);
  await client.query('COMMIT');
} catch (err) {
  await client.query('ROLLBACK');
  throw err;
} finally {
  client.release();                            // always return the connection to the pool
}
```

(MongoDB: `session.withTransaction(...)` on a replica set.)

**ACID:**

| Property | Meaning |
|---|---|
| **Atomicity** | All operations commit, or none do |
| **Consistency** | Constraints and invariants still hold afterwards |
| **Isolation** | Concurrent transactions don't see each other's partial work (to a degree set by the isolation level) |
| **Durability** | Once committed, the data survives crashes |

Other tools:
- **Unique constraints.**
- **Optimistic locking:** a `version` column, `UPDATE … WHERE id = $1 AND version = $2`; 0 rows means someone else changed it, so retry.
- **Pessimistic locking:** `SELECT … FOR UPDATE` locks the row until commit.
- **Stricter isolation levels** (`REPEATABLE READ`/`SERIALIZABLE`), with retries on serialisation failures.
- **Idempotency keys** for retried requests.

Interview-ready answer: *"A race condition is when concurrent operations on shared data produce a result that depends on timing, like two buyers both seeing one item in stock. I avoid read-then-write in application code. For inventory I use a conditional atomic update (`stock > 0` with a decrement in one statement). When several writes must be consistent I wrap them in a transaction, and I add unique constraints, optimistic or pessimistic locking, or a stricter isolation level where the business rule requires it."*

Senior-level insight: a transaction **alone** doesn't stop every race. Under the default `READ COMMITTED` isolation level, two transactions can still read the same stock and both write unless the check is in the `UPDATE`'s `WHERE` clause or the row is locked. Match the isolation level and locking strategy to the invariant you're protecting.

**Q (Scenario): An order flow spans Order, Payment and Inventory services. Payment succeeds but reserving inventory fails. How do you keep data consistent?**
A: Each service owns its own database, so there is **no single ACID transaction** across them; two-phase commit is slow, fragile and rarely used. Instead, use a **Saga**: a sequence of **local transactions**, where each completed step has a **compensating action** that undoes its business effect if a later step fails.

```
Create order (PENDING) ──▶ Charge payment ──▶ Reserve inventory ✗ fails
                                   ◀── compensate: refund payment ◀── cancel order
```

| Completed step | Compensating action |
|---|---|
| Order created | Cancel the order |
| Payment captured | **Refund** (a new financial transaction, not a delete) |
| Inventory reserved | Release the reservation |

Compensation is **semantic**, not a literal rollback: the refund and the cancelled order remain in the history. Ordering also matters: reserving inventory *before* charging means fewer refunds.

Two styles:
- **Choreography:** services react to each other's events (`OrderCreated` → Payment → `PaymentCompleted` → Inventory → `InventoryReserved` → `OrderConfirmed`). It is decoupled, but the flow is hard to see and debug as it grows.
- **Orchestration:** a central **orchestrator** (an order saga service, or Temporal or AWS Step Functions) tells each service what to do and triggers compensations. The flow is explicit and easier to monitor, at the cost of a central component.

The techniques that make it reliable:
- **Transactional outbox:** write the business change **and** the event to publish in the **same local DB transaction** (`INSERT INTO orders …; INSERT INTO outbox …`). A relay, or CDC with Debezium, then publishes outbox rows to Kafka or RabbitMQ. This avoids the "saved to the DB but the publish failed", and the reverse, dual-write problem.
- **Idempotent consumers:** messages are delivered **at least once**, so deduplicate by event ID.
- **Retries with exponential backoff**, and **dead-letter queues** for poison messages.
- **Reconciliation jobs** that compare states (for example, payments captured but orders not confirmed) and fix or alert.

Interview-ready answer: *"Across services I don't try to hold one distributed ACID transaction. I use the Saga pattern: each service does a local transaction and publishes an event, and if a later step fails, compensating actions undo the earlier steps, such as refunding the payment and cancelling the order. I publish events reliably with a transactional outbox, make consumers idempotent, add retries and dead-letter queues, and run reconciliation jobs to catch anything that slips through."*

Senior-level insight: distributed systems trade immediate consistency for **eventual consistency**. Design the UX for it, with states like `PENDING`/`PROCESSING`, and make every step idempotent and observable. And question whether the split is needed: if these three "services" share a team and a database, a modular monolith with a real transaction is far simpler.

### [P4.2 — Static File Server](P4.2-static-file-server.md)

**Q: How is path traversal blocked?**
A: Decode once, reject null bytes and any segment starting with `.`, then verify the resolved path stays inside the root.

**Q: How do range requests work?**
A: `Range: bytes=start-end` returns 206 with `Content-Range`; out-of-bounds ranges return 416.

**Q: When is gzip applied?**
A: For compressible types over 1 KB when the client sends `Accept-Encoding: gzip`, with `Vary: Accept-Encoding`.

**Q: How is caching set?**
A: Fingerprinted assets get `max-age=31536000, immutable`; others get `no-cache` with ETag/Last-Modified revalidation.

## Phase 5 — Security & Cryptography

### [A5.1 — The crypto Module](A5.1-the-crypto-module.md)

**Q: Hash vs HMAC vs signature?**
A: A hash fingerprints data; an HMAC proves authenticity using a shared secret; a signature is verifiable by anyone with the public key.

**Q: Which hashes are broken?**
A: MD5 and SHA-1 — use SHA-256/512.

**Q: Why not `Math.random()` for tokens?**
A: It's predictable; use `randomBytes`, `randomUUID` or `randomInt`.

**Q: Why use `timingSafeEqual`?**
A: `===` stops at the first difference, leaking timing information; `timingSafeEqual` takes constant time (lengths must match).

**Q: Why verify webhooks on the raw body?**
A: Re-serialised JSON may differ byte for byte from what was signed.

**Q (Scenario): How do you handle payment webhooks (Razorpay or Stripe) reliably?**
A: Assume webhooks can be **forged, duplicated, delayed, out of order, or arrive while you're down**, and design for all of these.

```
Provider ─▶ POST /webhooks/payment
             1. verify signature (raw body)
             2. INSERT event (UNIQUE event_id) ─ duplicate? still answer 200
             3. enqueue job (jobId = event_id) ─▶ 200 fast
Worker ─▶ 4. process idempotently in a DB transaction: payment = PAID, order = CONFIRMED, event = processed
Cron   ─▶ 5. reconciliation: fetch the provider's payments and fix any mismatches
```

1. **Verify the signature** with the **raw body**, because re-serialised JSON won't match the HMAC, and compare with `timingSafeEqual` (see "Why verify webhooks on the raw body?" above). Stripe: `stripe.webhooks.constructEvent(rawBody, sig, secret)`. Razorpay: HMAC-SHA256 of the raw body with the webhook secret.
2. **Persist the event first** in a `webhook_events` table: `event_id` (**UNIQUE**), `type`, `payload`, `status` (received, processing, processed, failed), `attempts`, `created_at`.
3. **Respond 200 quickly** and do the work in the background. Providers time out after a few seconds and **retry** anything that isn't a 2xx.
4. **Process idempotently.** The unique `event_id` means a duplicate delivery is a no-op. Guard state transitions too (`UPDATE payments SET status = 'paid' WHERE id = $1 AND status = 'pending'`), so an out-of-order or replayed event can't move a refunded payment back to paid.
5. **Update payment and order atomically** in one transaction, together with marking the event processed.
6. **Reconcile:** a scheduled job compares the provider's records (their API) with yours and fixes or alerts on mismatches. This covers webhooks that never arrived.

```js
app.post('/webhooks/payment', express.raw({ type: 'application/json' }), async (req, res) => {
  let event;
  try {
    event = verifyWebhookSignature(req.body, req.get('x-razorpay-signature')); // throws if invalid
  } catch {
    return res.status(400).send('Invalid signature');
  }

  try {
    const { rowCount } = await pool.query(
      `INSERT INTO webhook_events (event_id, type, payload, status)
       VALUES ($1, $2, $3, 'received') ON CONFLICT (event_id) DO NOTHING`,
      [event.id, event.type, event]
    );
    if (rowCount === 1) {
      await webhookQueue.add('process-payment-event', { eventId: event.id }, { jobId: event.id, attempts: 5 });
    }
    res.status(200).json({ received: true });  // duplicates also get 200, so the provider stops retrying
  } catch (err) {
    logger.error({ err, eventId: event.id }, 'webhook persist failed');
    res.status(500).send('Retry later');        // the provider will retry
  }
});
```

Note that `express.raw` must apply to this route **before** any global `express.json()` touches the body.

Interview-ready answer: *"I verify the provider's signature against the raw request body, persist the event under its unique provider event ID, enqueue it, and return 200 quickly. A worker processes it idempotently: duplicate deliveries are ignored by the unique constraint, and state transitions are guarded. It updates payment and order status in one transaction. I also track event status, retry with backoff, and run a reconciliation job against the provider's API to catch anything missed."*

Senior-level insight: **never treat the frontend's "payment success" callback as the truth.** It can be faked or lost when the browser closes. Order confirmation should come from a verified webhook, or from a server-side check of the payment with the provider's API.

### [A5.2 — Password Hashing & Signed Tokens](A5.2-password-hashing-and-signed-tokens.md)

**Q: Why are fast hashes bad for passwords?**
A: Attackers can try billions of guesses per second, and unsalted identical passwords share hashes.

**Q: What do salts and work factors do?**
A: A per-password random salt defeats precomputed tables; a slow, memory-hard KDF like scrypt makes each guess expensive.

**Q: Why store parameters with the hash?**
A: So old hashes verify after you raise the cost, and can be rehashed on next login.

**Q: What are a JWT's parts?**
A: Base64url header, payload (claims like `sub`, `iat`, `exp` — readable by anyone) and HMAC signature.

**Q: What are classic JWT vulnerabilities?**
A: Accepting `alg: none`, algorithm confusion, missing `exp` checks, non-constant-time comparison, and weak secrets.

**Q: JWT vs OAuth 2.0 vs RBAC: what's the difference?**
A: They solve **different problems** and are often used together.

| | What it is | Answers |
|---|---|---|
| **JWT** | A **token format**: `header.payload.signature`, base64url-encoded, signed (HS256/RS256) | "Is this token genuine and unexpired, and what claims does it carry?" |
| **OAuth 2.0** | An **authorisation framework** for *delegated access*. An app gets an access token to call an API on a user's behalf (scopes, consent, authorisation code + PKCE flow) | "May this *app* access these resources on my behalf?" |
| **OpenID Connect (OIDC)** | An identity layer **on top of** OAuth 2.0 that adds an `id_token`. This is what "Sign in with Google" actually uses | "Who is the user?" (authentication) |
| **RBAC** | An **authorisation model**: permissions are attached to roles, and roles to users | "Can this user, with this role, do this action?" |

```json
{ "sub": "123", "role": "admin", "iss": "api.example.com", "aud": "web", "iat": 1794200000, "exp": 1794200900 }
```

| Role | Permissions |
|---|---|
| Admin | Create, read, update, delete |
| Manager | Create, read, update |
| User | Read and update own profile |

A typical flow: login → validate credentials → issue a **short-lived access token** (5–15 min) plus a **refresh token** (in an `HttpOnly` cookie, rotated on use, stored hashed server-side so it can be revoked) → the client sends `Authorization: Bearer <token>` → the server **verifies the signature, `exp`, `iss`, `aud` and the algorithm** → an RBAC check → allow or deny.

```ts
// NestJS: RBAC with a decorator + guard (runs after the JWT AuthGuard has set req.user)
export const Roles = (...roles: string[]) => SetMetadata('roles', roles);

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}
  canActivate(ctx: ExecutionContext) {
    const required = this.reflector.getAllAndOverride<string[]>('roles', [ctx.getHandler(), ctx.getClass()]);
    if (!required) return true;
    const { user } = ctx.switchToHttp().getRequest();
    return required.includes(user?.role);
  }
}

@UseGuards(JwtAuthGuard, RolesGuard)
@Roles('admin', 'manager')
@Get('reports')
getReports() { /* ... */ }
```

Interview-ready answer: *"JWT is a signed token format for carrying claims. OAuth 2.0 is a framework for delegated authorisation, with OIDC on top for login. RBAC is a model for deciding what a role may do. In practice I issue short-lived JWT access tokens with rotated refresh tokens in HttpOnly cookies, verify the signature, expiry, issuer, audience and algorithm on every request, and enforce RBAC with NestJS guards, plus ownership checks for user-specific resources."*

Senior-level insight:
- A signed JWT is **not encrypted**. Anyone can base64-decode the payload, so **never put passwords, PII or secrets in it**.
- JWTs can't be revoked before they expire. Keep them short-lived, and use refresh-token rotation or a denylist for logout and compromise.
- RBAC alone doesn't stop IDOR: "a user can read orders" must also mean "**their own** orders". For fine-grained rules, use ABAC or policies (CASL).

### [A5.3 — Encryption with AES-GCM](A5.3-encryption-with-aes-gcm.md)

**Q: Why authenticated encryption?**
A: Plain modes let attackers flip ciphertext bits undetected; GCM's auth tag makes any tampering fail decryption.

**Q: What are the inputs to AES-256-GCM?**
A: A 32-byte key, a unique 12-byte IV per encryption (never reused with the same key), a 16-byte auth tag, and optional AAD.

**Q: What does AAD do?**
A: Binds unencrypted context (e.g. a record ID) to the ciphertext so it can't be moved elsewhere.

**Q: How do you encrypt with a password?**
A: Derive the key with scrypt and a random salt, storing salt, IV and tag in a versioned envelope.

**Q: What is the hard part of encryption?**
A: Key management — secure storage, rotation (key IDs) and envelope encryption.

### [A5.4 — Node.js Security Best Practices](A5.4-nodejs-security-best-practices.md)

**Q: How do you prevent path traversal?**
A: Resolve the path and verify it stays inside the root; reject null bytes.

**Q: How do you prevent command injection?**
A: Use `execFile`/`spawn` with an argument array and no shell.

**Q: What is prototype pollution?**
A: Merging user JSON with `__proto__` keys modifies `Object.prototype` for every object; block those keys and use `Map`/`Object.create(null)`.

**Q: What is ReDoS?**
A: Regexes with nested quantifiers taking exponential time on crafted input, freezing the event loop; rewrite them and cap input length.

**Q: What is SSRF and how do you guard against it?**
A: Tricking the server into fetching internal URLs (e.g. cloud metadata); allow only https, block private/loopback/link-local IPs, and don't follow redirects blindly.

**Q: What does the permission model do?**
A: `node --permission` denies file system, child process and worker access except what you allow with flags like `--allow-fs-read`.

**Q: How do you secure a Node.js backend application?**
A: **Defence in depth**: several layers, so one mistake isn't fatal.

1. **Validate all input** on the server, both shape and type: body, params, query and headers. Use Zod/Joi/express-validator, and reject unknown fields.
   ```js
   const { body, validationResult } = require('express-validator');
   app.post('/register',
     body('email').isEmail().normalizeEmail(),
     body('password').isLength({ min: 8 }),
     (req, res) => {
       const errors = validationResult(req);
       if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
       // ...
     });
   ```
2. **Hash passwords** with bcrypt (cost ≥ 12) or Argon2id. Never store plain text, and never use a fast hash such as SHA-256 (see A5.2).
   ```js
   const hash = await bcrypt.hash(password, 12);
   const ok = await bcrypt.compare(enteredPassword, hash);
   ```
3. **Authentication and authorisation:**
   - Use sessions or short-lived JWTs with refresh token rotation, in `HttpOnly`, `Secure`, `SameSite` cookies.
   - Apply **RBAC**.
   - Verify **resource ownership** on every request (`order.userId === req.user.id`), which prevents IDOR (insecure direct object reference).
4. **Rate limiting** with stricter limits on login, OTP and password reset (see the next question).
5. **Security headers** with `helmet()`, plus **CORS** with an explicit origin allowlist (never `*` with credentials).
6. **Prevent injection:**
   - Use parameterised SQL or an ORM, never string-concatenated queries.
   - Guard against **NoSQL injection** (`{ "$gt": "" }` in a login body) by validating types and using `express-mongo-sanitize`.
   - Never pass user input to a shell (A6.1).
   - Block prototype pollution, path traversal, ReDoS and SSRF (above).
7. **Secrets:** keep them in environment variables or a secrets manager (AWS Secrets Manager, Vault), never in Git. Rotate them and use least-privilege DB and cloud credentials.
8. **File uploads:**
   - Limit the size.
   - Check the real type with magic bytes, not the client's MIME type.
   - Generate random file names.
   - Store files outside the web root or in private S3 with presigned URLs.
   - Malware-scan where needed.
9. **Don't leak information:** use generic error messages, no stack traces in responses, no secrets or PII in logs, and `app.disable('x-powered-by')`.
10. **Dependencies and runtime:**
    - Run `npm audit` and Dependabot or Snyk, and commit the lockfile.
    - Keep Node on an LTS version.
    - Use HTTPS everywhere, and run as a non-root user in the container.

Interview-ready answer: *"I use defence in depth: server-side validation, bcrypt or Argon2 password hashing, solid authentication plus authorisation with ownership checks, rate limiting, Helmet headers and strict CORS, parameterised queries and NoSQL sanitisation, secrets in a secrets manager, and careful file upload handling. I keep sensitive data out of logs and responses, and keep dependencies audited and patched."*

Senior-level insight: **authentication** answers *who are you?* **Authorisation** answers *what may you do?* Both must be enforced **on the backend**, because frontend checks are UX only. Most real breaches are broken authorisation (IDOR) rather than broken cryptography.

**Q (Scenario): How would you implement rate limiting in a distributed Node.js application?**
A: Rate limiting caps how often a client (identified by IP, user ID or API key) can call an endpoint, for example 100 requests per minute. It protects against abuse, brute force and overload, and returns `429 Too Many Requests` with a `Retry-After` header.

**A single instance can use an in-memory counter** (a demo only, not production-ready):

```js
const hits = new Map();
function rateLimiter(req, res, next) {
  const now = Date.now(), windowMs = 60_000, limit = 100;
  const rec = hits.get(req.ip) ?? { count: 0, start: now };
  if (now - rec.start > windowMs) { rec.count = 0; rec.start = now; }
  rec.count++;
  hits.set(req.ip, rec);                    // also needs cleanup of old IPs, or it leaks
  if (rec.count > limit) return res.status(429).json({ message: 'Too many requests' });
  next();
}
```

**Why this breaks in a distributed setup:** behind a load balancer with 3 instances, each keeps its **own** counter. A client spread across them gets about 3 × the limit, and counters reset on every deploy or restart.

**Fix it with a shared store, usually Redis**, which all instances read and update **atomically**:

```js
const { rateLimit } = require('express-rate-limit');
const { RedisStore } = require('rate-limit-redis');
const { createClient } = require('redis');

const redisClient = createClient({ url: process.env.REDIS_URL });
await redisClient.connect();

app.set('trust proxy', 1);          // real client IP from the load balancer only (one hop)

const apiLimiter = rateLimit({
  windowMs: 60_000,
  limit: 100,
  standardHeaders: 'draft-8',       // RateLimit headers
  legacyHeaders: false,
  store: new RedisStore({ sendCommand: (...args) => redisClient.sendCommand(args) }),
});
const loginLimiter = rateLimit({
  windowMs: 15 * 60_000,
  limit: 5,                          // stricter for brute-force targets
  keyGenerator: (req) => `${req.ip}:${req.body?.email}`,
  store: new RedisStore({ sendCommand: (...args) => redisClient.sendCommand(args), prefix: 'rl:login:' }),
});

app.use('/api', apiLimiter);
app.post('/auth/login', loginLimiter, loginHandler);
```

Under the hood, a fixed window is just `INCR key` plus `EXPIRE key 60` done atomically, with `MULTI` or a Lua script. Atomicity matters: a separate "read, check, write" races between instances.

| Algorithm | How it works | Trade-off |
|---|---|---|
| Fixed window | Count per clock window (12:00–12:01) | Simple, but allows a 2× burst at window edges |
| Sliding window (log or counter) | Count over the last N seconds | Smoother and fairer; slightly more work |
| Token bucket | Tokens refill at a steady rate; each request spends one | Allows controlled bursts; common for APIs |
| Leaky bucket | Requests drain from a queue at a fixed rate | Smooths output; adds queueing delay |

Interview-ready answer: *"In-memory counters work for one instance, but with multiple instances behind a load balancer each keeps its own count, so the effective limit multiplies. I'd keep counters in Redis, updated atomically, so every instance enforces the same limit. I'd pick fixed window, sliding window or token bucket depending on how much burst I want to allow, and use stricter per-route limits for login, OTP and password reset."*

Senior-level insight:
- **Client identity:** set `trust proxy` correctly and never trust a raw `X-Forwarded-For` header, which anyone can spoof. Prefer user ID or API key over IP for authenticated routes, since many users can share one NAT IP.
- **Redis failure policy:** decide between fail-open (keep serving, risking abuse) and fail-closed (block everyone). Usually fail-open, with an alert.
- **Layering:** also rate-limit at the edge (an API gateway, Nginx, Cloudflare or a WAF), so floods never reach Node.

### [P5.1 — Encrypted Secrets Vault CLI](P5.1-encrypted-secrets-vault-cli.md)

**Q: What is the vault file format?**
A: Magic `VLT1`, scrypt cost byte, salt, IV, tag, then AES-256-GCM ciphertext of the JSON secrets.

**Q: Can it tell a wrong password from tampering?**
A: No — GCM fails authentication in both cases (exit code 3).

**Q: How is the master password handled?**
A: Read without echo in raw TTY mode (or from `VAULT_PASSWORD`), and stripped from the environment of `vault exec` children.

**Q: What does `passwd` do?**
A: Re-encrypts the vault with a new salt, IV and key.

### [P5.2 — Signed URL Service](P5.2-signed-url-service.md)

**Q: What is signed?**
A: A canonical string of method, path and sorted query parameters (excluding `sig`), with HMAC-SHA256.

**Q: Why trust parameters only after verifying the signature?**
A: Until then, `exp` and other values could be attacker-controlled.

**Q: How do key rotation and one-time links work?**
A: Links carry a `kid` selecting the key; one-time links carry a nonce recorded on first use, with reuse returning 410.

**Q: Which status codes does it use?**
A: 403 bad signature, 410 expired or used, 404 missing file, 401 for unauthenticated link creation.

## Phase 6 — Performance & Concurrency

### [A6.1 — Child Processes](A6.1-child-processes.md)

**Q: `spawn` vs `execFile` vs `exec` vs `fork`?**
A: `spawn` streams output for long-running processes; `execFile` buffers small output without a shell; `exec` uses a shell (trusted input only); `fork` runs Node scripts with an IPC channel.

Typical uses: running FFmpeg on a video, generating a PDF with an external tool, running a shell command or git, or a separate Node worker process.

```js
const { exec, spawn, fork } = require('node:child_process');

// exec: runs through a shell and buffers all output (maxBuffer is 1 MB by default)
exec('node --version', (error, stdout, stderr) => {
  if (error) return console.error(error);
  console.log('Output:', stdout);
});

// spawn: no shell, arguments as an array, output as streams
const ff = spawn('ffmpeg', ['-i', 'input.mp4', '-vf', 'scale=1280:-1', 'output.mp4']);
ff.stderr.on('data', (d) => console.log(d.toString()));   // ffmpeg reports progress on stderr
ff.on('error', (err) => console.error('Failed to start', err));
ff.on('close', (code) => console.log('Exited with', code));

// fork: a new Node process with an IPC channel
const child = fork('./worker.js');
child.send({ task: 'generateReport' });
child.on('message', (result) => console.log('Worker result:', result));

// worker.js
process.on('message', (msg) => {
  // ...do the work
  process.send({ status: 'completed' });
});
```

| | `exec()` | `execFile()` | `spawn()` | `fork()` |
|---|---|---|---|---|
| Shell | Yes | No | No (unless `shell: true`) | No |
| Output | Buffered, passed to a callback | Buffered | **Streams** | Streams + **IPC messages** |
| Best for | Small output, trusted command strings with pipes or globs | Small output, safe arguments | Large or continuous output, long-running processes | A separate **Node.js** process you talk to |
| IPC channel | No | No | No (unless configured) | Yes (`send`/`on('message')`) |

Interview-ready answer: *"They're all in `child_process`. `exec` runs a command in a shell and buffers the output, which is good for small output from trusted commands. `spawn` streams stdin, stdout and stderr, which suits long-running or large-output processes; I'd use it for FFmpeg. `fork` starts another Node script with an IPC channel for message passing. For CPU-heavy JavaScript I'd usually pick worker threads over fork, because they are lighter."*

Security: **never interpolate user input into `exec`.** `exec(\`convert ${filename}\`)` with `filename = "x; rm -rf /"` is command injection. Use `spawn`/`execFile` with an **argument array**, so no shell parses the input, and validate it against an allowlist (see A5.4).

**Q: `'error'` vs a non-zero exit?**
A: `'error'` means the process couldn't start (e.g. `ENOENT`); a failing command still starts and exits non-zero.

**Q: How do you stop a hung child?**
A: Use `timeout` or `signal: AbortSignal.timeout()`, send `SIGTERM`, and escalate to `SIGKILL` after a deadline.

### [A6.2 — Worker Threads](A6.2-worker-threads.md)

**Q: What is the difference between CPU-bound and I/O-bound operations?**
A: A **CPU-bound** task is limited by processor time: the code is busy computing. An **I/O-bound** task spends most of its time **waiting** on something external, such as a database, a disk, the network or another API. Think of one employee solving maths problems (CPU-bound) and another waiting for a reply from another department (I/O-bound).

```js
// CPU-bound: blocks the main thread for the whole loop, so every other request waits
function calculate() {
  let result = 0;
  for (let i = 0; i < 1_000_000_000; i++) result += i;
  return result;
}

// I/O-bound: the thread is free while the database works
const user = await User.findById(userId);
```

| | CPU-bound | I/O-bound |
|---|---|---|
| Bottleneck | Processor | External systems (DB, network, disk) |
| Examples | Video encoding, image resizing, large data transforms, compression, hashing, big `JSON.parse` | DB queries, HTTP calls, file reads, S3 uploads |
| Node handles it with | Worker threads or a pool, child processes, a separate service, job queues | Async I/O (built in) |
| Risk in Node | **Blocks the event loop** for every user | Exhausting connections or pools; slow dependencies |
| Optimise with | Better algorithms, moving work off the main thread, horizontal scaling | Connection pooling, caching, query and index tuning, timeouts, bounded concurrency |

Interview-ready answer: *"CPU-bound work is limited by computation; I/O-bound work mostly waits on databases, files or the network. Node is excellent at I/O-bound work because of its non-blocking model. CPU-heavy work I move to worker threads, separate processes or background jobs, so it doesn't block the event loop."*

Senior-level insight: the same service can **change category under load**. An API that is I/O-bound at 100 requests per second can become CPU-bound at 5,000, from JSON serialisation, validation and logging. Measure (event loop utilisation, CPU profiles; see A6.4) before choosing an optimisation.

**Q: What are Worker Threads, and when should you use them?**
A: For CPU-bound JavaScript; I/O-bound work is already async. Worker threads (`node:worker_threads`) run JavaScript on **separate threads inside the same process**. Each has its own V8 instance, event loop and heap, so heavy computation doesn't block the main thread that is serving requests.

```js
// main.js
const { Worker } = require('node:worker_threads');

function runReport(limit) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./worker.js', { workerData: { limit } });
    worker.once('message', resolve);
    worker.once('error', reject);
    worker.once('exit', (code) => code !== 0 && reject(new Error(`Worker exited with ${code}`)));
  });
}

// worker.js
const { parentPort, workerData } = require('node:worker_threads');
let total = 0;
for (let i = 0; i < workerData.limit; i++) total += i;
parentPort.postMessage(total);
```

**Use them for:** CPU-intensive calculations, image or video processing in JS, large data transforms or parsing, heavy reports, compression, and hashing in JS.

**Don't use them for:** DB queries, HTTP calls and file I/O. These are already non-blocking, so a worker only adds overhead and complexity.

Interview-ready answer: *"Worker threads run JavaScript in parallel threads inside one Node process. I use them for CPU-intensive work such as heavy calculations, image processing or large data transformations, so the main event loop stays responsive. For I/O-bound work I rely on Node's async APIs. In production I use a worker pool rather than a new worker per request."*

Senior-level insight: spawning a worker costs tens of milliseconds and several MB of memory, so creating one **per request** is an anti-pattern. Use a fixed pool sized to the cores (`piscina`, or a hand-rolled pool; see "Why use a pool?" below). For truly heavy or long jobs, a queue (BullMQ) plus separate worker services scales better than threads inside the API process.

**Q: How do workers communicate?**
A: Structured-clone messages, transferred `ArrayBuffer`s (zero copy), or `SharedArrayBuffer` coordinated with `Atomics`.

**Q: Why use a pool?**
A: Starting a worker costs milliseconds and memory; a fixed pool reuses workers and queues tasks.

**Q: Workers vs child processes vs cluster?**
A: Workers share a process and can share memory (CPU tasks); child processes isolate programs; cluster scales an HTTP server across cores.

| | Cluster | Child process | Worker threads |
|---|---|---|---|
| Unit | Multiple **processes** (copies of your app) | A separate **process** (any program) | Multiple **threads** in one process |
| Memory | Separate per process | Separate | Separate JS heaps; can share `SharedArrayBuffer` |
| Communication | IPC (via the primary) | stdio streams, or IPC with `fork` | `postMessage`, transferable objects, shared memory |
| Startup cost | High (a whole Node process each) | High | Lower |
| Crash isolation | One worker crashes, the others keep serving | Full isolation | A crash in a worker can be caught; bad native code can take down the whole process |
| Typical use | Use every CPU core for an **HTTP server** | Run **external commands** (FFmpeg, git) or isolate untrusted or unstable code | **CPU-heavy JS** (hashing, parsing, image work) |

Interview-ready answer: *"Cluster forks multiple copies of my server that share one port, to use all CPU cores for request handling. Child processes run separate programs or scripts with full isolation. Worker threads run CPU-heavy JavaScript in parallel inside one process. I'd scale request handling with cluster or more instances, offload computation to a worker pool, and use child processes for external tools or isolation."*

Senior-level insight: in modern deployments, scaling usually happens **outside** Node: one process per container, with more replicas behind a load balancer (Kubernetes, ECS) or PM2 cluster mode on VMs. The `cluster` module is mainly for bare-metal or VM setups (see A6.3).

**Q (Scenario): How would you implement background jobs with BullMQ and Redis? (For example, processing a video after upload)**
A: Anything slow or retryable (transcoding, thumbnails, emails, PDFs, notifications, webhooks) shouldn't block the HTTP response. The API **enqueues a job** and returns `202 Accepted` with a job ID. Separate **worker processes** pull jobs from a Redis-backed queue.

```
Client ─▶ API: save the upload ─▶ queue.add()  ─▶ 202 { jobId }
                                   Redis (BullMQ)
                                   Worker(s): FFmpeg, thumbnails, validation ─▶ update DB ─▶ notify the user
```

```js
// queue.js
const { Queue } = require('bullmq');
const IORedis = require('ioredis');
const connection = new IORedis(process.env.REDIS_URL, { maxRetriesPerRequest: null }); // required by BullMQ workers
const videoQueue = new Queue('video-processing', { connection });
module.exports = { videoQueue, connection };

// API
app.post('/upload', async (req, res, next) => {
  try {
    const video = await saveVideo(req.file);
    const job = await videoQueue.add(
      'process-video',
      { videoId: video.id, path: video.path },  // small, serialisable payload: IDs, not file contents
      {
        jobId: `video:${video.id}`,               // dedupes double-enqueues
        attempts: 3,
        backoff: { type: 'exponential', delay: 5000 },
        removeOnComplete: 100,
        removeOnFail: 500,
      }
    );
    res.status(202).json({ message: 'Video uploaded and queued', jobId: job.id });
  } catch (err) { next(err); }
});

// worker.js: a separate process or deployment
const { Worker } = require('bullmq');
const worker = new Worker('video-processing', async (job) => {
  const { videoId, path } = job.data;
  if (await isAlreadyProcessed(videoId)) return { skipped: true };   // idempotent
  await job.updateProgress(10);
  await processVideo(path);
  await updateVideoStatus(videoId, 'completed');
  return { success: true };
}, { connection, concurrency: 3 });

worker.on('failed', (job, err) => logger.error({ jobId: job?.id, err }, 'job failed'));
process.on('SIGTERM', async () => { await worker.close(); process.exit(0); }); // finish the current job first
```

| Concept | Meaning |
|---|---|
| Queue | Where jobs wait, stored in Redis |
| Job | One task with data and options |
| Worker | The process that runs jobs; `concurrency` = jobs in parallel per worker |
| Attempts / backoff | Automatic retries with fixed or exponential delay |
| Delayed / repeatable jobs | Run later (`delay`), or on a schedule (cron) |
| Priority, rate limiting | Higher-priority jobs first; cap jobs per second (for example to respect a provider's API limits) |
| Flows | Parent and child job dependencies |
| Stalled jobs | A worker died mid-job, so the job is re-queued |

Interview-ready answer: *"For work that shouldn't block the response, the API stores what it needs, adds a BullMQ job to Redis, and returns 202 with a job ID. Separate workers process jobs, for example FFmpeg for video, update the status in the database and notify the user. I configure retries with exponential backoff, concurrency limits, cleanup of old jobs, failure alerts and a dashboard such as Bull Board. Since jobs can be retried, the processing is idempotent."*

Senior-level insight: queues give **at-least-once** delivery, not exactly-once. A worker can crash after doing the work but before acknowledging it, so the job runs again. Make handlers **idempotent** (check state first, use unique constraints). Run workers as a **separate deployment**, so CPU-heavy jobs never compete with API requests and the two scale independently. Redis needs persistence (AOF) and memory monitoring, because it is now holding your jobs.

**Q: Kafka vs RabbitMQ: what's the difference, and when would you use each?**
A: **Kafka** is a distributed **event streaming platform**: an append-only, partitioned **log** that keeps events for a retention period. Each consumer group tracks its own **offset**, so many independent consumers can read, and **replay**, the same events. **RabbitMQ** is a **message broker**: producers publish to **exchanges**, which **route** messages to queues. Consumers **acknowledge** messages and they are removed. It is designed for delivering tasks, not storing history.

| | Kafka | RabbitMQ | BullMQ (Redis) |
|---|---|---|---|
| Model | Retained, partitioned log | Queues + exchanges | Job queue |
| After consumption | Kept (retention); consumers move their offset | Removed on ack | Completed or failed job records |
| Replay | **Built in** (rewind the offset) | Not its typical use | No |
| Ordering | Guaranteed **within a partition** (use a key such as `orderId`) | Per queue, with a single consumer | Per queue (FIFO/LIFO, priority) |
| Routing | Topics and partitions | **Flexible**: direct, topic, fanout and headers exchanges | Named queues |
| Throughput | Very high (millions per second), horizontally scaled | High | Moderate |
| Ops complexity | High (brokers, partitions, KRaft/ZooKeeper) | Medium | Low (you probably already run Redis) |
| Typical use | Event pipelines, analytics and activity tracking, event sourcing, CDC, many services consuming the same events | Task distribution, RPC, complex routing, per-message retries and DLQs | Node background jobs: emails, video, scheduled and delayed jobs |

Interview-ready answer: *"Kafka is an event streaming platform: it retains events in partitioned logs, so multiple consumer groups can process and replay them independently, with ordering per partition. RabbitMQ is a message broker with flexible exchange-based routing and acknowledgements, built for distributing tasks. I'd pick Kafka for high-throughput event pipelines and event-driven architectures where several systems consume the same stream, and RabbitMQ for task queues and routing. For ordinary Node background jobs, BullMQ on Redis is usually enough."*

Senior-level insight: don't add Kafka for a handful of background jobs. Its operational cost is real. Whatever the broker, delivery is **at-least-once** in practice, so consumers must be **idempotent**, and you need retries, dead-letter queues and consumer-lag monitoring.

### [A6.3 — Cluster & Load Distribution](A6.3-cluster-and-load-distribution.md)

**Q: How does cluster work?**
A: The primary forks workers that share one port; connections are distributed round-robin (except on Windows).

```js
const cluster = require('node:cluster');
const os = require('node:os');
const http = require('node:http');

if (cluster.isPrimary) {
  for (let i = 0; i < os.availableParallelism(); i++) cluster.fork();
  cluster.on('exit', (worker, code) => {
    console.log(`Worker ${worker.process.pid} died (${code}), restarting`);
    cluster.fork();                          // keep the pool at full size
  });
} else {
  http.createServer((req, res) => res.end(`Handled by ${process.pid}`)).listen(3000);
}
```

**Q: Do workers share memory?**
A: No — shared state must live in Redis or a database.

**Q: How does a zero-downtime restart work?**
A: Replace workers one at a time: start a new one, wait for `'listening'`, then gracefully close the old one.

**Q: When avoid cluster?**
A: In containers/Kubernetes, prefer one process per container and scale replicas.

**Q (Scenario): Your API must handle 10,000 concurrent requests. How would you design it?**
A: **Clarify first.** 10,000 *concurrent connections* (many idle, like WebSockets) is very different from 10,000 *requests per second*. Ask about payload sizes, the read/write mix, the latency target (p95/p99), and whether the work is I/O-bound or CPU-bound. Then:

```
            Load balancer / Nginx / ALB  (TLS, keep-alive, rate limit at the edge)
             │            │            │
         Node #1      Node #2      Node #N      (stateless, autoscaled)
             │            │            │
   ┌─────────┼────────────┼────────────┼─────────┐
 Postgres or Mongo     Redis cache     BullMQ ─▶ worker fleet (emails, video, PDFs)
 (pooled, indexed,
  read replicas)
```

1. **Keep the event loop free:** no sync I/O and no CPU-heavy work in request handlers. Offload to worker threads or background jobs.
2. **Scale horizontally:** run **stateless** instances (sessions in Redis, files in S3) behind a load balancer, one process per container, with autoscaling.
3. **Pool database connections**, sized to a total budget across all instances (see the next question). Add indexes and use read replicas for read-heavy traffic.
4. **Cache** hot, read-heavy data in Redis (and at the CDN for public GETs), with TTLs and invalidation.
5. **Move heavy or slow work to queues:** emails, notifications, video, PDFs.
6. **Protect the system:**
   - Rate limiting.
   - **Timeouts** on every outbound call and DB query.
   - Circuit breakers.
   - Bounded concurrency.
   - Load shedding (return 503 early when overloaded rather than collapsing).
7. **Tune the server:**
   - HTTP keep-alive.
   - Server timeouts (`headersTimeout`, `requestTimeout`).
   - File-descriptor limits (`ulimit -n`) high enough for the number of connections.
   - Compression, ideally at the proxy.
8. **Monitor:**
   - Requests per second.
   - **p95/p99 latency.**
   - Error rate.
   - **Event loop delay and utilisation.**
   - CPU and memory.
   - DB pool usage and waits.
   - Queue depth.
9. **Load-test** (k6, autocannon, Artillery) to find the real limit per instance before trusting the design.

Interview-ready answer: *"First I'd clarify whether it's 10,000 concurrent connections or 10,000 requests per second, and what the latency targets are. Then I'd keep handlers non-blocking, run stateless Node instances behind a load balancer with autoscaling, pool and index the database, cache hot data in Redis, push heavy work to BullMQ workers, and protect the system with rate limits, timeouts and circuit breakers. I'd monitor p99 latency, event loop delay and pool usage, and prove capacity with load tests rather than assume it."*

Senior-level insight: Little's law, *concurrency = throughput × latency*. 10,000 requests per second at 100 ms average means about 1,000 requests in flight at any moment. Usually the **database**, not Node, is the first bottleneck: cutting per-request DB work (caching, better queries) scales further than adding Node instances.

**Q: What is database connection pooling, and why is it important?**
A: Opening a DB connection is expensive: a TCP handshake, TLS, authentication, and server-side memory (Postgres forks a process per connection). A **pool** keeps a set of open connections and **lends** them to requests, getting each one back afterwards. It's like a restaurant with a fixed team of waiters serving many tables, instead of hiring a new waiter for every customer.

```js
// ❌ A new connection per request: slow, and under load it exhausts the DB's connection limit
app.get('/users', async (req, res) => {
  const conn = await createDatabaseConnection();
  const users = await conn.query('SELECT * FROM users');
  await conn.end();
  res.json(users);
});

// ✅ One pool per process, created once at startup
const { Pool } = require('pg');
const pool = new Pool({
  host: process.env.DB_HOST, user: process.env.DB_USER,
  password: process.env.DB_PASSWORD, database: process.env.DB_NAME,
  max: 20,                        // maximum connections from this process
  idleTimeoutMillis: 30_000,      // close idle connections
  connectionTimeoutMillis: 2_000, // fail fast if no connection is free
  statement_timeout: 5_000,       // kill runaway queries
});

app.get('/users', async (req, res, next) => {
  try {
    const { rows } = await pool.query('SELECT id, name FROM users LIMIT 50'); // checks out and returns automatically
    res.json(rows);
  } catch (err) { next(err); }
});
// For transactions: const client = await pool.connect(); ... finally { client.release(); }
```

Benefits:
- No per-request connection cost, so lower latency.
- **Caps** the load on the database.
- Handles bursts by queueing for a connection instead of overwhelming the DB.

| Setting | Purpose |
|---|---|
| `max` | Maximum connections per process (Mongoose: `maxPoolSize`, default 100) |
| `min` | Connections kept warm (where supported) |
| `idleTimeoutMillis` | When idle connections are closed |
| `connectionTimeoutMillis` | How long a request waits for a free connection before erroring |
| Statement or query timeout | Stops one slow query from holding a connection forever |

Interview-ready answer: *"A connection pool keeps reusable DB connections open instead of opening one per request. That removes connection overhead, improves latency, and limits how many connections hit the database. I configure the max size, idle, connection and query timeouts based on database capacity, always release clients in `finally`, and budget the total across every instance."*

Senior-level insight: **the pool size is per process.** 10 instances × `max: 20` = **200** connections, which may exceed Postgres's `max_connections` (100 by default), especially with autoscaling, workers and cron jobs added on top. Budget the total, or put **PgBouncer** or RDS Proxy in front. A bigger pool isn't always faster: past a point the DB thrashes, and a smaller pool with queueing gives better throughput. A **leaked client** (one that is never `release()`d) slowly drains the pool until every request times out.

**Q: How would you deploy a Node.js backend with Docker, Nginx and PM2?**
A:

| Component | Responsibility |
|---|---|
| **Docker** | Packages the app and its dependencies into a reproducible image |
| **Nginx** | Reverse proxy: **TLS termination**, gzip, static files, request size limits, basic rate limiting, load balancing across instances |
| **PM2** | Process manager on a VM: restarts on crash, cluster mode, log management, zero-downtime `reload` |
| **Cloud or VPS** (+ orchestrator) | Hosts it: EC2 or a VPS with Docker Compose, or ECS/Kubernetes |
| **TLS certificate** | HTTPS (Let's Encrypt / certbot, or a managed certificate on the load balancer) |

```
User ──HTTPS:443──▶ Nginx (TLS, proxy) ──HTTP:3000──▶ Docker container (Node API) ──▶ Postgres / Redis
```

A production Dockerfile (**multi-stage**, production dependencies only, **non-root** user):

```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build && npm prune --omit=dev

FROM node:22-alpine
ENV NODE_ENV=production
WORKDIR /app
COPY --from=build --chown=node:node /app/node_modules ./node_modules
COPY --from=build --chown=node:node /app/dist ./dist
COPY --chown=node:node package.json ./
USER node
EXPOSE 3000
HEALTHCHECK CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "dist/main.js"]
```

```nginx
server {
  listen 443 ssl http2;
  server_name api.example.com;
  ssl_certificate     /etc/letsencrypt/live/api.example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;
  client_max_body_size 10m;

  location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_http_version 1.1;                       # needed for WebSockets:
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }
}
```

Practices:
1. Multi-stage builds, `npm ci`, production dependencies only, a pinned base image, scanned with Trivy.
2. **Secrets from the environment or a secrets manager**, never baked into the image. Add a `.dockerignore` that excludes `.env`.
3. Nginx or a cloud load balancer in front, with HTTPS and HTTP → HTTPS redirects. Set `app.set('trust proxy', 1)` in Express.
4. **Health checks**: `/health` for liveness and `/ready` for readiness, which checks the DB and Redis.
5. Structured logs to stdout, collected centrally, plus metrics and alerts.
6. **Graceful shutdown** on `SIGTERM`: stop accepting connections, finish in-flight requests, close the DB pool, and exit within the deadline. Use `CMD ["node", ...]`, not `npm start`, so the signal reaches Node.
7. **Rolling or blue-green deploys** with readiness gates, for zero downtime and easy rollback.
8. DB migrations as a separate, backward-compatible deploy step, plus backups.
9. CI/CD: lint, test, build the image, push to a registry, deploy to staging, run smoke tests, then promote to production.

Interview-ready answer: *"I build a multi-stage Docker image that runs as a non-root user with production dependencies only, and inject secrets at runtime. Nginx, or a cloud load balancer, terminates TLS and reverse-proxies to the containers. I add health and readiness endpoints, structured logging and metrics, graceful SIGTERM handling, and rolling deploys through CI/CD. On a plain VPS, PM2 can supervise and cluster the processes. Under Kubernetes or ECS I run one Node process per container and let the orchestrator handle restarts and scaling."*

Senior-level insight: **don't run PM2 inside containers under an orchestrator.** It duplicates what Kubernetes or ECS already does (restarts, scaling, health), and it hides crashes from the orchestrator. Use PM2 (`pm2 start ecosystem.config.js -i max`, `pm2 reload`) on VMs without one.

### [A6.4 — Profiling & Memory Leaks](A6.4-profiling-and-memory-leaks.md)

**Q: How do you find CPU hot spots?**
A: Run under load with `--cpu-prof` and read the flame graph for wide frames in your code.

**Q: What is garbage collection in Node.js, and how does it work?**
A: Garbage collection is the automatic reclaiming of memory held by objects that are **no longer reachable**. "Reachable" means the object can be reached from a root (globals, the current call stack, closures, active handles) through a chain of references. Like tidying a room, it throws out only what nobody is using. Node uses **V8's garbage collector**, which is **generational**, based on the observation that most objects die young:

| Space | What lives there | How it's collected |
|---|---|---|
| **Young generation** (new space, a few MB) | Newly allocated objects | **Scavenge** (minor GC): frequent, fast, copies the survivors. Objects that survive two scavenges are **promoted** |
| **Old generation** | Long-lived objects (caches, app state, promoted objects) | **Mark-sweep-compact** (major GC): less frequent and more expensive. V8 does most of it **concurrently or incrementally** to keep pauses short |

```js
function createUser() {
  let user = { name: 'Pulkit', age: 28 };
  // ...use it
}                     // once the function returns, nothing references `user`, so it is collectable
createUser();
```

Major GCs can still pause the main thread ("stop-the-world" phases). A heap close to its limit triggers frequent, long GCs, which shows up as **latency spikes**. Raise the limit with `--max-old-space-size=4096` (in MB) if the app legitimately needs it.

Interview-ready answer: *"Garbage collection automatically frees memory for objects that are no longer reachable. V8 uses generational collection: new objects go in the young generation and are cleaned by fast, frequent scavenges, and survivors are promoted to the old generation, which a concurrent mark-sweep-compact collector cleans less often. As a backend developer I monitor heap usage and GC pauses, and avoid holding references longer than needed."*

Senior-level insight: GC **can't fix leaks**. If your code still holds a reference (a global Map, a listener, a closure), the object is reachable and stays in memory forever.

**Q: `heapUsed` vs `rss` vs `external`?**
A: Live JS objects; total process memory; memory for Buffers outside the heap. The full output of `process.memoryUsage()`:

| Field | Meaning |
|---|---|
| `rss` | Resident set size: **total** memory the OS gives the process (heap + stacks + native code + buffers) |
| `heapTotal` | The V8 heap reserved so far |
| `heapUsed` | The part of the heap holding live JS objects (**the one to watch for JS leaks**) |
| `external` | Memory of C++ objects tied to JS objects, such as Buffers |
| `arrayBuffers` | The `ArrayBuffer`/`Buffer` part of `external` |

If `rss` grows while `heapUsed` stays flat, suspect Buffers or native memory (`external`), not JS objects.

**Q: What is a leak vs normal growth?**
A: Normal memory rises and falls with GC; a leak keeps growing across GCs under steady load.

**Q: What is a memory leak in Node.js, and what commonly causes it?**
A: A leak is memory the app **no longer needs but still references**, so GC can't free it. Memory then climbs until the process slows down from constant GC and finally crashes with `JavaScript heap out of memory`. It's like a bucket you keep filling but never empty. Common causes:
1. **Unbounded collections:** module-level arrays, Maps or caches that only grow (per-user or per-request data stored forever).
2. **Event listeners** added per request or connection and never removed (`MaxListenersExceededWarning`; see A2.4).
3. **Timers** (`setInterval`) that are never cleared and keep their closures alive.
4. **Closures** that capture large objects and outlive them (callbacks stored in long-lived structures).
5. **Global variables**, including accidental ones.
6. **Unclosed resources:** sockets, streams, DB connections that are never released.
7. **Unbounded concurrency**, such as queuing or buffering unlimited in-flight work.

```js
// ❌ Leak: grows forever
const users = [];
setInterval(() => users.push({ id: Date.now(), data: new Array(10000).fill('data') }), 1000);

// ✅ Quick fix: bound it
function addUser(user) {
  users.push(user);
  if (users.length > 100) users.shift();
}

// ✅ Production fix for caches: an LRU with a size limit and a TTL
const { LRUCache } = require('lru-cache');
const cache = new LRUCache({ max: 10_000, ttl: 60_000 });
```

For caches keyed by an object, a `WeakMap` lets entries be collected along with the key.

**Q: How do you find a leak?**
A: Compare two heap snapshots, sort by delta, and follow retainers to the code holding objects. Step by step:
1. **Confirm it's a leak.** Watch `heapUsed` over time under steady load: in metrics (Prometheus or Grafana), or by logging `process.memoryUsage()` every few seconds. A sawtooth that keeps climbing after each GC means a leak.
2. **Reproduce** it locally or in staging with a load test (autocannon, k6).
3. **Take heap snapshots:** `node --inspect app.js`, open `chrome://inspect`, then the Memory tab. In production you can use `node --heapsnapshot-signal=SIGUSR2` or `v8.writeHeapSnapshot()`. Take one snapshot, run load, take a second.
4. **Compare** them: sort by size delta or object count delta, and follow the **retainers** chain to the variable or listener holding the objects.
5. **Fix the reference** (remove the listener, bound the cache, clear the timer), then load-test again to confirm memory is flat.

Interview-ready answer: *"A memory leak is when the app keeps references to objects it no longer needs, so the garbage collector can't free them. Common causes are unbounded caches and arrays, listeners or timers that are never removed, closures holding large objects, and globals. I confirm it by watching heapUsed climb across GCs under steady load, compare heap snapshots in Chrome DevTools to find what is accumulating and what retains it, then fix that reference."*

Senior-level insight: raising `--max-old-space-size` or adding auto-restarts only **delays** the crash; it doesn't fix the leak. A restart can be a temporary mitigation, but find the retainer.

**Q: What does ELU near 100% mean?**
A: The event loop is CPU-saturated; reduce work per request or add workers/processes.

**Q (Scenario): An API takes 5 seconds to respond. How would you optimise it?**
A: Don't open with "I'll add Redis." **Measure first, find the bottleneck, fix that, then measure again.**

**1. Measure.** Collect:
- Latency (p50, p95, p99) and throughput.
- Error rate.
- CPU and memory use.
- **Event loop delay and utilisation.**
- DB query time.
- External API latency.

Tools: structured logs with timings (Pino, Winston), metrics (Prometheus + Grafana), and **distributed tracing (OpenTelemetry)**. Tracing shows exactly which span inside the request takes the 5 seconds.

**2. Identify the bottleneck.** A common one is independent queries awaited **sequentially**:

```js
// ❌ About t1 + t2 + t3
app.get('/dashboard', async (req, res) => {
  const user = await User.findById(req.user.id);
  const orders = await Order.find({ userId: user.id });
  const notifications = await Notification.find({ userId: user.id });
  res.json({ user, orders, notifications });
});
```

**3. Run independent work concurrently:**

```js
// ✅ About max(t1, t2, t3)
app.get('/dashboard', async (req, res) => {
  const userId = req.user.id;
  const [user, orders, notifications] = await Promise.all([
    User.findById(userId),
    Order.find({ userId }),
    Notification.find({ userId }),
  ]);
  res.json({ user, orders, notifications });
});
```

**4. Optimise the database.** This is usually the biggest win.
- Add **indexes** for your filters and sorts, and check the query plan with `EXPLAIN` or `.explain()`.
- Use **projection** so only the needed fields are fetched.
- Fix **N+1 queries** (batch with `$in`/`JOIN`s or a DataLoader).
- **Paginate.**
- Use `.lean()` in Mongoose to skip document hydration.
- Size the connection pool correctly.

```js
const orders = await Order.find({ userId })
  .select('total status createdAt')   // projection
  .sort({ createdAt: -1 })
  .limit(20)                          // pagination
  .lean();                            // plain objects, faster
```

**5. Cache what's expensive and read-heavy**, with a **per-user key**, a TTL and a deliberate invalidation strategy:

```js
const key = `dashboard:user:${req.user.id}`;
const cached = await redis.get(key);
if (cached) return res.json(JSON.parse(cached));

const dashboard = await getDashboardData(req.user.id);
await redis.set(key, JSON.stringify(dashboard), { EX: 60 }); // node-redis v4 syntax
res.json(dashboard);
// When the user's orders change: await redis.del(key)
```

**6. Everything else:**
- Remove sync or CPU-heavy work from the request path (worker threads, a job queue).
- Add **timeouts** and circuit breakers for slow third-party APIs, and call them in parallel or in the background.
- Compress responses.
- Stream or paginate large payloads.
- Drop unnecessary middleware.
- Move non-critical work (emails, analytics) to a queue.

**7. Verify** with a load test (k6, autocannon) and compare the before and after metrics.

Interview-ready answer: *"I don't guess. I measure latency, DB query times, external calls, CPU, memory and event loop delay, ideally with tracing, to find where the 5 seconds go. Then I fix that bottleneck: run independent queries with Promise.all, add indexes and projections, fix N+1 queries, paginate, cache hot read-heavy data in Redis with proper invalidation, move CPU work off the main thread, and add timeouts on slow dependencies. Finally I load-test to confirm the improvement."*

Senior-level insight: caching everything adds stale data, invalidation bugs and another system to operate. Fix the slow query first and cache only what's still needed. `Promise.all` also multiplies concurrent DB load, so make sure the connection pool and database can take it.

**Q: How would you implement Redis caching? (Strategies, invalidation, stampedes)**
A: Redis is an in-memory key-value store with sub-millisecond reads. The default pattern is **cache-aside** (lazy loading): the app checks the cache, falls back to the DB on a miss, then fills the cache.

```
Request ─▶ API ─▶ Redis hit?  ── yes ─▶ return
                        └─ no ─▶ DB ─▶ SET in Redis (with TTL) ─▶ return
```

```ts
async function getUser(userId: string) {
  const key = `user:v1:${userId}`;                  // namespaced, versioned key
  const cached = await redis.get(key);
  if (cached) return cached === 'null' ? null : JSON.parse(cached);

  const user = await User.findById(userId).select('name email avatar').lean(); // plain object, no secrets
  await redis.set(key, JSON.stringify(user), { EX: user ? 600 : 60 });         // cache "not found" briefly too
  return user;
}

async function updateUser(userId: string, data: Partial<User>) {
  const user = await User.findByIdAndUpdate(userId, data, { new: true });
  await redis.del(`user:v1:${userId}`);             // write to the DB first, then invalidate
  return user;
}
```

| Strategy | How it works | Trade-off |
|---|---|---|
| **Cache-aside** | The app reads the cache, and on a miss loads from the DB and fills the cache | Simple and resilient (works if Redis is down); the first read is slow |
| Read-through | The cache library loads from the DB on a miss | Cleaner app code; needs library support |
| Write-through | Every write updates the DB **and** the cache | The cache is always fresh; slower writes |
| Write-behind | Write to the cache, flush to the DB asynchronously | Fast writes; **risk of losing data** |

Key concepts:
- **TTL:** an expiry for every key, so stale data heals itself. Add **jitter** (`600 + random(60)`) so keys don't all expire together.
- **Invalidation:** delete or update keys on writes. Deleting is safer than updating, because two concurrent writes can otherwise leave a stale value.
- **Cache miss / hit ratio:** monitor it. A low hit ratio means the cache is just cost.
- **Cache stampede (thundering herd):** a hot key expires and 1,000 requests hit the DB at once. Mitigate with:
  - A **lock** (`SET lock:key 1 NX PX 5000`), so only one request rebuilds the value while the others wait or get the stale copy.
  - Early probabilistic refresh, or **stale-while-revalidate**.
  - Request coalescing inside one process (share the same pending promise).
- **Hot keys:** a single key with extreme traffic. Add a short in-process LRU in front, or replicate the key.
- **Penetration:** repeated lookups for IDs that don't exist. Cache the `null` with a short TTL (as above), or use a Bloom filter.

Interview-ready answer: *"I usually use cache-aside: check Redis, on a miss read the DB, and populate the cache with a TTL. On writes I update the database first, then delete the related keys. I use namespaced keys and jittered TTLs, protect hot keys against stampedes with a lock or stale-while-revalidate, cache negative results briefly, and monitor the hit ratio and Redis memory. Caching is an optimisation layered on a correct database, never the source of truth."*

Senior-level insight:
- Cache **read-heavy, expensive, tolerably-stale** data. Don't cache what's cheap or must be perfectly fresh, such as balances or stock at checkout.
- Never cache sensitive fields you don't need (password hashes, tokens).
- Set `maxmemory` with an eviction policy (`allkeys-lru`).
- The app must still work if Redis is down: degrade to the DB, guarded by timeouts.

### [P6.1 — Parallel File Hasher](P6.1-parallel-file-hasher.md)

**Q: Why batch small files?**
A: Per-message overhead would dominate hashing tiny files.

**Q: Where does parallelism stop helping?**
A: When disk or memory bandwidth becomes the bottleneck rather than CPU.

**Q: How are unreadable files handled?**
A: Reported in the manifest with an error instead of crashing a worker.

### [P6.2 — Clustered HTTP Server](P6.2-clustered-http-server.md)

**Q: How are hung workers detected?**
A: Workers send heartbeats every second; three missed heartbeats get a worker killed and replaced.

**Q: How are crash loops prevented?**
A: Restarts use exponential backoff, and too many crashes per minute stop the supervisor.

**Q: How are stats aggregated?**
A: Workers ask the primary over IPC, which combines every worker's heartbeat data.

## Phase 7 — Testing & Debugging

### [A7.1 — The node:test Runner](A7.1-the-node-test-runner.md)

**Q: How does `node --test` find tests?**
A: Files like `*.test.js` or inside `test/`, each run in its own process in parallel.

**Q: Which assert module should you use?**
A: `node:assert/strict`.

**Q: What is the forgotten-await bug?**
A: An un-awaited `assert.rejects` lets the test pass before the assertion runs.

**Q: How do you enforce coverage?**
A: `--experimental-test-coverage` with thresholds such as `--test-coverage-lines=90`.

**Q: How do you test a Node.js backend?**
A: Use a layered strategy, run in CI on every pull request:

| Level | What it verifies | Tools |
|---|---|---|
| **Unit** | One function or service in isolation, with dependencies mocked: business rules, calculations, edge cases | Jest, Vitest or `node:test`, with mocks or fakes |
| **Integration** | Components working together for real: service ↔ a **real DB**, repositories, queues, the HTTP layer | Supertest, **Testcontainers** (real Postgres or Redis in Docker), MSW or nock for external APIs |
| **End-to-end** | Whole user flows through the running system: register → login → create booking → pay | Supertest against the deployed stack, Playwright for UI flows, Postman/Newman |
| Contract (microservices) | Producer and consumer agree on the API shape | Pact |
| Load / performance | Throughput and latency under load | k6, autocannon |

```ts
// Unit: mock the repository
describe('UsersService', () => {
  it('returns the user', async () => {
    const repo = { findOne: jest.fn().mockResolvedValue({ id: '123', name: 'Pulkit' }) };
    const service = new UsersService(repo as any);
    await expect(service.findById('123')).resolves.toEqual({ id: '123', name: 'Pulkit' });
  });

  it('throws NotFound when missing', async () => {
    const repo = { findOne: jest.fn().mockResolvedValue(null) };
    await expect(new UsersService(repo as any).findById('x')).rejects.toThrow(NotFoundException);
  });
});

// Integration/API: real app and a real test DB, through HTTP
import request from 'supertest';
it('POST /users validates input', async () => {
  const res = await request(app).post('/users').send({ email: 'not-an-email' });
  expect(res.status).toBe(400);
});
it('GET /admin is forbidden for normal users', async () => {
  await request(app).get('/admin').set('Authorization', `Bearer ${userToken}`).expect(403);
});
```

What to cover beyond the happy path:
- Validation errors.
- **Authorisation**: 401, 403, and accessing another user's resource.
- Not-found cases and error mapping.
- **Concurrency**: two simultaneous orders for the last item, duplicate idempotency keys.
- Retries and timeouts of external calls.
- Migrations.

Interview-ready answer: *"I test at three levels. Unit tests with Jest cover business logic with mocked dependencies. Integration tests use Supertest and a real database via Testcontainers to check the API, data access and error handling. A few end-to-end tests cover the critical user flows. I specifically test validation, authorisation, edge cases and concurrent operations, mock only external third-party services, and run everything in CI so failing tests block deployment. Coverage is a guide, not the goal; behaviour is."*

Senior-level insight: **integration tests against a real database give the most confidence per test** for a backend, because mocking the ORM hides real query, constraint and transaction bugs. Keep the tests isolated (a transaction rollback or truncation per test), deterministic (fake timers, fixed seeds) and fast enough to run on every push.

### [A7.2 — Mocking & Test Doubles](A7.2-mocking-and-test-doubles.md)

**Q: Stub vs spy vs fake?**
A: A stub returns canned values, a spy records calls, a fake is a working lightweight implementation.

**Q: Why prefer dependency injection to module mocking?**
A: Injected fakes are clearer and need no experimental flags or import ordering tricks.

**Q: How do you test timers without waiting?**
A: `t.mock.timers.enable()` and `tick(ms)`, or inject a `sleep`/clock.

### [A7.3 — Debugging with the Inspector](A7.3-debugging-with-the-inspector.md)

**Q: `--inspect` vs `--inspect-brk`?**
A: The first runs normally and accepts a debugger; the second pauses before the first line.

**Q: Why is `--inspect=0.0.0.0` dangerous?**
A: The inspector allows arbitrary code execution to anyone who can reach the port.

**Q: Why never pause a production server?**
A: While paused, every request hangs.

**Q: What is `util.debuglog`?**
A: Logging that only prints when `NODE_DEBUG` includes its section name.

### [A7.4 — Error Handling & Diagnostics](A7.4-error-handling-and-diagnostics.md)

**Q: How do you handle errors in Node.js?**
A: Match the mechanism to the execution model:

| Code style | How to catch |
|---|---|
| Synchronous | `try/catch` |
| Error-first callback | Check `err` first: `if (err) return ...` (try/catch can't catch it; see A2.1) |
| Promise chain | `.catch()` at the end of the chain |
| async/await | `try/catch` around `await` |
| EventEmitter or stream | An `'error'` listener, or `pipeline` |

```js
try { JSON.parse('{invalid json}'); } catch (e) { console.error(e.message); }

fs.readFile('missing.txt', (err, data) => {
  if (err) return console.error('File error:', err.message);
});

getUser().then(console.log).catch(console.error);

async function getUserDetails(id) {
  try {
    return await getUser(id);
  } catch (err) {
    throw new Error(`Failed to load user ${id}`, { cause: err }); // add context, keep the original
  }
}
```

**Centralised handling in Express:** throw or `next(err)` from controllers, and map errors to responses in **one** error middleware. It is recognised by its **four arguments** and registered **after** all routes.

```js
class AppError extends Error {
  constructor(message, statusCode, code) {
    super(message);
    this.name = 'AppError';
    this.statusCode = statusCode;
    this.code = code;             // a stable machine-readable code, e.g. 'USER_NOT_FOUND'
    this.isOperational = true;    // expected failure (see the next question)
  }
}

app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) throw new AppError('User not found', 404, 'USER_NOT_FOUND');
    res.json(user);
  } catch (err) {
    next(err);                    // Express 5 does this automatically for async handlers
  }
});

app.use((req, res) => res.status(404).json({ message: 'Route not found' }));

app.use((err, req, res, next) => { // error handler: always 4 arguments, always last
  const status = err.statusCode ?? 500;
  logger.error({ err, path: req.path, requestId: req.id });
  res.status(status).json({
    success: false,
    code: err.code ?? 'INTERNAL_ERROR',
    message: status === 500 ? 'Internal Server Error' : err.message, // never leak stack traces
  });
});
```

Interview-ready answer: *"I handle errors according to the execution model: try/catch for sync code and async/await, error-first checks for callbacks, `.catch()` for promise chains, and `'error'` listeners for emitters and streams. In Express I use custom error classes and one centralised error middleware, so controllers stay clean and responses are consistent. I log with context using Winston or Pino, and never expose stack traces or internal details to clients."*

Senior-level insight:
- **Never swallow errors silently** (an empty `catch {}`).
- Add context when rethrowing and keep the original as `cause`.
- Separate expected business errors from unexpected system failures.
- Include a request or correlation ID in logs, and send unexpected errors to Sentry or similar.

**Q: Why use error codes rather than messages?**
A: Messages change; stable `code`s let callers branch reliably.

**Q: What is `cause` for?**
A: Keeping the original low-level error when rethrowing a higher-level one.

**Q: Operational vs programmer errors?**
A: Handle and recover from operational failures; for bugs, log and let the process crash and be restarted.

- **Operational errors** are **expected** runtime failures in correct code: invalid input, a missing record, authentication failure, a DB timeout, a third-party API that is down, `ENOENT`, a payment provider timeout. Handle them: validate, return a 4xx, retry with backoff, use a fallback, or report.
- **Programmer errors** are **bugs**: reading a property of `undefined`, calling something that isn't a function, a wrong type, broken assumptions about state, a logic error. You can't "handle" a bug at runtime. Fix the code.

An analogy: a power cut in a restaurant kitchen is operational, while the chef following the wrong recipe is a programmer error.

```js
// Operational: an expected condition, handled as a 404
const user = await User.findById(id);
if (!user) throw new AppError('User not found', 404);

// Programmer error: a bug, because calculateTotal wasn't designed for undefined
const calculateTotal = (order) => order.items.reduce((t, i) => t + i.price, 0);
calculateTotal(undefined); // TypeError: Cannot read properties of undefined
```

| Operational | Programmer |
|---|---|
| Expected runtime failure | A bug in the code |
| Often recoverable | Needs a code fix |
| Handle, report, maybe retry | Log, investigate, fix, and add a test |
| Example: DB timeout | Example: `TypeError` |

Interview-ready answer: *"Operational errors are expected failures such as timeouts, validation errors, missing records or third-party outages. I handle them with validation, timeouts, bounded retries and centralised error responses. Programmer errors are bugs, such as accessing undefined or calling an invalid function. Those need debugging, tests and a fix, not suppression. If one crashes the process, I let it restart cleanly."*

Senior-level insight: retry **only** transient operational errors (timeouts, 503s, connection resets), with **bounded retries, exponential backoff and jitter**, and only for idempotent operations. Retrying everything blindly, including 400s and bugs, can turn a small outage into a retry storm.

**Q: What are `uncaughtException` and `unhandledRejection`, and what should their handlers do?**
A: Both are **process-level** events, a last line of defence for errors nothing else handled.
- **`uncaughtException`:** a thrown error propagated all the way up the call stack with no `try/catch`.
- **`unhandledRejection`:** a promise was rejected and no `.catch`/`try` handler was attached by the end of that turn of the event loop. Since Node 15 the default (`--unhandled-rejections=throw`) turns it into an uncaught exception, so it **crashes the process** (see A2.2).

| | `uncaughtException` | `unhandledRejection` |
|---|---|---|
| Trigger | An uncaught synchronous throw | A rejected promise with no handler |
| Typical cause | An unexpected bug | A missing `.catch`/`try`, or an un-awaited promise |
| Response | **Treat it as fatal** | Fix the source; in practice also treat it as fatal |

What the handler should do: **log, stop accepting work, and exit**. The process state is unknown; a half-finished operation may have left shared state corrupt. Do **not** catch it and continue serving requests.
1. Log the error with full context and write diagnostics (see the diagnostic report below).
2. Stop accepting new connections (`server.close()`).
3. Close the DB, queues and other resources where possible.
4. Exit with a non-zero code, with a **timeout** so a hung shutdown can't block the exit.
5. Let PM2, systemd, Docker or Kubernetes **restart** the process.

```js
function fatal(type) {
  return (err) => {
    logger.fatal({ err }, type);
    server.close(() => process.exit(1));             // finish in-flight requests, then exit
    setTimeout(() => process.exit(1), 10_000).unref(); // hard deadline if close hangs
  };
}
process.on('uncaughtException', fatal('uncaughtException'));
process.on('unhandledRejection', fatal('unhandledRejection'));
```

Interview-ready answer: *"`uncaughtException` fires when a thrown error isn't caught anywhere; `unhandledRejection` fires when a promise rejects with no handler. I treat both as fatal: log with context, shut down gracefully with a timeout, and let the process manager or orchestrator restart it. These handlers are a safety net, not a replacement for handling errors where they occur."*

**Q: What is a diagnostic report?**
A: A JSON snapshot with JS and native stacks, heap stats, handles and environment, written on fatal errors or on demand.

**Q: How do you implement logging, monitoring and distributed tracing (observability)?**
A: They answer three different questions:

| Signal | Tells you | Example | Tools |
|---|---|---|---|
| **Logs** | *What* happened, with details | `payment failed for order 101: card_declined` | Pino or Winston → ELK, Loki, CloudWatch, Datadog |
| **Metrics** | *How healthy* the system is, as numbers over time, which you alert on | p99 latency 800 ms, error rate 2%, CPU 90% | `prom-client` → Prometheus → Grafana dashboards and alerts |
| **Traces** | *Where* a single request spent its time, across services | API 40 ms → DB 600 ms → payment service 150 ms | **OpenTelemetry** → Jaeger, Tempo, Datadog APM |

**Logging:** **structured JSON** with levels and a **request or correlation ID** on every line, so one request can be followed across logs and services.

```ts
import pino from 'pino';
import { randomUUID } from 'node:crypto';

const logger = pino({ level: process.env.LOG_LEVEL ?? 'info', redact: ['req.headers.authorization', '*.password', '*.cardNumber'] });

app.use((req, res, next) => {
  req.id = req.get('x-request-id') ?? randomUUID();    // pass it on to downstream services too
  req.log = logger.child({ requestId: req.id });
  res.setHeader('x-request-id', req.id);
  next();
});

req.log.info({ userId: '123' }, 'user created');
```

(Winston works the same way: `winston.format.combine(timestamp(), json())`. Pino is faster. Log to **stdout** and let the platform ship the logs, rather than writing files inside containers.)

**Metrics to monitor:**
- The **RED** metrics for every route: **R**ate, **E**rrors, **D**uration (p50/p95/p99).
- CPU and memory, **event loop delay**, GC.
- DB pool usage and wait time, slow queries.
- Redis latency and memory.
- Queue depth, failed jobs, job age.
- Business KPIs (orders per minute, payment success rate).

**Alert** on symptoms users feel (an SLO burn: error rate, p99), not on every CPU blip.

**Tracing:** OpenTelemetry auto-instruments `http`, Express, `pg`, Mongo, Redis and more, and propagates the trace context (`traceparent`) between services:

```bash
node --require @opentelemetry/auto-instrumentations-node/register dist/main.js
# with OTEL_SERVICE_NAME=orders-api and OTEL_EXPORTER_OTLP_ENDPOINT=http://collector:4318
```

Interview-ready answer: *"I use structured JSON logging with Pino or Winston, with a request ID on every log line and sensitive fields redacted. For monitoring I export RED metrics plus event loop delay, memory, DB pool and queue metrics to Prometheus, with Grafana dashboards and SLO-based alerts. For distributed systems I add OpenTelemetry tracing, so I can follow a request across services and see exactly where the latency is. I never log passwords, tokens or payment details."*

Senior-level insight: **link the three signals.** Put the `traceId` in every log line, so from a slow trace you jump straight to its logs and from an alert to example traces. Also watch costs: sample traces and keep log volume sensible, because observability bills grow fast at scale.

**Q (Scenario): Your Node.js app is crashing in production: 500s, timeouts, restarts, high memory, DB connection errors. How do you debug it?**
A: **Priority one is restoring service and limiting customer impact. Root cause comes second.**

1. **Assess the impact and communicate.** Which endpoints, how many users, since when? Declare an incident and open a status channel.
2. **Check what changed.** Most incidents follow a change: a deploy, a config or env change, a dependency bump, a migration, a feature flag, or a traffic spike or marketing campaign. If a recent deploy lines up with the start, **roll back first** and investigate afterwards.
3. **Logs.** Use centralised logs (ELK, Loki, CloudWatch, Datadog), or `pm2 logs`, `docker logs -f <container>`, `kubectl logs <pod> --previous`. Look for:
   - Uncaught exceptions and unhandled rejections.
   - `JavaScript heap out of memory`.
   - `ECONNREFUSED`, `ETIMEDOUT`, pool timeouts.
   - Errors from third-party APIs.
4. **Process health.** `pm2 status` or `kubectl describe pod` shows restart counts, uptime and exit codes. **Exit code 137 / OOMKilled** means the container ran out of memory; 1 means an uncaught error.
5. **System resources:**
   - `top`/`htop` for CPU.
   - `free -h` for memory.
   - `df -h` for disk; a full disk from logs is a classic cause.
   - Network and file-descriptor limits.
6. **Application metrics** (Prometheus/Grafana or APM):
   - Latency (p99).
   - Error rate.
   - **Event loop delay.** If it is high, something is blocking: CPU work, sync calls, huge `JSON.parse`.
   - Heap growth. If `heapUsed` climbs across GCs, it's a leak (see A6.4).
   - GC pauses.
7. **Dependencies:**
   - DB health, slow query log, **pool exhaustion** (leaked clients, a pool too small for the traffic), lock contention, replication lag.
   - Redis memory.
   - Third-party API latency.
8. **Capture evidence before restarting:**
   - A diagnostic report (`--report-on-fatalerror`).
   - A heap snapshot (`--heapsnapshot-near-heap-limit=1`, or a signal).
   - A CPU profile.
   - Traces from OpenTelemetry.
9. **Mitigate:** roll back, restart, scale out, raise a limit temporarily, disable a feature flag, rate-limit the abusive client, or fail over the DB.
10. **After it's stable:**
    - Root-cause analysis.
    - **Reproduce it**, and add a regression test.
    - Fix the code.
    - Add the missing **alert or dashboard**.
    - Write a **blameless postmortem** with action items.

Interview-ready answer: *"First I restore service and reduce impact. I check what changed recently and roll back if a deploy lines up. Then I look at logs, process restarts and exit codes, CPU, memory and disk, event loop delay, heap growth, and database health, including connection pool exhaustion and slow queries, using tools like PM2 or kubectl, centralised logging, and Prometheus with Grafana. Once it's stable, I do a root-cause analysis, reproduce the issue, add a regression test and the missing monitoring, and write a blameless postmortem."*

Senior-level insight: **a restart is recovery, not a fix.** If the process OOMs every 6 hours, auto-restart just hides a leak. Grab a heap snapshot or diagnostic report *before* restarting where you safely can; evidence disappears with the process.

---

## Backend Architecture & Leadership

These interview topics go beyond the curriculum files (framework, architecture and team questions), so they have no source file to link to.

### Architecture

**Q: How would you design a scalable backend architecture?**
A: Give each component one responsibility, keep the app tier **stateless**, and scale the parts that are actually under pressure. It's like a restaurant: ordering, kitchen, cashier and delivery each do one job.

```
                Client / Frontend ─▶ CDN (static assets, cacheable GETs)
                         │
                  Load balancer (TLS)
                         │
        API instances (NestJS modular monolith, stateless, autoscaled)
        ┌──────────┬──────────┬──────────┬───────────────┐
        Auth       Users      Bookings   Payments  …  (modules with clear boundaries)
        └──────────┴────┬─────┴──────────┴───────────────┘
           ┌────────────┼─────────────┬───────────────┐
         Redis       PostgreSQL /    Queue (BullMQ)  Object storage (S3)
     (cache, rate    MongoDB (pooled,    │
      limits,        indexed, replicas)  Workers (email, video, reports)
      sessions)
                         │
     Observability: logs + metrics + traces, alerts, health checks
```

| Component | Role |
|---|---|
| Load balancer | Spreads traffic across instances, health-checks them, terminates TLS |
| API layer | Authentication, validation, routing, orchestrating the business logic |
| Business modules | One per domain (users, bookings, payments, notifications), with explicit interfaces between them |
| Redis | Cache, shared counters (rate limits), sessions, locks, queue backend |
| Database | The source of truth. Pooled, indexed, with read replicas when reads dominate |
| Queue + workers | Async and heavy work (emails, video, PDFs, webhooks) off the request path |
| Observability | Logs, metrics, traces and alerts (see A7.4) |

Interview-ready answer: *"I'd start with a modular monolith in NestJS, with each business domain in its own module behind a clear interface. PostgreSQL or MongoDB depending on the data model, Redis for caching and shared state, and BullMQ workers for background processing. For scale I run stateless instances behind a load balancer with autoscaling, pool connections, tune queries and indexes, and add read replicas and caching where measurements show a need. Centralised logging, metrics, tracing and health checks come from day one. If a module later needs independent scaling, deployment or ownership, its clean boundary makes it easy to extract into a service."*

Senior-level insight: don't reach for microservices because the app is "growing". **Find the real bottleneck and the real business boundaries first.** Most scale problems are solved by stateless horizontal scaling, caching, queues and database work, long before service boundaries matter. (For concrete numbers, see the 10,000-concurrent-requests scenario in A6.3.)

**Q: Monolith vs microservices: what's the difference, and when should you split?**
A: A **monolith** is one deployable application containing all the modules. **Microservices** are many independently deployable services that talk over the network (HTTP/gRPC or messaging), each owning its own data. A **modular monolith** is one deployable app with strictly separated modules, and is usually the best starting point.

| | Monolith (modular) | Microservices |
|---|---|---|
| Deployment | One artefact | Each service independently |
| Development | Simpler at first; one repo, one build | More moving parts, version and contract management |
| Scaling | The whole app scales together | Per service |
| Communication | In-process function calls (fast, typed) | Network calls: latency, failures, retries, timeouts |
| Data consistency | **Local ACID transactions** | Eventual consistency, sagas and outboxes (see P4.1) |
| Debugging | A single stack trace and log | Distributed tracing required |
| Infrastructure and ops | Simple | Service discovery, gateways, per-service CI/CD and monitoring |
| Team fit | One or a few teams | Many teams that need autonomy |

Consider splitting when:
- A module needs **independent scaling** (video transcoding, search).
- **Separate teams** need to deploy independently without coordinating.
- The **business boundary is clear and stable**.
- It needs a different technology or runtime.
- It needs **independent availability**, so its outages must not take down the core.

Interview-ready answer: *"I prefer starting with a modular monolith: development, deployment and debugging are simpler, and I keep real transactions. I enforce clean module boundaries, such as auth, users, payments and notifications, with no reaching into another module's tables. When a specific module becomes a scaling bottleneck, or needs independent deployment or team ownership, I extract it into a service. That way we pay the cost of a distributed system only where it buys something."*

Senior-level insight: microservices trade **code complexity for operational complexity**. A "distributed monolith", where services must deploy together, share a database, or call each other synchronously in long chains, gets the costs of both and the benefits of neither. Use the **strangler-fig** approach to extract services gradually from a well-modularised monolith.

**Q: How would you structure a NestJS application?**
A: Use a **feature-based modular** structure. Each business domain is a module that owns its controller, service, DTOs and data access, while cross-cutting infrastructure lives in `common/` and `config/`.

```
src/
├── main.ts                      # bootstrap: global pipes, filters, CORS, helmet, shutdown hooks
├── app.module.ts                # imports feature modules + ConfigModule, DB module
├── config/                      # typed, validated config (@nestjs/config + Joi/Zod)
├── common/
│   ├── guards/                  # JwtAuthGuard, RolesGuard
│   ├── filters/                 # global exception filter → consistent error shape
│   ├── interceptors/            # logging, timing, response mapping, caching
│   ├── decorators/              # @CurrentUser(), @Roles()
│   └── pipes/                   # custom validation/transformation
└── modules/
    ├── auth/      { auth.module.ts, auth.controller.ts, auth.service.ts, strategies/, dto/ }
    ├── users/     { users.module.ts, users.controller.ts, users.service.ts, entities/, dto/ }
    └── payments/  { payments.module.ts, payments.controller.ts, payments.service.ts, dto/ }
```

| Building block | Responsibility |
|---|---|
| **Module** | Groups related providers; defines what the module `exports` to others (its public API) |
| **Controller** | HTTP concerns only: routes, params, status codes. No business logic |
| **Service** (provider) | Business logic, injected through **dependency injection** |
| **Repository** / data access | DB queries (TypeORM, Prisma, Mongoose) |
| **DTO** + `ValidationPipe` | The request shape, validated with `class-validator` (`whitelist: true` strips unknown fields) |
| **Guard** | Can this request proceed? Authentication and authorisation |
| **Pipe** | Validate or transform input (`ParseUUIDPipe`) |
| **Interceptor** | Wraps a handler: logging, timing, response mapping, caching |
| **Exception filter** | Maps exceptions to HTTP responses consistently |

Request lifecycle: **middleware → guards → interceptors (before) → pipes → controller → service → interceptors (after) → exception filters** (on error).

```ts
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get(':id')
  getUser(@Param('id', ParseUUIDPipe) id: string) {
    return this.usersService.findById(id);
  }
}

@Injectable()
export class UsersService {
  constructor(@InjectRepository(User) private readonly userRepository: Repository<User>) {}

  async findById(id: string) {
    const user = await this.userRepository.findOne({ where: { id } });
    if (!user) throw new NotFoundException('User not found');   // the filter turns this into a 404
    return user;
  }
}

// main.ts
app.useGlobalPipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true, transform: true }));
app.enableShutdownHooks();
```

Interview-ready answer: *"I organise NestJS by feature: each domain has its own module with a controller, service, DTOs and repository. Controllers handle only HTTP, services hold the business logic and are injected via DI, DTOs with a global ValidationPipe validate input, guards handle authentication and RBAC, interceptors handle cross-cutting concerns like logging and response mapping, and a global exception filter standardises errors. Shared infrastructure lives in common and config modules, modules expose only what others need, and I avoid circular dependencies between business modules."*

Senior-level insight: if you need `forwardRef()` to break circular module imports, it usually means the boundaries are wrong. Extract the shared concept into its own module, or communicate through events (`@nestjs/event-emitter`, or a queue). Keeping modules this independent is what makes extracting a microservice later cheap.

### Leadership

**Q: As a Tech Lead, how do you manage developers, code reviews and architecture decisions?**
A: The job is making the **team** deliver reliable, maintainable software that meets business goals, not writing the most code.

1. **Code reviews.** Check, in priority order:
   - **Correctness** and edge cases.
   - **Security**: authorisation checks, injection, secrets, PII in logs.
   - Error handling.
   - **Data and performance**: N+1 queries, missing indexes, unbounded queries, blocking calls.
   - Tests that cover the change.
   - Readability.
   - Consistency with the architecture.

   Automate style with linters, formatters and CI, so reviews don't argue about preferences. Keep PRs small, review quickly (within a day), explain the *why*, and label comments (blocking vs nit). Praise good work too.
2. **Leading developers:**
   - Break epics into small, independently shippable tasks with clear acceptance criteria.
   - Assign work by strengths **and** growth goals (stretch tasks with support).
   - Estimate realistically, with buffers.
   - Unblock people fast.
   - Run design discussions and hold regular one-to-ones.
   - Track progress through outcomes, not micromanagement.
   - Make sure knowledge isn't held by one person (pairing, rotation, docs).
3. **Architecture decisions:**
   - Start from the business requirements and constraints.
   - Compare options on scalability, performance, security, **cost**, maintainability, team skills and time to market.
   - Write it down in a short **design doc or ADR** (Architecture Decision Record): context, options, the decision, the consequences.
   - Get input, decide, and revisit when the facts change.
   - Prefer reversible, boring technology unless there's a strong reason.
4. **Technical debt:**
   - Keep it visible in a backlog.
   - Prioritise by **business impact × engineering risk**. A slow query on the most-used API beats refactoring an unrelated utility.
   - Reserve regular capacity for it (for example 15–20% of each sprint).
5. **Quality and operations:**
   - Coding standards and a definition of done (tests, docs, monitoring).
   - CI/CD and quality gates.
   - On-call and incident handling with blameless postmortems.
   - Delivery metrics (DORA: deployment frequency, lead time, change failure rate, time to restore).
6. **Stakeholders:** translate between the business and engineering, flag risks early, negotiate scope versus deadline, and say no with alternatives.

Interview-ready answer (adapt it with your own real examples): *"As a tech lead my responsibility is that the team ships maintainable, reliable and scalable software that meets the business goals. I start from the requirements, shape the technical approach, often with a short design doc or ADR, and break the work into small tasks with clear owners. In code reviews I focus on correctness, security, performance and maintainability rather than personal style, which tooling handles. I mentor developers, unblock them quickly, and assign stretch work so people grow. I make architecture decisions from requirements and trade-offs, cost included, and I manage tech debt by business impact. For example, on [project], I [concrete decision or result]."*

Senior-level insight: interviewers want **evidence**, not adjectives. Prepare 3–4 STAR stories (Situation, Task, Action, Result) with numbers: a production incident you led, a disagreement you resolved, a developer you grew, an architecture decision and its trade-off, a performance win ("p99 from 2 s to 300 ms"). A good tech lead balances **technical quality, team delivery and business priorities**, and multiplies the team instead of being its bottleneck.

---

## Study Strategy

### [HOW_TO_LEARN.md](HOW_TO_LEARN.md)

**Q: Why does the curriculum avoid frameworks?**
A: Frameworks are built from `node:http`, streams, EventEmitter and the event loop; knowing those makes any framework easy to learn and debug.

**Q: What is the learning loop?**
A: Read → type → run → break it on purpose → explain why → fix.

**Q: Why might a process never exit?**
A: An open server, socket, interval or connection keeps the event loop alive — close it or `unref()` it.

### Interview Quick Revision

**Q: What is the one-line takeaway for each common interview topic?**
A:

| Topic | Key concept to remember | Section |
|---|---|---|
| What is Node.js? | V8 + libuv + event loop; non-blocking I/O | A1.1 |
| Single or multithreaded? | One main JS thread, a multithreaded runtime | A1.1 |
| Event loop | Runs async callbacks when their operations complete | A2.3 |
| Event loop phases | Timers → Pending → Idle → Poll → Check → Close | A2.3 |
| Sync vs async | Blocking vs non-blocking; async improves throughput, not speed | A2.1 |
| Callbacks | A function passed in and called later; error-first; callback hell | A2.1 |
| Promises | Pending → fulfilled or rejected, settled exactly once | A2.2 |
| async/await | Promise syntax; suspends the function, not the thread | A2.2 |
| nextTick vs setImmediate vs setTimeout | Before the loop continues vs check phase vs timers phase | A2.3 |
| CommonJS vs ESM | `require` vs `import`; sync vs static; copied value vs live binding; `import()` for conditional loading | A1.2 |
| Express middleware | `(req, res, next)` in registration order; end the response or call `next()` | A4.2 |
| Streams | Chunks, not whole files: Readable, Writable, Duplex, Transform; use `pipeline` | A3.3 |
| Buffers | Raw binary data outside the V8 heap; `alloc` is safe, `allocUnsafe` is not | A3.4 |
| readFile vs createReadStream | Whole file in memory vs chunks with constant memory | A3.3 |
| EventEmitter | In-process pub-sub; synchronous `emit`; not a durable queue | A2.4 |
| spawn vs exec vs fork | Streams vs shell + buffered output vs Node process + IPC | A6.1 |
| Error handling | Match the execution model; one central Express error handler | A7.4 |
| Operational vs programmer errors | Expected failures you handle vs bugs you fix | A7.4 |
| uncaughtException / unhandledRejection | Fatal: log, shut down gracefully, restart | A7.4 |
| libuv | Event loop, OS async I/O, and a 4-thread pool (fs, DNS, crypto, zlib) | A1.1 |
| Worker threads | Parallel threads for CPU-heavy JS; use a pool | A6.2 |
| Cluster vs child process vs workers | Scale the server vs run other programs vs CPU work in threads | A6.2, A6.3 |
| Backpressure | Producer faster than consumer; `write()` returns false, wait for `'drain'`; use `pipeline` | A3.3 |
| Garbage collection | V8 generational: scavenge young, mark-sweep-compact old; only unreachable objects are freed | A6.4 |
| Memory leaks | Retained references; confirm with metrics, compare heap snapshots, fix the retainer | A6.4 |
| CPU-bound vs I/O-bound | Computing vs waiting; workers vs async I/O | A6.2 |
| Slow API | Measure, find the bottleneck, fix it, verify | A6.4 |
| Backend security | Defence in depth; authentication ≠ authorisation | A5.4 |
| Distributed rate limiting | Shared, atomic Redis counters; trust the proxy carefully | A5.4 |
| 10,000 concurrent requests | Clarify the load; stateless horizontal scaling, pooling, caching, queues, load tests | A6.3 |
| Duplicate requests | Atomic idempotency-key reservation + unique constraints | P4.1 |
| Background jobs | BullMQ + Redis; 202 + job ID; retries with backoff; idempotent workers | A6.2 |
| Connection pooling | Reuse connections; budget the total across instances | A6.3 |
| Large video uploads | Presigned multipart uploads to S3, then queue + FFmpeg workers | A3.3 |
| WebSockets | Upgrade to a persistent two-way channel; Redis adapter to scale | A4.4 |
| Payment webhooks | Verify the signature, persist a unique event ID, return 200 fast, process idempotently, reconcile | A5.1 |
| Race conditions | Atomic conditional updates and transactions; ACID; locking | P4.1 |
| Distributed consistency | Saga + compensations, transactional outbox, idempotent consumers | P4.1 |
| Production crashes | Restore service first (roll back), then logs, metrics, root cause, postmortem | A7.4 |
| Scalable architecture | Stateless instances behind an LB, modules, Redis, DB, queues, observability | Architecture |
| Monolith vs microservices | Start with a modular monolith; split only when scaling, deployment or ownership justifies it | Architecture |
| NestJS structure | Feature modules: controller (HTTP), service (logic), DTO, guard, interceptor, filter | Architecture |
| JWT / OAuth / RBAC | Token format vs delegated-access framework (+ OIDC) vs permission model | A5.2 |
| Redis caching | Cache-aside, TTL with jitter, invalidate on write, stampede protection | A6.4 |
| Kafka vs RabbitMQ | Retained, replayable log vs routed task broker; BullMQ is often enough | A6.2 |
| Observability | Logs (what), metrics (how healthy), traces (where); link them with IDs | A7.4 |
| Testing | Unit + integration (real DB) + E2E, in CI; behaviour over coverage | A7.1 |
| Deployment | Multi-stage non-root image, Nginx TLS proxy, health checks, graceful shutdown; PM2 only without an orchestrator | A6.3 |
| Tech Lead | Quality + delivery + business; reviews, mentoring, ADRs, tech debt by impact | Leadership |

**Q: Fundamentals self-test. Can you answer these without looking?**
A: Answers are in brackets.
1. Which JavaScript engine does Node.js use? *(V8)*
2. Is Node.js entirely single-threaded? *(No. JS runs on one main thread, but the runtime uses additional threads.)*
3. What coordinates asynchronous callbacks in Node.js? *(The event loop.)*
4. Which event loop phase runs `setImmediate` callbacks? *(Check.)*
5. Which operation blocks the JavaScript thread: `fs.readFile()`, `fs.readFileSync()`, `setTimeout()` or `Promise.resolve()`? *(`fs.readFileSync()`.)*
6. What is callback hell? *(Nested callbacks that become hard to read and maintain.)*
7. How many states does a Promise have? *(Three: pending, fulfilled, rejected.)*
8. Does `await` block the Node.js main thread? *(No. It suspends only the surrounding async function.)*
9. Which can starve the event loop if scheduled recursively? *(`process.nextTick()`.)*
10. Which syntax do ES Modules use? *(`import`/`export`.)*

**Q: Core Node.js and Express self-test. Can you answer these without looking?**
A: Answers are in brackets.
1. Which function passes control to the next Express middleware? *(`next()`.)*
2. Which stream type supports both reading and writing? *(Duplex. Transform is a special kind of Duplex.)*
3. Which is generally better for reading a very large file? *(`fs.createReadStream()`.)*
4. Which Node.js object handles raw binary data? *(Buffer.)*
5. Which `child_process` method buffers command output? *(`exec()`, and `execFile()`.)*
6. Which EventEmitter method triggers an event? *(`emit()`.)*
7. What is the Express error middleware signature? *(`(err, req, res, next)`.)*
8. A database connection timeout is what kind of error? *(Operational.)*
9. What should you do after an uncaught exception in production? *(Log, shut down gracefully, and let a supervisor restart the process.)*
10. What is the main purpose of `fork()`? *(To create a separate Node.js process with an IPC channel.)*

**Q: Advanced Node.js self-test. Can you answer these without looking?**
A: Answers are in brackets.
1. What is libuv primarily responsible for? *(The event loop and asynchronous I/O, plus the thread pool.)*
2. Which Node.js feature is best for CPU-intensive JavaScript? *(Worker threads.)*
3. What creates multiple Node.js processes that can share a server port? *(Cluster.)*
4. What is stream backpressure? *(A producer that is faster than its consumer.)*
5. Which engine performs garbage collection in Node.js? *(V8.)*
6. What commonly causes a memory leak? *(Holding references to objects you no longer need.)*
7. Which is CPU-bound: a DB query, an HTTP request, a complex calculation, or reading an API response? *(The complex calculation.)*
8. What should you do first when an API is slow? *(Measure and identify the bottleneck.)*
9. What is the difference between authentication and authorisation? *(Authentication identifies the user; authorisation checks what they're allowed to do.)*
10. Why use Redis for distributed rate limiting? *(It provides shared state across all application instances.)*

**Q: Scenario self-test. Can you answer these without looking?**
A: Answers are in brackets.
1. What is the primary purpose of horizontal scaling? *(Adding more application instances.)*
2. What prevents duplicate payment processing? *(Idempotency keys and unique constraints, not disabling the button.)*
3. What does BullMQ use as its backing store? *(Redis.)*
4. What is the main purpose of connection pooling? *(Reusing database connections.)*
5. How should very large video uploads be handled? *(Multipart or streaming uploads straight to object storage.)*
6. What communication model do WebSockets provide? *(Persistent, bidirectional communication.)*
7. What must you verify before trusting a payment webhook? *(Its signature.)*
8. What prevents overselling the last product? *(An atomic conditional database update.)*
9. Which pattern manages distributed business transactions with compensating actions? *(Saga.)*
10. What is the first priority during a production outage? *(Restoring service and reducing customer impact.)*

**Q: Final senior backend self-test. Can you answer these without looking?**
A: Answers are in brackets.
1. The API receives 10,000 requests at once. What do you focus on first? *(Bottleneck analysis, load balancing, connection pooling and caching.)*
2. Two identical payment requests arrive at the same moment. How do you prevent a double charge? *(An idempotency key with atomic database constraints.)*
3. A video takes 2 minutes to process. What's the better architecture? *(A background queue, with a job status returned to the client.)*
4. When should a NestJS module become a microservice? *(When independent scaling, deployment or ownership justifies it.)*
5. What happens when cached data in Redis expires? *(The key is gone from the cache and must be fetched again from the source.)*
6. Which tool is meant for distributed tracing: OpenTelemetry, Nodemon, Prettier or PM2? *(OpenTelemetry.)*
7. A BullMQ job fails and is retried. What must the job support? *(Idempotent processing.)*
8. What is the best testing strategy? *(Unit, integration and end-to-end tests.)*
9. What does Nginx do in a typical Node deployment? *(Acts as a reverse proxy and terminates TLS.)*
10. What is a tech lead's most important responsibility? *(Balancing technical quality, team delivery and business priorities.)*

Tip: be able to explain all of the fundamentals aloud before moving on. Streams, Worker Threads, performance and production debugging all build on the event loop.

**Q: Which topics matter most for interviews?**
A: The event loop, streams, worker threads vs cluster, and error handling.
