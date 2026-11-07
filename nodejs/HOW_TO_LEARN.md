# HOW TO LEARN — Backend Developer using Node.js

A practical guide to getting the most out of this curriculum.

---

## The Core Principle

This curriculum deliberately avoids frameworks. Express, NestJS, and Fastify are all built from the pieces you will use directly here — `node:http`, streams, EventEmitter, the event loop. If you understand those pieces, every framework becomes easy to learn and easy to debug.

The loop for every concept:

```
Read the Background → Type the code → Run it → Break it on purpose → Explain why it broke → Fix it
```

The "break it" step matters most. Remove an `await`, block the event loop with a `while` loop, forget to handle the `error` event — and watch what happens.

---

## How to Work Through an Assignment

1. **Read the whole file first** — Goal, Background, Tasks, Deliverables.
2. **Create a fresh folder** for each assignment:
   ```bash
   mkdir a2-3-event-loop && cd a2-3-event-loop
   npm init -y && npm pkg set type=module
   ```
3. **Type every snippet** — do not paste. Node APIs have many similar names (`readFile`, `readFileSync`, `createReadStream`); typing them builds the distinction.
4. **Run after every task** with `node --watch file.js` so you see changes immediately.
5. **Predict before you run.** For async code, write down the order you expect the logs to appear, then run it. Being wrong is where you learn.
6. **Tick a deliverable only when you can explain it** without looking at the file.

### Time budget per assignment
- Background: 15–20 min
- Tasks: 60–90 min
- Experiments & deliverables: 20–30 min

---

## How to Work Through a Project

1. **Read the full spec and architecture** before writing code.
2. **Create the directory tree with empty files** first.
3. **Build the smallest end-to-end slice** (one command, one route) before adding features.
4. **Do not read the Implementation section until you are stuck for 20+ minutes.** It is there to unblock you, not to copy.
5. **Write at least one test** per project from Phase 7 onwards (and retroactively for earlier ones).
6. **Commit after each working feature** with a descriptive message.

---

## Environment Setup

```bash
# Node.js 22+ via nvm (24 LTS recommended)
nvm install 24 && nvm use 24
node --version

# Useful global tools
npm install -g npm@latest
```

Recommended VS Code extensions: **ESLint**, **Prettier**, **Error Lens**, **REST Client**.

Recommended `package.json` baseline for every exercise:

```json
{
  "type": "module",
  "scripts": {
    "dev": "node --watch src/index.js",
    "start": "node src/index.js",
    "test": "node --test"
  },
  "engines": { "node": ">=22" }
}
```

---

## Mental Models to Keep in Your Head

### One thread runs your JavaScript
Everything you write runs on a single thread. I/O happens elsewhere (the OS or libuv's thread pool) and comes back as callbacks. **If your code is busy, nothing else runs** — not timers, not other requests.

### Order of execution
```
Synchronous code
  → process.nextTick queue
  → Promise microtasks
  → Event loop phase: timers → pending → poll (I/O) → check (setImmediate) → close
```

### Streams over buffers
If data might be large, stream it. `readFile` on a 4 GB file crashes; `createReadStream` uses ~64 KB of memory.

### Errors have two flavours
- **Operational** (file missing, network down) → handle and recover.
- **Programmer** (`undefined is not a function`) → fix the bug; don't hide it.

---

## Debugging Node.js — Common Issues

### `SyntaxError: Cannot use import statement outside a module`
Your `package.json` is missing `"type": "module"`, or the file ends in `.cjs`. Add the field or rename to `.mjs`.

### `ERR_MODULE_NOT_FOUND` for a relative import
ESM requires file extensions: `import { x } from './utils.js'`, not `'./utils'`.

### `__dirname is not defined`
ESM has no `__dirname`. Use `import.meta.dirname` (Node 20.11+) or `path.dirname(fileURLToPath(import.meta.url))`.

### The process never exits
Something is keeping the event loop alive: an open server, socket, interval, or database connection. Close it, or `unref()` timers that shouldn't hold the process open.

### `MaxListenersExceededWarning`
You're adding listeners in a loop (often inside a request handler) without removing them. This is usually a memory leak.

### `EMFILE: too many open files`
You started thousands of file operations at once. Limit concurrency (see A2.2).

---

## What "Done" Means

An assignment is done when:
- [ ] All tasks run without errors
- [ ] Every deliverable is ticked
- [ ] You can explain every line you wrote
- [ ] You rewrote the core pattern once from memory

A project is done when:
- [ ] All deliverables pass
- [ ] It handles bad input and failures gracefully (no crashes, no hangs)
- [ ] It has a README with usage examples
- [ ] It is committed to git

---

## Progression Advice

- **New to Node.js:** spend double time on Phase 2. Almost every Node bug in production traces back to misunderstanding async execution order or error propagation.
- **Already use Express:** skim Phase 1, but do every project. Phase 4's framework-free API and Phase 9's mini framework will change how you think about Express.
- **Preparing for interviews:** the event loop (A2.3), streams (A3.3), worker threads vs cluster (A6.2, A6.3), and error handling (A7.4) are the most commonly asked Node.js topics. Use [q&A.md](q&A.md) for revision.
- **Preparing for production work:** Phases 6 and 8 matter most — profiling, graceful shutdown, configuration, and containers.

---

## Resources

- [Node.js API docs](https://nodejs.org/api/) — the primary reference; read the page for every module you use
- [Node.js Learn guides](https://nodejs.org/en/learn) — official tutorials, including the event loop guide
- [libuv design overview](https://docs.libuv.org/en/v1.x/design.html)
- [npm docs](https://docs.npmjs.com/)
