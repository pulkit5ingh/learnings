# Backend Developer using Node.js — Full Roadmap

A structured, project-driven curriculum for mastering the Node.js runtime itself — no frameworks. You will learn how Node executes code, how async I/O really works, and how to build servers, CLIs, and production services using only the standard library (plus a handful of small packages).

**Stack:** Node.js 22+ (24 LTS recommended) · ES Modules · built-in `node:` modules
**Duration:** ~5 months (part-time) or ~2.5 months (full-time)
**Pairs with:** `../expressjs/` and `../nestjs/` — once you finish this, frameworks become thin layers over things you already understand.

---

## How to Use This Curriculum

- Complete assignments in order within each phase before starting projects.
- Each assignment is hands-on — open your editor and write the code.
- Projects integrate everything from the phase. Build them from scratch, not from templates.
- Every file uses ES Modules (`"type": "module"` in `package.json`) and `node:`-prefixed imports.
- See [HOW_TO_LEARN.md](HOW_TO_LEARN.md) for the full study strategy and [q&A.md](q&A.md) for revision questions.

---

## Phase Overview

| Phase | Name | Assignments | Projects |
|-------|------|-------------|----------|
| 1 | Node.js Runtime Fundamentals | A1.1–A1.4 | P1.1–P1.2 |
| 2 | Asynchronous Programming | A2.1–A2.4 | P2.1–P2.2 |
| 3 | File System & Streams | A3.1–A3.4 | P3.1–P3.2 |
| 4 | Networking & HTTP | A4.1–A4.4 | P4.1–P4.2 |
| 5 | Security & Cryptography | A5.1–A5.4 | P5.1–P5.2 |
| 6 | Performance & Concurrency | A6.1–A6.4 | P6.1–P6.2 |
| 7 | Testing & Debugging | A7.1–A7.4 | P7.1–P7.2 |
| 8 | Tooling & Production | A8.1–A8.4 | P8.1–P8.2 |
| 9 | Advanced Patterns | A9.1–A9.4 | P9.1–P9.3 |

---

## Phase 1 — Node.js Runtime Fundamentals

**Goal:** Understand what Node.js is, how it loads and runs code, and how to build command-line programs.

### Assignments
- **[A1.1](A1.1-how-nodejs-runs-javascript.md)** — How Node.js runs JavaScript: V8, libuv, the REPL, `node` CLI flags, `--watch`
- **[A1.2](A1.2-modules-commonjs-and-esm.md)** — Modules: CommonJS vs ESM, `import.meta`, module resolution, the `node:` prefix
- **[A1.3](A1.3-npm-and-package-json.md)** — npm & package.json: dependencies, semver, lockfiles, scripts, `npx`
- **[A1.4](A1.4-process-globals-and-the-cli.md)** — Process, globals & the CLI: `process.argv`, `util.parseArgs`, env, exit codes, stdin/stdout

### Projects
- **[P1.1](P1.1-cli-todo-app.md)** — CLI todo app: subcommands, flags, JSON file persistence, coloured output
- **[P1.2](P1.2-project-scaffolder-cli.md)** — Project scaffolder: interactive prompts, template copying, variable substitution

---

## Phase 2 — Asynchronous Programming

**Goal:** Master every async style in Node.js and understand exactly when each callback runs.

### Assignments
- **[A2.1](A2.1-callbacks-and-error-first-patterns.md)** — Callbacks & error-first patterns: callback hell, `util.promisify`, never-throw rules
- **[A2.2](A2.2-promises-and-async-await.md)** — Promises & async/await: combinators, concurrency limits, `AbortController`, timeouts
- **[A2.3](A2.3-the-event-loop-in-depth.md)** — The event loop in depth: phases, microtasks, `nextTick`, `setImmediate`, blocking
- **[A2.4](A2.4-eventemitter.md)** — EventEmitter: `on`/`once`/`off`, the `error` event, `events.once`, async iteration

### Projects
- **[P2.1](P2.1-concurrent-url-health-checker.md)** — Concurrent URL health checker: concurrency pool, timeouts, retries, summary report
- **[P2.2](P2.2-event-driven-task-runner.md)** — Event-driven task runner: dependency graph, lifecycle events, parallel execution

---

## Phase 3 — File System & Streams

**Goal:** Read, write, and transform data of any size efficiently.

### Assignments
- **[A3.1](A3.1-the-fs-module.md)** — The fs module: promises API, flags, stats, atomic writes, error codes
- **[A3.2](A3.2-paths-directories-and-file-watching.md)** — Paths, directories & file watching: `node:path`, recursive `readdir`, `fs.watch`, globbing
- **[A3.3](A3.3-streams.md)** — Streams: Readable/Writable/Transform, backpressure, `pipeline`, async iterators
- **[A3.4](A3.4-buffers-and-binary-data.md)** — Buffers & binary data: encodings, `TypedArray`s, reading binary formats, base64/hex

### Projects
- **[P3.1](P3.1-log-file-analyzer.md)** — Log file analyzer: stream multi-GB logs line by line, aggregate stats, gzip support
- **[P3.2](P3.2-file-backup-tool.md)** — File backup tool: incremental copy by hash, compression, restore, progress reporting

---

## Phase 4 — Networking & HTTP

**Goal:** Build servers and clients with the core networking modules — no frameworks.

### Assignments
- **[A4.1](A4.1-http-servers-with-node-http.md)** — HTTP servers with node:http: request/response lifecycle, headers, status codes, keep-alive
- **[A4.2](A4.2-routing-and-request-parsing-from-scratch.md)** — Routing & request parsing from scratch: URL patterns, JSON bodies, size limits, errors
- **[A4.3](A4.3-http-clients-and-fetch.md)** — HTTP clients & fetch: global `fetch`, streaming bodies, timeouts, retries with backoff
- **[A4.4](A4.4-tcp-and-udp-with-net-and-dgram.md)** — TCP & UDP with net and dgram: sockets, framing, a line protocol, broadcast

### Projects
- **[P4.1](P4.1-framework-free-rest-api.md)** — Framework-free REST API: router, middleware chain, validation, JSON file store
- **[P4.2](P4.2-static-file-server.md)** — Static file server: MIME types, range requests, ETags, gzip, directory listing

---

## Phase 5 — Security & Cryptography

**Goal:** Use `node:crypto` correctly and defend Node services against common attacks.

### Assignments
- **[A5.1](A5.1-the-crypto-module.md)** — The crypto module: hashing, HMAC, secure randomness, `timingSafeEqual`, Web Crypto
- **[A5.2](A5.2-password-hashing-and-signed-tokens.md)** — Password hashing & signed tokens: `scrypt`, salts, building a JWT by hand
- **[A5.3](A5.3-encryption-with-aes-gcm.md)** — Encryption with AES-GCM: keys, IVs, auth tags, key derivation, envelope format
- **[A5.4](A5.4-nodejs-security-best-practices.md)** — Node.js security best practices: path traversal, prototype pollution, ReDoS, `npm audit`, permission model

### Projects
- **[P5.1](P5.1-encrypted-secrets-vault-cli.md)** — Encrypted secrets vault CLI: master password, scrypt-derived key, AES-GCM file
- **[P5.2](P5.2-signed-url-service.md)** — Signed URL service: HMAC-signed expiring download links, replay protection

---

## Phase 6 — Performance & Concurrency

**Goal:** Use every CPU core, keep the event loop free, and find performance problems with real tools.

### Assignments
- **[A6.1](A6.1-child-processes.md)** — Child processes: `spawn`, `exec`, `execFile`, `fork`, IPC, shell-injection safety
- **[A6.2](A6.2-worker-threads.md)** — Worker threads: CPU-bound work, `postMessage`, transferables, `SharedArrayBuffer`, worker pools
- **[A6.3](A6.3-cluster-and-load-distribution.md)** — Cluster & load distribution: primary/worker model, restarts, zero-downtime reloads
- **[A6.4](A6.4-profiling-and-memory-leaks.md)** — Profiling & memory leaks: `--cpu-prof`, heap snapshots, event loop lag, `perf_hooks`

### Projects
- **[P6.1](P6.1-parallel-file-hasher.md)** — Parallel file hasher: worker pool hashing thousands of files, benchmark vs single thread
- **[P6.2](P6.2-clustered-http-server.md)** — Clustered HTTP server: one worker per core, crash recovery, rolling restart

---

## Phase 7 — Testing & Debugging

**Goal:** Test Node code with the built-in runner and debug problems systematically.

### Assignments
- **[A7.1](A7.1-the-node-test-runner.md)** — The node:test runner: `test`/`describe`, `assert`, hooks, watch mode, coverage
- **[A7.2](A7.2-mocking-and-test-doubles.md)** — Mocking & test doubles: `mock.fn`, `mock.method`, mock timers, dependency injection
- **[A7.3](A7.3-debugging-with-the-inspector.md)** — Debugging with the inspector: `--inspect`, breakpoints, VS Code, `debugger`, `util.inspect`
- **[A7.4](A7.4-error-handling-and-diagnostics.md)** — Error handling & diagnostics: custom errors, `cause`, `uncaughtException`, `unhandledRejection`, reports

### Projects
- **[P7.1](P7.1-tested-utility-library.md)** — Tested utility library: retry, debounce, LRU cache, rate limiter — 90%+ coverage
- **[P7.2](P7.2-debugging-a-leaky-service.md)** — Debugging a leaky service: find and fix a memory leak and a blocked event loop

---

## Phase 8 — Tooling & Production

**Goal:** Ship Node.js code that is typed, configured, observable, and packaged.

### Assignments
- **[A8.1](A8.1-typescript-with-nodejs.md)** — TypeScript with Node.js: type stripping, `tsc`, `tsx`, tsconfig for Node, ESM interop
- **[A8.2](A8.2-configuration-and-environment-variables.md)** — Configuration & environment variables: `--env-file`, validation, 12-factor config
- **[A8.3](A8.3-logging-and-graceful-shutdown.md)** — Logging & graceful shutdown: structured JSON logs, signals, draining connections
- **[A8.4](A8.4-publishing-npm-packages.md)** — Publishing npm packages: `exports` map, `bin`, dual ESM/CJS, versioning, `npm publish`

### Projects
- **[P8.1](P8.1-published-cli-package.md)** — Published CLI package: a typed, tested CLI shipped to npm with a release workflow
- **[P8.2](P8.2-dockerized-nodejs-service.md)** — Dockerized Node.js service: multi-stage image, non-root user, health checks, CI

---

## Phase 9 — Advanced Patterns

**Goal:** Build real-time systems and the infrastructure that frameworks hide from you.

### Assignments
- **[A9.1](A9.1-websockets-with-ws.md)** — WebSockets with ws: upgrade handshake, broadcasting, heartbeats, backpressure
- **[A9.2](A9.2-server-sent-events-and-streaming-responses.md)** — Server-Sent Events & streaming responses: `text/event-stream`, reconnection, chunked output
- **[A9.3](A9.3-asynclocalstorage-and-request-context.md)** — AsyncLocalStorage & request context: request IDs without passing `req`, `diagnostics_channel`
- **[A9.4](A9.4-building-a-job-queue.md)** — Building a job queue: concurrency, retries with backoff, persistence, graceful drain

### Projects
- **[P9.1](P9.1-real-time-chat-server.md)** — Real-time chat server: rooms, presence, history, heartbeats over WebSockets
- **[P9.2](P9.2-mini-web-framework.md)** — Mini web framework: router, middleware, body parsing, error handling — your own tiny Express
- **[P9.3](P9.3-production-url-shortener.md)** — Production URL shortener: everything combined — API, persistence, rate limiting, logging, tests, Docker

---

## Completion Checklist

- [ ] Phase 1 — Node.js Runtime Fundamentals
- [ ] Phase 2 — Asynchronous Programming
- [ ] Phase 3 — File System & Streams
- [ ] Phase 4 — Networking & HTTP
- [ ] Phase 5 — Security & Cryptography
- [ ] Phase 6 — Performance & Concurrency
- [ ] Phase 7 — Testing & Debugging
- [ ] Phase 8 — Tooling & Production
- [ ] Phase 9 — Advanced Patterns

---

## Recommended Timeline

```
Weeks 1-2:   Phase 1 (Runtime Fundamentals)
Weeks 3-5:   Phase 2 (Async Programming)   ← spend extra time here
Weeks 6-7:   Phase 3 (File System & Streams)
Weeks 8-10:  Phase 4 (Networking & HTTP)
Week 11:     Phase 5 (Security & Crypto)
Weeks 12-13: Phase 6 (Performance & Concurrency)
Weeks 14-15: Phase 7 (Testing & Debugging)
Weeks 16-17: Phase 8 (Tooling & Production)
Weeks 18-20: Phase 9 (Advanced Patterns)
```

---

## Core Modules Reference

| Area | Modules |
|------|---------|
| Process & runtime | `node:process`, `node:util`, `node:os`, `node:perf_hooks` |
| Async | `node:events`, `node:timers/promises`, `node:async_hooks` |
| Files | `node:fs/promises`, `node:path`, `node:stream`, `node:zlib` |
| Networking | `node:http`, `node:https`, `node:net`, `node:dgram`, global `fetch` |
| Security | `node:crypto` |
| Concurrency | `node:child_process`, `node:worker_threads`, `node:cluster` |
| Testing | `node:test`, `node:assert/strict` |
