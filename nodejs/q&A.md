# Node.js — Questions & Answers

Questions and answers drawn from every file in this folder, in curriculum order. Each section links back to its source file.

- [Phase 1 — Node.js Runtime Fundamentals](#phase-1--nodejs-runtime-fundamentals)
- [Phase 2 — Asynchronous Programming](#phase-2--asynchronous-programming)
- [Phase 3 — File System & Streams](#phase-3--file-system--streams)
- [Phase 4 — Networking & HTTP](#phase-4--networking--http)
- [Phase 5 — Security & Cryptography](#phase-5--security--cryptography)
- [Phase 6 — Performance & Concurrency](#phase-6--performance--concurrency)
- [Phase 7 — Testing & Debugging](#phase-7--testing--debugging)
- [Study Strategy](#study-strategy)

---

## Phase 1 — Node.js Runtime Fundamentals

### [A1.1 — How Node.js Runs JavaScript](A1.1-how-nodejs-runs-javascript.md)

**Q: What is Node.js made of?**
A: V8 (parses, JIT-compiles and garbage-collects JavaScript), libuv (the event loop and async I/O, plus a thread pool), and bindings plus a standard library (`node:fs`, `node:http`, `node:crypto`…) that expose OS features.

**Q: What runs on libuv's thread pool?**
A: File system operations, DNS lookups and some crypto. It has 4 threads by default, configurable with `UV_THREADPOOL_SIZE`. Network sockets use the OS's non-blocking APIs (epoll, kqueue, IOCP) instead.

**Q: How can one Node process handle thousands of connections?**
A: JavaScript runs on one thread, but I/O is handed to libuv; the main thread never waits idle for I/O, and callbacks run when results are ready.

**Q: What is the downside of single-threaded execution?**
A: CPU-heavy JavaScript blocks everything — a 2-second loop delays all timers and requests by 2 seconds.

**Q: Which web-standard globals does Node provide?**
A: `fetch`, `URL`, `AbortController`, `TextEncoder`, `structuredClone`, `setTimeout`, `crypto.randomUUID()` — plus Node-specific `process`, `Buffer` and `globalThis`. There is no `window` or `document`.

**Q: What do `--watch`, `--env-file` and `--inspect` do?**
A: Restart on file changes, load environment variables from a file, and enable the debugger.

### [A1.2 — Modules: CommonJS & ESM](A1.2-modules-commonjs-and-esm.md)

**Q: CommonJS vs ES Modules?**
A: CJS uses synchronous `require()` and `module.exports`, with cached results. ESM uses static `import`/`export`, loads asynchronously, and allows top-level `await`.

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

**Q: What is the error-first callback convention?**
A: The callback is the last argument and receives `(err, ...results)`; `err` is `null` on success. Always check `err` first.

**Q: Why can't try/catch catch errors in callbacks?**
A: The callback runs later on a fresh call stack, after the `try` block has finished; errors must be passed as values.

**Q: What are the rules for writing callback APIs?**
A: Call back exactly once (`return callback(err)`), always asynchronously, and never throw from an async function.

**Q: What is "releasing Zalgo"?**
A: An API that sometimes calls back synchronously (e.g. on a cache hit) and sometimes asynchronously, making ordering unpredictable. Defer sync results with `process.nextTick`.

**Q: What is callback hell?**
A: Deeply nested sequential callbacks with duplicated error handling and manual counters for parallelism.

**Q: How do you convert callback APIs to promises?**
A: `util.promisify(fn)` — or use built-in promise modules like `node:fs/promises`.

### [A2.2 — Promises & async/await](A2.2-promises-and-async-await.md)

**Q: What are a promise's states?**
A: Pending, then settled exactly once as fulfilled or rejected. `.then` callbacks always run asynchronously as microtasks.

**Q: What is the common sequential-await bug?**
A: Awaiting independent operations one by one (time A + B) instead of `Promise.all` (time max(A, B)).

**Q: `all` vs `allSettled` vs `race` vs `any`?**
A: `all` fails fast on any rejection; `allSettled` waits for all and never rejects; `race` settles with the first to settle; `any` resolves with the first success and rejects with `AggregateError` if all fail.

**Q: Why is unbounded parallelism a bug?**
A: `Promise.all` over 50,000 files opens them all at once → `EMFILE`. Use a concurrency limit (N workers pulling from a queue).

**Q: How do you cancel async work?**
A: Pass an `AbortController`'s `signal` to APIs that accept it and call `abort()`. `AbortSignal.timeout(ms)` aborts automatically; `AbortSignal.any()` combines signals.

**Q: What happens to an unhandled rejection?**
A: It triggers `unhandledRejection`, and Node crashes by default since v15.

### [A2.3 — The Event Loop in Depth](A2.3-the-event-loop-in-depth.md)

**Q: What are the event loop phases?**
A: Timers → pending callbacks → idle/prepare → poll (I/O) → check (`setImmediate`) → close callbacks.

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

### [A2.4 — EventEmitter](A2.4-eventemitter.md)

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

**Q: Why use streams?**
A: They process data in chunks (64 KB for files), so memory stays flat regardless of size and processing starts early.

**Q: What are the four stream types?**
A: Readable (file read, `req`), Writable (file write, `res`), Duplex (TCP socket), Transform (gzip, hash).

**Q: What is backpressure?**
A: When a producer outpaces a consumer, `write()` returns `false` once the buffer exceeds `highWaterMark`; the producer must wait for `'drain'`.

**Q: Why use `pipeline` instead of `.pipe()`?**
A: `pipeline` propagates errors, destroys every stream on failure, handles backpressure, returns a promise, and accepts async generators as stages.

**Q: How do you consume a stream simply?**
A: `for await (const chunk of readable)`, which respects backpressure.

**Q: Why must line splitting keep a leftover?**
A: A chunk can end mid-line; the partial line is carried into the next chunk.

### [A3.4 — Buffers & Binary Data](A3.4-buffers-and-binary-data.md)

**Q: What is a Buffer?**
A: A fixed-length byte sequence, a subclass of `Uint8Array`, stored outside the V8 heap.

**Q: `alloc` vs `allocUnsafe`?**
A: `alloc` zero-fills; `allocUnsafe` is faster but may contain old memory — only use it if you overwrite every byte.

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

### [P4.1 — Framework-Free REST API](P4.1-framework-free-rest-api.md)

**Q: How are concurrent writes to the JSON file kept safe?**
A: Changes are applied in memory, then saves are chained on a promise queue so writes never interleave; each uses an atomic temp-file rename, and a failed write doesn't block later ones.

**Q: How does conditional GET work?**
A: Responses carry an `ETag`; a matching `If-None-Match` returns 304.

**Q: How is CORS preflight handled?**
A: `OPTIONS` requests with `Access-Control-Request-Method` get a 204 with allowed methods and headers, skipping the router.

**Q: What does graceful shutdown do?**
A: Stops accepting connections, closes idle ones, and flushes pending writes before exit.

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

**Q: `'error'` vs a non-zero exit?**
A: `'error'` means the process couldn't start (e.g. `ENOENT`); a failing command still starts and exits non-zero.

**Q: How do you stop a hung child?**
A: Use `timeout` or `signal: AbortSignal.timeout()`, send `SIGTERM`, and escalate to `SIGKILL` after a deadline.

### [A6.2 — Worker Threads](A6.2-worker-threads.md)

**Q: When do worker threads help?**
A: For CPU-bound JavaScript; I/O-bound work is already async.

**Q: How do workers communicate?**
A: Structured-clone messages, transferred `ArrayBuffer`s (zero copy), or `SharedArrayBuffer` coordinated with `Atomics`.

**Q: Why use a pool?**
A: Starting a worker costs milliseconds and memory; a fixed pool reuses workers and queues tasks.

**Q: Workers vs child processes vs cluster?**
A: Workers share a process and can share memory (CPU tasks); child processes isolate programs; cluster scales an HTTP server across cores.

### [A6.3 — Cluster & Load Distribution](A6.3-cluster-and-load-distribution.md)

**Q: How does cluster work?**
A: The primary forks workers that share one port; connections are distributed round-robin (except on Windows).

**Q: Do workers share memory?**
A: No — shared state must live in Redis or a database.

**Q: How does a zero-downtime restart work?**
A: Replace workers one at a time: start a new one, wait for `'listening'`, then gracefully close the old one.

**Q: When avoid cluster?**
A: In containers/Kubernetes, prefer one process per container and scale replicas.

### [A6.4 — Profiling & Memory Leaks](A6.4-profiling-and-memory-leaks.md)

**Q: How do you find CPU hot spots?**
A: Run under load with `--cpu-prof` and read the flame graph for wide frames in your code.

**Q: `heapUsed` vs `rss` vs `external`?**
A: Live JS objects; total process memory; memory for Buffers outside the heap.

**Q: What is a leak vs normal growth?**
A: Normal memory rises and falls with GC; a leak keeps growing across GCs under steady load.

**Q: How do you find a leak?**
A: Compare two heap snapshots, sort by delta, and follow retainers to the code holding objects.

**Q: What does ELU near 100% mean?**
A: The event loop is CPU-saturated; reduce work per request or add workers/processes.

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

**Q: Why use error codes rather than messages?**
A: Messages change; stable `code`s let callers branch reliably.

**Q: What is `cause` for?**
A: Keeping the original low-level error when rethrowing a higher-level one.

**Q: Operational vs programmer errors?**
A: Handle and recover from operational failures; for bugs, log and let the process crash and be restarted.

**Q: What should an `uncaughtException` handler do?**
A: Log, stop accepting work, and exit — the process state is unknown.

**Q: What is a diagnostic report?**
A: A JSON snapshot with JS and native stacks, heap stats, handles and environment, written on fatal errors or on demand.

---

## Study Strategy

### [HOW_TO_LEARN.md](HOW_TO_LEARN.md)

**Q: Why does the curriculum avoid frameworks?**
A: Frameworks are built from `node:http`, streams, EventEmitter and the event loop; knowing those makes any framework easy to learn and debug.

**Q: What is the learning loop?**
A: Read → type → run → break it on purpose → explain why → fix.

**Q: Why might a process never exit?**
A: An open server, socket, interval or connection keeps the event loop alive — close it or `unref()` it.

**Q: Which topics matter most for interviews?**
A: The event loop, streams, worker threads vs cluster, and error handling.
